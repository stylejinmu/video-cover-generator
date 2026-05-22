# Video Cover Generator

English | [简体中文](README_CN.md)

Automatically generate video cover images for major Chinese video platforms from a single source image. The original image content is preserved in full; blank areas introduced by aspect ratio differences are seamlessly filled using AI outpainting.

## Features

- **Multi-Platform Support**: Generates covers for WeChat Video, Douyin, Kuaishou, Bilibili, and Xiaohongshu (RED) — 10 specifications across vertical and horizontal orientations.
- **AI Outpainting**: Naturally extends the background into blank areas, keeping the original subject intact and centered. No black bars, no awkward cropping.
- **Customization**: Pass text or decoration requirements as prompts (e.g., add a title, apply a border style) to be applied during AI generation.
- **Graceful Fallback**: If the AI service is unavailable, automatically falls back to a Gaussian blur background composite.

## Supported Platforms & Specifications

| Platform | Orientation / Usage | Size (W×H) | Ratio | Max File Size | `platform_key` |
|----------|---------------------|------------|-------|--------------|----------------|
| WeChat Video | Vertical | 1080×1440 | 3:4 | 20MB | `wechat_vertical` |
| WeChat Video | Horizontal | 1440×1080 | 4:3 | 20MB | `wechat_horizontal` |
| Douyin | Vertical | 1080×1440 | 3:4 | 50MB | `douyin_vertical` |
| Douyin | Horizontal | 1440×1080 | 4:3 | 50MB | `douyin_horizontal` |
| Kuaishou | Vertical | 1080×1440 | 3:4 | 15MB | `kuaishou_vertical` |
| Kuaishou | Horizontal | 1440×1080 | 4:3 | 15MB | `kuaishou_horizontal` |
| Bilibili | Homepage Recommended Cover | 1280×960 | 4:3 | 20MB | `bilibili_4_3` |
| Bilibili | Profile Page Cover | 1920×1080 | 16:9 | 20MB | `bilibili_16_9` |
| Xiaohongshu | Vertical | 1080×1440 | 3:4 | 32MB | `xiaohongshu_vertical` |
| Xiaohongshu | Horizontal | 1440×1080 | 4:3 | 32MB | `xiaohongshu_horizontal` |

> **Platform Notes:**
> - **WeChat Video**: Recommended ratio 3:4 to 2:1, minimum resolution 720×960.
> - **Douyin**: Recommended ratios 3:4, 4:3, 1:1, 9:16, 16:9; ratio exceeding 1:2 is not recommended; GIF format not supported.
> - **Kuaishou**: Recommended 3:4 vertical.
> - **Bilibili**: Two sizes generated — 4:3 for homepage recommended cover, 16:9 for profile page cover.
> - **Xiaohongshu**: Recommended ratio 3:4 to 2:1, minimum resolution 720×960.

## Usage

```bash
# Install dependencies
pip install pillow openai

# Generate covers for all platforms
python scripts/generate_covers.py <input_image> [output_dir]

# Generate covers for specific platforms
python scripts/generate_covers.py <input_image> [output_dir] \
    --platforms wechat_vertical douyin_vertical bilibili_4_3 bilibili_16_9

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
- `--platforms`: One or more `platform_key` values; omit to generate all 10
- `--text`: Text content and placement description for AI to render
- `--style`: Visual decoration description (border, filter, vignette, etc.)
- `--no-ai`: Skip AI outpainting; use Gaussian blur background instead

## How It Works

1. **Ratio check**: If the aspect ratio difference between source and target is ≤15%, the image is proportionally cropped. Otherwise, AI outpainting is used.
2. **Canvas build**: The source image is scaled to fit (contain) and centered on the target canvas; blank areas are set to transparent.
3. **AI fill**: The transparent areas are filled by AI with contextually consistent content matching the original style.
4. **Fallback**: On API failure, a blurred and dimmed version of the source image is used as the background.

## Requirements

- Python 3.x
- `Pillow`
- `openai`
- Corresponding API key environment variable configured

## Directory Structure

```
video-cover-generator/
├── SKILL.md                  # Skill metadata and workflow instructions
├── scripts/
│   └── generate_covers.py    # Core processing script
├── README.md                 # English documentation
└── README_CN.md              # Chinese documentation
```
