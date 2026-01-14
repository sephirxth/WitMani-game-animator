# WitMani Game Animator

Generate game character animations with AI. Text to sprite sheet in seconds.

![Demo](https://via.placeholder.com/600x200?text=Demo+GIF+Coming+Soon)

## Features

- **Text-to-Animation** - Describe character and action, get sprite sheet
- **Auto Preview** - Browser preview with Phaser 3 & Babylon.js
- **Multiple Formats** - Export to Phaser, Godot
- **Smart Background Removal** - Bria RMBG 2.0 or fast chromakey
- **Zero Config** - Setup wizard guides you through

## Installation

```bash
/plugin install https://github.com/sephirxth/WitMani-game-animator
```

After installation, **run setup**:
```bash
/witmani:setup
```

## Quick Start

```bash
# Generate a pixel knight running animation
/witmani:animate "pixel knight" "running right"

# Output in animations/ folder with preview.html
```

## Commands

| Command | Description |
|---------|-------------|
| `/witmani:animate` | Generate sprite sheet animation |
| `/witmani:setup` | Configure API keys and preferences |

## Usage Examples

```bash
# Basic
/witmani:animate "fire mage" "casting spell"

# With options
/witmani:animate "cyber samurai" "sword slash" --frames 8 --fps 12

# For Godot
/witmani:animate "slime" "bouncing" --format godot
```

## Output

Each generation creates a folder with:

```
animations/pixel_knight_20240115/
├── pixel_knight.png      # Sprite sheet (transparent)
├── pixel_knight.json     # Animation config
├── pixel_knight.gif      # Quick preview
└── preview.html          # Interactive browser preview
```

### Browser Preview

Open `preview.html` in any browser - it auto-loads and plays your animation with Phaser 3 or Babylon.js. No server needed.

## Prompt Tips

For best results, use this formula:

```
"[character] with [details], [action], game sprite style"
```

**Good examples**:
- `"pixel knight with silver armor" "running right"`
- `"cute slime, blue and bouncy" "jumping"`
- `"fire mage in red robe" "casting fireball spell"`

**Tips**:
- Keep character description concise but specific
- Action should be clear and single (not "running and jumping")
- Add style hints: `"pixel art style"`, `"anime style"`, `"chibi"`

## Requirements

- **ffmpeg** - For frame extraction (setup will guide installation)
- **API Key** - fal.ai (recommended) or Gemini

## Supported Providers

| Provider | API Key | Get Key |
|----------|---------|---------|
| fal.ai | `FAL_KEY` | [fal.ai/dashboard/keys](https://fal.ai/dashboard/keys) |
| Gemini | `GEMINI_API_KEY` | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |

## License

MIT

---

Made with AI by **WitMani**
