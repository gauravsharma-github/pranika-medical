# Child Medical Records - single-file webpage

A private, self-contained webpage for keeping a child's medical information in one place:
a summary of key details, the medical history with linked PDFs from Google Drive, and
daily therapy sessions with a link to the Therapy Tracker Google Sheet.

Everything lives in **one file: `index.html`**. There is no server, no database, no
installation and no build step. Your data stays inside the file, on your own device.

---

## Quick start

1. Keep `index.html` somewhere private (your laptop, or a private Google Drive folder).
2. **Double-click it** to open it in any browser (Chrome, Edge, Firefox).
3. To update it: open the same file in a **text editor** (Notepad, VS Code),
   edit only the **STEP 1 / 2 / 3 blocks** inside the `<script>` section,
   save, and refresh the browser tab.

> Tip: internet is only needed for the nicer fonts and for viewing the
> Drive PDFs / Google Sheet. The rest of the page works offline.

---

## What's on the page

| Section | What it shows |
|---|---|
| **Patient header** | Name, date of birth with the age calculated automatically, blood group, latest height/weight. Also the **Print summary** button - a clean printout to carry to doctor visits. |
| **Info cards** | Ongoing conditions, current medications, and the care team (pediatrician, neurologist, therapy centre) plus emergency contacts with tap-to-call numbers. |
| **Medical history** | One card per record, grouped by year, with tabs per category (tabs appear only for categories that have entries), a search box, and **View PDF** / **Open in Drive** buttons. |
| **Daily therapy sessions** | The green **Therapy Tracker** bar (quick-view popup + open the Google Sheet), then one card per session with mood/prompting chips, skills, instructions, words used, basics (food/water/bathroom) and therapist/parent notes, grouped by month. |
| **How to edit** | A collapsible on-page copy of the editing instructions. |

---

## Where your data lives - the three STEP blocks

Open `index.html` in a text editor and scroll down to the `<script>` tag.
Everything you ever edit sits between the big `STEP` banners.

### STEP 1 - `CHILD`
Basic details: `name`, `dob` (format `YYYY-MM-DD` - the age is calculated from it),
`bloodGroup`, `heightCm` / `weightKg`, `conditions`, `medications`,
`pediatrician`, `careTeam`, `emergencyContacts`, optional `notes`, and
`lastUpdated` (shown in the footer - update it whenever you edit the file).

### STEP 2 - `RECORDS` (medical history)
One `{ ... }` block per record. To add a record, copy an existing block,
paste it below, and edit the values. Fields:

| Field | Meaning |
|---|---|
| `date` | `"YYYY-MM-DD"` |
| `category` | One of: `Vaccination`, `Doctor Visit`, `Neurology`, `Therapy Assessment`, `Lab Report`, `Prescription`, `Hospitalization`, `Other` |
| `title`, `doctor`, `facility`, `summary` | Free text |
| `driveLink` | The record's PDF in Google Drive: right-click the file in Drive → **Share** → **Copy link** → paste it here (keep the quotes) |

### STEP 3 - `TRACKER` + `THERAPY_LOGS` (therapy)
- **`TRACKER.sheetLink`** - the Therapy Tracker Google Sheet link
  (Sheets → **Share** → **Copy link**). Once pasted, the green bar shows
  **Quick view** (read-only popup) and **Open sheet**.
- **`THERAPY_LOGS`** - one `{ ... }` block per session, using the same
  fields as the sheet's *Daily Log* tab:

| Field | Type | Shown as |
|---|---|---|
| `date` | `"YYYY-MM-DD"` (required) | Session heading |
| `therapyType` | text (e.g. `"ABA"`) | Purple badge |
| `sessionDuration` | text (e.g. `"180 min"`) | Next to the date |
| `mood` | `Very good` / `Good` / `Okay` / `Difficult` | Colour-coded chip |
| `promptingNeeded` | `None` / `Minimal` / `Moderate` / `A lot` | Colour-coded chip |
| `challengingBehaviour` | short text, or `null` | Red highlight line |
| `mainSkillsWorkedOn` | list **or** sentence | Grey chips / paragraph |
| `instructionsFollowed` | list **or** sentence | Blue chips / paragraph |
| `communicationWordsUsed` | list **or** sentence | Green word chips |
| `foodSnack`, `water`, `bathroom` | text | One compact "basics" line |
| `otherActivities` | text | Paragraph |
| `therapistNotes`, `parentNotes` | text (`\n` = line break) | Bordered note blocks |
| `driveLink` | optional Drive link | "Daily report" button |

**Two rules worth remembering:**
- A `["list", "of", "items"]` renders as chips; a plain `"sentence"` renders as text.
  Both are fine, per field, per entry.
- `null` or `""` means *not recorded* - the field simply doesn't appear. Nothing
  ever shows as "null" or as an empty label.

---

## How the code works (brief)

The file has three layers, top to bottom:

1. **CSS** (inside `<style>`) - the look. All colours and fonts are defined once
   as variables in `:root` and reused everywhere, so re-theming means changing
   a few lines.
2. **HTML** (inside `<body>`) - the skeleton. Mostly *empty* containers, each
   with an `id` (e.g. `id="timeline"`). They are placeholders for JavaScript
   to fill.
3. **JavaScript** (inside `<script>`) - two parts:
   - **Your data**: the `CHILD`, `RECORDS`, `TRACKER` and `THERAPY_LOGS`
     constants (the STEP blocks).
   - **Rendering code**: small helper functions (date formatting, age
     calculation, extracting the file ID from a Google link, HTML-escaping)
     plus one `render...()` function per page section. Each render function
     reads the data, builds HTML text from it, and injects it with
     `.innerHTML`. On page load everything renders once; typing in a search
     box or clicking a tab updates a small `state` object and re-runs the
     relevant render function.

The document popup works by converting any normal Google share link into
Google's read-only `/preview` address (for Drive files, Sheets, Docs and
Slides) and loading it in an `<iframe>`. That's also why viewers must be
signed in to a Google account that has access to the file.

Every piece of your data passes through `esc()` before being placed into
HTML, so quotes, `<`, `&` etc. in notes can't break the page.

---

## Troubleshooting

| Problem | Likely cause / fix |
|---|---|
| Page is blank after editing | A missing comma or quote in a STEP block. Press **F12** → *Console* in the browser; the error message shows the line number. Undo the last edit and retry. |
| Preview popup is empty | Sign in to the Google account that has access to that file, or use the **Open** button instead. |
| "No PDF linked yet" on a record | That record's `driveLink` still contains the `PASTE_...` placeholder. |
| Tracker bar shows a hint instead of buttons | `TRACKER.sheetLink` still contains the placeholder. |
| A tab is missing | Tabs appear only for categories that have at least one record. |

---

## Privacy

- This file contains personal medical information. **Keep it private** - on your
  own device or a private Drive folder. Don't upload it to a public website or
  a public code repository. (Note: the name `index.html` is the default homepage
  name on web hosts - if this file is ever placed on a hosted site, it becomes
  the site's front page.)
- To share with family, share the file itself privately, and share the linked
  Drive PDFs / the tracker sheet with their specific Google accounts as *Viewer*.
