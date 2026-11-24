# Guangyu内容工具集

本目录提供29个独立Skill，覆盖内容选题、文稿、标题、图像与视频辅助任务。每个模块位于同名文件夹，主入口为SKILL.md，脚本和参考资料随文件夹保留。

## 使用方式

在支持Skill的Agent中安装需要的完整模块目录，再通过该应用支持的名称调用方式使用。例如：

```text
$guangyu-grill 帮我逐个问清楚这个创作计划。
$guangyu-content 把这个已确定选题组织成内容方案。
$guangyu-title-cover-intro 根据原稿生成标题、封面文字和开头。
$guangyu-video-perception 分析这条成片的画面与声音。
```

安装或分发时，将根目录的《来源与许可》和LICENSE-third-party.txt与模块文件夹同级保留，以保留署名、许可和模块中的来源链接。

不同应用的调用语法可能不同。先确认模块已被识别；本包没有自动调度总入口，不需要输入单独的guangyu命令。

## 完整目录

### 定位、选题与研究

| 模块 | 用途 |
| --- | --- |
| [guangyu-benchmark](guangyu-benchmark/SKILL.md) | 用五重过滤法寻找值得模仿的对标，并排除主体差异造成的噪音 |
| [guangyu-jtbd](guangyu-jtbd/SKILL.md) | 用 Jobs to Be Done 识别具体情境中用户想推进的进展、切换方案的力量和选择标准，解释产品、服务或内容为何被选择 |
| [guangyu-content-value](guangyu-content-value/SKILL.md) | 评估发布前或低流量内容会吸引谁、开头能否留人，以及流量和商业价值 |
| [guangyu-standard-answer](guangyu-standard-answer/SKILL.md) | 围绕复杂问题先挖掘相关行业、学科、作者与可信理论，再比较结构同构的历史成功、失败和反例，提炼带条件的标准答案 |
| [guangyu-theory-grounding](guangyu-theory-grounding/SKILL.md) | 理论溯源与案例重释 |

### 文稿、口播与标题

| 模块 | 用途 |
| --- | --- |
| [guangyu-content](guangyu-content/SKILL.md) | 围绕已经确定的选题诊断内容怎么组织、论证和表达，形成创作方案 |
| [guangyu-resonate](guangyu-resonate/SKILL.md) | 诊断已经写成的稿件哪里没有打中目标受众，结合原句解释共鸣断点并提出具体删改建议 |
| [guangyu-script-flow](guangyu-script-flow/SKILL.md) | 检查短视频逐字稿的段落衔接、信息密度和口播流畅度，定位观众可能划走的位置 |
| [guangyu-ai-check](guangyu-ai-check/SKILL.md) | 扫描文案中的 AI 写作特征并输出检测报告，默认只诊断不改写；用户看完报告说「我想改」后，逐条追问写法背后的意图并给出修改方向 |
| [guangyu-plain-writing](guangyu-plain-writing/SKILL.md) | 按事实与功能顺序直接撰写、改写或检查产品、工具、功能、项目和更新说明，内置 ASD-STE100 清晰表达原则，核对主体、动作、指代、术语、条件与承诺强度，保留原意和用户锁定的文字，不加营销钩子、痛点渲染和抽象包装 |
| [guangyu-hook](guangyu-hook/SKILL.md) | Guangyu短视频开头优化 |
| [guangyu-title-cover-intro](guangyu-title-cover-intro/SKILL.md) | 为短视频、图文和文章生成或修改文字标题、封面文字和视频开头 |
| [guangyu-xhs-title](guangyu-xhs-title/SKILL.md) | 小红书标题公式工具 |
| [guangyu-grill](guangyu-grill/SKILL.md) | 围绕用户的计划或设计持续深入访谈，直到双方形成共同理解，并逐一厘清决策树的各个分支 |

### 图像、图文与翻译

| 模块 | 用途 |
| --- | --- |
| [guangyu-image-gen](guangyu-image-gen/SKILL.md) | 通过OpenAI、Azure OpenAI、Google、OpenRouter、DashScope、Z.AI、MiniMax、即梦、Seedream、Replicate及Agnes生成图片，支持参考图、比例和批量任务 |
| [guangyu-xhs-images](guangyu-xhs-images/SKILL.md) | 把内容拆成1—10张适合社交平台的图文卡片，支持12种视觉风格、8种布局与3种配色 |
| [guangyu-diagram](guangyu-diagram/SKILL.md) | 生成深色主题的独立SVG图解，支持架构、流程、时序、结构、思维导图、时间线、概念关系、状态机和数据流 |
| [guangyu-compress-image](guangyu-compress-image/SKILL.md) | 压缩图片并自动选择工具，默认输出WebP，也支持PNG与JPEG |
| [guangyu-translate](guangyu-translate/SKILL.md) | 翻译文章、资料或文案，支持快速、普通、精细三种模式与自定义术语表 |

### 视频、内容资产与发布准备

| 模块 | 用途 |
| --- | --- |
| [guangyu-video-perception](guangyu-video-perception/SKILL.md) | 当用户提供视频文件（.mp4、.mov、.avi、.mkv、.webm）、YouTube链接，要求观看、分析或审查视频，或在对话中引用视频内容时使用 |
| [guangyu-video-extract](guangyu-video-extract/SKILL.md) | 从抖音、小红书、微信视频号等平台链接或分享文案查询作品／账号数据并提取语音文字稿，按作者和标题归档为 Markdown |
| [guangyu-video-navigation](guangyu-video-navigation/SKILL.md) | 根据本地视频或带时间戳字幕划分章节、话题或步骤，生成可导入剪辑软件的导航条 MP4、动态分段进度和时间表 |
| [guangyu-content-system](guangyu-content-system/SKILL.md) | 把本地文稿、推文、选题、案例和课程稿整理成可持续生长、可重组复用的内容工程 |
| [guangyu-spread](guangyu-spread/SKILL.md) | 解释一段已有内容为什么引发共鸣、情绪或分享，识别传播机制、有效立场和受众动机，并提炼讨论方向 |
| [guangyu-content-risk-check](guangyu-content-risk-check/SKILL.md) | 面向创作者的内容发布风险检查 |
| [guangyu-wechat-html](guangyu-wechat-html/SKILL.md) | 把 Markdown 转成可粘贴到微信公众号后台的 HTML，并提供 15 种内置风格 |

### 存档、续接与反馈

| 模块 | 用途 |
| --- | --- |
| [guangyu-save](guangyu-save/SKILL.md) | 保存材料卡、稿件版本、关键判断、用户取舍与确认的下一步 |
| [guangyu-restore](guangyu-restore/SKILL.md) | 恢复当前或指定内容项目，核对版本，等用户确认后继续 |
| [guangyu-content-review](guangyu-content-review/SKILL.md) | 根据真实发布数据与评论形成复盘卡和下一条实验；无反馈时只给观察计划 |

课堂最小闭环与完整闭环见 [内容创作闭环总览](内容创作闭环.html)，支持全屏（F 键）和目录跳转。完整流程包含人工拍摄、剪辑和发布，不能以Skill调用代替实际完成这些环节。

## 下一步与模块边界

当前模块完成后直接结束。只有用户提出后续需求时，才根据问题提示对应的现有模块；不能确定时先问一个具体问题，等用户回答。不自动换流程、不引用未打包模块，也不要求返回导航入口。

部分模块有相邻用途：guangyu-hook专门处理开头，guangyu-xhs-title按公式处理小红书标题，guangyu-title-cover-intro联动标题、封面文字与开头；根据本次任务选择，不要求全部执行。

## 配置与依赖

- 图像与翻译偏好位于各模块说明中的 `.guangyu-skills` 或XDG配置路径。
- 视频提取凭证默认位于 `~/.config/guangyu/API_Keys.md`；配置脚本与读取脚本使用同一路径。不会读取、复制或迁移其他工具的真实凭证。
- 视频导航样式默认位于 `~/.guangyu-skills/video-navigation/`，可用 `GUANGYU_VIDEO_NAV_HOME` 覆盖；`GUANGYU_VIDEO_EXTRACT_DISABLE_KEYCHAIN`控制视频提取的钥匙串回退。
- 模型名、服务商API名称、MCP名称、API密钥变量名和宿主工具名称是实际运行接口，按原样保留。
- 不包含远程安装仓库。将所需完整guangyu模块放入当前Agent的技能目录即可；已有同名模块时先确认版本，避免覆盖个人修改。

## 创作存档位置

保存与恢复共用当前项目的 `.guangyu-skills/archive-config.json`，默认存档根目录为 `~/.guangyu-skills/archive/`，可在明确指定后切换到项目内或自定义位置。详情见两个模块的同版存档契约，不读取旧工具的存档目录。

## 内容来源

案例、经验、数字与引语属于参考材料，不表示Guangyu或使用者的亲历。不凭空补写来源作者的原话，也不把未经验证的经验判断写成统计事实。来源署名与适用许可集中见 [来源与许可](来源与许可.md)。

历史上游原文、制作对照及清理前备份保存在工作区的skill审阅目录，不作为技能执行材料。当前文件已做适配，不再声称仅修改前缀。

## 检查

本轮检查结果见 [检查结果](检查结果.json)。外部付费API、生图、视频转写与实际平台发布不属于本轮测试范围。本次只修改当前目录，没有同步到全局已安装版本。
