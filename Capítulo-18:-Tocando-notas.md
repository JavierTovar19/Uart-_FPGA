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
#-- notas_gen.py
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

Al ejecutarlo, se nos genera por pantalla la tabla completa, que pegaremos en el archivo **divider.dh**:

    $ python notas_gen.py

## divider.dh: Notas musicales añadidas

El nuevo archivo con todas las constantes para usar nuestro divisor queda:

``` verilog
//-- Fichero divisor.vh
//-------------------- Frecuencias
//-- Megaherzios  MHz
`define F_4MHz 3
`define F_3MHz 4
`define F_2MHz 6
`define F_1MHz 12

//-- Kilohercios KHz
`define F_4KHz 3000
`define F_3KHz 4000
`define F_2KHz 6000
`define F_1KHz 12000

//-- Hertzios (Hz)
`define F_4Hz   3000000
`define F_3Hz   4000000
`define F_2Hz   6000000
`define F_1Hz   12000000

//------- Frecuencias para notas musicales
//-- Octava: 0
`define DO_0   733873 //-- 16.352 Hz
`define DOs_0  692684 //-- 17.324 Hz
`define RE_0   653807 //-- 18.354 Hz
`define REs_0  617111 //-- 19.445 Hz
`define MI_0   582476 //-- 20.602 Hz
`define FA_0   549784 //-- 21.827 Hz
`define FAs_0  518927 //-- 23.125 Hz
`define SOL_0  489802 //-- 24.500 Hz
`define SOLs_0 462311 //-- 25.957 Hz
`define LA_0   436364 //-- 27.500 Hz
`define LAs_0  411872 //-- 29.135 Hz
`define SI_0   388756 //-- 30.868 Hz

//-- Octava: 1
`define DO_1   366937 //-- 32.703 Hz
`define DOs_1  346342 //-- 34.648 Hz
`define RE_1   326903 //-- 36.708 Hz
`define REs_1  308556 //-- 38.891 Hz
`define MI_1   291238 //-- 41.203 Hz
`define FA_1   274892 //-- 43.654 Hz
`define FAs_1  259463 //-- 46.249 Hz
`define SOL_1  244901 //-- 48.999 Hz
`define SOLs_1 231156 //-- 51.913 Hz
`define LA_1   218182 //-- 55.000 Hz
`define LAs_1  205936 //-- 58.270 Hz
`define SI_1   194378 //-- 61.735 Hz

//-- Octava: 2
`define DO_2   183468 //-- 65.406 Hz
`define DOs_2  173171 //-- 69.296 Hz
`define RE_2   163452 //-- 73.416 Hz
`define REs_2  154278 //-- 77.782 Hz
`define MI_2   145619 //-- 82.407 Hz
`define FA_2   137446 //-- 87.307 Hz
`define FAs_2  129732 //-- 92.499 Hz
`define SOL_2  122450 //-- 97.999 Hz
`define SOLs_2 115578 //-- 103.826 Hz
`define LA_2   109091 //-- 110.000 Hz
`define LAs_2  102968 //-- 116.541 Hz
`define SI_2   97189 //-- 123.471 Hz

//-- Octava: 3
`define DO_3   91734 //-- 130.813 Hz
`define DOs_3  86586 //-- 138.591 Hz
`define RE_3   81726 //-- 146.832 Hz
`define REs_3  77139 //-- 155.563 Hz
`define MI_3   72809 //-- 164.814 Hz
`define FA_3   68723 //-- 174.614 Hz
`define FAs_3  64866 //-- 184.997 Hz
`define SOL_3  61226 //-- 195.998 Hz
`define SOLs_3 57789 //-- 207.652 Hz
`define LA_3   54545 //-- 220.000 Hz
`define LAs_3  51484 //-- 233.082 Hz
`define SI_3   48594 //-- 246.942 Hz

//-- Octava: 4
`define DO_4   45867 //-- 261.626 Hz
`define DOs_4  43293 //-- 277.183 Hz
`define RE_4   40863 //-- 293.665 Hz
`define REs_4  38569 //-- 311.127 Hz
`define MI_4   36405 //-- 329.628 Hz
`define FA_4   34361 //-- 349.228 Hz
`define FAs_4  32433 //-- 369.994 Hz
`define SOL_4  30613 //-- 391.995 Hz
`define SOLs_4 28894 //-- 415.305 Hz
`define LA_4   27273 //-- 440.000 Hz
`define LAs_4  25742 //-- 466.164 Hz
`define SI_4   24297 //-- 493.883 Hz

//-- Octava: 5
`define DO_5   22934 //-- 523.251 Hz
`define DOs_5  21646 //-- 554.365 Hz
`define RE_5   20431 //-- 587.330 Hz
`define REs_5  19285 //-- 622.254 Hz
`define MI_5   18202 //-- 659.255 Hz
`define FA_5   17181 //-- 698.456 Hz
`define FAs_5  16216 //-- 739.989 Hz
`define SOL_5  15306 //-- 783.991 Hz
`define SOLs_5 14447 //-- 830.609 Hz
`define LA_5   13636 //-- 880.000 Hz
`define LAs_5  12871 //-- 932.328 Hz
`define SI_5   12149 //-- 987.767 Hz

//-- Octava: 6
`define DO_6    11467 //-- 1046.502 Hz
`define DOs_6   10823 //-- 1108.731 Hz
`define RE_6    10216 //-- 1174.659 Hz
`define REs_6   9642 //-- 1244.508 Hz
`define MI_6    9101 //-- 1318.510 Hz
`define FA_6    8590 //-- 1396.913 Hz
`define FAs_6   8108 //-- 1479.978 Hz
`define SOL_6   7653 //-- 1567.982 Hz
`define SOLs_6  7224 //-- 1661.219 Hz
`define LA_6    6818 //-- 1760.000 Hz
`define LAs_6   6436 //-- 1864.655 Hz
`define SI_6    6074 //-- 1975.533 Hz

//-- Octava: 7
`define DO_7   5733 //-- 2093.005 Hz
`define DOs_7  5412 //-- 2217.461 Hz
`define RE_7   5108 //-- 2349.318 Hz
`define REs_7  4821 //-- 2489.016 Hz
`define MI_7   4551 //-- 2637.020 Hz
`define FA_7   4295 //-- 2793.826 Hz
`define FAs_7  4054 //-- 2959.955 Hz
`define SOL_7  3827 //-- 3135.963 Hz
`define SOLs_7 3612 //-- 3322.438 Hz
`define LA_7   3409 //-- 3520.000 Hz
`define LAs_7  3218 //-- 3729.310 Hz
`define SI_7   3037 //-- 3951.066 Hz

//-- Octava: 8
`define DO_8   2867 //-- 4186.009 Hz
`define DOs_8  2706 //-- 4434.922 Hz
`define RE_8   2554 //-- 4698.636 Hz
`define REs_8  2411 //-- 4978.032 Hz
`define MI_8   2275 //-- 5274.041 Hz
`define FA_8   2148 //-- 5587.652 Hz
`define FAs_8  2027 //-- 5919.911 Hz
`define SOL_8  1913 //-- 6271.927 Hz
`define SOLs_8 1806 //-- 6644.875 Hz
`define LA_8   1705 //-- 7040.000 Hz
`define LAs_8  1609 //-- 7458.620 Hz
`define SI_8   1519 //-- 7902.133 Hz

//-- Octava: 9
`define DO_9   1433 //-- 8372.018 Hz
`define DOs_9  1353 //-- 8869.844 Hz
`define RE_9   1277 //-- 9397.273 Hz
`define REs_9  1205 //-- 9956.063 Hz
`define MI_9   1138 //-- 10548.082 Hz
`define FA_9   1074 //-- 11175.303 Hz
`define FAs_9  1014 //-- 11839.822 Hz
`define SOL_9  957 //-- 12543.854 Hz
`define SOLs_9 903 //-- 13289.750 Hz
`define LA_9   852 //-- 14080.000 Hz
`define LAs_9  804 //-- 14917.240 Hz
`define SI_9   759 //-- 15804.266 Hz

//-- Octava: 10
`define DO_10   717 //-- 16744.036 Hz
`define DOs_10  676 //-- 17739.688 Hz
`define RE_10   638 //-- 18794.545 Hz
`define REs_10  603 //-- 19912.127 Hz
`define MI_10   569 //-- 21096.164 Hz
`define FA_10   537 //-- 22350.607 Hz
`define FAs_10  507 //-- 23679.643 Hz
`define SOL_10  478 //-- 25087.708 Hz
`define SOLs_10 451 //-- 26579.501 Hz
`define LA_10   426 //-- 28160.000 Hz
`define LAs_10  403 //-- 29834.481 Hz
`define SI_10   380 //-- 31608.531 Hz
```
## notas.v: Descripción del hardware

El diseño de ejemplo tiene 8 canales independientes, cada uno emitiendo una nota distinta. El esquema es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-3.png)

y la descripción en verilog:

``` verilog
//-- notas.v
`include "divider.vh"

//-- Parameteros:
//-- clk: Reloj de entrada de la placa iCEstick
//-- chx: salidas de los 8 canales. Una nota por cada uno
module notas(input wire clk, output wire ch0, ch1, ch2, ch3, ch4, ch5, ch6, ch7);

//-- Parametros: valores del divisor
//-- Se definen como parametron para poder modificarlo desde el testbench
//-- para hacer pruebas
parameter N0 = `DO_4;
parameter N1 = `RE_4;
parameter N2 = `MI_4;
parameter N3 = `FA_4;
parameter N4 = `SOL_4;
parameter N5 = `LA_4;
parameter N6 = `SI_4;
parameter N7 = `DO_5;

//-- Generador de tono 0
divider #(N0)
  CH0 (
    .clk_in(clk),
    .clk_out(ch0)
  );

//-- Generador de tono 1
divider #(N1)
  CH1 (
    .clk_in(clk),
    .clk_out(ch1)
  );

//-- Generador de tono 2
divider #(N2)
  CH2 (
    .clk_in(clk),
    .clk_out(ch2)
  );

//-- Generador de tono 3
divider #(N3)
  CH3 (
    .clk_in(clk),
    .clk_out(ch3)
  );

divider #(N4)
  CH4 (
    .clk_in(clk),
    .clk_out(ch4)
  );

divider #(N5)
  CH5 (
    .clk_in(clk),
    .clk_out(ch5)
  );

divider #(N6)
  CH6 (
    .clk_in(clk),
    .clk_out(ch6)
  );

divider #(N7)
  CH7 (
    .clk_in(clk),
    .clk_out(ch7)
  );

endmodule
```

## Síntesis
Para sintetizarlo conectamos a la entrada de reloj la señal de 12Mhz de la IceStick y las salidas de los 8 canales por los pines 44, 45, 47, 48, 56, 60, 61 y 62 a los que se tiene acceso en la parte inferior derecha de la placa

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T18-notas/images/notas-1.png)

Igual que en el ejemplo anterior, usamos un único altavoz que vamos conectando al canal con la nota a escuchar (Selección manual)

Lo sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 9 / 96
|PLBs      | 45 / 160
|BRAMs     | 0 / 16

Lo cargamos en la fpga con:

    $ sudo iceprog tones.bin

Hacemos el mismo conexionado que en el capítulo anterior. Introducimos con un cable la señal del canal que queremos escuchar. En **este vídeo de youtube** se muestra el diseño en acción, seleccionándose los 8 canales manualmente para escuchar la escala:

[![Click to see the youtube video](http://img.youtube.com/vi/pkqy3xJv16k/0.jpg)](https://www.youtube.com/watch?v=pkqy3xJv16k)

## Simulación
No se simula la generación de las notas reales porque llevaría muchísimo tiempo. En vez de eso se comprueba que los 8 divisores están funcionando correctamente con los valores de 2, 3, 4, 5, 6, 7, 8 y 9.

``` verilog
module notas_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Salidas de los canales
wire ch0, ch1, ch2, ch3, ch4, ch5, ch6, ch7;


//-- Instanciar el componente y establecer el valor del divisor
//-- Se pone un valor bajo para simular (de lo contrario tardaria mucho)
notas #(2, 3, 4, 5, 6, 7, 8, 9)
  dut(
    .clk(clk),
    .ch0(ch0),
    .ch1(ch1),
    .ch2(ch2),
    .ch3(ch3),
    .ch4(ch4),
    .ch5(ch5),
    .ch6(ch6),
    .ch7(ch7)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;


//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("notas_tb.vcd");
  $dumpvars(0, notas_tb);

  # 200 $display("FIN de la simulacion");
  $finish;
end

endmodule
```

Para realizar la simulación ejecutamos:

    $ make sim

y el resultado es este:

Dib.

Se comprueba que los 8 canales están funcionando con sus divisores correspondientes, y que son independientes

## Referencias

[1] [Frecuencia de las notas musicales](http://latecladeescape.com/h/2015/08/frecuencia-de-las-notas-musicales). Por Vic (**La Tecla de Escape**) [Última consulta: 1-Sep-2015]