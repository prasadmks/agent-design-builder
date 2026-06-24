# Agent Constitution Builder

A Claude skill suite for AI-native Product Managers.

## The Problem

Most teams building AI agents define 2-3 layers of the system — a rough persona, some rules, a basic flow. The other 5 layers get left to engineering to figure out. That's where hallucinations, production incidents, and behavior drift come from.

A production-grade AI agent needs 8 layers explicitly defined before engineering starts:

1. **Role + Persona** — who the agent is, tone, 
   behavioral identity
2. **Response Rules** — how the agent produces 
   output, tool call vs conversational boundaries
3. **Domain Reasoning Rules** — domain-specific 
   logic that prevents hallucinations
4. **Vocabulary Guardrails** — brand and legal 
   language constraints
5. **Constraint Matrix** — safety triggers, scope 
   boundaries, escalation conditions
6. **Context Schema** — runtime variables the 
   agent needs injected
7. **Behavior Spec** — the state machine, subtasks, 
   steps, triggers, branching logic
8. **Tool Registry** — every tool the agent can 
   call, parameters, error handling

Most teams define 2-3 of these. The other 5 get left to engineering. That's where production incidents come from.

## What This Does

This skill suite runs inside Claude Code. A PM describes what they are building — in plain language or by uploading a PRD — and the skills produce a complete, structured Agent Constitution across all 
8 layers. Developer-ready. Engineering picks it up without making product decisions.

## How It Works
