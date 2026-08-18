# Contributing to Awesome AI Agents

Thank you for your interest in contributing! This project is a community resource, and we welcome improvements.

感谢你的贡献兴趣！本项目是社区资源，我们欢迎改进。

## How to Contribute / 如何贡献

### Adding a New Skill / 添加新技能

1. Fork this repository
2. Create a new `.md` file in the appropriate `skills/` subdirectory
3. Follow the [skill template](#skill-template) below
4. Submit a Pull Request

### Improving Existing Skills / 改进现有技能

1. Fork this repository
2. Make your changes
3. Submit a Pull Request with a clear description of what you changed and why

### Reporting Issues / 报告问题

1. Use the [Issue Tracker](https://github.com/liangzhengtao/awesome-ai-agents/issues)
2. Use the appropriate issue template
3. Be specific about the problem

## Skill Template / 技能模板

Every skill file must follow this structure:

```markdown
# Skill Title

> One-line description of what this skill teaches.

## When to Use

| Scenario | Use this skill? |
|----------|----------------|
| ... | Yes/No/Alternative |

## Architecture

(ASCII diagram of the system design)

## Code Template: Name

(Python code with comments)

## Patterns

### Pattern: Name
(Description + code)

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| ... | ... | ... |

## Proven Patterns

1. ...
2. ...

## Dependencies

\```bash
pip install ...
\```

## See Also

- [Related Skill](./path.md) — Why it's related
```

## Guidelines / 指南

- **Code must be runnable** — use real APIs, not pseudocode. Mock where needed.
- **Code must have comments** — explain non-obvious decisions.
- **Use Python 3.11+** — type hints, f-strings, modern stdlib.
- **Be bilingual** — title and key sections in English and Chinese where practical.
- **Be specific** — "use asyncio" is not helpful. Show the code.
- **No advertisements** — skills should be framework-agnostic. Mention frameworks as options, not endorsements.

## Code of Conduct / 行为准则

Please read our [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

请在贡献前阅读我们的[行为准则](CODE_OF_CONDUCT.md)。

## Questions? / 有问题？

Open a [Discussion](https://github.com/liangzhengtao/awesome-ai-agents/discussions) for general questions.
