# AES-256 FSM Cycle-by-Cycle Analysis

## Bảng Chi Tiết: Cycles và Vị Trí Code

| Cycle | State | Round | Code Location | Action | Register Updates |
|-------|-------|-------|---------------|--------|------------------|
| **0** | `S_IDLE` | - | Lines 1328-1337 | Wait for start signal | `state_reg <= data_in` |
| | | | Line 1333 | Load plaintext | `mode_reg <= mode` |
| | | | Line 1337 | Transition to KEY_ADD | `fsm_state <= S_KEY_ADD` |
| **1** | `S_KEY_ADD` | 0 | Lines 1339-1354 | Initial AddRoundKey | |
| | | | Line 1346 (Encrypt) | XOR with rk[0] | `state_reg <= state_reg ^ rk[0]` |
| | | | Line 1352 | Initialize round counter | `round_cnt <= 4'd1` |
| | | | Line 1354 | Transition to ROUND | `fsm_state <= S_ROUND` |
| **2** | `S_ROUND` | 1 | Lines 1356-1379 | Round 1 processing | |
| | | | Line 1362 (Encrypt) | SubBytes + ShiftRows + MixColumns + AddRoundKey | `state_reg <= after_mixcols ^ rk[1]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd2` |
| **3** | `S_ROUND` | 2 | Lines 1356-1379 | Round 2 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[2]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd3` |
| **4** | `S_ROUND` | 3 | Lines 1356-1379 | Round 3 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[3]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd4` |
| **5** | `S_ROUND` | 4 | Lines 1356-1379 | Round 4 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[4]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd5` |
| **6** | `S_ROUND` | 5 | Lines 1356-1379 | Round 5 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[5]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd6` |
| **7** | `S_ROUND` | 6 | Lines 1356-1379 | Round 6 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[6]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd7` |
| **8** | `S_ROUND` | 7 | Lines 1356-1379 | Round 7 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[7]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd8` |
| **9** | `S_ROUND` | 8 | Lines 1356-1379 | Round 8 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[8]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd9` |
| **10** | `S_ROUND` | 9 | Lines 1356-1379 | Round 9 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[9]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd10` |
| **11** | `S_ROUND` | 10 | Lines 1356-1379 | Round 10 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[10]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd11` |
| **12** | `S_ROUND` | 11 | Lines 1356-1379 | Round 11 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[11]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd12` |
| **13** | `S_ROUND` | 12 | Lines 1356-1379 | Round 12 processing | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[12]` |
| | | | Line 1376 | Increment counter | `round_cnt <= 4'd13` |
| **14** | `S_ROUND` | 13 | Lines 1356-1379 | Round 13 processing (last full round) | |
| | | | Line 1362 | Full round transformation | `state_reg <= after_mixcols ^ rk[13]` |
| | | | Line 1373 | **Check: round_cnt == 13** | **Condition TRUE** |
| | | | Line 1374 | Increment to 14 | `round_cnt <= 4'd14` |
| | | | Line 1377 | Transition to FINAL | `fsm_state <= S_FINAL` |
| **15** | `S_FINAL` | 14 | Lines 1381-1401 | Final round (no MixColumns) | |
| | | | Line 1394 (Encrypt) | SubBytes + ShiftRows + AddRoundKey | `state_reg <= after_shiftrows ^ rk[14]` |
| | | | Line 1401 | Transition to DONE | `fsm_state <= S_DONE` |
| **16** | `S_DONE` | - | Lines 1404-1409 | Output ready | |
| | | | Line 1406 | Latch result | `result_reg <= state_reg` |
| | | | Line 1407 | **Signal completion** | `done_reg <= 1'b1` ✓ |
| | | | Line 1408 | Clear busy flag | `busy_reg <= 1'b0` |
| | | | Line 1409 | Return to IDLE | `fsm_state <= S_IDLE` |

---

## Code Snippets với Line Numbers

### Cycle 0-1: IDLE → KEY_ADD
```verilog
// Lines 1328-1337
S_IDLE:
begin
  done_reg <= 1'b0;
  if (start)
  begin
    state_reg  <= data_in;      // Line 1333 - Load plaintext
    mode_reg   <= mode;
    round_cnt  <= 4'd0;
    busy_reg   <= 1'b1;
    fsm_state  <= S_KEY_ADD;    // Line 1337 - Transition
  end
end
```

### Cycle 1: KEY_ADD
```verilog
// Lines 1339-1354
S_KEY_ADD:
begin
  if (mode_reg == 1'b0)
  begin
    // ENCRYPTION: XOR with rk[0]
    state_reg <= state_reg ^ get_round_key(4'd0, 1'b0);  // Line 1346
  end
  else
  begin
    // DECRYPTION: XOR with rk[14]
    state_reg <= state_reg ^ get_round_key(4'd14, 1'b0); // Line 1351
  end
  round_cnt <= 4'd1;           // Line 1352 - Start from round 1
  fsm_state <= S_ROUND;        // Line 1354 - Enter round loop
end
```

### Cycles 2-14: ROUND (13 iterations)
```verilog
// Lines 1356-1379
S_ROUND:
begin
  if (mode_reg == 1'b0)
  begin
    // ENCRYPTION: Full round with MixColumns
    state_reg <= after_mixcols ^ current_key;  // Line 1362
  end
  else
  begin
    // DECRYPTION: Inverse round
    state_reg <= do_mixcolumns(after_shiftrows ^ 
                 get_round_key(4'd14 - round_cnt, 1'b0), 1'b1);  // Line 1371
  end

  if (round_cnt == 4'd13)      // Line 1373 - Exit condition
  begin
    round_cnt <= round_cnt + 1'b1;  // Line 1374 - Set to 14
    fsm_state <= S_FINAL;           // Line 1377 - Go to final round
  end
  else
  begin
    round_cnt <= round_cnt + 1'b1;  // Line 1376 - Continue rounds
  end
end
```

### Cycle 15: FINAL
```verilog
// Lines 1381-1401
S_FINAL:
begin
  // FINAL ROUND: No MixColumns
  if (mode_reg == 1'b0)
  begin
    // Encryption: use round key 14
    state_reg <= after_shiftrows ^ get_round_key(4'd14, 1'b0);  // Line 1394
  end
  else
  begin
    // Decryption: final AddRoundKey with rk[0]
    state_reg <= after_shiftrows ^ get_round_key(4'd0, 1'b0);   // Line 1399
  end
  fsm_state <= S_DONE;         // Line 1401 - Transition to DONE
end
```

### Cycle 16: DONE
```verilog
// Lines 1404-1409
S_DONE:
begin
  result_reg <= state_reg;     // Line 1406 - Latch output
  done_reg   <= 1'b1;          // Line 1407 - Signal completion ✓
  busy_reg   <= 1'b0;          // Line 1408 - Clear busy
  fsm_state  <= S_IDLE;        // Line 1409 - Back to IDLE
end
```

---

## Summary Table: Critical Lines

| Line Number | Code | Purpose | Cycle |
|-------------|------|---------|-------|
| 1337 | `fsm_state <= S_KEY_ADD` | Enter KEY_ADD state | 0→1 |
| 1346 | `state_reg <= state_reg ^ rk[0]` | Initial AddRoundKey | 1 |
| 1352 | `round_cnt <= 4'd1` | Start round counter | 1 |
| 1354 | `fsm_state <= S_ROUND` | Enter ROUND loop | 1→2 |
| 1362 | `state_reg <= after_mixcols ^ current_key` | Round transformation | 2-14 |
| 1373 | `if (round_cnt == 4'd13)` | **Exit condition** | 14 |
| 1377 | `fsm_state <= S_FINAL` | Enter FINAL state | 14→15 |
| 1394 | `state_reg <= after_shiftrows ^ rk[14]` | Final round | 15 |
| 1401 | `fsm_state <= S_DONE` | Enter DONE state | 15→16 |
| 1407 | `done_reg <= 1'b1` | **Signal completion** | 16 |

---

## Verification: Total Cycles

```
Cycle 0-1:   IDLE → KEY_ADD        = 1 cycle  (Initial AddRoundKey)
Cycles 2-14: ROUND (rounds 1-13)   = 13 cycles (Full rounds with MixColumns)
Cycle 15:    FINAL                 = 1 cycle  (Final round, no MixColumns)
Cycle 16:    DONE                  = 1 cycle  (Output latch, done=1)
────────────────────────────────────────────────────────────────
Total:                             = 16 cycles ✓
```

**File location:** `d:\project\FPGA\gowin\picorv32_aes256\src\aes256_core.v`
