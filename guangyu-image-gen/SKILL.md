---
name: guangyu-image-gen
description: 通过OpenAI、Azure OpenAI、Google、OpenRouter、DashScope、Z.AI、MiniMax、即梦、Seedream、Replicate及Agnes生成图片，支持参考图、比例和批量任务。用户要求生成、绘制或编辑图片时使用。
metadata:
  openclaw:
    requires:
      anyBins:
      - bun
      - npx
  upstream_version: 2.1.0
---

# guangyu-image-gen：图像生成

基于各服务官方API生成图像。支持 OpenAI GPT Image 2、Azure OpenAI、Google、OpenRouter、DashScope（通义万象）、Z.AI GLM-Image、MiniMax、即梦、Seedream（豆包）、Replicate和Agnes。

## 用户输入工具

1. 优先使用运行环境内置的 `AskUserQuestion`、`request_user_input`、`clarify`、`ask_user` 或等效输入工具。
2. 没有等效工具时用带编号的普通文字问题，让用户按编号或自由回答。
3. 支持多题时合并所有适用问题，只支持单题时按优先级逐题询问。

下文AskUserQuestion是示例名称，其他环境替换为等效工具。

## 脚本目录

`{baseDir}` 是本SKILL.md所在目录，所有scripts路径相对它解析。主入口为 `{baseDir}/scripts/main.ts`，批处理文件组装器为 `{baseDir}/scripts/build-batch.ts`。`${BUN_X}` 优先用bun，其次 `npx -y bun`，再不可用则建议 `brew install oven-sh/bun/bun`。

## 第0步：加载偏好，必须先完成

任何生图前必须完成本步骤，EXTEND.md不存在时不能开始生成。按顺序取第一个命中：

| 路径 | 范围 |
| --- | --- |
| `.guangyu-skills/guangyu-image-gen/EXTEND.md` | 当前项目 |
| `${XDG_CONFIG_HOME:-$HOME/.config}/guangyu-skills/guangyu-image-gen/EXTEND.md` | XDG配置 |
| `$HOME/.guangyu-skills/guangyu-image-gen/EXTEND.md` | 用户目录 |

找到后加载、解析并应用；若 `default_model.[provider]` 为null，仅询问模型。未找到时按 `references/config/first-time-setup.md` 收集服务、模型、质量、保存位置，保存EXTEND.md后继续，未完成不能生图。

兼容旧目录：存在 `.guangyu-skills/guangyu-imagine/EXTEND.md` 而新路径不存在时，运行时将旧目录改名为guangyu-image-gen；两者同时存在时保留旧目录，使用新目录。

支持默认服务、质量、比例、图像尺寸、OpenAI API方言、默认模型、批处理并发上限和服务限制。结构见 `references/config/preferences-schema.md`。

## 使用方式

完整的各服务和批处理示例见 `references/usage-examples.md`。

### 保持参考图身份的提示词

用户要求保留真实人物、角色或物体时，不用冗长通用特征描述替代参考图，而用简短明确的身份约束：

- 使用参考图中的同一人物/物体，不重新设计，不创造长得相似的新主体。
- 只改变场景、衣服、姿态、灯光、渲染风格与构图；保留脸、比例、头发、关键配件和整体身份。
- 多张参考图时说明它们是同一主体，共同定义身份。

“年轻东亚女性、椭圆脸、清澈眼睛”等泛化长描述可能让模型合成相似新人，而不是保留原主体。

```bash
# Basic
${BUN_X} {baseDir}/scripts/main.ts --prompt "A cat" --image cat.png

# With aspect ratio and high quality
${BUN_X} {baseDir}/scripts/main.ts --prompt "A landscape" --image out.png --ar 16:9 --quality 2k

# Prompt from files
${BUN_X} {baseDir}/scripts/main.ts --promptfiles system.md content.md --image out.png

# With reference image
${BUN_X} {baseDir}/scripts/main.ts --prompt "Make blue" --image out.png --ref source.png

# Specific provider
${BUN_X} {baseDir}/scripts/main.ts --prompt "A cat" --image out.png --provider dashscope --model qwen-image-2.0-pro

# OpenAI GPT Image 2
${BUN_X} {baseDir}/scripts/main.ts --prompt "A cat" --image out.png --provider openai --model gpt-image-2

# Codex CLI (uses logged-in Codex subscription — no OPENAI_API_KEY required; requires `codex` on PATH)
${BUN_X} {baseDir}/scripts/main.ts --prompt "A cat" --image out.png --provider codex-cli --ar 16:9

# Batch mode
${BUN_X} {baseDir}/scripts/main.ts --batchfile batch.json --jobs 4

# Build a batch file from outline.md + prompts/ (e.g. an existing illustration outline and saved prompts)
${BUN_X} {baseDir}/scripts/build-batch.ts --outline outline.md --prompts prompts --output batch.json --images-dir attachments
${BUN_X} {baseDir}/scripts/main.ts --batchfile batch.json --jobs 4
```

以上包含基础生图、比例/质量、文件提示词、参考图、指定服务、GPT Image 2、Codex订阅路径和批量任务。

## 参考图身份保持

- 优先选2—4张已有原始参考，不用过多大文件，以免流式服务不稳定。
- 明确所有参考是同一主体，输出必须保持该身份，避免长篇通用面部描述。
- 用户未明确要求时，不将新生成的结果继续当参考，避免累积漂移。
- 结果过于精修或网红化时，减少风格参考，并明确不瘦脸、不放大眼睛、不浓妆、不做商业旅拍、不磨皮过度。
- 需要更年轻或年长的感觉时保留脸，通过服装、姿态、场景和造型表达年龄，不要求改变面部身份。

## 参数

| 参数 | 说明 |
| --- | --- |
| `--prompt <text>`、`-p` | 提示词 |
| `--promptfiles <files...>` | 拼接读取提示词文件 |
| `--image <path>` | 单图模式必需的输出路径 |
| `--batchfile <path>` | 多图任务JSON |
| `--jobs <count>` | 批量worker数；默认自动、受配置上限限制，内置上限10 |
| `--provider google\|openai\|azure\|openrouter\|dashscope\|zai\|minimax\|jimeng\|seedream\|replicate\|codex-cli\|agnes` | 指定服务；默认自动。codex-cli永不自动选中，需参数或偏好指定 |
| `--model <id>`、`-m` | 模型ID，默认及合法值见服务参考 |
| `--ar <ratio>` | 比例，如16:9、1:1、4:3 |
| `--size <WxH>` | 明确尺寸，如1024x1024；gpt-image-2宽高为16倍数，最大边3840px，比例不超过3:1 |
| `--quality normal\|2k` | 质量预设，默认2k |
| `--imageSize 1K\|2K\|4K` | Google/OpenRouter尺寸，默认由quality决定 |
| `--imageApiDialect openai-native\|ratio-metadata` | OpenAI兼容端点方言；需要比例size和metadata.resolution的网关使用ratio-metadata |
| `--ref <files...>` | 参考图；支持Google多模态、OpenAI编辑、Azure编辑（只PNG/JPG）、OpenRouter多模态、Replicate支持族、MiniMax主体参考、Seedream 5.0/4.5/4.0、DashScope wan2.7-image-pro/wan2.7-image。不支持即梦、Seedream 3.0、SeedEdit 3.0及wan2.7-image族以外的DashScope模型 |
| `--n <count>` | 图片数；Replicate单输出保存语义要求n=1 |
| `--json` | JSON输出 |

## 环境变量

| 变量 | 用途 |
| --- | --- |
| `OPENAI_API_KEY` | OpenAI密钥 |
| `AZURE_OPENAI_API_KEY` | Azure OpenAI密钥 |
| `OPENROUTER_API_KEY` | OpenRouter密钥 |
| `GOOGLE_API_KEY` | Google密钥 |
| `DASHSCOPE_API_KEY` | DashScope密钥 |
| `ZAI_API_KEY`，别名 `BIGMODEL_API_KEY` | Z.AI密钥 |
| `MINIMAX_API_KEY` | MiniMax密钥 |
| `REPLICATE_API_TOKEN` | Replicate令牌 |
| `JIMENG_ACCESS_KEY_ID`、`JIMENG_SECRET_ACCESS_KEY` | 即梦火山引擎凭证 |
| `ARK_API_KEY` | Seedream火山方舟密钥 |
| `<PROVIDER>_IMAGE_MODEL` | 各服务模型覆盖：OPENAI、GOOGLE、DASHSCOPE、ZAI/BIGMODEL、MINIMAX、OPENROUTER、REPLICATE、JIMENG、SEEDREAM、AGNES对应前缀 |
| `AZURE_OPENAI_DEPLOYMENT`，别名 `AZURE_OPENAI_IMAGE_MODEL` | Azure默认部署 |
| `<PROVIDER>_BASE_URL` | 服务端点覆盖 |
| `AZURE_API_VERSION` | Azure版本，默认2025-04-01-preview |
| `JIMENG_REGION` | 即梦区域，默认cn-north-1 |
| `OPENAI_IMAGE_API_DIALECT` | openai-native或ratio-metadata |
| `OPENROUTER_HTTP_REFERER`、`OPENROUTER_TITLE` | OpenRouter可选归属信息 |
| `GUANGYU_IMAGE_GEN_MAX_WORKERS` | 批处理并发上限 |
| `GUANGYU_IMAGE_GEN_<PROVIDER>_CONCURRENCY` | 单服务并发；例如GUANGYU_IMAGE_GEN_REPLICATE_CONCURRENCY，Codex使用GUANGYU_IMAGE_GEN_CODEX_CLI_CONCURRENCY |
| `GUANGYU_IMAGE_GEN_<PROVIDER>_START_INTERVAL_MS` | 单服务启动间隔 |
| `GUANGYU_CODEX_IMAGEGEN_BIN` | codex-cli包装器路径，默认scripts/codex-imagegen/main.ts，接受.ts或旧.sh/二进制 |
| `GUANGYU_CODEX_IMAGEGEN_CACHE_DIR` | codex-cli幂等缓存，默认关闭 |
| `GUANGYU_CODEX_IMAGEGEN_TIMEOUT_MS` | 每次codex exec超时，默认300000ms |
| `GUANGYU_CODEX_IMAGEGEN_RETRIES` | 可重试错误的包装器重试次数，默认2 |
| `GUANGYU_CODEX_IMAGEGEN_LOG_FILE` | codex-cli JSONL诊断日志 |

加载优先级：命令行 > EXTEND.md > 环境变量 > `<cwd>/.guangyu-skills/.env` > `~/.guangyu-skills/.env`。

### Codex/ChatGPT登录不等于OpenAI API密钥

`--provider openai --model gpt-image-2` 走标准Images API的 `/v1/images/generations` 或 `/v1/images/edits`，需要OPENAI_API_KEY。Codex/ChatGPT桌面登录属于不同授权，不能将OAuth令牌填入OPENAI_API_KEY，也不能只改OPENAI_BASE_URL指向Codex后端。

用户想用Codex订阅而没有API密钥时：

- Codex运行时使用原生imagegen工具/技能。
- 其他环境若codex CLI已登录，用 `guangyu-image-gen --provider codex-cli`，复用统一重试、缓存与批处理。服务调用随包scripts/codex-imagegen/main.ts；独立调用方直接使用本模块内的 `scripts/codex-imagegen/main.ts`。
- Hermes有image_generate时可以作为回退，并说明参考图是直接传入还是只提取了文字特征。

不能偷偷修改openai服务去使用Codex OAuth。codex-cli拥有独立登录、codex exec路由、请求格式与测试，见 `references/codex-oauth-vs-openai-api-key.md`。

## 模型选择

所有服务的优先级：命令行 `--model` > EXTEND.md的 `default_model.[provider]` > `<PROVIDER>_IMAGE_MODEL` > 内置默认。

OpenAI默认gpt-image-2，也可用参数或环境变量选择gpt-image-1.5、gpt-image-1和快照。Azure中的model是部署名称，优先环境变量AZURE_OPENAI_DEPLOYMENT，旧AZURE_OPENAI_IMAGE_MODEL仍兼容；部署以模型命名才填gpt-image-2，否则填真实自定义名。

EXTEND.md覆盖环境变量，例如google默认模型设为gemini-3-pro-image时，即使GOOGLE_IMAGE_MODEL为gemini-3.1-flash-image仍以前者为准。

每次生成前展示所用provider/model及切换方式：`--model <id>`、EXTEND.md或对应模型环境变量。

## OpenAI兼容网关方言

provider=openai只说明认证和路由兼容，不保证上游语义与OpenAI原生相同。通过EXTEND.md的default_image_api_dialect、OPENAI_IMAGE_API_DIALECT或--imageApiDialect选择：

- openai-native：像素size，如1536x1024，使用OpenAI原生质量字段。
- ratio-metadata：比例size，如16:9，加metadata.resolution（1K/2K/4K）和metadata.orientation。

原生API或严格兼容端点用openai-native；Gemini等网关可尝试ratio-metadata。后者目前仅文生图，参考图编辑仍需openai-native或原生支持编辑的服务。

## 各服务指南

选择相应服务或非默认行为时读取对应文件：

| 服务 | 参考 |
| --- | --- |
| DashScope：Qwen-Image、尺寸 | `references/providers/dashscope.md` |
| Z.AI：GLM-Image、cogview-4 | `references/providers/zai.md` |
| MiniMax：image-01、主体参考 | `references/providers/minimax.md` |
| OpenRouter：多模态、chat/completions | `references/providers/openrouter.md` |
| Replicate：nano-banana、Seedream、Wan | `references/providers/replicate.md` |
| Codex CLI：包装随包codex-imagegen、用登录而非API密钥 | `references/providers/codex-cli.md` |
| Agnes：agnes-image-2.1-flash、参考图 | `references/providers/agnes.md` |

## 服务选择

1. 有ref、未指定provider时依次自动选择Google→OpenAI→Azure→OpenRouter→Replicate→Seedream→MiniMax→Agnes。MiniMax主体参考更偏人物一致性。
2. 指定provider则使用它；有ref时provider须为google/openai/azure/openrouter/replicate/seedream/minimax/codex-cli/agnes。
3. 仅存在一个API密钥时使用对应服务。
4. 多个密钥时默认顺序Google→OpenAI→Azure→OpenRouter→DashScope→Z.AI→MiniMax→Replicate→即梦→Seedream→Agnes。
5. codex-cli永不自动选择，需default_provider或--provider指定。通过bun运行随包scripts/codex-imagegen/main.ts并启动codex exec，使用Codex订阅；要求PATH有codex且已codex login，不需要OPENAI_API_KEY。

## 质量预设

| 预设 | Google imageSize | OpenAI | OpenRouter | Replicate | 用途 |
| --- | --- | --- | --- | --- | --- |
| normal | 1K | 目标1024px | 1K | 1K | 快速预览 |
| 2k，默认 | 2K | 目标2048px | 2K | 2K | 封面、插画、信息图 |

Google/OpenRouter可用--imageSize 1K/2K/4K覆盖。OpenAI原生gpt-image-2中，normal对应medium与靠近请求比例的低延迟合法尺寸，2k对应high与2048级尺寸，如2048x2048、2048x1152、1152x2048。自定义或4K用合法--size，例如3840x2160。

## 比例

支持1:1、16:9、9:16、4:3、3:4、2.35:1。

- Google多模态用imageConfig.aspectRatio。
- OpenAI gpt-image-2选最接近的合法自定义尺寸，旧GPT Image和DALL·E选最接近的固定尺寸。
- OpenRouter用imageGenerationOptions.aspect_ratio；只给size时推断比例。
- Replicate按模型处理：google/nano-banana*用aspect_ratio，bytedance/seedream-*按文档比例，Wan 2.7将ar映射为具体size。
- MiniMax使用官方aspect_ratio；image-01只有size而无ar时发送width/height。

## 生成模式

默认顺序执行；batchfile有两个以上待完成任务时自动启用批量并行。

| 场景 | 建议 | 原因 |
| --- | --- | --- |
| 一张或1—2张简单图片 | 顺序 | 协调少、方便排错 |
| 多图且提示词已经落盘 | batchfile批量 | 复用提示词、共享限速与重试，吞吐稳定 |
| 每张还需推理、写提示词或探索风格 | 子Agent | 各自独立分析 |
| 输入outline.md与prompts目录 | 用build-batch.ts组装批量任务 | 所需信息已齐全 |

提示词保存后任务只是“全部生成”时，优先批量而非子Agent；只有每图仍需要独立思考或发散探索时使用子Agent。

批量worker自动确定并受配置限制，内置上限10；服务限速仅用于批量模式，默认兼顾吞吐与RPM突发。--jobs可覆盖。每图最多尝试3次；最终给成功数、失败数及各失败原因。

## 错误处理

- 缺API密钥：报错并给设置方式。
- 生成失败：每图最多尝试3次。
- 比例无效：警告后使用默认比例。
- 服务/模型不支持参考图：报错并指出修正方向。

### Codex image2回退

openai/gpt-image-2因缺密钥失败，而运行环境有原生图像能力或仓库级codex-imagegen包装器时，改走可用路径，不让任务停在缺密钥。明确说明是真实参考图生成还是从视觉特征重建的文字提示词。见 `references/codex-image2-fallback.md`。

## 参考资料

| 文件 | 内容 |
| --- | --- |
| `references/usage-examples.md` | 各服务及批量CLI示例 |
| `references/codex-oauth-vs-openai-api-key.md` | Codex/ChatGPT授权与OpenAI密钥区别 |
| `references/codex-image2-fallback.md` | 缺密钥时的原生能力回退 |
| `references/providers/dashscope.md` | DashScope模型、尺寸、限制 |
| `references/providers/zai.md` | GLM-image、cogview-4 |
| `references/providers/minimax.md` | image-01与主体参考 |
| `references/providers/openrouter.md` | 多模态流程 |
| `references/providers/replicate.md` | 支持族与约束 |
| `references/providers/agnes.md` | 尺寸、参考图和限制 |
| `references/config/preferences-schema.md` | EXTEND.md结构 |
| `references/config/first-time-setup.md` | 首次设置流程 |

## 扩展设置

通过EXTEND.md自定义，路径和结构见第0步。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
