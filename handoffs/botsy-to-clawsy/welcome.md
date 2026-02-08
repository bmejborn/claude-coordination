# Welcome to Claude Coordination! 👋

**From:** Botsy (@BotsyMcBotFace_bot)
**To:** Clawsy (@ClawsyMcClawFace_bot)
**Date:** 2026-02-08

## What Is This?

This is our **dedicated coordination repository** - separate from any specific project. Both of us can clone it independently and use it to hand off work, create tasks, and stay in sync across machines!

## The Setup

- **You're in**: A container on a separate machine running desktop Claude Code
- **I'm in**: Mac at `/Users/bustermejborn/Desktop/Apps/` running as a Telegram bot

We both clone this repo, and coordinate through git commits!

## How It Works

### 1. **Tasks** (in `tasks/`)
When one of us needs the other to do something:
- Create a JSON file in `tasks/pending/`
- Other agent pulls, sees it, moves to `in-progress/`
- Completes the work
- Moves to `completed/` with results
- Commits and pushes

### 2. **Handoffs** (in `handoffs/`)
When we need to share context or work products:
- I write to `handoffs/botsy-to-clawsy/` (research, analysis, findings)
- You write to `handoffs/clawsy-to-botsy/` (build results, logs, outputs)
- We reference these in task `context_file` fields

### 3. **Status** (in `status/`)
Each of us maintains our current state:
- Update `status/{agent}-status.json` when things change
- Shows: state (idle/working), current task, health, capabilities
- Helps us know what the other is up to

### 4. **Events** (in `events/`)
Optional append-only log for tracking:
- Task created/started/completed events
- JSONL format (one JSON object per line)
- Never conflicts because it's append-only

## Your First Task!

There's an example task waiting in `tasks/pending/example-task-1707410100.json`

It asks you to:
1. Update your status file with your real info
2. Practice moving the task through the workflow
3. Commit and push

Try it out! 🚀

## Example Workflow

**User tells me**: "@Botsy research how auth works in the golf app"

**I do research**, then:
```bash
cd /path/to/claude-coordination
git pull --rebase origin main

# Write findings
echo "Auth uses Supabase..." > handoffs/botsy-to-clawsy/golf-auth-research.md

# Create task for you
cat > tasks/pending/task-$(date +%s).json << EOF
{
  "id": "task-1707410200",
  "from": "botsy",
  "to": "clawsy",
  "type": "review",
  "description": "Review auth implementation in golf app",
  "context_file": "handoffs/botsy-to-clawsy/golf-auth-research.md",
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF

git add .
git commit -m "Botsy: Golf auth research complete, task for Clawsy"
git push origin main
```

**User tells you**: "@Clawsy check for tasks"

**You pull and work**:
```bash
cd /path/to/claude-coordination
git pull --rebase origin main

# See the task
ls tasks/pending/

# Move to in-progress
mv tasks/pending/task-1707410200.json tasks/in-progress/

# Read my research
cat handoffs/botsy-to-clawsy/golf-auth-research.md

# Do your review work...

# Write results
echo "Review complete..." > handoffs/clawsy-to-botsy/golf-auth-review.md

# Update task with results
# (edit JSON to add completion timestamp and result field)

# Move to completed
mv tasks/in-progress/task-1707410200.json tasks/completed/

git add .
git commit -m "Clawsy: Golf auth review complete"
git push origin main
```

## My Capabilities

I can help with:
- 🔍 Research (reading code, analyzing patterns)
- 📝 Code review
- 📁 File operations
- 💬 Answering user questions via Telegram
- 📋 Creating tasks for you

## Your Capabilities

You can help with:
- 🔨 Implementation (writing code)
- 🏗️ Building apps
- 🧪 Running tests
- 🚀 Deployment
- 💻 Complex dev workflows

## Communication

Since we can't see each other's Telegram messages:
- **User relays messages** between us
- **Git commits** carry our work
- **Handoff files** explain context
- **Task descriptions** make intentions clear

## Questions?

Ask the user to check this welcome file and the example task. Once you complete the example task, we'll know the system works end-to-end!

Looking forward to collaborating! 🤖🤝🤖

— Botsy
