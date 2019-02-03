# Readme
This guide will help you to setup a Freedom E300 Core on a Digilent Arty A7 board.
Some information was deducted from the 
[offical guide](https://static.dev.sifive.com/SiFive-E310-arty-gettingstarted-v1.0.6.pdf).
Sadly the official guide the official guide is partially outdated and misses some
parts that where important for me.
Also due to a reconstruction of the SiFive homepage some links will lead to 404 pages.

## What is the Freedom E300 platform
The E300 platform is based on the SiFive E31 RISC-V Core ip 
(Documentation -> https://www.sifive.com/documentation/).  
The SiFive E31 IP implements one in-order, 32 bit hart (hardware thread).  
A SiFive E31 can't run linux. Instead we will run compiled C-code on it.  
The following RISC-V extension are included:
- M (multiplication and division)
- A (atomic memory access)
- C (compressed instructions, allows for 16-bit RISC-V instructions)

## Hardware/Software requirements
PC with: (you can outsorce the compile process to a compile server, )
- Ubuntu/Debian (other OSs are possible, but you are on your own)
- 8GB of RAM (4GB if you execute the build processes sequentially. With IntelliJ and some web browser tabs open 16GB is very recommendable)
- Dual core CPU (more is definitly better)

## Setup Steps
I recommend to do the steps, that require downloading many files, in parallel to speed up the progress. 
You will need about 70GB of space on your hard drive for the full installation.  
A fast internet connection will also be helpful (thought many files are compress for the download).

### Install Java-8
You want to have the 8. version of Java.
I encountered many problems while I used the 11. version of Java to compile the RISC-V source with sbt.  
Execute: 
```console
sudo apt-get install openjdk-8-jdk openjdk-8-demo openjdk-8-doc openjdk-8-jre-headless openjdk-8-source ca-certificates-java -y
java -version
```
If Java version 8 is reported, you are good to continue.
If not -> https://www.linuxnix.com/how-to-change-the-java-version-in-linux/

### Install other dependencies
The compilation of the Freedom and Freedom E SDK repo require a variety of dependencies.  
Not all are listed in the official repository.  
Execute this to get them all  
```console
sudo apt-get install git autoconf automake libmpc-dev libmpfr-dev libgmp-dev gawk bison flex texinfo libtool libusb-1.0-0-dev make g++ pkg-config libexpat1-dev zlib1g-dev device-tree-compiler -y
```

### Install Vivado
To download Xilinx Vivado you will need to create a account on their website.  
Do so here: https://www.xilinx.com/registration/create-account.html

To download the software follow [this link](https://www.xilinx.com/support/download.html)
and scroll down to "Vivado HLx 2018.2: WebPACK and Editions - Linux Self Extracting Web Installer"
and download it.  
After the download has finished, execute it in the console with
```console
./*installer*.bin
```
In the installer: sign in, read threw all licenses and requirements ;) and 
select Vivado HL WebPACK to install.  
In this guide we assume that Xilinx Vivado is installed to /home/*user*/Xilinx/ (then you don't need root).  
Warning after the installation: Do not upgrade to 2018.2.2.
That broke the Arty A7 Board support for me.  
The installation will take some time. Continue with the next step.

After the installation has finished we need to source the settings file of Vivado.
This will make Vivado executable from every terminal window.
Execute
```console
sudo nano ~/.bashrc
```
and paste the following line at the end of the file:
```console
source ~/Xilinx/Vivado/2018.2/settings64.sh
```

### Install Vivado board files
Do after Vivado installation has finished.  
Download the files from https://github.com/Digilent/vivado-boards/archive/master.zip
and copy vivado-boards-master/new/board_files/* to Xilinx/Vivado/2018.2/data/boards/board_files

### Install the drivers for the JTAG
Do after Vivado installation has finished.
The JTAG needs a driver to work under Linux, which is not automatically installed
Without this driver OpenOCD won't recognize the adapter.
Execute
```console
sudo ~/Xilinx/Vivado/2018.2/data/xicom/cable_drivers/lin64/install_script/install_drivers/install_drivers
```

### Create a base folder
This guide will asume you checkout all files to /home/*user*/riscv/ (~/riscv).
If you do not have this folder yet, execute
```console
mkdir ~/riscv
```

### Checkout and compile Freedom E SDK
The Freedom E SDK repo provides you with the C compiler that you need to compile
programs for the SiFive E31 core.  
Execute (replace n with the count of cores your machine has)
```console
cd ~/riscv
git clone --recursive https://github.com/sifive/freedom-e-sdk.git
cd freedom-e-sdk

export MAKEFLAGS="$MAKEFLAGS -j*n*"
make tools
```

### Checkout and compile Freedom
The Freedom repo is required if you want to change the Chisel code for the base image of a
RISC-V. A precompiled file without changes exists.  
Execute 
```console
cd ~/riscv
git clone https://github.com/sifive/freedom.git
cd freedom
git submodule update --init --recursive
```  

To continue with the normal build of the Freedom repo execute
(replace n with the count of cores your machine has. Reduce n if you run out of memory)
``` console
export MAKEFLAGS="$MAKEFLAGS -j*n*"

make -f Makefile.e300artydevkit verilog
export RISCV=~/riscv/freedom/rocket-chip/riscv-tools/installation
export PATH=$RISCV/bin:$PATH
./rocket-chip/riscv-tools/build.sh
./rocket-chip/riscv-tools/build-rv32ima.sh

(next step needs a complete installation of Vivado)
make -f Makefile.e300artydevkit mcs
```


### Install Arduino IDE
Download it from https://www.arduino.cc/en/Main/Software, unpack it to
for example /home/*user*/Arduino/.  
Go to *File -> Preferences* and paste http://static.dev.sifive.com/bsp/arduino/package_sifive_index.json
into the field next to "Additional Boards Manager URLs".
Change the board to Freedom E300 Arty DevKit via *Tools -> Board*
Change the programmer to SiFive OpenOCD via *Tools -> Programmer*

If you are using the Olimex ARM-USB-TINY-H JTAG you are fine.  
If you are using the Olimex ARM-USB-OCD-H JTAG replace the file at
/home/*user*/.arduino15/packages/sifive/hardware/riscv/1.0.2/freedom-e-sdk/bsp/env/freedom-e300-arty/openocd.cfg  
(path might change in future versions) with the version from this repo.

### Install and setup IntelliJ
Download the newest version of IntelliJ from https://www.jetbrains.com/idea/download/#section=linux  
The Community version is sufficient.
Unpack it to /home/*user*/IntelliJ/ and run it with ~/IntelliJ/*version_root*/bin/idea.sh
In the install process select the option to install Scala
(can also be done later, hit CTRL+A and type sbt. Options to enable sbt will show up).
Import the folder ~/riscv/freedom and select SBT as build tool.
If you are asked for the compilation tool, select the sbt shell.
After the project has been imported, you need to change the Scala cross-compile version, else the project won't build.
Enter the following into the sbt shell in IntelliJ
```++2.12.4```
(Scala version might change in the future, use the version which is listed in the Makefile)

Further information can be found here: https://github.com/sifive/freedom/pull/98


### Troubleshooting

#### Error unresolved dependency: org.scala-sbt#sbt;1.1.1: not found
Check your Java version. You need Java 8, Java 11 gave me funny errors.
Also check if the package ca-certificates-java is installed.

#### This is taking so long
Yes the compile process takes quite some time, especially if you are on a dual core platform but remember:
you are compiling a complete CPU and the corresponding GNU C compiler with it.  
If your machine is overheating or takes a very long time to compile, it might be a good
idea to outsorce the compilation process to a compile server with many cores.
On my trusty AMD Ryzen 2600 @ 4.1 GHz with six cores it took about 40 minutes to install and compile everything.

#### If have found an error in the guide
It is possible that I made a mistake while I wrote this guide. If you find one open a issue or if you have some
time on your hands fork the project and submit a pull request.  
Thanks in advance.

#### If have got a different problem
First of all ask Google and DuckDuckGo. Also have a look at the [RISC-V forum](https://riscv.org/forum/).
If they can't help you, open a issue here. I'm not a total expert in the field but maybe I can help you.  
The last resort would be the [RISC-V mailing lists](https://riscv.org/mailing-lists/).

