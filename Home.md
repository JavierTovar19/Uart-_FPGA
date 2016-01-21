![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/tutorial-fpga-logo.png)

# Diseño Digital para FPGAs, con herramientas libres

## Descripción

Tutorial para aprender a diseñar **sistemas digitales sintetizables** en FPGAs usando **SOLO** herramientas libres. Descripción del hardware en lenguaje **Verilog**
* **Placa de desarrollo**: [iCEStick](http://www.latticesemi.com/icestick). Con la FPGA iCE40HX-1k de Lattice
* **Simulador de Verilog**: [ícarus Verilog](http://iverilog.icarus.com/) 
* **Visualizador de señales**: [Gtkwave](http://gtkwave.sourceforge.net/)
* **Sintetizador**: [Yosys](http://www.clifford.at/yosys/) ([Repo en github](https://github.com/cliffordwolf/yosys))
* **Place & route**: [Arachne-pnr](https://github.com/cseed/arachne-pnr) (en github)
* **Utilidades y descarga en FPGA**: [Proyecto icestorm](http://www.clifford.at/icestorm/)

Es un **tutorial incremental**, en el que los conceptos se van introduciendo progresivamente, a la vez que se van probando en un hardware real. Los circuitos presentados inicialmente no están diseñados siguiendo las reglas de diseño habituales, sino que se prima la simplicidad. Luego poco a poco, según se avanza en el tutorial, se van refinando y mejorando.

El viaje comienza con el circuito más sencillo posible: un cable. Y termina con el diseño de **un microprocesador muy simple**: MICROBIO

## Comunidad: FPGA-WARS
Únete al grupo [FPGA-WARS: explorando el lado libre](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre) para conocer las últimas noticias, preguntar, compartir y colaborar en el desarrollo de hardware libre reconfigurable

## Repositorio

[Repositorio con el código verilog](https://github.com/Obijuan/open-fpga-verilog-tutorial)

## Autor
[Juan González Gómez](http://obijuan.github.io/) (Obijuan)

## Licencia
<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/bq-logo-cc-sa-small-150px.png">

Licensed under a  [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)

## Créditos
* [Proyecto icestorm](http://www.clifford.at/icestorm/), por Clifford Wolf and Mathias Lasser
* [Arachne-pnr](https://github.com/cseed/arachne-pnr), por Cotton Seed 

## Agradecimientos
* Este tutorial no habría sido posible sin las increibles herramientas libres del proyecto Icestorm y Arachne-pnr, por **Clifford Wolf**, **Mathias Lasser** y **Cotton Seed**. Representan un avance espectacular para el patrimonio tecnólogico de la humanidad ¡Muchísimas gracias! 

* A [BQ](http://bq.com), por financiar el material usado en el tutorial. Soy un verdadero privilegiado por trabajar en una empresa donde fomentan el espíritu maker y apoyan las iniciativas de exploración y experimentación con tecnologías Libres ¡Muchas gracias!

* A **Jesús Arroyo**, por hacer la compra de las placas iCEStick y empezar con el empaquetado para Debian/Ubuntu de las herramientas Arachne-pnr y las del proyecto icestorm. ¡Muchas Gracias!

* A **David Sánchez Falero**, por enseñarme cómo resaltar código Verilog en los markdown de github :-) ¡Esto ha hecho que el código sea muchísimo más legible! ¡Muchas Gracias! 

* A **Carlos García Saura***, por las instrucciones de instalación de las herramientas icestorm en Fedora. ¡Muchas gracias!

* A **Javibc (J Bc)**, por las instrucciones para relanzar el udev al configurar la icestick ¡Muchas Gracias!

* A **David Cuartielles** por el instalador de las herramientas libres y los ejemplos del makefile para la placa con la FPGA ICE40HX8K. ¡Muchas gracias!

* A **Fernando Barcala** (Barkalez) por su feedback sobre los problemas haciendo el tutorial 1. Con ello se ha mejorado la documentación. ¡Gracias!

## FAQs
* **¿Qué diferencia hay entre una FPGA y Arduino?**  
 Arduino es una placa que lleva un microcontrolador, que se programa.  Las FPGAs NO SE PROGRAMAN, se configuran para implementar en ellas HARDWARE. Nos sirven para diseñar Circuitos

* **¿Qué hace una FPGA que no pueda hacer Arduino?**  
Muchas aplicaciones se pueden hacer tanto por software (programando) o por hardware. Por ejemplo un contador. La diferencia está en que al hacerlo por hardware conseguimos velocidades muchísimo mayores. Además, el hardware se puede "clonar" directamente y funciona en paralelo. Tener en software varias cosas funcionando en paralelo no es inmediato.

* **¿Por qué Verilog y no VHDL?**  
Verilog es el lenguaje que soporta el [proyecto icestorm](http://www.clifford.at/icestorm/) de momento. Pero como es un proyecto libre, la comunidad lo irá ampliando y el VHDL se soportará tarde o temprano. Al día de hoy (sep-2015), para trabajar con herramientas libres y FPGAs, hay que usar Verilog

* **¿Dónde puedo comprar la tarjeta IceStick**
-[Icestick en Mouser España ](http://www.mouser.es/new/Lattice-Semiconductor/lattice-icestick-kit/)
-[Icestick en LatticeUSA](http://www.latticesemi.com/icestick)

* **He hecho el tutorial Hola mundo y la diferencia es que al sintetizar el archivo no me genera un .bin si no un .blif**  
El fichero .blif es lo que se genera en la síntesis, después de ejecutar el programa yosys. Sin embargo, para obtener el .bin todavía tienes que ejecutar más comandos. Todos los comandos a ejecutar para obtener el .bin a partir del .v en el primero tutorial son:  
    ```
$ yosys -p "synth_ice40 -blif setbit.blif" setbit.v
$ arachne-pnr -d 1k -p setbit.pcf setbit.blif -o setbit.txt
$ icepack setbit.txt setbit.bin
    ```  
Para hacerlo más fácil, lo mejor es ejecutar el comando **make sint**. Para ello tienes que clonar el repositorio del tutorial de github, entrar en el directorio de trabajo y ejecutar make sint:  
    
    ```
$ git clone https://github.com/Obijuan/open-fpga-verilog-tutorial.git
$ cd open-fpga-verilog-tutorial/tutorial/ICESTICK/T01-setbit/
$ make sim
    ```

# NOTICIAS

* **2016-01-12**: Jesús Arroyo replica los [ejemplos de clifford del 32CCC](http://www.clifford.at/papers/2015/icestorm-flow/) en nuestra icoboard, y crea ejemplos nuevos que sube a [este repositorio en github](https://github.com/Jesus89/picorv32-c-examples)

* **2016-01-11**: Recibimos la [tarjeta icoboard](http://icoboard.org/) que nos donan en el 32CCC, al que asistió Víctor Díaz. ¡Muchas gracias! :-)

* **2016-01-02**: David Cuartielles [crea un instalador](https://github.com/dcuartielles/open-fpga-install) para facilitar la instalación de todas las herramientas libres para trabajar con FPGA (en sistemas Ubuntu / Debian)

* **2015-12-31**: [Pull request de David Cuartielles](https://github.com/Obijuan/open-fpga-verilog-tutorial/pull/3) al repo del tutorial. Añadido un nuevo makefile para soportar [la tarjeta ICE40HX8K board de lattice](http://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/iCE40HX8KBreakoutBoard.aspx). 

* **2015-12-30**: Recibimos dos [tarjetas ICE40HX8K board de lattice](http://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/iCE40HX8KBreakoutBoard.aspx), compradas en [MOUSER](http://www.mouser.es/ProductDetail/Lattice/ICE40HX8K-B-EVN/?qs=%2fha2pyFadugG1hxFZcqbAJG8jgdGBbaecmBJrQx0SABRTeEI3HN%252bIw%3d%3d). Tienen la FPGA HX8K de lattice, con 8 veces el tamaño de la HX1K de la icestick. En esta ya se puede sintetizar un procesador risc-v de 32 bits (no probado todavía)

* **2015-12-27**: Clifford da la charla ["A Free and Open Source Verilog-to-Bitstream Flow for iCE40 FPGAs"](https://media.ccc.de/v/32c3-7139-a_free_and_open_source_verilog-to-bitstream_flow_for_ice40_fpgas#video) en la edición 32 del Chaos Communication Congress, en Hamburgo. [Todo el material está publicado en su web](http://www.clifford.at/papers/2015/icestorm-flow/).  En la demo usan una FPGA HX8K en la [placa icoboard](http://icoboard.org/) con un procesador risc-v de 32 bits sintetizado usando herramientas libres. Víctor Díaz que está allí conoce a Clifford, y éste le entrega una icoboard para dársela a obijuan :-)

* **2015-12-08**: Encontrada **[la solución a un Bug en iceprog](https://github.com/cliffordwolf/icestorm/issues/16)**: Ahora los tiempos de descarga en la icestick son siempre menores de 3 segundos!!!  Clifford lo integra el repo oficial casi al instante. El ciclo de desarrollo es ahora mucho más rápido

* **2015-12-04**: Se crea [el grupo de FPGA-WARS](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre). Se [anuncia oficialmente en clonewars](https://groups.google.com/d/msg/asrob-uc3m-impresoras-3d/Lv6Tjb_3js8/_C4ZTISADAAJ)

* **2015-12-01**: Cena de David Cuartielles de Arduino.cc y Rodrigo del Prado y Juan González de BQ, en Madrid. Le comentamos a David el nuevo ecosistema de herramientas libres para FPGA y la posibilidad de colaborar juntos

* **2015-11-26**: Primera versión del procesador educacional **SIMPLEZ** sintetizado en la icestick, ejecutando un programa que muestra una secuencia por los leds: [Repo](https://github.com/Obijuan/simplez-fpga)

# Historia
* **2015-11-20**: Terminado el último capítulo del tutorial (30)

* **2015-10-21**: **Marcha imperial** sonando en la icestick. Le envío [este vídeo a Clifford](https://www.youtube.com/watch?v=IDU8kHAaLTw). También le comento que estoy trabajando en estos tutoriales.  Le gusta la idea y **publica el enlace de los tutoriales en la web de icestorm** :-)

* **2015-08-12**: Primer contacto con Clifford wolf (el autor del proyecto icestorm). Le envío un correo dándole las gracias por el proyecto. Me responde :-)

* **2015-08-10**: Devuelta en Madrid, aunque todavía de vacaciones. Me paso por BQ a recoger una icestick. Logro un hito histórico: Descargo mi primer diseño verilog en la fpga, usando herramientas libres!!! Un programa "hola mundo" que enciende un led!!!!! :-)  Comienzo esta wiki para hacer los tutoriales a medida que voy aprendiendo

* **2015-08-05**: Se reciben en BQ las dos placas icestick (Y yo de vacaciones con el SAV a tope!!!!)

* **2015-08-03**: **Jesús Arroyo** compra 2 tarjetas icesticks directamente desde Lattice, para el departamento de innovación y robótica de BQ

* **2015-07-31**: Primeras pruebas. Instalado todo el tool-chain de la web de icestorm en un ubuntu 15.04. ¡Funciona!. Todos los ejemplos de prueba los subo a [este repo](https://github.com/Obijuan/mytests/tree/master/verilog). El SAV se me dispara. Me voy de vacaciones (sin internet durante 10 días). Me llevo PDFs con manuales de verilog para leer y aprender

* **2015-07-30**: Termino [la segunda temporada de los tutoriales de Freecad](http://www.iearobotics.com/wiki/index.php?title=Tutorial_Freecad._Temporada_2). ¡Ya me puedo poner con las FPGAs!

* **2015-07-04**: Empieza a escribir las [Notas sobre FPGAs libres](http://www.iearobotics.com/wiki/index.php?title=Obijuan:Notas_sobre_FPGAs_libres) en la wiki, para recopilar información. Todavía no puedo evaluar nada porque antes tengo que acabar con [los tutoriales de la temporada 2 de Freecad](http://www.iearobotics.com/wiki/index.php?title=Tutorial_Freecad._Temporada_2). Hasta que no termino una cosa no puedo empezar con la siguiente

* **2015-07-03**: Sale otro [post en hack-a-day](http://hackaday.com/2015/07/03/hackaday-prize-entry-they-make-fpgas-that-small/). Me entero de su existencia a través de Hackaday y empiezo a evaluar las herramientas (obijuan). Se me dispara el SAV

* **2015-05-29**: La noticia sale en un [post en hack-a-day](http://hackaday.com/2015/05/29/an-open-source-toolchain-for-ice40-fpgas/). Me entero de su existencia a través de este post y empiezo a evaluar las herramientas (obijuan)

* **2015-05-27**: El [proyecto icestorm](http://www.clifford.at/icestorm/) alcanza el primer gran hito: ya se tiene un flujo de trabajo que funciona completamente usando sólo herramientas libres

[next](Cap%C3%ADtulo-1%3A-%C2%A1Hola-mundo%21-Setbit)