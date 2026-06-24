# Product Requirements Document
## Patient Self-Service AI Agent

### Overview
A conversational AI agent that helps patients 
manage their healthcare interactions across 
three core jobs: appointment management, 
care plan comprehension, and insurance 
navigation.

### Problem Statement
Patients spend significant time on hold or 
navigating complex phone trees to accomplish 
simple tasks — rescheduling an appointment, 
understanding what their care plan says, or 
checking if a procedure is covered. Most of 
these interactions don't require a human agent 
but currently have no self-service alternative 
that patients trust.

### Target Users
Patients of a mid-size regional health system. 
Mix of demographics — from younger patients 
managing routine care to older patients 
managing chronic conditions. Varying levels 
of health literacy and comfort with technology.

### Core Jobs to Be Done

**Job 1 — Appointment Management**
- Schedule, reschedule, cancel appointments
- Check appointment status and technician ETA
- Receive confirmation and tracking links
- Agent acts autonomously — no human review 
  required for routine transactions

**Job 2 — Care Plan Comprehension**
- Explain what the care team has documented 
  in plain language
- Surface upcoming steps and relevant details
- Never interpret, recommend, or advise — 
  explain only what the care team defined
- Any consequential question routes to 
  live human transfer

**Job 3 — Insurance Navigation**
- Answer coverage questions
- Check claim status
- Explain referral requirements
- Coverage disputes and denials route to 
  live human transfer

### Key Design Decisions

**Consequential question detection:**
The agent detects when a question crosses 
from informational to consequential — when 
the patient's next action depends on the 
answer. This triggers live transfer regardless 
of topic category.

**Transfer behavior:**
Agent always explains why it is transferring 
before handing off. Transparency is a trust 
design decision, not just UX.

**Clinical boundary:**
Agent never gives clinical advice. When a 
patient asks what they should do medically, 
the agent holds the boundary warmly and 
routes to the care team.

**Language:**
Plain language always. Clinical and insurance 
jargon must be explained in everyday terms. 
Reading level target: 8th grade or below.

### Out of Scope
- Clinical diagnosis or treatment recommendations
- Billing disputes
- New patient registration
- Anything requiring licensed clinical or 
  legal judgment

### Success Metrics
- Containment rate: % of contacts resolved 
  without human transfer
- Transfer appropriateness: % of transfers 
  that were correctly triggered
- Patient satisfaction: post-interaction score
- Care plan comprehension accuracy: % of 
  explanations validated against source record
