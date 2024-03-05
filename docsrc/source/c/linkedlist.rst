.. _c_linkedlist:

#############
Linked List
#############

https://www.youtube.com/watch?v=C4JXCFlIf6Q&list=PLn3A1FGnKiUyrMnXjwShy0LVV3zTgYg_z&index=9

Remove Element
===============

Es soll ein Element aus einer Linked List gelöscht werden. 

Prototype
----------

Funktion muss folgendes berücksichtigen. Das zu löschende Element befindet sich 

1. head
2. middle
3. tail
4. empty list: 0 length list
5. List mit Länge 1
6. List mit Länge 2
7. Element ist nicht vorhanden


Implementierung
----------------

.. code-block:: c

    typedef struct Element{
      struct Element *next;
      int data;
      } Element;

    int removeL(Element *elem);
 
    Element *head, *tail;
    
    int removeL(Element *elem){
    
        Element *cur=head;
    
        if (!elem)
            return 1;
        
        // case 1 = head
        if(elem==head){
            head=elem->next;  // wichtig, nicht free head, da head pointer bereits verschoben worden ist.
            free(elem);
             // case 5, Liste mit Länge = 1
            if (!head)
                tail=NULL;      
            return 0;
        }
        
        // case 2 = middle
        while(cur){
            if (cur->next ==elem){   // Abfrage der vorherigen Pointers, zeigt dieser auf das nächste Element, dann Pointer-Manipulation
                cur->next=elem->next;  // Pointer auf das darauffolgende Element zeigen
                free(elem);
                // case 3 = last element
                if (cur->next==NULL)   // wenn der aktuelle Pointer auf NULL zeigt, dann bin ich am Ende der Liste
                    tail=cur;
                return 0;
            }
            cur=cur->next;
        }
    
         // case 4 und 7, 0-Liste oder aber Element nicht vorhanden
        return 1;
    }
    
    void printL(){
        Element *cur=head;
        
        printf("List:");
        
        while(cur){
            printf(" %d ",cur->data);
            cur=cur->next;
        }
        
        printf("\n");
    }
    
    int main(){
        
        /* create a 3 element linked list */
        head=(Element *)malloc(sizeof(Element *));
        head->data=5;
        
        head->next=(Element *)malloc(sizeof(Element *));
        head->next->data=6;
        
        head->next->next=(Element *)malloc(sizeof(Element *));
        head->next->next->data=7;
        
        tail=head->next->next;
        
        /* print out the list */
        printL();
        
        /* remove element (try head, head->next or head->next->next in the argument) */
        removeL(head->next);
        
        /* reprint the modified list */
        printL();
        
            
        return 0;
    }

    