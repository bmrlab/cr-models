# CR 接口

脚本不够用、需要直接调接口时读这份。除了标「未测」的，下面都实测过。

## 通用

- 地址：`https://api.creative-reasoning.com`
- 请求头：`Authorization: Bearer $CR_API_KEY`、`Content-Type: application/json`
- 所有生成都是异步任务：提交后拿到 `poll_url`，轮询到 `completed` 或 `failed`。

| 用途 | 接口 |
|---|---|
| 生成视频 | `POST /unified/v1/videos/tasks` |
| 生成图片 | `POST /unified/v1/images/tasks` |
| 编辑图片 | `POST /unified/v1/images/edit-tasks` |
| 查询任务 | `GET` 提交响应里的 `poll_url` |

### 提交

请求头里带上 `Idempotency-Key`，1–256 个可打印 ASCII 字符，不能有空格。

- 同一个 Key 和同一个 `Idempotency-Key`，发同样的请求，会返回原来那个任务（`200`），不会新建。
- 同一个 `Idempotency-Key` 发不同的请求，返回 `409 idempotency_conflict`。
- 提交失败需要重试时，用同一个 `Idempotency-Key`，这样不会重复建任务。

提交成功返回 `202`：

```json
{"id": "01a0de6b-...", "status": "pending", "created_at": 1790437954,
 "model": "kling-v3-text-to-video", "poll_url": "/unified/v1/tasks/01a0de6b-..."}
```

`poll_url` 是相对路径，要拼上地址。

### 查询

状态依次是 `pending` → `processing` → `completed` 或 `failed`。只有提交任务的 Key 能查这个任务。

失败时：

```json
{"status": "failed", "data": [],
 "error": {"type": "invalid_request_error", "code": "invalid_parameter",
           "message": "aspect_ratio: must be one of [\"auto\",\"16:9\",\"9:16\"]", "param": "aspect_ratio"}}
```

- 参数取值不对时，提交通常照样返回 `202`，第一次查询时才是 `failed`。
- 只有类型错误（比如 `duration` 传了字符串）会在提交时直接返回 `400`。
- 大多数模型的 `message` 只写「A request parameter was rejected as invalid or unsupported for this model」，不说是哪个参数，要对照 `video-models.md` 或 `image-models.md` 去查。
- 见过的 `code`：`invalid_parameter`、`unsupported_capability`、`input_too_large`。

其他 HTTP 状态（这张表照接口文档写，**未测**）：

| 状态 | 含义 |
|---|---|
| `401` | Key 无效 |
| `403` | 这个 Key 没有该模型的权限 |
| `404` | 模型不存在，或者查的任务不属于这个 Key |
| `429` | 这个 Key 已经有 128 个未完成任务 |
| `503` | 暂时不可用，用同一个 `Idempotency-Key` 重试 |

## 视频

请求体：

```json
{
  "model": "kling-v3-text-to-video",
  "content": [{"type": "text", "text": "海边日落，镜头缓慢推进"}],
  "duration": 5,
  "aspect_ratio": "16:9",
  "resolution": "720p",
  "provider_options": {"generate_audio": false}
}
```

- 提示词放在 `content[].text` 里，没有顶层 `prompt`。
- `duration` 必须是整数。
- 声音开关放在 `provider_options.generate_audio`，其他参数放顶层。
- 图片输入写成 `content` 里的一项：`{"type": "image_url", "role": "first_frame", "image_url": {"url": "..."}}`，`role` 可以是 `first_frame`、`last_frame`、`reference_image`。**未测。**

成功时：

```json
{"status": "completed", "data": [{"url": "https://api.creative-reasoning.com/_cr/v1/media/video/..."}],
 "created_at": 1790437954, "completed_at": 1790438071}
```

`data[0].url` 不需要 Key 就能下载，有效期未知，拿到后立即下载。

## 图像

请求体：

```json
{
  "model": "gpt-image-2",
  "prompt": "白色背景上的蓝色陶瓷杯",
  "size": "1536x1024",
  "n": 2,
  "output_format": "jpeg"
}
```

- 图像用顶层 `prompt`，和视频不一样。
- 编辑时走 `/images/edit-tasks`，加上 `image`：可以是 URL、裸 base64，或者 `data:image/png;base64,...`。
- 多张输入图（`image` 传数组）和 `mask` **未测**。
- `size` 有像素写法（`1536x1024`）和档位写法（`2K`），`aspect_ratio` 是比例（`16:9`）。三种写法不能互相代替，每个模型只认其中一部分，见 `image-models.md`。

成功时，`data` 每项是一张图，格式因模型而异：

- `{"b64_json": "iVBORw0..."}`，有的还带 `mime_type`
- `{"url": "https://..."}`

图片格式要从解出来的文件内容判断，不能只看 `mime_type`。

## 其他

- `usage` 字段的结构和单位因模型而异，不要用它估算用量。
- `callback_url`（任务结束时回调你的 HTTPS 地址）**未测**。
