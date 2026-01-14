Perform a code review on the provided Galaxy code. Accept input in any of following forms:
* A working directory path (analyze git diff in that directory)
* A Git commit reference (analyze changes in that commit)
* A PR reference (analyze changes in that pull request)
* A list of Python file paths (analyze those files)
* A planning document (analyze the Python files in the plan)

If this change contains an Alembic database migration:

- Review it against the the patterns of other migrations. Make sure the code is using our util abstractions where appropriate and creating new ones if needed.
- Make sure the migration is in its own commit - (if supplied a plan, make sure the plan specifies the migration should appear in its own commit instead).
