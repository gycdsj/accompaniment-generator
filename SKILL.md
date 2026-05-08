---
name: accompaniment-generator
description: 从 YouTube 或本地音频文件分离人声和伴奏，生成纯伴奏音乐。当用户需要提取伴奏、消除人声、制作卡拉OK版音频时使用。支持三种模式：(1) 歌名+歌手搜索 YouTube 下载后分离，(2) 直接提供视频 URL，(3) 处理本地音频文件。
---

# 伴奏生成器 (Accompaniment Generator)

## 概述

从歌曲中分离人声，输出纯伴奏音乐。核心流程：获取音频 → Demucs AI 分离人声/伴奏 → 输出 MP3。

## 核心脚本

**`scripts/get_accompaniment.py`**

### 四种使用模式

```bash
# 1. 搜索 YouTube + 下载 + 分离
python3 scripts/get_accompaniment.py "歌名" "歌手名"

# 2. 直接提供视频 URL
python3 scripts/get_accompaniment.py --url "https://youtube.com/watch?v=xxx"

# 3. 处理本地音频文件
python3 scripts/get_accompaniment.py --file /path/to/song.mp3

# 4. 带 cookies（免 YouTube bot 验证）
python3 scripts/get_accompaniment.py "歌名" "歌手" --cookies cookies.txt
```

### 输出

输出到 `~/accompaniment_output/` 目录，包含：
- `{歌名}_伴奏.mp3` — 纯伴奏（已消除人声）
- `{歌名}_人声.mp3` — 仅人声（可选）

### 参数说明

| 参数 | 说明 |
|------|------|
| `song` `artist` | 歌名和歌手（搜索用） |
| `--url` | 直接指定视频 URL |
| `--file (-f)` | 处理本地音频文件 |
| `--output (-o)` | 输出目录 |
| `--cookies` | YouTube cookies 文件路径 |
| `--json` | JSON 格式输出（供 agent 解析） |

## 工作流程

### 获取音频

**YouTube 搜索 + 下载：**
1. 用网页抓取 YouTube 搜索结果（无需 cookies）
2. 选择第一个匹配结果
3. 用 `yt-dlp` 下载音频（mp3，192kbps）

> ⚠️ 当前 YouTube 对未认证请求做 bot 限制。首次使用需提供 cookies：
> - Chrome 安装扩展 "Get cookies.txt LOCALLY"
> - 访问 youtube.com 登录后导出 cookies.txt
> - 放入 `~/.yt-dlp/cookies.txt` 或运行时 `--cookies` 指定

**本地文件：**
直接传入文件路径，跳过下载。

### AI 分离

使用 Demucs（htdemucs 模型）分离人声和伴奏：
- `--two-stems vocals`：分离为 vocals + no_vocals
- 输出 MP3 192kbps

## 依赖安装

```bash
# yt-dlp（YouTube 下载）
pip install yt-dlp

# Demucs（AI 人声分离，建议在虚拟环境中安装）
pip install demucs

# ffmpeg（格式转换）
# Ubuntu/Debian: sudo apt install ffmpeg
# macOS: brew install ffmpeg

# Deno（JS 运行时，可选，用于 YouTube 签名解算）
# curl -fsSL https://deno.land/install.sh | sh
```

## Agent 集成

使用 `--json` 参数获取结构化输出，便于自动化调用：

```json
{"success": true, "song": "晴天", "accompaniment": "/path/to/伴奏.mp3", "vocals": "/path/to/人声.mp3", "accompaniment_size_mb": 7.3}
```
