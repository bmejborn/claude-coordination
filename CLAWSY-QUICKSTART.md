# Instructions for Clawsy - Claude Coordination System

**COPY THIS ENTIRE FILE AND SEND TO CLAWSY**

---

## 🤖 Hi Clawsy!

You are **Clawsy** - a desktop Claude Code instance. You coordinate with **Botsy** (Telegram bot) via a shared GitHub repository.

## 🚀 Quick Start

### 1. Clone the Coordination Repo

```bash
git clone https://github.com/bmejborn/claude-coordination.git
cd claude-coordination
```

### 2. Read the Welcome Message

```bash
cat handoffs/botsy-to-clawsy/welcome.md
```

### 3. Complete the Example Task

There's a test task waiting for you. Follow these steps exactly:

#### Step A: Move task to in-progress

```bash
git pull --rebase origin main
mv tasks/pending/example-task-1707410100.json tasks/in-progress/
```

#### Step B: Update your status

Create your status file (replace the placeholder values):

```bash
cat > status/clawsy-status.json << 'EOF'
{
  "agent": "clawsy",
  "state": "working",
  "current_task": "example-task-1707410100",
  "last_activity": "2026-02-08T16:00:00Z",
  "capabilities": [
    "implementation",
    "building",
    "testing",
    "deployment",
    "task-coordination"
  ],
  "location": "desktop-container",
  "working_directory": "/path/to/your/workspace",
  "health": "ok",
  "message": "Testing coordination - working on example task"
}
EOF
```

**Replace:**
- `2026-02-08T16:00:00Z` with current timestamp
- `/path/to/your/workspace` with your actual working directory

#### Step C: Update the task with completion

Edit `tasks/in-progress/example-task-1707410100.json` to add these fields (use the Edit tool):

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
  "started_at": "2026-02-08T16:00:00Z",
  "completed_at": "2026-02-08T16:05:00Z",
  "instructions": "1. Update status/clawsy-status.json with your actual info\n2. Move this task to in-progress/ while working\n3. When done, move to completed/ and add result field\n4. Commit and push\n\nThis tests the full workflow!",
  "result": {
    "status": "success",
    "message": "Coordination system tested. Status updated, workflow verified.",
    "artifacts": []
  }
}
```

**Update timestamps** to current time.

#### Step D: Move to completed

```bash
mv tasks/in-progress/example-task-1707410100.json tasks/completed/
```

#### Step E: Update status to idle

```bash
cat > status/clawsy-status.json << 'EOF'
{
  "agent": "clawsy",
  "state": "idle",
  "current_task": null,
  "last_activity": "2026-02-08T16:05:00Z",
  "capabilities": [
    "implementation",
    "building",
    "testing",
    "deployment",
    "task-coordination"
  ],
  "location": "desktop-container",
  "working_directory": "/path/to/your/workspace",
  "health": "ok",
  "message": "Example task complete - ready for coordination!"
}
EOF
```

**Replace** placeholders with actual values.

#### Step F: Log the event

```bash
echo '{"timestamp":"2026-02-08T16:05:00Z","agent":"clawsy","event":"task_completed","task_id":"example-task-1707410100","status":"success"}' >> events/coordination-log.jsonl
```

**Update timestamp**.

#### Step G: Commit and push

```bash
git add .
git commit -m "Clawsy: Completed example task - coordination verified"
git push origin main
```

#### Step H: Report back

In Telegram, tell the user:

> ✅ Example task completed! Coordination system working. Ready to collaborate with @Botsy.

---

## 📖 Ongoing Usage

### Check for Tasks

When user says **"@Clawsy check for tasks"**:

```bash
cd claude-coordination
git pull --rebase origin main

# Look for tasks addressed to you
for task in tasks/pending/*.json; do
    if [ -f "$task" ] && grep -q '"to": "clawsy"' "$task"; then
        echo "📬 Task found: $(basename $task)"
        cat "$task"
    fi
done
```

Tell user what you found (or "No pending tasks").

### Pick Up a Task

When you want to work on a task:

```bash
TASK_ID="task-123456"  # Replace with actual ID

# Move to in-progress
mv "tasks/pending/${TASK_ID}.json" "tasks/in-progress/"

# Update status
cat > status/clawsy-status.json << EOF
{
  "agent": "clawsy",
  "state": "working",
  "current_task": "${TASK_ID}",
  "last_activity": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "/your/path",
  "health": "ok",
  "message": "Working on ${TASK_ID}"
}
EOF

# Log it
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"agent\":\"clawsy\",\"event\":\"task_started\",\"task_id\":\"${TASK_ID}\"}" >> events/coordination-log.jsonl

git add .
git commit -m "Clawsy: Started ${TASK_ID}"
git push origin main

# Read context file if it exists
CONTEXT=$(grep '"context_file"' "tasks/in-progress/${TASK_ID}.json" | cut -d'"' -f4)
if [ -n "$CONTEXT" ] && [ -f "$CONTEXT" ]; then
    cat "$CONTEXT"
fi
```

### Complete a Task

When done:

```bash
TASK_ID="task-123456"

# If you created results, write handoff
cat > handoffs/clawsy-to-botsy/result-${TASK_ID}.md << 'EOF'
# Results

**Task:** task-123456
**Date:** 2026-02-08

## What Was Done
- Implemented feature X
- Added tests
- Verified functionality

## Outcomes
- Feature working as expected
- All tests passing
EOF

# Edit task JSON to add completion timestamp and result
# (Use Edit tool on tasks/in-progress/${TASK_ID}.json)

# Move to completed
mv "tasks/in-progress/${TASK_ID}.json" "tasks/completed/"

# Update status to idle
cat > status/clawsy-status.json << EOF
{
  "agent": "clawsy",
  "state": "idle",
  "current_task": null,
  "last_activity": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "/your/path",
  "health": "ok",
  "message": "Completed ${TASK_ID}"
}
EOF

# Log completion
echo "{\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"agent\":\"clawsy\",\"event\":\"task_completed\",\"task_id\":\"${TASK_ID}\",\"status\":\"success\"}" >> events/coordination-log.jsonl

git add .
git commit -m "Clawsy: Completed ${TASK_ID} - description here"
git push origin main
```

Tell user: "✅ Task ${TASK_ID} completed!"

### Create Task for Botsy

When you need Botsy to do something:

```bash
TASK_ID="task-$(date +%s)"

# Write context if needed
cat > handoffs/clawsy-to-botsy/context-for-botsy.md << 'EOF'
# Context for Botsy

What you need Botsy to do and why.
EOF

# Create task
cat > "tasks/pending/${TASK_ID}.json" << EOF
{
  "id": "${TASK_ID}",
  "from": "clawsy",
  "to": "botsy",
  "type": "review",
  "priority": "medium",
  "description": "Review the implementation I just completed",
  "context_file": "handoffs/clawsy-to-botsy/context-for-botsy.md",
  "created_at": "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
}
EOF

git add .
git commit -m "Clawsy: Created ${TASK_ID} for Botsy"
git push origin main
```

Tell user: "✅ Created task for @Botsy"

### Check Botsy's Status

```bash
git pull --rebase origin main
cat status/botsy-status.json
```

Tell user what Botsy is up to.

---

## 🎯 Common Workflows

### Workflow: Implement Based on Botsy's Research

1. User: "@Clawsy check for tasks"
2. You: Pull, find task from Botsy (type: "implement")
3. You: Pick up task (move to in-progress, update status)
4. You: Read Botsy's research from context file
5. You: Do the implementation work
6. You: Write results to handoffs/clawsy-to-botsy/
7. You: Complete task (update JSON, move to completed)
8. You: Tell user: "✅ Implemented based on Botsy's research"

### Workflow: Build and Request Review

1. User asks you to build feature X
2. You: Build it in your project
3. You: Write summary to handoffs/clawsy-to-botsy/build-summary.md
4. You: Create task for Botsy (type: "review")
5. You: Push, tell user: "✅ Built, created review task for @Botsy"

---

## 📝 Task File Format

```json
{
  "id": "task-123456",
  "from": "botsy|clawsy",
  "to": "botsy|clawsy",
  "type": "research|implement|test|review|deploy|build",
  "priority": "low|medium|high|urgent",
  "description": "What to do",
  "context_file": "handoffs/path/to/file.md",
  "created_at": "2026-02-08T15:00:00Z",
  "started_at": "2026-02-08T15:05:00Z",
  "completed_at": "2026-02-08T15:30:00Z",
  "result": {
    "status": "success|failed|partial",
    "message": "What happened",
    "artifacts": ["handoffs/path/to/output.md"]
  }
}
```

## 📊 Status File Format

```json
{
  "agent": "clawsy",
  "state": "idle|working|waiting",
  "current_task": "task-ID or null",
  "last_activity": "2026-02-08T15:30:00Z",
  "capabilities": ["implementation", "building", "testing", "deployment", "task-coordination"],
  "location": "desktop-container",
  "working_directory": "/your/workspace",
  "health": "ok|degraded|error",
  "message": "Optional message"
}
```

---

## ⚠️ Important Rules

1. **Always pull before working:** `git pull --rebase origin main`
2. **Always push after changes:** So Botsy sees updates
3. **Update status when state changes:** working → idle, etc.
4. **Use clear commit messages:** "Clawsy: [action] - [description]"
5. **Write detailed handoffs:** Help Botsy understand context
6. **Never force push:** Always rebase

---

## 🎯 Quick Commands Reference

```bash
# Sync
cd claude-coordination && git pull --rebase origin main

# Check tasks
ls tasks/pending/*.json

# Check status
cat status/botsy-status.json status/clawsy-status.json

# Recent activity
tail events/coordination-log.jsonl

# Handoffs from Botsy
ls -t handoffs/botsy-to-clawsy/
```

---

## 📚 Full Documentation

Full setup guide is in the repo:
```bash
cat CLAWSY-SETUP.md
```

---

## ✅ Action Items

1. **Clone the repo** (if not done)
2. **Complete the example task** (steps A-H above)
3. **Report completion** to user in Telegram
4. **Ready for coordination!** 🤖🤝🤖

---

That's it! Once you complete the example task, the coordination system is verified and we can start collaborating.

Good luck!
— Botsy
