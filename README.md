# 🔧 sre-toolkit

A collection of opinionated SRE tools for Kubernetes troubleshooting, backup validation, and deployment verification.

Built for real-world incident response — not demos.

## Tools

| Tool | Description |
|------|-------------|
| [`kdiag`](#kdiag) | One-command Kubernetes cluster diagnosis — finds broken pods, misconfigured services, and resource pressure |
| [`backup-lint`](#backup-lint) | Static analysis for backup scripts — catches common pg_dump, cron, and permission bugs before they bite |
| [`deploy-check`](#deploy-check) | Post-deployment verification — confirms rollout health, endpoint reachability, and rollback readiness |
| [`pod-rx`](#pod-rx) | Interactive pod prescription engine — reads symptoms, suggests fixes, applies them |
| [`py-diagnose`](#py-diagnose) | Python web app diagnostic — finds syntax errors, Django/Flask misconfig, DB bugs, Dockerfile issues |

## Install

```bash
git clone https://github.com/bcwilsondotcom/sre-toolkit.git
cd sre-toolkit
chmod +x bin/*
export PATH="$PWD/bin:$PATH"
```

Or copy individual tools:
```bash
curl -sL https://raw.githubusercontent.com/bcwilsondotcom/sre-toolkit/main/bin/kdiag -o /usr/local/bin/kdiag
chmod +x /usr/local/bin/kdiag
```

## Requirements

- `kubectl` (for kdiag, deploy-check, pod-rx)
- `docker` (for deploy-check)
- `bash` 4+ or `zsh`
- Standard coreutils (`grep`, `awk`, `sed`, `find`)

---

## kdiag

Full cluster health check in one command. Finds the problems, explains why, suggests fixes.

```bash
kdiag                    # scan default namespace
kdiag -n production      # scan specific namespace
kdiag -A                 # scan all namespaces
kdiag --json             # machine-readable output
```

**What it checks:**
- Pod status anomalies (CrashLoopBackOff, ImagePullBackOff, Pending, OOMKilled)
- Restart storms (pods restarting frequently)
- Resource pressure (CPU/memory requests vs limits vs actual)
- Service→Pod selector mismatches
- Dangling ConfigMaps/Secrets (referenced but missing)
- Recent warning events
- Node conditions (disk pressure, memory pressure, PID pressure)

---

## backup-lint

Static analyzer for backup scripts. Catches the bugs that cause 3 AM pages.

```bash
backup-lint backup.sh           # lint a bash backup script
backup-lint backup.py           # lint a python backup script
backup-lint --fix backup.sh     # suggest inline fixes
```

**What it catches:**
- Missing `set -euo pipefail` (bash)
- Unquoted variables in paths
- pg_dump without error checking
- Missing `mkdir -p` for backup directories
- Wrong `.pgpass` permissions (must be 600)
- Missing compression
- No retention/cleanup policy
- Broken cron expressions
- Missing shebang lines
- Hardcoded credentials (suggests env vars)

---

## deploy-check

Post-deployment smoke test. Run after `kubectl apply` to confirm everything actually works.

```bash
deploy-check <deployment-name>              # verify in default ns
deploy-check <deployment-name> -n staging   # specific namespace
deploy-check --rollback-on-fail <name>      # auto-rollback if checks fail
```

**What it verifies:**
- Rollout completed successfully
- All pods in Ready state
- No restart loops in last 60s
- Service endpoints populated
- HTTP health check passes (if endpoint detected)
- Previous revision available for rollback

---

## pod-rx

Interactive troubleshooter. Point it at a sick pod, get a diagnosis and prescription.

```bash
pod-rx <pod-name>           # diagnose a specific pod
pod-rx --worst              # auto-find the most broken pod
pod-rx --watch              # continuous monitoring mode
```

---

## py-diagnose

Find bugs in Python web apps before they bite. Built for Django, Flask, and FastAPI.

```bash
py-diagnose /app               # scan app directory
py-diagnose /srv/webapp --deep  # deep scan (test imports, check DB, verify migrations)
```

**What it catches:**
- Syntax errors (compile-time check on every .py file)
- Missing imports and requirements gaps
- Django: empty ALLOWED_HOSTS (→ 400), wrong DB host/port, placeholder passwords, missing migrations
- Flask: binding to 127.0.0.1 (unreachable in Docker), missing config
- Dockerfile issues: missing WORKDIR, port mismatches, no pip install
- Python 2 vs 3 issues (print statements)
- Tab/space mixing

---

## Philosophy

These tools exist because the first 5 minutes of an incident matter most. Instead of remembering 15 kubectl commands and piping them together under pressure, run one tool and get answers.

Every tool follows three rules:
1. **Show your work** — print what you're checking and why
2. **Suggest, don't just report** — every finding includes a recommended fix
3. **Exit codes matter** — 0 = healthy, 1 = issues found, 2 = critical

## License

MIT
