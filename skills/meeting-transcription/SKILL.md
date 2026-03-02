---
name: meeting-transcription
description: |
  Use for local meeting transcription — starting/stopping the daemon, viewing transcripts,
  checking status, managing meeting recordings, and processing AI summaries. Triggers on:
  /meetings, "transcribe my meeting", "start recording", "meeting notes", "what was discussed",
  reviewing transcripts, "summarize meeting". This is a Local-First Free Alternative — replaces
  Otter.ai/Fireflies for $0/month. Supports Windows (WASAPI) and macOS (BlackHole).
version: 2.0.0
---

# Meeting Transcription

You manage local meeting transcription for BizBrain OS. This feature captures system audio
via platform-native loopback, transcribes with faster-whisper (local, offline), and saves
brain-compatible transcripts with automatic intake integration, entity detection, and
AI summary preparation.

## Brain Location

Check: BIZBRAIN_PATH env → ~/bizbrain-os/

## Platform Support

| Platform | Audio Capture | Meeting Detection | Setup Required |
|----------|--------------|-------------------|----------------|
| **Windows** | WASAPI loopback (built-in) | Win32 window titles (ctypes) | None — works out of the box |
| **macOS** | BlackHole virtual device | psutil + Quartz window API | Install BlackHole + configure Multi-Output Device |

## How It Works

1. **Audio Capture:** Platform-native loopback records whatever plays through speakers/headphones
2. **Platform Agnostic:** Works with Zoom, Meet, Teams, Slack, Discord — any audio source
3. **Local Transcription:** faster-whisper runs entirely on the user's machine (no cloud, no API keys)
4. **Brain Integration:** Transcripts saved as markdown, enriched intake files dropped for AI summarization
5. **Entity Detection:** Automatically detects mentioned entities from ENTITY-INDEX.md
6. **Permanent Recordings:** Full meeting audio stitched into single WAV files (kept forever by default)
7. **Optional Diarization:** pyannote-audio identifies individual speakers across full meeting (not just first chunk)

## Key Paths

| Item | Path |
|------|------|
| Transcripts | `<BRAIN>/Operations/meetings/transcripts/` |
| Recordings | `<BRAIN>/Operations/meetings/recordings/` |
| Audio chunks (temp) | `<BRAIN>/Operations/meetings/_audio/` |
| Daemon PID | `<BRAIN>/.bizbrain/meeting-daemon.pid` |
| Daemon status | `<BRAIN>/.bizbrain/meeting-daemon-status.json` |
| Intake summaries | `<BRAIN>/_intake-dump/files/meeting-*.md` |
| Python package | `${CLAUDE_PLUGIN_ROOT}/tools/meeting-transcriber/` |

## Auto-Install (Zero-Friction Setup)

The meeting transcriber auto-installs on first use. **Do NOT show install instructions to the user.**

### How Auto-Install Works

When ANY `/meetings` command is invoked, follow this sequence BEFORE executing the command:

1. **Check if installed** — run via Bash:
   ```bash
   python -c "import meeting_transcriber; print('installed')" 2>/dev/null && echo "OK" || echo "NOT_INSTALLED"
   ```

2. **If NOT_INSTALLED** — auto-install via Bash:
   ```bash
   bizbrain-meetings install
   ```
   This command (in `${CLAUDE_PLUGIN_ROOT}/tools/meeting-transcriber`) handles everything:
   - Detects platform (Windows/macOS)
   - Finds `uv` or falls back to `pip`
   - Installs with correct platform extras
   - Creates brain directories
   - On macOS: checks for BlackHole and prints setup steps if missing

3. **If install succeeds** (exit code 0) — proceed with the original command
4. **If install fails** — show the error output and suggest:
   ```
   Manual install: cd ${CLAUDE_PLUGIN_ROOT}/tools/meeting-transcriber && uv pip install -e ".[windows]"
   ```
   (Replace `[windows]` with `[macos]` on macOS)

### Important: Run the Commands Yourself

**You (Claude) run these commands via Bash. Do NOT show them to the user and ask them to run them.** The user should only see a brief status message like "Installing meeting transcriber..." followed by confirmation.

### macOS BlackHole (One Manual Step)

macOS requires BlackHole for system audio capture. This is the ONE thing the user must do manually — `bizbrain-meetings install` will detect if BlackHole is missing and print instructions. If you see the BlackHole warning in install output, relay these steps to the user:

1. Install BlackHole 2ch: `brew install blackhole-2ch` (or https://existential.audio/blackhole/)
2. Open Audio MIDI Setup → Create Multi-Output Device
3. Check both speakers/headphones AND BlackHole 2ch
4. Set Multi-Output Device as system output

Windows requires no manual audio setup (WASAPI is built into the OS).

## Commands

### `/meetings` or `/meetings status`

Show current daemon status:
1. Read `<BRAIN>/.bizbrain/meeting-daemon-status.json`
2. Show: running/stopped, PID, meeting active, chunks recorded
3. If meeting is active, show platform, title, duration so far

### `/meetings setup`

Run diagnostics (does NOT install — use `/meetings start` which auto-installs):
1. Execute `bizbrain-meetings setup` via Bash (if installed) or `bizbrain-meetings install` (if not)
2. Report dependency status and platform readiness
3. On macOS: check BlackHole audio device configuration

### `/meetings start`

Start the transcription daemon:
1. **Auto-install if needed** (follow the Auto-Install sequence above)
2. Check if daemon is already running (read PID file, check if process exists)
3. If not running, start it in the background:
   ```bash
   bizbrain-meetings daemon --model base &
   ```
4. Available flags:
   - `--model tiny|base|small|medium|large-v3` — Whisper model size (default: base)
   - `--language en` — Force language (default: auto-detect)
   - `--diarize` — Enable speaker diarization (needs pyannote + HF_TOKEN)
   - `--keep-audio` — Keep recordings forever (default)
   - `--delete-audio-after N` — Delete audio chunks after N days
5. Confirm daemon started, show PID

**Important:** The daemon is resource-intensive when transcribing (loads Whisper model into memory).
The `base` model uses ~150MB RAM; `large-v3` uses ~3GB. Only start when the user expects a meeting.

### `/meetings stop`

Stop the running daemon:
1. Execute `bizbrain-meetings stop`
2. Confirm stopped
3. If a meeting was in progress, it will be transcribed before stopping

### `/meetings list`

List recent transcripts:
1. Read files in `<BRAIN>/Operations/meetings/transcripts/*.md`
2. Show table: date, title, platform, duration, word count
3. Sort by date descending, show last 10

### `/meetings <filename>`

View a specific transcript:
1. Read the file from `<BRAIN>/Operations/meetings/transcripts/<filename>`
2. If filename doesn't end in .md, append it
3. Display the transcript content
4. Also read the `.meta.json` sidecar if it exists for additional context

## Transcript Format

Transcripts are saved as markdown:

```markdown
# Meeting Transcript — Weekly Standup

**Platform:** zoom
**Date:** 2026-02-28
**Started:** 09:00
**Ended:** 09:30
**Duration:** 30 minutes
**Recording:** Operations/meetings/recordings/2026-02-28-weekly-standup.wav

---

## Transcript

[00:00] Welcome everyone, let's start with updates...
[00:45] The API refactor is on track for Friday.
[02:15] Can we talk about the deployment pipeline?
...

---

*Transcribed locally by BizBrain OS meeting transcriber.*
```

With speaker diarization:

```markdown
## Transcript

**SPEAKER_00** [00:00]
> Welcome everyone, let's start with updates.

**SPEAKER_01** [00:15]
> The API refactor is on track for Friday.
```

## Intake Integration & AI Summary

When a meeting is transcribed, an enriched intake file is dropped to `<BRAIN>/_intake-dump/files/`.
This file includes the **full transcript text** and a `Needs-AI-Summary: true` flag.

### Post-Meeting AI Summary Workflow

When you detect an intake file with `Needs-AI-Summary: true`:

1. **Read** the full intake file (contains complete transcript)
2. **Generate** summary sections:
   - Executive summary (3-5 sentences)
   - Key topics (bullet points)
   - Decisions made
   - Action items (with assignees if mentioned)
3. **Write** the generated content back to the intake file, replacing the HTML comment placeholders
4. **Route action items** to appropriate entity files:
   ```python
   from meeting_transcriber.brain_updater import BrainUpdater
   updater = BrainUpdater(brain_path)
   updater.write_action_items(meeting_info, action_items)
   ```
   Each action item dict: `{"text": "...", "owner": "Name", "entity": "EntityName"}`
5. **Remove** the `Needs-AI-Summary: true` flag (or set to `false`)

### Entity Detection

The intake file includes a `Detected-Entities:` field listing entities found by
keyword-matching against ENTITY-INDEX.md. Entity history files are automatically
updated with a meeting reference entry (no LLM needed for this step).

## Audio Retention

- **Permanent recordings:** Stitched single WAV files in `Operations/meetings/recordings/` — kept forever by default
- **Temp chunks:** Raw 5-minute WAV chunks in `Operations/meetings/_audio/` — kept forever by default, configurable via `--delete-audio-after N`
- **Transcripts:** Always kept permanently

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "No loopback device found" (Windows) | Check that a WASAPI audio output device is active. Plug in headphones or speakers. |
| "BlackHole audio device not found" (macOS) | Install BlackHole and run `bizbrain-meetings setup` for instructions. |
| "Model download failed" | faster-whisper downloads models on first use. Ensure internet connection for initial setup. |
| Daemon exits immediately | Run `bizbrain-meetings daemon` in foreground (without &) to see error messages |
| No audio captured (Windows) | Ensure meeting audio plays through the default output device |
| No audio captured (macOS) | Ensure Multi-Output Device is set as system output and includes both speakers + BlackHole |
| Poor transcription quality | Try a larger model: `--model small` or `--model medium` |
| Only first speaker labeled | Upgrade to v0.2.0 — fixes full-meeting diarization (was only first 5-min chunk) |
