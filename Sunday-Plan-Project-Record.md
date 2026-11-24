# Sunday Plan project record

**Release 0.3.2 · commit 6617b79 · November 23, 2026 (Pacific) · Owner: Zoren Ramirez**

**The public web tester is live. The Mac pilot supports Google accounts and cloud saves. A native iPhone app is a future milestone.**

## Major matching and public testing

A Mathematics student saw a choir, an engineering society and a chemistry club, each labeled “Fits Mathematics.” Claude traced this to a shared college tag. The planner now checks the selected major against each listing. It explains related subjects and names the evidence when only the description matches. We added five WSU listings that were missing from the main directory.

Claude and Codex tested all 74 profile choices. In a fresh clubs-only plan, Mathematics led with Math Club, Chemistry with Chemistry Club and Mechanical Engineering with ASME. The 74 choices produced 69 distinct sets of picks, up from 17. Three majors had no verified club for that test profile and instead linked to their WSU program. A broad, undecided profile still relies on general suggestions.

Claude’s 235 planner checks and 32 simulated account checks passed on the release commit. Codex verified that creating a plan saved three dated steps, changing majors preserved saved work, and the Apple Calendar download contained the selected step and date. The public tester stores each plan in the visitor’s browser. Google sign-in and cross-device saves are available in the Mac pilot. Physical iPhone testing and a real two-device account test are still pending.

[Open the public Sunday Plan tester](https://sunday-plan-wsu.zorenramirez.chatgpt.site).

## The student problem

A WSU student can already search hundreds of clubs, events and internships. Search is not the hard part. The hard question on Sunday is "What should I actually do this week?"

Sunday Plan answers with a short list: up to three specific steps drawn from real listings, each with the reason it fits, the time it takes and a day to do it.

Two rules follow from that, and they shaped most of the build:

- **A step has to be doable this week.** During Thanksgiving week, a December event belongs under Coming up, not at the top of the plan.
- **The app does not state what it does not know.** A club whose meeting time is unposted or alternates gets "ask the organizer", with a question ready to copy. It does not get "go Wednesday".

The first audience is WSU Pullman students. The catalog holds campus clubs and events, internships from 24 public employer job boards, and 39 hand-checked resources.

## What a student can do today

On the Mac, at **http://localhost:4173**, a student can:

1. **Log in or create an account with Google.** A returning student's plan opens; a new student starts onboarding.
2. **Answer a short onboarding.** Six steps cover year, major, interests, weekly time and where to look, plus an optional goal in the student's own words, such as meeting other students who invest.
3. **Create a plan.** The result is up to three real opportunities, each with a reason, an action and the total time. The steps are placed on available days automatically, skipping days marked busy.
4. **Work from This week.** The saved list shows each step with its day. When the week is full, the screen leads with the next saved step, previews up to two relevant dates and offers one idea for later.
5. **Manage a step.** Change its day, mark it done, undo, skip it with a reason, or mark the listing out of date.
6. **Save a club meeting.** Weekly or every two weeks, anchored to a confirmed date.
7. **Add to a calendar.** Google Calendar by link; Apple Calendar through the Mac's own import dialog.
8. **Browse.** Clubs, Internships, Explore, Coming up and a four-year checklist.
9. **Sign out and come back** to the same plan.

Deleting an account is now switched on in the hosted database and was verified there with synthetic students. A deletion in a real browser, with a disposable Google account, is still to run (see below).

## Where the pilot stands

| Area | Status | Evidence |
|---|---|---|
| Sign in, save, return | Working with one real Google account | A saved step was still on the list after sign-out, a fresh tab and a new sign-in. |
| Private per-student data | Access rules verified live | Row-level security on, anonymous reads denied, a second simulated identity saw zero rows. No second real student yet. |
| Plan creation and scheduling | Working on the Mac | Three ten-minute steps were saved on Monday, Wednesday and Thursday around a busy Tuesday, and survived a reload. |
| Catalog refresh | Working | All 26 public sources responded on November 23. |
| Apple Calendar | Working on the Mac | The native calendar chooser opened for a real event. The tester canceled before adding it. |
| Account deletion | **Live in the database; browser run pending** | The function is installed on hosted Supabase. A rollback-only test on the hosted database passed with two synthetic students. A browser walkthrough with a disposable Google account and a second device is pending. |
| Automated checks | Passing | The working copy based on `c58db03` passed 52 Node and 2 Python checks, including the Profile opt-in test. Both builds previously passed for the released version. |
| Public website | Live for public testing | The HTTPS tester accepts anyone with the link. Plans save in each visitor’s browser. |
| iPhone app | Not started | No native build. |

### Pilot, tester and native app

| | Mac browser pilot (now) | Public web tester (live) | iPhone app (future) |
|---|---|---|---|
| Where it runs | A local server on Zoren's Mac | A published HTTPS website, independent of the Mac | A native app |
| Accounts and saves | Google and Supabase, working | Device-local saves in the tester. Hosted Google accounts remain a future milestone. | Same identity and database |
| Matching | Apple’s on-device sentence matching, with a profile-based fallback | Profile-based matching with no model | Native on-device matching, to be built |
| Apple Calendar | Native import dialog | Downloads a standard .ics file | Apple's EventKit event editor |
| Catalog refresh | Hourly check while the Mac is awake and the server is running | Needs its own scheduler | Reads the same published catalog |

## Build decisions

- **Keep the original app and make it readable.** The prototype began as a single Claude app. We preserved that export under `original/` and split the code into ordered, plain files. There is no framework, and the server uses only standard libraries.
- **Browser first, phone later.** The browser lets us test whether the weekly plan is useful before paying for a native build. The account and database are reusable by a phone client.
- **Google sign-in through Supabase, one row per student.** We created a dedicated Supabase project in Oregon, because an older project already served another product (Propstack).
- **Google's own button on the app's address.** The first redirect flow displayed a generated Supabase host name. The redirect remains as a fallback.
- **Rules first, model second.** Catalog rules decide what is eligible. The model only reorders a shortlist and chooses among reasons the app already wrote. Titles, steps, dates and links always come from the catalog.
- **An on-device model with no cloud key.** Matching incurs no cloud model fees, and the student's goal text stays on the Mac for matching. The trade-off is that model matching exists only in the Mac pilot. This milestone used no OpenAI key or API credit.
- **Be honest about source age.** A listing counts as stale after 3 days for events, 7 for job postings, 14 for clubs and 21 for curated entries. Stale events tell the student to confirm on the event page first. Support-group listings can be browsed and are never suggested unless a student opts in.
- **Keep the last good catalog.** A source that fails never erases what was there.
- **Saved steps keep their own copy** of the title and action, so they stay manageable if the listing later leaves its source.
- **Calendar is a one-time export, not a sync.** A change in the planner needs a new export.
- **Deletion is decided by the database.** The app never sends a user ID; the database deletes whoever is signed in.

## How it works

### Accounts and saves

- **Sign-in.** Log in (`#signin`) and Create account (`#signup`) are separate pages, and each shows one Google button. If Google's button fails to load or never appears, a redirect button takes its place. A small help link offers the same redirect. Supabase starts the session. The browser holds only publishable settings. The Google client secret lives in Supabase's provider settings.
- **Storage.** Each student has one row in `public.student_plans`: the plan as JSON plus a revision number. Three owner policies limit reads and writes to that student.
- **Competing edits.** Every write checks the previous revision. If two devices disagree, the student can download the draft or take the online copy.
- **Unsynced changes.** A pending draft is held in that browser tab, per account. Closing the tab before the save succeeds can lose it.
- **Deletion.** In Profile, Clear my data opens a confirmation that names the profile, saved steps, notes and progress. Confirming calls `delete_my_sunday_plan_account()`. The database removes the signed-in identity, and the plan row goes with it. The app then clears that student's drafts, signals other open tabs and shows Create your account. Signing up again with the same email creates a new identity with a blank plan. The student's Google account itself is not touched. Only a signed-in student may call the function; the database refuses guests.
- **Local mode.** Started without the Supabase settings, the server runs the original single-user mode and saves to `local-data/database.json`. We use it for disposable testing.

During Google setup, the first client secret appeared in a setup transcript. It was disabled before use and replaced. No credential values are in this record.

### Catalog refresh

- **Sources.** `npm run refresh` fetches WSU clubs, WSU events and 24 public employer boards into one dated file, `app/data/current.json`. On November 23 it held 399 clubs, 146 events and 71 internship listings. The 39 curated entries live separately and carry their own review dates.
- **Failure handling.** If every source fails, the previous file is kept byte for byte. If one fails, that source's records and dates are kept.
- **Schedule.** The Mac server checks at startup and hourly while it runs.
- **Dates.** Multi-day events keep their end dates (seven in the last run).

### Weekly plan and on-device matching

1. **Screen.** Listings are filtered by interests, year, major, budget, campus access, remaining time, existing commitments, skipped items and out-of-date marks.
2. **Shortlist.** Up to 36 eligible steps go forward. A stated weekly priority narrows the list first when it has matches.
3. **Rank.** The local server hands the shortlist to a small helper compiled from `native/WeeklyRanker.swift` on first use. It compares the student's goal with each listing using Apple's English sentence embeddings, topic overlap and catalog fit. If Apple's Foundation Models text generator is available, it can refine the order.
4. **Validate.** The helper may return only positions in the shortlist and the number of a prewritten reason. Anything else is discarded. The Mac server accepts requests from the app’s own address and checks the signed-in session in account mode. The helper has an 18-second limit. The browser waits up to 27 seconds for the request.
5. **Schedule and save.** The chosen steps are placed on available days and saved to Your list.
6. **Reopen.** Opening a saved plan shows the scheduled steps with their current days and completion status. Changes to planning preferences, the planning week or the catalog date invalidate the cached ranking for the next plan. Previously saved steps stay on the list.

**What the model sees.** The goal, planning preferences, category-level feedback and listing text. It does not see account identity, exact GPA or private notes. Supabase still stores the profile, goal and ranked choices under the per-student rules.

**Fallback.** A timeout, a missing model, a compiler failure or the static website all fall back to profile-based matching. The result labels which method was used.

**What the trial showed.** A first-year Finance student with one hour, Clubs and Scholarships selected and a funding goal received the WSU General Scholarship Application as one 30-minute step. In a networking trial with the same major and time budget, the plan suggested FIRE Club as a one-hour step.

### Apple Calendar

- **A saved step** exports as an all-day entry on the chosen day. The organizer question, when there is one, is included.
- **An event** exports with the organizer's actual time and place.
- **Club meetings and the weekly reminder** repeat in Pacific wall time across daylight saving changes.
- **On the Mac,** one click opens Apple Calendar's own destination chooser, and the student confirms there. Elsewhere the same `.ics` file downloads.

## Evidence

### Automated checks at 0.3.1

| Suite | Checks | Covers |
|---|---|---|
| Weekly planning and saved list | 20 | Event cutoffs, committed time, busy days, uncertain and alternating meetings, undo, archived listings |
| Catalog quality | 12 | Support-group opt-in, stale events, clubs and job postings |
| Calendar | 8 | Mac handoff, browser fallback, dated and all-day entries, daylight saving, overnight events, encoding, invalid input |
| Save synchronization | 4 | Competing edits, failed requests, sign-out race, a lost success response |
| Account | 2 | Sign-in routing, retry after failure, clearing a deleted student's drafts, return to onboarding |
| Ranker input | 1 | Bounds on what the helper will accept |
| Catalog refresh (Python) | 2 | Complete and partial source failure |
| **Total** | **49** | All pass. `npm run build` also passes. |

### Browser walkthroughs

Claude ran these in a disposable local-mode copy with a fixed test clock, never against the real account.

- **Weekly flow:** 51 of 52 on the first pass. The miss was the main button falling below the fold on a 375 × 667 phone with a saved list. After reordering, the button bottom measured 470 px against navigation at 600 px.
- **Day picker:** 46 of 46 at phone and laptop sizes.
- **Sign-in pages:** 34 of 34 at laptop, 390 px and 375 px phone sizes, using a stand-in for Google's button. The checks cover one button per page, the button visible without scrolling, the links between the two pages and both fallback cases.
- **Full control sweep (at 0.2.1):** 195 of 196 across onboarding, saved rows, meetings, calendar files, filters, profile, reset and navigation. It found a copied report showing an internal ID, and a card pointing at a question it did not show. Both were fixed and retested.

Two further defects surfaced during this testing and were fixed: Undo done removed a scheduled step (668f408), and the first tap after typing a GPA was lost (f8890d2).

### Account verification

- **Sign-in round trip.** Google supplied the first name. After onboarding, saving, signing out and signing in from a new tab, the saved steps returned, including after the switch to Google's own button.
- **Access rules.** Live readback showed row-level security enabled, three owner policies and anonymous reads denied. In a read-only transaction the owner saw one row and a different simulated identity saw none.
- **Plan creation.** A plan was generated and reopened through the real signed-in account.
- **Account deletion, isolated.** The checked-in migrations ran on a separate PostgreSQL instance with two synthetic students. Deleting one removed that student's account, linked identity, session and saved plan. The other kept all four. Reusing the first email produced a new identity and an empty list.
- **Account deletion, hosted.** Supabase first confirmed the function was missing from the Sunday Plan project, then accepted migration `202611230002`. The check in `tools/test_hosted_account_deletion.sql` then ran on the actual hosted database, inside a transaction that rolls back. Acting as a signed-in synthetic student, it deleted that student and their plan. A second synthetic student and plan remained. Reusing the first email under a fresh identity found no saved plan. The function's permissions deny guests. All fixture data was rolled back, and no real Google user was deleted.

## Known limitations

- **One Mac and one real Google account.** There has been no second real student, no physical iPhone and no real two-device conflict.
- **Deletion has not been run in a real browser.** The database side is live and verified with synthetic students only. A walkthrough with a designated disposable Google account (delete, then sign up again), with a second device open, is still pending. The redesigned pages also loaded Google’s live buttons in a private Safari window on the Mac. Selecting Sign up opened Google’s email prompt. This browser trial stopped before authentication.
- **The text generator did not run.** Tested rankings came from the sentence model. Apple's Foundation Models service reported itself available but returned a system error in a direct test. Matching is English only.
- **The 0.3.2 control sweep passed.** The account portion used simulated Google and Supabase services.
- **Usefulness is unproven.** We do not yet know whether students act on the first step.
- **Refresh depends on the Mac.** It must be awake with the server running. The curated entries and Handshake-related records still need human review.
- **A draft can be lost** if the tab closes before a save succeeds.
- **Support-group suggestions.** In the working copy, Profile has an opt-in for weekly picks. New profiles start with it off. The generated personal plan excludes support groups even after opt-in.
- **Skips refill without limit.** In the 0.2.1 sweep, 80 skips in a row still left three picks. This was not rechecked at 0.3.1.

## Proposed milestones

1. **Finish verifying account deletion.** The migration is applied and tested on the hosted database. What remains is the browser walkthrough: sign in with a designated disposable Google account, save a step, delete the account, confirm nothing remains, and sign up again to a blank plan, with a second device open.
2. **A second real student and a real iPhone.** Run the same flow in iPhone Safari, and force one two-device conflict.
3. **A small student pilot.** The measure is whether students complete the first step within the week.
4. **Public website.** The public tester is live. For a hosted account beta, register the address with both sign-in providers, verify Google branding and schedule catalog refreshes.
5. **iPhone app.** A native client on the same identity and database, with on-device matching and the EventKit editor.
6. **Smaller product gaps.** A cap on repeated skips and editorial review of curated entries.

## History

Zoren set the product direction and reviewed each screen. Codex integrated the work in the shared checkout and owned accounts, storage, refresh, calendar export, plan creation and the test suites. Claude built the original prototype, split the code, wrote the catalog safeguards, the weekly and discovery screens and the day picker, and ran the reviews and browser walkthroughs.

| Commit | Milestone | Outcome |
|---|---|---|
| 1db63db | Recover the prototype | Imported the original Claude app and configured VS Code. Baseline: 398 clubs, 143 events, 73 internships, 39 curated resources. A test save survived a reload, a server restart and a fresh Incognito session. |
| adf6e88 | Fix weekly planning | A finance freshman's top pick for November 22 to 28 had pointed to a December 1 workshop, and a biweekly club was treated as weekly. Six regression cases now cover event cutoffs, committed time, busy days and uncertain meetings. |
| 2c3a800 | Accounts, saves and refresh | Google sign-in, per-student storage with access rules, revision checks, catalog refresh with last-good data, and the catalog safeguards. |
| 7be9a81 | Dashboard and calendar | The weekly screen redesign, Google's own sign-in button and the native Mac calendar handoff. |
| 668f408 to 0955d83 | QA fixes | Undo keeps the original day. Campus dates display accurately. Out-of-date marks are respected in recommendations. GPA edits no longer swallow the next tap. Organizer questions appear on picks, the next step and its calendar entry. Reports name flagged listings. |
| e931719 | 0.3.0: personal plans | Goal in the student's own words, on-device matching, up to three grounded steps with reasons and total time. |
| c992782 | 0.3.1: scheduling and accounts | Plan steps are scheduled on creation. Explicit Log in and Create account screens. Confirmed account deletion, tested in isolation. |
| Since c992782 | Sign-in pages and hosted deletion | Log in and Create account became separate pages with one Google button each. Migration `202611230002` was applied to hosted Supabase and verified there with a rollback-only test. |

### Design review

Zoren's feedback on the first screen was specific: a large date, two banners and a tall semester checklist pushed the first recommendation below the fold, and a single saved item left a gap beside the checklist. His priority was to see a useful pick and what he had already committed to as soon as the app opens.

We reviewed Todoist's Today and Upcoming views for how a task acquires a date and gets rescheduled, and Handshake's student discovery for showing relevant opportunities first with filters within reach. The result is a compact header, the saved list and top pick visible together, long lists that fold on small screens, and a semester focus that opens on demand. Light is the default theme, including on a Mac set to dark mode.

Zoren's feedback on the sign-in screen was just as specific: the Log in page showed two Google buttons, and Create account showed the same screen again. Log in now has one Google button, with the redirect kept as a small help link. Create account is its own page with its own headline, an example week, and three numbered steps that say what happens next: sign up, answer a few questions, get a first week. A line tells returning students they will go straight to their saved plan.

## Run and review

Zoren sets priorities and approves commits. GitHub Copilot owns implementation, including code fixes and regression tests. Claude helps design the changes, reviews the code and tests the student experience in the browser. Codex maintains this project record and coordinates handoffs. For each change, record the intended behavior, working copy or commit tested, results, and anything still open.


Open the SundayPlan folder in VS Code.

| Command | What it does |
|---|---|
| `npm start` | Serves the app at **http://localhost:4173**. Use this same address for Google sign-in. |
| `npm test` | Runs the Node and Python suites. |
| `npm run refresh` | Fetches the catalog sources. |
| `npm run build` | Prepares the static web files in `dist/`. |

Profile shows the running version at the bottom. `README.md` has the environment and startup details.

**Code map.** Weekly rules: `app/js/06-engine.js`. Plan creation: `app/js/06-weekly-plan.js` and `native/WeeklyRanker.swift`. Accounts and drafts: `app/js/account.js` and `app/js/plan-sync.js`. Catalog rules: `app/js/04-feed-builder.js`. Calendar files: `app/js/ical.js`. Database: `supabase/migrations/`. Hosted deletion check: `tools/test_hosted_account_deletion.sql`.

## References

- **Design:** [Todoist Today](https://www.todoist.com/help/todoist/get-started/plan-your-day-with-the-today-view-UVUXaiSs) · [Todoist Upcoming](https://www.todoist.com/help/todoist/get-started/plan-your-week-with-the-upcoming-view-OKOg1mR8) · [Handshake student discovery](https://support.joinhandshake.com/hc/en-us/articles/360000641047-How-to-Help-Students-Filter-for-Jobs)
- **Accounts:** [Supabase sign-in with Google](https://supabase.com/docs/guides/auth/social-sign-in/auth-google) · [Google brand verification](https://support.google.com/cloud/answer/15549049?hl=en)
- **Calendar:** [Apple EventKit UI](https://developer.apple.com/documentation/technotes/tn3152-migrating-to-the-latest-calendar-access-levels) · [iCalendar specification](https://datatracker.ietf.org/doc/html/rfc5545) · [Carson College event used in the test](https://business.wsu.edu/cnn/)
- **Matching:** [Apple Natural Language embeddings](https://developer.apple.com/documentation/naturallanguage/nlembedding) · [Apple Foundation Models](https://developer.apple.com/documentation/foundationmodels)

## November 24 website review

The first release is a website for WSU students. A student should be able to find an opportunity that fits their major and constraints, understand why it fits, save a next step, and return to their plan. Desktop browser flows are the first review target. Native iPhone development is deferred.

Copilot added save status and retry feedback and corrected personalized opportunity and date lists that ignored the support-group preference. Claude reproduced a separate lost-update problem when two tabs edit the same plan. The review checkout is the November 24 SundayPlan folder. The Terminal screenshot shows the November 23 folder.

The public tester served a catalog built November 23. The November 24 local catalog lists 399 clubs, 145 events and 71 internships. The daily refresh task is scheduled for 6 a.m. Pacific starting November 25. Public delivery still needs a successful write and read-back. The hosting source server could not be reached from this workspace. The 39 curated resources follow a manual review process.

Claude tested a synthetic first-year Mathematics student in a desktop browser. The generated week included Math Club and Future Teachers of Mathematics. The plan survived reload, and completed steps stayed completed on the next week's visit. Claude then reproduced a lost update across two tabs in both browser-only and local-server modes.

Copilot implemented a merge for independent edits and regression tests. Codex independently ran `npm test` (60 Node and two Python tests passed), `npm run build:tester` (passed), and `git diff --check` (passed). The public tester saves in the current browser. Hosted account sync still needs an end-to-end test. A stale open tab can restore a plan after a profile reset. Claude also flagged local-server saves during tab closure and placement of the returning student's plan button for follow-up.

## November 24 website usability update

Zoren asked for a Wazzu crimson and gray website and a weekly list that feels distinct from a list of deadlines. The main heading shows the student's local day and date. Your list shows the week range. Adding an item saves it immediately. Dates belonging to events or deadlines and dates explicitly chosen by the student have distinct labels. Students can mark a step done or undo that action without a follow-up survey. Meeting times are optional inside details. Existing records are preserved.

Copilot implemented the changes. The review considered Todoist's optional scheduling and Handshake's one-click bookmarking. The automated suite covers independent disclosure controls, switching from scheduling to details, month and year boundaries, undated saves, and removal of redundant prompts. Codex ran 70 Node tests and two Python tests successfully and built the account and tester versions. This entry describes the local website build.

Design references: [Todoist dates and scheduling](https://www.todoist.com/help/todoist/features/schedule-a-date-and-time-for-your-todoist-tasks-q7VobO) and [Handshake saved jobs](https://support.joinhandshake.com/hc/en-us/articles/227072527-My-Saved-Jobs-in-Handshake).
