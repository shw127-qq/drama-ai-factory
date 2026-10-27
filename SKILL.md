---
name: ai-drama-factory
description: |
  AI剧工厂 — 短剧制作流水线：从一本小说或一个故事创意，到可以直接喂给 Seedance / MiniMax H3 的分镜投产包。
  包含 7 个阶段：改编大纲 → 角色设定 → 美术设定 → 剧本 → 开场站位 → 分镜 → 角色参考图。
  当用户提到"做短剧""写剧本""拆分镜""角色设定""美术设定""短剧大纲""分镜图""AI剧工厂"时使用。
  可以全流程走，也可以单独调用某一个阶段。
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

## 产物目录约定

建议按阶段建子目录存放产物：

```
项目目录/
├── outline/       ← repaceclaw-outline 产出
├── characters/    ← repaceclaw-characters 产出
├── art/           ← repaceclaw-art 产出
├── script/        ← repaceclaw-script 产出
├── staging/       ← repaceclaw-staging 产出
└── storyboard/    ← repaceclaw-storyboard 产出（含投产包）
```

## 最终输出

跑完 repaceclaw-storyboard 后，会导出：
- `manifest.json`：投产包清单
- 每个段一个文件夹，里面有 `prompt.md` + 分镜图

这个投产包可以直接喂给 Seedance 2.5 / MiniMax H3 出视频。
