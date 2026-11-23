# AnyPS5 Auto-Lauf 2026-11-23

Laufzeit: 18:20 bis ca. 19:10 CET (Budget 2 h, nicht ausgeschöpft). Orchestrator: Claude Code, Opus 5.5. Upstream boykopovar/AnyPS5, Account Romannachwievor. Detail-Log: `~/Projects/anyps5-reports/auto-run-2026-11-23/log.md`.

## Angelegte Subagenten

In `.claude/agents/` gab es im Projekt keine Agenten, also habe ich alle sieben neu angelegt. Die User-Agenten `builder` und `reviewer` unter `~/.claude/agents/` sind unverändert. Draft-PR im Fork: https://github.com/Romannachwievor/AnyPS5/pull/1

| Agent | Modell | Aufgabe |
|---|---|---|
| scout | haiku | lesen, sammeln, kein Code |
| builder | sonnet | Features, Multi-File, Review-Kommentare |
| architect | opus | Priorisierung, unklare Bugs, finaler Review von Sol-Diffs |
| codex-sol | haiku-Wrapper, `cx/gpt-6-sol` high | harte Implementierung |
| codex-terra | haiku-Wrapper, `cx/gpt-5.6-terra` medium | Bugfixes mit klarer Ursache, Tests |
| codex-luna | haiku-Wrapper, `cx/gpt-6-luna` low | mechanische Änderungen |
| reviewer-cross | sonnet | Review über Kreuz |

Codex-Modelle per Testaufruf geprüft: `cx/gpt-6-sol`, `cx/gpt-5.6-terra`, `cx/gpt-6-luna`, `cx/gpt-6-astra`. Eine `gpt-6-terra` gibt es nicht, deshalb läuft Terra auf 5.6. Astra funktioniert, aber seine Stufe ist unklar, deshalb habe ich es nicht eingesetzt.

Die neuen Agenten-Dateien waren in dieser Sitzung noch nicht als Agent-Typen geladen. Ich habe die Rollen deshalb über `general-purpose`/`builder` mit gesetztem Modell und Codex direkt per `codex exec` besetzt. Ab der nächsten Sitzung sind sie direkt nutzbar.

Den Agenten-PR und diesen Bericht habe ich bewusst nur im Fork geöffnet. Upstream-AGENTS.md erlaubt keine zusätzlichen Dateien.

## Erledigt

**#22932 fix(libc): long host paths** (https://github.com/boykopovar/AnyPS5/pull/22932)
- Der Windows-CI-Job schlug seit dem Rebase fehl (`GuestKernelSys.cpp:1456`, guest errno 2).
- Ursache laut Architect, von mir am Code nachgeprüft: Vor dem Rebase lief `CheckRawGetcwdPathLimit` nur unter `#ifndef _WIN32`. #25001 hat den Guard auf main entfernt. Der host-seitige `std::filesystem::is_directory(inside)` bekam dann einen etwa 1056 Zeichen langen Pfad, und das Test-Manifest hat kein `longPathAware`.
- Fix `d6cde1fdd6`: Der Test prüft den Host-Pfad jetzt über `NativeVerbatimPath_nid_no_patch`, wie `GuestFilesystem.cpp`. Die `Require` sind aufgeteilt.
- Mac: isolierter Check 3/3. Der volle Test bricht auf macOS schon an der bekannten Zeile 1190 ab, auf Base und Branch.
- Codex Sol Review: APPROVE.
- MarcZX10s Frage zur Root-Containment beantwortet: https://github.com/boykopovar/AnyPS5/pull/22932#issuecomment-7109207120
- Windows-CI: siehe Nachtrag unten.

**#24714 fix(shader): v_exp_f32 auf M1** (https://github.com/boykopovar/AnyPS5/pull/24714)
- Helldiverr hat auf RDNA4 gemessen: Das plain `Exp2` von main ist dort bitgenau, der PR wich bei 103 von 8191 Eingaben ab. Außerdem liefen zwei `Exp2` pro Instruktion.
- Neu: ein Start-Probe `ProbeFloat32HostExp2`. Er prüft, ob plain `Exp2` für alle ganzen n in [-126, 127] exakt 2^n liefert. Wenn ja, wird das Flag `float32HostExp2` gesetzt und das Gerät nutzt die Lowering von main. Sonst gilt die range-reduzierte Form, mit nur noch einem `Exp2` pro Pfad.
- Das Flag ist im Cache-Key. Dazu kommen Env `APS5_NO_HOST_F32_EXP2`, Replay-Schalter `--f32-host-exp2` und Doku-Zeilen in `ENVIRONMENT.md` und `ARCHITECTURE.md`.
- `agc_driver_float16_nan_probe` prüft den Probe jetzt auf jedem Gerät.
- Ablauf:
  1. Sol hat entworfen.
  2. Der Architect fand einen Stride-Fehler im Probe (Shader schreibt pro Lane 64 Byte, Host las 16). Der M1-Lauf hatte ihn bestätigt: 253 von 254 „ungenau“, obwohl `exp2(0/1/2)` auf dem M1 exakt ist.
  3. Sol hat den Stride-Fehler behoben.
  4. Terra hat die übrigen Review-Punkte umgesetzt, ich habe den Diff gelesen.
- M1 mit Validation-Layer, 0 VUID: Beide SDWA-Tests sind auf Base rot und mit dem PR grün. Der Probe meldet 125 von 254 ungenau und wählt range-reduziert. Die Negativkontrolle schlägt erwartungsgemäß fehl.
- Gepusht ohne Force: Merge von main `10cc7ae711` plus `aee281b495`. PR-Text aktualisiert, Antwort an Helldiverr: https://github.com/boykopovar/AnyPS5/pull/24714#issuecomment-7109527019

## Geprüft, keine Aktion

- **#23587, #23595:** Der Scout meldete rote Conventions-Checks. Das waren alte Läufe, die aktuellen Checks sind grün.
- **#24064, #24065, #24341:** Nur abgebrochene Workflows (`Progress report`, Conventions, Labels). `gh run rerun` wird abgelehnt: „Must have admin rights to Repository“. Von unserer Seite nicht lösbar.
- **#24081, #24072, #23700, #23255:** sauber, keine offenen Fragen.
- Keine ungelösten Review-Threads in allen 11 PRs, keine Merge-Konflikte.

## Abgebrochen oder übersprungen

- **#12284**, masked_store-Flake unter Rosetta: übersprungen. Auf dem M1 0 von 5000 Läufen, nur in CI. Ein Fix wäre geraten.
- **nkwebkit_filteroperation:** übersprungen. Ein Fix liegt im Fork, aber #16510 besitzt den Getter. Kommentar 7053907560 existiert schon.
- **guest_curl_sockets**, Abbruch nur bei `-j6`: übersprungen. Braucht einen frischen Komplett-Build; build-full liegt 1327 Commits hinter main.
- Abgebrochene Tasks: keine. Fehlversuche: einer (Probe-Stride, im ersten Fixversuch behoben).

## Braucht Roman

- **#24714:** Ich habe main per Merge-Commit in den PR-Branch geholt statt zu rebasen, weil Force-Push für diesen Lauf verboten war. CONTRIBUTING erlaubt Rebase plus Force-Push. Falls die Maintainer eine lineare Historie wollen, wäre das ein Force-Push durch dich.
- **#24714:** Ob die Geräte-Weiche gewünscht ist oder das PS5-Verhalten (RDNA2) Vorrang hat, kann nur ein RDNA-Lauf zeigen. Ich habe Helldiverr gebeten, auf gfx1201 zu prüfen, ob der Probe `host (0 of 254)` meldet.
- Upstream-Workflows können wir nicht neu starten (keine Admin-Rechte). #24341 bleibt blockiert, bis #17425 und #11049 gemergt sind.

## Vorschläge für riskante oder große Tasks (nicht umgesetzt)

- **#18785**, macOS speichert das ganze Linear-Target nach jedem Draw (PPSA02070): Ursache ist im Issue beschrieben (`FillClear` setzt `pendingFromOthers`, `MarkDrawn` liefert false). Ein Perf-PR braucht `tools/trace_digest.py`-Zahlen von main und Änderung aus derselben Szene. Ohne Titel nicht machbar.
- **#20118**, zweites Label-Ordered-Flush-Ergebnis, und **#24728**, SuperTuxKart-Roundtrip pro Draw: GPU-lastig und groß.

## Modellnutzung pro Task

| Task | Worker | Modell | Effort | Grund |
|---|---|---|---|---|
| PR-Scan | scout | haiku | - | lesen, sammeln |
| Repo-Scan | scout | haiku | - | lesen, sammeln |
| #22932 Ursache | architect | opus | - | unklarer Cross-Plattform-Bug ohne Windows-Rechner |
| #22932 Fix | Orchestrator | opus | - | 7 Zeilen, Briefing wäre länger gewesen |
| #22932 Review | codex-sol | gpt-6-sol | high | Claude-Diff, Review über Kreuz |
| #24714 Entwurf | codex-sol | gpt-6-sol | high | Multi-File-Feature nach float16HostNans-Muster |
| #24714 Testbaum | builder | sonnet | - | Build- und Test-Plumbing |
| #24714 Review | architect | opus | - | Pflicht-Review für Sol-Diffs |
| #24714 Probe-Fix | codex-sol | gpt-6-sol | high | Fixversuch 1 |
| #24714 Restpunkte | codex-terra | gpt-5.6-terra | medium | Tests, Doku, kleine Fixes |
| #24714 Terra-Review | Orchestrator | opus | - | Claude prüft Codex |

Luna kam nicht zum Einsatz, es gab keine rein mechanische Aufgabe.

## Nächste Schritte

1. CI von #22932 (Windows) und #24714 beobachten, auf Reviews antworten.
2. Bei RDNA-Rückmeldung zu #24714 die Probe-Ausgabe im PR nachtragen.
3. build-full auf aktuellen main bringen (main plus #17425 plus #11049) und eine neue macOS-Baseline ziehen. Dann ist guest_curl_sockets prüfbar.
4. Die neuen Projekt-Agenten in `.claude/agents/` liegen nur auf dem Fork-Branch `auto/agents-setup`. Falls sie lokal dauerhaft gelten sollen, ins lokale `.claude/` übernehmen.

AI-assisted: yes (Claude Code)
