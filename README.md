# AI 剧工厂（ai-drama-factory）

从一本小说或一个故事创意，到可以直接投喂视频模型的短剧分镜投产包。
7 个阶段的短剧生产流水线，每一步都有确定性脚本校验（18 道质量门），零依赖、零 API Key。

```
故事/小说原文
    ↓
① repaceclaw-outline      改编大纲（人物表、爽点表、分集梗概、资产清单）
    ↓
② repaceclaw-characters   角色设定（人物画像、形象提示词、音色提示词）
③ repaceclaw-art          美术设定（场景设计、一致性锚点、光照变体）
    ↓
④ repaceclaw-script       剧本（场次 + 节拍流，动作与台词交替，时长按语速折算）
    ↓
⑤ repaceclaw-staging      开场站位（俯视平面，谁在哪、坐站、朝向）
    ↓
⑥ repaceclaw-storyboard   分镜（段→分镜→分镜图，导出 Seedance / MiniMax H3 投产包）
    ↓
repaceclaw-character-refs 角色参考图（单独用，出角色多角度参考图）
```

最终产物是**投产包**：`manifest.json` + 每段一条 Seedance / MiniMax H3 视频提示词 + 分镜图清单，
直接对接 [RepaceClaw](https://api.repaceclaw.com) 的视频生成 API（OpenAI 兼容）与链接器（MCP）。

## 安装

**方式一：skills.sh 一行安装**（Claude Code / Trae / Cursor / Cline / Codex 等）

```bash
npx skills add shw127-qq/drama-ai-factory
```

**方式二：手动下载**（豆包等客户端）

从 [RepaceClaw 官网文档](https://api.repaceclaw.com/about) 获取下载链接，解压后把
`ai-drama-factory` 目录完整放进客户端的技能目录即可。

**方式三：甩给 AI 自己装**（客户端带终端时，把下面这段粘给 AI）

```
帮我安装 AI 剧工厂技能包：
1. 下载 https://repaceclaw.com/downloads/ai-drama-factory.zip 并解压
2. 把 ai-drama-factory 目录完整放进你的技能目录
3. 装好后读一遍 SKILL.md，告诉我你能承接哪些短剧任务
```

## 使用

| 你说 | 进入哪一步 |
|---|---|
| "把这个小说改成短剧" "排成大纲" | ① repaceclaw-outline |
| "拆角色" "人物设定" | ② repaceclaw-characters |
| "场景设定" "美术设定" | ③ repaceclaw-art |
| "写剧本" "写台词" | ④ repaceclaw-script |
| "站位" "走位" | ⑤ repaceclaw-staging |
| "拆分镜" "分镜头" "storyboard" | ⑥ repaceclaw-storyboard |
| "出角色图" "角色参考图" | repaceclaw-character-refs |
| "做完整短剧" "全流程" | 从 ① 按顺序走完全部 6 步 |

也可以整包安装后直接说"做完整短剧"，由 AI 按路由规则自动串联。

## 环境要求

- 支持 Agent Skills 的 AI 客户端（Claude Code、Trae、Cursor、豆包等）
- 校验/报告脚本需要 **Node.js ≥ 18**（只用标准库，无 npm 依赖）；纯对话客户端可跳过脚本，按 SKILL.md 流程执行

## 与 RepaceClaw 链接器配合

流水线跑完拿到投产包后，两种方式出片：

1. **链接器（MCP）**：在 Trae / 豆包 / Cherry Studio 里接入 `https://api.repaceclaw.com/mcp`，让 AI 读取投产包逐段调用 `generate_video`
2. **OpenAI 兼容 API**：脚本化批量提交 `POST /v1/videos`，模型选 `seedance-2.5`（Seedance 提示词）或 `minimax-h3`（H3 提示词）

接入说明见 [RepaceClaw 接入指南](https://api.repaceclaw.com/about)。

## License

Apache-2.0
