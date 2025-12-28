---
name: test-mechanics
description: Test workflow validating OpenCode skill mechanics. Use this skill when you need to verify that skill loading, reference file access, subagent delegation, state tracking, and output generation all work correctly. This is a diagnostic skill for testing the OpenCode implementation.
---

# Test Mechanics Workflow

A minimal 3-step workflow to validate all OpenCode skill mechanics work correctly.

## What This Tests

1. Skill loading via skill tool
2. Reference file loading from skill directory
3. Subagent delegation via task tool
4. State tracking in output file frontmatter
5. Output file generation from template

## Process

Execute these 3 steps in order:

| Step | Focus | Details |
|------|-------|---------|
| 1 | Setup | Create output file, confirm reference loading |
| 2 | Research | Delegate to market-researcher subagent |
| 3 | Complete | Finalize state, report results |

## Step 1: Setup

See [references/01-setup.md](references/01-setup.md) for detailed instructions.

Create the output file using template from [assets/output-template.md](assets/output-template.md).

## Step 2: Research

See [references/02-research.md](references/02-research.md) for detailed instructions.

Delegate a brief research task to the `market-researcher` subagent.

## Step 3: Complete

See [references/03-complete.md](references/03-complete.md) for detailed instructions.

Finalize the output and report all test results.

## Output

Save to: `test-output.md` in current working directory.
