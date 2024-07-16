---
title: "An Introduction to Zero-Knowledge Proofs - Part 1"
date: 2024-07-16 00:00:00 -0300
categories: [crypto, zk]
tags: [crypto, zk]
math: true
mermaid: true
# image: //
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

_This is the part 1 of a study on ZKPs. Part 2 will contain the implementation of these protocols using Circom._
# What are Zero-Knowledge Proofs?
ZK Proofs are exactly what the name implies: proofs of something that do not demonstrate any additional "knowledge" to the verifier.

### An example
A very simple example can be seen in the cave game:

![Alibaba_cave](/assets/img/Alibaba_cave.png)

In this example, Peggy and Victor know that there is a _magic door_ in the middle of the circular cave that needs a password to be opened. What happens here is that if Peggy chooses path A and exits through the entrance of path B, she would have to pass through this door. This way, it proves to Victor that Peggy knows the password, but he gains no information about it or anything else during the interaction.

### Definition

A ZKP is a protocol between ($P$, $V$) (with $P$ being the prover and $V$ the verifier) that is a _[PPT (probabilistic polynomial time)](https://en.wikipedia.org/wiki/PP_(complexity))_ algorithm.\
A common public input is $x$ (where $P$ wants to prove that $x$ is indeed a member of a language $L$).\
A private input is $W$, the witness.

## Properties

### Completeness

If $x \in L$, $P$ has a **correct** witness $W$ and $(P, V)$ are honest, $V$ will accept the protocol.

### Soundness

If $x \notin L$, $\forall$ algorithm $P*$, $V$ will reject the protocol with a probability $P$ (soundness parameter) set. Ideally, we want $P = 1 - negl(n)$. See more at [3](https://www.youtube.com/watch?v=l5A3oEG-XKk)

### Zero-Knowledge Intuition
1. The PPT algorithm $V*$ learns **nothing** from the protocol.
2. Everything that $ V* $ learns, $ V* $ could have computed alone.
3. $\exists$ a PPT algorithm $S$ that $V*$ could use to generate the same knowledge ($\tau$). Basically, $S$ is a simulator.
4. $\exists$ a public PPT algorithm $S$ that generates a sequence of $\tau_{sim}$ such that:
{$\tau_{sim}$} is [computationally indistinguishable](https://en.wikipedia.org/wiki/Computational_indistinguishability) from {$\tau_{real}$}

# Example protocol

Now that we have a slightly more formal definition, we can look at a protocol that uses zero-knowledge proofs.

## Graph Isomorphism

Let $G_1, G_2$ be two presumptively isomorphic graphs. We want to prove that we know an isomorphism between these two graphs without showing this isomorphism.
We will define our public parameters as $X$, which is a pair of graphs $({G_0}$ and ${G_1})$. Our private parameter (our witness ${W}$) is the permutation $\Pi$ that leads to the isomorphism between $({G_0}$ and ${G_1})$. So that $G_1 = \Pi(G_0)$ and $G_0 = \Pi^{-1}(G_1)$. We will call the prover $P$ and the verifier $V$.

### How the protocol begins:

* $P$ randomly chooses a value of $b$ from {$0, 1$}
* $P$ computes a random permutation $\sigma$ that can be applied to either graph.
* $P$ calculates $G = \sigma(G_0)$ if $b=0$ and calculates $G = \sigma(G_1)$ if $b=1$.
* $P$ sends $G$ to $V$
* $V$ randomly chooses a value of $b'$ from ${0, 1}$ and sends it to $P$
    * If $b = b'$, $P$ sends $\phi$ = $\sigma$ to $V$
    * If $b=1, b'=0$, $P$ sends $\phi$ = $\sigma \circ \Pi$
    * If $b=0, b'=1$, $P$ sends $\phi$ = $\sigma \circ \Pi^{-1}$
* $V$ receives $\phi$ and performs the verification:
    * $G$ $\stackrel{?}{=}$ $\phi(G_{b'})$
    * If the verification holds, $V$ accepts. Otherwise, it rejects.

### Completeness

To verify completeness, we can see that if $b \neq b'$, then $\phi(G_{b'})$=$\sigma(G_b)$

If $b = b'$, although we do not verify the isomorphism, we verify that $P$ is not being dishonest by choosing values of $G_0$ or $G_1$ when sending $G$ to $V$.

### Soundness

If $b \neq b'$ and $b'=0$, then:\
$G \stackrel{?}{=} \phi(G_0)$\
$G \stackrel{?}{=} \sigma \circ \Pi(G_0)$\
$G \stackrel{?}{=} \sigma(G_1)$\
Which concludes as true.

Now for $b'=1$:\
$G \stackrel{?}{=} \phi(G_1)$\
$G \stackrel{?}{=} \sigma \circ \Pi^{-1}(G_1)$\
$G \stackrel{?}{=} \sigma(G_0)$\
Which also holds true.

Now for $b = b'$:\
If $b = b'$, $P*$ will **fail** with a probability of $1/2$. By running the protocol $n$ times, the probability of ending up in this case $n$ times is practically zero.\
More formally we have:\
Run the protocol $n$ times. Accept the proof only if all verifications are accepted. The soundness parameter will be: $P = 1 - 1/2^n$

It is important to choose a new value of $\sigma$ for each interaction of the protocol. If we choose the same value, $V*$ can ask for the isomorphism between $G$ and $G_0$ and then ask for the isomorphism between $G$ and $G_1$. From there, the verifier would have the isomorphism between $G_0$ and $G_1$ (yikes!).

## Schnorr Identification Protocol

Now that we have seen a bit of how ZK-Proofs work, we can take a brief look at another example that is also zero-knowledge.

### How it works

Let $E$ be an elliptic curve with generator point $G$ and of order $q$. Suppose a private key $x$ and a public key $X$, with $X = xG$. We want to prove that we know the private key without saying anything more about it. That is, using zero-knowledge.

We start the protocol with the prover $(P)$ generating a randomly chosen number $r$ from $\mathbb{Z}_{q}$.\
After that, $P$ calculates $R = rG$ and sends it to the verifier $(V)$\
$V$ receives the value of $R$ and calculates a value of $c$ randomly chosen from a subset of challenges called $Challenge\ space$\
$V$ sends $c$ to $P$.\
$P$ calculates $e = r + cx (mod\ q)$ and sends this to $V$\
$V$ multiplies $e$ by $G$ and verifies if this equals $R + cX$:\
$eG \stackrel{?}{=} R + cx$\
If confirmed, $V$ accepts the protocol and trusts $P$

## Conclusion

We will discuss the Schnorr protocol in more detail in part 2, where we will use Circom to implement this and the graph isomorphism protocol. Until then~ ^^

# Referências
[1] [Cave example](https://www.youtube.com/watch?v=MwgpuY5X9Iw)\
[2] [PPT algorithms](https://en.wikipedia.org/wiki/PP_(complexity))\
[3] [Negligible functions](https://www.youtube.com/watch?v=l5A3oEG-XKk)\
[4] [Computational indistinguishability](https://en.wikipedia.org/wiki/Computational_indistinguishability)
[5] Neal Koblitz. A Course in Number Theory and Cryptography\
[6] [Dan Boneh, Victor Shoup. A Graduate Course in Applied Cryptography](https://crypto.stanford.edu/~dabo/cryptobook/BonehShoup_0_4.pdf)\
[7] [Yehuda Lindell. How To Simulate It – A Tutorial on the Simulation Proof Technique](https://eprint.iacr.org/2016/046.pdf)\
[8] [Lecture notes 18 from CS6180: Theory of computing - Cornell University](https://www.cs.cornell.edu/courses/cs6810/2009sp/scribe/lecture18.pdf)\
[9] [Nguyen Thoi Minh Quan. Intuitive Advanced Cryptography](https://github.com/cryptosubtlety/intuitive-advanced-cryptography/blob/master/advancedcrypto.pdf)\
[10] [Vipul Goyal. Lecture 17: Zero-Knowledge Proofs I](https://www.youtube.com/watch?v=VnZVW1iG2po&list=PLI3cKEs5b6gvelkJnHf16r3ADhYvcQjdr&index=17)
