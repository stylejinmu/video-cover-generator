---
name: video-cover-generator
description: 视频封面图生成工具。当用户上传一张图片，需要生成适配各大视频平台（微信视频号、抖音、快手、B站、小红书）的视频封面图时使用。封面图最大程度保留原图内容，留白区域使用 AI 扩图自然补全（outpainting）。支持按平台筛选、生成前确认横/竖版方向、添加文案或装饰等加工需求。
---

# 视频封面图生成

## 平台规格速查

| 平台 | 方向/用途 | 尺寸（宽×高）| 比例 | 文件大小上限 | 推荐格式 | platform_key |
|------|----------|-------------|------|------------|---------|--------------|
| 微信视频号 | 竖版 | 1080×1440 | 3:4 | 20MB | jpg/png/常见格式 | `wechat_vertical` |
| 微信视频号 | 横版 | 1440×1080 | 4:3 | 20MB | jpg/png/常见格式 | `wechat_horizontal` |
| 抖音 | 竖版 | 1080×1440 | 3:4 | 50MB | jpg/jpeg/png/webp | `douyin_vertical` |
| 抖音 | 横版 | 1440×1080 | 4:3 | 50MB | jpg/jpeg/png/webp | `douyin_horizontal` |
| 快手 | 竖版 | 1080×1440 | 3:4 | 15MB | jpg/png/常见格式 | `kuaishou_vertical` |
| 快手 | 横版 | 1440×1080 | 4:3 | 15MB | jpg/png/常见格式 | `kuaishou_horizontal` |
| B站 | 首页推荐封面 | 1280×960 | 4:3 | 20MB | jpg/png/常见格式 | `bilibili_4_3` |
| B站 | 个人主页封面 | 1920×1080 | 16:9 | 20MB | jpg/png/常见格式 | `bilibili_16_9` |
| 小红书 | 竖版 | 1080×1440 | 3:4 | 32MB | jpg/jpeg/png | `xiaohongshu_vertical` |
| 小红书 | 横版 | 1440×1080 | 4:3 | 32MB | jpg/jpeg/png | `xiaohongshu_horizontal` |

> **平台说明：**
> - **微信视频号**：推荐 3:4 至 2:1 之间，分辨率不低于 720×960。
> - **抖音**：推荐比例 3:4、4:3、1:1、9:16、16:9；不建议宽高比超过 1:2；不支持 gif 格式。
> - **快手**：推荐 3:4 竖版。
> - **B站**：生成两种尺寸——4:3 用于首页推荐封面，16:9 用于个人主页封面。
> - **小红书**：推荐 3:4 至 2:1 之间，分辨率不低于 720×960。

## 工作流程

### 第一步：确认生成参数（必须）

收到图片后，**若用户未明确说明需要哪些平台或横/竖版方向**，须先向用户确认以下信息，再开始生成：

1. **目标平台**：需要哪些平台的封面？（微信视频号 / 抖音 / 快手 / B站 / 小红书，或全部）
2. **横版还是竖版**：每个平台需要横版还是竖版？（B站会同时生成 4:3 和 16:9 两种；其他平台若只有一种方向则无需确认）
3. **文案/装饰**（可选）：是否需要在封面上添加文字、标题、装饰元素等？

示例确认问题：
> 请问您需要生成哪些平台的封面图？需要横版还是竖版？是否需要添加文案或装饰？

若用户已明确说明所有信息，可直接进入第二步。

### 第二步：运行脚本

```bash
# 生成全部平台封面
python3 <skill_dir>/scripts/generate_covers.py \
    <input_image_path> <output_dir>

# 仅生成指定平台（横版/竖版由 platform_key 区分）
python3 <skill_dir>/scripts/generate_covers.py \
    <input_image_path> <output_dir> \
    --platforms wechat_vertical douyin_vertical bilibili_4_3 bilibili_16_9

# 带文案和装饰描述（传递给 AI 扩图 prompt）
python3 <skill_dir>/scripts/generate_covers.py \
    <input_image_path> <output_dir> \
    --platforms douyin_vertical xiaohongshu_vertical \
    --text "在图片顶部添加白色标题文字'夏日旅行记'" \
    --style "添加简约白色边框装饰"

# 禁用 AI 扩图（使用模糊背景降级方案）
python3 <skill_dir>/scripts/generate_covers.py \
    <input_image_path> <output_dir> --no-ai
```

**参数说明：**
- `<skill_dir>`：本技能所在目录的路径
- `<input_image_path>`：输入图片的路径
- `<output_dir>`：输出目录，建议使用 `<input_dir>/covers/`
- `--platforms`：平台 key 列表（见上表），不填则生成全部 10 种
- `--text`：文案需求，描述要添加的文字内容和位置
- `--style`：装饰风格，描述边框、滤镜、氛围等视觉加工需求
- `--no-ai`：禁用 AI 扩图，改用模糊背景填充（速度更快，效果较差）

### 第三步：交付结果

将生成的全部封面图作为附件发送给用户，并说明每张图对应的平台、用途和尺寸。

## AI 扩图策略

脚本采用 **AI outpainting** 实现留白区域的 AI 补全：

1. **判断是否需要扩图**：若原图比例与目标比例差异 ≤15%，直接等比裁剪（无需扩图）；差异 >15% 时启用 AI 扩图。
2. **构建画布**：将原图等比缩放（contain 模式）居中放置在目标尺寸画布上，留白区域设为透明。
3. **AI 补全**：以透明区域为 mask，让 AI 根据原图风格自然延伸背景内容。
4. **降级方案**：若 AI 扩图失败（如网络错误、API 不可用），自动降级为模糊背景填充。

## 文案与装饰加工

通过 `--text` 和 `--style` 参数将加工需求传递给 AI 扩图 prompt：

- `--text`：描述文字内容、位置、颜色，如 `"在底部添加白色半透明标题'城市漫步'"`
- `--style`：描述视觉风格，如 `"添加电影感暗角效果"` 或 `"添加简约几何边框"`

> 注意：文案和装饰由 AI 在扩图时一并生成，效果取决于 AI 理解能力，复杂排版建议后期用图像编辑工具精调。

## 注意事项

- 输出格式为 JPEG，quality=95，适合平台上传
- 抖音不支持 gif 格式，本工具输出 JPEG 已符合要求
- AI 扩图需要配置相应的 API 密钥环境变量
- 依赖：`Pillow`、`openai`（`pip install pillow openai`）
- B站同时生成 4:3（首页推荐）和 16:9（个人主页）两种封面，platform_key 分别为 `bilibili_4_3` 和 `bilibili_16_9`
