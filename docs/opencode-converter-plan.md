# BMAD to OpenCode Converter - Implementation Plan

## Objective

Create a deterministic converter that transforms BMAD source files into native OpenCode format following the Agent Skills open standard.

---

## Input → Output Overview

```
BMAD Source                          OpenCode Native Output
─────────────────────────────────    ─────────────────────────────────
src/modules/*/agents/*.agent.yaml    .opencode/agent/bmad-*.md
src/modules/*/sub-modules/           .opencode/agent/bmad-*--*.md
  claude-code/sub-agents/*.md          (subagents)
src/modules/*/workflows/*/           .opencode/skill/bmad-*/
  workflow.yaml + steps/ + templates/    SKILL.md + references/ + assets/
src/modules/*/tasks/*.md             .opencode/command/bmad/*.md
src/core/                            (merged into above)
```

---

## Output Directory Structure

```
.opencode/
├── agent/
│   ├── bmad-master.md                 # Primary - orchestrator
│   ├── bmad-pm.md                     # Primary - product manager
│   ├── bmad-architect.md              # Primary - architect
│   ├── bmad-dev.md                    # Primary - developer
│   ├── bmad-pm--analyst.md            # Subagent - PM's analyst
│   ├── bmad-pm--reviewer.md           # Subagent - PM's reviewer
│   └── bmad-cis--brainstorm.md        # Subagent - CIS brainstorm coach
│
├── command/
│   └── bmad/
│       ├── status.md                  # /bmad/status
│       ├── list-tasks.md              # /bmad/list-tasks
│       └── list-workflows.md          # /bmad/list-workflows
│
├── skill/
│   └── bmad/
│       ├── prd/
│       │   ├── SKILL.md               # Conductor
│       │   ├── references/            # Step files (on-demand)
│       │   │   ├── 01-problem-discovery.md
│       │   │   ├── 02-user-research.md
│       │   │   └── ...
│       │   └── assets/                # Templates
│       │       └── prd-template.md
│       │
│       ├── architecture/
│       │   ├── SKILL.md
│       │   ├── references/
│       │   └── assets/
│       │
│       └── brainstorm/
│           ├── SKILL.md
│           ├── references/
│           │   └── techniques/        # 150+ techniques
│           └── assets/
│
└── opencode.json                      # Optional config
```

---

## Conversion Rules

### 1. Primary Agents

**Source:** `src/modules/*/agents/*.agent.yaml`

**Transform:**

| YAML Field | Output Location |
|------------|-----------------|
| `metadata.name` | Frontmatter `name` |
| `metadata.title` | Frontmatter `description` |
| `persona.role` | Opening paragraph |
| `persona.identity` | Background section |
| `persona.communication_style` | "Your Style" section |
| `persona.principles` | "Principles" bulleted list |
| `critical_actions` | "Always" section (embedded, not runtime) |
| `menu` items | "Available Skills" + "Available Commands" lists |

**Output Format:**
```markdown
---
name: {metadata.name} ({metadata.title})
description: {persona.role}
mode: primary
---

# {metadata.title} - {metadata.name}

{persona.identity}

## Your Style
{persona.communication_style}

## Principles
- {principles as bullets}

## Always
- {critical_actions as bullets, with config values resolved}

## Available Skills
When structured work is needed:
- `bmad/{skill-name}` - {description from menu item}

## Available Commands
For quick actions:
- `/bmad/{command}` - {description}

## Subagents
Delegate specialized tasks to:
- `{subagent-name}` - {description}
```

**Excluded:**
- XML activation blocks
- Menu trigger syntax
- Handler dispatch logic
- Fuzzy matching instructions

---

### 2. Subagents

**Source:** `src/modules/*/sub-modules/claude-code/sub-agents/*.md`

**Transform:**
- Already markdown, minimal changes needed
- Add `mode: subagent` to frontmatter
- Rename with `bmad-{context}--{name}.md` pattern
- Ensure description clearly states when to invoke

**Naming Convention:**
| Source Path | Output Name |
|-------------|-------------|
| `bmad-planning/requirements-analyst.md` | `bmad-pm--requirements-analyst.md` |
| `bmad-analysis/codebase-analyzer.md` | `bmad-dev--codebase-analyzer.md` |
| `bmad-review/document-reviewer.md` | `bmad-pm--document-reviewer.md` |

**Output Format:**
```markdown
---
name: {name}
description: {description} - invoke when {context}
mode: subagent
---

{existing content, cleaned up}
```

---

### 3. Skills (Workflows)

**Source:** `src/modules/*/workflows/*/`
- `workflow.yaml` - metadata and config
- `workflow.md` or `instructions.md` - main instructions
- `steps/*.md` - step files
- `templates/*.md` - output templates

**Transform:**

**SKILL.md (Conductor):**
```markdown
---
name: bmad-{workflow-name}
description: {workflow.yaml description}
---

# {Workflow Title}

{Brief overview from workflow.md}

## Prerequisites
- {extracted from workflow.yaml config references}

## Process

### Step 1: {Step Title}
Follow instructions in @references/01-{step-name}.md

{Brief summary of what this step accomplishes}

### Step 2: {Step Title}
Follow @references/02-{step-name}.md

...

## Output
Generate using @assets/{template-name}.md
Save to: {workflow.yaml default_output_file}
```

**references/ (Step Files):**
- Copy step files from `steps/`
- Clean up any BMAD-specific syntax
- Resolve `{project-root}` and config placeholders

**assets/ (Templates):**
- Copy from `templates/`
- Clean up placeholders

**Size Constraint:**
- SKILL.md body should be <5k words per standard
- Detailed instructions go in `references/`

---

### 4. Commands (Simple Tasks)

**Source:**
- Menu items with `action` type
- Standalone tasks from `tasks/*.md`

**Transform:**

```markdown
---
description: {task description}
---

{action prompt content}

Reference: @{path to relevant file if needed}
```

**Examples:**
| Source | Command |
|--------|---------|
| `action: "list all tasks from manifest"` | `/bmad/list-tasks` |
| `action: "show workflow status"` | `/bmad/status` |

---

### 5. Configuration

**Source:** `src/modules/*/module.yaml` + `src/core/module.yaml`

**Transform:**
- Config values that were loaded at runtime become static in agent prompts
- Project-specific config (paths, project name) referenced as files
- Model/tool config goes in `opencode.json`

**opencode.json (optional):**
```json
{
  "agents": {
    "bmad-master": {
      "model": "claude-sonnet-4-20250514"
    }
  },
  "permissions": {
    "skill": {
      "bmad/*": "allow"
    }
  }
}
```

---

## Implementation Steps

### Phase 1: Core Infrastructure

1. **Create converter project structure**
   ```
   tools/opencode-converter/
   ├── convert.js              # Main entry point
   ├── lib/
   │   ├── parser/
   │   │   ├── agent-yaml.js   # Parse agent.yaml files
   │   │   ├── workflow.js     # Parse workflow directories
   │   │   └── subagent.js     # Parse existing subagent md
   │   ├── transformer/
   │   │   ├── primary-agent.js
   │   │   ├── subagent.js
   │   │   ├── skill.js
   │   │   └── command.js
   │   └── writer/
   │       └── output.js       # Write to .opencode/
   ├── templates/
   │   ├── primary-agent.hbs
   │   ├── subagent.hbs
   │   ├── skill.hbs
   │   └── command.hbs
   └── config/
       └── mappings.yaml       # Subagent context mappings, etc.
   ```

2. **Define mapping configuration**
   - Which subagents belong to which context (pm, dev, architect)
   - Which workflows become skills vs commands
   - Config value defaults

### Phase 2: Agent Conversion

3. **Implement agent.yaml parser**
   - Extract metadata, persona, menu, critical_actions
   - Handle Handlebars conditionals in source

4. **Implement primary agent transformer**
   - Apply template to generate markdown
   - Resolve menu items to skill/command references
   - Embed config values statically

5. **Implement subagent transformer**
   - Parse existing markdown
   - Add/update frontmatter
   - Apply naming convention

### Phase 3: Skill Conversion

6. **Implement workflow parser**
   - Read workflow.yaml for metadata
   - Discover step files in steps/
   - Discover templates in templates/

7. **Implement skill transformer**
   - Generate SKILL.md conductor
   - Copy and clean step files to references/
   - Copy templates to assets/
   - Resolve placeholders

### Phase 4: Command Conversion

8. **Implement command extraction**
   - Parse menu items with `action` type
   - Parse standalone tasks
   - Generate command markdown files

### Phase 5: Integration

9. **Create main converter script**
   - Accept source directory and output directory
   - Run all transformers
   - Generate opencode.json

10. **Add validation**
    - Verify SKILL.md <5k words
    - Check all references resolve
    - Validate frontmatter format

---

## Proof of Concept Scope

For initial validation, convert:

1. **Agents:**
   - `bmad-master` (core orchestrator)
   - `bmad-pm` (product manager)

2. **Subagents:**
   - `bmad-pm--analyst` (from market-researcher)
   - `bmad-pm--reviewer` (from document-reviewer)

3. **Skills:**
   - `bmad/prd` (PRD creation workflow)

4. **Commands:**
   - `/bmad/status`
   - `/bmad/list-tasks`

---

## Success Criteria

1. **Deterministic** - Same input always produces same output
2. **Valid format** - Output passes OpenCode/Agent Skills validation
3. **Functional** - Converted agents/skills work in OpenCode
4. **Context-efficient** - Skills use progressive disclosure correctly
5. **Maintainable** - Clear mapping from source to output

---

## Open Questions

1. **CIS Module** - Convert facilitators as primary agents or subagents?
   - Recommendation: Subagents (invoked for creative tasks)

2. **Party Mode** - How to handle multi-agent orchestration?
   - Recommendation: Defer to future phase, or convert to skill

3. **Workflow State** - Keep BMAD's frontmatter state tracking?
   - Recommendation: Yes, it works fine with skills

4. **Config Files** - Ship a default config or reference project files?
   - Recommendation: Reference project files, provide example

---

## Timeline

| Phase | Deliverable |
|-------|-------------|
| Phase 1 | Project structure, templates |
| Phase 2 | Agent conversion working |
| Phase 3 | Skill conversion working |
| Phase 4 | Command conversion working |
| Phase 5 | Full integration, validation |
| PoC | PM + PRD skill functional in OpenCode |
