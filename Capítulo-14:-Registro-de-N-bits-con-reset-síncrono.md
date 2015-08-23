
![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T14-regreset)

## Introducción
Los registros son elementos muy usados para hacer circuitos digitales. Aunque su descripción en verilog es muy sencilla, crearemos un **registro genérico de N bits** que podremos instanciar en nuestros diseños. Esto nos permitirá **colocar muchos registros fácilmente**. Además, añadiremos una **entrada de reset síncrona** que cargará el registro con un valor inicial (Por defecto será 0 pero podremos indicar otro valor al instanciar el registro). Como ejemplo haremos un secuenciador con 2 registros, sacándose por los leds los datos almacenados en ellos, alternativamente.

## register.v: Registro de N bits, con reset síncrono

El esquema del registro es el mostrado en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-2.png)

Tiene una **entrada de datos din de N bits**, y su **salida dout** correspondiente. Los datos se capturan en **flanco de subida** de la señal de reloj. Dispone de una **entrada de reset síncrona activa a nivel bajo**: Si se recibe un 0 por esta señal y llega un flanco de reloj, el registro se carga con su valor inicial (INI), que se define al instanciarlo

Esta manera de inicilizar el registro ya la conocemos del capítulo anterior, pero ahora se ha integrado dentro del propio registro. Su descripción en Verilog es la siguiente:


