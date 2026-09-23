# Hyre Training Academy

Hyre Training Academy is the online onboarding and training site for Hyre's
Virtual Assistants, internal team members, and clients. It walks every new
enrollee through required onboarding material, gates access to role-specific
courses until prerequisites are actually completed, and issues a certificate
once someone finishes a course (and, where required, submits a real project
for review).

**Live site:** https://hyretrainingacademy2026.github.io/hyre-training-academy/

This is currently a GitHub Pages URL. The recommendation on the table is to
move it onto the main Hyre domain (for example `academy.hyreup.com`) so it
reads as part of hyreup.com rather than a separate site.

## What's on the site

The Academy serves three audiences, each with their own portal:

- **VA Portal** — for Independent Contractors. Enrollees must complete, in
  order: the VA Welcome Packet, the New VA Onboarding Checklist, Global
  Compliance training, and Cultural Differences Training. Only after all
  four are cleared do the role-specific courses (Executive Assistant, Admin
  Assistant, Social Media Management, Paid Ads Specialist, Articulation,
  AI & Automation) unlock.
- **Internal Portal** — for Hyre's own staff. Same idea, a three-step
  prerequisite sequence (no Welcome Packet, since that's VA-specific).
- **Client Portal** — currently offers one self-contained resource (Cultural
  Fluency training for clients); the rest of the client curriculum is still
  in production.

Every course enforces a few rules by design: you can only have one course
"in progress" at a time, lessons can't be marked complete the instant they
load (a short reading-time delay scales with how much content there is), and
a certificate isn't issued until every lesson is done and, for courses that
require it, a real project has been submitted for review.

## How it's built

There's no framework and no build step in the traditional sense — the whole
site is one HTML file (`index.html`) containing all of the markup, styling,
and JavaScript. It's a single-page application: navigating around the site
just swaps out content based on the URL's `#/...` fragment, so it works as a
static file with no server-side code required to render pages.

A couple of pieces do talk to the outside world:

- **Login and registration** go through a Google Apps Script Web App backed
  by a Google Sheet ("Registrations"), which is where approved enrollees and
  their registration details are recorded. Logging in also returns the
  enrollee's registered name, which the certificate name gets checked
  against.
- **Lesson progress and checklist status live primarily in each enrollee's
  own browser** (using a feature called local storage), keyed to their
  verified email — that copy is what the site itself reads to decide what's
  unlocked. Every time it changes, a lightweight snapshot (prerequisites
  cleared, current course and %, certificates earned, last activity time)
  is also mirrored out to that same Registrations sheet, so anyone with the
  sheet open — not just the enrollee, in that one browser — can see how
  someone is doing. That sync is fire-and-forget: if it fails or is
  blocked, the enrollee's own experience is unaffected, so the sheet can
  occasionally lag a browser's local copy.

## Repository layout

Because the deployed file is one large HTML document, it's kept editable by
splitting it into pieces under `build/`:

| Path | What it is |
| --- | --- |
| `index.html` | The actual deployed file — this is what GitHub Pages serves. Generated; don't hand-edit. |
| `build/head.html` | Everything before the page's data and script (head, styles, static markup). |
| `build/content-data.json` | Structured course/lesson content, loaded as JSON at page load. |
| `build/app.js` | All of the site's JavaScript — routing, rendering, progress logic, gating rules. This is where almost all real changes happen. |
| `build/image_tokens.json` | Maps short placeholder tokens inside `app.js` back to the real (large) embedded images, so `app.js` stays small enough to read and edit normally. |
| `assemble.py` | Rebuilds `index.html` from everything above. Run this after any change to `build/`. |
| `test-feedback-fixes.js` | An automated browser test suite (Playwright) covering the prerequisite gates, progress persistence, certificates, and other core behavior. |
| `apps-script-updated.gs` | The Google Apps Script backend's source, kept here for reference. It isn't deployed from this repo — it's pasted into the Apps Script editor attached to the Registrations Google Sheet directly. |

## Making a change

1. Edit the relevant file under `build/` (almost always `build/app.js`).
2. Rebuild the deployed file: `python3 assemble.py`
3. Run the test suite to confirm nothing broke: `node test-feedback-fixes.js`
4. Upload the regenerated `index.html` to GitHub (replacing the existing
   file) and commit. GitHub Pages rebuilds the live site automatically a
   short while after the commit lands on `main`.

## Known limitations and recommendations

- **Progress is now mirrored to a shared dashboard, but it's still
  browser-first.** The Registrations sheet gets a live snapshot of each
  enrollee's prerequisites, current course, and certificates as they
  happen, so anyone with the sheet can check in without needing that
  person's browser. The enrollee's own browser is still the source of
  truth the site itself reads from, though, so a lesson-by-lesson audit
  trail (exactly which lessons, when) would still need a proper backend if
  that level of detail is ever needed.
- **Certificate names can now be cross-checked, once the updated Apps
  Script is deployed.** An enrollee still types their name once and it
  locks permanently, but login now also returns the name on file in the
  registration record, so a mismatch is something the site (or a reviewer)
  can actually detect going forward.
- **Deployment is manual.** Updates go out by uploading the rebuilt
  `index.html` through GitHub's web interface, and backend changes go out
  by pasting the updated script into the Apps Script editor and deploying a
  new version of the existing Web App. A proper CI/CD pipeline would reduce
  the chance of a bad upload reaching the live site.
