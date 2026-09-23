# Architecture and Ports

## Core architectures (`arch/` in [rt/rt](https://git.rtng.org/rt/rt))

| Directory | Role | Notes |
|-----------|------|-------|
| `arch/arm` | 32-bit Arm (Cortex-M/R/A families as used by QEMU & BSPs) | SVCall/PendSV or IRQ soft-IRQ patterns; optional MPU |
| `arch/aarch64` | AArch64 | QEMU `virt` / Cortex-A53 style bring-up |
| `arch/riscv` | RISC-V (A + Zicsr; Zbb helpful) | ecall + MSI/pendable syscall wiring |
| `arch/c28` | Texas Instruments C28x | Used with Delfino-class BSPs |
| `arch/signal` | **Host POSIX simulation** | Not a chip port — Linux/macOS, x86_64 & aarch64 |

QEMU board glue for CI/dev sits under `qemu/` (`m7`, `m55`, `r52`, `a53`, `riscv`).

## What “supported” means

For this documentation:

- **In-tree arch** — context switch, syscall, tick, and interrupt glue compile for that architecture class.
- **External BSP repo** — a repository under [https://git.rtng.org/rt](https://git.rtng.org/rt) that vendors MCU headers, linker scripts, board bring-up, and `examples/` programs. “Enabled” means those examples are wired in that repo’s `SConstruct` (not that Tesla ships a vehicle configuration).
- **Empty / thin** — repository exists but has little or no content yet.

## Single-hart scheduling

Stock `src/rt.c` keeps one active task, one ready bitmap/lists, and one sleep list. Treat the published kernel as **largely single-hart**. SMP is not documented as a supported configuration in-tree.

## External BSP / device repositories

See the full table in [Supported Devices](Supported-Devices.md). Summary of the [rt group](https://git.rtng.org/rt) (fetched 2026-09-23):

| Repo | Family | Status (clone depth 1) |
|------|--------|------------------------|
| `rt` | Core kernel | Active |
| `rt-sitara` | TI Sitara AM263x | Board/MCU/examples present |
| `rt-bb` | Sitara AM335x (BeagleBone class) | **Empty repository** |
| `rt-hercules` | TI Hercules (TMS570 / RM46) | Board/MCU/examples present (no README) |
| `rt-delfino` | TI C2000 Delfino F28379D | README + LaunchPad flow |
| `rt-c29` | TI C29x F29H85x | README + LaunchPad |
| `rt-stm32` | ST STM32 | Multiple Nucleo MCUs/examples |
| `rt-esp32-riscv` | Espressif ESP32-C6 | README + watchdog note |

## Cargo / QEMU feature flags (core repo)

From `Cargo.toml` / `test.bash`: `qemu-m7`, `qemu-m55`, `qemu-r52`, `qemu-a53`, `qemu-rv32`, `qemu-rv64`, optional `task-mpu`, plus chip-oriented features referenced by BSPs (`hercules`, `sitara-*`, etc. when enabled in those trees).

## Porting

See [Porting Guide](10-Porting-Guide.md) for the architecture hooks a new board must satisfy.
