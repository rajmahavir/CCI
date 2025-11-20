# Repository Creation Workflow

**Location:** `.claude/workflows/repo-creation.md`  
**Back to main:** `.claude/instructions.md`

---

## Creating a New GitHub Repository

### Prerequisites

Environment variables already available:
- `GITHUB_TOKEN`
- `GITHUB_USERNAME`

### Step-by-Step Process

#### 1. Create the Repository

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/user/repos \
  -d '{
    "name": "my-new-repo",
    "description": "Repository description here",
    "private": false,
    "auto_init": false
  }'
```

**Response:** You'll get JSON with repository details including `html_url`

#### 2. Initialize Local Git (Optional)

```bash
git init
git branch -m main
git remote add origin https://$GITHUB_TOKEN@github.com/$GITHUB_USERNAME/my-new-repo.git
```

#### 3. Add Files

**Option A: Use upload script (recommended)**
```bash
./scripts/upload-to-github.sh my-new-repo "Initial commit"
```

**Option B: Upload individual files**
```bash
content=$(base64 -w 0 "README.md")
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/my-new-repo/contents/README.md \
  -d "{\"message\":\"Add README\",\"content\":\"$content\"}"
```

#### 4. Verify

Repository will be available at:
`https://github.com/$GITHUB_USERNAME/my-new-repo`

---

## Common Options

### Private Repository

```bash
curl -X POST ... -d '{"name":"repo","private":true}'
```

### With Auto-initialization

```bash
curl -X POST ... -d '{"name":"repo","auto_init":true}'
```

### With .gitignore Template

```bash
curl -X POST ... -d '{
  "name":"repo",
  "gitignore_template":"Python"
}'
```

### With License

```bash
curl -X POST ... -d '{
  "name":"repo",
  "license_template":"mit"
}'
```

---

## Complete Example

```bash
# 1. Create repo
RESPONSE=$(curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos \
  -d '{"name":"awesome-project","description":"My awesome project"}')

# 2. Extract URL
REPO_URL=$(echo $RESPONSE | grep -o '"html_url":"[^"]*"' | cut -d'"' -f4)
echo "Created: $REPO_URL"

# 3. Upload files
./scripts/upload-to-github.sh awesome-project "Initial commit"

# 4. Done!
echo "Repository ready: $REPO_URL"
```

---

## Troubleshooting

**Error: "name already exists"**
- Repository name is taken
- Try a different name

**Error: "Bad credentials"**
- Check `$GITHUB_TOKEN` is set
- Verify token hasn't expired

**Error: "Not Found"**
- Check `$GITHUB_USERNAME` is correct
- Verify token has `repo` scope

---

**Next steps:**
- Upload files: `.claude/workflows/file-uploads.md`
- Deploy to Pages: `.claude/workflows/github-pages.md`
