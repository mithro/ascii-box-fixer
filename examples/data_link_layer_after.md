┌────────────────────────────▼────────────────────────────────────┐
│                    DATA LINK LAYER (DLL)                        │
│                    Location: litepcie/dll/                      │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │   DLL TX     │  │   DLL RX     │  │   Retry Buffer         │ │
│  │              │  │              │  │                        │ │
│  │ • LCRC gen   │  │ • LCRC check │  │ • Store TLPs           │ │
│  │ • Seq num    │  │ • ACK/NAK    │  │ • Replay on NAK        │ │
│  │ • DLLP gen   │  │ • DLLP parse │  │ • 4KB circular buffer  │ │
│  └──────────────┘  └──────────────┘  └────────────────────────┘ │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              LTSSM (Link Training State Machine)         │   │
│  │                                                          │   │
│  │  States: DETECT → POLLING → CONFIG → L0 → RECOVERY       │   │
│  │  Controls: Speed negotiation, TS1/TS2 exchange           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  DLLP Types: ACK, NAK, UpdateFC, PM_Enter_L1, etc.              │
└────────────────────────────┬────────────────────────────────────┘
