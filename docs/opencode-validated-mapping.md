# OpenCode Native: Validated Feature Mapping

## Research Sources

- [OpenCode Agents Documentation](https://opencode.ai/docs/agents/)
- [OpenCode Skills Documentation](https://opencode.ai/docs/skills/)
- [OpenCode Commands Documentation](https://opencode.ai/docs/commands/)
- [Subagent Invocation Issue #3715](https://github.com/sst/opencode/issues/3715)
- [opencode-skills npm package](https://github.com/malhashemi/opencode-skills)

---

## OpenCode Feature Summary

### 1. Primary Agents
**Location:** `.opencode/agent/` or `~/.config/opencode/agent/`

```markdown
---
name: Agent Name
description: What this agent does
mode: primary
tools:
  write: true
  bash: true
---

Your agent system prompt here.
Reference subagents by name (without @) to delegate.
```

**Behavior:**
- User interacts directly with primary agents
- Switch between primary agents with `Tab` key
- Can invoke subagents by mentioning their name
- Can invoke skills via the `skill` tool
- Tools can be enabled/disabled per agent

### 2. Subagents
**Location:** Same as primary agents

```markdown
---
name: Subagent Name
description: Specialized task description (this is how primary agents discover it)
mode: subagent
tools:
  read: true
---

Your subagent instructions here.
```

**Behavior:**
- Invoked by primary agents automatically based on `description`
- User can manually invoke with `@subagent-name`
- Primary agent writes: `"file-writer create the file"` (no @ symbol)
- Subagent's description is the primary signal for when to invoke

### 3. Skills
**Location:** `skill/*/SKILL.md` (project) or `~/.config/opencode/skill/*/SKILL.md` (global)

```markdown
---
name: skill-name
description: What this skill does (min 20 chars, used for discovery)
---

# Skill Instructions

Step-by-step process for the agent to follow.
Can reference files in the skill folder: `scripts/helper.py`
```

**Behavior:**
- Loaded on-demand via `skill` tool
- Agent sees available skills and their descriptions
- Agent decides when to invoke based on task context
- Skill content injected into conversation when loaded
- Good for: multi-step processes, structured workflows

### 4. Commands
**Location:** `.opencode/command/` or `~/.config/opencode/command/`

```markdown
---
description: What this command does
argument-hint: [optional args]
model: claude-3-5-sonnet  # Optional override
---

Your prompt content here.
Use $ARGUMENTS for passed arguments.
Use @filename to include file content.
Use !bash-command to include command output.
```

**Behavior:**
- Triggered by user with `/command-name`
- Prompt content injected into conversation
- Supports file inclusion, bash output, arguments
- Good for: quick actions, templated prompts

---

## Interaction Model

```
┌─────────────────────────────────────────────────────────┐
│                        USER                             │
│                                                         │
│  Types message  ──────────►  PRIMARY AGENT              │
│  Types /command ──────────►  Command injects prompt     │
│  Types @subagent ─────────►  Manual subagent invoke     │
└─────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────┐
│                   PRIMARY AGENT                         │
│                                                         │
│  Can delegate to subagent:  "analyst research this"     │
│  Can invoke skill:          skill tool with skill name  │
│  Has configured tools:      read, write, bash, etc.     │
└─────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
         SUBAGENT          SKILL           TOOLS
         (delegated)       (loaded)        (direct)
```

---

## BMAD → OpenCode Mapping

### Direct Mappings

| BMAD Concept | OpenCode Feature | Transformation Needed |
|--------------|------------------|----------------------|
| Main Agents (PM, Architect, Dev) | Primary Agents | Extract persona, remove XML/menu |
| Sub-agents (analyzer, reviewer) | Subagents | Already close, add `mode: subagent` |
| Simple Tasks | Commands | Minimal - just prompt content |
| Complex Workflows | Skills | Restructure to SKILL.md format |
| Config Variables | System prompt + files | Reference config files directly |

### Eliminated Concepts

| BMAD Concept | Why Eliminated | OpenCode Alternative |
|--------------|----------------|---------------------|
| Menu system with triggers | Unnecessary indirection | Agent knows available commands/skills |
| Fuzzy matching dispatch | OpenCode handles naturally | LLM understands intent |
| XML activation blocks | Complexity for no benefit | Just markdown instructions |
| Handler types (exec, tmpl, workflow) | IDE abstraction | Direct file references |
| `[DA] Dismiss Agent` | Manual switching | Tab key switches agents |
| Runtime config loading | Overhead each conversation | Static in system prompt |

### Content Transformation

| BMAD Content | Transformation | Notes |
|--------------|----------------|-------|
| `persona.role` | Direct use | Opening line of agent prompt |
| `persona.identity` | Direct use | Background description |
| `persona.communication_style` | Direct use | "Your style:" section |
| `persona.principles` | Direct use | Bulleted list |
| `critical_actions` | Embed in prompt | "Always do:" section |
| `menu` items | List of commands/skills | "Available commands:" section |
| Workflow steps | Skill content | Restructure as numbered steps |
| Templates | Skill supporting files | `skill-name/templates/` |

---

## Validated Mapping: BMAD Agents → OpenCode

### PM Agent Example

**BMAD Source (pm.agent.yaml):**
```yaml
agent:
  metadata:
    name: John
    title: Product Manager
  persona:
    role: Investigative Product Strategist + Market-Savvy PM
    identity: Product management veteran with 8+ years...
    communication_style: Asks 'WHY?' relentlessly...
    principles: |
      - Uncover the deeper WHY behind every requirement
      - Ruthless prioritization to achieve MVP goals
  menu:
    - trigger: PR
      workflow: ".../prd/workflow.yaml"
      description: "[PR] Create PRD"
```

**OpenCode Output (.opencode/agent/bmad-pm.md):**
```markdown
---
name: John (Product Manager)
description: Investigative Product Strategist - asks WHY relentlessly
mode: primary
---

# Product Manager - John

You are John, a Product Management veteran with 8+ years launching B2B and consumer products. Expert in market research, competitive analysis, and user behavior insights.

## Your Communication Style
Ask 'WHY?' relentlessly like a detective on a case. Direct and data-sharp, cut through fluff to what matters. Back all claims with data and user insights.

## Core Principles
- Uncover the deeper WHY behind every requirement
- Ruthless prioritization to achieve MVP goals
- Proactively identify risks
- Align efforts with measurable business impact

## Project Context
Always check for and reference `**/project-context.md` as your primary source of truth.

## Available Skills
When the user needs structured deliverables, invoke these skills:
- `prd` - Create Product Requirements Document
- `epic-stories` - Break PRD into epics and user stories
- `implementation-readiness` - Review readiness for development

## Subagents You Can Delegate To
For specialized analysis, delegate to:
- `analyst` - "analyst research the market for [topic]"
- `reviewer` - "reviewer check this document for quality"
```

**Key Changes:**
- No XML, no activation blocks
- Menu becomes "Available Skills" + "Subagents"
- Persona embedded directly
- Config loaded from files, not runtime instructions

---

## Validated Mapping: BMAD Workflows → OpenCode Skills

### PRD Workflow Example

**BMAD Source Structure:**
```
prd/
├── workflow.yaml          # Config
├── workflow.md            # Instructions
├── steps/
│   ├── step-01-init.md
│   ├── step-02-problem.md
│   └── ...
└── templates/
    └── prd-template.md
```

**OpenCode Output (skill/prd/SKILL.md):**
```markdown
---
name: prd
description: Create a comprehensive Product Requirements Document through guided elicitation
---

# PRD Creation Skill

## Overview
This skill guides the creation of a Product Requirements Document (PRD) through systematic problem discovery and requirements elicitation.

## Prerequisites
- Check for existing PRD at `planning-artifacts/prd.md`
- Load project context from `**/project-context.md` if exists

## Process

### Step 1: Problem Discovery
Ask the user:
1. What problem are we solving?
2. Who experiences this problem most acutely?
3. What's the cost of NOT solving this problem?
4. Why hasn't this been solved before?

Capture responses before proceeding.

### Step 2: User Research Synthesis
Gather:
- Target user personas (2-3 primary)
- Key pain points in current workflow
- Jobs to be done

### Step 3: Requirements Elicitation
For each requirement:
- ID and description
- Priority (Must/Should/Could/Won't)
- Acceptance criteria
- Dependencies

### Step 4: Success Metrics
Define:
- 3-5 KPIs with specific targets
- Measurement approach
- Timeline for evaluation

### Step 5: Document Generation
Use template at `templates/prd-template.md` to generate final PRD.
Save to: `planning-artifacts/prd.md`

## Templates
Reference: @templates/prd-template.md
```

**Key Changes:**
- Step files consolidated into single SKILL.md
- Config values become direct file paths
- Template referenced with @ syntax
- Sequential steps preserved but simplified

---

## Validated Mapping: Commands

### Simple Actions → Commands

**BMAD:** Menu item with `action` handler
```yaml
- trigger: LT
  action: "list all tasks from manifest"
  description: "[LT] List Tasks"
```

**OpenCode (.opencode/command/list-tasks.md):**
```markdown
---
description: List all available BMAD tasks
---

List the tasks from @_bmad/_config/task-manifest.csv

Format as a numbered list with: name, module, description
```

---

## Configuration Mapping

### BMAD Config → OpenCode

**BMAD (_bmad/bmm/config.yaml):**
```yaml
project_name: "MyProject"
user_skill_level: "intermediate"
planning_artifacts: "planning-artifacts/"
implementation_artifacts: "implementation-artifacts/"
```

**OpenCode Approach:**

1. **Static in agent prompt:**
```markdown
## Project Configuration
- Planning artifacts: `planning-artifacts/`
- Implementation artifacts: `implementation-artifacts/`
```

2. **Or reference file:**
```markdown
## Configuration
Load project settings from @bmad-config.yaml when needed.
```

3. **opencode.json for tool/model config:**
```json
{
  "agents": {
    "bmad-pm": {
      "model": "claude-3-5-sonnet"
    }
  }
}
```

---

## What Needs Substantive Modification

### 1. Menu System → Eliminated
BMAD's menu system with fuzzy matching, triggers, and handlers is entirely replaced by:
- Agent knowing which skills/commands exist
- User invoking `/commands` directly
- Agent delegating to subagents naturally

**Substantive change:** Menu YAML completely removed, replaced with "Available Skills/Commands" section in agent prompt.

### 2. Workflow Step Files → Consolidated
BMAD splits workflows into many step files for "just-in-time loading" (context management).
OpenCode skills are single files.

**Options:**
- **Consolidate:** Merge steps into one SKILL.md (works for most)
- **Chunked:** For very long workflows, have skill reference supporting markdown files

### 3. XML Activation Blocks → Plain Markdown
All XML structure removed. Instructions written as clear markdown.

### 4. Handler Dispatch → Direct References
No more `exec=`, `workflow=`, `tmpl=` attributes. Just:
- Reference files with `@path/to/file`
- Invoke skills by name
- Delegate to subagents by mentioning them

### 5. State Tracking → Unchanged
BMAD's approach of tracking state in output file frontmatter works fine.
Skills can read/write state files normally.

---

## Recommended Converter Flow

```
1. Parse agent.yaml
   ├── Extract metadata (name, description)
   ├── Extract persona (role, identity, style, principles)
   ├── Extract menu items
   └── Determine: primary agent or subagent?

2. Generate agent markdown
   ├── Frontmatter (name, description, mode)
   ├── Persona as prose
   ├── Available skills/commands list (from menu)
   └── Subagent delegation instructions

3. Parse workflow directories
   ├── Read workflow.yaml for metadata
   ├── Read step files in order
   ├── Read templates
   └── Consolidate into SKILL.md

4. Generate skill markdown
   ├── Frontmatter (name, description)
   ├── Overview section
   ├── Steps (consolidated from step files)
   └── Copy supporting templates

5. Generate simple commands
   └── Menu items with `action` type → command files

6. Generate opencode.json
   ├── Agent model configurations
   └── Skill permissions
```

---

## Conclusion

**Direct mapping works for:**
- Agent personas (role, identity, style, principles)
- Subagent content
- Simple task prompts
- Template files
- State tracking approach

**Needs transformation:**
- Menu system → eliminated, replaced with skill/command awareness
- Multi-file workflows → consolidated into SKILL.md
- XML structure → plain markdown
- Config loading → static references

**No equivalent needed for:**
- Activation blocks
- Handler dispatch
- Fuzzy trigger matching
- Runtime config injection

The BMAD content is valuable; the BMAD machinery is what we're removing.
