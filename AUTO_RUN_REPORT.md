# Lauf 5, 2026-11-24 (16:53 bis 21:53 CET)

**GRÜN**: zwei echte macOS-Brüche von heute gefunden und als Draft-PRs gemeldet, #26739 hat schon ein Approve. Abgebrochen wurde nur die statische Prüfung durch Codex Sol, die zweimal im Codex-Filter hängen blieb. Die CI der beiden neuen PRs steckt in der vollen Upstream-Queue.

## Ergebnis

Auf `main` läuft keine macOS-CI, deshalb sind heute zwei macOS-Brüche unbemerkt gelandet. Beide habe ich mit der vollen macOS-CPU-Suite gefunden, und für beide gibt es jetzt einen Draft-PR: #26729 repariert den Test `guest_webrtc_network_manager` (seit #24668 rot), #26739 macht libc auf macOS wieder baubar (seit #10416 bricht der Build ab). Der Linux-Weg über Docker Offload ging nicht, der Sanitizer-Sweep der macOS-only-Pfade blieb ohne Befund, und #26673 hat die beiden Testfälle mit Adresse 4 mod 8 bekommen.

## Erledigt

- [#26739](https://github.com/boykopovar/AnyPS5/pull/26739) `fix(libc): declare the __sub_kind values on libc++abi hosts`, Draft, head 6d85fbf66b. #10416 (MarcZX10, gemergt 16:56Z) nutzt in `Runtime.cpp` die Enumeratoren von `__class_type_info::__sub_kind`. libstdc++ deklariert sie, libc++abi nicht, und der `_LIBCPPABI_VERSION`-Zweig in `CxxAbi.hpp` kannte nur `__unknown`. Seitdem bricht der libc-Build auf macOS bei `Runtime.cpp:296` ab. Der Fix ergänzt die Werte von libstdc++, geprüft gegen GCC 15.2 `cxxabi.h:386-447`. `guest_dynamic_cast` 3/3, auch nach dem Merge von main. Mutation `__contained_public = 4`: der Test bricht ab. 160 Exception-, RTTI- und JSON-Tests: mit Fix 157/160, mit dem main-Stand vor #10416 ebenfalls 157/160, dieselben drei bekannten Fehler. Umgesetzt von Codex Luna. Auf #10416 steht ein Kommentar mit Fehlerzeile und Verweis auf den Fix.
- [#26729](https://github.com/boykopovar/AnyPS5/pull/26729) `test(libwebrtc): zero the address unions in the NetworkManager test`, Draft, head 920773c7c7. Der Test initialisiert die Adress-Union mit `{}`. Das setzt nur das erste Mitglied, also 4 der 16 Bytes. AppleClang lässt im Rest Müll stehen, GCC zufällig Nullen, deshalb war der Test nur auf macOS rot (Zeile 109). Jetzt `{.ip6 = {}}` an allen vier Stellen, dazu ein Lookup, bei dem nur Byte 8 abweicht. main 0/3, Fix 3/3, Mutation (`operator==` vergleicht nur 8 Bytes) rot an der neuen Zeile 111. Zusammen mit #26739 lokal 3/3. Diagnose und Fix von Codex Terra, Nacharbeit von Codex Luna.
- [#26673](https://github.com/boykopovar/AnyPS5/pull/26673): Testfälle mit Records an Adresse 4 mod 8 für module, clazz und library, dazu `static_assert(alignof == 8)`. Commits e455901286 und 1d869622d7. Fix 3/3, jede der drei Mutationen 0/2 mit "accepted". CI: alle Checks grün.
- Eigene PRs (Prüfung um 17:00, 18:45 und 20:06): #26739 hat ein Approve von kripotker (Werte gegen GCC 15.2 geprüft), seine Checks warten noch. #26729: 4 grün, 3 warten. #26673 und #26661: alle Checks grün, Helldiverr hat beide auf Windows bestätigt. #26488, #24064 und #24065 grün. #26488 bleibt, weil #21939 noch offen ist. Die pr-overlap-Warnung auf #26661 nennt #24124. Der ändert in `tests/WtfSha1.cpp` die Zeilen 3 bis 27 und ab 351, #26661 nur Zeile 155, es gibt also keine Überschneidung.
- macOS-CPU-Suite zweimal. Auf main adacf0bd83 (+ #17425 + #11049): 4508 Tests, 16 rot, neu nur `guest_webrtc_network_manager` (daraus #26729) und ein Last-Timeout von `guest_posix_spawn`. Auf main 17220db6a3 + #26739: 50 rot, davon 34 neu. 31 der neuen liefen gar nicht erst an: Dem Worktree fehlte #17425, deshalb baute das curl-Backend arm64, `libcurl.prx` linkte nicht, und alle Tests, die daran hängen, fehlten. Mit #17425 lokal gemergt waren alle 34 grün. Die übrigen drei waren auch vorher seriell 3/3 grün (Last unter `-j6`). Der aktuelle main ist mit #26739 auf macOS-CPU also sauber, bis auf den Test, den #26729 repariert.
- GPU-Suite seriell, zweimal. adacf0bd83: 2036 Tests, 34 rot, keiner neu gegenüber Lauf 4. 6d85fbf66b + #26739: 2042 Tests, 36 rot, zwei davon neu. `cross_workgroup_flag` wechselt zwischen grün und rot (#14217). `agc_driver_profiled_dispatch_run` stürzt unter Wiederholung ab (Lauf 11 von 30), auf adacf0bd83 genauso (Lauf 19 von 60). Das ist der MoltenVK-Fehler, den #25421 (Gonic0) beschreibt, also alt und mit Besitzer.
- UBSan-Sweep der macOS-only-Pfade in libc und libkernel (469 Tests, Flags wie im Nightly): 0 Reports. Mit Clangs Standardflags gibt es 50 Treffer von `-fsanitize=function`. Das Projekt schaltet die Prüfung ab (CMakeLists.txt:233), es ist also kein Task.

## Setup

- Docker Offload: `docker info` ok. `docker offload create --cpus 8 --memory 16g --idle-timeout 10m --ttl 4h` hing 240 s ohne jede Ausgabe und ohne Trace-ID (rc 124). `docker offload stop` war erfolgreich. Rechenzeit: 0 min CPU, 0 min GPU. Eine alte Sandbox stand seit fünf Tagen auf "Stopping"; am Ende des Laufs zeigt `status` "Stopped".
- Weg 2 (Nightly-Workflow im Fork per `workflow_dispatch`): HTTP 404, weil `sanitizers.yml` auf Fork-main nicht registriert ist. Registrieren hieße CI-Config im Fork ändern, das ist verboten.
- Weg 3 (lokales Docker, `linux/amd64` unter Rosetta): Kontrolltest bestanden, GCC 13.3 mit ASan meldet einen bekannten heap-use-after-free. Gebraucht habe ich ihn nicht, weil kein Linux-only-Fehler ohne Besitzer übrig war.
- Builds liefen nur über den neuen Wrapper `~/Projects/anyps5-tools/aps5-build` (Lock, Log, ninja-rc). `pause-builds stop|cont` hält laufende Builds an, wenn ein Test mit Timeout laufen muss.
- Codex: Smoke-Test aller vier Modelle rc 0, PONG in 25 bis 28 s. Jeder Job lief mit `-c 'plugins."ruflo-core@ruflo".enabled=false'`.

## Top 5 Backlog (N² × S × M × G)

| # | Punkt | Score | Stand |
|---|---|---|---|
| 1 | #18785 Linear-Target-Store (Performance) | 144 | braucht `trace_digest` mit Titel-Trace |
| 2 | #26623 AMD Vega, letzter Vertex liest 0 | 128 | riskanter Vorschlag, braucht AMD |
| 3 | OCSP feldweise lesen (LibreSsl3 OcspFindStatus.cpp:251-262) | 128 | Vorschlag, gurtgg arbeitet daneben |
| 4 | Forward-Slot-Prüfungen LibcxxCondition.cpp:15, VersionedFilesystemCalls.cpp:38 | 128 | nur Test-Fakes erreichen den Pfad |
| 5 | macOS-Gast-Copy (#26519) | 128 | Vorschlag, kein Maintainer hat geantwortet |

Neu: `GuestCopyOut` auf macOS kürzt die Byte-Zahl auf 32 Bit (libc/src/General.cpp:992-993, betrifft erst Kopien ab 4 GiB, Score 48), ein Flake-Kandidat `guest_kernel_sys_paths_symlinkat` unter `-j6` (nicht reproduzierbar, 12) und der Last-Timeout von `guest_posix_spawn` (64).

## Abgebrochen

- Statische Prüfung des macOS-only-Codes durch Codex Sol (Jobs M und M2): beide Male nach rund 380k Tokens vom Codex-Filter beendet ("This content was flagged for possible cybersecurity risk"), ohne Ergebnis. Aus dem Transkript stammt nur der `GuestCopyOut`-Punkt oben. Nicht wiederholt.
- Vollbuild mit UBSan: nach 119 von 12016 Schritten abgebrochen. Er hätte alle Tests rund 75 min blockiert, und der Linux-Nightly deckt den Code außerhalb von `__APPLE__` schon ab. Der Sweep lief stattdessen über 331 gezielte Ziele.
- Die drei ersten Läufe von W nach dem Merge von main habe ich verworfen. libc baute nicht (das war R), die Tests liefen also gegen das alte Binary.

## Duplikate (vor Arbeitsbeginn ausgeschlossen)

- Pflichtkandidat `nkwebkit_dragimage`: #25970, gemergt 12:01Z.
- `patched_self_imports` (rot auf main Linux, Run 45411652885): #26697 (oneandonlydean).
- `draw_allocations`: #25615 (kevinfavv).
- Issues: #26650 (#26662, #26693), #26649 (#26652), #26636 (#26691, #26630). #26642 und #26685 brauchen AMD- bzw. NVIDIA-Hardware, #26728 dazu Dusk Diver 2.
- Übrige Nightly-Sanitizer-Fehler: alle mit offenem Besitzer-PR (#26222, #21336, #24992, #23507, #24032, #23886, #16813).
- Für W und R: keine offenen PRs oder Issues nach Testname, Dateiname, Funktionsname und Fehlertext; `pr_overlap --names` und `--branch` ohne Überschneidung.

## Braucht Roman (nach Wichtigkeit)

1. #26739 und #26729 sind Drafts. Sobald die CI grün ist, auf Review stellen: `gh pr ready 26739 --repo boykopovar/AnyPS5` und `gh pr ready 26729 --repo boykopovar/AnyPS5`. #26739 zuerst, ohne ihn baut libc auf macOS nicht.
2. Fork-Branch `auto/san-main-run5` stammt aus dem Versuch über Weg 2. Der Prompt erlaubt mir nicht, ihn zu löschen: `git push fork --delete auto/san-main-run5`
3. Docker Offload: `create` hängt ohne Ausgabe. Die alte Sandbox stand zu Beginn seit fünf Tagen auf "Stopping", am Ende zeigt `docker offload status` "Stopped", und desktop-linux ist der aktive Kontext. Offload für das Konto prüfen oder die Offload-Regel aus dem Prompt nehmen.
4. Bei #26519 bis zum 2026-12-01 nicht nachfassen (unverändert).
5. Codex-Wrapper in `~/.claude/agents/codex-*.md`: Den `codex exec`-Befehlen fehlt das Pflicht-Flag `-c 'plugins."ruflo-core@ruflo".enabled=false'`, und es fehlt der Hinweis, Briefings ohne Sicherheitsvokabular zu formulieren. Ich wollte beides ergänzen, der Auto-Mode-Klassifikator hat die Änderung abgelehnt. Diff: `reflect-agents-proposal.md` im Run-Ordner. In diesem Lauf habe ich Codex direkt mit dem Flag gestartet.

## Riskante Vorschläge

- #26623 und der macOS-Gast-Copy-Fix (#26519) bleiben Vorschläge, Begründung wie in Lauf 4.
- `GuestCopyOut` macOS: `mach_vm_write` bekommt die Länge als `mach_msg_type_number_t` (32 Bit). Ein Fix müsste in Blöcken schreiben. Ohne einen Gast, der 4 GiB am Stück kopiert, lässt sich das nicht testen.

## Modelle

| Rolle | Modell | Wofür |
|---|---|---|
| Orchestrator | Opus 5.5 | Planung, Briefings, Duplikatsuche, alle Builds, Testläufe und Mutationen, PR-Texte, Kommentar, Bericht |
| Codex Terra | cx/gpt-6.1-terra medium | Task W (Diagnose und Fix) |
| Codex Luna | cx/gpt-6-luna low | #26673-Testfälle (drei Runden), W2, Task R |
| Codex Sol | cx/gpt-6.2-sol high | statische macOS-Prüfung M und M2, beide vom Filter beendet |
| Codex Astra | cx/gpt-6.2-astra high | nur Smoke-Test |
| Sonnet | claude-sonnet-5-5 | reviewer-cross, pr-test-analyzer und silent-failure-hunter für A, W und R |
| Haiku | claude-haiku-5-5 | Signal-Scan |

Opus-Anteil an Code und Tests: 0 von 3 Diffs. Kein Codex-Job lag über seinem Zeitlimit.

## Zeit

- 16:53 bis 17:05: Phase 0 und 1 (Kontext, Tools, Codex-Smoke-Test, Offload-Versuch, Weg 2, eigene PRs, Pflichtkandidat als Duplikat erkannt)
- 17:05 bis 17:32: Testfälle für #26673 (Codex Luna, drei Runden, Verifikation), UBSan-Sweep der macOS-Pfade, statische Prüfung durch Sol (zweimal abgebrochen), Kontrolltest Weg 3
- 17:32 bis 17:55: macOS-CPU-Suite auf adacf0bd83, daraus W
- 17:55 bis 18:38: Task W (Terra, dann Luna), parallel die GPU-Suite und die Overlap-Scans, #26729
- 18:36 bis 19:26: Task R, beim Merge von main entdeckt. Verifikation, Reviews, Overlap-Scans, #26739, Kommentar auf #10416
- 18:50 bis 19:23: Vollbuild und CPU-Suite auf dem aktuellen main
- 19:27 bis 20:05: Vollbuild und GPU-Suite auf dem aktuellen main, Flake-Prüfung von `profiled_dispatch_run`
- 20:05 bis 20:15: Nachreviews, Schluss-Check der eigenen PRs, Nachtest mit #17425
- ab 20:15: Phase 4 (Bericht, Report-PR, Reflect, Aufräumen)

Genutzt: etwa 3,5 von 5 Stunden. Danach gab es keinen Kandidaten mit Beleg mehr, und der Platz für Testinfrastruktur war mit W belegt. Auf die CI der beiden neuen PRs zu warten, hätte nichts geändert: Sie steht in der Upstream-Queue.

## Reflect

Umgesetzt in `~/.claude/CLAUDE.md`, alles Verschärfungen:
- Build-rc prüfen gilt jetzt auch nach jedem Merge von main. Nach dem Merge in W liefen drei Tests gegen das alte Binary, weil libc nicht mehr baute.
- Die ctest-Fehlerliste nur aus den nummerierten Zeilen bilden. Meine erste Zählung nahm drei Textzeilen mit (53 statt 50).
- `docker offload create` nur mit `timeout 240`.
- Suite-Vergleiche nur zwischen Builds mit denselben lokalen Merges (#17425, #11049).

Lockerung, nur als Vorschlag: Ein Test, der wegen eines Fehlers im Test nur auf einer Plattform rot ist (wie W), sollte als Korrektheit zählen und nicht den einen Platz für Testinfrastruktur belegen.

## Nächste Schritte

- CI von #26739 und #26729 verfolgen. Die Upstream-Queue war am Abend voll (100 und mehr Runs in der Warteschlange).
- Nach einem Merge von #26739 die macOS-CPU-Suite auf main wiederholen, dann mit #17425 im Build. Ohne #17425 laufen 31 Tests auf macOS gar nicht an.
- Die volle macOS-Suite auf dem aktuellen main gehört an den Anfang jedes Laufs. Beide Funde dieses Laufs kamen aus ihr, und keine CI prüft macOS.
