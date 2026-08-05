#CASCADIA_MASTER_SETUP.md (Windows)

## Do not modify this file 

# Cascadia Master Setup Instructions file 

This file is a step-by-step set of system instructions for installing and
configuring Cascadia and Cascadia Harness, written for automated execution.

Follow the steps in order — do not skip ahead. Each step should only be
performed after the prior step is confirmed complete.

## Logging

Log file is ClaudeLog.txt 

Log each completed step to ClaudeLog.txt with a terse one- or two-sentence
summary note. EVERY log entry — the run START marker, each completed step, and
the run END marker — MUST begin with a full timestamp containing BOTH date AND
time of day to the second, in 24-hour local time. Format: `YYYY-MM-DD HH:MM:SS`.

Do NOT hand-type the timestamp and do NOT reuse the date from your context —
your context may carry only a coarse date with no clock, which is why a real
timestamp must be read from the system immediately before writing each entry:

    PowerShell:  Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    bash:        date "+%Y-%m-%d %H:%M:%S"

Write a timestamped START entry before Step 1, a timestamped entry after each
completed step, and a timestamped END entry after the final task.

Include the Time zone. Example: In San Francisco local time zone is Pacific / Los Angeles (PDT/PST). 

## Step 1: Clone the two source repositories

This document runs from inside the `cascadia-workshop/` folder that Claude
Desktop is pointed at — the folder that already contains these three setup
`.md` files. Clone both source repos into this current directory so they sit
alongside the setup files as subdirectories (`./cascadia/` and
`./cascadia-harness-node/`). Every later step references them by relative path
from here, so do not clone them elsewhere.

- `cascadia` — clone from https://github.com/docraticmethod/cascadia
  (node.js web app that generates system instructions for Claude Code)
- `cascadia-harness-node` — clone from https://github.com/docraticmethod/cascadia-harness-node
  (node.js starter app that consumes the generated system instructions)

Commands to run (from the `cascadia-workshop/` directory):

```powershell
git clone https://github.com/docraticmethod/cascadia.git
git clone https://github.com/docraticmethod/cascadia-harness-node.git
```

Confirm both directories are present and populated (each contains its own
`package.json`, `README.md`, `src`, `CLAUDE.md`, etc.). This
`cascadia-workshop/` directory is the "directory that contains both cloned
repos" referenced by later steps (e.g. the absolute-path resolution in the
Console setup Step 5).


## Step 2: Cascadia Harness Setup

Follow instructions in # CASCADIA_HARNESS_SETUP.md

## Step 3: Cascadia Console Setup

Follow instructions in # CASCADIA_CONSOLE_SETUP.md


## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 







