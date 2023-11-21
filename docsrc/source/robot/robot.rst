.. _robot_allg:

###############
Robot Allgemein
###############

CMD
=====

Nach einem abgebrochenen Test bei dem vorhandenen Test neu aufsetzen. Alle Tests befinden sich in einem Verzeichnis

.. code-block:: shell

    robot --exitonfailure --outputdir ~/x86-test/robot/logs/xhana2/ScaleOut/hadr/20231120-174617 
                        --rerunfailed ~/x86-test/robot/logs/xhana2/ScaleOut/hadr/20231120-174617/output.xml 
                        --output ~/x86-test/robot/logs/xhana2/ScaleOut/hadr/20231120-174617/output_2.xml .

