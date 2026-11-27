Ampel: GELB. 3 von 8 Gate-PRs auf macOS gemessen und ohne Bruch, die übrigen fünf blieben aus Zeitgründen offen.

# Lauf 11 (Claude), 2026-11-26 23:45 bis 2026-11-27 ~01:50

## Ergebnis
Von den acht geplanten Gate-PRs liefen #22091, #24277 und #24415 durch; keiner bricht auf macOS einen Test, der auf der Basis grün ist. Die Prüfung von Lauf 9 fand keinen Verstoß. Phase 4 (eigene PRs, Antworten) entfiel, weil Codex noch läuft.

## Prüfung von Lauf 9
Geprüft bis 23:47 (Momentaufnahme, Codex läuft weiter).

Verstöße: keine belegt.

Befunde:
- Force-Pushes: 0 bei 15 offenen PRs von Romannachwievor seit 2026-11-25 23:00Z; 0 Commits auf upstream main von Romannachwievor.
- Kommentare: als Romannachwievor seit 23:00Z nur Review 6080902950 auf #27621 (Lauf 10, nicht Lauf 9).
- `~/.codex/AGENTS.md`: gegen das Backup nur 4 Zeilen angehängt.
- `~/.codex/config.toml`: Änderung 16:59, einziges Backup vom 10.10., daher nicht auf Lauf 9 eingrenzbar. Neue Schlüssel (hooks.state, mcp_servers, projects, env), nichts entfernt, Werte nicht geprüft.
- `~/.claude/settings.json`: seit dem Backup nur Schutzhooks ergänzt, keine Lockerung.
- Hindernis aus Lauf 9 "pre-push/pre-commit fehlen" ist behoben: Hooks seit 23:41 da, in Lauf 11 per stdin getestet (upstream main rc=1, Non-FF rc=1, FF auf Fork rc=0).
- Weitere Lauf-9-Hindernisse: Codex-Timeouts (rc=124), fehlende Cross- und Finalreviews, #24341 NO-GO, Zeitbudget überschritten, Lock-Zustand am Berichtsende nicht verifiziert.

## macOS-Gate
Basis `local/claude-gate-base` = f42464a6fe: origin/main a901acf65c plus #17425, #11049, #26739, #27621 (webrtc-Fix). Apple M1, x86_64-Build, Release, ccache. Basis-Build rc=1 nach 678 s, einziger Fehler `PipelinedFlushReadOrder.cpp:232` (bekannt, auch auf main). Basis-Tests: 4665 CPU-Tests, 18 rot, 1143 s.

Methode: PR-Head in die Basis mergen, `select_tests` wählen lassen, nur den CPU-Teil der Auswahl laufen lassen (GPU-Tests `^agc_|videoout|vulkan|shader|gpu` ausgelassen, keiner der PRs ändert AGC, Shader-Recompiler oder VideoOut), ctest -j8, Timeout 300 s. Eine volle Auswahl (CPU plus GPU) hätte je PR rund 50 min gebraucht. Rot durch den PR heißt: Basis grün und mit dem PR seriell 0/3.

| PR | Head | Auswahl / gelaufen | Ergebnis | Kommentar |
|---|---|---|---|---|
| #22091 | b4635cf2d2 | 6621 gewählt, 4529 CPU, alle gelaufen | 17 rot, alle auch Basis-rot; kein Bruch | https://github.com/boykopovar/AnyPS5/pull/22091#pullrequestreview-6082024430 |
| #24277 | 76abfe6eb8 | 1103 gewählt, 1048 CPU | 1048/1048 grün | https://github.com/boykopovar/AnyPS5/pull/24277#pullrequestreview-6082007362 |
| #24415 | c55bfa776f | 6618 gewählt, 4526 CPU | 18 rot, 16 Basis-rot, 2 flüchtig unter -j8; kein Bruch | https://github.com/boykopovar/AnyPS5/pull/24415#pullrequestreview-6082007508 |
| #25091 | - | nicht gemessen (Zeit) | - | - |
| #25255 | - | nicht gemessen (Zeit) | - | - |
| #25426 | - | ersetzt, CONFLICTING mit main | - | - |
| #25491 | - | nicht gemessen (Zeit) | - | - |
| #26646 | - | nicht gemessen (Zeit) | - | - |
| #27341 | - | nicht gemessen (Zeit) | - | - |

Zu den Einzelwerten:
- #22091: `guest_firmware_constant_returns` und `guest_posix_spawn` bestanden seriell. `guest_throw_bad_array_new_length` bricht auf der Basis ab ("Subprocess aborted") und besteht mit dem PR; mit #24415 bleibt er 0/3 rot, #24415 fasst keine Exceptions an. `exception_personality` grün. Weiter rot auf Basis und PR: `guest_exception_pointer`, `guest_libcxx_exception_pointer`, `guest_libc_internal_exception_virtuals`.
- #24415: `guest_net_oob_inline` und `guest_main_stack_move` waren auf der Basis grün und beim ersten seriellen Retry grün. `guest_libressl_ssl_lifecycle` grün.
- #24277: `game_patches`, `relinker_game_patch`, `dry_run_patches` grün, Build rc=0.

Skriptfehler und Behebung:
1. `ctest -I` mit `-E`: ctest nummeriert bei `-E` neu, die Indizes aus `select.tests` trafen andere Tests. Der erste #24277-Lauf (866 statt 1048 Tests) wurde verworfen, #24415 abgebrochen. Fix: GPU-Namen aus `select.tests` entfernen, ctest ohne `-E`; Neustart mit #24277. Zweiter Fehler dabei: Der Worktree stand auf dem Branch, der Merge schob ihn mit, jetzt feste SHA.
2. `wait_codex` hielt #24277 etwa 10 min fest, weil `pgrep -f` die eigenen Shell-Befehle traf. Jetzt `pgrep -x` auf den Binärnamen.

Weil bei #22091 46 Auswahltests im ersten Lauf fehlten, liefen sie nachträglich direkt per ctest -I gegen die PR-Binaries: 47/47 grün (mit einer Fixture).

## Discord
- Gelesen: announcements, dev, general, github-feed, linux, perf-improvement, bug-reports (plus ein Thread). Etwa 47 von 60 Aktionen. Nur lesend.
- Funde: keine Nennung der Gate-PRs, keine Frage an Roman, keine @-Erwähnung. Ein Contributor kündigt am 26.11. 20:52 in #general "den macOS-Branch" an, ohne PR (https://discord.com/channels/1556655479495589999/1556655480590442578/1575600050388738091).
- Downloads: keine.
- Aktionen: nichts geschrieben, Tab geschlossen. Nicht gelesen: faq, ideas, showcase, best-memes, website-feedback, project-updates, about, rules, verify.

## Eigene PRs und Antworten
Phase 4 übersprungen: Codex (Prozess 42595) läuft noch, Claims ab 20:49 sind active und jünger als 6 h. Die Messung für #24081 entfiel damit ebenfalls.

## Wächter-Verdacht
- Last: kein Zusammenhang belegbar. Nightly 11-26, 11-25-2341 und 11-25-2136 hatten 1, 3 und 1 flüchtige Fehler bei einer 15-min-Last von 19.85, 10.22 und nicht notiert; der Lauf mit der höchsten Last hatte die wenigsten.
- Zeitfenster: nicht ableitbar, summary.md nennt keine Zeit je Test (nur die Rerun-Runden direkt nach der CPU-Phase und rund 50 min später).

## Hindernisse
- RUN11.md enthielt nur die Ergänzung, der Haupttext kam per Chat.
- Codex-goal.lock abgelaufen, nicht erneuert (Kopie `goal.lock.codex-expired`). Eigener Lock bis 03:00, kein kickstart (RUN11 verbietet ihn).
- BTLEServer mit 95 % CPU, läuft als root, daher kein killall.
- `ctest -R` mit 6707 Namen: "Expression too big", 0 Tests. Ersetzt durch `-E` und `select_tests ctest`.
- Volle select_tests-Auswahl je PR passt für acht PRs nicht ins Budget; Umstellung auf den CPU-Teil.
- Basis-Build rc=1 durch bekannten Fehler `PipelinedFlushReadOrder.cpp`.
- #25426 CONFLICTING mit main, ersetzt durch #27341; #27341 trotzdem nicht mehr erreicht.
- `ctest -E` plus `-I`: falsche Indizes, #24277 verworfen, #24415 abgebrochen.
- Merge schob den Branch des Gate-Worktrees mit; feste SHA.
- `wait_codex` mit `pgrep -f` blockierte etwa 10 min.
- Zeitgrenze: nach #24415 keine weiteren PRs (#25091 wäre nicht vor 02:05 fertig geworden).
- `select_tests ctest` lehnte die #22091-Teilmenge ab ("CTest registration changed", Fixture fehlt); Nachtrag direkt per ctest -I.
- Phase 4 wegen laufendem Codex übersprungen, Codex-Claims nicht verwaist.

## Braucht Roman
- Codex-Prozess 42595 seit 15:15 offen, Lock abgelaufen, Claims ab 20:49 active: beenden oder Lauf 9 abschließen?
- `~/.codex/config.toml`-Änderung 16:59 nicht auf Lauf 9 eingrenzbar (Backup vom 10.10.): Diff selbst prüfen.
- Fünf Gate-PRs ungemessen: #25091, #25255, #25491, #26646, #27341.
- #25426 hat einen Konflikt mit main.
- Ein Contributor kündigt in #general einen "MacOS-Branch" an (Link oben): Absprache wegen Überschneidung.
- Die vier Basis-PRs #17425, #11049, #26739, #27621 sind weiter offen und für jeden macOS-Lauf nötig.

## Modellnutzung
Opus orchestriert. Sonnet: Lauf-9-Prüfung, Discord, Kommentarentwürfe (2 Aufrufe), Bericht. Haiku: Wächter-Auswertung. Kein Codex (Codex-Lauf aktiv).

## Zeitnutzung
| Phase | Zeit |
|---|---|
| Setup, Basis-Merges und Submodule; parallel Lauf-9-Prüfung, Discord, Wächter-Auswertung | 23:45 bis 23:53 |
| Basis-Build | 23:53 bis 00:04 |
| Basis-Tests | ca. 00:07 bis 00:26 |
| #22091 | 00:25 bis 00:52 |
| Skriptfehler, Neustart | ca. 01:00 |
| #24277 | 00:59 bis 01:14 |
| #24415 | 01:13 bis 01:36 |
| Nachtrag #22091 | 01:36 bis 01:40 |
| Kommentare, Bericht, Aufräumen | 01:37 bis ca. 01:50 |

Deadline war 02:45; der Lauf endete früher, weil kein weiterer Gate-PR vor der Abbruchgrenze 02:05 fertig geworden wäre.

## Nächste Schritte
1. Codex-Zustand klären (Prozess 42595, Claims, Lock), danach Phase 4 und die #24081-Messung nachholen.
2. Gate für #25091, #25255, #25491, #26646, #27341 fortsetzen; das korrigierte Skript (ctest ohne `-E`, `pgrep -x`) liegt bereit.
3. #25426 nach dem Konflikt mit main beobachten, Autor bei Bedarf informieren.
4. Mit dem Contributor des "MacOS-Branch" klären, wer was macht.
5. `PipelinedFlushReadOrder.cpp:232` und `guest_throw_bad_array_new_length` als bestehende Fehler auf main im Blick behalten.
