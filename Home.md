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
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/FPGA-wars-logo-small.png)  
Únete al grupo [FPGA-WARS: explorando el lado libre](http://fpgawars.github.io/) para conocer las últimas noticias, preguntar, compartir y colaborar en el desarrollo de hardware libre reconfigurable

## Repositorio

[Repositorio con el código verilog](https://github.com/Obijuan/open-fpga-verilog-tutorial)

## Autor
[Juan González Gómez](http://obijuan.github.io/) (Obijuan)

## Licencia
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/attribution-share-alike-creative-commons-license.png)  
Licensed under a  [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/)

## Créditos
* [Proyecto icestorm](http://www.clifford.at/icestorm/), por Clifford Wolf and Mathias Lasser
* [Arachne-pnr](https://github.com/cseed/arachne-pnr), por Cotton Seed 
* [BQ](http://bq.com), por apoyar este proyecto desde el 2015-07-01 hasta el 2016-04-14  

## Agradecimientos
* Este tutorial no habría sido posible sin las increibles herramientas libres del proyecto Icestorm y Arachne-pnr, por **Clifford Wolf**, **Mathias Lasser** y **Cotton Seed**. Representan un avance espectacular para el patrimonio tecnólogico de la humanidad ¡Muchísimas gracias! 

* A [BQ](http://bq.com), por financiar el material usado en el tutorial. ¡Muchas gracias!

* A **Jesús Arroyo**, por las increibles herramientas: icestudio y apio que está desarrollando para hacer más fácil la utilización de FPGAs ¡Muchas Gracias!

* A **David Sánchez Falero**, por enseñarme cómo resaltar código Verilog en los markdown de github :-) ¡Esto ha hecho que el código sea muchísimo más legible! ¡Muchas Gracias! 

* A **Carlos García Saura**, por las instrucciones de instalación de las herramientas icestorm en Fedora. ¡Muchas gracias!

* A **Javibc (J Bc)**, por las instrucciones para relanzar el udev al configurar la icestick ¡Muchas Gracias!

* A **David Cuartielles** por el instalador de las herramientas libres y los ejemplos del makefile para la placa con la FPGA ICE40HX8K. ¡Muchas gracias!

* A **Fernando Barcala** (Barkalez) por su feedback sobre los problemas haciendo el tutorial 1. Con ello se ha mejorado la documentación. ¡Gracias!

* A **Alberto Piganti** (pighixxx) por hacer el **pinout** de la icestick. ¡Gracias!

* A **Cristóbal Bueno** por sus pruebas de las toolchains de icestorm para Linux-32bits. ¡Gracias!

* A **Sebastián Gallardo** Por las prueba de la libusb en Windows 32-bits. ¡Gracias!

* A **Carlos Díaz** por las pruebas de la libusb en Windows 10 64-bits. ¡Gracias!

* A **Pablo Cisneros** (ZioGuillo) por las instrucciones de síntesis/descarga en MAC. ¡Gracias!

* A **Julian Caro** por todas las issues enviadas para mejorar este tutorial. ¡Graciass!

## FAQs
* **¿Qué diferencia hay entre una FPGA y Arduino?**  
 Arduino es una placa que lleva un microcontrolador, que se programa.  Las FPGAs NO SE PROGRAMAN, se configuran para implementar en ellas HARDWARE. Nos sirven para diseñar Circuitos

* **¿Qué hace una FPGA que no pueda hacer Arduino?**  
Muchas aplicaciones se pueden hacer tanto por software (programando) o por hardware. Por ejemplo un contador. La diferencia está en que al hacerlo por hardware conseguimos velocidades muchísimo mayores. Además, el hardware se puede "clonar" directamente y funciona en paralelo. Tener en software varias cosas funcionando en paralelo no es inmediato.

* **¿Por qué Verilog y no VHDL?**  
Verilog es el lenguaje que soporta el [proyecto icestorm](http://www.clifford.at/icestorm/) de momento. Pero como es un proyecto libre, la comunidad lo irá ampliando y el VHDL se soportará tarde o temprano. Al día de hoy (sep-2015), para trabajar con herramientas libres y FPGAs, hay que usar Verilog  

  **Actualización (Agosto-2016)**: El [Proyecto Yodl](https://github.com/forflo/yodl) está implementando VHDL para el sintetizador libre Yosys. Todavía está verde, pero el soporte para VHDL está en camino :-)


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

* **2016/12/28**: ¡Clifford Wolf ya tiene su Icezum Alhambra!. La recibió de manos de Ismael Olea, en el 33C3 en Hamburgo. Todo nuestro reconocimiento al trabajo de Clifford. Y Muchas gracias a Ismael por dársela :-)

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/Clifford-wolf-Ismael-Olea-icezum-alhambra.jpeg)

* **2016/12/21**: 
  * ¡Entregadas las **100 Icezum Alhambras** (v1.1) del primer Lote! :-) #elRepartidorDeLaAlhambra ya puede descansar
  * **Julian Caro** hace [este vídeo tan épico](https://www.youtube.com/watch?v=ImtLt6a4EI4)  ([Más información](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/_HGMov0MS5w/Nem_c3pkEAAJ))

* **2016/12/10**: Actividades en la **Ourense MakersLab 2016**:
  * Charla: "**FPGAs Libres**": ([Más información](https://github.com/Obijuan/myslides/wiki/2016_11_18:-Maker-faire-Bilbao,-FPGAs-Libres))
  * Impartido el **III taller de FPGAs Libres** ([Más información](https://github.com/FPGAwars/workshops/wiki/2016_12_10:-Ourense-MakersLab))

* **2016/11/18**: Charla: "**FPGAs Libres**" en la **Bilbao MakerFaire** ([Más información](https://github.com/Obijuan/myslides/wiki/2016_11_18:-Maker-faire-Bilbao,-FPGAs-Libres))

* **2016/11/09**: Charla: "**FPGAs Libres**" en la **Universidad Rey Juan Carlos**, campus de fuenlabrada ([Más información](https://github.com/Obijuan/myslides/wiki/2016_11_09:-URJC,-Fuenlabrada,-Madrid,-FPGAs-Libreshttps://github.com/Obijuan/myslides/wiki/2016_11_09:-URJC,-Fuenlabrada,-Madrid,-FPGAs-Libres))

* **2016/11/05**:  Actividades en la **OSHWDEM 2016 de A Coruña**, en el Museo Domus
  * Charla: "**FPGAs Libres**" ([Más información](https://github.com/Obijuan/myslides/wiki/2016_11_05:-OSHWdem-2016,-Coru%C3%B1a.-FPGAs-libres))
  * Impartido el **II taller de FPGAs Libres** en España ([Más información](https://github.com/FPGAwars/workshops/wiki/2016_11_05:-OSHWDem16-A-Coru%C3%B1a))

* **2016/10/30**: Lanzada la **Icezum Alhambra Peregrina**, para que todo el mundo pueda frikear con FPGAs libres ([Más información](https://groups.google.com/forum/#!msg/fpga-wars-explorando-el-lado-libre/DsyM8_VJm5U/dJXNHKtEAAAJ)) ([Google Docs](https://docs.google.com/spreadsheets/d/11Riyfs-KfIxmTNFPu6D26Ne3IdXgFJP3_fDqvvpL_3c/edit?usp=sharing))

* **2016/10/28**: Organizado el **I taller sobre FPGAs libres de España**. Impartido en la Escuela Técnica Superior de Ingenieros Industriales, en la UPM ([Más información](https://github.com/FPGAwars/workshops/wiki/2016_10_28:-Reset-ETSII-UPM))

* **2016/10/24**: Charla: "**FPGAs libres**", dada en EL EVENTO 2016, en la Universidad Carlos III de Madrid (UC3M) ([Más información](https://github.com/Obijuan/myslides/wiki/2016_10_24:-El-Evento-2016,-UC3M.-FPGAs-libres))

* **2016/10/20**: Publicada la **Tarjeta Alhambra-led**: Periférico con un led, para la Icezum Alhambra, con **PCB imprimible** ([Más información](https://github.com/FPGAwars/alhambra-led/wiki))

* **2016/10/18**: Podcast 97: [Obijuan, que el software libre te compañe](http://programarfacil.com/podcast/software-libre-hardware-libre-clone-wars/). Entrevista a obijuan en el Podcast de **Programarfacil.com**, por **Luis del Valle**

* **2016/10/02**: Clifford anuncia que empezará a hacer **ingeniería inversa** del bitstream de las **FPGAs de la serie 7 de Xilinx**. Llevará más de 1 año, pero es un hito muy importante para incrementar el parque de FPGAs libres ([Más información](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/RWrDKMzHFdw/3iwhS5XhBgAJ))

* **2016/09/24**: Charla: "FPGAwars: Explorando el lado libre de las FPGAs" dada en la **Madrid Mini maker faire** ([Más información](https://github.com/Obijuan/myslides/wiki/2016_09_24-Madrid-Maker-faire:FPGAwars-explorando-el-lado-libre))

* **2016/09/23**: Primera charla dada en España sobre **FPGAs libres**. Se dió en la ETSIIT de la Universidad de Granada, organizada por la **Oficina de Software Libre de la Universidad de Granada** ([Más información](https://github.com/Obijuan/myslides/wiki/2016_09_23-Granada-Geek-FPGAs-Libres))

* **2016/08/23**: Videoblog 23: **ACC: Apollo CPU Core**. Anuncio del proyecto ACC: la implementación de un core básico de la CPU del apollo en Verilog para FPGAs libres ([VIDEO](https://www.youtube.com/watch?v=2PzKB2mfwoU&index=23&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30)) ([wiki](https://github.com/Obijuan/ACC/wiki))

* **2016/07/29**: Videoblog 22: **Icezum Alhambra V1.0K** Patrimonio de la Humanidad Edition ([VIDEO](https://www.youtube.com/watch?v=d-9dIAo4FVw&index=22&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30))

* **2016/07/28**: [Icezum Alhambra V1.0K](https://github.com/FPGAwars/icezum/wiki) liberada! Se ha creado desde cero utilizando sólo herramientas libres, de nuestro patrimonio tecnológico

* **2016/07/07**: Frontend de **VHDL** para Yosys! Todavía verde, pero la comunidad ya comienza el **soporte de VHDL en las FPGAs libres**: https://github.com/forflo/yodl

* **2016/07/03**: Finalizada la fase de recaudación de dinero para el crowdfunding de la fabricación la **Icezum Alhambra**. Ya hemos recibido los **6500€** de los financiadores. ¡Muchísimas gracias a todos por vuestra ayuda! [Listado de todos los financiadores](https://docs.google.com/spreadsheets/d/1yngU6kfMr5JWJYXrxBMJ7l1Cai2iuucMDoND4NpgU5Y)

* **2016/07/02**: **Jesús arroyo** libera la [versión 0.2.0-Beta1](https://github.com/FPGAwars/icestudio/releases/tag/0.2.0-beta1) del Icestudio. Publica [este vídeo enseñando las nuevas características más relevantes](https://www.youtube.com/watch?v=mAIKb47z2Do): bloques de código y diseño jerárquico.  **Clifford Wolf** se hace eco de la noticia y lo prueba en una icestick :-) [Pone este post en twitter](https://twitter.com/oe1cxw/status/748838491010830336)

* **2016/05/30**: El **esquemático** de la Icezum Alhambra se ha **migrado a [Kicad](http://kicad-pcb.org/)**, una herramienta libre de diseño electrónico ([repo](https://github.com/FPGAwars/icezum/tree/master/src-kicad))

* **2016/05/19**: El **pinout de la Icezum Alhambra**, del maestro Piganti, se ha migrado al **formato SVG**, con Inkscape, para que cualquiera lo pueda modificar o utilizar fácilmente ([repo](https://github.com/FPGAwars/icezum/tree/master/doc/pinout))

* **2016/05/18**: [Objetivo conseguido](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/oLFzYPqCOcQ/luBcnXpBBwAJ). Ya están apalabradas las 100 Icezum Alhambras. ¡Empezamos las gestiones para la fabricación de la primera tirada de 100 unidades!

* **2016-05-04**: Ya tenemos [el pinout de la tarjeta Icezu Alhambra](http://www.pighixxx.com/test/2016/05/icezum-pinout/), hecho por Alberto Piganti ¡Gracias!

* **2016-05-01**: Se pide ayuda en la **lista de FPGA-wars** para la financiación de la **fabricación de 100 tarjetas Icezum Alhambra**. Un mini-crowdfunding ([Mensaje original en el grupo](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/oLFzYPqCOcQ/OsMxYKnuAQAJ))

* **2016-04-29**: [Platformio 2.9.0](https://community.platformio.org/t/big-update-platformio-ide-1-2-0-and-platformio-cli-2-9-0/202) ya soporta las FPGAs libres :-) 

* **2016-04-20**: [Organizacion FPGAwars en github](https://github.com/FPGAwars). Todos los repos relacionados con fpgas libres los hemos trasladado a esa organización. BQ ya no continúa con las FPGAs libres

* **2016-03-24**: Cross-compilación de las herramientas icestorm para el **Ubuntu Phone** ([VIDEO](https://www.youtube.com/watch?v=xZG6to8HW7I&index=19&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30))

* **2016-03-22**: [Documento con las instrucciones de síntesis para MAC](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/tutorial-mac/apple-mac-capitan.pdf), creado por **Pablo Cisneros** (ZioGuillo). ¡Gracias!

* **2016-03-15**: Locomoción de un **robot modular** tipo gusano usando **osciladores hardware en FPGA**, con la tarjeta IceZUM ([VIDEO](https://www.youtube.com/watch?v=6I5Z70eewrg))

* **2016-03-12**: Videoblog sobre la IceZUM Alhambra ([VIDEO](https://www.youtube.com/watch?v=LC8nucTiswM&index=17&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30))

* **2016-03-09**: Publican un [post en Hackaday](http://hackaday.com/2016/03/09/does-the-world-need-an-fpga-arduino/) sobre la IceZUM Alhambra

* **2016-03-08**: ¡Ya tenemos 5 prototipos de la [IceZUM Alhambra](https://github.com/FPGAwars/icezum/wiki) funcionando!  ([Imagen](https://raw.githubusercontent.com/FPGAwars/icezum/master/doc/IceZUM-rev1-1607-img4.jpg)). Hemos hecho algunas pruebas que puedes ver en [estos vídeos](https://github.com/FPGAwars/icezum/wiki#videos)

* **2016-03-05**: **¡Primer bitstream descargado en la tarjeta IceZUM Alhambra!** :-) La placa funciona. Además, las pruebas se han hecho desde Windows, usando las [icestorm MULTIPLATAFORMA](https://github.com/FPGAwars/toolchain-icestorm/releases/tag/0.7) ¡Doble validación! ;-) ([Imagen](https://github.com/FPGAwars/icezum/raw/master/doc/2016-03-04-Mounting-first-prototype/icezum-alhambra-mounting-15.jpg))

* **2016-03-04**: Primera versión de las [icestorm MULTIPLATAFORMA](https://github.com/FPGAwars/toolchain-icestorm/releases/tag/0.7). Ya se tienen cross-compilados las herramientas para Mac y Windows. Pruebas iniciales ok

* **2016-03-03**: Recibimos los primeros PCBs de la [Icezum Alhambra](https://github.com/FPGAwars/icezum/wiki). Están en Pinos del valle, en Granada ([Imagen](https://github.com/FPGAwars/icezum/raw/master/doc/Icezum-Alhambra-pcb-1.jpg))

* **2016-02-29**: La versión de la [tarjeta icezum](https://github.com/FPGAwars/icezum) que está por llegar se ha bautizado como **Icezum Alhambra**. El nombre lo sugirió **Sebastián Gallardo** en [este post de la lista FPGA-wars](https://groups.google.com/d/msg/fpga-wars-explorando-el-lado-libre/f1W0Vtt5NdE/LEDRSXudGwAJ)

* **2016-02-28**: **Cross-compilación** de la herramienta **iceprog para windows**. **Cristóbal Bueno** ha reportado [las primeras pruebas de correcto funcionamiento en Windows 10 de 64 bits](https://github.com/FPGAwars/toolchain-icestorm/wiki#more-tests)

* **2016-02-26**: Clifford lanza [la versión 0.6](https://github.com/cliffordwolf/yosys/releases/tag/yosys-0.6)  de Yosys ([Descarga](http://www.clifford.at/yosys/download.html)). Dentro de poco estará empaquetada para ubuntu y su instalación será trivial

* **2016-02-25**: **Cross-compilación** de las herramientas de icestorm para **Linux-32 bits**. Se generan los paquetes y se pueden instalar directamente desde [Apio](https://github.com/FPGAwars/apio). **Cristóbal Bueno**, de la lista de FPGA-wars, se encarga de hacer las pruebas. ¡Gracias!

* **2016-02-23**: [El icestudio sale publicado en hackaday](http://hackaday.com/2016/02/23/icestudio-an-open-source-graphical-fgpa-tool/) :-)

* **2016-02-19**: [Tarjeta Icezum](https://github.com/FPGAwars/icezum) publicada en github. Diseñada por **Eladio Delgado**. Ayer se mandaron a fabricar los primeros 5 pcbs. Es un tarjeta entrenadora como la icestick pero con forma de BQ ZUM,que es a su vez como un Arduino uno

* **2016-02-08**: **Alberto Piganti** (pighixxx) ha hecho el [dibujo del pinout de la icestick](http://www.pighixxx.com/test/2016/02/icestick-pinout/)

* **2016-02-06**: [Videoblog 15: Programación visual de hardware en FPGAs libres](https://www.youtube.com/watch?v=3nnQ7VTrSLo&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30&index=15)

* **2016-02-04**: Más mejoras en el [icestudio](https://github.com/Jesus89/icestudio), por **Jesús Arroyo**: ([VIDEO](https://www.youtube.com/watch?v=RLr2cC8dgOg)) ([VIDEO](https://www.youtube.com/watch?v=pG1DsF9MIj0))

* **2016-02-02**: Primera prueba del [icestudio](https://github.com/Jesus89/icestudio), diseñado por **Jesús Arroyo**: Bloques lógicos gráficos convertidos a json, de ahí a verilog, y sintetizados y cargados en la FPGA, a través de platformio: ([VIDEO](https://www.youtube.com/watch?v=lIjSNTmHkQU))

* **2016-01-31**: Primera maqueta de **platformio + FPGA**, usado desde Atom ([VIDEO](https://www.youtube.com/watch?v=a_m_Nx66eOs&index=14&list=PLmnz0JqIMEzVo7w2pv6Q04QgRaBTkqR30)). Desarrollar hardware ahora es muchísimo más fácil. Ya no hace falta usar la consola y make ;-)

* **2016-01-28**: Se comienza a dar soporte a las FPGA ice40 de lattice en **Platformio**: [Platformio+FPGA](https://github.com/FPGAwars/Platformio-FPGA/wiki/Platformio-FPGA-wiki-home)

* **2016-01-24**: Simplez documentando y con mejoras: ([VIDEO](https://www.youtube.com/watch?v=_3sWETbaDig))

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

* **2015-07-04**: Empiezo a escribir las [Notas sobre FPGAs libres](http://www.iearobotics.com/wiki/index.php?title=Obijuan:Notas_sobre_FPGAs_libres) en la wiki, para recopilar información. Todavía no puedo evaluar nada porque antes tengo que acabar con [los tutoriales de la temporada 2 de Freecad](http://www.iearobotics.com/wiki/index.php?title=Tutorial_Freecad._Temporada_2). Hasta que no termino una cosa no puedo empezar con la siguiente

* **2015-07-03**: Sale otro [post en hack-a-day](http://hackaday.com/2015/07/03/hackaday-prize-entry-they-make-fpgas-that-small/). Se me dispara el SAV (Síndrome del ansia viva)

* **2015-05-29**: La noticia sale en un [post en hack-a-day](http://hackaday.com/2015/05/29/an-open-source-toolchain-for-ice40-fpgas/). Me entero de su existencia a través de este post y empiezo a evaluar las herramientas (obijuan)

* **2015-05-27**: El [proyecto icestorm](http://www.clifford.at/icestorm/) alcanza el primer gran hito: ya se tiene un flujo de trabajo que funciona completamente usando sólo herramientas libres

# Recopilación de Enlaces
* [Proyecto icestorm](http://www.clifford.at/icestorm/) ([Github](https://github.com/cliffordwolf/icestorm)). Un proyecto para englobarlos a todos
* [Yosys: herramienta de síntesis](http://www.clifford.at/yosys/) ([Github](https://github.com/cliffordwolf/yosys))
* [Arachne: Place & route](https://github.com/cseed/arachne-pnr)
* [Icarus Verilog](http://iverilog.icarus.com/) Simulador de Verilog ([Github](https://github.com/steveicarus/iverilog))
* [Gtkwave](http://gtkwave.sourceforge.net/). Visualizador de señales
* [Placa Icestick](http://www.latticesemi.com/icestick)
* [iCE40LPHXFamilyDataSheet.pdf](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/iCE40LPHXFamilyDataSheet.pdf). Hoja de datos de las FPGAs ICE40
* [icestickusermanual.pdf](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/icestickusermanual.pdf). Hoja de datos de la placa icestick
* [Pinout de la placa Icestick](http://www.pighixxx.com/test/2016/02/icestick-pinout/) por Alberto Piganti
* [Procesador Simplez-F](https://github.com/Obijuan/simplez-fpga/wiki/Procesador-SIMPLEZ-F). Procesador educacional Simplez, del profesor Gregorio Fernández, escrito en Verilog y sintetizado en una Icestick
* [Simplez en Atom](https://github.com/Obijuan/simplez-grammar). Paquete para resaltar la sintáxis de Simplez desde Atom
* [Grupo FPGA-WARS](https://groups.google.com/forum/#!forum/fpga-wars-explorando-el-lado-libre): explorando el lado libre
* [Platformio+FPGA](https://github.com/FPGAwars/Platformio-FPGA/wiki/Platformio-FPGA-wiki-home). Inclusión de las FPGAs ICE40 en el entorno Platformio
* [Apio](https://github.com/FPGAwars/apio). Ecosistema multiplataforma para sintetizar hardware en FPGAs libres
* [Icestudio](https://github.com/FPGAwars/icestudio). Herramienta visual para sintetizar hardware en FPGAs libres
* [Tarjeta IceZUM Alhambra](https://github.com/FPGAwars/icezum/wiki)

# Bibliografía
*  "**Digital Design, An Embedded Systems Approach Using Verilog**". P. Ashenden. 2007.  Es el libro de referencia de diseño digital con lenguajes hdl

[Next](https://github.com/Obijuan/open-fpga-verilog-tutorial/wiki/Cap%C3%ADtulo-0%3A-you-are-leaving-the-privative-sector)