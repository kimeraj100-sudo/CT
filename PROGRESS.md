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
| 8B-fix | Build fix: two classes named `PickedFile` (Piece 4C2 `WorkerSubmit.kt` and Piece 8B `NcScreens.kt`); the 4C2 one is now `PickedProof` | DONE in piece-08b-z-fix-pickedfile.zip |
| 8C | Worker BNB claim entry + signed NC_CLAIM report to the manager | DONE in piece-08c-nc-claims.zip (DB v8; build result: check Actions) |
| 8D | Matching engine (coin + exact amount + about 2 min, strict one-to-one), manual override, link to Review, NC_VERIFIED update to the worker | DONE in piece-08d-nc-matching.zip (build result: check Actions) |
| T1 | Team mining: contributions ledger, decayed shares, 70/30 split, UGX rates | DONE in piece-08t1-team-ledger.zip (DB v9; build result: check Actions) |
| T2a | Team mining: signed TEAM_VIEW update to the worker + worker "Team mining" card | DONE in piece-08t2a-team-worker-sync.zip (build result: check Actions) |
| T2b | Team mining: manager's daily mined estimate, expected daily payout, what-if calculator, team capacity totals | DONE in piece-08t2b-team-daily-whatif.zip (no DB change; build result: check Actions) |
| T3a | Team mining: real daily mined amounts (DB v10), 7-day average replaces the typed estimate | DONE in piece-08t3a-mined-days.zip (app 26 / 0.25.0; build result: check Actions) |
| T3b | Team mining: weekly hash goal + rig ledger (capacity), progress carried to workers | DONE in piece-08t3b-goal-rigs.zip (DB v11, app 27 / 0.26.0; build result: check Actions) |
| T3c | Team mining: gifts (multiplier for a period or fixed extra payout, reason required) shown to the worker | DONE in piece-08t3c-gifts.zip (DB v12, app 28 / 0.27.0; build result: check Actions) |
| T4 | Team mining: leaderboard, levels, streaks, badges, quiet-worker list, weekly summary | DONE in piece-08t4-leaderboard.zip (no DB change, app 29 / 0.28.0; build result: check Actions) |
| 9A | Telegram, manager side: connect your own bot (token), read worker messages and text files while the manager area is open, auto-import (same checks as Sync), chat links, inbox for items needing a tap | DONE in piece-09a-telegram-reader.zip (DB v13, app 30 / 0.29.0; first use of the INTERNET permission; build result: check Actions) |
| 9B-1 | Telegram, manager -> worker: send updates (new only / everything), reset codes (incl. approve-and-send) and JOIN codes from the app to the worker's linked chat; send-all; retry-safe numbers | DONE in piece-09b-1-telegram-send.zip (no DB change, app 31 / 0.30.0; build result: check Actions) |
| 9B-2 | Worker side: Android share target (Telegram > Share > CT Team), automatic import on the worker phone, clipboard "USE IT" offer, reset code fills itself | DONE in piece-09b-2-worker-import.zip (no DB change, app 32 / 0.31.0; needs 9B-1 in the repo; build result: check Actions) |
| 9B-3 | Announcements: one message, signed separately per worker, sent to every linked Telegram chat; shows on the worker's home screen | DONE in piece-09b-3-announcements.zip (no DB change, app 33 / 0.32.0; needs 9B-1 and 9B-2; build result: check Actions) |
| 9C-1 | Originals: a worker sends the picture to the bot as a FILE; the manager app downloads it, works out SHA-256, saves it, compares with the reports, logs the result | DONE in piece-09c-1-original-pictures.zip (no DB change, app 34 / 0.33.0; needs 9A; build result: check Actions) |
| 9C-2 | Review screen, per screenshot: original received or not, SHA-256 match, dHash + hidden tags re-worked from the saved original and compared with the report, show the picture, ASK FOR THE ORIGINAL on Telegram | DONE in piece-09c-2-review-originals.zip (no DB change, app 35 / 0.34.0; needs 9C-1; build result: check Actions) |
| 9D-1 | Weekly summaries (T4 text) sent on Telegram, one worker or all, once per worker per week | DONE in piece-09d-1-weekly-summaries.zip (no DB change, app 36 / 0.35.0; needs 9B-1; build result: check Actions) |
| 9D-2a | Support chat, data + worker side: worker writes to the manager (rides in the normal report), manager's reply travels as a signed message and lands in the worker's chat | DONE in piece-09d-2a-support-worker.zip (no DB change, app 37 / 0.36.0; needs 9B-3; build result: check Actions) |
| 9D-2b | Support chat, manager screen: list of chats with unread badge, thread, reply (Telegram or copy) | DONE in piece-09d-2b-support-manager.zip (no DB change, app 38 / 0.37.0; needs 9D-2a; build result: check Actions) |
| 9D-3 | Notifications: manager phone checks every ~15 min and shows counts; worker phone gets a daily "unsent items" reminder (WorkManager + notification permission) | DONE in piece-09d-3-notifications.zip (no DB change, app 39 / 0.38.0; needs 9D-2b; build result: check Actions) |
| 10A | Learn & Earn, manager side: write, save as draft, publish / hide, delete lessons (text, category, picture link, video link) | DONE in piece-10a-learn-manager.zip (no DB change, app 40 / 0.39.0; needs 9D-3; build result: check Actions) |
| 10B | Send lessons to workers: new M2W type LEARN (one signed message per lesson and worker, version-aware; hide = remove), via Telegram or copy | DONE in piece-10b-learn-send.zip (no DB change, app 41 / 0.40.0; needs 10A; build result: check Actions) |
| 10C | Worker Learn screen: list by category, read, open video/picture link, "read" tick | DONE in piece-10c-learn-worker.zip (no DB change, app 42 / 0.41.0; needs 10B; build result: check Actions) |
(Announcements were already done in 9B-3.)
| 11a | Main manager login built into the build: code name MAIN + password + PIN (hashes only) | DONE in piece-11a-main-login.zip (no DB change, app 43 / 0.42.0; build result: check Actions) |
| 11b | Backup export: one encrypted, integrity-checked, versioned file (passphrase, AES-GCM, SHA-256 inside), saved with the file picker; CHECK A BACKUP FILE | DONE in piece-11b-backup-export.zip (no DB change, app 44 / 0.43.0; build result: check Actions) |
| 11c | Restore: read + verify the file, preview (counts now vs backup, age, warnings), typed RESTORE, safety copy, replace the database, sync numbers kept moving forward, restart, UNDO | DONE in piece-11c-restore.zip (no DB change, app 45 / 0.44.0; build result: check Actions) |
| 11d | Send the backup to Telegram (own bot, as a document) / Drive / SD card through the share sheet; reminder when the last backup is old | DONE in piece-11d-backup-send.zip (no DB change, app 46 / 0.45.0; build result: check Actions) |
| 11e-1 | Migration, part 1: message numbers stay in step after a restore or a new manager phone (worker tells the manager the last number it received) | DONE in piece-11e-1-number-sync.zip (no DB change, app 47 / 0.46.0; needs 11d; build result: check Actions) |
| 11e-2 | Migration, part 2: manager screen "Move to a new phone" (step list with live checks) + RETIRE THIS PHONE (stops Telegram reading, notifications and sign-in on the old phone) | DONE in piece-11e-2-move-phone.zip (no DB change, app 48 / 0.47.0; needs 11e-1; build result: check Actions) |
| 11e-3 | Migration, part 3: worker card "The manager has a new phone" (one tap: connection code + everything again) | DONE in piece-11e-3-worker-new-manager.zip (no DB change, app 49 / 0.48.0; needs 11e-2; build result: check Actions) |
| 12a | Polish, touch feel: spring press-scale + light haptic on every button, chip and tappable card; cards fade/rise in; both off when phone animations are off | DONE in piece-12a-touch-feel.zip (no DB change, app 50 / 0.49.0; build result: check Actions) |
| 12b-1 | Three-bar menu at the top right of the MANAGER dashboard: every screen grouped inside it, home keeps only the daily cards (owner request) | DONE in piece-12b-1-menu-manager.zip (no DB change, app 51 / 0.50.0; needs 12a; build result: check Actions) |
| 12b-2 | The same three-bar menu on the WORKER home: team mining, learn, support, sync, payout number, reminder, security moved into it | DONE in piece-12b-2-menu-worker.zip (no DB change, app 52 / 0.51.0; needs 12b-1; build result: check Actions) |
| 12b-3a | Polish: screens slide / cross-fade between each other (manager and worker), direction follows depth | DONE in piece-12b-3a-slide-between-screens.zip (no DB change, app 53 / 0.52.0; needs 12b-2; build result: check Actions) |
| 12b-3b | Polish: count-up on the stat tiles (manager dashboard and worker home), pulsing placeholder while the dashboard loads | DONE in piece-12b-3b-count-up-shimmer.zip (no DB change, app 54 / 0.53.0; needs 12b-3a; build result: check Actions) |
| 12c | Polish, finishing: friendly empty states + a drawn success checkmark after approve / reject, support reply, lesson save / send | DONE in piece-12c-empty-states-success.zip (no DB change, app 55 / 0.54.0; needs 12b-3b; build result: check Actions). Glass bottom bar: NOT built (only if the owner asks). Next: run the whole app on a phone and report problems; then fixes. |
| R1 | Referral rewards, data + manager screen: referral codes, referral links, points ledger, status ladder, rules (DB v14) | DONE in piece-13r1-referral-data.zip (DB v14, app 56 / 0.55.0; needs 12c; build result: check Actions) |
| R1b | Referral rules editable by the MAIN manager only (points, reduction per referral, minimum, team or own count, approved work needed, UGX per point, status steps) + live preview | DONE in piece-13r1b-referral-rules.zip (no DB change, app 57 / 0.56.0; needs R1; build result: check Actions) |
| R2 | Worker types a referral code when connecting (ENROLL gets field `rc`), manager sees it when approving the enrollment and the referral is recorded | DONE in piece-13r2-referral-code.zip (no DB change, app 58 / 0.57.0; needs R1; build result: check Actions) |
| R3a | REFERRAL_VIEW update to the worker, data only (manager builds it, worker phone checks and keeps it; no screens) | DONE in piece-13r3a-referral-view-data.zip (no DB change, app 59 / 0.58.0; needs R2; build result: check Actions) |
| R3b | Worker "Referrals & status" page in the three-bar menu: my code (COPY + SHARE text), status ladder with progress to the next status, points balance, my referrals and their progress (reads `ReferralViewCodec.load`) | NEXT |
| R4 | Points spending: worker asks to use points (cash-out at the manager's UGX per point, or a gift), manager approves; ledger REDEEM entry; link to T3c gifts / payout | todo |
| R5 | Status perks: a task can ask for a minimum status (Builder, Leader, Champion); TASK message carries it; worker sees locked tasks with the reason | todo |

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
2. Package each piece as `piece-NN-name.zip` (files at zip root as in the project). Keep pieces small enough to finish in one chat; if one is still too big, cut it again (see "SPLIT AGAIN" below the splitting trick).
3. Do not restart or simplify the concept; if something can't be done yet, leave the architecture for it.
4. After each piece: update this file, and ask the user to paste the GitHub Actions error log if the build fails.
5. Keep honesty in the UI: automated checks flag, the manager decides.

## HOW TO KEEP EACH CHAT SMALL (the splitting trick) — read before planning any piece
Big pieces make a chat run out of room before the zip is finished. Do this every time:
1. **Estimate first, then split.** Before writing code, list the features of the piece and the files each one touches. If the piece would need more than about 4 new files, OR a database migration AND a screen AND a sync message, OR roughly 900+ new lines, SPLIT it into lettered parts (T3 -> T3a, T3b, T3c; 4C -> 4C1..4C4; 5 -> 5A..5E). Say the split to the user in two lines and build part (a) only.
2. **One slice that works alone.** Each part must be usable and testable by itself: its data (at most ONE migration) + the smallest screen that shows it. Never leave a part half-wired for the next one.
3. **Zip names.** `piece-<number><letter>-<name>.zip`; keep the sort order right (`08t3a` < `08t3b` < `piece-09`). A name that sorts later than a newer piece can overwrite it.
4. **New file beats replaced file.** Put new code in NEW files. When an old file must change, change only the lines needed: unzip every zip in sorted order, read the real latest file, and edit with exact-string replace that ASSERTS the old text exists once (a Python patch script). Ship the patched file whole (the repo replaces by path). Never retype big files from memory.
5. **Messages grow by optional fields.** Add new fields to the sync messages as optional (old phones ignore them); bump a `rev` or fingerprint only through the existing helpers.
6. **Check what you can.** No Kotlin compiler is available in the sandbox, so: balance-check braces, run the pure formulas in a Python/Node port, check SQL migrations in sqlite, and keep Room entity/migration column types identical.
7. **Close the piece in this order:** zip -> update THIS file (status row, "what exists now", limits, next) -> tell the user which zips to upload to the next chat.
8. **Tell the user what to upload next:** only `PROGRESS.md` and the zips of files the next part changes or depends on (see FILE OWNERS at the bottom: the latest zip that holds each file). When in doubt upload all zips; the sandbox unzips them in sorted order.
9. **Keep this file short.** When a "what exists now" section is older than three pieces, shrink it to its API names and limits; the code is the source of truth.

### SPLIT AGAIN — when a part is STILL too big (levels 2 and 3, any depth)
Use this when the first split (a, b, c...) still leaves a part that would not fit in one chat. Cut again, and again, until each zip fits. A zip is "too big" if it has more than ~6 files, or more than ~700 lines of new Kotlin, or a file over ~350 lines, or two migrations. Say the new cut to the user in two lines, build the first zip only, and write what is left into PROGRESS.md ("Half-done" below).

**Level 2 — cut by layer, bottom-up (every zip must compile ALONE).**
1. `-1` data: entities + DAO + ONE migration + repository + pure formulas. No UI. It compiles because nothing calls it yet.
2. `-2` leaf screens: NEW composables in NEW files. Nothing links to them yet, so they cannot break the app.
3. `-3` wiring: only the tiny hooks into old files (one line per hook: a menu card, a route, a call), applied with the assert-once patch script.
Rule: LEAF FIRST, CALLER LAST. Never ship a call to code that is in a later zip.

**Level 3 — cut one big file into small files.** Split a long screen into `XxxA.kt`, `XxxB.kt` (each a leaf composable), ship them first, and ship the file that calls them last. Keep each file under ~350 lines.

**The STUB trick (when two parts need each other).** The first zip ships a tiny stub in its own file, e.g. `ui/XStub.kt` containing `@Composable fun XSection(repos: Repos, actor: String) {}`, plus the hook that calls it. The later zip ships the real code at the SAME PATH (`ui/XStub.kt` replaced by the real `XSection`). Same path = newer zip wins, so nothing else changes and the app compiled in between. Never leave a stub when closing the last part; note every stub in "Half-done".

**Zip names for sub-parts (this decides the build order, get it right).**
- Key = 2-digit number, then a lowercase letter, then optional alternating digit / letter: `09d`, `09d2`, `09d2a`, `09d2a1`. Name = text after the key and a hyphen: `piece-09d2a1-support-data.zip`.
- Siblings must differ at the SAME position with the SAME kind of character (digit vs digit, letter vs letter). No key may be the beginning of another key in the repo: when you split a part, the parent zip name is NOT used any more.
- The name text never starts with a digit.
- A fix zip takes the key of the zip it fixes + `-z-`: `piece-08b-z-fix-pickedfile.zip` sorts after `piece-08b-nc-statements.zip` and before `piece-08c`. Do not add a bare letter to a key that has none (`piece-07b-fix-...` next to `piece-07-payouts.zip` sorts BEFORE the payouts zip in locales that ignore punctuation, and would be overwritten; rename it `piece-07-z-fix-matchresult.zip` if the build ever shows the old error again).
- Before shipping, check the order with `sortcheck.py` (below) using ALL zip names of the repo in build order. It must print OK.

```
import re, sys
names = sys.argv[1:]                      # zip names in the order they must be built
c_order = sorted(names)
loc_order = sorted(names, key=lambda n: (re.sub(r'[^0-9a-z]', '', n.lower()), n))
print("OK" if names == c_order == loc_order else "WRONG ORDER\n C : %s\n loc: %s" % (c_order, loc_order))
```

**Name-clash check (before zipping).** For every new top-level class, object, function, constant: `grep -rn "\bName\b"` in the merged project. A clash is a build error (this is what piece 7b fixed: `MatchResult`). Prefix new names with the feature (`TeamGift...`, `NcClaim...`).

**Half-done note.** If a chat must stop in the middle of a split, close it cleanly: ship the last zip that compiles, then add a "Half-done" block to this file listing (1) zips shipped, (2) files NOT written yet with the exact public names and signatures they must expose, (3) hooks still missing (file + the exact line to add), (4) stubs to replace. The next chat starts from that block, not from memory.

**Ask for the next upload the smart way.** After each sub-zip, tell the user the exact zips to upload next (see FILE OWNERS) and keep the list short; sub-parts that only add NEW files need only this file.

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
- `data/CTDatabase.kt` — piece-09a-telegram-reader.zip
- `data/ServiceLocator.kt` — piece-09c-1-original-pictures.zip
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

## Piece 8C — Worker BNB claims (written from the code by the next AI; the author did not leave notes)
Files in `piece-08c-nc-claims.zip`: `ui/WorkerNcClaim.kt` (worker form + list "My claims"), `data/NcRepository.kt` (`createClaim`, edit/delete until reported, `reportedUids`, `COIN_RULE`, `cleanAmount`, `eatToUtc`), `data/Sync.kt` (W2M type `NC_CLAIM` in the report, `applyNcClaim` on the manager phone), `data/CTDatabase.kt` (v8: `nc_claims.claimUid` (unique) and `nc_claims.taskId`), worker screens/demo hooks. The worker types coin, EXACT amount text and the time in EAT; stored as UTC. A claim is idempotent by `claimUid`. Worker status text: not sent / sent, waiting / "Verified by the manager" (status MATCHED or MANUAL).

## Piece 8D — Matching engine + manual override + Review hook (what exists now)
Files (zip: `piece-08d-nc-matching.zip`, sorts after `piece-08c-nc-claims.zip`; NO database change, DB stays v8): `data/NcMatcher.kt` (new), `data/NcRepository.kt` (replaced), `data/Daos.kt` (replaced: `NcDao` gains `rowById`, `claimById`, `rowOfClaim`, `allUnmatchedRows`, `rowCountNow`, `observeAllClaims`, `observeUnmatchedClaimCount`), `data/Sync.kt` (replaced: new M2W type `NC_VERIFIED`), `ui/NcMatchScreens.kt` (new), `ui/NcScreens.kt` (replaced: mode `MATCH`, button "MATCHING: CLAIMS ↔ STATEMENT ROWS"), `ui/ManagerScreens.kt` (replaced: "NC Wallet check" card in `ReviewDetail`), `app/build.gradle.kts` (22 / 0.21.0).
- `NcMatcher.match(claims, rows)` (pure): a claim and a row FIT when coin is equal, amount is EXACTLY equal (`BigDecimal.compareTo`, so 0.50 = 0.5) and `|row time - claim time| <= 120 s`. Only deposits fit (rows with amount <= 0 never do). The fitting claims/rows form a graph; a group of exactly 1 claim + 1 row = CLEAR; any bigger group = AMBIGUOUS (nothing automatic); claims with no fitting row = `noRow`; unmatched rows no claim fits = `spareRows`. `NcMatchReport.stateOf(claimId)` gives CLEAR / AMBIGUOUS / NOT_FOUND. Checked in a Python port (60 s fits, 200 s does not, 2 claims + 1 row and 1 claim + 2 rows are ambiguous, withdrawals and other coins never fit).
- `NcRepository`: `runMatch()` (changes nothing), `confirmPair(claimId, rowId, actor)` (re-runs the matcher inside the transaction, so a stale screen cannot confirm something that changed), `confirmAllClear(actor)` (only clear pairs; the screen asks a second tap), `manualLink(claimId, rowId, note, actor)` (any unmatched claim to any unmatched row, reason of 3+ characters required, status MANUAL, the audit note records whether coin/amount fit and the time gap), `unlink(claimId, note, actor)` (reason required; frees claim and row; status back to UNMATCHED), `reviewLines(sub)` (claims of that worker for that task, or with no task, within 36 h of the submission, in words: matched / manual / one row fits but not confirmed / needs your decision / no matching row / no statement imported). A row can never be matched twice (`matchedClaimId` check) and a linked claim must be unlinked first. Audit entries use entityType `nc_claim` and entityId = `claimUid`, actions `NC_MATCH_CONFIRMED`, `NC_MANUAL_LINK`, `NC_UNLINKED`.
- Screen Manager > NC Wallet > Matching: counts, one shared "Reason" box, clear matches (CONFIRM THIS MATCH per pair, CONFIRM ALL n with a second tap), groups needing a decision (LINK buttons per claim/row), claims with no fitting row (LINK TO A ROW MANUALLY lists the 15 closest unmatched rows), linked claims (UNLINK). Wording says a match is evidence only and the manager decides.
- Review (`ReviewDetail`): card "NC Wallet check" with 🟢 / 🟡 / ⚪ lines and the evidence-only sentence; the Approve / Reject buttons are unchanged and never blocked.
- Worker update `NC_VERIFIED` (M2W, 5C style): body `{id: claimUid, st: MATCHED|MANUAL|UNMATCHED, ver, cn, am}`. `ver` = number of manager changes of that claim (audit count via `SyncDao.ncChanges`), outbox key `NCV:<uid>:<ver>`. The worker phone (`applyNcVerified`) applies it only if `ver` is newer than the one saved in `meta` key `ncv:<uid>`, sets the claim status, and then shows "Verified by the manager" (or back to waiting after an unlink). It travels with the normal worker update (Sync > update for a worker). The decisions on submissions are separate (`DECISION`).
- Limits: matching needs claims AND statement rows on the SAME phone (the manager phone). The time tolerance is a constant (`NcMatcher.TOLERANCE_MS`). Rows below the 100 newest are still matched (the matcher reads all unmatched rows). Not compile-tested (no Kotlin/Gradle in the sandbox). Public names added: NcMatchPair, NcAmbiguous, NcClaimState, NcMatchReport, NcMatcher, NcReviewLine, NcMatchingScreen.

## Piece T1 — Team mining: ledger, shares, split, rates (what exists now)
Files (zip: `piece-08t1-team-ledger.zip`): `data/Team.kt` (new: entities, `TeamDao`, `TeamMath`, `TeamRepository`, `TeamSnapshot`), `ui/TeamScreens.kt` (new: `TeamMiningScreen`), `data/CTDatabase.kt` (replaced: version 9, ONE migration `MIGRATION_8_9` with `team_contributions` and `team_rates`), `data/ServiceLocator.kt` (`Repos.team`), `data/NcRepository.kt` (replaced: hooks), `ui/ManagerScreens.kt` (replaced: route `MgrRoute.Team`, dashboard card "Team mining"), `app/build.gradle.kts` (23 / 0.22.0).
ZIP NAMING RULE for the next team pieces: use `piece-08t2-...`, `piece-08t3-...`, `piece-08t4-...`. The workflow unzips in sorted order; "08t" sorts after `piece-08d-...` and BEFORE `piece-09-...`, so a later Piece 9 can never be overwritten by an older team zip (a name starting `piece-t1` would sort after `piece-09` and overwrite it).
- Contribution = a VERIFIED claim. `NcRepository.link` (clear match, manual link) calls `TeamRepository.onClaimVerified(claim, row, actor)`; `unlink` calls `onClaimUnlinked`. One contribution per claim (`claimId` is unique). Its date for the decay is the STATEMENT row time (`contributedAtUtc`). The UGX value is fixed at verification: `valueUgx = round_half_up(BigDecimal(amountText) x BigDecimal(rate))`; `rateText` and `rateSetAt` are stored on the row and never change. No rate for the coin at that moment -> the row is saved UNPRICED (`valueUgx` null), it does not count, and the screen offers PRICE THEM WITH TODAY'S RATES (`priceUnpriced`). Unlink -> status VOIDED (voidKind UNLINKED); linking again re-activates it with the rate of that day. The manager can also mark a verified deposit "not a team contribution" (VOIDED, voidKind MANAGER, reason required); a new link does NOT bring it back, only RESTORE (reason required, and the claim must still be verified) does.
- Formulas (`TeamMath`, pure, tested in a Python port): `weight = valueUgx * 0.5^(ageDays / halfLifeDays)`; `share = worker weight / total weight`; only ACTIVE + priced rows count. `split(minedUgx, poolPercent)` -> pool = floor(mined x percent / 100), manager = mined - pool (parts always add up). `distribute(pool, shares)` = largest-remainder rounding, ties by worker id, so the paid parts add up exactly to the pool. Checked: 1 UGX -> pool 0 / manager 1; 100 UGX three equal shares -> 34/33/33; a contribution 180 days old weighs 0.5, 360 days 0.25.
- Settings in `meta` (audit-logged on change): `team:halfLifeDays` (default 180, allowed 7..3650), `team:poolPercent` (default 70, allowed 1..99). Rates: `team_rates` keeps the full history (coin, exact text, setAt, setBy); `TeamRepository.setRate`, `currentRate`, `rateAt(coin, time)`.
- `TeamRepository.snapshot()` -> `TeamSnapshot(shares[], totalAllTimeUgx, totalWeight, unpricedCount, ...)` is what T2 (worker view) and T3 (mined amounts, goals) must use. `observeOfWorker(workerId)` lists one worker's contributions for T2.
- Screen Manager > Team mining: settings, rates (latest per coin + history, "set by the manager on <date>"), unpriced banner, shares per worker (percent, all-time contributed, counts towards share, number of contributions), team totals, split calculator (type a mined UGX amount, see pool / manager / each worker, labelled ESTIMATE), contribution ledger (newest 60) with the rate used and the void/restore buttons. Honest wording everywhere: estimates, rates typed by the manager, no promise of payment.
- NOT in T1 (by plan): daily mined amounts, weekly hash goal, gifts, rig ledger (T3 adds their tables with its own migration v10); worker screens and the signed M2W update that carries team figures to worker phones (T2, new type in the 5C style); leaderboard, levels, streaks, badges (T4).
- Known limit: shares are computed on the MANAGER phone only (it holds the claims and statement rows); workers will see them after a T2 update. Not compile-tested (no Kotlin/Gradle in the sandbox). If Actions fails, paste the error log (reminder: Room/KSP generates Java, so never name a Kotlin parameter with a Java keyword such as `new`).

## Piece T2a + T2b — Team mining worker view (what exists now)
T2a (`piece-08t2a-team-worker-sync.zip`): `data/TeamWorker.kt`, `data/Sync.kt` (new M2W type `TEAM_VIEW`, part of the normal worker update; outbox key `TV:<worker>:<rev>`), `ui/WorkerTeam.kt` (`WorkerTeamCard`, in the real worker home only), `ui/WorkerScreens.kt` (one line). Body: rev, at, hl, pp, n, ta, tw, mw, ma, ct[] (this worker's 40 newest contributions), more. `rev` rises only when the ledger/settings changed or a new Monday began; same ledger = byte-identical message. Worker keeps the newest in `meta` (`teamview:json:<worker>`, `teamview:rev:<worker>`); manager counters `team:vfp/vrev/vat:<worker>`. No schema change.
T2b (`piece-08t2b-team-daily-whatif.zip`, after T2a; app 25 / 0.24.0, still DB v9): `data/Team.kt` (`TeamRepository.dailyMinedUgx()`, `setDailyMined(ugx|null, actor)`, `meta` key `team:dailyMinedUgx`, audit `TEAM_DAILY_MINED_CHANGED`, constant `KEY_DAILY_MINED`), `ui/TeamScreens.kt` (Settings card: "Team's estimated mined amount per day in UGX", empty = none), `data/TeamWorker.kt` (optional `md` in the message and in the fingerprint, so changing it makes a new `rev`; `TeamView.dailyMinedUgx`, `dailyPoolUgx()`, `expectedDailyUgx()` = floor(pool x share), `shareIfAdded(x)`, `expectedDailyIfAdded(x)`), `ui/WorkerTeam.kt` (expected payout per day, "What if I contribute more?", team capacity totals).
- Pool = `TeamMath.split(md, poolPercent).workersPoolUgx`; worker estimate = floor(pool x share) (the worker phone knows only its own share, so parts may not add up exactly; the manager's calculator uses largest-remainder).
- What-if: share = (myWeight + X) / (teamWeight + X); assumes nobody else changes, ignores decay. Labelled estimate.
- Capacity: only contributed totals and contributors. Hash-rate capacity and the weekly goal are NOT built (need T3's rig ledger); the card says so.
- Not compile-tested. If Actions fails, paste the error log.

## Piece T3a — Real daily mined amounts (what exists now)
Files (zip: `piece-08t3a-mined-days.zip`, sorts after 08t2b; app 26 / 0.25.0; DB v10 with `MIGRATION_9_10` = ONE new table): `data/Team.kt` (replaced: entity `TeamMinedDayEntity` table `team_mined_days` [day yyyy-MM-dd EAT = primary key, minedUgx, note, setBy, setAt]; `TeamDao` mined-day queries; `TeamMath.averageDaily`; `TeamRepository.setMinedDay(day, ugx, note, actor)` [replaces the same day, audit `TEAM_MINED_DAY_SET` with old and new value; no future days; not older than 400 days; 0 allowed], `deleteMinedDay`, `observeMinedDays`, `estimateDaily`; companion `TeamRepository.estimateDaily(db, now)`), `data/TeamWorker.kt` (replaced: `md` now comes from `estimateDaily`), `data/CTDatabase.kt` (replaced: v10), `ui/TeamScreens.kt` (replaced: new card "Daily mined amounts"), `app/build.gradle.kts`.
- The figure workers see (`md` in TEAM_VIEW) = AVERAGE of the amounts the manager ENTERED for the last 7 East Africa days, today included (days without an entry are skipped, not counted as zero). If NO day was entered in that window the old typed estimate (T2b, `meta` `team:dailyMinedUgx`) is used. If days were entered and average 0, workers get no estimate. The message format did not change, so T2a/T2b worker phones work unchanged; a new average makes a new `rev` through the existing fingerprint.
- Manager screen: figure being sent, mined this week (Mon–Sun) and last week, day entry (TODAY / YESTERDAY buttons, UGX, note), CONVERT WITH MY RATE (coin + amount x the manager's current rate -> UGX, to be checked then saved), list of the 14 newest days with SHOW SPLIT (pool, manager, each worker with `TeamMath.distribute` and TODAY'S shares, labelled estimate), EDIT, DELETE. The old typed field is now labelled a fallback.
- Limits: the amounts are typed by the manager (the app cannot see what was mined); the split per past day uses today's shares, not the shares of that day; nothing here is a payment or a promise.
- Not compile-tested (no Kotlin/Gradle in the sandbox). Brace balance, the migration SQL and the 7-day window logic were checked. If Actions fails, paste the error log.

## Piece T3b — Weekly hash goal + rig ledger (what exists now)
Files (zip: `piece-08t3b-goal-rigs.zip`, sorts after 08t3a and before piece-09; app 27 / 0.26.0; DB v11 with `MIGRATION_10_11` = TWO new tables, nothing existing touched): `data/TeamGoals.kt` (new), `ui/TeamGoalScreens.kt` (new), `data/CTDatabase.kt` (replaced: v11, entities `TeamGoalEntity`, `TeamRigEntity`, `teamGoalDao()`), `data/ServiceLocator.kt` (`Repos.teamGoals`), `data/TeamWorker.kt` (replaced: optional goal fields in TEAM_VIEW), `ui/WorkerTeam.kt` (replaced: progress bar), `ui/TeamScreens.kt` (replaced: one line, `TeamGoalSection(repos, actor)` inserted above the "Daily mined amounts" card), `app/build.gradle.kts`.
- Tables: `team_goals` (`weekKey` = Monday date in East Africa, primary key; `targetText` exact decimal; `unit`; `note`; `setBy`; `setAt`) and `team_rigs` (`id`, `name`, `hashText` exact decimal, `unit`, `costUgx` nullable, `boughtAt`, `fundedNote`, `status` ACTIVE|SOLD|BROKEN, `soldAt` = when it stopped being active, `setBy`, `createdAt`; index on status). Units: `HashUnits.ALL` = H/s KH/s MH/s GH/s TH/s PH/s EH/s. Numbers: up to 12 digits and 6 decimals, more than zero (`TeamGoalMath.HASH_RULE`).
- Rules (`TeamGoalRepository`): a goal is for ONE week given as its Monday; only this week or later (at most 8 weeks ahead); saving the same week again replaces it. Rigs are never deleted (status changes only; a broken rig can be made active again). ONE unit at a time: adding a rig, re-activating one, or setting a goal in a different unit than the ACTIVE rigs is refused with a clear message. Capacity = sum of ACTIVE rigs in the goal's unit (`TeamGoalMath.capacity`); progress = capacity / target (BigDecimal, 6 decimals). Audit log: `TEAM_GOAL_SET` (before -> after), `TEAM_GOAL_DELETED`, `TEAM_RIG_ADDED`, `TEAM_RIG_STATUS`, `TEAM_SHOW_TOP_CHANGED`.
- Consent switch `meta` key `team:showTop` (default OFF): when ON and the goal is reached, the worker message carries the first name + last initial of the top 3 shares (names only, no amounts).
- TEAM_VIEW (optional fields, old phones ignore them, new phones accept old messages): `gt` target, `gu` unit, `cap` capacity in that unit, `gw` week key, `gn` note, `tp` top names. The goal block is all-or-nothing on the worker phone (if `gt` is present, `gu`, `cap` and `gw` must be valid or the whole message is refused). The goal is part of the fingerprint, so any change (target, capacity, note, names) makes a new `rev` through the existing mechanism. A worker with no contributions still gets the message when a goal exists (before, no contributions meant no message).
- Manager screen (Team mining): capacity now + progress bar of this week's goal, week choice (This week / Next week), target, unit chips, note, SAVE GOAL, list of goals with REMOVE (two taps), the top-contributors switch, the rig ledger (status buttons) and an add-rig form (name, hash rate, unit, cost, bought on, what paid for it).
- Worker card (`WorkerTeamCard`): the "capacity will appear here" sentence is replaced by the weekly goal block: target, gold progress bar, capacity and percent, the note, and when reached a celebration line and the top names if the manager allowed them. Without a goal it says the manager has not set one.
- Limits: all figures are typed by the manager; the app cannot see machines or hash rate. A goal only counts the rigs in its own unit. Nothing here is a payment or a promise.
- Not compile-tested (no Kotlin/Gradle in the sandbox). Brace balance and the migration SQL against the entity columns were checked, new public names were checked against the whole project. If Actions fails, paste the error log (reminder: Room/KSP generates Java, so no Kotlin parameter may be named with a Java keyword such as `new`).

## Piece T3c — Gifts (what exists now)
Files (zip: `piece-08t3c-gifts.zip`, sorts after 08t3b and before piece-09; app 28 / 0.27.0; DB v12 with `MIGRATION_11_12` = ONE new table): `data/TeamGifts.kt` (new), `ui/TeamGiftScreens.kt` (new), `data/CTDatabase.kt` (v12, `TeamGiftEntity`, `teamGiftDao()`), `data/ServiceLocator.kt` (`Repos.teamGifts`), `data/TeamWorker.kt` (optional `gf` in TEAM_VIEW, `TeamViewGift`, `TeamView.gifts`, `activeMultiplier`, `expectedDailyWithGiftUgx`), `ui/WorkerTeam.kt` (gift lines), `ui/TeamScreens.kt` (one line: `TeamGiftSection(repos, actor)` after the goal section), `app/build.gradle.kts`.
- Table `team_gifts`: `id`, `workerId`, `kind` MULTIPLIER|FIXED (`GiftKind`), `multiplierText` (two decimals, e.g. 1.10), `fixedUgx`, `fromDay`/`toDay` (yyyy-MM-dd East Africa; same day for FIXED), `reason`, `createdBy`, `createdAt`, `voidedAt`, `voidedBy`; index on workerId.
- Rules (`TeamGiftRepository`): reason REQUIRED, 5 to 160 characters. Multiplier 1.01 to 3.00; the period cannot start in the past, can start at most 60 days ahead, lasts at most 90 days; only ONE multiplier per worker at any time (an overlapping one is refused: void it first). Fixed gift 1 to 1,000,000,000 UGX, dated today. Only active workers. A gift is never deleted, only voided. Audit log: `TEAM_GIFT_GIVEN` (worker name, figures, reason) and `TEAM_GIFT_VOIDED`.
- Effect: a multiplier changes ONLY the worker's expected DAILY payout estimate, only on the days of the gift (`expectedDailyUgx x multiplier`, rounded down, computed on the worker phone with the worker's East Africa date); contributions, shares and the team figures are never changed. A fixed gift is just a one-time extra figure shown as a gift.
- TEAM_VIEW: optional `gf[]` of at most 10 objects (`id` "g<id>", `k` M|F, `m` or `f`, `a` first day, `b` last day, `r` reason, `at`), only THIS worker's gifts that are not voided and have not ended more than 7 days ago. The gift list is in the fingerprint, so giving, voiding or an old gift dropping out makes a new `rev` through the existing mechanism; a worker with only gifts still gets the message. On the worker phone each gift entry is validated separately (a broken one is skipped, the rest is used).
- Worker card: "With the manager's gift (xM today): about UGX ..." when a multiplier is active, plus a "Gifts from the manager" list (what, status active/starts/ended, "Reason: ...") and the honest sentence that gifts are the manager's own choice, not promised, can be taken back and do not change the share.
- Manager screen (Team mining): choose a worker (chips), Multiplier or Extra amount, fields, reason, GIVE GIFT, and the list of the 20 newest gifts with status (ACTIVE / UPCOMING / ENDED / GIVEN / VOIDED) and a two-tap VOID.
- Limits: the multiplier is judged on the worker phone's date, so a wrong phone clock shows it early or late; amounts are estimates; nothing here pays anyone, the manager still pays by hand.
- Not compile-tested (no Kotlin/Gradle in the sandbox). Brace balance and the migration SQL against the entity columns were checked, new public names were checked against the whole project. If Actions fails, paste the error log.

## Piece T4 — Levels, rank, streaks, badges, leaderboard, quiet workers, weekly summary (what exists now)
Files (zip: `piece-08t4-leaderboard.zip`, sorts after 08t3c and before piece-09; app 29 / 0.28.0; NO database change): `data/TeamBoard.kt` (new: `TeamLevels`, `TeamStreaks`, `TeamBadges`/`Badge`/`BadgeInput`, `BoardRow`, `QuietRow`, `TeamBoardRepository`), `ui/WorkerTeamBoard.kt` (new: `WorkerTeamBoardCard`, `TeamBoardContent`), `ui/TeamBoardScreens.kt` (new: `TeamBoardSection`), `data/TeamWorker.kt` (replaced), `data/ServiceLocator.kt` (`Repos.teamBoard`), `ui/TeamScreens.kt` (one line: `TeamBoardSection` after the gift section), `ui/WorkerScreens.kt` (card in the real home + demo), `ui/WorkerDemo.kt` (`teamView()` made-up figures), `app/build.gradle.kts`.
- Levels follow the ALL-TIME verified deposits (UGX value fixed at verification), so a level never goes down: Starter 0, Bronze from 1, Silver 100,000, Gold 500,000, Platinum 2,000,000, Diamond 10,000,000 (constants in `TeamLevels.ALL`).
- Streaks: deposit weeks in a row (Mon–Sun, East Africa; the current week may still be empty without breaking the streak) and approved-task days in a row. Both are computed ON THE WORKER PHONE from the team figures it already has and its own submissions.
- Badges (15, `TeamBadges.ALL`): first deposit, five deposits, Silver/Gold/Platinum/Diamond, 4 and 12 weeks in a row, 7 task days, 30 and 100 tasks, Top 3, Rising, Goal week, Gift. Locked badges show 🔒; tapping one shows how to get it.
- TEAM_VIEW got two OPTIONAL fields (old phones ignore them, new phones accept old messages): `rk` = my rank (by share, 1-based, only if I have a priced active deposit) and `lb[]` = top 5 `{nm, lv}` (first name + last initial + level index) ONLY if the manager switched it on (`meta` `team:showBoard`, default OFF, audit `TEAM_SHOW_BOARD_CHANGED`). Rank and board are in the fingerprint, so a change makes a new `rev`; ranks move only when the ORDER changes, so the daily decay alone does not create new revs. The worker phone remembers the previous rank (`prk` in the stored JSON) to show ▲/▼.
- Worker card "My level, streaks and badges": level + progress bar, rank + movement, two streak tiles, badge grid, "My weeks" (this week / last week: approved tasks with UGX, verified deposits with UGX), the top 5 when allowed. Honest wording: motivation only, estimates, no promise of payment.
- Manager (Team mining, new card): ranking (rank, name, level, share, all-time, deposits, 🔥 week streak, last deposit), the consent switch, QUIET WORKERS (active workers with no proof for 7 days, or no verified deposit for 4 weeks, or never), and WEEKLY SUMMARY TEXT per worker (last Mon–Sun: tasks approved + UGX, rejected/pending, deposits + UGX, level, rank + share, streak, team goal %) with COPY and SHARE (Android share sheet). Nothing is sent by the app; Piece 9 can send these by Telegram.
- Limits: rank/level figures reach a worker only with the manager's next update; the worker's deposit count uses the 40 newest deposits plus `more`; the "Rising" badge needs two updates with different ranks. Not compile-tested (no Kotlin/Gradle in the sandbox); brace balance and public-name uniqueness were checked. If Actions fails, paste the error log.

## Piece 9A — Telegram, manager side (what exists now)
Files (zip: `piece-09a-telegram-reader.zip`; sorts after `piece-08t4-leaderboard.zip` and before `piece-10-...`): `data/Telegram.kt` (new), `ui/TelegramScreens.kt` (new), `data/CTDatabase.kt` (v13, `MIGRATION_12_13`: tables `tg_chats`, `tg_inbox`), `data/ServiceLocator.kt` (`Repos.telegram`), `ui/ManagerScreens.kt` (route `MgrRoute.Telegram`, dashboard card "Telegram" with a badge for waiting items, the reading loop started after sign-in, `syncInitial`), `ui/SyncScreens.kt` (`ManagerSyncScreen(..., initialText: String? = null)` processes that text when opened), `app/src/main/AndroidManifest.xml` (adds `INTERNET`, the only change), `app/build.gradle.kts` (30 / 0.29.0).
Design (no server, free): the manager makes a bot with @BotFather and pastes the token in Manager > Telegram. ONLY the manager phone holds the token. A worker puts a signed CT1 message into the bot's chat with the existing SHARE button (Telegram share target), or sends a `.txt`/`.ctt` file with it. The manager app long-polls `getUpdates` (25 s) while a manager is signed in; leaving the manager area stops the loop (`TelegramRepository.pollLoop`, one loop at a time through a Mutex; backs off 15 s after an error; PAUSE button).
- `TelegramClient` — plain `HttpURLConnection` (no new library): `getMe`, `deleteWebhook` (run when the token is saved, because a webhook blocks getUpdates), `getUpdates`, `downloadFile` (getFile + download, size capped; bots can download up to 20 MB, text files are capped at 3 MB). Errors become words (401 bad token, 409 another reader or webhook, 429 slow down, no internet, timeout). The token is NEVER written to the audit log or shown again.
- `TelegramRepository.connect/disconnect/setPaused/pollLoop`; settings in `meta`: `tg:token` (empty string = removed), `tg:bot`, `tg:offset` (saved after each update), `tg:paused`.
- Each update: only PRIVATE chats; text or caption plus a text document are read; no `CT1.` -> ignored in silence (a file from an already linked chat is logged as FILE). Everything with `CT1.` goes through `SyncRepository.receiveAll(text, actor)` — identical rules to copy/paste (HMAC, replay protection, idempotent imports, manager approval for ENROLL and RESET_REQ). Results are stored in `tg_inbox`: ENROLL / RESET_REQ / HELD (worker not connected yet) = PENDING with the signed text kept; OK / FAIL / FILE = log rows (last ~300 kept). The same message sent twice makes one pending row. The screen's button "OPEN IN SYNC" calls `markHandled` (drops the stored text) and opens Sync with that text (`ManagerSyncScreen(initialText)`); the manager approves there exactly as before. If they leave without approving, the worker sends again (said on screen).
- Chat links (`tg_chats`, one row per worker): set only when a NEW signed message from that worker was accepted (its seq is higher than anything seen before this paste, and the HMAC verifies). So a copied old message sent from another account cannot move a worker's chat. A changed chat is audited (`TG_CHAT_CHANGED`). Audit actions: `TG_CONNECTED`, `TG_DISCONNECTED`, `TG_PAUSED`, `TG_RESUMED`, `TG_CHAT_BOUND`. Used by 9B to send back.
- Screen: connect steps, bot status (reading / paused / problem, last successful check), COPY THE BOT LINK, waiting items, linked chats, recent activity, honest limits.
Limits: Telegram bot chats are not end-to-end encrypted (messages stay signed but readable by Telegram); the token is stored in the app database, not the Keystore (Piece 11 backups must encrypt it); only one program can read a bot; reading happens only while the manager area is open and Telegram keeps unread messages about 24 h; no background service yet (a later piece may add WorkManager). Workers must open the bot link once and press START before they can write to it. Nothing is sent TO workers yet (9B). Not compile-tested here (no Kotlin/Gradle in the sandbox); if Actions fails, paste the error log (Room/KSP rule: never use a Java keyword such as `new` as a parameter name).
Plan for the rest: 9B sends M2W updates/replies/announcements to the linked chat (`sendMessage` for short codes, `sendDocument` of a `.txt` for long ones) and adds an Android share target so a worker taps a received file/message to import it into the app; 9C original screenshots (`sendDocument` from the worker, download on the manager phone, compare SHA-256 with the report, extend Piece 6 verdicts); 9D support chat, weekly summaries (T4 text) and notifications.

## Piece 9B — split in two zips (what exists now)
Zip names (sort after `piece-09a-telegram-reader.zip`, before `piece-10-...`): `piece-09b-1-telegram-send.zip`, `piece-09b-2-worker-import.zip`. Upload ALL of 9B-1, 9B-2, 9B-3 and 9C-1, 9C-2 (9C-1's Telegram.kt calls code from 9B-3, so skipping one breaks the build); 9B-2 needs 9B-1 (it overwrites `ui/SyncScreens.kt` with a version that has the same Telegram buttons plus the reset-code auto-fill). No database change.

### 9B-1 — manager sends on Telegram (app 31 / 0.30.0)
Files: `data/Telegram.kt`, `data/Sync.kt`, `ui/SyncScreens.kt`, `ui/TelegramScreens.kt`, `app/build.gradle.kts`.
- `TelegramClient.sendMessage` (link preview off) and `sendDocument` (multipart `.txt`); 429 answers wait `retry_after` and retry. `TgResult.Err(code, message, retryAfterSec)`. New log kinds `SENT`, `SEND_FAIL`.
- `TelegramRepository`: `sendUpdate(workerId, everything, actor)`, `sendAllPending(actor)`, `approveResetAndSend(inboxId, actor)`, `sendCodes` (up to `MAX_TEXT_PARTS = 3` messages of at most `MAX_TEXT_LENGTH = 3900` characters; longer = one `.txt` document), `canSendTo`, `chatOf`. Results: `TgSendReport(ok, message, sentParts, totalParts)`, `TgBulkLine`, `TgResetResult(sent, message, manualCode)`.
- Retry-safe numbers: `SyncRepository.rollbackUpdate` (new `ReportResult.counterBefore` / `newKeys`, DAO `deleteOutboxKeys`) undoes an update's numbers when NOTHING was delivered, so "new only" never loses items. After a partial delivery nothing is rolled back (the worker may hold some numbers); the manager is told to use SEND EVERYTHING AGAIN. Audit `SYNC_UPDATE_ROLLED_BACK`.
- Manager Sync screen, per worker with a linked chat: SEND UPDATE ON TELEGRAM (n new), SEND EVERYTHING AGAIN, SEND RESET CODE, SEND JOIN CODE (needs a confirm tap: the code carries the worker's secret key and Telegram can read it). Reset-request card: APPROVE AND SEND ON TELEGRAM (the approval tap stays on purpose); if sending fails the code is copied so it can be sent by hand.
- Telegram screen: per chat "not sent yet" count + SEND UPDATE, SEND UPDATES TO ALL, APPROVE AND SEND on reset requests in the inbox, log icons for sent/failed.

### 9B-2 — worker receives (app 32 / 0.31.0)
Files: `AndroidManifest.xml` (MainActivity `singleTop` + `ACTION_SEND text/plain` filter), `MainActivity.kt`, `ui/ShareInbox.kt` (new), `ui/WorkerIncoming.kt` (new), `ui/CTApp.kt`, `ui/ManagerScreens.kt`, `ui/WorkerScreens.kt`, `ui/SyncScreens.kt`, `app/build.gradle.kts`.
- Android share target: Telegram > message or `.txt` file > Share > CT Team. `MainActivity.readShared` reads the text and/or a shared text file (max 300 KB, off the main thread) into `ShareInbox` (memory only, keeps it only if it contains a CT1 message).
- Routing by direction: M2W goes to the worker side, W2M to the manager Sync screen (processed as if pasted). Mixed text is not auto-routed.
- `WorkerIncomingCard` (top of the worker home): imports a shared update at once through `receiveFromManager` (same checks as paste) and shows a short result. It also offers USE IT / NOT NOW when the clipboard holds an update (checked at most once a minute; nothing happens without a tap).
- A shared RESET code for that worker opens the recovery screen and fills the box.

### 9B-3 — announcements (app 33 / 0.32.0)
Zip: `piece-09b-3-announcements.zip`. Files: `data/Sync.kt`, `data/Telegram.kt`, `ui/TelegramScreens.kt`, `ui/WorkerScreens.kt`, `ui/WorkerNotices.kt` (new), `app/build.gradle.kts`. No database change.
- New M2W message type `NOTICE` (body: id, text up to 600 characters `NOTICE_MAX`, time). `SyncRepository.makeNotice(workerId, text, actor)` signs one per worker with the normal next message number; `releaseSeq` gives the number back when nothing was delivered (no gap warning on the worker phone). Audit `SYNC_NOTICE_MADE` (text is not logged) / `NOTICE_RECEIVED`.
- `TelegramRepository.sendAnnouncement(text, actor)` loops over all linked chats. Manager > Telegram: "Announcement to all workers" card (needs a second tap to send; the box clears only if all were sent).
- Worker: received like any update (Share or USE IT); stored in `meta` key `notices:<workerId>` (newest first, last 20 kept, duplicates ignored by id); `WorkerNoticesCard` on the home screen shows the latest 5 with a CLEAR button. Old app versions answer "needs a newer version" and the message can be sent again.
- Limits: announcements only go to workers who have a linked chat; Telegram can read the text (signed, not secret). No announcement history on the manager side yet (the Telegram activity log shows each send).

### Limits (both)
Telegram bot chats are not end-to-end encrypted: the messages are signed but readable by Telegram, and ENROLL/JOIN codes contain the worker's secret key. The worker must press START in the bot once. Sending only happens while the manager app is open and online. Not compile-tested here; if Actions fails, paste the error log (Room/KSP: no Java keywords as parameter names).
Still to do: nothing in piece 9 (9D-2b and 9D-3 are done).

## Piece 9C-1 — original pictures on Telegram (what exists now)
Zip: `piece-09c-1-original-pictures.zip` (sorts after `piece-09b-3-announcements.zip`). Files: `data/Telegram.kt`, `data/ServiceLocator.kt` (`Repos(db, filesDir)`; `TelegramRepository` gets the folder `filesDir/originals`), `ui/TelegramScreens.kt`, `app/build.gradle.kts` (34 / 0.33.0). No database change.
- Worker: in Telegram sends the screenshot to the bot with paperclip > File (NOT as a photo: Telegram shrinks photos and the fingerprint would never match). A photo is answered in the manager's log with that advice.
- Manager phone (reading loop, only from a chat linked to a worker; strangers ignored): picture documents (image/* or .png .jpg .jpeg .webp .heic, up to 20 MB, the Telegram bot limit) are downloaded, SHA-256 computed HERE, saved as `originals/<sha>.img` (stops saving at 400 MB total; the result is still recorded), and kept in `meta` key `orig:<sha>` (JSON: w worker, n file name, s size, at time, f saved?). Read with `TelegramRepository.originalOf(sha)` and `originalFile(sha)`.
- Compared at once with `proofs.sha256`: same worker = log "Original matches the report"; the hash is in another worker's report = red log "File belongs to another worker's report"; no report has it yet = saved and logged (it will match after the report is imported; the lookup is by hash, so order does not matter). Audit `TG_ORIGINAL_MATCH`, `TG_ORIGINAL_OTHER_WORKER`, `TG_ORIGINAL_UNMATCHED`. New log kinds `ORIGINAL`, `ORIGINAL_BAD`.
- Honest limit: a match proves the file is the SAME one the report names. It does not prove the picture is genuine (an edited picture would match its own hash). 9C-2 therefore re-derives dHash and hidden tags from the saved original on the manager phone and compares them with what the report claimed.

## Piece 9C-2 — originals in the review screen (what exists now)
Zip: `piece-09c-2-review-originals.zip` (sorts after `piece-09c-1-original-pictures.zip`). Files: `ui/OriginalCheck.kt` (new, `OriginalProofBlock`), `ui/ManagerScreens.kt` (one line in `ReviewDetail`: the block under each screenshot with a fingerprint), `app/build.gradle.kts` (35 / 0.34.0). No database change, no Telegram.kt change.
- Not received: orange "Original: not received yet" + ASK FOR THE ORIGINAL ON TELEGRAM (needs a linked chat; plain message through `sendCodes`, not signed: it asks for the file as a FILE, not a photo).
- Received (`originalOf(sha)`): green "very same file as in the report"; red line if it came from another worker's chat. If the file is saved, `ProofAnalyzer.dHash` and `ProofAnalyzer.tags` are run again on the manager phone and compared with the report's `dhash` / `metadataTags` (different = red "✖ ... DIFFERENT from what was reported"; report without dHash = nothing to compare). SHOW THE ORIGINAL opens it down-scaled (longest side about 1400 px) / HIDE.
- Evidence only: nothing changes the verdict or decides; honesty text stays (a match does not prove the picture is genuine).
- Still not done: a worker-side hint inside the app that says "send the original as a file" (the ask message says it); automatic ask for every doubtful proof; 9D.

## Piece 9D-1 — weekly summaries on Telegram (what exists now)
Zip: `piece-09d-1-weekly-summaries.zip` (sorts after `piece-09c-2-review-originals.zip`). Files: `ui/TeamBoardScreens.kt`, `app/build.gradle.kts` (36 / 0.35.0). No database change, no Telegram.kt change (uses `TelegramRepository.sendCodes`).
- Manager > Team > "Weekly summary text": pick a worker, the existing COPY / SHARE stay; new SEND ON TELEGRAM (button reads SEND AGAIN when this week's was already sent) for workers with a linked chat; and SEND LAST WEEK'S SUMMARY TO ALL ON TELEGRAM (n not sent this week), two taps to confirm. The text is `TeamBoardRepository.weeklySummary` (last Monday-Sunday), plain text, NOT signed and not imported by the worker app (just a chat message).
- "Sent this week" is kept in `meta` key `tg:weekly:<workerId>` = the week key of the day it was sent. Failed sends are not marked, so they are offered again. The activity log in Manager > Telegram shows each send (`TG_SENT` audit).
- Not done: scheduled/automatic weekly sending (needs 9D-3 background work); the SPEC's weekly file of all transactions.

## Piece 9D-2a — support chat, data and worker side (what exists now)
Zip: `piece-09d-2a-support-worker.zip` (sorts after `piece-09d-1-weekly-summaries.zip`). Files: `data/Daos.kt` (SupportDao: new queries only, no schema change), `data/Sync.kt`, `data/Telegram.kt`, `ui/Support.kt` (new, `WorkerSupportCard`), `ui/WorkerScreens.kt` (card on the worker home under the team board), `app/build.gradle.kts` (37 / 0.36.0). The existing table `support_messages` is used as it is; the shared message id is kept in its `telegramMessageId` column (same id on both phones, so nothing is stored twice).
- New message types: `SUPPORT` (W2M; body id, tx up to 600 characters `NOTICE_MAX`, at) and `SUPPORT_REPLY` (M2W, same body). SUPPORT is a normal report item (`SUP:<id>` in the outbox): MAKE REPORT sends it with submissions etc., resends are harmless, the manager ignores an id it already has. Added to `collectItems` and `unsentCount`.
- Worker: "Ask the manager" card (OPEN CHAT / thread of the last 8 / SAVE MESSAGE / CLOSE; max 20 unsent; opening marks the manager's replies read). `SyncRepository.askSupport`, `unsentSupportCount`.
- Manager phone: `applySupport` stores the question as `TO_MANAGER` (unread). `makeSupportReply(workerId, text, actor)` signs a reply (next M2W number), `saveSupportReply` adds it to the thread AFTER it really left (sent on Telegram or copied) and marks the worker's messages read. `TelegramRepository.sendSupportReply` does it all for a linked chat (gives the number back if nothing was delivered, `releaseSeq`). The worker gets the reply like any update (Share / USE IT / paste) and `applySupportReply` adds it to their chat.
- Not in this zip: the manager screen to read and answer (9D-2b, see next section).

## Piece 9D-2b — support chat, manager screen (what exists now)
Zip: `piece-09d-2b-support-manager.zip` (sorts after `piece-09d-2a-support-worker.zip`). Files: `ui/SupportManager.kt` (new, `ManagerSupportScreen(repos, actor, onBack)`), `ui/ManagerScreens.kt` (route `MgrRoute.Support`, dashboard card "Support" with the unread number from `SupportDao.observeUnreadForManager()`), `app/build.gradle.kts` (38 / 0.37.0). No database change.
- List: one card per worker, newest first, "● N new" in gold, last line + time. Opening a thread marks that worker's messages read (`markManagerRead`).
- Thread: all messages oldest first, reply box (600 characters). SEND ON TELEGRAM (only when the worker has a linked chat and the bot is connected; `TelegramRepository.sendSupportReply`) or COPY REPLY CODE (`SyncRepository.makeSupportReply` + `saveSupportReply`; the reply joins the thread only after it was really sent or copied).

## Piece 9D-3 — notifications (what exists now)
Zip: `piece-09d-3-notifications.zip` (sorts after `piece-09d-2b-support-manager.zip`). Files: `data/Notify.kt` (new: `CTNotify`, `ManagerCheckWorker`, `WorkerReminderWorker`), `ui/NotifyScreens.kt` (new: `ManagerNotifyScreen`, `WorkerReminderCard`), `data/Telegram.kt` (+ `peekWaiting()`), `ui/ManagerScreens.kt` (route `MgrRoute.Notify`, dashboard card "Notifications"), `ui/WorkerScreens.kt` (reminder card on the worker home, under the support card), `AndroidManifest.xml` (`POST_NOTIFICATIONS`), `app/build.gradle.kts` (39 / 0.38.0, new dependency `androidx.work:work-runtime-ktx:2.9.0`). No database change; settings live in `meta` (keys in `CTNotify`).
- Manager phone: off until switched on (Manager > Notifications). WorkManager periodic job every 15 min with internet (`ManagerCheckWorker`). It counts pending submissions, unread support messages and bot messages waiting. `TelegramRepository.peekWaiting()` only LOOKS (getUpdates with timeout 0, the saved offset is not moved, nothing is processed while signed out); it returns null if the app's own `pollLoop` is running (`loopLock.tryLock`) so two readers never clash. One notification, numbers only (no names/text); shown again only when a number goes UP since the last check (`notify:mgr:last`). Buttons: CHECK NOW, SEND A TEST NOTIFICATION, phone notification settings, app (battery) settings.
- Worker phone: reminder card on the home screen (`WorkerReminderCard`). `WorkerReminderWorker` every ~6 h, no internet needed: if `SyncRepository.unsentCount(workerId)` > 0, one quiet notification at most every 20 h. Replies and updates from the manager already ring through Telegram itself, so no extra worker alert for those.
- Android 13+ asks for the notification permission when the person turns a switch on.
- Limits: Android may delay or stop background work on battery saver / aggressive phones (Tecno, Infinix, Xiaomi, Samsung): set the app to Unrestricted. Telegram keeps waiting messages about 24 h, so a manager who never opens the app for a day can miss them (workers can resend). Not compile-tested here.

## Piece 10A — Learn & Earn, manager side (what exists now)
Zip: `piece-10a-learn-manager.zip` (sorts after `piece-09d-3-notifications.zip`). Files: `data/Learn.kt` (new `LearnRepository`: `save`, `setPublished`, `delete`; limits title 3–80, text up to 3000, category 30, links https only up to 300), `ui/LearnManager.kt` (new `ManagerLearnScreen`), `data/Daos.kt` (ContentDao: `observeAllLearn`, `learn(id)`, `deleteLearn`), `data/ServiceLocator.kt` (`repos.learn`), `ui/ManagerScreens.kt` (route `MgrRoute.Learn`, dashboard card "Learn & Earn"), `app/build.gradle.kts` (40 / 0.39.0). No schema change: the existing table `learn_content` is used (`version` rises on every content change or re-publish; `published` false = draft).
- Audit: LEARN_CREATED / LEARN_EDITED / LEARN_PUBLISHED / LEARN_HIDDEN / LEARN_DELETED.
- 10B (sending) is done, see "Piece 10B" below.

## Piece 10B — Sending lessons to workers (what exists now)
Zip: `piece-10b-learn-send.zip` (sorts after `piece-10a-learn-manager.zip`; no database change, DB stays v13; app 41 / 0.40.0). Files (all replaced, patched with assert-once edits): `data/Sync.kt`, `data/Telegram.kt`, `ui/LearnManager.kt`, `app/build.gradle.kts`.
- New manager -> worker message type `SyncRepository.TYPE_LEARN = "LEARN"` (added to `WORKER_TYPES` and to the `receive` dispatch). Body: `id` (lesson id, `L-...`), `vs` version, `pb` 1 = published / 0 = take back, `tt` title (3-80), `ct` category (30), `tx` text (3000, empty when pb = 0), `im` picture link, `vd` video link (https only, 300), `at`.
- Manager phone: `SyncRepository.makeLearn(workerId, lessonId, actor)` signs the SAVED lesson for one worker (own message number, audit `SYNC_LEARN_MADE`). `TelegramRepository.sendLesson(lessonId, actor)` does it for every linked chat (one message per worker, long ones go as one text file like every other message, a message number is handed back with `releaseSeq` when nothing at all reached the chat). Audit on the worker phone: `LEARN_RECEIVED` / `LEARN_TAKEN_BACK`.
- Worker phone: `applyLearn` is version-aware: a lesson is used only if its version is HIGHER than the one in `learn_content` ("already up to date" otherwise, still counted as received). A take-back (hidden or draft lesson) keeps a record with `published = false` and empty text, so a late older message cannot bring the lesson back; 10C must show only `published = true` rows. Invalid messages (bad id, short title, empty text for a published lesson, non-https link) are refused without using the message number.
- Manager screen (Manager > Learn & Earn > open a saved lesson): "Send to workers" card. SEND ON TELEGRAM TO ALL (two taps; needs a bot and at least one linked chat; shows how many went out and why others failed) and, for every connected worker, COPY CODE (one code for that worker, to paste in a private chat; they open it with Share or the "Use it" offer). Text tells the manager: the SAVED version is sent, save first; send again after every change; a hidden/draft lesson is taken back from phones that have it.
- Limits: sending is manual (nothing goes out when you save or publish). Copying a code uses up a message number even if you never pass it on (the worker may then see a "gap" note, harmless). Lessons travel through Telegram when sent there (signed, so they cannot be forged, but Telegram can read them). On a single phone used as manager AND worker the lesson is already in the same table, so the message says "already up to date".
- Not compile-tested (no Kotlin/Gradle in the sandbox). Brace balance and new names were checked against the whole project. If Actions fails, paste the error log.

Next (10C is now DONE, see below): 10C (worker Learn & Earn screen: reads `learn_content` where `published = 1`, list by category, read, open links with `LocalUriHandler`, a local "read" tick in `meta` keyed by lesson id and version, a card on the worker home).

## Piece 10C — Worker Learn & Earn screen (what exists now)
Zip: `piece-10c-learn-worker.zip` (sorts after `piece-10b-learn-send.zip`; no database change, DB stays v13; app 42 / 0.41.0). Files: `ui/WorkerLearn.kt` (new: `WorkerLearnCard(repos, workerId)`), `ui/WorkerScreens.kt` (one hook: the card sits on the REAL worker home between the team board card and the support card; not in demo mode), `app/build.gradle.kts`.
- Card: "Learn & Earn" with the count of lessons and how many are new or changed. Tap opens a full-screen dialog (back button: lesson -> list -> closed). List: category chips (only when there is more than one category; blank category = "General"), newest first, NEW / Read chip, 110-character preview, icons for video / picture. Lesson: title, category, time (EAT), the full text, OPEN VIDEO / OPEN PICTURE (https links only, shows the host first, a clear message if the phone cannot open it), MARK AS READ.
- Reads `ContentDao.observeLearn()` (published = 1 only, so a taken-back lesson disappears). Read tick = `meta` key `learn:read:<workerId>:<lessonId>` with value = the lesson version; an edited lesson (higher version) shows as NEW again. Nothing is sent to the manager.
- Limits: no search; the tick is local to the phone; images are links only (no in-app picture). Not compile-tested. If Actions fails, paste the error log.

Next: Piece 11 (backup / restore / migration). It must also solve the main manager's lost password + PIN (see 4B limits).

## Piece 11a — Main manager login built into the build (what exists now)
Zip: `piece-11a-main-login.zip` (sorts after `piece-10c-learn-worker.zip`; no database change; app 43 / 0.42.0). Files: `data/ManagerAccounts.kt` (`ensureMain`, new constants `SEED_PASS_SALT`, `SEED_PASS_HASH`, `SEED_PIN_FINGERPRINT`), `app/build.gradle.kts`.
- A new install creates the main manager already ACTIVATED: code name `MAIN`, the owner's chosen password and the seed PIN, all stored only as salted PBKDF2 hashes (the password and the PIN are NOT in the source as text). The seed PIN also goes into `pin_ledger`, so no manager can ever be issued it. The first sign-in on a phone is the full one (code name + password + PIN) and then the 30-day quick unlock applies.
- An older install where the main manager never finished activating (never signed in with a password, `credVersion` 1, seed PIN still in place) gets the built-in password at the next app start (audit `MAIN_MANAGER_BUILTIN_PASSWORD`). An install where the main manager already chose their own password keeps it.
- Both can still be changed in Manager > My security. Honest limit: anyone who gets the APK can try to guess the password and PIN offline from the hashes (the 6-digit PIN is the weak part). A longer PIN and a longer password make that much harder. A manager phone that trusts another phone's main manager (5E) has no main account of its own and is not affected.
- Lost password AND PIN after changing them: there is still no in-app way back; Piece 11b/11c (backup and restore) is the answer, plus reinstalling gives the built-in login again (but empty data).

## Piece 11b — Backup export (what exists now)
Zip: `piece-11b-backup-export.zip` (sorts after `piece-11a-main-login.zip`; no database change; app 44 / 0.43.0). Files: `data/Backup.kt` (new), `ui/BackupScreens.kt` (new), `ui/ManagerScreens.kt` (route `MgrRoute.Backup`, dashboard card "Backup" for the main manager only), `app/build.gradle.kts`.
- `BackupFormat` (pure, tested in a Python port: wrong passphrase and a flipped byte are both refused): file `CTBK` + version 1 + salt 16 + nonce 12 + AES-256-GCM of `[4-byte header length][header JSON][database bytes]`, key = PBKDF2-HMAC-SHA256 200 000 iterations; magic is the GCM AAD. Header: `fmt`, `app`, `db` (Room version), `created`, `bytes`, `sha256` of the database, `counts` (rows per main table). `BackupFormat.seal(dbBytes, headerJson, pass)`, `BackupFormat.open(file, pass): Outcome<BackupOpened(header, db)>` (decrypt + sha + size check), `BackupFormat.infoOf(header)`, `passError` (10-64 chars, letters + a digit). Limit 64 MB database.
- `BackupRepository(db, audit)`: `export(context, passphrase, out, actor)` (WAL checkpoint, copy `ctteam.db`, open the COPY and run SQLite `integrity_check`, seal, write, meta `backup:lastAt` / `backup:lastBytes`, audit `BACKUP_EXPORTED` without the passphrase), `check(context, fileBytes, passphrase)` (full verification, changes nothing), `lastBackupAt()`, `readLimited(stream, limit)`. 11c reuses `BackupFormat.open` and then replaces the database file.
- Screen (Manager > Backup, main manager only): last backup time, passphrase + repeat, CREATE BACKUP (system file picker: phone storage, SD card, Drive), CHECK A BACKUP FILE. Honest texts: no recovery if the passphrase is lost; original Telegram pictures (folder `originals`) are not inside.
- Limits: a write by the app in the same instant as the copy can be missed (the next backup has it); the whole file is held in memory (hence 64 MB); notification/WorkManager settings and the `originals` folder are not saved. Not compile-tested. If Actions fails, paste the error log.

## Piece 11c — Restore (what exists now)
Zip: `piece-11c-restore.zip` (sorts after `piece-11b-backup-export.zip`; no database change; app 45 / 0.44.0). Files: `data/Restore.kt` (new), `ui/RestoreScreens.kt` (new), `ui/BackupScreens.kt` (one line: `RestoreSection` before the message line), `app/build.gradle.kts`.
- Where: Manager > Backup (main manager only, same gate as 11b). Card "Restore from a backup": passphrase, CHOOSE BACKUP FILE (file picker, max 64 MB), then a PREVIEW (nothing changed yet): made-at (EAT), app build, database version, a table "now -> in the backup" for workers, tasks, submissions, payout requests, payments, NC rows, NC claims, manager accounts, audit log (red when the phone has more than the backup), and warnings (age in days, rows that will be lost, older database version is upgraded, sign in with the managers INSIDE the backup, workers keep working). Then type `RESTORE` (exact, capitals) and RESTORE NOW.
- `BackupRestorer(db)` (not in `Repos`; the screen makes it): `preview(context, bytes, pass)`, `restore(context, bytes, pass, actor): RestoreOutcome(ok, message, restart)`, `undo(context, actor)`, `safetyTime(context)`. Order: `BackupFormat.open` (passphrase + GCM + SHA-256) -> refuse a backup whose Room DB version is NEWER than this app's -> write the DB to `ctteam.db.restore-tmp` next to the real file -> SQLite `integrity_check` -> in that temp file only: move sync counters forward and append a hash-chained audit row (`BACKUP_RESTORED`, or `RESTORE_UNDONE`) and set meta `restore:lastAt` / `restore:fromCreated` -> WAL checkpoint -> SAFETY COPY of the live database (`filesDir/restore-safety/ctteam-before-restore.db`, one slot; the live DB is checkpointed first, size is verified) -> `db.close()`, delete `-wal`/`-shm`/`-journal`, rename the temp file over `ctteam.db` (copy fallback; if placing fails the safety copy is put back) -> `CTRestart.now` (`Intent.makeRestartActivityTask` + process exit after 300 ms).
- SYNC NUMBERS (important, 5A–5E rule): replay protection is a set of seen message numbers, so an OLD backup would make the manager phone reuse numbers the workers already accepted and every new update would be refused as a copy. Before the swap the app reads the LIVE counters and, in the new file, sets every `sync:out:M2W:<worker>` to max(restored, live) + 20; other `sync:out:*` keys and `tg:offset` get max(restored, live). Workers see, once, a harmless note that some updates "have not arrived". Not recoverable by design: reports the manager imported AFTER the backup are forgotten, so a worker can send the same report again; items carry their own ids, so nothing is counted twice.
- UNDO: card "Undo the last restore" appears while a safety copy exists; type `UNDO`; the same install routine puts it back and keeps what it replaces as the new safety copy (so undo can itself be undone).
- After the restart the manager signs in with the manager names, passwords and PINs that exist inside the backup (the built-in MAIN login applies only to a new, empty install). The Telegram token is part of the database and comes back with it.
- Limits / honest notes: the whole file and database are held in memory (64 MB limit). The `originals` picture folder is not part of a backup and is not touched by a restore. In the 300 ms between closing the database and the process ending, a screen that was still reading the old database could in theory show a crash; the data is already correct and the next start is clean. Auto-restart can be blocked by some phones; the text tells the user to open the app again. Not compile-tested (no Kotlin/Gradle in the sandbox); brace balance and public-name uniqueness were checked. If Actions fails, paste the error log.
- Next: 11d (send the backup to Telegram/Drive/SD, old-backup reminder), then 11e (emergency migration to a new phone), then 12.

## Piece 11 plan (c–e, decided with the owner; 11c is done, build 11d next)
11b: data -> one file. Simplest safe design: checkpoint the Room database (`PRAGMA wal_checkpoint(TRUNCATE)`), copy `ctteam.db`, zip it with a small header JSON (format version, app versionCode, DB version, created time, table row counts), encrypt with AES-256-GCM using a key from the owner's passphrase (PBKDF2-HMAC-SHA256, 200 000 iterations, random salt), SHA-256 of the plain bytes in the header (checked on restore). Saved through the Android file picker (create document) and offered to the share sheet. File name `ctteam-backup-YYYYMMDD-HHmm.ctb`. Screen: Manager > Backup, only the main manager. Photos of proofs (files) are NOT in the database: say so, or include the proofs folder as a second part if small enough.
11c: restore only on the main manager's phone: pick file, passphrase, verify GCM tag + SHA-256, show the preview, typed confirmation "RESTORE", close the database, replace the files, restart the app. A backup from a NEWER DB version is refused; an older one is migrated by Room as usual.
11d: send the same file to Telegram as a document (9A bot), plus "last backup" age on the dashboard with a reminder after 7 days (reuse the 9D-3 notification worker).
11e: emergency migration: what must move to a new phone (manager accounts and credVersion, pin_ledger, per-worker keys, sync counters `sync:out:*`, last-seen numbers), and what a worker does when the manager's phone changes. Check against the 5A-5E rules before building.

## Piece 11e-1 — message numbers stay in step (what exists now)
Zip: `piece-11e-1-number-sync.zip` (sorts after `piece-11d-backup-send.zip`). Files: `data/Sync.kt`, `app/build.gradle.kts` (47 / 0.46.0). No database change, no screen change.
Why: replay protection means a worker phone accepts each manager message number once. If the manager phone is restored from an OLDER backup (11c) or replaced, its counter `sync:out:M2W:<worker>` can be lower than what the worker already received, and new updates would be refused as copies. 11c adds +20 as a first guard; this piece makes it exact.
- New worker -> manager type `SYNCINFO` (`TYPE_SYNCINFO`, body `ls` = highest manager message number this worker phone has received). It rides in every report as an item `SYI:<worker>:<ls>` (new item each time the number changes; a report that would hold ONLY this note counts as "nothing new", and it is not counted in "Not reported yet" or the 9D-3 reminder).
- The ENROLL (connection) code also carries `ls` now (field `ls`, signed with the rest).
- Manager phone: `SyncRepository.raiseCounter(workerId, lastSeen, actor)` raises the counter to `ls` when it is lower, never lowers it, and never by more than `COUNTER_JUMP_MAX` = 1,000,000 in one step; audit `SYNC_COUNTER_RAISED` ("counter a -> b"). Called by `applySyncInfo` (import result line "Message numbers aligned for <name>") and by `acceptEnrollment` (a worker who enrolls on a fresh manager phone after getting updates from the old one).
- Effect: after any restore or phone change the first report each worker sends fixes the manager's numbering by itself; no manual step. Until that report arrives, an update sent early may be refused by the worker as "Already received" and must be sent again (SEND EVERYTHING AGAIN).
- Limits: it does not bring back data the manager phone lost (tasks, payouts, manager accounts: that is what the backup is for). A worker who never reports keeps the old gap.

## Piece 11e-2 — Move to a new phone + RETIRE (what exists now)
Zip: `piece-11e-2-move-phone.zip` (sorts after `piece-11e-1-number-sync.zip`). Files: `data/Migrate.kt` (new: `PhoneMove`, `MoveStatus`), `ui/MigrateScreens.kt` (new: `PhoneMoveSection`, `RetiredPhoneScreen`), `ui/BackupScreens.kt` (one line: the section under "Restore from a backup"), `ui/ManagerScreens.kt` (retired gate in `ManagerArea`), `data/Notify.kt` (the manager check does nothing on a retired phone), `app/build.gradle.kts` (48 / 0.47.0). No database change; the flag is `meta` key `phone:retired` (millis; empty = not retired).
- Manager > Backup (main manager only) > card "Move to a new phone" > OPEN THE STEPS: numbered steps for the OLD phone (fresh backup, check the file), the NEW phone (install, Restore, switch notifications off/on because the WorkManager schedule is not in a backup, check the Telegram bot is reading), then RETIRE; what workers do (nothing: their next report re-aligns numbers, see 11e-1); the NO-BACKUP path (sign in as MAIN, workers re-enroll with the same key and send "Everything"; tasks, payments, manager accounts and team data are lost); the LOST/STOLEN path (revoke the bot token in @BotFather, change passwords and PINs, honest limit below). Live checks: last backup time (warns when overdue), connected workers, bot connected/paused, notifications on/off.
- RETIRE THIS PHONE (type `RETIRE`): `PhoneMove.retire` switches the manager notification check off, pauses Telegram (`TG_PAUSED`), sets `phone:retired`, audits `PHONE_RETIRED`. A signed-in manager is signed out within 2 s. A retired phone shows "Phone retired" instead of the manager sign-in (`RetiredPhoneScreen`), with USE THIS PHONE AGAIN (type `USE AGAIN`; audits `PHONE_UNRETIRED`; Telegram and notifications stay OFF until switched on again). Nothing is deleted. The worker side of the same phone is not affected.
- Honest limits: a backup made AFTER retiring carries the flag and would retire the phone it is restored on (the retired screen has the unlock). Worker secret keys are stored in the database and cannot be changed yet: a lost phone whose data is opened lets someone forge messages for workers (a later piece could rotate worker keys). A lost phone's 30-day quick unlock cannot be revoked remotely. Not compile-tested. If Actions fails, paste the error log.

## Piece 11e-3 — worker card "The manager has a new phone?" (what exists now)
Zip: `piece-11e-3-worker-new-manager.zip` (sorts after `piece-11e-2-move-phone.zip`). Files: `ui/SyncScreens.kt` (new private `NewManagerPhoneCard` at the end of `WorkerSyncCard`), `app/build.gradle.kts` (49 / 0.48.0). No database change.
- Worker home > Sync cards: collapsed card (OPEN). PREPARE BOTH FOR THE NEW PHONE makes (1) the connection code (`makeEnrollment`: same secret key, now also carries the last manager number received, 11e-1) and (2) a report with EVERYTHING (`makeReport(SCOPE_ALL)`: same message numbers as before, so the new manager phone ignores nothing it needs and counts nothing twice). Both show with the usual copy / share / save-as-file buttons. Order printed on the card: connection code first (the manager approves it and reads the key code by phone), then every report part. Also tells the worker to press START in a NEW Telegram bot if the manager has one.
- Worker secret key is NOT changed by this; if the manager's phone was lost or stolen, key rotation is still missing (see 11e-2 limits).
- Piece 11 is complete (11a-11e). Next: 12 (premium polish and animations).

## Piece 12a — touch feel (what exists now)
Zip: `piece-12a-touch-feel.zip` (sorts after `piece-11e-3-worker-new-manager.zip`). Files: `ui/Motion.kt` (new), `ui/Components.kt` (GoldButton, GlassCard), `ui/Forms.kt` (GhostButton, CTChip), `app/build.gradle.kts` (50 / 0.49.0). No new dependency, no database change.
- `Motion.kt`: `ctReduceMotion()` (true when the phone's animator duration scale is 0), `CTPress` + `rememberCTPress(pressedScale, onClick)` (shared `MutableInteractionSource`, a spring-animated scale and a light haptic tick before the real `onClick`), `Modifier.ctScale(press)`, and `Modifier.ctEnter(delayMs)` (fade in + rise 12 dp, 360 ms, once).
- Used in: GoldButton (scale 0.96), GhostButton (0.96), CTChip (0.94), GlassCard (tappable ones scale 0.985; every card fades/rises in once). The ripple is kept (same order of modifiers as before: scale, clip, fill, clickable). Every tap still calls exactly the same function as before.
- Honest notes: cards inside a list that appear later (new item) animate in too; with "Remove animations" on the phone everything is instant and there is no scale. Not compile-tested. If Actions fails, paste the error log.
- Build lesson: an edit script once truncated `Forms.kt` (opened it for writing before reading it); the file was restored from `piece-03-manager.zip` (its only owner) and re-patched. The zip contains the full, correct file.

## Piece 12b-1 — three-bar menu, manager home (what exists now)
Zip: `piece-12b-1-menu-manager.zip` (sorts after `piece-12a-touch-feel.zip`). Files: `ui/MenuDrawer.kt` (new, reusable: `HamburgerItem(title, subtitle, badge, onClick)`, `HamburgerGroup(title, items)`, `HamburgerButton(dot, onClick)`, `ScreenHeaderMenu(title, onBack, dot, onMenu)`, `HamburgerMenu(title, groups, onClose)` = full-screen Dialog, choosing an item closes it then navigates), `ui/ManagerScreens.kt` (dashboard only), `app/build.gradle.kts` (51 / 0.50.0). No database change.
- Manager home now shows: header with the three bars at the right, stats, then the daily cards Review submissions, Workers, Tasks, Payouts, Support, and Recent activity. The menu lists everything in four groups (Daily work, Team and learning, Connections, Safety and access; Managers and Backup only for the main manager) with the same badges. A teal dot on the bars appears when something that is NOT on the home page waits (NC unmatched rows, Telegram items, backup overdue).
- For 12b-2 reuse `MenuDrawer.kt` as it is; do not re-declare these names.

## Piece 12b-2 — three-bar menu, worker home (what exists now)
Zip: `piece-12b-2-menu-worker.zip` (sorts after `piece-12b-1-menu-manager.zip`). Files: `ui/WorkerScreens.kt`, `app/build.gradle.kts` (52 / 0.51.0). No database change. Uses `ui/MenuDrawer.kt` from 12b-1 unchanged.
- `WorkerShell` got two optional parameters `menuGroups` and `menuDot`; when `menuGroups` is not empty the home header is `ScreenHeaderMenu` (three bars top right) and `HamburgerMenu` opens. The demo worker passes none, so it looks as before.
- Real worker home keeps: incoming-update card, announcements, profile card, this week tiles, payout-in-progress card, MY TASKS, SUBMISSIONS & EARNINGS, REQUEST PAYOUT, BNB DEPOSIT CLAIMS, and a one-line hint about the menu.
- Menu (groups Team and learning / Manager / My account): Team mining (team card + team board), Learn & Earn, Support (badge = unread replies), Sync (badge = items not yet sent, `SyncRepository.unsentCount`), Change payout number, Reminder, Security. Each opens as a page inside `WorkerHomeScreen` (state `section`, back arrow and the phone back button return to the home); the dot on the bars shows when support has unread replies or items are unsent.
- Because a menu page replaces the home, the home's own state (open task etc.) restarts at the home page when a page is closed.

## Piece 12b-3a + 12b-3b — motion between screens, count-up, shimmer (what exists now)
Zips: `piece-12b-3a-slide-between-screens.zip` (app 53 / 0.52.0) then `piece-12b-3b-count-up-shimmer.zip` (app 54 / 0.53.0; includes the 3a changes of the same files, so upload both). Files: `ui/Motion.kt`, `ui/ManagerScreens.kt`, `ui/WorkerScreens.kt`, `app/build.gradle.kts`. No database change, no new dependency (`androidx.compose.animation` was already there).
- 3a: `rememberCTScreenSpec<S>(depthOf)` in `Motion.kt` returns an `AnimatedContent` transition: deeper = new screen slides in from the right and fades while the old one drifts left; back = the reverse; same depth = cross-fade; no movement when the phone's animations are off (`ctReduceMotion`). Used on the manager route switch in `ManagerArea` (`mgrDepth`: Dashboard 0, WorkerDetail/TaskEdit 2, others 1) and on `WorkerShell` (`wDepth`: Home 0, lists 1, TaskDetail 2, Submit and claims-from-a-task 3). The `when` that picks the screen now sits inside `AnimatedContent(... ) { r -> when (r) {...}; Unit }`; every branch uses `r` (the lambda parameter), never `route`.
- 3b: `ctCountUp(target: Long): Long` counts from 0 on first show and glides from the old to the new value on change (whole-number maths, ends exactly on the target). `StatTile` (manager) and `SummaryTile` (worker home) now take `(label, value: Long, show: (Long) -> String, modifier)`. `ShimmerBlock(height)` = pulsing placeholder; the manager dashboard shows three of them instead of "Loading…".
- Honest notes: during a slide both screens are alive for 300 ms (their effects run briefly); a screen's scroll position restarts when it is re-entered. Not compile-tested. If Actions fails, paste the error log.

## Piece 12c — empty states + success checkmark (what exists now)
Zip: `piece-12c-empty-states-success.zip` (sorts after `piece-12b-3b-count-up-shimmer.zip`). Files: `ui/Polish.kt` (new: `EmptyState(icon, title, hint)`, `SuccessNote(text)` = circle draws itself then the check mark, beside the green message; both static when phone animations are off), `ui/ManagerScreens.kt` (empty states for workers, tasks, review; after APPROVE / REJECT the review list shows a SuccessNote for 3.5 s; the decision now runs in the list screen's scope so it finishes even when the detail screen disappears), `ui/SupportManager.kt`, `ui/LearnManager.kt`, `ui/PayoutScreens.kt` (empty state "No payout is waiting"), `app/build.gradle.kts` (55 / 0.54.0). No database change.
- Errors keep their red text; only successes use `SuccessNote`.
- Not built: the glass bottom bar (listed in the SPEC polish idea; the three-bar menu covers navigation).

## Referral program plan (owner idea 2026-10-11) and Piece R1 (what exists now)
Owner idea: workers who refer new workers get a higher STATUS and POINTS; points can become money or gifts when spent; status means easier access to opportunities.
Design decisions (all rules are manager-editable; confirm with the owner before building R2+):
- One code per worker (`CT-XXXXX`, no 0/O/1/I/L), made by the manager phone. A new worker types a code once; one referrer per worker; no own code, no circles (checked up the chain).
- A referral counts only after the referred worker has real APPROVED work (default 3 approved submissions). The manager sees READY referrals and taps AWARD: a human check against fake accounts.
- Points: default 100 per awarded referral, append-only ledger `points_ledger` (unique per referred worker, so no double award); manual BONUS / ADJUST need a reason and are audited.
- Status ladder from AWARDED referrals: Member 0, Connector 1, Builder 3, Leader 7, Champion 15. Spending points never lowers status.
- ONE LEVEL ONLY: points come only from workers you referred yourself, never from their referrals (keeps it away from pyramid / MLM schemes; the owner should still check Uganda rules before paying money for referrals).
- Points to money: UGX per point typed by the manager (default 50), paid only when the manager approves a request (R4).
R1 zip: `piece-13r1-referral-data.zip` (sorts after `piece-12c-empty-states-success.zip`). Files: `data/Referral.kt` (new: entities `ReferralCodeEntity`, `ReferralEntity`, `PointsEntryEntity`; `ReferralDao`; `ReferralRules` (tiers, code normalising, readiness); `ReferralRepository`: `codeFor`, `link`, `lines`, `award`, `reject`, `adjust`, `balance`, `awardedCount`, `statusOf`, settings `pointsPerAward` / `needApproved` / `ugxPerPoint` / `saveSettings`), `data/CTDatabase.kt` (version 14, `MIGRATION_13_14`, `referralDao()`), `data/ServiceLocator.kt` (`repos.referral`), `ui/ReferralScreens.kt` (new: `ManagerReferralScreen`, tabs Referrals / Points / Codes / Rules), `ui/ManagerScreens.kt` (route `MgrRoute.Referral`, menu item "Referrals" in the three-bar menu group Team and learning), `app/build.gradle.kts` (56 / 0.55.0).
- Manager screen: Referrals tab (record a referral by picking the new worker and typing the code; READY / WAITING / AWARDED / REJECTED list; AWARD POINTS; REJECT with a reason), Points tab (status and balance per worker; add/remove points with a reason; latest entries), Codes tab (every worker's code with COPY), Rules tab (points per referral, approved submissions needed, UGX per point; status ladder).
- Audit actions: REFERRAL_SETTINGS, REFERRAL_LINKED, REFERRAL_AWARDED, REFERRAL_REJECTED, POINTS_BONUS, POINTS_ADJUST.
- Workers see nothing yet (R2 to R5). Backups include the new tables automatically (the whole database file).
- Do not re-declare these public names later: `RefStatus`, `PointsKind`, `ReferralRules`, `ReferralLine`, `PointsBalance`, `ReferralCodeEntity`, `ReferralEntity`, `PointsEntryEntity`, `ReferralDao`, `ReferralRepository`, `ManagerReferralScreen`.

## Piece R1b — editable rules + falling reward (what exists now)
Zip: `piece-13r1b-referral-rules.zip` (sorts after `piece-13r1-referral-data.zip`). Files: `data/Referral.kt`, `ui/ReferralScreens.kt`, `ui/ManagerScreens.kt` (passes `isMain = me.isMain`), `app/build.gradle.kts` (57 / 0.56.0). No database change (settings live in `meta`: `ref:pts`, `ref:need`, `ref:ugxpp`, `ref:rate`, `ref:floor`, `ref:scope`, `ref:tiers`).
- ONLY the main manager can change the rules: the Rules tab is read-only for other managers and `ReferralRepository.saveSettings(settings, byMain, actor)` refuses when `byMain` is false. Every change is audited with all values (REFERRAL_SETTINGS).
- Tunable: points for the FIRST referral (default 100), reduction per earlier referral (default 5.0 %, 0 = none, max 50 %), minimum points per referral (default 10), who is counted (`TEAM` = all referrals of the team, so acting early pays; `WORKER` = the referrer's own referrals), approved submissions the new worker needs (default 3), UGX per point (default 50), and the four status thresholds (default 1 / 3 / 7 / 15; must rise and start at 1 or more).
- Falling reward: points = floor(base x (1 - rate)^earlier + 0.5), kept between the minimum and the base (`ReferralRules.pointsFor`, checked in a Python port: 100, 95, 81, 63, 38, 10 for the 1st, 2nd, 5th, 10th, 20th, 50th at 5 %). `earlier` = referrals that were recorded BEFORE this one (rejected ones do not count; same instant: smaller worker id first; scope TEAM or WORKER). It depends on the place in line, not on when the manager taps AWARD, so a slow manager never changes what a worker earns. The ledger note says "worth X of Y points because of its place in line". Points already awarded never change when the rules change.
- Screen: READY / WAITING lines show "worth N points"; the Rules tab has a live preview ("Referral number 1, 2, 5, 10, 20, 50, 100") and the status ladder with the typed thresholds.
- API for later pieces: `repos.referral.settings()` (class `RefSettings`), `pointsForReferral`, `thresholds()`, `ReferralRules.tierOf(awarded, mins)`, `ReferralRules.thresholdFor(index, mins)`, `ReferralRules.nextTier(awarded, mins)`.


## Piece R2 — referral code at connect

Zip: `piece-13r2-referral-code.zip` (sorts after `piece-13r1b-referral-rules.zip`). Files: `data/Sync.kt`, `ui/SyncScreens.kt`, `app/build.gradle.kts` (58 / 0.57.0). No database change. Needs R1.
- Worker phone (Connect card, after SHOW CONNECTION OPTIONS): optional field "Referral code". Saved in `meta` key `ref:in:<workerId>` so the connection code can be made again; `makeEnrollment` adds JSON field `rc` (normalised) when not blank.
- Manager phone: `EnrollPreview.refCode` (new last field, default ""). The approval card shows "Referral code used".
- `acceptEnrollment`: after the connection is saved, calls `ReferralRepository.link(workerId, rc, actor)`. A wrong, own, circular or second code never blocks the worker; it is written to the audit log as `REFERRAL_CODE_REFUSED`.
- The manager can still add or fix a referral by hand in Referrals > Codes (R1).
- Not changed: the "join code" made by the manager (no referral field), Telegram (still says open Sync to approve).

## Piece R3a — referral figures to the worker, data only (what exists now)
Zip: `piece-13r3a-referral-view-data.zip` (sorts after `piece-13r2-referral-code.zip`; no database change, DB stays v14; app 59 / 0.58.0). Files: `data/ReferralView.kt` (new), `data/Sync.kt` (replaced, patched with assert-once edits), `app/build.gradle.kts`. No screens in this zip (R3b).
- New manager -> worker update type `SyncRepository.TYPE_REFERRAL_VIEW = "REFERRAL_VIEW"` (in `WORKER_TYPES`, dispatched to `applyReferralView`, built in `collectUpdateItems` right after the TEAM_VIEW item, item key `RV:<workerId>:<rev>`). It rides the normal update flow, so "send updates (new only / everything)", Telegram send and copy-code all carry it with no extra work. Because every worker gets a code, every worker will show ONE new pending update after this build; that is expected.
- Manager phone: `ReferralViewCodec.build(db, repo, workerId)` makes the worker's code if missing (`codeFor`), then the body: `rev`, `at`, `cd` code, `bal` points balance, `aw` awarded referrals, `th` the four thresholds, `nd` approved submissions needed, `up` UGX per point, `rf` the worker's OWN referrals (newest 30: `n` first name + last initial, `s` P waiting / A awarded / R rejected, `ap` approved submissions so far, `pt` points awarded or, while waiting, what it is worth now, `at`). Same data gives the same message; any change in points, status, referrals or rules gives a new `rev` (meta keys `ref:vfp:`, `ref:vrev:`, `ref:vat:` + workerId). Never sent: the worker's own referrer, other workers' referrals, refusal notes.
- Worker phone: `ReferralViewCodec.apply` validates (code `CT-` + 5 letters/digits, balance within +-1e9, awarded 0..100000, four rising thresholds, need 1..100, UGX per point 0..1,000,000; a broken referral line is skipped, the rest is used), keeps the newest only (`ref:vjson:<w>`, `ref:vrevin:<w>`), audit `REFERRAL_VIEW_RECEIVED`. `ReferralViewCodec.load(db, workerId)` returns `ReferralView?` with helpers `tier()`, `nextTier()`, `toNext()`, `worthUgx()`, `waiting()` and the list `entries` (`RefViewEntry`).
- Do not re-declare: `RefViewEntry`, `ReferralView`, `ReferralViewApply`, `ReferralViewCodec`.
- Workers on an OLD app refuse this message type (it is unknown to them): update the workers' app first, then send updates.
- Not compile-tested (no Kotlin/Gradle in the sandbox). Brace balance and new names were checked against the whole project. If Actions fails, paste the error log.

Next: R3b (the worker page). Read `ui/WorkerScreens.kt` (three-bar menu of the worker home, `WRoute`) and `ui/WorkerTasks.kt` (`WRoute`); use `ReferralViewCodec.load`.

## Next: T4 (planned, decided with the owner)
**T3b — DONE** (see "Piece T3b" below).
**T3c — DONE** (see "Piece T3c" below).
T4 is done. Piece 9 (Telegram) is split 9A–9D (see below), then 10, 11, 12.

## Build-fix note (8B)
The 8B build failed with `Redeclaration: PickedFile`: `ui/WorkerSubmit.kt` (screenshot picker, 4C2/6) and `ui/NcScreens.kt` (statement file, 8B) both declared a public top-level class `PickedFile` in package `com.ctteam.app.ui`. Fix zip `piece-08b-z-fix-pickedfile.zip` (sorts after `piece-08b-nc-statements.zip`) replaces `WorkerSubmit.kt` (class renamed `PickedProof`) and bumps the version to 20 / 0.19.1. RULE for later pieces: every public top-level name in `ui/` and `data/` must be unique in the package; prefix names or make them `private`.

## Team mining plan (T1–T4) — decided with the owner, build after 8D
Idea: CT Team is a shared-mining team. Workers contribute crypto mined on their own accounts to the shared NC Wallet (verified through the 8C/8D BNB claims). Contributions buy more rigs, so team hash rate grows. Mined crypto is split: 70% to workers by contribution, 30% to the manager.
Decisions:
- Share formula: ALL-TIME contributions with a slight DECAY. weight = sum(value_i * 0.5^(ageDays_i / halfLifeDays)); worker share = weight / total weight of all workers. Default half-life 180 days (about 0.4% per day), manager can change it (stored in `meta`, every change in the audit log). The screen shows both "all-time contributed" and "counts towards share".
- Only VERIFIED contributions count (matched claim from 8D, or manager override with a note).
- Common unit for mixing coins: UGX value of a contribution, fixed when it is verified, using the rate then (rate stored with the row so it never changes later).
- UGX conversion: the app has no internet yet, so the manager sets a rate per coin (BNB, USDT, ...) with a date; every crypto amount is shown with "≈ UGX ..." and the text "estimate, rate set by manager on <date>". When internet arrives (Piece 9) an optional live-rate fetch can pre-fill the rate; the manager still confirms.
- Daily payout estimate: manager enters (or later imports) the team's mined amount per day; workers' pool = 70%; worker's expected daily payout = pool * share. Manager's 30% shown on the manager dashboard only. Everything labelled as an ESTIMATE.
- Weekly hash goal: manager sets target hash rate (TH/s etc.) for the week; progress bar for everyone; when reached, shares and payouts are recalculated and a celebration + top contributors list appear.
- Gifts: manager gives a multiplier (for a period) or a fixed extra payout to a worker, always with a reason, in the audit log, shown to the worker as a gift.
- Extra ideas agreed: levels (Bronze..Diamond), streaks, badges, rank changes, rig ledger (what contributions bought), inactive-worker alerts for the manager, weekly summary per worker (Piece 9 sends it by Telegram), "what if I contribute X" calculator, honest labels (estimates only, no guaranteed returns).
- Pieces: T1 data + formulas + rates (tables via ONE migration), T2 worker screens, T3 manager screens (mined amounts, hash goal, gifts, rig ledger), T4 leaderboard, levels, streaks, badges. Team figures reach worker phones as signed M2W updates (new type, same 5C style).
- Legal/honesty note for the owner: workers pool real money and see payout figures; keep all numbers as estimates and keep records transparent; check Uganda rules on pooled investments.

## Piece 11d — Send the backup out, and remind (what exists now)
Files (zip: `piece-11d-backup-send.zip`; sorts after `piece-11c-restore.zip`): `data/Backup.kt` (replaced), `data/Telegram.kt` (replaced), `data/Notify.kt` (replaced), `ui/BackupScreens.kt` (replaced), `ui/ManagerScreens.kt` (replaced), `app/src/main/AndroidManifest.xml` (replaced: a `FileProvider`), `app/src/main/res/xml/file_paths.xml` (NEW), `app/build.gradle.kts` (46 / 0.45.0). No database change.
- Where it is: Manager > Backup (main manager only). Under CREATE BACKUP there are two new buttons that use the SAME passphrase fields: SEND TO MY TELEGRAM and SHARE… (DRIVE, SD CARD, OTHER APPS). CREATE BACKUP with the file picker is unchanged.
- Flow: `BackupRepository.exportToCache` makes the normal encrypted `.ctb` into a temporary file in `cacheDir/backup/` (older temporary copies are deleted first; `export(..., markLast = false)` so nothing counts yet). TELEGRAM: `TelegramRepository.sendBackupFile` uploads it as a document, then the temporary file is deleted. SHARE: `FileProvider` (authority `<applicationId>.files`, only the `cache-path backup/`) + `ACTION_SEND` (`application/octet-stream`, `ClipData`, read-permission flag) opens the Android share window; the temporary encrypted copy stays in the private cache until the next backup (it is encrypted; it is NOT part of Android auto-backup because it is cache).
- "Last backup" (`backup:lastAt`) now moves only when the backup really left the phone: Telegram accepted it, or the share window was opened (`BackupRepository.markLastBackup`). If Telegram refuses, the screen says the backup was made but NOT delivered and the date is not changed. The app cannot know whether a share target (Drive, e-mail) finished, and says so; CHECK A BACKUP FILE is the way to verify the copy.
- The manager's OWN Telegram chat: the bot can only write to chats that wrote to it first, and worker chats are separate. So there is a pairing step (card "Telegram for backups"): LINK MY TELEGRAM shows a one-time code `BK-` + 8 digits (`startBackupLink`), valid 5 minutes, dies after 5 wrong tries. The manager sends it to the bot from their own Telegram; `TelegramRepository.handleUpdate` calls `tryBackupLink` BEFORE the CT1 handling (only for text without a document); on a match it saves `meta tg:backupChat`, replies "Linked" in that chat and audits `TG_BACKUP_CHAT_LINKED`. The Backup screen polls every 3 s while a code is pending (needs the Telegram reading loop, i.e. the manager area open). UNLINK MY TELEGRAM clears it (`TG_BACKUP_CHAT_UNLINKED`). A text starting `BK-` with no code pending is ignored like any chatter. Audit: `TG_BACKUP_LINK_STARTED`, `TG_BACKUP_LINK_LOCKED`, `TG_BACKUP_SENT` (size only), `BACKUP_REMIND_DAYS`.
- Size: Telegram lets bots send up to 50 MB; the app refuses above 49,000,000 bytes with the advice to use SHARE or CREATE BACKUP. `TelegramClient.sendDocument` now takes an optional `mime` (default `text/plain`, backup uses `application/octet-stream`), announces the exact length (`setFixedLengthStreamingMode`) so a big file is streamed instead of buffered twice, and its read timeout is 120 s. Existing callers are unchanged.
- Privacy: the file is encrypted (AES-256-GCM, PBKDF2, 11b) before it leaves the app, so Telegram, Drive or e-mail hold only ciphertext; the passphrase is never stored or sent.
- Reminder: setting `backup:remindDays` (default 7, allowed 1..90, card "Backup reminder", audited). `isOverdue` / `overdueText`: no backup ever, or older than that many days. Shown as (1) a warning card at the top of the Backup screen, (2) a "!" badge on the dashboard Backup card (main manager only), (3) a phone notification from the existing manager check (9D-3, `CTNotify.runManagerCheck`) via `BackupRepository.reminderDue`: at most one per day (`backup:remindedAt`), id `CTNotify.ID_BACKUP` (9104), and ONLY while the manager notification check is turned on in Manager > Notifications (there is no separate switch; the banner and badge always work).
- Limits: the phone notification needs the manager check on and Android notification permission; the Telegram link needs the manager area open to see the code; backups above 49 MB cannot go by Telegram; share-window delivery is not confirmed by Android. NOT in the backup file (unchanged from 11b): the saved original pictures. Not compile-tested (no Kotlin/Gradle in the sandbox); if Actions fails, paste the error log.
Next: 11e (emergency migration to a new phone), then 12.

## FILE OWNERS (latest zip that holds each file; to change a file, read its owner zip; upload this list's zips for the files you touch + their callers)
- `app/build.gradle.kts` — piece-13r3a-referral-view-data.zip
- `data/Backup.kt` — piece-11d-backup-send.zip
- `ui/BackupScreens.kt` — piece-11e-2-move-phone.zip
- `data/Migrate.kt` — piece-11e-2-move-phone.zip
- `ui/MigrateScreens.kt` — piece-11e-2-move-phone.zip
- `data/Restore.kt` — piece-11c-restore.zip
- `ui/RestoreScreens.kt` — piece-11c-restore.zip
- `ui/WorkerLearn.kt` — piece-10c-learn-worker.zip
- `app/src/main/res/xml/file_paths.xml` — piece-11d-backup-send.zip
- `app/src/main/AndroidManifest.xml` — piece-11d-backup-send.zip
- `MainActivity.kt` — piece-09b-2-worker-import.zip
- `data/CTDatabase.kt` — piece-13r1-referral-data.zip
- `data/Daos.kt` — piece-10a-learn-manager.zip
- `data/Learn.kt` — piece-10a-learn-manager.zip
- `data/Referral.kt` — piece-13r1b-referral-rules.zip
- `ui/ReferralScreens.kt` — piece-13r1b-referral-rules.zip
- `data/Entities.kt` — piece-08c-nc-claims.zip
- `data/ManagerAccounts.kt` — piece-11a-main-login.zip
- `data/ManagerSync.kt` — piece-05e-manager-access.zip
- `data/NcMatcher.kt` — piece-08d-nc-matching.zip
- `data/NcRepository.kt` — piece-08t1-team-ledger.zip
- `data/NcStatementParser.kt` — piece-08b-nc-statements.zip
- `data/PayoutMatching.kt` — piece-07-payouts.zip
- `data/PdfText.kt` — piece-08a-pdf-reader.zip
- `data/ProofAnalyzer.kt` — piece-06-screenshot-verification.zip
- `data/Repositories.kt` — piece-02-data.zip
- `data/ScreenshotChecks.kt` — piece-06-screenshot-verification.zip
- `data/Security.kt` — piece-04c2-worker-proof.zip
- `data/ServiceLocator.kt` — piece-13r1-referral-data.zip
- `data/Sync.kt` — piece-13r3a-referral-view-data.zip
- `data/ReferralView.kt` — piece-13r3a-referral-view-data.zip
- `data/Telegram.kt` — piece-11d-backup-send.zip
- `data/Notify.kt` — piece-11e-2-move-phone.zip
- `ui/TelegramScreens.kt` — piece-09c-1-original-pictures.zip
- `ui/ShareInbox.kt` — piece-09b-2-worker-import.zip
- `ui/WorkerIncoming.kt` — piece-09b-2-worker-import.zip
- `ui/WorkerNotices.kt` — piece-09b-3-announcements.zip
- `ui/OriginalCheck.kt` — piece-09c-2-review-originals.zip
- `ui/Motion.kt` — piece-12b-3b-count-up-shimmer.zip
- `ui/Polish.kt` — piece-12c-empty-states-success.zip
- `ui/SupportManager.kt` — piece-12c-empty-states-success.zip
- `ui/LearnManager.kt` — piece-12c-empty-states-success.zip
- `ui/Support.kt` — piece-09d-2a-support-worker.zip
- `data/Team.kt` — piece-08t3a-mined-days.zip
- `data/TeamBoard.kt` — piece-08t4-leaderboard.zip
- `ui/TeamBoardScreens.kt` — piece-09d-1-weekly-summaries.zip
- `ui/WorkerTeamBoard.kt` — piece-08t4-leaderboard.zip
- `data/TeamGifts.kt` — piece-08t3c-gifts.zip
- `data/TeamGoals.kt` — piece-08t3b-goal-rigs.zip
- `data/TeamWorker.kt` — piece-08t4-leaderboard.zip
- `data/Util.kt` — piece-02-data.zip
- `data/Verification.kt` — piece-06-screenshot-verification.zip
- `ui/CTApp.kt` — piece-09b-2-worker-import.zip
- `ui/Components.kt` — piece-12a-touch-feel.zip
- `ui/Forms.kt` — piece-12a-touch-feel.zip
- `ui/ManagerAccess.kt` — piece-05e-manager-access.zip
- `ui/ManagerScreens.kt` — piece-13r1b-referral-rules.zip
- `ui/MenuDrawer.kt` — piece-12b-1-menu-manager.zip
- `ui/NotifyScreens.kt` — piece-09d-3-notifications.zip
- `ui/ManagerSyncScreens.kt` — piece-05e-manager-access.zip
- `ui/NcMatchScreens.kt` — piece-08d-nc-matching.zip
- `ui/NcScreens.kt` — piece-08d-nc-matching.zip
- `ui/PayoutScreens.kt` — piece-12c-empty-states-success.zip
- `ui/Screens.kt` — piece-03-manager.zip
- `ui/SecurityScreens.kt` — piece-04c1-worker-tasks.zip
- `ui/SyncScreens.kt` — piece-13r2-referral-code.zip
- `ui/TeamGiftScreens.kt` — piece-08t3c-gifts.zip
- `ui/TeamGoalScreens.kt` — piece-08t3b-goal-rigs.zip
- `ui/TeamScreens.kt` — piece-08t4-leaderboard.zip
- `ui/Theme.kt` — piece-01-foundation.zip
- `ui/WorkerDemo.kt` — piece-08t4-leaderboard.zip
- `ui/WorkerHistory.kt` — piece-04c3-worker-history.zip
- `ui/WorkerNcClaim.kt` — piece-08c-nc-claims.zip
- `ui/WorkerPayout.kt` — piece-04c4-worker-payout.zip
- `ui/WorkerScreens.kt` — piece-12b-3b-count-up-shimmer.zip
- `ui/WorkerSubmit.kt` — piece-08b-z-fix-pickedfile.zip
- `ui/WorkerTasks.kt` — piece-08c-nc-claims.zip
- `ui/WorkerTeam.kt` — piece-08t3c-gifts.zip
- `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml` — piece-01-foundation.zip
- `app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml` — piece-01-foundation.zip
- `app/src/main/res/values/colors.xml` — piece-01-foundation.zip
- `app/src/main/res/values/themes.xml` — piece-01-foundation.zip
- `build.gradle.kts` — piece-02-data.zip
- `settings.gradle.kts` — piece-01-foundation.zip

## Idea parked (owner, to confirm later)
The CT mining pool may have a website with public stats. If it does, a later piece can read the daily mined amount from it (needs internet permission, which Piece 9A added). The manager would still confirm every figure. Pool bot messages on Telegram: bots cannot read other bots, so use share/paste or forwarding to our own bot (send 2–3 sample messages so a reader can be written).
