# About

A small collection of agent skills.

## Skills

| Skill | What it does | Install |
|-------|--------------|---------|
| [`code-audit`](./code-audit/SKILL.md) | Audits any codebase for dark patterns, code smells, security and architecture issues. Adapts to the repo's detected modules/units and stacks instead of assuming web/mobile/API. | `npx skills@latest add guidobuilds/skills --skill code-audit` |
| [`working-workflow`](./working-workflow/SKILL.md) | General way of working for the agent: Linear as the single source of truth, a defined status flow, and specs and sub-issues registered before development. Launch at the start of any work. | `npx skills@latest add guidobuilds/skills --skill working-workflow` |

## Installation

1. Run:

   ```bash
   npx skills@latest add guidobuilds/skills
   ```

2. Select the skills you want to use
3. Start a new chat/session. The skill is picked up automatically when its trigger phrases match, or when you ask the agent to run it.
