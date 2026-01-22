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
LOG_FILE="/var/log/myapp/app.log"
MAX_SIZE=104857600  # 100MB in bytes
KEEP_ROTATIONS=5

if [ -f "$LOG_FILE" ]; then
    size=$(stat -c%s "$LOG_FILE")
    if [ $size -gt $MAX_SIZE ]; then
        timestamp=$(date +%Y%m%d_%H%M%S)
        mv "$LOG_FILE" "${LOG_FILE}.${timestamp}"
        gzip "${LOG_FILE}.${timestamp}"
        touch "$LOG_FILE"
        
        # Remove old rotations beyond limit
        ls -t ${LOG_FILE}.*.gz 2>/dev/null | tail -n +$((KEEP_ROTATIONS+1)) | xargs -r rm
        echo "Log rotated: ${LOG_FILE}.${timestamp}.gz"
    fi
fi
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
LOG="/var/log/apache2/access.log"

echo "=== 404 Errors Summary ==="
awk '$9 == 404 {print $7}' "$LOG" | sort | uniq -c | sort -rn | head -20
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
LOG="/var/log/nginx/access.log"

echo "=== Top 10 IPs by Request Count ==="
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -10

# With requests per second calculation
echo -e "\n=== IPs with >100 requests in last minute ==="
awk -v threshold=100 '
{
    ip[$1]++
}
END {
    for (i in ip) {
        if (ip[i] > threshold) print ip[i], i
    }
}' "$LOG" | sort -rn
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
LOG="/var/log/nginx/access.log"
OUTPUT="5xx_errors_$(date +%F).log"

awk '$9 ~ /^5[0-9][0-9]$/ {print $4, $5, $1, $9, $7}' "$LOG" > "$OUTPUT"

count=$(wc -l < "$OUTPUT")
echo "Found $count 5xx errors. Saved to $OUTPUT"

# Group by error type
echo -e "\n=== Error Distribution ==="
awk '{print $4}' "$OUTPUT" | sort | uniq -c | sort -rn
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
LOG="/var/log/application.log"
ALERT_EMAIL="admin@company.com"
SLACK_WEBHOOK="https://hooks.slack.com/services/XXX"

tail -F "$LOG" | while read line; do
    if echo "$line" | grep -q "CRITICAL\|FATAL\|OOM"; then
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        message="[$timestamp] ALERT: $line"
        
        # Send to Slack
        curl -s -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$message\"}" "$SLACK_WEBHOOK"
        
        # Also log locally
        echo "$message" >> /var/log/alerts.log
    fi
done
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
LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/mnt/nfs/logs/$(hostname)"
COMPRESS_DAYS=1
DELETE_DAYS=30

mkdir -p "$ARCHIVE_DIR"

# Compress logs older than 1 day
find "$LOG_DIR" -name "*.log" -type f -mtime +$COMPRESS_DAYS ! -name "*.gz" -exec gzip {} \;

# Move compressed to archive
find "$LOG_DIR" -name "*.gz" -type f -mtime +$COMPRESS_DAYS -exec mv {} "$ARCHIVE_DIR/" \;

# Delete archives older than 30 days
find "$ARCHIVE_DIR" -name "*.gz" -type f -mtime +$DELETE_DAYS -delete

echo "Log archival complete: $(date)"
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
LOG_DIR="/var/log"
THRESHOLD_MB=500

find "$LOG_DIR" -name "*.log" -type f | while read file; do
    size_mb=$(du -m "$file" | cut -f1)
    if [ "$size_mb" -gt "$THRESHOLD_MB" ]; then
        echo "WARNING: $file is ${size_mb}MB (threshold: ${THRESHOLD_MB}MB)"
        # Optional: Auto-truncate
        # > "$file"
    fi
done
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
SERVERS="server1.example.com server2.example.com server3.example.com"
REPORT="/tmp/health_report_$(date +%F).txt"

echo "=== Health Check Report $(date) ===" > "$REPORT"

for server in $SERVERS; do
    echo -n "Checking $server... "
    
    # Ping check
    if ping -c 1 -W 2 "$server" &>/dev/null; then
        ping_status="OK"
    else
        ping_status="FAIL"
    fi
    
    # HTTP check (port 80)
    http_code=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "http://$server/health" 2>/dev/null)
    
    # SSH check (port 22)
    if nc -z -w 2 "$server" 22 &>/dev/null; then
        ssh_status="OK"
    else
        ssh_status="FAIL"
    fi
    
    echo "$server | Ping: $ping_status | HTTP: $http_code | SSH: $ssh_status" | tee -a "$REPORT"
done
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
MEM_THRESHOLD=80  # percentage

echo "=== Top 10 Memory Consumers ==="
ps aux --sort=-%mem | head -11 | awk 'NR>1 {printf "%-10s %-8s %-8s %s\n", $1, $2, $4"%", $11}'

echo -e "\n=== Processes exceeding ${MEM_THRESHOLD}% memory ==="
ps aux | awk -v thresh="$MEM_THRESHOLD" '
NR>1 && $4 > thresh {
    printf "ALERT: PID %s (%s) using %.1f%% memory\n", $2, $11, $4
}'
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
echo "=== Zombie Process Report ==="

zombies=$(ps aux | awk '$8 ~ /Z/ {print $2}')

if [ -z "$zombies" ]; then
    echo "No zombie processes found."
    exit 0
fi

for pid in $zombies; do
    ppid=$(ps -o ppid= -p $pid 2>/dev/null | tr -d ' ')
    pname=$(ps -o comm= -p $ppid 2>/dev/null)
    echo "Zombie PID: $pid | Parent PID: $ppid ($pname)"
    
    # Uncomment to kill parent (use with caution!)
    # kill -9 $ppid
done

echo -e "\nTotal zombies: $(echo $zombies | wc -w)"
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
CPU_THRESHOLD=90
DURATION=300  # 5 minutes
INTERVAL=60   # Check every minute
declare -A cpu_track

while true; do
    timestamp=$(date +%s)
    
    ps aux --sort=-%cpu | awk -v thresh="$CPU_THRESHOLD" 'NR>1 && $3>thresh {print $2, $3}' | \
    while read pid cpu; do
        process_name=$(ps -o comm= -p $pid 2>/dev/null)
        
        if [ -n "${cpu_track[$pid]}" ]; then
            start_time=${cpu_track[$pid]}
            elapsed=$((timestamp - start_time))
            
            if [ $elapsed -ge $DURATION ]; then
                echo "ALERT: $process_name (PID $pid) at ${cpu}% CPU for ${elapsed}s"
            fi
        else
            cpu_track[$pid]=$timestamp
        fi
    done
    
    sleep $INTERVAL
done
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
THRESHOLD=80  # % utilization

echo "=== Disk I/O Report ==="
iostat -dx 1 2 | tail -n +4 | awk -v thresh="$THRESHOLD" '
NF > 0 && $1 !~ /^Device/ && $1 !~ /^$/ {
    util = $NF + 0
    if (util > thresh) {
        printf "ALERT: %s at %.1f%% utilization\n", $1, util
    } else {
        printf "OK: %s at %.1f%% utilization\n", $1, util
    }
}'
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
TIMEWAIT_THRESHOLD=1000

echo "=== TCP Connection States ==="
ss -tan | awk 'NR>1 {state[$1]++} END {for (s in state) print s, state[s]}' | sort -k2 -rn

timewait=$(ss -tan state time-wait | wc -l)
if [ "$timewait" -gt "$TIMEWAIT_THRESHOLD" ]; then
    echo "WARNING: $timewait TIME_WAIT connections (threshold: $TIMEWAIT_THRESHOLD)"
fi

echo -e "\n=== Connections per Port ==="
ss -tan | awk 'NR>1 {split($4,a,":"); port=a[length(a)]; ports[port]++} END {for (p in ports) print p, ports[p]}' | sort -k2 -rn | head -10
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
DOMAINS="example.com api.example.com shop.example.com"
WARN_DAYS=30
CRIT_DAYS=7

for domain in $DOMAINS; do
    expiry_date=$(echo | openssl s_client -servername "$domain" -connect "$domain:443" 2>/dev/null | \
                  openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
    
    if [ -z "$expiry_date" ]; then
        echo "ERROR: Cannot check $domain"
        continue
    fi
    
    expiry_epoch=$(date -d "$expiry_date" +%s)
    current_epoch=$(date +%s)
    days_left=$(( (expiry_epoch - current_epoch) / 86400 ))
    
    if [ $days_left -le $CRIT_DAYS ]; then
        echo "CRITICAL: $domain expires in $days_left days ($expiry_date)"
    elif [ $days_left -le $WARN_DAYS ]; then
        echo "WARNING: $domain expires in $days_left days ($expiry_date)"
    else
        echo "OK: $domain expires in $days_left days"
    fi
done
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
SERVERS_FILE="/etc/server_list.txt"
REPORT="/var/www/html/uptime_report.html"

cat > "$REPORT" << 'EOF'
<!DOCTYPE html>
<html><head><title>Server Uptime Report</title>
<style>table{border-collapse:collapse}td,th{border:1px solid #ddd;padding:8px}</style>
</head><body>
<h1>Server Uptime Report - $(date)</h1>
<table><tr><th>Server</th><th>Uptime</th><th>Load</th><th>Status</th></tr>
EOF

while read server; do
    result=$(ssh -o ConnectTimeout=5 "$server" "uptime" 2>/dev/null)
    if [ $? -eq 0 ]; then
        uptime=$(echo "$result" | awk -F'up ' '{print $2}' | awk -F',' '{print $1}')
        load=$(echo "$result" | awk -F'load average: ' '{print $2}')
        status="<span style='color:green'>UP</span>"
    else
        uptime="N/A"
        load="N/A"
        status="<span style='color:red'>DOWN</span>"
    fi
    echo "<tr><td>$server</td><td>$uptime</td><td>$load</td><td>$status</td></tr>" >> "$REPORT"
done < "$SERVERS_FILE"

echo "</table></body></html>" >> "$REPORT"
echo "Report generated: $REPORT"
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
SOURCE="/var/www/html"
BACKUP_BASE="/backup/www"
DATE=$(date +%Y-%m-%d_%H%M)
LATEST="$BACKUP_BASE/latest"
CURRENT="$BACKUP_BASE/$DATE"
KEEP_DAYS=7

# Create backup with hard links to previous
if [ -d "$LATEST" ]; then
    rsync -av --delete --link-dest="$LATEST" "$SOURCE/" "$CURRENT/"
else
    rsync -av --delete "$SOURCE/" "$CURRENT/"
fi

# Update latest symlink
rm -f "$LATEST"
ln -s "$CURRENT" "$LATEST"

# Remove old backups
find "$BACKUP_BASE" -maxdepth 1 -type d -mtime +$KEEP_DAYS ! -name "latest" -exec rm -rf {} \;

echo "Incremental backup complete: $CURRENT"
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
MYSQL_USER="backup_user"
MYSQL_PASS="secure_password"
BACKUP_DIR="/backup/mysql"
S3_BUCKET="s3://company-backups/mysql"
DATE=$(date +%Y-%m-%d)
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

# Get list of databases (exclude system DBs)
databases=$(mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" -e "SHOW DATABASES;" | grep -Ev "(Database|information_schema|performance_schema|sys)")

for db in $databases; do
    backup_file="$BACKUP_DIR/${db}_${DATE}.sql.gz"
    
    echo "Backing up $db..."
    mysqldump -u"$MYSQL_USER" -p"$MYSQL_PASS" --single-transaction --routines "$db" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        echo "Uploading $db to S3..."
        aws s3 cp "$backup_file" "$S3_BUCKET/$(hostname)/" --quiet
    else
        echo "ERROR: Backup failed for $db"
    fi
done

# Cleanup old local backups
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "MySQL backup complete: $(date)"
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
PG_USER="postgres"
BACKUP_DIR="/backup/postgresql"
S3_BUCKET="s3://company-backups/postgresql"
DATE=$(date +%Y-%m-%d_%H%M)
DB_NAME="production"

mkdir -p "$BACKUP_DIR"

# Full backup with custom format (supports parallel restore)
backup_file="$BACKUP_DIR/${DB_NAME}_${DATE}.dump"
pg_dump -U "$PG_USER" -Fc -f "$backup_file" "$DB_NAME"

if [ $? -eq 0 ]; then
    # Get backup size
    size=$(du -h "$backup_file" | cut -f1)
    echo "Backup successful: $backup_file ($size)"
    
    # Upload to S3 with server-side encryption
    aws s3 cp "$backup_file" "$S3_BUCKET/" --sse AES256
    
    # Verify upload
    if aws s3 ls "$S3_BUCKET/$(basename $backup_file)" &>/dev/null; then
        echo "S3 upload verified"
        rm "$backup_file"  # Remove local copy after verified upload
    fi
else
    echo "ERROR: Backup failed" >&2
    exit 1
fi
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
BACKUP_FILE="$1"
TEST_DB="backup_test_$(date +%s)"
PG_USER="postgres"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

echo "=== Backup Verification Started ==="

# Create test database
createdb -U "$PG_USER" "$TEST_DB"

# Restore backup
echo "Restoring to test database..."
pg_restore -U "$PG_USER" -d "$TEST_DB" "$BACKUP_FILE" 2>/dev/null

if [ $? -eq 0 ]; then
    # Run integrity checks
    table_count=$(psql -U "$PG_USER" -d "$TEST_DB" -t -c "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='public'")
    row_counts=$(psql -U "$PG_USER" -d "$TEST_DB" -c "SELECT schemaname, relname, n_live_tup FROM pg_stat_user_tables ORDER BY n_live_tup DESC LIMIT 5")
    
    echo "Verification PASSED"
    echo "Tables: $table_count"
    echo "$row_counts"
else
    echo "Verification FAILED"
fi

# Cleanup
dropdb -U "$PG_USER" "$TEST_DB"
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
SOURCE="/data/critical"
DR_SERVER="dr-server.example.com"
DR_PATH="/data/replicated"
BANDWIDTH="10000"  # KB/s limit
LOG="/var/log/dr_sync.log"

echo "=== DR Sync Started: $(date) ===" >> "$LOG"

# Sync with bandwidth limit and checksum verification
rsync -avz --checksum --delete \
    --bwlimit=$BANDWIDTH \
    --log-file="$LOG" \
    "$SOURCE/" "$DR_SERVER:$DR_PATH/"

exit_code=$?

# Verify sync
if [ $exit_code -eq 0 ]; then
    local_count=$(find "$SOURCE" -type f | wc -l)
    remote_count=$(ssh "$DR_SERVER" "find $DR_PATH -type f | wc -l")
    
    if [ "$local_count" -eq "$remote_count" ]; then
        echo "DR Sync VERIFIED: $local_count files" >> "$LOG"
    else
        echo "WARNING: File count mismatch (local: $local_count, remote: $remote_count)" >> "$LOG"
    fi
else
    echo "ERROR: DR Sync failed with exit code $exit_code" >> "$LOG"
fi
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
CURRENT_ENV=$(cat /etc/active_environment)  # "blue" or "green"
NGINX_CONF="/etc/nginx/conf.d/app.conf"
HEALTH_ENDPOINT="/health"

if [ "$CURRENT_ENV" == "blue" ]; then
    NEW_ENV="green"
    NEW_UPSTREAM="10.0.2.0"
else
    NEW_ENV="blue"
    NEW_UPSTREAM="10.0.1.0"
fi

echo "Switching from $CURRENT_ENV to $NEW_ENV..."

# Health check new environment
for i in {1..5}; do
    status=$(curl -s -o /dev/null -w "%{http_code}" "http://$NEW_UPSTREAM$HEALTH_ENDPOINT")
    if [ "$status" == "200" ]; then
        echo "Health check $i: OK"
    else
        echo "Health check failed. Aborting deployment."
        exit 1
    fi
    sleep 2
done

# Update nginx configuration
sed -i "s/server 10.0.[12].0/server $NEW_UPSTREAM/" "$NGINX_CONF"

# Reload nginx
nginx -t && systemctl reload nginx

if [ $? -eq 0 ]; then
    echo "$NEW_ENV" > /etc/active_environment
    echo "Successfully switched to $NEW_ENV"
else
    echo "Failed to reload nginx. Rolling back..."
    git checkout "$NGINX_CONF"
    exit 1
fi
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
SERVERS="app1 app2 app3 app4"
SERVICE="myapp"
HEALTH_URL_TEMPLATE="http://%s:8080/health"
DRAIN_TIME=30
HEALTH_TIMEOUT=60

for server in $SERVERS; do
    echo "=== Processing $server ==="
    
    # Drain connections
    echo "Draining connections ($DRAIN_TIME seconds)..."
    ssh "$server" "touch /tmp/drain_flag"
    sleep $DRAIN_TIME
    
    # Restart service
    echo "Restarting $SERVICE..."
    ssh "$server" "systemctl restart $SERVICE"
    
    # Wait for healthy
    health_url=$(printf "$HEALTH_URL_TEMPLATE" "$server")
    echo "Waiting for $server to become healthy..."
    
    for i in $(seq 1 $HEALTH_TIMEOUT); do
        if curl -s "$health_url" | grep -q "ok"; then
            echo "$server is healthy after $i seconds"
            ssh "$server" "rm -f /tmp/drain_flag"
            break
        fi
        
        if [ $i -eq $HEALTH_TIMEOUT ]; then
            echo "ERROR: $server failed to become healthy"
            exit 1
        fi
        sleep 1
    done
    
    echo "$server complete. Moving to next..."
done

echo "Rolling restart complete!"
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
# Save as .git/hooks/pre-commit

echo "Running pre-commit checks..."

# Check for secrets/passwords
if git diff --cached --name-only | xargs grep -l -E "(password|secret|api_key)\s*=\s*['\"][^'\"]+['\"]" 2>/dev/null; then
    echo "ERROR: Potential secrets detected in staged files!"
    exit 1
fi

# Check for debug statements
if git diff --cached --name-only -- '*.py' | xargs grep -l "import pdb\|breakpoint()" 2>/dev/null; then
    echo "ERROR: Debug statements found in Python files!"
    exit 1
fi

# Syntax check for shell scripts
for file in $(git diff --cached --name-only -- '*.sh'); do
    if ! bash -n "$file"; then
        echo "ERROR: Syntax error in $file"
        exit 1
    fi
done

# Check YAML syntax
for file in $(git diff --cached --name-only -- '*.yml' '*.yaml'); do
    if ! python -c "import yaml; yaml.safe_load(open('$file'))" 2>/dev/null; then
        echo "ERROR: Invalid YAML in $file"
        exit 1
    fi
done

echo "All pre-commit checks passed!"
exit 0
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
# Get version from git tags
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
VERSION=${LAST_TAG#v}

# Parse semantic version
IFS='.' read -r major minor patch <<< "$VERSION"

# Determine version bump from commit messages
if git log --oneline "$LAST_TAG"..HEAD | grep -qi "BREAKING\|major"; then
    NEW_VERSION="$((major+1)).0.0"
elif git log --oneline "$LAST_TAG"..HEAD | grep -qi "feat\|feature"; then
    NEW_VERSION="$major.$((minor+1)).0"
else
    NEW_VERSION="$major.$minor.$((patch+1))"
fi

# Add build metadata
COMMIT_SHORT=$(git rev-parse --short HEAD)
BUILD_NUM=${BUILD_NUMBER:-local}
FULL_VERSION="${NEW_VERSION}+build.${BUILD_NUM}.${COMMIT_SHORT}"

echo "Previous: $VERSION"
echo "New: $FULL_VERSION"

# Export for use in CI
echo "VERSION=$NEW_VERSION" >> "$GITHUB_ENV" 2>/dev/null || true
echo "FULL_VERSION=$FULL_VERSION" >> "$GITHUB_ENV" 2>/dev/null || true

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
ENV=${1:-development}
TEMPLATE="config/app.conf.template"
OUTPUT="config/app.conf"

# Load environment-specific variables
source "config/${ENV}.env"

# Required variables check
REQUIRED_VARS="DB_HOST DB_NAME API_KEY"
for var in $REQUIRED_VARS; do
    if [ -z "${!var}" ]; then
        echo "ERROR: $var is not set"
        exit 1
    fi
done

# Generate config from template
envsubst < "$TEMPLATE" > "$OUTPUT"

# Verify no unreplaced variables
if grep -q '\${' "$OUTPUT"; then
    echo "WARNING: Unreplaced variables in config:"
    grep '\${' "$OUTPUT"
    exit 1
fi

echo "Generated $OUTPUT for $ENV environment"
chmod 600 "$OUTPUT"
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
echo "=== Docker Cleanup Script ==="

# Show current disk usage
echo "Current Docker disk usage:"
docker system df

# Remove stopped containers
stopped=$(docker ps -aq -f status=exited | wc -l)
if [ "$stopped" -gt 0 ]; then
    echo "Removing $stopped stopped containers..."
    docker container prune -f
fi

# Remove dangling images
dangling=$(docker images -f "dangling=true" -q | wc -l)
if [ "$dangling" -gt 0 ]; then
    echo "Removing $dangling dangling images..."
    docker image prune -f
fi

# Remove images older than 30 days (excluding latest tags)
echo "Removing images older than 30 days..."
docker images --format "{{.ID}} {{.CreatedSince}}" | \
    grep -E "months|weeks" | \
    awk '{print $1}' | \
    xargs -r docker rmi 2>/dev/null

# Remove unused volumes
echo "Removing unused volumes..."
docker volume prune -f

# Final report
echo -e "\n=== Cleanup Complete ==="
docker system df
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
NAMESPACE=${1:-default}
DEPLOYMENT=${2:-""}

if [ -z "$DEPLOYMENT" ]; then
    echo "Usage: $0 <namespace> <deployment>"
    echo "Available deployments:"
    kubectl get deployments -n "$NAMESPACE" -o name
    exit 1
fi

echo "Restarting $DEPLOYMENT in $NAMESPACE..."

# Check current status
kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o wide

# Perform rolling restart
kubectl rollout restart deployment/"$DEPLOYMENT" -n "$NAMESPACE"

# Watch rollout status
kubectl rollout status deployment/"$DEPLOYMENT" -n "$NAMESPACE" --timeout=300s

if [ $? -eq 0 ]; then
    echo "Restart successful!"
    kubectl get pods -n "$NAMESPACE" -l "app=$DEPLOYMENT" -o wide
else
    echo "Restart failed. Rolling back..."
    kubectl rollout undo deployment/"$DEPLOYMENT" -n "$NAMESPACE"
    exit 1
fi
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
RELEASE_NAME="$1"
CHART_PATH="$2"
NAMESPACE="${3:-default}"
VALUES_FILE="${4:-values.yaml}"

if [ -z "$RELEASE_NAME" ] || [ -z "$CHART_PATH" ]; then
    echo "Usage: $0 <release_name> <chart_path> [namespace] [values_file]"
    exit 1
fi

echo "=== Helm Release Manager ==="
echo "Release: $RELEASE_NAME"
echo "Chart: $CHART_PATH"
echo "Namespace: $NAMESPACE"

# Lint chart
echo -e "\n--- Linting chart ---"
helm lint "$CHART_PATH" -f "$VALUES_FILE"

# Show diff (requires helm-diff plugin)
echo -e "\n--- Changes preview ---"
helm diff upgrade "$RELEASE_NAME" "$CHART_PATH" \
    -n "$NAMESPACE" \
    -f "$VALUES_FILE" \
    --allow-unreleased 2>/dev/null || echo "Install helm-diff for change preview"

# Confirm deployment
read -p "Proceed with deployment? (y/n): " confirm
if [ "$confirm" != "y" ]; then
    echo "Deployment cancelled"
    exit 0
fi

# Deploy with atomic (auto-rollback on failure)
helm upgrade --install "$RELEASE_NAME" "$CHART_PATH" \
    -n "$NAMESPACE" \
    -f "$VALUES_FILE" \
    --atomic \
    --timeout 10m \
    --wait

if [ $? -eq 0 ]; then
    echo "Deployment successful!"
    helm status "$RELEASE_NAME" -n "$NAMESPACE"
else
    echo "Deployment failed (auto-rolled back)"
    exit 1
fi
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
LOG="/var/log/auth.log"
THRESHOLD=10
BLOCK_LIST="/etc/hosts.deny"

echo "=== Failed SSH Login Report ==="

# Find failed attempts
failed_ips=$(grep "Failed password" "$LOG" | \
    grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | \
    sort | uniq -c | sort -rn)

echo "IP Address       Attempts"
echo "-------------------------"
echo "$failed_ips" | head -20

# Block IPs exceeding threshold
echo -e "\n=== Blocking IPs with >$THRESHOLD attempts ==="
echo "$failed_ips" | while read count ip; do
    if [ "$count" -gt "$THRESHOLD" ]; then
        if ! grep -q "$ip" "$BLOCK_LIST" 2>/dev/null; then
            echo "Blocking $ip ($count attempts)"
            echo "sshd: $ip" >> "$BLOCK_LIST"
        fi
    fi
done

# Summary by country (requires geoiplookup)
echo -e "\n=== Geographic Distribution ==="
echo "$failed_ips" | awk '{print $2}' | while read ip; do
    country=$(geoiplookup "$ip" 2>/dev/null | cut -d: -f2 | cut -d, -f1)
    echo "${country:-Unknown}"
done | sort | uniq -c | sort -rn | head -10
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
WATCH_DIRS="/etc /usr/bin /usr/sbin"
BASELINE="/var/lib/integrity/baseline.md5"
ALERT_LOG="/var/log/integrity_alerts.log"

generate_baseline() {
    echo "Generating baseline checksums..."
    > "$BASELINE"
    for dir in $WATCH_DIRS; do
        find "$dir" -type f -exec md5sum {} \; >> "$BASELINE" 2>/dev/null
    done
    echo "Baseline saved: $(wc -l < "$BASELINE") files"
}

check_integrity() {
    if [ ! -f "$BASELINE" ]; then
        echo "No baseline found. Generate first with: $0 --init"
        exit 1
    fi
    
    echo "Checking file integrity..."
    changes=0
    
    while IFS=' ' read -r hash file; do
        if [ -f "$file" ]; then
            current_hash=$(md5sum "$file" 2>/dev/null | cut -d' ' -f1)
            if [ "$hash" != "$current_hash" ]; then
                echo "MODIFIED: $file" | tee -a "$ALERT_LOG"
                ((changes++))
            fi
        else
            echo "DELETED: $file" | tee -a "$ALERT_LOG"
            ((changes++))
        fi
    done < "$BASELINE"
    
    # Check for new files
    for dir in $WATCH_DIRS; do
        find "$dir" -type f | while read file; do
            if ! grep -q "$file" "$BASELINE"; then
                echo "NEW: $file" | tee -a "$ALERT_LOG"
                ((changes++))
            fi
        done
    done
    
    echo "Integrity check complete. Changes detected: $changes"
}

case "$1" in
    --init) generate_baseline ;;
    --check) check_integrity ;;
    *) echo "Usage: $0 [--init|--check]" ;;
esac
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
ALLOWED_PORTS="22 80 443"
SCAN_TARGET="${1:-localhost}"

echo "=== Port Scan: $SCAN_TARGET ==="
echo "Allowed ports: $ALLOWED_PORTS"
echo ""

# Scan common ports
open_ports=""
for port in {1..1024}; do
    (echo >/dev/tcp/"$SCAN_TARGET"/$port) 2>/dev/null && open_ports="$open_ports $port"
done

# Alternative: use nmap if available
# open_ports=$(nmap -p 1-1024 --open "$SCAN_TARGET" -oG - | grep -oP '\d+/open' | cut -d/ -f1 | tr '\n' ' ')

echo "Open ports: $open_ports"

# Check for unauthorized ports
echo -e "\n=== Compliance Check ==="
for port in $open_ports; do
    if echo "$ALLOWED_PORTS" | grep -qw "$port"; then
        echo "OK: Port $port (allowed)"
    else
        service=$(getent services "$port" 2>/dev/null | cut -d' ' -f1)
        echo "WARNING: Port $port ($service) - NOT in allowed list!"
    fi
done
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
echo "=== User Security Audit Report ==="
echo "Generated: $(date)"
echo ""

# Users with login shells
echo "=== Users with Login Shells ==="
grep -E '/bin/(ba)?sh$' /etc/passwd | cut -d: -f1,3,6

# Users with UID 0 (root equivalent)
echo -e "\n=== Users with UID 0 (Root) ==="
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Sudo access
echo -e "\n=== Sudo Access ==="
grep -v '^#' /etc/sudoers 2>/dev/null | grep -v '^$' | grep -E 'ALL|NOPASSWD'
for file in /etc/sudoers.d/*; do
    [ -f "$file" ] && cat "$file" 2>/dev/null
done

# Users with empty passwords
echo -e "\n=== Users with Empty Passwords ==="
awk -F: '$2 == "" {print $1}' /etc/shadow 2>/dev/null

# Last login for all users
echo -e "\n=== Last Login (Recent 30 days) ==="
lastlog -t 30 | grep -v "Never logged in"

# SSH keys
echo -e "\n=== SSH Authorized Keys ==="
for home in /home/*; do
    user=$(basename "$home")
    if [ -f "$home/.ssh/authorized_keys" ]; then
        keys=$(wc -l < "$home/.ssh/authorized_keys")
        echo "$user: $keys authorized keys"
    fi
done
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
LOG="/var/log/auth.log"
OUTPUT="/var/log/sudo_audit_$(date +%F).log"

echo "=== Sudo Command Audit ===" | tee "$OUTPUT"
echo "Date range: Last 24 hours" | tee -a "$OUTPUT"
echo "" | tee -a "$OUTPUT"

# Extract sudo commands
grep "sudo:" "$LOG" | grep "COMMAND=" | \
    awk -F'[:,]' '{
        for(i=1;i<=NF;i++) {
            if($i ~ /USER=/) user=$i
            if($i ~ /COMMAND=/) cmd=$i
        }
        print $1, user, cmd
    }' | tee -a "$OUTPUT"

# Summary by user
echo -e "\n=== Commands per User ===" | tee -a "$OUTPUT"
grep "sudo:" "$LOG" | grep "COMMAND=" | \
    grep -oP 'USER=\K\w+' | sort | uniq -c | sort -rn | tee -a "$OUTPUT"

# Dangerous commands
echo -e "\n=== Potentially Dangerous Commands ===" | tee -a "$OUTPUT"
grep -E "rm -rf|chmod 777|dd if=|mkfs\." "$LOG" | tee -a "$OUTPUT"
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
BACKUP_DIR="/var/backups/firewall"
DATE=$(date +%Y%m%d_%H%M)

mkdir -p "$BACKUP_DIR"
cd "$BACKUP_DIR"

# Initialize git if needed
[ -d .git ] || git init

# Backup iptables
echo "Backing up iptables..."
iptables-save > "iptables_$DATE.rules"
cp "iptables_$DATE.rules" iptables_current.rules

# Backup firewalld (if present)
if command -v firewall-cmd &>/dev/null; then
    echo "Backing up firewalld..."
    firewall-cmd --list-all-zones > "firewalld_$DATE.txt"
    cp "firewalld_$DATE.txt" firewalld_current.txt
fi

# Backup ufw (if present)
if command -v ufw &>/dev/null; then
    echo "Backing up ufw..."
    ufw status verbose > "ufw_$DATE.txt"
    cp "ufw_$DATE.txt" ufw_current.txt
fi

# Commit changes
git add -A
if git diff --cached --quiet; then
    echo "No firewall changes detected"
else
    git commit -m "Firewall backup $DATE"
    echo "Changes committed to git"
    git log --oneline -5
fi
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
WARN_DAYS=14

echo "=== Password Expiry Report ==="
echo ""

while IFS=: read -r user pass lastchg min max warn inactive expire rest; do
    # Skip system accounts
    uid=$(id -u "$user" 2>/dev/null)
    [ "$uid" -lt 1000 ] && continue
    
    # Skip locked/disabled accounts
    [[ "$pass" == "!" || "$pass" == "*" || "$pass" == "!!" ]] && continue
    
    if [ "$max" != "" ] && [ "$max" != "99999" ] && [ "$lastchg" != "" ]; then
        today=$(( $(date +%s) / 86400 ))
        expiry_day=$((lastchg + max))
        days_left=$((expiry_day - today))
        expiry_date=$(date -d "1970-01-01 +${expiry_day} days" +%Y-%m-%d)
        
        if [ $days_left -lt 0 ]; then
            echo "EXPIRED: $user (expired $((-days_left)) days ago)"
        elif [ $days_left -lt $WARN_DAYS ]; then
            echo "WARNING: $user expires in $days_left days ($expiry_date)"
        else
            echo "OK: $user expires in $days_left days ($expiry_date)"
        fi
    fi
done < /etc/shadow

# Check for accounts that never expire
echo -e "\n=== Accounts with No Expiry ==="
awk -F: '$5 == "" || $5 == 99999 {print $1}' /etc/shadow | while read user; do
    uid=$(id -u "$user" 2>/dev/null)
    [ "$uid" -ge 1000 ] && echo "$user"
done
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
ACTION="$1"
ENVIRONMENT="${2:-development}"
REGION="${AWS_REGION:-us-east-1}"

usage() {
    echo "Usage: $0 <list|start|stop> <environment>"
    echo "Environments: development, staging, production"
    exit 1
}

list_instances() {
    echo "=== EC2 Instances (Environment: $ENVIRONMENT) ==="
    aws ec2 describe-instances \
        --region "$REGION" \
        --filters "Name=tag:Environment,Values=$ENVIRONMENT" \
        --query 'Reservations[].Instances[].[InstanceId,State.Name,InstanceType,Tags[?Key==`Name`].Value|[0]]' \
        --output table
}

manage_instances() {
    local action="$1"
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
    [ "$confirm" != "y" ] && exit 0
    
    aws ec2 "${action}-instances" --region "$REGION" --instance-ids $instances
    echo "$action initiated. Waiting..."
    
    aws ec2 wait "instance-${action}ed" --region "$REGION" --instance-ids $instances
    echo "Complete!"
}

case "$ACTION" in
    list) list_instances ;;
    start) manage_instances "start" ;;
    stop) manage_instances "stop" ;;
    *) usage ;;
esac
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
LOCAL_DIR="$1"
S3_BUCKET="$2"
LOG_FILE="/var/log/s3sync_$(date +%F).log"

if [ -z "$LOCAL_DIR" ] || [ -z "$S3_BUCKET" ]; then
    echo "Usage: $0 <local_dir> <s3://bucket/path>"
    exit 1
fi

echo "=== S3 Sync Started: $(date) ===" | tee -a "$LOG_FILE"
echo "Source: $LOCAL_DIR" | tee -a "$LOG_FILE"
echo "Destination: $S3_BUCKET" | tee -a "$LOG_FILE"

# Pre-sync stats
local_files=$(find "$LOCAL_DIR" -type f | wc -l)
local_size=$(du -sh "$LOCAL_DIR" | cut -f1)
echo "Local: $local_files files, $local_size" | tee -a "$LOG_FILE"

# Perform sync with progress
aws s3 sync "$LOCAL_DIR" "$S3_BUCKET" \
    --delete \
    --exclude "*.tmp" \
    --exclude ".git/*" \
    2>&1 | tee -a "$LOG_FILE"

sync_status=$?

# Post-sync verification
if [ $sync_status -eq 0 ]; then
    s3_files=$(aws s3 ls "$S3_BUCKET" --recursive | wc -l)
    echo "S3: $s3_files files synced" | tee -a "$LOG_FILE"
    echo "Sync SUCCESSFUL" | tee -a "$LOG_FILE"
else
    echo "Sync FAILED with exit code $sync_status" | tee -a "$LOG_FILE"
fi

echo "=== S3 Sync Completed: $(date) ===" | tee -a "$LOG_FILE"
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
BUDGET_THRESHOLD=1000
ALERT_EMAIL="finance@company.com"

# Get current month dates
START_DATE=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d)
END_DATE=$(date +%Y-%m-%d)

echo "=== AWS Cost Report ==="
echo "Period: $START_DATE to $END_DATE"
echo ""

# Get cost data
cost_data=$(aws ce get-cost-and-usage \
    --time-period Start="$START_DATE",End="$END_DATE" \
    --granularity MONTHLY \
    --metrics UnblendedCost \
    --group-by Type=DIMENSION,Key=SERVICE \
    --output json)

# Parse and display
total_cost=$(echo "$cost_data" | jq -r '.ResultsByTime[0].Total.UnblendedCost.Amount // 0')
echo "Total Cost: \$$(printf '%.2f' $total_cost)"
echo ""

echo "=== Cost by Service ==="
echo "$cost_data" | jq -r '.ResultsByTime[0].Groups[] | "\(.Keys[0]): $\(.Metrics.UnblendedCost.Amount)"' | \
    sort -t'$' -k2 -rn | head -10

# Budget alert
if (( $(echo "$total_cost > $BUDGET_THRESHOLD" | bc -l) )); then
    message="AWS Cost Alert: Current spend \$$total_cost exceeds budget \$$BUDGET_THRESHOLD"
    echo "$message"
    echo "$message" | mail -s "AWS Budget Alert" "$ALERT_EMAIL"
fi
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
RETENTION_DAYS=7
REGION="${AWS_REGION:-us-east-1}"

echo "=== EBS Snapshot Automation ==="

# Find volumes tagged for backup
volumes=$(aws ec2 describe-volumes \
    --region "$REGION" \
    --filters "Name=tag:Backup,Values=true" \
    --query 'Volumes[].VolumeId' \
    --output text)

if [ -z "$volumes" ]; then
    echo "No volumes tagged for backup"
    exit 0
fi

# Create snapshots
for vol_id in $volumes; do
    name=$(aws ec2 describe-volumes \
        --region "$REGION" \
        --volume-ids "$vol_id" \
        --query 'Volumes[0].Tags[?Key==`Name`].Value' \
        --output text)
    
    echo "Creating snapshot for $vol_id ($name)..."
    
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
cutoff_date=$(date -d "-$RETENTION_DAYS days" +%Y-%m-%d)

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
NAMESPACE="Custom/System"
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)

push_metric() {
    local metric_name="$1"
    local value="$2"
    local unit="${3:-Count}"
    
    aws cloudwatch put-metric-data \
        --namespace "$NAMESPACE" \
        --metric-name "$metric_name" \
        --value "$value" \
        --unit "$unit" \
        --dimensions InstanceId="$INSTANCE_ID"
}

# Memory utilization
mem_total=$(grep MemTotal /proc/meminfo | awk '{print $2}')
mem_available=$(grep MemAvailable /proc/meminfo | awk '{print $2}')
mem_used_pct=$(echo "scale=2; (($mem_total - $mem_available) / $mem_total) * 100" | bc)
push_metric "MemoryUtilization" "$mem_used_pct" "Percent"

# Disk utilization
disk_used_pct=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
push_metric "DiskUtilization" "$disk_used_pct" "Percent"

# Open file handles
open_files=$(cat /proc/sys/fs/file-nr | awk '{print $1}')
push_metric "OpenFileHandles" "$open_files" "Count"

# Process count
process_count=$(ps aux | wc -l)
push_metric "ProcessCount" "$process_count" "Count"

# Network connections
tcp_connections=$(ss -s | grep 'estab' | awk '{print $4}' | tr -d ',')
push_metric "TCPConnections" "${tcp_connections:-0}" "Count"

echo "Metrics pushed to CloudWatch ($NAMESPACE)"
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
SUBNET="${1:-192.168.1}"
TIMEOUT=1
PARALLEL=50

echo "=== Scanning subnet $SUBNET.0/24 ==="

scan_host() {
    local ip="$1"
    if ping -c 1 -W $TIMEOUT "$ip" &>/dev/null; then
        hostname=$(getent hosts "$ip" 2>/dev/null | awk '{print $2}')
        echo "UP: $ip ${hostname:+($hostname)}"
    fi
}

export -f scan_host
export TIMEOUT

# Parallel scan
seq 1 254 | xargs -P $PARALLEL -I{} bash -c "scan_host $SUBNET.{}"

echo -e "\nScan complete!"
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
DOMAINS="api.company.com www.company.com db.company.com"
DNS_SERVERS="8.8.8.8 1.1.1.1 $(grep nameserver /etc/resolv.conf | head -1 | awk '{print $2}')"

echo "=== DNS Resolution Check ==="

for domain in $DOMAINS; do
    echo -e "\n--- $domain ---"
    prev_ip=""
    
    for dns in $DNS_SERVERS; do
        result=$(dig +short @"$dns" "$domain" 2>/dev/null | head -1)
        
        if [ -z "$result" ]; then
            echo "  $dns: FAILED (no response)"
        else
            if [ -n "$prev_ip" ] && [ "$result" != "$prev_ip" ]; then
                echo "  $dns: $result (WARNING: mismatch!)"
            else
                echo "  $dns: $result"
            fi
            prev_ip="$result"
        fi
    done
done

# Check propagation time
echo -e "\n=== DNS Propagation Test ==="
for domain in $DOMAINS; do
    start=$(date +%s%N)
    dig +short "$domain" &>/dev/null
    end=$(date +%s%N)
    time_ms=$(( (end - start) / 1000000 ))
    echo "$domain: ${time_ms}ms"
done
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
BACKENDS="10.0.1.10 10.0.1.11 10.0.1.12 10.0.1.13"
HEALTH_PATH="/health"
PORT=8080
TIMEOUT=5

echo "=== Load Balancer Backend Health Check ==="

healthy=0
unhealthy=0

for backend in $BACKENDS; do
    url="http://$backend:$PORT$HEALTH_PATH"
    
    response=$(curl -s -o /dev/null -w "%{http_code} %{time_total}" --connect-timeout $TIMEOUT "$url")
    http_code=$(echo "$response" | cut -d' ' -f1)
    response_time=$(echo "$response" | cut -d' ' -f2)
    
    if [ "$http_code" == "200" ]; then
        echo "✓ $backend - HTTP $http_code (${response_time}s)"
        ((healthy++))
    else
        echo "✗ $backend - HTTP $http_code (UNHEALTHY)"
        ((unhealthy++))
    fi
done

echo ""
echo "Summary: $healthy healthy, $unhealthy unhealthy"

# Alert if less than 50% healthy
total=$((healthy + unhealthy))
if [ $((healthy * 100 / total)) -lt 50 ]; then
    echo "CRITICAL: Less than 50% backends healthy!"
    exit 1
fi
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
INTERFACE="${1:-eth0}"
INTERVAL=1

get_bytes() {
    cat /proc/net/dev | grep "$INTERFACE" | awk '{print $2, $10}'
}

echo "=== Bandwidth Monitor: $INTERFACE ==="
echo "Press Ctrl+C to stop"
echo ""

read prev_rx prev_tx <<< $(get_bytes)

while true; do
    sleep $INTERVAL
    read curr_rx curr_tx <<< $(get_bytes)
    
    rx_rate=$(( (curr_rx - prev_rx) / INTERVAL ))
    tx_rate=$(( (curr_tx - prev_tx) / INTERVAL ))
    
    # Convert to human readable
    rx_human=$(numfmt --to=iec $rx_rate 2>/dev/null || echo "$rx_rate B")
    tx_human=$(numfmt --to=iec $tx_rate 2>/dev/null || echo "$tx_rate B")
    
    printf "\r[%s] RX: %s/s | TX: %s/s    " "$(date +%H:%M:%S)" "$rx_human" "$tx_human"
    
    prev_rx=$curr_rx
    prev_tx=$curr_tx
done
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
echo "=== TCP Connection Analysis ==="
echo "Generated: $(date)"
echo ""

# Connection states
echo "--- Connection States ---"
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn

# Connections per local port
echo -e "\n--- Top Local Ports ---"
ss -tan | awk 'NR>1 {split($4,a,":"); print a[length(a)]}' | sort | uniq -c | sort -rn | head -10

# Connections per remote IP
echo -e "\n--- Top Remote IPs ---"
ss -tan | awk 'NR>1 {split($5,a,":"); ip=""; for(i=1;i<length(a);i++) ip=ip""a[i]; print ip}' | \
    grep -v "^\*$" | sort | uniq -c | sort -rn | head -10

# TIME_WAIT analysis
timewait=$(ss -tan state time-wait | wc -l)
echo -e "\n--- TIME_WAIT Connections: $timewait ---"

# CLOSE_WAIT (potential resource leak)
closewait=$(ss -tan state close-wait | wc -l)
if [ "$closewait" -gt 10 ]; then
    echo "WARNING: $closewait CLOSE_WAIT connections (potential resource leak)"
    ss -tan state close-wait | head -10
fi
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
SERVERS="server1 server2 server3 server4 server5"
COMMAND="${1:-uptime}"
MAX_PARALLEL=5
TIMEOUT=30

echo "=== Parallel Execution ==="
echo "Command: $COMMAND"
echo "Servers: $SERVERS"
echo ""

execute_on_server() {
    local server="$1"
    local cmd="$2"
    local output
    
    output=$(timeout $TIMEOUT ssh -o ConnectTimeout=5 -o StrictHostKeyChecking=no "$server" "$cmd" 2>&1)
    exit_code=$?
    
    if [ $exit_code -eq 0 ]; then
        echo "[$server] SUCCESS:"
        echo "$output" | sed 's/^/  /'
    else
        echo "[$server] FAILED (exit: $exit_code):"
        echo "$output" | sed 's/^/  /'
    fi
}

# Run in parallel with limit
running=0
for server in $SERVERS; do
    execute_on_server "$server" "$COMMAND" &
    ((running++))
    
    if [ $running -ge $MAX_PARALLEL ]; then
        wait -n
        ((running--))
    fi
done

# Wait for remaining
wait
echo -e "\nAll tasks complete!"
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
FILE1="$1"
FILE2="$2"

if [ -z "$FILE1" ] || [ -z "$FILE2" ]; then
    echo "Usage: $0 <current_config> <new_config>"
    exit 1
fi

echo "=== Config Comparison ==="
echo "Current: $FILE1"
echo "New: $FILE2"
echo ""

# Show diff
if diff -q "$FILE1" "$FILE2" &>/dev/null; then
    echo "Files are identical"
    exit 0
fi

echo "--- Differences ---"
diff --color=auto -u "$FILE1" "$FILE2"

echo -e "\n--- Options ---"
echo "1. Keep current"
echo "2. Use new config"
echo "3. Merge interactively"
echo "4. View side-by-side"

read -p "Choice [1-4]: " choice

case $choice in
    1)
        echo "Keeping current config"
        ;;
    2)
        backup="${FILE1}.$(date +%Y%m%d_%H%M%S).bak"
        cp "$FILE1" "$backup"
        cp "$FILE2" "$FILE1"
        echo "Updated. Backup: $backup"
        ;;
    3)
        backup="${FILE1}.$(date +%Y%m%d_%H%M%S).bak"
        cp "$FILE1" "$backup"
        # Use vimdiff or sdiff for interactive merge
        if command -v vimdiff &>/dev/null; then
            vimdiff "$FILE1" "$FILE2"
        else
            sdiff -o "$FILE1.merged" "$FILE1" "$FILE2"
            mv "$FILE1.merged" "$FILE1"
        fi
        ;;
    4)
        sdiff "$FILE1" "$FILE2" | less
        ;;
esac
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

echo "=== Cron Job Manager ==="

# List all cron jobs for all users
list_all_crons() {
    echo "--- System Crontabs ---"
    for file in /etc/cron.d/*; do
        [ -f "$file" ] && echo "==> $file" && cat "$file" | grep -v '^#' | grep -v '^$'
    done
    
    echo -e "\n--- User Crontabs ---"
    for user in $(cut -d: -f1 /etc/passwd); do
        crontab=$(crontab -l -u "$user" 2>/dev/null)
        if [ -n "$crontab" ]; then
            echo "==> $user"
            echo "$crontab" | grep -v '^#'
        fi
    done
}

# Validate cron syntax
validate_cron() {
    local cron_line="$1"
    # Simple validation of 5 time fields
    if echo "$cron_line" | grep -qE '^[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+[0-9*,/-]+\s+'; then
        echo "Valid: $cron_line"
        return 0
    else
        echo "Invalid: $cron_line"
        return 1
    fi
}

# Find duplicate jobs
find_duplicates() {
    echo "--- Checking for Duplicate Jobs ---"
    for user in $(cut -d: -f1 /etc/passwd); do
        crontab -l -u "$user" 2>/dev/null | grep -v '^#' | grep -v '^$' | sort | uniq -d
    done
}

# Report jobs running soon
upcoming_jobs() {
    echo "--- Jobs Running in Next Hour ---"
    current_min=$(date +%M)
    current_hour=$(date +%H)
    
    for user in $(cut -d: -f1 /etc/passwd); do
        crontab -l -u "$user" 2>/dev/null | grep -v '^#' | grep -v '^$' | while read line; do
            min=$(echo "$line" | awk '{print $1}')
            hour=$(echo "$line" | awk '{print $2}')
            # Simplified check
            if [ "$hour" == "*" ] || [ "$hour" == "$current_hour" ]; then
                echo "$user: $line"
            fi
        done
    done
}

case "$1" in
    list) list_all_crons ;;
    validate) shift; validate_cron "$*" ;;
    duplicates) find_duplicates ;;
    upcoming) upcoming_jobs ;;
    *) echo "Usage: $0 <list|validate|duplicates|upcoming>" ;;
esac
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
SERVICE_NAME="myapp"
CONFIG_FILE="/etc/myapp/dependencies.conf"

# Sample config format:
# port:localhost:5432:PostgreSQL
# url:http://redis:6379/ping:Redis
# file:/var/run/myapp.pid
# env:DATABASE_URL

check_port() {
    local host="$1"
    local port="$2"
    local name="$3"
    
    if nc -z -w 5 "$host" "$port" 2>/dev/null; then
        echo "✓ $name ($host:$port)"
        return 0
    else
        echo "✗ $name ($host:$port) - NOT AVAILABLE"
        return 1
    fi
}

check_url() {
    local url="$1"
    local name="$2"
    
    if curl -s -f -o /dev/null --connect-timeout 5 "$url"; then
        echo "✓ $name ($url)"
        return 0
    else
        echo "✗ $name ($url) - NOT RESPONDING"
        return 1
    fi
}

check_file() {
    local file="$1"
    
    if [ -f "$file" ]; then
        echo "✓ File exists: $file"
        return 0
    else
        echo "✗ File missing: $file"
        return 1
    fi
}

check_env() {
    local var="$1"
    
    if [ -n "${!var}" ]; then
        echo "✓ Env set: $var"
        return 0
    else
        echo "✗ Env missing: $var"
        return 1
    fi
}

echo "=== Dependency Check: $SERVICE_NAME ==="
failed=0

while IFS=: read -r type arg1 arg2 arg3; do
    [ -z "$type" ] || [[ "$type" == \#* ]] && continue
    
    case "$type" in
        port) check_port "$arg1" "$arg2" "$arg3" || ((failed++)) ;;
        url) check_url "$arg1" "$arg2" || ((failed++)) ;;
        file) check_file "$arg1" || ((failed++)) ;;
        env) check_env "$arg1" || ((failed++)) ;;
    esac
done < "$CONFIG_FILE"

echo ""
if [ $failed -gt 0 ]; then
    echo "FAILED: $failed dependencies not met"
    exit 1
else
    echo "SUCCESS: All dependencies satisfied"
    exit 0
fi
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
SERVICE="nginx"
HEALTH_URL="http://localhost/health"
CHECK_INTERVAL=30
MAX_RETRIES=3
ESCALATION_EMAIL="oncall@company.com"
LOG="/var/log/${SERVICE}_healing.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"
}

check_health() {
    # Check systemd status
    if ! systemctl is-active --quiet "$SERVICE"; then
        return 1
    fi
    
    # Check HTTP health endpoint
    http_code=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 "$HEALTH_URL")
    [ "$http_code" == "200" ]
}

recover_service() {
    log "Attempting recovery..."
    
    # Graceful restart first
    systemctl restart "$SERVICE"
    sleep 5
    
    if check_health; then
        log "Recovery successful (graceful restart)"
        return 0
    fi
    
    # Force kill and start
    log "Graceful restart failed, forcing..."
    systemctl stop "$SERVICE"
    pkill -9 -f "$SERVICE" 2>/dev/null
    sleep 2
    systemctl start "$SERVICE"
    sleep 5
    
    if check_health; then
        log "Recovery successful (forced restart)"
        return 0
    fi
    
    return 1
}

escalate() {
    local message="$1"
    log "ESCALATION: $message"
    echo "$message" | mail -s "[$SERVICE] Self-Healing Failed" "$ESCALATION_EMAIL"
}

# Main monitoring loop
failure_count=0

log "Self-healing monitor started for $SERVICE"

while true; do
    if check_health; then
        [ $failure_count -gt 0 ] && log "Service recovered, resetting failure count"
        failure_count=0
    else
        ((failure_count++))
        log "Health check failed (count: $failure_count/$MAX_RETRIES)"
        
        if [ $failure_count -ge $MAX_RETRIES ]; then
            log "Max retries reached, attempting recovery..."
            
            if recover_service; then
                failure_count=0
            else
                escalate "Failed to recover $SERVICE after $MAX_RETRIES attempts. Manual intervention required."
                failure_count=0  # Reset to avoid spam
                sleep 300  # Wait 5 minutes before retrying
            fi
        fi
    fi
    
    sleep $CHECK_INTERVAL
done
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
