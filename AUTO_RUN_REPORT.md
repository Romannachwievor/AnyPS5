# Lauf 6, 2026-11-24 (20:25 bis 00:19 CET)

**GELB**: Für den nächsten Lauf musst du nichts tun. Offen sind nur optionale Punkte: den Agents-Vorschlag einspielen und den Mac um 03:25 wecken lassen.

## Ergebnis

Der nächtliche macOS-Wächter ist installiert und hat einen vollen Lauf mit rc 0 hinter sich. Er läuft ab heute Nacht um 03:30. Die volle macOS-Suite auf main fand einen neuen echten Fehler, `guest_socket_options` seit #25533; der Fix ist #26806. `guest_main_stack_move` schlägt je nach Messung in jedem 4. bis 20. Lauf fehl und galt bisher als Last-Flake. Es ist ein Testfehler, den #26812 behebt. #24341 trägt jetzt nur noch Ausschlüsse mit offenem Besitzer.

## macOS-CI-Stand

- #24341 (`ci(build): run macOS full build and CPU tests nightly`) ist offen. Head 7eae4b658c streicht `nkwebkit_filteroperation` (behoben durch #24771) und `guest_kernel_app_archive_mmap` (behoben durch #11625). Beide waren auf dem M1 im Selbsttest, im Vollauf und seriell 3/3 grün. Es bleiben 11 Ausschlüsse mit Besitzer. Der Sonnet-Review hat den Diff freigegeben. Die Checks des Pushs warten noch in der Queue (4 pending).
- Blocker: #17425 (approved, nur ein alter Windows-Flake `guest_appcontent` 0xc0000409), #11049 (CHANGES_REQUESTED von malekaifa wegen des SDL-Init-Race zwischen AudioIn und AudioOut) und #26739 (seit 00:19 ready, alle Checks grün, Approve von kripotker).
- Fünf Tests sind auf main mit den drei Merges rot, stehen aber nicht in der Liste: `guest_webrtc_network_manager` (#26729), `guest_socket_options` (#26806), `guest_udp` (#25134), `guest_icu_utf_conversions` (#26783) und `guest_libcxx_exception_pointer` (#21538). Ich habe sie nicht ergänzt, weil die Regeln für #24341 nur das Entfernen erlauben. Sie stehen mit ihren Besitzern im Kommentar [7146585653](https://github.com/boykopovar/AnyPS5/pull/24341#issuecomment-7146585653).
- Auf #24117 hat 3thoxyl4t3s am 2026-11-24 um 12:54Z einen eigenen PR für denselben Job angekündigt. Mein Kommentar [7142285090](https://github.com/boykopovar/AnyPS5/issues/24117#issuecomment-7142285090) verweist auf #24341 und #26739.

## Wächter

- Installiert: ja. Dazu gehören `~/Projects/anyps5-tools/aps5-macos-nightly`, `nightly.conf` (Merges 17425, 11049, 26739; GPU an; 6 CPU-Jobs) und `~/Library/LaunchAgents/com.romannachwievor.aps5-macos-nightly.plist` (03:30, `caffeinate -i`), geladen per `launchctl bootstrap gui/501`.
- Selbsttest mit Teilmenge (nightly/2026-11-24-subset, rc 0): fünf Tests liefen, die drei erwarteten waren grün, `guest_poll` und `guest_webrtc_network_manager` 0/3 und damit echt. Voller Lauf (nightly/2026-11-24-2145, main 61cce6e86b, rc 0, 5032 s): CPU 4529 Tests, 16 echte Fehler, 6 Last-Flakes; GPU 2042 Tests, 36 rot. Der Abbruchpfad ist unter launchd belegt (nightly/2026-11-24, "aborted at step worktree"), der Goal-Lock per `kickstart` ("goal run active until 2026-11-25 01:25", Exit 0).
- Reviews: Zwei Sonnet-Durchgänge mit 11 und 1 Fund, behoben über N3 bis N5 und N7. N6 lässt einen Test, der 3/3 seriell bestanden hat, noch `--repeat until-fail:20` laufen, bevor er als Last-Flake gilt. Die Wiederholungsschleife habe ich mit einem gefälschten ctest unter `/bin/bash` 3.2 geprüft. Grenze: Ein Test, der jeden 20. Lauf fehlschlägt, besteht 20 Läufe noch mit rund 36 %.
- Nächster Termin: 2026-11-25 03:30. Der Goal-Lock läuft um 01:25 ab. Schläft der Mac um 03:30, startet launchd den Lauf erst beim Aufwachen.

## Erledigt

- [#26806](https://github.com/boykopovar/AnyPS5/pull/26806) `fix(libkernel): reject 0.0.0.0/8 multicast interface addresses on every host`, Draft, head 91cbf540f9. #25533 hatte die Prüfung `ZeroNetworkAddress` nur unter `#ifdef _WIN32` eingebaut. Der macOS-Host antwortet dort `EINVAL` statt `EADDRNOTAVAIL`. Ergebnisse: main 0/3 (errno 22), Fix 3/3, beide Mutationen 0/3, Netz-Set 61/64 (die drei roten Tests sind auch auf main rot und haben Besitzer). reviewer-cross gab APPROVE. Umgesetzt von Codex Luna. Auf #25533 steht der Messkommentar [7146586244](https://github.com/boykopovar/AnyPS5/pull/25533#issuecomment-7146586244).
- [#26812](https://github.com/boykopovar/AnyPS5/pull/26812) `test(libkernel): probe the start block the main stack move keeps`, Draft, head 24543f9753 (getestet bce996ecc6). Der Emulator behält die alte Stack-Seite ab `argv - sizeof(char*)`. Der Test prüfte aber den 64-KiB-Block einer lokalen Variable in `main`, und wegen Stack-ASLR liegen die beiden manchmal in verschiedenen Blöcken. Ergebnisse: Fix 260/260 unter ctest; main 38/40 unter ctest, 39/40 direkt, und `until-fail:60` war gleich im ersten Lauf rot. Eine temporäre Messzeile zeigte, dass der Prüfzweig in 100 von 100 Läufen lief. Diagnose von Codex Terra, Fix von Codex Luna. reviewer-cross gab APPROVE, ponytail-review "Ship".
- #24341 aktualisiert (siehe oben).
- [#26783](https://github.com/boykopovar/AnyPS5/pull/26783) (cozejarr) auf dem M1 gemessen: main 0/3 mit Timeout nach 10,03 s, mit dem PR 3/3 (4,84 s, 0,22 s, 0,22 s). Kommentar [7146044398](https://github.com/boykopovar/AnyPS5/pull/26783#issuecomment-7146044398).
- Eigene PRs: #26673 und #26661 sind gemergt (oneandonlydean, 21:16Z und 21:17Z). #26729 ist seit 22:55Z ready. Gesetzt wurde das über das Konto Romannachwievor, aber nicht aus diesem Lauf (kein Eintrag im Log). [#26739](https://github.com/boykopovar/AnyPS5/pull/26739) habe ich um 00:19 per `gh pr ready` auf Review gestellt: Alle Checks waren grün (20 pass, 2 skipping, auch `windows`), kripotker hatte den Head 6d85fbf66b approved, und es gab keine offenen Threads. #26488 wartet weiter auf #21939, #26519 ruht bis 2026-12-01.

## Top 5 Backlog (N² × S × M × G)

| # | Punkt | Score | Stand |
|---|---|---|---|
| 1 | #24341: fünf rote Tests fehlen in der Ausschlussliste | 240 | nach den Regeln nicht änderbar, Besitzer im Kommentar |
| 2 | #18785 Linear-Target-Store (Performance) | 144 | braucht `trace_digest` mit Titel-Trace |
| 3 | #26623 AMD Vega, letzter Vertex liest 0 | 128 | riskanter Vorschlag, braucht AMD |
| 4 | OCSP feldweise lesen (LibreSsl3 OcspFindStatus.cpp:251-262) | 128 | Vorschlag, gurtgg arbeitet daneben |
| 5 | Forward-Slot-Prüfungen LibcxxCondition.cpp:15, VersionedFilesystemCalls.cpp:38 | 128 | nur Test-Fakes erreichen den Pfad |

Erledigt aus dem Backlog: `guest_socket_options` (900), der Wächter-Fall mit 3 Wiederholungen (500, lokal) und `guest_main_stack_move` (324).

## Abgebrochen

- Die erste CPU-Suite um 21:11 ist nach etwa einer Minute eingefroren. `pause-builds` hält jede Prozessgruppe mit einem ninja an, und ein Test hatte über cmake try_compile selbst ninja gestartet. Dadurch stand ctest 31 Minuten. Ich habe das Log verworfen und Phase 2 als vollen Wächterlauf wiederholt.
- Der erste `kickstart` des Wächters startete einen echten Lauf, weil `goal.lock` beide Schlüssel in einer Zeile hatte. Ich habe ihn mit TERM beendet, den halben Worktree entfernt und das Format korrigiert.
- P1 `guest_posix_spawn` (9 bis 10 s auf macOS): Die Zeit geht an 20 serielle Kindprozesse und 2,5 s Sleeps im Test. Der einzige Hebel berührt die Race, die #12116 abgesichert hat. Score 24, verworfen.

## Duplikate

- `guest_icu_utf_conversions`: #26783 (cozejarr).
- `nkwebkit_filteroperation`: #24771 (gemergt). `guest_kernel_app_archive_mmap`: #11625 (gemergt).
- Alle 16 echten CPU-Fehler auf main haben einen offenen Besitzer-PR: 11 stehen in der Liste von #24341, dazu #26729, #26806, #25134, #26783 und #21538.
- `guest_main_stack_move`: Die Suche nach Testname, Dateinamen und `MainThreadMacOs` fand keinen PR, der die beiden Dateien ändert. `pr_overlap --branch` fand unter 2467 offenen PRs keine Überschneidung.

## Braucht Roman (nach Wichtigkeit)

1. Agents-Vorschlag einspielen (ruflo-Flag in den `codex exec`-Zeilen der vier Codex-Wrapper). In Lauf 5 hat der Auto-Mode das abgelehnt, deshalb habe ich es nicht noch einmal versucht. Datei: `reflect-agents-proposal.md` im Run-Ordner. `sed -i '' "s/-c 'notify=\[\]' -s /-c 'notify=[]' -c 'plugins.\"ruflo-core@ruflo\".enabled=false' -s /" ~/.claude/agents/codex-{luna,terra,sol,astra}.md`
2. Optional, damit der Wächter nicht auf das Aufwachen wartet: `sudo pmset repeat wakeorpoweron MTWRFSU 03:25:00`
3. Optional, die fünf fehlenden Ausschlüsse in #24341 eintragen lassen (Liste im Kommentar 7146585653): im nächsten Prompt die Ausnahme für #24341 auf "Ergänzen mit Besitzer-PR" erweitern.
4. Falls der Wächter weg soll: `launchctl bootout gui/$(id -u)/com.romannachwievor.aps5-macos-nightly && rm ~/Library/LaunchAgents/com.romannachwievor.aps5-macos-nightly.plist`

## Riskante Vorschläge

- #26623 (AMD Vega, letzter Vertex mit Stride liest 0): Der Fix in `VertexInput.hpp:340` bzw. `Draw.cpp:2164` lässt sich ohne AMD-Karte nicht prüfen.
- `GuestCopyOut` kürzt auf macOS die Byte-Zahl auf 32 Bit (libc/src/General.cpp:992-993). Das betrifft erst Kopien ab 4 GiB und hat keinen Test.
- `guest_posix_spawn`: Ein Handshake statt des 2-s-Sleeps nach SIGCONT würde etwa 2 s sparen, berührt aber die Race aus #12116.

## Modellnutzung

| Worker | Jobs | Aufgaben |
|---|---|---|
| Codex Luna (low) | 8 | Wächter-Fixes N3 bis N7, Ausschlüsse #24341, Fix #26806, Fix #26812 |
| Codex Terra (medium) | 3 | Wächter N1, Analyse `guest_posix_spawn`, Diagnose `guest_main_stack_move` |
| Codex Sol, Astra | 0 | nur Smoke-Test |
| Sonnet 5.5 | 9 | Reviews: Wächter (3, einer davon zusammen mit #24341), #26806 (3), #26812 (3) |
| Haiku 5.5 | 2 | Bestandsaufnahme macOS-CI, Signal-Scan |
| Opus 5.5 | - | Orchestrierung, Briefings, Prüfläufe, PR-Texte, 4 Kommentare, Log, Bericht |

Opus-Anteil am Code in Commits: 0 Zeilen. Opus hat zwei temporäre Messzeilen geschrieben, die nicht im PR sind: die Zählung von `originalInArena` und das gefälschte ctest für den Wächtertest.

## Zeitnutzung

- 20:25 bis 21:11: Setup, Smoke-Tests, Wächter (Terra in 6 min, zwei Review-Runden), Build von ../anyps5-build (2571 s, rc 1 nur durch den bekannten Fehler #25331).
- 21:11 bis 21:44: eingefrorene CPU-Suite, 31 Minuten verloren.
- 21:44 bis 23:11: voller Wächterlauf als Phase 2 (5032 s), parallel die Analyse P1 und die Ausschlüsse für #24341.
- 23:11 bis 23:44: Task `guest_socket_options` bis zum PR #26806, dazu die Messung für #26783.
- 23:44 bis 00:17: Task `guest_main_stack_move` bis zum PR #26812, Wächter N6 und N7, Reflect.
- 00:17 bis 00:19: Bericht, HANDOFF, CLAIMS, Memory, Aufräumen.

## Reflect

Sechs Verschärfungen stehen jetzt in `~/.claude/CLAUDE.md` (Diff: `reflect-claude-md.diff` im Run-Ordner): Ad-hoc-grep nur nach "FAILED:", `pause-builds` nie während ctest, `check_conventions` erst nach dem Commit, `until-fail:20` vor dem Urteil "Last-Flake", grüne Läufe nur mit nachweislich gelaufenem Prüfzweig, `goal.lock` mit einem Schlüssel pro Zeile plus `kickstart`-Probe. Lockerungen gibt es keine.

## Nächste Schritte

- Morgen früh den ersten nächtlichen Lauf lesen: `~/Projects/anyps5-reports/nightly/2026-11-25/summary.md`. Er vergleicht zum ersten Mal mit dem Lauf von 21:45.
- #26806 und #26812: CI abwarten, dann ready nach denselben Regeln wie #26739.
- #24341: Sobald #17425, #11049 und #26739 gemergt sind, die Ausschlussliste neu gegen main prüfen.

AI-assisted: yes (Claude Code)
