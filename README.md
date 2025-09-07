# Task: Fix subtle subtotal bug (do not rewrite)

You are given buggy_subtotals.js which inserts subtotal rows for a sorted dataset with XAC dimensions and YAC metrics.

Requirements:
- Keep the streaming O(n) approach and public behavior.
- Do not refactor architecture; fix the bug with minimal, surgical changes.
- Add at least 4 unit tests covering:
  - Multi-level subtotals (region/city/store)
  - YACs with at least two numeric fields
  - Group boundaries at both mid-stream and end-of-stream
  - A dataset with single-level (should still work)

The provided acceptance.test.js must pass.

Notes:
- Input rows are already sorted by xacs.
- YACs are numeric; missing values are treated as 0.
- Subtotal rows are marked with `__nf__subtotal: true` and `__nf__subtotal_level`.

Deliver:
- A small patch to `buggy_subtotals.js`
- Your `*.test.js` tests (you can keep or modify acceptance.test.js)
- A short note (2–5 lines) describing what was wrong and why your fix is correct.
