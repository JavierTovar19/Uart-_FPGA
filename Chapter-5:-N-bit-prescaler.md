<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-iCEstick-1.png" width="400" align="center">

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T05-prescaler)

## Introduction

_Prescalers_ are used to **slow down clock signals**. A clock with a frequency of "f" goes into the input, and you get a lower frequency coming from the output. In this tutorial, we'll make an **N-bit prescaler** to make the LED blink at different frequencies.

![Image header](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-1.png)

For a N-bit prescaler, the **formulas** that relate the input frequencies and periods, with the ones at the output are:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-2.png)

## Understanding the 2-bit prescaler

Before implementing a N-bit prescaler, we are going to understand how a 2-bit prescaler works:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-3.png)

Internally, it's made of a **2-bit counter**, whose outputs are d0 and d1. The most significant bit is the one that is sent as output signal. This counter increases with each rising edge of clk, that has a period of "T". If we see the output signals of its 2 bits: (d0 and d1):

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-4.png)

we see that the period of d0 is 2 times T, and the d1 signal's is 4 times T. That means that every new bit duplicates the period of the previous signal. Following the general rule, the period of this 2-bit prescaler is: Tout = 2^2 * T = 4 * T (we can check that graphically).

## Frequency and periods of the prescaler

The **input frequency** into the prescaler in the iCEStick is *12MHz*. Applying the previous rule, we get this **periods and frequencies table** for prescalers with different number of bits (N). This gives us an idea of which value of N to choose, to make the LED blink.

| Bits (N)  | Frequency  |  Period | Visible
|-------|-------------|---------|-----------
|  1    |  6 MHz      |  0.167 usec | No
|  2    |  3 MHz      |  0.333 usec | No
|  3    |  1.5 Mhz    |  0.666 usec | No
|  4    |  750 Khz    |  1.333 usec | No
|  5    |  375 Khz    |  2.666 usec | No
|  6    |  187.5 Khz  |  5.333 usec | No
|  7    |  93.75 KHz  |  10.666 usec | No
|  8    |  46.875 Khz |  21.333 usec | No
|  9    |  23437.5 Hz |  42.666 usec | No
| 10    |  11718.7 Hz |  85.333 usec | No
| 11    |  5859.37 Hz |  170.66 usec | No
| 12    |  2929.68 Hz |  341.33 usec | No
| 13    |  1464.84 Hz |  682.66 usec | No
| 14    |  732.42 Hz  |  1.365 ms    | No
| 15    |  366.21 Hz  |  2.73 ms | No
| 16    |  183.1 Hz   |  5.46 ms | No
| 17    |  92.552 Hz  |  10.92 ms | No
| 18    |  45.776 Hz  |  21.84 ms | No
| 19    |  22.888 Hz  |  43.69 ms | Yes
| 20    |  11.444 Hz  |  87.38 ms | Yes
| 21    |  5.722 Hz   |  174.76 ms | Yes
| 22    |  2.861 Hz   |  349.52 ms | Yes

Human eyes have a refresh rate of around 25Hz. This means that anything happening at higher frequencies remain unnoticed. If we make the LED blink at a higher frequency, we'll see that as if was always on. (We won't see it blinking).

When you use the prescaler with an LED, **you notice the blinking when using 19 bits or more**. The more bits you use, the slower the LED will blink.

## Hardware description

The code is almost the same as the one of the counter, but this time we introduce this new feature that **this prescaler is parametric**, this way the number of bits is determined by the **N parameter**. We can synthesize different sizes of prescalers by just changing this parameter.

```verilog
//-- prescaler.v
//-- clk_in: input clock signal
//-- clk_out: output clock signal, lower frequency
module prescaler(input clk_in, output clk_out);
wire clk_in;
wire clk_out;
    
//-- Number of bits of the prescaler (default)
parameter N = 22;
    
//-- Register for implementing the N bit counter
reg [N-1:0] count = 0;
    
//-- The most significant bit goes through the output
assign clk_out = count[N-1];
    
//-- Counter: increases upon a rising edge
always @(posedge(clk_in)) begin
  count <= count + 1;
end
    
endmodule
```

We define a N-bit register, that increases with each rising edge of the input clock signal. It's most significant bit is connected directly to the clk_out signal.

**The prescaler is 22-bit by default**, so **clk_out's frequency** will be around **2.9Hz**. You only need to give N another value to change the output frequency.

## Synthesis into the FPGA

The 12MHz clock signal from the iCEStick goes through the pin 21 of the FPGA. The clk_out signal is sent to the D1 LED (pin 99), so it blinks at the same frequency.

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-5.png)

To synthesize, we execute:

    $ make sint

Resource usage is:

| Resource  | Usage
|----------|-----------
|PIOs      | 2 / 96
|PLBs      | 5 / 160
|BRAMs     | 0 / 16

We upload into the FPGA by executing:

    $ sudo iceprog prescaler.bin

The D1 LED will start blinking:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-iCEstick-1.png" width="400" align="center">

In this **Youtube video** you can see the LED blinking:

[![Click to see the youtube video](http://img.youtube.com/vi/HSqNa5iC2Qc/0.jpg)](https://www.youtube.com/watch?v=HSqNa5iC2Qc)

## Simulation

In the testbench we put the **N-bit prescaler** (N = 2 by default), a **clock signal generator** and a **check block** that executes every falling edge of the clock. This block has an internal register that increases each update, and its most significant bit is checked against clk_out, to assure that the latter is working properly. There's also a fourth block that initializes everything and waits for the simulation to finish.

![Imagen 6](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/prescaler-6.png)

The testbench code is the following:

```verilog
//-- prescaler_tb.v
module prescaler_tb();
    
//-- Number of bits of the tested prescaler
parameter N = 2;
    
//-- Register for generating the clock signal
reg clk = 0;
    
//-- Prescaler output
wire clk_out;
    
//-- Register for checking if the prescaler works
reg [N-1:0] counter_check = 0;
    
//-- Instantiate the N-bit prescaler
prescaler #(.N(N))  //-- N parameter
  Pres1(
    .clk_in(clk),
    .clk_out(clk_out)
  );
    
//-- Clock generator. 2 time units period
always #1 clk = ~clk;

//-- Counter value check
//-- Each falling edge, the counter output is checked.
//-- and the expected value is increased
always @(negedge clk) begin
    
  //-- Increase the check counter value
  counter_check = counter_check + 1;
    
  //-- The most significant bit must be the same as clk_out
  if (counter_check[N-1] != clk_out) begin
    $display("--->ERROR! Prescaler is malfunctioning");
    $display("Clk out: %d, counter_check[2]: %d", 
              clk_out, counter_check[N-1]);
  end
    
end
    
//-- Begin process
initial begin
    
  //-- File to store the results
  $dumpfile("prescaler_tb.vcd");
  $dumpvars(0, prescaler_tb);
    
  # 99 $display("END of simulation");
  # 100 $finish;
end
endmodule
```

To simulate we execute:

    $ make sim

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-sim-N-2.png)

In this simulation of the 2-bit prescaler we can see how indeed the output signal has a period 4 times greater than the input signal.

We repeat the simulation, but with a 3-bit prescaler, changing this line:

    parameter N = 2;

The result is:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T05-prescaler/images/T05-prescaler-sim-N-3.png)

Now the output signal has a period 8 times greater that the input signal.

## Proposed exercises
* Modify the prescaler with values of N = 18, 19, 20 and 21. Synthesize them and upload into the iCEStick.
* Make a simulation for N = 4

## Conclusions
TODO


