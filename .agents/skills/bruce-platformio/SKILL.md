---
name: bruce-platformio
description: Build and test Bruce firmware with PlatformIO, diagnose Windows toolchain failures, or verify an explicitly assigned hardware scenario.
---

# Build and Verify Bruce

1. Check `default_envs`, `extra_configs`, the target profile, and its source filters. For the T-Embed CC1101 Plus, build with `pio run -e lilygo-t-embed-cc1101`. Before running `pio test`, verify that a suitable test suite and environment exist. On Windows, start with `Get-Command pio`. If PlatformIO is installed, check the executable path in the IDE or at `$env:USERPROFILE/.platformio/penv/Scripts/pio.exe`.
2. Check `pio --version`. If dependencies fail, check `git submodule status` and the installed packages. For example, `ModuleNotFoundError: urllib3` when launching `pio` indicates an environment failure before compilation. Report CLI startup, package setup, and the build separately. Keep environment fixes outside the firmware source.
3. Record the exact command, the first meaningful failure, the profile, and the result. After a successful build, report flash and RAM usage from PlatformIO and include a comparison baseline if available. Record host checks, compilation, flashing, and observations on the board separately.
4. Before hardware verification, agree on the board, port, storage medium, and who controls the test bench. For MSC, use a test microSD card and the agreed scenarios for re-entry, copying, ejecting, and disconnecting. For parallel builds, isolate artifact directories. Give one operator exclusive access to USB and serial for the duration of the scenario.
5. Report the outcome as `PASS`, `FAIL`, `NOT_RUN`, or `BLOCKED` and give the actual reason for any skip or blocker. If the task is assigned to the `tester`, route proposed changes to the file owner through the primary agent.
