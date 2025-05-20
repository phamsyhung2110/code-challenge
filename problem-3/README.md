# Step 1: Identify Which Directories Are Consuming Space
```bash
sudo du -h --max-depth=1 / | sort -hr | head -n 10
```
Drill into large directories (e.g., /var, /usr, /home, /tmp) to find heavy subfolders:
```bash
sudo du -h --max-depth=1 /var | sort -hr
```
# Step 2: Check Disk Usage by File Type (Log Files, Cache, Temp Files)
```bash
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k 5 -hr | head -n 20
```
# Step 3: Check for Log Files (especially under /var/log)
```bash
sudo du -sh /var/log/*
```

# Step 4: Check NGINX Logs Specifically
```bash
sudo ls -lh /var/log/nginx/
Look for very large files like access.log, error.log.
```
# Step 5: Check for Orphaned Files or Docker Usage (if Docker is present)
```bash
sudo docker system df
sudo docker system prune -af
```
# Step 6: Check for Deleted Open Files Still Using Space
```bash
sudo lsof | grep deleted
```
Restart processes that are holding deleted file descriptors:
```bash
sudo systemctl restart nginx
```

# Expected Root Causes & Recovery Steps
## Scenario 1: NGINX Log File Bloat
### Cause
Access logs or error logs growing without rotation (e.g., /var/log/nginx/access.log > 30GB).

### Impact
VM disk fills up, causing NGINX to crash, or system becomes unresponsive due to no space for temp files or PID files.

### Recovery Steps
Truncate large log files:
```bash
sudo truncate -s 0 /var/log/nginx/access.log
```
Enable log rotation:

- Ensure /etc/logrotate.d/nginx exists and is configured properly.

- Test with: sudo logrotate -f /etc/logrotate.conf

Restart NGINX:
```bash
sudo systemctl restart nginx
```
## Scenario 2: Orphaned Temporary Files or Deleted Logs Still Held by NGINX
### Cause
- Files deleted from the filesystem but still held open by NGINX or other processes, especially in /var/log/ or /tmp.

### Impact
- Space appears full (df -h) but not visible with du commands.

### Recovery Steps
- Find such files:
```bash
sudo lsof | grep deleted
```
Restart the process holding them:
```bash
sudo systemctl restart nginx
```
### Preventative Measures
- Enable and tune logrotate for NGINX and system logs.

- Set up disk usage alerts (if not already in place) with thresholds at 80% and 90%.

- Use a separate volume or mount point for logs (/var/log) if logs grow rapidly.

- Automate cleanup of old files in /tmp using systemd-tmpfiles or cron.

- Monitor lsof | grep deleted periodically in health scripts.

