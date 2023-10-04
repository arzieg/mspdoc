.. _tf_allg:

###############
Terraform
###############

Variablen:
===========

.. code-block:: bash

   variable "NAME" {
    description = "..."
    type = number | list | list(number) | map(string) ...
    default = 42 | ["a","b","c"] | [1,2,3] | { key1 = "value1"
                                               key2 = "value2"
                                               key3 = "value3" }
  }

im Code wird dann via var.<VARIABLE> der Wert eingefügt


Bei terraform apply ist zu unterscheiden: 

  terraform apply   - Interaktive Angabe der Parameter 
  terraform apply -var "port=8888"   - Ohne Abfrage
  terraform apply   - Ohne Abfrage, wenn Environmentvariable definiert ist TF_VAR_port=8888

