![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T22-syncrules)

## Introducción
En este capítulo se explican algunas **reglas de diseño síncrono** [1] para la **creación de circuitos digitales complejos**. Su aplicación nos evitará problemas según crece la complejidad de nuestros circuitos. Las aplicaremos para **mejorar el transmisor serie** que se esbozó en el capítulo anterior

## Sistemas síncronos
Los **sistemas digitales complejos son síncronos**: existe **una señal de reloj** (clk) que marca los tiempos en los que el resto de señales cambian. Es un **reloj "corazón"**, al ritmo de cuyos pulsos se transportan y modifican los bits. En la placa iCEstick, nuestro reloj es de **12Mhz**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/sync-corazon.png)

A la hora de diseñar circuitos digitales síncronos conviene tener **una serie de reglas en la cabeza**, que nos ahorrarán muchos problemas, sobre todo cuando se hacen diseños más complejos. Algunas de estas reglas **se han violado** en los ejemplos presentados hasta ahora. Se trata de ejemplos sencillos, que nos han permitido introducir conceptos y hacer pruebas rápidamente en nuestra FPGA.  Es el momento de aprender **reglas de diseño más potentes**, y aplicarlas

Pero antes vamos a conocer un poco más el **origen de los problemas**

## Problemas en los circuitos digitales

### La causa de los males: el retardo

Los problemas en los diseños están causados por el **retardo en las señales**, debido a la propagación y al tiempo de procesado de las puertas lógicas. En el mundo digital diseñamos la funcionalidad asumiendo que las señales digitales son perfectas, que cambian instantáneamente y todos los bits a la vez.

La realidad es diferente. Así por ejemplo, si estamos usando una **puerta NOT** y la simulamos, vemos el caso ideal, en el que la salida B es la negada de la A en todo momento.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/retardo-not.png)

En la realidad **la salida B está retrasada con respecto a la entrada** (además de que el cambio de 0 a 1 no es instantáneo). ¿Y por qué es un problema? Porque mirando el cronograma, vemos que hay un instante de tiempo donde **NO SE CUMPLE que B sea el negado de A**. Sucede que A=1 y B=1, lo que significa que **NO SE CUMPLEN LAS ECUACIONES BOOLEANAS de la lógica digital** en ese intervalo de tiempo (esto hay que tenerlo en la cabeza). Además, estos retrasos originan **pulsos espúreos transitorios** (glitches)

### Pulsos espúreos (Glitches)

**Cada señal** de un circuito digital **tiene un retraso diferente** que dependerá de la longitud de sus cables y de las puertas que atraviese. Así, dos señales de entrada llegarán a la misma puerta en tiempos diferentes. Esto provoca la **aparición de pulsos espúreos**, que desaparecen en el régimen permanente.

Vamos a ver su efecto en un circuito combinacional sencillo: una **puerta XOR** de dos entradas.  Las dos entradas están a 0 (y por tanto la salida permanece a 0). En un instante las señales de entrada A y B cambian a 1. Idealmente, lo que se obtiene por la salida es también 0, ya que 1 xor 1 es igual a 0.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/glitches-xor.png)

Sin embargo, en este ejemplo, debido a los retardos, la señal B ha llegado un poco después que la A. Esto hace que haya un momento en el que A = 1 y B = 0, por lo que la salida es C = 1.  Finalmente, cuando B = 1 la salida se pone a su valor estable: C = 0.

El resultado es que **ha aparecido un pulso** en una señal que **se suponía que debería estar siempre a 0**

Imaginemos lo que puede pasar si tenemos este circuito conectado a **la entrada de reloj de un contador**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/xor-contador.png)

Debido al pulso espúreo, **¡el contador hará una cuenta falsa!** ¡Contará cuando NO tiene que contar!. El diseñador, por más que mire y remire el diseño digital, verá que está todo correcto. Y al simularlo idealmente todo funcionará bien... pero al cargarlo en la FPGA: ¡zas! Contará mal. **Estos errores son muy difíciles de detectar**

Para evitar estos problemas es por lo que usamos **las reglas de diseño síncrono**

## Reglas de diseño síncrono

Las reglas son:

### Regla 1: Un único reloj para gobernarlos a todos

**TODAS las entradas de reloj se deben conectar directamente a un único reloj**, común a todos. Esta regla se ha violado en casi todos los ejemplos de este tutorial hasta ahora, para que fuesen sencillos. A partir de ahora todos los ejemplos respetarán esta regla

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-2-unico-reloj.png)

Esta regla nos **prohibe conectar a la entrada del reloj cualquier otra cosa que NO sea el reloj del sistema**. De forma que no podemos conectar ningún circuito combinacional (por ej. la puerta xor de la explicación de los pulsos espúreos) ni tampoco secuencial (por ejemplo la salida de un contador para usarlo como divisor). En las siguientes secciones veremos qué añadir a los circuitos para que cumplan esta regla.

### Regla 2: Sensibilidad al mismo flanco: ¡Todos a una, fuenteovejuna!
Todos los elementos que lleven reloj, serán sensibles al mismo flanco. Bien al de subida o al de bajada, es indiferente, pero **todos sensibles al mismo**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-1-mismo-flanco.png)

### Regla 3: Antes de entrar a un circuito combinacional, pase por registro, por favor

**Todas las entradas de los circuitos combacionales** deben estar **conectadas a salidas de circuitos secuenciales**, sincronizadas con el reloj del sistema, o bien a **otros circuitos combinacionales que cumplan esta regla**. Es decir, que **cualquier entrada** de un circuito combinacional **tiene que ser capturada** antes por un registro. Las entradas que cumplen esta regla se denominan **entradas sincronizadas**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/c5baf5771fea985001de3c0976583c5f0f3b1e9f/tutorial/T22-syncrules/images/regla-3-entradas-sincronizadas.png)

### Regla 4: Antes de entrar a un circuito secuencial, pase por registro por favor
**Todas las entradas de los circuitos secuenciales** deben provenir de **las salidas de otros circuitos secuenciales** o bien de **combinacionales que cumplan la regla 3**. Es decir, que incluso para entrar en los circuitos secuenciales, es necesario que **las señales estén sincronizadas**.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-4-entradas-sec-sincronizadas.png)

Combinando la regla 3 y la 4, llegamos a la conclusión de que **cualquier entrada a nuestro circuito síncrono tiene que estar registrada**.

### Regla 5: Salidas de un combinacional: Sólo a entradas de otro combinacional, entradas síncronas o salidas del circuito síncrono

Esta regla nos indica **dónde ponemos conectar la salida de un circuito combinacional**. Bien a la **entrada de otro combinacional**, a **la entrada síncrona de un secuencial** o bien como **salida directa de nuestro circuito**. Queda terminantemente prohibido conectarlas a las entradas del propio combinacional (nada de realimentaciones directas) o a la entrada de la señal de reloj

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/regla-5-salidas-combinacionales.png)

## Mejorando el transmisor serie

Rediseñaremos los ejemplos del transmisor serie para cumplir las reglas

### Ejemplo 1: txtest.v

Partimos del ejemplo baudtx, donde se envía el carácter "K" con el flanco de subida de la señal DTR

### Aplicando las reglas de diseño síncrono

El esquema del circuito original es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/baudtx-1-errors.png)

Observamos varias violaciones de las reglas del diseño síncrono:

1. **No hay un reloj global único** (no se respeta la regla 1). La señal de reloj no entra directamente en el registro de desplazamiento. La señal proveniente del divisor **NO puede entrar directamente por la entrada de reloj** sino que debe haber una entrada nueva sincronizada. Para que el divisor funcione correctamente, el ancho del pulso de salida deberá ser de 1 periodo de longitud.

2. **La entrada load NO está registrada** (se viola la regla 3)

3. La **salida TX** proviene de un circuito combinacional y se puede sacar directamente sin violar ninguna regla. Sin embargo al estar conectada a un **bus asíncrono**, cualquier pulso espúreo enviado será interpretado como una dato y provocará un **error en las comunicaciones**. Por ello es más seguro registrar esta salida.

### Baudgen.v: Modificación del divisor
Diseñaremos un **divisor específico para usar en el transmisor**. Puesto que la salida del divisor NO se puede introducir directamente por la entrada de reloj del registro de desplazamiento del transmisor (prohibido por las reglas de diseño síncrono), se conectará a una entrada nueva síncrona. Sin embargo, será necesario que el **pulso sea sólo de 1 periodo de longitud**

Como mejora, pondremos **una señal de habilitación del divisor** (**clk_ena**) de forma que cuando clk_ena = 1 se producirán los pulsos y en caso contrario la salida (clk_out) permanecerá a 0

El esquema del circuito es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/baudgen-diagram.png)

y la descripción en verilog:

```verilog
//-- Fichero baudgen.v
`include "baudgen.vh"

//-- ENTRADAS:
//--     -clk: Senal de reloj del sistema (12 MHZ en la iceStick)
//--     -clk_ena: Habilitacion. 
//--            1. funcionamiento normal. Emitiendo pulsos
//--            0: Inicializado y parado. No se emiten pulsos
//
//-- SALIDAS:
//--     - clk_out. Señal de salida que marca el tiempo entre bits
//--                Anchura de 1 periodo de clk. SALIDA NO REGISTRADA
module baudgen(input wire clk,
               input wire clk_ena, 
               output wire clk_out);

//-- Valor por defecto de la velocidad en baudios
parameter M = `B115200;

//-- Numero de bits para almacenar el divisor de baudios
localparam N = $clog2(M);

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
assign clk_out = (divcounter == 0) ? clk_ena : 0;

endmodule
```
El funcionamiento del nuevo divisor se puede ver en **este cronograma**. En el ciclo siguiente a activarse _clk_enable_ aparece un pulso de 1 periodo de longitud, que se repite cada M ciclos de reloj. Cuando _clk_ena_ es 0, no se generan pulsos

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/34459e41edf1a66b8319585446144ad7ab098da1/tutorial/T22-syncrules/images/baudgen-chronogram.png)

### txtest.v: Modificación del transmisor

El nuevo esquema se muestra a continuación. Además de **cumplir con las reglas del diseño síncrono**, se ha añadido **una mejora**: desde que se recibe el flanco hasta que se empieza a transmitir **sólo transcurre 1 ciclo de reloj**. Antes había que esperar a que además llegase un 1 en la señal de los baudios, por lo que este tiempo variaba. Para lograrlo se controla _baudgen_ mediante su nueva entrada de habilitación: _clk_ena_.  Cuando no hay transmisión de información, la señal de baudios está deshabilitada.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest-diagram.png)

El nuevo **registro de desplazamiento**, en verilog es el siguiente:

```verilog
always @(posedge clk)
  //-- Modo carga
  if (load_r == 0)
    shifter <= {"K",2'b01};

  //-- Modo desplazamiento
  else if (load_r == 1 && clk_baud == 1)
    shifter <= {1'b1, shifter[9:1]};
```

Ahora **depende de la señal de reloj global**, para cumplir con la regla 1. Además es en **flanco de subida**, porque el resto de circuitos los hemos hecho así y por la regla 2 todos tienen que ser sensibles al mismo flanco

La señal de carga usada es _load_r_, que es la señal _load_ registrada:

```verilog
always @(posedge clk)
  load_r <= load;
```
El registro desplaza los bits hacia la derecha cuando no estamos en modo carga (load_r == 1) y **la señal clk_baud está a 1**. Por este motivo, la señal _clk_baud_ **sólo puede estar a 1 durante 1 ciclo de reloj**. Si el ancho del pulso fuera mayor, se realizaría un desplazamiento con cada flanco de reloj global, perdiéndose la temporización.

La **salida TX de salida está registrada**, para garantizar que **entra totalmente limpia en el bus asíncrono** y que no se envían pulsos espúreos.

```verilog
always @(posedge clk)
  tx <= (load_r) ? shifter[0] : 1;
```

El **código completo** se muestra a continuación:

```verilog
//-- Fichero: txtest.v
`default_nettype none

`include "baudgen.vh"

//--- Modulo que envia un caracter cuando load esta a 1
//--- La salida tx ESTA REGISTRADA
module txtest(input wire clk,       //-- Reloj del sistema (12MHz en ICEstick)
              input wire load,      //-- Señal de cargar / desplazamiento
              output reg tx         //-- Salida de datos serie (hacia el PC)
             );

//-- Parametro: velocidad de transmision
//-- Pruebas del caso peor: a 300 baudios
parameter BAUD =  `B300;

//-- Registro de 10 bits para almacenar la trama a enviar:
//-- 1 bit start + 8 bits datos + 1 bit stop
reg [9:0] shifter;

//-- Señal de load registrada
reg load_r; 

//-- Reloj para la transmision
wire clk_baud;

//-- Registrar la entrada load
//-- (para cumplir con las reglas de diseño sincrono)
always @(posedge clk)
  load_r <= load;

//-- Registro de desplazamiento, con carga paralela
//-- Cuando load_r es 0, se carga la trama
//-- Cuando load_r es 1 y el reloj de baudios esta a 1 se desplaza hacia
//-- la derecha, enviando el siguiente bit 
//-- Se introducen '1's por la izquierda
always @(posedge clk)
  //-- Modo carga
  if (load_r == 0)
    shifter <= {"K",2'b01};

  //-- Modo desplazamiento
  else if (load_r == 1 && clk_baud == 1)
    shifter <= {1'b1, shifter[9:1]};

//-- Sacar por tx el bit menos significativo del registros de desplazamiento
//-- Cuando estamos en modo carga (load_r == 0), se saca siempre un 1 para 
//-- que la linea este siempre a un estado de reposo. De esta forma en el 
//-- inicio tx esta en reposo, aunque el valor del registro de desplazamiento
//-- sea desconocido
//-- ES UNA SALIDA REGISTRADA, puesto que tx se conecta a un bus sincrono
//-- y hay que evitar que salgan pulsos espureos (glitches)
always @(posedge clk)
  tx <= (load_r) ? shifter[0] : 1;

//-- Divisor para obtener el reloj de transmision
baudgen #(BAUD)
  BAUD0 (
    .clk(clk),
    .clk_ena(load_r),
    .clk_out(clk_baud)
  );

endmodule
```
#### Simulación
El banco de pruebas **se ha mejorado para poder simular a cualquier velocidad**. Por defecto la simulación está a 115200 baudios para que se ejecute rápidamente. Si simulamos a 300 baudios, los tiempos son mayores por lo que hay que emplear un tiempo de simulación mayor y tardará más tiempo en realizarse.

El banco de pruebas genera **2 pulsos en la señal dtr**. El circuito debe enviar por la línea serie el carácter K. El código es:

```verilog
//-- Fichero txtest.v
`include "baudgen.vh"

module txtest_tb();

//-- Baudios con los que realizar la simulacion
//-- A 300 baudios, la simulacion tarda mas en realizarse porque los
//-- tiempos son mas largos. A 115200 baudios la simulacion es mucho
//-- mas rapida
localparam BAUD = `B115200;

//-- Tics de reloj para envio de datos a esa velocidad
//-- Se multiplica por 2 porque el periodo del reloj es de 2 unidades
localparam BITRATE = (BAUD << 1);

//-- Tics necesarios para enviar una trama serie completa, mas un bit adicional
localparam FRAME = (BITRATE * 11);

//-- Tiempo entre dos bits enviados
localparam FRAME_WAIT = (BITRATE * 4);

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;

//-- Simulacion de la señal dtr
reg dtr = 0;

//-- Instanciar el componente
txtest #(.BAUD(BAUD))
  dut(
    .clk(clk),
    .load(dtr),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("txtest_tb.vcd");
  $dumpvars(0, txtest_tb);

  #1 dtr <= 0;

  //-- Enviar primer caracter
  #FRAME_WAIT dtr <= 1;
  #FRAME dtr <=0;

  //-- Segundo envio
  #FRAME_WAIT dtr <=1;
  #FRAME dtr <=0;

  #FRAME_WAIT $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Los simulamos con el comando:

    $ make sim

El resultado en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest-1-sim.png)

Observamos que efectivamente al llegar un **flanco de subida en dtr**, el circuito envía el carácter. También vemos que la señal de reloj de los bits sólo está funcionando en el momento de transmitir los bits. En cuanto dtr se pone a 0 la señal se para.

#### Síntesis y pruebas

Hacemos la síntesis con el siguiente comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 5 / 96
|PLBs      | 17 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog txtest.bin

Por defecto las pruebas están a la velocidad de **300 baudios**, que es la más lenta. Abrimos el gtkterm. Cada vez que enviamos un pulso en la señal de DTR, se recibe el carácter K:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest-1-gtkterm.png)

Si dejamos apretada la tecla F7, se recibirán continuamente Ks. Se puede observar cómo **de vez en cuando se recibe un carácter erroneo**. Esto dependerá del PC en el que lo probemos. El problema está en que si por alguna razón **la señal DTR se desactiva mientras se está enviando un carácter**, lo que se recibirá es "basura". Esto será algo que tendremos que solucionar en el próximo capítulo mediante la introducción de un **controlador**, implementado con autómatas finitos.

Pasando al modo de vista hexadecimal (View / Hexadecimal) se puede ver exactamente cuál es el **carácter basura recibido**: **0xCB**, cuando se debería recibir el **0x4B**. Analizando los bits se observa que el cambio es en el **bit de mayor peso**, que cambia de 0 a 1, convirtiendo 4B en CB. Justo el bit de mayor peso es el último que se envía

### txtest2.v: Ejemplo de transmisión continua

Este ejemplo es similar al anterior, pero **la salida serie del registro de desplazamiento se realimenta por la entrada**, de manera que cuando la señal dtr esté a 1, se realiza una **transmisión continua del carácter K**. Esto permite comprobar que funciona correctamente a su máxima velocidad

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest2-diagram.png)

El código es igual que el de txtest.v, pero cambiando ligeramente el registro de desplazamiento:

```verilog
always @(posedge clk)
  //-- Modo carga
  if (load_r == 0)
    shifter <= {"K",2'b01};

  //-- Modo desplazamiento
  else if (load_r == 1 && clk_baud == 1)
    shifter <= {shifter[0], shifter[9:1]};
```

#### Simulación

El banco de pruebas es similar al del ejemplo pasado, pero las señales de dtr se mantienen a 1 por más tiempo, para poder ver la transmisión continua

```verilog
`include "baudgen.vh"

module txtest2_tb();

//-- Baudios con los que realizar la simulacion
//-- A 300 baudios, la simulacion tarda mas en realizarse porque los
//-- tiempos son mas largos. A 115200 baudios la simulacion es mucho
//-- mas rapida
localparam BAUD = `B115200;

//-- Tics de reloj para envio de datos a esa velocidad
//-- Se multiplica por 2 porque el periodo del reloj es de 2 unidades
localparam BITRATE = (BAUD << 1);

//-- Tics necesarios para enviar una trama serie completa, mas un bit adicional
localparam FRAME = (BITRATE * 11);

//-- Tiempo entre dos bits enviados
localparam FRAME_WAIT = (BITRATE * 4);

//-- Registro para generar la señal de reloj
reg clk = 0;

//-- Linea de tranmision
wire tx;

//-- Simulacion de la señal dtr
reg dtr = 0;

//-- Instanciar el componente
txtest2 #(.BAUD(BAUD))
  dut(
    .clk(clk),
    .load(dtr),
    .tx(tx)
  );

//-- Generador de reloj. Periodo 2 unidades
always 
  # 1 clk <= ~clk;

//-- Proceso al inicio
initial begin

  //-- Fichero donde almacenar los resultados
  $dumpfile("txtest2_tb.vcd");
  $dumpvars(0, txtest2_tb);

  #1 dtr <= 0;

  //-- Enviar primer caracter
  #FRAME_WAIT dtr <= 1;
  #(FRAME * 3) dtr <=0;

  //-- Segundo envio
  #FRAME_WAIT dtr <=1;
  #(FRAME * 3) dtr <=0;

  #FRAME_WAIT $display("FIN de la simulacion");
  $finish;
end

endmodule
```
Se simula con el comando:

    $ make sim2

El resultado en gtkwave es:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest-2-sim.png)

Se observa cómo envían 3 caracteres "K" en cada ráfaga de activación del dtr

#### Síntesis y pruebas
Hacemos la síntesis con el siguiente comando:

    $ make sint2

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 5 / 96
|PLBs      | 13 / 160
|BRAMs     | 0 / 16

y lo cargamos en la FPGA con:

    $ sudo iceprog txtest2.bin

La prueba se ha hecho a 300 baudios. Al apretar F7 para que cortar la ráfaga de transmisiones se obtienen caracteres basura, porque la transmisión está a medias y se aborta

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T22-syncrules/images/txtest-2-gtkterm.png)

### txtest3.v: Ejemplo de transmisión temporizada
(dibujo)
### Simulación
### Síntesis y pruebas

## Lo que le falta al transmisor
## Ejercicios propuestos
## Conclusiones
TODO

## Créditos
* [Corazón](https://commons.wikimedia.org/wiki/File:Coraz%C3%B3n.svg#/media/File:Coraz%C3%B3n.svg). Licensed under CC BY-SA 3.0 via Wikimedia Commons 
* [Puerta XOR](https://commons.wikimedia.org/wiki/File:Puerta_XOR.svg#/media/File:Puerta_XOR.svg) by Dnu72 - Own work. Licensed under CC BY-SA 3.0 via Wikimedia Commons
* [Unico Anello](https://commons.wikimedia.org/wiki/File:Unico_Anello.png#/media/File:Unico_Anello.png) by Xander - own work, (not derivative from the movies). Licensed under Public Domain via Commons

## Referencias
[1]: "Diseño síncrono de Circuitos Digitales". Miguel Angel Freire Rubio. ETSIST. UPM. Septiembre 2008 ([PDF](http://www.etsist.upm.es/uploaded/464/DS_Sep_2008.pdf))

