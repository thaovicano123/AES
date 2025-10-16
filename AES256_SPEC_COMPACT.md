# 📋 AES-256 Encryption Core - FPGA Specification

**Author:** Nguyễn Đức Thạo  
**Date:** 2025-10-16  
**Version:** 1.0  
**Standard:** FIPS-197 (Advanced Encryption Standard)

---

## 🎯 Requirement of AES-256 Encryption Core

✅ Block size is 128 bits (16 bytes)  
✅ Key size is 256 bits (32 bytes)  
✅ Number of rounds is 14  
✅ Support both Encryption and Decryption  
✅ Key expansion generates 15 round keys (240 bytes total)  
✅ Implement 4 transformations: SubBytes, ShiftRows, MixColumns, AddRoundKey  
✅ Final round does NOT include MixColumns transformation  
✅ Support block mode (fixed 128-bit input/output)  
✅ Tuân thủ chuẩn FIPS-197 Appendix C.3 test vectors  

---

## � Complete I/O Summary

### Total Interface: 9 Ports

#### Control & Clock Signals (3 ports)
| Port | Type | Width | Description |
|------|------|-------|-------------|
| clk | Input | 1 | System clock |
| rst_n | Input | 1 | Active-low reset |
| start_i | Input | 1 | Start processing (pulse) |

#### Configuration (1 port)
| Port | Type | Width | Description |
|------|------|-------|-------------|
| mode_i | Input | 1 | 0=Encrypt, 1=Decrypt |

#### Data Input (2 ports)
| Port | Type | Width | Description |
|------|------|-------|-------------|
| plaintext_i | Input | 128 | Data input (plaintext for encrypt, ciphertext for decrypt) |
| key_i | Input | 256 | 256-bit key (same for both modes) |

#### Data Output (1 port)
| Port | Type | Width | Description |
|------|------|-------|-------------|
| ciphertext_o | Output | 128 | Data output (ciphertext for encrypt, plaintext for decrypt) |

#### Status Signals (2 ports)
| Port | Type | Width | Description |
|------|------|-------|-------------|
| valid_o | Output | 1 | Output data valid |
| busy_o | Output | 1 | Core is processing |

### Data Flow by Mode

```
ENCRYPTION MODE (mode_i = 0):
══════════════════════════════
Input:  plaintext_i[127:0]  = Original plaintext (e.g., "Hello World")
        key_i[255:0]        = 256-bit encryption key
Output: ciphertext_o[127:0] = Encrypted ciphertext (scrambled data)
        valid_o             = 1 (when done)
        busy_o              = 1 (during processing)

DECRYPTION MODE (mode_i = 1):
══════════════════════════════
Input:  plaintext_i[127:0]  = Encrypted ciphertext (scrambled data)
        key_i[255:0]        = Same 256-bit key used in encryption
Output: ciphertext_o[127:0] = Recovered plaintext (e.g., "Hello World")
        valid_o             = 1 (when done)
        busy_o              = 1 (during processing)
```

**⚠️ Critical Design Notes:**
1. **Port Reuse**: `plaintext_i` and `ciphertext_o` serve dual purposes
2. **Same Key**: Encryption and decryption use identical key
3. **Same Key Expansion**: Key schedule is identical for both modes
4. **Different Operations**: Encryption uses forward transforms, decryption uses inverse transforms
5. **Different Order**: Encryption applies round keys 0→14, decryption applies 14→0

---

## �📊 Block Diagrams

### Overall System Diagram

```
════════════════════════════════════════════════════════════════════════════════
                        AES-256 ENCRYPTION/DECRYPTION CORE
════════════════════════════════════════════════════════════════════════════════

                         INPUTS (6 signals)
                         ══════════════════

                    plaintext_i[127:0]
                    ─────────────────
                    • 128-bit data input
                    • Mode 0: Original plaintext
                    • Mode 1: Encrypted ciphertext
                           │
                           │
                           ▼
           ┌───────────────────────────────────────────┐
           │                                           │
key_i      │                                           │      ciphertext_o
[255:0] ──→│                                           │──→   [127:0]
           │                                           │      ─────────────
256-bit    │                                           │      • 128-bit output
Key        │           AES-256 CORE                    │      • Mode 0: Ciphertext
           │                                           │      • Mode 1: Plaintext
           │                                           │
           │   ┌─────────────────────────────────┐    │
start_i ──→│   │  • Key Expansion (15 keys)      │    │──→   valid_o
           │   │  • 14 Round Transformations     │    │      ────────
Start      │   │  • SubBytes / InvSubBytes       │    │      Output Valid
Trigger    │   │  • ShiftRows / InvShiftRows     │    │      (High when ready)
(Pulse)    │   │  • MixColumns / InvMixColumns   │    │
           │   │  • AddRoundKey                  │    │
           │   └─────────────────────────────────┘    │
mode_i ───→│                                           │──→   busy_o
           │   Mode Select:                            │      ────────
0=Encrypt  │   • 0: Encryption path                   │      Processing Status
1=Decrypt  │   • 1: Decryption path                   │      (High during work)
           │                                           │
           │                                           │
clk ──────→│   Clock & Control:                       │
           │   • Positive edge triggered              │
System     │   • Synchronous operations               │
Clock      │                                           │
           │                                           │
rst_n ─────→│   Reset:                                 │
           │   • Active LOW                           │
Active-Low │   • Asynchronous reset                   │
Reset      │                                           │
           │                                           │
           └───────────────────────────────────────────┘

                         OUTPUTS (3 signals)
                         ═══════════════════


═══════════════════════════════════════════════════════════════════════════════
                              SIGNAL DETAILS
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ INPUTS (6 ports)                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. plaintext_i[127:0]                                                       │
│    ├─ Type: Input, 128 bits                                                │
│    ├─ Purpose: Data input (dual function)                                  │
│    ├─ Encryption mode: Original plaintext data                             │
│    └─ Decryption mode: Encrypted ciphertext data                           │
│                                                                             │
│ 2. key_i[255:0]                                                             │
│    ├─ Type: Input, 256 bits                                                │
│    ├─ Purpose: Master encryption/decryption key                            │
│    ├─ Same key used for both encryption and decryption                     │
│    └─ Expands to 15 round keys (240 bytes total)                           │
│                                                                             │
│ 3. start_i                                                                  │
│    ├─ Type: Input, 1 bit (control signal)                                  │
│    ├─ Purpose: Start trigger                                               │
│    ├─ Active: Positive edge                                                │
│    └─ Triggers encryption or decryption process                            │
│                                                                             │
│ 4. mode_i                                                                   │
│    ├─ Type: Input, 1 bit (configuration)                                   │
│    ├─ Purpose: Operation mode selection                                    │
│    ├─ Value 0: Encryption mode                                             │
│    └─ Value 1: Decryption mode                                             │
│                                                                             │
│ 5. clk                                                                      │
│    ├─ Type: Input, 1 bit (clock)                                           │
│    ├─ Purpose: System clock signal                                         │
│    ├─ Active: Positive edge                                                │
│    └─ Target frequency: ≥ 100 MHz                                          │
│                                                                             │
│ 6. rst_n                                                                    │
│    ├─ Type: Input, 1 bit (reset)                                           │
│    ├─ Purpose: System reset                                                │
│    ├─ Active: LOW (0)                                                      │
│    └─ Type: Asynchronous reset                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ OUTPUTS (3 ports)                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. ciphertext_o[127:0]                                                      │
│    ├─ Type: Output, 128 bits                                               │
│    ├─ Purpose: Data output (dual function)                                 │
│    ├─ Encryption mode: Encrypted ciphertext                                │
│    └─ Decryption mode: Recovered plaintext                                 │
│                                                                             │
│ 2. valid_o                                                                  │
│    ├─ Type: Output, 1 bit (status signal)                                  │
│    ├─ Purpose: Output data valid indicator                                 │
│    ├─ High (1): Output data is ready and valid                             │
│    └─ Low (0): Output data not ready                                       │
│                                                                             │
│ 3. busy_o                                                                   │
│    ├─ Type: Output, 1 bit (status signal)                                  │
│    ├─ Purpose: Processing status indicator                                 │
│    ├─ High (1): Core is busy processing                                    │
│    └─ Low (0): Core is idle, ready for new input                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
                          OPERATION MODES DETAIL
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ MODE 0: ENCRYPTION (mode_i = 0)                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT SIGNALS:                                                             │
│  ──────────────                                                             │
│  plaintext_i[127:0]  = Original plaintext (e.g., "Hello World......")      │
│  key_i[255:0]        = 256-bit encryption key                              │
│  start_i             = 1 (pulse to start)                                  │
│  mode_i              = 0 (encryption mode)                                 │
│  clk                 = System clock                                        │
│  rst_n               = 1 (not in reset)                                    │
│                                                                             │
│  INTERNAL PROCESS:                                                          │
│  ─────────────────                                                          │
│  1. Key Expansion: key_i[255:0] → 15 round keys                           │
│  2. Round 0: AddRoundKey(plaintext, RoundKey[0])                           │
│  3. Rounds 1-13: SubBytes → ShiftRows → MixColumns → AddRoundKey          │
│  4. Round 14: SubBytes → ShiftRows → AddRoundKey (NO MixColumns)          │
│                                                                             │
│  OUTPUT SIGNALS:                                                            │
│  ───────────────                                                            │
│  ciphertext_o[127:0] = Encrypted ciphertext (scrambled data)               │
│  valid_o             = 1 (when encryption complete)                        │
│  busy_o              = 1 (during processing), 0 (when done)                │
│                                                                             │
│  TIMING:                                                                    │
│  ───────                                                                    │
│  Latency: ~20-50 clock cycles (depends on implementation)                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ MODE 1: DECRYPTION (mode_i = 1)                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT SIGNALS:                                                             │
│  ──────────────                                                             │
│  plaintext_i[127:0]  = Encrypted ciphertext (scrambled data)               │
│                        ⚠️  NOTE: Uses same port, but different data        │
│  key_i[255:0]        = Same 256-bit key used in encryption                 │
│  start_i             = 1 (pulse to start)                                  │
│  mode_i              = 1 (decryption mode)                                 │
│  clk                 = System clock                                        │
│  rst_n               = 1 (not in reset)                                    │
│                                                                             │
│  INTERNAL PROCESS:                                                          │
│  ─────────────────                                                          │
│  1. Key Expansion: key_i[255:0] → 15 round keys (same as encryption)      │
│  2. Round 14: AddRoundKey(ciphertext, RoundKey[14])                        │
│  3. Rounds 13-1: InvShiftRows → InvSubBytes → AddRoundKey → InvMixColumns │
│  4. Round 0: InvShiftRows → InvSubBytes → AddRoundKey (NO InvMixColumns)  │
│                                                                             │
│  OUTPUT SIGNALS:                                                            │
│  ───────────────                                                            │
│  ciphertext_o[127:0] = Recovered plaintext (e.g., "Hello World......")     │
│                        ⚠️  NOTE: Uses same port, but different data        │
│  valid_o             = 1 (when decryption complete)                        │
│  busy_o              = 1 (during processing), 0 (when done)                │
│                                                                             │
│  TIMING:                                                                    │
│  ───────                                                                    │
│  Latency: ~20-50 clock cycles (depends on implementation)                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
                       PORT REUSE ARCHITECTURE
═══════════════════════════════════════════════════════════════════════════════

                    WHY PORT REUSE?
                    ───────────────
    • Hardware optimization: Reduce pin count
    • FPGA efficiency: Share same data buses
    • Interface simplicity: Single unified interface
    • Area saving: Minimize routing resources

    ENCRYPTION FLOW:
    ════════════════
    
    plaintext_i[127:0] ────┐
    (Real Plaintext)       │
                          ▼
                    ┌──────────┐
    key_i[255:0] ──→│ AES-256  │
    mode_i = 0   ──→│   CORE   │
    start_i = 1  ──→│          │
                    └──────────┘
                          │
                          ▼
    ciphertext_o[127:0] ──┘
    (Real Ciphertext)


    DECRYPTION FLOW:
    ════════════════
    
    plaintext_i[127:0] ────┐
    (Ciphertext Input)     │  ⚠️ SAME PORT, DIFFERENT DATA
                          ▼
                    ┌──────────┐
    key_i[255:0] ──→│ AES-256  │
    mode_i = 1   ──→│   CORE   │  (Inverse operations)
    start_i = 1  ──→│          │
                    └──────────┘
                          │
                          ▼
    ciphertext_o[127:0] ──┘
    (Plaintext Output)         ⚠️ SAME PORT, DIFFERENT DATA


═══════════════════════════════════════════════════════════════════════════════
                            COMPLETE I/O SUMMARY
═══════════════════════════════════════════════════════════════════════════════

TOTAL: 9 PORTS (6 Inputs + 3 Outputs)

✅ DATA PATHS:
   • Input data bus: 128 bits (plaintext_i)
   • Output data bus: 128 bits (ciphertext_o)
   • Key input: 256 bits (key_i)

✅ CONTROL SIGNALS:
   • Start trigger: 1 bit (start_i)
   • Mode select: 1 bit (mode_i)

✅ TIMING SIGNALS:
   • Clock: 1 bit (clk)
   • Reset: 1 bit (rst_n)

✅ STATUS SIGNALS:
   • Valid indicator: 1 bit (valid_o)
   • Busy indicator: 1 bit (busy_o)

TOTAL BIT WIDTH: 128 + 256 + 128 + 5 control/status = 517 bits
```

---

## 🔧 Parameters

| No | Name | Value | Description |
|----|------|-------|-------------|
| 1 | P_CLK_FREQ | 100_000_000 | The frequency of clock signal = 100 MHz (default) |
| 2 | P_DATA_WIDTH | 128 | Data block width = 128 bits (plaintext/ciphertext) |
| 3 | P_KEY_WIDTH | 256 | Master key width = 256 bits (AES-256) |
| 4 | P_NUM_ROUNDS | 14 | Number of encryption/decryption rounds |
| 5 | P_NUM_ROUND_KEYS | 15 | Total round keys = rounds + 1 |
| 6 | P_ROUND_KEY_WIDTH | 128 | Each round key width = 128 bits |
| 7 | P_KEY_WORDS | 60 | Total key expansion words (w[0] to w[59]) |
| 8 | P_STATE_SIZE | 16 | State matrix size = 16 bytes (4×4 matrix) |
| 9 | P_SBOX_SIZE | 256 | S-box lookup table size = 256 entries |
| 10 | P_RCON_SIZE | 14 | Round constant table size for key expansion |

---

## 🔌 Interface Specification

### Interface Table

| Port Name | I/O | Bitwidth | Clock domain | Active type | Active level | Description |
|-----------|-----|----------|--------------|-------------|--------------|-------------|
| clk | I | 1 | - | Edge | Positive Edge | System clock signal for synchronous operations |
| rst_n | I | 1 | - | Level | Low | Asynchronous reset signal (active low) |
| plaintext_i | I | 128 | clk | Level | High | 128-bit data input. **Encryption mode**: original plaintext data. **Decryption mode**: encrypted ciphertext data |
| key_i | I | 256 | clk | Level | High | 256-bit master key for encryption/decryption. Same key used for both modes |
| start_i | I | 1 | clk | Edge | Positive Edge | Start signal. The positive edge of this signal will trigger the encryption or decryption process |
| mode_i | I | 1 | clk | Level | High | Operation mode selection. **0 = Encryption mode**, **1 = Decryption mode** |
| ciphertext_o | O | 128 | clk | Level | High | 128-bit data output. **Encryption mode**: encrypted ciphertext. **Decryption mode**: recovered plaintext |
| valid_o | O | 1 | clk | Level | High | Output valid signal. High indicates the output data is ready and valid |
| busy_o | O | 1 | clk | Level | High | Busy status signal. High indicates the core is processing. Low indicates ready for new input |

---

## 🔑 Key Expansion Module

```
                    key_i[255:0]
                    (32 bytes)
                         │
                         ▼
                ┌────────────────┐
                │                │
                │  Key Expansion │
                │     Module     │
                │                │
                │  8 words       │
                │    ↓           │
                │  60 words      │
                │                │
                └────────┬───────┘
                         │
                         ▼
              round_keys[239:0]
              (15 keys × 16 bytes)
              
              RoundKey[0]:  w[0..3]   (16 bytes)
              RoundKey[1]:  w[4..7]   (16 bytes)
              RoundKey[2]:  w[8..11]  (16 bytes)
              ...
              RoundKey[14]: w[56..59] (16 bytes)
```

**Algorithm Details:**
```
┌─────────────────────────────────────────────┐
│ i % 8 == 0:                                 │
│   temp = RotWord(w[i-1])                    │
│   temp = SubWord(temp)                      │
│   temp = temp ⊕ Rcon[i/8]                   │
│   w[i] = w[i-8] ⊕ temp                      │
│                                             │
│ i % 8 == 4: (AES-256 SPECIAL)               │
│   temp = SubWord(w[i-1])  // NO RotWord    │
│   w[i] = w[i-8] ⊕ temp                      │
│                                             │
│ Otherwise:                                  │
│   w[i] = w[i-1] ⊕ w[i-8]                    │
└─────────────────────────────────────────────┘
```

---

### Module 2: Encryption Core

```
        plaintext_i[127:0]          round_keys[239:0]
        (16 bytes)                  (from Key Expansion)
              │                              │
              ▼                              ▼
      ┌───────────────────────────────────────┐
      │                                       │
      │         ENCRYPTION CORE               │
      │                                       │
      │  ┌─────────────────────────────┐     │
      │  │  Round 0                    │     │
      │  │  AddRoundKey(RK[0])         │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      │  ┌─────────────────────────────┐     │
      │  │  Rounds 1-13 (×13)          │     │
      │  │  ┌─────────────────┐        │     │
      │  │  │ SubBytes        │        │     │
      │  │  │ ShiftRows       │        │     │
      │  │  │ MixColumns      │        │     │
      │  │  │ AddRoundKey(RK) │        │     │
      │  │  └─────────────────┘        │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      │  ┌─────────────────────────────┐     │
      │  │  Round 14 (Final)           │     │
      │  │  SubBytes                   │     │
      │  │  ShiftRows                  │     │
      │  │  AddRoundKey(RK[14])        │     │
      │  │  ❌ NO MixColumns            │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      └─────────────┬─────────────────────────┘
                    ▼
            ciphertext_o[127:0]
            (16 bytes)
```

**Transformations Used:**
- ✅ SubBytes (S-box)
- ✅ ShiftRows (Left shift)
- ✅ MixColumns (GF(2^8) matrix)
- ✅ AddRoundKey (XOR)

---

### Module 3: Decryption Core

```
        ciphertext_i[127:0]         round_keys[239:0]
        (16 bytes)                  (from Key Expansion)
              │                              │
              ▼                              ▼
      ┌───────────────────────────────────────┐
      │                                       │
      │         DECRYPTION CORE               │
      │                                       │
      │  ┌─────────────────────────────┐     │
      │  │  Round 14 (Initial)         │     │
      │  │  AddRoundKey(RK[14])        │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      │  ┌─────────────────────────────┐     │
      │  │  Rounds 13-1 (×13)          │     │
      │  │  ┌─────────────────┐        │     │
      │  │  │ InvShiftRows    │        │     │
      │  │  │ InvSubBytes     │        │     │
      │  │  │ AddRoundKey(RK) │        │     │
      │  │  │ InvMixColumns   │        │     │
      │  │  └─────────────────┘        │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      │  ┌─────────────────────────────┐     │
      │  │  Round 0 (Final)            │     │
      │  │  InvShiftRows               │     │
      │  │  InvSubBytes                │     │
      │  │  AddRoundKey(RK[0])         │     │
      │  │  ❌ NO InvMixColumns         │     │
      │  └──────────┬──────────────────┘     │
      │             ▼                         │
      └─────────────┬─────────────────────────┘
                    ▼
            plaintext_o[127:0]
            (16 bytes - recovered)
```

**Inverse Transformations Used:**
- ✅ InvSubBytes (Inverse S-box)
- ✅ InvShiftRows (Right shift)
- ✅ InvMixColumns (Inverse GF(2^8) matrix)
- ✅ AddRoundKey (XOR - same as encryption)

---

### Complete System Integration

```
                     ┌──────────────────────────────────────┐
                     │                                      │
    key_i[255:0] ────┤   Key Expansion Module               │
                     │   (Generate 15 Round Keys)           │
                     │   Same for both Encrypt & Decrypt    │
                     └─────────────┬────────────────────────┘
                                   │
                          round_keys[239:0]
                          (15 keys × 128 bits)
                                   │
                     ┌─────────────┴────────────────┐
                     │                              │
                     ▼                              ▼
         ┌───────────────────┐        ┌───────────────────┐
         │  Encryption Core  │        │  Decryption Core  │
         │   (mode_i = 0)    │        │   (mode_i = 1)    │
         │                   │        │                   │
plaintext│  14 Rounds        │        │  14 Rounds        │ciphertext
  [127:0]│  RK[0]→RK[14]     │        │  RK[14]→RK[0]     │[127:0]
    ────▶│  (Forward order)  │        │  (Reverse order)  │◀────
         │                   │        │                   │
         │  SubBytes         │        │  InvSubBytes      │
         │  ShiftRows        │        │  InvShiftRows     │
         │  MixColumns       │        │  InvMixColumns    │
         │  AddRoundKey      │        │  AddRoundKey      │
         │                   │        │                   │
         └─────────┬─────────┘        └─────────┬─────────┘
                   │                            │
                   ▼                            ▼
            ciphertext_o[127:0]          ciphertext_o[127:0]
            (Encrypted output)           (Recovered plaintext)


    🔄 PORT REUSE MAPPING:
    ════════════════════════════════════════════════════
    
    ENCRYPTION (mode_i = 0):
    ┌─────────────────┐         ┌─────────────────────┐
    │ plaintext_i     │  ─────→ │ ciphertext_o        │
    │ (Real plaintext)│   AES   │ (Real ciphertext)   │
    └─────────────────┘         └─────────────────────┘
    
    DECRYPTION (mode_i = 1):
    ┌─────────────────┐         ┌─────────────────────┐
    │ plaintext_i     │  ─────→ │ ciphertext_o        │
    │ (Ciphertext in) │ AES⁻¹   │ (Plaintext out)     │
    └─────────────────┘         └─────────────────────┘
```

---

## 🔧 Parameters

| No | Name | Bitwidth | Description |
|----|------|----------|-------------|
| 1 | P_CLK_FREQ | 32 | The frequency of clock signal (Hz) |
| 2 | P_DATA_WIDTH | 32 | Data width = 128 bits |
| 3 | P_KEY_WIDTH | 32 | Key width = 256 bits |
| 4 | P_NUM_ROUNDS | 32 | Number of rounds = 14 |

---

## 🔌 Interface Specification

### Main Interface

| Port Name | I/O | Bitwidth | Clock domain | Active type | Active level | Description |
|-----------|-----|----------|--------------|-------------|--------------|-------------|
| clk | I | 1 | - | Edge | Positive Edge | System clock signal |
| rst_n | I | 1 | - | Level | Low | Asynchronous reset (active low) |
| plaintext_i | I | 128 | clk | Level | High | 128-bit input data. **Encryption mode**: plaintext input. **Decryption mode**: ciphertext input |
| key_i | I | 256 | clk | Level | High | 256-bit encryption/decryption key (same key for both modes) |
| start_i | I | 1 | clk | Edge | Positive Edge | Start signal. Positive edge triggers encryption or decryption process |
| mode_i | I | 1 | clk | Level | High | Operation mode: **0 = Encryption**, **1 = Decryption** |
| ciphertext_o | O | 128 | clk | Level | High | 128-bit output data. **Encryption mode**: ciphertext output. **Decryption mode**: recovered plaintext output |
| valid_o | O | 1 | clk | Level | High | Output valid signal. High when output data is ready and valid |
| busy_o | O | 1 | clk | Level | High | Busy signal. High when core is processing. Low when ready for new input |

### 📌 I/O Port Mapping Summary

| Mode | Input Port | Input Type | Output Port | Output Type |
|------|------------|------------|-------------|-------------|
| **Encryption** (mode_i=0) | plaintext_i[127:0] | Original Plaintext | ciphertext_o[127:0] | Encrypted Ciphertext |
| **Decryption** (mode_i=1) | plaintext_i[127:0] | Encrypted Ciphertext | ciphertext_o[127:0] | Recovered Plaintext |

**⚠️ Important Notes:**
- Both modes use the **same physical ports** (plaintext_i and ciphertext_o)
- Port naming reflects encryption mode convention
- Hardware implementation: Use `mode_i` to control datapath multiplexing
- The `key_i[255:0]` is the **same key** for both encryption and decryption
- Key expansion process is **identical** for both modes
- Only the **round key usage order** differs (forward vs reverse)

---

## 📝 Detailed Function Description

### 1. Encryption Mode (mode_i = 0)

**Input:**
- `plaintext_i[127:0]`: 128-bit plaintext (original data to encrypt)
- `key_i[255:0]`: 256-bit encryption key

**Process:**
```
Round 0:    AddRoundKey(State, RoundKey[0])

Rounds 1-13:
    - SubBytes(State)
    - ShiftRows(State)
    - MixColumns(State)
    - AddRoundKey(State, RoundKey[i])

Round 14 (Final):
    - SubBytes(State)
    - ShiftRows(State)
    - AddRoundKey(State, RoundKey[14])
    ❌ NO MixColumns
```

**Output:**
- `ciphertext_o[127:0]`: 128-bit ciphertext (encrypted data)
- `valid_o`: High when encryption complete
- `busy_o`: Low when ready for next operation

### 2. Decryption Mode (mode_i = 1)

**Input:**
- `plaintext_i[127:0]`: 128-bit ciphertext (encrypted data to decrypt) ⚠️ **Note: Uses same port as plaintext**
- `key_i[255:0]`: 256-bit decryption key (same key as encryption)

**Process:**
```
Round 14:   AddRoundKey(State, RoundKey[14])

Rounds 13-1:
    - InvShiftRows(State)
    - InvSubBytes(State)
    - AddRoundKey(State, RoundKey[i])
    - InvMixColumns(State)

Round 0 (Final):
    - InvShiftRows(State)
    - InvSubBytes(State)
    - AddRoundKey(State, RoundKey[0])
    ❌ NO InvMixColumns
```

**Output:**
- `ciphertext_o[127:0]`: 128-bit plaintext (recovered original data) ⚠️ **Note: Uses same port as ciphertext**
- `valid_o`: High when decryption complete
- `busy_o`: Low when ready for next operation

---

## 🔑 Key Expansion

**Input:** 256-bit master key (8 words)  
**Output:** 15 round keys (60 words = 240 bytes)

**Algorithm:**
```
w[0..7] = key_i[255:0]  // Initial 8 words

For i = 8 to 59:
    temp = w[i-1]
    
    If (i % 8 == 0):
        temp = SubWord(RotWord(temp)) ⊕ Rcon[i/8]
    Else if (i % 8 == 4):  // AES-256 SPECIAL CASE
        temp = SubWord(temp)
    
    w[i] = w[i-8] ⊕ temp
```

**Round Keys:**
- Round Key 0 = w[0..3] (16 bytes)
- Round Key 1 = w[4..7] (16 bytes)
- ...
- Round Key 14 = w[56..59] (16 bytes)

---

## 🧮 Core Transformations

### 1. SubBytes Transformation

| Parameter | Value |
|-----------|-------|
| Input | State[127:0] (4×4 matrix) |
| Output | State[127:0] (4×4 matrix) |
| Operation | S-box lookup for each byte |
| S-box size | 256 bytes (ROM/LUT) |
| Latency | 1 clock cycle (parallel) or 16 cycles (sequential) |

**Function:**
```
For each byte in State:
    State[i] = SBOX[State[i]]
```

### 2. ShiftRows Transformation

| Parameter | Value |
|-----------|-------|
| Input | State[127:0] (4×4 matrix) |
| Output | State[127:0] (4×4 matrix) |
| Operation | Cyclic left shift |
| Latency | 0 cycles (wiring only) |

**Function:**
```
Row 0: No shift       [a b c d] → [a b c d]
Row 1: Shift left 1   [a b c d] → [b c d a]
Row 2: Shift left 2   [a b c d] → [c d a b]
Row 3: Shift left 3   [a b c d] → [d a b c]
```

### 3. MixColumns Transformation

| Parameter | Value |
|-----------|-------|
| Input | State[127:0] (4×4 matrix) |
| Output | State[127:0] (4×4 matrix) |
| Operation | Matrix multiplication in GF(2^8) |
| Polynomial | 0x11b (x^8+x^4+x^3+x+1) |
| Latency | 2-3 clock cycles |

**Matrix:**
```
[02 03 01 01]
[01 02 03 01]
[01 01 02 03]
[03 01 01 02]
```

**Function:**
```
For each column c in State:
    out[0] = (02•in[0]) ⊕ (03•in[1]) ⊕ (01•in[2]) ⊕ (01•in[3])
    out[1] = (01•in[0]) ⊕ (02•in[1]) ⊕ (03•in[2]) ⊕ (01•in[3])
    out[2] = (01•in[0]) ⊕ (01•in[1]) ⊕ (02•in[2]) ⊕ (03•in[3])
    out[3] = (03•in[0]) ⊕ (01•in[1]) ⊕ (01•in[2]) ⊕ (02•in[3])
```

### 4. AddRoundKey Transformation

| Parameter | Value |
|-----------|-------|
| Input | State[127:0] + RoundKey[127:0] |
| Output | State[127:0] |
| Operation | XOR operation |
| Latency | 0 cycles (combinational) or 1 cycle |

**Function:**
```
State[127:0] = State[127:0] ⊕ RoundKey[127:0]
```

---

## 📊 Lookup Tables

### S-box Table (256 bytes)

| Usage | Size | Implementation | Access Time |
|-------|------|----------------|-------------|
| SubBytes | 256 bytes | ROM/LUT | 1 cycle |

### Inverse S-box Table (256 bytes)

| Usage | Size | Implementation | Access Time |
|-------|------|----------------|-------------|
| InvSubBytes | 256 bytes | ROM/LUT | 1 cycle |

### Rcon Table (14 values)

| Usage | Values | Implementation |
|-------|--------|----------------|
| Key Expansion | [01,02,04,08,10,20,40,80,1b,36,6c,d8,ab,4d] | ROM/Constants |

---

## ⏱️ Timing Specification

### Encryption Timing

| Parameter | Value | Note |
|-----------|-------|------|
| **Latency** | ~20-50 clock cycles | Depends on implementation |
| **Throughput** | 1 block per latency | For single block |
| **Key Expansion Time** | ~10-20 cycles | One-time setup |
| **Clock Frequency** | ≥ 100 MHz | Target for FPGA |

### Control Signals Timing

```
Clock cycle:     0   1   2   3   4   ... N-1  N   N+1
                 │   │   │   │   │       │   │   │
clk         ─────┐   ┐   ┐   ┐   ┐      ┐   ┐   ┐
            ─────┘   ┘   ┘   ┘   ┘      ┘   ┘   ┘

start_i     ─────┐
            ─────┘───────────────────────────────

busy_o      ─────────┐
            ─────────┘───────────────────────────┘

valid_o     ─────────────────────────────────┐
            ─────────────────────────────────┘───
```

---

## 🧪 Test Vectors (FIPS-197 Appendix C.3)

### Encryption Test

**Input:**
```
Plaintext:  00112233445566778899aabbccddeeff
Key:        000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
```

**Expected Output:**
```
Ciphertext: 8ea2b7ca516745bfeafc49904b496089
```

### Round-trip Test

**Encrypt then Decrypt:**
```
Original:   00112233445566778899aabbccddeeff
↓ Encrypt
Ciphertext: 8ea2b7ca516745bfeafc49904b496089
↓ Decrypt
Recovered:  00112233445566778899aabbccddeeff  ✅ Match!
```

---

## 💾 Resource Estimation (FPGA)

| Resource | Estimated Usage | Note |
|----------|----------------|------|
| **Logic Elements** | 5,000 - 10,000 LEs | Depends on optimization |
| **Memory (RAM)** | ~10 KB | S-box, Round keys |
| **ROM** | ~512 bytes | S-box tables |
| **DSP Blocks** | 0 | Not required |
| **Clock Frequency** | ≥ 100 MHz | Target |
| **Power Consumption** | < 500 mW | Estimated |

---

## 📐 State Machine

### Main States

```
IDLE           → Wait for start_i
KEY_EXPAND     → Expand key to 15 round keys
ENCRYPT_INIT   → Initialize encryption
ENCRYPT_ROUND  → Execute rounds 1-13
ENCRYPT_FINAL  → Execute round 14 (no MixColumns)
DECRYPT_INIT   → Initialize decryption
DECRYPT_ROUND  → Execute rounds 13-1
DECRYPT_FINAL  → Execute round 0 (no InvMixColumns)
DONE           → Output valid, assert valid_o
```

### State Transitions

```
       ┌────┐
  ┌───→│IDLE│←──────────┐
  │    └─┬──┘           │
  │      │ start_i=1    │
  │      ▼              │
  │  ┌────────────┐     │
  │  │KEY_EXPAND  │     │
  │  └─────┬──────┘     │
  │        │            │
  │   mode_i=0│1        │
  │    ┌─────┴─────┐    │
  │    ▼           ▼    │
  │ ┌──────┐   ┌──────┐ │
  │ │ENCR  │   │DECR  │ │
  │ │_INIT │   │_INIT │ │
  │ └─┬────┘   └───┬──┘ │
  │   ▼            ▼    │
  │ ┌──────┐   ┌──────┐ │
  │ │ENCR  │   │DECR  │ │
  │ │_ROUND│   │_ROUND│ │
  │ └─┬────┘   └───┬──┘ │
  │   ▼            ▼    │
  │ ┌──────┐   ┌──────┐ │
  │ │ENCR  │   │DECR  │ │
  │ │_FINAL│   │_FINAL│ │
  │ └─┬────┘   └───┬──┘ │
  │   └──────┬──────┘    │
  │          ▼           │
  │      ┌──────┐        │
  └──────│ DONE │────────┘
         └──────┘
```

---

## 🔒 Security Features

### Cryptographic Strength

| Feature | Value |
|---------|-------|
| **Key Space** | 2^256 combinations |
| **Brute Force Resistance** | Computationally infeasible |
| **Differential Cryptanalysis** | Resistant |
| **Linear Cryptanalysis** | Resistant |
| **Known Attacks** | None practical |

### Compliance

✅ FIPS-197 compliant  
✅ NIST approved  
✅ Used by US Government (Top Secret level)  
✅ Industry standard worldwide  

---

## 📚 Design Recommendations

### For High Performance

```
✅ Pipeline architecture (14 stages)
✅ Parallel S-box lookups (16 instances)
✅ Unrolled rounds
✅ Pre-computed round keys
```

**Trade-off:** Higher area, maximum throughput

### For Low Area

```
✅ Sequential processing (1 round per cycle)
✅ Shared S-box (1 instance)
✅ On-the-fly key expansion
✅ State machine control
```

**Trade-off:** Lower throughput, minimum area

### Balanced Design

```
✅ Partially pipelined (4-5 stages)
✅ Shared resources with multiplexing
✅ Pre-computed round keys (stored in RAM)
✅ Moderate parallelism
```

**Trade-off:** Good balance of area and speed

---

## 📖 References

1. **FIPS-197** - Advanced Encryption Standard (AES)
   - National Institute of Standards and Technology
   - November 26, 2001

2. **Implementation Files:**
   - `aes256.py` - Python reference implementation
   - `DEEP_STEP_VERIFICATION.md` - Detailed verification report
   - `SPECIFICATION.md` - Full specification document

3. **Test Resources:**
   - FIPS-197 Appendix C.3 (AES-256 test vectors)
   - NIST Cryptographic Algorithm Validation Program

---

## 📊 FINAL I/O VERIFICATION CHECKLIST

### ✅ All Inputs (6 ports)

| # | Port Name | Width | Purpose | Notes |
|---|-----------|-------|---------|-------|
| 1 | **clk** | 1 bit | System clock | Positive edge |
| 2 | **rst_n** | 1 bit | Reset signal | Active low |
| 3 | **start_i** | 1 bit | Start trigger | Pulse to begin |
| 4 | **mode_i** | 1 bit | Operation mode | 0=Encrypt, 1=Decrypt |
| 5 | **plaintext_i** | 128 bits | Data input | Dual purpose (see below) |
| 6 | **key_i** | 256 bits | Encryption key | Same for both modes |

### ✅ All Outputs (3 ports)

| # | Port Name | Width | Purpose | Notes |
|---|-----------|-------|---------|-------|
| 1 | **ciphertext_o** | 128 bits | Data output | Dual purpose (see below) |
| 2 | **valid_o** | 1 bit | Output ready | High when done |
| 3 | **busy_o** | 1 bit | Processing status | High during operation |

### ✅ Dual-Purpose Ports Clarification

#### Port: plaintext_i[127:0]
- **Encryption mode (mode_i=0)**: Accepts original plaintext data
- **Decryption mode (mode_i=1)**: Accepts encrypted ciphertext data
- **Reason**: Hardware optimization - reuse same input bus

#### Port: ciphertext_o[127:0]
- **Encryption mode (mode_i=0)**: Outputs encrypted ciphertext
- **Decryption mode (mode_i=1)**: Outputs recovered plaintext
- **Reason**: Hardware optimization - reuse same output bus

### ✅ Data Flow Verification

```
┌─────────────────────────────────────────────────────────────┐
│                    ENCRYPTION PATH                           │
│  plaintext_i[127:0] ──→ [AES Core] ──→ ciphertext_o[127:0] │
│   (Plain data)         mode_i=0         (Encrypted data)    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    DECRYPTION PATH                           │
│  plaintext_i[127:0] ──→ [AES Core] ──→ ciphertext_o[127:0] │
│  (Encrypted data)      mode_i=1         (Plain data)        │
└─────────────────────────────────────────────────────────────┘
```

### ✅ Complete I/O Coverage

| Category | Encryption | Decryption | Status |
|----------|-----------|------------|--------|
| **Input Data** | plaintext_i | plaintext_i (reused) | ✅ Covered |
| **Input Key** | key_i | key_i (same key) | ✅ Covered |
| **Output Data** | ciphertext_o | ciphertext_o (reused) | ✅ Covered |
| **Control** | start_i, mode_i | start_i, mode_i | ✅ Covered |
| **Clock/Reset** | clk, rst_n | clk, rst_n | ✅ Covered |
| **Status** | valid_o, busy_o | valid_o, busy_o | ✅ Covered |

**Total I/O Count:** 9 ports (6 inputs + 3 outputs)

---

**Document Version:** 1.0  
**Last Updated:** 2025-10-16  
**Status:** ✅ READY FOR FPGA IMPLEMENTATION  
**I/O Completeness:** ✅ ALL INPUTS/OUTPUTS VERIFIED FOR BOTH ENCRYPTION & DECRYPTION

---

*This specification provides a compact, clear reference for FPGA implementation of AES-256 encryption core following FIPS-197 standard.*
