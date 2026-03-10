## Standard Workflow

1. First think through the problem, read the codebase for relevant files, and write a plan to tasks/todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please every step of the way just give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.
8. DO NOT BE LAZY. NEVER BE LAZY. IF THERE IS A BUG FIND THE ROOT CAUSE AND FIX IT. NO TEMPORARY FIXES. YOU ARE A SENIOR DEVELOPER. NEVER BE LAZY
9. MAKE ALL FIXES AND CODE CHANGES AS SIMPLE AS HUMANLY POSSIBLE. THEY SHOULD ONLY IMPACT NECESSARY CODE RELEVANT TO THE TASK AND NOTHING ELSE. IT SHOULD IMPACT AS LITTLE CODE AS POSSIBLE. YOUR GOAL IS TO NOT INTRODUCE ANY BUGS. IT'S ALL ABOUT SIMPLICITY

## Coding Standards

All Python code in this project must follow these standards. Apply them when writing new code and when modifying existing code.

### 1. Docstrings — Every Function and Method
- Every function/method must have a docstring documenting: what it does, Args, Returns, Raises, and edge cases.

### 2. Type Hints — Every Signature
- All parameters and return types must have type annotations.
- Use `Optional[T]` for nullable params. Avoid `Any` unless truly necessary.

### 3. Single Responsibility
- Each function should do one thing. If it's >20 lines of logic or has multiple verbs in its name, split it.

### 4. Error Handling — Raise Low, Catch High
- Lower-level functions raise specific exceptions; higher-level functions catch and handle them.
- Every external call (file I/O, network, DB) must have error handling.
- No silent failures. No bare `except:` without re-raise or logging.

### 5. No Magic Numbers or Strings
- Extract all hardcoded literals into named constants with self-documenting names.

### 6. AI-Generated Code Vigilance
- Watch for happy-path-only code, leaky abstractions, missing input validation, and undocumented assumptions.

### 7. Contract Alignment
- Function names must accurately describe behavior. Variable names must be self-documenting.
- Remove dead code and commented-out code.

### 8. Linting and Formatting
- All code must pass `ruff check` and `ruff format`.
- Run ruff after every code change before considering the task complete.

### 9. Commits to Git
- Never add Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com> to any commit
