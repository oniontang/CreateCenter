---
name: guangyu-diagram
description: 生成深色主题的独立SVG图解，支持架构、流程、时序、结构、思维导图、时间线、概念关系、状态机和数据流。用户要求画图、可视化结构或流程时使用，输出独立.svg文件。
metadata:
  upstream_version: 1.117.3
---

# guangyu-diagram：流程与结构图

制作专业SVG图解。每次交付一份独立 `.svg` 文件，在文件内包含样式和字体设置。

## 支持的图解类型

| 类型 | 适用问题 | 主要特征 |
| --- | --- | --- |
| 架构图 | 系统组件与关系 | 分组、连接箭头、区域边界 |
| 流程图 | 决策逻辑、处理步骤 | 菱形判断、圆角步骤框、有向连线 |
| 时序图 | 参与者按时间交互 | 垂直生命线、水平消息、激活条 |
| 结构图 | 类图、ER图、组织图 | 分区框与继承、组合等关系 |
| 思维导图 | 发散思考、主题探索 | 中心节点、辐射分支、有机布局 |
| 时间线 | 事件先后 | 横向或纵向时间轴、事件点 |
| 概念示意图 | 解释、比较 | 自由布局、图标、标注与视觉比喻 |
| 状态机 | 状态迁移与生命周期 | 圆角状态、有标签的迁移、起止节点 |
| 数据流 | 数据转换流水线 | 处理节点、存储、外部实体 |

## 设计系统

### 色彩

按组件语义分配颜色：

| 类别 | 填充RGBA | 描边 | 用途 |
| --- | --- | --- | --- |
| Primary | `rgba(8, 51, 68, 0.4)` | `#22d3ee` | 前端、用户界面、输入 |
| Secondary | `rgba(6, 78, 59, 0.4)` | `#34d399` | 后端、服务、处理 |
| Tertiary | `rgba(76, 29, 149, 0.4)` | `#a78bfa` | 数据库、存储、持久化 |
| Accent | `rgba(120, 53, 15, 0.3)` | `#fbbf24` | 云、基础设施、区域 |
| Alert | `rgba(136, 19, 55, 0.4)` | `#fb7185` | 安全、错误、提醒 |
| Connector | `rgba(251, 146, 60, 0.3)` | `#fb923c` | 总线、队列、中间件 |
| Neutral | `rgba(30, 41, 59, 0.5)` | `#94a3b8` | 外部、通用、未知 |
| Highlight | `rgba(59, 130, 246, 0.3)` | `#60a5fa` | 激活、焦点、当前步骤 |

流程图和时序图按参与者、判断、处理等角色着色，不按技术名称着色。

### 字体

通过SVG的 `@font-face` 或系统等宽字体回退：

```svg
<style>
  @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&amp;display=swap');
  text { font-family: 'JetBrains Mono', 'SF Mono', 'Cascadia Code', monospace; }
</style>
```

字号：标题16px、字重700；组件名11—12px、字重600；副标签/说明9px、颜色 `#94a3b8`；标注8px、颜色 `#94a3b8`；箭头上的小标签7—8px。

### 基础视觉元素

背景使用 `#0f172a`（slate-900）和轻量网格：

```svg
<defs>
  <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
    <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#1e293b" stroke-width="0.5"/>
  </pattern>
</defs>
<rect width="100%" height="100%" fill="#0f172a"/>
<rect width="100%" height="100%" fill="url(#grid)"/>
```

标准箭头：

```svg
<marker id="arrow" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#64748b"/>
</marker>
```

彩色箭头按需分别定义：

```svg
<marker id="arrow-cyan" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polygon points="0 0, 10 3.5, 0 7" fill="#22d3ee"/>
</marker>
```

异步或返回消息使用开放箭头：

```svg
<marker id="arrow-open" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
  <polyline points="0 0, 10 3.5, 0 7" fill="none" stroke="#64748b" stroke-width="1.5"/>
</marker>
```

### SVG结构与图层

SVG按从后向前的顺序绘制：

1. 背景与网格。
2. 区域/分组虚线边界。
3. 箭头与连接线。
4. 与组件框同位置的不透明遮罩矩形，`fill="#0f172a"`。
5. 半透明填充与描边的组件框。
6. 文字标签。
7. 右下或底部图例，放在所有边界之外。
8. 左上标题区。

遮罩不可省略，否则半透明组件会透出下层箭头：

```svg
<!-- Mask layer: opaque background to hide arrows -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="#0f172a"/>
<!-- Visual layer: styled component -->
<rect x="100" y="100" width="160" height="60" rx="6" fill="rgba(8,51,68,0.4)" stroke="#22d3ee" stroke-width="1.5"/>
<text x="180" y="125" fill="white" font-size="11" font-weight="600" text-anchor="middle">API Gateway</text>
<text x="180" y="141" fill="#94a3b8" font-size="9" text-anchor="middle">Kong / Nginx</text>
```

### 间距

严格遵守以下规则，避免重叠：

- 标准组件框高50—70px，大型或复杂框高80—120px。
- 组件间至少纵向40px、横向30px。
- 箭头标签距离任何框边至少10px。
- 区域内组件距边界留20px。
- 图例至少在最低元素下方20px。
- 标题距左上20px，不占图解内容区。
- viewBox覆盖全部内容，并在四周留30px。

### 组件模式

标准服务/处理框：

```svg
<rect x="X" y="Y" width="160" height="60" rx="6" fill="#0f172a"/>
<rect x="X" y="Y" width="160" height="60" rx="6" fill="FILL" stroke="STROKE" stroke-width="1.5"/>
<text x="CX" y="Y+24" fill="white" font-size="11" font-weight="600" text-anchor="middle">Name</text>
<text x="CX" y="Y+40" fill="#94a3b8" font-size="9" text-anchor="middle">description</text>
```

流程判断菱形：

```svg
<g transform="translate(CX, CY)">
  <polygon points="0,-35 50,0 0,35 -50,0" fill="#0f172a"/>
  <polygon points="0,-35 50,0 0,35 -50,0" fill="rgba(120,53,15,0.3)" stroke="#fbbf24" stroke-width="1.5"/>
  <text y="4" fill="white" font-size="10" font-weight="600" text-anchor="middle">Condition?</text>
</g>
```

数据库圆柱：

```svg
<g transform="translate(X, Y)">
  <rect x="0" y="10" width="120" height="50" rx="2" fill="#0f172a"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="#0f172a"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="#0f172a"/>
  <rect x="0" y="10" width="120" height="50" fill="rgba(76,29,149,0.4)"/>
  <ellipse cx="60" cy="10" rx="60" ry="12" fill="rgba(76,29,149,0.4)" stroke="#a78bfa" stroke-width="1.5"/>
  <ellipse cx="60" cy="60" rx="60" ry="12" fill="rgba(76,29,149,0.4)" stroke="#a78bfa" stroke-width="1.5"/>
  <line x1="0" y1="10" x2="0" y2="60" stroke="#a78bfa" stroke-width="1.5"/>
  <line x1="120" y1="10" x2="120" y2="60" stroke="#a78bfa" stroke-width="1.5"/>
  <text x="60" y="40" fill="white" font-size="11" font-weight="600" text-anchor="middle">PostgreSQL</text>
</g>
```

区域边界：

```svg
<rect x="X" y="Y" width="W" height="H" rx="12" fill="none" stroke="#fbbf24" stroke-width="1" stroke-dasharray="8,4"/>
<text x="X+12" y="Y+16" fill="#fbbf24" font-size="9" font-weight="600">AWS us-east-1</text>
```

安全组：

```svg
<rect x="X" y="Y" width="W" height="H" rx="8" fill="none" stroke="#fb7185" stroke-width="1" stroke-dasharray="4,4"/>
<text x="X+10" y="Y+14" fill="#fb7185" font-size="8" font-weight="500">VPC / Security Group</text>
```

## 不同类型的布局

`{baseDir}` 是本 SKILL.md 所在目录。开始布局前，读取对应类型的参考文件，详细布局算法和例子位于 `{baseDir}/references/`。

### 架构图

读取 `{baseDir}/references/architecture.md`。数据从左到右或从上到下流动；相关服务放入区域，用总线/连接器联系不同层；数据库置底部或右侧。

### 流程图

读取 `{baseDir}/references/flowchart.md`。主流程由上到下；菱形表示判断，出口写Yes/No；起止用圆角矩形；正常路径使用Highlight色。

### 时序图

读取 `{baseDir}/references/sequence.md`。参与者放顶部，垂直虚线为生命线，水平箭头为消息；实线表示同步，虚线表示返回。时间向下推进，激活条表示处理，复杂时给消息编号。

### 结构图

读取 `{baseDir}/references/structural.md`。类图框分名称、属性、方法；实线实心菱形为组合，实线空心菱形为聚合，虚线箭头为依赖，实线三角为继承。

### 思维导图

从中心概念向外辐射，使用三次贝塞尔 `<path>` 绘制自然分支，按色板区分分支。中心字号最大，向外递减。

### 时间线

横向或纵向主轴，事件用圆点或菱形标记；说明交替放在两侧防止重叠，用颜色区分事件类型。

### 状态机

状态用圆角矩形，组合状态用双边框；实心圆为初始，靶心圆为结束；自迁移用曲线；全部迁移标注 `event [guard] / action`。

## 输出规则

1. 输出**单个.svg文件**，除Google Fonts导入外不依赖外部资源。
2. viewBox覆盖全部内容并留30px，不设置固定width/height，让图形响应式缩放。
3. 根svg包含 `xmlns="http://www.w3.org/2000/svg"`。
4. style、defs、箭头标记与pattern置于顶部。
5. 居中文字使用 `text-anchor="middle"`，文字不得超出框。
6. 中文标签使用 `font-family: 'JetBrains Mono', 'Noto Sans SC', 'PingFang SC', sans-serif`；中文更宽，需要增加框宽。
7. 输入是文件时保存到 `{inputFileDir}/diagram/`，其他情况保存到 `{projectDir}/diagram/{topic-slug}/`，不存在就创建目录。

## 脚本

脚本路径 `{baseDir}/scripts/main.ts`。`${BUN_X}` 优先为bun，其次 `npx -y bun`，都不可用则建议安装bun。

### SVG转@2x PNG

SVG保存后转换成2倍PNG：

```bash
${BUN_X} {baseDir}/scripts/main.ts <svg-path> [options]
```

参数：`-s, --scale <n>` 为缩放倍数，默认2；`-o, --output <path>` 自定义路径，默认 `<input>@2x.png`；`--json` 输出JSON。

## 制作流程

1. 根据用户请求判断图解类型。
2. 对有对应参考的类型先读参考。
3. 列组件、决定分组与流向、计算位置。
4. 按上面的图层顺序写SVG。
5. 检查间距、重叠、边界外图例和viewBox范围。
6. 保存SVG。
7. 执行 `${BUN_X} {baseDir}/scripts/main.ts <svg-path>` 生成@2x PNG。
8. 同时交付SVG与PNG。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
