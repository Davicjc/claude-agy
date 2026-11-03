# claude-agy

**[Leia em português](README.md)** | **English**

A **Claude Code** plugin that makes Claude and **agy** (Antigravity CLI, running Gemini models) work together.
Claude specifies, delegates, checks and decides. agy does the work where it is strongest (images, audio, video
and bulk work) and acts as an **independent code reviewer**.

<img src="plugins/agy/.claude-plugin/icon.png" width="96" alt="plugin icon">

---

## What it does

| Feature | How |
|---|---|
| **Delegate tasks** | `agy-tarefa` runs the request in agy in the background, with the allowed folders, a time limit and fixed safety rules in the prompt |
| **Don't trust "done"** | Compares `git status` before and after and lists **what actually changed**. Flags it (exit code 3) if agy makes a commit |
| **Always the current model** | `agy-modelo` reads `agy models` at run time and picks the newest version for each role (code, review, image, research, fast). A new Gemini release is used without editing anything |
| **Cross review** | `agy-revisar` sends a `git diff` to agy for a **read-only** review and gets a verdict and findings in a fixed format: severity, confidence, file:line, evidence and suggestion |
| **Parallel work** | The skill teaches Claude to split work into parts that share no files (one `git worktree` per part), run them at the same time and validate each one before merging |
| **Images, audio, video** | Banners, mockups, icons, WhatsApp voice note transcription, .mp4 analysis, straight from the file |
| **Everything logged** | Each run keeps the prompt, answer, JSON, tokens, model and changes in `~/.cache/agy-skill/execucoes/` |

Claude uses the skill on its own when the task calls for it (images, audio, large work, "get a second opinion"). You can also ask directly, for example:

- *"Have agy make the README banner using the brand colors."*
- *"Transcribe this WhatsApp audio in ~/Downloads."*
- *"Split this migration into parts and run them in parallel with agy."*
- *"Do a cross review of what changed since main."*

---

## Actually tested

| Test | Result |
|---|---|
| Fix a bug and add a test (`agy-tarefa`, fast model) | Done in 27 s; the git report confirmed the changed and new files |
| Review a diff with 3 planted problems (SQL injection, hard-coded password, division by zero), one in a new untracked file | **All 3 found**, each with a concrete failure scenario and a fix; verdict "reprovar" (reject) |
| Ask agy on purpose to make a commit | Detected: exit code 3, with the commit and its files listed |
| Forget `--dir` | Refused before calling agy |
| Simulate models that don't exist yet (`gemini-5-krypton-*`, `gemini-4.5-flash-*`) | Picked the newest ones by itself |
| This plugin's icon | Made by agy itself with `agy-tarefa --papel imagem` |
| **The skill reviewing itself** (`agy-revisar` on its own scripts) | 6 findings, **all confirmed and fixed**: infinite loop on an option without a value, swallowed `git diff` errors, edits to an already modified file going unnoticed, a stale cache hiding a logged-out agy, identical folders in parallel runs, and a timeout reported as success |

---

## Installation

### Requirements
- [Claude Code](https://claude.com/claude-code)
- **agy** (Antigravity CLI) installed and logged in: `agy models` must list the models
- `git`, `python3` and `bash` (standard on Linux and macOS)

### In Claude Code

In one step (Claude Code 2.1.275 or newer):

```
/plugin install agy --marketplace Davicjc/claude-agy
```

Or in two steps (any version):

```
/plugin marketplace add Davicjc/claude-agy
/plugin install agy@davicjc
```

Then run `/reload-plugins` (or start a new session). To check: ask *"list the agy models"*.

### Update

```
/plugin marketplace update davicjc
```

---

## The scripts (they also work on their own)

They live in `plugins/agy/skills/agy/scripts/`. Script names, options and messages are in Portuguese
(`--papel` = role, `--modo` = mode, `--nome` = label, `--prompt-arquivo` = prompt file).

```bash
# Which model to use (read from agy now)
agy-modelo --listar
agy-modelo codigo                      # -> e.g. gemini-4-argon-high

# Run a task that edits files
agy-tarefa --dir ~/project --papel codigo --nome "login screen" \
           --prompt-arquivo request.md --timeout 2400

# Read/analyze only (no edits)
agy-tarefa --dir ~/project --modo plan --papel pesquisa --prompt "Explain how the cache works"

# Cross review
agy-revisar --dir ~/project                      # everything since HEAD, new files included
agy-revisar --dir ~/project --base main          # since main
agy-revisar --dir ~/project --intervalo HEAD~3..HEAD --foco "concurrency"
```

Roles: `codigo` (code), `revisao` (review), `pesquisa` (research), `imagem` (image), `rapido` (fast).

**Force a model:** `AGY_MODELO_CODIGO=<id>` (per role) or `AGY_MODELO=<id>` (all roles).

**`agy-tarefa` exit codes:** `0` ok · `1` agy error or empty answer · `2` bad usage · `3` ok, but agy made a commit · `124` timed out.

---

## Built-in safety rules

- agy can only see and edit the folders passed with `--dir` (without `--dir`, the script refuses to run).
- Every request carries fixed rules: no commit, no push, don't delete work, no `sudo`, stay inside the folders, and stop and explain when something blocks the task.
- Never uses `--dangerously-skip-permissions`.
- Reviews run in `plan` mode (read-only).
- The skill tells Claude to **check everything** (diff, tests, screen) before saying it is done, and not to apply review findings without confirming each one.

Common problems and fixes: [`plugins/agy/skills/agy/references/problemas.md`](plugins/agy/skills/agy/references/problemas.md) (in Portuguese).

---

## License

MIT. Made by [Davi Castro](https://davicjc.com).
