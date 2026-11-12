# Changelog

## Unreleased (2026-11-11)

- Agents: coder, tester, security reviewer, planner (2026-11-10); documenter (`/developer/document`), assistant
  (`/developer/chat`, the editor's chat), diagnoser (`/developer/diagnose`: kind by rules, evidence by the tools, the
  next step) (2026-11-11).
- Every prompt a model reads is a file of the graph (`prompts/agents/*.md`, `Workbench.prompt`); the agents hold no
  prompt text.
- The agents write only Nuclear: `platform` instead of `dialect`, the graph's `nuclear` collection, a piece's errors by
  line (no scratch-file path), the diagnoser reads failures told in Nuclear.
- The planner builds paths from the platform's `[layout]`, takes kinds and forgotten pieces from the spec's words and
  appends the spec's own sentence to each piece.
- The tester gets only the plan's names (also when it repairs a test file), is told when there is nothing to inject,
  and its search no longer leans on `inject`.
- The assistant reports the size of the prompt it read (`context` in its trace) for the editor's context meter.
- Evaluation: 45 tasks (18 held out), rounds 1–15; rounds 13 and 15: 38 / 39 against Sonnet, 15 / 17 held out, one
  of the three features passed by the local agents alone.
