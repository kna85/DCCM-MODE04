# ICU ICD-10 Coder — redesign (`index-redesign.html`)

A visual redesign of the ICU ICD-10 Coder. Same engine, same codes, same
offline guarantee — a new way of reading confidence.

This is a candidate, not the live page. `index(1).html` still holds the current
design and is what deploys today. Both files work standalone; open either one
in a browser.

> ⚠️ **Coding aid, not a source of truth.** Every code and summary must be
> validated against the institution's current ICD-10-CM/AM master and the
> original note before any clinical, billing, or documentation use. Do not paste
> patient identifiers (name, MRN, DOB) when testing.

---

## What changed

Nothing about matching, scoring, negation, the seed list, the SOI/ROM
heuristic, or the AI summary prompt. The redesign is the presentation layer,
built around one job: **confidence triage** — making the suggested / possible /
negated distinction unmissable.

### One certainty scale instead of two columns

The previous design showed *Suggested* and *Possible* as parallel columns.
This one shows a single list ordered by descending confidence, with a **certainty
gutter** down the left edge. Each scored row places a tick in that gutter, inset
from a rail that marks 100%, so the ticks form a descending profile you read as
a shape before you read a word. Negated codes carry no tick — they were never
scored, and an empty gutter beside them says so.

Where confidence crosses the matcher's cutoff (`conf >= 45`), a labelled rule
runs the full width. The three regions become regions of one scale rather than
three boxes:

```
 │▌  J96.01   Acute respiratory failure with hypoxia      [x]
 │ ▌ A41.9    Sepsis, unspecified organism                [x]
 ├──── BELOW THRESHOLD — POSSIBLE · 9 ──────────────────────
 │  ▌ E87.2   Acidosis                                    [ ]
 ├──── NEGATED — NOT SELECTED · 1 ──────────────────────────
 │    K92.2   G̶I̶ ̶h̶a̶e̶m̶o̶r̶r̶h̶a̶g̶e̶
 │            "no evidence of gi bleed"
```

Negated rows quote the clause that held the code back. The one thing you need
from a negated code is proof the engine read the sentence correctly.

### Colour means physiology, ink means certainty

Confidence is carried entirely by type — size, weight, and ink value — never by
hue. The acuity hues appear in exactly one place, the SOI/ROM band, because in a
clinical tool those colours already mean *how sick*, and spending them on *how
sure* muddies both.

Keeping confidence off the colour channel means the triage survives a
black-and-white print, a colourblind reader, and a badly calibrated ward
monitor. Pen blue is reserved for a third meaning: things you touched —
selection, focus, links.

### Light and dark are both first-class

The tool is used day and night, so neither scheme is an afterthought. The page
follows the OS by default; the **Dark** / **Light** button in the masthead
overrides that and the choice persists in `localStorage` under `icd-theme`. Dark
is not an inversion — the background is a blue-slate rather than near-black, and
the acuity hues desaturate so they don't glow.

### Smaller changes

- The reference tab's 34 filter chips are now a single specialty-group picker.
  The chips ran six rows deep and pushed the code table below the fold.
- Code rows are two lines: code and description, then specialty group and the
  four longest matched evidence terms on one subordinate line.
- On the AI Summary tab, the deterministic codes render in their own mono block
  beneath the model's prose, so you can see at a glance which text came from the
  engine and which came from the model.
- The disclaimer is a permanent strip at the foot of every tab. Chart forms print
  their small print at the foot of the page; the tool never stops saying what it is.

---

## Design tokens

Everything derives from these. They're CSS custom properties at the top of the
`<style>` block.

| Token | Light | Dark | Job |
|---|---|---|---|
| `--chart` | `#F1F3F2` | `#0F1A22` | Page background |
| `--sheet` | `#FFFFFF` | `#17242E` | Panels, input panes, rows |
| `--ink` | `#16232E` | `#E3EBEF` | Full certainty |
| `--ink-2` | `#5C7080` | `#93A9B6` | Reduced certainty |
| `--ink-3` | `#93A3AD` | `#5E7684` | Negated, metadata, hints |
| `--rule` | `#D3DAD9` | `#243541` | Hairlines |
| `--pen` | `#20539B` | `#5C9BD6` | Selection, focus, links |
| `--acuity-low` | `#1F7A5C` | `#3FA383` | SOI/ROM band only |
| `--acuity-mid` | `#C77B2A` | `#D9A05B` | SOI/ROM band only |
| `--acuity-high` | `#B03A3A` | `#D46A6A` | SOI/ROM band only |

Three type roles, three stacks:

- `--serif` — `"Sitka Text", Charter, "Iowan Old Style", Georgia, serif`. Code
  descriptions and prose. Sitka and Charter are both Matthew Carter designs cut
  for legibility at small sizes, so the pairing holds its character across
  Windows and macOS.
- `--util` — `Bahnschrift, "Avenir Next Condensed", "Helvetica Neue", sans-serif`.
  Labels, tabs, buttons, rule captions. The instrument layer.
- `--mono` — `Consolas, "SF Mono", Menlo, ui-monospace, monospace`. ICD codes,
  scores, evidence terms, quoted clauses.

**There are no webfonts and there must never be any.** Every face ships with the
operating system. Adding `@font-face` with a remote `src` would break the
Coder and Reference tabs' no-network guarantee; base64-embedding a family would
bloat a file that needs to stay small enough to email or drop on a share drive.

---

## Privacy and network access

Unchanged from the original, and the constraint the design was built around:

- The Coder and Reference tabs make **no network calls at all**.
- The AI Summary tab downloads model weights once from a CDN (`esm.run`), then
  works offline. Pasted text is processed on-device via WebGPU.
- The Feedback tab posts to Web3Forms when you submit it. That is the only other
  request the page makes, and only when you press the button.

Clinical text stays in the browser tab and is discarded on close or reload.

---

## Browser support

Coder and Reference work in any current browser. AI Summary needs a recent
desktop Chrome or Edge with WebGPU.

Responsive down to 390px: input panes stack, the certainty gutter narrows from
40px to 22px rather than being dropped, and the acuity band wraps. Keyboard
focus is visible on every control, including the tabs and the custom
checkboxes, which are native inputs underneath. `prefers-reduced-motion: reduce`
disables the row stagger and the rule draw — verified that rows and rules render
fully with animation off, since their initial state comes only from the
animation's `both` fill.

---

## Maintaining it

The rules from the original README all still apply — `SEED` row shape, the `SYN`
map, preserving negation behaviour, no backend, keep it one self-contained file.
On top of those:

- **Don't add a fourth colour meaning.** Physiology, certainty, interaction.
  If something needs emphasis, reach for the utility face or a hairline stripe
  before reaching for a hue.
- **The threshold rule reads a real number.** It's drawn where `conf >= 45`
  in `generate()`. If you retune that cutoff, the rule moves with it — that's
  the point. Don't hard-code a position.
- **`match()` records `negClause`**, the clause that held a negated code back.
  It is display-only and must stay that way; it has no effect on scoring or
  negation.
- **Watch selector specificity** between type-based classes (`.section-h`) and
  element-based ones (`.btn`). Section spacing is set in one place; overriding
  it per-element is how these two cancel each other out.
- Both theme blocks must stay in sync: `:root`, the
  `prefers-color-scheme: dark` block, and `:root[data-theme="dark"]`. The
  explicit attribute has to win in both directions.

---

## Design record

The full design rationale, including the four changes made after seeing it
rendered, is in
[`docs/superpowers/specs/2026-07-21-visual-redesign-design.md`](docs/superpowers/specs/2026-07-21-visual-redesign-design.md).
That spec refers to the work as a change to `index(1).html`, which is where it
was built before being split into this file.

---

## Disclaimer

Provided for educational and documentation-support purposes only. Not a medical
device; does not provide medical advice. The SOI/ROM acuity estimate is a
transparent heuristic, **not** the validated 3M APR-DRG grouper, and must not be
used for prognostication, benchmarking, or billing. A qualified clinician is
responsible for validating all output before any clinical use.
