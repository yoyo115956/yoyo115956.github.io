Hey, I'm **Li Yongting** — an IC Verification Engineer at [Hygon Information Technology](https://www.hygon.cn/).

I studied Software Engineering (B.Eng.) at Nanjing University with a focus on front-end development, then pivoted into chip verification during my master's at Tsinghua University through industry-collaboration projects. I have **1+ year** of hands-on experience in digital chip verification and design, and am proficient in **SystemVerilog and UVM methodology**.

At **ZhongKe ShenLong Technology**, I contributed to the verification of the Z4 single-core processor and a 32-bit floating-point IP core — discovering and resolving **30+ critical bugs** with functional coverage consistently above **95%** (98%+ for the FP IP). Earlier, as an IC Design intern, I designed a Mesh-topology NoC for ZKP cryptography, achieving a **21.8% throughput improvement** on the 8×8 benchmark.

Feel free to reach out or explore my work on [GitHub](https://github.com/yoyo115956).

---

##### Work Experience

| Period | Company | Role |
|--------|---------|------|
| 2025.07 — Present | **Hygon Information Technology** (Chengdu) | IC Verification Engineer |
| 2024.12 — 2025.06 | **ZhongKe ShenLong Technology** (Beijing) | IC Verification Intern |
| 2024.03 — 2024.07 | **ZhongKe ShenLong Technology** (Beijing) | IC Design Intern |

##### Featured Projects

**Z4 Single-Core Processor Verification**
- Built partial UVM environment (TCM Scoreboard); wrote SCALAR tests cross-validated against GEM5
- Verified IOBUS (AXI) transactions including debug features (Breakpoints / Single-Step), MMIO/CSR register sweeps, and ITCM/DTCM/DRAM burst non-blocking access
- Established functional coverage collection & analysis flow

**32-bit Floating-Point IP Core Verification**
- Independently designed and maintained a reusable UVM verification environment with DPI integration (C reference model)
- Covered boundary values, special values, and sign patterns; achieved **98%+ coverage** and resolved **15+ critical bugs**

**ZKP On-Chip Network (NoC) Design & Optimization**
- Implemented Mesh-topology NoC for ZKP cryptography using Constellation + Chisel
- Proposed static priority-based packet scheduling for matrix transpose; achieved **21.8% speedup** on 8×8 benchmark
- Developed interactive Python simulation tool for traffic-flow and hotspot visualization

##### Technical Skills

- **Verification**: SystemVerilog, UVM, SVA, DPI, Coverage-Driven Verification
- **EDA Tools**: VCS (simulation & regression), Verdi (waveform debug / coverage)
- **Protocols**: AXI (AMBA), RISC-V ISA
- **Simulation**: gem5 microarchitecture simulation
- **Programming**: C/C++, Python (automation scripts), JavaScript (pre-transition)
- **Toolchain**: Linux, Git / SVN, LaTeX