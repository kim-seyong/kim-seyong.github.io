---
layout: post
title: "Multibeam Satellite Communications with Massive MIMO: Asymptotic Performance Analysis and Design Insights"
date: 2026-08-16
math: true
description: "A short overview of our work on fixed-beam precoding, location-based user selection, and asymptotic scaling laws for massive-MIMO multibeam satellite communications."
---

Multibeam satellite systems can substantially increase throughput by reusing the same time-frequency resources across multiple spot beams. A major challenge, however, is **inter-beam interference**.

A conventional approach is to design MIMO precoders using channel state information (CSI) from ground users. While effective, this requires accurate CSI acquisition, repeated precoder computation, and potentially joint processing across gateways.

In this work, we ask a simpler question:

> **Can a massive-MIMO satellite achieve near-optimal performance using predefined fixed beams, without instantaneous CSI-based precoding?**

The key is to combine **fixed-beam precoding**, **location-based user selection**, and **massive MIMO**. We then study how the number of antennas, the ground-user density, and the number of beams should scale together.

## Core idea

We consider a GEO downlink satellite equipped with a uniform planar array (UPA). There are \(M\) antenna elements along each axis, so the total number of antennas is

$$
M^2.
$$

The satellite forms predefined fixed beams using the array steering vectors. The precoding vector for beam \(k\) is

$$
\mathbf{f}_k
=
\mathbf{v}(\vartheta_k^x)
\otimes
\mathbf{v}(\vartheta_k^y).
$$

Instead of adapting the beam to instantaneous CSI, each beam selects the ground user closest to its beam center:

$$
k^*
=
\arg\min_{\mathbf{d}_i\in\Phi_k}
\left\|
\mathbf{b}_k-\mathbf{d}_i
\right\|^2.
$$

The fixed precoder therefore does not require instantaneous CSI, while user selection relies only on spatial location, which corresponds to slowly varying long-term channel information.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-16-multibeam-satellite-massive-mimo/system_model.png' | relative_url }}" alt="Multibeam satellite system with fixed-beam precoding and location-based user selection" style="width: 4in; max-width: 100%; height: auto;">
</div>

*The satellite forms predefined spot beams and selects a nearby ground user for each beam using spatial information.*

## Why massive MIMO helps

A larger antenna array provides more array gain, but fixed-beam transmission introduces an important tradeoff.

As \(M\) increases, the beam becomes narrower. This is beneficial for suppressing inter-beam interference, but it also means that a selected user must be located increasingly close to the beam center.

If the user density is too low, the nearest user can remain outside the narrow main lobe, causing severe beam mismatch. On the other hand, if sufficiently many users are available, the satellite is more likely to find a user naturally aligned with each predefined beam.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-16-multibeam-satellite-massive-mimo/Beam_gain.png' | relative_url }}" alt="Beam gain comparison between phased array and parabolic reflector array" style="width: 4in; max-width: 100%; height: auto;">
</div>

*As the number of antennas increases, the phased-array beam becomes narrower. This makes user density an important part of massive-MIMO system design.*

This observation motivates the central scaling-law question of the paper:

> **How fast should the user density increase as the antenna array grows?**

## Single-beam scaling law

We first consider a single fixed beam.

Let the ground-user density scale as

$$
\lambda \sim M^q,
$$

where \(q\) controls how quickly the number of candidate users grows with the array size.

The main asymptotic result is

$$
\mathcal{R}_1
\sim
(q-1)\log M^2,
$$

where \(\mathcal{R}_1\) is the ergodic rate of the selected user.

This gives a simple interpretation.

- If \(q<1\), the user density grows too slowly and beam mismatch dominates asymptotically.
- If \(q>1\), the fixed-beam system achieves a non-vanishing rate scaling.
- If \(q=2\), the user density scales at the same rate as the total number of UPA antennas \(M^2\).

The last case is particularly important. When

$$
\lambda \sim M^2,
$$

the fixed-beam method achieves the same asymptotic rate scaling as an ideal scheme that perfectly aligns the beam with the selected user using CSI.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-16-multibeam-satellite-massive-mimo/single_rate.png' | relative_url }}" alt="Single-beam ergodic rate versus number of antennas" style="width: 4in; max-width: 100%; height: auto;">
</div>

*With insufficient user density, increasing the antenna array does not necessarily improve the fixed-beam rate. A denser user population is needed to exploit the narrower beams created by massive MIMO.*

## Multiple beams

We next extend the analysis to a multibeam system, where multiple fixed beams simultaneously reuse the same time-frequency resources.

We let the number of beams scale as

$$
K \sim M^{2\ell},
\qquad
0<\ell<1.
$$

The parameter \(\ell\) controls the beam-packing density.

Increasing \(\ell\) has two opposite effects. It allows the satellite to support more simultaneous beams, creating a larger spatial multiplexing gain. At the same time, the beams become more closely spaced, which increases inter-beam interference.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-16-multibeam-satellite-massive-mimo/sumrate.png' | relative_url }}" alt="Multibeam sum rate versus number of antennas" style="width: 4in; max-width: 100%; height: auto;">
</div>

*The multibeam sum rate reflects the tradeoff between spatial multiplexing gain and inter-beam interference.*

## Controlling inter-beam interference

The fixed-beam geometry lets us explicitly relate interference to beam spacing and user density.

As the beams are packed more densely, a larger user density is needed so that the selected user remains sufficiently close to the desired beam center and sufficiently separated from neighboring beam directions.

The analysis shows that the probability of maintaining sufficiently small interference improves as the user density grows relative to the beam-packing density.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-16-multibeam-satellite-massive-mimo/Fig_interfernece.png' | relative_url }}" alt="Probability of maintaining low inter-beam interference" style="width: 4in; max-width: 100%; height: auto;">
</div>

*Higher user density makes it more likely that inter-beam interference remains below the desired asymptotic level.*

## Multibeam scaling law

Under the asymptotic conditions established in the paper, the per-beam ergodic rate scales as

$$
\mathcal{R}_1^M
\sim
(q-\ell-1)\log M^2.
$$

Compared with the single-beam condition \(q>1\), the multibeam system requires

$$
q>\ell+1
$$

for non-vanishing per-beam rate scaling.

This additional \(\ell\) captures the extra user-density requirement created by operating a growing number of simultaneous beams.

At the same time, increasing the number of beams provides a multiplexing gain. This leads to a clear tradeoff:

> **More beams provide more spatial multiplexing, but they also require a denser user population to control beam mismatch and inter-beam interference.**

Most importantly, when

$$
\lambda \sim M^2,
$$

the fixed-beam multibeam system achieves the asymptotically optimal sum-rate scaling under the considered conditions, regardless of the beam-spacing exponent \(\ell\).

## Design insight

The main design message can be summarized through the interaction

$$
\boxed{
\text{antenna array size}
\quad\leftrightarrow\quad
\text{user density}
\quad\leftrightarrow\quad
\text{number of beams}
}
$$

A larger array provides higher beamforming gain and narrower beams.

A larger number of beams provides higher spatial multiplexing gain, but increases the interference burden.

A larger user density provides more opportunities to select users that are naturally well aligned with the predefined beams.

Therefore, massive MIMO should not be designed by considering the antenna array alone. The array size, beam spacing, and expected spatial user density need to be considered jointly.

## Takeaway

The main message of this work is that **massive MIMO and multi-user diversity can compensate for the lack of instantaneous CSI-based precoding in multibeam satellite systems**.

For a single beam,

$$
\lambda\sim M^q
\quad\Longrightarrow\quad
\mathcal{R}_1\sim(q-1)\log M^2.
$$

For multiple beams with

$$
K\sim M^{2\ell},
$$

the corresponding per-beam scaling becomes

$$
\mathcal{R}_1^M
\sim
(q-\ell-1)\log M^2.
$$

In particular, if the user density scales with the total number of antennas,

$$
\lambda\sim M^2,
$$

the fixed-beam approach can asymptotically achieve the ideal rate scaling without instantaneous CSI-based precoder adaptation.

In short, the key question is not simply how many antennas a satellite has, but whether the **antenna array, beam density, and ground-user density scale together in the right way**.

## Paper

This post is based on our paper:

**S. Kim, J. Choi, W. Shin, N. Lee, and J. Park, "Multibeam Satellite Communications with Massive MIMO: Asymptotic Performance Analysis and Design Insights."**

<!-- Add the paper URL here when available. -->
