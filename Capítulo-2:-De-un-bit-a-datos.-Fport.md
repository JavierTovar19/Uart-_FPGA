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

Ahora descargamos en la fpga el fichero Fport.bin:

    $ sudo iceprog Fport.bin

Al terminar, dos leds estarán encendidos y dos apagados, ya que estamos enviando el valor 1010:

<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T02-Fport/images/Fport-iCEstick-2.png" width="400" align="center">


