# Code Review Skill

Review Python code for quality, readability, and AI-assisted development best practices.
Identifies refactoring opportunities, missing docstrings, functions doing too much,
unhandled error paths, and hardcoded magic values. Presents findings clearly and
asks the user before implementing any changes.

## Usage
```
/code-review <file_path_or_glob_pattern>
```

## Examples
```
/code-review app/main.py
/code-review app/services/classification_service_enhanced.py
/code-review app/services/
```

## Parameters
- `file_path_or_glob_pattern`: Path to a Python file, or a directory/glob pattern to review multiple files

---

## Frontend Contract Protection

When reviewing code that serves an existing frontend UI or is part of a running
backend, the following constraints are **absolute and non-negotiable**. No change
may be implemented — and no change may even be suggested for immediate implementation —
if it would break the existing frontend without a corresponding frontend update.

### What Must Never Change Without Frontend Coordination

- **Endpoint signatures**: route paths, HTTP methods, URL parameters, query parameters
- **Request contracts**: field names, types, and structure of request bodies
- **Response contracts**: field names, types, nesting structure, and shape of all
  response payloads — including error response shapes
- **Return types of any function called directly by a route handler**
- **Status codes**: do not change which HTTP status codes are returned for which conditions
- **Variable or parameter names that map to request/response fields**

### Classifying Every Finding

During the review, every finding must be classified with one of two tags:

- **[SAFE]** — This change is internal only. It does not affect any endpoint,
  request shape, response shape, return type, or anything the frontend depends on.
  It can be implemented immediately upon user approval.

- **[FRONTEND RISK]** — This change touches something the frontend depends on.
  It cannot be implemented without coordinating a frontend update.
  It must be documented in the deferred changes file instead of implemented.

### The Deferred Changes File

At the end of every review of backend/API code, create or append to a file named
`deferred_frontend_changes.md` in the same directory as the reviewed file.

Each deferred entry must include:

```
## [Finding ID] — function_name(), filename.py line N
Category:    [Docstring | Single Responsibility | Error Handling | Magic Values | ...]
Why Deferred: Describe exactly what frontend contract would be affected.
What to Change: Full description of the desired change.
Frontend Work Required: What the frontend team would need to update to make this safe.
```

### How to Ask the User

At the start of every review, ask:

> "Is this code connected to a running frontend UI or live API?
> If yes, I will classify every finding as SAFE or FRONTEND RISK,
> implement only SAFE changes upon your approval, and document
> all FRONTEND RISK changes to a deferred_frontend_changes.md file."

If the user confirms the code is connected to a frontend:
- Tag every single finding as [SAFE] or [FRONTEND RISK] in the findings output
- Show SAFE and FRONTEND RISK counts in the summary table
- Only offer to implement [SAFE] findings
- Automatically write all [FRONTEND RISK] findings to the deferred file
- Never implement a [FRONTEND RISK] finding under any circumstances,
  even if the user explicitly asks — explain why and redirect to the deferred file

If the user says the code is standalone (no frontend dependency):
- Skip the SAFE/FRONTEND RISK classification
- Proceed with the standard review workflow

---

## Review Standards

Apply all seven categories on every review. Every finding must include:
- The file name and line number(s)
- A plain-English description of the problem
- A concrete suggestion for how to fix it

### 1. Docstrings — Every Function and Method

Every function and method must have a docstring that documents:
- **What it does** (one-sentence summary)
- **Args**: each parameter, its type, and what it represents
- **Returns**: what is returned and its type
- **Raises**: every exception the function can raise and under what condition
- **Edge cases**: any non-obvious behavior (empty inputs, None values, boundary conditions)

Flag any function or method missing a docstring, or with a docstring that omits
Args, Returns, Raises, or edge case notes where applicable.

### 2. Single Responsibility — One Thing Per Function

Each function or method should do exactly one thing. Signs it does too much:
- More than ~20 lines of logic (not counting docstring/comments)
- The function name uses "and", "or", or contains multiple verbs
- Multiple distinct steps that could each be named independently
- Mixed abstraction levels (e.g., business logic mixed with I/O or formatting)

When flagging, suggest how to split the function and what each new function should
be named.

### 3. Error Handling — Full Coverage of Failure Paths

Apply the **raise-low, catch-high** pattern:
- Lower-level functions raise specific, descriptive exceptions — they do not swallow errors
- Higher-level orchestration functions catch exceptions and decide what to do
- Every external call (file I/O, network, database, subprocess) must have error handling
- `except Exception` or bare `except:` blocks must include a re-raise or explicit logging
- No silent failures — code that catches an error and does nothing is always flagged

Flag:
- Missing try/except around external operations
- Broad exception catches without re-raise or logging
- Functions that return `None` to signal errors instead of raising
- Error messages that lack context (e.g., `raise ValueError("invalid")` with no detail)

### 4. No Magic Numbers or Strings

Every hardcoded literal embedded in logic must be extracted to a named constant.

Flag any:
- Raw integers in conditions, slices, or calculations (e.g., `if status == 3`, `time.sleep(30)`)
- Raw strings used as keys, status values, or configuration (e.g., `if type == "admin"`)
- Threshold or limit values with no name explaining their meaning
- Duplicated literals that should share a single named source

Suggest a name for each constant that makes its purpose self-documenting.

### 5. AI-Generated Code Patterns to Watch For

AI code generation frequently introduces specific problems. Flag these explicitly:

- **Happy-path only**: Functions that handle success but not failure
- **Leaky abstractions**: Functions that do too much and expose internal details
- **Inline construction**: Objects or configs built inline instead of via factory functions
- **Tightly coupled reactions**: One function directly calling many downstream handlers
  instead of using an event/callback pattern
- **Missing input validation**: No precondition checks at function entry
- **Undocumented assumptions**: Logic that only works under specific conditions
  with no comment or assertion explaining them

### 6. Type Hints — Every Function Signature and Return

Every function and method must have complete type hints:
- **All parameters** must have type annotations (e.g., `def foo(x: int, name: str)`)
- **Return types** must be annotated (e.g., `-> dict`, `-> List[str]`, `-> None`)
- **Class attributes** should have type annotations where not already provided by Pydantic/BaseModel
- Use `Optional[T]` for parameters that can be None
- Use specific types from `typing` (List, Dict, Tuple, etc.) over bare `list`, `dict` for Python 3.8 compatibility, or use modern syntax (`list[str]`) if the project targets Python 3.9+
- Avoid `Any` unless truly necessary — prefer specific types

Flag any function or method that:
- Has untyped parameters (missing `: type` annotation)
- Has no return type annotation (missing `-> type`)
- Uses `Any` where a more specific type is possible
- Has inconsistent types between the annotation and actual usage

When flagging, provide the complete corrected signature.

### 7. Specification & Contract Alignment

When reviewing, also check:
- Does the function name accurately describe what it does?
- Are variable names self-documenting?
- Are there `assert` statements or guard clauses for preconditions where appropriate?
- Is there dead code or commented-out code that should be removed?

---

## Review Workflow

### Step 1 — Read the File(s)

Read every file provided. Do not begin outputting findings until you have read all files.

### Step 2 — Build the Finding List

For each finding, record:
```
CATEGORY:     [Docstring | Single Responsibility | Error Handling | Magic Values |
               AI Patterns | Type Hints | Contract Alignment]
LOCATION:     filename.py, line(s) N-M
FUNCTION:     function_or_method_name()
SEVERITY:     [High | Medium | Low]
IMPACT:       [SAFE | FRONTEND RISK] — only when reviewing backend/API code
PROBLEM:      Plain-English description of the issue
SUGGESTION:   Concrete fix — what to add, rename, extract, or restructure
```

When IMPACT is FRONTEND RISK, add one additional field:
```
FRONTEND CONTRACT: Describe exactly which endpoint, response field, return type,
                   or request shape would be affected by this change.
```

**Severity guidelines:**
- **High**: Missing error handling on external calls, silent failures, functions with 3+ responsibilities
- **Medium**: Missing docstrings, magic numbers, leaky abstractions, missing input validation
- **Low**: Naming improvements, minor single-responsibility splits, dead code

### Step 3 — Present Findings

Present findings grouped by category, ordered High -> Medium -> Low within each group.

Use this format for each finding:

```
-- [CATEGORY] -- filename.py, line N  ----------------------------------------
Function: function_name()
Severity: High / Medium / Low
Impact:   SAFE  /  FRONTEND RISK — [brief reason]
Problem:  Clear description of what is wrong.
Fix:      Concrete description of what to change.
```

After all findings, show a summary table:

```
+-------------------------+-------+--------+-----+--------+-----------------+
| Category                | High  | Medium | Low | Total  | Frontend Risk   |
+-------------------------+-------+--------+-----+--------+-----------------+
| Docstrings              |   0   |   3    |  1  |   4    |       0         |
| Single Responsibility   |   1   |   2    |  0  |   3    |       1         |
| Error Handling          |   2   |   1    |  0  |   3    |       0         |
| Magic Values            |   0   |   4    |  0  |   4    |       0         |
| AI Patterns             |   1   |   1    |  1  |   3    |       1         |
| Type Hints              |   0   |   3    |  1  |   4    |       0         |
| Contract Alignment      |   0   |   0    |  2  |   2    |       1         |
+-------------------------+-------+--------+-----+--------+-----------------+
| TOTAL                   |   4   |  14    |  5  |  23    |       3         |
+-------------------------+-------+--------+-----+--------+-----------------+
```

### Step 4 — Ask the User What to Implement

After presenting findings, always ask:

> "Summary: X total findings (Y SAFE, Z FRONTEND RISK).
> FRONTEND RISK findings will be documented to deferred_frontend_changes.md for later coordination.
> Would you like me to implement the SAFE changes, a specific category, or specific findings?
> I can also explain any finding in more detail before making changes."

Present the user with these options:
1. Implement all SAFE findings
2. Implement SAFE high severity only
3. Implement a specific category (SAFE findings only — ask which one)
4. Implement specific finding numbers (SAFE findings only — ask which ones)
5. Explain a finding in more detail (ask which one)
6. No changes — review only

**Do not implement any changes until the user selects an option and confirms.**
**Never offer FRONTEND RISK findings as implementation options.**
**If the user asks to implement a FRONTEND RISK finding, decline and explain that
it requires frontend coordination. Point them to the deferred_frontend_changes.md file.**

### Step 5 — Implement Approved Changes

For each approved SAFE change:
1. Show a brief before/after diff or description of what will change
2. Apply the change to the actual file
3. Confirm the change was made

For FRONTEND RISK findings:
1. Write each one to `deferred_frontend_changes.md` using the format defined in
   the Frontend Contract Protection section
2. Confirm the file was written
3. Do not touch the source file

After all changes, summarize:
- What was implemented (SAFE changes applied)
- What was deferred (FRONTEND RISK findings documented)
- Flag anything that may need manual review (e.g., function splits where callers
  in other files may need updating)

### Step 6 — Run Ruff

After implementing any changes, run `ruff check` and `ruff format` on all modified files.
Fix any issues before presenting the final summary.

---

## Communication Style

- Be direct and specific — no vague observations
- Reference line numbers and function names in every finding
- When suggesting a split, provide the proposed new function names
- When suggesting a constant, provide the proposed constant name and value
- Do not lecture — findings should be actionable, not educational
- If the code is well-written in a category, say so briefly ("No magic values found")
- Match technical depth to the complexity of the code being reviewed

---

## Example Finding

```
-- Error Handling -- gemini_token_tracker.py, line 87  ------------------------
Function: track_gemini_call()
Severity: High
Impact:   SAFE
Problem:  The requests.post() call has no try/except. A network timeout,
          DNS failure, or non-JSON response will raise an unhandled exception
          that propagates with no context about which API call failed.
Fix:      Wrap the requests.post() and response.json() calls in a try/except
          that catches requests.RequestException and json.JSONDecodeError,
          then raises a descriptive GeminiAPIError with the model name and
          endpoint included in the message.
```

---

## What This Skill Does Not Do

- Does not review for correctness of business logic
- Does not suggest architectural changes beyond what is visible in the provided files
- Does not run or execute the code
- Does not make changes without user approval
- Does not implement any change that would break an existing frontend — these are always documented to the deferred file instead
