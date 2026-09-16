# Fieldsets & Inputs

Covers the Fieldset wrapper, Text Input, Text Area, Search Input and Select Input.

**Fieldset anatomy.** Every form field is the same stack: a label, a 4px gap, the control, a 4px gap,
then optional helper or error text.

- Label — `fieldset-regular` (or `fieldset-small` in dense forms), `--text-secondary`. It switches to
  `--text-negative` on error and `--text-disabled` when disabled.
- Value — `running-heavy` (or `running-small-heavy` in dense forms), `--text-primary` when filled.
- Placeholder — the same `running-heavy` weight as a filled value, in `--text-secondary`. Only the
  color separates a placeholder from a value, never the weight.
- Helper text — `running-small` in `--text-secondary`, or `--text-negative` for a validation error.
- When space is tight, replace inline helper text with a 16px info icon next to the label that opens
  a [tooltip](tooltips.md).

**Sizing.** A medium input is 32px tall, which falls out of 6px vertical padding plus the 20px
`running-heavy` line-height. The platform also has a 28px small input; only the medium size has a
recipe below. Radius is `--radius-xs`. Inline icons are 16px in `--text-secondary`, dropping to
`--text-disabled` when disabled.

Inputs are the one component whose default border is `--color-grey-20` rather than the structural
`--border`.

### IN0 · Shared base

`.field` wraps a label and a `.text-input` container; the container (not the `<input>`) owns the border/background/focus styling.

```css
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
.text-input input,
.text-input select {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  background: transparent;
  appearance: none;
  padding: 0;
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-primary);
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
```

### IN1 · Text input

Add `.is-error` and `.is-disabled` classes to apply the visual style for inputs in error and disabled state, respectively

```html
<div class="field" style="width: 240px">
  <label class="field-label">Scenario name</label>
  <div class="text-input">
    <input type="text" placeholder="e.g. FY26 Budget" />
  </div>
</div>

<div class="field" style="width: 240px">
  <label class="field-label">Scenario name</label>
  <div class="text-input is-error">
    <input type="text" value="FY26 Budget!!" />
  </div>
</div>

<div class="field" style="width: 240px">
  <label class="field-label">Scenario name</label>
  <div class="text-input is-disabled">
    <input type="text" value="FY26 Budget" disabled />
  </div>
</div>
```

```css
.field-label {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-secondary);
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
.text-input.is-disabled svg {
  color: var(--text-disabled);
}
```

### IN2 · Search input

Has a leading search icon, no field label.

```html
<div class="text-input" style="width: 240px">
  <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
    <circle cx="7" cy="7" r="5" stroke="currentColor" stroke-width="1.5" />
    <path d="M11 11L14 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
  </svg>
  <input type="text" placeholder="Search scenarios" />
</div>
```

### IN3 · Select input

A native `<select>` styled to match `.text-input`, with a trailing chevron.

Reserve this native recipe for trivial, low-stakes choices — a 2-3 item picker in a dense form with
many fields. For anything more prominent, such as the main filters on a dashboard, build a **custom
dropdown** instead: keep this `.text-input`-styled trigger, then render the option list as a popover
with `--shadow-floating-container` and `--radius-xs`. Options take a `--color-background-alpha`
background with `--text-highlight` text on hover, and `--color-primary-light-transparent` with
`--text-highlight` for the current value. Keep the native keyboard behaviour — arrow keys, Escape to
close, Enter to select. A dropdown that only works with a mouse is not accessible.

```html
<div class="field" style="width: 240px">
  <label class="field-label">Currency</label>
  <div class="text-input">
    <select>
      <option>USD</option>
      <option>EUR</option>
      <option>GBP</option>
    </select>
    <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path
        d="M4 6l4 4 4-4"
        stroke="currentColor"
        stroke-width="1.5"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
    </svg>
  </div>
</div>
```

```css
.text-input select {
  width: 100%;
  padding-right: calc(var(--space-2) + 16px);
  cursor: pointer;
}
.text-input select ~ svg {
  position: absolute;
  right: var(--space-2);
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}
```
