# Galaxy Test Type Review Command

Perform a code review on the provided Galaxy code. Accept input in any of following forms:
* A working directory path (analyze git diff in that directory)
* A Git commit reference (analyze changes in that commit)
* A PR reference (analyze changes in that pull request)
* A list of Python file paths (analyze those files)
* A planning document (analyze the Python files in the plan)

## Primary Focus: Test Type Verification

The most common mistake is placing tests in the wrong location. Review each test file and verify it belongs in its current location.

## Galaxy Test Types Decision Tree

```
Does test need running Galaxy server?
├─ NO → Unit test (test/unit/ or doctests)
└─ YES → Does test need web browser?
         ├─ NO → Does test need custom Galaxy config?
         │       ├─ NO → Is it testing only tools or workflows?
         │       │       ├─ Tools → Framework test (test/functional/tools/)
         │       │       ├─ Workflows → Workflow Framework test (lib/galaxy_test/workflow/)
         │       │       └─ Neither → API test (lib/galaxy_test/api/)
         │       └─ YES → Integration test (test/integration/)
         └─ YES → Does test need custom config?
                  ├─ NO → Selenium test (lib/galaxy_test/selenium/)
                  └─ YES → Selenium Integration test (test/integration_selenium/)
```

## API vs Integration Test Distinction

### API Tests (`lib/galaxy_test/api/`)
**Use when ALL of these are true:**
- Test needs running Galaxy server
- Test does NOT need custom Galaxy configuration
- Test does NOT need direct access to Galaxy internals (`self._app`, database models)
- Test can theoretically run against any external Galaxy server

**Characteristics:**
- Inherits from `ApiTestCase`
- Uses populators: `DatasetPopulator`, `WorkflowPopulator`, `DatasetCollectionPopulator`
- Uses HTTP methods: `self._get()`, `self._post()`, `self._put()`, `self._delete()`
- Tests via the Galaxy API only

### Integration Tests (`test/integration/`)
**Use when ANY of these are true:**
- Test requires custom Galaxy configuration (enabled via `handle_galaxy_config_kwds`)
- Test requires Galaxy features that are off by default (quotas, specific object stores, job runners)
- Test requires direct access to `self._app` or database models
- Test requires external services started by the test (Docker containers, etc.)
- Test sets config options like: `enable_quotas`, `job_config_file`, `object_store_config_file`, `metadata_strategy`, `ftp_upload_dir`, custom auth settings

**Characteristics:**
- Inherits from `IntegrationTestCase`
- Overrides `handle_galaxy_config_kwds(cls, config)` to modify Galaxy config
- May set class attributes: `require_admin_user`, `framework_tool_and_types`
- May access `self._app` for internal state
- Each test class spins up its own Galaxy server

## Red Flags: Test in Wrong Location

### API test that should be Integration test:
- Contains `handle_galaxy_config_kwds` method (DEFINITE: should be integration)
- Accesses `self._app` (DEFINITE: should be integration)
- Imports from `galaxy.model` and queries database directly (LIKELY: should be integration)
- Comments mentioning "requires X to be enabled" but no config setup (LIKELY: missing config, should be integration)
- Uses `@skip_unless_docker()`, `@skip_unless_postgres()`, `@skip_unless_amqp()` (LIKELY: should be integration)

### Integration test that should be API test:
- Does NOT override `handle_galaxy_config_kwds` OR only calls `super().handle_galaxy_config_kwds(config)` with no modifications
- Does NOT access `self._app` or database models
- Only uses populator methods and HTTP assertions
- Could run against external Galaxy server with default config
- Test is in `test/integration/` but all it does is API calls with default config

## Review Output Format

For each test file reviewed, provide:

1. **Location**: Current file path
2. **Current Type**: What type of test it currently is based on location
3. **Correct Type**: What type it SHOULD be based on content analysis
4. **Status**: ✅ Correct | ⚠️ Misplaced | ❓ Needs Review
5. **Evidence**: Specific code patterns that determine the correct type
6. **Recommendation**: If misplaced, where it should go and what changes needed

## Example Analysis

```
File: test/integration/test_example.py
Current Type: Integration test
Correct Type: API test
Status: ⚠️ Misplaced

Evidence:
- No handle_galaxy_config_kwds override
- No self._app access
- Only uses DatasetPopulator and self._get/self._post
- No custom config requirements found

Recommendation: Move to lib/galaxy_test/api/test_example.py
- Change base class from IntegrationTestCase to ApiTestCase
- Update imports accordingly
```

## Additional Checks

Beyond test type placement, also flag:
- Tests that modify database directly when API would suffice
- Integration tests that could be split (config-dependent parts vs API-testable parts)
- Missing `super().handle_galaxy_config_kwds(config)` calls
- Unit tests with external dependencies that should use `@external_dependency_management` marker
