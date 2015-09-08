(Dibujo) (dibujo portatil - icestick)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T20-serialcomm-1)

## Introducción

Una forma de intercambiar datos entre dos sistemas digitales es mediante las **comunicaciones serie asíncronas**. Durante los siguientes capítulos aprenderemos a diseñar nuestra propia **unidad de comunicaciones serie asíncronas** (UART) que nos permitirá comunicar la FPGA con el pc y otras placas micocontroladoras (como [Arduino](http://www.arduino.org/), [BQ Zum](http://www.bq.com/es/placa-zum-bt), etc).

En este capítulo empezaremos por lo más simple: reenviar al PC todo lo recibido tirando un cable en la FPGA  que una el canal de recepción (Rx) con el de transmisión (Tx). Nos aseguramos así que el **nivel físico está controlado**.

## Comunicaciones serie asíncronas

 Este tipo de comunicación tiene la ventaja de que **sólo se necesita un hilo** para cada sentido: del transmisor al receptor. El pin del transmisor, por donde salen los datos digitales, se denomina **Tx**. Y el pin del receptor es **Rx**. Los bits se envían secuencialmente, uno detrás de otro.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-1.png)

Las comunicaciones pueden ser en **ambas direcciones simultáneamente** (full duplex). En ese caso habrá **dos hilos**, y cada circuito hará tanto de transmisor (pin Tx) como de receptor (Rx), como se muestra en la figura.

Al conjunto de pines **tx** y **rx** se le denomina **puerto serie**. Y se utiliza muchísimo para conectar el PC con diferentes circuitos. Por ejemplo para cargar programas en las **placas Arduino**, o comunicarte con la [BQ Zum](http://www.bq.com/es/placa-zum-bt) a través del **bluetooth-serie**.

## Comunicaciones serie en la placa iCEstick

(Dibujo)

## Comunicaciones serie en el PC

-> Permisos
-> Gtkterm

## Experimento 1: Conectando tx y rx directamente


## Experimento 2: Conectando tx y rx mediante cable externo

## Ejercicios propuestos

## Conclusiones