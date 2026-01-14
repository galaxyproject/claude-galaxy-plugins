# gx-arch-review

A Claude Code plugin providing slash commands for reviewing Galaxy codebase contributions.

## Installation

This plugin is distributed via the `claude-galaxy-plugins` marketplace.

### Via Marketplace (Recommended)

```bash
# Add marketplace
/plugin marketplace add galaxyproject/claude-galaxy-plugins

# Install plugin
/plugin install gx-arch-review@claude-galaxy-plugins
```

### Development/Local Testing

```bash
claude --plugin-dir /path/to/galaxy-plugins
```

## Usage

All commands are namespaced under `gx-arch-review:`. They accept input in these forms:
- Working directory path (analyze git diff)
- Git commit reference
- PR reference (e.g., `#1234`)
- List of Python file paths
- Planning document (analyzes files referenced in plan)

### Primary Command: `gx-review`

```
/gx-arch-review:gx-review <input>
```

Orchestrates a comprehensive review by evaluating preconditions and launching appropriate sub-reviews. Automatically invokes relevant specialized commands based on change content.

### Specialized Commands

Run individually for targeted reviews:

| Command | Focus Area | Precondition |
|---------|------------|--------------|
| `/gx-arch-review:py-review-code-structure` | Type annotations, import organization | Python code |
| `/gx-arch-review:py-challenge-patches` | Mock/patch quality in tests | Python tests |
| `/gx-arch-review:gx-vitest-review` | Frontend unit test quality | Client test code (*.test.js/ts) |
| `/gx-arch-review:gx-fastapi-review` | FastAPI layer patterns | API endpoint changes |
| `/gx-arch-review:gx-review-migration` | Alembic migration patterns | Database migrations |
| `/gx-arch-review:gx-review-model` | Database model patterns | New database models |
| `/gx-arch-review:gx-review-test-types` | Test placement verification | Test files |
| `/gx-arch-review:review-business-logic-organization` | Controller/Service/Manager layers | API endpoint changes |
| `/gx-arch-review:review-di` | Dependency injection patterns | Python code |

## Command Details

### gx-review (Orchestrator)

Evaluates changes and spawns sub-agents for each applicable review type. Use this as your default entry point.

### py-review-code-structure

Reviews Python code for:
- Missing type annotations on public methods
- Inline imports without justification comments
- Import organization issues

### py-challenge-patches

Challenges test quality by examining mocks/patches:
- Identifies tests that only test mocks
- Suggests abstractions to eliminate patching (DI, fakes, ports/adapters)
- Prefers state verification over interaction verification

### gx-vitest-review

Reviews Vue/TypeScript unit tests for:
- Testing behavior vs implementation
- Proper Composition/Options API testing
- Pinia store isolation
- AI-generated test quality

### gx-fastapi-review

Ensures FastAPI layer follows Galaxy patterns:
- No manual `include_router` calls (Galaxy auto-detects routers)
- Thin controller layer

### gx-review-migration

Reviews Alembic migrations for:
- Proper use of Galaxy's migration utilities
- Migration isolation (own commit)

### gx-review-model

Reviews new database models for Galaxy patterns.

### gx-review-test-types

Verifies tests are in correct locations:

| Test Type | Location | When to Use |
|-----------|----------|-------------|
| Unit | `test/unit/` | No running server needed |
| API | `lib/galaxy_test/api/` | Server needed, default config |
| Integration | `test/integration/` | Server + custom config |
| Selenium | `lib/galaxy_test/selenium/` | Browser automation |

**Common mistake**: API test vs Integration test confusion
- API tests: default Galaxy config, can run against external servers
- Integration tests: require `handle_galaxy_config_kwds` or `self._app` access

### review-business-logic-organization

Enforces Galaxy's three-layer architecture:
- **Controllers**: Thin, request/response only
- **Services**: Optional thin layer for API concerns
- **Managers**: Business logic lives here
- **Models**: Database access only

### review-di

Enforces dependency injection patterns:
- Galaxy-style `depends()` over FastAPI `Depends()`
- Constructor injection over `app` property access
- `StructuredApp` interface over `UniverseApplication`
- `@galaxy_task` for Celery tasks

## Plugin Structure

This plugin is part of the `claude-galaxy-plugins` marketplace:

```
galaxy-plugins/
├── .claude-plugin/
│   └── marketplace.json   # Marketplace manifest
├── plugins/
│   └── gx-arch-review/
│       ├── commands/      # Slash commands
│       │   ├── gx-review.md
│       │   ├── py-review-code-structure.md
│       │   ├── gx-vitest-review.md
│       │   └── ...
│       └── README.md
└── README.md
```

## Relationship to galaxy-architecture Repository

This plugin is built from the [galaxy-architecture](https://github.com/jmchilton/galaxy-architecture) repository. See the [Agentic Code Review](https://github.com/jmchilton/galaxy-architecture#agentic-code-review) section in the root README for:

- Build infrastructure in `review/`
- The feedback loop between documentation, commands, and reviews
- How generated commands improve architecture documentation
- How to contribute new review commands.

## License

MIT License - see [LICENSE](../../LICENSE) for details.
