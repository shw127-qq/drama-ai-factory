---
name: ai-drama-factory
version: 1.1.0
description: |
  AI剧工厂 — 短剧制作流水线：从一本小说或一个故事创意，到可以直接投喂视频模型（Seedance / MiniMax H3）的分镜投产包。
  7 个阶段：改编大纲 → 角色设定 → 美术设定 → 剧本 → 开场站位 → 分镜 → 角色参考图，
  每一步都有确定性脚本质量门（大纲 14 道、美术 10 道、剧本 10 道、站位 8 道、分镜 18 道），不靠模型自觉。
  可以全流程走，也可以单独调用某一个阶段；每个阶段产出 JSON + Markdown + 单页评审报告，上一步的 JSON 是下一步的输入。
  跑完拿到投产包（manifest.json + 逐段视频提示词 + 分镜图清单）后，可对接 RepaceClaw 视频生成（链接器/MCP 或 OpenAI 兼容 API）出成片。
  当用户提到"做短剧""改编小说""写剧本""写台词""拆分镜""分镜头""角色设定""人物画像""角色卡""美术设定""场景设定""道具设定""站位""走位""排座位""短剧大纲""分集梗概""出角色图""角色参考图""定妆照""AI剧工厂"时使用；
  Use when asked to make an AI short drama, adapt a novel into episodes, write a screenplay, create a storyboard, design characters or scenes, or generate character reference images.
license: Apache-2.0
metadata:
  requires:
    bins:
      - node  # >= 18，校验/报告脚本用（只用标准库，无 npm 依赖）；纯对话客户端可跳过脚本
  runtimes:
    - claude-code
    - codex
---

# AI剧工厂 · 短剧制作流水线

这是一个 7 阶段的短剧生产流水线，按顺序调用，每一步都有确定性脚本校验。

## 完整流水线

```
故事/小说原文
    ↓
① repaceclaw-outline      改编大纲（人物表、爽点表、分集梗概、资产清单）
    ↓
② repaceclaw-characters   角色设定（人物画像、形象提示词、音色提示词）
③ repaceclaw-art         美术设定（场景设计、一致性锚点、光照变体）
    ↓
④ repaceclaw-script      剧本（场次 + 节拍流，动作与台词交替，时长按语速折算）
    ↓
⑤ repaceclaw-staging     开场站位（俯视平面，谁在哪、坐站、朝向）
    ↓
⑥ repaceclaw-storyboard  分镜（段→分镜→分镜图，导出 Seedance/H3 投产包）
    ↓
repaceclaw-character-refs 角色参考图（单独用，出角色多角度参考图）
```

## 路由规则

根据用户需求，决定从哪一步开始：

| 用户说 | 调用哪个 skill |
|---|---|
| "把这个小说改成短剧""排成大纲""分集梗概" | ① repaceclaw-outline |
| "拆角色""人物设定""角色表" | ② repaceclaw-characters |
| "场景设定""美术设定""道具设定" | ③ repaceclaw-art |
| "写剧本""写台词""剧本怎么写" | ④ repaceclaw-script |
| "站位""走位""谁站哪" | ⑤ repaceclaw-staging |
| "拆分镜""分镜头""storyboard" | ⑥ repaceclaw-storyboard |
| "出角色图""角色参考图""角色多角度" | repaceclaw-character-refs |
| "做完整短剧""全流程""从小说到成片" | 从 ① 开始按顺序走完全部 6 步 |

## 调用方式

每个子 skill 都是独立目录，里面有自己的 SKILL.md。调用时：

1. 先读对应子目录的 `SKILL.md`，按它的流程执行
2. 子目录里的 `scripts/` 是确定性 Node.js 脚本，用于校验、渲染报告
3. `examples/` 里有样例数据可以参考
4. 每一步产出的 JSON 文件，是下一步的输入

## 快速上手

用户说"把这个小说改成短剧，全流程走一遍"时：

1. 让用户把小说原文（或故事梗概）放进工作目录，例如 `novel.txt`
2. 从 ① `repaceclaw-outline` 开始，读它的 `SKILL.md` 执行；产出 `outline/outline.json`
3. 依次进入 ② 角色设定 → ③ 美术设定 → ④ 剧本 → ⑤ 站位 → ⑥ 分镜，每步先读该子目录的 `SKILL.md`，把上一步的 JSON 作为种子数据
4. 跑完 ⑥ 后执行它的 export 流程，得到投产包 `manifest.json` + 每段提示词
5. 向用户确认：是只走某几步（例如"先出大纲给我看"），还是一路走完；每步产物都先给用户过目再进下一步

用户只要某一步（例如"帮我拆分镜"）时，直接路由到对应子 skill，缺上游数据就按该子 skill 的 SKILL.md 里的补全流程现场生成。

## 环境要求与降级

- 校验/报告脚本是纯 Node.js 标准库脚本（无 npm 依赖），需要 **Node.js ≥ 18**；先 `node --version` 确认
- 没有可用 Node 时降级执行：跳过脚本校验与报告渲染，按各子 skill `SKILL.md` 的流程直接产出 JSON 与 Markdown，并在交付时说明"质量门未运行"
- 技能本身不出图、不联网、不用 API Key——它交付的是提示词与结构化数据；真正出视频在下游（见下一节）

## 与 RepaceClaw 出片（下游）

拿到投产包后，两种方式把分镜变成成片：

1. **链接器（MCP）**：客户端接入 `https://api.repaceclaw.com/mcp` 后，读 `manifest.json` 逐段调用 `generate_video`（模型用 `seedance-2.5` 或 `minimax-h3`），再用 `query_task` 轮询取成片地址
2. **OpenAI 兼容 API**：脚本化批量 `POST /v1/videos`，参数与响应见 [RepaceClaw 接入指南](https://api.repaceclaw.com/about)

## 常见问题

- **技能没被触发？** 让用户说出关键词（"做短剧""拆分镜"等），或直接让它读本目录 `SKILL.md`
- **报告打不开？** 报告是单页 HTML，双击即可；打不开就交付同目录的 JSON + Markdown
- **脚本跑不过？** 质量门报错信息会写明哪一项不达标，按提示修正 JSON 后重跑脚本即可
- **只想要角色图？** 直接用 `repaceclaw-character-refs`，它不依赖小说原文
