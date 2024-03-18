.. _c_valgrind:

#########
valgrind
#########

Valgrind is a program that checks for both memory leaks and runtime errors. A memory leak occurs whenever 
you allocate memory using keywords like new or malloc, without subsequently deleting or freeing that memory 
before the program exits. Runtime errors, as their name implies, are errors that occur while your program is 
running. These errors may not cause your code to crash, but they can cause unpredictable results and should be 
resolved.

https://students.cs.byu.edu/~cs235ta/labs/valgrind/valgrind.php

Aufruf: ``valgrind --leak-check=full --show-leak-kinds=all <prg> <arg1> <arg2>``

