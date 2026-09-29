---
name: cr-media
description: 通过 creative-reasoning（CR）模型网关生成图片和视频。图像：文生图和图片编辑，模型有 gpt-image-2.5-sunburst、gpt-image-2.5-flare、gpt-image-2、nano-banana-beta、gemini-3.1-flash-image、gemini-3.1-flash-lite-image、gemini-3-pro-image、doubao-seedream-5.0-pro、qwen-image-3.0-pro、wan2.7-image-pro 等。视频：文生视频，模型有 seedance-2.0、wan2.7、happyhorse-1.1、kling-v3、minimax-h3、veo3.1-fast、doubao-seedance-2-5 等；图生视频（首帧、首尾帧），模型有 wan2.7、seedance-2.0、happyhorse-1.1、minimax-h3、kling-v3-turbo；参考图生视频（同一个角色放进新场景），模型有 seedance-2.0、wan2.7、happyhorse-1.1、minimax-h3；视频编辑，模型有 wan2.7-video-edit、happyhorse-1.0-video-edit。用户要生成或修改图片、封面、配图、镜头素材、视频片段、B-roll，或者让一张图动起来、让同一个角色出现在不同场景里、修改一段视频，或提到 CR、creative-reasoning、api.creative-reasoning.com、上面这些模型名时使用。
---

# cr-media

用 CR 生成图片和视频。本 skill 目录下有两个脚本，都会一次完成提交、轮询、下载和核对，最后在标准输出打印一行 JSON 摘要：

- `scripts/cr-image`：文生图；加 `--image` 就是编辑图片。
- `scripts/cr-video`：文生视频；加 `--first-frame`（和 `--last-frame`）就是图生视频；换成参考图生视频模型、加 `--reference-image` 就是把图里的角色放进新场景；换成视频编辑模型、加 `--video` 就是改一段现有视频。

下面用 `$SKILL_DIR` 表示本 skill 所在的目录，也就是加载 skill 时给出的 Base directory。

## 准备

需要 `curl`、`jq`、`ffprobe`、`file`、`base64`。缺哪个，脚本会直接报出来。

**API Key。** 脚本先读环境变量 `CR_API_KEY`，没有的话再读 `~/.config/cr/env`。两处都没有时，停下来问用户要 Key，不要去别处找。用户给了 Key，就帮他写进配置文件：

```bash
mkdir -p ~/.config/cr
printf 'CR_API_KEY=%s\n' '<用户的 Key>' > ~/.config/cr/env
chmod 600 ~/.config/cr/env
```

## 流程

1. **挑模型。** 图像读 `references/image-models.md`，视频读 `references/video-models.md`。按需求挑一个模型，参数只用表里列出的值。表里标了「不要传」「不生效」的参数不要传。
2. **先 dry run。** 加 `--dry-run` 看一眼请求体，确认没问题再去掉它正式提交。
3. **生成。**
   ```bash
   # 生成图片
   "$SKILL_DIR/scripts/cr-image" \
     --model gpt-image-2 --prompt "白色背景上的蓝色陶瓷杯" \
     --size 1536x1024 --out ./cr-output --name cup

   # 编辑图片：--image 可以是 URL，也可以是本地文件
   "$SKILL_DIR/scripts/cr-image" \
     --model nano-banana-beta --prompt "把杯子改成红色，背景不变" \
     --image ./cr-output/cup-0.png --out ./cr-output --name cup-red

   # 生成视频
   "$SKILL_DIR/scripts/cr-video" \
     --model kling-v3-text-to-video --prompt "海边日落，镜头缓慢推进" \
     --duration 5 --aspect-ratio 16:9 --resolution 1080p \
     --out ./cr-output --name sunset

   # 图生视频：给定首帧和尾帧。seedance 只收 HTTP(S) URL，本地文件只有 minimax 接受
   "$SKILL_DIR/scripts/cr-video" \
     --model seedance-2.0-fast-image-to-video --prompt "猫从草地走到灰色背景前坐下" \
     --first-frame https://example.com/start.png --last-frame https://example.com/end.png \
     --out ./cr-output --name cat-walk

   # 参考图生视频：同一个角色放进新场景
   "$SKILL_DIR/scripts/cr-video" \
     --model seedance-2.0-fast-reference-to-video --prompt "这只白猫在海边沙滩上奔跑" \
     --reference-image https://example.com/cat.png --out ./cr-output --name cat-beach

   # 视频编辑：改一段现有视频
   "$SKILL_DIR/scripts/cr-video" \
     --model wan2.7-video-edit --prompt "把视频里的白猫换成一只黑猫，其他保持不变" \
     --video https://example.com/cat.mp4 --out ./cr-output --name cat-black
   ```
   - 一张图要 15 秒到 5 分钟，一条视频要 1 到 5 分钟。
   - 要生成多条时，并行跑多个进程。每个 Key 最多同时有 128 个未完成任务。
   - 各选项的完整说明用 `--help` 查看。
4. **核对参数。** 拿摘要和要求对一遍：图片看 `files` 里每张的 `size`、`mime` 和张数；视频看 `duration_s`、`size`、`audio`。不生效的参数不会报错，只有核对才能发现。对不上的地方要告诉用户，不要当成已经满足了要求。
5. **看内容。** 编辑过的图一定要打开看，确认改对了。图生视频要抽成片的第一帧（给了尾帧的话还有最后一帧）和输入图对一下：有的模型会把图挪作他用，而任务照样成功。视频需要判断内容时，用 ffmpeg 抽几帧来看。

## 输出

结果都在 `--out` 指定的目录下（默认 `./cr-output`）：

- 图片：`<name>-<i>.<png|jpg|webp>`，从 0 开始编号
- 视频：`<name>.mp4`
- 两者都有 `<name>.json`：请求、提交响应、终态响应和核对结果。base64 内容已替换成长度说明。任务失败时也会写这个文件，报错在 `final.error` 里。

退出码：`0` 成功，`1` 任务失败，`2` 用法错误或网络错误。

## 规则

- 图像的提示词是顶层 `prompt`，视频的提示词在 `content[].text`。脚本已经处理好了，直接调接口时要注意。
- `--extra '<json>'` 可以加表里没有的字段。这些字段都没测过：用了要跟用户说明是在试，并核对输出。
- 报错大多不说是哪个参数错了。遇到报错先对照表格找原因，不要换着值乱试。
- 等太久超时了，或者会话中断了，用 `--resume <task_id>` 接着等，不要重新提交。task id 在提交后就写进了 `<name>.json`。
- 结果链接的有效期未知，脚本拿到后会立即下载。

## 遇到和表格不一致的情况

比如某个参数没生效、报了表里没有的错、耗时远远超出范围。遇到这种情况时，在给用户的回答里单独说明：用的模型和参数、task id、看到了什么，以及 `<name>.json` 的路径。这些信息足够维护者去复现。

## 参考

- `references/image-models.md`：十二个图像模型各自能用哪些参数、会出什么尺寸。用图像模型前必读。
- `references/video-models.md`：二十多个视频模型（文生、图生、参考图生、视频编辑）各自能用哪些参数、接受哪些输入、会出什么尺寸，以及哪些不要用。用视频模型前必读。
- `references/api.md`：接口、请求和响应结构、错误码、幂等规则。脚本不够用、需要直接调接口时读。
