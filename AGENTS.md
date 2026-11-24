# Bruce Firmware

Use the build profile specified in the task for builds and verification. For the T-Embed CC1101 Plus, use `lilygo-t-embed-cc1101`. Before changing the HAL, check the board revision, `boards/<board>/`, the active build flags, and shared bus ownership.

Launch X-Dev only when the user explicitly asks to work as a team or involve subagents. Apply the same rule when editing the X-Dev configuration. The workflow, roles, and skill routing are documented in [.codex/CREW.md](.codex/CREW.md).

Report which checks were performed on the host, during the build, and on the board. Flash firmware and change device state only within the scope of an explicit hardware verification request.

Write all project instructions, technical descriptions, and commit messages in English. This applies to both commit subjects and bodies.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
- A committed graphify-out/graph.json is rewritten whole on every build, so a large diff to it in a pull request is expected. Do not flag it as churn or suggest skipping the graph rebuild or its hooks.
