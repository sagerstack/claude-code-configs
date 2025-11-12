# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Getting Started with Claude Code

### Installation and Setup

1. **Download Claude Code**: Visit https://claude.ai/code to download Claude Code for your platform (macOS, Windows, Linux)

2. **Install Claude Code**:
   - macOS: Download the `.dmg` file and drag Claude Code to Applications
   - Windows: Download the `.exe` installer and run it
   - Linux: Download the appropriate package for your distribution

3. **Authenticate**: Launch Claude Code and sign in with your Anthropic account

4. **Configure with this Repository**:
   - Clone this repository: `git clone <repository-url>`
   - Navigate to the directory: `cd claude-code-configs`
   - Open the repository in Claude Code to automatically load the custom agents and commands

### Using Repeated Instructions

There are two ways to add repeated instructions that Claude Code should follow:

**Method 1: Global Instructions in `~/.claude/CLAUDE.md`**
- Create or edit `~/.claude/CLAUDE.md` (your personal global configuration)
- Instructions here apply to ALL projects and repositories
- Use this for development preferences, coding standards, or workflow patterns you want everywhere

**Method 2: Project-Specific Instructions in Repository `CLAUDE.md`**
- Add instructions to the `CLAUDE.md` file in any specific repository
- Instructions here apply ONLY to that repository
- Use this for project-specific patterns, architecture guidelines, or domain-specific requirements

**Example Global Instruction** (add to `~/.claude/CLAUDE.md`):
```markdown
# My Global Development Preferences

- Always use TypeScript for JavaScript projects
- Follow semantic versioning for releases
- Include comprehensive error handling
- Use conventional commit messages (feat:, fix:, docs:, etc.)
```

**Example Project Instruction** (add to repository `CLAUDE.md`):
```markdown
# Project-Specific Rules

- Always validate API responses with Zod schemas
- Use dependency injection for services
- Follow the repository's specific naming conventions
- Test critical paths with integration tests
```

**Important**: Instructions in your global `~/.claude/CLAUDE.md` take precedence over project-specific instructions when there are conflicts.

### Permission Bypass Mode

For enhanced functionality, you may want to enable permission bypass mode, which allows Claude Code to execute commands and file operations without requiring interactive confirmation for each action.

**⚠️ Security Note**: This mode reduces security by allowing Claude Code to execute commands without confirmation. Only enable this if you trust the repository and understand the implications.

**To enable permission bypass mode**:

1. Create a file named `settings.local.json` in your `.claude` directory:
   ```bash
   # In your project directory
   touch .claude/settings.local.json
   ```

2. Add the following configuration to `.claude/settings.local.json`:
   ```json
   {
     "defaultMode": "bypassPermissions"
   }
   ```

3. The `.claude/settings.local.json` file is already included in `.gitignore` to prevent accidental commits of sensitive settings.

**What this does**:
- Automatically approves file read/write operations
- Automatically approves command executions
- Eliminates interactive prompts for permissions
- Maintains all other safety features and restrictions

**When to use this**:
- Trusted development environments
- Automated workflows
- Projects where you frequently execute similar operations
- When you understand the security implications

## Project Overview

This is a **Claude Code configuration repository** that contains custom agents and slash commands for extending Claude Code functionality. The project is structured around the `.claude/` directory which houses specialized agents and command definitions.

## Architecture

### Directory Structure
- `.claude/` - Main configuration directory
  - `agents/` - Custom agent definitions
    - `timekeeper.md` - Agent for providing current date/time
    - `random-number-generator.md` - Agent for generating random numbers (1-1000)
    - `number-fact-generator.md` - Agent for generating interesting facts about numbers
  - `commands/` - Custom slash command definitions
    - `daily-fact.md` - Command that orchestrates all three agents to create daily facts

### Agent System
The repository implements a modular agent system where:
- Each agent has a specific, focused responsibility
- Agents are defined as markdown files with YAML frontmatter
- The `daily-fact` command orchestrates multiple agents in sequence
- Agents inherit the model from the parent and use Read/Bash tools

### Agent Configuration Pattern
All agents follow this structure:
```yaml
---
name: agent-name
description: Agent description
tools: Read, Bash
model: inherit
color: yellow
---
```

## Development Commands

Since this is a configuration repository (not a code project), there are no traditional build/test commands. Development involves:

1. **Creating new agents**: Add `.md` files to `.claude/agents/`
2. **Creating new commands**: Add `.md` files to `.claude/commands/`
3. **Testing agents**: Use Claude Code to invoke the agents directly
4. **Testing commands**: Use Claude Code to run the slash commands

## Key Patterns

### Agent Design
- Agents should have a single, clear responsibility
- Agent names should be descriptive and hyphen-separated
- Agents respond with minimal, focused output
- All agents use `tools: Read, Bash` and `model: inherit`

### Command Design
- Commands orchestrate multiple agents for complex workflows
- Commands define clear sequences of agent invocations
- Commands provide structured, user-friendly output formatting

### File Naming Conventions
- Agent files: `{agent-name}.md`
- Command files: `{command-name}.md`
- Use kebab-case for all file names
- Descriptive names that clearly indicate functionality

## Testing

To test agents and commands:
1. Use Claude Code's built-in agent invocation
2. Test slash commands using the `/command-name` syntax
3. Verify agent outputs are correct and minimal
4. Test command orchestration flows end-to-end

## Git Repository Management

This repository follows standard Git practices with protected main branch workflow:
- Main branch is protected (cannot push directly)
- Create feature branches for changes
- Use pull requests to merge to main
- Always merge latest main changes into feature branches to avoid conflicts