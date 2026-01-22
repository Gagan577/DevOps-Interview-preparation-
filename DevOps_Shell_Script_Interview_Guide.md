# DevOps Shell Scripting Interview Guide

This guide contains 50 shell script examples ranging from basic to advanced levels. Each example includes the problem statement, the core logic/approach, and the solution code.

## Table of Contents

### Basics
1. [Hello World & Variables](#1-hello-world--variables)
2. [User Input](#2-user-input)
3. [Arithmetic Operations](#3-arithmetic-operations)
4. [If-Else Condition](#4-if-else-condition)
5. [File Existence Check](#5-file-existence-check)
6. [Directory Check & Create](#6-directory-check--create)
7. [Check File Permissions](#7-check-file-permissions)
8. [Logical Operators](#8-logical-operators)
9. [For Loop (Simple)](#9-for-loop-simple)
10. [While Loop](#10-while-loop)

### Control Flow & Functions
11. [Until Loop](#11-until-loop)
12. [Case Statement (Menu)](#12-case-statement-menu)
13. [Functions](#13-functions)
14. [Local vs Global Variables](#14-local-vs-global-variables)
15. [Reading File Line by Line](#15-reading-file-line-by-line)
16. [Count Lines in File](#16-count-lines-in-file)
17. [Find Files by Extension](#17-find-files-by-extension)
18. [Delete Old Log Files](#18-delete-old-log-files)
19. [Bulk Rename Files](#19-bulk-rename-files)
20. [String Length](#20-string-length)
21. [Extract Substring](#21-extract-substring)

### Text Processing (Sed/Awk/Grep)
22. [Grep Pattern Search](#22-grep-pattern-search)
23. [Sed Replace Text](#23-sed-replace-text)
24. [Awk Print Specific Column](#24-awk-print-specific-column)
25. [Remove Duplicate Lines](#25-remove-duplicate-lines)
26. [Count Word Frequency](#26-count-word-frequency)

### System Monitoring
27. [Check Disk Space](#27-check-disk-space)
28. [Check RAM Usage](#28-check-ram-usage)
29. [Check CPU Load](#29-check-cpu-load)
30. [Check Service Status](#30-check-service-status)
31. [Restart Service if Down](#31-restart-service-if-down)
32. [System Info Report](#32-system-info-report)
33. [Check Root Privileges](#33-check-root-privileges)
34. [Port Connectivity Check](#34-port-connectivity-check)

### DevOps Automation
35. [Website Availability (Curl)](#35-website-availability-curl)
36. [Parse Command Arguments](#36-parse-command-arguments)
37. [Parse Flags (getopts)](#37-parse-flags-getopts)
38. [Error Handling (Exit Codes)](#38-error-handling-exit-codes)
39. [Trap Signals (Cleanup)](#39-trap-signals-cleanup)
40. [Create User Account](#40-create-user-account)
41. [Backup Directory (Tar)](#41-backup-directory-tar)
42. [Remote Copy (SCP Wrapper)](#42-remote-copy-scp-wrapper)
43. [Monitor Log for Errors](#43-monitor-log-for-errors)
44. [Send Email Alert](#44-send-email-alert)
45. [JSON Parsing (jq)](#45-json-parsing-jq)
46. [Database Backup Mock](#46-database-backup-mock)
47. [Docker Status Check](#47-docker-status-check)
48. [Git Auto Pull](#48-git-auto-pull)
49. [Parallel Execution](#49-parallel-execution)
50. [Random Password Generator](#50-random-password-generator)

---

## Solutions

### 1. Hello World & Variables
**Problem:** Print "Hello" followed by the current logged-in username.
**Logic:** Use the `echo` command and the system environment variable `$USER`.
```bash
#!/bin/bash
echo "Hello, $USER!"
```

### 2. User Input
**Problem:** Ask the user for their name and greet them.
**Logic:** Use `read` to capture input into a variable.
```bash
#!/bin/bash
echo "Enter your name:"
read name
echo "Welcome, $name"
```

### 3. Arithmetic Operations
**Problem:** Calculate the sum of two numbers provided by the user.
**Logic:** Use `$(( ... ))` for arithmetic expansion.
```bash
#!/bin/bash
read -p "Enter num1: " a
read -p "Enter num2: " b
sum=$((a + b))
echo "Sum is: $sum"
```

### 4. If-Else Condition
**Problem:** Check if a number is positive, negative, or zero.
**Logic:** Use `if`, `elif`, `else` with comparison operators (`-gt`, `-lt`, `-eq`).
```bash
#!/bin/bash
read -p "Enter a number: " num
if [ $num -gt 0 ]; then
    echo "Positive"
elif [ $num -lt 0 ]; then
    echo "Negative"
else
    echo "Zero"
fi
```

### 5. File Existence Check
**Problem:** Check if a specific file exists.
**Logic:** Use the `-f` flag in a test condition.
```bash
#!/bin/bash
filename="test.txt"
if [ -f "$filename" ]; then
    echo "File exists."
else
    echo "File does not exist."
fi
```

### 6. Directory Check & Create
**Problem:** Check if a directory exists; if not, create it.
**Logic:** Use the `-d` flag to check and `mkdir` to create.
```bash
#!/bin/bash
dir="my_backup"
if [ ! -d "$dir" ]; then
    mkdir "$dir"
    echo "Directory created."
else
    echo "Directory exists."
fi
```

### 7. Check File Permissions
**Problem:** Check if a file is writable.
**Logic:** Use the `-w` flag.
```bash
#!/bin/bash
file="script.sh"
if [ -w "$file" ]; then
    echo "File is writable"
else
    echo "File is not writable"
fi
```

### 8. Logical Operators
**Problem:** Check if a number is between 10 and 20.
**Logic:** Use `-a` (AND) or `&&` inside conditions.
```bash
#!/bin/bash
read -p "Enter number: " num
if [ $num -ge 10 ] && [ $num -le 20 ]; then
    echo "Number is within range."
else
    echo "Out of range."
fi
```

### 9. For Loop (Simple)
**Problem:** Print numbers from 1 to 5.
**Logic:** Iterate through a sequence range `{1..5}`.
```bash
#!/bin/bash
for i in {1..5}; do
    echo "Number: $i"
done
```

### 10. While Loop
**Problem:** Print a countdown from 5 to 1.
**Logic:** Loop while the counter is greater than 0, decrement logic `((count--))`.
```bash
#!/bin/bash
count=5
while [ $count -gt 0 ]; do
    echo "$count"
    ((count--))
done
```

### 11. Until Loop
**Problem:** Wait until a file is created (simple polling).
**Logic:** `until` loops execute *until* the condition becomes true.
```bash
#!/bin/bash
until [ -f "start.lock" ]; do
    echo "Waiting for lock file..."
    sleep 1
done
echo "Lock file found!"
```

### 12. Case Statement (Menu)
**Problem:** Create a simple menu to start or stop a service.
**Logic:** Use `case` to handle different string matches.
```bash
#!/bin/bash
echo "1. Start"
echo "2. Stop"
read -p "Choice: " choice
case $choice in
    1) echo "Starting..." ;;
    2) echo "Stopping..." ;;
    *) echo "Invalid option" ;;
esac
```

### 13. Functions
**Problem:** Create a function to greet a user.
**Logic:** Define `func_name() {}` and call it. Arguments are accessed via `$1`.
```bash
#!/bin/bash
greet() {
    echo "Hello, $1"
}
greet "DevOps"
```

### 14. Local vs Global Variables
**Problem:** Demonstrate variable scope.
**Logic:** Use `local` keyword inside functions.
```bash
#!/bin/bash
var="Global"
my_func() {
    local var="Local"
    echo "Inside: $var"
}
my_func
echo "Outside: $var"
```

### 15. Reading File Line by Line
**Problem:** Read and print each line of `names.txt`.
**Logic:** Use a `while` loop with `read` command expanding input file.
```bash
#!/bin/bash
file="names.txt"
while IFS= read -r line; do
    echo "Line: $line"
done < "$file"
```

### 16. Count Lines in File
**Problem:** Count how many lines are in a file.
**Logic:** Use `wc -l`.
```bash
#!/bin/bash
file="data.log"
count=$(wc -l < "$file")
echo "Total lines: $count"
```

### 17. Find Files by Extension
**Problem:** Find all `.log` files in `/var/log`.
**Logic:** Use `find` command.
```bash
#!/bin/bash
find /var/log -name "*.log"
```

### 18. Delete Old Log Files
**Problem:** Delete logs older than 7 days.
**Logic:** Use `find` with `-mtime` and `-delete` (or `-exec rm`).
```bash
#!/bin/bash
find /var/log/app -name "*.log" -type f -mtime +7 -delete
```

### 19. Bulk Rename Files
**Problem:** Rename all `.txt` files to `.md`.
**Logic:** Loop through files and use `mv` with string substitution/trimming.
```bash
#!/bin/bash
for file in *.txt; do
    mv "$file" "${file%.txt}.md"
done
```

### 20. String Length
**Problem:** Find the length of a string.
**Logic:** Use `${#variable}`.
```bash
#!/bin/bash
str="DevOps"
echo "Length: ${#str}"
```

### 21. Extract Substring
**Problem:** Extract "World" from "Hello World".
**Logic:** Use `${string:start:length}`.
```bash
#!/bin/bash
str="Hello World"
echo "${str:6:5}"
```

### 22. Grep Pattern Search
**Problem:** Find lines containing "ERROR" in a log file.
**Logic:** Use `grep`.
```bash
#!/bin/bash
grep "ERROR" application.log
```

### 23. Sed Replace Text
**Problem:** Replace "localhost" with "127.0.0.1" in a config file.
**Logic:** Use `sed -i` for in-place editing.
```bash
#!/bin/bash
sed -i 's/localhost/127.0.0.1/g' config.conf
```

### 24. Awk Print Specific Column
**Problem:** Print only the second column of a CSV/Space separated file.
**Logic:** Use `awk '{print $2}'`.
```bash
#!/bin/bash
echo "ID Name Role" | awk '{print $2}'
```

### 25. Remove Duplicate Lines
**Problem:** Remove duplicate lines from a sorted file.
**Logic:** Use `uniq`.
```bash
#!/bin/bash
sort names.txt | uniq
```

### 26. Count Word Frequency
**Problem:** Count occurrences of each word in a file.
**Logic:** Combine `tr`, `sort`, and `uniq -c`.
```bash
#!/bin/bash
cat text.txt | tr ' ' '\n' | sort | uniq -c
```

### 27. Check Disk Space
**Problem:** Alert if disk usage is above 90%.
**Logic:** Parse `df -h` output using `awk` and compare.
```bash
#!/bin/bash
usage=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')
if [ $usage -gt 90 ]; then
    echo "Disk full!"
fi
```

### 28. Check RAM Usage
**Problem:** Display free memory.
**Logic:** Use `free -m`.
```bash
#!/bin/bash
free -m | grep Mem | awk '{print "Free Memory: " $4 " MB"}'
```

### 29. Check CPU Load
**Problem:** Print the 1-minute load average.
**Logic:** Use `uptime` or `/proc/loadavg`.
```bash
#!/bin/bash
uptime | awk -F'load average:' '{ print $2 }' | cut -d, -f1
```

### 30. Check Service Status
**Problem:** Check if Nginx is running.
**Logic:** Use `systemctl is-active`.
```bash
#!/bin/bash
if systemctl is-active --quiet nginx; then
    echo "Nginx is running"
else
    echo "Nginx is NOT running"
fi
```

### 31. Restart Service if Down
**Problem:** Restart a service if it's not active.
**Logic:** Combine check and `systemctl restart`.
```bash
#!/bin/bash
service="nginx"
if ! systemctl is-active --quiet "$service"; then
    echo "$service down, restarting..."
    sudo systemctl restart "$service"
fi
```

### 32. System Info Report
**Problem:** Generate a file with date, hostname, and uptime.
**Logic:** Redirect `>` standard output to a file.
```bash
#!/bin/bash
echo "Date: $(date)" > info.txt
echo "Hostname: $(hostname)" >> info.txt
echo "Uptime: $(uptime)" >> info.txt
```

### 33. Check Root Privileges
**Problem:** Ensure the script is run as sudo/root.
**Logic:** Check valid user ID (`$EUID` or `id -u`). Root is 0.
```bash
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
    echo "Please run as root"
    exit 1
fi
```

### 34. Port Connectivity Check
**Problem:** Check if a remote port is open (e.g., Database port).
**Logic:** Use `nc` (netcat) or `/dev/tcp`.
```bash
#!/bin/bash
nc -z -v -w5 google.com 80
```

### 35. Website Availability (Curl)
**Problem:** Check if a website returns 200 OK.
**Logic:** Use `curl` with write-out variable.
```bash
#!/bin/bash
url="http://example.com"
code=$(curl -s -o /dev/null -w "%{http_code}" "$url")
if [ "$code" -eq 200 ]; then
    echo "Site is Up"
else
    echo "Site is Down ($code)"
fi
```

### 36. Parse Command Arguments
**Problem:** Accept filename as a command Line argument.
**Logic:** Check if `$1` is empty.
```bash
#!/bin/bash
if [ -z "$1" ]; then
    echo "Usage: $0 filename"
    exit 1
fi
echo "Processing $1..."
```

### 37. Parse Flags (getopts)
**Problem:** Handle flags like `-u username -p password`.
**Logic:** Use `getopts` loop.
```bash
#!/bin/bash
while getopts "u:p:" opt; do
  case $opt in
    u) user="$OPTARG" ;;
    p) pass="$OPTARG" ;;
    *) echo "Invalid flag"; exit 1 ;;
  esac
done
echo "User: $user"
```

### 38. Error Handling (Exit Codes)
**Problem:** Exit script if a command fails.
**Logic:** Check `$?` (exit status of last command) or use `set -e`.
```bash
#!/bin/bash
cp source.txt dest.txt
if [ $? -ne 0 ]; then
    echo "Copy failed"
    exit 1
fi
```

### 39. Trap Signals (Cleanup)
**Problem:** Clean up temporary files if user presses Ctrl+C.
**Logic:** Use `trap` command.
```bash
#!/bin/bash
temp_file="temp.dat"
touch "$temp_file"
trap "rm -f $temp_file; echo ' Cleaned up'; exit" SIGINT
echo "Running... (Press Ctrl+C to stop)"
sleep 10
```

### 40. Create User Account
**Problem:** Create a user if it doesn't exist.
**Logic:** Use `id` to check and `useradd` to create.
```bash
#!/bin/bash
user="devops_user"
if id "$user" &>/dev/null; then
    echo "User exists"
else
    sudo useradd "$user"
    echo "User created"
fi
```

### 41. Backup Directory (Tar)
**Problem:** Compress a folder into a tarball with a timestamp.
**Logic:** Use `tar czf` and `date`.
```bash
#!/bin/bash
src="/var/www/html"
dest="/backups/site-$(date +%F).tar.gz"
tar -czf "$dest" "$src"
echo "Backup saved to $dest"
```

### 42. Remote Copy (SCP Wrapper)
**Problem:** Copy a file to a remote server.
**Logic:** Use `scp`.
```bash
#!/bin/bash
scp myfile.txt user@remote-server:/home/user/
```

### 43. Monitor Log for Errors
**Problem:** Real-time monitoring of logs for the word "Error".
**Logic:** Use `tail -f` piped to `grep`.
```bash
#!/bin/bash
tail -f /var/log/syslog | grep --line-buffered "Error"
```

### 44. Send Email Alert
**Problem:** Send an email if a condition is met.
**Logic:** Use `mail` command (requires mail utils).
```bash
#!/bin/bash
echo "Disk space low" | mail -s "Alert" admin@example.com
```

### 45. JSON Parsing (jq)
**Problem:** Extract a value from a JSON file.
**Logic:** Use `jq` tool (standard for JSON in shell).
```bash
#!/bin/bash
# { "name": "MyApp", "version": "1.0" }
version=$(cat config.json | jq -r '.version')
echo "Version is $version"
```

### 46. Database Backup Mock
**Problem:** Simulate a DB backup command.
**Logic:** Execute dump command and check success.
```bash
#!/bin/bash
db_name="mydb"
# mysqldump -u root -p $db_name > $db_name.sql
echo "Backing up $db_name..."
```

### 47. Docker Status Check
**Problem:** Check if a specific container is running.
**Logic:** `docker ps` with grep.
```bash
#!/bin/bash
container="my-app"
if docker ps | grep -q "$container"; then
    echo "Container running"
else
    echo "Container stopped"
fi
```

### 48. Git Auto Pull
**Problem:** Pull changes from a git repo.
**Logic:** Navigate to dir and run `git pull`.
```bash
#!/bin/bash
cd /opt/project
git pull origin main
```

### 49. Parallel Execution
**Problem:** Run two background tasks and wait for them.
**Logic:** Use `&` to background and `wait` to pause.
```bash
#!/bin/bash
command1 &
pid1=$!
command2 &
pid2=$!
wait $pid1 $pid2
echo "Both tasks done"
```

### 50. Random Password Generator
**Problem:** Generate a random 12-character alphanumeric string.
**Logic:** Read from `/dev/urandom` and filter with `tr`.
```bash
#!/bin/bash
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 12 | head -n 1
```
