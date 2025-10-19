```
┌─────────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                          │
│                                                                  │
│  User Logic: DMA Engines, Memory Controllers, Custom Logic      │
│  Interface: TLP-level read/write requests                       │
└────────────────────────────┬────────────────────────────────────┘
                             │ 64-512 bit TLP interface
┌────────────────────────────▼────────────────────────────────────┐
│                    TRANSACTION LAYER (TLP)                       │
│                    Location: litepcie/tlp/                       │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  TLP Packetizer  │  │ TLP Depacketizer │  │ Flow Control │  │
│  │                  │  │                  │  │              │  │
│  │ • Header gen     │  │ • Header parse   │  │ • Credits    │  │
│  │ • CRC calc       │  │ • CRC check      │  │ • Throttling │  │
│  │ • Routing        │  │ • Type decode    │  │              │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
│                                                                  │
│  TLP Types: Memory Read/Write, Config, Completion, Messages     │
└────────────────────────────┬────────────────────────────────────┘
                             │ 64-bit packets (phy_layout)
┌────────────────────────────▼────────────────────────────────────┐
│                    DATA LINK LAYER (DLL)                         │
│                    Location: litepcie/dll/                       │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐│
│  │   DLL TX     │  │   DLL RX     │  │   Retry Buffer         ││
│  │              │  │              │  │                        ││
│  │ • LCRC gen   │  │ • LCRC check │  │ • Store TLPs           ││
│  │ • Seq num    │  │ • ACK/NAK    │  │ • Replay on NAK        ││
│  │ • DLLP gen   │  │ • DLLP parse │  │ • 4KB circular buffer  ││
│  └──────────────┘  └──────────────┘  └────────────────────────┘│
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              LTSSM (Link Training State Machine)         │  │
│  │                                                           │  │
│  │  States: DETECT → POLLING → CONFIG → L0 → RECOVERY      │  │
│  │  Controls: Speed negotiation, TS1/TS2 exchange          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  DLLP Types: ACK, NAK, UpdateFC, PM_Enter_L1, etc.             │
└────────────────────────────┬────────────────────────────────────┘
                             │ 64-bit packets + ordered sets
┌────────────────────────────▼────────────────────────────────────┐
│                      PIPE INTERFACE LAYER                        │
│                      Location: litepcie/dll/pipe.py              │
│                                                                  │
│  ┌──────────────────┐         ┌──────────────────┐             │
│  │  TX Packetizer   │         │  RX Depacketizer │             │
│  │                  │         │                  │             │
│  │ • 64→8 bit conv  │         │ • 8→64 bit conv  │             │
│  │ • STP/SDP/END    │         │ • START detect   │             │
│  │ • K-char framing │         │ • Symbol accum   │             │
│  └──────────────────┘         └──────────────────┘             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           Ordered Set Generation/Detection               │  │
│  │                                                           │  │
│  │  • SKP insertion (every 1180 symbols)                    │  │
│  │  • TS1/TS2 generation (link training)                    │  │
│  │  • COM symbol handling (alignment)                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Interface: 8-bit data + K-char flag + control signals          │
└────────────────────────────┬────────────────────────────────────┘
                             │ PIPE signals (8-bit + ctrl)
┌────────────────────────────▼────────────────────────────────────┐
│                    TRANSCEIVER BASE LAYER                        │
│                    Location: litepcie/phy/transceiver_base/      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              PIPETransceiver Base Class                   │  │
│  │                                                           │  │
│  │  Common interface for all transceivers                   │  │
│  │  • TX/RX datapaths (CDC: sys_clk ↔ tx/rx_clk)          │  │
│  │  • Reset sequencing (PLL → PCS → CDR)                   │  │
│  │  • Speed control (Gen1/Gen2 switching)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │ TX Datapath      │  │ RX Datapath      │  │ 8b/10b       │  │
│  │                  │  │                  │  │              │  │
│  │ • AsyncFIFO CDC  │  │ • AsyncFIFO CDC  │  │ • Encoder    │  │
│  │ • sys→tx domain  │  │ • rx→sys domain  │  │ • Decoder    │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ 10-bit encoded symbols
┌────────────────────────────▼────────────────────────────────────┐
│                    SERDES/TRANSCEIVER LAYER                      │
│            Location: litepcie/phy/xilinx/, litepcie/phy/lattice/ │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  Xilinx GTX      │  │ Xilinx GTY       │  │ ECP5 SERDES  │  │
│  │  (7-Series)      │  │ (UltraScale+)    │  │ (Lattice)    │  │
│  │                  │  │                  │  │              │  │
│  │ • GTXE2 wrapper  │  │ • GTYE4 wrapper  │  │ • DCUA wrap  │  │
│  │ • CPLL/QPLL      │  │ • QPLL0/QPLL1    │  │ • SCI config │  │
│  │ • Gen1/Gen2      │  │ • Gen1/Gen2/Gen3 │  │ • Gen1/Gen2  │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Physical Layer Functions                     │  │
│  │                                                           │  │
│  │  • Serialization/Deserialization (10 Gbps line rate)     │  │
│  │  • Clock recovery (RX CDR)                               │  │
│  │  • Equalization (DFE, CTLE)                              │  │
│  │  • Electrical idle detection                              │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ Differential serial (TX+/-, RX+/-)
                             ▼
                    Physical PCIe Link
                    (PCB traces, connector)
```
