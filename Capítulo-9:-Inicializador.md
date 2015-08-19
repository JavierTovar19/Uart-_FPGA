![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T09-inicializador)

## Introducción
Muchos circuitos digitales **necesitan inicializarse** antes de comenzar a trabajar normalmente. Su funcionamiento se divide en un **estado de arranque**, donde se inicializan los valores de los registros y un estado de **régimen permanente** donde se realiza la función para la que han sido diseñados.

Para lograr esto necesitamos un circuito de inicialización que nos genere una **señal escalón**: que inicialmente esté a cero y al llegar el primer flanco de reloj pase a 1 y permanezca a 1 durante todo el funcionamiento de la máquina.

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-2.png)

Esto se implementa de una manera muy sencilla utilizando un **registro de 1 bit** [1] al que se le cablea un "1" en su entrada. Inicialmente el registro estará a 0. Al llegar el primer flanco de reloj, se captura el 1 y se saca por su salida, **generando el flanco de subida para realizar la inicialización**. Para el resto de ciclos de reloj esta señal siempre estará a 1

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-3.png)

## Init.v: Descripción del hardware

Al tratarse de un registro, lo podemos implementar igual que en el capítulo anterior. Esta es la **implementación natural**. Otra posibilidad es hacer una **implementación optimizada**, usando menos código verilog

### Implementación natural

Es la implementación más lógica. Se parte de un registro genérico de 1 bit, que ya sabemos cómo se modela y simplemente cableamos su entrada a 1 y sacamos su salida hacia fuera:

    //-- init.v (implementación natural)
    //-- Entrada: cable del reloj
    //-- Salida: Cable con la señal de inicialización
    module init(input wire clk, output wire ini);
    
    //-- Cable de entrada el registro de 1 bit
    wire din;
    
    //-- Salida del registro de 1 bit (inicializado a 0) (solo para simulacion)
    //-- En la sintesis siempre estará a 0
    reg dout = 0;

    //-- Registro genérico: en flanco de subida se captura la entrada
    always @(posedge(clk))
      dout <= din;
    
    //-- Cablear la entrada a 1
    assign din = 1;
    
    //-- Conectar la salida del registro a la señal ini
    assign ini = dout;
    
    endmodule

Esta implementación es muy sencilla y se entiende muy bien. Se ve claramente que es un registro con un "1" cableado por la entrada, sin embargo es muy "verbosa". Hay que escribir mucho código

### Implementación optimizada

Podemos hacer lo mismo pero con mucho menos código implementando directamente un registro que asigne siempre un "1" a su salida:

    //-- init.v  (version optimizada)
    module init(input wire clk, output ini);

    //-- Registro de 1 bit inicializa a 0 (solo para simulacion)
    //-- Al sintetizarlo siempre estará a cero con independencia del valor al que lo pongamos
    reg ini = 0;
    
    //-- En flanco de subida sacamos un "1" por la salida
    always @(posedge(clk))
      ini <= 1;
    
    endmodule

## Síntesis en la FPGA

Para probarlo en la fpga simplemente vamos a **conectar la salida ini a un led**. Al cargarlo en la FPGA sólo veremos cómo se enciende un led, sin embargo con ello tenemos nuestro circuito validado y listo para inicalizar los diseños de los capítulos siguientes

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T09-inicializador/images/init-1.png)

Lo sintetizamos con:

    $ make sint

Los recursos utilizados son:



## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO

[1] Un registro de 1 bit es en realidad un **flip-flop**. Sin embargo prefiero llamarlo registro de 1 bit para generaliza. Su implementación en Verilog es la misma que la de un registro de N bits

