# healthup-recipes

Remote-Datenspeicher für die Rezeptvorschläge der iOS-App **HealthUp+**. Das Repository dient als Content Delivery Network (CDN): Die App lädt `recipes.json` und die Rezeptbilder zur Laufzeit per HTTPS.

Aktueller Stand: **3600 Rezepte** (siehe `meta.json`, Schema-Version 3).

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
7. **Jedes Rezept hat ein eigenes Bild aus diesem Repo** (`images/`). Keine Fremdquellen wie Unsplash.
8. **Qualitätscheck vor jedem Upload:** JSON gültig, alle Bild-URLs erreichbar, keine Doppelten, Nährwerte plausibel.
9. **Kein Schweinefleisch.** Keine Rezepte mit Schweinefleisch oder daraus hergestellten Produkten (z. B. Schinken, Speck, Chorizo, Salami, Schweinehack, Schmalz, Gelatine). Wurst und Schinken nur aus Geflügel oder Rind, entsprechend gekennzeichnet.

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
| `calories`, `protein`, `fat` | Int | Pro Portion (kcal bzw. Gramm) |
| `carbs` | Int | Kohlenhydrate pro Portion in Gramm, **netto, ohne Ballaststoffe** (EU/BLS-Konvention) |
| `fiber` | Int | Ballaststoffe pro Portion in Gramm, als eigene Zeile. Nicht von `carbs` abgezogen, deshalb kann `fiber` > `carbs` vorkommen (Beispiel `rec_017`: carbs 15, fiber 17) |
| `servingSizeGrams` | Int | Ungefähres Gewicht einer fertig zubereiteten Portion in Gramm. Die App (`RecipeService`) liest dieses Feld und führt es intern als `portionGrams`. |
| `prepTimeMinutes` | Int | Zubereitungsdauer |
| `suitableMealTypes` | [String] | `breakfast`, `lunch`, `dinner`, `snack` |
| `timeOfDay` | [String] | `morning` (06:00–10:30), `noon` (11:30–14:30), `afternoon` (15:00–17:30), `evening` (18:00–22:00) |
| `imageUrl` | String | Raw-URL zum Bild |
| `dietTags` | [String] | z. B. `vegetarisch`, `vegan`, `glutenfrei`, `laktosefrei`, `high-protein`, `kalorienarm`, `ballaststoffreich` (≥ 10 g Ballaststoffe pro Portion), sowie die neuen Tags `keto`, `low-carb`, `mediterran`, `dash`, `paleo`, `pescetarisch`, `low-fodmap`, `low-gi`, `volumenreich`, `quick-win` (Kriterien siehe unten) |
| `allergens` | [String] | z. B. `gluten`, `milch`, `ei`, `fisch`, `krebstiere`, `schalenfruechte`, `erdnuss`, `soja`, `sesam`, `sellerie`, `senf`, `weichtiere`, `lupinen`, `sulfite` (Schwefeldioxid/Sulfite, konservativ aus typisch geschwefelten Zutaten abgeleitet, z. B. Essig, Senf, Trockenobst, Eingelegtes, Garnelen) |
| `ingredients` | [{`name`, `amount`}] | Zutaten mit Mengenangabe |
| `instructions` | [String] | Geordnete Zubereitungsschritte |

### Bild-Vorgaben

- JPG, Seitenverhältnis 4:3 oder 1:1
- Maximal 1200 x 900 Pixel, maximal 250 KB pro Bild
- Ab 2026-11-10 werden neue Bilder einheitlich auf **1152 × 864 Pixel** hochskaliert (Lanczos) und auf höchstens 250 KB JPEG-komprimiert.
- Fotorealistisch (siehe Regel 2)

## URL-Konvention

Es werden ausschließlich Raw-Content-URLs verwendet:

- Rezepte: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/recipes.json`
- Metadaten: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/meta.json`
- Bilder: `https://raw.githubusercontent.com/HealthUp1123/healthup-recipes/main/images/<DATEINAME>.jpg`

## Hinweise

Die Nährwerte und das Portionsgewicht sind Richtwerte, berechnet aus Standard-Nährwerttabellen, und keine medizinische Beratung. Ernährungsform-Tags und Allergene werden aus den Zutaten abgeleitet und berücksichtigen keine Spuren; bei Allergien bitte immer die Packungsangaben prüfen.


## Neue dietTags (ab 2026-11-10)

Die folgenden Tags werden konservativ aus den Zutaten und den Nährwerten **pro Portion** abgeleitet. Ist eine Zutat nicht sicher einzuordnen, wird der Tag nicht vergeben. Die Tags beschreiben nur die Zusammensetzung, sie sind keine medizinische Empfehlung oder Aussage zu Gesundheitseffekten.

| Tag | Vergabe (kurz) |
|---|---|
| `keto` | Höchstens 10 g Netto-Kohlenhydrate, mindestens 65 % der Kalorien aus Fett, keine zugesetzten Zucker (Honig, Sirup u. Ä.) |
| `low-carb` | Höchstens 20 g Netto-Kohlenhydrate |
| `mediterran` | Olivenöl, kein rotes oder verarbeitetes Fleisch, keine typischen „schlechten“ Fette (Butter, Sahne, Kokosöl u. Ä.); Käse höchstens 40 g; gesättigte Fettsäuren höchstens 10 % der Kalorien; zugesetzter Zucker höchstens 10 g; Gemüse, Obst, Hülsenfrüchte, Vollkorn, Nüsse/Samen, Fisch, Olivenöl und Kräuter machen mindestens die Hälfte der Feststoffe aus; raffiniertes Getreide höchstens 60 g |
| `dash` | Alle Zutaten mit bekanntem Natriumwert, kein unquantifiziertes Salz; höchstens 600 mg Natrium; mindestens 120 g Gemüse/Obst; kein rotes oder verarbeitetes Fleisch, keine geräucherten/gesalzenen Fische, keine typischen „schlechten“ Fette; fettreiche Milchprodukte höchstens 20 g; gesättigte Fettsäuren höchstens 8 % der Kalorien; zugesetzter Zucker höchstens 10 g |
| `paleo` | Nur Zutaten aus einer Whitelist (Fleisch, Geflügel, Fisch, Ei, Gemüse, Obst, Nüsse/Samen und bestimmte Öle); keine Getreide, Hülsenfrüchte, Milchprodukte, verarbeiteten Fleischwaren oder zugesetzten Zucker |
| `pescetarisch` | Enthält Fisch oder Meeresfrüchte und kein Fleisch |
| `low-fodmap` | Alle Zutaten stehen auf einer Green-Serve-Liste (Monash) und bleiben unter der jeweiligen Höchstmenge pro Portion; typische High-FODMAP-Namen (Zwiebel, Knoblauch, Weizen, Honig u. Ä.) schließen den Tag aus |
| `low-gi` | Kohlenhydratquellen mit typischem glykämischen Index ≤ 55 (oder unter 5 g Kohlenhydratbeitrag); mindestens 15 g Netto-Kohlenhydrate und 5 g Ballaststoffe; kein zugesetzter Zucker und keine typischen High-GI-Früchte |
| `volumenreich` | Sättigende, kalorienarme Portion: mindestens 350 g, höchstens 400 kcal und höchstens 80 kcal/100 g, dazu mindestens 20 g Eiweiß oder 8 g Ballaststoffe; gilt nicht für Getränke (Smoothie, Shake, Saft u. Ä.) |
| `quick-win` (ab 2026-11-12) | Schnelles Diät-Frühstück als einfacher Teller oder Schale: als Frühstück geeignet, Zubereitung höchstens 10 Minuten, mindestens 15 g Eiweiß, höchstens 450 kcal und mindestens 100 g Gemüse/Obst pro Portion (Avocado, Kartoffel, Oliven, Kapern zählen nicht mit); wird immer als letzter Tag angehängt. Bisher nur bei rec_3527–3576 vergeben, Bestandsrezepte unverändert |

`vegan` plus `high-protein` und `kalorienarm` plus `volumenreich` kennzeichnen die Gruppen „vegan high-protein“ bzw. „kalorienarm sättigend“. Die Tags wurden auch auf die 700 Bestandsrezepte angewendet, dort ändert sich ausschließlich das Feld `dietTags`.

## Changelog

- 2026-10-25: Schema 2 (`servingSizeGrams`, `dietTags`, `allergens`), 150 Rezepte (rec_001–150), Regeln ergänzt.
- 2026-10-26: 50 neue Rezepte (rec_151–200), Fokus Frühstück und Mittag, viele büro- und meal-prep-tauglich.
- 2026-10-28: 200 neue Rezepte (rec_201–400) mit deutlich mehr Vielfalt: Mittelmeer und Orient, Asien, Amerika und Europa, Hausmannskost und saisonale Gerichte; insgesamt 400 Rezepte.
- 2026-10-28: Schema 3: neues Feld `fiber` (Ballaststoffe pro Portion) für alle 400 Rezepte.
- 2026-10-29: 50 neue ballaststoffreiche Rezepte (rec_401–450, je ≥ 10 g Ballaststoffe pro Portion, Tag `ballaststoffreich`); neues Allergen `lupinen`; Tag `ballaststoffreich` zusätzlich bei allen bestehenden Rezepten mit ≥ 10 g Ballaststoffen (insgesamt 164); insgesamt 450 Rezepte.
- 2026-11-01: Neues Allergen `sulfite` (95 Rezepte, konservativ aus den Zutaten abgeleitet); Kohlenhydrat-Konvention dokumentiert (`carbs` netto ohne Ballaststoffe, `fiber` separat); Hinweis zum Portionsfeld (`servingSizeGrams` = intern `portionGrams`); neue Regel: jedes Rezept hat ein eigenes Bild aus `images/`. Korrektur: rec_376 Allergen `senf` ergänzt (Senf war in den Zutaten).
- 2026-11-01: 50 vegane Rezepte rec_451–500 (20 Haupt, 15 Frühstück, 15 Snacks), eigene Bilder, en/tr ergänzt; insgesamt 500 Rezepte.
- 2026-11-01: Korrekturen nach Nachprüfung: Allergen `sulfite` bei 5 weiteren Rezepten ergänzt (rec_082, 093, 136, 306, 444); fehlende Zutaten aus der Zubereitung ergänzt (rec_082 Senf und Zitronensaft, Allergen `senf`; rec_093 Balsamico; rec_136, 306, 444 Essig), Nährwerte unverändert (je ≤ 5 kcal); rec_094 Tag `vegetarisch` entfernt (Gelatine); rec_268 Currypaste als vegan ohne Garnelenpaste gekennzeichnet; Parmesan in 10 vegetarischen Rezepten (rec_031, 053, 102, 119, 149, 192, 212, 219, 220, 426) durch „vegetarischer Hartkäse (Parmesan-Art)“ ersetzt, auch in vier Titeln; en/tr angepasst.
- 2026-11-05: 50 neue Rezepte rec_501–550 für mehr Abwechslung bei den Vorschlägen: 15 Frühstück, 12 Snacks (meal-prep-tauglich, 6 aus dem Airfryer), 15 schnelle Hauptgerichte (≤ 30 Min., u. a. Fisch, Meeresfrüchte, Lamm, Wild, Ente, Hack und Hülsenfrüchte aus persischer, koreanischer, vietnamesischer, mexikanischer, peruanischer, äthiopischer, skandinavischer und osteuropäischer Küche) und 8 Trendrezepte (Protein-Desserts, Shakes, Eis, Fibermaxxing); 18 mit Fleisch oder Fisch; eigene Bilder, en/tr ergänzt; insgesamt 550 Rezepte.
- 2026-11-09: Neue Regel 9 „Kein Schweinefleisch“: 26 Rezepte mit Schweinefleisch entfernt (rec_206, 214, 216, 262, 267, 275, 280, 281, 284, 286, 300, 301, 307, 315, 321, 325, 329, 336, 342, 343, 351, 360, 371, 373, 397, 442; IDs werden nicht wiederverwendet), en/tr entsprechend bereinigt; rec_094 Gelatine durch Agar-Agar ersetzt (jetzt wieder `vegetarisch`, Nährwerte angepasst). 176 neue Rezepte rec_551–726: 76 türkische Gerichte (49 vegetarisch, Titel „Türkischer Originalname (Deutsch)“) und 100 internationale, überwiegend vegetarische Gerichte; insgesamt 149 vegetarisch (davon 46 vegan), 27 mit Fleisch oder Fisch (ohne Schwein); eigene Bilder, en/tr ergänzt; insgesamt 700 Rezepte.
- 2026-11-10: 500 neue Rezepte rec_727–1226, ausgerichtet auf verschiedene Ernährungsformen (60 keto, 60 low-carb, 70 mediterran, 50 DASH, 40 paleo, 50 pescetarisch, 40 low-FODMAP, 50 low-GI, 40 vegan high-protein, 40 kalorienarm sättigend); neue dietTags nach den oben dokumentierten, konservativen Zutatenregeln, auch rückwirkend auf die 700 Bestandsrezepte (nur `dietTags`: u. a. 119 mediterran, 91 low-GI, 90 pescetarisch, 64 volumenreich, 51 low-carb, 18 DASH, 18 low-FODMAP, 8 paleo); eigene Bilder 1152×864 (Lanczos-Hochskalierung, höchstens 250 KB), en/tr ergänzt; insgesamt 1200 Rezepte.
- 2026-11-10: 500 neue Rezepte rec_1227–1726 (internationale Küche und Hausmannskost, Airfryer, Protein-Desserts und Gerichte für keto, paleo, low-FODMAP und DASH, wo die Zutaten das ehrlich hergeben); Primärtags u. a. 77 mediterran, 63 pescetarisch, 50 vegan high-protein, 48 low-GI, 39 DASH, 26 volumenreich, 21 low-carb, 17 low-FODMAP, 12 keto, 10 paleo; 329 Hauptgerichte, 63 Frühstück, 54 Snacks, 54 Desserts; 280 vegetarisch (davon 117 vegan); eigene Bilder 1152×864 (Lanczos-Hochskalierung, höchstens 250 KB), en/tr ergänzt; Allergen `sulfite` bei den neuen Rezepten aus den Zutaten abgeleitet; insgesamt 1700 Rezepte.
- 2026-11-11: 1800 neue Rezepte rec_1727–3526 für den Diät-Ernährungsberater (mediterrane Küche, Low-Carb und Keto, High-Protein vegan und vegetarisch, Fisch, Low-GI und ballaststoffreich, DASH und volumenreich, Paleo und Low-FODMAP, türkisch und nahöstlich, Airfryer und schnelle Gerichte, Frühstück und Snacks, Ost- und Südostasien, DACH-Hausmannskost, lateinamerikanische und afrikanische Küche, Suppen, Salate und Meal-Prep, Desserts und Eis, Protein-Bowls, saisonale Geflügel- und Wildgerichte ohne Wildschwein, dazu Lückenfüller); kein Schweinefleisch; eigene Bilder 1152×864 (höchstens 250 KB), en/tr ergänzt; Allergen `sulfite` bei den neuen Rezepten aus den Zutaten abgeleitet; Korrekturen: Titel und Bildname rec_2858 (Kalbsgeschnetzeltes mit Rösti und Feldsalat), rec_3227 von Hirse auf Buchweizen umgestellt, Zubereitungsschritte rec_2355 ergänzt; insgesamt 3500 Rezepte.
- 2026-11-12: rec_1472 Doro Wat: Tag low-gi entfernt
- 2026-11-14: 50 neue Rezepte rec_3577–3626 für den Diät-Ernährungsberater: eiweiß- und ballaststoffreiche Mittag- und Abendessen (Hähnchen, Pute, Rind, Fisch und Garnelen, Hülsenfrüchte, Tofu und Tempeh; 10 aus dem Airfryer; 370–580 kcal, mindestens 31 g Eiweiß und 10 g Ballaststoffe pro Portion); alle `high-protein` und `ballaststoffreich`, dazu u. a. 35 low-GI, 18 mediterran, 9 pescetarisch; 18 vegetarisch (davon 10 vegan); kein Schweinefleisch; eigene Bilder 1152×864 (höchstens 250 KB), en/tr ergänzt; Allergen `sulfite` aus den Zutaten abgeleitet (rec_3582, 3602 Senf; rec_3604, 3608 Garnelen); kein Schemawechsel; insgesamt 3600 Rezepte.
