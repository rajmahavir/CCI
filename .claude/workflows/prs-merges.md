# Pull Requests & Merging

**Location:** `.claude/workflows/prs-merges.md`  
**Back to main:** `.claude/instructions.md`

---

## Quick PR Creation

```bash
# Create PR
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls \
  -d '{
    "title":"PR Title",
    "head":"feature-branch",
    "base":"main",
    "body":"Description"
  }'
```

---

## Complete PR Workflow

### 1. Create Feature Branch Content

Upload files to a feature branch:

```bash
# Upload to feature branch
content=$(base64 -w 0 "newfile.txt")
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/newfile.txt \
  -d "{\"message\":\"Add feature\",\"content\":\"$content\",\"branch\":\"feature-branch\"}"
```

### 2. Create Pull Request

```bash
PR_RESPONSE=$(curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls \
  -d '{
    "title":"Add new feature",
    "head":"feature-branch",
    "base":"main",
    "body":"## Changes\n- Added feature X\n- Fixed bug Y"
  }')

# Get PR number
PR_NUMBER=$(echo $PR_RESPONSE | grep -o '"number":[0-9]*' | grep -o '[0-9]*')
echo "Created PR #$PR_NUMBER"
```

### 3. Merge Pull Request

```bash
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls/$PR_NUMBER/merge \
  -d '{"merge_method":"merge"}'
```

### 4. Delete Branch (Optional)

```bash
curl -X DELETE \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/git/refs/heads/feature-branch
```

---

## Merge Methods

### Standard Merge

```bash
curl -X PUT ... -d '{"merge_method":"merge"}'
```

Creates merge commit, preserves all individual commits.

### Squash and Merge

```bash
curl -X PUT ... -d '{
  "merge_method":"squash",
  "commit_title":"Feature: Add X",
  "commit_message":"All changes in one commit"
}'
```

Combines all commits into one.

### Rebase and Merge

```bash
curl -X PUT ... -d '{"merge_method":"rebase"}'
```

Applies commits on top of base branch.

---

## Check PR Status

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls/$PR_NUMBER
```

**Useful fields:**
- `mergeable` - true/false
- `state` - "open" or "closed"
- `merged` - true/false

---

## List Pull Requests

### All Open PRs

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls
```

### Specific State

```bash
# Open PRs
curl ... /pulls?state=open

# Closed PRs
curl ... /pulls?state=closed

# All PRs
curl ... /pulls?state=all
```

---

## Add Review Comments

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls/$PR_NUMBER/reviews \
  -d '{
    "body":"Looks good!",
    "event":"APPROVE"
  }'
```

**Events:**
- `APPROVE` - Approve PR
- `REQUEST_CHANGES` - Request changes
- `COMMENT` - General comment

---

## Complete Example Workflow

```bash
#!/bin/bash
# Feature branch to PR to merge workflow

REPO="my-project"
BRANCH="feature/new-login"

# 1. Create feature branch with changes
echo "Creating feature branch..."
content=$(base64 -w 0 "login.js")
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/$REPO/contents/login.js \
  -d "{\"message\":\"Add login\",\"content\":\"$content\",\"branch\":\"$BRANCH\"}"

# 2. Create PR
echo "Creating PR..."
PR_RESPONSE=$(curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/$REPO/pulls \
  -d "{
    \"title\":\"Feature: User Login\",
    \"head\":\"$BRANCH\",
    \"base\":\"main\",
    \"body\":\"Adds user authentication\"
  }")

PR_NUMBER=$(echo $PR_RESPONSE | grep -o '"number":[0-9]*' | grep -o '[0-9]*')
echo "✓ Created PR #$PR_NUMBER"

# 3. Merge PR
echo "Merging PR..."
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/$REPO/pulls/$PR_NUMBER/merge \
  -d '{"merge_method":"squash"}'

echo "✓ Merged PR #$PR_NUMBER"

# 4. Delete branch
echo "Cleaning up..."
curl -s -X DELETE \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/$REPO/git/refs/heads/$BRANCH

echo "✅ Done!"
```

---

## Draft Pull Requests

```bash
curl -X POST ... -d '{
  "title":"WIP: Feature",
  "head":"feature-branch",
  "base":"main",
  "draft":true
}'
```

Convert to ready:

```bash
curl -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pulls/$PR_NUMBER \
  -d '{"draft":false}'
```

---

## Troubleshooting

**"Validation Failed" error**
- Check head branch exists
- Verify base branch exists
- Ensure branches are different

**"No commits between X and Y"**
- Branches are identical
- Push changes to head branch first

**Merge conflicts**
- Cannot auto-merge via API
- Resolve manually or use git

---

**See also:**
- Create repo: `.claude/workflows/repo-creation.md`
- Upload files: `.claude/workflows/file-uploads.md`
