<!--
Copyright (c) 2026 8848 Digital LLP. All rights reserved.
Proprietary and confidential. Unauthorized copying, distribution, or use
of this file, via any medium, is strictly prohibited without prior
written permission from 8848 Digital LLP.
-->

# Setup

This guide tells you how to add the 8848 skills to a Frappe app, and how
to keep them up to date. Each case has manual steps and a prompt that you
can give to Claude Code.

## Contents

1. [What goes into an app](#what-goes-into-an-app)
2. [Before you start](#before-you-start)
3. [Find your case](#find-your-case)
4. [Case A: Create a new app with the skills](#case-a-create-a-new-app-with-the-skills)
5. [Case B: Add the skills to an existing app](#case-b-add-the-skills-to-an-existing-app)
6. [Case C: Update the skills in an app](#case-c-update-the-skills-in-an-app)
7. [Change the skills](#change-the-skills)
8. [Troubleshooting](#troubleshooting)

## What goes into an app

The setup copies five parts of this repo into the app repo:

| From this repo | To the app repo | What it does |
| -------------- | --------------- | ------------ |
| `.claude/skills/` | `.claude/skills/` | The skills. Claude Code finds them at this path. |
| `CLAUDE.md` | `CLAUDE.md` | The rules that Claude Code reads at the start of each session. |
| `project_base_template/commands/` | `<app_name>/commands/` | The `8848-export-fixtures` bench command. |
| `project_base_template/api_handlers/` | `<app_name>/utils/api_handlers/` | The shared API response helpers. |
| `scripts/sync_skills.sh` | `scripts/sync_skills.sh` | The script that updates the four parts above. |

`commands/` and `api_handlers/` also need entries in `hooks.py`. The
setup adds these entries. The update script never changes `hooks.py`.

## Before you start

Make sure that you have:

- a bench, with the virtual environment active (`source env/bin/activate`
  in the bench folder)
- read access to `https://github.com/8848digital/skills` from the
  terminal (an SSH key, or `gh auth login`)
- `git`, `bash` and `rsync`
- Claude Code, if you want to use the prompts

Do all work inside one app folder, `apps/<app_name>`. Do not run the
setup or the update script in the bench folder.

## Find your case

| Your app | Case |
| -------- | ---- |
| The app does not exist yet. | [Case A](#case-a-create-a-new-app-with-the-skills) |
| The app exists, and has no `.claude/skills/` and no `CLAUDE.md`. | [Case B](#case-b-add-the-skills-to-an-existing-app) |
| The app has `.claude/skills/` or `CLAUDE.md`. | [Case C](#case-c-update-the-skills-in-an-app) |

To find your case, run this in the app folder:

```bash
ls -d .claude/skills CLAUDE.md scripts/sync_skills.sh 2>/dev/null
```

No output means Case B. Any output means Case C.

## Case A: Create a new app with the skills

A new app gets the full setup: the skills, the linters, pre-commit, the
CI workflow and the shared templates. The full guide is
[`project_base_template/custom_app_setup.md`](./project_base_template/custom_app_setup.md).

### Manual steps

1. Go to the bench folder.
2. Create the app:

   ```bash
   bench new-app <app_name>
   ```

3. Do every section of
   [`custom_app_setup.md`](./project_base_template/custom_app_setup.md),
   in order. Section 4.10 copies the five parts from
   [What goes into an app](#what-goes-into-an-app).
4. Do the `hooks.py` steps in sections 4.7 and 4.9.
5. Delete the default module that `bench new-app` makes (a module with the
   same name as the app), if nothing uses it. `CLAUDE.md` tells why.
6. Commit.

### Prompt for Claude Code

Start Claude Code in the bench folder. Replace the values in `<...>`,
then give this prompt:

````text
Create a new Frappe app and set it up with the 8848 skills.

App details:
- App name (snake_case): <app_name>
- App title: <App Title>
- Description: <one line about what the app does>
- Publisher: 8848 Digital LLP
- Email: <email>
- Modules (each namespaced to the app name, never the app name itself):
  <for example: <app_name>_core, <app_name>_billing>

Do these steps:
1. Clone the skills repo to a temp folder:
   git clone --depth 1 --branch 8848-skills https://github.com/8848digital/skills.git /tmp/8848-skills
2. Read /tmp/8848-skills/CLAUDE.md and
   /tmp/8848-skills/project_base_template/custom_app_setup.md completely.
3. Run `bench new-app <app_name>` in this bench folder.
4. Do every section of custom_app_setup.md for apps/<app_name>, in order.
   Use Version B of the CI workflow (section 5.2).
5. Section 4.10: copy .claude/skills/, CLAUDE.md, commands/,
   utils/api_handlers/ and scripts/sync_skills.sh into the app.
   Replace <app_name> in the three api_handlers files.
6. hooks.py: add `commands`, `custom_fixtures` and `after_request` as in
   sections 4.7 and 4.9. For more than one module, use ONE
   custom_fixtures entry with an "in" filter, because the export writes
   one file per DocType:
   custom_fixtures = [{"dt": "Custom Field", "filters": {"module": ["in", [<all modules>]]}}]
7. Create the modules listed above, each with its own README.md. Delete
   the default module named after the app if nothing uses it.
8. Add license.txt and the copyright headers from licensing.md.
9. Add this to the top of .pre-commit-config.yaml, so the linters do not
   reformat the synced files:
   exclude: |
     (?x)(
         node_modules|.git|
         ^\.claude/|
         ^<app_name>/commands/(__init__|export_fixtures)\.py$|
         ^<app_name>/utils/api_handlers/(envelope|error_messages|response_formatter)\.py$
     )
10. Run pre-commit on all files and fix what fails. Do not reformat the
    excluded files.
11. Delete /tmp/8848-skills.
12. Do not commit or push. Show me a summary of what you created.
````

## Case B: Add the skills to an existing app

An existing app usually has its linters and CI already. Case B adds only
the five parts. The update script cannot do this. It only updates an app
that has the setup.

> **Warning:** `after_request` in `hooks.py` changes the response format
> of every `/api/method/<app_name>...` endpoint. If other systems call
> the app's API, the change can break them. Wire `after_request` only
> when you are ready to move every API client to the new format.

### Manual steps

1. Go to the app folder:

   ```bash
   cd apps/<app_name>
   ```

2. Make sure that `git status` is clean, then create a branch.
3. Clone the skills repo to a temp folder:

   ```bash
   git clone --depth 1 --branch 8848-skills \
       https://github.com/8848digital/skills.git /tmp/8848-skills
   ```

4. Copy the five parts. Set `APP` to the app's package name first:

   ```bash
   APP=<app_name>          # for example: APP=easysell
   K=/tmp/8848-skills

   mkdir -p .claude && cp -r $K/.claude/skills .claude/skills
   cp $K/CLAUDE.md CLAUDE.md

   mkdir -p $APP/commands
   cp $K/project_base_template/commands/{__init__.py,export_fixtures.py,README.md} \
       $APP/commands/

   mkdir -p $APP/utils/api_handlers
   touch $APP/utils/__init__.py $APP/utils/api_handlers/__init__.py
   for f in envelope.py error_messages.py response_formatter.py; do
       sed "s/<app_name>/$APP/g" $K/project_base_template/api_handlers/$f \
           > $APP/utils/api_handlers/$f
   done

   mkdir -p scripts
   cp $K/scripts/sync_skills.sh scripts/ && chmod +x scripts/sync_skills.sh
   ```

   The `sed` command replaces the `<app_name>` text in the templates with
   the app's package name.

5. If the app already has a `commands/__init__.py` that registers other
   commands, do not overwrite it. Add `export_fixtures` to its `commands`
   list instead.
6. Add these lines to `<app_name>/hooks.py`, after the `app_*` lines. Put
   every module from `modules.txt` in the list:

   ```python
   custom_fixtures = [
   	{
   		"dt": "Custom Field",
   		"filters": {"module": ["in", ["<Module 1>", "<Module 2>"]]},
   	}
   ]

   commands = ["<app_name>.commands.export_fixtures.export_fixtures"]
   ```

7. Add `after_request` only if the warning above allows it:

   ```python
   after_request = ["<app_name>.utils.api_handlers.response_formatter.format_frappe_response_to_custom"]
   ```

8. If the app uses pre-commit, add the `exclude` block from
   [Troubleshooting](#black-or-another-linter-reformats-the-synced-files)
   to `.pre-commit-config.yaml`.
9. Make sure that bench finds the command:

   ```bash
   cd ../.. && bench --help | grep 8848-export-fixtures
   ```

10. Delete the temp folder:

    ```bash
    rm -rf /tmp/8848-skills
    ```

11. Commit. Include `.claude/skills/.synced-from-8848-skills` after the
    first update (Case C) creates it.

### Prompt for Claude Code

Start Claude Code in the app folder (`apps/<app_name>`). Give this
prompt:

````text
Add the 8848 skills setup to this existing Frappe app. Follow
project_base_template/custom_app_setup.md section 4.10 from the skills
repo, with the rules below.

Do these steps:
1. Check that this folder is a Frappe app repo root (it has
   <app>/hooks.py) and that `git status` is clean. If it is not clean,
   stop and tell me. Create a branch named chore/setup-8848-skills.
2. Clone the skills repo to a temp folder:
   git clone --depth 1 --branch 8848-skills https://github.com/8848digital/skills.git /tmp/8848-skills
3. Read /tmp/8848-skills/CLAUDE.md and section 4.10 of
   /tmp/8848-skills/project_base_template/custom_app_setup.md.
4. Copy .claude/skills/, CLAUDE.md, project_base_template/commands/,
   project_base_template/api_handlers/ (to <app>/utils/api_handlers/,
   with <app_name> replaced by the app's package name) and
   scripts/sync_skills.sh into this app. Keep files that already exist in
   the app. If <app>/commands/__init__.py already registers other
   commands, add export_fixtures to it instead of replacing it.
5. hooks.py: add `commands` and ONE `custom_fixtures` entry that covers
   every module in modules.txt with an "in" filter. Place them after the
   app_* lines.
6. after_request: count the @frappe.whitelist() endpoints in the app.
   If there are any, do NOT add after_request. Tell me how many there
   are and ask me first, because it changes the response format of every
   endpoint.
7. If .pre-commit-config.yaml exists, add the synced files to its
   top-level exclude (.claude/, the two template files in commands/ and
   the three files in utils/api_handlers/). Keep the existing exclude
   patterns working exactly as before.
8. Run pre-commit on the changed files with the bench Python
   (env/bin/python -m pre_commit run). Fix what fails.
9. Check that `bench --help` lists 8848-export-fixtures.
10. Delete /tmp/8848-skills.
11. Do not commit or push. Show me a summary and anything you skipped.
````

## Case C: Update the skills in an app

The update script gets the latest `8848-skills` branch and updates the
five parts in the app. It does not change `hooks.py`, and it does not
commit.

### The app has `scripts/sync_skills.sh`

1. Go to the app folder, and commit or stash your changes:

   ```bash
   cd apps/<app_name>
   git status
   ```

2. Show what will change:

   ```bash
   scripts/sync_skills.sh --dry-run
   ```

3. Update:

   ```bash
   scripts/sync_skills.sh
   ```

4. Review the changes with `git status` and `git diff`, then commit.

The script clones this repo to a temp folder, runs the newest version of
itself from that clone, then deletes the clone. You do not need a clone
of this repo.

### The app has the skills but no `scripts/sync_skills.sh`

The app got the skills before the script existed. Run the script once
from a clone of this repo. The script then copies itself into the app.

```bash
git clone --depth 1 --branch 8848-skills \
    https://github.com/8848digital/skills.git /tmp/8848-skills
/tmp/8848-skills/scripts/sync_skills.sh apps/<app_name>
rm -rf /tmp/8848-skills
```

Then review and commit, as in step 4 above.

### What the script does

- It updates `.claude/skills/`, `CLAUDE.md`, `<app_name>/commands/`,
  `<app_name>/utils/api_handlers/` and `scripts/sync_skills.sh`.
- It keeps skills that the app added itself, and other files in
  `commands/` and `utils/api_handlers/`.
- It removes a skill from the app when the skill is removed from this
  repo. It uses `.claude/skills/.synced-from-8848-skills` to know which
  skills came from this repo. Commit this file.
- It skips `commands/` or `utils/api_handlers/` if the app does not have
  that folder.
- It replaces `commands/__init__.py` with the template. If the app
  registers other commands in this file, add them back before you commit.

| Option | What it does |
| ------ | ------------ |
| `--dry-run` | Shows what will change, and changes nothing. |
| `--force` | Updates also when the updated paths have uncommitted changes. These changes are lost. |
| `--no-pull` | Does not pull first. Use it only when you run the script from a clone of this repo. |
| `-h`, `--help` | Shows the help. |

| Environment variable | Default | What it does |
| -------------------- | ------- | ------------ |
| `SKILLS_REPO_URL` | `https://github.com/8848digital/skills.git` | The repo to clone. |
| `SKILLS_BRANCH` | `8848-skills` | The branch to clone. |

### Prompt for Claude Code

Start Claude Code in the app folder (`apps/<app_name>`). Give this
prompt:

````text
Update the 8848 skills in this Frappe app to the latest version.

Do these steps:
1. Run `git status`. If .claude/skills, CLAUDE.md, <app>/commands,
   <app>/utils/api_handlers or scripts/sync_skills.sh have uncommitted
   changes, stop and tell me.
2. If scripts/sync_skills.sh exists, run `scripts/sync_skills.sh --dry-run`
   and show me the result. Then run `scripts/sync_skills.sh`.
3. If scripts/sync_skills.sh does not exist, clone the skills repo:
   git clone --depth 1 --branch 8848-skills https://github.com/8848digital/skills.git /tmp/8848-skills
   then run /tmp/8848-skills/scripts/sync_skills.sh <this app path>,
   then delete /tmp/8848-skills.
4. Check the diff of <app>/commands/__init__.py. If the update removed
   commands that the app registered, add them back.
5. If .pre-commit-config.yaml exists, check that its top-level exclude
   covers the synced files. If not, add them as in the skills repo
   SETUP.md (Troubleshooting).
6. Read the new CLAUDE.md. Tell me about rules that changed and that may
   need changes in this app, for example new hooks.py entries.
7. Do not commit or push. Show me a summary of what changed.
````

## Change the skills

Make all changes in this repo. Do not edit the copied files in an app,
because the next update overwrites them.

1. Create a branch from `8848-skills` in this repo.
2. Change the skills, `CLAUDE.md`, the templates in
   `project_base_template/` or `scripts/sync_skills.sh`.
3. Open a pull request into `8848-skills` on `8848digital/skills`. This
   repo is a fork of `frappe/skills`, so use
   `gh pr create --repo 8848digital/skills --base 8848-skills`.
4. After the merge, run [Case C](#case-c-update-the-skills-in-an-app) in
   each app.

## Troubleshooting

### "uncommitted changes in the paths above"

The app has changes in a path that the script updates. Commit or stash
them, then run the script again. Use `--force` only if you want to lose
these changes.

### "has no .claude/skills or CLAUDE.md; this script only updates"

The app does not have the setup. Use
[Case B](#case-b-add-the-skills-to-an-existing-app).

### "is a bench" or "is not the app repo root"

Run the script in the app folder `apps/<app_name>`, not in the bench
folder and not in a sub-folder of the app.

### "could not clone"

Your terminal has no read access to the skills repo. Do `gh auth login`
or add an SSH key, then make sure that this command works:

```bash
git ls-remote https://github.com/8848digital/skills.git 8848-skills
```

### black or another linter reformats the synced files

If the linter changes the synced files, each update shows a diff. Add
the synced files to the top-level `exclude` in `.pre-commit-config.yaml`.
Keep your existing patterns in the list. Replace `<app_name>` with the
package name:

```yaml
exclude: |
  (?x)(
      node_modules|.git|
      ^\.claude/|
      ^<app_name>/commands/(__init__|export_fixtures)\.py$|
      ^<app_name>/utils/api_handlers/(envelope|error_messages|response_formatter)\.py$
  )
```

### `pre-commit: command not found`

pre-commit is installed in the bench virtual environment. Run it with
the bench Python:

```bash
../../env/bin/python -m pre_commit run
```
