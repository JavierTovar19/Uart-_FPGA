![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-3.png)

[Ejemplos de este capítulo en github](https://github.com/Obijuan/open-fpga-verilog-tutorial/tree/master/tutorial/ICESTICK/T10-shif-register)

## Introducción
Los registros de desplazamiento **almacenan un valor** y nos pemiten **desplazarlo**. Son extremadamente útiles. Se utilizan para convertir la información de paralelo a serie (y vice-versa) para usar en las **comunicaciones síncronas**. Las comunicaciones a través de SPI, I2C, etc, se implementan con estos registros. También nos permiten realizar las operaciones de multiplicar por 2 y dividir entre 2 para número enteros.

En este capítulo los utilizaremos para generar una secuencia de 4 estados en los leds de la placa iCEstick, moviendo las luces en sentido horario

## Descripción del registro
El registro de desplazamiento que usaremos es como el siguiente:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-2.png)

La **salida** del registro es **de N bits** (en nuestro ejemplo usaremos un registro de 4 bits). Tiene una **entrada en paralelo** de N bits, que nos permite cargar en el registro con un valor nuevo. La **señal load_shift** nos permite determinar el **modo de funcionamiento**: cuando está a 0 se realiza la **carga** de un valor nuevo al llegar un flanco de subida de reloj. Cuando está a 1 se realiza un **desplazamiento hacia la izquierda** en el flanco de subida del reloj.

En este desplazamiento **el bit más significativo se pierde**, y el nuevo se lee de la **entrada serin** (serial input). Si tenemos almacenado el valor inicial 1001, la señal load_shift está a 1, serin está a 0 y llega un flanco de subida de reloj, obtendremos el valor:  0010.  En el siguiente flanco (si serin sigue a 0) obtendremos 0100, luego 1000 y luego 0000.

## shift4: Rotación de bits

Como ejemplo usaremos un **registro de desplazamiento de 4 bits** para rotar una secuencia de bits y mostrarlos por los 4 leds de la placa iCEStick. La secuencia obtenida por los leds dependerá del valor inicial cargado en el registro.

El diagrama de bloques del componente **shift4** es:

![Imagen 1](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-1.png)

El componente principal es un **registro de desplazamiento de 4 bits**. Por su entrada de reloj se conecta el reloj de la placa iCEstick a través de un **prescaler**, para disminuir su frecuencia y poder ver la rotación de los bits en los leds.

El bit más significativo del registro (**data[3]**) se conecta directamente a la entrada **serin**, de forma que se consiga la **rotación de bits** (el más significativo pasa a ser el de menor peso)

Por la entrada paralelo introducimos el **valor inicial**, que por defecto será el **0001**. Al rotar aparecerá la secuencia 0010, 0100, 1000 y 0001.

La carga inicial la realizamos usando un **inicializador**, como el mostrado en el capítulo 9. Está conectado a la entrada load_shift, de manera que inicialmente está a 0 y al llegar el primer flanco de reloj pasa a '1', cargándose el valor inicial y pasando a modo desplazamiento. En el resto de flancos se realizará el desplazamiento (y no la carga).  Este es un ejemplo de cómo funciona el inicializador.


## Descripción del hardware

### Registro de desplazamiento

El registro de desplazamiento se describe con muy pocas líneas. Es un proceso que depende del flanco de subida del reloj. 

```verilog
always @(posedge(clk_pres)) begin
  if (load_shift == 0)  //-- Load mode
    data <= INI;
  else
    data <= {data[2:0], serin};
end
```

En el **modo carga** se saca el valor inicial (INI) por la salida. En el de desplazamiento se sacan los tres bits menos significativos y se añade serin como menos significativo. Esto se hace con el **operador de concatenación {}**: A los tres cables definidos por data[2:0] se le añade un cuarto cable: serin

### Componente shift4

El componente final shift4 tiene **2 parámetros**: el **número de bits del prescaler** (NP), que determina la velocidad de rotación de los bits y el **valor inicial** (INI) a cargar, que determina la secuencia. Por defecto, este valor inicial es 0001.

El código para describir el componente shift4 completo es:

```verilog
//-- shift4.v
module shift4(input wire clk, output reg [3:0] data);
    
//-- Parametros del secuenciador
parameter NP = 21;  //-- Bits del prescaler
parameter INI = 1;  //-- Valor inicial a cargar en el registro
    
//-- Reloj de salida del prescaler
wire clk_pres;
    
//-- Shift / load. Señal que indica si el registro
//-- se carga o desplaza
//-- shift = 0: carga
//-- shift = 1: desplaza
reg load_shift = 0;
    
//-- Entrada serie del registro
wire serin;
    
//-- Instanciar el prescaler de N bits
prescaler #(.N(NP))
  pres1 (
    .clk_in(clk),
    .clk_out(clk_pres)
  );
    
//-- Inicializador
always @(posedge(clk_pres)) begin
    load_shift <= 1;
end
    
//-- Registro de desplazamiento
always @(posedge(clk_pres)) begin
  if (load_shift == 0)  //-- Load mode
    data <= INI;
  else
    data <= {data[2:0], serin};
end
    
//-- Salida de mayor peso se re-introduce por la entrada serie
assign serin = data[3];
    
endmodule
```

El registro de desplazamiento se ha implementado directamente como un proceso en el propio componente, en vez de hacer como un componente separado (diseño jerárquico).

## Síntesis en la FPGA

Para sintetizarlo en la fpga conectaremos las salidas data a los leds, y la entrada de reloj a la de la placa iCEstick

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-3.png)

Sintetizamos con el comando:

    $ make sint

Los recursos empleados son:

| Recurso  | ocupación
|----------|-----------
|PIOs      | 4 / 96
|PLBs      | 8 / 160
|BRAMs     | 0 / 16

Para cargar en la FPGA ejecutamos:

    $ sudo iceprog shift4.bin

En este **vídeo de Youtube** se puede ver la salida de los leds:

[![Click to see the youtube video](http://img.youtube.com/vi/JgwFiXQobi4/0.jpg)](https://www.youtube.com/watch?v=JgwFiXQobi4)

## Simulación

El banco de pruebas es uno básico, que instancia el componente shift4, con 1 bit para el prescaler (para que la simulación tarde menos) y un valor inicial del registro de 0001. Tiene un proceso para la señal de reloj y uno para la inicialización de la simulación

![Imagen 3](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/shift4-4.png)

La simulación se realiza con:

    $ make sim

El resultado en gtkwave es:

![Imagen 4](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T10-shif-register/images/T10-shift4-sim-1.png)

Vemos cómo inicialmente el registro tiene un valor indefinido (está en rojo, aunque en la síntesis tendrá un 0) y al llegar el **primer flanco de subida** se inicializa con el valor **0001**  (Gracias al circuito inicializador). En los siguientes flancos vemos cómo efectivamente el bit se desplaza hacia la izquierda: 0010, 0100, 1000  y por último vuelve al valor inicial: 0001, repitiéndose la secuencia hasta el final de la simulación.

## Ejercicios propuestos
* Ejercicio 1: Cambiar el valor del prescaler para que la rotación sea más rápid y el valor inicial del registro, para que salga otra secuencia
* Ejercicio 2: Utilizar un registro de desplazamiento de 8 bits, conectando los 4 menos significativos a los leds. De esta forma se pueden hacer secuencias en las que haya momentos en los que los leds están apagados.

## Conclusiones
TODO



