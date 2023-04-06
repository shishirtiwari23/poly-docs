---
id: commitment-scheme
title: zkProver Commitment Scheme Setting
sidebar_label: Commitment Scheme
description: Understanding the zkProver's Commitment Scheme Setting
keywords:
  - docs
  - zk rollups
  - polygon
  - wiki
  - zkEVM
  - Polygon zkEVM
image: https://wiki.polygon.technology/img/thumbnail/polygon-zkevm.png
---

The framework for the proof/verification system of our mFibonacci SM is that of a **Polynomial Commitment Scheme**. The mechanism for proving correctness of the computations carried out by the mFibonacci SM (or, any state machine in the zkEVM setting), is best described in terms of an interactive zero-knowledge proof system. One therefore thinks of the proof/verification system as enabled by an interaction of two parties, traditionally called the **Verifier** and the **Prover**.

In practice though, the so-called **Fiat-Shamir transformation** is used to turn such interactive schemes into non-interactive ones.

:::caution

This document dives deep into the technical details of commitment schemes underpinning the zkProver.

:::

It describes what a commitment scheme is, the necessary properties such a scheme must possess, and how the previously stated polynomial identities look like in reality.

## Commitment Scheme Protocol

In the case of our mFibonacci state machine, the Prover needs to commit to the polynomials $P(X)$, $Q(X)$ $P(X\omega)$ and $Q(X\omega)$, and the Verifier will request the Prover to evaluate these polynomials at randomly selected points (i.e., field elements).

The general protocol, in an interactive setting, is as follows;

1. The Prover commits to a polynomial or a number of polynomials, using a specified polynomial commitment scheme.
2. The Verifier randomly selects one or more field elements, which she sends *one at a time* to the Prover as **challenges**. That is, the Verifier requests the Prover to provide evaluations (called **openings**) of the committed polynomials at these *randomly selected* field elements.
3. This *back and forth* interaction can occur as many times as the number of openings the Verifier deems sufficient to guarantee soundness.  
4. The Verifier uses relevant polynomial constraints to test veracity of the Prover's openings.

If all the relevant constraints hold true, then the Verifier accepts that the Prover has knowledge of the correct polynomials $P(X)$, $Q(X)$, $P(X\omega)$ and $Q(X\omega)$.

## Properties of Commitment Schemes

For all practical purposes, such the constructed proof/verification system needs to be secure. That is, it must have several cryptographic properties. The most crucial properties are; being *hiding* and *binding*, as well as *soundness* and *completeness*.

- **Binding** means users are able to commit to values but once committed, it should be impossible for users to change or repudiate their committed values. The committed values are called **commitments**.

- **Hiding** literally means users can commit to values without revealing the actual values, and it should be infeasible for anyone else to deduce the actual values.

- **Openness** means, given a challenge $\alpha$ , the prover can generate a verifiable proof such that $P(\alpha) = y$.

- **Soundness** has to do with whether it is infeasible for the adversarial prover to convince the verifier to accept invalid proofs (or commitments). A proof system is **sound** if the probability for the verifier to accept a false proof is less than a third, $\Big( \text{probability} < \dfrac{1}{3} \Big)$. In other words, the soundness property of a proof system requires that any proof created from a false **witness** should not be convincing to the verifier.

- A proof system has **completeness** if every valid proof is convincing and acceptable to a verifier.

Proof systems based on testing polynomial identities take advantage of a basic property of polynomials expressed by the [**Schwartz-Zippel Lemma**](https://courses.cs.washington.edu/courses/cse521/17wi/521-lecture-7.pdf). According to the Schwartz-Zippel Lemma;
$$
\text{Let } {Q(X_1, . . . , X_n)} \text{ be a nonzero polynomial of } {n} \text{ variables with degree } {d}. \text{ Let } {S} \text{ be a finite but }\qquad\qquad\qquad\text{ }\text{ }\text{ }\text{ } \\ 
\text{sufficiently large subset of the field } {\mathbb{F}}.
\text{ If we assign } { X_1, . . . , X_n }
\text{ values from } {S} \text{ independently and } \qquad\qquad\qquad\text{ }\\ 
\text{ uniformly at random, then }\ 
{Pr[Q(X_1, . . . , X_n) = 0] ≤ \dfrac{d}{|S|}}. \qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\text{ }\text{ } \\
\text{ }
$$

**Here's what the Schwartz-Zippel Lemma means in our specific case**. In the proof/verification system of our mFibonacci state machine:

>​If the verifier selects the challenges $\{ \alpha_1, \alpha_2 . . . , \alpha_l \}$ randomly and uniformly, then the probability of the prover finding a false polynomial ${Q'}$ of degree $d$, such that ${Q'(\alpha_j) = 0 = Q(\alpha_j)}$ for all $j \in \{ 1, 2, \dots , l \}$ , is at most ${\dfrac{d}{|S|}}$, which is very small.

This speaks of the **soundness** of our polynomial commitment scheme.

## Proving the mFibonacci SM via PCS

Suppose the prover has to prove knowledge of the initial values of the mFibonacci Series that yields the $1024$-th term, $A_{1023} = \mathtt{14\ 823\ 897\ 298\ 192\ 278\ 947}$. Suppose a certain polynomial commitment scheme (PCS) is used to facilitate proving and verification.

In a typical commitment scheme, the following  protocol is followed;

1. The prover commits to the polynomials $P(X)$ and $Q(X)$.

2. The verifier selects a random point $\alpha$, sends it to the prover, with a request for relevant *openings*.
3. The prover then provides the openings; $P(\alpha)$, $P(\omega \alpha)$, $Q(\alpha)$ and $Q(\omega \alpha)$.
4. The verifier can check correctness of the openings, by using transition constraints,

$$
\big( 1 − R(X) \big) \cdot \big[ P(X\cdot \omega) − Q(X) \big] = \bigg\lvert_{\mathcal{H}}\ 0\qquad\quad\text{ }\text{ }\text{ }\text{ } \\
\big(1 − R(X)\big) · [Q(X\cdot \omega) − (P(X) · Q(X))] = \bigg\lvert_{\mathcal{H}}\ 0\text{ }\text{ }\\
$$

5. In order to enable the verifier to check the boundary constraint,
   $$
   P(\omega^{\mathtt{1023}}) = \mathtt{14\ 823\ 897\ 298\ 192\ 278\ 947}\qquad\qquad
   $$
   the prover must send a witness $\large{\mathtt{w}}$ as proof that he or she knows the correct value of the $1024$-th term, without disclosing the actual value $A_{1023} = \mathtt{14\ 823\ 897\ 298\ 192\ 278\ 947}$.

6. The verifier then uses a formula, which is specific to the commitment scheme in use and it takes the witness as an input, to check whether the prover has computed the correct $A_{1023}$.

Note that the prover does not provide any values concerning the constant polynomial $R(X)$, because this is known to both the prover and the verifier.

Any PCS such as [KZG](https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf) or [FRI-based](https://link.springer.com/content/pdf/10.1007%2F3-540-46766-1_9.pdf) can be used to efficiently prove correctness of computations of our mFibonacci state machine.

## Range of Identities

Let's look carefully at the constraints of the mFibonacci state machine;

$$
\big( 1 − R(X) \big) \cdot \big[ P(X\cdot \omega) − Q(X) \big] = \bigg\lvert_{\mathcal{H}}\ 0\qquad\quad\text{ }\text{ }\text{ }\text{ } \\
\big(1 − R(X)\big) · [Q(X\cdot \omega) − (P(X) · Q(X))] = \bigg\lvert_{\mathcal{H}}\ 0\text{ }\text{ }\\
\big(P(\omega^{\mathtt{T}}) - \mathcal{K} \big)\cdot R(X) = \bigg\lvert_{\mathcal{H}}\ 0\quad\text{ }\text{ }\quad\qquad\quad\quad\qquad\text{ }\text{ }\text{ }
$$

where  $\mathtt{T}+1$  is the number of rows in the execution trace and $\mathcal{K}$ is an evaluation of $P(X)$ at $\omega^{\mathtt{T}}$, corresponding to the value of the registry $\mathtt{A}$ in the $(\mathtt{T}+1)$-st state, when specific initial values $\mathtt{A}_0$ and $\mathtt{B}_0$ are used in the state machine.

The problem with the transition constraints, as presented above, is that they hold true only for $X \in \mathcal{H}$, and not necessarily for $X \in \mathbb{F}_p$. Note that the left-hand sides of the polynomial identities are but polynomials, which can be labelled $p_1(X)$, $p_2(X)$ and $p_3(X)$. That is;

$$
p_1(X) =\big( 1 − R(X) \big) \cdot \big[ P(X\cdot \omega) − Q(X) \big] =  \bigg\lvert_{\mathcal{H}}\ 0\qquad\quad\text{ }\text{ }\text{ }\text{ } \\
p_2(X) = \big(1 − R(X)\big) · [Q(X\cdot \omega) − (P(X) · Q(X))] = \bigg\lvert_{\mathcal{H}}\ 0\text{ }\text{ }\\
p_3(X) = \big(P(\omega^{\mathtt{T}}) - \mathcal{K} \big)\cdot R(X)  = \bigg\lvert_{\mathcal{H}}\ 0\quad\text{ }\text{ }\text{}\quad\qquad\qquad\qquad\text{ }
$$

Define the so-called *vanishing polynomial* on $\mathcal{H}$, denoted by $\mathtt{Z}_{\mathcal{H}}(X)$, as the monomial of maximal degree such that $\mathtt{Z}_{\mathcal{H}}(X) = 0$ for all $X \in \mathcal{H}$. Therefore;

$$
\mathtt{Z}_{\mathcal{H}}(X) = (X-1)\cdot(X-\omega)\cdot(X-\omega^2)\cdots(X-\omega^{n-1}) = X^{n} - 1
$$

Since  $p_i(X) = 0$  for all  $X \in \mathcal{H} = \{ \omega, \omega^2, \omega^3, \dots , \omega^n = 1 \}$, then;

$$
p_i(X)\ =\ \big((X-1)\cdot(X-\omega)\cdot(X-\omega^2)\cdots(X-\omega^{n-1})\big)\cdot q_i(X)\ =\ \big( X^{n} - 1 \big) \cdot q_i(X)
$$

for some quotient polynomial $q_i(X)$, for each $i \in \{ 1, 2, 3 \}$.

The polynomial identities of our mFibonacci state machine can therefore be rewritten as:

$$
\big( 1 − R(X) \big) \cdot \big[ P(X\cdot \omega) − Q(X) \big] = \mathtt{Z}_{\mathcal{H}}(X)\cdot q_1(X) \qquad\quad\text{ }\text{ }\text{ }\text{ } \\
\big(1 − R(X)\big) · [Q(X\cdot \omega) − (P(X) · Q(X))] = \mathtt{Z}_{\mathcal{H}}(X)\cdot q_2(X) \text{}\text{ }\\
\big(P(\omega^{\mathtt{T}}) - \mathcal{K} \big)\cdot R(X) = \mathtt{Z}_{\mathcal{H}}(X)\cdot q_3(X) \qquad\text{}\text{}\quad\qquad\qquad\qquad\text{ }
$$

The representatives $R(X)$ and $Z_{\mathcal{H}}(X)$ in the PCS, can be preprocessed and be made public (i.e., known to both the Prover and the Verifier). The Verifier can check specific openings of these polynomials, $R(X)$ and $Z_{\mathcal{H}}(X)$.