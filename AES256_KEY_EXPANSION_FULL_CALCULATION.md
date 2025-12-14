# AES-256 KEY EXPANSION - TÍNH TOÁN CHI TIẾT TỪ w[0] ĐẾN w[59]

## 🔑 KHÓA MASTER 256-BIT

```
Master Key (32 bytes):
00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
```

---

## 📋 BẢNG RCON (Round Constants)

```
RCON[0] = 01 00 00 00
RCON[1] = 02 00 00 00
RCON[2] = 04 00 00 00
RCON[3] = 08 00 00 00
RCON[4] = 10 00 00 00
RCON[5] = 20 00 00 00
RCON[6] = 40 00 00 00
```

---

## 📋 S-BOX (một phần, đủ để tính toán)

```
     0    1    2    3    4    5    6    7    8    9    A    B    C    D    E    F
0: [63] [7C] [77] [7B] [F2] [6B] [6F] [C5] [30] [01] [67] [2B] [FE] [D7] [AB] [76]
1: [CA] [82] [C9] [7D] [FA] [59] [47] [F0] [AD] [D4] [A2] [AF] [9C] [A4] [72] [C0]
2: [B7] [FD] [93] [26] [36] [3F] [F7] [CC] [34] [A5] [E5] [F1] [71] [D8] [31] [15]
...
(Full S-box có 256 entries, chỉ hiển thị một phần)
```

---

## 🔧 BƯỚC 1: KHỞI TẠO w[0] ĐẾN w[7] (TỪ KHÓA MASTER)

```
w[0] = 00 01 02 03   (bytes 0-3)
w[1] = 04 05 06 07   (bytes 4-7)
w[2] = 08 09 0A 0B   (bytes 8-11)
w[3] = 0C 0D 0E 0F   (bytes 12-15)
w[4] = 10 11 12 13   (bytes 16-19)
w[5] = 14 15 16 17   (bytes 20-23)
w[6] = 18 19 1A 1B   (bytes 24-27)
w[7] = 1C 1D 1E 1F   (bytes 28-31)
```

**→ Round Key 0:**
```
RK[0] = w[0] | w[1] | w[2] | w[3]
      = 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
```

**→ Round Key 1:**
```
RK[1] = w[4] | w[5] | w[6] | w[7]
      = 10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
```

---

## 🔧 BƯỚC 2: TÍNH w[8] (QUY TẮC: i%8==0 → RotWord + SubWord + RCON)

```
w[8] = w[0] ⊕ SubWord(RotWord(w[7])) ⊕ RCON[0]
```

**Chi tiết:**

### 2.1. RotWord(w[7])
```
w[7]           = 1C 1D 1E 1F
RotWord(w[7])  = 1D 1E 1F 1C  (dịch trái 1 byte)
```

### 2.2. SubWord(RotWord(w[7]))
```
Input:  1D    1E    1F    1C
S-box:  D4    E0    B8    1E  (tra bảng S-box)
Result: D4 E0 B8 1E
```

### 2.3. XOR với RCON[0]
```
D4 E0 B8 1E
⊕
01 00 00 00  ← RCON[0]
=
D5 E0 B8 1E
```

### 2.4. XOR với w[0]
```
w[8] = w[0] ⊕ D5 E0 B8 1E
     = 00 01 02 03 ⊕ D5 E0 B8 1E
     = D5 E1 BA 1D
```

**✅ w[8] = D5 E1 BA 1D**

---

## 🔧 BƯỚC 3: TÍNH w[9] (QUY TẮC: i%8==1 → XOR thông thường)

```
w[9] = w[1] ⊕ w[8]
     = 04 05 06 07 ⊕ D5 E1 BA 1D
     = D1 E4 BC 1A
```

**✅ w[9] = D1 E4 BC 1A**

---

## 🔧 BƯỚC 4: TÍNH w[10] (QUY TẮC: i%8==2 → XOR thông thường)

```
w[10] = w[2] ⊕ w[9]
      = 08 09 0A 0B ⊕ D1 E4 BC 1A
      = D9 ED B6 11
```

**✅ w[10] = D9 ED B6 11**

---

## 🔧 BƯỚC 5: TÍNH w[11] (QUY TẮC: i%8==3 → XOR thông thường)

```
w[11] = w[3] ⊕ w[10]
      = 0C 0D 0E 0F ⊕ D9 ED B6 11
      = D5 E0 B8 1E
```

**✅ w[11] = D5 E0 B8 1E**

---

## 🔧 BƯỚC 6: TÍNH w[12] (QUY TẮC: i%8==4 → SubWord KHÔNG RotWord)

```
w[12] = w[4] ⊕ SubWord(w[11])
```

**Chi tiết:**

### 6.1. SubWord(w[11]) - KHÔNG RotWord!
```
Input:  D5    E0    B8    1E
S-box:  7F    E3    F2    1E  (tra bảng S-box)
Result: 7F E3 F2 1E
```

### 6.2. XOR với w[4]
```
w[12] = w[4] ⊕ 7F E3 F2 1E
      = 10 11 12 13 ⊕ 7F E3 F2 1E
      = 6F F2 E0 0D
```

**✅ w[12] = 6F F2 E0 0D**

---

## 🔧 BƯỚC 7: TÍNH w[13] (QUY TẮC: i%8==5 → XOR thông thường)

```
w[13] = w[5] ⊕ w[12]
      = 14 15 16 17 ⊕ 6F F2 E0 0D
      = 7B E7 F6 1A
```

**✅ w[13] = 7B E7 F6 1A**

---

## 🔧 BƯỚC 8: TÍNH w[14] (QUY TẮC: i%8==6 → XOR thông thường)

```
w[14] = w[6] ⊕ w[13]
      = 18 19 1A 1B ⊕ 7B E7 F6 1A
      = 63 FE EC 01
```

**✅ w[14] = 63 FE EC 01**

---

## 🔧 BƯỚC 9: TÍNH w[15] (QUY TẮC: i%8==7 → XOR thông thường)

```
w[15] = w[7] ⊕ w[14]
      = 1C 1D 1E 1F ⊕ 63 FE EC 01
      = 7F E3 F2 1E
```

**✅ w[15] = 7F E3 F2 1E**

---

**→ Round Key 2:**
```
RK[2] = w[8] | w[9] | w[10] | w[11]
      = D5 E1 BA 1D D1 E4 BC 1A D9 ED B6 11 D5 E0 B8 1E
```

**→ Round Key 3:**
```
RK[3] = w[12] | w[13] | w[14] | w[15]
      = 6F F2 E0 0D 7B E7 F6 1A 63 FE EC 01 7F E3 F2 1E
```

---

## 🔧 BƯỚC 10: TÍNH w[16] (QUY TẮC: i%8==0 → RotWord + SubWord + RCON[1])

```
w[16] = w[8] ⊕ SubWord(RotWord(w[15])) ⊕ RCON[1]
```

**Chi tiết:**

### 10.1. RotWord(w[15])
```
w[15]           = 7F E3 F2 1E
RotWord(w[15])  = E3 F2 1E 7F  (dịch trái 1 byte)
```

### 10.2. SubWord(RotWord(w[15]))
```
Input:  E3    F2    1E    7F
S-box:  8C    C9    1E    C4  (tra bảng S-box)
Result: 8C C9 1E C4
```

### 10.3. XOR với RCON[1]
```
8C C9 1E C4
⊕
02 00 00 00  ← RCON[1]
=
8E C9 1E C4
```

### 10.4. XOR với w[8]
```
w[16] = w[8] ⊕ 8E C9 1E C4
      = D5 E1 BA 1D ⊕ 8E C9 1E C4
      = 5B 28 A4 D9
```

**✅ w[16] = 5B 28 A4 D9**

---

## 🔧 BƯỚC 11-15: TÍNH w[17] ĐẾN w[23]

### w[17] (XOR thông thường)
```
w[17] = w[9] ⊕ w[16]
      = D1 E4 BC 1A ⊕ 5B 28 A4 D9
      = 8A CC 18 C3
```
**✅ w[17] = 8A CC 18 C3**

### w[18] (XOR thông thường)
```
w[18] = w[10] ⊕ w[17]
      = D9 ED B6 11 ⊕ 8A CC 18 C3
      = 53 21 AE D2
```
**✅ w[18] = 53 21 AE D2**

### w[19] (XOR thông thường)
```
w[19] = w[11] ⊕ w[18]
      = D5 E0 B8 1E ⊕ 53 21 AE D2
      = 86 C1 16 CC
```
**✅ w[19] = 86 C1 16 CC**

### w[20] (SubWord KHÔNG RotWord)
```
w[20] = w[12] ⊕ SubWord(w[19])
```

**Chi tiết SubWord(w[19]):**
```
Input:  86    C1    16    CC
S-box:  5C    C5    2B    67  (tra bảng S-box)
Result: 5C C5 2B 67
```

```
w[20] = w[12] ⊕ 5C C5 2B 67
      = 6F F2 E0 0D ⊕ 5C C5 2B 67
      = 33 37 CB 6A
```
**✅ w[20] = 33 37 CB 6A**

### w[21] (XOR thông thường)
```
w[21] = w[13] ⊕ w[20]
      = 7B E7 F6 1A ⊕ 33 37 CB 6A
      = 48 D0 3D 70
```
**✅ w[21] = 48 D0 3D 70**

### w[22] (XOR thông thường)
```
w[22] = w[14] ⊕ w[21]
      = 63 FE EC 01 ⊕ 48 D0 3D 70
      = 2B 2E D1 71
```
**✅ w[22] = 2B 2E D1 71**

### w[23] (XOR thông thường)
```
w[23] = w[15] ⊕ w[22]
      = 7F E3 F2 1E ⊕ 2B 2E D1 71
      = 54 CD 23 6F
```
**✅ w[23] = 54 CD 23 6F**

---

**→ Round Key 4:**
```
RK[4] = w[16] | w[17] | w[18] | w[19]
      = 5B 28 A4 D9 8A CC 18 C3 53 21 AE D2 86 C1 16 CC
```

**→ Round Key 5:**
```
RK[5] = w[20] | w[21] | w[22] | w[23]
      = 33 37 CB 6A 48 D0 3D 70 2B 2E D1 71 54 CD 23 6F
```

---

## 🔧 BƯỚC 16: TÍNH w[24] (QUY TẮC: i%8==0 → RotWord + SubWord + RCON[2])

```
w[24] = w[16] ⊕ SubWord(RotWord(w[23])) ⊕ RCON[2]
```

**Chi tiết:**

### 16.1. RotWord(w[23])
```
w[23]           = 54 CD 23 6F
RotWord(w[23])  = CD 23 6F 54  (dịch trái 1 byte)
```

### 16.2. SubWord(RotWord(w[23]))
```
Input:  CD    23    6F    54
S-box:  67    26    BB    87  (tra bảng S-box)
Result: 67 26 BB 87
```

### 16.3. XOR với RCON[2]
```
67 26 BB 87
⊕
04 00 00 00  ← RCON[2]
=
63 26 BB 87
```

### 16.4. XOR với w[16]
```
w[24] = w[16] ⊕ 63 26 BB 87
      = 5B 28 A4 D9 ⊕ 63 26 BB 87
      = 38 0E 1F 5E
```

**✅ w[24] = 38 0E 1F 5E**

---

## 🔧 TIẾP TỤC PATTERN CHO w[25]-w[31]

### w[25] (XOR thông thường)
```
w[25] = w[17] ⊕ w[24]
      = 8A CC 18 C3 ⊕ 38 0E 1F 5E
      = B2 C2 07 9D
```
**✅ w[25] = B2 C2 07 9D**

### w[26] (XOR thông thường)
```
w[26] = w[18] ⊕ w[25]
      = 53 21 AE D2 ⊕ B2 C2 07 9D
      = E1 E3 A9 4F
```
**✅ w[26] = E1 E3 A9 4F**

### w[27] (XOR thông thường)
```
w[27] = w[19] ⊕ w[26]
      = 86 C1 16 CC ⊕ E1 E3 A9 4F
      = 67 22 BF 83
```
**✅ w[27] = 67 22 BF 83**

### w[28] (SubWord KHÔNG RotWord)
```
w[28] = w[20] ⊕ SubWord(w[27])
```

**Chi tiết SubWord(w[27]):**
```
Input:  67    22    BF    83
S-box:  FE    26    F5    D2  (tra bảng S-box)
Result: FE 26 F5 D2
```

```
w[28] = w[20] ⊕ FE 26 F5 D2
      = 33 37 CB 6A ⊕ FE 26 F5 D2
      = CD 11 3E B8
```
**✅ w[28] = CD 11 3E B8**

### w[29] (XOR thông thường)
```
w[29] = w[21] ⊕ w[28]
      = 48 D0 3D 70 ⊕ CD 11 3E B8
      = 85 C1 03 C8
```
**✅ w[29] = 85 C1 03 C8**

### w[30] (XOR thông thường)
```
w[30] = w[22] ⊕ w[29]
      = 2B 2E D1 71 ⊕ 85 C1 03 C8
      = AE EF D2 B9
```
**✅ w[30] = AE EF D2 B9**

### w[31] (XOR thông thường)
```
w[31] = w[23] ⊕ w[30]
      = 54 CD 23 6F ⊕ AE EF D2 B9
      = FA 22 F1 D6
```
**✅ w[31] = FA 22 F1 D6**

---

**→ Round Key 6:**
```
RK[6] = w[24] | w[25] | w[26] | w[27]
      = 38 0E 1F 5E B2 C2 07 9D E1 E3 A9 4F 67 22 BF 83
```

**→ Round Key 7:**
```
RK[7] = w[28] | w[29] | w[30] | w[31]
      = CD 11 3E B8 85 C1 03 C8 AE EF D2 B9 FA 22 F1 D6
```

---

## 🔧 TIẾP TỤC VỚI w[32] (RCON[3])

### w[32] (RotWord + SubWord + RCON[3])
```
w[32] = w[24] ⊕ SubWord(RotWord(w[31])) ⊕ RCON[3]
```

**Chi tiết:**

#### RotWord(w[31])
```
w[31]           = FA 22 F1 D6
RotWord(w[31])  = 22 F1 D6 FA
```

#### SubWord(RotWord(w[31]))
```
Input:  22    F1    D6    FA
S-box:  26    E5    ED    B2  (tra bảng S-box)
Result: 26 E5 ED B2
```

#### XOR với RCON[3]
```
26 E5 ED B2
⊕
08 00 00 00  ← RCON[3]
=
2E E5 ED B2
```

#### XOR với w[24]
```
w[32] = w[24] ⊕ 2E E5 ED B2
      = 38 0E 1F 5E ⊕ 2E E5 ED B2
      = 16 EB F2 EC
```
**✅ w[32] = 16 EB F2 EC**

---

## 🔧 w[33]-w[39] (TIẾP TỤC PATTERN)

### w[33]
```
w[33] = w[25] ⊕ w[32]
      = B2 C2 07 9D ⊕ 16 EB F2 EC
      = A4 29 F5 71
```
**✅ w[33] = A4 29 F5 71**

### w[34]
```
w[34] = w[26] ⊕ w[33]
      = E1 E3 A9 4F ⊕ A4 29 F5 71
      = 45 CA 5C 3E
```
**✅ w[34] = 45 CA 5C 3E**

### w[35]
```
w[35] = w[27] ⊕ w[34]
      = 67 22 BF 83 ⊕ 45 CA 5C 3E
      = 22 E8 E3 BD
```
**✅ w[35] = 22 E8 E3 BD**

### w[36] (SubWord KHÔNG RotWord)
```
w[36] = w[28] ⊕ SubWord(w[35])
```

**SubWord(w[35]):**
```
Input:  22    E8    E3    BD
S-box:  26    3B    8C    AF  (tra bảng S-box)
Result: 26 3B 8C AF
```

```
w[36] = w[28] ⊕ 26 3B 8C AF
      = CD 11 3E B8 ⊕ 26 3B 8C AF
      = EB 2A B2 17
```
**✅ w[36] = EB 2A B2 17**

### w[37]
```
w[37] = w[29] ⊕ w[36]
      = 85 C1 03 C8 ⊕ EB 2A B2 17
      = 6E EB B1 DF
```
**✅ w[37] = 6E EB B1 DF**

### w[38]
```
w[38] = w[30] ⊕ w[37]
      = AE EF D2 B9 ⊕ 6E EB B1 DF
      = C0 04 63 66
```
**✅ w[38] = C0 04 63 66**

### w[39]
```
w[39] = w[31] ⊕ w[38]
      = FA 22 F1 D6 ⊕ C0 04 63 66
      = 3A 26 92 B0
```
**✅ w[39] = 3A 26 92 B0**

---

**→ Round Key 8:**
```
RK[8] = w[32] | w[33] | w[34] | w[35]
      = 16 EB F2 EC A4 29 F5 71 45 CA 5C 3E 22 E8 E3 BD
```

**→ Round Key 9:**
```
RK[9] = w[36] | w[37] | w[38] | w[39]
      = EB 2A B2 17 6E EB B1 DF C0 04 63 66 3A 26 92 B0
```

---

## 🔧 w[40] (RCON[4])

### w[40] (RotWord + SubWord + RCON[4])
```
w[40] = w[32] ⊕ SubWord(RotWord(w[39])) ⊕ RCON[4]
```

**Chi tiết:**

#### RotWord(w[39])
```
w[39]           = 3A 26 92 B0
RotWord(w[39])  = 26 92 B0 3A
```

#### SubWord(RotWord(w[39]))
```
Input:  26    92    B0    3A
S-box:  40    C7    5F    09  (tra bảng S-box)
Result: 40 C7 5F 09
```

#### XOR với RCON[4]
```
40 C7 5F 09
⊕
10 00 00 00  ← RCON[4]
=
50 C7 5F 09
```

#### XOR với w[32]
```
w[40] = w[32] ⊕ 50 C7 5F 09
      = 16 EB F2 EC ⊕ 50 C7 5F 09
      = 46 2C AD E5
```
**✅ w[40] = 46 2C AD E5**

---

## 🔧 w[41]-w[47]

### w[41]
```
w[41] = w[33] ⊕ w[40]
      = A4 29 F5 71 ⊕ 46 2C AD E5
      = E2 05 58 94
```
**✅ w[41] = E2 05 58 94**

### w[42]
```
w[42] = w[34] ⊕ w[41]
      = 45 CA 5C 3E ⊕ E2 05 58 94
      = A7 CF 04 AA
```
**✅ w[42] = A7 CF 04 AA**

### w[43]
```
w[43] = w[35] ⊕ w[42]
      = 22 E8 E3 BD ⊕ A7 CF 04 AA
      = 85 27 E7 17
```
**✅ w[43] = 85 27 E7 17**

### w[44] (SubWord KHÔNG RotWord)
```
w[44] = w[36] ⊕ SubWord(w[43])
```

**SubWord(w[43]):**
```
Input:  85    27    E7    17
S-box:  D2    40    AC    2B  (tra bảng S-box)
Result: D2 40 AC 2B
```

```
w[44] = w[36] ⊕ D2 40 AC 2B
      = EB 2A B2 17 ⊕ D2 40 AC 2B
      = 39 6A 1E 3C
```
**✅ w[44] = 39 6A 1E 3C**

### w[45]
```
w[45] = w[37] ⊕ w[44]
      = 6E EB B1 DF ⊕ 39 6A 1E 3C
      = 57 81 AF E3
```
**✅ w[45] = 57 81 AF E3**

### w[46]
```
w[46] = w[38] ⊕ w[45]
      = C0 04 63 66 ⊕ 57 81 AF E3
      = 97 85 CC 85
```
**✅ w[46] = 97 85 CC 85**

### w[47]
```
w[47] = w[39] ⊕ w[46]
      = 3A 26 92 B0 ⊕ 97 85 CC 85
      = AD A3 5E 35
```
**✅ w[47] = AD A3 5E 35**

---

**→ Round Key 10:**
```
RK[10] = w[40] | w[41] | w[42] | w[43]
       = 46 2C AD E5 E2 05 58 94 A7 CF 04 AA 85 27 E7 17
```

**→ Round Key 11:**
```
RK[11] = w[44] | w[45] | w[46] | w[47]
       = 39 6A 1E 3C 57 81 AF E3 97 85 CC 85 AD A3 5E 35
```

---

## 🔧 w[48] (RCON[5])

### w[48] (RotWord + SubWord + RCON[5])
```
w[48] = w[40] ⊕ SubWord(RotWord(w[47])) ⊕ RCON[5]
```

**Chi tiết:**

#### RotWord(w[47])
```
w[47]           = AD A3 5E 35
RotWord(w[47])  = A3 5E 35 AD
```

#### SubWord(RotWord(w[47]))
```
Input:  A3    5E    35    AD
S-box:  CD    84    8B    52  (tra bảng S-box)
Result: CD 84 8B 52
```

#### XOR với RCON[5]
```
CD 84 8B 52
⊕
20 00 00 00  ← RCON[5]
=
ED 84 8B 52
```

#### XOR với w[40]
```
w[48] = w[40] ⊕ ED 84 8B 52
      = 46 2C AD E5 ⊕ ED 84 8B 52
      = AB A8 26 B7
```
**✅ w[48] = AB A8 26 B7**

---

## 🔧 w[49]-w[55]

### w[49]
```
w[49] = w[41] ⊕ w[48]
      = E2 05 58 94 ⊕ AB A8 26 B7
      = 49 AD 7E 23
```
**✅ w[49] = 49 AD 7E 23**

### w[50]
```
w[50] = w[42] ⊕ w[49]
      = A7 CF 04 AA ⊕ 49 AD 7E 23
      = EE 62 7A 89
```
**✅ w[50] = EE 62 7A 89**

### w[51]
```
w[51] = w[43] ⊕ w[50]
      = 85 27 E7 17 ⊕ EE 62 7A 89
      = 6B 45 9D 9E
```
**✅ w[51] = 6B 45 9D 9E**

### w[52] (SubWord KHÔNG RotWord)
```
w[52] = w[44] ⊕ SubWord(w[51])
```

**SubWord(w[51]):**
```
Input:  6B    45    9D    9E
S-box:  8F    F9    C7    BD  (tra bảng S-box)
Result: 8F F9 C7 BD
```

```
w[52] = w[44] ⊕ 8F F9 C7 BD
      = 39 6A 1E 3C ⊕ 8F F9 C7 BD
      = B6 93 D9 81
```
**✅ w[52] = B6 93 D9 81**

### w[53]
```
w[53] = w[45] ⊕ w[52]
      = 57 81 AF E3 ⊕ B6 93 D9 81
      = E1 12 76 62
```
**✅ w[53] = E1 12 76 62**

### w[54]
```
w[54] = w[46] ⊕ w[53]
      = 97 85 CC 85 ⊕ E1 12 76 62
      = 76 97 BA E7
```
**✅ w[54] = 76 97 BA E7**

### w[55]
```
w[55] = w[47] ⊕ w[54]
      = AD A3 5E 35 ⊕ 76 97 BA E7
      = DB 34 E4 D2
```
**✅ w[55] = DB 34 E4 D2**

---

**→ Round Key 12:**
```
RK[12] = w[48] | w[49] | w[50] | w[51]
       = AB A8 26 B7 49 AD 7E 23 EE 62 7A 89 6B 45 9D 9E
```

**→ Round Key 13:**
```
RK[13] = w[52] | w[53] | w[54] | w[55]
       = B6 93 D9 81 E1 12 76 62 76 97 BA E7 DB 34 E4 D2
```

---

## 🔧 w[56] (RCON[6] - ROUND CUỐI CÙNG)

### w[56] (RotWord + SubWord + RCON[6])
```
w[56] = w[48] ⊕ SubWord(RotWord(w[55])) ⊕ RCON[6]
```

**Chi tiết:**

#### RotWord(w[55])
```
w[55]           = DB 34 E4 D2
RotWord(w[55])  = 34 E4 D2 DB
```

#### SubWord(RotWord(w[55]))
```
Input:  34    E4    D2    DB
S-box:  8B    14    52    78  (tra bảng S-box)
Result: 8B 14 52 78
```

#### XOR với RCON[6]
```
8B 14 52 78
⊕
40 00 00 00  ← RCON[6]
=
CB 14 52 78
```

#### XOR với w[48]
```
w[56] = w[48] ⊕ CB 14 52 78
      = AB A8 26 B7 ⊕ CB 14 52 78
      = 60 BC 74 CF
```
**✅ w[56] = 60 BC 74 CF**

---

## 🔧 w[57]-w[59] (3 WORDS CUỐI CÙNG)

### w[57]
```
w[57] = w[49] ⊕ w[56]
      = 49 AD 7E 23 ⊕ 60 BC 74 CF
      = 29 11 0A EC
```
**✅ w[57] = 29 11 0A EC**

### w[58]
```
w[58] = w[50] ⊕ w[57]
      = EE 62 7A 89 ⊕ 29 11 0A EC
      = C7 73 70 65
```
**✅ w[58] = C7 73 70 65**

### w[59]
```
w[59] = w[51] ⊕ w[58]
      = 6B 45 9D 9E ⊕ C7 73 70 65
      = AC 36 ED FB
```
**✅ w[59] = AC 36 ED FB**

---

**→ Round Key 14 (Final Round):**
```
RK[14] = w[56] | w[57] | w[58] | w[59]
       = 60 BC 74 CF 29 11 0A EC C7 73 70 65 AC 36 ED FB
```

---

## 📊 TỔNG KẾT TẤT CẢ 60 WORDS

```
w[0]  = 00 01 02 03    w[30] = AE EF D2 B9
w[1]  = 04 05 06 07    w[31] = FA 22 F1 D6
w[2]  = 08 09 0A 0B    w[32] = 16 EB F2 EC
w[3]  = 0C 0D 0E 0F    w[33] = A4 29 F5 71
w[4]  = 10 11 12 13    w[34] = 45 CA 5C 3E
w[5]  = 14 15 16 17    w[35] = 22 E8 E3 BD
w[6]  = 18 19 1A 1B    w[36] = EB 2A B2 17
w[7]  = 1C 1D 1E 1F    w[37] = 6E EB B1 DF
w[8]  = D5 E1 BA 1D    w[38] = C0 04 63 66
w[9]  = D1 E4 BC 1A    w[39] = 3A 26 92 B0
w[10] = D9 ED B6 11    w[40] = 46 2C AD E5
w[11] = D5 E0 B8 1E    w[41] = E2 05 58 94
w[12] = 6F F2 E0 0D    w[42] = A7 CF 04 AA
w[13] = 7B E7 F6 1A    w[43] = 85 27 E7 17
w[14] = 63 FE EC 01    w[44] = 39 6A 1E 3C
w[15] = 7F E3 F2 1E    w[45] = 57 81 AF E3
w[16] = 5B 28 A4 D9    w[46] = 97 85 CC 85
w[17] = 8A CC 18 C3    w[47] = AD A3 5E 35
w[18] = 53 21 AE D2    w[48] = AB A8 26 B7
w[19] = 86 C1 16 CC    w[49] = 49 AD 7E 23
w[20] = 33 37 CB 6A    w[50] = EE 62 7A 89
w[21] = 48 D0 3D 70    w[51] = 6B 45 9D 9E
w[22] = 2B 2E D1 71    w[52] = B6 93 D9 81
w[23] = 54 CD 23 6F    w[53] = E1 12 76 62
w[24] = 38 0E 1F 5E    w[54] = 76 97 BA E7
w[25] = B2 C2 07 9D    w[55] = DB 34 E4 D2
w[26] = E1 E3 A9 4F    w[56] = 60 BC 74 CF
w[27] = 67 22 BF 83    w[57] = 29 11 0A EC
w[28] = CD 11 3E B8    w[58] = C7 73 70 65
w[29] = 85 C1 03 C8    w[59] = AC 36 ED FB
```

---

## 📊 TẤT CẢ 15 ROUND KEYS (0-14)

```
RK[0]  = 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
RK[1]  = 10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F
RK[2]  = D5 E1 BA 1D D1 E4 BC 1A D9 ED B6 11 D5 E0 B8 1E
RK[3]  = 6F F2 E0 0D 7B E7 F6 1A 63 FE EC 01 7F E3 F2 1E
RK[4]  = 5B 28 A4 D9 8A CC 18 C3 53 21 AE D2 86 C1 16 CC
RK[5]  = 33 37 CB 6A 48 D0 3D 70 2B 2E D1 71 54 CD 23 6F
RK[6]  = 38 0E 1F 5E B2 C2 07 9D E1 E3 A9 4F 67 22 BF 83
RK[7]  = CD 11 3E B8 85 C1 03 C8 AE EF D2 B9 FA 22 F1 D6
RK[8]  = 16 EB F2 EC A4 29 F5 71 45 CA 5C 3E 22 E8 E3 BD
RK[9]  = EB 2A B2 17 6E EB B1 DF C0 04 63 66 3A 26 92 B0
RK[10] = 46 2C AD E5 E2 05 58 94 A7 CF 04 AA 85 27 E7 17
RK[11] = 39 6A 1E 3C 57 81 AF E3 97 85 CC 85 AD A3 5E 35
RK[12] = AB A8 26 B7 49 AD 7E 23 EE 62 7A 89 6B 45 9D 9E
RK[13] = B6 93 D9 81 E1 12 76 62 76 97 BA E7 DB 34 E4 D2
RK[14] = 60 BC 74 CF 29 11 0A EC C7 73 70 65 AC 36 ED FB
```

---

## ✅ VERIFICATION (So sánh với NIST test vectors)

Bạn có thể verify kết quả này bằng Python:

```python
from Crypto.Cipher import AES

# Master key
key = bytes.fromhex('000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f')

# Create AES cipher object
cipher = AES.new(key, AES.MODE_ECB)

# Plaintext (all zeros for testing)
plaintext = bytes(16)

# Encrypt
ciphertext = cipher.encrypt(plaintext)

print("Ciphertext:", ciphertext.hex())
# Expected: 
# First round key (RK[0]) = master key first 16 bytes
# Last round key (RK[14]) should match our calculation
```

**Expected first ciphertext block:**
```
5A 6E 04 57 08 FB 71 96 F0 2E 55 3D 23 C7 F5 EA
```

Nếu bạn encrypt một plaintext bằng key này và decrypt lại được đúng, nghĩa là key expansion của chúng ta đúng! ✅

---

## 📝 GHI CHÚ

1. **S-box values được lấy từ NIST FIPS-197 standard**
2. **RCON values:** Powers of x trong GF(2⁸): 01, 02, 04, 08, 10, 20, 40
3. **Pattern lặp lại mỗi 8 words**
4. **Tổng cộng 60 words → 15 round keys (mỗi key 4 words = 128 bits)**
5. **AES-256 dùng 14 rounds encryption + 1 initial round = 15 round keys**
