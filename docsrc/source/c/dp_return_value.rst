.. _c_return_value:

#################
C - Return Value
#################

https://www.youtube.com/watch?v=2mKJxtWejGY


Return Value = sized integer

  * <0  Fehler
  * 0 OK
  * >0 partiell erfolgreich

Every non-trivial function must return integer status code
return values must be checked
error codes are always constant
Use **errno.h**, negative error codes correspond to errno.h

Use parameters for complex return values. Write this complex value into a struct pointed to be one of your functions parameters. 
Do not return complex structures as return values. 

Pitfall: when you need to return a struct it is better to pass it as a parameter and then simply return an integer status code signaling whether 
the struct parameter was filled with valid data or not

Alternatives: 

    Exception -> nicht möglich
    
    Logging
    
    Panic
    
    Long Jump -> sollte man nicht nutzen, weil dann das CleanUp ggfs. nicht funktioniert. Besser ist hier dann ein panic. 

 
 A common mistake is to do

.. code-block:: c

    if (somecall() == -1) {
            printf("somecall() failed\n");
            if (errno == ...) { ... }
        }

where errno no longer needs to have the value it had upon return from somecall() (i.e., it may have been changed by the printf(3)).  If the value of errno should  be  preserved
across a library call, it must be saved:

.. code-block:: c

        if (somecall() == -1) {
            int errsv = errno;
            printf("somecall() failed\n");
            if (errsv == ...) { ... }
        }

On  some  ancient systems, <errno.h> was not present or did not declare errno, so that it was necessary to declare errno manually (i.e., extern int errno).  Do not do this.  It
long ago ceased to be necessary, and it will cause problems with modern versions of the C library.

Mit errno -l kann man sich die definierten Error-Codes für die Plattform ansehen (Enthalten im Paket moreutils)

## Anzeigen des Fehlers

mit der Funktion perror in <stdlib.h>

    void perror(const char *s)
    Bsp.: perror("open"); -> zeigt dann den Fehlercode an mit voranstehendem "open". 

mit der Funktion strerror in <string.h>. strerror konvertiert nur die Nummer in einen Text, der dann ausgegeben werden kann. 

    char *strerror(int errnum); 
    Bsp.: write(2, strerror(errno), strlen(strerror(errno)));  // 2 = stderr, wandle errno in text, länge des Textes

