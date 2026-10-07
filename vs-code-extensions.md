---
layout: doc
title: VS Code Extensions
subtitle: The extensions the team uses for web development, Git, testing and AI.
updated: 2026-04-07
author: Milos
tags: [vs-code]
---

## Mandatory for web dev

| Extension                                                                                     | Why                                                                                                |
| --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| [Tailwind CSS](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss) | Shows the original CSS when you hover over a utility class. Also adds autocompletion for Tailwind. |
| [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)          | Without it, you won't see ESLint errors in your editor.                                            |
| [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)        | Formats everyone's code the same way, so we don't get conflicts in the repo.                       |

> [!WARNING]
> Turn on Format on Save
>
> Prettier only helps if it runs. Press `Cmd + ,`, search for "Format on Save" and turn it on.

---

## Nice to have

| Extension                                                                                                     | Why                                                                                                                  |
| ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)                                | A bunch of useful Git tools across the editor: a stash UI, history lookup and more. You don't need the paid version. |
| [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)                           | `git status`, but as a UI. Also nicer looking.                                                                       |
| [Mark#](https://marketplace.visualstudio.com/items?itemName=jonathan-yeung.mark-sharp)                        | A great Markdown editor.                                                                                             |
| [GitHub Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) | Review GitHub PRs from VS Code. The diffs are also much better.                                                      |
| [SemanticDiff](https://semanticdiff.com/vscode/)                                                              | Makes diffs easier to scan. Great for Tailwind code because it shows only the real changes.                          |

---

## For testing

| Extension                                                                     | Why                                                                                                                          |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [Vitest](https://marketplace.visualstudio.com/items?itemName=vitest.explorer) | Adds a "Run Test" button next to Vitest tests and shows the results in a separate panel. Useful if your project uses Vitest. |

---

## AI

| Extension                                                                                | Why                                                                                                        |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| [Claude Code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) | Brings Claude Code into VS Code. See [Getting Started with Claude Code](getting-started-with-claude.html). |
