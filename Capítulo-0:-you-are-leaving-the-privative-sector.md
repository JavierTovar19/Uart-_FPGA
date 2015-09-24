<img src="https://github.com/Obijuan/open-fpga-verilog-tutorial/raw/master/tutorial/T00-Intro/images/checkpoint-charlie.png" 
width="400" align="center">

_You are leaving the privative sector_ ... ¡y estás entrando en el sector LIBRE! ¡Bienvenido! De aquí en adelante sólo usaremos herramientas del patrimonio tecnológico de la humanidad

# Introducción

Las [FPGAs](https://es.wikipedia.org/wiki/Field_Programmable_Gate_Array) son unos chips "en blanco" que nos permiten configurarlos para crear dentro de ellos nuestros propios circuitos digitales. ¡Si! ¡Con las FPGAs estamos creando hardware! 

Todos los circuitos digitales se descomponen en sus elementos básicos: **puertas lógicas** para hacer operaciones booleanas con los bits y **biestables** para almacenarlos. Como primera aproximación, podemos pensar en una FPGA como una chip que tiene en su interior arrays de estos elementos, sin conectar. Al configurarla, establecemos estas uniones y obtenemos nuestro circuito.

(dibujo)

# Entorno libre
TODO

Usaremos [icestorm](http://www.clifford.at/icestorm/), yosys y aracne, para la síntesis y descarga en la FPGA
Entorno: Ubuntu Linux 15.04

# Instalación

Fedora 22
--

Dependencias:
```
sudo dnf install ftdi-devel tcl-devel readline-devel flex clang bison gawk libffi-devel git mercurial graphviz python python3
```

Instalación de IceStorm Tools (icepack, icebox, iceprog):
```
git clone https://github.com/cliffordwolf/icestorm.git icestorm
cd icestorm
make -j$(nproc)
sudo make install
```
**NOTA:** si aparecen errores relacionados con "ftdi.h" puede ser necesario enlazar la librería FTDI de este modo:
```
sudo ln -s /usr/lib64/libftdi1.so /usr/local/lib/libftdi.so
sudo ln -s /usr/include/libftdi1/ftdi.h /usr/local/include/ftdi.h
```

Instalación de Arachne-PNR (the place&route tool):
```
git clone https://github.com/cseed/arachne-pnr.git arachne-pnr
cd arachne-pnr
make -j$(nproc)
sudo make install
```

Instalación de Yosys (Verilog synthesis):
```
git clone https://github.com/cliffordwolf/yosys.git yosys
cd yosys
make -j$(nproc)
sudo make install
```

Instalación de Icarus Verilog y GTKwave
```
sudo dnf install iverilog gtkwave
```

# Placa ICEStick

Esta es la placa que usaremos para probar todos los diseños digitales de este tutorial:
[Placa IceStrick](http://www.latticesemi.com/icestick)

Si no dispones de esta placa, ¡No problem! También simularemos todos los diseños con icarus verilog y gtkwave

# Enlaces
