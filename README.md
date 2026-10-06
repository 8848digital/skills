# 8848 Custom Skills

A collection of agent skills for building [Frappe Framework](https://frappeframework.com/) applications, plus general code-style, code-review, and UI design skills — for use with [Claude Code](https://claude.com/claude-code).

## Skills

| Skill                 | What it covers                                                                                                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `frappe-app-dev`      | Full-stack Frappe: DocTypes, controllers, APIs, database/ORM, hooks, permissions, jobs, realtime, caching, testing, app setup, frontend (Desk/Vue/portal), and the bench CLI + site management |
| `code-style`          | General code style rules                                                                                                                                                                        |
| `quality-code-review` | Performing code reviews, audits, or pull-request feedback                                                                                                                                       |
| `ui-design`           | General UI/UX design principles                                                                                                                                                                 |
| `fix-issue`           | Fix a GitHub issue in three steps: extract the facts, reproduce and fix with a test, then validate the fix in a separate context                                                               |
| `deep-app-audit`      | Multi-agent audit of a Frappe app for security, correctness, and customization defects. Every candidate is verified, and the result is one report. User-invoked only: run `/deep-app-audit <app path>` |
| `technical-writing`   | Write documentation, READMEs, commits, pull requests, and release notes in Simplified Technical English |
| `draft-security-advisory` | Write a publication-ready GitHub Security Advisory (GHSA) from a vulnerability report. User-invoked only: run `/draft-security-advisory` |
| `resolve-backport-conflicts` | Resolve the conflict markers that Mergify commits into a failed backport pull request. User-invoked only: run `/resolve-backport-conflicts <PR>` |

## Repo layout

```
.
├── .claude/
│   └── skills/                  # the skills above, one folder each
│       ├── frappe-app-dev/
│       ├── code-style/
│       ├── quality-code-review/
│       ├── ui-design/
│       ├── fix-issue/
│       ├── deep-app-audit/
│       ├── technical-writing/
│       ├── draft-security-advisory/
│       └── resolve-backport-conflicts/
├── CLAUDE.md                    # standing instructions — read at the start of every session
├── SETUP.md                     # set up or update the skills in an app, with Claude Code prompts
├── project_base_template/       # templates used when scaffolding a new custom Frappe app
└── scripts/
    └── sync_skills.sh           # update an app repo from this repo
```

## Install (into your own app repo)

For the full guide, with a Claude Code prompt for each case (new app,
existing app, update), see [SETUP.md](./SETUP.md).

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

## Update an app repo

`scripts/sync_skills.sh` updates an app repo that is already set up (see
[`custom_app_setup.md`](./project_base_template/custom_app_setup.md) §4.10).
It does not set up a new app. It gets the latest version of this repo,
then updates these parts of the app:

| From this repo | To the app |
| -------------- | ---------- |
| `.claude/skills/` | `.claude/skills/` |
| `CLAUDE.md` | `CLAUDE.md` |
| `project_base_template/commands/` | `<app_name>/commands/` |
| `project_base_template/api_handlers/` | `<app_name>/utils/api_handlers/` (with `<app_name>` replaced) |
| `scripts/sync_skills.sh` | `scripts/sync_skills.sh` (the script itself) |

**Each time this repo changes**, go to the app repo and run the copy of
the script that is in the app:

```bash
cd ~/frappe-bench/apps/<your-app>
scripts/sync_skills.sh              # update the app
scripts/sync_skills.sh --dry-run    # only show what would change
```

The script clones the latest version of this repo to a temp folder, runs
the script from that clone, then deletes the clone. Thus the newest
version of the script always runs, and the app gets the new script too.
The script clones the `8848-skills` branch, and uses your normal git
access to GitHub. To use a different repo or branch, set
`SKILLS_REPO_URL` or `SKILLS_BRANCH`.

**First time, for an app that does not have `scripts/sync_skills.sh` yet**
(an app set up before the script existed), run the script once from a
clone of this repo:

```bash
git clone https://github.com/8848digital/skills.git /tmp/8848-skills
/tmp/8848-skills/scripts/sync_skills.sh ~/frappe-bench/apps/<your-app>
rm -rf /tmp/8848-skills
```

The script runs at app level only. It refuses a bench root, a sub-folder
of an app, a folder that is not a Frappe app repo, and an app that has no
`.claude/skills/` or `CLAUDE.md` yet. It keeps project-local skills and
other files in `commands/` and `utils/api_handlers/`. It removes a skill
from the app when that skill is removed from this repo. It skips
`commands/` or `utils/api_handlers/` if the app does not have that folder.
It stops if the app has uncommitted changes in the updated paths, unless
you add `--force`. It does not commit. Review and commit the changes in
the app. Run `scripts/sync_skills.sh --help` for all options.

## Usage

Once `.claude/skills/` and `CLAUDE.md` are in place in your app repo, each
skill activates automatically when Claude Code matches a task to it —
creating DocTypes, building a Vue SPA, or running `bench migrate`
(`frappe-app-dev`); enforcing code style (`code-style`); reviewing a PR
(`quality-code-review`); UI/UX judgment (`ui-design`); fixing a GitHub
issue (`fix-issue`); writing docs or commit messages
(`technical-writing`). `deep-app-audit`, `draft-security-advisory`, and
`resolve-backport-conflicts` run only when you call them with their slash
command. `CLAUDE.md`
documents when each skill activates and the custom-app structure
conventions referenced throughout them.
