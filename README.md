# Booth's Algorithm Multiplier — Verilog Implementation

A hardware implementation of **Booth's Multiplication Algorithm** for signed 4-bit integers, written in Verilog. Produces a signed 8-bit product.

---

## Algorithm Overview

[Booth's algorithm](https://en.wikipedia.org/wiki/Booth%27s_multiplication_algorithm) is an efficient method for multiplying two signed binary integers in two's complement representation. It reduces the number of partial additions by examining pairs of bits in the multiplier, enabling both positive and negative multipliers to be handled uniformly.

Each iteration inspects the current LSB of the multiplier (`QR[0]`) alongside the previously shifted-out bit (`Qn1`):

| `QR[0]` | `Qn1` | Action          |
|---------|-------|-----------------|
| 0       | 0     | No operation    |
| 1       | 1     | No operation    |
| 0       | 1     | Add `BR` to `AC`|
| 1       | 0     | Subtract `BR` from `AC` |

After each step, the combined register `{AC, QR, Qn1}` is **arithmetic right-shifted by 1**.

---

## Module: `booth`

### Ports

| Port      | Direction | Width   | Description                        |
|-----------|-----------|---------|------------------------------------|
| `QR`      | Input     | 4-bit signed | Multiplier                    |
| `BR`      | Input     | 4-bit signed | Multiplicand                  |
| `Product` | Output    | 8-bit signed | Result of `QR × BR`           |

### Internals

| Signal     | Width   | Purpose                                      |
|------------|---------|----------------------------------------------|
| `AC`       | 4-bit   | Accumulator register                         |
| `QR_temp`  | 4-bit   | Working copy of the multiplier               |
| `Qn1`      | 1-bit   | Previously shifted-out bit (starts at `0`)   |
| `temp`     | 9-bit   | Temporary register for arithmetic right shift|

The loop runs **4 iterations** (once per bit of the multiplier). The final product is the concatenation `{AC, QR_temp}`.

---

## File Structure

```
.
├── booth.v      # Booth multiplier module + testbench
└── README.md
```

---

## Simulation

### Tools Used
- **[EDA Playground](https://www.edaplayground.com/)** — online simulation (Icarus Verilog)
- **Yosys** — synthesis on macOS terminal

### Running with Yosys (macOS)

```bash
# Install via Homebrew if not already installed
brew install yosys

# Synthesize
yosys -p "read_verilog booth.v; synth; show" 
```

### Testbench Cases

| `BR`    | `QR`    | Expected Product |
|---------|---------|-----------------|
| `0111` (+7)  | `1101` (−3)  | −21 |
| `0011` (+3)  | `0010` (+2)  | +6  |
| `0101` (+5)  | `0011` (+3)  | +15 |
| `1101` (−3)  | `0010` (+2)  | −6  |
| `1101` (−3)  | `1110` (−2)  | +6  |
| `0111` (+7)  | `1110` (−2)  | −14 |

---

## Example Output

```
BR = 0111 QR = 1101 || product = 11101011   // 7 × -3 = -21
BR = 0011 QR = 0010 || product = 00000110   // 3 ×  2 =   6
BR = 0101 QR = 0011 || product = 00001111   // 5 ×  3 =  15
BR = 1101 QR = 0010 || product = 11111010   // -3 × 2 =  -6
BR = 1101 QR = 1110 || product = 00000110   // -3 × -2 =  6
BR = 0111 QR = 1110 || product = 11110010   //  7 × -2 = -14
```

---

## Notes

- Both inputs and outputs use **two's complement** signed representation.
- The design is purely **combinational** (`always @(*)`), making it suitable for synthesis into combinational logic.
- The 9-bit `temp` register ensures the arithmetic right shift (`>>>`) correctly propagates the sign bit across the full `{AC, QR, Qn1}` window.
