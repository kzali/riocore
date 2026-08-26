# RMII UDP Interface Plugin

The `rmii` plugin provides a direct, low-latency UDP network interface for `riocore` using an external RMII Ethernet PHY module. 

This plugin is designed to bypass the traditional SPI bottlenecks associated with modules like the W5500, leveraging the FPGA's internal logic to handle the MAC and UDP state machines directly.

![LAN8720 ETH Board](lan8720-eth-board.png)
*(Standard LAN8720 RMII Breakout Board)*

---

## Key Advantages of this Refactor

This version of the RMII plugin has been heavily optimized for resource-constrained FPGAs like the Gowin GW1NR-9 architecture.

* **BSRAM Optimization:** Previous implementations relied on asynchronous Verilog arrays for the RX/TX UDP packet queues. This caused synthesis toolchains (like Gowin and Yosys) to infer the memory using distributed logic (LUTs), quickly exhausting the logic units on smaller FPGAs. This refactor implements strict, synchronous Dual-Port Block RAM (BSRAM) instantiation, freeing up thousands of LUTs for actual machine control IO (Step/Dir, PWM, etc.).
* **Tang Nano 9K Support:** Fully tested and officially supported on the Tang Nano 9K.
* **Direct 50MHz Clocking:** Eliminates the need for complex internal FPGA PLL generation by feeding the strict 50MHz RMII reference clock directly from the LAN8720's onboard oscillator into the UDP core.
* **Hardened State Machine:** Optimized ARP resolution and UDP checksum calculation to prevent packet drops and maintain the strict real-time determinism required by LinuxCNC.

---

Hardware Setup & Wiring

This plugin expects a standard 3.3V RMII PHY, such as the **LAN8720**. 

**Important Hardware Notes:**
1. **Logic Levels:** The LAN8720 and the FPGA GPIOs must both operate at **3.3V**. Do not route these signals through 5V level shifters, as the 50MHz RMII clock will suffer severe signal degradation.
2. **Clock Wiring:** Keep the jumper wire for the 50MHz clock (`REF_CLK`) as physically short as possible (under 10cm) to prevent timing skew.
3. **Power:** Standard LAN8720 modules can draw up to 150mA under load. Ensure your 3.3V rail can supply this current alongside the FPGA.

### Recommended Tang Nano 9K Pin Mapping

| LAN8720 Pin | RMII Signal | Tang Nano 9K Pin | `config.json` Key |
| :--- | :--- | :--- | :--- |
| **RET_CLK** | 50MHz Clock | Pin 25 | `pin_netrmii_clk50m` |
| **CRS_DV** | Carrier Sense | Pin 26 | `pin_netrmii_rx_crs` |
| **RXD0** | Receive Data 0 | Pin 27 | `pin_netrmii_rxd_0` |
| **RXD1** | Receive Data 1 | Pin 28 | `pin_netrmii_rxd_1` |
| **TX_EN** | Transmit Enable | Pin 29 | `pin_netrmii_txen` |
| **TXD0** | Transmit Data 0 | Pin 30 | `pin_netrmii_txd_0` |
| **TXD1** | Transmit Data 1 | Pin 33 | `pin_netrmii_txd_1` |
| **MDC** | Management Clk | Pin 34 | `pin_netrmii_mdc` |
| **MDIO** | Management Data | Pin 40 | `pin_netrmii_mdio` |
| **nRST** | PHY Reset | Pin 41 | `pin_phyrst` |

---

## ⚙️ Configuration Example (`config.json`)

To use this plugin, define `"transport": "rmii"` in your `config.json` and map the 9 necessary RMII signals.

```json
{
  "board": "TangNano9K",
  "transport": "rmii",
  "ip": "192.168.10.14",
  "mac": "02:00:00:00:00:14",
  "gw": "192.168.10.1",
  "mask": "255.255.255.0",
  "port": 2390,

  "pin_netrmii_clk50m": "25",
  "pin_netrmii_rx_crs": "26",
  "pin_netrmii_rxd_0": "27",
  "pin_netrmii_rxd_1": "28",
  "pin_netrmii_txen": "29",
  "pin_netrmii_txd_0": "30",
  "pin_netrmii_txd_1": "33",
  "pin_netrmii_mdc": "34",
  "pin_netrmii_mdio": "40",
  "pin_phyrst": "41",

  "plugins": [
    // Add your machine IO plugins here
  ]
}