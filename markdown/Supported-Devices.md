# Supported Devices and Boards

Upstream group: [https://git.rtng.org/rt](https://git.rtng.org/rt).

This page lists **open-source** ports and boards discovered from those repositories. It does **not** claim Tesla vehicle production hardware support.

## Core architecture backends

| Arch tree | Typical targets |
|-----------|-----------------|
| `arch/signal` | Host Linux/macOS simulation |
| `arch/arm` | Cortex-M7/M55, R52, etc. via QEMU & BSPs |
| `arch/aarch64` | Cortex-A53 QEMU virt |
| `arch/riscv` | RV32/RV64 QEMU virt |
| `arch/c28` | TI C28x (Delfino BSP) |

## External BSP repositories

| Repository | MCU(s) found in tree | Board(s) | Examples (count / samples) | Notes |
|------------|----------------------|----------|----------------------------|-------|
| [rt-stm32](https://git.rtng.org/rt/rt-stm32) | `f042k6`, `l031k6`, `f401re`, `f446re`, `f767zi`, `l552ze` (+ `shared`) | `nucleo-32`, `nucleo-64`, `nucleo-144`, `nucleo-144-l5` | ~10 (`blinky`, `serial`, `eth`, `tim_*`, `toggler`, …) | No top-level README; configs in `SConstruct` |
| [rt-hercules](https://git.rtng.org/rt/rt-hercules) | `tms570ls1224-pge`, `rm46l852c-pge` | `launchxl2` | ~8 (`blinky`, `*_irq`, `vim_test`, …) | No README; Hercules safety MCUs |
| [rt-sitara](https://git.rtng.org/rt/rt-sitara) | `am2634` | `lp263` | ~8 (`blinky`, `*_irq`, `ecc`, …) | No README; Sitara AM263x LaunchPad class |
| [rt-delfino](https://git.rtng.org/rt/rt-delfino) | `f28379d` | `launchxl` | ~3 (+ DSS scripts) | README: LAUNCHXL-F28379D via `./test.bash` |
| [rt-c29](https://git.rtng.org/rt/rt-c29) | `f29h85x` | `launchxl` | present | README: LAUNCHXL-F29H85X, `scons --jobs $(nproc)` |
| [rt-esp32-riscv](https://git.rtng.org/rt/rt-esp32-riscv) | `c6` | `c6-devkit` | `blinky`, `no-wdt`, … | README: esp toolchain; flash `no-wdt` before debug |
| [rt-bb](https://git.rtng.org/rt/rt-bb) | — | — | — | **Empty** (group description mentions AM335x) |

### QEMU targets in core `rt` (not physical boards)

| `test.bash` arg | QEMU machine (from script) | CPU class |
|-----------------|----------------------------|-----------|
| `m7` | `mps2-an500` | Cortex-M7 |
| `m55` | `mps3-an547` | Cortex-M55 |
| `r52` | `mps3-an536` | Cortex-R52 |
| `a53` | `virt` GIC v3 | Cortex-A53 |
| `rv32` / `rv64` | `virt` | RISC-V |

## Honesty checklist

- Board names above come from each BSP’s `board/` and `SConstruct` `mcu=` / `board=` entries.
- Empty `rt-bb` is listed explicitly so it is not mistaken for a working AM335x port.
- “Examples build” means the BSP wires `env.Program` (or equivalent) for those files when the right toolchain is installed — local toolchains are **not** verified in this documentation pass.
