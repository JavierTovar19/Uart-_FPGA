![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/buffer-1.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/T28-ram)

# Introducción
Las **memorias RAM** nos permiten almacenar datos y recuperarlos durante el funcionamiento del circuito. Son memorias donde podemos leer y escribir

Crearemos una **memoria RAM genérica**, síncrona, con una entrada de datos para escritura y una salida para lectura. Como ejemplo diseñaremos un **buffer de almacenamiento** de 16 bytes que cargue a través de datos provenientes del puerto serie, y una vez llenado, se vuelca su contenido también por el puerto serie.

# Memoria RAM genérica

La memoria RAM genérica la denominaremos genram

## Puertos y parámetros

Los puertos y parámetros se muestran esta figura:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/genram-1.png)

Los **parámetros** son los mismos que en la memoria ROM:
* **DW** (Data width): Anchura de los datos (en bits)
* **AW** (Address width): Anchura de las direcciones (en bits)
* **ROMFILE**: Fichero con el contenido inicial precargado de la ram

Los **puertos** son:
* **data_in**: Entrada de datos, para escritura
* **data_out**: Salida de datos, en la lectura
* **addr**: Dirección de acceso, tanto para escritura como para lectura
* **rw**: Modo de acceso: lectura (rw = 1) o escritura (rw = 0)

## Cronograma

En este cronograma se muestran **dos ciclos de lectura** y **uno de escritura** intercalado:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T28-ram/images/genram-2.png)

**Ciclo de lectura**:
* Se coloca la dirección de lectura en **addr**
* Se pone la **señal rw a 1** para indicar que se quiere leer
* En el siguiente **flanco de subida** se **devolverá el dato** por el puerto **data_out**

**Ciclo de escritura**:
* Se coloca la **dirección** de escritura en** addr**
* Se coloca el **dato a escribir** en el puerto **data_in**
* Se pone la **señal rw a 0** para indicar **escritura**
* En el siguiente **flanco de subida** del reloj se escribirá el **dato en la memoria**

## Descripción en verilog

El código es similar al de la memoria rom pero añadiendo un proceso nuevo que se encarga de **la escritura en cada flanco de subida del reloj**, usando la señal rw como habilitación

```verilog
module genram #(             //-- Parametros
         parameter AW = 5,   //-- Bits de las direcciones (Adress width)
         parameter DW = 4)   //-- Bits de los datos (Data witdh)

       (        //-- Puertos
         input clk,                      //-- Señal de reloj global
         input wire [AW-1: 0] addr,      //-- Direcciones
         input wire rw,                  //-- Modo lectura (1) o escritura (0)
         input wire [DW-1: 0] data_in,   //-- Dato de entrada
         output reg [DW-1: 0] data_out); //-- Dato a escribir

//-- Parametro: Nombre del fichero con el contenido de la RAM
parameter ROMFILE = "bufferini.list";

//-- Calcular el numero de posiciones totales de memoria
localparam NPOS = 2 ** AW;

  //-- Memoria
  reg [DW-1: 0] ram [0: NPOS-1];

  //-- Lectura de la memoria
  always @(posedge clk) begin
    if (rw == 1)
    data_out <= ram[addr];
  end

  //-- Escritura en la memoria
  always @(posedge clk) begin
    if (rw == 0)
     ram[addr] <= data_in;
  end

//-- Cargar en la memoria el fichero ROMFILE
//-- Los valores deben estan dados en hexadecimal
initial begin
  $readmemh(ROMFILE, ram);
end

endmodule
```

# Ejemplo: Buffer

Para probar la memoria ram, implementaremos un **buffer de 16 bytes**. Se comienza volcando el buffer a través del puerto serie (tiene unos datos iniciales que establecemos en el fichero bufferini.list). Los siguientes caracteres recibidos por el puerto serie, se van almacenando en la memoria, comenzando por la dirección 0. Una vez llenada la memoria, se procede a su volcado

## Diagrama de bloques

La arquitectura del circuito es la clásica: una ruta de datos y un controlador:

(dibujo)

## Descripción en verilog

## Simulación

## Síntesis y pruebas

# Ejercicios propuestos

# Conclusiones
TODO
