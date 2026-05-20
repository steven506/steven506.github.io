# Linux Troubleshooting Commands for DevOps and Support

This page is a practical Linux troubleshooting reference for DevOps, Cloud Support, SRE, Infrastructure, and Technical Support work.

The goal is to document common Linux commands, troubleshooting workflows, and investigation patterns used to diagnose real server issues.

---

## Table of Contents

- [1. System Information](#system-information)
- [2. CPU Troubleshooting](#cpu-troubleshooting)
- [3. Memory Troubleshooting](#memory-troubleshooting)
- [4. Disk Usage Troubleshooting](#disk-usage-troubleshooting)
- [5. Storage and Partitions](#storage-and-partitions)
- [6. Process Troubleshooting](#process-troubleshooting)
- [7. Service Troubleshooting](#service-troubleshooting)
- [8. Logs and Journalctl](#logs-and-journalctl)
- [9. Network Interfaces](#network-interfaces)
- [10. Connectivity Testing](#connectivity-testing)
- [11. DNS Troubleshooting](#dns-troubleshooting)
- [12. Ports and Listening Services](#ports-and-listening-services)
- [13. Firewall Checks](#firewall-checks)
- [14. Permissions and Ownership](#permissions-and-ownership)
- [15. File Search and Disk Cleanup](#file-search-and-disk-cleanup)
- [16. Package Management](#package-management)
- [17. Users and Groups](#users-and-groups)
- [18. SSH Troubleshooting](#ssh-troubleshooting)
- [19. Performance Quick Checks](#performance-quick-checks)
- [20. Real Troubleshooting Scenarios](#real-troubleshooting-scenarios)

---

<a id="system-information"></a>

## 1. System Information

Use these commands to understand the server, OS version, hostname, uptime, kernel, and current user context.

```bash
hostname
hostnamectl
uname -a
uptime
whoami
id
date
```

Check OS release:

```bash
cat /etc/os-release
```

Check kernel version:

```bash
uname -r
```

Check system architecture:

```bash
uname -m
```

Useful when:

- Joining a troubleshooting bridge
- Validating which server you are on
- Checking OS compatibility
- Confirming uptime after a reboot
- Collecting basic evidence for a ticket

---

<a id="cpu-troubleshooting"></a>

## 2. CPU Troubleshooting

Use these commands when the server is slow or CPU usage is high.

### Real-time CPU usage

```bash
top
```

If installed:

```bash
htop
```

### Check load average

```bash
uptime
```

Example output:

```text
10:30:15 up 5 days, 2:10, 2 users, load average: 1.20, 2.10, 3.50
```

The load average shows system demand over:

```text
1 minute, 5 minutes, 15 minutes
```

### Find top CPU-consuming processes

```bash
ps aux --sort=-%cpu | head
```

### Show CPU information

```bash
lscpu
```

### Troubleshooting logic

- Check if the CPU issue is current or historical.
- Compare load average with the number of CPU cores.
- Identify the top CPU-consuming process.
- Check if the process is expected or abnormal.
- Review application logs if the process belongs to an application.
- Check if a recent deployment, cron job, backup, or batch process started.

---

<a id="memory-troubleshooting"></a>

## 3. Memory Troubleshooting

Use these commands when the server is slow, applications are crashing, or the system may be running out of memory.

### Check memory usage

```bash
free -h
```

### Real-time memory usage

```bash
top
```

### Find top memory-consuming processes

```bash
ps aux --sort=-%mem | head
```

### Check virtual memory statistics

```bash
vmstat 1
```

### Check swap usage

```bash
swapon --show
free -h
```

### Check OOM killer events

```bash
dmesg | grep -i "out of memory"
dmesg | grep -i "killed process"
```

Or with journalctl:

```bash
journalctl -k | grep -i "out of memory"
journalctl -k | grep -i "killed process"
```

### Troubleshooting logic

- Check total memory, used memory, free memory, and available memory.
- Identify top memory-consuming processes.
- Check if swap is being heavily used.
- Look for OOM killer events.
- Check application logs for memory errors.
- Validate if a memory leak may be happening.

---

<a id="disk-usage-troubleshooting"></a>

## 4. Disk Usage Troubleshooting

Use these commands when the disk is full or applications cannot write files.

### Check filesystem usage

```bash
df -h
```

### Check inode usage

```bash
df -i
```

A disk can fail because of:

```text
Storage full
Inodes full
Read-only filesystem
Permission issues
Large log files
```

### Check largest folders in current directory

```bash
du -sh * | sort -h
```

### Check largest folders under root

```bash
sudo du -xh / | sort -h | tail -n 20
```

### Find large files

```bash
sudo find / -type f -size +500M 2>/dev/null
```

### Find recently modified large files

```bash
sudo find / -type f -mtime -1 -size +100M 2>/dev/null
```

### Troubleshooting logic

- Run `df -h` to identify the full filesystem.
- Run `df -i` to check inode exhaustion.
- Use `du` to find the largest directories.
- Use `find` to locate large files.
- Check logs under `/var/log`.
- Check application directories.
- Avoid deleting files without understanding their purpose.

---

<a id="storage-and-partitions"></a>

## 5. Storage and Partitions

Use these commands to inspect disks, mounts, partitions, and block devices.

### List block devices

```bash
lsblk
```

### Show mounted filesystems

```bash
mount
```

More readable:

```bash
findmnt
```

### Check disk partitions

```bash
sudo fdisk -l
```

### Check filesystem type

```bash
df -Th
```

### Check UUIDs

```bash
blkid
```

### Check persistent mounts

```bash
cat /etc/fstab
```

### Troubleshooting logic

- Confirm if the disk is visible with `lsblk`.
- Confirm if the partition is mounted.
- Check filesystem type.
- Validate `/etc/fstab` if the disk should mount after reboot.
- Check for read-only mounts.
- Be careful with partition commands in production.

---

<a id="process-troubleshooting"></a>

## 6. Process Troubleshooting

Use these commands to inspect running processes.

### Show all processes

```bash
ps aux
```

### Find a process by name

```bash
ps aux | grep nginx
```

Better:

```bash
pgrep -a nginx
```

### Show process tree

```bash
pstree
```

If not installed:

```bash
ps -ef --forest
```

### Kill a process

```bash
kill <PID>
```

Force kill:

```bash
kill -9 <PID>
```

Use `kill -9` only when normal termination does not work.

### Troubleshooting logic

- Identify the process.
- Check CPU and memory usage.
- Check who started it.
- Check how long it has been running.
- Check related service with `systemctl`.
- Avoid killing critical processes without validation.

---

<a id="service-troubleshooting"></a>

## 7. Service Troubleshooting

Use these commands when a Linux service is down, unhealthy, or failing to start.

### Check service status

```bash
systemctl status nginx
```

### Start, stop, restart service

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
```

### Enable service at boot

```bash
sudo systemctl enable nginx
```

### Disable service at boot

```bash
sudo systemctl disable nginx
```

### Check if service is active

```bash
systemctl is-active nginx
```

### Check if service is enabled

```bash
systemctl is-enabled nginx
```

### View service logs

```bash
journalctl -u nginx
```

Follow logs live:

```bash
journalctl -u nginx -f
```

View recent logs:

```bash
journalctl -u nginx --since "1 hour ago"
```

### Troubleshooting logic

- Check service status.
- Review recent logs.
- Validate configuration files.
- Check if the port is already in use.
- Check permissions.
- Check recent changes.
- Restart only if safe and approved.

---

<a id="logs-and-journalctl"></a>

## 8. Logs and Journalctl

Logs are one of the most important sources during troubleshooting.

### Common log locations

```text
/var/log/syslog
/var/log/messages
/var/log/auth.log
/var/log/secure
/var/log/nginx/
/var/log/httpd/
/var/log/audit/
```

### View logs live

```bash
tail -f /var/log/syslog
```

For RHEL/CentOS:

```bash
tail -f /var/log/messages
```

### Search errors

```bash
grep -i "error" /var/log/syslog
```

### Search failed logins

Ubuntu/Debian:

```bash
grep -i "failed password" /var/log/auth.log
```

RHEL/CentOS:

```bash
grep -i "failed password" /var/log/secure
```

### Journal logs

```bash
journalctl
```

Follow logs live:

```bash
journalctl -f
```

Kernel logs:

```bash
journalctl -k
```

Logs since one hour ago:

```bash
journalctl --since "1 hour ago"
```

Logs for a service:

```bash
journalctl -u nginx
```

### Troubleshooting logic

- Check logs around the time the issue started.
- Search for errors, failures, denied, timeout, refused, and permission messages.
- Compare application logs with system logs.
- Check if the issue started after a restart or deployment.

---

<a id="network-interfaces"></a>

## 9. Network Interfaces

Use these commands to inspect network interfaces and routes.

### Show IP addresses

```bash
ip a
```

### Show routes

```bash
ip route
```

### Show interface statistics

```bash
ip -s link
```

### Show DNS resolver config

```bash
cat /etc/resolv.conf
```

### Troubleshooting logic

- Confirm the server has the expected IP address.
- Confirm the interface is up.
- Check the default gateway.
- Check DNS resolver configuration.
- Compare network config with expected environment.

---

<a id="connectivity-testing"></a>

## 10. Connectivity Testing

Use these commands to test if the server can reach another host or service.

### Ping test

```bash
ping 8.8.8.8
```

### Test HTTP/HTTPS connectivity

```bash
curl -v http://example.com
curl -I https://example.com
```

### Test a specific port with netcat

```bash
nc -vz example.com 443
```

If `nc` is not installed, use:

```bash
telnet example.com 443
```

### Trace network path

```bash
traceroute example.com
```

If using systems with `tracepath`:

```bash
tracepath example.com
```

### Troubleshooting logic

- Test by IP first to separate network issues from DNS issues.
- Test DNS resolution.
- Test the target port.
- Use curl for HTTP-level troubleshooting.
- Use traceroute or tracepath for routing issues.

---

<a id="dns-troubleshooting"></a>

## 11. DNS Troubleshooting

Use these commands when hostname resolution is failing.

### Resolve a domain

```bash
dig example.com
```

Alternative:

```bash
nslookup example.com
```

### Query a specific DNS server

```bash
dig @8.8.8.8 example.com
```

### Check DNS config

```bash
cat /etc/resolv.conf
```

### Check local hosts file

```bash
cat /etc/hosts
```

### Troubleshooting logic

- Check if the domain resolves.
- Query a public DNS server to compare results.
- Check `/etc/resolv.conf`.
- Check `/etc/hosts` for overrides.
- Confirm if the issue is DNS or connectivity.

---

<a id="ports-and-listening-services"></a>

## 12. Ports and Listening Services

Use these commands when an application is not reachable or a port conflict is suspected.

### Show listening TCP/UDP ports

```bash
ss -tulpn
```

### Show listening TCP ports

```bash
ss -tlpn
```

### Find what is using a port

```bash
sudo lsof -i :8080
```

Alternative:

```bash
sudo ss -tulpn | grep 8080
```

### Test local application port

```bash
curl -v http://localhost:8080
```

### Troubleshooting logic

- Confirm the application is listening.
- Confirm it is listening on the correct IP and port.
- Check if another process is using the port.
- Test locally before testing remotely.
- Check firewall rules if local works but remote fails.

---

<a id="firewall-checks"></a>

## 13. Firewall Checks

Firewall commands depend on the Linux distribution.

### UFW - Ubuntu

Check status:

```bash
sudo ufw status verbose
```

Allow a port:

```bash
sudo ufw allow 80/tcp
```

### firewalld - RHEL/CentOS

Check status:

```bash
sudo firewall-cmd --state
```

List rules:

```bash
sudo firewall-cmd --list-all
```

Allow a port:

```bash
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```

### iptables

List rules:

```bash
sudo iptables -L -n -v
```

### Troubleshooting logic

- Confirm the application is listening locally.
- Check if the firewall allows the port.
- Check cloud security groups or network ACLs if running in AWS/Azure.
- Check corporate firewall or VPN paths if relevant.

---

<a id="permissions-and-ownership"></a>

## 14. Permissions and Ownership

Use these commands when seeing `Permission denied` errors.

### Show permissions

```bash
ls -l
```

### Show hidden files too

```bash
ls -la
```

### Change permissions

```bash
chmod 644 file.txt
chmod 755 script.sh
chmod +x script.sh
```

### Change owner

```bash
sudo chown user:user file.txt
```

### Change owner recursively

```bash
sudo chown -R user:user /path/to/directory
```

### Troubleshooting logic

- Check file ownership.
- Check file permissions.
- Check directory permissions.
- Check which user is running the process.
- Avoid using `chmod 777` as a quick fix.

---

<a id="file-search-and-disk-cleanup"></a>

## 15. File Search and Disk Cleanup

Use these commands to find files, clean logs, and investigate disk usage.

### Find files by name

```bash
find /var/log -name "*.log"
```

### Find files by size

```bash
find / -type f -size +500M 2>/dev/null
```

### Find files modified in the last day

```bash
find . -type f -mtime -1
```

### Find files by extension

```bash
find . -type f -name "*.yaml"
```

### Clean package cache

Ubuntu/Debian:

```bash
sudo apt clean
```

RHEL/CentOS:

```bash
sudo yum clean all
```

### Truncate a large log file

```bash
sudo truncate -s 0 /var/log/app.log
```

Use with caution.

### Troubleshooting logic

- Identify what is consuming space.
- Confirm whether the file can be deleted, archived, or truncated.
- Avoid deleting active files without checking the application.
- Consider log rotation.

---

<a id="package-management"></a>

## 16. Package Management

Package commands depend on the Linux distribution.

### Ubuntu/Debian

Update package index:

```bash
sudo apt update
```

Install package:

```bash
sudo apt install nginx
```

Remove package:

```bash
sudo apt remove nginx
```

Search package:

```bash
apt search nginx
```

### RHEL/CentOS

Install package:

```bash
sudo yum install nginx
```

Remove package:

```bash
sudo yum remove nginx
```

For newer systems:

```bash
sudo dnf install nginx
sudo dnf remove nginx
```

### Troubleshooting logic

- Confirm the OS family.
- Check if the package exists in enabled repositories.
- Check network/DNS if package installation fails.
- Check repository configuration.
- Review package manager logs.

---

<a id="users-and-groups"></a>

## 17. Users and Groups

Use these commands for access and permission troubleshooting.

### Current user

```bash
whoami
id
```

### List logged-in users

```bash
who
w
```

### Add user

```bash
sudo useradd username
```

### Change password

```bash
sudo passwd username
```

### Add user to a group

```bash
sudo usermod -aG groupname username
```

### Check groups

```bash
groups username
```

### Troubleshooting logic

- Confirm the user exists.
- Confirm the user belongs to the required group.
- Check file and directory permissions.
- Check sudo access if needed.
- Check login shell and account status.

---

<a id="ssh-troubleshooting"></a>

## 18. SSH Troubleshooting

Use these commands when you cannot connect to a server over SSH.

### Basic SSH

```bash
ssh user@server
```

### Verbose SSH debug

```bash
ssh -v user@server
```

More verbose:

```bash
ssh -vvv user@server
```

### Check SSH service

```bash
systemctl status ssh
```

On some systems:

```bash
systemctl status sshd
```

### Check SSH port

```bash
ss -tulpn | grep ssh
```

### Check SSH logs

Ubuntu/Debian:

```bash
sudo tail -f /var/log/auth.log
```

RHEL/CentOS:

```bash
sudo tail -f /var/log/secure
```

### Troubleshooting logic

- Confirm the server is reachable.
- Confirm port 22 or custom SSH port is open.
- Check username.
- Check key permissions.
- Check SSH service status.
- Check firewall and cloud security groups.
- Review SSH logs.

---

<a id="performance-quick-checks"></a>

## 19. Performance Quick Checks

Use this quick command set when a server is slow.

```bash
hostname
uptime
free -h
df -h
df -i
top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
ss -tulpn
journalctl --since "1 hour ago" | tail -n 100
```

Quick investigation logic:

```text
1. Confirm the server and time.
2. Check CPU and load average.
3. Check memory and swap.
4. Check disk and inodes.
5. Check top processes.
6. Check listening ports.
7. Check recent system logs.
8. Check application logs.
9. Check recent changes.
10. Document findings.
```

---

<a id="real-troubleshooting-scenarios"></a>

## 20. Real Troubleshooting Scenarios

### Scenario 1: Server is slow

Commands:

```bash
uptime
top
free -h
df -h
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

What to check:

- High CPU usage
- High memory usage
- Swap usage
- Full disk
- Heavy application process
- Backup or cron job running

Possible fixes:

- Restart unhealthy application service if approved.
- Stop unnecessary heavy process if safe.
- Scale resources if needed.
- Review application logs.
- Investigate recent changes.

---

### Scenario 2: Disk is full

Commands:

```bash
df -h
df -i
du -sh * | sort -h
sudo find / -type f -size +500M 2>/dev/null
```

What to check:

- Full filesystem
- Inode exhaustion
- Large logs
- Large temporary files
- Failed log rotation

Possible fixes:

- Rotate or compress logs.
- Delete confirmed temporary files.
- Truncate approved log files.
- Increase disk size.
- Fix log rotation.

---

### Scenario 3: Service is down

Commands:

```bash
systemctl status nginx
journalctl -u nginx --since "1 hour ago"
ss -tulpn | grep nginx
```

What to check:

- Service status
- Recent errors
- Port conflicts
- Configuration issues
- Permission issues

Possible fixes:

- Fix configuration.
- Restart the service if approved.
- Free the required port.
- Restore missing files.
- Roll back recent changes.

---

### Scenario 4: Application is not listening on expected port

Commands:

```bash
ss -tulpn
sudo lsof -i :8080
curl -v http://localhost:8080
systemctl status app-service
```

What to check:

- App is running
- App is bound to the correct port
- Port conflict
- Firewall rules
- Application startup errors

Possible fixes:

- Correct app config.
- Restart application.
- Change port.
- Update firewall rules.
- Fix environment variables.

---

### Scenario 5: DNS resolution is failing

Commands:

```bash
dig example.com
nslookup example.com
cat /etc/resolv.conf
cat /etc/hosts
```

What to check:

- DNS server configuration
- Local hosts file override
- DNS timeout
- Wrong domain record
- Internal DNS issue

Possible fixes:

- Correct resolver config.
- Fix `/etc/hosts`.
- Use correct DNS server.
- Escalate to DNS/network team if needed.

---

### Scenario 6: Cannot SSH into server

Commands:

```bash
ping server
nc -vz server 22
ssh -vvv user@server
systemctl status ssh
sudo tail -f /var/log/auth.log
```

What to check:

- Network reachability
- SSH service status
- Firewall rules
- Security groups
- Username/key problems
- Failed login logs

Possible fixes:

- Start SSH service.
- Correct firewall/security group.
- Fix key permissions.
- Use correct username.
- Restore access through console if needed.

---

### Scenario 7: High memory usage

Commands:

```bash
free -h
top
ps aux --sort=-%mem | head
swapon --show
dmesg | grep -i "killed process"
```

What to check:

- Top memory process
- Swap usage
- OOM killer events
- Application memory leak
- Recent deployments

Possible fixes:

- Restart leaking process if approved.
- Increase memory.
- Tune application memory settings.
- Investigate application logs.
- Roll back recent deployment if related.

---

### Scenario 8: Permission denied error

Commands:

```bash
ls -l
ls -la
whoami
id
namei -l /path/to/file
```

What to check:

- File permissions
- Directory permissions
- File ownership
- User running the process
- Group membership

Possible fixes:

- Correct ownership with `chown`.
- Correct permissions with `chmod`.
- Add user to required group.
- Avoid using `chmod 777`.

---
