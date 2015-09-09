![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-2.png)
[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T20-serialcomm-1)

## Introducción

Una forma de intercambiar datos entre dos sistemas digitales es mediante las **comunicaciones serie asíncronas**. Durante los siguientes capítulos aprenderemos a diseñar nuestra propia **unidad de comunicaciones serie asíncronas** (UART) que nos permitirá **comunicar la FPGA con el PC y otras placas micocontroladoras** (como [Arduino](http://www.arduino.org/), [BQ Zum](http://www.bq.com/es/placa-zum-bt), etc).

En este capítulo empezaremos por lo más simple: reenviar al PC todo lo recibido tirando un cable en la FPGA  que una el canal de recepción (Rx) con el de transmisión (Tx). Nos aseguramos así que el **nivel físico está controlado**.

## Comunicaciones serie asíncronas

 Este tipo de comunicación tiene la ventaja de que **sólo se necesita un hilo** para cada sentido: del transmisor al receptor. El pin del transmisor, por donde salen los datos digitales, se denomina **Tx**. Y el pin del receptor es **Rx**. Los bits se envían secuencialmente, uno detrás de otro.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-1.png)

Las comunicaciones pueden ser en **ambas direcciones simultáneamente** (full duplex). En ese caso habrá **dos hilos**, y cada circuito hará tanto de transmisor (pin Tx) como de receptor (Rx), como se muestra en la figura.

Los pines **tx** y **rx** forman parte del denominado **puerto serie**. Y se utiliza muchísimo para conectar el PC con diferentes circuitos. Por ejemplo para cargar programas en las **placas Arduino**, o comunicarte con la [BQ Zum](http://www.bq.com/es/placa-zum-bt) a través del **bluetooth-serie**.

## Comunicaciones serie en la placa iCEstick

La placa iCEstick incluye un **conversor USB-serie** (chip FTDI) por el que llegan las señales de **Tx** y **Rx** a la FPGA, a través de los **pines 8 y 9**. De esta forma tenemos **conexión directa** entre el **puerto serie del PC** y la **FPGA**, y podremos implementar nuestra UART para la transferencia de datos.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-3.png)

Los **puertos serie**, además de Tx y Rx para la transferencia de datos, **incorporan otras señales de control**. Dos muy empleadas se denominan **RTS** y **DTR**, y van en sentido **PC -> FPGA**.  Son 2 bits que podemos usar para próposito general. Por ejemplo, en las placas compatibles Arduino el PC usa una de estas señales para hacer _reset_ y comenzar así la carga de programas. 

## Comunicaciones serie en el PC

-> Permisos
-> Gtkterm

## Experimento 1: Conectando tx y rx directamente


## Experimento 2: Conectando tx y rx mediante cable externo

## Ejercicios propuestos

## Conclusiones