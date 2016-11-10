<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/T04-counter-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T04-counter)

## Introducción
Modelaremos nuestro **primer circuito secuencial**: un contador conectado a los LEDs. Los circuitos secuenciales, a diferencia de los combinacionales, **almacenan información**. El contador almacena un número que se incrementa con cada tic del reloj.

Esta es la pinta de nuestro componente. Se actualiza en cada flanco de subida del reloj, y su **salida data** es de **26 bits**.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-1.png)

**La señal de reloj** de la placa iCEstick es de **12Mhz**. Si hacemos un contador de sólo 4 bits y le conectamos a su entrada clk esta señal de 12Mhz, el resultado será que se incrementará tan rápido que siempre veremos los leds encendidos. Por ello utilizaremos un contador de 26 bits y usaremos los 4 más significativos para mostrarlos en los leds. 

## Descripción del hardware

El contador tiene **una entrada clk**, que es un cable, y una **salida data de 26  bits** que nos devuelve el valor del contador. Esta salida es un **registro de 26 bits**, que almacena el valor de la cuenta.

```verilog
//-----------------------------------
//-- Entrada: señal de reloj
//-- Salida: contador de 26 bits
//-----------------------------------
module counter(input clk, output [25:0] data);
wire clk;
    
//-- La salida es un registro de 26 bits, inicializado a 0
reg [25:0] data = 0;
    
//-- Sensible al flanco de subida
always @(posedge clk) begin
  //-- Incrementar el registro
  data <= data + 1;
end
endmodule
```

El funcionamiento del componente se describe dentro del bloque **always @(posedge clk)**, que indica que todo lo que está dentro de este bloque sólo se evaluará cada vez que llegue un **flanco de subida** en la señal clk. Cada vez que llega uno, se incrementa en una unidad el registro data (y saldrá por la salida del componente).

Cualquier contador, con independencia de su número de bits, se construye de esta manera. Si queremos que tenga 20 bits, sólo tenemos que cambiar el número 25 por 19 en las definiciones del registro data.

## Síntesis en la FPGA

Los 4 bits de mayor peso (data[25], data[24], data[23] y data[22]) los conectaremos a los pines de la fpga donde están los leds. El reloj de la iCEstick entra por el pin 21, que lo conectaremos a la señal de reloj clk de nuestro contador

![Image 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-2.png)

La asociación entre pines de nuestro componente y pines de la fpga se establece en el fichero **counter.pcf**:

    set_io data[25] 96
    set_io data[24] 97
    set_io data[23] 98
    set_io data[22] 99
    set_io clk 21

Realizamos la síntesis como siempre:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 14 / 96
|PLBs      | 6 / 160
|BRAMs     | 0 / 16

Para probarlo lo cargamos en la FPGA como siempre:

    $ sudo iceprog counter.bin

En este vídeo de youtube podemos ver el contador en funcionamiento:

[![Click to see the youtube video](http://img.youtube.com/vi/x9_OwUAtts4/0.jpg)](https://www.youtube.com/watch?v=x9_OwUAtts4)

## Simulación

El banco de pruebas está compuesto por 4 elementos (en paralelo) unidos por cables. El diagrama es el siguiente:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/counter-3.png)

Hay un generador de reloj que produce una señal cuadrada para incrementar el contador. La salida del contador se comprueba en dos componentes diferentes. Uno hace la comprobación inicial, verificando que inicialmente arranca desde 0.  El segundo tiene una variable interna que se incrementa con cada flanco de bajada del generador del reloj y su salida se comprueba con la del contdor, para verificar que efectivamente está contando. Como es un contador de 26 bits, no se comprueban todos los 67108864 valores, sino que la simulación se para transcurridas 100 unidades de tiempo.

El código en verilog es:

```verilog
//-- counter_tb.v
module counter_tb();
    
//-- Registro para generar la señal de reloj
reg clk = 0;
    
//-- Datos de salida del contador
wire [25:0] data;

//-- Registro para comprobar si el contador cuenta correctamente
reg [25:0] counter_check = 1;
    
//-- Instanciar el contador
counter C1(
  .clk(clk),
  .data(data)
);
    
//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;
    
//-- Comprobacion del valor del contador
//-- En cada flanco de bajada se comprueba la salida del contador
//-- y se incrementa el valor esperado
always @(negedge clk) begin
  if (counter_check != data)
    $display("-->ERROR!. Esperado: %d. Leido: %d",counter_check, data);
    
  counter_check <= counter_check + 1;
end
    
//-- Proceso al inicio
initial begin
    
  //-- Fichero donde almacenar los resultados
  $dumpfile("counter_tb.vcd");
  $dumpvars(0, counter_tb);
    
  //-- Comprobación del reset.
  # 0.5 if (data != 0)
          $display("ERROR! Contador NO está a 0!");
        else
          $display("Contador inicializado. OK.");

  # 99 $display("FIN de la simulacion");
  # 100 $finish;
 end
 endmodule
```

Es importante darse cuenta de que todos estos componentes están funcionando en paralelo, todos a la vez (el código NO es secuencial). Por eso, daría igual cambiar el orden de escritura de los elementos.

Para simular, ejecutamos:

    $ make sim

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T04-counter/images/T04-counter-sim-1.png)

Efectivamente el contador cuenta. En la imagen sólo se muestran los primeros valores, pero desplazando la imagen se pueden ver hasta el instante 100

## Ejercicios propuestos
* Cambiar el contador de 26 a 24 bits, para que se incremente más rápidamente

## Conclusiones
TODO



