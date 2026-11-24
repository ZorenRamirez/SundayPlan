# Sunday Plan

A weekly opportunity planner for WSU Pullman students. Students choose interests, time and location, pick an opportunity and save a concrete action to a date. The working prototype runs in a browser. The account and database can also serve a future iPhone client.

## Run it on the Mac

Open this SundayPlan folder in VS Code. Run `npm start` in its terminal, or select **Terminal → Run Task → Start Sunday Plan**. Open **http://localhost:4173**. Stop the server with Control-C. Use the same origin for Google sign-in.

This project uses Node.js 22.9 or later and Python 3.9 or later. The server uses standard libraries. The browser loads Supabase, Google Sign-In and web fonts online.

| Command | What it does |
|---|---|
| `npm start` | Serves the app on the Mac's loopback address. Checks for a new catalog on startup and hourly while running. |
| `npm test` | Exercises weekly planning, catalog quality and draft synchronization with repeatable fixtures. |
| `npm run refresh` | Fetches WSU clubs and events and the configured public employer boards. Produces one dated catalog in app/data/current.json. |
| `npm run build` | Produces a static site in dist using the configured account backend. Requires the public Supabase settings. |

## Accounts and saves

The local configuration is in .env.local, excluded from Git. Its public setting names are in .env.example. SUPABASE_URL and SUPABASE_PUBLISHABLE_KEY connect the app to the database. GOOGLE_WEB_CLIENT_ID enables Google's own sign-in button. The OAuth client secret belongs in Supabase's Google provider settings. The browser app uses only publishable configuration.

A signed-in student gets one plan row in public.student_plans. PostgreSQL row-level security limits reads and writes to the student's account. Each write checks the previous revision. If two devices disagree, the student can download the draft or use the online copy. An unsynced draft stays in that browser tab. Closing the tab before saving can lose it. A shared browser should use Sign out after a session.

Starting the server with both Supabase settings absent opens the original single-user local mode. It saves to local-data/database.json. Its data is independent of Google accounts. The account-based server disables the single-user database API.

## Catalog and deployment

[Open the public tester](https://sunday-plan-wsu.zorenramirez.chatgpt.site). It runs the same planner code and saves the visitor’s plan in that browser. It uses profile-based recommendations and offers calendar file downloads. The Mac pilot supports Google accounts and Apple’s on-device ranker.

Run `npm run build:tester` to generate the static tester in `hosting/tester/dist`. The tester build uses the public catalog and requires no credentials. Its hosting manifest is in `hosting/tester/.openai/hosting.json`. Republish the build when the catalog or app changes.

A refresh keeps the last valid records and dates for any source that fails. Old events and jobs are restricted in weekly picks. Club schedules may need confirmation. The separately curated entries in local-data/seed.json require editorial review. The Mac must be awake with the server running for its periodic refresh. A static deployment needs a separate scheduler to refresh and publish the catalog.

On a new backend, apply the files in supabase/migrations in order, enable Google under Authentication, and configure its callback and the app's origin on both providers. Before a public launch, register the production HTTPS origin, configure the corresponding site URL and redirects, and complete the Google branding setup.

## Apple Calendar and the current build

The local Mac server opens Apple Calendar’s import dialog when the student selects Apple Calendar. The student chooses a destination and confirms. The browser-only build downloads the same .ics file. A saved preparation step uses the chosen day, while an event uses its organizer’s time and location. Weekly reminders and club meetings keep Pacific wall time through daylight saving changes. Calendar entries are one-time imports.

To confirm which version you are testing, open Profile and check the version at the bottom. The local server serves current files on reload. The light design is the default.

## Personalized plans on the Mac

New students finish onboarding with **Create my plan**. Returning students use **Your personal plan** on This week or Profile. An optional goal and priority guide a ranked list of up to three eligible steps. Creating the plan adds each selected step to Your list on an available day. Reopening the result shows the stored dates and completion status. A previously generated preview has an **Add these steps to my list** button. Editing preferences enables a fresh ranking.

On a supported Apple Silicon Mac with the Swift command-line tools and a macOS 26+ SDK, the local server compiles native/WeeklyRanker.swift into the ignored .plan-runtime directory on first use. The native helper uses English sentence embeddings and tries the Apple Foundation Models generator if available. An authenticated, same-origin request supplies bounded shortlist data to the helper. A timeout, missing model, compiler failure, or static deployment falls back to profile-based recommendations. The result labels the method used. No cloud model key is required.

## Project record

See [Sunday Plan project record](docs/Sunday-Plan-Project-Record.md) for the product decisions, collaboration log, evidence and pilot checklist. At each milestone, update the project record with the decision, what changed, what was tested, and what a student still needs to try. The original Claude export is preserved under original/. The working application is in app/.


## Sign-in and account deletion

The signed-out routes are **#signin** and **#signup**. Both use Google. A returning identity loads its saved plan, and a new identity enters onboarding. Signing out preserves the student’s saved data.

In Profile, **Clear my data** opens a confirmation. **Delete my Sunday Plan account** calls the authenticated database function with no user ID parameter. The database deletes the calling identity and cascades to its student plan. After database confirmation the app clears that student’s tab drafts and opens Create your account. Signing up again with the same email creates a separate identity with a blank plan. The single-user local development mode clears its own saved document and reopens onboarding.
