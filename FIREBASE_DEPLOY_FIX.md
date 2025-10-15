# 🚒 Firebase Hosting Initialization Failure (CLI Timeout Fix)

## Problem Summary
When running:
```bash
firebase init hosting
```
Firebase CLI failed at:
```
Error: Failed to make request to https://www.gstatic.com/firebasejs/releases.json
```
### Symptoms
- CLI freezes or errors out during hosting setup  
- Network test via Node (`https` module) shows:
  ```
  connect ETIMEDOUT 172.217.215.94:443
  ```
- Manual token refresh succeeds, but fetching `firebasejs/releases.json` fails  

### Root Cause
Firebase CLI attempts to fetch `releases.json` during interactive init.  
Some networks (especially behind firewalls, VPNs, or strict IPv6 routes) block or drop requests to `www.gstatic.com`.  
Even though authentication and project linking succeed, the init wizard crashes before writing config files.

---

## ✅ Manual Fix (Working Procedure)

### 1. Build the React app
Make sure your app’s static files are ready for deployment:
```bash
npm run build
```

### 2. Create `.firebaserc` manually
This file tells Firebase which project to deploy to:
```bash
cat > .firebaserc <<'EOF'
{
  "projects": {
    "default": "project-name"
  }
}
EOF
```

### 3. Create `firebase.json` manually
Define hosting configuration and SPA rewrites:
```bash
cat > firebase.json <<'EOF'
{
  "hosting": {
    "public": "build",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      { "source": "**", "destination": "/index.html" }
    ]
  }
}
EOF
```

### 4. Deploy manually
Skip `firebase init` entirely — deploy straight to Hosting:
```bash
firebase deploy --only hosting --project skillforge-01185 --debug
```

**Expected output:**
- Project detected successfully (`Using project skillforge-01185`)
- Upload progress shown for `/build` directory  
- Hosting URL printed (e.g. `https://skillforge-01185.web.app/`)

---

## 🧠 Pro Tips
- If CLI timeouts persist, disable IPv6 temporarily:
  ```bash
  sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
  ```
- You can test connectivity manually:
  ```bash
  curl -I https://www.gstatic.com/firebasejs/releases.json
  ```
- Keep both `.firebaserc` and `firebase.json` under version control.  
- Future deployments need only:
  ```bash
  npm run build && firebase deploy --only hosting
  ```
