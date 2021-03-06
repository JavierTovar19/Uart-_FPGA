<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/checkpoint-charlie.png" 
width="400" align="center">

_You are leaving the privative sector_ ... ¡y estás entrando en el sector LIBRE! ¡Bienvenido! De aquí en adelante **sólo usaremos herramientas del patrimonio tecnológico de la humanidad**

# Introducción

Las [FPGAs](https://es.wikipedia.org/wiki/Field_Programmable_Gate_Array) son unos chips "en blanco" que nos permiten configurarlos para crear dentro de ellos nuestros propios circuitos digitales. ¡Si! ¡Con las FPGAs estamos creando hardware! 

Todos los circuitos digitales se descomponen en sus elementos básicos: **puertas lógicas** para hacer operaciones booleanas con los bits y **biestables** para almacenarlos. Como primera aproximación, podemos pensar en una FPGA como un chip que tiene en su interior arrays de estos elementos, sin conectar. Al **configurarla**, establecemos estas uniones y obtenemos nuestro circuito.

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/fpga-config1.png)

Esta configuración se consigue **descargando en la FPGA** un fichero binario, denominado **bitstream**, que contiene toda la información necesaria para establecer las conexiones entre los elementos internos de la FPGA

## Generación del bitstream

La magia de las FPGAs está en las **herramientas software** que permiten generar el bitstream a partir de la **descripción del circuito** en un lenguaje HDL

Los circuitos se diseñan utilizando un **lenguaje de descripción hardware** (HDL), como Verilog o VHDL. Son los ficheros fuentes. La generación del bitstream se hace en dos fases, a partir de las fuentes:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/fpga-bitstream1.png)

**1 Síntesis**: La herramienta de síntesis infiere los elementos hardware básicos a partir de su descripción, y obtiene un fichero **netlist** que describe las uniones entre ellos. Esta fase no depende de la FPGA a usar

**2 Emplazado y rutado**. Los componentes del netlist se hacen corresponder con los elementos físicos de la FPGA, se determina su colocación y se realiza el rutado. Toda la información de configuración de la FPGA se condensa en el bitstream. Esta fase sí depende del modelo concreto de FPGA que se disponga

**NOTA SOBRE TERMINOLOGÍA**: Aunque técnicamente la fase de síntesis en sólo una parte de la generación del bitstream, coloquialmente **cuando se habla de síntesis nos solemos referir al proceso completo**. Así, si decimos que "hemos sintetizado este circuito en la FPGA", nos estamos refiriendo a que se han realizado todas las fases: síntesis, emplazado y rutado, generación del bitstream y carga en la FPGA

## Cautivos del fabricante

Las FPGAs se conocen desde hace 30 años. Son unas herramientas tremendamente útiles con muchísimo potencial. ¡Te permiten diseñar tu propio chip! Sin embargo, los fabricantes de FPGAs nunca han liberado ni el software ni las especificaciones de los formatos de sus bitstreams. 

Esto ha hecho que nadie pueda crear software para trabajar con las FPGAs, sino que sólo se puede utilizar el del fabricante. Y sólo se puede utilizar en los ordenadores que el fabricante te diga. Sólo puedes diseñar lo que el fabricante haya pensado que se puede diseñar con sus herramientas.  Si se te ocurre algo nuevo, no soportado por su software, no podrás hacer nada.  Todo esto es bastante frustrante. Y ha hecho que al final mucha gente deje de utilizar las FPGAs.

# Herramientas libres para trabajar con FPGAs
En **mayo de 2015** ocurrió un **hito histórico**: se tuvieron por primera vez todas las herramientas necesarias para **generar el bitstream** a partir de **código en Verilog** usando **sólo software libre**, gracias al [proyecto icestorm](http://www.clifford.at/icestorm/), liderado por **Clifford Wolf**.  A partir de ese momento,ya tenemos herramientas que pertenecen al **patrimonio tecnológico de la humanidad** para trabajar con FPGAs, y poder desarrollar hardware usando sólo herramientas de este patrimonio

## Ventajas del uso de las herramientas libres
* **Autonomía**: Los desarrolladores de hardware pueden desarrollar sus sistemas con independencia del fabricante. Ya no se depende de los caprichos de cada fabricante, o de sus gustos. Con las herramientas libres nos independizamos. Los diseñadores decidimos qué sistema operativo utilizar. O qué entorno usar. Ya no estamos obligados a hacer lo que nos diga el fabricante.

* **Acceso al conocimiento**: Estas herramientas las podemos usar normalmente, igual que las privativas. Sin embargo, si tenemos curiosidad, tenemos acceso al conocimiento de cómo están programadas, qué algoritmos se usan, cómo se implementa la síntesis... Esto fomenta el espíritu científico de comprender cómo funcionan las cosas... para luego mejorarlas. Ahora es posible que investigadores de cualquier parte del mundo analicen los algoritmos, los comprendan y los mejoren. Antes sólo los fabricantes lo podían hacer

* **Nuevas aplicaciones**: Se abre el camino a probar nuevos usos de la FPGA no previstos por los fabricantes. Desde el comienzo de las FPGAs han surgido las ideas de usar hardware bajo demanda, de codiseño hw/hw, sistemas operativos que usen tareas hw, etc. Aunque se han escrito muchísimas tesis sobre ello, las implementaciones reales eran muy específicas para un fabricante concreto. Y poco reproducibles por la comunidad. Ahora ya es viable hacer implementaciones que corran por ejemplo en una raspberry pi, y que se sintetice el hardware bajo demanda. Con las herramientas privativas era imposible, porque no estaba previsto por los fabricantes

* **Participación de la comunidad**:  Ahora TODOS podemos participar en la evolución de las FPGAs, no sólo limitándonos a usarlas, sino haciendo crecer y mejorando las propias herramientas.

* **Repositorios de Hardware libre reconfigurable**: Llegó el momento de "reinventar la rueda libre". Ya es posible crear repositorios de diseños de hw libre que nos pertenezcan a todos y que los podamos hacer evolucionar entre todos. Compartirlos. Mejorarlos. Estos diseños se podrán sintetizar con las herramientas libres. Y es un conocimiento que perdurará en el tiempo

## Limitaciones

Ninguna herramienta recién nacida tiene todo lo que deseamos. Pero al ser libre, potencialmente cualquier característica se puede implementar. Por eso todos los sistemas de software/hardware libres, evolucionan y maduran con el tiempo.  Einstein también fue bebé, y con esa edad no podía crear sus teorías. Lo importante es el potencial.

Las herramientas del [proyecto icestorm](http://www.clifford.at/icestorm/) acaban de nacer. Y tienen todavía que madurar y desarrollarse. Algunas limitaciones son:
* Sólo sirven para las **FPGAs de Lattice**, modelos: **HX1K-TQ144** y **HX8K-CT256**
* Las herramientas sólo cubren el bajo nivel: se usan en la **línea de comandos**. No hay un entorno gráfico que permita gestionar proyectos. Hay que hacerlo a base de makefiles
* El soporte a puertas triestado todavía es muy limitado
* No hay soporte para análisis de tiempo post-rutado

## Las herramientas del proyecto Icestorm

Las herramientas libres para trabajar con las FPGAs de lattice son las siguientes:

**Sintetizador**: [Yosys](http://www.clifford.at/yosys/) ([Repo en github](https://github.com/cliffordwolf/yosys))
**Place & route**: [Arachne-pnr](https://github.com/cseed/arachne-pnr) (en github)
**Utilidades y descarga en FPGA**: [Proyecto icestorm](http://www.clifford.at/icestorm/)

En la siguiente figura se muestran las diferentes herramientas usadas en las etapas, y las extensiones de los archivos que se van generando:

![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/ICESTICK/T00-Intro/images/icestorm-1.png)

Se parte de los ficheros **fuente en verilog** (.v). Usando el sintetizador **Yosys**, se generan los ficheros **netlist** (.blif). El emplazado y rutado se realiza con **arachne-pnr**, generándose el **bitstream en formato ascii** (.txt). Con **icepack** se crea el **bitstream binario** (.bin) que finalmente se envía a la FPGA con **iceprog**

En la **linea de comandos**, los pasos a seguir para llevar el fichero test.v hasta la FPGA serían:

```
$ yosys -p "synth_ice40 -blif test.blif" test.v
$ arachne-pnr -d 1k -p test.pcf test.blif -o test.txt
$ icepack test.txt test.bin
$ iceprog test.bin
```

## Herramientas libres para simulación

Para diseñar los circuitos es fundamental disponer de un simulador de verilog. Las herramientas libres que usaremos son:

* **Simulador de Verilog**: [ícarus Verilog](http://iverilog.icarus.com/) 
* **Visualizador de señales**: [Gtkwave](http://gtkwave.sourceforge.net/)

**Icarus verilog** crea un fichero ejecutable a partir del código Verilog. Al ejecutarlo se realiza la simulación. Los resultados se vuelcan a un **fichero .vcd** que se visualiza con la herramienta **Gtkwave**. Esto nos permite inspeccionar las señales para comprobar su correcto funcionamiento

# Instalación

## Ubuntu 14.04, 15.10

### Instalación automática, con instalador

David Cuartielles ha creado [este instalador](https://github.com/dcuartielles/open-fpga-install) que realiza todo el proceso automáticamente: se baja todas las herramientas del github, las compila y las instala, así como todas las dependencias necesarias. La forma de usarlo es:

```
git clone https://github.com/dcuartielles/open-fpga-install.git
cd open-fpga-install
sudo bash install.sh
```

### Instalación Manual

Instalación de **dependencias**:
```
sudo apt-get install build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev git mercurial graphviz  xdot pkg-config python python3 libftdi-dev
```
Instalación de **IceStorm Tools** (icepack, icebox, iceprog):
```
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
cd ..
```
Instalación de **Arachne-PNR** (the place&route tool):
```
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
cd ..
```
Instalación de **Yosys** (Verilog synthesis):
```
git clone https://github.com/cliffordwolf/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
cd ..
```

Instalación de **Icarus Verilog** y **GTKwave**
```
sudo apt-get install gtkwave iverilog
```

## Fedora 22

Instalación de **dependencias**:
```
sudo dnf install libftdi-devel tcl-devel readline-devel flex clang bison gawk libffi-devel git mercurial graphviz python python3
```

Instalación de IceStorm Tools (icepack, icebox, iceprog):
```
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
cd ..
```
**NOTA:** si aparecen errores relacionados con "ftdi.h" puede ser necesario enlazar la librería FTDI de este modo:
```
sudo ln -s /usr/lib64/libftdi1.so /usr/local/lib/libftdi.so
sudo ln -s /usr/include/libftdi1/ftdi.h /usr/local/include/ftdi.h
```

Instalación de **Arachne-PNR** (the place&route tool):
```
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
cd ..
```

Instalación de **Yosys** (Verilog synthesis):
```
git clone https://github.com/cliffordwolf/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
cd ..
```

Instalación de **Icarus Verilog** y **GTKwave**
```
sudo dnf install iverilog gtkwave
```
Adicionalmente, para poder ejecutar "sudo iceprog" hay que enlazar:
```
sudo ln -s /usr/local/bin/iceprog /usr/bin/iceprog
```

Si lo que quieres es **actualizar** las herramientas, puedes re-utilizar los repositorios que clonaste durante la instalación, usando el comando "git reset --hard & git pull" (revertir cambios locales y descargar la última versión):
```
cd icestorm
git reset --hard & git pull
make -j$(nproc)
sudo make install

cd ../arachne-pnr
git reset --hard & git pull
make -j$(nproc)
sudo make install

cd ../yosys
git reset --hard & git pull
make -j$(nproc)
sudo make install
```

# Placa ICEStick

La placa que usaremos en estos tutoriales es la [IceStick](http://www.latticesemi.com/icestick)
Si no dispones de ella (o de otra similar), ¡No problem! También simularemos todos los diseños con icarus verilog y gtkwave

## Configuración
La **descarga del bitstream** a la placa icestick se hace directamente por **USB**, usándose la biblioteca [libftdi](https://www.intra2net.com/en/developer/libftdi/). Para ello es necesario tener **permisos de acceso**.

Si intentamos cargar en la icestick sin permisos, obtendremos el siguiente **mensaje de error**:

```
$ iceprog scicad1.bin
init..
Can't find iCE FTDI USB device (vedor_id 0x0403, device_id 0x6010).
ABORT.
```

Una manera de solucionarlo es utilizar **sudo** al ejecutar **iceprog**, descargando con el comando:

    $ sudo iceprog bitstream.com

Esto tiene el inconveniente de que hay que estar metiendo la clave cada cierto tiempo.

La otra forma es **configurar el sistema [udev](https://es.wikipedia.org/wiki/Udev)** para que al conectar la icestick al USB **el usuario tenga permisos**. Para ello hay que hacer lo siguiente:

* Crear el archivo **/etc/udev/rules.d/80-icestick.rules** con el siguiente contenido
```
ACTION=="add", SUBSYSTEM=="usb", ATTRS{idVendor}=="0403",
 ATTRS{idProduct}=="6010", OWNER="user", GROUP="dialout",
 MODE="0777"
```
* Ejecutar este comando para relanzar el administrador de [udev](https://es.wikipedia.org/wiki/Udev) y cargue la nueva regla:

```
$ sudo udevadm control --reload-rules && sudo udevadm trigger 
```

Ahora ya se puede hacer la descarga directamente ejecutando:

```
$ iceprog bitstream.bin
```

En ubuntu es necesario reiniciar la máquina para que funcione correctamente

## Cómo hacer los tutoriales

Para realizar estos tutoriales, y poder probar todos los ejemplos, hay que seguir los siguientes pasos:

1. **Clona el repo de los tutoriales**, para tener en tu ordenador todos los ficheros:
    
    ```
    $ git clone https://github.com/Obijuan/open-fpga-verilog-tutorial.git
    ```
    
2. **Entra en el directorio de trabajo** del tutorial que toca. Por ejemplo, para el 1 sería:
    
    ```
$ cd open-fpga-verilog-tutorial/tutorial/ICESTICK/T01-setbit/
    ```
    
3. **Simula y sintetiza**

En cada tutorial se irán indicando los comandos a ejecutar. En el caso del primero, para **realizar la simulación** hay que ejecutar:

```
make sim
```
* Para **sintetizar**:
```
make sint
```

y finalmente para **configurar la FPGA**:

```
$ iceprog setbit.bin
```

## Documentación
* [Manual de la ICEstick](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/icestickusermanual.pdf)(PDF)

* [Hoja de datos FPGA iCE40LPHXF](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/iCE40LPHXFamilyDataSheet.pdf)(PDF)

## Pinout de la Icestick
![](https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/doc/images/icestick_pinout.png)

# Enlaces
* [Proyecto Icestorm](http://www.clifford.at/icestorm/) ([Github](https://github.com/cliffordwolf/yosys))
* [Página personal de Clifford Wolf](http://www.clifford.at/)
* [Sintetizador Yosys](http://www.clifford.at/yosys/) ([Github](https://github.com/cliffordwolf/icestorm))
* [Emplazamiento y enrutado: arachne-pnr](https://github.com/cseed/arachne-pnr)
* [Gtkwave](http://gtkwave.sourceforge.net/)
* [Icarus Verilog](http://iverilog.icarus.com/) ([Github](https://github.com/steveicarus/iverilog))

# Créditos
* "AND ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:AND_ANSI.svg#/media/File:AND_ANSI.svg
* "OR ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:OR_ANSI.svg#/media/File:OR_ANSI.svg
* "NOT ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:NOT_ANSI.svg#/media/File:NOT_ANSI.svg
* "XOR ANSI" by jjbeard - Own Drawing, made in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:XOR_ANSI.svg#/media/File:XOR_ANSI.svg
* "D-Type Flip-flop" by Inductiveload - Own Drawing in Inkscape 0.43. Licensed under Public Domain via Commons - https://commons.wikimedia.org/wiki/File:D-Type_Flip-flop.svg#/media/File:D-Type_Flip-flop.svg


