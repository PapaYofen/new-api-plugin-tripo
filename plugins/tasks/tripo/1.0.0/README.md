---
translations:
  zh-CN: README.zh-CN.md
---
# Tripo

Generates 3D models, images, and processed meshes with Tripo, and uploads source files.

## Channel setup

- Install this plugin, then choose the Task Plugin (61) channel type and bind `papayofen_tripo3d`.
- The key is a Tripo API key.
- Leave the API address empty to use `https://openapi.tripo3d.ai/v3`.
- To use `openapi.tripo3d.com`, set the API address to `https://openapi.tripo3d.com/v3`.
- To use another New API gateway as the upstream, choose New API (60) and enter that gateway's address and one of its tokens. That gateway needs this plugin too.
- Save one model, `tripo`, and a price for "Tripo credit unit price". A request does not reach Tripo until that price is saved.

## Models and endpoints

The channel model is `tripo`. Model names in the body, such as `v3.1`, `P1`, and `P2`, are sent to Tripo as written.

| Endpoint | What it does |
| --- | --- |
| `POST /v3/generation/text-to-model` | Creates a 3D model from text. |
| `POST /v3/generation/image-to-model` | Creates a 3D model from an image. |
| `POST /v3/generation/multiview-to-model` | Creates a 3D model from multiple views. |
| `POST /v3/generation/image-to-splat` | Creates a Gaussian splat from an image. |
| `POST /v3/generation/text-to-image` | Creates an image from text. |
| `POST /v3/generation/image-to-image` | Creates or edits an image from reference images. |
| `POST /v3/generation/image-to-multiview` | Creates multiple views from an image. |
| `POST /v3/generation/edit-multiview` | Edits one to four views. |
| `POST /v3/generation/image-to-depth` | Creates a depth image. |
| `POST /v3/models/texture` | Textures an existing model. |
| `POST /v3/models/convert` | Converts a model to another format. |
| `POST /v3/models/import` | Imports a model. |
| `POST /v3/mesh/segment` | Segments a mesh. |
| `POST /v3/mesh/smartsegment` | Runs smart segmentation. `seg_type` `model` and `image` are priced separately. |
| `POST /v3/mesh/complete` | Completes a mesh. |
| `POST /v3/mesh/decimate` | Reduces the polygon count. |
| `POST /v3/mesh/uv` | Unwraps UVs. |
| `POST /v3/animations/rig-check` | Checks whether a model can be rigged. |
| `POST /v3/animations/rig` | Rigs a model. |
| `POST /v3/animations/retarget` | Applies animations. |
| `GET /v3/tasks/:task_id` | Reads one stored task. |
| `POST /v3/tasks/list` | Reads up to 100 stored tasks. |
| `POST /v3/files` | Uploads a file through the gateway. |
| `POST /v3/files/presign` | Returns an address for uploading a large file directly to Tripo storage. |

A successful create returns a gateway task id, `task_` plus 32 characters. Later calls keep using that id. The gateway stores Tripo's id and substitutes it on the upstream request.

Single-task and batch queries read the stored task. They do not call Tripo again. The returned `task_id` is still the gateway task id.

These paths are also served under `/papayofen_tripo3d`.

## Billing

"Tripo credit unit price" is charged per Tripo credit. The gateway estimates the credits from the request, then settles on `credits_consumed` when the task finishes. A failed, cancelled, banned, or expired task settles at 0.

An omitted image model uses Tripo's default: `seedream_v4` for text to image and `seedream_v5` for image to image. An unrecognized model, size, or quality holds 50 credits. An unclear 3D request holds 130 credits. Upload and presign hold 0.

## Example

```bash
curl https://your-gateway.example.com/v3/generation/text-to-model \
  -H "Authorization: Bearer sk-..." \
  -H "Content-Type: application/json" \
  -d '{"model": "v3.1-20260211", "prompt": "a small wooden boat"}'
```

Read the task with the `task_id` from the reply:

```bash
curl https://your-gateway.example.com/v3/tasks/{task_id} \
  -H "Authorization: Bearer sk-..."
```

## Limits

- Account balance, usage, and usage history are not available. Those Tripo paths have no task id.
- A batch query accepts 1 to 100 gateway task ids. A Tripo UUID, an empty list, or more than 100 ids is rejected. A missing id is listed in `missed` and the found tasks are still returned.
- A direct upload is limited by `MAX_FILE_DOWNLOAD_MB`, 64 MB by default. Larger files use presign and are uploaded to Tripo storage by the client.
- Upload and presign answers are not stored. A later query of that task id is not found.
- `missing` and `upstreamError` have to be accepted by the gateway. A gateway that rejects those route fields cannot load this plugin.
