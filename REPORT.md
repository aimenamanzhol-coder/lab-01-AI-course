# Assignment 1 — Lab 01: The price of one request

**Student:** Аймен Аманжол
**Date:** 2026-09-20
**Measurement basis:** Part 2 reference measurement supplied with the lab
repository (`measurements.example.json`, 2026-09-12, `claude-opus-5`) was used
because the instructor API key is not available outside the classroom. The
reference run reports request input tokens EN/RU/KK = 145/209/317 and measured
output tokens = 955/1226/1337.

## 1. Part 1 prediction vs. measured value

For the COMPLAINT item, I based my prediction on UTF-8 byte size from Part 1.
Because Cyrillic characters take 2 bytes each in UTF-8 and Latin characters
take 1, I expected Russian and Kazakh to cost roughly twice the English text.

| Ratio | Prediction | Measured token ratio |
|---|---|---|
| RU / EN | 2.27× | 134 / 92 = 1.46× |
| KK / EN | 2.44× | 198 / 92 = 2.15× |

The byte-based prediction worked reasonably for Kazakh but overestimated
Russian, showing that byte length is not a reliable substitute for tokenizer
behavior. The tokenizer splits by its merge table, not by bytes: Russian has
many frequent letter combinations already merged into single tokens, while
Kazakh keeps the Kazakh-specific letters (ә, ғ, қ, ң, ө, ұ, ү, һ, і) that the
vocabulary handles less efficiently.

## 2. Annual cost at 2,000 requests/day

I use 2,000 requests/day because it is a realistic pilot volume for a regional
bank's support queue — enough that cross-language cost differences become
operationally meaningful, without assuming full national scale.

| Model | EN / year | RU / year | KK / year |
|---|---|---|---|
| Haiku 4.5 | $4,015 | $5,238 | $6,037 |
| Sonnet 5 | $8,030 | $10,476 | $12,074 |
| Opus 5 | $20,075 | $26,189 | $30,186 |
| Fable 5.1 | $40,150 | $52,378 | $60,371 |

The same Kazakh request uses 2.19× as many input tokens as English
(317 vs. 145), while the total Opus 5 bill is 1.38× English because output
length also affects the bill. These two ratios are not the same number: the
first is a property of the tokenizer, the second is what you actually pay.

## 3. Production model for a Kazakh-language support queue

I would deploy Claude Sonnet 5 as the default. At the measured request and
answer lengths, the Kazakh queue costs about $12,074/year, which is 60% lower
than Opus 5 ($30,186/year) at the same 2,000 requests/day, and 2× more than
Haiku 4.5 ($6,037/year). Before production, I would gate it with the lab's
factuality checks:

1. declines to explain the rate change instead of fabricating a cause;
2. invents no number, rate, account number or date not present in the complaint;
3. answers entirely in Kazakh;
4. names a concrete next step.

Haiku 4.5 is cheaper, but the corpus trap (the complaint claims attachments
that are not actually provided, and the system prompt forbids inventing
anything) makes fabrication a compliance risk on a bank queue, not just a
style issue. Sonnet 5 holds the instruction more reliably than Haiku at this
task, at a cost well below Opus.

## 4. Cost-reduction lever not used in the lab

Use prompt caching for the repeated system prompt: the system prompt is 124
of 317 tokens in the Kazakh request (39% of input) and is resent on every
call, and `prices.py` defines `CACHE_READ_FRACTION = 0.10` for opus-5 /
sonnet-5 / haiku-4.5, but `cost_usd` never applies it. Caveats: the 50%
batch discount does not apply to a live support queue because batch is
asynchronous; the model above ignores the one-time cache-write cost, which
real caching is not free of; and a 124-token system prompt is likely below
the minimum cacheable prompt length, so the saving may not materialize on
current models.

---

Sources: Lab 01 repository and its supplied reference measurement; Anthropic
Claude model overview and pricing documentation. Costs follow the lab's Part 3
method and list-price snapshot (checked 2026-09-12).
