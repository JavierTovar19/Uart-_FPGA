![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T25-uart-rx)

# Introducción
Diseñaremos una **unidad de recepción serie asíncrona**, que nos permita recibir datos tanto del PC como de otro dispositivo de comunicación serie. Obtendremos el componente final, listo para usar en nuestros diseños

Para probarla haremos dos ejemplos: uno que **saca los 4 bits menos segnificativos** del dato recibido por **los leds** de la ICEStick. Y otro que **hace eco de todos los caracteres recibidos**. Los caracteres recibidos se envían al transmisor serie para que lleguen de nuevo al PC. Esto nos permitirá comprobar que efectivamente hemos recibido el dato correctamente.

# Módulo UART-RX

La unidad de transmisión serie la encapsularemos dentro del **módulo uart-rx**

## Descripción de la interfaz

La unidad tiene 3 entradas y 2 salidas:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-1.png)

* **Entradas**:
    * **clk**: Reloj del sistema (12MHz en la ICEstick)
    * **rstn**: Reset negado. Cuando rstn es 0, se hace un reset síncrono de la unidad de transmisión
    * **rx**: Linea de recepción serie. Por donde llegan los datos en serie

* **Salidas**:
    * **rcv**: Notificación de carácter recibido. Es un pulso de 1 ciclo de ancho
    * **data**: Dato de recibido (8 bits)

## Cronograma

Inicialmente, cuando **la línea está en reposo** y no se ha recibido nada, la señal **rcv está a 0**. Al recibirse el **bit de start** por rx, el receptor comienza a funcionar, leyendo los siguientes bits y almacenándolos internametne en su registro de desplazamiento.  En el **instante t1**, cuando se ha recibido el **bit de stop**, **el dato se captura** y se saca por la **salida data**. En el siguiente ciclo de reloj, **instante t2** (en el cronograma el tiempo se ha exagerado para que se aprecie), aparece un pulso de un ciclo de reloj de anchura (exagerado también en el dibujo) que permita **capturar el dato en un registro externo**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-3.png)

**Esta interfaz es muy cómoda**. Para usar uart-rx en nuestros diseños, sólo hay que conectar la salida _data_ a la entrada de datos de un **registro** y la señal _rcv_ usarla como **habilitación**. El dato recibido se capturará automáticamente. Esta señal rcv también la podemos usar en los controladores para saber cuándo se ha **recibido un dato nuevo**.

## Diagrama de bloques

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

## Descripción en verilog

# Ejemplo 1: Visualizando en los leds
## Descripción 
## Simulación
## Síntesis y pruebas

# Ejemplo 2: eco
## Descripción
## Simulación
## Síntesis y pruebas

# Ejercicios propuestos
# Conclusiones
TODO
