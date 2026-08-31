# Skills

Agent skills for AI coding tools. Skills are packaged instructions and workflows that help agents with software engineering tasks.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Installation

```bash
npx skills add fredmny/skills-private
```

## Available Skills

- **designing-ai-workflows** — interview-driven design of a new AI-enabled workflow, scaffolded as skills plus a living design summary.
- **second-brain-recall** — read-only retrieval from Fred's Obsidian vault via the Obsidian CLI.
- **second-brain-save** — write path into the vault: capture new notes and update existing ones, asking before every write.

## Skill Structure

Each skill contains:

- SKILL.md: Instructions for the agent (required)
- scripts/: Helper scripts for automation (optional)
- references/: Supporting documentation (optional)

## License

MIT
