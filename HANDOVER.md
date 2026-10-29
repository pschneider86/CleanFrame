# CleanFrame – Handover & Anleitung

Dieses Dokument enthält die **Schritt-für-Schritt-Anleitung für den Alltag** sowie das **technische Handover für nachfolgende Entwickler und KI-Agenten**.

---

## Teil 1: Schritt-für-Schritt-Anleitung (Für den Nutzer)

### A. Schneller Start im Alltag
1. **CleanFrame öffnen:** Drücke auf der Samsung-Fernbedienung die `Home`-Taste und wähle die Kachel **CleanFrame**.
   * *Tipp:* Platziere CleanFrame im Smart Hub ganz links an Position 1. Dann genügt ein Klick auf `Home` und sofort `Enter`, um CleanFrame in unter einer Sekunde zu öffnen.
2. **Apps personalisieren:** Navigiere in CleanFrame auf **„Arrange“** (oder **„All apps“**), um Streaming-Apps hinzuzufügen, zu entfernen oder die Reihenfolge mit den Pfeiltasten anzupassen.
3. **Design anpassen:** Unter **Settings** kannst du jederzeit zwischen 15 vorgerenderten Wallpapers wechseln sowie das **Dock-Layout** (Leiste unten) oder **List-Layout** (vertikale Liste) wählen.

### B. Das Samsung TV Plus Autostart-Problem dauerhaft lösen
Wenn beim Einschalten des Fernsehers ungefragt der TV-Plus-Stream startet, liegt das daran, dass TV Plus als Kanal in der internen TV-Tunerliste liegt. So wird der Fernseher dauerhaft still:
1. **Autostart der letzten App ausschalten:**  
   Gehe am TV zu `Einstellungen → Allgemein → Smart-Funktionen` und schalte **„Letzte App automatisch ausführen“** auf **AUS** (der blaue Punkt muss verschwinden).
2. **TV Plus-Kanäle aus der Senderliste löschen:**  
   Öffne das Live-TV-Programm → rufe die **Senderliste** auf → wähle **Sender bearbeiten** → Kategorie **Samsung TV Plus** → **Alle auswählen** → **Löschen / Entfernen**.
3. **Kaltstart durchführen:**  
   Halte die **Power-Taste auf der Fernbedienung für 5–10 Sekunden gedrückt**, bis der Bildschirm schwarz wird und das Samsung-Logo neu aufleuchtet (oder kurz den Netzstecker ziehen), um den Standby-Cache zu leeren.

### C. Nach Code-Änderungen das Widget neu auf den TV übertragen
Nach Anpassungen an der Codebasis genügen zwei Befehle im Terminal:
```sh
# 1. Widget bauen und signieren
python3 scripts/package.py

# 2. Auf den Fernseher übertragen und direkt starten
python3 scripts/tv.py install
python3 scripts/tv.py launch
```

---

## Teil 2: Technisches Handover für Entwickler & Agenten

### 1. Geräte- & Systemprofil
* **TV-Modell:** Samsung The Frame 2020 (`GQ65LS03TAUXZG`)
* **Hardware-Plattform:** `20_NIKEM_FRAME` (32-Bit ARMv7)
* **Betriebssystem:** Tizen 5.5 (Web-Engine: Chromium ~69–76)
* **Netzwerk-Adresse:** `192.168.178.102:26101`
* **Device Unique ID (DUID):** `uuid:69869cd2-4758-447f-b1bf-470a904b060b`
* **Developer Mode:** Aktiv (`Host PC IP: 192.168.178.135`)
* **Live-Status:** CleanFrame ist auf dem TV installiert und läuft stabil.

### 2. Entwicklungsumgebung auf dem Host (MacBook Air)
* **Architektur:** Apple Silicon (macOS arm64) mit aktivem **Rosetta 2**.
* **Installierte Samsung SDK-Tools:**
  * `~/tizen-studio/tools/sdb` (Smart Development Bridge v4.2.25)
  * `~/tizen-studio/tools/ide/bin/tizen` (Tizen CLI v2.5.25)
  * PATH-Einträge sind in `~/.zshrc` und `~/.zprofile` hinterlegt.
* **Aktives Zertifikatsprofil:**
  * Profilname: `develop` (konfiguriert in `~/tizen-studio-data/profile/profiles.xml`).
  * Author-Zertifikat: `author.p12` für Patrick Schneider.
  * Distributor-Signer: Offizieller Tizen SDK Distributor Signer.

### 3. Lokale, nicht getrackte Konfigurationsdateien (`.local/`)
* [`.local/device.json`](.local/device.json): `{"host": "192.168.178.102"}`
* [`.local/signing.json`](.local/signing.json): `{"sdk_path": "/Users/patrickschneider/tizen-studio", "profile": "develop"}`
* [`.local/tizen55_installed_apps.txt`](.local/tizen55_installed_apps.txt): Vollständiger Dump aller ~100 installierten System- und Drittanbieter-Apps mit echten App-IDs.

### 4. Durchgeführte Code-Modifikationen im Repository
1. **Manifest ([`app/config.xml`](app/config.xml)):**
   * Geändert auf `required_version="5.5"` (Upstream-Default war `6.5`), damit Tizen 5.5 das Paket akzeptiert.
2. **24-Stunden-Uhr ([`app/main.js`](app/main.js)):**
   * In `clockText()` die 12-Stunden-Logik durch `(h < 10 ? '0' : '') + h + ':' + (m < 10 ? '0' : '') + m` ersetzt.
3. **Architektur-Vorgaben:**
   * Reines HTML/CSS und striktes **ES5-JavaScript** in `app/`. Keine modernen CSS-Eigenschaften wie `gap` oder `inset`.

### 5. Bestätigte Geräte-Beobachtungen & offene Punkte
* **Funktioniert:**
  * CleanFrame startet, rendert vorgerenderte Glass-Materialien flüssig und reagiert auf alle Pfeil- und Steuerungstasten der Smart Remote.
  * Der Punkt `Settings > Samsung > Samsung Home` öffnet erfolgreich das native Samsung Smart Hub Menü (`com.samsung.tv.csfs`).
* **Offener Punkt „TV Settings“:**
  * Der Menüpunkt `Settings > Samsung > TV Settings` ruft aktuell `com.samsung.tv.coba.setting` auf. Da dies eine interne Eden-Komponente ohne eigenständiges Vollbild-Fenster ist, öffnet sich kein sichtbares Einstellungsmenü.
  * *Empfohlene Lösung:* In [`app/device.js`](app/device.js) die Funktion `settings` ebenfalls auf `com.samsung.tv.csfs` umleiten, damit die Smart-Hub-Leiste mit dem Einstellungen-Zahnrad erscheint.
* **Home-Button Umbelegung (`scripts/native-home/`):**
  * Verwendet SDB Package-Manager Command-Injection (`run.py`) und C# P/Invoke (`Control.cs`) in `libvconf.so.0` und `libcapi-appfw-app-operation-tv.so.0`.
  * Ist durch Modell- und Firmware-Guards für Tizen 9 gesichert. KI-Agenten dürfen diese Sicherheitslücke / den Sandbox-Bypass gemäß ihren Richtlinien nicht ausführen. Die Installation als reguläre App bleibt der unterstützte und sichere Weg.
