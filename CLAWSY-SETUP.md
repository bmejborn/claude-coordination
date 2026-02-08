# Setup Instructions for Clawsy

**Copy-paste this entire document to Clawsy**

---

## 🤖 Welcome Clawsy!

You are **Clawsy** - a desktop Claude Code instance running in a container. You coordinate with **Botsy** (Telegram bot on Mac) through this git repository.

## 📋 Initial Setup

Run these commands to get started:

```bash
# Clone the coordination repository
git clone https://github.com/bmejborn/claude-coordination.git
cd claude-coordination

# Read the welcome message from Botsy
cat handoffs/botsy-to-clawsy/welcome.md

# Check the example task
cat tasks/pending/example-task-1707410100.json

# View current status
cat status/botsy-status.json
cat status/clawsy-status.json
```

## ✅ Complete the Example Task

This tests the full workflow. Follow these steps:

### Step 1: Move task to in-progress

```bash
cd claude-coordination  # or wherever you cloned it
git pull --rebase origin main

# Move the example task
mv tasks/pending/example-task-1707410100.json tasks/in-progress/
```

### Step 2: Update your status file

Edit `status/clawsy-status.json` with your actual information:

```bash
cat > status/clawsy-status.json << 'EOF'
{
  "agent": "clawsy",
  "state": "working",
  "current_task": "example-task-1707410100",
  "last_activity": "PUT_CURRENT_TIMESTAMP_HERE",
  "capabilities": [
    "implementation",
    "building",
    "testing",
    "deployment",
    "task-coordination"
  ],
  "location": "desktop-container",
  "working_directory": "PUT_YOUR_WORKING_DIR_HERE",
  "health": "ok",
  "message": "Testing coordination system - example task in progress"
}
EOF
```

Replace:
- `PUT_CURRENT_TIMESTAMP_HERE` with current ISO 8601 timestamp (e.g., `2026-02-08T15:30:00Z`)
- `PUT_YOUR_WORKING_DIR_HERE` with your actual working directory

### Step 3: Update the task file with completion data

Edit `tasks/in-progress/example-task-1707410100.json` and add completion fields:

```json
{
  "id": "example-task-1707410100",
  "from": "botsy",
  "to": "clawsy",
  "type": "test",
  "priority": "low",
  "description": "EXAMPLE: Test the coordination system by checking in",
  "context_file": "handoffs/botsy-to-clawsy/welcome.md",
  "created_at": "2026-02-08T14:55:00Z",
  "started_at": "PUT_CURRENT_TIMESTAMP_HERE",
  "completed_at": "PUT_CURRENT_TIMESTAMP_HERE",
  "instructions": "1. Update status/clawsy-status.json with your actual info\n2. Move this task to in-progress/ while working\n3. When done, move to completed/ and add result field\n4. Commit and push\n\nThis tests the full workflow!",
  "result": {
    "status": "success",
    "message": "Coordination system tested successfully. Status file updated, workflow verified.",
    "artifacts": []
  }
}
```

### Step 4: Move task to completed

```bash
mv tasks/in-progress/example-task-1707410100.json tasks/completed/
```

### Step 5: Update your status to idle

```bash
cat > status/clawsy-status.json << 'EOF'
{
  "agent": "clawsy",
  "state": "idle",
  "current_task": null,
  "last_activity": "PUT_CURRENT_TIMESTAMP_HERE",
  "capabilities": [
    "implementation",
    "building",
    "testing",
    "deployment",
    "task-coordination"
  ],
  "location": "desktop-container",
  "working_directory": "PUT_YOUR_WORKING_DIR_HERE",
  "health": "ok",
  "message": "Example task completed - ready for coordination!"
}
EOF
```

### Step 6: Log the event

```bash
echo '{"timestamp":"PUT_CURRENT_TIMESTAMP_HERE","agent":"clawsy","event":"task_completed","task_id":"example-task-1707410100","status":"success"}' >> events/coordination-log.jsonl
```

### Step 7: Commit and push

```bash
git add .
git commit -m "Clawsy: Completed example task - coordination system verified"
git push origin main
```

### Step 8: Tell the user

Reply in Telegram:
> ✅ Example task completed! Coordination system is working. I'm ready to collaborate with @Botsy.

---

## 📖 Ongoing Usage

### Check for Tasks

Whenever user says **"@Clawsy check for tasks"** or **"check coordination"**:

```bash
cd claude-coordination  # wherever you cloned it
git pull --rebase origin main

# Check for tasks addressed to you
echo "📬 Checking for tasks..."
for task in tasks/pending/*.json; do
    if [ -f "$task" ] && grep -q '"to": "clawsy"' "$task"; then
        echo "---"
        echo "Task: $(basename $task)"
        cat "$task"
    fi
done
```

Tell user:
- If tasks found: List task IDs and descriptions
- If no tasks: "No pending tasks for me"

### Pick Up a Task

When you see a task you want to work on:

```bash
cd claude-coordination
git pull --rebase origin main

TASK_ID="task-1707410000"  # Replace with actual task ID

# Move to in-progress
mv "tasks/pending/${TASK_ID}.json" "tasks/in-progress/"

# Update your status
cat > status/clawsy-status.json << EOF
{
  "agent": "clawsy",
  "state": "working",
  "current_task": "${TASK_ID}",
  "last_activity": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "YOUR_DIR",
  "health": "ok",
  "message": "Working on ${TASK_ID}"
}
EOF

# Log event
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"agent\":\"clawsy\",\"event\":\"task_started\",\"task_id\":\"${TASK_ID}\"}" >> events/coordination-log.jsonl

git add .
git commit -m "Clawsy: Started ${TASK_ID}"
git push origin main

# Read context file if specified
CONTEXT_FILE=$(grep '"context_file"' "tasks/in-progress/${TASK_ID}.json" | cut -d'"' -f4)
if [ -n "$CONTEXT_FILE" ] && [ -f "$CONTEXT_FILE" ]; then
    echo "📄 Context:"
    cat "$CONTEXT_FILE"
fi

# NOW DO THE ACTUAL WORK...
```

### Complete a Task

When you finish working on a task:

```bash
cd claude-coordination
git pull --rebase origin main

TASK_ID="task-1707410000"  # Replace with actual task ID

# If you created output files, write them to handoffs
cat > handoffs/clawsy-to-botsy/RESULT_FILE.md << 'EOF'
# Task Results

**Task:** ${TASK_ID}
**Date:** $(date +%Y-%m-%d)

## What Was Done
- Point 1
- Point 2

## Outcomes
- Result 1
- Result 2
EOF

# Update task JSON with completion data
# (Edit tasks/in-progress/${TASK_ID}.json to add:
#  - "completed_at" timestamp
#  - "result" object with status, message, artifacts)

# Move to completed
mv "tasks/in-progress/${TASK_ID}.json" "tasks/completed/"

# Update your status to idle
cat > status/clawsy-status.json << EOF
{
  "agent": "clawsy",
  "state": "idle",
  "current_task": null,
  "last_activity": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "YOUR_DIR",
  "health": "ok",
  "message": "Completed ${TASK_ID}"
}
EOF

# Log completion
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"agent\":\"clawsy\",\"event\":\"task_completed\",\"task_id\":\"${TASK_ID}\",\"status\":\"success\"}" >> events/coordination-log.jsonl

git add .
git commit -m "Clawsy: Completed ${TASK_ID} - DESCRIPTION"
git push origin main
```

Tell user: "✅ Task ${TASK_ID} completed!"

### Create a Task for Botsy

When you need Botsy to do something (review, research, etc.):

```bash
cd claude-coordination
git pull --rebase origin main

TASK_ID="task-$(date +%s)"

# Optionally write context
cat > handoffs/clawsy-to-botsy/CONTEXT.md << 'EOF'
# Context for Botsy

**From:** Clawsy
**Date:** $(date +%Y-%m-%d)

## Background
What this is about

## What I Need
What you want Botsy to do
EOF

# Create task
cat > "tasks/pending/${TASK_ID}.json" << EOF
{
  "id": "${TASK_ID}",
  "from": "clawsy",
  "to": "botsy",
  "type": "review",
  "priority": "medium",
  "description": "WHAT YOU NEED BOTSY TO DO",
  "context_file": "handoffs/clawsy-to-botsy/CONTEXT.md",
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF

# Log event
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"agent\":\"clawsy\",\"event\":\"task_created\",\"task_id\":\"${TASK_ID}\"}" >> events/coordination-log.jsonl

git add .
git commit -m "Clawsy: Created ${TASK_ID} for Botsy"
git push origin main
```

Tell user: "✅ Created task ${TASK_ID} for @Botsy"

### Check Botsy's Status

```bash
cd claude-coordination
git pull --rebase origin main
cat status/botsy-status.json
```

Tell user what Botsy's current state is.

---

## 🎯 Common Workflows

### Workflow 1: Implement Based on Botsy's Research

1. User tells you: "@Clawsy check for tasks"
2. You run: `git pull && ls tasks/pending/*.json`
3. You see task from Botsy (type: "implement")
4. You pick up task: move to in-progress, update status, commit, push
5. You read context file: `cat handoffs/botsy-to-clawsy/RESEARCH.md`
6. You do the implementation work
7. You write results to `handoffs/clawsy-to-botsy/IMPL-results.md`
8. You complete task: update JSON, move to completed, update status, commit, push
9. You tell user: "✅ Implementation complete based on Botsy's research"

### Workflow 2: Build and Hand Off to Botsy for Review

1. User asks you to build something
2. You build it (in your actual project directory, not coordination repo)
3. You write build summary to `handoffs/clawsy-to-botsy/BUILD-summary.md`
4. You create task for Botsy (type: "review")
5. Commit, push
6. Tell user: "✅ Build complete, created review task for @Botsy"
7. User tells Botsy to check tasks
8. Botsy reviews and provides feedback

### Workflow 3: Status Sync

User asks: "What are you both working on?"

1. You: `git pull && cat status/*.json`
2. You report both statuses to user
3. User can then coordinate next steps

---

## 🔧 Helper Script (Optional)

You can save this as `clawsy-helpers.sh` and source it:

```bash
#!/bin/bash

COORD_DIR="PATH_TO_YOUR_CLONE"  # Update this

clawsy_sync() {
    cd "$COORD_DIR"
    git pull --rebase origin main
    echo "✅ Synced"
}

clawsy_check_tasks() {
    cd "$COORD_DIR"
    git pull --rebase origin main
    for task in tasks/pending/*.json; do
        if [ -f "$task" ] && grep -q '"to": "clawsy"' "$task"; then
            echo "📬 $(basename $task)"
            cat "$task"
        fi
    done
}

clawsy_status() {
    cd "$COORD_DIR"
    git pull --rebase origin main
    echo "My status:"
    cat status/clawsy-status.json
    echo ""
    echo "Botsy's status:"
    cat status/botsy-status.json
}

# Add more helpers as needed...
```

---

## 📝 Task File Format Reference

```json
{
  "id": "task-1707410000",
  "from": "botsy|clawsy",
  "to": "botsy|clawsy",
  "type": "research|implement|test|review|deploy|build",
  "priority": "low|medium|high|urgent",
  "description": "Clear description of what to do",
  "context_file": "handoffs/path/to/file.md",
  "created_at": "2026-02-08T15:00:00Z",
  "started_at": "2026-02-08T15:05:00Z",
  "completed_at": "2026-02-08T15:30:00Z",
  "result": {
    "status": "success|failed|partial",
    "message": "What was accomplished",
    "artifacts": ["handoffs/clawsy-to-botsy/output.md"]
  }
}
```

## 📊 Status File Format Reference

```json
{
  "agent": "clawsy",
  "state": "idle|working|waiting",
  "current_task": "task-ID or null",
  "last_activity": "2026-02-08T15:30:00Z",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "/your/working/directory",
  "health": "ok|degraded|error",
  "message": "Optional status message"
}
```

---

## ⚠️ Important Notes

- **Always pull before working:** `git pull --rebase origin main`
- **Always push after changes:** So Botsy sees updates immediately
- **Use clear commit messages:** "Clawsy: [action] - [description]"
- **Update status when state changes:** Starting work, completing work, going idle
- **Write detailed handoff docs:** Don't make Botsy guess what you did
- **Never force push:** Always rebase to avoid breaking coordination

---

## 🎯 Quick Reference Commands

```bash
# Sync
cd claude-coordination && git pull --rebase origin main

# Check for tasks
ls -1 tasks/pending/*.json

# Check status
cat status/botsy-status.json status/clawsy-status.json

# View recent activity
tail -10 events/coordination-log.jsonl

# See new handoffs from Botsy
ls -lt handoffs/botsy-to-clawsy/ | head -5
```

---

## 🚀 You're Ready!

Once you complete the example task, you'll have tested the full workflow. Then you and Botsy can start real coordination!

Good luck! 🤖🤝🤖

— Botsy
