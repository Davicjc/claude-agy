# claude-agy

**[Leia em português](README.md)** | **English**

[![tests](https://github.com/Davicjc/claude-agy/actions/workflows/testes.yml/badge.svg)](https://github.com/Davicjc/claude-agy/actions/workflows/testes.yml)

A **Claude Code** plugin that makes Claude and **agy** (Antigravity CLI, running Gemini models) work together.
Claude specifies, delegates, checks and decides. agy does the work where it is strongest (images, audio, video
and bulk work) and acts as an **independent code reviewer**.

<img src="plugins/agy/.claude-plugin/icon.png" width="96" alt="plugin icon">

---

## What it does

| Feature | How |
|---|---|
| **Delegate tasks** | `agy-tarefa` runs the request in agy in the background, with the allowed folders, a time limit and fixed safety rules in the prompt |
| **Don't trust "done"** | Compares git before and after (with a content hash of each file) and shows **what actually changed**. Flags it if agy makes a commit, times out or returns an empty answer |
| **Always the current model** | `agy-modelo` reads `agy models` at run time and picks the newest version for each role (code, review, image, research, fast). A new Gemini release is used without editing anything |
| **Cross review** | `agy-revisar` sends a `git diff` to agy for a **read-only** review and gets a verdict and findings in a fixed format: severity, confidence, file:line, evidence and suggestion |
| **Saves context** | Claude only gets the essentials of the answer (up to 150 lines). The full text stays in a file, read only when needed |
| **Parallel work** | The skill teaches Claude to split work into parts that share no files (one `git worktree` per part), run them at the same time and validate each one before merging |
| **Images, audio, video** | Banners, mockups, icons, WhatsApp voice note transcription, .mp4 analysis, straight from the file |
| **Accountability** | `agy-historico` lists runs with status, model, time and tokens. Each run keeps its prompt, answer and changes in `~/.cache/agy-skill/execucoes/` |

Claude uses the skill on its own when the task calls for it. You can also call it directly:

| Command | What it does |
|---|---|
| `/agy:delegar <task>` | Delegates to agy and validates the result before calling it done |
| `/agy:revisar [--base main]` | Cross review of the current changes; Claude checks each finding and says which ones are real |
| `/agy:modelos` | Models available in agy and which one each role uses |
| `/agy:historico [N]` | Latest runs and tokens spent |

Or in plain words: *"have agy make the README banner with the brand colors"*, *"transcribe this WhatsApp audio in ~/Downloads"*, *"do a cross review of what changed since main"*.

---

## Actually tested

**With the real agy:**

| Test | Result |
|---|---|
| Fix a bug and add a test | Done in 27 s; the git report confirmed the changed and new files |
| Review with 3 planted problems (SQL injection, hard-coded password, division by zero), one in a new untracked file | **All 3 found**, each with a failure scenario and a fix |
| Continue the same conversation (`--continuar`) | agy remembered what was said in the previous call |
| Ask agy on purpose to make a commit | Detected (exit code 3), with the commit and its files listed |
| This plugin's icon | Made by agy itself (`--papel imagem`) and checked before use |
| **The skill reviewing itself** (4 rounds) | 11 real findings in its own scripts, each one checked first and all fixed. One suggestion was **rejected**: turning off `safe.directory` as root would open a security hole |

**Automated** ([`tests/rodar.sh`](tests/rodar.sh), with a fake agy, no network and no cost): 94 checks on every push. That is 78 as a regular user, on **Linux** and on **macOS with its stock bash 3.2**, plus 16 that only make sense **as root**, on Linux. They cover model selection (including models that don't exist yet), bad options, unwanted commits, timeouts, empty answers, edits to an already modified file, a 150 KB request, long output being trimmed, parallel runs, file ownership (with hardlink protection), git refusing a repo owned by someone else, reviewing from a subfolder, large diffs and repositories with no commits.

---

## Installation

### Requirements

- [Claude Code](https://claude.com/claude-code)
- **agy** (Antigravity CLI) installed and logged in: `agy models` must list the models
- `bash`, `git` and `python3`

| System | Status |
|---|---|
| Linux | Tested with the real agy and in the automated tests |
| macOS | Automated tests with the stock bash 3.2. Optional: `brew install coreutils` for the extra external time limit |
| Windows | Through WSL (it is Linux); Git Bash was not tested |

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

Then run `/reload-plugins` (or start a new session) and try `/agy:modelos`.

### Update

```
/plugin marketplace update davicjc
```

---

## The scripts (they also work on their own)

They live in `plugins/agy/skills/agy/scripts/`. Script names, options and messages are in Portuguese
(`--papel` = role, `--modo` = mode, `--nome` = label, `--prompt-arquivo` = prompt file, `--continuar` = continue).

```bash
# Which model to use (read from agy now)
agy-modelo --listar --atualizar
agy-modelo codigo                      # -> e.g. gemini-4-argon-high

# A task that edits files
agy-tarefa --dir ~/project --papel codigo --nome "login screen" \
           --prompt-arquivo request.md --timeout 2400

# Read/analyze only (no edits)
agy-tarefa --dir ~/project --modo plan --papel pesquisa --prompt "Explain how the cache works"

# Continue the same conversation (id in meta.json)
agy-tarefa --dir ~/project --continuar <id> --prompt "Now fix the button"

# Cross review
agy-revisar --dir ~/project                      # everything since HEAD, new files included
agy-revisar --dir ~/project --base main          # since main
agy-revisar --dir ~/project --intervalo HEAD~3..HEAD --foco "concurrency"
agy-revisar --dir ~/project --exigir-aprovado    # exit code 4 unless approved (automation)

# History and tokens
agy-historico 20
```

**Roles:** `codigo` (code), `revisao` (review), `pesquisa` (research), `imagem` (image), `rapido` (fast). **Force a model:** `AGY_MODELO_CODIGO=<id>` (per role) or `AGY_MODELO=<id>` (all roles).

| Exit code | `agy-tarefa` | `agy-revisar` |
|---|---|---|
| `0` | ok | review done |
| `1` | agy error or empty answer | agy failed |
| `2` | bad usage | bad usage |
| `3` | ok, but agy made a commit | — |
| `4` | — | verdict not approved (with `--exigir-aprovado`) |
| `124` | timed out | — |

---

## Built-in safety rules

- agy can only see and edit the folders passed with `--dir` (without `--dir`, the script refuses to run).
- Every request carries fixed rules: no commit, no push, don't delete work, no `sudo`, stay inside the folders, stop and explain when something blocks the task.
- Never uses `--dangerously-skip-permissions`. Reviews run in `plan` mode (read-only).
- In reviews, the diff is delimited and comes with a notice that text inside the code is data, not instructions: a malicious comment cannot "tell" the reviewer to approve.
- When running as root in another user's folder, whatever agy creates from the terminal (which starts out owned by root) is given back to the folder's owner.
- The skill tells Claude to **check everything** (diff, tests, screen) before saying it is done, and not to apply review findings without confirming each one.

Common problems and fixes: [`references/problemas.md`](plugins/agy/skills/agy/references/problemas.md) (in Portuguese).

Version history: [`CHANGELOG.md`](CHANGELOG.md).

---

## License

MIT. Made by [Davi Castro](https://davicjc.com).
