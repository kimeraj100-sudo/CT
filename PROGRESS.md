# CT TEAM — PROGRESS & HANDOFF (read this first)

Full requirements are in `SPEC.md`. This file says what is built, how the repo works, and what to build next. Update it after every piece.

## How the repo works (phone-friendly)
- Repo root holds: `.github/workflows/main.yml`, `SPEC.md`, `PROGRESS.md`, and `piece-NN-name.zip` files.
- Each piece is a ZIP containing real project files with their real paths (e.g. `app/src/main/java/com/ctteam/app/...`).
- The workflow unzips every `piece-*.zip` in sorted order into `build_project/`, then runs `gradle assembleDebug`.
- A later piece can REPLACE a file from an earlier piece by including the same path. Newer zip wins.
- Piece zips contain only NEW or CHANGED files, never the whole project again.
- The APK is signed with a fixed key (`app/ct-team.jks`, in piece 1), so every new APK installs over the old one and keeps local data. Never change the key, `applicationId` (`com.ctteam.app`) or database name.

## Tech decisions (keep consistent)
- Kotlin 1.9.24, Jetpack Compose (compiler 1.5.14, BOM 2024.06.00), Material3, AGP 8.5.2, Gradle 8.9, Java 17, compileSdk/targetSdk 34, minSdk 26.
- Package: `com.ctteam.app`. UI code in `com.ctteam.app.ui`. Palette object: `CT` in `ui/Theme.kt` (Navy, Emerald, Teal, Violet, Gold...).
- Reusable UI: `CTBackground`, `ShimmerLogo`, `GoldButton`, `GlassCard` in `ui/Components.kt`.
- Navigation: simple `Screen` enum in `ui/CTApp.kt` with per-destination transitions. Replace placeholders as pieces land.
- Logo assets: `res/drawable-nodpi/logo_shield.png` (in-app, transparent) and `ic_launcher_foreground.png` (adaptive launcher icon, navy background colour `@color/ct_navy`).
- Local data only (no server). Currency UGX, store money as Long (whole shillings). Store times as UTC epoch millis.

## STATUS
| # | Piece | Status |
|---|-------|--------|
| 1 | Foundation: build, icon, theme, animated splash, role select | DONE (build confirmed green) |
| 2 | Data layer | DONE |
| 3 | Manager: PIN, dashboard, workers, tasks, review | DONE in piece-03-manager.zip (build result: check Actions) |
| 4A | Manager access (20 PINs, code names, main manager) + worker registration/home | DONE in piece-04a-managers-worker.zip (build result: check Actions) |
| 4B | Worker: task list, submit proof, earnings history, payout request | NEXT |
| 5 | Security & sync (HMAC reports/updates) | todo |
| 6 | Screenshot verification | todo |
| 7 | Payouts, phone/name normalization, statement import | todo |
| 8 | NC Wallet PDF / BNB matching | todo |
| 9 | Telegram bot, notifications, support chat | todo |
| 10 | Learn & Earn, announcements | todo |
| 11 | Backup / restore / migration | todo |
| 12 | Premium polish & animations | todo |

## What each piece contains
1. **Foundation** — compiling project, fixed signing key, launcher icon from the approved logo, jewel theme, animated logo opening with gold light-sweep, Manager/Worker choice screen, different transition per button.
2. **Data layer** — Room database (add KSP plugin, version matching Kotlin 1.9.24, e.g. `1.9.24-1.0.20`) with tables: workers (+ phone history), tasks (+ task versions), submissions, proofs/hashes, payout requests, payments, NC statement rows & claims, support messages, announcements, learn content, audit log, events. Repository classes. Rules: phone change once/week and not during payout in progress. Schema export on, migrations always provided so updates never wipe data.
3. **Manager: workers & tasks** — dashboard stats, add/edit workers, create/edit/disable tasks (daily vs one-time, pay, example links, proof requirements), approve/reject submissions with audit log.
4. **Worker** — split in two. 4A: multi-manager PIN access + worker registration and home. 4B: task list with daily renewal, proof submission screen, earnings history, payout request.
5. **Security & sync** — per-worker secret key, HMAC-SHA256 signed reports and updates, task IDs + sequence numbers, replay/duplicate protection, copy/paste and file exchange.
6. **Screenshot verification** — SHA-256, dHash, size, dimensions, EXIF/editor tags, timestamps, duplicate detection → 🟢/🟡/🔴. Manager can always override. UI must say it cannot prove authenticity.
7. **Payouts** — Mon–Sun weeks, request → 🟡 In Progress → ✅ Paid, Uganda phone normalization (0772…, 256772…, +256772…), name normalization, statement import with Strong/Possible/No match, manual confirmation, permanent archive.
8. **NC Wallet / BNB** — import PDF statement, statement time is UTC, user sees EAT (UTC+3), match coin + exact amount + time within ~2 min, strict one-to-one, remember matched/unmatched rows across overlapping statements, manual override.
9. **Telegram** — free bot, worker→manager weekly report/support (documents for original screenshots), manager→worker verification/payout/announcements, support history in app.
10. **Learn & Earn** — content (text, images, video links) published by manager through signed updates, no APK rebuild.
11. **Backup** — encrypted + integrity-checked + versioned export/import (phone storage, Drive, SD, Telegram), emergency migration.
12. **Polish** — extra animations, glass nav bar, hero sections, premium imagery.

## Rules for any AI continuing this
1. Give complete files with exact paths, never snippets.
2. Package each piece as `piece-NN-name.zip` (files at zip root as in the project). Keep pieces small enough to finish in one chat.
3. Do not restart or simplify the concept; if something can't be done yet, leave the architecture for it.
4. After each piece: update this file, and ask the user to paste the GitHub Actions error log if the build fails.
5. Keep honesty in the UI: automated checks flag, the manager decides.

## Piece 2 — Data layer (what exists now)
Files (package `com.ctteam.app.data`, folder `app/src/main/java/com/ctteam/app/data/`):
- `Entities.kt` — all Room tables + status constants (`TaskType`, `SubStatus`, `PayoutStatus`, `Verdict`, `Dir`, `NcStatus`). Tables: workers, phone_history, tasks, task_versions, submissions, proofs, payout_requests, payments, nc_statements, nc_rows, nc_claims, support_messages, announcements, learn_content, audit_log, events, meta, manager_identity.
- `Daos.kt` — one DAO per area (WorkerDao, TaskDao, SubmissionDao, PayoutDao, NcDao, SupportDao, ContentDao, AuditDao, EventDao, MetaDao, ManagerDao). Lists the UI needs come as `Flow`.
- `CTDatabase.kt` — database `ctteam.db`, version 1, schema export on. `Migrations.ALL` is empty now. RULE: any entity change = version +1 AND a Migration; never destructive migration; never rename the DB.
- `Util.kt` — `Hashing.sha256Hex`, `TimeUtil` (EAT day key, Monday-start week key, `WEEK_MS`), `PhoneUtil.normalize/parse/networkOf` (0772…, 256772…, +256772… all -> 256772…; MTN prefixes 77,78,76,39; Airtel 70,75,74,20), `NameUtil.key` (case/order/spacing-insensitive name key).
- `Repositories.kt` — `Outcome.Ok/Err`; `AuditRepository` (hash-chained `log`, `event`, `verifyChain`), `WorkerRepository` (`register`, `changePhone` [once per 7 days, blocked while payout IN_PROGRESS, history kept], `editNames`, `setActive`), `TaskRepository` (`create`, `update` [auto version + task_versions snapshot], `setActive`), `SubmissionRepository` (`availableTasks` [daily renews next EAT day, one-time once], `submit` [stores proofs + per-worker `seq`], `decide` [manager final say], `reassignOneTime`, `screenshotSeenBefore`, `expectedForWeek`), `PayoutRepository` (`request` [week sum, attaches submissions so nothing is counted twice], `confirmPaid` [AUTO_MATCH or MANUAL, PAID is permanent]), `ManagerRepository` (`isSetUp`, `setUp(pin)` only once, `verifyPin`, PBKDF2).
- `ServiceLocator.kt` — `ServiceLocator.get(context)` returns `Repos` with `.audit .workers .tasks .submissions .payouts .manager .db`.
- `MainActivity.kt` (replaced) — opens the database in the background and logs `APP_START`.
- Root `build.gradle.kts` and `app/build.gradle.kts` replaced: added KSP `1.9.24-1.0.20` and Room 2.6.1, versionCode 2 / versionName 0.2.0 (bump versionCode every piece).

Decisions:
- Money = Long UGX. Times = UTC millis. Day/week keys use EAT (UTC+3); payout weeks Mon–Sun, key = Monday date.
- National ID length is `WorkerRepository.NIN_LENGTH = 14` (change in one place if needed).
- Only ONE manager identity can exist (`manager_identity` id=1, PIN). No "create manager" feature exists. Piece 5 adds the manager signing key; workers must trust only updates signed by it. Piece 11 must include it in the encrypted backup.
- The schema JSON files are generated in CI and not stored in the repo; migrations are still mandatory.


## Piece 3 — Manager area (what exists now)
Files in `app/src/main/java/com/ctteam/app/ui/`:
- `Forms.kt` — shared UI helpers: `CTField` (styled text field; `keyboard=`, `secret=`, `singleLine=`, `minLines=`), `ScreenHeader(title, onBack)`, `GhostButton(text, onClick, danger=false)`, `CTChip`, `InfoRow`, `fmtTime(ms)` (EAT), `ugx(n)`. Existing from piece 1 in Components.kt: `GoldButton`, `GlassCard(onClick=)`, `ShimmerLogo`, `CTBackground`; theme object `CT` (Gold, GoldLight, Teal, TextSoft…).
- `ManagerScreens.kt` — `ManagerArea(onExit)` is the entry. `PinGate` (first time: create PIN = the one-and-only manager; later: unlock; 5 wrong tries = 30 s lock; PIN is asked every time Manager mode is opened). Routes (`MgrRoute`): Dashboard (counts, expected UGX, recent activity, audit-chain check), Workers (list, add worker, detail: edit names, change phone with the once-a-week rule, enable/disable, phone history), Tasks (list, new/edit with versioning), Review (pending submissions, proofs with 🟢🟡🔴 labels, Approve / Reject [reject needs a note]).
- `CTApp.kt` (replaced) — Manager destination now shows `ManagerArea`. `Screens.kt` — only the version text changed. `MainActivity.kt` — logs the real versionName at start. `app/build.gradle.kts` — versionCode 3 / versionName 0.3.0.

Notes for the next AI:
- Screens read data with Room `Flow`s via `remember(repos) { dao.observeX() }` + `collectAsState` (always wrap the Flow in `remember`).
- Actor string for manager actions in the audit log is `"manager"`.
- Review screen cannot show anything until Piece 4 creates submissions (via `SubmissionRepository.submit`).
- Manager "approve" does not yet check or show the NC/BNB or payout parts — those arrive in Pieces 7–8.
- Worker mode is still the placeholder screen. Next: Piece 4 (worker registration on the worker's own phone, daily/one-time task list from `availableTasks`, submit proof, earnings/week view, payout request via `PayoutRepository.request`).


## Piece 4A — Manager access + worker start (what exists now)
Files (zip: `piece-04a-managers-worker.zip`):
- `data/ManagerAccounts.kt` — table `manager_accounts` (up to 20): code name (stored UPPERCASE, unique), personal PIN (PBKDF2, 40 000 iterations, per-row salt), `isMain`, `suspendedUntil` (null / time / `Long.MAX_VALUE` = until re-enabled), `accessExpiresAt` (null / time). `ManagerAccountRepository`: `ensureMain`, `login(pin)` (PIN-only; PINs are unique), `blockReason`, `statusText`, `add`, `disable(durationMs|null)`, `allowFor(durationMs)`, `enableNow`, `setPin`, `rename`, `remove`. Only the main manager may call the change methods. Every action goes to the audit log with actor `manager:CODENAME`.
- MAIN manager is seeded on first use with code name `MAIN` and PIN 114477, stored only as a salted hash (the PIN is not in the source as text). Main can change its own PIN and name in Manager > Managers > MAIN. The main manager can never be disabled or removed.
- `data/CTDatabase.kt` — database version 2, `Migrations.MIGRATION_1_2` creates `manager_accounts`. The old `manager_identity` table (piece 3's single PIN) is kept but no longer used; it is NOT migrated, the old PIN stops working.
- `data/ServiceLocator.kt` — `Repos.managerAccounts` added.
- `ui/ManagerAccess.kt` — `ManagerSession.actor`, `ManagerAccessGate` (PIN sign-in, 5 wrong = 30 s lock, shows the reason if blocked), `ManagersScreen` (main only: list with status, add, detail with Disable for period / Disable until re-enabled / Allow only for period / Enable now / rename / reset PIN / remove). Periods: hours, days, weeks, months (30 d), years (365 d), 1–1000.
- `ui/ManagerScreens.kt` (replaced) — old PinGate removed; `ManagerArea` signs in with the new gate; a watchdog re-checks the signed-in manager every 5 s and signs them out when disabled; dashboard shows "Signed in as CODE" and a Managers card for the main manager; `ACTOR` now equals `ManagerSession.actor`.
- `ui/WorkerScreens.kt` — `WorkerArea`: first time = registration (National ID, names, MTN/Airtel number, "number is in my own name" confirmation) via `WorkerRepository.register`; the worker id is remembered in `meta` key `WORKER_META_KEY`. Then `WorkerHomeScreen`: identity card, this-week expected payout (`SubmissionRepository.expectedForWeek`), pending/approved/rejected counts, change payout number (`changePhone`, once/week rule).
- `ui/CTApp.kt` — Worker destination now `WorkerArea`. `app/build.gradle.kts` — versionCode 4 / 0.4.0.

IMPORTANT limits to remember (Piece 5 must solve):
- Each phone has its own local database. Disabling/enabling a manager only takes effect on the phone where it is done until Piece 5 adds signed sync (include manager_accounts rules in the signed update from the main manager). Same for workers: a worker registers on their own phone; Piece 5 must carry the registration to the manager's phone in a signed report and keep ids consistent.
- The main PIN 114477 is 6 digits, so anyone who extracts the APK could brute-force the stored hash. Advise changing it to a longer PIN.

## Next: Piece 4B — Worker tasks, proof, earnings, payout request
Build in `ui/WorkerScreens.kt` (or new `ui/WorkerTasks.kt` + `ui/WorkerEarnings.kt`) and add buttons on `WorkerHomeScreen`:
1. Task list from `repos.submissions.availableTasks(workerId)` (daily tasks renew next EAT day; one-time once). Show name, pay, type, example link/video (open with an Intent), proof requirements.
2. Submission screen: pick a screenshot (Android photo/document picker `ActivityResultContracts.OpenDocument`), read file name/size/dimensions/modified time/mime; compute SHA-256 with `Hashing.sha256Hex(bytes)`; reference/explanation text fields; call `repos.submissions.submit(workerId, taskId, proofs: List<ProofInput>, referenceText, explanation)` (see `ProofInput` in Repositories.kt). Verdict colours stay `Verdict.NONE` until Piece 6.
3. My submissions / earnings history: `repos.db.submissionDao().observeForWorker(workerId)` grouped by week; status chips Pending/Approved/Rejected with the manager's note.
4. Payout: show weekly expected, button "Request payout" -> `repos.payouts.request(workerId, weekKey)`; show IN_PROGRESS / PAID with `observeForWorker`; remember phone changes are blocked while IN_PROGRESS.
5. Keep versionCode increasing (5 / 0.5.0 for 4B) and update this file.
