---
changelogVersion: 1
plugin: "papayofen_tripo3d"
version: "1.0.0"
locale: "en"
translations:
  zh-CN: "CHANGELOG.zh-CN.md"
---
# Changelog

## [1.0.0]

### Added

- Generate 3D models, images, depth maps, and Gaussian splats on Tripo's `/v3` paths, then texture, convert, import, segment, complete, decimate, unwrap, rig, and retarget the results.
- Query one stored task with `GET /v3/tasks/:task_id`, or up to 100 stored tasks with `POST /v3/tasks/list`. A missing id is returned in `missed` while the found tasks stay in the reply.
- Upload a file through `POST /v3/files`, or request a direct-to-storage address with `POST /v3/files/presign`.
- Settle each finished task on Tripo's `credits_consumed`. Failed, cancelled, banned, and expired tasks settle at 0.

### Migration

- Save a price for "Tripo credit unit price" on the `tripo` model before enabling it. Requests are refused until that price exists.
- Install this version on a gateway that accepts the route fields `missing` and `upstreamError`. Older gateways reject the plugin.
