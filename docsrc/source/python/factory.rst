.. _factory:

#################
Factory Pattern
#################

Klassische Modell
==================

*Ziel des Patterns:* 
Trennung von Erstellung von Objekten und Verwendung von Objekten im Code, um Abhängigkeiten aufzulösen. 
Das Factory-Pattern eignet sich dafür, Objekte zu gruppieren.Beispiel bei Arjan: Eine Kombination aus 
Video/Audioexporter. Wenn alle Kombinationen erlaubt sind, dann ist ein Factory-Pattern **nicht** sinnvoll, da jede 
Kombination aus Audio/Videoexporter als Klasse abgebildet werden muss. Hier ist dann eine Kombination aus
Composition und Dependency Inversion vorzuziehen.

1. Definition einer abstrakten Klasse sowie konkrete Ableitung von Klassen die etwas kombinieren.

.. code-block:: python

    class Abstract (ABC)
        def method1()
        
        def method2()

    class KonkreteKombination1(Abstract):
        def methode1()
            return Wert1
        def methode2()
            return Wert2
    
    class KonkreteKombination2(Abstract):
        def methode1()
            return Wert3
        def methode2()
            return Wert4

    
2. Funktion zum Erstellen einer Instanz (aus Arjancodes)

.. code-block:: python

    def read_exporter() -> Abstract:
    """constructs a factory based exporter based on the user preference"""
    factories = {
        "low": KonkreteKombination1(),
        "high": KonkreteKombination2(),
        }
    # read the desired export quality
    while True:
        export_quality = input(
            "Enter desired output quality (low, high): ")
        if export_quality in factories:
            return factories[export_quality]
        print(f"Unknown output quality option: {export_quality}.")

3. Verwenden der Factory

.. code-block:: python

    video_exporter = fac.methode1()
    audio_exporter = fac.methode2()

Python-Weg
==========
Der python Weg des factory Ansatzes:

