# ⚡ Quick Setup Checklist

Copy this checklist for each new project deployment setup.

---

## 🔐 **Step 1: SSH Key (5 minutes)**

```bash
# Generate key
ssh-keygen -t ed25519 -f ~/github_actions_deploy_PROJECTNAME -C "github-actions"

# Get base64 (copy entire output)
cat ~/github_actions_deploy_PROJECTNAME | base64 | tr -d '\n'

# Get public key
cat ~/github_actions_deploy_PROJECTNAME.pub

# Add to server
ssh user@server
echo "PASTE_PUBLIC_KEY" >> ~/.ssh/authorized_keys
exit

# Test
ssh -i ~/github_actions_deploy_PROJECTNAME user@server
```

- [ ] Key generated
- [ ] Base64 copied
- [ ] Public key added to server
- [ ] Test connection successful

---

## 🔑 **Step 2: GitHub Secrets (3 minutes)**

Go to: `Settings > Secrets and variables > Actions > New repository secret`

### Required:
- [ ] `SSH_HOST` = Server IP
- [ ] `SSH_USER` = SSH username
- [ ] `SSH_PRIVATE_KEY_B64` = Base64 from Step 1
- [ ] `APP_DIR` = `/var/www/myapp`

### Optional:
- [ ] `SSH_PORT` (if not 22)
- [ ] `FRONTEND_DIST_DIR` (if serving frontend separately)
- [ ] `PM2_PROCESS_NAME` (custom PM2 name)
- [ ] `BACKEND_PORT` (if not using PM2)

---

## 📝 **Step 3: Workflow File (2 minutes)**

```bash
# Create directory
mkdir -p .github/workflows

# Copy template
cp deploy-workflow-template.yml .github/workflows/deploy.yml
```

### Edit `.github/workflows/deploy.yml`:

**Line 5:** Branch name
```yaml
branches: ["main"]  # or "master"
```

**Lines 83-84:** Repository info
```yaml
REPO_OWNER: YourGitHubUsername
REPO_NAME: YourRepoName
```

**Line 143:** Branch name (if not main)
```yaml
git reset --hard origin/main  # match line 5
```

**Line 158:** Entry point (if not server.js)
```yaml
pm2 start server.js  # or index.js, app.js, etc.
```

- [ ] Workflow file created
- [ ] Repository info updated
- [ ] Branch name correct
- [ ] Entry point correct

---

## 🚀 **Step 4: Deploy (1 minute)**

```bash
git add .github/workflows/deploy.yml
git commit -m "Add auto-deployment workflow"
git push origin main
```

- [ ] Committed
- [ ] Pushed
- [ ] Check GitHub Actions tab
- [ ] First deployment successful ✅

---

## ✅ **Verification**

1. Go to: `https://github.com/USERNAME/REPO/actions`
2. Click latest workflow run
3. All steps should be green ✅
4. Visit your app URL - should show latest changes

---

## 🐛 **Quick Fixes**

| Error | Solution |
|-------|----------|
| "Failed to decode" | Re-copy base64, ensure no spaces |
| "Permission denied" | Check public key on server |
| "Repository not found" | Check REPO_OWNER/REPO_NAME spelling |
| "Cannot reach server" | Check SSH_HOST and SSH_PORT |
| Backend won't start | Check server.js path, install PM2 |

---

## 📌 **Remember**

- **Any push to main = auto deploy**
- **Merge PR = auto deploy**
- **Manual trigger:** `git commit --allow-empty -m "Deploy" && git push`

---

## 🎯 **Next Project?**

1. Copy `deploy-workflow-template.yml` to new project
2. Run through this checklist
3. Done in ~10 minutes! 🎉

---

**Tip:** Save this checklist for quick reference! ⭐
