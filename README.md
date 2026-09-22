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
  by a Google Sheet, which is where approved enrollees and their registration
  details are recorded.
- **Everything else — lesson progress, checklist status, certificate
  names — lives only in each enrollee's own browser** (using a feature
  called local storage), keyed to their verified email. It is not sent
  anywhere or visible centrally. This is a known limitation; see
  Recommendations below.

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

## Making a change

1. Edit the relevant file under `build/` (almost always `build/app.js`).
2. Rebuild the deployed file: `python3 assemble.py`
3. Run the test suite to confirm nothing broke: `node test-feedback-fixes.js`
4. Upload the regenerated `index.html` to GitHub (replacing the existing
   file) and commit. GitHub Pages rebuilds the live site automatically a
   short while after the commit lands on `main`.

## Known limitations and recommendations

- **Progress isn't tracked centrally.** There's currently no report or
  dashboard anyone can check to see how an enrollee is progressing —
  everything lives in that person's own browser. Adding real backend
  tracking (logging events to a Sheet or database) is the main piece of
  follow-up work worth prioritizing.
- **Certificate names are self-entered, not verified.** An enrollee types
  their name once and it locks permanently, which prevents printing
  certificates under different names, but it isn't yet cross-checked
  against their registration record. Fixing that requires a small change to
  the Google Apps Script backend so it returns the registrant's name at
  login.
- **Deployment is manual.** Updates go out by uploading the rebuilt
  `index.html` through GitHub's web interface. A proper CI/CD pipeline would
  reduce the chance of a bad upload reaching the live site.
