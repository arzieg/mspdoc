.._c_pointer :

########
Pointer
########

Memory Leak 
=============

https://www.youtube.com/watch?v=cpbkXzd2TKY&list=PLfqABt5AS4FkIiyvV8mnZmf3p6PxbAtc8&index=13

Wenn dynamisches Memory Alloc benutzt wird, liegt Memoryverantwortung bei einem selbst. Dann free nutzen.

Beispiel Problem: 

.. code-block:: c

    void process_arr(int** arr, int){
      // some processing
      free(*arr);
      }

    int main(int argc, char* argv[]) {
      int* arr = malloc(sozeof(int)*10);
      process_arr(&arr, 10);
      free(arr);   <- Programm bricht ab, da hier wieder free aufgerufen wird
      return 0;
    }


Wenn möglich, immer im Block malloc und free verwenden, wo es verwendet wird. Weiterhin sollte nach 
einem free noch ein pointer=NULL gesetzt werden. Siehe Dokumentation zu free. Wenn Pointer auf NULL zeigt, 
macht free nichts. 

.. code-block:: c

    void process_arr(int* arr, int n){
      // some processing
      }

    int main(int argc, char* argv[]) {
      int* arr = malloc(sozeof(int)*10);
      process_arr(arr, 10);
      free(arr);   
      *arr= NULL; <- das noch am Ende hinzufügen, 
      return 0;
    }



Hi... to avoid dangling pointers, you could define a macro like 
"#define FREE(ptr) do { free(ptr); ptr = NULL; } while (0)" or 
"#define FREE(ptr) ({ free(ptr); ptr = NULL; })" and call FREE() instead of free()
