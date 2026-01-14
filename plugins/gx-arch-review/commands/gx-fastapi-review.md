Perform a Python code review on the provided code. Accept input in any of following forms:
1. A working directory path (analyze git diff in that directory)
2. A Git commit reference (analyze changes in that commit)
3. A list of Python file paths (analyze those files)
4. A planning document (analyze the Python files in the plan)    

Ensure the FastAPI layer follows Galaxy best practices (only one needs to be checked currently):

- If there is a Python FastAPI layer, ensure we're not trying to ``include_router`` or register the router somehow like would typically be needed in FastAPI. We detect our routers automatically.

Galaxy has other best practices around keeping this layer thin and using type-based DI but other
review commands can be used to check these.
