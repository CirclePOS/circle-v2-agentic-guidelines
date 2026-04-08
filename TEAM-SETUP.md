---
name: Team Onboarding - Claude Agentic Guidelines
description: Setup instructions for team members to clone and configure circle-v2-agentic-guidelines
---

# Team Setup — Circle V2 Agentic Guidelines

This document guides new team members through setting up the shared agentic guidelines.

## 🚀 Quick Start (5 minutes)

```bash
# 1. Clone the guidelines repo (once per developer)
cd ~/Documents/LocalProjects
git clone https://github.com/CirclePOS/circle-v2-agentic-guidelines.git

# 2. Symlink from your Circle V2 working repo
cd ~/Documents/GitHub/Circle-v2-master4-deploy  # (or your branch)
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines

# 3. Verify symlink
ls -la .claude-guidelines/CLAUDE.md
# Should show: .claude-guidelines/CLAUDE.md -> ...circle-v2-agentic-guidelines/CLAUDE.md

# 4. Verify Claude Code sees the guidelines
claude --version  # Should load CLAUDE.md automatically
```

✅ **Done!** You now have access to:
- `@.claude-guidelines/CLAUDE.md` — Project guidelines
- Agents: `backend-agent`, `frontend-agent`, `tester-agent`
- Skills: `/code-standards-check`, `/tdd-workflow`, etc.
- Standards: Code, testing, architecture docs

---

## 📋 Step-by-Step Setup

### **Step 1: Clone Guidelines Repo**

```bash
# Create directory
mkdir -p ~/Documents/LocalProjects
cd ~/Documents/LocalProjects

# Clone (first time only)
git clone https://github.com/CirclePOS/circle-v2-agentic-guidelines.git

# Verify
cd circle-v2-agentic-guidelines
ls -la
# Should show: CLAUDE.md, SETUP.md, agents/, skills/, standards/, etc.
```

**What this does:**
- Downloads shared guidelines to your local machine
- Creates git history tracking (updates, contributors, changes)
- Single copy shared across all your projects

### **Step 2: Create Symlink**

```bash
# Navigate to your Circle V2 repo
cd ~/Documents/GitHub/Circle-v2-master4-deploy
# (or wherever your working repo is)

# Create symlink to guidelines
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines

# Verify it worked
ls -la .claude-guidelines
# Should show: .claude-guidelines -> ...circle-v2-agentic-guidelines
```

**What this does:**
- Links your repo to shared guidelines
- `.gitignore` already excludes this (won't commit symlink)
- Claude Code auto-discovers `.claude-guidelines/CLAUDE.md`

### **Step 3: Test in Claude Code**

```bash
# Open Claude Code
claude-code --agent backend-agent

# You should see guidelines loaded:
# - CLAUDE.md context loaded
# - Backend agent description shows

# Or in VS Code extension:
# - Open Claude Code chat
# - Type: /tdd-workflow
# - Should load the TDD skill from guidelines
```

### **Step 4: Keep Updated**

```bash
# Weekly (or as needed):
cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines
git pull origin main

# You now have latest standards, agents, skills
# All your projects see the updates (via symlink)
```

---

## 🔄 Multiple Projects Setup

If you work on **multiple Circle V2 projects**, symlink all of them:

```bash
# Single shared guidelines (git clone once)
~/Documents/LocalProjects/circle-v2-agentic-guidelines/

# Symlink from each project
~/Documents/GitHub/Circle-v2-master4-deploy/.claude-guidelines → (symlink)
~/Documents/GitHub/Circle-v2-mobile/.claude-guidelines → (symlink)
~/Documents/GitHub/Circle-v2-admin/.claude-guidelines → (symlink)
# etc.

# All projects use same standards!
```

---

## 🔧 Configuration (If Needed)

### **Custom Local Settings**

Your local `.claude/settings.json` overrides guidelines:

```bash
# Your repo only:
~/Documents/GitHub/Circle-v2-master4-deploy/.claude/settings.json

# Shared guidelines (don't edit):
~/Documents/LocalProjects/circle-v2-agentic-guidelines/CLAUDE.md
```

**Rule**: 
- ✅ Edit `.claude/settings.json` for local preferences (MCP servers, hooks)
- ❌ Don't modify files in circle-v2-agentic-guidelines locally (pull instead)

### **Update Guidelines (Seniors Only)**

If you're a senior and need to update standards:

```bash
cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines

# Create feature branch
git checkout -b improve/hotwire-testing-pattern

# Edit the .md file
# (e.g., standards/testing-standards.md)

# Commit and push
git add .
git commit -m "Clarify Hotwire system test pattern"
git push origin improve/hotwire-testing-pattern

# Create PR in GitHub: https://github.com/CirclePOS/circle-v2-agentic-guidelines
# → Team lead reviews and merges
```

Then other developers just do `git pull` to get updates.

---

## 📚 What You Get

After setup, you have access to:

### **Agents** (`.claude-guidelines/agents/`)
- `backend-agent` — Rails development
- `frontend-agent` — Hotwire + Vue 3
- `tester-agent` — Test strategy & QA

**Invoke**: `claude-code --agent backend-agent`

### **Skills** (`.claude-guidelines/skills/`)
- `code-standards-check` — Validate code
- `tdd-workflow` — Test-first development
- `rspec-runner` — Backend testing
- `regression-tester` — Full test suite
- `unit-tester` — Quick validation
- `ux-tester` — Accessibility checks
- `pr-helper` — Create GitHub PR
- `pr-evaluator` — Review feedback
- `test-writer` — Generate tests
- `uat-helper` — Push to alpha4
- `smoke-tester` — Quick sanity checks
- `user-guide-writer` — Generate docs

**Invoke**: `/code-standards-check`, `/tdd-workflow`, etc.

### **Standards** (`.claude-guidelines/standards/`)
- `code-standards-backend.md` — Rails patterns
- `code-standards-frontend.md` — Vue 3 patterns
- `code-standards-hotwire.md` — Hotwire patterns (NEW)
- `testing-standards.md` — RSpec, Cypress, system tests

**Load in Claude Code**: `@.claude-guidelines/standards/code-standards-backend.md`

### **Workflows** (`.claude-guidelines/workflow/`)
- `issue-workflow.md` — Issue → PR → merge (Steps 1–7)
- `pr-conventions.md` — Branch naming, PR rules
- `deployment-pipeline.md` — Alpha → beta → prod

### **Architecture** (`.claude-guidelines/architecture/`)
- `overview.md` — Project stack summary
- `backend-patterns.md` — Command/Service/Entity patterns
- `frontend-patterns.md` — Turborepo, API, Hotwire patterns

---

## ❓ FAQ

**Q: What if the symlink breaks?**
```bash
# Re-create it
rm .claude-guidelines
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines
```

**Q: How do I get latest updates from the team?**
```bash
cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines
git pull origin main
# That's it! All your projects see the updates.
```

**Q: Can I edit guidelines locally?**
```
No — pull from GitHub instead.
If you have a suggestion, contact a senior to create a PR.
```

**Q: What if I'm offline?**
```
Your symlink points to local files (already downloaded).
You don't need internet after the first git clone.
When you're online, do `git pull` to sync.
```

**Q: Can I delete the symlink?**
```
Yes, but then Claude Code won't find CLAUDE.md.
Just re-create it:
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines
```

**Q: What if .claude-guidelines already exists?**
```bash
# If it's a symlink (correct):
ls -la .claude-guidelines
# Shows: .claude-guidelines -> ...circle-v2-agentic-guidelines

# If it's a directory (wrong):
rm -rf .claude-guidelines
ln -s ~/Documents/LocalProjects/circle-v2-agentic-guidelines .claude-guidelines
```

---

## 🚨 Troubleshooting

### **Claude Code Doesn't Load CLAUDE.md**

```bash
# Check symlink exists
ls -la .claude-guidelines/CLAUDE.md

# Check it points to right place
readlink .claude-guidelines
# Should show: ...circle-v2-agentic-guidelines

# Restart Claude Code and try again
```

### **Skills Not Available (/code-standards-check not found)**

```bash
# Verify guidelines are symlinked
ls -la .claude-guidelines/skills/

# Verify symlinks in .claude/commands/
ls -la .claude/commands/

# If missing, recreate them
cd .claude/commands
for file in ../.claude-guidelines/skills/*.md; do
  ln -s "$file" "$(basename $file)"
done
```

### **Git Pull Fails**

```bash
cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines
git status  # Check for uncommitted changes
git pull origin main
```

---

## ✅ Verification Checklist

After setup, verify everything works:

- [ ] Guidelines repo cloned to `~/Documents/LocalProjects/circle-v2-agentic-guidelines`
- [ ] Symlink created: `.claude-guidelines` → guidelines repo
- [ ] Can read: `.claude-guidelines/CLAUDE.md` (should show project info)
- [ ] Skills available: `/code-standards-check`, `/tdd-workflow`, etc.
- [ ] Agents available: `claude-code --agent backend-agent` works
- [ ] Can git pull: `cd ~/Documents/LocalProjects/circle-v2-agentic-guidelines && git pull` succeeds

✅ **All checked?** You're ready to code with the agentic system!

---

## 📞 Support

**Issue**: [Contact your Bryan]  
**Suggestion**: Create PR in circle-v2-agentic-guidelines repo  
**Emergency**: Use local `.claude/` config to override

---

## 🎯 Next Steps

1. **Complete setup** (this page)
2. **Read CLAUDE.md** (project guidelines)
3. **Read QUICK-START.md** (agent usage)
4. **Try an agent**: `claude-code --agent backend-agent -p "Your task"`

Welcome to the agentic workflow! 🚀
