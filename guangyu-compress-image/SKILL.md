---
name: guangyu-compress-image
description: 压缩图片并自动选择工具，默认输出WebP，也支持PNG与JPEG。当用户要求压缩、优化图片、转WebP或减少文件体积时使用。
metadata:
  openclaw:
    requires:
      anyBins:
      - bun
      - npx
  upstream_version: 1.56.1
---

# guangyu-compress-image：图片压缩

按可用性选择压缩工具，顺序为 sips → cwebp → ImageMagick → Sharp。

## 脚本目录

`{baseDir}` 是本 SKILL.md 所在目录，脚本位于 `scripts/`。确定 `${BUN_X}`：优先 `bun`，其次 `npx -y bun`，两者都没有时建议安装 bun。执行时将占位符换成实际值。

| 脚本 | 用途 |
| --- | --- |
| `scripts/main.ts` | 图片压缩命令行入口 |

## 偏好设置（EXTEND.md）

依次检查，使用第一个找到的文件：

| 优先级 | 路径 | 范围 |
| --- | --- | --- |
| 1 | `.guangyu-skills/guangyu-compress-image/EXTEND.md` | 当前项目 |
| 2 | `${XDG_CONFIG_HOME:-$HOME/.config}/guangyu-skills/guangyu-compress-image/EXTEND.md` | XDG配置 |
| 3 | `$HOME/.guangyu-skills/guangyu-compress-image/EXTEND.md` | 用户目录 |

均不存在时使用默认值。EXTEND.md 支持默认格式、默认质量及是否保留原文件。

## 使用方式

```bash
${BUN_X} {baseDir}/scripts/main.ts <input> [options]
```

## 参数

| 参数 | 简写 | 说明 | 默认值 |
| --- | --- | --- | --- |
| `<input>` | | 图片文件或目录 | 必填 |
| `--output` | `-o` | 输出路径 | 原路径、更换扩展名 |
| `--format` | `-f` | webp、png、jpeg | webp |
| `--quality` | `-q` | 质量0—100 | 80 |
| `--keep` | `-k` | 保留原文件 | false |
| `--recursive` | `-r` | 递归处理子目录 | false |
| `--json` | | 输出JSON | false |

## 示例

```bash
# Single file → WebP (replaces original)
${BUN_X} {baseDir}/scripts/main.ts image.png

# Keep PNG format
${BUN_X} {baseDir}/scripts/main.ts image.png -f png --keep

# Directory recursive
${BUN_X} {baseDir}/scripts/main.ts ./images/ -r -q 75

# JSON output
${BUN_X} {baseDir}/scripts/main.ts image.png --json
```

上面的示例分别是：单图转WebP并替换原文件、保持PNG并保留原文件、递归压缩目录、返回JSON。需要保留原文件时使用 `--keep`。

输出示例：

```
image.png → image.webp (245KB → 89KB, 64% reduction)
```

## 扩展设置

通过 EXTEND.md 自定义，路径及支持字段见前面的偏好设置。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
