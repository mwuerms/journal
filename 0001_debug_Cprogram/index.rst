C-Programm auf BeagleBone Black debugen, crasht ohne Nachricht
===============================================================

Ausgangslage
----------------------

Auf dem BeagleBone Black läuft ein Programm, welches eine BLE-Verbindung unterhält
und empfangene Daten an einen Server weitersendet. Dieses Programm wird mit dem 
*start-stop-daemon* gestartet, läuft im Hintergrund und loggt gewisse Ereignisse im 
syslog (siehe *busybox logger*, *busybox logread*).
Zudem wird dieses Programm mit einem Skript überwacht, ob es noch läuft 
(*rc-status program_name == started*) oder gecrasht ist (*== crashed*).

Nun kann es vorkommen, dass das beschriebene Programm abstürzt (*SEGFAULT*), dann 
startet das Überwachungsskript das Program neu. Oder es kann vorkommen, dass das
Programm nicht mehr weiterläuft, keine Nachricht mehr loggt und auch nicht mehr 
auf Signale (*USRSIG*) reagiert.

Es gilt herauszufinden, warum das Programm abstürzt resp. ohne Nachricht und 
Reaktion hängen bleibt.

Die Entwicklungsumgebung sind eclipse, GCC, GLib auf einem Linux (Ubuntu, Virtualbox).

Aktueller Stand
-----------------

Die Massnahmen haben geholfen das Programm stabiler zu machen. Bisweilen stürzte
das Programm nur noch an einer definierten Stelle ab.




Entdeckungsreise
------------------

Folgende Schritte wurden ausprobiert, um herauszufinden, wo im Code das Programm
denn crasht.

stdout und stderr mitloggen
............................

Dem *start-stop-daemon* werden die Argumente *-1 /pfad/log_stdout.txt* (für *stdout*) 
und *-2 /pfad/log_stderr.txt* (für *stderr*) mitgegeben. In diesem Fall wurde 
ein USB-Stick unter */media/usb* gemountet, auf welchem diese Logs gespeichert
werden. 

Anpassung in der init-Datei

.. code-block:: bash
    
    NOW=$(date +%Y-%m-%d_%H-%M-%S)
    LOG_PATH="/media/usb"
    LOG_STDOUT_FILE="${LOG_PATH}/${NOW}_stdout.txt"
    LOG_STDERR_FILE="${LOG_PATH}/${NOW}_stderr.txt"
    
    start-stop-daemon --start -1 ${LOG_STDOUT_FILE} -2 ${LOG_STDERR_FILE} --exec ${PROGRAM_NAME} -m --pidfile ${PID_FILE} -b -- ${PARAMS}

Wo wird bei jedem neuen Start neue Logs mit der aktuellen Zeit erstellt. 

Arrays auf dem Heap statt auf dem Stack verwenden
......................................................

Es ist zwar sehr bequem Arrays mit passender Grösse zur Laufzeit auf dem Stack
zu erstellen, um Daten zwischenzuspeichern. Z.B in einer Sende-Routine.

Das Programm wurde umgestellt, so dass es nur noch Arrays mit fester Grösse verwendet.

Eingehende Daten auf gültigkeit Prüfen
........................................

Eingehende Daten können **IRGEND ETWAS** sein, statt das, was der Hersteller eines
Chips angibt. Der Datenstrom vom verwendeten BLE-Modul kann leider durcheinander 
geraten. Eine genaue Prüfung der eingehenden Daten, insbesondere einer gültigen 
Länge, ist hier sehr wichtig.


AddressSanitizer
.....................

see: https://releases.llvm.org/7.0.0/tools/clang/docs/AddressSanitizer.htm

Der AddressSanitizer analysiert zur Laufzeit das Speicherzugriffsverhalten des 
Programms, ist etwas ausserhalb des Erlaubten (Z.B. Zugriff auf nicht mehr 
existierende Variablen auf dem Heap), bricht der AddressSanitizer das Programm 
augenblicklich ab und gibt die Stelle an, an welcher der Fehlzugriff passierte.

Mit dem AddressSanitizer konnte breits ein kleiner Fehler gefunden werden.

Mit den zusätzlichen Schaltern 

.. code-block:: bash
    
    CFLAGS += -fsanitize=address
    LDFLAGS += -fsanitize=address

wird der AddressSanitizer dem Programm hinzugefügt. 
Allerdings ist der AddressSanitizer gerade nicht fürs BeagleBone Black verfügbar, 
so wird er nur in der Entwicklungsumgebung mit kompiliert.

valgrind
............

see: 

* https://de.wikipedia.org/wiki/Valgrind
* http://valgrind.org/docs/manual/manual-core.html

start: 
valgrind --leak-check=full --show-reachable=yes --track-fds=yes --trace-children=yes --log-file=/media/usb/2019-09-10_08-30-00_valgrind.txt -v PROGRAM_NAME AND PARAMS > /media/usb/2019-09-10_08-30-00_stdout.txt

Valgrind starten mit Ausgabe von Valgrind in eine Logdatei 
(--log-file=/media/usb/2019-09-10_08-30-00_valgrind.txt) und stdout in eine Logdatei
( > /media/usb/2019-09-10_08-30-00_stdout.txt)


trace
......

Es gibt zwar die Möglichkeit bei einem Seg-Fault den Stack auszugeben (*backtrace*).
Allerdings ist dieser in diesem Programm jeweils leer (== 0). So wurde ein 
*trace*-Modul geschrieben, Um den Programmablauf zu verfolgen. 


Allerdings hat dieses zur Zeit den riesigen Nachteil, dass die Ausgabe auf *stdout*
überflutet wird. Daher ist eine Weiterentwicklung angedacht.

Weiterentwicklung trace
__________________________

Es ist geplant, dass *trace* die Markierungen nicht auf *stdout* ausgibt sondern
in einem statischen FIFO (z.B. für 100 Einträge, zu je 200 Zeichen (char)) ablegt
und diesen bei Bedarf (*SIGSEGV*, *SIGUSR*) ausgibt. Dadurch wird *stdout* nicht
mehr überflutet.



RST/Sphinx
-------------

see:

* http://docutils.sourceforge.net/docs/user/rst/quickref.html
* https://www.sphinx-doc.org/en/1.8/usage/restructuredtext/directives.html?highlight=code#showing-code-examples

