# Autonomer Lauf 2, 2026-11-24 (02:28 bis 03:45 CET)

Orchestrator Claude Code (Opus 5.5). Log: `log.md`, Backlog: `backlog.md`, Scans: `scan/`.

## Ergebnis in drei Sätzen

Die eigenen PRs brauchten keinen Code: kein offener Thread, Helldiverr meldet für #24714 „nothing blocks“, für #24064 und #24065 habe ich um einen Re-Run des abgebrochenen `report`-Checks gebeten. Von 14 Scan-Kandidaten hielten nach Skeptiker und eigener Prüfung nur wenige stand; die beiden Debt-Kandidaten habe ich selbst widerlegt (PR #9172, Debt-Zeile 351), und die zwei CI-Fehler auf main gehören schon offenen PRs. Umgesetzt ist ein reproduzierter CI-Timeout (`configure_seed_round_trips`, Draft #26488); neu belegt ist, dass geprüfte Guest-Kopien auf macOS je Aufruf einen Mach-Call kosten (`guest_icu_comparisons` 16 s auf M1, 0,19 s auf Linux-CI).

## Erledigt

- #24064, #24065: Bitte um Re-Run des abgebrochenen `Progress report` an oneandonlydean ([7119290147](https://github.com/boykopovar/AnyPS5/pull/24064#issuecomment-7119290147), [7119290649](https://github.com/boykopovar/AnyPS5/pull/24065#issuecomment-7119290649)). Vorher hatte niemand darum gebeten.
- Draft-PR [#26488](https://github.com/boykopovar/AnyPS5/pull/26488) `test(tools): split the configure seed round trips into two CTest entries`, Head `64f1553253`. Beleg: main-CI Run 45277500115 Timeout 30,67 s; M1 vorher 20,7 bis 31,9 s, nachher 12,1 bis 15,6 s je Hälfte. Review: reviewer-cross APPROVE, pr-test-analyzer ohne Lücken, Negativkontrolle (falscher Klassenname gibt rc 1). pr_overlap: 44 verglichen, 16 manuell, nur #21939 berührt die Zeilen.
- macOS-Regressionscheck auf frischem main (`c55872d43f` + #17425 + #11049): NKWebKit 263/264 (nur bekanntes `nkwebkit_filteroperation`, gehört #24771), LibreSsl/LibreSsl3/Icu/FreeType/Ssl 394/395 (nur `guest_icu_comparisons` Timeout, siehe unten).
- Schlusscheck der eigenen PRs (03:40): #26488 MERGEABLE, 5 grün, 4 laufen. #24714 11 grün; upipibi hat um 03:29 auf RTX 3070 den aktuellen Head getestet (10/10 auf main und PR, 0 `VUID-`), nichts zu tun. #24064/#24065 weiter nur `report` rot, noch keine Antwort auf die Re-Run-Bitte. #24341 4 grün. Der pr-overlap-Bot nennt auf #24064 #13589, dessen Diff berührt libcurl aber nicht mehr: veraltete Warnung.
- Setup: Codex-Modelle neu ermittelt, Reflect-Regeln in `~/.claude/CLAUDE.md` und `~/.claude/agents` ergänzt, `codex-astra.md` angelegt, Fork-PR #1 geschlossen, alte Branches gelöscht.
- Reflect am Ende (nur Verschärfungen): `~/.claude/CLAUDE.md` bekommt drei Regeln (Log-Zeiten nur per `date`, Stub-Meldungen von Workern vor dem Backlog auf Herkunfts-PR, pinnenden Test und Debt-Eintrag prüfen, zsh-Listen als Array oder `${=VAR}`), `scout.md` eine (kein lokales `git log -S` oder `git blame` im Partial Clone).
- Aufgeräumt: Worktrees `wt-main` und `split-seed` entfernt, `git worktree prune` hat dabei auch verwaiste Einträge alter Scratchpad-Worktrees entfernt (Ordner gab es nicht mehr). `../anyps5-wt/label` ist älter als dieser Lauf und bleibt.

## Merge-Entscheidung

Merge-Commits bleiben, kein Rebase, kein Force-Push. Belege: Ruleset 24042318 auf main erlaubt `merge`, `squash`, `rebase` und hat keine Linear-History-Regel (Branch-Protection-API 404); 28 der letzten 30 gemergten PRs sind Merge-Commits; CONTRIBUTING.md:45 verlangt Rebase nur, wenn ein PR nicht mehr sauber mergt; in #24714 hat niemand Rebase verlangt. Gilt für alle weiteren Updates meiner PR-Branches (steht in HANDOFF.md).

## Top 5 des Backlogs (nach eigener Nachprüfung)

| # | Punkt | Beleg | Score | Stand |
|---|---|---|---|---|
| 1 | `configure_seed_round_trips` aufteilen | Run 45277500115, M1 31,9 s, CONTRIBUTING.md:19 | 240 | erledigt, #26488 |
| 2 | macOS: geprüfte Guest-Kopie ohne In-Process-Pfad | General.cpp:96/:116, Test 16 s vs. 0,19 s | 4×4×1×2 = 32 | Vorschlag/Issue-Entwurf |
| 3 | macOS-Regressionscheck heißer Bereiche | S1, #24117 | 192 | erledigt, keine neue Regression |
| 4 | #18785 Linear-Target-Store nach jedem Draw | eigener Bench, Issue offen | 48 | Vorschlag: braucht trace_digest auf GPU |
| 5 | `guest_curl_sockets` paralleler Abbruch | Baseline 11-21 | 54 | nicht neu geprüft (Zeit) |

## Abgebrochen oder verworfen (mit Grund)

- K1 libkernel_sys „Silent stubs“ (`sceKernelAddResource`, `sceKernelEventLogInit`, `sceKernelReleaseTraceDirectMemory`): widerlegt. PR #9172 hat sie aus der Firmware-Disassembly (`xor eax, eax; ret`) gesetzt, `GuestKernelSysConstants.cpp:21-24` pinnt sie. S4 und der Skeptiker hatten das übersehen.
- K8 DCC-Keys mipmapped Array/3D: steht schon in `docs/dev/debt/libSceAgcDriver.md:351`.
- `agc_driver_scattered_runs` auf main-Linux rot (9 Runs): gehört #21649, dessen Body genau diesen Fehler nennt.
- `agc_driver_pm4_order_signal`: #24170.
- `nkwebkit_crossthreadtaskhandler` (Windows, Zeile 415): auf M1 0 von 320 Läufen rot, nicht reproduzierbar, also kein Fix.
- `PipelinedFlushReadOrder.cpp:232` baut auf macOS nicht (`mapped` fehlt ohne `MAP_FIXED_NOREPLACE`): gehört #25331, auf M5 Pro getestet.
- Voller Build von main: nach 7411/11852 Schritten gestoppt (66 Schritte/min, Ende wäre nach der Deadline), danach nur gezielte Ziele gebaut.

## Braucht Roman

- macOS-Guest-Kopie (Punkt 2): Issue-Text liegt fertig in `issue-macos-guest-copy.md`, Duplikatsuche ohne Treffer. Der Auto-Mode von Claude Code hat `gh issue create` abgelehnt, ich habe es nicht umgangen. Posten mit: `gh issue create --repo boykopovar/AnyPS5 --title "macOS: every checked guest copy is a mach_vm_read_overwrite call (guest_icu_comparisons 16 s on M1, 0.19 s on Linux CI)" --body-file ~/Projects/anyps5-reports/auto-run-2026-11-24/issue-macos-guest-copy.md`
- `fix/nkwebkit-filter-operation-type-width` im Fork ist jetzt ein Duplikat von #24771. Löschen nur mit deinem OK.
- #11049 hat seit 2026-11-23 ein CHANGES_REQUESTED von malekaifa; #24341 hängt weiter an #17425 und #11049.
- Reflect-Vorschlag, der eine Grenze lockert und deshalb nicht umgesetzt ist: eine Permission-Regel, die `gh issue create --repo boykopovar/AnyPS5` im Auto-Mode erlaubt. Nur mit deinem OK.

## Riskante Vorschläge (nicht umgesetzt)

- macOS: fehlertolerante In-Process-Kopie für `CopyGuestBytes`, analog `GuestFaultCopy` unter Linux. Signal-Handler unter Rosetta, betrifft jede Guest-Kopie.
- K5 `VulkanDevice.cpp:5184-5194`: GraphicsLibrary-Prebuild markiert den Key auch nach Fehler. Laut Funktionsname gewollt, nur mit Maintainer-Meinung.
- Windows-CI-Job: maximal 65 von 90 min, CI-Konfiguration ist tabu.

## Empfehlungen von claude-code-setup (nichts installiert)

- Hooks: PreToolUse blockt Edits unter `3rdparty/`; PostToolUse lässt `tools/check_conventions.py` nach C++-Edits laufen.
- Skills: `pr-open` (pr_overlap, check_conventions, Template), `debt-fragment` (Fragment-Format).
- MCP: GitHub-MCP oder weiter `gh`.
- Subagent: `convention-reviewer` (read-only, AGENTS.md-Regeln).

## Modellnutzung

| Task | Worker | Modell | Effort | Grund |
|---|---|---|---|---|
| Orchestrierung, Merge-Analyse, Bewertung, eigene Nachprüfung, T3-Review | Orchestrator | Opus 5.5 | – | Architect-Rolle |
| P1, S1, S2, S3, S5, S7 | scout | Haiku | – | Lesen und Sammeln |
| S4 Code-Lücken | general-purpose | Sonnet | – | Vorgabe; git grep statt clangd (compile_commands alt) |
| S6 Fehlerbehandlung | silent-failure-hunter | Sonnet | – | Vorgabe |
| claude-code-setup | general-purpose | Haiku | – | Lesen |
| Skeptiker | `codex exec` direkt | cx/gpt-6.1-terra | medium | Vorgabe; Wrapper braucht Worktree |
| T3 Umsetzung | codex-luna | cx/gpt-6-luna | low | mechanisches Verschieben |
| T3 Review | reviewer-cross | Sonnet | – | Codex-Diff, Claude prüft |
| T3 Tests / stille Fehler | pr-test-analyzer, silent-failure-hunter | Sonnet | – | Phase 4 |

Codex-Smoke-Test: OK für cx/gpt-6.1-terra, cx/gpt-6.1-sol, cx/gpt-6-astra, cx/gpt-6-luna; Timeout für alle 6.2-Namen, gpt-6-terra, gpt-6.1-astra, gpt-6.1-luna. Astra wurde nicht gebraucht.

## Laufzeit

Start 02:28:20, Ende 03:45 (Deadline 04:28:20), also etwa 1 h 17 min. Länger liefen `pr_overlap --branch` (03:12 bis 03:35, 2598 PR-Köpfe) und die Tests unter Rosetta neben dem gestoppten Vollbuild.

## Nächste Schritte

1. #26488 beobachten; landet #21939 zuerst, die Registrierung nach JSON verschieben.
2. macOS-Guest-Kopie: Issue posten (siehe „Braucht Roman“), Antwort abwarten, dann ggf. Port von `GuestFaultCopy` mit Rosetta-Messung.
3. Re-Runs auf #24064/#24065 prüfen.
4. `guest_curl_sockets` mit dem aktuellen Build wiederholen.
