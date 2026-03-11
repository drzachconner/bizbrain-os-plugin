---
name: media-downloader
description: |
  Universal media downloader — videos, audio, playlists, entire courses.
  Specialized support for Skool.com classroom bulk downloads.
  Also handles YouTube, Vimeo, Loom, Wistia, Twitter/X, Instagram, TikTok,
  and 1800+ sites via yt-dlp. Triggers on: /download, "download video",
  "download from skool", "save video", "rip video", "bulk download",
  "download course", "download playlist", "grab video", "media download".
version: 1.0.0
---

# Media Downloader

Universal media downloader with deep Skool.com classroom support. Downloads videos, audio, playlists, and entire courses from virtually any platform.

## Brain Location

Check: BIZBRAIN_PATH env → ~/bizbrain-os/

Downloads go to: `<BRAIN_PATH>/Downloads/media/` (or user-specified path)

## Installed Tools

| Tool | Location | Purpose |
|------|----------|---------|
| **yt-dlp** | `~/.local/bin/yt-dlp` | Universal video downloader (1800+ sites) |
| **gallery-dl** | `~/.local/bin/gallery-dl` | Image galleries, social media bulk downloads |
| **ffmpeg** | `C:/tools/ffmpeg/ffmpeg-8.0.1-essentials_build/bin/ffmpeg` | Video processing, merging, conversion |
| **curl** | System | Fallback HTTP downloads, cookie-based fetching |

## Core Capabilities

### 1. Standard Video Download (Any URL)

For YouTube, Vimeo, Loom, Twitter/X, TikTok, Instagram, and 1800+ sites:

```bash
# Best quality video + audio
~/.local/bin/yt-dlp -f "bestvideo+bestaudio/best" --merge-output-format mp4 "<URL>"

# Audio only (e.g., podcast, music)
~/.local/bin/yt-dlp -x --audio-format mp3 "<URL>"

# Playlist / channel (all videos)
~/.local/bin/yt-dlp -f "bestvideo+bestaudio/best" --merge-output-format mp4 \
  -o "%(playlist_title)s/%(playlist_index)03d - %(title)s.%(ext)s" "<PLAYLIST_URL>"

# With subtitles
~/.local/bin/yt-dlp --write-subs --sub-langs en -f best "<URL>"
```

### 2. Skool Classroom Downloads (SPECIALIZED)

Skool.com uses multiple video hosting backends (Wistia, Loom, Vimeo, YouTube, native HLS). Each requires a different extraction approach.

#### Prerequisites

1. User must be logged into Skool.com in their browser
2. Need to export cookies for CLI tools (see Cookie Export below)

#### Cookie Export (Required for Skool)

Tell the user to install the **"Get cookies.txt LOCALLY"** browser extension, then:

1. Navigate to `skool.com` while logged in
2. Click the extension icon
3. Export cookies for `skool.com` domain
4. Save as `cookies.txt` in the download directory

Alternatively, extract cookies from browser directly:
```bash
# yt-dlp can extract cookies from browser automatically
~/.local/bin/yt-dlp --cookies-from-browser chrome "<SKOOL_URL>"
# Or firefox:
~/.local/bin/yt-dlp --cookies-from-browser firefox "<SKOOL_URL>"
```

#### Method A: yt-dlp with Browser Cookies (Try First)

```bash
# Single lesson
~/.local/bin/yt-dlp --cookies-from-browser chrome \
  --referer "https://www.skool.com/" \
  -f "bestvideo+bestaudio/best" --merge-output-format mp4 \
  -o "%(title)s.%(ext)s" \
  "<SKOOL_LESSON_URL>"

# With cookies.txt file instead
~/.local/bin/yt-dlp --cookies cookies.txt \
  --referer "https://www.skool.com/" \
  -f "bestvideo+bestaudio/best" --merge-output-format mp4 \
  "<SKOOL_LESSON_URL>"
```

#### Method B: Bulk Classroom URL Extraction + Download

For downloading an ENTIRE Skool classroom (all modules, all lessons):

**Step 1 — Extract all lesson URLs from the classroom page.**

Tell the user to:
1. Navigate to the Skool classroom page (the one showing all modules/lessons)
2. Open browser console (F12 → Console)
3. Paste and run this JavaScript:

```javascript
// Extract all lesson URLs from Skool classroom
(() => {
  const links = [...document.querySelectorAll('a[href*="/classroom/"]')]
    .map(a => a.href)
    .filter(href => href.includes('?md='))
    .filter((v, i, a) => a.indexOf(v) === i);
  const text = links.join('\n');
  const blob = new Blob([text], {type: 'text/plain'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'skool-urls.txt';
  a.click();
  console.log(`Extracted ${links.length} lesson URLs`);
})();
```

This downloads a `skool-urls.txt` file with all lesson URLs.

**Step 2 — Download all videos from the URL list.**

```bash
# Bulk download all lessons
~/.local/bin/yt-dlp --cookies-from-browser chrome \
  --referer "https://www.skool.com/" \
  -f "bestvideo+bestaudio/best" --merge-output-format mp4 \
  -o "%(playlist_index)03d - %(title)s.%(ext)s" \
  --sleep-interval 2 --max-sleep-interval 5 \
  -a skool-urls.txt

# If yt-dlp can't extract video from lesson pages, use Method C
```

#### Method C: Manual m3u8 Stream Extraction (Fallback for Native Skool Video)

When Skool uses its own native video player (not Loom/Vimeo/YouTube), videos are served as HLS streams via Fastly CDN. These require manual URL extraction:

1. Open the Skool lesson in browser
2. Open DevTools → Network tab
3. Filter by `m3u8`
4. Play the video — look for request to `manifest-gcp-*.fastly.video.skool.com`
5. Copy the full URL (includes `?signature=...` token)

```bash
~/.local/bin/yt-dlp \
  --referer "https://www.skool.com/" \
  --add-header "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  -f "bestvideo+bestaudio/best" --merge-output-format mp4 \
  -o "%(title)s.%(ext)s" \
  "<M3U8_URL_WITH_SIGNATURE>"
```

**Important:** m3u8 tokens expire quickly. Extract and download one at a time, or batch extract URLs and download immediately.

#### Method D: Full Page Archive with curl (Offline Browsing)

For archiving entire classroom pages (HTML + assets + videos):

```bash
# Create download directory
mkdir -p skool_archive

# Download each lesson page with all assets
while IFS= read -r url; do
  curl -b cookies.txt \
    -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
    -L -o "skool_archive/$(echo "$url" | sed 's/.*\///' | sed 's/\?.*//').html" \
    "$url"
  sleep 2
done < skool-urls.txt
```

### 3. Social Media Downloads

```bash
# Twitter/X video
~/.local/bin/yt-dlp -f best "https://twitter.com/user/status/123..."

# Instagram reel/post
~/.local/bin/yt-dlp --cookies-from-browser chrome "https://instagram.com/reel/..."

# TikTok (no watermark)
~/.local/bin/yt-dlp -f best "https://tiktok.com/@user/video/..."

# Image galleries (Instagram profile, Reddit, etc.)
~/.local/bin/gallery-dl "https://instagram.com/username/"
```

### 4. Audio/Podcast Downloads

```bash
# YouTube → MP3
~/.local/bin/yt-dlp -x --audio-format mp3 --audio-quality 0 "<URL>"

# Spotify podcast (if public)
~/.local/bin/yt-dlp -x --audio-format mp3 "<SPOTIFY_EPISODE_URL>"
```

### 5. Batch Operations

```bash
# Download from URL list file
~/.local/bin/yt-dlp -a urls.txt -f best -o "%(title)s.%(ext)s"

# Download with rate limiting (polite)
~/.local/bin/yt-dlp -a urls.txt --sleep-interval 3 --max-sleep-interval 8 \
  --limit-rate 5M -f best

# Resume interrupted downloads
~/.local/bin/yt-dlp -a urls.txt --download-archive downloaded.txt \
  -f best -o "%(title)s.%(ext)s"
```

## Output Organization

Default output structure:
```
<download_dir>/
├── <platform_or_course_name>/
│   ├── 001 - Lesson Title.mp4
│   ├── 002 - Lesson Title.mp4
│   └── ...
├── downloaded.txt          # Archive of completed downloads
└── skool-urls.txt          # Source URL list (if bulk)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| 403 Forbidden | Cookie expired — re-export cookies or use `--cookies-from-browser` |
| No video found | Try `--verbose` flag; Skool may use native HLS → use Method C |
| Slow download | Add `--limit-rate 10M` to avoid throttling |
| Merge error | Ensure ffmpeg is in PATH: `export PATH="/c/tools/ffmpeg/ffmpeg-8.0.1-essentials_build/bin:$PATH"` |
| yt-dlp outdated | Run `~/.local/bin/uv tool upgrade yt-dlp` |
| Token expired (m3u8) | Re-extract from Network tab — tokens are short-lived |
| gallery-dl auth | Use `--cookies-from-browser chrome` or config file |

## Recommended Workflow for Skool Bulk Download

The optimal workflow for downloading an entire Skool classroom:

1. **Ask user** for the Skool classroom URL
2. **Guide them** through cookie export (or use `--cookies-from-browser`)
3. **Have them run** the JavaScript URL extractor in browser console
4. **Get the `skool-urls.txt`** file from them
5. **Run yt-dlp** with Method B (bulk download from URL list)
6. **If failures**, fall back to Method C for individual videos
7. **Report** results — how many downloaded, any failures

## Updating Tools

```bash
# Update yt-dlp to latest
~/.local/bin/uv tool upgrade yt-dlp

# Update gallery-dl
~/.local/bin/uv tool upgrade gallery-dl
```

## Rules

- Always ensure user has legitimate access to content before downloading
- Use `--sleep-interval` for bulk downloads to avoid rate limiting
- Never store cookies.txt in brain or git — treat as temporary credentials
- Default to best quality unless user specifies otherwise
- Always use `--merge-output-format mp4` for maximum compatibility
- For Skool: always try `--cookies-from-browser` first before manual cookie export
- Set ffmpeg path if merge operations fail: `--ffmpeg-location "C:/tools/ffmpeg/ffmpeg-8.0.1-essentials_build/bin/"`
- Report download progress and any errors clearly to user
- When downloading courses, use numbered output format (`%(playlist_index)03d`) to preserve order
