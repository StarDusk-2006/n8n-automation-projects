# Google Reviews Monitor: AI Review Triage (n8n)

A scheduled workflow that pulls the latest Google Maps reviews for a business, has an AI classify each one, logs everything to a Google Sheet, and emails an alert when a review is high priority.

> Demo: `[add 30-second video or GIF here]`
> Workflow canvas: `[add screenshot here]`

## The problem

Business owners rarely read every review. A bad review about hygiene or rudeness can sit unanswered for days, which hurts the business, while positive reviews go unthanked.

## What it does

1. Runs on a schedule and scrapes recent reviews for a Google Maps place (Apify actor).
2. An AI chain classifies each review: sentiment (positive, negative, neutral), a suggested action (thank the customer, investigate a complaint, improve service, respond to feedback) and a priority (high, medium, low).
3. Appends each review with its classification to a Google Sheet.
4. Emails an alert for **high-priority** reviews only (1-2 stars, or serious complaints like hygiene, safety, or rudeness).

## How it works

```
Schedule ─> Apify Google Reviews scraper ─> select fields
        ─> AI chain (OpenRouter) ─> parse JSON ─> append to Google Sheet
        ─> High priority? ─ yes ─> Email alert
```

The parsing step extracts the JSON object from the AI reply. If the reply is malformed, it falls back to rules based on the star rating (1-2 high, 3 medium, 4-5 low), so one bad AI response doesn't break the run.

## Stack

n8n · Apify (Google Reviews Scraper) · OpenRouter (gpt-oss-120b) · Google Sheets · Gmail

## Setup

1. Import `google-reviews-monitor.json` into n8n.
2. Create credentials: Apify, OpenRouter, Google Sheets, Gmail.
3. In the Apify node, replace the example Google Maps URL with your business's place URL.
4. Create a sheet with these columns: `placename`, `placeaddress`, `reviewname`, `rating`, `reviewtext`, `reviewrating`, `action`, `priority`, and point the Sheets node at it.
5. Set the alert email in the Email alert node and the interval in the Schedule Trigger.

## Limitations

- Fetches 5 reviews per run, so it can append duplicates if the same reviews appear in consecutive runs. A duplicate check (match on reviewer and text) would fix this.
- Handles one location. Multiple locations would need a loop over place URLs.

## Next steps

- De-duplicate reviews before appending
- Draft a suggested reply for each negative review
- Weekly summary of review trends
