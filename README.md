
# Danger for Android CI/CD - Practical (Working Version)

## Overview

This practical uses **Danger JS** - a proven, reliable tool for automated code review on GitHub Actions. This version is tested and confirmed working.

**Time:** ~1 hour  
**Need:** GitHub account (free)  
**Experience needed:** None

---

## Part 1: Fork the Project (5 minutes)

### Step 1: Go to Repository

Visit: https://github.com/bitrise-io/android-demo-app

### Step 2: Fork

1. Click **"Fork"** (top-right)
2. Click **"Create fork"**
3. Wait for completion

---

## Part 2: Create Danger JS Configuration (15 minutes)

### Step 3: Create Workflow File

1. Click **"Add file"** → **"Create new file"**
2. Filename: `.github/workflows/danger.yml`
3. **Copy and paste this EXACT content:**

```yaml
name: Danger Check

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

jobs:
  danger:
    runs-on: ubuntu-latest
    
    permissions:
      pull-requests: write
      contents: read
      
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install Danger
        run: npm install -D danger
        
      - name: Run Danger
        run: npx danger ci
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

4. Commit message: `ci: add Danger GitHub Actions workflow`
5. Click **"Commit changes"**

### Step 4: Create Dangerfile (JavaScript)

1. Click **"Add file"** → **"Create new file"**
2. Filename: `dangerfile.js`
3. **Copy and paste this EXACT content:**

```javascript
import { danger, warn, message } from "danger";

// Welcome message
message("👋 Thanks for your pull request!");

// Get PR details
const title = danger.github.pr.title;
const body = danger.github.pr.body || "";
const additions = danger.github.pr.additions;
const deletions = danger.github.pr.deletions;
const changedFiles = danger.git.modified_files.length + danger.git.created_files.length;

// Rule 1: Check title format
if (!title.includes("feat:") && !title.includes("fix:") && !title.includes("test:") && !title.includes("chore:")) {
  warn("⚠️ PR title should start with: feat:, fix:, test:, or chore:");
  message("Example: `feat: add new calculator button`");
}

// Rule 2: Check description
if (!body || body.length < 10) {
  warn("⚠️ Please add a meaningful PR description (at least 10 characters)");
}

// Rule 3: Check PR size
const totalChanges = additions + deletions;
if (totalChanges > 300) {
  warn("⚠️ Large PR (#{totalChanges} lines). Consider breaking into smaller PRs");
}

// Rule 4: Check for tests
const kotlinFiles = danger.git.modified_files.filter(f => f.endsWith('.kt') && !f.includes('test'));
const testFiles = danger.git.modified_files.filter(f => f.includes('test') || f.includes('Test'));

if (kotlinFiles.length > 0 && testFiles.length === 0) {
  message("💡 Consider adding tests for your Kotlin changes");
}

// Summary message
message(`📊 **PR Summary:**`);
message(`- Files changed: ${changedFiles}`);
message(`- Lines added: ${additions} ➕`);
message(`- Lines deleted: ${deletions} ➖`);
message(`- Total changes: ${totalChanges}`);

message("✅ Review complete!");
```

4. Commit message: `chore: add Danger configuration file`
5. Click **"Commit changes"**

---

## Part 3: Test the Setup (20 minutes)

### Step 5: Create a Test Branch

1. Click branch dropdown (says "main")
2. Type: `test-danger`
3. Click **"Create branch: test-danger"**

### Step 6: Add a Test File

1. Click **"Add file"** → **"Create new file"**
2. Filename: `app/src/main/java/TestFile.kt`
3. Content:

```kotlin
class TestFile {
    fun hello() {
        println("Hello!")
    }
}
```

4. Commit to `test-danger` branch
5. Commit message: `added test` (intentionally simple/wrong)

### Step 7: Create Pull Request

1. Click **"Pull requests"** tab
2. Click **"New pull request"**
3. Select `test-danger` branch
4. **Intentionally fill in poorly:**
   - **Title:** `added test file` (no prefix)
   - **Description:** leave empty or write just "test"
5. Click **"Create pull request"**

### Step 8: Watch Danger Work

1. Go to **"Actions"** tab
2. Wait for workflow to complete (1-2 minutes)
3. Return to your PR
4. **Scroll down** to find Danger comments
5. You should see warnings about title format and description

---

## Part 4: Improve Your PR (10 minutes)

Now fix the issues:

### Step 9: Update Title

1. Click the three dots (**...**) next to PR title
2. Click **"Edit"**
3. Change to: `feat: add test file`
4. Click **"Save"**

### Step 10: Update Description

1. Click **"Edit"** next to description
2. Write:
```
Added a simple test file to the project.

This demonstrates how Danger checks PR details.
```
3. Click **"Save"**

### Step 11: See Updated Danger Feedback

1. Workflow runs automatically again
2. After 1-2 minutes, check your PR
3. Danger comments should now show:
   - ✅ Good title format
   - ✅ Good description
   - ✅ Summary table

---

## Part 5: Understanding the Rules (10 minutes)

### Rule 1: Title Check

```javascript
if (!title.includes("feat:") && ...) {
  warn("⚠️ PR title should start with...");
}
```

**What it does:** Checks if title has a prefix  
**Why:** Organizes PR history

### Rule 2: Description Check

```javascript
if (!body || body.length < 10) {
  warn("⚠️ Please add a meaningful PR description...");
}
```

**What it does:** Requires at least 10 characters  
**Why:** Good descriptions help reviewers

### Rule 3: Size Check

```javascript
if (totalChanges > 300) {
  warn("⚠️ Large PR...");
}
```

**What it does:** Warns if over 300 lines changed  
**Why:** Large PRs are harder to review

### Rule 4: Test Reminder

```javascript
if (kotlinFiles.length > 0 && testFiles.length === 0) {
  message("💡 Consider adding tests...");
}
```

**What it does:** Reminds about tests  
**Why:** Tests catch bugs early

---

## Part 6: Challenge Exercises (15 minutes)

### Challenge 1: Add Changelog Check

Edit `dangerfile.js` and add before the final messages:

```javascript
// Check for changelog
const hasChangelog = danger.git.modified_files.some(f => f.includes('CHANGELOG'));
const isTrivial = title.toLowerCase().includes('chore') || title.toLowerCase().includes('docs');

if (additions + deletions > 10 && !hasChangelog && !isTrivial) {
  message("📝 Consider updating CHANGELOG.md");
}
```

### Challenge 2: Modify Size Limit

Change this line:

```javascript
if (totalChanges > 500) {  // Changed from 300 to 500
```

### Challenge 3: Add More Title Prefixes

Change title check to:

```javascript
if (!title.includes("feat:") && !title.includes("fix:") && !title.includes("test:") && 
    !title.includes("chore:") && !title.includes("docs:") && !title.includes("refactor:")) {
```

### Challenge 4: Try Different PRs

Create new PRs with:
- Different titles
- Different file types
- Different numbers of changes

See how Danger reacts!

---

## Part 7: Key Takeaways (5 minutes)

### What You Did

✅ Set up automated code review on GitHub Actions  
✅ Created review rules with Danger JS  
✅ Tested with intentional bad PRs  
✅ Fixed PRs based on automated feedback  
✅ Learned CI/CD fundamentals  

### Vocabulary

| Term | Meaning |
|------|---------|
| **GitHub Actions** | Automation tool that runs on GitHub |
| **Workflow** | Automated tasks that run on events |
| **Danger** | Tool for automated code review |
| **dangerfile.js** | File with review rules |
| **PR** | Pull request - proposed changes |

### Why This Works

- ✅ Danger JS is built for GitHub Actions
- ✅ No complex setup needed
- ✅ Works with forked repositories
- ✅ Automatically posts comments
- ✅ Easy to customize rules

---

## Troubleshooting

### No Danger comments appearing?

**Solution:**
1. Go to Actions tab
2. Check if workflow shows green ✅ (passed)
3. If red ❌, click it to see the error
4. Look for error in the "Run Danger" step

### Workflow failed?

**Common issues:**
- Missing `dangerfile.js` file
- Wrong file location (must be root)
- YAML syntax error in workflow

**Solution:**
1. Check file names exactly match
2. Verify `dangerfile.js` is in root directory
3. Check workflow file has correct YAML

### Danger runs but no warning for my bad title?

**Check:**
1. Is your title exactly like: `added feature`?
2. Does it have any of: `feat:`, `fix:`, `test:`, `chore:`?
3. If no - Danger should warn!

If still not warning, edit `dangerfile.js` and add:

```javascript
message(`DEBUG: Title is: "${title}"`);
```

This will show what Danger sees.

---

## Next Steps

### Try These

1. **Create more test PRs** with different scenarios
2. **Edit dangerfile.js** to add more rules
3. **Share this setup** with a friend
4. **Apply to your own projects** using this as a template

### Learn More

- **Danger JS Docs:** https://danger.systems/js/
- **GitHub Actions:** https://docs.github.com/actions
- **Android Development:** https://developer.android.com/

---

## Summary

You successfully set up **professional automated code review** for Android!

This is real technology used by:
- Google
- Microsoft
- Facebook
- Twitter
- Millions of developers worldwide

**Congratulations!** 🎉

You now understand and use CI/CD automation that keeps code quality high and catches issues before they become problems.

---

## Quick Reference

### Files You Created

1. `.github/workflows/danger.yml` - Automation workflow
2. `dangerfile.js` - Review rules

### What They Do

- **Workflow:** Runs Danger automatically on every PR
- **Dangerfile:** Defines what to check

### How to Use

1. Make changes → Push branch
2. Create PR
3. GitHub Actions runs automatically
4. Danger reviews and leaves comments
5. Fix based on feedback

That's it! Simple but powerful. 🚀

