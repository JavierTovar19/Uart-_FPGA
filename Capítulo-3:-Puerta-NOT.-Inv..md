<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/T03-inv-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T03-inv)

## Introducción
Modelaremos nuestro primer **circuito combinacional**: una **puerta inversora** (puerta NOT). 

Los circuitos combinacionales realizan operaciones con los bits de entrada y los sacan por las salidas. **No almacenan bits, sólo los transforman**. El más básico de todos es una puerta NOT, que tiene un bit de entrada y otro de salida. Por la salida siempre saca el inverso del bit de la entrada: El "1" lo transforma a "0"  y el "0" a "1".

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-1.png)

Nuestro componente lo llamaremos INV. Su entrada es A y su salida B

## inv.v: Descripción del hardware

La descripción es muy parecida al componente hola mundo setbit.v, pero ahora tenemos una entrada y una salida:

    //-- inv.v
    //-- El componente tiene una entrada (A) y una salida (B)
    module inv(input A, output B);
    
    //-- Tanto la entrada como la salida son "cables"
    wire A;
    wire B;
    
      //-- Asignar a la salida la entrada negada
      assign B = ~A;
    
    endmodule

A la salida B le asignamos la entrada A negada. Usamos el **operador ~** delante de A para negar (mismo operador que en el lenguaje C).

Como se trata de un circuito combinacional, que NO almacena información, **tanto A como B se define como cables**: la información se transmite por ellos, pero no se almacena.

## Síntesis en la FPGA

![Imagen 2](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-2.png)


<img src="" width="400" align="center">

## Simulación

![Imagen 3]()

![Imagen 4]()

## Ejercicios propuestos
* Ej1
* Ej2

## Conclusiones
TODO




