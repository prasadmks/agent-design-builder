# /tool-registry

## Purpose
Generates the Tool Registry layer of the 
Agent Design — every tool the agent can 
call, what each tool does, when to call it, 
what parameters it takes, what it returns, 
and how the agent handles errors.

## How to Run
Describe the tools and external systems 
your agent needs to call. Claude will ask 
targeted questions and generate the 
Tool Registry layer.

## When to Use
- As part of `/design-agent` (runs automatically)
- Standalone when adding a new tool 
  to an existing agent
- When production failures trace back 
  to incorrect tool usage
- When engineering asks "what tools 
  does the agent need and when does 
  it call them?"
- When the agent is calling tools 
  incorrectly or at wrong times

## Input
One of:
- Plain language description of external 
  systems the agent needs to interact with
- Existing API documentation
- Partial Agent Design (paste current state)
- Production failure examples showing 
  incorrect tool usage

## Output
File: `outputs/layers/tool-registry-[agent-name].xml`

```xml
<tool_registry>

  <tool name="">
    <purpose>
      <!-- What this tool does in one sentence -->
    </purpose>
    <when_to_call>
      <!-- Exact trigger conditions — 
      what must be true before calling -->
    </when_to_call>
    <when_not_to_call>
      <!-- Explicit conditions where this 
      tool must not be called -->
    </when_not_to_call>
    <parameters>
      <!-- Each parameter — name, type, 
      where the value comes from -->
    </parameters>
    <returns>
      <!-- What the tool returns — 
      structure and how agent uses it -->
    </returns>
    <error_handling>
      <!-- What agent does for each 
      error type this tool can return -->
    </error_handling>
    <silent>
      <!-- true/false — whether agent 
      emits tool call silently or with 
      conversational context -->
    </silent>
  </tool>

</tool_registry>
```

## System Prompt

You are generating the Tool Registry layer 
of an Agent Design.

Your job is to produce a complete specification 
for every tool this agent can call — precise 
enough that an engineer knows exactly what 
to build and an agent knows exactly when 
and how to use each tool.

### What Good Looks Like

A strong tool registry defines six things 
for every tool:

1. **Purpose** — what the tool does 
   in one sentence
2. **When to call** — exact trigger 
   conditions, not vague guidance
3. **When not to call** — explicit 
   conditions that forbid calling
4. **Parameters** — every parameter 
   with type and source
5. **Returns** — what comes back 
   and how the agent uses it
6. **Error handling** — what the agent 
   does for each error type

### The Critical Design Questions

Two questions unlock the most important 
tool registry decisions:

> "What should the agent never do 
> before calling this tool — and 
> what should it never do instead 
> of calling this tool?"

> "What does the agent do when this 
> tool returns nothing, an error, 
> or unexpected data?"

Probe both for every tool.

### What to Probe

Ask one question at a time:

**Tool inventory:**
"Walk me through the external systems 
this agent needs to interact with — 
what does it need to read, what does 
it need to write, what does it need 
to trigger?"

**Trigger precision:**
"For each tool — what exact condition 
triggers the call? Not 'when the user 
asks about appointments' but 'when 
the user requests to reschedule and 
the appointment has been identified.'"

**Silent vs conversational:**
"For each tool — should the agent 
say something to the user before 
calling it, or call it silently?"

**Parameter sources:**
"For each parameter — where does 
the value come from? User input, 
context variable, prior tool output, 
or agent inference?"

**Error scenarios:**
"For each tool — what are the failure 
modes? Timeout, empty result, 
authorization error, bad parameter? 
What should the agent do in each case?"

**Chaining:**
"Are there tools that always follow 
another tool — where one tool's output 
triggers the next call?"

### Quality Check Before Generating

Before producing output confirm you have:
- [ ] All tools identified
- [ ] Purpose defined for each tool
- [ ] Exact trigger conditions per tool
- [ ] Explicit when-not-to-call per tool
- [ ] All parameters with types and sources
- [ ] Return structure and usage defined
- [ ] Error handling per error type
- [ ] Silent vs conversational defined
- [ ] Tool chaining relationships identified

If any item is missing — probe for it.
Flag as incomplete rather than assume.

### What You Never Do
- Never leave trigger conditions vague
- Never skip when-not-to-call — 
  this prevents the most common 
  tool misuse patterns
- Never assume parameter sources — 
  always confirm where values come from
- Never skip error handling — 
  tool failures are the most common 
  source of production incidents
- Never generate a tool spec based 
  on assumed API behavior — 
  only what the PM has confirmed

## Example
<!-- Example below uses a healthcare agent.
The skill works for any industry — 
see /examples for additional domains. -->

**Input:**
"Patient-facing healthcare agent. Needs 
tools for: getting available appointment 
windows, modifying appointments, verifying 
contact details, searching knowledge base, 
checking emergency status, escalating to 
live agent, and ending the session."

**Output:**
```xml
<tool_registry>

  <tool name="get_available_schedule_windows">
    <purpose>
      Retrieves available appointment windows 
      for rescheduling based on the current 
      appointment and patient location.
    </purpose>
    <when_to_call>
      Appointment has been identified AND 
      confirmed eligible for rescheduling AND 
      patient has requested to reschedule.
    </when_to_call>
    <when_not_to_call>
      - Before appointment is identified
      - Before eligibility is confirmed
      - When patient is only asking about 
        their current appointment status
      - When patient has not explicitly 
        requested rescheduling
    </when_not_to_call>
    <parameters>
      - appointment_number: string — 
        from appointments_context, 
        never ask patient
      - location_id: string — 
        from appointments_context
    </parameters>
    <returns>
      Array of available time windows — 
      each with date (YYYY-MM-DD), 
      timeRange, and slot_id.
      Agent presents options in plain 
      language using 12-hour format.
      Never alter date strings from output.
    </returns>
    <error_handling>
      - Empty array: inform patient no 
        windows available, escalate 
        to live agent
      - System error: inform patient, 
        escalate to live agent
      - Never retry more than once
    </error_handling>
    <silent>true</silent>
  </tool>

  <tool name="modify_appointment">
    <purpose>
      Executes appointment rescheduling 
      or cancellation — writes the change 
      to the scheduling system.
    </purpose>
    <when_to_call>
      Patient has explicitly confirmed 
      the new window OR cancellation 
      in Pre-Commit Confirmation step.
    </when_to_call>
    <when_not_to_call>
      - Before patient explicit confirmation
      - Before contact_verification 
        has completed if required
      - Never call to "check" — 
        this tool executes a transaction
    </when_not_to_call>
    <parameters>
      - appointment_number: string — 
        from appointments_context
      - action: enum — 
        RESCHEDULE or CANCEL
      - new_date: string YYYY-MM-DD — 
        exact string from 
        get_available_schedule_windows 
        output, never reconstructed
      - new_slot_id: string — 
        from get_available_schedule_windows
      - notes: string — 
        from captured_notes, optional
    </parameters>
    <returns>
      Confirmation of transaction — 
      success status, confirmation number.
      Agent confirms to patient 
      using returned details only — 
      never fabricates confirmation number.
    </returns>
    <error_handling>
      - Actionable error: correct 
        parameter and retry once
      - General error: inform patient, 
        escalate to live agent
      - Never retry more than once
    </error_handling>
    <silent>true</silent>
  </tool>

  <tool name="contact_verification">
    <purpose>
      Verifies or retrieves the contact 
      number to be used for appointment 
      confirmation messages.
    </purpose>
    <when_to_call>
      During rescheduling flow when 
      on_file_contact_description 
      does not contain a valid number.
    </when_to_call>
    <when_not_to_call>
      - When on_file_contact_description 
        already contains a valid number
      - Never call proactively before 
        patient initiates a transaction
    </when_not_to_call>
    <parameters>
      - patient_id: string — 
        from patient_context
    </parameters>
    <returns>
      response_guidance: string — 
      instructions for how agent 
      should handle contact number 
      in this specific case.
      Agent follows response_guidance 
      exactly — never overrides it.
    </returns>
    <error_handling>
      - No number found: follow 
        response_guidance instructions
      - System error: proceed without 
        contact number update, 
        note in captured_notes
    </error_handling>
    <silent>true</silent>
  </tool>

  <tool name="search_knowledge_agent">
    <purpose>
      Searches the knowledge base for 
      answers to patient questions about 
      policies, procedures, services, 
      and equipment.
    </purpose>
    <when_to_call>
      Patient asks a question that cannot 
      be answered from appointments_context 
      or work_information tag and is 
      not out of scope.
    </when_to_call>
    <when_not_to_call>
      - When answer exists in 
        appointments_context
      - When question is clearly 
        out of scope — escalate instead
      - Never call for appointment 
        lookup questions — use 
        appointments_context directly
    </when_not_to_call>
    <parameters>
      - request: string — fully formed 
        self-contained question with 
        all relevant context from 
        conversation included.
        Never pass vague keywords 
        or short query strings.
    </parameters>
    <returns>
      Answer text or no-answer signal.
      If answer found — respond using 
      returned text only, never supplement 
      with training data.
      If no answer — escalate 
      OUT_OF_SCOPE, never guess.
    </returns>
    <error_handling>
      - No answer returned: escalate 
        with scenario O, 
        intent OUT_OF_SCOPE
      - System error: inform patient, 
        escalate to live agent
    </error_handling>
    <silent>false</silent>
  </tool>

  <tool name="emergency_and_missed_status_check">
    <purpose>
      Checks for emergency conditions 
      or missed appointment status 
      when patient volunteers a 
      safety concern.
    </purpose>
    <when_to_call>
      Patient explicitly volunteers 
      a safety hazard in their utterance — 
      sparking, exposed wiring, 
      medical illness, total service loss.
    </when_to_call>
    <when_not_to_call>
      - Based on historical data in 
        appointments_context alone
      - Based on ambiguous phrasing 
        or phonetic similarity
      - Based on inferred tone — 
        only on explicit utterance
    </when_not_to_call>
    <parameters>
      - appointment_number: string — 
        from appointments_context
      - trigger_reason: string — 
        what the patient said that 
        triggered the check
    </parameters>
    <returns>
      Instructions for how to proceed — 
      agent follows returned instructions 
      exactly.
    </returns>
    <error_handling>
      - Error: escalate to live agent 
        immediately with safety scenario
    </error_handling>
    <silent>false</silent>
    <!-- EXCEPTION: this is the only tool 
    where conversational acknowledgment 
    AND tool call happen in same turn.
    Patient must hear response before 
    check initiates. -->
  </tool>

  <tool name="escalation_call">
    <purpose>
      Transfers the patient to a live 
      agent with context about why 
      the transfer is happening.
    </purpose>
    <when_to_call>
      Any escalation condition is met — 
      see Constraint Matrix for 
      full escalation trigger list.
    </when_to_call>
    <when_not_to_call>
      - Before emergency_and_missed_status_check 
        has been called for safety scenarios
      - Before agent has addressed any 
        pending patient question
    </when_not_to_call>
    <parameters>
      - scenario: enum — specific 
        escalation scenario code
      - intent: enum — what the 
        patient was trying to do
    </parameters>
    <returns>
      Transfer confirmation.
      Agent routes to Call Wrap-up 
      after escalation completes.
    </returns>
    <error_handling>
      - Error: inform patient transfer 
        failed, attempt once more
      - Second failure: end session 
        with apology and callback number
    </error_handling>
    <silent>true</silent>
  </tool>

  <tool name="call_wrap_up">
    <purpose>
      Closes the session with intent 
      and resolution summary — 
      logs the interaction outcome.
    </purpose>
    <when_to_call>
      Every conversation end — 
      after closing flow completes 
      or after escalation completes.
    </when_to_call>
    <when_not_to_call>
      - Never skip — every session 
        must call this tool
      - Never call before patient 
        interaction is complete
    </when_not_to_call>
    <parameters>
      - intent: enum — exact value 
        from allowed list, 
        never approximate
      - issue: string — what 
        patient requested
      - resolution: string — 
        what was done and 
        whether it succeeded
    </parameters>
    <returns>
      Wrap-up confirmation.
      Agent calls end_session 
      immediately after.
    </returns>
    <error_handling>
      - Error: call end_session anyway — 
        never leave session open
    </error_handling>
    <silent>true</silent>
  </tool>

  <tool name="end_session">
    <purpose>
      Terminates the session — 
      always the final action 
      in every conversation.
    </purpose>
    <when_to_call>
      Immediately after call_wrap_up 
      returns — no exceptions.
    </when_to_call>
    <when_not_to_call>
      - Before call_wrap_up has been called
      - Never call mid-conversation
    </when_not_to_call>
    <parameters>
      None.
    </parameters>
    <returns>
      Session termination confirmation.
    </returns>
    <error_handling>
      - Error: log and terminate anyway
    </error_handling>
    <silent>true</silent>
  </tool>

</tool_registry>
```

## Connected Skills
All 8 layers are now complete. Run:
- `/design-agent` — to generate a complete 
  Agent Design using all 8 skills in sequence
- `/audit-design` — to validate an existing 
  Agent Design for gaps and conflicts
  
