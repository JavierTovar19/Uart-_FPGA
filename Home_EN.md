![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/tutorial-fpga-logo.png)
# Digital Design for FPGAs, with free tools

## Description

Tutorial of **synthesizable digital systems** design in FPGAs using ONLY free and open source tools. The Hardware descriptions in this tutorial will be in **Verilog**. 

* **Development board** : [iCEstick](http://www.latticesemi.com/icestick) . With the Lattice iCE40HX-1k FPGA
* **Verilog Simulator** : [Icarus Verilog](http://iverilog.icarus.com/) 
* **Signal Viewer** : [Gtkwave](http://gtkwave.sourceforge.net/)
* **Synthesizer** : [Yosys](http://www.clifford.at/yosys/) ([Repository on github](https://github.com/cliffordwolf/yosys))
* **Place & route** :  [Arachne-pnr](https://github.com/cseed/arachne-pnr) (on github)
* **FPGA Utilities and Download** : [Project icestorm](http://www.clifford.at/icestorm/)

It is an **incremental tutorial** , in which concepts are introduced progressively, while being tested on real hardware. The circuits presented initially are not designed according to the usual design rules, but rather simplicity is emphasized. Then little by little, as the tutorial progresses, they are refined and improved.

The journey begins with the simplest circuit possible: a wire. And it ends with the design of **a very simple microprocessor** : MICROBIO

## Community: FPGA-WARS
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/FPGA-wars-logo-small.png)  

Join the [FPGA-WARS group : exploring the free side](http://fpgawars.github.io/) for the latest news, questions, share and collaborate on the development of free reconfigurable hardware. 

## Repository

[Repository with verilog code](https://github.com/Obijuan/open-fpga-verilog-tutorial)

## Author

[Juan González Gómez](http://obijuan.github.io/) (Obijuan)

## License
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/attribution-share-alike-creative-commons-license.png)  

Licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)

## Credits

* [Project icestorm](http://www.clifford.at/icestorm/), by Clifford Wolf and Mathias Lasser
* [Arachne-pnr](https://github.com/cseed/arachne-pnr), by Cotton Seed
* [BQ](http://bq.com) , for supporting this project from 2015-07-01 to 2016-04-14

## Thanks

* This tutorial would not have been possible without the incredible free tools of the Icestorm and Arachne-pnr project, by **Clifford Wolf** , **Mathias Lasser** and **Cotton Seed** . They represent a spectacular advance for the technological heritage of mankind. Thank you very much!
* To [BQ](http://bq.com) , to finance the material used in the tutorial. Thank you very much!
* To **Jesús Arroyo** , for the incredible tools: icestudio and celery you are developing to make FPGAs easier to use Thank you very much!
* To **David Sánchez Falero** , for teaching me how to highlight Verilog code in the github markdown :-) This has made the code much more readable! Thank you so much!
* To **Carlos García Saura** , for the installation instructions of icestorm tools in Fedora. Thank you very much!
* To **Javibc (J Bc)** , by the instructions to relaunch the udev when configuring the icestick ¡Muchas Gracias!
* To **David Cuartielles** by the installer of free tools and makefile examples for the board with the FPGA ICE40HX8K. Thank you very much!
* To **Fernando Barcala** (Barkalez) for his feedback on the problems doing the tutorial 1. This has improved the documentation. Thank you!
* To **Alberto Piganti** (pighixxx) for doing the pinout of the icestick. Thank you!
* To **Cristóbal Bueno** for his tests of icestorm toolchains for Linux-32bits. Thank you!
* To **Sebastián Gallardo** For the test of the libusb in Windows 32-bits. Thank you!
* To **Carlos Diaz** for the tests of the libusb in Windows 10 64-bits. Thank you!
* To **Pablo Cisneros** (ZioGuillo) by the synthesis / download instructions in MAC. Thank you!
* To **Julian Caro** for all the issues sent to improve this tutorial. Graciass!

## FAQs

* **What is the difference between an FPGA and Arduino?** 
Arduino's are a microcontroller, which are programmed. FPGAs are NOT PROGRAMMED, instead, they are configured to implement HARDWARE and are used to design circuits.

* **What does an FPGA do that an Arduino can't do?** 
Many applications can be done either by software (programming) or by hardware. For example a counter. The difference is that in designing with hardware we get much higher speeds. In addition, the hardware can be "cloned" directly and operates in parallel. Whereas having several things running in parallel in software is not trivial.

* **Why Verilog and not VHDL?** 
Verilog is the language that is supported by the [icestorm project](http://www.clifford.at/icestorm/) at the moment. But as it is a free project, the community will expand it and the VHDL will be supported sooner or later. As of today (Sep-2015), to work with free tools and FPGAs, you have to use Verilog

**Update (August-2016)** : The [Yodl Project](https://github.com/forflo/yodl) is implementing VHDL for the free Yosys synthesizer. It's still green, but VHDL support is on the way :-)

* **Where can I buy the IceStick** 
-[Icestick at Mouser ](http://www.mouser.es/new/Lattice-Semiconductor/lattice-icestick-kit/)
-[Icestick en LatticeUSA](http://www.latticesemi.com/icestick)


* **I have done the Hello World tutorial but synthesizing the file does not generate a .bin , instead it's a .blif**
The .blif file is what is generated in the synthesis, after running the yosys program. However, to get the .bin you still have to execute more commands. All the commands to execute to get the .bin from .v in the first tutorial are:
```
$ yosys -p "synth_ice40 -blif setbit.blif" setbit.v
$ arachne-pnr -d 1k -p setbit.pcf setbit.blif -o setbit.txt
$ icepack setbit.txt setbit.bin
```
To make it easier, it is best to run the **make sint** command . To do this you have to clone the repository of the github tutorial, enter the working directory and execute make sint:
```
$ git clone https://github.com/Obijuan/open-fpga-verilog-tutorial.git
$ cd open-fpga-verilog-tutorial/tutorial/ICESTICK/T01-setbit/
$ make sim
```
## NEWS

**2016/12/29** : Clifford adds an [example hello world for the Icezum Alhambra Alhambra](https://github.com/cliffordwolf/icestorm/tree/master/examples/icezum) inside the Icestorm project.

**2016/12/28** : Clifford Wolf already has his Icezum Alhambra !. He received it from Ismael Olea, 33C3 in Hamburg. All our recognition of Clifford's work. And many thanks to Ismael for giving it to him :-)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/Clifford-wolf-Ismael-Olea-icezum-alhambra.jpeg)

**2016/12/21** :

 * Delivered the **100 Icezum Alhambras** (v1.1) of the first Lot! :-) #ElAlhambraRepairer can rest now
 * **Julian Caro** makes this [epic video](https://www.youtube.com/watch?v=ImtLt6a4EI4) ( [More information](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/_HGMov0MS5w/Nem_c3pkEAAJ) )

**2016/12/10** : Activities in the **Ourense MakersLab 2016** :

 * Chat: " **Free FPGAs** ": ( [More information](https://github.com/Obijuan/myslides/wiki/2016_11_18:-Maker-faire-Bilbao,-FPGAs-Libres) )
 * Delivered the **III Workshop of Free FPGAs** ( [More information](https://github.com/FPGAwars/workshops/wiki/2016_12_10:-Ourense-MakersLab) )

**2016/11/18** : Talk: " **Free FPGAs** " in the **Bilbao MakerFaire** ( [More information](https://github.com/Obijuan/myslides/wiki/2016_11_18:-Maker-faire-Bilbao,-FPGAs-Libres) )

**2016/11/09** : Talk: " **Free FPGAs** " at the **Rey Juan Carlos University**, fuenlabrada campus ( [More information](https://github.com/Obijuan/myslides/wiki/2016_11_09:-URJC,-Fuenlabrada,-Madrid,-FPGAs-Libres) )

**2016/11/05** : Activities at **OSHWDEM 2016 of A Coruña** , at the Domus Museum
 * Chat: " **Free FPGAs** " ( [More information](https://github.com/Obijuan/myslides/wiki/2016_11_18:-Maker-faire-Bilbao,-FPGAs-Libres) )
 * Delivered the **II workshop of Free FPGAs** in Spain ( [More information](https://github.com/FPGAwars/workshops/wiki/2016_11_05:-OSHWDem16-A-Coru%C3%B1a) )

**2016/10/30** : Launched the **Icezum Alhambra Peregrina**, so that everyone can freeze with free FPGAs ( [More information](https://groups.google.com/forum/#!msg/fpga-wars-explorando-el-lado-libre/DsyM8_VJm5U/dJXNHKtEAAAJ) ) ( [Google Docs](https://docs.google.com/spreadsheets/d/11Riyfs-KfIxmTNFPu6D26Ne3IdXgFJP3_fDqvvpL_3c/edit?usp=sharing) )

**2016/10/28** : Organized the **1st workshop on free FPGAs in Spain** . Taught at the Technical School of Industrial Engineering, UPM ( [More information](https://github.com/FPGAwars/workshops/wiki/2016_10_28:-Reset-ETSII-UPM )

**2016/10/24** : Talk: " **Free FPGAs** ", given in THE EVENT 2016, at the University Carlos III of Madrid (UC3M) ( More [information](https://github.com/Obijuan/myslides/wiki/2016_10_24:-El-Evento-2016,-UC3M.-FPGAs-libres) )

**2016/10/20** : The **Alhambra-led card** is published: Peripheral with an LED, for the Icezum Alhambra, with printable PCB ( [More information](https://github.com/FPGAwars/alhambra-led/wiki))

**2016/10/18** : Podcast 97: [Obijuan, free software to join you](http://programarfacil.com/podcast/software-libre-hardware-libre-clone-wars/). Interview with obijuan in the Podcast of **Programarfacil.com** , by **Luis del Valle**

**2016/10/02** : Clifford **announces** that it will begin **reverse** engineering the bitstream of **Xilinx 7 Series FPGAs** . It will take more than 1 year, but it is a very important milestone to increase the free FPGAs park ( [More information](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/RWrDKMzHFdw/3iwhS5XhBgAJ) )

**2016/09/24** : Talk: "FPGAwars: Exploring the free side of FPGAs" given at **Madrid Mini maker faire** ( [More information](https://github.com/Obijuan/myslides/wiki/2016_09_24-Madrid-Maker-faire:FPGAwars-explorando-el-lado-libre) )

**2016/09/23** : First talk given in Spain on **free FPGAs**. It was given at the ETSIIT of the University of Granada, organized by the **Free Software Office of the University of Granada** ( [More information](https://github.com/Obijuan/myslides/wiki/2016_09_23-Granada-Geek-FPGAs-Libres) )

**2016/08/23** : Videoblog 23: **ACC: Apollo CPU Core**. Announcement of the ACC project: implementation of a core core of the Apollo CPU in Verilog for free FPGAs ( [VIDEO](https://www.youtube.com/watch?v=2PzKB2mfwoU&index=23&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30) ) ( [wiki](https://github.com/Obijuan/ACC/wiki) )

**2016/07/29** : Videoblog 22: **Icezum Alhambra V1.0K** World Heritage Edition ( [VIDEO](https://www.youtube.com/watch?v=d-9dIAo4FVw&index=22&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30 ) )

**2016/07/28** : [Icezum Alhambra V1.0K](https://github.com/FPGAwars/icezum/wiki) released! It has been created from scratch using only free tools, from our technological heritage

**2016/07/07** : **VHDL** frontend for Yosys! Still green, but the community is already starting **VHDL support on free FPGAs** : https://github.com/forflo/yodl

**2016/07/03** : Finished the phase of raising money for the crowdfunding of the manufacture of the **Alhambra Icezum**. We have already received the **€ 6,500** from the funders. Thank you very much to all of you for your help! [List of all funders](https://docs.google.com/spreadsheets/d/1yngU6kfMr5JWJYXrxBMJ7l1Cai2iuucMDoND4NpgU5Y)

**2016/07/02** : Jesús Arroyo releases [version 0.2.0-Beta1](https://github.com/FPGAwars/icestudio/releases/tag/0.2.0-beta1) of Icestudio. Publish [this video showing the most relevant new features](https://www.youtube.com/watch?v=mAIKb47z2Do) : code blocks and hierarchical design. **Clifford Wolf** echoes the news and tries it on a icestick :-) [twitter post](https://twitter.com/oe1cxw/status/748838491010830336)

**2016/05/30** : The **Icezum** Alhambra schematic has been **migrated to [Kicad](http://kicad-pcb.org/)** , a free electronic design tool ( [repo](https://github.com/FPGAwars/icezum/tree/master/src-kicad) )

**2016/05/19** : The **pinout of Icezum Alhambra** by Maestro Piganti has been migrated to **SVG format** with Inkscape so that anyone can modify or use it easily ( [repo](https://github.com/FPGAwars/icezum/tree/master/doc/pinout )

**2016/05/18** : [Objective achieved](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/oLFzYPqCOcQ/luBcnXpBBwAJ) . The 100 Icezum Alhambras are now in full swing. We started the process for the manufacture of the first 100 units!

**2016-05-04** : We already have the [pinout of the card Icezum Alhambra](http://www.pighixxx.com/test/2016/05/icezum-pinout/) , made by Alberto Piganti. Thanks!

**2016-05-01** : Help is **requested** on the **list of FPGA-wars** for the financing of the **manufacture of 100 cards Icezum Alhambra**. A mini-crowdfunding ( [Original message in the group](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/oLFzYPqCOcQ/OsMxYKnuAQAJ) )

**2016-04-29** : [Platformio 2.9.0](https://community.platformio.org/t/big-update-platformio-ide-1-2-0-and-platformio-cli-2-9-0/202) already supports free FPGAs :-)

**2016-04-20** : [FPGAwars organization in github](https://github.com/FPGAwars). All repos related to free fpgas have been transferred to that organization. BQ no longer continues with free FPGAs

**2016-03-24** : Cross-compilation of icestorm tools for **Ubuntu Phone** ( [VIDEO](https://www.youtube.com/watch?v=xZG6to8HW7I&index=19&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30) )

**2016-03-22** : [Document with the synthesis instructions for Mac](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/tutorial-mac/apple-mac-capitan.pdf) , created by **Pablo Cisneros** (ZioGuillo). Thank you!

**2016-03-15** : Locomotion of a worm-like **modular robot** using **hardware oscillators in a FPGA**, with the IceZUM card ( [VIDEO](https://www.youtube.com/watch?v=6I5Z70eewrg) )

**2016-03-12** : Videoblog on the IceZUM Alhambra ( [VIDEO](https://www.youtube.com/watch?v=LC8nucTiswM&index=17&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30) )

**2016-03-09** : [Post on Hackaday](http://hackaday.com/2016/03/09/does-the-world-need-an-fpga-arduino/) on the IceZUM Alhambra.

**2016-03-08** : We already have 5 prototypes of the [IceZUM Alhambra](https://github.com/FPGAwars/icezum/wiki) running! ( [Image](https://raw.githubusercontent.com/FPGAwars/icezum/master/doc/IceZUM-rev1-1607-img4.jpg) ). We have done some tests that you can see in [these videos](https://github.com/FPGAwars/icezum/wiki#videos)

**2016-03-05** : **First downloaded bitstream on the IceZUM Alhambra card!** :-) The board works. In addition, the tests have been done from Windows, using [icestorm MULTIPLATAFORMA](https://github.com/FPGAwars/toolchain-icestorm/releases/tag/0.7) Double validation! ;-) ( [Image](https://github.com/FPGAwars/icezum/raw/master/doc/2016-03-04-Mounting-first-prototype/icezum-alhambra-mounting-15.jpg) )

**2016-03-04** : First version of [icestorm MULTIPLATAFORMA](https://github.com/FPGAwars/toolchain-icestorm/releases/tag/0.7) . You already have cross-compiled tools for Mac and Windows. Initial tests ok

**2016-03-03** : We received the first PCBs from the [Icezum Alhambra](https://github.com/FPGAwars/icezum/wiki) . They are in Pinos del Valle, in Granada ( [Image](https://github.com/FPGAwars/icezum/raw/master/doc/Icezum-Alhambra-pcb-1.jpg) )

**2016-02-29** : The version of the [icezum card](https://github.com/FPGAwars/icezum) that is coming has been christened **Icezum Alhambra**. The name was **suggested by Sebastian Gallardo** in [this post from the FPGA-wars list](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/f1W0Vtt5NdE/LEDRSXudGwAJ)

**2016-02-28** : **Cross-compilation** of the iceprog **tool for windows**. **Cristóbal Bueno** has reported [the first tests of correct operation in Windows 10 64-bit](https://github.com/FPGAwars/toolchain-icestorm/wiki#more-tests)

**2016-02-26** : Clifford launches [version 0.6](https://github.com/cliffordwolf/yosys/releases/tag/yosys-0.6) of Yosys ( [Download](http://www.clifford.at/yosys/download.html) ). Soon it will be packaged for ubuntu and its installation will be trivial

**2016-02-25** : **Cross-compilation** of icestorm tools for **Linux-32-bit** . The packages are generated and can be installed directly from [Apio](https://github.com/FPGAwars/apio). **Cristóbal Bueno** , from the FPGA-wars list, is in charge of making the tests. Thank you!

**2016-02-23** : [Icestudio is published in hackaday](http://hackaday.com/2016/02/23/icestudio-an-open-source-graphical-fgpa-tool/) :-)

**2016-02-19** : [Icezum card](https://github.com/FPGAwars/icezum) published in github. Designed by **Eladio Delgado** . Yesterday they were sent to manufacture the first 5 pcbs. It is a coach card like the icestick but shaped like BUM ZUM, which is in turn as an Arduino one

**2016-02-08** : **Alberto Piganti** (pighixxx) has done the [pinout drawing of the icestick](http://www.pighixxx.com/test/2016/02/icestick-pinout/)

**2016-02-06** : [Videoblog 15: Visual hardware programming in free FPGAs](https://www.youtube.com/watch?v=3nnQ7VTrSLo&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30&index=15)

**2016-02-04** : More **improvements** in [icestudio](https://github.com/Jesus89/icestudio) , by **Jesús Arroyo** : ( [VIDEO](https://www.youtube.com/watch?v=lIjSNTmHkQU) ) ( [VIDEO](https://www.youtube.com/watch?v=RLr2cC8dgOg) )

**2016-02-02** : First test of [icestudio](https://github.com/Jesus89/icestudio) , designed by **Jesús Arroyo** : Graphic logic blocks converted to json, from there to verilog, and synthesized and loaded in the FPGA, through platformio: ( [VIDEO](https://www.youtube.com/watch?v=lIjSNTmHkQU) )

**2016-01-31** : First **platformio + FPGA** model , used since Atom ( [VIDEO](https://www.youtube.com/watch?v=a_m_Nx66eOs&index=14&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30) ). Developing hardware now is a lot easier. You no longer need to use the console and make ;-)

**2016-01-28** : Starting to support lattice FPGA ice40 in **Platformio** : [Platformio + FPGA](https://github.com/FPGAwars/Platformio-FPGA/wiki/Platformio-FPGA-wiki-home)

**2016-01-24** : Simplez documenting and with improvements: ( [VIDEO](https://www.youtube.com/watch?v=_3sWETbaDig) )

**2016-01-12** : Jesús Arroyo **replicates** the [Clifford examples from the 32CCC](http://www.clifford.at/papers/2015/icestorm-flow/) in our icoboard, and creates new examples that go up to [this repository in github](https://github.com/Jesus89/picorv32-c-examples)

**2016-01-11** : We received the icoboard [card donated](http://icoboard.org/) to us at 32CCC, which was attended by Victor Diaz. Thank you very much! :-)

**2016-01-02** : David Cuartielles [creates an installer](https://github.com/dcuartielles/open-fpga-install) to facilitate the installation of all free tools to work with FPGA (on Ubuntu / Debian systems)

**2015-12-31** : [Pull request from David Cuartielles](https://github.com/Obijuan/open-fpga-verilog-tutorial/pull/3) to repo of the tutorial. Added a new makefile to support [the ICE40HX8K lattice board](http://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/iCE40HX8KBreakoutBoard.aspx).

**2015-12-30** : We received two [ICE40HX8K lattice board cards](http://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/iCE40HX8KBreakoutBoard.aspx), purchased at [MOUSER](http://www.mouser.es/ProductDetail/Lattice/ICE40HX8K-B-EVN/?qs=%2fha2pyFadugG1hxFZcqbAJG8jgdGBbaecmBJrQx0SABRTeEI3HN%252bIw%3d%3d). They have the FPGA HX8K lattice, with 8 times the size of the HX1K of the icestick. In this can already be synthesized a 32-bit risc-v processor (not yet tested)

**2015-12-27** : Clifford gives the lecture ["A Free and Open Source Verilog-to-Bitstream Flow for iCE40 FPGAs"](https://media.ccc.de/v/32c3-7139-a_free_and_open_source_verilog-to-bitstream_flow_for_ice40_fpgas#video) in the 32nd edition of the Chaos Communication Congress in Hamburg. [All material is published on his website](http://www.clifford.at/papers/2015/icestorm-flow/). In the demo they use an HX8K FPGA on the [icoboard](http://icoboard.org/) with a 32-bit risc-v processor synthesized using free tools. Víctor Díaz who is there knows Clifford, and this one gives him an icoboard to give it to obijuan :-)

**2015-12-08** : Found [**the solution to a bug in iceprog**](https://github.com/cliffordwolf/icestorm/issues/16): Now download times on the icestick are always less than 3 seconds !!! Clifford integrates the official repo almost instantly. The development cycle is now much faster

**2015-12-04** : The [FPGA-WARS group](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre) is created . It is [officially announced on clonewars](https://groups.google.com/d/msg/asrob-uc3m-impresoras-3d/Lv6Tjb_3js8/_C4ZTISADAAJ)

**2015-12-01** : Dinner of David Cuartielles of Arduino.cc and Rodrigo of the Prado and Juan González of BQ, in Madrid. We talked to David about the new ecosystem of free FPGA tools and the possibility of collaborating together

**2015-11-26** : First version of the SIMPLEZ educational **processor** synthesized in the icestick, executing a program that shows a sequence by the leds: [Repo](https://github.com/Obijuan/simplez-fpga)

## History

**2015-11-20** : Finished the last chapter of the tutorial (30)

**2015-10-21** : **Imperial March** plays on the icestick. I'm sending [this video to Clifford](https://www.youtube.com/watch?v=IDU8kHAaLTw). I also tell him that I am working on these tutorials. He likes the idea and **publishes the link of the tutorials on the icestorm website** :-)

**2015-08-12** : First contact with Clifford Wolf (the author of the icestorm project). I send him an email thanking him for the project. He answers.

**2015-08-10** : Returned to Madrid, although still on vacation. I passed by BQ to pick up an icestick. I achieved a historic milestone: I downloaded my first verilog design to the fpga, using free tools !!! A "hello world"" program that lights a led !!!!! :-) I start this wiki to do the tutorials as I'm learning. 

**2015-08-05** : The two icestick packages are received at BQ (And I on vacation with the SAV butt !!!!)

**2015-08-03** : **Jesús Arroyo** buys 2 icesticks cards directly from Lattice, for the department of innovation and robotics of BQ

**2015-07-31** : First tests. Installed the entire tool-chain of icestorm web in a ubuntu 15.04. It works!. All test examples I upload to [this repo](https://github.com/Obijuan/mytests/tree/master/verilog) . The SAV shoots me. I'm going on vacation (no internet for 10 days). I take PDFs with verilog manuals to read and learn

**2015-07-30** : I finish the [second season of the Freecad tutorials](http://www.iearobotics.com/wiki/index.php?title=Tutorial_Freecad._Temporada_2). I can already use it with FPGAs!

**2015-07-04** : I begin to write the [Notes on free FPGAs](http://www.iearobotics.com/wiki/index.php?title=Obijuan:Notas_sobre_FPGAs_libres) in the wiki, to gather information. I still can not evaluate anything because first I have to finish [the Freecad season 2 tutorials](http://www.iearobotics.com/wiki/index.php?title=Tutorial_Freecad._Temporada_2). Until I finish one thing I can not start with the next

**2015-07-03** : Another [post in hack-a-day](http://hackaday.com/2015/07/03/hackaday-prize-entry-they-make-fpgas-that-small/) . I get shot SAV (live anxiety syndrome)

**2015-05-29** : The news comes out in a [hack-a-day post](http://hackaday.com/2015/05/29/an-open-source-toolchain-for-ice40-fpgas/). I learn of its existence through this post and I begin to evaluate the tools (obijuan)

**2015-05-27** : The [icestorm project](http://www.clifford.at/icestorm/) reaches the first big milestone: They already have a workflow that works completely using only free tools

## Compilation of Links

* [Project icestorm](http://www.clifford.at/icestorm/) ( [Github](https://github.com/cliffordwolf/icestorm) ). A project to encompass all the low level iCE tools
* [Yosys: synthesis tool](http://www.clifford.at/yosys/) ( [Github](https://github.com/cliffordwolf/yosys) )
* [Arachne: Place & route](https://github.com/cseed/arachne-pnr)
* [Icarus Verilog Simulator](http://iverilog.icarus.com/) ( [Github](https://github.com/steveicarus/iverilog) )
* [Gtkwave](http://gtkwave.sourceforge.net/) : Waveform display
* [Icestick board](http://www.latticesemi.com/icestick)
* [ICE40LPHXFamilyDataSheet.pdf](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/iCE40LPHXFamilyDataSheet.pdf) : ICE40 FPGA datasheet
* [Icestickusermanual.pdf](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/icestickusermanual.pdf) : Icestick data sheet
* [Pinout of the Icestick board](http://www.pighixxx.com/test/2016/02/icestick-pinout/) by Alberto Piganti
* [Processor Simplez-F](https://github.com/Obijuan/simplez-fpga/wiki/Procesador-SIMPLEZ-F). Educational processor Simplez, by Professor Gregorio Fernández, written in Verilog and synthesized into an Icestick
* [Simplez on Atom](https://github.com/Obijuan/simplez-grammar) . Package to highlight the Simplez syntax from Atom
* [FPGA-WARS Group](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre) : Exploring the Free Side
* [Platformio + FPGA](https://github.com/FPGAwars/Platformio-FPGA/wiki/Platformio-FPGA-wiki-home) . Inclusion of ICE40 FPGAs in the Platformio environment
* [Apio](https://github.com/FPGAwars/apio) . Multiplatform ecosystem to synthesize hardware in free FPGAs
* [Icestudio](https://github.com/FPGAwars/icestudio) . Visual tool to synthesize hardware in free FPGAs
* [IceZUM Alhambra Card](https://github.com/FPGAwars/icezum/wiki)
## Tools used for the preparation of this tutorial

[Inkscape](https://inkscape.org/es/), 0.91: For the figures (All of the figures source files in .SVG are in the repository)

[Gimp](https://www.gimp.org/): Retouching and processing of photos

**Operating System** : Ubuntu GNU / Linux 16.04
## Bibliography

* "**Digital Design, An Embedded Systems Approach Using Verilog**". P. Ashenden. 2007. It is a reference book of digital design using hdl languages

[Next](https://github.com/Obijuan/open-fpga-verilog-tutorial/wiki/Cap%C3%ADtulo-0%3A-you-are-leaving-the-privative-sector)