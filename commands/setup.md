---
description: Check environment and setup for WitMani Game Animator
---

# /setup

Check your environment and configure API keys for WitMani Game Animator.

## Instructions

### Step 1: Load Existing Config

```bash
CONFIG_DIR="$HOME/.config/witmani"
CONFIG_FILE="$CONFIG_DIR/config"
[ -f "$CONFIG_FILE" ] && source "$CONFIG_FILE"
```

### Step 2: Check ffmpeg

```bash
if command -v ffmpeg &> /dev/null; then
  FFMPEG_VERSION=$(ffmpeg -version | head -1 | awk '{print $3}')
  echo "ffmpeg: installed (v$FFMPEG_VERSION)"
else
  echo "ffmpeg: not found"
fi
```

### Step 3: Check API Keys

```bash
[ -n "$FAL_KEY" ] && echo "FAL_KEY: configured" || echo "FAL_KEY: not set"
[ -n "$GEMINI_API_KEY" ] && echo "GEMINI_API_KEY: configured" || echo "GEMINI_API_KEY: not set"
```

### Step 4: Display Results

Show environment status to user in this format:

```
WitMani Game Animator - Environment Check

DEPENDENCIES
  ffmpeg: [status]

API KEYS (need at least one)
  FAL_KEY:         [status]
  GEMINI_API_KEY:  [status]

Config file: ~/.config/witmani/config
```

### Step 5: Handle Missing Dependencies

**If ffmpeg not found:**
```
ffmpeg is required. Install with:
  Ubuntu/Debian: sudo apt install ffmpeg
  macOS:         brew install ffmpeg
```

**If no API key configured:**

Use AskUserQuestion tool to let user choose provider and paste their key:

Question: "Which AI provider do you want to use?"
Options:
- "fal.ai (recommended)" - Get key at https://fal.ai/dashboard/keys
- "Gemini (free tier)" - Get key at https://aistudio.google.com/apikey

After user selects, ask them to paste the API key directly in the chat.

Then **save the key to config file** (persists across sessions):

```bash
mkdir -p "$HOME/.config/witmani"
echo 'export FAL_KEY="user-provided-key"' > "$HOME/.config/witmani/config"
chmod 600 "$HOME/.config/witmani/config"
echo "API key saved to ~/.config/witmani/config"
```

Or for Gemini:
```bash
mkdir -p "$HOME/.config/witmani"
echo 'export GEMINI_API_KEY="user-provided-key"' > "$HOME/.config/witmani/config"
chmod 600 "$HOME/.config/witmani/config"
echo "API key saved to ~/.config/witmani/config"
```

Also export in current session so it works immediately:
```bash
export FAL_KEY="user-provided-key"
```

### Step 6: Ready Message

If everything is configured:
```
Ready! Try: /witmani:animate "pixel knight" "running right"
```
