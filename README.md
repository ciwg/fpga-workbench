# FPGA Development Codespace

Codespace for FPGA development targeting the Lattice ICE40UP5K chip on the [IceSugar board](https://github.com/wuxx/icesugar).

## Quick Start

1. Create a codespace from this repo
2. Wait for setup to complete (~5-10 min)
3. Try the blinky example:

```bash
cd icesugar-blinky
make          # synthesize + place & route + pack bitstream
make sim      # simulate with iverilog, produces blinky_tb.vcd
```

## What's Included

- **oss-cad-suite** — Yosys, nextpnr-ice40, icepack, iceprog, Icarus Verilog, GTKWave
- **Verilog HDL** VS Code extension
- **Blinky hello world** — complete example with testbench and Makefile

## Limitations

- No USB access from Codespaces — you can synthesize in the cloud but must flash locally
- GTKWave requires a display (works locally, not in browser-based Codespaces)
- ICE40 only — other FPGA families would need additional tooling

See **[GUIDE.md](GUIDE.md)** for a full walkthrough of the toolchain, pin reference, project ideas, flashing workarounds, and recommended workflow.

## Tool Versions

To update oss-cad-suite, edit `OSS_CAD_DATE` in `.devcontainer/setup.sh`.
Releases: https://github.com/YosysHQ/oss-cad-suite-build/releases


## Local Workflow

### MacOS

This is procedural is written for running local instance of dev container on MacOS.

The specific local setup used to write & test this procedure: 
- Macbook Pro 2019, macOS Sequoia
- Colima https://colima.run/ & Docker CLI
- VSCode

**Starting container**

1. Open a terminal, run the following commands:
    - `colima start`
    - `docker ps`
2. Open the repo in VSCode
3. "Reopen in Container"

Initial build will take up to 20 minutes (assuming good wifi download speeds).

**Closing container**

1. Close VSCode
2. Run  `colima stop`