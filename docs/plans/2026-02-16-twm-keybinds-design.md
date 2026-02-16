# TWM Keybinds Redesign

Date: 2026-02-16
Status: Approved
Context: Tiling window manager shortcuts for Komorebi (Windows) and Hyprland (Linux)

## Goals

1. Update TWM shortcuts from Alt to Super (matching actual TWM configs)
2. Leverage existing NAV layer HRMs for Move and Resize (no new combos needed)
3. Add workspace switching on NAV left bottom row
4. Remove unused cycle windows and redundant move combos

## Key Insight: HRM Modifier Trick

On the NAV layer, the left home row provides Shift (LM2) and Ctrl (LM1) via HRM.
TWM keys send Super+Direction. Combining modifiers from the left hand:

- TWM key alone = Super+Direction = **Focus**
- Hold LM1 (Ctrl) + TWM key = Super+Ctrl+Direction = **Move window**
- Hold LM2 (Shift) + TWM key = Super+Shift+Direction = **Resize window**

Same trick for workspaces:
- Workspace key = Super+Number = **Switch workspace**
- Hold LM2 (Shift) + workspace key = Super+Shift+Number = **Move window to workspace**

## Changes

### 1. Update TWM aliases from Alt to Super

**File:** `config/base.keymap` (~line 117)

```c
// Before:
#define TWM_LEFT    &kp LA(LEFT)
#define TWM_RIGHT   &kp LA(RIGHT)
#define TWM_DOWN    &kp LA(DOWN)
#define TWM_UP      &kp LA(UP)
#define TWM_CYCW    &kp LA(END)

// After:
#define TWM_LEFT    &kp LG(LEFT)
#define TWM_RIGHT   &kp LG(RIGHT)
#define TWM_DOWN    &kp LG(DOWN)
#define TWM_UP      &kp LG(UP)
```

TWM_CYCW removed entirely.

### 2. Remove move combo aliases and combos

**File:** `config/combos.dtsi`

Remove alias definitions:
```c
#define TWM_MOVE_L  &kp LC(LA(LEFT))
#define TWM_MOVE_R  &kp LC(LA(RIGHT))
```

Remove combo definitions:
```c
ZMK_COMBO(kml,   TWM_MOVE_L,    RT1 RT2,         NAV      , COMBO_TERM_FAST, COMBO_IDLE_FAST)
ZMK_COMBO(kmr,   TWM_MOVE_R,    RT2 RT3,         NAV      , COMBO_TERM_FAST, COMBO_IDLE_FAST)
```

### 3. Update NAV layer

**File:** `config/base.keymap` (Nav layer)

- RT4 (was TWM_CYCW): becomes `___`
- LB4-LB1 (were `___`): become workspace keys

New aliases:
```c
#define TWM_WS1     &kp LG(N1)
#define TWM_WS2     &kp LG(N2)
#define TWM_WS3     &kp LG(N3)
#define TWM_WS4     &kp LG(N4)
```

NAV bottom-left row: `TWM_WS1  TWM_WS2  TWM_WS3  TWM_WS4  ___`

## Result: 20 TWM Actions, 8 Physical Keys

| Action | Shortcut | Keyboard |
|--------|----------|----------|
| Focus left | Super+Left | TWM_LEFT |
| Focus down | Super+Down | TWM_DOWN |
| Focus up | Super+Up | TWM_UP |
| Focus right | Super+Right | TWM_RIGHT |
| Move left | Super+Ctrl+Left | Ctrl HRM + TWM_LEFT |
| Move down | Super+Ctrl+Down | Ctrl HRM + TWM_DOWN |
| Move up | Super+Ctrl+Up | Ctrl HRM + TWM_UP |
| Move right | Super+Ctrl+Right | Ctrl HRM + TWM_RIGHT |
| Resize left | Super+Shift+Left | Shift HRM + TWM_LEFT |
| Resize down | Super+Shift+Down | Shift HRM + TWM_DOWN |
| Resize up | Super+Shift+Up | Shift HRM + TWM_UP |
| Resize right | Super+Shift+Right | Shift HRM + TWM_RIGHT |
| Workspace 1 | Super+1 | TWM_WS1 |
| Workspace 2 | Super+2 | TWM_WS2 |
| Workspace 3 | Super+3 | TWM_WS3 |
| Workspace 4 | Super+4 | TWM_WS4 |
| Move to WS 1 | Super+Shift+1 | Shift HRM + TWM_WS1 |
| Move to WS 2 | Super+Shift+2 | Shift HRM + TWM_WS2 |
| Move to WS 3 | Super+Shift+3 | Shift HRM + TWM_WS3 |
| Move to WS 4 | Super+Shift+4 | Shift HRM + TWM_WS4 |

## What's Removed

- `TWM_CYCW` alias (cycle windows)
- `TWM_MOVE_L` / `TWM_MOVE_R` aliases (replaced by HRM trick)
- `kml` / `kmr` combos (replaced by HRM trick)
