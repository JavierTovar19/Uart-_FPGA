(Dibujo)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T20-serialcomm-1)

## Introducción

Una forma de intercambiar datos entre dos sistemas digitales es mediante las **comunicaciones serie asíncronas**. Tienen la ventaja de que **sólo se necesita un hilo** para cada sentido: del transmisor al receptor. El pin del transmisor por donde salen los datos digitales se denomina **Tx**. Y el pin del receptor es **Rx**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-1.png)

Las comunicaciones pueden ser en **ambas direcciones simultáneamente** (full duplex). En ese caso habrá **dos hilos**, y cada circuito hará tanto de transmisor (pin Tx) como de receptor (Rx), como se muestra en la figura.

