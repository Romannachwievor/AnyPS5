# Lauf 4, 2026-11-24 (13:02 bis 18:02 CET)

**GRÜN**: zwei neue Draft-PRs aus einer neuen Signalquelle, beide mit grüner CI, dazu zwei M1-Belege auf fremden PRs. Kein Task abgebrochen. Etwa 80 Minuten gingen auf das Warten auf die CI, weil nach der Kategorie-Regel kein weiterer Task frei war.

## Ergebnis

Der nächtliche Sanitizer-Job auf `main` ist seit sechs Nächten rot. Von seinen 45 Fehlern hatten nur zwei Gruppen keinen Besitzer, und für beide gibt es jetzt einen Draft-PR: #26673 behebt echtes undefiniertes Verhalten im Emulator (libSceFreeTypeOt), #26661 zwei Tests mit `memcmp` auf leeren Bereichen. Die beiden Pflichtkandidaten aus Lauf 3 gehörten schon fremden PRs; dort steht jetzt je ein M1-Beleg (Basis 0/3, mit PR 3/3), und #26572 aus Lauf 3 ist gemerged.

## Erledigt

- [#26673](https://github.com/boykopovar/AnyPS5/pull/26673) `fix(libSceFreeTypeOt): refuse misaligned guest records in ft_module_get_service`, Draft, head 2b8a758c5f. `ft_module_get_service` griff auf Felder von Gast-Zeigern zu, bevor es sie geprüft hatte. Bei Adresse 1 ist das UB, und UBSan bricht `guest_freetype_ot_module_service` und `_patched` ab (Nightly-Run 45321508811, `ModuleService.cpp:39`). Jetzt wird ein falsch ausgerichteter Zeiger abgelehnt wie ein unlesbarer, nach dem Muster von libSceFreeType, dazu ein Debt-Fragment. M1 mit UBSan: main 0/3, Fix 3/3 ohne `runtime error`, auch nach dem Merge von main. Mutation (nur den clazz-Check entfernen): wieder rot bei `ModuleService.cpp:47:91`. Umgesetzt von Codex Terra, Nacharbeit (toter Code, Fragmentname) von Codex Luna. CI: alle Checks grün, Windows 22m32s (Run 45413063430). Die Linux-, macOS- und Relinker-Jobs hat der Workflow übersprungen, der Linux-Sanitizer-Lauf selbst kommt erst nach einem Merge im Nightly.
- [#26661](https://github.com/boykopovar/AnyPS5/pull/26661) `test(libSceJsc,libSceLibreSsl3): skip memcmp for empty ranges in two tests`, Draft, head 0ce942e24a. glibc deklariert `memcmp` als nonnull, und `data()` eines leeren Vektors darf null sein. Zwei Zeilen Testcode, beide Assertions bleiben gleich streng (Länge war in diesen Fällen 0). macOS sieht den Fehler normalerweise nicht, weil das SDK `memcmp` ohne nonnull deklariert. Nachgestellt habe ich ihn mit einem Header, der per `-include` eingebunden wird und `memcmp` mit `nonnull(1, 2)` neu deklariert: main 0/2 an denselben Zeilen wie im Nightly, Fix 2/2. Umgesetzt von Codex Luna. CI: alle Checks grün, Windows 15m32s (Run 45408926859), Linux, macOS und Relinker vom Workflow übersprungen.
- M1-Belege auf fremden PRs (Heads direkt vor dem Posten geprüft):
  - [#26591](https://github.com/boykopovar/AnyPS5/pull/26591#issuecomment-7131477397), `nkwebkit_jscdateparsing` in Europe/Berlin und New_York: Basis 0/3, mit PR 3/3.
  - [#25134](https://github.com/boykopovar/AnyPS5/pull/25134#issuecomment-7131478366), `guest_udp`: Basis 0/3, mit PR 3/3.
- #26572 (Lauf 3, `nid_database_cache`) ist um 14:20 CET gemerged.
- Eigene PRs: 13 offen, alle Checks grün, keine neuen Reviews oder Kommentare von Menschen. Schluss-Check um 16:45: seit 13:40Z kein neues Ereignis. Bei #24714, #24065, #23595 und #23587 hat sich um 15:27 bis 15:30Z nur `updatedAt` geändert, die Timeline zeigt dort nichts.
- macOS-CPU-Suite (main 403fbe6fc1 + #17425 + #11049, `-j6`): 4478 Tests, 15 Fehler, alle bekannt und mit Besitzer.
- GPU-Suite seriell: 2036 Tests, 37 Fehler. Die neuen haben einen Besitzer (`cross_workgroup_flag` #14217, `scratch_pool` #24201) oder liegen an der Umgebung. `runtime_buffer_spans_x3` lief beim zweiten Versuch grün. `draw_allocations` zählt drei Allokationen in Apples Metal-Treiber mit; das steht im Backlog.
- Signale: Issue #26623 (AMD Vega, Vertex-Range) hat Codex Sol bewertet. Einen sicheren Kleinfix gibt es nicht, es bleibt ein Vorschlag. #26602 ist ein Duplikat (#26604 gemerged, #26605 offen). In den Discussions ist nichts offen.

## Setup

- Build-Worktree ../anyps5-build: main 403fbe6fc1 + #17425 + #11049, 7777/7778. Einziger Fehler ist `agc_driver_pipelined_flush_read_order_tests` (bekannt, #25331). Ab 13:25 mit voller Leistung (`-j $(sysctl -n hw.ncpu)`, ohne nice). Tests mit Timeouts liefen nie parallel zu einem Build, GPU-Tests nur seriell.
- UBSan-Builds für A und B in eigenen Worktrees (`build-ubsan`, x86_64, AppleClang 21, `-g1 -fsanitize=undefined -fno-sanitize=vptr`). Konfigurieren 125 s, erster Build 146 s (A) bzw. 294 s (B).
- Docker lokal: nicht genutzt.
- Docker Offload: ein Versuch um 14:09:42, für den Linux-Repro von B. Antwort nach 24 s: "Docker Offload is not enabled for this account" (Trace a1cee436ac2fd01065c15fede88dd161). 0 Minuten Rechenzeit, keine GPU. Am Ende zeigt `docker offload status` "Stopped", keine Session. Ersatz war der nonnull-Header auf macOS.
- Codex: Der ruflo-Plugin-Eintrag in `~/.codex/config.toml` bleibt unverändert. Jeder `codex exec` lief mit `-c 'plugins."ruflo-core@ruflo".enabled=false'`. Smoke-Test aller vier Modelle: PONG in 29 s.
- Sanitizer-Workflow als neue Signalquelle: `.github/workflows/sanitizers.yml`, nightly 03:41Z, Linux, GCC, ASan und UBSan. Die Liste bekannter Fehler hat 3 Einträge.

## Top 5 Backlog (N² × S × M × G)

| # | Punkt | Score | Stand |
|---|---|---|---|
| 1 | #18785 Linear-Target-Store (Performance) | 144 | braucht `trace_digest` mit Titel-Trace |
| 2 | #26623 AMD Vega, letzter Vertex liest 0 | 128 | Sol: kein sicherer Kleinfix, nicht auf M1 reproduzierbar |
| 3 | OCSP feldweise lesen (LibreSsl3 OcspFindStatus.cpp:251-262) | 128 | Vorschlag aus Lauf 3 |
| 4 | macOS-Gast-Copy (#26519) | 128 | Vorschlag, kein Maintainer hat geantwortet |
| 5 | `nkwebkit_dragimage` ASan heap-use-after-free in cairo | 72 | ohne Besitzer, braucht Linux-ASan-Stack |

Neu in diesem Lauf: die beiden Sanitizer-Punkte (jetzt PRs), `draw_allocations` (Testinfra, 48) und `nkwebkit_dragimage`. Die übrigen Sanitizer-Fehler haben Besitzer, siehe unten.

## Abgebrochen

Kein Task abgebrochen. Nicht weiterverfolgt:
- G2 (Lücken neben frisch gemergten Fixes): Codex Terra hörte nach 266k Tokens mit Teilergebnissen auf. Die zwei `memcpy` in libSceNpCommon `Runtime.cpp:1807` und `:1628` gehören zur Serie von #26060 (milescoviello), Duplikatrisiko. Nicht wiederholt.
- Linux-Repro von B per Offload: nicht freigeschaltet.
- Mein erster Mutationsversuch für A war ungültig. zsh hat die Zielliste in `$P` nicht in Wörter zerlegt, der Build schlug still fehl, und der Test lief gegen das alte Binary. Mit `xargs` wiederholt, Ergebnis rot wie erwartet.

## Duplikate (vor Arbeitsbeginn ausgeschlossen)

- `nkwebkit_jscdateparsing`: #26591 (gccody)
- `guest_udp`: #25134 (Gonic0)
- Windows-Flake `nkwebkit_wtf` (#26602): #26604, #26605
- Sanitizer: 24 `nkwebkit_jsc*`, `nkwebkit_base`, `jsc_yarr_regular_expression` und `nkwebkit_yarrregularexpression` gehören zu #26222; `agc_driver_recorded_draw_retention` zu #21336; `nkwebkit_wtfstringargumentguestpointers` zu #24992; `agc_shader_mode_register` zu #23507; `agc_gcn_color_target` zu #24032; `agc_driver_pm4_order_signal` zu #23886; drei `agc_driver_graphics*` stehen schon in der Liste bekannter Fehler (#16813).
- GPU: `cross_workgroup_flag` #14217, `scratch_pool` #24201, weitere siehe gpu/triage.md
- Langsame Tests: `guest_ces_wide`/`_module` #24848

## Braucht Roman (nach Wichtigkeit)

1. Beide PRs haben grüne CI. Auf Review stellen: `gh pr ready 26673 --repo boykopovar/AnyPS5` und `gh pr ready 26661 --repo boykopovar/AnyPS5`
2. Docker Offload für das Konto freischalten oder die Offload-Regel aus dem Prompt nehmen. Ohne Offload lassen sich Linux-only-Fehler nur über Umwege wie den nonnull-Header nachstellen.
3. #26519 (macOS-Gast-Copy): weiter keine Antwort eines Maintainers. Nachfassen oder liegen lassen.

Nichts davon blockiert den nächsten Lauf.

## Riskante Vorschläge

- #26623: Laut Sol ließe sich das Problem nur lösen, wenn gepolsterte Vertex-Bindings separat gestaged und aus der Snapshot-Wiederverwendung herausgehalten werden. Das betrifft Draw.cpp und Resources.cpp und lässt sich auf dem M1 nicht testen. Nur mit einem Maintainer und AMD-Hardware angehen.
- macOS-Gast-Copy-Fix bleibt Vorschlag, solange in #26519 kein Maintainer zustimmt.

## Modelle

| Rolle | Modell | Wofür |
|---|---|---|
| Orchestrator | Opus 5.5 | Planung, Briefings, Duplikatsuche, alle Verifikationsläufe, M1-Belege, PR-Texte, Bericht. Kein Produktivcode, keine Tests. |
| Codex Terra | cx/gpt-6.1-terra medium | Task A (Fix + Fragment), G2-Scan |
| Codex Luna | cx/gpt-6-luna low | Task B, Nacharbeit A |
| Codex Sol | cx/gpt-6.2-sol high | Bewertung #26623 (445k Tokens) |
| Codex Astra | cx/gpt-6.2-astra high | nur Smoke-Test |
| Sonnet | claude-sonnet-5-5 | silent-failure-hunter (A), reviewer-cross (B), pr-test-analyzer (A, B) |
| Haiku | claude-haiku-5-5 | Scouts: Signale, Discussions, langsame Tests, GPU-Triage |

Opus-Anteil an Code und Tests: 0 von 2 Diffs. Opus hat Ergebnisse geprüft (Builds, Testläufe, Mutation) und gh-Abfragen selbst ausgeführt.

## Zeit

- 13:02 bis 13:10: Phase 0 und 1 (Kontext, Smoke-Test der Codex-Modelle, eigene PRs, beide Pflichtkandidaten als Duplikate erkannt)
- 13:10 bis 13:45: Scans (Signale, G2, Discussions, langsame Tests), M1-Belege für #26591 und #25134, macOS-CPU-Suite
- 13:44 bis 14:33: GPU-Suite seriell, parallel die Bewertung von #26623 durch Sol
- 14:05 bis 14:58: Sanitizer-Funde, Tasks A und B mit UBSan-Builds, Repro, Fix, Mutation und Reviews; #26661 um 14:58
- 14:58 bis 15:20: Overlap-Prüfung für A (`--names` 22 min, `--branch` 20 min), #26673 um 15:20
- 15:20 bis 16:43: Kreuz-Review, Reflect, HANDOFF, Memory, Warten auf die Windows-CI
- 16:43 bis Schluss: Phase 4 (Bericht, Report-PR, Aufräumen)

Genutzt: etwa 4 von 5 Stunden. Die letzte Stunde war vor allem Warten auf CI.

## Nächste Schritte

- Regelvorschlag (Lockerung, nur hier): Bleiben nach den Tasks mehr als zwei Stunden, sollte ein zweiter Korrektheits-Task erlaubt sein, wenn er aus derselben neuen Signalquelle kommt und keine Kategorie offen ist. In diesem Lauf blieb die Zeit sonst ungenutzt.

- CI von #26673 und #26661 verfolgen (`gh pr checks`). Nach einem Merge zeigt der nächste Nightly-Sanitizer-Lauf, ob die vier Tests aus der Fehlerliste verschwinden.
- `nkwebkit_dragimage`: einen Besitzer gibt es nicht. Ohne Linux-ASan-Stack ist unklar, welcher Frame den Speicher freigibt.
- `draw_allocations` auf macOS: Die Obergrenze zählt Allokationen in AGXMetal mit. Ein Testinfra-Task für einen späteren Lauf.
- Optional für A: ein Testfall mit einer Adresse, die auf 4 aber nicht auf 8 ausgerichtet ist (pr-test-analyzer, Wichtigkeit 2/10).
