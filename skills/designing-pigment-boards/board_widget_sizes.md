# Widget Sizing Guidelines

These guidelines give the heights to respect when creating Widgets.
You MUST follow them.

**General rule:** When a data widget has a title, add 1 to the minimum height compared to widgets without titles.

**Text Widgets**

**1. Section titles (default; use this first for typical boards)**

For a **section title** text widget - short H2 text (and optional one-line **Normal** subtitle), **not** long instructional body copy - set height as follows **at full board width** (`width` = **12**):

- **Title only:** `height` = **2**
- **Title + subtitle (short):** `height` = **3**

**2. All other text widgets (long, mixed, or non–section-title copy)**

When the content is not covered by **§1** (e.g. instructions, long body text, or several paragraphs), use the **word-count** method (steps A–D). This avoids scrollbars for wrapped text.

*Full width (`width` = 12):*

- **A. Words:** A *word* is a segment separated by whitespace. For each *text* node in `editor_value` (or each paragraph’s children, grouped by `font_size` if several runs share a style), add up words for that `font_size` (Normal, H2, H3).
- **B. Visual lines (one divisor per `font_size`; weighting is built in here):** A **larger** on-screen style fits **fewer** words per line, so it uses a **smaller** divisor `k` and **more** lines for the same `w`. For each group, add `**ceil(w / k)`** with:
  - **H2** → `k` = 17
  - **H3** → `k` = 27
  - **Normal** → `k` = 32  
  Order `k_H2 < k_H3 ≤ k_Normal` (here 17 < 27 < 32). `w` = word count for that group. Adjust `k` only to fix clear bias, not the formula.
- **C. Sum:** `L` = total from B, summed over all groups. If there are **3+** top-level paragraphs, add **+1** to `L` for each from the **3rd** onward; with 1–2 paragraphs, add 0.
- **D. Height:** `height` = `max(2, min(24, L))`. If a widget still scrolls, add 1 to `height` or lower the relevant `k` slightly.

**Non-full width (applies to heights from either §1 or §2, computed as if at `width` = 12):** If the widget’s grid `width` is `n` with **1 < n < 12**, the same text wraps onto more lines. Multiply that `height` by **`ceil(12 / n)`** (i.e. **12 / n** rounded **up** to a whole number). *Example: `n` = 6 → factor `ceil(12/6)` = 2.*

**Spacers Widgets:**

- Spacer widget (between sections): height = 1

**KPI Widgets:**

- With widget title: height = 5-7
- Without widget title: height = 4-6

**Chart Widgets:**

- With widget title: height = 12-18
- Without widget title: height = 11-18
- Note: Actual height depends on chart type, data points, legends, Y-axis labels, and complexity

**Grid Widgets:**

- With widget title: height = 12-24
- Without widget title: height = 11-24
- Note: Actual height depends on number of rows, column count, and need to minimize scrolling