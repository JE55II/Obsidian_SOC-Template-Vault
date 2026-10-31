# SOC Template Vault for Obsidian

A ready-to-use [Obsidian](https://obsidian.md) vault for SOC analysts. Press one shortcut, answer three questions, and you get a ticket note that is **named, filled in, tagged, filed in the right folder, and timed**. Meetings and night-duty (on-call) notes work the same way. A dashboard shows your ticket activity as year heatmaps, and three ready-made tables list every ticket and meeting.

No coding needed to use it. If you want to adapt it, every template is plain text and the settings sit at the top of each file.

| You want to… | Read |
|---|---|
| Install it and get the first note working | [Quick start](#quick-start) below, then [Docs/Setup.md](Docs/Setup.md) |
| Learn the daily routine (tickets, on-call, meetings, dashboard) | [Docs/Using-the-vault.md](Docs/Using-the-vault.md) |
| Understand or change how it works | [Docs/How-it-works.md](Docs/How-it-works.md) |
| Fix a problem | [Docs/Troubleshooting.md](Docs/Troubleshooting.md) |

---

## What you get

- **3 templates** that automate note creation: SOC ticket, night-shift (on-call) ticket, client meeting.
- **Automatic time tracking**: the timer starts when the note is created and stops when you tick the task.
- **Customer name matching**: `customer_name`, `Customer Name` and `Customer-Name` are treated as one customer.
- **A dashboard** with a vault graph and two heatmaps (SOC tickets, on-call tickets).
- **3 Bases** (database-style tables): SOC tickets, on-call tickets, meetings.
- **Daily notes**, a task roadmap note and a list of common SOC web tools.
- **21 community plugins already installed and configured**. Nothing to download from the plugin store.

---

## Quick start

> New to Obsidian? Obsidian is a free note-taking app that stores plain Markdown files in a normal folder. That folder is called a **vault**. This repository *is* a vault.

1. **Install Obsidian** from <https://obsidian.md/download> (Windows, macOS, Linux).
2. **Get this vault.** Click **Code → Download ZIP** on the repository page and unzip it somewhere permanent (for example `Documents/SOC-Vault`). Or use `git clone`.
3. **Open it.** Start Obsidian → **Open folder as vault** → choose the unzipped folder.
4. **Trust it.** Obsidian asks *"Do you trust the author of this vault?"* → click **Trust author and enable plugins**. Without this, the templates and the dashboard do not work. On this first launch the **dashboard** opens (left tab) next to this **README**. Afterwards Obsidian reopens whatever you had open last.
5. **Set your ticket prefix** (once). Open `Obsidian-Templates/templates/- DailySOC tickets.md`, and at the very top change:
   ```js
   const TICKET_PREFIX = "placeholder-ticket-id"   // for example "INC"
   ```
   Do the same in `- Night-shift-Duty.md`. Details: [Docs/Setup.md](Docs/Setup.md#2-set-your-ticket-prefix).
6. **Make your first ticket.** Press **`Ctrl + S`** (**`Cmd + S`** on Mac), pick `- DailySOC tickets`, and answer the pop-ups. A note named `INC-1234` (or whatever you typed) appears in `♻️ DailyNotes/Soc Operation/SocTickets/…`.

That is all you need to start. Everything else is optional.

---

## The daily routine in 30 seconds

| I want to… | Do this |
|---|---|
| Open today's daily note | Click the **calendar icon** in the left bar |
| Create a SOC ticket note | **`Ctrl/Cmd + S`** → `- DailySOC tickets` |
| Create a night-duty note | **`Ctrl/Cmd + S`** → `- Night-shift-Duty` |
| Create a client meeting note | **`Ctrl/Cmd + S`** → `- meeting` |
| Stop the timer | Tick the checkbox on the `⏱️` line at the top of the note |
| See my activity | Click the **dashboard icon** in the left bar |
| List all tickets or meetings | Open a file in the `Bases/` folder |
| Search notes | **`Ctrl/Cmd + Shift + F`**. For a smarter search that also reads images and PDFs, run the command *Omnisearch: Vault search* (`Ctrl/Cmd + P`) |

> `Ctrl/Cmd + S` normally means *Save*. Obsidian saves automatically, so this vault reuses the shortcut for "new note from template". You can change it in **Settings → Hotkeys**.

Full walkthrough with examples: [Docs/Using-the-vault.md](Docs/Using-the-vault.md).

---

## What is in the vault

```
♻️ DailyNotes/                    daily notes, plus everything filed under it
  Daily Usage/                    roadmap tracker note, SOC tools link list
  Soc Operation/SocTickets/       SOC ticket notes, by year / month / day
🔎⏰On-Call/                       night-duty notes, by year / month
🏢 Clients/📊 Monthly Meetings/   meeting notes, by year / month
Bases/                            3 ready-made tables (tickets, on-call, meetings)
Obsidian-Templates/
  templates/                      the 3 working templates  <- edit these
  ClientFolderTemplate/           empty folder set to copy for a new client
Docs/                             this documentation
.obsidian/                        Obsidian settings, plugins, styling (hidden folder)
```

Folders for tickets, on-call notes and meetings are created automatically the first time a template needs them, so some are empty in a fresh vault.

---

## Good to know

- **It only changes what you tell it to.** Templates run when you start them; nothing rewrites existing notes.
- **Everything is local.** Your notes are plain files on your disk, and none of the bundled plugins needs an account.
- **No account needed.** Obsidian Sync and Publish (paid Obsidian services) are switched off. Turn Sync on in **Settings → Core plugins** only if you subscribe to it.
- **Licence.** The original content (templates, docs, Bases, settings) is CC0 / public domain: see `LICENSE`. The bundled plugins keep their own licences: see `THIRD-PARTY-NOTICES.md`.
- **The dashboard is a modified build** of the open-source *Aurora Dashboard* plugin, bundled in the vault under its own name, *Aurora Dashboard (SOC)*. Do not replace it with the original from the plugin store. Source: <https://github.com/JE55II/custom_aurora-dashboard>.
- **Obsidian's interface language** can be changed in **Settings → General → Language**. The notes and templates are in English.

---

## Side-note

This vault comes from day-to-day SOC work. The author also publishes personal notes and research online. They are **not needed** to use the vault, but you may find them useful:

- [Koffei: README](https://publish.obsidian.md/koffei/%F0%9F%93%A1Koffei/README): the entry page
- [Daily CyberNews: Index](https://publish.obsidian.md/koffei/%F0%9F%93%A1Koffei/Daily+CyberNews/%F0%9F%93%86+Daily+CyberNews+Index)
- [Threat Actors: Index](https://publish.obsidian.md/koffei/%F0%9F%93%A1Koffei/Articles/Threat-Actor+Mapping/%F0%9F%8E%AD+Threat+Actors+Index) (Threat Actor Mapping)
- [CVE Report: README](https://publish.obsidian.md/koffei/%F0%9F%93%A1Koffei/Articles/CVE-Report/README)
- [Report Samples](https://publish.obsidian.md/koffei/%F0%9F%93%A1Koffei/Report+Samples/Report+Samples)
- [ThreatActor Map](https://je55ii.github.io/koffei-threat-map/)

---

## Troubleshooting (the three most common)

| Symptom | Fix |
|---|---|
| Nothing happens when I press `Ctrl/Cmd + S` | Community plugins are off. **Settings → Community plugins → Turn on community plugins**. |
| The note contains raw code like `<%* … %>` | You used Obsidian's built-in *Templates* command. Use **Templater** (`Ctrl/Cmd + S`). |
| Ticket notes are named `placeholder-ticket-id-…` | You have not set your ticket prefix yet. See step 5 above. |

More: [Docs/Troubleshooting.md](Docs/Troubleshooting.md).
