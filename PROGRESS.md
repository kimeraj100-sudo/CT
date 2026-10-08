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
| 4B | SECURITY UPGRADE: manager code name + password + PIN, worker username + password + PIN, 30-day quick unlock, random unique manager PINs, reset to default | DONE in piece-04b-security.zip (build result: check Actions) |
| 4C1 | Worker task list + task detail + DEMO worker login | DONE in piece-04c1-worker-tasks.zip (build result: check Actions) |
| 4C2 | Worker proof submission + sign-in to an existing account | DONE in piece-04c2-worker-proof.zip (build result: check Actions) |
| 4C3 | Worker submissions + earnings history | NEXT |
| 4C4 | Worker payout request + payout status, final home wiring | todo |
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
4. **Worker** — split in three. 4A: multi-manager PIN access + worker registration and home. 4B: security upgrade (passwords, usernames, code names, quick unlock, random manager PINs, reset). 4C is split into four small pieces: 4C1 task list and detail, 4C2 proof submission, 4C3 submissions and earnings history, 4C4 payout request and status.
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

## Piece 4B — Security upgrade (what exists now)
Files (zip: `piece-04b-security.zip`, sorts AFTER `piece-04a-managers-worker.zip`, so it replaces the 4A files below):
- `data/Security.kt` (new) — tables `pin_ledger`, `device_trust`, `worker_credentials`; `SecurityDao`; `Auth<T>` result (`Ok` / `Wrong` = counts as failed try / `Denied` = refused, not counted); `Secrets` (PBKDF2 hashing: password 100 000 iterations stored as `"<iterations>:<hex>"`, PIN 100 000; rules; `randomPin`; `TRUST_MS` = 30 days); `LockoutStore` (persistent wrong-try lock in `meta`: 5 wrong = 30 s, each further 5 doubles, max 1 h); `WorkerAuthRepository` (`checkNew`, `create`, `fullLogin`, `quick`, `quickAvailable`, `recover`, `changePassword`, `changePin`, `forgetDevice`).
- `data/ManagerAccounts.kt` (replaced) — entity gains `passSalt`, `passHash`, `credVersion` (+ `activated`). Repository: `fullLogin(code, password, pin)`, `quick(accountId, pinOrPassword)`, `quickCandidate`, `activate(code, issuedPin, newPassword)`, `changeOwnPin`, `changeOwnPassword`, `forgetThisDevice`, `trustedUntil`, main-only `issueNew(by, code, pinLength 6..10)` -> `IssuedAccess(account, pin)` and `resetToDefault(by, targetId, pinLength)`. Old `login(pin)`, `add(code, pin)` and `setPin` are GONE.
- `data/CTDatabase.kt` (replaced) — version 3, `MIGRATION_2_3` (3 ALTER TABLE columns + 3 new tables). `data/ServiceLocator.kt` (replaced) — `Repos.workerAuth` added.
- `ui/SecurityScreens.kt` (new) — `CredentialGate` (shared QUICK / FULL / ACTIVATE sign-in screen), `ChangeSecretsCard`, `MySecurityScreen` (manager), `SetupWorkerLoginScreen`, `WorkerLoginFields`, `RecoverWorkerScreen`, `WorkerSecurityCard`.
- `ui/ManagerAccess.kt` (replaced) — `ManagerAccessGate` now uses `CredentialGate`; Managers screen: ADD MANAGER = code name + PIN length (6/8/10) -> app makes the PIN, shows it ONCE with a "copy message to send" button; manager detail has RESET TO DEFAULT (tap twice, new PIN shown once).
- `ui/ManagerScreens.kt` (replaced) — route `MySecurity`, dashboard card "My security", watchdog also signs a manager out when `credVersion` changed (reset or credentials changed elsewhere).
- `ui/WorkerScreens.kt` (replaced) — `WorkerArea` phases: Register -> (Setup for pre-4B workers) -> Gate -> Home; registration now also asks username, password, PIN; home shows username and a change PIN/password card; the "coming next" card now says Piece 4C.
- `app/build.gradle.kts` — versionCode 5 / 0.5.0.

Rules implemented:
- FULL sign-in = name (manager code name / worker username) + password + PIN. Needed first time on a phone and again every 30 days on that phone. QUICK unlock = only PIN OR only password, for 30 days after the last FULL sign-in on that phone (`device_trust`). Trust is void when `credVersion` changes (reset, password change, PIN change) or the manager taps "Forget this phone".
- New manager: main manager enters a code name only. App creates a random PIN (first digit never 0) that was never issued before: checked against `pin_ledger` (a PBKDF2 fingerprint of EVERY PIN ever issued, kept forever) AND against all current accounts. PIN is shown once and never stored readably. Manager opens Manager > "First time or after a reset", enters code name + PIN, creates their own password (8+ chars, a letter and a digit) -> account is ACTIVATED.
- Reset to default (main manager, any non-main manager): removes the password, issues a NEW unique PIN, bumps `credVersion`, clears trust. The manager must activate again. MAIN can never be reset, disabled or removed.
- Existing accounts after the update: MAIN and any 4A managers keep their PIN but have no password, so they are "not activated". MAIN activates once with code name MAIN + its current PIN (114477 unless it was changed in 4A) and then creates a password. 4A's manual PIN screens are replaced by My security / Reset to default. Existing workers get a one-time "Secure your account" screen.
- Worker PIN 4–10 digits, manager PIN 6–10 digits. Own PIN/password changes need the current PIN or password (so a forgotten password can be replaced while the PIN still works).
- Worker "forgot password or PIN" (INTERIM): National ID + the payout number saved in the app, then new password + PIN. Piece 5 should replace or tighten this with a manager-signed one-time reset code.

Limits to remember (Piece 5 / 11 must solve):
- Each phone still has its own database. A manager created or reset on the main manager's phone exists only there until the signed sync of Piece 5 carries the account (code name, PIN hash, `credVersion`, status) to the manager's own phone. Piece 5 must also carry `pin_ledger` fingerprints so uniqueness holds across phones, and worker usernames must be checked for uniqueness across workers (today only checked on the worker's own phone).
- If the main manager forgets BOTH password and PIN after the 30 days, there is no recovery inside the app. Piece 11 (encrypted backup / emergency migration) must cover this. Until then tell the main manager to keep both safe.
- `pin_ledger` must be part of Piece 11 backups.

## Piece 4C1 — Worker tasks + demo worker (what exists now)
Files (zip: `piece-04c1-worker-tasks.zip`, sorts after `piece-04b-security.zip`):
- `ui/WorkerTasks.kt` (new) — UI models `WTask`, `TaskState` (AVAILABLE / WAITING / DONE), `WorkerHomeData`, route type `WRoute` (Home, Tasks, TaskDetail, Submit, History, Payout); `buildTaskCards(tasks, subs, now)` (same renewal rules as `availableTasks`: daily = once per EAT day, one-time = once; a REJECTED submission frees the task and shows the manager's note); `WorkerTaskListScreen`, `WorkerTaskDetailScreen` (description, pay, proof requirements, example video/link opened with `Intent.ACTION_VIEW`, http(s) only), `WorkerComingSoonScreen`.
- `ui/WorkerDemo.kt` (new) — `WorkerDemo.isDemoLogin(...)`, `homeData()` with 6 made-up tasks covering every state.
- `ui/WorkerScreens.kt` (replaced) — `WorkerShell` (shared by real worker and demo; route state + BackHandler), `WorkerHomeScreen` (real data from Room flows, tasks re-evaluated every 30 s), `WorkerDemoScreen`. Buttons: MY TASKS (real), SUBMISSIONS & EARNINGS and REQUEST PAYOUT (placeholders for 4C3/4C4); "Submit proof" opens a placeholder for 4C2. Disabled worker sees no tasks plus the disabled message.
- `ui/SecurityScreens.kt` (replaced) — `CredentialGate` gets `allowDemo` / `onDemo` (default off, so the Manager gate never has it); `demoAwarePin`; `WorkerLoginFields(allowDemo=)`.
- `app/build.gradle.kts` — versionCode 6 / 0.6.0.

DEMO worker rules:
- Type `demo` in username, password and PIN on the Worker sign-in screen (or the one word `demo` in the quick PIN/password box), or on the Worker REGISTER screen type it in username, password and PIN and tap REGISTER. Not available in Manager mode.
- The demo is memory-only: no database reads or writes, no worker/audit/trust/payout records, no lockout counting. Leaving the demo returns to the normal sign-in or register screen.
- On the worker sign-in and register screens the PIN box uses the normal keyboard (letters allowed only for the word demo). Real PINs are still validated as digits-only.
- Later pieces: 4C2 replaces the `WRoute.Submit` placeholder, 4C3 `WRoute.History`, 4C4 `WRoute.Payout` (one `when` branch each in `WorkerShell`). Demo data feeds the same screens; when adding real features, give the demo fake data instead of calling repos.

## Piece 4C2 — Proof submission + existing-account sign-in (what exists now)
Files (zip: `piece-04c2-worker-proof.zip`, sorts after `piece-04c1-worker-tasks.zip`):
- `ui/WorkerSubmit.kt` (new) — `WorkerSubmitScreen`: `OpenMultipleDocuments` image picker (max `MAX_PROOF_FILES` = 5, 15 MB each, images only). For each file: name, size, width/height (`BitmapFactory` bounds), last-modified (`DocumentsContract`, best effort), MIME, streamed SHA-256. Same picture twice in one submission is refused. Optional reference + explanation. Calls `repos.submissions.submit(...)`; only metadata + hash are stored, not the picture. Verdict stays `NONE` (Piece 6 fills `dhash`, `metadataTags`, verdicts). Success screen -> back to task list; the task then shows "Waiting for review". Double submit is blocked by the repository (PENDING/APPROVED uses the task up).
- `ui/WorkerScreens.kt` (replaced) — `WorkerShell` now takes `onSend`; Submit route is the real screen. Demo `onSend` only moves the demo task to Waiting in memory. New phase `WPhase.Existing` + `ExistingWorkerLoginScreen`: reached from the "I ALREADY HAVE AN ACCOUNT" button on the register screen; username + password OR PIN (one box); shares the worker wrong-try lockout (`lock:worker`); on success saves `WORKER_META_KEY` so the phone is tied to that worker, then Home. `demo` + `demo` opens the demo here too.
- `data/Security.kt` (replaced) — `WorkerAuthRepository.loginExisting(username, secret): Auth<String>` (returns the worker id; refreshes 30-day trust; audit `WORKER_LOGIN`).
- `app/build.gradle.kts` — versionCode 7 / 0.7.0.

Notes / limits:
- This sign-in is weaker than the full username + password + PIN rule (one secret is enough) because it was requested that way. Change `loginExisting` if you want it to require both.
- It only finds accounts in THIS phone's database. Workers added by the manager (no login yet) cannot use it. Cross-phone accounts need Piece 5 sync.
- Next: 4C3 replaces the `WRoute.History` placeholder; 4C4 `WRoute.Payout`. Demo data must feed the same screens (demo submissions/payouts in memory, never repos).

## Next: Piece 4C — Worker tasks, proof, earnings, payout (split into 4C1–4C4)
Zip names (they sort after `piece-04b-security.zip`): `piece-04c1-worker-tasks.zip`, `piece-04c2-worker-proof.zip`, `piece-04c3-worker-history.zip`, `piece-04c4-worker-payout.zip`. All four sit behind the 4B worker sign-in (`WPhase.Home`). Worker audit actor is `"worker:" + workerId`. Bump versionCode each piece (6 / 0.6.0 for 4C1, 7 for 4C2, 8 for 4C3, 9 for 4C4). Update this file after each.

Shared navigation (built in 4C1): `WorkerHomeScreen` gets a small route state (`WRoute`: Home, Tasks, TaskDetail(id), Submit(taskId), History, Payout). 4C1 creates the routes and the four buttons; buttons for pieces not built yet show a "coming in 4C2/3/4" note. Each later piece adds its own screen file and swaps the note for the real screen (a one-line change in `WorkerScreens.kt`).

### 4C1 — Task list + task detail (`ui/WorkerTasks.kt`, replaces `ui/WorkerScreens.kt`)
1. Task list from `repos.submissions.availableTasks(workerId)`: daily tasks renew next EAT day, one-time tasks once. Show name, pay (`ugx`), type chip (Daily / One-time), and a "done today" or "waiting for review" state.
2. Task detail: description, pay, proof requirements, example links/video opened with an Intent (`Intent.ACTION_VIEW`).
3. "Submit proof" button goes to the Submit route (placeholder until 4C2).
4. Disabled worker (`worker.active == false`): list is empty with the manager-disabled message.

### 4C2 — Proof submission (`ui/WorkerSubmit.kt`)
1. Screenshot picker with `ActivityResultContracts.OpenDocument` (one or more images, as the task requires).
2. Read file name, size, dimensions, modified time, mime. Compute SHA-256 with `Hashing.sha256Hex(bytes)`.
3. Reference text and explanation fields.
4. Call `repos.submissions.submit(workerId, taskId, proofs: List<ProofInput>, referenceText, explanation)` (see `ProofInput` in Repositories.kt). Verdict stays `Verdict.NONE` until Piece 6.
5. Success screen, then back to the task list. Block double-submit of the same task in the same renewal period.

### 4C3 — Submissions and earnings history (`ui/WorkerHistory.kt`)
1. `repos.db.submissionDao().observeForWorker(workerId)` grouped by Mon–Sun week (`TimeUtil.weekKey`).
2. Status chips Pending / Approved / Rejected, with the manager's note on rejected items.
3. Weekly totals: approved, pending, rejected, expected payout (`expectedForWeek`).
4. Tap a submission to see its proofs and times (EAT).

### 4C4 — Payout request + status (`ui/WorkerPayout.kt`) and final home wiring
1. Show this week's expected amount and a "Request payout" button calling `repos.payouts.request(workerId, weekKey)`.
2. Show past payout requests with status IN_PROGRESS or PAID (`observeForWorker`), amount, week and times.
3. Explain clearly that the phone number cannot be changed while a payout is IN_PROGRESS.
4. Final wiring on `WorkerHomeScreen`: remove all "coming in…" notes, show pending / approved / rejected counts and expected payout as shortcuts to History and Payout.
