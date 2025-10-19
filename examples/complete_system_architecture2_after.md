```
┌────────────────────────────▼────────────────────────────────────┐
│                      PIPE INTERFACE LAYER                       │
│                      Location: litepcie/dll/pipe.py             │
│                                                                 │
│  ┌──────────────────┐         ┌──────────────────┐              │
│  │  TX Packetizer   │         │  RX Depacketizer │              │
│  │                  │         │                  │              │
│  │ • 64→8 bit conv  │         │ • 8→64 bit conv  │              │
│  │ • STP/SDP/END    │         │ • START detect   │              │
│  │ • K-char framing │         │ • Symbol accum   │              │
│  └──────────────────┘         └──────────────────┘              │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           Ordered Set Generation/Detection               │   │
│  │                                                          │   │
│  │  • SKP insertion (every 1180 symbols)                    │   │
│  │  • TS1/TS2 generation (link training)                    │   │
│  │  • COM symbol handling (alignment)                       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Interface: 8-bit data + K-char flag + control signals          │
└────────────────────────────┬────────────────────────────────────┘
```
