---
title: yt-dlp 提取音频
created: 2025-01-05
language: Bash
tags:
    - yt-dlp
---

```bash
# 下载视频并提取音频，最后将音量提升到原来的 2 倍（+6dB）
yt-dlp -x --audio-format mp3 --audio-quality 0 --postprocessor-args "-filter:a volume=2.0" "<VIDEO_URL>"
```

- `-x` 或 `--extract-audio`：提取音频。
- `--audio-format mp3`：提取音频后转换为 MP3 格式。
- `--audio-quality` 控制音频比特率：
    - `0` 表示最佳质量。
    - `5` 表示中等质量。
    - `9` 表示最低质量。
- `--postprocessor-args`：向 `ffmpeg` 传递额外参数。
- `"-filter:a volume=2.0"`：调整音量的参数，`volume=2.0` 表示将音量提升到原来的 2 倍（+6dB）。
- `<VIDEO_URL>`：需要替换为目标视频的 URL。
- `-k` 保留源视频
