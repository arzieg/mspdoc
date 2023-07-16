.. _template:

##########
Template 
##########

Template ersetzt Platzhalter in Texten. Typischerweise $$,$ oder ${}

..code-block:: python

    from string import Template

    class MyTemplate(Template):
        delimiter = "%"

    text = """My example text in %language
              This is the second line - start in %day Days
           """

    src = MyTemplate(text)

    result = src.substitue (
        {
           "language" : "English"
           "day" : 5
        }
    )

    print(result)

src.safe_substitue = wenn nicht alle Placeholder gefunden werden sollen (erzeugt keine Fehlermeldung)


Template file
==============

..code-block:: python

    from string import Template

    with open ("settings.yaml.template", "r") as f:
       src = Template(f.read())

    resulte = src.substitute (
        {
           "language" : "English"
    )

