---
name: Code-Review Plugin Integration
description: Supplementary code review tool that works alongside existing skills (code-standards-check, pr-evaluator) to accelerate feedback cycles
---

# Code-Review Plugin — Supplementary Integration

The Claude Code-Review Plugin is enabled as a **supplementary tool** that complements (not replaces) your existing code review workflow.

## 🎯 Purpose

**Accelerate code review feedback** by:
- ✅ Auto-analyzing PRs against code standards before human review
- ✅ Flagging common violations early (before `/code-standards-check`)
- ✅ Extracting actionable feedback from reviewer comments (like `/pr-evaluator`)
- ✅ Identifying test coverage gaps (before `/regression-tester`)
- ✅ Providing context-aware suggestions based on codebase patterns

## 📋 Configuration

**File**: `.claude/settings.json`

```json
{
  "codeReview": {
    "enabled": true,
    "strategy": "supplementary",
    "scope": [
      "code-standards-backend.md",
      "code-standards-frontend.md",
      "code-standards-hotwire.md",
      "testing-standards.md"
    ],
    "triggers": {
      "prComments": true,      # ← Auto-analyze PR comments
      "preCommit": false,      # ← Don't block commits
      "manualInvoke": true     # ← Can invoke manually
    }
  }
}
```

**Key Points:**
- `strategy: "supplementary"` — Does NOT replace existing skills
- `prComments: true` — Auto-analyzes reviewer feedback
- `preCommit: false` — Never blocks development workflow
- `manualInvoke: true` — Can call `/code-review` manually

---

## 🔄 Workflow Integration

### **Skill Chain (Unchanged)**

Your existing workflow remains **exactly the same**:

```
1. /code-standards-check    ← Validate code
2. /unit-tester             ← Run tests
3. /regression-tester       ← Full suite
4. /ux-tester (if UI)       ← Accessibility
5. /uat-helper              ← Push to alpha4
6. /pr-helper               ← Create PR
```

### **Code-Review Plugin (New — Optional Supplement)**

Plugin runs **in parallel**, not in the chain:

```
PR created by /pr-helper
    ↓
Code-Review Plugin (optional, automatic):
├─ Auto-scans code against standards
├─ Flags violations (informational)
├─ Analyzes human reviewer comments
├─ Suggests improvements
└─ Reports findings (no blocking)
    ↓
Human reviewers proceed with 2-review requirement
```

**Key**: Plugin provides **context**, not enforcement. Human reviewers still make final decisions.

---

## 🎮 Using Code-Review Plugin

### **Manual Invocation**

```bash
# In Claude Code chat:
/code-review

# Analyzes current changes against standards
# Reports violations and suggestions
# Does NOT block any workflow
```

### **Automatic on PR Comments**

When human reviewers comment on your PR, the plugin:
1. Extracts feedback from comments
2. Categorizes by severity (critical/warning/info)
3. Links to relevant code standards
4. Suggests fixes (like `/pr-evaluator` does)

### **Use Cases**

| Scenario | Action | Plugin Role |
|----------|--------|-------------|
| **Before human review** | Run `/code-review` manually | Early feedback |
| **After reviewer comments** | Read plugin analysis | Context on feedback |
| **Coverage gaps** | Plugin flags untested code | Planning test improvements |
| **Style inconsistencies** | Plugin suggests standards alignment | Accelerate fixes |

---

## 📊 Code-Review vs Skills

### **How It Complements Existing Skills**

| Skill | Purpose | Plugin Adds |
|-------|---------|-----------|
| `/code-standards-check` | Validate against standards | Pre-validation, broader scope |
| `/pr-evaluator` | Parse reviewer comments | Auto-analysis of feedback |
| `/regression-tester` | Test suite classification | Coverage gap identification |
| `/ux-tester` | Accessibility checks | Cross-checks against UX patterns |

**Plugin is NOT a replacement** — it provides early signals and accelerates the feedback loop.

### **When to Use Each**

| Situation | Use |
|-----------|-----|
| After implementation | `/code-standards-check` (required) |
| Before PR submission | `/code-review` optional (see early issues) |
| After human review comments | Read plugin analysis (informational) |
| Full validation before alpha4 | `/regression-tester` (required) |

---

## 🚀 Best Practices

### **DO**
- ✅ Run `/code-review` optionally before submitting PR (catch issues early)
- ✅ Read plugin feedback on reviewer comments (understand intent)
- ✅ Use plugin suggestions to improve code quality
- ✅ Trust your skill chain (`/code-standards-check` → `/regression-tester`)

### **DON'T**
- ❌ Treat plugin findings as blocking (they're not — skills are)
- ❌ Rely on plugin instead of `/code-standards-check` (use both)
- ❌ Let plugin slow down your development (it's optional)
- ❌ Prioritize plugin suggestions over human reviewers (humans decide)

---

## 📋 Scope & Standards

Plugin validates against your standards documents:

```
code-standards-backend.md     ← Rails patterns
code-standards-frontend.md    ← Vue 3 patterns
code-standards-hotwire.md     ← Hotwire patterns (NEW)
testing-standards.md          ← RSpec + Cypress + system tests
```

**Auto-updated when standards change** — no configuration needed.

---

## 🔧 Configuration in Settings

### **Enable/Disable**

```json
{
  "codeReview": {
    "enabled": true      # ← Toggle plugin on/off
  }
}
```

### **Adjust Triggers**

```json
{
  "codeReview": {
    "triggers": {
      "prComments": true,     # Auto-analyze PR comments
      "preCommit": false,     # Don't block commits
      "manualInvoke": true    # Allow /code-review command
    }
  }
}
```

### **Scope Specific Standards**

```json
{
  "codeReview": {
    "scope": [
      "code-standards-backend.md",
      "code-standards-frontend.md",
      "code-standards-hotwire.md",
      "testing-standards.md"
    ]
  }
}
```

---

## 📊 Example: Plugin in Action

### **Scenario: New Hotwire Feature**

```
1. You implement checkout form with Stimulus controller
2. Run /code-standards-check
   → All green

3. Submit PR with /pr-helper
   → PR created

4. Code-Review Plugin auto-runs:
   ├─ ✅ Stimulus controller follows patterns
   ├─ ⚠️  Missing system tests for Turbo frame interaction
   ├─ ℹ️  Consider Hotwire event-driven pattern for form updates
   └─ Suggest: Add spec/system/hotwire/checkout_spec.rb

5. Human reviewers see both:
   ├─ Your implementation
   └─ Plugin pre-analysis (context)

6. Reviewers provide feedback
   → Plugin auto-analyzes comments (like /pr-evaluator)
   → You see structured feedback summary

7. You fix issues, push new commit
   → Run /regression-tester to verify
   → Merge after 2 reviews approve
```

---

## 🔗 Related Documentation

- **Code Standards**: `code-standards-backend.md`, `code-standards-frontend.md`, `code-standards-hotwire.md`
- **Testing Standards**: `testing-standards.md`
- **PR Workflow**: `pr-helper.md` skill
- **Code Review Skill**: `pr-evaluator.md` skill

---

## ❓ FAQ

**Q: Does the plugin block commits or PRs?**  
A: No. It's informational only. Your skill chain (`/code-standards-check`, `/regression-tester`) is what gates PRs.

**Q: Can I disable it?**  
A: Yes. Set `"enabled": false` in settings.json.

**Q: Does it replace `/code-standards-check`?**  
A: No. Use `/code-standards-check` for official validation. Plugin provides early signals.

**Q: Will it slow down my workflow?**  
A: No. It runs in background (if enabled). Manual `/code-review` is optional.

**Q: What if plugin feedback conflicts with human reviewer?**  
A: Trust the human. Plugin is advisory. Your reviewers make the final call.

**Q: Can I configure it per-project?**  
A: Yes, via `.claude/settings.json`. Each project can have different scopes.

---

## 🎯 Summary

**Code-Review Plugin is:**
- ✅ Supplementary (alongside skills, not replacing them)
- ✅ Optional (manual invoke or auto on PR comments)
- ✅ Non-blocking (informational feedback)
- ✅ Standards-aware (validates against your docs)
- ✅ Team-friendly (helps reviewers understand code faster)

**Your skill chain remains the source of truth for code quality.**

Use the plugin to **accelerate feedback**, not replace rigorous validation.
