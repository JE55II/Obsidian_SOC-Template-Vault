<%*
// ==================================================
// MEETING SETTINGS: edit this block only
// ==================================================

// Replace every REPLACEME-... value with your own.
// Keep each value unique inside its list.

// Meeting types (first pop-up)
let typeOptions = ["REPLACEME-type-1", "REPLACEME-type-2"]

// Providers (second pop-up)
let providerOptions = ["REPLACEME-provider-1", "REPLACEME-provider-2"]

// Task Time Tracker project the meetings are filed under (created per client if missing)
const TT_PROJECT = "Meetings"

// Participants, shown group by group (add or rename groups freely)
let groups = {
    "Analysts": ["REPLACEME-1", "REPLACEME-2"],
    "CSM": ["REPLACEME-3", "REPLACEME-4"],
    "Business": ["REPLACEME-5", "REPLACEME-6"]
};

// --------------------------------------------------
// CUSTOMER NAME NORMALISATION + TIME TRACKER PROJECT
// --------------------------------------------------

const projectManager =
    app.plugins.plugins["task-time-tracker"]?.projectManager

// "Customer Name", "customer_name", "Customer-Name" -> "customername"
function normName(value) {

    return String(value || "")
        .normalize("NFD")
        .replace(/[̀-ͯ]/g, "")
        .toLowerCase()
        .replace(/[^a-z0-9]/g, "")
}

// 0..1 (1 = same name): containment, else Levenshtein distance
function similarity(a, b) {

    if (!a || !b)
        return 0

    if (Math.min(a.length, b.length) >= 3 && (a.includes(b) || b.includes(a)))
        return 0.9

    let prev = Array.from({ length: b.length + 1 }, (_, j) => j)

    for (let i = 1; i <= a.length; i++) {

        const cur = [i]

        for (let j = 1; j <= b.length; j++)
            cur[j] = Math.min(
                prev[j] + 1,
                cur[j - 1] + 1,
                prev[j - 1] + (a[i - 1] === b[j - 1] ? 0 : 1)
            )

        prev = cur
    }

    return 1 - prev[b.length] / Math.max(a.length, b.length)
}

// TT_PROJECT project for this client, created if missing
async function socProjectFor(client) {

    const find = () => projectManager.getProjects().find(p =>
        p.name === TT_PROJECT && normName(p.client) === normName(client)
    )

    const found = find()

    if (found)
        return found

    const res = await projectManager.addProject(TT_PROJECT, client)
    const created = find()

    console.log("Meeting: addProject", TT_PROJECT, client, res, created)

    if (created) {
        new Notice(`Time tracker: project created\n${TT_PROJECT} / ${client}`)
        return created
    }

    new Notice(`Time tracker: project NOT created (${res?.error ?? "unknown"})\n${TT_PROJECT} / ${client}`, 10000)
    return null
}

// existing client -> project; else prompt: similar clients / create / manual check
async function pickProject(name) {

    if (!name)
        return null

    if (!projectManager) {
        new Notice("Time tracker: Task Time Tracker plugin not loaded, no project linked")
        return null
    }

    const key = normName(name)

    const exact = projectManager.getProjects().find(p =>
        p.name === TT_PROJECT && normName(p.client) === key
    )

    if (exact)
        return exact

    // known client names (meeting projects + other projects), one per client
    const clients = new Map()

    for (const p of projectManager.getProjects()) {

        const k = normName(p.client)

        if (k && (!clients.has(k) || p.name === TT_PROJECT))
            clients.set(k, p.client)
    }

    // typed name is a known client without a meeting project yet
    if (clients.has(key))
        return socProjectFor(clients.get(key))

    const similar = [...clients]
        .map(([k, client]) => ({ client, score: similarity(key, k) }))
        .filter(s => s.score >= 0.5)
        .sort((a, b) => b.score - a.score)
        .slice(0, 8)
        .map(s => s.client)

    const CREATE = "\u0000create"
    const MANUAL = "\u0000manual"

    const choice = await tp.system.suggester(
        [
            ...similar.map(c => `🔗 ${c}`),
            `➕ Yes, create "${TT_PROJECT} / ${name}"`,
            "✋ No, pick a client from the list"
        ],
        [...similar, CREATE, MANUAL],
        false,
        similar.length
            ? `Client "${name}" not found, pick a similar one`
            : `Client "${name}" not found, create it?`
    )

    if (choice === CREATE)
        return socProjectFor(name)

    if (choice && choice !== MANUAL)
        return socProjectFor(choice)

    // manual check: searchable list (type to filter) of every TimeTracker client
    if (choice === MANUAL) {

        const all = [...clients.values()]
            .sort((a, b) => a.localeCompare(b, undefined, { sensitivity: "base" }))

        const picked = await tp.system.suggester(
            all,
            all,
            false,
            `Client for ${name} (type to filter)`
        )

        if (picked)
            return socProjectFor(picked)
    }

    // Escape
    new Notice(`Time tracker: no project linked to ${name}, check manually`)
    return null
}

let qcFileName = await tp.system.prompt("Note Title")

if (!qcFileName) {
    throw new Error("Note title cannot be empty")
}

let project = null

try {
    project = await pickProject(qcFileName)
} catch (e) {
    console.error("Meeting: project link failed", e)
    new Notice(`Time tracker: project link failed (${e?.message ?? e})`, 10000)
}

// linked -> client = project client
if (project)
    qcFileName = project.client

// task id written on the task line: the tracker reuses it, so the
// project link is set now, before the timer starts
let ttId = ""

if (projectManager) {

    const chars = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

    ttId = Array.from(
        crypto.getRandomValues(new Uint8Array(8)),
        b => chars[b % chars.length]
    ).join("")

    if (project)
        await projectManager.assignProject(ttId, project.id)
}

// title is used as a file name -> strip path / file-system characters
qcFileName =
    qcFileName.replace(/[\\/:*?"<>|]/g, "").trim()

let noteType = await tp.system.suggester(typeOptions, typeOptions)

let provider = await tp.system.suggester(providerOptions, providerOptions)

let titleName = tp.date.now("MMMM") + " - " + qcFileName
await tp.file.rename(titleName)

let baseFolder = "/🏢 Clients/📊 Monthly Meetings/"
let year = tp.date.now('YYYY')
let month = tp.date.now('MM-MMMM')
let newFolder = `${baseFolder}${year}/${month}/`

await tp.file.move(newFolder + titleName);

// YAML-safe values
function sanitizeYaml(value) {
    if (!value) return ""
    return String(value)
        .replace(/[\r\n]+/g, " ")
        .replace(/\\/g, "\\\\")
        .replace(/"/g, '\\"')
        .trim()
}

// ⚠️ DEFER MOVE (critical fix) + TIME TRACKING (AUTO-START)
// Task Time Tracker starts from its command on the cursor's task line.
// The checkbox stays "[ ]": checking it = task done = timer stops.
tp.hooks.on_all_templates_executed(async () => {
    try {
        if (tp.file.path(true) !== `${newFolder}${titleName}.md`.replace(/^\//, ""))
            await tp.file.move(newFolder + titleName)
    } catch (e) {
        console.error("Move failed:", e)
    }

    setTimeout(() => {

        const editor = app.workspace.activeEditor?.editor
        if (!editor) return

        for (let i = 0; i < editor.lineCount(); i++) {

            const line = editor.getLine(i)

            if (/^\s*[-*+] \[ \] ⏱️/.test(line)) {

                editor.setCursor({ line: i, ch: line.length })

                const ok = app.commands.executeCommandById(
                    "task-time-tracker:start-tracking-current-task"
                )

                if (!ok)
                    new Notice("Task Time Tracker: timer not started")

                return
            }
        }
    }, 500)
})

// metadata
// tag: lowercase, spaces/punctuation -> "_" ("Customer Name" -> "customer_name")
const customerTag = String(qcFileName || "")
    .trim()
    .toLowerCase()
    .replace(/[^\p{L}\p{N}_\/-]+/gu, "_")
    .replace(/^_+|_+$/g, "")

const customerTagLine =
    customerTag ? `\n- "${customerTag}"` : ""

let currentMonth = tp.date.now("MMMM")
let creationDate = tp.date.now("YYYY-MM-DD[T]HH:mm:ss")

tR += `---
date: ${creationDate}
type: "${sanitizeYaml(noteType)}"
provider: "${sanitizeYaml(provider)}"

tags:
- "${sanitizeYaml(currentMonth)}"${customerTagLine}

month: "${sanitizeYaml(currentMonth)}"
client: "${sanitizeYaml(qcFileName)}"
---

- [ ] ⏱️ ${noteType || "Meeting"} - ${titleName}${ttId ? ` [tt-id:: ${ttId}]` : ""}

`;
-%>
<%*
let selectedByGroup = {};
let output = "";

for (let [group, people] of Object.entries(groups)) {
    selectedByGroup[group] = [];
    let done = false;

    while (!done) {
        let displayList = ["[Done]"].concat(
            people.map(p => selectedByGroup[group].includes(p) ? `✅ ${p}` : p)
        );

        let choice = await tp.system.suggester(
            displayList,
            ["[Done]"].concat(people),
            false,
            `Select ${group} (multiple allowed, stop with [Done])`
        );

        if (choice === "[Done]") {
            done = true;
        } else if (!selectedByGroup[group].includes(choice)) {
            selectedByGroup[group].push(choice);
        }
    }
}

output += "## Participants\n\n";

for (let [group, chosen] of Object.entries(selectedByGroup)) {
    if (chosen.length > 0) {
        output += `**${group}**\n`;
        for (let name of chosen) {
            output += `- ${name}\n`;
        }
        output += `\n`;
    }
}

tR += output;
-%>
**Client** :
- 
  
## Agenda/Questions
- 

## To Do List

| Topic | Decision/action | Owner |
| ----- | --------------- | ----- |
|       |                 |       |

## Notes
- 

## Remarks
- 
