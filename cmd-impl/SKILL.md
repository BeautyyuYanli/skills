---
name: cmd-impl
description: Run the multi-agent implementation and staged-review loop described in this skill. Use only when the user explicitly names `cmd-impl` or explicitly asks to use this skill's described workflow; do not use it for ordinary coding or implementation requests.
---

The loop should be orchestrated by Mei. Others should follow their own role.
To implement code, the loop is described as follows:

1. Spawn exactly one Elysia subagent at the start of the loop and explicitly tell her to load `persona-elysia`. Reuse this same subagent for every implementation and rework turn so Elysia retains the full loop context.
2. By default the task starts from Elysia to implement the task if not specified.
3. Divide the numbered review stages into three Eden groups: stages 1–2, stages 3–4, and stage 5. When the workflow enters a group, spawn exactly one new Eden subagent with no inherited conversation context (`fork_turns: none` or equivalent) and explicitly tell it to load `persona-eden`. Reuse that Eden across every stage and retry within the group.
4. Give each newly spawned Eden only the current task, review group, relevant constraints, and current implementation artifacts needed for that review. When continuing with the same Eden, provide the current numbered stage, updated implementation artifacts, and any other new information needed for the next review.
5. Once Eden finishes with a FAILED result, send the reviewed feedback to the persistent Elysia subagent for implementation.
6. Create a new Eden only when the workflow transitions to a different Eden group. Reuse the stages 1–2 Eden when moving between or retrying stages 1 and 2, reuse the stages 3–4 Eden when moving between or retrying stages 3 and 4, and use a separate Eden for stage 5. If the workflow leaves a group and later returns to it, treat that return as a new group visit and create a new Eden. Each newly created Eden must use `fork_turns: none` or equivalent.

Elysia keeps her context for the entire loop. Each Eden's context lasts for one continuous visit to an Eden group: reuse it across the group's stages and retries, then discard it when the workflow leaves that group. Returning to a group after leaving it requires a new Eden. Elysia and Eden do not share context, so you (Mei) should pass enough current information between them. If the task is presented as a proposal or doc file, pass the file path instead of its content.

Review every result from Eden before forwarding it to Elysia. Ensure the feedback stays focused on the current implementation goal, and do not let it unnecessarily expand the scope.

The review stages for Eden should be as follows. In every stage you (Mei) should request Eden to review only the scope of the stage.

1. Review code and logic hygiene, with deletion as the primary goal. Identify dead paths, duplication, unnecessary abstractions, other code smells, and custom code that unnecessarily reimplements functionality already available in the codebase, current dependencies, or a well-established and actively maintained library. If the review fails, return to Elysia and restart at stage 1.
2. Review if the implementation is good enough and the task(proposal, doc) is fully completed. If failed, back to Elysia and the next review starts from stage 1.
3. Review the hygiene of tests with deletion as the primary goal. Identify and remove redundant, meaningless, or misleading tests, including meaningless negative checks such as "something does not exist" and tests exhibiting the Texas sharpshooter fallacy, especially those introduced while editing the implementation. Prefer removing unnecessary tests over adding or restructuring tests. If failed, back to Elysia and the next review starts from stage 3.
4. Review if the tests are good enough. If failed, back to Elysia and the next review starts from stage 3.
5. Review whether the implementation's notes, docstrings, comments, and documentation are sufficient. Treat the proposal as implementation input rather than documentation: do not edit it or request changes to it during this stage. If failed, back to Elysia and the next review starts from stage 5.

Elysia should edit only in the scope of the review report, otherwise she should explicitly report, and you should guide Eden to the coresponding review stage in this case.

You (Mei) should simply wait when Elysia or Eden is working. Dont edit the code on your own when in the loop.

Once the loop is finished, you (Mei) should create a file in `.context/impl/` with the name `YYMMDD-<slug>.md` to describe the implementation, especially explain its diff (major additions and deletions) from the origin task. And then send the context without ```...``` wrapper to the conversation.
