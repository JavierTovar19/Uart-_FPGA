<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/T08-blink4-iCEstick-1.png" width="400" align="center">

[Examples of this chapter on github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T08-register)

## Introduction
**Registers** are **essential elements** in digital circuits. We can store information, from 1 to N bits. They are used to implement processors, perform segmentation, storage of intermediate results, and more. 

The basic register captures the input data on the rising or falling edge of a clock, stores the data, and outputs it. The diagram is: 

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/blink4-2.png)

In this chapter we will use a 4 bit register to make the 4 LEDs on the iCEstick board blink. 

## blink4: Turning the LEDs on and off

The design we will use is the following: 

![Image 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/blink4-3.png)

The 4-bit register is initialized to 0. For the output dout 4 bits are output to 0. These 4 bits are changed to 1 when passing through the invertor. On the next rising ege of the clock, this new value of 4'b1111 is captured in the register. When the output dout is updated to 4b'1111, these results are again inverted and we've cycled back to the beginning. The result is the following sequence 0000, 1111, 0000, 1111 ... is obtained with each change corresponding to the rising edge of the clock. If the dout is connected to the LEDs, they were start turning on and off at 1/2 the clock frequency. 

## Hardware Description 

The complete design to be implemented in the FPGA is shown in this figure: 

![Image 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/blink4-1.png)

In order to actually be able to see the LEDs flicker, a prescaler is included. The hardware description in verilog is: 

```verilog
//-- blink4.v
module blink4(input wire clk,           //--clock
              output wire [3:0] data    //-- output register);
    
//-- Bits for the prescaler
parameter N = 22;
    
//-- main clock (prescalor)
wire clk_base;
    
//-- register data
reg [3:0] dout = 0;
    
//-- wire to the register
wire [3:0] din;
    
//-- Instance of the prescaler
prescaler #(.N(N))
  PRES (
    .clk_in(clk),
    .clk_out(clk_base)
  );
    
//-- Register
always @(posedge(clk_base))
  dout <= din;
    
//-- Not gate sets the input to the output
assign din = ~dout;
    
//-- output data from the register to the output of the module
assign data = dout;
    
endmodule
```

We remember that the **implementation of a register** is extremely simple. Just use this code: 

```verilog
 always @(posedge(clk_base))
   dout <= din;
```

This is the reason why we won't normally use hiearchical design for registers: it's too simple for hierarchy makes any improvements. 

We have used a **compact definition** of the blink4 module parameters: everything is defined in the module declaration itself: 

```verilog
module blink4(input wire clk, output wire [3:0] data);
```

But this could have been done as well: 

```verilog
module blink4(input clk, output [3:0] data);
wire clk;
wire data;
```
    
or this way: 

```verilog
module blink4(clk, data);
input clk;
output data;
wire clk;
wire [3:0] data;
```

The later form is used to define parametric components (see below)    

## Synthesis of the FPGA
To synthesize the design in the FPGA we run:

    $ make sint

The resources used are: 

| Resource | utilization
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

To download to the FPGA we do: 

    $ sudo iceprog blink4.bin

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/T08-blink4-iCEstick-1.png" width="400" align="center">

In the video you can see the result: 

[![Click to see the youtube video](http://img.youtube.com/vi/YFdEBQlX604/0.jpg)](https://www.youtube.com/watch?v=YFdEBQlX604)

### Initialization of registers

The **synthesized registers** always have an **initial value of 0**. The line of code defining the register is: 

```verilog
reg [3:0] dout = 0;
```

But the initialization in this line only works for simulation. It can give any value and we will see it in the simulation, however in "real hardware" we cannot do this. They will always be initialized to 0. To put another value we would have to load the register with this value. In the example blink4, if we could load the record initially with the value 1'b1010 (instead of 1'b0000), inverting it woudl yield 1'b0101. So we would have a different sequence: 1010, 0101, 1010 ... How coudl we implement this load? It is left as an exercise for the user to think about. In the following chapters we will show the necessary elements to accomplish it. 

## Simulation

The test bench is very basic. It simply instantiates the blink4 component, generates the clock signal, and starts the simulation. For the simulation to run for less cycles, the prescaler N parameter has been set to 1 bit. 

![Image 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/blink4-4.png)

The code is: 

```verilog
//-- blink4.v
module blink4_tb();
    
//-- register to generate the clock signal
reg clk = 0;
    
//-- output data of the component
wire [3:0] data;
    
//-- Instantiate the component
blink4 #(.N(1))
  TOP (
    .clk(clk),
    .data(data)
  );
    
//-- generate the clock with a 2 period cycle
always #1 clk = ~clk;
    
//-- process to start
initial begin
    
  //-- file to store the results 
  $dumpfile("blink4_tb.vcd");
  $dumpvars(0, blink4_tb);
    
  # 100 $display("FIN de la simulacion");
  $finish;     
end
endmodule
```

to simulate, we execute: 

    $ make sim

The output of gtkwave is:

![Image 5](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/T08-blink4-sim-1.png)

You can see the sequence 0000, 1111, 0000, 1111 ...


## Proposed exercises 
* Ex1: Change the flashing frequency of the LEDs
* Ex2: How could we make it so that the registers are loaded with 1'b1010 initially and thus get the sequence 1010, 0101, 1010.... instead of the current one? Think about it. In the following chapters we will go over the necessary elements to do it. 
* Ex3: Add self checking to the simulation so that the simulation warns you if the sequence is wrong. Change the simulation initialization to 1'b1111 to verify the self checking works. 

## Conclusions
TODO



