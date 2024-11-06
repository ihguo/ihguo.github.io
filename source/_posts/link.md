---
title: Wireline Link Budgeting and Modeling
date: 2024-11-02 12:34:55
tags:
categories:
- analog
mathjax: true
---

![image-20241019142915175](link/image-20241019142915175.png)

---



## CDR Loop Latency

![image-20241102235118149](link/image-20241102235118149.png)

![image-20241102235145417](link/image-20241102235145417.png)

loop latency is represented as $e^{-sD}$ in linear model

---

![image-20241102235736432](link/image-20241102235736432.png)

![image-20241103000223470](link/image-20241103000223470.png)

![image-20241103000653906](link/image-20241103000653906.png)



## Sensitivity to Loop Latency

![image-20241103142137640](link/image-20241103142137640.png)

---

![image-20241103142656134](link/image-20241103142656134.png)

![image-20241103142531277](link/image-20241103142531277.png)

![image-20241103142938907](link/image-20241103142938907.png)



## Enhancing Resolution with a $\Delta \Sigma$ Modulator

> **Sub-Resolution Time Averaging**

![image-20241103160332995](link/image-20241103160332995.png)

$\Delta \Sigma$ modulator effectively **dithers** the **LSB** bit between *zero and one*, such that you can get the effective resolution of a much higher resolution DAC in the number of bits



## Decimation

![image-20241020140430663](link/image-20241020140430663.png)

DLF's input bit-width can be reduced by *decimating* BBPD's output. Decimation is typically performed by realizing either **majority voting (MV)** or **boxcar filtering**.

> Note that **deserialization** is inherent to both **MV** and **boxcar** filtering

![image-20241019225016868](link/image-20241019225016868.png)

- Decimation is commonly employed to alleviate the high-speed requirement. However, decimation increases loop-latency which causes excessive dither jitter.
- Decimation is basically, widen the data and slowing it down
- Decimating by $L$ means frequency register only added once every $L$ UI, thus *integral path gain* reduced by $L$ in linear model
- *proportional path gain* is unchanged

![intg_path_decim.drawio](link/intg_path_decim.drawio.svg)


### Decimation by Summing


> The loop gain of a **proportional path** is **unchanged**

![phug_loop.drawio](link/phug_loop.drawio.svg)

In *(a)*, the loop gain is $\frac{\phi_o(z)}{\phi_e(z)}$, which is
$$
LG_a(z)=\frac{\phi_o(z)}{\phi_e(z)} = \frac{1}{1-z^{-1}}
$$

In *(b)*, Accumulate-and-dump (AAD) is $\frac{1-z^{-L}}{1-z^{-1}}$, then $\phi_m(\eta)$ can be expressed as
$$
\phi_m(\eta) = \frac{1-\eta^{-1}}{1-\eta^{-1/L}}\cdot \frac{1}{L}
$$
Hence
$$\begin{align}
\phi_o(\eta) &= \phi_m(\eta) \frac{1}{1-\eta^{-1}} \\
&= \frac{1-\eta^{-1}}{1-\eta^{-1/L}}\cdot \frac{1}{L}\cdot \frac{1}{1-\eta^{-1}} \\
&= \frac{1}{1-\eta^{-1/L}}\cdot \frac{1}{L}
\end{align}$$

After *zero-order hold process*, we obtain $\phi_f(z)$, which is
$$\begin{align}
\phi_f(z) &= \phi_o(z^L) \cdot \frac{1-z^{-L}}{1-z^{-1}} \\
&=\frac{1}{1-z^{-1}}\cdot \frac{1}{L}\cdot \frac{1-z^{-L}}{1-z^{-1}}
\end{align}$$
That is,
$$
LG_b(z) = \frac{1}{1-z^{-1}}\cdot \frac{1}{L}\cdot \frac{1-z^{-L}}{1-z^{-1}}
$$

while bandwidth is less than sampling rate (data rate), $\frac{1}{L}\cdot \frac{1-z^{-L}}{1-z^{-1}} \approx 1$, i.e. $LG_a(z)\approx LG_b(z)$. with

$$
\frac{1}{1-z^{-1}}\cdot \frac{1}{L}\cdot \frac{1-z^{-L}}{1-z^{-1}} \approx \frac{1}{1-z^{-1}}
$$

---

> **integral path gain reduced by $L$**

![frug_loop.drawio](link/frug_loop.drawio.svg)

In *(a)*, $\phi_o(z)=\frac{1}{(1-z^{-1})^2}$, i.e.
$$
LG_a(z) = \frac{1}{(1-z^{-1})^2}
$$

In (b), after Accumulate-and-dump (AAD), $\phi_(\eta)$ is
$$
\phi_m(\eta) = \frac{1-\eta^{-1}}{1-\eta^{-1/L}}\cdot \frac{1}{L}
$$

After frequency integrator and phase integrator
$$\begin{align}
\phi_o(\eta) &= \phi_m(\eta) \cdot \frac{1}{(1-\eta^{-1})^2} \\
&= \frac{1-\eta^{-1}}{1-\eta^{-1/L}}\cdot \frac{1}{L} \cdot  \frac{1}{(1-\eta^{-1})^2}
\end{align}$$
Then $\phi_f(z)$ is shown as below
$$\begin{align}
\phi_f(z) &= \phi_o(z^L)\cdot \frac{1-z^{-L}}{1-z^{-1}} \\
&= \frac{1-z^{-L}}{1-z^{-1}}\cdot \frac{1}{L}\cdot \frac{1}{(1-z^{-L})^2}\cdot \frac{1-z^{-L}}{1-z^{-1}} \\
&= \frac{1}{L} \cdot \frac{1}{(1-z^{-1})^2}
\end{align}$$

That is,
$$
LG_b(z) = \frac{1}{L} \cdot \frac{1}{(1-z^{-1})^2} = \frac{1}{L}\cdot LG_a(z)
$$


### Decimation by Voting

> equivalent $\sum d_n \lt 0 \to -1$, $\sum d_n = 0 \to 0$ and $\sum d_n\gt 0 \to 1$
>
> Compared to the boxcar filter, voting is able to reduce the loop delay and lower the output noise of the MMPD

- Decimation via voting has a reduced gain, $K_V$, which can be determined through simulation





## CDR Linear Model

![image-20220504101924272](link/image-20220504101924272.png)

> condition:
>
> Linear model of the CDR is used in a **frequency lock condition** and is approaching to achieve **phase lock**

Using this model, the power spectral density (PSD) of jitter in the recovered clock $S_{out}(f)$ is
$$
S_{out}(f)=|H_T(f)|^2S_{in}(f)+|H_G(f)|^2S_{VCO}(f)
$$
Here, we assume $\varphi_{in}$ and $\varphi_{VCO}$ are uncorrelated as they come from independent sources.

### Jitter Transfer

$$
H_T(s) = \frac{\varphi_{out}(s)}{\varphi_{in}(s)}|_{\varphi_{vco}=0}=\frac{K_{PD}K_{VCO}R_s+\frac{K_{PD}K_{VCO}}{C}}{s^2+K_{PD}K_{VCO}R_s+\frac{K_{PD}K_{VCO}}{C}}
$$

Using below notation
$$\begin{align}
\omega_n^2=\frac{K_{PD}K_{VCO}}{C} \\
\xi=\frac{K_{PD}K_{VCO}}{2\omega_n^2}
\end{align}$$

We can rewrite transfer function as follows
$$
H_T(s)=\frac{2\xi\omega_n s+\omega_n^2}{s^2+2\xi \omega_n s+\omega_n^2}
$$

> The jitter transfer represents a **low-pass filter** whose magnitude is around 1 (0 dB) for low jitter frequencies and drops at 20 dB/decade for frequencies above $\omega_n$

![image-20220504104202197](link/image-20220504104202197.png)

- the recovered clock **track** the *low-frequency jitter* of the input data
- the recovered clock **DONT** track the *high-frequency jitter* of the input data

The recovered clock does not suffer from high-frequency jitter even though the input signal may contain high-frequency jitter, which will limit the CDR tolerance to high-frequency jitter.



### Jitter Peaking in Jitter Transfer Function

The peak, slightly larger than 1 (0dB) implies that jitter will be **amplified** at some frequencies in the CDR, producing a jitter amplitude in the recovered clock, and thus also in the recovered data, that is slightly **larger** than the jitter amplitude in the input data.

This is certainly undesirable, especially in applications such as repeaters.

![image-20220504110722442](link/image-20220504110722442.png)

### Jitter Generation

If the input data to the CDR is clean with no jitter, i.e., $\varphi_{in}=0$, the jitter of the recovered clock comes directly from the VCO jitter. The transfer function that relates the VCO jitter to the recovered clock jitter is known as **jitter generation**.
$$
H_G(s)=\frac{\varphi_{out}}{\varphi_{VCO}}|_{\varphi_{in}=0}=\frac{s^2}{s^2+2\xi \omega_n s+\omega_n^2}
$$
Jitter generation is **high-pass filter** with two zeros, at zero frequency, and two poles identical to those of the jitter transfer function

![image-20220504110737718](link/image-20220504110737718.png)

### Jitter Tolerance

To quantify jitter tolerance, we often apply a sinusoidal jitter of a fixed frequency to the CDR input data and observe the BER of the CDR

The jitter tolerance curve **DONT** capture a CDR's true tolerance to **random** jitter. Because we are applying "sinusoidal" jitter, which is **deterministic** signal.

We can deal only with the jitter's amplitude and frequency instead of the PSD of the jitter thanks to deterministic sinusoidal jitter signal.
$$
JTOL(f) = \left | \varphi_{in}(f)  \right |_{\text{pp-max}} \quad \text{for a fixed BER}
$$
Where the subscript $\text{pp-max}$ indicates the maximum peak-to-peak amplitude. We can further expand this equation as follows
$$
JTOL(f)=\left| \frac{\varphi_{in}(f)}{\varphi_{e}(f)} \right| \cdot |\varphi_e(f)|_{pp-max}
$$
![image-20220504114650749](link/image-20220504114650749.png)

> Relative jitter, $\varphi_e$ must be less than 1UIpp for error-free operation



In an ideal CDR, the **maximum peak-to-peak amplitude** of $|\varphi_e(f)|$ is **1UI**, i.e.,$|\varphi_e(f)|_{pp-max}=1UI$

Accordingly, jitter tolerance can be expressed in terms of the number of UIs as
$$
JTOL(f)=\left| \frac{\varphi_{in}(f)}{\varphi_{e}(f)} \right|\quad \text{[UI]}
$$
Given the linear CDR model, we can write
$$
JTOL(f)=\left| 1+\frac{K_{PD}K_{VCO}H_{LF}(f)}{j2\pi f} \right|\quad \text{[UI]}
$$
Expand $H_{LF}(f)$ for the CDR, we can write
$$
JTOL(f)=\left| 1-2\xi j \left(\frac{f_n}{f}\right) - \left(\frac{f_n}{f}\right)^2 \right|\quad \text{[UI]}
$$
![image-20220504120538534](link/image-20220504120538534.png)

At frequencies far below and above the natural frequency, the jitter tolerance can be approximated by the following
$$
JTOL(f) = \left\{ \begin{array}{cl}
\left(\frac{f_n}{f}\right)^2 & : \ f\ll f_n \\
1 & : \ f\gg f_n
\end{array} \right.
$$

- the jitter tolerance at very **high jitter frequencies** is limited to **1UIpp**
  - This is consistent with that the recovered clock does not track the high-frequency jitter, limiting the maximum peak-to-peak deviation of the data edge from its nominal position to 1UI
  - The circumstance, (b) jittery data with ideal clock
- the jitter tolerance is increased at 40dB/decade for jitter frequencies below $f_c$
  - This is consistent with our obervation earlier that the recovered clock better tracks data jitter at lower jitter frequencies
  - Equivalently, the data edge and the clock edge move together in the **same direction**. As a result, the relative jitter between the data and the clock remains small, i.e., below 1UI peak-to-peak
  - The circumstance, (c) jittery data and jittery clock

### OJTF

#### Concepts of JTF and OJTF

Simplified Block Diagram of a Clock-Recovery PLL
![pll_block_diagram](link/pll_block_diagram.gif)

**Jitter Transfer Function (JTF)**

- Input Signal Versus Recovered Clock
- JTF, by jitter frequency, compares how much input signal jitter is transferred to the **output of a clock-recovery's PLL (recovered clock)**
  - Input signal jitter that is within the clock recovery PLL's loop bandwidth results in jitter that is faithfully transferred (closed-loop gain) to the clock recovery PLL's output signal. JTF in this situation is approximately 1.
  - Input signal jitter that is outside the clock recovery PLL's loop bandwidth results in decreasing jitter (open-loop gain) on the clock recovery PLL's output, because the jitter is filtered out and no longer reaches the PLL's VCO

**Observed Jitter Transfer Function**

- Input Signal Versus Sampled Signal
- OJTF compares how much input signal jitter is transferred to the **output of a receiver's decision making circuit** as effected by a clock recovery's PLL. As the recovered clock is the reference for detecting the input signal
  - Input signal jitter that is **within** the clock recovery PLL's loop bandwidth results in jitter on the recovered clock which reduces the amount of jitter that can be detected. The input signal and clock signal are closer in phase
  - Input signal jitter that is **outside** the clock recovery PLL's loop bandwidth results in **reduced jitter on the recovered cloc**k which increases the amount of jitter that can be detected. The input signal and clock signal are more out of phase. Jitter that is on both the input and clock signals can not detected or is reduced

JTF and OJTF for 1st Order PLLs

![jsa_1st_order_graph](link/jsa_1st_order_graph.png)

![neuhelium-jtf-ojtf](link/neuhelium-jtf-ojtf.png)

> The observed jitter is a complement to the PLL jitter transfer response OJTF=1-JTF **(Phase matters!)**
>
> OTJF gives the amount of jitter which is tracked and therefore not observed at the output of the CDR as a function of the jitter rate applied to the input.


![A-jtf-ojtf](link/A-jtf-ojtf.png)

#### Jitter Measurement

$$
J_{\text{measured}} = JTF_{\text{DUT}} \cdot OJTF_{\text{instrument}}
$$

The combination of the OJTF of a jitter measurement device and the JTF of the clock generator under test gives the measured jitter as a function of frequency.

![image-20220716094732273](link/image-20220716094732273.png)

For example, a clock generator with a type 1, 1st order PLL measured with a jitter measurement device employing a golden PLL is
$$
J_{\text{measured}} = \frac{\omega_1}{s+\omega_1}\frac{s}{s+\omega_2}
$$

Accurate measurement of the clock JTF requires that the OJTF cutoff of the jitter measurement be significantly below that of the clock JTF and that the measurement is compensated for the instrument's OJTF.

The overall response is a band pass filter because the clock JTF is low pass and the jitter measurement device OJTF is high pass.

The compensation for the instrument OJTF is performed by measuring the jitter of the reference clock at each jitter rate being tested and comparing the **reference jitter** with the **jitter measured at the output of the DUT**.

![jtf-ojtf](link/jit_resp.png)

The lower the cutoff frequency of the jitter measurement device the better the accuracy of the measurement will be.

The cutoff frequency is limited by several factors including the phase noise of the DUT and measurement time.

#### Digital Sampling Oscilloscope

How to analyze jitter:

- TIE (Time Interval Error) track
- histogram
- FFT

**TIE track** provides a direct view of how the phase of the clock evolves over time.

**histogram** provides valuable information about the long term variations in the timing.

**FFT** allows jitter at specific rates to be measured down to the femto-second range.

> Maintaining the record length at a minimum of $1/10$ of the inverse of the PLL loop bandwidth minimizes the response error



### reference

Dalt, Nicola Da and Ali Sheikholeslami. “Understanding Jitter and Phase Noise: A Circuits and Systems Perspective.” (2018).

neuhelium, 抖动、眼图和高速数字链路分析基础 URL: [http://www.neuhelium.com/ueditor/net/upload/file/20200826/DSOS254A/03.pdf](http://www.neuhelium.com/ueditor/net/upload/file/20200826/DSOS254A/03.pdf)

Keysight JTF & OJTF Concepts, [https://rfmw.em.keysight.com/DigitalPhotonics/flexdca/FlexPLL-UG/Content/Topics/Quick-Start/jtf-pll-theory.htm?TocPath=Quick%20Start%7C_____4](https://rfmw.em.keysight.com/DigitalPhotonics/flexdca/FlexPLL-UG/Content/Topics/Quick-Start/jtf-pll-theory.htm?TocPath=Quick%20Start%7C_____4)

Complementary Transmitter and Receiver Jitter Test Methodlogy, URL: [https://www.ieee802.org/3/bm/public/mar14/ghiasi_01_0314_optx.pdf](https://www.ieee802.org/3/bm/public/mar14/ghiasi_01_0314_optx.pdf)

SerDesDesign.com CDR_BangBang_Model URL: [https://www.serdesdesign.com/home/web_documents/models/CDR_BangBang_Model.pdf](https://www.serdesdesign.com/home/web_documents/models/CDR_BangBang_Model.pdf)

M. Schnecker, Jitter Transfer Measurement in Clock Circuits, LeCroy Corporation, DesignCon 2009. URL: [http://cdn.teledynelecroy.com/files/whitepapers/designcon2009_lecroy_jitter_transfer_measurement_in_clock_circuits.pdf](http://cdn.teledynelecroy.com/files/whitepapers/designcon2009_lecroy_jitter_transfer_measurement_in_clock_circuits.pdf)







## Link Budgets



### VCO model

*TODO* &#128197;

> respone to vctrl
> focus on phase




> [[https://designers-guide.org/verilog-ams/functional-blocks/vco/vco.va](https://designers-guide.org/verilog-ams/functional-blocks/vco/vco.va)]





### ADC Spec

*TODO* &#128197;

ENOB - Not sufficient & not accurate enough

- Based on SNDR
- Assume unbounded Gaussian distribution

> quantization noise is ~ bounded uniform distribution
>
> Using unbounded Gaussian -> pessimistic BER prediction



### AFE Nonlinearity

> ***"total harmonic distortion" (THD)*** in AFE

Relative to NRZ-based systems, PAM4 transceivers require more stringent circuit linearity, equalizers which can implement multi-level inter-symbol interference (ISI) cancellation, and improved sensitivity

![image-20240923204055369](link/image-20240923204055369.png)



Because if it compresses, it turns out you have to use a much more complicated feedback filter. As long as *it behaves linearly*, the feedback filter itself can remain *a linear FIR*



![image-20240923211841053](link/image-20240923211841053.png)

Linearity can actually be a critical constraint in these signal paths, and you really want to stay as linear as you can all the way up until the point where you've canceled all of the ISI

![image-20240923222650556](link/image-20240923222650556.png)



> A. Roshan-Zamir, O. Elhadidy, H. -W. Yang and S. Palermo, "A Reconfigurable 16/32 Gb/s Dual-Mode NRZ/PAM4 SerDes in 65-nm CMOS," in *IEEE Journal of Solid-State Circuits*, vol. 52, no. 9, pp. 2430-2447, Sept. 2017  [[https://people.engr.tamu.edu/spalermo/ecen689/2017_reconfigurable_16_32Gbps_NRZ_PAM4_SERDES_roshanzamir_jssc.pdf](https://people.engr.tamu.edu/spalermo/ecen689/2017_reconfigurable_16_32Gbps_NRZ_PAM4_SERDES_roshanzamir_jssc.pdf)]
>
> Hongtao Zhang, designcon2016. "PAM4 Signaling for 56G Serial Link Applications − A Tutorial"[[https://www.xilinx.com/publications/events/designcon/2016/slides-pam4signalingfor56gserial-zhang-designcon.pdf](https://www.xilinx.com/publications/events/designcon/2016/slides-pam4signalingfor56gserial-zhang-designcon.pdf)]
>
> Elad Alon, ISSCC 2014, "T6: Analog Front-End Design for Gb/s Wireline Receivers"






### BER with Quantization Noise

![image-20240804110522955](link/image-20240804110522955.png)




> $$
> \text{Var}(X) = E[X^2] - E[X]^2
> $$
>
> ![image-20240804110235178](link/image-20240804110235178.png)



### Impulse Response or Pulse Response

![image-20240807221637401](link/image-20240807221637401.png)

![image-20240807224407213](link/image-20240807224407213.png)![image-20240807224505987](link/image-20240807224505987.png)




###  TX FFE

TX FFE suffers from the peak power constraint, which in effect attenuates the average power of the outgoing signal -  the low-frequency signal content has been attenuated down to the high-frequency level

![image-20240727225120002](link/image-20240727225120002.png)

> [[https://www.signalintegrityjournal.com/articles/1228-feedforward-equalizer-location-study-for-high-speed-serial-systems](https://www.signalintegrityjournal.com/articles/1228-feedforward-equalizer-location-study-for-high-speed-serial-systems)]
>
> **S. Palermo**, "CMOS Nanoelectronics Analog and RF VLSI Circuits," [Chapter 9: High-Speed Serial I/O Design for Channel-Limited and Power-Constrained Systems](https://people.engr.tamu.edu/spalermo/docs/serial_links_chapter_palermo_2011.pdf), McGraw-Hill, 2011.



### Eye-Opening Monitor (EOM)

An architecture that evaluates the received signal quality



> data slicers, phase slicers, error slicers, scope slicers

![image-20240922143125270](link/image-20240922143125270.png)

![image-20240922144605196](link/image-20240922144605196.png)

> Analui, Behnam & Rylyakov, Alexander & Rylov, Sergey & Meghelli, Mounir & Hajimiri, Ali. (2006). A 10-Gb/s two-dimensional eye-opening monitor in 0.13-??m standard CMOS. Solid-State Circuits, IEEE Journal of. 40. 2689 - 2699, [[https://chic.caltech.edu/wp-content/uploads/2013/05/B-Analui_JSSC_10-Gbs_05.pdf](https://chic.caltech.edu/wp-content/uploads/2013/05/B-Analui_JSSC_10-Gbs_05.pdf)]





### reference

G. Balamurugan, A. Balankutty and C. -M. Hsu, "56G/112G Link Foundations Standards, Link Budgets & Models," *2019 IEEE Custom Integrated Circuits Conference (CICC)*, Austin, TX, USA, 2019, pp. 1-95 [[https://youtu.be/OABG3u2H2J4?si=CxryBSGbxrUpZNBT](https://youtu.be/OABG3u2H2J4?si=CxryBSGbxrUpZNBT)]

Paul Muller Yusuf Leblebici École Polytechnique Fédérale de Lausanne (EPFL). [Pattern generator model for jitter-tolerance simulation](https://designers-guide.org/modeling/JTOL_rev1.0.pdf); [VHDL-AMS models](https://designers-guide.org/modeling/fc_jtol_src_ns.vhd)

Savo Bajic, ECE1392, Integrated Circuits for Digital Communications: **StatOpt in Python** [[https://savobajic.ca/projects/academic/statopt](https://savobajic.ca/projects/academic/statopt/)]

Anritsu Company, "Measuring Channel Operating Margin," 2016. [[https://dl.cdn-anritsu.com/en-us/test-measurement/files/Technical-Notes/White-Paper/11410-00989A.pdf](https://dl.cdn-anritsu.com/en-us/test-measurement/files/Technical-Notes/White-Paper/11410-00989A.pdf)]

JLSD - Julia SerDe [[https://github.com/kevjzheng/JLSD](https://github.com/kevjzheng/JLSD)]

Kiran Gunnam, Selected Topics in RF, Analog and Mixed Signal Circuits and Systems

H. Shakiba, D. Tonietto and A. Sheikholeslami, "High-Speed Wireline Links-Part I: Modeling," in IEEE Open Journal of the Solid-State Circuits Society, vol. 4, pp. 97-109, 2024 [[https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10608184](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10608184)]

H. Shakiba, D. Tonietto and A. Sheikholeslami, "High-Speed Wireline Links-Part II: Optimization and Performance Assessment," in IEEE Open Journal of the Solid-State Circuits Society, vol. 4, pp. 110-121, 2024 [[https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10579874](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10579874)]


G. Souliotis, A. Tsimpos and S. Vlassis, "Phase Interpolator-Based Clock and Data Recovery With Jitter Optimization," in IEEE Open Journal of Circuits and Systems, vol. 4, pp. 203-217, 2023 [[https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10184121](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10184121)]


## loop dynamic

### Dithering Jitter in Bang-bang PLL

>  *hunting jitter* is also called as **dithering jitter**
>  the time error between *data clock* and input data
>
>  - proportional gain 
>  - loop latency 

![image-20240924225542342](link/image-20240924225542342.png)

where the proportional gain ($K_P$), *heavily damped systems* means that $K_P \gg K_I$



![image-20240924234154476](link/image-20240924234154476.png)



> Hanumolu, Pavan Kumar. 2006. *Design Techniques for Clocking High Performance Signaling Systems.* : Oregon State University. [https://ir.library.oregonstate.edu/concern/graduate_thesis_or_dissertations/1v53k219r](https://ir.library.oregonstate.edu/concern/graduate_thesis_or_dissertations/1v53k219r)]
>
> Hae-Chang Lee, "An Estimation Approach To Clock And Data Recovery" [[https://www-vlsi.stanford.edu/people/alum/pdf/0611_HaechangLee_Phase_Estimation.pdf](https://www-vlsi.stanford.edu/people/alum/pdf/0611_HaechangLee_Phase_Estimation.pdf)]
>
> R. Walker, “Designing Bang-Bang PLLs for Clock and Data Recovery in Serial Data Transmission Systems,” in Phase-Locking in High-Performance Systems, B. Razavi, Ed. New Jersey: IEEE Press, 2003, pp. 34-45. [[http://www.omnisterra.com/walker/pdfs.papers/BBPLL.pdf](http://www.omnisterra.com/walker/pdfs.papers/BBPLL.pdf)]
>
> J. Kim, Design of CMOS Adaptive-Supply Serial Links, Ph.D. Thesis, Stanford University, December 2002.  [[https://www-vlsi.stanford.edu/people/alum/pdf/0212_Kim_______Design_Of_CMOS_AdaptiveSu.pdf](https://www-vlsi.stanford.edu/people/alum/pdf/0212_Kim_______Design_Of_CMOS_AdaptiveSu.pdf)]
>
> P. K. Hanumolu, M. G. Kim, G. -y. Wei and U. -k. Moon, "A 1.6Gbps Digital Clock and Data Recovery Circuit," *IEEE Custom Integrated Circuits Conference 2006*, San Jose, CA, USA, 2006, pp. 603-606 [[https://sci-hub.se/10.1109/CICC.2006.320829](https://sci-hub.se/10.1109/CICC.2006.320829)]
>
> Da Dalt N. A design-oriented study of the nonlinear dynamics of digital bang-bang PLLs. IEEE Transactions on Circuits and Systems I: Regular Papers. 2005;52(1):21–31. [[https://sci-hub.se/10.1109/TCSI.2004.840089](https://sci-hub.se/10.1109/TCSI.2004.840089)]
>
> Jang S, Kim S, Chu SH, Jeong GS, Kim Y, Jeong DK. An optimum loop gain tracking all-digital PLL using autocorrelation of bang–bang phasefrequency detection. IEEE Transactions on Circuits and Systems II: Express Briefs. 2015;62(9):836–840. [[https://sci-hub.se/10.1109/TCSII.2015.2435691](https://sci-hub.se/10.1109/TCSII.2015.2435691)]

---



![ditheringjitter.drawio](link/ditheringjitter.drawio.svg)



> ![image-20240925213924764](link/image-20240925213924764.png)



### CDR Loop Latency 

Denoting the CDR loop latency by $\Delta T$ , we note that the loop transmission is multiplied by $exp(-s\Delta T)\simeq 1-s\Delta T$.The resulting right-half-plane zero, $f_z$ degrades the phase margin and must remain about **one decade beyond the BW**
$$
f_z\simeq \frac{1}{2\pi \Delta T}
$$


> This assumption is true in practice since the bandwidth of the CDR (few mega Hertz) is much smaller than the data rate (multi giga bits/second).

> Fernando , Marvell Italy."Considerations for CDR Bandwidth Proposal" [[https://www.ieee802.org/3/bs/public/16_03/debernardinis_3bs_01_0316.pdf](https://www.ieee802.org/3/bs/public/16_03/debernardinis_3bs_01_0316.pdf)]





### Loop Bandwidth

> The *closed-loop −3-dB bandwidth* is sometimes called the **“loop bandwidth”**





### Continuous-Time Approximation Limitations 

A rule of thumb often used to ensure slow changes in the loop is to select the *loop bandwidth* approximately equal to **one-tenth** of the *input frequency*. 

![image-20240806230158367](link/image-20240806230158367.png)

![image-20240928095850580](link/image-20240928095850580.png)



> Gardner, F.M. (1980). Charge-Pump Phase-Lock Loops. *IEEE Trans. Commun., 28*, 1849-1858.
>
> Homayoun, Aliakbar and Behzad Razavi. “On the Stability of Charge-Pump Phase-Locked Loops.” *IEEE Transactions on Circuits and Systems I: Regular Papers* 63 (2016): 741-750.
>
> N. Kuznetsov, A. Matveev, M. Yuldashev and R. Yuldashev, "Nonlinear Analysis of Charge-Pump Phase-Locked Loop: The Hold-In and Pull-In Ranges," in *IEEE Transactions on Circuits and Systems I: Regular Papers*, vol. 68, no. 10, pp. 4049-4061, Oct. 2021
>
> Deog-Kyoon Jeong, Topics in IC Design - 2.1 Introduction to Phase-Locked Loop [[https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%202%20-%20Charge-Pump%20PLL%2C%20Freuqency%20Synthesizers%2C%20and%20SSCG.pdf](https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%202%20-%20Charge-Pump%20PLL%2C%20Freuqency%20Synthesizers%2C%20and%20SSCG.pdf)]





### Limit Cycle Oscillation

> limit cycles imply self-sustained oscillators due *nonlinear nature*







> Ouzounov, S., Hegt, H., Van Roermund, A. (2007). SUB-HARMONIC LIMIT-CYCLE SIGMA-DELTA MODULATION, APPLIED TO AD CONVERSION. In: Van Roermund, A.H., Casier, H., Steyaert, M. (eds) Analog Circuit Design. Springer, Dordrecht. [[https://sci-hub.se/10.1007/1-4020-5186-7_6](https://sci-hub.se/10.1007/1-4020-5186-7_6)]



## BB PD

> It's **ternary**, because *early*, *late* and *no transition*


### Linearing BB-PD

BB Gain is the slope of average BB output $\mu$, versus phase offset $\phi$, i.e. $\frac {\partial \mu}{\partial \phi}$,

BB only produces output for a transition and this de-rates the gain. Transition density = *0.5* for random data

$$
K_{BB} = \frac{1}{2}\frac {\partial \mu}{\partial \phi}
$$

where $\mu = (1)\times \mathrm{P}(\text{late}|\phi) + (-1)\times \mathrm{P}(\text{early}|\phi)$

![bb-PDF.drawio](link/bb-PDF.drawio.svg)


> Both jitter and amplitude noise distribution are same, just scaled by slope 


### Self-Noise Term

One price we pay for *BB PD* versus *linear PD* is the self-noise term. For small phase errors BB output noise is the full
magnitude of the sliced data.

> BB-PD don't have any measure as to how early or how late and the way that tell loop is locked, is over a long time average, BB-PD have an equal number of earlies and lates 

$$\begin{align}
\sigma_{BB} &= [E(X^2) - E(X)^2] \cdot \mathrm{P}(\text{trans}) \\
&= [1 - 0]\cdot 0.5 \\
&= 0.5
\end{align}$$



> John T. Stonick, ISSCC 2011 TUTORIALS *T5: DPLL-Based Clock and Data Recovery*
>
> Walker, Richard. (2003). Designing Bang-Bang PLLs for Clock and Data Recovery in Serial Data Transmission Systems.  [[pdf](https://www.omnisterra.com/walker/pdfs.papers/BBPLL.pdf)]
>
> \- Clock and Data Recovery for Serial Data Communications, focusing on bang-bang CDR design methodology, ISSCC Short Course, February 2002. [[slides](https://www.omnisterra.com/walker/pdfs.talks/ISSCC2002.pdf)]



## Digital CDR Category

![image-20241024221619909](link/image-20241024221619909.png)

- DCO part is *analogous* so that it *cannot be perfectly modeled*
- Digital-to-phase converter is well-defined phase output, thus, very good to model real situation



## DCO

![image-20241024224500048](link/image-20241024224500048.png)

![image-20241024224603927](link/image-20241024224603927.png)



### limit cycle

![image-20241026230332655](link/image-20241026230332655.png)



### Z-domain modeling

![image-20241027001226490](link/image-20241027001226490.png)





The difference equation is
$$
\phi[n] = \phi[n-1] + K_{DCO}V_C[n]\cdot T\cdot2\pi
$$
z-transform is
$$
\frac{\Phi(z)}{V_C(z)}=\frac{2\pi K_{DCO}T}{1-z^{-1}}
$$


where $K_{DCO}$ : $\Delta f$ (Hz/bit)



## $\Delta \Sigma$-dithering in DCO









## Quantization noise

![image-20241019200102827](link/image-20241019200102827.png)

Here, $\alpha_T$ is data transition density

BBPD quantization noise

DAC quantization noise


> M. -J. Park and J. Kim, "Pseudo-Linear Analysis of Bang-Bang Controlled Timing Circuits," in IEEE Transactions on Circuits and Systems I: Regular Papers, vol. 60, no. 6, pp. 1381-1394, June 2013  [[https://sci-hub.st/10.1109/TCSI.2012.2220502](https://sci-hub.st/10.1109/TCSI.2012.2220502)]

## Time to Digital Converter (TDC)



##  Digital to Phase Converter (DPC)





## IIR low pass filter

![image-20241024232055792](link/image-20241024232055792.png)

simple approximation:
$$
z = 1 + sT
$$
bilinear-z transform
$$
z =\frac{}{}
$$




![image-20241024232111368](link/image-20241024232111368.png)




## Peak-to-peak jitter in ADPLL with BBPD

![image-20241025001015194](link/image-20241025001015194.png)







## Accumulate-and-dump (AAD) decimator

accumulating the input for $N$ cycles and then latching the result and resetting the integrator

![image-20241015222205883](link/image-20241015222205883.png)

> It adds up $N$ succeeding input samples at rate $1/T$ and delivers their sum in a *single* sample at the output. Therefore, the process comprises a **filter (in the accumulation)** and a **down-sampler (in the dump)**




## Moving Average and CIC Filters

> **cascade-integrator-comb (CIC)** decimator

*TODO* &#128197;





> An Intuitive Look at Moving Average and CIC Filters [[web](https://tomverbeure.github.io/2020/09/30/Moving-Average-and-CIC-Filters.html), [code](https://github.com/tomverbeure/pdm/tree/master/modeling/cic_filters)]
>
> A Beginner's Guide To Cascaded Integrator-Comb (CIC) Filters [[https://www.dsprelated.com/showarticle/1337.php](https://www.dsprelated.com/showarticle/1337.php)]





## Linearized Model



> *TODO* &#128197;
>
> Tristate: $\alpha=1$
>
> XOR: $\alpha=1$
>
> $\frac{1}{T}$ in Divider

![image-20240928004526381](link/image-20240928004526381.png)

![image-20240928004308700](link/image-20240928004308700.png)

> Michael H. Perrott, PLL Design Using the PLL Design Assistant Program. [[https://designers-guide.org/forum/Attachments/pll_manual.pdf](https://designers-guide.org/forum/Attachments/pll_manual.pdf)]



---

$\frac{1}{T}$ & $T$ come from *CT-DT* & *DT-CT*

![image-20240928203714450](link/image-20240928203714450.png)

> H. Kang *et al*., "A 42.7Gb/s Optical Receiver With Digital Clock and Data Recovery in 28nm CMOS," in *IEEE Access*, vol. 12, pp. 109900-109911, 2024  [[https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10630516](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10630516)]





## Majority Voter (MV)





> Y. Xia *et al*., "A 10-GHz Low-Power Serial Digital Majority Voter Based on Moving Accumulative Sign Filter in a PS-/PI-Based CDR," in *IEEE Transactions on Microwave Theory and Techniques*, vol. 68, no. 12 [[https://sci-hub.se/10.1109/TMTT.2020.3029188](https://sci-hub.se/10.1109/TMTT.2020.3029188)]
>
> J. Liang, A. Sheikholeslami, "On-Chip Jitter Measurement and Mitigation Techniques for Clock and Data Recovery Circuits" [[https://tspace.library.utoronto.ca/bitstream/1807/91138/3/Liang_Joshua_201706_PhD_thesis.pdf](https://tspace.library.utoronto.ca/bitstream/1807/91138/3/Liang_Joshua_201706_PhD_thesis.pdf)]
>
> J. Liang, A. Sheikholeslami. ISSCC2017. "A 28Gbps Digital CDR with Adaptive Loop Gain for Optimum Jitter Tolerance" [[slides](https://picture.iczhiku.com/resource/eetop/whiGpozAyuTUAnmb.pdf),[paper](https://www.eecg.utoronto.ca/~ali/papers/isscc2017-josh.pdf)]
>
> J. Liang, A. Sheikholeslami,, "Loop Gain Adaptation for Optimum Jitter Tolerance in Digital CDRs," in *IEEE Journal of Solid-State Circuits* [[https://sci-hub.se/10.1109/JSSC.2018.2839038](https://sci-hub.se/10.1109/JSSC.2018.2839038)]
>
> M. M. Khanghah, K. D. Sadeghipour, D. Kelly, C. Antony, P. Ossieur and P. D. Townsend, "A 7-Bit 7-GHz Multiphase Interpolator-Based DPC for CDR Applications," in *IEEE Transactions on Circuits and Systems I: Regular Papers* [[https://cora.ucc.ie/bitstreams/7ae5bfaa-8dd9-45a7-8276-99676b7b6078/download](https://cora.ucc.ie/bitstreams/7ae5bfaa-8dd9-45a7-8276-99676b7b6078/download)]





## FAQ

### PLL vs. CDR

| PLL                                  | CDR                                                          |
| ------------------------------------ | ------------------------------------------------------------ |
| Clock edge periodic                  | Data edge random                                             |
| Phase & Frequency detecting possible | Phase detecting possible , <br />Frequency detecting impossible |



> PLL or FD(Frequency Detector) for frequency detecting in CDR



## reference

J. Stonick. ISSCC 2011 "DPLL-Based Clock and Data Recovery" [[slides](https://www.nishanchettri.com/isscc-slides/2011%20ISSCC/TUTORIALS/ISSCC2011Visuals-T5.pdf),[transcript](https://www.nishanchettri.com/isscc-slides/2011%20ISSCC/TUTORIALS/Transcription_T5.pdf)]

P. Hanumolu. ISSCC 2015 "Clock and Data Recovery Architectures and Circuits" [[slides](https://www.nishanchettri.com/isscc-slides/2015%20ISSCC/TUTORIALS/ISSCC2015Visuals-T6.pdf)]

Amir Amirkhany. ISSCC 2019 "Basics of Clock and Data Recovery Circuits" 

Fulvio Spagna. INTEL, CICC2018, "Clock and Data Recovery Systems" [[slides](https://picture.iczhiku.com/resource/eetop/WhiTfzdJZSZyDcBM.pdf)]

M. Perrott. 6.976 High Speed Communication Circuits and Systems (lecture 21). Spring 2003. Massachusetts Institute of Technology: MIT OpenCourseWare, [[lec21.pdf](https://ocw.mit.edu/courses/6-976-high-speed-communication-circuits-and-systems-spring-2003/b396ec97fe1fc0eb7e39e88bf86c2979_lec21.pdf)]

Akihide Sai. ISSCC 2023, T5 "All Digital Plls From Fundamental Concepts To Future Trends" [[T5.pdf](https://www.nishanchettri.com/isscc-slides/2023%20ISSCC/TUTORIALS/T5.pdf)]

J. L. Sonntag and J. Stonick, "A Digital Clock and Data Recovery Architecture for Multi-Gigabit/s Binary Links," in *IEEE Journal of Solid-State Circuits*, vol. 41, no. 8, pp. 1867-1875, Aug. 2006  [[https://sci-hub.se/10.1109/JSSC.2006.875292](https://sci-hub.se/10.1109/JSSC.2006.875292)]

J. Sonntag and J. Stonick, "A digital clock and data recovery architecture for multi-gigabit/s binary links," *Proceedings of the IEEE 2005 Custom Integrated Circuits Conference, 2005.*. [[https://sci-hub.se/10.1109/CICC.2005.1568725](https://sci-hub.se/10.1109/CICC.2005.1568725)]

---

Liu, Tao, Tiejun Li, Fangxu Lv, Bin Liang, Xuqiang Zheng, Heming Wang, Miaomiao Wu, Dechao Lu, and Feng Zhao. 2021. "Analysis and Modeling of Mueller-Muller Clock and Data Recovery Circuits" *Electronics* 10, no. 16: 1888. https://doi.org/10.3390/electronics10161888

Gu, Youzhi & Feng, Xinjie & Chi, Runze & Chen, Yongzhen & Wu, Jiangfeng. (2022). Analysis of Mueller-Muller Clock and Data Recovery Circuits with a Linearized Model. 10.21203/rs.3.rs-1817774/v1. [[https://assets-eu.researchsquare.com/files/rs-1817774/v1_covered.pdf?c=1664188179](https://assets-eu.researchsquare.com/files/rs-1817774/v1_covered.pdf?c=1664188179)]

H. Kang et al., "A 42.7Gb/s Optical Receiver With Digital Clock and Data Recovery in 28nm CMOS," in IEEE Access, vol. 12, pp. 109900-109911, 2024 [[https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10630516](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10630516)]

Marinaci, Stefano. "Study of a Phase Locked Loop based Clock and Data Recovery Circuit for 2.5 Gbps data-rate" [[https://cds.cern.ch/record/2870334/files/CERN-THESIS-2023-147.pdf](https://cds.cern.ch/record/2870334/files/CERN-THESIS-2023-147.pdf)]

P. Palestri *et al*., "Analytical Modeling of Jitter in Bang-Bang CDR Circuits Featuring Phase Interpolation," in *IEEE Transactions on Very Large Scale Integration (VLSI) Systems*, vol. 29, no. 7, pp. 1392-1401, July 2021 [[https://sci-hub.se/10.1109/TVLSI.2021.3068450](https://sci-hub.se/10.1109/TVLSI.2021.3068450)]

F. M. Gardner, "Phaselock Techniques", 3rd Edition, Wiley Interscience, Hoboken, NJ, 2005 [[https://picture.iczhiku.com/resource/eetop/WyIgwGtkDSWGSxnm.pdf](https://picture.iczhiku.com/resource/eetop/WyIgwGtkDSWGSxnm.pdf)]

Rhee, W. (2020). *Phase-locked frequency generation and clocking : architectures and circuits for modern wireless and wireline systems*. The Institution of Engineering and Technology

M.H. Perrott, Y. Huang, R.T. Baird, B.W. Garlepp, D. Pastorello, E.T. King, Q. Yu, D.B. Kasha, P. Steiner, L. Zhang, J. Hein, B. Del Signore, "A 2.5 Gb/s Multi-Rate 0.25μm CMOS Clock and Data Recovery Circuit Utilizing a Hybrid Analog/Digital Loop Filter and All-Digital Referenceless Frequency Acquisition," IEEE J. Solid-State Circuits, vol. 41, Dec. 2006, pp. 2930-2944 [[https://cppsim.com/Publications/JNL/perrott_jssc06.pdf](https://cppsim.com/Publications/JNL/perrott_jssc06.pdf)]

M.H. Perrott. CICC 2009 "Tutorial on Digital Phase-Locked Loops" [[https://www.cppsim.com/PLL_Lectures/digital_pll_cicc_tutorial_perrott.pdf](https://www.cppsim.com/PLL_Lectures/digital_pll_cicc_tutorial_perrott.pdf)]

-, Short Course On Phase-Locked Loops and Their Applications Day 4, PM Lecture "Examples of Leveraging Digital Techniques in PLLs" [[https://www.cppsim.com/PLL_Lectures/day4_pm.pdf](https://www.cppsim.com/PLL_Lectures/day4_pm.pdf)]

-, Short Course On Phase-Locked Loops IEEE Circuit and System Society, San Diego, CA "Digital Frequency Synthesizers" [[https://www.cppsim.com/PLL_Lectures/digital_pll.pdf](https://www.cppsim.com/PLL_Lectures/digital_pll.pdf)]

---

Deog-Kyoon Jeong Topics in IC(Wireline Transceiver Design) - 3.1. Introduction to All Digital PLL [[https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%203%20-%20ADPLL.pdf](https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%203%20-%20ADPLL.pdf)]

Deog-Kyoon Jeong Topics in IC(Wireline Transceiver Design) - 6.1 Introduction to Clock and Data Recovery [[https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%206%20-%20Clock%20and%20Data%20Recovery.pdf](https://ocw.snu.ac.kr/sites/default/files/NOTE/Lec%206%20-%20Clock%20and%20Data%20Recovery.pdf)]

High-speed Serial Interface Lect. 16 – Clock and Data Recovery 3 [[http://tera.yonsei.ac.kr/class/2013_1_2/lecture/Lect16_CDR-3.pdf](http://tera.yonsei.ac.kr/class/2013_1_2/lecture/Lect16_CDR-3.pdf)]