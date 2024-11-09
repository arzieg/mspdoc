# GDB


Quelle: 
https://developers.redhat.com/blog/2021/04/30/the-gdb-developers-gnu-debugger-tutorial-part-1-getting-started-with-the-debugger#why_another_gdb_tutorial_

https://developers.redhat.com/articles/2022/01/10/gdb-developers-gnu-debugger-tutorial-part-2-all-about-debuginfo

https://sourceware.org/gdb/current/onlinedocs/gdb.html/


## Kompilierung

Regel beim Debuggen: Compile without Optimizing
  -O0 bei der Entwicklung
  -g3 debug flag (inkl. macro flags)

## GDB Startprozess

Bei dem GDB start werden mehrere Files ausgeführt:
1. /etc/gdbinit und Files in /etc/gdbinit.d  (globale Konfiguration)
2. $HOME/.gdbinit (lokale Konfiguration im Userkontext)
3. ./.gdbinit  (lokales Script im Entwicklungsverzeichnis)

Beispiel: analog wie .bash_history

```
set pagination off
set history save on
set history expansion on
```

## Help

help <comand> inkl. tab Komplementierung

apropos ebenfalls nutzbar

## History

kann in die .gdbinit eingetragen werden. GDB unterstützt STRG+R

```
set history save on
set history size 10000
set history filename ~/.gdb_history
```

## Weitere Einstellmöglichkeiten

set listsize 20  -> zeigt 20 Zeilen bei list an

## Start

-q       quite, d.h. keine Startinformation

-tui     text ui

--args   Argumente, gdb -q --args prg 1 2 3 4

--pid    Attach to a running proccess

--core   Debuggen eines Cors files, i.d.R. benötigt man hierzu auch das executable (gdb -q <prg> --core <corefile>)

Wenn kein Core File gefunden wird, dann per ulimit -c prüfen, ob auf unlimited steht. Wenn ja, zeigt coredumpctl den Speicherort an.

coredumpctl debug (Ruft direkt GDB auf und lädt den letzten Coredump)

coredumpctl zeigt die coredumps an, dann mit *coredumpctl debug <pid>* ruft den coredump in gdb auf. Sehr praktisch

in Linux Mint wird Core durch systemd geschrieben und komprimiert in /var/lib/systemd/coredump/<file>.zst
Dies kann von gdb nicht gelesen werden. Vorgehen:

```
- uncompress mit coredumpctl -o mycorefile dump /home/arne/dev/c/cex/romemory/readonly
- dann gdb /home/arne/dev/c/cex/romemory/readonly --core mycorefile
```

Befehlsautomatisierung:

--ex CMD   run the command after gdb and program is loaded

--iex CMD  analog, aber bevor das Programm geladen wird

-x FILE    executes GDB commands from File after program is loaded and --ex commands execute

--batch    exit immediately at the first command prompt

--tui  grafische Benutzeroberfläche


## Befehl-Kurzreferenz

| Command    | Description |
| -------- | ------- |
| run or r  | Executes the program from start to end.    |
| break or b  | Executes the program from start to end.    |
| break *line*  |     |
| break *file_name:line*  |     |
| break *function_name*  |     |
| disable  | Disables a breakpoint    |
| enable  | Enables a disabled breakpoint.    |
| next [n] | Executes the next line of code without diving into functions.    |
| step [n] | Goes to the next instruction, diving into the function.    |
| list *sourceline* | Displays the code.    |
| list  *function* |     |
| list  *+/-* |  list + 10 lines, list - 10 lines   |
| l |     |
| print  | Displays the value of a variable.    |
| display *expression*  | Print value of expression each time the program stops.    |
| delete display  | Cancel some expressions to be displayed when program stops.    |
| clear  | Clears all breakpoints.    |
| continue  | Continues normal execution    |
| until  | Execute until the program reaches a source line greater than the current. If your program enters a for loop and you grow weary of typing next, you can use the command to exit the loop.    |
| finish  | Execute until current function return.   |
| kill  | Kill execution of the program being run. Typically used to prepare to re-start the program from the beginning.   |
| k  |    |
| shell *cmd* | run shell command   |
| \<return\>  | repeat prevous command   |



## Breakpoint management

| Command    | Description |
| -------- | ------- |
| delete  | Delete all breakpoints   |
| clear *function*  | Deletes any breakpoints set on at the entry of function.  |
| clear *line*  | Deletes any breakpoints set on the line.  |


## Print

### Artificial Arrays

It is often useful to print a contiguous region of memory as if it were an array. A common occurrence of this is when using malloc to allocate a buffer for a list of integers.

```
int num_elements = 100;
int *elements = malloc(num_elements * sizeof(int));
```

We can print this entire array using one of two ways. First, we can cast it to a int[100] array and print that.

```
(gdb) p (int[100])*elements
$10 = {0, 1, 2, 3, 4, 5, ...
```

Or we can use GDB’s artificial arrays! An artificial array is denoted by using the binary operator @. The left operand of @ should be the first element in thee array. The right operand should be the desired length of the array.

```
(gdb) p *elements@100
$11 = {0, 1, 2, 3, 4, 5, ...
```


## Examine of Stack

| Command    | Description |
| -------- | ------- |
| bt   | Print a backtrace of all the active functions on the execution stack. This is very useful in determining the location of execution, and order in which functions call each other.   |
| where   |    |
| frame *number*   |  Select and print a stack frame.  |
| up   |  Select and print a stack frame (function) that called this one.  |
| down  |  Select and print a stack frame called by this one.  |
| info register | print register    |
| p *register*  | print register variable p $sp, p $pc    |

## Listing Memory Regions

| Command    | Description |
| -------- | ------- |
| info files   | list memory regions - helpful when you are trying to figure out exactly where a variable exists in memory.   |
| x   | examine memory    |

https://interrupt.memfault.com/blog/advanced-gdb

1. find size of the stack:

```
(gdb) p sizeof(my_stack_area)
$1 = 2980
```

It’s 2980 bytes, so we want to print 2980/4 = 745 words. That should be x/745a then.

```
(gdb) x/745a my_stack_area
```

 This stack dump technique is especially useful if your GDB provides you with no backtrace information, such as:

 ```
(gdb) bt
#0  0x00015f5a in ?? ()
 ```

### Searching Memory using find

Sometimes you know a pattern that you are looking for in memory, and you want to quickly find out if it exists in memory on the system. Maybe it’s a magic string or a specific 4-byte pattern, like 0xdeadbeef.

Let’s search for the string shell_uart, which is the task name of a thread in my Zephyr system. I’ll search the entire writeable RAM space, which can be found by running the info files command mentioned previously.

```
(gdb) find 0x20000000, 0x200117e0, "shell_uart"
0x2000121c <shell_uart_thread+104>
1 pattern found.
```

If we use x/s to examine that memory, we can see that it is indeed shell_uart.

```
(gdb) x/s 0x2000121c
0x2000121c <shell_uart_thread+104>:	"shell_uart"
```

The find command can also be useful for finding pointers pointing to arbitrary structs. For example, I want to find all the pointers that contain a reference to the variable mgmt_thread_data.

```
(gdb) find 0x20000000, 0x200117e0, &mgmt_thread_data
0x20002a18 <tx_classes+120>
0x2000d068 <mgmt_stack+648>
0x2000d074 <mgmt_stack+660>
0x2000d088 <mgmt_stack+680>
0x2001169c <network_event>
0x200116a0 <network_event+4>
6 patterns found.
```

It looks like there are some references on the mgmt_stack, and a few other references, which are actually pointers from linked lists.

Using find can help track down memory leaks, memory corruption, and possible hard faults by seeing what pieces of the system are continuing to reference memory or values when they shouldn’t.


## Conditional Breakpoints and Watchpoints

A conditional breakpoint in GDB follows the format *break WHERE if CONDITION*. It will only bubble up a breakpoint to the user if CONDITION is true.

```
(gdb) break compute_fft if num_samples == 0xdeadbeef
```

You can also do the same with watchpoints, which will only prompt the user in GDB if the conditional is true.

```
(gdb) watch i if i == 100

(gdb) info watchpoints
Num     Type           Disp Enb Address    What
1       hw watchpoint  keep y              i stop only if i == 100
```

## Struct

### sizeof

You can easily get the size of any type using sizeof within GDB, as you would in C.

### offsetof

Gibt es so nicht, man kann aber ein Macro schreiben und in .gdbinit schreiben

```
(gdb) macro define offsetof(t, f) &((t *) 0)->f
```

With this in place, we can now print the offset of any struct members.

```
(gdb) p/d offsetof(struct k_thread, next_thread)
$3 = 100
```


## Fehler

### no debugging symbols found

gdb meldet: no debugging symbols found. 

Sowohl Compilierung als auch linken muss mit -g erfolgen, also CFLAGS als auch LDFLAGS mit -g erweitern, damit debug Informationen erzeugt werden. 


## Weitere brauchbare Programme

xxd  - hexdump 
objdump - hexdump 

