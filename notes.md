# Inhaltsverzeichnis: Netzwerkarchitektur und Datenkommunikation

## 1. Einleitung
### 1.1 Motivation und Ausgangslage
> **Kurznotiz:** Warum baut ihr das? Datenerfassung im Fahrzeug, Live-Überwachung von Sensorwerten, Aufbau eines unabhängigen Netzwerks.
### 1.2 Zielsetzung des Projekts
> **Kurznotiz:** Was soll am Ende stehen? Ein stabiler Datenfluss vom Sensor bis zur Visualisierung, ein autarkes lokales WLAN und die externe Erreichbarkeit des Dashboards.
### 1.3 Abgrenzung und Aufgabenteilung
> **Kurznotiz:** Hier klärst du den Scope. Dein Partner macht die Datenbankstruktur, das Dashboard und die Sicherheit. Du fokussierst dich auf die Transportwege: WLAN, OSI-Modell, Docker-Routing und die Tunnel-Anbindung nach außen.

## 2. Theoretische Grundlagen
### 2.1 Das OSI-Referenzmodell
> **Kurznotiz:** Kurze Erklärung der 7 Schichten. Wichtig für euch: Schicht 3 (IP), Schicht 4 (TCP) und Schicht 7 (HTTP/MQTT). Das bildet den Rahmen für alles Folgende.
### 2.2 Das Transmission Control Protocol (TCP)
> **Kurznotiz:** Warum nutzt ihr TCP? Erklärung der verbindungsorientierten Übertragung (Three-Way-Handshake, Zuverlässigkeit). Das ist wichtig, um später bei der Evaluation den "Connection reset by peer"-Fehler erklären zu können!
### 2.3 Anwendungsprotokolle: MQTT und HTTP
> **Kurznotiz:** Der Vergleich! 
> - **MQTT:** Leichtgewichtig, Publish/Subscribe, perfekt für den ESP32 und instabiles WLAN.
> - **HTTP:** Request/Response-Modell, schwergewichtiger. Wird intern von Telegraf (um Daten an InfluxDB zu senden) und von Grafana (für das Web-Dashboard) genutzt.
### 2.4 Hardware-Plattformen: ESP32 und Raspberry Pi (und Sensor)
> **Kurznotiz:** Kurze Vorstellung der Hardware. Beim Raspberry Pi **unbedingt erwähnen**, dass das neueste Betriebssystem ("Bookworm") den `NetworkManager` nutzt, wodurch der Pi nativ und ohne Zusatzsoftware als Access Point (Hotspot) fungieren kann.
### 2.5 Container-Netzwerke (Docker Bridge)
> **Kurznotiz:** Wie kommunizieren Docker-Container? Erklärung des virtuellen Switches (`docker0`), der ein komplett eigenes Subnetz aufspannt, getrennt vom physischen WLAN.
### 2.6 Tunneling und Reverse Proxies (Cloudflare Tunnel)
> **Kurznotiz:** Die Theorie, wie man ein lokales Netzwerk sicher ins Internet bringt. Erklärung, dass der Tunnel von *innen* nach *außen* aufgebaut wird, wodurch man NAT-Firewalls am Router umgeht, ohne Ports öffnen zu müssen.

## 3. Planung und Konzeption
### 3.1 Systemarchitektur und Datenfluss
> **Kurznotiz:** Der Weg der Daten durch die OSI-Schichten. Sensor -> TCP/MQTT (WLAN) -> Raspberry Pi -> TCP/HTTP (Docker Bridge) -> Datenbank. Später dann: Grafana -> Cloudflare Tunnel -> Endnutzer.
### 3.2 Netzwerktopologie
> **Kurznotiz:** Geplant war eine autarke Sterntopologie mit dem Raspberry Pi als zentralem Access Point für alle Geräte.

## 4. Implementierung der Netzwerkinfrastruktur
### 4.1 Lokale IoT-Datenübertragung
> **Kurznotiz:** Wie der ESP32 sich ins WLAN einwählt, die TCP-Verbindung aufbaut und die Sensordaten per MQTT an den Port 1883 des Raspberry Pis sendet.
### 4.2 Interne Systemkommunikation (Docker Routing)
> **Kurznotiz:** Die Umsetzung der Theorie aus 2.5. Grafana und Telegraf finden die InfluxDB einfach über den Container-Namen (z.B. `http://influxdb:8086`). Daten verlassen hierfür nie das virtuelle `docker0`-Netz.
### 4.3 Externe Anbindung via Cloudflare Tunnel
> **Kurznotiz:** Die praktische Umsetzung von 2.6. Installation des Cloudflared-Dienstes auf dem Pi, Verknüpfung mit der gekauften Domain und das automatische HTTPS-Zertifikat für den sicheren Browser-Zugriff.

## 5. Kritische Betrachtung und Evaluation
### 5.1 Evaluation der autarken WLAN-Infrastruktur
> **Kurznotiz:** Die Hotspot-Story aus der Praxis! Pi wurde per `NetworkManager` zum Access Point gemacht. Problem: Sende-Modus zieht zu viel Strom -> Undervolting -> Hardware stürzt ab. Fallback auf den iPhone-Hotspot.
### 5.2 Analyse von TCP-Verbindungsabbruchen
> **Kurznotiz:** Hier kannst du super dein Wissen aus dem Theorieteil anwenden! Diagnose des `errno: 104 "Connection reset by peer"` Fehlers. Wie ihr analysiert habt, dass die TCP-Verbindung auf unterster Ebene vom Host (Pi) abgelehnt wurde, weil der Broker-Container sich in einer Neustart-Schleife befand.

## 6. Fazit und Ausblick
### 6.1 Zusammenfassung der Ergebnisse
> **Kurznotiz:** Das Netzwerk steht. Daten fließen von Sensoren über MQTT und HTTP sicher bis ins Internet, ohne Router-Ports öffnen zu müssen.
### 6.2 Ausblick
> **Kurznotiz:** Anschaffung eines stärkeren Netzteils (min. 5.1V / 3.0A), um den Raspberry Pi in Zukunft stabil als eigenen Access Point betreiben zu können.

## 7. Literaturverzeichnis

## 8. Anhang