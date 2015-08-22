![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T13-reg-init)

## Introducción
Ya podemos responder a la pregunta planteada en el capítulo 8 sobre **cómo realizar la inicialización de los registros**. Los registros sintetizados están a 0. Normalmente necesitamos cargar en ellos un valor inicial, y que luego funcionen para lo que hayan sido diseñados. En este capítulo mostraremos cómo hacerlo, usando las herramientas que ya conocemos: el **multiplexor** y el **inicializador**

## Inicializando registros

Partimos de un **registro genérico de N bits**, que ya conocemos, con una entrada din, una salida dout y una señal de reloj

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-2.png)

Queremos que se cargue con un valor inicial al principio y que luego funcione normalmente. Para hacerlo colocamos un **multiplexor de 2 a 1** en su entrada (para dividir la entrada en 2). Por una entrada del multiplexor ponemos el **valor inicial** y por la otra la entrada genérica del registro din2.

Dibujo: Registro + mux 2 - 1

**Es muy importante que el valor inicial se introduzca por la fuente 0 del multiplexor**.

Ahora ya simplemente conectamos un inicializador a la entrada de selección del multiplexor.

Dibujo: registr + mux + inicializador

 De esta forma, al arrancar, el inicializador emitirá un 0 y por la entrada din del registro llegará el valor inicial. En el siguiente franco de subida este valor inicial se captura y el inicializador pasa a 1, por lo que ahora se seleccionará la fuente 1, que será por donde vengan los datos del registro en el régimen normal de funcionamiento



## reginit.v: Secuenciador de 2 estados con registro

## Síntesis en la FPGA

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T13-reg-init/images/reginit-1.png)