---
paths:
  - "<path/to/hard/area/**>"
---

# Rules for <the hard area, e.g. the pricing engine>

This file loads only when Claude opens a file matching `paths` above, so sessions that never touch this area don't pay
for it. Replace the placeholders, add one file per area, and delete this one if the project has no hard area.

This is a hard area: the `planner` subagent plans the change first, and the reviewer runs on Opus.

- <A rule that applies only here, e.g. "change the formula only with a test that compares old and new outputs">
- <What must stay true, e.g. "results must match the reference implementation on the fixtures">
- <How changes are judged, e.g. "the analyst designs and judges any experiment">
