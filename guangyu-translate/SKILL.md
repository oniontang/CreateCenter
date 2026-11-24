---
name: guangyu-translate
description: 翻译文章、资料或文案，支持快速、普通、精细三种模式与自定义术语表。用户要求翻译、快翻、精翻、本地化、校对译文，或提供URL/文件要求翻译时使用。
metadata:
  openclaw:
    requires:
      anyBins:
      - bun
      - npx
  upstream_version: 1.59.0
---

# guangyu-translate：内容翻译

支持三种模式：**quick** 直接翻译，**normal** 先分析再翻译，**refined** 按出版质量完成分析、翻译、审查和润色。

## 用户输入工具

需要询问用户时按以下优先级执行：

1. 优先使用当前运行环境提供的内置输入工具，如 `AskUserQuestion`、`request_user_input`、`clarify`、`ask_user` 或等效工具。
2. 没有此类工具时，发送带编号的问题，让用户按编号或自由回答。
3. 工具支持同次多个问题时合并询问；仅支持单题时，按优先级逐题询问。

下文的 `AskUserQuestion` 是示例名称，其他运行环境使用等效工具。

## 脚本目录

`{baseDir}` 是本 SKILL.md 所在目录。脚本位于 `scripts/`。`${BUN_X}` 优先取 `bun`，其次 `npx -y bun`，都没有则建议安装 bun。调用时替换成实际值。

| 脚本 | 用途 |
| --- | --- |
| `scripts/main.ts` | 命令行入口，默认将Markdown分块，也支持显式 `chunk` 子命令 |
| `scripts/chunk.ts` | 分块实现，由main.ts使用，也兼容直接调用 |

## 偏好设置（EXTEND.md）

按优先级使用第一个存在的文件：

| 优先级 | 路径 | 范围 |
| --- | --- | --- |
| 1 | `.guangyu-skills/guangyu-translate/EXTEND.md` | 当前项目 |
| 2 | `${XDG_CONFIG_HOME:-$HOME/.config}/guangyu-skills/guangyu-translate/EXTEND.md` | XDG配置 |
| 3 | `$HOME/.guangyu-skills/guangyu-translate/EXTEND.md` | 用户目录 |

| 检查结果 | 动作 |
| --- | --- |
| 找到 | 读取、解析并应用。会话首次使用时简短提醒：正在使用该路径的偏好，可编辑EXTEND.md定制术语、读者等 |
| 未找到 | 必须执行首次设置，不静默采用默认值 |

支持目标语言、默认模式、目标读者、自定义术语（内嵌或外部文件）、翻译风格与分块设置。结构见 [references/config/extend-schema.md](references/config/extend-schema.md)。

### 首次设置：必须先完成

未找到 EXTEND.md 时，任何翻译开始前都必须先设置。完整步骤见 [references/config/first-time-setup.md](references/config/first-time-setup.md)。

一次调用 `AskUserQuestion` 收集目标语言、模式、读者、风格和保存位置。回答后在用户选择的位置建立 EXTEND.md，告知保存路径，再继续。

## 默认值

配置集中如下。EXTEND.md 覆盖默认值，命令行参数覆盖 EXTEND.md。

| 设置 | 默认值 | EXTEND.md字段 | 命令行参数 | 说明 |
| --- | --- | --- | --- | --- |
| 目标语言 | `zh-CN` | `target_language` | `--to` | 翻译目标语言 |
| 模式 | `normal` | `default_mode` | `--mode` | 翻译模式 |
| 读者 | `general` | `audience` | `--audience` | 目标读者 |
| 风格 | `storytelling` | `style` | `--style` | 译文语气 |
| 分块阈值 | `4000` | `chunk_threshold` | — | 触发分块的词数 |
| 块最大词数 | `5000` | `chunk_max_words` | — | 每块上限 |

## 模式

| 模式 | 参数 | 步骤 | 适用情况 |
| --- | --- | --- | --- |
| 快速 | `--mode quick` | 翻译 | 短文本、非正式内容、快速任务 |
| 普通 | `--mode normal`（默认） | 分析→翻译 | 文章、博客、通用内容 |
| 精细 | `--mode refined` | 分析→翻译→审查→润色 | 重要材料、发布级文稿 |

默认普通模式，可由 EXTEND.md 的 `default_mode` 修改。

**风格预设**独立于读者设置：

| 值 | 描述 | 表达效果 |
| --- | --- | --- |
| `storytelling` | 叙事，默认 | 叙述流畅、转折自然、措辞具体 |
| `formal` | 正式、结构清楚 | 中性、条理清晰、少用口语 |
| `technical` | 技术文档 | 精确简洁、术语集中、少修饰 |
| `literal` | 贴近原结构 | 少重组，保留原句式 |
| `academic` | 学术 | 正式严谨，可用复杂从句，注意引用 |
| `business` | 商务 | 简洁、强调行动与结果、便于管理者阅读 |
| `humorous` | 幽默 | 保留并转化原文笑点 |
| `conversational` | 日常对话 | 友好自然，像向朋友解释 |
| `elegant` | 文学性 | 注重节奏和选词，文字精炼 |

也接受自定义描述，例如 `--style "poetic and lyrical"`。

自动识别：“快翻”“quick”“直接翻译”使用quick；“精翻”“refined”“publication quality”“proofread”使用refined；其他情况使用默认normal。

普通模式完成后提醒：“译文已保存。需要进一步审查和润色，可回复‘继续润色’或‘refine’。”用户回应后，在现有结果上继续执行精细流程的审查、修订、润色（refined-workflow.md第4—6步）。

**读者预设**：

| 值 | 描述 | 表达效果 |
| --- | --- | --- |
| `general` | 普通读者，默认 | 清楚易懂，必要时增加术语解释 |
| `technical` | 开发者/工程师 | 常见技术术语少注释 |
| `academic` | 研究者/学者 | 正式表达、术语精确 |
| `business` | 商务人士 | 商务语气，解释技术背景 |

也接受自定义读者，如 `--audience "AI感兴趣的普通读者"`。

## 工作流程

### 第1步：加载偏好

1.1 按上文路径检查EXTEND.md。

1.2 如果语言对有内置术语表则读取。英译中使用 [references/glossary-en-zh.md](references/glossary-en-zh.md)。

1.3 合并 EXTEND.md 内嵌 `glossary`、外部 `glossary_files`、内置术语表及 `--glossary` 文件。外部路径相对EXTEND.md位置解析；命令行术语表覆盖其他来源。

### 第2步：保存原文并建立输出目录

文件输入保持原文件；对话文本或URL内容先保存至 `translate/{slug}.md`。建立 `{source-dir}/{source-basename}-{target-lang}/`。未指定 `--from` 时识别原文语言。细节见 [references/workflow-mechanics.md](references/workflow-mechanics.md)。

所有中间产物和最终结果放在同一输出目录：

| 文件 | 模式 | 内容 |
| --- | --- | --- |
| `translation.md` | 全部 | 最终译文，文件名固定 |
| `01-analysis.md` | 普通/精细 | 领域、语气、术语等分析 |
| `02-prompt.md` | 普通/精细 | 组装后的翻译提示词 |
| `03-draft.md` | 精细 | 审查前初稿 |
| `04-critique.md` | 精细 | 只诊断问题的审查报告 |
| `05-revision.md` | 精细 | 根据审查形成的修订稿 |
| `chunks/` | 分块时 | 原文分块与对应译文 |

### 第3步：判断内容长度

quick不分块，直接整篇翻译。翻译前估算词数；超过默认4000词时主动提醒篇幅与单次处理限制，并说明normal更适合长文术语一致性。用户未切换模式则继续。

normal/refined：低于阈值整篇处理，达到或超过阈值则分块。

**3.1 长文准备，仅normal/refined且达到阈值时执行**

1. 扫描全文，提取专有名词、技术术语和反复出现的表达。
2. 将提取术语与已加载术语表合并，建立本次统一译法。
3. 执行 `${BUN_X} {baseDir}/scripts/main.ts <file> [--max-words <chunk_max_words>] [--output-dir <output-dir>]` 分块。脚本按Markdown标题、段落、列表、代码、表格等块边界切分；单块超限则依次回退到按行、按词切分。
4. 主Agent读取存在的 `01-analysis.md`，用 [references/subagent-prompt-template.md](references/subagent-prompt-template.md) 第1部分组装共享上下文，内嵌目标风格、背景、合并术语与翻译难点。保存为 `02-prompt.md`，只放共享上下文，不放分块任务指令。
5. 有Agent工具时，每块启动一个子Agent并行起草，使用模板第2部分。每个子Agent读取 `02-prompt.md`，接收块序号、总块数及论述位置的简要背景，保存为 `chunks/chunk-NN-draft.md`。共享术语、比喻映射、理解难点和原文语气保持一致。未分块时可由一个子Agent处理全文；没有Agent工具时，主Agent依共享提示词按顺序翻译。
6. 所有分块完成后按顺序合并。有 `chunks/frontmatter.md` 时放在开头；精细模式保存 `03-draft.md`，普通模式保存 `translation.md`。
7. 保留chunks目录里的原文块及译文块。合并后交回主Agent继续审查、修订和润色。

### 第4步：翻译与精修

所有模式遵循：

- **按目标语言重新表达**：让译文像母语作者写的内容，不机械照搬句序。检验标准是读起来是否像原本就用目标语言写成。
- **准确优先**：事实、数据和逻辑必须与原文一致。
- **自然衔接**：使用目标语言习惯的顺序，长句可拆开；比喻和习语按意思处理，不逐字硬译。
- **术语统一**：采用标准译法，专业术语首次出现时括注原文。
- **保留格式**：标题、粗体、斜体、图片、链接、代码块等Markdown格式均保留。
- **必要解释**：读者缺乏背景的术语或概念，可用 `（**解释**）` 添加简短注释，只在影响理解时使用，避免过多。
- **Frontmatter**：源文有YAML时，原始元数据字段加 `source` 前缀并用camelCase，如 `url`→`sourceUrl`、`title`→`sourceTitle`。译后字段作为新顶层字段；正文已有H1时不新增title。其他字段保留。

#### 快速模式

直接翻译并保存 `translation.md`，仍遵守全部翻译原则。

#### 普通模式

1. 分析领域、语气、术语与难点，保存 `01-analysis.md`。
2. 组装含背景、术语和难点的翻译提示词，保存 `02-prompt.md`。
3. 按提示词翻译，保存 `translation.md`。

完成后提示用户可回复“继续润色”或“refine”。需要继续时，将当前译文改存为 `03-draft.md`，再形成 `04-critique.md`、`05-revision.md` 和更新后的 `translation.md`。

#### 精细模式

完整发布级流程见 [references/refined-workflow.md](references/refined-workflow.md)。分块子Agent只负责初稿；后续审查、修订、润色由主Agent处理，也可按需要委派。

1. 分析→`01-analysis.md`，记录领域、语气、术语和难点。
2. 组装提示词→`02-prompt.md`，内嵌上下文。
3. 起草→`03-draft.md`，带必要译注；分块时由分块稿合并。
4. 批判性审查→`04-critique.md`，只诊断准确性、欧化表达、策略执行和表达问题。
5. 修订→`05-revision.md`，落实审查发现。
6. 润色→`translation.md`，形成最终发布稿。

每步读取上一步文件，基于已有结果推进。

### 第5步：交付

最终译文固定为输出目录中的 `translation.md`。

写完后做一次轻量图片语言检查：收集文章图片引用，识别封面、截图、图解、图表、框架及信息图等可能含大量文字的图片。如果主要文字语言可能与译文不一致，主动提醒。提醒只列清单，不自动本地化图片，除非用户明确要求。

格式示例，沿用文章的Markdown或wikilink图片语法：

```text
Possible image localization needed:
- ![example cover](attachments/example-cover.png): likely still contains source-language text while the article is now in target language
- ![example diagram](attachments/example-diagram.png): likely text-heavy framework graphic, check whether labels need translation
```

完成摘要：

```
**Translation complete** ({mode} mode)

Source: {source-path}
Languages: {from} → {to}
Output dir: {output-dir}/
Final: {output-dir}/translation.md
Glossary terms applied: {count}
```

若存在疑似语言未同步的图片，在摘要后简短说明并列出候选。技术字段和值保留原样，向用户说明时使用用户语言。

## 扩展设置

EXTEND.md的路径与支持字段见前面的偏好设置。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
