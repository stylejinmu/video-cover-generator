# Video Cover Generator

English | [简体中文](README_CN.md)

Automatically generate video cover images for major Chinese video platforms from a single source image. The original image content is preserved in full; blank areas introduced by aspect ratio differences are seamlessly filled using OpenAI `gpt-image-2` outpainting.

## Features

- **Multi-Platform Support**: Generates covers for WeChat Video, Douyin, Kuaishou, Bilibili, and Xiaohongshu (RED) — 9 specifications across vertical and horizontal orientations.
- **AI Outpainting**: Uses OpenAI `gpt-image-2` to naturally extend the background into blank areas, keeping the original subject intact and centered. No black bars, no awkward cropping.
- **Customization**: Pass text or decoration requirements as prompts (e.g., add a title, apply a border style) to be applied during AI generation.
- **Graceful Fallback**: If the AI service is unavailable, automatically falls back to a Gaussian blur background composite.

## Supported Platforms & Specifications

| Platform | Orientation | Size (W×H) | Ratio | `platform_key` |
|----------|-------------|------------|-------|----------------|
| WeChat Video | Vertical | 1080×1260 | 6:7 | `wechat_vertical` |
| WeChat Video | Horizontal | 1080×608 | 16:9 | `wechat_horizontal` |
| Douyin | Vertical | 1080×1920 | 9:16 | `douyin_vertical` |
| Douyin | Horizontal | 1920×1080 | 16:9 | `douyin_horizontal` |
| Kuaishou | Vertical | 1080×1920 | 9:16 | `kuaishou_vertical` |
| Kuaishou | Horizontal | 1920×1080 | 16:9 | `kuaishou_horizontal` |
| Bilibili | Horizontal | 1280×960 | 4:3 | `bilibili` |
| Xiaohongshu | Vertical | 1080×1440 | 3:4 | `xiaohongshu_vertical` |
| Xiaohongshu | Horizontal | 1440×1080 | 4:3 | `xiaohongshu_horizontal` |

## Usage

```bash
# Install dependencies
pip install pillow openai

# Generate covers for all platforms
python scripts/generate_covers.py <input_image> [output_dir]

# Generate covers for specific platforms
python scripts/generate_covers.py <input_image> [output_dir] \
    --platforms wechat_vertical douyin_vertical bilibili

# Add text and decoration via AI prompt
python scripts/generate_covers.py <input_image> [output_dir] \
    --platforms douyin_vertical xiaohongshu_vertical \
    --text "Add a white title 'Summer Travel' at the top" \
    --style "Add a minimal white border"

# Disable AI outpainting (use blur background fallback)
python scripts/generate_covers.py <input_image> [output_dir] --no-ai
```

**Arguments:**
- `<input_image>`: Path to the source image
- `[output_dir]`: Output directory (defaults to `covers/` next to the input file)
- `--platforms`: One or more `platform_key` values; omit to generate all 9
- `--text`: Text content and placement description for AI to render
- `--style`: Visual decoration description (border, filter, vignette, etc.)
- `--no-ai`: Skip AI outpainting; use Gaussian blur background instead

## How It Works

1. **Ratio check**: If the aspect ratio difference between source and target is ≤15%, the image is proportionally cropped. Otherwise, AI outpainting is used.
2. **Canvas build**: The source image is scaled to fit (contain) and centered on the target canvas; blank areas are set to transparent.
3. **AI fill**: The transparent areas are passed to `gpt-image-2` via `images.edit`, which fills them with contextually consistent content.
4. **Fallback**: On API failure, a blurred and dimmed version of the source image is used as the background.

## Requirements

- Python 3.x
- `Pillow`
- `openai`
- `OPENAI_API_KEY` environment variable

## Directory Structure

```
video-cover-generator/
├── SKILL.md                  # Skill metadata and workflow instructions
├── scripts/
│   └── generate_covers.py    # Core processing script
├── README.md                 # English documentation
└── README_CN.md              # Chinese documentation
```
