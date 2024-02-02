.. _c_allg:

###
C
###

Install SUSE Linux
====================
https://linuxconfig.org/how-to-switch-between-multiple-gcc-and-g-compiler-versions-on-ubuntu-20-04-lts-focal-fossa

YaST2->Software->Filter auf Pattern wechseln
  C/C++ und Kerneldevelopmen auswählen

Installation weiterer Compiler Versionen:
    
    zypper in gcc13
    
    zypper in gcc12
    
    zypper in gcc13-c++
    
    zypper in gcc12-c++

Alternatives definieren
    
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 7
    
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-7 7
    
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12
    
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 12
    
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 13
    
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-13 13

Change configuration
    update-alternatives --config gcc
    
    update-alternatives --config g++

Check version
    gcc --version
    
    g++ --version

GDB installieren
   
    zypper in gdb