![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-3.png)

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T10-shif-register)

## Introduction
The shift registers **store a value** and **shifts it**. They are extremely useful. They are used to convert information from parallel to serial (and vice versa) for use in **syncronous communications**. Communications through SPI, I2C, and more are implemented with these registers. The also allow us to perform the operations of multiplying by powers of 2 and dividing by powers of 2 for integers. 

In this chapter we will use them to generate a sequence of 4 states on the LEDs of the iCEstick, moving the lights clockwise. 

## Description of the register. 
The shift register we will use is as follows: 

![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-2.png)

The **output** of the register is **N bits** (in our example we will use a 4 bit register). It has an N-bit **Parallel input**, which allows us to load the register with a new value. The **load_shift signal** allows us to determine the **operating mode**: when it is at 0, a new value is **loaded** when a rising edge of the clock arrives. When it is at 1, a **right shift** is made on the rising edge of the clock.

In this shift **the most significant bit is lost** and the new value is read from the **serin input** (serial input). If we have the initial value 1'b1001 stored, and the signal load_shift is at 1, while serin is at 0 and a rising edge of the clock arrives, we get the value: 0010. On the next cycle (if serin statys 0) we get 0100, then 1000, and then 0000. 

## shift4: Rotation of bits

As an example, we will use a **4-bit shift register** to rotate a sequence of bits and display them by the 4 LEDs on the iCEstick. The sequence obtained by the LEDs will depend on the initial value loaded in the register. 

The block diagram of the shift4 component is: 

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-1.png)

The main component is a **4-bit shift register**. Its clock input is connected to the iCEstick clock via a prescaler component to lower its frequency and to be able to see the shifting of the bits in the LEDs. 

The most significant bit of the register (**data[3]**) is connected directly to the **serin** input, so that bit rotation is achieved (the most significant one becomes the least significant). 

Via the parallel input we introduce the **initial value**, which by default will be **1'b0001**. Rotation will show the sequence 0010, 0100, 1000, and 0001. 

The initial load is done using an **initializer**, as shown in chapter 9. It is connected to the input **load_shift**, so that initially it is 0 and when the first clock edge arrives it changes to 1, loading the initial value and changing to shift mode. The rest of the cycles will be shifting. This is an example of how the initializer works. 

## Hardware Description

### Shift Register

The shift register is described with very few lines. It is a process that depends on the rising edge of the clock. 

```verilog
always @(posedge(clk_pres)) begin
  if (load_shift == 0)  //-- Load mode
    data <= INI;
  else
    data <= {data[2:0], serin};
end
```

In the **load mode** the initial value (INI) is output. In the shift, the three least significant bits are shifted left and serin is concatenated as the least significant bit. This is done with the **concatenation operator{}**: a fourth wire is added to the three wires defined by data[2:0]. 


### Shift4 Component

The final component of shift4 has **2 parameters**: the **number of bits of the prescaler (NP), which determines the speed of ration of the bits, and the **initial value** (INI) to be loaded, which determines the sequence. By default, this initial value is 4'b0001

The code to describe the complete shift4 component is: 

```verilog
//-- shift4.v
module shift4(input wire clk, output reg [3:0] data);
    
//-- parameters of the sequencer
parameter NP = 21;  //-- Bits of the prescaler
parameter INI = 1;  //-- initial value to load into the shift register
    
//-- clock from the prescaler
wire clk_pres;
    
//-- Shift / load. Signal indicating whether the register is loaded or shifted
//-- shift = 0: load
//-- shift = 1: shifted
reg load_shift = 0;
    
//-- serial input of the register
wire serin;
    
//-- Instantiate the N bit prescaler
prescaler #(.N(NP))
  pres1 (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
//-- Initializer
always @(posedge(clk_pres)) begin
    load_shift <= 1;
end
    
//-- shift register
always @(posedge(clk_pres)) begin
  if (load_shift == 0)  //-- Load mode
    data <= INI;
  else
    data <= {data[2:0], serin};
end
    
//-- assign the MSB to the serial input to create a ring
assign serin = data[3];
    
endmodule
```

Thie shift register has been directly implemented as a process in the component itself, rather than as a separate component (hierarchical design).

## Synthesis of the FPGA

To synthesize it in the FPGA we will connect the data output to the LEDs and the clock input to that of the iCEstick. 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-3.png)

Synthesize with the command:

    $ make sint

The resources used are:

| Resource | utilization
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 10 / 160
|BRAMs     | 0 / 16

To load into the FPGA we execute:

    $ sudo iceprog shift4.bin

In this **Youtube video** you can see the output of the LEDs: 

[![Click to see the youtube video](http://img.youtube.com/vi/JgwFiXQobi4/0.jpg)](https://www.youtube.com/watch?v=JgwFiXQobi4)

## Simulation

The testbench is a basic one, which instantiates the shift4 component, with 1 bit for the prescaler parameter (to make the simulation run over fewer clock cycles) and an initial value of 4'b0001 for the shift register. It has a process for the clock signal, and one for initialization of simulation. 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-4.png)

The simulation is run with:

    $ make sim

The result in gtkwave is: 

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/T10-shift4-sim-1.png)

We see how initially the register has an undefined value (x) and when the first rising edge arrives it is initialized with the value 0001, thanks to the initializer circuit). In the following cycles, we see how the bit is shifted to the left: 0010, 0100, 1000 and finally returns to the initial value: 0001, repeating the sequence until the end of the simulation. 

## Proposed Exercises
* Exercise 1: Change the value of the presacler so that the rotation is faster, and the initial value of the register, so that another sequence comes out. 
* Exercise 2: Use an 8-bit shift register, connecting the least significant 4 bits to the LEDs. This allows for making sequences in which all LEDs are off. 

## Conclusion
TODO



