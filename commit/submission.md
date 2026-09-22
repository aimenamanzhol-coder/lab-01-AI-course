# Lab 01 — The Price of One Request

**Course:** LLMs, Agentic AI and Reinforcement Learning · Narxoz University  
**Week 1**

---

## 1. Prediction and Measured Value

Before running the measurement, I predicted the RU/EN and KK/EN token ratios using the byte sizes from Part 1:

- English complaint: ~300 bytes
- Russian complaint: ~576 bytes → predicted RU/EN ≈ **1.92×**
- Kazakh complaint: ~640 bytes → predicted KK/EN ≈ **2.13×**

Measured values (from the tokenizer run on the same corpus):

| Ratio | Predicted | Measured |
|-------|-----------|----------|
| RU / EN (input) | 1.92× | **1.32×** |
| KK / EN (input) | 2.13× | **2.48×** |
| RU / EN (output) | — | **1.84×** |
| KK / EN (output) | — | **2.60×** |

**Observation.** For Russian, my byte-based prediction was in the right neighbourhood but still off. For Kazakh, the measured ratio was noticeably higher than predicted. The reason is that the tokenizer's merge table is trained mostly on English and Russian text; Kazakh-specific letters (ә, ғ, қ, ң, ө, ұ, ү, һ, і) are split into more sub-word units, so they cost more tokens than their byte count suggests. Bytes and tokens are correlated but not equivalent — the tokenizer is the reason the three numbers differ.

---

## 2. Annual Cost at 5,000 Requests per Day

**Volume justification:** 5,000 requests per day is a reasonable figure for a mid-size bank's customer-support queue across three language groups (EN, RU, KK) — roughly one request every 17 seconds during business hours.

Pricing used (Gemini 3.6 Flash reference, valid through 2026):  
Input = $0.75 / 1M tokens · Output = $3.75 / 1M tokens

| Language | Input tokens | Output tokens | Cost / request | Annual cost (5,000/day) |
|----------|-------------:|--------------:|---------------:|------------------------:|
| English  | 60  | 628  | $0.000040 | **$4,380.00** |
| Russian  | 79  | 1155 | $0.000073 | **$8,012.66** |
| Kazakh   | 149 | 1635 | $0.000104 | **$11,393.47** |
| **Total (all three queues)** | | | | **$23,786.14** |

**Kazakh is the most expensive queue** — not because the input is longer, but because the model writes a longer answer in Kazakh (1,635 output tokens vs 628 in English). Output is priced at 5× input on every model, so answer length dominates the bill.

---

## 3. Model Choice for a Kazakh-Language Support Queue

I would put a **fast, low-cost model with strong instruction-following** (e.g. a Haiku-class model) into production for the Kazakh queue — with a mandatory quality gate before rollout.

**Cost argument.** Kazakh output tokens are 2.6× the English count, and output is billed at 5× input. The only way to keep the Kazakh queue affordable is to choose a model whose output price per token is low, and to shorten the answer (see §4).

**Quality argument.** In the reference run, the model **fabricated plausible reasons** for the rate change (administrative error, tiered balance, bonus-rate expiry) even though the system prompt says to answer only from provided documents and the complaint explicitly claims attachments that were never provided. This is a **compliance failure**, not a language failure: a fluent Kazakh answer that invents a cause for the rate change breaks the lab's own system prompt. Before production, I would run a pass/fail checklist on at least 30 Kazakh answers covering: (1) declines to explain the rate change when documents are missing; (2) invents no number not present in the complaint; (3) answers entirely in Kazakh; (4) names a concrete next step.

A cheap model that fails the checklist is more expensive than a slightly pricier model that passes it, because every fabricated answer becomes a support ticket.

---

## 4. Cost Lever Not Used in This Lab

**Prompt caching of the system prompt.** The system prompt is ~38–40 % of every request's input tokens (58/145 EN, 80/209 RU, 124/317 KK in the reference run) and it is identical on every call; caching it after the first call and reading it at the cache rate (≈10 % of the input price) would cut the largest single, repeated input cost without touching answer quality.

---

## AI-Use Declaration

**Tools used:** DeepSeek (chat interface) and ChatGPT were used to explain the tokenizer concepts (BPE merge tables, o200k_base vs cl100k_base), to help structure this one-page submission, and to check my arithmetic on the annual-cost table.

**What I did myself:** I read the lab instructions, chose the volume assumption (5,000/day), interpreted the measured RU/EN and KK/EN ratios against my Part 1 prediction, and wrote the quality argument in §3 based on the system prompt's "answer only from provided documents" rule.

**Data note:** The token counts in §1–§2 come from a reference measurement run on the lab corpus (Gemini 3.6 Flash). I did not have access to an Anthropic API key, so the numbers shown are the reference run, not my own API call. This is disclosed here for transparency.

**Not used:** No AI tool was used to fabricate measurements. All numbers in the tables are taken directly from the reference `measurements.json` supplied with the lab.
