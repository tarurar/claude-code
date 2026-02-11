# Feature Development Plugin

A team-based feature development workflow using agent teams for coordinated codebase exploration, collaborative architecture design, and multi-perspective quality review.

## Overview

The Feature Development Plugin provides a systematic 7-phase approach to building new features, powered by **agent teams**. Instead of jumping straight into code, it creates a coordinated team of specialized agents that explore your codebase, design architecture collaboratively, and review code from multiple perspectives — resulting in better-designed features that integrate seamlessly with your existing code.

## Philosophy

Building features requires more than just writing code. You need to:
- **Understand the codebase** before making changes
- **Ask questions** to clarify ambiguous requirements
- **Design thoughtfully** before implementing
- **Review for quality** after building

This plugin embeds these practices into a structured workflow that runs automatically when you use the `/feature-dev` command. Agent teams enable parallel exploration, collaborative design debates, and cross-referenced code review — going beyond what a single agent session can achieve.

## Prerequisites

Agent teams must be enabled. Add this to your `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Command: `/feature-dev`

Launches a guided feature development workflow with 7 distinct phases.

**Usage:**
```bash
/feature-dev Add user authentication with OAuth
```

Or simply:
```bash
/feature-dev
```

The command will guide you through the entire process interactively.

## The 7-Phase Workflow

### Phase 1: Discovery & Team Setup

**Goal**: Understand what needs to be built and set up the agent team

**What happens:**
- Clarifies the feature request if it's unclear
- Asks what problem you're solving
- Identifies constraints and requirements
- Summarizes understanding and confirms with you
- Creates an agent team and a shared task list with tasks for all phases

**Example:**
```
You: /feature-dev Add caching
Claude: Let me understand what you need...
        - What should be cached? (API responses, computed values, etc.)
        - What are your performance requirements?
        - Do you have a preferred caching solution?

Creating team "feature-dev-caching" with task list...
```

### Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns through coordinated team exploration

**What happens:**
- Spawns 2-3 `code-explorer` teammates, each exploring a different aspect
- Explorers share discoveries with each other via direct messaging, avoiding duplicated work
- Each explorer returns comprehensive analysis with key files to read
- The lead reads all identified files to build deep understanding
- Explorers are shut down after completing their work
- Presents comprehensive summary of findings

**Teammates spawned:**
- "Find features similar to [feature] and trace implementation"
- "Map the architecture and abstractions for [area]"
- "Analyze current implementation of [related feature]"

**What's new with teams:** Explorers communicate directly. If one discovers that the codebase uses a specific pattern (e.g., repository pattern), they message the others so everyone factors it in — no duplicated discovery, faster convergence.

**Example output:**
```
explorer-similar found:
  Similar features:
  - User authentication (src/auth/): Uses JWT tokens, middleware pattern
  - Session management (src/session/): Redis-backed, 24hr expiry
  → Messaged explorer-arch: "Codebase uses middleware pattern consistently"

explorer-arch found:
  Architecture layers:
  - src/api/middleware/ — Rate limiting, CORS
  - src/services/ — Business logic layer
  - src/repositories/ — Data access layer

Key files to understand:
- src/auth/AuthService.ts:45 - Core authentication logic
- src/middleware/authMiddleware.ts:12 - Request authentication
- src/config/security.ts:8 - Security configuration
```

### Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities

**What happens:**
- The lead reviews exploration findings and feature request
- Identifies underspecified aspects:
  - Edge cases
  - Error handling
  - Integration points
  - Backward compatibility
  - Performance needs
- Presents all questions in an organized list
- **Waits for your answers before proceeding**

**Critical**: This phase ensures nothing is ambiguous before design begins.

**Example:**
```
Before designing the architecture, I need to clarify:

1. OAuth provider: Which OAuth providers? (Google, GitHub, custom?)
2. User data: Store OAuth tokens or just user profile?
3. Existing auth: Replace current auth or add alongside?
4. Sessions: Integrate with existing session management?
5. Error handling: How to handle OAuth failures?
```

### Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches through collaborative architecture work

**What happens:**
- Spawns 2-3 `code-architect` teammates with different focuses:
  - **Minimal changes**: Smallest change, maximum reuse
  - **Clean architecture**: Maintainability, elegant abstractions
  - **Pragmatic balance**: Speed + quality
- Architects challenge each other's designs via direct messaging
- The lead reviews all approaches and forms a recommendation
- Architects are shut down after completing their designs
- Presents comparison with trade-offs and recommendation
- **Asks which approach you prefer**

**What's new with teams:** Architects engage in constructive debate. If the "minimal" architect notices the "clean" approach introduces unnecessary complexity, they message each other to refine. The lead gets architectures that have already been stress-tested by peers.

**Example output:**
```
I've received 3 architecture blueprints from the team:

Approach 1: Minimal Changes (architect-minimal)
- Extend existing AuthService with OAuth methods
- Add new OAuth routes to existing auth router
- Minimal refactoring required
Pros: Fast, low risk
Cons: Couples OAuth to existing auth, harder to test

Approach 2: Clean Architecture (architect-clean)
- New OAuthService with dedicated interface
- Separate OAuth router and middleware
- Refactor AuthService to use common interface
Pros: Clean separation, testable, maintainable
Cons: More files, more refactoring
Note: architect-minimal challenged this for over-engineering,
      architect-clean defended with testability argument

Approach 3: Pragmatic Balance (architect-pragmatic)
- New OAuthProvider abstraction
- Integrate into existing AuthService
- Minimal refactoring, good boundaries
Pros: Balanced complexity and cleanliness
Cons: Some coupling remains

Recommendation: Approach 3 - gives you clean boundaries without
excessive refactoring, and fits your existing architecture well.

Which approach would you like to use?
```

### Phase 5: Implementation

**Goal**: Build the feature

**What happens:**
- **Waits for explicit approval** before starting
- The lead reads all relevant files identified in previous phases
- Implements following chosen architecture
- Follows codebase conventions strictly
- Writes clean, well-documented code
- Updates tasks as progress is made

**Notes:**
- Implementation only starts after you approve
- Follows patterns discovered in Phase 2
- Uses architecture designed in Phase 4
- Continuously tracks progress via the shared task list

### Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, and functionally correct through multi-perspective review

**What happens:**
- Spawns 3 `code-reviewer` teammates with different focuses:
  - **Simplicity/DRY/Elegance**: Code quality and maintainability
  - **Bugs/Correctness**: Functional correctness and logic errors
  - **Conventions/Abstractions**: Project standards and patterns
- Reviewers cross-reference each other's findings to reduce false positives
- The lead consolidates findings and identifies highest severity issues
- Reviewers are shut down after completing their reviews
- **Presents findings and asks what you want to do**:
  - Fix now
  - Fix later
  - Proceed as-is
- Addresses issues based on your decision

**What's new with teams:** Reviewers validate each other. If the "bugs" reviewer flags a potential null reference, the "conventions" reviewer can confirm or challenge it. This reduces false positives and strengthens high-confidence findings.

**Example output:**
```
Code Review Results (consolidated from 3 reviewers):

High Priority Issues:
1. Missing error handling in OAuth callback (src/auth/oauth.ts:67)
   — Found by reviewer-bugs, confirmed by reviewer-conventions
2. Memory leak: OAuth state not cleaned up (src/auth/oauth.ts:89)
   — Found by reviewer-bugs

Medium Priority:
1. Could simplify token refresh logic (src/auth/oauth.ts:120)
   — Found by reviewer-quality
2. Consider extracting OAuth config validation
   — Found by reviewer-conventions

All tests pass. Code follows project conventions.

What would you like to do?
```

### Phase 7: Summary & Cleanup

**Goal**: Document what was accomplished and clean up team resources

**What happens:**
- Marks all tasks complete
- Summarizes:
  - What was built
  - Key decisions made
  - Files modified
  - Suggested next steps
- Cleans up the agent team and shared task list

**Example:**
```
Feature Complete: OAuth Authentication

What was built:
- OAuth provider abstraction supporting Google and GitHub
- OAuth routes and middleware integrated with existing auth
- Token refresh and session integration
- Error handling for all OAuth flows

Key decisions:
- Used pragmatic approach with OAuthProvider abstraction
- Integrated with existing session management
- Added OAuth state to prevent CSRF

Files modified:
- src/auth/OAuthProvider.ts (new)
- src/auth/AuthService.ts
- src/routes/auth.ts
- src/middleware/authMiddleware.ts

Suggested next steps:
- Add tests for OAuth flows
- Add more OAuth providers (Microsoft, Apple)
- Update documentation
```

## Agents

### `code-explorer`

**Purpose**: Deeply analyzes existing codebase features by tracing execution paths

**Focus areas:**
- Entry points and call chains
- Data flow and transformations
- Architecture layers and patterns
- Dependencies and integrations
- Implementation details

**Team behavior:** Shares discoveries with fellow explorers, claims tasks from the shared task list, marks tasks complete when done.

**Output:**
- Entry points with file:line references
- Step-by-step execution flow
- Key components and responsibilities
- Architecture insights
- List of essential files to read

### `code-architect`

**Purpose**: Designs feature architectures and implementation blueprints

**Focus areas:**
- Codebase pattern analysis
- Architecture decisions
- Component design
- Implementation roadmap
- Data flow and build sequence

**Team behavior:** Challenges other architects' designs, shares critical patterns discovered, engages in constructive debate to produce stronger architectures.

**Output:**
- Patterns and conventions found
- Architecture decision with rationale
- Complete component design
- Implementation map with specific files
- Build sequence with phases

### `code-reviewer`

**Purpose**: Reviews code for bugs, quality issues, and project conventions

**Focus areas:**
- Project guideline compliance (CLAUDE.md)
- Bug detection
- Code quality issues
- Confidence-based filtering (only reports high-confidence issues >= 80)

**Team behavior:** Cross-references findings with other reviewers, avoids duplicate reports, confirms or challenges others' findings to reduce false positives.

**Output:**
- Critical issues (confidence 75-100)
- Important issues (confidence 50-74)
- Specific fixes with file:line references
- Project guideline references

## Usage Patterns

### Full workflow (recommended for new features):
```bash
/feature-dev Add rate limiting to API endpoints
```

Let the workflow guide you through all 7 phases with coordinated team exploration.

### Manual agent invocation:

**Explore a feature:**
```
"Launch code-explorer to trace how authentication works"
```

**Design architecture:**
```
"Launch code-architect to design the caching layer"
```

**Review code:**
```
"Launch code-reviewer to check my recent changes"
```

## Best Practices

1. **Use the full workflow for complex features**: The 7 phases with agent teams ensure thorough planning
2. **Answer clarifying questions thoughtfully**: Phase 3 prevents future confusion
3. **Choose architecture deliberately**: Phase 4 gives you peer-reviewed options for a reason
4. **Don't skip code review**: Phase 6 catches issues with cross-validated findings
5. **Read the suggested files**: Phase 2 identifies key files — read them to understand context
6. **Let teammates finish**: The lead should wait for teammates to complete before synthesizing. Use delegate mode (Shift+Tab) during team phases to prevent the lead from implementing prematurely.

## When to Use This Plugin

**Use for:**
- New features that touch multiple files
- Features requiring architectural decisions
- Complex integrations with existing code
- Features where requirements are somewhat unclear

**Don't use for:**
- Single-line bug fixes
- Trivial changes
- Well-defined, simple tasks
- Urgent hotfixes

## Requirements

- Claude Code installed
- Agent teams enabled (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)
- Git repository (for code review)
- Project with existing codebase (workflow assumes existing code to learn from)

## Troubleshooting

### Agents take too long

**Issue**: Exploration or architecture teammates are slow

**Solution**:
- This is normal for large codebases
- Teammates run in parallel when possible
- The thoroughness pays off in better understanding

### Too many clarifying questions

**Issue**: Phase 3 asks too many questions

**Solution**:
- Be more specific in your initial feature request
- Provide context about constraints upfront
- Say "whatever you think is best" if truly no preference

### Architecture options overwhelming

**Issue**: Too many architecture options in Phase 4

**Solution**:
- Trust the recommendation — it's based on peer-reviewed codebase analysis
- If still unsure, ask for more explanation
- Pick the pragmatic option when in doubt

### Lead implementing instead of delegating

**Issue**: The lead starts doing exploration or review work itself

**Solution**:
- Tell the lead to wait for teammates
- Use delegate mode (Shift+Tab) to restrict the lead to coordination-only tools

### Teammates not communicating

**Issue**: Teammates work in isolation without sharing findings

**Solution**:
- Ensure spawn prompts explicitly instruct teammates to use SendMessage
- Check that agent definitions include SendMessage in their tools list

## Tips

- **Be specific in your feature request**: More detail = fewer clarifying questions
- **Trust the process**: Each phase builds on the previous one
- **Review teammate outputs**: Teammates provide valuable insights about your codebase
- **Don't skip phases**: Each phase serves a purpose
- **Use for learning**: The exploration phase teaches you about your own codebase
- **Use delegate mode**: During Phases 2, 4, and 6, delegate mode keeps the lead focused on coordination

## Author

Sid Bidasaria (sbidasaria@anthropic.com)

## Version

2.0.0
