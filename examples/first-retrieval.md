# First Retrieval — Fictional Example

> **EXAMPLE — fictional.** This file demonstrates the interaction shape only.
> It does **not** contain a real Pally response.
> Names, stages, and reminder times below are invented for illustration.

## Scenario (fictional)

You previously told Pally over iMessage (fictional):

> "Track Bluebird Robotics for me. I'm interested in their field-robotics internship. Remind me to follow up next week."

Days later, from inside Pi (your development environment), you want that context back without switching apps.

## Prompt you send via Pi (read-only intent)

```text
What is the current status of my Bluebird Robotics opportunity, and when is my next follow-up reminder? Read-only — do not send, schedule, or outreach.
```

In Pi's codemode this becomes:

```js
await tools.mcp__pally__message_pally({
  text: "What is the current status of my Bluebird Robotics opportunity, and when is my next follow-up reminder? Read-only — do not send, schedule, or outreach."
})
```

If the reply reports `status: "working"`, poll once with:

```js
await tools.mcp__pally__check_pally({ turn_id: "<turn_id from message_pally>" })
// or, for the newest turn:
// await tools.mcp__pally__check_pally({})
```

Do **not** use `start_call` for this starter.

## EXAMPLE response shape (fictional — not an actual Pally response)

```text
[EXAMPLE — fictional illustration only, not an actual Pally response]

Your Bluebird Robotics outreach is in stage "researching".

- You saved the Field Robotics Internship posting on Oct 20 via iMessage.
- No outreach sent yet.
- Open item: draft a 3-line intro referencing their orchard trial video.
- Next follow-up reminder: fictional — Monday Nov 3 at 09:00 (your local time).
```

## What a real verified run looks like (sanitized summary)

A tested read-only retrieval in this starter's development (Pi 1.3.0) returned:

- Tool invoked: `mcp__pally__message_pally`
- Result: `status: "complete"` (so `check_pally` was not needed)
- Reported wall time: ~20.7s (single sample, not a benchmark)
- Content type: outreach stage + channel summary + proof-of-work items + follow-up reminder time, assembled from previously stored cross-client (iMessage-originated) context
- Private names, email bodies, and account details are omitted by design

## Safety rules for this example

1. Keep prompts read-only: include “do not send / schedule / outreach”.
2. Never paste tokens, codes, or session identifiers into prompts or docs.
3. If Pally ever reports `working`, use `check_pally` — do not re-send the same `message_pally` (avoids duplicate work).
4. This starter performs no writes and makes no calls.
