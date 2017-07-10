![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/init-2.png)

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T09-inicializador)

## Introduction
Many digital circuits **need to be initialized** before starting to work normally. A circuits operation is divided into a start state, where the values of the registers are initialized, and a steady state where the function for which they have been designed is performed. 

To achieve this, we need an initialization circuit that generates a **step signal**: initially it is at zero and when the first clock edge arrives, it changes to a 1 and remains at 1 during the whole operation of the machine. 

![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/init-2.png)

this is implemented in a very simple way using a **1-bit register** (or flip-flop) which is wired to 1'b1 at it's input. Initially the register will be 0. When the first clock edge arrives, the 1 is captured by the circuitry and passed on to it's output, **generating the rising edge to initialize**. For the rest of the systems clock cycles this signal will always be at 1'b1. 

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/init-3.png)

## Init.v: Hardware Description 

Since this is a register, we will implement it similar to last chapter. This is the **natural implementation** . Another possibility is to make an **optimized implementation** using less verilog code. 

### Natural Implementation

This logical implementation starts from a generic 1 bit register that we understand how to model. From there we simply wire it's input to 1, and take it's output: 

```verilog
//-- init.v (natural implementation)
//-- input: clock 
//-- output: initialization signal 
module init(input wire clk, output wire ini);
    
//-- register input 
wire din;
    
//-- register output (initialized to 0 in simulation) 
//-- For synthesis, initializations cannot be non-zero.
reg dout = 0;

//-- generic rising edge clock register 
always @(posedge(clk))
  dout <= din;
    
//-- assign 1 to the input
assign din = 1;
    
//-- connect the register output to the module output
assign ini = dout;
    
endmodule
```

This implementation is simple and easy to understand. It's clear to see that it's a register with 1'b1 wired to it's input, but it's also very verbose -- there's a lot of code defining a simple thing. 

### Optimized Implementation

We can do the same thing but with much less code by directly implemnting a register that always assigns 1 to it's output: 

```verilog
//-- init.v  (optimized version)
module init(input wire clk, output ini);

//-- 1 bit register initialized to 0 for sim
//-- for synthesize, this initialization value is ignored. 
reg ini = 0;
    
//-- on the rising edge of the clock, we assign 1 to the output
always @(posedge(clk))
  ini <= 1;
    
endmodule
```

## Synthesis of the FPGA 

to test this in the fpga we will simply **connect the output ini to an LED**. When loading the design into the FPGA we will only see that an LED is turned on, however this validates this simple circuit enough to initialize the designs in the following chapters. 

![Image 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/init-1.png)

Synthesize the design with:

    $ make sint

The resources used are:

| Resource | utilization
|----------|-----------
|PIOs      | 2 / 96
|PLBs      | 1 / 160
|BRAMs     | 0 / 16

Load into the FPGA with:

    $ sudo iceprog init.bin

The LED will light up. Our initialization circuit works, and we'll get more out of it in the following chapters.  :-)

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/T09-init-iCEstorm-1.png" width="400" align="center">

## Simulation

To simulate the design we will use a simple test bench in which we do not carry out verification of the output signal, but instead do it visually. The init component is instantiated, the clock is generated, and the test init process is run. 

![Image 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/init-4.png)

The Verilog code is:

```verilog
//-- init_tb.v
module init_tb();
    
//-- register for the clock signal
reg clk = 0;
    
//-- output data of the component
wire ini;
    
//-- Instantiate the component
init 
  INIT (
    .clk(clk),
    .ini(ini)
  );
    
//-- generate the clock with a 2 cycle period
always #2 clk = ~clk;
    
//-- start process
initial begin

//-- file to store the results
$dumpfile("init_tb.vcd");
$dumpvars(0, init_tb);

# 20 $display("FIN de la simulacion");
$finish;

end
endmodule
```

The result of the simulation is: 

![Image 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T09-inicializador/images/T09-init-sim.png)

We observe how the signal ini is 0 at the beginning and as soon as the first clock edge arrives it is set to one and remains in that state the rest of the simulation. 

## Modifications to boot time
By default, this initializer is in the **initial state** for **1 clock cycle**. If we are interested in having more cycles, we could just **add a prescaler** to the clock input, and connect it's output clk to the clock of the initialization register. 

## Proposed exercises
* Exercise 1: Modify the initializer so that it's initial state lasts for more than 1 clock cycle. 
* Exercise 2: Add self checking to the test bench to verify both initial state and final state. 

## Conclusions
TODO


