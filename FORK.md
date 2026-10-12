# Fork: OleRaehcke/Bonfire-JellyProfiles

Eigener Fork von [AHouseOfBards/Bonfire-JellyProfiles](https://github.com/AHouseOfBards/Bonfire-JellyProfiles).
Jellyfin installiert und aktualisiert das Plugin ausschließlich aus diesem Fork.

## In Jellyfin einrichten

1. **Dashboard → Plugins → Repositories**: das Original-Repo
   (`ahouseofbards.github.io/...` und ggf. das Beta-Repo) **entfernen**.
   Beide verwenden dieselbe Plugin-GUID – bleibt das Original drin, aktualisiert Jellyfin
   von dort.
2. **＋** und diese URL eintragen:
   ```
   https://raw.githubusercontent.com/OleRaehcke/Bonfire-JellyProfiles/main/fork-manifest.json
   ```
3. **Plugins → Katalog → Bonfire** → neueste Version installieren → Jellyfin neu starten.
   Einstellungen und Profile bleiben erhalten (gleiche GUID, gleiche Konfiguration).

## Aufbau

| Datei | Zweck |
|---|---|
| `fork-manifest.json` | Plugin-Repository für Jellyfin. Wird nur von der Action geschrieben. |
| `.github/workflows/fork-release.yml` | Baut bei Tag `fork-v*`, testet, veröffentlicht Release + Manifest-Eintrag. |
| `.github/workflows/upstream-watch.yml` | Prüft täglich das Original und öffnet ein Issue, wenn es Neues gibt. Merged nie selbst. |
| `manifest.json`, `release.yml` | Gehören dem Original – **nicht anfassen**, damit Merges konfliktfrei bleiben. `release.yml` ist im Fork deaktiviert. |

## Eigene Version veröffentlichen

```bash
git tag -a fork-v1.6.4.2 -m "Was sich geändert hat (wird zum Changelog in Jellyfin)"
git push origin fork-v1.6.4.2      # nie `git push --tags`!
```

Versionsschema: Basis des Originals + vierte Stelle hochzählen
(`1.6.4` → `1.6.4.1`, `1.6.4.2`, …). Ist die Original-Version schon vierstellig
(z. B. `1.6.3.5`), dann `1.6.3.501`, `1.6.3.502`, … Die Action lehnt Versionen ab, die nicht
höher als die letzte veröffentlichte sind, und Tags auf Commits, die nicht auf `main` liegen.

## Updates vom Original übernehmen

```bash
git fetch upstream
git log --oneline main..upstream/main          # was ist neu?
git diff main...upstream/main                  # Code lesen (oder Claude reviewen lassen)
git merge upstream/main                        # nur wenn OK
git push origin main                           # CI läuft
git tag -a fork-v1.6.5.1 -m "Upstream 1.6.5 übernommen" && git push origin fork-v1.6.5.1
```

Besonders genau hinschauen bei: neuen Netzwerkaufrufen (`HttpClient`, `fetch`, URLs),
Änderungen an `.github/workflows/`, `.claude/` und `tests/run.sh` (läuft in CI und
lokal), sowie Authentifizierung (`Auth/`, `Controllers/`).

Schutzmaßnahmen im lokalen Clone: `upstream` hat keine Push-URL (`DISABLED`) und holt
keine Tags (die Original-Tags `v*` würden sonst beim Push upstreams Release-Workflow
auslösen).
