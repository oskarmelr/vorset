# VORSET.

Drikkelekapp for vors under Rauma Rock '26. Festivalplakat/punk-design — Archivo Black, harde skygger, 0 border-radius.

## Bruk på iPhone
1. Åpne Pages-linken i Safari.
2. Del-knappen → «Legg til på Hjem-skjerm».
3. Appen åpnes i fullskjerm med eget ikon.

## Innhold
- `index.html` — hele appen (mal + logikk). Under 520 px fyller appen hele viewporten; på større skjermer vises telefon-mockup med notatpanel.
- `dc-runtime.js` — runtime som tolker malen
- `*.woff2` — Archivo Black + Space Grotesk (selvhostet, ingen eksterne kall)
- `*.png` + `manifest.json` — PWA-oppsett for hjemskjerm
- `vorset-standalone-prototype.html` — original selvpakkende prototype (referanse)

## Viktig begrensning
Kun klientside — all state er lokal i nettleseren til den som åpner siden. Ekte flerspiller (felles runder, hemmelige oppgaver til én telefon) krever backend (f.eks. Firebase/Supabase). Naturlig neste steg.

## Admin
Bak 🔒-knappen, PIN `7067`.
