<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

[Примеры для этой главы на github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T01-setbit)
## Перед тем как начать 
Не забывайте скачивать (клонировать) примеры учебника на свой компьютер, чтобы иметь доступ ко всем примерам:
```
https://github.com/Obijuan/open-fpga-verilog-tutorial.git
```
Войдите в каталог первого раздела учебника:
```
cd open-fpga-verilog-tutorial/tutorial/ICESTICK/T01-setbit/
```
Теперь вы готовы к упарженниям!

## Введение

Простейшая цифровая схема это просто провод подключенный к определенному, заранее известному, логическому уровню, например "1". Таким образом, если Вы подключите светодиод к нему, он будет светиться (если уровень "1") или не будет светиться (если уровень "0").

![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/setbit-1.png)

Выставляет уровень на выходе, который назовем "A".

## setbit.v: Описание в железе.

Для того, чтобы синетзировать эту схему в ПЛИС, нам нужно описать ее сначала используя язык аппаратного описания (hardware description language - далее HDL). В этом учебнике мы будем использовать Verilog, поскольку для него имеется свободный инструментарий для симуляции и синтеза.

Verilog это язык описания аппаратных схем. Обратите внимание! ЭТО НЕ ЯЗЫК ПРОГРАММИРОВАНИЯ! Это язык описания! Он позволяет нам описать соединения между компонентами цифровой схемы.

Код на Verilog для описания нашей схемы "hello world" содержится в файле [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v). Он выглядит вот так:

```verilog
//-- Fichero setbit.v
module setbit(output A);
wire A;
    
assign A = 1;
    
endmodule
```

## Синтез в ПЛИС

Дополнительно к файлу Verilog, нам так же необходима информация о том, какой физический контакт микросхемы (далее - "пин") будет подключен выходу "A". Связь описывается в файле setbit.pcf, (pcf = Physical Constraint File / Файл описания физических ограничений). Мы будем выводить сигнал "A" через пин (контакт) 99, который подключен в светодиоду D1; но это может быть вообще любой пин:

    set_io A 99

В данном примере, файл содержит только одну строку, подключая сигнал "A" к пину 99 ПЛИС.

Рисунок 2 графически отражает эту идею. Мы описываем электрические цепи, при этом рисуем электрическую схему, чтобы было понятно и наглядно: 

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/setbit-2.png)

Чтобы сделать синтез переходим в каталог _tutorial/T01-setbit_ folder, и запускаем команду **make sint** из командной строки:

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

Много сообщений появляется в консоле. Мой компьютер успевает синтезировать этот пример менее чем за секунду. В конце Вы получите примерно такие сообщения: 

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

После завершения будет создан файл "setbit.bin". Этот файл, представляющий из себя конфигурацию ПЛИС, уже может быть загружен в ПЛИС. Подключим iCEStick к порту USB и выполним следующую команду:

    $ sudo iceprog setbit.bin

**NOTE**: Если ICEStick настроен так, как описано в главе 0, загрузка конфигурации может осуществляться без sudo. 

Процесс занимает около 3 секунд, при этом появляются примерно такие сообщения:

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

Светодиод D1 на iCEStick загорается:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

**Замечание:** Остальные светодиоды могут едва светиться. Это нормально, поскольку пины, к которым они подключены не указаны в конфигурации, никакие сигналы на них не назначены и подтягивающие (pull-up) резисторы для них активны поумолчанию. 

## Симуляция

### Сначала симулируем, затем синтезируем

In the previous example we directly uploaded to the FPGA because the design was **PREVIOUSLY TESTED**. When we work with FPGAs **we are actually making hardware** and we always have to be very careful. We could write a design that contains a short circuit. And it could also happen that the tools don't warn us, (especially in these first versions, still in an alpha stage). If we upload that circuit into the FPGA we could break it.

Because of that, we **ALWAYS SIMULATE THE CODE** that we write. Once we are sure enough that it works, (and it doesn't have a critical mistake) we upload it into the FPGA.

### Testing components: the testbench

If we buy a chip and we want to test it, what do we do? Usually we solder it directly onto a PCB or we put it in a socket. But we can also plug it into a breadboard and place all the wires by ourselves.

In Verilog (and the rest of HDL languages) it's the same idea. **You can't simulate a verilog component right away**, you need to write a **testbench** that indicates which cables connect to which pins, which input signals to send the circuit, and check that the circuit outputs the right values. This testbench file is also written in Verilog.

How do we test the setbit component? It's a chip that has only one output pin, always high. In real life we would plug it onto a breadboard, we power it up, we would connect a cable to the output pin, and we would check it's high in voltage with a multimeter. We will do exactly the same, but with Verilog. We would get something like this:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/setbit-3.png)

The file is called setbit_tb.v. We always use the "_tb" suffix to indicate that it the file is a testbench and not a real piece of hardware. This testbench is NOT SYNTHESIZABLE, is a code FOR SIMULATION PURPOSES ONLY. That's why this time we can think of Verilog as a programming language, we are programming something that will "happen" to our circuit.

The testbench has 3 elements: the "setbit" component, a cable called "A" and a check block (The equivalent of a multimeter). This is the code:
```verilog
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
```
## Simulating!

We use icarus verilog and GTKwave tools for simulation. We execute the command:

    $ make sim

and a window will pop up with the results of the simulation:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/T01-setbit-simul-1.png" width="600" align="center">

At a glance we can see it behaves as intended: the signal in A is always a "1". We can ignore the time units for now, they are seconds by default. After 20 units, simulation ends. 

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
We have created our first circuit in verilog , a "hello world" consisting of 1 wire. We have synthesized and loaded it into the FPGA. We have also written one test bench in verilog to simulate it. We already have all the tools installed, configured and we have verified that they work. We are ready to tackle more complex designs and learn more verilog

### Introduced concepts:

* pin connections are assigned
* Creating components with module
* Definition of output ports with output
* Definition of a wire with wire
* testbench