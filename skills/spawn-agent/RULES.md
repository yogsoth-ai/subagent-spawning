# Subagent SOP Authoring Rules

This document defines how to write an SOP that spawns a subagent. Consult this when creating a new SOP with `execution: subagent`.

## SOP Frontmatter Format

```yaml
---
name: [Descriptive Name]
description: [One-line description of what this SOP does]
type: sop
layer: sop
execution: subagent
prompt: ./prompt.md
input: [field1] ([type]), [field2] ([type])
output: markdown ([brief description of output structure])
---
```

### Required Fields

| Field | Value | Purpose |
|-------|-------|---------|
| execution | `subagent` | Signals spawn-agent skill activation |
| prompt | `./prompt.md` | Relative path to role prompt file |
| input | field list with types | Documents what parameters the SOP receives |
| output | `markdown (...)` | Documents expected output structure |

### Optional Override Fields

| Field | Values | Default | When to use |
|-------|--------|---------|-------------|
| model | haiku, sonnet, opus | opus | Almost never |
| tools | array of MCP names | all | Almost never |

## prompt.md Structure

```markdown
# Role

You are [role]. Your task is [what you do].

## Framework

[Step-by-step methodology — what the subagent should do]

## Input

You receive:
- [FIELD_NAME]: [what it contains and how to use it]

## Output

Return your analysis as markdown with:
- [section/heading 1]: [what goes here]
- [section/heading 2]: [what goes here]

## Constraints

- [Behavioral rules]
- [What NOT to do]
```

### Key Principles

1. **Self-contained** — subagent gets ONLY this prompt + user message. No prior conversation context.
2. **Tool-aware** — mention when the subagent should search, verify, or look up evidence. It has full MCP access.
3. **Output in natural language** — define structure via headings/sections, not JSON schema.
4. **Concise** — keep under 500 words. The subagent is Opus; it doesn't need verbose instructions.

## Complete Example

### SOP SKILL.md

```yaml
---
name: SCAMPER Substitute
description: Apply Substitute lens — replace components with alternatives to generate idea variants
type: sop
layer: sop
execution: subagent
prompt: ./prompt.md
input: idea (string), context (string)
output: markdown (variants with title, description, novelty assessment)
---
```

```markdown
# SCAMPER Substitute SOP

## Layer Rules
- **Layer**: sop
- **Called by**: tactic/idea-generation, tactic/scamper
- **Calls**: spawn-agent skill (creates subagent)

## Purpose
Generate idea variants by systematically substituting components.

## Workflow
1. Main CC loads this SKILL.md
2. Reads ./prompt.md
3. Loads spawn-agent skill
4. Spawns subagent with prompt.md + input parameters
5. Returns subagent's markdown output to calling tactic
```

### prompt.md

```markdown
# Role

You are a creative research ideation agent specializing in the SCAMPER Substitute method.

## Framework

1. Decompose the idea into key components (method, data, evaluation, assumption)
2. For each component, identify alternatives that could replace it
3. Assess what changes with each substitution
4. Search for evidence: are the proposed substitutions used elsewhere?
5. Generate 2-4 concrete variants with different substitution targets
6. Rate each variant's novelty potential (low/medium/high)

## Input

You receive:
- IDEA: A research idea description (may include title, method, evaluation details)
- CONTEXT: Research background, accumulated knowledge, and constraints

## Output

Return as markdown:

### Substitution Analysis
Brief overview of which components were examined and why.

### Variants
For each variant (2-4):
#### [Variant Title]
- **What's substituted**: [component] → [replacement]
- **Why interesting**: [what this unlocks or enables]
- **Evidence**: [any supporting literature or precedent found]
- **Novelty**: low | medium | high

### Summary
Which substitutions are most promising and why.

## Constraints

- Each substitution must have a rationale — no random alternatives.
- Search for evidence when possible.
- Focus on substitutions that change the idea's character, not trivial parameter swaps.
```
