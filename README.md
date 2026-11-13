# nuclear-knowledge

The knowledge base of [NuclearLang](https://github.com/nuclearlang/nuclear) and Dragonfly: what the
[Smart Graph](https://github.com/nuclearlang/nuclear-graph) ingests, and so what the
[Nuclear Developer](https://github.com/nuclearlang/nuclear-developer)'s local agents and every MCP client know. It is
not a copy of the documentation: it is rows, each one a card that answers one question, on the shelf of its kind of
question, linked to the cards it has to do with. Decision record:
[ADR-0104](https://github.com/nuclearlang/nuclear/blob/main/docs/adr/0104-smart-graph-shelves.md).

**Start at [INDEX.md](INDEX.md)**: the shelves, what each one answers, and its rows.

## What is here

| | |
|---|---|
| `assets/<shelf>/<name>.json` | the rows: `{shelf, origin, about, rows}`; a row's body is a list of lines |
| `assets/<shelf>/<name>.overlay.json` | hand-written fields merged into the generated rows of `<name>.json` |
| `schema.json` | the shelves (the question each answers, its row types, id prefixes and filters) and the fields of a row |
| `sections.toml` | which markdown files of the working tree the base holds, on which shelf, and who reads them (edited by hand) |
| `questions.toml` | the questions people and agents ask, and the row each must return; and the requests people make (`request = true`), with the strategy each must get: the base's acceptance test (edited by hand) |
| `manifest.json` | the schema, the revision, and each row's shelf, hash and the revision that last changed it; for a hand-written row, the hash of each of its sources at its last review |
| `revisions.json` | the log: what each revision added, changed and removed |
| `INDEX.md` | the shelves and their rows, for people |

## The shelves

`concepts` (what, why, how) · `language` (how it is written) · `platforms` (kinds, decorators, layout, plugins) ·
`tools` (every command, MCP tool, route, generator, stage, agent, version) · `errors` (every diagnostic, habit,
failure and pitfall, linked) · `capabilities` (what can and cannot be done; the agents' limits) · `playbooks`
(the strategy of each kind of request, procedures, roles, skills) · `examples` (validated Nuclear) · `docs` (the long documents, section by section).

## Two kinds of row

- **Generated** (`"origin": "generated"`): read from a structured source of the working tree by
  `nuclear-graph knowledge build`. Never edit one: change its source and build again.
- **Curated** (`"origin": "curated"`): written by hand, in `assets/concepts`, `assets/capabilities`,
  `assets/playbooks`, `assets/errors/failures.json` and `assets/errors/pitfalls.json`. Each names the `sources` it was
  written from; when one changes, the check asks for the row to be read again.

A row:

```json
{
 "id": "failure:the-graph-is-not-running",
 "type": "failure",
 "title": "Failure: the Nuclear Smart Graph is not reachable",
 "summary": "The answer in a sentence or two: what is embedded and shown in a list.",
 "body": ["The whole card, in markdown, one line per item.", "Nuclear only, unless `agents` is false."],
 "keys": ["graph is not running", "7878"],
 "asks": ["why can't the agent reach the graph"],
 "platforms": ["backend"],
 "agents": true,
 "links": [{"rel": "tool", "to": "tool:nuclear-graph.serve"}],
 "sources": ["nuclear-graph/README.md#Install"],
 "meta": {"family": "transient", "severity": "error"}
}
```

`keys` are the exact names a query may write (lower case); `asks` are questions the row answers, as people write
them (each is searched as a question); `links` point to rows that must exist; `platforms` left out means every
platform; `agents: false` keeps a row from the local agents (they read only Nuclear).

## Keeping it current

```sh
nuclear-graph knowledge build        # write the generated assets, validate everything, raise the revision
nuclear-graph knowledge check        # fail when the base is behind the tree, a row is invalid or a source changed
nuclear-graph knowledge review       # confirm the hand-written rows whose sources changed and still hold
nuclear-graph ingest                 # the graph learns the revision (only what changed is embedded again)
nuclear-graph knowledge ask          # every question of questions.toml finds its row
```

After a change of the ecosystem: build; for each `review:` line, read the change and update the row or confirm it;
write a row for whatever is new and has no structured source (a capability, a limit, a failure, a procedure); add
the question the change answers to `questions.toml`; check; ingest. The working tree's end-of-turn hook runs the
check. The same procedure is a row of the base: `playbook:update-the-knowledge-base`.

A base is valid or it is not built: a link to a missing row, a duplicated id, a type or a filter that is not of the
shelf, a diagnostic without documentation, a document no section claims, or target-language code in a row the local
agents read stops the build.

## License

Apache-2.0, like the sources.
