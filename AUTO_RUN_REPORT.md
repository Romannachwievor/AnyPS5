# Lauf 8, 2026-11-25 (21:35 bis 23:43 CET)

**GELB**: Nichts ist kaputt. #24341 kann aber erst grün laufen, wenn du zwei Dinge außerhalb meiner Ausnahme entscheidest (Punkt 1 unter Braucht Roman). Die Actions im Fork laufen seit 00:08Z nicht mehr (Punkt 2). Wegen einer Panne bei der Abschluss-Probe läuft bis etwa 01:30 ein zusätzlicher Wächterlauf, siehe unten.

## Ergebnis

Der Wächter lief auf frischem main `ff5b409dda` und meldete 22 echte CPU-Fehler. Seriell nachgemessen blieben davon nur `guest_exception_pointer` und `patched_libraries` als neue echte Fehler übrig, beide ohne Besitzer. Den zweiten habe ich bis auf #26124 zurückverfolgt und dort mit Messung gemeldet. Der Wächter ist gehärtet und hat jetzt eine zweite Wiederholungsrunde nach der GPU-Phase, ein Self-Test belegt das. #24341 hat 15 Ausschlüsse. Ein echter Lauf des Jobs im Fork kam nicht zustande, weil die Actions dort keine Läufe mehr anlegen.

## Wächter

- Lauf `nightly/2026-11-25-2136`, manuell per `kickstart` um 21:36. main `ff5b409dda` plus #17425, #11049, #26739 (alle drei offen, `nightly.conf` unverändert). Der Termin 03:30 am 2026-11-25 war vom Lock aus Lauf 7 geblockt, wie geplant (`goal run active until 2026-11-25 04:50, exiting`).
- Build 2119 s, rc 1. Bekannt ist `PipelinedFlushReadOrder.cpp` (#25331). Neu ist `GuestLibcxxRegexError.cpp:45` (AppleClang: "casts away qualifiers"), Besitzer #27320 von magnub. Während der CPU-Phase lag XProtect bei etwa 58 % CPU, weil es jedes neu gebaute Test-Binary beim ersten Start prüft. Die CPU-Phase dauerte deshalb 26 statt 20 Minuten.
- CPU: 4578 Tests, 23 parallel rot, 22 echt (0/3), 1 Last-Flake (`guest_icu_comparisons`). GPU: 2057 Tests seriell, 37 rot. Neu gegenüber 2026-11-25 sind zwei, beide nach Ende des Wächters seriell mit Vulkan-Env nachgemessen. `agc_driver_cross_workgroup_flag` scheitert gleich beim ersten Lauf. Er war bis 2026-11-24-2145 rot, am 2026-11-25 grün und ist jetzt wieder rot, kippt also zwischen den Nächten. `agc_driver_buffer_atomics_or_swap64` bestand 5 Läufe und scheiterte im sechsten, ist also intermittierend. `agc_pipeline_journal_replay_priority` ist weiter rot, war in Lauf 7 allein aber 20/20 grün. Für GPU-Arbeit reichte die Zeit nicht.
- Nicht mehr rot gegenüber 2026-11-25: `nkwebkit_inspectorcontentsearchregex` (#26856 gemergt, der Revert #26855 ist geschlossen), `guest_icu_utf_conversions` (#26783), `guest_webrtc_network_manager` (#26729) und der Audio-Test aus Lauf 7.
- Seriell nachgemessen ab 23:01 mit `--repeat until-fail:20`, unter `nice` und zeitgleich mit der GPU-Phase, weil die Deadline sonst nicht gereicht hätte:
  - flüchtig, 20/20 grün: `font_face_lifetime`, `font_face_lifetime_churn`, `font_face_lifetime_replace`, `guest_avplayer`, `guest_kqueue`. Die beiden Font-Subtests sind neu. Die anderen drei wurden schon in Lauf 7 als echt gemeldet und waren seriell grün. Das ist jetzt zweimal passiert, also kein Zufall: Direkt nach der parallelen Phase scheitern sie dreimal in Folge, später nicht mehr.
  - neu und echt, ohne Besitzer: `guest_exception_pointer` (`current_exception inside catch returned wrong pointer`; der Test kam heute mit #25087) und `patched_libraries` (siehe unten).
- `patched_libraries`: `dlopen(libSceAudiodec.native.prx): Symbol not found: _av_frame_alloc`, erwartet in `libSceAjm.native.prx`. Die Ursache hat Codex Terra gefunden, ich habe sie mit `dyld_info` geprüft. #26124 (a45b29e961) linkt `libSceAjm.native` in `libSceAudiodec.native`. Der macOS-Linker bindet die FFmpeg-Importe deshalb an Ajm, und dessen gepatchte Fassung exportiert `_av_frame_alloc` nicht mehr (ungepatcht 1, gepatcht 0). Linux umgeht das mit `--exclude-libs,ALL` (`CMakeLists.txt:1152`). Messkommentar: [#26124](https://github.com/boykopovar/AnyPS5/pull/26124#issuecomment-7176014597). Einen Fix habe ich nicht versucht, die Zeit reichte nicht für einen macOS-Build mit Gegenprobe.
- Härtung von `aps5-macos-nightly`, umgesetzt von Codex Terra, Nits von Luna:
  - Runde 2 läuft nach der GPU-Phase, auch im Subset-Modus und mit `RUN_GPU=0`: Jeder Test aus Runde 1 läuft noch dreimal seriell. Echt ist nur, was in beiden Runden 0/3 grün ist. Sonst kommt er nach `cpu-transient.txt`.
  - `summary.md` hat neue Abschnitte `## cpu-transient` und `## cpu-real details`: pro Test Zeit, `load averages` und goal.lock-Stand beider Runden. Dazu kommen die Zeile `Load after parallel CPU phase` und `N transient` in der CPU-Zeile.
  - Selbsttest `~/Projects/anyps5-tools/aps5-nightly-selftest.sh`: ein Wegwerf-CMake-Projekt unter `mktemp -d`, per `sandbox-exec` von den echten Pfaden ferngehalten. Darin ein immer roter Test, ein Test rot nur in Runde 1, einer rot nur beim ersten Aufruf und ein GPU-Test. Die erste Fassung hatte einen wackligen Check: `-nt` vergleicht nur ganze Sekunden, deshalb scheiterte der erste Test gegen den installierten Pfad an "round 2 after GPU". Ich habe sofort die `.bak` zurückgespielt. Luna hat den Check dann auf die Aufruf-Reihenfolge umgestellt. Danach: Kopie dreimal 12/12, alte Fassung 9 FAIL, installiert (per `mv`, `.bak` daneben) zweimal 12/12.
  - Reviews: Sonnet reviewer-cross APPROVE (7 Nits, 3 umgesetzt), ponytail-review "Ship". pr-test-analyzer fand Lücken im Selbsttest: Marker `cpu-suite-did-not-run`, Subset-Modus und `RUN_GPU=0` werden nicht geprüft. Der Code selbst ist laut Review an diesen Stellen richtig.

## #21538 (guest_libcxx_exception_pointer)

- Kein Assignee, kein Claim. Gonic0 hat am 2026-11-21 einen lokalen Patch gepostet, ausdrücklich nicht als PR, und chaseb64 sowie simonbronk gebeten, ihn in #22091 oder #23395 einzubauen. Passiert ist das nicht. Kein offener PR nennt den Test.
- Reproduziert: rot in den Wächter-Reruns (0/3) und seriell (`libc++ exception_ptr did not rethrow its guest exception`).
- Codex Sol (read-only, 30 min) kommt zum Urteil C: ohne Umleitung der Catch-ABI (`__cxa_begin_catch`, `__cxa_end_catch`, `__cxa_rethrow`) nicht sicher fixbar. Ein konkreter Bruch des Patches: Bei Mehrfachvererbung bekommt ein Host-`catch (const Right&)` die Adresse `unwind + 1` statt des angepassten Unterobjekts (`Personality.cpp:47/79`, `Runtime.cpp:484`). Dazu bestätigt Sol den Konflikt mit `RecordHostCatch` aus #22091. Das ist statische Analyse, nicht gemessen.
- Kein Fix-PR. Stattdessen der Ausschluss in #24341 mit `# issue #21538`, weil Gonic0s Kommentar im Issue genau diesen macOS-Fehler beschreibt.

## #24341

- Head [`9049fb06c7`](https://github.com/boykopovar/AnyPS5/pull/24341#issuecomment-7176030696), 15 Ausschlüsse. Gestrichen sind `guest_webrtc_network_manager`, `guest_icu_utf_conversions` und `nkwebkit_inspectorcontentsearchregex`: Ihre Fixes sind gemergt, und jeder bestand 3/3 seriell. Neu sind `guest_libcxx_exception_pointer` (issue #21538) und `guest_libcxx_regex_error` (#27320). `ctest -N` zählt 6635 Tests, mit Liste 6620, alle 15 Namen exakt. Umgesetzt von Luna, committet von Opus (unveränderter Codex-Diff). `check_conventions` grün, Body aktualisiert.
- RavenCurseBurg (19:26Z) meldete zwei Blocker. (1) CTest liest den relativen `--exclude-from-file`-Pfad relativ zu `build` und bricht mit rc 8 ab. Lokal bestätigt: relativ rc 8 mit `Problem reading test list file`, absolut rc 0. (2) Das SDK von `macos-15` hat kein `std::from_chars` für Gleitkomma, deshalb baut `UpscaleMode.hpp:25` nicht. Beides steht im PR-Kommentar, umgesetzt ist es nicht, weil die Ausnahme nur Liste und Parallelität erlaubt.
- Fork-Lauf: Fork-main per `git push` auf `ff5b409dda` gebracht. `gh repo sync` scheiterte, weil dem gh-Token der Scope `workflow` fehlt. Test-Branch `ci-test/macos-full-24341` angelegt (b789a9a38e plus `pull_request`-Trigger) und Draft-PR Romannachwievor/AnyPS5#9 geöffnet. Es entstand kein Lauf. Der Fork legt seit 2026-11-25 00:08Z überhaupt keine Actions-Läufe an, auch nicht für den Push auf main oder für den Report-PR #8. Am Commit hängt nur die Check-Suite der Claude-App (queued), obwohl die API `enabled: true` meldet. PR #9 habe ich um 23:10 geschlossen und den Branch gelöscht. Ohne Lauf gibt es keine Zahlen und keinen Grund, die Parallelität zu ändern.
- Selbst wenn alle Ausschlüsse greifen, bleibt der Job rot: Auf main kompiliert `GuestLibcxxRegexError.cpp` mit Clang nicht, bis #27320 gemergt ist, und `guest_exception_pointer` sowie `patched_libraries` haben keinen Besitzer.
- Blocker: #17425 offen (approved MarcZX10), #11049 offen (CHANGES_REQUESTED von malekaifa, Head `978f05f442`), #26739 offen (approved kripotker und MarcZX10).

## Neue Issues

62 Issues geprüft: 53 mit Update seit 2026-11-24, dazu 14 mit dem Label bug, zusammengeführt. Kein einziges hat einen Assignee. 42 haben einen PR-Bezug oder einen Claim. Die übrigen 20 habe ich einzeln geprüft. Lösbar ist keins: Die meisten brauchen ein bestimmtes Spiel oder AMD-, NVIDIA- oder Intel-Hardware. #27308 braucht Messungen am Firmware-Modul, #26379 eine Entscheidung der Maintainer. #15018 ist seit #11575 erledigt, #26048 seit #25437. Umgesetzt: nichts. Messkommentare auf Issues: keine. Tabelle: `issues-scan.md`.

## Eigene PRs

- [#26812](https://github.com/boykopovar/AnyPS5/pull/26812) per `gh pr ready` auf Review gestellt: `windows` grün (Job 138921246235), Head `24543f9753` unverändert, keine offenen Threads, MarcZX10 APPROVED.
- Alle 15 offenen PRs sind mergeable. Keine roten oder hängenden Checks, keine offenen Threads, kein CHANGES_REQUESTED (`own-prs.md`). Neu seit Lauf 7 sind Approvals von MarcZX10 auf #26739, #26812, #26488 und #24341, ein Handoff-Kommentar von malekaifa auf #22932 und ein COMMENTED ohne Befund auf #24714. Zu beantworten war davon nichts.
- #26488 wartet weiter auf #21939, #26519 ruht bis 2026-12-01.

## Duplikate

- `guest_libcxx_regex_error`: #27320 (magnub).
- `patched_libraries` / `_av_frame_alloc`: kein PR und kein Issue. #26655 nennt `av_frame_alloc` nur im eigenen Diff.
- `guest_exception_pointer`: kein PR und kein Issue.
- `guest_libcxx_exception_pointer`: nur #21538, kein PR.

## Braucht Roman

1. #24341, zwei Änderungen außerhalb der Ausnahme. Zum einen in der ctest-Zeile `--exclude-from-file "$GITHUB_WORKSPACE/.github/macos-full-expected-failures.txt"` (wie RavenCurseBurgs Commit 6f445098d4). Zum anderen ein Runner mit macOS-26-SDK statt `macos-15`. Freigeben, dann setze ich beides um.
2. Fork-Actions: Seit 2026-11-25 00:08Z entsteht kein Lauf mehr. Bitte unter Settings → Actions im Fork nachsehen, ob dort etwas deaktiviert oder gesperrt ist.
3. gh-Token: `gh auth refresh -s workflow`, damit `gh repo sync` den Fork aktualisieren kann. `git push` über den Schlüsselbund ging trotzdem.
4. Zwei neue echte macOS-Fehler ohne Besitzer: `patched_libraries` (die Linkreihenfolge aus #26124, Diagnose in `briefs/patched-libraries-diag.out`) und `guest_exception_pointer`. Sollen das Fix-PRs in Lauf 9 werden?
5. Optional, aus Lauf 7: `sudo pmset repeat wakeorpoweron MTWRFSU 03:25:00`.

## Modellnutzung

| Worker | Jobs | Aufgaben |
|---|---|---|
| Codex Terra (medium) | 2 | Wächter-Härtung, Diagnose patched_libraries |
| Codex Sol (high) | 1 | Bewertung #21538 |
| Codex Luna (low) | 2 | Wächter-Nits, Liste #24341 |
| Codex Astra | 0 | - |
| Sonnet 5.5 | 3 | reviewer-cross Wächter und Liste, pr-test-analyzer beide |
| Haiku 5.5 | 2 | Issue-Sammlung, Stand der eigenen PRs |
| Opus 5.5 | - | Orchestrierung, Nachmessungen, Commits der Codex-Diffs, PR-Texte, 2 Kommentare, ponytail-review, Bericht |

Opus-Anteil am Code: 0 Zeilen in Upstream-Commits. Eine Zeile YAML (`pull_request:`) auf dem Fork-Test-Branch, der inzwischen gelöscht ist.

## Zeitnutzung

- 21:35 bis 21:53: Kontext, Wächter per `kickstart`, Lock. Fork-Prüfung und Test-PR, RavenCurseBurgs Punkte lokal nachgestellt. #26812 ready. Issue-Sammlung und PR-Stand (Haiku). Wächter-Härtung (Terra) mit Review und Nits fertig, Self-Test auf der Kopie grün.
- 21:53 bis 22:38: Der Wächter baut 35 Minuten und läuft dann 26 Minuten in der CPU-Phase. Daneben die Sol-Bewertung von #21538. Viel Wartezeit.
- 22:38 bis 23:00: Reruns des Wächters, keine Jobs. Duplikatsuche für die neuen Fehler, Terra-Diagnose `patched_libraries` gestartet.
- 23:00 bis 23:20: serielle Nachmessung, Liste #24341 (Luna), Reviews, Push, Body und Kommentar, Messkommentar #26124, Fork-PR geschlossen. Installation des Wächters: der erste Versuch zurückgerollt, der zweite grün. Reflect.
- 23:20 bis 23:43: GPU-Phase abgewartet (Ende 23:38), GPU-Neuheiten nachgemessen, Bericht, Aufräumen.

Ungefähr 45 Minuten waren reine Wartezeit auf den Wächter. Der Build dauerte 35 statt 10 Minuten, weil main 1214 Commits weiter war, und XProtect bremste beide Testphasen.

## Reflect

Fünf Verschärfungen in `~/.claude/CLAUDE.md` (Diff `reflect-claude-md.diff`):

- CI-Befehle lokal in derselben Pfadform belegen wie im Workflow. In Lauf 7 habe ich `--exclude-from-file` mit absolutem Pfad geprüft und den rc-8-Fehler deshalb übersehen.
- Vor einem Fork-Lauf prüfen, ob der Fork überhaupt Läufe anlegt.
- Ein laufendes Bash-Skript nur per `mv` ersetzen. Reihenfolgen in Selbsttests nicht per `-nt` prüfen.
- Nach einem großen Sprung von main dauert der Wächter wegen XProtect deutlich länger.
- Für die Wächter-Probe vor dem `mkdir` des Fake-Locks prüfen, dass die PID nicht leer ist und lebt (siehe Panne unten).

Lockerungen, nur als Vorschlag: Die Ausnahme für #24341 könnte die ctest-Zeile und das Runner-Label des neuen Jobs einschließen. Beides war heute belegt nötig, ich durfte es aber nicht anfassen.

## Panne bei der Abschluss-Probe

Nach dem Entfernen von goal.lock wollte ich per `kickstart` zeigen, dass der Wächter nicht mehr blockiert ist, ohne einen echten Lauf zu starten. Der Plan war ein Fake-`nightly.lock` auf eine lebende PID. `pgrep -x Terminal` lieferte aber nichts, die pid-Datei blieb leer, und der Wächter meldete `removing stale nightly lock`. Er startete um 23:41 einen echten Lauf, `nightly/2026-11-25-2341`, pid 75553, schon mit dem gehärteten Skript. Mein anschließendes `rmdir` löschte sogar seinen Lock. Den habe ich mit pid 75553 wiederhergestellt. Abgebrochen habe ich den Lauf nicht, das hätte den Fehler nicht ungeschehen gemacht. Er ist jetzt der erste echte Test von Runde 2 und sollte gegen 01:30 fertig sein, vor dem Termin 03:30. Der Lauf um 03:30 vergleicht dann mit ihm statt mit `2026-11-25-2136`. Belegt ist damit immerhin, dass goal.lock nicht mehr blockiert.

## Nächste Schritte

- `agc_driver_cross_workgroup_flag` (kippt zwischen den Nächten) und `agc_driver_buffer_atomics_or_swap64` (1 von 6 rot): Besitzer suchen, sonst als Kandidaten für Lauf 9.

- Nach deiner Freigabe Punkt 1 in #24341 umsetzen. Danach den Job einmal echt laufen lassen, im Fork oder per `workflow_dispatch` nach dem Merge.
- Den versehentlichen Lauf `nightly/2026-11-25-2341` und den Lauf vom 2026-11-26 03:30 auswerten, beide mit dem neuen Skript. `font_face_lifetime`, `guest_avplayer` und `guest_kqueue` sollten dann unter `cpu-transient` stehen.
- `nkwebkit`, `guest_icu_utf_conversions`, `guest_webrtc_network_manager`: erledigt. `guest_libcxx_regex_error` aus der Liste nehmen, sobald #27320 gemergt ist.

AI-assisted: yes (Claude Code)
