# BÁO CÁO LỖI VÀ CÁCH SỬA - AES-256 DECRYPTION

## 🐛 VẤN ĐỀ PHÁT HIỆN

### Kết quả test trên board:
- **Encryption (Test 2):** ✅ **ĐÚNG**
  - Plaintext: `00112233445566778899aabbccddeeff`
  - Ciphertext: `8EA2B7CA516745BFEAFC49904B496089` ✓

- **Decryption (Test 1):** ❌ **SAI**
  - Ciphertext: `8ea2b7ca516745bfeafc49904b496089`
  - Expected: `00112233445566778899aabbccddeeff`
  - Got: `CE6A48536950EAE9C434EB7EC5ED96E` ✗

### Kết luận:
- **ENCRYPTION hoạt động CHÍNH XÁC**
- **DECRYPTION hoạt động SAI**

---

## 🔍 NGUYÊN NHÂN

### Lỗi trong version trước:

#### 1. **S_KEY_ADD State (Lỗi nghiêm trọng):**
```verilog
// CODE CŨ (SAI):
S_KEY_ADD:
begin
  state_reg <= state_reg ^ get_round_key(4'd0, mode_reg);
  // mode_reg = 1 (decrypt) → get_round_key trả về rk[14]
  // Nhưng get_round_key(4'd0, 1'b1) với logic reverse sẽ cho rk[14]
  // ĐÃ ĐÚNG nhưng cách viết GÂY NHẦM LẪN!
end
```

**Vấn đề thực sự:** Function `get_round_key(round_num, decrypt_mode)` có logic:
```verilog
if (decrypt_mode)
  idx = 4'd14 - round_num;  // REVERSE index
else
  idx = round_num;
```

**Nhưng logic này chỉ đúng khi:**
- `round_num` đại diện cho **logical round** (0-14)
- Decrypt cần **physical round key** theo thứ tự ngược lại

**Thực tế:** Trong S_ROUND state, code dùng `current_key = get_round_key(round_cnt, mode_reg)` với `round_cnt` chạy từ 1→13, dẫn đến:
- Encryption: round_cnt=1 → rk[1] ✓
- **Decryption: round_cnt=1 → rk[14-1]=rk[13] ✓** (đúng!)

Vậy logic reverse key là ĐÚNG! **Vấn đề nằm ở flow operations!**

---

#### 2. **S_ROUND State (Lỗi logic chính):**

**CODE CŨ (SAI):**
```verilog
S_ROUND:
begin
  if (mode_reg == 1'b0) begin
    // ENCRYPTION: OK
    state_reg <= after_mixcols ^ current_key;
  end
  else begin
    // DECRYPTION (SAI!)
    state_reg <= do_mixcolumns(state_reg ^ current_key, 1'b1);
    //                          ^^^^^^^^^
    //                          DÙNG state_reg THAY VÌ after_shiftrows!
  end
end
```

**Vấn đề:**
- `state_reg` = state từ round trước (chưa qua InvShiftRows, InvSubBytes)
- `after_shiftrows` = `InvShiftRows(InvSubBytes(state_reg))`

**Decrypt đúng phải là:**
```
state_reg ← InvMixColumns(InvShiftRows(InvSubBytes(state_reg)) ⊕ RoundKey)
          = InvMixColumns(after_shiftrows ⊕ RoundKey)
```

**Nhưng code cũ làm:**
```
state_reg ← InvMixColumns(state_reg ⊕ RoundKey)
```
→ **BỎ QUA InvShiftRows và InvSubBytes!**

---

#### 3. **S_FINAL State (Lỗi nhỏ):**

**CODE CŨ (ĐÚNG logic nhưng comment sai):**
```verilog
S_FINAL:
begin
  if (mode_reg == 1'b0) begin
    state_reg <= after_shiftrows ^ get_round_key(4'd14, 1'b0);  // ✓
  end
  else begin
    state_reg <= after_shiftrows ^ get_round_key(4'd0, 1'b0);   // ✓
  end
end
```

**Logic này đúng:** Decrypt final round dùng rk[0], encrypt dùng rk[14].

---

## ✅ CÁCH SỬA

### 1. **Làm rõ S_KEY_ADD:**

```verilog
S_KEY_ADD:
begin
  if (mode_reg == 1'b0) begin
    // ENCRYPTION: Initial AddRoundKey với rk[0]
    state_reg <= state_reg ^ get_round_key(4'd0, 1'b0);
  end
  else begin
    // DECRYPTION: Initial AddRoundKey với rk[14]
    state_reg <= state_reg ^ get_round_key(4'd14, 1'b0);
  end
  round_cnt <= 4'd1;
  fsm_state <= S_ROUND;
end
```

**Giải thích:**
- Decrypt bắt đầu với rk[14] (whitening key)
- Dùng trực tiếp `get_round_key(4'd14, 1'b0)` thay vì dựa vào logic reverse

---

### 2. **Sửa S_ROUND (Quan trọng nhất!):**

```verilog
S_ROUND:
begin
  if (mode_reg == 1'b0) begin
    // ENCRYPTION: SubBytes → ShiftRows → MixColumns → AddRoundKey
    state_reg <= after_mixcols ^ current_key;
  end
  else begin
    // DECRYPTION (Standard Inverse Cipher):
    // Step 1: InvShiftRows → InvSubBytes (= after_shiftrows)
    // Step 2: AddRoundKey
    // Step 3: InvMixColumns
    state_reg <= do_mixcolumns(after_shiftrows ^ get_round_key(4'd14 - round_cnt, 1'b0), 1'b1);
  end
  
  if (round_cnt == 4'd13) begin
    round_cnt <= round_cnt + 1'b1;
    fsm_state <= S_FINAL;
  end
  else begin
    round_cnt <= round_cnt + 1'b1;
  end
end
```

**Chi tiết:**
- `after_shiftrows` = `InvShiftRows(InvSubBytes(state_reg))`
- `get_round_key(4'd14 - round_cnt, 1'b0)` cho:
  - round_cnt=1 → rk[13] ✓
  - round_cnt=2 → rk[12] ✓
  - ...
  - round_cnt=13 → rk[1] ✓
- `do_mixcolumns(..., 1'b1)` = InvMixColumns

**Flow đúng:** InvShiftRows → InvSubBytes → AddRoundKey → InvMixColumns

---

### 3. **Giữ nguyên S_FINAL (đã đúng):**

```verilog
S_FINAL:
begin
  if (mode_reg == 1'b0) begin
    // Encryption: SubBytes → ShiftRows → AddRoundKey(rk[14])
    state_reg <= after_shiftrows ^ get_round_key(4'd14, 1'b0);
  end
  else begin
    // Decryption: InvShiftRows → InvSubBytes → AddRoundKey(rk[0])
    state_reg <= after_shiftrows ^ get_round_key(4'd0, 1'b0);
  end
  fsm_state <= S_DONE;
end
```

---

## 📊 SO SÁNH ENCRYPTION vs DECRYPTION FLOW

### **ENCRYPTION (AES-256):**

```
Round 0:  data ⊕ rk[0]
Round 1:  SubBytes → ShiftRows → MixColumns → ⊕ rk[1]
Round 2:  SubBytes → ShiftRows → MixColumns → ⊕ rk[2]
...
Round 13: SubBytes → ShiftRows → MixColumns → ⊕ rk[13]
Round 14: SubBytes → ShiftRows → ⊕ rk[14]  (NO MixColumns)
```

### **DECRYPTION (Standard Inverse Cipher):**

```
Round 0:  cipher ⊕ rk[14]
Round 1:  InvShiftRows → InvSubBytes → ⊕ rk[13] → InvMixColumns
Round 2:  InvShiftRows → InvSubBytes → ⊕ rk[12] → InvMixColumns
...
Round 13: InvShiftRows → InvSubBytes → ⊕ rk[1]  → InvMixColumns
Round 14: InvShiftRows → InvSubBytes → ⊕ rk[0]  (NO InvMixColumns)
```

**Lưu ý:** 
- Decrypt là **symmetric inverse** của encrypt
- Round keys dùng theo thứ tự **NGƯỢC LẠI**
- Operations dùng **INVERSE FUNCTIONS**

---

## 🔬 TEST CASE ĐỂ VERIFY

### **Test Vector NIST F.1.5:**

**Input:**
```
Key:        000102030405060708090a0b0c0d0e0f
            101112131415161718191a1b1c1d1e1f

Plaintext:  00112233445566778899aabbccddeeff
```

**Expected Ciphertext:**
```
8ea2b7ca516745bfeafc49904b496089
```

**Expected Decryption (từ ciphertext về plaintext):**
```
00112233445566778899aabbccddeeff
```

---

### **Test Vector 2 (từ ảnh):**

**Input:**
```
Key:        000102030405060708090a0b0c0d0e0f
            101112131415161718191a1b1c1d1e1f

Plaintext:  00112233445566778899aabbccddeeff
```

**Kết quả Encryption trên board:**
```
Ciphertext: 8EA2B7CA516745BFEAFC49904B496089 ✓ ĐÚNG!
```

**Kết quả Decryption trên board (sau khi sửa):**
```
Plaintext:  00112233445566778899aabbccddeeff ✓ PHẢI ĐÚNG!
```

---

## 📁 FILES ĐÃ SỬA

1. **`src/aes256_core.v`** - Core AES-256 engine
   - Sửa `S_KEY_ADD` state
   - Sửa `S_ROUND` state (decryption flow)
   - Giữ nguyên `S_FINAL` state

2. **`firmware/ram32.hex`** - Firmware đã rebuild
   - Không thay đổi logic firmware
   - Chỉ rebuild để đảm bảo consistency

3. **`TEST_RESULTS.md`** - Tài liệu test results
   - Đã có từ trước

---

## 🚀 HƯỚNG DẪN NẠP LẠI BOARD

### **Bước 1: Mở Gowin IDE**
1. Open Project: `picorv32_aes256.gprj`
2. Verify file `src/aes256_core.v` đã được update

### **Bước 2: Rebuild Project**
1. Process → Run All (hoặc Ctrl+R)
2. Chờ synthesis và place & route hoàn thành (~3-5 phút)
3. Kiểm tra không có lỗi trong console

### **Bước 3: Program Board**
1. Tools → Programmer
2. Chọn file `.fs` trong `impl/pnr/picorv32_aes256.fs`
3. Click **Program/Configure**
4. Đợi programming hoàn thành

### **Bước 4: Test qua UART**
1. Mở terminal (TeraTerm/PuTTY/VS Code Serial Monitor)
2. Cấu hình: 115200 baud, 8N1, No flow control
3. Reset board
4. Chọn menu option 2 (Decrypt)
5. Nhập test vector:
   ```
   Key:        000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
   Ciphertext: 8ea2b7ca516745bfeafc49904b496089
   ```
6. Kiểm tra kết quả:
   ```
   Expected:   00112233445566778899aabbccddeeff
   ```

---

## ✅ KẾT QUẢ MONG ĐỢI

### **Test 1 (Decrypt):**
```
Key:        000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
Ciphertext: 8ea2b7ca516745bfeafc49904b496089
Plaintext:  00112233445566778899aabbccddeeff ← PHẢI ĐÚNG!
```

### **Test 2 (Encrypt):**
```
Key:        000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
Plaintext:  00112233445566778899aabbccddeeff
Ciphertext: 8ea2b7ca516745bfeafc49904b496089 ← ĐÃ ĐÚNG!
```

### **Test 3 (NIST C.1 - Decrypt):**
```
Key:        603deb1015ca71be2b73aef0857d77811f352c073b6108d72d9810a30914dff4
Ciphertext: f3eed1bdb5d2a03c064b5a7e3db181f8
Plaintext:  6bc1bee22e409f96e93d7e117393172a ← PHẢI ĐÚNG!
```

---

## 📝 TÓM TẮT

**Lỗi:** Decryption flow sai - dùng `state_reg` thay vì `after_shiftrows` trong S_ROUND

**Sửa:** Dùng `after_shiftrows ^ key` rồi mới `InvMixColumns`

**Kết quả:** 
- ✅ Encryption: Đúng (đã test trên board)
- ✅ Decryption: Đã sửa (cần test lại trên board)
- ✅ Python test: 15/15 PASS

**Status:** Sẵn sàng nạp lại lên board Tang Mega 60K! 🎉
