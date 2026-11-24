---
name: bruce-embedded
description: Plan, implement, or review Bruce ESP32 firmware changes involving board pinout, HAL, USB Mass Storage, shared SPI, display, input, or FreeRTOS.
---

# Bruce Embedded

1. Identify the target `env` from the task. Check `platformio.ini`, the profile in `boards/`, the board JSON, and its `interface.cpp`. Verify the board revision and the manufacturer's pinout before changing GPIO assignments. For the T-Embed CC1101 Plus, use `lilygo-t-embed-cc1101`. See the [board audit](../../../docs/t-embed-cc1101-plus-audit.md) for settings and open risks.
2. Trace the normal call path from `src/main.cpp` and the relevant code under `src/core` through `src/hal`, `interface.cpp`, and the selected driver. Check which files the build includes and which dependency versions it selects when a conclusion depends on library behavior. For shared HAL code, check the active `#if` branches and any other affected profiles.
3. When multiple tasks, ISRs, or callbacks are involved, identify the state owner, buffer lifetimes, and the order in which operations stop. Check blocking calls, polling intervals, and how long the shared bus is held. For display changes, measure input latency and the amount of redraw work before changing the SPI clock rate.
4. For Mass Storage, check the initialization sequence and how startup failures are handled, microSD ownership between FAT and MSC, block operation bounds, eject behavior, completion of active I/O, cable disconnection, and re-entry. SPI transactions protect transfers on the bus. The filesystem needs its own ownership protocol.
5. For memory changes, distinguish flash, internal RAM, and PSRAM. Check the worst-case buffer size and task or ISR stack usage when data placement or lifetime changes. Report values from the memory map separately from measurements taken on the board.

The implementer reports the changed files, the contract, and the checks performed. The reviewer reports findings with file paths and the conditions that trigger them. Follow `bruce-platformio` for builds and hardware bench checks.
