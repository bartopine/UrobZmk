# Keymap Reorganization Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Reorganize ZMK keymap to make Colemak-DH default, consolidate thumb keys, remove FN mirrors, and add bare gaming layer.

**Architecture:** All changes are in `config/base.keymap` and `config/combos.dtsi`. New behaviors are added before the keymap section. Layer order is swapped. A new GAME layer is appended. The `ZMK_BASE_LAYER` macro is updated to reflect new outer column and thumb keys.

**Tech Stack:** ZMK firmware, Devicetree syntax, C preprocessor macros

**Design doc:** `docs/plans/2026-02-16-keymap-reorganization-design.md`

---

### Task 1: Add New Behaviors

Add the 5 new behaviors needed before updating the macro/layers. These are unused initially, so the keymap remains compilable.

**Files:**
- Modify: `config/base.keymap:198-222` (after existing behaviors, before combos include)

**Step 1: Add leader_morph behavior**

In `config/base.keymap`, replace the `tmux_col` morph (line 200-201) with the new `leader_morph`:

Replace:
```c
// tap: tmux leader | shft + tap: colemak layer
SIMPLE_MORPH(tmux_col, SFT, &kp LC(SPACE), &tog COL)
```

With:
```c
// tap: leader | shft + tap: shift+leader (uppercase accents)
SIMPLE_MORPH(leader_morph, SFT, &leader, &leader_sft)
```

**Step 2: Replace num_ldr with tmux_num behavior**

In `config/base.keymap`, replace the `num_ldr` hold-tap and `ldr_sldr` morph (lines 218-222) with new `tmux_num` behaviors:

Replace:
```c
// Tap: leader | Shift + tap: shift_leader | Hold: num layer
ZMK_HOLD_TAP(num_ldr, bindings = <&mo>, <&ldr_sldr>; flavor = "balanced";
             tapping-term-ms = <200>; quick-tap-ms = <QUICK_TAP_MS>;)
SIMPLE_MORPH(ldr_sldr, SFT, &leader, &leader_sft)
ZMK_MACRO(leader_sft, bindings = <&sk LSHFT &leader>;)
```

With:
```c
// Tap: tmux leader (Ctrl+Space) | Shift + tap: smart_num | Hold: num layer
SIMPLE_MORPH(tmux_smart, SFT, &kp LC(SPACE), &num_dance)
ZMK_HOLD_TAP(tmux_num, bindings = <&mo>, <&tmux_smart>; flavor = "balanced";
             tapping-term-ms = <200>; quick-tap-ms = <QUICK_TAP_MS>;)
ZMK_MACRO(leader_sft, bindings = <&sk LSHFT &leader>;)
```

Note: `leader_sft` macro is kept — it's used by `leader_morph`.

**Step 3: Add smart_bare_fn behavior**

In `config/base.keymap`, add after the `tmux_num` block (before the combos include):

```c
// Tap: one-shot game layer | Shift + tap: toggle FN | Hold: game layer
SIMPLE_MORPH(sl_game_fn, SFT, &sl GAME, &tog FN)
ZMK_HOLD_TAP(smart_bare_fn, bindings = <&mo>, <&sl_game_fn>; flavor = "balanced";
             tapping-term-ms = <200>; quick-tap-ms = <QUICK_TAP_MS>;)
```

**Step 4: Commit**

```bash
git add config/base.keymap
git commit -m "refactor: replace thumb behaviors for keymap reorganization

Replace tmux_col with leader_morph, num_ldr with tmux_num,
and add smart_bare_fn for the new game layer access."
```

---

### Task 2: Add GAME Layer Define and Layer

Add the GAME layer constant and the bare Colemak-DH layer definition.

**Files:**
- Modify: `config/base.keymap:23` (add define)
- Modify: `config/base.keymap:329` (add layer after Mouse)

**Step 1: Add GAME layer define**

In `config/base.keymap`, after `#define MOUSE 6` (line 23), add:

```c
#define GAME 7
```

**Step 2: Add GAME layer definition**

In `config/base.keymap`, after the Mouse layer closing `)` (line 329), add:

```c
ZMK_BASE_LAYER(Game,
//╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮ ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮
    &kp Q         &kp W         &kp F         &kp P         &kp B       ,   &kp J         &kp L         &kp U         &kp Y         &kp SQT     ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &kp A         &kp R         &kp S         &kp T         &kp G       ,   &kp M         &kp N         &kp E         &kp I         &kp O       ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &kp Z         &kp X         &kp C         &kp D         &kp V       ,   &kp K         &kp H         &kp COMMA     &kp DOT       &kp FSLH    ,
//╰─────────────┼─────────────┴─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┴───────────────────────────╯
                                              ___           &kp SPACE   ,   &kp RET       &kp BSPC
//                                          ╰─────────────┴─────────────╯ ╰─────────────┴─────────────╯
)
```

Notes:
- LH1 is `___` (transparent — falls through to tmux_num from layer 0)
- LH0 is plain `&kp SPACE` (no NAV hold-tap delay)
- RH0 is plain `&kp RET` (no FN hold-tap delay)
- RH1 is plain `&kp BSPC` (no delete morph)
- All letter/punctuation keys are plain `&kp` (no HRM, no morphs)
- Outer column keys come from `ZMK_BASE_LAYER` macro

**Step 3: Commit**

```bash
git add config/base.keymap
git commit -m "feat: add bare GAME layer (Colemak-DH without HRM)

Layer 7 provides plain key presses without home-row mods or
smart punctuation for gaming and HRM-free typing."
```

---

### Task 3: Update ZMK_BASE_LAYER Macro

Update the macro that controls outer column and thumb edge keys for all layers.

**Files:**
- Modify: `config/base.keymap:232-239`

**Step 1: Update the macro**

In `config/base.keymap`, replace the macro definition (lines 232-239):

Replace:
```c
#define ZMK_BASE_LAYER(name, LT, RT, LM, RM, LB, RB, LH, RH)                   \
    ZMK_LAYER(                                                                 \
        name,                                                                  \
         &sl MED LT RT CUS_P                                                   \
           CUS_L LM RM CUS_D                                                   \
        &kp LGUI LB RB &none                                                   \
       &tmux_col LH RH &smart_mouse                                            \
    )
```

With:
```c
#define ZMK_BASE_LAYER(name, LT, RT, LM, RM, LB, RB, LH, RH)                   \
    ZMK_LAYER(                                                                 \
        name,                                                                  \
         &sl MED LT RT CUS_P                                                   \
           CUS_L LM RM CUS_D                                                   \
        &tog COL LB RB &smart_mouse                                            \
       &leader_morph LH RH &smart_bare_fn                                      \
    )
```

Changes:
- LB5: `&kp LGUI` → `&tog COL` (layout toggle)
- RB5: `&none` → `&smart_mouse` (relocated from RH2)
- LH2: `&tmux_col` → `&leader_morph` (leader key for accents)
- RH2: `&smart_mouse` → `&smart_bare_fn` (bare/FN key)

**Step 2: Update Amut base layer thumb bindings**

In `config/base.keymap`, in the Amut layer (line 255), replace the left thumb binding:

Replace:
```c
                                              &num_ldr NUM 0  &lt_spc NAV 0,   &lt FN RET    &bs_del
```

With:
```c
                                              &tmux_num NUM 0  &lt_spc NAV 0,   &lt FN RET    &bs_del
```

Change: `&num_ldr` → `&tmux_num` (LH1 now does tmux+smartnum instead of leader+num)

**Step 3: Commit**

```bash
git add config/base.keymap
git commit -m "refactor: update base layer macro and thumb bindings

LB5: GUI → layout toggle, RB5: none → smart_mouse,
LH2: tmux_col → leader_morph, RH2: smart_mouse → smart_bare_fn,
LH1: num_ldr → tmux_num."
```

---

### Task 4: Swap Layer Order (Colemak-DH as Default)

Swap the Colemak-DH and QWERTY/Amutated layer definitions so Colemak-DH is layer 0.

**Files:**
- Modify: `config/base.keymap:247-269`

**Step 1: Swap the two base layer definitions**

In `config/base.keymap`, the two base layers (currently Amut then Colemak) need to be swapped. The first ZMK_BASE_LAYER defined becomes layer 0 (DEF), the second becomes layer 1 (COL).

Replace the entire block from the Amut layer through the Colemak layer with:

```c
ZMK_BASE_LAYER(Colemak,
//╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮ ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮
    &kp Q         &kp W         &kp F         &kp P         &kp B       ,   &kp J         &kp L         &kp U         &kp Y         &kp SQT     ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &hml LGUI A   &hml LALT R   &hml LSHFT S  &hml LCTRL T  &kp G       ,   &kp M         &hmr LCTRL N  &hmr RSHFT E  &hmr LALT I   &hmr LGUI O ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &kp Z         &kp X         &kp C         &kp D         &kp V       ,   &kp K         &kp H         &comma_morph  &dot_morph    &qexcl      ,
//╰─────────────┼─────────────┴─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┴───────────────────────────╯
                                              &tmux_num NUM 0  &lt_spc NAV 0,   &lt FN RET    &bs_del
//                                          ╰─────────────┴─────────────╯ ╰─────────────┴─────────────╯
)

ZMK_BASE_LAYER(Amut,
//╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮ ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮
    &kp Q         &kp W         &kp E         &kp R         &kp T       ,   &kp Y         &kp U         &kp I         &kp O         &kp P     ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &hml LGUI A   &hml LALT S   &hml LSHFT D  &hml LCTRL F  &kp G       ,   &kp H         &hmr LCTRL J  &hmr RSHFT K  &hmr LALT L   &hmr LGUI SQT ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &kp Z         &kp X         &kp C         &kp V         &kp B       ,   &kp N         &kp M         &comma_morph  &dot_morph    &qexcl      ,
//╰─────────────┼─────────────┴─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┴───────────────────────────╯
                                              ___           ___         ,   ___           ___
//                                          ╰─────────────┴─────────────╯ ╰─────────────┴─────────────╯
)
```

Key changes:
- Colemak layer now comes FIRST (becomes layer 0 = DEF)
- Colemak layer gets explicit thumb bindings (`&tmux_num NUM 0` and `&lt_spc NAV 0` etc.)
- Amut layer now comes SECOND (becomes layer 1 = COL)
- Amut layer thumbs become transparent (`___`) — falls through to Colemak base
- `#define DEF 0` and `#define COL 1` values stay the same (layer order matches)

**Step 2: Commit**

```bash
git add config/base.keymap
git commit -m "feat: make Colemak-DH the default layout

Swap layer order so Colemak-DH is layer 0 (DEF) and
QWERTY/Amutated is layer 1 (COL). Toggle switches to QWERTY."
```

---

### Task 5: Remove FN Layer Right-Side Mirror

Clear the right side of the FN layer to transparent keys.

**Files:**
- Modify: `config/base.keymap` (Fn layer, ~lines 283-293)

**Step 1: Update FN layer**

Replace the Fn layer:

```c
ZMK_BASE_LAYER(Fn,
//╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮ ╭─────────────┬─────────────┬─────────────┬─────────────┬─────────────╮
    &kp F12       &kp F7        &kp F8        &kp F9        ___         ,   ___           ___           ___           ___           ___         ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &hml LGUI F11 &hml LALT F4  &hml LSHFT F5 &hml LCTRL F6 ___         ,   ___           ___           ___           ___           ___         ,
//├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
    &kp F10       &kp F1        &kp F2        &kp F3        ___         ,   ___           ___           ___           ___           ___         ,
//╰─────────────┼─────────────┴─────────────┼─────────────┼─────────────┤ ├─────────────┼─────────────┼─────────────┴───────────────────────────╯
                                              ___           ___         ,   ___           ___
//                                          ╰─────────────┴─────────────╯ ╰─────────────┴─────────────╯
)
```

Change: Right-side function keys (F12, F7-F9, F11, F4-F6, F10, F1-F3) all become `___` (transparent).

**Step 2: Commit**

```bash
git add config/base.keymap
git commit -m "refactor: remove FN layer right-side mirror

Right side was unreachable while holding right thumb for FN.
Keys now fall through to base layer."
```

---

### Task 6: Remove Combos

Remove the two combos that are now redundant.

**Files:**
- Modify: `config/combos.dtsi:30,34`

**Step 1: Remove nums combo**

In `config/combos.dtsi`, delete line 30:

```c
ZMK_COMBO(nums,  SMART_NUM,     LT2 LT3,     DEF NAV NUM COL , COMBO_TERM_SLOW, COMBO_IDLE_SLOW)
```

Reason: smart_num is now on LH1 shift+tap.

**Step 2: Remove lcol combo**

In `config/combos.dtsi`, delete line 34 (after removing nums, this will be line 33):

```c
ZMK_COMBO(lcol,  &tog COL,      LM1 LM3,     DEF     NUM COL , COMBO_TERM_FAST, COMBO_IDLE_FAST)
```

Reason: layout toggle is now on LB5 outer column.

**Step 3: Commit**

```bash
git add config/combos.dtsi
git commit -m "refactor: remove redundant nums and lcol combos

smart_num moved to LH1 shift+tap, layout toggle moved to LB5."
```

---

### Task 7: Verify Build

Verify the firmware compiles correctly.

**Step 1: Check if local build is available**

```bash
cd /home/workmachine/.projects/Workstation/UrobZmk
ls -la build.yaml 2>/dev/null || echo "No local build config"
```

If GitHub Actions is the build system, push to a feature branch and verify CI:

```bash
git checkout -b feat/keymap-reorganization
git push -u origin feat/keymap-reorganization
```

Then check the GitHub Actions build status.

**Step 2: If build fails, fix issues**

Common ZMK build errors:
- Undefined behavior references: check spelling of new behavior names
- Layer count mismatch: verify GAME layer is layer 7 (8 total layers)
- Macro expansion issues: check `ZMK_BASE_LAYER` macro escaping

**Step 3: Commit any fixes**

```bash
git add -A
git commit -m "fix: address build issues from keymap reorganization"
```

---

## File Change Summary

| File | Changes |
|------|---------|
| `config/base.keymap` | Replace 3 behaviors, add 2 new behaviors, update macro, swap layer order, clear FN right side, add GAME layer, add GAME define |
| `config/combos.dtsi` | Remove 2 combo lines (nums, lcol) |

## Verification Checklist

After all changes, confirm:
- [ ] Colemak-DH is layer 0 (first ZMK_BASE_LAYER in file)
- [ ] QWERTY/Amut is layer 1 (second ZMK_BASE_LAYER in file)
- [ ] GAME is layer 7 (last ZMK_BASE_LAYER in file, after Mouse)
- [ ] `#define GAME 7` exists after `#define MOUSE 6`
- [ ] Macro has: `&tog COL` (LB5), `&smart_mouse` (RB5), `&leader_morph` (LH2), `&smart_bare_fn` (RH2)
- [ ] Colemak layer has explicit thumb bindings (`&tmux_num NUM 0`, etc.)
- [ ] Amut layer has transparent thumbs (`___`)
- [ ] FN layer right side is all `___`
- [ ] `nums` combo removed from combos.dtsi
- [ ] `lcol` combo removed from combos.dtsi
- [ ] `tmux_col` behavior removed (replaced by `leader_morph`)
- [ ] `num_ldr` and `ldr_sldr` behaviors removed (replaced by `tmux_num`/`tmux_smart`)
- [ ] `leader_sft` macro preserved (used by `leader_morph`)
- [ ] Firmware builds successfully
