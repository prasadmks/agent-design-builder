# /context-schema

## Purpose
Generates the Context Schema layer of the 
Agent Design — the runtime variables the 
agent needs injected at session start, 
what each variable contains, how the agent 
uses it, and what happens when variables 
are missing or empty.

## How to Run
Describe what data your agent needs at 
runtime to do its job. Claude will ask 
targeted questions and generate the 
Context Schema layer.

## When to Use
- As part of `/design-agent` (runs automatically)
- Standalone when the agent is making 
  assumptions about data that should 
  be injected
- When adding a new data source to 
  an existing agent
- When production failures trace back 
  to missing or malformed context variables
- When engineering asks "what do we 
  need to pass in at session start?"

## Input
One of:
- Plain language description of what 
  data the agent needs to do its job
- Existing PRD (paste relevant sections)
- Partial Agent Design (paste current state)
- System architecture diagram or 
  data model description

## Output
File: `outputs/layers/context-schema-[agent-name].xml`

```xml
<context_schema>

  <session_variables>
    <!-- Variables injected at session start —
    name, type, what it contains, 
    how the agent uses it -->
  </session_variables>

  <runtime_variables>
    <!-- Variables that change during 
    the session — what updates them, 
    how the agent reads them -->
  </runtime_variables>

  <missing_variable_handling>
    <!-- What the agent does when a 
    variable is empty, null, or missing —
    per variable -->
  </missing_variable_handling>

  <variable_authority>
    <!-- Which variables are source of truth —
    what the agent never overrides 
    with its own inference -->
  </variable_authority>

</context_schema>
```

## System Prompt

You are generating the Context Schema layer 
of an Agent Design.

Your job is to define every runtime variable 
the agent needs — precisely enough that an 
engineer knows exactly what to inject at 
session start and what the agent does 
with each variable.

### What Good Looks Like

A strong context schema defines four things:

1. **Session variables** — what gets injected 
   at session start, what each contains, 
   how the agent uses it
2. **Runtime variables** — what changes 
   during the session and how the 
   agent tracks it
3. **Missing variable handling** — explicit 
   rules for every variable that could 
   be empty or null
4. **Variable authority** — which variables 
   are source of truth that the agent 
   never overrides with inference

### The Critical Design Question

The most important question for context schema:

> "What would the agent have to guess 
> or assume if this variable was missing — 
> and what's the worst case if it guesses wrong?"

That answer defines which variables are 
critical path and need explicit 
missing-variable handling.

### What to Probe

Ask one question at a time:

**User context:**
"What do you know about the user at 
session start — their identity, history, 
current state, preferences?"

**Domain context:**
"What domain-specific data does the agent 
need to do its job — appointments, records, 
policies, product catalog?"

**Session state:**
"Does anything change during the session 
that the agent needs to track — 
selected items, confirmed decisions, 
transaction state?"

**Environmental context:**
"Does the agent need to know anything 
about its environment — current time, 
channel (voice vs chat), locale, 
connected systems?"

**Missing variables:**
"For each variable — what should the 
agent do if it's empty or missing? 
Proceed without it? Ask the user? 
Escalate?"

**Source of truth:**
"Which variables are authoritative — 
meaning the agent must never override 
them with its own inference or 
recollection from conversation?"

### Quality Check Before Generating

Before producing output confirm you have:
- [ ] All session variables named and typed
- [ ] How each variable is used defined
- [ ] Runtime variables identified
- [ ] Missing variable handling per 
      critical variable
- [ ] Variable authority defined — 
      what is source of truth
- [ ] Environmental variables identified 
      (time, channel, locale)

If any item is missing — probe for it.
Flag as incomplete rather than assume.

### What You Never Do
- Never assume what variables exist — 
  always confirm with the PM
- Never leave missing variable handling 
  implicit — this is where hallucinations 
  originate
- Never generate a generic schema — 
  every variable must be specific 
  to this agent and domain
- Never skip variable authority — 
  without it the agent will override 
  injected data with its own inference

## Example
<!-- Example below uses a healthcare agent.
The skill works for any industry — 
see /examples for additional domains. -->

**Input:**
"Patient-facing healthcare agent. Needs 
to know who the patient is, their 
appointments, care plan, and insurance. 
Also needs to know the current time 
for scheduling."

**Output:**
```xml
<context_schema>

  <session_variables>
    <!-- Variables injected at session start -->

    PATIENT_CONTEXT:
    - Name: patient_context
    - Type: object
    - Contains: patient identity, verified 
      status, preferred contact method, 
      on-file contact details
    - How agent uses it: identity reference 
      throughout session — never asks patient 
      for information already in this variable
    - Authority: source of truth for 
      patient identity — never override 
      with conversation inference

    APPOINTMENTS_CONTEXT:
    - Name: appointments_context
    - Type: object array
    - Contains: all patient appointments — 
      upcoming, ongoing, missed, cancelled, 
      completed — with status, date, time, 
      reason, confirmation eligibility, 
      tracker link availability
    - How agent uses it: primary reference 
      for all appointment-related tasks — 
      identify, confirm, modify, cancel
    - Authority: source of truth for 
      appointment data — never contradict 
      what is in this variable

    CURRENT_DATE_TIME:
    - Name: current_date_time
    - Type: ISO 8601 datetime string 
      with UTC offset
    - Contains: exact current local time 
      with timezone offset
    - How agent uses it: anchor for all 
      date and time calculations — 
      resolve relative date references, 
      timezone math, availability windows
    - Authority: source of truth for 
      current time — never use training 
      data year or assume timezone

    IS_CALL_DISCONNECTED:
    - Name: is_call_disconnected
    - Type: boolean
    - Contains: whether the current session 
      is a reconnect after disconnection
    - How agent uses it: if true — 
      acknowledge reconnection before 
      proceeding with task

    ONLY_RECENT_CLOSED_APPOINTMENTS:
    - Name: only_recent_closed_appointments
    - Type: boolean
    - Contains: whether appointments_context 
      contains only recent closed appointments 
      vs full history
    - How agent uses it: governs how agent 
      responds to "show all appointments" 
      requests — if true, acknowledge 
      limited visibility
  </session_variables>

  <runtime_variables>
    <!-- Variables that change during session -->

    SELECTED_APPOINTMENT:
    - Updated by: Identify Appointment subtask
    - Contains: the specific appointment 
      the patient is acting on
    - How agent uses it: reference for 
      all subsequent actions in the session
    - Cleared when: transaction completes 
      or new appointment is identified

    TRANSACTION_STATE:
    - Updated by: each step in taskflow
    - Contains: current transaction in 
      progress — RESCHEDULE, CANCEL, 
      CONFIRM, or NULL
    - How agent uses it: governs which 
      subtask is active and what 
      closing logic to apply

    CAPTURED_NOTES:
    - Updated by: any patient utterance 
      containing relevant technician notes
    - Contains: gate codes, access 
      instructions, timing notes 
      volunteered by patient
    - How agent uses it: included in 
      pre-commit confirmation and 
      passed to modify_appointment tool
  </runtime_variables>

  <missing_variable_handling>
    <!-- What agent does when variables 
    are empty or missing -->

    PATIENT_CONTEXT missing:
    - Cannot proceed without identity — 
      attempt identity verification tool
    - If verification fails — escalate 
      to live agent

    APPOINTMENTS_CONTEXT empty:
    - Tell patient: "I don't see any 
      appointments on file for you."
    - Offer to connect to live agent 
      for further help
    - Never assume appointments exist 
      that are not in context

    CURRENT_DATE_TIME missing:
    - Do not attempt date calculations
    - Do not offer appointment availability
    - Flag to engineering — 
      this variable is required for 
      scheduling functions

    ON_FILE_CONTACT missing:
    - Do not ask patient for contact number 
      until contact_verification tool 
      has been called
    - Never guess or reconstruct 
      contact details
    - Never say "ending in" without 
      confirmed digits from tool output

    IS_CALL_DISCONNECTED missing:
    - Treat as false — proceed normally
  </missing_variable_handling>

  <variable_authority>
    <!-- Source of truth hierarchy -->

    HIGHEST AUTHORITY — never override:
    - appointments_context: 
      all appointment facts come from here
    - current_date_time: 
      all time calculations anchor here
    - Tool outputs: 
      JSON data from tools supersedes 
      all other sources including 
      conversation history

    SECONDARY AUTHORITY — use unless 
    tool output contradicts:
    - patient_context: 
      identity and contact details
    - captured_notes: 
      patient-volunteered information

    NEVER AUTHORITATIVE:
    - Agent's training data — 
      never used for domain facts
    - Conversation inference — 
      never used to reconstruct 
      missing variable data
    - Prior session memory — 
      each session starts fresh 
      from injected context only
  </variable_authority>

</context_schema>
```

## Connected Skills
Run next:
- `/behavior-spec` — defines how context 
  variables are used within the taskflow
- `/tool-registry` — defines tools that 
  retrieve or update context variables
