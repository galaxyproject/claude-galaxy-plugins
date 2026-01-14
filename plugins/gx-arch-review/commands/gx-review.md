Perform a code review on the provided Galaxy code. Accept input in any of following forms:
* A working directory path (analyze git diff in that directory)
* A Git commit reference (analyze changes in that commit)
* A PR reference (analyze changes in that pull request)
* A list of Python file paths (analyze those files)
* A planning document (analyze the Python files in the plan)

This Claude command should orchestrate a review of the supplied code. There are preconditions
and other target commands in the same directory as this command for each precondition. This agent
should evaluate each precondition and if it is met, launch a subagent to perform the review
of the supplied changes to this agent.

- Precondition: These changes contain Python Code.
  Command: py-review-code-structure.md
- Precondition: These changes contain client test code (src/client/*test.js or src/client/*test.ts)
  Command: gx-vitest-review.md
- Precondition: These changes contain Python code that add or modify an API endpoint.
  Command: py-fastapi-review.md
- Precondition: These changes contain Python code that add or modify an API endpoint.
  Command: gx-review-business-logic-organization.md
- Precondition: These changes contain Python code/
  Command: gx-review-di.md
- Precondition: These changes contain Python tests.
  Command: py-challenge-patches.md
- Precondition: These changes contain an alembic database migration.
  Command: gx-review-migration.md
- Precondition: These changes contain a new database model.
  Command: gx-review-model.md
