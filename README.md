# RFC: Proposal for a Coordinated 70 cm Amateur MeshCore / LoRa Data Channel for IARU Region 1

| | |
| --- | --- |
| **Draft version** | 0.3 |
| **Prepared by** | Joeri Van Dooren, [ON6URE](https://on6ure.be) / [RF.Guru](https://rf.guru) |
| **Intended audience** | IARU Region 1 national societies and their VHF/UHF managers, the IARU R1 VHF/UHF/Microwaves Committee (C5), repeater and unmanned-station coordinators, and technical working groups |
| **Status** | Request for comments |
| **Comment period** | 60 days from publication of this draft (via [GitHub Issues](https://github.com/Guru-RF/meshcore-rfc-be/issues)), to allow circulation to IARU Region 1 national societies and their VHF managers |

---

## 1. Purpose of this RFC

This document proposes the coordinated use of a single dedicated 70 cm amateur-radio frequency for ham-only MeshCore / LoRa-based digital mesh experiments across IARU Region 1 (Europe). It began as a Belgian proposal and has been broadened to a Region-1 scope, because a mesh is inherently border-crossing: a channel that stops at a national boundary cannot serve a European MeshCore network.

The goal is to obtain broad technical and organisational backing — from national societies and, ultimately, the IARU Region 1 VHF/UHF/Microwaves Committee (C5) — before wider deployment, so that the project remains compatible with existing 70 cm usage and IARU Region 1 band planning, and so that a single channel can be used lawfully by amateurs in as many countries as possible.

This is not a request to replace existing repeater coordination. It is a proposal to avoid fragmented, uncoordinated use of 433 MHz and 868 MHz devices by licensed radio amateurs, to avoid conflict with non-ham users that use the ISM band on 70 cm, and to provide a cleaner amateur-radio alternative — together with a specific coordination request to IARU Region 1 (see [Section 12.1](#121-frequency-selection-rationale) and [Section 13](#13-coordination-proposal)).

---

## 2. Background

MeshCore and similar LoRa-based mesh systems are becoming increasingly popular among radio amateurs. Many existing devices are designed for licence-free ISM/SRD use around 433 MHz or 868 MHz. That creates several problems when radio amateurs start using the same systems for ham-radio style communication:

1. The ISM/SRD bands are not amateur bands in the regulatory sense.
2. The ISM/SRD bands are shared with many non-amateur users.
3. ISM/SRD equipment is often subject to strict power, duty-cycle, channel-access, and equipment rules.
4. Radio amateurs may incorrectly assume that using a callsign or amateur purpose automatically makes ISM/SRD operation "amateur radio".
5. Many amateurs already transmit their callsign on 868 MHz (and on 433 MHz), which is explicitly not permitted: callsign identification is strictly an amateur-band operating practice and has no place on ISM/SRD spectrum. Adding a callsign to an ISM/SRD transmission does not make it amateur radio — it simply puts non-compliant identification onto a licence-free band.
6. Uncoordinated high-site LoRa nodes on 433 MHz or 868 MHz can create avoidable interference and regulatory confusion.
7. A fragmented situation may later create problems with BIPT/IBPT or any national regulator if amateur stations are seen to be operating in licence-free spectrum as if it were an amateur allocation.

Technical and regulatory background notes have been published by RF.Guru:

- "Meshtastic, MeshCore, 868 MHz, and the Ham Radio Trap" (April 2026) — <https://shop.rf.guru/pages/meshtastic-meshcore-868-mhz-and-the-ham-radio-trap>
- "Meshtastic, MeshCore, CE Marking, and the Hardware Trap" (updated July 2026) — <https://shop.rf.guru/pages/meshtastic-meshcore-ce-marking-and-the-hardware-trap>. CE marking of a LoRa board as SRD/ISM equipment does not make it compliant for amateur use; regulatory compliance depends on the complete system (antenna, firmware, power, duty cycle), not the certification of individual components.

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

This RFC adopts MeshCore's **stock Europe profile, which uses 62.5 kHz** (see [Section 5.1](#51-on-air-radio-settings-the-testable-mode)). That deliberately favours range, weak-signal performance, and a small spectrum footprint over raw speed — appropriate for a low-rate messaging and emergency-fallback layer — and it keeps every node on the same settings as the wider MeshCore ecosystem. Centred on 434.890 MHz the channel occupies ≈ 434.859–434.921 MHz, fitting inside the 434.790–435.000 MHz window between the ISM top and the satellite floor, and inside Germany's 434.800–435.000 MHz digital / automatic-station segment (200 kHz max) — so a 62.5 kHz channel is comfortably conformant there, with headroom to widen to 125 kHz later if a segment-wide designation is agreed (see [Section 12.1](#121-frequency-selection-rationale)).

---

## 4. Proposed operating concept

The proposed system is a European (IARU Region 1) ham-only MeshCore / LoRa digital mesh layer using the 70 cm amateur band.

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

> **434.890 MHz**

With **62.5 kHz** occupied bandwidth, a 434.890 MHz centre frequency occupies approximately:

> **434.85875 MHz to 434.92125 MHz**

This places the channel in the only part of the 432–438 MHz amateur sub-band that is simultaneously **above the 70 cm ISM/LPD433 band** (which ends at 434.790 MHz) and **below the amateur-satellite segment** (which starts at 435.000 MHz) — the ~210 kHz window 434.790–435.000 MHz. Centred at 434.890 MHz the channel keeps ≈ 68.75 kHz of guard to the ISM top below and ≈ 78.75 kHz to the satellite floor above, biased slightly toward extra satellite guard. It aligns with **Germany's DARC 2025 designation of 434.800–435.000 MHz for networked digital / automatic-station experiments** (up to 200 kHz), inside which a 62.5 kHz channel is fully conformant.

> **Hard spectral limit:** no station may emit above **435.000 MHz**. The amateur-satellite service begins there; every node must be kept, by clean modulation and adequate filtering, strictly below that boundary (cf. the DARC rule "_die Bereichsgrenze 435 MHz darf nicht überschritten werden_"), using the lowest necessary power.

Section [12.1](#121-frequency-selection-rationale) records in full why 434.890 MHz was chosen for a Region-1 channel — why the previously proposed Belgian frequency 430.475 MHz could not be harmonised across Europe, and the two caveats (a nominal repeater-output designation and the satellite edge) that this choice carries and how they are handled.

The proposed frequency also avoids the 433 MHz ISM/LPD433 area, which is heavily affected by ISM/SRD devices, remote controls, sensors, weather stations, consumer LoRa devices, and uncoordinated Meshtastic-type nodes; its lower edge sits ≈ 79 kHz above the top of that band.

### 5.1 On-air radio settings (the testable mode)

So that anyone can join the channel and actually decode it, the network uses a single, explicitly defined mode. It is **MeshCore's stock Europe profile with only the frequency changed to 434.890 MHz** — existing MeshCore users need only retune. All nodes must match every parameter below; a mismatch in any one of them (bandwidth, spreading factor, coding rate) means stations cannot hear each other.

| Parameter | Value |
| --- | --- |
| Modulation | LoRa (Chirp Spread Spectrum) |
| Firmware | MeshCore (companion or repeater role) |
| Centre frequency | **434.890 MHz** |
| Bandwidth | **62.5 kHz** |
| Spreading factor | **SF8** |
| Coding rate | **4/8** (shown as `8` in the MeshCore apps) |
| Occupied bandwidth | ≈ 434.85875 – 434.92125 MHz (never above 435.000 MHz) |
| TX power | MeshCore default 22 dBm (≈ 158 mW) for testing; RFC maximum **2 W** conducted at fixed sites (see [Section 9](#9-power-proposal)) |
| Channel / key | Default **public** MeshCore channel with its published shared key — do **not** configure private/secret keys (no encryption for amateur traffic — see [Section 11](#11-technical-requirements-for-participating-repeaters)) |

This is a deliberately long-range, low-rate, narrow profile (see the bandwidth trade-off in [Section 3.5](#35-signal-bandwidth-and-the-625-khz-choice)), which suits a messaging and emergency-fallback layer and keeps the spectrum footprint small.

---

## 6. Compatibility with existing 70 cm repeaters

The 434.890 MHz channel sits in the 434.790–435.000 MHz window. Two neighbours matter, and both are handled explicitly:

- **The amateur-satellite segment, 435.000–438.000 MHz**, begins ≈ 78.75 kHz above the channel's upper edge (434.92125 MHz). Satellite reception is weak-signal and internationally coordinated, so the RFC mandates a hard limit — **no emission above 435.000 MHz** — plus lowest-necessary power and clean, non-overdriven transmitters (see [Section 5](#5-proposed-frequency-and-on-air-mode) and [Section 9](#9-power-proposal)). LoRa's 62.5 kHz occupied bandwidth and low power spectral density help keep energy off the edge; the **434.875 MHz fallback** in [Section 12](#12-proposed-initial-frequency-plan) raises the satellite guard to ≈ 93.75 kHz if a satellite coordinator objects.
- **Legacy 1.6 / 2 MHz-shift "RU" repeater outputs, 434.600–434.9875 MHz.** In most of Region 1 this block is defunct (modern repeaters use the 7.6 MHz-shift system with outputs at 438.65–439.425 MHz), and Germany has re-designated its top (434.800–435.000 MHz) to digital / automatic-station use. **But it is not universally empty:** live repeaters are known to occupy 434.900 MHz in **Finland (OH3RUX, 50 W)** and **Norway (LA5YR)**. The channel therefore must **not** be deployed there until locally re-coordinated — see the per-country checklist in [Section 13](#13-coordination-proposal).

This proposal therefore aims to avoid:

- direct channel overlap with any _deployed_ repeater, link, or packet station (hence the OH/NO exclusions and the per-country check);
- any emission crossing into the protected satellite segment.

Where a MeshCore node is installed at an existing repeater site, co-location filtering is addressed in [Section 11.1](#111-co-siting-at-existing-repeater-sites).

---

## 7. Why not use 433 MHz ISM/SRD?

Although 433 MHz equipment is widely available, the 433 MHz area is not a clean solution for a coordinated ham-only network.

Reasons:

1. 433 MHz is heavily used by non-amateur ISM/SRD devices.
2. The band is shared with consumer devices that cannot coordinate with amateur stations.
3. Many devices use unknown duty cycles, unknown protocols, and low-cost RF stages.
4. It is difficult to distinguish amateur use from non-amateur ISM/SRD use.
5. The standard European LoRa-APRS frequency, **433.775 MHz** (125 kHz, SF12), is already in active use, and Europe has the densest LoRa-APRS network in the world. A MeshCore deployment in the 433 MHz area would conflict directly with this existing traffic.
6. Long-range high-site amateur nodes may create conflicts with licence-free users.
7. Some parts of the 433 MHz area overlap with IARU Region 1 repeater input/output planning.
8. Using 433 MHz for ham-only infrastructure risks creating confusion with any national regulator if radio amateurs appear to be using ISM/SRD spectrum as an amateur allocation.

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
7. A growing number of uncoordinated 868 MHz nodes could later create avoidable issues with any national regulator.

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

1. **A personal amateur licence is not sufficient.** A personal amateur licence (e.g. HAREC) authorises _you_ to operate a station under your direct control. It does not, on its own, authorise an unattended automatic relay. Every fixed MeshCore repeater/relay node therefore requires a separate **unmanned-station / repeater authorisation** from the national regulator (in Belgium, BIPT/IBPT) before it may operate.
2. **The mesh is single-frequency and mesh-based.** Because MeshCore is mesh-based, every relaying node receives and re-transmits on the _same_ coordinated frequency; there is no separate input/output pair as on an FM repeater. All licensed nodes share one channel — see [Section 3](#3-how-lora-and-meshcore-work-on-the-air).
3. **Register every licensed node.** Each authorised unmanned node and its responsible operator should be registered nationally, so the node list stays coordinated — see [Section 13](#13-coordination-proposal).
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
12. Registration of the node and its responsible operator with the relevant national coordination — see [Section 13](#13-coordination-proposal).

### 11.1 Co-siting at existing repeater sites

Where a MeshCore node is installed on the same mast or site as an existing repeater, the node's transmitter can degrade the repeater's receiver (and, less often, the reverse). Coordinating the frequency and site is the first line of defence — a node should not be co-sited with a repeater whose receive frequency is close to the node channel (see the offset table below) — but at a shared site additional filtering is usually still needed. Two **distinct** mechanisms are involved, and they need **different** fixes:

1. **Transmitter noise falling in the repeater's passband.** Every transmitter emits low-level wideband noise on either side of its carrier. Any of that noise landing on the repeater's _receive_ frequency is indistinguishable from a wanted signal there and **cannot be removed by any filter at the repeater**. It must be cleaned at the source — with a band-pass cavity / filter on the MeshCore transmitter output, and by using a clean PA that is not overdriven. This is why transmitter cleanliness matters more than raw power.
2. **Receiver blocking / desensitisation.** The strong nearby MeshCore _carrier_ itself (off the repeater's frequency) can overload the repeater's front-end and raise its noise floor. This is what a **band-reject (notch) filter or a band-pass preselector at the repeater's receive input** can reduce — provided there is enough frequency separation for the filter's skirts.

A notch filter in the repeater's RX chain therefore addresses **mechanism 2 only**. It is a useful and often necessary component, but on its own it is **not a complete answer**, because it does nothing about mechanism 1 (you cannot notch out energy sitting on your own wanted frequency). The robust approach is layered, roughly in order of effectiveness and cost:

- **Antenna separation** (especially vertical spacing) — usually the cheapest large amount of isolation;
- a **band-pass filter / cavity on the MeshCore transmitter output** — kills transmitter wideband noise in the repeater's receive passband (mechanism 1);
- a **band-pass preselector on the repeater receiver** — rejects the off-channel MeshCore carrier and other on-site signals broadly (mechanism 2);
- a **notch tuned to the MeshCore channel** — a targeted supplement to the preselector when one specific carrier dominates;
- the node's inherently **low power (1–2 W) and low duty cycle**, which already limit the severity.

The further the MeshCore channel sits from the co-located repeater's input, the wider the filter skirts can be and the cheaper every one of these measures becomes. Each co-located installation should document its co-siting arrangement (antenna separation, TX filtering, RX filtering) alongside the technical parameters required in [Section 9.3](#93-erpeirp-consideration) and the list above.

**Practical guidance by offset from the repeater input.** Because the node runs only **1 W (≈ +30 dBm)**, co-siting is far easier than for a normal high-power repeater — but the receive-side numbers still bite. With typical on-mast antenna isolation of 40–60 dB (vertical separation helps most), the node's carrier still arrives at roughly −10 to −20 dBm at the repeater's front end — more than enough to desensitise a sensitive receiver. The aim is to add enough rejection to bring that carrier down to about −40 dBm or lower at the receiver input; a cavity notch tuned to the node frequency supplies it, **provided the offset Δf between the node channel and the repeater input is large enough for the notch skirts to clear the wanted signal.** As a rough rule of thumb (always measure on site):

| Offset Δf (node channel ↔ repeater input) | Same-site outlook at 1 W |
| --- | --- |
| **< ~0.1 MHz** | Very difficult. A notch deep enough to protect the input would also attenuate the wanted receive signal. Not recommended on a shared antenna system — use large vertical antenna separation, a separate site, or avoid co-siting with an input this close. |
| **~0.1–0.5 MHz** | Feasible but demanding: needs one or two **high-Q cavity notches** at the node frequency on the repeater RX line, good antenna separation, and a clean / band-pass-filtered node TX. Tune and measure. |
| **~0.5–2 MHz** | Comfortable: a single good **cavity notch** at the node frequency on the repeater RX, plus antenna separation, is normally enough. |
| **> ~2 MHz** | Usually fine: the repeater's existing duplexer / front-end and antenna separation generally suffice; just keep the node TX clean. |

Worked example: a node on 434.890 MHz co-sited with a 7.6 MHz-shift repeater whose receiver is around 431.2 MHz sees Δf ≈ 3.7 MHz — comfortably in the "usually fine" row; even a co-located repeater receiving in the 433.000–433.375 MHz input block is Δf ≈ 1.5 MHz away, still comfortable. Note the notch protects only against receiver **blocking**; the node's own transmitter **noise** landing on the repeater's input frequency cannot be notched at the repeater and must be handled by the band-pass filter on the node TX (mechanism 1 above) — which is itself least effective at very small Δf, another reason to keep the node well away from any co-sited receiver.

**Turning the node's power down is often the simplest fix — and the only one that also cures TX noise.** Every dB of node output removed reduces both the blocking carrier and the transmitter noise reaching the repeater by the same dB, so dropping a co-sited node from 1 W to, say, 100 mW (−10 dB) or 10 mW (−20 dB) can do more, more cheaply, than a cavity — and it is the only measure that also helps the un-notchable TX-noise path (mechanism 1). Because a MeshCore relay's coverage comes mainly from siting and antenna height rather than raw power (see [Section 9](#9-power-proposal)), a node on a good repeater mast can usually afford to run low power and still cover well. At shared sites, therefore, use the lowest power that still gives reliable mesh coverage — a co-located node will often need far less than the general 1–2 W.

**Spreading is not a substitute for this.** It is tempting to assume LoRa's chirp spreading makes the node harmless: its 1 W is smeared across 62.5 kHz, so its power spectral density is low, and a narrowband repeater receiver only catches the fraction of that energy that falls in its own passband (a 12 kHz FM receiver sees roughly a fifth — about 7 dB less — of overlapping in-band energy than it would from an equal-power narrowband source). That genuinely makes the node a gentler co-channel / adjacent-channel neighbour, and — together with LoRa's processing gain — it also makes the node itself far more tolerant of a noisy band (see [Section 3.2](#32-processing-gain--hearing-below-the-noise) and [Section 3.3](#33-why-a-single-shared-frequency-rarely-collapses)). But spreading does **not** reduce front-end **blocking / desensitisation**: a receiver's front end is wideband and sees the node's _total_ carrier power regardless of how it is spread, so the −10 to −20 dBm figures above are unchanged by the spreading factor. Blocking is defeated only by frequency separation, filtering, antenna isolation and lower power — not by spreading.

---

## 12. Proposed initial frequency plan

Initial proposal:

| Use | Centre frequency | Bandwidth | Notes |
| --- | --- | --- | --- |
| Primary Region-1 ham-only MeshCore channel | **434.890 MHz** | 62.5 kHz | In the 434.790–435.000 MHz window; aligns with the German 434.800–435.000 digital / automatic-station segment |
| Satellite-conservative fallback | 434.875 MHz | 62.5 kHz | Adopt if a satellite coordinator objects to the edge — raises the guard to 435.000 to ≈ 93.75 kHz |

The channel occupies 434.85875–434.92125 MHz (primary) and must never emit above **435.000 MHz**.

**Segments to avoid across the 70 cm band:**

| Range / frequency | Reason |
| --- | --- |
| 430.000–432.000 MHz | Lower 70 cm — **not** available to amateurs in several countries (ITU RR 5.274/5.275: DK, NO, SE, FI, EE; see [Section 12.1](#121-frequency-selection-rationale)); FM repeater outputs/inputs and digital sub-bands elsewhere |
| 432.000–432.490 MHz | Narrow-band weak-signal (CW / SSB / EME / MGM) and exclusive beacons — never place wideband data here |
| 432.500–433.375 MHz | APRS (432.500) and Region-1 repeater inputs |
| 433.400–433.575 MHz | FM/DV calling channels (433.450 DV, 433.500 FM) and SSTV |
| 433.050–434.790 MHz | 70 cm ISM / LPD433 band — the licence-free spectrum this project exists to leave (includes 433.775 LoRa-APRS) |
| 435.000–438.000 MHz | Amateur-satellite segment — **hard boundary; no emission above 435.000 MHz** |
| 438.000–440.000 MHz | 7.6 MHz-shift repeater outputs (438.65–439.425), digital links and paging; also footnote-restricted (5.274/5.275) in DK/NO/SE/FI/EE |

The chosen 434.890 MHz sits in the narrow 434.790–435.000 MHz gap between the ISM top and the satellite floor. This gap falls inside the IARU R1 434.600–434.981 MHz legacy repeater-**output** designation; the RFC deliberately occupies it on the basis that the legacy 1.6/2 MHz-shift output system is defunct across most of Region 1 and that Germany has already re-designated 434.800–435.000 to digital / automatic-station use — see [Section 12.1](#121-frequency-selection-rationale) for the full rationale and the coordination request that goes with it.

### 12.1 Frequency selection rationale

This channel began, in drafts 0.1–0.2, as a Belgian frequency in the lower 70 cm digital-link segment (430.500, then 430.475 MHz). Draft 0.3 re-scopes the RFC to IARU Region 1 and moves the channel to **434.890 MHz**. This section explains why.

**Why the lower 70 cm (430.475 MHz) cannot be a European channel.** The segment 430.475 MHz sits in is legal for amateurs in Belgium and most of Region 1, but it is **not available to amateurs at all in several countries**, because ITU Radio Regulations footnotes **5.274** (alternative allocation — fixed/mobile primary in 430–432 and 438–440 MHz in **Denmark, Norway, Sweden**) and **5.275** (additional fixed/mobile primary in the same edges in **Finland, Estonia, Croatia** and others) displace or subordinate the amateur service there. In DK/NO/SE/FI/EE the national 70 cm amateur band simply starts at 432 MHz. A channel a MeshCore user cannot legally key up in Scandinavia and the Baltics cannot be a European standard. (These footnotes touch only 430–432 and 438–440 MHz; they leave 432–438 MHz untouched. Latvia, Lithuania and Iceland are **not** in either footnote — the exclusion is a five-country matter, not "the whole lower band.")

**Why 432–438 MHz, and why 434.890 specifically.** Only 432–438 MHz is amateur spectrum region-wide. Nearly all of it is spoken for — weak-signal/beacons (432.0–432.49), repeater inputs and APRS (432.5–433.375), calling channels (433.4–433.575), the ISM/LPD433 band (433.05–434.79) and the amateur-satellite segment (435.0–438.0). The **only** spectrum at once above the ISM top and below the satellite floor is the ~210 kHz window **434.790–435.000 MHz**, and 434.890 MHz is essentially its centre (≈ 68.75 kHz to ISM below, ≈ 78.75 kHz to satellite above, biased for a little extra satellite guard). It also lands inside **Germany's DARC 2025 segment 434.800–435.000 MHz for "networked digital / automatic-station" experiments** (200 kHz max), which both endorses this exact use and dissolves the channel-width problem below.

**This resolves the channelisation problem.** At 430.475 MHz the 62.5 kHz channel was ~3× the 20 kHz the plan allows in that sub-band. In the German 434.800–435.000 digital segment the allowance is 200 kHz, so 62.5 kHz is fully conformant — with headroom to widen to 125 kHz later. The clean pan-European fix is to ask IARU R1 to make the German-style designation region-wide (see below), which cures the channel width and the repeater-output label in one step.

**It also helps on the AST-SpaceMobile front.** The AST SpaceMobile "Bluebird" constellation uses 70 cm frequencies for satellite telemetry, tracking and command, reportedly including **430.500 MHz** — which lands directly on the old lower channel — as well as 432.300, 434.100, 435.900 and 439.500 MHz. 434.890 MHz falls in the clear gap between 434.100 and 435.900 (≈ 0.8–1.0 MHz either side); the lower channel did not. (IARU issued a position statement on the AST 70 cm use in 2025.)

**The two caveats this choice carries, and how they are handled.**

- _Nominal repeater-output designation._ 434.890 MHz is inside the IARU R1 434.600–434.981 MHz legacy "RU" repeater-**output** band, and on paper that re-arms the very "you occupied a repeater output" objection this project set out to avoid. In practice the legacy 1.6/2 MHz-shift output system is **defunct across most of Region 1** (modern repeaters use the 7.6 MHz shift, outputs 438.65–439.425), and Germany has affirmatively repurposed the top of the block. But it is **not universally empty**: live repeaters occupy 434.900 MHz in **Finland (OH3RUX, 50 W)** and **Norway (LA5YR)**. The channel is therefore treated as a _coordination request_, not a clean plan-match: it is **excluded in OH and NO until re-coordinated**, subject to a per-country check ([Section 13](#13-coordination-proposal)), and the headline ask to IARU R1 is to re-designate 434.790–435.000 MHz (or 434.800–435.000) as a networked-digital / automatic-station segment.
- _Satellite edge._ The channel's upper edge is ≈ 78.75 kHz below the 435.000 MHz satellite boundary. This is a boundary/optics concern, not a space-service violation (the channel stays wholly in the terrestrial 430–440 allocation), but the amateur-satellite community is protective of the 435 MHz edge — especially during the 2025 AST dispute. It is handled by the **hard "no emission above 435.000 MHz" rule** (mirroring the DARC boundary rule), lowest-necessary power, clean transmitters, and the **434.875 MHz satellite-conservative fallback** if a coordinator formally objects.

**Honest summary.** For a Belgium-only channel, 430.475 MHz was cleaner (a proper digital-link segment with no satellite or repeater-output issue). For a **European** channel it is disqualified — legally in five countries and by the AST 430.500 co-channel — so 434.890 MHz is the best available Region-1 window, adopted together with a coordination request rather than presented as an unqualified plan match.

---

## 13. Coordination proposal

This RFC proposes that IARU Region 1 national societies and repeater/unmanned-station coordinators agree on a common channel before large-scale deployment, and that the request be carried to the IARU Region 1 VHF/UHF/Microwaves Committee (C5).

Coordination, registration, and maintenance of the known-node list are proposed to run per country — through each national society or its unmanned-station coordinator — keeping each country's node list, responsible-operator contacts, and technical parameters in one authoritative, publicly visible place.

> **Initial consultation:** early informal consultation with neighbouring amateur colleagues (by Rudy, ON6VDS, and others) was carried out on an earlier Belgian lower-70 cm proposal. The re-scope to a Region-1 channel at 434.890 MHz supersedes it and requires fresh consultation, now at European level — see the coordination steps below.

**Suggested coordination steps:**

1. Circulate this RFC among IARU Region 1 national societies (via their VHF/UHF managers) and repeater/technical coordinators — see the [IARU R1 VHF Managers Handbook](https://www.iaru-r1.org/wp-content/uploads/2024/11/VHF_Handbook_V10_02.pdf) and the [VHF/UHF/Microwaves Committee (C5)](https://www.iaru-r1.org/about-us/committees-and-working-groups/vhf-uhf-shf-committee-c5/).
2. Collect objections, local conflicts, and alternative proposals.
3. **Per-country check before deployment.** Confirm no live repeater or other deployed station occupies 434.890 MHz nationally. Known conflicts to clear first: **Finland (OH3RUX)** and **Norway (LA5YR)** operate FM repeaters on 434.900 MHz — do **not** deploy there until re-coordinated. Also confirm national unmanned-station rules permit an automatic station on this frequency (some administrations restrict automatic-station output frequencies).
4. **Petition IARU R1** to designate **434.790–435.000 MHz** (or 434.800–435.000, following the German DARC precedent) as a networked-digital / automatic-station segment — the single change that clears both the legacy repeater-output designation and the channel-width question.
5. Define a list of known fixed nodes and responsible operators, maintained per country by the national society or unmanned-station coordinator.
6. Start with low-power test deployments, keeping all emission strictly below 435.000 MHz.
7. Monitor for interference or complaints; adjust power, or fall back to 434.875 MHz, if needed.
8. Document the final recommendation and make it publicly available.
9. Where appropriate, use the coordinated position in communication with national regulators (in Belgium, BIPT/IBPT).

### 13.1 Comment period and next steps

1. **Comment period — 60 days.** Comments, objections, and reports of national conflicts are invited for **60 days** from publication of this draft, via GitHub Issues (see [Section 15](#15-how-to-comment-and-contribute)) — a longer window than a purely national draft, to allow circulation to IARU Region 1 national societies and their VHF managers.
2. **Request test-site licences.** From the close of the comment period onward, we will apply for a small number of **unmanned-station (repeater) authorisations** for initial test sites (see [Section 10](#10-licensing-and-unmanned-station-status)).
3. **Deploy and evaluate.** Once the test sites are licensed and on the air, the project will be evaluated on how it performs and evolves in practice — coverage, interference, traffic and congestion behaviour, and node density — and this RFC will be revised accordingly before any wider roll-out.

### 13.2 Initial test-bed nodes

The following Belgian stations are planned to form the initial test bed, operating on the proposed **434.890 MHz** channel with the on-air mode defined in [Section 5.1](#51-on-air-radio-settings-the-testable-mode):

- ON0BXL
- ON0VDS
- ON0ORA
- ON0AND
- ON0OST
- ON0UHF
- ON0VHF

Each will be brought up as a licensed unmanned station (see [Section 10](#10-licensing-and-unmanned-station-status)) and registered with the relevant national coordination. Results from these nodes will drive the evaluation described in [Section 13.1](#131-comment-period-and-next-steps).

---

## 14. Why a dedicated amateur frequency helps with national regulators

A dedicated and coordinated amateur-band frequency helps prevent future regulatory problems.

Without coordination, radio amateurs may independently deploy MeshCore, Meshtastic, or LoRa systems on 433 MHz or 868 MHz ISM/SRD frequencies. That creates a grey zone where amateur-style communication, high-site antennas, callsigns, and repeater-like behaviour are mixed with licence-free spectrum.

By proposing a coordinated 70 cm amateur channel, the amateur-radio community can show that it is:

- acting responsibly;
- respecting the IARU band plan and coordinating any re-designation openly (see [Section 12.1](#121-frequency-selection-rationale));
- keeping strictly clear of the amateur-satellite segment;
- avoiding unnecessary use of ISM/SRD spectrum for amateur infrastructure;
- documenting technical parameters;
- keeping the system ham-only and non-encrypted;
- reducing the risk of later regulator complaints or enforcement discussions.

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

If GitHub is unfamiliar to you, do not let that stop you: send your comment by your usual channel to [ON6URE](https://on6ure.be) / RF.Guru and it will be entered into the tracker on your behalf.

### 15.3 Language

Please keep Issues and Pull Requests in **English**, so the discussion stays accessible across Region 1 language communities and to any IARU / national-regulator readers who later review the record.

---

## 16. Open questions for comments

Feedback is requested on the following points:

1. Is 434.890 MHz acceptable as a Region-1-wide ham-only MeshCore / LoRa centre frequency (with 434.875 MHz as the satellite-conservative fallback)?
2. Should IARU R1 be asked to designate 434.790–435.000 MHz (German DARC style) as a networked-digital / automatic-station segment, which would make the 62.5 kHz channel fully conformant and clear the legacy repeater-output designation?
3. Are there national conflicts at 434.890 MHz beyond the known Finnish (OH3RUX) and Norwegian (LA5YR) repeaters on 434.900 MHz, and does your administration permit an automatic station there?
4. Should the normal recommended conducted RF output be 1 W, 2 W, or another value?
5. Is 2 W conducted RF output acceptable as an upper technical limit?
6. Should ERP/EIRP limits be defined more explicitly?
7. Should mobile/portable nodes use the same frequency or a separate access frequency?
8. Should fixed repeater/backbone nodes be registered through each national society / unmanned-station coordinator?
9. Should this be proposed as a national recommendation across associations?
10. Should a small technical working group be created to maintain the frequency and node list?

---

## 17. Proposed conclusion

The proposed starting point is:

- **434.890 MHz** centre frequency (Region-1 scope), with **434.875 MHz** as a satellite-conservative fallback; **no emission above 435.000 MHz**
- **62.5 kHz** bandwidth, **SF8**, **CR 4/8** — MeshCore Europe stock profile, retuned to 434.890 MHz (see [Section 5.1](#51-on-air-radio-settings-the-testable-mode))
- **1–2 W** conducted RF output recommended
- **2 W** conducted RF output maximum when justified
- Ham-only use
- No encryption
- Callsign identification required
- Every fixed relay node licensed as an unmanned station; all nodes sharing the one coordinated, mesh-based frequency
- Coordinated fixed nodes, registered per country through the national society or unmanned-station coordinator
- A coordination request to IARU R1 to designate 434.790–435.000 MHz for networked-digital / automatic-station use; a per-country check before deployment, excluding Finland and Norway until re-coordinated
- Avoid 433 MHz and 868 MHz ISM/SRD for ham-only infrastructure

This approach gives radio amateurs across Region 1 a coordinated, technically defensible path for MeshCore / LoRa experimentation on a single harmonised channel, keeping the network out of ISM/SRD spectrum and strictly clear of the amateur-satellite band.

---

## 18. References

- **IARU Region 1 band plans (incl. 430–440 MHz / 70 cm)**
  <https://www.iaru-r1.org/reference/band-plans/>
  Direct 70 cm (UHF) band plan: <https://www.iaru-r1.org/wp-content/uploads/2021/03/UHF-Bandplan.pdf>
- **LoRa-APRS — project and default frequency settings (433.775 MHz, Europe)**
  <https://lora-aprs.org/>
- **National regulator — radio amateur information (example: BIPT/IBPT, Belgium)**
  <https://www.bipt.be/consumers/radio-frequencies/private-use-hobby/radioamateurs>
- **RF.Guru technical note: "Meshtastic, MeshCore, 868 MHz, and the Ham Radio Trap"**
  <https://shop.rf.guru/pages/meshtastic-meshcore-868-mhz-and-the-ham-radio-trap>
- **RF.Guru technical note: "Meshtastic, MeshCore, and the Legal Framework"** — why 868 MHz LoRa is SRD spectrum, not an amateur allocation, and why a callsign or a software duty-cycle limiter does not confer amateur status
  <https://shop.rf.guru/pages/meshtastic-meshcore-and-the-legal-framework>
- **RF.Guru technical note: "Meshtastic, MeshCore, CE Marking, and the Hardware Trap"** — CE/RED compliance of consumer LoRa hardware vs amateur use
  <https://shop.rf.guru/pages/meshtastic-meshcore-ce-marking-and-the-hardware-trap>
- **IARU Region 1 VHF/UHF/Microwaves Committee (C5) and VHF Managers Handbook**
  <https://www.iaru-r1.org/about-us/committees-and-working-groups/vhf-uhf-shf-committee-c5/>
  Handbook v10.02 (Nov 2024): <https://www.iaru-r1.org/wp-content/uploads/2024/11/VHF_Handbook_V10_02.pdf>
- **ITU Radio Regulations footnotes 5.274 / 5.275** — fixed/mobile allocations in 430–432 and 438–440 MHz (DK/NO/SE and FI/EE/HR + others) that exclude or subordinate amateurs in the lower 70 cm in those countries
- **DARC 70 cm band plan (2025)** — 434.800–435.000 MHz for networked digital / automatic-station experiments
  <https://www.darc.de/der-club/referate/hf/bandplaene/>
- **IARU position on AST SpaceMobile 70 cm use (2025)** — commercial satellite TT&C in the amateur band
  <https://www.iaru-r1.org/>

---

## Contributing

See [Section 15 — How to comment and contribute](#15-how-to-comment-and-contribute). Comments via [GitHub Issues](https://github.com/Guru-RF/meshcore-rfc-be/issues); proposed text changes via Issues or Pull Requests. English only.
