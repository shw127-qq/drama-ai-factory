# AI 剧工厂（ai-drama-factory）

从一本小说或一个故事创意，到可以直接投喂视频模型的短剧分镜投产包。
7 个阶段的短剧生产流水线，5 个阶段带确定性脚本质量门（大纲 14 道、美术 10 道、剧本 10 道、站位 8 道、分镜 18 道，共 60 道），零依赖、零 API Key。

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
直接对接 [RepaceClaw](https://api.repaceclaw.com) 的视频生成 API（OpenAI 兼容）与链接器（MCP）出成片。

## 它能做什么

- **小说改编短剧**：分集梗概、人物表、爽点表、资产清单一次生成，爽点间隔、每集钩子由脚本卡着
- **角色与美术设定**：人物画像、三视图版面指令、音色提示词；场景一致性锚点、光照时段变体、道具状态变体
- **剧本与站位**：逐句台词带说话人与语气、时长按语速确定性折算；每场俯视站位平面（谁在哪、坐站、朝向）
- **分镜投产包**：段（≤15 秒一次生成）→ 分镜（2–5 秒切镜）→ 分镜图三层结构，自动生成对齐指令完整的视频提示词
- **角色参考图**：先出一张正面全身锚点，其余视图只参考锚点，跨图长得一样；每张可单独重出，重出后自动标过期
- 每一步产出 JSON + Markdown + 单页评审报告（双击即开），质量门全部脚本确定性检查，不靠模型自觉

## 安装

**方式一：一行安装**（Claude Code / Trae / Cursor / Cline / Codex 等支持 Agent Skills 的客户端）

```bash
npx skills add shw127-qq/drama-ai-factory
```

更新：重新执行上面这条命令即可覆盖为最新版。

**方式二：手动下载**（豆包等客户端）

从 [RepaceClaw 接入指南](https://api.repaceclaw.com/about) 获取下载链接（`ai-drama-factory.zip`），
解压后把 `ai-drama-factory` 目录完整放进客户端的技能目录即可。

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

整包安装后直接说"**做完整短剧**"，AI 会按路由规则自动串联；也可以只要某一步（例如"帮我拆分镜"），缺上游数据时该阶段会现场补全。

## 各阶段输入与产出

| 阶段 | 输入 | 产出 | 质量门 |
|---|---|---|---|
| ① repaceclaw-outline | 小说原文 | `outline.json` + Markdown + 评审报告 | 14 道 |
| ② repaceclaw-characters | 小说/大纲 | `characters.json` + Markdown + report.html | 脚本校验 |
| ③ repaceclaw-art | outline.json | `art.json` + Markdown + 评审报告 | 10 道 |
| ④ repaceclaw-script | outline.json（可选 art.json） | `script.json` + Markdown + 评审报告 | 10 道 |
| ⑤ repaceclaw-staging | script.json + art.json | `staging.json` + 可拖动调整的 HTML 站位工具 | 8 道 |
| ⑥ repaceclaw-storyboard | script.json + staging.json | `storyboard.json` + 投产包 `manifest.json` | 18 道 |
| repaceclaw-character-refs | 一段角色描述 | `asset.json` + 一组 PNG 参考图 + 报告 | — |

## 环境要求

- 支持 Agent Skills 的 AI 客户端（Claude Code、Trae、Cursor、Cline、Codex 等；豆包用手动安装）
- 校验/报告脚本需要 **Node.js ≥ 18**（只用标准库，无 npm 依赖）；纯对话客户端可跳过脚本，按各阶段 SKILL.md 流程执行
- 技能本身不出图、不联网、不用 API Key——交付的是提示词与结构化数据；真正出片在下游

## 与 RepaceClaw 链接器配合出片

流水线跑完拿到投产包后，两种方式出片：

1. **链接器（MCP）**：在 Trae / 豆包 / Cherry Studio 里接入 `https://api.repaceclaw.com/mcp`，让 AI 读取投产包逐段调用 `generate_video`，再用 `query_task` 轮询取成片地址
2. **OpenAI 兼容 API**：脚本化批量提交 `POST /v1/videos`，模型选 `seedance-2.5`（Seedance 提示词）或 `minimax-h3`（H3 提示词）

接入说明见 [RepaceClaw 接入指南](https://api.repaceclaw.com/about)。

## 常见问题

**装完没反应？** 说出关键词（"做短剧""拆分镜"），或直接让 AI 读 `SKILL.md`。

**没有 Node.js？** 技能仍可用：跳过脚本校验与报告，按各阶段 SKILL.md 的流程直接产出；建议装 Node ≥ 18 以获得质量门与评审报告。

**需要 API Key 吗？** 技能本身不需要。只有最后"出片"一步用到 RepaceClaw 的视频生成服务（注册即送体验额度）。

**报告打不开？** 评审报告是单页 HTML，双击即可；打不开就看同目录的 JSON + Markdown。

**只想出角色参考图？** 直接说"给我的角色出参考图"，`repaceclaw-character-refs` 不依赖小说原文，单独可用。

## 目录结构

```
ai-drama-factory/
├── SKILL.md                     ← 路由入口：判断走哪个阶段
├── repaceclaw-outline/          ← 每个阶段一个独立技能
│   ├── SKILL.md                 ← 该阶段的完整流程说明
│   ├── scripts/                 ← 确定性 Node.js 校验/报告脚本
│   └── examples/                ← 样例数据
├── repaceclaw-characters/
├── repaceclaw-art/
├── repaceclaw-script/
├── repaceclaw-staging/
├── repaceclaw-storyboard/
└── repaceclaw-character-refs/
```

## License

Apache-2.0
