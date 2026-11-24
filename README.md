# Mensa Sverige – SWAG-statistik

Infografiksida för Mensa Sveriges årsträff (Mensa SWAG) och halvårsträff. Visar medlemsutveckling, deltagarstatistik, trender, kartor och ekonomisk uppskattning.

**Live:** Hostas via GitHub Pages – `index.html` renderas direkt utan build.

## Teknik

- En enda `index.html` – ingen build, inga beroenden att installera
- Chart.js (CDN) för grafer
- Leaflet + OpenStreetMap-plattor (CDN) för karta. Mörkt läge görs med CSS-filter, så ingen API-nyckel behövs.
- Ren JavaScript, ingen framework
- Dark mode följer enhetens tema. Knappen sparar bara ett val som avviker från enheten (localStorage), och graferna ritas om utan omladdning.
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

Saknas ett år räknas befolkningen fram från senast kända år och snittillväxten de fem senaste åren. Punkten visas då ihålig i grafen.

### 6. Inflation (KPI)

Sök: `const KPI = {`

Lägg till årsmedeltalet när SCB publicerat det ([KPI, historiska tal, juli 1914=100](https://www.scb.se/hitta-statistik/statistik-efter-amne/priser-och-ekonomiska-tendenser/priser/konsumentprisindex-kpi/pong/tabell-och-diagram/historiska-tal/kpi-historiska-tal-1830/)) och flytta `KPI_BASE_YEAR` till senaste hela år:

```javascript
2026: 6140,
```

KPI används i ekonomimodellen för mat, dryck, aktiviteter och saknade konferenskostnader, och för att visa beloppen i fasta priser. År efter tabellen räknas med 2 % inflation (`KPI_FUTURE_INFLATION`).

## Automatiska beräkningar

Följande behöver **inte** uppdateras manuellt:

- Mensa SWAG-datum (beräknas från påskdagen → Kristi himmelsfärd)
- Halvårsträff-datum (tredje helgen i november, fre–sön)
- Bokning öppnar (första lördagen i februari) och sista ordinarie bokningsdag (~en vecka före SWAG)
- Countdowns (byter automatiskt år)
- Trendlinjer och prognoser (linjär regression från `TREND_START_YEAR`, `FORECAST_HORIZON` år framåt, samma för alla grafer och prognosrutan)
- Slider min/max och färgförklaring på kartan
- "Senast uppdaterad" i footern (filens ändringsdatum)
- Nästa SWAG-prognos, inklusive ekonomi (samma modell som historiken)

## Ikoner och delningsbild

- `favicon.svg` är källan för `favicon-32.png` och `apple-touch-icon.png`
- `og-image.svg` är källan för `og-image.png` (förhandsbild när länken delas)

Rendera om efter ändring:

```bash
inkscape og-image.svg -o og-image.png
inkscape favicon.svg -w 32 -h 32 -o favicon-32.png
inkscape favicon.svg -w 180 -h 180 -o apple-touch-icon.png
```

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

Sök: `function renderAll()` och lägg till `renderMyNewChart();`. Funktionen körs vid sidladdning och vid temabyte, så grafen ritas om med rätt färger.

Ge även `<canvas>` en beskrivning för skärmläsare: `role="img" aria-label="..."`.

### Konventioner

- **Färger:** Blå (`#1a237e`, i mörkt läge `primaryLineColor()`) = medlemmar/SWAG, Orange (`#ff5722`) = procent, Grön (`#2e7d32`) = penetration, Lila (`#7c4dff`) = halvårsträff, Gul streckad (`#ffc107`) = trendlinjer
- **Trend-box:** Grön bakgrund, visar R² och prognos
- **Trender:** Använd `TREND_START_YEAR` och `FORECAST_HORIZON`, eller `membersTrend()`/`attendeesTrend()`, så att alla prognoser stämmer överens
- **Pandemiår:** Exkluderas med `!d.cancelled` i filter
- **Namngivning:** "Mensa SWAG" (inte bara "SWAG"), "Mensa Halvårsträff"
- **Dark mode:** Fungerar automatiskt – Chart.js defaults sätts i `applyTheme()`
- **Hover:** Tooltip och markering av hela året under pekaren är globala defaults, inget behöver göras per graf

## Hjälpfunktioner tillgängliga

- `linearRegression(points)` – returnerar `{ slope, intercept, r2 }`
- `forecast(reg, year)` – beräknar värde för ett år (aldrig < 0)
- `membersTrend()`, `attendeesTrend()`, `hotelPriceTrend()` – gemensamma trender
- `easterSunday(year)` – påskdagen
- `ascensionDay(year)` – Kristi himmelsfärd
- `swagDates(year)`, `halfSwagDates(year)` – `{ start, end }`
- `bookingOpenDate(year)`, `lastBookingDate(year)` – bokningsdatum
- `hasSwagPassed(year)` – har årets SWAG redan varit?
- `isHeld(d)` – genomförd Mensa SWAG (även utan deltagarsiffra)
- `hasAttendance(d)` – genomförd med känd deltagarsiffra
- `getNextSwag()` – nästa SWAG utan data
- `kpiFor(year)`, `adjustForInflation(amount, fromYear, toYear)` – inflation
- `populationFor(year)` – befolkning, uträknad om SCB-siffra saknas
- `economyModel(year, attendees)` – ekonomimodellen
- `yearColor(index, total)` – gradient blå→orange för kartan

## Filer

- `index.html` – hela sidan (HTML + CSS + JS + data)
- `favicon.svg`, `favicon-32.png`, `apple-touch-icon.png` – ikoner
- `og-image.svg`, `og-image.png` – delningsbild
- `README.md` – denna fil
