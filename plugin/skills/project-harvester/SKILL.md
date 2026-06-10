---
name: project-harvester
description: "Harvest external signals (Slack, Gmail, GDocs, scsiwyg, Jira, Confluence, Linear) relevant to a specific project and write them as classified intel docs into `project-state/documents/inbox/`. Jira, Confluence, and Linear are pulled through their Claude MCP connectors (Atlassian + Linear). Reads the project manifest to discover which channels, contacts, keywords, projects/boards, spaces, and surfaces to watch. Tracks per-surface cursors in `project-state/state.json`. Designed to be called by `project-orchestrator` as part of the daily routine. Trigger: `/project-harvester` or invoked by project-orchestrator."
---

# project-harvester

Pull external intelligence relevant to a project and deposit it into `project-state/documents/inbox/` for the `project-document-curator` to classify, link to milestones/decisions, and promote.

---

## Philosophy

The harvesters (work-harvester-slack, gmail, gdocs, scsiwyg) ingest *everything* into `~/work-state/` as canonical event envelopes. `project-harvester` is a **focused lens** that runs *after* or *independently* of those harvesters and asks: "of all the signals in these surfaces, which ones are relevant to *this project right now*?"

Relevance is determined from the project manifest — no hardcoded rules. Every project configures its own surfaces, contacts, channels, and keyword terms.

---

## What gets harvested

### Slack
- All messages in `surfaces.slack.channel` (and any additional channels in `surfaces.slack.extra_channels[]`) since the cursor.
- DMs from any person in the project's `consortium.*.contacts[]` list.
- Threads where a project keyword appears in `#general` or other monitored channels.

### Gmail
- All threads involving any `consortium.*.contacts[].email` address (sent or received).
- Threads where subject or body contains any `surfaces.gmail.keywords[]` (if set).
- Drafts under `surfaces.gmail.from_identity` that reference project keywords (if `drafts_only: true`, skip inbound filtering).

### Google Docs
- Docs in `surfaces.gdocs.gdocs_root` folder that were modified since the cursor.
- Also: docs shared *with* or *by* known consortium contacts.

### scsiwyg
- Posts on `surfaces.scsiwyg.site_slug` published or updated since the cursor.
- Posts on any *other* sites (e.g., Rafal's site) that contain project keywords in title or body — this catches consortium partner writing that references the project.

### Jira *(via the Atlassian MCP connector)*
- Issues in `surfaces.jira.projects[]` (project keys, e.g. `LEDGER`) created or **updated** since the cursor — title, status, assignee, latest comments.
- Issues matching `surfaces.jira.jql` (an explicit JQL filter) if set — overrides the project-key scan for power users.
- Issues mentioning a project keyword in summary/description, and issues assigned to / commented on by anyone in the contact roster.

### Confluence *(via the Atlassian MCP connector)*
- Pages in `surfaces.confluence.spaces[]` (space keys) created or updated since the cursor.
- Pages matching `surfaces.confluence.cql` (explicit CQL) if set.
- Pages whose title or body contains a project keyword (catches partner documentation that references the project).

### Linear *(via the Linear MCP connector)*
- Issues in `surfaces.linear.teams[]` (team keys) or `surfaces.linear.projects[]` updated since the cursor — title, state, assignee, latest comments.
- Issues matching `surfaces.linear.query` (free-text/filter) if set.
- Issues mentioning a project keyword, or assigned to / commented on by a contact-roster member (matched by email where Linear exposes it).

---

## Manifest keys consumed

```yaml
# project-state/manifest.yaml

surfaces:
  slack:
    enabled: true
    channel: "#ledger-rt"               # primary project channel
    extra_channels: []                  # additional channels to watch
    workspace: ~                        # Slack workspace name (optional; MCP default if null)
  gmail:
    enabled: true
    from_identity: "david@atomic47.co"  # the project owner's send-as address
    drafts_only: false                  # if true, skip inbound filtering (pre-award mode)
    keywords: []                        # optional subject/body keywords to match inbound mail
  gdocs:
    enabled: true
    gdocs_root: ~                       # Google Drive folder ID; null = skip folder scan but still scan by contact
  scsiwyg:
    enabled: true
    site_slug: ~                        # slug of the project's own blog; null = skip own-site scan
    watch_sites: []                     # other site slugs to watch for keyword mentions
  jira:                                 # via the Atlassian MCP connector
    enabled: false
    projects: []                        # Jira project keys to watch, e.g. ["LEDGER","PLAT"]
    jql: ~                              # optional explicit JQL; overrides the project-key scan
    site: ~                             # Atlassian site/cloud id if the connector serves several
  confluence:                           # via the Atlassian MCP connector
    enabled: false
    spaces: []                          # Confluence space keys to watch, e.g. ["LEDGER","ENG"]
    cql: ~                              # optional explicit CQL
    site: ~                             # Atlassian site/cloud id if the connector serves several
  linear:                               # via the Linear MCP connector
    enabled: false
    teams: []                           # Linear team keys to watch, e.g. ["ENG","PLAT"]
    projects: []                        # optional Linear project ids/names to scope to
    query: ~                            # optional free-text/filter query

# Consortium contacts are the other key input:
consortium:
  members:
    - contacts:
        - email: "r.rohozinski@secdev.com"
          name: "Rafal Rohozinski"
  lead_applicant:
    contact:
      email: "ishtiaque.ahmed@utoronto.ca"
      name: "Syed Ishtiaque Ahmed"
```

---

## Cursor management

Cursors are stored in `project-state/state.json`:

```json
{
  "harvest_cursors": {
    "slack":      "2026-05-04T00:00:00Z",
    "gmail":      "2026-05-04T00:00:00Z",
    "gdocs":      "2026-05-04T00:00:00Z",
    "scsiwyg":    "2026-05-04T00:00:00Z",
    "jira":       "2026-05-04T00:00:00Z",
    "confluence": "2026-05-04T00:00:00Z",
    "linear":     "2026-05-04T00:00:00Z"
  }
}
```

Default cursor when missing: 7 days ago. Cursor is only advanced after a surface is fully harvested without errors.

---

## Output — inbox documents

Each harvested item becomes a markdown file in `project-state/documents/inbox/`:

```
YYYY-MM-DD-{surface}-{slug}.md
```

### File format

```markdown
---
source: slack                          # slack | gmail | gdocs | scsiwyg | jira | confluence | linear
source_id: "C123/1714389612.123456"   # channel/ts, thread_id, doc_id, post_id, issue_key, page_id
harvested_at: "2026-05-04T12:00:00Z"
surface_timestamp: "2026-05-04T09:30:00Z"
author: "Rafal Rohozinski"
author_contact: "r.rohozinski@secdev.com"
channel: "#ledger-rt"                  # slack only
subject: ~                             # gmail only
doc_title: ~                           # gdocs only
post_title: ~                          # scsiwyg only
issue_key: ~                           # jira/linear only (e.g. LEDGER-142)
issue_url: ~                           # jira/linear only — link back to the issue
issue_status: ~                        # jira/linear only (e.g. "In Progress")
page_id: ~                             # confluence only
page_url: ~                            # confluence only
space: ~                               # confluence only (space key)
relevance_signals:                     # why this was flagged
  - contact_match: "r.rohozinski@secdev.com"
  - channel_match: "#ledger-rt"
status: inbox                          # always "inbox" on write; curator promotes
---

# {title or first line or subject}

{full text or rich excerpt}

---
_Harvested by project-harvester from {surface} on {date}._
```

The `status: inbox` field is what `project-document-curator` looks for when it scans the inbox. The `relevance_signals` array tells the curator why this doc landed here, which helps it decide how to classify it.

---

## How a harvest run works

### Step 1 — Load manifest and cursors

Read `project-state/manifest.yaml`:
- `surfaces.*` config
- Build the **contact roster**: all emails from `consortium.*.contacts[].email` + `consortium.lead_applicant.contact.email`
- Build the **keyword list**: project `id`, project `name`, any explicit `surfaces.*.keywords[]`

Read `project-state/state.json` → `harvest_cursors`. Default missing cursors to 7 days ago.

### Step 2 — Slack harvest (if `surfaces.slack.enabled`)

```
channels_to_watch = [manifest.surfaces.slack.channel] + manifest.surfaces.slack.extra_channels
cursor_ts = harvest_cursors["slack"]

for each channel in channels_to_watch:
  mcp__42fdfc76__slack_read_channel(channel, oldest=cursor_ts)
  → messages since cursor

  for each message:
    emit inbox doc if:
      - any message (it's a project channel, all messages are relevant)
      - OR author is in contact_roster
      - OR text contains a project keyword
```

Also search DMs from known contacts:
```
mcp__42fdfc76__slack_search_public_and_private(
  query="{contact_name} OR {contact_email}",
  after=cursor_ts
)
→ filter to DMs involving self and the contact
```

### Step 3 — Gmail harvest (if `surfaces.gmail.enabled`)

Build search queries:
```
contact_query = "from:({emails joined by OR}) OR to:({emails joined by OR})"
keyword_query = manifest.surfaces.gmail.keywords joined by OR (if any)
combined = f"({contact_query}) after:{cursor_date}"
if keywords:
  combined += f" OR ({keyword_query} after:{cursor_date})"

mcp__81e68767__search_threads(query=combined, max_results=50)
for each thread:
  mcp__81e68767__get_thread(thread_id)
  → extract messages, build inbox doc per thread
```

One inbox doc per Gmail thread (not per message) — threads are the natural unit of conversation.

### Step 4 — GDocs harvest (if `surfaces.gdocs.enabled`)

```
if gdocs_root is set:
  mcp__aab407bf__search_files(
    query="modifiedTime > '{cursor_date}' and '{gdocs_root}' in parents"
  )
  → for each file: mcp__aab407bf__read_file_content(file_id)
  → emit inbox doc

# Also scan for docs shared by/with known contacts:
mcp__aab407bf__list_recent_files(count=20)
→ filter by: sharedWithMe == true AND (lastModifyingUser in contact_roster)
→ for each: read and emit
```

### Step 5 — scsiwyg harvest (if `surfaces.scsiwyg.enabled`)

```
if site_slug is set:
  mcp__scsiwyg__list_posts(site_slug, includeUnpublished=false)
  → filter to posts updatedAt > cursor
  → mcp__scsiwyg__get_post(post_id)
  → emit inbox doc

for each watch_site in surfaces.scsiwyg.watch_sites:
  mcp__scsiwyg__list_posts(watch_site)
  → filter to posts where title or body contains project keyword
  → emit inbox doc
```

> **Connectors note (Jira / Confluence / Linear).** These surfaces are served by
> their **Claude MCP connectors** — the Atlassian connector (Jira + Confluence) and
> the Linear connector — connected in the operator's Claude account / CLI, not by a
> project-state-owned server. The exact MCP tool names vary by which connector build
> is installed, so **discover the available `mcp__*` tools at runtime** and use the
> ones that match the operations below. If the connector for a surface isn't
> connected, skip that surface and log it (same as any other surface). All access is
> read-only.

### Step 5b — Jira harvest (if `surfaces.jira.enabled`, Atlassian MCP connected)

```
# Prefer an explicit JQL; otherwise build one from the project keys + cursor.
jql = surfaces.jira.jql or
      "project in (" + join(surfaces.jira.projects) + ") AND updated >= '{cursor_date}'"

<atlassian-mcp search-issues tool>(jql, limit=50)        # e.g. searchJiraIssuesUsingJql / jira_search
for each issue:
  <atlassian-mcp get-issue tool>(issue.key)              # full fields + comments
  emit inbox doc if:
    - the issue is in a watched project (all such issues are relevant), OR
    - summary/description/comment contains a project keyword, OR
    - assignee/reporter/commenter email ∈ contact_roster
  → frontmatter: source=jira, source_id=issue.key, issue_key, issue_url, issue_status,
    author=assignee||reporter, surface_timestamp=issue.updated
```

### Step 5c — Confluence harvest (if `surfaces.confluence.enabled`, Atlassian MCP connected)

```
cql = surfaces.confluence.cql or
      "space in (" + join(surfaces.confluence.spaces) + ") AND lastmodified >= '{cursor_date}'"

<atlassian-mcp search-pages tool>(cql, limit=50)          # e.g. searchConfluenceUsingCql / confluence_search
for each page:
  <atlassian-mcp get-page tool>(page.id, body=true)
  emit inbox doc if: in a watched space, OR title/body contains a project keyword
  → frontmatter: source=confluence, source_id=page.id, page_id, page_url, space,
    doc_title=page.title, surface_timestamp=page.lastModified
```

### Step 5d — Linear harvest (if `surfaces.linear.enabled`, Linear MCP connected)

```
# List issues for watched teams/projects updated since the cursor (or run the query).
<linear-mcp list-issues tool>(
  team=surfaces.linear.teams, project=surfaces.linear.projects,
  query=surfaces.linear.query, updatedAfter=cursor_ts, limit=50
)
for each issue:
  <linear-mcp get-issue / list-comments tool>(issue.id)
  emit inbox doc if:
    - the issue is in a watched team/project, OR
    - title/description/comment contains a project keyword, OR
    - assignee/creator/commenter ∈ contact_roster (by email where exposed)
  → frontmatter: source=linear, source_id=issue.identifier, issue_key=issue.identifier,
    issue_url=issue.url, issue_status=issue.state, author=assignee, surface_timestamp=issue.updatedAt
```

### Step 6 — Write inbox docs

For each item flagged for ingest:
1. Build the filename: `{YYYY-MM-DD}-{surface}-{slug}.md` where slug = sanitized title or channel+ts
2. Check if file already exists (dedup by source_id hash) — skip if so
3. Write the markdown file to `project-state/documents/inbox/`
4. Append a one-line entry to `project-state/harvest/harvest.log`:
   ```
   2026-05-04T12:00:00Z  slack   #ledger-rt/1714389612.123456  → 2026-05-04-slack-ledger-rt-abc123.md
   ```

### Step 7 — Advance cursors

For each surface that completed without error:
```json
state.harvest_cursors["slack"] = max_message_timestamp_seen
state.harvest_cursors["gmail"] = max_thread_date_seen
state.harvest_cursors["gdocs"] = now
state.harvest_cursors["scsiwyg"] = now
state.harvest_cursors["jira"] = max_issue_updated_seen
state.harvest_cursors["confluence"] = max_page_modified_seen
state.harvest_cursors["linear"] = max_issue_updated_seen
```

Write back to `project-state/state.json`.

### Step 8 — Report

Return a summary:
```
## Harvest complete — 2026-05-04

| Surface    | Items found | Written | Skipped (dup) | Errors |
|------------|-------------|---------|---------------|--------|
| Slack      | 14          | 12      | 2             | 0      |
| Gmail      | 3           | 3       | 0             | 0      |
| GDocs      | 1           | 1       | 0             | 0      |
| scsiwyg    | 0           | 0       | 0             | 0      |
| Jira       | 6           | 6       | 0             | 0      |
| Confluence | 2           | 2       | 0             | 0      |
| Linear     | 4           | 4       | 0             | 0      |

12 new docs in project-state/documents/inbox/ — run /project-document-curator to classify.
```

---

## CLI invocation

```bash
/project-harvester                          # all surfaces, use cursors
/project-harvester --since 7d              # override cursor with lookback
/project-harvester --surface slack         # single surface only
/project-harvester --surface gmail,gdocs   # comma-separated surfaces
/project-harvester --surface jira,linear   # connector surfaces (jira | confluence | linear)
/project-harvester --dry-run               # show what would be written, don't write
/project-harvester --no-advance-cursor     # harvest but don't move cursors (re-run safe)
```

---

## Deduplication

Dedup key: `{surface}:{source_id}`. Stored in `project-state/harvest/seen.json` as a set of hashed keys. On each write attempt, hash the key and check the set. If present, skip. After writing, add to set.

`seen.json` is append-only — never prune. It stays small (one 12-byte hash per harvested item).

---

## Integration with project-orchestrator

`project-orchestrator` calls this skill as the **first step** of its daily routine (before checking milestones, deadlines, etc.) so that the inbox is populated before curator recommendations are made.

```yaml
# Orchestrator daily routine order:
1. project-harvester              ← pull fresh intel
2. project-document-curator       ← classify inbox docs
3. project-milestone-manager      ← check at-risk milestones
4. project-phase-gate             ← check gate items
5. project-status-reporter        ← draft weekly if due
```

---

## Error handling

| Error                              | Behavior                                      |
|------------------------------------|-----------------------------------------------|
| MCP not connected for a surface    | Skip that surface; log; continue with others  |
| Rate limit on a surface            | Pause 5s, retry once; then skip + log         |
| Malformed message/doc              | Skip item; log; continue                      |
| Disk write failure                 | Halt; do NOT advance cursor; report error     |
| `project-state/` not found        | Fail fast — wrong working directory           |

---

## What this skill does NOT do

- Does not classify or promote docs — that's `project-document-curator`
- Does not send or modify anything on any surface — read-only
- Does not harvest GitHub (commits are already in `work-state`; surface them via the kanban's milestone linking instead)
- Does not run sentiment analysis — that's downstream
- Does not replace the work-state harvesters — it's a project-scoped lens on the same surfaces
