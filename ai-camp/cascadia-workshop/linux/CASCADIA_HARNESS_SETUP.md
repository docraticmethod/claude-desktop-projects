# CASCADIA_HARNESS_SETUP.md

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

## Step 5: Verify Node.js install and report to developer

Verify the Node.js install and report the version output to the developer
in chat:

```bash
node --version
npm --version
```

Report the resulting `node` and `npm` versions to the developer in chat,
and confirm the major version matches the README requirement (Node 24).

## Step 6: Install Node dependencies

From `cascadia-harness-node/`, install the project dependencies before
running any smoke tests (the smoke tests import the Anthropic SDK and
SQLite, which are not present until this runs):

```bash
npm install
```

## Step 7: Verify dependency install and report to developer

Verify `node_modules` was created and the key dependencies resolved:

```bash
ls node_modules >/dev/null 2>&1 && echo present
for d in @anthropic-ai/sdk better-sqlite3 express dotenv; do
  [ -d "node_modules/$d" ] && echo "$d OK" || echo "$d MISSING"
done
```

Report the result to the developer in chat, confirming success (all key
dependencies resolved) or naming any dependency that failed to install.

## Step 8: Fix dependency vulnerabilities

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

## Step 9: Run smoke tests and report to developer

Run the three smoke tests documented in the README, from
`cascadia-harness-node/`. Example (illustrative only) output for each is
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
   the background), then hit both routes and stop it:

   ```bash
   node src/server.js &
   curl -s http://localhost:3000/health
   curl -s http://localhost:3000/
   ```

   Expect `listening on port 3000 [DEVELOPMENT]` on startup,
   `/health` -> `{"status":"ok","uptime":<n>,"version":"1.0.0"}`, and
   `/` -> `HELLO WORLD`. Stop the background server once verified.

Report pass/fail for each of the three tests to the developer in chat.

## Step 10: Cascadia Harness Setup Complete. 

 Next: Read Claude_Cascadia_Master_Setup.md, Step 3. 
 
 ## copyright footer: do not edit this section # 
Instructions for AI to automate installation and set up of Cascadia Console with Cascadia Harness.
Copyright Blake Rogers 2026 All Rights Reserved.
## end do not edit this section # 
