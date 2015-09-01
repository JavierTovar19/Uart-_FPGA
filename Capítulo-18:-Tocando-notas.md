![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T18-notas)

## Introducción
Diseñaremos un circuito con **8 canales**, cada uno emitiendo **una nota musical**: do, re, mi, fa, sol, la, si, do. Al conectar cada canal a un altavoz / zumbador oiremos las notas.

El circuito es similar al del capítulo anterior. Sólo necesitamos **calcular los valores de los divisores** para obtener las frecuencias de las notas

## Frecuencias de las notas musicales

En la siguiente tabla se resumen las frecuencias y valores del divisor para generar las 12 notas de la **cuarta octava**:

| NOTA  | Frecuencia (Hz)  |  Valor divisor
|-------|------------------|---------------


[1]

Cálculo notas en python:

freq en Hz:

f = 440.0 * m.exp(((octava-4)+(nota-10)/12.0) * m.Log(2)));

octava = 0 - 10
nota = n=1 para Do, n=2 para Do#... n=12 para Si



## Referencias

[1] [Frecuencia de las notas musicales](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales). Por Vic (**La Tecla de Escape**)