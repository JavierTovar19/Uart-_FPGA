![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T18-notas)

## Introducción
Diseñaremos un circuito con **8 canales**, cada uno emitiendo **una nota musical**: do, re, mi, fa, sol, la, si, do. Al conectar cada canal a un altavoz / zumbador oiremos las notas.

El circuito es similar al del capítulo anterior. Sólo necesitamos **calcular los valores de los divisores** para obtener las frecuencias de las notas

## Frecuencias de las notas musicales

En la siguiente tabla se resumen las **frecuencias** y **valores de los divisores** para generar las **12 notas** de la **cuarta octava**:

| NOTA  | Valor del divisor |  Frecuencia (Hz)
|-------|-------------------|---------------
| DO    | 45867             |  261.626 Hz 
| DO#   | 43293             |  277.183 Hz
| RE    | 40863             |  293.665 Hz
| RE#   | 38569             |  311.127 Hz
| MI    | 36405             |  329.628 Hz
| FA    | 34361             |  349.228 Hz
| FAs   | 32433             |  369.994 Hz
| SOL   | 30613             |  391.995 Hz
| SOL#  | 28894             |  415.305 Hz
| LA    | 27273             |  440.000 Hz
| LA#   | 25742             |  466.164 Hz
| SI    | 24297             |  493.883 Hz

**Las frecuencias de las notas**, según su **octava**, se obtiene mediante esta ecuación (cuya nota de referencia es LA de la 4ª octava, con una frecuencia de 440 Hz)

![Ecuación para las frecuencias de las notas, según su octava](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-2.png)

Donde **o** es la **octava** (toma valores desde 0 hasta 10) y **n** la **nota** (valores desde 1 hasta 12), siendo n = 1 el DO

En [ [1] ](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales) se hay una explicación muy buena sobre las frecuencias de las notas así como del desarrollo de esa ecuación

## Obteniendo la tabla de frecuencias y divisores

Con esta **función en python** calculamos las frecuencias:

``` python
import math as m

def freq(note, octave):
	return 440.0 * m.exp(((octave-4)+(note-10)/12.0) * m.log(2))
```

Así, para conocer por ejemplo la **frecuencia de la nota DO de la 4ª octava**, ejecutamos freq:
``` python
>>> freq(1, 4)
261.6255653005986
```
y el **valor del divisor** para generar esa nota en la iCEstick lo obtenemos dividiendo 12Mhz entre su frecuencia:

``` python
>>> 12000000 / freq(1, 4)
45867.07719565716
```
Ese número lo tenemos que redondear para que sea entero, quedando: 45867

La función en python para calcular directamente el valor del divisor es:

``` python
def divisor(note, octave):
	return int(round(12000000 / freq(note, octave)))
```

El valor del divisor para la nota DO de la cuarta octava la obtenemos con:

``` python
>>> divisor(1, 4)
45867
```

Este **programa** en python **genera automáticamente una tabla con las frecuencias y divisores**, en formato Verilog, para poder ser copiado al fichero divisor.vh:

``` python
import math as m

##-- Diccionario con los nombres de las notas
nname = {1: 'DO',   2: 'DOs', 3: 'RE',   4 : 'REs',
         5: 'MI',   6: 'FA',  7: 'FAs',  8: 'SOL',
         9: 'SOLs', 10: 'LA', 11: 'LAs', 12: 'SI'};

#-- Calcular la frecuencia de una nota de una octava
def freq(note, octave = 4):
	return 440.0 * m.exp(((octave-4)+(note-10)/12.0) * m.log(2))
	
#-- Calcular el valor del divisor para tocar la nota en la FPGA
#-- de la placa iCEstick
def divisor(note, octave = 4):
	return int(round(12000000 / freq(note, octave)))

#-- Imprimir la table, con salida verilog
def print_table(octave = 4):
	print("//-- Octava: {}".format(octave))
	for note in range(12):
		#-- Print table in verilog sintax
		print("`define {}_{} {} //-- {:.3f} Hz".format(
		       nname[note + 1], 
		       octave, 
		       divisor(note+1, octave), 
		       freq(note+1, octave)))
	print("\n")

#-- Programa principal
#-- Sacar la tabla por la pantalla
for oct in range(11):
	print_table(oct)
```

## Referencias

[1] [Frecuencia de las notas musicales](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales). Por Vic (**La Tecla de Escape**) [Última consulta: 1-Sep-2015]