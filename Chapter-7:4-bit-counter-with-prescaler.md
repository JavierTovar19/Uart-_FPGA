![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T07-contador-prescaler/images/counter4-1.png)

[Examples of this chapter on github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T07-contador-prescaler)

## Introduction
**4 bit counter** connected to the LEDs. To count more slowly, the clock signal of the counter is passed through a **22 bit prescaler**. This is the same counter as in Chapter 4, but with an improved design: to change the frequency of the counter, just change the parameter of the prescaler (and no need to modify the width of the counter or reassign them in the .pcf file). 




## Hardware Description
The counter has a **clock input** and a **4-bit data output**. It also has parameter N to indicate the number of bits of the prescaler and set it's operating frequency. The verilog code is as follows: 

```verilog
//-- counter4.v
module counter4(input clk, output [3:0] data);
wire clk;
reg [3:0] data = 0;
    
//-- Parameter for the prescaler
parameter N = 22;
    
//-- Output clock of the prescaler
wire clk_pres;
    
//-- Instantiate the prescaler of N bits
prescaler #(.N(N))
  pres1 (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
//-- Increment the counter on each rising edge. 
always @(posedge(clk_pres)) begin
  data <= data + 1;
end
    
endmodule
```

You could have defined a generic counter in a seperate file and instantiate it (just like you did with the prescaler). However, in this case this is to show an example of hierarchical design mixed with a component defined in a process. In addition, the counter is so simple that it is not worth it to store in a seperate file. 

The code has been implemented following the design in image 1. 

## Synthesis of the FPGA

The 12 Mhz clock signal enters the FPGA via pin 21, and the output data is sent to each pin connected to the LEDs, from 99 to 96. 

to synthesis the design, execute the command: 

    $ make sint

The resources used are:

| Resources| utilization
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Load the design into the fpga with the command: 

    $ sudo iceprog counter4.bin

## Simulation
The testbench is similar to Chapter 4, but the counter test process has been slightly modified. Now instead of performing the check on the falling edge, it is done when any change occurs in the data output of the counter. This is accomplished by putting data in the sensitivity list of the checking process. 

```verilog
//-- counter4_tb.v
module counter4_tb();
    
//-- register to generate the clock signal
reg clk = 0;
    
//-- output data from the counter
wire [3:0] data;
    
//-- register for checking the counter
reg [3:0] counter_check = 0;
    
//-- instantiate the counter, with 1 bit prescaler during simulation
counter4 #(.N(1))
  C1(
    .clk(clk),
    .data(data)
  );

//-- clock generator with 2 cycle period
always #1 clk = ~clk;
    
//-- Check Process. Every time there's a change in 
//-- the counter, the design value is checked against the expected value
always @(data) begin
    
  if (counter_check != data)
    $display("-->ERROR!. Expected: %d. Actual: %d",counter_check, data);
    
  counter_check = counter_check + 1;
end
    
//-- Start Process
initial begin
    
  //-- result file
  $dumpfile("counter4_tb.vcd");
  $dumpvars(0, counter4_tb);
    
  //-- check the reset
  # 0.5 if (data != 0)
          $display("ERROR! Counter is NOT 0!");
        else
	  $display("Counter initialized. OK.");
    
 # 99 $display("End of the simulation");
 # 100 $finish;
end
    
endmodule
```


The verification process begins with 

    always @(data) begin

This means it runs every time there is a change in `data`

To simulate we execute:

    $ make sim

The result is:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T07-contador-prescaler/images/T07-counter4-simulation-1.png)

We see that the counter is counting. a 1 bit prescaler was used to make this occur much faster in simulation. 

## Proposed exercises
* Change the prescaler to count faster.
* Change the counter to count down. 

## Conclusions
TODO
