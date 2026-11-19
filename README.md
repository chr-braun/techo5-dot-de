# TECHO5 Dot – Einstieg auf Deutsch (Echo Dot 2. Gen.)

Inoffizielle deutsche Anleitung: vom originalen **Echo Dot 2. Generation (2016, `biscuit`)** zum
**Home-Assistant-Sprachassistenten** mit [TECHO5 Dot](https://github.com/HuskerMinion/techo5-dot) –
ohne Amazon-Cloud, ohne Alexa.

> **Stand:** TECHO5 Dot **v1.0.73**, Home Assistant 2026.11, ESPHome 2026.5.
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
6. [Bekanntes Problem: Wake Word nach HA-Neustart](#6-bekanntes-problem-wake-word-nach-ha-neustart)
7. [Wenn etwas nicht klappt](#7-wenn-etwas-nicht-klappt)
8. [Quellen und Dank](#8-quellen-und-dank)

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
| **Windows** | [Python 3](https://www.python.org/downloads/) („Add python.exe to PATH“ anhaken), [Git for Windows](https://git-scm.com/download/win), [Platform-Tools](https://developer.android.com/tools/releases/platform-tools) entpacken und in den PATH. Statt `python3` gilt `python`. |

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
bundled“*, passt der Build nicht – **abbrechen** und die richtige Datei holen.

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
zweite Lauf installiert und wartet, bis der erste Start „healthy“ meldet.

✅ Prüfung: Der Installer endet mit „healthy“, und der Home-Assistant-Port antwortet.

### Schritt 7 – In Home Assistant hinzufügen

Siehe nächstes Kapitel.

## 4. In Home Assistant einrichten (Deutsch!)

1. **Hinzufügen:** Home Assistant findet den Dot als ESPHome-Gerät (Einstellungen → Geräte &
   Dienste). Den **Verschlüsselungsschlüssel** einfügen, den der Installer ausgegeben hat (auch in
   `backups/<Seriennummer>/home-assistant.key`).
2. **Aktionen erlauben:** Beim ESPHome-Eintrag → Konfigurieren: **„Allow the device to perform Home
   Assistant actions“** einschalten.
3. **Wake Word wählen** am Assist-Satelliten: Okay Nabu, Hey Jarvis, Hey Mycroft, Alexa und weitere.
   Die ersten vier sind die leichten.
4. **Updates** kommen danach über die Update-Karte in HA (signiert, mit automatischem Rollback).

### Sprache: `de_DE`, nicht `de`

Damit der Dot **deutsch spricht**, muss die Sprachpipeline stimmen (Einstellungen → Sprachassistenten):

- **Sprache der Pipeline:** Deutsch.
- **Text-zu-Sprache:** Piper mit einer deutschen Stimme.
- **TTS-Sprache `de_DE`** (mit Region), **nicht** die Kurzform `de`. Bei mir kam mit `de` von Piper
  ein „nicht unterstützt“-Fehler, und es wurde **nichts gesprochen**. (Das ist meine Erfahrung mit
  Piper in Home Assistant; ob die Kurzform bei jeder Konstellation scheitert, habe ich nicht geprüft.)

**Er hört, antwortet aber nicht laut?** Das liegt fast immer an der Pipeline, nicht am Gerät:

1. Einstellungen → Sprachassistenten → die Pipeline öffnen, die der Dot nutzt.
2. Bei **Text-zu-Sprache** prüfen: steht dort *Keine*, eine Engine wählen (Piper), „Aktualisieren“.
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

## 6. Bekanntes Problem: Wake Word nach HA-Neustart

Nach einem **Neustart von Home Assistant** springt die Wake-Word-Auswahl mancher Dots auf
`no_wake_word`. Das Gerät behält sein Wake Word, aber Home Assistant verliert die Zuordnung
*Wake Word → Pipeline*, und alles läuft auf der **ersten** Pipeline. Das betrifft vor allem, wenn man
pro Dot zwei Assistenten mit verschiedenen Pipelines nutzt.

Gemeldet ist das als [techo5-dot#51](https://github.com/HuskerMinion/techo5-dot/issues/51) (offen, Stand 19.11.2026).
HuskerMinion hat geantwortet: Ein Reload der ESPHome-Integration behielt bei ihm beide
Wake-Word-Slots korrekt – der vollständige HA-Neustart scheint einen anderen Pfad zu nehmen.
Er bittet um Debug-Logs: `homeassistant.components.esphome` auf Debug setzen, HA neu starten
und die Zeilen zu „satellite configuration“ für einen Dot anhängen, zusammen mit dem TECHO5-Log
(`/data/techo5-linux/techo5.log`) aus denselben Minuten.

Bis das behoben ist, setzt eine Automation die Auswahl nach dem Start zurück:
[`ha/wake-word-nach-neustart.yaml`](ha/wake-word-nach-neustart.yaml). Sie sollte irgendwann
überflüssig werden.

## 7. Wenn etwas nicht klappt

- **Vor der TECHO5-Installation:** Im XDA-Thread gibt es einen Unbrick-Abschnitt. **Nicht mit
  Bootloader-Images herumprobieren**, so werden Echos endgültig unbrauchbar.
- **Danach:** Der Dot fällt auf den vorherigen Slot und dann in eine Rettungsumgebung mit
  USB-Seriell-Konsole zurück, wenn ein Start nicht „healthy“ wird.
- **Installer meldet Prüfsummenfehler:** Der Download wird einmal wiederholt. Passt die Byte-Zahl
  nicht zur Release-Seite, ist die Verbindung abgebrochen (weniger) oder Virenscanner/Proxy haben die
  Datei verändert (gleiche Zahl).
- **Fehler melden:** In den GitHub-Issues des Projekts, mit Gerät, Schritt und Ausgabe.

## 8. Quellen und Dank

Diese Seite fasst die Arbeit anderer zusammen und ergänzt sie. Alles Wesentliche stammt von:

- **HuskerMinion** – [TECHO5](https://github.com/HuskerMinion/techo5) und
  [TECHO5 Dot](https://github.com/HuskerMinion/techo5-dot) (MIT-Lizenz)
- **R0rt1z2** – [amonet](https://github.com/R0rt1z2/amonet), der Entsperr-Exploit
- **@proffalken** – [Write-up zur Installation unter Linux](https://gist.github.com/proffalken/377ae50146affe1886dddaaacb87926b)
- Die Entwickler hinter **EchoLocal** und **EchoMuse**, auf denen TECHO5 aufbaut

Lizenz dieser Anleitung: MIT. Keine Gewähr; Nutzung auf eigenes Risiko.
