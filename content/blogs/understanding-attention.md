---
title: 'Understanding Attention'
date: 2026-09-20
description: 'A concise look at scaled dot-product attention.'
math: true
---

Attention lets each item in a sequence gather information from every other item. In its common scaled dot-product form:

$$
\begin{aligned}
\operatorname{Attention}(Q,K,V)
&=
\operatorname{softmax}
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)V
\end{aligned}
$$

Here, `Q`, `K`, and `V` are the query, key, and value matrices. The scale factor \(\sqrt{d_k}\) keeps the dot products from growing too large as the key dimension increases.

## The calculation

1. Compute similarity scores with \(QK^T\).
2. Scale the scores by \(\sqrt{d_k}\).
3. Normalize each row with `softmax`.
4. Use those weights to combine the values in \(V\).

| Symbol | Meaning | Typical shape |
|:--|:--|:--|
| \(Q\) | Queries | sequence × key dimension |
| \(K\) | Keys | sequence × key dimension |
| \(V\) | Values | sequence × value dimension |

The operation is differentiable and can be evaluated in parallel. That combination is central to the [Transformer architecture](https://arxiv.org/abs/1706.03762).[^paper]

> Attention is a learned, content-dependent way to move information between positions.

## A minimal implementation

```python {linenos=table}
def attention(q, k, v):
    scale = q.shape[-1] ** -0.5
    weights = softmax((q @ k.T) * scale)
    return weights @ v
```

The snippet leaves batching and masking aside so the main operation stays visible.

---

[^paper]: Vaswani et al., “Attention Is All You Need,” 2017.
