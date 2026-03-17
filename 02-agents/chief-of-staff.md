# Agent: chief-of-staff

**Category:** Productivity  
**Model Tier:** `opus` (complex reasoning across multi-channel context)  
**Source:** `agents/chief-of-staff.md`

---

## Purpose

The chief-of-staff agent is a **personal communication chief of staff** that triages email, Slack, LINE, and Messenger messages. It classifies messages into 4 tiers, generates draft replies matching the user's tone, enforces post-send follow-through, and manages scheduling.

---

## When to Use

Invoke this agent:
- When managing high volumes of incoming messages across multiple channels
- When prioritizing communication response queue
- When drafting replies in a consistent tone
- When tracking follow-up obligations after sending
- When calculating scheduling availability

---

## Responsibilities

1. **Multi-channel triage** — Process email, Slack, LINE, Messenger in parallel
2. **4-tier classification** — Categorize each message by required action
3. **Draft generation** — Write replies matching user's tone and signature
4. **Follow-through enforcement** — Calendar, todo, relationship notes after sends
5. **Availability calculation** — Schedule from calendar data
6. **Stale response detection** — Flag overdue pending responses

---

## Inputs

- Raw messages from any supported channel
- User's communication history for tone matching
- Calendar data for availability calculation

## Outputs

- Classified message queue with tier labels
- Draft replies for action-required messages
- Calendar entries for meeting commitments
- Follow-up task list

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read message threads and history |
| `Grep` | Search prior communications for context |
| `Glob` | Find message export files |
| `Bash` | Process message data files |
| `Edit` | Update follow-up logs |
| `Write` | Create draft response files |

---

## 4-Tier Classification System

| Tier | Label | Description | Action |
|---|---|---|---|
| 0 | `skip` | Automated, spam, irrelevant | No reply needed |
| 1 | `info_only` | FYI, announcements, newsletters | Read and file |
| 2 | `meeting_info` | Meeting invites, schedule changes | Calendar update only |
| 3 | `action_required` | Questions, requests, decisions | Draft reply required |

---

## Triage Workflow

```
Incoming Messages (all channels)
    ↓
Classify each message (4-tier system)
    ↓
Filter: action_required + meeting_info
    ↓
Generate draft replies (action_required)
    ↓
Add calendar entries (meeting_info)
    ↓
Create follow-up tasks (for post-send tracking)
    ↓
Output: prioritized queue + draft responses
```

---

## Tone Matching

The agent analyzes user's existing sent messages to match:
- **Formality level** — formal/professional vs casual
- **Greeting style** — "Hi X," vs "Hey X," vs "X,"
- **Sign-off pattern** — "Best," vs "Thanks," vs "Regards,"
- **Sentence length** — concise vs detailed
- **Response latency norms** — same-day vs 48-hour expectation

---

## Post-Send Follow-Through

After a reply is sent, the agent creates:
1. **Calendar entry** — If meeting was proposed or accepted
2. **Todo item** — If commitment was made ("I'll send the report by Friday")
3. **Relationship note** — If personal context was shared

---

## Platform Adaptation

This agent is unique in that it operates **outside the coding workflow** — it is a personal productivity tool that happens to be deployed in a coding harness.

### GitHub Copilot
Not directly applicable. Consider separate dedicated communication management tools.

### CLI
Works well in CLI context — feed message exports as files, get structured output.

### Standalone Usage
Can be deployed as a standalone AI assistant with API integration to email/Slack/messaging platforms.

---

## Dependencies

- Standalone — does not typically interact with other coding agents
- May coordinate with: **planner** (to convert action items into implementation plans)
