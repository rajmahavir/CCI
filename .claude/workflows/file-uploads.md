# File Uploads Workflow

**Location:** `.claude/workflows/file-uploads.md`  
**Back to main:** `.claude/instructions.md`

---

## Why Use API Uploads?

**Git commits FAIL in Claude Code Web/iOS** with signing errors:
```
error: signing failed: signing operation failed
fatal: failed to write commit object
```

**Solution:** Upload files directly via GitHub API

---

## Method 1: Bulk Upload Script (Recommended)

### Usage

```bash
./scripts/upload-to-github.sh REPO_NAME "commit message" [branch]
```

### Examples

```bash
# Upload all files to main
./scripts/upload-to-github.sh my-repo "Initial commit"

# Upload to specific branch
./scripts/upload-to-github.sh my-repo "Feature update" feature-branch

# Custom message
./scripts/upload-to-github.sh my-repo "Add documentation and examples"
```

### What It Does

1. ✅ Finds all files in current directory
2. ✅ Encodes each file to base64
3. ✅ Uploads via GitHub API
4. ✅ Shows progress with colors
5. ✅ Handles errors gracefully
6. ✅ Updates existing files (checks SHA)

---

## Method 2: Single File Upload

### Basic Upload

```bash
# Encode file
content=$(base64 -w 0 "myfile.txt")

# Upload
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/myfile.txt \
  -d "{\"message\":\"Add myfile\",\"content\":\"$content\"}"
```

### Upload to Subdirectory

```bash
content=$(base64 -w 0 "docs/guide.md")
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/docs/guide.md \
  -d "{\"message\":\"Add guide\",\"content\":\"$content\"}"
```

### Update Existing File

```bash
# Get current SHA
SHA=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/file.txt | \
  grep '"sha"' | head -1 | cut -d'"' -f4)

# Update with SHA
content=$(base64 -w 0 "file.txt")
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/file.txt \
  -d "{\"message\":\"Update file\",\"content\":\"$content\",\"sha\":\"$SHA\"}"
```

---

## Method 3: Upload Function (Reusable)

```bash
upload_file() {
  local file="$1"
  local repo="$2"
  local message="${3:-Update $file}"
  
  # Get SHA if exists
  local sha=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
    https://api.github.com/repos/$GITHUB_USERNAME/$repo/contents/$file | \
    grep '"sha"' | head -1 | cut -d'"' -f4)
  
  # Encode content
  local content=$(base64 -w 0 "$file")
  
  # Upload
  if [ -n "$sha" ]; then
    curl -s -X PUT \
      -H "Authorization: token $GITHUB_TOKEN" \
      https://api.github.com/repos/$GITHUB_USERNAME/$repo/contents/$file \
      -d "{\"message\":\"$message\",\"content\":\"$content\",\"sha\":\"$sha\"}"
  else
    curl -s -X PUT \
      -H "Authorization: token $GITHUB_TOKEN" \
      https://api.github.com/repos/$GITHUB_USERNAME/$repo/contents/$file \
      -d "{\"message\":\"$message\",\"content\":\"$content\"}"
  fi
}

# Use it
upload_file "README.md" "my-repo" "Update README"
```

---

## Bulk Upload Loop

```bash
# Upload all .md files
for file in *.md; do
  content=$(base64 -w 0 "$file")
  curl -s -X PUT \
    -H "Authorization: token $GITHUB_TOKEN" \
    https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/$file \
    -d "{\"message\":\"Add $file\",\"content\":\"$content\"}"
  echo "✓ Uploaded $file"
done
```

---

## Working with Directories

### Upload Directory Recursively

```bash
find . -type f ! -path './.git/*' | while read file; do
  # Remove leading ./
  path="${file#./}"
  
  # Encode
  content=$(base64 -w 0 "$file")
  
  # Upload
  curl -s -X PUT \
    -H "Authorization: token $GITHUB_TOKEN" \
    https://api.github.com/repos/$GITHUB_USERNAME/REPO/contents/$path \
    -d "{\"message\":\"Add $path\",\"content\":\"$content\"}"
  
  echo "✓ $path"
done
```

---

## Important Notes

### File Size Limits

- **GitHub API limit:** 100 MB per file
- **Base64 increases size:** ~33% larger
- **Effective limit:** ~75 MB original file size

### Rate Limits

- **Authenticated:** 5000 requests/hour
- **Each file upload:** 1 request
- **Bulk uploads:** May hit limits with 1000+ files

### Binary Files

✅ Works fine - base64 encoding handles binary correctly

### Large Projects

For projects with many files, consider:
1. Using the upload script (shows progress)
2. Adding delays between uploads
3. Or use git push as fallback (may fail in Web/iOS)

---

## Troubleshooting

**"Invalid request" error**
- Check JSON escaping in file content
- Verify content is valid base64

**"Not Found" error**
- Repository doesn't exist
- Check repo name and username

**"Unprocessable Entity" error**
- File might be too large
- Check base64 encoding

**See also:** `.claude/workflows/troubleshooting.md`

---

**Next:**
- Deploy to GitHub Pages: `.claude/workflows/github-pages.md`
- Create PRs: `.claude/workflows/prs-merges.md`
