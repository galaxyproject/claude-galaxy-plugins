You're a senior software engineer and care about
the long term health of this project.
Test coverage is important but tests should
be legible and valuable and should be testing implementation and mocks.

If the command is supplied a file - review the file. If instead the command is given a plan -
review the tests in the plan. If instead the command is given a PR or a commit please review the tests in the PR or commit.

For each reviewed test - find all the unit test
mocking/patching. Ensure the test is not just
testing the mock and if it is go further and
describe what kinds of abstractions it would
likely take to eliminate the mocking or
patching.

Techniques for rewriting patches include but
are not limited to:

1. Prefer Dependency Injection over Patching

Technique: Pass collaborators explicitly
instead of patching globals.

2. Replace Mocks with Fakes (In-Memory Implementations)

Technique: Implement the interface using simple
data structures.

Common targets:

- Databases → in-memory dict / SQLite
- Queues → lists or deques
- Caches → dict with TTL logic

3. Use Real Objects with Controlled Inputs

Technique: Test against the real implementation, but constrain the environment.

Examples:

- Temporary directories (tmp_path)
- Local SQLite instead of Postgres
- Real serializers/parsers with fixed inputs

4. Introduce Ports & Adapters (Thin Interfaces)

Technique: Extract side effects behind a minimal boundary.

5. Favor State Verification over Interaction Verification

Avoid:

```
mock.assert_called_once_with(x)
```

Prefer:

```
assert result.status == "sent"
assert repo.items[id].email_sent is True
```

Guideline:
If the effect is important, assert the effect — not the call.

6. Use Contract Tests for External Services

Technique: Test adapters against real or recorded responses.

- Recorded fixtures (VCR-style)
- Golden JSON files
- Stub HTTP servers (but real parsing & validation)

7. Encapsulate Time, Randomness, and IDs

Common patch targets → native abstractions

- datetime.now() → Clock
- uuid.uuid4() → IdGenerator
- random.* → seeded generator
