---
name: build-market-ready-desktop-app
description: Baut eine reale, installierbare und marktfähige Desktop-Anwendung (Windows/macOS/Linux) – vom Bestands-Audit über Kernfunktion, Datenhaltung, Sicherheit, Installer, Updates und Signierung bis zur belegten Freigabetabelle. PFLICHT-TRIGGER, sobald der User „Desktop-App“, „Desktop App“, „Desktop-app“, „Desktop-Anwendung“ oder „Desktop-Programm“ (bzw. desktop app / desktop application) im Zusammenhang mit bauen, entwickeln, erstellen, ausbauen, erweitern, fertigstellen, releasen oder marktreif machen erwähnt. Auch für Electron-, Tauri-, Qt-, .NET/WPF/WinUI-, SwiftUI/AppKit- oder Flutter-Desktop-Projekte, die ausgeliefert werden sollen.
argument-hint: "[Projektpfad oder kurze Beschreibung der App]"
---

# Marktreife Desktop-App bauen

Ziel: Eine **echte, installierbare Desktop-App** mit mindestens einem vollständig funktionierenden Kernablauf – kein Plan, kein UI-Mockup, keine Platzhalter, die als fertig verkauft werden.

Argumente des Aufrufs: `$ARGUMENTS`

## Grundregeln (nicht verhandelbar)

1. **Ehrlichkeit vor Vollständigkeit.** Niemals behaupten, etwas sei getestet, gebaut, signiert, notarisiert oder veröffentlicht, wenn es nicht nachweisbar tatsächlich passiert ist. Jede Aussage „bestanden“ braucht einen Beleg (Befehl + Ergebnis, Log, Testausgabe, Artefaktpfad).
2. **„Marktreif“** nur verwenden, wenn alle kritischen Punkte der Freigabetabelle belegt `bestanden` sind. Sonst: „nicht marktreif – Blocker: …“.
3. **Selbstständig arbeiten.** Nur nachfragen bei (a) wirklich entscheidenden offenen Punkten, die sich nicht aus dem Kontext ableiten lassen, (b) externen Veröffentlichungen (Store-Upload, Release-Publish, Push auf öffentliche Kanäle, Update-Feed live schalten), (c) irreversiblen Entscheidungen (Daten löschen, History umschreiben, Lizenzwahl, Bundle-ID/App-ID festlegen, kostenpflichtige Konten).
4. **Bestehende Arbeit erhalten.** Nichts Brauchbares wegwerfen oder neu schreiben, nur weil ein anderer Stack „schöner“ wäre.
5. **Keine funktionslosen Platzhalter** als fertig ausgeben. Nicht Implementiertes wird im UI deaktiviert/ausgeblendet und in der Freigabetabelle als `offen` geführt.

## Phase 0 – Bestandsaufnahme (immer zuerst)

Prüfen und kurz zusammenfassen:
- Quellcode, Architektur, Stack, Einstiegspunkte
- Dokumentation (README, CHANGELOG, docs/)
- Build-Konfiguration (package.json, Cargo.toml, tauri.conf.json, electron-builder/forge, CMake, .csproj, Xcode-Projekt, pubspec.yaml), CI-Workflows
- Tests und deren aktueller Status (tatsächlich ausführen)
- Assets (Icons in allen benötigten Größen/Formaten: .ico, .icns, .png)
- `git status`, Branch, uncommitted Changes, letzte Commits
- Lizenz des Projekts + Lizenzen der Abhängigkeiten
- Vorhandene Signier-/Notarisierungs-Setups, Update-Feeds, Secrets-Handling

Ergebnis: Liste „behalten / reparieren / ersetzen / fehlt“.

## Phase 1 – Anforderungen ableiten

Aus Kontext, Code und Gespräch ableiten (nicht abfragen, wenn ableitbar):
- Zielplattformen (Windows / macOS [Intel/ARM] / Linux [Distro, Format])
- Zielnutzer und deren Kompetenzniveau
- Kernabläufe (1 primärer, der vollständig funktionieren muss)
- Datenarten, Datenmenge, Sensibilität (personenbezogen? Geheimnisse?)
- Offlinebetrieb ja/nein, Sync-Bedarf
- Geräte-/OS-Integration (Dateisystem, Audio/MIDI, USB/Seriell, Netzwerk, Tray, Autostart, Dateiverknüpfungen, Deep Links, Benachrichtigungen)
- Vertriebsweg (Direktdownload, Microsoft Store, Mac App Store, Homebrew, winget, Flathub/Snap/AppImage, interne Verteilung)

Annahmen explizit auflisten. Nur bei echten Weichenstellungen nachfragen.

**Stack-Wahl (nur bei Neuprojekt):** Bestehenden Stack beibehalten. Neu: Standardempfehlung **Tauri 2** (klein, sicher, Rust-Backend, eingebauter signierter Updater); **Electron** wenn Node-Ökosystem/Chromium-Features zwingend; nativ (SwiftUI, WinUI/.NET) wenn nur eine Plattform und tiefe OS-Integration. Entscheidung einmal begründen, dann umsetzen.

## Phase 2 – Architektur

Klare Trennung:
- **UI** (Views/Komponenten, keine Geschäftslogik)
- **Domain/Geschäftslogik** (rein, testbar, ohne UI-/OS-Abhängigkeit)
- **Integrationen** (OS, Hardware, Netzwerk, externe APIs) hinter Interfaces
- **Persistenz** (Repository-/Storage-Schicht mit Schema-Version)
- Bei Electron/Tauri: strikte IPC-Grenze, nur explizit freigegebene Befehle (contextIsolation, kein nodeIntegration im Renderer, Tauri-Capabilities/Allowlist minimal).

## Phase 3 – Kernablauf zuerst, vollständig

Einen Kernablauf End-to-End funktionsfähig machen, bevor Breite dazukommt. Pro Ablauf:
- Ladezustände (Spinner/Skeleton/Fortschritt, abbrechbar bei langen Operationen)
- Verständliche Fehlermeldungen (was ist passiert, was kann der Nutzer tun) – keine Stacktraces im UI
- Leer-Zustände
- Tastaturbedienung (Tab-Reihenfolge, Shortcuts plattformkonform: Cmd auf macOS, Ctrl sonst, sichtbarer Fokus)
- Barrierefreiheit (Labels/ARIA bzw. native Accessibility-APIs, Kontrast, Skalierung/HiDPI, Screenreader-Grundtauglichkeit)
- Plattformkonventionen (Menüleiste macOS, Fenster-State merken, Single-Instance falls sinnvoll)

## Phase 4 – Daten, Einstellungen, Migration

- Nutzerdaten und Einstellungen **im Benutzerprofil**, nie im Installationsordner:
  - Windows: `%APPDATA%\<App>` bzw. `%LOCALAPPDATA%\<App>`
  - macOS: `~/Library/Application Support/<BundleID>`
  - Linux: `$XDG_CONFIG_HOME` / `$XDG_DATA_HOME` (Fallback `~/.config`, `~/.local/share`)
  - Plattform-API verwenden (`app.getPath('userData')`, Tauri `path` API, `QStandardPaths`, `Environment.SpecialFolder`, `FileManager`)
- Atomare Schreibvorgänge (temp-Datei + rename), Korruptionsschutz
- Schema-Version speichern; **Migrationen** vorwärts, idempotent, mit Backup vor Migration
- **Backup & Wiederherstellung**: Export/Import-Funktion für Nutzer, automatisches Backup vor Updates/Migrationen
- Deinstallation: Nutzerdaten standardmäßig behalten, Löschen nur optional und explizit

## Phase 5 – Sicherheit & Compliance

- Alle Eingaben und externen Daten validieren (Dateien, IPC, Netzwerk, Deep Links, CLI-Argumente); Pfad-Traversal, Injection, unsichere Deserialisierung verhindern
- Minimale Berechtigungen (Entitlements, Sandbox, Tauri-Capabilities, Manifest-Capabilities, keine Admin-Rechte ohne Grund)
- Web-Inhalte: CSP setzen, keine Remote-Code-Ausführung, externe Links im Systembrowser öffnen
- **Geheimnisse niemals im Quellcode** oder im Bundle; zur Laufzeit im OS-Keychain/Credential Manager/libsecret; Build-Secrets nur über CI-Secrets/Umgebungsvariablen
- Abhängigkeiten auditieren (`npm audit`, `cargo audit`, `pip-audit`, `dotnet list package --vulnerable` o. ä.)
- Lizenzen prüfen (Kompatibilität mit eigener Lizenz, GPL/AGPL-Risiken), Third-Party-Notices erzeugen und mitliefern
- Datenschutz: Telemetrie nur opt-in, dokumentiert

## Phase 6 – Build, Versionierung, Logging

- **Reproduzierbare Builds**: Lockfiles committet, Toolchain-Versionen gepinnt, ein Befehl pro Plattform (`npm run dist:win` o. ä.), CI-Workflow für alle Zielplattformen
- **Versionsschema**: SemVer, eine einzige Quelle der Wahrheit, Version in App (About-Dialog) sichtbar, CHANGELOG gepflegt
- **Logs**: strukturierte, rotierende Logdateien im Benutzerprofil, Log-Level konfigurierbar, **keine sensiblen Daten** (Tokens, Passwörter, personenbezogene Inhalte redacten); „Logs exportieren“ für Support

## Phase 7 – Pakete, Installer, Updates, Signierung

Für **jede vereinbarte Plattform** ein funktionierendes Release-Artefakt:
- Windows: MSI/NSIS/MSIX (+ ggf. winget-Manifest)
- macOS: signierte `.app` in `.dmg`/`.pkg`, Universal oder getrennt arm64/x64
- Linux: AppImage und/oder .deb/.rpm/Flatpak

Tatsächlich prüfen (wo die Umgebung es erlaubt, sonst als `offen` markieren): **Installation, erster Start, Upgrade von Vorversion (Daten bleiben erhalten, Migration läuft), Deinstallation** (sauber, Nutzerdaten-Verhalten wie dokumentiert).

**Updates:**
- Nur über HTTPS, mit **Integritäts- und Signaturprüfung** (z. B. Tauri Updater mit Ed25519-Signatur, electron-updater mit Code-Signing-Verifikation, Sparkle mit EdDSA, MSIX/App Installer)
- Kein Downgrade-Angriff möglich, Fehlerfall = alte Version bleibt lauffähig
- Update-Signierschlüssel nie im Repo

**Code-Signierung / Notarisierung:**
- Einrichten, soweit Zertifikate und Konten vorhanden sind (Windows Authenticode/EV oder Azure Trusted Signing; Apple Developer ID + Notarisierung + Stapling; Linux GPG für Repos)
- **Fehlende Zertifikate/Konten klar als Release-Blocker nennen** – nicht umgehen, nicht verschweigen. Unsignierte Builds nur als „Test-Build, nicht für Endkunden“ kennzeichnen.

## Phase 8 – Dokumentation

Erstellen/aktualisieren:
- **Bedienungsanleitung** (Kernabläufe, Shortcuts, FAQ)
- **Installationsanleitung** pro Plattform inkl. Systemanforderungen und SmartScreen/Gatekeeper-Hinweisen
- **Support-Hinweise** (Log-Speicherort, Logs exportieren, Backup/Restore, bekannte Probleme, Kontakt)
- **Release-Hinweise / Release-Prozess** (Build, Signierung, Upload, Update-Feed, Rollback)
- CHANGELOG, LICENSE, THIRD_PARTY_NOTICES

## Phase 9 – Tests

Tatsächlich ausführen und Ergebnisse festhalten:
- Unit-Tests der Geschäftslogik
- Integrations-/E2E-Tests der Kernabläufe (z. B. Playwright für Electron, WebDriver für Tauri)
- Fehlerfälle (kein Netz, volle Platte, fehlende Rechte, korrupte Datei, ungültige Eingaben)
- Persistenz (Neustart, Absturz während Schreibvorgang, Migration alt → neu)
- Installer (Install / Upgrade / Uninstall)
- Update-Pfad inkl. manipuliertem/unsigniertem Update → muss abgelehnt werden
- Sicherheit (Audit-Tools, IPC-Missbrauch, Pfad-Traversal)
- Jede zugesagte Plattform; nicht verfügbare Plattformen/Hardware explizit als `offen` mit Grund

## Phase 10 – Abschluss: Freigabetabelle (Pflicht)

Am Ende immer diese Tabelle ausgeben. Status nur `bestanden`, `offen` oder `blockiert`. Jede Zeile mit Beleg oder Grund.

| Bereich | Status | Beleg / Grund | Nächster Schritt |
|---|---|---|---|
| Funktionen (Kernabläufe) | | | |
| Plattformtests (je Plattform eine Zeile) | | | |
| Installer | | | |
| Updates | | | |
| Datenmigration | | | |
| Sicherheit | | | |
| Lizenzen | | | |
| Dokumentation | | | |

Danach:
- **Gesamturteil**: „marktreif“ nur wenn alle kritischen Zeilen `bestanden` sind. Sonst „nicht marktreif“ + Liste der Blocker.
- **Top 3 nächste Aktionen**, priorisiert nach Wirkung.
- Hinweis auf alles, was eine Bestätigung des Users erfordert (Veröffentlichung, Zertifikatskauf, Store-Konten).
