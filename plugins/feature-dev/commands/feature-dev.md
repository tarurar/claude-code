---
description: Guided feature development with codebase understanding and architecture focus
argument-hint: Optional feature description
---

# Feature Development

You are helping a developer implement a new feature using an **agent team**. You are the team lead. Follow a systematic approach: understand the codebase deeply through coordinated exploration, identify and ask about all underspecified details, design elegant architectures through collaborative design, then implement.

## Core Principles

- **Ask clarifying questions**: Identify all ambiguities, edge cases, and underspecified behaviors. Ask specific, concrete questions rather than making assumptions. Wait for user answers before proceeding with implementation. Ask questions early (after understanding the codebase, before designing architecture).
- **Understand before acting**: Read and comprehend existing code patterns first
- **Read files identified by teammates**: After exploration teammates complete their work, read the files they identified to build detailed context before proceeding.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Coordinate, don't duplicate**: As team lead, delegate exploration and review to teammates. Focus on synthesis, decision-making, and implementation.
- **Manage the team lifecycle**: Create the team early, shut down teammates when their phase is done, clean up at the end.

---

## Phase 1: Discovery & Team Setup

**Goal**: Understand what needs to be built and set up the agent team

Initial request: $ARGUMENTS

**Actions**:
1. If the feature is unclear, ask the user for:
   - What problem are they solving?
   - What should the feature do?
   - Any constraints or requirements?
2. Summarize understanding and confirm with user
3. Create the agent team using `TeamCreate` with a descriptive team name (e.g., "feature-dev-auth" or "feature-dev-caching")
4. Create tasks for all major phases using `TaskCreate`:
   - Exploration tasks (Phase 2) — one per exploration focus area
   - Architecture tasks (Phase 4) — one per design approach
   - Review tasks (Phase 6) — one per review focus area
   - Set dependencies: architecture tasks `addBlockedBy` exploration tasks; review tasks `addBlockedBy` architecture tasks

---

## Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns at both high and low levels through coordinated team exploration

**Actions**:
1. Spawn 2-3 explorer teammates using the `Task` tool with `team_name` set to your team name and `subagent_type` set to `feature-dev:code-explorer`. Give each teammate a distinct `name` (e.g., "explorer-similar", "explorer-arch", "explorer-patterns"). Assign each a different exploration focus:

   **Example spawn prompts**:
   - "Find features similar to [feature] and trace through their implementation comprehensively. Share any critical patterns you discover with other explorers via SendMessage."
   - "Map the architecture and abstractions for [feature area], tracing through the code comprehensively. If other explorers share patterns they've found, factor those into your analysis."
   - "Analyze the current implementation of [existing feature/area], tracing through the code comprehensively. Identify UI patterns, testing approaches, or extension points relevant to [feature]."

2. Assign the exploration tasks you created in Phase 1 to each teammate using `TaskUpdate` with the `owner` parameter
3. Wait for all explorer teammates to complete their tasks and send you their findings. Do NOT start doing exploration work yourself — let the team handle it.
4. Once all explorers have reported back, read all files they identified to build deep understanding
5. Shut down explorer teammates using `SendMessage` with `type: "shutdown_request"` — they are no longer needed
6. Present comprehensive summary of findings and patterns discovered to the user

---

## Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities before designing

**CRITICAL**: This is one of the most important phases. DO NOT SKIP.

**Actions**:
1. Review the exploration findings and original feature request
2. Identify underspecified aspects: edge cases, error handling, integration points, scope boundaries, design preferences, backward compatibility, performance needs
3. **Present all questions to the user in a clear, organized list**
4. **Wait for answers before proceeding to architecture design**

If the user says "whatever you think is best", provide your recommendation and get explicit confirmation.

---

## Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches with different trade-offs through collaborative architecture work

**Actions**:
1. Spawn 2-3 architect teammates using the `Task` tool with `team_name` set to your team name and `subagent_type` set to `feature-dev:code-architect`. Give each a distinct `name` (e.g., "architect-minimal", "architect-clean", "architect-pragmatic"). Each should focus on a different approach:
   - **Minimal changes**: Smallest change, maximum reuse of existing code
   - **Clean architecture**: Maintainability, elegant abstractions, proper separation
   - **Pragmatic balance**: Speed + quality, practical trade-offs

   Include in each spawn prompt: the feature description, user's answers to clarifying questions, and key findings from Phase 2. Tell architects to share key patterns they find and to challenge each other's designs via `SendMessage`.

2. Assign the architecture tasks to each teammate using `TaskUpdate` with the `owner` parameter
3. Wait for all architect teammates to complete their designs and send you their blueprints. Do NOT start designing yourself.
4. Shut down architect teammates using `SendMessage` with `type: "shutdown_request"`
5. Review all approaches and form your opinion on which fits best for this specific task (consider: small fix vs large feature, urgency, complexity, team context)
6. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences
7. **Ask user which approach they prefer**

---

## Phase 5: Implementation

**Goal**: Build the feature

**DO NOT START WITHOUT USER APPROVAL**

**Actions**:
1. Wait for explicit user approval of the chosen architecture
2. Read all relevant files identified in previous phases
3. Implement following chosen architecture
4. Follow codebase conventions strictly
5. Write clean, well-documented code
6. Update tasks as you progress using `TaskUpdate`

---

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct through multi-perspective team review

**Actions**:
1. Spawn 3 reviewer teammates using the `Task` tool with `team_name` set to your team name and `subagent_type` set to `feature-dev:code-reviewer`. Give each a distinct `name` (e.g., "reviewer-quality", "reviewer-bugs", "reviewer-conventions"). Each should focus on a different aspect:
   - **Simplicity/DRY/Elegance**: Code quality and maintainability
   - **Bugs/Functional correctness**: Logic errors, security vulnerabilities, edge cases
   - **Project conventions/Abstractions**: Adherence to codebase patterns and CLAUDE.md rules

   Tell reviewers to cross-reference each other's findings via `SendMessage` to reduce false positives and strengthen high-confidence issues.

2. Assign the review tasks to each reviewer using `TaskUpdate` with the `owner` parameter
3. Wait for all reviewer teammates to complete and send you their findings. Do NOT start reviewing yourself.
4. Shut down reviewer teammates using `SendMessage` with `type: "shutdown_request"`
5. Consolidate findings and identify highest severity issues that you recommend fixing
6. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is)
7. Address issues based on user decision

---

## Phase 7: Summary & Cleanup

**Goal**: Document what was accomplished and clean up team resources

**Actions**:
1. Mark all remaining tasks as completed using `TaskUpdate`
2. Summarize:
   - What was built
   - Key decisions made
   - Files modified
   - Suggested next steps
3. Clean up the team using `TeamDelete` to remove shared team resources

---
