# Nuclear Developer

Local AI agents for [NuclearLang](https://github.com/nuclearlang/nuclear) and Dragonfly, on models of at most 3B
parameters, as the junior developer of a frontier model: **coder**, **tester**, **security reviewer** and **planner**.
What a small model cannot hold in its head comes from the [Nuclear Smart Graph](https://github.com/nuclearlang/nuclear-graph)
(the ecosystem's private knowledge base); what it writes is checked by the Nuclear compiler before it answers.

It is itself a Dragonfly backend written in Nuclear (platform `backend`, dialect `fastapi`): FastAPI + LangGraph,
generated from the `.nuc` files of this repository. Decision record:
[ADR-0102](https://github.com/nuclearlang/nuclear/blob/main/docs/adr/0102-ai-loop.md).

## How an agent works

A small model follows fixed steps better than it chooses tools, so each agent is a LangGraph graph with fixed steps:

```
recall ──► write ──► nuclear check ──ok──► answer
  │          ▲            │
  │          └── repair ◄─┘ errors (at most 3 attempts)
  └─ from the Smart Graph: the validated prompt of the use case, the closest knowledge
     (specs, words of the dialect), solved examples (distilled from Claude) and example files
```

| Agent | Model | Route | Does | Does not |
|---|---|---|---|---|
| Coder | `qwen2.5-coder:3b` | `POST /developer/code` | writes ONE `.nuc` file from a precise task, repairs it with the compiler | choose names, data or architecture |
| Tester | `qwen2.5-coder:3b` | `POST /developer/test` | writes the Nuclear tests (`tests` kind) of a piece of code | delete a failing test |
| Security | `gemma4:e2b` | `POST /developer/security` | reviews code against the platform's security rules + the compiler | change the code |
| Documenter | `gemma4:e2b` | `POST /developer/document` | reads a run's journal and writes what was done: what it built, how it checked it, what is left (only facts of the journal) | change the code |
| Assistant | `gemma4:e2b` | `POST /developer/chat` | answers a message of the editor's chat about the project, from the project's map, the session's last messages and the graph | change a file (an order is a Loop run) |
| Diagnoser | `gemma4:e2b` | `POST /developer/diagnose` | when a pipeline stage fails: the kind of failure by fixed rules (transient, environment, code, test, spec), the evidence from the tools (`nuclear check`, `dragonfly doctor`) and the next step in plain words | change anything |
| Planner | `gemma4:e2b` | `POST /developer/plan` | splits a spec into pieces of one file, in order; each path comes from the platform's layout (`nuclear help layout`), each kind and forgotten piece from the spec's own words | choose a folder, or plan what the spec does not name |

Chat: `{message, platform, map, history}` → the same answer (`report`). Request (`code`, `test`, `security`, `document`: the journal in `context`): `{task, platform = "backend", path, kind, context}`; answer: `{code, report,
check, attempts, trace: [{step, detail}]}`. Plan: `{spec, platform}` → `{pieces: [{path, kind, task}], raw, trace}`.
Each request runs on a thread of its own. **The agents write only Nuclear**: they name a platform (`backend`, `mobile`,
`web`), never a dialect, read the graph's `nuclear` collection (no target-language code) and check with
`nuclear check --platform` (ADR-0102 §11).

The sources: `components/developer/ai/` (`workbench.nuc` — what every agent shares —, `coder.nuc`, `tester.nuc`,
`security.nuc`, `documenter.nuc`, `assistant.nuc`, `diagnoser.nuc`, `planner.nuc`, `states/piece_state.nuc`), `components/developer/domain/models/`,
`components/developer/api/developer_controller.nuc`, `config/project_config.nuc` (the two Ollama models),
`config.nuc` (`dependencies(nuclear_graph = "../nuclear-graph")`), `defs/` (the type definitions of
`nuclear_graph.client` and `nuclear_graph.toolchain`, generated with `tools/defs/python/gen.py`).

**The prompts are files**, never strings in the code: every prompt a model reads is
`nuclear-graph/src/nuclear_graph/prompts/agents/<step>.md` (`{{name}}` filled by `Workbench.prompt`, an empty `## `
section dropped; the list is that folder's README), over the use-case guides `prompts/<use case>.md`.

## Run it

```sh
ollama pull qwen2.5-coder:3b && ollama pull gemma4:e2b
nuclear-graph serve &                                   # the Smart Graph (see its README)
nuclear-graph connect nuclear-developer --scopes read,write
dragonfly build && dragonfly install                    # build/ from the .nuc files
cd build && .venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 7879
# or, from the nuclear working tree: ./install.sh --ai, then `nuclear-developer` (NUCLEAR_DEVELOPER_PORT)
```

```sh
curl -s -X POST 127.0.0.1:7879/developer/code -H 'content-type: application/json' \
  -d '{"task": "A model NewTask with a title (string, at least 3 characters) and a priority (int, default 2).",
       "path": "components/tasks/domain/models/task_models.nuc", "kind": "model"}'
```

Models: `CODER_MODEL`, `ASSISTANT_MODEL`, `OLLAMA_URL` (the editor's *Settings and integrations* page starts it with
the models chosen there). Each model runs with a context window of 8192 tokens (`numCtx` in
`config/project_config.nuc`; the editor's `nuclear.ai.contextTokens` says the same for its chat meter).

## What it knows

The knowledge base of the whole project is [`data/`](data/README.md): its own repository (`nuclear-knowledge`), every
document, example and tool help of the working tree copied byte for byte and ordered step by step (`data/INDEX.md`).
The Smart Graph ingests it; the agents read every section but the ones about the target languages and the dialects
underneath. Rebuild it after a change: `nuclear-graph knowledge build`, then `nuclear-graph ingest data`.

## Where it works

| Who calls it | How | What the Developer does |
|---|---|---|
| The **Loop** (VS Code `Ctrl+Alt+P`, Android Studio *Nuclear Loop*, `nuclear-loop run`) | spec → plan → tests first → code → verification → security → artifacts → deploy | the planner splits the spec, the tester writes the tests first (and fixes a test file the project rejects), the coder writes each piece and repairs it with the compiler and with failing tests told in Nuclear (`dragonfly explain`), the security reviewer reads each new file; when a stage fails the **diagnoser** says why; at the end the **documenter** writes the journal's summary |
| The **chat** (VS Code `Ctrl+Alt+A`, *Nuclear agent*) | a question, or `/task <order>` | the **assistant** answers from the project's map and the graph; an order is a Loop run with the local agents only |
| A **frontier model** over MCP (Claude Code, Codex, Cursor: `nuclear-graph mcp`) | `developer_task` first, then validate, correct, `developer_review`; `developer_retry` after a failure | the whole task by the local agents (`nuclear-loop run --lead local`), documented in the run's journal; the lead's corrected files become graph examples |

**Every run is documented.** `.nuclear/loop/runs/<id>.md` (beside the `.json` with every log whole): the documenter's
summary (what it built, how it checked it, what is left), why it failed (the diagnoser), the files written, what is
left for the reviewer, and each step with its agent, its trace and what it read; a frontier model's review is
`<id>.review.md`.

**When something fails.** A call to the Developer that fails for a passing reason (down, busy, 408/425/429/5xx) is
retried by its caller with exponential backoff and full jitter (5 tries, 1–30 s by default; `Retry-After` wins); a
compiler error or a failing test is never retried blindly — it goes to the repair rounds, then to the diagnoser (kind by
rules: transient, environment, code, test, spec; evidence from `nuclear check` or `dragonfly doctor`; the next step).
A failed run goes on from the step that failed (*↻ Retry*, `nuclear-loop retry <run>`, `developer_retry`): every step
that passed is kept, the earlier attempt stays in the run file.

**When the editor closes in the middle.** A run keeps a trace log (`.nuclear/loop/runs/<id>.log`), a heartbeat and a
ledger of the files it writes; an orphaned run is recovered (its unfinished step's files undone) and goes on from that
step. A call to the Developer nothing works on (the Developer silent, Ollama holding no model) is cut and retried, and
every run first checks that the Developer, the graph, Ollama and the models it needs answer.

**What the agents may read.** Only Nuclear and the platform (`backend`, `mobile`, `web`): the graph's `nuclear`
collection has no target-language code, a failing test reaches them told in `.nuc` terms, a piece's errors are given by
line, and the diagnoser's prompt forbids speaking of the generated code.

## Evaluation: how it was trained

There is no dedicated GPU here, so the agents are not trained by changing weights: they are trained by **retrieval
and distillation**, measured by `eval/run.py` over the tasks of `eval/tasks.toml`, each with acceptance texts (`must`): 42 tasks of one
file (every backend kind, and the mobile and web platforms) and 3 **features** (`[[feature]]`) of 4–6 files that the
planner splits, the coder writes piece by piece and the compiler checks together as a fresh `dragonfly new` project:

1. `reference` — Claude Sonnet answers every task once, with the same guide and the dialect's sheet, and one repair
   with the compiler's errors (USD 0.89 for the 45). A feature is one answer with every file.
2. `distill` — the Sonnet answers of the `train` tasks that pass become examples in the graph's `examples`
   collection (a feature as one example per file). The 18 `test` tasks are held out: their answers never reach the
   graph.
3. `local` — the local coder answers every task through the Developer; an answer passes when `nuclear check` reports
   no errors and it has every acceptance text. The local developer is **equivalent** to Sonnet on a task when both pass.
4. A failure is studied and fixed with a **general rule or more knowledge** in the graph (never the answer), then a
   new round.

| Round | What changed | Local / Sonnet (all 32) | Local / Sonnet (12 held out) |
|---|---|---|---|
| 1 | the ingested knowledge only (Sonnet's references were still running: its numbers here are partial) | 23 / — | 9 / — |
| 2 | + 15 distilled Sonnet examples | 26 / 25 | 8 / 10 |
| 3 | + 7 rules in the coder's guide (decorators above their field, `gte(...)` in queries, graph nodes, `replaceAll`...); two acceptance texts that were wrong for both models fixed | **28 / 26** | **10 / 11** |

From round 4 the bank has 45 tasks (18 held out):

| Round | What changed | Local / Sonnet (all 45) | Local / Sonnet (18 held out) |
|---|---|---|---|
| 4 | + 10 mobile and web tasks, 3 features; the web platform's brief (Sonnet 0/4 → 4/4 on web); `dragonfly new`'s mobile skeleton fixed; N0116 through runtime classes, N0013 for a dangling decorator | 38 / 39 | 14 / 17 |
| 5–7 | the planner: paths from the platform's `[layout]`, kinds and forgotten pieces from the spec's words, the spec's sentence in each task; the coder's rule "only what the task declares" | 38 / 39 | 14 / 17 |
| 8–9 | a feature distilled as one example per file (a whole-feature example made the coder write every file into each piece) | 37 / 39 | 14 / 17 |
| 10–11 | the agents write only Nuclear (`platform`, the graph's `nuclear` collection, the platform sheets); each piece's task is the spec's own sentence; few-shots (aliases, columns, `replaceAll`, state models, a test file); the entity by folder accepted | 37 / 39, first feature passed | 13 / 17 |
| 12 | void: an `ingest` run from the wrong folder had emptied the graph (fixed: an ingest removes only its own roots' sources) | (29 / 39) | (8 / 17) |
| 13 | every prompt moved to a file (`prompts/agents/*.md`, the same text) on the re-ingested graph | **38 / 39** | **15 / 17** |
| 14 | void: `/tmp` filled up during the run | — | — |
| 15 | the tester: only the plan's names, nothing to inject for models, a search without `inject`; errors by line | **38 / 39**, 1 of 3 features | **15 / 17** |
| 16 | void: stopped at 10 minutes while another agent held Ollama | — | — |
| 17 | the words of the kinds in Nuclear only (`[underneath]` for the framework), the compiler gaps fixed | 37 / 39, 1 of 3 features | 14 / 17 (`entity-reader`: the decorator written after its field, not repaired) |

On single files the local 3B coder is at Sonnet's level (36 of 42 against 36). On features it is behind: after round
11 it passes one of the three by itself (planned, written piece by piece and compiled as a project), where Sonnet
passes all three; the others miss an application component the planner does not recognise and a repository detail.
Measured and discarded: giving the coder the file `dragonfly make` writes (29 of 42 against 37). Each local answer takes 1.5–10 s on the CPU, a feature 1–1.5 min. The full table,
task by task, is `eval/results/REPORT.md`.

```sh
cd eval
../../nuclear-graph/.venv/bin/python run.py reference   # paid: Claude, cached in results/reference.json
../../nuclear-graph/.venv/bin/python run.py distill
../../nuclear-graph/.venv/bin/python run.py local        # the Developer must be running
../../nuclear-graph/.venv/bin/python run.py report
```

## Known limits

- The 3B tester copies a solved example of the graph when it is close to the task (a spec that declares only a model
  got a test of an application component from the examples): the run now fails at *Tests first* with the diagnosis
  instead of passing without tests; Claude-led runs fix it.
- The planner maps a kind only from its own word ("application component" is not `appcomponent`) and names a
  component after the first model of a plain spec (`newbooks`).
- The words of the kinds (`docs.toml`) name the framework in prose ("pydantic models"): the `nuclear` collection
  removes code, not prose.
- The context window is fixed at 8192 in `config/project_config.nuc` (reading it from the environment needs the
  compiler to know the return types of the type definitions' functions: Pending in STATUS).

## Development

```sh
nuclear check . && nuclear build .
cd build && ../../tools/python/.venv/bin/pyright --pythonpath .venv/bin/python app   # 0 errors (strict)
```

Never edit `build/`: change the `.nuc` files.

## License

Apache-2.0.
