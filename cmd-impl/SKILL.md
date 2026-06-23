---
name: cmd-impl
description: The loop to implement code
---

The loop should be orchestrated by Mei. Others should follow their own role.
To implement code, the loop is described as follows:

1. Spawn two subagents: Elysia and Eden, explicitly tell them to load their coresponding persona skill `persona-<name_in_lowercase>`. Use model `gpt-5.4` with thinking level `high`
2. By default the task starts from the Elysia to implement the task if not specified.
3. Once Elysia finished, let Eden to review her implementation or modification.
4. Once Eden finished as a FALIED result, let Elysia to implement her review report.
5. Once Eden finished as a PASS result, go to the next stage of review until all are PASS.

Notice that they do not share the same context, you (Mei) should pass enough information between them. If the task is presented as a proposal or doc file, passing the file path instead of content.

The review stages for Eden should be as follows. In every stage you (Mei) should request Eden to review only the scope of the stage.

1. Review if the implementation is good enough and the task(proposal, doc) is fully completed. If failed, back to Elysia and the next review starts from stage 1.
2. Review the hygiene of code and logic, check if there are dead paths, oder in code or anything not elegant enough. If failed, back to Elysia and the next review starts from stage 1. 
3. Review if the tests are good enough. If failed, back to Elysia and the next review starts from stage 3. 
4. Review the hygiene of tests, check if there are useless tests like meaningless nagative checking "somethings doesnt exists", or Texas sharpshooter fallacy, etc. which are usually created during editing the implementation. If failed, back to Elysia and the next review starts from stage 3. 
5. Review if the notes, docstring, comments and docs is enough. If failed, back to Elysia and the next review starts from stage 5.

Elysia should edit only in the scope of the review report, otherwise she should explicitly report, and you should guide Eden to the coresponding review stage in this case.

You (Mei) should simply wait when Elysia or Eden is working. Dont edit the code on your own when in the loop.

Once the loop is finished, you (Mei) should create a file in `.context/impl/` with the name `YYMMDD-<slug>.md` to describe the implementation, especially explain its diff (major additions and deletions) from the origin task. And then send the context without ```...``` wrapper to the conversation.