# VORSET — teknisk gjennomgang

Skrevet for deg som skal sette denne sammen med en annen app. Her er alt du trenger for å forstå hvordan den henger sammen.

## Kjapp oppsummering
Statisk nettside uten byggesteg. Én HTML-fil inneholder både markup og logikk. All flerspiller-state ligger i Firebase Realtime Database. Ingen server, ingen npm, ingen bundler — dobbeltklikk `index.html` bak en lokal webserver, så kjører det.

```bash
python3 -m http.server 8000      # åpne http://localhost:8000
```

Må serveres over HTTP (ikke `file://`), fordi Firebase-SDK-en lastes som ES-modul.

## Filstruktur
| Fil | Hva det er |
|---|---|
| `index.html` | Hele appen: Firebase-oppsett i `<head>`, markup i `<x-dc>`, logikk i `<script type="text/x-dc">` |
| `dc-runtime.js` | Runtime som tolker malspråket og rendrer (React under panseret) |
| `*.woff2` | Archivo Black + Space Grotesk, selvhostet |
| `*.png`, `manifest.json` | PWA-ikoner for hjemskjerm |
| `vorset-standalone-prototype.html` | Original selvpakkende prototype, kun referanse |

## Hvordan malspråket virker
Runtimen leser dokumentet, finner `<x-dc>`-blokken og rendrer den. Klassen i `<script type="text/x-dc">` har én metode, `renderVals()`, som returnerer et objekt. Alle nøklene i det objektet kan brukes i markupen.

```html
<div>{{ roundTitle }}</div>
<button sc-camel-on-click="{{ nextRound }}">NESTE</button>
<sc-if value="{{ hasStarted }}"> … </sc-if>
<sc-for list="{{ voteOpts }}" as="p"> {{ p.name }} </sc-for>
```

Tre fallgruver som kostet tid:

1. **camelCase-attributter må prefikses med `sc-camel-`.** `onClick` → `sc-camel-on-click`.
2. **`src` på bilder må være `sc-camel-src`.** Skriver du `src="{{ p.img }}"` prøver nettleseren å hente den bokstavelige strengen `{{ p.img }}` og du får 404.
3. **`<sc-if>` må balansere.** Sjekk med `grep -c '<sc-if ' index.html` mot `grep -c '</sc-if>'`.

State: `this.state` er lokal UI-state (fane, åpne dialoger, skjemafelt). Alt som skal deles ligger i Firebase og kommer inn via `this.state.room`.

## Datamodellen i Firebase
Alt under `/rooms/{ROMKODE}`. Romkoden er 4 tegn (A–Z uten forvekslingsbokstaver, 2–9).

```
/rooms/K7M2/
  started   : bool          — kvelden i gang?
  roundIdx  : int           — hvilken leke er aktiv
  rounds    : [ {type, title, body, info, opts?, right?} ]
  tasks     : [ {t, who, state} ]        state: idle|sent|done|failed
  picks     : { roundIdx: playerId }     — hvem som er trukket ut
  players   : { pid: {name, emoji, img, pts, benched, crown, joined} }
  votes     : { roundIdx: { pid: targetPid } }
  quiz      : { roundIdx: { pid: svarIndeks } }
  toast     : { msg, ts }                — felles kunngjøring
  interval  : int
  pong      : { phase, invites, teams, matches, stages, champ }
```

**Spiller-ID** genereres tilfeldig og lagres i `localStorage` (`vorset_pid`), sammen med profilen (`vorset_me`). Derfor beholder man poeng og identitet ved reload.

**Turneringen** (`pong`):
- `phase`: `lag` (velger makkere) → `kamp` (bracket kjører) → `ferdig`
- `invites`: `{ fraPid: tilPid }` — slettes når den andre godtar
- `teams`: `{ "pidA-pidB": {a, b} }`
- `matches`: `{ "s0m0": {stage, slot, t1, t2, winner} }` — nøkkel er `s<stage>m<slot>`
- Vinner i `s{n}m{k}` føres til `s{n+1}m{floor(k/2)}`, som `t1` hvis `k` er partall, ellers `t2`. Lag uten motstander vinner automatisk (bye). Se `resolveBracket()`.

## Firebase-oppsett
Config ligger i klartekst i `<head>` i `index.html`. **Det er meningen** — Firebase sine web-API-nøkler er offentlige og fungerer som en peker til prosjektet, ikke som passord. GitHub sin secret scanning flagger dem uansett; det er en kjent falsk positiv.

Det som faktisk beskytter dataene er reglene:

```json
{
  "rules": {
    "rooms": {
      "$rom": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

Tilgang gis kun inne i ett rom om gangen — ingen kan lese roten eller liste opp hvilke rom som finnes. Alle med romkoden kan lese og skrive i det rommet, så ikke legg sensitive data inn.

Skal du peke mot ditt eget Firebase-prosjekt: lag et prosjekt, slå på Realtime Database, lim inn reglene over, og bytt ut `initializeApp({...})`-objektet i `<head>`.

## Hvis dere skal slå sammen to apper
Noen ting å vite før dere begynner:

- **Runtimen er ikke et vanlig rammeverk.** Skal dere over på React/Vue/Svelte, er `renderVals()` en ganske grei mal å porte fra — den er allerede en ren funksjon fra state til view-data. Markupen må skrives om.
- **Ingen autentisering.** Identitet er en tilfeldig ID i `localStorage`, og admin er en firesifret PIN (`7067`) som er lik for alle rom. Skal appene slås sammen med noe som har ekte brukere, er dette punktet som må rives ut først.
- **Ingen serverlogikk.** All validering skjer i klienten, så en teknisk anlagt gjest kan skrive hva som helst inn i sitt eget rom. Greit for et vors, ikke greit hvis noe skal telle på ordentlig.
- **Alt er samlet i én fil.** Det gjør sammenslåing enklere enn det ser ut — det finnes ingen skjulte importer eller byggekonfigurasjon.
- **Alle eksterne kall er lokale unntatt to:** Firebase-SDK-en fra `gstatic.com` og selve databasen. Fontene ligger i repoet.

## Testing
Testene ble kjørt med Playwright, med flere nettleserkontekster mot samme rom for å verifisere sanntidssynk.

```bash
pip install playwright && playwright install chromium
```

Merk: fire samtidige nettlesere sprengte minnet under utvikling. To ekte klienter, med resten av spillerne skrevet rett inn i databasen, fungerte bedre.

## Kjente løse tråder
- Notatpanelet som vises ved siden av telefon-mockupen på store skjermer inneholder utdatert tekst fra førsteutkastet. Skjules under 520 px, så det påvirker ikke mobil.
- Admin-PIN er hardkodet og lik for alle rom.
- Ingen opprydding av gamle rom i databasen.
- Ingen håndtering av at admin mister nett midt i en runde.
