![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-1.png)

[Examples of this chapter in githube capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T13-reg-init)

## Introduction 
We can now answer the question posed in Chapter 8 on **how to initialize registers**. The synthesized registers start out at 0. Often we need to load an initial value into them, and then enable them to start functioning. In this chapter we will show you how to do that, using the tools we already know -- the **multiplexer** and the **initializer**. 

## Initializing registers 

We start from a **generic register of N bits**, which we already know. It has din as input, dout as output, and a clock signal. 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-2.png)

We want it to load an initial value at the beginning, and then run normally. to do this, we put a **2 to 1 multiplexer** at it's input to allow two different inputs. For one input of the multiplexer we put the **initial value** and on the other, the regular input of the register. 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-3.png)

**It is very important that the initial value is using the 0 source of the multiplexer for this design**.

Now we connect an initializer to the selection input of the multiplexer. 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-4.png)

In this way, at the start the initializer will emit a 0 and the the multiplexer will pass on the initial value. On the next rising edge this initial value is captured by the register, and the initializer is set to and held at 1 so the multiplexer passes on it's source 1 input. This will be whatever the normal operating mode data is. expected to be. 

## reginit.v: 2-state sequencer with a register

We will redo the blink4 circuit of chapter 8. This circuit would blink all 4 LEDs at the same time, 0000, 1111, 0000,...

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T08-register/images/blink4-3.png)

Now, we will improve this design by making it possible to assign the register an initial value, achieving the sequence INI, ~INI, INI ...: 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-5.png)

La descripción de este circuito en Verilog es:

```verilog
//-- reginit.v
module reginit(input wire clk, output wire [3:0] data);
    
//-- sequencer parameters:
parameter NP = 23;        //-- prescaler bits 
parameter INI = 4'b1100;  //-- value to initialize the state register to
    
//-- output clock from the prescaler
wire clk_pres;
    
//-- output of the register
reg [3:0] dout;
    
//-- input of the register
wire [3:0] din;
    
//-- select signal of the multiplexer
reg sel = 0;
    
//-- Register
always @(posedge(clk_pres))
  dout <= din;
    
//-- Connect the register to the output
assign data = dout;
    
//-- initialization multiplexer
assign din = (sel == 0) ? INI : ~dout;
    
//-- Inicializer
always @(posedge(clk_pres))
  sel <= 1;
    
//-- Prescaler
prescaler #(.N(NP))
  PRES (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
endmodule
```

the 2 to 1 multiplexer has been implemented using the **operator ?:** (similar to the _continaonal operator_ of the C language). It's an abbreviated if-else: 
```verilog
assign din = (sel == 0) ? INI : ~dout;
```

which is equivelant to :

```
always @*
  if (sel == 0)
    din <= INI;
  else
    din <= ~dout;
```
    
But the first notation is more compact. 

## Synthesize the FPGA

To synthesize the design in the FPGA we will connect the data outputs to the LEDs, and the clock input to that of the iCEstick. 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-1.png)

Synthesize with the command:

    $ make sint

The resources utilized are:

| Resource | utilization
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 10 / 160
|BRAMs     | 0 / 16

To program the FPGA execute: 

    $ sudo iceprog reginit4.bin

In this **Youtube Video** you can see the flashing of the LEDs:

[![Click to see the youtube video](http://img.youtube.com/vi/dYikGANv1t4/0.jpg)](https://www.youtube.com/watch?v=dYikGANv1t4)

## Simulation
The testbench is a basic one, which instantiates the register init component, with 1 bit for the prescaler (so the simulation runs over less cycles). It runs the clock in a process, and initializes the simulation. 

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/reginit-6.png)

The verilog code is:

```verilog
//-- reginit_tb.v
module reginit_tb();
    
//-- Register for the clock
reg clk = 0;
    
//-- data from the component
wire [3:0] data;

//-- Instantiate the component with prescaler set to 1
reginit #(.NP(1))
  dut(
   .clk(clk),
   .data(data)
  );
    
//-- generate the clock
always #1 clk = ~clk;
    
//-- initialization process
initial begin
    
  //-- files for the results
  $dumpfile("reginit_tb.vcd");
  $dumpvars(0, reginit_tb);
    
  # 30 $display("FIN de la simulacion");
  $finish;
end
endmodule
```

Run the simulation with the command: 

    $ make sim

The results in gtkwave are:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T13-reg-init/images/T13-reginit-sim.png)

You can see how the register is loaded with its initial vallue 1100 and the values 0011 and 1100 are alternated. 

## Proposed exercise: 
* **Exercise 1**: Modify the initial values to get a different sequence
* **Exercise 2**: change the simulation to be self-checking

## Conclusions
TODO


