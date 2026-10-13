# tucck — HANDOFF (as of 2026-10-13 ~07:15 UTC)

## 1. Project overview

**What tucck is.** tucck is one inbox for the links, screenshots, PDFs, voice notes and ideas people otherwise lose across apps.
- The Android app (Expo) captures. tucck web (https://tucck.app, Next.js) works through what was captured.
- Plans:
  - Free: everything stays on the phone.
  - Starter (paid, Google Play via RevenueCat): sync to the server, plus tucck web.
- AI is BYOK (your own Anthropic, OpenAI or Google key), kept on the phone and shared sealed with paired web.
- Design source of truth: Figma file `7LIZTmcnfMy20hskMpG4Rn`. Screen IDs (B1, D3, WEB5, SP9-P1…) appear in code comments.

**What is planned next** (from `docs/OVERVIEW.md` → "Features planned"):
- **Settings rework (NEXT milestone):** Settings becomes the one place reachable from every tab. Account & sync, plan, devices and sign-in move inside it, beside Appearance, Notifications, Privacy, AI with your key, Help and About. The user asked for this on the phone on 2026-10-13.
- **M21 follow-up:** show event tuccks on the Tasks calendar (M17 calendar) by start date.
- **Audit phases still to do:**
  - Phase 4: visual consistency.
  - Phase 5: copy and accessibility.
  - Phase 6: launch QA on a real phone.
- **Go-to-market:** Play listing, Product Hunt (launch video in `marketing/launch-video`), first users.
- **Website follow-ups:** a real Google Play link; the Play pricing service account (DEPLOY.md §5).

**Current goal and immediate next task.**
- M21 is finished and merged (PR #34 merged 2026-10-13 07:07 UTC).
- The final APK (head 3c04dcd) built green: https://github.com/aniketh703/tucck/actions/runs/38358730710 → Artifacts → `tucck-apk`.
- The user has not yet been given this exact link in chat; give it first.
- Next steps:
  1. Give the user the APK link.
  2. Wait for their device-test feedback on M21.
  3. Start the **Settings rework** milestone. Plan first, get approval, then implement.

**Status.**
| Area | Status |
|---|---|
| M1–M20, Audit Phases 1–3, M21 | DONE (merged to `main`) |
| Production on Heroku (tucck.app) | LIVE. Deploy runs on each push to main through CI then `workflow_run`. NEEDS CONFIRMATION that the Deploy after the PR #34 merge went green; check Actions → Deploy |
| Device test of M21 APK | NOT STARTED (user to do) |
| Settings rework | NOT STARTED (next) |
| Events on Tasks calendar | NOT STARTED |
| Audit Phases 4–6 | NOT STARTED |

## 2. Technical context

**Where things live.**
- Repo: `aniketh703/tucck` (GitHub), cloned at `/home/user/tucck`.
  - The session's primary dir `/home/user/aniketh703` is a different, unrelated repo; only this file is there.
- Work branch: `claude/figma-screens-app-dev-bqtnwc`.
  - Every PR so far came from this branch (latest #34, now merged).
  - **Rule:** after a merge, restart the branch from `origin/main` (same name) and open a NEW PR. Never stack on merged history.
- Draft PRs. The PR template has Summary / Changes / Tests sections.

**Monorepo** (pnpm 10.33.0 workspaces + turbo; Node >= 22; TypeScript 5.9.3).

| Path | What it is |
|---|---|
| `apps/mobile` | Expo SDK 57.0.24, React Native 0.86.3, expo-router, React Compiler on |
| `apps/web` | Next.js 15.5.26, `@clerk/nextjs` 7.x |
| `packages/core` | Business rules; vitest against real Postgres |
| `packages/db` | Prisma 6.19.3. `prisma/schema.prisma`, `prisma/migrations/*` |
| `packages/shared` | zod DTOs/API (`src/api.ts`), domain (`src/domain.ts`), sync (`src/sync.ts`), analytics allow-list, M21 `groups.ts` and `cards.ts` |
| `packages/tokens` | Generated from `tokens.json` (`pnpm --filter @tucck/tokens build`) |
| `packages/icons` | Material Symbols list in `icons.json`; `node scripts/fetch.mjs` regenerates `src/paths.ts` and `svg/` |
| `apps/worker` | ASSUMPTION: background jobs. Can run in the web dyno via `TUCCK_WORKER_IN_WEB=1` |
| `docs/OVERVIEW.md` | Living feature/plan/architecture doc. **Must be updated in every feature PR** (CLAUDE.md rule) |
| `docs/SECURITY.md` | Data model, auth flow, isolation rules (M20) |
| `docs/GROWTH.md` | Growth notes |
| `DEPLOY.md` | Deploy guide |
| `CLAUDE.md` | Contributor rules |

**Read first:**
- `CLAUDE.md`
- `docs/OVERVIEW.md`
- `docs/SECURITY.md`
- `DEPLOY.md`
- `packages/shared/src/api.ts`
- `apps/mobile/src/lib/store.ts`
- `apps/mobile/src/lib/sync.ts`
- `apps/mobile/app.config.ts`
- `packages/core/src/saves.ts`
- `.github/workflows/*.yml`

**Mobile app key paths** (`apps/mobile/src`):
- `lib/store.ts`: local SQLite store and outbox.
  - Schema changes go in the `MIGRATIONS` array (append only).
  - Uses `newId` / `makeId`, `listGroup`, `setHiddenSpaces`, `visibleSql`.
- `lib/sync.ts`: push/pull through `/api/sync/*`.
- `lib/capture.ts`: share → drafts (captions, subject).
- `features/capture/receive.ts`: share receiver; assigns `groupId`.
- `features/capture/useQuickCapture.ts`: `(files, source, spaceId, caption)`.
- `features/capture/Composer.tsx`, `features/capture/AddMenu.tsx`: "+" menu, 8 tiles.
- `features/capture/pendingPhotos.ts`: in-memory handoff to the send screen.
- `app/(tabs)/index.tsx`: Inbox. Uses `foldAlbums` → `Entry`; `toggle(entry)`.
- `app/(tabs)/search.tsx`: Search; `TYPES` list.
- `features/inbox/FilterSheet.tsx`: `TYPE_CHIPS`.
- `features/inbox/SaveRow.tsx`:
  - `TYPE_ICON`, `TYPE_LABEL`, `useTypeTint`;
  - `SaveMeta`, the one-line meta row;
  - renders albums, contact cards and event cards.
- `features/inbox/AlbumGrid.tsx`.
- `features/contacts/*`: `Avatar`, `ContactCard`, `ContactPeople`, `actions.ts`.
- `features/events/*`: `EventCard` (takes `onEdit` prop; must NOT import expo-router), `EventHero`, `actions.ts`.
- `lib/contacts.ts`, `lib/events.ts`, `lib/groups.ts`: pure, tested.
- `app/capture/photos.tsx`, `app/capture/contacts.tsx`, `app/capture/event.tsx` (`?id=` edits).
  - All registered as modals in `app/_layout.tsx`.
- `app/saved/[id].tsx`: detail. `Hero` renders `ContactPeople` / `EventHero`.
- `app/saved/album/[group].tsx`: album page.
- `lib/usePermission.ts`, `lib/permissions.ts`, `ui/PermissionSheet.tsx`: four-beat permissions for notifications, microphone and contacts.
- `scripts/check-permissions.mjs`: Android manifest allow-list.
- `app.config.ts`: `RUNTIME_PERMISSIONS` and `BLOCKED_PERMISSIONS`.

**Web key paths** (`apps/web/src`):
- `library/InboxView.tsx`: folds albums; actions on albums use `a.bulk`.
- `library/DetailPane.tsx`, `library/FullDetail.tsx`.
- `library/SaveCards.tsx`: `EventInfo`, `ContactList`, `AlbumGrid`, .ics/.vcf download.
- `ui/ListItem.tsx`: `album` prop, mosaic.
- `ui/Thumb.tsx`.
- `library/SearchView.tsx`: `TYPES`.
- `app/privacy/page.tsx`.
- API routes in `app/api`. They stay thin through `route()` / `publicRoute()` in `server/http.ts`.

**Commands.**
- Postgres must be running for core and web tests: `service postgresql start`. It is down after a container restart.
- Full gate, run before every push:
  - `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test`
  - plus `pnpm --filter @tucck/mobile check:permissions`
- Apps import packages' built `dist`. After changing shared/core/tokens/icons, run `pnpm --filter @tucck/shared build` (or `pnpm build --filter "./packages/*"`).
- Mobile tests: `cd apps/mobile && pnpm exec jest [path]`. jest-expo; use `memoryStore()` from `src/lib/testing.ts`.
- Core tests: `pnpm --filter @tucck/core exec vitest run test/<file>`.
- `pnpm format` writes; `pnpm format:check` checks.
- `.prettierignore` covers `packages/icons/src/paths.ts` and the generated tokens. To format `paths.ts` after a fetch: `prettier --ignore-path /dev/null --write`.
- Adding Expo deps: `npx expo install <pkg>` in `apps/mobile`, then **run `pnpm install` at the root**. expo install broke `node_modules` once (missing `@babel/runtime`).
- Prisma migrations: hand-written SQL in `packages/db/prisma/migrations/<timestamp>_name/migration.sql`. Prisma doesn't model the generated tsvector `searchVector`; see `packages/db/README.md`.

**Environment variables** (names only; values live in Heroku config vars, GitHub secrets/vars and the user's password manager):
- **Heroku (app `tucck`):**
  - `APP_ORIGIN=https://tucck.app`
  - `DATABASE_URL` (Heroku Postgres essential-0), `DATABASE_SSL=no-verify`, `DATABASE_POOL_SIZE=4`
  - `TUCCK_WORKER_IN_WEB=1`
  - `CLERK_SECRET_KEY`, `CLERK_JWT_KEY`
  - `S3_ENDPOINT`, `S3_REGION=auto`, `S3_BUCKET=tucck`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY` (Cloudflare R2)
  - Optional: `SENTRY_*`, PostHog key, `REVENUECAT_SECRET_KEY` (not set yet), `PLAY_STARTER_PRODUCT_ID`, `ANDROID_CERT_SHA256`, `SKIP_MIGRATIONS`.
- **GitHub:**
  - secret `HEROKU_API_KEY`;
  - vars `HEROKU_APP=tucck` and `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` (pk_live).
- **Phone build-time:**
  - `EXPO_PUBLIC_API_ORIGIN`
  - Clerk publishable key (ASSUMPTION: `EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY`)
  - `EXPO_PUBLIC_POSTHOG_KEY` / `_HOST`
  - `EXPO_PUBLIC_REVENUECAT_ANDROID_KEY`
  - `EXPO_PUBLIC_SENTRY_DSN` / `_ENVIRONMENT`

**Services.**
- Heroku (container stack, Basic dyno; student credits).
- Cloudflare: DNS, Email Routing (`support@`, `hello@` → Gmail; possibly `aniketh@`), R2.
- Clerk production instance for tucck.app: Google SSO with custom credentials; native redirect `tucck://sso-callback`.
- PostHog (US). Sentry (MCP not authorized in-session). RevenueCat (not configured).
- Domain at Name.com, auto-renew OFF (expires 29 Sep 2027).

**CI/CD** (`.github/workflows`):
- `ci.yml`: check, docker, GitGuardian.
- `deploy.yml`: `workflow_run` after CI on main → Heroku. It pushes Docker v2 manifests (`oci-mediatypes=false`) because Heroku rejects OCI.
- `android-apk.yml`: `workflow_dispatch`, input `api_origin`. Builds a release APK as artifact `tucck-apk`.
  - Frees disk first and does a sparse checkout without `design/`.
  - Builds the workspace packages first. arm64 only. Takes about 20–50 min.
  - Dispatch: GitHub MCP `actions_run_trigger` (`run_workflow`, `workflow_id: android-apk.yml`, `ref` = branch, `inputs {api_origin: "https://tucck.app"}`).

**Architecture decisions (keep).**
- Server is canonical. The phone is offline-first and the only creator of saves; the web edits through REST and goes read-only while the phone is unreachable.
- Sync: per-user `changeSeq` cursor and `version`. Saves merge field by field with `SaveRevision` (3-way merge, conflict → CONF1). Other entities are last-writer-wins. Soft delete plus `Tombstone`.
- Auth: Clerk only proves identity. tucck issues its own sessions: web uses an HttpOnly cookie, the phone a bearer token in SecureStore.
- Isolation lives in the app layer (scoped by `auth.userId`) and is pinned by `packages/core/test/isolation.test.ts`, not Postgres RLS.
- **M21 data model:**
  - `Save.groupId` (uuid, client-chosen, points to no row, so there's no ownership check).
  - `Save.details` (JSONB, zod `SaveDetails` discriminated by `kind`: `contact` | `event`, must match `Save.type`).
  - `SaveType` gains `contact` and `event`.
  - Contact/event searchable text goes into `content`, so the existing tsvector covers it with no generated-column change.
  - Event description lives in `note`.
  - Event reminder = the normal save reminder (`remindAt = startsAt − reminderMin`), via `lib/reminders.ts` `remind()`.
  - `store.clearSaveReminder` covers "Never".
- **Albums** are a display fold (`foldAlbums` in shared): consecutive `image` saves with the same `groupId`, two or more. Search and spaces still list single photos.
- **Captions:** text shared with files becomes the first file's `note`. A URL in that text also becomes a link save in the same group.
- **No READ_MEDIA_IMAGES:** Play policy. Gallery uses the system Photo Picker, then the send screen.
- **READ_CONTACTS** is asked only on first use; WRITE_CONTACTS is blocked. "Add to contacts" uses `Contacts.presentFormAsync` from `expo-contacts/legacy` (the new expo-contacts API exists, but the legacy API is used deliberately).
- **"Add to calendar"** on the phone: `expo-intent-launcher` `startActivityAsync("android.intent.action.INSERT", {data: "content://com.android.calendar/events", extra})`. `Linking.sendIntent` can't set data. No calendar permission.

## 3. Current work state

- **Last task:** M21 (WhatsApp-style capture) plus Codex review fixes. Merged as PR #34. Final head `3c04dcd`; CI green; APK green (run 38358730710).
- **Commits in PR #34**, after Phase 3 `d265647` and device fixes `4ff7eb8`:
  - `48d8e79`: captions, albums, send screen, data model, migration `20261013000000_save_groups_details`.
  - `5533df8`: contacts and events.
  - `845653e`: web support, shared `groups.ts` / `cards.ts`, privacy, OVERVIEW.
  - `3c04dcd`:
    - `listGroup` returns visible images only;
    - contacts and events added to the type filters (Inbox `TYPE_CHIPS`, Search `TYPES`, web `SearchView` `TYPES`);
    - the event test made independent of the clock.
- **Uncommitted changes:** none in `/home/user/tucck` (working tree clean).
  - Do NOT commit `apps/mobile/.expo/types` if it appears.
- **Failing tests:** none at last run.
  - Counts: mobile 473, core 112 (+4 in `m21.test.ts`), shared 33, web 39.
  - Lint, typecheck, format and permissions all green.
- **Pending outward state:**
  - The send_later self-check `trig_013ZCN9SzhK7yXqrwkJcDTQ9` was deleted.
  - No PR is open; the session is unsubscribed.
  - The local branch is at `3c04dcd`. **Before new work:**
    ```
    git fetch origin main && git checkout -B claude/figma-screens-app-dev-bqtnwc origin/main
    ```
- **Next steps, in priority order:**
  1. Send the user the APK link (run 38358730710 → `tucck-apk`) and the device test list:
     - share 3 photos plus text from Google Photos → one album with a caption;
     - share a link from Chrome → no "From <page title>", and the meta line stays on screen;
     - "+" → Gallery → caption → Send;
     - "+" → Contact → pick 2 → Add;
     - "+" → Event with a 15-min reminder → notification; "Add to calendar" opens the calendar pre-filled.
  2. Confirm the Deploy workflow on main after the merge is green (the new migration must apply on Heroku).
  3. Act on any device feedback (bugs first).
  4. **Settings rework:** explore the current Settings/Account screens (`apps/mobile/src/app/account/*`, `apps/mobile/src/app/settings/*` (ASSUMPTION: path), tab layout), write the plan, get approval, then implement on a fresh branch and new draft PR. Update OVERVIEW.
  5. Events on the Tasks calendar (`features/tasks/TaskCalendar.tsx`): show event saves by `details.startsAt`.

## 4. Decisions and constraints

**Product and UX.**
- Inbox looks like messaging yourself:
  - notes are accent bubbles on the right;
  - files and links are cards;
  - the meta line reads "From X · time · Space • Unsorted".
- One card at most above the Inbox list.
- Every action says what it did (snackbar, Undo where possible).
- Calm permission flow: our rationale first, then the OS dialog, then the denied state, then Android settings. Never ask on cold start.
- The "+" menu has 8 tiles in 4 columns: Note, Link, Voice, Task, Gallery, Paste, Contact, Event. The footer line "PDFs and files: share them from any app" stays.
- Long-press on an album enters selection with the whole album picked. Tapping an album opens the album page.
- Event card: calendar-page date block, "Edit event", "Add to calendar". Contact card: Call / Add, or "View contacts" when there are several.
- WhatsApp is the reference the user picked for capture UX. Mobbin was used for references.

**Technical constraints.**
- No READ_MEDIA_*, no exact alarms, no location permission. Runtime permissions only:
  - POST_NOTIFICATIONS
  - RECORD_AUDIO
  - CAMERA
  - READ_CONTACTS
- Business logic lives in `packages/core`, tested on real Postgres. Web routes stay thin.
- All syncable writes go through core (`version`, `nextSeq`). Hard deletes need a Tombstone.
- Phone logic stays in `src/lib`, pure and Jest-tested, with thin screens. Components imported by `SaveRow` must not import `expo-router`: it breaks the SwipeRow test's ESM resolution. Pass callbacks instead.
- Hard-coded colours are only allowed through the allow-list in `src/__tests__/DarkMode.test.tsx`. Otherwise use theme tokens.
- Tokens and icons are generated; never hand-edit them.
- Analytics: only allow-listed props, never content.

**User's explicit instructions and workflow.**
- "Implement correctly." Use plan mode for big work; plan files live in `/root/.claude/plans/`.
- The user uses device APKs for testing and wants the APK link when ready.
- Use Mobbin, icons and web search for UI references when asked.
- **Commits:**
  - clear messages;
  - trailer `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` and `Claude-Session: <session url>`;
  - no model IDs in commits, PRs or code.
- **PR bodies** end with "🤖 Generated with [Claude Code](https://claude.com/claude-code)" and the session link.
- **GitHub comments** end with the Claude Code footer.
- Reply on every review thread; fix valid bot findings (Codex P1/P2 have been real bugs).
- **Code style:**
  - match the surrounding code;
  - JSDoc-style comments that explain the user-facing "why", often citing Figma screen IDs or milestone tags (M21, P3…);
  - short, plain copy in UI strings (curly apostrophes "’").
- **Response style:** concise, factual, action-oriented; short status updates; no filler.

## 5. Important context from this chat

**Earlier in the session (done):**
- Heroku go-live (tucck.app) and the Docker v2 manifest fix.
- Clerk production set up.
- Production APK workflow.
- M20 security work (PR #33).
- Claude for Startups application help:
  - company email through a Cloudflare routing address, e.g. `aniketh@tucck.app` (ASSUMPTION: created by the user);
  - Console sign-up by email.
- A note the user asked to keep: the Zoho Catalyst student offer is non-commercial; don't use it for production.

**Bugs the user found on device and their fixes:**
- Image plus text became 2 items → caption.
- The meta line overflowed → `SaveMeta` single line.
- "From <page title>" → share subject used as the title.
- Device-fix batch (`4ff7eb8`):
  - calendar missing Saturday → rows of 7 with flex;
  - suggested tags overlapped;
  - Delete task → `dangerSecondary` button;
  - task edit affordance (chevron);
  - images and PDFs open on tap;
  - long-press menu on tall items;
  - stable space colours (`spaceTint` hashed by space id);
  - the "+" / filter pill rendering.

**Problems hit:**
- Running `git checkout -B` right after a merge once tripped a destructive-git classifier. Use the documented command; it is allowed for restarting from main after a merge.
- A stale shared `dist` gave type errors → rebuild shared.
- Unawaited RNTL `fireEvent` broke later tests → always `await fireEvent...`.
- A prettier/heredoc issue lost the `\\;` escape in `cards.ts`. Fixed; the `.ics`/`.vcf` tests now check that `;` is escaped.
- Time-of-day flakiness in the event reminder test. Use "At the start" (`reminder-0`) in UI tests.
- Codex findings fixed in `3c04dcd`: album query exposed locked or non-image saves; new types missing from filters.
- CodeRabbit (free plan) gets rate-limited. Copilot review is out of quota.

**Do NOT reconsider:**
- An inline recent-photos strip (Play policy).
- Contacts/events as structured notes (the user chose first-class types).
- `expo-contacts` writes or a WRITE_CONTACTS permission.
- `Linking.sendIntent` for the calendar.
- Moving albums into a separate table.

## 6. Open questions and risks

**Open questions:**
- NEEDS CONFIRMATION: production Deploy after the PR #34 merge, including migration `20261013000000_save_groups_details`, which runs `ALTER TYPE "SaveType" ADD VALUE` (fine on PG12+ inside a transaction, as long as it isn't used in the same transaction).
- NEEDS CONFIRMATION: M21 device behaviour hasn't been tested on a real phone:
  - Photo Picker selection order;
  - `presentFormAsync` on the user's Android version;
  - Google Calendar handling of the INSERT intent extras;
  - `geo:` links.
- Settings rework scope details (exact grouping, entry point from each tab) are not specified beyond the OVERVIEW note. Propose them in the plan.

**Risks and debt:**
- **Older app builds** sync saves with types `contact`/`event` they don't know. The server sends them in pulls, which an older phone build may mis-render. ASSUMPTION: acceptable since there are no public users yet.
- **The web Inbox** folds albums only within the loaded page. A group split across a page boundary shows as two parts.
- **Album captions** live on one photo's `note`. Deleting that photo drops the caption.
- **`pendingPhotos`** is an in-memory handoff. If the process dies between the picker and the send screen, the screen closes empty (by design).
- **`clearSaveReminder`** enqueues `reminder.clear` with `target: "save"`. It is verified only by unit flow, not against a live server.
- **Contact and event text** is duplicated into `content`. Editing `details` must keep `content` in sync; the event form does, but any other future editor must too.
- **Codex runs on "ready for review"**: mark PRs ready only when they are green.
- **CodeRabbit** may be rate-limited for about 50 min.

## 7. New-chat kickoff prompt

> This is a continuation of an existing project: tucck (Android Expo app + Next.js web + Postgres core). The repo is `aniketh703/tucck` (clone or attach it; it was at `/home/user/tucck`). Read `HANDOFF.md` (from my previous session), then `CLAUDE.md` and `docs/OVERVIEW.md`, before changing anything.
>
> Immediate objective: M21 (captions, albums, contacts, events) is merged (PR #34, head 3c04dcd), and its APK is built (Actions run 38358730710 → artifact `tucck-apk`). Next comes the **Settings rework** milestone.
>
> First actions:
> 1. Restart the work branch from main: `git fetch origin main && git checkout -B claude/figma-screens-app-dev-bqtnwc origin/main`. Then `service postgresql start`, `pnpm install`, `pnpm build --filter "./packages/*"`.
> 2. Check that Actions → Deploy on the latest main commit is green (the M21 migration must have applied). Report it if red.
> 3. Give me the APK link and the M21 device test list from HANDOFF §3, and wait for my feedback if I send any.
> 4. Inspect the current Settings/Account screens and tab layout in `apps/mobile/src/app`, then propose a plan for the Settings rework (plan mode) before any large edits.
> 5. After approval: implement, run the full gate (`pnpm format:check && pnpm lint && pnpm typecheck && pnpm test` and `pnpm --filter @tucck/mobile check:permissions`), update `docs/OVERVIEW.md`, push, open a draft PR, watch CI and reviews, and dispatch the Android APK workflow.
>
> Keep responses concise and action-oriented; no unnecessary explanations.
