---
name: cmd-proposal
description: Create proposal for certain task
---
Write a proposal with name `YYMMDD-<slug>.md` at folder `.context/proposals/` (create if not existed) for the given task. If no task specified, it should be from the last conversation.

The proposal should carefully design the architecture, including bussiness logic, code, infra schema, etc.

Do not leave modifing history or questions in the proposal, discuss with the user to determine any uncertain points before finishing the proposal.

After finishing it, send the full proposal in the conversation without wrapper ```...```. After modifing, send only the updated pieces of proposal with its origin and modified like the unified git diff view (not real git diff).