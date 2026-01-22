# DevOps Shell Scripting - Advanced Real-World Examples

This guide contains 50 medium to advanced shell script examples based on real-world DevOps scenarios. Each example includes:
- **🎯 Problem Statement** - What real-world issue we're solving
- **🧠 Core Logic Explained** - Step-by-step breakdown of the approach
- **💡 Key Concepts** - Shell scripting techniques used
- **📝 Complete Solution** - Working code you can use

---

## Table of Contents

### Log Management & Analysis
1. [Log Rotation Script](#1-log-rotation-script)
2. [Parse Apache Access Logs](#2-parse-apache-access-logs)
3. [Find Top 10 IPs from Logs](#3-find-top-10-ips-from-logs)
4. [Extract 5xx Errors from Nginx](#4-extract-5xx-errors-from-nginx)
5. [Real-time Log Alerting](#5-real-time-log-alerting)
6. [Compress and Archive Logs](#6-compress-and-archive-logs)
7. [Log File Size Monitor](#7-log-file-size-monitor)

### Server Health & Monitoring
8. [Multi-Server Health Check](#8-multi-server-health-check)
9. [Process Memory Monitor](#9-process-memory-monitor)
10. [Zombie Process Killer](#10-zombie-process-killer)
11. [High CPU Process Alert](#11-high-cpu-process-alert)
12. [Disk I/O Monitor](#12-disk-io-monitor)
13. [Network Connection Monitor](#13-network-connection-monitor)
14. [SSL Certificate Expiry Check](#14-ssl-certificate-expiry-check)
15. [Server Uptime Report](#15-server-uptime-report)

### Backup & Recovery
16. [Incremental Backup Script](#16-incremental-backup-script)
17. [MySQL Database Backup with Retention](#17-mysql-database-backup-with-retention)
18. [PostgreSQL Backup to S3](#18-postgresql-backup-to-s3)
19. [Backup Verification Script](#19-backup-verification-script)
20. [Disaster Recovery Sync](#20-disaster-recovery-sync)

### Deployment & CI/CD
21. [Blue-Green Deployment Switch](#21-blue-green-deployment-switch)
22. [Rolling Restart Script](#22-rolling-restart-script)
23. [Git Hook Pre-commit Validator](#23-git-hook-pre-commit-validator)
24. [Build Artifact Versioning](#24-build-artifact-versioning)
25. [Environment Config Generator](#25-environment-config-generator)
26. [Docker Image Cleanup](#26-docker-image-cleanup)
27. [Kubernetes Pod Restart](#27-kubernetes-pod-restart)
28. [Helm Release Manager](#28-helm-release-manager)

### Security & Compliance
29. [Failed SSH Login Detector](#29-failed-ssh-login-detector)
30. [File Integrity Checker](#30-file-integrity-checker)
31. [Open Port Scanner](#31-open-port-scanner)
32. [User Audit Script](#32-user-audit-script)
33. [Sudo Command Logger](#33-sudo-command-logger)
34. [Firewall Rules Backup](#34-firewall-rules-backup)
35. [Password Expiry Checker](#35-password-expiry-checker)

### AWS & Cloud Operations
36. [AWS EC2 Instance Manager](#36-aws-ec2-instance-manager)
37. [S3 Bucket Sync with Logging](#37-s3-bucket-sync-with-logging)
38. [AWS Cost Alert Script](#38-aws-cost-alert-script)
39. [EBS Snapshot Automation](#39-ebs-snapshot-automation)
40. [CloudWatch Metrics Pusher](#40-cloudwatch-metrics-pusher)

### Networking & Troubleshooting
41. [Multi-Host Ping Sweep](#41-multi-host-ping-sweep)
42. [DNS Resolution Checker](#42-dns-resolution-checker)
43. [Load Balancer Health Probe](#43-load-balancer-health-probe)
44. [Bandwidth Usage Monitor](#44-bandwidth-usage-monitor)
45. [TCP Connection States Report](#45-tcp-connection-states-report)

### Advanced Automation
46. [Parallel Task Executor](#46-parallel-task-executor)
47. [Config File Diff & Merge](#47-config-file-diff--merge)
48. [Cron Job Manager](#48-cron-job-manager)
49. [Service Dependency Checker](#49-service-dependency-checker)
50. [Self-Healing Service Script](#50-self-healing-service-script)

---

## Solutions

---

### 1. Log Rotation Script

**🎯 Problem Statement:**
Application logs grow continuously and can fill up disk space, causing system failures. We need to automatically rotate logs when they exceed a size limit while keeping a limited number of backup copies.

**🧠 Core Logic Explained:**
1. **Check if log file exists** → Use `-f` flag to verify file presence
2. **Get file size in bytes** → `stat -c%s` returns file size
3. **Compare with threshold** → If size > max, trigger rotation
4. **Rename with timestamp** → `mv` file to file.timestamp format
5. **Compress the rotated log** → `gzip` saves disk space
6. **Create fresh empty log** → `touch` creates new log file
7. **Cleanup old rotations** → `ls -t` sorts by time, `tail -n +N` skips first N files, delete the rest

**💡 Key Concepts:**
- `stat -c%s` → Get file size in bytes
- `ls -t` → List files sorted by modification time (newest first)
- `tail -n +N` → Skip first N-1 lines (used to keep N newest files)
- `xargs -r rm` → Delete files from stdin (-r means don't run if empty)

```bash
#!/bin/bash
# Log Rotation Script - Prevents disk space exhaustion

LOG_FILE="/var/log/myapp/app.log"    # Path to the log file to rotate
MAX_SIZE=104857600                    # 100MB in bytes (100 * 1024 * 1024)
KEEP_ROTATIONS=5                      # Number of old log files to keep

if [ -f "$LOG_FILE" ]; then           # Check if log file exists
    size=$(stat -c%s "$LOG_FILE")     # Get file size in bytes (-c%s = size format)
    
    if [ $size -gt $MAX_SIZE ]; then  # If file exceeds max size
        timestamp=$(date +%Y%m%d_%H%M%S)  # Generate timestamp: 20260122_103045
        
        mv "$LOG_FILE" "${LOG_FILE}.${timestamp}"  # Rename: app.log → app.log.20260122_103045
        gzip "${LOG_FILE}.${timestamp}"            # Compress: creates .gz file
        touch "$LOG_FILE"                          # Create new empty log file
        
        # Remove old rotations beyond limit
        # ls -t = sort by time (newest first)
        # tail -n +6 = skip first 5 files, show rest (files to delete)
        # xargs -r rm = delete files (-r = don't run if empty)
        ls -t ${LOG_FILE}.*.gz 2>/dev/null | tail -n +$((KEEP_ROTATIONS+1)) | xargs -r rm
        
        echo "Log rotated: ${LOG_FILE}.${timestamp}.gz"
    fi
fi
```

**Example Execution:**
```
# Before rotation:
$ ls -lh /var/log/myapp/
-rw-r--r-- 1 root root 150M Jan 22 10:30 app.log

# Run script:
$ ./log_rotate.sh
Log rotated: /var/log/myapp/app.log.20260122_103045.gz

# After rotation:
$ ls -lh /var/log/myapp/
-rw-r--r-- 1 root root   0 Jan 22 10:31 app.log          # Fresh empty
-rw-r--r-- 1 root root 12M Jan 22 10:31 app.log.20260122_103045.gz
```

**How Size Calculation Works:**
```
100 MB = 100 × 1024 × 1024 = 104,857,600 bytes

stat -c%s /var/log/myapp/app.log
→ Returns: 157286400 (150MB)

157286400 > 104857600 → TRUE → Rotate!
```

---

### 2. Parse Apache Access Logs

**🎯 Problem Statement:**
Website returning many 404 (Not Found) errors. We need to identify which URLs are broken so developers can fix missing pages or update links.

**🧠 Core Logic Explained:**
1. **Read the access log** → Apache logs have standard format: IP, timestamp, request, status, etc.
2. **Filter by status code** → Column 9 contains HTTP status code (200, 404, 500, etc.)
3. **Extract the URL** → Column 7 contains the requested URL path
4. **Count occurrences** → `sort | uniq -c` groups and counts duplicates
5. **Sort by frequency** → `sort -rn` shows most common errors first

**💡 Key Concepts:**
- Apache log format: `IP - - [timestamp] "METHOD URL PROTOCOL" STATUS SIZE`
- `awk '$9 == 404'` → Filter rows where 9th column equals 404
- `sort | uniq -c` → Classic pattern to count occurrences
- `sort -rn` → Sort numerically (-n) in reverse (-r) order

```bash
#!/bin/bash
# Parse Apache Access Logs for 404 Errors

LOG="/var/log/apache2/access.log"     # Apache access log location

echo "=== 404 Errors Summary ==="
# awk '$9 == 404' → Filter lines where column 9 (status code) equals 404
# {print $7} → Print column 7 (the requested URL)
# sort → Alphabetize URLs (required for uniq)
# uniq -c → Count consecutive duplicates
# sort -rn → Sort by count, reverse numeric (highest first)
# head -20 → Show only top 20 results
awk '$9 == 404 {print $7}' "$LOG" | sort | uniq -c | sort -rn | head -20
```

**Example Apache Log Format:**
```
192.168.1.100 - - [22/Jan/2026:10:30:00 +0000] "GET /missing.html HTTP/1.1" 404 1234
    $1       $2 $3        $4              $5          $6    $7       $8       $9   $10
     │        │  │         │               │           │     │        │        │     │
     IP     ident user  timestamp       method       URL  protocol status  bytes
```

**Example Output:**
```
$ ./parse_404.sh
=== 404 Errors Summary ===
    523 /old-product.html
    312 /images/deleted-logo.png
    189 /api/v1/deprecated
    156 /wp-admin/install.php
     89 /favicon.ico
```

**Pipeline Visualization:**
```
Step 1: awk '$9 == 404 {print $7}'
/old-product.html
/images/deleted-logo.png
/old-product.html
...

Step 2: sort
/images/deleted-logo.png
/old-product.html
/old-product.html
...

Step 3: uniq -c
    312 /images/deleted-logo.png
    523 /old-product.html
...

Step 4: sort -rn | head -20
    523 /old-product.html        ← Most 404s first
    312 /images/deleted-logo.png
```

---

### 3. Find Top 10 IPs from Logs

**🎯 Problem Statement:**
Detect potential DDoS attacks or abusive bots by identifying IP addresses making unusually high numbers of requests to the server.

**🧠 Core Logic Explained:**
1. **Extract IP addresses** → First column in access logs is the client IP
2. **Sort IPs alphabetically** → Required for `uniq` to work properly
3. **Count duplicates** → `uniq -c` counts consecutive identical lines
4. **Sort by count** → Highest request counts appear first
5. **Show top offenders** → Limit output to top 10 IPs

**💡 Key Concepts:**
- `awk '{print $1}'` → Print first column (IP address)
- `uniq -c` requires sorted input to count properly
- AWK associative arrays → `ip[$1]++` creates a counter per IP
- `END` block → Executes after processing all lines

```bash
#!/bin/bash
# Find Top 10 IPs - Detect potential DDoS or abusive bots

LOG="/var/log/nginx/access.log"       # Nginx access log location

echo "=== Top 10 IPs by Request Count ==="
# awk '{print $1}' → Extract first column (IP address)
# sort → Alphabetize IPs (required for uniq to work)
# uniq -c → Count consecutive identical IPs
# sort -rn → Sort numerically, reverse (highest first)
# head -10 → Show only top 10 IPs
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -10

# Advanced: Find IPs exceeding threshold
echo -e "\n=== IPs with >100 requests in last minute ==="
awk -v threshold=100 '                  # -v passes shell variable to awk
{                                       # For each line in log
    ip[$1]++                            # Increment counter for this IP
                                        # ip["1.2.3.4"] = count (associative array)
}
END {                                   # After processing all lines
    for (i in ip) {                     # Loop through all IPs
        if (ip[i] > threshold)          # If count exceeds threshold
            print ip[i], i              # Print: count IP
    }
}' "$LOG" | sort -rn                    # Sort by count descending
```

**Example Output:**
```
$ ./top_ips.sh
=== Top 10 IPs by Request Count ===
  15234 192.168.1.100       ← Possible bot/scraper
   8921 10.0.0.50
   5432 172.16.0.25
   3211 192.168.1.200
   2100 10.0.0.100

=== IPs with >100 requests in last minute ===
  15234 192.168.1.100       ← INVESTIGATE THIS!
   8921 10.0.0.50
```

**How Associative Array Works:**
```
Line 1: 192.168.1.100 ...
ip["192.168.1.100"]++  → ip["192.168.1.100"] = 1

Line 2: 10.0.0.50 ...
ip["10.0.0.50"]++      → ip["10.0.0.50"] = 1

Line 3: 192.168.1.100 ...
ip["192.168.1.100"]++  → ip["192.168.1.100"] = 2

END: Print all IPs where count > threshold
```

---

### 4. Extract 5xx Errors from Nginx

**🎯 Problem Statement:**
Server errors (500, 502, 503, 504) indicate backend problems. We need to extract all 5xx errors with context for debugging application issues.

**🧠 Core Logic Explained:**
1. **Match 5xx pattern** → Use regex `^5[0-9][0-9]$` to match 500-599
2. **Extract relevant fields** → Timestamp, IP, status code, URL
3. **Save to dated file** → Organize reports by date for tracking
4. **Count total errors** → `wc -l` counts lines in output file
5. **Group by error type** → Show distribution of 500 vs 502 vs 503

**💡 Key Concepts:**
- `awk '$9 ~ /regex/'` → Match column against regular expression
- `^5[0-9][0-9]$` → Regex: starts with 5, followed by any two digits
- `wc -l < file` → Count lines without showing filename
- Redirecting output → `>` creates new file, `>>` appends

```bash
#!/bin/bash
# Extract 5xx Server Errors from Nginx logs

LOG="/var/log/nginx/access.log"              # Nginx access log location
OUTPUT="5xx_errors_$(date +%F).log"          # Output file: 5xx_errors_2026-01-22.log

# awk '$9 ~ /regex/' → Match column 9 against regex pattern
# ^5[0-9][0-9]$ → Starts with 5, followed by any two digits (500-599)
# Print: timestamp, IP, status code, URL
awk '$9 ~ /^5[0-9][0-9]$/ {print $4, $5, $1, $9, $7}' "$LOG" > "$OUTPUT"

count=$(wc -l < "$OUTPUT")                   # Count lines (errors found)
echo "Found $count 5xx errors. Saved to $OUTPUT"

# Group by error type to see distribution
echo -e "\n=== Error Distribution ==="
# $4 in OUTPUT file is the status code (500, 502, 503, etc.)
awk '{print $4}' "$OUTPUT" | sort | uniq -c | sort -rn
```

**Example Output:**
```
$ ./extract_5xx.sh
Found 1247 5xx errors. Saved to 5xx_errors_2026-01-22.log

=== Error Distribution ===
    892 502    ← Bad Gateway (backend down)
    234 500    ← Internal Server Error
    121 503    ← Service Unavailable
```

**5xx_errors_2026-01-22.log Content:**
```
[22/Jan/2026:10:30:01 +0000] 192.168.1.100 502 /api/users
[22/Jan/2026:10:30:02 +0000] 10.0.0.50 500 /checkout
[22/Jan/2026:10:30:03 +0000] 172.16.0.25 503 /search
```

**Regex Explanation:**
```
^5[0-9][0-9]$
│ │     │   │
│ │     │   └── $ = End of string
│ │     └── [0-9] = Any digit (0-9)
│ └── [0-9] = Any digit (0-9)
└── ^5 = Starts with "5"

Matches: 500, 501, 502, 503, 504... 599
Does NOT match: 200, 404, 5000
```

---

### 5. Real-time Log Alerting

**🎯 Problem Statement:**
Critical errors in production need immediate attention. We need to monitor logs in real-time and send instant alerts via Slack when critical issues occur.

**🧠 Core Logic Explained:**
1. **Follow log file continuously** → `tail -F` follows even if file is rotated
2. **Read each new line** → `while read` processes lines as they appear
3. **Pattern matching** → Check if line contains CRITICAL, FATAL, or OOM
4. **Format alert message** → Add timestamp for context
5. **Send to Slack** → Use curl to POST JSON to webhook URL
6. **Local logging** → Also save alerts to a separate file

**💡 Key Concepts:**
- `tail -F` vs `tail -f` → `-F` handles log rotation (keeps following new file)
- Pipe to while loop → Creates continuous processing pipeline
- `grep -q` → Quiet mode, returns exit code without output
- `\|` in grep → OR operator to match multiple patterns
- curl with JSON → POST data to REST API endpoints

```bash
#!/bin/bash
# Real-time Log Alerting - Instant notifications for critical errors

LOG="/var/log/application.log"                    # Log file to monitor
ALERT_EMAIL="admin@company.com"                   # Email for alerts
SLACK_WEBHOOK="https://hooks.slack.com/services/XXX"  # Slack webhook URL

# tail -F = Follow file, even through rotation (capital F)
# | while read line = Process each new line as it appears
tail -F "$LOG" | while read line; do
    # grep -q = Quiet mode (no output, just exit code)
    # "CRITICAL\|FATAL\|OOM" = Match any of these patterns (\| = OR)
    if echo "$line" | grep -q "CRITICAL\|FATAL\|OOM"; then
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')    # Current timestamp
        message="[$timestamp] ALERT: $line"       # Format alert message
        
        # Send to Slack via webhook API
        # -s = Silent (no progress bar)
        # -X POST = HTTP POST method
        # -H = Header (Content-type)
        # --data = JSON payload with message
        curl -s -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$message\"}" "$SLACK_WEBHOOK"
        
        # Also log locally for audit trail
        echo "$message" >> /var/log/alerts.log
    fi
done
```

**Example Execution:**
```
# Terminal 1: Run the monitor
$ ./alert_monitor.sh
(waits for log entries...)

# Terminal 2: Simulate critical error
$ echo "2026-01-22 10:30:00 CRITICAL Database connection failed" >> /var/log/application.log

# Result: Slack receives instant notification!
```

**tail -f vs tail -F:**
```
tail -f file    → Follows file by file descriptor
                  STOPS if file is rotated (deleted/renamed)

tail -F file    → Follows file by NAME
                  CONTINUES after rotation (follows new file)

For log monitoring, ALWAYS use -F
```

**Pattern Matching Examples:**
```
Log line: "2026-01-22 10:30:00 CRITICAL Database timeout"
grep "CRITICAL\|FATAL\|OOM" → MATCHES (contains CRITICAL)

Log line: "2026-01-22 10:30:00 INFO User logged in"
grep "CRITICAL\|FATAL\|OOM" → NO MATCH (just INFO)

Log line: "2026-01-22 10:30:00 OOM killer invoked"
grep "CRITICAL\|FATAL\|OOM" → MATCHES (contains OOM)
```

---

### 6. Compress and Archive Logs

**🎯 Problem Statement:**
Logs accumulate over time, consuming disk space. We need to compress old logs, move them to archive storage, and delete very old archives to manage storage efficiently.

**🧠 Core Logic Explained:**
1. **Find files by age** → `find -mtime +N` finds files modified more than N days ago
2. **Compress eligible files** → `gzip` reduces file size significantly
3. **Move to archive** → Transfer compressed files to NFS/backup storage
4. **Delete old archives** → Remove files older than retention period
5. **Exclude already compressed** → `! -name "*.gz"` skips .gz files

**💡 Key Concepts:**
- `find -mtime +N` → Files modified MORE than N days ago
- `find -mtime -N` → Files modified LESS than N days ago
- `-exec gzip {} \;` → Run gzip on each found file
- `$(hostname)` → Include server name in path for identification
- `find -delete` → Built-in delete action (safer than -exec rm)

```bash
#!/bin/bash
# Compress and Archive Logs - Manage disk space efficiently

LOG_DIR="/var/log/myapp"                         # Source log directory
ARCHIVE_DIR="/mnt/nfs/logs/$(hostname)"          # Archive location (NFS mount)
COMPRESS_DAYS=1                                  # Compress logs older than 1 day
DELETE_DAYS=30                                   # Delete archives older than 30 days

mkdir -p "$ARCHIVE_DIR"                          # Create archive dir if needed

# Find and compress logs older than 1 day
# -name "*.log" → Match .log files
# -type f → Regular files only (not directories)
# -mtime +1 → Modified more than 1 day ago
# ! -name "*.gz" → Exclude already compressed files
# -exec gzip {} \; → Run gzip on each found file
find "$LOG_DIR" -name "*.log" -type f -mtime +$COMPRESS_DAYS ! -name "*.gz" -exec gzip {} \;

# Move compressed files to archive storage
find "$LOG_DIR" -name "*.gz" -type f -mtime +$COMPRESS_DAYS -exec mv {} "$ARCHIVE_DIR/" \;

# Delete old archives to free space
# -delete → Built-in delete action (safer than -exec rm)
find "$ARCHIVE_DIR" -name "*.gz" -type f -mtime +$DELETE_DAYS -delete

echo "Log archival complete: $(date)"
```

**Example Execution Flow:**
```
Day 0 (Today):
/var/log/myapp/
├── app.log           (current, 50MB)
├── error.log         (current, 10MB)

Day 2 (Script runs):
/var/log/myapp/
├── app.log           (current, new)
├── error.log         (current, new)
└── app.log.gz        (compressed yesterday's)
    ↓ moves to archive
/mnt/nfs/logs/web-server-01/
├── app.log.gz        (moved here)
└── error.log.gz

Day 32 (Script runs):
- Archives older than 30 days are deleted
```

**Understanding -mtime:**
```
-mtime +1  → Modified MORE than 1 day ago (older)
-mtime -1  → Modified LESS than 1 day ago (newer)
-mtime 1   → Modified EXACTLY 1 day ago

Timeline:
|----+----+----+----+----+----|
Now  1d   2d   3d   4d   5d ago

-mtime +1: [    matches 2d, 3d, 4d, 5d    ]
-mtime -1: [matches Now to <1d]
-mtime 1:  [    exactly 1-2d ago          ]
```

---

### 7. Log File Size Monitor

**🎯 Problem Statement:**
Runaway logging can fill disk space quickly. We need to monitor all log files and alert when any exceeds a size threshold before it causes problems.

**🧠 Core Logic Explained:**
1. **Find all log files** → Use `find` to locate files ending in .log
2. **Get size of each** → `du -m` returns size in megabytes
3. **Compare against threshold** → Alert if size exceeds limit
4. **Loop through results** → Process each file individually
5. **Optional auto-fix** → Can truncate files if needed

**💡 Key Concepts:**
- `du -m` → Disk usage in megabytes
- `cut -f1` → Extract first field (size) from tab-separated output
- `while read file` → Process find results one at a time
- `> "$file"` → Truncate file to zero bytes (empty it)
- Comparison → `[ "$size_mb" -gt "$THRESHOLD_MB" ]`

```bash
#!/bin/bash
# Log File Size Monitor - Alert before disk fills up

LOG_DIR="/var/log"                               # Directory to monitor
THRESHOLD_MB=500                                 # Alert if file exceeds 500MB

# find ... | while read file → Process each file found
find "$LOG_DIR" -name "*.log" -type f | while read file; do
    # du -m = Disk usage in megabytes
    # cut -f1 = Extract first field (size) from "500  /path/file"
    size_mb=$(du -m "$file" | cut -f1)
    
    # Compare size against threshold
    if [ "$size_mb" -gt "$THRESHOLD_MB" ]; then
        echo "WARNING: $file is ${size_mb}MB (threshold: ${THRESHOLD_MB}MB)"
        # Optional: Auto-truncate dangerous large files
        # > "$file"    # This empties the file (use with caution!)
    fi
done
```

**Example Output:**
```
$ ./log_size_monitor.sh
WARNING: /var/log/nginx/access.log is 892MB (threshold: 500MB)
WARNING: /var/log/mysql/slow-query.log is 1245MB (threshold: 500MB)
WARNING: /var/log/app/debug.log is 567MB (threshold: 500MB)
```

**Understanding du Output:**
```
$ du -m /var/log/syslog
25    /var/log/syslog
 │         │
 │         └── File path
 └── Size in MB

$ du -m /var/log/syslog | cut -f1
25    ← Just the number
```

**Safe Truncation Options:**
```bash
# Option 1: Truncate to zero (loses all data)
> "$file"

# Option 2: Keep last 1000 lines
tail -n 1000 "$file" > "$file.tmp" && mv "$file.tmp" "$file"

# Option 3: Keep last 10MB
tail -c 10M "$file" > "$file.tmp" && mv "$file.tmp" "$file"

# Option 4: Rotate instead of truncate
mv "$file" "$file.$(date +%s)"
touch "$file"
```

---

### 8. Multi-Server Health Check

**🎯 Problem Statement:**
In a distributed system, we need to quickly verify that all servers are healthy by checking network connectivity (ping), web service (HTTP), and remote access (SSH).

**🧠 Core Logic Explained:**
1. **Define server list** → List all servers to monitor
2. **Loop through each server** → Check one by one
3. **Ping check** → Verify network reachability
4. **HTTP check** → Verify web service is responding
5. **SSH check** → Verify remote access port is open
6. **Aggregate results** → Save to report file

**💡 Key Concepts:**
- `ping -c 1 -W 2` → Send 1 packet, wait 2 seconds timeout
- `curl -w "%{http_code}"` → Extract HTTP status code from response
- `nc -z -w 2` → Netcat zero-I/O mode to test port (2s timeout)
- `&>/dev/null` → Redirect both stdout and stderr to null
- `tee -a` → Append to file AND show on screen

```bash
#!/bin/bash
# Multi-Server Health Check - Monitor distributed systems

SERVERS="server1.example.com server2.example.com server3.example.com"
REPORT="/tmp/health_report_$(date +%F).txt"      # Daily report file

echo "=== Health Check Report $(date) ===" > "$REPORT"

for server in $SERVERS; do                       # Loop through each server
    echo -n "Checking $server... "               # -n = no newline (stay on same line)
    
    # Ping check - verify network connectivity
    # -c 1 = Send 1 packet only
    # -W 2 = Wait max 2 seconds for reply
    # &>/dev/null = Discard all output (both stdout and stderr)
    if ping -c 1 -W 2 "$server" &>/dev/null; then
        ping_status="OK"
    else
        ping_status="FAIL"
    fi
    
    # HTTP check - verify web service
    # -s = Silent mode (no progress)
    # -o /dev/null = Discard response body
    # -w "%{http_code}" = Output only HTTP status code
    # --connect-timeout 5 = Max 5 seconds to connect
    http_code=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "http://$server/health" 2>/dev/null)
    
    # SSH check - verify remote access
    # nc = netcat (network utility)
    # -z = Zero I/O mode (just scan)
    # -w 2 = Timeout after 2 seconds
    # 22 = SSH port
    if nc -z -w 2 "$server" 22 &>/dev/null; then
        ssh_status="OK"
    else
        ssh_status="FAIL"
    fi
    
    # Output results to both screen and file
    # tee -a = Append to file AND show on screen
    echo "$server | Ping: $ping_status | HTTP: $http_code | SSH: $ssh_status" | tee -a "$REPORT"
done
```

**Example Output:**
```
$ ./health_check.sh
Checking server1.example.com... server1.example.com | Ping: OK | HTTP: 200 | SSH: OK
Checking server2.example.com... server2.example.com | Ping: OK | HTTP: 503 | SSH: OK
Checking server3.example.com... server3.example.com | Ping: FAIL | HTTP: 000 | SSH: FAIL
```

**HTTP Status Codes Reference:**
```
200 = OK (healthy)
000 = Connection failed (server down/unreachable)
404 = Health endpoint not found
500 = Internal server error
502 = Bad gateway (backend down)
503 = Service unavailable
```

**Health Check Flow:**
```
For each server:
┌─────────────────────────────────────────┐
│ 1. PING → Can we reach it at all?       │
│    ↓                                    │
│ 2. HTTP → Is web service running?       │
│    ↓                                    │
│ 3. SSH → Can we connect for management? │
│    ↓                                    │
│ 4. Report results                       │
└─────────────────────────────────────────┘
```

---

### 9. Process Memory Monitor

**🎯 Problem Statement:**
Memory leaks or resource-heavy processes can crash servers. We need to identify top memory consumers and alert when any process uses excessive memory.

**🧠 Core Logic Explained:**
1. **Get process list sorted by memory** → `ps aux --sort=-%mem`
2. **Format output nicely** → Use awk to select and format columns
3. **Filter by threshold** → Only show processes exceeding limit
4. **Show useful info** → User, PID, memory %, command name

**💡 Key Concepts:**
- `ps aux` → Show all processes with detailed info
- `--sort=-%mem` → Sort by memory descending (- means reverse)
- `$4` in ps output → Memory percentage column
- `awk 'NR>1'` → Skip header row (row number > 1)
- `printf` in awk → Formatted output like C language

```bash
#!/bin/bash
# Process Memory Monitor - Find memory hogs

MEM_THRESHOLD=80  # Alert if process uses more than 80% memory

echo "=== Top 10 Memory Consumers ==="
# ps aux = All processes with details
# --sort=-%mem = Sort by memory descending (- = reverse)
# head -11 = Show 11 lines (1 header + 10 processes)
# awk 'NR>1' = Skip header row (Number Row > 1)
# printf = Formatted output (like C language)
# $1=user, $2=PID, $4=memory%, $11=command
ps aux --sort=-%mem | head -11 | awk 'NR>1 {printf "%-10s %-8s %-8s %s\n", $1, $2, $4"%", $11}'

echo -e "\n=== Processes exceeding ${MEM_THRESHOLD}% memory ==="
# -v thresh="$MEM_THRESHOLD" = Pass shell variable to awk
# NR>1 = Skip header
# $4 > thresh = Memory column > threshold
ps aux | awk -v thresh="$MEM_THRESHOLD" '
NR>1 && $4 > thresh {
    printf "ALERT: PID %s (%s) using %.1f%% memory\n", $2, $11, $4
}'
```

**Example Output:**
```
$ ./memory_monitor.sh
=== Top 10 Memory Consumers ===
mysql      1234     45.2%    /usr/sbin/mysqld
java       5678     32.1%    /usr/bin/java
nginx      9012     8.5%     nginx:
redis      3456     5.2%     redis-server
postgres   7890     4.8%     postgres

=== Processes exceeding 80% memory ===
ALERT: PID 1234 (mysqld) using 85.2% memory
```

**ps aux Columns Explained:**
```
$ ps aux | head -2
USER       PID  %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1   0.0  0.1 167820 11784 ?        Ss   Jan01   0:10 /sbin/init
 $1        $2    $3   $4     $5    $6   $7       $8    $9    $10   $11

$1  = USER (process owner)
$2  = PID (process ID)
$3  = %CPU (CPU usage percentage)
$4  = %MEM (Memory usage percentage)  ← We check this
$11 = COMMAND (process name)
```

**printf Format Specifiers:**
```
%-10s = Left-aligned string, 10 chars wide
%-8s  = Left-aligned string, 8 chars wide
%.1f  = Float with 1 decimal place

Output alignment:
mysql      1234     45.2%    /usr/sbin/mysqld
│          │        │        │
└──10 char─┘└8 char─┘└8 char─┘└─rest of line
```

---

### 10. Zombie Process Killer

**🎯 Problem Statement:**
Zombie processes are dead processes that haven't been cleaned up by their parent. They consume PID entries and can indicate application bugs. We need to detect and optionally fix them.

**🧠 Core Logic Explained:**
1. **Find zombie processes** → Check process state column for 'Z'
2. **Get zombie's PID** → Extract process ID
3. **Find parent process** → Use `ps -o ppid=` to get parent PID
4. **Identify parent** → Get parent's command name for context
5. **Report findings** → Show zombie and responsible parent
6. **Optional fix** → Kill parent to clean up zombies (risky!)

**💡 Key Concepts:**
- `ps aux` column 8 → Process state (R=running, S=sleeping, Z=zombie)
- `awk '$8 ~ /Z/'` → Filter where column 8 contains 'Z'
- `ps -o ppid= -p $pid` → Get parent PID (= removes header)
- Zombie can only be killed by killing parent or reboot
- `tr -d ' '` → Remove whitespace from output

```bash
#!/bin/bash
# Zombie Process Killer - Clean up defunct processes

echo "=== Zombie Process Report ==="

# Find zombie processes
# ps aux column 8 = STAT (process state)
# Z = Zombie state (dead but not reaped by parent)
# awk '$8 ~ /Z/' = Match where column 8 contains 'Z'
zombies=$(ps aux | awk '$8 ~ /Z/ {print $2}')

if [ -z "$zombies" ]; then                       # -z = Check if string is empty
    echo "No zombie processes found."
    exit 0
fi

for pid in $zombies; do
    # Get parent PID of zombie
    # ps -o ppid= -p $pid = Show parent PID for this process
    # = after ppid removes header
    # tr -d ' ' = Remove any whitespace
    ppid=$(ps -o ppid= -p $pid 2>/dev/null | tr -d ' ')
    
    # Get parent process name for context
    # ps -o comm= = Show command name (without header)
    pname=$(ps -o comm= -p $ppid 2>/dev/null)
    
    echo "Zombie PID: $pid | Parent PID: $ppid ($pname)"
    
    # Uncomment to kill parent (use with extreme caution!)
    # Zombies can only be cleaned by killing their parent
    # kill -9 $ppid
done

echo -e "\nTotal zombies: $(echo $zombies | wc -w)"   # wc -w = word count
```

**Example Output:**
```
$ ./zombie_killer.sh
=== Zombie Process Report ===
Zombie PID: 12345 | Parent PID: 6789 (python3)
Zombie PID: 12346 | Parent PID: 6789 (python3)
Zombie PID: 12347 | Parent PID: 9999 (bash)

Total zombies: 3
```

**Process States (STAT column):**
```
R = Running (actively using CPU)
S = Sleeping (waiting for event)
D = Uninterruptible sleep (usually I/O)
Z = Zombie (dead but not reaped)  ← We're looking for this
T = Stopped (paused by signal)
```

**Why Zombies Happen:**
```
Normal process lifecycle:
1. Parent creates child (fork)
2. Child runs and exits
3. Parent calls wait() to get exit status
4. Child is fully removed

Zombie scenario:
1. Parent creates child
2. Child exits
3. Parent DOESN'T call wait()  ← Bug in parent!
4. Child becomes zombie (waiting to report exit status)

Fix: Kill the parent → Zombies adopted by init → init reaps them
```

**Safe Cleanup Command:**
```bash
# Find and report zombies only (no action)
ps aux | awk '$8 ~ /Z/ {print "Zombie:", $2, "Parent:", ppid}'

# Aggressive cleanup (kills parent processes!)
# for pid in $(ps aux | awk '$8 ~ /Z/ {print $2}'); do
#     ppid=$(ps -o ppid= -p $pid | tr -d ' ')
#     kill -SIGCHLD $ppid  # Signal parent to reap (polite)
# done
```

---

### 11. High CPU Process Alert

**🎯 Problem Statement:**
A process using high CPU briefly is normal, but sustained high CPU usage indicates problems (infinite loop, resource exhaustion). We need to alert only when high CPU persists for a duration.

**🧠 Core Logic Explained:**
1. **Continuous monitoring loop** → Check repeatedly at intervals
2. **Track when high CPU started** → Store timestamp per PID
3. **Calculate duration** → Current time minus start time
4. **Alert if sustained** → Only alert if duration exceeds threshold
5. **Use associative array** → Track multiple PIDs simultaneously

**💡 Key Concepts:**
- `declare -A` → Create associative array (hash map)
- `${cpu_track[$pid]}` → Access array element by key
- `date +%s` → Unix timestamp (seconds since 1970)
- `ps aux --sort=-%cpu` → Sort by CPU descending
- Infinite loop with `while true` → Continuous monitoring

```bash
#!/bin/bash
# High CPU Process Alert - Alert only for sustained high CPU

CPU_THRESHOLD=90   # Alert if CPU exceeds 90%
DURATION=300       # Must sustain for 5 minutes (300 seconds)
INTERVAL=60        # Check every 60 seconds

# declare -A = Create associative array (like hash map/dictionary)
# Key = PID, Value = timestamp when high CPU first detected
declare -A cpu_track

while true; do                                   # Infinite monitoring loop
    timestamp=$(date +%s)                        # Current Unix timestamp
    
    # Get high CPU processes
    # --sort=-%cpu = Sort by CPU descending
    # awk -v thresh="$CPU_THRESHOLD" = Pass threshold to awk
    # NR>1 = Skip header row
    # $3>thresh = CPU% column > threshold
    ps aux --sort=-%cpu | awk -v thresh="$CPU_THRESHOLD" 'NR>1 && $3>thresh {print $2, $3}' | \
    while read pid cpu; do
        process_name=$(ps -o comm= -p $pid 2>/dev/null)  # Get process name
        
        # Check if we've seen this PID before
        if [ -n "${cpu_track[$pid]}" ]; then     # -n = Not empty (seen before)
            start_time=${cpu_track[$pid]}        # When we first saw high CPU
            elapsed=$((timestamp - start_time))  # Seconds since first seen
            
            # Alert if sustained beyond duration
            if [ $elapsed -ge $DURATION ]; then
                echo "ALERT: $process_name (PID $pid) at ${cpu}% CPU for ${elapsed}s"
            fi
        else
            cpu_track[$pid]=$timestamp           # First time seeing this PID
        fi
    done
    
    sleep $INTERVAL                              # Wait before next check
done
```

**Example Output (after 5+ minutes of high CPU):**
```
$ ./cpu_alert.sh
ALERT: java (PID 1234) at 98.5% CPU for 312s
ALERT: python3 (PID 5678) at 95.2% CPU for 480s
```

**Why Track Duration?**
```
Problem: Brief CPU spikes are normal
- Compiling code: 100% CPU for 30 seconds → Normal
- Processing request: 95% CPU for 5 seconds → Normal
- Infinite loop bug: 99% CPU for hours → BAD!

Solution: Only alert if high CPU PERSISTS

Timeline:
Time 0min:   java at 95% CPU → Start tracking
Time 1min:   java at 96% CPU → Still tracking...
Time 2min:   java at 94% CPU → Still tracking...
Time 5min:   java at 97% CPU → ALERT! (sustained)

vs

Time 0min:   python at 98% CPU → Start tracking
Time 1min:   python at 5% CPU → Remove from tracking (back to normal)
```

**Associative Array Explained:**
```bash
declare -A cpu_track

# Store: cpu_track[PID] = timestamp
cpu_track[1234]=1706012400
cpu_track[5678]=1706012340

# Retrieve
echo ${cpu_track[1234]}  → 1706012400

# Check if exists
if [ -n "${cpu_track[1234]}" ]; then
    echo "PID 1234 is being tracked"
fi
```

---

### 12. Disk I/O Monitor

**🎯 Problem Statement:**
Slow application performance is often caused by disk I/O bottlenecks. We need to monitor disk utilization and alert when disks are overloaded.

**🧠 Core Logic Explained:**
1. **Run iostat** → Get disk I/O statistics
2. **Sample twice** → First sample is since boot, second is current
3. **Parse output** → Extract utilization percentage (last column)
4. **Compare threshold** → Alert if utilization too high
5. **Report all disks** → Show status of each disk device

**💡 Key Concepts:**
- `iostat -dx 1 2` → Extended stats, 1 second interval, 2 samples
- Last column in iostat → %util (percentage of time disk was busy)
- `$NF` in awk → Last field (Number of Fields)
- `tail -n +4` → Skip first 4 lines (headers)
- `+ 0` → Force string to number conversion in awk

```bash
#!/bin/bash
# Disk I/O Monitor - Detect storage bottlenecks

THRESHOLD=80  # Alert if disk utilization exceeds 80%

echo "=== Disk I/O Report ==="
# iostat -dx = Extended disk statistics
# 1 2 = 1 second interval, 2 samples (second sample is current)
# tail -n +4 = Skip first 4 lines (headers and first sample)
# NF > 0 = Line has fields (not empty)
# $1 !~ /^Device/ = Skip "Device" header lines
# $NF = Last field (utilization percentage)
# + 0 = Force string to number conversion
iostat -dx 1 2 | tail -n +4 | awk -v thresh="$THRESHOLD" '
NF > 0 && $1 !~ /^Device/ && $1 !~ /^$/ {
    util = $NF + 0                        # Last column = %util, +0 converts to number
    if (util > thresh) {
        printf "ALERT: %s at %.1f%% utilization\n", $1, util
    } else {
        printf "OK: %s at %.1f%% utilization\n", $1, util
    }
}'
```

**Example Output:**
```
$ ./disk_io_monitor.sh
=== Disk I/O Report ===
OK: sda at 12.5% utilization
ALERT: sdb at 89.3% utilization     ← This disk is overloaded!
OK: nvme0n1 at 5.2% utilization
```

**Understanding iostat Output:**
```
$ iostat -dx
Device    r/s     w/s    rkB/s    wkB/s  %util
sda       5.00   25.00   120.00   500.00  12.50
sdb      85.00  150.00  3400.00  6000.00  89.30
           │       │       │        │       │
           │       │       │        │       └── %util (what we check)
           │       │       │        └── Write KB/s
           │       │       └── Read KB/s
           │       └── Writes per second
           └── Reads per second

$NF = Last Field = %util column
```

**Utilization Meanings:**
```
%util = Percentage of time disk was busy

0-50%   = Normal, healthy
50-80%  = Moderate load, monitor
80-95%  = High load, potential bottleneck
95-100% = Disk saturated, I/O wait likely

High %util + slow response = Storage bottleneck
```

**Why Two Samples?**
```
iostat 1 2  →  Take 2 samples, 1 second apart

Sample 1: Averages since boot (not useful for current state)
Sample 2: Activity during last 1 second (what we want!)

tail -n +4 skips headers and sample 1
```

---

### 13. Network Connection Monitor

**🎯 Problem Statement:**
Too many connections in certain states (like TIME_WAIT) can exhaust resources and prevent new connections. We need to monitor TCP connection states and alert on anomalies.

**🧠 Core Logic Explained:**
1. **Get all TCP connections** → `ss -tan` shows TCP connections
2. **Group by state** → Count connections in each state
3. **Check TIME_WAIT** → High count indicates connection issues
4. **Analyze by port** → Find which services have most connections
5. **Alert on threshold** → Warn if too many in problematic states

**💡 Key Concepts:**
- `ss -tan` → Socket statistics: TCP, all states, numeric (no DNS)
- TCP states: ESTABLISHED, TIME_WAIT, CLOSE_WAIT, SYN_SENT, etc.
- `state[$1]++` → awk pattern to count occurrences
- TIME_WAIT → Connection closed but waiting (normal, but too many = problem)
- `split(string, array, ":")` → Split string into array by delimiter

```bash
#!/bin/bash
# Network Connection Monitor - Track TCP states

TIMEWAIT_THRESHOLD=1000                          # Alert if TIME_WAIT exceeds this

echo "=== TCP Connection States ==="
# ss -tan = Socket Statistics: TCP, All states, Numeric (no DNS)
# NR>1 = Skip header row
# state[$1]++ = Count occurrences of each state (associative array)
# END block = Execute after all lines processed
ss -tan | awk 'NR>1 {state[$1]++} END {for (s in state) print s, state[s]}' | sort -k2 -rn

# Check TIME_WAIT specifically
# ss -tan state time-wait = Show only TIME_WAIT connections
timewait=$(ss -tan state time-wait | wc -l)
if [ "$timewait" -gt "$TIMEWAIT_THRESHOLD" ]; then
    echo "WARNING: $timewait TIME_WAIT connections (threshold: $TIMEWAIT_THRESHOLD)"
fi

echo -e "\n=== Connections per Port ==="
# Split destination address by : to get port
# split($4,a,":") = Split column 4 by colon into array a
# a[length(a)] = Last element (the port number)
ss -tan | awk 'NR>1 {split($4,a,":"); port=a[length(a)]; ports[port]++} END {for (p in ports) print p, ports[p]}' | sort -k2 -rn | head -10
```

**Example Output:**
```
$ ./network_monitor.sh
=== TCP Connection States ===
ESTAB 2847               ← Active connections
TIME-WAIT 523            ← Closed, waiting to expire
CLOSE-WAIT 12            ← Other end closed, we haven't
LISTEN 45                ← Waiting for connections
SYN-SENT 3               ← Connecting to remote
FIN-WAIT-1 2             ← We initiated close

=== Connections per Port ===
443 1523                 ← HTTPS
80 892                   ← HTTP
3306 234                 ← MySQL
6379 156                 ← Redis
```

**TCP State Explanations:**
```
LISTEN     → Server waiting for connections
ESTAB      → Active, working connection
TIME-WAIT  → Connection closed, waiting 2*MSL (usually 60s)
CLOSE-WAIT → Remote closed, local hasn't responded yet
FIN-WAIT   → Local closed, waiting for remote ACK
SYN-SENT   → Connection attempt in progress
```

**Why Monitor TIME_WAIT?**
```
Normal: Some TIME_WAIT is expected
Problem: Too many = socket exhaustion

Each TCP connection uses a unique (srcIP:srcPort, dstIP:dstPort) tuple
TIME_WAIT holds this tuple for ~60 seconds after close
Too many = can't create new connections!

Causes of high TIME_WAIT:
- High traffic web server
- Short-lived connections
- Connection pool not being reused
```

---

### 14. SSL Certificate Expiry Check

**🎯 Problem Statement:**
Expired SSL certificates cause website outages and security warnings. We need to proactively check certificate expiry dates and alert before they expire.

**🧠 Core Logic Explained:**
1. **Connect to each domain** → Use openssl to establish SSL connection
2. **Extract certificate** → Get the server's certificate
3. **Parse expiry date** → Extract "notAfter" date from certificate
4. **Convert to epoch** → Convert date to Unix timestamp for calculation
5. **Calculate days remaining** → Subtract current time from expiry
6. **Categorize severity** → Critical if <7 days, Warning if <30 days

**💡 Key Concepts:**
- `openssl s_client` → SSL/TLS client for testing connections
- `-servername` → Required for SNI (Server Name Indication)
- `openssl x509 -noout -enddate` → Extract certificate end date
- `date -d "date" +%s` → Convert date string to Unix timestamp
- Epoch arithmetic → `(expiry - now) / 86400` = days remaining

```bash
#!/bin/bash
# SSL Certificate Expiry Check - Prevent outages from expired certs

DOMAINS="example.com api.example.com shop.example.com"
WARN_DAYS=30                                     # Warn if expires in 30 days
CRIT_DAYS=7                                      # Critical if expires in 7 days

for domain in $DOMAINS; do
    # Connect to domain and extract certificate expiry date
    # echo | = Send empty input (just want to connect)
    # openssl s_client = SSL/TLS client
    # -servername = SNI (Server Name Indication) - required for shared hosting
    # -connect = Host and port to connect
    # 2>/dev/null = Suppress connection messages
    # | openssl x509 = Parse the certificate
    # -noout = Don't output the certificate itself
    # -enddate = Show only the expiry date
    # cut -d= -f2 = Extract value after "notAfter="
    expiry_date=$(echo | openssl s_client -servername "$domain" -connect "$domain:443" 2>/dev/null | \
                  openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
    
    if [ -z "$expiry_date" ]; then               # Check if we got a date
        echo "ERROR: Cannot check $domain"
        continue                                 # Skip to next domain
    fi
    
    # Convert dates to Unix timestamps for comparison
    # date -d "string" +%s = Parse date string, output as epoch
    expiry_epoch=$(date -d "$expiry_date" +%s)   # Expiry as timestamp
    current_epoch=$(date +%s)                    # Current time as timestamp
    days_left=$(( (expiry_epoch - current_epoch) / 86400 ))  # Seconds to days
    
    # Categorize by severity
    if [ $days_left -le $CRIT_DAYS ]; then
        echo "CRITICAL: $domain expires in $days_left days ($expiry_date)"
    elif [ $days_left -le $WARN_DAYS ]; then
        echo "WARNING: $domain expires in $days_left days ($expiry_date)"
    else
        echo "OK: $domain expires in $days_left days"
    fi
done
```

**Example Output:**
```
$ ./ssl_check.sh
OK: example.com expires in 245 days
WARNING: api.example.com expires in 23 days (Feb 14 12:00:00 2026 GMT)
CRITICAL: shop.example.com expires in 5 days (Jan 27 12:00:00 2026 GMT)
```

**Days Calculation Explained:**
```
expiry_epoch  = 1735689600 (Jan 1, 2027 00:00:00)
current_epoch = 1706000000 (Jan 23, 2026 ~12:00)
difference    = 29689600 seconds

29689600 / 86400 (seconds per day) = 343.63 days
```

**Understanding the openssl Pipeline:**
```
Step 1: echo |
  → Sends empty input to openssl (we just want the cert, not interaction)

Step 2: openssl s_client -connect domain:443
  → Establishes SSL connection and outputs certificate

Step 3: openssl x509 -noout -enddate
  → Parses cert, outputs: notAfter=Jan 1 12:00:00 2027 GMT

Step 4: cut -d= -f2
  → Extracts: Jan 1 12:00:00 2027 GMT
```

**What is SNI (-servername)?**
```
Many domains share one IP address (virtual hosting)
SNI tells server which certificate to send
Without SNI, you might get the wrong certificate!
```

---

### 15. Server Uptime Report

**🎯 Problem Statement:**
Management needs weekly reports showing server fleet health. We need to collect uptime and load information from all servers and generate an HTML report.

**🧠 Core Logic Explained:**
1. **Read server list** → Get servers from config file
2. **SSH to each server** → Run `uptime` command remotely
3. **Parse uptime output** → Extract uptime duration and load average
4. **Handle failures** → Mark servers as DOWN if SSH fails
5. **Generate HTML** → Create formatted report for web viewing
6. **Color coding** → Green for UP, Red for DOWN

**💡 Key Concepts:**
- `ssh -o ConnectTimeout=5` → Fail fast if server unreachable
- `uptime` output format → Includes uptime and load averages
- `awk -F'delimiter'` → Set field separator
- Here document `<<'EOF'` → Multi-line string (quotes prevent expansion)
- HTML generation → Build report programmatically

```bash
#!/bin/bash
# Server Uptime Report - Generate HTML report for management

SERVERS_FILE="/etc/server_list.txt"              # File with one server per line
REPORT="/var/www/html/uptime_report.html"        # Output HTML file

# Create HTML header using here document
# <<'EOF' = Literal (no variable expansion until we write the body)
cat > "$REPORT" << 'EOF'
<!DOCTYPE html>
<html><head><title>Server Uptime Report</title>
<style>table{border-collapse:collapse}td,th{border:1px solid #ddd;padding:8px}</style>
</head><body>
<h1>Server Uptime Report - $(date)</h1>
<table><tr><th>Server</th><th>Uptime</th><th>Load</th><th>Status</th></tr>
EOF

# Process each server from the list
while read server; do
    # SSH with timeout to get uptime
    # -o ConnectTimeout=5 = Fail after 5 seconds if can't connect
    # 2>/dev/null = Suppress SSH errors
    result=$(ssh -o ConnectTimeout=5 "$server" "uptime" 2>/dev/null)
    
    if [ $? -eq 0 ]; then                        # SSH succeeded
        # Parse uptime output: "10:30:00 up 5 days, 2:30, 3 users, load average: 0.52, 0.58, 0.59"
        # awk -F'up ' = Split by "up ", take second part
        # awk -F',' = Split that by comma, take first part (duration)
        uptime=$(echo "$result" | awk -F'up ' '{print $2}' | awk -F',' '{print $1}')
        # awk -F'load average: ' = Split by "load average: ", take second part
        load=$(echo "$result" | awk -F'load average: ' '{print $2}')
        status="<span style='color:green'>UP</span>"
    else                                         # SSH failed
        uptime="N/A"
        load="N/A"
        status="<span style='color:red'>DOWN</span>"
    fi
    
    # Append row to HTML table
    echo "<tr><td>$server</td><td>$uptime</td><td>$load</td><td>$status</td></tr>" >> "$REPORT"
done < "$SERVERS_FILE"                           # Read servers from file

echo "</table></body></html>" >> "$REPORT"       # Close HTML tags
echo "Report generated: $REPORT"
```

**Example Output (HTML rendered):**
```
┌───────────────────────────────────────────────────────────┐
│ Server Uptime Report - Wed Jan 22 10:30:00 UTC 2026      │
├──────────────────┬──────────────┬─────────────┬──────────┤
│ Server           │ Uptime       │ Load        │ Status   │
├──────────────────┼──────────────┼─────────────┼──────────┤
│ web-server-01    │ 45 days      │ 0.52, 0.58  │ UP       │
│ web-server-02    │ 12 days      │ 1.20, 0.98  │ UP       │
│ db-server-01     │ N/A          │ N/A         │ DOWN     │
│ api-server-01    │ 90 days      │ 0.10, 0.15  │ UP       │
└──────────────────┴──────────────┴─────────────┴──────────┘
```

**Understanding uptime Output Parsing:**
```
Raw output:
" 10:30:00 up 45 days, 2:30, 3 users, load average: 0.52, 0.58, 0.59"

After awk -F'up ' '{print $2}':
"45 days, 2:30, 3 users, load average: 0.52, 0.58, 0.59"

After awk -F',' '{print $1}':
"45 days"                    ← This is what we show
```

---

### 16. Incremental Backup Script

**🎯 Problem Statement:**
Full backups take too long and too much space. We need incremental backups that only store changed files while still allowing point-in-time restores.

**🧠 Core Logic Explained:**
1. **Use rsync with hard links** → `--link-dest` creates hard links to unchanged files
2. **Each backup is complete** → Appears full but shares unchanged files with previous
3. **Update "latest" symlink** → Always points to most recent backup
4. **Automatic rotation** → Delete backups older than retention period
5. **Space efficient** → Only changed files consume new disk space

**💡 Key Concepts:**
- `rsync --link-dest` → Hard link unchanged files from reference backup
- Hard links → Multiple directory entries pointing to same file data
- Symlink → Pointer to another file/directory path
- `ln -s` → Create symbolic link
- `-maxdepth 1` → Don't recurse into subdirectories

```bash
#!/bin/bash
# Incremental Backup - Space-efficient using hard links

SOURCE="/var/www/html"                           # Directory to backup
BACKUP_BASE="/backup/www"                        # Base backup location
DATE=$(date +%Y-%m-%d_%H%M)                      # Timestamp: 2026-01-22_1030
LATEST="$BACKUP_BASE/latest"                     # Symlink to most recent backup
CURRENT="$BACKUP_BASE/$DATE"                     # This backup's directory
KEEP_DAYS=7                                      # Keep backups for 7 days

# Create backup with hard links to previous backup
# If "latest" exists, use it as reference for hard links
if [ -d "$LATEST" ]; then
    # --link-dest = Hard link unchanged files from this backup
    # -a = Archive mode (preserves permissions, timestamps, etc.)
    # -v = Verbose output
    # --delete = Remove files from dest that don't exist in source
    rsync -av --delete --link-dest="$LATEST" "$SOURCE/" "$CURRENT/"
else
    # First backup - no reference, full copy
    rsync -av --delete "$SOURCE/" "$CURRENT/"
fi

# Update "latest" symlink to point to this backup
rm -f "$LATEST"                                  # Remove old symlink
ln -s "$CURRENT" "$LATEST"                       # Create new symlink

# Remove old backups beyond retention
# -maxdepth 1 = Don't recurse into backup directories
# -type d = Directories only
# -mtime +7 = Modified more than 7 days ago
# ! -name "latest" = Don't delete the symlink
find "$BACKUP_BASE" -maxdepth 1 -type d -mtime +$KEEP_DAYS ! -name "latest" -exec rm -rf {} \;

echo "Incremental backup complete: $CURRENT"
```

**Example Execution:**
```
$ ls -la /backup/www/
drwxr-xr-x  5 root root 4096 Jan 22 10:00 2026-01-20_1000
drwxr-xr-x  5 root root 4096 Jan 21 10:00 2026-01-21_1000
drwxr-xr-x  5 root root 4096 Jan 22 10:30 2026-01-22_1030  ← New backup
lrwxrwxrwx  1 root root   30 Jan 22 10:30 latest -> /backup/www/2026-01-22_1030

$ du -sh /backup/www/*
150M  2026-01-20_1000     ← Full size (no reference)
 12M  2026-01-21_1000     ← Only changed files stored!
  8M  2026-01-22_1030     ← Only today's changes
```

**How Hard Links Save Space:**
```
Without --link-dest (Traditional full backups):
Day 1: 150MB (full backup)
Day 2: 150MB (full backup)
Day 3: 150MB (full backup)
Total: 450MB

With --link-dest (Incremental with hard links):
Day 1: 150MB (full backup)
Day 2:  12MB (only changed files, rest are hard links to Day 1)
Day 3:   8MB (only changed files, rest are hard links)
Total: 170MB  ← 62% savings!

Each backup LOOKS complete (all files present) but shares unchanged files
```

**Hard Links Explained:**
```
Regular file:
file.txt → [data on disk]

Hard link:
file.txt ────┐
             ├──→ [same data on disk]
file_link.txt┘

Both names point to SAME data
Data only deleted when ALL links removed
```

---

### 17. MySQL Database Backup with Retention

**🎯 Problem Statement:**
Databases need regular backups with off-site storage. We need to backup all MySQL databases, compress them, upload to cloud storage, and manage retention.

**🧠 Core Logic Explained:**
1. **Get list of databases** → Query MySQL for all database names
2. **Exclude system databases** → Skip information_schema, mysql, etc.
3. **Dump each database** → Use mysqldump for consistent backup
4. **Compress on-the-fly** → Pipe directly to gzip
5. **Upload to S3** → AWS CLI for cloud storage
6. **Cleanup old backups** → Delete local files beyond retention

**💡 Key Concepts:**
- `mysqldump --single-transaction` → Consistent backup without locking (InnoDB)
- `--routines` → Include stored procedures and functions
- Pipe to gzip → `mysqldump | gzip > file.sql.gz` (saves disk I/O)
- `grep -Ev` → Extended regex, invert match
- `$?` → Exit status of last command (0 = success)

```bash
#!/bin/bash
# MySQL Multi-Database Backup with S3 Upload

MYSQL_USER="backup_user"                         # MySQL username
MYSQL_PASS="secure_password"                     # MySQL password
BACKUP_DIR="/backup/mysql"                       # Local backup directory
S3_BUCKET="s3://company-backups/mysql"           # S3 destination bucket
DATE=$(date +%Y-%m-%d)                           # Date stamp: 2026-01-22
RETENTION_DAYS=7                                 # Keep local backups for 7 days

mkdir -p "$BACKUP_DIR"                           # Create backup dir if needed

# Get list of databases, excluding system databases
# mysql -e "query" = Execute query and exit
# SHOW DATABASES = List all database names
# grep -Ev = Extended regex, invert match (exclude lines matching pattern)
# Pattern excludes: header row, system databases
databases=$(mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" -e "SHOW DATABASES;" | \
    grep -Ev "(Database|information_schema|performance_schema|sys)")

# Loop through each database
for db in $databases; do
    backup_file="$BACKUP_DIR/${db}_${DATE}.sql.gz"  # Output: /backup/mysql/myapp_2026-01-22.sql.gz
    
    echo "Backing up $db..."
    
    # mysqldump = Create SQL dump of database
    # --single-transaction = Consistent snapshot without locking (for InnoDB)
    # --routines = Include stored procedures and functions
    # | gzip = Pipe directly to compression (saves disk I/O and space)
    mysqldump -u"$MYSQL_USER" -p"$MYSQL_PASS" \
        --single-transaction \
        --routines \
        "$db" | gzip > "$backup_file"
    
    # Check if mysqldump succeeded
    if [ $? -eq 0 ]; then
        echo "Uploading $db to S3..."
        # aws s3 cp = Copy file to S3
        # $(hostname) = Include server name in path for organization
        # --quiet = Suppress progress output
        aws s3 cp "$backup_file" "$S3_BUCKET/$(hostname)/" --quiet
    else
        echo "ERROR: Backup failed for $db"
    fi
done

# Cleanup old local backups (keep S3 copies longer)
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "MySQL backup complete: $(date)"
```

**Example Execution:**
```
$ ./mysql_backup_s3.sh
Backing up production_db...
Uploading production_db to S3...
Backing up analytics_db...
Uploading analytics_db to S3...
Backing up user_db...
Uploading user_db to S3...
MySQL backup complete: Wed Jan 22 03:00:05 UTC 2026

$ ls -la /backup/mysql/
-rw-r--r-- 1 root root 45M Jan 22 03:00 production_db_2026-01-22.sql.gz
-rw-r--r-- 1 root root 12M Jan 22 03:00 analytics_db_2026-01-22.sql.gz
-rw-r--r-- 1 root root  8M Jan 22 03:00 user_db_2026-01-22.sql.gz

$ aws s3 ls s3://company-backups/mysql/web-server-01/
2026-01-22 03:00:05   45M production_db_2026-01-22.sql.gz
2026-01-22 03:00:08   12M analytics_db_2026-01-22.sql.gz
2026-01-22 03:00:10    8M user_db_2026-01-22.sql.gz
```

**grep -Ev Pattern Breakdown:**
```
MySQL output:
Database
information_schema
performance_schema
sys
production_db
analytics_db
user_db

After grep -Ev "(Database|information_schema|performance_schema|sys)":
production_db
analytics_db
user_db    ← Only user databases remain
```

**Restore from Backup:**
```bash
# Download from S3 and restore
aws s3 cp s3://company-backups/mysql/web-server-01/production_db_2026-01-22.sql.gz .
gunzip < production_db_2026-01-22.sql.gz | mysql -u root -p production_db
```

---

### 18. PostgreSQL Backup to S3

**🎯 Problem Statement:**
PostgreSQL databases need backups that support parallel restore for faster recovery. We need to create backups in custom format and upload securely to cloud storage.

**🧠 Core Logic Explained:**
1. **Use custom dump format** → `-Fc` flag enables parallel restore
2. **Create backup file** → pg_dump creates binary dump
3. **Verify backup success** → Check exit code before upload
4. **Upload with encryption** → S3 server-side encryption (SSE)
5. **Verify upload** → Confirm file exists in S3 before deleting local

**💡 Key Concepts:**
- `pg_dump -Fc` → Custom format (compressed, supports parallel restore)
- `pg_restore -j N` → Restore using N parallel jobs (with -Fc dumps)
- `--sse AES256` → Server-side encryption in S3
- `du -h` → Human-readable file size
- Verify before delete → Never delete local until remote confirmed

```bash
#!/bin/bash
# PostgreSQL Backup to S3 with Encryption

PG_USER="postgres"                               # PostgreSQL username
BACKUP_DIR="/backup/postgresql"                  # Local backup directory
S3_BUCKET="s3://company-backups/postgresql"      # S3 destination
DATE=$(date +%Y-%m-%d_%H%M)                      # Timestamp: 2026-01-22_1030
DB_NAME="production"                             # Database to backup

mkdir -p "$BACKUP_DIR"                           # Create directory if needed

# Full backup with custom format (supports parallel restore)
backup_file="$BACKUP_DIR/${DB_NAME}_${DATE}.dump"  # Output file path

# pg_dump = PostgreSQL backup utility
# -U = Username
# -Fc = Custom format (compressed, allows parallel restore with pg_restore -j N)
# -f = Output file
# Alternative formats: -Fp (plain SQL), -Ft (tar), -Fd (directory)
pg_dump -U "$PG_USER" -Fc -f "$backup_file" "$DB_NAME"

if [ $? -eq 0 ]; then                            # Check if pg_dump succeeded
    # Get backup size for reporting
    # du -h = Disk usage in human-readable format (e.g., "1.5G")
    # cut -f1 = Extract first field (size only)
    size=$(du -h "$backup_file" | cut -f1)
    echo "Backup successful: $backup_file ($size)"
    
    # Upload to S3 with server-side encryption
    # --sse AES256 = Server-side encryption using AES-256
    # This encrypts data at rest in S3
    aws s3 cp "$backup_file" "$S3_BUCKET/" --sse AES256
    
    # Verify upload succeeded before deleting local
    # aws s3 ls = List S3 objects matching path
    # &>/dev/null = Suppress all output (we only check exit code)
    if aws s3 ls "$S3_BUCKET/$(basename $backup_file)" &>/dev/null; then
        echo "S3 upload verified"
        rm "$backup_file"                        # Remove local after confirmed upload
    fi
else
    echo "ERROR: Backup failed" >&2              # >&2 = Write to stderr
    exit 1
fi
```

**Example Execution:**
```
$ ./pg_backup_s3.sh
Backup successful: /backup/postgresql/production_2026-01-22_1030.dump (2.3G)
upload: /backup/postgresql/production_2026-01-22_1030.dump to s3://company-backups/postgresql/production_2026-01-22_1030.dump
S3 upload verified

$ aws s3 ls s3://company-backups/postgresql/
2026-01-22 10:35:42  2.3G production_2026-01-22_1030.dump
```

**pg_dump Format Options:**
```
Format   Flag   Features
────────────────────────────────────────────────────────────
Plain    -Fp    SQL text file, human-readable, no parallel restore
Custom   -Fc    Compressed binary, parallel restore with -j N
Tar      -Ft    Tar archive, can extract individual tables
Directory -Fd   Directory with one file per table, parallel dump/restore

Custom format (-Fc) is best for:
✓ Compression (saves space)
✓ Fast parallel restore (-j 4 uses 4 CPU cores)
✓ Selective restore (restore specific tables)
```

**Restore from Custom Format:**
```bash
# Standard restore
pg_restore -U postgres -d production production_2026-01-22_1030.dump

# Parallel restore (4 jobs) - much faster!
pg_restore -U postgres -d production -j 4 production_2026-01-22_1030.dump

# Restore specific table only
pg_restore -U postgres -d production -t users production_2026-01-22_1030.dump
```

---

### 19. Backup Verification Script

**🎯 Problem Statement:**
Backups are useless if they're corrupted. We need to verify backup integrity by performing test restores and running data validation checks.

**🧠 Core Logic Explained:**
1. **Accept backup file as argument** → Pass backup path to script
2. **Create temporary database** → Use timestamp to ensure unique name
3. **Restore backup** → Use pg_restore to load data
4. **Run integrity checks** → Count tables, verify row counts
5. **Report results** → Show pass/fail with details
6. **Cleanup** → Always drop test database

**💡 Key Concepts:**
- `createdb` / `dropdb` → PostgreSQL utilities for database management
- `psql -t` → Tuples only (no headers/footers)
- `pg_stat_user_tables` → PostgreSQL system view with table statistics
- `date +%s` → Unix timestamp for unique naming
- Always cleanup → Test database should never remain

```bash
#!/bin/bash
# Backup Verification Script - Test restore integrity

BACKUP_FILE="$1"                                 # Backup file passed as argument
TEST_DB="backup_test_$(date +%s)"                # Unique test DB name using timestamp
PG_USER="postgres"                               # PostgreSQL user

# Validate argument provided
if [ -z "$BACKUP_FILE" ]; then                   # -z = String is empty
    echo "Usage: $0 <backup_file>"
    exit 1
fi

echo "=== Backup Verification Started ==="

# Step 1: Create temporary test database
# createdb = PostgreSQL utility to create database
# Using timestamp ensures unique name even if script runs twice
createdb -U "$PG_USER" "$TEST_DB"

# Step 2: Restore backup to test database
echo "Restoring to test database..."
# pg_restore = PostgreSQL restore utility for custom format dumps
# -d = Target database
# 2>/dev/null = Suppress warnings (expected during restore)
pg_restore -U "$PG_USER" -d "$TEST_DB" "$BACKUP_FILE" 2>/dev/null

if [ $? -eq 0 ]; then                            # Restore succeeded
    # Step 3: Run integrity checks
    
    # Count tables in restored database
    # psql -t = Tuples only (no headers/footers)
    # -c = Execute command
    # information_schema.tables = System view listing all tables
    table_count=$(psql -U "$PG_USER" -d "$TEST_DB" -t -c \
        "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='public'")
    
    # Get top 5 tables by row count
    # pg_stat_user_tables = Statistics on user tables
    # n_live_tup = Number of live rows (not deleted)
    row_counts=$(psql -U "$PG_USER" -d "$TEST_DB" -c \
        "SELECT schemaname, relname, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC LIMIT 5")
    
    echo "Verification PASSED"
    echo "Tables: $table_count"
    echo "$row_counts"
else
    echo "Verification FAILED"
fi

# Step 4: Always cleanup test database
# dropdb = PostgreSQL utility to remove database
dropdb -U "$PG_USER" "$TEST_DB"
```

**Example Execution:**
```
$ ./verify_backup.sh /backup/postgresql/production_2026-01-22_1030.dump
=== Backup Verification Started ===
Restoring to test database...
Verification PASSED
Tables:      45

 schemaname |    relname     | n_live_tup 
------------+----------------+------------
 public     | audit_logs     |   12450000
 public     | transactions   |    8230000
 public     | users          |     450000
 public     | sessions       |     125000
 public     | products       |      85000
```

**Why Use Timestamps for Test DB Names:**
```bash
# Problem: Fixed name could conflict
TEST_DB="backup_test"
# If script runs twice, second run fails: "database already exists"

# Solution: Include timestamp
TEST_DB="backup_test_$(date +%s)"
# Creates unique names:
# backup_test_1706000000
# backup_test_1706000060  (1 minute later)

# date +%s = Unix timestamp (seconds since Jan 1, 1970)
```

**Adding More Integrity Checks:**
```bash
# Verify specific critical tables exist
psql -U "$PG_USER" -d "$TEST_DB" -c "\dt users" > /dev/null 2>&1 || echo "MISSING: users table"
psql -U "$PG_USER" -d "$TEST_DB" -c "\dt orders" > /dev/null 2>&1 || echo "MISSING: orders table"

# Verify foreign key constraints
psql -U "$PG_USER" -d "$TEST_DB" -c "SELECT COUNT(*) FROM information_schema.table_constraints WHERE constraint_type='FOREIGN KEY'"

# Verify indexes exist
psql -U "$PG_USER" -d "$TEST_DB" -c "SELECT COUNT(*) FROM pg_indexes WHERE schemaname='public'"
```

---

### 20. Disaster Recovery Sync

**🎯 Problem Statement:**
Critical data must be replicated to a disaster recovery site. We need continuous synchronization with bandwidth limits to avoid network saturation.

**🧠 Core Logic Explained:**
1. **Use rsync for sync** → Efficient delta transfer
2. **Limit bandwidth** → `--bwlimit` prevents network saturation
3. **Use checksums** → `--checksum` verifies file integrity
4. **Delete removed files** → `--delete` keeps DR in sync
5. **Verify file counts** → Compare local vs remote counts
6. **Log everything** → Audit trail for compliance

**💡 Key Concepts:**
- `rsync -avz` → Archive mode, verbose, compress
- `--checksum` → Compare by checksum instead of time/size
- `--bwlimit=N` → Limit bandwidth to N KB/s
- `--delete` → Delete files on destination not in source
- File count verification → Simple but effective integrity check

```bash
#!/bin/bash
# Disaster Recovery Sync - Replicate critical data to DR site

SOURCE="/data/critical"                          # Source directory to replicate
DR_SERVER="dr-server.example.com"                # DR site server
DR_PATH="/data/replicated"                       # Destination path on DR
BANDWIDTH="10000"                                # KB/s limit (10 MB/s)
LOG="/var/log/dr_sync.log"                       # Log file for audit

echo "=== DR Sync Started: $(date) ===" >> "$LOG"

# Sync with bandwidth limit and checksum verification
# rsync flags:
#   -a = Archive mode (preserves permissions, timestamps, symlinks, etc.)
#   -v = Verbose output
#   -z = Compress during transfer
#   --checksum = Compare by file checksum, not just size/time
#   --delete = Delete files on dest that don't exist in source
#   --bwlimit = Limit transfer speed to prevent network saturation
#   --log-file = Write detailed log
rsync -avz --checksum --delete \
    --bwlimit=$BANDWIDTH \
    --log-file="$LOG" \
    "$SOURCE/" "$DR_SERVER:$DR_PATH/"            # Trailing / = contents, not directory

exit_code=$?                                     # Capture rsync exit code

# Verify sync by comparing file counts
if [ $exit_code -eq 0 ]; then
    # Count files locally
    local_count=$(find "$SOURCE" -type f | wc -l)
    
    # Count files on remote via SSH
    remote_count=$(ssh "$DR_SERVER" "find $DR_PATH -type f | wc -l")
    
    # Compare counts
    if [ "$local_count" -eq "$remote_count" ]; then
        echo "DR Sync VERIFIED: $local_count files" >> "$LOG"
    else
        echo "WARNING: File count mismatch (local: $local_count, remote: $remote_count)" >> "$LOG"
    fi
else
    echo "ERROR: DR Sync failed with exit code $exit_code" >> "$LOG"
fi
```

**Example Execution:**
```
$ ./dr_sync.sh
(output goes to log file)

$ tail -f /var/log/dr_sync.log
=== DR Sync Started: Wed Jan 22 02:00:00 UTC 2026 ===
sending incremental file list
data/critical/database.dump
data/critical/configs/app.conf
sent 1,234,567 bytes  received 456 bytes  limited to 10000 KB/s
DR Sync VERIFIED: 15234 files
```

**rsync Trailing Slash Behavior:**
```bash
# SOURCE with trailing slash: Copy CONTENTS of directory
rsync /data/critical/ remote:/data/replicated/
# Result: Files from critical/ appear directly in replicated/

# SOURCE without trailing slash: Copy directory itself
rsync /data/critical remote:/data/replicated/
# Result: Creates replicated/critical/ containing the files

Always use trailing slash for DR sync to avoid nested directories!
```

**Why --checksum?**
```
Default rsync behavior:
- Compare by size + modification time
- Fast, but can miss silent corruption

With --checksum:
- Compare by file content hash
- Slower, but catches:
  - Bit rot (silent data corruption)
  - Files modified without timestamp change
  - Corruption during previous transfer

Use --checksum for critical data, skip for large low-risk transfers
```

**Bandwidth Calculation:**
```
--bwlimit=10000 (KB/s)

10000 KB/s = 10 MB/s = 80 Mbps

If you have 1 Gbps link and want to use 50%:
1 Gbps = 1000 Mbps = 125 MB/s = 128000 KB/s
50% = 64000 KB/s → --bwlimit=64000
```

---

### 21. Blue-Green Deployment Switch

**🎯 Problem Statement:**
Zero-downtime deployments require running two environments (blue/green). We need to safely switch traffic between them after verifying the new environment is healthy.

**🧠 Core Logic Explained:**
1. **Read current environment** → Know which is active (blue or green)
2. **Determine new environment** → Switch to the other one
3. **Health check new env** → Multiple checks to ensure stability
4. **Update load balancer** → Change nginx upstream server
5. **Verify nginx config** → Test before reload
6. **Rollback on failure** → Revert if anything goes wrong

**💡 Key Concepts:**
- Blue-Green → Two identical environments, switch traffic instantly
- Health check loop → Multiple checks reduce false positives
- `nginx -t` → Test configuration syntax before reload
- `sed -i` → In-place file editing
- `git checkout` → Quick rollback to previous config version

```bash
#!/bin/bash
# Blue-Green Deployment Switch - Zero-downtime environment switch

CURRENT_ENV=$(cat /etc/active_environment)       # Read current: "blue" or "green"
NGINX_CONF="/etc/nginx/conf.d/app.conf"          # Nginx config to update
HEALTH_ENDPOINT="/health"                        # Health check path

# Determine which environment to switch TO
# Simple toggle: if blue → green, if green → blue
if [ "$CURRENT_ENV" == "blue" ]; then
    NEW_ENV="green"
    NEW_UPSTREAM="10.0.2.0"                      # Green environment IP
else
    NEW_ENV="blue"
    NEW_UPSTREAM="10.0.1.0"                      # Blue environment IP
fi

echo "Switching from $CURRENT_ENV to $NEW_ENV..."

# Health check new environment (5 consecutive checks)
# Multiple checks reduce false positives from temporary issues
for i in {1..5}; do
    # curl flags:
    #   -s = Silent (no progress)
    #   -o /dev/null = Discard body
    #   -w "%{http_code}" = Output only HTTP status code
    status=$(curl -s -o /dev/null -w "%{http_code}" "http://$NEW_UPSTREAM$HEALTH_ENDPOINT")
    
    if [ "$status" == "200" ]; then
        echo "Health check $i: OK"
    else
        echo "Health check failed. Aborting deployment."
        exit 1
    fi
    sleep 2                                      # Wait between checks
done

# Update nginx configuration to point to new environment
# sed -i = In-place edit (modify file directly)
# Pattern: Change server IP from 10.0.1.0 or 10.0.2.0 to new IP
sed -i "s/server 10.0.[12].0/server $NEW_UPSTREAM/" "$NGINX_CONF"

# Reload nginx (test config first!)
# nginx -t = Test configuration syntax
# && = Only reload if test passes
nginx -t && systemctl reload nginx

if [ $? -eq 0 ]; then
    # Record new active environment
    echo "$NEW_ENV" > /etc/active_environment
    echo "Successfully switched to $NEW_ENV"
else
    echo "Failed to reload nginx. Rolling back..."
    # Restore previous config from git
    git checkout "$NGINX_CONF"
    exit 1
fi
```

**Example Execution:**
```
$ ./blue_green_switch.sh
Switching from blue to green...
Health check 1: OK
Health check 2: OK
Health check 3: OK
Health check 4: OK
Health check 5: OK
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
Successfully switched to green
```

**Blue-Green Architecture:**
```
               Load Balancer (nginx)
                      │
         ┌────────────┴────────────┐
         ▼                         ▼
    ┌─────────┐              ┌─────────┐
    │  BLUE   │              │ GREEN   │
    │ 10.0.1.0│              │ 10.0.2.0│
    │ (v1.2)  │              │ (v1.3)  │
    └─────────┘              └─────────┘
      ACTIVE ←─────switch─────→ INACTIVE

Before switch: Traffic → Blue (v1.2)
After switch:  Traffic → Green (v1.3)
Rollback:      Traffic → Blue (v1.2) instantly!
```

**Why Multiple Health Checks?**
```
Single check problem:
- Server might return 200 once but crash immediately after
- Network blip could cause false negative

5 consecutive checks (10 seconds total):
- Proves stability over time
- High confidence service is truly healthy
- Catches startup issues that appear after few seconds
```

**Nginx Config Change Example:**
```nginx
# Before (app.conf):
upstream backend {
    server 10.0.1.0:8080;  # Blue
}

# After sed command:
upstream backend {
    server 10.0.2.0:8080;  # Green
}
```

---

### 22. Rolling Restart Script

**🎯 Problem Statement:**
Restarting all servers at once causes downtime. We need to restart services one at a time while ensuring the service remains available throughout.

**🧠 Core Logic Explained:**
1. **Process one server at a time** → Never take all down simultaneously
2. **Drain connections** → Signal server to stop accepting new requests
3. **Wait for drain** → Give existing requests time to complete
4. **Restart service** → Apply updates/changes
5. **Health check loop** → Wait until server is healthy
6. **Remove drain flag** → Allow new traffic
7. **Continue to next** → Only after current is healthy

**💡 Key Concepts:**
- Rolling restart → One at a time maintains availability
- Connection draining → Graceful handling of in-flight requests
- `printf` → Format strings with variables
- `seq 1 N` → Generate sequence of numbers
- Health check polling → Retry until healthy or timeout

```bash
#!/bin/bash
# Rolling Restart - Zero-downtime service restart across servers

SERVERS="app1 app2 app3 app4"                    # List of servers to restart
SERVICE="myapp"                                  # Service name
HEALTH_URL_TEMPLATE="http://%s:8080/health"      # Health check URL template
DRAIN_TIME=30                                    # Seconds to drain connections
HEALTH_TIMEOUT=60                                # Max seconds to wait for healthy

# Process one server at a time (maintains availability)
for server in $SERVERS; do
    echo "=== Processing $server ==="
    
    # Step 1: Drain connections
    # Touch file tells app to stop accepting NEW requests
    # Existing requests continue to be served
    echo "Draining connections ($DRAIN_TIME seconds)..."
    ssh "$server" "touch /tmp/drain_flag"
    sleep $DRAIN_TIME                            # Wait for in-flight requests
    
    # Step 2: Restart service
    echo "Restarting $SERVICE..."
    ssh "$server" "systemctl restart $SERVICE"
    
    # Step 3: Wait for healthy
    # printf substitutes %s with server name
    health_url=$(printf "$HEALTH_URL_TEMPLATE" "$server")
    echo "Waiting for $server to become healthy..."
    
    # Poll health endpoint up to HEALTH_TIMEOUT seconds
    for i in $(seq 1 $HEALTH_TIMEOUT); do
        # Check if health endpoint returns "ok"
        if curl -s "$health_url" | grep -q "ok"; then
            echo "$server is healthy after $i seconds"
            # Remove drain flag to accept new traffic
            ssh "$server" "rm -f /tmp/drain_flag"
            break                                # Exit loop, move to next server
        fi
        
        # Timeout reached - abort!
        if [ $i -eq $HEALTH_TIMEOUT ]; then
            echo "ERROR: $server failed to become healthy"
            exit 1
        fi
        sleep 1                                  # Wait 1 second before retry
    done
    
    echo "$server complete. Moving to next..."
done

echo "Rolling restart complete!"
```

**Example Execution:**
```
$ ./rolling_restart.sh
=== Processing app1 ===
Draining connections (30 seconds)...
Restarting myapp...
Waiting for app1 to become healthy...
app1 is healthy after 8 seconds
app1 complete. Moving to next...
=== Processing app2 ===
Draining connections (30 seconds)...
Restarting myapp...
Waiting for app2 to become healthy...
app2 is healthy after 5 seconds
app2 complete. Moving to next...
=== Processing app3 ===
...
Rolling restart complete!
```

**Why Rolling Restart?**
```
4 servers handling traffic:
[app1] [app2] [app3] [app4]   ← All handling requests

Standard restart (BAD):
[DOWN] [DOWN] [DOWN] [DOWN]   ← 100% downtime!

Rolling restart (GOOD):
[DOWN] [app2] [app3] [app4]   ← 75% capacity
[app1] [DOWN] [app3] [app4]   ← 75% capacity
[app1] [app2] [DOWN] [app4]   ← 75% capacity
[app1] [app2] [app3] [DOWN]   ← 75% capacity
[app1] [app2] [app3] [app4]   ← 100% capacity restored

Users never experience complete outage!
```

**Connection Draining Explained:**
```
Without draining:
User request → app1 → RESTART → Connection dropped! (Error)

With draining:
1. touch /tmp/drain_flag
2. App checks flag, stops accepting NEW requests
3. Existing requests complete normally
4. sleep 30 (wait for in-flight to finish)
5. RESTART (no active connections)
6. rm /tmp/drain_flag (accept new requests)

Result: Zero dropped connections
```

**printf for URL Building:**
```bash
HEALTH_URL_TEMPLATE="http://%s:8080/health"

printf "$HEALTH_URL_TEMPLATE" "app1"
# → http://app1:8080/health

printf "$HEALTH_URL_TEMPLATE" "app2"  
# → http://app2:8080/health

# %s = String placeholder, replaced with argument
```

---

### 23. Git Hook Pre-commit Validator

**🎯 Problem Statement:**
Prevent bad code from being committed: secrets, debug statements, syntax errors. We need automated checks that run before every commit.

**🧠 Core Logic Explained:**
1. **Run on pre-commit** → Git hook executes automatically
2. **Get staged files** → Only check files being committed
3. **Check for secrets** → Regex match for passwords/API keys
4. **Check for debug code** → Find breakpoints, print statements
5. **Syntax validation** → Check shell scripts, YAML files
6. **Block on failure** → Exit 1 prevents commit

**💡 Key Concepts:**
- Git hooks → Scripts in `.git/hooks/` that run on events
- `git diff --cached --name-only` → List staged files
- `xargs` → Pass stdin as arguments to command
- `grep -qE` → Quiet mode, extended regex
- Exit codes → 0 allows commit, non-zero blocks it

```bash
#!/bin/bash
# Git Pre-commit Hook - Prevent bad code from being committed
# Save as: .git/hooks/pre-commit (must be executable: chmod +x)

echo "Running pre-commit checks..."

# Check 1: Detect potential secrets/passwords in staged files
# git diff --cached --name-only = List files staged for commit
# xargs = Pass file names as arguments to grep
# grep -l = List files containing matches (not the matches)
# grep -E = Extended regex (allows |, +, etc.)
# Pattern: variable = "value" format for common secret names
if git diff --cached --name-only | xargs grep -l -E \
    "(password|secret|api_key)\s*=\s*['\"][^'\"]+['\"]" 2>/dev/null; then
    echo "ERROR: Potential secrets detected in staged files!"
    exit 1                                       # Exit 1 = Block commit
fi

# Check 2: Find debug statements in Python files
# -- '*.py' = Only Python files
# Pattern: import pdb or breakpoint() (Python debugger)
if git diff --cached --name-only -- '*.py' | xargs grep -l \
    "import pdb\|breakpoint()" 2>/dev/null; then
    echo "ERROR: Debug statements found in Python files!"
    exit 1
fi

# Check 3: Syntax check all staged shell scripts
for file in $(git diff --cached --name-only -- '*.sh'); do
    # bash -n = Check syntax only (don't execute)
    if ! bash -n "$file"; then
        echo "ERROR: Syntax error in $file"
        exit 1
    fi
done

# Check 4: Validate YAML syntax
for file in $(git diff --cached --name-only -- '*.yml' '*.yaml'); do
    # Use Python to parse YAML (validates syntax)
    if ! python -c "import yaml; yaml.safe_load(open('$file'))" 2>/dev/null; then
        echo "ERROR: Invalid YAML in $file"
        exit 1
    fi
done

echo "All pre-commit checks passed!"
exit 0                                           # Exit 0 = Allow commit
```

**Example Execution:**
```
$ git add config.py
$ git commit -m "Update config"
Running pre-commit checks...
ERROR: Potential secrets detected in staged files!
config.py                        ← Shows file with secrets

# Fix the issue, then retry:
$ git commit -m "Update config"
Running pre-commit checks...
All pre-commit checks passed!
[main abc1234] Update config
```

**Installing the Hook:**
```bash
# Option 1: Manual
cp pre-commit.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

# Option 2: Shared hooks (in repo)
mkdir -p .githooks
cp pre-commit.sh .githooks/pre-commit
git config core.hooksPath .githooks
```

**Secret Detection Patterns:**
```
Pattern: (password|secret|api_key)\s*=\s*['\"][^'\"]+['\"]

Matches:
  password = "myp@ssw0rd"        ✓ BLOCKED
  secret="abc123"                ✓ BLOCKED
  api_key = 'sk-12345abcde'      ✓ BLOCKED

Doesn't match:
  password = os.getenv("PASS")   ✗ (no literal value)
  # password = "example"         ✗ (commented out)
  password = ""                  ✗ (empty string)
```

**Why Check Staged Files Only:**
```bash
# git diff --cached --name-only
# Shows ONLY files staged for THIS commit

Working directory might have:
- app.py (modified, not staged)
- config.py (modified AND staged)  ← Only this is checked

This prevents checking files you're still working on!
```

---

### 24. Build Artifact Versioning

**🎯 Problem Statement:**
CI/CD needs consistent version numbers. We need to automatically generate semantic versions based on git tags and commit messages.

**🧠 Core Logic Explained:**
1. **Get last git tag** → Starting point for version calculation
2. **Parse semantic version** → Split into major.minor.patch
3. **Analyze commit messages** → Determine bump type
4. **Apply version bump** → BREAKING=major, feat=minor, else=patch
5. **Add build metadata** → Commit hash, build number
6. **Export for CI** → Write to environment variables

**💡 Key Concepts:**
- Semantic versioning → MAJOR.MINOR.PATCH format
- `git describe --tags` → Get most recent tag
- `${LAST_TAG#v}` → Remove 'v' prefix from string
- `IFS='.' read` → Split string by delimiter into variables
- `grep -qi` → Case-insensitive quiet grep

```bash
#!/bin/bash
# Build Artifact Versioning - Automatic semantic version from git

# Get most recent git tag (default to v0.0.0 if none)
# git describe --tags = Describe current commit using tags
# --abbrev=0 = Don't include commit count suffix
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")

# Remove 'v' prefix from tag
# ${variable#pattern} = Remove prefix matching pattern
VERSION=${LAST_TAG#v}                            # v1.2.3 → 1.2.3

# Parse semantic version into components
# IFS='.' = Use . as field separator
# read -r major minor patch = Read into three variables
IFS='.' read -r major minor patch <<< "$VERSION" # 1.2.3 → major=1, minor=2, patch=3

# Determine version bump type from commit messages since last tag
# grep -qi = Quiet (no output), case-insensitive
if git log --oneline "$LAST_TAG"..HEAD | grep -qi "BREAKING\|major"; then
    # BREAKING CHANGE → Major bump (1.0.0 → 2.0.0)
    # Reset minor and patch to 0
    NEW_VERSION="$((major+1)).0.0"
elif git log --oneline "$LAST_TAG"..HEAD | grep -qi "feat\|feature"; then
    # New feature → Minor bump (1.2.0 → 1.3.0)
    # Reset patch to 0
    NEW_VERSION="$major.$((minor+1)).0"
else
    # Bug fix or other → Patch bump (1.2.3 → 1.2.4)
    NEW_VERSION="$major.$minor.$((patch+1))"
fi

# Add build metadata
# git rev-parse --short HEAD = Get short commit hash (7 chars)
COMMIT_SHORT=$(git rev-parse --short HEAD)
# ${VAR:-default} = Use default if VAR is empty
BUILD_NUM=${BUILD_NUMBER:-local}                 # CI provides BUILD_NUMBER
FULL_VERSION="${NEW_VERSION}+build.${BUILD_NUM}.${COMMIT_SHORT}"

echo "Previous: $VERSION"
echo "New: $FULL_VERSION"

# Export for use in GitHub Actions CI
# >> "$GITHUB_ENV" = Append to GitHub environment file
# || true = Don't fail if not in CI
echo "VERSION=$NEW_VERSION" >> "$GITHUB_ENV" 2>/dev/null || true
echo "FULL_VERSION=$FULL_VERSION" >> "$GITHUB_ENV" 2>/dev/null || true
```

**Example Execution:**
```
$ git log --oneline v1.2.3..HEAD
abc123 feat: Add user authentication
def456 fix: Correct date formatting
ghi789 docs: Update README

$ ./version.sh
Previous: 1.2.3
New: 1.3.0+build.42.abc123d

# Because "feat" was found → minor bump (1.2.3 → 1.3.0)
```

**Semantic Version Rules:**
```
Format: MAJOR.MINOR.PATCH

MAJOR: Breaking changes (incompatible API changes)
       Commit: "BREAKING: Remove deprecated API"
       1.2.3 → 2.0.0

MINOR: New features (backwards compatible)
       Commit: "feat: Add dark mode support"
       1.2.3 → 1.3.0

PATCH: Bug fixes (backwards compatible)
       Commit: "fix: Correct calculation error"
       1.2.3 → 1.2.4
```

**Build Metadata:**
```
Version: 1.3.0+build.42.abc123d
         │     │     │    │
         │     │     │    └── Short commit hash
         │     │     └── CI build number
         │     └── Metadata separator
         └── Semantic version

Build metadata is informational only
1.3.0+build.42 == 1.3.0+build.99 (same version!)
```

**Using in CI (GitHub Actions):**
```yaml
- name: Generate version
  run: ./version.sh
  
- name: Build Docker image
  run: docker build -t myapp:${{ env.VERSION }} .
  
- name: Push with full version tag
  run: docker push myapp:${{ env.FULL_VERSION }}
```

# Tag artifact
echo "$FULL_VERSION" > VERSION.txt
```

---

### 25. Environment Config Generator

**🎯 Problem Statement:**
Applications need different configurations for dev/staging/production. We need to generate config files from templates using environment-specific variables.

**🧠 Core Logic Explained:**
1. **Accept environment as argument** → dev, staging, prod
2. **Load environment variables** → Source the env-specific file
3. **Validate required vars** → Fail fast if anything missing
4. **Substitute variables** → Replace ${VAR} with actual values
5. **Verify no leftovers** → Catch any unreplaced placeholders
6. **Secure the output** → Restrict file permissions

**💡 Key Concepts:**
- `envsubst` → Substitute environment variables in text
- `source` → Load variables from file into current shell
- `${!var}` → Indirect variable reference (value of var named $var)
- Template files → Use ${VAR} placeholders
- `chmod 600` → Owner read/write only (secure config)

```bash
#!/bin/bash
# Environment Config Generator - Create configs from templates

ENV=${1:-development}                            # Default to development
TEMPLATE="config/app.conf.template"              # Template file with ${VAR} placeholders
OUTPUT="config/app.conf"                         # Generated config file

# Load environment-specific variables
# source = Execute file in current shell (makes vars available)
# Different files for each env: development.env, staging.env, production.env
source "config/${ENV}.env"

# Required variables check - fail fast if anything missing
REQUIRED_VARS="DB_HOST DB_NAME API_KEY"
for var in $REQUIRED_VARS; do
    # ${!var} = Indirect reference: if var="DB_HOST", ${!var} = ${DB_HOST}
    # This dereferences the variable name stored in var
    if [ -z "${!var}" ]; then                    # -z = String is empty
        echo "ERROR: $var is not set"
        exit 1
    fi
done

# Generate config from template
# envsubst = Replace ${VAR} with actual environment variable values
# < "$TEMPLATE" = Read template as stdin
# > "$OUTPUT" = Write result to output file
envsubst < "$TEMPLATE" > "$OUTPUT"

# Verify no unreplaced variables remain
# Pattern '\${' matches literal ${ (unreplaced placeholder)
if grep -q '\${' "$OUTPUT"; then
    echo "WARNING: Unreplaced variables in config:"
    grep '\${' "$OUTPUT"                         # Show which ones
    exit 1
fi

echo "Generated $OUTPUT for $ENV environment"
# Secure the config file (contains secrets!)
chmod 600 "$OUTPUT"                              # -rw------- (owner only)
```

**Example Execution:**
```
$ ./generate_config.sh production
Generated config/app.conf for production environment

$ cat config/app.conf
database:
  host: prod-db.example.com
  name: production_db
  
api:
  key: sk-prod-12345abcde
```

**Template Example (app.conf.template):**
```
database:
  host: ${DB_HOST}
  name: ${DB_NAME}
  
api:
  key: ${API_KEY}
  timeout: ${API_TIMEOUT:-30}
```

**Environment File Example (production.env):**
```bash
DB_HOST=prod-db.example.com
DB_NAME=production_db
API_KEY=sk-prod-12345abcde
API_TIMEOUT=60
```

**Indirect Variable Reference:**
```bash
# Problem: How to check if variable NAME stored in another variable is set?
var="DB_HOST"        # var contains the STRING "DB_HOST"
DB_HOST="localhost"  # DB_HOST contains "localhost"

# Wrong: This checks if "var" is set (it is)
if [ -z "$var" ]; then ...

# Right: ${!var} = "Dereference" - get value of variable named $var
echo ${!var}         # → localhost (value of DB_HOST)

if [ -z "${!var}" ]; then
    # DB_HOST is empty!
fi
```

---

### 26. Docker Image Cleanup

**🎯 Problem Statement:**
Docker accumulates unused images, containers, and volumes consuming disk space. We need to safely clean up unused resources while preserving what's in use.

**🧠 Core Logic Explained:**
1. **Show current usage** → `docker system df` displays disk usage
2. **Remove stopped containers** → Containers no longer running
3. **Remove dangling images** → Images not tagged or referenced
4. **Remove old images** → Images not used recently
5. **Remove unused volumes** → Orphaned data volumes
6. **Report savings** → Show freed space

**💡 Key Concepts:**
- `docker system df` → Shows Docker disk usage summary
- `docker prune` commands → Clean up unused resources
- Dangling images → `<none>:<none>` images (no tag)
- `-f` flag → Force, no confirmation prompt
- `xargs -r` → Don't run command if stdin is empty

```bash
#!/bin/bash
# Docker Cleanup Script - Free disk space safely

echo "=== Docker Cleanup Script ==="

# Show current disk usage
echo "Current Docker disk usage:"
docker system df                                 # Overview of disk usage

# Remove stopped containers
# docker ps -aq = All container IDs (quiet)
# -f status=exited = Filter to stopped containers only
stopped=$(docker ps -aq -f status=exited | wc -l)
if [ "$stopped" -gt 0 ]; then
    echo "Removing $stopped stopped containers..."
    docker container prune -f                    # -f = No confirmation prompt
fi

# Remove dangling images (images with no tag: <none>:<none>)
# These are usually old build layers no longer needed
dangling=$(docker images -f "dangling=true" -q | wc -l)
if [ "$dangling" -gt 0 ]; then
    echo "Removing $dangling dangling images..."
    docker image prune -f
fi

# Remove images older than 30 days (excluding latest tags)
# --format = Custom output format
# CreatedSince shows: "2 days ago", "3 weeks ago", "2 months ago"
echo "Removing images older than 30 days..."
docker images --format "{{.ID}} {{.CreatedSince}}" | \
    grep -E "months|weeks" | \                   # Match old images
    awk '{print $1}' | \                         # Extract image ID
    xargs -r docker rmi 2>/dev/null              # -r = Skip if empty

# Remove unused volumes
# Volumes not attached to any container
echo "Removing unused volumes..."
docker volume prune -f

# Final report
echo -e "\n=== Cleanup Complete ==="
docker system df
```

**Example Output:**
```
$ ./docker_cleanup.sh
=== Docker Cleanup Script ===
Current Docker disk usage:
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          45        12        15.2GB    8.5GB (55%)
Containers      23        5         2.1GB     1.8GB (85%)
Volumes         18        3         5.4GB     4.2GB (77%)

Removing 18 stopped containers...
Deleted Containers: 18
Total reclaimed space: 1.8GB

Removing 12 dangling images...
Deleted Images: 12
Total reclaimed space: 3.2GB

Removing images older than 30 days...
Removing unused volumes...
Deleted Volumes: 15
Total reclaimed space: 4.2GB

=== Cleanup Complete ===
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          21        12        6.7GB     0B (0%)
Containers      5         5         0B        0B
Volumes         3         3         1.2GB     0B (0%)
```

**What Gets Cleaned:**
```
Stopped Containers: Containers that have exited/stopped
  [RUNNING] webapp-1    ← KEPT
  [STOPPED] webapp-2    ← REMOVED
  [STOPPED] test-build  ← REMOVED

Dangling Images: Untagged images (old layers)
  myapp:latest         ← KEPT (has tag)
  <none>:<none>        ← REMOVED (dangling)
  
Unused Volumes: Not attached to any container
  db-data (in use)     ← KEPT
  old-backup (orphan)  ← REMOVED
```

---

### 27. Kubernetes Pod Restart

**🎯 Problem Statement:**
Sometimes pods need restarting to pick up config changes or clear memory leaks. We need to gracefully restart pods while maintaining availability.

**🧠 Core Logic Explained:**
1. **Accept namespace and deployment** → Target specific workload
2. **Validate inputs** → Show available options if missing
3. **Show current state** → Display existing pods
4. **Trigger rolling restart** → `kubectl rollout restart`
5. **Wait for completion** → Monitor rollout status
6. **Rollback on failure** → Undo if restart fails

**💡 Key Concepts:**
- `kubectl rollout restart` → Restarts pods one by one
- `kubectl rollout status` → Wait for rollout to complete
- `kubectl rollout undo` → Revert to previous version
- `--timeout` → Fail if not complete within time
- `-n namespace` → Specify Kubernetes namespace

```bash
#!/bin/bash
# Kubernetes Pod Restart - Graceful rolling restart

NAMESPACE=${1:-default}                          # Default to "default" namespace
DEPLOYMENT=${2:-""}                              # Deployment name (required)

# Validate deployment argument
if [ -z "$DEPLOYMENT" ]; then
    echo "Usage: $0 <namespace> <deployment>"
    echo "Available deployments:"
    # List available deployments in namespace
    kubectl get deployments -n "$NAMESPACE" -o name
    exit 1
fi

echo "Restarting $DEPLOYMENT in $NAMESPACE..."

# Check current status - show pods before restart
# -l = Label selector (assumes app=deployment-name pattern)
# -o wide = Show additional columns (node, IP)
kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o wide

# Perform rolling restart
# This adds a new annotation triggering a gradual pod replacement
# Old pods kept running until new pods are ready
kubectl rollout restart deployment/"$DEPLOYMENT" -n "$NAMESPACE"

# Watch rollout status - waits for completion
# --timeout=300s = Fail if not complete in 5 minutes
kubectl rollout status deployment/"$DEPLOYMENT" -n "$NAMESPACE" --timeout=300s

if [ $? -eq 0 ]; then
    echo "Restart successful!"
    # Show new pods
    kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o wide
else
    echo "Restart failed. Rolling back..."
    # Undo = Revert to previous ReplicaSet
    kubectl rollout undo deployment/"$DEPLOYMENT" -n "$NAMESPACE"
    exit 1
fi
```

**Example Execution:**
```
$ ./k8s_restart.sh production webapp
Restarting webapp in production...
NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE
webapp-6d4f5c8b9-abc12    1/1     Running   0          5d    10.0.1.100   node-1
webapp-6d4f5c8b9-def34    1/1     Running   0          5d    10.0.1.101   node-2
webapp-6d4f5c8b9-ghi56    1/1     Running   0          5d    10.0.1.102   node-3

deployment.apps/webapp restarted
Waiting for deployment "webapp" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "webapp" rollout to finish: 3 out of 3 new replicas have been updated...
deployment "webapp" successfully rolled out
Restart successful!

NAME                      READY   STATUS    RESTARTS   AGE   IP           NODE
webapp-7e5f6d9c0-jkl78    1/1     Running   0          30s   10.0.1.103   node-1
webapp-7e5f6d9c0-mno90    1/1     Running   0          45s   10.0.1.104   node-2
webapp-7e5f6d9c0-pqr12    1/1     Running   0          60s   10.0.1.105   node-3
```

**Rolling Restart Process:**
```
Before restart:
[pod-old-1] [pod-old-2] [pod-old-3]   ← All serving traffic

kubectl rollout restart:
[pod-old-1] [pod-old-2] [pod-new-1]   ← New pod starting
[pod-old-1] [pod-new-1] [pod-new-2]   ← Gradual replacement
[pod-new-1] [pod-new-2] [pod-new-3]   ← All new, old terminated

Traffic served throughout - zero downtime!
```

**Why Rollback on Failure?**
```
Possible failures:
- New image crashes on startup
- Resource limits exceeded
- Config errors cause crash loop

Without rollback:
Partial deployment stuck = Degraded service

With rollback:
Failure detected → kubectl rollout undo → Previous pods restored
Service returns to known-good state automatically
```

---

### 28. Helm Release Manager

**🎯 Problem Statement:**
Deploying Helm charts requires validation and safe rollback. We need a script that lints, shows changes, and deploys with automatic rollback on failure.

**🧠 Core Logic Explained:**
1. **Accept release parameters** → Name, chart, namespace, values
2. **Lint the chart** → Catch syntax/schema errors early
3. **Show diff** → Preview what will change before deploying
4. **Confirm deployment** → Interactive approval
5. **Deploy atomically** → All-or-nothing with auto-rollback
6. **Show final status** → Display deployed release info

**💡 Key Concepts:**
- `helm lint` → Validate chart syntax and best practices
- `helm diff` → Show what will change (requires plugin)
- `--atomic` → Auto-rollback entire release on failure
- `--wait` → Wait for all resources to be ready
- `helm upgrade --install` → Install if new, upgrade if exists

```bash
#!/bin/bash
# Helm Release Manager - Safe Helm deployments with validation

RELEASE_NAME="$1"                                # Helm release name
CHART_PATH="$2"                                  # Path to chart or chart name
NAMESPACE="${3:-default}"                        # Kubernetes namespace
VALUES_FILE="${4:-values.yaml}"                  # Values override file

# Validate required arguments
if [ -z "$RELEASE_NAME" ] || [ -z "$CHART_PATH" ]; then
    echo "Usage: $0 <release_name> <chart_path> [namespace] [values_file]"
    exit 1
fi

echo "=== Helm Release Manager ==="
echo "Release: $RELEASE_NAME"
echo "Chart: $CHART_PATH"
echo "Namespace: $NAMESPACE"

# Step 1: Lint chart - catch errors before deploying
# helm lint = Validates chart structure and values
echo -e "\n--- Linting chart ---"
helm lint "$CHART_PATH" -f "$VALUES_FILE"

# Step 2: Show diff - preview what will change
# helm-diff plugin required: helm plugin install https://github.com/databus23/helm-diff
# --allow-unreleased = Works for new installs too
echo -e "\n--- Changes preview ---"
helm diff upgrade "$RELEASE_NAME" "$CHART_PATH" \
    -n "$NAMESPACE" \
    -f "$VALUES_FILE" \
    --allow-unreleased 2>/dev/null || echo "Install helm-diff for change preview"

# Step 3: Confirm deployment (interactive)
read -p "Proceed with deployment? (y/n): " confirm
if [ "$confirm" != "y" ]; then
    echo "Deployment cancelled"
    exit 0
fi

# Step 4: Deploy with atomic flag (auto-rollback on failure)
# --install = Install if not exists (upgrade --install = upsert)
# --atomic = If deployment fails, rollback entire release
# --timeout = Max time to wait for completion
# --wait = Wait for all pods to be ready before marking success
helm upgrade --install "$RELEASE_NAME" "$CHART_PATH" \
    -n "$NAMESPACE" \
    -f "$VALUES_FILE" \
    --atomic \
    --timeout 10m \
    --wait

if [ $? -eq 0 ]; then
    echo "Deployment successful!"
    helm status "$RELEASE_NAME" -n "$NAMESPACE"  # Show release details
else
    echo "Deployment failed (auto-rolled back)"
    exit 1
fi
```

**Example Execution:**
```
$ ./helm_manager.sh webapp ./charts/webapp production values-prod.yaml
=== Helm Release Manager ===
Release: webapp
Chart: ./charts/webapp
Namespace: production

--- Linting chart ---
==> Linting ./charts/webapp
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed

--- Changes preview ---
production, webapp, Deployment (apps) has changed:
  # Source: webapp/templates/deployment.yaml
  spec:
    replicas: 3
+   replicas: 5
    containers:
      - image: webapp:1.2.3
+     - image: webapp:1.3.0

Proceed with deployment? (y/n): y
Release "webapp" has been upgraded. Happy Helming!
Deployment successful!
NAME: webapp
NAMESPACE: production
STATUS: deployed
REVISION: 5
```

**Atomic Deployment Explained:**
```
Without --atomic:
Step 1: Update ConfigMap     ✓
Step 2: Update Deployment    ✓
Step 3: Update Service       ✗ (fails)
Result: Partial deployment! Some resources updated, some not.

With --atomic:
Step 1: Update ConfigMap     ✓
Step 2: Update Deployment    ✓
Step 3: Update Service       ✗ (fails)
→ Automatic rollback ALL resources to previous state
Result: Clean rollback, no partial deployment
```

**Why --wait Matters:**
```
Without --wait:
helm upgrade completes when Kubernetes ACCEPTS the manifests
Pods might still be starting, crashing, or ImagePullBackOff

With --wait:
helm upgrade completes when pods are actually READY
Combined with --atomic: Rollback if pods never become ready
```

---

### 29. Failed SSH Login Detector

**🎯 Problem Statement:**
Brute force SSH attacks are common. We need to detect repeated failed login attempts and optionally block attacking IPs automatically.

**🧠 Core Logic Explained:**
1. **Parse auth logs** → Linux logs SSH attempts to auth.log
2. **Extract failed logins** → Grep for "Failed password"
3. **Extract IP addresses** → Use regex to find IPs
4. **Count per IP** → Sort and count unique IPs
5. **Block heavy offenders** → Add to hosts.deny if over threshold
6. **Geographic analysis** → Optional geo-location lookup

**💡 Key Concepts:**
- `/var/log/auth.log` → SSH authentication log (Debian/Ubuntu)
- `grep -oE` → Only print matching part, extended regex
- `[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+` → IPv4 address regex
- `/etc/hosts.deny` → TCP wrappers block list
- `geoiplookup` → Geographic IP lookup tool

```bash
#!/bin/bash
# Failed SSH Login Detector - Find brute force attacks

LOG="/var/log/auth.log"                          # SSH log location
THRESHOLD=10                                     # Block IPs with this many failures
BLOCK_LIST="/etc/hosts.deny"                     # TCP wrappers deny file

echo "=== Failed SSH Login Report ==="

# Find failed login attempts and count by IP
# grep "Failed password" = Find failed SSH logins
# grep -oE '[0-9]+...' = Extract only IP addresses (regex)
# sort | uniq -c = Count occurrences of each IP
# sort -rn = Sort by count, highest first
failed_ips=$(grep "Failed password" "$LOG" | \
    grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | \
    sort | uniq -c | sort -rn)

echo "IP Address       Attempts"
echo "-------------------------"
echo "$failed_ips" | head -20                    # Show top 20 offenders

# Block IPs exceeding threshold
echo -e "\n=== Blocking IPs with >$THRESHOLD attempts ==="
# Read count and IP from each line
echo "$failed_ips" | while read count ip; do
    if [ "$count" -gt "$THRESHOLD" ]; then
        # Check if already blocked
        if ! grep -q "$ip" "$BLOCK_LIST" 2>/dev/null; then
            echo "Blocking $ip ($count attempts)"
            # Add to hosts.deny (blocks SSH connections from this IP)
            echo "sshd: $ip" >> "$BLOCK_LIST"
        fi
    fi
done

# Summary by country (requires geoiplookup package)
echo -e "\n=== Geographic Distribution ==="
echo "$failed_ips" | awk '{print $2}' | while read ip; do
    # geoiplookup returns: "GeoIP Country Edition: US, United States"
    # cut extracts just the country name
    country=$(geoiplookup "$ip" 2>/dev/null | cut -d: -f2 | cut -d, -f1)
    echo "${country:-Unknown}"                   # Default to Unknown if lookup fails
done | sort | uniq -c | sort -rn | head -10
```

**Example Output:**
```
$ sudo ./ssh_detector.sh
=== Failed SSH Login Report ===
IP Address       Attempts
-------------------------
    847 185.234.218.41
    523 103.99.0.185
    312 195.54.160.149
    245 61.177.173.12
    189 218.92.0.163

=== Blocking IPs with >10 attempts ===
Blocking 185.234.218.41 (847 attempts)
Blocking 103.99.0.185 (523 attempts)
Blocking 195.54.160.149 (312 attempts)

=== Geographic Distribution ===
    523 CN (China)
    312 RU (Russia)
    245 KR (Korea)
    189 US (United States)
```

**IP Address Regex Explained:**
```
Pattern: [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+

[0-9]+  = One or more digits (0-9)
\.      = Literal dot (escaped)

Matches:
  192.168.1.1      ✓
  10.0.0.255       ✓
  8.8.8.8          ✓
  
Doesn't match:
  192.168.1        ✗ (only 3 octets)
  abc.def.ghi.jkl  ✗ (not digits)
```

**hosts.deny Format:**
```
# /etc/hosts.deny
sshd: 185.234.218.41      ← Block SSH from this IP
sshd: 103.99.0.185
ALL: 10.0.0.0/8           ← Block ALL services from this range
```

---

### 30. File Integrity Checker

**🎯 Problem Statement:**
Hackers may modify system files. We need to detect unauthorized changes to critical files like /etc configs and system binaries.

**🧠 Core Logic Explained:**
1. **Generate baseline** → Create MD5 checksums of all files
2. **Store baseline** → Save hashes for future comparison
3. **Check integrity** → Compare current hashes against baseline
4. **Detect modifications** → Hash mismatch = file changed
5. **Detect deletions** → File in baseline but missing
6. **Detect additions** → New file not in baseline

**💡 Key Concepts:**
- `md5sum` → Generate MD5 hash of file contents
- Baseline comparison → Store known-good state for comparison
- `IFS=' ' read -r hash file` → Parse space-separated fields
- `((changes++))` → Increment counter
- Functions in bash → `function_name() { }` for reusable code

```bash
#!/bin/bash
# File Integrity Checker - Detect unauthorized file changes

WATCH_DIRS="/etc /usr/bin /usr/sbin"             # Directories to monitor
BASELINE="/var/lib/integrity/baseline.md5"       # Stored checksums
ALERT_LOG="/var/log/integrity_alerts.log"        # Alert log file

# Function: Generate baseline checksums
generate_baseline() {
    echo "Generating baseline checksums..."
    > "$BASELINE"                                # Empty/create baseline file
    
    # Generate MD5 hash for every file in watched directories
    for dir in $WATCH_DIRS; do
        # find -type f = Regular files only
        # -exec md5sum {} \; = Run md5sum on each file
        find "$dir" -type f -exec md5sum {} \; >> "$BASELINE" 2>/dev/null
    done
    echo "Baseline saved: $(wc -l < "$BASELINE") files"
}

# Function: Check integrity against baseline
check_integrity() {
    # Verify baseline exists
    if [ ! -f "$BASELINE" ]; then
        echo "No baseline found. Generate first with: $0 --init"
        exit 1
    fi
    
    echo "Checking file integrity..."
    changes=0
    
    # Read each line from baseline: "hash  filepath"
    # IFS=' ' = Split by space
    while IFS=' ' read -r hash file; do
        if [ -f "$file" ]; then
            # Calculate current hash
            current_hash=$(md5sum "$file" 2>/dev/null | cut -d' ' -f1)
            
            # Compare hashes
            if [ "$hash" != "$current_hash" ]; then
                echo "MODIFIED: $file" | tee -a "$ALERT_LOG"
                ((changes++))                    # Increment counter
            fi
        else
            # File in baseline but doesn't exist anymore
            echo "DELETED: $file" | tee -a "$ALERT_LOG"
            ((changes++))
        fi
    done < "$BASELINE"
    
    # Check for new files not in baseline
    for dir in $WATCH_DIRS; do
        find "$dir" -type f | while read file; do
            # grep -q = Quiet mode, just check if found
            if ! grep -q "$file" "$BASELINE"; then
                echo "NEW: $file" | tee -a "$ALERT_LOG"
                ((changes++))
            fi
        done
    done
    
    echo "Integrity check complete. Changes detected: $changes"
}

# Main: Handle command line arguments
case "$1" in
    --init) generate_baseline ;;                 # Create baseline
    --check) check_integrity ;;                  # Verify integrity
    *) echo "Usage: $0 [--init|--check]" ;;
esac
```

**Example Execution:**
```
# First run: Create baseline
$ sudo ./integrity_checker.sh --init
Generating baseline checksums...
Baseline saved: 15423 files

# Later: Check for changes
$ sudo ./integrity_checker.sh --check
Checking file integrity...
MODIFIED: /etc/passwd
MODIFIED: /etc/shadow
NEW: /etc/backdoor.conf
DELETED: /usr/bin/suspicious
Integrity check complete. Changes detected: 4
```

**How MD5 Checksum Works:**
```
File: /etc/passwd
Content: "root:x:0:0:root:/root:/bin/bash..."

MD5 Hash: d41d8cd98f00b204e9800998ecf8427e
          │
          └── 128-bit fingerprint of file content

Any change (even 1 character) = completely different hash!

Before: d41d8cd98f00b204e9800998ecf8427e
After:  7d793037a0760186574b0282f2f435e7  ← DIFFERENT!
```

**Baseline File Format:**
```
# /var/lib/integrity/baseline.md5
d41d8cd98f00b204e9800998ecf8427e  /etc/passwd
a3c9e3b5c2d1a0b9e8f7d6c5b4a3e2d1  /etc/shadow
b2c4d6e8f0a2c4e6d8b0a2c4e6d8f0a2  /usr/bin/sudo
```

---

### 31. Open Port Scanner

**🎯 Problem Statement:**
Unauthorized services may open ports creating security vulnerabilities. We need to scan systems for open ports and compare against an approved list.

**🧠 Core Logic Explained:**
1. **Define allowed ports** → List of ports that should be open
2. **Scan port range** → Test each port for connectivity
3. **Use /dev/tcp** → Bash built-in for TCP connections
4. **Collect open ports** → Build list of responding ports
5. **Compare with allowed** → Flag unauthorized ports
6. **Identify services** → Look up service names for ports

**💡 Key Concepts:**
- `/dev/tcp/host/port` → Bash special file for TCP connections
- Port scanning → Test if port accepts connections
- `(command) 2>/dev/null` → Run in subshell, suppress errors
- `getent services` → Look up service by port number
- `grep -qw` → Quiet, whole word match

```bash
#!/bin/bash
# Open Port Scanner - Find unauthorized listening ports

ALLOWED_PORTS="22 80 443"                        # Ports that SHOULD be open
SCAN_TARGET="${1:-localhost}"                    # Default to localhost

echo "=== Port Scan: $SCAN_TARGET ==="
echo "Allowed ports: $ALLOWED_PORTS"
echo ""

# Scan common ports (1-1024 are "well-known" ports)
open_ports=""
for port in {1..1024}; do
    # /dev/tcp/host/port = Bash special file for TCP connections
    # (echo >/dev/tcp/...) = Try to connect in subshell
    # 2>/dev/null = Suppress connection errors
    # && = If connection succeeded, add to list
    (echo >/dev/tcp/"$SCAN_TARGET"/$port) 2>/dev/null && open_ports="$open_ports $port"
done

# Alternative: use nmap if available (faster and more accurate)
# nmap -p 1-1024 = Scan ports 1-1024
# --open = Only show open ports
# -oG - = Grepable output to stdout
# open_ports=$(nmap -p 1-1024 --open "$SCAN_TARGET" -oG - | grep -oP '\d+/open' | cut -d/ -f1 | tr '\n' ' ')

echo "Open ports: $open_ports"

# Check for unauthorized ports
echo -e "\n=== Compliance Check ==="
for port in $open_ports; do
    # grep -qw = Quiet mode, whole word match
    if echo "$ALLOWED_PORTS" | grep -qw "$port"; then
        echo "OK: Port $port (allowed)"
    else
        # getent services = Look up service name by port
        service=$(getent services "$port" 2>/dev/null | cut -d' ' -f1)
        echo "WARNING: Port $port ($service) - NOT in allowed list!"
    fi
done
```

**Example Output:**
```
$ ./port_scanner.sh webserver.example.com
=== Port Scan: webserver.example.com ===
Allowed ports: 22 80 443

Open ports:  22 25 80 443 3306

=== Compliance Check ===
OK: Port 22 (allowed)
WARNING: Port 25 (smtp) - NOT in allowed list!
OK: Port 80 (allowed)
OK: Port 443 (allowed)
WARNING: Port 3306 (mysql) - NOT in allowed list!
```

**/dev/tcp Explained:**
```bash
# /dev/tcp is a bash pseudo-device (not a real file)
# It creates a TCP connection when accessed

# Try to connect to google.com port 80:
(echo >/dev/tcp/google.com/80) 2>/dev/null && echo "Port open"

# How it works:
# 1. echo > = Try to write to...
# 2. /dev/tcp/host/port = TCP connection
# 3. If connection succeeds = command succeeds
# 4. If connection fails = command fails (exit code 1)

# NOTE: This only works in bash, not sh or dash!
```

**Common Port Numbers:**
```
Port    Service         Should be open?
─────────────────────────────────────────
22      SSH             ✓ Usually
25      SMTP            ✗ Unless mail server
80      HTTP            ✓ Web server
443     HTTPS           ✓ Web server
3306    MySQL           ✗ Should be internal only
5432    PostgreSQL      ✗ Should be internal only
6379    Redis           ✗ Should be internal only
```

---

### 32. User Audit Script

**🎯 Problem Statement:**
Security audits require knowing who has access to systems. We need to audit user accounts, sudo privileges, and authentication methods.

**🧠 Core Logic Explained:**
1. **Find users with shells** → Can interactively log in
2. **Find UID 0 users** → Root-equivalent accounts (security risk)
3. **Check sudo access** → Who can run privileged commands
4. **Find empty passwords** → Major security vulnerability
5. **Check last logins** → Detect unused accounts
6. **Count SSH keys** → Who has key-based access

**💡 Key Concepts:**
- `/etc/passwd` → User account information (name, UID, shell, home)
- `/etc/shadow` → Password hashes (requires root to read)
- `/etc/sudoers` → Sudo privilege configuration
- `lastlog` → Display last login times
- UID 0 → Root user ID (multiple UID 0 accounts = suspicious)

```bash
#!/bin/bash
# User Security Audit - Comprehensive user account audit

echo "=== User Security Audit Report ==="
echo "Generated: $(date)"
echo ""

# Users with login shells (can interactively log in)
# grep -E = Extended regex
# '/bin/(ba)?sh$' = Matches /bin/sh or /bin/bash at end
# cut -d: -f1,3,6 = Extract fields 1 (user), 3 (UID), 6 (home)
echo "=== Users with Login Shells ==="
grep -E '/bin/(ba)?sh$' /etc/passwd | cut -d: -f1,3,6

# Users with UID 0 (root equivalent - security concern!)
# awk -F: = Field separator is colon
# $3 == 0 = UID (field 3) equals 0
# Should only show "root"!
echo -e "\n=== Users with UID 0 (Root) ==="
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Sudo access (who can run privileged commands)
echo -e "\n=== Sudo Access ==="
# Filter out comments and empty lines
# grep -E 'ALL|NOPASSWD' = Show lines with full access or no password required
grep -v '^#' /etc/sudoers 2>/dev/null | grep -v '^$' | grep -E 'ALL|NOPASSWD'

# Also check sudoers.d directory (drop-in configs)
for file in /etc/sudoers.d/*; do
    [ -f "$file" ] && cat "$file" 2>/dev/null
done

# Users with empty passwords (CRITICAL security issue!)
# In /etc/shadow, field 2 is password hash
# Empty field 2 = no password required
echo -e "\n=== Users with Empty Passwords ==="
awk -F: '$2 == "" {print $1}' /etc/shadow 2>/dev/null

# Last login for all users (detect unused accounts)
# lastlog -t 30 = Only show logins within 30 days
echo -e "\n=== Last Login (Recent 30 days) ==="
lastlog -t 30 | grep -v "Never logged in"

# SSH keys (who has key-based access)
echo -e "\n=== SSH Authorized Keys ==="
for home in /home/*; do
    user=$(basename "$home")                     # Extract username from path
    if [ -f "$home/.ssh/authorized_keys" ]; then
        keys=$(wc -l < "$home/.ssh/authorized_keys")  # Count keys
        echo "$user: $keys authorized keys"
    fi
done
```

**Example Output:**
```
=== User Security Audit Report ===
Generated: Wed Jan 22 10:30:00 UTC 2026

=== Users with Login Shells ===
root:0:/root
admin:1000:/home/admin
developer:1001:/home/developer

=== Users with UID 0 (Root) ===
root
toor                           ← SUSPICIOUS! Another root account!

=== Sudo Access ===
admin ALL=(ALL:ALL) ALL
developer ALL=(ALL) NOPASSWD: /usr/bin/docker

=== Users with Empty Passwords ===
(empty - good!)

=== Last Login (Recent 30 days) ===
admin   pts/0    192.168.1.100    Wed Jan 22 09:15

=== SSH Authorized Keys ===
admin: 3 authorized keys
developer: 1 authorized keys
```

**/etc/passwd Format:**
```
root:x:0:0:root:/root:/bin/bash
  │  │ │ │  │     │      │
  │  │ │ │  │     │      └── Login shell
  │  │ │ │  │     └── Home directory  
  │  │ │ │  └── Comment/Full name
  │  │ │ └── Primary GID
  │  │ └── UID (0 = root)
  │  └── Password placeholder (actual in /etc/shadow)
  └── Username
```

---

### 33. Sudo Command Logger

**🎯 Problem Statement:**
Compliance requires audit trails of privileged commands. We need to extract and analyze all sudo commands run on the system.

**🧠 Core Logic Explained:**
1. **Parse auth logs** → Sudo logs to auth.log
2. **Filter sudo entries** → Find lines containing "sudo:"
3. **Extract command info** → User, command, timestamp
4. **Summarize by user** → Count commands per user
5. **Flag dangerous commands** → Highlight risky operations
6. **Generate report** → Save for audit purposes

**💡 Key Concepts:**
- Sudo logging → Every sudo command is logged with user and command
- `awk -F'[:,]'` → Multiple field separators
- `grep -E` → Extended regex for multiple patterns
- `tee` → Write to file and stdout simultaneously
- Audit trail → Compliance requirement for privileged access

```bash
#!/bin/bash
# Sudo Command Audit - Track privileged command usage

LOG="/var/log/auth.log"                          # Auth log location
OUTPUT="/var/log/sudo_audit_$(date +%F).log"     # Output report file

echo "=== Sudo Command Audit ===" | tee "$OUTPUT"
echo "Date range: Last 24 hours" | tee -a "$OUTPUT"
echo "" | tee -a "$OUTPUT"

# Extract sudo commands from auth log
# grep "sudo:" = Find sudo log entries
# grep "COMMAND=" = Only entries with actual commands
# awk parses the complex log format
grep "sudo:" "$LOG" | grep "COMMAND=" | \
    awk -F'[:,]' '{
        # Loop through all fields to find USER and COMMAND
        for(i=1;i<=NF;i++) {
            if($i ~ /USER=/) user=$i             # Find USER field
            if($i ~ /COMMAND=/) cmd=$i           # Find COMMAND field
        }
        print $1, user, cmd                      # Print timestamp, user, command
    }' | tee -a "$OUTPUT"

# Summary: Commands per user
echo -e "\n=== Commands per User ===" | tee -a "$OUTPUT"
# grep -oP 'USER=\K\w+' = Extract just username after "USER="
# \K = Reset match start (Perl regex)
grep "sudo:" "$LOG" | grep "COMMAND=" | \
    grep -oP 'USER=\K\w+' | sort | uniq -c | sort -rn | tee -a "$OUTPUT"

# Flag potentially dangerous commands
echo -e "\n=== Potentially Dangerous Commands ===" | tee -a "$OUTPUT"
# Pattern matches: rm -rf, chmod 777, dd, mkfs
grep -E "rm -rf|chmod 777|dd if=|mkfs\." "$LOG" | tee -a "$OUTPUT"
```

**Example Output:**
```
=== Sudo Command Audit ===
Date range: Last 24 hours

Jan 22 09:15:32 USER=root COMMAND=/usr/bin/apt update
Jan 22 10:23:45 USER=root COMMAND=/bin/systemctl restart nginx
Jan 22 11:45:12 USER=root COMMAND=/usr/bin/vim /etc/hosts
Jan 22 14:30:00 USER=admin COMMAND=/usr/bin/docker ps

=== Commands per User ===
     15 root
      8 admin
      3 developer

=== Potentially Dangerous Commands ===
Jan 22 12:00:00 ... COMMAND=/bin/rm -rf /tmp/oldfiles
```

**Sudo Log Format:**
```
Jan 22 09:15:32 server sudo: admin : TTY=pts/0 ; PWD=/home/admin ; USER=root ; COMMAND=/usr/bin/apt update
         │         │     │                │            │           │              │
         │         │     │                │            │           │              └── Actual command
         │         │     │                │            │           └── Target user (who to run as)
         │         │     │                │            └── Working directory
         │         │     │                └── Terminal
         │         │     └── Who ran sudo
         │         └── Service (sudo)
         └── Timestamp
```

---

### 34. Firewall Rules Backup

**🎯 Problem Statement:**
Firewall rule changes should be tracked for audit and rollback capability. We need to backup firewall configurations with version control.

**🧠 Core Logic Explained:**
1. **Create backup directory** → Organized storage location
2. **Initialize git** → Version control for rules
3. **Export iptables** → `iptables-save` dumps all rules
4. **Export firewalld** → If installed, backup zones
5. **Export ufw** → If installed, backup status
6. **Commit if changed** → Only commit when rules differ

**💡 Key Concepts:**
- `iptables-save` → Export all iptables rules to text
- `command -v` → Check if command exists
- `git diff --cached --quiet` → Check if staged changes exist
- Version control for configs → Track who changed what and when
- Multiple firewall systems → Different distros use different tools

```bash
#!/bin/bash
# Firewall Rules Backup - Version controlled firewall configs

BACKUP_DIR="/var/backups/firewall"               # Backup location
DATE=$(date +%Y%m%d_%H%M)                        # Timestamp

mkdir -p "$BACKUP_DIR"
cd "$BACKUP_DIR"

# Initialize git repository if not already done
# This allows tracking changes over time
[ -d .git ] || git init

# Backup iptables (most common Linux firewall)
# iptables-save = Export all rules in restore-compatible format
echo "Backing up iptables..."
iptables-save > "iptables_$DATE.rules"
cp "iptables_$DATE.rules" iptables_current.rules  # Current snapshot

# Backup firewalld (RHEL/CentOS/Fedora)
# command -v = Check if command exists (returns 0 if found)
# &>/dev/null = Suppress all output
if command -v firewall-cmd &>/dev/null; then
    echo "Backing up firewalld..."
    firewall-cmd --list-all-zones > "firewalld_$DATE.txt"
    cp "firewalld_$DATE.txt" firewalld_current.txt
fi

# Backup ufw (Ubuntu Uncomplicated Firewall)
if command -v ufw &>/dev/null; then
    echo "Backing up ufw..."
    ufw status verbose > "ufw_$DATE.txt"
    cp "ufw_$DATE.txt" ufw_current.txt
fi

# Commit changes to git (only if there are changes)
git add -A                                       # Stage all changes
# git diff --cached --quiet = Exit 0 if no staged changes
if git diff --cached --quiet; then
    echo "No firewall changes detected"
else
    git commit -m "Firewall backup $DATE"
    echo "Changes committed to git"
    git log --oneline -5                         # Show recent history
fi
```

**Example Execution:**
```
$ sudo ./firewall_backup.sh
Backing up iptables...
Backing up ufw...
[master abc1234] Firewall backup 20260122_1030
 2 files changed, 15 insertions(+), 2 deletions(-)
Changes committed to git
abc1234 Firewall backup 20260122_1030
def5678 Firewall backup 20260121_1030
ghi9012 Firewall backup 20260120_1030
```

**Why Git for Firewall Rules?**
```
Benefits:
1. See what changed: git diff
2. See who changed it: git log
3. Rollback easily: git checkout <commit>
4. Track history: git log --oneline

Example rollback:
$ git log --oneline
abc1234 Firewall backup 20260122 (broken)
def5678 Firewall backup 20260121 (working)

$ git checkout def5678 -- iptables_current.rules
$ iptables-restore < iptables_current.rules
# Firewall restored to working state!
```

**iptables-save Output Format:**
```
# Generated by iptables-save
*filter
:INPUT ACCEPT [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED -j ACCEPT
-A INPUT -p tcp --dport 22 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -j DROP
COMMIT
```

---

### 35. Password Expiry Checker

**🎯 Problem Statement:**
Expired passwords lock users out and create security risks. We need to identify users with expired or soon-to-expire passwords.

**🧠 Core Logic Explained:**
1. **Read shadow file** → Contains password aging information
2. **Skip system accounts** → UIDs below 1000 are usually system
3. **Skip locked accounts** → Passwords starting with ! or *
4. **Calculate expiry** → Last change + max days = expiry
5. **Convert to days** → Epoch seconds / 86400 = days
6. **Categorize status** → Expired, warning, or OK

**💡 Key Concepts:**
- `/etc/shadow` fields → user:pass:lastchg:min:max:warn:inactive:expire
- `lastchg` → Days since epoch when password was last changed
- `max` → Maximum days password is valid
- Day arithmetic → Seconds/86400 converts to days
- `[[ ]]` → Bash extended test for pattern matching

```bash
#!/bin/bash
# Password Expiry Checker - Find expired and expiring passwords

WARN_DAYS=14                                     # Warn if expiring within 14 days

echo "=== Password Expiry Report ==="
echo ""

# Read /etc/shadow line by line
# IFS=: = Use colon as field separator
# Fields: user:pass:lastchg:min:max:warn:inactive:expire
while IFS=: read -r user pass lastchg min max warn inactive expire rest; do
    # Skip system accounts (UID < 1000)
    uid=$(id -u "$user" 2>/dev/null)
    [ "$uid" -lt 1000 ] && continue
    
    # Skip locked/disabled accounts
    # Password starting with ! or * means locked
    [[ "$pass" == "!" || "$pass" == "*" || "$pass" == "!!" ]] && continue
    
    # Calculate expiry if max is set and not infinite (99999)
    if [ "$max" != "" ] && [ "$max" != "99999" ] && [ "$lastchg" != "" ]; then
        # Convert current time to days since epoch
        today=$(( $(date +%s) / 86400 ))
        
        # Calculate expiry day and days remaining
        expiry_day=$((lastchg + max))
        days_left=$((expiry_day - today))
        
        # Convert expiry_day back to date format
        expiry_date=$(date -d "1970-01-01 +${expiry_day} days" +%Y-%m-%d)
        
        # Categorize status
        if [ $days_left -lt 0 ]; then
            echo "EXPIRED: $user (expired $((-days_left)) days ago)"
        elif [ $days_left -lt $WARN_DAYS ]; then
            echo "WARNING: $user expires in $days_left days ($expiry_date)"
        else
            echo "OK: $user expires in $days_left days ($expiry_date)"
        fi
    fi
done < /etc/shadow

# Find accounts with no expiry set (potential risk)
echo -e "\n=== Accounts with No Expiry ==="
# $5 = max field (empty or 99999 = no expiry)
awk -F: '$5 == "" || $5 == 99999 {print $1}' /etc/shadow | while read user; do
    uid=$(id -u "$user" 2>/dev/null)
    [ "$uid" -ge 1000 ] && echo "$user"
done
```

**Example Output:**
```
=== Password Expiry Report ===

EXPIRED: olduser (expired 15 days ago)
WARNING: developer expires in 7 days (2026-01-29)
OK: admin expires in 45 days (2026-03-08)

=== Accounts with No Expiry ===
service_account
backup_user
```

**/etc/shadow Format:**
```
admin:$6$xyz...abc:19380:0:90:7:::
  │       │        │   │  │  │
  │       │        │   │  │  └── Warning days before expiry
  │       │        │   │  └── Max days password valid (90)
  │       │        │   └── Min days between changes (0)
  │       │        └── Last change: days since 1970-01-01 (19380)
  │       └── Encrypted password hash
  └── Username
```

**Days Calculation:**
```
lastchg = 19380 (days since 1970-01-01)
max = 90 (password valid for 90 days)
today = 19410 (current day)

expiry_day = 19380 + 90 = 19470
days_left = 19470 - 19410 = 60 days remaining
```

---

### 36. AWS EC2 Instance Manager

**🎯 Problem Statement:**
Managing EC2 instances manually is error-prone. We need a script to list, start, and stop instances based on environment tags.

**🧠 Core Logic Explained:**
1. **Accept action and environment** → list/start/stop + dev/staging/prod
2. **Use AWS CLI** → Query EC2 API
3. **Filter by tags** → Find instances matching environment
4. **JMESPath queries** → Extract specific fields from JSON
5. **Confirm before action** → Prevent accidental operations
6. **Wait for completion** → Ensure operation finishes

**💡 Key Concepts:**
- AWS CLI → Command-line interface for AWS services
- `--filters` → Filter resources by tags or properties
- `--query` → JMESPath expression to extract data
- `aws ec2 wait` → Block until instance reaches desired state
- Environment variables → `AWS_REGION`, `AWS_PROFILE`

```bash
#!/bin/bash
# AWS EC2 Instance Manager - List, start, stop by environment

ACTION="$1"                                      # list, start, or stop
ENVIRONMENT="${2:-development}"                  # Default to development
REGION="${AWS_REGION:-us-east-1}"                # Use env var or default

# Display usage help
usage() {
    echo "Usage: $0 <list|start|stop> <environment>"
    echo "Environments: development, staging, production"
    exit 1
}

# Function: List instances matching environment
list_instances() {
    echo "=== EC2 Instances (Environment: $ENVIRONMENT) ==="
    
    # aws ec2 describe-instances = Query EC2 API
    # --filters = Filter by tag (Environment=development)
    # --query = JMESPath to extract specific fields from JSON response
    # Query explained:
    #   Reservations[].Instances[] = Flatten nested structure
    #   [InstanceId, State.Name, InstanceType, ...] = Select fields
    #   Tags[?Key==`Name`].Value|[0] = Get Name tag value
    aws ec2 describe-instances \
        --region "$REGION" \
        --filters "Name=tag:Environment,Values=$ENVIRONMENT" \
        --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,Tags[?Key==`Name`].Value|[0]]' \
        --output table
}

# Function: Start or stop instances
manage_instances() {
    local action="$1"                            # start or stop
    
    # Get instance IDs for this environment
    instances=$(aws ec2 describe-instances \
        --region "$REGION" \
        --filters "Name=tag:Environment,Values=$ENVIRONMENT" \
        --query 'Reservations[].Instances[].InstanceId' \
        --output text)
    
    if [ -z "$instances" ]; then
        echo "No instances found for environment: $ENVIRONMENT"
        exit 1
    fi
    
    echo "Instances: $instances"
    read -p "Confirm $action? (y/n): " confirm
    [ "$confirm" != "y" ] && exit 0              # Abort if not confirmed
    
    # Execute start-instances or stop-instances
    aws ec2 "${action}-instances" --region "$REGION" --instance-ids $instances
    echo "$action initiated. Waiting..."
    
    # Wait for instances to reach desired state
    # aws ec2 wait = Block until condition met
    aws ec2 wait "instance-${action}ed" --region "$REGION" --instance-ids $instances
    echo "Complete!"
}

# Main: Handle command line actions
case "$ACTION" in
    list) list_instances ;;
    start) manage_instances "start" ;;
    stop) manage_instances "stop" ;;
    *) usage ;;
esac
```

**Example Execution:**
```
$ ./ec2_manager.sh list development
=== EC2 Instances (Environment: development) ===
-----------------------------------------------------------------------
| i-0abc123def456 | running | t3.medium | dev-web-server-1    |
| i-0def789ghi012 | running | t3.small  | dev-api-server-1    |
| i-0ghi345jkl678 | stopped | t3.large  | dev-database-server |
-----------------------------------------------------------------------

$ ./ec2_manager.sh stop development
Instances: i-0abc123def456 i-0def789ghi012 i-0ghi345jkl678
Confirm stop? (y/n): y
{
    "StoppingInstances": [...]
}
stop initiated. Waiting...
Complete!
```

**JMESPath Query Explained:**
```
JSON response:
{
  "Reservations": [{
    "Instances": [{
      "InstanceId": "i-0abc123",
      "State": {"Name": "running"},
      "InstanceType": "t3.medium",
      "Tags": [
        {"Key": "Name", "Value": "web-server"},
        {"Key": "Environment", "Value": "development"}
      ]
    }]
  }]
}

Query: Reservations[].Instances[].[InstanceId,State.Name,InstanceType,Tags[?Key==`Name`].Value|[0]]

Step by step:
1. Reservations[]           → Flatten Reservations array
2. .Instances[]             → Flatten Instances array
3. [InstanceId, ...]        → Select specific fields
4. Tags[?Key==`Name`]       → Filter tags where Key=Name
5. .Value|[0]               → Get Value, take first match

Result: ["i-0abc123", "running", "t3.medium", "web-server"]
```

---

### 37. S3 Bucket Sync with Logging

**🎯 Problem Statement:**
File uploads to S3 need tracking and verification. We need to sync directories to S3 with detailed logging for audit and troubleshooting.

**🧠 Core Logic Explained:**
1. **Accept source and destination** → Local dir and S3 bucket
2. **Log start time** → Audit trail
3. **Show pre-sync stats** → File count and size
4. **Perform sync** → Use aws s3 sync with exclusions
5. **Verify success** → Compare file counts
6. **Log completion** → Record outcome

**💡 Key Concepts:**
- `aws s3 sync` → Efficient sync (only transfers changed files)
- `--delete` → Remove files from S3 not in source
- `--exclude` → Skip files matching pattern
- `2>&1` → Redirect stderr to stdout for complete logging
- `tee -a` → Append to log AND show on screen

```bash
#!/bin/bash
# S3 Bucket Sync - Sync local directory to S3 with logging

LOCAL_DIR="$1"                                   # Local source directory
S3_BUCKET="$2"                                   # S3 destination (s3://bucket/path)
LOG_FILE="/var/log/s3sync_$(date +%F).log"       # Daily log file

# Validate arguments
if [ -z "$LOCAL_DIR" ] || [ -z "$S3_BUCKET" ]; then
    echo "Usage: $0 <local_dir> <s3://bucket/path>"
    exit 1
fi

echo "=== S3 Sync Started: $(date) ===" | tee -a "$LOG_FILE"
echo "Source: $LOCAL_DIR" | tee -a "$LOG_FILE"
echo "Destination: $S3_BUCKET" | tee -a "$LOG_FILE"

# Pre-sync stats (for comparison)
local_files=$(find "$LOCAL_DIR" -type f | wc -l)
local_size=$(du -sh "$LOCAL_DIR" | cut -f1)
echo "Local: $local_files files, $local_size" | tee -a "$LOG_FILE"

# Perform sync with progress
# aws s3 sync = Only transfer changed files (efficient!)
# --delete = Remove files from S3 that don't exist locally
# --exclude = Skip files matching pattern
# 2>&1 = Redirect stderr to stdout (capture all output)
# tee -a = Append to log AND show on screen
aws s3 sync "$LOCAL_DIR" "$S3_BUCKET" \
    --delete \
    --exclude "*.tmp" \
    --exclude ".git/*" \
    2>&1 | tee -a "$LOG_FILE"

sync_status=$?                                   # Capture exit code

# Post-sync verification
if [ $sync_status -eq 0 ]; then
    # Count files in S3 bucket
    s3_files=$(aws s3 ls "$S3_BUCKET" --recursive | wc -l)
    echo "S3: $s3_files files synced" | tee -a "$LOG_FILE"
    echo "Sync SUCCESSFUL" | tee -a "$LOG_FILE"
else
    echo "Sync FAILED with exit code $sync_status" | tee -a "$LOG_FILE"
fi

echo "=== S3 Sync Completed: $(date) ===" | tee -a "$LOG_FILE"
```

**Example Execution:**
```
$ ./s3_sync.sh /var/www/html s3://mycompany-website/html
=== S3 Sync Started: Wed Jan 22 10:30:00 UTC 2026 ===
Source: /var/www/html
Destination: s3://mycompany-website/html
Local: 1523 files, 45M
upload: /var/www/html/new-page.html to s3://mycompany-website/html/new-page.html
upload: /var/www/html/css/style.css to s3://mycompany-website/html/css/style.css
delete: s3://mycompany-website/html/old-page.html
S3: 1523 files synced
Sync SUCCESSFUL
=== S3 Sync Completed: Wed Jan 22 10:30:45 UTC 2026 ===
```

**Why aws s3 sync vs aws s3 cp?**
```
aws s3 cp:
- Copies ALL files every time
- 1000 files = 1000 uploads
- Slow and expensive

aws s3 sync:
- Only copies CHANGED files
- Compares file size and timestamp
- 1000 files, 5 changed = 5 uploads
- Fast and efficient!
```

**--delete Flag Explained:**
```
Without --delete:
Local:  [file1] [file2] [file3]
S3:     [file1] [file2] [file3] [old_file]  ← old_file stays!

With --delete:
Local:  [file1] [file2] [file3]
S3:     [file1] [file2] [file3]             ← old_file removed

Use --delete to keep S3 as exact mirror of local
```

---

### 38. AWS Cost Alert Script

**🎯 Problem Statement:**
Cloud costs can spiral out of control. We need to monitor AWS spending and alert when costs exceed budget thresholds.

**🧠 Core Logic Explained:**
1. **Define budget threshold** → Alert level in dollars
2. **Get date range** → Current month start to today
3. **Query Cost Explorer** → AWS API for billing data
4. **Parse JSON with jq** → Extract cost values
5. **Group by service** → Show top spenders
6. **Alert on threshold** → Send email if over budget

**💡 Key Concepts:**
- `aws ce get-cost-and-usage` → Cost Explorer API
- `jq` → Command-line JSON processor
- `bc -l` → Calculator for floating-point comparison
- `UnblendedCost` → Actual costs without savings plans
- `--granularity MONTHLY` → Aggregate by month

```bash
#!/bin/bash
# AWS Cost Alert - Monitor spending and alert on threshold

BUDGET_THRESHOLD=1000                            # Alert threshold in dollars
ALERT_EMAIL="finance@company.com"                # Email for alerts

# Calculate date range (current month)
START_DATE=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d)  # First of month
END_DATE=$(date +%Y-%m-%d)                       # Today

echo "=== AWS Cost Report ==="
echo "Period: $START_DATE to $END_DATE"
echo ""

# Query Cost Explorer API
# --time-period = Date range
# --granularity MONTHLY = Aggregate by month
# --metrics UnblendedCost = Actual costs (not savings plans adjusted)
# --group-by SERVICE = Break down by AWS service
cost_data=$(aws ce get-cost-and-usage \
    --time-period Start="$START_DATE",End="$END_DATE" \
    --granularity MONTHLY \
    --metrics UnblendedCost \
    --group-by Type=DIMENSION,Key=SERVICE \
    --output json)

# Parse total cost using jq
# jq = JSON processor
# '.ResultsByTime[0].Total.UnblendedCost.Amount' = Navigate JSON path
# '// 0' = Default to 0 if null
total_cost=$(echo "$cost_data" | jq -r '.ResultsByTime[0].Total.UnblendedCost.Amount // 0')
echo "Total Cost: \$$(printf '%.2f' $total_cost)"
echo ""

# Display cost breakdown by service
echo "=== Cost by Service ==="
# Extract each service and its cost, sort by cost descending
echo "$cost_data" | jq -r '.ResultsByTime[0].Groups[] | "\(.Keys[0]): $\(.Metrics.UnblendedCost.Amount)"' | \
    sort -t'$' -k2 -rn | head -10

# Check budget threshold
# bc -l = Calculator with floating point support
# Returns 1 if condition true, 0 if false
if (( $(echo "$total_cost > $BUDGET_THRESHOLD" | bc -l) )); then
    message="AWS Cost Alert: Current spend \$$total_cost exceeds budget \$$BUDGET_THRESHOLD"
    echo "$message"
    echo "$message" | mail -s "AWS Budget Alert" "$ALERT_EMAIL"
fi
```

**Example Output:**
```
=== AWS Cost Report ===
Period: 2026-01-01 to 2026-01-22

Total Cost: $1,234.56

=== Cost by Service ===
Amazon EC2: $567.89
Amazon RDS: $234.56
Amazon S3: $123.45
AWS Lambda: $78.90
Amazon CloudWatch: $45.67

AWS Cost Alert: Current spend $1234.56 exceeds budget $1000
```

**jq JSON Parsing:**
```json
// API Response structure:
{
  "ResultsByTime": [{
    "TimePeriod": {"Start": "2026-01-01", "End": "2026-01-22"},
    "Groups": [
      {"Keys": ["Amazon EC2"], "Metrics": {"UnblendedCost": {"Amount": "567.89"}}},
      {"Keys": ["Amazon RDS"], "Metrics": {"UnblendedCost": {"Amount": "234.56"}}}
    ]
  }]
}

// jq queries:
.ResultsByTime[0]                              → First time period
.Groups[]                                       → Iterate all services
.Keys[0]                                        → Service name
.Metrics.UnblendedCost.Amount                  → Cost value
```

---

### 39. EBS Snapshot Automation

**🎯 Problem Statement:**
EBS volumes need regular backups for disaster recovery. We need to automatically create snapshots of tagged volumes and manage retention.

**🧠 Core Logic Explained:**
1. **Find tagged volumes** → Look for volumes with Backup=true tag
2. **Get volume name** → Extract Name tag for identification
3. **Create snapshot** → Take point-in-time backup
4. **Tag the snapshot** → Mark as automated for cleanup
5. **Calculate cutoff date** → Retention period calculation
6. **Delete old snapshots** → Remove snapshots beyond retention

**💡 Key Concepts:**
- EBS snapshots → Point-in-time backups to S3
- Tag-based filtering → Use tags to identify resources
- `--tag-specifications` → Apply tags during creation
- `date -d "-N days"` → Calculate date N days ago
- JMESPath queries → Filter by date in AWS CLI

```bash
#!/bin/bash
# EBS Snapshot Automation - Create and manage volume backups

RETENTION_DAYS=7                                 # Keep snapshots for 7 days
REGION="${AWS_REGION:-us-east-1}"                # AWS region

echo "=== EBS Snapshot Automation ==="

# Find volumes tagged for backup (Backup=true)
volumes=$(aws ec2 describe-volumes \
    --region "$REGION" \
    --filters "Name=tag:Backup,Values=true" \
    --query 'Volumes[].VolumeId' \
    --output text)

if [ -z "$volumes" ]; then
    echo "No volumes tagged for backup"
    exit 0
fi

# Create snapshots for each volume
for vol_id in $volumes; do
    # Get volume's Name tag for snapshot description
    name=$(aws ec2 describe-volumes \
        --region "$REGION" \
        --volume-ids "$vol_id" \
        --query 'Volumes[0].Tags[?Key==`Name`].Value' \
        --output text)
    
    echo "Creating snapshot for $vol_id ($name)..."
    
    # Create snapshot with tags
    # --tag-specifications = Apply tags during creation
    snapshot_id=$(aws ec2 create-snapshot \
        --region "$REGION" \
        --volume-id "$vol_id" \
        --description "Automated backup $(date +%Y-%m-%d)" \
        --tag-specifications "ResourceType=snapshot,Tags=[{Key=Name,Value=$name-backup},{Key=AutoBackup,Value=true}]" \
        --query 'SnapshotId' \
        --output text)
    
    echo "Created: $snapshot_id"
done

# Cleanup old snapshots
echo -e "\n=== Cleaning up snapshots older than $RETENTION_DAYS days ==="
# Calculate cutoff date
cutoff_date=$(date -d "-$RETENTION_DAYS days" +%Y-%m-%d)

# Find old automated snapshots
old_snapshots=$(aws ec2 describe-snapshots \
    --region "$REGION" \
    --filters "Name=tag:AutoBackup,Values=true" \
    --query "Snapshots[?StartTime<='$cutoff_date'].SnapshotId" \
    --output text)

for snap_id in $old_snapshots; do
    echo "Deleting old snapshot: $snap_id"
    aws ec2 delete-snapshot --region "$REGION" --snapshot-id "$snap_id"
done

echo "Snapshot automation complete!"
```

**Example Execution:**
```
$ ./ebs_snapshot.sh
=== EBS Snapshot Automation ===
Creating snapshot for vol-0abc123def (production-database)...
Created: snap-0abc123def456789
Creating snapshot for vol-0def456ghi (production-webserver)...
Created: snap-0def456ghi789012

=== Cleaning up snapshots older than 7 days ===
Deleting old snapshot: snap-0old111aaa222bbb
Deleting old snapshot: snap-0old333ccc444ddd
Snapshot automation complete!
```

**Tag-Based Backup Strategy:**
```
Tag volumes in AWS Console or CLI:
aws ec2 create-tags --resources vol-xyz --tags Key=Backup,Value=true

Volume Tags:
┌─────────────────────┬─────────────┬────────────────┐
│ VolumeId            │ Name        │ Backup Tag     │
├─────────────────────┼─────────────┼────────────────┤
│ vol-0abc123         │ prod-db     │ Backup=true    │ ← Backed up
│ vol-0def456         │ prod-web    │ Backup=true    │ ← Backed up
│ vol-0ghi789         │ temp-data   │ (none)         │ ← Skipped
└─────────────────────┴─────────────┴────────────────┘
```

**Snapshot Tagging for Cleanup:**
```
Tags applied during creation:
  Name = production-database-backup
  AutoBackup = true

Filter for cleanup:
  --filters "Name=tag:AutoBackup,Values=true"
  
This ONLY deletes automated backups, never manual ones!
```

---

### 40. CloudWatch Metrics Pusher

**🎯 Problem Statement:**
AWS CloudWatch doesn't collect memory, disk, or custom application metrics by default. We need to push custom metrics for monitoring.

**🧠 Core Logic Explained:**
1. **Get instance identity** → Metadata service provides instance ID
2. **Create reusable function** → `push_metric` for any metric
3. **Collect memory stats** → Parse /proc/meminfo
4. **Collect disk stats** → Parse df output
5. **Collect system stats** → File handles, processes, connections
6. **Push to CloudWatch** → AWS CLI put-metric-data

**💡 Key Concepts:**
- EC2 metadata service → `169.254.169.254` provides instance info
- `/proc/meminfo` → Linux memory information
- `bc` → Calculator for floating-point math
- CloudWatch custom metrics → User-defined measurements
- Namespace → Group related metrics together

```bash
#!/bin/bash
# CloudWatch Custom Metrics - Push memory, disk, and system metrics

NAMESPACE="Custom/System"                        # CloudWatch namespace
# Get instance ID from EC2 metadata service
# 169.254.169.254 = Link-local address for metadata
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)

# Function: Push a metric to CloudWatch
push_metric() {
    local metric_name="$1"                       # Metric name
    local value="$2"                             # Metric value
    local unit="${3:-Count}"                     # Unit (default: Count)
    
    # aws cloudwatch put-metric-data = Send custom metric
    # --dimensions = Add instance ID for filtering
    aws cloudwatch put-metric-data \
        --namespace "$NAMESPACE" \
        --metric-name "$metric_name" \
        --value "$value" \
        --unit "$unit" \
        --dimensions InstanceId="$INSTANCE_ID"
}

# Metric 1: Memory utilization
# Parse /proc/meminfo for memory values (in KB)
mem_total=$(grep MemTotal /proc/meminfo | awk '{print $2}')
mem_available=$(grep MemAvailable /proc/meminfo | awk '{print $2}')
# Calculate percentage used
# bc = Calculator, scale=2 = 2 decimal places
mem_used_pct=$(echo "scale=2; (($mem_total - $mem_available) / $mem_total) * 100" | bc)
push_metric "MemoryUtilization" "$mem_used_pct" "Percent"

# Metric 2: Disk utilization (root filesystem)
# df / = Disk usage for root
# tail -1 = Last line (skip header)
# $5 = Percentage column
# tr -d '%' = Remove percent sign
disk_used_pct=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
push_metric "DiskUtilization" "$disk_used_pct" "Percent"

# Metric 3: Open file handles
# /proc/sys/fs/file-nr = allocated / free / max
# $1 = Currently allocated
open_files=$(cat /proc/sys/fs/file-nr | awk '{print $1}')
push_metric "OpenFileHandles" "$open_files" "Count"

# Metric 4: Process count
process_count=$(ps aux | wc -l)
push_metric "ProcessCount" "$process_count" "Count"

# Metric 5: TCP connections
# ss -s = Socket statistics summary
tcp_connections=$(ss -s | grep 'estab' | awk '{print $4}' | tr -d ',')
push_metric "TCPConnections" "${tcp_connections:-0}" "Count"

echo "Metrics pushed to CloudWatch ($NAMESPACE)"
```

**Example Output:**
```
$ ./cloudwatch_push.sh
Metrics pushed to CloudWatch (Custom/System)

# View in CloudWatch Console:
Custom/System namespace:
  MemoryUtilization: 67.45%
  DiskUtilization: 42%
  OpenFileHandles: 1234
  ProcessCount: 156
  TCPConnections: 89
```

**EC2 Metadata Service:**
```
URL: http://169.254.169.254/latest/meta-data/

Available endpoints:
/instance-id         → i-0abc123def456789
/instance-type       → t3.medium
/local-ipv4          → 10.0.1.100
/public-ipv4         → 54.123.45.67
/ami-id              → ami-0abc123def
/hostname            → ip-10-0-1-100.ec2.internal
```

**/proc/meminfo Explained:**
```
$ cat /proc/meminfo
MemTotal:       16384000 kB    ← Total RAM
MemFree:         2048000 kB    ← Completely unused
MemAvailable:    8192000 kB    ← Available for apps (includes cached)
Buffers:          512000 kB
Cached:          4096000 kB    ← File cache (can be freed)

Memory calculation:
used = total - available = 16384000 - 8192000 = 8192000 kB
percent = (used / total) * 100 = 50%
```

**Run Every Minute with Cron:**
```bash
# Add to crontab for continuous monitoring:
* * * * * /opt/scripts/cloudwatch_push.sh >> /var/log/cloudwatch_push.log 2>&1
```

---

### 41. Multi-Host Ping Sweep

**🎯 Problem Statement:**
Need to quickly discover which hosts are alive in a network subnet for inventory or troubleshooting.

**🧠 Core Logic Explained:**
1. **Accept subnet prefix** → e.g., 192.168.1
2. **Create scan function** → Test single IP
3. **Export function** → Make available to subshells
4. **Generate IP sequence** → 1-254 for /24 subnet
5. **Parallel execution** → xargs runs multiple pings simultaneously
6. **Resolve hostnames** → Look up DNS names for found hosts

**💡 Key Concepts:**
- `ping -c 1 -W 1` → Single packet, 1 second timeout
- `export -f` → Export function for use in subshells
- `xargs -P N` → Run N processes in parallel
- `seq 1 254` → Generate sequence for /24 subnet (254 hosts)
- `getent hosts` → DNS lookup (forward/reverse)

```bash
#!/bin/bash
# Multi-Host Ping Sweep - Discover live hosts on a network subnet

SUBNET="${1:-192.168.1}"   # First 3 octets of subnet (default: 192.168.1)
TIMEOUT=1                   # Ping timeout in seconds (fail fast)
PARALLEL=50                 # Number of parallel ping processes

echo "=== Scanning subnet $SUBNET.0/24 ==="  # Show which subnet we're scanning

# Function to scan a single host - will be exported to subshells
scan_host() {
    local ip="$1"           # IP address passed as argument
    
    # Send 1 ping with 1 second timeout, suppress all output
    if ping -c 1 -W $TIMEOUT "$ip" &>/dev/null; then
        # If ping succeeds, try to resolve hostname via DNS
        # getent queries system name service (includes /etc/hosts + DNS)
        hostname=$(getent hosts "$ip" 2>/dev/null | awk '{print $2}')
        # ${hostname:+($hostname)} = if hostname is set, show (hostname)
        echo "UP: $ip ${hostname:+($hostname)}"
    fi
    # If ping fails, output nothing (host is down)
}

# Export function so it's available in xargs subshells
export -f scan_host
export TIMEOUT          # Also export TIMEOUT variable for subshells

# Generate IP sequence 1-254 and scan in parallel:
# seq 1 254      → outputs numbers 1 through 254 (one per line)
# xargs -P 50    → run up to 50 processes in parallel
# xargs -I{}     → replace {} with each number from seq
# bash -c "..."  → run scan_host in a new shell (needed for exported function)
seq 1 254 | xargs -P $PARALLEL -I{} bash -c "scan_host $SUBNET.{}"

echo -e "\nScan complete!"
```

**Example Execution:**
```bash
$ ./ping_sweep.sh 192.168.1
=== Scanning subnet 192.168.1.0/24 ===
UP: 192.168.1.1 (router.local)
UP: 192.168.1.10 (server1.local)
UP: 192.168.1.11 (server2.local)
UP: 192.168.1.25
UP: 192.168.1.50 (workstation.local)
UP: 192.168.1.100 (printer.local)

Scan complete!
```

**How Parallel Scanning Works:**
```
Input: seq 1 254 generates:
  1
  2
  3
  ...
  254

xargs -P 50 distributes to 50 workers:
  
  Worker 1    Worker 2    Worker 3    ...   Worker 50
  ────────    ────────    ────────          ─────────
  ping .1     ping .2     ping .3           ping .50
  ping .51    ping .52    ping .53          ping .100
  ping .101   ...         ...               ...
  
  ┌──────────────────────────────────────────────────┐
  │ Without -P 50: 254 pings × 1 sec = ~254 seconds  │
  │ With -P 50:    254 pings / 50 = ~6 seconds       │
  └──────────────────────────────────────────────────┘
```

**Export Function Explained:**
```
Normal bash:
  function exists only in current shell
  
  Parent Shell ──→ xargs spawns subshell ──→ Function NOT found!
  
With export -f:
  function is copied to all child processes
  
  Parent Shell ──→ export -f scan_host ──→ Function AVAILABLE
       │                                        │
       └────────→ xargs spawns subshell ────────┘
```

---

### 42. DNS Resolution Checker

**🎯 Problem Statement:**
DNS issues cause mysterious outages. We need to verify DNS resolution works correctly across multiple DNS servers and detect inconsistencies.

**🧠 Core Logic Explained:**
1. **Define critical domains** → Services that must resolve
2. **Query multiple DNS servers** → Compare results for consistency
3. **Detect failures** → Empty response means DNS failure
4. **Detect mismatches** → Different IPs from different servers
5. **Measure propagation time** → How long queries take

**💡 Key Concepts:**
- `dig +short @server domain` → Query specific DNS server
- DNS propagation → Changes take time to spread
- `date +%s%N` → Nanosecond timestamp for timing
- Mismatch detection → Different DNS servers returning different IPs
- `/etc/resolv.conf` → System's configured DNS servers

```bash
#!/bin/bash
# DNS Resolution Checker - Verify DNS resolution across multiple servers

# Critical domains that must resolve correctly for services to work
DOMAINS="api.company.com www.company.com db.company.com"

# DNS servers to query: Google, Cloudflare, and system's primary DNS
# grep nameserver /etc/resolv.conf → extracts configured DNS servers
DNS_SERVERS="8.8.8.8 1.1.1.1 $(grep nameserver /etc/resolv.conf | head -1 | awk '{print $2}')"

echo "=== DNS Resolution Check ==="

# Loop through each domain we need to verify
for domain in $DOMAINS; do
    echo -e "\n--- $domain ---"    # Section header for each domain
    prev_ip=""                      # Track previous result for mismatch detection
    
    # Query each DNS server for this domain
    for dns in $DNS_SERVERS; do
        # dig +short    → Return only the IP (not full DNS response)
        # @"$dns"       → Query specific DNS server
        # head -1       → Take first IP if multiple A records
        result=$(dig +short @"$dns" "$domain" 2>/dev/null | head -1)
        
        # Check if we got a response
        if [ -z "$result" ]; then
            # Empty result = DNS lookup failed
            echo "  $dns: FAILED (no response)"
        else
            # Got a response - check if it matches previous server's response
            if [ -n "$prev_ip" ] && [ "$result" != "$prev_ip" ]; then
                # Different IP from different DNS server = potential DNS issue
                echo "  $dns: $result (WARNING: mismatch!)"
            else
                echo "  $dns: $result"       # Normal successful lookup
            fi
            prev_ip="$result"               # Store for next comparison
        fi
    done
done

# Measure how long DNS resolution takes (propagation time)
echo -e "\n=== DNS Propagation Test ==="
for domain in $DOMAINS; do
    start=$(date +%s%N)              # Start time in nanoseconds
    dig +short "$domain" &>/dev/null # Perform the lookup
    end=$(date +%s%N)                # End time in nanoseconds
    # Calculate milliseconds: (end - start) / 1,000,000
    time_ms=$(( (end - start) / 1000000 ))
    echo "$domain: ${time_ms}ms"
done
```

**Example Execution:**
```bash
$ ./dns_checker.sh

=== DNS Resolution Check ===

--- api.company.com ---
  8.8.8.8: 10.0.1.50
  1.1.1.1: 10.0.1.50
  10.0.0.2: 10.0.1.50

--- www.company.com ---
  8.8.8.8: 203.0.113.10
  1.1.1.1: 203.0.113.10
  10.0.0.2: 203.0.113.11 (WARNING: mismatch!)

--- db.company.com ---
  8.8.8.8: 10.0.1.100
  1.1.1.1: FAILED (no response)
  10.0.0.2: 10.0.1.100

=== DNS Propagation Test ===
api.company.com: 12ms
www.company.com: 8ms
db.company.com: 15ms
```

**Why DNS Mismatches Happen:**
```
DNS Change Propagation:

Time 0:   Admin updates www.company.com to new IP 203.0.113.11
          ↓
Time 1min: Internal DNS (10.0.0.2) has new IP
          ↓
Time 5min: Google DNS (8.8.8.8) still has old IP (cached)
          ↓
Time 30min: All DNS servers have new IP

┌─────────────────────────────────────────────────────┐
│ TTL (Time To Live) controls how long DNS is cached  │
│ Lower TTL = faster propagation, more DNS queries    │
│ Higher TTL = slower propagation, fewer queries      │
└─────────────────────────────────────────────────────┘
```

**dig Command Options:**
```
dig +short @8.8.8.8 example.com
     │       │       │
     │       │       └── Domain to lookup
     │       └────────── Query this specific DNS server
     └────────────────── Return only answer (IP address)

Full dig output:
;; ANSWER SECTION:
example.com.    300   IN    A    93.184.216.34
                │           │    │
                TTL(sec)    Type IP Address

dig +short returns just: 93.184.216.34
```

---

### 43. Load Balancer Health Probe

**🎯 Problem Statement:**
When websites have issues, we need to check individual backend servers to identify which ones are failing behind the load balancer.

**🧠 Core Logic Explained:**
1. **List all backends** → IPs of servers behind LB
2. **Check each directly** → Bypass load balancer
3. **Call health endpoint** → Standard /health check
4. **Measure response time** → Identify slow backends
5. **Count healthy/unhealthy** → Calculate availability
6. **Alert on threshold** → Warn if too many unhealthy

**💡 Key Concepts:**
- Health check endpoint → /health returns 200 if healthy
- `curl -w "%{http_code} %{time_total}"` → Get code and timing
- Bypass load balancer → Direct backend IP access
- Availability calculation → healthy / total * 100
- Exit codes → Non-zero indicates failure for alerting

```bash
#!/bin/bash
# Load Balancer Backend Health Probe - Check individual backend servers

# List of backend server IPs (servers behind the load balancer)
BACKENDS="10.0.1.10 10.0.1.11 10.0.1.12 10.0.1.13"
HEALTH_PATH="/health"   # Standard health check endpoint
PORT=8080               # Application port
TIMEOUT=5               # Connection timeout in seconds

echo "=== Load Balancer Backend Health Check ==="

healthy=0               # Counter for healthy backends
unhealthy=0             # Counter for unhealthy backends

# Check each backend server individually
for backend in $BACKENDS; do
    # Build the full URL to the health endpoint
    url="http://$backend:$PORT$HEALTH_PATH"
    
    # curl options:
    # -s             → Silent mode (no progress bar)
    # -o /dev/null   → Discard response body (we only want status)
    # -w "format"    → Write out custom format after completion
    # %{http_code}   → HTTP response code (200, 404, 500, etc.)
    # %{time_total}  → Total time for the request in seconds
    # --connect-timeout → Max time to wait for connection
    response=$(curl -s -o /dev/null -w "%{http_code} %{time_total}" --connect-timeout $TIMEOUT "$url")
    
    # Split response into HTTP code and response time
    http_code=$(echo "$response" | cut -d' ' -f1)     # First field = HTTP code
    response_time=$(echo "$response" | cut -d' ' -f2) # Second field = time
    
    # HTTP 200 = healthy, anything else = unhealthy
    if [ "$http_code" == "200" ]; then
        echo "✓ $backend - HTTP $http_code (${response_time}s)"
        ((healthy++))           # Increment healthy counter
    else
        echo "✗ $backend - HTTP $http_code (UNHEALTHY)"
        ((unhealthy++))         # Increment unhealthy counter
    fi
done

echo ""
echo "Summary: $healthy healthy, $unhealthy unhealthy"

# Calculate availability percentage and alert if below threshold
total=$((healthy + unhealthy))
# Bash integer math: healthy * 100 / total gives percentage
if [ $((healthy * 100 / total)) -lt 50 ]; then
    echo "CRITICAL: Less than 50% backends healthy!"
    exit 1                      # Exit with error code for alerting systems
fi
```

**Example Execution:**
```bash
$ ./lb_health_check.sh

=== Load Balancer Backend Health Check ===
✓ 10.0.1.10 - HTTP 200 (0.023s)
✓ 10.0.1.11 - HTTP 200 (0.019s)
✗ 10.0.1.12 - HTTP 503 (UNHEALTHY)
✓ 10.0.1.13 - HTTP 200 (0.045s)

Summary: 3 healthy, 1 unhealthy
```

**Load Balancer Architecture:**
```
Client Request
     │
     ▼
┌────────────────┐
│ Load Balancer  │
│  (10.0.1.1)    │
└───────┬────────┘
        │
  ┌─────┴─────┬─────────┬─────────┐
  ▼           ▼         ▼         ▼
┌────┐     ┌────┐    ┌────┐    ┌────┐
│.10 │     │.11 │    │.12 │    │.13 │
│ ✓  │     │ ✓  │    │ ✗  │    │ ✓  │
└────┘     └────┘    └────┘    └────┘
 200        200       503       200

This script bypasses the LB to check backends directly!
LB health checks might pass even if backends are unhealthy
because LB uses cached health status.
```

**curl -w Format Specifiers:**
```
%{http_code}      → 200, 404, 500, 503, etc.
%{time_total}     → Total time in seconds (0.023)
%{time_connect}   → Time to establish connection
%{time_starttransfer} → Time to first byte
%{size_download}  → Total bytes downloaded
%{speed_download} → Average download speed

Example:
curl -s -o /dev/null -w "Code:%{http_code} Time:%{time_total}s" http://example.com
Output: Code:200 Time:0.156s
```

**HTTP Status Codes for Health:**
```
200 OK           → Healthy, serving requests
503 Service Unavailable → Overloaded/maintenance
502 Bad Gateway  → Backend crashed
504 Gateway Timeout → Backend too slow
000              → Connection refused/timeout
```

---

### 44. Bandwidth Usage Monitor

**🎯 Problem Statement:**
Network throughput issues require real-time bandwidth monitoring. We need to track network usage per interface to identify bottlenecks.

**🧠 Core Logic Explained:**
1. **Read /proc/net/dev** → Linux kernel exposes network stats
2. **Extract RX/TX bytes** → Received and transmitted counts
3. **Wait interval** → Allow traffic to accumulate
4. **Read again** → Get new values
5. **Calculate delta** → New - old = bytes transferred
6. **Convert to rate** → Bytes per second

**💡 Key Concepts:**
- `/proc/net/dev` → Real-time network statistics
- Delta calculation → (current - previous) / interval = rate
- `numfmt --to=iec` → Convert bytes to human-readable (KB, MB)
- `\r` → Carriage return for in-place updates
- Continuous loop → Real-time monitoring

```bash
#!/bin/bash
# Bandwidth Usage Monitor - Real-time network throughput monitoring

INTERFACE="${1:-eth0}"  # Network interface to monitor (default: eth0)
INTERVAL=1              # Measurement interval in seconds

# Function to get current byte counters from kernel
# /proc/net/dev contains real-time network statistics
get_bytes() {
    # grep "$INTERFACE" → find line for our interface
    # awk '{print $2, $10}' → extract RX bytes (col 2) and TX bytes (col 10)
    cat /proc/net/dev | grep "$INTERFACE" | awk '{print $2, $10}'
}

echo "=== Bandwidth Monitor: $INTERFACE ==="
echo "Press Ctrl+C to stop"
echo ""

# Get initial byte counts
# read var1 var2 <<< $(command) → read command output into two variables
read prev_rx prev_tx <<< $(get_bytes)

# Infinite loop for continuous monitoring
while true; do
    sleep $INTERVAL          # Wait for the measurement interval
    
    # Get new byte counts after interval
    read curr_rx curr_tx <<< $(get_bytes)
    
    # Calculate bytes transferred during interval
    # (current - previous) / interval = bytes per second
    rx_rate=$(( (curr_rx - prev_rx) / INTERVAL ))
    tx_rate=$(( (curr_tx - prev_tx) / INTERVAL ))
    
    # Convert bytes to human-readable format (KB, MB, GB)
    # numfmt --to=iec → converts to IEC units (1024-based: KiB, MiB)
    # 2>/dev/null → suppress error if numfmt not available
    # || echo "..." → fallback to raw bytes if numfmt fails
    rx_human=$(numfmt --to=iec $rx_rate 2>/dev/null || echo "$rx_rate B")
    tx_human=$(numfmt --to=iec $tx_rate 2>/dev/null || echo "$tx_rate B")
    
    # Print on same line using carriage return
    # \r     → move cursor to start of line (overwrite)
    # %s     → string placeholder for printf
    # "   "  → trailing spaces to clear previous longer output
    printf "\r[%s] RX: %s/s | TX: %s/s    " "$(date +%H:%M:%S)" "$rx_human" "$tx_human"
    
    # Current values become previous for next iteration
    prev_rx=$curr_rx
    prev_tx=$curr_tx
done
```

**Example Execution:**
```bash
$ ./bandwidth_monitor.sh eth0

=== Bandwidth Monitor: eth0 ===
Press Ctrl+C to stop

[14:32:15] RX: 1.2M/s | TX: 450K/s      ← Updates in place
[14:32:16] RX: 1.5M/s | TX: 523K/s
[14:32:17] RX: 890K/s | TX: 1.1M/s
^C
```

**/proc/net/dev Format:**
```
$ cat /proc/net/dev
Inter-|   Receive                            |  Transmit
 face |bytes    packets errs drop ...        |bytes    packets errs
 eth0: 1234567890 9876543  0    0 ...        567890123 4567890  0
       │          │                           │
       Column 2   Column 3                    Column 10
       RX bytes   RX packets                  TX bytes

Interface statistics are cumulative since boot!
To get rate: measure twice, calculate difference.
```

**Rate Calculation Explained:**
```
Time T0: RX bytes = 1,000,000
         sleep 1 second
Time T1: RX bytes = 1,500,000

Delta = 1,500,000 - 1,000,000 = 500,000 bytes
Rate  = 500,000 bytes / 1 second = 500 KB/s

┌────────────────────────────────────────────┐
│ cumulative     cumulative                  │
│ at T0          at T1                       │
│    │              │                        │
│    ▼              ▼                        │
│  1.0 MB ──────> 1.5 MB                     │
│           ↑                                │
│           └── 500 KB transferred in 1 sec  │
└────────────────────────────────────────────┘
```

**numfmt IEC Conversion:**
```
numfmt --to=iec 1024       → 1.0K
numfmt --to=iec 1048576    → 1.0M
numfmt --to=iec 1073741824 → 1.0G

IEC units (base 1024): KiB, MiB, GiB
SI units  (base 1000): KB, MB, GB

For networking, both are commonly used!
```

---

### 45. TCP Connection States Report

**🎯 Problem Statement:**
Network problems often manifest as connection issues. We need detailed TCP connection analysis to identify issues like connection leaks or high connection counts.

**🧠 Core Logic Explained:**
1. **Get all TCP connections** → `ss -tan` shows all states
2. **Count by state** → Group connections by their state
3. **Identify top ports** → Which services have most connections
4. **Identify top remote IPs** → Who's connecting most
5. **Check TIME_WAIT** → Too many indicates rapid open/close
6. **Check CLOSE_WAIT** → Indicates application not closing connections

**💡 Key Concepts:**
- `ss` → Modern replacement for netstat (faster)
- TCP states → ESTABLISHED, TIME_WAIT, CLOSE_WAIT, etc.
- CLOSE_WAIT → Application received FIN but hasn't closed socket
- TIME_WAIT → Connection closed, waiting for stray packets
- `ss state <state>` → Filter by specific TCP state

```bash
#!/bin/bash
# TCP Connection States Report - Analyze network connection health

echo "=== TCP Connection Analysis ==="
echo "Generated: $(date)"
echo ""

# Count connections grouped by their TCP state
echo "--- Connection States ---"
# ss -tan → show all TCP connections in numeric format
# -t = TCP only, -a = all states, -n = don't resolve names
# NR>1   → skip header line
# $1     → first column is the state (ESTAB, TIME-WAIT, etc.)
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn

# Find which local ports have most connections (busy services)
echo -e "\n--- Top Local Ports ---"
# $4 = local address:port (e.g., 10.0.1.5:443)
# split(a,":") → split by colon to get port
# a[length(a)] → last element is the port number
ss -tan | awk 'NR>1 {split($4,a,":"); print a[length(a)]}' | sort | uniq -c | sort -rn | head -10

# Find which remote IPs connect most often (top clients)
echo -e "\n--- Top Remote IPs ---"
# $5 = remote address:port (e.g., 192.168.1.100:54321)
# Complex awk: rebuild IP from all parts except last (the port)
ss -tan | awk 'NR>1 {split($5,a,":"); ip=""; for(i=1;i<length(a);i++) ip=ip""a[i]; print ip}' | \
    grep -v "^\*$" |       # Exclude wildcard entries (*:*)
    sort | uniq -c |       # Count occurrences
    sort -rn | head -10    # Top 10 by count

# Check TIME_WAIT connections (normal after closing)
# ss state <state> → filter by specific TCP state
timewait=$(ss -tan state time-wait | wc -l)
echo -e "\n--- TIME_WAIT Connections: $timewait ---"

# Check CLOSE_WAIT connections (potential application bug!)
# CLOSE_WAIT = received FIN from peer but application hasn't closed socket
closewait=$(ss -tan state close-wait | wc -l)
if [ "$closewait" -gt 10 ]; then
    # More than 10 CLOSE_WAIT is suspicious - might be connection leak
    echo "WARNING: $closewait CLOSE_WAIT connections (potential resource leak)"
    # Show details of CLOSE_WAIT connections for debugging
    ss -tan state close-wait | head -10
fi
```

**Example Execution:**
```bash
$ ./tcp_report.sh

=== TCP Connection Analysis ===
Generated: Mon Jan 15 14:35:22 UTC 2024

--- Connection States ---
    523 ESTAB
    156 TIME-WAIT
     12 LISTEN
      3 CLOSE-WAIT

--- Top Local Ports ---
    312 443
    156 80
     45 22
     10 3306

--- Top Remote IPs ---
     89 10.0.1.50
     67 192.168.1.100
     45 10.0.2.15

--- TIME_WAIT Connections: 156 ---
WARNING: 3 CLOSE_WAIT connections (potential resource leak)
State    Recv-Q Send-Q Local Address:Port   Peer Address:Port
CLOSE-WAIT 0     0      10.0.1.5:80         10.0.1.50:54321
```

**TCP State Machine:**
```
┌─────────────────────────────────────────────────────────┐
│                  TCP Connection Lifecycle               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Client                              Server             │
│    │                                   │                │
│    │ ────── SYN ─────────────────────► │ SYN_SENT      │
│    │ ◄───── SYN+ACK ────────────────── │ SYN_RECV      │
│    │ ────── ACK ─────────────────────► │                │
│    │         ESTABLISHED ◄────────────► ESTABLISHED    │
│    │              (data exchange)                       │
│    │                                                    │
│    │ ────── FIN ─────────────────────► │ FIN_WAIT_1    │
│    │ ◄───── ACK ─────────────────────  │ CLOSE_WAIT ⚠ │
│    │ ◄───── FIN ─────────────────────  │ LAST_ACK     │
│    │ ────── ACK ─────────────────────► │                │
│    │         TIME_WAIT                  CLOSED          │
│                                                         │
└─────────────────────────────────────────────────────────┘

⚠ CLOSE_WAIT: Server received FIN but app hasn't closed socket!
   This usually indicates a bug in the application.
```

**Common Connection Issues:**
```
High TIME_WAIT (normal):
  - Many short connections closed recently
  - Waiting 2*MSL (60-120 sec) for stray packets
  - Not usually a problem unless > 10,000

High CLOSE_WAIT (problem!):
  - Application not closing sockets properly
  - Memory/file descriptor leak
  - Fix: Find and fix application bug

High SYN_RECV (attack?):
  - Many half-open connections
  - Could be SYN flood attack
  - Or slow application accept()
```

**ss vs netstat:**
```
netstat -tan     →  older, slower
ss -tan          →  modern, faster (reads /proc directly)

ss is the recommended tool for modern Linux!
```

---

### 46. Parallel Task Executor

**🎯 Problem Statement:**
Running commands on multiple servers sequentially is slow. We need to execute commands in parallel while controlling concurrency to avoid overwhelming the network.

**🧠 Core Logic Explained:**
1. **Define server list** → Targets for command execution
2. **Create execution function** → Run command on one server
3. **Background execution** → Run functions with `&`
4. **Limit parallelism** → Control max concurrent jobs
5. **Wait for completion** → `wait -n` waits for any job
6. **Aggregate results** → Show success/failure for each

**💡 Key Concepts:**
- `&` → Run command in background
- `wait -n` → Wait for any background job to finish (bash 4.3+)
- `timeout` → Kill command if runs too long
- `ssh -o StrictHostKeyChecking=no` → Don't prompt for new hosts
- Job control → Track and limit parallel processes

```bash
#!/bin/bash
# Parallel Task Executor - Run commands on multiple servers concurrently

# List of target servers to execute commands on
SERVERS="server1 server2 server3 server4 server5"
COMMAND="${1:-uptime}"      # Command to run (default: uptime)
MAX_PARALLEL=5              # Maximum concurrent SSH connections
TIMEOUT=30                  # Max time for each command

echo "=== Parallel Execution ==="
echo "Command: $COMMAND"
echo "Servers: $SERVERS"
echo ""

# Function to execute command on a single server
execute_on_server() {
    local server="$1"       # Target server hostname/IP
    local cmd="$2"          # Command to execute
    local output           # Variable to capture output
    
    # timeout $TIMEOUT         → Kill command if exceeds time limit
    # ssh -o ConnectTimeout=5  → Max 5 seconds to establish connection
    # -o StrictHostKeyChecking=no → Don't prompt for unknown host keys
    # 2>&1                     → Capture both stdout and stderr
    output=$(timeout $TIMEOUT ssh -o ConnectTimeout=5 -o StrictHostKeyChecking=no "$server" "$cmd" 2>&1)
    exit_code=$?             # Capture exit code immediately after command
    
    # Report results with server identification
    if [ $exit_code -eq 0 ]; then
        echo "[$server] SUCCESS:"
        # sed 's/^/  /' → indent each line with 2 spaces for readability
        echo "$output" | sed 's/^/  /'
    else
        echo "[$server] FAILED (exit: $exit_code):"
        echo "$output" | sed 's/^/  /'
    fi
}

# Main parallel execution loop
running=0                   # Track number of running background jobs

for server in $SERVERS; do
    # Start command in background with & (non-blocking)
    execute_on_server "$server" "$COMMAND" &
    ((running++))            # Increment running job counter
    
    # Limit parallelism to avoid overwhelming network/resources
    if [ $running -ge $MAX_PARALLEL ]; then
        # wait -n → Wait for ANY single background job to finish (bash 4.3+)
        # This is key: instead of waiting for all, we wait for one
        # Then we can start the next job immediately
        wait -n
        ((running--))        # One job finished, decrement counter
    fi
done

# Wait for all remaining background jobs to complete
wait
echo -e "\nAll tasks complete!"
```

**Example Execution:**
```bash
$ ./parallel_exec.sh "df -h / | tail -1"

=== Parallel Execution ===
Command: df -h / | tail -1
Servers: server1 server2 server3 server4 server5

[server1] SUCCESS:
  /dev/sda1       50G   23G   25G  48% /
[server3] SUCCESS:
  /dev/sda1       50G   31G   17G  65% /
[server2] SUCCESS:
  /dev/sda1       50G   18G   30G  38% /
[server4] FAILED (exit: 255):
  ssh: connect to host server4 port 22: Connection refused
[server5] SUCCESS:
  /dev/sda1       50G   42G   6G   88% /

All tasks complete!
```

**Parallel Execution Flow:**
```
Without parallelism (sequential):
┌────────────────────────────────────────────────────┐
│ server1 ──────► server2 ──────► server3 ───►...   │
│   30s            30s             30s               │
│ Total: 5 servers × 30s = 150 seconds              │
└────────────────────────────────────────────────────┘

With MAX_PARALLEL=5:
┌────────────────────────────────────────────────────┐
│ server1 ────────────►                              │
│ server2 ────────────►   All run simultaneously!    │
│ server3 ────────────►                              │
│ server4 ────────────►                              │
│ server5 ────────────►                              │
│ Total: max(30s, 30s, 30s, 30s, 30s) = 30 seconds  │
└────────────────────────────────────────────────────┘
```

**Background Jobs & wait -n:**
```
Timeline with MAX_PARALLEL=2 and 5 servers:

T=0:  Start server1 &, Start server2 &  (running=2)
      [server1 ████████████████████]      
      [server2 ████████]  (finishes early)
      
T=8:  wait -n returns (server2 done), Start server3 &
      [server1 ████████████████████]
      [server3 ██████████████]

T=12: wait -n returns (server3 done), Start server4 &
      ... and so on

wait -n advantage:
  - Don't wait for slowest server before starting next
  - Start new job as soon as ANY job finishes
  - Better resource utilization
```

**SSH Options Explained:**
```
-o ConnectTimeout=5        → Don't wait forever for unreachable hosts
-o StrictHostKeyChecking=no → Skip "Are you sure you want to connect?"
                              (Use in automation only, not production!)

For production, use SSH config file (~/.ssh/config):
Host *
    ConnectTimeout 5
    StrictHostKeyChecking accept-new
```

---

### 47. Config File Diff & Merge

**🎯 Problem Statement:**
Configuration changes need review before applying. We need to compare config files, show differences, and safely merge changes with backup.

**🧠 Core Logic Explained:**
1. **Accept two files** → Current and new config
2. **Check if identical** → No action needed if same
3. **Show unified diff** → Highlight line-by-line changes
4. **Present options** → Keep, replace, merge, or compare
5. **Backup before change** → Always preserve original
6. **Interactive merge** → Use vimdiff or sdiff for manual merge

**💡 Key Concepts:**
- `diff -q` → Quick check if files differ (quiet)
- `diff -u` → Unified diff format (most readable)
- `vimdiff` → Side-by-side visual comparison in vim
- `sdiff -o` → Merge interactively to output file
- Always backup → Never lose original config

```bash
#!/bin/bash
# Config File Diff & Merge - Compare and safely merge configuration files

FILE1="$1"                  # Current/existing config file
FILE2="$2"                  # New/proposed config file

# Validate arguments
if [ -z "$FILE1" ] || [ -z "$FILE2" ]; then
    echo "Usage: $0 <current_config> <new_config>"
    exit 1
fi

echo "=== Config Comparison ==="
echo "Current: $FILE1"
echo "New: $FILE2"
echo ""

# Quick check if files are identical (diff -q = quiet mode)
# If identical, diff returns 0 and exits without output
if diff -q "$FILE1" "$FILE2" &>/dev/null; then
    echo "Files are identical"
    exit 0
fi

# Show unified diff format (most readable)
echo "--- Differences ---"
# --color=auto → colorize if terminal supports it
# -u           → unified format (context + changes together)
diff --color=auto -u "$FILE1" "$FILE2"

# Present interactive options to user
echo -e "\n--- Options ---"
echo "1. Keep current"
echo "2. Use new config"
echo "3. Merge interactively"
echo "4. View side-by-side"

read -p "Choice [1-4]: " choice    # Read user's choice

case $choice in
    1)
        # Option 1: Do nothing, keep existing file
        echo "Keeping current config"
        ;;
    2)
        # Option 2: Replace current with new (ALWAYS backup first!)
        # Create timestamp-based backup filename
        backup="${FILE1}.$(date +%Y%m%d_%H%M%S).bak"
        cp "$FILE1" "$backup"       # Save original
        cp "$FILE2" "$FILE1"        # Overwrite with new
        echo "Updated. Backup: $backup"
        ;;
    3)
        # Option 3: Interactive merge using visual tools
        backup="${FILE1}.$(date +%Y%m%d_%H%M%S).bak"
        cp "$FILE1" "$backup"       # Always backup before merge
        
        # Prefer vimdiff if available (powerful visual merge)
        # command -v → check if command exists
        if command -v vimdiff &>/dev/null; then
            vimdiff "$FILE1" "$FILE2"  # Opens side-by-side in vim
        else
            # Fallback to sdiff (simpler side-by-side merge)
            # -o = output file for merged result
            sdiff -o "$FILE1.merged" "$FILE1" "$FILE2"
            mv "$FILE1.merged" "$FILE1"
        fi
        ;;
    4)
        # Option 4: View side-by-side comparison only (no changes)
        # sdiff without -o just displays comparison
        # Pipe to less for paging if output is long
        sdiff "$FILE1" "$FILE2" | less
        ;;
esac
```

**Example Execution:**
```bash
$ ./config_diff.sh /etc/nginx/nginx.conf nginx.conf.new

=== Config Comparison ===
Current: /etc/nginx/nginx.conf
New: nginx.conf.new

--- Differences ---
--- /etc/nginx/nginx.conf       2024-01-10 09:00:00
+++ nginx.conf.new              2024-01-15 14:30:00
@@ -15,7 +15,7 @@
 http {
     include       mime.types;
     default_type  application/octet-stream;
-    keepalive_timeout  65;
+    keepalive_timeout  120;
     
     server {
         listen       80;
@@ -25,6 +25,10 @@
             root   /var/www/html;
             index  index.html;
         }
+        
+        location /api {
+            proxy_pass http://backend:8080;
+        }
     }
 }

--- Options ---
1. Keep current
2. Use new config
3. Merge interactively
4. View side-by-side
Choice [1-4]: 2
Updated. Backup: /etc/nginx/nginx.conf.20240115_143500.bak
```

**Unified Diff Format Explained:**
```
--- old_file    2024-01-10 09:00:00    ← Original file info
+++ new_file    2024-01-15 14:30:00    ← New file info
@@ -15,7 +15,7 @@                       ← Context marker
                                          -15,7 = starting line 15, 7 lines
                                          +15,7 = same range in new file
 http {                                 ← Unchanged context line
     include       mime.types;          ← Unchanged
-    keepalive_timeout  65;             ← REMOVED (starts with -)
+    keepalive_timeout  120;            ← ADDED (starts with +)
     
     server {                           ← Unchanged context
```

**vimdiff vs sdiff:**
```
vimdiff (powerful):
  ┌─────────────────┬─────────────────┐
  │   FILE 1        │    FILE 2       │
  │ ───────────     │ ───────────     │
  │ line 1          │ line 1          │
  │ old value  ←RED │ new value ←GREEN│
  │ line 3          │ line 3          │
  └─────────────────┴─────────────────┘
  Commands: ]c (next diff), [c (prev), do (get other), dp (put)

sdiff (simpler):
  line 1                                line 1
  old value                           | new value
  line 3                                line 3
  
  | = lines differ
  < = only in file 1
  > = only in file 2
```

**Why Always Backup?**
```
Before dangerous operations, ALWAYS create backup:

backup="${FILE}.$(date +%Y%m%d_%H%M%S).bak"

Results in: /etc/nginx/nginx.conf.20240115_143527.bak

Timestamp ensures:
  - Multiple backups don't overwrite each other
  - Easy to find when backup was made
  - Can roll back to any previous version
```

---

### 48. Cron Job Manager

**🎯 Problem Statement:**
Managing cron jobs requires editing crontab directly, which is error-prone. We need a script to list, validate, and manage cron jobs across all users safely.

**🧠 Core Logic Explained:**
1. **List all crons** → Check system crontabs and user crontabs
2. **Validate cron syntax** → Ensure 5 time fields are valid
3. **Find duplicates** → Detect repeated job entries
4. **Track upcoming jobs** → Show jobs running in next hour
5. **Report issues** → Identify malformed entries

**💡 Key Concepts:**
- `crontab -l -u user` → List cron jobs for specific user
- `/etc/cron.d/*` → System-wide cron jobs
- Cron format: `MIN HOUR DAY MONTH WEEKDAY command`
- `grep -qE` → Quiet extended regex match
- `sort | uniq -d` → Find duplicate lines only

```bash
#!/bin/bash
# Cron Job Manager - List, validate, and audit cron jobs across all users

echo "=== Cron Job Manager ==="

# Function to list all cron jobs system-wide
list_all_crons() {
    # System-level cron jobs in /etc/cron.d/
    echo "--- System Crontabs ---"
    for file in /etc/cron.d/*; do
        # [ -f "$file" ] → only process regular files (not directories)
        # grep -v '^#'   → exclude comment lines
        # grep -v '^$'   → exclude blank lines
        [ -f "$file" ] && echo "==> $file" && cat "$file" | grep -v '^#' | grep -v '^$'
    done
    
    # User-specific cron jobs
    echo -e "\n--- User Crontabs ---"
    # Loop through all users defined in /etc/passwd
    for user in $(cut -d: -f1 /etc/passwd); do
        # crontab -l -u "$user" → list crontab for specific user
        # 2>/dev/null → suppress "no crontab for user" errors
        crontab=$(crontab -l -u "$user" 2>/dev/null)
        # Only show if user actually has cron jobs
        if [ -n "$crontab" ]; then
            echo "==> $user"
            echo "$crontab" | grep -v '^#'   # Exclude comments
        fi
    done
}

# Function to validate cron syntax
validate_cron() {
    local cron_line="$1"
    
    # Cron format: MIN HOUR DAY MONTH WEEKDAY command
    # This regex validates the 5 time fields:
    # [0-9*,/-]+ → digits, asterisk, comma, slash, dash
    # \s+        → followed by whitespace
    # Repeated 5 times for: minute, hour, day, month, weekday
    if echo "$cron_line" | grep -qE '^[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+'; then
        echo "Valid: $cron_line"
        return 0                # Exit code 0 = success
    else
        echo "Invalid: $cron_line"
        return 1                # Exit code 1 = failure
    fi
}

# Function to find duplicate cron entries
find_duplicates() {
    echo "--- Checking for Duplicate Jobs ---"
    for user in $(cut -d: -f1 /etc/passwd); do
        # sort | uniq -d → show only lines that appear more than once
        crontab -l -u "$user" 2>/dev/null | grep -v '^#' | grep -v '^$' | sort | uniq -d
    done
}

# Function to show jobs that will run soon
upcoming_jobs() {
    echo "--- Jobs Running in Next Hour ---"
    current_min=$(date +%M)      # Current minute (00-59)
    current_hour=$(date +%H)     # Current hour (00-23)
    
    for user in $(cut -d: -f1 /etc/passwd); do
        # Read each cron line and check if it runs this hour
        crontab -l -u "$user" 2>/dev/null | grep -v '^#' | grep -v '^$' | while read line; do
            min=$(echo "$line" | awk '{print $1}')    # First field = minute
            hour=$(echo "$line" | awk '{print $2}')   # Second field = hour
            
            # Simplified check: job runs this hour if hour matches or is wildcard
            if [ "$hour" == "*" ] || [ "$hour" == "$current_hour" ]; then
                echo "$user: $line"
            fi
        done
    done
}

# Main: dispatch based on command argument
case "$1" in
    list) list_all_crons ;;
    validate) shift; validate_cron "$*" ;;    # shift removes $1, "$*" gets rest
    duplicates) find_duplicates ;;
    upcoming) upcoming_jobs ;;
    *) echo "Usage: $0 <list|validate|duplicates|upcoming>" ;;
esac
```

**Example Execution:**
```bash
$ ./cron_manager.sh list

=== Cron Job Manager ===
--- System Crontabs ---
==> /etc/cron.d/certbot
0 0,12 * * * root /usr/bin/certbot renew --quiet
==> /etc/cron.d/logrotate
0 0 * * * root /usr/sbin/logrotate /etc/logrotate.conf

--- User Crontabs ---
==> deploy
*/5 * * * * /opt/scripts/health_check.sh
0 2 * * * /opt/scripts/backup.sh
==> monitor
* * * * * /opt/scripts/metrics_push.sh

$ ./cron_manager.sh validate "*/5 * * * * /script.sh"
Valid: */5 * * * * /script.sh

$ ./cron_manager.sh validate "5 * /script.sh"
Invalid: 5 * /script.sh

$ ./cron_manager.sh upcoming
--- Jobs Running in Next Hour ---
deploy: */5 * * * * /opt/scripts/health_check.sh
monitor: * * * * * /opt/scripts/metrics_push.sh
```

**Cron Format Reference:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday = 0)
│ │ │ │ │
* * * * * command to execute

Examples:
0 * * * *     → Every hour at minute 0
*/15 * * * *  → Every 15 minutes
0 2 * * *     → Daily at 2:00 AM
0 0 1 * *     → Monthly on the 1st at midnight
0 0 * * 0     → Every Sunday at midnight
```

**Cron Location Hierarchy:**
```
System crontabs:
  /etc/crontab              → Main system crontab
  /etc/cron.d/*             → Drop-in files for system services
  /etc/cron.daily/          → Scripts run daily
  /etc/cron.hourly/         → Scripts run hourly
  /etc/cron.weekly/         → Scripts run weekly
  /etc/cron.monthly/        → Scripts run monthly

User crontabs:
  /var/spool/cron/crontabs/username    → User-specific crontabs
  Access via: crontab -l -u username
```

**Common Cron Issues:**
```
1. PATH not set:
   ✗ 0 * * * * script.sh
   ✓ 0 * * * * /full/path/to/script.sh

2. Output not captured:
   ✗ 0 * * * * /script.sh
   ✓ 0 * * * * /script.sh >> /var/log/script.log 2>&1

3. Environment variables missing:
   Add at top of crontab:
   SHELL=/bin/bash
   PATH=/usr/local/bin:/usr/bin:/bin
   MAILTO=admin@example.com
```

---

### 49. Service Dependency Checker

**🎯 Problem Statement:**
Services often fail at startup because dependencies (database, cache, files) aren't ready. We need to verify all dependencies before starting a service.

**🧠 Core Logic Explained:**
1. **Read config file** → List all dependencies to check
2. **Check port availability** → Use nc (netcat) to test TCP ports
3. **Check URL endpoints** → Use curl to verify HTTP responses
4. **Check file existence** → Verify required files exist
5. **Check environment vars** → Ensure configs are set
6. **Report status** → Show ✓ for success, ✗ for failure
7. **Exit code** → Non-zero if any dependency fails

**💡 Key Concepts:**
- `nc -z -w 5` → Zero I/O mode (port scan), 5s timeout
- `curl -s -f -o /dev/null` → Silent, fail on HTTP error, discard output
- `[ -f "$file" ]` → Check if file exists
- `[ -z "${VAR}" ]` → Check if variable is empty
- Exit code aggregation → Track failures for final status

```bash
#!/bin/bash
# Service Dependency Checker - Verify all dependencies before starting a service

SERVICE_NAME="myapp"
CONFIG_FILE="/etc/myapp/dependencies.conf"

# Config file format (one dependency per line):
# port:localhost:5432:PostgreSQL     → Check TCP port is open
# url:http://redis:6379/ping:Redis   → Check HTTP endpoint responds
# file:/var/run/myapp.pid            → Check file exists
# env:DATABASE_URL                   → Check environment variable is set

# Function to check if a TCP port is reachable
check_port() {
    local host="$1"         # Hostname or IP
    local port="$2"         # Port number
    local name="$3"         # Friendly name for display
    
    # nc (netcat) options:
    # -z    → Zero I/O mode (just check if port is open)
    # -w 5  → Timeout after 5 seconds
    if nc -z -w 5 "$host" "$port" 2>/dev/null; then
        echo "✓ $name ($host:$port)"
        return 0            # Success
    else
        echo "✗ $name ($host:$port) - NOT AVAILABLE"
        return 1            # Failure
    fi
}

# Function to check if an HTTP endpoint responds successfully
check_url() {
    local url="$1"          # Full URL to check
    local name="$2"         # Friendly name
    
    # curl options:
    # -s             → Silent (no progress bar)
    # -f             → Fail silently on HTTP errors (returns exit code 22)
    # -o /dev/null   → Discard response body
    # --connect-timeout 5 → Give up after 5 seconds
    if curl -s -f -o /dev/null --connect-timeout 5 "$url"; then
        echo "✓ $name ($url)"
        return 0
    else
        echo "✗ $name ($url) - NOT RESPONDING"
        return 1
    fi
}

# Function to check if a required file exists
check_file() {
    local file="$1"
    
    # -f tests if path is a regular file (not directory)
    if [ -f "$file" ]; then
        echo "✓ File exists: $file"
        return 0
    else
        echo "✗ File missing: $file"
        return 1
    fi
}

# Function to check if an environment variable is set
check_env() {
    local var="$1"          # Variable name (not value)
    
    # ${!var} → Indirect expansion: get value of variable whose name is in $var
    # Example: if var="DATABASE_URL", ${!var} gives value of $DATABASE_URL
    if [ -n "${!var}" ]; then
        echo "✓ Env set: $var"
        return 0
    else
        echo "✗ Env missing: $var"
        return 1
    fi
}

echo "=== Dependency Check: $SERVICE_NAME ==="
failed=0                    # Counter for failed dependency checks

# Read config file line by line
# IFS=: → Split each line by colons into fields
# read -r type arg1 arg2 arg3 → Read fields into variables
while IFS=: read -r type arg1 arg2 arg3; do
    # Skip empty lines and comments
    [ -z "$type" ] || [[ "$type" == \#* ]] && continue
    
    # Dispatch to appropriate check function based on type
    case "$type" in
        port) check_port "$arg1" "$arg2" "$arg3" || ((failed++)) ;;
        url) check_url "$arg1" "$arg2" || ((failed++)) ;;
        file) check_file "$arg1" || ((failed++)) ;;
        env) check_env "$arg1" || ((failed++)) ;;
    esac
done < "$CONFIG_FILE"       # Redirect config file as input to while loop

echo ""
# Report final status
if [ $failed -gt 0 ]; then
    echo "FAILED: $failed dependencies not met"
    exit 1                  # Exit with error for calling scripts to detect
else
    echo "SUCCESS: All dependencies satisfied"
    exit 0
fi
```

**Example Config File (`/etc/myapp/dependencies.conf`):**
```ini
# Database must be available
port:localhost:5432:PostgreSQL
port:localhost:6379:Redis

# Required HTTP endpoints
url:http://auth-service:8080/health:Auth Service
url:http://storage-service:9000/health:Storage Service

# Required files
file:/etc/myapp/config.yml
file:/var/run/secrets/db_password

# Required environment variables
env:DATABASE_URL
env:REDIS_URL
env:API_KEY
```

**Example Execution:**
```bash
$ ./dependency_check.sh

=== Dependency Check: myapp ===
✓ PostgreSQL (localhost:5432)
✓ Redis (localhost:6379)
✓ Auth Service (http://auth-service:8080/health)
✗ Storage Service (http://storage-service:9000/health) - NOT RESPONDING
✓ File exists: /etc/myapp/config.yml
✗ File missing: /var/run/secrets/db_password
✓ Env set: DATABASE_URL
✓ Env set: REDIS_URL
✗ Env missing: API_KEY

FAILED: 3 dependencies not met
$ echo $?
1
```

**Service Startup Pattern:**
```bash
# In your service start script:

#!/bin/bash
# Start myapp service

# Step 1: Check all dependencies first
if ! /opt/scripts/dependency_check.sh; then
    echo "Cannot start myapp: dependencies not met"
    exit 1
fi

# Step 2: All dependencies OK, start the service
echo "Starting myapp..."
exec /usr/bin/myapp --config /etc/myapp/config.yml
```

**Indirect Variable Expansion:**
```bash
# Normal expansion:
DATABASE_URL="postgres://localhost/db"
echo $DATABASE_URL           # Output: postgres://localhost/db

# Indirect expansion (variable containing variable name):
varname="DATABASE_URL"
echo ${!varname}            # Output: postgres://localhost/db

# How it works:
#   varname = "DATABASE_URL"
#   ${!varname} = "look up the variable whose name is stored in varname"
#               = ${DATABASE_URL}
#               = "postgres://localhost/db"

# Use case: Loop through a list of variable names to check
for var in DATABASE_URL REDIS_URL API_KEY; do
    if [ -n "${!var}" ]; then
        echo "$var is set to: ${!var}"
    fi
done
```

**IFS (Internal Field Separator):**
```
Default IFS = space, tab, newline

IFS=: read -r a b c <<< "host:5432:name"
# a = "host"
# b = "5432"  
# c = "name"

IFS is only changed for the duration of that read command
```

---

### 50. Self-Healing Service Script

**🎯 Problem Statement:**
Services can crash or become unresponsive. Manual intervention takes time. We need automatic monitoring that detects failures and recovers services without human intervention.

**🧠 Core Logic Explained:**
1. **Health check loop** → Continuously monitor service status
2. **Multi-layer check** → systemd status + HTTP health endpoint
3. **Graduated recovery** → Try graceful restart first
4. **Forced recovery** → If graceful fails, force kill and start
5. **Retry tracking** → Count consecutive failures
6. **Escalation** → Alert humans after max retries exceeded
7. **Logging** → Record all actions for debugging

**💡 Key Concepts:**
- `systemctl is-active --quiet` → Check service status silently
- `curl -w "%{http_code}"` → Extract HTTP response code
- Graceful vs Forced restart → Try nice first, then aggressive
- `pkill -9 -f` → Force kill by pattern
- `mail -s` → Send email alerts
- Retry counter → Prevent infinite restart loops

```bash
#!/bin/bash
# Self-Healing Service Script - Automatic monitoring and recovery

SERVICE="nginx"                              # Service to monitor
HEALTH_URL="http://localhost/health"         # Health check endpoint
CHECK_INTERVAL=30                            # Seconds between health checks
MAX_RETRIES=3                                # Failures before attempting recovery
ESCALATION_EMAIL="oncall@company.com"        # Email for escalation alerts
LOG="/var/log/${SERVICE}_healing.log"        # Log file for all actions

# Logging function with timestamp
# tee -a → write to both stdout AND append to log file
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"
}

# Health check function - returns 0 (healthy) or 1 (unhealthy)
check_health() {
    # First check: Is the systemd service running?
    # --quiet → don't output anything, just set exit code
    if ! systemctl is-active --quiet "$SERVICE"; then
        return 1                # Service not running = unhealthy
    fi
    
    # Second check: Does the HTTP health endpoint respond with 200?
    # This catches cases where process is running but not serving
    http_code=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "$HEALTH_URL")
    [ "$http_code" == "200" ]   # Return true if HTTP 200, false otherwise
}

# Recovery function - attempts to restore service
recover_service() {
    log "Attempting recovery..."
    
    # Step 1: Try graceful restart first (least disruptive)
    systemctl restart "$SERVICE"
    sleep 5                     # Give service time to start
    
    # Check if graceful restart worked
    if check_health; then
        log "Recovery successful (graceful restart)"
        return 0                # Success
    fi
    
    # Step 2: Graceful failed, try force kill and start
    log "Graceful restart failed, forcing..."
    systemctl stop "$SERVICE"   # Send SIGTERM
    
    # pkill -9 → Send SIGKILL (force kill, cannot be ignored)
    # -f       → Match against full command line
    # 2>/dev/null → Suppress "no process found" errors
    pkill -9 -f "$SERVICE" 2>/dev/null
    sleep 2                     # Wait for processes to die
    
    systemctl start "$SERVICE"  # Start fresh
    sleep 5                     # Give service time to start
    
    # Check if forced restart worked
    if check_health; then
        log "Recovery successful (forced restart)"
        return 0                # Success
    fi
    
    return 1                    # Recovery failed
}

# Escalation function - alert humans when automation fails
escalate() {
    local message="$1"
    log "ESCALATION: $message"
    # Send email alert using mail command
    # -s → Subject line
    echo "$message" | mail -s "[$SERVICE] Self-Healing Failed" "$ESCALATION_EMAIL"
}

# === Main monitoring loop ===
failure_count=0                 # Track consecutive failures

log "Self-healing monitor started for $SERVICE"

# Infinite loop - this script runs as a daemon
while true; do
    if check_health; then
        # Service is healthy
        # Only log if we were previously in failure state (recovered)
        [ $failure_count -gt 0 ] && log "Service recovered, resetting failure count"
        failure_count=0         # Reset failure counter
    else
        # Service is unhealthy
        ((failure_count++))     # Increment failure counter
        log "Health check failed (count: $failure_count/$MAX_RETRIES)"
        
        # Have we exceeded retry threshold?
        if [ $failure_count -ge $MAX_RETRIES ]; then
            log "Max retries reached, attempting recovery..."
            
            if recover_service; then
                failure_count=0 # Recovery worked, reset counter
            else
                # Recovery failed - escalate to humans
                escalate "Failed to recover $SERVICE after $MAX_RETRIES attempts. Manual intervention required."
                failure_count=0 # Reset to avoid email spam
                sleep 300       # Wait 5 minutes before retrying (backoff)
            fi
        fi
    fi
    
    sleep $CHECK_INTERVAL       # Wait before next health check
done
```

**Example Execution (as a daemon):**
```bash
# Start as background process
$ nohup ./self_healing.sh &
[1] 12345

# Check the log
$ tail -f /var/log/nginx_healing.log

[2024-01-15 14:00:00] Self-healing monitor started for nginx
[2024-01-15 14:00:30] Health check passed
[2024-01-15 14:01:00] Health check passed
[2024-01-15 14:01:30] Health check failed (count: 1/3)
[2024-01-15 14:02:00] Health check failed (count: 2/3)
[2024-01-15 14:02:30] Health check failed (count: 3/3)
[2024-01-15 14:02:30] Max retries reached, attempting recovery...
[2024-01-15 14:02:35] Recovery successful (graceful restart)
[2024-01-15 14:03:05] Service recovered, resetting failure count
```

**Self-Healing Flow Diagram:**
```
                    ┌─────────────┐
                    │   START     │
                    └──────┬──────┘
                           │
             ┌─────────────▼─────────────┐
             │      Check Health         │◄─────────────────────┐
             └─────────────┬─────────────┘                      │
                           │                                    │
              ┌────────────┴────────────┐                       │
              │                         │                       │
           HEALTHY                  UNHEALTHY                   │
              │                         │                       │
              ▼                         ▼                       │
    ┌─────────────────┐     ┌─────────────────┐                 │
    │ Reset counter   │     │ Increment       │                 │
    │ failure_count=0 │     │ failure_count++ │                 │
    └────────┬────────┘     └────────┬────────┘                 │
             │                       │                          │
             │              ┌────────┴────────┐                 │
             │              │ count >= 3?     │                 │
             │              └────────┬────────┘                 │
             │               NO      │      YES                 │
             │               │       │       │                  │
             │               ▼       │       ▼                  │
             │          ┌────────┐   │  ┌─────────────┐         │
             │          │ Wait   │   │  │   RECOVER   │         │
             │          │ 30 sec │   │  └──────┬──────┘         │
             │          └────┬───┘   │         │                │
             │               │       │    SUCCESS?              │
             │               │       │    /      \              │
             │               │       │  YES       NO            │
             │               │       │   │         │            │
             │               │       │   ▼         ▼            │
             │               │       │ Reset   ESCALATE         │
             │               │       │ counter  + Wait 5min     │
             │               │       │         │                │
             └───────────────┴───────┴─────────┴────────────────┘
```

**Graceful vs Forced Restart:**
```
Graceful (systemctl restart):
  ┌─────────────────────────────────────────┐
  │ 1. Send SIGTERM to process              │
  │ 2. Process handles signal gracefully:   │
  │    - Finish current requests            │
  │    - Close connections properly         │
  │    - Clean up resources                 │
  │ 3. Process exits                        │
  │ 4. systemd starts new process           │
  └─────────────────────────────────────────┘
  Advantage: No dropped connections, clean shutdown

Forced (pkill -9):
  ┌─────────────────────────────────────────┐
  │ 1. Send SIGKILL to process              │
  │ 2. Kernel immediately terminates it     │
  │    - No chance to clean up              │
  │    - In-flight requests dropped         │
  │    - Connections severed                │
  │ 3. systemd starts new process           │
  └─────────────────────────────────────────┘
  Advantage: Works even if process is stuck/hung

Use forced restart only when graceful fails!
```

**Running as a Systemd Service:**
```ini
# /etc/systemd/system/self-healing.service
[Unit]
Description=Self-Healing Monitor for nginx
After=network.target nginx.service

[Service]
Type=simple
ExecStart=/opt/scripts/self_healing.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl enable self-healing
sudo systemctl start self-healing
```

---

## Quick Reference Cheat Sheet

### Essential Commands for DevOps

| Task | Command |
|------|---------|
| Find large files | `find / -size +100M -type f 2>/dev/null` |
| Check listening ports | `ss -tulpn` or `netstat -tulpn` |
| Watch log in real-time | `tail -f /var/log/syslog \| grep ERROR` |
| Disk usage by folder | `du -sh /* 2>/dev/null \| sort -h` |
| Memory by process | `ps aux --sort=-%mem \| head` |
| Kill process by name | `pkill -f "process_name"` |
| Parallel SSH | `parallel-ssh -h hosts.txt -i "command"` |
| JSON pretty print | `cat file.json \| jq '.'` |
| Base64 encode | `echo -n "text" \| base64` |
| Generate UUID | `uuidgen` or `cat /proc/sys/kernel/random/uuid` |

### Useful Bash One-Liners

```bash
# Find and replace in multiple files
find . -name "*.conf" -exec sed -i 's/old/new/g' {} \;

# Get public IP
curl -s ifconfig.me

# Monitor file changes
watch -n 1 'ls -la /path/to/file'

# Count files by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Remove lines matching pattern
sed -i '/pattern/d' file.txt

# Extract IPs from text
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' file.txt

# Calculate file checksum
sha256sum file.txt

# Compare two directories
diff -rq dir1/ dir2/

# Find files modified in last hour
find /path -mmin -60 -type f
```
