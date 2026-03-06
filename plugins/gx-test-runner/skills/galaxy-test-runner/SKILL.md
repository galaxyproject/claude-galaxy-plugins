---
name: galaxy-test-runner
description: "Run Galaxy test suites using ./run_tests.sh. Use this skill whenever the user asks to run tests, verify a fix, check if tests pass, or validate changes in the Galaxy codebase. This includes unit tests, API tests, integration tests, framework tool tests, framework workflow tests, selenium tests, and toolshed tests. Trigger on any mention of running tests, test failures, pytest, or test validation — even if the user just says 'run the tests' or 'does this pass?'"
---

# Galaxy Test Runner

Galaxy uses `./run_tests.sh` as the primary entry point for all test types. Always prefer it over running pytest directly — it handles virtualenv setup, dependency installation, common startup, and environment configuration that raw pytest would miss.

## Choosing the right test type

Pick the test type based on what changed and what the user is asking:

| What changed | Test type | Flag |
|---|---|---|
| Backend logic, models, utilities | Unit | `-unit` |
| API behavior, endpoint responses | API | `-api` |
| Tool XML, tool evaluation | Framework | `-framework` |
| Workflow evaluation logic | Framework workflows | `-framework-workflows` |
| Special Galaxy configurations | Integration | `-integration` |
| UI behavior | Selenium | `-selenium` |
| ToolShed functionality | ToolShed | `-toolshed` |

If you're unsure, look at the test file's location:
- `test/unit/` or doctests in `lib/` → `-unit`
- `lib/galaxy_test/api/` → `-api`
- `test/functional/tools/` → `-framework`
- `test/integration/` → `-integration`
- `lib/galaxy_test/selenium/` → `-selenium`

## Command patterns

### Running a specific test file
```bash
./run_tests.sh -api lib/galaxy_test/api/test_workflows.py
```

### Running a specific test class
```bash
./run_tests.sh -api lib/galaxy_test/api/test_tools.py::TestToolsApi
```

### Running a specific test method
```bash
./run_tests.sh -api lib/galaxy_test/api/test_tools.py::TestToolsApi::test_map_over_with_output_format_actions
```

### Using pytest selectors with -k
Pass extra pytest args after `--`:
```bash
./run_tests.sh -api lib/galaxy_test/api/test_workflows.py -- -k "test_run_workflow" -s
```

### Unit tests with a specific path
```bash
./run_tests.sh -unit test/unit/data/test_galaxy_mapping.py
```

### Framework tool tests by tool ID
```bash
./run_tests.sh -framework -id <tool_id>
```

### Framework workflow tests by name
```bash
./run_tests.sh -framework-workflows -id <workflow_name>
```

## Useful extra flags

- `-- -s` — show stdout/stderr (disable capture), useful for debugging
- `-- -x` — stop on first failure
- `-- -k "expression"` — select tests by name expression
- `--verbose_errors` — more verbose error reporting
- `--skip_flakey_fails` — skip known flakey tests

## Important notes

- API and integration tests start a full Galaxy server instance, so they take longer. Unit tests are fast.
- The `--` separator is needed to pass extra pytest arguments through `run_tests.sh`.
- Framework tests use `-id` (not `-k`) to select specific tools.
- If a test needs output visibility for debugging, always add `-- -s`.
- Tests produce an HTML report file (printed at the end of the run). The user rarely needs this but it's there if needed.

## Finding the right test to run

When the user says "run the tests" without specifying which ones, figure out what they changed:

1. Check `git diff` or recent file edits to see what was modified
2. Search for existing tests that cover the changed code — look in the corresponding test directory
3. If modifying `lib/galaxy/`, look for tests in `test/unit/`, `lib/galaxy_test/api/`, or `test/integration/`
4. Run the most specific test possible rather than a whole suite — faster feedback
