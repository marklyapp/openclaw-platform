# OpenClaw Platform Setup

One-time setup for running [OpenClaw](https://openclaw.ai) workflows on your machine. Complete this before setting up any specific workflow.

## What This Is

OpenClaw is an AI agent orchestration platform. It runs a **gateway** that connects AI agents to chat channels (Discord, Mattermost) and tools (Docker sandboxes, GitHub CLI). You define **workflows** — collections of agents with specific roles — and the gateway coordinates them.

This repo covers the **platform layer**: the shared infrastructure that all workflows need. Individual workflows (like a software dev pod or a support bot) live in their own repos with their own agent configs.

```
Platform (this repo — do once)          Workflows (separate repos — one per workflow)
  Install OpenClaw CLI                     ~/.openclaw-devpod/    (port 18789)
  Build Docker sandbox                     ~/.openclaw-support/   (port 18790)
  Create Discord bot                       ~/.openclaw-research/  (port 18791)
  Set up Mattermost                        ...
  Auth GitHub CLI
```

## Prerequisites

| Requirement | Why |
|-------------|-----|
| **Windows 10/11** with [Git Bash](https://git-scm.com/downloads) | Shell environment (ships with Git for Windows) |
| **Docker Desktop** | Agents run in sandboxed containers |
| **Node.js 18+** | OpenClaw CLI runtime |

## Step 1: Install OpenClaw

```bash
npm install -g openclaw
openclaw --version
```

## Step 2: Build the Docker Sandbox Image

Clone this repo and build the shared sandbox image:

```bash
git clone git@github.com:marklyapp/openclaw-platform.git ~/openclaw-platform
cd ~/openclaw-platform
docker build -f Dockerfile.sandbox -t openclaw-sandbox:bookworm-slim .
```

This image (Debian bookworm-slim + Python 3 + Node.js 20 + GitHub CLI) is used by all workflows. You only build it once.

## Step 3: Set Up Chat Channels

Set up one or both — each workflow can use either or both simultaneously.

| Channel | Guide | When to use |
|---------|-------|-------------|
| **Discord** | [docs/discord-setup.md](docs/discord-setup.md) | Quick setup, hosted, free |
| **Mattermost** | [docs/mattermost-setup.md](docs/mattermost-setup.md) | Self-hosted, full control |

Save the bot tokens and channel IDs — you'll need them when configuring each workflow.

## Step 4: Authenticate GitHub CLI

Agents need GitHub access inside their Docker sandboxes. Follow [docs/github-cli.md](docs/github-cli.md) to auth once — the credentials are bind-mounted into every workflow's containers.

```bash
# Quick version:
mkdir -p gh-config
docker run --rm -it \
  -v "$(pwd)/gh-config:/root/.config/gh" \
  openclaw-sandbox:bookworm-slim \
  gh auth login
```

## Step 5: Get an Anthropic API Key

Most workflows use Claude as the AI model.

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an API key (starts with `sk-ant-...`)
3. Save it — you'll enter it in each workflow's config

## Step 6 (Optional): Set Up a Local LLM

If you have an NVIDIA GPU (16+ GB VRAM), you can run a local model for free inference on simple tasks. See [docs/local-llm.md](docs/local-llm.md).

## Next Steps

Platform setup is done. Now set up a workflow:

| Workflow | Repo | Description |
|----------|------|-------------|
| **Software Dev Pod** | [openclaw-config](https://github.com/marklyapp/openclaw-config-devteam) | Autonomous dev team: architect, dev, reviewer, coordinator |

Each workflow repo has its own `setup.sh` that detects paths, creates config files, and tells you what to fill in. The tokens and IDs from the platform setup above are what you'll enter.

## File Reference

| File | Purpose |
|------|---------|
| `Dockerfile.sandbox` | Shared Docker image for all workflows |
| `gh-config/` | GitHub CLI credentials (gitignored) |
| `docs/discord-setup.md` | Discord bot creation guide |
| `docs/mattermost-setup.md` | Mattermost setup guide |
| `docs/github-cli.md` | GitHub CLI auth for Docker sandboxes |
| `docs/local-llm.md` | Local GPU model setup (optional) |
