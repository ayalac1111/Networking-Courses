# 26S-CST8371 — Introduction to Enterprise Networking

## Repository Status

This directory is the working course structure for the **Spring 2026** delivery of **CST8371 — Introduction to Enterprise Networking**.

This repo section is used to organize student-facing labs, challenge/audit files, examples, and visual references.

---

## Directory Structure

```text
26s-cst8371/
├── challenges
├── examples
├── infographics
└── labs
```

---

## Directory Purpose

| Directory | Purpose |
|---|---|
| `challenges/` | Apply / audit / troubleshooting challenge instructions |
| `examples/` | Sample outputs, reference configs, command examples, and instructor-provided examples |
| `infographics/` | Visual references used by labs, weekly pages, or lab-book support |
| `labs/` | Student-facing Practice lab instructions |

---

## Naming Convention

Use sequence-based names instead of week-bound names.

```text
l01-activity-name.md
l02-activity-name.md
a01-audit-name.md
```

Recommended examples:

```text
labs/l01-be-ready-alpine-tool-readiness.md
labs/l02-ipv4-management-baseline.md
challenges/a02-ipv4-evidence-audit.md
examples/l02-sample-evidence.txt
infographics/lab-book-template.png
```

---

## Submission Filename Pattern

Student submissions should use predictable filenames.

```text
lNN-shortname-<username>.txt
```

Examples:

```text
l01-be-ready-ayalac.txt
l02-ipv4-management-baseline-ayalac.txt
```

---

## Evidence Standard

Every checked or graded activity should define:

```text
Action
Evidence
Success Indicator
Failure Signal
```

Minimum evidence block:

```text
Device / Platform:
Command:
Target:
Exact evidence line:
What this proves:
What this does not prove:
```

---

## Lab Book Standard

Each lab should identify what students should record in their lab book.

Minimum lab-book prompts:

```text
Topic:
Topology / device notes:
Command map:
Expected good states:
Common failure signals:
Proof line:
Minimal fix:
Reusable rule:
```

Reusable rule format:

```text
Next time I see __________________, I will run __________________ first because __________________.
```
