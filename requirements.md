# 会议备忘录应用 — 需求文档

## 目标

一个本地运行的会议备忘录工具。用户可以创建多条备忘录，每条支持 Markdown 文本编辑或语音录入。语音录入在点击"停止"时自动发送后端转写并将文字追加到备忘录中。所有数据和处理完全本地，不依赖云服务。

## 功能需求

### 备忘录管理
- 可创建新备忘录（自动以时间戳命名）
- 备忘录列表展示，可切换查看不同备忘录
- 每条备忘录支持 Markdown 编辑（纯文本输入，实时预览）

### 语音录入
- 点击"开始录音"按钮开始录制
- 录制中显示实时波形（确认麦克风正常工作）
- 点击"停止"后自动将音频发送后端转写
- 转写结果自动追加到当前备忘录末尾

### 文件上传转写
- 支持上传音频文件（wav/mp3/m4a 等），自动转写并追加到备忘录

## 技术架构

- 后端：Python + FastAPI，sherpa-onnx 本地语音识别
- 前端：纯 HTML/CSS/JS 单页面，由 FastAPI 直接托管，不需要 Node.js 或任何前端构建工具
- 模型：sherpa-onnx SenseVoice int8，固定路径 `models/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17/`
- 模型下载：`https://du7u4d2q1sjz6.cloudfront.net/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17.tar.bz2`
- 运行命令：`python3 server.py`，访问 `http://localhost:8000`
- **不需要写测试**，不加登录/权限等额外功能，代码越简单越好

## 硬性约束（踩坑经验，必须遵守）

以下是实际开发中验证过的约束，违反会导致录音静默或转写失败：

### 前端录音

1. **禁止客户端做音频格式转换** — `OfflineAudioContext` 在部分浏览器对 16kHz 输出全零。客户端只负责录音，原始格式直接上传，服务端用 ffmpeg 转码。

2. **禁止指定 AudioContext 采样率** — `new AudioContext({ sampleRate: 44100 })` 会导致 `MediaStreamSource` 输出静音。必须用 `new AudioContext()`（无参数）。

3. **必须处理 AudioContext suspended 状态** — Chrome autoplay policy 会把新建的 AudioContext 设为 suspended，录音前必须 `await audioContext.resume()`。

4. **必须提供麦克风设备选择器** — macOS 等多设备环境默认设备可能不对。用 `enumerateDevices()` 列出设备，`{ deviceId: { exact: id } }` 指定，监听 `devicechange` 更新。

5. **使用 AudioWorklet 代替 ScriptProcessorNode** — ScriptProcessorNode 已废弃，新版 Chrome 可能不触发回调。AudioWorklet processor 脚本需通过 HTTP 提供（不能 inline）。

6. **必须有实时波形可视化** — 用 `AnalyserNode` + Canvas 画波形。波形是直线=设备有问题，有波动=正常录音。这是用户确认麦克风工作的唯一可靠反馈。

### 后端处理

7. **服务端用 ffmpeg 做格式转换** — 客户端上传的可能是 webm/ogg/mp4/wav 任何格式，服务端统一用 `ffmpeg -ar 16000 -ac 1 -sample_fmt s16 -f wav` 转换后再送入模型。

8. **sherpa-onnx 输入要求** — 16000Hz 单声道，PCM 转 float32 归一化到 [-1.0, 1.0]（`int16 / 32768.0`）。

## 数据存储

- 所有数据以文件形式存储在本地 `data/` 目录下，不用数据库
- 每条备忘录一个子目录，目录名用创建时间戳（如 `data/2024-07-18_143052/`）
- 文字笔记存为 Markdown 文件（如 `note.md`）
- 语音录音保存原始音频文件（如 `recording_001.webm`），同时保存转写结果
- 备忘录元数据（标题、创建时间、条目列表）用一个 JSON 索引文件管理
- 重新打开应用时能加载之前保存的所有备忘录
- 编辑或新增内容后自动保存

## 运行前提

- Python 3 + `pip install sherpa-onnx fastapi uvicorn`
- ffmpeg
- 模型下载地址：`https://du7u4d2q1sjz6.cloudfront.net/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17.tar.bz2`
  - 解压后放到 `models/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17/` 目录
- 不需要 Node.js，不需要任何前端构建工具

## 课后扩展方向（可选探索）

以下方向可以在基础版本完成后继续探索，每个都可以作为独立的需求交给 AI Agent 实现：

- **会议纪要自动生成**：录音结束后，自动生成包含要点摘要、决策结论、待办事项的结构化会议纪要
- **说话人识别**：多人会议时自动区分谁在说话，标注"张三：…"、"李四：…"
- **实时字幕**：录音过程中边录边出文字，像视频字幕一样实时显示
- **智能提醒**：从会议内容中自动提取时间、截止日期，生成待办提醒
- **会议回放**：点击文字段落时跳转到对应的录音位置播放，方便回听上下文
- **多语言会议支持**：参会者说不同语言时，自动翻译成统一语言的纪要
- **团队协作**：多人同时编辑同一份备忘录，看到彼此的修改
- **知识库沉淀**：跨多次会议搜索"上次讨论过的XX方案"，关联相关会议记录
- **会议模板**：周会、1v1、头脑风暴等不同场景的预设模板，自动引导记录结构
- **语音指令**：说"标记重点"、"新建待办"等口令时自动执行对应操作
