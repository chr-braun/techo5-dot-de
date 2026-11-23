# TECHO5 Dot – Einstieg auf Deutsch (Echo Dot 2. Gen.)
Inoffizielle deutsche Anleitung: vom originalen **Echo Dot 2. Generation (2016, `biscuit`)** zum
**Home-Assistant-Sprachassistenten** mit [TECHO5 Dot](https://github.com/HuskerMinion/techo5-dot) –
ohne Amazon-Cloud, ohne Alexa.

> **Stand:** TECHO5 Dot **v1.0.78** (23.11.2026), Home Assistant 2026.11, ESPHome 2026.5.
> TECHO5 ändert sich fast wöchentlich. Maßgeblich ist immer die
> [englische Anleitung](https://github.com/HuskerMinion/techo5/blob/main/docs/getting-started.md);
> diese Seite ergänzt sie um deutsche Hinweise und die Stolperstellen, die mir beim Aufbau
> begegnet sind. Ich bin nicht Teil des Projekts.

> ⚠️ **Zuerst lesen.** Das Entsperren nutzt Bootloader-Exploits. Es kann das Gerät **unbrauchbar
> machen**, die Garantie erlischt, und alle Daten auf dem Gerät werden gelöscht. Nur an Geräten
> machen, die du notfalls verlierst. Während des Vorgangs am USB-Kabel bleiben und es **nie
> abziehen, solange geschrieben wird**.

---

## Inhalt

1. [Was du brauchst](#1-was-du-brauchst)
2. [Rechner vorbereiten](#2-rechner-vorbereiten)
3. [Der Weg in sieben Schritten](#3-der-weg-in-sieben-schritten)
4. [In Home Assistant einrichten (Deutsch!)](#4-in-home-assistant-einrichten-deutsch)
5. [Mikrofone](#5-mikrofone)
6. [Bekanntes Problem: Wake Word nach HA-Neustart oder Firmware-Update](#6-bekanntes-problem-wake-word-nach-ha-neustart-oder-firmware-update)
7. [Neue Funktionen (v1.0.74 bis v1.0.78)](#7-neue-funktionen-v1074-bis-v1078)
8. [Wenn etwas nicht klappt](#8-wenn-etwas-nicht-klappt)
9. [Quellen und Dank](#9-quellen-und-dank)

---

## 1. Was du brauchst

- **Echo Dot 2. Generation**, Modell **RS03QR**. Das Modell steht auf der Unterseite. Der Dot der
  3. Generation und die anderen Echos sind ein eigenes Kapitel im Projekt, hier geht es nur um den 2.
- **USB-Datenkabel** (Micro-USB). Ein reines Ladekabel funktioniert nicht.
- **Home Assistant** mit der ESPHome-Integration (ist eingebaut).
- Für deutsche Sprache: ein Assist-Setup mit Spracherkennung und Sprachausgabe, z. B.
  **faster-whisper** (STT) und **Piper** (TTS) als Add-ons.
- Ein Windows-, Linux- oder macOS-Rechner.

Kostenloses **XDA-Konto**: Die Entsperr-Dateien hängen als Anhang im XDA-Thread.

## 2. Rechner vorbereiten

Die Installer sind Python-3-Skripte und laufen auf allen drei Systemen gleich. Du brauchst Python 3,
die Android-Platform-Tools (`adb`, `fastboot`) und `git`.

| System | Vorbereitung |
|--------|--------------|
| **macOS** | `xcode-select --install`, dann `brew install android-platform-tools` |
| **Linux** (Ubuntu/Debian) | `sudo apt install python3 git adb fastboot`, `sudo usermod -aG dialout $USER`, neu anmelden. Falls ModemManager läuft: `sudo systemctl stop ModemManager` |
| **Windows** | [Python 3](https://www.python.org/downloads/) („Add python.exe to PATH" anhaken), [Git for Windows](https://git-scm.com/download/win), [Platform-Tools](https://developer.android.com/tools/releases/platform-tools) entpacken und in den PATH. Statt `python3` gilt `python`. |

Test: `python3 --version` und `adb version`.

### Wichtig für Mac-Nutzer: BootROM vs. Fastboot

Beim **Echo Show** braucht der Entsperr-Exploit den *BootROM*-Weg, und auf dem Mac nimmt man dafür
einen Linux-Live-USB-Stick (beim **Echo Spot** gibt es zusätzlich einen Sonderweg über `fastboot -S 1G`,
siehe die englische Anleitung). Beim **Dot 2** ist das **nicht** nötig: Der
Exploit läuft hier über den **Fastboot-Modus**, nicht über den BootROM. Das Entsperren funktioniert
deshalb direkt am Mac mit `fastbrick.sh` aus **`amonet-biscuit-v2.0.1.zip`** (dem Release-Anhang im
XDA-Thread). Der Branch `mt8163-biscuit` auf GitHub ist noch v2.0.0 und bringt **kein**
macOS-`fastboot` mit. Nimm also den ZIP-Anhang, nicht das bloße Git-Repository.

## 3. Der Weg in sieben Schritten

Die Schritte folgen dem [Write-up von @proffalken](https://gist.github.com/proffalken/377ae50146affe1886dddaaacb87926b),
der den passenden Fire-OS-Stand gefunden hat, der Boot-Schleifen durch SELinux vermeidet.

**Dateien:**
- `amonet-biscuit-v2.0.1.zip` (Anhang im
  [XDA-Thread](https://xdaforums.com/t/unlock-root-twrp-unbrick-amazon-echo-dot-2nd-gen-2016-biscuit.4761416/)).
  Darin: `fastbrick.sh` / `fastbrick.bat` / `fastbrick.ps1` und `boot-root.zip`.
- Fire-OS-Update `update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin` (im selben Thread verlinkt).
- Der Installer: `git clone https://github.com/HuskerMinion/techo5-dot`.

### Schritt 1 – Entsperren mit amonet-biscuit

1. Den Dot **nur über Micro-USB am Rechner** anschließen (nicht an einem Netzteil). Das Kabel ist
   zugleich Stromversorgung und Datenleitung.
2. **Action-Taste gedrückt halten**, während das Kabel in den Dot gesteckt wird. Der Ring leuchtet
   **grün** (Amazons Werks-Fastboot).
3. `./fastbrick.sh` (macOS/Linux) bzw. `fastbrick.bat` (Windows) starten, das gefundene Gerät
   prüfen und `YES` eingeben. Am Ende steht `Exploit most likely successful!`, und der Dot startet
   in **TWRP** (**weißer** Ring).
4. Später zurück ins TWRP: Kabel ab, **Lauter** halten, Kabel wieder anstecken, auf Weiß warten.

✅ Prüfung: `adb devices` zeigt den Dot als `recovery`.

### Schritt 2 – Fire OS 6574.1 in beide Slots

TECHO5 Dot nutzt Kernel und WLAN-Treiber, die zu genau diesem Build (NS6574, 7623) passen. Andere
Builds passen nicht. Aus dem TWRP:

```bash
adb shell twrp wipe cache
adb shell twrp wipe data
adb push update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin /sdcard/update.zip
adb shell twrp install /sdcard/update.zip
adb reboot recovery
adb shell twrp install /sdcard/update.zip
```

Zweimal installieren heißt: in **beide** A/B-Slots. Erscheint im Log *„no selinux policy
bundled"*, passt der Build nicht – **abbrechen** und die richtige Datei holen.

### Schritt 3 – Root

```bash
adb push boot-root.zip /sdcard/
adb shell twrp install /sdcard/boot-root.zip
```

### Schritt 4 – adb-Schlüssel dieses Rechners vertrauen

Der Dot hat keinen Bildschirm für die Freigabe-Abfrage, also wird der Schlüssel vorab hinterlegt:

```bash
adb push ~/.android/adbkey.pub /sdcard/adbkey.pub
adb shell "mkdir -p /data/misc/adb && cp /sdcard/adbkey.pub /data/misc/adb/adb_keys && chown 1000:2000 /data/misc/adb/adb_keys && chmod 640 /data/misc/adb/adb_keys"
adb shell restorecon -v /data/misc/adb/adb_keys
adb reboot
```

Unter Windows liegt der Schlüssel in `%USERPROFILE%\.android\adbkey.pub`. Existiert er noch nicht,
einmal `adb devices` ausführen.

### Schritt 5 – WLAN (optional)

Entweder einmal in Fire OS das WLAN über die Alexa-App einrichten (den Rest der Alexa-Einrichtung
darf man überspringen), oder dem Installer das Netz mit `--wifi "<Netzwerkname>"` mitgeben. Er fragt
dann nach dem Passwort.

✅ Prüfung: `adb devices` zeigt `device`, und `adb shell id` meldet `uid=0`.

### Schritt 6 – TECHO5 Dot installieren

```bash
git clone https://github.com/HuskerMinion/techo5-dot
cd techo5-dot
python3 tools/install-dot.py --dry-run     # prüft, sichert, lädt, baut – schreibt nichts
python3 tools/install-dot.py
```

Der Trockenlauf sichert **alle Boot-Partitionen** nach `backups/<Seriennummer>/`. **Diesen Ordner
unbedingt aufbewahren**, er ist der Weg zurück und enthält auch den Home-Assistant-Schlüssel. Der
zweite Lauf installiert und wartet, bis der erste Start „healthy" meldet.

✅ Prüfung: Der Installer endet mit „healthy", und der Home-Assistant-Port antwortet.

### Schritt 7 – In Home Assistant hinzufügen

Siehe nächstes Kapitel.

## 4. In Home Assistant einrichten (Deutsch!)

1. **Hinzufügen:** Home Assistant findet den Dot als ESPHome-Gerät (Einstellungen → Geräte &
   Dienste). Den **Verschlüsselungsschlüssel** einfügen, den der Installer ausgegeben hat (auch in
   `backups/<Seriennummer>/home-assistant.key`).
2. **Aktionen erlauben:** Beim ESPHome-Eintrag → Konfigurieren: **„Allow the device to perform Home
   Assistant actions"** einschalten.
3. **Wake Word wählen** am Assist-Satelliten: Okay Nabu, Hey Jarvis, Hey Mycroft, Alexa und weitere.
   Die ersten vier sind die leichten.
4. **Updates** kommen danach über die Update-Karte in HA (signiert, mit automatischem Rollback).

### Sprache: `de_DE`, nicht `de`

Damit der Dot **deutsch spricht**, muss die Sprachpipeline stimmen (Einstellungen → Sprachassistenten):

- **Sprache der Pipeline:** Deutsch.
- **Text-zu-Sprache:** Piper mit einer deutschen Stimme.
- **TTS-Sprache `de_DE`** (mit Region), **nicht** die Kurzform `de`. Bei mir kam mit `de` von Piper
  ein „nicht unterstützt"-Fehler, und es wurde **nichts gesprochen**. (Das ist meine Erfahrung mit
  Piper in Home Assistant; ob die Kurzform bei jeder Konstellation scheitert, habe ich nicht geprüft.)

**Er hört, antwortet aber nicht laut?** Das liegt fast immer an der Pipeline, nicht am Gerät:

1. Einstellungen → Sprachassistenten → die Pipeline öffnen, die der Dot nutzt.
2. Bei **Text-zu-Sprache** prüfen: steht dort *Keine*, eine Engine wählen (Piper), „Aktualisieren".
3. Test: Entwicklerwerkzeuge → Aktionen → `tts.speak` mit deiner TTS-Entität und der
   Media-Player-Entität des Dots.
   - Spricht er, liegt es an der Pipeline.
   - Bleibt er stumm, hilft auf der Setup-Seite des Dots **Status → Download diagnostics**.

**Findet HA den Dot nicht?** mDNS überquert keine Subnetze/VLANs; HA im Docker-Container braucht
`network_mode: host`. Alternativ von Hand hinzufügen: ESPHome → Gerät hinzufügen → IP, Port
**6053**, Schlüssel.

## 5. Mikrofone

Manche ältere Anleitungen und Forenbeiträge empfehlen, das **Center-Mikrofon** zu verwenden. Das ist
**überholt**. Der Dot mittelt inzwischen **alle sieben Mikrofone** und rechnet das Echo des eigenen
Lautsprechers heraus; das ist die Standardeinstellung (Auswahl **Mix** in Home Assistant:
*All microphones*). In den Messungen des Projekts war der Durchschnitt in jeder Aufnahme besser als
das Center-Mikrofon allein. Hintergrund: Issue
[#46](https://github.com/HuskerMinion/techo5-dot/issues/46) und PR #48, beides inzwischen behoben.

Wer noch auf *Center mic* steht und leise Aufnahmen hat, stellt auf *All microphones* zurück.

## 6. Bekanntes Problem: Wake Word nach HA-Neustart oder Firmware-Update

Es gibt zwei verwandte, aber unterschiedliche Bugs:

**Bug 1 – nach HA-Neustart** ([techo5-dot#51](https://github.com/HuskerMinion/techo5-dot/issues/51), **behoben in v1.0.74**, am 21.11.2026 geschlossen):
Nach einem Neustart von Home Assistant sprang die Aktivierungswort-Auswahl mancher Dots auf
`no_wake_word`. Das Gerät selbst behielt sein Wake Word, aber HA verlor die Zuordnung
*Wake Word → Pipeline*. HuskerMinion hat festgestellt, dass der Verlust auf diesem Weg
(Slots leer, danach schickt HA nur noch das, was seine Auswahlfelder halten) von demselben Fehler
kam wie Bug 2. Der Fix aus v1.0.74 deckt deshalb beide Fälle ab. Springt das Wake Word nach einem
HA-Neustart trotzdem wieder zurück, das Issue bitte wieder öffnen.

**Bug 2 – nach Firmware-Update** ([techo5#657](https://github.com/HuskerMinion/techo5/issues/657), **behoben in v1.0.74**):
Nach einem TECHO5-Firmware-Update standen beide Aktivierungswort-Slots auf `No wake word` bzw.
`unavailable`. Der Fix ist in v1.0.74 enthalten: Das Gerät liest seine gespeicherten Modelle und
die gewählten Wörter neu ein, sobald Home Assistant danach fragt – die Slots bleiben beim Update
erhalten. Wer noch auf einer älteren Version läuft und diesen Bug trifft, schützt sich mit der
Automation unten, bis das Update eingespielt ist.

Auf **v1.0.74 oder neuer** sollte keine Automation mehr nötig sein. Wer noch eine ältere Version
nutzt oder als Absicherung eine haben möchte: Diese Automation prüft vier Minuten nach dem
HA-Start, ob die Wörter stimmen, und setzt sie ggf. zurück:
[`ha/wake-word-nach-neustart.yaml`](ha/wake-word-nach-neustart.yaml).

> **Entity-IDs:** Bei deutschsprachiger HA-Oberfläche heißen die Selects
> `select.<name>_aktivierungswort` und `select.<name>_aktivierungswort_2`.
> Bei englischer Oberfläche `select.<name>_wake_word` / `select.<name>_wake_word_2`.
> Die Automation im Repo ist für eine deutsche Oberfläche eingerichtet und muss
> an die eigenen Gerätenamen angepasst werden.

## 7. Neue Funktionen (v1.0.74 bis v1.0.78)

TECHO5 Dot erscheint fast täglich in neuen Versionen. Hier steht, was seit v1.0.73 für den
Echo Dot 2. Gen. dazugekommen ist (Stand 23.11.2026). Vollständig und verbindlich sind die
[Release-Notizen auf GitHub](https://github.com/HuskerMinion/techo5-dot/releases) (Englisch).

> **Update einspielen:** In Home Assistant über die Update-Karte des Dots (Auto-Update ist
> standardmäßig aus). Danach kurz prüfen, ob beide Aktivierungswörter noch gesetzt sind.

### v1.0.78 (23.11.2026)

[Release dot-v1.0.78](https://github.com/HuskerMinion/techo5-dot/releases/tag/dot-v1.0.78):

- **Helligkeit:** Der Dot meldet die Raumhelligkeit an Home Assistant, sobald sie sich ändert, und
  nicht mehr nur alle fünf Minuten. Ändert sich das Licht ständig (z. B. durch einen Fernseher),
  kommt höchstens alle 15 Sekunden ein Wert ([techo5-dot#54](https://github.com/HuskerMinion/techo5-dot/issues/54)).
- **Antworten auf einem anderen Lautsprecher:** Steht **Reply delivery** auf *Early*, markierte
  eine abgebrochene Antwort die nächste fälschlich als „auf dem anderen Lautsprecher gespielt".
  Das ist behoben.
- **Stopp-Wort:** Das Log vermerkt, wenn die Antwort auf einem anderen Lautsprecher lief
  ([#709](https://github.com/HuskerMinion/techo5/issues/709), [#710](https://github.com/HuskerMinion/techo5/issues/710)).

### v1.0.77 (23.11.2026)

[Release dot-v1.0.77](https://github.com/HuskerMinion/techo5-dot/releases/tag/dot-v1.0.77):

- **Frühe Antworten auf einem anderen Lautsprecher:** Mit **Reply delivery** = *Early* beginnt die
  Wiedergabe auf dem anderen Media Player schon, während der Agent noch schreibt, statt erst
  nach der kompletten Antwort. Wird die Runde gestoppt, stoppt sie dort auch
  ([#707](https://github.com/HuskerMinion/techo5/issues/707)).
- **Nächstes Gerät antwortet:** Ein von Home Assistant bestätigtes Wake Word zählt jetzt bei der
  Entscheidung mit, welches Gerät antwortet – in beide Richtungen. Ein weiter entferntes Gerät
  antwortet dadurch nicht mehr mit ([#685](https://github.com/HuskerMinion/techo5/issues/685)).
- **Room floor** (Ruhepegel des Raums) folgt dem tatsächlichen Raum und klebt nicht mehr an der
  alten Untergrenze ([#377](https://github.com/HuskerMinion/techo5/issues/377)).
- **„Let in from Home Assistant"** funktioniert auch für einen Browser im Heimnetz mit geöffneter
  Setup-Seite, ohne dass **Remote setup** eingeschaltet sein muss.
- **„Leave Home Assistant"** vergisst jetzt auch die Sendspin-Identität und das Pairing-Token,
  das der Music Assistant des alten Besitzers bekommen hatte. Wichtig, wenn ein Dot den
  Besitzer oder die HA-Instanz wechselt.

### v1.0.76 (22.11.2026)

[Release dot-v1.0.76](https://github.com/HuskerMinion/techo5-dot/releases/tag/dot-v1.0.76):

- **Music Assistant – verschlüsseltes Sendspin:** Die Verbindung ist jetzt verschlüsselt (wie in
  der Sendspin-1.0-Spezifikation), das Pairing läuft von selbst: Home Assistant gibt Music
  Assistant das Pairing-Token des Dots. Voraussetzungen für das automatische Pairing:
  **Home Assistant 2026.11 oder neuer** und ein **API-Verschlüsselungsschlüssel** am Dot. Das Token
  steht auch auf der Setup-Seite unter *Music Assistant*, falls man manuell koppeln will.
  Ein Dot mit eingerichtetem Music Assistant wählt diesen zuerst, ein Dot hinter einem VPN spielt
  deshalb schon Sekunden nach dem Start ([#679](https://github.com/HuskerMinion/techo5/issues/679)).
  **Music Assistant 2.10** zeigt nach der ersten verschlüsselten Verbindung einen **zweiten Player**
  für den Dot – den alten, nicht verfügbaren Player einfach löschen
  ([music-assistant/support#7199](https://github.com/music-assistant/support/issues/7199)).
- **Kritische Benachrichtigungen:** `notify_critical` klingelt auch nachts und in den Ruhezeiten
  mit eigener Lautstärke, bis das Stopp-Wort oder die Aktionstaste es beendet. Solange es klingelt,
  ist der Sensor **Sounding** an; das Stopp-Wort oder **Stop alarm or timer** beendet es. Der
  einfache Gong nimmt jetzt ebenfalls eine `volume` an ([#686](https://github.com/HuskerMinion/techo5/issues/686), [#687](https://github.com/HuskerMinion/techo5/issues/687)).
- **WLAN:** Der Watchdog las bei jedem Wiederverbinden das gesamte Kernel-Log; das kostete die
  Mikrofone etwa 16 ms Audio. Er folgt dem Log jetzt fortlaufend, dadurch gehen beim
  Wiederverbinden keine Audiodaten mehr verloren ([#690](https://github.com/HuskerMinion/techo5/issues/690)).
- **Wake Word:** Fällt das Modell eines Wake Words bei Live-Audio aus, wird es von selbst wieder
  eingesetzt. Der neue Sensor **Wake word problem** nennt den Grund, wenn ein gewähltes Wake Word
  nicht zuhört (z. B. ein Modell, das der Dot ablehnt) ([#692](https://github.com/HuskerMinion/techo5/issues/692)).

### v1.0.75 (21.11.2026)

[Release dot-v1.0.75](https://github.com/HuskerMinion/techo5-dot/releases/tag/dot-v1.0.75):

- **Stimme:** Mit eingeschalteter **Microphone noise reduction** wurden die ruhigen Momente einer
  Anfrage zu reiner digitaler Stille, das Gerät hörte „Lücken" zwischen den Wörtern. Die
  Rauschminderung senkt einen ruhigen Raum jetzt um die beabsichtigten 20 dB (vorher 40) ab
  ([#662](https://github.com/HuskerMinion/techo5/issues/662)).
- **Home Assistant:** Bei einem in der Music-Assistant-Bibliothek gespeicherten Titel nennt der
  Sensor **Now playing source** den echten Dienst (YouTube Music, Spotify …) statt „Library".
  Dafür braucht das Gerät das Music-Assistant-Token, sonst steht dort „Music Assistant"
  ([#674](https://github.com/HuskerMinion/techo5/issues/674)).

### v1.0.74 (21.11.2026)

[Release dot-v1.0.74](https://github.com/HuskerMinion/techo5-dot/releases/tag/dot-v1.0.74) vom 21.11.2026 bringt mehrere nützliche Ergänzungen:

#### Wecker mit Musik

Ein Alarm kann jetzt mit einer Radio-Station oder einem Music-Assistant-Favoriten wecken, nicht nur
mit einem Ton. Die Musik startet mit der Wecker-Lautstärke; Snooze setzt sie genau dort fort, wo
sie gestoppt wurde. Spielt die Musik nicht an, greift der normale Weckton als Fallback.

Einzurichten über die Aktion `alarm_wake_with` oder (bei den Echo Shows) direkt im Alarm-Editor
unter **Wake with**. Der HA-Schalter **Wake light minutes** setzt die Standardzeit für das
Aufwachlicht.

#### Wake Words bleiben beim Update erhalten ✅

Nach einem Firmware-Update standen bisher beide Aktivierungswort-Slots manchmal auf
„No wake word" oder „unavailable", bis das Gerät neu gestartet wurde
([techo5#657](https://github.com/HuskerMinion/techo5/issues/657)). Das Gerät liest seine
Modelle und die gespeicherten Wörter jetzt neu ein, sobald Home Assistant danach fragt.
Der Bug ist damit behoben.

#### Wake Words aus dem Home-Assistant-Ordner

Wake Words, die im `custom_wake_words`-Ordner von Home Assistant liegen, werden jetzt auch gehört –
zusätzlich zu demselben Wort, das über die Setup-Seite geladen wurde.

#### Nächstes Gerät: kein doppelter Aufruf

Reagierten zwei Geräte auf dasselbe Wake Word, öffnete sich auf einem zweiten Gerät manchmal ein
zweiter Gesprächskanal. Das passiert nicht mehr.

#### NetBird: integrierter Client entfernt → Add-on

Der eingebaute NetBird-Client ist entfernt. An seiner Stelle tritt
[tioan's techo5-netbird Add-on](https://github.com/tioan/techo5-netbird), das die bestehende
Verbindung übernimmt. **Wer NetBird nutzt**, sollte das Add-on installieren, bevor oder direkt
nachdem das Update eingespielt wird. Die alten NetBird-Entitäten (Switch, Status) in Home
Assistant zeigen danach „unavailable" – diese Entitäten können dort gelöscht werden.

---

## 8. Wenn etwas nicht klappt

- **Vor der TECHO5-Installation:** Im XDA-Thread gibt es einen Unbrick-Abschnitt. **Nicht mit
  Bootloader-Images herumprobieren**, so werden Echos endgültig unbrauchbar.
- **Danach:** Der Dot fällt auf den vorherigen Slot und dann in eine Rettungsumgebung mit
  USB-Seriell-Konsole zurück, wenn ein Start nicht „healthy" wird.
- **Installer meldet Prüfsummenfehler:** Der Download wird einmal wiederholt. Passt die Byte-Zahl
  nicht zur Release-Seite, ist die Verbindung abgebrochen (weniger) oder Virenscanner/Proxy haben die
  Datei verändert (gleiche Zahl).
- **Fehler melden:** In den GitHub-Issues des Projekts, mit Gerät, Schritt und Ausgabe.

## 9. Quellen und Dank

Diese Seite fasst die Arbeit anderer zusammen und ergänzt sie. Alles Wesentliche stammt von:

- **HuskerMinion** – [TECHO5](https://github.com/HuskerMinion/techo5) und
  [TECHO5 Dot](https://github.com/HuskerMinion/techo5-dot) (MIT-Lizenz)
- **R0rt1z2** – [amonet](https://github.com/R0rt1z2/amonet), der Entsperr-Exploit
- **@proffalken** – [Write-up zur Installation unter Linux](https://gist.github.com/proffalken/377ae50146affe1886dddaaacb87926b)
- Die Entwickler hinter **EchoLocal** und **EchoMuse**, auf denen TECHO5 aufbaut

Lizenz dieser Anleitung: MIT. Keine Gewähr; Nutzung auf eigenes Risiko.
