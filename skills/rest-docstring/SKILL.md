---
name: rest-docstring
description: Enforce reStructuredText (reST) docstring format in Python code. Use this skill whenever the user asks to write, add, fix, review, audit, generate, or update Python docstrings. Also trigger when the user asks to document a function, class, module, or method, or when writing any new Python code that will need docstrings. Trigger even if the user just says "document this" or "add docs", assume reST format unless told otherwise.
---

# reST Docstring Skill

Produce and enforce reStructuredText docstrings in all Python code. Never use Google style. Never use NumPy style. No exceptions unless the user explicitly overrides.

---

## Format Rules

### Functions and Methods

```python
def fn(param: type) -> type:
    """One-line summary in imperative mood. No period if it fits one line.

    Optional extended description. Use when the one-liner is insufficient
    to convey behavior, side effects, or important constraints.

    :param param: What this parameter represents and any constraints.
    :type param: int
    :returns: What the return value represents (not "Returns X", just describe X).
    :rtype: str
    :raises ValueError: When and why this is raised.
    :raises KeyError: When and why this is raised.
    """
```

### Classes

```python
class MyClass:
    """One-line summary of the class purpose.

    Extended description if needed. Do NOT document __init__ params here.
    Params go in __init__.
    """

    def __init__(self, value: int) -> None:
        """Initialize MyClass.

        :param value: The initial value to store.
        :type value: int
        """
```

### Properties

```python
@property
def name(self) -> str:
    """The user's display name."""
```

No `:param:`, no `:returns:` for properties. One-liner only.

### Modules

```python
"""Short description of what this module does.

Extended description if the module has non-obvious purpose, important
usage notes, or configuration requirements.
"""
```

### Dunder Methods

One-liner minimum. No field blocks needed unless behavior is genuinely non-obvious.

```python
def __repr__(self) -> str:
    """Return unambiguous string representation."""
```

### Private Methods (leading underscore)

One-liner minimum. Full field blocks if the method is complex.

---

## Field Reference

| Field                    | Usage                                                 |
| ------------------------ | ----------------------------------------------------- |
| `:param name:`           | One per parameter. Describe what it is, not its type. |
| `:type name:`            | One per parameter. Python type as a string.           |
| `:returns:`              | Describe what the value represents, not "Returns X".  |
| `:rtype:`                | Python type of the return value.                      |
| `:raises ExceptionType:` | One per exception. Explain when/why it's raised.      |

---

## Rules

1. **Never omit docstrings** on public functions, classes, or modules.
2. **Never repeat type info** in the description if `:type:` already captures it.
3. **Never write "Returns X"** in `:returns:`. Just describe what X is.
4. **Always use imperative mood** in the one-line summary: "Calculate", not "Calculates".
5. **Blank line** between summary and extended description. Blank line before fields block.
6. **No trailing period** on single-line summaries. Period on multi-sentence extended descriptions.
7. **Order of fields**: `:param:` and `:type:` pairs together, then `:returns:` and `:rtype:`, then `:raises:`.
8. **Use only plain ASCII characters**. No em dashes (--), en dashes, smart/curly quotes, ellipsis characters, or any other non-keyboard character. Use only what a person would type on a standard keyboard: hyphens (-), straight quotes ("), and regular periods.

---

## When Auditing Existing Code

When asked to audit or fix docstrings in existing code:

1. Identify all public functions, classes, and modules missing docstrings and list them.
2. Identify existing docstrings using Google or NumPy style and convert them.
3. Identify incomplete docstrings (missing params, returns, raises) and complete them.
4. Report a summary: N missing, N converted, N completed.

Do not silently skip any public symbol.

---

## Common Mistakes to Avoid

- Writing `:returns: Returns the user object` and drop "Returns"
- Putting class params in the class docstring instead of `__init__`
- Omitting `:type:` fields when `:param:` is present
- Using `Args:` / `Returns:` / `Raises:` headers (Google style)
- Using `Parameters` / `----------` sections (NumPy style)
- Skipping `:raises:` when the function clearly raises exceptions
- Using em dashes (--), curly quotes, or other non-ASCII characters that a person would not type directly on a keyboard
