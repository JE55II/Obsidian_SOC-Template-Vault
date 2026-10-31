# Troubleshooting

Find your symptom, try the fix. Still stuck? Check the [Setup](Setup.md) steps again, then [How it works](How-it-works.md) for the details behind each behaviour.

← Back to the [README](../README.md)

**Jump to:** [Templates](#templates) · [Time tracking](#time-tracking) · [Dashboard and heatmaps](#dashboard-and-heatmaps) · [Bases](#bases) · [Dates and daily notes](#dates-and-daily-notes) · [Plugins and startup](#plugins-and-startup) · [Getting help](#getting-help)

---

## Templates

| Symptom | Cause | Fix |
|---|---|---|
| Nothing happens when I press `Ctrl/Cmd + S` | Community plugins are off, or another app/OS uses the shortcut | **Settings → Community plugins → Turn on community plugins**. If it still fails, open **Settings → Hotkeys**, search *Templater: Create new note from template* and set another shortcut. |
| The new note contains raw code like `<%* … %>` | You used Obsidian's built-in *Templates: Insert template* | Use **Templater**: `Ctrl/Cmd + S`, or `Ctrl/Cmd + P` → *Templater: Create new note from template*. |
| The template list is empty or shows other notes | Templater's folder points elsewhere | **Settings → Templater → Template folder location** must be `Obsidian-Templates/templates`. |
| Ticket notes are named `placeholder-ticket-id-…` | You have not set your ticket prefix | [Setup → Set your ticket prefix](Setup.md#2-set-your-ticket-prefix), in each ticket template. |
| The meeting pop-ups offer `REPLACEME-…` choices | You have not filled the meeting settings | [Setup → Set up the meeting template](Setup.md#3-set-up-the-meeting-template). |
| An error pop-up appears and no note is made | A script error, often after editing a template | Undo your last edit to the template, or re-download the original from the repository. Keep quotes, commas and brackets balanced in the settings block. |
| The note was created but stays in the wrong folder | The move happens at the end; an error stopped it | Move it by hand, then check the template's `baseFolder` line. |
| Typed `inc 1234` but the note is `INC-1234` | Working as designed | The prefix is normalised to the exact spelling in `TICKET_PREFIX`. |

## Time tracking

| Symptom | Fix |
|---|---|
| The timer did not start | Check **Task Time Tracker** is enabled. The timer starts about half a second after the note opens; click into the note and put the cursor on the `⏱️` line and run *Task Time Tracker: Start tracking on current task* from the command palette. |
| A notice says *project link failed* or *plugin not loaded* | Task Time Tracker is disabled or was updated. Re-enable it. The note is still created, but without its customer project. |
| The timer is still running after the ticket is closed | Tick the `⏱️` checkbox on the first line of the note, or run *Task Time Tracker: Stop active tracking*. |
| The same customer appears under two spellings | Pick the **similar customer** option the next time instead of *create*. |
| A customer tag looks wrong or is missing | Tags are made from the customer name: lowercase, spaces become `_`. A name with only symbols or only digits cannot form a valid tag; the note is still created without it. |

## Dashboard and heatmaps

| Symptom | Fix |
|---|---|
| The dashboard icon is missing | The plugin is off: **Settings → Community plugins → Aurora Dashboard (SOC)** → on. |
| The heatmaps are empty ("No notes in …") | They only count notes inside `♻️ DailyNotes/Soc Operation/SocTickets` (SOC) or `🔎⏰On-Call` (night duty) carrying the tags `soc_operation` / `on_call`. Create a ticket with the templates, or check you did not move the folders. |
| Every note counts as 1 ticket | The heatmap could not read your ticket prefix. Check that `const TICKET_PREFIX = "…"` is still in the ticket templates and that **Settings → Templater → Template folder location** is correct. |
| Squares do not update | They refresh a few seconds after Obsidian re-indexes. Switch to another note and back, or reload with `Ctrl/Cmd + R`. |
| I deleted a widget | Open **Toggle dashboard edit mode** from the command palette and re-add it with **Add widget**. |
| The dashboard changed after a plugin update | You updated or reinstalled the original *Aurora Dashboard* from the plugin store. Remove it, keep **Aurora Dashboard (SOC)** (id `aurora-dashboard-soc`) and, if needed, restore this vault's copy of `.obsidian/plugins/aurora-dashboard-soc/`. |

### The dashboard or heatmaps changed after a plugin update

Obsidian's *Check for updates* only touches plugins listed in its store. Aurora Dashboard (SOC) has its own id so it is left alone. If you installed the store's *Aurora Dashboard* as well, you now have two dashboards; disable and remove the store one.

## Bases

| Symptom | Fix |
|---|---|
| A Base is empty | A Base lists notes from one folder only (SOC Tickets, Night-Duty Tickets, Meetings). Make a note with the matching template first. |
| The Date column sorts wrongly or shows text | The `date` property must be a *Date & time*. In a note, click the property icon next to `date` and choose that type, or check `.obsidian/types.json` contains `"date": "datetime"`. Notes whose date was typed with a space instead of the `T` (`2026-10-31 09:42`) are treated as text. |
| Two columns for the same property (`Client` and `client`) | Property names are case sensitive. Rename old notes' `Client` to `client`. |
| A property is empty for tickets (for example Provider, Type, Month) | Expected: those are meeting properties. Tickets leave them blank. |

## Dates and daily notes

| Symptom | Fix |
|---|---|
| Daily notes get different names, or links to "today" break | Date formats differ between settings. See [Date format](How-it-works.md#date-format) and set all to `yyyy MMMM DD`. |
| Month names are in another language | They follow Obsidian's language (**Settings → General → Language**). |
| A new daily note appears every day | Normal: it is created when you open that day's note. Delete empty ones if you like. |

## Plugins and startup

| Symptom                                                 | Fix                                                                                                                                                                     |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Do you trust the author of this vault?" on every start | Click **Trust author and enable plugins** once. If it returns, the vault folder may be read-only or inside a synced folder that resets it.                              |
| Plugins show as disabled after opening                  | **Settings → Community plugins → Turn on community plugins**, then enable the plugins you need.                                                                         |
| The dashboard and README do not open at startup         | They open the first time only. Afterwards Obsidian restores your last session. Open the README from the file explorer, and the dashboard from its icon in the left bar. |
| Pasted screenshots are not where I expect               | They are saved in a `📷 pictures` folder next to the note (**Settings → Files and links → Default location for new attachments**).                                      |

## Getting help

When you report a problem, include:

1. What you did (the template and the answers you typed).
2. What you expected and what happened.
3. Your Obsidian version (**Settings → General**) and operating system.
4. Any error text. Open the developer console with `Ctrl + Shift + I` (`Cmd + Option + I` on Mac) → **Console** and copy red lines.
