# Cairn design

This is the starting design. Everything here is open to change until the first release; the open questions are listed at the end.

## Why start over

[Pulse](https://github.com/andymitch/pulse) began as a dashboard that planned one person's day from GitHub, Linear, Slack, Mail and meetings. By v4 its most valuable part was something else: a store of short, dated, sourced facts about the work, kept current on every run, that other agents read and add to over MCP. Pulse calls it a context nexus.

The rest of Pulse is built for one person: one repository, one tracker, one chat tool, a briefing prompt written for that stack, and fixed charts. Making it general in place would mean undoing most of it. Cairn starts with the context nexus as the core and builds everything else to depend on what the user connects.

## Who it's for

Just its author at first, but built so anyone can pick it up, including people who aren't engineers:

- Setup is choosing what to connect, not editing config files.
- Nothing assumes a code host, a terminal or a particular tracker.
- It's free and open source forever, so it never depends on a paid developer account (see Distribution).

## Principles

- **Context first.** Every feature reads from or adds to the nexus. If a feature doesn't need context, it probably doesn't belong in Cairn.
- **Newest wins.** When sources disagree, the most recent one is the truth. Older facts are background and history, never overrides.
- **Only ask for what needs a person.** The nexus lives in the background. The interface surfaces what needs a decision, such as facts an agent wants to add or facts Cairn changed its mind about, and answers questions. It doesn't ask people to browse a database.
- **Show loading, not emptiness.** Nothing says "not connected" or "nothing here" until it has actually checked.
- **Local and private.** Everything stays on the user's computer, except what goes to the services and the AI agent they choose.

## The core: the context nexus

### Projects

Everything belongs to a project, such as work and personal, from the first row. Each project has its own sources, facts, tools and dashboards, and an agent asking Cairn says which project it means, or Cairn picks it from the folder the agent is working in.

### Facts

The model carries over from Pulse:

- **Subjects** are real things: an issue in the tracker, a project in the tracker, a teammate, or else a topic with a short name. A subject that names a real project is that project; a name that's never been seen becomes a topic.
- **Facts** are one sentence each, about one subject, with a kind (decision, status, blocker, owner, commitment, open question, preference), the date the source said it, and at least one source.
- **Sources** are links: a message, an issue, a document, a meeting's notes, a session digest. Their kind comes from the link, not from what an agent called it.
- A newer fact **replaces** an older one about the same thing; the older one is kept as history. An older fact can never replace a newer one.
- Facts that other agents add start **unconfirmed** and never replace a confirmed fact until a later source backs them up or the user confirms them.
- Facts are never deleted; a wrong one is rejected, and whatever it replaced becomes current again.

### Learning

On a schedule, and when the user asks, an agent reads what's new since the last run from every connected source and writes fact changes: new facts, replacements and confirmations. Cairn checks and applies them; the agent never writes to the store directly. The first run reads a fixed window back; later runs read only what's new.

### Sources through MCP

Pulse read Linear and Slack through the agent's own MCP servers. Cairn makes that the rule: any MCP server the user connects (a tracker, chat, docs, email, calendar, code host) becomes a source the learning agent reads, with no Cairn code for that service. Cairn only needs native connectors where an agent isn't the right tool:

- **Events for metrics**, such as an issue closed or a pull request merged, need exact counts and times, not an agent's summary.
- **Capture**, such as recording and transcribing meetings, or digesting finished coding-agent sessions.

### Asking

Asking works the way it does in Pulse v4: Cairn picks the facts that match the question, a small model on the user's computer summarizes them when one is available, and when it can't answer, the user's chosen agent answers instead, starting from those facts and looking further in the connected sources. The user sees its steps while it works and an answer with each claim linked to its source.

### The MCP server

Agents read and write the nexus through Cairn's own MCP server: search facts, see where a subject stands, list what changed, add a fact (unconfirmed), and reject one the user says is wrong. Tools can add their own MCP tools, such as tasks or meetings.

## Tools

Tools are built on the nexus and declare what they need. A tool appears only when its needs are met, so someone without a tracker or a code host still gets a useful app.

| Tool | Needs |
|---|---|
| Today's tasks | A tracker, chat or email |
| Briefing | Any source |
| Replies you owe | Chat or email |
| Meetings | A microphone and a calendar |
| Start work on an issue | A tracker and a code folder |
| Dashboards | Any source with events |

## Dashboards: a graph builder

Pulse hard-coded its charts. Cairn lets the user build them, the way Grafana does:

- Native connectors store **events**: a timestamp, a kind, a person and a few fields, such as an issue closed with its project and labels, or a pull request merged with its size.
- A **panel** chooses a dataset, a measure (count, sum, median), a grouping (person, project, label), a time bucket and a chart type.
- **Targets**, such as "40 pull requests this month", are reference lines that any panel can show.
- Panels go on **dashboards**, one or more per project.
- Asking can build a panel too: "issues closed per person per week" becomes a panel the user can then adjust.
- People are counted by each tool's own identity. Cairn never guesses that two accounts are the same person; the user can link them.

## Platform: the open question

Cairn should run on macOS, Windows and Linux, and feel as native as Pulse does. Those pull in different directions.

**What Pulse learned.** Pulse started on Tauri 2 with SvelteKit and was rewritten in Swift at about 730 lines. The menu-bar popover was the reason: in Tauri it needed a borderless window, a positioning plugin, hide-when-focus-is-lost logic, a guard against toggling twice and a raw AppKit call to show over full-screen apps, which never worked reliably. SwiftUI's `MenuBarExtra` did all of it, and it opened over full-screen apps on the first try. gpui was set aside because it would mean drawing every chart and laying out long text with links by hand.

**Options to explore:**

| | Native feel | Cross-platform | Charts and rich text |
|---|---|---|---|
| SwiftUI | Best on macOS | macOS only | Swift Charts; good |
| Tauri 2 | Web view in a native window; tray handling varies by platform | Yes | Any web charting library |
| gpui | Native rendering, fast | macOS and Linux today, Windows in progress | Built by hand |
| A shared core with native shells | Native per platform | Yes, at the cost of one interface per platform | Per platform |

A shared core with native shells could put the nexus, connectors, learning, the MCP server and the event store in one cross-platform service, with a thin native interface on each platform (SwiftUI on macOS). The graph builder could be a web view inside the native shell, since charting libraries are the hard part to rebuild. The next step is a small spike of the menu-bar popover and one chart in Tauri 2 and gpui, measured against Pulse.

## Distribution without a paid developer account

Cairn is free and open source, and will never pay for an Apple developer account. On macOS that means:

- Builds are ad hoc signed. The first launch needs right-click → Open (or clearing the quarantine flag); the install script does this.
- The Keychain ties an item to the exact build of an app signed this way and asks again after every update, so Cairn keeps secrets in files only the user can read, as Pulse v4 does. Pulse tried a self-signed certificate and an "any app" Keychain item first; neither works without an Apple team ID.
- macOS privacy permissions (microphone, screen recording, calendar) may be asked for again after updates for the same reason.
- Updates are signed with Cairn's own Ed25519 key and checked before installing, as in Pulse.

Windows and Linux builds have no equivalent cost.

## Carried over from Pulse

- The facts model, newest-wins, agent write-back with confirmation, and rejecting a fact to bring back what it replaced.
- Learning only from what's new since the last run.
- Session digests: the conversation without tool output, capped in size, only from the project's folders, and never learning personal matters that aren't about the work.
- Asking: local first, then the agent, with steps shown and sources linked.
- The triage list: only agent facts and changed facts, with Looks right and Wrong.
- Meeting capture and on-device transcription.
- Signed self-updates.

## Open questions

- Platform, after the spike.
- Which agent runs learning and asking for someone without a coding agent installed: bundle one, or connect to a hosted model with the user's own key?
- How a non-engineer connects MCP servers: a catalog of known servers with one-click setup, or OAuth for the common ones?
- Where events for metrics come from when a service has no native connector yet.

## Milestones

1. **Core:** projects, the facts store, learning through MCP sources, the MCP server, and asking.
2. **Interface:** the menu-bar app with triage, asking and the briefing.
3. **Tools:** tasks, replies and meetings, each appearing when its needs are met.
4. **Dashboards:** events from native connectors and the graph builder.
