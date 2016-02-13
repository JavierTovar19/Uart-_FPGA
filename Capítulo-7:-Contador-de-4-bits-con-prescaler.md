![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T07-contador-prescaler/images/counter4-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T07-contador-prescaler)

## Introducción
**Contador de 4 bits** conectado a los leds. Para que cuente más lentamente, la señal de reloj se pasa por un **prescaler de 22 bits**.  Se trata del mismo contador del capítulo 4, pero con un diseño mejorado. Para cambiar la frecuencia de cuenta sólo hay que cambiar los bits del prescaler (y no hace falta modificar los bits del contador ni reasignarlos en el fichero .pcf)



## Descripción del hardware

El contador tiene una **entrada de reloj clk** y una **salida de datos data** de 4 bits. También tiene un paráemtro N para indicar el número de bits del prescaler y establecer su frecuencia de funcionamiento. El código verilog es el siguiente:

```verilog
//-- counter4.v
module counter4(input clk, output [3:0] data);
wire clk;
reg [3:0] data = 0;
    
//-- Parametro para el prescaler
parameter N = 22;
    
//-- Reloj de salida del prescaler
wire clk_pres;
    
//-- Instanciar el prescaler de N bits
prescaler #(.N(N))
  pres1 (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
//-- Incrementar el contador en cada flanco de subida
always @(posedge(clk_pres)) begin
  data <= data + 1;
end
    
endmodule
```

Se podría haber definido un contador genérico en un fichero aparte e instanciarlo (igual que se ha hecho con el prescaler). Sin embargo se ha hecho así para mostrar un ejemplo de diseño jerárquico mezclado con un componente definido en un proceso.  Además, el contador es tan sencillo, que no merece la pena almacernarlo en un fichero separado.

El código se ha implementado a partir del dibujo  de la imagen 1

## Síntesis en la FPGA

La señal de reloj de 12Mhz entra por el pin 21 de la fpga, y la salida de datos se cada por los pines de los leds, del 99 al 96

Para sintentizarlo ejecutar el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 3 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Cargar en la FPGA con el comando:

    $ sudo iceprog counter4.bin

## Simulación
El banco de pruebas es similar al del capítulo 4, sin embargo, el proceso de comprobación del contador se ha modificado ligeramente. Ahora en vez de realizarse la comprobación en el flanco de bajada, se hace cuando hay algún cambio en la salida data del contador.  Esto se consigue poniendo a data en la lista de sensibilidad del proceso de comprobación:

```verilog
//-- counter4_tb.v
module counter4_tb();
    
//-- Registro para generar la señal de reloj
reg clk = 0;
    
//-- Datos de salida del contador
wire [3:0] data;
    
//-- Registro para comprobar si el contador cuenta correctamente
reg [3:0] counter_check = 0;
    
//-- Instanciar el contador, con prescaler de 1 bit (para la simulacion)
counter4 #(.N(1))
  C1(
    .clk(clk),
    .data(data)
  );

//-- Generador de reloj. Periodo 2 unidades
always #1 clk = ~clk;
    
//-- Proceso de comprobación. Cada vez que hay un cambio en
//-- el contador se comprueba con el valor de prueba
always @(data) begin
    
  if (counter_check != data)
    $display("-->ERROR!. Esperado: %d. Leido: %d",counter_check, data);
    
  counter_check = counter_check + 1;
end
    
//-- Proceso al inicio
initial begin
    
  //-- Fichero donde almacenar los resultados
  $dumpfile("counter4_tb.vcd");
  $dumpvars(0, counter4_tb);
    
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


El proceso de comprobación comienza con

    always @(data) begin

Esto significa que cada vez que ejecuta cada vez que hay un cambio en data

Para simular ejecutamos:

    $ make sim

El resultado es:

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T07-contador-prescaler/images/T07-counter4-simulation-1.png)

Vemos que el contador cuenta. Se ha utilizado un prescaler de 1 bit para que vaya más rápido en l simulación

## Ejercicios propuestos
* Cambiar el prescaler para que cuente más rápido
* Cambiar el prescaler para que cuente más lento

## Conclusiones
TODO
