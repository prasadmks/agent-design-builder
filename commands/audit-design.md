# /audit-design

## Purpose
Audits an existing Agent Design for gaps, 
conflicts, and missing coverage across 
all 8 layers. Produces a structured gap 
report with specific recommendations 
for each issue found.

## How to Run
Paste your existing Agent Design XML. 
Claude will analyze it across all 8 layers 
and produce a structured audit report.

## When to Use
- Before handing Agent Design to engineering
- After a production incident to find 
  the root cause in the design
- When adding a new capability to an 
  existing agent
- Quarterly agent health check
- When eval scorecard surfaces patterns 
  that trace back to design gaps

## Input
One of:
- Complete Agent Design XML
- Partial Agent Design XML 
  (audit will flag missing layers)
- Agent Design plus production 
  failure examples

## Output
File: `outputs/audits/audit-[agent-name]-[date].xml`

```xml
<audit_report>

  <summary>
    <!-- Overall assessment — how complete 
    and consistent is this Agent Design -->
  </summary>

  <layer_coverage>
    <!-- Coverage status per layer — 
    complete, partial, or missing -->
  </layer_coverage>

  <gaps>
    <!-- Specific gaps found — what is 
    missing, which layer, impact, 
    recommendation -->
  </gaps>

  <conflicts>
    <!-- Rules that contradict each other — 
    what conflicts, where, 
    how to resolve -->
  </conflicts>

  <engineering_decisions>
    <!-- Places where the design is thin 
    and engineering will have to make 
    product decisions — flag for PM -->
  </engineering_decisions>

  <recommendations>
    <!-- Prioritized list of changes — 
    critical, important, minor -->
  </recommendations>

</audit_report>
```

## System Prompt

You are auditing an Agent Design across 
all 8 layers.

Your job is to find every gap, conflict, 
and ambiguity that would cause an engineer 
to make a product decision — or cause a 
production incident.

### What to Look For

**Layer coverage:**
Check each of the 8 layers exists and 
has sufficient content:
- Role + Persona
- Response Rules
- Domain Reasoning Rules
- Vocabulary Guardrails
- Constraint Matrix
- Context Schema
- Behavior Spec
- Tool Registry

**Gaps:**
- Rules referenced but not defined
- Branches in behavior spec with 
  no defined outcome
- Tools called but not in tool registry
- Context variables used but not 
  in context schema
- Constraints referenced in behavior 
  spec but not in constraint matrix

**Conflicts:**
- Safety triggers that contradict 
  response rules
- Scope boundaries that conflict 
  with behavior spec flows
- Tool call rules that contradict 
  behavior spec steps
- Override hierarchy that is 
  ambiguous or circular

**Engineering decisions:**
- Sections marked as incomplete
- Vague trigger conditions that 
  require judgment to implement
- Missing error handling
- Parameter sources not specified
- Branches with no defined outcome

### Audit Output Format

For each issue found:
ISSUE TYPE: Gap / Conflict /

Engineering Decision

LAYER: [which layer]
LOCATION: [specific section or rule]
DESCRIPTION: [what is missing or wrong]
IMPACT: [what goes wrong in production]
RECOMMENDATION: [specific fix]
PRIORITY: Critical / Important / Minor

### Priority Definitions

**Critical:** will cause production 
incident or safety failure if not fixed

**Important:** will cause inconsistent 
behavior or poor user experience

**Minor:** improvement opportunity — 
does not block launch

### What You Never Do
- Never generate fixes — 
  only identify issues and 
  recommend what to address
- Never mark a layer complete 
  if it has vague or generic content
- Never skip engineering decisions — 
  these are as important as gaps
- Never prioritize everything as critical — 
  be honest about severity

## Example Output

```xml
<audit_report>

  <summary>
    <!-- Agent Design is 6 of 8 layers complete. 
    Context Schema and Tool Registry are missing. 
    3 critical gaps found in Constraint Matrix. 
    2 conflicts between Response Rules and 
    Behavior Spec. Not ready for engineering 
    handoff without addressing critical items. -->
  </summary>

  <layer_coverage>
    - Role + Persona: complete
    - Response Rules: partial
    - Domain Reasoning: complete
    - Vocabulary Guardrails: complete
    - Constraint Matrix: partial
    - Context Schema: missing
    - Behavior Spec: partial
    - Tool Registry: missing
  </layer_coverage>

  <gaps>

    ISSUE TYPE: Gap
    LAYER: Constraint Matrix
    LOCATION: Escalation Conditions
    DESCRIPTION: Escalation trigger defined 
    for patient requesting human agent but 
    no trigger defined for agent being 
    unable to fulfill request after retry
    IMPACT: Agent will loop indefinitely 
    on unresolvable requests instead 
    of escalating
    RECOMMENDATION: Add escalation trigger 
    for "agent cannot fulfill after one retry"
    PRIORITY: Critical

    ISSUE TYPE: Gap
    LAYER: Behavior Spec
    LOCATION: Rescheduling subtask — 
    Get Available Windows step
    DESCRIPTION: No branch defined for 
    patient changing their mind 
    mid-rescheduling flow
    IMPACT: Agent has no defined behavior 
    when patient says "never mind" 
    after seeing options
    RECOMMENDATION: Add branch — 
    "patient abandons flow → Closing subtask"
    PRIORITY: Important

    ISSUE TYPE: Gap
    LAYER: Tool Registry
    DESCRIPTION: Entire layer missing — 
    behavior spec references 4 tool calls 
    with no specifications
    IMPACT: Engineering cannot implement 
    tool integrations without making 
    product decisions
    RECOMMENDATION: Run /tool-registry 
    skill to generate this layer
    PRIORITY: Critical

  </gaps>

  <conflicts>

    ISSUE TYPE: Conflict
    LAYERS: Response Rules + Behavior Spec
    LOCATION: Response Rules silent tool 
    call rule vs Behavior Spec 
    Question Handling step
    DESCRIPTION: Response Rules states 
    agent never produces conversational 
    text and tool call in same turn. 
    Behavior Spec Question Handling 
    step says agent answers question 
    then calls knowledge tool.
    IMPACT: Ambiguous — engineer will 
    implement one interpretation, 
    PM may expect the other
    RECOMMENDATION: Clarify sequence — 
    either answer first then call tool 
    in next turn, or call tool first 
    then answer from output
    PRIORITY: Critical

  </conflicts>

  <engineering_decisions>

    ISSUE TYPE: Engineering Decision
    LAYER: Constraint Matrix
    LOCATION: Safety trigger — 
    medical emergency
    DESCRIPTION: Trigger defined as 
    "patient explicitly states a medical 
    emergency" but no confidence threshold 
    specified — engineer will decide 
    what counts as explicit
    RECOMMENDATION: PM to define 
    specific trigger phrases or 
    confidence threshold
    PRIORITY: Important

  </engineering_decisions>

  <recommendations>
    CRITICAL — address before handoff:
    1. Add Tool Registry layer — 
       run /tool-registry
    2. Add Context Schema layer — 
       run /context-schema
    3. Resolve Response Rules vs 
       Behavior Spec conflict on 
       Question Handling sequence
    4. Add missing escalation trigger 
       to Constraint Matrix

    IMPORTANT — address before launch:
    5. Add patient abandons flow branch 
       to Rescheduling subtask
    6. Define confidence threshold 
       for safety trigger

    MINOR — improvement opportunities:
    7. Add before/after examples 
       to Vocabulary Guardrails
    8. Add tool chaining documentation 
       to Tool Registry
  </recommendations>

</audit_report>
```

## Connected Commands
- `/gap-report` — compares Agent Design 
  against eval scorecard
- `/design-agent` — rebuild thin or 
  missing layers
