# Replacing the RTC6705 in a 5.8 GHz analog FPV VTX

Research note, 2026-08-24, second pass the same day after the MAX2871 datasheet was read in full and the market premise was re-checked.
Tags: [P] primary source read directly, [I] inferred from primary data, [G] guess, unverified.
Prices are LCSC list, USD, qty 1 to 9 unless stated. Stock as of 2026-08-24 via jlcsearch.tscircuit.com (JLCPCB in-stock mirror).
MAX2871 numbers in section 8 are from the datasheet 19-6547 Rev 2, mirrored at
https://datasheet.lcsc.com/datasheet/pdf/7df3658eebf9adf710f9045737cdc8ae.pdf (analog.com blocks non-browser fetches). [P]

## 1. The video

https://www.youtube.com/watch?v=r6h3y7gFdR8, Joshua Bardwell, "Analog vTX have a BIG problem. Divimath just solved it.", published 2026-08-05, 26 min. [P]

Product review, not a build video. No schematic, no chip names, no GitHub. Content:

- Every 5.8 GHz analog VTX uses the same RichWave chip; military drone demand pushed its price up roughly 5x, so the $15 VTX is gone. [P]
- Divimath (HDZero sister company, Carl Zhao; NDAA-compliant, built in Thailand) ships a "Dual-Band Analog VTX" that does not use it. Product page: "does not use conventional Richwave RTC670x chipset". [P] https://www.divimath.com/products/divimath-dual-band-analog-vtx
- Because it is not tied to the RichWave part it is frequency agile: any 4-digit MHz entry, Bardwell's vtxtable ran 4900 to 6030 MHz plus 3.3 GHz. Two MMCX outputs, one 5.8 GHz, one 3.3 GHz. [P]
- Power levels shown 1/14/23/26/30/36 dBm (4 W, "authorized applications" unlock; public listing is 25/200/400 mW). 6 to 25 V in, 37x28x10 mm, 8.7 g, $69.99. SmartAudio and Tramp from one firmware. [P]
- Flight test: 4 W with a Divimath 5.8 GHz bandpass filter on the goggle RX gave a very stable picture; without the filter, dropouts. [P]
- What it proves for us: a non-RTC6705 analog VTX with an arbitrary-frequency synthesiser is shippable in 2026 at a hobby price. The architecture is undisclosed; a wideband fractional-N PLL+VCO synth with the video summed into the VCO tune line is the obvious reading of "any frequency 3.3 to 6 GHz" [I]. Nothing to copy from it.

### 1.1 Wideband products from the big brands are not evidence of a new chip

Foxeer ships Reaper Extreme V2 and V3 at 4.9 to 6.0 GHz, 80 channels, 2.5 to 3 W, and Reaper Infinity at 5 and 10 W; iFlight BLITZ covers a 4.9 GHz band alongside 5.8 GHz. [P, vendor product pages] It is tempting to read that fleet as proof that the industry replaced the RichWave part. It is not.

OpenVTx already programs the RTC6705 from `MIN_FREQ 5000` to `MAX_FREQ 5999` (`src/src/rtc6705.h:21-22`, read 2026-08-24). [P] The chip's VCO runs at half the output, so OpenVTx is already claiming a 2.50 to 3.00 GHz VCO span against a 5725 to 5865 MHz datasheet number. Foxeer's 4.9 to 6.0 GHz needs 2.45 to 3.00 GHz, 50 MHz beyond what OpenVTx claims works. So the whole wideband fleet is explainable as an RTC6705 driven past its datasheet behind a broadband output filter, with the extra power coming from a bigger WLAN PA. Higher output power is a PA problem and every 5 GHz PA in section 6 is an off-the-shelf part. [I]

Divimath is the only vendor making an explicit claim to the contrary. Everyone else is silent, and silence plus a stretched-but-plausible tuning range is weak evidence. No teardown or FCC internal photo was found for any Foxeer or iFlight VTX, so the chip identity of those products is unresolved. [P: searched, nothing found]

Consequence for this project: do not assume a solved problem exists to be copied. The supply and price problem in section 3 stands whatever Foxeer is doing.

## 2. What the RTC6705 is

Source: RTC6705-DST-001 datasheet V0.2, Sep 2007, copy in the OpenVTx repo: https://raw.githubusercontent.com/OpenVTx/OpenVTx/master/docs/RTC6705-RichWave.pdf [P]

Function block [P]:
- Integer-N synthesiser: R counter (default 400 from 8 MHz crystal, so PFD = 20 kHz), N/A counters with a 64 prescaler, charge pump on pin 27 CP, VCO tune on pin 29 VT. The loop filter is external (CP to VT). Frequency formula `FRF = 2*(N*64+A)*(Fosc/R)`: the VCO runs at half the output, about 2.8 to 3.0 GHz, with an on-chip doubler; the register map confirms this with "2G VCO" bias fields in reg 0x05 and "5G" pre-driver/PA fields in reg 0x07. [P] (formula and fields), [I] (doubler topology)
- Step size with default R: 2 x 20 kHz = 40 kHz; OpenVTx computes `freq_kHz/40` and splits into N (div 64) and A (mod 64). [P]
- FM modulation: video into pin 10 VT_Mod, audio subcarrier composite into pin 11 RF_VT2, both analog inputs onto the VCO tune node. Video input 1 Vpp into 75 ohm. [P] The 20 kHz PFD implies a loop bandwidth well under 1 kHz, so the loop cannot track video content and the whole 0 to 6.5 MHz modulation rides on the VCO open loop; only the DC/near-DC average is corrected by the loop. [I]
- Two audio FM subcarrier VCO+PLLs at 6.0 and 6.5 MHz, +-25 kHz audio deviation, video-to-audio carrier ratio -25 dBc. [P]
- Output: pre-driver plus PA, PAOUT1 +2 dBm, PAOUT2 +13 dBm, harmonics -60 dBc with the reference filter. Phase noise -90 dBc/Hz at 100 kHz, -115 at 1 MHz. 3.3 V, 95 mA. [P]
- Datasheet frequency range 5725 to 5865 MHz over temperature, but hardware pins already select 5645 to 5945 (band E) and every FPV VTX runs 5645 to 5945 via SPI. [P]
- Package in this datasheet revision: QFN 6x6, 40 pins. The 5x5 figure in the task brief is not what the datasheet says. [P]
- Pin-selectable 24 channels (bands A/B/E) when SPI_SE = 0; 3-wire SPI when SPI_SE = 1. [P]

Register interface firmware expects (OpenVTx `src/src/rtc6705.c`, `.h`, GPL-3.0) [P]:
- 3-wire bit-banged SPI: SS (SPILE), SCK, MOSI (SPIDATA). 25-bit frames, LSB first: 4 address bits, 1 R/W bit (1 = write), 20 data bits. Read cycles turn MOSI into an input for the 20 data bits.
- Registers used: 0x00 SYN_RF_R (default 0x190 = 400); 0x01 SYN_RF_N[12:0] at bit 7, SYN_RF_A[6:0] at bit 0 (in the 20-bit data field); 0x07 pre-driver/PA control (`POWER_AMP_ON = 0b10011111011111100000`, all-zero = PA off); 0x0F state register (reset by writing zero). Regs 0x02 to 0x06 (CP current, VCO cal, audio VCO) are left at defaults.
- Sequence on channel change: PA off, external PA to 0 dB, write reg 0x00, write reg 0x01, wait `PLL_SETTLE_TIME` 500 ms, restore power.
- Output power is not set through the RTC6705 beyond on/off. Power is set by a PWM/DAC on the external PA bias (`VREF`, `VPD` on Generic_GD32F130 target, 12 kHz PWM, `target_set_power_dB`) with a table per board, plus optional `RTC_BIAS` pin. So the firmware-facing contract of the chip is small: set frequency in 40 kHz steps 5645 to 5945 MHz, PA on/off, lock wait.
- Target MCUs: GD32F130 (Generic, Eachine TX801) and STM8 (Eachine TX526). SmartAudio 4800 baud, Tramp 9600, MSP 9600 on one half-duplex UART.

What a replacement must expose (to keep OpenVTx/Betaflight/SmartAudio/Tramp semantics; the protocol side is MCU firmware, the chip only needs):
1. Frequency set 5645 to 5945 MHz, step 1 MHz or finer (vtxtable entries are integer MHz), lock in well under 500 ms. Nice to have: 5000 to 5999 (`MIN_FREQ`/`MAX_FREQ` in OpenVTx), and the Divimath-style "any frequency". [P]
2. An analog FM modulation input, 1 Vpp composite video, plus the audio subcarrier(s) summed in. [P]
3. RF out about +13 dBm to drive the same PA chain (or +2 dBm if the chain gains 11 dB more). [P]
4. Chip-level RF mute/PA-off for pit mode and during retune (Betaflight expects pit mode to be near-silent, SmartAudio 2.1 "pit mode" powers the PA down; OpenVTx uses PA off plus PA bias 0). [P]
5. Audio subcarrier generation is optional: nearly every FPV VTX ships without audio, and receivers demodulate video without the subcarriers. [I]

## 3. Integrated substitutes: none exist

Richwave "Wireless Video/Audio" product list (richwave.com.tw blocks bots; mirror https://richwave.bike.idv.tw/page_prd_new.aspx?Id=8) [P]:

| Part | Function | Band | Package | Note |
|---|---|---|---|---|
| RTC6705 / RTC6705A | FM TX | 5.8 GHz | QFN-40 6x6 | the only 5.8 GHz TX in the portfolio. RTC6705A silkscreen "AV05BMP" (the marking seen on BetaFPV/NBD boards) [P]. A vs non-A difference undocumented [G: die revision] |
| RTC6715 | FM RX | 5.8 GHz | QFN-48 7x7 | RX5808 core, 480 MHz IF [P] |
| RTC6701 / RTC6711 | FM TX / RX | 2.4 GHz | QFN-32 / QFN-48 | 2.4 GHz only; hobbyists have run RTC6701 at 1.2 GHz [P] |
| RTC6712 | dual-band FM RX | 2.4 GHz | QFN-48 | receiver, not a TX [P] |
| RTC67033 / RTC67133 / RTC6706S / RTC6716S | FM RX | 1.2 / 3.3 / 6.5 GHz | QFN | receivers only [P] |
| RTC76401 / RTC76402S | PA | 4.88 to 6.06 / 5.15 to 5.85 GHz | QFN-20 4x4 | 34 dBm companion PAs [P] |

- No Chinese equivalent found in English or Chinese queries (国产替代, pin to pin, 兼容). Chinese trade articles (51cos.com 2025, chinaham.cn 2024, oshwhub 2022) all state mainstream 5.8G 图传 = RTC6705 TX + RTC6715 RX. [P]
- Teardowns: Foxeer Reaper (drin.com.ua), Boscam TX5813/TX5823 (EEWorld OpenVTX article references RTC6705 pins), Eachine TX526/TX801 (OpenVTx targets) all RTC6705. [P] Rush/TBS/Happymodel/Walksnail not verified individually [G: same chip, since no alternative exists].
- Beken BK5811/BK5813 are 5.8 GHz GFSK data transceivers (DJI Phantom 2 link), no analog FM video path. [P]
- Reddit r/AskElectronics Feb 2026 "Alternative chips to the RTC6705": asked, no alternative offered. [P]
- LCSC RF-modulator category (126 parts): nothing above 5 GHz except IQ modulators. DigiKey: no RTC6705. [P]

Availability 2026-08-24 [P]:

| Source | Part | Stock | Price |
|---|---|---|---|
| LCSC C913074 (QFN-40-EP 6x6) https://www.lcsc.com/product-detail/C913074.html | RTC6705 | 0, "notify me" | reference $2.32 @1, $0.85 @1k |
| JLCPCB parts | C913074 | 0, pre-order | $2.44 est. |
| jlcsearch in-stock index | RTC6705, RTC67* | not present | |
| utsource | RTC6705 | in stock (claim) | $23 @1 |
| Win-Source, DigiPart aggregate | RTC6705A | 3k to 30k (broker claims) | $6.6 to $11.7 |
| AliExpress / Taobao / 1688 | RTC6705A AV05BMP | yes | $3.7 to $7.6 @1 |

Real 2026 price is $5 to $12 against a $0.85 to $2.3 reference: 3x to 10x [I]. Lifecycle "Active" per Avaq aggregator, Richwave still lists it [P]. Single source, fabless Taiwan, no second source [I].

Consequence: keeping the RTC6705 footprint and consigning broker parts is an option (~$6 to $8 per board at small volume [I]), but the question asked is how to build without it, so the rest is about a discrete synthesiser.

## 4. Discrete build: PLL + integrated VCO, video summed into the tune line

### 4.1 What the receiver needs

- RTC6715 (RX5808 and every analog goggle module): 5725 to 5865 MHz spec, 480 MHz IF, FM demodulator, sensitivity spec quoted at +-2.5 MHz test deviation, 8 MHz crystal, 1 MHz PFD, LO = RF - 479 MHz. https://assets.flitetest.com/article_files/RTC6715_1420104047.pdf [P]
- IF filter bandwidth is not in the RTC6715 datasheet (it is an external 480 MHz SAW/LC on the module) [P]. Community figures for the occupied bandwidth of an FPV VTX are roughly 20 to 27 MHz; Carson with +-4 MHz video deviation and a 6.5 MHz top subcarrier gives about 21 MHz [G].
- The receiver is a plain FM discriminator with sync clamping; it does not care whether the transmitter used an integer PLL, a fractional PLL or a doubler, only about carrier frequency (within the IF passband, so +-2 MHz of nominal is harmless [I]), deviation, and spectral purity within the channel.
- The RTC6705 deviation is not stated in its datasheet; it is set by the external video attenuator into VT_Mod on every VTX (typically a resistor divider from a 1 Vpp source) [P] so a discrete design can match it by trimming a resistor with a spectrum analyser or an RX5808 RSSI/video quality check [I].

### 4.2 Modulation method

The RTC6705 is itself "direct VCO modulation through a slow loop": PFD 20 kHz, loop bandwidth in the hundreds of Hz [I from P]. Any synthesiser with an external loop filter and an accessible VCO tune node can do the same:

- Sum the video (AC-coupled, attenuated to 100 to 300 mVpp depending on KVCO) into the Vtune node after the loop filter, through a series R or small C, so the charge pump filter does not load the video. [I]
- Loop bandwidth around 500 Hz. Frequencies above that ride on the open-loop VCO; below that the loop cancels them, and the receiver's back-porch clamp restores what is lost once per line at 15.6 kHz. Section 8.2 works the number and the filter out. [I]
- Deviation = KVCO x Vmod. KVCO values: MAX2871 ~100 MHz/V typical at 6 GHz (datasheet "VCO sensitivity") [P via jina extract]; LMX2572 VCO6 (5750 to 6400 MHz) 57 to 79 MHz/V, VCO5 (5200 to 5750) 61 to 82 MHz/V, datasheet table 135 [P]; ADF4355 ~15 MHz/V nominal (varies) [P via jina extract]; ADF4351 ~40 MHz/V at 2.2 to 4.4 GHz [P via jina extract], x2 after a doubler. For 8 MHz p-p: MAX2871 80 mVpp, LMX2572 100 to 140 mVpp, ADF4355 500 mVpp, ADF4351 100 mVpp at the VCO. All easy. [I]
- KVCO varies by VCO sub-band and by tune voltage (2:1 on LMX2572 within one core [P]). Deviation therefore changes with channel unless firmware scales the video attenuator (a DAC-controlled divider or a digital pot) using a per-band table, or the design tolerates +-30 % deviation. RTC6705 has the same effect over its 300 MHz span and nobody compensates it. [I]
- Vtune must stay inside the valid window (about 0.5 V to VCC-0.5 V on MAX2871 [P]) including the modulation swing: 0.1 to 0.3 Vpp is no problem. [I]
- VCO auto-calibration (band select) happens at every frequency write; modulation must be muted during calibration or the wrong sub-band gets picked. Mute video with an analog switch or by holding the video buffer at mid-level for ~1 ms after each write, then unmute. [I]
- Known failure report: ADI EngineerZone thread "Problem with excessive audio pre-emphasis when FM modulating the ADF4351": AC-coupling audio into TUNE gives a high-pass response below 1 kHz because the loop tracks it out; unanswered by ADI. https://ez.analog.com/rf/f/q-a/104488/problem-with-excessive-audio-pre-emphasis-when-fm-modulating-the-adf4351 [P] This is exactly the expected behaviour: the poster's loop bandwidth was in the kHz range. Video tolerates far more of it than audio does, because the receiver clamps per line rather than integrating, which is why 500 Hz works here and would not work for the ADF4351 poster's audio. [I]
- Two-point modulation (LF part through the fractional modulator, HF part through the VCO) is what phones do; no listed part accepts a 6.5 MHz analog modulation word, and the DC to 100 Hz part of video does not matter, so it is not needed. LMX2572's "FSK direct digital modulation" is discrete-level/pulse-shaped for wireless mics, not analog video. [P for LMX2572 feature, I for conclusion]
- Fractional-N sigma-delta spurs land at PFD-related offsets; with PFD >= 10 MHz they are outside the 21 MHz channel. Integer-N with a 1 MHz PFD (like the RTC6715 itself) gives 1 MHz steps, matches vtxtable resolution, no frac spurs, and OpenVTx's `freq/40` arithmetic just becomes `freq/1000`. Reference: 8 MHz crystal as on the RTC boards, or a 25/26 MHz TCXO. [I]
- Audio subcarrier(s): generate 6.0/6.5 MHz FM by MCU timer + varactor or skip. Most FPV VTX omit audio. [I]

### 4.3 Candidate synthesiser ICs (jlcsearch 2026-08-24, LCSC list prices qty 1 to 9)

| Part | Package | VCO fundamental | Vtune access | Out | LCSC | Stock | Price $ | DigiKey $ @1 | Verdict |
|---|---|---|---|---|---|---|---|---|---|
| MAX2871ETJ+T | TQFN-32 5x5 | 3 to 6 GHz, output 23.5 MHz to 6 GHz | CP and TUNE pins, external loop filter [P] | up to +5 dBm [P] | C7458627 | 752 | 9.94 (6.93 @100) | 18.03 | First choice. Covers 5645 to 5945 at fundamental. KVCO ~100 MHz/V. -101 dBc/Hz at 100 kHz open-loop VCO. 3.3 V, up to 200 mA with both outputs. [P] |
| MAX2870ETJ+T | TQFN-32 5x5 | same, older, worse PN [I] | same | +5 dBm | C7454677 | 222 | 11.16 | 19.79 | Second source for the same footprint: MAX2871 datasheet states it is "fully pin and software-compatible with the MAX2870" [P] |
| LMX2572RHAR | VQFN-40 6x6 | 3.2 to 6.4 GHz [P] | CPout and Vtune pins, external LF, needs >=1.5 nF at Vtune [P] | +4.5 dBm at 6.4 GHz [P] | C2665711 | 2220 | 10.21 (7.75 @100) | 32.87 (reel only), 26 wk lead | Viable alternate, best LCSC stock. Six VCO cores, KVCO 57 to 82 MHz/V in the FPV band [P]. The mandatory 1.5 nF at Vtune limits how you inject 6.5 MHz video (drive through ~10 ohm or inject before the cap) [I]. Larger package. |
| LMX2582RHAR | VQFN-40 6x6 | max 5.5 GHz [P] | | | C2864397 | 90 | 12.10 | 19.70 | Out: does not reach 5.6 GHz. |
| ADF4355BCPZ | LFCSP-32 5x5 | 3.4 to 6.8 GHz [P] | yes | -4 to +5 dBm | C578985 | 5 | 29.59 | 78.24 | Out on price and stock; needs 5 V VCO rail. KVCO ~15 MHz/V. |
| ADF4356 / ADF4371 | 5x5 / 7x7 | 6.8 / 32 GHz | | | C578986 / C654704 | 1 / 10 | 61 / 536 | | Out. |
| HMC833LP6GE | QFN-40 6x6 | 1.5 to 3 GHz VCO + internal doubler to 6 GHz [I] | yes | | C514408 | 174 | 15.82 | no stock | Viable but pricier, bigger; doubled VCO. |
| ADF4351BCPZ(-RL7) | LFCSP-32 5x5 | 2.2 to 4.4 GHz [P] | CP + VTUNE, external LF | -4 to +5 dBm | C654681 / C71362 | 289 / 191 | 14.79 / 14.26 | 25.16 (12.33 @100) | Needs external x2 doubler (see 5). |
| ADF4350BCPZ-RL7 | LFCSP-32 | same as 4351, worse PN | | | C98705 | 323 | 11.92 | | Cheapest ADI, doubler needed. |
| LMX2487SQ | WQFN-24 4x4 | none, external VCO, PLL to 6 GHz | n/a | | C2876370 | 272 | 5.62 | | For the discrete-VCO route (see 5). |
| ADF4113 | TSSOP-16 / LFCSP-20 | none, external VCO, PLL to 4 GHz | n/a | | C654605 / C208137 | 20 / 15 | 8.45 to 9.49 | | Integer-N, 4 GHz max: only with a 2.9 GHz VCO + doubler. Low stock. |
| STW81200T | VFQFPN-36 6x6 | to 6 GHz | | | C1527287 | 9 | 16.67 | | Low stock. |
| Chinese clones | | | | | none found under ADF4351/MAX2871/PLL/synthesizer/频率合成器/锁相环/VCO/5.8GHz on jlcsearch [P] | | | | The only LCSC-native RF synth-class parts above 1 GHz are ADI/TI/Maxim/HMC/Mini-Circuits. |

Datasheets: MAX2871 https://www.analog.com/media/en/technical-documentation/data-sheets/MAX2871.pdf , ADF4351 https://www.analog.com/media/en/technical-documentation/data-sheets/ADF4351.pdf , ADF4355 https://www.analog.com/media/en/technical-documentation/data-sheets/ADF4355.pdf , LMX2572 https://www.ti.com/lit/ds/symlink/lmx2572.pdf , HMC833 https://www.analog.com/media/en/technical-documentation/data-sheets/hmc833.pdf

Core area estimate for MAX2871 route [I]: 5x5 QFN + loop filter (3 caps, 2 R, 0402) + TCXO/crystal (2016 or 3225) + video summing network (3 parts) + output DC block and 3.3 V decoupling. About 9 x 9 mm excluding the MCU. LMX2572: 6x6 plus the same, about 10 x 10 mm.

### 4.4 Domestic Chinese synthesisers (off-LCSC, added on the second pass)

The LCSC-only sweep in 4.3 found no part under $9. Dropping that constraint changes the picture, because the Chinese domestic RF industry has integrated-VCO synthesisers that never reach LCSC and are sold through 世强 (Sekorm) and the vendors' own agents.

| Part | Vendor | Spec | Package | Source |
|---|---|---|---|---|
| **X214** | 重庆西南集成电路设计 (Chongqing SWIC) | frac-N, integrated VCO 1.5 to 3 GHz, RF out 25 MHz to 6 GHz, PFD 100 MHz, ref in to 250 MHz, noise floor FOM -226 frac / -230 int dBc/Hz, flicker FOM -264, 3.3 or 5 V, 36 to 65 mA at 3.3 V, 120 to 240 mA at 5 V, SPI | QFN40 6x6x0.75, 0.5 mm | [P] sekorm.com/news/42316599.html and the SWIC PLL selection table on the same page |
| CLF2574 | 核芯互联 | 10 MHz to 8.5 GHz, multi-core VCO, -231 dBc/Hz floor, 90 fs RMS jitter | not checked | [P via search snippet] eefocus |
| unnamed PLL family | 润积电 | to 12 GHz, integer and fractional, BiCMOS | QFN-24 4x4 | [P via search snippet] zhihu |
| XN406 | SWIC | frac-N, integrated VCO, RF out 25 to 3000 MHz | QFN40 6x6 | [P] sekorm.com/news/39802261.html |
| XN405 | SWIC | frac-N, external VCO, RF in to 8 GHz, <=80 mA | not checked | [P] sekorm.com/news/52893268.html |

X214 is the interesting one, for three reasons.

1. Its topology is the RTC6705's: a sub-3 GHz VCO with an on-chip multiplier to reach 6 GHz [I, from "integrated VCO 1.5-3 GHz" plus "RF output to 6000 MHz" on the same feature list]. Doubling doubles the FM deviation, so video injection at the tune node needs half the swing a fundamental 6 GHz VCO needs.
2. Its package is QFN40 6x6, byte for byte the RTC6705's package. Density parity is not an approximation, it is the same land pattern class.
3. On the vendor's own Q&A page, dated 2023-06-19, an engineer asks (translated) "I am making an FPV drone transmission module, I need a wideband VCO for the 3.3 GHz band, 3.2 to 3.6 GHz, preferably an integrated PLL+VCO chip, any domestic recommendation?" and is answered with X214. [P] Divimath's second output is 3.3 GHz. That is circumstantial and nothing more, but it is the only lead found on what the non-RichWave products contain.

Open before X214 can be designed in:
- The datasheet is behind a Sekorm member login (HTTP 457 to non-members). No public price, no public phase noise plot, no confirmation that the charge pump and tune node are brought out for external loop filtering, which is the whole requirement.
- SWIC is a CETC subsidiary. Divimath advertises NDAA compliance and builds in Thailand; a CETC part in the RF chain of an EU open-hardware VTX is a commercial and export decision, not only a technical one.
- Small-quantity supply to a Belgian buyer is unproven.

## 5. Alternatives: doubler, and discrete VCO with a cheap PLL

### 5.1 2.9 GHz synth + external doubler

ADF4351/ADF4350 at 2822 to 2973 MHz, then x2. Doubling doubles deviation (helps: only 4 MHz p-p at the VCO) and adds 6 dB to phase noise [I]. Problems: no cheap doubler IC on LCSC (HMC575 is 3 to 4.5 GHz input and $37, CY2-143+ 4 to 14 GHz $9, both low stock [P]); a passive or BJT doubler (BFP840/BFR193-class transistor driven into class C, output tank at 5.8 GHz, then the band-pass filter) costs 6 to 10 parts and 3 x 6 mm, and the fundamental leak at 2.9 GHz must be filtered to -60 dBc [I]. This is what the RTC6705 does internally (2.9 GHz VCO, "2G VCO" registers, "5G" pre-driver) [I from P]. Only worth it if the 6 GHz synths vanish; ADF4351 clones are not on LCSC either, so no cost advantage.

### 5.2 Discrete VCO + PLL (pre-RTC6705 style)

LCSC has an unexpected item: Innotion (Shenzhen) YSGM VCO modules, SMD 9x7x2 mm, 0 to 5 V tune, 4.2 to 6 V supply [P via LCSC product page]:

| Part | LCSC | Range | Pout | Stock | Price $ |
|---|---|---|---|---|---|
| YSGM556006 | C52043380 | 5320 to 6060 MHz | >= 6 dBm at 5 V | 1982 | 1.39 (0.89 @100) |
| YSGMTC5800 | C52043383 | 5430 to 6060 MHz | >= 7 dBm at 5 V | 1901 | 2.56 (1.70 @100) |
| YSGM515906 | C52043378 | not fetched, name suggests 5150 to 5900 [G] | | 1958 | 1.39 |

Phase noise, KVCO and pushing are not on the LCSC page [P]; the range implies ~150 MHz/V [I], so video drive is ~50 mVpp and supply noise matters. Lock it with LMX2487 (C2876370, 4x4, $5.62, 272 stock, 6 GHz frac-N, external VCO) [P]. Core area: 9x7 VCO + 4x4 PLL + loop filter + reference, about 10 x 16 mm [I], so it misses the 10 x 10 target. This is architecturally the 2010-era Boscam/Airwave approach (see 5.3) with a module VCO. Cheapest BOM (~$8) but two unknown-quality parts and a 5 V rail. A fully discrete transistor+varactor VCO at 5.8 GHz on JLCPCB FR-4 is not repeatable enough for assembly without per-unit tuning [I].

### 5.3 What pre-RTC6705 VTXs used

No documented 5.8 GHz FPV transmitter without an RTC6705 was found. Boscam TX5813 (10 mW) and TX5823 (200 mW) spec V1.3 dated 2010-09-02 (skytech.ir/DownLoad/File/6640_TX5823-Spec-V1.pdf, foxtechfpv.com/product/5.8G%20modules/tx5813/TX5813-Spec-V1.pdf): 8 channels 5705 to 5945 MHz on 3 parallel pins, 6.5 MHz audio subcarrier, 5 V 170 mA, 22x19 mm [P]. That 3-pin table is the RTC6705 pin-select mode and the chip datasheet predates the module (Sep 2007), so TX5813 = RTC6705, TX5823 = RTC6705 + PA [I]. Boscam TS351 (2012 manual, rigpix.com/atv/boscam_ts351_manual.pdf) is a TX5823 in a box [I]; TS832 was made by Skyzone (Oscar Liang 2014) [P], RTC6705 inside. ImmersionRC 25 mW (2010) was "based on an Airwave module" (retailer claim) [P]; Airwave AWM661TX/AWM6W5V_TX datasheets say "built-in worldwide 5.8GHz ISM band RF IC", PLL synthesiser, 3-bit channel pins, chip not named (airwave.com.tw/en/product/file/614397-1) [P], most likely RTC6705 [G]. Lawmate and FatShark "NexwaveRF": no internal photos on fccid.io, nothing verified [P].

The only pre-integration architecture on record is amateur 5.7 GHz ATV: a 2.7 to 3 GHz VCO plus doubler or a 5.8 GHz VCO plus prescaler, locked by a 1.2 GHz PLL (Fujitsu MB15E03SL, 64/65 prescaler) or a 4 GHz PLL (ADF4113) with an external /2 or /4 (HMC433E, DC to 8 GHz), video summed into the tune line (YO4HFU, LMX2326-based builds) [P for the chips, G that any commercial FPV module used them]. Today: MB15E03 LCSC C21579 out of stock and obsolete at DigiKey [P]; ADF4113 20 pcs (C654605) [P]; HMC433E C455140 18 pcs $10.03 [P]. Dead end for a JLCPCB build.

## 6. Output chain on LCSC (jlcsearch 2026-08-24)

A synth gives +5 dBm max. The RTC6705 gave +13 dBm, and existing VTX PA stages expect that. Two options: one 12 dB gain block to recreate the +13 dBm node and keep a known PA, or feed a 27 to 32 dB WLAN PA directly from the synth (5 dBm + 27 dB = 32 dBm before back-off, which is more than a 400 mW class needs; the PA bias/Vcc control sets the level as OpenVTx already does) [I].

| Role | Part | Package | Spec | LCSC | Stock | Price $ @1 | Basic? |
|---|---|---|---|---|---|---|---|
| Gain block | TRF37A73IDSGR (TI) | WSON-8 2x2 | 1 MHz to 6 GHz, 12 dB, P1dB 14.5 dBm at 2 GHz (lower at 5.8 [G]), 3.3 V 65 mA | C2151364 | 5024 | 1.18 (0.85 @100) | no |
| Gain block | TRF37C73IDSGT | WSON-8 2x2 | 18.5 dB, P1dB 16.5 dBm | C2654174 | 46 | 1.07 | no |
| Gain block | GRF2505 (Guerrilla RF) | DFN-6 1.5x1.5 | 4 to 6 GHz LNA/driver, 11.3 dB, P1dB 19 dBm | C20616772 | 34 | 6.34 | no |
| Gain block | PMA3-83LNW+ (Mini-Circuits) | QFN-12 | 0.4 to 8 GHz, 20.5 dB, P1dB 20.5 dBm, 5 V | C5200777 | 2013 | 9.51 | no |
| PA 25 to 400 mW | SKY85712-21 (Skyworks) | QFN-16 3x3 | 5.15 to 5.85 GHz FEM: PA 27 dB, +19 to 20 dBm linear WLAN, switch P1dB ~25 dBm, 5 V 275 to 330 mA | C2654407 | 5 | 0.88 | no |
| PA 25 to 400 mW | SKY85405-11 | QFN-20 4x4 | 5 GHz InGaP PA, 5 V; LCSC file is a 2-page brief, no gain/P1dB [P] | C2151489 | 100 | 2.25 | no |
| PA 1 W class | QPA9501TR13 (Qorvo) | QFN-20 4x4 | 32 dB at 5800 MHz, P1dB 29.5 min / 33 typ dBm, 3.3 to 5 V 520 mA [P] | C2911573 | 261 | 5.82 | no |
| PA 1 W class | TQP5525 (Qorvo) | QFN-20 4x4 | 32 dB, P1dB 32 dBm, 350 mA, power detector [P] | C471153 | 28 | 6.01 | no |
| PA 1 W class, Chinese | GWQ5929A (GPowerTek) | QFN-20 4x4 | 31 dB, Psat 34 dBm [P] | C41410383 | 51 | 4.96 (3.09 @1500) | no |
| FEM | RFFM4558TR7 (Qorvo) | QFN-16 2.5x2.5 | 32 dB TX, 24 dBm MCS0, integrated filter + detector [P] | C43526386 | 30 | 2.27 | no |
| Gain block | GVA-63+ (Mini-Circuits) | SOT-89 | 15.9 dB, P1dB +11.8 dBm at 6 GHz, 5 V 69 mA [P] | C3193270 | 6764 | 2.04 | no |
| Gain block | GVA-83+ | SOT-89 | 12.3 dB, P1dB +18.1 dBm at 6 GHz, 5 V 72 mA [P] | C3193256 | 4 | 4.49 | no |
| Gain block | TQP3M9037 (Qorvo) | DFN-8 2x2 | 0.7 to 6 GHz, 20 dB, P1dB 20 dBm (spec at 1.9 GHz) [P] | C415712 | 2477 | 2.93 | no |
| Gain block | QPL9547TR7 (Qorvo) | DFN-8 2x2 | 0.1 to 6 GHz, 16.8 dB, P1dB +23 dBm at 5.1 GHz [P] | C5367093 | 0 LCSC / 2663 JLC | 2.05 | no |
| Gain block | SKY65017-70LF | SOT-89 | 0.1 to 6 GHz, 20 dB / 20 dBm at 2 GHz, +-1.5 dB to 6 GHz, 5 V 120 mA [P] | C2649469 | 5345 | 2.16 | no |
| PA 25 to 400 mW | SKY85717-11 | QFN-16 2.5x2.5 | 5 GHz, 28 dB | C2654452 | 41 | 2.73 | no |
| PA 25 to 400 mW | SKY85743-21 | FEM | 5 GHz LNA+PA+switch | C5348950 | 729 | 2.91 | no |
| PA 400 mW to 1 W | SE5004L-R (Skyworks) | QFN-20 4x4 | 5.15 to 5.85 GHz, 32 dB, P1dB 30 to 34 dBm, Psat 26 dBm at 5 V, 600 to 800 mA, power detector | C210263 | 1866 | 7.77 (5.38 @100) | no |
| PA | QPA9501 (Qorvo) | QFN-20 4x4 | 5 GHz WiFi PA with detector | C2911573 | 263 | 5.80 | no |
| PA | PHA-83W+ (Mini-Circuits) | SOT-89 | 50 MHz to 8 GHz, 15.7 dB, P1dB 23.3 dBm, 9 V | C20231740 | 72 | 11.97 | no |
| Band-pass | RFBPF1608060K98Q1C (Walsin) | 1608 3P | 5150 to 5950 MHz, 0.6 dB IL, 40 dB rej [P] | C2442150 | 13370 | 0.074 | no |
| Band-pass | DEA165538BT-2236B1-H / -2263A1-H (TDK) | 1608 3P | 5150 to 5925 MHz, 1.16 / 0.63 dB, 31.5 / 38 dB rej [P] | C2651072 / C2835388 | 925 / 100 | 0.22 / 0.18 | no |
| Band-pass | DEA165363BT-2124A3 (TDK) | 1608 4P | 4900 to 5825 MHz, 1.1 dB, 49 dB rej [P] | C307897 | 7890 | 0.14 | no |
| Band-pass | LFB185G37CF2D114 (Murata) | 1.6x0.8 4P | 4.9 to 5.84 GHz, 1.5 dB | C2766051 | 4706 | 0.15 | no |
| Band-pass | BPF1608LM08R5000A (Yageo) | 1608 3P | 4900 to 5840 MHz, 1.5 dB, 35 dB [P] | C513637 | 3440 | 0.036 | no |
| Low-pass | LFCN-5850+ (Mini-Circuits) | 3216 | DC to 5850, fco 6.54 GHz [P] | C2683450 | 100 | 3.01 | no |
| Coupler | TFSC06054125-2111C1X (TDK) | 0605 | 5.15 to 5.85 GHz directional coupler, for a power detector [P] | C2833716 | 7464 | 0.15 | no |
| Triplexer as BPF | TPX255850MT-7013A3 (TDK) | 2.5x2.0 | high band port 5150 to 5850 MHz, 0.35 dB IL, 13 to 28 dB rejection | C531312 | 3795 | 0.22 | no |
| Band-pass | BFCN-5750+ (Mini-Circuits) | 3.2x1.6 | 5650 to 5850 MHz, 1.84 dB | C4989833 | 139 | 8.72 | no |
| Band-pass | BFCG-5600+ | 2x1.2 | 5150 to 5990 MHz, 1.2 dB | C4989507 | 6 | 9.39 | no |
| Low-pass | LFCN-6000+ (Mini-Circuits) | 3.2x1.6 | fc 6.8 GHz | C879870 | 74 | 3.14 | no |
| Not on LCSC | RFPA5542 (Qorvo, BetaFPV PA): EOL announced 2023-10-18, DigiKey none [P]. SKY85747-11 (34.5 dB, 27 dBm MCS0, the best fit) not on LCSC or DigiKey [P]. SKY65135 and SE2623L are 2.4 GHz parts [P]. Richwave RTC76401/76402S not indexed [P]. Johanson 5515BP15B200 none; 5515BP15B0725001E 5150 to 5875 MHz 0805 is DigiKey only, $0.69 [P]. | | | | | | |

Observations:
- No JLCPCB basic part in this table; everything is extended (one-off feeder fee each) [P].
- Every cheap 5 GHz PA is a WLAN part rated 5150 to 5850 or 5925 MHz. Band E top (5945 MHz) and Raceband 8 (5917 MHz) sit at or above the rated band; gain roll-off there is a few dB and unspecified [I]. Filters: pick the Walsin RFBPF1608060K98Q1C (5150 to 5950) or TDK DEA165538 (5150 to 5925) so the whole FPV table is inside the passband; the 4900 to 5840 parts clip band E high channels [I].
- 25 mW EU build: MAX2871 plus one gain block, no PA. The +5 dBm output figure is specified at 3000 MHz, so section 8.5 budgets +2 dBm at 5.8 GHz, which needs about 16 dB of gain to reach +14 dBm at the connector. 400 mW: add SE5004L or QPA9501 (both 32 dB, in stock) with Vcc/bias from the MCU PWM as OpenVTx already does [I].
- Harmonics: a synth's square-ish output has strong 2nd/3rd harmonics (MAX2871 datasheet: -40 dBc second, -34 dBc third at the fundamental output [P]); the RTC6705 spec was -60 dBc after its reference filter, so the low-pass or band-pass after the gain block is mandatory, and a second one after the PA for the 400 mW build [I].

## 7. Candidate approaches compared

| Approach | Core parts | LCSC / stock / price | Core area | Modulation | Firmware effort | Risk |
|---|---|---|---|---|---|---|
| A. Keep RTC6705(A), consign broker stock | RTC6705A, 8 MHz xtal | LCSC C913074 0 stock; brokers $5 to $12 | 6x6 QFN + 3 parts, 8x8 mm | proven | none (OpenVTx as is) | single source, price volatility, counterfeit risk from brokers, no future |
| B. MAX2871 (or MAX2870) fundamental synth, video into TUNE | MAX2871, 26 MHz TCXO (REF_IN minimum is 10 MHz, so no 8 MHz crystal), loop filter, unity-gain video buffer, Cinj, mute | C7458627, 752, $9.94 ($6.93 @100); DigiKey 8.5k | ~9x9 mm | KVCO 100 MHz/V, 39 pF into TUNE gives +-3.8 MHz from 1 Vpp; loop BW 505 Hz; manual VCO band select | new driver: 6 x 32-bit registers, integer-N, band table; SmartAudio/Tramp/MSP layers unchanged | KVCO per band changes deviation; VAS unusable at this loop bandwidth (8.4); 200 mA at 3.3 V vs 95 mA; DigiKey price 2x LCSC |
| C. LMX2572 fundamental synth, same modulation | LMX2572RHAR + same periphery | C2665711, 2220, $10.21 ($7.75 @100); DigiKey 26 wk | ~10x10 mm | KVCO 57 to 82 MHz/V; Vtune pin needs 1.5 nF shunt which fights 6.5 MHz injection | new driver, ~110 registers but TI gives a TICS Pro register dump | bigger package, less community use, injection point awkward |
| D. ADF4351 at 2.9 GHz + discrete x2 doubler | ADF4351 + BJT doubler + 5.8 GHz tank + BPF | C654681, 289, $14.79 | 5x5 + 3x6 doubler, ~10x12 mm | 4 MHz p-p at VCO, 40 MHz/V, 100 mVpp; doubler doubles PN | driver exists in many hobby projects | more expensive than B, more RF parts, fundamental leak, no reason to prefer |
| E. Innotion YSGM VCO module + LMX2487 PLL | YSGM556006 + LMX2487 + loop filter + ref | C52043380 1982 $1.39; C2876370 272 $5.62 | 9x7 + 4x4, ~10x16 mm | ~150 MHz/V [I], 50 mVpp; supply pushing unknown | LMX2487 driver, simple | unknown phase noise and pushing, 5 V rail, exceeds area target, module vendor risk |
| F. Fully discrete transistor+varactor VCO + PLL | BFP840-class + SMV varactor + LMX2487 | parts on LCSC | 8x8 + 4x4 | direct varactor drive | as E | not repeatable on JLCPCB FR-4 without tuning, EMC spurs; only for hobby builds |

## 8. The design

Decisions taken 2026-08-24: FPV table only, 5645 to 5945 MHz, no 4.9 to 6.0 GHz wideband and no 3.3 GHz second band. MAX2871 for the prototype, with the MAX2870 as the pin- and software-compatible fallback [P] and the domestic parts of 4.4 pursued in parallel as a production cost-down. LMX2572 stays the second-source layout if Maxim stock disappears; different footprint, so a board variant rather than a swap.

Every value in this section comes from the MAX2871 datasheet or is computed from it. The design targets the same job the RTC6705 does, not more.

### 8.1 Frequency plan

- REF_IN accepts 10 to 210 MHz [P], so the 8 MHz crystal on RTC6705 boards cannot be reused. Use a 26 MHz TCXO, R = 26, PFD = 1 MHz.
- Integer-N, N = 5645 to 5945, one count per MHz. The vtxtable's integer-MHz entries map straight onto N, and OpenVTx's `freq_kHz/40` arithmetic collapses to `N = freq_MHz`. Set F01 = 1 and LDF = 1 for integer-N lock detect (5 locked cycles) [P].
- Output on the fundamental, DIVA = 1. The VCO covers 3000 to 6000 MHz gap-free across 4 cores x 16 sub-bands [P], so the whole FPV table sits comfortably inside one region.
- Integer-N means no fractional spurs anywhere. In-band phase noise = -230 + 10log10(1e6) + 20log10(5795) = -95 dBc/Hz [I from P], and that only applies below the loop bandwidth.
- Reference spurs land at 1 MHz offset. The datasheet quotes -88 dBc with a 50 kHz loop [P]; this loop is a hundred times narrower, with an open-loop gain of -167 dB at 1 MHz against -21 dB for a 50 kHz loop [I, computed], so the ripple that produces those spurs is far better filtered here.
- VCO phase noise at 6000 MHz is -101 dBc/Hz at 100 kHz and -128 at 1 MHz [P], against the RTC6705's -90 and -115 [P]. The synth route is 11 to 13 dB quieter than the part it replaces.

### 8.2 Loop filter

The loop bandwidth is the one number that decides whether this works. It must be low enough that the loop does not cancel the video, and high enough to lock inside OpenVTx's 500 ms budget.

The usual reflex is to push the loop under 50 Hz so it cannot track the field rate. That is wrong here. Every analog FPV receiver clamps on the back porch, so DC restoration happens once per line at 15.6 kHz, not once per field. The loop only has to be far below the line rate, and whatever it removes below a few hundred Hz the clamp puts back. 500 Hz is the right target, and it keeps lock time in milliseconds.

Third-order passive filter, RSET = 10 kohm and CP[3:0] = 0000 so ICP = 1.63/RSET x (1 + CP) = 163 uA [P, formula and RSET range 2.7 to 10 kohm], KVCO = 100 MHz/V [P], N = 5795:

| Element | Value | Note |
|---|---|---|
| C1 (CP_OUT to GND) | 100 nF | X7R 0603 |
| R2 | 1.3 kohm | split 330 ohm + 1.0 kohm if the built-in fast-lock (SW pin) is populated |
| C2 | 680 nF | X7R 0603 |
| R3 | 56 kohm | third pole, 6.0 kHz corner |
| C3 (at TUNE) | 470 pF | C0G |

Computed response: crossover 505 Hz, phase margin 45.8 degrees, loop gain +31.6 dB at 50 Hz, -7.7 dB at 1 kHz, -48.8 dB at 10 kHz, -167 dB at 1 MHz. [I, numeric check] Lock time from a full 300 MHz step is roughly 5 ms, forty times faster than needed, so the datasheet's fast-lock mode (CDM = 01, R2 split 1:3 onto the SW pin) is optional. Leave the two-resistor footprint and the SW pad, and leave SW open on the first build so both resistors stay in circuit.

R3 at 56 kohm contributes 30 nV/rtHz, which after its own 6 kHz corner integrates to 297 Hz rms of FM, 79 dB below a +-3.8 MHz signal. [I, computed] Thermal noise in the loop filter is not a limit.

### 8.3 Video injection

Inject with a series capacitor into the TUNE node, not a series resistor. The capacitor and C3 form a divider that is flat from well below the loop bandwidth to far above 6.5 MHz, so the deviation does not vary across the video band, and the same part does the DC blocking.

With C3 = 470 pF and a unity-gain buffer driving 1 Vpp of composite video:

| Cinj | Divider | Deviation | Buffer load at 6.5 MHz | Buffer peak current |
|---|---|---|---|---|
| 33 pF | 1/15.2 | 6.6 MHz p-p | 794 ohm | 0.63 mA |
| **39 pF** | **1/13.1** | **7.7 MHz p-p, +-3.8 MHz** | **680 ohm** | **0.74 mA** |
| 47 pF | 1/11.0 | 9.1 MHz p-p | 573 ohm | 0.87 mA |

39 pF lands on the community figure of about +-4 MHz with the CVBS source at its native 1 Vpp, so the buffer runs at unity gain and no attenuator is needed. Trim deviation by changing the buffer gain, never by changing Cinj: keeping the RF-side network fixed keeps the loop response fixed. A digital pot or a PWM-DAC in the buffer's feedback makes deviation a firmware parameter, which is the clean way to compensate the KVCO spread across the band.

Any rail-to-rail op amp with a few tens of MHz of gain-bandwidth drives 680 ohm at 0.74 mA. 75 ohm termination on the CVBS input as usual.

### 8.4 VCO band select: manual, not automatic

This is the trap in the whole design and it will not show up until a prototype misbehaves on some channels.

The MAX2871's VCO autoselect state machine runs at a fixed 50 kHz clock [P], so it evaluates a sub-band every 20 us. A 500 Hz loop moves the TUNE voltage with a time constant near 320 us. VAS therefore reads the tune voltage before it has settled and can select the wrong sub-band. The datasheet's own spur and lock specs are taken with a 50 kHz loop, a hundred times wider than this one.

Use manual selection, which the datasheet supports directly: VAS_SHDN = 1 with the band in register 3 bits 31:26 [P]. The documented procedure builds the lookup table by running VAS once per frequency at power-up and reading back register 6 bits 8:3 [P], but that inherits the same settling problem here. Build the table with the on-chip Tune ADC instead:

1. ADCM = 100 selects the TUNE pin, ADCS = 1 starts a conversion, CDIV = fPFD/100 kHz, read register 6 bits 22:16 after 100 us. V = 0.315 + ADC x 0.0165. [P]
2. For a handful of anchor frequencies across 5645 to 5945, binary-search the sub-band that puts TUNE nearest mid-window. The valid window is 0.5 V to VCC [P], so aim for about 1.7 V.
3. Interpolate between anchors for the rest of the table. The FPV span is 300 MHz out of a 3 GHz range, so only a few sub-bands are in play and the table is small.
4. On every channel change, read the ADC once after lock and step a band if TUNE has drifted toward either rail. That also covers temperature, and the on-chip temperature sensor is available on the same ADC if a feed-forward correction is wanted later.

Cost of this: zero parts, some firmware. The alternative, an analog switch that widens the loop during acquisition, is a fallback if the ADC approach proves fiddly.

### 8.5 Output chain

MAX2871 outputs are differential open-collector and need an RF choke or a 50 ohm pullup per side [P]. Single-ended off RFOUTA with a choke, RFOUTB tied to VCC_RF. APWR = 11 gives +5 dBm, but that is specified at 3000 MHz [P]; budget +2 dBm at 5.8 GHz until measured.

Harmonics at the fundamental output are -40 dBc second and -34 dBc third [P]. The RTC6705 delivered -60 dBc after its reference filter [P], so a band-pass after the synth is mandatory, not a nicety. The Walsin RFBPF1608060K98Q1C (5150 to 5950 MHz, 0.6 dB, 40 dB rejection, $0.074, 13k stock) covers the entire FPV table with margin and does it in an 1608.

- 25 mW EU build: 25 mW is +14 dBm at the connector. The synth's +2 dBm plus TRF37A73's 12 dB lands at +12.4 dBm after the band-pass and connector, which is 17 mW, and APWR only trims downward, so that combination cannot reach the target. Use a higher-gain block and trim down instead: QPL9547 (16.8 dB, P1dB +23 dBm specified at 5.1 GHz, DFN-8 2x2, $2.05, 2663 at JLCPCB) gives about +17 dBm, leaving 3 dB of APWR headroom. SKY65017-70LF (20 dB, flat to +-1.5 dB at 6 GHz, SOT-89, 5 V 120 mA, $2.16, 5345 stock) is the alternative. GVA-63+ is out: its P1dB is +11.8 dBm at 6 GHz, below the target. No PA on this build.
- 400 mW to 1 W build: add QPA9501 after the first band-pass (32 dB at 5800 MHz, P1dB 29.5 dBm min, 33 dBm typ [P]), then a second band-pass or low-pass. Power is set by PA bias from the MCU PWM exactly as OpenVTx already does, so `target_set_power_dB` and its per-board table carry over unchanged.

Pit mode: MTLD (register 4 bit 10) mutes the outputs whenever digital lock detect is low, and muted output power is -40 dBm [P]. That replaces OpenVTx's "PA off, wait 500 ms, PA on" dance with a hardware interlock. Keep the PA bias at zero on top of it, since -40 dBm through a 32 dB PA is still -8 dBm.

### 8.6 Supply and isolation

Open-loop frequency pushing is 0.8 MHz/V [P], and above the 500 Hz loop bandwidth the loop does not correct it. That sets a hard budget on the VCO rail:

| Supply noise on VCC_VCO | Resulting deviation |
|---|---|
| 26 uVrms (RT9080-class LDO) | 21 Hz |
| 1 mV | 800 Hz |
| 10 mV (typical buck ripple) | 8 kHz |

8 kHz of FM at the buck's switching frequency is a visible pattern on the picture. A dedicated quiet LDO for VCC_VCO with a ferrite from the digital rail is required, and the PA's switching current must not share it.

Open-loop pulling into a 2:1 VSWR is 70 kHz [P], 1.8 % of the deviation, so antenna loading is harmless. PA-to-VCO leakage on a 1 W build is the real isolation problem and is what the RF can over the synth and gain block is for.

### 8.7 Firmware contract

`rtc6705.c` becomes `max2871.c`. Everything above it, SmartAudio, Tramp, MSP, the OSD, is untouched.

- Six 32-bit registers R0 to R5, MSB first, the low 3 bits are the address, programmed R5 down to R0, R0 last because writing it triggers the frequency update [P].
- Frequency set: write the band from the table into R3, then N into R0.
- Replace the fixed 500 ms `PLL_SETTLE_TIME` with the digital lock detect on MUX, plus MTLD as the hardware backstop.
- Power stays external: APWR for the fine trim, PA bias PWM for the coarse level.
- Register 6 bits 31:28 read 0111 for MAX2871 and 0110 for MAX2870 [P], so one firmware image can detect which part is fitted.

### 8.8 What the prototype has to prove

Nothing in section 8 is hard except one thing that no public source documents: whether composite video summed into a synthesiser's tune node demodulates cleanly in an RX5808. Build a board that answers only that, and leave the PA off it.

Synth, TCXO, loop filter, video buffer, Cinj, u.FL, and any MCU with SPI. Measure occupied bandwidth and harmonics on a tinySA Ultra, then put an RX5808 on it and look at the picture and the RSSI against a known RTC6705 VTX at the same deviation. If the picture is clean, everything left is layout and sourcing.

### 8.9 Cost, honestly

Against the historic $0.85 to $2.30 catalogue price for an RTC6705, this loses and always will. Against the price that actually exists in 2026, $5 to $12 from brokers with no datasheet guarantee and no second source, the core RF BOM is roughly $10.25 at qty 100: MAX2871 $6.93, 26 MHz TCXO about $0.50, QPL9547 gain block $2.05, Walsin band-pass $0.07, loop filter and injection passives about $0.10, buffer and mute about $0.60. That is $3 to $4 per board over a broker RTC6705, for real stock, a public datasheet, a pin-compatible second source, and 11 dB better phase noise. The domestic route of 4.4 is where that gap closes, if the parts turn out to be buyable.

Block diagram:

```
 CVBS in 1 Vpp -> 75R term -> unity-gain buffer (mute to mid-rail during band search)
                                         |
                                     Cinj 39 pF
                                         |
 26 MHz TCXO ---------------> REF_IN     v
 MCU SPI (CLK/DATA/LE) + CE -> MAX2871 (R=26, PFD 1 MHz, integer-N, N = 5645..5945)
 MUX (lock detect + reg readback) <-|  CP_OUT -> C1 100n, R2 1k3 + C2 680n, R3 56k -> TUNE (C3 470p)
                                       RFOUTA (choke to VCC_RF, APWR 11)
                                            |
                                       DC block -> QPL9547 +16.8 dB -> Walsin RFBPF1608060K98Q1C
                                            |                             |
                             25 mW EU build ends here (+14 dBm)           v
                                                       QPA9501 +32 dB (bias from MCU PWM) -> 2nd BPF -> u.FL/MMCX
```

## 9. Regulatory note

In the EU the 5725 to 5875 MHz non-specific SRD band (ERC/REC 70-03 Annex 1, EN 300 440) allows 25 mW e.i.r.p.; FPV bands A/B/E/F/R span 5645 to 5945 MHz, so channels below 5725 (E band low, Raceband 1 and 2) and above 5875 (E band high, Raceband 8) are outside the SRD band and only tolerated because nobody enforces it against hobbyists [P for the band limits via CEPT EFIS/ECO docs, I for the FPV channel overlap]. A discrete synthesiser changes the emissions picture in three ways: the synth output has a harmonic content the RTC6705's internal pre-driver plus reference filter cleaned to -60 dBc, so an explicit filter after the gain block is not optional; a fractional-N modulator adds spurs at PFD-related offsets that can land in adjacent FPV channels and must be verified in-band with the 21 MHz Carson bandwidth; and reference/TCXO harmonics and the MCU clock ride on the tune node as FM sidebands unless the loop filter and supply are clean. Because the RF path is now three chips instead of one, the shielding can (a single RF can over synth + gain block + filter) is what makes EN 300 440 spurious limits (-30 dBm above 1 GHz, 4 nW e.r.p.) passable without measurement surprises [I]. Frequency agility beyond the FPV table (the Divimath selling point) is an emissions liability under CE: firmware should hard-limit to 5645 to 5945 or, for a strictly compliant SKU, 5725 to 5875.

## 10. Unverified

Resolved on the second pass, no longer open: MAX2871 KVCO (100 MHz/V), harmonics (-40 / -34 dBc), phase noise, charge-pump formula and RSET range, PFD and REF_IN limits, tune voltage window, frequency pushing and pulling, VAS clock, manual band select, Tune ADC transfer function and the register map are all read directly from the datasheet now.

Still open:

- **The central one.** No source anywhere describes composite video summed into a MAX2871 or ADF4351 tune node. The ADF4351 audio thread confirms the loop's high-pass behaviour and nothing else. The 500 Hz loop bandwidth, the clamp argument in 8.2 and the deviation figure are engineering expectations. Section 8.8 exists to settle them.
- FPV video deviation (about +-4 MHz) and receiver IF bandwidth (20 to 27 MHz) are community figures. The RTC6705 datasheet does not state deviation; the RTC6715 datasheet gives 480 MHz IF and a +-2.5 MHz test deviation only.
- The loop filter of 8.2 is computed and numerically checked, not simulated in EE-Sim and not built. Lock time of about 5 ms is an estimate.
- VAS mis-selecting sub-bands at a 500 Hz loop bandwidth is reasoned from the fixed 50 kHz state machine clock against the loop time constant, not observed. If VAS turns out to work, 8.4 gets simpler.
- MAX2871 output power is specified at 3000 MHz. The +2 dBm assumed at 5.8 GHz is a guess pending measurement.
- X214: datasheet is behind a Sekorm member login and was not read. The on-chip multiplier, whether the charge pump and tune node are brought out for an external loop filter, phase noise, price, and small-quantity availability outside China are all unconfirmed. The link between X214 and any shipping FPV product is circumstantial.
- CLF2574 and the 润积电 parts come from search snippets, not datasheets.
- That the RTC6705 VCO runs at half frequency with a doubler: inferred from FRF = 2*(N*64+A)*Fpfd and the 2G/5G register names, not stated.
- Divimath's actual architecture is undisclosed. Whether Foxeer, iFlight, Rush, TBS, Happymodel or Walksnail analog VTX use the RTC6705 is unresolved; section 1.1 argues they can, not that they do. No teardown or FCC internal photo was found for any of them.
- Innotion YSGM VCO phase noise, pushing and KVCO are not on the LCSC page.
- 5 GHz WLAN PA and ceramic filter behaviour above 5850 MHz is outside their rated band, no data. Band E channel 8 at 5945 MHz and Raceband 8 at 5917 MHz are both affected.
- LCSC stock and prices are jlcsearch mirror values on 2026-08-24; DigiKey prices are from the same day.
- EU SRD 5725 to 5875 MHz at 25 mW e.i.r.p. is confirmed only through search snippets of CEPT and ECO pages, not the current ERC/REC 70-03 Annex 1 text.
