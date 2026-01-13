# WitMani Game Animator

Generate game character animations with AI. Text to sprite sheet in seconds.

## Features

- Text-to-animation generation
- Export to Phaser & Godot formats
- Multiple AI providers (fal.ai, Gemini)
- Zero config - auto-detects your API key
- Open source (MIT)

## Installation

```bash
# Add marketplace
/plugin marketplace add witmani/game-animator

# Install plugin
/plugin install game-animator@witmani-game-animator

# Restart Claude Code
```

## Quick Start

1. **Set up API key** (choose one):

   ```bash
   # Option A: fal.ai (recommended)
   export FAL_KEY="your-key"

   # Option B: Gemini (generous free tier)
   export GEMINI_API_KEY="your-key"
   ```

2. **Generate animation**:

   ```bash
   /animate "pixel knight" "running right"
   ```

3. **Check output** in `animations/` folder

## Usage

### Basic

```bash
/animate "character description" "action"
```

### With options

```bash
/animate "pixel cat" "jumping" --frames 8 --fps 12
```

### Check environment

```bash
/animate-setup
```

## Output

```
animations/
├── pixel_knight.png      # Sprite sheet
├── pixel_knight.json     # Phaser/Godot config
└── pixel_knight.gif      # Preview
```

### Phaser Integration

```javascript
// Load sprite sheet
this.load.spritesheet('knight', 'pixel_knight.png', {
  frameWidth: 64,
  frameHeight: 64
});

// Create animation
this.anims.create({
  key: 'run',
  frames: this.anims.generateFrameNumbers('knight', { start: 0, end: 7 }),
  frameRate: 10,
  repeat: -1
});
```

## Supported Providers

| Provider | API Key | Best For |
|----------|---------|----------|
| fal.ai | `FAL_KEY` | Quality, speed |
| Gemini | `GEMINI_API_KEY` | Free tier |

## Requirements

- ffmpeg (for frame extraction)
- One API key from supported providers

## License

MIT

---

Made with care by WitMani
