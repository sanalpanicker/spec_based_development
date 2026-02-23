# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository demonstrates a **spec-driven AI development workflow** for building any app. The workflow emphasizes structured phases (Specifying → Planning → Implementing) to maintain control over AI-assisted development and prevent architectural drift.

## Directory Structure

The repository follows a three-tier organization pattern:

```
project-root/
├── .claude/commands/          # Custom slash commands
│   └── spec.md               # /spec command for generating specifications
├── _docs/                    # PRDs, HLDs, and reference documentation
│   ├── PRD.md                # High-level business requirements
│   ├── HLD.md                # Architectural decisions
│   ├── *-PRD.md              # Additional PRD documents (as needed)
│   ├── *-HLD.md              # Additional HLD documents (as needed)
│   └── README.md             # Documentation guide
├── _specs/                   # High-level feature specifications
│   ├── template.md           # Reusable spec template
│   └── W-XXXXX-<feature>.md  # Generated specs with WI numbers
├── _plans/                   # Technical implementation plans
│   └── W-XXXXX-<feature>-plan.md
└── src/                      # (Future) Application source code
```

**Note**: The underscore prefix on `_specs/` and `_plans/` keeps them at the top of directory listings for easy access.

## Workflow Phases

### Phase 1: Specification Generation

Use the custom `/spec` command with work item format:
```
/spec @W-XXXXX [FEAT|BUG|CHORE] <Feature Name>
```

**The `/spec` command automatically:**
- Discovers available PRD and HLD documents in `_docs/`
- Prompts user to select relevant PRD and HLD (if multiple exist)
- Reads selected documents for context
- Parses the ticket ID, type, and feature name
- Validates alignment with business requirements and architecture
- Creates spec file as `_specs/W-XXXXX-<feature-slug>.md`
- Creates appropriate git branch:
  - `[FEAT]` → `feature/W-XXXXX-<feature-slug>`
  - `[BUG]` → `bugfix/W-XXXXX-<feature-slug>`
  - `[CHORE]` → `chore/W-XXXXX-<feature-slug>`

**CRITICAL**:
- **PRD and HLD provide broader context** - They describe the overall product vision and system architecture
- **Your spec focuses on THIS feature only** - Use PRD/HLD to understand the big picture and ensure alignment, but spec and plan only the specific Work Item at hand
- **Don't over-scope** - If the PRD mentions 10 features, but the WI is for 1 feature, only spec that 1 feature
- The `/spec` command will prompt you to choose the correct documents if multiple exist

**Required in spec**: For front-end features, include Figma component/design references.

### Phase 2: Technical Planning

1. Enter **Plan Mode** (`Shift + Tab`)
2. Open the finalized spec file as context
3. Prompt: _"Plan the feature described in this spec"_
4. Save generated plan to `_plans/W-XXXXX-<feature>-plan.md`

**Important Context**: The HLD provides architectural guidance and constraints. The plan should respect the HLD's architecture but **only plan the specific feature in the spec**, not everything mentioned in the HLD or PRD.

Plans should include:
- Existing components/patterns to reuse
- Files to create/modify
- Architectural decisions (aligned with HLD)
- Feature flags or technical constraints (manually add if needed)

### Phase 3: Implementation

Two implementation options:

**Option A - Manual Implementation** (complex features):
- Switch to Opus 4.5 model
- Enable Extended Thinking Mode (Option+T on Mac, or type "ultraink")
- Prompt with plan: _"Can you implement this plan?"_

**Option B - Feature-Dev Plugin** (medium complexity):
```
/feature-dev:feature-dev Please implement the new feature exactly as outlined in @W-XXXXX-<feature>-plan.md
```

## Work Item Format

All tickets follow the format: `@W-XXXXX [TYPE] Description`
- Ticket ID format: `W-` followed by numbers
- Types: `FEAT` (features), `BUG` (bugfixes), `CHORE` (maintenance)
- Feature names are converted to kebab-case slugs in file names

## Integration Points

- **PRD/HLD (REQUIRED)**: Store PRD and HLD documents in `_docs/` directory. The `/spec` command discovers available documents and prompts you to select the correct ones, ensuring alignment with business requirements and architectural decisions
- **Multiple PRDs/HLDs**: You can maintain multiple PRD and HLD documents for different features or modules. Name them descriptively (e.g., `auth-PRD.md`, `data-sync-HLD.md`)
- **Additional docs**: Pass supplementary context using `@` symbol if needed
- **Remote docs**: Use Model Context Protocol (MCP) to read external documentation
- **Ticketing systems**: Use MCP to connect to Jira or similar systems
- **Figma designs**: Reference in specs for UI features

## Key Principles

- **Context vs. Scope**: PRD/HLD provide context for the big picture; specs and plans focus only on the current Work Item
- Never write code before completing the spec and plan phases
- Always include WI numbers in spec and plan file names
- Engineer must review and refine specs before planning
- Technical plans must be verified before implementation
- For front-end work, Figma references are mandatory
- **One feature at a time**: Don't try to spec/plan multiple features from the PRD in a single Work Item
