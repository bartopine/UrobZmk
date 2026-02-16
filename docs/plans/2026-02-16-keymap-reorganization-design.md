# Keymap Reorganization Design

Date: 2026-02-16
Status: Approved
Keyboard: Corne (42-key split, nice_nano_v2)

## Goals

1. Make Colemak-DH the default layout
2. Consolidate left thumb cluster for better ergonomics
3. Remove duplicated keymaps without logic (FN right-side mirror)
4. Free up thumb real estate by relocating smart_mouse
5. Add a bare/gaming layer for HRM-free typing

## Changes

### 1. Swap Default Layout

- Layer 0: Colemak-DH (was QWERTY/Amutated)
- Layer 1: QWERTY/Amutated (was Colemak-DH)
- `&tog COL` semantics flip: now toggles TO QWERTY and back

**Files:** `base.keymap` lines 247-269 (swap layer order), update `#define COL` semantics

### 2. Consolidate Left Thumb Cluster

**LH2 (outer left thumb):**
- Was: `&tmux_col` (tap: Ctrl+Space, shift+tap: &tog COL)
- Now: leader_morph (tap: &leader, shift+tap: &leader_sft)
- Implementation: New `SIMPLE_MORPH(leader_morph, SFT, &leader, &leader_sft)` behavior

**LH1 (middle left thumb):**
- Was: `&num_ldr NUM 0` (tap: leader, shift+tap: shift+leader, hold: NUM)
- Now: tmux_num (tap: Ctrl+Space tmux, shift+tap: smart_num, hold: &mo NUM)
- Implementation: New hold-tap wrapping a mod-morph:
  ```
  SIMPLE_MORPH(tmux_smart, SFT, &kp LC(SPACE), &num_dance)
  ZMK_HOLD_TAP(tmux_num, bindings = <&mo>, <&tmux_smart>; ...)
  ```

**Files:** `base.keymap` - new behaviors + update ZMK_BASE_LAYER macro (LH2 position) + update base layer thumb bindings

### 3. Outer Column Changes

**LB5 (outer left bottom):**
- Was: `&kp LGUI`
- Now: `&tog COL` (layout toggle, available on all layers via macro)

**RB5 (outer right bottom):**
- Was: `&none`
- Now: `&smart_mouse` (relocated from RH2)

**Files:** `base.keymap` - update ZMK_BASE_LAYER macro definition

### 4. RH2 Smart Bare/FN Key

**RH2 (outer right thumb):**
- Was: `&smart_mouse` (in macro)
- Now: smart_bare_fn behavior:
  - tap: `&sl GAME` (one-shot bare layer)
  - shift+tap: `&tog FN` (toggle function keys)
  - hold: `&mo GAME` (persistent bare layer)
- Implementation:
  ```
  SIMPLE_MORPH(sl_game_fn, SFT, &sl GAME, &tog FN)
  ZMK_HOLD_TAP(smart_bare_fn, bindings = <&mo>, <&sl_game_fn>; ...)
  ```

**Files:** `base.keymap` - new behaviors + update ZMK_BASE_LAYER macro (RH2 position)

### 5. New GAME Layer (Layer 7)

Bare Colemak-DH without home-row mods or smart punctuation:

```
╭──────┬──────┬──────┬──────┬──────╮ ╭──────┬──────┬──────┬──────┬──────╮
│  Q   │  W   │  F   │  P   │  B   │ │  J   │  L   │  U   │  Y   │  '   │
│  A   │  R   │  S   │  T   │  G   │ │  M   │  N   │  E   │  I   │  O   │
│  Z   │  X   │  C   │  D   │  V   │ │  K   │  H   │  ,   │  .   │  /   │
╰──────┴──────┼──────┼──────┼──────┤ ├──────┼──────┼──────┴──────┴──────╯
              │      │ Space│      │ │Return│ BSPC │
              ╰──────┴──────┴──────╯ ╰──────┴──────╯
```

- All keys plain `&kp` (no `&hml`/`&hmr`)
- Punctuation: plain comma, period, forward slash (no morphs)
- Thumbs: plain Space, Return, Backspace (no layer holds)

**Files:** `base.keymap` - add `#define GAME 7`, add new layer definition

### 6. FN Layer — Remove Right-Side Mirror

- Right side (RT0-RT4, RM0-RM4, RB0-RB4): all become `&trans`
- Left side: unchanged (F1-F12 with HRM on home row)

**Files:** `base.keymap` lines 283-293

### 7. Remove E+W Smart Num Combo

- Delete: `ZMK_COMBO(nums, SMART_NUM, LT2 LT3, ...)` from combos.dtsi line 30
- Reason: smart_num moves to LH1 shift+tap

**Files:** `combos.dtsi` line 30

### 8. Remove S+F Colemak Toggle Combo

- Delete: `ZMK_COMBO(lcol, &tog COL, LM1 LM3, ...)` from combos.dtsi line 34
- Reason: toggle moves to LB5 outer column (all layers)

**Files:** `combos.dtsi` line 34

## Updated ZMK_BASE_LAYER Macro

```c
#define ZMK_BASE_LAYER(name, LT, RT, LM, RM, LB, RB, LH, RH)                 \
    ZMK_LAYER(                                                                 \
        name,                                                                  \
         &sl MED LT RT CUS_P                                                   \
           CUS_L LM RM CUS_D                                                   \
        &tog COL LB RB &smart_mouse                                            \
       &leader_morph LH RH &smart_bare_fn                                      \
    )
```

Changes from current:
- `&kp LGUI` -> `&tog COL` (LB5)
- `&none` -> `&smart_mouse` (RB5)
- `&tmux_col` -> `&leader_morph` (LH2)
- `&smart_mouse` -> `&smart_bare_fn` (RH2)

## Updated Layer Definitions

```
#define DEF   0   // Colemak-DH (was Amutated QWERTY)
#define COL   1   // QWERTY/Amutated (was Colemak)
#define NAV   2   // Navigation (unchanged)
#define FN    3   // Function keys (right side cleared)
#define NUM   4   // Numbers (unchanged)
#define MED   5   // Media/BT (unchanged)
#define MOUSE 6   // Mouse (unchanged)
#define GAME  7   // Bare Colemak-DH (new)
```

## New Behaviors Summary

| Behavior | Type | Tap | Shift+Tap | Hold |
|----------|------|-----|-----------|------|
| leader_morph | mod-morph | &leader | &leader_sft | — |
| tmux_smart | mod-morph | &kp LC(SPACE) | &num_dance | — |
| tmux_num | hold-tap | tmux_smart | — | &mo NUM |
| sl_game_fn | mod-morph | &sl GAME | &tog FN | — |
| smart_bare_fn | hold-tap | sl_game_fn | — | &mo GAME |

## Removed Behaviors

| Behavior | Reason |
|----------|--------|
| tmux_col | Replaced by leader_morph (LH2) |
| num_ldr | Replaced by tmux_num (LH1) |
| ldr_sldr | Replaced by leader_morph |

## Removed Combos

| Combo | Keys | Reason |
|-------|------|--------|
| nums | LT2+LT3 | smart_num moved to LH1 shift+tap |
| lcol | LM1+LM3 | toggle moved to LB5 outer column |

## What Stays Unchanged

- NAV layer (all keys)
- NUM layer (mirrored, both sides)
- MED layer (mirrored, both sides)
- MOUSE layer
- All symbol combos (horizontal + diagonal)
- All other combos (caps, esc, tab, cut/copy/paste, etc.)
- Home-row mod configuration
- Leader key sequences (French accents)
- Smart behaviors (numword, capsword, smart-mouse tri-state)
