# GitHub Pages Deployment

**Location:** `.claude/workflows/github-pages.md`  
**Back to main:** `.claude/instructions.md`

---

## Quick Deploy

### Enable GitHub Pages

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO_NAME/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

### Your Site URL

```
https://$GITHUB_USERNAME.github.io/REPO_NAME/
```

---

## Complete Workflow

### 1. Upload Your Website Files

```bash
# Make sure you have index.html
ls index.html

# Upload all files
./scripts/upload-to-github.sh my-website "Initial website"
```

### 2. Enable Pages

```bash
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/my-website/pages \
  -d '{"source":{"branch":"main","path":"/"}}'
```

### 3. Wait for Deployment

Deployment takes 1-2 minutes. Check status:

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/my-website/pages | \
  grep '"status"'
```

### 4. Visit Your Site

```
https://$GITHUB_USERNAME.github.io/my-website/
```

---

## Deployment Options

### Deploy from Root Directory

```bash
curl -X POST ... -d '{"source":{"branch":"main","path":"/"}}'
```

### Deploy from /docs Directory

```bash
curl -X POST ... -d '{"source":{"branch":"main","path":"/docs"}}'
```

### Deploy from Different Branch

```bash
curl -X POST ... -d '{"source":{"branch":"gh-pages","path":"/"}}'
```

---

## Update Deployment Settings

```bash
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pages \
  -d '{"source":{"branch":"main","path":"/docs"}}'
```

---

## Check Deployment Status

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pages
```

**Response fields:**
- `status` - "built" when ready
- `html_url` - Your live site URL

---

## Custom Domain (Optional)

### Set Custom Domain

```bash
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pages \
  -d '{"cname":"www.example.com","source":{"branch":"main","path":"/"}}'
```

### Requirements

1. Add CNAME record in DNS pointing to `$GITHUB_USERNAME.github.io`
2. Create CNAME file in repository root with domain name
3. Wait for DNS propagation (up to 24 hours)

---

## Common Patterns

### Static Website

```bash
# Structure
index.html
styles.css
script.js
images/

# Deploy
./scripts/upload-to-github.sh my-site "Deploy website"
curl -X POST ... /pages -d '{"source":{"branch":"main","path":"/"}}'
```

### Documentation Site

```bash
# Structure
docs/
  index.html
  guide.html
  api.html

# Deploy from /docs
curl -X POST ... /pages -d '{"source":{"branch":"main","path":"/docs"}}'
```

### React/Vue App

```bash
# Build first
npm run build

# Deploy build directory
cd build
../scripts/upload-to-github.sh my-app "Deploy build"
curl -X POST ... /pages
```

---

## Troubleshooting

### "Not Found" Error

**Cause:** Repository doesn't exist or Pages not enabled

**Fix:**
```bash
# Verify repo exists
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO
```

### 404 on Live Site

**Causes:**
- No `index.html` in root (or specified path)
- Deployment still in progress
- Wrong branch/path configuration

**Fix:**
- Check files exist: `ls index.html`
- Wait 2-3 minutes for deployment
- Verify settings with GET /pages

### Blank Page

**Causes:**
- JavaScript errors
- Incorrect relative paths
- Missing CSS/JS files

**Fix:**
- Check browser console
- Use absolute paths or `.` relative paths
- Verify all files uploaded

### "409 Conflict" Error

**Cause:** Pages already enabled

**Fix:**
- Use PUT instead of POST to update
- Or delete and recreate:
```bash
curl -X DELETE -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/pages
```

---

## Complete Example

```bash
#!/bin/bash
# Deploy website to GitHub Pages

REPO="my-awesome-site"
BRANCH="main"

echo "📦 Creating repository..."
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos \
  -d "{\"name\":\"$REPO\"}"

echo "📤 Uploading files..."
./scripts/upload-to-github.sh $REPO "Initial commit"

echo "🚀 Enabling GitHub Pages..."
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/$REPO/pages \
  -d "{\"source\":{\"branch\":\"$BRANCH\",\"path\":\"/\"}}"

echo "✅ Done!"
echo "🌐 Site URL: https://$GITHUB_USERNAME.github.io/$REPO/"
echo "⏱️  Deployment takes 1-2 minutes"
```

---

**See also:**
- Upload files: `.claude/workflows/file-uploads.md`
- Troubleshooting: `.claude/workflows/troubleshooting.md`
