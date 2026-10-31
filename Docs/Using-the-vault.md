# Using the vault

How to work with the vault day to day. Set it up first: [Setup](Setup.md).

← Back to the [README](../README.md) · Next: [How it works](How-it-works.md)

**On this page:** [The one shortcut to remember](#the-one-shortcut-to-remember) · [Daily note](#daily-note) · [SOC ticket](#soc-ticket) · [Night-duty (on-call) ticket](#night-duty-on-call-ticket) · [Client meeting](#client-meeting) · [Time tracking](#time-tracking) · [Dashboard](#dashboard) · [Bases](#bases-tables-of-your-notes) · [Searching, tasks and other notes](#searching-tasks-and-other-notes) · [Adding a new client](#adding-a-new-client) · [Cheat sheet](#cheat-sheet)

---

## The one shortcut to remember

**`Ctrl + S`** on Windows and Linux, **`Cmd + S`** on Mac.

It opens the template list. Pick a template, answer the pop-ups, and the finished note appears. If the shortcut does nothing, press `Ctrl/Cmd + P`, type **Templater: Create new note from template**, and press Enter.

Do **not** use Obsidian's built-in *Templates: Insert template* command for these templates. It shows raw code instead of running it.

---

## Daily note

Click the **calendar icon** in the left bar ("Open today's daily note") or click a day in the **Calendar** panel (right sidebar; if it is hidden, run the command **Calendar: Open view**).

Daily notes are named like `2026 October 31` and live in `♻️ DailyNotes/`. They are blank on purpose: a place for today's thoughts.

Two reference notes sit in `♻️ DailyNotes/Daily Usage/`:

- **Daily Roadmap Tracker**: a planning layout (priorities, projects, learning, done). Copy what you need into your daily note.
- **Soc Tools**: a link list of everyday SOC web tools (IP and URL reputation, sandboxes, vulnerability sources, security news, CyberChef…). Add your own links.

---

## SOC ticket

**Template:** `- DailySOC tickets`

**You are asked for**

1. **Ticket ID** (the number, e.g. `1234`).
2. **Alert name**.
3. **Customer name**.

**What happens**

- The customer is matched against the customers you already used (see [Customer names](#customer-names)).
- The note is renamed `<prefix>-<number>` and filed in
  `♻️ DailyNotes/Soc Operation/SocTickets/<year>/<month>/<day>/`.
- A timer starts (see [Time tracking](#time-tracking)).

**What you get** (example with prefix `INC`):

```markdown
---
date: 2026-10-31T09:42:00
client: "Acme Corp"
Alert name: "Impossible travel detected"
tags:
  - "INC-1234"
  - "acme_corp"
  - "soc_operation"
---

- [ ] ⏱️ INC-1234 - Impossible travel detected - Acme Corp [tt-id:: AbC123xy]
```

Below that is a ready-to-fill **investigation form** (user, endpoint, IPs, hashes, command line, VirusTotal score, community knowledge, links), a draft **reply** to the customer ("Hi, we received an alert due to…") and a "more" block for extra fields. Fill what applies and delete the rest.

---

## Night-duty (on-call) ticket

**Template:** `- Night-shift-Duty`

Same three questions as a SOC ticket. Differences:

- Filed in `🔎⏰On-Call/<year>/<month>/` (no day folder).
- Two extra empty properties to fill in: `End time` and `Duration`.
- An extra tag, `on_call`. The task line starts with `On-Call`.
- The reply wording is geared to closing alerts ("We are closing the alert as a legitimate true positive" / "…as a false positive").

If you leave the customer empty, it is recorded as `unknown` and no time-tracker project is linked.

---

## Client meeting

**Template:** `- meeting`

**You are asked for**

1. **Note title**: the client name (this is what is matched against known clients).
2. **Meeting type**: from your list ([Setup → meeting template](Setup.md#3-set-up-the-meeting-template)).
3. **Provider**: from your list.
4. **Participants**, one group at a time. Click a name to tick it (a ✅ appears), click more names, then choose **[Done]**.

**What you get:** a note named `<Month> - <Client>` (for example `October - Acme Corp`) in `🏢 Clients/📊 Monthly Meetings/<year>/<month>/`, with:

```markdown
---
date: 2026-10-31T14:05:00
type: "Quarterly"
provider: "Acme Telecom"

tags:
- "October"
- "acme_corp"

month: "October"
client: "Acme Corp"
---

- [ ] ⏱️ Quarterly - October - Acme Corp [tt-id:: Zy98xW7v]

## Participants

**Analysts**
- Sam

**Business**
- Taylor
```

followed by empty sections: Client, Agenda/Questions, To Do List (a Topic / Decision/action / Owner table), Notes, Remarks.

Meetings are timed under the Task Time Tracker project **Meetings** instead of **SOC-Tickets**.

---

## Customer names

People type customers differently: `Acme Corp`, `acme_corp`, `ACME-corp`. The templates treat these as the same customer by ignoring case, accents, spaces and punctuation, and they look the name up among the customers the time tracker already knows.

| What you type | What happens |
|---|---|
| A name that matches an existing customer | The existing spelling is used silently. |
| A name close to existing ones | A list of similar customers appears. Pick one, or choose **create** to keep your spelling. |
| A new name | You are asked whether to **create** it, or **pick from the full list**. |
| Escape on any of those | The ticket is still created; no project is linked, and you get a notice. |

The customer also becomes a **tag** in lowercase with underscores (`Acme Corp` → `acme_corp`), so you can click it to find every note for that customer.

---

## Time tracking

Every ticket and meeting note starts with a line like:

```markdown
- [ ] ⏱️ INC-1234 - Impossible travel detected - Acme Corp [tt-id:: AbC123xy]
```

- The **timer starts automatically** when the note is created.
- **Ticking the checkbox ends the task and stops the timer.** Leave it unticked while you work.
- Do not edit the `[tt-id:: …]` part: it links the line to its customer project.
- To review the logged time, open the command palette (`Ctrl/Cmd + P`) and run **Task Time Tracker: Open time log panel**. To stop a timer without ticking the box, run **Task Time Tracker: Stop active tracking**.

Time is grouped into projects per customer: `SOC-Tickets / Acme Corp`, `Meetings / Acme Corp`.

---

## Dashboard

Click the **dashboard icon** in the left bar (or run **Aurora Dashboard (SOC): Open dashboard** from the command palette).

| Widget | What it shows |
|---|---|
| **Notes** (stats), **Search**, **Progress** | Your vault stats at a glance, a box that runs a global vault search, and how much of the day, month and year has passed |
| **Pomodoro**, **Daily streak** | A focus timer (25 minutes work / 5 minutes break) and the number of consecutive days you have written a daily note |
| **Graph** | The links between all your notes. Hover to highlight, click to open, drag to move, scroll to zoom, double-click to fit. |
| **Heatmap: SOC Operations** | One square per day since 1 January. Darker = more tickets that day. Hover a square for its date and count. |
| **Heatmap: On-Call** | The same for night-duty tickets. |

The heatmaps count **tickets**, using the prefix you set in the templates: a note called `INC-1234` is one ticket, a note called `INC-1 INC-2` is two.

To rearrange: run **Toggle dashboard edit mode** from the command palette, then drag and resize widgets.

---

## Bases (tables of your notes)

Open a file in the `Bases/` folder:

| Base | Lists | Source folder |
|---|---|---|
| **SOC Tickets** | every SOC ticket | `♻️ DailyNotes/Soc Operation/SocTickets` |
| **Night-Duty Tickets** | every on-call ticket | `🔎⏰On-Call` |
| **Meetings** | every client meeting | `🏢 Clients` |

Each Base lists the notes of the category its title names, and shows the same columns: **File name, Date, Type, Month, Client, Provider, Tags**, newest first. They update on their own as you create notes. Columns that do not apply to a category stay empty (for example *Provider* on a ticket). Click a column heading to re-sort, or use the Base's own **Filter** and **Sort** buttons to narrow them down (by client, month, tag…).

---

## Searching, tasks and other notes

- **Search:** `Ctrl/Cmd + Shift + F` for standard search. **Omnisearch** (command palette → *Omnisearch: Vault search*) is smarter and also reads text inside images and PDFs.
- **Tasks:** any `- [ ]` line is a task. Add `[due:: 2026-11-05]` for a due date; the **Tasks** plugin can list tasks across the vault in a code block.
- **Dates in text:** type `@today` or `@tomorrow` to insert a date link (Natural Language Dates).
- **Columns, nicer tables, emoji:** `:smile:` gives 😄; Multi-Column Markdown and Sheets help with layout. See the plugin list in [How it works](How-it-works.md#plugins).
- **Screenshots:** paste an image into a note; it is saved in a `📷 pictures` folder next to the note.
- **Sharing a screenshot safely:** the *Garble Text* plugin blurs note text on screen.

---

## Adding a new client

Copy `Obsidian-Templates/ClientFolderTemplate/` (in the file explorer: right-click → *Duplicate* or *Make a copy*), rename the copy to the client's name, and move it where you like. It contains three empty folders: `🗃️ Archive`, `📷 pictures` and `📒 Reports`.

---

## Cheat sheet

| I want to… | Do this |
|---|---|
| New note from a template | `Ctrl/Cmd + S` |
| Command palette | `Ctrl/Cmd + P` |
| Search | `Ctrl/Cmd + Shift + F` |
| Open today's daily note | Calendar icon (left bar) |
| Open the dashboard | Dashboard icon (left bar) |
| Stop a timer | Tick the `⏱️` checkbox |
| See all tickets / meetings | `Bases/` folder |
| Change a shortcut | Settings → Hotkeys |
