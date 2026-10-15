# titanRIO — Project Context

Legend: ★ = put in project memory (durable decision/fact). ☆ = put in project files/knowledge, not memory (bulky or changes often). ? = open question, track but don't treat as settled.

## What it is
- ★ roboRIO-style expansion module (daughterboard) for robotics: MCU + FPGA, I2C to host, safety chain, CAN, encoders, PWM. Goal: roboRIO-like experience, general-purpose, not tied to a custom carrier.
- ★ Originally a Jetson Orin Nano carrier idea; carrier is now "fantasy/dream." The daughterboard is the real project.
- ★ Name origin: "TitanRIO" came from another LLM's design (Orin Nano + RP2350 + Tang Nano 20K).
- ★ First PCB design, KiCad, expected 2-layer. Budget: college student prototyping.
- ★ Status: KiCad project "titanRIO rev 0.1" (sheets: root, Peripherals, Pin IO), work in progress.

## Architecture decisions
- ★ Host to MCU: I2C preferred (USB rejected as complicated; MCUs should be able to host the module). Current layout in KiCad: host-to-MCU over I2C, MCU-to-FPGA over SPI.
- ★ MCU: RP2350 (RP2350B in production); RP pin budget is tight. V1 prototype uses Raspberry Pi Pico 1 on female headers.
- ★ FPGA: fixed part of the module, Tang Nano 20K (Gowin IDE) in v1; pins direct-wired through level shifters, some pins wired to MCU. Prototype plugs in via headers; production board would include its own flash. JTAG for FPGA flashing.
- ★ CAN: 1x MCP2518FD controllers (not CAN in Verilog), isolated CAN transceiver (ISO1044) in v1. SocketCAN integration is a maybe-later.
- ★ IMU: BNO08x, I2C-only, 4-wire (like the Adafruit module). Needs interrupt pin handling on MCU.
- ★ I2C: single bus, MCU only; (?) PCA9685 PWM shares it. FPGA sensor bank dropped as redundant.
- ★ Level shifting: proper level shifters; per-bank JST-GH connectors with extra ground and Vref pins; DIP switch picks 5V vs bank Vref. Safety bank possibly unshifted and not gated by ACTIVE.
- ★ Connectors: JST-GH. Exposed MCU interfaces: SPI, CAN, 1x UART, 1x I2C. Analog inputs on a separate header.
- ★ Banks (sketch): FPGA DIO, RP DIO, RP AIN/FPGA in, FPGA undecided, safety/general. Subject to change as project develops.
- ★ Encoders: up to 4 quadrature (8 pins) on the FPGA.
- ★ Power: XL1509 (1x 5v, 1x 3.3v). INA219 in-line from battery connector (screw terminal) to power supplies.

## Safety
- ★ ACTIVE = internally controlled output enable; stays low until checks pass.
- ★ Safety chain monitored by 32.768kHz clock sent down to 74hc123 (1st multivibrator). If pulses miss, monostable multivibrator goes low.
- ★ WDT is controlled by host. If no pulse sent in 200ms, 74hc123 (2nd multivibrator) goes low.
- ★ Safety chain AND WDT AND RP Ready signal -> D Latch Clear. D pin is tied to 3v3, Clock is tied to reset button (and host-controlled pin, solder bridge in route). D Latch is sn74lvc1g175.
- ★ Override switches/jumpers on safety loops and watchdog (visible jumpers) for testing and simple robots. (?) AUTO_ARM so the MCU can run code without a host.
- ★ Safety is its own credible bus, e-stop included.

## UI / status
- ★ LEDs: red = power, green = active, blue = SCL (for comms). Might add more or change colors. Single NeoPixel (WS2812B) for user control.

## Parts already owned
- ☆ Tang Nano 20K, Raspberry Pi Pico.

## Open problems / working style
- ? Pico pin count (when working with RP2040); BNO08x interrupt pin and awkward I2C protocol.
- ? Module needs its own control software (users can't freely control both MCU and FPGA). Possible JetPack driver/library for the carrier case, EEPROM discovery (you're lukewarm on it). Goal is WPILib-style library.
- ? Power section still a to-do; floorplan deferred until parts list, pinouts, diagrams are done.
- ? Auto-directional level shifter suitability (TXB-family) for bidirectional signals like I2C/open-drain.
- ★ Dropped ideas (don't resurface): Kiwi KVM, extra PCIe lane, separate FPGA sensor bank, OVR_RD, FPGA_RST, BNO086 INT/RST connector pins.
- ★ Docs (guidance, inspiration, documentation) come after the architecture is validated; wants a use-case/goals doc, pin description file, and layout idea.
- ★ Preference for this project: short, blunt, practical answers; keep option lists short.

## Files to add to the project (☆)
- Use-case/goals doc
