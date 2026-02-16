# TWM Keybinds Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Update TWM keybinds to use Super instead of Alt, add workspace switching, and remove redundant combos.

**Architecture:** Update alias definitions in `base.keymap`, add workspace aliases, update NAV layer bottom-left row, and remove move combos from `combos.dtsi`.

**Tech Stack:** ZMK firmware, Devicetree syntax, C preprocessor macros

**Design doc:** `docs/plans/2026-02-16-twm-keybinds-design.md`

---

### Task 1: Update TWM aliases and add workspace aliases

**Files:**
- Modify: `config/base.keymap:117-121` (TWM alias block)

**Step 1: Replace TWM alias block**

In `config/base.keymap`, find the tiling window manager aliases section (~line 117) and replace:

```c
// tiling window manager aliases (komorebi / hyprland)
#define TWM_LEFT    &kp LA(LEFT)
#define TWM_RIGHT   &kp LA(RIGHT)
#define TWM_DOWN    &kp LA(DOWN)
#define TWM_UP      &kp LA(UP)
#define TWM_CYCW    &kp LA(END)
```

With:

```c
// tiling window manager aliases (komorebi / hyprland)
#define TWM_LEFT    &kp LG(LEFT)
#define TWM_RIGHT   &kp LG(RIGHT)
#define TWM_DOWN    &kp LG(DOWN)
#define TWM_UP      &kp LG(UP)

// tiling window manager workspace aliases
#define TWM_WS1     &kp LG(N1)
#define TWM_WS2     &kp LG(N2)
#define TWM_WS3     &kp LG(N3)
#define TWM_WS4     &kp LG(N4)
```

Changes:
- `LA()` (Alt) → `LG()` (Super) for all directional TWM keys
- `TWM_CYCW` removed
- 4 new workspace aliases added

**Step 2: Commit**

```bash
git add config/base.keymap
git commit -m "refactor: update TWM aliases from Alt to Super, add workspaces

Switch tiling WM shortcuts to Super modifier for Komorebi/Hyprland
compatibility. Add workspace 1-4 aliases. Remove cycle windows."
```

---

### Task 2: Update NAV layer

**Files:**
- Modify: `config/base.keymap` (Nav layer, ~line 277-287)

**Step 1: Update NAV layer top-right and bottom-left rows**

In the Nav layer definition, replace:

```c
    CUS_Q         CUS_W         CUS_E         CUS_R         CUS_T       ,   TWM_LEFT      TWM_DOWN      TWM_UP        TWM_RIGHT     TWM_CYCW    ,
```

With:

```c
    CUS_Q         CUS_W         CUS_E         CUS_R         CUS_T       ,   TWM_LEFT      TWM_DOWN      TWM_UP        TWM_RIGHT     ___         ,
```

Then replace the bottom-left row:

```c
    ___           ___           ___           ___           ___         ,   &kp HOME      &kp END       &kp PG_UP     &kp PG_DN     ___         ,
```

With:

```c
    TWM_WS1       TWM_WS2       TWM_WS3       TWM_WS4       ___         ,   &kp HOME      &kp END       &kp PG_UP     &kp PG_DN     ___         ,
```

Changes:
- RT4: `TWM_CYCW` → `___` (transparent)
- LB4: `___` → `TWM_WS1` (Super+1)
- LB3: `___` → `TWM_WS2` (Super+2)
- LB2: `___` → `TWM_WS3` (Super+3)
- LB1: `___` → `TWM_WS4` (Super+4)
- LB0: stays `___`

**Step 2: Commit**

```bash
git add config/base.keymap
git commit -m "feat: add workspace keys to NAV layer, remove cycle

Workspace 1-4 on left bottom row. Shift+workspace = move window
to workspace via HRM trick. Cycle windows key removed."
```

---

### Task 3: Remove move combos from combos.dtsi

**Files:**
- Modify: `config/combos.dtsi:25-26,41-42`

**Step 1: Remove TWM move alias definitions**

In `config/combos.dtsi`, delete the two alias lines (~lines 25-26):

```c
#define TWM_MOVE_L  &kp LC(LA(LEFT))
#define TWM_MOVE_R  &kp LC(LA(RIGHT))
```

**Step 2: Remove kml/kmr combo definitions**

In `config/combos.dtsi`, delete the two combo lines (~lines 41-42):

```c
ZMK_COMBO(kml,   TWM_MOVE_L,    RT1 RT2,         NAV      , COMBO_TERM_FAST, COMBO_IDLE_FAST)
ZMK_COMBO(kmr,   TWM_MOVE_R,    RT2 RT3,         NAV      , COMBO_TERM_FAST, COMBO_IDLE_FAST)
```

**Step 3: Commit**

```bash
git add config/combos.dtsi
git commit -m "refactor: remove TWM move combos, replaced by HRM trick

Move window is now Ctrl HRM + TWM direction key (Super+Ctrl+Arrow).
No dedicated combos needed."
```

---

## Verification Checklist

After all changes, confirm:
- [ ] TWM aliases use `LG()` (Super) not `LA()` (Alt)
- [ ] `TWM_CYCW` is gone from aliases and NAV layer
- [ ] `TWM_WS1-4` aliases defined
- [ ] NAV bottom-left has workspace keys (LB4-LB1)
- [ ] RT4 on NAV is `___`
- [ ] `TWM_MOVE_L`/`TWM_MOVE_R` aliases removed from combos.dtsi
- [ ] `kml`/`kmr` combos removed from combos.dtsi
- [ ] Firmware builds successfully
