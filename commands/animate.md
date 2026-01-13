---
description: Generate game character animation sprite sheet
argument-hint: <character> <action>
aliases:
  - witmani
---

# /animate

Generate game character animations from text descriptions. Creates sprite sheets ready for Phaser, Godot, and other game engines.

## Usage

```bash
/animate "pixel knight" "running right"
/animate "cute cat" "jumping" --frames 8
/witmani "robot" "walking" --format godot
```

## Parameters

- `$1` — Character description (required)
- `$2` — Action description (required)
- `--frames N` — Frame count (default: 8)
- `--fps N` — Animation frame rate (default: 10)
- `--format` — Output format: phaser (default), godot

## Instructions

You are a game animation generator. Execute these steps to create a sprite sheet.

### Step 0: Detect Provider

Check which API key is available (in order of priority):

```bash
if [ -n "$FAL_KEY" ]; then
  PROVIDER="fal"
elif [ -n "$GEMINI_API_KEY" ]; then
  PROVIDER="gemini"
else
  PROVIDER="none"
fi
echo "Provider: $PROVIDER"
```

**If PROVIDER is "none"**, stop and show this welcome message:

```
Welcome to WitMani Game Animator!

To get started, set up one API key (takes 2 minutes):

Option A: fal.ai (recommended - best quality)
  1. Visit https://fal.ai/dashboard/keys
  2. Create a key
  3. Run: export FAL_KEY="your-key"

Option B: Gemini (generous free tier)
  1. Visit https://aistudio.google.com/apikey
  2. Create a key
  3. Run: export GEMINI_API_KEY="your-key"

Then try: /animate "pixel knight" "running"
```

### Step 1: Setup

```bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
CHARACTER_NAME=$(echo "<CHARACTER_DESCRIPTION>" | tr ' ' '_' | tr -cd '[:alnum:]_' | head -c 20)
OUTPUT_DIR="animations/${CHARACTER_NAME}_${TIMESTAMP}"
mkdir -p "$OUTPUT_DIR/frames"
echo "Output: $OUTPUT_DIR"
```

---

## Provider: fal

### Step 2a: Generate Character Image (fal Flux)

```bash
curl -s "https://fal.run/fal-ai/flux/schnell" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Single game character sprite: <CHARACTER_DESCRIPTION>. Centered, facing right, neutral pose. Style: clean, game-ready, high contrast, full body, solid background.",
    "image_size": {"width": 512, "height": 512},
    "num_images": 1
  }' > "$OUTPUT_DIR/image_response.json"

IMAGE_URL=$(jq -r '.images[0].url' "$OUTPUT_DIR/image_response.json")
curl -s "$IMAGE_URL" -o "$OUTPUT_DIR/character.png"
echo "Character image saved"
```

### Step 3a: Remove Background (fal BiRefNet)

```bash
curl -s "https://fal.run/fal-ai/birefnet" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"image_url\": \"$IMAGE_URL\"}" > "$OUTPUT_DIR/bg_response.json"

TRANSPARENT_URL=$(jq -r '.image.url' "$OUTPUT_DIR/bg_response.json")
curl -s "$TRANSPARENT_URL" -o "$OUTPUT_DIR/character_transparent.png"
echo "Background removed"
```

### Step 4a: Generate Animation Video (fal Kling)

```bash
curl -s "https://fal.run/fal-ai/kling-video/v1/standard/image-to-video" \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"prompt\": \"<ACTION_DESCRIPTION>, smooth loop animation, game sprite style, consistent character\",
    \"image_url\": \"$TRANSPARENT_URL\",
    \"duration\": \"5\",
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
    "contents": [{"parts": [{"text": "Generate a single game character sprite: <CHARACTER_DESCRIPTION>. Centered on solid color background, facing right, neutral pose. Style: clean, game-ready, high contrast."}]}],
    "generationConfig": {"responseModalities": ["image", "text"]}
  }' | jq -r '.candidates[0].content.parts[] | select(.inlineData) | .inlineData.data' | base64 -d > "$OUTPUT_DIR/character.png"
echo "Character image saved"
```

### Step 3b: Remove Background (requires REMOVEBG_API_KEY or skip)

If REMOVEBG_API_KEY is set:
```bash
curl -s -H "X-Api-Key: $REMOVEBG_API_KEY" \
  -F "image_file=@$OUTPUT_DIR/character.png" \
  -F "size=auto" \
  https://api.remove.bg/v1.0/removebg \
  -o "$OUTPUT_DIR/character_transparent.png"
```

Otherwise, use the original image:
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
      \"prompt\": \"<ACTION_DESCRIPTION>, smooth loop animation, game sprite style\",
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

### Step 6: Create Sprite Sheet

```bash
COLS=$([ $FRAME_COUNT -le 8 ] && echo $FRAME_COUNT || echo 8)
ROWS=$(( ($FRAME_COUNT + $COLS - 1) / $COLS ))

ffmpeg -i "$OUTPUT_DIR/frames/frame_%04d.png" -filter_complex "tile=${COLS}x${ROWS}" "$OUTPUT_DIR/${CHARACTER_NAME}.png" -y 2>/dev/null
echo "Sprite sheet created"
```

### Step 7: Create Preview GIF

```bash
FPS=<FPS>  # from --fps or default 10
ffmpeg -i "$OUTPUT_DIR/frames/frame_%04d.png" -vf "fps=$FPS,scale=128:-1:flags=lanczos" -loop 0 "$OUTPUT_DIR/${CHARACTER_NAME}.gif" -y 2>/dev/null
echo "Preview GIF created"
```

### Step 8: Generate Config File

Get frame dimensions:
```bash
FRAME_WIDTH=$(ffprobe -v error -select_streams v:0 -show_entries stream=width -of default=noprint_wrappers=1:nokey=1 "$OUTPUT_DIR/frames/frame_0001.png")
FRAME_HEIGHT=$(ffprobe -v error -select_streams v:0 -show_entries stream=height -of default=noprint_wrappers=1:nokey=1 "$OUTPUT_DIR/frames/frame_0001.png")
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

### Step 9: Report Results

Show the user:
```
Animation generated successfully!

Output folder: $OUTPUT_DIR/
  - ${CHARACTER_NAME}.png (sprite sheet)
  - ${CHARACTER_NAME}.json (Phaser config)
  - ${CHARACTER_NAME}.gif (preview)

Frame size: ${FRAME_WIDTH}x${FRAME_HEIGHT}
Total frames: $FRAME_COUNT
FPS: $FPS

Phaser usage:
  this.load.spritesheet('${CHARACTER_NAME}', '${CHARACTER_NAME}.png', {
    frameWidth: $FRAME_WIDTH,
    frameHeight: $FRAME_HEIGHT
  });
```

## Error Handling

- If ffmpeg not found: Tell user to install it (`apt install ffmpeg` or `brew install ffmpeg`)
- If API call fails: Show error message and suggest checking API key
- If video generation times out: Suggest trying again or using a different provider

## Output Structure

```
animations/<character>_<timestamp>/
├── <character>.png       # Sprite sheet (game-ready)
├── <character>.json      # Phaser animation config
├── <character>.gif       # Preview animation
├── character.png         # Original character
├── character_transparent.png
├── animation.mp4         # Source video
└── frames/
    ├── frame_0001.png
    └── ...
```
