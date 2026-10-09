/*
  Pigment design-language stylesheet.

  This skill bundles no font and names no family, by design. --font-sans and
  --font-mono resolve to the platform generics, so every viewer sees the same
  rendering. Do not add an @font-face rule, a webfont URL, or a named family
  to either variable.
*/

:root {
  /* Colors — neutral scale */
  --color-white: #ffffff;
  --color-grey-10: #f7f7f8;
  --color-grey-20: #e8eaed;
  --color-grey-30: #949fb2;
  --color-grey-50: #5a657a;
  --color-grey-90: #020d23;
  --color-neutral-alpha: rgba(2, 13, 35, 0.08);
  --color-background-alpha: rgba(2, 13, 35, 0.04);
  --color-light-alpha: rgba(255, 255, 255, 0.64);

  /* Colors — brand */
  --color-primary-10: #f0f5ff;
  --color-primary-20: #95b9ff;
  --color-primary-30: #2684ff;
  --color-primary-50: #0355f3;
  --color-primary-90: #0038a4;
  --color-primary-light-transparent: rgba(3, 85, 243, 0.05);

  /* Colors — semantic */
  --color-positive-10: #eefdee;
  --color-positive-20: #d1fad1;
  --color-positive-50: #1a9f1a;
  --color-positive-90: #094f09;
  --color-cautious-10: #fff8e5;
  --color-cautious-20: #fff5cc;
  --color-cautious-50: #f3b921;
  --color-cautious-90: #7a5d11;
  --color-negative-10: #fff0f2;
  --color-negative-20: #ffd8dd;
  --color-negative-50: #d02b41;
  --color-negative-90: #740c19;
  --color-backdrop: rgba(2, 13, 35, 0.6);

  /* Colors — illustrative palette */
  --color-cobalt-soft: hsl(212, 100%, 85%);
  --color-cobalt-vivid: hsl(219, 98%, 48%);
  --color-cobalt-bold: hsl(227, 72%, 29%);
  --color-emerald-soft: hsl(80, 66%, 88%);
  --color-emerald-vivid: hsl(147, 99%, 33%);
  --color-emerald-bold: hsl(147, 56%, 26%);
  --color-amethyst-soft: hsl(274, 100%, 90%);
  --color-amethyst-vivid: hsl(259, 100%, 77%);
  --color-amethyst-bold: hsl(270, 50%, 30%);
  --color-ochre-soft: hsl(47, 90%, 84%);
  --color-ochre-vivid: hsl(43, 100%, 70%);
  --color-ochre-bold: hsl(43, 100%, 16%);
  --color-sienna-soft: hsl(32, 90%, 84%);
  --color-sienna-vivid: hsl(36, 100%, 70%);
  --color-sienna-bold: hsl(19, 83%, 28%);
  --color-turquoise-soft: hsl(191, 100%, 85%);
  --color-turquoise-vivid: hsl(185, 100%, 40%);
  --color-turquoise-bold: hsl(190, 100%, 19%);
  --color-fuchsia-soft: hsl(328, 100%, 90%);
  --color-fuchsia-vivid: hsl(345, 87%, 63%);
  --color-fuchsia-bold: hsl(328, 89%, 24%);

  /* Colors — categorical / random-assignment palette (lists of entities: apps, boards, scenarios...) */
  --color-categorical-1-fg: #013496;
  --color-categorical-1-bg: rgba(3, 85, 243, 0.24);
  --color-categorical-2-fg: #0a5442;
  --color-categorical-2-bg: rgba(20, 184, 146, 0.24);
  --color-categorical-3-fg: #32562f;
  --color-categorical-3-bg: rgba(116, 199, 109, 0.24);
  --color-categorical-4-fg: #414a75;
  --color-categorical-4-bg: rgba(143, 161, 255, 0.24);
  --color-categorical-5-fg: #4a00a2;
  --color-categorical-5-bg: rgba(87, 0, 191, 0.24);
  --color-categorical-6-fg: #134289;
  --color-categorical-6-bg: rgba(33, 115, 239, 0.24);
  --color-categorical-7-fg: #001eb9;
  --color-categorical-7-bg: rgba(0, 41, 255, 0.24);
  --color-categorical-8-fg: #354156;
  --color-categorical-8-bg: rgba(69, 84, 111, 0.24);
  --color-categorical-9-fg: #634f32;
  --color-categorical-9-bg: rgba(255, 204, 128, 0.24);
  --color-categorical-10-fg: #004871;
  --color-categorical-10-bg: rgba(0, 128, 200, 0.24);
  --color-categorical-11-fg: #2c5267;
  --color-categorical-11-bg: rgba(103, 191, 239, 0.24);
  --color-categorical-12-fg: #354d72;
  --color-categorical-12-bg: rgba(118, 173, 255, 0.24);
  --color-categorical-13-fg: #003293;
  --color-categorical-13-bg: rgba(0, 52, 154, 0.24);
  --color-categorical-14-fg: #0c5513;
  --color-categorical-14-bg: rgba(25, 183, 41, 0.24);
  --color-categorical-15-fg: #3a2e84;
  --color-categorical-15-bg: rgba(60, 48, 137, 0.24);
  --color-categorical-16-fg: #7b0344;
  --color-categorical-16-bg: rgba(215, 5, 118, 0.24);
  --color-categorical-17-fg: #772d48;
  --color-categorical-17-bg: rgba(240, 91, 145, 0.24);
  --color-categorical-18-fg: #761f54;
  --color-categorical-18-bg: rgba(206, 55, 146, 0.24);
  --color-categorical-19-fg: #6e4223;
  --color-categorical-19-bg: rgba(252, 151, 79, 0.24);
  --color-categorical-20-fg: #004631;
  --color-categorical-20-bg: rgba(0, 76, 53, 0.24);
  --color-categorical-21-fg: #0d4a34;
  --color-categorical-21-bg: rgba(18, 104, 73, 0.24);
  --color-categorical-22-fg: #615101;
  --color-categorical-22-bg: rgba(255, 215, 3, 0.24);
  --color-categorical-23-fg: #001391;
  --color-categorical-23-bg: rgba(0, 19, 145, 0.24);

  /* Colors — chart palette (ordered categorical series for data visualization) */
  --color-chart-1: #0355f3;
  --color-chart-2: #4bc766;
  --color-chart-3: #ffbe5c;
  --color-chart-4: #f95a77;
  --color-chart-5: #6b1cb0;
  --color-chart-6: #1b2970;
  --color-chart-7: #c2dffa;
  --color-chart-8: #d0faca;
  --color-chart-9: #ffc8ac;
  --color-chart-10: #caeb9d;
  --color-chart-11: #c096e3;
  --color-chart-12: #fdaebc;

  /* Color aliases */
  --text-primary: var(--color-grey-90);
  --text-secondary: var(--color-grey-50);
  --text-disabled: var(--color-grey-30);
  --text-highlight: var(--color-primary-50);
  --text-light-primary: var(--color-white);
  --text-light-secondary: var(--color-light-alpha);
  --text-positive: var(--color-positive-50);
  --text-negative: var(--color-negative-50);
  --border: var(--color-neutral-alpha);
  --bg-secondary-surface: var(--color-background-alpha);
  --bg-primary-surface: var(--color-white);

  /* Typography */
  --font-sans: sans-serif;
  --font-mono: monospace;

  /* Radii */
  --radius-min: 1px;
  --radius-xxs: 2px;
  --radius-xs: 4px;
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-circular: 5000px;

  /* Shadows */
  --shadow-card:
    0 1px 1px 0 rgba(2, 13, 35, 0.04), 0 0 0 1px rgba(2, 13, 35, 0.06),
    0 2px 5px 0 rgba(2, 13, 35, 0.05);
  --shadow-card-hover:
    rgba(2, 13, 35, 0.08) 0px 2px 17px -1px, rgba(2, 13, 35, 0.06) 0px 0px 0px 1px,
    rgba(2, 13, 35, 0.04) 0px 1px 1px 0px;
  --shadow-floating-container:
    0px 0px 0px 1px hsl(220deg 89.2% 7.25% / 5%), 0px 6px 10px -6px hsl(220deg 89.2% 7.25% / 12%),
    0px 6px 20px 0px hsl(220deg 89.2% 7.25% / 16%);
  --shadow-floating-side-panel:
    0px 0px 0px 1px hsl(220deg 89.2% 7.25% / 5%), 0px 4px 24px -4px hsl(220deg 89.2% 7.25% / 8%);
  --shadow-modal: hsl(226.7deg 73% 7.25% / 25%) 0px 2px 6px;
  --shadow-dragged-item:
    hsl(226.7deg 73% 7.25% / 20%) 0px 6px 20px -4px, hsl(226.7deg 73% 7.25% / 4%) 0px 0px 0px 1px;

  /* Motion */
  --motion-snappy: 150ms ease-in;
  --motion-soft: 200ms ease-in-out;
  --focus-ring: 0 0 0 3px var(--color-primary-20);

  /* Spacing — 4px grid */
  --space-0-5: 2px;
  --space-1: 4px;
  --space-1-5: 6px;
  --space-2: 8px;
  --space-2-5: 10px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-7: 28px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Breakpoints */
  --breakpoint-xs: 0px;
  --breakpoint-sm: 600px;
  --breakpoint-md: 960px;
  --breakpoint-lg: 1280px;
  --breakpoint-xl: 1440px;
}

