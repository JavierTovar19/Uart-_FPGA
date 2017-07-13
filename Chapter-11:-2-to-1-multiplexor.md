![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/mux2-1.png)

[Examples of this chapter on github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T11-mux-2-1)

## Introduction
The **multiplexers** are combinational circuts that allow us to **select** from multiple data sources. In this chapter we will use a multiplexer of 2 to 1 to select between 2 sequences to display on the LEDs and alternate between them. 

## Description of the 2 to 1 multiplexer 

A 2 to 1 multipleser selects between 2 data sources according to the value of its sel **selection input**. If sel is 0, source 0 is output, if 1 then the source 1 is used. 

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/mux2-2.png)

We can describe multiplexers in Verilog using the if ... else statement: 

```verilog
always @(source0 or source1 or sel)
  if (sel == 0)
    dout <= source0;
  else
    dout <= source1;
```

It is every importnat that there is the **else**. Because this is a combinational circuit, **all cases must be covered**. If they are not, you will infer a latch. It is also very important for simulation to palce all the input signal into the **sensitivity list**: source0, source1, and sel. 

The sensitivity list can be written abbreviated with the argument @ * 

    always @*

This list automatically includes all input signals. 

## mux2.v: 2-state sequencer 

As an example use case, we will make a **2-state sequencer**: a circuit that alternately sends two nibbles (4-bits) to the LEDs. The schematic of the circuit is as follows: 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/mux2-3.png)

Two fixed data sources are used (they are wired to fixed values) that determine the status of the LEDs at any given time. The multiplexer selects alternatively between one and another through a clock signal that passes through a presaler (to reduce the fequency so we can appreciate the toggling of the LEDs). 

The verilog code is as follows:

```verilog
//-- mux2.v
module mux2(input wire clk, output reg [3:0] data);
    
//-- Parametros del secuenciador:
parameter NP = 22;         //-- Bits del prescaler
parameter VAL0 = 4'b1010;  //-- Valor secuencia 0
parameter VAL1 = 4'b0101;  //-- Valor secuencia 1
    
//-- wires of the three inputs of the multiplexer 
wire [3:0] val0;
wire [3:0] val1;
wire sel;
    
//-- for the inputs of the mux we wire the input data; 
assign val0 = VAL0;
assign val1 = VAL1;
    
//-- Implementation of the multiplexer
always @(sel or val0 or val1)
  if (sel==0)
    data <= val0;
  else
    data <= val1;
    
//-- Prescaler to control the select signal of the multiplexer
prescaler #(.N(NP))
  PRES (
    .clk_in(clk),
    .clk_out(sel)
  );
    
endmodule
```

The implementation of the multiplexer is simple so it is included directly in a process instead of defining it in a seperate file and then instantiating it (hierarchical design).

## Synthesis of the FPGA

To synthesize the design into the FPGA we will connect the data outputs to the LEDs, and the clock input to the iCEstick board. 

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/mux2-1.png)

Synthesize with the command:

    $ make sint

The resources used are:

| Resource | utilization
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 7 / 160
|BRAMs     | 0 / 16

to load in the FPGA we execute:

    $ sudo iceprog mux2.bin

In this **Youtube video** you can see the output of the LEDs:

[![Click to see the youtube video](http://img.youtube.com/vi/4GnH5lqlTOU/0.jpg)](https://www.youtube.com/watch?v=4GnH5lqlTOU)

## Simulation
The simulation is basic, with an instantiation of the sequncer (passing in 1 for the prescaler parameter) and a simple clock and initialization process. It is for visual inspection of the output. 

![Imagen 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/mux2-4.png)

The simulation is done with:

    $ make sim

The result in gtkwave is:

![Imagen 6](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T11-mux-2-1/images/T11-mux2-simulation.png)

We see how the two outputs are alternating: 1010 and 0101 alternately, each associated with a level of the sel signal which comes from the clock. 

## Proposed exercises
* **Exercise 1**: Change the input values of the multiplexer to output another different sequence by the FPGA. 
* **Exercise 2**: Write a self checking testbench

## Conclusions
TODO