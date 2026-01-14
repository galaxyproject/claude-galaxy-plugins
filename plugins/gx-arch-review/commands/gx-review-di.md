# Review Galaxy Code for Dependency Injection Best Practices

Perform a Python code review on the provided code. Accept input in any of the following forms:
1. A working directory path (analyze git diff in that directory)
2. A Git commit reference (analyze changes in that commit)
3. A list of Python file paths (analyze those files)
4. A planning document (analyze the Python files in the plan)

## Review Criteria

Review the code and ensure it follows best practices for dependency injection in Galaxy:

1. **Avoid FastAPI-style injection** - Prefer Galaxy-style DI using Lagom and type-based injection
2. **Proper service/manager layer dependencies** - Dependencies should be injected via constructors, not via `app` access
3. **Avoid `app` property creation** - Don't add new properties to the `app` object when DI can be used

## Galaxy DI Architecture Background

### The Problem: God Object Anti-pattern

Galaxy's `app` object was historically a "god object" - an object that knows/does too much. Every component, controller, and web transaction had a reference to `app`. This creates:
- Circular dependencies (manager imports `UniverseApplication`, which creates that manager)
- Difficult testing (hard to know what parts of `app` are needed, hard to mock)
- Brittle construction ordering

### The Solution: Type-based Dependency Injection

Galaxy uses **Lagom** for type-based DI:
- Container keeps track of singletons and construction recipes
- Dependencies are resolved automatically based on type annotations
- Dependencies should form a **DAG (directed acyclic graph)**

### Interface Pattern for Breaking Cycles

Use `StructuredApp` interface instead of `UniverseApplication` to break circular dependencies:

```python
# BAD - creates circular dependency
class MyManager:
    def __init__(self, app: UniverseApplication):
        ...

# GOOD - uses interface, breaks cycle
class MyManager:
    def __init__(self, app: StructuredApp):
        ...

# BETTER - inject only what you need
class MyManager:
    def __init__(self, model: GalaxyModelMapping, security: IdEncodingHelper):
        ...
```

## Code Patterns to Review

### Manager/Service Pattern

**Anti-pattern - using `app` directly:**
```python
class DatasetCollectionManager:
    def __init__(self, app):
        self.model = app.model
        self.security = app.security
        self.hda_manager = hdas.HDAManager(app)  # BAD: internal construction
        self.history_manager = histories.HistoryManager(app)
```

**Correct pattern - injecting dependencies:**
```python
class DatasetCollectionManager:
    def __init__(
        self,
        model: GalaxyModelMapping,
        security: IdEncodingHelper,
        hda_manager: HDAManager,
        history_manager: HistoryManager,
    ):
        self.model = model
        self.security = security
        self.hda_manager = hda_manager
        self.history_manager = history_manager
```

**Benefits:**
- Type signature clearly delineates required dependencies
- Unit testing can inject precise mock dependencies
- No dependency on `app` object

### FastAPI Controller Pattern

**Anti-pattern - FastAPI Depends():**
```python
def get_tags_manager() -> TagsManager:
    return TagsManager()

@cbv(router)
class FastAPITags:
    manager: TagsManager = Depends(get_tags_manager)  # BAD: FastAPI style
```

**Correct pattern - Galaxy's depends():**
```python
@cbv(router)
class FastAPITags:
    manager: TagsManager = depends(TagsManager)  # GOOD: Galaxy style
```

### Legacy WSGI Controller Pattern

**Anti-pattern:**
```python
class TagsController(BaseAPIController):
    def __init__(self, app):
        super().__init__(app)
        self.manager = TagsManager()  # BAD: manual construction
```

**Correct pattern:**
```python
class TagsController(BaseGalaxyAPIController):
    manager: TagsManager = depends(TagsManager)  # GOOD: DI
```

### Celery Task Pattern

**Correct pattern - using @galaxy_task:**
```python
@celery_app.task(ignore_result=True)
@galaxy_task
def purge_hda(hda_manager: HDAManager, hda_id):
    hda = hda_manager.by_id(hda_id)
    hda_manager._purge(hda)
```

Dependencies are automatically injected based on type annotations.

**Task with multiple dependencies:**
```python
@celery_app.task
@galaxy_task
def set_metadata(
    hda_manager: HDAManager,
    ldda_manager: LDDAManager,
    dataset_id,
    model_class='HistoryDatasetAssociation'
):
    ...
```

## Related Code Paths

When reviewing, reference these Galaxy codebase locations for patterns:

- `lib/galaxy/di/` - DI container setup
- `lib/galaxy/structured_app.py` - `StructuredApp` interface definition
- `lib/galaxy/app.py` - `UniverseApplication` implementation
- `lib/galaxy/managers/` - Manager implementations using DI
- `lib/galaxy/webapps/galaxy/api/` - FastAPI controllers using DI
- `lib/galaxy/celery/tasks.py` - Celery tasks using DI

## Review Checklist

For each file/change, check:

- [ ] **No `app` access for dependencies** - Does code access `app.some_manager` instead of injecting?
- [ ] **Type annotations present** - Are constructor parameters properly type-annotated?
- [ ] **Interface types used** - Uses `StructuredApp` instead of `UniverseApplication` where needed?
- [ ] **Galaxy-style `depends()`** - Controllers use `depends(Type)` not FastAPI `Depends()`?
- [ ] **No internal construction** - Components don't construct their own dependencies internally?
- [ ] **DAG structure** - Dependencies form a directed acyclic graph (no cycles)?
- [ ] **@galaxy_task for Celery** - Celery tasks use `@galaxy_task` decorator with type-annotated parameters?

## Output Format

For each issue found, report:
1. **File and line number**
2. **Issue type** (from review criteria above)
3. **Current code snippet**
4. **Recommended fix**
5. **Severity** (high/medium/low based on impact on architecture)

Summarize findings with counts by issue type and overall assessment of DI compliance.
