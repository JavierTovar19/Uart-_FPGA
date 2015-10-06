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

El diagrama de bloques completo del receptor se muestra en la siguiente figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-2.png)

### Ruta de datos

La **señal rx se registra**, para cumplir con las normas de diseño síncrono, y se introduce por el bit más significativo de un **registro de desplazamiento** de 10 bits. El desplazamiento se realiza cuando llega un pulso por la señal **clk_baud**, proveniente del **generador de baudios**. Este generador sólo funciona cuando la miroorden **bauden** está activada.

Un **contador de 4 bits** realiza la cuenta de los bits recibidos (cuenta cada pulso de clk_baud). Se pone a 0 con la microórden **clear**

Por último tenemos el **controlador**, que genera las microórdenes **baudgen**, **load**, **clear** y la señal de interfaz **rcv**. La señal load se activa para que el dato recibido se almacene en el **registro de datos de 8 bits**, de manera que se mantenga estable durante la recepción del siguiente carácter

### baudgen_rx: Generador de baudios para recepción

El receptor tiene su **propio generador de baudios** que es **diferente al del transmisor**. En el transmisor, al activar su generador con la microorden bauden, emite inmediatamente un pulso. Sin embargo, en el receptor, se emite **en la mitad del periodo**. De esta forma se garantiza que el dato se lee en la mitad del periodo, donde es **mucho más estable** (y la probabilidad de error es mejor)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-4.png)

En el cronograma se puede ver cómo al recebir el **flanco de bajada** del bit de start **bauden se activa**, para que comience a funcionar el reloj del receptor. Sin embargo, hasta que **no ha alcanzado la mitad del periodo de bit no se pone a 1**. A partir de entonces, los pulsos coinciden con la mitad de los periodos de los bits recibidos 

### Controlador

El controlador está modelado como una máquina de **4 estados**:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T25-uart-rx/images/uart-rx-5.png)

Los estados son:

* **IDLE**: Estado de reposo. Esperando a recibir el bit de start por rx.  En cuanto se recibe se pasa al siguiente estado
* **RCV**: Recibiendo datos. Se activa **el temporizador de bits** mediante la microorden **baudgen** y se van recibiendo todos los bits, que se almacenan en **el registro de desplazamiento**. Cuando se han recibido 10 bits (1 de start + 8 de datos + 1 de stop) la salida del **contador** (**bitc**) estará a 10 y se pasa al siguiente estado
* **LOAD**: Almacenamiento del dato recibido. Se activa la microorden **load** para guardar el dato recibido (8 bits) en el **registro de datos**
* **DAV**: (Data AVailable). Señalización de que existe un **dato disponible**. Se pone a uno la señal **rcv** para que los circuitos externos puedan capturar el dato

## Descripción en verilog

La **unidad de recepción** se compone de dos ficheros: el generador de baudios de recepción (_baudgen_rx.v_) y el propio receptor (_uart_rx.v_)

### baudgen_rx.v

Este componente es similar al generador de baudios del transmisor, pero una vez habilitado el pulso se envía transcurrida la mitad del periodo

El código verilog es el siguiente:

```verilog
//-- Fichero: baudgen_rx.v
`include "baudgen.vh"

//-- ENTRADAS:
//--     -clk: Senal de reloj del sistema (12 MHZ en la iceStick)
//--     -clk_ena: Habilitacion. 
//--            1. funcionamiento normal. Emitiendo pulsos
//--            0: Inicializado y parado. No se emiten pulsos
//
//-- SALIDAS:
//--     - clk_out. Señal de salida para lograr la velocidad en baudios indicada
//--                Anchura de 1 periodo de clk. SALIDA NO REGISTRADA
module baudgen_rx(input wire clk,
               input wire clk_ena, 
               output wire clk_out);

//-- Valor por defecto de la velocidad en baudios
parameter M = `B115200;

//-- Numero de bits para almacenar el divisor de baudios
localparam N = $clog2(M);

//-- Valor para llegar a la mitad del periodo
localparam M2 = (M >> 1);

//-- Registro para implementar el contador modulo M
reg [N-1:0] divcounter = 0;

//-- Contador módulo M
always @(posedge clk)

  if (clk_ena)
    //-- Funcionamiento normal
    divcounter <= (divcounter == M - 1) ? 0 : divcounter + 1;
  else
    //-- Contador "congelado" al valor maximo
    divcounter <= M - 1;

//-- Sacar un pulso de anchura 1 ciclo de reloj si el generador
//-- esta habilitado (clk_ena == 1)
//-- en caso contrario se saca 0
assign clk_out = (divcounter == M2) ? clk_ena : 0;

endmodule
```

### uart_rx.v

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
