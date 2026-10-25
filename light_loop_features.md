# LightLoop Project Feature Catalog

An exhaustive list of every feature in the LightLoop project, divided into Hardware and Software with simple and direct descriptions.

---

## 🛠️ Hardware Features

### 1. Compute & Radio Co-Processors
- **Arduino UNO R4 WiFi Main Controller**: 32-bit Renesas RA4M1 ARM Cortex-M4 microcontroller running at 48 MHz with hardware floating-point unit (FPU), 256 KB Flash, and 32 KB SRAM.
- **ESP32-S3 Wireless Co-Processor**: Dual-core 240 MHz radio chip running an AT bridge firmware for Bluetooth Low Energy (BLE 5.0) and 2.4 GHz Wi-Fi (802.11 b/g/n).
- **Hardware Watchdog Timers (WDT & IWDT)**: Internal hardware timers that automatically reboot the system if the main render or communications loop hangs.
- **Non-Volatile Data Flash & EEPROM**: Onboard flash storage preserving mode parameters, system configuration, and offline forecast data across reboots.
- **Retained SRAM Storage**: Uninitialized SRAM section (`.noinit`) that preserves crash telemetry and register states across warm watchdog resets.

### 2. LED Strips & Photonics
- **900 Addressable RGB/RGBW LEDs**: High-density 5V WS2812B / WS2814 pixel strip array (standard 900-LED ring, with legacy support for 420-LED / 60-IC strips).
- **Parallel DMA / High-Speed Assembly Driver**: 4-channel parallel output using sub-microsecond cycle-counted assembly driving digital pins D6, D7, D8, and D9.
- **Multi-Coil Ring Geometry**: Physical mounting supporting 1 single coil (900 LEDs), 2 overlapping coils (450 LEDs each), or 3 coils (300 LEDs each).
- **Dual-Strip Physical Topology**: Support for dual parallel strips wired in mirror, counter-rotating, 90° phase-delayed echo, or harmonic offset configurations.

### 3. Visual Indicators & Display
- **Onboard 12x8 LED Matrix**: Built-in red LED grid on the Arduino board displaying the running firmware version (e.g., "496"), connection status, and diagnostic icons (such as a sad face on unexpected resets).

### 4. Distance & Spatial Sensors
- **HC-SR04 Ultrasonic Distance Sensor**: Native 5V proximity sensor wired to Digital Pin D2 (Trigger) and Digital Pin D3 (Echo) measuring distance up to 1.7 meters.
- **SparkFun VL53L5CX Time-of-Flight (ToF) LiDAR**: 64-zone (8x8 grid) optical distance sensor wired over I2C Wire1 (Pins 26 and 27) providing 2D spatial obstacle tracking and gesture sensing.
- **Dual-Source Proximity Gate**: Microcontroller-level logical OR fusion combining ultrasonic and ToF readings for touchless touchdown detection.

### 5. Timekeeping & Audio
- **DS1307 Real-Time Clock (RTC)**: Dedicated I2C clock module (Pins A4 and A5) with coin-cell battery backup maintaining accurate real-world time while powered off.
- **I2C Bus Auto-Recovery**: Microcontroller pin-toggling routine that clears stuck I2C slave devices holding SDA low.
- **Piezo Buzzer / Speaker**: Audio transducer connected to Digital Pin 10 (PWM channel 5) generating dynamic collision tones, system alerts, and landing fanfares.

### 6. External Hardware Controllers
- **Elgato Stream Deck MK.2 Controller**: 15-key physical desktop controller with full-color LCD screens connected directly via USB HID.

### 7. Enclosure & Mechanical Optics
- **Modular Snap Carrier Chassis**: 3D-printed mounting brackets for the Arduino UNO R4 WiFi and DS1307 RTC module.
- **Weatherproof Desert Enclosure**: 3D-printed protective housing featuring a TPU sealing ring, outer cowl lid, and cable gland bulkheads for dust and water resistance.
- **5-Meter Arc Track Segments**: Modular 3D-printed snap-together curved extrusion tracks (20x and 24x profiles) holding the LED ring securely in place.

---

## 💻 Software Features

### 1. Embedded Microcontroller Firmware (`light_loop_900.ino`)
- **60 FPS Real-Time Render Engine**: Non-blocking animation loop generating 60 frames per second across 900 LEDs.
- **Fast Polynomial Math**: Inlined sine and cosine approximations that bypass heavy math libraries to minimize binary flash size.
- **40 Built-in Animation Modes**: Comprehensive preset library including:
  - *Mode 5*: Solid Color with interactive touch scrubbing and explosive ember burst release.
  - *Mode 8*: Diode Calibration & Coil Alignment with pinpoint boundary targeting.
  - *Mode 28*: Blinker strobe with quadratic green decay.
  - *Mode 29*: Eclipse Corona with solar core, radial symmetry lookup table, and Fourth of July embers.
  - *Mode 30 / 32*: Matrix Grid mapping.
  - *Mode 33*: Smooth Physics Simulation with multi-head turbine and dynamic particle collisions.
  - *Mode 34*: Real-Time Clock face with continuous sweep kinematics and 5 color themes.
  - *Mode 35*: Touchdown Landing deck playing 6 selectable celebration animations.
  - *Mode 36*: LiDAR 64-Zone Spatial Heatmap projection.
  - *Mode 37*: Quantum Flux relativistic plasma turbulence.
  - *Mode 38*: Meteorological Wind Telemetry with live anemometer vectors.
  - *Mode 39*: Tetherball elastic pendulum orbital simulation.
  - *Mode 40*: Kinetic Incline Marbles run.
- **7 Live Modulators**: Real-time parameter controls for Turbine Heads (1–12), Collision Ratio, Phosphor Decay (Fluidity), Chromatic Prism Dispersion, Viscosity / Elasticity, Sparkler Particle Density, and Magnesium White toggle.
- **200-Particle Sparkler Engine**: Real-time ember simulation featuring wake counter-velocity, aerodynamic drag, and chaotic sputtering.
- **Meteorological Wind Engine**: Real-time display of wind speed, gusts, direction, and Beaufort scale with 4 display styles (Windsock Stream, Windsock Vortex, Precision Compass, and Wave).
- **14-Day Offline Wind Forecast Storage**: Microcontroller stores up to 336 hours of weather forecast data in dedicated 1 KB flash memory blocks for autonomous offline playback.
- **Sub-Second Precision Clockwork**: Continuous timekeeping synchronized from the DS1307 RTC with fractional second positioning.
- **Touchless Touchdown Detection**: Automatic trigger of landing animations via ultrasonic proximity (20–200 mm) or ToF LiDAR coverage with hysteresis re-arm latching.
- **Crash Telemetry Preservation**: Automatic recording of fault program counter, return address, and active subsystem to SRAM before watchdog reboot.
- **Hardware Segment Calibration**: In-firmware support for switching between 4-tap (450 LEDs) and 6-tap (300 LEDs) physical segment layouts.

### 2. Frontend Web Application (React, TypeScript, Vite)
- **Adaptive Responsive Posture**: Automatically switches interface layout between Desktop Adaptive Studio, Mobile Phone view, and Google Pixel Fold dual-screen foldable posture.
- **Interactive 60 FPS Canvas Ring Simulator**: In-browser virtual preview rendering 900 physical LEDs, coil layering, and particle physics in real time.
- **5-Segment Kinetic Scrubber**: Multi-tier scrubbing bar allowing micro, medium, and macro frame positioning across 360 degrees.
- **Upward-Yielding Panel Expansion**: UI panels that detect the bottom of the screen and expand upward within 80ms, preventing controls from clipping off-screen.
- **Live Photonics HUD**: Real-time telemetry indicators showing active mode, frame position, speed, and communication latency.
- **Scrub Sensitivity Visualizer**: Dynamic visual curve showing touch and mouse scrub sensitivity across the scrubber track.
- **Triple Communication Interfaces**:
  - *Web Bluetooth (BLE)*: Browser-based wireless link using Nordic UART 16-byte command packets (`CMD`, `MOD`, `WND`, `TAP`, `SAV`) with automatic reconnect.
  - *Web Serial (USB)*: High-speed 500,000 baud serial stream for real-time control and debugging.
  - *Wi-Fi REST API*: Networked HTTP control endpoint (`/api/control`) secured by password authentication.
- **4-Phase BLE Post-Connect Settling Sequence**: Phased transmission delay (550ms, 900ms, 1100ms, 1400ms) ensuring zero dropped packets on newly established Bluetooth links.
- **Decoupled Tri-Tier Versioning System**: Independent version tracking for Web App (`webVersion`), Target Firmware (`targetFirmwareVersion`), and Backward Compatibility (`minSupportedFirmwareMinor`) with an emerald reassurance badge.
- **User Profile & Preset Manager (`UserProfileManager`)**: Browser storage saving custom parameter presets, themes, and calibration configurations.
- **Minimal Blocking Status Overlay**: Modal overlay that activates strictly during active asset compilation or Arduino flashing, leaving the app interactive during normal use.

### 3. Stream Deck Direct USB HID Daemon (macOS Service)
- **Direct USB HID Controller (`controller.py`)**: Communicates directly with Stream Deck hardware via `hidapi`, bypassing the vendor app to eliminate border clipping and downscaling.
- **24/7 Persistent macOS LaunchAgent**: System daemon running under `com.antigravity.streamdeck.plist` with automatic recovery and single-instance PID locking.
- **220ms Elastomer Debounce**: Schmitt-trigger filter eliminating contact bounce on physical silicone keys.
- **Decoupled KeyDown / KeyUp Actions**: Instant negative-inversion visual feedback on key press, followed by non-blocking asynchronous command execution on key release.
- **Continuous 2D Matrix Wave Animations**: Full-panel shockwave and ripple effects spanning all 15 LCD buttons, rendered at 40 FPS using precomputed alpha masks.
- **Borderless Key Typography**: High-contrast, full-bleed icon graphics without artificial borders or padding.

### 4. Android Companion App (Kotlin)
- **Native Android Companion**: Mobile client application optimized specifically for Google Pixel Fold devices.
- **Extended BLE Coded PHY Support**: Long-range Bluetooth communication taking full advantage of modern mobile radio hardware.
- **ADB Reverse Tunneling & Broadcast Control**: Command interface allowing headless automated control and testing over USB ADB.

### 5. Auditory System Integration
- **Spoken Technical Debriefs & Briefings**: Integration with local auditory server (port 8005) delivering natural voice announcements and progress updates via Google Cloud TTS.
- **Hardware Lease Mutex Coordinator**: Software lock preventing simultaneous access to the serial port or radio during flashing operations.

### 6. Development, Diagnostic & Flashing Tooling
- **Autonomous CLI Upload Script**: Single-command flashing utilizing 1200-bps DFU touch reset to upload firmware without touching physical board buttons.
- **Automated Serial Teardown (`.kill_serial`)**: Coordination system that gracefully closes active browser serial ports before flashing firmware.
- **Dedicated Diagnostic Tools**:
  - `scripts/read-serial.py`: Safe, non-rebooting serial telemetry reader.
  - `scripts/scan-ble.py`: Real-time interactive Bluetooth Low Energy advertisement scanner.
  - `scripts/test_ble.py`: Automated GATT handshake, packet verification, and stress fuzzing suite.
- **ESP32-S3 Bridge Flasher**: Scripts to compile, upload, or restore custom Coded PHY AT bridge firmware to the radio co-processor.
