![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-1.png)

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T12-mux-4-1)

## Introduction
Multiplexers allow us to **select between M inputs** (M being a pwoer of 2). Only the source that is selected is output. In the previous chapter we saw how to implement a simple multiplexer, from 2 to 1. In this chapter we generalize to M to 1, making an example fo 4 to 1. We will use it to create a sequencer of 4 states that will be output to the LEDs. 

## Multiplexer M to 1
**The multiplexer has M data inputs**, M being a power of 2(2, 4, 8, 16, ...). All inputs and output are N bits. The selection input has b bits, with M = 2^b 

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-2.png)

## Multiplexer 4 to 1

As an example, we will implement a 4-bit multiplexer passing on 4 bits. The selection input has 2 bits: 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-3.png)

the verilog code is intuitive. We will use the case statement: 

```verilog
always @*
  case (sel)
     0 : out <= source0;
     1 : out <= source1;
     2 : out <= source2;
     3 : out <= source3;
     default : out <= 0;
  endcase
```

Note that **all valid cases are covered**. Even so, the **default** case has been added. This ensures that if sel has a Z or X value, known output occurs. This guarantees that a **combinatorial circuit** is implemented. If all cases are not covered, the synthesizer may infer a latch. 

In the sensitivity list, we could have placed 5 entries, but instead used @* which automatically determines all variables in the process and uses them all in the sensitivity list. 

## mux4: 4 state sequencer 
We will use a multiplexer of 4 to 1 to make a sequencer of 4 states, generating a sequenc that is visible via the LEDs. For the 4 inputs we wire the values of the sequence. The default used is 0000, 1010, 1111, 0101. The selection input is connected to a 2 bit counter so that the 4 inputs are sequentially selected. The counter clock is connected to a prescaler to slow down the frequency, making the different selections visible to the naked eye. 
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-4.png)

The Verilog code is:

```verilog
//-- mux4.v
module mux4(input wire clk, output reg [3:0] data);
    
//-- Parametros del secuenciador:
parameter NP = 23;         //-- Bits of the prescaler
parameter VAL0 = 4'b0000;  //-- sequence value 0
parameter VAL1 = 4'b1010;  //-- sequence value 1
parameter VAL2 = 4'b1111;  //-- sequence value 2
parameter VAL3 = 4'b0101;  //-- sequence value 3
    
//-- wires for the 5 inputs to the multiplexer
wire [3:0] val0;
wire [3:0] val1;
wire [3:0] val2;
wire [3:0] val3;
wire [1:0] sel;  //-- 2 bits of selection. 
    
//-- 2 bit counter
reg [1:0] count = 0;
wire clk_pres; //-- output clock of the prescaler 
    
//-- wire the constants to the inputs of the mux. 
assign val0 = VAL0;
assign val1 = VAL1;
assign val2 = VAL2;
assign val3 = VAL3;
    
//-- implement the 4 to 1 multiplexer 
always @*
  case (sel)
     0 : data <= val0;
     1 : data <= val1;
     2 : data <= val2;
     3 : data <= val3;
     default : data <= 0;
  endcase
    
 //-- counter to drive the sel wire 
 always @(posedge(clk_pres))
  count <= count + 1;
     
//-- drive the sel wire with the counter value
assign sel = count;
    
//-- prescaler that controls the counter frequency
prescaler #(.N(NP))
  PRES (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
endmodule
```


## Synthesis in the FPGA 

To synthesize the design into the FPGA we will connect the data outputs to the LEDs, and the clock input to that of the iCEstick. 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-1.png)

Synthesize with the command: 

    $ make sint

The resources used are:

| Resource | utilization
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 6 / 160
|BRAMs     | 0 / 16

To load the FPGA we execute:

    $ sudo iceprog mux4.bin

In this **Youtube video** you can see the output of the LEDs:

[![Click to see the youtube video](http://img.youtube.com/vi/6z3PyGcX_eg/0.jpg)](https://www.youtube.com/watch?v=6z3PyGcX_eg)

## Simulation
The testbench is a basic one, which instantiates the mux4 compnent, with 1 bit for the prescaler for less cycles in the simulation. It has a process for the clock signal, and one for the initialization of the simulation. 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/mux4-5.png)

The simulation is performed with: 

    $ make sim

The result in gtkwave is:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T12-mux-4-1/images/T12-mux4-sim-1.png)

We see how the 4 outputs are alternated: 0000, 1010, 1111, 0101, each associated with a value of the signal of sel. 

## Proposed Exercises
* Exercise 1: Modify the values to get a different sequence
* Exercise 2: Make an 8-state sequencer using an 8-1 multiplexer
* Exercise 3: Make a self checking testbench
## Conclusions
TODO