#CLAUDE-CASCADIA-SETUP.md (Windows)

(Do not modify this file)

 ## Step 1: Read only step --

 Key points from the Cascadia README:

- This repo is for Cascadia, which generates context documents (Requirements.md, Strategy.md, Architecture.md, and System_Instructions.md -- filenames are in uppercase) for Claude code to build node.js applications. It is meant to work with the cascadia-harness-node applications, and writes to context-docs/workflow/ directory in the cascadia-harness-node, and also reads an architecture_template.md which will be dropped into a templates directory. 
- Default build uses `ANTHROPIC_API_KEY`.
- Requires a `.env` file in the project directory with:
```
ANTHROPIC_API_KEY=
PORT=3001
ENVIRONMENT=LOCAL_CONSOLE
ANTHROPIC_MODEL=claude-opus-4-8
TARGET_WORKFLOW=
TARGET_TEMPLATE=
```
- Per the README setup (§3), `TARGET_WORKFLOW` is the absolute path to
  `<project>/context-docs/workflow` and `TARGET_TEMPLATE` is the absolute path
  to `<project>/context-docs/template`.
- Node setup: install Node 24 via NodeSource, `npm init`, `npm install`.
- Smoke tests documented: `node src/smoke-test-db.js` (SQLite check),
  `node src/smoke-test-sdk.js` (Anthropic SDK check), `node src/server.js`
  (starts web server on port 3001, with `/health` route).

No commands need to be run — this step is read-only.

## Step 2: Generate .env file in cascadia

`.env` files are not included in the repository (per `.gitignore`), so
generate the file from scratch.

Create `cascadia/.env` with the key/value scaffold specified
in the README (leave values blank/defaulted, to be filled in next):

```
ANTHROPIC_API_KEY=
PORT=3001
ENVIRONMENT=LOCAL_CONSOLE
ANTHROPIC_MODEL=claude-opus-4-8
```

Note: the `ANTHROPIC_API_KEY` value is unique per developer and must
never be logged in the ClaudeLog.txt log file or displayed in chat.

Note: The user can also optionally edit the .env file manually to change the anthropic_model. 


## Step 3: STOP — request ANTHROPIC_API_KEY (MANDATORY BLOCKING GATE)

⛔ DO NOT SKIP. DO NOT DEFER. DO NOT BUNDLE. DO NOT PROCEED WITHOUT IT. ⛔

This step MUST be fully completed before ANY later step runs. It is a hard stop,
not a background task. Do not run Step 4 — or anything after it — until the key
has been received from the developer and written to the .env file.

STOP here and send the developer ONE standalone chat message whose ONLY content
is a request for their ANTHROPIC_API_KEY. Do not combine it with Node checks,
npm output, status tables, smoke tests, or any other step's work. That request
must be the entire message.

Then WAIT for the developer's reply. Do not continue other steps "in the
meantime," do not guess a value, do not invent a placeholder.

Note: Alert the user that they can also optionally edit the .env file manually to enter the anthropic_key if they don't want to supply it in chat.

When the developer supplies the key, write it verbatim into
`cascadia/.env` as the value of `ANTHROPIC_API_KEY=`. Never echo the
value back in chat and never write it to ClaudeLog.txt.

Confirm the key is present in `.env`. ONLY THEN continue to Step 4.


## Step 4: Setup Cascadia context docs and templates in Cascadia Harness application 

- Cascadia Template source directory is`cascadia/TARGET/template-library-ga/`.
- Harness Workflow directory is `cascadia-harness-node/context-docs/workflow/`. 
- Harness Template directory is `cascadia-harness-node/context-docs/template/`, to be created next. 

A. Make new directory 'template' in `cascadia-harness-node/context-docs/` if that is not present.

B. Ask user which template they want to install in Harness Template directory: "generic web app" or "agentic web app". Present them with two options. If they select "generic web app", copy ARCHITECTURE_TEMPLATE_GENERIC.md from Cascadia Template Source Directory to Harness Template directory and rename file ARCHITECTURE_TEMPLATE.md. If they select "agentic web app", copy ARCHITECTURE_TEMPLATE_AGENTIC_HARNESS.md from Cascadia Template Source Directory to Harness Template directory and rename file ARCHITECTURE_TEMPLATE.md. 

C. If user selected "agentic web app", ask user if their agents require guardrails. If they select yes, copy GUARDRAIL_SPEC.md from Cascadia Template Source Directory to Harness Template directory. 

## Step 5: Add Cascadia Console target workflow and template directories to .env file

`TARGET_WORKFLOW` and `TARGET_TEMPLATE` must each be a fully-resolved ABSOLUTE
path. `dotenv` does not expand `~`, `$HOME`, or shell variables, so a literal
absolute path is required. Do not copy the example below — every participant's
path differs. Derive each path at runtime from the actual harness location.

Windows note (slash direction): on Windows `path.resolve` returns BACKSLASH
paths (e.g. `C:\Users\...\workflow`). Written into `.env` those are risky —
if a value is double-quoted, `dotenv` expands `\n`/`\r`, so a folder segment
beginning with `n` or `r` (e.g. `...\node\...`) is silently corrupted into a
newline. To avoid this entirely, normalize to FORWARD slashes and write the
value UNQUOTED. Node's `fs`/`path` accept forward slashes on Windows
everywhere, so `C:/Users/.../workflow` works exactly the same.

A. Compute the two absolute paths, normalized to forward slashes. Run these
   from the directory that contains both cloned repos (`path.resolve` is
   cross-platform; the `.replace(/\\/g,'/')` converts the Windows backslashes):

   ```powershell
   node -e "console.log(require('path').resolve('cascadia-harness-node/context-docs/workflow').replace(/\\/g,'/'))"
   node -e "console.log(require('path').resolve('cascadia-harness-node/context-docs/template').replace(/\\/g,'/'))"
   ```

B. Append the two computed paths to `cascadia/.env`, using the exact output of
   each command above. Write them UNQUOTED (no surrounding quotes):

   ```
   TARGET_WORKFLOW=<absolute path printed by the first command>
   TARGET_TEMPLATE=<absolute path printed by the second command>
   ```

Illustrative only (a Windows workstation — yours will differ; do not paste):
   TARGET_WORKFLOW=C:/Users/<user>/claude-desktop-projects/ai-camp/cascadia-workshop/windows/cascadia-harness-node/context-docs/workflow
   TARGET_TEMPLATE=C:/Users/<user>/claude-desktop-projects/ai-camp/cascadia-workshop/windows/cascadia-harness-node/context-docs/template

C. Validate the target directories and workflow docs before proceeding. Confirm
   both TARGET directories exist and that all four workflow documents
   (`REQUIREMENTS.md`, `STRATEGY.md`, `ARCHITECTURE.md`, `SYSTEM_INSTRUCTIONS.md`)
   are readable in the workflow path. This uses only built-in Node modules, so
   it runs before `npm install`:

   ```powershell
   node -e "
   const fs=require('fs'), path=require('path');
   const wf=path.resolve('cascadia-harness-node/context-docs/workflow');
   const tp=path.resolve('cascadia-harness-node/context-docs/template');
   for (const d of [wf, tp]) {
     if (!fs.existsSync(d) || !fs.statSync(d).isDirectory())
       { console.error('MISSING DIR: '+d); process.exit(1); }
     console.log('DIR OK: '+d);
   }
   for (const f of ['REQUIREMENTS.md','STRATEGY.md','ARCHITECTURE.md','SYSTEM_INSTRUCTIONS.md']) {
     const p=path.join(wf, f);
     try { fs.accessSync(p, fs.constants.R_OK); console.log('READABLE: '+p); }
     catch { console.error('NOT READABLE: '+p); process.exit(1); }
   }
   console.log('Step 5 validation passed.');
   "
   ```

   Expect a `DIR OK` line for each directory, a `READABLE` line for each of the
   four docs, and a final `Step 5 validation passed.` If any line reports
   `MISSING DIR` or `NOT READABLE`, stop and report it — do not continue to
   Step 6, because the Console server hard-exits at boot when `TARGET_WORKFLOW`
   is missing or unreadable.


## Step 6: Check for Node.js; install if missing

Check whether Node.js and npm are already installed before installing:

```powershell
node --version
npm --version
```

If Node.js is present, confirm it is Node 24 (the version the README
requires) and skip installation. If it is missing, install Node 24 on
Windows with winget (fully scripted, no GUI). Node 24 is the current LTS
line, so install the LTS package.

Pin `--source winget` (Node is on the winget community source, not the
Microsoft Store) and pass `--disable-interactivity` so winget never blocks
on a prompt — without these, winget stops to ask you to accept the `msstore`
source agreement and the install fails in an automated run:

```powershell
winget install --id OpenJS.NodeJS.LTS -e --source winget --silent `
  --accept-source-agreements --accept-package-agreements --disable-interactivity
```

Fallback only if winget is unavailable (rare on Windows 10/11): resolve the
latest Node 24 x64 MSI and install it silently.

```powershell
$base = "https://nodejs.org/dist/latest-v24.x/"
$msiName = (Invoke-WebRequest $base -UseBasicParsing).Links.href |
  Where-Object { $_ -match '^node-v24.*-x64\.msi$' } | Select-Object -First 1
Invoke-WebRequest -Uri ($base + $msiName) -OutFile "$env:TEMP\$msiName"
Start-Process msiexec.exe -ArgumentList "/i `"$env:TEMP\$msiName`" /qn" -Wait
```

After installing, the Windows installer updates the machine PATH, but
Claude Desktop (and every shell it spawns) still holds the environment it
was launched with, so `node` will not be found. The refresh below fixes
only the single shell it runs in — it does NOT carry over to the next
command Claude Desktop runs, because each command is a fresh shell spawned
from Claude Desktop's stale environment. The reliable fix is to **restart
Claude Desktop** after installing Node, so every subsequent shell inherits
the updated PATH.

To confirm the install in the current shell (one-off check only):

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" +
            [System.Environment]::GetEnvironmentVariable("Path","User")
node --version
npm --version
```

Which case applies:

- If Node 24 was already installed before Claude Desktop was launched (the
  intended prerequisite state), Claude Desktop already inherited the correct
  PATH and no restart is needed — the verify check above will simply pass.
- If this script had to install Node just now (attendee arrived without a
  working Node install), then `node`/`npm` will NOT be visible to subsequent
  steps until Claude Desktop is restarted. Tell the attendee to fully quit
  and reopen Claude Desktop, then re-run this setup from Step 6 — the verify
  check will now find Node and skip the install.

Report to the developer in chat whether Node.js was already present or had
to be installed, along with the detected version.

## Step 7: Verify Node.js install and report to developer

Verify the Node.js install and report the version output to the developer
in chat:

```powershell
node --version
npm --version
```

Report the resulting `node` and `npm` versions to the developer in chat,
and confirm the major version matches the README requirement (Node 24).

## Step 8: Install Node dependencies

From `cascadia/`, install the project dependencies before
running any smoke tests (the smoke tests import the Anthropic SDK and
SQLite, which are not present until this runs):

```powershell
npm install
```

## Step 9: Verify dependency install and report to developer

Verify `node_modules` was created and the key dependencies resolved:

```powershell
if (Test-Path node_modules) { "present" }
foreach ($d in @("@anthropic-ai/sdk","better-sqlite3","express","dotenv")) {
  if (Test-Path "node_modules/$d") { "$d OK" } else { "$d MISSING" }
}
```

Report the result to the developer in chat, confirming success (all key
dependencies resolved) or naming any dependency that failed to install.

Note (native module): `better-sqlite3` is a native addon that compiles a
`.node` binary during its install script. If npm is configured to skip
install scripts (some machines print `npm warn allow-scripts` and do NOT
run them), the binary is missing and the SQLite smoke test in Step 11 will
fail. Confirm the binary exists:

```powershell
Test-Path "node_modules/better-sqlite3/build/Release/better_sqlite3.node"
```

If this returns `False`, rebuild it explicitly before Step 11:

```powershell
npm rebuild better-sqlite3
```
 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 
## Step 10: Fix dependency vulnerabilities

If `npm install` reports vulnerabilities, review them and apply the
available fixes:

```powershell
npm audit
npm audit fix
```

Re-run `npm audit` afterward to confirm the count reaches `found 0
vulnerabilities`. Report the before/after result to the developer in chat.
If any vulnerability cannot be resolved without a breaking change (i.e.
requires `npm audit fix --force`), stop and report it rather than forcing
the change.

## Step 11: Run smoke tests and report to developer

Run the three smoke tests documented in the README, from
`cascadia/`. Example (illustrative only) output for each is
in the README; compare against it.

A. SQLite check:

   ```powershell
   node src/smoke-test-db.js
   ```

   Expect `SQLite OK: { k: 'hello' }`.

B. Anthropic SDK check (exercises the API key from `.env`):

   ```powershell
   node src/smoke-test-sdk.js
   ```

   Expect a live model response plus a `Usage: { ... }` token summary. A
   failure here usually means the `ANTHROPIC_API_KEY` is missing or invalid.

C. Web server check. On Windows/PowerShell there is no `&` background
   operator and `curl` is an alias for `Invoke-WebRequest`, so start the
   server as a background process with `Start-Process` (capturing its output
   to a log so the startup line can be checked), poll until it answers the
   `/health` route, then stop it with `Stop-Process`:

   ```powershell
   $log = "$env:TEMP\console-server.log"
   $srv = Start-Process node -ArgumentList "src/server.js" -PassThru `
     -RedirectStandardOutput $log -RedirectStandardError "$env:TEMP\console-server.err" -WindowStyle Hidden

   # Wait for the server to come up (poll /health for up to ~10s)
   for ($i = 0; $i -lt 10; $i++) {
     Start-Sleep -Seconds 1
     try { Invoke-RestMethod http://localhost:3001/health -TimeoutSec 2 | Out-Null; break } catch {}
   }

   Get-Content $log                                                   # startup line
   Invoke-RestMethod http://localhost:3001/health | ConvertTo-Json -Compress

   Stop-Process -Id $srv.Id -Force                                    # stop the server
   ```

   Expect `listening on port 3001 [LOCAL_CONSOLE]` in the captured log, and
   `/health` -> `{"status":"ok","uptime":<n>,"version":"2.0.0"}`.
   `Stop-Process` shuts the background server down.

Report pass/fail for each of the three tests to the developer in chat.



## Step 12: Start Cascadia Console

Start the Console server so it keeps running (unlike the Step 11C check, do
not stop it here — this launch is meant to stay up for the session). On
Linux this is a foreground `node src/server.js`, but under Claude Desktop on
Windows a foreground server would block the shell indefinitely. Instead
launch it **detached** with `Start-Process` (it keeps running independently
of this shell and of Claude Desktop), capture its output to a log, and poll
until it answers. From `cascadia/`:

```powershell
$log = "$env:TEMP\cascadia-console.log"
$console = Start-Process node -ArgumentList "src/server.js" -PassThru `
  -RedirectStandardOutput $log -RedirectStandardError "$env:TEMP\cascadia-console.err" -WindowStyle Hidden

# Wait for the Console to come up (poll /health for up to ~10s)
for ($i = 0; $i -lt 10; $i++) {
  Start-Sleep -Seconds 1
  try { Invoke-RestMethod http://localhost:3001/health -TimeoutSec 2 | Out-Null; break } catch {}
}

Get-Content $log                                                     # startup line
"Console PID: $($console.Id)"                                        # note this to stop it later
```

Wait for the startup line `listening on port 3001 [LOCAL_CONSOLE]` in the
log, then confirm to the user in chat that the Cascadia Console is up and
reachable at:

   http://localhost:3001

To stop the Console later, use the PID printed above:

```powershell
Stop-Process -Id <Console PID> -Force
```

If the server exits immediately instead of printing the startup line, report
the error output to the user. The most common causes are a missing or invalid
`.env` value (`ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL`) or a `TARGET_WORKFLOW`
that is missing or not a readable directory (revisit Step 5, including the
Step 5C validation).

 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 

