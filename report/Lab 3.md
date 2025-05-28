---
title: report in Hackmd

---

# Soc Lab3
*Github: https://github.com/constant619/EESM6000C-Lab3*
## Block Diagram
### datapath flow
![扫描全能王 2025-05-29 05.01](https://hackmd.io/_uploads/HJgW5grzxg.jpg)
### Control signals
AXI-Lite: awready/wready/arready/rvalid for register programming.
AXI-Stream: ss_tready/sm_tvalid/sm_tlast for data streaming.
SRAM Control: tap_WE/tap_EN/data_WE/data_EN for memory access.
State Machine: cs (current state) drives all control logic.
## Describe the operation
How to receive data-in and tap parameters and place into SRAM
• How to access DataRAM and tapRAM to do the computation
• How ap_done is generated.
### Coefficient Loading:
Host writes coefficients via AXI-Lite to addresses 0x020-0x044.
FSM maps writes to Tap SRAM addresses 0x000-0x028.
tap_WE asserted during writes in input_2 state.
### Data Streaming:
Input samples arrive via AXI-Stream.
Stored in Data SRAM circular buffer when ss_tready=1 .
Write pointer (ptr) auto-increments and wraps at 10.
### Computation:
Triggered by write to control reg.
State machine sequences through:
calculte_1/calculte_2: Multiply-accumulate operations.
last_multi: Final multiplication.
accumulate: Sum completion.
out_state: Output result.
Reads alternate between Data SRAM and Tap SRAM.
Accumulator reset every 11 samples.
### ap_done Generation:
pattern_number counter increments per input sample.
ap_done=1 when counter reaches 600.
## Resource usage
```
+-------------------------+------+-------+------------+-----------+-------+
|        Site Type        | Used | Fixed | Prohibited | Available | Util% |
+-------------------------+------+-------+------------+-----------+-------+
| Slice LUTs*             |  286 |     0 |          0 |     53200 |  0.54 |
|   LUT as Logic          |  286 |     0 |          0 |     53200 |  0.54 |
|   LUT as Memory         |    0 |     0 |          0 |     17400 |  0.00 |
| Slice Registers         |  231 |     0 |          0 |    106400 |  0.22 |
|   Register as Flip Flop |  231 |     0 |          0 |    106400 |  0.22 |
|   Register as Latch     |    0 |     0 |          0 |    106400 |  0.00 |
| F7 Muxes                |    0 |     0 |          0 |     26600 |  0.00 |
| F8 Muxes                |    0 |     0 |          0 |     13300 |  0.00 |
+-------------------------+------+-------+------------+-----------+-------+
```
```
+----------------+------+-------+------------+-----------+-------+
|    Site Type   | Used | Fixed | Prohibited | Available | Util% |
+----------------+------+-------+------------+-----------+-------+
| Block RAM Tile |    0 |     0 |          0 |       140 |  0.00 |
|   RAMB36/FIFO* |    0 |     0 |          0 |       140 |  0.00 |
|   RAMB18       |    0 |     0 |          0 |       280 |  0.00 |
+----------------+------+-------+------------+-----------+-------+
```
## Timing Report
![image](https://hackmd.io/_uploads/Bk66y-SGel.png)

```
Max Delay Paths
--------------------------------------------------------------------------------------
Slack (MET) :             0.832ns  (required time - arrival time)
  Source:                 genblk1.cs_reg[2]/C
                            (rising edge-triggered cell FDCE clocked by axis_clk  {rise@0.000ns fall@5.000ns period=11.000ns})
  Destination:            genblk1.multi_result_reg[29]/D
                            (rising edge-triggered cell FDCE clocked by axis_clk  {rise@0.000ns fall@5.000ns period=11.000ns})
  Path Group:             axis_clk
  Path Type:              Setup (Max at Slow Process Corner)
  Requirement:            11.000ns  (axis_clk rise@11.000ns - axis_clk rise@0.000ns)
  Data Path Delay:        10.063ns  (logic 7.555ns (75.074%)  route 2.508ns (24.926%))
  Logic Levels:           8  (CARRY4=4 DSP48E1=2 LUT3=1 LUT5=1)
  Clock Path Skew:        -0.145ns (DCD - SCD + CPR)
    Destination Clock Delay (DCD):    2.128ns = ( 13.128 - 11.000 ) 
    Source Clock Delay      (SCD):    2.456ns
    Clock Pessimism Removal (CPR):    0.184ns
  Clock Uncertainty:      0.035ns  ((TSJ^2 + TIJ^2)^1/2 + DJ) / 2 + PE
    Total System Jitter     (TSJ):    0.071ns
    Total Input Jitter      (TIJ):    0.000ns
    Discrete Jitter          (DJ):    0.000ns
    Phase Error              (PE):    0.000ns

    Location             Delay type                Incr(ns)  Path(ns)    Netlist Resource(s)
  -------------------------------------------------------------------    -------------------
                         (clock axis_clk rise edge)
                                                      0.000     0.000 r  
                                                      0.000     0.000 r  axis_clk (IN)
                         net (fo=0)                   0.000     0.000    axis_clk
                                                                      r  axis_clk_IBUF_inst/I
                         IBUF (Prop_ibuf_I_O)         0.972     0.972 r  axis_clk_IBUF_inst/O
                         net (fo=1, unplaced)         0.800     1.771    axis_clk_IBUF
                                                                      r  axis_clk_IBUF_BUFG_inst/I
                         BUFG (Prop_bufg_I_O)         0.101     1.872 r  axis_clk_IBUF_BUFG_inst/O
                         net (fo=231, unplaced)       0.584     2.456    axis_clk_IBUF_BUFG
                         FDCE                                         r  genblk1.cs_reg[2]/C
  -------------------------------------------------------------------    -------------------
                         FDCE (Prop_fdce_C_Q)         0.478     2.934 r  genblk1.cs_reg[2]/Q
                         net (fo=119, unplaced)       0.845     3.779    genblk1.cs[2]
                                                                      r  genblk1.multi_result0_i_1/I2
                         LUT5 (Prop_lut5_I2_O)        0.295     4.074 r  genblk1.multi_result0_i_1/O
                         net (fo=1, unplaced)         0.800     4.874    genblk1.multi_result0_i_1_n_0
                                                                      r  genblk1.multi_result0/A[16]
                         DSP48E1 (Prop_dsp48e1_A[16]_PCOUT[47])
                                                      4.036     8.910 r  genblk1.multi_result0/PCOUT[47]
                         net (fo=1, unplaced)         0.055     8.965    genblk1.multi_result0_n_106
                                                                      r  genblk1.multi_result0__0/PCIN[47]
                         DSP48E1 (Prop_dsp48e1_PCIN[47]_P[0])
                                                      1.518    10.483 r  genblk1.multi_result0__0/P[0]
                         net (fo=1, unplaced)         0.800    11.283    genblk1.multi_result0__0_n_105
                                                                      r  genblk1.multi_result[19]_i_7/I2
                         LUT3 (Prop_lut3_I2_O)        0.124    11.407 r  genblk1.multi_result[19]_i_7/O
                         net (fo=1, unplaced)         0.000    11.407    genblk1.multi_result[19]_i_7_n_0
                                                                      r  genblk1.multi_result_reg[19]_i_1/S[1]
                         CARRY4 (Prop_carry4_S[1]_CO[3])
                                                      0.533    11.940 r  genblk1.multi_result_reg[19]_i_1/CO[3]
                         net (fo=1, unplaced)         0.009    11.949    genblk1.multi_result_reg[19]_i_1_n_0
                                                                      r  genblk1.multi_result_reg[23]_i_1/CI
                         CARRY4 (Prop_carry4_CI_CO[3])
                                                      0.117    12.066 r  genblk1.multi_result_reg[23]_i_1/CO[3]
                         net (fo=1, unplaced)         0.000    12.066    genblk1.multi_result_reg[23]_i_1_n_0
                                                                      r  genblk1.multi_result_reg[27]_i_1/CI
                         CARRY4 (Prop_carry4_CI_CO[3])
                                                      0.117    12.183 r  genblk1.multi_result_reg[27]_i_1/CO[3]
                         net (fo=1, unplaced)         0.000    12.183    genblk1.multi_result_reg[27]_i_1_n_0
                                                                      r  genblk1.multi_result_reg[31]_i_2/CI
                         CARRY4 (Prop_carry4_CI_O[1])
                                                      0.337    12.520 r  genblk1.multi_result_reg[31]_i_2/O[1]
                         net (fo=1, unplaced)         0.000    12.520    genblk1.multi_result_reg[31]_i_2_n_6
                         FDCE                                         r  genblk1.multi_result_reg[29]/D
  -------------------------------------------------------------------    -------------------

                         (clock axis_clk rise edge)
                                                     11.000    11.000 r  
                                                      0.000    11.000 r  axis_clk (IN)
                         net (fo=0)                   0.000    11.000    axis_clk
                                                                      r  axis_clk_IBUF_inst/I
                         IBUF (Prop_ibuf_I_O)         0.838    11.838 r  axis_clk_IBUF_inst/O
                         net (fo=1, unplaced)         0.760    12.598    axis_clk_IBUF
                                                                      r  axis_clk_IBUF_BUFG_inst/I
                         BUFG (Prop_bufg_I_O)         0.091    12.689 r  axis_clk_IBUF_BUFG_inst/O
                         net (fo=231, unplaced)       0.439    13.128    axis_clk_IBUF_BUFG
                         FDCE                                         r  genblk1.multi_result_reg[29]/C
                         clock pessimism              0.184    13.311    
                         clock uncertainty           -0.035    13.276    
                         FDCE (Setup_fdce_C_D)        0.076    13.352    genblk1.multi_result_reg[29]
  -------------------------------------------------------------------
                         required time                         13.352    
                         arrival time                         -12.520    
  -------------------------------------------------------------------
                         slack                                  0.832    
```
## Simulation Waveform
Issue: The current implementation doesn't fully comply with AXI-Lite read handshake requirements.
* Coefficient program, and read back
![image](https://hackmd.io/_uploads/S1fjX-rGgg.png)
* Data-in stream-in
![image](https://hackmd.io/_uploads/r1wWE-Hzel.png)
* Data-out stream-out
![image](https://hackmd.io/_uploads/H1z_NbrMel.png)
* RAM access control
![image](https://hackmd.io/_uploads/B1r8S-BGlx.png)
