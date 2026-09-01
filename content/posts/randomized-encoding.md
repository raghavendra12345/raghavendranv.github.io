---
title: "Randomized Encoding"
date: 2026-09-01T23:14:07+05:30
draft: false
math: true
tags: ["basic-mpc"]
description: "What is Randomized Encoding."
---

Suppose Raghavendra wants Pratik to compute a function $f(x)$ on his secret input $x$. The obvious naive method could be to send $x$ to Pratik who can then compute $f(x)$. What Raghavendra really wants is something slightly strange: Pratik should learn the result of the computation without necessarily learning the input to the computation.

Can we somehow transform $x$ into another object that contains enough information to recover $f(x)$, but not enough information to recover $x$? This is the idea behind a randomized encoding.

What if Raghavendra could instead send a randomized encoding of $x$ that Pratik can evaluate, while learning essentially nothing about $x$ beyond the answer $f(x)$? That is the basic idea behind a \textbf{randomized encoding}.

## Encoding a Function

Let

$$
f : \mathcal{X} \to \mathcal{Y}
$$

be the function Raghavendra wants to compute.

Instead of sending $x$ directly, Raghavendra samples some fresh randomness $r$ and computes

$$
\widehat{x} \gets \mathsf{RE}(f,x;r),
$$

where $\mathsf{RE}$ is a randomized encoding algorithm.

He sends only $\widehat{x}$ to Pratik.

The encoding is constructed so that there is a decoder $\mathsf{Dec}$ satisfying

$$
\mathsf{Dec}(\widehat{x}) = f(x).
$$

So the picture is

$$
x
\quad\xrightarrow[\text{randomness }r]{\mathsf{RE}(f,\cdot)}
\quad
\widehat{x}
\quad\xrightarrow{\mathsf{Dec}}\quad
f(x).
$$

Raghavendra knows $x$.

Pratik receives only $\widehat{x}$.

Pratik can recover the answer $f(x)$, but the encoding is supposed to conceal the input $x$. But what does "conceal" actually mean?

That is where the interesting part begins.

First, the encoding must preserve the computation.


{{< box "Correctness" >}}

A randomized encoding is correct if
$$
\Pr\left[\mathsf{Dec}(\mathsf{RE}(f,x;r)) = f(x)\right] \geq 1-\negl(\lambda).
$$
{{< /box >}}

But correctness alone says nothing about privacy.

For example, Raghavendra could simply send

$$
\widehat{x}=x.
$$

Pratik could then compute $f(x)$ perfectly. But he also learns everything about $x$. So we need another property that is privacy.

## Privacy

The encoding should reveal no useful information about $x$ beyond what is already revealed by $f(x)$.

This is captured using a simulator. Imagine a simulator that does not know $x$. It only knows the output $f(x)$.

If the simulator can produce something that looks indistinguishable from a real randomized encoding, then the encoding cannot be revealing useful additional information about $x$.

The idea is:

$$
\boxed{
\mathsf{RE}(f,x)
\quad\approx_c\quad
\mathsf{Sim}(f(x))
}
$$

The left-hand side is the real encoding. The right-hand side is something generated using only the output.

{{< box "Privacy" >}}

A randomized encoding is private if there exists an efficient simulator $\mathsf{Sim}$ such that

$$
\mathsf{RE}(f,x)
\approx_c
\mathsf{Sim}(f(x)),
$$

where $\approx_c$ denotes computational indistinguishability.

The simulator receives $f(x)$ but does not receive $x$.

{{< /box >}}

This is the familiar simulation paradigm from cryptography. Instead of proving directly that the adversary cannot learn something from the encoding, we show that whatever the adversary sees could have been generated using only the information it is supposed to learn.

What does the encoding actually hide? It is important to phrase the privacy guarantee carefully. A randomized encoding does not necessarily hide $x$ completely. It cannot hide information that $f(x)$ itself reveals. This also gives an equivalent way to view the guarantee.

Suppose two inputs $x$ and $x'$ produce the same output:

$$
f(x)=f(x').
$$

Then both real encodings must be indistinguishable from the same simulated distribution. Consequently,

$$
\mathsf{RE}(f,x)
\approx_c
\mathsf{RE}(f,x').
$$

In other words, from the encoding alone, an efficient observer should not be able to tell apart two inputs that the function itself treats identically.

We can now put the pieces together.

{{< box "Definition — Randomized Encoding" >}}

Let

$$
f:\mathcal{X}\to\mathcal{Y}
$$

be a function.

A randomized encoding of $f$ consists of a randomized encoding algorithm $\mathsf{RE}$ and a decoding algorithm $\mathsf{Dec}$.

For an input $x$, the encoder produces

$$
\widehat{x}
\gets
\mathsf{RE}(f,x;r).
$$

It satisfies:

Correctness.
$$
f(x)
\geq
1-\negl(\lambda).
$$

Privacy.

There exists an efficient simulator $\mathsf{Sim}$ such that

$$
\mathsf{RE}(f,x)
\approx_c
\mathsf{Sim}(f(x)).
$$

{{< /box >}}

The two properties capture the entire philosophy:

$$
\boxed{
\text{Correctness: recover }f(x)
}
$$

and

$$
\boxed{
\text{Privacy: reveal nothing beyond }f(x).
}
$$

The first says that the encoding preserves the computation. The second says that the encoding does not expose the input unnecessarily.

## Multiple Parties with Inputs

From one secret input to many. So far, there is only one secret input: $x$.

Now imagine that the input itself is distributed among several parties. Suppose Raghavendra, Pratik, and Sita hold

$$
x_1,\qquad x_2,\qquad x_3
$$

respectively.

They want to compute

$$
f(x_1,x_2,x_3)
$$

without simply revealing their individual inputs to one another. This is the setting of secure multiparty computation (MPC). At a high level, the goal is now:

$$
\boxed{
\text{private inputs}
\quad\longrightarrow\quad
\text{compute }f
\quad\longrightarrow\quad
\text{learn the permitted output}
}
$$

Randomized encodings provide one useful way of thinking about how a computation can be represented while hiding private inputs that the computation does not need to reveal.

But moving from

$$
\mathsf{RE}(f,x)
$$

to a setting where

$$
x=(x_1,\ldots,x_n)
$$

is distributed across different parties is where the real cryptographic questions begin. Who constructs the encoding? What does each party send? Who evaluates it? What does each party learn? How many rounds of communication are necessary? And what happens if one of the parties is malicious?

Those are questions for another post.

## Summary

The main idea to take away is simple:

A randomized encoding lets you reveal the result of a computation without revealing the input itself. After this post, the key picture to remember is

$$
x
\quad\longrightarrow\quad
\widehat{f(x)}
\quad\longrightarrow\quad
f(x),
$$

where the encoding reveals no useful information about $x$ beyond what is already revealed by $f(x)$.

---



