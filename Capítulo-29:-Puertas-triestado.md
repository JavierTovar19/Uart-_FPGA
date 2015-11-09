(dibujo)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T29-tristate)

# Introducción
Las **puertas triestado** (También conocidas como [Buffers triestado](https://es.wikipedia.org/wiki/Buffer_triestado)) nos permiten **desconetar las salidas de nuestros circuitos**, impidiendo que vuelquen su tensión (0 ó 1) en el cable al que se conectan.

Las describiremos en Verilog, las simularemos y documentaremos las **LIMITACIONES QUE PRESENTAN** al ser **sintetizadas** con las **herramientas libres** del proyecto icestorm. El soporte de Yosys, Arachne y Icestorm para puertas triestado es muy limitado todavía al día de hoy (Nov - 2015)

# Puertas triestado
## Funcionamiento
Las puertas triestado se comportan como **interruptores** que conectan la entrada con la salida cuando su señal de habilitación está activada:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T29-tristate/images/tristate-1.png)

Tiene 3 puertos de 1 bit:
* **entrada**:  Señal de entrada
* **salida**: Señal de salida
* **enable**: Señal de habilitación

## Descripción en Verilog

# Ejemplo 1: conexión de un led mediante una puerta triestado

# Ejemplo 2: Conexión de un led a un bus de 1 bit

# Limitaciones en síntesis

# Alternativas a las puertas triestado

# Ejercicios

# Conclusiones
TODO