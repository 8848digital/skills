# Licensing & File Headers

## LICENSE.md — mandatory in every repo

Every project ships a `LICENSE.md` at the repo root, verbatim, no
modification except the year if instructed otherwise:

```markdown
Copyright (c) 2026 8848 Digital LLP. All rights reserved.

This software and its source code are the proprietary and confidential
property of 8848 Digital LLP ("the Company"). The software is licensed,
not sold.

No part of this software may be used, copied, reproduced, modified,
distributed, published, sublicensed, or transmitted in any form or by
any means without the prior written permission of the Company.

Unauthorized use or reproduction of this software, in whole or in part,
may result in civil and criminal liability under applicable law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NONINFRINGEMENT.
```

## Per-file copyright header — mandatory on every source and doc file

Every `.py`, `.js`, and `.md` file in the app gets this header as the
**first lines of the file** (before imports, before frontmatter-equivalent
content):

**Python (`.py`):**
```python
# Copyright (c) 2026 8848 Digital LLP. All rights reserved.
# Proprietary and confidential. Unauthorized copying, distribution, or use
# of this file, via any medium, is strictly prohibited without prior
# written permission from 8848 Digital LLP.
```

**JavaScript (`.js`):**
```javascript
// Copyright (c) 2026 8848 Digital LLP. All rights reserved.
// Proprietary and confidential. Unauthorized copying, distribution, or use
// of this file, via any medium, is strictly prohibited without prior
// written permission from 8848 Digital LLP.
```

**Markdown (`.md`, e.g. README.md, SETUP.md):** place as an HTML comment
above the `# Title` so it doesn't render visibly:
```markdown
<!--
Copyright (c) 2026 8848 Digital LLP. All rights reserved.
Proprietary and confidential. Unauthorized copying, distribution, or use
of this file, via any medium, is strictly prohibited without prior
written permission from 8848 Digital LLP.
-->
```

## Exemptions

- **`.json` files** (DocType definitions, fixtures, workspace/report JSON)
  — no comment syntax in standard JSON, so no header. Do not add a
  fake `"_comment"` key to work around this.
- **Auto-generated files** you don't hand-edit (e.g. `modules.txt`,
  build output) — skip the header; add it to the generator/template
  instead if one exists.
- `LICENSE.md` itself does not repeat the header — it *is* the license.

## Rules

- Header goes at the very top of the file — above module docstrings in
  Python, above the first JSDoc block in JS.
- Do not vary the wording file-to-file — copy verbatim from this file.
- New files created by an agent must include the header as part of file
  creation, not as a follow-up edit.

## Anti-patterns

- **Don't add the header to `.json` files.** It breaks the file.
- **Don't paraphrase the header text.** Copy exactly — this is a legal
  notice, not prose to be improved.
- **Don't forget the header on new files mid-task.** If you're creating
  `expense_utils.py`, the header is part of that file from the first
  write, not a cleanup pass at the end.