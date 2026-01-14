Perform a Python code review on the provided code. Accept input in any of following forms:
1. A working directory path (analyze git diff in that directory)
2. A Git commit reference (analyze changes in that commit)
3. A list of Python file paths (analyze those files)
4. A planning document (analyze the Python files in the plan)    

Review the code focusing on these two criteria:

**1. Type Annotations**
- Methods and functions should have type annotations on parameters and return types
- Don't over-type: basic operations and obvious cases don't need extensive annotations
- Flag missing annotations on public methods and functions with non-obvious signatures
- Local helper functions and simple internal methods may not need annotations if context is clear
- Annotate complex parameter types (dicts, lists of objects, unions)

**2. Import Organization**
- All imports must be at the top of the file (after module docstring if present)
- Inline imports (imports appearing mid-file) should be moved to the top UNLESS they have an inline comment explaining why (e.g., # circular import, # lazy load, # conditional)
- Flag any inline imports without explanation and move them to the top
- Don't reorganize import groups - isort handles that

**Output Format:**
For each file reviewed, provide:
- **File**: filename
- **Typing Issues**: List missing or problematic type annotations (or "None" if clear)
- **Import Issues**: List inline imports found and moved (or "None" if correct)
- **Summary**: Brief assessment of the file's adherence to standards

At the end, provide:
- **Overall Assessment**: How many files pass review / total files
- **Key Recommendations**: Top 3-5 items to address across all files
