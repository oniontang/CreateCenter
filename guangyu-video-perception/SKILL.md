---
name: guangyu-video-perception
description: 当用户提供视频文件（.mp4、.mov、.avi、.mkv、.webm）、YouTube链接，要求观看、分析或审查视频，或在对话中引用视频内容时使用。
---

# guangyu-video-perception：视频内容感知

通过 claude-video-vision MCP 服务调用视频理解工具，结合结构、画面和声音理解内容。

## 可用工具

- `video_analyze`：用 ffmpeg 滤镜分析结构，包括场景变化、静默、运动等。先分析，再决定如何抽帧。
- `video_watch`：提取画面并处理音频，支持不同片段使用不同帧率与分辨率。
- `video_detail`：深入查看局部片段。将提取与查看分开，可先提取多帧、再按需查看少量画面。
- `video_info`：获取视频元数据，不处理正文内容。
- `video_configure`：修改后端、分辨率、enable_index 等设置。
- `video_setup`：检查或安装依赖。

## 工作流程

**必须按顺序执行，不能跳过第2步。**

1. 先运行 `video_info`，取得时长、分辨率及是否有音轨。用户提供 YouTube 链接时，直接将链接作为 `path`。服务通过 `yt-dlp` 下载，优先使用 YouTube 人工或自动字幕；仅在字幕缺失、为空或明显不完整时，才回退到已配置的音频后端。
2. **超过30秒的视频，必须先调用 `video_analyze`，再提取画面。** 结构分析用于决定抽帧位置，不是可省略的步骤。根据问题选择滤镜：

   | 用户问题 | 滤镜 |
   | --- | --- |
   | 视频讲了什么 | scene_changes、silence、transcription |
   | 查找转场 | scene_changes、black_intervals |
   | 有没有卡住或冻结的片段 | freeze、blur |
   | 是口播还是动作画面 | motion |
   | 音乐何时开始 | silence、loudness |
   | 分析光线 | exposure |
   | 总结课程 | transcription、scene_changes、silence |
   | 通用或尚未明确 | scene_changes、silence、transcription |

   视频有声音时，始终包含 `transcription: true`，用转写定位需要观察的位置。`scene_changes: true` 检测硬切，scdet 阈值为8。要看叠化、淡入淡出等柔和转场时用 `scene_changes: { threshold: 4 }`；手持或高速运动造成过多结果时可提高到 `{ threshold: 15 }`。
3. 根据分析和转写安排抽帧。静态或可预测片段用0.1—0.5 FPS；只在转场、运动峰值或“看这里”“如你所见”“我来演示”等语音指向的位置提高到1—3 FPS。使用完成任务所需的最低帧率，优先少量片段和较低帧率，再按需深入。
4. 调用 `video_watch`。少于2分钟的短视频用 `fps: "auto"`，不传 `view_sample`，以免漏掉短暂细节；自动帧率已按时长调整。超过2分钟的视频按分析结果传 `segments`，分别设置帧率，用 `view_sample` 控制首次返回数量，再用 `video_detail` 深入。
5. 查看局部时，从关注点前后3—5秒开始，先用 `view_sample: 3` 看首、中、尾三帧，需要时再用 `view` 指定时间点。只有现有窗口不足时才扩大。像二分查找一样缩小观察范围，不一次查看所有已提取画面。
6. 用户继续问同一个视频时，复用上下文中的 manifest。已有相同分辨率的画面不重复提取，已在上下文中的画面不重复请求。

## 参数说明

- **fps**：概览用 `"auto"`；逐帧分析用 `video_info` 中的原视频帧率；短时刻细查用5—10；长视频用0.1—0.5。
- **resolution**：快速扫描用256—512，普通分析用512—768，读取屏幕文字和细节时用1024或更高。
- **segments**：有分析数据时使用。每段可单独设置帧率与分辨率，并覆盖全局 fps、start_time、end_time。
- **view_sample**：从已提取画面中均匀返回N帧，避免大量图片占满上下文。
- **skip_audio**：只分析画面时设为 true。
- **YouTube链接**：支持的URL直接作为 `path`。`transcription_source: "youtube_subtitles"` 的可靠性高于 `youtube_auto_captions`，自动字幕仍可能有识别错误。

## 结果处理

工具可能返回：

- **Manifest**：开启 enable_index 时，按分辨率和时间戳索引缓存画面，避免重复请求。
- **画面图片**：实际查看，理解视觉事件。
- **带时间戳的音频转写**：读取讲话内容。
- **音频标签**：音乐、音效等非语音事件。
- **分析数据**：转场、静默区间、运动量等。

结合以上来源形成完整理解。用结构分析和转写指导看哪些画面：分析告诉你事件**何时**发生，画面告诉你**发生了什么**。

## 来源与许可

来源署名与适用许可见 [根目录说明](../来源与许可.md)。执行时使用本文件与当前目录内的参考资料。
