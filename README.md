# M3L4 Project Manager Bot

This repository contains a lightweight Discord bot that helps you store and manage portfolio projects using an SQLite database. The bot is designed to be simple, beginner-friendly, and easy to extend.

Contents
- [Overview](#overview)
- [Features](#features)
- [Files of interest](#files-of-interest)
- [Quick start](#quick-start)
- [Commands (how to use)](#commands-how-to-use)
- [Database details & helpers](#database-details--helpers)
- [Examples / Usage Flows](#examples--usage-flows)
- [Development notes & tips](#development-notes--tips)
- [Safety & token](#safety--token)

## Overview

The bot stores projects (name, description, link, status), skills, and links between projects and skills. It uses `sqlite3` so there are no external DB dependencies — everything lives in a single `.db` file.

The goal is to be readable and copy/paste-friendly for beginners while still being useful for small portfolio workflows.

## Features

- Create and store projects with user association
- Add skills and link skills to projects
- List your projects
- Update project fields (name, description, url, status)
- Delete projects
- View full project info including linked skills (`!project_info`)
- Database helpers to safely add columns (uses `ALTER TABLE` when needed) so schema changes won't delete data
- Beginner-friendly DB helper methods to add/update/delete statuses & skills

## Files of interest
- [M3L4/Bot.py](M3L4/Bot.py) — the Discord bot and user-facing commands
- [M3L4/logic.py](M3L4/logic.py) — `DB_Manager` class (database helpers & methods)
- [M3L4/config.py](M3L4/config.py) — `DATABASE` path and `TOKEN` (keep token secret)

## Quick start

1. Put your Discord bot token in [M3L4/config.py](M3L4/config.py) (do not share or commit it):

```python
DATABASE = 'M3L3_projects.db'
TOKEN = 'your-bot-token-here'
```

2. Initialize the database and create tables (also adds a `screenshot` column if needed):

```bash
python M3L4/logic.py
```

3. Run the bot:

```bash
python M3L4/Bot.py
```

The bot will print a startup message when connected.

## Commands (how to use)

All commands are prefix `!` (default in `Bot.py`). Commands implemented:

- `!start` — Short greeting and shows `!info` help message.
- `!info` — Lists available commands and short usage notes.
- `!new_project` — Interactive flow to add a new project:
  - Bot asks for project name, link, and status.
- `!projects` — Lists your projects and links.
- `!skills` — Interactive flow to choose one of your projects and add an existing skill to it.
- `!delete` — Interactive flow to pick and delete one of your projects.
- `!update_projects` — Interactive flow to choose a project and update one attribute (name, description, url, or status).
- `!project_info` — Asks for a project name and returns its details: name, description, link, status, and a comma-separated list of skills.

Notes:
- Most interactive commands check that the message author is the same user and uses the same channel so the flow is private and predictable.

## Database details & helpers

`DB_Manager` in [M3L4/logic.py](M3L4/logic.py) provides these methods (simple and beginner-friendly):

- `create_tables()` — Creates `projects`, `skills`, `project_skills`, and `status` tables.
- `default_insert()` — Adds a small default set of skills and statuses (no duplicates).
- `insert_project(data)` — Insert one or more projects. `data` is a list of tuples matching `(user_id, project_name, url, status_id)`.
- `insert_skill(user_id, project_name, skill)` — Link an existing skill to a project for a user.
- `get_projects(user_id)` — Return all projects for a user.
- `get_project_info(user_id, project_name)` — Return full project info (name, description, url, status).
- `get_project_skills(project_name)` — Return a comma-separated string of skill names linked to the project.

Schema change helpers:
- `column_exists(table, column)` — Check whether a column exists.
- `add_column_if_not_exists(table, column, col_type)` — Runs `ALTER TABLE ... ADD COLUMN ...` only when needed.
- `add_screenshot_column()` — Convenience wrapper to add `screenshot TEXT` to `projects` safely.

Beginner helpers for status & skills:
- `add_status(status_name)`
- `delete_status(status_id)`
- `update_status(status_id, new_name)`
- `add_skill_name(skill_name)`
- `update_skill_name(skill_id, new_name)`
- `remove_skill_from_project(project_id, skill_name)`
- `list_statuses_simple()` and `list_skills_simple()` — return plain Python lists of names for easy UI display.

These helpers were intentionally written simply so beginners can copy and use them.

## Examples / Usage Flows

1) Add a new status (script or REPL):

```python
from M3L4.logic import DB_Manager
from M3L4.config import DATABASE

mgr = DB_Manager(DATABASE)
mgr.create_tables()
mgr.add_status('Menunggu Review')
```

2) Add a new skill and link it to a project (script):

```python
mgr.add_skill_name('Docker')
# assume you have a project with id 3
mgr.remove_skill_from_project(3, 'Docker')  # remove if needed
```

3) Show project info from the bot:

- In Discord, type `!project_info` and follow prompts to enter the project name. The bot replies with project name, description, link, status, and skills.

## Development notes & tips

- The database file path is defined in [M3L4/config.py](M3L4/config.py). If you want separate dev/production DBs, change this value.
- The bot uses `INSERT OR IGNORE` for default data so running initialization multiple times is safe.
- `ALTER TABLE` in SQLite can only add columns; this project uses `add_column_if_not_exists` to add the `screenshot` TEXT column safely.
- If you need to remove a column or do more complex migrations consider exporting, creating a new table, copying data, and renaming tables (advanced; not included here).

## Safety & token

- Never commit a real bot token to git. Keep your token in `M3L4/config.py` and add the file to `.gitignore` if you plan to push to a public repo.
- The bot shown here is intended for learning and small private servers.

## Want more?

If you'd like I can:
- Add a simple CLI script to manage the DB (add project, list, etc.)
- Add tests for a few `DB_Manager` methods
- Add an example Discord message transcript showing the interactive flows

---

Created to be clear and practical — copy/paste friendly for beginners. If you want the README shorter, translated, or with more screenshots, tell me which part to change.
# M3L4-BotDatabase
