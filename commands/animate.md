---
description: Generate game character animation sprite sheet
argument-hint: <character> <action>
---

# /animate

Generate game character animations from text descriptions. Creates sprite sheets ready for Phaser, Godot, and other game engines.

## Usage

```bash
/witmani:animate "pixel knight" "running right"
/witmani:animate "cute cat" "jumping" --frames 8
/witmani:animate "robot" "walking" --format godot
```

## Parameters

- `$1` — Character description (required)
- `$2` — Action description (required)
- `--frames N` — Frame count (default: 8)
- `--fps N` — Animation frame rate (default: 10)
- `--format` — Output format: phaser (default), godot

## Instructions

You are a game animation generator. Execute these steps to create a sprite sheet.

### Step 0: Load Config and Detect Provider

First, load saved API keys from config file, then check environment:

```bash
CONFIG_FILE="$HOME/.config/witmani/config"
[ -f "$CONFIG_FILE" ] && source "$CONFIG_FILE"

if [ -n "$FAL_KEY" ]; then
  PROVIDER="fal"
elif [ -n "$GEMINI_API_KEY" ]; then
  PROVIDER="gemini"
else
  PROVIDER="none"
fi
echo "Provider: $PROVIDER"

# Background color: magenta (avoids conflict with common character colors)
BG_COLOR="#FF00FF"
echo "Background: magenta"
```

**If PROVIDER is "none"**: Use Skill tool to invoke `/witmani:setup` automatically, then return to continue after setup is complete.

### Step 1: Setup

```bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
CHARACTER_NAME=$(echo "<CHARACTER_DESCRIPTION>" | tr ' ' '_' | tr -cd '[:alnum:]_' | head -c 20)
OUTPUT_DIR="animations/${CHARACTER_NAME}_${TIMESTAMP}"
mkdir -p "$OUTPUT_DIR/frames" "$OUTPUT_DIR/frames_clean"
echo "Output: $OUTPUT_DIR"
```

---

## Provider: fal

### Step 2a: Generate Character Image (fal Flux)

Request character on **magenta background** (avoids color conflicts):

```bash
curl -s "https://fal.run/fal-ai/flux/schnell" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Single game character sprite: <CHARACTER_DESCRIPTION>. Centered, facing right, neutral pose. Style: clean, game-ready, high contrast, full body. Solid bright magenta (#FF00FF) background.",
    "image_size": {"width": 512, "height": 512},
    "num_images": 1
  }' > "$OUTPUT_DIR/image_response.json"

IMAGE_URL=$(jq -r '.images[0].url' "$OUTPUT_DIR/image_response.json")
curl -s "$IMAGE_URL" -o "$OUTPUT_DIR/character.png"
echo "Character image saved"
```

### Step 3a: Remove Background (Bria RMBG 2.0)

```bash
curl -s "https://fal.run/fal-ai/bria/background/remove" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"image_url\": \"$IMAGE_URL\"}" > "$OUTPUT_DIR/bg_response.json"

TRANSPARENT_URL=$(jq -r '.image.url' "$OUTPUT_DIR/bg_response.json")
curl -s "$TRANSPARENT_URL" -o "$OUTPUT_DIR/character_transparent.png"
echo "Background removed"
```

### Step 4a: Generate Animation Video (ByteDance Seedance Fast)

Request animation on **magenta background**:

```bash
curl -s "https://fal.run/fal-ai/bytedance/seedance/v1/pro/fast/image-to-video" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"prompt\": \"<ACTION_DESCRIPTION>, smooth loop animation, game sprite style, consistent character, solid bright magenta (#FF00FF) background, no background changes\",
    \"image_url\": \"$TRANSPARENT_URL\",
    \"duration\": 5,
    \"aspect_ratio\": \"1:1\"
  }" > "$OUTPUT_DIR/video_response.json"

VIDEO_URL=$(jq -r '.video.url' "$OUTPUT_DIR/video_response.json")
curl -s "$VIDEO_URL" -o "$OUTPUT_DIR/animation.mp4"
echo "Animation video saved"
```

---

## Provider: gemini

### Step 2b: Generate Character Image (Gemini)

```bash
curl -s "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "Generate a single game character sprite: <CHARACTER_DESCRIPTION>. Centered on solid bright magenta (#FF00FF) background, facing right, neutral pose. Style: clean, game-ready, high contrast."}]}],
    "generationConfig": {"responseModalities": ["image", "text"]}
  }' | jq -r '.candidates[0].content.parts[] | select(.inlineData) | .inlineData.data' | base64 -d > "$OUTPUT_DIR/character.png"
echo "Character image saved"
```

### Step 3b: Remove Background (skip for Gemini)

```bash
cp "$OUTPUT_DIR/character.png" "$OUTPUT_DIR/character_transparent.png"
```

### Step 4b: Generate Animation Video (Gemini Veo)

```bash
IMAGE_BASE64=$(base64 -w 0 "$OUTPUT_DIR/character_transparent.png")

RESPONSE=$(curl -s "https://generativelanguage.googleapis.com/v1beta/models/veo-2.0-generate-001:predictLongRunning?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"instances\": [{
      \"prompt\": \"<ACTION_DESCRIPTION>, smooth loop animation, game sprite style, solid magenta background\",
      \"image\": {\"bytesBase64Encoded\": \"$IMAGE_BASE64\"}
    }],
    \"parameters\": {\"aspectRatio\": \"1:1\"}
  }")

OPERATION_NAME=$(echo "$RESPONSE" | jq -r '.name')

# Poll until done
for i in {1..30}; do
  sleep 10
  STATUS=$(curl -s "https://generativelanguage.googleapis.com/v1beta/$OPERATION_NAME?key=$GEMINI_API_KEY")
  if [ "$(echo "$STATUS" | jq -r '.done')" = "true" ]; then
    echo "$STATUS" | jq -r '.response.predictions[0].bytesBase64Encoded' | base64 -d > "$OUTPUT_DIR/animation.mp4"
    echo "Animation video saved"
    break
  fi
  echo "Generating video... ($i/30)"
done
```

---

## Common Steps (all providers)

### Step 5: Extract Frames

```bash
FRAME_COUNT=<FRAME_COUNT>  # from --frames or default 8

DURATION=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$OUTPUT_DIR/animation.mp4")
EXTRACT_FPS=$(echo "scale=2; $FRAME_COUNT / $DURATION" | bc)

ffmpeg -i "$OUTPUT_DIR/animation.mp4" -vf "fps=$EXTRACT_FPS" -frames:v $FRAME_COUNT "$OUTPUT_DIR/frames/frame_%04d.png" -y 2>/dev/null
echo "Extracted $FRAME_COUNT frames"
```

### Step 5.5: Remove Background (Bria RMBG 2.0 AI)

Use AI-based background removal for accurate results (no color-based artifacts):

```bash
echo "Removing backgrounds with Bria RMBG 2.0 AI..."

# Resize frames for faster processing (API works better with smaller images)
mkdir -p "$OUTPUT_DIR/frames_small"
for f in "$OUTPUT_DIR/frames/"frame_*.png; do
  FRAME_NAME=$(basename "$f")
  ffmpeg -i "$f" -vf "scale=256:256" "$OUTPUT_DIR/frames_small/$FRAME_NAME" -y 2>/dev/null
done

# Process each frame with Bria AI
for f in "$OUTPUT_DIR/frames_small/"frame_*.png; do
  FRAME_NAME=$(basename "$f")

  TEMP_JSON=$(mktemp)
  printf '{"image_url": "data:image/png;base64,' > "$TEMP_JSON"
  base64 -w 0 "$f" >> "$TEMP_JSON"
  printf '"}' >> "$TEMP_JSON"

  RESPONSE=$(curl -s --max-time 60 "https://fal.run/fal-ai/bria/background/remove" \
    -H "Authorization: Key $FAL_KEY" \
    -H "Content-Type: application/json" \
    -d @"$TEMP_JSON")

  rm "$TEMP_JSON"

  TRANSPARENT_URL=$(echo "$RESPONSE" | jq -r '.image.url' 2>/dev/null)
  if [ "$TRANSPARENT_URL" != "null" ] && [ -n "$TRANSPARENT_URL" ]; then
    curl -s "$TRANSPARENT_URL" -o "$OUTPUT_DIR/frames_clean/$FRAME_NAME"
    echo "  $FRAME_NAME done"
  else
    echo "  $FRAME_NAME failed, retrying..."
    sleep 2
    # Retry once
    TEMP_JSON=$(mktemp)
    printf '{"image_url": "data:image/png;base64,' > "$TEMP_JSON"
    base64 -w 0 "$f" >> "$TEMP_JSON"
    printf '"}' >> "$TEMP_JSON"
    RESPONSE=$(curl -s --max-time 60 "https://fal.run/fal-ai/bria/background/remove" \
      -H "Authorization: Key $FAL_KEY" -H "Content-Type: application/json" -d @"$TEMP_JSON")
    rm "$TEMP_JSON"
    TRANSPARENT_URL=$(echo "$RESPONSE" | jq -r '.image.url' 2>/dev/null)
    [ "$TRANSPARENT_URL" != "null" ] && curl -s "$TRANSPARENT_URL" -o "$OUTPUT_DIR/frames_clean/$FRAME_NAME"
  fi

  sleep 1  # Rate limiting
done

echo "Bria RMBG 2.0 completed"

FRAMES_DIR="$OUTPUT_DIR/frames_clean"
```

### Step 6: Create Sprite Sheet

```bash
COLS=$([ $FRAME_COUNT -le 8 ] && echo $FRAME_COUNT || echo 8)
ROWS=$(( ($FRAME_COUNT + $COLS - 1) / $COLS ))

ffmpeg -i "$FRAMES_DIR/frame_%04d.png" -filter_complex "tile=${COLS}x${ROWS}" "$OUTPUT_DIR/${CHARACTER_NAME}.png" -y 2>/dev/null
echo "Sprite sheet created"
```

### Step 7: Create Preview GIF (with transparency)

```bash
FPS=<FPS>  # from --fps or default 10
ffmpeg -i "$FRAMES_DIR/frame_%04d.png" -vf "fps=$FPS,scale=128:-1:flags=lanczos,split[s0][s1];[s0]palettegen=reserve_transparent=1[p];[s1][p]paletteuse=alpha_threshold=128" -loop 0 "$OUTPUT_DIR/${CHARACTER_NAME}.gif" -y 2>/dev/null
echo "Preview GIF created"
```

### Step 8: Generate Config File

Get frame dimensions:
```bash
FRAME_WIDTH=$(ffprobe -v error -select_streams v:0 -show_entries stream=width -of default=noprint_wrappers=1:nokey=1 "$FRAMES_DIR/frame_0001.png")
FRAME_HEIGHT=$(ffprobe -v error -select_streams v:0 -show_entries stream=height -of default=noprint_wrappers=1:nokey=1 "$FRAMES_DIR/frame_0001.png")
```

**Phaser format** (default):
```bash
cat > "$OUTPUT_DIR/${CHARACTER_NAME}.json" << EOF
{
  "textureName": "${CHARACTER_NAME}",
  "frameWidth": $FRAME_WIDTH,
  "frameHeight": $FRAME_HEIGHT,
  "totalFrames": $FRAME_COUNT,
  "animations": [
    {
      "key": "<ACTION_KEY>",
      "start": 0,
      "end": $(($FRAME_COUNT - 1)),
      "frameRate": $FPS,
      "repeat": -1
    }
  ]
}
EOF
```

**Godot format** (if --format godot):
```bash
cat > "$OUTPUT_DIR/${CHARACTER_NAME}.tres" << EOF
[gd_resource type="SpriteFrames" format=3]

[resource]
animations = [{
  "frames": [],
  "loop": true,
  "name": &"<ACTION_KEY>",
  "speed": ${FPS}.0
}]
EOF
```

### Step 9: Copy Preview HTML with Embedded Data

```bash
PREVIEW_SRC="$HOME/.claude/plugins/marketplaces/game-animator/assets/preview.html"
if [ -f "$PREVIEW_SRC" ]; then
  # Use Python to inject base64 (handles large files)
  python3 << PYEOF
import base64, json

with open("$OUTPUT_DIR/${CHARACTER_NAME}.png", "rb") as f:
    sprite_b64 = base64.b64encode(f.read()).decode()

with open("$OUTPUT_DIR/${CHARACTER_NAME}.json", "r") as f:
    config_json = f.read().strip()

with open("$PREVIEW_SRC", "r") as f:
    html = f.read()

html = html.replace('const AUTO_SPRITE_BASE64 = null;', f'const AUTO_SPRITE_BASE64 = "{sprite_b64}";')
html = html.replace('const AUTO_CONFIG_JSON = null;', f'const AUTO_CONFIG_JSON = {config_json};')

with open("$OUTPUT_DIR/preview.html", "w") as f:
    f.write(html)

print("Preview created with embedded animation")
PYEOF
fi
```

### Step 10: Report Results

Show the user:
```
Animation generated successfully!

Output folder: $OUTPUT_DIR/
  - ${CHARACTER_NAME}.png (sprite sheet)
  - ${CHARACTER_NAME}.json (Phaser config)
  - ${CHARACTER_NAME}.gif (preview)
  - preview.html (browser preview)

Frame size: ${FRAME_WIDTH}x${FRAME_HEIGHT}
Total frames: $FRAME_COUNT
FPS: $FPS

Preview: Open $OUTPUT_DIR/preview.html in browser, drag sprite sheet to preview.

Phaser usage:
  this.load.spritesheet('${CHARACTER_NAME}', '${CHARACTER_NAME}.png', {
    frameWidth: $FRAME_WIDTH,
    frameHeight: $FRAME_HEIGHT
  });
```

## Error Handling

- If PROVIDER is "none": Auto-invoke /witmani:setup
- If ffmpeg not found: Tell user to install it (`apt install ffmpeg` or `brew install ffmpeg`)
- If API call fails: Show error message and suggest checking API key
- If video generation times out: Suggest trying again or using a different provider

## Output Structure

```
animations/<character>_<timestamp>/
├── <character>.png       # Sprite sheet (game-ready, transparent)
├── <character>.json      # Phaser animation config
├── <character>.gif       # Preview animation
├── character.png         # Original character
├── character_transparent.png
├── animation.mp4         # Source video
├── frames/               # Raw extracted frames
│   ├── frame_0001.png
│   └── ...
└── frames_clean/         # Background-removed & cleaned frames
    ├── frame_0001.png
    └── ...
```
