# Voice assistant flow — Uitleg

Deze n8n-workflow doet twee dingen:

## 1) Interview / Survey
Vragen ophalen → gesprek/antwoorden genereren → antwoorden wegschrijven naar Google Sheets (**kolomkoppen = vragen, elke respondent = 1 rij**).

## 2) Analytics / Charts
Leest alle antwoorden uit de sheet → telt frequenties per vraag/kolom → maakt **bar charts** met QuickChart (PNG) → uploadt elke chart naar een gekozen **Google Drive**-map.

---

## Blokdiagram (hoogover)

```
[Webhook]
   → [Code (pre)] → [Survey-vragen ophalen] → [Survey/Interview (LLM + memory + parser)]
   → [Code (normalize antwoorden)] → [Append row in Google Sheets]

   → [Get row(s) in Google Sheets] → [Code (chart builder)]
   → [HTTP POST → quickchart.io/chart] → (binary PNG)
   → [Upload File (Google Drive)]
```

---

## Benodigdheden

- **n8n** (self-host of cloud)
- **Google Sheets & Google Drive credentials** in n8n (OAuth / Service Account)
- **QuickChart** vereist geen key; de flow gebruikt een HTTP-request
- Een spreadsheet **Interview results** met in **rij 1** de vragen als kolomtitels

> Tip: werk met één tab, bv. `Sheet1`, waarin rij 1 je **vaste kolomkoppen** bevat; elke nieuwe respondent wordt als **nieuwe rij** toegevoegd.

---

## Deel 1 — Interview / Survey

### 1) Webhook
- **Doel:** startpunt van de flow (manueel, via CI, of vanaf je app/agent).
- **Payload:** mag leeg zijn; de flow haalt zelf data/vragen op.

### 2) Code (pre-processing)
- **Doel:** evt. normaliseren van input, sessie-ID’s, timestamp, etc.
- **Output:** intern gebruikt door de volgende node(s).

### 3) Survey vragen ophalen
- **Type:** Code of HTTP (afhankelijk van je implementatie).
- **Doel:** lijst met vragen aanleveren.
- **Opmerking:** Vragen worden later ook gebruikt als **kolomkoppen** in de sheet.

### 4) Survey / Interview (LLM + Memory + Parser)
- **Type:** OpenAI/Claude Chat node + Memory + Unstructured Output Parser.
- **Doel:** dialoog voeren, antwoorden structureren naar `{ vraag, antwoord }[]`.
- **Output:** array van QA-paren.

### 5) Code (normalize antwoorden)
- **Doel:** van `{ vraag, antwoord }[]` naar **één rij** met als keys **exact** de vraagteksten (die in rij 1 staan) en als values de gegeven antwoorden.
- **Let op multi-select:** antwoorden kunnen met `;` gescheiden zijn.

### 6) Append row in Google Sheets
- **Document:** `Interview results`  
- **Sheet:** `Sheet1`  
- **Mapping Column Mode:** *Map Each Column Manually*  
- **Values to Send:** enkel **antwoorden** (geen vragen) → **1 rij**  
- **Resultaat:** elke respondent verschijnt als **nieuwe rij** onder de header in rij 1.

---

## Deel 2 — Analytics / Charts

### 7) Get row(s) in Google Sheets
- **Document:** `Interview results`  
- **Sheet:** `Sheet1`  
- **Return All:** **aan**  
- *(optioneel)* **Range:** `A1:ZZ` (ruim)

**Output:** alle rijen met kolomnamen als keys.

### 8) Code (chart builder)
**Doel:** voor **elke kolom** (vraag) → frequentietelling maken en een **Chart.js** payload opbouwen.

**Belangrijkste logica:**
- Multi-select antwoorden worden gesplitst op `;`.
- Numerieke/slider-achtige vragen (0–100) worden automatisch in **buckets** gezet: `0–20, 21–40, …, 81–100`.
- **Y-as altijd vanaf 0** (zowel v4- als v2-syntax gezet).
- **Uitvoer:** één item **per vraag/kolom** met:
  ```json
  {
    "width": 1200,
    "height": 600,
    "backgroundColor": "white",
    "filename": "<slug-van-vraag>.png",
    "chart": { "... Chart.js config ..." }
  }
  ```

> **Top-N per chart?** In de code staat een variabele `TOP_N` om bijv. top-10 categorieën te tonen.

### 9) HTTP Request → QuickChart
- **Method:** `POST`
- **URL:** `https://quickchart.io/chart`
- **Send Body:** **aan**, **Content Type:** JSON
- **Body parameters:**
  - `chart` → `{{$json.chart}}`
  - `width` → `{{$json.width}}`
  - `height` → `{{$json.height}}`
  - `backgroundColor` → `{{$json.backgroundColor}}`
  - `format` → `png`
  - `version` → `4` *(forceert Chart.js v4; nodig voor de y-as opties)*
- **Response Format:** **File**
- **Put Output in Field:** `data`

**Resultaat:** per item een **binary PNG** onder sleutel `data`.

### 10) Upload File (Google Drive)
- **Binary Property:** `data`  *(of eerst via “Move Binary Data” naar `chart` + bestandsnaam)*
- **File Name:** `{{$json.filename}}`
- **Parents:** de map-ID van je Drive-doelmap

> **Optioneel:** i.p.v. losse PNG’s eerst met **Archive → Compress** alle binaries bundelen tot één **ZIP**, en die uploaden.

---

## Belangrijke keuzes & conventies

### Kolomkoppen = vraagteksten
Zorg dat de koppen in rij 1 **exact** matchen met de vragen die uit de interviewstap komen.

### Multi-select delimiter
Standaard wordt `;` gebruikt. Pas dit in de Code-node aan als je een andere scheiding gebruikt.

### Y-as start op 0
In de chart-payload worden v4 **én** v2 schaalopties gezet. In de HTTP-node staat `version: 4`.

### Slider/percent-vragen
De Code-node detecteert numerieke kolommen (≥ 80% numeriek) of kolomnamen die `slider` of `0–100` bevatten, en bucketiseert automatisch.

---

## Veelvoorkomende fouten

### Charts starten niet bij 0
Check dat in de HTTP-node de **Body parameter** `version: 4` is meegegeven en dat in de payload `options.scales.y.min = 0` staat.

### Rij komt op verkeerde plek in de sheet
Controleer dat je **Append Row** doet (en geen Update), en dat je **header** in **rij 1** staat, zonder lege rijen ertussen.

### Upload naar Drive faalt
Meestal credentials of map-ID. Test met een simpele Upload node en kijk in het n8n-execution log.

---

## Variaties / uitbreidingen

### Horizontale bars
Voeg `indexAxis: 'y'` toe aan `chart.options`.

### Top-N per vraag
Zet `TOP_N` in de Code-node (bijv. `const TOP_N = 10;`).

### Google Slides deck genereren
Na QuickChart: maak een presentatie aan en voeg elke PNG als slide-afbeelding toe.
