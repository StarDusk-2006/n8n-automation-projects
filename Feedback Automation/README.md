# User Feedback Simple Automation

An n8n workflow that collects customer feedback through a form, classifies its sentiment with an LLM, logs every submission to Google Sheets, and sends the customer a tailored email.

## How it works

1. **Form** (`Feedback :)`): the customer submits name, phone, email, and feedback.
2. **AI Agent** (Groq `openai/gpt-oss-120b`): returns a sentiment score from -100 to +100, a confidence score (0-100), and a short reason.
3. **Code**: parses and validates the AI output, then sets the label from the score.
4. **Merge** (Combine by Position): joins the AI result with the original form data.
5. **Append row in sheet**: logs every submission to the main sheet.
6. **Switch**: routes by label to a matching email.
7. **Email + log**:
   - **Good**: thank-you email.
   - **Bad**: apology email, and the entry is logged to the *Bad Feedbacks* sheet.
   - **Neutral**: acknowledgement email, and the entry is logged to the *Neutral feedbacks* sheet.

```
Form -> AI Agent -> Code -> Merge -> Append row -> Switch -> GOOD
  \______________________/                                 -> BAD     -> Bad Feedbacks
        (form data)                                        -> NEUTRAL -> Neutral Feedbacks
```

## Sentiment thresholds

| Label   | Score range   |
|---------|---------------|
| Good    | +20 to +100   |
| Neutral | -19 to +19    |
| Bad     | -100 to -20   |

The labels are set in the Code node, and the Switch routes on that label. To change the thresholds, edit them only in the Code node (and in the agent's system prompt so the wording stays consistent).

## Setup

1. In n8n, import `User_Feedback_Simple_Automation_fixed.json`.
2. Connect credentials: **Groq API**, **Google Sheets OAuth2**, **Gmail OAuth2**.
3. Prepare three Google Sheets and point the nodes at them:

   | Sheet | Columns |
   |-------|---------|
   | Main log (`Append row in sheet`) | `name`, `phone`, `email`, `feedback`, `sentiment`, `sentiment score`, `confidence`, `reason` |
   | Bad Feedbacks | `Name`, `Phone`, `Email`, `Action/Reason` |
   | Neutral feedbacks | `Name`, `Phone`, `Email`, `Action/Reason` |

   Format the phone columns as plain text so leading zeros are kept.
4. Replace the `© 2026 Your Company` placeholder in the three email nodes.
5. Test with the form's test URL, using your own email address, then activate the workflow.

## Notes

- Emails are sent to the address the customer enters in the form.
- Customer feedback is HTML-escaped before it is placed in the emails.
- The AI agent retries up to 3 times on failure, and the main-sheet append continues on error so emails still go out.
- If the AI returns invalid JSON or an out-of-range score, the Code node throws an error and the run stops.
