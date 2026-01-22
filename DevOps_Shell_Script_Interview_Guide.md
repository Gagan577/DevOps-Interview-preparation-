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
#!/bin/bash                 # Shebang - tells system to use bash interpreter
echo "Hello, $USER!"        # Print text with $USER variable (auto-set by system)
```

**Example Output:**
```
$ ./script.sh
Hello, john!
```

### 2. User Input
**Problem:** Ask the user for their name and greet them.
**Logic:** Use `read` to capture input into a variable.
```bash
#!/bin/bash                 # Shebang line
echo "Enter your name:"     # Print prompt message to user
read name                   # Wait for input, store in variable 'name'
echo "Welcome, $name"       # Print greeting using the captured variable
```

**Example Output:**
```
$ ./script.sh
Enter your name:
Alice
Welcome, Alice
```

### 3. Arithmetic Operations
**Problem:** Calculate the sum of two numbers provided by the user.
**Logic:** Use `$(( ... ))` for arithmetic expansion.
```bash
#!/bin/bash                  # Shebang line
read -p "Enter num1: " a     # -p shows prompt inline, stores input in 'a'
read -p "Enter num2: " b     # Same for second number in 'b'
sum=$((a + b))               # $((...)) performs arithmetic, result stored in 'sum'
echo "Sum is: $sum"          # Display the calculated sum
```

**Example Output:**
```
$ ./script.sh
Enter num1: 15
Enter num2: 25
Sum is: 40
```

**How Arithmetic Works:**
- `$((5 + 3))` → 8 (addition)
- `$((10 - 4))` → 6 (subtraction)
- `$((3 * 4))` → 12 (multiplication)
- `$((20 / 4))` → 5 (division)
- `$((10 % 3))` → 1 (modulo/remainder)

### 4. If-Else Condition
**Problem:** Check if a number is positive, negative, or zero.
**Logic:** Use `if`, `elif`, `else` with comparison operators (`-gt`, `-lt`, `-eq`).
```bash
#!/bin/bash                      # Shebang line
read -p "Enter a number: " num   # Prompt user, store in 'num'
if [ $num -gt 0 ]; then          # -gt means "greater than" (num > 0)
    echo "Positive"              # Executes if condition is true
elif [ $num -lt 0 ]; then        # -lt means "less than" (num < 0)
    echo "Negative"              # Executes if first false, this true
else                             # Catches all other cases (num = 0)
    echo "Zero"                  # Default when no conditions match
fi                               # 'fi' closes the if block (if backwards)
```

**Example Output:**
```
$ ./script.sh
Enter a number: 5
Positive

$ ./script.sh
Enter a number: -3
Negative
```

**Comparison Operators:**
| Operator | Meaning | Example |
|----------|---------|--------|
| `-eq` | Equal | `[ 5 -eq 5 ]` → true |
| `-ne` | Not equal | `[ 5 -ne 3 ]` → true |
| `-gt` | Greater than | `[ 5 -gt 3 ]` → true |
| `-lt` | Less than | `[ 3 -lt 5 ]` → true |
| `-ge` | Greater or equal | `[ 5 -ge 5 ]` → true |
| `-le` | Less or equal | `[ 3 -le 5 ]` → true |

### 5. File Existence Check
**Problem:** Check if a specific file exists.
**Logic:** Use the `-f` flag in a test condition.
```bash
#!/bin/bash                        # Shebang line
filename="test.txt"                # Store filename in variable
if [ -f "$filename" ]; then        # -f checks if file exists AND is regular file
    echo "File exists."            # True branch - file found
else                               # False branch - file not found
    echo "File does not exist."   
fi                                 # Close if block
```

**Example Output:**
```
$ touch test.txt          # Create the file first
$ ./script.sh
File exists.

$ rm test.txt             # Delete the file
$ ./script.sh
File does not exist.
```

**File Test Operators:**
| Flag | Checks if... |
|------|-------------|
| `-f` | File exists and is regular file |
| `-d` | Directory exists |
| `-e` | File/directory exists (any type) |
| `-r` | File is readable |
| `-w` | File is writable |
| `-x` | File is executable |
| `-s` | File exists and not empty |

### 6. Directory Check & Create
**Problem:** Check if a directory exists; if not, create it.
**Logic:** Use the `-d` flag to check and `mkdir` to create.
```bash
#!/bin/bash                    # Shebang line
dir="my_backup"                # Directory name stored in variable
if [ ! -d "$dir" ]; then       # ! negates condition: "if dir does NOT exist"
    mkdir "$dir"               # Create the directory
    echo "Directory created."  # Confirm creation
else                           # Directory already exists
    echo "Directory exists."   # Inform user
fi                             # Close if block
```

**Example Output:**
```
$ ./script.sh
Directory created.

$ ./script.sh          # Run again
Directory exists.

$ ls -la
drwxr-xr-x  2 user user 4096 Jan 22 10:00 my_backup
```

**Logic Flow:**
```
[ ! -d "my_backup" ]  →  Directory doesn't exist?  →  YES → mkdir
                                                    →  NO  → skip to else
```

### 7. Check File Permissions
**Problem:** Check if a file is writable.
**Logic:** Use the `-w` flag.
```bash
#!/bin/bash                    # Shebang line
file="script.sh"               # Target file to check
if [ -w "$file" ]; then        # -w tests if current user can write to file
    echo "File is writable"    # User has write permission
else                           # No write permission
    echo "File is not writable"
fi                             # Close if block
```

**Example Output:**
```
$ ls -l script.sh
-rw-r--r-- 1 user user 100 Jan 22 10:00 script.sh
$ ./check.sh
File is writable

$ chmod 444 script.sh      # Remove write permission
$ ./check.sh
File is not writable
```

**Permission Flags:**
- `-r` → Readable?
- `-w` → Writable?
- `-x` → Executable?

### 8. Logical Operators
**Problem:** Check if a number is between 10 and 20.
**Logic:** Use `-a` (AND) or `&&` inside conditions.
```bash
#!/bin/bash                              # Shebang line
read -p "Enter number: " num             # Get number from user
if [ $num -ge 10 ] && [ $num -le 20 ]; then   # -ge: >=10 AND -le: <=20
    echo "Number is within range."       # Both conditions true
else                                     # One or both conditions false
    echo "Out of range."
fi                                       # Close if block
```

**Example Output:**
```
$ ./script.sh
Enter number: 15
Number is within range.

$ ./script.sh
Enter number: 25
Out of range.
```

**Logical Operators:**
| Operator | Meaning | Example |
|----------|---------|--------|
| `&&` | AND (both true) | `[ 5 -gt 3 ] && [ 5 -lt 10 ]` |
| `\|\|` | OR (either true) | `[ 5 -lt 3 ] \|\| [ 5 -gt 3 ]` |
| `!` | NOT (negate) | `[ ! -f file ]` |

**Logic Flow:**
```
Input: 15
[ 15 -ge 10 ] → TRUE
[ 15 -le 20 ] → TRUE
TRUE && TRUE → TRUE → "within range"

Input: 25
[ 25 -ge 10 ] → TRUE
[ 25 -le 20 ] → FALSE
TRUE && FALSE → FALSE → "out of range"
```

### 9. For Loop (Simple)
**Problem:** Print numbers from 1 to 5.
**Logic:** Iterate through a sequence range `{1..5}`.
```bash
#!/bin/bash               # Shebang line
for i in {1..5}; do       # Loop: i takes values 1,2,3,4,5 one by one
    echo "Number: $i"     # Print current value of i
done                      # End of loop, go back to 'for' with next value
```

**Example Output:**
```
$ ./script.sh
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

**Loop Variations:**
```bash
# Range with step
for i in {0..10..2}; do    # 0, 2, 4, 6, 8, 10 (step by 2)

# Loop through list
for fruit in apple banana cherry; do
    echo "$fruit"
done

# Loop through files
for file in *.txt; do
    echo "Found: $file"
done

# C-style loop
for ((i=1; i<=5; i++)); do
    echo "$i"
done
```

### 10. While Loop
**Problem:** Print a countdown from 5 to 1.
**Logic:** Loop while the counter is greater than 0, decrement logic `((count--))`. 
```bash
#!/bin/bash                 # Shebang line
count=5                     # Initialize counter variable to 5
while [ $count -gt 0 ]; do  # Continue loop while count > 0
    echo "$count"           # Print current count value
    ((count--))             # Decrement count by 1 (count = count - 1)
done                        # End loop, re-check condition
```

**Example Output:**
```
$ ./script.sh
5
4
3
2
1
```

**Execution Flow:**
```
Iteration 1: count=5 → 5>0? YES → print 5 → count=4
Iteration 2: count=4 → 4>0? YES → print 4 → count=3
Iteration 3: count=3 → 3>0? YES → print 3 → count=2
Iteration 4: count=2 → 2>0? YES → print 2 → count=1
Iteration 5: count=1 → 1>0? YES → print 1 → count=0
Iteration 6: count=0 → 0>0? NO  → EXIT LOOP
```

**Increment/Decrement:**
- `((count++))` → Add 1
- `((count--))` → Subtract 1
- `((count+=5))` → Add 5

### 11. Until Loop
**Problem:** Wait until a file is created (simple polling).
**Logic:** `until` loops execute *until* the condition becomes true.
```bash
#!/bin/bash                          # Shebang line
until [ -f "start.lock" ]; do        # Keep looping UNTIL file exists
    echo "Waiting for lock file..."  # Inform user we're waiting
    sleep 1                          # Pause 1 second before re-checking
done                                 # Exit when file is found
echo "Lock file found!"              # Executes after loop exits
```

**Example Output:**
```
$ ./script.sh &              # Run in background
Waiting for lock file...
Waiting for lock file...
Waiting for lock file...

$ touch start.lock           # Create the file in another terminal
Lock file found!
```

**While vs Until:**
```
while [ condition ]   → Loop WHILE true, stop when false
until [ condition ]   → Loop UNTIL true, stop when true

They are opposites:
while [ ! -f file ]  =  until [ -f file ]
```

### 12. Case Statement (Menu)
**Problem:** Create a simple menu to start or stop a service.
**Logic:** Use `case` to handle different string matches.
```bash
#!/bin/bash                  # Shebang line
echo "1. Start"              # Display menu option 1
echo "2. Stop"               # Display menu option 2
read -p "Choice: " choice    # Get user's selection
case $choice in              # Start case block - match $choice against patterns
    1) echo "Starting..." ;; # If choice=1, print this. ;; ends the case
    2) echo "Stopping..." ;; # If choice=2, print this
    *) echo "Invalid option" ;; # * matches anything else (default)
esac                         # End case block ('case' backwards)
```

**Example Output:**
```
$ ./script.sh
1. Start
2. Stop
Choice: 1
Starting...

$ ./script.sh
1. Start
2. Stop
Choice: 5
Invalid option
```

**Case Pattern Matching:**
```bash
case $input in
    yes|y|Y)     echo "Confirmed" ;;     # Multiple patterns with |
    no|n|N)      echo "Cancelled" ;;     
    [0-9])       echo "Single digit" ;;  # Character ranges
    [a-z]*)      echo "Starts lowercase";; # Wildcards
    *)           echo "Unknown" ;;       # Default catch-all
esac
```

### 13. Functions
**Problem:** Create a function to greet a user.
**Logic:** Define `func_name() {}` and call it. Arguments are accessed via `$1`.
```bash
#!/bin/bash                  # Shebang line
greet() {                    # Define function named 'greet'
    echo "Hello, $1"         # $1 = first argument passed to function
}                            # End function definition
greet "DevOps"               # Call function with argument "DevOps"
```

**Example Output:**
```
$ ./script.sh
Hello, DevOps
```

**Function Arguments:**
```bash
my_function() {
    echo "Arg 1: $1"        # First argument
    echo "Arg 2: $2"        # Second argument
    echo "All args: $@"     # All arguments as separate words
    echo "Arg count: $#"    # Number of arguments
}

my_function apple banana cherry
# Output:
# Arg 1: apple
# Arg 2: banana
# All args: apple banana cherry
# Arg count: 3
```

**Return Values:**
```bash
add() {
    local result=$(($1 + $2))   # Calculate sum
    echo $result                 # Output result (capture with $())
}

sum=$(add 5 3)                   # Capture function output
echo "Sum is: $sum"              # Sum is: 8
```

### 14. Local vs Global Variables
**Problem:** Demonstrate variable scope.
**Logic:** Use `local` keyword inside functions.
```bash
#!/bin/bash                  # Shebang line
var="Global"                 # Global variable - accessible everywhere
my_func() {                  # Define function
    local var="Local"        # Local variable - only exists inside function
    echo "Inside: $var"      # Uses local var: prints "Local"
}                            # Local var destroyed when function ends
my_func                      # Call the function
echo "Outside: $var"         # Uses global var: prints "Global"
```

**Example Output:**
```
$ ./script.sh
Inside: Local
Outside: Global
```

**Scope Visualization:**
```
+---------------------------+
| GLOBAL SCOPE              |
| var = "Global"            |
|                           |
|  +---------------------+  |
|  | FUNCTION SCOPE      |  |
|  | local var = "Local" |  |
|  | (shadows global)    |  |
|  +---------------------+  |
|                           |
| var still = "Global"      |
+---------------------------+
```

**Without `local`:**
```bash
var="Global"
my_func() {
    var="Changed"           # Modifies global variable!
}
my_func
echo "$var"                  # Prints: Changed
```

### 15. Reading File Line by Line
**Problem:** Read and print each line of `names.txt`.
**Logic:** Use a `while` loop with `read` command expanding input file.
```bash
#!/bin/bash                      # Shebang line
file="names.txt"                 # File to read
while IFS= read -r line; do      # Read one line at a time into $line
    echo "Line: $line"           # Process/print current line
done < "$file"                   # < redirects file content into loop
```

**Example:**
```
# names.txt content:
Alice
Bob
Charlie

$ ./script.sh
Line: Alice
Line: Bob
Line: Charlie
```

**Understanding the Syntax:**
```
IFS=        → Internal Field Separator = empty (preserves leading spaces)
read -r     → -r prevents backslash from acting as escape character
line        → Variable name to store each line
done < file → Feed file into the loop's stdin
```

**Common Variations:**
```bash
# Read with line numbers
count=1
while IFS= read -r line; do
    echo "$count: $line"
    ((count++))
done < file.txt

# Process CSV (comma-separated)
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age"
done < data.csv
```

### 16. Count Lines in File
**Problem:** Count how many lines are in a file.
**Logic:** Use `wc -l`.
```bash
#!/bin/bash                    # Shebang line
file="data.log"                # Target file
count=$(wc -l < "$file")       # wc -l counts lines, < feeds file to stdin
echo "Total lines: $count"     # Display the count
```

**Example:**
```
# data.log has 150 lines
$ ./script.sh
Total lines: 150
```

**Why `< "$file"` instead of `wc -l $file`?**
```bash
# With filename argument - includes filename in output:
$ wc -l data.log
150 data.log

# With input redirection - just the number:
$ wc -l < data.log
150
```

**Other `wc` Options:**
```bash
wc -l file    # Line count
wc -w file    # Word count  
wc -c file    # Byte count
wc -m file    # Character count

# Count multiple things
$ wc -lwc file.txt
  100   500  3000 file.txt
#lines words bytes
```

### 17. Find Files by Extension
**Problem:** Find all `.log` files in `/var/log`.
**Logic:** Use `find` command.
```bash
#!/bin/bash                        # Shebang line
find /var/log -name "*.log"        # Search recursively for files matching *.log
# find = command
# /var/log = starting directory
# -name "*.log" = match files ending with .log
```

**Example Output:**
```
$ ./script.sh
/var/log/syslog.log
/var/log/auth.log
/var/log/nginx/access.log
/var/log/nginx/error.log
```

**Common `find` Options:**
```bash
# Find by type
find /path -type f           # Files only
find /path -type d           # Directories only

# Find by size
find /path -size +100M       # Files larger than 100MB
find /path -size -1k         # Files smaller than 1KB

# Find by modification time
find /path -mtime -7         # Modified in last 7 days
find /path -mtime +30        # Modified more than 30 days ago

# Combine conditions
find /var/log -name "*.log" -type f -size +1M

# Execute command on results
find /path -name "*.tmp" -exec rm {} \;
```

### 18. Delete Old Log Files
**Problem:** Delete logs older than 7 days.
**Logic:** Use `find` with `-mtime` and `-delete` (or `-exec rm`).
```bash
#!/bin/bash                                           # Shebang line
find /var/log/app -name "*.log" -type f -mtime +7 -delete
# find           = search command
# /var/log/app   = directory to search in
# -name "*.log"  = match .log files
# -type f        = regular files only (not directories)
# -mtime +7      = modified MORE than 7 days ago
# -delete        = remove matching files
```

**Example:**
```
# Before: Files in /var/log/app/
app.log      (2 days old)   → KEPT
old.log      (10 days old)  → DELETED
archive.log  (30 days old)  → DELETED

$ ./script.sh
# After: only app.log remains
```

**Understanding `-mtime`:**
```
-mtime +7   → More than 7 days ago (older)
-mtime -7   → Less than 7 days ago (newer)
-mtime 7    → Exactly 7 days ago

Time options:
-mtime  → Modification time (days)
-atime  → Access time (days)
-ctime  → Change time (days)
-mmin   → Modification time (minutes)
```

**Safer Alternative (preview first):**
```bash
# List files that WOULD be deleted
find /var/log/app -name "*.log" -type f -mtime +7 -print

# Then delete
find /var/log/app -name "*.log" -type f -mtime +7 -delete
```

### 19. Bulk Rename Files
**Problem:** Rename all `.txt` files to `.md`.
**Logic:** Loop through files and use `mv` with string substitution/trimming.
```bash
#!/bin/bash                      # Shebang line
for file in *.txt; do            # Loop through all .txt files in current dir
    mv "$file" "${file%.txt}.md" # Rename: remove .txt suffix, add .md
done                             # End loop
```

**Example:**
```
# Before:
readme.txt
notes.txt
data.txt

$ ./script.sh

# After:
readme.md
notes.md
data.md
```

**String Manipulation Explained:**
```bash
file="document.txt"

${file%.txt}      → "document"     # Remove .txt from END
${file%.txt}.md   → "document.md"  # Remove .txt, add .md

# Pattern: ${variable%pattern}
# Removes shortest match of pattern from END
```

**More String Operations:**
```bash
filename="backup.tar.gz"

${filename%.*}     → "backup.tar"    # Remove last extension
${filename%%.*}    → "backup"        # Remove all extensions
${filename#*.}     → "tar.gz"        # Remove up to first dot
${filename##*.}    → "gz"            # Get only last extension

# Summary:
# % = from end (short)   %% = from end (greedy)
# # = from start (short) ## = from start (greedy)
```

### 20. String Length
**Problem:** Find the length of a string.
**Logic:** Use `${#variable}`.
```bash
#!/bin/bash                # Shebang line
str="DevOps"               # String to measure
echo "Length: ${#str}"     # ${#var} returns character count
```

**Example Output:**
```
$ ./script.sh
Length: 6
```

**How It Works:**
```
str = "DevOps"
       D e v O p s
       1 2 3 4 5 6

${#str} = 6
```

**Practical Uses:**
```bash
# Validate input length
password="secret123"
if [ ${#password} -lt 8 ]; then
    echo "Password too short!"
fi

# Check if string is empty
if [ ${#str} -eq 0 ]; then
    echo "String is empty"
fi

# Alternative empty check
if [ -z "$str" ]; then    # -z = zero length
    echo "Empty"
fi
```

### 21. Extract Substring
**Problem:** Extract "World" from "Hello World".
**Logic:** Use `${string:start:length}`.
```bash
#!/bin/bash                    # Shebang line
str="Hello World"              # Source string
echo "${str:6:5}"              # Extract 5 chars starting at position 6
# Position:  0-5 = "Hello " (positions 0,1,2,3,4,5)
# Position:  6-10 = "World" (positions 6,7,8,9,10)
```

**Example Output:**
```
$ ./script.sh
World
```

**Position Visualization:**
```
H  e  l  l  o     W  o  r  l  d
0  1  2  3  4  5  6  7  8  9  10

${str:6:5} = Start at 6, take 5 characters = "World"
```

**Substring Variations:**
```bash
str="Hello World"

${str:0:5}      → "Hello"      # First 5 characters
${str:6}        → "World"      # From position 6 to end
${str: -5}      → "World"      # Last 5 characters (note space!)
${str: -5:3}    → "Wor"        # 3 chars starting 5 from end
```

**Negative Index (from end):**
```bash
str="filename.tar.gz"
${str: -6}      → "tar.gz"     # Last 6 chars
${str: -6:3}    → "tar"        # 3 chars, starting 6 from end
```

### 22. Grep Pattern Search
**Problem:** Find lines containing "ERROR" in a log file.
**Logic:** Use `grep`.
```bash
#!/bin/bash                       # Shebang line
grep "ERROR" application.log      # Search for "ERROR" in file, print matching lines
# grep = Global Regular Expression Print
# "ERROR" = pattern to search for
# application.log = file to search in
```

**Example:**
```
# application.log content:
2024-01-22 10:00:00 INFO Starting app
2024-01-22 10:00:01 ERROR Database connection failed
2024-01-22 10:00:02 INFO Retrying...
2024-01-22 10:00:03 ERROR Timeout occurred

$ ./script.sh
2024-01-22 10:00:01 ERROR Database connection failed
2024-01-22 10:00:03 ERROR Timeout occurred
```

**Common Grep Options:**
```bash
grep -i "error" file      # Case-insensitive (matches ERROR, Error, error)
grep -n "ERROR" file      # Show line numbers
grep -c "ERROR" file      # Count matching lines
grep -v "ERROR" file      # Invert: show lines WITHOUT "ERROR"
grep -r "ERROR" /path     # Recursive search in directory
grep -l "ERROR" *.log     # List filenames only
grep -A 2 "ERROR" file    # Show 2 lines After match
grep -B 2 "ERROR" file    # Show 2 lines Before match
grep -E "ERR|WARN" file   # Extended regex (OR pattern)
```

### 23. Sed Replace Text
**Problem:** Replace "localhost" with "127.0.0.1" in a config file.
**Logic:** Use `sed -i` for in-place editing.
```bash
#!/bin/bash                              # Shebang line
sed -i 's/localhost/127.0.0.1/g' config.conf
# sed = Stream EDitor
# -i = in-place (modify file directly)
# 's/old/new/g' = substitute old with new, globally (all occurrences)
# config.conf = target file
```

**Example:**
```
# config.conf BEFORE:
host=localhost
db_host=localhost
redis=localhost:6379

$ ./script.sh

# config.conf AFTER:
host=127.0.0.1
db_host=127.0.0.1
redis=127.0.0.1:6379
```

**Sed Syntax Breakdown:**
```
s/pattern/replacement/flags
│ │       │           │
│ │       │           └─ g = global (all matches)
│ │       └─ replacement text
│ └─ pattern to find
└─ s = substitute command
```

**Common Sed Operations:**
```bash
# Replace first occurrence only (no g flag)
sed 's/old/new/' file

# Delete lines containing pattern
sed '/ERROR/d' file

# Print only matching lines
sed -n '/ERROR/p' file

# Replace on specific line number
sed '3s/old/new/' file      # Only line 3

# Create backup before editing
sed -i.bak 's/old/new/g' file    # Creates file.bak

# Multiple replacements
sed -e 's/a/A/g' -e 's/b/B/g' file
```

### 24. Awk Print Specific Column
**Problem:** Print only the second column of a CSV/Space separated file.
**Logic:** Use `awk '{print $2}'`.
```bash
#!/bin/bash                       # Shebang line
echo "ID Name Role" | awk '{print $2}'
# awk = pattern scanning and processing language
# $1 = first column, $2 = second column, etc.
# $0 = entire line
```

**Example Output:**
```
$ ./script.sh
Name
```

**Column Visualization:**
```
Input: "ID Name Role"
        $1  $2   $3
           ↓
       print $2 → "Name"
```

**More Awk Examples:**
```bash
# File: employees.txt
# ID  Name    Salary
# 1   Alice   50000
# 2   Bob     60000
# 3   Charlie 55000

# Print name and salary
awk '{print $2, $3}' employees.txt

# Print with formatting
awk '{print "Name: " $2 ", Salary: $" $3}' employees.txt

# Sum a column
awk '{sum += $3} END {print "Total: " sum}' employees.txt

# Filter rows (salary > 55000)
awk '$3 > 55000 {print $2}' employees.txt    # Output: Bob

# Custom field separator (CSV)
awk -F',' '{print $2}' data.csv

# Print line numbers
awk '{print NR ": " $0}' file    # NR = line number
```

### 25. Remove Duplicate Lines
**Problem:** Remove duplicate lines from a sorted file.
**Logic:** Use `uniq`.
```bash
#!/bin/bash                 # Shebang line
sort names.txt | uniq       # Sort first, then remove adjacent duplicates
# sort = arrange lines alphabetically
# uniq = remove ADJACENT duplicate lines (requires sorted input!)
```

**Example:**
```
# names.txt content:
Alice
Bob
Alice
Charlie
Bob

$ ./script.sh
Alice
Bob
Charlie
```

**Why Sort First?**
```
uniq only removes ADJACENT duplicates:

Without sort:          With sort:
Alice                  Alice
Bob                    Alice    ← adjacent, removed
Alice   ← NOT adjacent  Bob
                       Bob      ← adjacent, removed
Result: Alice,Bob,Alice Charlie

                       Result: Alice,Bob,Charlie
```

**Uniq Options:**
```bash
sort file | uniq           # Remove duplicates
sort file | uniq -c        # Count occurrences
sort file | uniq -d        # Show ONLY duplicates
sort file | uniq -u        # Show ONLY unique lines

# Example with count:
$ sort names.txt | uniq -c
      2 Alice
      2 Bob
      1 Charlie
```

### 26. Count Word Frequency
**Problem:** Count occurrences of each word in a file.
**Logic:** Combine `tr`, `sort`, and `uniq -c`.
```bash
#!/bin/bash                           # Shebang line
cat text.txt | tr ' ' '\n' | sort | uniq -c
# cat text.txt      = output file contents
# tr ' ' '\n'       = translate spaces to newlines (one word per line)
# sort              = sort words alphabetically
# uniq -c           = count consecutive duplicates
```

**Example:**
```
# text.txt content:
the quick brown fox jumps over the lazy dog the fox

$ ./script.sh
      1 brown
      1 dog
      2 fox
      1 jumps
      1 lazy
      1 over
      1 quick
      3 the
```

**Pipeline Visualization:**
```
Step 1: cat text.txt
"the quick brown fox jumps over the lazy dog the fox"

Step 2: tr ' ' '\n'   (spaces → newlines)
the
quick
brown
fox
...

Step 3: sort          (alphabetical)
brown
dog
fox
fox
...

Step 4: uniq -c       (count duplicates)
      1 brown
      1 dog
      2 fox
      ...
```

**Sort by Frequency:**
```bash
cat text.txt | tr ' ' '\n' | sort | uniq -c | sort -rn
# -rn = reverse numeric sort (highest first)
      3 the
      2 fox
      1 quick
      ...
```

### 27. Check Disk Space
**Problem:** Alert if disk usage is above 90%.
**Logic:** Parse `df -h` output using `awk` and compare.
```bash
#!/bin/bash                                              # Shebang line
usage=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')
# df /           = disk free for root partition
# grep /         = get line containing "/"
# awk '{print $5}' = extract 5th column (percentage)
# sed 's/%//g'   = remove the % symbol
# Result: just the number (e.g., 75)

if [ $usage -gt 90 ]; then     # If usage > 90
    echo "Disk full!"          # Alert!
fi
```

**Example:**
```
$ df /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1      50000000 45000000   5000000  92% /

$ ./script.sh
Disk full!
```

**Pipeline Breakdown:**
```
df / output:
Filesystem  1K-blocks  Used   Avail Use% Mounted
/dev/sda1   50000000   45M    5G    92%  /
            $1         $2     $3    $4   $5    $6

grep / → gets the data line (not header)
awk '{print $5}' → "92%"
sed 's/%//g' → "92"
```

**Enhanced Version:**
```bash
#!/bin/bash
THRESHOLD=90
usage=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$usage" -gt "$THRESHOLD" ]; then
    echo "WARNING: Disk usage is ${usage}% (threshold: ${THRESHOLD}%)"
    # Send email, Slack, etc.
fi
```

### 28. Check RAM Usage
**Problem:** Display free memory.
**Logic:** Use `free -m`.
```bash
#!/bin/bash                                              # Shebang line
free -m | grep Mem | awk '{print "Free Memory: " $4 " MB"}'
# free -m     = show memory info in megabytes
# grep Mem    = get line starting with "Mem"
# awk         = extract $4 (free column) and format output
```

**Example Output:**
```
$ ./script.sh
Free Memory: 2048 MB
```

**Understanding `free -m` output:**
```
$ free -m
              total   used   free   shared  buff/cache  available
Mem:          16384   8192   2048   512     5632        7168
Swap:          4096   1024   3072

       $1     $2      $3     $4     $5      $6          $7
```

**Column Meanings:**
- `total` ($2) = Total physical RAM
- `used` ($3) = RAM in use
- `free` ($4) = Completely unused RAM
- `available` ($7) = RAM available for applications (free + reclaimable cache)

**Better Script (check available, not free):**
```bash
#!/bin/bash
free -m | awk '/Mem:/ {
    total=$2
    available=$7
    percent=int((available/total)*100)
    print "Available: " available "MB (" percent "%)"
}'
# Output: Available: 7168MB (43%)
```

### 29. Check CPU Load
**Problem:** Print the 1-minute load average.
**Logic:** Use `uptime` or `/proc/loadavg`.
```bash
#!/bin/bash                                          # Shebang line
uptime | awk -F'load average:' '{ print $2 }' | cut -d, -f1
# uptime              = show system uptime and load
# awk -F'load average:' = use "load average:" as field separator
# print $2            = everything after "load average:"
# cut -d, -f1         = split by comma, take first field (1-min avg)
```

**Example Output:**
```
$ uptime
 10:30:00 up 5 days,  2:30,  3 users,  load average: 0.52, 0.58, 0.59

$ ./script.sh
 0.52
```

**Understanding Load Average:**
```
load average: 0.52, 0.58, 0.59
              │     │     │
              │     │     └─ 15-minute average
              │     └─ 5-minute average
              └─ 1-minute average

Load = number of processes waiting for CPU
- Load 1.0 on 1-core CPU = 100% utilized
- Load 1.0 on 4-core CPU = 25% utilized
- Load 4.0 on 4-core CPU = 100% utilized
```

**Alternative Methods:**
```bash
# Read directly from proc filesystem
cat /proc/loadavg | cut -d' ' -f1

# Using top (batch mode)
top -bn1 | grep "load average" | awk '{print $12}'
```

### 30. Check Service Status
**Problem:** Check if Nginx is running.
**Logic:** Use `systemctl is-active`.
```bash
#!/bin/bash                                    # Shebang line
if systemctl is-active --quiet nginx; then     # Check if nginx is running
    echo "Nginx is running"                    # --quiet: no output, just exit code
else                                           # Exit 0=active, non-zero=inactive
    echo "Nginx is NOT running"
fi
```

**Example Output:**
```
$ sudo systemctl start nginx
$ ./script.sh
Nginx is running

$ sudo systemctl stop nginx
$ ./script.sh
Nginx is NOT running
```

**How `--quiet` works:**
```bash
# Without --quiet:
$ systemctl is-active nginx
active

# With --quiet (just exit code):
$ systemctl is-active --quiet nginx
$ echo $?
0    # 0 means active/running

$ systemctl is-active --quiet nginx
$ echo $?
3    # non-zero means inactive/stopped
```

**Service States:**
```bash
systemctl is-active service   # Is it running?
systemctl is-enabled service  # Will it start on boot?
systemctl status service      # Full status info

# Common states:
# active   - running
# inactive - stopped
# failed   - crashed/error
```

### 31. Restart Service if Down
**Problem:** Restart a service if it's not active.
**Logic:** Combine check and `systemctl restart`.
```bash
#!/bin/bash                                       # Shebang line
service="nginx"                                   # Service name variable
if ! systemctl is-active --quiet "$service"; then # ! negates: if NOT active
    echo "$service down, restarting..."          # Log the action
    sudo systemctl restart "$service"            # Restart the service
fi                                               # End if block
```

**Example Output:**
```
$ sudo systemctl stop nginx    # Simulate crash
$ ./script.sh
nginx down, restarting...

$ ./script.sh                  # Run again
# (no output - service is running)
```

**Logic Flow:**
```
systemctl is-active --quiet nginx
         │
         ├─ Returns 0 (true)  → ! makes it false → skip restart
         └─ Returns 3 (false) → ! makes it true  → do restart
```

**Enhanced Version with Logging:**
```bash
#!/bin/bash
service="nginx"
log_file="/var/log/service-monitor.log"

if ! systemctl is-active --quiet "$service"; then
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] $service was down, restarting..." >> "$log_file"
    sudo systemctl restart "$service"
    
    if systemctl is-active --quiet "$service"; then
        echo "[$timestamp] $service restarted successfully" >> "$log_file"
    else
        echo "[$timestamp] FAILED to restart $service!" >> "$log_file"
    fi
fi
```

### 32. System Info Report
**Problem:** Generate a file with date, hostname, and uptime.
**Logic:** Redirect `>` standard output to a file.
```bash
#!/bin/bash                           # Shebang line
echo "Date: $(date)" > info.txt       # > creates/overwrites file
echo "Hostname: $(hostname)" >> info.txt  # >> appends to file
echo "Uptime: $(uptime)" >> info.txt      # >> appends more
```

**Example Output (info.txt content):**
```
Date: Wed Jan 22 10:30:00 UTC 2026
Hostname: web-server-01
Uptime: 10:30:00 up 5 days, 2:30, 3 users, load average: 0.52, 0.58, 0.59
```

**Redirect Operators:**
```bash
>   → Create file or OVERWRITE if exists
>>  → Create file or APPEND if exists

echo "first" > file.txt    # file.txt contains: first
echo "second" > file.txt   # file.txt contains: second (overwritten!)

echo "first" > file.txt    # file.txt contains: first
echo "second" >> file.txt  # file.txt contains: first\nsecond (appended)
```

**Enhanced System Report:**
```bash
#!/bin/bash
report="system_report_$(date +%F).txt"

cat << EOF > "$report"
=== System Report ===
Generated: $(date)

Hostname: $(hostname)
OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)
Kernel: $(uname -r)
Uptime: $(uptime -p)

CPU Cores: $(nproc)
RAM Total: $(free -h | awk '/Mem:/ {print $2}')
Disk Usage: $(df -h / | awk 'NR==2 {print $5}')

Top 5 Processes by Memory:
$(ps aux --sort=-%mem | head -6)
EOF

echo "Report saved to $report"
```

### 33. Check Root Privileges
**Problem:** Ensure the script is run as sudo/root.
**Logic:** Check valid user ID (`$EUID` or `id -u`). Root is 0.
```bash
#!/bin/bash                       # Shebang line
if [ "$EUID" -ne 0 ]; then        # EUID = Effective User ID
    echo "Please run as root"     # -ne = not equal; root's ID is always 0
    exit 1                        # Exit with error code
fi                                # Continue if root
```

**Example Output:**
```
$ ./script.sh
Please run as root

$ sudo ./script.sh
# (continues execution)
```

**Understanding User IDs:**
```bash
$ id
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo)

$ sudo id
uid=0(root) gid=0(root) groups=0(root)

# UID 0 = root user (always)
# Regular users start from 1000
```

**Alternative Methods:**
```bash
# Method 1: Using $EUID
if [ "$EUID" -ne 0 ]; then
    echo "Not root"
fi

# Method 2: Using id command
if [ "$(id -u)" -ne 0 ]; then
    echo "Not root"
fi

# Method 3: Check username
if [ "$USER" != "root" ]; then
    echo "Not root"
fi

# Method 4: Auto-elevate with sudo
if [ "$EUID" -ne 0 ]; then
    exec sudo "$0" "$@"    # Re-run this script with sudo
fi
```

### 34. Port Connectivity Check
**Problem:** Check if a remote port is open (e.g., Database port).
**Logic:** Use `nc` (netcat) or `/dev/tcp`.
```bash
#!/bin/bash                      # Shebang line
nc -z -v -w5 google.com 80       # Test TCP connection to port 80
# nc = netcat (network utility)
# -z = zero I/O mode (just check, don't send data)
# -v = verbose output
# -w5 = timeout after 5 seconds
# google.com = target host
# 80 = target port (HTTP)
```

**Example Output:**
```
$ nc -z -v -w5 google.com 80
Connection to google.com 80 port [tcp/http] succeeded!

$ nc -z -v -w5 google.com 12345
nc: connect to google.com port 12345 (tcp) timed out: Operation now in progress
```

**Using in Scripts:**
```bash
#!/bin/bash
host="db-server.internal"
port=5432

if nc -z -w5 "$host" "$port" 2>/dev/null; then
    echo "✓ Database is reachable"
else
    echo "✗ Database is NOT reachable"
    exit 1
fi
```

**Alternative Methods:**
```bash
# Method 1: Using /dev/tcp (bash built-in)
if timeout 5 bash -c "echo > /dev/tcp/google.com/80" 2>/dev/null; then
    echo "Port open"
fi

# Method 2: Using telnet
timeout 5 telnet google.com 80 2>/dev/null | grep -q Connected

# Method 3: Using curl
if curl -s --connect-timeout 5 telnet://host:port &>/dev/null; then
    echo "Port open"
fi

# Check multiple ports
for port in 22 80 443 3306; do
    nc -z -w2 server.com $port && echo "$port open" || echo "$port closed"
done
```

### 35. Website Availability (Curl)
**Problem:** Check if a website returns 200 OK.
**Logic:** Use `curl` with write-out variable.
```bash
#!/bin/bash                                            # Shebang line
url="http://example.com"                               # Target URL
code=$(curl -s -o /dev/null -w "%{http_code}" "$url")  # Get HTTP status code
# -s = silent mode (no progress bar)
# -o /dev/null = discard response body
# -w "%{http_code}" = write out only the HTTP status code

if [ "$code" -eq 200 ]; then     # Check if code equals 200
    echo "Site is Up"            # Success!
else                             # Any other code
    echo "Site is Down ($code)"  # Show the actual code for debugging
fi
```

**Example Output:**
```
$ ./script.sh
Site is Up

# If site is down:
$ ./script.sh
Site is Down (503)
```

**Common HTTP Status Codes:**
```
200 = OK (Success)
301 = Moved Permanently (Redirect)
302 = Found (Temporary Redirect)
400 = Bad Request
401 = Unauthorized
403 = Forbidden
404 = Not Found
500 = Internal Server Error
502 = Bad Gateway
503 = Service Unavailable
```

**Enhanced Version:**
```bash
#!/bin/bash
url="http://example.com"
timeout=10

# Get both code and time
response=$(curl -s -o /dev/null -w "%{http_code}:%{time_total}" \
           --connect-timeout $timeout "$url")

code=$(echo $response | cut -d: -f1)
time=$(echo $response | cut -d: -f2)

if [ "$code" -eq 200 ]; then
    echo "✓ $url is UP (${time}s)"
else
    echo "✗ $url is DOWN (HTTP $code)"
fi
```

### 36. Parse Command Arguments
**Problem:** Accept filename as a command Line argument.
**Logic:** Check if `$1` is empty.
```bash
#!/bin/bash                       # Shebang line
if [ -z "$1" ]; then              # -z checks if string is empty (zero length)
    echo "Usage: $0 filename"     # $0 = script name itself
    exit 1                        # Exit with error
fi                                # End if
echo "Processing $1..."           # $1 = first argument passed to script
```

**Example Output:**
```
$ ./script.sh
Usage: ./script.sh filename

$ ./script.sh myfile.txt
Processing myfile.txt...
```

**Understanding Positional Parameters:**
```bash
./script.sh arg1 arg2 arg3 arg4
    $0      $1   $2   $3   $4

$0  = Script name (./script.sh)
$1  = First argument (arg1)
$2  = Second argument (arg2)
$#  = Number of arguments (4)
$@  = All arguments as separate words
$*  = All arguments as single string
```

**Multiple Arguments Example:**
```bash
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "Arg count: $#"
echo "All args: $@"

# Check minimum arguments
if [ $# -lt 2 ]; then
    echo "Need at least 2 arguments"
    exit 1
fi

# Loop through all arguments
for arg in "$@"; do
    echo "Processing: $arg"
done
```

### 37. Parse Flags (getopts)
**Problem:** Handle flags like `-u username -p password`.
**Logic:** Use `getopts` loop.
```bash
#!/bin/bash                               # Shebang line
while getopts "u:p:" opt; do              # Loop through options
  # getopts "u:p:" = expect -u and -p flags, : means they need values
  case $opt in                            # Check which option was found
    u) user="$OPTARG" ;;                  # -u found, OPTARG has its value
    p) pass="$OPTARG" ;;                  # -p found, OPTARG has its value
    *) echo "Invalid flag"; exit 1 ;;    # Unknown flag
  esac                                    # End case
done                                      # End while
echo "User: $user"                        # Use parsed values
```

**Example Output:**
```
$ ./script.sh -u admin -p secret123
User: admin

$ ./script.sh -u john
User: john
```

**Understanding getopts:**
```
getopts "u:p:vh" opt
         │ ││ │
         │ ││ └─ h = -h flag (no value, help)
         │ │└─ v = -v flag (no value, verbose)
         │ └─ : = -p requires a value
         └─ u: = -u requires a value

Colon after letter = requires argument
No colon = flag only (boolean)
```

**Complete Example with Help:**
```bash
#!/bin/bash
usage() {
    echo "Usage: $0 -u username -p password [-v] [-h]"
    exit 1
}

verbose=false
while getopts "u:p:vh" opt; do
    case $opt in
        u) user="$OPTARG" ;;
        p) pass="$OPTARG" ;;
        v) verbose=true ;;
        h) usage ;;
        *) usage ;;
    esac
done

# Validate required options
if [ -z "$user" ] || [ -z "$pass" ]; then
    echo "Error: -u and -p are required"
    usage
fi

[ "$verbose" = true ] && echo "Verbose mode ON"
echo "Connecting as $user..."
```

### 38. Error Handling (Exit Codes)
**Problem:** Exit script if a command fails.
**Logic:** Check `$?` (exit status of last command) or use `set -e`.
```bash
#!/bin/bash                        # Shebang line
cp source.txt dest.txt             # Try to copy file
if [ $? -ne 0 ]; then              # $? = exit code of previous command
    echo "Copy failed"             # -ne 0 means command failed
    exit 1                         # Exit script with error code
fi                                 # Continue if successful
```

**Example Output:**
```
$ ls
source.txt
$ ./script.sh
# (no output, copy succeeded)

$ rm source.txt
$ ./script.sh
cp: cannot stat 'source.txt': No such file or directory
Copy failed
```

**Understanding Exit Codes:**
```
Exit Code 0    = Success
Exit Code 1-255 = Error (different codes for different errors)

$ ls /tmp           # exists
$ echo $?
0                   # success

$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ echo $?
2                   # error
```

**Better Error Handling Methods:**
```bash
# Method 1: set -e (exit on any error)
#!/bin/bash
set -e              # Script stops on first error
cp source.txt dest.txt
rm old.txt          # If this fails, script exits immediately
echo "All done"     # Only reached if all commands succeed

# Method 2: OR operator for error handling
cp source.txt dest.txt || { echo "Copy failed"; exit 1; }

# Method 3: Trap ERR signal
trap 'echo "Error on line $LINENO"; exit 1' ERR
cp source.txt dest.txt    # If fails, trap catches it

# Method 4: Full defensive scripting
set -euo pipefail   # e=exit on error, u=error on undefined vars, pipefail=pipe errors
```

### 39. Trap Signals (Cleanup)
**Problem:** Clean up temporary files if user presses Ctrl+C.
**Logic:** Use `trap` command.
```bash
#!/bin/bash                                          # Shebang line
temp_file="temp.dat"                                 # Temp file we'll create
touch "$temp_file"                                   # Create the temp file
trap "rm -f $temp_file; echo ' Cleaned up'; exit" SIGINT
# trap = set signal handler
# "commands" = what to do when signal received
# SIGINT = signal sent by Ctrl+C (interrupt)

echo "Running... (Press Ctrl+C to stop)"
sleep 10                                             # Simulate long work
```

**Example Output:**
```
$ ./script.sh
Running... (Press Ctrl+C to stop)
^C Cleaned up

$ ls temp.dat
ls: cannot access 'temp.dat': No such file or directory
# File was cleaned up!
```

**Common Signals:**
```
SIGINT  (2)  = Ctrl+C interrupt
SIGTERM (15) = Termination request (kill command)
SIGHUP  (1)  = Terminal closed
SIGQUIT (3)  = Ctrl+\ quit
EXIT    (0)  = Script exit (not a real signal, but trap catches it)
```

**Practical Cleanup Example:**
```bash
#!/bin/bash
# Create temp directory for work
tmp_dir=$(mktemp -d)
log_file="$tmp_dir/process.log"

# Cleanup function
cleanup() {
    echo "Cleaning up..."
    rm -rf "$tmp_dir"
    echo "Done!"
}

# Trap multiple signals
trap cleanup EXIT SIGINT SIGTERM
# EXIT = runs cleanup on normal exit too

# Your work here
echo "Working in $tmp_dir"
echo "Log data" > "$log_file"
sleep 30

# Cleanup runs automatically on exit or Ctrl+C
```

### 40. Create User Account
**Problem:** Create a user if it doesn't exist.
**Logic:** Use `id` to check and `useradd` to create.
```bash
#!/bin/bash                         # Shebang line
user="devops_user"                  # Username to create
if id "$user" &>/dev/null; then     # id checks if user exists
    echo "User exists"              # &>/dev/null silences all output
else                                # User doesn't exist
    sudo useradd "$user"            # Create the user
    echo "User created"
fi
```

**Example Output:**
```
$ ./script.sh
User created

$ ./script.sh
User exists

$ id devops_user
uid=1001(devops_user) gid=1001(devops_user) groups=1001(devops_user)
```

**Understanding the Check:**
```bash
id username
# If user exists: outputs user info, exit code 0
# If user missing: outputs error, exit code 1

&>/dev/null
# & = both stdout and stderr
# > = redirect
# /dev/null = discard (black hole)
# So: run silently, just check exit code
```

**Complete User Creation Script:**
```bash
#!/bin/bash
username="$1"

if [ -z "$username" ]; then
    echo "Usage: $0 username"
    exit 1
fi

if id "$username" &>/dev/null; then
    echo "User '$username' already exists"
else
    # Create user with home directory and bash shell
    sudo useradd -m -s /bin/bash "$username"
    
    # Set initial password
    echo "$username:changeme123" | sudo chpasswd
    
    # Force password change on first login
    sudo chage -d 0 "$username"
    
    echo "User '$username' created successfully"
    echo "Temporary password: changeme123"
fi
```

### 41. Backup Directory (Tar)
**Problem:** Compress a folder into a tarball with a timestamp.
**Logic:** Use `tar czf` and `date`.
```bash
#!/bin/bash                                       # Shebang line
src="/var/www/html"                               # Source directory to backup
dest="/backups/site-$(date +%F).tar.gz"           # Destination with date stamp
# $(date +%F) = YYYY-MM-DD format (e.g., 2026-01-22)
tar -czf "$dest" "$src"                           # Create compressed archive
# tar = tape archive command
# -c = create new archive
# -z = compress with gzip
# -f = filename follows
echo "Backup saved to $dest"                      # Confirm completion
```

**Example Output:**
```
$ ./script.sh
Backup saved to /backups/site-2026-01-22.tar.gz

$ ls -lh /backups/
-rw-r--r-- 1 root root 15M Jan 22 10:30 site-2026-01-22.tar.gz
```

**Understanding Tar Flags:**
```
Create:  tar -czf archive.tar.gz /source
              │││
              ││└─ f = filename (archive.tar.gz)
              │└─ z = gzip compression
              └─ c = create archive

Extract: tar -xzf archive.tar.gz
              │
              └─ x = extract (instead of c)

List:    tar -tzf archive.tar.gz
              │
              └─ t = list contents (instead of c or x)
```

**Date Format Options:**
```bash
date +%F           # 2026-01-22 (YYYY-MM-DD)
date +%Y%m%d       # 20260122
date +%Y-%m-%d_%H%M # 2026-01-22_1030
date +%s           # 1737539400 (Unix timestamp)
```

**Enhanced Backup Script:**
```bash
#!/bin/bash
src="/var/www/html"
backup_dir="/backups"
retention_days=7

# Create backup
filename="site-$(date +%F_%H%M%S).tar.gz"
tar -czf "$backup_dir/$filename" "$src"

# Delete old backups
find "$backup_dir" -name "site-*.tar.gz" -mtime +$retention_days -delete

echo "Backup: $filename (old backups cleaned)"
```

### 42. Remote Copy (SCP Wrapper)
**Problem:** Copy a file to a remote server.
**Logic:** Use `scp`.
```bash
#!/bin/bash                                         # Shebang line
scp myfile.txt user@remote-server:/home/user/       # Secure copy
# scp = Secure Copy Protocol (uses SSH)
# myfile.txt = local file to copy
# user = remote username
# remote-server = hostname or IP
# :/home/user/ = destination path on remote server
```

**Example Output:**
```
$ ./script.sh
myfile.txt                   100%  1024     1.0KB/s   00:01
```

**SCP Syntax Variations:**
```bash
# Local to Remote (upload)
scp file.txt user@server:/path/
scp -r folder/ user@server:/path/          # -r = recursive (folders)

# Remote to Local (download)
scp user@server:/path/file.txt ./local/
scp -r user@server:/path/folder/ ./local/

# Remote to Remote
scp user@server1:/file.txt user@server2:/path/

# With custom SSH port
scp -P 2222 file.txt user@server:/path/    # -P = port

# With SSH key
scp -i ~/.ssh/mykey.pem file.txt user@server:/path/
```

**Practical Wrapper Script:**
```bash
#!/bin/bash
# Usage: ./deploy.sh local_file remote_path

if [ $# -lt 2 ]; then
    echo "Usage: $0 <local_file> <remote_path>"
    exit 1
fi

local_file="$1"
remote_path="$2"
server="deploy@production-server"

# Copy with progress and compression
scp -C "$local_file" "$server:$remote_path"
# -C = enable compression

if [ $? -eq 0 ]; then
    echo "✓ Deployed $local_file to $remote_path"
else
    echo "✗ Deployment failed!"
    exit 1
fi
```

### 43. Monitor Log for Errors
**Problem:** Real-time monitoring of logs for the word "Error".
**Logic:** Use `tail -f` piped to `grep`.
```bash
#!/bin/bash                                           # Shebang line
tail -f /var/log/syslog | grep --line-buffered "Error"
# tail -f = follow file (show new lines as they're added)
# | = pipe output to next command
# grep = filter for lines containing "Error"
# --line-buffered = output each match immediately (don't buffer)
```

**Example Output:**
```
$ ./script.sh
Jan 22 10:30:01 server app[1234]: Error: Connection refused
Jan 22 10:30:15 server app[1234]: Error: Timeout occurred
# Continues showing new errors in real-time...
# Press Ctrl+C to stop
```

**Why `--line-buffered`?**
```
Without --line-buffered:
- grep buffers output for efficiency
- You might not see matches immediately

With --line-buffered:
- Each matching line is printed immediately
- Essential for real-time monitoring
```

**Enhanced Monitoring Script:**
```bash
#!/bin/bash
log_file="/var/log/syslog"
patterns="Error|Warning|Critical|Failed"

echo "Monitoring $log_file for: $patterns"
echo "Press Ctrl+C to stop"
echo "-----------------------------------"

tail -f "$log_file" | grep --line-buffered -E "$patterns" | while read line; do
    timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] $line"
    
    # Optional: Send alert for critical errors
    if echo "$line" | grep -q "Critical"; then
        echo "ALERT: Critical error detected!" | mail -s "Alert" admin@example.com
    fi
done
```

**Alternative: Multiple Files**
```bash
# Monitor multiple log files simultaneously
tail -f /var/log/syslog /var/log/app/*.log | grep --line-buffered "Error"
```

### 44. Send Email Alert
**Problem:** Send an email if a condition is met.
**Logic:** Use `mail` command (requires mail utils).
```bash
#!/bin/bash                                                # Shebang line
echo "Disk space low" | mail -s "Alert" admin@example.com
# echo "message" = email body content
# | = pipe to mail command
# mail = send email command
# -s "Alert" = subject line
# admin@example.com = recipient address
```

**Example Output:**
```
$ ./script.sh
# (no output - email sent silently)

# Check mail queue
$ mailq
-Queue ID-  --Size-- ----Arrival Time---- -Sender/Recipient-------
ABC123      150 Wed Jan 22 10:30:00  root@server
                                    admin@example.com
```

**Prerequisites:**
```bash
# Install mail utilities (Ubuntu/Debian)
sudo apt-get install mailutils

# Install (RHEL/CentOS)
sudo yum install mailx

# Note: Requires configured mail server (postfix, sendmail, etc.)
```

**More Mail Options:**
```bash
# Simple message
echo "Body text" | mail -s "Subject" user@example.com

# From file
mail -s "Report" user@example.com < report.txt

# With attachment (using mutt)
echo "See attached" | mutt -s "Report" -a file.pdf -- user@example.com

# Multiple recipients
echo "Alert" | mail -s "Alert" user1@example.com,user2@example.com

# CC and BCC
echo "Message" | mail -s "Subject" -c cc@example.com -b bcc@example.com to@example.com
```

**Practical Alert Script:**
```bash
#!/bin/bash
THRESHOLD=90
EMAIL="admin@example.com"
HOSTNAME=$(hostname)

usage=$(df / | tail -1 | awk '{print $5}' | tr -d '%')

if [ "$usage" -gt "$THRESHOLD" ]; then
    cat << EOF | mail -s "[⚠️ ALERT] Disk Space on $HOSTNAME" "$EMAIL"
WARNING: Disk usage on $HOSTNAME has exceeded ${THRESHOLD}%

Current Usage: ${usage}%
Timestamp: $(date)

Disk Details:
$(df -h /)

Please investigate immediately.
EOF
fi
```

### 45. JSON Parsing (jq)
**Problem:** Extract a value from a JSON file.
**Logic:** Use `jq` tool (standard for JSON in shell).
```bash
#!/bin/bash                                  # Shebang line
# { "name": "MyApp", "version": "1.0" }      # Sample JSON structure
version=$(cat config.json | jq -r '.version')
# cat config.json = output file contents
# jq = JSON processor
# -r = raw output (no quotes around strings)
# '.version' = extract the "version" field
echo "Version is $version"                   # Display result
```

**Example Output:**
```
$ cat config.json
{"name": "MyApp", "version": "1.0"}

$ ./script.sh
Version is 1.0
```

**Understanding jq Syntax:**
```bash
# Given JSON: {"name": "MyApp", "version": "1.0", "port": 8080}

jq '.name'      # "MyApp" (with quotes)
jq -r '.name'   # MyApp (raw, no quotes)
jq '.port'      # 8080

# Nested JSON: {"server": {"host": "localhost", "port": 8080}}
jq '.server.host'    # "localhost"
jq '.server.port'    # 8080
```

**Common jq Operations:**
```bash
# Array access: {"items": ["a", "b", "c"]}
jq '.items[0]'       # "a" (first element)
jq '.items[-1]'      # "c" (last element)
jq '.items | length' # 3 (array length)

# Filter array: {"users": [{"name":"Alice","age":30}, {"name":"Bob","age":25}]}
jq '.users[] | select(.age > 26)'    # Users older than 26
jq '.users[].name'                    # All names

# Create new JSON
jq '{app: .name, ver: .version}' config.json
# Output: {"app": "MyApp", "ver": "1.0"}

# Multiple fields
jq -r '[.name, .version] | @csv' config.json
# Output: "MyApp","1.0"
```

**Practical Example:**
```bash
#!/bin/bash
# Parse Kubernetes pod status
kubectl get pods -o json | jq -r '
  .items[] | 
  "\(.metadata.name)\t\(.status.phase)\t\(.status.podIP)"
'
```

### 46. Database Backup Mock
**Problem:** Simulate a DB backup command.
**Logic:** Execute dump command and check success.
```bash
#!/bin/bash                              # Shebang line
db_name="mydb"                           # Database name to backup
# mysqldump -u root -p $db_name > $db_name.sql   # Actual command (commented)
echo "Backing up $db_name..."            # Simulation message
```

**Real-World Database Backup Script:**
```bash
#!/bin/bash
# Configuration
DB_HOST="localhost"
DB_USER="backup_user"
DB_PASS="secret123"            # Better: use ~/.my.cnf or env var
DB_NAME="production_db"
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory if needed
mkdir -p "$BACKUP_DIR"        # -p = create parent dirs if needed

# Perform backup
backup_file="$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"
mysqldump -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" | gzip > "$backup_file"
# mysqldump = MySQL backup utility
# -h = host
# -u = username  
# -p = password (no space after -p!)
# | gzip = compress output
# > = save to file

# Check if backup succeeded
if [ $? -eq 0 ] && [ -s "$backup_file" ]; then
    echo "✓ Backup successful: $backup_file"
    echo "  Size: $(ls -lh "$backup_file" | awk '{print $5}')"
    
    # Cleanup old backups (keep 7 days)
    find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +7 -delete
else
    echo "✗ Backup FAILED!"
    exit 1
fi
```

**PostgreSQL Version:**
```bash
pg_dump -h localhost -U postgres mydb | gzip > backup.sql.gz
```

**MongoDB Version:**
```bash
mongodump --db mydb --archive=backup.gz --gzip
```

### 47. Docker Status Check
**Problem:** Check if a specific container is running.
**Logic:** `docker ps` with grep.
```bash
#!/bin/bash                                 # Shebang line
container="my-app"                          # Container name to check
if docker ps | grep -q "$container"; then   # docker ps = list running containers
    echo "Container running"                # grep -q = quiet (just exit code)
else                                        # -q returns 0 if found, 1 if not
    echo "Container stopped"
fi
```

**Example Output:**
```
$ docker ps
CONTAINER ID   IMAGE     COMMAND   STATUS    NAMES
a1b2c3d4e5f6   nginx     "nginx"   Up 2h     my-app

$ ./script.sh
Container running

$ docker stop my-app
$ ./script.sh
Container stopped
```

**Better Docker Check:**
```bash
#!/bin/bash
container="my-app"

# Method 1: Check specific container status
status=$(docker inspect -f '{{.State.Status}}' "$container" 2>/dev/null)
if [ "$status" == "running" ]; then
    echo "Container is running"
else
    echo "Container is $status (or doesn't exist)"
fi

# Method 2: Using docker ps filter
if [ "$(docker ps -q -f name=$container)" ]; then
    echo "Running"
else
    echo "Not running"
fi
```

**Docker Management Script:**
```bash
#!/bin/bash
container="my-app"
image="nginx:latest"

# Check if container exists
if ! docker ps -a --format '{{.Names}}' | grep -q "^${container}$"; then
    echo "Creating container..."
    docker run -d --name "$container" -p 80:80 "$image"
elif ! docker ps --format '{{.Names}}' | grep -q "^${container}$"; then
    echo "Starting container..."
    docker start "$container"
else
    echo "Container already running"
fi
```

### 48. Git Auto Pull
**Problem:** Pull changes from a git repo.
**Logic:** Navigate to dir and run `git pull`.
```bash
#!/bin/bash                     # Shebang line
cd /opt/project                 # Change to project directory
git pull origin main            # Pull latest changes from remote
# git pull = fetch + merge
# origin = remote repository name (default)
# main = branch name to pull
```

**Example Output:**
```
$ ./script.sh
Remote: Counting objects: 3, done.
Remote: Compressing objects: 100% (2/2), done.
Unpacking objects: 100% (3/3), done.
From github.com:user/project
   abc1234..def5678  main -> origin/main
Updating abc1234..def5678
Fast-forward
 config.yml | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

**Enhanced Git Pull Script:**
```bash
#!/bin/bash
PROJECT_DIR="/opt/project"
BRANCH="main"

# Change to directory (exit if fails)
cd "$PROJECT_DIR" || { echo "Directory not found!"; exit 1; }

# Check for uncommitted changes
if [ -n "$(git status --porcelain)" ]; then
    echo "⚠️  Warning: Uncommitted changes exist"
    git status --short
    echo ""
    read -p "Stash changes and continue? (y/n): " answer
    if [ "$answer" == "y" ]; then
        git stash
    else
        echo "Aborting."
        exit 1
    fi
fi

# Pull changes
echo "Pulling from origin/$BRANCH..."
git pull origin "$BRANCH"

if [ $? -eq 0 ]; then
    echo "✓ Pull successful"
    echo "Current commit: $(git log -1 --format='%h %s')"
else
    echo "✗ Pull failed!"
    exit 1
fi
```

**Auto-Deploy Script:**
```bash
#!/bin/bash
cd /var/www/myapp
git pull origin main
npm install          # Install new dependencies
npm run build        # Build app
sudo systemctl restart myapp   # Restart service
```

### 49. Parallel Execution
**Problem:** Run two background tasks and wait for them.
**Logic:** Use `&` to background and `wait` to pause.
```bash
#!/bin/bash                   # Shebang line
command1 &                    # Run command1 in background
pid1=$!                       # $! = PID of last backgrounded process
command2 &                    # Run command2 in background  
pid2=$!                       # Save its PID too
wait $pid1 $pid2              # Wait for BOTH to finish
echo "Both tasks done"        # Continues after both complete
```

**Example with Real Commands:**
```bash
#!/bin/bash
echo "Starting parallel tasks..."

# Task 1: Download file
curl -sO https://example.com/file1.zip &
pid1=$!
echo "Started download 1 (PID: $pid1)"

# Task 2: Download another file
curl -sO https://example.com/file2.zip &
pid2=$!
echo "Started download 2 (PID: $pid2)"

# Wait for both
echo "Waiting for downloads to complete..."
wait $pid1 $pid2
echo "✓ Both downloads finished!"
```

**Execution Flow:**
```
Time →

     |-- command1 running (background) --|
     |-- command2 running (background) -----|
     |
start &    &                              wait
     |                                      |
     |                                      |
  launch                              both done
  both                                continue
```

**Parallel with Exit Code Checking:**
```bash
#!/bin/bash

# Run tasks
task1() { sleep 2; echo "Task 1 done"; return 0; }
task2() { sleep 3; echo "Task 2 done"; return 0; }

task1 &
pid1=$!

task2 &
pid2=$!

# Wait and check each result
wait $pid1
result1=$?

wait $pid2  
result2=$?

# Check results
if [ $result1 -eq 0 ] && [ $result2 -eq 0 ]; then
    echo "✓ All tasks succeeded"
else
    echo "✗ Some tasks failed"
    exit 1
fi
```

**Parallel Server Health Checks:**
```bash
#!/bin/bash
servers=("server1" "server2" "server3" "server4")

for server in "${servers[@]}"; do
    ping -c 1 -W 2 "$server" > /dev/null && echo "✓ $server" || echo "✗ $server" &
done

wait
echo "All checks complete"
```

### 50. Random Password Generator
**Problem:** Generate a random 12-character alphanumeric string.
**Logic:** Read from `/dev/urandom` and filter with `tr`.
```bash
#!/bin/bash                                           # Shebang line
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 12 | head -n 1
# cat /dev/urandom = read random bytes from system
# tr -dc 'a-zA-Z0-9' = delete all chars NOT in this set
#                     -d = delete, -c = complement (NOT in set)
# fold -w 12 = wrap lines at 12 characters
# head -n 1 = take only first line (one password)
```

**Example Output:**
```
$ ./script.sh
K7mPx2nQ9bWs

$ ./script.sh
a3FjL8vYcR1n
```

**Understanding Each Step:**
```
/dev/urandom output: \x8f\xab\x12K7\xff\x00mP...(binary garbage)
        ↓
tr -dc 'a-zA-Z0-9': K7mPx2nQ9bWsL8vYcR1na3Fj... (only letters/numbers)
        ↓
fold -w 12:    K7mPx2nQ9bWs
               L8vYcR1na3Fj
               ...
        ↓
head -n 1:     K7mPx2nQ9bWs  (just first line)
```

**Password Generation Variations:**
```bash
# Include special characters
tr -dc 'a-zA-Z0-9!@#$%^&*' < /dev/urandom | head -c 16

# Using openssl (if available)
openssl rand -base64 12 | tr -d '=' | head -c 12

# Using /dev/random (more secure but slower)
tr -dc 'a-zA-Z0-9' < /dev/random | head -c 12
```

**Complete Password Generator Script:**
```bash
#!/bin/bash
# Usage: ./genpass.sh [length] [type]
# Types: alpha, num, alphanum, special

length=${1:-12}          # Default 12 if not specified
type=${2:-alphanum}      # Default alphanum

case $type in
    alpha)     chars='a-zA-Z' ;;
    num)       chars='0-9' ;;
    alphanum)  chars='a-zA-Z0-9' ;;
    special)   chars='a-zA-Z0-9!@#$%^&*()_+-=' ;;
    *)         echo "Unknown type: $type"; exit 1 ;;
esac

password=$(tr -dc "$chars" < /dev/urandom | head -c "$length")
echo "Generated password: $password"
echo "Length: ${#password} characters"
```
