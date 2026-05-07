# Meeting Summary and Act — Copilot Skill

A Copilot skill that closes out meetings end-to-end: it produces a Word summary, drafts a recap email to attendees, and proactively starts any action items assigned to you (drafts only, never auto-sends).

## What it does

After a meeting ends, this skill:

1. **Resolves the meeting** — figures out which meeting you mean (by name, "my last meeting", "my 10am", etc.).
2. **Pulls the transcript** — fetches the Teams transcript if available, falls back to chat/notes if not.
3. **Extracts structured content** — key discussion points, decisions, and action items with owners.
4. **Builds a Word summary** — saved to `output/<meeting-subject>-summary.docx`.
5. **Drafts a recap email** — addressed to all attendees, with the Word summary attached, held in your Drafts for review.
6. **Acts on your action items** — starts every item assigned to you (drafts emails, begins documents, finds meeting times, summarizes docs to read, etc.).
7. **Reports back** — a concise summary of what was produced and what needs your input.

## Installation

1. Open Copilot Cowork and attach the `SKILL.md` file from this folder.
2. Say: *"Add this as a personal skill"* or *"Create a new personal skill from the attached file."*
3. Copilot saves it to your personal skills folder

## Personalizing it for you

The skill is written generically — it uses "the user" everywhere and resolves to whoever is signed in via `GetMyDetails`, so it should work out of the box.

If you'd like to tailor it (your name in outputs, default email signature, preferred file naming, specific projects/people to prioritize), say:

> *"Edit my meeting-summary-and-act skill to personalize it for me — replace 'the user' references with my name where natural, and add any defaults that make sense for how I work."*

## How to use it

After a meeting, try any of these:

- *"Wrap up the meeting I just had"*
- *"Summarize and act on my last meeting"*
- *"Post-meeting actions for the budget review"*
- *"Close out my 10am"*
- *"Recap and action my last call"*

## What it won't do

- Pre-meeting prep or briefs → use `meeting-intel`
- Just a summary with no follow-through → use `meeting-intel`
- Scheduling new meetings → use `schedule-meeting`
- Auto-send any email or meeting invite — **everything stays as a draft for your review**

## Guardrails

- Never auto-sends the recap email or any draft created from action items.
- Grounds every claim in the transcript or retrieved data — no fabricated content.
- Resolves owners by email, not display name, to avoid misattribution.
- Surfaces gaps clearly when an action item needs your judgment.
