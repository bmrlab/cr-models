# 视频模型

下面的取值都在 2026-09-27 实测过：提交真实请求，用 ffprobe 核对成片；用了图片或视频输入的，还抽帧看过。有些值没有真的生成过，只是模型的报错信息里列了出来，这些标为「报错列出」。

所有模型输出都是 24fps 的 H.264 MP4。

## 怎么选

| 需要 | 用 |
|---|---|
| 文字生成视频 | 多数场景用 seedance-2.0-fast-text-to-video 或 wan2.7-text-to-video；要 1080p 用 kling、wan2.7 或 happyhorse-1.1；要最快用 veo3.1-fast（只能 8 秒 720p） |
| 从一张图开始（首帧） | wan2.7-image-to-video（最快）、seedance 两个、happyhorse-1.1-image-to-video、minimax-h3-image-to-video、kling-v3-turbo-image-to-video；veo3.1-fast、doubao-seedance-2-5-260628 也行 |
| 给定开头和结尾两帧（首尾帧） | wan2.7-image-to-video、seedance 两个、doubao-seedance-2-5-260628、minimax-h3-image-to-video |
| 同一个角色放进新场景（参考图） | seedance-2.0-fast/mini-reference-to-video 角色还原最好；另有 wan2.7-reference-video、happyhorse-1.1-reference-to-video、doubao-seedance-2-5-260628、minimax-h3-reference-to-video |
| 改一段现有视频（视频编辑） | wan2.7-video-edit、happyhorse-1.0-video-edit |
| 图片只有本地文件 | 只有 minimax 的模型接受；其他模型要先把图放到能公开访问的 URL |
| 不要声音 | kling-v3-text-to-video（永远没有声音）；seedance 系列加 `--audio off` |
| 4:5、5:4、9:21 | happyhorse-1.1 |
| 默认就是 10 秒 | doubao-seedance-2-5-260628 |

**不要用**：kling-v3-omni（任何请求都会被拒）；不带日期的 doubao-seedance-2-5（首帧、首尾帧总是失败，默认还是竖屏）。要 seedance 2.5 就用 doubao-seedance-2-5-260628。

## 图片和视频输入

- 用 `--first-frame`、`--last-frame`、`--reference-image` 给图，用 `--video` 给要编辑的视频。可以是 URL，也可以是本地文件（脚本会转成 data URI）。
- **本地文件只有 minimax 的模型接受。** 其他模型会报「URL scheme must be HTTP(S)」，需要能公开访问的 HTTP(S) 地址。视频输入只测过 URL。
- **`--reference-image` 只配参考图生视频模型用**（名字里带 `reference`），以及 doubao-seedance-2-5-260628。配图生视频模型时，kling-v3-turbo 和 seedance 会把它当成首帧，成片从这张图原样开始。
- 图生视频的比例跟着输入图走。wan2.7、happyhorse-1.1、kling-v3-turbo 的图生视频都不能传 `--aspect-ratio`。
- 文生视频模型里，veo3.1-fast 接受 `--first-frame`；kling-v3-text-to-video 不接受图片，会失败。

## 耗时

从提交到完成的时间。样本少的只供参考，同样的参数也可能差一倍。安排时间时按最长的估。

| 模型 | 条数 | 范围 |
|---|---|---|
| wan2.7-image-to-video | 2 | 34–66 秒 |
| kling-v3-turbo-text-to-video | 1 | 40 秒 |
| happyhorse-1.1-reference-to-video | 2 | 74–76 秒 |
| happyhorse-1.1-image-to-video | 1 | 87 秒 |
| wan2.7-text-to-video | 1 | 89 秒 |
| happyhorse-1.0-video-edit | 1 | 92 秒 |
| happyhorse-1.1-text-to-video | 1 | 101 秒 |
| veo3.1-fast | 3 | 71–207 秒 |
| seedance-2.0-mini-text-to-video | 2 | 114–147 秒 |
| seedance-2.0-fast/mini-reference-to-video | 2 | 125–134 秒 |
| minimax-h3-reference-to-video | 3 | 137–166 秒 |
| seedance-2.0-fast-image-to-video | 9 | 107–191 秒 |
| seedance-2.0-mini-image-to-video | 9 | 123–220 秒 |
| doubao-seedance-2-5 / -260628 | 6 | 153–211 秒 |
| kling-v3-text-to-video | 17 | 77–242 秒 |
| wan2.7-video-edit | 1 | 240 秒 |
| wan2.7-reference-video | 1 | 273 秒 |
| seedance-2.0-fast-text-to-video | 3 | 121–330 秒 |
| minimax-h3-text-to-video | 16 | 115–332 秒 |
| kling-v3-turbo-image-to-video | 7 | 106–392 秒 |
| minimax-h3-image-to-video | 8 | 143–484 秒 |

---

# 文生视频

## seedance-2.0-fast-text-to-video、seedance-2.0-mini-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 4–15 的整数，默认约 5 秒 |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1` 实测过；`4:3`、`3:4`、`21:9`、`adaptive` 报错列出 |
| `--resolution` | `480p`、`720p`（默认） |
| `--audio` | 默认有声音，`off` 有效 |

输出尺寸：720p 下 16:9 → 1280×720，1:1 → 960×960；480p 下 9:16 → 496×864。

## wan2.7-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 2–15 的整数，默认 5（报错列出范围） |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1`、`4:3`、`3:4`（报错列出） |
| `--resolution` | `720p`（默认）、`1080p`（报错列出） |
| `--audio` | **不要传**，会失败。成片总是有声音 |

## happyhorse-1.1-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 3–15 的整数，默认约 5 秒（报错列出范围） |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1`、`4:3`、`3:4`、`4:5`、`5:4`、`9:21`、`21:9`（报错列出） |
| `--resolution` | `720p`（默认）、`1080p`（报错列出） |
| `--audio` | **不要传**，会失败。成片总是有声音 |

## kling-v3-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 3–15 的整数，默认 5 |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1`。其他比例会失败 |
| `--resolution` | `720p`（默认）、`1080p` |
| `--audio` | **没有效果**：传 `on` 也不会失败，但成片仍然没有声音 |
| 图片输入 | 不支持，会失败 |

输出尺寸：720p 下 16:9 → 1280×720，9:16 → 720×1280，1:1 → 960×960；1080p 下 16:9 → 1920×1080。

## kling-v3-turbo-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 3–15 的整数，默认 5（报错列出范围） |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1`（报错列出） |
| `--resolution` | `720p`（默认）、`1080p`（报错列出） |
| `--audio` | **不要传**，会失败。成片总是有声音（和 kling-v3-text-to-video 相反） |

偶尔会返回 `task_failed`「The generation task failed」，重试一次就好。

## minimax-h3-text-to-video

| 参数 | 用法 |
|---|---|
| `--duration` | 4–15 的整数，默认 4 |
| `--aspect-ratio` | `16:9`（默认）、`9:16`、`1:1`、`4:3`、`3:4`、`21:9` |
| `--resolution` | 只有 `768p`，不传也是 768p |
| `--audio` | **不要传 `off`**，会失败。成片总是有声音 |

成片会比设定的时长略长，最多长 0.6 秒：4 → 4.46s，6 → 6.58s，8 → 8.00s，10 → 10.13s，15 → 15.08s。

输出尺寸：16:9 → 1344×768，9:16 → 768×1344，1:1 → 768×768，4:3 → 1024×768，3:4 → 768×1024，21:9 → 1536×672。

## veo3.1-fast

| 参数 | 用法 |
|---|---|
| `--duration` | **不要传**，会失败。成片固定 8 秒 |
| `--aspect-ratio` | `16:9`（默认）→ 1280×720，`9:16` → 720×1280。另有 `auto`（报错列出） |
| `--resolution` | 只有 `720p`（报错列出） |
| `--audio` | **不要传**，会失败。成片总是有声音 |
| `--first-frame` | 有效 |

## doubao-seedance-2-5-260628

同一个模型支持文生、首帧、首尾帧、参考图。

| 参数 | 用法 |
|---|---|
| 默认 | **10 秒**、1280×720、有声音 |
| `--first-frame`、`--last-frame` | 有效，只收 HTTP(S) URL |
| `--reference-image` | 有效，角色放进了新场景 |
| `--duration`、`--aspect-ratio`、`--resolution` | 非法值只返回「The generation task failed」，看不出合法范围。拿不准就不要传 |

---

# 图生视频

以下都是用一张 1280×720 的横图当首帧测的。

## wan2.7-image-to-video

| 参数 | 用法 |
|---|---|
| `--first-frame`、`--last-frame` | 有效，只收 HTTP(S) URL |
| `--duration` | 2–15 的整数，默认 5（报错列出范围） |
| `--aspect-ratio` | **不要传**，会失败。输出跟着输入图 |
| `--resolution` | `720p`（默认）、`1080p`（报错列出） |

## seedance-2.0-fast-image-to-video、seedance-2.0-mini-image-to-video

| 参数 | 用法 |
|---|---|
| `--first-frame`、`--last-frame` | 有效，**只收 HTTP(S) URL** |
| `--duration` | 4–15 的整数，默认约 5 秒 |
| `--aspect-ratio` | 不传时跟着输入图。`9:16` 有效：从横图中间裁出竖屏，720×1280 |
| `--resolution` | `480p`（横图 → 864×496）、`720p`（默认） |
| `--audio` | 默认有声音，`off` 有效 |

## happyhorse-1.1-image-to-video

| 参数 | 用法 |
|---|---|
| `--first-frame` | 有效，只收 HTTP(S) URL |
| `--last-frame` | **不支持**，会失败 |
| `--duration` | 3–15 的整数，默认约 5 秒（报错列出范围） |
| `--aspect-ratio` | **不要传**，会失败 |
| `--resolution` | `720p`（默认）、`1080p`（报错列出） |

## minimax-h3-image-to-video

| 参数 | 用法 |
|---|---|
| `--first-frame`、`--last-frame` | 有效，URL 和本地文件都能用 |
| `--duration` | 4–15 的整数，默认 4（成片 4.46 秒） |
| `--aspect-ratio` | **不生效**：传什么都出 1344×768（横图输入）。不要传 |
| `--resolution` | 只有 `768p` |
| `--audio` | **不要传 `off`**，会失败 |

## kling-v3-turbo-image-to-video

| 参数 | 用法 |
|---|---|
| `--first-frame` | 有效，只收 HTTP(S) URL |
| `--last-frame` | **不支持**，会失败 |
| `--duration` | 3–15 的整数，默认 5 |
| `--aspect-ratio` | **不要传**，会失败 |
| `--resolution` | `720p`（默认）、`1080p`（→ 1920×1080） |
| `--audio` | **不要传**，会失败。成片总是有声音 |

---

# 参考图生视频

用 `--reference-image` 给一张角色图，提示词描述新场景。成片会把这个角色画进新场景，第一帧不是参考图本身。测试用的是灰背景前的白猫，场景是「在海边沙滩上奔跑」「在壁炉前打盹」。

| 模型 | 角色还原 | 本地文件 | 默认输出 |
|---|---|---|---|
| seedance-2.0-fast-reference-to-video | 最像 | 不接受 | 5.09s、1280×720 |
| seedance-2.0-mini-reference-to-video | 最像 | 不接受 | 5.09s、1280×720 |
| wan2.7-reference-video | 像，脸更圆、毛更蓬 | 不接受 | 5.04s、1280×720 |
| happyhorse-1.1-reference-to-video | 像，跑动时模糊 | 不接受 | 5.16s、1280×720 |
| minimax-h3-reference-to-video | 差得最多，两次都给白猫加上了灰色花纹 | 接受 | 4.46s、1344×768 |

都默认有声音。参数的合法值（报错列出）：

| 模型 | `--duration` | `--resolution` | `--aspect-ratio` |
|---|---|---|---|
| seedance 两个 | 4–15 | 480p、720p | 16:9、9:16、1:1、4:3、3:4、21:9、adaptive |
| happyhorse-1.1-reference-to-video | 3–15 | 720p、1080p | 16:9、9:16、1:1、4:3、3:4、4:5、5:4、9:21、21:9 |
| wan2.7-reference-video | 2–15 | 720p、1080p | 16:9、9:16、1:1、4:3、3:4 |
| minimax-h3-reference-to-video | 报错不说明 | 报错不说明 | 报错不说明 |

---

# 视频编辑

用 `--video` 给一段视频的 URL，提示词说要怎么改。测试时让两个模型把视频里的白猫换成黑猫：都照办了，场景、动作、镜头都和原视频一样。

| 模型 | 输出 |
|---|---|
| wan2.7-video-edit | 5.04s、1280×720、有声音 |
| happyhorse-1.0-video-edit | 5.09s、1280×720、有声音 |

视频输入只测过 URL；时长、分辨率等参数都没测。

## 没测过

`fps`、`seed`、`negative_prompt`、`size`；多张参考图；竖图或方图当首帧时的输出尺寸；`callback_url`。用了要跟用户说明是在试，并核对成片。
