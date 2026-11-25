# Lauf 7, 2026-11-25 (01:42 bis 04:18 CET)

**GELB**: Nichts ist kaputt, für den nächsten Lauf musst du nichts tun. Offen ist eine Entscheidung zu `guest_libcxx_exception_pointer` in #24341 (Punkt 1 unter Braucht Roman). #26812 wartet nur noch auf seinen `windows`-Job.

## Ergebnis

Der Wächterlauf von heute lief, manuell um 01:43 gestartet, weil der Termin um 03:30 erst nach der Hälfte des Budgets gekommen wäre. Er meldete fünf neue echte CPU-Fehler. Vier davon waren ein vorübergehender Zustand des Macs zwischen etwa 02:13 und 02:29: seriell per ctest bestanden sie danach 20 von 20 Läufen. Der fünfte, `nkwebkit_inspectorcontentsearchregex`, ist echt und hat mit dem Maintainer-Revert #26855 schon einen Besitzer. #24341 schließt jetzt 16 Tests mit offenem Besitzer-PR aus. #26806 ist seit 04:12 ready. Bei #26812 hängt nur noch der `windows`-Job in der vollen GitHub-Queue, deshalb bleibt er Draft.

## Wächter

- Lauf: `nightly/2026-11-25`, main `9ca3cec822` plus #17425, #11049 und #26739. Build ok bis auf den bekannten Fehler `PipelinedFlushReadOrder.cpp` (#25331). Der Lauf endete um 03:06 vollständig mit `summary.md` (Build 610 s).
- CPU: 4536 Tests, 25 parallel rot. Eingeordnet: 21 echt, 2 Last-Flakes (`guest_fios2`, `guest_posix_spawn`), 1 intermittierend (`guest_icu_comparisons`). GPU: 2042 Tests seriell, 35 rot. Neu ist nur `agc_pipeline_journal_replay_priority`; allein mit Vulkan-Env bestand er `until-fail:20` mit 20/20, ist also ein Reihenfolge-Flake. `agc_driver_cross_workgroup_flag` und `agc_driver_image_table` sind nicht mehr rot
- Gegenüber 2026-11-24-2145 neu als echt gemeldet: `font_face_lifetime` (Timeout 30 s), `guest_audio_in_status_reroute_shared_bounds` (SIGILL nach 90 s), `guest_avplayer` (Video 163 ms hinter Audio, `GuestAvPlayer.cpp:1558`), `guest_kqueue` (`GuestKqueue.cpp:1895`), `nkwebkit_inspectorcontentsearchregex` (Zeile 283).
- Nachmessung seriell per ctest, 02:35 bis 02:39: `font_face_lifetime` 20/20, `guest_avplayer` 20/20, `guest_kqueue` 20/20, der Audio-Test 1/1 in 2 s. Der Audio-Test stürzte um 02:27 auch im alten Build (main `5f04682cef`) ab, kommt also nicht aus den 107 neuen Commits. Drei Codex-Terra-Diagnosen fanden keinen verursachenden Commit und konnten in ihrer Sandbox keinen Fehler auslösen (Font 58/58, Audio 101/101). `guest_kqueue` lief in den Wächter-Reruns um 02:29 parallel zu diesen Terra-Jobs, das war meine Last. Für die drei anderen kenne ich die Ursache nicht. In der Geräteliste steht ein Continuity-Mikrofon eines iPhones; ob es das Fenster erklärt, ist ungeprüft.
- Neu und echt bleibt nur `nkwebkit_inspectorcontentsearchregex`: #25170 hat den Test eingeführt, er scheitert auch auf Linux, und oneandonlydean hat dafür den Revert #26855 geöffnet. Nichts angefasst.
- nightly.conf stimmt: keiner der drei Merge-PRs ist gemergt. Goal-Lock `goal-run-7` (bis 04:50) deckt den Termin 03:30 ab. Probe per `kickstart` nach dem Lauf: "goal run active until 2026-11-25 04:50, exiting".

## Erledigt

- [#24341](https://github.com/boykopovar/AnyPS5/pull/24341) Head [`b789a9a38e`](https://github.com/boykopovar/AnyPS5/pull/24341#issuecomment-7149845836): fünf Ausschlüsse ergänzt, je mit offenem Besitzer-PR (#26729, #26806, #25134, #26783, #26855). `ctest -N` auf dem Nightly-Build: 6578 Tests, mit Liste 6562, alle 16 Namen exakt. Body aktualisiert, kurzer Kommentar gepostet. Umgesetzt von Codex Luna (zwei Jobs); die Sandbox durfte nicht committen, deshalb hat Opus den Commit gemacht und eine überzählige Leerzeile entfernt. Reviews: Sonnet reviewer-cross zweimal APPROVE, ponytail-review "Ship", pr-test-analyzer mit 8 Punkten (zwei widerlegt, Rest unten unter Risiken). `pr_overlap --branch`: 2497 PRs, keine Überschneidung. `check_conventions` grün.
- [#26806](https://github.com/boykopovar/AnyPS5/pull/26806) um 04:12 per `gh pr ready` auf Review gestellt: alle Checks grün (7 pass, 7 skipping, `windows` in Run 45503662133 erfolgreich), Head `91cbf540f9` unverändert seit dem APPROVE von reviewer-cross in Lauf 6, keine offenen Threads.
- [#22932](https://github.com/boykopovar/AnyPS5/pull/22932#issuecomment-7148836499): MarcZX10 fragte um 00:30Z (nicht blockierend), welchen `*at`-Fall der Test abdeckt. Antwort: Der Test nutzt `AT_FDCWD`, der direkt auf `mkdir`/`unlink` geht (`Export.cpp:1036`, `:1178`). Der Deskriptor-Fall aus dem Debt-Eintrag bleibt eine bekannte Lücke.

## #24341-Stand

- Head `b789a9a38e`, mergeable, keine offenen Threads. Die Checks des neuen Heads warten in der Queue (conventions, eligibility, labels, pr; Runs 45527612684, 45527612851, 45527612635, 45527609304). Der Job selbst läuft nur per `schedule` oder `workflow_dispatch` auf `main`, ein PR-Lauf beweist ihn also nicht.
- Blocker: #17425 (approved von MarcZX10, nur ein alter Windows-Lauf rot), #11049 (CHANGES_REQUESTED von malekaifa seit 2026-11-23, Head unverändert `978f05f442`), #26739 (ready, approved von kripotker, 20 Checks grün).
- `guest_libcxx_exception_pointer` fehlt weiter in der Liste: Es gibt nur das Issue #21538, keinen Fix-PR. Die Regel verlangt einen offenen Besitzer-PR.
- #24117: 3thoxyl4t3s hat keinen eigenen PR geöffnet. Auf meinen Hinweis von gestern kam um 19:38Z die Antwort "Noted". Der einzige PR von 3thoxyl4t3s seitdem (#26768) betrifft libSceAjm und ist geschlossen. Nichts kommentiert.

## Duplikate

- `nkwebkit_inspectorcontentsearchregex`: #26855 (Revert, oneandonlydean).
- `font_face_lifetime`, `guest_avplayer`, `guest_audio_in_status_reroute_shared_bounds`, `guest_kqueue`: kein PR, kein Issue, nicht in CLAIMS, HANDOFF oder `docs/dev/debt/`. Kein Task, weil nicht reproduzierbar.
- macOS-Full-Build-Job: Kein zweiter offener PR neben #24341.

## Braucht Roman

1. `guest_libcxx_exception_pointer` in #24341: Darf ein offenes Issue (#21538) als Besitzer zählen? Wenn ja, ist es ein Eintrag mehr. Sonst bleibt der erste Lauf des Jobs an diesem Test rot.
2. Optional: `sudo pmset repeat wakeorpoweron MTWRFSU 03:25:00`, damit der Wächter nicht auf das Aufwachen wartet.
3. Erledigt von dir seit Lauf 6: Die vier Codex-Wrapper tragen das ruflo-Flag schon.

## Risiken

- #24341 auf dem gehosteten Runner: Er hat 3 Kerne und läuft mit `--parallel 6 --timeout 120`. Die lokalen Last-Flakes `guest_fios2`, `guest_posix_spawn`, `patched_libraries` und `guest_icu_comparisons` sind dort ungeprüft und nach den Regeln nicht ausschließbar. Erst ein echter Lauf des Jobs zeigt, ob er grün startet.
- Wächter: Er wiederholt rote Tests direkt nach der parallelen Phase. Ein kurzer Störzustand wie heute zwischen 02:13 und 02:29 macht dann aus flüchtigen Fehlern "echte". Vorschlag für den nächsten Lauf: die Wiederholungen der als echt gemeldeten Tests nach der GPU-Phase noch einmal laufen lassen.

## Modellnutzung

| Worker | Jobs | Aufgaben |
|---|---|---|
| Codex Luna (low) | 2 | Ausschlüsse #24341 |
| Codex Terra (medium) | 3 | Diagnosen font, avplayer, audio |
| Codex Sol, Astra | 0 | - |
| Sonnet 5.5 | 3 | reviewer-cross #24341 (2), pr-test-analyzer #24341 |
| Haiku 5.5 | 1 | Duplikatsuche |
| Opus 5.5 | - | Orchestrierung, Briefings, Nachmessungen, PR-Texte, 2 Kommentare, Bericht |

Opus-Anteil am Code in Commits: 0 Zeilen geschrieben, 1 Leerzeile gelöscht (Luna-Ausgabe in `.github/macos-full-expected-failures.txt`).

## Zeitnutzung

- 01:42 bis 01:50: Kontext, Wächter per `kickstart`, Stand der eigenen PRs, Antwort auf #22932, Ausschlüsse für #24341 (Luna).
- 01:50 bis 02:13: Wächter baut und testet. Parallel liefen `pr_overlap` (9 min) und der erste Sonnet-Review.
- 02:13 bis 02:40: Einordnung der fünf neuen Fehler: alter Build, drei Terra-Diagnosen, `guest_avplayer` 20 Mal direkt, serielle Nachmessung per ctest.
- 02:40 bis 02:45: #24341 gepusht, Body und Kommentar, zwei Reviews, Reflect.
- 02:45 bis 03:06: GPU-Phase des Wächters, Berichtsentwurf.
- 03:06 bis 04:18: GPU-Kandidat nachgemessen, Lock-Probe, Aufräumen, Warten auf die CI von #26806 und #26812 (bis 04:12), #26806 ready, Bericht.

Rund 25 Minuten gingen in die Diagnose von Fehlern, die sich danach als flüchtig erwiesen. Die neue Regel in `~/.claude/CLAUDE.md` (Diff `reflect-claude-md.diff`) setzt die serielle Nachmessung vor jede Diagnose.

## Reflect

Vier Verschärfungen in `~/.claude/CLAUDE.md` (Diff `reflect-claude-md.diff`): Wächter-"echt" vor jeder Diagnose mit `until-fail:20` nachmessen; keine Test- oder Codex-Jobs während der Wächter-Reruns; Codex kann in einem Worktree nicht committen; Pfad der Memory-Datei. Lockerungen: keine.

## Nächste Schritte

- #26812 auf ready setzen, sobald `windows` grün ist (Job 138921246235). Das Cross-Review-APPROVE aus Lauf 6 gilt für den unveränderten Head `24543f9753`.
- #24341: Den Eintrag `nkwebkit_inspectorcontentsearchregex` streichen, sobald #26855 gemergt ist. Nach dem Merge von #17425, #11049 und #26739 die Liste neu gegen `main` prüfen.
- Wächter: Der Lauf vom 2026-11-26 vergleicht mit heute. Die vier flüchtigen Tests sollten dort fehlen. Tauchen sie wieder auf, ist es kein Zufall mehr.
- Optional am Wächter: die als echt gemeldeten Tests nach der GPU-Phase ein zweites Mal wiederholen (siehe Risiken).

AI-assisted: yes (Claude Code)
