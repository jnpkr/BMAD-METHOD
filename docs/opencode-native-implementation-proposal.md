# OpenCode Native Implementation Proposal for BMad Core and CIS

## Executive Summary

This document analyzes how BMad Core and the Creative Intelligence Suite (CIS) could be reimplemented using OpenCode's native agents, skills, and plugins instead of the current launcher-based approach. The goal is to leverage OpenCode's built-in architecture for a more integrated and performant experience.

---

## Current Architecture Analysis

### How BMAD Currently Works with OpenCode

The current OpenCode integration (`tools/cli/installers/lib/ide/opencode.js`) uses a **launcher pattern**:

1. **Agent Launchers** → `.opencode/agent/bmad-agent-{module}-{name}.md`
   - Small markdown files with `mode: primary`
   - Contain activation instructions that reference the actual agent file in `_bmad/`
   - The AI must load and interpret the referenced agent file at runtime

2. **Workflow Commands** → `.opencode/command/bmad-{module}-{workflow-name}.md`
   - Flat structure for workflow invocation
   - References workflow files in `_bmad/`

3. **Task/Tool Commands** → `.opencode/command/bmad-task-{module}-{name}.md`
   - Standalone tasks and tools as slash commands

### Limitations of Current Approach

| Issue | Impact |
|-------|--------|
| **Indirection overhead** | AI must load agent file, parse persona, interpret activation steps |
| **No native subagent support** | Can't leverage OpenCode's subagent system for orchestration |
| **No skill integration** | Workflows aren't registered as OpenCode skills |
| **No MCP tool integration** | BMAD tools not available as native OpenCode tools |
| **Menu system workaround** | BMAD's menu system conflicts with OpenCode's native commands |

---

## OpenCode Native Capabilities

Based on research of [OpenCode documentation](https://opencode.ai/docs/) and [GitHub repository](https://github.com/sst/opencode):

### 1. Native Agent System

**Primary Agents** (switchable via Tab key):
- `build` - Full access development agent
- `plan` - Read-only analysis agent
- Custom agents via `.opencode/agent/*.md` with `mode: primary`

**Subagents** (invoked by primary agents):
- `general` - Multi-step task handling
- `explore` - Codebase exploration
- Custom subagents via `mode: subagent`

**Key Configuration Options:**
```yaml
# opencode.json agent configuration
{
  "agents": {
    "custom-agent": {
      "model": "claude-3.5-sonnet",
      "tools": {
        "bash": true,
        "read": true,
        "write": false  # Can restrict tools per agent
      }
    }
  }
}
```

### 2. Skills System

Skills are markdown files discovered at startup:
- **User skills**: `~/.config/opencode/skills/`
- **Project skills**: `.opencode/skills/`

Skills can be loaded via the `skill` tool during conversation.

### 3. MCP Server Integration

External tools via Model Context Protocol:
```json
{
  "mcp": {
    "my-server": {
      "type": "local",
      "command": "node",
      "args": ["./mcp-server.js"]
    }
  }
}
```

### 4. Custom Tools

Define callable functions in config:
```json
{
  "tools": {
    "my-tool": {
      "description": "What this tool does",
      "command": "node ./my-tool.js"
    }
  }
}
```

---

## Implementation Proposal

### Architecture Overview

```
OpenCode Native BMAD Architecture
├── Primary Agents (.opencode/agent/)
│   ├── bmad-master.md (mode: primary) ─── Master Orchestrator
│   ├── bmad-pm.md (mode: primary) ─────── Product Manager
│   ├── bmad-architect.md (mode: primary)─ Architect
│   ├── bmad-dev.md (mode: primary) ────── Developer
│   └── bmad-{role}.md ─────────────────── Other roles
│
├── Subagents (.opencode/agent/)
│   ├── bmad-analyst.md (mode: subagent) ─ Market Research
│   ├── bmad-reviewer.md (mode: subagent)─ Document Review
│   ├── bmad-evaluator.md (mode: subagent) Technical Evaluation
│   └── bmad-cis-*.md (mode: subagent) ─── CIS Facilitators
│
├── Skills (.opencode/skills/)
│   ├── bmad-prd.md ────────────────────── PRD Creation Skill
│   ├── bmad-architecture.md ───────────── Architecture Skill
│   ├── bmad-epics-stories.md ──────────── Epics & Stories
│   ├── bmad-brainstorm.md ─────────────── Brainstorming Skill
│   ├── bmad-design-thinking.md ────────── Design Thinking Skill
│   └── bmad-{workflow}.md ─────────────── Other workflow skills
│
├── MCP Server (optional)
│   └── bmad-mcp-server/ ───────────────── BMAD Tools as MCP
│       ├── manifest-tools.js ──────────── List tasks/workflows
│       ├── status-tools.js ────────────── Workflow status
│       └── config-tools.js ────────────── Config management
│
└── Configuration
    └── opencode.json ──────────────────── Agent/tool config
```

---

### Detailed Implementation

#### 1. Primary Agents (BMad Core Roles)

Transform agent.yaml files into native OpenCode primary agents:

**Example: BMad Master Agent**
```markdown
---
name: BMad Master
description: Master Task Executor, Knowledge Custodian, and Workflow Orchestrator
mode: primary
---

# BMad Master

You are the **BMad Master Executor**, a master-level expert in the BMAD Core Platform. You serve as the primary execution engine for BMAD operations.

## Identity
- Expert in all BMAD modules with comprehensive knowledge of resources, tasks, and workflows
- Direct and comprehensive communication style, referring to yourself in 3rd person
- Present information systematically using numbered lists

## Configuration
On activation, load and parse:
- `{project-root}/_bmad/core/config.yaml` for user_name, communication_language, output_folder
- `{project-root}/_bmad/bmm/config.yaml` for project settings

Always address the user by {user_name} and communicate in {communication_language}.

## Capabilities

You have access to these subagents for specialized tasks:
- **analyst** - Market research and requirements analysis
- **reviewer** - Document quality review
- **evaluator** - Technical evaluation
- **brainstorm-coach** - Creative brainstorming facilitation

You can invoke skills for structured workflows:
- **skill:bmad-prd** - Product Requirements Document creation
- **skill:bmad-architecture** - Technical architecture design
- **skill:bmad-epics-stories** - Epic and story breakdown

## Commands

When the user asks about tasks or workflows:
1. Read `_bmad/_config/task-manifest.csv` to list available tasks
2. Read `_bmad/_config/workflow-manifest.csv` to list available workflows
3. Present as numbered list for easy selection

When executing a workflow, invoke the appropriate skill.
```

**Example: Product Manager Agent**
```markdown
---
name: John (PM)
description: Investigative Product Strategist + Market-Savvy Product Manager
mode: primary
---

# Product Manager - John

You are **John**, a Product Management veteran with 8+ years launching B2B and consumer products. Expert in market research, competitive analysis, and user behavior insights.

## Communication Style
- Ask 'WHY?' relentlessly like a detective on a case
- Direct and data-sharp, cut through fluff to what matters
- Back all claims with data and user insights

## Core Principles
- Uncover the deeper WHY behind every requirement
- Ruthless prioritization to achieve MVP goals
- Proactively identify risks and align with measurable business impact

## Available Workflows

Invoke these skills for structured work:
- **skill:bmad-prd** → Create Product Requirements Document
- **skill:bmad-epics-stories** → Create Epics and User Stories
- **skill:bmad-implementation-readiness** → Implementation Readiness Review
- **skill:bmad-correct-course** → Course Correction Analysis

## Subagent Delegation

Use these subagents for specialized analysis:
- **analyst** for market research and competitive analysis
- **reviewer** for document quality checks
- **evaluator** for technical feasibility assessment

## Project Context
Always search for and treat `**/project-context.md` as the primary reference.
```

#### 2. Subagents (Specialized Assistants)

Current BMAD "sub-agents" in `sub-modules/claude-code/` map directly to OpenCode subagents:

**Example: Market Researcher Subagent**
```markdown
---
name: Market Researcher
description: Specialized in market research, competitive analysis, and industry trends
mode: subagent
---

# Market Researcher

You are a specialized market research analyst. When invoked, you:

1. **Analyze the request** - Understand what market information is needed
2. **Research systematically** - Use available tools to gather data
3. **Synthesize findings** - Present actionable insights
4. **Return to caller** - Provide structured output for the primary agent

## Capabilities
- Competitive landscape analysis
- Market sizing and opportunity assessment
- User persona development
- Industry trend identification

## Output Format
Always structure findings as:
- Executive Summary
- Key Findings (bulleted)
- Recommendations
- Data Sources
```

**Example: Brainstorming Coach Subagent (CIS)**
```markdown
---
name: Carson (Brainstorm Coach)
description: Elite Brainstorming Specialist - Master facilitator for breakthrough sessions
mode: subagent
---

# Carson - Elite Brainstorming Specialist

You are **Carson**, an elite facilitator with 20+ years leading breakthrough sessions. Expert in creative techniques, group dynamics, and systematic innovation.

## Communication Style
- Enthusiastic improv coach energy
- Build on ideas with "YES AND"
- Celebrate wild thinking
- Use humor and play as innovation tools

## Core Principles
- Psychological safety unlocks breakthroughs
- Wild ideas today become innovations tomorrow
- No idea is too crazy to explore

## Facilitation Process

When invoked for brainstorming:

### 1. Frame the Challenge
- Clarify the core problem
- Set creative constraints (optional)
- Establish psychological safety

### 2. Divergent Thinking
Apply techniques like:
- SCAMPER (Substitute, Combine, Adapt, Modify, Put to other uses, Eliminate, Reverse)
- Random Word Association
- Reverse Brainstorming
- Worst Possible Idea

### 3. Convergent Thinking
- Cluster similar ideas
- Dot voting (priority)
- Feasibility check

### 4. Action Items
- Top 3 ideas to explore
- Next steps for each
- Owner assignments (if applicable)

## Available Skills
Can invoke **skill:bmad-brainstorm** for structured multi-step facilitation.
```

#### 3. Skills (Workflow Execution)

Transform BMAD workflows into OpenCode skills:

**Example: PRD Creation Skill**
```markdown
# skill:bmad-prd

## Product Requirements Document (PRD) Workflow

This skill guides the creation of a comprehensive Product Requirements Document.

### Prerequisites
- Load config from `_bmad/bmm/config.yaml`
- Check for existing PRD at `{planning_artifacts}/prd.md`
- Load project context from `**/project-context.md` if exists

### Step 1: Problem Discovery
Elicit from the user:
- What problem are we solving?
- Who experiences this problem?
- Why hasn't this been solved before?
- What happens if we don't solve it?

### Step 2: User Research Synthesis
- Target user personas
- User journey pain points
- Jobs to be done

### Step 3: Solution Requirements
For each requirement, capture:
- Requirement ID
- Description
- Priority (Must/Should/Could/Won't)
- Acceptance Criteria
- Dependencies

### Step 4: Success Metrics
- Key Performance Indicators
- Success thresholds
- Measurement approach

### Step 5: Scope and Timeline
- In-scope features
- Out-of-scope items
- Phasing recommendations

### Output
Generate PRD document at: `{planning_artifacts}/prd.md`

Use the template at: `_bmad/bmm/workflows/2-plan-workflows/prd/templates/prd-template.md`
```

**Example: Brainstorming Skill (CIS)**
```markdown
# skill:bmad-brainstorm

## Structured Brainstorming Workflow

### Configuration
- Output: `{output_folder}/brainstorm-session-{date}.md`
- Techniques library: `_bmad/core/workflows/brainstorming/techniques/`

### Phase 1: Setup (5 min)
1. Capture the challenge/question
2. Set ground rules (no judgment, quantity over quality)
3. Choose 2-3 techniques to use

### Phase 2: Ideation (Variable)
For each selected technique:
1. Explain the technique briefly
2. Generate ideas (aim for 20+ per technique)
3. Record ALL ideas without filtering

### Phase 3: Clustering (10 min)
1. Group similar ideas
2. Name each cluster
3. Identify outliers worth keeping

### Phase 4: Prioritization (10 min)
Rate each cluster on:
- Impact (1-5)
- Effort (1-5)
- Innovation (1-5)

### Phase 5: Action Planning
For top 3 ideas:
- Define next concrete step
- Identify owner
- Set timeline

### Output Document
```yaml
session_date: {date}
challenge: "{challenge_statement}"
participants: [user]
techniques_used: []
ideas_generated: 0
top_ideas:
  - idea: ""
    impact: 0
    effort: 0
    next_step: ""
```
```

#### 4. MCP Server (Optional Advanced Integration)

For deeper integration, create an MCP server that exposes BMAD functionality as tools:

**bmad-mcp-server/index.js**
```javascript
// MCP Server exposing BMAD tools to OpenCode
const { Server } = require('@modelcontextprotocol/server');

const server = new Server({
  name: 'bmad-tools',
  version: '1.0.0',
  description: 'BMAD Method tools for OpenCode'
});

// Tool: List available workflows
server.addTool({
  name: 'bmad_list_workflows',
  description: 'List all available BMAD workflows',
  parameters: {
    module: { type: 'string', optional: true }
  },
  handler: async ({ module }) => {
    const manifest = await readManifest('workflow-manifest.csv');
    return module ? manifest.filter(w => w.module === module) : manifest;
  }
});

// Tool: Get workflow status
server.addTool({
  name: 'bmad_workflow_status',
  description: 'Get current workflow status for the project',
  handler: async () => {
    const statusPath = findStatusFile();
    return statusPath ? await readYaml(statusPath) : { status: 'not_initialized' };
  }
});

// Tool: List available agents
server.addTool({
  name: 'bmad_list_agents',
  description: 'List all available BMAD agents',
  handler: async () => {
    return await readManifest('agent-manifest.csv');
  }
});

server.start();
```

**opencode.json configuration:**
```json
{
  "mcp": {
    "bmad": {
      "type": "local",
      "command": "node",
      "args": ["./_bmad/mcp-server/index.js"]
    }
  }
}
```

#### 5. Configuration (opencode.json)

Central configuration for the native implementation:

```json
{
  "agents": {
    "bmad-master": {
      "model": "claude-3.5-sonnet",
      "description": "BMAD Master Orchestrator"
    },
    "bmad-pm": {
      "model": "claude-3.5-sonnet",
      "description": "Product Manager - John"
    },
    "bmad-architect": {
      "model": "claude-3.5-sonnet",
      "description": "Technical Architect"
    },
    "bmad-dev": {
      "model": "claude-3.5-sonnet",
      "description": "Senior Developer"
    }
  },
  "default_agent": "bmad-master",
  "mcp": {
    "bmad": {
      "type": "local",
      "command": "node",
      "args": ["./_bmad/mcp-server/index.js"]
    }
  },
  "permissions": {
    "skill": "allow"
  }
}
```

---

## Migration Strategy

### Phase 1: Core Infrastructure
1. Create base agent templates for primary agents
2. Implement skill loader for workflows
3. Test with BMad Master and one module (BMM)

### Phase 2: Agent Migration
1. Convert all BMM agents to native primary agents
2. Convert Claude Code sub-agents to native subagents
3. Preserve persona/communication style from YAML definitions

### Phase 3: Workflow Migration
1. Convert step-based workflows to skill format
2. Maintain state tracking via output files
3. Support workflow continuation

### Phase 4: CIS Module
1. Convert all CIS facilitator agents to subagents
2. Create brainstorming, design-thinking, problem-solving skills
3. Integrate creative techniques library

### Phase 5: MCP Integration (Optional)
1. Create MCP server for tool access
2. Expose manifest queries, status checks, config tools
3. Enable cross-agent tool sharing

---

## Comparison: Current vs Native

| Aspect | Current (Launcher) | Native OpenCode |
|--------|-------------------|-----------------|
| Agent Loading | Runtime file reference | Direct persona in agent file |
| Switching Agents | Menu selection | Tab key cycling |
| Subagent Invocation | Manual @ mention | Automatic by primary agent |
| Workflow Execution | Slash command → file reference | Skill invocation |
| Tool Access | Via agent instructions | Per-agent tool config |
| State Management | _bmad/ files | Same, but simpler access |
| Configuration | YAML in _bmad/ | opencode.json + _bmad/ |

---

## Benefits of Native Implementation

1. **Performance**: No indirection overhead - agents load directly
2. **User Experience**: Tab switching between agents, natural subagent delegation
3. **Maintainability**: Simpler file structure, standard OpenCode patterns
4. **Extensibility**: Easy to add new agents/skills following patterns
5. **Tool Integration**: Native MCP support for advanced tooling
6. **Consistency**: Same patterns as other OpenCode users expect

---

## Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| Loss of menu system | Document common commands, use skills for complex flows |
| Agent proliferation | Limit primary agents to 5-7, use subagents for specialists |
| Workflow complexity | Break into skills, use state files for continuation |
| Backward compatibility | Keep _bmad/ structure, support both approaches during transition |

---

## Recommended Next Steps

1. **Prototype**: Create native versions of BMad Master + PM + one workflow skill
2. **User Testing**: Validate the native experience with existing BMAD users
3. **Installer Update**: Modify `opencode.js` installer to generate native format
4. **Documentation**: Create user guide for native OpenCode BMAD usage
5. **CIS Integration**: Apply same pattern to Creative Intelligence Suite

---

## References

- [OpenCode Documentation](https://opencode.ai/docs/)
- [OpenCode Agents Guide](https://opencode.ai/docs/agents/)
- [OpenCode Tools & MCP](https://opencode.ai/docs/tools/)
- [OpenCode GitHub Repository](https://github.com/sst/opencode)
- [MCP Server Protocol](https://modelcontextprotocol.io/)

---

*Document created: 2025-12-28*
*Author: Claude Code Analysis*
