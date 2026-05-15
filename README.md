# subagent-spawning

**Claude Code skill for spawning customized subagents with full MCP tool access — replaces dare-agents MCP with pure markdown orchestration.**

- 🤖 **Role-specific subagents** — spawn specialized thinking agents from SOP definitions
- 🔧 **Full tool inheritance** — each subagent gets all MCP tools (literature search, web browsing, citation tracing)
- 📝 **Pure markdown** — zero code, zero dependencies, zero runtime processes
- 🎯 **Opus-level reasoning** — subagents operate at the same quality as the main orchestrator

## What is this?

This is a [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code) that standardizes how subagents are spawned during research workflows. When an SOP declares `execution: subagent`, this skill tells main CC exactly how to create that subagent — what model to use, what tools to give it, how to pass inputs, and how to handle outputs.

Designed for the NOESYNTH/DARE research engine ecosystem. Works alongside [literature-engine](https://github.com/yogsoth-ai/literature-engine), [web-browsing](https://github.com/yogsoth-ai/web-browsing), and [semantic-scholar-mcp](https://github.com/yogsoth-ai/semantic-scholar-mcp).

## How It Works

```
SOP SKILL.md declares: execution: subagent, prompt: ./prompt.md
  → Main CC reads prompt.md (role definition)
  → Main CC loads spawn-agent skill (execution rules)
  → Main CC invokes Agent tool:
      - prompt = [prompt.md] + [input parameters]
      - model = opus (default)
      - tools = all (inherited)
  → Subagent executes with full MCP access
  → Returns markdown result
```

## Default Strategy

| Dimension | Default | Rationale |
|-----------|---------|-----------|
| Model | opus | Quality-first; same reasoning power as main CC |
| Tools | all | Subagent autonomously decides what to use |
| Output | markdown | Natural, structured, no parsing needed |

## Quick Start

### 1. Install Skill

Clone this repository:

```bash
git clone https://github.com/yogsoth-ai/subagent-spawning.git
```

### 2. Write an SOP with Subagent

In your SOP's SKILL.md frontmatter:

```yaml
---
execution: subagent
prompt: ./prompt.md
input: idea (string), context (string)
output: markdown (analysis with variants)
---
```

Create a `prompt.md` next to it defining the subagent's role.

### 3. Consult RULES.md

See `skills/spawn-agent/RULES.md` for the full authoring guide with examples.

## SOP Declaration Format

```yaml
---
name: SCAMPER Substitute
description: Apply Substitute lens to generate idea variants
type: sop
layer: sop
execution: subagent
prompt: ./prompt.md
input: idea (string), context (string)
output: markdown (variants with title, description, novelty assessment)
---
```

| Field | Purpose |
|-------|---------|
| `execution: subagent` | Signals spawn-agent skill activation |
| `prompt: ./prompt.md` | Relative path to role prompt file |
| `input` | Documents what parameters the SOP receives |
| `output` | Documents expected output structure |

## Project Structure

```
subagent-spawning/
├── skills/
│   └── spawn-agent/
│       ├── SKILL.md      # Runtime — loaded by CC when executing subagent SOPs
│       └── RULES.md      # Authoring reference — how to write subagent SOPs
├── assets/
│   └── repo-info.txt
├── README.md
├── .gitignore
└── LICENSE
```

## What This Replaces

| Before (dare-agents MCP) | After (subagent-spawning) |
|---------------------------|---------------------------|
| 34 TypeScript tool files | 0 code files |
| 34 zod schemas | 0 schemas |
| pi-ai + OpenRouter dependency | None |
| MCP server process | None |
| Information-isolated execution | Full MCP tool access |
| ~2000 lines TypeScript | ~200 lines markdown |

## Links

- 🐙 [GitHub repository](https://github.com/yogsoth-ai/subagent-spawning)
- 📚 [literature-engine](https://github.com/yogsoth-ai/literature-engine)
- 🌐 [web-browsing](https://github.com/yogsoth-ai/web-browsing)
- 🔬 [semantic-scholar-mcp](https://github.com/yogsoth-ai/semantic-scholar-mcp)

## Version

**v0.1.0** — Initial release (spawn-agent skill + authoring rules)

## License

[Apache-2.0](LICENSE)
