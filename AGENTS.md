# AGENTS.md

## Important Notes

- ALWAYS review this file before making any changes to the codebase, no matter how small the change
- The venv used by this project, if any, should be activated when making changes

## Coding Guidelines

### Design Considerations

- When first implementing something, be minimalist. "As simple as possible, but no simpler." Do not add bells or
  whistles like --dry-run or --verbose or --verify-output unless specifically asked.

### Code Style

#### Errors

- Strongly prefer throwing errors rather than defensively programming against unexpected values, unexpected state, or unexpected inputs. For example, if a parsed JSON value should be an int, or at least convertible to an int, simply attempt the cast. Don't check to see if it's a valid int; Python will do that for us and throw an error as appropriate. Likewise, if a field is expected to be present, use `foo["bar"]` instead of `foo.get("bar")`. Fail fast so that somebody higher up the call stack has the opportunity to handle the problem. If the problem isn't handled by anyone up the call stack, the program should simply crash.
- Strongly prefer to throw, rather than suppress, unexpected or irrecoverable errors. Let it bubble up so that callers have the opportunity (though not the responsibility) to handle them. If the program crashes because nobody handled the error, that's often correct behavior. It's much easier to debug a crash due to unexpected or irrecoverable conditions than to debug a program pretending everything's ok when in fact things have gone awry.
- Do not wrap and rethrow errors unless there is critical context to be added. Instead, just throw the original error.
  If it's a specific, known error, consider making mention of it in the function's docstring.
- Highly readable lines of code must not be commented. Comments should exist only to explain opaque logic, missing business/operational context, or code with middling to low readability. If there's a `json.loads` call, for example, you should absolutely not comment that you're loading a JSON string. Likewise, there should absolutely not be a comment `# send result to user` for an invocation of `send_to_user`.
- Avoid using None to represent an error condition. For example, bad input params, missing files, missing fields in files, disk OOM, and other such problems should almost always result in a thrown error, not a returned None. Generally, only return None when None is a valid value.
- Avoid returning True/False for success/failure. Throw errors.

#### Everything Else

- Strongly favor immutability. Avoid mutation.
- Avoid using Any when at all possible.
- Log messages should be no more than one sentence long. The first letter should not be capitalized unless it's an
  acronym or proper noun. The message should not end with a period.
- Avoid empty lines of whitespace unless required by PEP / Ruff / Black style requirements.
- Use Ruff formatter (Black-compatible) with 120 character line length
- Use mypy to enforce types
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
