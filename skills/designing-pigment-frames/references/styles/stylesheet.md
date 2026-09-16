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

/* Type Scale */
.type-marketing-title {
  font-family: var(--font-sans);
  font-size: 56px;
  font-weight: 500;
  line-height: 68px;
  letter-spacing: -1px;
  color: var(--text-primary);
  margin: 0;
}
.type-screen-title {
  font-family: var(--font-sans);
  font-size: 24px;
  font-weight: 500;
  line-height: 32px;
  color: var(--text-primary);
  margin: 0;
}
.type-content-title {
  font-family: var(--font-sans);
  font-size: 22px;
  font-weight: 500;
  line-height: 32px;
  color: var(--text-primary);
  margin: 0;
}
.type-section-title {
  font-family: var(--font-sans);
  font-size: 16px;
  font-weight: 600;
  line-height: 24px;
  color: var(--text-primary);
  margin: 0;
}
.type-kicker,
.type-column-title {
  font-family: var(--font-sans);
  font-size: 11px;
  font-weight: 400;
  line-height: 16px;
  letter-spacing: 0.6px;
  text-transform: uppercase;
  color: var(--text-secondary);
}
.type-highlight-figure {
  font-family: var(--font-sans);
  font-size: 28px;
  font-weight: 500;
  line-height: 36px;
  color: var(--text-primary);
  margin: 0;
}
.type-running-regular {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
  color: var(--text-primary);
  margin: 0;
}
.type-running-heavy,
.text-input input,
.text-input select {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-primary);
  margin: 0;
}
.type-running-small {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}
.type-running-small-heavy {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}
.type-fieldset-regular,
.field-label {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-secondary);
}
.type-fieldset-small {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: var(--text-secondary);
}
.type-button-label,
.button,
.button-medium,
.button-small {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  white-space: nowrap;
  color: inherit;
}
.type-legal-notice {
  font-family: var(--font-sans);
  font-size: 10px;
  font-weight: 400;
  line-height: 12px;
  color: var(--text-secondary);
  margin: 0;
}
.type-code-running-regular {
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
  color: var(--text-primary);
}
.type-code-running-small {
  font-family: var(--font-mono);
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-primary);
}
.type-chip {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 400;
  line-height: 12px;
  color: var(--text-secondary);
  margin: 0;
}
.type-chart-label {
  font-family: var(--font-sans);
  font-size: 11px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}

/* Font helpers */
.anchor-link {
  font-family: var(--font-sans);
  font-weight: 400;
  color: var(--text-highlight);
  text-decoration: underline;
  text-underline-offset: var(--space-0-5);
  text-decoration-thickness: 1px;
  width: fit-content;
}

/* Buttons */
.button,
.icon-button,
.toggle-button,
.chip.is-clickable {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-1);
  border: none;
  border-radius: var(--radius-xs);
  box-shadow: inset 0 0 0 2px var(--button-border-color, transparent);
  cursor: pointer;
  transition:
    background-color var(--motion-snappy),
    box-shadow var(--motion-snappy),
    color var(--motion-snappy);
}
.button svg {
  width: 16px;
  height: 16px;
}
.button:focus-visible {
  outline: none;
  box-shadow:
    inset 0 0 0 2px var(--button-border-color, transparent),
    var(--focus-ring);
}
.button-medium {
  padding: var(--space-1-5) var(--space-3);
}
.button-small {
  padding: var(--space-1) var(--space-2-5);
}
.button-primary {
  background: var(--color-primary-50);
  --button-border-color: var(--color-primary-50);
  color: var(--text-light-primary);
}
.button-primary:hover {
  background: var(--color-primary-90);
  --button-border-color: var(--color-primary-90);
}
.button-secondary {
  background: var(--color-primary-light-transparent);
  --button-border-color: transparent;
  color: var(--color-primary-50);
}
.button-secondary:hover {
  --button-border-color: var(--color-primary-light-transparent);
}
.button-ghost {
  background: transparent;
  --button-border-color: transparent;
  color: var(--color-primary-50);
}
.button-ghost:hover {
  background: var(--color-primary-light-transparent);
}
.button-ghost-grey {
  background: transparent;
  --button-border-color: transparent;
  color: var(--text-secondary);
}
.button-ghost-grey:hover {
  background: var(--color-background-alpha);
  color: var(--text-primary);
}
.button-danger-primary {
  background: var(--color-negative-50);
  --button-border-color: var(--color-negative-50);
  color: var(--text-light-primary);
}
.button-danger-primary:hover {
  background: var(--color-negative-90);
  --button-border-color: var(--color-negative-90);
}
.button-danger-secondary {
  background: var(--color-white);
  --button-border-color: var(--color-white);
  color: var(--color-negative-50);
}
.button-danger-secondary:hover {
  background: var(--color-negative-10);
  --button-border-color: var(--color-negative-10);
}
.button-transparent {
  background: rgba(255, 255, 255, 0.16);
  --button-border-color: transparent;
  color: var(--color-white);
}
.button-transparent:hover {
  background: rgba(255, 255, 255, 0.24);
}
.button[disabled],
.icon-button[disabled] {
  color: var(--text-disabled);
  cursor: not-allowed;
  pointer-events: none;
}
.button-primary[disabled],
.button-secondary[disabled],
.button-transparent[disabled] {
  background: var(--color-grey-10);
  --button-border-color: var(--color-grey-10);
}
.button-ghost[disabled],
.button-ghost-grey[disabled],
.button-danger-primary[disabled],
.button-danger-secondary[disabled] {
  background: transparent;
  --button-border-color: transparent;
}

/* Icon Button */
.icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border: none;
  border-radius: var(--radius-xs);
  box-shadow: inset 0 0 0 2px var(--button-border-color, transparent);
  cursor: pointer;
  transition:
    background-color var(--motion-snappy),
    box-shadow var(--motion-snappy),
    color var(--motion-snappy);
}
.icon-button:focus-visible {
  outline: none;
  box-shadow:
    inset 0 0 0 2px var(--button-border-color, transparent),
    var(--focus-ring);
}
.icon-button svg,
.toggle-button svg {
  width: 16px;
  height: 16px;
}
.icon-button-medium {
  padding: var(--space-2);
}
.icon-button-small {
  padding: var(--space-1-5);
}

/* Button Groups */
.button-group {
  display: flex;
  flex-direction: row;
  gap: var(--space-1);
  margin: 0;
  border: none;
  padding-inline: 0;
  padding-block: 0;
  padding: 0;
}
.toggle-button input {
  position: absolute;
  left: -100px;
  opacity: 0;
}
.toggle-button {
  color: var(--text-secondary);
  background: transparent;
  box-shadow: inset 0 0 0 1px var(--color-neutral-alpha, grey);
  border-radius: var(--radius-circular);
  padding-inline: var(--space-4);
}
.toggle-button:hover {
  color: var(--text-primary);
  background: var(--color-background-alpha);
  box-shadow: none;
}
.toggle-button:has(input:checked) {
  background: var(--color-primary-light-transparent);
  color: var(--text-highlight);
  box-shadow: none;
}
.toggle-button:has(input:checked):hover {
  box-shadow: inset 0 0 0 2px var(--color-primary-light-transparent, blue);
}
.toggle-button:has(input:focus-visible) {
  outline: none;
  box-shadow: var(--focus-ring);
}

/* Headers */
.page-header,
.section-header,
.table-header,
.filter-header {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-8);
}
.page-header-text,
.section-header-text,
.table-header-text,
.filter-header-primary-actions {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.page-header-actions,
.section-header-actions,
.table-header-actions,
.filter-header-secondary-actions {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  flex-shrink: 0;
}
.divider {
  width: 100%;
  height: 1px;
  border: none;
  margin: var(--space-6) 0 0;
  background: var(--border);
}

/* Fieldset / Text Input */
.field {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}
.text-input {
  display: flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--space-1-5) var(--space-2);
  border-radius: var(--radius-xs);
  background: var(--color-white);
  box-shadow: inset 0 0 0 1px var(--input-border-color, var(--color-grey-20));
  transition: box-shadow var(--motion-snappy);
  position: relative;
}
.text-input:hover {
  --input-border-color: var(--color-primary-50);
}
.text-input:focus-within {
  box-shadow:
    inset 0 0 0 1px var(--color-primary-30),
    var(--focus-ring);
}
.text-input.is-error {
  --input-border-color: var(--color-negative-50);
}
.text-input.is-error:hover {
  --input-border-color: var(--color-negative-90);
}
.text-input.is-disabled {
  background: var(--color-grey-10);
  --input-border-color: var(--color-grey-20);
}
.text-input input,
.text-input select {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  background: transparent;
  appearance: none;
  -webkit-appearance: none;
  padding: 0;
}
.text-input select {
  width: 100%;
  padding-right: calc(var(--space-2) + 16px);
  cursor: pointer;
}
.text-input input::placeholder {
  color: var(--text-secondary);
  font-weight: 500;
}
.text-input input:disabled,
.text-input select:disabled {
  color: var(--text-disabled);
  cursor: not-allowed;
}
.text-input svg {
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  color: var(--text-secondary);
}
.text-input select ~ svg {
  position: absolute;
  right: var(--space-2);
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}
.text-input.is-disabled svg {
  color: var(--text-disabled);
}

/* Cards */
.card-grid {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--space-4);
}
.card {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
  box-sizing: border-box;
  padding: var(--space-4);
  background: var(--color-white);
  border-radius: var(--radius-xs);
}
.card-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--space-12);
  height: var(--space-12);
  background: var(--color-primary-light-transparent);
  color: var(--text-highlight);
  border-radius: var(--radius-xs);
}
.card-icon svg {
  width: var(--space-4);
  height: var(--space-4);
}
.card-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-2);
}
.card-grouped-text {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}
.card-grouped-figure-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.card-footer {
  margin-top: auto;
  padding-top: var(--space-3);
  display: flex;
  align-items: center;
  gap: var(--space-2);
}

.card.is-outlined {
  border: 1px solid var(--border);
}
.card.is-elevated {
  box-shadow: var(--shadow-card);
}
.card.is-filled {
  background: var(--bg-secondary-surface);
}
.card.is-clickable {
  cursor: pointer;
}
.card.is-elevated.is-clickable:hover {
  box-shadow: var(--shadow-card-hover);
}
.card.is-filled .card-icon {
  background: var(--color-background-alpha);
  color: var(--text-primary);
}

.card.has-gradient,
.card.is-filled.has-gradient,
.card.is-elevated.has-gradient {
  --card-base-background-color: var(--color-cobalt-soft);
  --card-glow-color-left: var(--color-amethyst-soft);
  --card-glow-color-right: var(--color-amethyst-soft);
  background-color: var(--card-base-background-color);
  background-image:
    linear-gradient(6deg, rgba(255, 255, 255, 0.8) 20%, transparent 80%),
    linear-gradient(12deg, var(--card-base-background-color) 0%, transparent 100%),
    radial-gradient(1000% 100% at 0% 100%, var(--card-base-background-color), transparent),
    radial-gradient(356px circle at 120% -20%, var(--card-glow-color-right), transparent),
    radial-gradient(128px circle at 0 -10%, var(--card-glow-color-left), transparent);
}

/* Avatars */
.avatar {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--space-6);
  height: var(--space-6);
  border-radius: var(--radius-circular);
  color: var(--text-light-primary);
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
}
.avatar-large {
  width: var(--space-8);
  height: var(--space-8);
}

/* Lists */
.list {
  width: 100%;
  display: flex;
  flex-direction: column;
  list-style: none;
  margin: 0;
  padding: 0;
}

.list-item {
  display: flex;
  align-items: center;
  gap: var(--space-2-5);
  padding: 0 var(--space-2-5);
  box-sizing: border-box;
  transition: background-color var(--motion-snappy);
}
.list-item .list-item-content {
  flex: 1;
  display: flex;
  flex-direction: row;
  gap: var(--space-2);
  padding: var(--space-2) 0;
}
.list-item .list-item-decoration {
  min-width: var(--space-8);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
}
.list-item .list-item-icon-container {
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-background-alpha);
  width: var(--space-8);
  height: var(--space-8);
  border-radius: var(--radius-sm);
}
.list-item .list-item-decoration svg {
  width: 16px;
  height: 16px;
}
.list-item .list-item-primary {
  flex: 1;
}
.list-item .list-item-secondary {
  display: flex;
  align-items: center;
  gap: var(--space-1);
  flex-shrink: 0;
}
.list-item.has-divider {
  border-bottom: 1px solid var(--border);
}
.list-item.has-outline {
  border: 1px solid var(--border);
  margin-bottom: var(--space-2);
  border-radius: var(--radius-sm);
}
.list-item.is-elevated {
  box-shadow: var(--shadow-card);
  background: var(--color-white);
  margin-bottom: var(--space-2);
  border-radius: var(--radius-sm);
}
.list-item.is-clickable {
  cursor: pointer;
}
.list-item.is-clickable:hover,
.list-item.is-clickable.is-elevated:hover,
.list-item.is-clickable.has-outline:hover {
  background: var(--color-background-alpha);
}
.list-surface {
  box-sizing: border-box;
  width: 100%;
  padding: var(--space-1) var(--space-2);
  background: var(--color-grey-10);
  border-radius: var(--radius-xs);
}

/* Tooltips */
.tooltip {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  background: var(--color-grey-90);
  padding: var(--space-2-5);
  border-radius: var(--radius-xs);
  box-shadow: var(--shadow-floating-container);
  max-width: 420px;
}
.tooltip-grouped-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.tooltip-list {
  display: grid;
  grid-template-columns: auto 1fr;
  grid-column-gap: var(--space-4);
  grid-row-gap: var(--space-1);
  margin: 0;
}
.tooltip .divider {
  background-color: rgba(255, 255, 255, 0.24);
  margin: 0;
}
.tooltip-list dt {
  color: var(--text-light-secondary);
  overflow: hidden;
  text-overflow: ellipsis;
  text-wrap: nowrap;
}
.tooltip-list dt .series-dot {
  display: inline-block;
  width: var(--space-1-5);
  height: var(--space-1-5);
  border-radius: var(--radius-circular);
  background: var(--color-background-alpha);
  border: 1px solid var(--border);
  margin-right: var(--space-1);
}
.tooltip-list dd {
  color: var(--text-light-primary);
}

/* Chips */
.chip {
  display: inline-flex;
  align-items: center;
  flex-shrink: 0;
  gap: var(--space-1);
  height: 20px;
  padding: var(--space-0-5) var(--space-2);
  border-radius: var(--radius-circular);
  background: var(--color-background-alpha);
  box-sizing: border-box;
  width: fit-content;
}
.chip svg {
  width: 12px;
  height: 12px;
  flex-shrink: 0;
}
.chip-dot {
  height: var(--space-1-5);
  width: var(--space-1-5);
  border-radius: var(--radius-circular);
  background-color: var(--color-background-alpha);
  border: 1px solid var(--border);
}
.chip-row {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
}

/* Gradient background */
.gradient-background {
  --gradient-depth: 128px;
  --gradient-glow-1: var(--color-turquoise-soft);
  --gradient-glow-2: var(--color-ochre-soft);
  --gradient-glow-3: var(--color-cobalt-soft);
  background-color: var(--color-white);
  background-image:
    linear-gradient(rgba(255, 255, 255, 0) 0%, var(--color-white) var(--gradient-depth)),
    linear-gradient(rgba(255, 255, 255, 0) 0%, var(--color-white) calc(var(--gradient-depth) - 28px)),
    linear-gradient(rgba(255, 255, 255, 0) 0%, var(--color-white) calc(var(--gradient-depth) - 36px)),
    radial-gradient(circle calc(var(--gradient-depth) * 2) at 0 0, var(--gradient-glow-1), transparent),
    radial-gradient(circle calc(var(--gradient-depth) * 2) at 40% 0, var(--gradient-glow-2), transparent),
    radial-gradient(
      circle closest-corner at 20% calc(var(--gradient-depth) / 2),
      var(--gradient-glow-3) 0%,
      transparent 78%
    );
}
