uSD Card mit dd unter Win7 mit Cygwin kopieren
=================================================

Idee
-----

Ein uSD Card unter Win7 zur Analyse in ein Image kopieren, dazu dd und Cygwin 
verwenden.

Grund: die uSD Card soll analysiert werden, denn es gingen "einfach Daten verloren".

Vorbereitung
----------------

siehe:

* https://superuser.com/questions/580968/how-to-use-cygwin-to-copy-an-image-from-an-sd-memory-card

Die uSD Card ist in "H:\" eingehängt.


Schritte
---------

1. Cygwin als **Administrator** starten
2. Partitionen finden: **cat /proc/partitions** --> H:\ : sdc1
3. **cd** in Verzeichnis, in welchem das Image agbelegt werden soll
4. Image erstellen: **dd if=/dev/sdc1 of=usd.img**
5. Einen Kaffee holen, das dauer **lange**: 15927345152 bytes (16 GB, 15 GiB) copied, 936,778 s, 17,0 MB/s
6. Nun gehört die Datei aber dem **Administrator**, es muss noch der Besitzer werden: 

   a. aktuelle Berechtigungen erfragen: **ls -alh**
   b. Berechtigungen ändern: **chown USER usd.img**, ev. auch **chgrp ID usd.img**, ID erfragen mit **id**

reStructuredText Links
--------------------------

siehe: 

* http://docutils.sourceforge.net/docs/user/rst/quickref.html
* https://stackoverflow.com/questions/5550089/how-to-create-a-nested-list-in-restructuredtext
* https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html
