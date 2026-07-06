# CLAUDE.md — Frappe Skills & Custom App Structure

This file documents the agent skills available in this repository and the
canonical structure of a custom Frappe application used in this project.
Read this file at the start of every session that involves Frappe development.

All paths below are **relative to this file** (the skills repo root).
To resolve any path dynamically in Python:

```python
import pathlib

# Resolve the skills repo root from any script inside the repo
SKILLS_ROOT = pathlib.Path(__file__).resolve().parent
# -- or, from anywhere on disk --
SKILLS_ROOT = pathlib.Path("CLAUDE.md").resolve().parent

# Example: open a skill file
skill_path = SKILLS_ROOT / "skills" / "frappe-app-dev" / "SKILL.md"
ref_path   = SKILLS_ROOT / "skills" / "frappe-app-dev" / "references" / "new-app.md"
```

---

## Skills Overview

Skills live under `skills/<name>/` relative to this repo root.
Each skill has a `SKILL.md` — read it before writing code for that topic.

| Skill | When to activate | Entry point |
| ----- | ---------------- | ----------- |
| `frappe-app-dev` | Creating/modifying DocTypes, controllers, APIs, hooks, permissions, background jobs, scheduler, bench CLI, site management, tests | [SKILL.md](./skills/frappe-app-dev/SKILL.md) |
| `code-style` | Writing or reviewing any Python/JS code; questions about style, naming, line length, function size, helper ordering | [SKILL.md](./skills/code-style/SKILL.md) |
| `quality-code-review` | Performing code reviews, audits, or pull-request feedback | [SKILL.md](./skills/quality-code-review/SKILL.md) |
| `ui-design` | Building Frappe Desk UI, Vue SPAs, portal pages, or any front-end component | [SKILL.md](./skills/ui-design/SKILL.md) |

### Activation Rules

- **Always** load `code-style` when writing or editing Python or JavaScript.
- **Always** load `frappe-app-dev` for any Frappe/bench task.
- Load `ui-design` for any front-end or UX task.
- Load `quality-code-review` only when explicitly reviewing existing code.
- Do **not** load all skills at once. Load only what the current task needs.

---

## frappe-app-dev — Flow Selection

Once [SKILL.md](./skills/frappe-app-dev/SKILL.md) is loaded, pick exactly one flow:

| Situation | Reference file |
| --------- | -------------- |
| Creating a new app | [new-app.md](./skills/frappe-app-dev/references/new-app.md) |
| Working on an existing app | [existing-app.md](./skills/frappe-app-dev/references/existing-app.md) |

Then load only the feature references you need for the task:

| Topic | When to load | Reference file |
| ----- | ------------ | -------------- |
| Site management | Finding/creating/managing sites | [site-management.md](./skills/frappe-app-dev/references/site-management.md) |
| DocTypes | Creating/modifying DocTypes, fields, naming | [doctypes.md](./skills/frappe-app-dev/references/doctypes.md) |
| Controllers | Document lifecycle, server logic | [controllers.md](./skills/frappe-app-dev/references/controllers.md) |
| Whitelisted APIs | REST endpoints, `@frappe.whitelist()` | [api.md](./skills/frappe-app-dev/references/api.md) |
| Database & ORM | `frappe.db`, queries, raw SQL | [database.md](./skills/frappe-app-dev/references/database.md) |
| Caching | Redis, `frappe.cache` | [caching.md](./skills/frappe-app-dev/references/caching.md) |
| Realtime | WebSocket, `publish_realtime` | [realtime.md](./skills/frappe-app-dev/references/realtime.md) |
| Background jobs | `frappe.enqueue`, scheduled jobs | [background-jobs.md](./skills/frappe-app-dev/references/background-jobs.md) |
| Hooks | `hooks.py` patterns | [hooks.md](./skills/frappe-app-dev/references/hooks.md) |
| Permissions | Roles, DocType permissions, `has_permission` | [permissions.md](./skills/frappe-app-dev/references/permissions.md) |
| Testing | Writing & running tests | [testing.md](./skills/frappe-app-dev/references/testing.md) |
| Frontend & UI | Desk UI, Vue SPA, portal pages | [frontend.md](./skills/frappe-app-dev/references/frontend.md) (router → 3 sub-files) |
| Frontend — Desk | Desk form/list customisation | [frontend-desk.md](./skills/frappe-app-dev/references/frontend-desk.md) |
| Frontend — Vue | Vue SPA apps | [frontend-vue.md](./skills/frappe-app-dev/references/frontend-vue.md) |
| Frontend — Portal | Portal/website pages | [frontend-portal.md](./skills/frappe-app-dev/references/frontend-portal.md) |
| Bench CLI | All bench commands reference | [bench-operations.md](./skills/frappe-app-dev/references/bench-operations.md) |


---

## Custom App Structure Convention

All custom Frappe apps in this project follow the layout below.
Replace `<app_name>` with the actual app name (snake_case).
Replace `<module_name>` with the actual module name (snake_case). **`<module_name>` must never be the same string as `<app_name>`** — the module directory is a distinct, purposefully-named submodule of the app package, not a repeat of the app's own name.

```
<app_name>/                            ← repo root
├── <app_name>/                        ← Python package (pip-installable)
│   ├── <module_name>/                 ← Module directory (name MUST differ from <app_name>)
│   │   ├── api/                       ← ALL versioned public REST endpoints live here — and ONLY here
│   │   │   ├── v1/
│   │   │   │   ├── bank.py
│   │   │   │   ├── braniac.py
│   │   │   │   └── company.py
│   │   │   ├── api_error_handler.py   ← Centralised exception → HTTP response
│   │   │   ├── before_request.py      ← Pre-request guards / auth checks
│   │   │   └── response_formatter.py  ← Standardised JSON envelope helpers
│   │   │
│   │   ├── customization/             ← Overrides for standard Frappe/ERPNext docs
│   │   │   └── customer/
│   │   │       ├── bank.py
│   │   │       ├── customer.js        ← Client-side form script
│   │   │       ├── customer.py        ← Server-side controller / hooks
│   │   │       ├── player_kyc.py
│   │   │       ├── social_media.py
│   │   │       ├── sponsor.py
│   │   │       ├── utils.py
│   │   │       └── (other business logic files — no api.py here, see below)
│   │   │
│   │   ├── dashboard/                 ← Dashboard chart / number-card scripts
│   │   │   ├── prize_agreement.py
│   │   │   ├── quotation.py
│   │   │   └── sales_order.py
│   │   │
│   │   ├── doctype/                   ← Custom DocTypes (one sub-folder each)
│   │   │   └── creatives/
│   │   │       ├── __init__.py
│   │   │       ├── creatives.js       ← Client script
│   │   │       ├── creatives.json     ← DocType definition (source of truth)
│   │   │       ├── creatives.py       ← Controller
│   │   │       └── (other business logic files — no api.py here, see below)
│   │   │
│   │   ├── print_format/              ← Custom print format definitions
│   │   │   ├── __init__.py
│   │   │   ├── csr_certificate/
│   │   │   │   ├── __init__.py
│   │   │   │   └── csr_certificate.json
│   │   │   └── sponsor_donation/
│   │   │       ├── __init__.py
│   │   │       └── sponsor_donation.json
│   │   │
│   │   ├── report/                    ← Script/Query reports (one sub-folder each)
│   │   │   └── blanket_order_tracking_report/
│   │   │       ├── __init__.py
│   │   │       ├── blanket_order_tracking_report.js
│   │   │       ├── blanket_order_tracking_report.json
│   │   │       ├── blanket_order_tracking_report.py
│   │   │       └── (other business logic files)
│   │   ├── web_form/                  ← Portal web forms (one sub-folder each)
│   │   │   ├── __init__.py
│   │   │   ├── agent/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── agent.js
│   │   │   │   ├── agent.json
│   │   │   │   └── agent.py
│   │   │   ├── sales_invoice/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── sales_invoice.js
│   │   │   │   ├── sales_invoice.json
│   │   │   │   └── sales_invoice.py
│   │   │   └── ticket/
│   │   │       ├── __init__.py
│   │   │       ├── ticket.js
│   │   │       ├── ticket.json
│   │   │       └── ticket.py
│   │   │
│   │   ├── workspace/                 ← Workspace JSON definitions
│   │   │   └── automation/
│   │   │       └── automation.json
│   │   │
│   │   └── __init__.py
│   │
│   ├── commands/                      ← Custom `bench` CLI commands
│   │   ├── __init__.py
│   │   ├── export_fixtures.py
│   │   └── README.md
│   │
│   ├── config/                        ← App-level config (desktop icons etc.)
│   │   └── __init__.py
│   │
│   ├── fixtures/                      ← Exportable fixture JSON files
│   │   ├── account_closing_balance.json
│   │   ├── address.json
│   │   └── asset_capitalization_stock_item.json
│   │
│   ├── public/                        ← Static assets served by nginx/gunicorn
│   │   └── js/
│   │       ├── <app_name>.bundle.js   ← Webpack entry point (auto-loaded desk-wide)
│   │       ├── lead.js
│   │       ├── ticket_web_form.js
│   │       └── workflow_action.js
│   │
│   │
│   ├── templates/                     ← Jinja templates for portal pages
│   │   ├── pages/
│   │   │   └── __init__.py
│   │   └── __init__.py
│   │
│   ├── __init__.py
│   ├── hooks.py                       ← App hooks wiring everything together
│   ├── install.py                     ← Post-install setup (run once on install)
│   ├── modules.txt                    ← List of modules in this app
│   ├── patches.txt                    ← Migration patch list
│   └── utils.py                       ← Shared utility functions
│
├── scripts/                           ← Repo-level scripts (linting, CI)
│   └── check_max_lines.py
│
├── commitlint.config.js               ← Conventional commits config
├── license.txt
├── pyproject.toml                     ← PEP 517 build metadata
└── README.md
```

### Directory Purposes

| Directory | Purpose |
| --------- | ------- |
| `<module_name>/api/` | The **only** location for versioned, custom whitelisted endpoints not tied to a single DocType this app owns. See versioning pattern below. No `api/` folder or `api.py` file may exist anywhere else in the app. |
| `<module_name>/customization/` | Hook logic and thin business-logic files for DocTypes **owned by another app** (core Frappe/ERPNext, or a different installed app) that this app extends. Never place an `api.py` here — whitelisted endpoints belong under `<module_name>/api/`. |
| `<module_name>/dashboard/` | Chart / number-card / dashboard data providers. |
| `<module_name>/doctype/` | DocTypes **this app owns** — standard controller layout. Never place an `api.py` here — whitelisted endpoints belong under `<module_name>/api/`. |
| `<module_name>/print_format/` | Custom print format JSON (and JS if using a script-based format). |
| `<module_name>/report/` | Query / script reports. |
| `<module_name>/web_form/` | Portal-facing web forms. |
| `<module_name>/workspace/` | Desk workspace JSON. |
| `commands/` | Custom `bench` CLI commands for this app. |
| `config/` | App config (desktop icons, module config). |
| `fixtures/` | Data exported via `fixtures` in `hooks.py`, synced across sites/environments. |
| `public/js/` | Bundled client-side assets not tied to a single doctype form (global scripts, workflow actions). |
| `scripts/` | Repo tooling — not shipped to sites (e.g. `check_max_lines.py`). |
| `templates/` | Jinja templates and website page controllers. |
| `hooks.py` | App-level hook registration only — no business logic. |
| `boot_session.py` | Data injected into `frappe.boot` on session start. |
| `install.py` | `after_install`/`before_install` logic for `bench install-app`. |
| `modules.txt` | Registered module list — managed by Frappe, don't hand-edit casually. |
| `patches.txt` | Data migration patches run on `bench migrate`. |
| `utils.py` | App-wide utilities with no clear module home. Prefer a more specific module before adding here. |

### Key Conventions

| Area | Rule |
| ---- | ---- |
| **`<module_name>` naming** | The module directory must be named distinctly from `<app_name>`. Never reuse the app's own name as the module name. |
| **API location** | All API code — `api.py` files, `api/` folders, and versioned endpoint files — lives **only** under `<module_name>/api/`. It must never appear inside `doctype/`, `customization/`, or as a standalone folder anywhere else in the app. |
| **Whitelisting** | `@frappe.whitelist()` may only be used on functions inside `<module_name>/api/`. Never whitelist a method or function inside `doctype/<name>/<name>.py` or `customization/<name>/*.py` — controller and customization files hold plain, non-whitelisted logic; expose it via a thin wrapper in `<module_name>/api/`. |
| **API versioning** | All public endpoints live under `<module_name>/api/v1/`. Never put versioned logic directly in the module root. |
| **API docstrings** | Every `@frappe.whitelist()` function must have a docstring documenting a 2–3 line explanation, the endpoint path, HTTP method, parameters (name, type, required/optional, description), and the response format. Dotted paths use the `<app_name>.<module_name>` convention. See [api.md](./skills/frappe-app-dev/references/api.md) for the required template. |
| **Customization vs DocType** | Use `customization/` for overriding standard ERPNext/Frappe documents. Use `doctype/` for net-new custom DocTypes only. Neither folder may contain API code. |
| **Fixtures** | Keep fixture JSON files in `fixtures/`. Export via the custom `commands/export_fixtures.py` bench command. |
| **Public JS bundles** | `public/js/<app_name>.bundle.js` is the Webpack entry point. Additional form scripts go in the appropriate `doctype/` or `customization/` folder, **not** in `public/js/`. |
| **Print formats** | One sub-folder per format under `print_format/`. Each folder must contain `__init__.py` + the JSON definition. |
| **Web forms** | One sub-folder per form under `web_form/`. Each folder must contain `__init__.py`, `.json`, `.py` (server script), and `.js` (client script). |
| **Max line length** | Enforced by `scripts/check_max_lines.py`. Run it in CI and locally before committing. |
| **Commit messages** | Follow Conventional Commits (enforced by `commitlint.config.js`). |

---

## `customization/` vs `doctype/`

Both hold Python logic tied to a DocType, but **ownership** differs:

- **`doctype/<name>/`** — this app defines the DocType (has its own `.json`).
  The `.py` here is the actual `Document` subclass. Wire hooks as class
  methods (`before_save`, `on_submit`, etc.).
- **`customization/<name>/`** — the DocType is owned elsewhere (core Frappe,
  ERPNext, or another installed app). This app cannot add a `Document`
  subclass for it, so instead:
  - `<name>.py` holds `doc_events` hook functions (registered in `hooks.py`,
    not as class methods).
  - `utils.py` / feature-named files hold the actual logic.
  - `<name>.js` holds client-script customizations for that doctype's form.
  - Any whitelisted endpoint scoped to this customization still belongs
    under `<module_name>/api/` — **not** inside `customization/<name>/`.

```python
# customization/customer/customer.py — doc_events hook, NOT a Document subclass
from <app_name>.<module_name>.customization.customer.utils import sync_customer_kyc

def on_update(doc, method):
    sync_customer_kyc(doc)
```

```python
# hooks.py
doc_events = {
    "Customer": {
        "on_update": "<app_name>.<module_name>.customization.customer.customer.on_update"
    }
}
```

Note: neither `customer.py` nor `utils.py` here may contain `@frappe.whitelist()`
— any whitelisted endpoint touching Customer still lives under
`<module_name>/api/`, never inside `customization/`.

---

## Versioned `api/`

App-wide custom APIs (not scoped to one DocType) are versioned, and live
**exclusively** under `<module_name>/api/`:

```
<module_name>/
    api/
        v1/
            bank.py
            company.py
        api_error_handler.py    ← applies across all versions
        before_request.py       ← applies across all versions
        response_formatter.py   ← standard response-shape helper
```

- Each resource gets its own file under `v1/` (or the current version).
- Cross-cutting concerns (`api_error_handler.py`, `before_request.py`,
  `response_formatter.py`) live at the `api/` root — they apply across
  versions and must not be duplicated inside each version folder.
- `response_formatter.py` is where the `api_response(...)` helper belongs.
- When introducing `v2/`, keep `v1/` working — do not break existing clients.
- No other folder in the app (`doctype/`, `customization/`, module root, or
  app root) may contain an `api/` folder or an `api.py` file.

---

## Anti-patterns

- **Don't dump unrelated helpers into `utils.py`.** A single catch-all
  `utils.py` at the app root becomes a dumping ground. Prefer a file scoped
  to what the function does (`customization/customer/utils.py`,
  `api/v1/response_formatter.py`). Reserve the app-root `utils.py` for truly
  generic, cross-cutting helpers used by many unrelated modules.
- **Don't write `Document`-style controller code in `customization/`.** You
  don't own that DocType's class — use `doc_events` hook functions instead
  of trying to subclass or monkey-patch the controller.
- **Don't put repo tooling in `<app_name>/scripts/`.** Keep it at the
  repo-root `scripts/` so it's clearly excluded from what gets installed to
  a site.
- **Don't skip `fixtures/` for environment-portable config data** (custom
  roles, custom fields, workflow states). Hand-managing these via the UI on
  each site causes drift between dev / staging / production.
- **Don't create `api.py` files or `api/` folders inside `doctype/`,
  `customization/`, or the app root.** All API code lives in exactly one
  place: `<module_name>/api/`. This is a hard rule, not a preference —
  it keeps the whitelisted surface area auditable from a single location.
- **Don't use `@frappe.whitelist()` inside a controller or customization
  file.** A method like `approve()` on an `Expense` controller stays plain
  Python. If the client needs to call it, write a whitelisted wrapper under
  `<module_name>/api/` that loads the document and calls the method
  internally — see [api.md](./skills/frappe-app-dev/references/api.md).
- **Don't name the module directory the same as the app.** `<module_name>`
  must be a distinct, meaningful name — never a repeat of `<app_name>`.