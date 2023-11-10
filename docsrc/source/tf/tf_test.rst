.. _tf_test:

###############
Terraform Test
###############

Allgemein
==========

Folgende Verzeichnisstruktur wird empfohlen:

.. code-block:: shell
    
    module
        example
            <hier der Example Aufbau>
        modules
            <hier die Module>
        test
            <hier der Testcode>

Manuelle Tests können z.B. über den Example - Code erfolgen. Dieser ist bei Änderungen ebenso anzupassen wie der Modulecode. 

Automatisierte Tests
=====================

Drei Arten von automatisierten Tests: 

1. UNIT Test  -> test eine kleinen Funktionalität des Codes
2. Integration Test -> Test das die verschiedenen Softwarekomponenten zusammen arbeiten und sich integrieren lassen.
3. End-2-End Test  -> Test der kompletten Lösung

Unit-Tests
-----------
Die Form der Test muss ein wenig an TF angepasst werden, da viele Funktionalitäten von TF Modulen sich nicht testen lassen (da sie verschiedenen API Calls durchführen). 
Man verlässt sich schon auf die grundlegende Funktionalität eines Modules. Dennoch versucht man, die eigentliche Funktionalität (z.B. Aufbau eines Servers) zu testen. 

Folgendes Vorgehen wird empfohlen: 

1. Erzeuge ein generisches, standalone Module
2. Erzeuge ein einfaches Beispiel im example - Ordner
3. run *terraform apply* um das in der Cloud zu deployen
4. Validiere was dort deployed wurde
5. run *terraform destroy* um das Objekt wieder zu löschen

Beispiel mit Terratest:

1. sudo apt-get install golang-go golang-go.tools
2. Setze Environmentvariablen

    .. code-block:: shell
        
        # Terratest
        export GOPATH=$HOME/go
        export PATH=$PATH:$GOPATH/bin

3. erzeuge ein Verzeichnis im $GOPATH für die Testscripte ($GOPATH/src/tf-up-and-running)
4. erzeuge ein Sanity Checklist
   
    .. code-block:: go

        package test

        import (
            "fmt"
            "testing"
        )

        func TestGoIsWorking(t *testing.T) {
            fmt.Println()
            fmt.Println("Its working")
            fmt.Println()
        }

5. im Verzeichnis 
   
    .. code-block:: 

        go mod init
        go mod tidy
        go test -v


