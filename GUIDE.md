# FPGA Codespace Guide

A walkthrough of what this codespace gives you, what it doesn't, and how to get the most out of it.

---

## What's Inside

### Synthesis Toolchain (oss-cad-suite)

The codespace installs the full open-source FPGA toolchain from [YosysHQ](https://github.com/YosysHQ/oss-cad-suite-build):

| Tool | What It Does |
|------|-------------|
| **Yosys** | Verilog synthesis — turns your `.v` files into a gate-level netlist |
| **nextpnr-ice40** | Place and route — maps the netlist onto the physical ICE40 chip |
| **icepack** | Packs the routed design into a binary bitstream (`.bin`) |
| **iceprog** | Programs the bitstream onto the FPGA via USB (local use only) |

These are the same tools used by professional open-source FPGA developers. They are **completely free and open source**.

### Simulation Tools

| Tool | What It Does |
|------|-------------|
| **Icarus Verilog** (`iverilog`) | Compiles and simulates Verilog testbenches |
| **VVP** | Executes the compiled simulation, produces waveform dumps |
| **GTKWave** | Waveform viewer for `.vcd` files (needs a display — works locally, not in browser Codespaces) |

All included in oss-cad-suite. **Free and open source.**

### VS Code Extensions

- **Verilog HDL** (`mshr-h.veriloghdl`) — syntax highlighting, linting, and basic navigation for `.v` files

### Reference Code

- **OpenCores I2C Slave Controller** — cloned to `/workspaces/i2cslave` as a reference implementation for I2C peripheral design

---

## Example Project: Blinky

The `icesugar-blinky/` directory contains a complete hello-world project:

- **`blinky.v`** — A 26-bit free-running counter driven by the 12 MHz clock. Bits 23, 24, and 25 drive the red, green, and blue channels of the on-board RGB LED. Since the LED is active-low, each channel inverts its counter bit. The result is the LED cycling through all 8 color combinations.
- **`blinky_tb.v`** — A testbench that simulates the design, prints LED state changes to the console, and dumps a `.vcd` waveform file.
- **`icesugar.pcf`** — Pin constraint file mapping signal names to physical pins on the IceSugar board (sourced from the [official repo](https://github.com/wuxx/icesugar)).
- **`Makefile`** — Build automation with targets for synthesis, simulation, timing analysis, and programming.

### Build flow

```
blinky.v ──► yosys ──► blinky.json ──► nextpnr-ice40 ──► blinky.asc ──► icepack ──► blinky.bin
 (Verilog)   (synth)    (netlist)       (place+route)     (routed)       (pack)      (bitstream)
```

### Commands

```bash
cd icesugar-blinky
make            # full build: synthesis → place & route → bitstream
make sim        # simulate with iverilog, writes blinky_tb.vcd
make timing     # timing analysis report via icetime
make prog       # copy bitstream to iCELink USB drive (local only)
make clean      # remove all generated files
```

---

## Shortcomings and Limitations

### No USB Access

This is the biggest limitation. GitHub Codespaces runs on remote cloud VMs with **no access to your local USB devices**. That means:

- `iceprog` and `make prog` will not work from a Codespace
- You cannot flash, debug, or interact with physical hardware from the cloud
- There is no USB forwarding, no WebUSB workaround, and no VS Code Remote USB passthrough

**Workarounds:**

1. **Build in Codespace, flash locally.** Download the `.bin` file from the Codespace and copy it to the iCELink USB mass-storage drive on your local machine.
2. **Run the devcontainer locally.** Use VS Code Dev Containers with Docker on your own machine. The same `.devcontainer/` config works locally and gives you full USB access to your board.

### GTKWave Needs a Display

GTKWave is a graphical application. It works when running the devcontainer locally, but **not in a browser-based Codespace** (no X11/Wayland display). You can still generate `.vcd` files in the Codespace and download them to view locally.

### No Hardware-in-the-Loop Testing

Since there is no USB access, you cannot do hardware-in-the-loop testing or interact with the running FPGA from the Codespace. All testing is limited to simulation.

### Single Target Architecture

The toolchain is configured for **ICE40 only** (specifically the UP5K). If you want to target other FPGA families (ECP5, Gowin, Xilinx, Altera), you would need to install additional tools and modify the build flow.

### Setup Time

The initial Codespace build takes **5-10 minutes**, mostly spent downloading and extracting the ~1.3 GB oss-cad-suite archive. Subsequent starts are fast thanks to Codespace caching.

### No Formal Verification

The toolchain does not include formal verification tools like [sby (SymbiYosys)](https://github.com/YosysHQ/sby). Formal verification can prove that your design satisfies certain properties exhaustively, rather than relying on simulation testbenches. sby is open source and could be added to the setup if needed.

---

## Target Hardware

| Spec | Value |
|------|-------|
| **Chip** | Lattice ICE40UP5K-SG48 |
| **Logic Elements** | 5,280 (blinky uses ~30) |
| **Block RAM** | 120 Kbit (15 x 8 Kbit) |
| **DSP blocks** | 8 (multiply-accumulate) |
| **Board** | [IceSugar](https://github.com/wuxx/icesugar) |
| **Programmer** | iCELink (shows up as USB mass-storage — drag and drop) |
| **Clock** | 12 MHz on-board oscillator |
| **I/O** | RGB LED, 4 DIP switches, UART over USB, PMOD headers |

---

## IceSugar Pin Map (Quick Reference)

| Signal | Pin | Notes |
|--------|-----|-------|
| `clk` | 35 | 12 MHz oscillator |
| `LED_R` | 40 | Red LED, active low |
| `LED_G` | 41 | Green LED, active low |
| `LED_B` | 39 | Blue LED, active low |
| `SW[0]` | 18 | DIP switch |
| `SW[1]` | 19 | DIP switch |
| `SW[2]` | 20 | DIP switch |
| `SW[3]` | 21 | DIP switch |
| `RX` | 4 | UART receive (via iCELink) |
| `TX` | 6 | UART transmit (via iCELink) |

Full pinout: [icesugar/src/common/io.pcf](https://github.com/wuxx/icesugar/blob/master/src/common/io.pcf)

---

## Project Ideas (After Blinky)

| Project | What You Learn | Extra Hardware? |
|---------|---------------|-----------------|
| **PWM LED fader** | Comparators, duty cycle, smooth analog-like control | No |
| **Button debouncer** | Metastability, synchronizers, shift-register filtering | No (uses DIP switches) |
| **UART echo** | Serial protocol, baud rate generation, shift registers | No (uses built-in USB-UART) |
| **I2C slave** | State machines, bus protocols, clock stretching | Needs an I2C master (another board or Raspberry Pi) |
| **Quadrature decoder** | Edge detection, direction decoding, counters | Needs a rotary encoder |
| **SPI flash reader** | SPI protocol, the ICE40's on-board flash | No (ICE40UP5K has built-in SPI flash) |

---

## Recommended Workflow

```
┌─────────────────────────────────────────────┐
│              GitHub Codespace               │
│                                             │
│   Write Verilog ──► Simulate ──► Synthesize │
│                                             │
│   Everything up to bitstream generation     │
│   works in the cloud.                       │
└──────────────────┬──────────────────────────┘
                   │ download .bin
                   ▼
┌─────────────────────────────────────────────┐
│              Local Machine                  │
│                                             │
│   Flash board via iCELink USB drive         │
│   (just copy the .bin file)                 │
│                                             │
│   Or: run the devcontainer locally with     │
│   Docker for the full flow in one place.    │
└─────────────────────────────────────────────┘
```

---

---

## What Works and What Doesn't in the Codespace

This Codespace is set up for FPGA development targeting the iCE40UP5K. The target project is the [FPGA Encoder](https://github.com/ciwg/fpga-encoder) — an FPGA-based quadrature encoder counter with an I2C interface. Here's what you can and can't do from the browser.

### What Works

**Writing and editing Verilog.** The Verilog HDL extension provides syntax highlighting, linting, and navigation for `.v` files. Everything you need to write RTL.

**Simulation.** Icarus Verilog (`iverilog`) and VVP are installed. You can compile and run testbenches, generate `.vcd` waveform files, and iterate on your design — all from the terminal.

**Viewing waveforms.** VaporView and Surfer are both installed as VS Code extensions. Click any `.vcd` file and it opens in a tab — no GTKWave, no display server, works fully in the browser. The blinky example generates a `.vcd` you can try right away: `cd icesugar-blinky && make sim`, then open `blinky_tb.vcd`.

**Synthesis and bitstream generation.** Yosys, nextpnr-ice40, and icepack are all available. You can synthesize your design, run place-and-route, and generate a `.bin` bitstream file. The synthesis output also reports resource usage so you can confirm your design fits on the iCE40UP5K.

**Python testbenches with cocotb.** cocotb 2.0.1 and cocotb-bus 0.3.0 are installed. You can write testbenches in Python instead of Verilog, which is especially useful for testing bus protocols like I2C.

### What Doesn't Work

**Flashing the FPGA.** Codespaces run on remote VMs with no USB access. You cannot program the board from the browser. Download the `.bin` file and copy it to the iCELink USB drive on your local machine, or run the devcontainer locally with Docker for the full flow in one place.

**GTKWave.** It's a GUI application that needs a display server. Use VaporView or Surfer instead — they work in the browser and are already installed.

**Hardware-in-the-loop testing.** No physical encoder signals, no I2C probing, no oscilloscope. All testing is simulation-only until you move to local hardware.

**Real-time debugging.** No logic analyzer, no JTAG. Simulation and synthesis reports are your debugging tools in the Codespace.

### What to Do When You Need Hardware

Build in the Codespace, flash locally. The workflow is:

1. Write and simulate in the Codespace until you're confident
2. Run `make` to generate the `.bin` bitstream
3. Download the `.bin` file
4. Copy it to the iCELink USB mass-storage drive on your local machine

Alternatively, run the same devcontainer locally with VS Code + Docker. The `.devcontainer/` config works identically on your own machine and gives you full USB access.


---

## References

- [oss-cad-suite](https://github.com/YosysHQ/oss-cad-suite-build) — the toolchain
- [IceSugar board](https://github.com/wuxx/icesugar) — board repo with pinouts, schematics, examples
- [Yosys documentation](https://yosyshq.readthedocs.io/) — synthesis tool docs
- [nextpnr](https://github.com/YosysHQ/nextpnr) — place and route
- [Project IceStorm](https://github.com/YosysHQ/icestorm) — ICE40 reverse engineering / bitstream tools
- [Icarus Verilog](https://steveicarus.github.io/iverilog/) — simulator
- [fpga4fun](https://www.fpga4fun.com/) — beginner FPGA tutorials
- [nandland](https://nandland.com/) — FPGA fundamentals and example projects
