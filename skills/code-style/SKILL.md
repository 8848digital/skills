---
name: code-style
description: Code style rules for readable, maintainable implementation. Load this skill always when writing or editing code, and whenever the user asks about code style, refactoring shape, function/file size, object-oriented structure, helper ordering, or comments. For Frappe-specific work, prefer frappe-app-dev.
---

# Code Style Rules

- Keep functions small. Split when a function has multiple jobs or needs sections to be readable.
- Keep files under 300 lines when practical. Split by responsibility, not by arbitrary layer.
- Prefer object-oriented code over scattered functions.
- Put higher-order/public functions near the top of the file; keep low-level utilities at the bottom.
- Prefix file-local functions (not meant to be imported elsewhere) with `__`.
- Do not use nested for-loops; refactor (e.g. early continue, helper function, flatten data first) to avoid deep nesting.
- Write terse, simple English comments. Explain why something is surprising; do not narrate obvious code. Add a short example or use case in the comment when it clarifies non-obvious usage.
- Do not add abstractions until there are repeated concrete uses.
- Do not duplicate logic across files — extract shared code into a common helper once it repeats.
- Import only what's used; never `import *`.
- Leave a blank line between import blocks (stdlib / third-party / app-local) and before each function or class definition.
- Add blank lines between logical sections within a function body for readability.
- Use descriptive names — avoid single-letter names like `i`, `j` outside tight loops; prefer `invoice_index`, `customer_count`, etc.
- Strip unused variables/functions, and debug statements (`print`, `console.log`, `breakpoint`) before finishing a change.

For Frappe-specific work, prefer `frappe-app-dev`.