.. _gdb:

#####
GDB
#####

Quelle: https://developers.redhat.com/blog/2021/04/30/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger#why_another_gdb_tutorial_

Kompilierung
=============
Regel beim Debuggen: Compile without Optimizing
  -O0 bei der Entwicklung
  -g3 debug flag (inkl. macro flags)

GDB Startprozess
=================
Bei dem GDB start werden mehrere Files ausgeführt:
1. /etc/gdbinit und Files in /etc/gdbinit.d  (globale Konfiguration)
2. $HOME/.gdbinit (lokale Konfiguration im Userkontext)
3. ./.gdbinit  (lokales Script im Entwicklungsverzeichnis)

Beispiel: analog wie .bash_history

.. code-block:: 

      set pagination off
      set history save on
      set history expansion on


Help
=====

help <comand> inkl. tab Komplementierung
apropos ebenfalls nutzbar

Start
======
-q       quite, d.h. keine Startinformation
--args   Argumente, gdb -q --args prg 1 2 3 4
--pid    Attach to a running proccess
--core   Debuggen eines Cors files, i.d.R. benötigt man hierzu auch das executable (gdb -q <prg> --core <corefile>)
         Wenn kein Core File gefunden wird, dann per ulimit -c prüfen, ob auf unlimited steht. Wenn ja, zeigt
         coredumpctl den Speicherort an.

         coredumpctl debug (Ruft direkt GDB auf und lädt den Coredump)

         in Linux Mint wird Core durch systemd geschrieben und komprimiert in /var/lib/systemd/coredump/<file>.zst
         Dies kann von gdb nicht gelesen werden. Vorgehen:

           - uncompress mit coredumpctl -o mycorefile dump /home/arne/dev/c/cex/romemory/readonly
           - dann gdb /home/arne/dev/c/cex/romemory/readonly --core mycorefile

Befehlsautomatisierung:

--ex CMD   run the command after gdb and program is loaded

--iex CMD  analog, aber bevor das Programm geladen wird

-x FILE    executes GDB commands from File after program is loaded and --ex commands execute

--batch    exit immediately at the first command prompt
