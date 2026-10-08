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
| 4C3 | Worker submissions + earnings history | DONE in piece-04c3-worker-history.zip (build result: check Actions) |
| 4C4 | Worker payout request + payout status, final home wiring | DONE in piece-04c4-worker-payout.zip (build result: check Actions) |
| 5A | Sync core: HMAC envelope, per-worker key, enrollment (manager approves + key code), test message, replay protection | DONE in piece-05a-sync-core.zip (build result: check Actions) |
| 5B | Worker -> manager reports: submissions + proofs metadata, payout requests, phone changes; manager imports into Review | DONE in piece-05b-worker-reports.zip (build result: check Actions) |
| 5C | Manager -> worker updates: tasks, approve/reject decisions, payout PAID, worker enable/disable; worker imports | DONE in piece-05c-manager-updates.zip (build result: check Actions) |
| 5D | Manager accounts + pin_ledger sync (signed by main manager), worker username uniqueness, worker reset code replacing interim recovery, manager-made JOIN code (moved here from 5C) | NEXT |
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

## Piece 4C3 — Submissions + earnings history (what exists now)
Files (zip: `piece-04c3-worker-history.zip`, sorts after `piece-04c2-worker-proof.zip`):
- `ui/WorkerHistory.kt` (new) — models `WSub`, `WProof`; `buildSubCards(subs, tasks)`, `toWProof(entity)`; `WorkerHistoryScreen` (LazyColumn, newest week first; each Mon–Sun week has a summary card: Approved / Pending / Rejected counts + UGX, Expected payout = approved and not yet in a payout, "Already in a payout request"; then that week's submissions as cards with status chip, time (EAT) and the manager's note when rejected). Tapping a submission opens `SubmissionDetail` (times, decision note, reference, explanation, proof files: name, size, dimensions, file date, fingerprint). Verdict labels are NOT shown to workers on purpose. Inner BackHandler returns from detail to the list.
- `ui/WorkerTasks.kt` (replaced) — `WorkerHomeData.submissions: List<WSub>` (default empty).
- `ui/WorkerDemo.kt` (replaced) — 7 made-up submissions (this week + paid last week), `proofsFor`; demo home figures now match them (expected UGX 2,000). Demo "send proof" also adds a Pending entry to the demo history (memory only).
- `ui/WorkerScreens.kt` (replaced) — `WorkerShell` takes `loadProofs`; History route is real; real home builds `subCards` from the Room flows.
- `app/build.gradle.kts` — versionCode 8 / 0.8.0.

Notes: week totals are computed from the submission list (approved and `payoutId == null`, same rule as `unpaidApprovedTotal`). The home tiles still count all-time pending/approved/rejected under the "This week" heading (4C4 final wiring should make them per-week and link them to History). Next: 4C4 replaces the `WRoute.Payout` placeholder.

## Piece 4C4 — Worker payout request + status + final home wiring (what exists now)
Files (zip: `piece-04c4-worker-payout.zip`, sorts after `piece-04c3-worker-history.zip`):
- `ui/WorkerPayout.kt` (new) — UI model `WPayout` + `toWPayout(entity)`; `WorkerPayoutScreen`. Shows (1) a "Payout in progress" card when one exists (amount, week, frozen number, requested time, and the rule that no new payout or number change is possible until it is Paid); (2) "Ready to request": every Mon–Sun week that still has APPROVED, not-yet-attached, not-superseded money (same rule as `unpaidApprovedTotal`), newest first, each with a REQUEST PAYOUT button and a confirm step ("Request UGX x to 07xx…?"). Buttons are replaced by a note when the account is disabled, when a payout is already in progress, or when that week already has a payout (late-approved extra money cannot be requested on its own: repository rule `forWeek`; worker is told to ask the manager); (3) "My payout requests" list with chips 🟡 In progress / ✅ Paid, amount, week, number, requested and paid times (EAT). Calls `repos.payouts.request(workerId, weekKey)` through `onRequestPayout`; repository errors are shown as-is.
- `ui/WorkerTasks.kt` (replaced) — `WorkerHomeData.payouts: List<WPayout>` (default empty). `WorkerComingSoonScreen` is now unused (kept, harmless).
- `ui/WorkerDemo.kt` (replaced) — made-up PAID payout for last week (UGX 5,500); demo home counts are now THIS WEEK only (pending 1, approved 2, rejected 1, expected 2,000).
- `ui/WorkerScreens.kt` (replaced) — `WorkerShell` takes `onRequestPayout`; Payout route is the real screen. Home: Expected payout tile opens Payout; Pending/Approved/Rejected tiles open History and now count THIS WEEK only (superseded excluded); new "Payout in progress" card on home when one exists. Real worker reads `payoutDao().observeForWorker`; expected amount is recomputed when payouts change. Demo request is memory only (creates an IN_PROGRESS demo payout, marks the week's approved demo submissions as in a payout, expected -> 0); never touches repos.
- `app/build.gradle.kts` — versionCode 9 / 0.9.0.

Notes / limits:
- The manager confirms payment with `PayoutRepository.confirmPaid` (Piece 3 review code / Piece 7 payout screens). Until Piece 7 the manager UI has no "mark paid" button, so a real payout stays IN_PROGRESS (and blocks the worker's phone change and new requests). Piece 7 must add it, and Piece 5 must carry the request to the manager's phone (each phone has its own database).
- Piece 4C is finished: the worker side (register / sign in, tasks, proof, history, payout) is complete on one phone.
- Not compile-tested here (no Gradle in the sandbox). If the Actions build fails, paste the error log.

## Piece 5 — Security & sync, split into 5A–5D
Why split: it touches both phones and every table. Each part ships alone and is testable by copy/paste. Zip names sort after 4C4: `piece-05a-sync-core.zip`, `piece-05b-...`, `piece-05c-...`, `piece-05d-...`. Bump versionCode each piece (5A = 10 / 0.10.0).

### 5A — Sync core (what exists now)
Files (zip: `piece-05a-sync-core.zip`):
- `data/Sync.kt` (new) — tables `sync_keys` (workerId, keyHex) and `sync_seen` (PK workerId+direction+seq), `SyncDao`, `Envelope`, `SyncCodec`, `SyncRepository`.
- `data/CTDatabase.kt` (replaced) — version 4, `MIGRATION_3_4` (two new tables only). `data/ServiceLocator.kt` (replaced) — `Repos.sync`.
- `ui/SyncScreens.kt` (new) — `WorkerSyncCard` (worker home) and `ManagerSyncScreen` (manager dashboard > Sync).
- `ui/ManagerScreens.kt` (replaced) — route `MgrRoute.Sync`, dashboard card "Sync" (all managers). `ui/WorkerScreens.kt` (replaced) — real worker home shows `WorkerSyncCard` (not in demo). `app/build.gradle.kts` — 10 / 0.10.0.

Wire format (every message, one line, no spaces): `CT1.<base64url(JSON)>.<HMAC-SHA256 hex over "CT1.<base64url>">`. JSON = `{v:1, t:type, w:workerId, d:"W2M"|"M2W", s:seq, ts:senderMillis, b:{body}}`. `SyncCodec.parseAll` finds EVERY code inside pasted text (SPEC: the manager pastes many at once), tolerating line breaks/spaces/extra words; the MAC is exactly 64 hex chars. `SyncRepository.receiveAll(text)` returns one result per message; the Sync screen shows a summary ("N checked: X accepted") and queues enrollment approvals one by one.
Key: 32 random bytes made on the worker phone (`ensureKey`, lazily, so pre-5 workers need no migration). Key check code = first 6 hex of SHA-256("CTKEY|key") shown as `7F3-A92` on both phones to compare by voice.
Enrollment (`ENROLL`, seq 0, resendable): body carries nin, names, phone, network, createdAt and the KEY. Manager phone: `receiveAll(text)` -> `SyncIncoming.Enroll(preview, wire)`; the manager compares the key code and approves -> `acceptEnrollment` re-checks everything in one transaction. Modes: NEW (creates worker with the worker's own id), LINK (manager already added the same National ID by hand: ids are re-pointed with `relink*` queries, manager's record/history kept, phone difference warned), ATTACH, ALREADY. Blocking problems: key conflict (different key for same id), National ID or phone owned by another worker, invalid fields, bad signature. `resetKey(workerId, actor)` (button in Sync) clears key + seen rows for a reinstall/new phone.
Other messages: `receiveEnvelope` looks up the key, verifies the HMAC (bad = rejected + `SYNC_REJECTED` event), then `markSeen` — a repeated (worker, direction, seq) is refused ("Already received"). Gaps are reported but late messages are still accepted. Types the manager phone handles are listed in `SyncRepository.MANAGER_TYPES` (5A: `PING` only; unknown types are NOT consumed, so they can be re-sent after the app is updated). Outgoing counters live in `meta` keys `sync:out:<dir>:<workerId>` (`nextSeq`).
Delivery: COPY, SHARE (Android chooser), SAVE AS FILE (CreateDocument) on the worker side; paste, paste-from-clipboard, OPEN FILE (max 300 KB) on the manager side. No FileProvider/manifest change was needed. Telegram automation comes in Piece 9.

How to extend (5B/5C): add a type string, make it in `SyncRepository` with `ensureKey` + `nextSeq` + `SyncCodec.seal`, add it to `MANAGER_TYPES` (or a worker-side set) and handle it inside the `when (env.type)` in `receive` (already inside the transaction that records the seq). Always apply messages idempotently by their own ids (submission id, payout id) — the seq check stops replays, ids stop duplicates from re-sent content.

SPEC.md check (sync section, line 79): per-worker HMAC-SHA256 key, signed worker reports pasted in bulk, signed per-worker update codes, task IDs + sequence numbers against double counting/replays, join codes sent privately — all consistent with 5A–5C. One difference: the SPEC has the MANAGER create the key inside a "join code" sent to the worker; 5A has the WORKER create the key and send an enrollment code to the manager (same private-channel + key-code voice check). Both are kept: 5C adds a manager-made JOIN code (type `JOIN`, M2W, carries worker id, names, key) so a worker on a new phone/reinstall can join without the manager typing anything. Other SPEC items still to honour later: weekly file of all transactions/completed tasks sent on Telegram (5B builds the file, Piece 9 the bot — the user said yes to the bot and has NOT given a Telegram username); original screenshots go as Telegram Documents (Piece 9); report must carry SHA-256, dHash, size, dimensions, file-name time, modified time, editor tags, device screen size, claim time (Piece 6 fills dHash/tags; 5B must include every field that exists, leaving room for the rest); worker types exact BNB amount + time with time zone next to a task (Piece 8 claims, `nc_claims` table exists); only verified tasks add to payout, manager can override.

Limits (be honest in the UI and to the user):
- The enrollment message contains the key, so it must go to the manager in a PRIVATE chat; the key-code voice check detects a swapped message. HMAC proves holder-of-key and integrity, not the person.
- Keys sit in the app's private database (not Android Keystore); Piece 11 backups must encrypt them. If a worker's key leaks, use RESET SYNC KEY.
- Timestamps are shown but not enforced (phone clocks differ). Replay protection is by sequence number only.
- Not compile-tested here (no Gradle in the sandbox). If the Actions build fails, paste the error log.

### 5B — Worker reports (what exists now)
Files (zip: `piece-05b-worker-reports.zip`, sorts after `piece-05a-sync-core.zip`): `data/Sync.kt` (replaced), `data/CTDatabase.kt` (replaced: version 5, `MIGRATION_4_5` = one new table `sync_outbox`), `ui/SyncScreens.kt` (replaced), `app/build.gradle.kts` (11 / 0.11.0).

New message types (W2M, same envelope as 5A): `SUBMISSION` (id, task id/name/version, claimed time + pay, worker seq, reference, explanation, device screen size `ds`, proofs `pr[]` with EVERY ProofEntity field: kind, sha256 `h`, dhash `dh`, file name, size, width/height, file modified time `fm`, mime, metadata tags `tg`, text), `PAYOUT_REQ` (id, week, claimed amount, number, time, ids of the submissions in it), `PHONE_CHANGE` (new number, time, previous number). The registration number travels only in ENROLL. Pictures are never inside a report (Piece 9 sends originals as Telegram documents).
Worker phone: `SyncRepository.makeReport(workerId, scope, device)`; scopes NEW / THIS_WEEK / LAST_WEEK / ALL. The first time an item is reported it gets a fixed message number (`sync_outbox`: itemKey `SUB:`/`PAY:`/`PHN:`, seq, madeAt); making a report again (weekly file, lost message) gives byte-identical codes with the same numbers, so repeats are harmless. Report is split into parts of at most 3500 characters (whole codes only) plus one "everything in one file" text. `unsentCount` feeds the "Not reported yet" counter. UI: `ReportCard` inside `WorkerSyncCard` (no change to WorkerScreens).
Manager phone: `receiveAll(text, actor)` sorts messages: number changes, then submissions, then payout requests (then by worker + seq). The seq is remembered ONLY when a message is used or permanently refused; messages that fail for a fixable reason (task missing, nothing approved yet, number unknown, another payout in progress, worker not connected) are NOT consumed and can be sent again. Everything is applied in one transaction.
- SUBMISSION -> `SubmissionEntity` PENDING with the worker's own submission id, so it shows in Review. Time sanity: claimed time more than 10 min in the future is replaced by the arrival time; day/week keys are recomputed here. A daily/one-time task already pending/approved is refused (paid once). PAY AND VERSION COME FROM THE MANAGER'S OWN TASK (task_versions snapshot for the reported version, else current task), never from the message; differences become warnings. Extra proof rows (kind TEXT) carry "arrived by sync", the worker's screen size, and each warning as a ⚠ line with verdict CHECK (Review shows TEXT proofs as plain lines). Piece 6: read `ds` from the TEXT row, fill dhash/tags/verdicts.
- PAYOUT_REQ -> `PayoutRequestEntity` IN_PROGRESS. Amount = sum of the listed submissions that are APPROVED, unattached and not superseded ON THIS PHONE; they are attached to the payout (`attachIds`). The worker's claimed amount only produces a warning (stored in the payout `note`). Number must be the worker's current number or one in their history (else wait for the number-change message). One IN_PROGRESS per worker and one payout per week, same as the worker side.
- PHONE_CHANGE -> updates worker phone + history, with warnings (second change within a week, payout in progress, previous number differs). A late OLDER change is only added to history and never overwrites a newer number. Number owned by another worker = refused.
- Manager Sync screen: after approving a connection, held messages that only waited for that worker are retried automatically; "N item(s) imported" hint; new section "Payout requests waiting" (all IN_PROGRESS payouts: worker, amount, number, week, warnings) with MARK AS PAID (two taps, calls `PayoutRepository.confirmPaid(..., "MANUAL", ...)`). Piece 7 will replace this with the full payout screen + statement matching; Piece 5C must send the PAID message when a payout becomes PAID.
Limits / honesty:
- Tasks only exist on the manager phone until 5C, so a worker phone cannot create real submissions for the manager yet; a submission for a task the manager phone does not have is refused (not consumed). On ONE phone (manager and worker in the same app) the same ids already exist, so reports just say "Already on this phone". Full two-phone testing needs 5C.
- Nothing is checked about screenshots yet (verdict stays NONE until Piece 6). The worker's phone clock is not trusted, only sanity-checked.
- A worker who edits the app can lie in any field; that is why money, pay and week sums are recomputed from manager-side approvals. The manager still decides every submission.
- Not compile-tested here (no Gradle/Kotlin in the sandbox). If the Actions build fails, paste the error log.

### 5C — Manager updates (what exists now)
Files (zip: `piece-05c-manager-updates.zip`, after 5B): `data/Sync.kt` (replaced; no schema change, DB stays v5), `ui/SyncScreens.kt` (replaced), `app/build.gradle.kts` (12 / 0.12.0).
M2W message types (signed with the worker's key, which the manager phone holds from enrollment; numbers from `nextSeq(workerId, M2W)`; same fixed-number `sync_outbox` trick as 5B, itemKey includes the worker id): `TASK` (full task incl. active + version; key `TASK:<id>:<version>:<worker>`, so each edit is a new message), `DECISION` (submission id, APPROVED/REJECTED, note, time, manager's pay; key `DEC:<id>`), `PAYOUT_PAID` (payout id, time, manager's amount, mode, ids of the submissions really included; key `PAID:<id>`), `WORKER_STATE` (version = number of WORKER_ENABLED/DISABLED audit rows for that worker, plus active; key `WST:<worker>:<ver>`; worker applies only a higher version, stored in meta `sync:wver:<workerId>`).
Manager: `SyncRepository.makeUpdate(workerId, everything)`, `pendingUpdateCount`. In Sync > connected worker: "Updates not sent yet", MAKE UPDATE (only new), SEND EVERYTHING AGAIN (use after the worker reinstalls). Parts of at most 3500 chars + one file text.
Worker: `receiveFromManager(workerId, text)`; new "Updates from the manager" card (paste / clipboard / file) in `WorkerSyncCard`. Applies in order TASK, DECISION, PAYOUT_PAID, STATE; only messages for this worker id with a valid HMAC; replay by (worker, M2W, seq); all idempotent (task version compare, decision only on PENDING, paid only on IN_PROGRESS). Task upsert keeps the manager's version number (it does NOT go through `TaskRepository.update`). A decision also overwrites the worker's frozen pay with the manager's amount; PAID overwrites the amount too and releases submissions the manager did not include (payoutId=null; note `forWeek` still blocks a new request for that week, worker is told to ask the manager). Worker screens update by themselves through the Room flows.
Not done / limits:
- JOIN message (worker joins on a new phone without the manager typing) moved to 5D because it must be tied to registration/credentials.
- `reassignOneTime` (superseded) is not synced yet; announcements/learn content come in Piece 10; manager accounts / pin_ledger in 5D.
- M2W messages are not yet signed by a separate manager key: they use the worker's key, which the worker also holds. A modified worker app could forge a decision on its own phone, but money is recomputed from the manager phone's approvals (5B), so nothing is gained. 5D/10 add the main-manager signature for content that must be trusted by everyone.
- Two-phone flow now works end to end: manager makes tasks -> update -> worker does task -> report -> Review -> update with decision -> worker requests payout -> report -> Payout requests waiting -> mark paid -> update (PAID).
- Not compile-tested here (no Kotlin/Gradle in the sandbox). If the Actions build fails, paste the error log.

## Reference (done): Piece 4C plan — Worker tasks, proof, earnings, payout (split into 4C1–4C4)
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
