# NuclearLang

A minimal language, made so AI agents and people write only the logic of a program, transpiled to TypeScript, Dart
and Python through declarative dialects. **Dragonfly** is its framework: one Nuclear codebase for three platforms.

| Repository | What it is |
|---|---|
| [nuclear](https://github.com/nuclearlang/nuclear) | the language: compiler, `nuclear` CLI, language server, the language dialects, specs |
| [dragonfly](https://github.com/nuclearlang/dragonfly) | Dragonfly: the `dragonfly` CLI and the platforms' philosophy |
| [dragonfly-mobile](https://github.com/nuclearlang/dragonfly-mobile) | mobile apps: dialect `flutter` |
| [dragonfly-backend](https://github.com/nuclearlang/dragonfly-backend) | backends: dialects `fastapi` (Python) and `nestjs` (TypeScript) |
| [dragonfly-web](https://github.com/nuclearlang/dragonfly-web) | web apps: dialect `react` |
| [nuclear-vscode](https://github.com/nuclearlang/nuclear-vscode) · [nuclear-jetbrains](https://github.com/nuclearlang/nuclear-jetbrains) · [nuclear-zed](https://github.com/nuclearlang/nuclear-zed) | editor extensions |
| [nuclear-graph](https://github.com/nuclearlang/nuclear-graph) | the Nuclear Smart Graph: the private knowledge base (RAG) and the MCP server |
| [nuclear-developer](https://github.com/nuclearlang/nuclear-developer) | the local 3B agents (coder, tester, security, planner), written in Nuclear |
| [nuclear-knowledge](https://github.com/nuclearlang/nuclear-knowledge) | the whole knowledge base in one folder, what the agents read |
| [website](https://github.com/nuclearlang/website) | the documentation site, written in Nuclear |

```sh
curl -fsSL https://nuclearlang.github.io/install.sh | sh     # Linux, macOS
irm https://nuclearlang.github.io/install.ps1 | iex          # Windows
```

Licensed under Apache-2.0.
