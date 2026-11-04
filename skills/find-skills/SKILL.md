---
name: find-skills
description: Searches the npx skills registry and skills.sh for an installable third-party package, vets its source and contents, and installs it only with consent. Use when the user asks "is there a skill for X", asks you to find, add, or install a skill, or wants to extend the agent with a new capability or specialized domain help (a framework, file format, or platform) that nothing installed covers.
license: MIT
metadata:
  adapted-from: https://github.com/vercel-labs/skills/tree/main/skills/find-skills
---

# Find Skills

## Overview

Locates a third-party skill in the open agent skills ecosystem (the `npx skills` CLI and the [skills.sh](https://skills.sh/) directory), checks that it is trustworthy, and presents it to the user before anything is installed. A skill is executable instructions that will run with the agent's permissions, so finding one is a supply-chain decision, not a search result to paste.

## When to Use

- The user asks "is there a skill for X", "find a skill for X", or "can you do X" where X is a specialized capability
- The user wants to extend the agent's capabilities in a domain (a framework, a file format, a deployment target)
- The task needs domain knowledge that no installed skill covers, and the user is open to installing one

**When NOT to use:**

- Choosing which already-installed skill applies to the current work: use `using-agent-skills`
- The task is routine and you can do it well without a skill: do it directly
- The user has not asked to extend the agent and the work is already underway: finish it, then mention a skill only if it would clearly help next time

## Process

### Step 1: Pin down the need

Name three things before searching:

1. **Domain** (React, testing, PDF processing, Kubernetes)
2. **Task** (writing tests, reviewing PRs, generating a changelog)
3. **Why a skill**: what an installed skill would add that you cannot do from general knowledge

If you cannot state item 3, skip the search and do the task directly.

### Step 2: Search

Check the [skills.sh leaderboard](https://skills.sh/) for the domain first; it ranks skills by install count. Then search with the CLI:

```bash
npx skills find <specific keywords> [--owner <github-owner>]
```

| User says | Query |
|---|---|
| "How do I make my React app faster?" | `npx skills find react performance` |
| "Can you help with PR reviews?" | `npx skills find pr review` |
| "I need to create a changelog" | `npx skills find changelog` |

Use specific keywords ("react testing", not "testing"). If nothing comes back, try synonyms ("deploy" → "deployment", "ci-cd").

### Step 3: Vet each candidate

Do not recommend a skill from search results alone. For each candidate, record:

| Check | Threshold |
|---|---|
| Install count | Prefer 1K+; treat under 100 with caution |
| Source | Known publishers (official org accounts) over unknown authors |
| Repository | Stars, recent commits, open issues; under 100 stars needs a reason to trust it |
| Contents | Read the `SKILL.md` and any `scripts/` it ships. Reject skills that fetch and execute remote code, ask for credentials, or disable safety checks |

The dependency-vetting rules in `security-and-hardening` apply: a skill is a dependency that runs with the agent's permissions.

### Step 4: Present options

For each vetted skill, give the user:

1. Name and one-line description
2. Install count and source repository
3. The exact install command
4. A link to the skill's page

```
The "react-best-practices" skill covers React and Next.js performance
guidelines (vercel-labs/agent-skills, 185K installs).

Install: npx skills add vercel-labs/agent-skills@react-best-practices
More:    https://skills.sh/vercel-labs/agent-skills/react-best-practices
```

### Step 5: Install only on explicit consent

Install only after the user says yes, and confirm the scope with them:

```bash
npx skills add <owner/repo@skill>        # project scope
npx skills add <owner/repo@skill> -g     # user (global) scope
```

Do not add `-y` to skip the CLI's own confirmation unless the user asked for a non-interactive install. After installing, confirm the skill appears in the agent's skill list.

### Step 6: When nothing fits

Say plainly that no suitable skill was found, offer to do the task directly, and, if the task recurs, point out that the user can write their own (`npx skills init <name>`).

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The top search result is obviously the right one" | Search ranks by keyword match, not quality or safety. Vet it first. |
| "It's only a markdown file, it can't do harm" | A skill instructs the agent, and can ship scripts. It runs with the agent's permissions. |
| "Installing it with `-g -y` saves the user a step" | It also skips the user's decision. Global, unconfirmed installs are how unwanted code lands everywhere. |
| "A skill exists, so I should use it" | A skill is worth installing only when it adds something you can't already do well. |

## Red Flags

- Recommending a skill without having opened its `SKILL.md` or repository
- Installing before the user said yes, or choosing global scope without asking
- A candidate skill that downloads and runs remote code, or asks for tokens or credentials
- Searching for a skill when the task is something the agent already does well
- Using this skill to pick between skills that are already installed

## Verification

- [ ] The need (domain, task, why a skill) was stated before searching
- [ ] Every recommended skill has its install count, source, and repository checked, and its `SKILL.md` read
- [ ] The user saw name, source, install count, command, and link before any install
- [ ] Nothing was installed without explicit consent; the scope (project or global) was confirmed
- [ ] After an install, the skill shows up in the agent's skill list
- [ ] If nothing fit, the user was told so and offered direct help
