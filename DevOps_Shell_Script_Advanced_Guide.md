# DevOps Shell Scripting - Advanced Real-World Examples

This guide contains 50 medium to advanced shell script examples based on real-world DevOps scenarios. Each example includes practical use cases you'll encounter in production environments.

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

### 1. Log Rotation Script
**Problem:** Rotate application logs when they exceed 100MB, keep last 5 rotations.
**Logic:** Check file size, rename with timestamp, compress old logs, delete extras.
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

### 2. Parse Apache Access Logs
**Problem:** Extract unique URLs with 404 errors and their count.
**Logic:** Use `awk` to filter status code, extract URL, sort and count.
```bash
#!/bin/bash
LOG="/var/log/apache2/access.log"

echo "=== 404 Errors Summary ==="
awk '$9 == 404 {print $7}' "$LOG" | sort | uniq -c | sort -rn | head -20
```

### 3. Find Top 10 IPs from Logs
**Problem:** Identify top 10 IP addresses making requests (potential DDoS detection).
**Logic:** Extract first field (IP), count occurrences, sort descending.
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

### 4. Extract 5xx Errors from Nginx
**Problem:** Extract all 5xx server errors with timestamps for debugging.
**Logic:** Filter where status code starts with 5.
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

### 5. Real-time Log Alerting
**Problem:** Monitor log file and send alert when "CRITICAL" appears.
**Logic:** Use `tail -f` with pattern matching and trigger alert.
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

### 6. Compress and Archive Logs
**Problem:** Compress logs older than 1 day, archive to NFS share, delete after 30 days.
**Logic:** Find by mtime, compress, move, cleanup old.
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

### 7. Log File Size Monitor
**Problem:** Alert when any log file in directory exceeds threshold.
**Logic:** Loop through files, check size, alert if exceeded.
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

### 8. Multi-Server Health Check
**Problem:** Check health of multiple servers and report status.
**Logic:** Read server list, ping/curl each, aggregate results.
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

### 9. Process Memory Monitor
**Problem:** Find top 10 memory-consuming processes and alert if any exceeds threshold.
**Logic:** Parse `ps` output, sort by memory, compare threshold.
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

### 10. Zombie Process Killer
**Problem:** Find and report zombie processes, optionally kill parent.
**Logic:** Check process state, identify parent, take action.
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

### 11. High CPU Process Alert
**Problem:** Detect processes using more than 90% CPU for over 5 minutes.
**Logic:** Sample CPU usage, track duration, alert if sustained.
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

### 12. Disk I/O Monitor
**Problem:** Monitor disk I/O and alert on high utilization.
**Logic:** Use `iostat` to get metrics, parse and compare.
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

### 13. Network Connection Monitor
**Problem:** Count connections per state and alert on too many TIME_WAIT.
**Logic:** Parse `ss` or `netstat` output, group by state.
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

### 14. SSL Certificate Expiry Check
**Problem:** Check SSL certificates for multiple domains and alert before expiry.
**Logic:** Use `openssl` to get cert expiry, calculate days remaining.
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

### 15. Server Uptime Report
**Problem:** Generate weekly uptime report for fleet of servers.
**Logic:** SSH to servers, collect uptime, generate HTML report.
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

### 16. Incremental Backup Script
**Problem:** Perform incremental backup using rsync with rotation.
**Logic:** Use rsync with `--link-dest` for space-efficient incremental backups.
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

### 17. MySQL Database Backup with Retention
**Problem:** Backup all MySQL databases with compression and S3 upload.
**Logic:** Loop through databases, dump each, compress, upload, cleanup.
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

### 18. PostgreSQL Backup to S3
**Problem:** Hot backup PostgreSQL with point-in-time recovery support.
**Logic:** Use `pg_dump` with custom format for parallel restore capability.
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

### 19. Backup Verification Script
**Problem:** Verify backup integrity by test restore.
**Logic:** Restore to temp location, run integrity checks, cleanup.
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

### 20. Disaster Recovery Sync
**Problem:** Sync critical data to DR site with bandwidth limiting.
**Logic:** Use rsync with bandwidth limit, verify sync, send report.
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

### 21. Blue-Green Deployment Switch
**Problem:** Switch traffic between blue and green environments.
**Logic:** Update load balancer config, verify health, rollback if needed.
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

### 22. Rolling Restart Script
**Problem:** Restart services one by one ensuring availability.
**Logic:** Loop through instances, drain, restart, wait for healthy.
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

### 23. Git Hook Pre-commit Validator
**Problem:** Validate code before commit (syntax, secrets, lint).
**Logic:** Check staged files for issues, block commit if problems found.
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

### 24. Build Artifact Versioning
**Problem:** Generate semantic version from git and tag artifacts.
**Logic:** Parse git describe, increment version, tag build.
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

### 25. Environment Config Generator
**Problem:** Generate environment-specific config from template.
**Logic:** Use envsubst or sed to replace placeholders with env vars.
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

### 26. Docker Image Cleanup
**Problem:** Remove unused Docker images, containers, and volumes.
**Logic:** Identify dangling resources, calculate space, cleanup.
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

### 27. Kubernetes Pod Restart
**Problem:** Restart pods in a deployment gracefully.
**Logic:** Use rollout restart or scale down/up.
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

### 28. Helm Release Manager
**Problem:** Deploy/upgrade Helm releases with validation.
**Logic:** Diff changes, apply with atomic flag, verify.
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

### 29. Failed SSH Login Detector
**Problem:** Detect and report failed SSH login attempts (brute force detection).
**Logic:** Parse auth logs, count failures per IP, alert on threshold.
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

### 30. File Integrity Checker
**Problem:** Monitor critical files for unauthorized changes.
**Logic:** Generate checksums, compare with baseline, alert on changes.
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

### 31. Open Port Scanner
**Problem:** Scan servers for unexpected open ports.
**Logic:** Compare current open ports against allowed list.
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

### 32. User Audit Script
**Problem:** Audit user accounts for security compliance.
**Logic:** Check for users with shells, sudo access, password status.
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

### 33. Sudo Command Logger
**Problem:** Log all sudo commands with context for audit.
**Logic:** Parse sudo logs, format for analysis.
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

### 34. Firewall Rules Backup
**Problem:** Backup and version control firewall rules.
**Logic:** Export rules, diff with previous, commit to git.
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

### 35. Password Expiry Checker
**Problem:** Check for users with expired or soon-to-expire passwords.
**Logic:** Parse shadow file, calculate expiry dates.
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

### 36. AWS EC2 Instance Manager
**Problem:** Start/stop/list EC2 instances by tag.
**Logic:** Use AWS CLI to filter and manage instances.
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

### 37. S3 Bucket Sync with Logging
**Problem:** Sync local directory to S3 with detailed logging.
**Logic:** Use aws s3 sync with logging and error handling.
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

### 38. AWS Cost Alert Script
**Problem:** Get AWS cost breakdown and alert if over budget.
**Logic:** Query Cost Explorer API, compare with thresholds.
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

### 39. EBS Snapshot Automation
**Problem:** Create EBS snapshots with retention policy.
**Logic:** Tag volumes for backup, create snapshots, cleanup old.
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

### 40. CloudWatch Metrics Pusher
**Problem:** Push custom metrics to CloudWatch.
**Logic:** Collect system metrics, format for CloudWatch, push via API.
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

### 41. Multi-Host Ping Sweep
**Problem:** Quickly identify live hosts in a subnet.
**Logic:** Parallel ping with timeout, aggregate results.
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

### 42. DNS Resolution Checker
**Problem:** Verify DNS resolution for critical domains.
**Logic:** Query multiple DNS servers, compare results.
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

### 43. Load Balancer Health Probe
**Problem:** Check health of all backends behind a load balancer.
**Logic:** Query each backend directly, report status.
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

### 44. Bandwidth Usage Monitor
**Problem:** Monitor network bandwidth per interface.
**Logic:** Sample /proc/net/dev, calculate delta.
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

### 45. TCP Connection States Report
**Problem:** Generate detailed TCP connection statistics.
**Logic:** Parse ss/netstat output, categorize by state and port.
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

### 46. Parallel Task Executor
**Problem:** Run commands on multiple servers in parallel.
**Logic:** Use background jobs with job control.
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

### 47. Config File Diff & Merge
**Problem:** Compare config files and merge changes safely.
**Logic:** Diff files, interactive merge, backup original.
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

### 48. Cron Job Manager
**Problem:** List, validate, and manage cron jobs across users.
**Logic:** Parse crontabs, validate syntax, report issues.
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

### 49. Service Dependency Checker
**Problem:** Verify all dependencies before starting a service.
**Logic:** Check ports, files, remote services, environment vars.
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

### 50. Self-Healing Service Script
**Problem:** Monitor and automatically recover a failing service.
**Logic:** Check health, attempt recovery, escalate if repeated failures.
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
