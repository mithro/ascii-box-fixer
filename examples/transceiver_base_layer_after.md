```
┌────────────────────────────▼────────────────────────────────────┐
│                    TRANSCEIVER BASE LAYER                       │
│                    Location: litepcie/phy/transceiver_base/     │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              PIPETransceiver Base Class                  │   │
│  │                                                          │   │
│  │  Common interface for all transceivers                   │   │
│  │  • TX/RX datapaths (CDC: sys_clk ↔ tx/rx_clk)            │   │
│  │  • Reset sequencing (PLL → PCS → CDR)                    │   │
│  │  • Speed control (Gen1/Gen2 switching)                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐   │
│  │ TX Datapath      │  │ RX Datapath      │  │ 8b/10b       │   │
│  │                  │  │                  │  │              │   │
│  │ • AsyncFIFO CDC  │  │ • AsyncFIFO CDC  │  │ • Encoder    │   │
│  │ • sys→tx domain  │  │ • rx→sys domain  │  │ • Decoder    │   │
│  └──────────────────┘  └──────────────────┘  └──────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
```
