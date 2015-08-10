<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

## Introducción

El circuito digital más sencillo es simplemente un cable que está conectado a un nivel lógico conocido, por ejemplo 1. De esta forma al conectar un led se iluminará (1) o se apagará (0)

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-1.png)

La salida de este circuito la hemos denominado A.

## setbit.v: Descripción del hardware

Para sintetizar este circuito en la fpga, primero lo tenemos que describir usando un lenguaje de descripción hardware (HDL). En este tutorial utilizaremos Verilog, ya que tenemos todas las herramientas libres para su simulación / síntesis.

Verilog es un lenguaje que sirve para describir hardware... pero ¡Cuidado! ¡NO ES UN LENGUAJE DE PROGRAMACIÓN! ¡Es un lenguaje de descripción! Nos permite describir las conexiones y los elementos de un sistema digital.

El código verilog que implementa este circuito "hola mundo" se encuentra en el fichero [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v). Tiene esta pinta:

    //-- Fichero setbit.v
    module setbit(output A);
    wire A;
    
      assign A = 1;
    
    endmodule

## Síntesis en la FPGA

Además del fichero en verilog del componente, necesitamos indicar a qué pin de la FPGA queremos conectar la salida A de nuestro componente. Este mapeo se realiza en el fichero setbit.pcf (pcf = Physical Constraint file). Lo sacaremos por el pin 99 que se corresponde con el led D1 de la placa ICEStick. Pero podría ser cualquier otro :

    set_io A 99

Sólo consta de una línea, en la que se asocia la etiqueta A del componente al pin 99 de la FPGA

En la figura 2 se muestra gráficamente esta idea.  Como lo que estamos describiendo es hardware, siempre es interesante hacerse esquemas y dibujos para comprenderlo mejor:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-2.png)

Para hacer la síntesis completa nos vamos al directorio _tutorial/T01-setbit_ y ejecutamos el comando **make sint** desde la consola:

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

Saldrán muchos más mensajes. En mi portátil tarda en sintetizar menos de 1 segundos. Los mensajes finales que se obtienen son:

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

Al terminar se habrá generado el fichero setbit.bin que es el que descargaremos en la FPGA para configurarla. Introducimos la iCEStick en el USB y ejecutamos este comando:

    $ sudo iceprog setbit.bin

El proceso dura aproximadamente 30 segundos (con la versión actual del iceprog. Este tiempo se irá reduciendo según se vaya mejorando el software por la comunidad). Los mensajes que aparecen son:

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

El led D1 de la ICEStick se encenderá:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

## Simulación

### Primero simulamos, luego sintetizamos

El ejemplo anterior lo hemos descargado directamente en la FPGA porque el diseño ya **ESTABA PROBADO PREVIAMENTE**. Cuando trabajamos con FPGAs **estamos haciendo hardware** y tenemos que tener siempre mucho cuidado. Podemos escribir un código que por ejemplo tenga un cortocircuito. Y podría ocurrir que las herramientas de síntesis no nos avisen con un warning (sobre todo con estas primeras versiones que todavía están en estado alfa). Si lo descargamos en la fpga la podríamos estropear parcialmente.

Por ello, **SIEMPRE** hay que **simular el código** que hagamos. Y una vez que estamos lo bastante seguros de que funciona (o que no tiene error gordos) es cuando lo cargamos en la FPGA.

### Probando componentes: banco de trabajo




