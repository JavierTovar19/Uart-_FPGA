<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

[Examples of this chapter in github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T01-setbit)

## Introduction

The simplest digital circuit is just a cable connected to a "high" logic level. "1", for example. This way, if you connect a LED to it will light on (1) or turn off (0).

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-1.png)

The output of this circuit has been called "A".

## setbit.v: Hardware description.

In order to synthesize this circuit into the FPGA, we have to describe it first, using a hardware description language (HDL). In this tutorial we use Verilog, given that we have all the free tools for its simulation/synthesis.

Verilog is a language used for describing hardware..  but beware! IT IS NOT A PROGRAMMING LANGUAGE! It's a description language! It allows us to describe the connections and the elements of a digital system.

The Verilog code for this "hello world" circuit implementation can be found in the [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v) file. It looks like this:

```verilog
//-- Fichero setbit.v
module setbit(output A);
wire A;
    
assign A = 1;
    
endmodule
```

## Synthetization into the FPGA

In addition to the Verilog file, we'll need to specify which pin of the FPGA we want the component's "A" output to be connected to. This map is made with the setbit.pcf file, (pcf = Physical Constraint File). We will output the "A" signal through the pin 99, that is connected to the D1 LED; but it could be any other pin:

    set_io A 99

In this example, it only has one line, linking the "A" label to pin 99 of the FPGA.

Figure 2 graphically displays this idea. We are describing hardware, so making sketches and drawings is a good idea in order to understand our design better: 

![Imagen 2](https://raw.githubusercontent.com/Obijuan/open-fpga-verilog-tutorial/master/tutorial/T01-setbit/images/setbit-2.png)

To make the complete synthesis of the circuit, we got to the  _tutorial/T01-setbit_ folder, and we execute the **make sint** from the console:

    $ make sint
    
    yosys -p "synth_ice40 -blif setbit.blif" setbit.v
    
    /----------------------------------------------------------------------------\
    |                                                                            |
    |  yosys -- Yosys Open SYnthesis Suite                                       |
    |                                                                            |
    |  Copyright (C) 2012 - 2015  Clifford Wolf <clifford@clifford.at>           |
    |                                                                            |
    |  Permission to use, copy, modify, and/or distribute this software for any  |
    |  purpose with or without fee is hereby granted, provided that the above    |
    ...

A lot more messages will pop up. My laptop takes less than a second to synthesize. Final messages a look like this: 

    ...
    After placement:
    PIOs       1 / 96
    PLBs       1 / 160
    BRAMs      0 / 16
    
    place time 0.00s
    route...
    pass 1, 0 shared.
    route time 0.00s
    write_txt setbit.txt...
    #-- Generar binario final, listo para descargar en fgpa
    icepack setbit.txt setbit.bin
    $

When it finishes, it generates a "setbit.h" file. This file will be uploaded to the FPGA in order to configure it. We put the iCEStick into the USB and execute this command:

    $ sudo iceprog setbit.bin

The process last about 30 seconds (With the current iceprog version. This time will be reduce as the community improves the software). The messages that pop up are:

    [sudo] password for obijuan: 
    init..
    cdone: high
    reset..
    cdone: low
    flash ID: 0x20 0xBA 0x16 0x10 0x00 0x00 0x23 0x12 0x67 0x21 0x23 0x00 0x21 0x00 0x43 0x04 0x11 0x11 0x5F 0x7D
    file size: 32216
    erase 64kB sector at 0x000000..
    programming..
    reading..
    VERIFY OK
    cdone: high
    Bye.
    $

LED D1 on the iCEStick will turn on:

<img src="https://raw.githubusercontent.com/Obijuan/open-fpga-verilog-tutorial/master/tutorial/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

**Note:** The other LEDs may flicker. That is normal. It is because they have no signal assigned, and internally they are unconnected.

## Simulation

### Simulate first, sythesize later

The latter example it's been directly uploaded to the FPGA because the design was **PREVIOUSLY TESTED**. When we work with FPGA **we are actually making hardware** and we always have to be very careful. We could write a code that contains a short. And it could also happen that the tools didn't warn us, (specially in these first versions, still in an alpha stage). If we upload that circuit into the FPGA we could break it.

Because of that, we **ALWAYS SIMULATE THE CODE** that we write. Once we are sure enough that it works, (and it doesn't have a critical mistake) we upload it into the FPGA.

### Testing components: the testbench

If we buy a chip and we want to test it, what do we do?. Usually we weld it directly onto a PCB or we put it in a socket. But we can also plug it into a breadboard and place all the cables by ourselves.

In Verilog (and the rest of HDL languages) it's the same idea. **You can't simulate a verilog component right away**, you need to write a **testbench** that indicates which cables connect to which pins, which input signals to send the circuit, and check that the circuit outputs the right values. This testbench file is also written in Verilog.

How do we test the setbit component? It's a chip that has only one output pin, always high. In real life we would plug it onto a breadboard, we power it up, we would connect a cable to the output pin, and we would check it's high in voltage with a multimeter. We will do exactly the same, but with Verilog. We would get something like this:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-3.png)

The file is called setbit_tb.v. We always use the "_tb" suffix to indicate that it the file is a testbench and not a real piece of hardware. This testbench is NOT SYNTHESIZABLE, is a code FOR SIMULATION PURPOSES ONLY. That's why this time we can think of Verilog as a programming language, we are programming something that will "happen" to our circuit.

The testbench has 3 elements: the "setbit" component, a cable called "A" and a check block (The equivalent of a multimeter). This is the code:

    //-- Test bench module
    module setbit_tb;
    
    //-- Cable for connecting the output pin to setbit
    //-- We could give it ANY name, but we call it A (like the setbit pin)
    wire A;
    
    //-- Place the component (it's actually called "instance") and
    //-- connect the A cable to the A pin.
    setbit SB1 (
     .A (A)
    );
    
    //-- Begin the test (Checking block)
    initial begin
    
        //-- Define a file to dump all the data.
        $dumpfile("setbit_tb.vcd");
    
        //-- Dump all the data into that file when simulation finishes
        $dumpvars(0, setbit_tb);
    
        //-- After 10 time units, check if the cable is high.
        //-- In case of not being high, report the problem but
        //-- don't stop the simulation.
        # 10 if (A != 1)
               $display("---->¡ERROR! Output is not 1");
             else
               $display("Component ok!");

      //-- End simulation 10 time units after checking
      # 10 $finish;
    end
    endmodule

## ¡Simulating!

We use icarus verilog y GTKwave tools for simulation. We execute the command:

    $ make sim

and a window will pop up with the results of the simulation:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-simul-1.png" width="600" align="center">

At first glance, we can see it behaves as intended: the signal in A is always a "1". We can ignore the time units for now, they are seconds by default. After 20 units, simulation ends. 

The console will show the following messages:

    #-- Compilar
    iverilog setbit.v setbit_tb.v -o setbit_tb.out
    #-- Simular
    ./setbit_tb.out
    VCD info: dumpfile setbit_tb.vcd opened for output.
    Component ok!
    #-- Ver visualmente la simulacion con gtkwave
    gtkwave setbit_tb.vcd setbit_tb.gtkw &

The one that says "**Component ok!**" is the one we placed in the testbench when the output was "1".

## Proposed exercises

* Change the component so it outputs a 0. Upload it into the FPGA. Check that the LED is off now.

* Change the original example so setbit is output through the D2 LED (associated with pin 98 instead of 99)

## Conclusions
TODO