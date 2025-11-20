# Claude Code - GitHub API Instructions

## 🚨 CRITICAL: Read This First

**Environment variables are ALREADY available in Claude Code:**
- `$GITHUB_TOKEN` - Your GitHub token
- `$GITHUB_USERNAME` - Your username

**Git commits FAIL in Web/iOS** - Use API uploads instead

---

## ⚡ Quick Commands (Copy-Paste Ready)

### Create Repository
```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/user/repos \
  -d '{"name":"REPO_NAME","description":"Description","private":false}'
```

### Upload Single File
```bash
content=$(base64 -w 0 "filename.txt")
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO_NAME/contents/filename.txt \
  -d "{\"message\":\"Add file\",\"content\":\"$content\"}"
```

### Bulk Upload (Recommended)
```bash
./scripts/upload-to-github.sh REPO_NAME "commit message"
```

### Deploy to GitHub Pages
```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO_NAME/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

---

## 📚 Detailed Workflows (Read When Needed)

**Need step-by-step guidance?** Read these:

- **Creating repositories:** `.claude/workflows/repo-creation.md`
- **Uploading files:** `.claude/workflows/file-uploads.md`
- **GitHub Pages deployment:** `.claude/workflows/github-pages.md`
- **Pull requests & merging:** `.claude/workflows/prs-merges.md`
- **Troubleshooting:** `.claude/workflows/troubleshooting.md`

---

## ❌ NEVER Do These

1. **NEVER** use `git commit` - fails with signing errors in Web/iOS
2. **NEVER** try to load .env files - variables already available
3. **NEVER** use `gh` CLI - not available in Web/iOS
4. **NEVER** ask user to create repos manually - you can via API

---

## ✅ ALWAYS Do These

1. **ALWAYS** use API uploads for files (`upload-to-github.sh` or curl)
2. **ALWAYS** use environment variables directly (`$GITHUB_TOKEN`)
3. **ALWAYS** use GitHub REST API for all operations
4. **ALWAYS** provide repo and live URLs when done

---

## 🔍 Need Help?

- **Commands not working?** Read `.claude/workflows/troubleshooting.md`
- **Need examples?** See `github-api-commands.md`
- **Understanding workflow?** Check specific workflow file above

---

**Keep it simple:** Most tasks only need the quick commands above.
