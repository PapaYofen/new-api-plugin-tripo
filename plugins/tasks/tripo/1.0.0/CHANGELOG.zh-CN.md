---
changelogVersion: 1
plugin: "papayofen_tripo"
version: "1.0.0"
locale: "zh-CN"
---
# Changelog

## [1.0.0]

### Added

- 在 Tripo 的 `/v3` 路径上生成 3D、图片、深度图和高斯溅射，并继续做贴图、转格式、导入、分割、补全、减面、展 UV、绑骨和套动画。
- 用 `GET /v3/tasks/:task_id` 查一个已保存的任务，或用 `POST /v3/tasks/list` 一次查最多 100 个。没有的任务号放在 `missed` 里，查到的任务仍返回。
- 用 `POST /v3/files` 经网关上传，或用 `POST /v3/files/presign` 拿直传存储的地址。
- 任务结束后按 Tripo 的 `credits_consumed` 结算。失败、取消、封禁和过期记 0。

### Migration

- 给用户启用 `tripo` 之前，先保存「Tripo 积分单价」。没保存价格时请求会被拒绝。
- 装到接受路由字段 `missing` 和 `upstreamError` 的网关上。更早的网关会拒绝加载这个插件。
