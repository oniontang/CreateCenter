---
name: guangyu-xhs-images
description: 把内容拆成1—10张适合社交平台的图文卡片，支持12种视觉风格、8种布局与3种配色。用户要求小红书图片、微信图文/贴图、图片卡片或系列信息图片时使用。
metadata:
  upstream_version: 2.0.1
---

# guangyu-xhs-images：系列图文卡片

将复杂内容拆成易读的系列图片卡片，并提供多种风格选择。

## 用户输入工具

按优先级使用当前环境内置的AskUserQuestion、request_user_input、clarify、ask_user或等效工具；没有则用带编号的文字问题。支持同次多题时合并，只有单题时按优先级逐题。下文的AskUserQuestion仅为示例名称。

## 图像生成工具

按以下顺序确定生图后端：

1. 当前请求明确指定后端，使用该后端。
2. EXTEND.md的preferred_image_backend指定且当前可用，使用该偏好。
3. 偏好是auto、未设置或指定后端不可用时自动选择：
   - 首先检查技能与工具列表。Codex中有imagegen时必须优先使用；通过运行环境实际支持的Skill调用方式传入已保存提示词正文，并按imagegen参数传比例与输出位置。原生imagegen优先于guangyu-image-gen等非原生后端，除非用户明确指定其他偏好。
   - 无原生imagegen，但PATH中有已登录的codex CLI时，优先 `guangyu-image-gen --provider codex-cli`；没有guangyu-image-gen则直接调用可用包装器。仅在选择此路径时读取 [references/codex-imagegen.md](references/codex-imagegen.md)，按文档检查运行时与参数。
   - 其他环境有原生工具，如Hermes image_generate，使用同样规则。
   - 只有一个可用非原生后端时，直接使用它。
   - 多个非原生后端且无原生工具时询问一次，可合并其他首次问题。
4. 都不可用时，说明情况并请用户选择如何继续。

**不能用SVG、HTML、Canvas或代码绘图代替位图生图。** 本模块已经确定需要位图；没有后端时不能擅自输出SVG或HTML/CSS画作代替，即使内容像图解。

**不能在生成图上用程序涂盖并重写文字。** 不用ImageMagick、Pillow、Canvas、SVG、HTML/CSS、OCR脚本等覆盖、擦除、描边或替换已生成卡片的标题、正文、标签等文字。文字错误时修正提示词重新生成，减少版面文字，或请用户选择可接受的候选。

preferred_image_backend设为ask时每次询问，不受可用后端数量影响。修改方式见文末。

**调用任何后端前，必须先将每张图完整最终提示词保存为 `prompts/NN-{type}-[slug].md`。** 提示词文件是可复现记录，也便于切换后端。imagegen、image_generate、guangyu-image-gen是具体工具示例，其他环境在同样规则下使用等效实现。

## 批量生成

当前组全部提示词已保存并检查存在后，默认批量生图：

1. 后端有原生批量/多任务接口时优先使用；每个任务保留自己的提示词、输出路径、比例、会话ID和直接参考图。
2. 没有原生批量但环境支持并行工具调用时，按generation_batch_size并发，默认4；当前请求的--batch-size或“并行4张”覆盖EXTEND.md。
3. 两者都没有时顺序生成。

先生成第1张锚点图，再让第2张及以后引用它。选中提示词未全部存在前不能开始。仅失败项重试一次，不重做成功项。不要仅为并行出图开子Agent，子Agent只用于独立提示词迭代或创意探索。

## 确认规则

默认生成前确认。显式调用本技能、给出文件路径、命中预设或EXTEND.md默认值，都只是推荐依据，不自动等于跳过确认。

用户完成第2步前不开始第3步。仅当前请求明确写--yes、“直接生成”“不用确认”“跳过确认”“按默认出图”等时跳过；跳过时在生成前的一次进度说明中写清采用的策略、风格、布局、配色、数量和后端。

## 语言

提问、进度、错误和完成说明都使用用户语言。风格名、路径、代码等技术标识保持英文。

## 参数

| 参数 | 说明 |
| --- | --- |
| `--style <name>` | 视觉风格 |
| `--layout <name>` | 信息布局 |
| `--palette <name>` | 覆盖配色：macaron、warm、neon |
| `--preset <name>` | 风格+布局+可选配色的快捷组合，提示词片段见references/style-presets.md |
| `--ref <files...>` | 用户参考图，用于第1张形成整组锚点 |
| `--batch-size <n>` | 本次并发数，优先EXTEND.md的generation_batch_size，否则4；限制1—8 |
| `--yes` | 非交互，跳过确认，使用偏好或默认值，自动采用路径A |

## 三个维度

风格控制线条、装饰与渲染；布局控制密度与信息排列；可选配色仅替换颜色，不改变渲染规则。共有12种风格、8种布局、3种配色，可自由组合。

例如 `--style notion --layout dense` 是知识卡，加 `--palette macaron` 只柔化颜色，不改变notion线条。preset是这些组合的简写。未指定palette用风格默认；部分风格有default_palette，例如sketch-notes默认macaron。

## 视觉风格（12种）

| 名称 | 特征 |
| --- | --- |
| cute，默认 | 甜美、可爱、少女感 |
| fresh | 干净、清新、自然 |
| warm | 温暖、友好、易亲近 |
| bold | 强冲击、醒目 |
| minimal | 极简、克制 |
| retro | 复古、怀旧 |
| pop | 鲜明、活力 |
| notion | 极简手绘线条、知识感 |
| chalkboard | 黑板彩色粉笔、教学感 |
| study-notes | 真实手写照片，蓝笔、红色批注与黄色荧光笔 |
| screen-print | 海报、半色调、有限配色、象征叙事 |
| sketch-notes | 手绘教学信息图、暖奶油底马卡龙色、不规则线条 |

各风格规范位于 `references/presets/<style>.md`。

## 信息布局（8种）

| 名称 | 特征 |
| --- | --- |
| sparse，默认 | 1—2点，强调视觉重点 |
| balanced | 3—4点，标准密度 |
| dense | 5—8点，知识卡密度 |
| list | 4—7项枚举或排名 |
| comparison | 并排对比 |
| flow | 3—6步流程或时间线 |
| mindmap | 中心辐射，4—8分支 |
| quadrant | 四象限或环形分区 |

布局规范见 `references/elements/canvas.md`。

## 配色覆盖

仅换颜色，保留线条和纹理等渲染方式。

| 名称 | 背景 | 分区颜色 | 强调色 | 感受 |
| --- | --- | --- | --- | --- |
| macaron | 暖奶油#F5F0E8 | 蓝#A8D8EA、淡紫#D5C6E0、薄荷#B5E5CF、桃#F8D5C4 | 珊瑚#E8655A | 柔和、教学 |
| warm | 浅桃#FFECD2 | 橙#ED8936、陶土#C05621、金#F6AD55、玫瑰#D4A09A | 赭#A0522D | 大地色、温暖 |
| neon | 深紫#1A1025 | 青#00F5FF、洋红#FF00FF、绿#39FF14、粉#FF6EC7 | 黄#FFFF00 | 高能、未来 |

见 `references/palettes/<palette>.md`。

## 快捷预设

用--preset指定，也可在第2步推荐。完整提示词片段见 `references/style-presets.md`。

### 知识与学习

| 预设 | 风格 | 布局 | 适用 |
| --- | --- | --- | --- |
| knowledge-card | notion | dense | 干货知识卡、概念科普 |
| checklist | notion | list | 清单、排行榜 |
| concept-map | notion | mindmap | 概念与知识脉络 |
| swot | notion | quadrant | SWOT、四象限 |
| tutorial | chalkboard | flow | 教程步骤 |
| classroom | chalkboard | balanced | 课堂笔记 |
| study-guide | study-notes | dense | 学习笔记、考试重点 |
| hand-drawn-edu | sketch-notes | flow | 手绘教程 |
| sketch-card | sketch-notes | dense | 手绘知识卡 |
| sketch-summary | sketch-notes | balanced | 手绘总结 |

### 生活与分享

| 预设 | 风格 | 布局 | 适用 |
| --- | --- | --- | --- |
| cute-share | cute | balanced | 日常分享、种草 |
| girly | cute | sparse | 甜美封面 |
| cozy-story | warm | balanced | 生活故事、情感 |
| product-review | fresh | comparison | 产品对比、测评 |
| nature-flow | fresh | flow | 健康流程、自然主题 |

### 提醒与观点

| 预设 | 风格 | 布局 | 适用 |
| --- | --- | --- | --- |
| warning | bold | list | 避坑、重要提醒 |
| versus | bold | comparison | 正反对比 |
| clean-quote | minimal | sparse | 金句、极简封面 |
| pro-summary | minimal | balanced | 专业总结、商务 |

### 潮流与娱乐

| 预设 | 风格 | 布局 | 适用 |
| --- | --- | --- | --- |
| retro-ranking | retro | list | 复古排行 |
| throwback | retro | balanced | 怀旧分享 |
| pop-facts | pop | list | 趣味知识 |
| hype | pop | sparse | 强冲击封面 |

### 海报与评论

| 预设 | 风格 | 布局 | 适用 |
| --- | --- | --- | --- |
| poster | screen-print | sparse | 海报封面、影评书评 |
| editorial | screen-print | balanced | 观点、文化评论 |
| cinematic | screen-print | comparison | 电影对比、戏剧张力 |

## 自动选择

按顺序匹配原文信号，首个命中行优先；均未匹配则用cute-share。

| 信号 | 风格 | 布局 | 推荐预设 |
| --- | --- | --- | --- |
| beauty、fashion、cute、girl、pink | cute | sparse/balanced | cute-share、girly |
| health、nature、fresh、organic | fresh | balanced/flow | product-review、nature-flow |
| life、story、emotion、warm | warm | balanced | cozy-story |
| warning、important、must、critical | bold | list/comparison | warning、versus |
| professional、business、elegant | minimal | sparse/balanced | clean-quote、pro-summary |
| classic、vintage、traditional | retro | balanced | throwback、retro-ranking |
| fun、exciting、wow、amazing | pop | sparse/list | hype、pop-facts |
| knowledge、concept、productivity、SaaS | notion | dense/list | knowledge-card、checklist |
| education、tutorial、learning、classroom | chalkboard | balanced/dense | tutorial、classroom |
| notes、handwritten、study guide、realistic | study-notes | dense/list/mindmap | study-guide |
| movie、poster、opinion、editorial、cinematic | screen-print | sparse/comparison | poster、editorial、cinematic |
| hand-drawn、infographic、workflow、手绘、图解 | sketch-notes | flow/balanced/dense | hand-drawn-edu、sketch-card、sketch-summary |

## 风格与布局匹配

✓✓最推荐，✓可用，✗避免。用户选择非默认组合时用来识别不合适的搭配。

| 风格 | sparse | balanced | dense | list | comparison | flow | mindmap | quadrant |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| cute | ✓✓ | ✓✓ | ✓ | ✓✓ | ✓ | ✓ | ✓ | ✓ |
| fresh | ✓✓ | ✓✓ | ✓ | ✓ | ✓ | ✓✓ | ✓ | ✓ |
| warm | ✓✓ | ✓✓ | ✓ | ✓ | ✓✓ | ✓ | ✓ | ✓ |
| bold | ✓✓ | ✓ | ✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓✓ |
| minimal | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| retro | ✓✓ | ✓✓ | ✓ | ✓✓ | ✓ | ✓ | ✓ | ✓ |
| pop | ✓✓ | ✓✓ | ✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓ |
| notion | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓✓ |
| chalkboard | ✓✓ | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓✓ | ✓ | ✓ |
| study-notes | ✗ | ✓ | ✓✓ | ✓✓ | ✓ | ✓ | ✓✓ | ✓ |
| screen-print | ✓✓ | ✓✓ | ✗ | ✓ | ✓✓ | ✓ | ✗ | ✓✓ |
| sketch-notes | ✓ | ✓✓ | ✓✓ | ✓✓ | ✓ | ✓✓ | ✓✓ | ✓ |

## 大纲策略

三种策略必须产生结构不同的大纲；工作流推荐一种，路径C则全部生成供选。

| 策略 | 核心 | 适用 | 结构 |
| --- | --- | --- | --- |
| A，故事驱动 | 个人经历为线索，先共鸣 | 测评、分享、变化过程 | 引子→问题→发现→体验→结论 |
| B，信息密集 | 价值优先、信息高效 | 教程、对比、清单 | 核心结论→信息卡→优缺点→建议 |
| C，视觉优先 | 视觉冲击、少字 | 高审美产品、生活氛围 | 主视觉→细节→生活场景→行动引导 |

## 参考图

用户参考图与“第1张作为全组锚点”机制分开处理，两者叠加。通过--ref或对话中的路径接收。

有路径则复制到 `refs/NN-ref-{slug}.{ext}`；只有粘贴图片而没有路径时，请求路径，或以文字提取风格特征作为回退。

| 用法 | 效果 |
| --- | --- |
| direct | 文件直接传后端，通常只传给第1张，以锚点向后传播 |
| style | 提取风格并加入每张提示词正文 |
| palette | 提取十六进制色值并加入每张提示词正文 |

受影响图片的提示词frontmatter记录参考：

```yaml
references:
  - ref_id: 01
    filename: 01-ref-brand.png
    usage: direct
```

生成时检查文件存在。第1张的direct参考在后端支持时直接传入，成为锚点；第2张以后只引用第1张，不再叠加用户参考以免冲突。style/palette特征写入每张提示词。

## 文件结构

```
image-cards/{topic-slug}/
├── source-{slug}.{ext}
├── analysis.md
├── outline-strategy-{a,b,c}.md    # Path C only
├── outline.md
├── prompts/NN-{type}-{slug}.md
├── NN-{type}-{slug}.png
└── refs/                          # only if --ref used
```

slug用2—4个单词、kebab-case，例如“AI工具推荐”→ai-tools-recommend；目录冲突则追加-YYYYMMDD-HHMMSS。

**备份规则贯穿全流程**：覆盖源文、大纲、提示词或图片前，先把旧文件改名为 `<name>-backup-YYYYMMDD-HHMMSS.<ext>`，保护用户修改。

## 工作流程

- [ ] 第0步：读取EXTEND.md，交互模式必须先完成。
- [ ] 第1步：分析内容，保存analysis.md。
- [ ] 第2步：智能确认，选择A/B/C路径。
- [ ] 第3步：生成图片。
- [ ] 第4步：交付报告。

### 第0步：读取偏好

按顺序取第一个命中：

| 路径 | 范围 |
| --- | --- |
| `.guangyu-skills/guangyu-xhs-images/EXTEND.md` | 当前项目 |
| `${XDG_CONFIG_HOME:-$HOME/.config}/guangyu-skills/guangyu-xhs-images/EXTEND.md` | XDG配置 |
| `$HOME/.guangyu-skills/guangyu-xhs-images/EXTEND.md` | 用户目录 |

找到后读取、解析，展示风格/布局/水印/语言摘要。未找到且为交互模式时，先按references/config/first-time-setup.md设置并保存，偏好未建立前不分析内容、不提前询问风格。未找到且--yes时跳过设置，使用内置默认：无水印、风格布局自动选、语言来自内容，不提问也不创建EXTEND.md。

支持水印、偏好风格/布局、自定义风格、语言、后端和批量大小。结构见references/config/preferences-schema.md。

### 第1步：分析内容

1. 保存原文，source.md已存在时执行备份规则。
2. 读取references/workflows/analysis-framework.md，分析内容类型、钩子潜力、受众、互动信号、视觉机会与滑动阅读顺序。
3. 识别原语言，建议图片数2—10张。
4. 按自动选择表推荐策略、风格、布局与配色。
5. 将结果写入analysis.md。

### 第2步：智能确认

依前面的确认规则，这是开始生成前的必要步骤，除非当前请求明确--yes或等效跳过。先展示推荐方案，让用户确认或修改。--yes则跳过整步，结合分析和命令行覆盖采用路径A。

提问前展示：

```
📋 内容分析
  主题：[topic] | 类型：[content_type]
  要点：[key points]
  受众：[audience]

🎨 推荐方案（自动匹配）
  策略：[A/B/C] [name]（[reason]）
  风格：[style] · 布局：[layout] · 配色：[palette or 默认] · 预设：[preset]
  图片：[N]张（封面+[N-2]内容+结尾）
  元素：[background] / [decorations] / [emphasis]
```

然后只问一个问题，提供三条路径。选项文案见references/confirmation.md。

**路径A，快速确认**：接受自动建议，按推荐策略和风格生成一个outline.md，再进入生成。

**路径B，自定义**：询问策略/风格、布局、配色、张数、可选补充五项。预填推荐，留空保持推荐；按用户选择生成outline.md，再进入生成。详见references/confirmation.md。

**路径C，详细模式**：

- 2a，内容理解：询问卖点（多选）、受众、风格偏好（真实/专业/审美/自动）及补充背景，更新analysis.md。
- 2b，三版大纲：生成outline-strategy-a.md、b.md、c.md。每版结构和推荐风格都必须不同，frontmatter包含style_reason。建议张数A约4—6、B约3—5、C约3—4。模板见references/workflows/outline-template.md，元数据例子见references/confirmation.md。
- 2c，选择：询问大纲A/B/C/合并、风格、视觉元素三项；选中或合并结果存outline.md，进入生成。

### 第3步：生成图片

基于已确认大纲、风格、布局与配色执行。

**视觉一致性：第1张锚点链。** 不设锚点时人物、吉祥物和色彩会跨调用漂移。先生成封面，再让所有后续图把第1张作为--ref。即使后端有会话ID也不跳过。没有用户direct参考时第1张不传--ref；有direct参考时按“参考图”一节传给第1张，再只向后传播第1张。

1. 用用户偏好语言把每张完整提示词保存到prompts/NN-{type}-{slug}.md，按需备份，检查本组全部文件存在。
2. 先生成第1张，生成前按PNG备份规则处理，建立锚点。
3. 第2张及以后建立任务列表，均传 `--ref <path-to-image-01.png>`。
4. 按批量规则执行：原生批量优先，其次运行环境并行，最后顺序。
5. 每完成一张报告进度，失败只从同一已保存提示词重试该项一次。

启用水印时将以下要求加入生图提示词：

```
Include a subtle watermark "[content]" positioned at [position].
The watermark should be legible but not distracting.
```

水印见references/config/watermark-guide.md。

后端按开头的选择规则确定，多后端时在生成前询问一次。--yes使用偏好，失败回退到首个可用后端。调用前必须有提示词文件。

选择codex-imagegen时，读取references/codex-imagegen.md：优先guangyu-image-gen的codex-cli路径，也包含包装器发现、参数、stdout结构与批量语义。包装器每次n=1，每张卡片独立调用；不支持--sessionId，因此一致性靠--ref链。

后端支持--sessionId时，全组使用 `cards-{topic-slug}-{timestamp}`，与参考链结合。

### 第4步：交付报告

按用户语言展示主题、快速/自定义/详细模式、策略、风格、配色、布局、保存目录与总张数，列出analysis.md、outline.md、详细模式三版大纲，以及每张图的文件名、类型、布局和实际状态。

原格式字段示例保留为：

```
Image Card Series Complete!

Topic: [topic]
Mode: [Quick / Custom / Detailed]
Strategy: [A/B/C/Combined]
Style: [name]
Palette: [name or "default"]
Layout: [name or "varies"]
Location: [directory]
Images: N total

✓ analysis.md
✓ outline.md
✓ outline-strategy-a/b/c.md (detailed mode only)

- 01-cover-[slug].png ✓ Cover (sparse)
- 02-content-[slug].png ✓ Content (balanced)
- ...
- NN-ending-[slug].png ✓ Ending (sparse)
```

## 内容拆分原则

| 位置 | 任务 | 常用布局 |
| --- | --- | --- |
| 第1张封面 | 钩子与视觉重点 | sparse |
| 中间内容 | 每张一个核心价值 | balanced/dense/list/comparison/flow |
| 最后1张 | 行动引导或总结 | sparse或balanced |

风格与布局兼容性按前面矩阵检查。

## 图片修改

| 操作 | 顺序 |
| --- | --- |
| 编辑 | 先更新对应提示词，再用相同会话ID重新生成 |
| 增加 | 指定位置、写提示词、生图、后续编号+1、更新大纲 |
| 删除 | 删除指定文件、后续编号-1、更新大纲 |

重新生成前始终先更新提示词，它是制作依据。

文字有错字、乱码、难读或表现弱时，不用代码修补位图。纠错生成使用新提示词文件和新输出路径，保留缺陷候选对照。后期处理只限不改变文字和主体构图的裁剪、缩放、压缩与格式转换。

## 参考资料

| 文件 | 内容 |
| --- | --- |
| references/confirmation.md | 各确认路径的提问文案 |
| references/style-presets.md | 完整快捷预设 |
| references/presets/<style>.md | 风格元素 |
| references/palettes/<name>.md | 配色定义 |
| references/elements/canvas.md | 比例、安全区、网格 |
| references/elements/image-effects.md | 抠图、描边、滤镜 |
| references/elements/typography.md | 装饰文字、标签、方向 |
| references/elements/decorations.md | 强调、背景、涂鸦、边框 |
| references/workflows/analysis-framework.md | 分析框架 |
| references/workflows/outline-template.md | 大纲与布局模板 |
| references/workflows/prompt-assembly.md | 提示词组装 |
| references/config/preferences-schema.md | 配置结构 |
| references/config/first-time-setup.md | 首次设置 |
| references/config/watermark-guide.md | 水印 |

## 注意事项

生成失败自动重试一次后再报错；涉及敏感公众人物时使用风格化卡通替代。智能确认不可省略，详细模式另包含2a与2c两处确认。

## 修改偏好

EXTEND.md使用第0步第一个存在的路径，三种修改方式：

- 直接编辑字段，结构见references/config/preferences-schema.md。
- 删除文件，或要求“重新配置guangyu-xhs-images偏好”，下次运行重新首次设置。
- 常用单行配置：
  - preferred_image_backend: auto，原生工具优先，只有一个非原生时回退，多个时询问。
  - preferred_image_backend: codex-imagegen，固定Codex后端。
  - preferred_image_backend: guangyu-image-gen，固定本套生图模块。
  - preferred_image_backend: ask，每次确认。
  - generation_batch_size: 4，环境支持时默认并发4。
  - preferred_style: notion、preferred_layout: dense、preferred_palette: macaron、language: zh。
  - watermark.enabled: true与watermark.content: "@handle"，启用水印。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
