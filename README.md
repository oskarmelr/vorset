# VORSET.

Drikkelekapp for vors under Rauma Rock '26. Festivalplakat/punk-design — Archivo Black, harde skygger, 0 border-radius.

**Live:** https://oskarmelr.github.io/vorset/

## Slik spiller dere
1. Én person trykker **«+ LAG NYTT VORS»** og får en 4-tegns romkode.
2. Alle andre taster samme romkode + brukernavn og trykker **«BLI MED I VORSET»**.
3. Alt synkes live: runder, poeng, stemmer, benking og hemmelige oppgaver.

Legg appen på hjemskjermen: åpne linken i Safari → Del → «Legg til på Hjem-skjerm».

## Admin
Bak 🔒-knappen, PIN `7067`. Admin kan:

- **RUNDER** — styre hvilken leke som er aktiv for alle, redigere alle leker (type, tittel, tekst, quiz-alternativer), legge til og slette leker.
- **OPPGAVER** — skrive, endre og slette hemmelige oppgaver, og sende dem til en tilfeldig aktiv spiller. Oppgaven dukker bare opp på den ene telefonen.
- **FOLK** — dele ut slurker, **benke** folk (de er med i rommet, men slipper alle leker og faller ut av pekelek/dueller), eller kaste dem ut.

## Teknisk
- Statisk side på GitHub Pages, ingen byggesteg.
- Sanntidssynk via **Firebase Realtime Database** (prosjekt `vorset-9010e`, region europe-west1).
- Delt state ligger under `/rooms/{ROMKODE}`: `rounds`, `roundIdx`, `tasks`, `players`, `votes`, `quiz`, `toast`.
- Spiller-ID lagres i `localStorage`, så du beholder poeng og identitet ved reload.
- Selfie skaleres til 112×112 JPEG i nettleseren før den lagres.

### Filer
- `index.html` — hele appen (mal + logikk + Firebase-oppsett)
- `dc-runtime.js` — runtime som tolker malen
- `*.woff2` — Archivo Black + Space Grotesk (selvhostet)
- `*.png` + `manifest.json` — PWA-oppsett for hjemskjerm
- `vorset-standalone-prototype.html` — original prototype (referanse)

## Viktig: sikkerhetsreglene utløper 28. august 2026
Databasen kjører foreløpig på Firebase sine **test-modus-regler**, som slutter å virke automatisk 28.08.2026. Da vil appen ikke lenger kunne koble til.

Fiks før den datoen: åpne [reglene](https://console.firebase.google.com/project/vorset-9010e/database/vorset-9010e-default-rtdb/rules), bytt ut alt med dette og trykk **Publish**:

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

Merk at dette (som test-modus) lar hvem som helst med romkoden lese og skrive i det rommet. Data er kun kallenavn, emoji, selfie og poeng — ikke legg inn noe sensitivt.
