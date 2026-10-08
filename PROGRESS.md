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
| 5D | Worker identity: username uniqueness across workers, manager-made reset code (replaces interim recovery when connected), JOIN code for a new phone | DONE in piece-05d-worker-identity.zip (build result: check Actions) |
| 5E | Manager accounts synced to manager phones, signed by the main manager's key (ECDSA P-256, key code checked by voice) | DONE in piece-05e-manager-access.zip (build result: check Actions) |
| 6 | Screenshot verification | DONE in piece-06-screenshot-verification.zip (build result: check Actions) |
| 7 | Payouts, phone/name normalization, statement import | DONE in piece-07-payouts.zip (build result: check Actions) |
| 7b | Build fix: `Sync.kt` used Kotlin's `MatchResult`, which Piece 7's own `MatchResult` class hid | DONE in piece-07b-fix-matchresult.zip |
| 8A | Shared PDF text reader (NC Wallet statements AND the Piece 7 payout statements) | DONE in piece-08a-pdf-reader.zip (build result: check Actions) |
| 8B | NC statement parser + import + remembered rows (overlapping statements) | DONE in piece-08b-nc-statements.zip (build result: check Actions) |
| 8C | Worker BNB claim entry + signed NC_CLAIM report to the manager | NEXT |
| 8D | Matching engine (coin + exact amount + about 2 min, strict one-to-one), manual override, link to Review | todo |
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
8. **NC Wallet / BNB** — split into four small pieces 8A–8D (see "Piece 8 plan" below). Overall: import PDF statement, statement time is UTC, user sees EAT (UTC+3), match coin + exact amount + time within ~2 min, strict one-to-one, remember matched/unmatched rows across overlapping statements, manual override.
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

### 5D — Worker identity (what exists now)
Files (zip: `piece-05d-worker-identity.zip`, after 5C): `data/Sync.kt`, `data/CTDatabase.kt` (version 6, `MIGRATION_5_6` = table `worker_usernames`), `ui/SyncScreens.kt`, `ui/WorkerScreens.kt` (new phase `WPhase.Join`, button on the register screen, Recover phase picks the screen), `app/build.gradle.kts` (13 / 0.13.0).
- Username uniqueness: ENROLL now carries `un`; the manager's enrollment check refuses a username owned by another worker (table `worker_usernames`, lowercase PK). New W2M `USERNAME` message (key `USR:<worker>:<username>`) rides in normal reports, so already-connected workers send theirs in their next report; a clash is refused with the other worker's name. Worker phone: "Change my username" card (needs current PIN or password; `SyncRepository.changeUsername`).
- Reset code: manager Sync > worker > RESET CODE (`makeResetCode`: M2W type `RESET`, HMAC with the worker key, fresh seq each time, valid 3 days by `ts`, usable once via the seen table). Worker "Forgot password or PIN": if the phone has ever accepted a manager message (`hasHeardFromManager`) the screen is `RecoverWithCodeScreen` (paste code + new password/PIN, shares `lock:worker-recover` lockout); otherwise the old interim National ID + number screen stays (worker never connected, so the manager has no key). A RESET pasted into the Updates card is refused with a pointer to the sign-in screen. Reset bumps `credVersion` (ends 30-day trust).
- JOIN code: manager Sync > worker > JOIN CODE (`makeJoinCode`: M2W `JOIN`, seq 0, carries identity, active flag, the KEY, and `s` = highest W2M seq the manager has accepted). Worker register screen > "I HAVE A JOIN CODE": preview with key code (read it to the manager by phone) -> `acceptJoin` creates worker row + key, sets the outgoing W2M counter to `s` (so new reports are not rejected as replays), then the existing Setup screen creates username/password/PIN. Refused if the phone already has that worker/National ID/number. Then the manager taps SEND EVERYTHING AGAIN. The new phone has no old submission history (decisions for unknown submissions are reported "not found" and skipped); Piece 11 backup/restore is the way to keep history.
Limits: a JOIN code is not single-use (it holds the key): send privately, use RESET SYNC KEY if it leaks. Reset/JOIN rely on the manager phone clock vs the worker clock within 3 days.
Not done (moved to 5E): manager accounts on other managers' phones, `pin_ledger` sync.
Decision for 5E: do NOT ship the whole `pin_ledger` to other phones (its PBKDF2 fingerprints of 6-digit PINs can be brute-forced offline, which would let one manager learn others' PINs). Only the main manager issues PINs, so uniqueness is kept on the issuing phone; the ledger belongs in the Piece 11 encrypted backup.

### 5D-x — One-tap reset request (what exists now)
Files (zip: `piece-05d-x-reset-request.zip`; the name is chosen so it sorts AFTER `piece-05d-worker-identity.zip` in every locale): `data/Sync.kt`, `ui/SyncScreens.kt` (both replaced), `app/build.gradle.kts` (14 / 0.14.0). No schema change.
Flow: worker "Forgot password or PIN" (phone already accepted something from the manager) > ASK THE MANAGER = signed W2M `RESET_REQ` (nonce, 4-digit check code shown on both phones) shared with one tap. Manager Sync: the pasted request shows as a "Reset request" card (worker, check code, time); APPROVE, COPY AND SEND = `approveReset` (re-verifies, consumes the request number so it works once, makes the normal `RESET` code), copies it and opens the share window straight away. Worker screen offers a reset code found on the clipboard with one tap (USE IT). Requests expire after 24 h; the reset code still lasts 3 days and works once. The manager's tap stays: without it anyone holding a worker's phone could take over the login and redirect the payout number. Piece 9 (Telegram bot) can carry the same two messages with no copy/paste; the approve tap should stay.

### 5E — Manager accounts between phones (what exists now)
Files (zip: `piece-05e-manager-access.zip`, sorts after 5D-x): `data/ManagerSync.kt` (new), `data/CTDatabase.kt` (replaced: version 7, `MIGRATION_6_7` = one new table `manager_rev`), `data/ServiceLocator.kt` (`Repos.managerSync`), `data/ManagerAccounts.kt` (replaced: `ensureMain` does nothing on a phone that trusts another phone's main manager), `ui/ManagerSyncScreens.kt` (new), `ui/ManagerAccess.kt` and `ui/ManagerScreens.kt` (replaced), `app/build.gradle.kts` (15 / 0.15.0).

How it works:
- The main manager's phone makes an ECDSA P-256 key pair the first time it needs one (`ManagerSyncRepository.ensureOwnKeyCode`; private key in `meta` key `mgrsync:priv`, public `mgrsync:pub`). Managers > top card "Your signing key code" shows `7F3-A92` (first 6 hex of SHA-256("CTMKEY|"+public key)) to read by voice.
- Wire format (separate from the worker codes, prefix `CTM1`): `CTM1.<base64url JSON>.<base64url 64-byte raw ECDSA r||s>`; the signature covers `CTM1.<base64url JSON>`. JSON = {v:1, t, rev, pk (public key), mn (main code name), ts, b:{...}}. Types: `ACCT` (id, code name, PIN salt+hash+iterations, suspendedUntil, accessExpiresAt, createdAt; NEVER a password; only that manager's own PIN hash) and `REMOVE` (id, code name). `SyncCodec`/worker `CT1.` codes are untouched and do not match the `CTM1.` pattern.
- `rev` = main phone time, always rising (`nextRev`). A manager phone applies a code only if `rev` is higher than the one in table `manager_rev` for that account, so old codes can not undo newer ones, and a REMOVE leaves a tombstone so an old ACCT can not bring the manager back.
- Main phone: Managers > a manager > MAKE ACCESS CODE (`makeAccountCode`) after any change (new, reset, disable, allow-for-period, enable, rename). Removing a manager makes a signed `REMOVE` code first (`makeRemoveCode`) and shows it on the Managers screen.
- Manager phone: at the sign-in screen, IMPORT ACCESS CODE (top right) works BEFORE signing in. First code from an unknown key shows a pending card with the key code; the person compares it with the main manager by phone and taps "IT MATCHES — APPROVE" (`approveKey`). After that the key is trusted (`mgrsync:trusted_pub`) and every later code signed by it is applied without asking. A different key later = red warning and a new voice check. FORGET THIS KEY clears the trust (accounts stay).
- Applying ACCT: new account is inserted not-activated (the manager then uses "First time or after a reset" with code name + PIN and creates a password on their own phone). Existing account: code name and disabled/allowed periods are updated; if the PIN hash changed (reset) the local password is removed, `credVersion` is bumped and quick-unlock trust is cleared, so the 5-second watchdog signs the manager out. REMOVE deletes the account and signs them out.
- A phone that approves another phone's key becomes a "manager phone" (`mgrsync:managerphone` = 1): its own seeded MAIN row is deleted and not re-created. It is refused if that MAIN account was already activated or if the phone holds its own signing key (it is a main phone).
- Decision kept from 5D: `pin_ledger` is NOT synced (uniqueness of issued PINs is guaranteed on the main phone, which is the only issuer; the ledger goes into the Piece 11 encrypted backup). Because the code holds the manager's PIN HASH, send each code only to that manager and send the PIN by a different route; long PINs (8 to 10 digits) make an intercepted code much harder to attack.

Limits (honest):
- Each manager phone is still its own island for DATA: workers, submissions and payouts live on the phone whose manager approved the worker's connection. 5E only carries manager ACCOUNTS and their access rules. Sharing worker data between manager phones is NOT built (needs its own piece before Piece 7 payouts across managers).
- Disabling a manager only takes effect on a phone after that phone imports the newer code; a manager who never imports it keeps working there. The signed code can not force a phone to read it.
- Phones with the APK but no trusted key still have their own local MAIN row until the first approve; anyone who knows the first PIN can activate it locally, but nothing signed by that phone is trusted by other phones.
- The signing private key sits in the app database (not Android Keystore) so Piece 11 can back it up; losing the main phone without a backup means managers must re-trust a new key (voice check again).
- Not compile-tested here (no Kotlin/Gradle in the sandbox). The DER/raw ECDSA conversion was checked against a Python reference. If the Actions build fails, paste the error log.


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


## Piece 6 — Screenshot verification (what exists now)
Files (zip: `piece-06-screenshot-verification.zip`, sorts after 5E; no database change, DB stays v7): `data/ScreenshotChecks.kt` (new), `data/Verification.kt` (new), `data/ProofAnalyzer.kt` (new), `data/Daos.kt` (replaced: 4 new queries on `SubmissionDao`: `updateProof`, `proofsWithDhash`, `pendingNow`, `issuesCount`), `data/ServiceLocator.kt` (`Repos.verification`), `ui/WorkerSubmit.kt` (replaced: also computes dHash + tags), `ui/ManagerScreens.kt` (replaced: Review + dashboard), `app/build.gradle.kts` (16 / 0.16.0).

How it works:
- WORKER phone (`ProofAnalyzer`, called from `readPicked`): `dHash` = 64-bit difference hash of 9x8 average-brightness cells (hex, 16 chars, so it survives re-saving/compression); `tags` = `fmt=png|jpeg|webp|gif|other` (from the file's first bytes) + `sw=<Software>` (PNG tEXt "Software" / JPEG EXIF) + `ed=<editor keyword>` found ONLY in metadata text (PNG tEXt/iTXt chunks, JPEG APPn/COM segments; never in picture data, so no random false hits). These go into `ProofInput.dhash` / `metadataTags` and already travel in the 5B `SUBMISSION` report (`dh`, `tg`).
- MANAGER phone (`VerificationRepository.verify(submissionId)` / `verifyAll()` / `issuesCount()`): for every SCREENSHOT proof builds a `CheckInput` and calls the pure function `ScreenshotChecker.evaluate`; stores `verdict` (OK / CHECK / DONT_TRUST) and `verdictNotes` (one line per finding, prefixed 🔴 🟡 🟢 ℹ) on the proof. Safe to run any number of times. Screen size comes from the sync row "Worker phone screen: WxH" (5B) for synced submissions, or from this phone for local ones. Other proofs' warnings (⚠ rows from sync) are included in the submission's worst verdict but never rewritten.
- RED (🔴 Do not trust): the exact same SHA-256 already used in another submission; editing-program keyword in hidden data; file date or file-name date more than 3 days before the submission.
- YELLOW (🟡 Check carefully): picture fingerprint within 6 of 64 bits of another submission's (says "a real task can look the same every day"); not PNG, or file type and content disagree; size unreadable; picture size does not fit the worker's phone screen (short side within 2%, long side within 12%, either orientation); file under 15 KB; file date/name date after the submission (+10 min) or from an earlier day; file date and name date more than a day apart.
- ℹ notes (no colour change): screen size/date unknown; made by an older app version (no dHash/tags).
- UI: Review list shows the worst verdict per submission; detail shows size, file date, hidden-data tags, fingerprint, verdict and reasons, plus the honesty text (green is not proof, red is not proof of cheating, ask for the original, the manager decides). Dashboard has a "Verification issues" tile (waiting submissions with a yellow or red proof). Workers still never see verdicts.
- File-name times are read as East Africa Time. Patterns: `20261007-232846`, `20261007_232846`, `2026-10-07-23-28-46`.

Limits / honesty:
- Everything is a warning sign only. Dates and tags can be changed by the sender; a modified app can leave facts out. Exact originals should be requested as Telegram documents (Piece 9) so SHA-256 can be compared again.
- The near-duplicate check can flag a genuine screen that looks the same every day; that is why it is only yellow.
- Not compile-tested here (no Kotlin/Gradle in the sandbox); the file-name regexes and the Hamming logic were checked with Python. If the Actions build fails, paste the error log.

Next: Piece 7 (payouts: Mon–Sun cycle screens for the manager, Uganda phone + name normalization in `PhoneUtil`/`NameUtil` already exist, payout statement import with Strong / Possible / No match, manual confirmation, permanent archive; reuse `PayoutRepository.confirmPaid` and the 5B "Payout requests waiting" card).


## Piece 7 — Payouts (what exists now)
Files (zip: `piece-07-payouts.zip`, sorts after piece-06; no database change, DB stays v7): `data/PayoutMatching.kt` (new), `ui/PayoutScreens.kt` (new), `data/Daos.kt` (replaced: `PayoutDao.observePaid`, `PayoutDao.paymentRefCount`), `ui/ManagerScreens.kt` (replaced: route `MgrRoute.Payouts`, dashboard card "Payouts" with the number waiting), `app/build.gradle.kts` (17 / 0.17.0).

Flow (SPEC section 16): worker requests (status In progress, reaches the manager phone by the 5B report) -> manager pays the worker's MTN/Airtel number outside the app -> Manager > Payouts: either IMPORT PAYMENT STATEMENT (automatic matching) or MARK PAID BY HAND -> manager confirms (two taps, always) -> `PayoutRepository.confirmPaid` (PAID is permanent; payments row written; audit `PAYOUT_CONFIRMED`) -> the 5C `PAYOUT_PAID` update tells the worker. The old "Payout requests waiting" card in the Sync screen still works and calls the same repository method.
- Statement reading (`StatementParser.parse(text)`): pasted text, clipboard or a CSV/text file (max 500 KB, no PDF yet: a PDF is refused with advice to copy the table text; Piece 8 will need PDF text reading for the NC Wallet statement anyway, build one reader and reuse it here). Layout-free: per line it finds Uganda phone numbers (0772..., 256772..., +256772..., normalized with `PhoneUtil`, only valid MTN/Airtel), a date/time (dd/MM/yyyy or yyyy-MM-dd, read as East Africa Time), EVERY whole-shilling number on the line (amount, fee, balance; commas and .00 handled; letters-attached IDs like TXN123456 ignored) and name words (stop-word list removes "sent", "ugx", "balance"...). A line counts as a payment line if it has an amount plus a phone or 2+ name words. Each line gets a stable key `ST:<sha256 prefix>` (+ `-n` for identical repeats).
- Matching (`PayoutMatcher.match`): for each waiting payout and statement line: phone agrees (payout's frozen number is on the line), name agrees (2+ of the worker's name words on the line; case, order, spacing, punctuation do not matter; middle name may be missing), amount agrees (payout amount is one of the numbers on the line). STRONG = all three; POSSIBLE = exactly two, or more than one line/payout fits equally; below that = NONE (amount alone is never enough). Strict one-to-one inside an import; lines whose key is already in `payments.statementRef` (used for an earlier confirmed payout) are ignored in later imports, so overlapping statements cannot pay twice.
- Results screen: counts, per payout a card with confidence, the statement line, what agreed or not, CONFIRM PAID (YES tap), CONFIRM ALL STRONG, plus statement lines that matched nothing. AUTO_MATCH confirmations store `statementRef` = line key, `matchConfidence`, and the line text in the note. Manual ones store MANUAL and an optional note (e.g. the transaction ID).
- Paid archive (permanent list from `observePaid`): worker, amount, number, week, requested/paid times, how (matched STRONG/POSSIBLE or by hand), manager code name, note.

Limits / honesty:
- Real MTN/Airtel statements were not available while writing, so the reader is deliberately tolerant but untested on real files. If a real statement parses badly, ask the user for a few sample lines (blank out names/numbers if they wish) and adjust `StatementParser`.
- A match is only a suggestion; confirming means "I really sent it". The app cannot see mobile money.
- If the payout amount differs from the line (partial payment) the pair can only be POSSIBLE; `confirmPaid` still records the payout amount. Handle partial payments manually.
- Not compile-tested (no Kotlin/Gradle in the sandbox); the line-parsing regexes were checked with Python. If the Actions build fails, paste the error log.

Next: Piece 8, split into 8A–8D (see "Piece 8 plan" at the end of this file).


## Piece 8A — Shared PDF text reader (what exists now)
Files (zip: `piece-08a-pdf-reader.zip`, sorts after `piece-07b-fix-matchresult.zip`; no database change, DB stays v7): `data/PdfText.kt` (new), `ui/PayoutScreens.kt` (replaced), `app/build.gradle.kts` (replaced: 18 / 0.18.0, new dependency `com.tom-roush:pdfbox-android:2.0.27.0`, extra `META-INF` excludes).
- `PdfText.read(context, uri): Outcome<String>` (suspend, runs on `Dispatchers.IO`, 45 s time limit): plain text of a PDF with `sortByPosition` so table rows stay on one line. Limits: 10 MB, 200 pages. Clear `Outcome.Err` messages for: not a PDF, too big, too many pages, password-protected, no readable text (scan), took too long, other read error. It never guesses text. `PdfText.looksLikePdf(head)` checks the "%PDF" marker. `PDFBoxResourceLoader.init` is called inside `read`.
- Payout statement import (Piece 7): "OPEN PDF, CSV OR TEXT FILE" now accepts PDF. A PDF is first shown in a preview card (line count, character count, the first 1 200 characters) with USE THIS TEXT / DISCARD; only after USE THIS TEXT does the text go into the statement box and through the unchanged `StatementParser`. Text/CSV files still load straight into the box (max 500 KB).
- Piece 8B must call `PdfText.read` and show the same kind of preview before parsing.
- Not compile-tested (no Kotlin/Gradle in the sandbox). Risks to check first if Actions fails: the pdfbox-android dependency (must resolve from Maven Central) and its packaging; the Java-style setters on `PDFTextStripper`. Real PDF layouts are untested: if a real NC Wallet or MTN PDF reads badly, ask the user for a few sample lines.

## Piece 8B — NC statement import (what exists now)
Files (zip: `piece-08b-nc-statements.zip`, sorts after 8A; no database change, DB stays v7): `data/NcStatementParser.kt` (new), `data/NcRepository.kt` (new), `ui/NcScreens.kt` (new), `data/Daos.kt` (replaced: `NcDao` gains `observeStatements`, `observeRecentRows(limit)`, `observeRowCount`, `observeUnmatchedRowCount`, `unmatchedRowCount`, `observeAddedCounts`), `data/ServiceLocator.kt` (`Repos.nc`), `ui/ManagerScreens.kt` (replaced: route `MgrRoute.NcWallet`, dashboard card "NC Wallet" with the number of unmatched rows, `DashStats.ncUnmatched`), `app/build.gradle.kts` (19 / 0.19.0).
- `NcStatementParser.parse(text)` -> `NcParseResult(rows, skippedLines)`; `NcParsedRow(rowNo, coin, amountText, occurredAtUtc, raw)`. Layout-free, per line: a date and time read as UTC (`yyyy-MM-dd HH:mm[:ss]` with `-` `/` `.` or `T`, `dd/MM/yyyy` [if the middle number is above 12 it is read as month/day], `Oct 7, 2026 11:28 PM`, `7 Oct 2026 20:28`; optional AM/PM; missing seconds count as :00), a coin (a known symbol such as BNB/BTC/ETH/USDT/USDC/BUSD..., else an UPPERCASE letters-only word of 2–10 letters that is next to a number and is not a stop word such as UTC/FEE/DEPOSIT), the amount kept as EXACT text (commas and a leading + removed, a leading - kept; zero amounts skipped), and an optional row number at the line start. Amount choice: the number right before the coin, else right after it, else the first number with decimals. `0.5BNB` is split. A row that wrapped over 2–3 lines is joined back only when none of the pieces is a row alone.
- `rowKey` = `coin|normalizedAmount|utcMillis` (`0.50` and `0.5` are the same; `BigDecimal.stripTrailingZeros`). If ONE statement truly contains the same coin, amount and second twice, the repeats get `#2`, `#3`.
- `NcRepository.importStatement(fileName, text, actor)`: refuses the same statement text twice (`nc_statements.fileSha256` = SHA-256 of the text with spacing ignored), stores a statement row, inserts every row with IGNORE on the unique `rowKey` (so overlapping statements add only new rows; `statementId` of a row stays the FIRST statement that held it; matched/unmatched state is never touched by a later import), audit `NC_STATEMENT_IMPORTED`. Result: rows read / new / already known / lines skipped.
- Screen (Manager > NC Wallet): totals (statements, rows, rows not matched yet), IMPORT: paste / clipboard / open PDF (through `PdfText`, with the 8A preview and USE THIS TEXT) / CSV / text file (max 2 MB), CHECK STATEMENT shows rows found, lines skipped and the first 8 rows in UTC and EAT so the manager can compare with the statement, then IMPORT N ROWS. List: statements (rows read, new, already known), newest 100 rows (coin, amount, UTC, EAT, statement row number, matched or not) with an All / Not matched filter.
- Hooks for 8C/8D: `nc_rows.matchedClaimId` null = not matched; `NcDao.unmatchedRows(coin)` and `unmatchedClaims()` already exist; amounts must be compared with `BigDecimal`, never double. The parser keeps the minus sign of a withdrawal-looking row: 8D decides which rows can match a deposit claim.

Limits / honesty:
- No real NC Wallet statement was available. The reader is deliberately tolerant and was checked with made-up lines (ISO, dd/MM/yyyy PM, month-name, `0.5BNB`, thousands commas, fee columns, header lines, ID words that look like coins) in a Python port of the same rules. If a real statement reads wrongly, ask the user for a few sample rows (blank out anything private) and adjust `NcStatementParser`.
- Not compile-tested (no Kotlin/Gradle in the sandbox). If the Actions build fails, paste the error log.

## Piece 8 plan — NC Wallet / BNB, split into 8A–8D
Zip names (they sort after `piece-07-payouts.zip`): `piece-08a-pdf-reader.zip`, `piece-08b-nc-statements.zip`, `piece-08c-nc-claims.zip`, `piece-08d-nc-matching.zip`. Bump versionCode each piece (18 / 0.18.0 for 8A, 19, 20, 21). Update this file after each piece. Existing tables `nc_statements`, `nc_rows`, `nc_claims` (DB v7) are used as they are; if 8C needs extra columns on `nc_claims` (for example `submissionId`), 8C adds ONE migration (v8). Money text is kept as exact decimal text, never a float. Statement times are UTC; the user always sees East Africa Time (UTC+3) next to them.

### 8A — Shared PDF text reader (`data/PdfText.kt`, small UI hook in `ui/PayoutScreens.kt`)
1. One function `PdfText.read(context, uri): Outcome<String>` that returns the plain text of a PDF, page by page, keeping line breaks and table rows on one line when it can. Preferred: the `com.tom-roush:pdfbox-android` library (add the dependency in `app/build.gradle.kts`, call `PDFBoxResourceLoader.init(context)` once). If a PDF has no text (a scan) return a clear message, never a guess.
2. Size limit (for example 10 MB), time limit, and reading on `Dispatchers.IO`, so the screen never freezes.
3. Hook it into the Piece 7 payout statement import: a PDF is no longer refused, its text goes through the same `StatementParser.parse`. Pasted text and CSV keep working.
4. No database change. Show the extracted text in a small preview box with "Use this text" so the manager can see what was read before anything is parsed.

### 8B — NC statement parser + import + remembered rows (`data/NcStatementParser.kt`, `data/NcRepository.kt`, `ui/NcScreens.kt`)
1. `NcStatementParser.parse(text)`: finds, per row, the row number, the coin (for example BNB), the exact amount text and the time. Time is read as UTC. Tolerant of layout (like the Piece 7 parser), checked with sample lines.
2. Import: statement file hash goes into `nc_statements` (the same file twice is refused politely); each row gets `rowKey = coin|amount|utcMillis` and is inserted with IGNORE, so overlapping statements never count a row twice. Rows already matched stay matched; rows still unmatched stay unmatched (remembered across imports).
3. Manager screen "NC Wallet": import (PDF through 8A, pasted text, text file), list of statements with row counts and "new rows added / already known", list of rows with time in UTC and EAT, matched or not.
4. Dashboard card "NC Wallet" with the number of unmatched rows. Audit log entry for every import.

### 8C — Worker BNB claim + signed report (`ui/WorkerNcClaim.kt`, sync message in the 5B style)
1. On the worker phone, next to a task that asks for a BNB deposit: a form with the coin (BNB default), the exact amount (text, up to the statement's decimals) and the time the worker saw in NC Wallet. The worker enters the time in East Africa Time; it is converted to UTC before saving (`timezoneContext = "EAT"` kept for the record). Saved in `nc_claims` (status UNMATCHED).
2. New W2M message type `NC_CLAIM` (same HMAC envelope, sequence number and replay protection as 5A/5B). It travels with the weekly report so the manager phone gets it. The manager's import creates the `nc_claims` row (idempotent by the worker's claim id).
3. Worker sees "sent to manager, waiting for check" and later "verified" (the verified status arrives through 5C in 8D). Claims can be edited only until reported.
4. Migration v8 only if a new column is needed (document it).

### 8D — Matching engine + manual override + link to Review (`data/NcMatcher.kt`, `ui/NcScreens.kt`, Review screen hook)
1. `NcMatcher.match(claims, rows)`: same coin, EXACT amount text equal (numeric compare with BigDecimal, never double), and the row time within about 2 minutes of the claim time. Strict one-to-one: a row serves one claim and a claim one row. When more than one row or claim fits equally it is shown as "needs a decision" and nothing is matched automatically.
2. Manager screen: run matching, results (matched, ambiguous, unmatched claims, unmatched rows), CONFIRM per pair and CONFIRM ALL CLEAR MATCHES (always a deliberate tap). Confirmed pairs set `matchedClaimId` / `matchedRowId` and status MATCHED.
3. Manual override: link any claim to any row, or unlink a pair, with a note; status MANUAL; audit log for every change. A row already matched cannot be matched twice.
4. Review screen: a submission with a BNB claim shows "NC Wallet: matched / not found / needs decision", using wording that it is evidence and the manager decides.
5. 5C update message `NC_VERIFIED` (or reuse the decisions update) so the worker sees the claim as verified.

