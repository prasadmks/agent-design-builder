# /gap-report

## Purpose
Compares an existing Agent Design against 
an eval scorecard to surface misalignment — 
rules with no eval coverage, eval dimensions 
with no corresponding design rule, and 
contradictions between what was designed 
and what is being tested.

## How to Run
Paste your Agent Design XML and your 
eval scorecard. Claude will cross-reference 
them and produce a structured gap report.

## When to Use
- After completing an Agent Design — 
  before writing evals from scratch
- After a production incident — 
  to find whether the design or 
  the eval missed it
- When eval scores are inconsistent — 
  to find whether the design 
  is ambiguous
- Quarterly alignment check between 
  design and test coverage

## Input
Both of:
- Complete or partial Agent Design XML
- Eval scorecard — dimensions, 
  criteria, pass/fail definitions

## Output
File: `outputs/gap-reports/gap-[agent-name]-[date].xml`

```xml
<gap_report>

  <summary>
    <!-- Overall alignment assessment — 
    how well does the eval scorecard 
    cover the Agent Design -->
  </summary>

  <coverage_matrix>
    <!-- Per layer — which design rules 
    have eval coverage and which do not -->
  </coverage_matrix>

  <design_gaps>
    <!-- Eval dimensions that test 
    behavior not defined in the design — 
    engineering is implementing something 
    the PM never specified -->
  </design_gaps>

  <eval_gaps>
    <!-- Design rules with no corresponding 
    eval coverage — behavior that is 
    specified but never tested -->
  </eval_gaps>

  <contradictions>
    <!-- Places where the eval tests 
    for behavior that contradicts 
    what the design specifies -->
  </contradictions>

  <failure_classification>
    <!-- For each gap or contradiction — 
    classify as Type A, B, or C failure 
    risk and explain why -->
  </failure_classification>

  <recommendations>
    <!-- Prioritized actions — 
    update design, update eval, 
    or both -->
  </recommendations>

</gap_report>
```

## System Prompt

You are comparing an Agent Design against 
an eval scorecard to find misalignment.

Your job is to surface every place where 
the design and the eval are out of sync — 
so the PM can fix the right artifact 
before a production incident reveals it.

### The Three Failure Types

Every gap or contradiction falls into 
one of three categories:

**Type A — Design Gap:**
The eval is testing for behavior that 
is not defined in the Agent Design. 
Engineering implemented something — 
correctly or not — that the PM 
never specified.
Fix: update the Agent Design to 
define the behavior explicitly.

**Type B — Eval Gap:**
The Agent Design defines a rule but 
the eval scorecard has no dimension 
that tests for it. The behavior is 
specified but never verified.
Fix: add an eval dimension that 
tests the specified rule.

**Type C — Contradiction:**
The eval tests for behavior that 
directly contradicts what the 
Agent Design specifies.
Fix: decide which is correct — 
the design or the eval — and 
update the other to match.

### What to Cross-Reference

For each eval dimension ask:
- Is there a corresponding rule 
  in the Agent Design?
- If yes — does the eval criteria 
  match what the design specifies?
- If no — is this behavior that 
  should be in the design?

For each design rule ask:
- Is there a corresponding eval 
  dimension that tests this rule?
- If no — what failure would occur 
  in production if this rule 
  were violated?
- How critical is it to add coverage?

### Coverage Matrix Format

For each of the 8 layers produce:
LAYER: [name]
Design rules: [count]
Eval dimensions covering this layer: [count]
Coverage: [percentage]
Uncovered rules: [list]

### Failure Classification

For each gap or contradiction classify:
TYPE: A / B / C
DESCRIPTION: [what is misaligned]
PRODUCTION RISK: [what goes wrong if not fixed]
FIX: [update design / update eval / both]
PRIORITY: Critical / Important / Minor

### Priority Definitions

**Critical:** misalignment will cause 
or has caused a production incident

**Important:** misalignment creates 
inconsistent behavior or untested 
risk in production

**Minor:** coverage improvement — 
does not block launch

### What You Never Do
- Never assume the design is correct 
  and the eval is wrong — 
  or vice versa
- Never skip Type C contradictions — 
  these are the most dangerous 
  because both artifacts exist 
  but conflict
- Never recommend adding eval coverage 
  for behavior that should not 
  be in the design at all
- Never classify everything as critical — 
  be precise about severity

## Example Output

```xml
<gap_report>

  <summary>
    <!-- Agent Design has 8 layers complete. 
    Eval scorecard has 6 dimensions. 
    Coverage analysis: 4 layers have 
    partial or no eval coverage. 
    2 Type C contradictions found — 
    require immediate resolution. 
    Not ready for production without 
    addressing critical items. -->
  </summary>

  <coverage_matrix>
    - Role + Persona: 
      2 rules / 1 eval dimension / 50%
      Uncovered: identity integrity rules
    
    - Response Rules: 
      4 rules / 0 eval dimensions / 0%
      Uncovered: all response rules untested
    
    - Domain Reasoning: 
      5 rules / 2 eval dimensions / 40%
      Uncovered: forbidden assumptions rules
    
    - Vocabulary Guardrails: 
      3 rules / 1 eval dimension / 33%
      Uncovered: required substitutions
    
    - Constraint Matrix: 
      6 rules / 3 eval dimensions / 50%
      Uncovered: edge case handling rules
    
    - Context Schema: 
      4 variables / 0 eval dimensions / 0%
      Uncovered: all variable handling untested
    
    - Behavior Spec: 
      8 flows / 4 eval dimensions / 50%
      Uncovered: failure paths, closing flow
    
    - Tool Registry: 
      6 tools / 2 eval dimensions / 33%
      Uncovered: error handling for 4 tools
  </coverage_matrix>

  <design_gaps>

    TYPE: A
    EVAL DIMENSION: "Agent confirms 
    appointment details before executing 
    reschedule"
    DESIGN RULE: Not defined in 
    Behavior Spec — no Pre-Commit 
    Confirmation step exists
    PRODUCTION RISK: Engineering 
    implemented confirmation step 
    based on eval — but PM never 
    specified what confirmation 
    looks like or what happens 
    if patient changes mind
    FIX: Add Pre-Commit Confirmation 
    step to Behavior Spec with 
    branching logic
    PRIORITY: Critical

  </design_gaps>

  <eval_gaps>

    TYPE: B
    DESIGN RULE: Response Rules — 
    agent never produces conversational 
    text and tool call in same turn
    EVAL COVERAGE: None — no dimension 
    tests for mixed output
    PRODUCTION RISK: Most common source 
    of response quality failures — 
    agent produces filler text 
    before tool calls
    FIX: Add eval dimension — 
    "Agent never produces conversational 
    text and tool call in same turn"
    PRIORITY: Critical

    TYPE: B
    DESIGN RULE: Constraint Matrix — 
    safety trigger for medical emergency
    EVAL COVERAGE: None — no dimension 
    tests safety trigger behavior
    PRODUCTION RISK: Safety critical — 
    if trigger fires incorrectly or 
    fails to fire it is never caught 
    in testing
    FIX: Add eval dimension — 
    "Agent correctly identifies and 
    responds to safety triggers"
    PRIORITY: Critical

  </eval_gaps>

  <contradictions>

    TYPE: C
    EVAL DIMENSION: "Agent says 
    'Let me check that for you' 
    before looking up appointments"
    DESIGN RULE: Response Rules — 
    filler language before tool 
    calls is explicitly forbidden
    CONTRADICTION: Eval rewards 
    behavior the design forbids
    PRODUCTION RISK: Engineering 
    will implement to pass eval — 
    producing exactly the behavior 
    the design was trying to prevent
    FIX: Remove filler language 
    requirement from eval — 
    update to test for silent 
    tool execution
    PRIORITY: Critical

  </contradictions>

  <failure_classification>
    <!-- Summary of all issues by type -->
    
    Type A failures (design gaps): 1
    Type B failures (eval gaps): 2
    Type C contradictions: 1
    
    Total critical items: 4
    Total important items: 3
    Total minor items: 2
  </failure_classification>

  <recommendations>
    CRITICAL — address before production:
    1. Resolve Type C contradiction — 
       remove filler language requirement 
       from eval
    2. Add Pre-Commit Confirmation step 
       to Behavior Spec
    3. Add eval dimension for 
       mixed output rule
    4. Add eval dimension for 
       safety trigger behavior

    IMPORTANT — address before launch:
    5. Add eval coverage for 
       Response Rules layer — 
       currently 0%
    6. Add eval coverage for 
       Context Schema layer — 
       currently 0%
    7. Add failure path coverage 
       to Behavior Spec eval dimensions

    MINOR — improvement opportunities:
    8. Increase Tool Registry 
       error handling coverage
    9. Add identity integrity 
       eval dimension to 
       Role + Persona layer
  </recommendations>

</gap_report>
```

## Connected Commands
- `/audit-design` — audit the Agent Design 
  itself before comparing to eval
- `/design-agent` — rebuild thin layers 
  surfaced by the gap report
  

