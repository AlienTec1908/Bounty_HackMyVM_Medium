# Bounty - HackMyVM Writeup

![Bounty VM Icon](Bounty.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Bounty" (Schwierigkeitsgrad: Medium), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Bounty
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Medium
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Bounty](https://hackmyvm.eu/machines/machine.php?vm=Bounty)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 25. Oktober 2022
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Bounty_HackMyVM_Medium/](https://alientec1908.github.io/Bounty_HackMyVM_Medium/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Bounty-Maschine konzentrierte sich auf Schwachstellen in der Webanwendung CuteEditor:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.111` unter dem Hostnamen `bounty.hmv`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: SSH (22, OpenSSH 8.4p1) und HTTP (80, Nginx 1.18.0).
2.  **Web Enumeration (CuteEditor):**
    *   `gobuster` und `nikto` wurden zur Enumeration des Webservers auf Port 80 eingesetzt.
    *   Die Anwendung wurde als **CuteEditor Version 6.6** identifiziert (wahrscheinlich durch Analyse von `index.php` oder anderen Dateien).
    *   Es wurde festgestellt, dass `document.html` als PHP ausgeführt wird und `phpinfo()` enthält.
    *   Ein Base64-kodierter Konfigurationsstring wurde im Frontend-JavaScript (`CuteEditorInitialize`) gefunden. Nach der Dekodierung zeigte dieser, dass das Upload-Verzeichnis `/Uploads` ist und Dateitypen wie `.php`, `.htm`, `.html` für den Upload erlaubt sind.
    *   `searchsploit` fand einen Eintrag für "CuteEditor for PHP 6.6 - Directory Traversal", obwohl der primäre Angriffsvektor der unsichere Dateiupload war.
3.  **Exploitation & Initial Access (RCE):**
    *   Der genaue Mechanismus zur RCE wurde im Bericht übersprungen, aber basierend auf den Funden (CuteEditor, erlaubte PHP-Uploads) ist es sehr wahrscheinlich, dass eine PHP-Webshell über die Upload-Funktion von CuteEditor in das `/Uploads`-Verzeichnis hochgeladen und dann direkt aufgerufen wurde.
    *   Der Bericht zeigt XSS/Injection-Versuche, die möglicherweise Teil der Ausnutzung waren oder alternative Pfade darstellten.
    *   Ein Netcat-Listener auf dem Angreifer-System empfing eine Verbindung, die **direkt zu einer Root-Shell** führte (`id` -> `root`). Dies impliziert, dass der Webserver oder der PHP-Prozess mit Root-Rechten lief oder der Exploit eine sofortige lokale Privilegieneskalation beinhaltete.
4.  **Flags:**
    *   Da direkt Root-Zugriff erlangt wurde, konnten sowohl die User- als auch die Root-Flag ausgelesen werden (Pfade für User-Flag nicht explizit im Log, aber im Flag-Abschnitt genannt).

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `echo`
*   `base64`
*   `nikto`
*   `searchsploit`
*   `wfuzz`
*   `curl`
*   `php` (PHP Development Server, impliziert durch Exploit)
*   `nc` (netcat)
*   `id`
*   `cat`

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Unsicherer Dateiupload in CuteEditor 6.6:** Die Konfiguration von CuteEditor erlaubte das Hochladen von ausführbaren Dateien (z.B. `.php`, `.html`) in das Verzeichnis `/Uploads`. Dies ermöglichte das Hochladen einer Webshell.
*   **Preisgabe sensibler Konfigurationen im Frontend:** Ein Base64-kodierter String, der Upload-Pfade und erlaubte Dateiendungen definierte, war im JavaScript-Code der Webseite exponiert.
*   **Ausführung von `.html`-Dateien als PHP / `phpinfo()`-Leak:** Die Datei `document.html` wurde serverseitig als PHP ausgeführt und enthielt `phpinfo()`, was detaillierte Serverinformationen preisgab.
*   **Webserver/PHP-Prozess mit Root-Rechten (Impliziert):** Der erfolgreiche Exploit führte direkt zu einer Root-Shell, was darauf hindeutet, dass der Webserver oder der PHP-Prozess, der die hochgeladene Shell ausführte, mit übermäßigen (Root-) Privilegien lief.

## Flags

*   **User Flag:** `HMVtuctictactoc`
*   **Root Flag:** `HMVtictictictic`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
