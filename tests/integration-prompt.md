# Integration Test: subagent-spawning

Use this prompt in a Claude Code session within this repo to test the spawn-agent skill against live MCP servers.

## Prerequisites

- alphaxiv MCP configured (SSE: https://mcp.alphaxiv.org)
- semantic-scholar MCP configured (@yogsoth-ai/semantic-scholar-mcp)
- brave-search MCP configured

## Test Topic

**"SCAMPER Substitute on knowledge distillation for LLM compression"**

This topic is ideal because:
- Well-covered in literature (many distillation variants exist)
- Has clear components to substitute (loss function, teacher-student architecture, evaluation)
- Tests whether subagent actually searches for evidence vs. hallucinating
- Verifiable output structure (markdown with specific sections)

---

## Test 1: Basic Execution Protocol

**Goal:** Verify CC correctly reads SKILL.md and follows the 4-step execution protocol.

**Prompt:**

```
Read skills/spawn-agent/SKILL.md. Then execute the following SOP:

SOP declaration:
- execution: subagent
- prompt: (inline below)
- input: idea (string), context (string)

prompt.md content:

# Role

You are a creative research ideation agent specializing in the SCAMPER Substitute method.

## Framework

1. Decompose the idea into key components (method, data, evaluation, assumption)
2. For each component, identify 1-2 alternatives that could replace it
3. Search for evidence: are the proposed substitutions used elsewhere in literature?
4. Generate 2-3 concrete variants
5. Rate each variant's novelty (low/medium/high)

## Input

You receive:
- IDEA: A research idea description
- CONTEXT: Research background and constraints

## Output

Return as markdown with sections: Substitution Analysis, Variants (each with title, what's substituted, why interesting, evidence, novelty), Summary.

## Constraints

- Each substitution must have a rationale.
- Search for supporting literature when possible.
- Focus on non-trivial substitutions.

---

Input parameters:

IDEA:
Using knowledge distillation to compress large language models into smaller student models for edge deployment, evaluated on MMLU and HellaSwag benchmarks.

CONTEXT:
Research on efficient LLM inference. Key challenge: maintaining reasoning capability after compression. Current SOTA uses layer-wise distillation with attention transfer.
```

**Expected behavior:**
- CC reads SKILL.md and identifies the execution protocol
- CC formats input as UPPER_CASE field names separated by blank lines
- CC invokes Agent tool with: prompt.md content + `---` separator + formatted input
- Subagent executes and returns markdown output
- Subagent uses MCP tools (alphaxiv, SS, brave-search) to find evidence

**Failure conditions:**
- CC does not invoke Agent tool (tries to answer directly)
- CC does not follow the input formatting convention (UPPER_CASE fields)
- CC uses wrong model (should be opus by default)
- Subagent returns unstructured text without the expected sections
- Subagent does not search for any evidence (no tool calls)

---

## Test 2: Tool Usage by Subagent

**Goal:** Verify the subagent actually uses MCP tools to find evidence rather than hallucinating.

**Verification checklist:**
- [ ] Subagent called at least one search tool (alphaxiv discover_papers, ss relevanceSearch, or brave_web_search)
- [ ] Evidence section in variants references real papers or methods
- [ ] Substitutions are grounded in actual literature, not generic suggestions

**Failure conditions:**
- Subagent produces output without any tool calls
- Evidence sections contain fabricated paper titles
- All variants are generic (e.g., "use a different loss function" without specifying which)

---

## Test 3: Output Structure

**Goal:** Verify output matches the structure defined in prompt.md.

**Expected output structure:**
```markdown
### Substitution Analysis
[Brief overview]

### Variants
#### [Variant 1 Title]
- **What's substituted**: [component] → [replacement]
- **Why interesting**: [rationale]
- **Evidence**: [literature reference]
- **Novelty**: low | medium | high

#### [Variant 2 Title]
...

### Summary
[Which substitutions are most promising]
```

**Failure conditions:**
- Missing required sections (Substitution Analysis, Variants, Summary)
- Variants missing required fields (what's substituted, why, evidence, novelty)
- Fewer than 2 variants generated

---

## Test 4: Parallel Execution (Advanced)

**Goal:** Verify CC can spawn multiple subagents in parallel.

**Prompt:**

```
Read skills/spawn-agent/SKILL.md. Execute TWO subagents in parallel:

Subagent A (Critic):
- prompt: "You are a hostile academic reviewer. Find fatal flaws in this idea. Search for papers that contradict or supersede it. Output: ## Flaws (numbered list with evidence)"
- input: IDEA: "Using knowledge distillation with attention transfer for LLM compression"

Subagent B (Defender):
- prompt: "You are a defender of this research idea. Find supporting evidence and address potential criticisms. Search for papers that validate the approach. Output: ## Strengths (numbered list with evidence)"
- input: IDEA: "Using knowledge distillation with attention transfer for LLM compression"

Spawn both as parallel Agent tool calls in a single message.
```

**Expected behavior:**
- CC invokes TWO Agent tool calls in a single message (parallel)
- Both subagents execute independently
- Both use MCP tools to find evidence
- Results are returned and presented together

**Failure conditions:**
- CC spawns subagents sequentially (one at a time)
- Only one subagent is spawned
- CC tries to answer as Critic/Defender itself without spawning subagents

---

## Success Criteria

1. CC correctly reads and follows SKILL.md execution protocol
2. Input formatted with UPPER_CASE field names and blank line separators
3. Agent tool invoked with correct structure (prompt.md + --- + input)
4. Subagent uses at least one MCP tool for evidence gathering
5. Output matches the structure defined in prompt.md
6. Parallel execution works (Test 4)
7. Default model is opus (visible in Agent tool call)
