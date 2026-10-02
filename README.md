# SnackCheck 🥗

**Is this snack healthy?** A little automation that takes a barcode and tells you (in a friendly, sometimes funny way) whether a packaged food is a smart everyday choice or more of an occasional treat.


## What it does

You send a barcode, it:
1. Checks the barcode is actually valid (present + numeric)
2. Looks the product up on Open Food Facts (a free, open food database)
3. Figures out if the sugar/salt/fat are "high" using traffic-light rules
4. Compares that against the product's official Nutri-Score
5. Takes whichever reading is *worse* — so a soda with sneaky-okay numbers per 100ml still gets flagged if its Nutri-Score says otherwise
6. Asks an AI to write a short, human verdict in the right tone (encouraging if it's fine, gently honest if it's not)
7. Sends back a nicely formatted message

**Who's this for?** Mostly just me proving I can build this, but the idea (from the project brief) is a shopper scanning something at the store and wanting a quick, no-judgment answer.

## How to use it

Send a `POST` request to the webhook with a barcode:

```json
{ "barcode": "3017620422003" }
```

You'll get back a plain-text message like:

```
🥗 SnackCheck Verdict

Product: Nutella by Ferrero
🔴 Sugar: 56.3g (high)
🔴 Salt: 0.107g (low) — wait this one's actually fine
🔴 Fat: 30.9g (high)
Nutri-Score: e

Okay, Nutella — we've all been there. This one's basically dessert wearing
a breakfast costume...
```


### Some barcodes to try
- `3017620422003` — Nutella (unhealthy, Nutri-Score e)
- `5449000000996` — Coca-Cola (this is the important test case — its sugar alone doesn't look "high" per 100ml, but the Nutri-Score correctly flags it as unhealthy anyway. This is the whole point of the "worse of the two readings" rule.)
- `0000000000000` — not a real product, tests the "not found" error
- `"barcode": "abc"` — tests the "not a number" error
- *(no barcode at all)* — you get a little self-explaining message instead of a scary error.

## Features

- Barcode validation (missing / non-numeric)
- Real nutrition data from Open Food Facts (no API key needed!)
- Traffic-light classification (sugar/salt/fat) based on thresholds from the project brief
- Combines that with the official Nutri-Score grade, always going with whichever verdict is worse
- A "concern score" from 0-3 (how many nutrients came out high)
- AI-generated verdict message, tone changes depending on if it's healthy/moderate/unhealthy
- Handles missing data gracefully (doesn't just guess "healthy" if the fields are empty)
- Returns proper JSON with real HTTP status codes for errors (400 for bad input, 404 for product not found), and plain text for the actual verdict

## Technical details

**APIs used:**
- [Open Food Facts](https://world.openfoodfacts.org/) — free, no signup, keyless. This is where all the real nutrition data comes from.
- Groq (running an LLM) — this is the one thing that needs an API key, for generating the friendly verdict text.

**The logic, roughly:**
```
Webhook → validate barcode exists → validate it's numeric
  → call Open Food Facts
  → check if product was actually found
  → check if it has enough nutrient data to classify
  → run the traffic-light + Nutri-Score classification (this is a Code node)
  → send the numbers to the AI for a friendly writeup
  → format everything into one message
  → send it back
```

Every error case (no barcode, invalid barcode, not found, not enough data) has its own path that skips straight to a formatted error response instead of crashing.

**AI integration:** one prompt, sent to Groq, that gets told the verdict level (healthy/moderate/unhealthy) and adjusts its own tone based on that — instead of me writing three separate prompts for three separate tones. I think this is actually a cleaner way to do it than branching into 3 different AI nodes, but I'm not 100% sure that's what was expected from the "conditional routing" wording in the brief, so noting that here just in case.

## Error handling

| Situation | What happens |
|---|---|
| No barcode sent | Friendly 200 response explaining how to use the service |
| Barcode isn't a number | 400 error with a hint and an example |
| Product not found in database | 404 error, doesn't crash |
| Product found but missing nutrition data | 200 response saying there's not enough info to give a verdict (this was specifically important — the brief said a product should never come out "healthy" just because its data is blank) |

## Limitations (aka things I know aren't perfect)

- Only checks sugar, salt, and fat — doesn't look at fiber, protein, or anything else that Nutri-Score actually factors in behind the scenes
- If Open Food Facts is slow or down, this whole thing just... waits. No timeout/retry logic yet
- The AI sometimes gets a little too jokey depending on the product — I tried to rein this in with the prompt but it's not 100% consistent every time
- Only handles one barcode per request, no batch lookups
- I built this and tested it myself, not with a real group of users, so I'm sure there are edge cases (weird barcodes, products with partial data) I haven't hit yet

