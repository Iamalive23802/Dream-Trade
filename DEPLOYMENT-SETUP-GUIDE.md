# 🚀 GitHub Actions Auto-Deployment Setup Guide

Complete step-by-step guide to set up automatic deployment for any Node.js project.

---

## 📋 **Prerequisites**

- [ ] A server with SSH access (Ubuntu/Debian recommended)
- [ ] A GitHub repository (public or private)
- [ ] Basic knowledge of terminal commands

---

## **Step 1: Generate SSH Key for GitHub Actions**

### On your local machine:

```bash
# Generate a new ED25519 SSH key specifically for GitHub Actions
ssh-keygen -t ed25519 -f ~/github_actions_deploy -C "github-actions-deploy"
# Press Enter for no passphrase (required for automation)

# Convert the private key to base64 (IMPORTANT: single line, no spaces)
cat ~/github_actions_deploy | base64 | tr -d '\n'
```

**📝 Copy the entire output** - you'll need this for Step 3.

---

## **Step 2: Add Public Key to Your Server**

```bash
# Display your public key
cat ~/github_actions_deploy.pub
```

**Copy the public key**, then SSH into your server:

```bash
# Connect to your server
ssh your-user@your-server-ip

# Add the public key to authorized_keys
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

### Test the connection:

```bash
# Test SSH connection with the new key
ssh -i ~/github_actions_deploy your-user@your-server-ip

# If successful, you should be logged into your server
# Type 'exit' to disconnect
```

✅ If you can login successfully, proceed to Step 3.

---

## **Step 3: Configure GitHub Secrets**

Go to your GitHub repository settings:

```
https://github.com/YOUR_USERNAME/YOUR_REPO/settings/secrets/actions
```

Click **"New repository secret"** and add these:

### Required Secrets:

| Secret Name | Value | Example |
|------------|-------|---------|
| `SSH_HOST` | Your server IP or hostname | `123.45.67.89` |
| `SSH_USER` | SSH username | `root` or `ubuntu` |
| `SSH_PRIVATE_KEY_B64` | Base64 key from Step 1 | `LS0tLS1CRUdJTi...` |
| `APP_DIR` | App directory on server | `/var/www/myapp` |

### Optional Secrets:

| Secret Name | Value | Default | Description |
|------------|-------|---------|-------------|
| `SSH_PORT` | SSH port | `22` | Only needed if different |
| `FRONTEND_DIST_DIR` | Frontend destination | - | Where to copy built frontend |
| `BACKEND_PORT` | Backend port | `5000` | For fallback restart method |
| `PM2_PROCESS_NAME` | PM2 process name | `app-backend` | For PM2 restart |
| `SYSTEMD_SERVICE` | Systemd service name | - | For systemd restart |

---

## **Step 4: Create Workflow File**

In your project root, create the directory structure:

```bash
mkdir -p .github/workflows
```

Create file: `.github/workflows/deploy.yml`

Copy the contents from `deploy-workflow-template.yml` and **make these changes**:

### Line 5: Change branch if needed
```yaml
branches: ["main"]  # Change to "master" or your default branch
```

### Lines 83-84: Add your repository info
```yaml
REPO_OWNER: YOUR_GITHUB_USERNAME  # e.g., "Iamalive23802"
REPO_NAME: YOUR_REPO_NAME         # e.g., "My-Project"
```

---

## **Step 5: Adjust Project Structure (if needed)**

The workflow expects this structure:

```
your-project/
├── backend/
│   ├── package.json
│   ├── server.js       # Main entry point
│   └── ...
├── frontend/
│   ├── package.json
│   ├── src/
│   └── ...
└── .github/
    └── workflows/
        └── deploy.yml
```

### If your structure is different:

**No backend folder?** Remove lines 145-169 from deploy.yml

**No frontend folder?** Remove lines 171-192 from deploy.yml

**Different entry point?** Change line 158:
```yaml
pm2 start server.js --name "${NAME}"
# Change to your entry point, e.g.:
pm2 start index.js --name "${NAME}"
# or
pm2 start npm --name "${NAME}" -- start
```

---

## **Step 6: Commit and Push**

```bash
# Add the workflow file
git add .github/workflows/deploy.yml

# Commit
git commit -m "Add GitHub Actions auto-deployment"

# Push to main branch (this will trigger the first deployment!)
git push origin main
```

---

## **Step 7: Monitor Deployment**

Go to GitHub Actions tab:
```
https://github.com/YOUR_USERNAME/YOUR_REPO/actions
```

Click on the latest workflow run to see:
- ✅ Setup SSH
- ✅ Deploy (all the deployment steps)

If anything fails, the logs will show exactly what went wrong.

---

## 🎯 **How It Works**

Once set up, deployments happen automatically:

1. **Push to main** → Workflow triggers
2. **SSH into server** → Using the private key
3. **Clone/update code** → Git pull latest changes
4. **Install dependencies** → npm install
5. **Build frontend** → npm run build
6. **Restart backend** → PM2/systemd/fallback method
7. **✅ Done!** → Your app is updated

---

## 🔄 **What Triggers Deployment?**

✅ Direct push to main branch
✅ Merging Pull Requests into main
✅ Merging branches into main
✅ Manual trigger (empty commit):

```bash
git commit --allow-empty -m "Trigger deployment"
git push origin main
```

---

## 🛠️ **Troubleshooting**

### "Failed to decode SSH_PRIVATE_KEY_B64"
- Make sure you copied the ENTIRE base64 string
- No extra spaces or newlines
- Re-run: `cat ~/github_actions_deploy | base64 | tr -d '\n'`

### "Permission denied (publickey)"
- Public key not added to server's `~/.ssh/authorized_keys`
- Test locally: `ssh -i ~/github_actions_deploy user@server`

### "Repository not found"
- Check `REPO_OWNER` and `REPO_NAME` in workflow file
- Make sure capitalization is correct (GitHub is case-sensitive!)

### "Cannot reach server"
- Check `SSH_HOST` secret is correct
- Check `SSH_PORT` if using non-standard port
- Server firewall might be blocking GitHub Actions IPs

### Backend not restarting
- Check PM2 is installed: `pm2 list`
- Check process name matches `PM2_PROCESS_NAME` secret
- Check server.js path is correct

---

## 🎉 **Success Checklist**

- [ ] SSH key generated and added to server
- [ ] All GitHub secrets configured
- [ ] Workflow file created with correct repo info
- [ ] Pushed to GitHub
- [ ] First deployment succeeded
- [ ] App is accessible on server

---

## 📚 **Additional Resources**

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [PM2 Documentation](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [SSH Key Authentication](https://www.ssh.com/academy/ssh/public-key-authentication)

---

## 🆘 **Need Help?**

Check the GitHub Actions logs for detailed error messages. Each step shows:
- ✅ What succeeded
- ❌ What failed and why

The workflow has built-in debugging to help identify issues quickly.

---

**Created:** January 2026  
**Last Updated:** January 2026
