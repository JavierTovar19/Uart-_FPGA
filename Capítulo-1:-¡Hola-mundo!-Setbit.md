<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-iCEstick.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T01-setbit)

## Introducción

El circuito digital más sencillo es simplemente un cable que está conectado a un nivel lógico conocido, por ejemplo 1. De esta forma al conectar un led se iluminará (1) o se apagará (0)

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-1.png)

La salida de este circuito la hemos denominado A.

## setbit.v: Descripción del hardware

Para sintetizar este circuito en la fpga, primero lo tenemos que describir usando un lenguaje de descripción hardware (HDL). En este tutorial utilizaremos Verilog, ya que tenemos todas las herramientas libres para su simulación / síntesis.

Verilog es un lenguaje que sirve para describir hardware... pero ¡Cuidado! ¡NO ES UN LENGUAJE DE PROGRAMACIÓN! ¡Es un lenguaje de descripción! Nos permite describir las conexiones y los elementos de un sistema digital.

El código verilog que implementa este circuito "hola mundo" se encuentra en el fichero [setbit.v](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T01-setbit/setbit.v). Tiene esta pinta:

```verilog
//-- Fichero setbit.v
module setbit(output A);
wire A;
    
assign A = 1;
    
endmodule
```

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

Si compramos un chip y lo queremos probar, ¿Qué hacemos?. Hombre, normalmente lo soldamos directamente en el PCB o lo introducimos en un zócalo.  Pero también lo podemos pinchar en una placa entrendora y colocar nosotros los cables de conexión a la alimentación, señales y resto de componentes.

En verilog (y resto de lenguajes HDL) hacemos lo mismo.  Un componente descrito en Verilog (como por ejemplo setbit.v) **no se puede simular directamente**. Es necesario escribir un **banco de pruebas** que indique qué cables conectar a sus pines, qué valores de prueba enviar y comprobar que por sus salidas salen resultados correctos. Este banco de pruebas es un fichero también en Verilog.

¿Cómo comprobamos el componente setbit? Se trata de un chip que sólo tiene un pin de salida que siempre está a '1'. En la vida real lo pondríamos en su placa de puntos, lo alimentaríamos, conectaríamos un cable en su pin de salida (A) y usando un polímetro comprobaríamos que sale una tensión igual a la de alimentación (un '1').  Haremos exactamente eso, pero describiéndolo en Verilog. Gráficamente tendríamos lo siguiente:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/setbit-3.png)

El fichero se llama setbit_tb.v. Siempre usaremos el sufijo _tb para indicar que se trata de un banco de pruebas (TestBench) y ¡No de hardware real!. Este banco de pruebas NO ES SINTETIZABLE. Es un código que sólo vale para la SIMULACIÓN.  Lo que sintetizamos es el componente setbit.v.  Por eso, a la hora de hacer bancos de pruebas, estamos empleando Verilog no como una herramienta de descripción hardware sino como un programa. Aquí sí podemos pensar en Verilog como un lenguaje de programación tradicional.

El banco de pruebas tiene 3 elementos: el componente setbit, un cable que hemos llamado A y un bloque de comprobación (que sería el equivalente al polímetro en el mundo real). El código es el siguiente:

    //-- Modulo para el test bench
    module setbit_tb;
    
    //-- Cable para conectar al pin de salida de setbit
    //-- Le podemos dar CUALQUIER nombre. Pero le llamamos A (igual que el pin de setbit)
    wire A;
    
    //-- Colocar el componente (se denomina "Instanciar") y
    //-- Conectar en el pin A el cable A
    setbit SB1 (
     .A (A)
    );
    
    //-- Comenzamos las pruebas (bloque de comprobacion)
    initial begin
    
        //-- Definir el fichero donde volcar los datos
        $dumpfile("setbit_tb.vcd");
    
        //-- Volcar todos los datos a ese fichero (al finalizar la simulacion)
        $dumpvars(0, setbit_tb);
    
        //-- Pasadas 10 unidades de tiempo comprobamos si el cable está a '1'
        //-- En caso de no estar a 1, se informa del problema, pero la
        //-- simulacion no se detiene
        # 10 if (A != 1)
               $display("---->¡ERROR! Salida no esta a 1");
             else
               $display("Componente ok!");

      //-- Terminar la simulacion tras 10 unidades de tiempo desde la comprobacion
      # 10 $finish;
    end
    endmodule

## ¡Simulando!

Para simular utilizamos las herramientas icarus verilog y GTKwave. Ejecutamos el comando:

    $ make sim

y automáticamente se nos abrirá una ventana con el resultado de la simulación:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T01-setbit/images/T01-setbit-simul-1.png" width="600" align="center">

A un golpe de vista comprobamos que se comporta como esperábamos: la señal del cable A está siempre a '1'. Ignoramos las unidades de tiempo que pone, que por defecto está en segundos. Para nosotros serán unidades de tiempo. Tras 20 unidades, la simulación termina.

En la consola veremos que han aparecido los siguientes mensajes:

    #-- Compilar
    iverilog setbit.v setbit_tb.v -o setbit_tb.out
    #-- Simular
    ./setbit_tb.out
    VCD info: dumpfile setbit_tb.vcd opened for output.
    Componente ok!
    #-- Ver visualmente la simulacion con gtkwave
    gtkwave setbit_tb.vcd setbit_tb.gtkw &

El que pone "**Componente ok!**" es el que hemos puesto nosotros en el banco de pruebas cuando la salida estaba a '1'

## Ejercicios propuestos

* Cambiar el componente para que saque un 0. Descargarlo en la fpga. Comprobar que el led está ahora apagado

* Cambiar el ejemplo original para que se encienda el led D2 en vez el D1  (está asociado al pin 98 en vez de al 99)

## Conclusiones
TODO
