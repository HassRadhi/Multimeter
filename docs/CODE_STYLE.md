# Code style and structure

Firmware and tooling for the DIY STM32F373 multimeter. Applies to new code under `firmware/` (and any host-side helpers we add later). Hardware stays in `hardware/`.

This doc merges the project’s working rules (smallest change, clear ownership, Australian English) with practices common at large engineering orgs (Google-style C/C++ clarity, reviewability, and explicit boundaries).

**Lab instrument only (±60 V / ±2.5 A). Not CAT-rated — do not use on mains. Never invent safety ratings in code comments or UI copy.**

---

## 1. Principles (decide before writing)

Climb this ladder; stop at the first rung that holds:

1. Does this need to exist? (YAGNI)
2. Does it already exist in this repo? Reuse it.
3. Does the standard library / CMSIS / HAL already do it? Prefer that.
4. Can it be one clear function or a few lines? Prefer that.
5. Only then: write the minimum new code that works.

Also:

- **Root cause, not symptom.** Fix the shared function once; don’t patch every caller.
- **Shortest correct diff wins** — but only after you understand the flow end to end.
- **Deletion over addition.** Boring over clever.
- **No abstractions nobody asked for.** No new dependency if an existing one or stdlib covers it.
- **One fix at a time** when debugging; validate, then continue or undo.

Deliberate shortcuts that have a known ceiling (global lock, O(n²) scan, naive filter) get a short comment:

```c
/* ponytail: O(n) full-buffer scan each frame; upgrade to running sum if n > 1k */
```

---

## 2. Repository structure

```
firmware/
  README.md                 # build, flash, board pin notes
  src/
    main.c                  # bring-up, main loop only — thin
    board/                  # clocks, pins, board constants (v15_1)
    drivers/                # SPI, I2C, GPIO wrappers — no UI policy
    display/                # SSD1306 + draw primitives
    measure/                # SDADC, ranging, scaling
    ui/                     # screens, buttons, hold/minmax
    util/                   # delay, math helpers, asserts
  include/                  # public headers mirroring src areas
  tests/                    # host-buildable unit checks where possible
docs/
  CODE_STYLE.md             # this file
hardware/
  v15_1/                    # fabbed board (source of truth for pins)
```

### Ownership

| Layer | Owns | Must not |
|-------|------|----------|
| `main` | init order, loop, mode dispatch | raw register twiddling, UI layout details |
| `board` | pin map, clocks, IRQ wiring | measurement maths, fonts |
| `drivers` | bus/peripheral ops | product modes, calibration policy |
| `display` | pixels / OLED protocol | voltage formulas |
| `measure` | ADC, ranges, SI values | SPI bit-banging, fonts |
| `ui` | screens, input, copy | direct SDADC register access |

**Free functions over “god objects.”** Prefer `ssd1306_clear()` / `measure_read_voltage()` over a large opaque API object with thin wrappers that only rename a call.

**Thin entrypoints.** `main` wires modules; modules own their domain.

Upstream MartinD firmware is **reference only** (pins, SDADC behaviour). Do not copy it wholesale into this tree.

---

## 3. Language and toolchain

- Primary: **C11** (or C++17 only if we explicitly adopt C++ for a subsystem — default stays C).
- Target: **STM32F373CCT6**, board `hardware/v15_1`.
- Toolchain: **VS Code + ST STM32 extension** (CubeMX, build, flash, debug). See `docs/FIRMWARE.md`.
- Pin names in code should match schematic nets where practical (`LCD_CLK`, `BTN1`, …) or document the mapping in `board/pins.h`.

When C++ is used:

- Follow [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) naming and header hygiene unless this doc overrides.
- No exceptions in firmware paths. Prefer explicit error returns.

---

## 4. Naming

| Kind | Convention | Example |
|------|------------|---------|
| Files | `snake_case` | `ssd1306.c`, `range_v.c` |
| Functions | `snake_case`, module prefix | `ssd1306_init`, `ui_on_btn` |
| Types | `snake_case` + `_t` | `measure_sample_t` |
| Enums | type `foo_t`, values `FOO_…` | `mode_t` / `MODE_VOLTAGE` |
| Macros | `ALL_CAPS` | `BOARD_HSE_HZ` |
| Static file-local | `snake_case` | `s_frame_buf` |
| Constants | `k` prefix *or* `ALL_CAPS` — pick one per module and stick to it | `kOledWidth` or `OLED_WIDTH` |

**Event / input handlers** use a `handle` / `on` prefix when they react to something: `ui_handle_btn`, `measure_on_dma_complete`.

**Names describe intent**, not implementation trivia. Prefer `voltage_mv` over `tmp2`.

**No Hungarian notation.** No `g_` for everything global — prefer file-scope `static` and narrow sharing via headers.

Australian English in **user-facing** strings only (`colour` if we ever show it; “initialising”, “cancelled”). Code identifiers stay ASCII `snake_case` / US spelling where that matches CMSIS/HAL (`initialize` in vendor APIs is fine).

---

## 5. Formatting

Match the project formatter once one is set (clang-format). Until then:

- Indent: **4 spaces** (no tabs).
- Column guide: **100** characters soft limit; break earlier if it helps readability.
- Braces: K&R / Allman is fine — **be consistent within a file**. Prefer:

```c
if (err != 0) {
    return err;
}
```

- One statement per line.
- Spaces around binary operators; no space after function name: `foo(x, y)`.
- Order includes: related header, then project headers, then CMSIS/HAL, then libc. Use include guards or `#pragma once` consistently.

Google-style header rule: every `.c` includes its own `.h` first so the header compiles alone.

---

## 6. Functions and control flow

- **Early returns**; keep nesting **≤ 4**.
- Aim for function bodies **~50 lines**; split when the story needs a name.
- File target **&lt; 400 lines**, hard ceiling **~500**. Split by responsibility, not by arbitrary line count alone.
- Prefer `const` for read-only data and pointer targets that must not be written.
- No “pass-through” wrappers that only rename another function — call the real one.
- Boolean parameters that obscure call sites: prefer enums or separate functions (`display_set_inverted(true)` is OK; `draw(x, y, true, false, true)` is not).

```c
/* good */
if (buf == NULL) {
    return ERR_ARG;
}
if (len == 0) {
    return ERR_OK;
}
/* process… */
```

---

## 7. Types, errors, and boundaries

- **Strict types at boundaries.** No “bag of ints” across modules without units in the name (`_mv`, `_ua`, `_counts`).
- Prefer **explicit result types** over magic numbers:

```c
typedef enum {
    ERR_OK = 0,
    ERR_ARG,
    ERR_TIMEOUT,
    ERR_BUSY,
} err_t;
```

- Discriminated outcomes beat overloaded return codes where it clarifies (`measure_result_t` with `ok` / `overrange` / `error`).
- Don’t silently ignore `err_t`. At minimum `assert` in debug or propagate.
- **No unbounded recursion.** Prefer iteration on MCU.
- Integer sizes: use `stdint.h` (`uint16_t`, `int32_t`) for wire/ADC/protocol widths; `size_t` for object counts in memory.

---

## 8. Embedded / concurrency

- **ISRs stay tiny:** flag, push to ring buffer, or kick DMA. No OLED updates, no `printf`, no blocking waits in IRQ.
- Document shared data: `volatile` only for hardware/ISR-shared flags; prefer C11 `atomic` or critical sections for multi-byte state.
- Never block forever without a timeout on bus ops used from UI paths.
- Power and analogue paths: don’t toggle ranging GPIOs without understanding settle time — name delays (`RANGE_SETTLE_US`) and keep them in `board` or `measure`, not magic numbers in UI.
- Calibration and reference voltages live in one place (`measure` / EEPROM helpers), not scattered literals.

---

## 9. Comments and documentation

- Comments explain **why**, invariants, and hardware quirks — not what the next line obviously does.
- Public headers: one-line purpose per exported function; units on parameters.
- UI / serial copy: Australian English; no CAT claims; no “safe for mains”.
- TODO only with a concrete follow-up (`TODO(name): settle time unmeasured on v15_1`). No placeholder stubs in committed “working” paths.

---

## 10. Testing and checks

Google-style expectation: **non-trivial logic leaves one runnable check**.

- Prefer **host-buildable** unit tests for pure logic (ranging tables, formatting, min/max/hold).
- Mock at the **hardware boundary** only (fake SPI/ADC), not every internal function.
- Drivers that can’t run on host: keep a small **self-check** or bring-up demo (`firmware` target that paints a known pattern).
- Trivial one-liners need no test.

---

## 11. Git commits (when asked to commit)

Format: `<type>[optional scope]: <description>`

- Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `perf`, `test`, …
- Imperative mood; no trailing period on the subject.
- Body explains **what/why** when useful; skip play-by-play how.
- Do not commit secrets, local KiCad locks (`~*`), or generated object trees.

Example: `feat(display): bring up SSD1306 over SPI1`

---

## 12. Code review checklist (author)

Before calling a change done:

- [ ] Diff is the smallest that solves the stated problem
- [ ] Pin / net names match `v15_1` (or map is updated in `board/`)
- [ ] No new dependency without a clear need
- [ ] Errors propagated or explicitly accepted
- [ ] ISR paths stay non-blocking
- [ ] User-visible strings: Australian English; no fake safety ratings
- [ ] Non-trivial logic has a test or bring-up check
- [ ] Files still under size / complexity targets

---

## 13. Explicit non-goals

- Re-implementing a full RTOS on day one
- Porting MartinD UI wholesale
- Abstract HAL over multiple MCUs “just in case”
- Claiming CAT / mains compliance in software or docs

---

## References

- Project handoff: `AGENT_HANDOFF.md`
- Board: `hardware/v15_1/`
- Upstream pin/ADC reference only: [MartinD OSM firmware](https://github.com/MartinD-CZ/STM32F1-open-source-multimeter/tree/master/firmware-multimeter)
- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html) (naming, headers, clarity)
- [Google Engineering Practices](https://google.github.io/eng-practices/) (small CLs, readable reviews)
