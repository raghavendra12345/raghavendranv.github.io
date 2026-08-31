---
title: "Diffie–Hellman and What It Actually Assumes"
date: 2026-08-24
draft: false
math: true
tags: ["basic-cryptography", "discrete-log", "key-exchange"]
description: "The protocol is four lines. The assumption underneath it is the interesting part, and it is strictly stronger than 'discrete log is hard'."
ShowToc: true
TocOpen: false
---

Everyone learns Diffie–Hellman as a party trick with paint colours. The trick is
fine. What it hides is that the security of the protocol does not rest on the
discrete logarithm problem, and the gap between the two matters in practice.

## The protocol

Fix a cyclic group $G$ of prime order $q$ with generator $g$. Alice draws
$a \gets \Z_q$ and sends $g^a$; Bob draws $b \gets \Z_q$ and sends $g^b$. Each
computes

$$
K = (g^b)^a = (g^a)^b = g^{ab}.
$$

That is the whole protocol. An eavesdropper sees $(g, g^a, g^b)$ and wants
$g^{ab}$.

## Three assumptions, not one

It is tempting to say "this is secure because discrete log is hard." That is the
weakest of the three relevant statements, and it is not the one we need.

{{< box "Definition 1 — Discrete Log (DL)" >}}
Given $(g, g^a)$ for $a \gets \Z_q$, output $a$. The problem is hard if for every
efficient $\mathcal{A}$,
$$\Pr\!\left[\mathcal{A}(g, g^a) = a\right] \le \negl(\lambda).$$
{{< /box >}}

{{< box "Definition 2 — Computational Diffie–Hellman (CDH)" >}}
Given $(g, g^a, g^b)$, output $g^{ab}$. Hard if
$$\Pr\!\left[\mathcal{A}(g, g^a, g^b) = g^{ab}\right] \le \negl(\lambda).$$
{{< /box >}}

{{< box "Definition 3 — Decisional Diffie–Hellman (DDH)" >}}
Distinguish $(g, g^a, g^b, g^{ab})$ from $(g, g^a, g^b, g^c)$ for
$a, b, c \gets \Z_q$. Hard if
$$
\begin{aligned}
\Adv^{\mathsf{ddh}}_{G}(\mathcal{A}) = \Bigl|\;
&\Pr\!\left[\mathcal{A}(g, g^a, g^b, g^{ab}) = 1\right] \\
&\;-\; \Pr\!\left[\mathcal{A}(g, g^a, g^b, g^{c}) = 1\right]
\;\Bigr| \;\le\; \negl(\lambda).
\end{aligned}
$$
{{< /box >}}

The implications run one way only:

$$
\text{DDH hard} \;\Longrightarrow\; \text{CDH hard} \;\Longrightarrow\; \text{DL hard}.
$$

Each arrow is easy — solve the harder-to-break problem and you break the easier
one. Neither converse is known to hold in general, and the first one provably
fails in some groups. That failure is the practical point of this post.

## Where DDH breaks and CDH survives

Take an elliptic curve group equipped with an efficiently computable bilinear
pairing $e : G \times G \to G_T$ satisfying

$$
e(g^x, g^y) = e(g, g)^{xy}.
$$

Now DDH is trivial. Given a candidate tuple $(g, g^a, g^b, g^c)$, just check
whether

$$
e(g^a,\, g^b) \overset{?}{=} e(g,\, g^c),
$$

which holds exactly when $c \equiv ab \pmod q$. No hardness assumption survives
that. CDH, meanwhile, is still believed hard in these groups — the pairing lets
you *verify* a Diffie–Hellman tuple without letting you *produce* one.

Groups like this have a name: **gap Diffie–Hellman** groups, where the decisional
problem is easy and the computational one is hard. Far from being a defect, this
gap is exactly what BLS signatures are built on.

## So which one does the protocol need?

If you want the shared secret $g^{ab}$ to be unguessable, CDH suffices. If you
want to feed $K$ straight into a stream cipher and claim the keystream is
indistinguishable from random, you need DDH — because you are asserting that
$g^{ab}$ looks like a uniform group element, which is precisely the decisional
statement.

Real protocols dodge the question. Rather than using $g^{ab}$ as a key directly,
they hash it:

$$
K = \mathsf{KDF}(g^{ab}).
$$

In the random oracle model this drops the requirement back to CDH, since an
adversary learns nothing about $\mathsf{KDF}(g^{ab})$ without querying the oracle
on $g^{ab}$ itself. This is one of the honest, load-bearing uses of the random
oracle model, and it is why every deployed key exchange runs its shared secret
through a KDF.

## Concrete parameters

Index calculus attacks the multiplicative group of $\F_p$ in subexponential time
$L_p[1/3, 1.923]$, while the best generic attack on a well-chosen elliptic curve
group of order $q$ is Pollard rho at $O(\sqrt{q})$. Hence the size gap:

| Security level | Finite field $\F_p^\times$ | Elliptic curve |
|---|---|---|
| 112-bit | 2048-bit $p$ | 224-bit $q$ |
| 128-bit | 3072-bit $p$ | 256-bit $q$ |
| 192-bit | 7680-bit $p$ | 384-bit $q$ |
| 256-bit | 15360-bit $p$ | 512-bit $q$ |

A sanity check on the elliptic curve column — Pollard rho on a 256-bit group
needs roughly $\sqrt{2^{256}} = 2^{128}$ operations:

```python
from math import log2

def rho_cost(bits: int) -> float:
    """log2 of expected Pollard rho operations for a group of given bit size."""
    return bits / 2

for b in (224, 256, 384, 512):
    print(f"{b}-bit group -> 2^{rho_cost(b):.0f} operations")
```

## What to take away

Writing "secure under the Diffie–Hellman assumption" is not a complete sentence.
There are at least three assumptions wearing that name, they are not equivalent,
and in pairing-friendly groups the strongest of them is simply false. Say which
one you mean.

---

*Next: why the small-subgroup check you skipped is not optional.*
