![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T06-multiples-prescalers/images/mpres-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T06-multiples-prescalers)

## Introducción
With **hierarchical design** we can reuse tested components to build more complex designs. In this chapter we will flash 4 LEDs independently, each with its own prescaler.

  In the previous chapter we have created a **parametric prescaler**, which has the number of bits as a parameter. We will instantiate 5 prescalers, with different number of bits, to independently control the LEDs, and that each one can flash at a different frequency. Depending on the frequencies used, we can create different patterns in the LEDs.

  The architecture is shown in the diagram at the beginning of the chapter. There are **4 independent prescalers** that control each LED. Its input signals, instead of coming directly from the 12Mhz signal of the board, come in turn from another prescaler. This allows us to set **the base frequency of the LEDs**. Then, each prescaler divides this signal to get its own. This allows us to save bits, and make the prescalers have fewer bits.

## Hardware Description

The component is called **mpres.v** (multiple prescalers). It has 5 parameters, N0, N1, N2, N3 and N4 that correspond to the bit-width of each of the 5 prescalers. It has an input clock signal and 4 outputs to control the flashing of the 4 LEDs.

The hardware description is obtained directly from transcribing image 1 to Verilog code:

```verilog
//-- mpres.v
module mpres(input clk_in, output D1, output D2, output D3, output D4);
wire clk_in;
wire D1;
wire D2;
wire D3;
wire D4;
    
//-- Component parameters
//-- Bits for the different prescalers
//-- Change these values according to the desired LED sequence
parameter N0 = 21;  //-- Prescaler base
parameter N1 = 1;
parameter N2 = 2;
parameter N3 = 1;
parameter N4 = 2;
    
//-- Wire with base clock signal: the output of prescaler 0
wire clk_base;
    
//-- Base prescaler. Connected to the input clock signal
//-- Its output is for clk_base
//-- It's N0 bits wide
prescaler #(.N(N0))  
  Pres0(
   .clk_in(clk_in),
   .clk_out(clk_base)
  );
    
//-- Channel 1: Prescaler of N1 bits, connexted to LED 1
prescaler #(.N(N1))
  Pres1(
    .clk_in(clk_base),
    .clk_out(D1)
  );
    
//-- Channel 2: Prescaler of N2 bits, connexted to LED 2
prescaler #(.N(N2))
  Pres2(
    .clk_in(clk_base),
    .clk_out(D2)
  );
    
//-- Channel 3: Prescaler of N3 bits, connexted to LED 3
prescaler #(.N(N3))
  Pres3(
    .clk_in(clk_base),
    .clk_out(D3)
  );
    
//-- Channel 4: Prescaler of N4 bits, connexted to LED 4
prescaler #(.N(N4))
  Pres4(
    .clk_in(clk_base),
    .clk_out(D4)
  );
    
endmodule
```

First, we define the name of the component, with all its ports. Then 5 parameters are declared: the bit size of all the prescalers. Next, we declare the base clock signal that goes into the `clk_in` inputs of the 4 prescalers associated with the LEDs, and finally we instantiate each of the 5 prescalers, defining their size with each parameter and connecting to their ports the corresponding wires.

Different values can be set for the parameters. In this example, **21 bits** have been used for the **base prescaler**, which generates a **clock signal whose period is approx. 175ms**. **Prescalers 1 and 3** are **1 bit** so LEDs **D1** and **D3** will have a **period of 350ms**, while **prescalers 2 And 4** are 2 bits, with a **period of 800 ms**.

## Synthesis into the FPGA

The input clock signal is the 12Mhz clock of the iCEstick, which enters on pin 21 of the FPGA. The 4 output signals are output directly to the LEDs on pins 99 to 96.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T06-multiples-prescalers/images/mpres-1.png)

To synthesize, we execute:

    $ make sint

Resource usage is:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

To load it into the FPGA we execute:

    $ sudo iceprog mpres.bin

The LEDs will start to flash.

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T06-multiples-prescalers/images/T06-mpres-iCEstick-1.png" width="400" align="center">

In this video you can see the original sequence:

[![Click to see the youtube video](http://img.youtube.com/vi/Kldn76w58VY/0.jpg)](https://www.youtube.com/watch?v=Kldn76w58VY)

## Simulación

The test bench is very basic. We simply **instantiate the component mpres.v**, add a **clock generator** and **  **a block to start and end the simulation**. There is no explicit check of whether the component is working properly. It has to be visually checked by watching the simulation signals.

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T06-multiples-prescalers/images/mpres-2.png)

To make the simulation simpler, we take N0 = 1, so that the base prescaler is only 1 bit. The remaining prescalers are set to 1, 2, 3 and 4 bits respectively. The verilog code is as follows:

```verilog
//-- mpres_tb.v
module mpres_tb();
    
//-- Number of bits of the prescalers
parameter N0 = 1;
parameter N1 = 1;
parameter N2 = 2;
parameter N3 = 3;
parameter N4 = 4;
    
//-- Register to generate the clock signal
reg clk = 0;
    
//-- Output wires
wire D1, D2, D3, D4;
    
//-- Instantiate the component
mpres 
  //-- Establish parameters
  #(
     .N0(N0), 
     .N1(N1), 
     .N2(N2), 
     .N3(N3), 
     .N4(N4) 
  )
  //-- Connect the ports 
  dut(
    .clk_in(clk),
    .D1(D1),
    .D2(D2),
    .D3(D3),
    .D4(D4)
  );
    
//-- Clock generator. 2-unit period
always #1 clk = ~clk;
    
//-- Initialization block
initial begin
    
  //-- Fichero donde almacenar los resultados
  $dumpfile("mpres_tb.vcd");
  $dumpvars(0, mpres_tb);
      
  # 99 $display("FIN de la simulacion");
  # 100 $finish;
end
    
endmodule
```

The parameters can be changed for other simulations. To run the simulation, do:

    $ make sim

This is the result in gtkwave:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T06-multiples-prescalers/images/T06-mpres-sim-1.png)


## Proposed exercises
* Generar otra secuencia de movimiento cambiando  las frecuencias de los prescalers

## Conclusions
TODO