# healthup-recipes

Remote-Datenspeicher für die Rezeptvorschläge der iOS-App **HealthUp+**. Das Repository dient als Content Delivery Network (CDN): Die App lädt `recipes.json` und die Rezeptbilder zur Laufzeit per HTTPS.

## Struktur

```
healthup-recipes/
├── README.md
├── recipes.json
└── images/
    └── rec_XXX_<name>.jpg
```

## Update-Prozedere

1. Neues Rezept als Objekt am Ende des Arrays in `recipes.json` anhängen (nächste freie `id`, z. B. `rec_011`).
2. Passendes Bild unter `images/` ablegen (Dateiname: `<id>_<kurzname>.jpg`).
3. Beides in einem Commit auf `main` pushen. Die App sieht neue Inhalte beim nächsten Abruf.

### Schema (pro Rezept)

| Schlüssel | Typ | Beschreibung |
|---|---|---|
| `id` | String | Eindeutige Kennung, z. B. `rec_001` |
| `title` | String | Name des Gerichts |
| `calories`, `protein`, `carbs`, `fat` | Int | Pro Portion (kcal bzw. Gramm) |
| `prepTimeMinutes` | Int | Zubereitungsdauer |
| `suitableMealTypes` | [String] | `breakfast`, `lunch`, `dinner`, `snack` |
| `timeOfDay` | [String] | `morning` (06:00–10:30), `noon` (11:30–14:30), `afternoon` (15:00–17:30), `evening` (18:00–22:00) |
| `imageUrl` | String | Raw-URL zum Bild |
| `ingredients` | [{`name`, `amount`}] | Zutaten mit Mengenangabe |
| `instructions` | [String] | Geordnete Zubereitungsschritte |

### Bild-Vorgaben

- JPG oder WebP, Seitenverhältnis 4:3 oder 1:1
- Maximal 1200 x 900 Pixel, maximal 250 KB pro Bild

## URL-Konvention

Es werden ausschließlich Raw-Content-URLs verwendet:

- Rezepte: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/recipes.json`
- Bilder: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/images/<DATEINAME>.jpg`

Hinweis: Die Nährwerte sind Richtwerte, berechnet aus Standard-Nährwerttabellen.
