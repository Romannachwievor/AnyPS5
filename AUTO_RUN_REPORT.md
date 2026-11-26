# Lauf 10 (Claude), 2026-11-26, 19:07-20:16 und 22:51-23:00 CET

Ampel: GELB. Die Grundlagen für macOS-Gate und Merge-Liste sind gelegt, gemessen wurde von den acht Gate-PRs keiner. Usage-Limit von 20:16 bis 22:51, danach war das 3-h-Budget vorbei.

## Ergebnis

Auf main baut libwebrtc seit #21347 nicht mehr auf macOS (`AudioDecoderOpusImpl.cpp:576:41`, `std::max` mit `size_t` und `uint64_t`). Mein Fix war fertig, da kam #27621 mit derselben Zeile. Statt eines doppelten PRs steht dort jetzt meine M1-Messung: Basis rot, mit der Änderung grün, `ctest -R webrtc` 127/127. Die Merge-Liste mit 712 PRs liegt in `merge-queue.md`. Romans Konto kann aber nicht mergen, weil es auf upstream nur Leserecht hat.

## Codex-Stand und Abgrenzung

Stand beim Start in `codex-status.md`: kein `codex exec`, Lauf 9 ohne Bericht, goal.lock abgelaufen. Zwischen 20:53 und 21:40 war Codex wieder aktiv: KNOWLEDGE.md, STATE.md, Bericht in auto-run-2026-11-26, neuer goal.lock bis etwa 23:53. Dazu kamen Claims auf HANDOFF.md, CLAIMS.md und anyps5-contribution.md, die auf `active` stehen.

Wegen Codex übersprungen:
- Wissensdateien, Wächter, Romans GitHub-Aktivität, patched_libraries, guest_exception_pointer, #24341, Issues-Scan.
- goal.lock (beide Male von Codex).
- HANDOFF.md und die Memory-Datei (aktiver Codex-Claim). Mein Stand steht stattdessen in `orchestrator/claude-state-2026-11-26.md`.
- In CLAIMS.md habe ich nur Zeilen angehängt, so wie Codex auch.

## Maintainer

- **Rechte:** `{"admin":false,"maintain":false,"pull":true,"push":false,"triage":false}`.
- **Re-runs:** 0. Dafür fehlt das Recht. #24064 und #24065 sind grün. 7 von 9 Gate-Kandidaten haben je 2 abgebrochene Checks.
- **Labels:** 0, aus demselben Grund.
- **macOS-Gate:** Aus 2426 offenen PRs blieben 129 mit libc-, libkernel-, specifics/macos- oder Top-CMake-Änderungen und mindestens 2 Approvals. Davon haben 16 macOS-Code im Diff. Gewählt habe ich 22091 24277 24415 25091 25255 25426 25491 26646. #23196 habe ich gestrichen, weil er einen Volllauf mit ICU-Neubau auslöst.
  - Alle acht mergen konfliktfrei auf main eb2e3e1cf7 + #17425 #11049 #26739.
  - Der Basis-Build ist fertig: 12348 Ziele, 45 min.
  - Gemessen und kommentiert habe ich keinen.
- **Zusätzlicher Gate-Kommentar:** #27621, Review 6080902950.
- **Merge-Liste:** `merge-queue.md`. Darin stehen 712 PRs, die nicht macOS-relevant sind, grün, MERGEABLE, mit Approval auf dem Head und ohne CHANGES_REQUESTED. 242 davon haben mindestens zwei Approvals. 168 macOS-relevante fehlen, weil sie nicht gemessen sind.

## Discord

Übersprungen. Der Sonnet-Worker bekam von `tabs_context_mcp` die Antwort "Browser extension is not connected", meine Prüfung von `list_connected_browsers` ergab `[]`. Computer-Use-Werkzeuge gab es in der Sitzung nicht. Verbraucht: 1 Aktion. Keine Kanäle gelesen, keine Downloads, kein Tab geöffnet.

## Eigene Fixes

- **libwebrtc macOS-Build:** Der Fehler stammt aus #21347 (027f55531a). Mein Fix e60573aee3 auf `auto/webrtc-opus-max-macos`:
  - Sonnet-Review APPROVE, `check_conventions` sauber.
  - `pr_overlap --names` meldet "open: none", `--branch` meldet "No overlap" (Lauf bis 20:31).
  - Um 20:36 kam #27621 mit identischer Zeile (Diff-Vergleich in `gate/27621.diff` gegen `gate/mine.diff`). Deshalb kein PR. Mein Branch ist lokal, nicht gepusht.
  - Messung im Kommentar: https://github.com/boykopovar/AnyPS5/pull/27621#pullrequestreview-6080902950
- **agc_driver_buffer_atomics_or_swap64:** Besitzer sind #24081 (Romans offener PR) und #21749. Kein Fix-Task, die serielle Messung lief nicht.
- **Backlog aus Lauf 8:** Außer den GPU-Tests nur Codex-Punkte, also nichts übernommen.

## Hindernisse

- RUN10.md fehlte unter orchestrator/. Roman hat den Text per Rückfrage geliefert, gespeichert ist er als RUN10.md.
- Das Konto hat auf upstream nur Leserecht. Re-runs, Labels und Merges sind damit unmöglich.
- Im gemeinsamen Hook-Verzeichnis liegen keine Git-Hooks (nur `*.sample`). Schutz bieten nur die Claude-Hooks.
- Der Hook `anyps5-maintainer-guard` blockt schon Text mit einem Merge-Befehl in einer Bash-Zeile. Das Skript für die Merge-Liste habe ich deshalb per Write angelegt. Es erzeugt nur Text.
- Beim Start waren 20 GiB frei, ein Worktree mit Submodulen braucht etwa 9 GB. Deshalb habe ich einen gemeinsamen Gate-Worktree statt einem pro PR angelegt (Abweichung von RUN10).
- Discord war nicht erreichbar.
- Systemlast 85 auf 8 Kernen (BTLEServer mit 100 % CPU) hat den Build gebremst.
- Usage-Limit von 20:16 bis 22:51. Das Budget endete um 22:07, nach der Pause habe ich keine neuen Tasks begonnen.
- #27621 kam 5 min nach dem sauberen Overlap-Lauf. Mein fertiger Fix wurde dadurch zum Duplikat.

## Braucht Roman

- Merge-Liste an einen Maintainer geben: `merge-queue.md`. Dein Konto kann nicht mergen.
- Aufräumen unter `~/Projects/anyps5-wt/`: rund 100 GB alter, nicht registrierter Ordner (audio, ci-full, diverge, exp2, fuzz, lock64, net, nkwebkit, paths, stack, stack-base).
- Für Discord in einem späteren Lauf die Claude-Erweiterung in Chrome verbinden.
- Branch `auto/webrtc-opus-max-macos` löschen, sobald #27621 gemergt ist: `git -C ~/Projects/AnyPS5 branch -D auto/webrtc-opus-max-macos`.

## Modellnutzung

- Opus: Orchestrator, kein Produktivcode.
- Sonnet, 4 Aufrufe:
  - Discord-Worker, abgebrochen
  - Builder für den webrtc-Fix, eine Zeile
  - Review mit reviewer-cross, pr-test-analyzer und ponytail-review
  - Entwurf des Kommentars für #27621
- Haiku: 0. Codex: 0.

## Zeitnutzung

| Zeitraum | Was |
|---|---|
| 19:07-19:25 | Plan, Codex-Stand, Rechte, PR-Abfragen |
| 19:20-20:11 | Gate-Basis: Submodule 6 min, Configure 7 min, Build 45 min |
| 19:53-20:16 | webrtc-Fix, Review, Overlap |
| 20:16-22:51 | Usage-Limit |
| 22:51-23:00 | Duplikatfund, Messung, Kommentar, Abschluss |

## Nächste Schritte

- **macOS-Gate mit den acht ausgewählten PRs:** Basis `local/claude-gate-base` (2b397128c2) liegt lokal. Der Worktree ist entfernt, ccache aber warm, ein Neubau dauert etwa 20 min.
- `agc_driver_buffer_atomics_or_swap64` seriell mit `until-fail:20` auf main und auf main + #24081 messen.
- Die Merge-Liste neu erzeugen: `gate/gen-queue.sh` auf eine frische GraphQL-Abfrage (`gate/fetch-rollup2.sh`).

AI-assisted: yes (Claude Code)
