![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-2.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T23-fsmtx)

# Introducción
Los **controladores** son la parte de los circuitos digitales que se **encargan de generar todas las señales** (microórdenes) **que gobiernan  el resto de componentes**. Se implementan mediante **autómatas finitos** (también denominadas **máquinas de estados**). En este capítulo diseñaremos un controlador para nuestro **transmisor serie**

# Arquitectura genérica

La **estructura** de los circuitos digitales la podemos descomponer en dos partes:  la **ruta de datos** y el **controlador**

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-3.png)

* **Ruta de datos**: Es la parte que **procesa los datos**. Comprende todas las unidades funciones junto con sus conexiones para realizar ese procesamiento: registros de desplazamiento, unidades aritmético-lógicas, contadores, etc.

* **Controlador**: Es la **parte de gobierno**, que genera las señales necesarias, denominadas **microórdenes**, para que la ruta de datos se comporte adecuadamente. Envía microórdenes para poner en marcha un contador, cargar un dato en un registro, realizar tal operación oritmética, etc. Todo ello secuenciado en el tiempo.

El controlador **toma decisiones** y genera unas microórdenes u otras en función de los resultados parciales de la ruta de datos. Por ejemplo, si un contador ha alcanzado un determinado valor, entonces se deberá general tal señal para hacer tal cosa.  Todo eso lo hace el controlador.

Los controladores se implementan mediante [autómatas finitos](https://es.wikipedia.org/wiki/Aut%C3%B3mata_finito) (también denominados máquinas de estados). Es necesario primero definir **los estados** del circuito y **las transiciones** entre estos estados según las condiciones del circuito. Luego, en función de estos estados se generan las **microórdenes**. 

# Máquinas de estado en verilog

Partimos de este **diagrama de 4 estados**, para explicar cómo describirlo en Verilog

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-4.png)

**Inicialmente** el circuito se encuentra en el **estado 0** y el controlador generará las microórdenes necesarias (no mostradas en el dibujo). Mientras que la **señal a** esté a 0, se mantendrá siempre en ese estado. En cuanto se ponga a 1 se pasará al **estado 1**, donde se generarán otras microórdenes. En este estado, según el valor de la **señal b**, bien se volverá al estado inicial o se avanzará al **estado 2**. El estado 2 **no tiene condiciones**, por lo que **en el siguiente ciclo de reloj** se pasa al **estado 4**. Cuando la **señal c** valga 0, se vuelve al estado inicial.

Primero definimos las** etiquetas para los estados** mediante **parámetros locales**, y un **registro** para **almacenar el estado**. Como tenemos 4 estados, este registro será de **2 bits**:

```verilog
localparam ESTADO0 = 0;
localparam ESTADO1 = 1;
localparam ESTADO2 = 2;
localparam ESTADO3 = 3;

reg [1:0] state;
```
Las **transiciones** entre estados las modelamos mediante un **proceso que depende del reloj** y que al recibir el **reset** se inicializa en el **estado inicial**. Usando un _**case**_ definimos las transiciones para cada estado:

```verilog
always @(posedge clk)
  if (rst)
    //-- Establecer el estado inicial al arrancar
    state <= ESTADO0;
  else 
    case (state)
      ESTADO0:
        //-- Cambio de estado condicional
        if (a) state <= ESTADO1;
        else state <= ESTADO0;

      ESTADO1:
        //-- Cabmio de estado condidiconal
        if (b) state <= ESTADO2;
        else state <= ESTADO1;

      ESTADO2:
        //-- Cambiar de estado en el próximo ciclo
        state <= ESTADO3;

      ESTADO3:
        //-- Cabmio de estado condidiconal
        if (c) state <= ESTADO0;
        else state <= ESTADO3;

      default:
        //-- Siempre hay que ponerlo, para evitar
        //-- latches en la síntesis
        state <= ESTADO0;

    endcase
```
En todos los estados salvo el ESTADO2 las **transiciones son condicionales**: según el estado de las señales a,b y c se salta a un estado o a otro.  **El ESTADO2 sólo dura 1 ciclo**. En cuanto se entra en él, se salta al ESTADO3 **incondicionalmente**, en el siguiente ciclo de reloj.

Ahora falta la parte de **generación de las microórdenes**. Supondremos que este circuito tiene 2 microórdenes: **m1** y **m2**. Dependiendo del estado en el que se encuentre, se asignarán diferentes valores. Lo implementamos como un **proceso combinacional** con un **case**:

```verilog
always @*
  case (state)
    ESTADO0: begin
      m1 <= 0;
      m2 <= 0;
    end

    ESTADO1: begin
      m1 <= 1;
      m2 <= 0;
    end

    ESTADO2: begin
      m1 <= 0;
      m2 <= 0;
    end

    ESTADO3: begin
      m1 <= 1;
      m2 <= 1;
    end

    default: begin
      m1 <= 0;
      m2 <= 0;
    end
  endcase
```
Usar el **case** es la manera directa, pero hay que **definir el valor de todas las microórdenes para cada estado**. Mediante **assignaciones directas** se puede reducir el código sustancialmente. En el ejemplo anterior, vemos que la microórden **m2** sólo se activa en el ESTADO3 y en el resto está desactivada. Para **m1** vemos que se activa en los estados ESTADO1 y ESTADO3. Esto lo podemos describir así:

```verilog
assign m1 = (state == ESTADO1 || state == ESTDO3) ? 1 : 0;
assign m2 = (state == ESTADO3) ? 1 : 0;
```
El código queda mucho más compacto.

# Transmisor serie mejorado

Aplicaremos esta arquitectura al transmisor serie, dividiéndolo en su ruta de datos y el controlador 

## fsmtx.v: Transmisión continua
Haremos un transmisor que envíe el **carácter A** cuando se ponga a 1 la **señal de entrada _start_**, que la controlaremos a través de la **señal dtr** (como en los ejemplos del capítulo anterior). Sin embargo, esta señal no controla directamente la transmisión, sino que **sólo la inicia**. Será el controlador el que controle esta transmisión. De esta manera, **nunca se quedará un carácter a medio transmitir** como pasaba en el capítulo anterior.

 El esquema es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-1.png)

## Ruta de datos
En la **ruta de datos** se encuentra el **registro de desplazamiento** que hace la serialización de los datos, el **generador de baudios**, el inicializador, un **contador de 4 bits** para llevar la cuenta de bits enviados y los **flip-flops** para registrar señales y cumplir con las normas de diseño síncrono

El contador lleva la cuenta de **cuántos bits se han enviado** y servirá para que el controlador sepa cuándo ha finalizado la transmisión. La señal de "load" sirve para **ponerlo a cero**, y es la misma que se usa para realizar la carga del registro de desplazamiento

## Controlador

El **diagrama de estados** del controlador es el siguiente:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T23-fsmtx/images/fsmtx-2.png)

El transmisor puede estar en 3 estados:

* **IDLE**: Estado de reposo. No se está transmitiendo nada. Se espera a que se active la señal de start para comenzar con la transmisión del carácter "A"
* **START**: Comienzo de transmisión. Se carga el dato en el registro de desplazamiento. Se arranca el reloj para la temporización de los bits a la velocidad configurada. Se pone el contador de bits a cero. Es un estado transitorio que dura 1 ciclo de reloj.
* **TRANS**: Estado de transmisión. Permanece en este estado durante la trasmisión del carácter. En cuanto el contador de bits alcanza el valor 11 (10 bits ya transmitidos) se pasa al estado inicial de reposo

Para controlar la ruta de datos se necesitan **2 microórdenes**, _load_ y _baudgen_. La se usa para cargar el dato en el registro de desplazamiento y poner a cero el contador de bits. La segunda es la habilitación del temporizador de bits (generador de baudios) 


