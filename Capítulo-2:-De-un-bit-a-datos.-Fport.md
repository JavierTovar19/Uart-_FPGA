<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T02-Fport)

## Introducción

Ahora en vez de 1 bit sacaremos 4, y los mostraremos por los leds. Se trata de un valor fijo, que está "cableado por hardware". Si queremos visualizar otro número por los leds, habrá que sintentizar otro circuito.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-1.png)

Este componente lo denominaremos Fport (Fixed port). Tiene un bus de salida de 4 bits, etiquetado como data, que está cableado al valor binario 1010

## Fport.v: Descripción del hardware

Este circuito es muy parecido al del tutorial anterior (setbit.v) pero en vez de tener 1 bit de salida tiene 4. Se describe así:

    //-- Fichero Fport.v
    module Fport(output [3:0] data);
    
    //-- La salida del modulo son 4 cables
    wire [3:0] data;
    
      //-- Sacar el valor por el bus de salida
      assign data = 4'b1010; //-- 4'hA
    
    endmodule

La salida ahora es un **array de 4 cables**. Esto se denota poniendo [3:0] delante del nombre. Para realizar la asignación escribimos el número en binario usando la notación de Verilog: Primero el número de bits, luego el carácter ', a continuación la base del número (b para binario) y por último los 4 dígitos binarios.  Este mísmo número se podría expresar mediante un único dígito hexadecimal mediante:  4'hA. También lo podriamos poner en decimal como 4'd10

## Síntesis en la FPGA

Cada uno de los 4 bits de la salida data se saca por los pines de la fpga donde están conectados los 4 leds:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-2.png)

Esto se especifica en el fichero Fport.pcf:

    set_io data[0] 99
    set_io data[1] 98
    set_io data[2] 97
    set_io data[3] 96

Para realizar la síntesis entramos en el directorio **tutorial/T02-Fport** y ejecutamos el comando **make sint**:

    $ make sint

En el mensaje final obtenemos un resumen de los recursos de la FPGA consumidos y los que quedan libres:

    After placement:
    PIOs       2 / 96
    PLBs       1 / 160
    BRAMs      0 / 16

Estos recursos son:
* PIO = Programmable I/O (Entradas / salidas programables)
* PLBs = Programmable Logic Blocks (Bloques lógicos programables)
* BRAMs = Block RAM Memory (Bloques de memoria)

Ahora descargamos en la fpga el fichero Fport.bin:

    $ sudo iceprog Fport.bin

Al terminar, dos leds estarán encendidos y dos apagados, ya que estamos enviando el valor 1010:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-iCEstick-2.png" width="400" align="center">

## Simulación

El banco de pruebas es similar al del capítulo anterior, pero ahora en vez de comprobar sólo un bit se comprueba el patrón de 4 bits. Si no es igual al esperado se emite un mensaje de error. El diagrama es el siguiente:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-3.png)

El banco de pruebas (Fport_tb.v) consta de tres elementos:

* El componente a probar: Fport  (En la liteatura se conoce como uut: unit under test)
* El bloque de comprobación 
* El cable DATA

El código del banco de pruebas es:

    //-- Fport_tb.v
    module Fport_tb;
    
    //-- Bus de 4 cables para conectarlos a la salida del componente Fport
    wire [3:0] DATA;
    
    //--Instanciar el componente. Conectar la salida a DATA
    Fport FP1 (
      .data (DATA)
    );
    
    //-- Comenzamos las pruebas
    initial begin
    
      //-- Fichero donde almacenar los resultados
      $dumpfile("Fport_tb.vcd");
      $dumpvars(0, Fport_tb);
    
      //-- Pasadas 10 unidades de tiempo comprobamos
      //-- si el cable tiene el patron establecido
      # 10 if (DATA != 4'b1010)
             $display("---->¡ERROR! Salida Erronea");
           else
             $display("Componente ok!");
    
      //-- Terminar la simulacion 10 unidades de tiempo despues
      # 10 $finish;
    end
    
    endmodule

Observamos que la salida del componente es data, y le hemos conectado el cable DATA, para enfatizar el hecho de que los nombres pueden ser diferentes
