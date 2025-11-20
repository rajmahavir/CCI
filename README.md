# CCI - Claude Code Instructions

**Optimized hierarchical structure for reliable Claude Code automation**

🎯 **Problem Solved:** Claude Code skips reading long instruction files  
✅ **Solution:** Short entry point + detailed workflow files

---

## 🚀 Quick Start

### 1. Set Environment Variables

In Claude Code Settings → Environment Variables:

- `GITHUB_TOKEN` = `ghp_your_token_here`
- `GITHUB_USERNAME` = `your_username`

### 2. Start Using

Claude Code automatically reads `.claude/instructions.md` when you open this project.

---

## 📁 Structure

```
CCI/
├── .claude/
│   ├── instructions.md           ← Entry point (50 lines)
│   └── workflows/
│       ├── repo-creation.md      ← Detailed guides
│       ├── file-uploads.md
│       ├── github-pages.md
│       ├── prs-merges.md
│       └── troubleshooting.md
├── scripts/
│   └── upload-to-github.sh       ← Bulk upload tool
└── README.md                     ← This file
```

---

## 🎯 Why This Works

### The Problem

**Long instructions files:**
- `.claude/instructions.md` = 400+ lines
- Claude Code reads ~50% then skips rest
- Makes wrong assumptions
- Wastes tokens on errors

### The Solution

**Hierarchical structure:**
- Entry point = 50-80 lines (Claude reads 100%)
- Detailed workflows = separate files (read when needed)
- Saves context window space
- Complete understanding

---

## 📖 How Claude Code Uses This

### 1. Reads Entry Point

`.claude/instructions.md` (50 lines)
- Quick commands ✅
- Critical rules ✅
- Pointers to detailed guides ✅

**Result:** Claude understands basics immediately

### 2. Loads Details When Needed

User: "Deploy to GitHub Pages"

Claude:
1. ✅ Reads `.claude/workflows/github-pages.md`
2. ✅ Gets complete deployment workflow
3. ✅ Executes correctly

---

## ⚡ Quick Commands

All commands available in `.claude/instructions.md`:

- **Create repo:** One curl command
- **Upload files:** Use `./scripts/upload-to-github.sh`
- **Deploy Pages:** One curl command
- **Create PR:** One curl command
- **Merge PR:** One curl command

---

## 📚 Detailed Workflows

When Claude needs more info, it reads:

| Task | File |
|------|------|
| Creating repositories | `.claude/workflows/repo-creation.md` |
| Uploading files | `.claude/workflows/file-uploads.md` |
| GitHub Pages | `.claude/workflows/github-pages.md` |
| Pull requests | `.claude/workflows/prs-merges.md` |
| Troubleshooting | `.claude/workflows/troubleshooting.md` |

---

## 🔧 For Developers

### Use This Template

Copy to your projects:

```bash
cp -r CCI/.claude /your-project/
cp CCI/scripts/upload-to-github.sh /your-project/scripts/
```

### Customize Entry Point

Edit `.claude/instructions.md` to add project-specific commands while keeping it short (~50-80 lines).

### Add New Workflows

Create new files in `.claude/workflows/` and reference them in main instructions.

---

## 🎯 Token Efficiency

### Old Approach

```
Read instructions.md (3000 tokens)
→ Hit limit at 50%
→ Missing critical info
→ Trial and error (waste tokens)
```

### New Approach

```
Read instructions.md (500 tokens) ✅
Need details? → Read workflow file (800 tokens) ✅
Total: 1300 tokens
Complete understanding ✅
```

**Savings:** ~60% fewer wasted tokens

---

## 🌟 Key Features

✅ **Short entry point** - Claude reads completely  
✅ **Modular workflows** - Load only what's needed  
✅ **Quick commands** - Copy-paste ready  
✅ **Comprehensive guides** - Detailed when needed  
✅ **Token efficient** - Saves context window  
✅ **Professional scripts** - Bulk upload tool included  

---

## 🔍 Comparison

| Metric | Old Structure | New Structure |
|--------|---------------|---------------|
| Entry file size | 400 lines | 50 lines |
| Claude reads | ~50% | 100% ✅ |
| Context wasted | High | Low ✅ |
| Load time | Slow | Fast ✅ |
| Clarity | Mixed | Clear ✅ |

---

## 🛠️ Maintenance

### Keep Entry Point Short

- Max 80 lines
- Quick commands only
- Pointers to detailed guides

### Detailed Workflows

- Can be longer (100-200 lines)
- Loaded on-demand
- Comprehensive examples

### Add New Workflows

1. Create `.claude/workflows/new-topic.md`
2. Add pointer in `.claude/instructions.md`
3. Keep entry point short

---

## 📦 What's Included

**Core Files:**
- `.claude/instructions.md` - Entry point
- 5 workflow guides (repo, upload, pages, PRs, troubleshooting)
- `scripts/upload-to-github.sh` - Bulk upload tool

**Documentation:**
- This README
- Inline examples in each workflow
- Troubleshooting guide

---

## 🎓 Best Practices

1. **Entry point:** Keep under 80 lines
2. **Workflows:** Detailed, 100-200 lines each
3. **Commands:** Copy-paste ready
4. **Navigation:** Clear pointers between files
5. **Updates:** Maintain both entry and workflows

---

## 🆚 Original vs Optimized

### claudecode-github-automation (Original)

- Single large instructions.md (418 lines)
- Claude reads partially
- Context window issues

### CCI (Optimized)

- Short entry point (50 lines)
- Modular workflows (100-200 lines each)
- Claude reads completely
- Efficient context usage

---

## 🚀 For Claude Code

**When you see this repository:**

1. Read `.claude/instructions.md` first (it's short!)
2. You'll find quick commands there
3. Need details? Check `.claude/workflows/` as indicated
4. Execute confidently with complete information

---

## 📊 Success Metrics

✅ Claude reads entry point completely (100%)  
✅ Loads only needed workflows (efficient)  
✅ Executes correctly first try (no trial/error)  
✅ Uses 60% fewer tokens (optimization)  

---

## 💡 Why It Matters

**User Experience:**
- Fewer repeated instructions
- Faster execution
- More reliable results

**Claude Code Performance:**
- Complete understanding
- Correct execution
- Efficient token usage

---

**Made for seamless Claude Code + GitHub integration** 🚀

Repository: https://github.com/rajmahavir/CCI
