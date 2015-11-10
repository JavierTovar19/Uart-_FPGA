(dibujo)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T29-tristate)

# Introducción
Las **puertas triestado** (También conocidas como [Buffers triestado](https://es.wikipedia.org/wiki/Buffer_triestado)) nos permiten **desconetar las salidas de nuestros circuitos**, impidiendo que vuelquen su tensión (0 ó 1) en el cable al que se conectan.

Las describiremos en Verilog, las simularemos y documentaremos las **LIMITACIONES QUE PRESENTAN** al ser **sintetizadas** con las **herramientas libres** del proyecto icestorm. El soporte de Yosys, Arachne y Icestorm para puertas triestado es muy limitado todavía al día de hoy (Nov - 2015)

# Puertas triestado

Describiremos cómo funcionan y cómo se modelan en verilog

## Funcionamiento
Las puertas triestado se comportan como **interruptores** que conectan la entrada con la salida cuando su señal de habilitación está activada. Su símbolo es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-1.png)

Tiene 3 puertos de 1 bit:
* **entrada**:  Señal de entrada
* **salida**: Señal de salida
* **enable**: Señal de habilitación

Cuando la señal de habilitacion está a **1**, la **entrada está conectada directamente a la salida**. Sin embargo, cuando está a **0**, **la salida queda desconectada**. Se considera que está en un tercer estado, denominado **alta impedancia**, y se representa con la letra **Z**.  Así, la salida de una puerta triestado puede encontrarse en los **estados 0**, **1** ó **Z**

## Descripción en Verilog

Una puerta triestado se modela de la siguiente forma:

```verilog
assign salida = (enable) ? entrada : 1'bz;
```

El sintetizador reconoce que es una puerta triestado porque se **le asigna el valor z** cuando enable es 0

# Ejemplo 1: conexión de un led mediante una puerta triestado

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-ex1.png)

# Ejemplo 2: Conexión de un led a un bus de 1 bit

# Limitaciones en síntesis

# Alternativas a las puertas triestado

# Ejercicios

# Conclusiones
TODO