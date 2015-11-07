<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T00-Intro/images/checkpoint-charlie.png" 
width="400" align="center">

_You are leaving the privative sector_ ... and you're entering the FREE sector! Welcome! From now on, **we'll only use tools that belong to human heritage.**

# Introduction

[FPGAs](https://es.wikipedia.org/wiki/Field_Programmable_Gate_Array) are this kind of "blank" chips, that can be configured in order to create our own digital circuits. Yep! With FPGAs we are creating actual hardware! 

Every digital circuit can be split in it's basic elements: **logic gates** that perform boolean operations with the bits and **flip-flops** to store the results. As a first approach, we can think of an FPGA as a chip that has arrays of these elements, unconnected. When you configure it, they connect to each other in an specific way and that's how we get our circuit.

(drawing)

# Free environment
TODO

For sythesis and upload to board, we'll use [icestorm](http://www.clifford.at/icestorm/), yosys and aracne.
Environment: Ubuntu Linux 15.04

# Instalation

Fedora 22
--

Dependencioes:
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

# ICEStick board

This is the board that we'll use to try all the digital designs in this tutorial:
[IceStrick board](http://www.latticesemi.com/icestick)

If you can't get your hands on one of this boards, don't worry! we'll also simulate the circuits with icarus verilog and gtkwave

# Links
