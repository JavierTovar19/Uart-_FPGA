<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-iCEstick-1.png" width="400" align="center">

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T02-Fport)

## Introduction

Now, instead of just one bit we will output four of them, and display them using the LEDs. It's a fixed value, wired "within the hardware". If we wanted to display any other number on the LEDs, we will need to synthesize again.

![Pic 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-1.png)

We will name this component "Fport" (Fixed port). It has a 4-bit output bus, labeled as "data", wired to the binary value "1010".

## Fport.v: Hardware description

This circuit is similar to the one in the last tutorial, (setbit.v) but instead of having just 1 output bits, it has 4 bits. It's described like this:
```verilog
    //-- Fichero Fport.v
    module Fport(output [3:0] data);
    
    //-- Module output is a 4 wire bus.
    wire [3:0] data;
    
      //-- Output the value through that 4-bit bus.
      assign data = 4'b1010; //-- 4'hA
    
    endmodule
```
The output is now a **4 wire array**. You can express that by writing [3:0] before the wire name. In order to assign a value to that wire, we write the value using Verilog's notation: First, the number of bits, then the ' character, after that, the base of the number (in this case, "b" for "binary") and finally, the 4 binary digits. This number could be expressed like a single hexadecimal digit, by writing 4'hA. We could also write it in decimal as 4'd10.

## Synthesis into the FPGA

Each one of the 4 output bits will be assigned to the pins connected to each one of the 4 leds of the FPGA:

![Pic 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-2.png)

This is specified in the Fport.pcf file:

    set_io data[0] 99
    set_io data[1] 98
    set_io data[2] 97
    set_io data[3] 96

In order to synthesize, we navigate to the **tutorial/T02-Fport** folder, and execute the command **make sint**:

    $ make sint

The final message will print out how many resources this circuits requires in the FPGA, and how many of them are available:

    After placement:
    PIOs       2 / 96
    PLBs       1 / 160
    BRAMs      0 / 16

These resources are:
* PIO = Programmable Inputs and Outputs
* PLBs = Programmable Logic Blocks
* BRAMs = Block RAM Memory 

Now we upload the Fport.bin file into the FPGA:

    $ sudo iceprog Fport.bin

Upon finishing, two LEDs will be on, and two LEDs will be off, given that we are sending the value "1010":

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-iCEstick-2.png" width="400" align="center">

## Simulation

This testbench is similar to the one in the last chapter. But now, instead of checking just one bit, we check the whole 4-bit bus. If it's not exacly as expected, it throws an error. The diagram is the following:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-3.png)

The testbench (Fport_tb.v) has this three elements:

* The UUT (Unit Under Test): Fport
* The checking block 
* The DATA wire

The testbench code is:
```verilog
    //-- Fport_tb.v
    module Fport_tb;
    
    //-- 4-wire bus, to connect it to the Fport component output.
    wire [3:0] DATA;
    
    //--Instantiating the component. Connect output to DATA.
    Fport FP1 (
      .data (DATA)
    );
    
    //-- Begin the test
    initial begin
    
      //-- File in which store the results.
      $dumpfile("Fport_tb.vcd");
      $dumpvars(0, Fport_tb);
    
      //-- After 10 time units we check
      //-- whether the cable has the previously given pattern or not.
      # 10 if (DATA != 4'b1010)
             $display("---->¡ERROR! Salida Erronea");
           else
             $display("Componente ok!");
    
      //-- Finish the simulation 10 time units after that.
      # 10 $finish;
    end
    
    endmodule
```
Note that the component output is labeled "data" and it is connected to "DATA". With this we want to make clear that the names can be different.

To make the simulation we execute **make sim**:

   $ make sim

This is the result displayed with gtkwave:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T02-Fport/images/Fport-sim-1.png)

We can check that the output is always "1010".

## Proposed exercises
* Change the value in order to output another pattern through the LEDs. Simulate, synthesize, and upload into the FPGA.
* Modify the component so it features a 5-bit bus instead of 4. Output the 5th bit through the 44 pin in the FPGA (this one is not connected to any LED)

## Conclusions
TODO

