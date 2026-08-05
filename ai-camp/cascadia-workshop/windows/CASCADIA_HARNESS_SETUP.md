# CASCADIA_HARNESS_SETUP.md (Windows)

(Do not modify this file)

 ## Step 1: Read only step --

 Key points from the Cascadia Harness Node README:

- This repo is a base harness for building Cascadia node.js applications.
  It's meant to be cloned, copied, and renamed for starting each new
  Cascadia node.js project.
- Default build uses `ANTHROPIC_API_KEY`.
- Requires a `.env` file in the project directory with:
  - `ANTHROPIC_API_KEY=`
  - `PORT=3000`
  - `ENVIRONMENT=DEVELOPMENT`
- Node setup: install Node 24 via NodeSource, `npm init`, `npm install`.
- Smoke tests documented: `node src/smoke-test-db.js` (SQLite check),
  `node src/smoke-test-sdk.js` (Anthropic SDK check), `node src/server.js`
  (starts web server on port 3000, with `/health` and `/` routes).

No commands need to be run — this step is read-only.

## Step 2: Generate .env file in cascadia-harness-node

`.env` files are not included in the repository (per `.gitignore`), so
generate the file from scratch.

Create `cascadia-harness-node/.env` with the key/value scaffold specified
in the README (leave values blank/defaulted, to be filled in next):

```
ANTHROPIC_API_KEY=
PORT=3000
ENVIRONMENT=DEVELOPMENT
```

Note: the `ANTHROPIC_API_KEY` value is unique per developer and must
never be logged in the ClaudeLog.txt log file or displayed in chat.

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
`cascadia-harness-node/.env` as the value of `ANTHROPIC_API_KEY=`. Never echo the
value back in chat and never write it to ClaudeLog.txt.

Confirm the key is present in `.env`. ONLY THEN continue to Step 4.

## Step 4: Check for Node.js; install if missing

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
  and reopen Claude Desktop, then re-run this setup from Step 4 — the verify
  check will now find Node and skip the install.

Report to the developer in chat whether Node.js was already present or had
to be installed, along with the detected version.

## Step 5: Verify Node.js install and report to developer

Verify the Node.js install and report the version output to the developer
in chat:

```powershell
node --version
npm --version
```

Report the resulting `node` and `npm` versions to the developer in chat,
and confirm the major version matches the README requirement (Node 24).

## Step 6: Install Node dependencies

From `cascadia-harness-node/`, install the project dependencies before
running any smoke tests (the smoke tests import the Anthropic SDK and
SQLite, which are not present until this runs):

```powershell
npm install
```

## Step 7: Verify dependency install and report to developer

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
run them), the binary is missing and the SQLite smoke test in Step 9 will
fail. Confirm the binary exists:

```powershell
Test-Path "node_modules/better-sqlite3/build/Release/better_sqlite3.node"
```

If this returns `False`, rebuild it explicitly before Step 9:

```powershell
npm rebuild better-sqlite3
```

## Step 8: Fix dependency vulnerabilities

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

## Step 9: Run smoke tests and report to developer

Run the three smoke tests documented in the README, from
`cascadia-harness-node/`. Example (illustrative only) output for each is
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
   to a log so the startup line can be checked), poll until it answers, hit
   both routes, then stop it with `Stop-Process`. Use `Invoke-RestMethod` for
   the JSON `/health` route, but use `Invoke-WebRequest -UseBasicParsing` and
   read `.Content` for the `/` route — its response is `text/html`, and
   `Invoke-RestMethod` would parse it into an object instead of returning the
   raw text:

   ```powershell
   $log = "$env:TEMP\harness-server.log"
   $srv = Start-Process node -ArgumentList "src/server.js" -PassThru `
     -RedirectStandardOutput $log -RedirectStandardError "$env:TEMP\harness-server.err" -WindowStyle Hidden

   # Wait for the server to come up (poll /health for up to ~10s)
   for ($i = 0; $i -lt 10; $i++) {
     Start-Sleep -Seconds 1
     try { Invoke-RestMethod http://localhost:3000/health -TimeoutSec 2 | Out-Null; break } catch {}
   }

   Get-Content $log                                                   # startup line
   Invoke-RestMethod http://localhost:3000/health | ConvertTo-Json -Compress
   (Invoke-WebRequest http://localhost:3000/ -UseBasicParsing).Content

   Stop-Process -Id $srv.Id -Force                                    # stop the server
   ```

   Expect `listening on port 3000 [DEVELOPMENT]` in the captured log,
   `/health` -> `{"status":"ok","uptime":<n>,"version":"1.0.0"}`, and the `/`
   body to contain `HELLO WORLD` (the harness wraps it as
   `<html> … HELLO WORLD … </html>`). `Stop-Process` shuts the server down.

Report pass/fail for each of the three tests to the developer in chat.

## Step 10: Cascadia Harness Setup Complete. 

 Next: Read Claude_Cascadia_Master_Setup.md, Step 3. 
 
 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 
