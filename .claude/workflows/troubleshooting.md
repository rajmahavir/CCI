# Troubleshooting Guide

**Location:** `.claude/workflows/troubleshooting.md`  
**Back to main:** `.claude/instructions.md`

---

## 🚨 Git Commit Signing Failures

### The Problem

```
error: signing failed: signing operation failed
fatal: failed to write commit object
```

### Why It Happens

Claude Code Web/iOS doesn't support GPG/SSH commit signing.

### What DOESN'T Work

❌ `git config --global commit.gpgsign false`  
❌ `git commit --no-gpg-sign`  
❌ Any git commit command  

### ✅ Solution

Use API uploads instead:

```bash
./scripts/upload-to-github.sh REPO "message"
```

Or individual file uploads:

```bash
content=$(base64 -w 0 "file.txt")
curl -X PUT -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/file.txt \
  -d "{\"message\":\"Add file\",\"content\":\"$content\"}"
```

---

## 🔐 Authentication Errors

### "Bad credentials"

**Causes:**
- Token expired
- Token not set
- Wrong token format

**Fix:**
```bash
# Check if token is set
echo ${GITHUB_TOKEN:0:10}...

# Should show: ghp_XXXXXX...
# If empty, set in Claude Code Settings → Environment Variables
```

### "Not Found" (404)

**Causes:**
- Repository doesn't exist
- Wrong username
- Token lacks permissions

**Fix:**
```bash
# Verify username
echo $GITHUB_USERNAME

# Check token scopes
curl -I -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user | grep x-oauth-scopes

# Should include: repo, workflow
```

### "Resource not accessible by integration"

**Cause:** Token missing required scope

**Fix:** Regenerate token with these scopes:
- `repo` (required)
- `workflow` (recommended)
- `admin:repo_hook` (for webhooks)

---

## 📤 Upload Issues

### "Invalid request" / "Unprocessable Entity"

**Causes:**
- File too large (>100MB)
- Invalid base64 encoding
- Special characters in JSON

**Fix:**
```bash
# Check file size
ls -lh file.txt

# Ensure proper encoding
content=$(base64 -w 0 "file.txt")  # -w 0 is important!

# Escape JSON properly
message="Update file"  # No special chars
```

### "409 Conflict"

**Cause:** File exists but no SHA provided

**Fix:** Get SHA first:
```bash
SHA=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/file.txt | \
  grep '"sha"' | head -1 | cut -d'"' -f4)

# Include SHA in update
curl -X PUT ... -d "{...,\"sha\":\"$SHA\"}"
```

---

## 🌐 GitHub Pages Issues

### "404 Not Found" after deployment

**Causes:**
- No index.html
- Wrong path setting
- Deployment in progress

**Fix:**
```bash
# Check files exist
ls index.html

# Verify Pages settings
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pages

# Wait 2-3 minutes for deployment
```

### Blank page loads

**Causes:**
- JavaScript errors
- Wrong relative paths
- Missing assets

**Fix:**
- Check browser console (F12)
- Use absolute paths: `/style.css`
- Or relative: `./style.css`
- Verify all files uploaded

### CSS/JS not loading

**Cause:** Incorrect paths in HTML

**Fix:**
```html
<!-- Wrong -->
<link href="style.css">

<!-- Correct -->
<link href="./style.css">
<link href="/repo-name/style.css">
```

---

## 🔄 Rate Limiting

### "API rate limit exceeded"

**Cause:** Too many requests (5000/hour limit)

**Fix:**
```bash
# Check rate limit status
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/rate_limit

# Wait for reset (shown in response)
# Or space out requests
```

**Prevention:**
- Use bulk upload script (batches requests)
- Add delays: `sleep 1` between uploads
- Avoid unnecessary API calls

---

## 💾 Environment Variable Issues

### Variables not available

**Symptoms:**
```bash
echo $GITHUB_TOKEN
# Shows nothing
```

**Fix:**

1. Go to Claude Code Settings
2. Select "Environment Variables"
3. Add:
   - `GITHUB_TOKEN` = your token
   - `GITHUB_USERNAME` = your username

### Variables not loading in script

**Wrong:**
```bash
source .env  # Doesn't work in Web/iOS
```

**Correct:**
```bash
# Variables already available, just use them
curl -H "Authorization: token $GITHUB_TOKEN" ...
```

---

## 🔀 Pull Request Issues

### "No commits between base and head"

**Cause:** Branches are identical

**Fix:**
- Make changes in head branch first
- Push changes before creating PR

### "Validation Failed"

**Causes:**
- Branch doesn't exist
- Same branch for head and base
- Invalid branch names

**Fix:**
```bash
# Check branches exist
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/branches

# Ensure head ≠ base
```

### Cannot merge PR

**Causes:**
- Merge conflicts
- Required reviews not approved
- Status checks failing

**Fix:**
- Resolve conflicts manually
- Get approvals
- Fix failing checks

---

## 🗑️ Deletion Issues

### "Reference does not exist"

**Cause:** Branch already deleted or never existed

**Fix:**
```bash
# List all branches
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/branches
```

### "Protected branch"

**Cause:** Branch has protection rules

**Fix:**
- Disable protection in repo settings
- Or use PR to merge changes

---

## 📊 Debugging Commands

### Check token validity

```bash
curl -I -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user
# Should return 200 OK
```

### Verify repo access

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO
# Should return repo details
```

### Test file upload

```bash
echo "test" > test.txt
content=$(base64 -w 0 test.txt)
curl -v -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/test.txt \
  -d "{\"message\":\"test\",\"content\":\"$content\"}"
# Check response for errors
```

### View API response

```bash
# Add -v for verbose output
curl -v -H "Authorization: token $GITHUB_TOKEN" ...

# Or save response
curl ... > response.json
cat response.json
```

---

## 🆘 Still Stuck?

### Check GitHub Status

Visit: https://www.githubstatus.com/

### Verify Token Permissions

1. Go to: https://github.com/settings/tokens
2. Find your token
3. Check scopes include: `repo`, `workflow`
4. Regenerate if needed

### Test Outside Claude Code

Try commands in regular terminal to isolate issue:
```bash
export GITHUB_TOKEN="your_token"
export GITHUB_USERNAME="your_username"
# Run your command
```

---

**Common patterns:**
- Authentication: Check token
- Upload fails: Check file size and encoding
- 404 errors: Verify repo/username
- Rate limits: Wait or space requests
- Git signing: Use API uploads

**See also:**
- Main instructions: `.claude/instructions.md`
- Specific workflows: Other files in `.claude/workflows/`
