# `~/.agentctl`

This directory is my global, git-managed source of truth for `agentctl`.

Project repository: https://github.com/orgforge-ai/agentcli

It holds the reusable definitions that should follow me across repositories:

- global agents
- global config
- global model mappings

`agentctl` reads from here, merges these global definitions with any project-local `.agentctl/` directory inside a repo, and then renders harness-specific files for tools like Claude Code and OpenCode.

## Layout

```text
~/.agentctl/
├── README.md
├── config.json
├── models.json
└── agents/
    └── ducky/
        ├── agent.json
        ├── description.md
        └── prompt.md
```

## What Each Path Does

- `config.json`: global `agentctl` configuration
- `models.json`: global model-class mappings such as `small`, `medium`, `large`, `planning`, `editing`, and `reasoning`
- `agents/`: canonical global agent definitions

Each agent lives in its own directory under `agents/`:

```text
~/.agentctl/agents/<name>/
├── agent.json
├── description.md
└── prompt.md
```

Agent files:

- `agent.json`: required manifest and metadata
- `description.md`: optional human-facing description
- `prompt.md`: optional prompt content rendered into harness-native agent files

Current global agent here:

- `ducky`: a skeptical review agent used to pressure-test architecture and implementation decisions

## How `agentctl` Resolves Things

`agentctl` uses two layers:

- Global: `~/.agentctl/`
- Project: `<repo>/.agentctl/`

For agents, project-local definitions override global ones with the same name.

That means:

- put an agent in `~/.agentctl/agents/` if you want it available everywhere
- put an agent in `<repo>/.agentctl/agents/` if it should belong only to that repo

## Core Workflow

Typical flow:

1. Edit a global agent in `~/.agentctl/agents/`
2. Run `agentctl sync`
3. Run `agentctl doctor`
4. Use the agent through `agentctl run`
5. Commit the source changes in this git repo

## Useful Commands

List canonical agents:

```bash
agentctl list agents
agentctl list agents --global
```

Sync canonical agents into harness-native files:

```bash
agentctl sync
agentctl sync claude
agentctl sync opencode
```

Preview sync without writing:

```bash
agentctl sync --dry-run
```

Run a harness with one of these global agents:

```bash
agentctl run -h claude --agent ducky
agentctl run -h opencode --agent ducky
```

Run a headless prompt:

```bash
agentctl run -h claude --agent ducky --headless --prompt "Review this design"
```

Inspect installed harness agents:

```bash
agentctl harness list claude agents
agentctl harness list opencode agents
```

Check config, harness detection, unmanaged files, and sync drift:

```bash
agentctl doctor
```

## Where Rendered Files Go

The files under `~/.agentctl/` are the canonical source. Harnesses read rendered output, not these source files directly.

Current global render targets:

- Claude Code: `~/.claude/agents/*.md`
- OpenCode: `~/.config/opencode/agents/*.md`

Project-local `.agentctl/` directories render into repo-local harness directories:

- Claude Code: `<repo>/.claude/agents/*.md`
- OpenCode: `<repo>/.opencode/agents/*.md`

## Adding A New Global Agent

1. Create `~/.agentctl/agents/<name>/`
2. Add `agent.json`
3. Add `prompt.md`
4. Optionally add `description.md`
5. Run `agentctl sync`
6. Commit the change

Minimal example:

```json
{
  "version": 1,
  "name": "reviewer",
  "description": "General code review agent",
  "defaultModelClass": "large"
}
```

## Git Workflow

If this directory is being managed as its own repo, the usual loop is:

```bash
cd ~/.agentctl
git status
git add .
git commit -m "Update global agent prompts"
```

After any agent change:

```bash
agentctl sync
agentctl doctor
```

That keeps the canonical source, rendered harness output, and installed state aligned.
