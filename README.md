

## 📋 Requirement of AES-256


- **Clock frequency**: 100MHz

- **Power**: < 500mW

- **RAM**: 10kB


### 🔒 Chức năng Mã hóa (Encryption)

- **Input**: plaintext 128 bit + key 256 bit

- **Output**: ciphertext 128 bit

- **Mục đích**: Dùng để mã hóa dữ liệu•	RAM : 10kB



### 🔓 Chức năng Giải mã (Decryption)

- **Input**: ciphertext 128 bit + key 256 bit

- **Output**: plaintext 128 bit

- **Mục đích**: Khôi phục dữ liệu gốc

### 🔑 Key Expansion

- **Input**: key 256 bit gốc
- **Output**: 15 round key (128 bit) = 60 words

- **Mục đích**: Dùng để tạo khóa (round key) cho từng vòng từ key gốc


### 🔄 State Transformation (4 phép biến đổi)

#### 1. SubBytes

- **Chức năng**: Sử dụng bảng S-box thay thế byte

- **Input**: State 4×4 bytes

- **Output**: State 4×4 bytes

#### 2. Shift Rows

- **Chức năng**: Dịch trái đối với mã hóa và dịch phải đối với giải mã

- **Chi tiết**:

  - Row 0: không dịch
  - Row 1: dịch 1 byte
  - Row 2: dịch 2 byte
  - Row 3: dịch 3 byte

#### 3. MixColumns

- **Chức năng**: Dùng để trộn dữ liệu các cột

- **Input**:Ma trận State (4 × 4)

- **Output**: Ma trận State (4 × 4)

- **Phương pháp**: Nhân với ma trận cố định trong GF(2^8)


#### 4. AddRoundKey

- **Chức năng**: XOR state với round key tùy theo từng vòng	

- **Input**: State + round key

- **Output**: State| **RAM Usage** | 10 kB |



### 🔁 Cấu trúc Round

#### 📤 Encryption (14 vòng)

- **Round 0**: AddRoundKey

- **Round 1→13**: ShiftRows → SubByte → MixColumns → AddRoundKey

- **Round 14**: ShiftRows → SubByte → AddRoundKey


#### 📥 Decryption (14 vòng)

- **Round 14**: AddRoundKey

- **Round 13→1**: InvShiftRows → InvSubByte → AddRoundKey → InvMixColumns

- **Round 0**: InvShiftRows → InvSubByte → AddRoundKey



### 📦 Các chức năng bổ sung


#### Padding

- **Mục đích**: Thêm padding cho data để đủ block size để thực hiện mã hóa


#### Unpadding

- **Mục đích**: Xóa padding sau khi mã hóa để lấy lại data gốc



#### Nhân ma trận cố định### 

- **Mục đích**: Sử dụng trong bước MixColumns và InvMixColumns

---



## 📊 Bảng tra cứu AES-256

### 🔢 Bảng S-box (AES-256)

Bảng S-box dùng cho phép biến đổi SubBytes trong quá trình mã hóa:

```
     | 0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
-----+------------------------------------------------
0x00 | 63 7C 77 7B F2 6B 6F C5 30 01 67 2B FE D7 AB 76
0x10 | CA 82 C9 7D FA 59 47 F0 AD D4 A2 AF 9C A4 72 C0
0x20 | B7 FD 93 26 36 3F F7 CC 34 A5 E5 F1 71 D8 31 15
0x30 | 04 C7 23 C3 18 96 05 9A 07 12 80 E2 EB 27 B2 75
0x40 | 09 83 2C 1A 1B 6E 5A A0 52 3B D6 B3 29 E3 2F 84
0x50 | 53 D1 00 ED 20 FC B1 5B 6A CB BE 39 4A 4C 58 CF
0x60 | D0 EF AA FB 43 4D 33 85 45 F9 02 7F 50 3C 9F A8
0x70 | 51 A3 40 8F 92 9D 38 F5 BC B6 DA 21 10 FF F3 D2
0x80 | CD 0C 13 EC 5F 97 44 17 C4 A7 7E 3D 64 5D 19 73
0x90 | 60 81 4F DC 22 2A 90 88 46 EE B8 14 DE 5E 0B DB
0xA0 | E0 32 3A 0A 49 06 24 5C C2 D3 AC 62 91 95 E4 79
0xB0 | E7 C8 37 6D 8D D5 4E A9 6C 56 F4 EA 65 7A AE 08
0xC0 | BA 78 25 2E 1C A6 B4 C6 E8 DD 74 1F 4B BD 8B 8A
0xD0 | 70 3E B5 66 48 03 F6 0E 61 35 57 B9 86 C1 1D 9E
0xE0 | E1 F8 98 11 69 D9 8E 94 9B 1E 87 E9 CE 55 28 DF
0xF0 | 8C A1 89 0D BF E6 42 68 41 99 2D 0F B0 54 BB 16
```

**Cách sử dụng**: Để thay thế byte `xy`, tra cứu giá trị tại hàng `0xx0` và cột `0xy`.
- Ví dụ: S-box[0x53] = 0xED (hàng 0x50, cột 0x03)

### 🔄 Bảng S-box ngược (Inverse S-box AES-256)

Bảng S-box ngược dùng cho phép biến đổi InvSubBytes trong quá trình giải mã:

```
     | 0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
-----+------------------------------------------------
0x00 | 52 09 6A D5 30 36 A5 38 BF 40 A3 9E 81 F3 D7 FB
0x10 | 7C E3 39 82 9B 2F FF 87 34 8E 43 44 C4 DE E9 CB
0x20 | 54 7B 94 32 A6 C2 23 3D EE 4C 95 0B 42 FA C3 4E
0x30 | 08 2E A1 66 28 D9 24 B2 76 5B A2 49 6D 8B D1 25
0x40 | 72 F8 F6 64 86 68 98 16 D4 A4 5C CC 5D 65 B6 92
0x50 | 6C 70 48 50 FD ED B9 DA 5E 15 46 57 A7 8D 9D 84
0x60 | 90 D8 AB 00 8C BC D3 0A F7 E4 58 05 B8 B3 45 06
0x70 | D0 2C 1E 8F CA 3F 0F 02 C1 AF BD 03 01 13 8A 6B
0x80 | 3A 91 11 41 4F 67 DC EA 97 F2 CF CE F0 B4 E6 73
0x90 | 96 AC 74 22 E7 AD 35 85 E2 F9 37 E8 1C 75 DF 6E
0xA0 | 47 F1 1A 71 1D 29 C5 89 6F B7 62 0E AA 18 BE 1B
0xB0 | FC 56 3E 4B C6 D2 79 20 9A DB C0 FE 78 CD 5A F4
0xC0 | 1F DD A8 33 88 07 C7 31 B1 12 10 59 27 80 EC 5F
0xD0 | 60 51 7F A9 19 B5 4A 0D 2D E5 7A 9F 93 C9 9C EF
0xE0 | A0 E0 3B 4D AE 2A F5 B0 C8 EB BB 3C 83 53 99 61
0xF0 | 17 2B 04 7E BA 77 D6 26 E1 69 14 63 55 21 0C 7D
```

**Cách sử dụng**: Để thay thế ngược byte `xy`, tra cứu giá trị tại hàng `0xx0` và cột `0xy`.
- Ví dụ: InvS-box[0xED] = 0x53 (hàng 0xE0, cột 0x0D)

### 🔑 Bảng Rcon (Round Constant - AES-256)

Bảng Rcon dùng cho quá trình Key Expansion (mở rộng khóa):

```
Round | Rcon[i] (hex) | Rcon[i] (binary)
------+---------------+------------------
  1   | 01 00 00 00   | 00000001 00000000 00000000 00000000
  2   | 02 00 00 00   | 00000010 00000000 00000000 00000000
  3   | 04 00 00 00   | 00000100 00000000 00000000 00000000
  4   | 08 00 00 00   | 00001000 00000000 00000000 00000000
  5   | 10 00 00 00   | 00010000 00000000 00000000 00000000
  6   | 20 00 00 00   | 00100000 00000000 00000000 00000000
  7   | 40 00 00 00   | 01000000 00000000 00000000 00000000
  8   | 80 00 00 00   | 10000000 00000000 00000000 00000000
  9   | 1B 00 00 00   | 00011011 00000000 00000000 00000000
 10   | 36 00 00 00   | 00110110 00000000 00000000 00000000
 11   | 6C 00 00 00   | 01101100 00000000 00000000 00000000
 12   | D8 00 00 00   | 11011000 00000000 00000000 00000000
 13   | AB 00 00 00   | 10101011 00000000 00000000 00000000
 14   | 4D 00 00 00   | 01001101 00000000 00000000 00000000
```

**Lưu ý**:
- Rcon chỉ sử dụng byte đầu tiên (3 byte còn lại luôn là 00)
- AES-256 sử dụng 14 giá trị Rcon (từ round 1 đến 14)
- Công thức: Rcon[i] = x^(i-1) trong GF(2^8), với x = 0x02
- Khi x^i >= 0x100, thực hiện XOR với 0x11B (polynomial bất khả quy)

---


## 🏗️ Sơ đồ khối AES-256 Core

### Block Diagram - AES-256 Core

```
                plaintext_i[127:0] ← Plaintext input (128 bits)
                         │
                         ▼
              ┌──────────────────────┐
              │                      │
key_i[255:0] ─┤                      │──→ ciphertext_o[127:0]
              │                      │
   start_i ───┤                      │──→ valid_o
              │     AES-256 Core     │
   mode_i ────┤                      │──→ busy_o
              │                      │
      clk ────┤                      │
              │                      │
    rst_n ────┤                      │
              │                      │
              └──────────────────────┘
```

### Mô tả các tín hiệu:

#### Input Signals (Tín hiệu đầu vào):
- **plaintext_i[127:0]**: Dữ liệu plaintext đầu vào (128 bits)
- **key_i[255:0]**: Khóa mã hóa (256 bits)
- **start_i**: Tín hiệu bắt đầu quá trình mã hóa/giải mã
- **mode_i**: Chế độ hoạt động (0 = Encryption, 1 = Decryption)
- **clk**: Clock hệ thống (100 MHz)
- **rst_n**: Reset tích cực mức thấp (active-low reset)

#### Output Signals (Tín hiệu đầu ra):
- **ciphertext_o[127:0]**: Dữ liệu ciphertext đầu ra (128 bits)
- **valid_o**: Tín hiệu báo dữ liệu đầu ra hợp lệ
- **busy_o**: Tín hiệu báo core đang xử lý

---

## 🔧 Parameters (Thông số thiết kế)

| No | Name | Value | Description |
|----|------|-------|-------------|
| 1 | P_CLK_FREQ | 100_000_000 | Tần số clock = 100 MHz (mặc định) |
| 2 | P_DATA_WIDTH | 128 | Độ rộng data block = 128 bits (plaintext/ciphertext) |
| 3 | P_KEY_WIDTH | 256 | Độ rộng master key = 256 bits (AES-256) |
| 4 | P_NUM_ROUNDS | 14 | Số vòng mã hóa/giải mã |
| 5 | P_NUM_ROUND_KEYS | 15 | Tổng số round keys = rounds + 1 |
| 6 | P_ROUND_KEY_WIDTH | 128 | Độ rộng mỗi round key = 128 bits |
| 7 | P_KEY_WORDS | 60 | Tổng số words trong key expansion (w[0] đến w[59]) |
| 8 | P_STATE_SIZE | 16 | Kích thước state matrix = 16 bytes (ma trận 4×4) |
| 9 | P_SBOX_SIZE | 256 | Kích thước bảng S-box = 256 entries |
| 10 | P_RCON_SIZE | 14 | Kích thước bảng round constant cho key expansion |

---

## 🔌 Interface Specification (Chi tiết giao diện)

### Bảng mô tả các Port

| Port Name | I/O | Bitwidth | Clock Domain | Active Type | Active Level | Description |
|-----------|-----|----------|--------------|-------------|--------------|-------------|
| **clk** | Input | 1 | - | Edge | Positive Edge | Tín hiệu clock hệ thống cho các hoạt động đồng bộ |
| **rst_n** | Input | 1 | - | Level | Low | Tín hiệu reset bất đồng bộ (active low) |
| **plaintext_i** | Input | 128 | clk | Level | High | 128-bit dữ liệu đầu vào. **Chế độ mã hóa**: plaintext gốc. **Chế độ giải mã**: ciphertext đã mã hóa |
| **key_i** | Input | 256 | clk | Level | High | 256-bit master key cho mã hóa/giải mã. Cùng một key cho cả hai chế độ |
| **start_i** | Input | 1 | clk | Edge | Positive Edge | Tín hiệu bắt đầu. Cạnh dương của tín hiệu này sẽ kích hoạt quá trình mã hóa hoặc giải mã |
| **mode_i** | Input | 1 | clk | Level | High | Chọn chế độ hoạt động. **0 = Encryption**, **1 = Decryption** |
| **ciphertext_o** | Output | 128 | clk | Level | High | 128-bit dữ liệu đầu ra. **Chế độ mã hóa**: ciphertext đã mã hóa. **Chế độ giải mã**: plaintext đã khôi phục |
| **valid_o** | Output | 1 | clk | Level | High | Tín hiệu dữ liệu hợp lệ. Mức cao cho biết dữ liệu đầu ra đã sẵn sàng và hợp lệ |
| **busy_o** | Output | 1 | clk | Level | High | Tín hiệu trạng thái bận. Mức cao cho biết core đang xử lý. Mức thấp cho biết sẵn sàng nhận input mới |

### 📊 Tổng kết Interface

**Tổng số Port: 9 ports (6 Inputs + 3 Outputs)**

#### Input Ports (6 ports):
- **Control & Clock**: clk, rst_n, start_i (3 ports)
- **Configuration**: mode_i (1 port)
- **Data Input**: plaintext_i[127:0], key_i[255:0] (2 ports)

#### Output Ports (3 ports):
- **Data Output**: ciphertext_o[127:0] (1 port)
- **Status Signals**: valid_o, busy_o (2 ports)

### 🔄 Ánh xạ Port theo Mode

| Mode | Input Port | Input Type | Output Port | Output Type |
|------|------------|------------|-------------|-------------|
| **Encryption** (mode_i=0) | plaintext_i[127:0] | Plaintext gốc | ciphertext_o[127:0] | Ciphertext đã mã hóa |
| **Decryption** (mode_i=1) | plaintext_i[127:0] | Ciphertext đã mã hóa | ciphertext_o[127:0] | Plaintext đã khôi phục |

**⚠️ Lưu ý quan trọng:**
- Cả hai mode đều sử dụng **cùng các port vật lý** (plaintext_i và ciphertext_o)
- Tên port phản ánh quy ước của chế độ mã hóa
- Trong hardware: Sử dụng `mode_i` để điều khiển multiplexing datapath
- `key_i[255:0]` là **cùng một key** cho cả mã hóa và giải mã
- Quá trình key expansion **giống hệt nhau** cho cả hai mode
- Chỉ **thứ tự sử dụng round key** khác nhau (thuận vs nghịch)

---

## ⏱️ Timing Diagram - Overall System (Biểu đồ thời gian hệ thống)

### 📊 AES-256 Overall System Timing Diagram - All 9 Ports (Continuous)

Biểu đồ dưới đây mô tả hoạt động thời gian của toàn bộ hệ thống AES-256 với **9 ports** (6 inputs + 3 outputs) cho cả hai chế độ **Encryption** và **Decryption**:

![AES-256 Overall System Timing Diagram](https://github.com/thaovicano123/AES/blob/main/bieudo.jpg?raw=true)

*Hình: Timing diagram hoàn chỉnh hiển thị tất cả 9 ports trong một biểu đồ liên tục*

### 🔍 Phân tích Timing Diagram

#### **Encryption Mode (Bên trái - mode_i = 0)**

| Cycle | clk | rst_n | start_i | mode_i | plaintext_i | key_i | busy_o | ciphertext_o | valid_o | Mô tả |
|-------|-----|-------|---------|--------|-------------|-------|--------|--------------|---------|-------|
| 0 | ⬆️ | **0** | 0 | 0 | Invalid | Invalid | 0 | Invalid | 0 | **Reset active** |
| 1 | ⬆️ | **1** | 0 | 0 | Invalid | Invalid | 0 | Invalid | 0 | **Release reset** |
| 2 | ⬆️ | 1 | **1** | 0 | **Valid** | **Valid** | 0 | Invalid | 0 | **Start + Input valid** |
| 3 | ⬆️ | 1 | **0** | 0 | Valid | Valid | **1** | Invalid | 0 | **Processing begins** |
| 4-20 | ⬆️ | 1 | 0 | 0 | Valid | Valid | **1** | Invalid | 0 | **Processing (18 cycles)** |
| 21 | ⬆️ | 1 | 0 | 0 | Valid | Valid | **0** | **Valid** | **1** | **Output ready** |
| 22 | ⬆️ | 1 | 0 | 0 | **Invalid** | **Invalid** | 0 | Valid | 1 | **Input drops** |
| 23-24 | ⬆️ | 1 | 0 | 0 | Invalid | Invalid | 0 | Valid | 1 | **Output remains** |

#### **Decryption Mode (Bên phải - mode_i = 1)**

| Cycle | clk | rst_n | start_i | mode_i | plaintext_i | key_i | busy_o | ciphertext_o | valid_o | Mô tả |
|-------|-----|-------|---------|--------|-------------|-------|--------|--------------|---------|-------|
| 0 | ⬆️ | **0** | 0 | 1 | Invalid | Invalid | 0 | Invalid | 0 | **Reset active** |
| 1 | ⬆️ | **1** | 0 | 1 | Invalid | Invalid | 0 | Invalid | 0 | **Release reset** |
| 2 | ⬆️ | 1 | **1** | 1 | **Valid (Cipher)** | **Valid** | 0 | Invalid | 0 | **Start + Cipher input** |
| 3 | ⬆️ | 1 | **0** | 1 | Valid | Valid | **1** | Invalid | 0 | **Processing begins** |
| 4-20 | ⬆️ | 1 | 0 | 1 | Valid | Valid | **1** | Invalid | 0 | **Processing (18 cycles)** |
| 21 | ⬆️ | 1 | 0 | 1 | Valid | Valid | **0** | **Valid (Plain)** | **1** | **Plaintext recovered** |
| 22 | ⬆️ | 1 | 0 | 1 | **Invalid** | **Invalid** | 0 | Valid | 1 | **Input drops** |
| 23-24 | ⬆️ | 1 | 0 | 1 | Invalid | Invalid | 0 | Valid | 1 | **Output remains** |

### 📌 Các đặc điểm chính

#### 1. **Latency (Độ trễ xử lý)**
```
Start to Output Ready: 18 clock cycles
├─ Cycle 2: start_i = 1 (trigger)
├─ Cycle 3-20: Processing (14 rounds)
└─ Cycle 21: valid_o = 1 (output ready)
```

#### 2. **Input Signals Behavior**
- **plaintext_i & key_i**: 
  - Valid từ **cycle 2 → cycle 21** (20 cycles)
  - Drop xuống **Invalid (0)** tại **cycle 22**
  - Tiết kiệm năng lượng sau khi processing hoàn tất

#### 3. **Control Signals**
- **rst_n**: Active LOW tại cycle 0, sau đó giữ HIGH
- **start_i**: 1-cycle pulse (HIGH tại cycle 2-3)
- **mode_i**: 
  - `0` = Encryption (bên trái)
  - `1` = Decryption (bên phải)

#### 4. **Output Signals**
- **busy_o**: HIGH từ cycle 3-20 (18 cycles processing)
- **valid_o**: HIGH từ cycle 21-23 (3 cycles)
- **ciphertext_o**: Valid từ cycle 21-24

#### 5. **Port Reuse (Tái sử dụng ports)**

| Mode | plaintext_i carries | ciphertext_o outputs |
|------|---------------------|----------------------|
| **Encryption** | Original Plaintext | Encrypted Ciphertext |
| **Decryption** | Encrypted Ciphertext | Recovered Plaintext |

### 🎯 Design Highlights

1. **✅ Deterministic Latency**: Luôn 18 cycles cho cả encryption và decryption
2. **✅ Symmetric Operation**: Cả hai modes có timing giống hệt nhau
3. **✅ Same Key Usage**: Sử dụng cùng một key cho cả mã hóa và giải mã
4. **✅ Power Efficient**: Input signals drop về 0 khi không cần thiết
5. **✅ Pipeline Ready**: Có thể bắt đầu operation mới ngay sau valid_o
6. **✅ Simple Interface**: Chỉ 9 ports cho toàn bộ hệ thống

### 💡 Timing Specifications

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Clock Frequency** | 100 MHz | Tần số hoạt động |
| **Clock Period** | 10 ns | Chu kỳ clock |
| **Latency** | 18 cycles = 180 ns | Từ start đến output valid |
| **Throughput** | ~5.56 M blocks/sec | 128 bits / 180 ns |
| **Processing Time** | 18 cycles | 14 rounds + overhead |
| **Input Setup Time** | 1 cycle | Cycle 2 |
| **Output Hold Time** | 3 cycles | Cycle 21-23 |

### 📝 Cách đọc Timing Diagram

1. **Trục ngang (X-axis)**: Clock cycles (0 → 50)
2. **Trục dọc (Y-axis)**: 9 signals (từ trên xuống)
   - `clk`: System clock
   - `rst_n`: Reset signal
   - `start_i`: Start trigger
   - `mode_i`: Mode selection
   - `plaintext_i[127:0]`: 128-bit input data
   - `key_i[255:0]`: 256-bit key
   - `busy_o`: Processing status
   - `ciphertext_o[127:0]`: 128-bit output data
   - `valid_o`: Output valid flag

3. **Màu sắc**:
   - 🔵 **Xanh dương**: Clock signal
   - 🔴 **Đỏ**: Reset signal (active LOW)
   - 🟠 **Cam**: Start trigger
   - 🟤 **Nâu**: Mode selection
   - 🟢 **Xanh lá**: Data signals
   - 🟣 **Tím**: Status signals

4. **Vùng highlights**:
   - 🟡 **Vàng nhạt**: Processing region (Encryption)
   - 🔵 **Xanh nhạt**: Processing region (Decryption)



