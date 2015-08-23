
![Image 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T14-regreset/images/regreset-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T14-regreset)

## Introducción
Los registros son elementos muy usados para hacer circuitos digitales. Aunque su descripción en verilog es muy sencilla, crearemos un **registro genérico de N bits** que podremos instanciar en nuestros diseños. Esto nos permitirá **colocar muchos registros fácilmente**. Además, añadiremos una **entrada de reset síncrona** que cargará el registro con un valor inicial (Por defecto será 0 pero podremos indicar otro valor al instanciar el registro)

## register.v: Registro de N bits, con reset síncrono