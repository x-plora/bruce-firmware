# X-Dev Team for Bruce Firmware

Launch X-Dev only when the user explicitly asks to work as a team or involve subagents. The same rule applies when discussing or configuring the team. The primary agent owns the outcome, chooses the roles needed, and accounts for the available slots and the limit in [config.toml](config.toml).

## Roles and Skills

Project skills are in `.agents/skills/`. Each role description specifies when to read the assigned `SKILL.md`. Load the skills relevant to the task.

| Role | Responsibilities and Deliverables | Skills |
| --- | --- | --- |
| [embed-engineer](agents/embed-engineer.toml) | Agrees on the hardware contract and implements C/C++, HAL, build, and regression test changes in the assigned files. Reports the diff, operating conditions, and verification results. | [bruce-embedded](../.agents/skills/bruce-embedded/SKILL.md), [bruce-platformio](../.agents/skills/bruce-platformio/SKILL.md) for builds and tests, [esp32](../.agents/skills/esp32/SKILL.md) for GPIO, [esp32-s3](../.agents/skills/esp32-s3/SKILL.md) for SoC peripherals |
| [clean-code](agents/clean-code.toml) | Reviews the plan or diff for complexity, reuse, state ownership, and verifiability. Reports findings. | [bruce-clean-code](../.agents/skills/bruce-clean-code/SKILL.md) |
| [reviewer](agents/reviewer.toml) | Independently reviews the completed diff for regressions, data loss, synchronization issues, and hardware risks. | [bruce-embedded](../.agents/skills/bruce-embedded/SKILL.md), [bruce-platformio](../.agents/skills/bruce-platformio/SKILL.md) for assessing verification, [esp32](../.agents/skills/esp32/SKILL.md) for GPIO, [esp32-s3](../.agents/skills/esp32-s3/SKILL.md) for SoC capabilities |
| [tester](agents/tester.toml) | Runs the agreed builds and scenarios. Reports actual outcomes and evidence. | [bruce-platformio](../.agents/skills/bruce-platformio/SKILL.md), [bruce-embedded](../.agents/skills/bruce-embedded/SKILL.md) for hardware scenarios, [esp32-s3](../.agents/skills/esp32-s3/SKILL.md) for USB, sleep, and SoC peripheral scenarios |
| [writer](agents/writer.toml) | Prepares public-facing copy, including LCD, CLI, and WebUI strings. Sends strings to the code owner for integration. | [bruce-docs](../.agents/skills/bruce-docs/SKILL.md) |
| [archivist](agents/archivist.toml) | Maintains technical documentation, canonical sources, and navigation. | [bruce-docs](../.agents/skills/bruce-docs/SKILL.md) |

## Workflow

1. The primary agent records the goal, completion criteria, target profile, and initial Git state. For the T-Embed CC1101 Plus, use the `lilygo-t-embed-cc1101` profile and share the [board audit](../docs/t-embed-cc1101-plus-audit.md) when relevant.
2. For changes to GPIO, buses, device state, memory, or timing, the primary agent obtains the contract from the `embed-engineer` before implementation that depends on it. If the plan spans multiple subsystems, changes resource ownership, or introduces an abstraction, `clean-code` reviews it before edits begin. For a small, localized task, assign only the review or implementation step the task needs.
3. Before work starts, the primary agent assigns one owner per file to be changed. Independent assignments may run in parallel when they use separate files and resources. Agree on ownership transfers before making further edits.
4. The `writer` receives verified facts through the primary agent. The `embed-engineer` receives the finished strings through the primary agent and checks their placement on the target device. The `archivist` updates technical documentation to reflect the agreed behavior. Each role works within its assigned files.
5. For verification, the primary agent gives the `reviewer` and `tester` the same snapshot of the changes. These roles can work in parallel with separate resources. Keep the files under review stable while checks run. Bring `clean-code` back when a revised plan or the complexity of the completed change needs review.
6. The primary agent sends confirmed findings to the owners and returns the fixes to the roles that found them. Reuse finding IDs on subsequent passes. The final report records the changes, checks performed, and limitations. The primary agent creates a Git commit only when the user separately requests it.

## Assignment Handoff

Provide the role, goal, completion criteria, target profile and board revision, files in scope, results from the preceding stage, and required checks. For verification, specify the base commit and the snapshot to check. For hardware scenarios, include the port, authorized operations, and the test bench owner. Give one operator at a time access to the USB or serial port and the test microSD card.

Changes to source files, tests, and build settings belong to the assigned implementer. The `reviewer`, `clean-code`, and `tester` roles report findings and results while leaving those files untouched. Route questions between roles, contract changes, and ownership transfers through the primary agent.

## Verification Results

- A finding includes a stable ID, priority, file and line, trigger condition, observed or expected impact, and proposed fix.
- A verification result includes the command or scenario, profile, outcome, and a status of `PASS`, `FAIL`, `NOT_RUN`, or `BLOCKED`. Give the reason for either of the last two statuses.
- Record host tests, builds, flashing, and device observations separately. Hardware verification stays within the explicitly assigned scope.
- Send a progress update when work is blocked, a contract changes, or a result that another role depends on is ready. Include facts, paths, and open questions in the final report.
