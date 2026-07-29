# VORSET.

Drikkelekapp for vors under Rauma Rock '26. Festivalplakat/punk-design — Archivo Black, harde skygger, 0 border-radius.

**Live:** https://oskarmelr.github.io/vorset/

## Slik spiller dere
1. Én person trykker **«+ LAG NYTT VORS»** og får en 4-tegns romkode.
2. Del koden med **«INVITER FOLK»** — åpner delingsmenyen på telefonen, så du kan AirDroppe eller sende den på SMS. Det finnes også egne knapper for SMS og kopier-lenke.
3. Alle andre taster romkoden + brukernavn og trykker **«BLI MED I VORSET»**.
4. Alle havner i et **venterom** med romkoden. Kvelden starter først når admin trykker **«START KVELDEN»**.
5. Alt synkes live: runder, poeng, stemmer, benking og hemmelige oppgaver.

Leaderboardet er hovedsiden. Hver leke har en **ⓘ-knapp** som viser en kort forklaring av reglene.

## Rundestart
Hver gang en ny runde settes i gang popper det opp en fullskjerm-intro hos alle samtidig: rundenummer, type, tittel, hva som skal skje, de korte reglene — og hvem som er trukket ut. Man trykker «KLAR →» for å gå videre.

En **tilfeldig aktiv spiller** trekkes ut hver runde og vises med 🎯 både i introen og på rundekortet. I quiz-runder er det den utvalgte som svarer; de andre får beskjed om at det ikke er deres tur. Admin kan trekke ny spiller med 🎲-knappen.

## Beerpong-turnering
Egen **PONG**-fane nederst.

1. Admin åpner en turnering med **«NY TURNERING — ÅPNE FOR LAG»**.
2. Alle velger makker selv: trykk på et navn → den andre får «X vil ha deg på lag» og må trykke **JA**. Først når begge har godtatt teller laget.
3. Admin trykker **«SETT OPP KAMPENE»**, og det genereres et turneringsoppsett med semifinaler/kvartfinaler fram mot finalen. Er antall lag ikke en toerpotens, går noen automatisk videre på bye.
4. Admin kårer vinner ved å trykke på laget i hver kamp. Vinneren føres videre i diagrammet automatisk.
5. Seierslaget får **👑** ved navnet sitt resten av kvelden — på leaderboardet, i pekeleker og i admin.

Admin kan fjerne krona fra en spiller med **👑✕** i FOLK-fanen, og starte en ny turnering når som helst.

Legg appen på hjemskjermen: åpne linken i Safari → Del → «Legg til på Hjem-skjerm».

## Lekene
16 runder ferdig utfylt — blant annet Psykiateren (sosial deduksjon uten utstyr), Venstrehåndsregelen, Siste bilde i kamerarullen, To sannheter og en løgn, Forbudt bokstav, Mest spilte låt, Aksentrunden, Telefontårnet, en vannpause og bunnefest. Admin kan endre alt og legge til egne.

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

## Sikkerhet
Databasereglene er permanente (ingen utløpsdato) og avgrenset til ett rom om gangen:

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

Det betyr at ingen kan liste opp alle rom eller røre noe annet i databasen.

### Om «Google API Key»-varselet fra GitHub
GitHub sin secret scanning flagger Firebase-nøkkelen i `index.html`. Det er en kjent falsk positiv: Firebase sine **web-API-nøkler er offentlige by design** og må ligge i klientkoden for at appen skal virke. Nøkkelen er bare en peker til riktig prosjekt, ikke et passord. Det som faktisk beskytter dataene er sikkerhetsreglene over. Varselet kan lukkes som «used in tests / false positive». Alle som kjenner en romkode kan derimot lese og skrive i akkurat det rommet — innholdet er kun kallenavn, emoji, selfie og poeng, så ikke legg inn noe sensitivt.
