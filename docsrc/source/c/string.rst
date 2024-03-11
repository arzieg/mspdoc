.. _c_string:

#######
String
#######

Parsing a string of numbers in c
=================================
https://www.youtube.com/watch?v=L8hVbPIVE0U&list=PLfqABt5AS4FmSwyvP5a3mYsaksq6yR3-Z&index=14

.. code-block:: c

    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>

    int main(int argc, char* argv[]) {

      char str[] = "100 10 222";
      char* cursor = str;
      long int sum = 0;
      while( cursor != str+strlen(str)){
        // string to long (strtol)
        long int x = strtol(cursor, &cursor, 10);   // base = 10
        while (*cursor = ' ' || *cursor == ',')  // behebt Problem 1) und 2)
          cursor++;   
        sum += x;
      }
      printf("Sum in %ld\n", sum);
      return 0;
    }

1) Issue: spaces am Ende führt zu einem Endloop;
2) Kommata im String
3) eine Mischung von Integer und Hex ist auch möglich

How to parse and validate a number in c
========================================
https://www.youtube.com/watch?v=js2kolXUQ8g&list=PLfqABt5AS4FmSwyvP5a3mYsaksq6yR3-Z&index=15

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <time.h>
    #include <errno.h>

    int main(int argc, char* argv[]) {
      char str[100] = "120";
      char* endPtr;
      long int x = strtol(str, &endPtr, 10);   // endPtr zeigt auf den Wert nach dem geparsten Wert
      if (str == endPtr) {
        printf("\nNumber could not be parsed"); // issue 1)
        return 0;
      }
      if (errno == ERANGE) {  // issue 2
        printf("\nNumber ist to big to store in the variable!\n");
        return 0;
      }
      printf ("\nx=%ld \n",x );
      return 0;
    }

Issue: 
1) Wert ist ein Wort und keine Nummer? endPtr kann verwendet werden. Wenn dort ein Wort im String steht, dann sind Adresse(str) == Adresse(endPtr). Wenn es eine Nummer ist, 
   dann unterscheiden sich die Werte. 
2) Was wenn die Zahl länger ist als MaxInt? Dann wird Fehlermeldung zurückgegeben. Hierfür ist #include <errno.h> aufzunehmen.


Strpbrk and strspn in c
=========================
https://www.youtube.com/watch?v=eeMEq510A5U&list=PLfqABt5AS4FmSwyvP5a3mYsaksq6yR3-Z&index=17

Extrahiere die Zahl aus einem String

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    int main(int argc, int *argv[]){
      char str[] ="Daniel is 25 years old"
      char *age = strpbrk(str, "0123456789");  // returns a pointer to the first occurence

      size_t number_of_digits = strspn(age, "0123456789");   // gibt die Anzahl der gefunden Werte zurück, die im Suchstring enthalten sind

      for (int i=0; i< number_of_digits; i++) {
        printf("%c", age[i]);
      }

      return 0;

    }

Issue: bei strspn muss man darauf achten, dass der erste Wert auch im Suchmuster ist, ansonsten wird eine 0 zurückgegeben. Wenn also Bspw. strspn(str, "0123456789") dort steht, 
       dann ist str[0]="D" <> dem Suchmuster und die Funktion bricht ab mit einem Returnwert 0; 
       