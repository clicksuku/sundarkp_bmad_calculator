---
name: CalcHub
description: A colorful, friendly web calculator where every key family has its own color, in matching light and dark themes.
status: final
created: 2026-10-21
updated: 2026-10-21
sources:
  - ../../prds/prd-simpleMathcalc-2026-10-19/prd.md
  - ../../architecture/architecture-simpleMathcalc-2026-10-21/ARCHITECTURE-SPINE.md
  - imports/ref-1-light-simple-keypad.png
  - imports/ref-2-dark-scientific-keypad.png
colors:
  # Light theme (default)
  page: '#F4F6FB'
  surface: '#FFFFFF'
  display: '#FFFFFF'
  border: '#D5DAE8'
  text: '#1B2033'
  text-muted: '#4B5470'
  key-digit: '#E6E9F5'
  key-digit-text: '#1B2033'
  key-operator: '#4F46E5'
  key-operator-text: '#FFFFFF'
  key-function: '#D3ECFA'
  key-function-text: '#0A4F78'
  key-memory: '#D2F2E9'
  key-memory-text: '#0B5A4A'
  key-clear: '#FFE0DB'
  key-clear-text: '#9B1C13'
  key-equals: '#15803D'
  key-equals-text: '#FFFFFF'
  focus-ring: '#4F46E5'
  error: '#B42318'
  phase-analysis: '#FFEBC8'
  phase-analysis-text: '#7A3E00'
  phase-planning: '#D3ECFA'
  phase-planning-text: '#0A4F78'
  phase-solutioning: '#EBDDFF'
  phase-solutioning-text: '#5B21B6'
  phase-implementation: '#D1F3DD'
  phase-implementation-text: '#0F5A2E'
  # Dark theme
  page-dark: '#0F1220'
  surface-dark: '#181C30'
  display-dark: '#0A0D19'
  border-dark: '#2F3558'
  text-dark: '#EEF0FA'
  text-muted-dark: '#A6ADCB'
  key-digit-dark: '#2A2F4C'
  key-digit-text-dark: '#EEF0FA'
  key-operator-dark: '#9B9BFF'
  key-operator-text-dark: '#0F1220'
  key-function-dark: '#1E3B55'
  key-function-text-dark: '#9BDAFF'
  key-memory-dark: '#134238'
  key-memory-text-dark: '#86E8D2'
  key-clear-dark: '#53222A'
  key-clear-text-dark: '#FFA79D'
  key-equals-dark: '#34D27B'
  key-equals-text-dark: '#06220F'
  focus-ring-dark: '#9B9BFF'
  error-dark: '#FF8C80'
  phase-analysis-dark: '#4A3210'
  phase-analysis-text-dark: '#FFD08A'
  phase-planning-dark: '#1E3B55'
  phase-planning-text-dark: '#9BDAFF'
  phase-solutioning-dark: '#3A2670'
  phase-solutioning-text-dark: '#D4B8FF'
  phase-implementation-dark: '#14452A'
  phase-implementation-text-dark: '#8EF0B4'
typography:
  display-result:
    fontFamily: 'JetBrains Mono, ui-monospace, SFMono-Regular, Menlo, monospace'
    fontSize: 40px
    fontWeight: 500
    lineHeight: 48px
  display-expression:
    fontFamily: 'JetBrains Mono, ui-monospace, SFMono-Regular, Menlo, monospace'
    fontSize: 20px
    fontWeight: 400
    lineHeight: 28px
  key-label:
    fontFamily: 'Inter, system-ui, -apple-system, Segoe UI, sans-serif'
    fontSize: 20px
    fontWeight: 600
    lineHeight: 24px
  key-label-small:
    fontFamily: 'Inter, system-ui, -apple-system, Segoe UI, sans-serif'
    fontSize: 16px
    fontWeight: 600
    lineHeight: 20px
  body:
    fontFamily: 'Inter, system-ui, -apple-system, Segoe UI, sans-serif'
    fontSize: 16px
    fontWeight: 400
    lineHeight: 24px
  caption:
    fontFamily: 'Inter, system-ui, -apple-system, Segoe UI, sans-serif'
    fontSize: 13px
    fontWeight: 500
    lineHeight: 18px
  heading:
    fontFamily: 'Inter, system-ui, -apple-system, Segoe UI, sans-serif'
    fontSize: 18px
    fontWeight: 700
    lineHeight: 24px
rounded:
  sm: 6px
  md: 12px
  lg: 20px
  full: 9999px
spacing:
  '1': 4px
  '2': 8px
  '3': 12px
  '4': 16px
  '6': 24px
  '8': 32px
  key-gap: 8px
  key-min-size: 48px
  panel-gutter: 24px
components:
  display:
    backgroundColor: '{colors.display}'
    textColor: '{colors.text}'
    rounded: '{rounded.lg}'
    padding: '{spacing.4}'
  key-digit:
    backgroundColor: '{colors.key-digit}'
    textColor: '{colors.key-digit-text}'
    typography: '{typography.key-label}'
    rounded: '{rounded.md}'
    height: '{spacing.key-min-size}'
  key-operator:
    backgroundColor: '{colors.key-operator}'
    textColor: '{colors.key-operator-text}'
    typography: '{typography.key-label}'
    rounded: '{rounded.md}'
  key-function:
    backgroundColor: '{colors.key-function}'
    textColor: '{colors.key-function-text}'
    typography: '{typography.key-label-small}'
    rounded: '{rounded.md}'
  key-memory:
    backgroundColor: '{colors.key-memory}'
    textColor: '{colors.key-memory-text}'
    typography: '{typography.key-label-small}'
    rounded: '{rounded.md}'
  key-clear:
    backgroundColor: '{colors.key-clear}'
    textColor: '{colors.key-clear-text}'
    typography: '{typography.key-label}'
    rounded: '{rounded.md}'
  key-equals:
    backgroundColor: '{colors.key-equals}'
    textColor: '{colors.key-equals-text}'
    typography: '{typography.key-label}'
    rounded: '{rounded.md}'
  segmented-toggle:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.text}'
    rounded: '{rounded.full}'
  trace-phase-chip:
    typography: '{typography.caption}'
    rounded: '{rounded.full}'
  trace-panel:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.text}'
    rounded: '{rounded.lg}'
    padding: '{spacing.4}'
---

# CalcHub Design

## Brand & Style

CalcHub is a friendly, colorful calculator. It borrows the *structure* of two reference keypads: the light tabbed keypad with a segmented RAD/DEG toggle, and the dark scientific keypad with a roomy display and many function keys. It does not copy their monochrome look. Here color carries meaning: each family of keys has its own hue, so users find the right key by color before they read the label.

The tone is playful but calm. Surfaces are quiet and the keys are the color. It should feel like a well-made handheld calculator, not a toy and not a lab instrument. The Pipeline Trace Panel is the one place the app teaches, and it uses its own four phase colors. `[ASSUMPTION: playful-but-calm tone and the indigo/teal/sky/coral/green key palette were chosen by the designer, not specified by the user.]`

Two themes ship in Phase I, **Light** and **Dark**, plus **System**, which follows the OS (FR-27). Both themes use the same hues, and only lightness changes.

## Colors

Every key family has a background and a text token that pass WCAG AA together (all pairs are 5.0:1 or higher; the dark-theme pairs are 6.9:1 or higher).

| Family | Used for | Light | Dark |
| --- | --- | --- | --- |
| Digit (`key-digit`) | 0-9, decimal point, `+/-` | soft blue-grey | slate |
| Operator (`key-operator`) | `+ − × ÷ ^`, parentheses | indigo | periwinkle |
| Function (`key-function`) | sin, cos, tan, log, ln, √, x², 1/x, n!, abs, π, e | sky blue | deep blue |
| Memory (`key-memory`) | MC, MR, M+, M−, MS | teal | deep teal |
| Clear (`key-clear`) | C, CE, backspace | coral | deep coral |
| Equals (`key-equals`) | `=` only | green | bright green |

Rules:
- Green is reserved for `=` and for success states. Coral is reserved for clearing and destructive actions. Do not reuse either for decoration.
- Operator indigo is also the **focus ring** color (`focus-ring`); the focus ring must contrast 3:1 or more against both the key and the page.
- Error text uses `error`, always paired with an icon and words, never color alone.
- The four trace phase colors (`phase-analysis`, `phase-planning`, `phase-solutioning`, `phase-implementation`) are used only in the Trace Panel. Each phase always keeps its name beside its color.
- Dark theme uses the `*-dark` tokens. Never apply light tokens on a dark page, or the reverse.

## Typography

- **Display and expression** use a monospace face with tabular figures, so digits line up and long results do not jump as they change.
- **Keys and interface text** use a clean sans face. Key labels are semi-bold.
- Scale: display result 40 px, expression 20 px, key labels 20 px (16 px for function and memory keys), body 16 px, captions 13 px.
- Fonts load from the system stack first, so nothing blocks rendering. `[ASSUMPTION: Inter and JetBrains Mono as preferred faces, with the system stack as fallback.]`
- Exponents are shown as superscripts (`x²`). Fractions are shown inline as `a/b` in Phase I.

## Layout & Spacing

- **Desktop (≥ 1024 px):** a split pane (FR-33). Calculator on the left (fixed 360-420 px wide), Trace Panel on the right filling the remaining width. Top bar above both.
- **Tablet and phone (< 1024 px):** the calculator fills the width. The Trace Panel collapses into a bottom sheet opened from the top bar.
- Key grid: 8 px gap, keys at least 48 px square (44 px minimum touch target per WCAG). Simple view is 4 columns; Scientific view adds a function block of 4 columns to the left on desktop, or above the number pad behind tabs on narrow screens.
- Spacing steps are 4, 8, 12, 16, 24, 32 px. Panels use a 24 px gutter on desktop and 16 px on phones.

## Elevation & Depth

Depth comes from tone, not heavy shadows. The page is the lowest layer, the calculator and panels are one step lighter (light theme) or lighter (dark theme), and the display is inset one step darker or whiter than its panel. Keys have a 1 px bottom edge for a pressed-button feel and a quick 80 ms pressed state. Drawers and bottom sheets use one soft shadow and a dimmed backdrop.

## Shapes

Rounded and friendly: keys and chips use medium radius (`{rounded.md}`), the display and panels use large (`{rounded.lg}`), and the RAD/DEG/GRAD toggle, phase chips and badges are fully rounded (`{rounded.full}`). The equals key is the same shape as the others: it stands out by color, not size. `[ASSUMPTION: equals key may span two rows on the number pad on desktop, as in the references.]`

## Components

- **Display** (`display`): two lines. The upper line shows the expression being built (secondary, 20 px); the lower line shows the result or current entry (primary, 40 px, right-aligned). Long values scroll horizontally with the end in view. An error replaces the result line and uses `error`.
- **Keys**: six families, each `key-*` token pair above. States: default, hover (a 6% darker background), pressed (inset edge), focus (2 px `focus-ring` plus 2 px offset), disabled (50% opacity, only where an action is invalid).
- **Segmented toggle** (`segmented-toggle`): DEG | RAD | GRAD, the active segment filled with the operator color. It is visible whenever a trig function can be used, and it sits above the keys as in the light reference.
- **View switch**: a tab-style control in the top bar for Simple, Scientific, Programmer and Converter. The active tab has an underline in the operator color, as the light reference does for `main`.
- **Memory indicator**: a small teal `M` badge in the display corner when the Memory Register is non-zero.
- **Trace Panel** (`trace-panel`): header with the title and the analogy notice; four collapsible phase groups, each led by a `trace-phase-chip`; steps listed in order with a number, a stage name, a detail, and a value; footer with step count and duration.
- **History drawer and Settings sheet**: panel surfaces (`surface`) with `border`, list rows 48 px high, destructive actions in `key-clear` colors.
- **Unit Converter**: two stacked fields with a swap button, a category tab row, and unit pickers; field backgrounds use `display`.
- **Programmer view**: the Scientific layout with hex keys A-F in `key-function`, a BIN/OCT/DEC/HEX selector, word size selector, and bitwise keys in `key-operator`.

## Do's and Don'ts

- **Do** keep each color tied to one key family.
- **Do** pair color with a label, icon or shape for anything that carries meaning.
- **Do** keep the display larger than any key label.
- **Do** show the trace phase colors only in the Trace Panel.
- **Don't** use green or coral for decoration.
- **Don't** put more than six key families on screen or add a seventh color.
- **Don't** use pure black or pure white text on tinted keys; use the paired text token.
- **Don't** animate the display digits; results appear instantly. Only the panels and sheets may animate, and they respect reduced motion.
