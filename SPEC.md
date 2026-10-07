# Handoff summary: apps built with Claude, and where things stand

Paste this whole file into another AI (or attach it) and say: "Continue this project from this summary."
The other AI cannot see the files made in the original chat, so keep the downloaded files (project.b64, build.yml, SETUP.txt) and give them to the new AI if it needs the exact code.

## About the user and working style
- Uses only an Android phone (no computer). Everything must be doable from the phone browser.
- Wants everything free. Prefers offline and copy/paste solutions over paid services or cloud databases.
- Explicitly said: give ALL details first, then build. Do not build more until the user says "that's everything".
- Currency is Ugandan shillings (UGX). Timezone is East Africa Time (UTC+3).

## Project 1: Agent Hub (agents that run scripted browser automation)
Goal: an app that hosts "agents" in groups, shows what each agent is doing live, and runs them in sequence, all at once, or one at a time.
Key facts and decisions:
- Claude cannot create AI agents with internet or login/click ability inside a normal web artifact. A published web page cannot control other sites.
- First version: a published web page (Claude artifact) with groups, agents from uploaded files, live status, run modes. Agents there only produce text (no web access).
- Chrome extension version (zip): free, no AI, "scripted agents". Chrome on Android does not support extensions, so it was not usable on the phone.
- Final approach: an Android APK containing its own WebView browser (same engine as Chrome), built free in the cloud with GitHub Actions, no computer needed.
- Agents are plain-text scripts, one step per line: open URL, click (CSS selector or button text), type SELECTOR = text, enter, wait ms, waitfor SELECTOR, scroll px, extract SELECTOR, pause message ({prev} passes text from the previous agent). Lines starting with # are comments.
- Features built: groups, named agents, upload/paste scripts, run modes, per-agent live status and activity log, progress bar, View browser button, Stop, reorder, and a step editor. Editor includes "Show me" teach mode: the user performs a step in the page and the app records taps/typing and replaces that script line (passwords are never recorded; a pause step is inserted instead).
- Known fix made: JS confirm()/alert() dialogs did not work in the WebView until native dialog handlers were added.
- Icon: the user's own photo was cropped into the launcher icon.
- Not built / declined: an agent that mass-creates Gmail accounts with random identities. Claude declined (fake accounts, terms violation).
- A free local-AI (Ollama) agent option was discussed but not built.

## The GitHub build process used (works, first build was green)
1. Create a GitHub repo (the user made a private one called Agents-hub).
2. Upload `project.b64` (a base64-encoded tar.gz of the whole Android project) with Add file > Upload files.
3. Create `.github/workflows/build.yml` and paste the workflow. GitHub Actions unpacks the project, builds with Gradle 8.9 / Java 17 / AGP 8.5.2 / Kotlin 1.9.24, and publishes `app-debug.apk` in Releases under tag "latest".
4. Download the APK from Releases and install it (allow unknown sources; Play Protect may warn).
5. To update the app: upload a new project.b64 with the same name; the build reruns.
The project uses a fixed keystore inside the project so new APKs install over old ones. Version code comes from the GitHub run number.
Workflow content (build.yml):
```
name: Build APK
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Unpack project
        run: base64 -d project.b64 | tar xz
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
      - name: Accept Android licenses
        run: yes | "$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager" --licenses > /dev/null || true
      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'
      - name: Build
        working-directory: project
        run: gradle assembleDebug -PvCode=${{ github.run_number }} --no-daemon
      - name: Publish APK
        uses: softprops/action-gh-release@v2
        with:
          tag_name: latest
          name: Latest build
          files: project/app/build/outputs/apk/debug/app-debug.apk
```
Architecture of these apps: a thin Kotlin Android shell (MainActivity with WebViews and a JavaScript bridge named Android) plus the whole UI as one `assets/index.html` file. State is saved in the WebView's localStorage.
Note: the Claude chat could not compile Android code. The first GitHub build of Agent Hub succeeded; later changes (teach mode) and the payout app were not compiled before delivery, so build errors may need fixing from the GitHub Actions log.

## Project 2: Staff payout app for the "CT team" (CURRENT, still in requirements stage)
The user manages workers. Workers complete tasks, prove it, and get paid. The user pays workers in UGX.
Requirements given so far:
- Team name: "CT team". One APK with a manager mode (user) and a worker mode.
- Manager adds workers, creates tasks, approves or rejects completions, sees payouts.
- Each task has: a pay amount in UGX (added to the worker's expected payout for the week), an example video shown as a YouTube or TikTok link, and a screenshot upload for proof.
- Some tasks are daily (renew each day); some are one-time. When all daily tasks are done the worker waits until the next day.
- Some tasks need extra verification: a text box next to the task where the worker pastes a transaction reference together with the screenshot.
- Sync without any server: copy/paste or file-based. Worker app produces a signed text code (report); manager pastes many at once; manager app produces signed update codes per worker (new tasks, approvals, pay). New tasks reach workers by an update file from the manager, or by a whole app update (new project.b64 build). Codes are signed with a per-worker secret key (HMAC-SHA256, pure JS) so edited codes are rejected; task IDs and sequence numbers stop double counting and replays. Join codes (contain the worker key) are sent privately to each worker.
- Weekly: the worker app builds a file of all transactions and completed tasks and sends it to the manager on Telegram. The user wants a link to their Telegram account. Options explained: share button (worker picks Telegram) or a free Telegram bot for automatic sending (token stored in the app; set up with @BotFather). The user said yes to the bot. The user's Telegram username has NOT been given yet.
- Payout week assumed Monday to Sunday (not yet confirmed).

Screenshot proof checks (already implemented in the draft build):
- Worker app records for each screenshot: SHA-256 file hash, perceptual hash (dHash), size, image dimensions, file name time, file modified time, editing-software tags, device screen size, time of claim. These go in the signed report.
- Worker sends the original screenshot file to the manager as a Document (not a photo, because chat apps recompress).
- Manager app compares the received file with the signed hash: identical, or changed (edited or recompressed); flags reused screenshots (same or very similar image across tasks), editing-software tags, screenshots taken before the task was created or in the future, mismatch with the worker's phone screen size, non-PNG files. Shows a verdict: Looks OK / Check carefully / Do not trust, with reasons; the manager can still approve manually.
- Honest limit: no tool can prove a screenshot is genuine; a worker could edit before submitting without leaving tags.

Money verification (decided design, NOT yet built):
- Workers send BNB from the "CT pool" to the manager's NC Wallet. The transfers are internal to the wallet platform (not on the blockchain), so there is no transaction hash to look up.
- The user has no transaction IDs. NC Wallet's PDF "Current account statement" is a text table with columns: number, coin name, date and time (UTC), and amount (for example "+0.00000822 BNB"). It has no sender or reference. The wallet app's History screen shows the same transfers 3 hours later (local East Africa time), so the app must convert everything to UTC.
- The worker types the exact BNB amount and time (with the phone's time zone recorded) next to the task.
- Planned manager "Verify" feature: pick the PDF statement; the app extracts rows and ignores the user's name and user ID; only BNB rows are used for matching but all rows are kept in a ledger; strict one-to-one matching (each statement row verifies one claim, each claim uses one row) by coin, exact amount, and time within about 2 minutes; ambiguous cases flagged for the manager; the ledger remembers verified rows, claims with no matching transfer, and transfers with no claim, so overlapping statements never double count and later statements can match older unmatched claims. Only verified tasks add to payout (manager can override).
- Extra automation ideas offered: read wallet notifications on the manager phone (Android notification access, keeping only coin/amount/time), and OCR of a History screenshot. Not yet decided.
- Other verification ideas offered: app-made reference codes if a note field exists, screen recordings for high-value tasks, random spot checks, worker trust levels.

## Files produced in the chat
- Agent Hub web page (published artifact), scripted-browser-agents.zip (Chrome extension), Android project for Agent Hub (project.b64), build.yml, SETUP.txt, icon preview.
- Staff payout draft: payouts/project.b64 (manager and worker modes, signed copy/paste sync, screenshot proof checks, team name). It has NOT been uploaded to GitHub; the user asked to wait until all requirements are given. If uploaded, use a new, separate repo so Agent Hub is not overwritten.

## Still to decide or do
1. User says "that's everything", then build the full payout app with: task fields (daily/one-time, pay in UGX, YouTube/TikTok link, optional reference/amount/time box), daily wait logic, weekly Monday to Sunday expected payout, weekly report file, Telegram link and bot, PDF statement verification with ledger, any notification/OCR automation.
2. Get the user's Telegram username and decide bot vs share button.
3. Ask whether NC Wallet sends notifications on arrival, and whether a worker's wallet shows any reference after sending.
4. Test builds on the real phone and fix any compile errors from the GitHub Actions log.
