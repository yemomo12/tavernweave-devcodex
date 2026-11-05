# TavernWeave Skills — DevCodex 格式适配

本仓库是 [LiarMTTT/TavernWeave](https://github.com/LiarMTTT/TavernWeave) 技能的**兼容性适配副本**，仅供非商业个人使用。

## 为什么有这个仓库

TavernWeave 上游 23 个技能中，有 12 个的 `SKILL.md` 使用 YAML 折叠块写 `description`：

```yaml
description: >-
  Activate, switch, combine, or close TavernWeave Soul ...
```

DevCodex（Android 端 AI Agent）的 frontmatter 解析器不支持折叠块，只取到 `>-` 三字符，导致技能简介在系统提示词里显示为 `>-`，AI 无法判断触发条件，**技能不会自动触发**。

本仓库把这 12 个文件的 `description` 压成单行（YAML 单引号包裹），其余内容一字未改：

```yaml
description: 'Activate, switch, combine, or close TavernWeave Soul ...'
```

## 改动范围

**仅 12 个 `SKILL.md` 的 frontmatter 第一行块。** 正文、references、scripts、assets 全部原样保留。

逐文件校验结果（词数比对，改前 = 改后）：

| 技能 | 词数 |
|---|---|
| activate-tavernweave-soul | 127 = 127 |
| code-quality-workflow | 54 = 54 |
| rewrite-natural-prose | 61 = 61 |
| sillytavern-card-components | 80 = 80 |
| sillytavern-card-pipeline | 117 = 117 |
| sillytavern-component-update | 60 = 60 |
| sillytavern-database-rolecards | 62 = 62 |
| sillytavern-extension-dev | 41 = 41 |
| sillytavern-media-live2d-runtime | 49 = 49 |
| sillytavern-render-regex-pipeline | 69 = 69 |
| sillytavern-rolecard-performance | 40 = 40 |
| sillytavern-rolecard-security | 59 = 59 |

每个文件均满足：删除 N 行折叠块 → 仅新增 1 行单行 description。

## 不包含

- 上游其余 11 个技能（它们的 description 本来就是单行，无需修改）
- DevCodex 内置的 android-helper / chinese-writing / prompt-writing

## 版权

原项目 Copyright 2026 LiarMTTT，采用 [PolyForm Noncommercial License 1.0.0](LICENSE)。
该许可证允许修改（Changes and New Works License）与分发（Distribution License），前提为非商业用途并保留声明。

本仓库遵循同一许可证。所有技能内容版权归原作者，本项目仅作格式适配，无逻辑与内容修改。

**如需使用请优先前往原仓库：** https://github.com/LiarMTTT/TavernWeave

## 安装（DevCodex）

设置 → 技能 Skills → 从 GitHub 导入，填写：

```
<你的用户名>/<仓库名>/tree/main/skills/<技能名>
```

或整包导入 `skills` 目录下的单个技能。建议先删除旧的同名技能再导入，避免重复。
