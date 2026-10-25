# 4-Line Segment Split WS2812 Parallel Driver Invariants

## 1. Physical Pin & Renesas RA4M1 Port Register Mapping
To drive two 900-LED WS2812 strands without exceeding 15ms frame latencies, each ring is divided into two 450-LED hardware segments driven in parallel across 4 GPIO pins:
- **Ring 1 (Strip 1)**:
  - **LEDs $0 \dots 449$**: Pin D6 $\rightarrow$ `BSP_IO_PORT_01_PIN_11` (`P111` / `R_PORT1` Bit 11, Mask `1U << 11`)
  - **LEDs $450 \dots 899$**: Pin D9 $\rightarrow$ `BSP_IO_PORT_03_PIN_03` (`P303` / `R_PORT3` Bit 3, Mask `1U << 3`)
- **Ring 2 (Strip 2)**:
  - **LEDs $0 \dots 449$**: Pin D7 $\rightarrow$ `BSP_IO_PORT_01_PIN_12` (`P112` / `R_PORT1` Bit 12, Mask `1U << 12`)
  - **LEDs $450 \dots 899$**: Pin D10 $\rightarrow$ `BSP_IO_PORT_01_PIN_03` (`P103` / `R_PORT1` Bit 3, Mask `1U << 3`)
- **Buzzer / Tone Generator**: Dedicated to **Pin D8** $\rightarrow$ `BSP_IO_PORT_03_PIN_04` (`P304` / `R_PORT3` Bit 4).

## 2. Pre-Calculated Sub-Microsecond WS2812 Bit Timing (Mandatory Rule)
At the 48MHz CPU clock of the Renesas RA4M1:
- $T_{0H} = 0.25\text{µs}$ (12 CPU clock cycles)
- $T_{1H} = 0.80\text{µs}$ (38 CPU clock cycles)
- $T_{\text{period}} = 1.25\text{µs}$ (60 CPU clock cycles)

**CRITICAL INVARIANT**:
Branch calculations for which bits to clear at $T_{0H}$ and $T_{1H}$ (`clr1_t0`, `clr1_t1`, `clr3_t0`, `clr3_t1`) **must always be pre-calculated before** asserting the timer counter (`ARM_DWT_CYCCNT`).
Calculating bit conditions after driving pins HIGH consumes 15–20 CPU cycles, blowing past the strict 12-cycle $T_{0H}$ threshold and causing the WS2812 LEDs to interpret $0$ bits as $1$ bits.

```cpp
// Pre-calculate clear masks BEFORE cycle start
uint16_t clr1_t0 = 0, clr1_t1 = 0;
if (!bit6)  clr1_t0 |= PIN6_MASK;  else clr1_t1 |= PIN6_MASK;
if (!bit7)  clr1_t0 |= PIN7_MASK;  else clr1_t1 |= PIN7_MASK;
if (!bit10) clr1_t0 |= PIN10_MASK; else clr1_t1 |= PIN10_MASK;

const uint16_t clr3_t0 = (!bit9) ? PIN9_MASK : 0;
const uint16_t clr3_t1 = (bit9)  ? PIN9_MASK : 0;

while (ARM_DWT_CYCCNT - cyc < CYCLES_800);
cyc = ARM_DWT_CYCCNT;
*set_reg1 = PORT1_MASK;
*set_reg3 = PORT3_MASK;

while (ARM_DWT_CYCCNT - cyc < CYCLES_800_T0H);
if (clr1_t0) *clr_reg1 = clr1_t0;
if (clr3_t0) *clr_reg3 = clr3_t0;

while (ARM_DWT_CYCCNT - cyc < CYCLES_800_T1H);
if (clr1_t1) *clr_reg1 = clr1_t1;
if (clr3_t1) *clr_reg3 = clr3_t1;
```

## 3. Parallel Transmission Performance
- Frame transmission duration: $450 \text{ LEDs} \times 30\text{µs} = \mathbf{13.5\text{ms}}$ (down from $27\text{ms}$).
- Maximum hardware refresh rate: $\approx \mathbf{74\text{ FPS}}$.
- 2µs interrupt window between LEDs ensures the UART FIFO and LED matrix ISR remain responsive.
