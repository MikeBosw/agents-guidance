# AGENTS.md

## Important Notes

- ALWAYS review this file before making any changes to the codebase, no matter how small the change
- The venv used by this project, if any, should be activated when making changes
- After making changes, run `git add` on the changed files, then `pre-commit run` to make sure types are checked, etc.

## Coding Guidelines

### Design Considerations

- When first implementing something, be minimalist. "As simple as possible, but no simpler." Do not add bells or
  whistles like --dry-run or --verbose or --verify-output unless specifically asked.
- Minimize state management and statefulness. If a solution exists that avoids maintaining state between function
  calls, prefer that solution. Especially for async or parallel code.
- Strongly favor immutability by default:
  - Never re-assign a value to a variable after its initial assignment. Find other solutions.
  - Favor list comprehensions, map/reduce, and functional programming.
  - Plain dataclasses should always have frozen=True when possible, and set the equivalent Config for Pydantic models.

### Code Style

#### Errors

- Strongly prefer to throw, rather than suppress, unexpected or locally irrecoverable errors. Let it bubble up so that
  callers have the opportunity (though not the responsibility) to handle them. If the program crashes because nobody
  handled the error, that's often correct behavior. It's much easier to debug a crash due to unexpected or irrecoverable
  conditions than to debug a program pretending everything's ok when in fact things have gone awry.
- Strongly prefer throwing errors over defensively programming against unexpected values, unexpected state, or
  unexpected inputs. For example, if a parsed JSON value should be an int, or at least convertible to an int, simply
  attempt the cast. Don't check to see if it's a valid int; Python will do that for us and throw an error as
  appropriate. Likewise, if a field is always expected to be present, use `foo["bar"]` instead of `foo.get("bar")`.
- Do not wrap and rethrow errors unless there is genuinely useful, local context to be added. Instead, just throw the
  original error. If it's a specific, known error, consider making mention of it in the function's docstring.
- Avoid using None to represent an error condition. For example, bad input params, missing files, missing fields in
  files, disk OOM, and other such problems should almost always result in a thrown error, not a returned None.
  Generally, only return None when None is a valid value.
- Avoid returning True/False for success/failure. Throw errors.

#### Everything Else

- Highly readable lines of code must not be commented. Comments should exist only to explain opaque logic, missing
  business/operational context, or code with middling to low readability. If there's a `json.loads` call, for example,
  you should absolutely not comment that you're loading a JSON string. Likewise, there should absolutely not be a
  comment `# send result to user` for an invocation of `send_to_user`.
- On the flip side, ALL pyright / mypy / tsc type errors must be commented. You should either explain why the type 
  system is sufficiently deficient to justify an override, or else fix the type error.
- Avoid using Any when at all possible.
- Log messages should be no more than one sentence long. The first letter should not be capitalized unless it's an
  acronym or proper noun. The message should not end with a period.
- Avoid empty lines of whitespace unless required by PEP / Ruff / Black style requirements.
- Pass the pre-commit hooks or modify and re-commit
- Minimize the scope of try blocks. Prefer to do only the exception-prone logic within the try block, rather than all
  the before and after logic.
- When possible, use native types (list, dict) for type hints rather than importing from typing (List, Dict, etc).
- Minimize both indentation and loss of context by putting abortive conditions into standalone if-then-return statements
  at the top of the function or loop. For example, this is an anti-pattern:
  ```python
    if conversations:
      print("\nFirst conversation details:")
      # ... a whole bunch of logic
    else:
      print("No conversations found")
  ```
  Instead, do this:
  ```python
    if not conversations:
      print("No conversations found")
      return  # or break, or continue
    print("\nFirst conversation details:")
    # ... a whole bunch of logic
  ```
- The first line of a Git commit message should begin with a lowercase letter (unless it's an acronym or proper noun),
  be concise, and end without a period
- Function docs use the present tense in the indicative voice and skip the subject, e.g., "Creates a new user."
- Class docs typically begin with "A" or "The" followed by a noun phrase describing what the entity is, followed by (if
  more than just a simple data object) what the entity does.
- Minimize the scope of variables to the smallest possible context. If something is only used within the else clause of
  an if-then-else, for example, declare it in the else clause, not before the if.
- Favor named enums over raw ints or strings
- Extrapolate equivalent guidance, based on the above, to other languages

## Attribution

https://raw.githubusercontent.com/MikeBosw/agents-guidance/refs/heads/main/AGENTS.md