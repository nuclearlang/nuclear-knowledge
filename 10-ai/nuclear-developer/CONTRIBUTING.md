# Contributing to nuclear-developer

Thank you for helping. This repository is part of [NuclearLang](https://github.com/nuclearlang): the language is the
[nuclear](https://github.com/nuclearlang/nuclear) repository, and the other repositories live as folders inside a clone of it, each with its
own git (the tools find each other that way).

## Set up

```sh
git clone https://github.com/nuclearlang/nuclear && cd nuclear
git clone https://github.com/nuclearlang/nuclear-graph
git clone https://github.com/nuclearlang/nuclear-developer
git clone https://github.com/nuclearlang/nuclear-knowledge nuclear-developer/data
./install.sh --ai
```

## Before a pull request

```sh
(cd nuclear-developer && dragonfly test)   # needs the graph running: nuclear-graph serve
```

- Everything is written in English: code, comments, docs and messages.
- A change that the user of NuclearLang can see updates its documentation in the same pull request (README, CHANGELOG,
  `docs/context/` for agents and contributors).
- Rules and the reasons behind them: the [nuclear repository's AGENTS.md](https://github.com/nuclearlang/nuclear/blob/main/AGENTS.md) and its
  [decision records](https://github.com/nuclearlang/nuclear/tree/main/docs/adr) (ADR-0102, ADR-0103).

By contributing you agree that your contribution is licensed under the [Apache License 2.0](LICENSE).
