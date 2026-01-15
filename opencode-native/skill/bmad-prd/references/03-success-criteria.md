# Step 3: Success Criteria Definition

**Progress: Step 3 of 11** | Next: User Journey Mapping

## Goal

Define comprehensive success criteria covering user success, business success, and technical success metrics.

## Sequence

### 1. Check for Existing Success Criteria

Review input documents for any success criteria already defined:
- Product brief may contain initial success metrics
- Research may have benchmark data
- Existing docs may have performance targets

If found, present them: "From your documents, I see these success criteria mentioned: [list]. Let's refine and expand on these."

### 2. Define User Success

Start with user-centered success:

"Let's define what success looks like, starting with users.

**User Success:**
- What would make a user say 'this was worth it'?
- What's the moment where they realize this solved their problem?
- After using this, what outcome are they walking away with?"

**Push for specificity:**
- NOT "users are happy" → "users complete [key action] within [timeframe]"
- Ask about emotional success: "When do they feel delighted/relieved/empowered?"
- Identify success moments: "What's the 'aha!' moment?"

### 3. Define Business Success

Transition to business metrics:

"Now let's look at success from the business perspective.

**Business Success:**
- What does success look like at 3 months? 12 months?
- Are we measuring revenue, user growth, engagement, something else?
- What metric would make you say 'this is working'?"

**Challenge vague metrics:**
- "10,000 users" → "What kind of users? Doing what?"
- "99.9% uptime" → "What's the real concern - data loss? Failed payments?"
- "Fast" → "How fast, and what specifically needs to be fast?"

### 4. Define Technical Success

For technical success criteria:
- Performance requirements (response time, throughput)
- Reliability requirements (uptime, error rates)
- Security requirements (based on domain/data sensitivity)
- Scalability requirements (expected growth)

### 5. Connect to Differentiator

Tie success metrics back to what makes the product special:

"So success means users experience [differentiator] and achieve [outcome]. Does that capture it?"

Adapt criteria to domain:
- **Consumer**: User love, engagement, retention
- **B2B**: ROI, efficiency, adoption
- **Developer tools**: Developer experience, community
- **Regulated**: Compliance, safety, validation

### 6. Scope Negotiation

Guide scope through success lens:

"Let's define scope levels:

1. **MVP** - What must work for this to be useful?
2. **Growth** - What makes it competitive?
3. **Vision** - What's the dream version?

For each feature idea, ask: Could that wait until after launch? Is that essential for proving the concept?"

### 7. Generate Content

Draft the success criteria section:

```markdown
## Success Criteria

### User Success
[User success criteria from conversation]

### Business Success
[Business success metrics from conversation]

### Technical Success
[Technical requirements from conversation]

### Measurable Outcomes
[Specific measurable outcomes with targets]

## Product Scope

### MVP - Minimum Viable Product
[MVP scope from conversation]

### Growth Features (Post-MVP)
[Growth features from conversation]

### Vision (Future)
[Future vision from conversation]
```

### 8. Confirm and Save

Present the draft to user:

"Here's the success criteria and scope I'll add: [show content]

Should I save this and continue to User Journey Mapping?"

On confirmation:
- Append content to PRD
- Update frontmatter: `stepsCompleted: [1, 2, 3]`
- Proceed to Step 4

## Domain Considerations

For regulated domains (healthcare, fintech, govtech):
- Include compliance milestones in success criteria
- Add regulatory approval timelines to MVP scope
- Consider audit requirements as technical success metrics

## Success Criteria

- User success criteria clearly identified and measurable
- Business success metrics defined with specific targets
- Technical success requirements appropriate to domain
- Scope properly negotiated (MVP, Growth, Vision)
- Content collaboratively developed (not assumed)
