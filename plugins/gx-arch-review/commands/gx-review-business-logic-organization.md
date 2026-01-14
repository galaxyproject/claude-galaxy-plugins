# Review Galaxy Code for Controller/Service/Manager Best Practices

Perform a Python code review on the provided code. Accept input in any of the following forms:
1. A working directory path (analyze git diff in that directory)
2. A Git commit reference (analyze changes in that commit)
3. A list of Python file paths (analyze those files)
4. A planning document (analyze the Python files in the plan)

## Review Criteria

Review the code and ensure it follows Galaxy best practices for what code belongs in controllers, services, and managers:

1. **Controllers should be thin** - Minimal logic, just request/response handling
2. **Services handle API processing details** - Shield application logic from FastAPI internals
3. **Managers contain business logic** - All non-trivial operations should live here
4. **Models mediate database interactions** - Isolate backend from database internals

## Galaxy Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│           API Controllers               │  ← Request/Response handling
│     (lib/galaxy/webapps/galaxy/api/)    │
├─────────────────────────────────────────┤
│              Services                   │  ← API processing details (thin layer)
│       (lib/galaxy/webapps/galaxy/       │
│            services/)                   │
├─────────────────────────────────────────┤
│              Managers                   │  ← Business logic
│         (lib/galaxy/managers/)          │
├─────────────────────────────────────────┤
│               Models                    │  ← Database interactions (SQLAlchemy ORM)
│          (lib/galaxy/model/)            │
├─────────────────────────────────────────┤
│             Database                    │
└─────────────────────────────────────────┘
```

## Layer Responsibilities

### Controllers (API Layer)
- Parse request parameters
- Call service/manager methods
- Format responses
- Handle HTTP-specific concerns

**Anti-pattern:**
```python
# BAD - business logic in controller
@router.post("/api/histories")
def create_history(payload: CreateHistoryPayload):
    history = model.History()
    history.name = payload.name
    history.user = trans.user
    session.add(history)
    session.commit()
    # Lots of validation, side effects...
    return history
```

**Correct pattern:**
```python
# GOOD - thin controller delegating to manager
@router.post("/api/histories")
def create_history(self, trans: ProvidesHistoryContext, payload: CreateHistoryPayload):
    return self.manager.create(trans, payload)
```

### Services (Optional Thin Layer)
- Handle high-level API and web processing details
- Shield application logic from FastAPI internals
- Often skipped - controllers can talk directly to managers

**Note:** In practice, it's fine to skip this layer. However, there are many places where controller or service layers are thicker than they should be - these are anti-patterns.

### Managers (Business Logic Layer)
- High-level business logic
- Coordinate between multiple models/services
- Controllers should be thin wrappers around manager actions
- Operations requiring more than just database access belong here

**Key manager files:**
- `lib/galaxy/managers/histories.py` - History operations
- `lib/galaxy/managers/hdas.py` - HDA operations
- `lib/galaxy/managers/users.py` - User operations
- `lib/galaxy/managers/workflows.py` - Workflow operations
- `lib/galaxy/managers/collections.py` - Collection operations

**Manager helpers:**
- `lib/galaxy/managers/base.py` - Base manager classes
- `lib/galaxy/managers/context.py` - Transaction context
- `lib/galaxy/managers/deletable.py` - Deletion patterns
- `lib/galaxy/managers/sharable.py` - Sharing patterns

### Models (Database Layer)
- Database interactions via SQLAlchemy ORM
- Classes mapped to database tables
- Think in terms of "objects" not "rows"
- Model classes in `lib/galaxy/model/__init__.py`

**Model rules:**
- Models should only contain database-related logic
- Complex operations go in managers, not models
- Use declarative mapping for table definitions

## Code Patterns to Review

### What Belongs in Controllers
- ✅ Route definitions
- ✅ Request parameter extraction
- ✅ Response formatting
- ✅ HTTP status codes
- ❌ Database queries
- ❌ Business validation
- ❌ Complex conditional logic
- ❌ Multi-step operations

### What Belongs in Managers
- ✅ Business logic
- ✅ Validation rules
- ✅ Coordinating multiple models
- ✅ Permission checks
- ✅ Side effects (notifications, audit logs)
- ❌ HTTP-specific handling
- ❌ Request/response formatting

### What Belongs in Models
- ✅ Column definitions
- ✅ Relationships
- ✅ Simple properties derived from columns
- ✅ Basic serialization
- ❌ Complex business logic
- ❌ Cross-model coordination
- ❌ External service calls

## Related Code Paths

When reviewing, reference these Galaxy codebase locations:

- `lib/galaxy/webapps/galaxy/api/` - FastAPI controllers
- `lib/galaxy/webapps/galaxy/services/` - Service layer
- `lib/galaxy/managers/` - Manager implementations
- `lib/galaxy/model/__init__.py` - Model definitions
- `lib/galaxy/model/migrations/` - Alembic migrations

## Review Checklist

For each file/change, check:

- [ ] **Controller thickness** - Is controller doing more than request/response handling?
- [ ] **Business logic location** - Is business logic in managers, not controllers/models?
- [ ] **Database access** - Are direct database operations in models/managers, not controllers?
- [ ] **Manager usage** - Does controller delegate to appropriate manager?
- [ ] **Model purity** - Do models avoid complex business logic?
- [ ] **Layer violations** - Does code respect layer boundaries?
- [ ] **Service layer (if present)** - Is it thin and focused on API concerns?

## Common Anti-patterns

1. **Fat Controllers**
   - Controllers with database queries
   - Controllers with complex validation
   - Controllers with multi-step operations

2. **Anemic Managers**
   - Managers that just pass through to models
   - Business logic scattered in controllers

3. **Smart Models**
   - Models with business logic beyond data access
   - Models coordinating external services
   - Models with complex validation rules

4. **Missing Manager Layer**
   - Direct controller-to-model access for complex operations

## Output Format

For each issue found, report:
1. **File and line number**
2. **Issue type** (from review criteria above)
3. **Current code snippet**
4. **Recommended fix**
5. **Severity** (high/medium/low based on impact on architecture)

Summarize findings with counts by issue type and overall assessment of layer compliance.
