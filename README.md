# Personalized Collar Label

用一张宠物照片，生成一张**正方形、写实风格**的刺绣衣领标签。

基础版包含：小狗名字、刺绣小狗、生日。
可选追加：三枚「喜欢 / 不喜欢」圆形徽章、底部三行制作者小字。

## 效果

```
┌──────────────────────────┐
│                          │
│          Mimi            │  ← 小狗名字（刺绣书法体）
│                          │
│         [小狗刺绣]        │  ← 来自你上传的照片
│                          │
│       2022.01.02         │  ← 日期（刺绣质感）
│                          │
│    ⊘       ○       ⊘     │  ← 三枚圆形徽章（可选）
│                          │
│    DESIGNED BY WENDY     │
│    MADE IN CHINA         │  ← 三行细体小字（可选）
│    WENDY CO.,LTD         │
└──────────────────────────┘
```

## 需要提供

**必填 4 项**（一句话就能答完）：

| 项目   | 示例             |
| ---- | -------------- |
| 颜色   | 柔粉色 / 玫瑰色 / 蓝色 |
| 小狗名字 | Mimi           |
| 出生日期 | 2022.1.2       |
| 宠物照片 | jpg / png      |

**可选 3 项**（先出基础版，再问要不要加）：

| 项目                    | 示例               |
| --------------------- | ---------------- |
| 3 个特征（喜欢 / 不喜欢，共 3 个） | 讨厌巧克力、喜欢面包、不喜欢洗澡 |
| 主人姓名                  | Wendy            |
| 地点                    | 中国               |

## 安装

把整个 `personalized-collar-label` 文件夹放进项目的 skills 目录：

```bash
.trae/skills/personalized-collar-label/
├── SKILL.md
├── README.md
└── label_overlay.ps1
```

## 使用

```
帮我做一个衣领标签，柔粉色 / Mimi / 2022.1.2
```

并附上宠物照片，或输入 `/personalized-collar-label`。

工具会先出一张基础版（名字 + 小狗 + 日期），再问一次要不要加徽章和小字。想加就回一行：

```
讨厌巧克力、喜欢面包、不喜欢洗澡 / 主人 Wendy / 中国
```

不想加直接说「不用」，交付基础版。

| 你的回答     | 结果      |
| -------- | ------- |
| 不用 / 就这样 | 交付基础版   |
| 只要 3 个特征 | 只叠加徽章行  |
| 只要主人和地点  | 只叠加三行小字 |
| 两样都给     | 完整版     |

## 工作流程

1. **问 4 项必填** —— 一行模板，用户照填即可
2. **参考图刺绣化** —— 用宠物照片作参考图生成刺绣质感版本，形状、毛色、姿态尽量保持不变
3. **生成标签底图** —— 模型绘制姓名、刺绣小狗、日期，下半部分留白
4. **可选追问** —— 展示基础版，问要不要加徽章 / 署名，只收集确实需要的部分
5. **代码合成** —— `label_overlay.ps1` 用 GDI+ 精确绘制圆框徽章和细字，线宽、字号、居中完全可控
6. **逐项校验** —— 姓名拼写、日期、图标斜线状态、小字内容、间距、居中

> 为什么分两层：文字和图标交给模型画时会漂（拼错名字、图标带颜色、字太粗），交给代码画就完全可控。

## 脚本用法

```powershell
# 完整版：徽章 + 三行小字
powershell -NoProfile -ExecutionPolicy Bypass -File label_overlay.ps1 `
  -Base "output\label-base.jpg" `
  -Out  "output\label-final.png" `
  -Badges "chocolate:cross,bread:like,bath:cross" `
  -Line1 "DESIGNED BY WENDY" `
  -Line2 "MADE IN CHINA" `
  -Line3 "WENDY CO.,LTD"

# 只加徽章（省略 -Line1~3）
powershell -NoProfile -ExecutionPolicy Bypass -File label_overlay.ps1 `
  -Base "output\label-base.jpg" -Out "output\label-badges.png" `
  -Badges "chocolate:cross,bread:like,bath:cross"

# 只加三行小字（省略 -Badges）
powershell -NoProfile -ExecutionPolicy Bypass -File label_overlay.ps1 `
  -Base "output\label-base.jpg" -Out "output\label-maker.png" `
  -Line1 "DESIGNED BY WENDY" -Line2 "MADE IN CHINA" -Line3 "WENDY CO.,LTD"
```

常用参数：

| 参数                         | 说明                                                           | 默认             |
| -------------------------- | ------------------------------------------------------------ | -------------- |
| `-Badges`                  | `<图标>:<cross\|like>` 逗号分隔，`cross` 表示不喜欢（加暗红 45° 斜线）；留空则不画徽章行 | 留空             |
| `-LabelRect`               | 标签在图中的位置 `x,y,宽,高`，省略则自动检测                                   | 自动             |
| `-BadgeY`                  | 徽章行纵向位置（占标签高比例）                                              | 0.774          |
| `-TextY`                   | 第一行小字纵向位置                                                    | 0.854          |
| `-LineGap`                 | 三行小字行距                                                       | 0.0426         |
| `-FontScale` / `-PenScale` | 字号 / 线宽（占标签宽比例）                                              | 0.035 / 0.0075 |

已支持的图标：`chocolate` `bread` `bath`。其他特征需要在脚本里新增对应的绘制函数。

居中不依赖 `-LabelRect` 的精度：脚本会扫描徽章行所在的高度带，找到连续的象牙白区域并以其中点为中轴。

## 环境

- Windows PowerShell 5.1+（使用 GDI+，无需安装 Python / OpenCV / GPU）

- 图片生成由 TRAE 的 `GenerateImage` 完成

## 已知限制

- 标签宽高比约 0.8（与参考图一致），并非严格 1:1 正方形

- 背景为纯色针织面料，透明底在多数生成中不稳定

- 姓名、小狗、日期的位置由模型决定，偶尔会轻微偏移，需要重新生成

- 选择「不加可选内容」时，标签下半部会留白——这是为叠加预留的区域，属正常现象

