# Meeting Invites to Departments: AI Meeting Notifier (n8n)

Type a meeting announcement in plain text and the workflow works out which departments are involved, looks up their email addresses in a Google Sheet, emails each one, and logs a summary as a Todoist task.

> Demo: `[add 30-second video or GIF here]`
> Workflow canvas: `[add screenshot here]`

## The problem

Telling the right departments about a meeting means finding who is attending, looking up each team's contact, and writing the same email over and over. It's repetitive and easy to get wrong.

## What it does

1. Emails you a prompt and waits for a free-text reply, like "Meeting today at 7:00 PM with Sales, Marketing and HR".
2. An AI chain extracts the meeting time, date, title, message and the list of teams as structured JSON.
3. An AI agent looks up each team's email address in a Google Sheet (`Department | Email`). Departments not found are skipped, and email addresses are never invented.
4. The agent sends one email per matched department.
5. A plain-text summary of who was emailed and who was skipped is saved as a Todoist task.

## How it works

```
Manual trigger ─> Gmail "send and wait" (you reply with the meeting details)
              ─> AI extraction chain ─> parse JSON
              ─> AI agent ─┬─ tool: Google Sheet lookup (department -> email)
                           └─ tool: Gmail send (recipient chosen from the lookup)
              ─> Create Todoist task (summary)
```

The parsing step extracts the JSON object from the AI reply and throws a clear error if it can't, so failures are easy to debug.

## Stack

n8n · Groq (free tier, gpt-oss-120b) · Google Sheets · Gmail · Todoist

## Setup

1. Import `meeting-invites.json` into n8n.
2. Create credentials: Groq, Google Sheets, Gmail, Todoist.
3. Create a sheet with two columns, `Department` and `Email`, and point the lookup tool at it.
4. Set your own email in the "send and wait" node and your Todoist project in the last node.
5. Run the workflow, reply to the email with the meeting details, and check the results.

## Limitations

- Starts from a manual trigger and an email reply, so it isn't fully hands-off. A Telegram or form trigger would be quicker.
- Department names must match the sheet closely. Spelling variations may be skipped.
- The agent decides when to send, so a first run should be tested with your own address in the sheet.

## Next steps

- Add a form or Telegram trigger
- Include calendar invites (Google Calendar)
- Log sent emails to a sheet
