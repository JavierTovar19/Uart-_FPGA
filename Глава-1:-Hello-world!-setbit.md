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

В предидущем примере мы сразу загрузили пример в ПЛИС. Мы так сделали, поскольку этот пример уже был **заранее протестирован**. Когда мы работаем с ПЛИС - мы создаем электрические схемы, поэтому всегда должны быть внимательны! Мы можем описать такую схему, в которой получится короткое замыкание и утилиты могут об этом не предупредить (особенно в ранних версиях средств разработки). Если мы загрузим такую схему в ПЛИС, мы, вероятнее всего, повредим ПЛИС.

Поэтому мы **ВСЕГДА СИМУЛИРУЕМ НАПИСАННЫЙ КОД**. Как только мы убедились что в симуляторе код работает и не содержит критических ошибок его можно загрузить в ПЛИС.

### Тестирование компонентов: testbench (испытательный стенд)

Если мы купили микросхему и хотим ее проверить, что мы делаем? Обычно паяем на плату или вставляем в панельку. Еще есть вариант вставить ее в макетную плату или подключить на проводах.

В Verilog (и других языках HDL) принцип аналогичный. **Вы не можете симулировать компонент сам по себе**, вам нужно написать **testbench**, который описывает какие "провода" подключаются к каким выводам и какие сигналы куда подавать, чтобы проверить работоспособность и сравнить выходные сигналы с ожидаемыми. Этот testbench так же пишется на Verilog.

Как нам протестировать компонент setbit? Это деталь которая имеет всего один выход и его значение всегда "1". В случае настоящей аппаратной микросхемы мы вставили бы ее в панельку, подали бы питание и подсоединили бы провод к выходному контакту, затем проверили бы что на нем логическая "1" вольтметром, осциллографом или логическим анализатором. Мы сделаем то же самое, но на Verilog. Мы сделаем вот так:

![Image 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/setbit-3.png)

Файл называется setbit_tb.v. Мы всегда будем использовать суффикс "_tb" чтобы показать что это testbench, а не часть схемы проекта. Стоит заметить что testbench не синтезируем в железе, это код только для симулятора. Поэтому в данный момент мы может рассматривать Verilog как язык программирования, мы программируем что произойдет с нашей схемой и какой результат мы ожидаем.

Testbench содержит 3 элемента: компонент "setbit", провод "A" и блок проверки. Вот код нашего testbench:
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
## Симуляция!

Мы будем использовать icarus verilog и GTKwave для симуляции. Запускаем:

    $ make sim

и откроется окно с результатами симуляции:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/ICESTICK/T01-setbit/images/T01-setbit-simul-1.png" width="600" align="center">

На логическом анализаторе мы видим ожидаемое поведение: сигнал A всегда в значении "1". Пока мы можем не обращать внимание на единицы измерения времени (по умолчанию это секунды). После 20 единиц, симуляция завершается. 

В консоле появятся такие сообщения:

    #-- Compilar
    iverilog setbit.v setbit_tb.v -o setbit_tb.out
    #-- Simular
    ./setbit_tb.out
    VCD info: dumpfile setbit_tb.vcd opened for output.
    Component ok!
    #-- Ver visualmente la simulacion con gtkwave
    gtkwave setbit_tb.vcd setbit_tb.gtkw &

Одно из них гласит "**Component ok!**" это то сообщение, которое мы поместили в testbench для случая, когда A = "1".

## Упражнения

* Измените компонент на фиксированное значение "0". Загрузите его в ПЛИС. Проверьте что светодиод погас.

* Измените пример так, чтобы setbit был подключен к светодиоду D2 (подключен к пину 98 вместо 99)

## Заключение
Мы сделали первую схему на  verilog, "hello world" содержаший 1 сигнал. Мы синтезировали его и загрузили в ПЛИС. Так же мы написали testbech для него на verilog и симулировали его. Теперь у нас установлены и настроены все утилиты и мы проверили их работоспособность. Теперь мы готовы к более сложным задачам и дальнейшему изучению Verilog.

### Пройденные понятия:

* Подключение сигнала к контактам (пинам)
* Создание компонентов при помощи конструкции module
* Определение выходов с помощью output
* Определение провода с помощью wire
* testbench