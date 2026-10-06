# Day 3 — Linux Process & Service Management

## 1. Goal

Learn how to inspect Linux processes and services
and establish a basic incident investigation workflow.

---

## 2. Environment

- OS: Ubuntu 26.04.1 LTS
- Environment: WSL
- Project: OpsLab

---

## 3. Process Inspection

### ps

```bash
ps
Displays processes associated with the current terminal.

ps aux
ps aux

Displays running processes and resource information.

Important fields:

USER: process owner
PID: process ID
%CPU: CPU usage
%MEM: memory usage
STAT: process state
COMMAND: executed command
4. Resource Investigation
Find high CPU processes
ps aux --sort=-%cpu | head
Find high memory processes
ps aux --sort=-%mem | head

These commands can be used as an initial step
when investigating performance problems.

5. PID Investigation

Current shell PID:

echo $$

Process information:

ps -p $$ -f

Process relationship:

ps -p $$ -o pid,ppid,stat,cmd
PID: current process ID
PPID: parent process ID
6. Process Tree
pstree -p

A process tree helps identify parent-child relationships
between processes.

7. Service Management

Check running services:

systemctl list-units --type=service --state=running

Check a specific service:

systemctl status ssh

The SSH service may not exist in the current WSL environment.

This is an environment-specific difference rather than necessarily an error.

8. System Logs

View recent logs:

journalctl -n 20

View recent 50 log entries:

journalctl --no-pager -n 50

View logs from the last 10 minutes:

journalctl --since "10 minutes ago"
9. Incident Investigation Workflow

Example:

The server suddenly becomes slow.

Investigation flow:

Performance symptom
        ↓
top
        ↓
ps aux --sort=-%cpu
        ↓
Identify suspicious PID
        ↓
ps -p PID -f
        ↓
Check process state / parent process
        ↓
journalctl
        ↓
Investigate logs
        ↓
Identify cause
10. What I Learned
PID

A PID identifies a running process.

PPID

PPID identifies the parent process.

Process

A running instance of a program.

Service

A long-running system process managed by the operating system,
often through systemd.

journalctl

A command used to inspect logs collected by systemd's journal.

11. Operational Perspective

The important lesson from Day 3 is not memorizing commands.

The important lesson is building an investigation sequence:

Symptom
→ Process
→ PID
→ Resource usage
→ Process relationship
→ Service
→ Logs
→ Root cause

This workflow will later be connected to
Nginx, Docker, PostgreSQL, Prometheus, Grafana,
and incident response exercises in OpsLab.


