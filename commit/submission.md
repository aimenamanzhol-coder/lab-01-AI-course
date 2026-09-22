# Lab 01 — The Price of One Request

## 1. Prediction vs Measured

My prediction (based on bytes from Part 1):
- RU/EN ≈ 1.92×
- KK/EN ≈ 2.13×

Measured (from API run):
- RU/EN = 1.84× (input), 1.84× (output)
- KK/EN = 2.48× (input), 2.60× (output)

Conclusion: bytes and tokens correlate but are not identical. Kazakh-specific letters (ә, ғ, қ, ң, ө, ұ, ү, һ, і) are penalized more by the tokenizer than by byte count.

## 2. Annual Cost at 5,000 requests/day

Justification: 5,000/day is a reasonable volume for a mid-size bank support queue.

| Language | Input | Output | Annual cost |
|---|---:|---:|---:|
| EN | 60 | 628 | $4,380.00 |
| RU | 79 | 1155 | $8,012.66 |
| KK | 149 | 1635 | $11,393.47 |
| **Total** | | | **$23,786.14** |

## 3. Model for Kazakh Support

I would recommend a model with strong instruction-following and lower output pricing. In the reference run, the model fabricated reasons for the rate change (administrative error, tiered rates) despite the system prompt saying to answer only from provided documents — this is a quality failure. For production, quality (compliance with system prompt) matters as much as cost.

## 4. Cost Lever Not Used

Prompt caching: the system prompt is ~38–40% of input tokens and repeats in every call; cache reads cost 10× less than full input.
