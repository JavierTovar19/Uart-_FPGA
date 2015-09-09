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

Para trabajar con la iCEstick, en estos tutorial, lo que nos importa es lo resumido en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/blob/master/tutorial/T20-serialcomm-1/images/serialcomm-4.png)

Para nosotros será como si **nuestro PC tuviese un puerto serie nativo** cuyas señales Tx, Rx, DTR, RTS están conectadas directamente a los pines de la FPGA. A partir de ahí empezaremos a "jugar" :-)

## Comunicaciones serie en el PC

Para **acceder al puerto serie del PC** tenemos que instalar un **terminal de comunicaciones**. Con él podremos enviar datos a la FPGA, actuar sobre las señales DTR y RTS y recibir datos.  El que utilizaremos en este tutorial es el [gtkterm](https://github.com/martinxyz/gtkterm), aunque podemos utilizar cualquier otro, como por ejemplo el monitor serie del entorno Arudino.

En ubuntu se instala muy fácilmente con:

    $ sudo apt-get install gtkterm

Para **tener permisos de acceso al puerto serie**, tenemos que meter a nuestro usuario dentro del **grupo dialout**, ejecutando el comando:

    $ sudo adduser $USER dialout

Es necesario **salir de la sesión y volver a entrar** para que tenga efecto. Si instalamos el entorno de arduino, los permisos se nos crearán automáticamente sin necesidad de usar ningún comando

Conectamos la placa iCEstick al USB y ejecutamos el gtkterm:

    $ gtkterm

Si es la primera vez que lo arrancamos, posiblemente obtengamos un mensaje de error. No hay problema. Primero tenemos que configurar el puerto serie pinchando en la opción _configuration / port_ y se nos abrirá una ventana de configuración. Establecemos como puerto serie el **/dev/ttyUSB1**, a la velocidad de **115200**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/gtkterm-screenshot-1.png)

**Nota**: En ubuntu 15.04, al pinchar la iCEstick nos aparecen 2 puertos serie. El que nos permite acceder a la FPGA como puerto serie es el /dev/ttyUSB1

Le damos al OK. Para recordar esta configuración pinchamos en la opción _configuration / save configuration_ y le damos al ok.

El terminal tiene esta pinta:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/gtkterm-screenshot-2.png)

## Experimento 1: Comprobando tx, rx, dtr y rts
Comprobaremos que todas las señales funcionan correctamente. Para ello haremos un circuito que simplemente haga un **cableado entre señales**: las señales **rts y dtr se conectan a los pines de los leds**. Esto nos permite visualizar su estado, y cambiarlo desde el PC. La señal **RX se conecta físicamente a TX**, de forma que todos los caracteres recibidos se reenvían de nuevo al PC, a nivel físico (no hay procesamiento del carácter en la fpga, es un simple cable)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T20-serialcomm-1/images/serialcomm-5.png)

La descripción en Verilog es inmediata:

```verilog
//-- echowire1.v
module echowire1(input wire dtr,
                 input wire rts,
                 input wire rx,
                 output wire tx,
                 output wire D1,
                 output wire D2);

//-- Sacar senal dtr y rts por los leds
assign D1 = dtr;
assign D2 = rts;

//-- Conectar rx a tx para que se haga un eco "cableado"
assign tx = rx;

endmodule
```

Lo sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 0 / 160
|BRAMs     | 0 / 16

Para probarlo lanzamos el **gtkterm**. Presionando la **F7** cambiamos el estado de **DTR** y por tanto, cambia el **led**. Con la tecla **F8** hacemos lo mismo pero con la señal **RTS**.

Cualquier carácter que pulsemos se enviará a la FPGA y se hará eco. El terminal saca por pantalla todo lo recibido. **El resultado es que veremos en la pantalla todo lo que escribimos**.

## Experimento 2: Conectando tx y rx mediante cable externo

## Ejercicios propuestos

## Conclusiones