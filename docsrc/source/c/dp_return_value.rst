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


