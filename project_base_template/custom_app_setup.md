# Custom App Setup

A complete, step-by-step guide to setting up a new (or existing) Frappe/ERPNext custom app with dependency management (flit), pre-commit hooks, linters (black, isort, ruff, flake8, eslint, prettier), commit linting, editor config, and GitHub Actions CI.

---

## 1. Activate the Bench Environment

Navigate to your bench's root directory and activate the virtual environment:

```bash
source env/bin/activate
```

You need to run this every time you open a new terminal session to work on the bench.

---

## 2. Set Up Dependency Management

Open the `pyproject.toml` file.

A `pyproject.toml` file is **already created by default** when the app is scaffolded (e.g. via `bench new-app`) — you don't need to run `flit init` to generate it. Just open the existing `pyproject.toml` in your app directory and **replace its contents** with the template below.

Replace `<app_name>`, `<email_id>`, and `<App Description>` with your actual app details. Add any real third-party packages your app needs to the `dependencies` list (e.g. `"PyMuPDF>=1.26.7"`) — the commented `frappe~=15.0.0` line is just a reminder that frappe itself is managed by bench, not listed here.

> **What to edit vs. what to copy as-is:**
- `[project]` — this is the only part you customize per app. Fill in `<app_name>`, `<email_id>`, `<App Description>`, and your real `dependencies`.
- `[tool.black]` through `[tool.ruff.format]` — copy this section exactly as written, with no changes. It's the shared lint/format config used across all apps, so it should be identical from app to app.


### Example `pyproject.toml`

```toml
[project]
name = "<app_name>"
authors = [
    { name = "8848 Digital LLP", email = "<email_id>"}
]
description = "<App Description>"
requires-python = ">=3.10"
readme = "README.md"
dynamic = ["version"]
dependencies = [
    # "frappe~=15.0.0" # Installed and managed by bench.
]

[build-system]
requires = ["flit_core >=3.4,<4"]
build-backend = "flit_core.buildapi"


[tool.black]
line-length = 99

[tool.isort]
profile = "black"
known_frappe = "frappe"
sections = ["FUTURE", "STDLIB", "THIRDPARTY", "FRAPPE", "FIRSTPARTY", "LOCALFOLDER"]
line_length = 99
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
indent = "\t"

# These dependencies are only installed when developer mode is enabled
[tool.bench.dev-dependencies]
# package_name = "~=1.1.0"

[tool.ruff]
line-length = 110
target-version = "py310"

[tool.ruff.lint]
select = [
    "F",
    "E",
    "W",
    "I",
    "UP",
    "B",
    "RUF",
]
ignore = [
    "B017", # assertRaises(Exception) - should be more specific
    "B018", # useless expression, not assigned to anything
    "B023", # function doesn't bind loop variable - will have last iteration's value
    "B904", # raise inside except without from
    "E101", # indentation contains mixed spaces and tabs
    "E402", # module level import not at top of file
    "E501", # line too long
    "E741", # ambiguous variable name
    "F401", # "unused" imports
    "F403", # can't detect undefined names from * import
    "F405", # can't detect undefined names from * import
    "F722", # syntax error in forward type annotation
    "W191", # indentation contains tabs
]
typing-modules = ["frappe.types.DF"]

[tool.ruff.format]
quote-style = "double"
indent-style = "tab"
docstring-code-format = true
```

---

## 3. Set Up `pre-commit`

**Step 1:** Install pre-commit:

```bash
pip install pre-commit
```

**Step 2:** Navigate to the app's directory (e.g. `/apps/<app_name>`) and register the git hook:

```bash
pre-commit install
```

### Running pre-commit manually

- **Run on all files:**

```bash
pre-commit run --all-files
```

- **Run on a specific folder/file list** (example — you can adapt this with other file-listing commands piped in):

```bash
git ls-files -- <app_name>/<app_name>/page/pos_interface/* | xargs pre-commit run --files
```

---

## 4. Add Required Configuration Files

Now you need to create several configuration files inside your app directory (`apps/<app_name>`). Each one is described below — **create the file with the exact name shown, then paste in the given content.**

### 4.1 `.pre-commit-config.yaml`

**Create a new file** named `.pre-commit-config.yaml` in the `<app_name>` directory (app root). Paste the following content into it:

```yaml
exclude: 'node_modules|.git'
default_stages: [commit]
fail_fast: false


repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
      - id: trailing-whitespace
        exclude: ".*json$|.*txt$|.*csv|.*md|.*svg"
      - id: check-yaml
      - id: no-commit-to-branch
        args: ['--branch', 'master', '--branch', 'develop']
      - id: check-merge-conflict
      - id: check-ast
      - id: check-json
      - id: check-toml
      - id: check-yaml
      - id: debug-statements

  - repo: https://github.com/frappe/black
    rev: 951ccf4d5bb0d692b457a5ebc4215d755618eb68
    hooks:
      - id: black

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v2.7.1
    hooks:
      - id: prettier
        types_or: [javascript, vue, scss]
        # Ignore any files that might contain jinja / bundles
        exclude: |
            (?x)^(
                <app_name>/public/dist/.*|
                .*node_modules.*|
                .*boilerplate.*|
                <app_name>/www/website_script.js|
                <app_name>/templates/includes/.*|
                <app_name>/public/js/lib/.*|
                <app_name>/website/doctype/website_theme/website_theme_template.scss
            )$


  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v8.44.0
    hooks:
      - id: eslint
        types_or: [javascript]
        args: ['--quiet']
        # Ignore any files that might contain jinja / bundles
        exclude: |
            (?x)^(
                <app_name>/public/dist/.*|
                cypress/.*|
                .*node_modules.*|
                .*boilerplate.*|
                <app_name>/www/website_script.js|
                <app_name>/templates/includes/.*|
                <app_name>/public/js/lib/.*
            )$

  - repo: https://github.com/PyCQA/isort
    rev: 5.12.0
    hooks:
      - id: isort

  - repo: https://github.com/PyCQA/flake8
    rev: 6.0.0
    hooks:
      - id: flake8
        additional_dependencies: ['flake8-bugbear',]

  - repo: local
    hooks:
    - id: check-max-lines
      name: Check Max Lines
      entry: python scripts/check_max_lines.py
      language: python
      types: [python]
```

> Note: this file references a local hook (`check-max-lines`) that needs a script — that's created in the next step.

---

### 4.2 `scripts/check_max_lines.py`

This is a **local pre-commit hook script** referenced in `.pre-commit-config.yaml` above. It enforces a maximum number of lines (default: 250) per Python file.

**Step 1:** Create a new folder named `scripts` inside your app directory:

```bash
mkdir -p apps/<app_name>/scripts
```

**Step 2:** Inside that folder, **create a new file** named `check_max_lines.py`:

```python
import sys

MAX_LINES = 250


def main(argv):
	files = argv[1:]
	for file_path in files:
		with open(file_path) as file:
			lines = file.readlines()
			if len(lines) > MAX_LINES:
				print(f"Error: File {file_path} has more than {MAX_LINES} lines.")
				sys.exit(1)


if __name__ == "__main__":
	main(sys.argv)
```

The final path should be: `apps/<app_name>/scripts/check_max_lines.py`

---

### 4.3 `.eslintrc`

This is the **JS linter** configuration. **Create a new file** named `.eslintrc` in the app root directory and paste the following:

> ⚠️ If you get a "declaration" / "undefined variable" error for any global variable during linting, add that variable to the `globals` dictionary below (following the same pattern as the existing entries), e.g. `"erpnext": true`.

```json
{
	"env": {
		"browser": true,
		"node": true,
		"es2022": true
	},
	"parserOptions": {
		"sourceType": "module"
	},
	"extends": "eslint:recommended",
	"rules": {
		"indent": "off",
		"brace-style": "off",
		"no-mixed-spaces-and-tabs": "off",
		"no-useless-escape": "off",
		"space-unary-ops": ["error", { "words": true }],
		"linebreak-style": "off",
		"quotes": ["off"],
		"semi": "off",
		"camelcase": "off",
		"no-unused-vars": "off",
		"no-console": ["warn"],
		"no-extra-boolean-cast": ["off"],
		"no-control-regex": ["off"]
	},
	"root": true,
	"globals": {
		"frappe": true,
		"Vue": true,
		"SetVueGlobals": true,
		"__": true,
		"repl": true,
		"Class": true,
		"locals": true,
		"cint": true,
		"cstr": true,
		"cur_frm": true,
		"cur_dialog": true,
		"cur_page": true,
		"cur_list": true,
		"cur_tree": true,
		"msg_dialog": true,
		"is_null": true,
		"in_list": true,
		"has_common": true,
		"posthog": true,
		"has_words": true,
		"validate_email": true,
		"open_web_template_values_editor": true,
		"validate_name": true,
		"validate_phone": true,
		"validate_url": true,
		"get_number_format": true,
		"format_number": true,
		"format_currency": true,
		"comment_when": true,
		"open_url_post": true,
		"toTitle": true,
		"lstrip": true,
		"rstrip": true,
		"strip": true,
		"strip_html": true,
		"replace_all": true,
		"flt": true,
		"precision": true,
		"CREATE": true,
		"AMEND": true,
		"CANCEL": true,
		"copy_dict": true,
		"get_number_format_info": true,
		"strip_number_groups": true,
		"print_table": true,
		"Layout": true,
		"web_form_settings": true,
		"$c": true,
		"$a": true,
		"$i": true,
		"$bg": true,
		"$y": true,
		"$c_obj": true,
		"refresh_many": true,
		"refresh_field": true,
		"toggle_field": true,
		"get_field_obj": true,
		"get_query_params": true,
		"unhide_field": true,
		"hide_field": true,
		"set_field_options": true,
		"getCookie": true,
		"getCookies": true,
		"get_url_arg": true,
		"md5": true,
		"$": true,
		"jQuery": true,
		"moment": true,
		"hljs": true,
		"Awesomplete": true,
		"Sortable": true,
		"Showdown": true,
		"Taggle": true,
		"Gantt": true,
		"Slick": true,
		"Webcam": true,
		"PhotoSwipe": true,
		"PhotoSwipeUI_Default": true,
		"io": true,
		"JsBarcode": true,
		"L": true,
		"Chart": true,
		"DataTable": true,
		"Cypress": true,
		"cy": true,
		"it": true,
		"describe": true,
		"expect": true,
		"context": true,
		"before": true,
		"beforeEach": true,
		"after": true,
		"qz": true,
		"localforage": true,
		"extend_cscript": true
	}
}
```

---

### 4.4 `.flake8`

This is the **Python linter** config. **Create a new file** named `.flake8` in the app root directory and paste the following:

```ini
[flake8]
ignore =
    B001,
    B007,
    B009,
    B010,
    B950,
    E101,
    E111,
    E114,
    E116,
    E117,
    E121,
    E122,
    E123,
    E124,
    E125,
    E126,
    E127,
    E128,
    E131,
    E201,
    E202,
    E203,
    E211,
    E221,
    E222,
    E223,
    E224,
    E225,
    E226,
    E228,
    E231,
    E241,
    E242,
    E251,
    E261,
    E262,
    E265,
    E266,
    E271,
    E272,
    E273,
    E274,
    E301,
    E302,
    E303,
    E305,
    E306,
    E402,
    E501,
    E502,
    E701,
    E702,
    E703,
    E741,
    F401,
    F403,
    F405,
    W191,
    W291,
    W292,
    W293,
    W391,
    W503,
    W504,
    E711,
    E129,
    F841,
    E713,
    E712,
    B028,
    W604,

max-line-length = 200
exclude=,test_*.py
```

---

### 4.5 `commitlint.config.js`

This enforces **conventional commit naming** (e.g. `feat: ...`, `fix: ...`). **Create a new file** named `commitlint.config.js` in the app root directory and paste the following:

```javascript
module.exports = {
  parserPreset: "conventional-changelog-conventionalcommits",
  rules: {
    "subject-empty": [2, "never"],
    "type-case": [2, "always", "lower-case"],
    "type-empty": [2, "never"],
    "type-enum": [
      2,
      "always",
      [
        "build",
        "chore",
        "ci",
        "docs",
        "feat",
        "fix",
        "perf",
        "refactor",
        "revert",
        "style",
        "test",
      ],
    ],
  },
};
```

---

### 4.6 `.editorconfig`

This defines **formatting rules for your editor** (tabs vs spaces, indent size, line endings, etc.). **Create a new file** named `.editorconfig` in the app root directory and paste the following:

```ini
# Root editor config file
root = true

# Common settings
[*]
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8

# python, js indentation settings
[{*.py,*.js,*.vue,*.css,*.scss,*.html}]
indent_style = tab
indent_size = 4
max_line_length = 99

# JSON files - mostly doctype schema files
[{*.json}]
insert_final_newline = false
indent_style = space
indent_size = 2
```

---

### 4.7 App `commands/` (custom bench CLI)

Every custom app ships a `commands/` package under the app Python package so
agents and developers get the shared `8848-export-fixtures` bench command.

**Step 1:** Create the destination folder (it may already exist empty from
scaffolding):

```bash
mkdir -p apps/<app_name>/<app_name>/commands
```

**Step 2:** Copy the template files **verbatim** from this repo's
`project_base_template/commands/` into that folder:

```bash
cp project_base_template/commands/__init__.py \
   project_base_template/commands/export_fixtures.py \
   project_base_template/commands/README.md \
   apps/<app_name>/<app_name>/commands/
```

Final paths:

```
apps/<app_name>/<app_name>/commands/
├── __init__.py
├── export_fixtures.py
└── README.md
```

**Step 3:** Add these lines to `apps/<app_name>/<app_name>/hooks.py` (near
the top, after the `app_*` metadata). Replace `<app_name>` with the Python
package name and `<Module Name>` with the exact value from `modules.txt`:

```python
custom_fixtures = [{"dt": "Custom Field", "filters": {"module": "<Module Name>"}}]

commands = ["<app_name>.commands.export_fixtures.export_fixtures"]
```

> **Usage:** `bench --site <site> 8848-export-fixtures --app <app_name>`.
> See `commands/README.md` for details. Export uses the `custom_fixtures`
> hook (not the built-in `fixtures` hook).

---

### 4.8 CodeGraph (code intelligence index)

CodeGraph builds a local, pre-computed knowledge graph (a SQLite index) of
every symbol, edge, and file in the workspace, so agent tooling
(`codegraph_explore` / `codegraph explore`) can answer "where is X defined"
/ "what calls Y" in one call instead of grepping the whole repo. Set this up
once per machine (the CLI) and once per app repo (the index).

**Step 1:** Install the CodeGraph CLI globally (machine-wide, not per-app):

```bash
npm install -g @colbymchenry/codegraph
```

**Step 2:** From the app's repo root (`apps/<app_name>`), build the index:

```bash
codegraph install
```

This creates a `.codegraph/` directory at the repo root. Its presence is
what tells agent tooling to prefer `codegraph_explore`/`codegraph explore`
over grep/find/reading files for this repo.

> **Note:** `.codegraph/` is a local, regenerable index, not source — add
> `.codegraph/` to `.gitignore` rather than committing it.

---

### 4.9 App `utils/api_handlers/` (shared API response helpers)

Per this repo's `CLAUDE.md` API conventions, every custom app centralizes
its whitelisted-endpoint response shaping in exactly one place:
`utils/api_handlers/` at the app root. It holds cross-cutting,
**non-whitelisted** helpers — the standard response envelope, error-message
cleanup, and the `after_request` formatter — reused by every module's
`<module_name>/api/vN/` endpoints. It is never duplicated per module, and
these files never live inside a module's `api/` folder itself.

**Step 1:** Create the destination folders (the `utils/` package may
already exist from earlier app work):

```bash
mkdir -p apps/<app_name>/<app_name>/utils/api_handlers
touch apps/<app_name>/<app_name>/utils/__init__.py \
      apps/<app_name>/<app_name>/utils/api_handlers/__init__.py
```

**Step 2:** Copy the template files **verbatim** from this repo's
`project_base_template/api_handlers/` into that folder:

```bash
cp project_base_template/api_handlers/envelope.py \
   project_base_template/api_handlers/error_messages.py \
   project_base_template/api_handlers/response_formatter.py \
   apps/<app_name>/<app_name>/utils/api_handlers/
```

Final paths:

```
apps/<app_name>/<app_name>/utils/
├── __init__.py
├── common.py                      # if/when you need generic app-wide helpers
└── api_handlers/
    ├── __init__.py
    ├── envelope.py
    ├── error_messages.py
    └── response_formatter.py
```

**Step 3:** Replace `<app_name>` inside the three copied files (the
`Wired in hooks.py:` line in `response_formatter.py`'s docstring and the
`/api/method/<app_name>` path check) with the real Python package name.

**Step 4:** Wire the `after_request` hook in
`apps/<app_name>/<app_name>/hooks.py`:

```python
after_request = ["<app_name>.utils.api_handlers.response_formatter.format_frappe_response_to_custom"]
```

> **Usage:** import `api_response(...)` from
> `<app_name>.utils.api_handlers.response_formatter` into any
> `<module_name>/api/v1/*.py` file to shape a response — see
> [api.md](../skills/frappe-app-dev/references/api.md). Never place
> `envelope.py`, `error_messages.py`, or `response_formatter.py` inside a
> module's `api/` folder — their home is always the app-root
> `utils/api_handlers/`, shared across every module.

---

## 5. Set Up the GitHub Actions Workflow

Now you need to set up a CI workflow in GitHub.

**Step 1:** Go to your repository on GitHub.

**Step 2:** Click on the **Actions** tab.

**Step 3:** Click **"set up a workflow yourself"**.

**Step 4:** Name the file `linters.yml` (this will be created under `.github/workflows/linters.yml`).

**Step 5:** Paste one of the two versions below, depending on your project type.

> ⚠️ Use **only one** of these — not both. Pick based on whether pre-commit is already configured (see Section 3–4 above).

### 5.1 Version A — For EXISTING projects (no pre-commit checks in linters)

Use this if your project does **not** yet have pre-commit configured, or you don't want the workflow to run pre-commit checks.

```yaml
name: Linters

on:
  pull_request:
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: commitcheck-${{ github.event_name }}-${{ github.event.number }}
  cancel-in-progress: true

jobs:
  commit-lint:
    name: 'Semantic Commits'
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 200
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          check-latest: true

      - name: Check commit titles
        run: |
          npm install @commitlint/cli @commitlint/config-conventional
          npx commitlint --verbose --from ${{ github.event.pull_request.base.sha }} --to ${{ github.event.pull_request.head.sha }}
        
  semgrep:
    name: semgrep
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python 3.10
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          cache: pip

      - name: Download Semgrep rules
        run: git clone --depth 1 https://github.com/frappe/semgrep-rules.git frappe-semgrep-rules

      - name: Download semgrep
        run: pip install semgrep

      - name: Run Semgrep rules
        run: semgrep ci --config ./frappe-semgrep-rules/rules --config r/python.lang.correctness

  deps-vulnerable-check:
    name: 'Vulnerable Dependency Check'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - uses: actions/checkout@v4

      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/*requirements.txt', '**/pyproject.toml', '**/setup.py') }}
          restore-keys: |
            ${{ runner.os }}-pip-
            ${{ runner.os }}-

      - name: Install and run pip-audit
        run: |
          pip install pip-audit
          cd ${GITHUB_WORKSPACE}
          pip-audit --desc on .
```

### 5.2 Version B — For NEW projects (includes pre-commit checks)

Use this if you've already set up `.pre-commit-config.yaml` and related files as described in Section 4.

```yaml
name: Linters

on:
  pull_request:
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: commitcheck-${{ github.event_name }}-${{ github.event.number }}
  cancel-in-progress: true

jobs:
  commit-lint:
    name: 'Semantic Commits'
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 200
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          check-latest: true

      - name: Check commit titles
        run: |
          npm install @commitlint/cli @commitlint/config-conventional
          npx commitlint --verbose --from ${{ github.event.pull_request.base.sha }} --to ${{ github.event.pull_request.head.sha }}

  linters:
    name: linters
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python 3.10
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          cache: pip

      - name: Install and Run Pre-commit
        uses: pre-commit/action@v3.0.0

        
  semgrep:
    name: semgrep
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python 3.10
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          cache: pip

      - name: Download Semgrep rules
        run: git clone --depth 1 https://github.com/frappe/semgrep-rules.git frappe-semgrep-rules

      - name: Download semgrep
        run: pip install semgrep

      - name: Run Semgrep rules
        run: semgrep ci --config ./frappe-semgrep-rules/rules --config r/python.lang.correctness

  deps-vulnerable-check:
    name: 'Vulnerable Dependency Check'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - uses: actions/checkout@v4

      - name: Cache pip
        uses: actions/cache@v3
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/*requirements.txt', '**/pyproject.toml', '**/setup.py') }}
          restore-keys: |
            ${{ runner.os }}-pip-
            ${{ runner.os }}-

      - name: Install and run pip-audit
        run: |
          pip install pip-audit
          cd ${GITHUB_WORKSPACE}
          pip-audit --desc on .
```

---

## 6. Final Directory Structure

After completing all the steps above, your app's root directory should look like this:

```
<app_name>/
├── .github/
│   └── workflows/
│       └── linters.yml
├── .codegraph/                   # local index built by `codegraph install` (gitignored)
├── <app_name>/                  # main app source folder
│   ├── commands/                # copied from project_base_template/commands/
│   │   ├── __init__.py
│   │   ├── export_fixtures.py
│   │   └── README.md
│   └── utils/
│       ├── __init__.py
│       └── api_handlers/        # copied from project_base_template/api_handlers/
│           ├── __init__.py
│           ├── envelope.py
│           ├── error_messages.py
│           └── response_formatter.py
├── scripts/
│   └── check_max_lines.py
├── .eslintrc
├── .flake8
├── .gitignore
├── .pre-commit-config.yaml
├── .editorconfig
├── README.md
├── commitlint.config.js
├── license.txt
├── pyproject.toml
```

**Files/folders you created/added as part of this guide** (highlighted in the reference screenshot):
- `.github/workflows/linters.yml`
- `scripts/` folder (containing `check_max_lines.py`)
- `<app_name>/commands/` (from `project_base_template/commands/`)
- `<app_name>/utils/api_handlers/` (from `project_base_template/api_handlers/`)
- `.codegraph/` (from `codegraph install`, gitignored)
- `.eslintrc`
- `.flake8`
- `.pre-commit-config.yaml`
- `.editorconfig`
- `commitlint.config.js`
- `pyproject.toml`

---

## 7. Quick Checklist (Summary)

| # | Action | File/Command |
|---|--------|--------------|
| 1 | Activate bench env | `source env/bin/activate` |
| 2 | Update `pyproject.toml` file | - |
| 3 | Install pre-commit | `pip install pre-commit` |
| 4 | Register pre-commit hook | `pre-commit install` |
| 5 | Create pre-commit config | `.pre-commit-config.yaml` |
| 6 | Create max-lines script | `scripts/check_max_lines.py` |
| 7 | Create JS lint config | `.eslintrc` |
| 8 | Create Python lint config | `.flake8` |
| 9 | Create commit lint config | `commitlint.config.js` |
| 10 | Create editor config | `.editorconfig` |
| 11 | Copy app `commands/` package | from `project_base_template/commands/` → `<app_name>/commands/` |
| 12 | Wire `custom_fixtures` + `commands` in `hooks.py` | See Section 4.7 |
| 13 | Install CodeGraph CLI + build index | `npm install -g @colbymchenry/codegraph` then `codegraph install` |
| 14 | Copy app `utils/api_handlers/` package | from `project_base_template/api_handlers/` → `<app_name>/utils/api_handlers/` |
| 15 | Wire `after_request` in `hooks.py` | See Section 4.9 |
| 16 | Set up GitHub Actions workflow | `.github/workflows/linters.yml` (choose new vs existing version) |
| 17 | Verify structure | Compare against Section 6 |
