<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/T04-counter-iCEstick-1.png" width="400" align="center">

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T04-counter)

## Introduction
We'll model our **first sequential circuit**: a counter, connected to the LEDs. Sequential circuits, unlike the combinational ones, **store information**. The counter stores a number that increases with every tic of the clock.

Our component looks like this. It's updated every rising edge of the clock signal, and it's **"data" output** is a **26-bit bus**.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-1.png)

iCEstick board's **clock signal** oscillates at **12Mhz**. If we made a counter with only 4 bits, and connect those bit to the LEDs; with that clock frequency they will change so fast that we'll always see the LEDs on. That's why we will use a 26-bit counter, and output the 4 most significant bits through the LEDs.

## Hardware description

The counter has a **clk input** which is a wire, and a **26-bit output** that returns the value of the counter. This output is a **26-bit register** stores the current counter value. 

```verilog
//-----------------------------------
//-- Input: clock signal
//-- Output: 26-bit counter
//-----------------------------------
module counter(input clk, output [25:0] data);
wire clk;
    
//-- Output is 26-bit bus, initialized at 0
reg [25:0] data = 0;
    
//-- Sensitive to rising edge
always @(posedge clk) begin
  //-- Incrementar el registro
  data <= data + 1;
end
endmodule
```

The behaviour of the component is described inside the **always @(posedge clk)** block, that line indicates that everything described within this block, will be evaluated each time a **rising edge** comes in, from the clk signal. When a rising edge arrives, the data register is incremented by 1, (and is output, through the component's output).

Every counter, regardless of the amount of bit it has, is built this way. If we want it to have 20 bits, we just have to change the 25 number to 19 in the data register definition.

## Synthesis into the FPGA

The 4 most significant bits, (data[25], data[24], data[23] y data[22]) will be connected to the pins of the FPGA that are connected to the LEDs. The iCEstick clock comes from pin 21, so we'll connect that to the clk signal of our counter.

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-2.png)

The connection between our component and the pins in our FPGA is established in our **counter.pcf** file:

    set_io data[25] 96
    set_io data[24] 97
    set_io data[23] 98
    set_io data[22] 99
    set_io clk 21

We synthesise as always:

    $ make sint

The used resources are:

| Resource | usage
|----------|-----------
|PIOs      | 14 / 96
|PLBs      | 6 / 160
|BRAMs     | 0 / 16

To test it, we upload it into the FPGA as always:

    $ sudo iceprog counter.bin

We can see the counter functioning in this Youtube video:

[![Click to see the youtube video](http://img.youtube.com/vi/x9_OwUAtts4/0.jpg)](https://www.youtube.com/watch?v=x9_OwUAtts4)

## Simulation

The testbench is made of 4 (parallel) elements, connected by wires. The diagram would be the following:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-3.png)

There's a clock generator that produces a square signal, to increase the counter. The counter's output is checked by two different components: One makes the initial check, verifying that it starts at 0. The second has an internal variable that increases with every rising edge of the clock, and it's output is checked against the counter's output, to verify that indeed it is counting. Because it's a 26-bit counter, we won't check through all the 67108864 values, we will stop the simulation after 100 time units.

The verilog code would be:

```verilog
//-- counter_tb.v
module counter_tb();
    
//-- Register for generating the clock signal
reg clk = 0;
    
//-- Counter's output data
wire [26:0] data;

//-- Register for checking if the counter is counting properly
reg [26:0] counter_check = 1;
    
//-- Instantiating the counter
counter C1(
  .clk(clk),
  .data(data)
);
    
//-- Clock generator. 2 unit period
always #1 clk = ~clk;
    
//-- Counter value check
//-- Upon each rising edge, the counter's output is checked
//-- and it's incremented by the expected value
always @(negedge clk) begin
  if (counter_check != data)
    $display("-->ERROR!. Expected: %d. Read: %d",counter_check, data);
    
  counter_check <= counter_check + 1;
end
    
//-- Begin process
initial begin
    
  //-- File for storing the results
  $dumpfile("counter_tb.vcd");
  $dumpvars(0, counter_tb);
    
  //-- Reset check.
  # 0.5 if (data != 0)
          $display("ERROR! Counter is not 0!");
        else
          $display("Counter initialized. OK.");

  # 99 $display("END of simulation");
  # 100 $finish;
 end
 endmodule
```

It's important to note that all these components are working **in parallel**, all at once (the code is **NOT sequential**). Because of that, changing the order these elements are written would lead to the same result.

To simulate, we'll execute:

    $ make sim

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/T04-counter-sim-1.png)

Indeed the counter is counting. The image shows only the first values, but if you scroll the signal you can see until time unit 100.

## Proposed exercises
* Changing the counter from 26 to 24 bits, so LEDs flicker more rapidly.

## Conclussions
TODO
