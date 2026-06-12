# Baby Tracker

Data lives in `data/records.json` on the `main` branch of `charraisolutions/baby`.

## Logging entries

When the user mentions a feeding or diaper change, immediately write a new record to `data/records.json` using the GitHub MCP tools — no confirmation needed.

### Steps to log
1. Read current file: `mcp__github__get_file_contents` on `data/records.json` (branch: `main`)
2. Decode base64 content, parse JSON array
3. Append new record (format below)
4. Re-encode to base64, write back with `mcp__github__create_or_update_file` using the `sha` from step 1
5. Confirm to user: "Logged: [brief description]"

### Record format

Feeding:
```json
{
  "id": "<timestamp-base36><random>",
  "type": "feeding",
  "feedingType": "breast|formula|solid",
  "amount": "4" ,
  "amountUnit": "oz",
  "duration": null,
  "notes": null,
  "time": "2025-01-15T14:30:00.000Z"
}
```

Diaper:
```json
{
  "id": "<timestamp-base36><random>",
  "type": "diaper",
  "diaperType": "wet|dirty|both",
  "notes": null,
  "time": "2025-01-15T14:30:00.000Z"
}
```

- `id`: use `Date.now().toString(36)` + 4 random alphanumeric chars
- `time`: ISO 8601 UTC. If user says "just now" use current time. If user gives a clock time (e.g. "3:20pm") assume today's date in their local time — ask for timezone once if unknown, then remember it in this session.
- `amount`: string number of oz, or null. If user says "ml" store as-is and set `amountUnit: "ml"`.
- Optional fields (`amount`, `duration`, `notes`) should be `null` when not provided, not omitted.

## Natural language hints

| User says | Interpretation |
|---|---|
| "4oz formula" / "4 oz bottle" | feeding, formula, amount: "4" |
| "nursed for 15 min" / "breastfed" | feeding, breast, duration: "15" |
| "wet diaper" / "peed" | diaper, wet |
| "poopy diaper" / "dirty" | diaper, dirty |
| "both" / "wet and dirty" | diaper, both |
| "solids" / "ate some food" | feeding, solid |
| time like "at 2" / "2pm" | use today's date, that time |
| "just now" / no time given | use current UTC time |

## Querying data

When user asks questions like "how many feedings today?" or "what was the last diaper?":
1. Read `data/records.json` via GitHub MCP
2. Filter/aggregate as needed
3. Answer concisely — no need to show raw JSON unless asked

## Commit messages

Use format: `Log [type]: [brief]` — e.g. `Log feeding: 4oz formula 2:30pm`
