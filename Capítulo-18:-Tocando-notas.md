![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T18-notas)

## Introducción
Diseñaremos un circuito con **8 canales**, cada uno emitiendo **una nota musical**: do, re, mi, fa, sol, la, si, do. Al conectar cada canal a un altavoz / zumbador oiremos las notas.

El circuito es similar al del capítulo anterior. Sólo necesitamos **calcular los valores de los divisores** para obtener las frecuencias de las notas

## Frecuencias de las notas musicales

En la siguiente tabla se resumen las **frecuencias** y **valores de los divisores** para generar las **12 notas** de la **cuarta octava**:

| NOTA  | Valor del divisor |  Frecuencia (Hz)
|-------|-------------------|---------------
| DO    | 45868             |  261.626 Hz 
| DO#   | 43293             |  277.183 Hz
| RE    | 40863             |  293.665 Hz
| RE#   | 38570             |  311.127 Hz
| MI    | 36405             |  329.628 Hz
| FA    | 34362             |  349.228 Hz
| FAs   | 32433             |  369.994 Hz
| SOL   | 30613             |  391.995 Hz
| SOL#  | 28895             |  415.305 Hz
| LA    | 27273             |  440.000 Hz
| LA#   | 25743             |  466.164 Hz
| SI    | 24298             |  493.883 Hz

**Las frecuencias de las notas**, según su **octava**, se obtiene mediante esta ecuación (cuya nota de referencia es LA de la 4ª octava, con una frecuencia de 440 Hz)

![Ecuación para las frecuencias de las notas, según su octava](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-2.png)

Donde **o** es la **octava** (toma valores desde 0 hasta 10) y **n** la **nota** (valores desde 1 hasta 12), siendo n = 1 el DO

En [1](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales) se hay una explicación muy buena sobre las frecuencias de las notas así como del desarrollo de esa ecuación

## Pendiente
[1]

Cálculo notas en python:

freq en Hz:

f = 440.0 * m.exp(((octava-4)+(nota-10)/12.0) * m.Log(2)));

octava = 0 - 10
nota = n=1 para Do, n=2 para Do#... n=12 para Si



## Referencias

[1] [Frecuencia de las notas musicales](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales). Por Vic (**La Tecla de Escape**) [Última consulta: 1-Sep-2015]