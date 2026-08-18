## Claude Code Multi‑Agent Workflows

Two ready‑to‑use patterns: one for simple projects (lead = enforcer), one for complex projects (dedicated enforcers + short‑lived agents to reduce drift).

***

## Pattern A — Simple Project (Lead Agent as Enforcer)

**Goal:** Minimal overhead, single session, lead agent handles architecture, tasking, implementation review, and merge.

### 1. Setup

- Create `claude.md` in the project root:

  ```md
  # Role
  You are the lead agent: architect, coordinator, and enforcer.

  # Rules
  - Do not write implementation code without explicit user approval.
  - Always propose:
    - Architecture overview
    - Module breakdown
    - Task plan (including subagents)
    Then stop and wait for approval.
  - Enforce architecture and contracts on all implementation.
  - Reject or request fixes for any work that violates:
    - Architecture rules
    - Module contracts
    - Naming / JSON / style rules
  ```

- Optionally add:
  - `ARCHITECTURE.md` (global structure, principles).
  - `CONTRACTS.md` (capability pattern, business logic rules, JSON contract, naming).

### 2. Session Start

In VS Code, open a single Claude Code session and say:

> “Read `claude.md`, `ARCHITECTURE.md`, and `CONTRACTS.md`. For this project, always follow them. Now propose an architecture and a subagent task plan for a simple REST API, then stop and wait for my approval.”

### 3. Task Plan Structure

Lead agent proposes something like:

- **Task 1:** Define capabilities (interfaces only).
- **Task 2:** Implement `Transformation` capability MVP.
- **Task 3:** Implement business logic layer.
- **Task 4:** Wire up `app.py` / `main.py`.
- **Task 5:** Basic manual tests / smoke checks.

For each task, it specifies:

- Goal.
- Inputs (which docs/contracts to use).
- Expected outputs (files, interfaces).
- Whether a subagent should be used.

You approve or adjust.

### 4. Execution Loop (Per Task)

For each approved task:

1. **Lead spawns a subagent** (via Task tool) with:
   - Task description.
   - Relevant contracts (e.g., “implement `Transformation` capability following `CONTRACTS.md`”).
2. **Subagent implements** the module in isolation.
3. **Subagent returns** a summary + list of changed files.
4. **Lead reviews**:
   - Architecture compliance.
   - Contract compliance.
   - Naming / JSON / style.
5. If OK → lead merges and moves to next task.  
   If not OK → lead sends it back with specific fixes.

No separate tabs needed; everything happens in one session with background subagents. 

### 5. When to Use This Pattern

- Small to medium projects.
- You want simplicity over maximal parallelism.
- One “brain” (the lead) can comfortably hold the whole design.

***

## Pattern B — Complex Project (Dedicated Enforcers + Short‑Lived Agents)

**Goal:** Reduce hallucination and drift on large/long projects by:

- Separating roles (architect, implementer, reviewer, scribe).
- Using short‑lived agents per task.
- Externalizing memory in log files.

### 1. Setup Files

Create in the project root:

- `claude.md` (global rules for all agents):

  ```md
  # Global Rules
  - No implementation code without explicit user approval of the task plan.
  - Always follow `ARCHITECTURE.md` and `CONTRACTS.md`.
  - Each task must:
    - Be scoped to a single module or concern.
    - Produce a short changelog entry in `AGENT_LOG.md`.
    - Stop if it hits an architecture or contract violation.
  ```

- `ARCHITECTURE.md` (global structure, principles, module map).
- `CONTRACTS.md` (capability pattern, business logic rules, JSON, naming).
- `AGENT_LOG.md` (progress + decisions + errors):

  ```md
  # Progress
  - [ ] Task 1: Transformation capability – pending
  - [ ] Task 2: Business logic – pending
  ...

  # Decisions
  - [Date] Decision: X architecture choice, reason Y.

  # Errors
  - [Date] Task 2: Violation of capability interface in file Z – blocked until fixed.
  ```

### 2. Roles

Define these logical roles (you don’t need separate tabs; they’re roles the lead assigns):

- **Primary Lead (Architect/Coordinator)**
  - Owns:
    - `ARCHITECTURE.md`, `CONTRACTS.md`.
    - Task graph.
    - `AGENT_LOG.md` (or delegates writes).
  - Responsibilities:
    - Break work into tasks.
    - Spawn subagents for each task.
    - Ensure tasks respect architecture and contracts.
    - Decide what to do when errors are reported.

- **Implementer Subagents** (short‑lived, per task)
  - Receive:
    - Task description.
    - Relevant slice of architecture/contracts.
  - Implement one module/capability.
  - Self‑check against contracts before returning.

- **Enforcer Subagents** (reviewers)
  - One or more, depending on complexity:
    - **Architecture/Contract Reviewer**
    - **Code Quality / Style Reviewer**
    - **Tests / Verification Reviewer** (optional)
  - Responsibilities:
    - Review implementer output.
    - Score or flag:
      - Architecture violations.
      - Contract violations.
      - Style/naming issues.
      - Test gaps.
    - Return a structured report (pass/fail + issues).

- **Scribe Subagent** (optional, or part of lead)
  - Updates `AGENT_LOG.md`:
    - Marks tasks as done/pending/blocked.
    - Records key decisions.
    - Logs errors that block progress.

### 3. Session Start

In VS Code, open a single Claude Code session and say:

> “Read `claude.md`, `ARCHITECTURE.md`, `CONTRACTS.md`, and `AGENT_LOG.md`. You are the primary lead. Propose:
> - A high‑level architecture for a REST API.
> - A task breakdown into independent modules.
> - A multi‑agent workflow using:
>   - Implementer subagents per task.
>   - At least one enforcer subagent for architecture/contract review.
>   - A scribe role to update `AGENT_LOG.md`.
> Then stop and wait for my approval.”

### 4. Task Execution Loop (Per Task)

For each task in the plan:

1. **Lead spawns an Implementer subagent**:
   - Input: task goal + relevant contracts.
   - Constraint: “Do not touch other modules. If you detect a conflict with architecture, stop and report.”

2. **Implementer**:
   - Implements the module.
   - Does a quick self‑review against contracts.
   - Returns:
     - Summary of changes.
     - List of files.
     - Any open questions.

3. **Lead spawns Enforcer subagent(s)**:
   - One for architecture/contracts.
   - Optionally one for code quality/tests.
   - Each gets:
     - Changed files.
     - Relevant sections of `ARCHITECTURE.md` / `CONTRACTS.md`.

4. **Enforcers**:
   - Review the implementation.
   - Return:
     - Pass/fail.
     - List of issues (with severity).
     - Suggested fixes.

5. **Lead decides**:
   - If pass → mark task as done, ask scribe to update `AGENT_LOG.md`.
   - If fail → send back to implementer with specific fixes, or create a new “fix” task.

6. **Scribe**:
   - Updates `AGENT_LOG.md`:
     - Task status.
     - Key decisions.
     - Any blocking errors.

7. **Lead checks `AGENT_LOG.md`** before assigning new work:
   - If there are blocking errors, it creates fix tasks first.
   - Otherwise, it proceeds to the next pending task.

### 5. Short‑Lived Agents to Reduce Hallucination

To limit drift:

- **Implementer and enforcer subagents are task‑scoped and disposable**:
  - They exist only for one task.
  - After the task is done, their context is discarded.
- **Long‑term memory lives in files**:
  - `ARCHITECTURE.md`
  - `CONTRACTS.md`
  - `AGENT_LOG.md`
- The **primary lead** persists across tasks but:
  - Doesn’t keep every detail in its head; it reads from logs and docs.
  - Focuses on coordination and high‑level decisions.

This matches the “temporary sessions + log file” idea you described: the system’s state is externalized, so no single agent needs a huge, long‑running context. [claudemarketplaces](https://claudemarketplaces.com/skills/obra/superpowers/subagent-driven-development)

### 6. When to Use This Pattern

- Larger codebases or long‑running projects.
- You expect many modules, evolving requirements, or multiple contributors.
- You want:
  - Stronger enforcement of architecture.
  - Clear audit trail (`AGENT_LOG.md`).
  - Less hallucination and drift over time.

***

## Choosing Between Patterns

- Use **Pattern A** when:
  - The project is small/medium.
  - You’re comfortable with one “brain” holding most of the design.
- Use **Pattern B** when:
  - The project is large or long‑term.
  - You want strict separation of roles and an explicit log of decisions/errors.
  - You care about reducing hallucination and maintaining a clean architectural history.

You can start with Pattern A and evolve into Pattern B as the project grows; the file contracts (`ARCHITECTURE.md`, `CONTRACTS.md`, `AGENT_LOG.md`) are reusable in both.
