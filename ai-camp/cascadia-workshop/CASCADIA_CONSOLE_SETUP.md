#CLAUDE-CASCADIA-SETUP.md

(Do not modify this file)

## Step 1: Read cascadia README

Read `cascadia/README.md` to understand Cascadia setup.

Key points from the README:

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
never be logged in this file.

Note: The user can edit the .env file manually to change the anthropic_model, or request Claude Code to do so.

## Step 3: Fill in ANTHROPIC_API_KEY in cascadia/.env

Request the `ANTHROPIC_API_KEY` value from the developer via chat.

The developer will supply it as input after you request it; write the value into
`cascadia/.env` as the value for `ANTHROPIC_API_KEY=`


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

A. Compute the two absolute paths. Run these from the directory that contains
   both cloned repos (Node is guaranteed present by this step, and
   `path.resolve` is cross-platform):

   ```bash
   node -e "console.log(require('path').resolve('cascadia-harness-node/context-docs/workflow'))"
   node -e "console.log(require('path').resolve('cascadia-harness-node/context-docs/template'))"
   ```

B. Append the two computed paths to `cascadia/.env`, using the exact output of
   each command above:

   ```
   TARGET_WORKFLOW=<absolute path printed by the first command>
   TARGET_TEMPLATE=<absolute path printed by the second command>
   ```

Illustrative only (a Linux workstation — yours will differ; do not paste):
   TARGET_WORKFLOW=/home/<user>/claude-desktop-projects/ai-camp/cascadia-harness-node/context-docs/workflow
   TARGET_TEMPLATE=/home/<user>/claude-desktop-projects/ai-camp/cascadia-harness-node/context-docs/template

C. Validate the target directories and workflow docs before proceeding. Confirm
   both TARGET directories exist and that all four workflow documents
   (`REQUIREMENTS.md`, `STRATEGY.md`, `ARCHITECTURE.md`, `SYSTEM_INSTRUCTIONS.md`)
   are readable in the workflow path. This uses only built-in Node modules, so
   it runs before `npm install`:

   ```bash
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

```bash
node --version
npm --version
```

If Node.js is present, confirm it is Node 24 (the version the README
requires) and skip installation. If it is missing, install Node 24 via
NodeSource per the README:

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs
```

Report to the developer in chat whether Node.js was already present or had
to be installed, along with the detected version.

## Step 7: Verify Node.js install and report to developer

Verify the Node.js install and report the version output to the developer
in chat:

```bash
node --version
npm --version
```

Report the resulting `node` and `npm` versions to the developer in chat,
and confirm the major version matches the README requirement (Node 24).

## Step 8: Install Node dependencies

From `cascadia/`, install the project dependencies before
running any smoke tests (the smoke tests import the Anthropic SDK and
SQLite, which are not present until this runs):

```bash
npm install
```

## Step 9: Verify dependency install and report to developer

Verify `node_modules` was created and the key dependencies resolved:

```bash
ls node_modules >/dev/null 2>&1 && echo present
for d in @anthropic-ai/sdk better-sqlite3 express dotenv; do
  [ -d "node_modules/$d" ] && echo "$d OK" || echo "$d MISSING"
done
```

Report the result to the developer in chat, confirming success (all key
dependencies resolved) or naming any dependency that failed to install.
 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 
## Step 10: Fix dependency vulnerabilities

If `npm install` reports vulnerabilities, review them and apply the
available fixes:

```bash
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

   ```bash
   node src/smoke-test-db.js
   ```

   Expect `SQLite OK: { k: 'hello' }`.

B. Anthropic SDK check (exercises the API key from `.env`):

   ```bash
   node src/smoke-test-sdk.js
   ```

   Expect a live model response plus a `Usage: { ... }` token summary. A
   failure here usually means the `ANTHROPIC_API_KEY` is missing or invalid.

C. Web server check. Start the server (it runs until stopped, so run it in
   the background), then hit health route and stop it:

   ```bash
   node src/server.js &
   curl -s http://localhost:3001/health
   ```

   Expect `listening on port 3001 [LOCAL_CONSOLE]` on startup,
   `/health` -> `{"status":"ok","uptime":<n>,"version":"2.0.0"}`. Stop the background server once verified.

Report pass/fail for each of the three tests to the developer in chat.



## Step 12: Start Cascadia Console

Start the Console server so it keeps running (unlike the Step 11C check, do
not background-and-kill it here — this launch is meant to stay up for the
session). From `cascadia/`:

```bash
node src/server.js
```

Wait for the startup line `listening on port 3001 [LOCAL_CONSOLE]`, then
confirm to the user in chat that the Cascadia Console is up and reachable at:

   http://localhost:3001

If the server exits immediately instead of printing the startup line, report
the error output to the user. The most common causes are a missing or invalid
`.env` value (`ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL`) or a `TARGET_WORKFLOW`
that is missing or not a readable directory (revisit Step 5, including the
Step 5C validation).

 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 

