# Ralphie — Parallel Task Swarm from Reeves

Pull tasks from a Reeves project, analyze file dependencies, spawn parallel background agents, and iterate each until done. You are the foreman running a crew.

## Arguments

`/ralphie <PROJECT_NAME_OR_ID> [--max N] [--filter TAG] [--dry-run]`

- **PROJECT_NAME_OR_ID** — Reeves project to pull tasks from (e.g., "Reeves 3.0" or project ID)
- **--max N** — Max tasks to run in parallel (default: 5, max: 10)
- **--filter TAG** — Only run tasks with this tag
- **--dry-run** — Show the execution plan without spawning agents

If no project is specified, use the most recently created active project.

---

## Execution Flow

### Phase 1: Gather Tasks

1. Call `reeves_project_list` to find the project by name or ID
2. Call `reeves_task_list` filtered to that project
3. Filter to enabled, non-completed tasks
4. Sort by priority (high first), then by ID (lower first)
5. If `--filter` is set, only include tasks with matching tag
6. Cap at `--max` tasks (default 5)

Display the task lineup:
```
RALPHIE SWARM — Project: "Reeves 3.0"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Slot 1: #4370 [HIGH] Build live operation cards near the orb
         Files: orb.js, base.html, style.css
Slot 2: #4373 [HIGH] Cross-session voice memory
         Files: voice.py
Slot 3: #4372 [HIGH] Inject intents/facts/PEFM into voice prompt
         Files: voice.py
...

Conflict check: Slots 2 & 3 both touch voice.py — SEQUENCING
Safe parallel groups: [1] [2→3] [4] [5]
```

### Phase 2: Dependency Analysis

For each task, extract the likely files it will modify from the task instructions.

Rules:
- Tasks touching the SAME file cannot run in parallel — sequence them
- Tasks touching DIFFERENT files run in parallel
- Group conflicting tasks into sequential chains
- Each chain runs as one agent that handles tasks in order

Display the execution plan. If `--dry-run`, stop here.

### Phase 3: Spawn Agents

For each parallel slot (task or chain), spawn a background Task agent:

```
Task tool:
  subagent_type: "general-purpose"
  run_in_background: true
  prompt: <see Agent Prompt Template below>
```

Launch ALL independent slots in a SINGLE message (parallel tool calls).

Track each agent's output file path for monitoring.

### Phase 4: Monitor & Report

After spawning all agents:
1. Wait 30 seconds
2. Check each agent's output file (Read tool) for progress
3. Report status of each slot:
   ```
   RALPHIE STATUS
   ━━━━━━━━━━━━━━
   Slot 1: RUNNING — "Created CardManager class, working on CSS..."
   Slot 2: DONE — "Added cross-session memory, 3 iterations"
   Slot 3: WAITING — Blocked on Slot 2 (voice.py conflict)
   Slot 4: RUNNING — "Implementing ambient state endpoint..."
   Slot 5: DONE — "ARIA live region added"
   ```
4. Repeat every 60 seconds until all slots complete or 30 minutes elapsed
5. For completed agents, mark the Reeves task as complete via `reeves_task_complete`
6. Devlog the full swarm results

### Phase 5: Summary

When all agents finish:
```
RALPHIE COMPLETE
━━━━━━━━━━━━━━━━
Project: Reeves 3.0
Duration: 14 minutes
Tasks completed: 4/5
Tasks failed: 1 (Slot 3 — voice.py conflict with Slot 2 changes)

Completed:
  #4370 Build live operation cards — 3 iterations, orb.js +180 lines
  #4373 Cross-session voice memory — 2 iterations, voice.py modified
  #4374 Ambient orb signals — 4 iterations, new endpoint + orb.js
  #4380 ARIA accessibility — 1 iteration, base.html + orb.js

Failed:
  #4372 Inject intents/PEFM — merge conflict with #4373 changes

Next: Run tests with `pytest tests/ -q`, then resolve conflicts manually.
```

Devlog the summary with tags: `ralphie`, `swarm`, `<project-name>`.

---

## Agent Prompt Template

Each background agent receives this prompt:

```
You are a focused engineering agent working on ONE task from the Reeves project.

## Your Task
Name: {task_name}
ID: {task_id}
Priority: {task_priority}
Instructions:
{task_instructions}

## Project Context
Project: {project_name}
Working directory: /Users/dshanklinbv/repos-personal/reeves-web

## Rules
1. Read the relevant files FIRST before making any changes
2. Make the minimum changes needed to complete the task
3. Follow existing code patterns and conventions
4. Do NOT modify files outside the scope of your task
5. Do NOT run the full test suite — just verify your changes don't have syntax errors
6. Do NOT commit or push — just make the code changes
7. When done, output a brief summary of what you changed and why

## Iteration
If your first attempt has issues, review what you did, identify the problem,
and fix it. You have up to 3 iterations to get it right.

## Completion
When the task is fully implemented, output:
RALPHIE_DONE: {task_id} — {brief summary of changes}

## Files you may touch:
{predicted_files}

## Files you must NOT touch (other agents are working on them):
{excluded_files}
```

---

## Conflict Resolution Strategy

When two tasks touch the same file:
1. Run them SEQUENTIALLY in one agent (Task 1 first, then Task 2)
2. The agent reads the file between tasks to see previous changes
3. This is the Ralph Loop insight — seeing your own previous work

When a sequential chain fails:
1. Mark the failed task, complete the successful ones
2. Report the conflict in the summary
3. User resolves manually or runs `/ralphie` again with just the failed tasks

---

## Integration Points

- **Task source**: `reeves_task_list` (Reeves MCP)
- **Task completion**: `reeves_task_complete` (Reeves MCP)
- **Progress tracking**: `devlog_add` with tags `ralphie`, `swarm`
- **File analysis**: Grep/Read to extract file paths from task instructions
- **Agent execution**: Task tool with `run_in_background: true`

---

## Error Handling

- If Reeves MCP is unavailable: fall back to reading task IDs from arguments
- If a task has no clear file targets: run it solo (no parallelism risk)
- If an agent crashes: mark as failed, continue others
- If max runtime (30 min) exceeded: kill remaining agents, report partial results
- If no tasks found: report "No actionable tasks in project"

---

## Example Usage

```
/ralphie "Reeves 3.0"
# Pulls all high-priority tasks, analyzes conflicts, spawns 5 agents

/ralphie "Reeves 3.0" --max 3 --filter pillar-1
# Only pillar-1 tagged tasks, max 3 parallel

/ralphie "Reeves 3.0" --dry-run
# Show the plan without executing

/ralphie 4585
# Use project ID directly
```
