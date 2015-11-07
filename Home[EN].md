# Digital designs for FPGAs, with free open source tools

## Description

Tutorial of **synthetizable digital systems** design, using **ONLY** free and open source tools. Hardware description using **Verilog**.
* **Development board**: [iCEStick](http://www.latticesemi.com/icestick). iCE40HX-1k FPGA from Lattice
* **Verilog simulator**: [icarus Verilog](http://iverilog.icarus.com/) 
* **Signal visualizer**: [Gtkwave](http://gtkwave.sourceforge.net/)
* **Synthetizer**: [Yosys](http://www.clifford.at/yosys/) ([Github repo](https://github.com/cliffordwolf/yosys))
* **Place & route**: [Arachne-prn](https://github.com/cseed/arachne-pnr) (in github)
* **Utils and upload to FPGA**: [IceStorm project](http://www.clifford.at/icestorm/)

## TODO  
Why libre?  
Por donde íbamos? --> Retomar artículos sobre el tema  
--> convirtiendo hw en sw  
--> Hispalinux  
--> JCRA barcelona  

## Repo

[Verilog code repo](https://github.com/Obijuan/open-fpga-verilog-tutorial)

## Author
[Juan González Gómez](http://obijuan.github.io/) (Obijuan)

## Licence
<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T00-Intro/images/bq-logo-cc-sa-small-150px.png">

Licensed under a  [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)

## Credits
* [IceStorm project](http://www.clifford.at/icestorm/ ), by Clifford Wolf and Mathias Lasser
* [Arachne-pnr](https://github.com/cseed/arachne-pnr), by Cotton Seed 

## Special thanks
* This tutorial wouldn't be possible without the astounding free tools, result of the iCEStorm and Arachne-pnr projects, by **Clifford Wolf**, **Mathias Lasser** and **Cotton Seed**. They represent a bit step for humankind technologic heritage. Thank you!

* To [BQ](http://bq.com), for financing the material used in this tutorial. I'm truly privileged for being able to work in a company that encourages the maker spirit, and support research and experimentation initiatives with free technologies. Thank you!

* To **Jesús Arroyo**, for actually go buying the iCEStick boards and begin making the Debian/Ubuntu packaging of the IceStorm and Arachne-pnr projects' tools. Thank you! 

* To **David Sánchez Falero**, for teaching me how to highlight verilog code in github's markdown. :-) This made the code way more readable! Thank you! 

* To **Carlos García Saura**, for IceStorm installation instructions in Fedora.  Thank you!

## FAQs
* **What's the difference between an FPGA and an Arduino?**  
Arduino boards have a microcontroller, that we program. With an FPGA is not the same, you don't actually PROGRAM them, you configure them to implement ACTUAL HARDWARE on them. Is a tool for digital circuit design.

* **What does and FPGA that an Arduino won't?**  
A lot of applicatons can be made out of software (programming) or hardware. A counter, for example. The difference is that when you do it with pure hardware, they work way faster. Plus, hardware can be just "cloned" inside the board, and work in parallel. Having things happening simultaneously by softare is not a trivial thing.

* **Why Verilog and not VHDL?**  
Verilog is the language is supported by the [IceStorm project](http://www.clifford.at/icestorm/) for now. It is a free project, so the community will eventally add VHDL support. By today (sep-2015), Verilog is required to use free tools and FPGAs.
