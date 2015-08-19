![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T09-inicializador)

## Introducción
Muchos circuitos digitales **necesitan inicializarse** antes de comenzar a trabajar normalmente. Su funcionamiento se divide en un **estado de arranque**, donde se inicializan los valores de los registros y un estado de **régimen permanente** donde se realiza la función para lo que han sido diseñados.

Para lograr esto necesitamos un circuito de inicialización que nos genere una **señal escalón**: que inicialmente esté a cero y al llegar el primer flanco de reloj pase a 1 y permanezca a 1 durante todo el funcionamiento de la máquina.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-2.png)

Esto se implementa de una manera muy sencilla utilizando un **registro de 1 bit**[1] al que se le cablea un "1" en su entrada. Inicialmente el registro estará a 0. Al llegar el primer flanco de reloj, se captura el 1 y se saca por su salida, **generando el flanco de subida para realizar la inicialización**. Para el resto de ciclos de reloj esta señal siempre estará a 1

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-3.png)

## Init.v: Descripción del hardware


## Síntesis en la FPGA
![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-1.png)



<img src="" width="400" align="center">

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO

[1] Un registro de 1 bit es en realidad un **flip-flop**. Sin embargo prefiero llamarlo registro de 1 bit para generaliza. Su implementación en Verilog es la misma que la de un registro de N bits

