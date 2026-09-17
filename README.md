# Fixed-Point Format Explorer

A single-page, no-build tool for picking a fixed-point (Q-format) representation
for a floating-point value or tensor - weighing **round-trip precision** against
**memory footprint**, which is the actual tradeoff behind choosing an INT4 / INT8 /
INT16 / INT32 quantization width when porting a model or embedded variable.

**[Live demo →](#)** *(https://ashwinambatwar-mythic.github.io/quant-width-picker/`)*

---

## What it does

Give it one or more floating-point values (or a representative sample of a
tensor), and it sweeps every fixed-point format across the container widths
you choose, showing:

- The exact round-trip conversion, e.g. `1.5 − 1.4888 = 0.0112`
- Max absolute / relative error across your sample
- Memory cost: bytes/element, total memory (if you give it an element count),
  and compression ratio vs. FP32
- Whether the format overflows for your value(s) or a declared value range
- The narrowest format that still meets an error tolerance you set — the same
  rule used to pick a quantization width for deployment

Everything runs client-side, synchronously, over a few dozen candidate formats
— there's no backend and no perceptible delay.

## Format naming

A format is named `f<F><s|u><I>`:

| Symbol | Meaning |
|---|---|
| `F` | fractional bits |
| `s` / `u` | signed (two's complement) / unsigned container |
| `I` | remaining bits (`I = total_bits − F`), shown for readability |

Example: **`f16s16`** = 16 fractional bits, signed, 32-bit container.
For `1.5`: `1.5 × 2^16 = 98304` (the fixed-point integer stored), and
`98304 / 2^16 = 1.5` converting back — zero error at this width.

## Core formulas

```
float → fixed:        fixed      = round(value × 2^F)
fixed → float:         value_back = fixed / 2^F
error:                 error      = value − value_back
memory / element:      bytes      = total_bits / 8
compression vs FP32:   ratio      = 32 / total_bits
```

## Using it

1. Open `index.html` directly in a browser — no install, no server, no
   dependencies beyond a Google Fonts CDN link (falls back gracefully offline).
2. Enter one value or a comma-separated sample of values.
3. Check the container widths you want to compare (4 / 8 / 16 / 32-bit).
4. Optionally pick a constraint (e.g. sin/cos → signed, range `[-1, 1]`) or
   define a custom range your variable must cover.
5. Optionally enter an element count to see real total memory.
6. Optionally set an error tolerance — the tool highlights the smallest
   format across your checked widths that still meets it.
7. Click any row to see the per-value breakdown and formulas filled in with
   real numbers.

## Deploying on GitHub Pages

1. Rename the file to `index.html` if it isn't already.
2. Push it to this repo (root, or a `/docs` folder — your choice).
3. **Settings → Pages → Source**: pick the branch and folder, save.
4. Your page is live at `https://ashwinambatwar-mythic.github.io/quant-width-picker/`.

## Notes / limitations

- The recommendation is only as good as the sample you give it. For a real
  weight tensor, test with the tensor's true min/max (or a representative
  sample) — not just a "typical" value — or the tool may recommend a format
  that overflows in production.
- A "remember my last input" convenience uses `localStorage`, scoped to your
  own browser; nothing is sent anywhere.
- Tested up to a 32-bit container; the math is plain double-precision
  arithmetic, so widths much beyond 32 bits would need a bignum library to
  stay exact (not included here).

## License

MIT — do whatever you like with it.
