<%*
// ==================================================
// TICKET ID SETTINGS: edit this block only
// ==================================================

// Replace "placeholder-ticket-id" with your ticket system prefix.
// "INC" -> notes named INC-1234. Must not be empty.
const TICKET_PREFIX = "placeholder-ticket-id"

// Text of the pop-up asking for the ticket number
const TICKET_PROMPT = "Ticket ID"

// "prefix 123", "prefix123", "PREFIX-123" -> "prefix-123" (any text field)
function fixTicket(value) {

    const prefix = TICKET_PREFIX.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")

    return String(value || "")
        .replace(new RegExp(`\\b${prefix}(?:[\\s_-]*(?=\\d)|-)`, "gi"), `${TICKET_PREFIX}-`)
}

// note name ALWAYS "<prefix>-<ticket>": "1234", "prefix 1234" -> "prefix-1234"
function toTicketName(value) {

    const prefix = TICKET_PREFIX.replace(/[.*+?^${}()|[\]\\]/g, "\\$&")

    const ticket = String(value || "")
        .replace(new RegExp(`^\\s*${prefix}[\\s_-]*`, "i"), "")
        .trim()

    return ticket ? `${TICKET_PREFIX}-${ticket}` : ""
}

// --------------------------------------------------
// INPUTS
// --------------------------------------------------

let titleName =
    await tp.system.prompt(TICKET_PROMPT)

let alertName =
    await tp.system.prompt(
        "Alert name"
    )

let customerName =
    await tp.system.prompt(
        "Customer name"
    )

// --------------------------------------------------
// SANITIZE YAML-BREAKING CHARACTERS
// --------------------------------------------------

function sanitizeYaml(value) {

    if (!value)
        return ""

    return String(value)

        // remove line breaks
        .replace(/[\r\n]+/g, " ")

        // escape backslashes (YAML escape char in "...")
        .replace(/\\/g, "\\\\")

        // escape quotes
        .replace(/"/g, '\\"')

        // trim spaces
        .trim()
}

// title is used as a file name -> strip path / file-system characters
titleName =
    toTicketName(String(titleName || "").replace(/[\\/:*?"<>|]/g, ""))

if (!titleName) {
    new Notice(`${TICKET_PROMPT} required`)
    return
}

// "/" is allowed in Alert name (YAML value only, never used in a path)
alertName =
    fixTicket(alertName)

customerName =
    fixTicket(customerName)

// --------------------------------------------------
// TIME TRACKER PROJECT (project "SOC-Tickets", client = Customer name)
// --------------------------------------------------

const TT_PROJECT = "SOC-Tickets"

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

// "SOC-Tickets" project for this client, created if missing
async function socProjectFor(client) {

    const find = () => projectManager.getProjects().find(p =>
        p.name === TT_PROJECT && normName(p.client) === normName(client)
    )

    const found = find()

    if (found)
        return found

    const res = await projectManager.addProject(TT_PROJECT, client)
    const created = find()

    console.log("SOC ticket: addProject", TT_PROJECT, client, res, created)

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

    // known client names (SOC-Tickets projects + other projects), one per client
    const clients = new Map()

    for (const p of projectManager.getProjects()) {

        const k = normName(p.client)

        if (k && (!clients.has(k) || p.name === TT_PROJECT))
            clients.set(k, p.client)
    }

    // typed name is a known client without a SOC-Tickets project yet
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
            `Client for ${titleName} (type to filter)`
        )

        if (picked)
            return socProjectFor(picked)
    }

    // Escape
    new Notice(`Time tracker: no project linked to ${titleName}, check manually`)
    return null
}

let project = null

try {
    project = await pickProject(customerName)
} catch (e) {
    console.error("SOC ticket: project link failed", e)
    new Notice(`Time tracker: project link failed (${e?.message ?? e})`, 10000)
}

// linked -> Client field = project client
if (project)
    customerName = project.client

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

// Safe values for YAML fields
const safeTitle =
    sanitizeYaml(titleName)

const safeAlert =
    sanitizeYaml(alertName)

const safeCustomer =
    sanitizeYaml(customerName)

// tag: lowercase, spaces/punctuation -> "_" ("Customer Name" -> "customer_name")
const customerTag = String(customerName || "")
    .trim()
    .toLowerCase()
    .replace(/[^\p{L}\p{N}_\/-]+/gu, "_")
    .replace(/^_+|_+$/g, "")

const customerTagLine =
    customerTag ? `\n  - "${customerTag}"` : ""


// --------------------------------------------------
// FILE OPERATIONS
// --------------------------------------------------

await tp.file.rename(safeTitle)

// location + date

let baseFolder =
    "/♻️ DailyNotes/Soc Operation/SocTickets/"

let year =
    tp.date.now("YYYY")

let month =
    tp.date.now("MM-MMMM")

let day =
    tp.date.now("DD")

let newFolder =
    `${baseFolder}${year}/${month}/${day}/`

await tp.file.move(
    newFolder + safeTitle
)

// --------------------------------------------------
// METADATA
// --------------------------------------------------

let creationDate =
    tp.date.now("YYYY-MM-DD[T]HH:mm:ss")

// task line for Task Time Tracker (single line, plain text)
const taskLabel =
    [titleName, alertName.replace(/[\r\n]+/g, " ").trim(), customerName]
        .filter(Boolean)
        .join(" - ")

tR += `---
date: ${creationDate}
client: "${safeCustomer}"
Alert name: "${safeAlert}"
tags:
  - "${safeTitle}"${customerTagLine}
  - "soc_operation"
---

- [ ] ⏱️ ${taskLabel}${ttId ? ` [tt-id:: ${ttId}]` : ""}
`

// --------------------------------------------------
// TIME TRACKING (AUTO-START)
// --------------------------------------------------

// Task Time Tracker starts from its command on the cursor's task line.
// The checkbox stays "[ ]": checking it = task done = timer stops.
tp.hooks.on_all_templates_executed(() => {
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
-%>


---
##### Investigation

Duplicated/Link of 

Date & Time : 
User : 
Endpoint : 
OS Version : 
Local Admin : 
Source Endpoint : 
Destination Endpoint : 
Group/Domain : 
File name : 
APK identifier : 
Source IP address : 
Detected IP address :  
Usual IP : 
Endpoint Mac address : 
###### FilePath : 
```

```
###### Command : 
```

```
###### Script : 
```

```
###### Hash (SHA256) :
```

```
###### Reg Key : 
```

```
###### Links :
```

```
###### VT Score :
  /75
###### Community Knowledge :
- 
###### Source Links :
```

```
---
### Invest

Hi,
We received an alert due to 







Is this action legitimate ?

scam / phishing link to 




#### more
Account name : 
Destination : 
Destination 2 : 
Local Admin : 
IP address :  
Local IP address :  
Host address : 
Domain account : 
Process : 
Email subject : 
Sender : 


Link (disabled for safety) : 

`JUSTIFY: <bug> SLA not stopped — at`

