# AutoReel

> 输入一句主题，自动产出可直接发布的 20s 竖屏短视频：分镜 → 素材生成 → 配乐 → 合成 → 质检，内置失败重试、断点续跑与成本核算。

An AI agent that turns a one-line idea into a publish-ready 20s vertical short video — storyboard, asset generation, music, render and QC, with retry, resume and cost tracking built in.

---

## 为什么要做这个

AIGC 短视频的生产链路本身并不复杂——真正的难点在**可靠性**：

- 生成接口不稳定，而且有些失败是**稳定失败**（敏感词）而不是随机失败，盲重试没有意义
- 多镜头并发生成需要限流，还要算得清成本
- 跑了 20 分钟崩在第 6 个镜头，不能从头再来
- 生成结果好坏需要自动判断，不能全靠人挑

AutoReel 把这些从「人工兜底」变成「系统能力」。

## 架构

```
                 ┌──────────────────────────────────────┐
   一句主题  ──▶  │  服务层  Spring Boot                  │
                 │  REST + SSE 进度 · 任务队列 · 持久化   │
                 └──────────────────┬───────────────────┘
                                    │
                 ┌──────────────────▼───────────────────┐
                 │  编排层  DeepSeek Harness 插件         │
                 │  工作流 · 工具注册 · 生命周期钩子       │
                 │  状态注入 · 重试 / 降级 / 断点续跑      │
                 └──────────────────┬───────────────────┘
                                    │  MCP
                 ┌──────────────────▼───────────────────┐
                 │  工具层  Python (MCP Server)          │
                 │  分镜 · 生图 · 生视频 · 配乐 · 合成     │
                 └──────────────────────────────────────┘
```

| 层 | 语言 | 职责 |
|---|---|---|
| 工具层 | Python (MCP Server) | 分镜拆解、图像/视频生成、配乐、FFmpeg 合成 |
| 编排层 | TypeScript (DeepSeek Harness 插件) | 工作流编排、工具注册、生命周期钩子、状态注入 |
| 服务层 | Java (Spring Boot) | REST + SSE 流式进度、任务队列、会话持久化 |

> 详细设计见 [docs/architecture.md](./docs/architecture.md)，接口草案见 [docs/api.md](./docs/api.md)。

## Roadmap

- [ ] **W1** 工具层打通 — 分镜 / 生图 / 合成三个工具可用，端到端产出第一条成片
- [ ] **W2** 编排与可靠性 — 并发、指数退避重试、模型降级、断点续跑、成本核算
- [ ] **W3** 服务化与评测 — REST + SSE、任务队列、20 条 case 评测集
- [ ] **W4+** 音频分析 MCP（和弦 / 节拍 / 音准）+ 配乐自动匹配

## 技术栈

Python · MCP · DeepSeek Harness (Cordis plugin) · Spring Boot · FFmpeg · 即梦 API · Suno

## 状态

🚧 开发中 · 2026.10 –

## 开发日志

每日进展、踩坑与取舍记录见 [DEVLOG.md](./DEVLOG.md)。

## License

[MIT](./LICENSE)
