# Full-Brilliance Zero-Clamp Power Protection & Timing Precision Invariant

## ⚡ Full-Brilliance Zero-Clamp Power Protection Invariant (Permanent Hardware Rule)
- **Zero Brightness Clamping**:
  - When resolving high-current LED transients, power rail collapse, or microcontroller brownout resets (e.g. on the 900-LED WS2812B loop), agents must **NEVER** clamp, cap, truncate, or reduce maximum pixel brightness (`wVal = 255`) or visual dynamic range.
- **Mandatory Temporal Inrush Slew ($\frac{di}{dt}$ Mitigation)**:
  - Instead of switching high-density LEDs from 0 to 255 in a single instantaneous frame latch (which creates an explosive 15–20A current spike that collapses the 5V power rail), slew the onset continuously over time ($\sim 45\text{ms}$ rise time, spanning 2–3 frames at ~65 FPS).
  - Continuous formulation:
    $$\text{attack}(t) = \min\left(1.0,\, \frac{t}{0.045}\right)$$
    $$wVal(t) = \left\lfloor 255.0 \times \text{fade}(t) \times \text{attack}(t) \right\rfloor$$
  - This eliminates the infinite $\frac{di}{dt}$ step change that induces supply rail collapse while delivering **100% full peak white brilliance (`255, 255, 255`)**.
- **Hardware Power Isolation Alternatives**:
  - **Schottky Diode + Local Capacitor Reservoir**: Insert a low-forward-drop Schottky diode (e.g. SS34, 1N5819) on the Arduino 5V input feed with a 470µF–1000µF low-ESR capacitor on the Arduino side. When the LED rail sags, the diode reverse-biases and the capacitor holds $V_{DD}$ steady above the LVD threshold.
  - **Dedicated Dual-Feed Power**: Power the microcontroller independently via USB-C or the VIN barrel jack (7–12V into the onboard buck regulator) while running LEDs from their dedicated 5V power supply, sharing a clean common ground.
  - **Data Line Protection**: Place a 330Ω–470Ω series resistor or 74AHCT125 buffer on the NeoPixel data line (`DIN`) to eliminate ground bounce latch-up across internal ESD protection diodes.

## ⏱️ Deterministic Mathematical Precision in Timing & Cadence Descriptions (Permanent Invariant)
- **Zero Vague Frame Ranges**:
  - Never describe timing, slews, or debounces with vague ranges (e.g. saying *"2–3 frames"*) without explicit mathematical definition.
- **Mandatory Three-Part Cadence Breakdown**:
  1. **Continuous Time Parameter**: Explicitly state the exact continuous duration ($t_{\text{rise}} = 45.0\text{ ms} = 0.045\text{ s}$).
  2. **Mathematical Formula**: Provide the governing equation (e.g. $\text{attack}(t) = \min(1.0,\, t / 0.045)$).
  3. **Operational Cadence Breakdown**: Provide the exact discrete integer values across operational frame rates:
     - **At 60.0 FPS ($16.67\text{ms}$ frame period)**: Exactly **2 intermediate rising frames** (Frame 1 at 94, Frame 2 at 189) before reaching full 255 on Frame 3 ($50\text{ms}$).
     - **At 71.4 FPS ($14.00\text{ms}$ frame period)**: Exactly **3 intermediate rising frames** (Frame 1 at 79, Frame 2 at 158, Frame 3 at 238) before reaching full 255 on Frame 4 ($56\text{ms}$).
