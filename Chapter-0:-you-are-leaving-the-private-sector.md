<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T00-Intro/images/checkpoint-charlie.png" 
width="400" align="center">

_You are leaving the private sector_ ... and you're entering the FREE sector! Welcome! From now on, **we'll only use tools that belong to technological heritage of humanity.**

# Introduction

[FPGAs](https://es.wikipedia.org/wiki/Field_Programmable_Gate_Array) are this kind of "blank" chips, that can be configured in order to create our own digital circuits. Yep! With FPGAs we are creating actual hardware! 

Every digital circuit can be split in it's basic elements: **logic gates** that perform boolean operations with the bits and **flip-flops** to store the results. As a first approximation, we can think of an FPGA as a chip that has unconnected  arrays of these elements. When you **configure** it, they connect to each other in an specific way and that's how we get our circuit.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/fpga-config1.png)

This configuration is achieved by **downloading to the FPGA** a binary file, called a **bitstream**, which contains all the information necessary to establish the connections between the internal elements of the FPGA. 

## Bitstream Generation

The magic of FPGAs is in the **software tools** that allow one to generate the bitstream from the **description of the circuit** in an HDL language. 

Circuits are designed using a **hardware description language** (HDL), such as Verilog or VHDL. These are the source files. The generation of the bitstream is done in two phases, from the sources:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/fpga-bitstream1.png)

**1 Synthesis**: The synthesis tool infers the basic hardware elements from its description, and obtains a file netlist that describes the connections between them. This phase does not depend on the FPGA to use. 

**2 Placed and routed**: The components of the netlist are mapped to the physical elements of the FPGA, their placement is determined and the routing is performed. All FPGA configuration information is condensed into the bitstream. This phase does depend on the particular FPGA model that is being targetted. 

**NOTE ON TERMINOLOGY**: Although technically the synthesis phase in only part of the generation of the bitstream, colloquially when speaking of synthesis we usually refer to the complete process. Thus, if we say that "we have synthesized this circuit for the FPGA", we are referring to the fact that all phases have been performed: synthesis, place and routing, bitstream generation and loading into the FPGA. 

## Captive Audience

FPGAs have been around for 30 years. They are tremendously useful tools with a lot of potential. They allow you to design your own chip! However, FPGA manufacturers have never released either the software source or the specifications of their bitstreams' formats.

This has meant that no one can create software to work with FPGAs, but can only use that of the manufacturer. And it can only be used on computers that the manufacturer allows you. You can only design what the manufacturer has thought can be designed with their tools. If you can think of something new, not supported by your software, you will not be able to do anything. All this is quite frustrating. And in the end, many people have stopped using FPGAs.


# Free tools for working with FPGAs

In May 2015 a historic milestone occurred: for the first time all the necessary tools were created to generate the bitstream from code in Verilog using only free software, thanks to the [icestorm project](http://www.clifford.at/icestorm/), led by Clifford Wolf. From that moment, we already have tools that belong to humanities technological heritage to work with FPGAs, and to be able to develop hardware using only tools of this heritage

## Advantages of using free tools

* **Autonomy**: Hardware developers can develop their systems independently of the manufacturer. Design no longer depends on the whims of each manufacturer, or their tastes. With the free tools we become independent. Designers decide which operating system to use, or what environment to use. We are no longer obliged to do what the manufacturer tells us.

* **Access to knowledge**: These tools can be used normally, just like the closed source ones. However, if we are curious, we have access to the knowledge of how they are programmed, what algorithms are used, how the synthesis is implemented ... This encourages the scientific spirit to understand how things work ... and then improve them. It is now possible for researchers around the world to analyze, understand and improve the algorithms. Before, only the manufacturers could do this

* **New applications**: It opens the way to test new uses of the FPGA not foreseen by the manufacturers. From the beginning, there have been ideas to use FPGAs as hardware on demand, co-designing hw / hw, operating systems that use hw tasks, etc. Although many theses have been written about it, the actual implementations were very specific to a particular manufacturer with little reproducible by the community. Now it is viable to make implementations that run for example on a raspberry pi, and to synthesize the hardware on demand. With the closed tools it was impossible, because it was not part of the manufacturers plan. 

* **Community Involvement**: Now ALL can participate in the evolution of FPGAs, not only by using them, but by growing and improving the tools themselves.

* **Reconfigurable Free Hardware Repositories**: The time has come to "reinvent the free wheel". It is already possible to create repositories of free HW designs that belong to us all and we can make them evolve over time. Share them. Improve them. These designs can be synthesized with free tools. And it is a knowledge that will last in time.

## Limitations

No newborn tools have everything we want. But by being free, potentially any feature can be implemented. That's why all free software/ hardware systems evolve and mature over time. Einstein was also a baby, and at that age he could not create his theories. The important thing is the potential.

The [project icestorm](http://www.clifford.at/icestorm/) tools have just been born. And they have yet to mature and develop. Some limitations (at the time of writing) are:

* Only for Lattice FPGAs , models: HX1K-TQ144 and HX8K-CT256 (more have since been added. see [project icestorm](http://www.clifford.at/icestorm/) for details)
* The tools only cover the low level: they are used in the command line. There is no graphical environment for managing projects. You have to do it based on makefiles. (tools have since been designed. see [ice studio](https://github.com/FPGAwars/icestudio) and [apio-ide](https://github.com/FPGAwars/apio-ide))
* The tristate output support is still very limited
* No post-routing time analysis support

## Project Icestorm tools
The free tools to work with the lattice FPGAs are the following:

* **Synthesizer** : [Yosys](http://www.clifford.at/yosys/) ([github](https://github.com/cliffordwolf/yosys))
* **Place & route** : [Arachne-pnr](https://github.com/cseed/arachne-pnr) (on github) 
* **Utilities and FPGA programmer**: [Project icestorm](http://www.clifford.at/icestorm/)

The following figure shows the different tools used in the steps, and the extensions of the files that are generated:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/icestorm-1.png)

It starts from the source files in verilog (.v). Using the Yosys synthesizer, the netlist (.blif) files are generated. The placement and routing is done with arachne-pnr, generating the bitstream in ascii (.txt) format. With icepack the binary bitstream (.bin) is created and finally sent to the FPGA with iceprog.

On the command line , the steps to take the file test.v to the FPGA would be:
```
$ yosys -p "synth_ice40 -blif test.blif" test.v
$ arachne-pnr -d 1k -p test.pcf test.blif -o test.txt
$ icepack test.txt test.bin
$ iceprog test.bin
```
## Free Simulation Tools
To design the circuits it is essential to have a verilog simulator. The free tools we will use are:

* **Verilog Simulator**: [Icarus Verilog](http://iverilog.icarus.com/) 
* **Waveform Viewer**: [Gtkwave](http://gtkwave.sourceforge.net/)

Icarus verilog creates an executable file from the Verilog code. When executed, the simulation is performed. The results are dumped into a .vcd file which is displayed with the Gtkwave tool. This allows us to inspect the signals to verify their correct operation

# Installation
## Ubuntu 14.04, 15.10
### Automatic installation via installer

David Cuartielles has created [this installer](https://github.com/dcuartielles/open-fpga-install) that performs the whole process automatically: it downloads all github tools, compiles them and installs them, as well as all necessary dependencies. The way to use it is:
```
git clone https://github.com/dcuartielles/open-fpga-install.git
cd open-fpga-install
sudo bash install.sh
```
### Manual Installation
Installation of dependencies :
```
sudo apt-get install build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev git mercurial graphviz  xdot pkg-config python python3 libftdi-dev
```
Installing IceStorm Tools (icepack, icebox, iceprog):
```
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
```
Installation of Arachne-PNR (the place & route tool):
```
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
```
Installation of Yosys (Verilog synthesis):
```
git clone https://github.com/cliffordwolf/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
```
Installing Icarus Verilog and GTKwave
```
sudo apt-get install gtkwave iverilog
```
## Fedora 22

Dependencies:
```
sudo dnf install libftdi-devel tcl-devel readline-devel flex clang bison gawk libffi-devel git mercurial graphviz python python3
```

IceStorm Tools installation (icepack, icebox, iceprog):
```
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
cd ..
```
**NOTE:** If you get errors related to "ftdi.h", you'll need to link the FTDI library this way:
```
sudo ln -s /usr/lib64/libftdi1.so /usr/local/lib/libftdi.so
sudo ln -s /usr/include/libftdi1/ftdi.h /usr/local/include/ftdi.h
```

Arachne-PNR installation (the place&route tool):
```
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
cd ..
```

Yosys installation (Verilog synthesis):
```
git clone https://github.com/cliffordwolf/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
cd ..
```

Icarus Verilog y GTKwave installation

```
sudo dnf install iverilog gtkwave
```

In order to execute "sudo iceprog" you'll need to link:

```
sudo ln -s /usr/local/bin/iceprog /usr/bin/iceprog
```

If you want to update the tools, you can re-use the repositories that you cloned during the installation, using the command 


```
"git reset --hard & git pull":

cd icestorm
git reset --hard & git pull
make -j$(nproc)
sudo make install

cd ../arachne-pnr
git reset --hard & git pull
make -j$(nproc)
sudo make install

cd ../yosys
git reset --hard & git pull
make -j$(nproc)
sudo make install
```
# ICEStick board
The board that we will use in these tutorials is the [IceStick](http://www.latticesemi.com/icestick) If you do not have it (or a similar one), No problem! We will also simulate all designs with icarus verilog and gtkwave

## Configuration
The download of the bitstream to the icestick plate is done directly by USB , using the library [libftdi](https://www.intra2net.com/en/developer/libftdi/). This requires access permissions .

If we try to load the icestick without permissions, we get the following error message :
```
$ iceprog scicad1.bin
init..
Can't find iCE FTDI USB device (vedor_id 0x0403, device_id 0x6010).
ABORT.
```
One way to solve this is to use sudo when running iceprog , downloading with the command:
```
$ sudo icprog bitstream.com
```
This has the drawback that you have to be reinsert the key every so often.

The other way is to configure the [udev](https://es.wikipedia.org/wiki/Udev) system so that when connecting the icestick to USB the user has permissions. To do this you have to do the following:

Create the /etc/udev/rules.d/80-icestick.rules file with the following content
```
ACTION=="add", SUBSYSTEM=="usb", ATTRS{idVendor}=="0403",
 ATTRS{idProduct}=="6010", OWNER="user", GROUP="dialout",
 MODE="0777"
```
Run this command to relaunch the [udev](https://es.wikipedia.org/wiki/Udev) administrator and load the new rule:
```
$ sudo udevadm control --reload-rules && sudo udevadm trigger 
```
Now you can do the download directly by running:
```
$ iceprog bitstream.bin
```
In ubuntu it is necessary to reboot the machine so that it works correctly

## How to do the tutorials
To perform these tutorials, and to be able to test all the examples, you have to follow the following steps:

Clone the repo of the tutorials , to have in your computer all the files:
```
$ git clone https://github.com/Obijuan/open-fpga-verilog-tutorial.git
```
Enter the working directory of the tutorial you play. For example, for 1 would be:
```
$ Cd open-fpga-verilog-tutorial / tutorial / ICESTICK / T01-setbit / `` `
```
Simulates and synthesizes
In each tutorial there will be commands to execute. To carry out the simulation you have to execute:
```
make sim
```
To synthesize :
```
make sint
```
And finally to configure the FPGA :
```
$ iceprog setbit.bin
```
## Documentation
* [ICEstick Handbook](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/icestickusermanual.pdf) (PDF)
* [FPGA Data Sheet iCE40LPHXF](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/iCE40LPHXFamilyDataSheet.pdf) (PDF)

## Pinout of the IceStick
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/icestick_pinout.png)

# Related Links
* [Project Icestorm](http://www.clifford.at/icestorm/) ([Github](https://github.com/cliffordwolf/yosys))
* [Clifford Wolf's personal page](http://www.clifford.at/)
* [Synthesizer Yosys](http://www.clifford.at/yosys/) ([Github](https://github.com/cliffordwolf/icestorm))
* [Place and route: arachne-pnr](https://github.com/cseed/arachne-pnr)
* [Gtkwave](http://gtkwave.sourceforge.net/)
* [Icarus Verilog](http://iverilog.icarus.com/) ([Github](https://github.com/steveicarus/iverilog))
# Credits
* "AND ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:AND_ANSI.svg#/media/File:AND_ANSI.svg
* "OR ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:OR_ANSI.svg#/media/File:OR_ANSI.svg
* "NOT ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:NOT_ANSI.svg#/media/File:NOT_ANSI.svg
* "XOR ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:XOR_ANSI.svg#/media/File:XOR_ANSI.svg
* "D-Type Flip-flop" by Inductiveload - Own Drawing in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:D-Type_Flip-flop.svg#/media/File:D-Type_Flip-flop.svg