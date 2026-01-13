---
description: Check environment and setup for WitMani Game Animator
---

# /animate-setup

Check your environment and get setup instructions for WitMani Game Animator.

## Usage

```bash
/animate-setup
```

## Instructions

Run environment checks and display status to the user.

### Step 1: Check ffmpeg

```bash
if command -v ffmpeg &> /dev/null; then
  FFMPEG_VERSION=$(ffmpeg -version | head -1 | awk '{print $3}')
  FFMPEG_STATUS="installed (v$FFMPEG_VERSION)"
else
  FFMPEG_STATUS="not found"
fi
```

### Step 2: Check API Keys

```bash
FAL_STATUS="not set"
GEMINI_STATUS="not set"

[ -n "$FAL_KEY" ] && FAL_STATUS="configured"
[ -n "$GEMINI_API_KEY" ] && GEMINI_STATUS="configured"
```

### Step 3: Display Results

Show this formatted output:

```
WitMani Game Animator - Environment Check

DEPENDENCIES
  ffmpeg: [FFMPEG_STATUS]

API KEYS (need at least one)
  FAL_KEY:         [FAL_STATUS]
  GEMINI_API_KEY:  [GEMINI_STATUS]
```

### Step 4: Show Setup Guide (if needed)

If ffmpeg is not found:
```
SETUP: ffmpeg

ffmpeg is required for video processing.

Install:
  Ubuntu/Debian: sudo apt install ffmpeg
  macOS:         brew install ffmpeg
  Windows:       choco install ffmpeg
```

If no API key is configured:
```
SETUP: API Key

Choose one provider and set up your key:

Option A: fal.ai (recommended - best quality)
  1. Go to https://fal.ai/dashboard/keys
  2. Click "Create Key"
  3. Run: export FAL_KEY="your-key-here"

Option B: Gemini (generous free tier)
  1. Go to https://aistudio.google.com/apikey
  2. Click "Create API Key"
  3. Run: export GEMINI_API_KEY="your-key-here"

Tip: Add the export to your ~/.bashrc or ~/.zshrc to make it permanent.
```

### Step 5: Ready Message

If everything is configured:
```
Ready to generate animations!

Try: /animate "pixel knight" "running right"
```
