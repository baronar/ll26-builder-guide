# Light Loop 5m Ring Light — 3D Printable Interlocking Arc Braces (Enhanced Snap-Fit)

High-precision interlocking curved brace segments designed to hold circular LED ring lights in a true geometric circle, with dual light strips seated vertically on their sides (back-to-back).

---

## 🔧 Enhanced Snap-Fit Interlocking Joint Design

The joint between arc segments has been engineered with:

1. **Stepped Lap / Halving Shelf**:
   * A 15 mm overlapping tongue-and-hood design that eliminates vertical sagging, twisting, or bending across segments.
2. **Dual Cantilever Locking Latches**:
   * Spring-action cantilever arms on the inner and outer walls flex horizontally into place during assembly and drop into mating window pockets with a tactile "click".
3. **Pinch-to-Release Disassembly**:
   * Side window cutouts allow you to pinch the latch tabs with your thumb and forefinger to deflect the spring arms inward, letting the arcs slide apart smoothly without tools or damage.
4. **Seamless Strip Pass-Through**:
   * The 5.6 mm center channel carries continuously through the joint, clamping both vertical LED strips back-to-back across the seam.
5. **Horizontal FDM Print-Bed Strength**:
   * Because the arc lays flat on the bed, all cantilever snap arms flex along continuous extruded XY filament tracks, providing maximum fatigue strength and zero layer delamination.

---

## 📐 Geometry & Specifications

| Parameter | 24-Segment (Recommended) | 20-Segment Option |
| :--- | :--- | :--- |
| **Total Circumference** | 5000 mm (5.0 meters) | 5000 mm (5.0 meters) |
| **Exact Centerline Radius** | **795.775 mm** (~795.8 mm) | **795.775 mm** (~795.8 mm) |
| **Arc Angle per Segment** | **15.0°** | **18.0°** |
| **Arc Length (Centerline)** | **208.33 mm** | **250.00 mm** |
| **Straight Chord Length** | **207.74 mm** (fits on all 220–256mm beds) | **248.97 mm** (for 250mm+ beds) |
| **Overall Radial Width** | 12.0 mm | 12.0 mm |
| **Overall Vertical Height** | 12.0 mm | 12.0 mm |
| **Base Floor Thickness** | 2.4 mm | 2.4 mm |
| **LED Strip Slot Width** | **5.6 mm** (fits 2x WS2812B/COB back-to-back on edge) | **5.6 mm** |
| **Retention Barbs** | 0.55 mm overhang with 45° lead-in chamfer | 0.55 mm overhang |
| **Lap Joint Length** | 15.0 mm | 15.0 mm |
| **Fit Clearance** | 0.25 mm (calibrated for standard 0.4mm FDM nozzles) | 0.25 mm |

---

## 📂 Included Files in `/3d_models/`

* **`light_loop_5m_arc_24x_snap.stl`**: Ready-to-print binary STL with enhanced snap joint (208 mm length).
* **`light_loop_5m_arc_20x_snap.stl`**: 20-piece segment variant (250 mm length).
* **`Fusion360_Scripts/LightLoopArcGenerator/`**: Fusion 360 script package with `.py` and `.manifest`.
* **`LightLoop_5m_Arc_Profile.dxf`**: 2D cross-section DXF for instant revolve in Fusion 360.
* **`light_loop_5m_arc_24x_snap.scad`**: OpenSCAD parametric source code.

---

## 🖨️ Recommended 3D Print Settings

* **Orientation**: Lay flat on the print bed (open U-channel facing upward).
* **Material**: **PETG** (Highly recommended for flexible, fatigue-resistant snap clips) or **PLA+**.
* **Layer Height**: `0.20 mm` (or `0.16 mm` for crisp snap detents).
* **Wall Loops / Perimeters**: `3` or `4` walls.
* **Infill**: `25%` Gyroid.
* **Supports**: **None needed** (all overhangs and snap wedges are angled $\le 45^\circ$).
