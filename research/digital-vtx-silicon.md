---
topic: Product
status: Open
owner: [Stan Coene]
reviewed: 2026-10-07
---

# Which silicon and firmware stack should the OpenDrone digital VTX use?

**Answer:** Phase 1 open. Primary candidates: SigmaStar SSC338Q (OpenIPC) or Rockchip RV1126B (OpenHD), each with an RTL8812EU radio module. Hedges: Divimath HDZero chipset (goggle-compatible on day 1) and Artosyn ARLink (Ascent-class, gatekept). Decision waits on bench data (latency p95, stutter) and written firmware-partner terms.

**Goal**

A plug-and-play digital VTX and VRX for OpenDrone at least on par with Caddx Ascent Lite+. Fully open where the silicon allows; where it does not, transparent: schematic, layout, BOM, measurement method and raw data published, every closed blob named, no geofencing and no kill switch.

**Benchmark to match: Caddx Ascent Lite+**

| Spec | Ascent Lite+ |
| --- | --- |
| Chipset | Artosyn AR8030 |
| VTX price | $39.99 |
| Weight | 10.5 g cased, 6.35 g decased |
| Mount, input | 20x20, 3 to 12.6 V |
| Latency | 35 ms average claimed, not independently measured; reviewers rate it above Avatar and DJI |
| Usable mode | 1080p60; 120 fps not usable on tested goggles |
| Bitrate | 25 Mbps max |
| TX power | FCC 27 dBm, CE under 14 dBm |
| Weak points | Random stutter, 450 ms audio delay, no DVR |

**Phase 1 decision matrix (desk evaluation, 2026-10-07)**

| Criterion | A1: SSC338Q + 8812EU (OpenIPC) | A2: RV1126B + 8812EU (OpenHD) | B1: Divimath DM6300/DM5680 | B2: Artosyn AR8030 |
| --- | --- | --- | --- | --- |
| Openness | Firmware open, SigmaStar SDK under NDA, binary blobs | Firmware open, Rockchip SDK and datasheets circulate, ISP tuning files gated | Closed chips and firmware, design publishable to be confirmed | Closed baseband and SDK, gatekept by Artosyn and KAP |
| Sourcing | SoC via Chinese channels; open reference board exists (Thinker338Q: PDF schematic and Gerbers, no licence) | RV1126 (non-B) on LCSC C2859017, PCBA only; RV1126B LCSC listing not verified | Divimath sales, terms unknown | Modules over 80 EUR per unit reported by OpenHD |
| Latency | 26 to 67 ms G2G (community reports); jitter reported on yaw | No published number; OpenHD lead claims low on RV1126 but publishes none | Lowest of any digital system | Ascent-class, about 35 ms claimed |
| Image | Up to 4K encode, 1080p60 practical | 4K30 encode, 1080p100 practical ceiling | 720p class | 1080p60 practical, 25 Mbps |
| Goggle plug-and-play | Needs our own VRX module | Needs our own VRX module | Works in existing HDZero goggles | Needs Caddx goggles or our VRX |
| Firmware partner | OpenIPC founder contacted us 2026-10-07; an OpenIPC developer is in our Discord | OpenHD lead developer is in our Discord; their X21 hardware is under NDA | Divimath | None |
| Main risk | Jitter, Wi-Fi power draw (8812EU module 5 V 1.8 A max) | Unproven latency, newer platform | Terms, range like analog | Access and price |

**Proposed hardware shape (from the community discussion)**

Sandwich VTX: RF board carrying a finished RTL8812EU module (Alfa Drone36 or LB-Link BL-M8812EU2) with clamped U.FL connectors; compute board with SoC, power (2 to 6S), USB and a DJI-style FC connector; 1 mm twin-row header between. Do not place the 8812 chip down on our own board: no SDK for EOL parts, PA tuning and certification risk.

**Plug-and-play requirements**

- Factory pairing plus a bind button, no SSH or config files
- MSP canvas OSD working with Betaflight and INAV out of the box
- OpenVRX goggle-bay module with HDMI out
- Updates from a web configurator

**Exit criteria for Phase 1**

- Bench data from Phase 0: glass-to-glass p50 and p95, stutter count on a yaw sweep, power, weight, for Ascent Lite+, RunCam WiFiLink 2 (OpenIPC) and an RV1126B dev board
- Written terms from one firmware partner (OpenIPC or OpenHD) on maintenance and licence
- Replies from Divimath and Artosyn on price and publishability
- One stack chosen; this entry moves to Decided with the decision linked

**Community requirements (Discord, 2026-04 to 2026-10)**

- 1080p target, 720p acceptable, 4K not a goal
- Latency and jitter matter more than resolution
- Split on OpenIPC: skeptics cite Wi-Fi latency, advocates cite availability
- NDA silicon acceptable to the founder if the design is transparent; some members reject NDA datasheets
- Volunteers offered help on goggles optics, FPGA goggle pipeline and an analog-first VRX puck

## Sources

OpenDrone Discord #digital-vtx export 2026-10-07 (244 messages, read-only); [https://openipc.org/low-latency](https://openipc.org/low-latency) ; [https://blog.unmanned.tech/caddx-ascent-lite-plus-vrx-pro-review/](https://blog.unmanned.tech/caddx-ascent-lite-plus-vrx-pro-review/) ; [https://www.caddxfpv.com/products/walksnail-ascent-hd-lite-plus-kit](https://www.caddxfpv.com/products/walksnail-ascent-hd-lite-plus-kit) ; [https://blog.unmanned.tech/ruby-fpv-runcam-wifilink-review-video-jitter-latency/](https://blog.unmanned.tech/ruby-fpv-runcam-wifilink-review-video-jitter-latency/) ; [https://docs.openipc.org/use-cases/fpv/net-cards/rtl8812eu/](https://docs.openipc.org/use-cases/fpv/net-cards/rtl8812eu/) ; [https://github.com/KennyPlus/Thinker338Q](https://github.com/KennyPlus/Thinker338Q) ; [https://www.alfa.com.tw/products/drone36](https://www.alfa.com.tw/products/drone36) ; [https://jlcpcb.com/partdetail/Rockchip-RV1126/C2859017](https://jlcpcb.com/partdetail/Rockchip-RV1126/C2859017) ; [https://www.cnx-software.com/2021/05/05/sigmastar-ssc33x-camera-soc-pin-to-pin-compatible-hisilicon-hi3516-hi3518/](https://www.cnx-software.com/2021/05/05/sigmastar-ssc33x-camera-soc-pin-to-pin-compatible-hisilicon-hi3516-hi3518/)
