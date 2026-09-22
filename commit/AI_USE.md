# AI Use Declaration — Lab 01

## Tools used

- **DeepSeek** (chat.deepseek.com) — used to explain tokenizer concepts (BPE merge tables, difference between o200k_base and cl100k_base), and to check my arithmetic on the annual cost table.
- **ChatGPT** (chat.openai.com) — used to help structure the one-page submission and to verify the RU/EN and KK/EN ratio calculations.

## What I did myself

- Read all lab instructions and the reference `measurements.json`.
- Chose the volume assumption (5,000 requests/day) and justified it for a mid-size bank support queue.
- Compared the measured token ratios against my Part 1 prediction and interpreted the difference.
- Wrote the quality argument in §3 based on the system prompt's "answer only from provided documents" rule.
- Checked every number in the tables against the source file.

## What I did NOT use AI for

- No AI tool was used to fabricate measurements or invent data.
- All token counts come directly from the reference measurement run supplied with the lab.
- No AI tool was used to write code in this submission (the lab's scripts were provided).

## Data disclosure

I did not have access to an Anthropic API key (it is shown only on the classroom projector). The token counts in §1–§2 are from the reference run distributed with the lab (Gemini 3.6 Flash). This is disclosed for transparency.
