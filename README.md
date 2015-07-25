opl3_fpga
=========
Reverse engineered SystemVerilog RTL version of the 
<a href="http://en.wikipedia.org/wiki/Yamaha_YMF262">Yamaha OPL3 (YMF262)</a> FM Synthesizer.
Design is complete and working on the Digilent ZYBO board. Further testing is needed to verify
full functionality and coverage. I'll mostly be adding and working on the software at this point.

There are some minor differences between this version and the original chip, mainly due to the hardware on
the board that I'm using. The design is targeted to the <a href="https://www.digilentinc.com/Products/Detail.cfm?Prod=ZYBO">Digilent ZYBO board</a>
which has the <a href="http://www.xilinx.com/products/silicon-devices/soc/zynq-7000.html">Xilinx Zynq-7000
SoC</a> containing an ARM dual-core Cortex-A9 and an FPGA. The board also has an 
<a href="http://www.analog.com/en/products/audio-video/audio-codecs/ssm2603.html#product-overview">Analog Devices SSM2603
audio codec</a> with dual 24-bit DACs. So the interfaces are different--the interface to the CPU is AXI4-Lite
and the interface to the DAC is I<sup>2</sup>S, matching the particular hardware I have to work with. These
interfaces are wrapped in the design and would be easy to swap out.

The original OPL3 chip used a 14.31818MHz master clock. The sample rate was 14.31818MHz/288 = 49.7159KHz.
With 36 operator slots, that gives 8 clocks to update each operator each sample (operator logic is
time-shared between slots).

The SSM2603 on the ZYBO uses 256x oversampling, and thus requires a master clock 256x the sample clock.
I decided to use a single clock domain and keep the sample clock as close as possible to the original
chip. The ZYBO provides the FPGA with an external clock of 125MHz. Using a mixed-mode clock manager (MMCM) in the FPGA,
I'm able to synthesize a 12.727MHz clock from that, which divided by 256 gets me very close to the original
sample rate at 49.7148KHz. Updating 36 operator slots using my slower master clock only gives me 7 clock
cycles per sample instead of 8, but that's okay because I can stack tons of combinational logic up with
few pipeline registers with this slow clock speed and modern FPGA.

Several other very smart people put in a lot of work before me to get as close as possible to bit-true
software implementations of the chip, and this FPGA design would not be possible without their work. They're
all over at http://forums.submarine.org.uk/phpBB/viewforum.php?f=9

Tools used are Modelsim, Vivado 2015.1, Octave (for sample analysis), and SVEditor (for SystemVerilog file editing).

## Final utilization in xc7z010:

    +----------------------------+-------+-------+-----------+-------+
    |          Site Type         |  Used | Fixed | Available | Util% |
    +----------------------------+-------+-------+-----------+-------+
    | Slice LUTs                 |  8012 |     0 |     17600 | 45.52 |
    |   LUT as Logic             |  7946 |     0 |     17600 | 45.15 |
    |   LUT as Memory            |    66 |     0 |      6000 |  1.10 |
    |     LUT as Distributed RAM |     0 |     0 |           |       |
    |     LUT as Shift Register  |    66 |     0 |           |       |
    | Slice Registers            | 11119 |     0 |     35200 | 31.59 |
    |   Register as Flip Flop    | 11119 |     0 |     35200 | 31.59 |
    |   Register as Latch        |     0 |     0 |     35200 |  0.00 |
    | F7 Muxes                   |   385 |     0 |      8800 |  4.38 |
    | F8 Muxes                   |     0 |     0 |      4400 |  0.00 |
    +----------------------------+-------+-------+-----------+-------+
    
    +-------------------+------+-------+-----------+-------+
    |     Site Type     | Used | Fixed | Available | Util% |
    +-------------------+------+-------+-----------+-------+
    | Block RAM Tile    |    1 |     0 |        60 |  1.67 |
    |   RAMB36/FIFO*    |    0 |     0 |        60 |  0.00 |
    |   RAMB18          |    2 |     0 |       120 |  1.67 |
    |     RAMB18E1 only |    2 |       |           |       |
    +-------------------+------+-------+-----------+-------+
    
    +----------------+------+-------+-----------+-------+
    |    Site Type   | Used | Fixed | Available | Util% |
    +----------------+------+-------+-----------+-------+
    | DSPs           |    1 |     0 |        80 |  1.25 |
    |   DSP48E1 only |    1 |       |           |       |
    +----------------+------+-------+-----------+-------+
