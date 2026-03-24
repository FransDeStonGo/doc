---
type: reference
tags: [skills, slash-commands, claude-code, examples, commit, review, refactor, deploy, report]
status: complete
updated: 2026-03-24
related:
  - skills/skills.md
  - skills/anatomy.md
  - agents/multi-agent.md
---

# Skills: примеры

## Overview

Готовые примеры skills для типовых задач разработки. Каждый — рабочий шаблон, адаптируй под свой проект.

---

## Git и разработка

### /commit — умный коммит

```markdown
---
name: commit
description: Create a git commit with a descriptive message
disable-model-invocation: true
allowed-tools: Bash(git *)
---

Create a git commit following these rules:

1. Run `git diff --staged` to see what's staged
2. If nothing staged, run `git status` to show what's available
3. Write a commit message following this format:
   - First line: imperative mood, max 72 chars (e.g., "Add user authentication")
   - Blank line
   - Body: explain WHY, not what (the diff already shows what)
4. Commit with `git commit -m "..."`

If no files are staged, suggest which files to stage based on `git status`.
```

### /review-pr — ревью pull request

```markdown
---
name: review-pr
description: Review a pull request for code quality, bugs, and best practices
disable-model-invocation: true
context: fork
agent: Explore
allowed-tools: Bash(gh *), Read, Grep
---

Review pull request $ARGUMENTS:

## Context
- PR diff: !`gh pr diff $ARGUMENTS`
- PR description: !`gh pr view $ARGUMENTS`
- Changed files: !`gh pr diff $ARGUMENTS --name-only`

## Review checklist

1. **Correctness** — does the code do what it claims?
2. **Edge cases** — what could go wrong?
3. **Security** — any injection, auth, or data exposure issues?
4. **Performance** — any obvious bottlenecks?
5. **Tests** — are changes covered by tests?
6. **Code style** — follows project conventions?

Format the review as:
- Summary (2-3 sentences)
- Issues (blocking / non-blocking)
- Suggestions (optional improvements)
- Approval recommendation
```

### /fix-issue — исправить задачу из трекера

```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
allowed-tools: Bash(gh *), Read, Edit, Grep
---

Fix GitHub issue #$ARGUMENTS:

1. Read the issue: `gh issue view $ARGUMENTS`
2. Understand the requirements
3. Find relevant files with Grep and Read
4. Implement the fix
5. Write or update tests
6. Create a commit referencing the issue: "Fix #$ARGUMENTS: <description>"

Follow existing code style. Don't break existing tests.
```

---

## Анализ и документация

### /explain — объяснить код

```markdown
---
name: explain
description: Explain code with analogies and diagrams. Use when asked "how does this work?" or to understand unfamiliar code
---

Explain $ARGUMENTS:

1. **Analogy** — compare to something from everyday life
2. **ASCII diagram** — show data flow or structure
3. **Step by step** — what happens when this code runs
4. **Gotcha** — common mistake or misconception

Keep it conversational. Adjust detail to apparent complexity.
```

### /docs — сгенерировать документацию

```markdown
---
name: docs
description: Generate or update documentation for code
disable-model-invocation: true
---

Generate documentation for $ARGUMENTS:

1. Read the file or module
2. Identify: purpose, inputs/outputs, edge cases, examples
3. Write documentation following the project's existing style

For Python: Google-style docstrings
For TypeScript: JSDoc
For a module: README.md in the same directory

Don't over-document obvious things. Focus on WHY and edge cases.
```

### /changelog — сгенерировать changelog

```markdown
---
name: changelog
description: Generate a changelog from git history
disable-model-invocation: true
allowed-tools: Bash(git *)
---

Generate a changelog for $ARGUMENTS (version or date range):

Commits: !`git log --oneline $ARGUMENTS`

Group changes by type:
- **Features** (feat:, add, implement)
- **Bug Fixes** (fix:, patch, resolve)
- **Performance** (perf:, optimize)
- **Breaking Changes** (!:, BREAKING)
- **Other** (docs, refactor, chore)

Format as Keep a Changelog (keepachangelog.com).
Exclude: merge commits, version bumps, typo fixes.
```

---

## Рефакторинг

### /refactor — рефакторинг файла

```markdown
---
name: refactor
description: Refactor code for better readability and maintainability
disable-model-invocation: true
---

Refactor $ARGUMENTS:

Rules:
- Keep the same behavior (no functional changes)
- Improve naming (clear, intention-revealing)
- Reduce complexity (extract functions if > 20 lines)
- Remove duplication
- Add types if missing (TypeScript/Python)

Before making changes:
1. Run existing tests to confirm they pass
2. Make changes
3. Run tests again to confirm nothing broke

Show a summary of what you changed and why.
```

### /migrate — миграция между фреймворками

```markdown
---
name: migrate
description: Migrate a component or module from one technology to another
disable-model-invocation: true
---

Migrate $0 from $1 to $2.

Steps:
1. Read the source file thoroughly
2. Understand all functionality (don't miss edge cases)
3. Write the equivalent in $2, following its conventions
4. Check: does the migrated version cover all original functionality?
5. Update imports in other files if needed

Preserve: tests, behavior, error handling, types.
```

---

## Деплой и операции

### /deploy — деплой в окружение

```markdown
---
name: deploy
description: Deploy the application to an environment
disable-model-invocation: true
allowed-tools: Bash
---

Deploy to $ARGUMENTS environment:

1. Check current branch and status: `git status`
2. Confirm all tests pass
3. Build: `npm run build` (or project-specific command)
4. Deploy to $ARGUMENTS:
   - staging: `npm run deploy:staging`
   - production: `npm run deploy:production`
5. Verify deployment: check health endpoint
6. Report: what was deployed, any issues

STOP if tests fail. Ask before deploying to production if anything looks wrong.
```

### /check-health — проверить состояние сервиса

```markdown
---
name: check-health
description: Check the health and status of running services
disable-model-invocation: true
allowed-tools: Bash
---

Check health for $ARGUMENTS (or all services if empty):

1. Check running processes
2. Test endpoints if applicable
3. Check recent error logs (last 50 lines)
4. Check resource usage (CPU, memory)

Report:
- Status (OK / WARNING / ERROR) for each service
- Any recent errors or anomalies
- Recommended actions if issues found
```

---

## Отчёты и исследование

### /report — отчёт о состоянии кодовой базы

```markdown
---
name: report
description: Generate a codebase health report
disable-model-invocation: true
context: fork
agent: Explore
---

Analyze the codebase and generate a health report:

## Areas to analyze

1. **Structure** — directory organization, module boundaries
2. **Dependencies** — outdated packages, security vulnerabilities
3. **Code quality** — complexity hotspots, duplicate code
4. **Test coverage** — what's tested, what's not
5. **Documentation** — what's missing

## Output format

```markdown
# Codebase Health Report — [date]

## Summary
[3-5 bullet points of key findings]

## Strengths
[What's working well]

## Issues
[Problems by severity: Critical / Warning / Info]

## Recommendations
[Prioritized list of improvements]
```

Keep the report actionable. Focus on the most impactful improvements.
```

### /search-docs — поиск в документации

```markdown
---
name: search-docs
description: Search project documentation and README files for information
---

Search documentation for information about $ARGUMENTS:

1. Find all .md files in the project: look in docs/, README*, wiki/
2. Search for relevant sections
3. Synthesize the answer from what you find
4. If not found in docs, suggest where the answer might be (code, tests)

Be precise: quote the relevant documentation, don't paraphrase.
```

---

## Специализированные

### /security-review — аудит безопасности

```markdown
---
name: security-review
description: Review code for security vulnerabilities
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

Security review of $ARGUMENTS:

Check for OWASP Top 10:
1. **Injection** (SQL, command, LDAP) — is user input sanitized?
2. **Broken auth** — session management, token handling
3. **Sensitive data** — is PII/credentials exposed in logs or responses?
4. **XXE** — XML processing vulnerabilities
5. **Broken access control** — missing authorization checks
6. **Security misconfiguration** — defaults, debug mode, headers
7. **XSS** — unescaped user content in HTML
8. **Insecure deserialization**
9. **Known vulnerable dependencies**
10. **Insufficient logging** — are security events logged?

Report: severity (Critical/High/Medium/Low), description, file:line, fix suggestion.
```

### /test-coverage — анализ покрытия тестами

```markdown
---
name: test-coverage
description: Analyze test coverage and suggest missing tests
context: fork
agent: Explore
---

Analyze test coverage for $ARGUMENTS:

1. Find test files (*.test.*, *.spec.*, tests/)
2. Map which modules/functions have tests
3. Find untested code paths
4. Look for edge cases not covered

Output:
- What's well tested
- Critical paths without tests (prioritized)
- Suggested test cases for the top 3 gaps

Be specific: name the function, describe the test case.
```

---

## Шаблон для нового skill

```markdown
---
name: my-skill
description: |
  Clear description of what this skill does.
  Use when the user wants to [action].
  Activate automatically when [trigger].
disable-model-invocation: false  # true = только /my-skill
allowed-tools: Read, Grep        # Инструменты без запроса
---

# My Skill

## When to use
[Условия применения]

## Steps

1. [Первый шаг]
2. [Второй шаг с $ARGUMENTS]
3. [Завершающий шаг]

## Output format
[Как должен выглядеть результат]
```

---

## See Also

- [Anatomy](anatomy.md) — структура, frontmatter, все поля
- [Multi-agent](../agents/multi-agent.md) — context: fork и subagents
- [Archivist Sonnet](../archivist/sonnet.md) — пример полноценного skill-агента
