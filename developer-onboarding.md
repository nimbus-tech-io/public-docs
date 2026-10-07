---
layout: doc
title: Developer Onboarding
subtitle: What to install and which accounts to ask for in your first week.
updated: 2026-10-07
author: Milos
tags: [onboarding]
---

## Accounts

### GitHub

1. Create a GitHub account at [github.com/signup](https://github.com/signup). Name it `firstname-lastname-nimbus-tech`, for example `ana-petrovic-nimbus-tech`.
2. Send the username to your manager.
3. Your manager adds you to the Nimbus Tech GitHub organization. Accept the invite in your email.

### Invites from your manager

Ask your manager for invites to:

- Slack
- 1Password
- AWS
- The back-office

### Back-office

The back-office is at [back-office.nimbus-tech.io](https://back-office.nimbus-tech.io). Use it to manage your vacations, client hours and lunch orders. An admin creates your account, so ask your manager if you can't sign in.

---

## Install Homebrew

[Homebrew](https://brew.sh) installs the other tools on this page. Install it first. Copy the install command from [brew.sh](https://brew.sh) and run it in the Terminal app. When it finishes, follow the "Next steps" it prints to add `brew` to your shell. Then check it with `brew --version`.

---

## Recommended tools

Everyone on the team uses these. This command installs all of them:

```bash
brew install --cask slack 1password drata-agent && brew install awscli
```

| Tool | What it is for | Download |
|---|---|---|
| Slack | Team chat | [slack.com/downloads](https://slack.com/downloads/mac) |
| 1Password | Passwords and shared secrets | [1password.com/downloads](https://1password.com/downloads/mac) |
| Drata Agent | Checks that your laptop meets our security rules | [drata.com](https://drata.com) |
| AWS CLI | Work with AWS from the terminal | [AWS CLI install guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) |

> [!NOTE]
> Claude Code
>
> Claude Code has its own guide. Follow [Getting Started with Claude Code](getting-started-with-claude.html).

---

## Pick your own

For these, use what you like. Each option has a Homebrew command and a download link.

### Browser

| Option | Command | Download |
|---|---|---|
| Chrome | `brew install --cask google-chrome` | [google.com/chrome](https://www.google.com/chrome/) |
| Brave | `brew install --cask brave-browser` | [brave.com](https://brave.com/download/) |

> [!WARNING]
> Brave Shields
>
> Brave's built-in blockers sometimes block developer tools and extensions. If a page or extension acts strangely, turn Shields off for that site.

### Editor

| Option | Command | Download |
|---|---|---|
| VS Code | `brew install --cask visual-studio-code` | [code.visualstudio.com](https://code.visualstudio.com/download) |
| Zed | `brew install --cask zed` | [zed.dev](https://zed.dev/download) |
| Neovim | `brew install neovim` | [neovim.io](https://neovim.io) |

If you pick VS Code, install the extensions in [VS Code Extensions](vs-code-extensions.html).

### Terminal

| Option | Command | Download |
|---|---|---|
| Warp | `brew install --cask warp` | [warp.dev](https://app.warp.dev/referral/8G46D6) |
| Ghostty | `brew install --cask ghostty` | [ghostty.org](https://ghostty.org/download) |

### Containers

| Option | Command | Download |
|---|---|---|
| OrbStack | `brew install --cask orbstack` | [orbstack.dev](https://orbstack.dev/download) |
| Docker Desktop | `brew install --cask docker-desktop` | [docker.com](https://www.docker.com/products/docker-desktop/) |
