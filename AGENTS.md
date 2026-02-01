# AGENTS.md

## Project overview

whisperer is a Linux push-to-talk dictation tool. Hold a key to record, transcribe via the OpenAI Audio Transcriptions API, and inject text into the focused app. Injection uses a uinput-based virtual keyboard (ydotool) with a clipboard+paste fallback for Unicode.

Main flow: push-to-talk (hold key). Extra modes: one-shot dictation (fixed duration) and looped dictation (fixed-length chunks).

## Repository layout

- `bin/whisperer`: main CLI script
- `systemd/ydotoold.service`: user service for ydotoold
- `README.md`: usage + install docs
- `LICENSE`: MIT

## Build/run

No build step. Run directly:

- `whisperer --ptt` (push-to-talk)
- `whisperer --duration 8` (one-shot)
- `whisperer --loop --chunk-seconds 4` (loop)

Optional user service:

- `whisperer --install-service` installs/enables `systemd` user service for `ydotoold`

## Dependencies (Ubuntu LTS example)

- `pipewire-bin`, `wl-clipboard`, `ydotool`, `curl`, `python3`, `python3-evdev`
- `ydotoold` may be missing on some LTS versions → build ydotool from source (see README).

## Permissions

- Requires `/dev/uinput` access (udev rule + `input` group).
- User must log out/in after adding to `input`.

## Key implementation notes

- Audio recording: `pw-record` (PipeWire), default 16kHz mono.
- Transcription: OpenAI Audio Transcriptions API via `curl`.
- Injection:
  - `ydotool type` for ASCII
  - Clipboard+paste for Unicode (Ctrl+V or Ctrl+Shift+V configurable)
- RMS gate: optional. If `WHISPERER_RMS_THRESHOLD > 0`, measure RMS on first `WHISPERER_RMS_SAMPLE_SECONDS` seconds and skip transcription if below threshold.

## Push-to-talk (PTT)

- Uses `python3-evdev` to read key events from `/dev/input/...`.
- Default key: `KEY_RIGHTALT` (some layouts use `KEY_ALTGR`).
- Device auto-detect uses `/dev/input/by-id/*-event-kbd` and prefers device names containing “Keyboard”.
- Optional selector: `WHISPERER_PTT_DEVICE_MATCH` substring.
- `--ptt-list-devices` lists candidates.
- `--debug` prints selected device/key and key events.

## Configuration

Config file: `~/.config/whisperer/config`

- Environment variables in config are sourced by the script.
- CLI flags override config values.

Key env vars:

- `OPENAI_API_KEY` (required)
- `WHISPERER_MODEL` (`whisper-1` default; also `gpt-4o-mini-transcribe`, `gpt-4o-transcribe`)
- `WHISPERER_MODE` (`auto` | `type` | `paste`)
- `WHISPERER_PASTE_KEYS` (`ctrl+v` or `ctrl+shift+v`)
- `WHISPERER_PTT`, `WHISPERER_PTT_KEY`, `WHISPERER_PTT_DEVICE`, `WHISPERER_PTT_DEVICE_MATCH`
- `WHISPERER_RMS_THRESHOLD`, `WHISPERER_RMS_SAMPLE_SECONDS`, `WHISPERER_PRINT_RMS`

## Debugging tips

- `whisperer --doctor` / `--doctor --fix` for common issues.
- If paste fails in terminals, set `WHISPERER_PASTE_KEYS=ctrl+shift+v`.
- If PTT doesn’t trigger, pick a specific device via `--ptt-device` or use `--ptt-device-match`.

## Conventions

- Keep CLI output terse; only verbose info under `--debug`.
- Prefer ASCII for edits unless the file already uses Unicode.
