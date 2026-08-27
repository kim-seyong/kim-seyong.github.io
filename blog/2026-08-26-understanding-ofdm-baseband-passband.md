---
layout: post
title: "Understanding OFDM: From Complex Baseband to Real Passband"
date: 2026-08-26
math: true
description: "A signal-flow note connecting OFDM complex baseband, I/Q conversion, real passband propagation, and receiver demodulation."
---

I wanted to write this post because, when I first studied wireless communications, I tended to learn OFDM and baseband-passband signaling as almost separate topics. That separation left me with several misconceptions for a long time.

What I eventually wanted to understand was the complete physical-layer signal flow:

$$
\begin{aligned}
\text{Digital Bits}
&\rightarrow
\text{OFDM Complex Baseband}
\rightarrow
\text{Analog Baseband}
\\
&\rightarrow
\text{Real Passband}
\rightarrow
\text{Wireless Channel}
\rightarrow
\text{Received Passband}
\\
&\rightarrow
\text{Complex Baseband}
\rightarrow
\text{OFDM Demodulation}.
\end{aligned}
$$

Following this entire chain helped connect several ideas that had previously seemed unrelated.

The MATLAB simulator used for this post is available here: [Git_OFDM_simulator](https://github.com/kim-seyong/Git_OFDM_simulator).

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/system-model.png' | relative_url }}" alt="End-to-end OFDM transceiver architecture considered in this post. The diagram connects digital OFDM processing, I/Q baseband-passband conversion, the TX and RX RF front-ends, the propagation channel, and receiver-side channel estimation, equalization, and decoding.">
  <p><em>Figure 1: End-to-end OFDM transceiver architecture considered in this post. The diagram connects digital OFDM processing, I/Q baseband-passband conversion, the TX and RX RF front-ends, the propagation channel, and receiver-side channel estimation, equalization, and decoding.</em></p>
</div>

Figure 1 expands this high-level flow into the end-to-end OFDM transceiver model used throughout the post. The goal is not to explain every block in equal detail. Instead, the figure serves as a map: we will mainly follow the signal as it moves from digital OFDM processing to analog I/Q waveforms, to a real RF passband waveform, through the channel, and back to complex baseband at the receiver. The exact baseline settings of implementation-dependent blocks such as interleaving, filters, gain stages, and converters are summarized later with the simulation parameters.

A complete discussion of channel coding, interleaving, QAM mapping, and decoding would make the post unnecessarily long. I therefore assume that the basic role of these blocks is already familiar and focus on the baseband-passband signal flow from the output of OFDM symbol.

## Constructing the OFDM Complex-Baseband Signal

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/map-to-baseband.png' | relative_url }}" alt="Generation of the OFDM complex-baseband signal.">
  <p><em>Figure 2: Generation of the OFDM complex-baseband signal.</em></p>
</div>

The symbols produced by a QAM or PSK mapper are not OFDM symbols by themselves. Each mapped symbol is assigned to a particular subcarrier by the resource mapper, while one OFDM symbol is represented by a set of frequency-domain values distributed across the $N$ FFT bins.
After resource mapping, the frequency-domain vector can be written as

$$
\mathbf{X}
=
\begin{bmatrix}
X[0] & X[1] & \cdots & X[N-1]
\end{bmatrix}^{T}.
$$

The resource mapper places data symbols, pilot symbols, the DC null, and guard subcarriers at the appropriate frequency-bin locations.
Using the unitary IFFT convention, the time-domain OFDM symbol is

$$
\boxed{
x[n]
=
\frac{1}{\sqrt{N}}
\sum_{k=0}^{N-1}
X[k]
e^{j2\pi kn/N},
\qquad
n=0,\ldots,N-1.
}
$$

A cyclic prefix of length $L_{\mathrm{CP}}$ is then appended. After parallel-to-serial conversion, the resulting sequence becomes the transmitted discrete-time complex-baseband waveform,

$$
\boxed{
x_b[n]
=
I[n]+jQ[n].
}
$$

## What Does a Complex Baseband Signal Physically Mean?

One point that initially confused me was the physical meaning of
$$
x_b[n]
=
I[n]+jQ[n].
$$
A circuit does not literally transmit a "complex voltage." The complex representation is a compact mathematical description of two real-valued orthogonal components:

$$
I[n]
=
\operatorname{Re}\{x_b[n]\},
\qquad
Q[n]
=
\operatorname{Im}\{x_b[n]\}.
$$

<div class="post-figure figure-half">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/complex-plane.png' | relative_url }}" alt="Complex baseband representation as two real I/Q components.">
  <p><em>Figure 3: Complex baseband representation as two real I/Q components.</em></p>
</div>

Therefore,

$$
\boxed{
x_b[n]
\quad\Longleftrightarrow\quad
\left(I[n],Q[n]\right).
}
$$

Although the $I$ and $Q$ components can carry independent symbol components in many modulation formats such as QAM, in an I/Q transceiver they are best understood as two orthogonal coordinates of a single complex baseband waveform rather than as two separate RF signals.

After digital-to-analog conversion and baseband reconstruction, the continuous-time complex envelope can be written as

$$
\boxed{
x_b(t)
=
I(t)+jQ(t)
=
A(t)e^{j\phi(t)}
}
$$

where the envelope magnitude is

$$
A(t)
=
|x_b(t)|
=
\sqrt{I^2(t)+Q^2(t)}
$$

and the envelope phase is

$$
\phi(t)
=
\angle x_b(t)
=
\angle \left(Q(t),I(t)\right).
$$

With this notation, the corresponding real passband waveform can also be written as

$$
x_p(t)
=
\sqrt{2}A(t)
\cos\left(2\pi f_ct+\phi(t)\right),
$$

which is equivalent to the I/Q expression derived below.

## DAC and Baseband Reconstruction

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/tx-iq.png' | relative_url }}" alt="I/Q DACs, reconstruction LPFs, and quadrature upconversion.">
  <p><em>Figure 4: I/Q DACs, reconstruction LPFs, and quadrature upconversion.</em></p>
</div>

The real and imaginary components are converted separately:

$$
I[n]
\rightarrow
\mathrm{DAC}
\rightarrow
I(t),
$$

$$
Q[n]
\rightarrow
\mathrm{DAC}
\rightarrow
Q(t).
$$

A practical DAC produces spectral images and often behaves approximately like a zero-order hold. For this reason, a reconstruction low-pass filter is commonly placed after each DAC. The corresponding physical signal flow is

$$
\boxed{
\mathrm{DAC}
\rightarrow
\mathrm{Reconstruction\ LPF}
\rightarrow
\mathrm{I/Q\ Upconverter}.
}
$$

In an ideal signal-flow derivation, this reconstruction stage is often treated as transparent. In a numerical simulator, however, the DAC and reconstruction filter still need a concrete implementation choice. The baseline choice used for the reported results, as well as the non-ideal effects that can be added later, is summarized in the parameter-setting section.

The continuous-time complex-baseband waveform is

$$
x_b(t)
=
I(t)+jQ(t).
$$

With the power-normalized convention used here, the corresponding real passband waveform is

$$
\boxed{
x_p(t)
=
\operatorname{Re}
\left\{
\sqrt{2}\,
x_b(t)
e^{j2\pi f_ct}
\right\}.
}
$$

Expanding,

$$
\begin{aligned}
x_p(t)
&=
\sqrt{2}\,
\operatorname{Re}
\left\{
\left(I(t)+jQ(t)\right)
\left(
\cos(2\pi f_ct)
+
j\sin(2\pi f_ct)
\right)
\right\}
\\
&=
\sqrt{2}I(t)\cos(2\pi f_ct)
-
\sqrt{2}Q(t)\sin(2\pi f_ct).
\end{aligned}
$$

The two carrier waveforms,

$$
\cos(2\pi f_ct)
$$

and

$$
-\sin(2\pi f_ct)
$$

are orthogonal and separated in phase by $90^\circ$. The factor $\sqrt{2}$ is introduced for power normalization. Without it,

$$
\operatorname{Re}
\left\{
x_b(t)e^{j2\pi f_ct}
\right\}
$$

has half the average power of the complex-baseband representation under the usual power definition. With the $\sqrt{2}$ normalization,

$$
\boxed{
\mathbb{E}
\left\{
|x_b(t)|^2
\right\}
=
\mathbb{E}
\left\{
x_p^2(t)
\right\}.
}
$$

This equality should be understood as an average-power relation, for example after averaging over the carrier phase or over a time interval long enough relative to the carrier period. It is not an instantaneous equality between $|x_b(t)|^2$ and $x_p^2(t)$.

## TX RF Front-End

<div class="post-figure figure-compact">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/tx-rf.png' | relative_url }}" alt="TX RF front-end consisting of an RF BPF and power amplifier.">
  <p><em>Figure 5: TX RF front-end consisting of an RF BPF and power amplifier.</em></p>
</div>

After ideal I/Q upconversion, $x_p(t)$ is already the desired real passband waveform. Therefore, an additional RF band-pass filter is not mathematically required in the ideal transmitter. In practical hardware, a TX RF BPF suppresses mixer spurs, LO leakage, translated DAC images, and other out-of-band emissions. The block is therefore kept in the architecture even when the baseline simulation treats the corresponding response ideally. The next block is the power amplifier.

For a linear PA,

$$
\boxed{
x'_p(t)
=
A_{\mathrm{PA}}x_p(t).
}
$$

If $G_{\mathrm{PA}}$ denotes its power gain,

$$
A_{\mathrm{PA}}
=
\sqrt{G_{\mathrm{PA}}}.
$$

If the receiver noise power is physically fixed, increasing the PA gain increases the received SNR. In a link-level SNR sweep, however, SNR can also be controlled directly as a simulation parameter. The specific convention used for the reported BLER and BER curves is described together with the simulation parameters.

The PA block is particularly important for OFDM because OFDM signals have a high peak-to-average power ratio. When the PA operates near saturation, nonlinear effects such as AM/AM distortion, AM/PM distortion, clipping, spectral regrowth, EVM degradation, and adjacent-channel leakage can become significant. The baseline simulation uses a linear PA, but the block-level implementation is kept explicit so that these nonlinear effects can be studied in future extensions.

## Passband Propagation Channel and Noise

<div class="post-figure figure-compact">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/tx-channel-rx.png' | relative_url }}" alt="Passband propagation channel with additive real AWGN.">
  <p><em>Figure 6: Passband propagation channel with additive real AWGN.</em></p>
</div>

The transmitted passband waveform $x'_p(t)$ then propagates through the wireless channel. A linear time-varying passband channel can be expressed as

$$
\boxed{
y'_p(t)
=
\int
h_p(t,\tau)
x'_p(t-\tau)
\,d\tau
+
n_p(t).
}
$$

The channel response

$$
h_p(t,\tau)
$$

can represent path loss, multipath propagation, fading, delay spread, and Doppler-induced time variation.

The additive disturbance can be represented as real, zero-mean, white Gaussian noise added to the passband waveform:

$$
\boxed{
n_p(t):
\text{ real passband AWGN}.
}
$$

For physical thermal noise, a conventional continuous-time model may use the two-sided power spectral density

$$
S_{n_p}(f)=\frac{N_0}{2}
$$

and the available thermal-noise power

$$
P_N=kTB,
$$

where $k$ is Boltzmann's constant, $T$ is the physical temperature, and $B$ is the receiver noise bandwidth. These expressions are useful physical background, but the baseline results use a link-level SNR definition rather than a thermal-noise calculation from temperature, bandwidth, and receiver noise figure. The exact noise-calibration convention is listed with the simulation parameters.

## RX RF Front-End: BPF and LNA

<div class="post-figure figure-compact">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/rx-lna.png' | relative_url }}" alt="RX RF front-end consisting of an RF BPF and LNA.">
  <p><em>Figure 7: RX RF front-end consisting of an RF BPF and LNA.</em></p>
</div>

In a practical receiver, the RX RF BPF selects the desired RF band,

$$
f_c-\frac{B}{2}
\leq
f
\leq
f_c+\frac{B}{2},
$$

and suppresses adjacent-channel interference, out-of-band noise, and unwanted mixer products. In an idealized baseline model, this RF filtering stage can be treated as transparent. In a more hardware-oriented model, the same block can be used to represent finite RF selectivity, passband ripple, group delay, and out-of-band rejection.

The resulting RF signal is then amplified by a low-noise amplifier (LNA). Although both the PA and the LNA are RF amplifiers, their design objectives are different. The PA is primarily designed for high output power, efficiency, and acceptable linearity at relatively large signal levels. The LNA is designed to provide sufficient gain while adding as little noise as possible to an already weak received signal.

An ideal linear LNA can be written as

$$
y_p(t)
=
A_{\mathrm{LNA}}
y_{\mathrm{BPF}}(t).
$$

Its noise factor is defined as

$$
\boxed{
F
=
\frac{
\mathrm{SNR}_{\mathrm{in}}
}{
\mathrm{SNR}_{\mathrm{out}}
}.
}
$$

An ideal noiseless LNA satisfies

$$
\boxed{
F=1.
}
$$

The corresponding noise figure is

$$
NF_{\mathrm{dB}}
=
10\log_{10}F,
$$

and therefore

$$
\boxed{
NF_{\mathrm{ideal}}
=
0~\mathrm{dB}.
}
$$

A larger noise factor or noise figure indicates greater SNR degradation. The importance of the LNA as the first active receiver stage can be understood from the Friis equation,

$$
\boxed{
F_{\mathrm{total}}
=
F_1
+
\frac{F_2-1}{G_1}
+
\frac{F_3-1}{G_1G_2}
+\cdots.
}
$$

A sufficiently large first-stage gain reduces the relative effect of noise introduced by subsequent receiver stages.

The simulation parameters later summarize the baseline LNA setting.

## I/Q Downconversion

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/rx-chain.png' | relative_url }}" alt="Receiver I/Q downconversion, low-pass filtering, VGA/AGC, ADC, and digital complex-baseband reconstruction.">
  <p><em>Figure 8: Receiver I/Q downconversion, low-pass filtering, VGA/AGC, ADC, and digital complex-baseband reconstruction.</em></p>
</div>

After the RX RF front-end, $y_p(t)$ is still a real passband waveform. Using the same normalization convention as in the transmitter, write it as

$$
\boxed{
y_p(t)
=
\sqrt{2}\hat I(t)\cos(2\pi f_ct)
-
\sqrt{2}\hat Q(t)\sin(2\pi f_ct).
}
$$

For the in-phase branch, the receiver multiplies the signal by

$$
\sqrt{2}\cos(2\pi f_ct).
$$

Then

$$
\begin{aligned}
y_p(t)\sqrt{2}\cos(2\pi f_ct)
&=
2\hat I(t)\cos^2(2\pi f_ct)
\\
&\quad
-
2\hat Q(t)
\sin(2\pi f_ct)
\cos(2\pi f_ct).
\end{aligned}
$$

Using

$$
2\cos^2\theta
=
1+\cos(2\theta)
$$

and

$$
2\sin\theta\cos\theta
=
\sin(2\theta),
$$

we obtain

$$
\boxed{
y_p(t)\sqrt{2}\cos(2\pi f_ct)
=
\hat I(t)
+
\hat I(t)\cos(4\pi f_ct)
-
\hat Q(t)\sin(4\pi f_ct).
}
$$

The LPF removes the components around $2f_c$, leaving

$$
\boxed{
\operatorname{LPF}
\left\{
y_p(t)\sqrt{2}\cos(2\pi f_ct)
\right\}
=
\hat I(t).
}
$$

Similarly, for the quadrature branch, the corresponding product is

$$
y_p(t)
\left[
-\sqrt{2}\sin(2\pi f_ct)
\right],
$$

which gives

$$
\begin{aligned}
&y_p(t)
\left[
-\sqrt{2}\sin(2\pi f_ct)
\right]
=
-2\hat I(t)
\cos(2\pi f_ct)
\sin(2\pi f_ct)
+
2\hat Q(t)
\sin^2(2\pi f_ct).
\end{aligned}
$$

Using

$$
2\sin^2\theta
=
1-\cos(2\theta),
$$

we obtain

$$
\boxed{
\begin{aligned}
y_p(t)
\left[
-\sqrt{2}\sin(2\pi f_ct)
\right]
&=
\hat Q(t)
-
\hat Q(t)\cos(4\pi f_ct)
\\
&\quad
-
\hat I(t)\sin(4\pi f_ct).
\end{aligned}
}
$$

Therefore,

$$
\boxed{
\operatorname{LPF}
\left\{
y_p(t)
\left[
-\sqrt{2}\sin(2\pi f_ct)
\right]
\right\}
=
\hat Q(t).
}
$$

The recovered continuous-time complex-baseband waveform is therefore

$$
\boxed{
\hat y_b(t)
=
\hat I(t)
+
j\hat Q(t).
}
$$

The derivation above describes the physical quadrature receiver. The numerical realization used in the simulator is summarized later with the other implementation-specific assumptions.

## VGA, AGC, and ADC

After I/Q downconversion and low-pass filtering, the signal passes through a variable-gain amplifier (VGA) before reaching the ADC. If the VGA amplitude gain is $g$, then

$$
I_{\mathrm{VGA}}(t)
=
g\hat I(t),
$$

$$
Q_{\mathrm{VGA}}(t)
=
g\hat Q(t).
$$

The same gain must be applied to both branches:

$$
\boxed{
g_I
=
g_Q
=
g.
}
$$

Otherwise, the received constellation is distorted by I/Q gain imbalance. The automatic gain control (AGC) does not directly amplify the signal. Instead, it measures the received signal level and determines the appropriate VGA gain.

If

$$
P_{\mathrm{in}}
=
\mathbb{E}
\left[
\hat I^2(t)
+
\hat Q^2(t)
\right]
$$

and the desired VGA output power is $P_{\mathrm{target}}$, an ideal AGC may choose

$$
\boxed{
g
=
\sqrt{
\frac{
P_{\mathrm{target}}
}{
P_{\mathrm{in}}
}
}.
}
$$

The purpose of the AGC is to keep the ADC input within an appropriate range, avoiding clipping while efficiently using the ADC dynamic range. The baseline AGC, VGA, and ADC assumptions used for the numerical results are summarized later in the parameter-setting section.

After ADC sampling,

$$
\boxed{
\hat y_b[n]
=
\hat I[n]
+
j\hat Q[n].
}
$$

## OFDM Demodulation

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/ofdm-demodulation.png' | relative_url }}" alt="OFDM demodulation, channel estimation, equalization, and decoding.">
  <p><em>Figure 9: OFDM demodulation, channel estimation, equalization, and decoding.</em></p>
</div>

The recovered discrete-time complex-baseband sequence $\hat y_b[n]$ is then processed by the digital OFDM receiver.

Synchronization may include

- frame detection,
- OFDM symbol timing,
- coarse carrier-frequency-offset correction,
- sampling-frequency-offset correction,
- residual phase tracking.

For the reported BLER/BER curves, timing is idealized using perfect channel-assisted alignment, and CFO/SFO impairments are absent. Thus, the list above should be understood as practical receiver context rather than a list of implemented synchronization estimators.

After synchronization, the cyclic prefix is removed and an $N$-point FFT is applied. Using the unitary FFT convention,

$$
\boxed{
\hat Y[k]
=
\frac{1}{\sqrt{N}}
\sum_{n=0}^{N-1}
\hat y[n]
e^{-j2\pi kn/N}.
}
$$

If the CP is sufficiently long and the channel does not vary too much within one OFDM symbol, the received signal on each subcarrier can be approximated as

$$
\boxed{
Y[k]
=
H[k]X[k]
+
W[k].
}
$$

## Pilot-Based Channel Estimation and Equalization

The receiver does not know the actual channel response $H[k]$ in advance. At a pilot subcarrier $k_{\mathrm{pilot}}$,

$$
Y[k_{\mathrm{pilot}}]
=
H[k_{\mathrm{pilot}}]
P[k_{\mathrm{pilot}}]
+
W[k_{\mathrm{pilot}}],
$$

where $P[k_{\mathrm{pilot}}]$ is a known transmitted pilot symbol. A simple least-squares pilot estimate is therefore

$$
\boxed{
\hat H[k_{\mathrm{pilot}}]
=
\frac{
\hat Y[k_{\mathrm{pilot}}]
}{
P[k_{\mathrm{pilot}}]
}.
}
$$

The channel response at the data subcarriers is then obtained by interpolation. In the numerical simulator, this basic pilot-aided idea is implemented using frequency-domain LMMSE estimation/interpolation. The LMMSE estimator is model-aided: it uses the configured CDL delay-profile statistics and calibrated noise variance, and temporal LMMSE combining across OFDM symbols is disabled.

For a zero-forcing equalizer,

$$
\boxed{
\hat X[k]
=
\frac{
\hat Y[k]
}{
\hat H[k]
}.
}
$$

After equalization, the receiver performs resource demapping, soft QAM demapping, rate recovery, LDPC decoding, and CRC checking to recover the transport block. The implementation-specific choices behind these receiver-side steps are listed in the parameter-setting section.

## MATLAB Realization of the CDL Channel

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/channel-equiv.png' | relative_url }}" alt="(a) Ideal passband channel representation. (b) Passband-oriented realization using MATLAB&#x27;s complex-baseband CDL channel model.">
  <p><em>Figure 10: (a) Ideal passband channel representation. (b) Passband-oriented realization using MATLAB's complex-baseband CDL channel model.</em></p>
</div>

In a physical communication system, the real passband signal $x'_p(t)$ propagates directly through the wireless channel. In contrast, MATLAB's `nrCDLChannel` implements a complex-baseband equivalent CDL channel. Therefore, in the passband-oriented MATLAB simulator considered here, $x'_p(t)$ is first downconverted to its complex-baseband representation, as illustrated in Fig. 10(b):

$$
\boxed{
x'_b(t)
=
\operatorname{LPF}
\left\{
\sqrt{2}
x'_p(t)
e^{-j2\pi f_ct}
\right\}.
}
$$

The resulting complex-baseband signal is then processed by the CDL channel. For a discrete-time linear time-varying representation,

$$
\boxed{
y_b^{\mathrm{CDL}}[n]
=
\sum_{\ell}
h_b^{\mathrm{CDL}}[n,\ell]
x'_b[n-\ell].
}
$$

The channel output is then converted back to a real passband representation:

$$
\boxed{
\tilde y'_p(t)
=
\sqrt{2}
\operatorname{Re}
\left\{
y_b^{\mathrm{CDL}}(t)
e^{j2\pi f_ct}
\right\}.
}
$$

Finally, real passband AWGN is added:

$$
\boxed{
y'_p(t)
=
\tilde y'_p(t)
+
n_p(t).
}
$$

This additional downconversion-CDL-upconversion sequence is not a physical requirement of the wireless channel itself. It is an implementation choice that comes from using a complex-baseband CDL channel model inside a passband-oriented simulator.

## Numerical Representation of the Analog Passband Signal

The signals after the DAC are conceptually continuous-time analog waveforms. MATLAB, however, can only represent them using discrete numerical samples. If the actual carrier frequency $f_c$ is explicitly represented in a real passband waveform and ordinary low-pass sampling is used for the numerical waveform, conventional Nyquist sampling requires a very high numerical sampling rate.

For example, when $f_c=3.5 \mathrm{GHz}$ and $B=20 \mathrm{MHz}$, this threshold becomes $2f_c+B=7.02 \mathrm{GHz}$. Thus, gigahertz-rate numerical sampling is required if the actual $3.5 \mathrm{GHz}$ RF carrier is explicitly represented in the time-domain simulation under this ordinary sampling approach, i.e., $F_s>7.02 \mathrm{GHz}$. Bandpass sampling can reduce the required sampling rate in some settings, but that is a different numerical modeling choice. This high-rate resampling is used only to numerically approximate the continuous-time analog/passband section. It does not change the OFDM FFT size or the OFDM parameters used to construct the original complex-baseband waveform.

## BLER and BER Results

<div class="post-figure">
  <img src="{{ '/assets/img/posts/2026-08-26-understanding-ofdm-baseband-passband/bler-ber-mcs-compare-full.png' | relative_url }}" alt="BLER and BER performance for MCS indices 3, 7, 11, and 15. Solid lines represent BLER and dashed lines represent BER. When zero errors are observed, the zero value is replaced only for logarithmic-axis visualization by 0.5/N_blocks for BLER or 0.5/N_bits for BER. The underlying measured error rate remains zero.">
  <p><em>Figure 11: BLER and BER performance for MCS indices 3, 7, 11, and 15. Solid lines represent BLER and dashed lines represent BER. When zero errors are observed, the zero value is replaced only for logarithmic-axis visualization by $0.5/N_{\mathrm{blocks}}$ for BLER or $0.5/N_{\mathrm{bits}}$ for BER. The underlying measured error rate remains zero.</em></p>
</div>

The block error rate is defined as

$$
\boxed{
\mathrm{BLER}
=
\frac{
N_{\mathrm{failed\ blocks}}
}{
N_{\mathrm{transmitted\ blocks}}
}.
}
$$

The bit error rate is evaluated over the decoded transport-block information bits:

$$
\boxed{
\mathrm{BER}
=
\frac{
N_{\mathrm{erroneous\ information\ bits}}
}{
N_{\mathrm{transmitted\ information\ bits}}
}.
}
$$

The BER count excludes the attached CRC bits and the rate-matched parity or repetition bits. It measures errors in the recovered information block.

Because a zero value cannot be displayed directly on a logarithmic axis, measured zero-error points are shown using the visualization surrogates

$$
\mathrm{BLER}_{\mathrm{plot}}
=
\frac{0.5}{N_{\mathrm{blocks}}}
$$

and

$$
\mathrm{BER}_{\mathrm{plot}}
=
\frac{0.5}{N_{\mathrm{bits}}}.
$$

These plotted values must not be interpreted as observed nonzero error rates. For example, if no block error is observed over 500 transmitted blocks, the measured BLER is $0/500=0$, but the point is displayed at $0.5/500=10^{-3}$ so that it remains visible on the logarithmic axis. Higher-SNR points may also be skipped after a zero-error stopping rule, so a missing higher-SNR point should be read as "not simulated after the stopping rule," not as an independently verified zero-error point.

The results show the expected trend: an MCS with higher modulation order or higher effective spectral efficiency generally requires a higher SNR to achieve the same BLER or BER. This does not mean that high-order QAM or high coding rates cannot achieve reliable communication without HARQ. At sufficiently high SNR, low error rates can also be obtained without retransmission. HARQ provides an additional reliability mechanism through retransmissions and, depending on the HARQ scheme, through combining information from multiple transmission attempts. HARQ is not included in the current baseline simulation.

## Summary

The overall signal flow can be summarized as

$$
\boxed{
\begin{aligned}
\text{OFDM Frequency-Domain Symbols}
&\rightarrow
\mathrm{IFFT}
\rightarrow
x_b[n]
\rightarrow
x_b(t)
\rightarrow
x_p(t)
\\
&\rightarrow
\text{Wireless Channel}
\rightarrow
y_p(t)
\rightarrow
\hat y_b(t)
\rightarrow
\hat y_b[n]
\rightarrow
\mathrm{FFT}.
\end{aligned}
}
$$

The main points are:

- The OFDM IFFT output is a complex-baseband waveform.
- The $I$ and $Q$ components can carry independent symbol components, but the representation $I+jQ$ describes two orthogonal coordinates of a single complex baseband waveform.
- I/Q upconversion transforms the complex-baseband representation into a real passband waveform.
- A conventional multipath propagation channel is linear, even when its coefficients vary over time.
- Pilot-based frequency-domain LMMSE channel estimation and ZF equalization compensate for the frequency-selective CDL channel.
- MATLAB's `nrCDLChannel` operates using a complex-baseband equivalent representation, which must be accounted for when explicitly modeling the real passband portion of the transceiver.
- Detailed choices for interleaving, converters, filters, gain stages, noise calibration, and result visualization are implementation-specific, so they are collected in the simulation-parameter section rather than scattered throughout the derivation.

Following the complete signal chain made it much easier for me to connect OFDM signal processing with the physical interpretation of baseband and passband signals in a wireless transceiver.

## Parameter Setting

The simulation results in Figure 11 use the following parameters.

<div class="table-wrap">
<table>
<caption>Table 1: OFDM waveform and transceiver simulation parameters.</caption>
<thead>
<tr><th>Category</th><th>Parameter</th><th>Value</th></tr>
</thead>
<tbody>
<tr><td>Carrier</td><td>Carrier frequency, $f_c$</td><td>$3.5 \mathrm{GHz}$</td></tr>
<tr><td>Bandwidth</td><td>Allocation/bandwidth parameter, $B$</td><td>$20 \mathrm{MHz}$</td></tr>
<tr><td>Sampling</td><td>Complex baseband sampling rate, $F_{s,\mathrm{BB}}$</td><td>$30.72 \mathrm{MHz}$</td></tr>
<tr><td>Sampling</td><td>CDL channel sampling rate, $F_{s,\mathrm{CDL}}$</td><td>$30.72 \mathrm{MHz}$</td></tr>
<tr><td>Sampling</td><td>Real-passband numerical sampling rate, $F_{s,\mathrm{RF}}$</td><td>$2f_c+1.5B=7.03 \mathrm{GHz}$</td></tr>
<tr><td>OFDM</td><td>FFT/IFFT size, $N_{\mathrm{FFT}}$</td><td>$2048$</td></tr>
<tr><td>OFDM</td><td>Subcarrier spacing, $\Delta f=F_{s,\mathrm{BB}}/N_{\mathrm{FFT}}$</td><td>$15 \mathrm{kHz}$</td></tr>
<tr><td>OFDM</td><td>Number of occupied subcarriers</td><td>$1332$</td></tr>
<tr><td>OFDM</td><td>Active subcarrier span</td><td>$1332\times15 \mathrm{kHz}=19.98 \mathrm{MHz}$</td></tr>
<tr><td>OFDM</td><td>Number of guard subcarriers</td><td>$715$</td></tr>
<tr><td>OFDM</td><td>DC subcarrier</td><td>One nulled subcarrier</td></tr>
<tr><td>OFDM</td><td>Cyclic-prefix length, $N_{\mathrm{CP}}$</td><td>$144$ samples</td></tr>
<tr><td>OFDM</td><td>Useful OFDM symbol duration</td><td>$66.6667 \mu\mathrm{s}$</td></tr>
<tr><td>OFDM</td><td>Total OFDM symbol duration</td><td>$71.3542 \mu\mathrm{s}$</td></tr>
<tr><td>Frame</td><td>Number of OFDM symbols per transport block</td><td>$14$</td></tr>
<tr><td>Frame</td><td>Number of baseband samples per frame</td><td>$30\,688$</td></tr>
<tr><td>Frame</td><td>Frame duration</td><td>$998.958 \mu\mathrm{s}$</td></tr>
<tr><td>Pilots</td><td>Pilot pattern</td><td>Custom comb pilots, not NR DM-RS</td></tr>
<tr><td>Pilots</td><td>Pilot spacing</td><td>One pilot every 12 occupied subcarriers</td></tr>
<tr><td>Pilots</td><td>Pilot symbol</td><td>$(1+j)/\sqrt{2}$</td></tr>
<tr><td>Pilots</td><td>Number of pilot resource elements per frame</td><td>$1554$</td></tr>
<tr><td>Data grid</td><td>Number of data resource elements per frame</td><td>$17\,094$</td></tr>
<tr><td>Modulation</td><td>Constellation mapping</td><td>Gray mapping with unit average symbol power</td></tr>
<tr><td>Channel coding</td><td>Coding scope</td><td>CRC24A and NR LDPC coding/rate-matching primitives; custom OFDM grid and TBS rule</td></tr>
<tr><td>Rate matching</td><td>Redundancy version</td><td>RV $=0$</td></tr>
<tr><td>Interleaving</td><td>NR rate-matching interleaving</td><td>Active</td></tr>
<tr><td>Interleaving</td><td>Extra full-vector random interleaver/deinterleaver</td><td>Disabled (bypass)</td></tr>
<tr><td>LDPC decoder</td><td>Decoding algorithm</td><td>Belief propagation with early termination</td></tr>
<tr><td>LDPC decoder</td><td>Maximum number of iterations</td><td>$100$</td></tr>
<tr><td>Equalization</td><td>Frequency-domain equalizer</td><td>Zero forcing (ZF)</td></tr>
<tr><td>Channel estimation</td><td>Estimator</td><td>Model-aided pilot-based frequency-domain LMMSE</td></tr>
<tr><td>Channel estimation</td><td>Temporal LMMSE combining</td><td>Disabled</td></tr>
<tr><td>Synchronization</td><td>Timing and frequency assumptions</td><td>Perfect channel-assisted timing alignment; CFO/SFO absent</td></tr>
<tr><td>RF components</td><td>DAC/ADC</td><td>Ideal rational sample-rate conversion</td></tr>
<tr><td>RF components</td><td>TX reconstruction LPF and TX/RX RF BPF</td><td>Explicit unity-gain bypasses</td></tr>
<tr><td>RF components</td><td>PA, LNA, and VGA gains</td><td>Linear $0 \mathrm{dB}$ gain blocks</td></tr>
<tr><td>RF components</td><td>AGC</td><td>Ideal bypass, $g=1$</td></tr>
<tr><td>Numeric type</td><td>Passband waveform</td><td>Single precision</td></tr>
</tbody>
</table>
</div>

<div class="table-wrap">
<table>
<caption>Table 2: Channel, noise, and Monte Carlo simulation parameters.</caption>
<thead>
<tr><th>Category</th><th>Parameter</th><th>Value</th></tr>
</thead>
<tbody>
<tr><td>Channel model</td><td>3GPP channel model</td><td>CDL-A</td></tr>
<tr><td>Antenna configuration</td><td>Transmit and receive antennas</td><td>SISO, $1\times1$, isotropic</td></tr>
<tr><td>Delay</td><td>Nominal RMS delay spread</td><td>$300 \mathrm{ns}$</td></tr>
<tr><td>Delay</td><td>Maximum CDL-A excess path delay</td><td>$2.89758 \mu\mathrm{s}$</td></tr>
<tr><td>Delay</td><td>MATLAB maximum discrete channel delay</td><td>$97$ samples at $30.72 \mathrm{MHz}$</td></tr>
<tr><td></td><td></td><td>($3.15885 \mu\mathrm{s}$, including filter implementation delay)</td></tr>
<tr><td>Delay</td><td>CDL channel-filter delay</td><td>$7$ samples ($0.227865 \mu\mathrm{s}$)</td></tr>
<tr><td>Doppler</td><td>Maximum Doppler shift, $f_{\mathrm{D,max}}$</td><td>$5 \mathrm{Hz}$</td></tr>
<tr><td>Doppler</td><td>Approximate equivalent terminal speed at $3.5 \mathrm{GHz}$</td><td>$0.429 \mathrm{m/s}$ ($1.54 \mathrm{km/h}$)</td></tr>
<tr><td>Channel normalization</td><td>Path-gain normalization</td><td>Enabled</td></tr>
<tr><td>Channel normalization</td><td>Output-power normalization</td><td>Enabled</td></tr>
<tr><td>Large-scale attenuation</td><td>Additional path loss</td><td>$0 \mathrm{dB}$</td></tr>
<tr><td>Channel realization</td><td>CDL random seed for transport block $b$</td><td>$73+b-1$</td></tr>
<tr><td>Noise</td><td>Noise model</td><td>Real zero-mean white Gaussian passband noise</td></tr>
<tr><td>Noise</td><td>Thermal-noise model</td><td>Not included</td></tr>
<tr><td>Noise</td><td>Receiver noise figure</td><td>Not included</td></tr>
<tr><td>Noise</td><td>Noise-bandwidth integration</td><td>Not used; noise is calibrated to target SNR</td></tr>
<tr><td>SNR definition</td><td>Reference SNR</td><td>Post-ADC complex-baseband SNR</td></tr>
<tr><td>SNR definition</td><td>Baseband noise variance</td><td>$\sigma^2_{n,\mathrm{BB}} =P_{s,\mathrm{BB}}10^{-\mathrm{SNR}_{\mathrm{dB}}/10}$</td></tr>
<tr><td>SNR calibration</td><td>Passband noise variance</td><td>$\sigma^2_{n,\mathrm{RF}} =\sigma^2_{n,\mathrm{BB}}/G_{\mathrm{RF}\rightarrow\mathrm{BB}}$</td></tr>
<tr><td>RF amplifier model</td><td>PA model</td><td>Linear, $0 \mathrm{dB}$, no nonlinear distortion</td></tr>
<tr><td>RF amplifier model</td><td>LNA model</td><td>Noiseless gain-only, $0 \mathrm{dB}$</td></tr>
<tr><td>SNR sweep</td><td>Evaluated SNR values</td><td>$\{-5,-2.143,0.714,3.571,6.429,9.286,12.143,15\} \mathrm{dB}$</td></tr>
<tr><td>BLER stopping rule</td><td>Minimum observed block errors</td><td>$250$</td></tr>
<tr><td>BER stopping rule</td><td>Minimum observed bit errors</td><td>$250$</td></tr>
<tr><td>Monte Carlo limit</td><td>Maximum number of blocks per MCS/SNR point</td><td>$500$</td></tr>
<tr><td>Early stopping</td><td>Zero-error stopping</td><td>Higher SNR points are skipped after measured BLER or BER reaches zero</td></tr>
<tr><td>Result visualization</td><td>Measured zero BLER</td><td>Plotted as $0.5/N_{\mathrm{blocks}}$ on the log axis</td></tr>
<tr><td>Result visualization</td><td>Measured zero BER</td><td>Plotted as $0.5/N_{\mathrm{bits}}$ on the log axis</td></tr>
</tbody>
</table>
</div>

### Simulation-specific implementation notes

The table entries above intentionally collect the implementation assumptions that are not central to the baseband-passband derivation:

- The simulator uses genuine NR coding and MCS-related components, including CRC24A, NR LDPC coding, NR rate matching, NR rate recovery, and NR LDPC decoding. However, the complete link is not a full NR PDSCH conformance implementation. The OFDM grid, pilot pattern, passband wrapper, and transport-block-size rule are simulator-specific.
- The explicit bit interleaver/deinterleaver outside the NR coding chain is disabled. The interleaving associated with NR LDPC rate matching remains active.
- The DAC and ADC are modeled as ideal rational sample-rate converters. Their polyphase filters provide the numerical interpolation, anti-imaging, and anti-aliasing operations. Finite converter resolution, quantization noise, clipping, saturation, and aperture jitter are not included in the baseline results.
- The TX reconstruction LPF and the TX/RX RF BPF blocks are implemented as unity-gain bypasses in the baseline. They are still kept as explicit blocks so that non-ideal reconstruction filtering, finite RF selectivity, passband ripple, group delay, or image rejection can be modeled later.
- The PA, LNA, and VGA are linear gain blocks with $0 \mathrm{dB}$ gain. The LNA does not add receiver noise, and the PA does not include nonlinear distortion in the baseline. Because OFDM has a high peak-to-average power ratio, PA nonlinearity remains an important future extension for AM/AM distortion, AM/PM distortion, clipping, spectral regrowth, EVM degradation, and adjacent-channel leakage studies.
- The AGC operates as an ideal bypass with $g=1$, and the same VGA gain is applied to the I and Q branches. I/Q gain mismatch is not included in the current baseline.
- Receiver downconversion is implemented using an analytic-signal/Hilbert-transform representation. Under the ideal image-rejection assumption, this is mathematically equivalent to quadrature mixing by $\sqrt{2}\cos(2\pi f_ct)$ and $-\sqrt{2}\sin(2\pi f_ct)$ followed by removal of the components around $2f_c$.
- The $20 \mathrm{MHz}$ value specifies the allocation/bandwidth parameter used to choose the active subcarrier set. The simulator does not calculate AWGN from $kTB$, physical temperature, receiver noise figure, or thermal-noise bandwidth integration. Instead, real passband AWGN is calibrated to the requested post-ADC complex-baseband SNR. Therefore, the horizontal axis of Figure 11 is SNR rather than $E_b/N_0$.

<div class="table-wrap">
<table>
<caption>Table 3: Complete MCS mapping used by the simulator.</caption>
<thead>
<tr><th>MCS</th><th>$Q_m$</th><th>Modulation</th><th>$R_{\mathrm{tab}}$</th><th>$R_{\mathrm{tab}}/1024$</th></tr>
</thead>
<tbody>
<tr><td>0</td><td>2</td><td>QPSK</td><td>120</td><td>0.11719</td></tr>
<tr><td>1</td><td>2</td><td>QPSK</td><td>157</td><td>0.15332</td></tr>
<tr><td>2</td><td>2</td><td>QPSK</td><td>193</td><td>0.18848</td></tr>
<tr><td>3</td><td>2</td><td>QPSK</td><td>251</td><td>0.24512</td></tr>
<tr><td>4</td><td>2</td><td>QPSK</td><td>308</td><td>0.30078</td></tr>
<tr><td>5</td><td>2</td><td>QPSK</td><td>379</td><td>0.37012</td></tr>
<tr><td>6</td><td>2</td><td>QPSK</td><td>449</td><td>0.43848</td></tr>
<tr><td>7</td><td>2</td><td>QPSK</td><td>526</td><td>0.51367</td></tr>
<tr><td>8</td><td>2</td><td>QPSK</td><td>602</td><td>0.58789</td></tr>
<tr><td>9</td><td>2</td><td>QPSK</td><td>679</td><td>0.66309</td></tr>
<tr><td>10</td><td>4</td><td>16QAM</td><td>340</td><td>0.33203</td></tr>
<tr><td>11</td><td>4</td><td>16QAM</td><td>378</td><td>0.36914</td></tr>
<tr><td>12</td><td>4</td><td>16QAM</td><td>434</td><td>0.42383</td></tr>
<tr><td>13</td><td>4</td><td>16QAM</td><td>490</td><td>0.47852</td></tr>
<tr><td>14</td><td>4</td><td>16QAM</td><td>553</td><td>0.54004</td></tr>
<tr><td>15</td><td>4</td><td>16QAM</td><td>616</td><td>0.60156</td></tr>
<tr><td>16</td><td>4</td><td>16QAM</td><td>658</td><td>0.64258</td></tr>
<tr><td>17</td><td>6</td><td>64QAM</td><td>438</td><td>0.42773</td></tr>
<tr><td>18</td><td>6</td><td>64QAM</td><td>466</td><td>0.45508</td></tr>
<tr><td>19</td><td>6</td><td>64QAM</td><td>517</td><td>0.50488</td></tr>
<tr><td>20</td><td>6</td><td>64QAM</td><td>567</td><td>0.55371</td></tr>
<tr><td>21</td><td>6</td><td>64QAM</td><td>616</td><td>0.60156</td></tr>
<tr><td>22</td><td>6</td><td>64QAM</td><td>666</td><td>0.65039</td></tr>
<tr><td>23</td><td>6</td><td>64QAM</td><td>719</td><td>0.70215</td></tr>
<tr><td>24</td><td>6</td><td>64QAM</td><td>772</td><td>0.75391</td></tr>
<tr><td>25</td><td>6</td><td>64QAM</td><td>822</td><td>0.80273</td></tr>
<tr><td>26</td><td>6</td><td>64QAM</td><td>873</td><td>0.85254</td></tr>
<tr><td>27</td><td>6</td><td>64QAM</td><td>910</td><td>0.88867</td></tr>
<tr><td>28</td><td>6</td><td>64QAM</td><td>948</td><td>0.92578</td></tr>
</tbody>
</table>
</div>

The BLER/BER comparison in Figure 11 evaluates MCS indices 3, 7, 11, and 15 from this table. For each selected MCS, the simulator computes the rate-matched coded-bit length as $E=N_{\mathrm{data\ RE}}Q_m$ and the number of information bits before CRC24A attachment as $A=\lfloor E(R_{\mathrm{tab}}/1024)\rfloor$.

## References

This post was written with reference to David Tse and Pramod Viswanath's *Fundamentals of Wireless Communication*, Upamanyu Madhow's *Fundamentals of Digital Communication*, and my earlier simulator work, **OFDM Simulator for Underwater Acoustic Communications**.
