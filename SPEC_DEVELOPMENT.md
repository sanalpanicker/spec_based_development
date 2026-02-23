**Proposal: Adopting a Spec-Driven AI Development Workflow with Claude Code**

**Objective** To standardize our approach to AI-assisted coding by adopting a lightweight, spec-driven development workflow using Claude Code. This workflow guarantees that our AI tools remain focused on our exact requirements, minimizes incorrect assumptions, and ensures high-quality code generation while keeping engineers firmly in control of the development loop.

**Core Philosophy** Instead of prompting the AI to immediately write code, we will break feature development into three distinct phases: **Specifying, Planning, and Implementing**. This prevents AI context-loss and ensures architectural decisions align with our existing codebase before any code is generated.

***


### **Phase 1: Workspace Setup (One-Time Initialization)**

Before engineers can begin, the repository must be configured to support the spec-driven workflow.

**1. Directory Structure**: Create dedicated folders in the project root for specifications and technical plans. The use of an underscore at the beginning of the folder names ensures they sit at the top of your directory tree for easy access.

    your-project-root/
    ├── .claude/
    │   └── commands/
    │       └── spec.md                           (Your custom /spec command file)
    ├── _plans/                                   (Dedicated to technical implementation plans)
    │   └── W-12345-authentication-forms-plan.md  (Example generated technical plan with WI number)
    ├── _specs/                                   (Dedicated to high-level specifications)
    │   ├── template.md                           (Your reusable spec template)
    │   └── W-12345-authentication-forms.md       (Example generated high-level spec with WI number)
    ├── src/
    └── ...

**2. Spec Template**: Inside the `_specs/` folder, create a `template.md` file. This template should include sections for a feature summary, functional requirements, a **Figma component or design reference** (for front-end features utilizing Figma), edge cases, acceptance criteria, open questions, and testing guidelines.

**3. Custom Spec Command (Configured for WI Parsing & File Naming)**: Create a `/spec` custom slash command in the project's `.claude/commands` directory (e.g., in a `spec.md` file). This command should explicitly grant the `bash`, `read`, `write`, and `glob` tools.

To support our Work Item format (`@W-XXXXX [FEAT/BUG/CHORE] <Feature Name>`) and ensure the WI number is in the file names, the command instructions must dictate how to parse the user's input:

- Parse the input to extract the **Ticket ID** (e.g., W-XXXXX), the **Type** (FEAT, BUG, or CHORE), and the **Feature Name**.

- Convert the Feature Name into a lowercase, kebab-case slug.

- **Generate the Spec File Name:** Instruct the model to write the spec markdown file into the `_specs/` directory using the format: `<Ticket-ID>-<feature-slug>.md` (e.g., `W-12345-new-login.md`).

- **Generate the Git Branch Name:**

  - If `[FEAT]`: create branch as `feature/<Ticket-ID>-<feature-slug>`

  - If `[BUG]`: create branch as `bugfix/<Ticket-ID>-<feature-slug>`

  - If `[CHORE]`: create branch as `chore/<Ticket-ID>-<feature-slug>`

***


### **Phase 2: Step-by-Step Guide for Engineers**

When an engineer picks up a new ticket or feature, they will follow this standardized process:


#### **Step 1: Generate and Refine the Feature Spec**

The goal of this step is to define _what_ the feature is and how it behaves from a user's perspective, without defining _how_ it will be coded.

- **Action**: In Claude Code, run the custom command with the ticket details.

- **Integrating Requirements**:

  - _If HLD/PRD are available locally:_ Pass them as context using the `@` symbol (e.g., `/spec @W-12345 [FEAT] New Auth @PRD.md @HLD.md`).

  - _If HLD/PRD are remote:_ Use the Model Context Protocol (MCP) to allow Claude to read external docs directly.

  - _If no HLD/PRD is available:_ Use MCP to connect to your Jira/ticketing system, or manually paste the ticket description into the prompt or a local text file (e.g., `@ticket.txt`).

- **Automation**: Claude will automatically switch to the properly formatted Git branch and generate a high-level Markdown specification file (named with the WI number, e.g., `W-12345-new-auth.md`) in the `_specs/` folder.

- **Engineer Review (CRITICAL)**: The engineer must read through the functional requirements and edge cases, and answer any "open questions" Claude generated. **If this is a front-end feature, ensure the correct Figma component or design reference is included**. Edit the document until the high-level requirements are perfectly accurate.


#### **Step 2: Generate the Technical Plan**

The goal of this step is to determine the optimal technical implementation strategy based on our existing codebase.

- **Action**: Press `Shift + Tab` in Claude Code to enter **Plan Mode**.

- **Automation**: Open the finalized spec file (and your HLD document, if available) so it is added as context, and prompt: _"Plan the feature described in this spec"_. Claude will spin up sub-agents to deeply research our repository, locate existing components, and generate a comprehensive step-by-step implementation plan.

- **Engineer Review**: When prompted, choose to save the plan to the `_plans/` folder. **Ensure you save the file with the WI number appended to the name** (e.g., `W-12345-new-auth-plan.md`). The engineer must review this plan to verify architectural choices and design patterns.

- **Adding Technical Constraints**: If the feature requires specific technical configurations not caught by the AI—such as wrapping the UI behind a specific **feature flag**—the engineer should manually type those requirements into this plan document now.


#### **Step 3: Implement the Plan**

With a verified technical plan, the actual code generation is highly predictable and reliable. Engineers have two paths for implementation:

**Option A: Manual Implementation (Best for highly complex features)**

- Switch the model to an advanced model like **Opus 4.5**, as it strictly follows detailed plan instructions without deviating.

- Enable **Extended Thinking Mode** (via settings, pressing `Option+T` on Mac, or typing `ultraink` in the prompt) to give the model maximum reasoning capacity for complex, multi-file changes.

- Open the newly named plan document as context and prompt: _"Can you implement this plan?"_.

**Option B: The** `feature-dev` **Plugin (Best for medium-complexity features)**

- Trigger Anthropic's `feature-dev` plugin using its custom command, referencing the specific plan file by its WI-numbered name: `/feature-dev:feature-dev Please implement the new feature exactly as outlined in @W-12345-[feature-slug]-plan.md`.

- The plugin will run a guided 7-phase process: asking clarifying questions, exploring the codebase, presenting an approach, and writing the code.

- It inherently includes a built-in quality review phase at the end, resulting in higher code quality with fewer bugs.


#### **Step 4: Verification and Merge**

- Once Claude Code completes the implementation, the engineer conducts standard verification: review the generated tests, check the UI in the browser, and ensure no linting errors were introduced.

- Stage and commit the code to the feature branch, and either merge locally or open a Pull Request for team review.
