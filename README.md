# 🦊 RingFuchsUpdate – Over-the-Air-Verbandskatalog

Statische Update-Quelle (GitHub Pages) für das RingFuchs-Ökosystem. Hier liegt der signierte Verbandskatalog
mit Disziplinen und Scheiben von **DSB, BDS, BDMP, DSU und vielen weiteren**. Die iOS-App lädt ihn im Hintergrund nach. DSB liefert
die App selbst mit.

- **Herausgeber:** RingFuchs Manager (macOS), Knopf „🌐 Katalog veröffentlichen“.
- **Abnehmer:** RingFuchs-App, `KatalogSync` (Einstellungen → Netzwerk → Verbandskatalog).
- **Adresse nach Aktivierung von Pages:** `https://db4ta.github.io/RingFuchsUpdate/v1/`

## Struktur

```
v1/
├── manifest.json        Inhaltsverzeichnis: Katalog-Revision, Pakete mit Revision, Größe, SHA-256
├── manifest.json.sig    Ed25519-Signatur über die Bytes von manifest.json (64 Byte, roh)
└── pakete/
    └── bds-r1.ringfuchs Verbandspaket im Austauschformat „ringfuchs-disziplinen“ (Disziplinen + Scheiben)
```

- **Unveränderliche Pakete:** Eine geänderte Fassung bekommt eine neue Revision im Dateinamen (`bds-r2.ringfuchs`).
  Alte Pakete bleiben liegen, damit ein Gerät, das gerade abgleicht, sie noch laden kann.
- **Unveränderliche Scheiben:** Die Geometrie einer veröffentlichten `scheiben_id` ändert sich nie. Der Manager
  sichert das über `inhalt_sha256` und `veroeffentlicht.lock.json`.
- **Prüfung in der App, in dieser Reihenfolge:**
  1. Signatur des Manifests gegen den eingebauten öffentlichen Schlüssel.
  2. Größe und SHA-256 jedes Pakets.
  3. Fachliche Prüfung wie beim Datei-Import.
  4. Atomarer Tausch der Generation.

  Schlägt etwas fehl, behält die App ihren bisherigen Stand.
- **Notbremse:** `"sync_pausiert": true` im Manifest. Apps übernehmen diese Fassung dann nicht.
- **Delta:** Die App fragt `manifest.json` mit ETag an und lädt nur Pakete, deren Prüfsumme sich geändert hat.

## Aktueller Inhalt

Muster-Katalog **r1** mit einem echten BDS-Paket:

- Disziplin `bds_25m_praezision`: „25 m Kurzwaffe Präzision“, Kurzwaffe, 20 Schuss, Kaliber 5,5–11,6 mm, 25 m.
- Scheibe `bds_kurzwaffenscheibe` (Z9): unveränderte Kopie der Referenzscheibe des Managers.
  - Karton 550 × 550 mm, Spiegel ab Ring 7.
  - Innenzehn Ø 25 mm, Ring 10 Ø 50 mm bis Ring 1 Ø 500 mm.
  - Geometrie-Hash `a0663fa6…bff5`.

⚠️ **`manifest.json.sig` ist ein Platzhalter.** Die App lehnt diesen Stand deshalb ab („Signatur ungültig“) und
arbeitet mit ihrem bisherigen Katalog weiter. Gültig wird der Katalog erst durch die erste echte Veröffentlichung
aus dem Manager. Sie ersetzt alle drei Dateien.

## GitHub Pages aktivieren

1. Repository → **Settings** → **Pages**.
2. **Build and deployment** → Source: **Deploy from a branch**.
3. Branch: **`main`**, Ordner: **`/ (root)`** → **Save**.
4. Nach etwa einer Minute ist `https://db4ta.github.io/RingFuchsUpdate/v1/manifest.json` erreichbar.

`.nojekyll` sorgt dafür, dass Pages die Dateien unverändert ausliefert (kein Jekyll-Build).

## Veröffentlichen (Ablauf)

1. Im RingFuchs Manager „🌐 Katalog veröffentlichen“. Der Manager signiert mit dem Schlüsselbund-Eintrag
   `ringfuchs-katalog-signatur` / `RingFuchs`, prüft die Signatur gegen den App-Schlüssel und schreibt nach
   `RingFuchs-Manager/veroeffentlichung/v1/`.
2. Den Inhalt dieses Ordners nach `v1/` in dieses Repository kopieren (vorhandene Pakete nicht löschen).
3. Commit und Push auf `main`. Pages aktualisiert sich automatisch.
4. `RingFuchs-Manager/veroeffentlicht.lock.json` im Manager-Repo committen, denn es ist das Gedächtnis für Revisionen
   und Scheibengeometrie.

## Lizenz

Proprietär, siehe [LICENSE](LICENSE). Copyright © 2026 RingFuchs. Alle Rechte vorbehalten.
