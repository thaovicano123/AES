# BÁO CÁO ĐỒ ÁN: THIẾT KẾ SoC RISC-V TÍCH HỢP AES-256 TRÊN FPGA

**Đề tài:** Thiết kế hệ thống trên chip (SoC) sử dụng lõi vi xử lý RISC-V PicoRV32 tích hợp bộ tăng tốc phần cứng mã hóa AES-256 trên nền tảng FPGA Sipeed Tang Mega 60K

**Ngày:** 13/12/2025

---

## MỤC LỤC

1. [Tổng quan đề tài](#1-tổng-quan-đề-tài)
2. [Vai trò các thành phần](#2-vai-trò-các-thành-phần)
3. [Quy trình thực hiện (Flow)](#3-quy-trình-thực-hiện-flow)
4. [Chi tiết xây dựng từng thành phần](#4-chi-tiết-xây-dựng-từng-thành-phần)
5. [Kết hợp các thành phần](#5-kết-hợp-các-thành-phần)
6. [Hướng dẫn thu thập số liệu từ Gowin IDE](#6-hướng-dẫn-thu-thập-số-liệu-từ-gowin-ide)
7. [Số liệu thực tế của dự án](#7-số-liệu-thực-tế-của-dự-án)
8. [Câu hỏi thường gặp từ hội đồng](#8-câu-hỏi-thường-gặp-từ-hội-đồng)

---

## 1. TỔNG QUAN ĐỀ TÀI

### 1.1. Mục tiêu

Đề tài hướng đến việc thiết kế một **System-on-Chip (SoC)** hoàn chỉnh trên nền tảng FPGA, bao gồm:

- **CPU lõi mềm RISC-V (PicoRV32)**: Xử lý logic điều khiển và giao tiếp người dùng
- **Bộ tăng tốc phần cứng AES-256**: Thực hiện mã hóa/giải mã tốc độ cao
- **Các ngoại vi cơ bản**: UART, GPIO, SPI Flash, JTAG

### 1.2. Ý nghĩa thực tiễn

| Khía cạnh | Mô tả |
|-----------|-------|
| **Học thuật** | Nắm vững quy trình thiết kế SoC từ RTL đến nạp board |
| **Kỹ năng** | Verilog HDL, C embedded, FPGA toolchain, Bus protocol |
| **Ứng dụng** | Bảo mật IoT, HSM (Hardware Security Module), Crypto Engine |

### 1.3. Sơ đồ khối tổng thể

```
┌─────────────────────────────────────────────────────────────────┐
│                    TANG MEGA 60K FPGA                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     SoC Design                           │   │
│  │  ┌─────────────┐    AHB Bus    ┌─────────────────────┐   │   │
│  │  │  PicoRV32   │◄─────────────►│  AES-256 Accelerator│   │   │
│  │  │  RISC-V CPU │               │  (Hardware Engine)  │   │   │
│  │  └──────┬──────┘               └─────────────────────┘   │   │
│  │         │ Wishbone Bus                                   │   │
│  │  ┌──────┴──────┬──────────────┬─────────────┐            │   │
│  │  │             │              │             │            │   │
│  │  ▼             ▼              ▼             ▼            │   │
│  │ ┌────┐    ┌─────────┐   ┌─────────┐   ┌─────────┐        │   │
│  │ │RAM │    │  UART   │   │  GPIO   │   │SPI Flash│        │   │
│  │ │32KB│    │115200bps│   │  LEDs   │   │  Boot   │        │   │
│  │ └────┘    └────┬────┘   └────┬────┘   └─────────┘        │   │
│  └────────────────┼─────────────┼───────────────────────────┘   │
│                   │             │                               │
└───────────────────┼─────────────┼───────────────────────────────┘
                    │             │
                    ▼             ▼
                ┌───────┐    ┌────────┐
                │  PC   │    │  LEDs  │
                │Terminal    │(Status)│
                └───────┘    └────────┘
```

---

## 2. VAI TRÒ CÁC THÀNH PHẦN

### 2.1. Board Tang Mega 60K

**Vai trò:** Nền tảng phần cứng chứa toàn bộ thiết kế

| Thông số | Giá trị |
|----------|---------|
| **Chip FPGA** | Gowin GW5AT-LV60PG484AC1/I0 |
| **Logic Elements** | 59,904 LUTs |
| **Registers** | 60,780 FFs |
| **Block RAM** | 118 blocks (BSRAM) |
| **DSP Blocks** | 118 units |
| **Clock Input** | 50 MHz oscillator |
| **Package** | PG484 (484 pins) |

**Tại sao chọn Tang Mega 60K:**
- Tài nguyên dồi dào cho SoC phức tạp
- Gowin IDE miễn phí (Education version)
- Giá thành hợp lý cho học tập
- Hỗ trợ RISC-V IP Core sẵn có

### 2.2. PicoRV32 (RISC-V CPU)

**Vai trò:** Bộ não điều khiển hệ thống

| Đặc điểm | Mô tả |
|----------|-------|
| **Kiến trúc** | RISC-V RV32IMC |
| **Pipeline** | 2-stage (Fetch + Execute) |
| **Bus Interface** | Native Memory Interface + AHB Master |
| **Interrupt** | 32 IRQ lines |
| **Footprint** | ~2000-3000 LUTs (nhỏ gọn) |

**Chức năng trong hệ thống:**
1. Chạy firmware C điều khiển menu UART
2. Nhận lệnh từ người dùng (Encrypt/Decrypt)
3. Gửi dữ liệu xuống AES Core qua AHB bus
4. Đọc kết quả và hiển thị ra terminal

### 2.3. AES-256 Hardware Accelerator

**Vai trò:** Cơ bắp xử lý mã hóa tốc độ cao

| Thông số | Giá trị |
|----------|---------|
| **Thuật toán** | AES-256 ECB Mode |
| **Key Size** | 256-bit |
| **Block Size** | 128-bit |
| **Số Round** | 14 rounds |
| **Latency** | ~15-20 clock cycles/block |

**Tại sao cần Hardware Accelerator:**

| Phương pháp | Số cycles/block | So sánh |
|-------------|-----------------|---------|
| Software (C trên PicoRV32) | ~3000-5000 cycles | Chậm |
| **Hardware Accelerator** | **~20 cycles** | **Nhanh hơn 150-250x** |

---

## 3. QUY TRÌNH THỰC HIỆN (FLOW)

### 3.1. Tổng quan Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    DESIGN FLOW                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────┐    ┌──────────┐    ┌───────────┐    ┌───────────┐  │
│  │ Step 1  │───►│  Step 2  │───►│  Step 3   │───►│  Step 4   │  │
│  │RTL Code │    │Simulation│    │ Synthesis │    │Place&Route│  │
│  │(Verilog)│    │(Testbench│    │(Gowin IDE)│    │(Gowin IDE)│  │
│  └─────────┘    └──────────┘    └───────────┘    └─────┬─────┘  │
│                                                        │        │
│                                                        ▼        │
│  ┌─────────┐    ┌──────────┐    ┌───────────┐    ┌───────────┐  │
│  │ Step 8  │◄───│  Step 7  │◄───│  Step 6   │◄───│  Step 5   │  │
│  │ Demo &  │    │  Debug   │    │  Program  │    │ Firmware  │  │
│  │  Test   │    │ & Verify │    │  FPGA     │    │   (C)     │  │
│  └─────────┘    └──────────┘    └───────────┘    └───────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2. Chi tiết từng bước

#### **Step 1: RTL Design (Viết code Verilog)**

| File | Chức năng |
|------|-----------|
| `picorv32_aes256_top.v` | Top module, kết nối tất cả thành phần |
| `aes256_core.v` | Lõi AES-256 chính (FSM + datapath) |
| `aes256_ahb_wrapper.v` | Wrapper giao tiếp AHB bus |
| `aes256_key_expansion_comb.v` | Sinh 15 round keys |
| `aes256_subbytes.v` | S-Box transformation |
| `aes256_shiftrows.v` | ShiftRows transformation |
| `aes256_mixcolumns.v` | MixColumns (GF(2^8) multiply) |

#### **Step 2: Simulation (Mô phỏng)**

- Tool: ModelSim, Icarus Verilog, hoặc Gowin Simulator
- Testbench: `tb_aes256_comprehensive.v`
- Verify với NIST test vectors

#### **Step 3: Synthesis (Tổng hợp)**

- Biến Verilog → Netlist (cổng logic)
- Output: `impl/gwsynthesis/picorv32_aes256.vg`

#### **Step 4: Place & Route (Đặt và đi dây)**

- Đặt các cổng logic lên FPGA fabric
- Đi dây kết nối giữa các thành phần
- Output: `impl/pnr/picorv32_aes256.fs` (bitstream)

#### **Step 5: Firmware Development**

- Viết code C (`firmware/main.c`)
- Biên dịch: `riscv32-unknown-elf-gcc`
- Tạo file hex nạp vào RAM

#### **Step 6: Program FPGA**

- Nạp bitstream (`.fs`) bằng Gowin Programmer
- Có thể nạp vào Flash để boot tự động

#### **Step 7 & 8: Debug và Test**

- Kết nối UART terminal (115200 baud)
- Chạy test vectors
- Verify kết quả với chuẩn NIST

---

## 4. CHI TIẾT XÂY DỰNG TỪNG THÀNH PHẦN

### 4.1. AES-256 Core Architecture

#### 4.1.1. Cấu trúc FSM (Finite State Machine)

```
         ┌──────────────────────────────────────────────────────┐
         │                  AES-256 FSM                         │
         │                                                      │
         │   ┌─────────┐     start=1     ┌───────────┐          │
         │   │ S_IDLE  │────────────────►│ S_KEY_ADD │          │
         │   │ (Wait)  │                 │ (Round 0) │          │
         │   └────▲────┘                 └─────┬─────┘          │
         │        │                            │                │
         │        │ done                       ▼                │
         │   ┌────┴────┐     round<13    ┌───────────┐          │
         │   │ S_DONE  │◄────────────────│ S_ROUND   │◄─┐       │
         │   │(Output) │                 │(Round 1-13)│  │       │
         │   └─────────┘                 └─────┬─────┘  │       │
         │        ▲                            │        │       │
         │        │ round=14                   │ round++│       │
         │   ┌────┴────┐     round=13    ┌─────┴─────┐  │       │
         │   │ S_FINAL │◄────────────────│  (loop)   │──┘       │
         │   │(Round 14)                 └───────────┘          │
         │   └─────────┘                                        │
         │                                                      │
         └──────────────────────────────────────────────────────┘
```

#### 4.1.2. Datapath cho mỗi Round

**Encryption (mode = 0):**
```
state_in ──► SubBytes ──► ShiftRows ──► MixColumns ──► AddRoundKey ──► state_out
                                           │
                                           └── (Skip ở Round 14)
```

**Decryption (mode = 1):**
```
state_in ──► InvShiftRows ──► InvSubBytes ──► AddRoundKey ──► InvMixColumns ──► state_out
                                                                   │
                                                                   └── (Skip ở Round 14)
```

#### 4.1.3. Key Expansion

- **Input:** 256-bit master key
- **Output:** 15 round keys (mỗi key 128-bit) = 1920 bits
- **Phương pháp:** Combinational logic (sinh tức thì, không cần clock)
- **Trade-off:** Tốn ~7000 LUTs nhưng latency = 0

### 4.2. AHB Wrapper - Memory Map

| Địa chỉ | Thanh ghi | Chức năng |
|---------|-----------|-----------|
| 0x80000000 | CTRL | bit[0]: Start, bit[1]: Mode (0=Enc, 1=Dec) |
| 0x80000004 | STATUS | bit[0]: Done, bit[1]: Busy |
| 0x80000010-0x8000002C | KEY[0-7] | 256-bit Key (8 x 32-bit) |
| 0x80000030-0x8000003C | DATA_IN[0-3] | 128-bit Input (4 x 32-bit) |
| 0x80000040-0x8000004C | DATA_OUT[0-3] | 128-bit Output (4 x 32-bit) |

### 4.3. Top Module Integration

```verilog
// Kết nối trong picorv32_aes256_top.v:

// 1. Clock và Reset
wire sys_clk = clk_50m;
wire sys_rst_n = rst_n & resetn_auto;

// 2. AES-256 trên AHB Bus
aes256_ahb_wrapper u_aes256 (
    .hclk(sys_clk),
    .hresetn(sys_rst_n),
    .haddr(haddr),
    .htrans(htrans),
    // ... các tín hiệu AHB khác
);

// 3. PicoRV32 CPU
Gowin_PicoRV32_Top u_picorv32 (
    .clk_in(sys_clk),
    .resetn_in(sys_rst_n),
    .wbuart_tx(uart_tx),
    .wbuart_rx(uart_rx),
    // ... bus connections đến AES
);
```

---

## 5. KẾT HỢP CÁC THÀNH PHẦN

### 5.1. Luồng hoạt động khi Encrypt

```
┌──────────────────────────────────────────────────────────────────┐
│                    ENCRYPTION FLOW                                │
│                                                                   │
│  User ──► UART ──► PicoRV32 ──► AHB Bus ──► AES Core ──► Result  │
│                                                                   │
│  Step 1: User nhập Key (64 hex chars = 256 bit)                  │
│          Terminal: > 00010203...1e1f                              │
│                                                                   │
│  Step 2: User nhập Plaintext (32 hex chars = 128 bit)            │
│          Terminal: > 00112233...eeff                              │
│                                                                   │
│  Step 3: CPU ghi Key vào AES_KEY[0-7]                            │
│          for(i=0; i<8; i++) AES_KEY(i) = key[i];                 │
│                                                                   │
│  Step 4: CPU ghi Data vào AES_DATA_IN[0-3]                       │
│          for(i=0; i<4; i++) AES_DATA_IN(i) = data[i];            │
│                                                                   │
│  Step 5: CPU trigger Start (mode=0 for encrypt)                  │
│          AES_CTRL = 0x01;                                         │
│                                                                   │
│  Step 6: AES Core thực hiện 14 rounds (~20 cycles)               │
│          SubBytes → ShiftRows → MixColumns → AddRoundKey         │
│                                                                   │
│  Step 7: CPU poll Status, chờ Done=1                             │
│          while(!(AES_STATUS & 0x01));                            │
│                                                                   │
│  Step 8: CPU đọc kết quả từ AES_DATA_OUT[0-3]                    │
│          for(i=0; i<4; i++) result[i] = AES_DATA_OUT(i);         │
│                                                                   │
│  Step 9: Hiển thị Ciphertext ra terminal                         │
│          Terminal: CIPHERTEXT: 8EA2B7CA516745BF...               │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### 5.2. Firmware Code Example

```c
// Hàm mã hóa trong firmware/main.c
int aes_encrypt_decrypt(int mode) {
    int i;
    
    // Load key (256-bit = 8 words)
    for (i = 0; i < 8; i++) {
        AES_KEY(i) = g_key[i];
    }
    
    // Load data (128-bit = 4 words)
    for (i = 0; i < 4; i++) {
        AES_DATA_IN(i) = g_data[i];
    }
    
    // Start operation: bit[0]=start, bit[1]=mode
    AES_CTRL = (mode << 1) | 0x01;
    
    // Wait for completion
    for (i = 0; i < 100000; i++) {
        if (AES_STATUS & 0x01) {  // Done flag
            // Read result
            for (int j = 0; j < 4; j++) {
                g_result[j] = AES_DATA_OUT(j);
            }
            return 1;  // Success
        }
    }
    return 0;  // Timeout
}
```

---

## 6. HƯỚNG DẪN THU THẬP SỐ LIỆU TỪ GOWIN IDE

### 6.1. Mở các Report Files

Sau khi chạy **Synthesis** và **Place & Route** trong Gowin IDE, các file report được tạo trong thư mục `impl/`:

| Loại Report | Đường dẫn | Nội dung |
|-------------|-----------|----------|
| Synthesis Resource | `impl/gwsynthesis/picorv32_aes256_syn_rsc.xml` | Chi tiết tài nguyên từng module |
| Synthesis Report | `impl/gwsynthesis/picorv32_aes256_syn.rpt.html` | Báo cáo tổng hợp |
| PnR Report | `impl/pnr/picorv32_aes256.rpt.txt` | Tài nguyên sau Place&Route |
| Timing Report | `impl/pnr/picorv32_aes256.tr.html` | Phân tích timing |
| Pin Report | `impl/pnr/picorv32_aes256.pin.html` | Gán chân I/O |
| Power Report | `impl/pnr/picorv32_aes256.power.html` | Ước tính công suất |

### 6.2. Cách đọc Resource Report

#### Từ Gowin IDE:
1. Mở project trong Gowin IDE
2. Menu: **Process** → **Reports** → **Place & Route Report**
3. Hoặc mở trực tiếp file `impl/pnr/picorv32_aes256.rpt.html`

#### Các mục quan trọng:

**Resource Usage Summary:**
```
Resources                   | Usage          | Utilization
---------------------------|----------------|------------
Logic                      | 19705/59904    | 33%
  --LUT,ALU,ROM16          | 19561          | -
Register                   | 5959/60780     | 10%
BSRAM                      | 84/118         | 72%
DSP                        | 4/118          | 4%
I/O Port                   | 14/297         | 5%
```

### 6.3. Cách đọc Timing Report

#### Từ Gowin IDE:
1. Menu: **Process** → **Reports** → **Timing Report**
2. Hoặc mở file `impl/pnr/picorv32_aes256.tr.html`

#### Các mục quan trọng:

**Max Frequency Summary:**
```
Clock Name  | Constraint | Actual Fmax | Status
------------|------------|-------------|--------
clk_50m     | 15 MHz     | 18.8 MHz    | PASS
jtag_TCK    | 5 MHz      | 99 MHz      | PASS
```

**Critical Path:**
- Thường nằm trong khối AES (Key Expansion hoặc MixColumns)
- Path delay ~ 50-60 ns

---

## 7. SỐ LIỆU THỰC TẾ CỦA DỰ ÁN

### 7.1. Resource Utilization

| Tài nguyên | Sử dụng | Tổng có | % |
|------------|---------|---------|---|
| **Logic (LUT+ALU)** | 19,705 | 59,904 | **33%** |
| - LUT | 18,900 | - | - |
| - ALU | 661 | - | - |
| **Register (FF)** | 5,959 | 60,780 | **10%** |
| **Block RAM** | 84 | 118 | **72%** |
| - SDPB | 64 | - | - |
| - pROM | 20 | - | - |
| **DSP (MULT27X36)** | 4 | 118 | **4%** |
| **I/O Ports** | 14 | 297 | **5%** |
| **CLS** | 12,963 | 29,952 | **44%** |

### 7.2. Phân bố theo Module

| Module | Registers | LUTs | BSRAM | Ghi chú |
|--------|-----------|------|-------|---------|
| **picorv32_aes256_top** | 4,000 | 6,250 | 64 | Top level |
| **u_aes256** (total) | 1,959 | 12,664 | 20 | AES wrapper + core |
| - u_aes256_core | 1,405 | 12,244 | 20 | AES datapath |
| - key_exp_inst | 0 | 6,985 | 0 | Key expansion (comb) |
| **u_picorv32** | ~2,500 | ~3,000 | 64 | CPU + peripherals |

### 7.3. Timing Analysis

| Clock | Constraint Period | Actual Fmax | Slack |
|-------|-------------------|-------------|-------|
| clk_50m | 66.67 ns (15 MHz) | ~18.8 MHz | Positive |
| jtag_TCK | 200 ns (5 MHz) | ~99 MHz | Positive |

### 7.4. Build Time

| Phase | Thời gian |
|-------|-----------|
| Synthesis | ~30 giây |
| Placement | ~1 phút 9 giây |
| Routing | ~1 phút 2 giây |
| **Total** | **~2 phút 18 giây** |

### 7.5. Performance Metrics

| Metric | Giá trị |
|--------|---------|
| Clock Frequency | 15 MHz |
| Cycles per AES block | ~20 cycles |
| **Latency per block** | **~1.33 μs** |
| **Throughput** | **~96 Mbps** |

---

## 8. CÂU HỎI THƯỜNG GẶP TỪ HỘI ĐỒNG

### Q1: Tại sao chọn PicoRV32 thay vì các CPU khác?

**Trả lời:**
- Open-source, kiến trúc RISC-V hiện đại
- Kích thước nhỏ (~2000 LUTs), tiết kiệm tài nguyên
- Đã được tích hợp sẵn trong Gowin IP Core
- Dễ customize và tích hợp với hardware accelerator

### Q2: AES-256 Hardware nhanh hơn Software bao nhiêu lần?

**Trả lời:**
- Software trên PicoRV32: ~3000-5000 cycles/block
- Hardware accelerator: ~20 cycles/block
- **Tốc độ tăng: 150-250 lần**

### Q3: Tại sao Max Frequency chỉ đạt ~18 MHz?

**Trả lời:**
- Critical path nằm trong Key Expansion (combinational 6985 LUTs)
- Trade-off: Latency thấp (1 cycle) đổi lấy Fmax thấp
- Giải pháp nếu cần Fmax cao: Pipeline key expansion (tăng latency)

### Q4: Làm sao verify AES hoạt động đúng?

**Trả lời:**
- Sử dụng NIST FIPS-197 test vectors
- Test vector chính: Key=00010203...1e1f, Plaintext=00112233...eeff
- Expected Ciphertext: 8EA2B7CA516745BFEAFC49904B496089
- Đã test 15 vectors → 15/15 PASS

### Q5: Khó khăn lớn nhất khi thực hiện?

**Trả lời:**
1. **Timing closure**: Phải giảm clock từ 50MHz xuống 15MHz
2. **Endianness**: Xử lý big-endian (AES) vs little-endian (RISC-V)
3. **Debug**: Khó debug hardware, phải dùng simulation kỹ

### Q6: Hướng phát triển tiếp theo?

**Trả lời:**
- Thêm mode CBC, CTR, GCM
- Pipeline AES core để tăng throughput
- Tích hợp thêm SHA-256, RSA
- Tối ưu để đạt Fmax cao hơn

---

## PHỤ LỤC: CHECKLIST TRƯỚC BÁO CÁO

### Chuẩn bị Hardware:
- [ ] Board Tang Mega 60K
- [ ] Cáp USB-C (data + power)
- [ ] Laptop có cài Gowin IDE + Driver

### Chuẩn bị Software:
- [ ] Project đã build thành công
- [ ] Bitstream file (.fs) đã tạo
- [ ] Serial terminal (Putty/TeraTerm) sẵn sàng

### Chuẩn bị Slide:
- [ ] Sơ đồ khối hệ thống
- [ ] Bảng Resource Utilization
- [ ] Bảng Timing Analysis
- [ ] Video demo backup (phòng lỗi kỹ thuật)

### Chuẩn bị Demo:
- [ ] Test vector NIST để demo
- [ ] Kịch bản demo 5 phút
- [ ] Câu trả lời cho Q&A

---

*Tài liệu được tạo để hỗ trợ báo cáo đồ án PicoRV32 AES-256 SoC*
*Ngày tạo: 13/12/2025*
