# agent-skills

A collection of skills for Coding agents. These are modular instruction sets that extend what Agent can do.

## What's a skill?

A skill is a folder with a `SKILL.md` at its root. When Coding Agent has access to a skill, it reads the instructions at task time and follows them. Skills can include:

- **`SKILL.md`** (required): YAML frontmatter (`name`, `description`) plus markdown instructions.
- **`scripts/`**: Executable code for deterministic or repetitive steps.
- **`references/`**: Documentation loaded into context as needed.
- **`assets/`**: Templates, fonts, or other files used in output.

Skills load progressively. The `description` field is always in context (~100 words). The full `SKILL.md` body loads when the skill triggers. Supporting files load only when needed.

## Structure

```
skill-name/
├── SKILL.md
├── scripts/
├── references/
└── assets/
```

## Skills

| Skill                                | Description                                                     |
| ------------------------------------ | --------------------------------------------------------------- |
| [`rest-docstring`](./skills/rest-docstring/SKILL.md) | Enforce reStructuredText (reST) docstring format in Python code |

## Writing a skill

1. Create a folder with the skill name.
2. Add `SKILL.md` with frontmatter:

```yaml
---
name: your-skill-name
description: What this skill does and when to use it.
---
```

3. Write the instructions in the body. Keep it under 500 lines. If it needs to be longer, split content into reference files and point to them from `SKILL.md`.
4. Add scripts or reference files if the skill needs them.

The `description` field drives triggering. It should clearly state both what the skill does and the specific contexts where Agent should use it.

## Installing skills

Install skills via [skills.sh](https://skills.sh/):

```bash
npx skills add https://github.com/sivarajakani/agent-skills --skill <skill-name>
```

## Contributing

Skills should be self-contained and narrowly scoped. One skill, one purpose. If a skill requires another skill to function, document that dependency in the frontmatter.

PRs welcome. Please include at least one example prompt that reliably triggers the skill.

## License

MIT
