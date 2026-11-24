# 存档契约

`guangyu-save` 写入存档，`guangyu-restore` 恢复存档。两者共用本文件定义的位置、命名和格式；复盘模块产生的结论通过guangyu-save保存，不依赖其他未打包模块。

两个 Skill 各自带一份内容相同的副本（`references/archive-contract.md`），任一 Skill 单独安装也能读取。

## 1. 用户面向的措辞

跟用户对话时不暴露内部术语：

- 「snapshot」说成「存档」，一份创作状态文件叫一份存档；
- 「session」说成「对话」或「下次回来」；
- 「slug」说成「项目」，每个项目下有独立的存档目录。

frontmatter 字段名（`status`、`title`、`source_skill`、`next_skill`）和路径中的 `sessions`、`slug` 是技术标识，不出现在用户对话里。

## 2. 项目名（slug）

每份存档属于一个项目，用来分隔不同内容项目。

- **默认项目名**：取自 `basename $(pwd)`。保留中文、英文字母和数字，英文字母转小写；空格、标点和其他符号替换成 `-`，连续的 `-` 合并为一个，去掉首尾的 `-`。处理后为空时用 `default`。例如「命名讨论」保持为 `命名讨论`，`My Shop (2026)` 变成 `my-shop-2026`。
- **用户显式指定**：使用 `--slug <项目名>`。
- 当前目录是用户家目录或没有明确项目时，项目名为 `default`。

## 3. 存档根目录

先解析存档根目录，后续路径都基于这个结果拼接。

配置文件固定为当前工作目录下的 `.guangyu-skills/archive-config.json`：

```json
{
  "version": 1,
  "mode": "default"
}
```

| mode | 存档根目录 |
|---|---|
| `default` | `~/.guangyu-skills/archive/` |
| `project` | 当前工作目录下的 `.guangyu-skills/archive/` |
| `custom` | `root` 字段指定的目录 |

自定义示例：

```json
{
  "version": 1,
  "mode": "custom",
  "root": "~/Documents/guangyu-archive"
}
```

解析规则：

1. 读取当前工作目录下的 `.guangyu-skills/archive-config.json`。
2. 文件不存在，或 `mode` 是 `default` 时，存档根目录为 `~/.guangyu-skills/archive/`。
3. `mode` 是 `project` 时，存档根目录为当前工作目录下的 `.guangyu-skills/archive/`。
4. `mode` 是 `custom` 时，读取 `root`。展开开头的 `~`；相对路径按当前工作目录解析。
5. 配置无法解析、`mode` 不受支持、`root` 为空，或路径指向 `/`、用户家目录、当前项目根目录时，停止当前操作并说明配置问题。不要到其他位置猜测，也不要静默退回默认位置。

只有 `guangyu-save` 的 `location` 命令可以写入配置文件，`guangyu-restore` 只读取配置。

## 4. 目录与文件名

```text
{存档根目录}/sessions/{项目名}/{YYYYMMDD-HHMMSS}-{标题}.md
```

- `YYYYMMDD-HHMMSS` 使用用户确认的时区时间；本课默认Asia/Shanghai，不直接假定宿主操作系统时区正确。
- 文件名中的标题把空格和标点替换成 `-`，保留中英文字符。
- 同一秒已有同名文件时，追加 4 位随机后缀，例如 `-a7k2`。
- 排序和「最新」一律按文件名前缀的时间判断，不依赖文件修改时间；云同步可能改写修改时间。
- 存档每次新建，不覆盖已有文件。

## 5. 存档文件格式

YAML frontmatter 加 Markdown 正文。

```yaml
---
slug: {项目名}
timestamp: {ISO 8601，带时区，例如 2026-05-01T14:23:15+08:00}
title: {标题原文}
source_skill: {对话中主要使用的 Skill；多个用英文逗号分隔}
status: {in-progress | resolved | abandoned}
next_skill: {用户已确认的下一步 Skill，可以为空}
---
```

`timestamp` 不能写成 `20260501T14:23:15+0800`。生成时优先使用：

```bash
python3 -c "from datetime import datetime, timezone, timedelta; print(datetime.now(timezone(timedelta(hours=8))).isoformat(timespec='seconds'))"
```

`status` 的取值与对用户展示的中文：

| 取值 | 含义 | 展示 |
|---|---|---|
| `in-progress` | 创作还没结束，下次回来要接着 | 进行中 |
| `resolved` | 本次创作任务已完成，结论已确定 | 已结论 |
| `abandoned` | 用户决定放弃这个方向 | 已放弃 |

不确定时默认 `in-progress`。resolved只表示本次任务结论已定，不代表已经发布或效果得到验证；拍摄、发布、反馈状态另在备注中记录。示例代码使用Asia/Shanghai的UTC+08:00；用户指定其他时区时按该时区生成。

正文固定 6 段，顺序如下：

1. `## 创作目标`：用户最初的问题，1～2 句，保留原话语气。
2. `## 已得出的结论`：每条一句话。
3. `## 用户已否决的方向`：每条写明否决理由；没有时写「（暂无）」。
4. `## 待验证假设`：没有时写「（暂无）」。
5. `## 已确认的下一步`：用户已经确认时，写做什么、为什么、对应哪个 Skill；尚未确认时写「待定，等用户明确下一步」。
6. `## 备注`：任何对以后回访有用的信息。

旧版存档中名为「用户目标」或「用户主诉」的段落按「创作目标」读取，名为「推荐下一步」的段落按「已确认的下一步」读取。读取时遇到缺失的字段或段落，使用现有内容继续，不中断。

## 6. next_skill 的含义

- 只有用户已经明确选择下一个 Skill，或明确确认按某个 Skill 继续时，才填写 `next_skill`。
- 上一个 Skill 只给出候选方向、用户尚未确认时，留空，恢复后询问用户从哪里继续。
- 恢复存档时，只有用户明确说按上次确认的下一步继续，才进入 `next_skill` 指向的 Skill，并把存档内容作为上下文。


## 7. 内容项目的补充记录

六个固定段落保留，内容记录放入相应段落，不另外引入不兼容的格式。详细字段见 [content-session.md](content-session.md)。

- 创作目标：受众、平台、本条解决的问题和用户已确认的方向。
- 已得出的结论：材料卡、内容方案、有效修改及用户选择的原因。
- 已否决的方向：不采用的标题、表达或论证与具体理由。
- 待验证假设：效果判断、证据缺口及下次内容实验。
- 已确认的下一步：只写用户已确认的动作；没有就写待定，不自动调用模块。
- 备注：最终稿和附件的实际路径、版本、拍摄/发布状态、数据采样窗口、复盘卡路径。

只有外部文件尚未落盘且用户要保存当前全文时，在备注下原样收录已确认最终稿。文件已存在则核对路径和版本；需要锁定历史时，经用户保存授权复制一份版本化副本，不覆盖原稿。不能只写一个未来可能不存在的文件名并声称全文已保存。

source_skill只填实际参与的模块。next_skill仅填本套实际存在且用户确认要继续的模块；不确定留空。保存不代表已经产生新Skill，单条内容的经验先作为带适用条件的候选判断。
