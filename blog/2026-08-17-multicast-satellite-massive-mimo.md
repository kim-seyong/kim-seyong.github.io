---
layout: post
title: "Asymptotic Scaling Law Analysis of Multicast Satellite Communications with Massive MIMO"
date: 2026-08-17
math: true
description: "A short overview of our work on fixed-beam multicast transmission and the scaling interplay among massive-MIMO antennas, user density, and multicast group size."
---

Massive MIMO is attractive for satellite communications because a large antenna array can provide substantial beamforming gain. In GEO satellite systems, however, continuously adapting the beamformer to instantaneous channel state information (CSI) can be difficult because CSI acquisition creates signaling overhead and the long propagation delay can make CSI quickly outdated.

This motivates a fixed-beam approach, where the satellite points a predefined beam toward a fixed location rather than adapting the beam to instantaneous CSI.

In our previous work, we studied this idea for unicast transmission, where one user is selected for each beam. In this work, we extend the analysis to **multicast transmission**, where a single fixed beam simultaneously serves multiple ground users.

The main question is:

> **How does the multicast group size change the user-density scaling required to exploit massive MIMO with fixed beams?**

The answer is captured by a simple scaling law. If the user density and the number of multicast users scale as

$$
\lambda \sim M^q,
\qquad
N \sim M^t,
$$

then the multicast rate scales as

$$
\mathcal{R}_N
\sim
(q-t-1)\log M^2.
$$

The additional term \(t\) precisely captures the rate loss caused by serving a growing multicast group.

## Core idea

We consider a GEO satellite equipped with a uniform planar array (UPA). There are \(M\) antennas along each axis, so the total number of antennas is

$$
M^2.
$$

The spatial locations of ground users are modeled by a homogeneous Poisson point process (PPP) with density \(\lambda\).

The satellite uses a nadir-pointing fixed beam

$$
\mathbf{f}_1
=
\mathbf{v}(0)
\otimes
\mathbf{v}(0),
$$

and serves \(N\) multicast users inside a prescribed radius \(R\) around the beam center.

The selected user set is

$$
\mathcal{S}
=
\left\{
i\in\mathcal{N}
\,\middle|\,
\|\mathbf{x}_i\|<R
\right\},
\qquad
|\mathcal{S}|=N.
$$

This selection rule only requires spatial location information. It does not require each user to report instantaneous SINR or CSI for beam selection.

The radius \(R\) plays an important role: it determines how far the selected multicast users can be from the fixed-beam center and therefore controls the minimum beam gain available to the multicast group.

## Why multicast is different

For unicast transmission, the satellite only needs one sufficiently well-aligned user.

For multicast transmission, all \(N\) users must decode the same information. Therefore, the multicast rate is determined by the **worst user** in the selected group.

Without loss of generality, suppose the selected users are ordered according to their distances from the beam center:

$$
\|\mathbf{x}_1\|
\le
\cdots
\le
\|\mathbf{x}_N\|.
$$

Then user \(N\), which is the farthest selected user, determines the multicast rate.

Its beam gain can be expressed using the UPA steering vectors as

$$
f(r_N,\phi_N)
=
\left|
\left[
\mathbf{v}(\vartheta_N^x)
\otimes
\mathbf{v}(\vartheta_N^y)
\right]^{\sf H}
\left[
\mathbf{v}(0)
\otimes
\mathbf{v}(0)
\right]
\right|^2.
$$

As the number of antennas increases, the fixed beam becomes narrower. This means that maintaining a sufficiently large beam gain for **all** multicast users becomes increasingly demanding.

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-17-multicast-satellite-massive-mimo/beam_gain.png' | relative_url }}" alt="Beam gain for different azimuth angles and antenna array sizes" style="width: 4in; max-width: 100%; height: auto;">
</div>

*As the antenna array grows, the fixed beam becomes narrower. Multicast transmission therefore requires all selected users to remain sufficiently close to the beam center.*

## Main scaling law

To characterize this tradeoff, we let the number of multicast users scale as

$$
N\sim M^t
$$

and the ground-user density scale as

$$
\lambda\sim M^q.
$$

Under the asymptotic conditions established in the paper, the multicast ergodic rate satisfies

$$
\mathcal{R}_N
\sim
(q-t-1)\log M^2.
$$

This expression gives a direct interpretation of the three competing effects.

The term \(q\) represents the benefit of increasing user density. A denser user population makes it more likely that \(N\) users can be found sufficiently close to the beam center.

The term \(t\) represents the multicast penalty. As the multicast group grows, the worst selected user tends to be farther from the beam center, reducing the achievable beam gain.

The remaining \(1\) captures the user-density scaling needed to compensate for the narrowing fixed beam as the antenna array grows.

Therefore, the user-density scaling factor \(q\) and the multicast-user scaling factor \(t\) directly offset each other.

## Relative to ideal beamforming

We also compare the multicast rate with an ideal unicast benchmark that perfectly aligns the beam with a user using CSI.

The asymptotic ratio is

$$
\lim_{M\rightarrow\infty}
\frac{
\mathcal{R}_N
}{
\mathbb{E}\!\left[\log(1+PM^2)\right]
}
=
q-t-1.
$$

This result makes the multicast penalty particularly clear.

For the corresponding unicast case, the asymptotic fraction is

$$
q-1.
$$

For multicast, it becomes

$$
q-t-1.
$$

Hence, increasing the multicast group according to \(N\sim M^t\) reduces the asymptotic performance fraction by exactly \(t\).

<div style="text-align: center;">
  <img src="{{ '/assets/img/posts/2026-08-17-multicast-satellite-massive-mimo/line.png' | relative_url }}" alt="Ratio between multicast ergodic rate and ideal rate for different multicast scaling factors" style="width: 4in; max-width: 100%; height: auto;">
</div>

*The achievable fraction decreases linearly with the multicast scaling factor \(t\), while increasing the user-density scaling factor \(q\) compensates for this loss.*

## Compensating the multicast penalty

The scaling law also gives a simple design rule.

Suppose a unicast system operates with user-density scaling

$$
\lambda\sim M^q.
$$

If the multicast group grows as

$$
N\sim M^t,
$$

then increasing the user-density scaling by the same exponent \(t\) compensates for the multicast loss:

$$
\lambda
\sim
M^{q+t}.
$$

In particular, the stronger scaling

$$
\lambda
\sim
M^{2+t}
$$

allows the multicast fixed-beam system to asymptotically match the ideal unicast rate scaling considered in the paper.

This gives the central message of the work:

> **The rate loss caused by increasing the multicast group size can be exactly compensated by a proportional increase in user-density scaling.**

## Toward multibeam multicast

We also study inter-beam interference as a first step toward extending the framework to multibeam multicast satellite systems.

Suppose the number of beams scales as

$$
K\sim M^{2\ell},
$$

where \(\ell\) controls the spacing between neighboring fixed beams.

For a target interference scaling of

$$
\frac{1}{M^{2s}},
$$

the analysis characterizes how \(\ell\), \(s\), user density, and multicast group size interact.

One useful design interpretation is that the feasible beam-scaling factor should satisfy

$$
\ell
\le
\min(s,1-s).
$$

This captures the familiar tradeoff between multiplexing and interference: packing more beams increases spatial reuse, but the beams cannot be made arbitrarily dense while preserving the desired interference decay.

The interference result does not yet provide the full multibeam multicast rate scaling law. Instead, it provides the analytical building block needed to extend the single-beam multicast framework to that setting.

## Takeaway

The main result can be summarized in one expression:

$$
\boxed{
\lambda\sim M^q,
\quad
N\sim M^t
\quad\Longrightarrow\quad
\mathcal{R}_N
\sim
(q-t-1)\log M^2
}
$$

The interpretation is simple.

A larger massive-MIMO array creates narrower fixed beams.

A larger multicast group makes it harder to keep every selected user inside the high-gain region of the beam.

A larger ground-user density makes it easier to find enough multicast users close to the beam center.

Most importantly, the multicast penalty \(t\) and the user-density gain \(q\) enter the rate scaling law in a directly compensating way.

In this sense, multicast performance with fixed-beam massive MIMO is governed not by the antenna array size alone, but by the joint scaling of the **antenna array, user density, and multicast group size**.

## Paper

This post is based on our paper:

**S. Kim and J. Park, "Asymptotic Scaling Law Analysis of Multicast Satellite Communications with Massive MIMO."**

<!-- Add the paper URL here when available. -->
