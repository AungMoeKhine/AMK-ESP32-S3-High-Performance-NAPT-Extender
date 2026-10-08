# AMK-ESP32-S3-High-Performance-NAPT-Extender
## ESP32-S3 Wi-Fi Extender & Long Range (LR) Gateway

A high-performance Wi-Fi Range Extender, Repeater, and Mesh-like Gateway firmware for **ESP32-S3**, optimized from Espressif's **`esp-gateway-long-range`** architecture.

---

## 1. Architectural Highlights & Engineering Optimizations

### A. Hardware-Level LwIP NAPT Routing (No CPU DNS Stall)
- **Zero Configuration**: Standard IP routing requires upstream routers to know static routes for downstream subnets. With LwIP's built-in **IPv4 NAPT** (`CONFIG_LWIP_IPV4_NAPT=y`), the ESP32 rewrites packet headers on the fly. Upstream routers see all traffic as originating directly from the ESP32's Station (STA) IP.
- **Wire-Speed DNS Forwarding**: Unlike basic repeaters that stall the CPU with a software UDP relay, DNS queries (`port 53`) are forwarded directly through native NAPT translation to upstream DNS servers. The Xtensa LX7 dual-core CPU stays 100% free to handle dashboard requests and TCP/IP routing without blocking.

### B. Smart Multi-Node Subnet Auto-Increment (Daisy-Chaining)
Consumer Wi-Fi repeaters often fail when connected to one another because their subnets collide. This firmware incorporates automated, multi-hop subnet progression:
- **Node 1** (connected to Main Router `192.168.1.x` or `0.x`): Broadcasts on **`192.168.4.1`**.
- **Node 2** (connected to Node 1 `192.168.4.x`): Automatically switches to **`192.168.5.1`**.
- **Node 3** (connected to Node 2 `192.168.5.x`): Automatically switches to **`192.168.6.1`**.
- **Node 4** (connected to Node 3 `192.168.6.x`): Automatically switches to **`192.168.7.1`**.

No IP conflicts occur across extended chains, and individual setup portals remain accessible on their respective gateway IPs.

### C. Wi-Fi Radio Rate Optimization (802.11g/n & Espressif LR)
- **Elimination of 802.11b Airtime Waste**: Downstream SoftAP runs strictly on `802.11g | 802.11n`. Legacy 802.11b management frames (1–2 Mbps) are disabled, preventing beacon airtime congestion and significantly boosting real-world wireless throughput for modern phones and PCs.
- **Espressif Long Range (LR) Mode**: When enabled on the Station interface, the ESP32-S3 uses Espressif's proprietary low-bitrate DSSS modulation (`WIFI_PROTOCOL_LR`), offering **+4 dB receiver sensitivity** and up to **1 km line-of-sight** coverage between ESP32 nodes.

### D. Zero Connection Drops & Gentle Watchdog
- **Controlled First-Connect Transition**: Connected setup clients are deauthenticated **only once** upon the initial boot transition from offline setup mode to online mode (allowing client OSs to drop captive portal badges and pull fresh internet routes). Subsequent DHCP renewals or temporary reconnects do not disrupt active clients.
- **Non-Disruptive Watchdog**: Recovers dropped connections using clean `WiFi.reconnect()` calls instead of tearing down the entire radio stack with `WiFi.disconnect()`.

---

## 2. Realistic Network Throughput Expectations

Because the ESP32-S3 operates as a single-radio, half-duplex repeater (simultaneously receiving and transmitting over one 2.4 GHz RF chain) and manages NAT in software:

* **Real-World TCP Throughput**: **10 to 14 Mbps** (peak ~15 Mbps).
* **Real-World UDP Throughput**: **15 to 18 Mbps**.
* **Ideal Use Cases**:
  * 1080p Full HD video streaming (Netflix, YouTube require ~5 Mbps).
  * Video calls, VoIP, and messaging.
  * Smart home IoT hubs, outdoor security cameras, and sensors.
  * Extending internet coverage into yards, garages, basements, workshops, and agricultural fields.

---

## 3. Project Structure

```text
esp32s3-wifi-extender/
├── esp32s3_wifi_extender.ino   <-- Optimized Arduino sketch (ESP32 core 3.x / 2.x)
├── logo.h                      <-- Embedded base64 / byte-array logo asset
└── README.md                   <-- Full project documentation & architectural analysis
