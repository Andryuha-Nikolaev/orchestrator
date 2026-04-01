# Orchestrator

This repository defines a multi-agent workflow for turning user requests into safe, phased execution.

## Agents

- **Orchestrator**: routes requests, applies stage gates, and coordinates handoffs between agents.
- **Brainstorming**: resolves ambiguity with one-question iterations, 2-3 approaches, trade-offs, a recommendation, and an explicit approval gate.
- **Planner**: converts approved direction into actionable execution steps.
- **Coder**: implements code changes for non-design phases.
- **Designer**: implements UI/UX-focused phases when visual or frontend design work is required.

## Execution Flow

1. **Step 0: Ambiguity/Creative Gate**
   - If request is ambiguous, has multiple plausible interpretations, or needs creative exploration: call Brainstorming first.
   - If key requirements or constraints are missing: ask targeted clarifying question(s) first, then route to Brainstorming before planning.
   - If request is clear and implementation-ready: continue directly to Step 1.
2. **Explicit User Approval**
   - Execution does not continue until the user explicitly approves the selected direction.
3. **Step 1: Planner**
   - Planner receives approved context and creates an execution-ready plan.
4. **Step 2: Phase Parsing by File Overlap**
   - Parse phases by target file overlap to preserve execution order and avoid conflicting edits.
5. **Step 3: Execution by Coder/Designer**
   - Route each parsed phase to Coder or Designer based on the phase type and affected files.

## Important Boundaries

- Brainstorming does not write code.
- Brainstorming does not replace Planner.
