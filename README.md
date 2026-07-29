# 8848 Custom Skills

A collection of agent skills for building [Frappe Framework](https://frappeframework.com/) applications, plus general code-style, code-review, and UI design skills — for use with [Claude Code](https://claude.com/claude-code).

## Skills

| Skill                 | What it covers                                                                                                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `frappe-app-dev`      | Full-stack Frappe: DocTypes, controllers, APIs, database/ORM, hooks, permissions, jobs, realtime, caching, testing, app setup, frontend (Desk/Vue/portal), and the bench CLI + site management |
| `code-style`          | General code style rules                                                                                                                                                                        |
| `quality-code-review` | Performing code reviews, audits, or pull-request feedback                                                                                                                                       |
| `ui-design`           | General UI/UX design principles                                                                                                                                                                 |

## Repo layout

```
.
├── .claude/
│   └── skills/                  # the four skills above, one folder each
│       ├── frappe-app-dev/
│       ├── code-style/
│       ├── quality-code-review/
│       └── ui-design/
├── CLAUDE.md                    # standing instructions — read at the start of every session
└── project_base_template/       # templates used when scaffolding a new custom Frappe app
```

## Install (into your own app repo)

Claude Code auto-discovers project-scoped skills only under
`.claude/skills/<skill-name>/SKILL.md` at your repo root, and loads
`CLAUDE.md` at the repo root automatically at the start of every session.
To bring both into your own app repo, clone this repo to a scratch
location, copy the two pieces you need, then discard the clone:

```bash
git clone https://github.com/8848digital/skills.git /tmp/8848-skills

mkdir -p <your-app>/.claude
cp -r /tmp/8848-skills/.claude/skills <your-app>/.claude/skills
cp /tmp/8848-skills/CLAUDE.md <your-app>/CLAUDE.md

rm -rf /tmp/8848-skills
```

To bring in only specific skills, copy just those subfolders instead of
the whole `.claude/skills/` directory:

```bash
mkdir -p <your-app>/.claude/skills
cp -r /tmp/8848-skills/.claude/skills/frappe-app-dev <your-app>/.claude/skills/
cp -r /tmp/8848-skills/.claude/skills/code-style <your-app>/.claude/skills/
```

If you're setting up a full custom Frappe app (linters, pre-commit,
CodeGraph, `commands/`, `utils/api_handlers/`, etc.), see
[`project_base_template/custom_app_setup.md`](./project_base_template/custom_app_setup.md#410-agent-tooling--templates-skills-folder-claudemd-commands-utilsapi_handlers)
§4.10, which covers this as one step of the full setup.

## Usage

Once `.claude/skills/` and `CLAUDE.md` are in place in your app repo, each
skill activates automatically when Claude Code matches a task to it —
creating DocTypes, building a Vue SPA, or running `bench migrate`
(`frappe-app-dev`); enforcing code style (`code-style`); reviewing a PR
(`quality-code-review`); UI/UX judgment (`ui-design`). `CLAUDE.md`
documents when each skill activates and the custom-app structure
conventions referenced throughout them.
