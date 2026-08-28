# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, dependency-free "slot machine" that picks a random research topic (in French) from a list. No build step, no package manager, no tests, no framework — the whole app is one self-contained HTML file with inline `<style>` and inline `<script>`.

## Running

The page fetches `./sujets-enigma.csv`, so it must be served over HTTP — opening `index.html` with `file://` silently falls back to the hardcoded list:

```
python -m http.server 8000   # then open http://localhost:8000/index.html
```

Deployment is GitHub Pages from `Michel-cliff/Enigma`; committing to `main` publishes.

## Files

- `index.html` — the app.
- `enigma-v2.html` — a byte-identical copy of `index.html`. Any change to one must be mirrored to the other, or they drift.
- `sujets-enigma.csv` — the live topic list. One sentence per line, `sujet` header row, no quoting needed unless a line contains a comma.
- `sujets-precedents.csv` — archived earlier topic set, not read by the app.

## How the reel works

Understanding these three pieces together is required before touching the animation or the layout:

1. **Two sources of topics.** `SUJETS` inside the script is the fallback; `loadSubjects()` fetches the CSV, strips the `sujet` header and surrounding quotes, and if it returns anything, replaces `list` and calls `render()` again. Edit topics in the CSV, not in the JS array — the array only matters offline.
2. **Everything is driven by `--item-h`.** The reel's height is `--item-h * 5`, the gradient fades are `--item-h * 1.9`, and the JS reads the variable at runtime (`itemH()`) to compute `translateY`. Changing row height in CSS alone is enough; changing it in JS alone breaks alignment. The selected row is always index `round(pos) + 2` (the middle of the five visible rows).
3. **`pos` is in units of rows, not pixels.** `spin()` eases `pos` from its current value over several whole laps of `list.length`, lands on `target`, then normalizes with `pos = end % list.length`. Motion blur is derived from the easing derivative. `prefers-reduced-motion` and a single-item list both short-circuit straight to the result.

## Layout constraint that is easy to re-break

`.reel` is `overflow:hidden` (that clipping is what makes the reel read as a window), so anything that widens a row gets cut off at the left and right edges. The winner emphasis is `transform: scale(var(--pop))` on `.item i`, which scales width as well as height. The two are kept compatible by capping the inner element at `max-width: calc(100% / var(--pop))`, so the scaled result lands back at 100%.

If you raise `--pop`, the cap follows automatically — but taller wrapped text can then overflow `--item-h` vertically, so check the longest topic at both the desktop and the `max-width:520px` breakpoint (which uses its own smaller `--pop`).
