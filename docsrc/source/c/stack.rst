.. _c_stack:

#######
Stack
#######

https://www.youtube.com/watch?v=XM328f3RqSc&list=PLn3A1FGnKiUyrMnXjwShy0LVV3zTgYg_z&index=8

Stack: allg. Daten die auf einem Stappel liegen. LIFO Prinzip, es gibt zwei Funktionen pop/push. push = ein Element hinzufügen, pop = ein Element abnehmen.



Implementierungsarten: 
-----------------------

1. Array
   ein Array wird definiert, wenn dies gefüllt ist, wird ein größeres Array definiert und das vorherige Array wird mit seinen Werten in das neue Array kopiert. 
   Nicht sehr effizient. 

2. Linked List
   Struct mit Data + Pointer to next Element. 
   Dann gibt es einen Pointer zum aktuellen Stackort. 

Prototypen: 
-------------

 * push-function: (Element \*\*stack, int \*data) -> return error-code (wenn z.B. kein Memory allokiert werden kann)
 * pop-function: (Element \*\*stack, int \*\*data) -> return data and error
 * create-function
 * delete-function 

   .. code-block:: c

      typedef struct Element {
        struct Element *next;  // pointer to the next Element
        int *data;   // Beispiel Daten, hier pointer to int
      };
      struct Element node; 

Zwei Probleme hier: 

   Problem beim Arbeiten am Stack. Wenn ein Pointer an eine c-function übergeben wird, dann arbeitet man an einer Kopie. D.h. wenn die Funktion endet, dann verliert 
   man die Änderungen in der Funktion. Die Lösung ist ein Pointer to the Pointer 
   Da Funktionen pass by value arbeiten, agiert man bei eine P2P auf der Kopie.
   Der Stack-Pointer2Pointer zeigt auf den StackPointer, der auf den Stack zeigt. 
   Wenn jetzt eine Funktion den StackPointer verschieben soll per Funktion, wird auf der Kopie des Stack-P2P gearbeitet, die aber weiterhin auf den Original 
   Stack-Pointer zeigt. Damit sind Manipulationen auf den Pointer möglich. 

   wenn zwei Werte zurückgegeben werden sollen (wie bei pop), dann hier die Lösung. Rückgabewert der Funktion ist der Errorcode. Rückgabe des Datenfeldes über
   ein P2P Kontrukt. 

Implementierung:
-----------------

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   typedef struct Element {
      int *data;
      struct Element *next;
   } Element;

   int push(Element **stack, int *data);
   int pop(Element **stack, int **data);
   int createStack(Element **stack);
   int deleteStack(Element **stack);

   int createStack(Element **stack) {
     *stack = NULL;
   return 0;
   }

   int push(Element **stack, int *data) {
      Element *elem;
      elem = (Element *)malloc(sizeof(Element *));
      if (!elem) return 1;
      elem->data = data;
      elem->next = *stack;
      *stack = elem;
      return 0;
   }

   int pop(Element **stack, int **data) {
      Element *elem;
      elem = *stack;
      if (!elem) return 1;
      *stack = elem->next;
      *data = elem->data;
      free(elem);
      return 0;
   }

   int deleteStack(Element **stack) {
      Element *next;

      while (*stack) {
         next = (*stack)->next;
         free(*stack);
         *stack = next;
      }
      return 0;
   }

   int main() {
      Element *stack;
      int i = 5;
      int *o;

      createStack(&stack);
      if (push(&stack, &i) == 1) printf("Error pushing\n");

      if (pop(&stack, &o) == 1) printf("Error poping\n");

      printf("o: %d\n", *o);

   return 0;
   }






