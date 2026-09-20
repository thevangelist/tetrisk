# Tetrisk — design system

Sleek dark product UI, canonical Tetris colour. The chrome is quiet enterprise-grade surface work —
flat panels, one accent, tabular numerals. The playfield is the only saturated thing on screen.

## 1. Tetromino palette (source of truth)

The seven Guideline colours, one per piece. These are the only saturated colours in the product —
everything else is neutral, so the playfield is the only thing that shouts.

| Piece | Token | Hex | Ink (shadow edge) |
|---|---|---|---|
| I | `--I` | `#31c7ef` | `#1b8fae` |
| O | `--O` | `#f7d308` | `#b89a05` |
| T | `--T` | `#ad4d9c` | `#7b3470` |
| S | `--S` | `#42b642` | `#2c7d2c` |
| Z | `--Z` | `#ef2029` | `#a9161c` |
| J | `--J` | `#5a65ad` | `#3c447a` |
| L | `--L` | `#ef7921` | `#ad5413` |

Rule: a piece's colour never changes meaning. Because a move re-rolls the piece, colour is the
player's only instant read of *what they are holding* — it is core signal, not decoration.

## 2. Neutrals

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#0b0d12` | page (with a soft radial lift at the top) |
| `--surface` | `#141822` | HUD panels, stat tiles, readout |
| `--surface-2` | `#1b2030` | button rest fill |
| `--line` | `#252b3b` | every border, 1px |
| `--text` | `#e6eaf2` | primary text |
| `--muted` | `#8a93a8` | labels, hints |
| `--accent` | `#31c7ef` | primary action only (shares I-piece cyan) |

Dark-first, single mode. A theme switch is not worth the surface on a game this small, and a light
playfield loses figure/ground once six piece colours are on screen.

## 3. Shape & type

- Radius: `8px` panels, `5px` buttons, `12px` playfield, `3px` cells. No decorative curves.
- Borders: `1px solid var(--line)` everywhere. Depth comes from fill steps and one inset shadow,
  not from stacked drop shadows.
- Type: Inter. 700 title, 600 numerals, 500 buttons, 400 body. Stat labels are 11px uppercase
  with `.09em` tracking; all numerals `font-variant-numeric: tabular-nums` so they stop jittering.

## 4. Components

**Cell** — `aspect-ratio:1`, 3px radius. Empty: 2.8% white wash, no border. Filled: piece colour
plus an inset top highlight and bottom shade — one rule gives every piece a bevel without
seven gradients.

**Playfield** — 10x16 grid, 2px gap, near-black interior, inset vignette and a single lifted
drop shadow. `aspect-ratio:10/16` with `height:100%` so it fills the viewport height and never
scrolls; the layout gives it whatever height is left and the width follows.

**Stat tile** — uppercase micro-label over a 22px tabular number. Four tiles: Score, Lines, Best,
Games. Persisted keys: `tetrisk.best`, `tetrisk.games`, `tetrisk.lines`, `tetrisk.last`, written on
game over inside a try/catch, so private mode degrades to zeroes instead of throwing.

**Overlay** — one element renders every non-playing state from a `VIEW` table (menu, stats, paused,
over): title, optional final score, optional stats grid, a primary action and an alternate. One
overlay rather than four screens, so a new state costs a table row.

**Button** — 46px, `--surface-2` fill, hover lightens fill and border together, active darkens
and drops 1px. The `.hit` class replays that active state for 110ms, so keyboard and swipe input
light up the same button a click would. One primary only: New game, filled in `--accent`.

**Key legend** — desktop-only panel of `kbd` chips, 2px bottom border for the keycap read.

**Piece readout** — the `Piece: X` line carries a colour swatch of the current piece. Because a
move re-rolls the piece, colour is the player's only instant read of what they are holding — it is
core signal, not decoration, and the letter is always there as the second channel.

## 5. Layout

One `.game` flex row: HUD, playfield, controls. Breakpoints:

- `< 860px` — stacked column, playfield takes the free height.
- `<= 700px` tall and narrow — hide the tagline, shrink stats and buttons.
- landscape `<= 560px` tall — three columns, title hidden, 38px buttons.
- `>= 860px` — HUD and controls each `flex:1` capped at 300px, playfield centred at natural width,
  key legend appears. The page never scrolls in any of these.

## 6. Rotation

SRS. Each piece lives in a fixed box — 4x4 for I, 2x2 for O, 3x3 for the rest — and rotates inside
that box, so the position is the box origin and a rotation never re-centres the shape. The old
normalise-to-bounding-box approach made pieces jump sideways as they turned. Wall kicks use the
standard SRS tables (separate table for I, none for O), y-flipped for a downward grid, and are
tried in order until one fits.

The swap on every move/rotate keeps the rotation index and box origin, and falls back through the
same kick offsets, so a swap can never place a piece inside the stack.

## 7. Game states

`menu` → `playing` ⇄ `paused` → `over`, plus `stats` reachable from menu/paused/over. Anything that
is not `playing` blurs and desaturates the playfield, disables the four movement buttons and stops
the tick — the board underneath stays visible as context but is provably inert, so a stray key or
tap can never move a piece while a dialog is up. Stop ends the run and records it like a real game
over; it is not a discard.

## 8. Input

Every action is a button; the keyboard and swipe gestures call `press(id)`, which flashes the
button and invokes it. Arrows + WASD + space, R restart, P/Esc pause. On the playfield: swipe left/right to move,
down to drop, tap to rotate.

## 9. Accessibility

- All seven piece colours pass 3:1 against the playfield interior.
- Colour is never the only channel: the readout names the piece letter.
- Focus is a 2px accent ring, offset 2px, on every control; full keyboard play.
