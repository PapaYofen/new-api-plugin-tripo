---
locale: zh-CN
---
# Tripo

用 Tripo 生成三维模型、图片和加工后的网格，并上传素材。

## 渠道

- 安装本插件后，渠道类型选任务插件（61），并绑定 `papayofen_tripo`。
- 密钥填 Tripo 的 API key。
- API 地址留空时使用 `https://openapi.tripo3d.ai/v3`。
- 要走 `openapi.tripo3d.com`，把 API 地址填成 `https://openapi.tripo3d.com/v3`。
- 上游如果是另一台 New API，渠道类型选 New API（60），填那台网关的地址和它的一个令牌。那台网关也要装这个插件。
- 模型只保存 `tripo`，并给「Tripo 积分单价」填价格。没保存价格时，请求到不了 Tripo。

## 模型和接口

渠道里的模型名是 `tripo`。请求正文里的 `v3.1`、`P1`、`P2` 原样转给 Tripo。

| 路径 | 作用 |
| --- | --- |
| `POST /v3/generation/text-to-model` | 文字生成 3D。 |
| `POST /v3/generation/image-to-model` | 图片生成 3D。 |
| `POST /v3/generation/multiview-to-model` | 多视图生成 3D。 |
| `POST /v3/generation/image-to-splat` | 图片生成高斯溅射。 |
| `POST /v3/generation/text-to-image` | 文字生图。 |
| `POST /v3/generation/image-to-image` | 参考图生图或改图。 |
| `POST /v3/generation/image-to-multiview` | 图片生成多视图。 |
| `POST /v3/generation/edit-multiview` | 修改 1 到 4 个视图。 |
| `POST /v3/generation/image-to-depth` | 生成深度图。 |
| `POST /v3/models/texture` | 给已有模型贴图。 |
| `POST /v3/models/convert` | 转换模型格式。 |
| `POST /v3/models/import` | 导入模型。 |
| `POST /v3/mesh/segment` | 分割网格。 |
| `POST /v3/mesh/smartsegment` | 智能分割。`seg_type` 为 `model` 和 `image` 时按不同数量预扣。 |
| `POST /v3/mesh/complete` | 补全网格。 |
| `POST /v3/mesh/decimate` | 减面。 |
| `POST /v3/mesh/uv` | 展 UV。 |
| `POST /v3/animations/rig-check` | 检查能否绑骨。 |
| `POST /v3/animations/rig` | 绑骨。 |
| `POST /v3/animations/retarget` | 套动画。 |
| `GET /v3/tasks/:task_id` | 查询一个已保存的任务。 |
| `POST /v3/tasks/list` | 一次查询最多 100 个已保存的任务。 |
| `POST /v3/files` | 经网关上传文件。 |
| `POST /v3/files/presign` | 返回大文件直传 Tripo 存储的地址。 |

创建成功时返回网关任务号，形如 `task_` 加 32 位。之后的请求继续传这个号。Tripo 自己的任务号留在网关里，发给上游时再换上。

查一个任务和查多个任务都读网关里保存的结果，不会当场再问 Tripo。返回里的 `task_id` 仍是网关任务号。

这些路径也可以从 `/papayofen_tripo` 下面访问。

## 计费

「Tripo 积分单价」按 Tripo 的 credit 计费。网关先按请求估算 credit 数量，任务结束后按 `credits_consumed` 结算。失败、取消、封禁、过期记 0。

文生图没写模型时按 `seedream_v4` 估算，图生图没写模型时按 `seedream_v5` 估算。不认识的模型、尺寸或质量预扣 50 credit。看不清的 3D 请求预扣 130 credit。上传和预签名预扣 0。

## 示例

```bash
curl https://your-gateway.example.com/v3/generation/text-to-model \
  -H "Authorization: Bearer sk-..." \
  -H "Content-Type: application/json" \
  -d '{"model": "v3.1-20260211", "prompt": "a small wooden boat"}'
```

用响应里的 `task_id` 查询：

```bash
curl https://your-gateway.example.com/v3/tasks/{task_id} \
  -H "Authorization: Bearer sk-..."
```

## 限制

- 账户余额、用量和用量历史没有接。这几条 Tripo 路径没有任务号。
- 批量查询只收 1 到 100 个网关任务号。Tripo 的 UUID、空列表或超过 100 个会被拒绝。没有的任务号出现在 `missed` 里，查到的任务仍会返回。
- 经网关的直接上传受 `MAX_FILE_DOWNLOAD_MB` 限制，默认 64 MB。更大的文件走预签名，由客户端直接传到 Tripo 的存储。
- 上传和预签名的结果不保存。之后再用那条任务号去查，会返回找不到。
- 网关必须接受路由字段 `missing` 和 `upstreamError`。不认识这两个字段的网关装不上这个插件。
