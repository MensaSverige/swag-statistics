# Mensa Sverige – SWAG-statistik

Infografiksida för Mensa Sveriges årsträff (Mensa SWAG) och halvårsträff. Visar medlemsutveckling, deltagarstatistik, trender, kartor och ekonomisk uppskattning.

**Live:** Hostas via GitHub Pages – `index.html` renderas direkt utan build.

## Teknik

- En enda `index.html` – ingen build, inga beroenden att installera
- Chart.js (CDN) för grafer
- Leaflet + OpenStreetMap/CARTO (CDN) för karta
- Ren JavaScript, ingen framework
- Dark mode med localStorage
- Alla beräkningar (trender, prognoser, datum) sker dynamiskt vid sidladdning

## Uppdatera data

All data finns i `index.html` i JavaScript-sektionen. Sök efter respektive array.

### 1. Årsträff (Mensa SWAG)

Sök: `const DATA = [`

Lägg till/uppdatera en rad:

```javascript
{ year: 2028, members: XXXX, attendees: XXX, location: "Stad", comment: null, hotelPrice: XXXX },
```

Fält:

- `members` – antal betalande medlemmar (källa: Mensa Legatus #2)
- `attendees` – deltagare på Mensa SWAG (källa: Eventgruppen)
- `location` – SWAG-ort
- `comment` – valfri kommentar (visas i tabellen)
- `hotelPrice` – standardrum/natt på huvudhotellet (används i ekonomimodellen)
- `cancelled: true` – lägg till om inställd (exkluderas från trender)

Om ny ort: lägg även till koordinater i `const COORDS = {`:

```javascript
"Stad": [latitud, longitud],
```

### 2. Halvårsträff

Sök: `const HALF_SWAG = [`

```javascript
{ year: 2027, attendees: XXX, location: "Stad", swagYear: 2028 },
```

- `year` = året halvårsträffen hålls (november)
- `swagYear` = Mensa SWAG-året den rekar för (året efter)
- `cancelled: true` + `comment` för digitala/inställda

### 3. Mensapristagare

Sök: `<ul class="prize-list">`

Lägg till överst:

```html
<li><strong>YYYY</strong> – Pristagarens namn</li>
```

### 4. Konferenskostnad (ekonomimodell)

Sök: `const CONFERENCE_COST = {`

```javascript
2028: 310000,
```

### 5. Sveriges befolkning (penetrationsgrafen)

Sök: `const SWEDEN_POP = {`

Lägg till nytt år (källa: SCB):

```javascript
2027: 10750000,
```

## Automatiska beräkningar

Följande behöver **inte** uppdateras manuellt:

- Mensa SWAG-datum (beräknas från påskdagen → Kristi himmelsfärd)
- Halvårsträff-datum (tredje fredagen i november)
- Countdowns (byter automatiskt år)
- Trendlinjer och prognoser (linjär regression)
- Slider min/max på kartan
- "Senast uppdaterad" i footern
- Nästa SWAG-prognos

## Lägga till nya grafer

Mönstret för en ny graf:

### 1. HTML-sektion

```html
<div class="section">
  <h2>Min nya graf</h2>
  <div class="chart-container chart-medium">
    <canvas id="myNewChart"></canvas>
  </div>
  <div class="trend-box" id="myNewNote"></div>
</div>
```

Höjdklasser: `chart-tall` (350px), `chart-medium` (280px), `chart-small` (220px).

### 2. JavaScript-funktion

```javascript
function renderMyNewChart() {
    // Filtrera data
    const valid = DATA.filter(d => d.year >= STAT_START_YEAR && d.members != null);
    const labels = valid.map(d => d.year);
    const values = valid.map(d => /* beräkning */);

    // Valfri trendlinje
    const points = valid.map(d => [d.year, /* värde */]);
    const reg = linearRegression(points);
    const trendData = labels.map(y => forecast(reg, y));

    new Chart(document.getElementById('myNewChart'), {
        type: 'line', // eller 'bar'
        data: { labels, datasets: [
            { label: 'Min data', data: values, borderColor: '#XX', backgroundColor: 'rgba(...)', fill: true, tension: 0.3, pointRadius: 3 },
            { label: 'Trend', data: trendData, borderColor: '#ffc107', borderDash: [6, 3], pointRadius: 0, fill: false },
        ]},
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } },
            scales: { y: { min: 0, title: { display: true, text: 'Enhet' } }, x: {} }
        }
    });

    // Trend-text
    document.getElementById('myNewNote').innerHTML = `📈 ...`;
}
```

### 3. Anropa i INIT

Sök: `// INIT` och lägg till `renderMyNewChart();`

### Konventioner

- **Färger:** Blå (`#1a237e`) = medlemmar/SWAG, Orange (`#ff5722`) = procent, Grön (`#2e7d32`) = penetration, Lila (`#7c4dff`) = halvårsträff, Gul streckad (`#ffc107`) = trendlinjer
- **Trend-box:** Grön bakgrund, visar R² och prognos
- **Pandemiår:** Exkluderas med `!d.cancelled` i filter
- **Namngivning:** "Mensa SWAG" (inte bara "SWAG"), "Mensa Halvårsträff"
- **Dark mode:** Fungerar automatiskt – Chart.js defaults sätts vid laddning

## Hjälpfunktioner tillgängliga

- `linearRegression(points)` – returnerar `{ slope, intercept, r2 }`
- `forecast(reg, year)` – beräknar värde för ett år (aldrig < 0)
- `easterSunday(year)` – påskdagen
- `ascensionDay(year)` – Kristi himmelsfärd
- `hasSwagPassed(year)` – har årets SWAG redan varit?
- `getNextSwag()` – nästa SWAG utan data
- `yearColor(index, total)` – gradient blå→orange för kartan

## Filer

- `index.html` – hela sidan (HTML + CSS + JS + data)
- `README.md` – denna fil
