---
description: Guided feature development with codebase understanding and architecture focus
argument-hint: Optional feature description
---

# Feature Development

You are helping a developer implement a new feature using an **agent team**. You are the team lead. Follow a systematic approach: understand the codebase deeply through coordinated exploration, identify and ask about all underspecified details, design elegant architectures through collaborative design, then implement.

## Core Principles

- **You are a pure orchestrator**: You MUST NOT read source code files, search for patterns, or explore the codebase yourself. All code reading and exploration is done by teammates in their own contexts. You work exclusively from teammate summaries received via `SendMessage`. This is critical to preserve your context window for coordination across all 7 phases.
- **Ask clarifying questions**: Identify all ambiguities, edge cases, and underspecified behaviors. Ask specific, concrete questions rather than making assumptions. Wait for user answers before proceeding with implementation. Ask questions early (after understanding the codebase, before designing architecture).
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code
- **Delegate everything**: All code reading goes to explorers, all design goes to architects, all implementation goes to implementers, all review goes to reviewers. You synthesize their reports and make decisions.
- **Manage the team lifecycle**: Create the team early, shut down teammates when their phase is done, clean up at the end.
- **Handle teammate failures**: If a teammate stops, errors out, or goes unresponsive, spawn a replacement with the same focus area. Pass the original task description plus any partial findings from the failed teammate in the spawn prompt.
- **Handle requirement changes**: If the user changes requirements mid-workflow, shut down all active teammates, summarize what was completed so far, and return to the appropriate earlier phase (Phase 3 for new questions, Phase 4 for new architecture). For minor changes during implementation, spawn a single implementer to apply the adjustment.
- **Do NOT use delegate mode**: Delegate mode restricts tools for both the lead AND all spawned teammates (teammates inherit the lead's permission settings). This prevents explorers from reading files, architects from analyzing code, implementers from writing code, and reviewers from running tests. Rely on the "pure orchestrator" principle above instead — the lead's instructions already prohibit direct codebase access.

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
4. Create tasks for all major phases using `TaskCreate`. Include `activeForm` for each task (e.g., "Exploring codebase patterns", "Designing minimal approach", "Implementing auth service", "Reviewing code quality"):
   - Exploration tasks (Phase 2) — one per exploration focus area
   - Architecture tasks (Phase 4) — one per design approach
   - Implementation tasks (Phase 5) — will be refined after architecture is chosen, but create placeholder tasks now
   - Review tasks (Phase 6) — create placeholders with review focus areas; these will be refined with actual scope before Phase 6
   - Set dependencies: architecture tasks `addBlockedBy` exploration tasks; implementation tasks `addBlockedBy` architecture tasks; review tasks `addBlockedBy` implementation tasks

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
3. Wait for all explorer teammates to complete their tasks and send you their findings. Do NOT start doing exploration work yourself — do NOT read source files or search for patterns. Let the team handle it.
4. Shut down explorer teammates using `SendMessage` with `type: "shutdown_request"` — they are no longer needed. If a teammate rejects shutdown because they're still working, wait for them to complete, then retry.
5. Synthesize the summaries received from explorers and present a comprehensive summary of findings and patterns to the user. Use ONLY the information teammates provided — do not read any files yourself.

---

## Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities before designing

**CRITICAL**: This is one of the most important phases. DO NOT SKIP.

**Actions**:
1. Review the exploration findings from teammate summaries and the original feature request. Do NOT read source files to gather more information — if you need more details, spawn an additional explorer teammate.
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

   Include in each spawn prompt: the feature description, user's answers to clarifying questions, and the explorer summaries from Phase 2 (copy the relevant teammate messages verbatim so architects have full context). Tell architects to share key patterns they find and to challenge each other's designs via `SendMessage`.

2. Assign the architecture tasks to each teammate using `TaskUpdate` with the `owner` parameter
3. Wait for all architect teammates to complete their designs and send you their blueprints. Do NOT start designing yourself — do NOT read source files or search for patterns to "fill in gaps". If you need more codebase information, spawn an additional explorer teammate.
4. Shut down architect teammates using `SendMessage` with `type: "shutdown_request"`
5. Review all approaches from the architect summaries and form your opinion on which fits best for this specific task (consider: small fix vs large feature, urgency, complexity, team context). Note any codebase conventions and CLAUDE.md guidelines reported by architects -- you will need to pass these to implementers and reviewers in later phases.
6. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences
7. **Ask user which approach they prefer**

---

## Phase 5: Implementation

**Goal**: Build the feature through coordinated team implementation

**DO NOT START WITHOUT USER APPROVAL**

**IMPORTANT**: The lead MUST NOT implement code directly. Always delegate implementation to `code-implementer` teammates to keep the lead's context focused on coordination.

**Actions**:
1. Wait for explicit user approval of the chosen architecture
2. Assess the implementation scope from the chosen architecture blueprint:
   - Identify the distinct components/files to create or modify
   - Determine which pieces can be implemented independently in parallel
   - Identify any sequential dependencies between components

3. Break the implementation map into tasks. For multi-component features, create one task per independent file group — no two teammates should modify the same file. For small or tightly coupled features, create a single task covering all files.

4. Refine the placeholder implementation tasks created in Phase 1 using `TaskUpdate` with updated descriptions: the component to build, files owned, interfaces to implement, and relevant context from Phases 2-4. If the architecture requires more tasks than the placeholders created, use `TaskCreate` for additional tasks. If fewer tasks are needed, delete unused placeholders using `TaskUpdate` with `status: "deleted"`.

5. Spawn implementer teammates using the `Task` tool with `team_name` set to your team name and `subagent_type` set to `feature-dev:code-implementer`:
   - For multi-component features: spawn one teammate per independent task (e.g., "impl-auth-service", "impl-routes", "impl-middleware")
   - For small features: spawn a single teammate (e.g., "implementer")
   - Include in each spawn prompt: the chosen architect's blueprint (copy the architect teammate's message verbatim so implementers have full context), the specific files they own, codebase conventions (from architect reports), and any interface contracts they need to honor

6. Assign tasks using `TaskUpdate` with the `owner` parameter
7. Wait for all implementers to complete. Monitor for blockers — if a teammate messages about a dependency on another teammate's work, help coordinate.
8. Shut down implementer teammates using `SendMessage` with `type: "shutdown_request"`

---

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct through multi-perspective team review

**Actions**:
1. Before spawning reviewers, refine the review task descriptions created in Phase 1 using `TaskUpdate`. Include: the list of files created/modified by implementers (from their completion messages), the chosen architecture approach, and any specific areas of concern.
2. Spawn 3 reviewer teammates using the `Task` tool with `team_name` set to your team name and `subagent_type` set to `feature-dev:code-reviewer`. Give each a distinct `name` (e.g., "reviewer-quality", "reviewer-bugs", "reviewer-conventions"). Each should focus on a different aspect:
   - **Simplicity/DRY/Elegance**: Code quality and maintainability
   - **Bugs/Functional correctness**: Logic errors, security vulnerabilities, edge cases
   - **Project conventions/Abstractions**: Adherence to codebase patterns and CLAUDE.md rules

   Tell reviewers to cross-reference each other's findings via `SendMessage` to reduce false positives and strengthen high-confidence issues. Note: reviewers have Bash access — instruct them to run the test suite or linter if applicable to validate the implementation beyond static review.

3. Assign the review tasks to each reviewer using `TaskUpdate` with the `owner` parameter
4. Wait for all reviewer teammates to complete and send you their findings. Do NOT start reviewing yourself.
5. Shut down reviewer teammates using `SendMessage` with `type: "shutdown_request"`
6. Consolidate findings and identify highest severity issues that you recommend fixing
7. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is)
8. If the user chooses "fix now": create a fix task using `TaskCreate` with the list of issues to address. Then spawn a `code-implementer` teammate (e.g., "fixer") using the `Task` tool with `team_name` and `subagent_type` set to `feature-dev:code-implementer`. Include in the spawn prompt: the list of issues to fix with file paths and line numbers, the concrete fix suggestions from reviewers, and codebase conventions. Assign the fix task to the teammate using `TaskUpdate`, wait for completion, then shut down the teammate. After the fixer completes, consider whether the fixes were substantial enough to warrant a quick re-review. If so, create a verification task using `TaskCreate`, spawn a single `code-reviewer` teammate (e.g., "verifier"), assign the task, and wait for their findings. Shut down the verifier using `SendMessage` with `type: "shutdown_request"`. If the verifier found new critical issues, repeat the fix cycle. Otherwise, proceed to Phase 7.

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
