# counter-ASIC--FPGA-Comparison

This report documents the complete RTL-to-GDSII flow for a 4-bit synchronous counter using the open-source OpenLane ASIC toolchain and the SKY130 PDK. It further provides a detailed comparative analysis between the results of synthesizing the same RTL design for an FPGA target (using Xilinx Vivado) and for an ASIC target. 

The primary objective was to transform a behavioral Verilog description into a GDSII file, the final standard format for semiconductor manufacturing. This process involves synthesis, floor planning, placement, clock tree synthesis, routing, and sign-off verification.

### Design: my_counter.v
```verilog
// A simple 4-bit synchronous counter
module my_counter (
    input  wire clk,
    input  wire rst_n,
    output reg [3:0] count
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            count <= 4'b0;
        else
            count <= count + 1;
    end
endmodule
```

### Configuration: config.json
```json
{
    "PDK": "sky130A",
    "DESIGN_NAME": "my_counter",
    "VERILOG_FILES": "dir::src/my_counter.v",
    "CLOCK_PORT": "clk",
    "CLOCK_PERIOD": 10,
    "FP_CORE_UTIL": 20,
    "FP_ASPECT_RATIO": 1,
    "DIE_AREA": "0 0 200 200"
}
```

### ASIC Implementation Flow with OpenLane
The flow was executed interactively within the OpenLane Docker environment:

1. **Environment Setup**
   - Command: `make mount` - Launches Docker container with EDA tools
   - Command: `./flow.tcl -interactive` - Starts OpenLane shell

2. **Step-by-Step Execution**
   - Preparation: `prep -design my_counter`
   - Synthesis: `run_synthesis` - Converts RTL to gate-level netlist
   - Floorplanning: `run_floorplan` - Defines chip core area and I/O placement
   - Placement: `run_placement` - Places standard cells optimally
   - Clock Tree Synthesis: `run_cts` - Builds clock distribution network
   - Routing: `run_routing` - Connects cells with metal wires
   - Signoff: `run_magic` (DRC) and `run_lvs` (LVS) - Final verification

3. **Final Output**
   - File: `counter.gds` - Final manufacturable blueprint

### Timing Analysis Fundamentals
Key timing parameters verified by Static Timing Analysis (STA):
- **Clock Period:** Time between active clock edges (10 ns for 100 MHz)
- **Setup Time (T_setup):** Data must be stable before clock edge
- **Hold Time (T_hold):** Data must be stable after clock edge
- **Slack:** Difference between required and actual signal arrival time
- **WNS (Worst Negative Slack):** Worst setup slack - key performance metric
- **WHS (Worst Hold Slack):** Worst hold slack - key correctness metric

### Comparative Analysis: Vivado (FPGA) vs. OpenLane (ASIC)

**Vivado Synthesis Results (FPGA Flow)**
- Target: Xilinx Artix-7 FPGA (xc7a35tcpg236-1)
- Resource Utilization:
  - Slice LUTs: 3 (<0.01%)
  - Slice Registers: 4 (<0.01%)
  - Bonded IOBs: 6
- Timing Analysis:
  - WNS: +8.303 ns
  - WHS: +0.142 ns

**OpenLane Synthesis Results (ASIC Flow)**
- Target: SKY130A HD Standard Cell Library
- Area Analysis:
  - Total Cell Area: 163.91 μm²
  - Cell Count: 10 cells
- Timing Analysis (Pre-Layout):
  - WNS: +7.17 ns
  - WHS: +0.32 ns

**Key Comparative Insights**
| Aspect | Vivado (FPGA) | OpenLane (ASIC) |
|--------|---------------|-----------------|
| Target | Pre-fabricated FPGA | Custom silicon chip |
| Cost Metric | Utilization (%) | Physical Area (μm²) |
| Logic Implementation | Generic LUTs | Fine-grained standard cells |
| Performance (WNS) | +8.303 ns | +7.17 ns |
| Design Focus | Resource mapping | PPA optimization |

### Inference from Results
Both synthesis flows successfully implemented the counter design and met the 100 MHz timing target with significant positive slack. The FPGA implementation showed slightly better timing performance (+8.303 ns WNS) using generic LUT-based architecture, while the ASIC implementation achieved efficient area usage (163.91 μm²) through optimized standard cell selection. The results demonstrate that while both approaches are valid, the choice between FPGA and ASIC depends on specific design priorities of performance versus area optimization.
