Run basic Supabase migration commands from agent

description: Guided helper for basic Supabase setup and migration tasks for students. Use when the user wants to prepare a repo for Supabase, connect it to an existing Supabase cloud project, create a migration, push schema changes, migrate local SQLite data to Supabase, or check current setup status.
---

# Supabase Helper

This helper is a guided wrapper around the local `supabase-helper.sh` (macOS/Linux) or `supabase-helper.ps1` (Windows) script.

## Supported actions

1. Check Supabase setup status
2. Prepare this local project for Supabase
3. Connect this project to my Supabase cloud project
4. Create a database migration
5. Apply local schema changes to Supabase
6. Migrate my local app database to Supabase

## Core rule

You must EXECUTE the helper script directly (but only **after** the user has chosen a menu option—see **Menu-first workflow** below).
Do NOT tell the user to manually run terminal commands unless the script output explicitly says a required tool is missing **or** the `status` action prints OS-specific install instructions for the Supabase CLI (then relay those commands and ask the user to re-run `status` after installing).
Do NOT replace the helper with raw Supabase CLI commands unless the helper script itself fails.
The shell script is non-interactive. The chat flow handles choices.

## Menu-first workflow (required)

- If the user invokes this command **without** a clear choice (e.g. `/supabase-helper` alone, or no `1`–`6` / action name), you **must** present the menu and ask which option they want, then **stop**. Do **not** run `supabase-helper.sh`, `supabase-helper.ps1`, or any script action in that same turn.
- Wait for the **user’s next message** with their selection (e.g. `1`, `option 4`, `status` as the choice for item 1). **Then** execute the helper with the matching `<action>`.
- Exception: if the user’s message **already** includes an explicit choice (e.g. “run supabase-helper option 1”, “`/supabase-helper` then 1” in one line, or “1” for status when they are clearly answering a menu you just showed), run the corresponding script action without re-posting the full menu unless they asked for it.

## Menu

When invoked **without** a choice yet, present this menu exactly:

1. Check Supabase setup status
2. Prepare this local project for Supabase
3. Connect this project to my Supabase cloud project
4. Create a database migration
5. Apply local schema changes to Supabase
6. Migrate my local app database to Supabase

Then ask the user which option they want (1–6) and **wait for their reply** before running any helper script.

## Execution

On **macOS or Linux**, execute:

```bash
./.cursor/scripts/supabase-helper.sh <action>
```

On **Windows** (PowerShell), execute:

```powershell
powershell -ExecutionPolicy Bypass -File .\.cursor\scripts\supabase-helper.ps1 <action>
```

Where `<action>` is one of:
- `status`
- `setup`
- `list-projects`
- `connect --project-ref <ref>`
- `migration`
- `push`
- `migrate-local-db --source sqlite --db-path <path>`

## Option behavior

### 1. Check Supabase setup status

Run:

```bash
./.cursor/scripts/supabase-helper.sh status
```

On Windows:

```powershell
powershell -ExecutionPolicy Bypass -File .\.cursor\scripts\supabase-helper.ps1 status
```

Then summarize the current state in a concise table or bullet-style structure, including:
- Supabase CLI installed/logged in
- local Supabase folder initialized or not
- cloud project connected or not
- project ref if available
- project URL if available
- migration files present or not
- helper config file present or not

**If Supabase CLI is not installed:** the script prints **platform-specific** install steps. Summarize them **explicitly** (copy the exact commands the script prints—do not paraphrase as “install the CLI”). On **Windows**, `status` recommends **Scoop first**, then `scoop install supabase` via the Supabase bucket. On **macOS/Linux**, follow the script (`supabase-helper.sh`). Tell the user to run those commands **in their own terminal**, verify with `supabase --help`, then **run `status` again** until it shows `Supabase CLI installed: yes`.

If already connected, recommend the next likely step from this set:
- prepare local Supabase (if not initialized)
- connect to a cloud project (if not connected)
- create migration
- push schema
- migrate SQLite data
- reconnect to another project

Do NOT include stale wording like “push needs Docker” unless the script explicitly says so.

### 2. Prepare this local project for Supabase

Use this when the repo does not yet have local Supabase files.

Run:

```bash
./.cursor/scripts/supabase-helper.sh setup
```

Then summarize the result.

### 3. Connect this project to my Supabase cloud project

This is a guided chat flow, not a terminal prompt flow.

Run:

```bash
./.cursor/scripts/supabase-helper.sh list-projects
```

Expected result:
- clean JSON list of available projects, or
- a clear prerequisite error from the script.

If projects are returned, present them as a numbered list in chat using:
- project name
- project ref
- status
- region

Then also present one more option:
- Create a new Supabase project in the dashboard

If the user chooses an existing project:
- run:

```bash
./.cursor/scripts/supabase-helper.sh connect --project-ref <chosen-ref>
```

- then summarize the result.

If the user chooses to create a new project:
- instruct them to open:
  `https://supabase.com/dashboard/projects`
- create the project in the dashboard
- wait until it is ready
- reply `done`
- then rerun `list-projects`
- show the refreshed list
- continue with project selection and `connect --project-ref <chosen-ref>`

Do NOT claim that push needs Docker.
Do NOT ask the shell script to wait for user input.

### 4. Create a database migration

Use the default migration name automatically unless the user explicitly asks for another name.

Default migration name:
- `create_initial_schema`

Run:

```bash
./.cursor/scripts/supabase-helper.sh migration
```

If the user explicitly wants a custom name, use the script's supported flag form if available; otherwise explain that the current helper uses the default name.

After it succeeds, tell the user that the migration file now exists under `supabase/migrations` and the next normal step is usually **push** (or edit SQL first if they are customizing schema).

### 5. Apply local schema changes to Supabase

Use this when migration files already exist and the repo is already connected to a cloud project.

Run:

```bash
./.cursor/scripts/supabase-helper.sh push
```

This step should be described as pushing local migration SQL to the linked Supabase cloud project.
Do NOT say Docker is required for this course flow.
If the script itself reports a missing prerequisite, reflect that exactly.

### 6. Migrate my local app database to Supabase

For this class, default to SQLite.
The local DB remains intact after migration.
Supabase becomes the new source of truth afterward.

If the user gives a local DB path, run:

```bash
./.cursor/scripts/supabase-helper.sh migrate-local-db --source sqlite --db-path <path>
```

If the user does not give a path, ask for the SQLite DB path in chat.
Do not invent one.

If the script generates SQL instead of applying it automatically, explain clearly what was generated and what the student should verify in the Supabase dashboard.

## Notes for the agent

- The shell script is the executor. The chat is the menu: **never auto-run an action on the same turn as showing the menu** unless the user already stated their choice in that message (see Menu-first workflow).
- Query / inspect / verify can be demonstrated in the Supabase dashboard.
- Cloud project creation stays in the dashboard for this teaching flow.
- The helper-owned local memory file is `.supabase-helper.json`.
- If some earlier uploaded files are unavailable, work from the latest script and the current chat agreement.
