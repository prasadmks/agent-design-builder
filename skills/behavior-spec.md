# /behavior-spec

## Purpose
Generates the Behavior Spec layer of the 
Agent Design — the state machine that defines 
what the agent does in every situation. 
Subtasks, steps, triggers, branching logic, 
edge cases, and failure paths — specific 
enough that engineering implements it 
without making product decisions.

## How to Run
/behavior-spec
Describe how your agent should behave 
across its core flows. Claude will ask 
targeted questions and generate the 
Behavior Spec layer.

## When to Use
- As part of `/design-agent` (runs automatically)
- Standalone when agent behavior is 
  inconsistent or unpredictable in production
- When adding a new flow to an existing agent
- When production failures trace back to 
  missing branching logic or edge cases
- When engineering asks "what should the 
  agent do when X happens?"

## Input
One of:
- Plain language description of the 
  agent's core flows
- Existing PRD with user stories or 
  flow diagrams
- Partial Agent Design (paste current state)
- Production failure examples showing 
  missing or incorrect flow logic

## Output
File: `outputs/layers/behavior-spec-[agent-name].xml`

```xml
<behavior_spec>

  <overview>
    <!-- High level summary of the agent's 
    core flows and how they connect -->
  </overview>

  <taskflow>

    <subtask name="">
      <!-- A distinct flow the agent handles -->

      <step name="">
        trigger: <!-- what initiates this step -->
        actions:
          <!-- numbered sequence of what 
          the agent does -->
        branches:
          <!-- what happens next based 
          on different outcomes -->
      </step>

    </subtask>

  </taskflow>

</behavior_spec>
```

## System Prompt

You are generating the Behavior Spec layer 
of an Agent Design.

Your job is to produce a complete state machine 
for this agent — precise enough that an engineer 
can implement every flow, branch, and failure 
path without making product decisions.

### What Good Looks Like

A strong behavior spec defines four things:

1. **Subtasks** — distinct flows the agent 
   handles, each with a clear entry trigger
2. **Steps** — the specific actions within 
   each subtask, in order
3. **Branches** — what happens at each 
   decision point based on different outcomes
4. **Failure paths** — what the agent does 
   when things go wrong at each step

### The Structure to Follow

Every subtask follows this pattern:
Subtask: [name]
Step: [name]
Trigger: [what initiates this step]
Actions: [numbered sequence]
Branches: [outcome → next step]

### What to Probe

Ask one question at a time:

**Core flows:**
"What are the three to five main things 
a user comes to this agent to do? 
List them — we will build a subtask 
for each."

**Entry points:**
"How does a conversation start — 
does the agent know why the user 
is calling before they speak, 
or does it need to figure that out?"

**Decision points:**
"For each core flow — what are the 
decision points where the agent 
needs to branch? What are the 
possible outcomes at each branch?"

**Failure paths:**
"For each core flow — what are the 
things that can go wrong? What should 
the agent do at each failure point?"

**Edge cases:**
"What are the situations that don't 
fit neatly into any core flow — 
users who change their mind mid-flow, 
requests that span multiple flows, 
or sequences that rarely happen 
but matter when they do?"

**Closing:**
"How does every conversation end — 
what does the agent do before 
the session closes regardless 
of which flow completed?"

### Quality Check Before Generating

Before producing output confirm you have:
- [ ] All core flows identified as subtasks
- [ ] Entry trigger for each subtask
- [ ] Steps defined for each subtask
- [ ] Decision branches at each step
- [ ] Failure path for each step
- [ ] Edge cases that don't fit 
      core flows handled
- [ ] Closing flow defined

If any item is missing — probe for it.
Flag as incomplete rather than assume.

### What You Never Do
- Never leave branches implicit — 
  every decision point needs 
  explicit outcomes
- Never skip failure paths — 
  they are as important as 
  the happy path
- Never generate a linear flow 
  without branching — real 
  conversations never are
- Never assume the closing flow — 
  always confirm with the PM

## Example
<!-- Example below uses a healthcare agent.
The skill works for any industry — 
see /examples for additional domains. -->

**Input:**
"Patient-facing healthcare agent. 
Core flows: appointment confirmation, 
rescheduling, cancellation, and 
care plan questions. Needs to identify 
which appointment before acting."

**Output:**
```xml
<behavior_spec>

  <overview>
    <!-- Four core flows: Appointment 
    Confirmation, Rescheduling, Cancellation, 
    and Question Handling. All flows that 
    involve a specific appointment first 
    route through the Identify Appointment 
    subtask. All flows close through the 
    Closing subtask which routes to 
    Call Wrap-up. -->
  </overview>

  <taskflow>

    <subtask name="Identify Appointment">
      <!-- Entry point for any flow requiring 
      a specific appointment to act on -->

      <step name="Detect Intent">
        trigger: Conversation starts OR 
        patient makes appointment-related request
        actions:
          1. Check appointments_context for 
             available appointments
          2. If one appointment exists — 
             select it automatically
          3. If multiple appointments exist — 
             present options and ask patient 
             to identify which one
          4. If no appointments exist — 
             go to No Appointments Found step
        branches:
          - One appointment → auto-selected → 
            return to calling subtask
          - Multiple appointments → patient 
            selects → return to calling subtask
          - No appointments → 
            No Appointments Found step
      </step>

      <step name="No Appointments Found">
        trigger: No appointments in 
        appointments_context
        actions:
          1. Tell patient no appointments 
             are on file
          2. Ask if there is anything 
             else to help with
          3. Wait for response
        branches:
          - Patient has another request → 
            route to appropriate subtask
          - Patient has no further needs → 
            Closing subtask
      </step>

    </subtask>

    <subtask name="Appointment Confirmation">

      <step name="Identify Target Appointment">
        trigger: Patient requests confirmation 
        or status of an appointment
        actions:
          1. Route to Identify Appointment 
             subtask
          2. Return here once appointment 
             is identified
        branches:
          - Appointment identified → 
            Status Lookup step
          - No appointment found → 
            Closing subtask
      </step>

      <step name="Status Lookup">
        trigger: Target appointment identified
        actions:
          1. Check confirmation_eligibility 
             in appointments_context
          2. If NOT ELIGIBLE — escalate 
             with scenario F
          3. If ELIGIBLE — retrieve status, 
             technician_status, technician_eta
             from appointments_context
        branches:
          - Not eligible → Escalation subtask
          - Eligible → Communicate Status step
      </step>

      <step name="Communicate Status">
        trigger: Status information available
        actions:
          1. Conversational response only — 
             no tool call this step
          2. State appointment status 
             in plain language
          3. Offer tracking link if available
        branches:
          - Tracking available → Offer Tracking step
          - Tracking not available → 
            Closing subtask
      </step>

    </subtask>

    <subtask name="Rescheduling">

      <step name="Identify Target Appointment">
        trigger: Patient requests rescheduling
        actions:
          1. Route to Identify Appointment 
             subtask
          2. Return here once identified
        branches:
          - Appointment identified → 
            Check Eligibility step
          - No appointment → Closing subtask
      </step>

      <step name="Check Eligibility">
        trigger: Target appointment identified
        actions:
          1. Check reschedule_eligibility 
             in appointments_context
          2. If NOT ELIGIBLE — 
             inform patient and escalate
          3. If ELIGIBLE — proceed
        branches:
          - Not eligible → Escalation subtask
          - Eligible → Get Available Windows step
      </step>

      <step name="Get Available Windows">
        trigger: Appointment eligible 
        for rescheduling
        actions:
          1. Call get_available_schedule_windows 
             tool silently
          2. Wait for tool output
          3. Present available options 
             in plain language
          4. Wait for patient selection
        branches:
          - Patient selects window → 
            Pre-Commit Confirmation step
          - No windows available → 
            Escalation subtask
          - Patient changes mind → 
            Closing subtask
      </step>

      <step name="Pre-Commit Confirmation">
        trigger: Patient has selected 
        a new window
        actions:
          1. Confirm selected date and time 
             with patient in plain language
          2. Include any captured notes
          3. Ask patient to confirm
          4. Wait for response
        branches:
          - Patient confirms → 
            Execute Reschedule step
          - Patient wants different time → 
            Get Available Windows step
          - Patient wants to cancel instead → 
            Cancellation subtask
      </step>

      <step name="Execute Reschedule">
        trigger: Patient confirmed new window
        actions:
          1. Call modify_appointment tool 
             silently with confirmed details
          2. Wait for tool output
          3. If success — confirm to patient
          4. If error — attempt once more
          5. If second error — 
             escalate to live agent
        branches:
          - Success → Closing subtask
          - Actionable error → retry once
          - General error → Escalation subtask
      </step>

    </subtask>

    <subtask name="Cancellation">

      <step name="Identify Target Appointment">
        trigger: Patient requests cancellation
        actions:
          1. Route to Identify Appointment 
             subtask
          2. Return here once identified
        branches:
          - Appointment identified → 
            Confirm Cancellation Intent step
          - No appointment → Closing subtask
      </step>

      <step name="Confirm Cancellation Intent">
        trigger: Target appointment identified
        actions:
          1. Confirm patient wants to cancel — 
             state appointment details
          2. Ask for explicit confirmation
          3. Wait for response
        branches:
          - Patient confirms → 
            Execute Cancellation step
          - Patient changes mind → 
            Closing subtask
      </step>

      <step name="Execute Cancellation">
        trigger: Patient confirmed cancellation
        actions:
          1. Call modify_appointment tool 
             silently with cancel intent
          2. Wait for tool output
          3. If success — confirm cancellation
          4. If error — attempt once more
          5. If second error — escalate
        branches:
          - Success → Closing subtask
          - Error → Escalation subtask
      </step>

    </subtask>

    <subtask name="Question Handling">

      <step name="Question Triage">
        trigger: Patient asks a question
        actions:
          1. Check if question is about 
             existing appointments — 
             if yes route to 
             Identify Appointment subtask
          2. Check if answer is in 
             appointments_context — 
             if yes answer directly
          3. Check if question is 
             out of scope — if yes 
             route to Escalation subtask
          4. Otherwise call 
             search_knowledge_agent tool
        branches:
          - Appointment question → 
            Identify Appointment subtask
          - In context → answer directly → 
            resume previous subtask
          - Out of scope → Escalation subtask
          - Knowledge needed → 
            Execute Knowledge Search step
      </step>

      <step name="Execute Knowledge Search">
        trigger: Answer not in context
        actions:
          1. Call search_knowledge_agent 
             with fully formed question
          2. Wait for tool output
          3. If answer found — respond 
             and resume previous subtask
          4. If no answer — 
             escalate OUT_OF_SCOPE
        branches:
          - Answer found → resume subtask
          - No answer → Escalation subtask
      </step>

    </subtask>

    <subtask name="Closing">

      <step name="Post Transaction Check">
        trigger: Transaction complete
        actions:
          1. Confirm result to patient 
             in plain language
          2. Ask if there is anything 
             else needed
          3. Wait for response
        branches:
          - Another request → 
            route to appropriate subtask
          - No further needs → 
            Call Wrap-up subtask
      </step>

    </subtask>

    <subtask name="Call Wrap-up">

      <step name="Execute Wrap-up">
        trigger: Conversation ending
        actions:
          1. Call call_wrap_up tool silently 
             with intent and resolution
          2. Wait for tool output
          3. Call end_session tool
        branches:
          - Success → session ends
          - Error → end session anyway
      </step>

    </subtask>

    <subtask name="Escalation">

      <step name="Execute Escalation">
        trigger: Any escalation condition met
        actions:
          1. Explain to patient why 
             transferring in one sentence
          2. Call escalation_call tool 
             with appropriate scenario
          3. Wait for tool output
          4. Route to Call Wrap-up subtask
        branches:
          - All scenarios → 
            Call Wrap-up subtask
      </step>

    </subtask>

  </taskflow>

</behavior_spec>
```

## Connected Skills
Run next:
- `/tool-registry` — defines every tool 
  called within this behavior spec
  


