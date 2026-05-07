---
name: meeting-summary-and-act
description: |
  After a meeting ends, produces a complete post-meeting package: a Word document
  summary saved to output/, a draft recap email to all attendees (held for review,
  not auto-sent), and proactive execution of every action item owned by the user.

  Use when user asks to "summarize and act on that meeting", "wrap up the meeting",
  "post-meeting actions", "after my meeting do X", "close out my meeting",
  "recap and action my last meeting", "handle the follow-ups from that call",
  "process my meeting", or asks to do all three (summary + recap + follow-throughs)
  after a meeting concludes.

  Do NOT use for: meeting prep or briefs (use meeting-intel), scheduling new
  meetings (use schedule-meeting), or summarizing without follow-through (use
  meeting-intel). This skill is specifically for the post-meeting close-out
  workflow where the user's owned actions get started, not just listed.
cowork:
  category: productivity
  icon: TaskListLtr
---

# Meeting Summary and Act

Closes out a meeting end-to-end: summarize, recap, and start the user's action items.

## When to Use

Trigger on post-meeting close-out requests like:
- "wrap up the meeting I just had"
- "summarize and act on the standup"
- "post-meeting actions for the budget review"
- "close out my 10am"
- "after my meeting, draft the summary, recap email, and start my action items"

## When NOT to Use

- **Meeting prep / pre-meeting briefs** → use `meeting-intel`
- **Just a summary, no follow-through** → use `meeting-intel`
- **Scheduling a new meeting** → use `schedule-meeting`
- **Drafting comms unrelated to a meeting** → use `stakeholder-comms`

## Workflow

### Step 1 — Resolve the meeting

Identify which meeting the user means.
- If the user names it ("the budget review", "my 10am"), search calendar via `ListCalendarView` for today and yesterday.
- If they say "the meeting I just had" / "my last meeting", pick the most recent past event from `ListCalendarView` ending before now.
- If ambiguous (multiple recent meetings), ask via `AskUserQuestion` with the top 2-3 candidates.

Capture: subject, start/end time, attendees (with emails), organizer, online meeting join URL.

### Step 2 — Pull the transcript

Use `GetMeetingTranscript` with the meeting's join URL. If no transcript exists:
- Check the calendar event body and any chat thread tied to the meeting via `ListChatMessages`.
- Tell the user transcript is unavailable and proceed using whatever notes/chat exist. Do not fabricate content.

### Step 3 — Extract structured content

From the transcript (or available notes), extract:
- **Key discussion points** (3-7 bullets, grounded in actual passages)
- **Decisions made** (with the decider when identifiable)
- **Action items** — for each: owner, task, due date if mentioned. Resolve owner names to email via `SearchPeople` so the user's items can be reliably identified.

Every item must trace to a specific passage. If something is unclear, say so rather than guessing.

### Step 4 — Build the Word summary (parallel with Step 5)

Invoke the `docx` skill to produce a meeting summary at `output/<meeting-subject>-summary.docx` containing:
- Header: meeting title, date/time, attendees
- Key discussion points
- Decisions
- Action items table (owner | task | due date)
- The user's owned actions called out in a separate section

After write, verify with `Glob output/**/*` per the delivery gate.

### Step 5 — Draft the recap email (parallel with Step 4)

Use `CreateDraftMessage` (do NOT send) addressed to all attendees.
- Subject: `Recap: <meeting subject> — <date>`
- Body: short summary + decisions + full action items table
- Attach the Word summary via `AddDraftAttachments` using the `output/` path
- Tell the user the draft is ready in their Drafts folder for review before sending

**Important:** Never auto-send. The user reviews and sends themselves. This is a hard rule.

### Step 6 — Act on the user's owned action items

Filter action items where the owner email matches the user's own email (resolve via `GetMyDetails` if not already known).

For each owned item, **start the work immediately** — don't just list it. Use `TaskCreate` to track each one, then dispatch in parallel where possible:

| Action item type | Start by |
|---|---|
| "Draft a doc / proposal / brief" | Begin a draft in `output/` (use `docx` skill if formal) |
| "Send X to Y" | Create a draft email via `CreateDraftMessage` to that person with a starting body |
| "Schedule a follow-up with Z" | Use `FindMeetingTimes` then propose slots; create draft event if appropriate |
| "Get info from / ask someone" | Draft the message in Outlook or Teams (don't auto-send) |
| "Review / read X" | Pull the document via `SearchM365` and produce a short summary inline |
| "Decide / think about" | Surface the decision with a recommendation and the main tradeoff (don't write a doc) |
| Unclear / needs the user's input | Note it as "needs your input" — don't fabricate content |

**Execution rules:**
- Run independent action items in parallel via `TaskCreate` + subagents where they don't share state.
- Drafts only — never auto-send messages, never send meeting invites without confirmation.
- If an item depends on info the user must provide, surface that gap clearly rather than guessing.
- Proactively gather context (search files, prior emails, related Teams threads) before drafting deliverables.

### Step 7 — Report back

Summarize concisely:
- Word summary saved (link via context alias)
- Recap email drafted (link to draft)
- The user's owned items: for each, what was started and where to find it
- Anything that needs the user's input before it can progress

## Guardrails

- **Never auto-send the recap email or any draft created from action items.** All outgoing messages remain as drafts.
- **Ground every claim in the transcript or retrieved data.** If transcript is missing, say so — don't fabricate content.
- **Resolve owners by email**, not by display name alone — common names cause misattribution.
- **Calendar events for follow-ups stay as proposals** until the user approves the time.
- **For action items requiring judgment the user hasn't given**, surface the question rather than inventing an answer.

## Output Format

End with a brief summary card or a tight bullet list:
- Word summary: `[file alias]`
- Recap email draft: `[draft alias]`
- Started: `<N>` of your action items (linked to drafts/files)
- Needs your input: `<list>` (if any)
