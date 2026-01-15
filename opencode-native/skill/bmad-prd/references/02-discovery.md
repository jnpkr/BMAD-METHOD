# Step 2: Project & Domain Discovery

**Progress: Step 2 of 11** | Next: Success Criteria Definition

## Goal

Conduct comprehensive project discovery that leverages existing input documents while allowing user refinement, with data-driven classification.

## Sequence

### 1. Acknowledge Context

Start by acknowledging what you learned from Step 1:

"From our initialization, I have:
- [X] product briefs loaded
- [X] research documents loaded
- [X] project docs loaded

[If brownfield]: This is an existing project - I'll focus on what you want to add or change.
[If greenfield]: This is a new project - I'll help you define the full product vision."

### 2. Begin Discovery Conversation

**Select ONE path based on available documents:**

#### Path A: Has Product Brief

"I've reviewed your product brief. Let me share what I understand:

**What you're building:** [extracted from brief]
**Problem it solves:** [extracted from brief]
**Target users:** [extracted from brief]
**What makes it special:** [extracted from brief]

How does this align with your current vision? Should we refine any of these?"

#### Path B: Brownfield (No Brief, Has Project Docs)

"I've reviewed your existing project documentation.

**Your existing system includes:**
- Tech Stack: [from docs]
- Architecture: [from docs]
- Key Components: [from docs]

This PRD will define new features or changes to add.

**Tell me about what you want to add or change:**
- What new capability do you want to build?
- What problem will this solve for your users?
- How should it integrate with the existing system?"

#### Path C: Greenfield (No Documents)

"Let's shape your product vision.

**Tell me about what you want to create:**
- What problem does it solve?
- Who are you building this for?
- What excites you most about this product?

I'll listen for signals to help classify the project and domain."

### 3. Classify the Project

As the user describes their product, identify:

**Project Type:**
- Web application
- Mobile app
- API/Backend service
- CLI tool
- Desktop application
- Embedded system
- Other

**Domain:**
- E-commerce
- Healthcare
- Fintech
- Developer tools
- Social/Consumer
- Enterprise/B2B
- Other

**Complexity indicators:**
- Regulatory requirements
- Integration complexity
- Scale requirements
- Security needs

### 4. Validate Classification

"Based on our conversation, I'm classifying this as:

- **Project Type:** [type]
- **Domain:** [domain]
- **Complexity:** [low/medium/high]

Does this sound right?"

### 5. Identify Differentiator

Ask focused questions to capture what makes this special:

- "What would make users say 'this is exactly what I needed'?"
- "What assumption about this problem space are you challenging?"
- "If this succeeds wildly, what changed for your users?"

### 6. Generate Executive Summary

Based on the conversation, draft:

```markdown
## Executive Summary

[Vision and problem statement from conversation]

### What Makes This Special

[Differentiator content from conversation]

## Project Classification

**Technical Type:** [project_type]
**Domain:** [domain]
**Complexity:** [complexity]
**Project Context:** [Greenfield/Brownfield]
```

### 7. Confirm and Save

Present the draft Executive Summary to the user:

"Here's what I'll add to the PRD: [show content]

Should I save this and continue to Success Criteria?"

On confirmation:
- Append content to PRD
- Update frontmatter: `stepsCompleted: [1, 2]`
- Proceed to Step 3

## Success Criteria

- Correct discovery path selected based on available documents
- Input documents leveraged for head start
- User classifications validated
- Product differentiator clearly identified
- Executive summary generated collaboratively
- Content saved to PRD document
