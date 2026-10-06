# TMXC OS - Hardware-Installationsanleitung

## ⚠️ Sicherheitswarnung

**WICHTIG:** Die Installation von TMXC OS auf physischer Hardware beinhaltet die Änderung der Geräte-Firmware, das Entsperren von Bootloadern und das Flashen von benutzerdefinierten Betriebssystem-Images. Diese Aktionen können:

- Die Herstellergarantie Ihres Geräts ungültig machen
- Ihr Gerät möglicherweise bricken (unbrauchbar machen), wenn sie falsch durchgeführt werden
- Ihr Gerät Sicherheitsrisiken aussetzen, wenn keine angemessenen Vorsichtsmaßnahmen getroffen werden
- Zu dauerhaftem Datenverlust führen

**Proceed at your own risk.** Das TMXC OS-Entwicklungsteam ist nicht verantwortlich für Schäden an Ihrem Gerät oder Datenverlust. Erstellen Sie vor dem Fortfahren immer eine Datensicherung.

## Voraussetzungen

### Hardware-Anforderungen
- TMXC OS-kompatibles Gerät (siehe [DEVICE_COMPATIBILITY.md](../../DEVICE_COMPATIBILITY.md))
- USB-Datenkabel (für Verbindung zum Computer)
- Computer mit Internetverbindung
- Mindestens 50% Batterieladung auf dem Zielgerät

### Software-Anforderungen
- TMXC OS-Aktivierungsschlüssel (erhalten von tmxc.os.destek@gmail.com)
- Platform-Tools (ADB/Fastboot) für Ihren Computer
- TMXC OS-Geräte-Image (.img oder .bin Datei)
- Gerätespezifisches Bootloader-Entsperrtool (variiert nach Hersteller)

## Installationsprozess

### Schritt 1: Aktivierungsschlüssel erhalten

Bevor Sie TMXC OS installieren, müssen Sie einen gültigen Aktivierungsschlüssel erhalten:

1. Senden Sie eine E-Mail an **tmxc.os.destek@gmail.com**
2. Fügen Sie Ihr Gerätemodell und die Seriennummer hinzu
3. Warten Sie auf die Zustellung des Aktivierungsschlüssels (typischerweise 24-48 Stunden)
4. Bewahren Sie Ihren Aktivierungsschlüssel sicher auf - Sie benötigen ihn beim ersten Boot

### Schritt 2: Ihren Computer vorbereiten

**Windows:**
```powershell
# Platform-tools von der Android Developer Website herunterladen
# Nach C:\platform-tools extrahieren
# Zum System PATH hinzufügen
```

**Linux:**
```bash
sudo apt-get install android-tools-adb android-tools-fastboot
```

**macOS:**
```bash
brew install android-platform-tools
```

### Schritt 3: Entwickleroptionen aktivieren

1. Gehen Sie zu **Einstellungen** → **Über das Telefon**
2. Tippen Sie 7-mal auf **Build-Nummer**, um Entwickleroptionen zu aktivieren
3. Gehen Sie zurück zu **Einstellungen** → **Entwickleroptionen**
4. Aktivieren Sie **USB-Debugging**
5. Aktivieren Sie **OEM-Entsperrung** (falls verfügbar)

### Schritt 4: Bootloader entsperren

**⚠️ WARNUNG:** Dies wird alle Daten auf Ihrem Gerät löschen.

#### Für die meisten Android-Geräte:

```bash
# Neustart zum Bootloader
adb reboot bootloader

# Bootloader entsperren (Befehl variiert nach Hersteller)
fastboot oem unlock
# ODER
fastboot flashing unlock
```

#### Gerätespezifische Anweisungen:

**Samsung-Geräte:**
- Odin-Tool herunterladen
- Samsung-Bootloader-Entsperrpaket herunterladen
- Herstellerspezifischen Entsperrprozess befolgen

**Xiaomi-Geräte:**
- Entsperrberechtigung von Mi Unlock-Website anfordern
- Mi Flash Unlock-Tool verwenden
- Anweisungen auf dem Bildschirm befolgen

**Google Pixel:**
- OEM-Entsperrung in Entwickleroptionen aktivieren
- `fastboot flashing unlock` verwenden

**OnePlus:**
- Erweiterten Neustart in Entwickleroptionen aktivieren
- `fastboot oem unlock` verwenden

### Schritt 5: Fastboot-Modus eingeben

```bash
# Aus ausgeschaltetem Zustand
# Lautstärke verringern + Ein-/Ausschalt-Taste gleichzeitig gedrückt halten
# ODER ADB verwenden
adb reboot bootloader
```

Verbindung überprüfen:
```bash
fastboot devices
```

### Schritt 6: TMXC OS-Image flashen

```bash
# TMXC OS-Image auf entsprechende Partition flashen
fastboot flash boot tmxc_os_boot.img
fastboot flash system tmxc_os_system.img
fastboot flash vendor tmxc_os_vendor.img

# ODER kombiniertes Image flashen
fastboot flash boot tmxc_os_combined.img
```

### Schritt 7: Recovery-Image flashen (Optional)

```bash
fastboot flash recovery tmxc_os_recovery.img
```

### Schritt 8: Neustart zum System

```bash
fastboot reboot
```

### Schritt 9: Erste Boot-Konfiguration

1. **TMXC OS wird booten** (erster Boot kann 5-10 Minuten dauern)
2. **Sprachauswahl** - Wählen Sie Ihre bevorzugte Sprache
3. **Netzwerkeinrichtung** - Mit Wi-Fi oder Mobilfunknetz verbinden
4. **Aktivierung** - Geben Sie Ihren Aktivierungsschlüssel ein, wenn dazu aufgefordert
5. **Geräteeinrichtung** - Erste Konfiguration abschließen

## Fehlerbehebung

### Gerät bootet nicht

**Wenn Gerät im Bootloop feststeckt:**
```bash
# Neustart zu Recovery
adb reboot recovery

# Daten/Fabrikeinstellungen zurücksetzen
# Neustart zum Bootloader
fastboot flash boot tmxc_os_boot.img
```

### Fastboot-Befehle nicht erkannt

- Stellen Sie sicher, dass Platform-Tools in Ihrem PATH sind
- Versuchen Sie, den vollständigen Pfad zur ausführbaren Fastboot-Datei zu verwenden
- Überprüfen Sie, ob das USB-Kabel ein Datenkabel und kein nur-Ladekabel ist

### Bootloader-Entsperrung fehlgeschlagen

- Überprüfen Sie, ob Ihr Gerät Bootloader-Entsperrung unterstützt
- Überprüfen Sie, ob Netzbetreiber-Einschränkungen gelten
- Wenden Sie sich bei Bedarf an den Hersteller für den Entsperrcode

### Aktivierungsschlüssel ungültig

- Überprüfen Sie, ob Sie den Schlüssel korrekt eingegeben haben
- Wenden Sie sich an tmxc.os.destek@gmail.com, wenn der Schlüssel ungültig erscheint
- Stellen Sie sicher, dass Ihr Gerätemodell mit dem registrierten übereinstimmt

## Nach der Installation

### Installation überprüfen

```bash
# TMXC OS-Version überprüfen
adb shell tmxc_version

# Systemstatus überprüfen
adb shell tmxc_status
```

### Sicherheitseinrichtung

1. Biometrische Authentifizierung aktivieren
2. Verschlüsselungs-Tresore konfigurieren
3. Geistermodus einrichten, falls gewünscht
4. Neural-Firewall-Einstellungen konfigurieren

### System aktualisieren

```bash
# Auf Updates prüfen
adb shell tmxc_update check

# Updates anwenden
adb shell tmxc_update install
```

## Wiederherstellung und Restaurierung

### Original-Firmware wiederherstellen

Wenn Sie Ihr Gerät auf Stock-Firmware wiederherstellen müssen:

1. Stock-Firmware für Ihr Gerät herunterladen
2. Neustart in Fastboot-Modus
3. Stock-Images flashen:
```bash
fastboot flash boot stock_boot.img
fastboot flash system stock_system.img
fastboot flash vendor stock_vendor.img
fastboot flash recovery stock_recovery.img
```

### Notfallwiederherstellung

Wenn Gerät gebrickt ist:
- Verwenden Sie gerätespezifische Unbrick-Tools
- Kontaktieren Sie TMXC OS-Support
- Erwägen Sie professionellen Reparaturservice

## Zusätzliche Ressourcen

- [Gerätekompatibilitätsliste](../../DEVICE_COMPATIBILITY.md)
- [Sicherheitsdokumentation](../security/activation_protocol.md)
- [Fehlerbehebungsleitfaden](troubleshooting.md)
- [Community-Forum](https://community.tmx-os.org)

## Support

Für Installationsprobleme:
- **E-Mail:** tmxc.os.destek@gmail.com
- **Dokumentation:** [docs.tmx-os.org](https://docs.tmx-os.org)
- **Community:** [community.tmx-os.org](https://community.tmx-os.org)

---

**Zuletzt aktualisiert:** 2026-07-15  
**TMXC OS-Version:** 1.0.0  
**Leitfaden-Version:** 1.0
