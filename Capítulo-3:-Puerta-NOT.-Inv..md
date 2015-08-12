<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/T03-inv-iCEstick-1.png" width="400" align="center">

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T03-inv)

## Introducción
Modelaremos nuestro primer **circuito combinacional**: una **puerta inversora** (puerta NOT). 

Los circuitos combinacionales realizan operaciones con los bits de entrada y los sacan por las salidas. **No almacenan bits, sólo los transforman**. El más básico de todos es una puerta NOT, que tiene un bit de entrada y otro de salida. Por la salida siempre saca el inverso del bit de la entrada: El "1" lo transforma a "0"  y el "0" a "1".

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T03-inv/images/inv-1.png)

Nuestro componente lo llamaremos INV. Su entrada es A y su salida B

## inv.v: Descripción del hardware

    //-- inv.v
    //-- El componente tiene una entrada (A) y una salida (B)
    module inv(input A, output B);
    
    //-- Tanto la entrada como la salida son "cables"
    wire A;
    wire B;
    
      //-- Asignar a la salida la entrada negada
      assign B = ~A;
    
    endmodule
