# Method - HackMyVM (Easy)

![Method.png](Method.png)

## Übersicht

*   **VM:** Method
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Method)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 2023-04-17
*   **Original-Writeup:** https://alientec1908.github.io/Method_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Method" zu erlangen. Der initiale Zugriff erfolgte durch Ausnutzung einer Command Injection-Schwachstelle in einem PHP-Skript (`secret.php`) auf dem Webserver. Dieses Skript nahm einen POST-Parameter (`HackMyVM`) entgegen und führte dessen Wert unsicher aus. Dies ermöglichte das Auslesen von Dateien, einschließlich des Quellcodes von `secret.php` selbst, der hardcodierte SSH-Credentials für den Benutzer `prakasaka` enthielt. Die finale Rechteausweitung zu Root gelang durch Ausnutzung einer unsicheren `sudo`-Regel, die `prakasaka` erlaubte, den Befehl `/bin/ip` als `root` auszuführen. Dies wurde mittels der `ip netns exec`-Technik missbraucht, um eine Root-Shell zu erhalten.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `curl`
*   `vi` (oder Texteditor)
*   `wfuzz`
*   `searchsploit`
*   `ssh`
*   `sudo`
*   `ip`
*   Standard Linux-Befehle (`cat`, `ls`, `grep`, `nc` (Versuch), `id`, `pwd`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Method" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Adresse des Ziels (192.168.2.127) mit `arp-scan` identifiziert. Hostname `method.hmv` in `/etc/hosts` eingetragen.
    *   `nmap`-Scan offenbarte Port 22 (SSH, OpenSSH 8.4p1) und Port 80 (HTTP, Nginx 1.18.0) mit einer Standard-Testseite.
    *   `nikto` fand keine kritischen Schwachstellen, aber einige potenziell interessante Backup-Dateien und `/sitemap.xml` (die sich als nutzlos erwies).
    *   Die Suche nach `secret.php` (durch nicht gezeigte Enumerationsschritte entdeckt) mit `curl -I` zeigte einen 302 Redirect, was die Existenz bestätigte.

2.  **Initial Access (Command Injection & SSH als `prakasaka`):**
    *   `curl http://method.hmv/secret.php?HackMyVM=id` gab die Antwort "Try other method", was auf einen korrekten Parameter, aber falsche HTTP-Methode hindeutete.
    *   Eine POST-Anfrage mit `curl -X POST 'http://method.hmv/secret.php' -d 'HackMyVM=id'` führte erfolgreich den `id`-Befehl als `www-data` aus.
    *   Über diese Command Injection-Schwachstelle wurde der Inhalt von `/etc/passwd` ausgelesen (Benutzer `prakasaka` identifiziert) und anschließend der Quellcode von `/var/www/html/secret.php`.
    *   Im Quellcode von `secret.php` wurden hardcodierte Credentials gefunden: `prakasaka:th3-!llum!n@t0r`.
    *   Ein SSH-Login als `prakasaka` mit dem gefundenen Passwort war erfolgreich.
    *   Die User-Flag (`e4408105ca9c2a5c2714a818c475d06F`) wurde in `/home/prakasaka/uSeR.txt` gefunden.

3.  **Privilege Escalation (von `prakasaka` zu `root` via `sudo ip`):**
    *   `sudo -l` als `prakasaka` zeigte zwei Regeln: `(!root) NOPASSWD: /bin/bash` und `(root) /bin/ip`.
    *   Die Regel `(root) /bin/ip` (benötigte das Passwort von `prakasaka`, das bekannt war) wurde ausgenutzt.
    *   Mittels der GTFOBins-Technik für `sudo ip` wurde eine Root-Shell erlangt:
        1.  `sudo ip netns add foo` (Passwort `th3-!llum!n@t0r` eingegeben)
        2.  `sudo ip netns exec foo /bin/bash`
    *   Die erhaltene Shell lief mit Root-Rechten (`uid=0(root)`).
    *   Die Root-Flag (`fc9c6eb6265921315e7c70aebd22af7F`) wurde in `/root/rOot.txt` gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Command Injection:** Ein PHP-Skript (`secret.php`) führte Benutzereingaben aus einem POST-Parameter (`HackMyVM`) unsicher als Betriebssystembefehle aus.
*   **Hardcodierte Credentials:** Anmeldeinformationen für den Benutzer `prakasaka` waren im Quellcode von `secret.php` hinterlegt.
*   **Unsichere `sudo`-Regel (ip):** Der Benutzer `prakasaka` durfte den Befehl `/bin/ip` als `root` ausführen. Dies wurde mittels der `ip netns exec`-Technik missbraucht, um eine Root-Shell zu erhalten.
*   **Ungewöhnliche `sudo`-Regel (!root):** Die Regel `(!root) NOPASSWD: /bin/bash` erlaubte die Ausführung von Bash als jeder andere Benutzer außer Root, was für die direkte Root-Eskalation nicht nützlich war.

## Flags

*   **User Flag (`/home/prakasaka/uSeR.txt`):** `e4408105ca9c2a5c2714a818c475d06F`
*   **Root Flag (`/root/rOot.txt`):** `fc9c6eb6265921315e7c70aebd22af7F`

## Tags

`HackMyVM`, `Method`, `Easy`, `Command Injection`, `Hardcoded Credentials`, `sudo Exploit`, `ip netns exec`, `GTFOBins`, `Linux`, `Web`, `Privilege Escalation`, `Nginx`
