# Jira AI Test Case Agent (n8n + Gemini)

A working n8n workflow that fetches a Jira ticket and generates structured
test cases using Google's Gemini API — built to speed up manual test case
authoring for QA workflows.

## How it works

```
Manual Trigger -> Set Ticket ID -> Get Jira Ticket -> Extract Ticket Text
    -> Generate Test Cases (Gemini) -> Parse Response -> Post Comment to Jira
```

1. **Manual Trigger** — run the workflow on demand
2. **Set Ticket ID** — the Jira ticket to process (e.g. `SCRUM-6`)
3. **Get Jira Ticket** — n8n's Jira node fetches summary + description
4. **Extract Ticket Text** — converts Jira's raw description format
   (Atlassian Document Format) into plain text, and sanitizes it
   (escapes newlines/quotes) so it's safe to embed in a JSON request body
5. **Generate Test Cases (Gemini)** — HTTP Request node calls Google's
   Gemini API with a QA-focused prompt
6. **Parse Response** — extracts the generated text from Gemini's response
7. **Post Comment to Jira** — writes the test cases back onto the ticket

## Setup

1. **Import the workflow**
   - Open n8n, click **Import from File**, select `n8n-workflow.json`

2. **Add your Jira credentials**
   - Credentials → New → "Jira Software Cloud API"
   - Domain: `https://yourname.atlassian.net`, Email, API Token
     (generate at [id.atlassian.com](https://id.atlassian.com/manage-profile/security) → API tokens)

3. **Add your Gemini API credential**
   - Get a free key at [aistudio.google.com](https://aistudio.google.com) → Get API key
   - In n8n: Credentials → New → "Header Auth"
   - Name: `x-goog-api-key`, Value: your Gemini key

4. **Set a real ticket ID** in the "Set Ticket ID" node

5. **Run it** — Execute Workflow, then check the ticket in Jira for the new comment

## Notes on debugging this build

A few real issues came up building this, worth documenting since they're
common n8n/API gotchas:

- **Raw newlines break JSON bodies.** Ticket descriptions often contain
  actual line breaks. Dropping that text directly into a JSON string
  causes `Bad control character` errors — fixed by sanitizing the text
  (escaping `\n`, `"`, `\\`) in the "Extract Ticket Text" node before it
  reaches the request body.
- **Model names change.** `gemini-1.5-flash` is deprecated/shut down as
  of mid-2026 — use a currently supported model name (check
  [ai.google.dev](https://ai.google.dev) for the latest).
- **Response shapes differ between providers.** Claude and Gemini return
  generated text at different JSON paths (`response.content[0].text` vs
  `response.candidates[0].content.parts[0].text`) — the parser node has
  to match whichever API you're actually calling.

## Tech stack

- n8n (workflow automation)
- Jira REST API (Jira Software Cloud)
- Google Gemini API (free tier)

## Why I built this

As a QA engineer, writing test cases manually from requirements is repetitive
and time-consuming. This agent automates the first draft — pulling ticket
context directly from Jira and generating a solid starting set of test cases,
so testers can focus on refining edge cases and exploratory testing instead
of writing boilerplate from scratch.

## Notes

- Tested against a personal/dummy Jira instance, not production data.
- API keys are never committed — see `.gitignore`.
