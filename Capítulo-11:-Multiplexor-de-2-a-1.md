[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T11-mux-2-1)

## Introducción
Los **multiplexores** son circuitos combinacionales que nos permiten **seleccionar** entre varias fuentes de datos. En este capítulo utilizaremos un multiplexor de 2 a 1 para mostrar por los leds una secuencia de dos valores, que se mostrarán alternativamente

## Descripción del multiplexor 2 a 1

Un multiplexor 2 a 1 selecciona entre 2 fuentes de datos según el valor de su **entrada de selección** sel. Si sel es 0, se saca por la salida la fuente 0, si es 1 se saca la fuente 1

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/e9142aee8f70b7342c69b990159529ce68601487/tutorial/T11-mux-2-1/images/mux2-2.png)

Los describimos en Verilog usando la intrucción if ... else:

    always @(fuente0 or fuente1 or sel)
      if (sel == 0)
        dout <= fuente0;
      else
        dout <= fuente1;

Es muy importante que exista el **else**. Al tratarse de un circuito combinacional, **todos los casos deben estar cubiertos**. Si no es así, se pueden inferir registros.  También es muy importante colocar en la **lista de sensibilidad** TODAS las señales de entrada: fuente0, fuente1 y sel.

La lista de sensibilidad se puede escribir de forma abreviada con el argumento @*

    always @*

Esta lista incluye automáticamente todas las señales de entrada

## mux2.v: Secuenciador de 2 estados

Como ejemplo de uso, haremos un **secuenciador de 2 estados**: Un circuito que envía alternativamente dos datos de 4 bits a los leds. El esquema del circuito es el siguiente:

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T11-mux-2-1/images/mux2-3.png)

Se utilizan 2 fuentes de datos fijas (están cableadas a valores fijos) que determinan el estado de los leds en cada momento. El multiplexor selecciona alternativamemente entre una y otra a través de una señal de reloj que pasa por un prescaler (para reducir la frecuencia y que podamos apreciar el movimiento de los leds).

El código verilog es el siguiente:



## Síntesis en la FPGA


![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T11-mux-2-1/images/mux2-1.png)

## Simulación

## Ejercicios propuestos

## Conclusiones
TODO