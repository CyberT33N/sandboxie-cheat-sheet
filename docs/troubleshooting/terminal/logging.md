# How to log .exe in terminal

Generell ist das Problem: Wenn ich z. B. eine Applikation habe, die ich sonst im Terminal starten kann und dabei direkt die Logs sehe, funktioniert das in einer Sandbox nicht mehr wie gewohnt. Sobald die Exe in einer Sandbox läuft und das Terminal-Fenster auf dem Host-System nicht in derselben Sandbox ist, werden die Logs nicht mehr normal im Terminal ausgegeben.

Daraus ergeben sich im Prinzip nur zwei Möglichkeiten:
- Ich starte ein Terminal direkt in der gleichen Sandbox, in der auch die Applikation läuft, und führe sie dort aus, um die Logs im Terminal zu sehen.
- Ich wechsle auf ein file-basiertes (oder anderes) Logging-System und lese die Logs darüber aus.
