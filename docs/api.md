# API 设计草案

> 状态：草稿（W3 实现）

Base URL：`http://localhost:8080/api`

## 提交任务

```http
POST /tasks
Content-Type: application/json

{
  "topic": "鸣人 vs 佐助 终末对决",
  "duration": 20,
  "aspectRatio": "9:16",
  "style": "90年代日本漫画风格",
  "shotCount": 4
}
```

```json
201 Created
{
  "taskId": "tsk_01J...",
  "status": "pending",
  "createdAt": "2026-10-__T__:__:__Z"
}
```

## 查询任务状态

```http
GET /tasks/{taskId}
```

```json
{
  "taskId": "tsk_01J...",
  "status": "running",
  "progress": { "stage": "render", "done": 2, "total": 4 },
  "shots": [
    { "index": 1, "status": "done",    "attempts": 1, "artifact": "..." },
    { "index": 2, "status": "done",    "attempts": 3, "artifact": "..." },
    { "index": 3, "status": "failed",  "attempts": 4, "error": "content_rejected" }
  ],
  "cost": { "tokens": 0, "generations": 13, "estimatedCny": 1.42 },
  "elapsedMs": 184320
}
```

`status` 取值：`pending` | `running` | `succeeded` | `partial` | `failed`

## 实时进度（SSE）

```http
GET /tasks/{taskId}/events
Accept: text/event-stream
```

```
event: stage
data: {"stage":"storyboard","status":"done"}

event: shot
data: {"index":2,"status":"retrying","attempt":2,"reason":"content_rejected"}

event: cost
data: {"generations":13,"estimatedCny":1.42}

event: done
data: {"status":"succeeded","video":"/artifacts/tsk_01J.../out.mp4"}
```

## 断点续跑

```http
POST /tasks/{taskId}/resume
```

从最后一个未完成的镜头继续，已完成的镜头不重新生成。

## 评测

```http
POST /eval/runs
GET  /eval/runs/{runId}
```

对固定 case 集跑一遍，输出成功率、平均耗时、平均成本。
