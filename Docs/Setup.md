# Setup

Everything you need to install the vault and configure it for your own team. About 10 minutes.

← Back to the [README](../README.md) · Next: [Using the vault](Using-the-vault.md)

**On this page:** [1. Install](#1-install) · [2. Set your ticket prefix](#2-set-your-ticket-prefix) · [3. Set up the meeting template](#3-set-up-the-meeting-template) · [4. Check that it works](#4-check-that-it-works) · [5. Settings worth knowing](#5-settings-worth-knowing) · [Glossary](#glossary)

---

## 1. Install

### 1.1 Install Obsidian

Download the free app from <https://obsidian.md/download> and install it like any other program (Windows, macOS, Linux). You do not need an Obsidian account.

### 1.2 Get the vault

You have two options:

- **Download ZIP:** on the repository page click **Code → Download ZIP**, then unzip it to a folder you will keep, for example `Documents/SOC-Vault`.
- **Git:** `git clone <repository-url> SOC-Vault`.

Do not run the vault from a temporary folder (such as *Downloads* or an unopened ZIP): Obsidian writes your notes into this folder.

### 1.3 Open it in Obsidian

1. Start Obsidian.
2. Click **Open folder as vault** and pick the folder from step 1.2.
3. Obsidian asks **"Do you trust the author of this vault?"**. Click **Trust author and enable plugins**.

Why this question? The vault ships with plugins, and plugins are programs. Obsidian asks before running any code from a vault it has not seen. Without trust, the templates and the dashboard stay off. If you clicked the wrong button: **Settings → Community plugins → Turn on community plugins**.

### 1.4 What to expect on first open

- Two tabs open side by side: the **Dashboard** (left tab) and this **README**. This only happens the first time; on later starts Obsidian reopens your last session.
- The file explorer on the left and a column of icons (calendar, dashboard…) in the left bar.
- Some folders are empty. That is normal: they are created when you make your first note.
- If Obsidian says a plugin was updated, ignore it for **Aurora Dashboard (SOC)** (see [Troubleshooting](Troubleshooting.md#the-dashboard-or-heatmaps-changed-after-a-plugin-update)).

---

## 2. Set your ticket prefix

Ticket notes are named `<prefix>-<number>`, for example `INC-1234`. The vault ships with a placeholder prefix, `placeholder-ticket-id`, that you replace once.

1. In the left file explorer open `Obsidian-Templates` → `templates`.
2. Open `- DailySOC tickets`. At the very top you see:

   ```js
   // ==================================================
   // TICKET ID SETTINGS: edit this block only
   // ==================================================

   // Replace "placeholder-ticket-id" with your ticket system prefix.
   const TICKET_PREFIX = "placeholder-ticket-id"

   // Text of the pop-up asking for the ticket number
   const TICKET_PROMPT = "Ticket ID"
   ```

3. Change `"placeholder-ticket-id"` to your prefix, for example `"INC"`. Keep the quotes. Optionally change `TICKET_PROMPT` to the wording you like.
4. Repeat in `- Night-shift-Duty`. The two templates keep **separate copies** on purpose, so use the same prefix in both unless your on-call tickets really use a different one.

Obsidian saves automatically; there is nothing to restart.

**What this does:** whatever the user types, the note is named with your prefix. `1234`, `inc 1234`, `INC1234` and `inc-1234` all become `INC-1234`. A bundled note such as `INC-1 INC-2` counts as two tickets in the heatmaps.

The dashboard heatmaps read this same line from the templates, so there is nothing to configure there.

---

## 3. Set up the meeting template

`- meeting` has its own settings block at the top, because meetings need lists of choices. Open `Obsidian-Templates/templates/- meeting` and edit:

```js
// Meeting types (first pop-up)
let typeOptions = ["REPLACEME-type-1", "REPLACEME-type-2"]

// Providers (second pop-up)
let providerOptions = ["REPLACEME-provider-1", "REPLACEME-provider-2"]

// Task Time Tracker project the meetings are filed under
const TT_PROJECT = "Meetings"

// Participants, shown group by group (add or rename groups freely)
let groups = {
    "Analysts": ["REPLACEME-1", "REPLACEME-2"],
    "CSM": ["REPLACEME-3", "REPLACEME-4"],
    "Business": ["REPLACEME-5", "REPLACEME-6"]
};
```

Replace every `REPLACEME-…` with your own values, add as many as you need, and keep each value **unique inside its list** (the participant picker marks choices by name). For example:

```js
let typeOptions = ["Monthly", "Quarterly", "Workshop"]
let providerOptions = ["Acme Telecom", "Globex"]
let groups = {
    "Analysts": ["Sam", "Alex", "Robin"],
    "Customer success": ["Jordan", "Casey"],
    "Business": ["Taylor"]
};
```

You can rename or add groups; each group becomes one participant picker, and groups with nobody selected are left out of the note.

---

## 4. Check that it works

Do this once to be sure the plugins are active.

1. Press **`Ctrl + S`** (**`Cmd + S`** on Mac). A list of templates appears. Pick `- DailySOC tickets`.
2. Type a test ticket number (`1`), an alert name (`test alert`) and a customer (`Test Customer`).
3. If asked about the customer ("not found, create it?") choose the **➕ create** option.

You should now have:

- a note named `<your prefix>-1` inside `♻️ DailyNotes/Soc Operation/SocTickets/<year>/<month>/<day>/`;
- properties at the top: `date`, `client`, `Alert name` and tags (the ticket ID, `test_customer`, `soc_operation`);
- a task line starting with `⏱️`, and a running timer;
- an entry in **Bases → SOC Tickets**, and a dot on the **SOC Operations** heatmap on the dashboard.

Tick the `⏱️` checkbox to stop the timer.

**Clean up the test:** delete the test note, and (optional) run *Task Time Tracker: Open time log panel* from the command palette and remove the test entry and the `Test Customer` project.

If anything above fails, see [Troubleshooting](Troubleshooting.md).

---

## 5. Settings worth knowing

You rarely need to change these, but they explain how the vault behaves.

| Setting                             | Value in this vault                        | Why                                                                                                                                                                       |
| ----------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Date format (everywhere)            | `yyyy MMMM DD`, e.g. `2026 October 31`     | Daily notes, links and plugins must all agree. See [How it works](How-it-works.md#date-format).                                                                           |
| New attachments                     | A `📷 pictures` subfolder next to the note | Pasted screenshots stay with the note they belong to.                                                                                                                     |
| Automatically update internal links | On                                         | Renaming or moving a note updates links to it.                                                                                                                            |
| Detect all file extensions          | On                                         | PDFs, images and spreadsheets show up in the file explorer.                                                                                                               |
| Shortcut `Ctrl/Cmd + S`             | *Templater: Create new note from template* | The usual *Save* shortcut is freed because Obsidian autosaves. Change it in **Settings → Hotkeys**.                                                                       |
| Startup                             | Restores your last session                 | Search **Settings** for *Default file to open* (choices: *Last opened*, *Daily note*, *Specific file*). The dashboard and README are only shown on the very first launch. |
| Interface language                  | Follows Obsidian                           | Change in **Settings → General → Language**. Month names in note titles follow it.                                                                                        |

Three small CSS snippets are enabled (**Settings → Appearance → CSS snippets**): table wrapping, nicer checkboxes, and a heatmap spacing fix.

---

## Glossary

| Term | Meaning |
|---|---|
| **Vault** | A folder of notes that Obsidian opens. This repository is one. |
| **Note** | A Markdown (`.md`) file. |
| **Property** | A field stored at the very top of a note (`date`, `client`…). Shown as a small form in Obsidian. |
| **Tag** | A label such as `#soc_operation` that groups notes. Used by the heatmaps and searches. |
| **Template** | A note that Templater turns into a finished note by asking questions and filling in the answers. |
| **Templater** | The plugin that runs the templates. |
| **Core plugin** | A feature built into Obsidian (daily notes, search, Bases…). |
| **Community plugin** | A plugin installed from outside Obsidian itself. All 21 used here are already in the vault. |
| **Base** | An Obsidian database view: a table built from note properties. |
| **Task Time Tracker** | The plugin that times tasks. Time is grouped into *projects* (here: `SOC-Tickets`, `Meetings`), each tied to a customer. |
| **On-call** | Night or weekend duty. Notes for it are filed under `🔎⏰On-Call`. |
