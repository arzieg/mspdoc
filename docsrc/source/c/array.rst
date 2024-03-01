.. _c_array:

######
Array
######

Falscher Code

.. code-block:: c

    char *return_string() {
      char buffer[] = "This is a local char buffer";  (2)
      return buffer
    
    int *return_int_arr() {
      int int_arr[] = {9,1,3};  (2)
      return int_arr;  (1)
    
    int main() {
      int a[10]; 
      a= return_int_arr();  (3)
      return 0;
    }

Fehler 1: man kann keine Arrays in einer Funktion zurückgeben

Fehler 2: int_arr[] sind lokale Variablen und kann nicht zurückgegeben werden

Fehler 3: man kann kein Array einer Funktion zuordnen. lvalue - error.

Variante 1: Return a string literal
====================================

.. code-block:: c

    char *func() {
      return "Returned String";
    }

(+) einfache Lösung

(-) Funktioniert nur mit Strings

(-) kann nicht verwendet werden, wenn der Inhalt berechnet werden soll

(-) manche compiler speichern strings in read-only Speicherbereich, das kann ein Problem werden, wenn man das später ändern möchte


Variante 2: use globally declared array
========================================

.. code-block:: c

    char global_arr[100];
    char *func() {

      ...
      global_arr[i] = ...;
      return global_arr;
    }

(+) Content kann kalkuliert werden

(-) jede Funktion kann globale Parameter überschreiben

(-) Das Array wird bei jedem Aufruf überschrieben. (caller needs to copy return value)

(-) Große Buffer können viel Speicher verschwenden

Variante 3: static array
=========================

.. code-block:: c

    char *func(){

      static char static_array[100];  // wird nur einmal initialisiert, da static
        // da static wird es im datenbereich des Prozess gespeichert.
      ...
      return static_arr;

(+) nur Pointer vom Caller kann das ändern (aufgrund des static keyword, nicht jeder)

(-) Das Array wird bei jedem Aufruf überschrieben. (caller needs to copy return value)

(-) Große Buffer können viel Speicher verschwenden


Variante 4: explicitly allocate memory to hold the return value
=================================================================

.. code-block:: c

    char *func() {

      char *buffer = malloc(100);
      ... 
      return buffer;
    }

(+) jeder Aufruf erzeugt einen neuen Buffer (wird also nicht überschrieben)

(-) Mögliche Memory management issues (wenn man etwas vergisst ;-) : 
    
    (-) Memory wird freigegeben, obwohl es noch in Verwendung ist

    (-) Memory wird nicht freigegeben (Memory leak)

Variante 5: Caller allocates memory to hold return value
=========================================================

**Best solution**

.. code-block:: c

    void func(char *result, int size) {

      ...
      strncpy(result, "Returned string", size);
      }
    
    int main(void) {
      char *buffer = malloc(size);
      func(buffer, size);
      ...
      free (buffer);
     }
   
(+) for safty, provide a count of the size if the buffer (like fgets in stdlib)

(+) Simplified Memory Management (free and malloc written by the same agent/caller)

(+) return from the function can be used for status code


Variante 6: wrap your array in a struct and return it
======================================================

eine mögliche aber nicht gängige Lösung!

.. code-block:: c

    #define SIZE 100

    struct Data {
      char buffer[SIZE];
      };

    struct Data func() {
      struct Data d;
      strncpy (d.buffer, "Returned string", SIZE);
      return d;
    }

(+) No Memory Management

(-) Fixed size array only

(-) costly for large arrays

