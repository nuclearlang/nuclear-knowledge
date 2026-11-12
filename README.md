# nuclear-knowledge

The whole knowledge base of [NuclearLang](https://github.com/nuclearlang/nuclear) and Dragonfly in one folder, step by
step: what the language is, its rules and library, the compiler, Dragonfly and its three platforms, the command line,
the editors, the AI loop (the Smart Graph and its MCP server, the Nuclear Developer, the Loop), every decision and the
state of the project. It is what the [Nuclear Developer](https://github.com/nuclearlang/nuclear-developer)'s local
agents know: the [Smart Graph](https://github.com/nuclearlang/nuclear-graph) ingests this folder.

**Start at [INDEX.md](INDEX.md)**: the sections in reading order, each file with the repository and path it comes from.

## How it is made

Every file is a byte-for-byte copy of its source in the working tree (or the website, or the help the tools print):
nothing is summarised or rewritten, not a comma. `sections.toml` says the order and which files each section holds;
a file no section claims goes to `99-other`, so none is ever left out.

```sh
nuclear-graph knowledge build        # copies every source into its section, writes INDEX.md and manifest.json
nuclear-graph knowledge check        # fails when a copy differs from its source, one is missing or one is extra
nuclear-graph ingest nuclear-developer/data   # the Smart Graph reads it (the whole knowledge: what it lacks is removed)
```

Never edit a copy: change its source, then build again. A section with `agents = false` (the target languages, the
dialects underneath, the runtimes, the ADRs) is for people and frontier models: the local agents write only Nuclear,
so they do not read it.

| | |
|---|---|
| `NN-section/<path>` | the copies, by section |
| `INDEX.md` | the reading order |
| `manifest.json` | every file: its source, repository, section, size and SHA-256 |
| `sections.toml` | the sections, in order (edited by hand) |

## License

Apache-2.0, like the sources.
