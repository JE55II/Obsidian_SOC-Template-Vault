# How it works

Technical reference: what the templates do, what they write, what reads it, and what to change together. Read [Using the vault](Using-the-vault.md) first if you are new.

← Back to the [README](../README.md) · See also: [Setup](Setup.md) · [Troubleshooting](Troubleshooting.md)

**On this page:** [Overview](#overview) · [The template pipeline](#the-template-pipeline) · [Settings blocks](#settings-blocks) · [Data model](#data-model) · [Customer matching](#customer-matching) · [Time tracking integration](#time-tracking-integration) · [Dashboard and heatmaps](#dashboard-and-heatmaps) · [Bases](#bases) · [Date format](#date-format) · [Plugins](#plugins) · [Changing things safely](#changing-things-safely) · [Limits and security notes](#limits-and-security-notes)

---

## Overview

The vault is plain Markdown plus configuration. Three **Templater** templates create notes. Everything else *reads* what those notes contain.

```
                       Templater templates (JavaScript)
              ┌───────────────────────────────────────────────┐
  you  ──►    │ - DailySOC tickets       - Night-shift-Duty   │
 Ctrl/Cmd+S   │ - meeting                                     │
              └───────┬───────────────────────┬───────────────┘
                      │ writes                │ talks to
                      ▼                       ▼
        notes with properties + tags    Task Time Tracker
        in dated folders                (projects per customer, timer)
                      │
        ┌─────────────┼─────────────────────┐
        ▼             ▼                     ▼
     Bases      Dashboard heatmaps      Search, Tasks, Graph
   (tables)    (count tickets per day)  (standard Obsidian)
```

There is no server, database or build step. Everything is files in the vault; the contract between templates and readers is **folders, properties, tags and the ticket-ID prefix** (see [Data model](#data-model) and [Changing things safely](#changing-things-safely)).

---

## The template pipeline

Templater compiles each template into one async JavaScript function; the text between `<%*` and `%>` is the script, and `tR += …` appends text to the new note. Templates run only when you start them (`trigger_on_file_creation_mode` is `none`, so nothing is applied automatically).

Steps of `- DailySOC tickets` (the on-call template is the same with a different folder, extra properties and tags):

1. **Settings block** at the top: `TICKET_PREFIX`, `TICKET_PROMPT`, and the helpers `fixTicket` / `toTicketName`.
2. **Prompts**: ticket ID, alert name, customer name.
3. **Sanitise**: `sanitizeYaml` escapes backslashes and quotes and flattens line breaks so a value can never break the YAML header; the file name has `\ / : * ? " < > |` stripped.
4. **Normalise the ID**: `toTicketName` yields `<prefix>-<number>`; `fixTicket` rewrites a typed `prefix 123` / `prefix123` inside the alert and customer text.
5. **Link the customer** to a Task Time Tracker project ([below](#customer-matching)).
6. **Generate a task ID** (8 random base-62 characters from `crypto.getRandomValues`) and register it with the project *before* the timer starts, so the time lands on the right project.
7. **Rename and move** the note (`tp.file.rename`, `tp.file.move`) to `…/SocTickets/<YYYY>/<MM-MMMM>/<DD>/`. Folders are created if missing.
8. **Write the header and task line** into the note (`tR += …`): properties, tags, and `- [ ] ⏱️ <ID> - <alert> - <customer> [tt-id:: <task id>]`.
9. **Start the timer**: a `tp.hooks.on_all_templates_executed` hook waits 500 ms for the editor to be ready, finds the first `- [ ] ⏱️` line, puts the cursor on it and runs the command `task-time-tracker:start-tracking-current-task`. The box stays unticked: ticking it marks the task done and stops the timer.
10. The rest of the file (below `-%>`) is plain Markdown: the investigation form and the reply draft.

`- meeting` follows the same pattern with a different order (customer first, then type, provider, then participant pickers) and a `Meetings` project.

---

## Settings blocks

Anything a user is expected to edit is gathered in a labelled block at the top of the template, with plain constants:

| Template | Block | Contents |
|---|---|---|
| `- DailySOC tickets` | **TICKET ID SETTINGS** | `TICKET_PREFIX`, `TICKET_PROMPT` |
| `- Night-shift-Duty` | **TICKET ID SETTINGS** | same two constants |
| `- meeting` | **MEETING SETTINGS** | `typeOptions`, `providerOptions`, `groups`, `TT_PROJECT` |

Each template keeps its **own copy** of these blocks and helper functions. That is deliberate: a template is one self-contained file you can copy, edit or delete without touching the others. The price is that a change (for example the ticket prefix) must be made in each template you use. Other constants live next to the code that uses them (`TT_PROJECT = "SOC-Tickets"` for tickets and on-call, `baseFolder` for the target folder).

---

## Data model

### Properties written by the templates

| Property | Written by | Value / format | Type in Obsidian |
|---|---|---|---|
| `date` | all three | `2026-10-31T09:42:00` (local time, unquoted) | **Date & time** (declared in `.obsidian/types.json`) |
| `client` | all three | customer name as chosen | Text |
| `Alert name` | ticket, on-call | alert text | Text |
| `End time`, `Duration` | on-call | empty, filled by hand | Text |
| `type` | meeting | one of `typeOptions` | Text |
| `provider` | meeting | one of `providerOptions` | Text |
| `month` | meeting | month name (follows Obsidian language) | Text |
| `tags` | all three | see below | Tags |

The property is `date`, lowercase, and written as an unquoted ISO value with a `T` between date and time. Quoting it, or using a space instead of the `T`, makes Obsidian treat it as plain text, and Bases could no longer sort or filter by it.

### Tags

| Tag | Meaning | On |
|---|---|---|
| `<prefix>-<number>` | the ticket ID | tickets, on-call |
| `<customer_name>` | the customer, lowercase with `_` (`Acme Corp` → `acme_corp`; empty if nothing usable) | all three |
| `soc_operation` | marks SOC tickets and on-call tickets | tickets, on-call |
| `on_call` | marks night-duty tickets | on-call |
| `<Month>` | meeting month | meetings |

### Folders

| Folder | Written by | Read by |
|---|---|---|
| `♻️ DailyNotes/Soc Operation/SocTickets/<YYYY>/<MM-MMMM>/<DD>/` | ticket template | Base *SOC Tickets*, heatmap *SOC Operations* |
| `🔎⏰On-Call/<YYYY>/<MM-MMMM>/` | on-call template | Base *Night-Duty Tickets*, heatmap *On-Call* |
| `🏢 Clients/📊 Monthly Meetings/<YYYY>/<MM-MMMM>/` | meeting template | Base *Meetings* |
| `♻️ DailyNotes/` | Obsidian (daily notes) | Dashboard streak widget, Calendar |

`MM-MMMM` is a number-and-name month such as `10-October`; the number keeps folders sorted.

---

## Customer matching

Implemented in each template (`normName`, `similarity`, `pickProject`):

- `normName(x)`: strip accents (Unicode NFD), lowercase, delete everything except `a–z` and `0–9`. So `Société-Générale`, `societe generale` and `SOCIETE_GENERALE` all become `societegenerale`.
- **Known customers** come from the Task Time Tracker project list (`projectManager.getProjects()`), one entry per normalised name.
- **Exact match** (same normalised name): use the stored spelling.
- Otherwise, `similarity(a, b)` scores the typed name against every known customer: `0.9` if one contains the other and the shorter has at least 3 characters, else `1 − Levenshtein distance / longer length`. Scores `≥ 0.5` are listed, best first, **at most 8**.
- You then choose a similar customer, **create** a new one, or (when nothing fits) pick from the full alphabetical list. Escape keeps the typed name and links no project.

The same normalisation backs the `customer_name` tag, so tags stay consistent even when the typed names differ.

---

## Time tracking integration

The templates use the **Task Time Tracker** plugin's internal `projectManager` object (`getProjects`, `addProject`, `assignProject`). This is not a public API; if a plugin update renames these methods, the templates still create notes but report "Task Time Tracker plugin not loaded / project link failed" and skip linking.

| Note | Project name (constant) | Client of the project |
|---|---|---|
| SOC ticket | `SOC-Tickets` (`TT_PROJECT`) | the customer |
| On-call ticket | `SOC-Tickets` | the customer (not linked if empty or `unknown`) |
| Meeting | `Meetings` (`TT_PROJECT`) | the client |

A project is created on first use for a customer. The `[tt-id:: …]` text on the task line is how the tracker finds the project later. Data lives in `.obsidian/plugins/task-time-tracker/data.json` (entries, projects, task → project links).

---

## Dashboard and heatmaps

The dashboard is **Aurora Dashboard (SOC)**, plugin id `aurora-dashboard-soc`, in `.obsidian/plugins/aurora-dashboard-soc/`. It is a fork of [stef4678/aurora-dashboard](https://github.com/stef4678/aurora-dashboard) maintained at <https://github.com/JE55II/custom_aurora-dashboard> (see the licence files there). The fork adds the Graph widget and the ticket heatmaps and removes the author-specific sources. It has its own plugin id so the community store cannot "update" it back to the original.

**Heatmap algorithm** (`src/widgets/nerds.ts`):

1. Source *SOC Operations*: all notes under `♻️ DailyNotes/Soc Operation/SocTickets`, tagged `#soc_operation` (property or inline). The **date** is read from the folder path (`…/<YYYY>/<MM-Month>/<DD>`).
   Source *On-Call*: all notes under `🔎⏰On-Call`, tagged `#on_call`. The **date** is the `date` property (or the older `Date`), then a `**Date** : YYYY-MM-DD` line in the body, then the file creation date.
2. **Prefix discovery**: the plugin reads Templater's templates folder (from Templater's own settings, default `Obsidian-Templates/templates`) and extracts every `const TICKET_PREFIX = "…"` with a regular expression. This is why the prefix is set in one place.
3. **Count**: distinct `<prefix>-<number>` IDs in the note's *file name*, case-insensitive (`INC-1 INC-2` → 2). No match counts as 1.
4. Year view from 1 January to today, one column per week, colour level scaled to the busiest day. It refreshes 3 seconds after Obsidian re-indexes the vault.

The folder paths and tag names above are **hard-coded in the plugin** (see [Changing things safely](#changing-things-safely)).

---

## Bases

Three `.base` files in `Bases/`. Each is a table view filtered with `file.inFolder("<folder>")`, showing **File name, Date, Type, Month, Client, Provider, Tags** (`file.name, date, type, month, client, provider, tags`), sorted by `date`, then `client`, then modification time (descending). Properties a note does not have (for example `provider` on a ticket) are simply empty.

To add a column, open the Base and use **Properties**; to change the folder it covers, edit its filter.

---

## Date format

One format everywhere: **`yyyy MMMM DD`** (`2026 October 31`). It must match in each place, otherwise daily notes are created under different names or links break.

| Where | Setting |
|---|---|
| Daily notes (core) | **Settings → Daily notes → Date format** (`.obsidian/daily-notes.json`) |
| Templates (core) | **Settings → Templates → Date format** |
| Natural Language Dates | **Settings → Natural Language Dates → Date format** |
| Aurora Dashboard (SOC) | **Settings → Aurora Dashboard (SOC) → Daily note format** (moment-style `YYYY MMMM DD`, same output) |
| Dataview | **Settings → Dataview → Date format** (`yyyy MMMM dd`, Luxon syntax) |

Dates **inside the template scripts** use different formats on purpose: `YYYY/MM-MMMM/DD` for folder paths (sortable) and `YYYY-MM-DD[T]HH:mm:ss` for the `date` property (the format Obsidian recognises). Tasks always writes `YYYY-MM-DD` and cannot be changed.

---

## Plugins

Everything is already in `.obsidian/plugins/` and enabled via `.obsidian/community-plugins.json`.

| Plugin | Role in this vault |
|---|---|
| **Templater** | Runs the templates. Templates folder: `Obsidian-Templates/templates`. Shortcut `Ctrl/Cmd+S` is bound to *Create new note from template*. |
| **Task Time Tracker** | Timer and per-customer projects. |
| **Aurora Dashboard (SOC)** | Dashboard, graph, ticket heatmaps. Modified build. |
| **Dataview** | Queries over notes and properties. Available for your own dashboards. |
| **Tasks** | Task queries, due dates (Dataview-style `[due:: …]`). |
| **Calendar** | Calendar panel for daily notes. |
| **Natural Language Dates** | `@today`, `@tomorrow`… |
| **Omnisearch**, **Text Extractor** | Full-text search including images and PDFs (OCR). |
| **Multi-Column Markdown**, **Sheets**, **Editing Toolbar**, **Emoji Shortcodes** | Writing and layout helpers. |
| **File Color**, **Hider**, **Style Settings** | Appearance. |
| **Image Converter** | Resize, compress, convert pasted images. |
| **Mind Map**, **Widgets** | Extra views and note widgets. |
| **Better Export PDF** | Cleaner PDF export. |
| **Garble Text** | Blur note text on screen before screenshots. |

Core plugins enabled: file explorer, search, quick switcher, graph, canvas, outgoing links, tags, properties, page preview, daily notes, templates, note composer, command palette, slash commands, status bar, bookmarks, outline, word count, slides, file recovery and **Bases**. **Sync** and **Publish** are off (both need a paid Obsidian account).


---

## Changing things safely

What has to move together if you change one of these:

| If you change… | Also change… |
|---|---|
| The ticket **prefix** | `TICKET_PREFIX` in `- DailySOC tickets` **and** `- Night-shift-Duty`. The heatmaps follow automatically. Old notes keep their old names (rename them if you want them counted). |
| A **target folder** (`baseFolder`) | The matching Base's `file.inFolder(…)` filter, and for SOC tickets / on-call notes the heatmap folder inside the plugin (`src/widgets/nerds.ts` in the plugin repo, rebuilt with `npm run build`). |
| The **`soc_operation` / `on_call` tags** | The same two places: template tag lists and the plugin's heatmap source. |
| The **`date` property name** | The three templates, `.obsidian/types.json`, the three Bases and the plugin's On-Call source. |
| The **date format** | The five settings in [Date format](#date-format). |
| **Template folder** | Templater's *Template folder location* and **Settings → Templates → Template folder location** (both should point to the same folder). |

**Adding a template:** drop a Markdown note with a Templater script into `Obsidian-Templates/templates/`. It appears in the `Ctrl/Cmd+S` list immediately. Copy an existing template's settings block and helpers if you want the same ticket-ID and customer behaviour.

**Adding a heatmap source:** requires editing the plugin (TypeScript in `src/widgets/nerds.ts`, then `npm run build`) and replacing `main.js`. Use the bundled sources as a model.

---

## Limits and security notes

- **Templates are code.** Templater runs JavaScript with full access to your vault. Only run templates you wrote or trust, and read a template before running one you received. This vault's templates run no shell commands (`shell_path` is empty, no user-script folder is configured).
- **Trust prompt.** Obsidian's "Trust author" decision applies to the whole vault's plugins. Review `.obsidian/plugins/` before trusting a copy from someone else.
- **Duplicated logic.** Prefix handling, customer matching and the project picker are copied into each template. Fixing a bug means fixing it in each copy.
- **Hard-coded plugin paths.** The heatmap folders and tags are constants inside the plugin build.
- **Tracker dependency.** Linking and auto-start rely on Task Time Tracker internals (see above).
- **Digit-only customers.** A customer name made only of digits produces a tag Obsidian rejects (`#123`); the note is still created.
- **Old notes** made before a change are not migrated. Rename or re-tag them yourself.
- **Not tested** against every Obsidian version; the plugin manifest requires Obsidian 1.13.0 or newer.
