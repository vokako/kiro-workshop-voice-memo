# Kiro Workshop Lab Guide: Building a Meeting Memo App

> 🌐 [中文版](README.md)

## Overview

In this lab, you'll use Kiro IDE's Spec mode to go from a requirement description to a fully working local meeting memo app — with AI Agent handling requirement analysis, technical design, task breakdown, and code implementation automatically.

**Final outcome**: A locally-running web app supporting memo creation, Markdown text editing, and voice-to-text transcription.

**Estimated time**: 30–45 minutes

---

## Step 1: Install Kiro IDE

Download and install Kiro IDE from [https://kiro.dev](https://kiro.dev). Log in and open an empty folder as your project directory.

> **Let Kiro handle environment setup**: Python dependencies, ffmpeg, model downloads, etc. — just include them in your prompt in Step 2 and let Kiro take care of it.

---

## Step 2: Develop with Spec Mode

### 2.1 Enter Spec Mode

Switch to **Spec mode** in Kiro (not Vibe mode).

### 2.2 Paste the Requirement Prompt

Paste the following requirement into the Spec session:

![Paste requirement prompt](images/01-kiro-spec-input.webp)

---

> Build a locally-running meeting memo application.
>
> **Functional requirements:**
> - Create multiple memos (auto-named by timestamp)
> - List all memos with switching between them
> - Each memo supports Markdown text editing (with live preview)
> - Persist data to local files; reload on app restart
> - No tests needed; keep code as simple as possible
>
> **Tech stack:**
> - Backend: Python + FastAPI
> - Frontend: plain HTML/CSS/JS served by FastAPI; no Node.js needed
> - Please check and set up the environment first: confirm Python 3 is installed, install pip dependencies (fastapi uvicorn)
>
> **Storage rules:**
> - All data stored in `data/` directory, one subdirectory per memo (e.g. `data/2024-07-18_143052/`)
> - Text notes saved as `note.md`; use a JSON index file for metadata

---

### 2.3 Choose Quick Plan

When Kiro asks how you'd like to proceed, select **Quick Plan**:

![Choose Quick Plan](images/02-kiro-quickplan.webp)

### 2.4 Answer Clarifying Questions

Kiro may ask some clarifying questions — answer them as appropriate:

![Answer questions](images/03-kiro-question.webp)

### 2.5 Confirm the Requirements Document

Kiro will generate a requirements document — review and confirm:

![Requirements document](images/04-kiro-requirements.webp)

### 2.6 Execute Tasks

Click **Run All Tasks** to let Kiro implement the code:

![Run All Tasks](images/05-kiro-run-all-task.webp)

> **Tip**: To finish quickly, only run essential tasks; skip those marked as optional.

> **Note**: Kiro may request permission to run terminal commands or write files — click **Approve** to continue.
>
> To skip all permission dialogs, enable "Trust All Tools" in Kiro settings:
>
> ![Trust Tools setting](images/05-kiro-trusttools.webp)

---

## Step 3: Launch and Test

### 3.1 Start the Server

Once all tasks show ✅ complete, type "start the app" in Kiro chat to have it launch the server.

> **Note**: First run may require additional setup (model download, etc.) — follow Kiro's output.

![Start app](images/06-kiro-startapp.webp)

### 3.2 Open the App

Open the link shown by Kiro (usually `http://localhost:8000`) in Chrome.

### 3.3 Debug Issues

If you hit errors (terminal or browser console), paste the error message to Kiro and let it debug:

![Debug](images/07-kiro-debug.webp)

### 3.4 Use the App

The app interface when everything is working:

![Use app](images/08-kiro-use-app.webp)

### 3.5 Verification Checklist

| Test | Action | Expected |
|------|--------|----------|
| Create memo | Click "New" | A new memo entry appears |
| Text editing | Type Markdown | Content is saved |
| Persistence | Refresh page | Previous memos still present |

### 3.6 Troubleshooting

Describe any issue to Kiro in natural language and let it fix it. For example:

- "Memos disappear after page refresh"
- "Markdown preview isn't showing"

---

## Step 4: Review & Reflect

After completing the lab, consider:

1. Did the Spec-generated requirements and design docs accurately reflect your intent?
2. Where did you need to intervene or correct the Agent?
3. How did the hard constraints (lessons learned) affect Agent output quality?

---

## Phase 2: Add Voice Recording & Transcription

Once the memo app is working, paste the following prompt into a new Kiro Spec session. Follow the same Quick Plan → Run All Tasks → Launch & Test flow.

---

> Add voice recording and transcription to the existing meeting memo app.
>
> **Functional requirements:**
> - Click "Start Recording" to begin; show real-time waveform while recording
> - Click "Stop" to automatically send audio to backend for transcription; append result to the current memo
> - Support uploading audio files (wav/mp3/m4a etc.) for transcription
> - Save original audio files alongside transcription text
>
> **Tech stack:**
> - Speech recognition: sherpa-onnx SenseVoice int8, path `models/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17/`
> - Model download: `https://du7u4d2q1sjz6.cloudfront.net/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2024-07-17.tar.bz2`
> - Please set up: confirm ffmpeg is installed, install pip dependency (sherpa-onnx), download and extract model to `models/`
>
> **Hard constraints (validated lessons learned, must follow):**
> 1. No client-side audio format conversion — client only records; upload raw format; server uses ffmpeg to transcode
> 2. Do not specify sampleRate when creating AudioContext — may cause silence
> 3. Must handle AudioContext suspended state — await resume() before recording
> 4. Must provide microphone device selector — use enumerateDevices() to list and select
> 5. Use AudioWorklet instead of ScriptProcessorNode
> 6. Must have real-time waveform visualization — AnalyserNode + Canvas
> 7. Server uses ffmpeg to transcode to 16kHz mono WAV before feeding the model
> 8. sherpa-onnx input: 16000Hz mono float32 normalized to [-1.0, 1.0]

---

### Phase 2 Verification Checklist

| Test | Action | Expected |
|------|--------|----------|
| Select mic | Choose from dropdown | All audio input devices listed |
| Recording | Click start | Waveform shows movement |
| Transcription | Click stop | Text auto-appended to memo |
| Upload | Upload audio file | Transcribed and appended |

---

## Optional Extensions (Homework)

Each of these can be submitted as a new requirement to Kiro Spec mode:

- **Auto-generate meeting minutes**: Produce structured summaries with action items after recording
- **Speaker identification**: Distinguish who said what in multi-person meetings
- **Real-time subtitles**: Show text as speech is being recorded
- **Smart reminders**: Extract dates/deadlines and create to-do items
- **Meeting playback**: Click text to jump to corresponding audio position
- **Multi-language**: Auto-translate mixed-language meetings into unified notes
- **Collaboration**: Multiple people editing the same memo simultaneously
- **Knowledge search**: Search across all past meeting records
- **Meeting templates**: Pre-set templates for standups, 1:1s, brainstorms
- **Voice commands**: Say "mark important" or "new action item" to trigger actions
