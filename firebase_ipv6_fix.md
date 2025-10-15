# 🧩 Firebase Login Debug — IPv6 Conflict Fix (Kali Linux)

## Problem
When attempting to run:
```bash
firebase login --reauth --debug
```
The CLI hangs, fails to open the browser, or throws connection/authentication errors.  
This is often caused by **IPv6 network conflicts or misconfiguration** on Kali Linux.

---

## Root Cause
Firebase CLI relies on Node.js networking under the hood.  
Some Linux environments (like Kali) have **IPv6 enabled by default** but not fully configured.  
This causes the Firebase authentication redirect to break when Node tries to bind to an IPv6 socket.

---

## Fix (Temporary until reboot)

### 1. Disable IPv6 for the current session
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

### 2. Verify IPv6 is actually disabled
```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
# Expected output: 1
```

### 3. Re-authenticate Firebase
```bash
firebase logout --force
firebase login --reauth --debug
```

✅ You should now be able to log in successfully via browser.

---

## Optional: Re-enable IPv6 later (without reboot)
If you need IPv6 again:
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=0
```

---

## Permanent Fix (if you keep hitting this)
To make IPv6 stay off between reboots:
1. Edit `/etc/sysctl.conf` or create `/etc/sysctl.d/99-disable-ipv6.conf`
2. Add:
   ```
   net.ipv6.conf.all.disable_ipv6 = 1
   net.ipv6.conf.default.disable_ipv6 = 1
   ```
3. Apply:
   ```bash
   sudo sysctl -p
   ```

---

## Notes
- Works for Firebase CLI on Kali Linux (tested on Node ≥18).
- Also fixes `firebase serve` or `emulators:start` binding issues on localhost.
- IPv6 re-enables automatically after reboot unless made permanent.
