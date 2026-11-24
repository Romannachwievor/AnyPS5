# Nachtlauf 3, 2026-11-24 (04:54 bis 09:54 CET)

**GELB**: zwei neue Draft-PRs mit CI-Beleg. Die zweite Hälfte ging für Wartezeit und ein Nutzungslimit drauf, und der Abschluss war knapp.

## Ergebnis

#26572 bringt `nid_database_cache` auf macOS-CI von 38,4 s auf 1,9 s. Ursache war ein Reverse-DNS-Lookup in `HTTPServer.server_bind` und nicht, wie zuerst vermutet, die Zahl der Subprozesse. #26574 teilt `pr_overlap_branch_scan`; auf Windows-CI dauert der längste Eintrag jetzt 13,5 s statt 22 bis 24 s. Die macOS-CPU-Suite läuft wieder vollständig, und für die neue `guest_formatting`-Lücke gibt es schon einen fremden Fix (#26118), den ich auf dem M1 bestätigt habe.

## Erledigt

- [#26572](https://github.com/boykopovar/AnyPS5/pull/26572) `test(tools): shorten the nid_database_cache test run`, Draft. Zwei Commits: die Tests mit leerem Filter laufen jetzt im selben Prozess (e99784a8f3), und `LocalHTTPServer` überspringt `getfqdn` (89f6062483). CI auf macOS: 1,89 s, vorher auf main 36,97 bis 41,38 s. Alle Checks grün (8 bestanden, 6 übersprungen).
- [#26574](https://github.com/boykopovar/AnyPS5/pull/26574) `test(tools): split pr_overlap_branch_scan into two CTest entries`, Draft. Run 45331670116: Windows 13,52 s + 10,49 s, macOS 5,90 s + 4,15 s. Die CI-Zeile steht jetzt im Body, `check_conventions` ist grün.
- [#26573](https://github.com/boykopovar/AnyPS5/pull/26573) (rebase_verify mit fünf Shards): geöffnet, dann mit Beleg geschlossen. Auf Windows trägt jeder Shard feste Kosten, deshalb wurde die Gesamtzeit länger und kein einzelner Shard kürzer.
- #26488: Body nachgezogen, wartet weiter auf #21939.
- macOS-CPU-Baseline (main 9111e0f78a + #17425 + #11049): 4436 Tests, 20 Fehler unter `-j6`, 1075 s. Davon laufen 5 seriell grün (Last), 12 haben schon einen Besitzer-PR, 3 sind neu (siehe Backlog).
- `guest_formatting` auf macOS: Das System-`vsnprintf` gibt bei genau INT_MAX Zeichen −1/`EOVERFLOW` zurück, die PS5 dagegen die Zahl. Gezeigt hat das ein kleines Probe-Programm, unter anderem mit `%2147483647d`, `a%2147483646d` und `%*d`. Mit #26118 (head 476e2e3b32) läuft `guest_formatting` auf dem M1 grün (3,07 s), ohne ihn rot. Kein eigener PR, der Patch ist wieder entfernt.

## Setup

- Build ../anyps5-build: 11476/11476 fertig um 06:44, nicht gestoppt, kein ninja läuft. Die Testdatei `guest_formatting_tests` ist wieder der saubere Stand von main.
- Hooks (neu, Backup `settings.json.bak-2026-11-24`): PreToolUse `anyps5-block-3rdparty.sh` blockt Edit/Write unter `3rdparty/`. Pipe-Test: 4 Fälle rc 2, 4 Fälle rc 0, live geprüft. PostToolUse `anyps5-conventions.sh` meldet `check_conventions`-Funde als Kontext, Laufzeit 6 bis 7 s, live mit einem Kommentar geprüft.
- Skill `pr-open` (~/.claude/skills/pr-open) über skill-creator angelegt, ohne Evals, in diesem Lauf benutzt.
- Skill-Regeln für den Rest des Laufs:
  - genutzt: diagnosing-bugs, cpp-testing, verification-before-completion, receiving-code-review, resolving-merge-conflicts, humanize-writing, pr-open, reflect; improve-codebase-architecture nur bis zum Bericht, nie das Interview.
  - nie aufgerufen (interaktiv): brainstorming, writing-plans, executing-plans, finishing-a-development-branch, grill-me, grilling, batch-grill-me, grill-with-docs, loop-me, to-questionnaire, wizard, setup-matt-pocock-skills, plan-ceo-review, plan-eng-review, plan-design-review, plan-devex-review, office-hours, careful, guard, using-superpowers.
  - nie aufgerufen (veröffentlicht oder baut neu): ship, land-and-deploy, setup-deploy, canary, to-spec, to-tickets, triage, wayfinder, qa, git-guardrails-claude-code, config-gc, setup-pre-commit.
  - nie aufgerufen (stört): codex (gstack), alle ruflo-Skills, caveman, ultracave, cavecrew. Caveman und using-superpowers kommen automatisch über SessionStart-Hooks und wurden nicht befolgt.

## Top 5 Backlog (N×S×M×G)

| # | Punkt | Score | Stand |
|---|---|---|---|
| 1 | nid_database_cache macOS > 30 s | 240 | Draft #26572 |
| 2 | test_registration Windows > 30 s | 240 | Duplikat von #26280, abgebrochen |
| 3 | rebase_verify Shards Windows | 180 | #26573 geschlossen, kein Gewinn |
| 4 | pr_overlap_branch_scan Windows 29 s | 120 | Draft #26574 |
| 5 | #24341: nkwebkit-Ausschluss streichen | 100 | CI-Config, Braucht Roman |

Neu dazu: `nkwebkit_jscdateparsing` scheitert nur in der Zeitzone Europe/Berlin (Sommerzeit-Lücke "Mar 28 2021 02:30", JscDateParsing.cpp:214). `guest_udp` bricht ohne Meldung ab. Für beide ist noch kein Besitzer gesucht.

## Abgebrochen

- T1 `guest_curl_sockets`: nicht reproduzierbar. 1500 Läufe unter Last ohne Fehler, auch in der vollen `-j6`-Suite grün.
- T3 test_registration teilen: Duplikat von #26280, zu spät gefunden.
- T6 progress_prs gemeinsame Fixture: 261 statt 183 git-Prozesse, aber kein messbarer Zeitgewinn. Der Patch liegt in `t6-progress-prs-shared-fixture.patch`.
- T7 pr_overlap-Fetch: #22252 macht das schon.
- T4 rebase_verify fünf Shards: kein Gewinn auf Windows, PR geschlossen.

## Braucht Roman (nach Wichtigkeit)

1. Reif für Review? `gh pr ready 26572 --repo boykopovar/AnyPS5` und `gh pr ready 26574 --repo boykopovar/AnyPS5`
2. #24064/#24065: CI-Stand prüfen (`gh pr checks 24064 --repo boykopovar/AnyPS5`, ebenso 24065) und neu anstoßen.
3. #24341: nkwebkit-Ausschluss nach #24771 streichen. Das ist CI-Config und für mich gesperrt.
4. Optional: den macOS-Beleg als Kommentar auf #26118 posten (Text oben unter Erledigt). Ich habe es nicht gemacht, weil Kommentare auf fremden PRs nicht klar erlaubt waren.
5. Codex: der ruflo-Marketplace-Eintrag in der Codex-Konfiguration lässt Clones hängen. Eintrag entfernen oder Marketplace deaktivieren.
6. Aufräumen nach Wunsch: `git push fork --delete auto/split-test-registration auto/rebase-verify-more-shards`

Nichts davon blockiert den nächsten Lauf.

## Riskante Vorschläge

- macOS-Gast-Copy-Fix bleibt Vorschlag, solange in #26519 kein Maintainer zustimmt.
- improve-codebase-architecture: Die CTest-Registrierung der Tools-Tests hängt an Klassennamen in CMakeLists. Ein Eintrag je Klasse, automatisch erzeugt, würde T3/T5 überflüssig machen. #26280 und #21939 gehen schon in diese Richtung, deshalb kein eigener PR.

## Modelle

| Rolle | Modell | Wofür |
|---|---|---|
| Architekt | Opus 5.5 | Orchestrierung, T2 bis T5, Diagnose, macOS-Triage, Bericht |
| Scout | Haiku | Skill-Liste, PR-Scans, Flake-Scan F1 |
| Builder/Review | Sonnet | Architektur-Scan A1, Lücken-Scan G1, Reviews |
| Codex Luna | cx/gpt-6-luna low | T2 erster Commit |
| Codex Terra | cx/gpt-6.1-terra medium | Gegenprüfung A1, T6 |
| Codex Sol | cx/gpt-6.2-sol high | 3 Cross-Reviews (T3/T4, T5, T2b), alle APPROVE |
| Codex Astra | cx/gpt-6.2-astra | nur Smoke-Test |

## Zeit

- 04:54 bis 05:10: Setup (Hooks, Skill, Skill-Regeln)
- 05:10 bis 06:10: T1 bis T7
- 06:10 bis 07:05: Build und CPU-Suite
- 07:05 bis 07:15: Triage `guest_formatting`
- etwa 07:15 bis 09:30: Pause durch Nutzungslimit
- 09:30 bis 09:54: #26574-Auswertung und Phase 4

## Nächste Schritte

- Besitzer für `nkwebkit_jscdateparsing` und `guest_udp` suchen. Falls keiner existiert, mit Reproduktion fixen. Beim Datumstest erst klären, ob Test oder Implementierung von der Zeitzone abhängt.
- Vor jeder Testzeit-Aufgabe offene PRs nach CTest-Name und Skriptname durchsuchen.
- Einen Gewinn erst behaupten, wenn die CI-Läufe des eigenen PRs auf Windows und macOS vorliegen. Eine lokale A/B-Messung auf dem M1 sagt Windows nicht voraus.
