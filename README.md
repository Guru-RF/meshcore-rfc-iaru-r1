# RFC: Proposal for a Coordinated 70 cm Amateur MeshCore / LoRa Data Channel in Belgium

| | |
| --- | --- |
| **Draft version** | 0.2 |
| **Prepared by** | Joeri Van Dooren, ON6URE / RF.Guru |
| **Intended audience** | Belgian repeater managers, unmanned-station coordinators, amateur radio associations, and technical working groups |
| **Status** | Request for comments |
| **Comment period** | 14 days from publication of this draft (via [GitHub Issues](https://github.com/Guru-RF/meshcore-rfc-be/issues)) |

---

## 1. Purpose of this RFC

This document proposes the coordinated use of a dedicated 70 cm amateur-radio frequency for ham-only MeshCore / LoRa-based digital repeater and mesh experiments in Belgium.

The goal is to obtain broad technical and organisational backing before wider deployment, so that the project remains compatible with existing 70 cm repeater usage, IARU Region 1 band planning, Belgian amateur-radio practice, and future discussions with BIPT/IBPT if needed.

This is not a request to replace existing repeater coordination. It is a proposal to avoid fragmented, uncoordinated use of 433 MHz and 868 MHz devices by licensed radio amateurs, to avoid conflict with non-ham users that use the ISM band on 70 cm, and to provide a cleaner amateur-radio alternative.

---

## 2. Background

MeshCore and similar LoRa-based mesh systems are becoming increasingly popular among radio amateurs. Many existing devices are designed for licence-free ISM/SRD use around 433 MHz or 868 MHz. That creates several problems when radio amateurs start using the same systems for ham-radio style communication:

1. The ISM/SRD bands are not amateur bands in the regulatory sense.
2. The ISM/SRD bands are shared with many non-amateur users.
3. ISM/SRD equipment is often subject to strict power, duty-cycle, channel-access, and equipment rules.
4. Radio amateurs may incorrectly assume that using a callsign or amateur purpose automatically makes ISM/SRD operation "amateur radio".
5. Many amateurs already transmit their callsign on 868 MHz (and on 433 MHz), which is explicitly not permitted: callsign identification is strictly an amateur-band operating practice and has no place on ISM/SRD spectrum. Adding a callsign to an ISM/SRD transmission does not make it amateur radio — it simply puts non-compliant identification onto a licence-free band.
6. Uncoordinated high-site LoRa nodes on 433 MHz or 868 MHz can create avoidable interference and regulatory confusion.
7. A fragmented situation may later create problems with BIPT/IBPT if amateur stations are seen to be operating in licence-free spectrum as if it were an amateur allocation.

A technical and regulatory background note was published by RF.Guru in April 2026:

- <https://shop.rf.guru/pages/meshtastic-meshcore-868-mhz-and-the-ham-radio-trap>

The main conclusion is that if radio amateurs want to build a ham-only mesh network, the cleaner solution is to do it inside the amateur service, under amateur-radio rules, with callsign identification, no encryption, coordinated frequencies, and responsible technical limits.

---

## 3. How LoRa and MeshCore work on the air

This section is a short primer for operators who are more familiar with FM, SSB, or CW than with spread-spectrum data modes. It explains why a whole mesh can share a single frequency without collapsing under its own traffic.

### 3.1 What LoRa is — chirp spread spectrum

LoRa ("Long Range") uses **Chirp Spread Spectrum (CSS)**. Instead of sending data on a fixed carrier, each symbol is a **chirp**: a tone whose frequency sweeps smoothly across the whole channel bandwidth (here 62.5 kHz) during the symbol. An "up-chirp" rises from the bottom of the channel to the top; a "down-chirp" does the reverse. It is, quite literally, the rising "chirp" of a bird, or an SSB operator slowly turning the VFO across the passband.

The information is carried in **where in the sweep each symbol starts** — a cyclic time/frequency shift of the basic chirp. The receiver already knows the exact shape of the sweep, so it "de-chirps" the incoming signal (multiplies it by a matching reference chirp). This collapses the wideband sweep back into a single steady tone, which a small FFT then reads out as one frequency bin = one data symbol.

### 3.2 Processing gain — hearing below the noise

Because each symbol's energy is spread across the entire bandwidth and then correlated back into one bin, the receiver enjoys **processing gain**. The familiar analogy is a CW operator using a narrow filter and a trained ear to pull a weak signal out of the noise: the wanted signal adds up coherently while the noise does not. This is why LoRa can decode signals **well below the noise floor** — roughly −120 dBm to −148 dBm depending on settings.

The **Spreading Factor (SF7…SF12)** sets how long each chirp lasts. A higher SF means longer, slower symbols: more processing gain, more range, better weak-signal performance — but a lower data rate. A lower SF is faster but needs a stronger signal. The European LoRa-APRS standard, for instance, uses SF12 (maximum range); a local MeshCore mesh will often use a lower SF for higher throughput over shorter hops.

### 3.3 Why a single shared frequency rarely collapses

A MeshCore network runs on **one frequency**, shared by every node. Newcomers often expect this to jam solid the moment several stations are active. In practice it holds up well, for several reasons:

1. **Capture effect.** When two signals overlap, the stronger one (by only a few dB) is still decoded and the weaker is rejected — the same capture effect FM operators know. A collision is therefore often _not_ a total loss.
2. **Quasi-orthogonal spreading factors.** Transmissions using different spreading factors on the same frequency are nearly orthogonal and can be received at the same time.
3. **Interference rejection.** Narrowband carriers and short bursts get spread out by the de-chirp process and are largely rejected, so the link tolerates a noisy band.
4. **Short, infrequent packets.** Messaging traffic is small and bursty. Airtime per packet is short and idle time is long, so the chance of two nodes keying at the exact same instant is low.
5. **Polite protocol behaviour.** MeshCore uses managed flooding with hop limits, randomised transmit delays, duplicate suppression, and — where the hardware supports it — listen-before-talk via Channel Activity Detection (CAD). This spaces transmissions out and prevents retransmission storms.

This is **not** collision-free. Very high traffic, very many nodes, or everyone crowding onto the same spreading factor can still cause congestion. That is exactly why power, duty cycle, node density, and coordination matter — see [Section 9](#9-power-proposal) and [Section 12](#12-proposed-initial-frequency-plan).

### 3.4 What a MeshCore "repeater" actually is

Unlike an FM repeater, a MeshCore repeater is **not** a split input/output pair. Every relaying node listens and re-transmits on the _same_ coordinated frequency, forwarding (flooding) packets so a message hops across the area from node to node. A MeshCore "repeater" is therefore a store-and-forward relay on the shared channel. This is why this RFC proposes a single channel rather than a repeater input/output pair — and why every fixed relay node is, in the regulatory sense, an unmanned station (see [Section 10](#10-licensing-and-unmanned-station-status)).

### 3.5 Signal bandwidth and the 62.5 kHz choice

LoRa channel bandwidth (commonly 62.5, 125, 250, or 500 kHz) is a deliberate trade-off, and it interacts directly with the congestion behaviour above:

- **Wider bandwidth shortens each symbol**, so the same message is sent _faster_, with _less time on air_. Shorter transmissions occupy the channel for briefer bursts, so the probability of two nodes overlapping drops and the mesh tolerates more traffic before it congests ("collapses"). In the **time** domain, wider is friendlier and frees the channel sooner.
- **But wider bandwidth occupies more spectrum.** Doubling the bandwidth doubles the _frequency_ width of every transmission, leaving room for fewer non-overlapping channels and increasing the chance of overlap with neighbouring users. In the **frequency** domain, wider is greedier.
- **Wider bandwidth also costs sensitivity.** Roughly speaking, doubling the bandwidth doubles the noise the receiver takes in (about 3 dB), so for a given spreading factor a wider link reaches _less far_.

So the choice balances airtime (favours wider) against spectrum footprint and range (favour narrower).

This RFC adopts MeshCore's **stock Europe profile, which uses 62.5 kHz** (see [Section 5.1](#51-on-air-radio-settings-the-testable-mode)). That deliberately favours range, weak-signal performance, and a small spectrum footprint over raw speed — appropriate for a low-rate messaging and emergency-fallback layer — and it keeps every node on the same settings as the wider MeshCore ecosystem. It fits inside the 175 kHz digital communication link segment (430.400–430.575 MHz): 62.5 kHz centred on 430.475 MHz occupies only ≈ 430.444–430.506 MHz, sitting low in the segment with roughly 34 kHz of guard to the ON0TVM packet channel (430.400 MHz) below and 38 kHz to the digital repeater-input extension (430.544 MHz) above. (For comparison, a 250 kHz channel — ≈ 430.350–430.600 MHz — would not fit at all: it would spill into the NBFM repeater-output block below and across the digital repeater-input edge above.) One honest caveat remains: 62.5 kHz is wider than the 20 kHz maximum channel bandwidth the IARU R1 plan lists for this sub-band, so the channel spans roughly three nominal 20 kHz slots — a point addressed openly in [Section 12.1](#121-frequency-selection-rationale).

---

## 4. Proposed operating concept

The proposed system is a Belgian ham-only MeshCore / LoRa digital mesh layer using the 70 cm amateur band.

**The intended use cases are:**

- low-speed amateur digital messaging;
- emergency and fallback communication experiments;
- RF propagation and network resilience experiments;
- ham-only telemetry and status messages;
- portable and fixed station interconnection;
- experimentation with decentralised digital infrastructure.

**The system is _not_ intended as:**

- a replacement for commercial mobile networks;
- a general public messaging system;
- encrypted private communication;
- an ISM/SRD network with amateur callsigns added afterwards;
- an uncontrolled high-power wide-area data network.

---

## 5. Proposed frequency and on-air mode

The proposed primary centre frequency is:

> **430.475 MHz**

With **62.5 kHz** occupied bandwidth, a 430.475 MHz centre frequency occupies approximately:

> **430.44375 MHz to 430.50625 MHz**

This keeps the signal low in the 430.400–430.575 MHz digital communication **link** segment of the IARU Region 1 70 cm band plan. The centre is deliberately placed toward the bottom of that segment so that its upper edge (≈ 430.506 MHz) stays clear of the **digital repeater-input extension that begins at 430.544 MHz** (the 7.6 MHz-shift digital-repeater input, paired with 438.194–438.531 MHz outputs), while its lower edge (≈ 430.444 MHz) stays above the FM repeater-output block below 430.375 MHz and clear of the ON0TVM packet channel at 430.400 MHz. The resulting guards are roughly 38 kHz to the repeater-input edge above and 34 kHz to ON0TVM below. See [Section 12.1](#121-frequency-selection-rationale) for why 430.475 MHz was chosen over 430.500 MHz and over the other candidates that were considered.

The proposed frequency also avoids the commonly used 433 MHz area, which is heavily affected by ISM/SRD devices, remote controls, sensors, weather stations, consumer LoRa devices, and uncoordinated Meshtastic-type nodes.

### 5.1 On-air radio settings (the testable mode)

So that anyone can join the channel and actually decode it, the network uses a single, explicitly defined mode. It is **MeshCore's stock Europe profile with only the frequency changed to 430.475 MHz** — existing MeshCore users need only retune. All nodes must match every parameter below; a mismatch in any one of them (bandwidth, spreading factor, coding rate) means stations cannot hear each other.

| Parameter | Value |
| --- | --- |
| Modulation | LoRa (Chirp Spread Spectrum) |
| Firmware | MeshCore (companion or repeater role) |
| Centre frequency | **430.475 MHz** |
| Bandwidth | **62.5 kHz** |
| Spreading factor | **SF8** |
| Coding rate | **4/8** (shown as `8` in the MeshCore apps) |
| Occupied bandwidth | ≈ 430.44375 – 430.50625 MHz |
| TX power | MeshCore default 22 dBm (≈ 158 mW) for testing; RFC maximum **2 W** conducted at fixed sites (see [Section 9](#9-power-proposal)) |
| Channel / key | Default **public** MeshCore channel with its published shared key — do **not** configure private/secret keys (no encryption for amateur traffic — see [Section 11](#11-technical-requirements-for-participating-repeaters)) |

This is a deliberately long-range, low-rate, narrow profile (see the bandwidth trade-off in [Section 3.5](#35-signal-bandwidth-and-the-625-khz-choice)), which suits a messaging and emergency-fallback layer and keeps the spectrum footprint small.

---

## 6. Compatibility with existing 70 cm repeaters

The proposed 430.475 MHz centre frequency is deliberately chosen to avoid existing 70 cm FM and digital repeater input and output channels, and the point-to-point digital-link and packet channels around it.

Nearby fixed stations and band-plan edges (from the ON0 unmanned-station database and the IARU R1 / UBA 70 cm plan):

- **FM repeater-output block:** 430.025–430.375 MHz — ≈ 69 kHz below the channel's lower edge
- **ON0TVM:** 430.40000 MHz (packet) — ≈ 34 kHz below the channel's lower edge
- **Digital repeater-input extension (7.6 MHz shift):** from 430.544 MHz — ≈ 38 kHz above the channel's upper edge
- **ON0SEA:** 430.57500 MHz (packet) — ≈ 59 kHz above the channel's upper edge

A 430.475 MHz LoRa signal with 62.5 kHz bandwidth occupies ≈ 430.444–430.506 MHz. Its lower edge stays well above the 430.025–430.375 MHz repeater-output block, and — importantly — its upper edge stays below the 430.544 MHz digital repeater-input edge, so the channel does not crowd the repeater-input side of the band. That was the concern that prompted moving down from the original 430.500 MHz proposal; the reasoning is recorded in [Section 12.1](#121-frequency-selection-rationale).

This proposal therefore aims to avoid both:

- direct channel overlap with existing repeaters, links, and packet stations;
- future claims that MeshCore activity has occupied repeater input or output channels.

Where a MeshCore node is installed at an existing repeater site, co-location filtering is addressed in [Section 11.1](#111-co-siting-at-existing-repeater-sites).

---

## 7. Why not use 433 MHz ISM/SRD?

Although 433 MHz equipment is widely available, the 433 MHz area is not a clean solution for a Belgian ham-only network.

Reasons:

1. 433 MHz is heavily used by non-amateur ISM/SRD devices.
2. The band is shared with consumer devices that cannot coordinate with amateur stations.
3. Many devices use unknown duty cycles, unknown protocols, and low-cost RF stages.
4. It is difficult to distinguish amateur use from non-amateur ISM/SRD use.
5. The standard European LoRa-APRS frequency, **433.775 MHz** (125 kHz, SF12), is already in active use, and Europe has the densest LoRa-APRS network in the world. A MeshCore deployment in the 433 MHz area would conflict directly with this existing traffic.
6. Long-range high-site amateur nodes may create conflicts with licence-free users.
7. Some parts of the 433 MHz area overlap with IARU Region 1 repeater input/output planning.
8. Using 433 MHz for ham-only infrastructure risks creating confusion with BIPT/IBPT if radio amateurs appear to be using ISM/SRD spectrum as an amateur allocation.

The LoRa-APRS conflict deserves particular attention. Most LoRa-APRS stations are iGates (which receive permanently) and position trackers; some also relay APRS text messages. All of these would be disrupted by, or interfere with, a co-channel MeshCore network. These LoRa-APRS devices generally operate at low power (typically below 0.5 W / ~500 mW) and with a much lower duty cycle than a typical MeshCore mesh, so a chattier MeshCore deployment would be a disproportionate interferer to the existing LoRa-APRS users.

A dedicated amateur-band frequency provides a cleaner and more defensible solution.

---

## 8. Why not use 868 MHz ISM/SRD?

The 868 MHz SRD band is useful for licence-free short-range devices, but it is not ideal for a ham-only repeater or mesh backbone.

Reasons:

1. 868 MHz is not a Belgian amateur allocation.
2. The band is governed by SRD/ISM rules, not amateur-radio rules.
3. Duty-cycle restrictions and equipment limits may apply.
4. Higher-gain antennas and high-site infrastructure can create regulatory ambiguity.
5. Amateur callsign identification does not automatically convert SRD operation into amateur-radio operation. On the contrary, transmitting a callsign on 868 MHz is explicitly forbidden — callsign usage belongs strictly on amateur bands, not on ISM/SRD spectrum.
6. Encryption, private messaging, and consumer-device behaviour may conflict with amateur-radio expectations.
7. A growing number of uncoordinated 868 MHz nodes could later create avoidable issues with BIPT/IBPT.

For these reasons, this RFC proposes that ham-only infrastructure should be moved into the amateur 70 cm band, where amateur-radio identification, coordination, and operating practice can be applied properly.

---

## 9. Power proposal

The proposed RF power policy is intentionally conservative. LoRa is a high-sensitivity, low-data-rate mode: coverage comes from siting, antenna height, and network density far more than from raw transmitter power.

### 9.1 Normal recommended output power

> **1-2 W conducted RF output** at the repeater module.

This should be sufficient for most fixed MeshCore repeater sites when combined with:

- a good antenna location;
- reasonable antenna gain;
- low-loss feedline;
- good filtering;
- proper receiver sensitivity;
- careful site selection.

For LoRa-style signals, coverage is often improved more effectively by antenna height, siting, and network density than by simply increasing transmitter power.

### 9.2 Maximum conducted output power

> **2 W conducted RF output maximum**, only where technically justified.

The 2 W value should be treated as an upper limit, not as the normal target. It may be useful for difficult links, lower antenna sites, or specific coordinated backbone paths, but should not be the default for every installation.

### 9.3 ERP/EIRP consideration

Because antenna gain can vary significantly between sites, the network should not focus only on transmitter-module power. Effective radiated power must also be considered.

A repeater running 1 W into a high-gain antenna from a good site may already outperform a poor 2 W installation. Therefore, every fixed repeater site should document at least:

- conducted RF output power;
- antenna gain;
- feedline loss;
- estimated ERP or EIRP;
- antenna height;
- site locator;
- responsible callsign/contact.

**Antenna gain — a modest (≈ 0 dBd) omni is often the better default.** A high-gain collinear is not automatically the right choice for a fixed node. From a high site — which repeater masts usually are — a high-gain antenna concentrates energy at the horizon and can overshoot nearby users, leaving a coverage hole below it; a roughly unity-gain omni (≈ 0 dBd, i.e. ~2.15 dBi) has a rounder pattern that fills in local and valley coverage and often works better in practice from a tall mast. Modest gain also keeps ERP modest, which suits a low-impact mesh node. So ≈ 0 dBd is a reasonable default for most fixed nodes, with higher gain justified by a specific coverage need and its ERP documented above. One caveat: antenna gain is _not_ a co-site desensitisation control (see [Section 11.1](#111-co-siting-at-existing-repeater-sites)) — on a vertically-stacked mast a low-gain antenna can couple _more_ energy into a neighbouring receiver than a high-gain collinear (whose pattern nulls point up and down), so co-site isolation comes from antenna separation, filtering and conducted power, not from reducing gain.

### 9.4 Recommended principle

> **Use the lowest power that provides reliable coverage.**

The network should be engineered as a coordinated amateur digital network, not as a maximum-power contest between nodes.

---

## 10. Licensing and unmanned-station status

A MeshCore mesh is built from relay/repeater nodes. Any fixed node that retransmits traffic automatically and operates without an operator present is an **unmanned (automatically controlled) amateur station** — a repeater in the regulatory sense — not merely a station operated under a personal licence.

Key consequences:

1. **A personal amateur licence is not sufficient.** A personal amateur licence (e.g. HAREC) authorises _you_ to operate a station under your direct control. It does not, on its own, authorise an unattended automatic relay. Every fixed MeshCore repeater/relay node therefore requires a separate **unmanned-station / repeater authorisation** — in Belgium, from BIPT/IBPT — before it may operate.
2. **The mesh is single-frequency and mesh-based.** Because MeshCore is mesh-based, every relaying node receives and re-transmits on the _same_ coordinated frequency; there is no separate input/output pair as on an FM repeater. All licensed nodes share one channel — see [Section 3](#3-how-lora-and-meshcore-work-on-the-air).
3. **Register every licensed node.** Each authorised unmanned node and its responsible operator should be registered on the overarching ON0.be platform — see [Section 13](#13-coordination-proposal).
4. **Attended portable/handheld nodes** operated under the direct control of a licensed amateur are not unmanned stations and fall under the normal personal licence. However, a portable node configured to relay automatically takes on repeater behaviour and should be treated accordingly.

This requirement is a feature, not a burden: routing MeshCore traffic through properly licensed unmanned stations on a coordinated amateur frequency is exactly what keeps the network inside the amateur service and out of the ISM "ham trap".

---

## 11. Technical requirements for participating repeaters

Participating fixed repeater or backbone nodes should meet the following minimum technical expectations:

1. A valid unmanned-station / repeater authorisation for the node (see [Section 10](#10-licensing-and-unmanned-station-status)).
2. Operation on the coordinated frequency or frequency set.
3. Callsign identification according to amateur-radio requirements.
4. No encryption for amateur-radio traffic.
5. Clean transmitter spectrum.
6. No overdriven RF power stages.
7. Suitable filtering where needed.
8. Stable frequency reference appropriate for the modulation used.
9. Documented antenna, power, location, and responsible operator.
10. Remote shutdown possibility for unattended stations where practical.
11. Willingness to reduce power or change configuration if interference is reported.
12. Registration of the node and its responsible operator on the overarching unmanned-station platform — see [Section 13](#13-coordination-proposal) and <https://on0.be>.

### 11.1 Co-siting at existing repeater sites

Where a MeshCore node is installed on the same mast or site as an existing repeater, the node's transmitter can degrade the repeater's receiver (and, less often, the reverse). Coordinating the frequency is the first line of defence — 430.475 MHz is deliberately placed as far from the digital repeater-input edge as the segment allows (see [Section 12.1](#121-frequency-selection-rationale)) — but at a shared site additional filtering is usually still needed. Two **distinct** mechanisms are involved, and they need **different** fixes:

1. **Transmitter noise falling in the repeater's passband.** Every transmitter emits low-level wideband noise on either side of its carrier. Any of that noise landing on the repeater's _receive_ frequency is indistinguishable from a wanted signal there and **cannot be removed by any filter at the repeater**. It must be cleaned at the source — with a band-pass cavity / filter on the MeshCore transmitter output, and by using a clean PA that is not overdriven. This is why transmitter cleanliness matters more than raw power.
2. **Receiver blocking / desensitisation.** The strong nearby MeshCore _carrier_ itself (off the repeater's frequency) can overload the repeater's front-end and raise its noise floor. This is what a **band-reject (notch) filter or a band-pass preselector at the repeater's receive input** can reduce — provided there is enough frequency separation for the filter's skirts.

A notch filter in the repeater's RX chain therefore addresses **mechanism 2 only**. It is a useful and often necessary component, but on its own it is **not a complete answer**, because it does nothing about mechanism 1 (you cannot notch out energy sitting on your own wanted frequency). The robust approach is layered, roughly in order of effectiveness and cost:

- **Antenna separation** (especially vertical spacing) — usually the cheapest large amount of isolation;
- a **band-pass filter / cavity on the MeshCore transmitter output** — kills transmitter wideband noise in the repeater's receive passband (mechanism 1);
- a **band-pass preselector on the repeater receiver** — rejects the off-channel MeshCore carrier and other on-site signals broadly (mechanism 2);
- a **notch tuned to the MeshCore channel** — a targeted supplement to the preselector when one specific carrier dominates;
- the node's inherently **low power (1–2 W) and low duty cycle**, which already limit the severity.

The further the MeshCore channel sits from the co-located repeater's input, the wider the filter skirts can be and the cheaper every one of these measures becomes — a further reason for the low-in-segment channel choice. Each co-located installation should document its co-siting arrangement (antenna separation, TX filtering, RX filtering) alongside the technical parameters required in [Section 9.3](#93-erpeirp-consideration) and the list above.

**Practical guidance by offset from the repeater input.** Because the node runs only **1 W (≈ +30 dBm)**, co-siting is far easier than for a normal high-power repeater — but the receive-side numbers still bite. With typical on-mast antenna isolation of 40–60 dB (vertical separation helps most), the node's carrier still arrives at roughly −10 to −20 dBm at the repeater's front end — more than enough to desensitise a sensitive receiver. The aim is to add enough rejection to bring that carrier down to about −40 dBm or lower at the receiver input; a cavity notch tuned to the node frequency supplies it, **provided the offset Δf between the node channel and the repeater input is large enough for the notch skirts to clear the wanted signal.** As a rough rule of thumb (always measure on site):

| Offset Δf (node channel ↔ repeater input) | Same-site outlook at 1 W |
| --- | --- |
| **< ~0.1 MHz** | Very difficult. A notch deep enough to protect the input would also attenuate the wanted receive signal. Not recommended on a shared antenna system — use large vertical antenna separation, a separate site, or avoid co-siting with an input this close. |
| **~0.1–0.5 MHz** | Feasible but demanding: needs one or two **high-Q cavity notches** at the node frequency on the repeater RX line, good antenna separation, and a clean / band-pass-filtered node TX. Tune and measure. |
| **~0.5–2 MHz** | Comfortable: a single good **cavity notch** at the node frequency on the repeater RX, plus antenna separation, is normally enough. |
| **> ~2 MHz** | Usually fine: the repeater's existing duplexer / front-end and antenna separation generally suffice; just keep the node TX clean. |

For the proposed 430.475 MHz channel on a shared site, the closest input you would realistically meet is a **digital repeater input at 430.544–430.925 MHz** (Δf ≈ 0.07–0.45 MHz — the hard end of the table, needing high-Q cavity notching or better antenna separation), whereas the **FM/DV repeater inputs at 431.050–431.975 MHz** (Δf ≈ 0.58–1.5 MHz) fall in the comfortable range. Note the notch protects only against receiver **blocking**; the node's own transmitter **noise** landing on the repeater's input frequency cannot be notched at the repeater and must be handled by the band-pass filter on the node TX (mechanism 1 above) — which is itself least effective at very small Δf, another reason the sub-0.1 MHz case is best avoided.

**Turning the node's power down is often the simplest fix — and the only one that also cures TX noise.** Every dB of node output removed reduces both the blocking carrier and the transmitter noise reaching the repeater by the same dB, so dropping a co-sited node from 1 W to, say, 100 mW (−10 dB) or 10 mW (−20 dB) can do more, more cheaply, than a cavity — and it is the only measure that also helps the un-notchable TX-noise path (mechanism 1). Because a MeshCore relay's coverage comes mainly from siting and antenna height rather than raw power (see [Section 9](#9-power-proposal)), a node on a good repeater mast can usually afford to run low power and still cover well. At shared sites, therefore, use the lowest power that still gives reliable mesh coverage — a co-located node will often need far less than the general 1–2 W.

**Spreading is not a substitute for this.** It is tempting to assume LoRa's chirp spreading makes the node harmless: its 1 W is smeared across 62.5 kHz, so its power spectral density is low, and a narrowband repeater receiver only catches the fraction of that energy that falls in its own passband (a 12 kHz FM receiver sees roughly a fifth — about 7 dB less — of overlapping in-band energy than it would from an equal-power narrowband source). That genuinely makes the node a gentler co-channel / adjacent-channel neighbour, and — together with LoRa's processing gain — it also makes the node itself far more tolerant of a noisy band (see [Section 3.2](#32-processing-gain--hearing-below-the-noise) and [Section 3.3](#33-why-a-single-shared-frequency-rarely-collapses)). But spreading does **not** reduce front-end **blocking / desensitisation**: a receiver's front end is wideband and sees the node's _total_ carrier power regardless of how it is spread, so the −10 to −20 dBm figures above are unchanged by the spreading factor. Blocking is defeated only by frequency separation, filtering, antenna isolation and lower power — not by spreading.

---

## 12. Proposed initial frequency plan

Initial proposal:

| Use | Centre frequency | Bandwidth | Notes |
| --- | --- | --- | --- |
| Primary Belgian ham-only MeshCore channel | **430.475 MHz** | 62.5 kHz | Preferred starting point; low in the 430.400–430.575 digital-link segment |
| Alternative test channel | 430.4875 MHz | 62.5 kHz | Only if primary is not usable locally; slightly higher, smaller guard to the 430.544 MHz input edge |

**Frequencies to avoid for this project:**

| Range / frequency | Reason |
| --- | --- |
| 430.025–430.375 MHz | FM repeater-output block (1.6 MHz-shift system) |
| 430.544–430.931 MHz | Digital repeater-input extension (7.6 MHz-shift system; outputs 438.194–438.531 MHz) |
| 431.050–431.825 MHz | FM/DV repeater inputs (7.6 MHz-shift system) |
| 431.625–431.975 MHz | FM repeater inputs (1.6 MHz-shift system) |
| 432.000–432.400 MHz | Narrow-band weak-signal segment (CW / SSB / EME / MGM) — do not put wideband data here |
| 432.400–432.490 MHz | Beacons (exclusive) |
| 432.500 MHz | APRS |
| 433.000–433.375 MHz | Repeater inputs (Region 1 standard) |
| 433.050–434.790 MHz | 70 cm ISM / LPD433 band — the licence-free spectrum this project exists to leave (includes 433.450 DV calling, 433.500 FM calling, 433.775 LoRa-APRS, and the 434.450 MHz digital sub-band) |
| 434.600–434.975 MHz | Repeater outputs |
| 435.000–438.000 MHz | Amateur satellite segment |
| 438.650–439.425 MHz | FM/DV repeater outputs (7.6 MHz-shift system) |
| 439.800–439.975 MHz | Digital-link channels, but already occupied (ON0BAF / ON0AN / ON0ANR) and immediately below the DAPNET paging transmitter |
| 439.9875 MHz | DAPNET / POCSAG paging (high-power, high-duty, transmit-only) |

### 12.1 Frequency selection rationale

The primary channel was refined from **430.500 MHz** (draft 0.1) to **430.475 MHz** (draft 0.2) after review comments that 430.500 MHz sits too close to the repeater-input side of the band, together with suggestions to weigh other placements — notably 434.450 MHz and the region above the repeater outputs. This section records why 430.475 MHz was chosen and why each alternative was set aside, so the trade-offs are on the record for further comment.

**The constraint.** In the lower 70 cm band the only segment designated for coordinated point-to-point / mesh _digital-link_ traffic that is not a repeater input, a repeater output, a beacon, a weak-signal (CW/SSB/EME) segment, the satellite segment, or licence-free ISM spectrum is the **430.400–430.575 MHz digital communication link segment**. That is the natural and defensible home for this channel. The segment is only 175 kHz wide, its maximum channel bandwidth is 20 kHz, and it is bracketed by two existing packet stations — **ON0TVM (430.400 MHz)** at the bottom and **ON0SEA (430.575 MHz)** at the top — with the **7.6 MHz-shift digital repeater-input extension beginning at 430.544 MHz** just above it.

**Why 430.475 MHz.** Centred at 430.475 MHz the 62.5 kHz channel occupies ≈ 430.444–430.506 MHz — the "maximin" position on the 12.5 kHz raster, i.e. the centre that maximises the smaller of its two guards inside the clear interior of the segment: ≈ 34 kHz to ON0TVM below and ≈ 38 kHz to the 430.544 MHz repeater-input edge above. It stays entirely within the digital-link segment, clear of the repeater-output block (top 430.375 MHz, ≈ 69 kHz below) and well clear of the FM/DV repeater inputs at 431.050 MHz (≈ 544 kHz above).

**Why not 430.500 MHz (draft 0.1).** Legal and workable, but skewed high in the segment: its upper edge (430.531 MHz) is only ≈ 13 kHz below the 430.544 MHz digital repeater-input edge and ≈ 34 kHz from ON0SEA, while wasting ≈ 59 kHz of unused guard on the low side. Moving down 25 kHz to 430.475 MHz roughly triples the guard to the repeater-input edge (13 → 38 kHz) at no cost. The 25 kHz shift makes little difference to raw propagation between distant stations, but it removes the "you are crowding the repeater input" objection and makes any co-site filtering easier — and it is free (same segment, same retune, same raster).

**Why not 434.450 MHz.** The IARU plan does list a 434.450–434.575 MHz "digital communications" sub-band, and it is empty of ON0 stations — but that is precisely because **434.450 MHz lies inside the 433.050–434.790 MHz ISM / LPD433 band**. Placing the channel there would put it back among the licence-free remote controls, sensors, LPD handhelds and consumer LoRa / Meshtastic devices that Sections [7](#7-why-not-use-433-mhz-ismsrd) and [8](#8-why-not-use-868-mhz-ismsrd) give as the whole reason for leaving 433 MHz. It also carries a tighter 12 kHz channelisation and sits directly below the 435 MHz satellite segment. Choosing it would contradict the central argument of this RFC, so it is rejected.

**Why not "above the repeater outputs" (e.g. 439.700–439.800 MHz).** This region _is_ far from every repeater input, which is attractive from a "do not desensitise anyone's receiver" standpoint. But it is a poor home in practice: (a) the designated digital-link channels there (439.800–439.975 MHz) are already occupied by ON0BAF (439.850), ON0AN (439.900) and ON0ANR (439.950), leaving no clean 62.5 kHz gap; (b) the whole area sits immediately below **DAPNET / POCSAG paging at 439.9875 MHz** — a high-power, high-duty, transmit-only network that would continuously desensitise the sensitive MeshCore receivers (the interference runs _into_ the mesh, and cannot be answered because DAPNET never listens); and (c) the spectrum just below (438.650–439.425 MHz) is the dense FM/DV repeater-output forest, so a receiver there is hammered by strong local outputs. A frequency in the undesignated gap at ≈ 439.700 MHz would fit and avoid the inputs, but it is not a designated digital segment and still self-desenses against the repeater outputs — a worse overall trade than 430.475 MHz.

**Why not the weak-signal, beacon or satellite segments.** 432.000–432.400 MHz is the narrow-band CW/SSB/EME weak-signal segment and 432.400–432.490 MHz is exclusive to beacons; wideband data there would be highly disruptive and is out of the question. 435.000–438.000 MHz is the amateur-satellite segment. All are excluded.

**Channelisation caveat (applies to any centre in this sub-band).** The 62.5 kHz occupied bandwidth of the MeshCore stock Europe profile is wider than the 20 kHz maximum channel bandwidth the IARU R1 plan lists for the 430.000–431.975 MHz sub-band, so the channel spans roughly three nominal 20 kHz slots. This is independent of the exact centre frequency and cannot be resolved by moving frequency — only by narrowing the channel (e.g. a ≤ 20 kHz LoRa profile) or by obtaining explicit national / sub-regional coordination for the wider occupied bandwidth. It is raised here openly so it can be settled with UBA / BIPT rather than surfacing later as a compliance objection. See open question 2 in [Section 16](#16-open-questions-for-comments).

---

## 13. Coordination proposal

This RFC proposes that Belgian associations and repeater managers agree on a common starting point before large-scale deployment.

Coordination, registration, and maintenance of the known-node list are proposed to run through **[ON0.be](https://on0.be)**, the overarching platform for managing unmanned stations in Belgium. Using a single overarching platform keeps the node list, responsible-operator contacts, and technical parameters in one authoritative, publicly visible place.

> **Initial consultation:** Rudy (ON6VDS) has already informally consulted several of our neighbouring amateur colleagues about the proposed channel in the 430.400–430.575 MHz digital-link segment (initially 430.500 MHz; now refined to 430.475 MHz — a 25 kHz move within the same segment, so the earlier consultation still holds), and none of them raised any objection.

**Suggested coordination steps:**

1. Circulate this RFC among repeater managers and technical coordinators.
2. Collect objections, local conflicts, and alternative proposals.
3. Confirm whether 430.475 MHz is acceptable as a national or semi-national starting point.
4. Define a list of known fixed nodes and responsible operators, maintained on the overarching ON0.be platform.
5. Start with low-power test deployments.
6. Monitor for interference or complaints.
7. Adjust the frequency or power policy if needed.
8. Document the final recommendation and make it publicly available to Belgian radio amateurs.
9. If appropriate, use the coordinated amateur-radio position in future communication with BIPT/IBPT.

### 13.1 Comment period and next steps

1. **Comment period — 14 days.** Comments, objections, and reports of local conflicts are invited for **14 days** from publication of this draft, via GitHub Issues (see [Section 15](#15-how-to-comment-and-contribute)).
2. **Request test-site licences.** From the close of the comment period onward, we will apply for a small number of **unmanned-station (repeater) authorisations** for initial test sites (see [Section 10](#10-licensing-and-unmanned-station-status)).
3. **Deploy and evaluate.** Once the test sites are licensed and on the air, the project will be evaluated on how it performs and evolves in practice — coverage, interference, traffic and congestion behaviour, and node density — and this RFC will be revised accordingly before any wider roll-out.

### 13.2 Initial test-bed nodes

The following stations are planned to form the initial test bed, operating on the proposed **430.475 MHz** channel with the on-air mode defined in [Section 5.1](#51-on-air-radio-settings-the-testable-mode):

- ON0BXL
- ON0VDS
- ON0ORA
- ON0AND
- ON0OST
- ON0UHF
- ON0VHF

Each will be brought up as a licensed unmanned station (see [Section 10](#10-licensing-and-unmanned-station-status)) and registered on the overarching [ON0.be](https://on0.be) platform. Results from these nodes will drive the evaluation described in [Section 13.1](#131-comment-period-and-next-steps).

---

## 14. Why a dedicated amateur frequency helps with BIPT/IBPT

A dedicated and coordinated amateur-band frequency helps prevent future regulatory problems.

Without coordination, radio amateurs may independently deploy MeshCore, Meshtastic, or LoRa systems on 433 MHz or 868 MHz ISM/SRD frequencies. That creates a grey zone where amateur-style communication, high-site antennas, callsigns, and repeater-like behaviour are mixed with licence-free spectrum.

By proposing a coordinated 70 cm amateur channel, the amateur-radio community can show that it is:

- acting responsibly;
- respecting the IARU band plan;
- avoiding existing repeater inputs and outputs;
- avoiding unnecessary use of ISM/SRD spectrum for amateur infrastructure;
- documenting technical parameters;
- keeping the system ham-only and non-encrypted;
- reducing the risk of later BIPT/IBPT complaints or enforcement discussions.

This is a proactive coordination effort, not a reaction to a problem.

---

## 15. How to comment and contribute

This RFC is maintained as a living document on GitHub:

> <https://github.com/Guru-RF/meshcore-rfc-be>

There are two ways to take part. You do **not** need to be a software developer for either.

### 15.1 Comments and objections — open an Issue

For comments, questions, objections, or local frequency conflicts, open a **GitHub Issue**:

> <https://github.com/Guru-RF/meshcore-rfc-be/issues>

An Issue is simply a public discussion thread attached to the document. Click "New issue", type your point in plain language (English, please — see below), and submit. The maintainer and other contributors can then reply and discuss. This is the easiest route and requires only a free GitHub account.

### 15.2 Proposing concrete text changes — Issue or Pull Request

Specific wording changes can be proposed in either of two ways:

- **The easy way:** open an Issue (as above) describing the change you want, and the maintainer will edit the document for you.
- **The direct way:** submit a **Pull Request (PR)**.

**What is a Pull Request?** A PR is GitHub's mechanism for proposing an exact edit to a document. In short: you make your own copy of the file (a "fork" or branch), change the text there, and then open a PR. The PR shows your proposed change side-by-side against the current text as a **diff** — added lines in green, removed lines in red — so everyone can see precisely what you are suggesting. The maintainer can review it, discuss it, ask for adjustments, and finally **merge** (accept) it or decline it. Nothing in the official document changes until a PR is merged, so it is a safe, transparent, fully reviewable way to suggest concrete wording. GitHub even lets you do all of this in the browser, with no special software installed.

If GitHub is unfamiliar to you, do not let that stop you: send your comment by your usual channel to ON6URE / RF.Guru and it will be entered into the tracker on your behalf.

### 15.3 Language

Please keep Issues and Pull Requests in **English**, so the discussion stays accessible to all Belgian language communities and to any IARU/BIPT readers who later review the record.

---

## 16. Open questions for comments

Feedback is requested on the following points:

1. Is 430.475 MHz acceptable as the primary Belgian ham-only MeshCore / LoRa centre frequency?
2. Is the MeshCore Europe stock profile (62.5 kHz, SF8, CR 4/8) the right on-air mode? In particular, 62.5 kHz exceeds the 20 kHz maximum channel bandwidth listed for this sub-band (see [Section 12.1](#121-frequency-selection-rationale)) — should the project keep 62.5 kHz and seek coordination for the wider bandwidth, or adopt a ≤ 20 kHz LoRa profile to conform?
3. Are there known local conflicts around 430.475 MHz?
4. Should the normal recommended conducted RF output be 1 W, 2 W, or another value?
5. Is 2 W conducted RF output acceptable as an upper technical limit?
6. Should ERP/EIRP limits be defined more explicitly?
7. Should mobile/portable nodes use the same frequency or a separate access frequency?
8. Should fixed repeater/backbone nodes be registered on the overarching ON0.be platform?
9. Should this be proposed as a national recommendation across associations?
10. Should a small technical working group be created to maintain the frequency and node list?

---

## 17. Proposed conclusion

The proposed starting point is:

- **430.475 MHz** centre frequency
- **62.5 kHz** bandwidth, **SF8**, **CR 4/8** — MeshCore Europe stock profile, retuned to 430.475 MHz (see [Section 5.1](#51-on-air-radio-settings-the-testable-mode))
- **1–2 W** conducted RF output recommended
- **2 W** conducted RF output maximum when justified
- Ham-only use
- No encryption
- Callsign identification required
- Every fixed relay node licensed as an unmanned station; all nodes sharing the one coordinated, mesh-based frequency
- Coordinated fixed nodes, registered on the overarching ([ON0.be](https://on0.be)) unmanned-station platform
- Avoid 433 MHz and 868 MHz ISM/SRD for ham-only infrastructure

This approach gives Belgian radio amateurs a clean, coordinated, technically defensible path for MeshCore / LoRa experimentation without occupying repeater channels and without creating unnecessary confusion in ISM/SRD spectrum.

---

## 18. References

- **IARU Region 1 band plans (incl. 430–440 MHz / 70 cm)**
  <https://www.iaru-r1.org/reference/band-plans/>
  Direct 70 cm (UHF) band plan: <https://www.iaru-r1.org/wp-content/uploads/2021/03/UHF-Bandplan.pdf>
- **ON0.be — overarching platform for managing unmanned stations in Belgium**
  <https://on0.be>
- **LoRa-APRS — project and default frequency settings (433.775 MHz, Europe)**
  <https://lora-aprs.org/>
- **BIPT / IBPT radio amateur information**
  <https://www.bipt.be/consumers/radio-frequencies/private-use-hobby/radioamateurs>
- **RF.Guru technical note: "Meshtastic, MeshCore, 868 MHz, and the Ham Radio Trap"**
  <https://shop.rf.guru/pages/meshtastic-meshcore-868-mhz-and-the-ham-radio-trap>

---

## Contributing

See [Section 15 — How to comment and contribute](#15-how-to-comment-and-contribute). Comments via [GitHub Issues](https://github.com/Guru-RF/meshcore-rfc-be/issues); proposed text changes via Issues or Pull Requests. English only.
