---
name: "personalized-collar-label"
description: "Generates a centered square photoreal collar label — ivory satin with the pet's name, birth date, embroidered pet motif from the user's photo, three laundry-care icons for the pet's likes/dislikes, and a three-line maker block (DESIGNED BY / MADE IN / CO.,LTD). Invoke when the user asks for a custom clothing or collar label."
---

# Personalized Collar Label

Two-stage workflow that preserves the uploaded pet photo's shape/color/pose 1:1 and only swaps surface texture for embroidery. Both stages run inside `GenerateImage` using the `image_paths` reference capability — no extra software or GPU needed.

## Inputs

**Required — four items only.** Everything else is optional and asked later, after the user has seen a result.

| Slot | User example | Placeholder |
| --- | --- | --- |
| Color | 柔粉色 / 玫瑰色 / 蓝色 | `{COLOR}` |
| **Pet name** (goes on the label) | Wendy / Mimi / OA | `{PET_NAME}` |
| Birth date | 2024.1.20 | `{DATE}` |
| Pet photo | file path to jpg/png | raw reference image |

**Optional — never asked up front.** Only after the base label has been shown:

| Slot | User example | Placeholder |
| --- | --- | --- |
| Three traits (likes / dislikes, exactly 3) | 讨厌巧克力、喜欢面包、不喜欢洗澡 | `{TRAIT_1}` `{TRAIT_2}` `{TRAIT_3}` |
| Owner name | Wendy | `{OWNER}` |
| Location | 中国 / 杭州 | `{PLACE}` |

Never invent a real person's name, date, or location. If a required slot is missing, ask for the missing ones in one message — see the opening line in Step 0.

> Naming rule: the label's big calligraphy is the **pet's** name (`{PET_NAME}`). `{OWNER}` only appears in the small maker block at the bottom.

## Step 0 — Ask for the four required items, then normalize

**Ask first, in one short message, with a copyable one-line pattern.** Do not list them as a form, and do not mention traits, owner or location yet:

```
做一张小狗衣领标签，只要 4 样东西：
颜色 ○○ / 小狗名字 ○○ / 生日 ○○ / 小狗照片
例如：柔粉色 / Mimi / 2022.1.2 + 附图
```

Rules:

- Missing items → ask only for the missing ones, keep the same one-line pattern.
- The user may answer in any order, or paste the whole line at once.
- Do not ask about traits, owner or location here — those come in Step 3.

**Color** → one lowercase English color phrase, used consistently everywhere. 柔粉色 → `soft pink`, 玫瑰色 / 粉色 → `rose-pink`, 蓝色 → `sky-blue`, 绿色 → `sage green`, 黄色 → `butter yellow`, 紫色 → `lavender`, 白色 → `soft white`, 灰色 → `warm grey`, 红色 → `berry red`, 棕色 → `camel brown`, 黑色 → `charcoal`.

**Date** → `YYYY.MM.DD`, zero-padded, dots kept. `2022.1.2` → `2022.01.02`.

**Pet name** → keep the user's original capitalization.

**Traits → three laundry-care badges.** One horizontal row of **three identical circular badges** — same diameter, same stroke weight, evenly spaced, no captions. Every badge is a **black thin-line circle**; the object inside is **black line art only, never colored or filled**.

- "讨厌 X" / "不喜欢 X" → black circular frame + black line-art object inside + one straight **dark-red slash from top-left to bottom-right at exactly 45°**, spanning the circle.
- "喜欢 X" / "爱 X" → the same black circular frame + black line-art object inside, **no slash at all**.

Never use a bare icon without the circular frame, and never mix filled/outlined styles across the three badges.

Map to simple recognizable line-art symbols:

| Trait keyword | Icon to draw |
| --- | --- |
| 巧克力 chocolate | segmented chocolate bar |
| 面包 bread | loaf / sliced toast |
| 洗澡 bath | bathtub with water and shower head |
| 骨头 bone / 肉 meat | bone / steak |
| 球 ball / 玩具 toy | ball with curved seams / toy ball |
| 散步 walk / 出门 | dog leash or walking dog silhouette |
| 睡觉 sleep | crescent moon with zzz |
| 猫 cat | cat head silhouette with whiskers |
| 牛奶 milk | milk carton / bottle |
| 苹果 apple / 水果 fruit | apple with leaf |
| 冰淇淋 ice cream | ice-cream cone |
| 拍照 photo | camera |
| 梳毛 brush / 理发 | grooming brush or scissors |

If a trait has no match, pick the closest simple household object that reads instantly at small size. Record the final three badges as `{ICON_1}` `{ICON_2}` `{ICON_3}`, each written as "<object> inside the black circle, crossed by a dark-red 45° top-left to bottom-right slash" (dislike) or "<object> inside the black circle, no slash" (like).

**Pet photo → anchor description**: read the image, write one line: species + pose + one unique mark, e.g. "fluffy cream-white puppy with pale golden ears, dark round eyes, black nose, lying down with front paws stretched forward and head lifted". The reference image carries the fidelity; the text is only an anchor.

## Step 1 — Generate the embroidered pet (texture-only transfer)

Call `GenerateImage` with the **raw pet photo** as `image_paths`.

```text
Convert the subject exactly as shown in the reference photo into embroidery. Keep the subject's shape, proportions, pose, ear shape, eye positions, nose shape, paw placement, fur colors and markings 100% identical to the reference — do NOT redraw or reinterpret anything. Only change the surface texture from fur to realistic embroidery stitches. The final result is a photoreal close-up macro of {PET_ANCHOR}, isolated on a plain light gray studio background, 1:1 square, high detail, soft daylight, no props, no people, no text, no labels. Avoid: altered shape, altered proportions, altered pose, altered ear shape, altered eye/nose positions, altered fur colors/markings, cartoon, painting, stylized, simplified, beautified, cluttered background, watermark, text, overlays
```

- `image_paths`: `[<user's pet photo path>]`
- `image_size`: `square`
- `path`: `output/embroidery-<pet-slug>`

Save the returned path as `{EMBROIDERED_PET_PATH}`.

## Step 2 — Generate the blank label base (name + pet + date only)

The badges and the maker block are **not** generated by the model — they are drawn by code in Step 3 so the stroke weight, circle size and alignment are exact. So the base image must leave that area empty.

Call `GenerateImage` with `{EMBROIDERED_PET_PATH}` as `image_paths`.

```text
Personalized collar label macro product photo. A square ivory satin clothing label — width and height roughly equal, a compact square patch, not a long narrow strip — perfectly centered top-down view with no rotation. Surrounding the label is a uniform solid pastel {COLOR} background with an embroidered-knit texture — fine, dense knitted surface that appears slightly stitched, no visible seams, no folds, no piping, no ribbon, no hem. The label is one continuous flat sheet of ivory satin with a thin pale {COLOR} trim band and neat {COLOR} zigzag stitching along its four outer edges only; no horizontal seam, no crease, no pocket line, no stitching across the middle.

The upper 58 percent of the label holds three stacked elements, compact and close together, every one of them horizontally centered on the label's vertical axis with identical left and right margins:
1. Large elegant {COLOR} embroidery-thread calligraphy reads exactly "{PET_NAME}" — raised satin-stitch letters with visible thread texture.
2. EXACTLY the embroidered pet from the reference image, moderate size — reproduce its shape, proportions, pose, ear shape, fur colors and markings 100% identical, maintaining the embroidery stitch texture. Do NOT redraw or reinterpret the pet.
3. Small {COLOR} date "{DATE}" rendered as EMBROIDERY, not printed type: raised satin-stitch numerals with visible thread texture, slight gloss and soft shadow, identical stitching quality to the name.

The bottom 40 percent of the label, below the date, is completely empty flat ivory satin — a large clean blank area with no icons, no symbols, no text, no decoration, no line.

Photorealistic macro product photography, tactile textile, realistic satin sheen, subtle natural shadows, soft diffuse top studio lighting, airy exposure, restrained contrast, sharp central details with gentle peripheral softness. 1:1 aspect ratio, no people, no hands, no extra props. Avoid: off-center name, off-center date, off-center pet, uneven left and right margins, altered pet shape, altered pet colors, altered pet pose, altered pet ears, altered pet proportions, cartoon, painting, redrawn subject, dark lighting, dramatic shadows, oversaturated {COLOR}, purple tones, cluttered background, extra labels, extra buttons, long narrow strip shape, tall skinny label, horizontal seam line, horizontal fold, crease, pocket line, stitching across the middle, flat printed date, plain ink date, any icons, any symbols, any text below the date, watermark, logo overlay, hands, people, garment folds, piping, ribbon, illegible text, wrong date
```

- `image_paths`: `[<{EMBROIDERED_PET_PATH}>]`
- `image_size`: `square`
- `path`: `output/label-base-<pet-slug>`

Save the returned path as `{BASE_PATH}`.

## Step 3 — Show the base and offer the optional pass (one message)

Deliver `{BASE_PATH}` first, then ask once — yes/no first, details only if they say yes:

```
基础版好了 👆（名字 + 小狗 + 日期）
要不要再加两样？想加回我一行就行：
1️⃣ 3 个「喜欢 / 不喜欢」→ 圆形徽章：讨厌 ○○、喜欢 ○○、不喜欢 ○○
   想不出就挑：洗澡 / 吹风机 / 吸尘器 / 剪指甲 / 去医院 / 打雷 ; 零食 / 面包 / 奶酪 / 球 / 散步 / 抱抱 / 睡觉
2️⃣ 底部三行小字：主人 ○○、地点 ○○
不想加就说「不用」，我直接交付这张。
```

Then:

- **"不用" / "就这样"** → deliver `{BASE_PATH}` as final. The empty lower band is intentional negative space; do not regenerate just to fill it, and do not run the script.
- **Wants 1 or 2** → run only the requested parts (the script skips blank `-Badges` and blank `-Line1..3`), and collect only the missing details with the same one-line pattern.

Trait rules (only when they opt in):

- Exactly 3 items, any mix of likes and dislikes. Fewer → ask only for the missing ones; more → keep the first three and say so.
- Vague answer ("不喜欢出门") → offer one or two concrete options: "是不喜欢坐车，还是不喜欢下雨天散步？"
- Never invent a trait — it gets printed on the label.
- Reduce each answer to one concrete object/action: "不喜欢洗澡" → `洗澡` + dislike.

Owner / place (only when they opt in):

- Owner name keeps the user's original capitalization. Place → uppercase English: 中国 → `CHINA`, 杭州 → `HANGZHOU`, 上海 → `SHANGHAI`; if only a country is given, use the country.
- Maker block (three lines, uppercase, black):
```
DESIGNED BY {OWNER}
MADE IN {PLACE}
{OWNER} CO.,LTD
```
- Missing owner or place → ask for it; never invent one.

## Step 4 — Composite the badges and maker block with code

Run the repository script with the `Shell` tool:

```bash
powershell -NoProfile -ExecutionPolicy Bypass -File "tools/label_overlay.ps1" ^
  -Base "{BASE_PATH}" ^
  -Out "output/collar-label-<pet-slug>-<YYYYMMDD>.png" ^
  -Badges "{BADGE_SPEC}" ^
  -Line1 "DESIGNED BY {OWNER}" ^
  -Line2 "MADE IN {PLACE}" ^
  -Line3 "{OWNER} CO.,LTD" ^
  -LabelRect "<x,y,w,h>"
```

`{BADGE_SPEC}` is a comma-separated list of `<kind>:<cross|like>` in display order, e.g. `chocolate:cross,bread:like,bath:cross`. Supported kinds in the script: `chocolate`, `bread`, `bath` — each drawn as pure black line art inside an identical black circle, with a dark-red 45° top-left→bottom-right slash only when the trait is a dislike.

`-LabelRect` is the label bounding box in pixels of the base image. Omit it first: the script estimates the box from the low-saturation band and prints `label rect: ...`. If the printed box is clearly too small (e.g. it only caught the blank lower area), read the base image, measure the label's left/top/width/height, and re-run with an explicit `-LabelRect`. Then check the result: the badge row sits just under the date (about half the old gap), the three maker lines are tight against each other, and the last line still keeps a safe clearance of roughly 5 percent of the label height above the bottom edge — close, but never touching the zigzag border. Adjust `-BadgeY` / `-TextY` / `-LineGap` if the base image's date sits higher or lower than expected.

Centering is handled by the script and does not depend on `-LabelRect` being pixel-accurate: before drawing, it scans the badge-row band for the contiguous ivory run and centres the badge row and the three text lines on that run's midpoint (printed as `centerX = ...`). So the overlay always lands on the label's true horizontal centre with equal left and right margins. Only the model-drawn elements (name, pet, date) can drift sideways — that is a base-image problem and is fixed by regenerating, not by the script.

Sizing is deliberately decoupled from the label shape: on the square label the width-relative scales were retuned (`r = 0.055 × label width`, font `= 0.035 × label width`, stroke `= 0.0075 × label width`) so the badges, icons and text keep exactly the same pixel size they had on the old tall label. Do not enlarge them to fill the wider patch.

## Step 5 — Verify before delivering

Only check the parts the user opted into (badges and/or maker block); checks 1–2 apply to every output.

1. `{PET_NAME}` spelled correctly in the base image, and the label reads as a compact square patch (not a tall strip).
2. Date matches `{DATE}` exactly and looks like raised embroidery thread, not flat printed ink.
3. Three badges: one row, identical circles, all black strokes, no colour fill; slash only on the dislikes, dark red, top-left → bottom-right 45°.
4. Maker block: three lines, correct spelling, uppercase, thin stroke, not crowded against the badges and not clipped by the label edge.
5. Spacing: the badge row sits close under the date (about half the previous gap); the three maker lines are tightly spaced; the last line keeps a safe ~5% clearance above the bottom edge; element sizes match the previous label — badges, icons and text are not scaled up.
6. Alignment: every element — name, pet, date, badge row, all three text lines — is centred on the label's vertical axis, with equal left and right margins.

Fix the base image by regenerating once if the name or date is wrong (quote the wrong text explicitly). Fix badge/text problems by adjusting the script parameters — never by regenerating, because the model will not reproduce exact strokes.

## Output

Deliver the final label path. Add one short line: the background is a knit texture rather than a truly transparent alpha — offer a regenerate or a de-background pass if a transparent PNG is needed.

## Notes

- One image per request unless the user asks for variants.
- Keep the icon row monochrome black; do not tint icons with `{COLOR}`.
- `label_overlay.ps1` (next to this file) currently knows three badge drawings: chocolate, bread, bath. If a trait needs another object, add a `Draw-<Kind>` function to the script first, then reference it in `-Badges`; do not fall back to generating icons with the model.
- If the pet photo is not a pet (toy, logo, object), treat the main subject the same way — it becomes the embroidery motif.
- If Step 1 drifts despite the reference, regenerate with a tighter anchor description. Do not touch the negative prompt lists — they are hand-tuned.
