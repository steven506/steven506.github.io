# Shell Scripting for DevOps

The goal is to document useful Bash scripting patterns that can be reused for troubleshooting, automation, health checks, log analysis, deployment validation, and daily Linux operations.

---

## Table of Contents

- [1. Basic Shell Script Structure](#basic-shell-script-structure)
- [2. Variables](#variables)
- [3. Script Arguments](#script-arguments)
- [4. Validate Required Arguments](#validate-required-arguments)
- [5. Exit Codes](#exit-codes)
- [6. Conditions](#conditions)
- [7. Case Statement](#case-statement)
- [8. Loops](#loops)
- [9. Functions](#functions)
- [10. Reading User Input](#reading-user-input)
- [11. Command Substitution](#command-substitution)
- [12. Input and Output Redirection](#input-and-output-redirection)
- [13. Pipes](#pipes)
- [14. Useful Script Safety Options](#useful-script-safety-options)
- [15. Logging Pattern](#logging-pattern)
- [16. Basic DevOps Script Examples](#basic-devops-script-examples)
- [17. Bash Scripting Best Practices](#bash-scripting-best-practices)
---

<a id="basic-shell-script-structure"></a>

## 1. Basic Shell Script Structure

A basic Bash script usually starts with a shebang, variables, commands, and optional logic.

```bash
#!/bin/bash

MESSAGE="Hello from Bash"
echo "$MESSAGE"
```

Run the script:

```bash
chmod +x script.sh
./script.sh
```

Basic structure:

```bash
#!/bin/bash

# Variables
APP_NAME="my-app"
ENVIRONMENT="dev"

# Command output
echo "Starting script for $APP_NAME in $ENVIRONMENT"

# Logic
if [ "$ENVIRONMENT" = "dev" ]; then
  echo "Running in development mode"
fi
```

---

<a id="variables"></a>

## 2. Variables

Variables store values that can be reused in the script.

```bash
#!/bin/bash

APP_NAME="payment-service"
PORT=8080
ENVIRONMENT="production"

echo "Application: $APP_NAME"
echo "Port: $PORT"
echo "Environment: $ENVIRONMENT"
```

Use quotes around variables to avoid issues with spaces or empty values.

Good:

```bash
if [ "$APP_NAME" = "payment-service" ]; then
  echo "App matched"
fi
```

Avoid:

```bash
if [ $APP_NAME = payment-service ]; then
  echo "App matched"
fi
```

Why quotes matter:

```bash
APP_NAME="payment service"

if [ "$APP_NAME" = "payment service" ]; then
  echo "App matched"
fi
```

Without quotes, values with spaces can break the script.

---

<a id="script-arguments"></a>

## 3. Script Arguments

Arguments allow you to pass values to a script from the command line.

```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Number of arguments: $#"
```

Run:

```bash
./deploy.sh dev payment-service
```

Example output:

```text
Script name: ./deploy.sh
First argument: dev
Second argument: payment-service
Number of arguments: 2
```

Common use:

```bash
#!/bin/bash

ENVIRONMENT=$1
APP_NAME=$2

echo "Deploying $APP_NAME to $ENVIRONMENT"
```

Common special variables:

```text
$0   Script name
$1   First argument
$2   Second argument
$3   Third argument
$#   Number of arguments
$?   Exit code of the last command
$$   Process ID of the current script
$@   All arguments
```

---

<a id="validate-required-arguments"></a>

## 4. Validate Required Arguments

Always validate input before running automation.

```bash
#!/bin/bash

if [ $# -lt 2 ]; then
  echo "Usage: $0 <environment> <app-name>"
  exit 1
fi

ENVIRONMENT=$1
APP_NAME=$2

echo "Deploying $APP_NAME to $ENVIRONMENT"
```

This prevents accidental or incomplete execution.

Example with allowed values:

```bash
#!/bin/bash

ENVIRONMENT=$1

if [ "$ENVIRONMENT" != "dev" ] && [ "$ENVIRONMENT" != "staging" ] && [ "$ENVIRONMENT" != "prod" ]; then
  echo "Invalid environment. Use: dev, staging, or prod"
  exit 1
fi

echo "Environment selected: $ENVIRONMENT"
```

---

<a id="exit-codes"></a>

## 5. Exit Codes

Every command returns an exit code.

```text
0        Success
Non-zero Failure
```

Check the last command with `$?`:

```bash
#!/bin/bash

systemctl restart nginx

if [ $? -eq 0 ]; then
  echo "Nginx restarted successfully"
else
  echo "Failed to restart Nginx"
  exit 1
fi
```

Better style:

```bash
#!/bin/bash

if systemctl restart nginx; then
  echo "Nginx restarted successfully"
else
  echo "Failed to restart Nginx"
  exit 1
fi
```

Why this matters in DevOps:

- CI/CD pipelines use exit codes to decide if a job passed or failed.
- Monitoring scripts use exit codes to trigger alerts.
- Automation scripts should stop when critical steps fail.

---

<a id="conditions"></a>

## 6. Conditions

Conditions are used to make decisions inside scripts.

### String comparison

```bash
#!/bin/bash

ENVIRONMENT="prod"

if [ "$ENVIRONMENT" = "prod" ]; then
  echo "Production environment"
else
  echo "Non-production environment"
fi
```

### Number comparison

```bash
#!/bin/bash

CPU_USAGE=85

if [ $CPU_USAGE -ge 90 ]; then
  echo "Critical CPU usage"
elif [ $CPU_USAGE -ge 70 ]; then
  echo "Warning: High CPU usage"
else
  echo "CPU usage is normal"
fi
```

Common numeric operators:

```text
-eq  Equal
-ne  Not equal
-gt  Greater than
-ge  Greater than or equal
-lt  Less than
-le  Less than or equal
```

### File checks

```bash
#!/bin/bash

FILE="/var/log/syslog"

if [ -f "$FILE" ]; then
  echo "File exists: $FILE"
else
  echo "File not found: $FILE"
fi
```

Common file checks:

```text
-f  File exists and is a regular file
-d  Directory exists
-r  File is readable
-w  File is writable
-x  File is executable
```

Example checking if a directory exists:

```bash
#!/bin/bash

DIR="/var/log"

if [ -d "$DIR" ]; then
  echo "Directory exists: $DIR"
else
  echo "Directory not found: $DIR"
fi
```

---

<a id="case-statement"></a>

## 7. Case Statement

Use `case` when there are multiple options.

```bash
#!/bin/bash

ACTION=$1
SERVICE=$2

case $ACTION in
  start)
    systemctl start "$SERVICE"
    ;;
  stop)
    systemctl stop "$SERVICE"
    ;;
  restart)
    systemctl restart "$SERVICE"
    ;;
  status)
    systemctl status "$SERVICE"
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status} <service>"
    exit 1
    ;;
esac
```

Run:

```bash
./service-manager.sh restart nginx
```

Common use cases:

- Service management scripts
- Deployment scripts
- Interactive menus
- Start/stop/restart/status workflows

---

<a id="loops"></a>

## 8. Loops

Loops repeat tasks.

### For loop

```bash
#!/bin/bash

for SERVER in web01 web02 web03; do
  echo "Checking server: $SERVER"
done
```

### Loop through files

```bash
#!/bin/bash

for FILE in /var/log/*.log; do
  echo "Processing $FILE"
done
```

### While loop

```bash
#!/bin/bash

COUNT=1

while [ $COUNT -le 5 ]; do
  echo "Attempt $COUNT"
  COUNT=$((COUNT + 1))
done
```

### Infinite monitoring loop

```bash
#!/bin/bash

while true; do
  date
  uptime
  sleep 10
done
```

Use infinite loops carefully.

### Break example

`break` exits a loop immediately.

```bash
#!/bin/bash

for NUMBER in 1 2 3 4 5; do
  if [ $NUMBER -eq 3 ]; then
    break
  fi

  echo "$NUMBER"
done
```

### Continue example

`continue` skips the current iteration and moves to the next one.

```bash
#!/bin/bash

for NUMBER in 1 2 3 4 5; do
  if [ $NUMBER -eq 3 ]; then
    continue
  fi

  echo "$NUMBER"
done
```

---

<a id="functions"></a>

## 9. Functions

Functions help organize reusable logic.

```bash
#!/bin/bash

log_info() {
  echo "[INFO] $1"
}

log_error() {
  echo "[ERROR] $1"
}

log_info "Starting deployment"
log_error "Deployment failed"
```

Example with service check:

```bash
#!/bin/bash

check_service() {
  SERVICE=$1

  if systemctl is-active --quiet "$SERVICE"; then
    echo "$SERVICE is running"
  else
    echo "$SERVICE is not running"
  fi
}

check_service nginx
check_service ssh
```

Why functions are useful:

- Avoid repeating the same code.
- Make scripts easier to read.
- Make troubleshooting logic reusable.
- Help organize larger automation scripts.

---

<a id="reading-user-input"></a>

## 10. Reading User Input

Use `read` for interactive scripts.

```bash
#!/bin/bash

read -p "Enter service name: " SERVICE
systemctl status "$SERVICE"
```

Confirmation example:

```bash
#!/bin/bash

read -p "Are you sure you want to restart nginx? yes/no: " ANSWER

if [ "$ANSWER" = "yes" ]; then
  systemctl restart nginx
else
  echo "Cancelled"
fi
```

Useful for:

- Confirmation before restarting services
- Asking for an environment name
- Asking for a filename
- Asking for a deployment version

---

<a id="command-substitution"></a>

## 11. Command Substitution

Command substitution stores command output in a variable.

```bash
#!/bin/bash

CURRENT_DATE=$(date)
HOST=$(hostname)
UPTIME=$(uptime -p)

echo "Date: $CURRENT_DATE"
echo "Host: $HOST"
echo "Uptime: $UPTIME"
```

Example:

```bash
FAILED_LOGINS=$(grep "Failed password" /var/log/auth.log | wc -l)
echo "Failed SSH logins: $FAILED_LOGINS"
```

Preferred syntax:

```bash
VALUE=$(command)
```

Older syntax:

```bash
VALUE=`command`
```

The `$(command)` syntax is preferred because it is easier to read.

---

<a id="input-and-output-redirection"></a>

## 12. Input and Output Redirection

Redirection sends output, errors, or input to/from files.

Overwrite a file:

```bash
echo "Starting deployment" > deploy.log
```

Append to a file:

```bash
echo "Deployment completed" >> deploy.log
```

Redirect errors:

```bash
ls /wrong/path 2> error.log
```

Redirect output and errors:

```bash
command > output.log 2>&1
```

Using `tee` to show output and save it:

```bash
kubectl get pods -A | tee pods-output.txt
```

Append with `tee`:

```bash
kubectl get events -A | tee -a troubleshooting.log
```

Useful for:

- Saving deployment logs
- Capturing command output
- Capturing errors
- Creating troubleshooting reports

---

<a id="pipes"></a>

## 13. Pipes

Pipes send the output of one command to another command.

```bash
ps aux | grep nginx
```

Count failed SSH logins:

```bash
grep "Failed password" /var/log/auth.log | wc -l
```

Find most common IPs in a log:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

Find errors in application logs:

```bash
grep -i "error" app.log | tail -n 50
```

Useful for:

- Filtering output
- Counting results
- Log analysis
- Troubleshooting
- Creating quick reports from command output

---

<a id="useful-script-safety-options"></a>

## 14. Useful Script Safety Options

These options make scripts safer.

```bash
#!/bin/bash
set -euo pipefail
```

Meaning:

```text
set -e          Exit if a command fails
set -u          Exit if using an undefined variable
set -o pipefail Fail if any command in a pipe fails
```

Example:

```bash
#!/bin/bash
set -euo pipefail

echo "Starting backup"
tar -czf backup.tar.gz /important/data
echo "Backup completed"
```

Use this carefully because the script exits immediately when something fails.

Recommended for:

- CI/CD scripts
- Backup scripts
- Deployment validation scripts
- Automation where failures should stop execution

---

<a id="logging-pattern"></a>

## 15. Logging Pattern

A simple logging pattern makes scripts easier to troubleshoot.

```bash
#!/bin/bash

LOG_FILE="script.log"

log() {
  echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

log "Script started"
log "Checking disk usage"
df -h | tee -a "$LOG_FILE"
log "Script completed"
```

Why this is useful:

- Creates timestamped logs.
- Helps with troubleshooting.
- Keeps evidence of what the script did.
- Useful for automation and support tasks.

---

<a id="basic-devops-script-examples"></a>

## 16. Basic DevOps Script Examples

### Example 1: Service Health Check

```bash
#!/bin/bash

SERVICE=$1

if [ $# -ne 1 ]; then
  echo "Usage: $0 <service-name>"
  exit 1
fi

if systemctl is-active --quiet "$SERVICE"; then
  echo "$SERVICE is running"
else
  echo "$SERVICE is not running"
  exit 1
fi
```

Run:

```bash
./check-service.sh nginx
```

---

### Example 2: Disk Usage Alert

```bash
#!/bin/bash

THRESHOLD=80
USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ "$USAGE" -ge "$THRESHOLD" ]; then
  echo "WARNING: Disk usage is ${USAGE}%"
else
  echo "Disk usage is normal: ${USAGE}%"
fi
```

---

### Example 3: Check if a Website is Up

```bash
#!/bin/bash

URL=$1

if [ $# -ne 1 ]; then
  echo "Usage: $0 <url>"
  exit 1
fi

if curl -s --fail "$URL" > /dev/null; then
  echo "Site is reachable: $URL"
else
  echo "Site is down or unreachable: $URL"
  exit 1
fi
```

Run:

```bash
./check-url.sh https://example.com
```

---

### Example 4: Collect Linux Troubleshooting Information

```bash
#!/bin/bash

OUTPUT="system-report-$(date +%F-%H%M%S).log"

{
  echo "===== HOSTNAME ====="
  hostname

  echo "===== UPTIME ====="
  uptime

  echo "===== DISK USAGE ====="
  df -h

  echo "===== MEMORY ====="
  free -h

  echo "===== TOP CPU PROCESSES ====="
  ps aux --sort=-%cpu | head

  echo "===== TOP MEMORY PROCESSES ====="
  ps aux --sort=-%mem | head

  echo "===== LISTENING PORTS ====="
  ss -tulpn
} | tee "$OUTPUT"

echo "Report saved to $OUTPUT"
```

---

### Example 5: Search Errors in Logs

```bash
#!/bin/bash

LOG_FILE=$1

if [ $# -ne 1 ]; then
  echo "Usage: $0 <log-file>"
  exit 1
fi

if [ ! -f "$LOG_FILE" ]; then
  echo "File not found: $LOG_FILE"
  exit 1
fi

echo "Last 50 errors from $LOG_FILE"
grep -i "error" "$LOG_FILE" | tail -n 50
```

---

### Example 6: Kubernetes Pod Quick Check

```bash
#!/bin/bash

NAMESPACE=${1:-default}

echo "Checking pods in namespace: $NAMESPACE"

kubectl get pods -n "$NAMESPACE" -o wide

echo "Recent events:"
kubectl get events -n "$NAMESPACE" --sort-by=.metadata.creationTimestamp | tail -n 20
```

Run:

```bash
./k8s-check.sh default
./k8s-check.sh production
```

---

### Example 7: Kubernetes Logs Collector

```bash
#!/bin/bash

NAMESPACE=$1
POD=$2

if [ $# -ne 2 ]; then
  echo "Usage: $0 <namespace> <pod-name>"
  exit 1
fi

OUTPUT="${POD}-logs-$(date +%F-%H%M%S).log"

kubectl logs "$POD" -n "$NAMESPACE" | tee "$OUTPUT"

echo "Logs saved to $OUTPUT"
```

---

### Example 8: Restart a Kubernetes Deployment

```bash
#!/bin/bash

NAMESPACE=$1
DEPLOYMENT=$2

if [ $# -ne 2 ]; then
  echo "Usage: $0 <namespace> <deployment-name>"
  exit 1
fi

echo "Restarting deployment $DEPLOYMENT in namespace $NAMESPACE"

kubectl rollout restart deployment "$DEPLOYMENT" -n "$NAMESPACE"
kubectl rollout status deployment "$DEPLOYMENT" -n "$NAMESPACE"
```

---

### Example 9: Simple Backup Script

```bash
#!/bin/bash

SOURCE_DIR=$1
BACKUP_DIR=$2

if [ $# -ne 2 ]; then
  echo "Usage: $0 <source-dir> <backup-dir>"
  exit 1
fi

TIMESTAMP=$(date +%F-%H%M%S)
BACKUP_FILE="backup-$TIMESTAMP.tar.gz"

tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"

echo "Backup created: $BACKUP_DIR/$BACKUP_FILE"
```

---

### Example 10: Deployment Validation Script

```bash
#!/bin/bash

APP_URL=$1
EXPECTED_STATUS=200

if [ $# -ne 1 ]; then
  echo "Usage: $0 <app-url>"
  exit 1
fi

STATUS_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$APP_URL")

if [ "$STATUS_CODE" -eq "$EXPECTED_STATUS" ]; then
  echo "Deployment validation passed. HTTP $STATUS_CODE"
else
  echo "Deployment validation failed. HTTP $STATUS_CODE"
  exit 1
fi
```

---

<a id="bash-scripting-best-practices"></a>

## 17. Bash Scripting Best Practices

- Always validate input arguments.
- Use quotes around variables.
- Use meaningful variable names.
- Use functions for repeated logic.
- Use exit codes properly.
- Avoid hardcoding secrets.
- Write logs for important actions.
- Test scripts in a safe environment first.
- Be careful with destructive commands like `rm`, `dd`, and `mkfs`.
- Use `set -euo pipefail` when appropriate.
- Add usage instructions to scripts.
- Keep scripts simple and readable.

---
