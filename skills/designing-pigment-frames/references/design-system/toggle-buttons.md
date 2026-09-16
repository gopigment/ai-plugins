# Button Groups / Toggle Buttons

A row of pill buttons that hold a pressed state, used instead of a vertical checkbox or radio list
when the options read better as compact pills. Use a **radio** group for a single choice among 2-5
inline options (e.g. a time range), a **checkbox** group for any number of independent choices (e.g.
filter pills).

Every toggle button is `button-medium` (32px). There is no small size and no separate pill opt-in —
the fully circular shape is the default `.toggle-button` shape, inherited from the same base rule as
Button and Icon Button, which is also where the `--motion-snappy` transition on state changes comes
from.

`.button-group` is a plain row with a 4px gap and no border or margin of its own.

### TG0 · Shared base

Built using a pill-shaped `<label class="toggle-button button-medium">` wrapping a visually-hidden (not `display: none`, so it stays focusable) radio or checkbox input, grouped inside a `<fieldset class="button-group">`.

```css
.button-group {
  display: flex;
  flex-direction: row;
  gap: var(--space-1);
  margin: 0;
  border: none;
}
.toggle-button input {
  position: absolute;
  left: -100px;
  opacity: 0;
}
.toggle-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-1);
  color: var(--text-secondary);
  background: transparent;
  box-shadow: inset 0 0 0 1px var(--color-neutral-alpha);
  border-radius: var(--radius-circular);
  padding-inline: var(--space-4);
  cursor: pointer;
  transition:
    background-color var(--motion-snappy),
    box-shadow var(--motion-snappy),
    color var(--motion-snappy);
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
  box-shadow: inset 0 0 0 2px var(--color-primary-light-transparent);
}
.toggle-button:has(input:focus-visible) {
  outline: none;
  box-shadow: var(--focus-ring);
}
```

### TG1 · Single select

Radio inputs sharing one `name`: exactly one segment can be active. Use for "pick one of 2-5" mutually-exclusive choices like a time range (eg: last week / last month / all time).

```html
<fieldset class="button-group">
  <label role="radio" class="toggle-button button-medium">
    <input type="radio" id="now" name="schedule" value="now" checked />
    Now
  </label>
  <label role="radio" class="toggle-button button-medium">
    <input type="radio" id="5m" name="schedule" value="5m" />
    In 5 min
  </label>
  <label role="radio" class="toggle-button button-medium">
    <input type="radio" id="15m" name="schedule" value="15m" />
    In 15 min
  </label>
  <label role="radio" class="toggle-button button-medium">
    <input type="radio" id="later" name="schedule" value="later" />
    Later
  </label>
</fieldset>
```

### TG2 · Multi select

Checkbox inputs with independent state: any number of segments can be active at once. Use for "pick any of N" complementary choices like a region filter (EMEA / APAC / AMER).

```html
<fieldset class="button-group">
  <label role="checkbox" class="toggle-button button-medium">
    <input type="checkbox" id="ch" name="cuisine" value="ch" checked />
    Chinese
  </label>
  <label role="checkbox" class="toggle-button button-medium">
    <input type="checkbox" id="fr" name="cuisine" value="fr" />
    French
  </label>
  <label role="checkbox" class="toggle-button button-medium">
    <input type="checkbox" id="it" name="cuisine" value="it" />
    Italian
  </label>
  <label role="checkbox" class="toggle-button button-medium">
    <input type="checkbox" id="mx" name="cuisine" value="mx" />
    Mexican
  </label>
</fieldset>
```
