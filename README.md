# healthup-recipes

Remote-Datenspeicher für die Rezeptvorschläge der iOS-App **HealthUp+**. Das Repository dient als Content Delivery Network (CDN): Die App lädt `recipes.json` und die Rezeptbilder zur Laufzeit per HTTPS.

Aktueller Stand: **450 Rezepte** (siehe `meta.json`, Schema-Version 3).

## Struktur

```
healthup-recipes/
├── README.md
├── meta.json          # schemaVersion, recipeCount, lastRecipeId, updated
├── recipes.json
└── images/
    └── rec_XXX_<name>.jpg
```

## Dauerhaft gültige Regeln

1. **Keine Doppeleinträge.** Weder gleiche oder ähnliche Titel noch gleiche Rezepte unter anderem Namen, noch doppelte IDs oder Bilder.
2. **Immer fotorealistische Bilder.** Jedes Bild ist KI-generiert, fotorealistisch, appetitlich und wird einzeln geprüft, ob es zum Gericht passt. Keine Texte, Logos, Personen oder Hände. Ein Bild, das nicht perfekt ist, wird neu erzeugt.
3. **Alle Nährwerte gelten pro Portion.** `calories`, `protein`, `carbs`, `fiber` und `fat` beziehen sich auf genau eine Portion. `servingSizeGrams` gibt das ungefähre Gewicht der fertig zubereiteten Portion in Gramm an, damit man weder zu viel noch zu wenig isst.
4. **Nährwerte nachvollziehbar berechnen.** Aus den Zutatengrammen mit Standard-Nährwerttabellen, mit Plausibilitätsprüfung (Kalorien ≈ Eiweiß×4 + Kohlenhydrate×4 + Fett×9). Ballaststoffe (`fiber`) werden separat aus den Zutatengrammen berechnet und auf ganze Gramm gerundet; sie sind nicht von `carbs` abgezogen.
5. **Schema nur erweitern.** Felder werden hinzugefügt, aber nie umbenannt oder entfernt. IDs werden nie wiederverwendet.
6. **Ernährungsform und Allergene** werden pro Rezept aus den Zutaten abgeleitet (`dietTags`, `allergens`).
7. **Qualitätscheck vor jedem Upload:** JSON gültig, alle Bild-URLs erreichbar, keine Doppelten, Nährwerte plausibel.

## Update-Prozedere

1. Neues Rezept als Objekt am Ende des Arrays in `recipes.json` anhängen (nächste freie `id`).
2. Passendes Bild unter `images/` ablegen (Dateiname: `<id>_<kurzname>.jpg`).
3. `meta.json` aktualisieren (`recipeCount`, `lastRecipeId`, `updated`).
4. Qualitätscheck durchführen und alles in einem Commit auf `main` pushen. Die App sieht neue Inhalte beim nächsten Abruf.

### Schema (pro Rezept)

| Schlüssel | Typ | Beschreibung |
|---|---|---|
| `id` | String | Eindeutige Kennung, z. B. `rec_001` |
| `title` | String | Name des Gerichts |
| `calories`, `protein`, `carbs`, `fat` | Int | Pro Portion (kcal bzw. Gramm) |
| `fiber` | Int | Ballaststoffe pro Portion in Gramm |
| `servingSizeGrams` | Int | Ungefähres Gewicht einer fertig zubereiteten Portion in Gramm |
| `prepTimeMinutes` | Int | Zubereitungsdauer |
| `suitableMealTypes` | [String] | `breakfast`, `lunch`, `dinner`, `snack` |
| `timeOfDay` | [String] | `morning` (06:00–10:30), `noon` (11:30–14:30), `afternoon` (15:00–17:30), `evening` (18:00–22:00) |
| `imageUrl` | String | Raw-URL zum Bild |
| `dietTags` | [String] | z. B. `vegetarisch`, `vegan`, `glutenfrei`, `laktosefrei`, `high-protein`, `kalorienarm`, `ballaststoffreich` (≥ 10 g Ballaststoffe pro Portion) |
| `allergens` | [String] | z. B. `gluten`, `milch`, `ei`, `fisch`, `krebstiere`, `schalenfruechte`, `erdnuss`, `soja`, `sesam`, `sellerie`, `senf`, `weichtiere`, `lupinen` |
| `ingredients` | [{`name`, `amount`}] | Zutaten mit Mengenangabe |
| `instructions` | [String] | Geordnete Zubereitungsschritte |

### Bild-Vorgaben

- JPG, Seitenverhältnis 4:3 oder 1:1
- Maximal 1200 x 900 Pixel, maximal 250 KB pro Bild
- Fotorealistisch (siehe Regel 2)

## URL-Konvention

Es werden ausschließlich Raw-Content-URLs verwendet:

- Rezepte: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/recipes.json`
- Metadaten: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/meta.json`
- Bilder: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/images/<DATEINAME>.jpg`

## Hinweise

Die Nährwerte und das Portionsgewicht sind Richtwerte, berechnet aus Standard-Nährwerttabellen, und keine medizinische Beratung. Ernährungsform-Tags und Allergene werden aus den Zutaten abgeleitet und berücksichtigen keine Spuren; bei Allergien bitte immer die Packungsangaben prüfen.

## Changelog

- 2026-10-25: Schema 2 (`servingSizeGrams`, `dietTags`, `allergens`), 150 Rezepte (rec_001–150), Regeln ergänzt.
- 2026-10-26: 50 neue Rezepte (rec_151–200), Fokus Frühstück und Mittag, viele büro- und meal-prep-tauglich.
- 2026-10-28: 200 neue Rezepte (rec_201–400) mit deutlich mehr Vielfalt: Mittelmeer und Orient, Asien, Amerika und Europa, Hausmannskost und saisonale Gerichte; insgesamt 400 Rezepte.
- 2026-10-28: Schema 3: neues Feld `fiber` (Ballaststoffe pro Portion) für alle 400 Rezepte.
- 2026-10-29: 50 neue ballaststoffreiche Rezepte (rec_401–450, je ≥ 10 g Ballaststoffe pro Portion, Tag `ballaststoffreich`); neues Allergen `lupinen`; Tag `ballaststoffreich` zusätzlich bei allen bestehenden Rezepten mit ≥ 10 g Ballaststoffen (insgesamt 164); insgesamt 450 Rezepte.
