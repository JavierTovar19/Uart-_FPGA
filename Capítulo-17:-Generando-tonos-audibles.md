<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/T17-tones-icestick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T17-tones)

## Introducción

Utilizaremos el **divisor de frecuencia** para **generar tonos** de 1, 2, 3 y 4Khz que se pueden escuchar conectando un altavoz o un zumbador a su salida

## Generación de un tono
En nuestro sistema digital tenemos una señal de reloj de entrada f_in, que en la iCEstick es de 12MHz. Para generar un tono de frecuencia f_t tenemos que calcular **el valor del divisor** usando la **fórmula**:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/T17-formula-divisor.png)

Así, para generar un **tono de 1Khz** tenemos que aplicar un divisor de valor:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/T17-calculo-divisor-1Khz.png)

En la siguiente tabla se muestran los valores de los divisores para generar tonos de 1, 2, 3 y 4KHz:

| Frecuencia del Tono   |  Valor del divisor
|-----------------------|---------------------
|  1KHz                 |  12000
|  2KHz                 |  6000
|  3KHz                 |  4000
|  4KHz                 |  3000

**Los divisores tienen que ser números enteros**. De manera que si al dividir f_in entre f_t **se obtiene un valor decimal, tendremos que redondearlo**

En el fichero **divisor.vh** se definen estos pares como constantes, para usarlos más fácilmente en el código verilog:
``` verilog
//-- divisor.vh
//-- Megaherzios  MHz
`define F_4MHz 3
`define F_3MHz 4
`define F_2MHz 6
`define F_1MHz 12
    
//-- Kilohercios KHz
`define F_4KHz 3_000
`define F_3KHz 4_000
`define F_2KHz 6_000
`define F_1KHz 12_000
    
//-- Hertzios (Hz)
`define F_2Hz   6_000_000
`define F_1Hz   12_000_000
```

## tones.v: Descripción del hardware
Como ejemplo, **generaremos 4 tonos simultáneamente, de 1, 2, 3 y 4KHz**. Cada canal está compuesto por su correspondiente divisor de frecuencia, con los valores calculados anteriormente

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/tones-2.png)

La descripción del componente en Verilog es:

``` verilog
//-- Incluir las constantes del modulo del divisor
`include "divider.vh"
    
//-- Parameteros:
//-- clk: Reloj de entrada de la placa iCEstick
//-- data: Valor del contador de segundos, a sacar por los leds de la iCEstick
module tones(input wire clk, output wire ch0, ch1, ch2, ch3);
    
//-- Parametro del divisor. Fijarlo a 1Hz
//-- Se define como parametro para poder modificarlo desde el testbench
//-- para hacer pruebas
parameter F0 = `F_1KHz;
parameter F1 = `F_2KHz;
parameter F2 = `F_3KHz;
parameter F3 = `F_4KHz;
    
//-- Generador de tono 0
divider #(F0)
  CH0 (
    .clk_in(clk),
    .clk_out(ch0)
  );
    
//-- Generador de tono 1
divider #(F1)
  CH1 (
    .clk_in(clk),
    .clk_out(ch1)
  );
    
//-- Generador de tono 2
divider #(F2)
  CH2 (
    .clk_in(clk),
    .clk_out(ch2)
  );
    
//-- Generador de tono 3
divider #(F3)
  CH3 (
    .clk_in(clk),
    .clk_out(ch3)
  );
    
endmodule
```

Se **definen los parámetros F0, F1, F2 y F3** para establecer las frecuencias. De esta manera se pueden cambiar desde el banco de pruebas para comprobar que funcionan.

A continuación **se instancian los 4 divisores**, cada uno con su frecuencia y conectados a la señal de reloj de entrada clk. La salida de cada divisor se saca por las salidas ch0, ch1, ch2 y ch3 del componente

## Síntesis

Para sintetizarlo conectamos a la entrada de reloj la señal de 12Mhz de la IceStick y las salidas de los 4 canales por los pines 44, 45, 47 y 48 a los que se tiene acceso en la parte inferior derecha de la placa

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/tones-1.png)

A cada canal se le puede conectar un altavoz externo, pero para hacer pruebas es más fácil utilizar un único **altavoz** (o zumbador) y conectarlo manualmente al canal que se quiere escuchar

Para sintetizar ejecutamos el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 7 / 96
|PLBs      | 22 / 160
|BRAMs     | 0 / 16

## Pruebas

Lo cargamos en la FPGA con el comando:

    $ sudo iceprog tones.bin

Alimentamos el zumbador desde los pines de 3.3v y gnd de la IceStick. El esquema es el siguiente: 

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/tones-3.png)

Como zumbador hemos utilizado el que viene en **mi primer kit de robótica**[1]

Introducimos con un cable la señal del canal que queremos escuchar. En este vídeo de youtube se muestra el diseño en acción:

[![Click to see the youtube video](http://img.youtube.com/vi/uMFJ4ET1wcg/0.jpg)](https://www.youtube.com/watch?v=uMFJ4ET1wcg)

## Simulación

No se simula la generación de los tonos de 1,2,3 y 4Khz porque llevaría muchísimo tiempo. En vez de eso se comprueba que los 4 divisores están funcionando correctamente con los valores de 3,5,7 y 10

``` verilog
//-- tones_tb.v
module tones_tb();

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Salidas de los canales
wire ch0, ch1, ch2, ch3;

//-- Instanciar el componente y establecer el valor del divisor
//-- Se pone un valor bajo para simular (de lo contrario tardaria mucho)
tones #(3, 5, 7, 10)
  dut(
    .clk(clk),
    .ch0(ch0),
    .ch1(ch1),
    .ch2(ch2),
    .ch3(ch3)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("tones_tb.vcd");
  $dumpvars(0, tones_tb);

  # 100 $display("FIN de la simulacion");
  $finish;
end

endmodule
```

Para realizar la simulación ejecutamos:

    $ make sim

y el resultado es este:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T17-tones/images/T17-tones-sim1.png)

Se puede comprobar que los 4 canales están funcionando con sus divisores correspondientes, y que son independientes

## Ejercicios propuestos
* Optimizar el diseño para que se ocupen menos PLBs. Para ello hay que sacar "factor común" en los divisores. La señal de reloj se pasará primero por un divisor común a todos los canales, y luego habrá 1 divisor para cada canal, pero de menos bits.

## Conclusiones
TODO

## Referencias
* [1]: http://www.bq.com/es/kit-de-robotica


