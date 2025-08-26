# Voice assistant flow — Code snippets

Deze file bundelt de belangrijkste n8n Code/HTTP-snippets uit de workflow, met per snippet een ultrakorte uitleg (één zin).

---

## 1) Code1 — Structured data uit het interview halen

```js
// Pak de structuredData uit de webhook payload
const sd = $json.body?.message?.analysis?.structuredData || {};

// Output: één item, één JSON-object met alleen de structured data + meta
return [{
  json: {
    session_id: $json.body?.message?.callId || null,
    timestamp: $json.body?.timestamp || new Date().toISOString(),
    structured_data: sd
  }
}];
```
**Uitleg:** Haalt `structuredData` + wat meta (call ID, timestamp) uit de webhook en geeft dat als net JSON-object door.

---

## 2) Code2 — Structured data normaliseren + string-versie meenemen

```js
// Mode: Run Once per Item
// Pak eerst het reeds gebouwde object, en val anders terug op de originele webhook-plek
const sd =
  $json.structured_data ??
  $json.body?.message?.analysis?.structuredData ??
  {};

// Bewaar én als object én als mooi geformatteerde string
return [{
  json: {
    ...$json, // behoud bestaande velden
    structured_data: sd,
    structured_data_str: JSON.stringify(sd, null, 2),
  }
}];
```
**Uitleg:** Zet de structured data zowel als object als prettige JSON-string op het item voor latere prompting/validatie.

---

## 3) Code3 — Array `{ question, user_answer[] }` → object `{ vraag: [antwoorden] }`

```js
// Pak de eerste item uit de array
const data = items[0].json;

// Haal de lijst met vragen en antwoorden op
const survey = data.output?.survey_data || [];

// Zet om naar één object: { vraag: [antwoorden] }
const result = {};
survey.forEach(entry => {
  const question = entry.question;
  const answers = entry.user_answer;
  if (question && answers) {
    result[question] = answers;
  }
});

// Return als één JSON-item
return [{ json: result }];
```
**Uitleg:** Vouwt een survey-array met QA’s samen tot één makkelijk te mappen object per vraag.

---

## 4) Code5 — Robuust parseren van rommelige JSON in `output` naar nette items

```js
// 1) Pak de ruwe waarde uit het inkomende item
const raw = $json.output;

// 2) Als het al een object is, gebruik het direct; anders: schoon en parse
let obj;
if (typeof raw === 'object' && raw !== null) {
  obj = raw;
} else if (typeof raw === 'string') {
  // haal alles vóór de eerste { en na de laatste } weg (veilig bij rommel ervoor/erna)
  const start = raw.indexOf('{');
  const end = raw.lastIndexOf('}');
  if (start === -1 || end === -1) {
    throw new Error('Kon geen JSON-object vinden in "output".');
  }
  const jsonSlice = raw.slice(start, end + 1);

  // maak van een dubbel-geëscape-tekenreeks weer normale JSON
  const normalized = jsonSlice
    .replace(/\r?\n/g, ' ')
    .replace(/\t/g, ' ')
    .replace(/\\\"/g, '"'); // \" -> "

  obj = JSON.parse(normalized);
} else {
  throw new Error('Onbekend formaat voor "output".');
}

// 3) Zet het object met "Vraag": "Antwoord" om naar n8n-items
const itemsOut = Object.entries(obj).map(([vraag, antwoord]) => ({
  json: { vraag, antwoord }
}));

return itemsOut;
```
**Uitleg:** Snijdt een JSON-object uit een rommelige string, de-escapet het en zet elke `vraag: antwoord` om naar een los n8n-item.

---

## 5) Code4 — Losse `{vraag, antwoord}`-items → één rij voor Google Sheets

```js
// items = records met {vraag, antwoord}
const list = items.map(i => i.json);

const row = {};
for (const item of list) {
  const key = String(item.vraag || '').trim();  // kolomkop exact gelijk
  let val = item.antwoord;

  // Lijsten samenvoegen
  if (Array.isArray(val)) val = val.join('; ');
  if (val === null || val === undefined) val = '';

  row[key] = String(val);
}

// Lever 1 item terug met alle kolommen
return [{ json: row }];
```
**Uitleg:** Combineert alle antwoorden tot één object waarvan de keys exact de kolomkoppen zijn en de values kant-en-klare celwaarden (multi-select met `;`).

---

## 6) Code (Chart Builder) — Frequenties per kolom tellen en Chart.js payload maken

```js
/**
 * INPUT  : items[] van Google Sheets → Get row(s) in sheet
 * OUTPUT : meerdere items, één per kolom:
 *          { width, height, backgroundColor, chart, filename }
 */

const WIDTH  = 1200;
const HEIGHT = 600;
const SPLIT_CHAR = ';';
const IGNORE_COLS = new Set(['row_number','RespondentID','Timestamp','ID','']);
const TOP_N = null;                   // bv. 10 om Top-10 te tonen
const ROTATE_X_TICKS = 35;
const SLIDER_BUCKETS = [[0,20],[21,40],[41,60],[61,80],[81,100]];
const NUMERIC_RATIO_THRESHOLD = 0.8;

// --- Helpers ---
const rows = items.map(i => i.json);
if (!rows.length) return [{ json: { error: 'Geen rijen opgehaald uit Google Sheets.' } }];

const headerNames = Object.keys(rows[0]);   // kolommen uit je sheet
const dataRows = rows;                      // Sheets-node levert al data zonder header-rij

const isBlank = v => v === undefined || v === null || String(v).trim() === '';
const slugify = s => String(s).toLowerCase().replace(/[^\w]+/g,'_').replace(/^_+|_+$/g,'').slice(0,80);

function makeCountsFromValues(values, colName){
  const nums = values.map(v => Number(v));
  const numericShare = nums.filter(n => !isNaN(n)).length / values.length;
  const looksLikeSlider = /slider/i.test(colName) || /0-100/.test(colName) || numericShare >= NUMERIC_RATIO_THRESHOLD;

  const counts = {};
  if (looksLikeSlider){
    for (const n of nums){
      if (isNaN(n)) continue;
      let label = null;
      for (const [a,b] of SLIDER_BUCKETS){ if (n >= a && n <= b){ label = `${a}-${b}`; break; } }
      if (!label) label = 'Overig';
      counts[label] = (counts[label] || 0) + 1;
    }
  } else {
    for (const s of values){ counts[s] = (counts[s] || 0) + 1; }
  }
  return counts;
}

function makeBarPayload(title, counts){
  let entries = Object.entries(counts).sort((a,b)=>b[1]-a[1]);
  if (TOP_N && TOP_N > 0) entries = entries.slice(0, TOP_N);

  const labels = entries.map(([k]) => k);
  const values = entries.map(([,v]) => v);

  const max = Math.max(0, ...values);
  const suggestedMax = max <= 5 ? 5 : Math.ceil(max * 1.15);

  return {
    width: WIDTH,
    height: HEIGHT,
    backgroundColor: 'white',
    chart: {
      type: 'bar',
      data: { labels, datasets: [{ label: title, data: values }] },
      options: {
        responsive: false,
        plugins: { legend: { display: false }, title: { display: true, text: title } },
        // v4 én v2 config → y-as altijd vanaf 0
        scales: {
          x: { ticks: { autoSkip: false, maxRotation: ROTATE_X_TICKS, minRotation: ROTATE_X_TICKS } },
          y: { beginAtZero: true, min: 0, suggestedMin: 0, suggestedMax, ticks: { beginAtZero: true } },
          xAxes: [{ ticks: { autoSkip: false, maxRotation: ROTATE_X_TICKS, minRotation: ROTATE_X_TICKS } }],
          yAxes: [{ ticks: { beginAtZero: true, min: 0 } }]
        }
      }
    },
    filename: slugify(title) + '.png'
  };
}

// --- Maak 1 item per kolom ---
const out = [];

for (const colName of headerNames){
  if (IGNORE_COLS.has(colName)) continue;

  // verzamel waarden in deze kolom
  const raw = [];
  for (const row of dataRows){
    const v = row[colName];
    if (isBlank(v)) continue;
    raw.push(String(v).trim());
  }
  if (!raw.length) continue;

  // explode meerkeuze
  const vals = [];
  for (const s of raw){
    if (s.includes(SPLIT_CHAR)){
      vals.push(...s.split(SPLIT_CHAR).map(x => x.trim()).filter(Boolean));
    } else {
      vals.push(s);
    }
  }

  const counts = makeCountsFromValues(vals, colName);
  if (!Object.keys(counts).length) continue;

  out.push({ json: makeBarPayload(colName, counts) });
}

// >>> BELANGRIJK: geef een ARRAY van items terug
return out;
```
**Uitleg:** Berekent per kolom de antwoordfrequenties (incl. bucketising van 0–100 vragen), bouwt een Chart.js v4-barconfig met y-as vanaf 0 en levert per vraag een PNG-payload.

---

## 7) HTTP → QuickChart — PNG genereren (v4)

```jsonc
{
  "chart": {{ $json.chart }},          // Chart.js config uit de Code-node
  "width": {{ $json.width }},          // breedte van de chart
  "height": {{ $json.height }},        // hoogte van de chart
  "backgroundColor": {{ $json.backgroundColor }}, // canvaskleur
  "format": "png",                     // outputformaat
  "version": 4                         // forceer Chart.js v4 (nodig voor scales)
}
```
**Uitleg:** Stuurt de Chart.js configuratie en afmetingen naar `https://quickchart.io/chart` en vraagt expliciet om een v4-PNG terug.

> **n8n-instelling tip:** zet **Response Format = File** en **Put Output In Field = `data`** zodat elke uitvoer een binaire PNG onder `data` heeft.

---

## 8) Google Drive — Upload PNG

```
Binary Property: data
File Name: {{ $json.filename }}
Parents (folderId): <JOUW_MAP_ID>
```
**Uitleg:** Uploadt de binaire PNG uit veld `data` naar de opgegeven Drive-map met een bestandsnaam afgeleid van de vraag.

---

## 9) (Optioneel) Multi-select delimiter en Top-N aanpassen

```js
// bovenin de Chart Builder:
const SPLIT_CHAR = ';';  // wijzig naar ',' of '|' als je een andere delimiter gebruikt
const TOP_N = 10;        // toon alleen de top-10 categorieën
```
**Uitleg:** Laat je de scheidingstekens voor meerkeuze en het maximaal aantal categorieën per grafiek configureren.

---

## 10) (Optioneel) Horizontale bars

```js
// in makeBarPayload → chart.options:
indexAxis: 'y',
```
**Uitleg:** Draait de assen om zodat categorieën verticaal staan en waardes horizontaal.
