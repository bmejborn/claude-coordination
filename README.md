# Claude Agent Coordination

Async coordination system for multiple Claude instances working together across machines.

## Agents

- **Botsy** - Telegram bot running on Mac (`@BotsyMcBotFace_bot`)
- **Clawsy** - Desktop Claude Code in container on separate machine (`@ClawsyMcClawFace_bot`)

## How It Works

Both agents clone this repository to their respective machines. Coordination happens through git:

1. Agent receives work from user
2. Agent does the work
3. Agent commits results to `handoffs/`
4. Agent optionally creates task JSON in `tasks/pending/` for handoff
5. Agent commits and pushes
6. Other agent pulls, sees task, picks it up
7. Repeat

## Directory Structure

```
claude-coordination/
├── tasks/
│   ├── pending/          # Tasks waiting to be picked up
│   ├── in-progress/      # Currently being worked on
│   └── completed/        # Done (archived after 7 days)
├── status/
│   ├── botsy-status.json
│   └── clawsy-status.json
├── handoffs/
│   ├── botsy-to-clawsy/    # Research, analysis from Botsy
│   └── clawsy-to-botsy/    # Build logs, results from Clawsy
└── events/
    └── coordination-log.jsonl  # Append-only event log
```

## Quick Start

### Setup (each agent)

```bash
# Clone this repo
git clone https://github.com/bmejborn/claude-coordination.git
cd claude-coordination
```

### Check for tasks

```bash
git pull --rebase origin main
ls -1 tasks/pending/*.json
```

### View current status

```bash
cat status/botsy-status.json
cat status/clawsy-status.json
```

### Create a task

```bash
TASK_ID="task-$(date +%s)"

cat > tasks/pending/${TASK_ID}.json << EOF
{
  "id": "${TASK_ID}",
  "from": "botsy",
  "to": "clawsy",
  "type": "implement",
  "priority": "high",
  "description": "Build auth screen",
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF

git add tasks/pending/${TASK_ID}.json
git commit -m "Botsy: Created ${TASK_ID}"
git push origin main
```

## Task Format

```json
{
  "id": "task-1707409234",
  "from": "botsy|clawsy|user",
  "to": "botsy|clawsy",
  "type": "research|implement|test|review|deploy|build",
  "priority": "low|medium|high|urgent",
  "description": "Human-readable task description",
  "context_file": "handoffs/botsy-to-clawsy/context.md (optional)",
  "created_at": "2026-02-08T14:50:00Z",
  "started_at": "2026-02-08T15:00:00Z (when moved to in-progress)",
  "completed_at": "2026-02-08T15:30:00Z (when done)",
  "result": {
    "status": "success|failed|partial",
    "message": "What was done",
    "artifacts": ["handoffs/clawsy-to-botsy/result.md"]
  }
}
```

## Status Format

```json
{
  "agent": "botsy",
  "state": "idle|working|waiting",
  "current_task": "task-1707409234",
  "last_activity": "2026-02-08T14:50:00Z",
  "capabilities": ["research", "code-review", "telegram"],
  "location": "telegram-bot-mac",
  "working_directory": "/Users/bustermejborn/Desktop/Apps/",
  "health": "ok|degraded|error",
  "message": "Optional status message"
}
```

## Workflow Example

**User**: "@Botsy research the auth flow in the golf app"

**Botsy**:
```bash
# Does research, writes findings
git pull --rebase origin main
echo "Auth research..." > handoffs/botsy-to-clawsy/auth-research.md

# Creates task for Clawsy
cat > tasks/pending/task-1707410000.json << EOF
{
  "id": "task-1707410000",
  "from": "botsy",
  "to": "clawsy",
  "type": "implement",
  "description": "Implement auth based on research",
  "context_file": "handoffs/botsy-to-clawsy/auth-research.md",
  "created_at": "2026-02-08T15:00:00Z"
}
EOF

git add .
git commit -m "Botsy: Auth research complete, task for Clawsy"
git push origin main
```

**User**: "@Clawsy check for tasks"

**Clawsy**:
```bash
git pull --rebase origin main
cat tasks/pending/task-1707410000.json
# Sees the task

# Moves to in-progress
mv tasks/pending/task-1707410000.json tasks/in-progress/
# Updates task with started_at timestamp

# Reads context
cat handoffs/botsy-to-clawsy/auth-research.md

# Does implementation work...

# Writes result
echo "Implementation complete..." > handoffs/clawsy-to-botsy/auth-impl-result.md

# Moves to completed
mv tasks/in-progress/task-1707410000.json tasks/completed/
# Updates task with completion data

git add .
git commit -m "Clawsy: Auth implementation complete"
git push origin main
```

## Conflict Resolution

Always use:
```bash
git pull --rebase origin main
```

- **Status files**: Last write wins
- **Task moves**: Append-only operations (no conflicts)
- **Handoffs**: Each agent writes to their own directory
- **Events log**: JSONL format (append-only, no conflicts)

## Guidelines

- ✅ Commit often, keep messages clear
- ✅ Use descriptive task descriptions
- ✅ Update your status file when state changes
- ✅ Archive completed tasks older than 7 days
- ✅ Write detailed handoff documents
- ❌ Don't edit other agent's handoff files
- ❌ Don't force-push (breaks coordination)
- ❌ Don't merge - always rebase

## Event Log

Append events in JSONL format to `events/coordination-log.jsonl`:

```jsonl
{"timestamp":"2026-02-08T14:50:00Z","agent":"botsy","event":"task_created","task_id":"task-1707410000"}
{"timestamp":"2026-02-08T15:00:00Z","agent":"clawsy","event":"task_started","task_id":"task-1707410000"}
{"timestamp":"2026-02-08T15:30:00Z","agent":"clawsy","event":"task_completed","task_id":"task-1707410000","status":"success"}
```

## Future Enhancements

- [ ] Auto-polling (agents check for tasks every N minutes)
- [ ] Task priorities with escalation
- [ ] GitHub webhooks for instant notifications
- [ ] Web dashboard for coordination state
- [ ] Task templates for common workflows
- [ ] Metrics (completion time, success rate)
