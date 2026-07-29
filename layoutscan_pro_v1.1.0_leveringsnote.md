# LayoutScan Pro v1.1.0 — Levering: Sprint 6, Business Case Engine

**Versionskode:** LayoutScan Pro v1.1.0 — Sprint 6 — Business Case Engine
**Fil:** `layoutscan_pro_v1.1.0.html` (bygget oven på det låste `layoutscan_pro_v1.0.html`)

**GitHub commit-besked:**
`LayoutScan Pro v1.1.0 – Added Business Case Engine with investment, savings, ROI, payback, scenarios and report integration.`

---

## 1. Kodegennemgang — hvad er ændret

Der er kun **én fil** i projektet (`layoutscan_pro_*.html`), så "hvilke filer er ændret" besvares her som: den samme fil, udelukkende udvidet. Jeg har kørt en linje-for-linje diff mod `layoutscan_pro_v1.0.html` for at bekræfte, at ændringerne udelukkende er:

- Version-tekst (titel/undertekst)
- To nye knapper i topbaren ("💼 Business Case" — "🎬 Demo" var allerede der)
- Et nyt CSS-afsnit til Business Case-modalen (step-wizard, kort, tabel, statusbokse) — genbruger samme farvepalet og komponentsprog som resten af appen
- Et nyt checkbox-felt ("Medtag Business Case") i de eksisterende rapportindstillinger
- Ny modal-markup (`#bcModal`) indsat før `<script>`-taggen
- Ét nyt felt i `state` (`businessCase: null`) og et opdateret felt i `reportSettings` (`includeBusinessCase:false`)
- Syv små, ét-linjes udvidelser i eksisterende funktioner: `saveProject()`, `loadProject()`, `exportProjectJSON()`, `importProjectJSON()`, `exportExcel()`, `exportAllCSV()` — hver af dem har fået tilføjet businessCase-håndtering uden at den oprindelige logik er ændret
- Én ny sektion indsat i `generateReportHTML()` (mellem "Forbedringsforslag" og "Konklusion"), betinget af at "Medtag Business Case" er markeret
- Ét nyt, samlet modul ("15. BUSINESS CASE ENGINE") indsat før INIT — al ny logik bor her

**Intet af følgende er rørt:** Canvas-motoren, analysemotoren (Sprint 3), Dashboard 2.0/heatmap (Sprint 4), rapport-/eksportmotoren fra Sprint 5 (bortset fra de nævnte tilføjelser), truckbiblioteket, layers, måleværktøjerne, demo-projektet eller startskærmen.

## 2. Business Case i appen

- Ny fane/knap **"💼 Business Case"** i topbaren, ved siden af Rapport og Demo.
- Åbner som en 4-trins guide: **Trin 1 (Nuværende lager) → Trin 2 (Ny løsning) → Trin 3 (Økonomi) → Trin 4 (Resultat)**, med step-indikatorer man kan klikke direkte på, samt Tilbage/Næste. Data bevares, uanset hvor man klikker hen.
- Trin 2 henter automatisk pallepladser, areal, antal reoler/fag/niveauer, valgt trucktype og anbefalet gangbredde fra det aktuelle projekts analyse — markeret med en lille "AUTO"-label. En "🔄 Opdatér fra layout"-knap genindlæser disse værdier, og alle felter kan altid rettes manuelt.
- Tre scenarier — **Forsigtig / Forventet / Ambitiøs** — med egne procentsatser for besparelser. "Forventet" er aktivt som standard, og der er ingen forudfyldte procenter (alt starter på 0).
- Trin 4 viser store resultatkort, en tydelig grøn/gul/rød statusboks med forklarende tekst, en scenariesammenligningstabel og en automatisk, regelbaseret konklusion — med den lovpligtige forbeholdstekst.

## 3. Beregningsformler (jf. kravspecifikationens afsnit 6, A–J)

Alle formler er implementeret præcis som beskrevet:

- **A. Kapacitetsforbedring** = nye pallepladser − nuværende pallepladser (samt i procent)
- **B. Arealudnyttelse** (nuværende og ny) = anvendt areal ÷ samlet areal × 100
- **C. Besparelse på arbejdstid** = (arbejdstid/dag × arbejdsdage/år) × reduktion% × timeomkostning
- **D. Besparelse på truckkørsel**: jeg har valgt metode 2 (**procentvis reduktion i årlig truckomkostning**: antal trucks × årlig omkostning pr. truck × reduktion%), da det er den metode, der matcher de felter, opgaven selv beder om i Trin 3 (der er ikke bedt om et "kr./km"-felt). Kilometertallet vises stadig informativt.
- **E. Eksternt lager** = eksterne lageromkostninger × reduktion%
- **F. Skader og fejl**: Trin 1 har **kun ét samlet felt** for "skader og fejl" (ikke to separate omkostningsfelter, som formlen ellers antyder). Jeg har løst det ved at beregne besparelsen som dette ene felt × **gennemsnittet** af de to procentsatser (fejlreduktion og skadereduktion). Det er en bevidst, gennemsigtig tilnærmelse — se punkt 6 nedenfor.
- **G. Samlet årlig besparelse** = løn + truck + eksternt lager + skader/fejl + øvrige − nye driftsomkostninger
- **H. Tilbagebetalingstid** = investering ÷ nettobesparelse, vist som "X år og Y måneder". Ved nul/negativ besparelse vises den krævede tekst: *"Investeringen har med de indtastede oplysninger ingen beregnet tilbagebetalingstid."*
- **I. Årlig ROI** = nettobesparelse ÷ investering × 100, tydeligt mærket som en simpel årlig ROI
- **J. Samlet gevinst** = nettobesparelse × antal år − investering

**Statusfarve** (grøn/gul/rød) er **ikke** baseret på vilkårlige grænser: grøn kræver positiv besparelse **og** tilbagebetaling under den angivne forventede levetid; gul er positiv besparelse med tilbagebetaling **over** levetiden; rød er nul/negativ besparelse eller ingen gyldig tilbagebetaling.

## 4. Sådan gemmes Business Case-data

Data ligger i en selvstændig gren i projektets JSON-struktur, præcis som foreslået i opgaven:

```
businessCase: {
  version: 1,
  currentState: {...},
  proposedState: {...},
  investment: {...},
  operatingCosts: {...},
  scenarios: { active, forsigtig, forventet, ambitios },
  results: {...}
}
```

Den gemmes sammen med resten af projektet i `localStorage` (autosave, som alt andet i appen) og indgår også i JSON-/Excel-/CSV-eksport samt -import. En ny funktion, `migrateBusinessCase()`, sikrer bagudkompatibilitet: åbnes et ældre projekt (fra v1.0 eller tidligere) uden Business Case-data, oprettes strukturen automatisk med sikre nulværdier — appen fejler ikke.

## 5. To bevidste fortolkninger (læs venligst)

1. **"Forventet kapacitetsforøgelse i %"** (nævnt som et Trin 2-inputfelt i opgaven) håndteres **ikke** som et separat, manuelt indtastet felt. Afsnit 6.A definerer det allerede som en beregnet værdi (nye minus nuværende pallepladser), så et ekstra manuelt procentfelt ville stå i modstrid med den formel. Kapacitetsforbedringen vises i stedet som et beregnet resultat i Trin 4.
2. **Skade- og fejl-besparelse** bruger, som beskrevet ovenfor, ét kombineret omkostningsfelt fra Trin 1 (der findes ikke separate felter for "skadeomkostninger" og "fejlomkostninger" i kravspecifikationens Trin 1-liste), ganget med gennemsnittet af de to reduktionsprocenter fra scenarierne. Hvis I hellere vil have to separate omkostningsfelter i Trin 1, er det en lille, isoleret ændring at tilføje i en senere rettelse.

## 6. Test udført

- **Syntaks**: `node --check` på hele JavaScript'en — bestået.
- **HTML-struktur**: html5lib-parser — ingen parse-fejl. 116 unikke id'er; alle 95 `getElementById`-kald er verificeret at pege på et eksisterende element (én oprindelig "mismatch" i den automatiske eftersyning viste sig at være et falsk alarm fra selve valideringsscriptet — det fejlfortolkede en escaped anførselstegn i kildekoden, ikke en fejl i appen; bekræftet ved manuelt eftersyn).
- **Diff mod v1.0**: bekræfter, at samtlige ændringer er additive og lokaliserede som beskrevet i afsnit 1 — intet i de eksisterende moduler er omskrevet.
- **Isolerede beregningstests** (Node, uden for browseren) af hele formel-motoren:
  1. Et realistisk eksempel (2.000 → 3.240 pallepladser, 3,8 mio. kr. investering) gav en tilbagebetaling på **6 år og 8 måneder** og status **grøn** — korrekt ift. levetiden på 10 år.
  2. Investering uden besparelse → **rød status** og den korrekte fallback-tekst for tilbagebetaling (ikke en ugyldig beregning).
  3. Besparelse uden investering (0 kr.) → ROI og tilbagebetaling vises korrekt som "–", ikke `Infinity`.
  4. Nul-værdier i nævneren (0 nuværende pallepladser, 0 m² samlet areal) → giver `null` (vises som "–"), ikke `NaN`.
  5. Meget store beløb (500 mio. kr.) → stadig et endeligt, korrekt resultat.
  6. Afrundingskant ved præcis 12 måneder → viser korrekt "4 år og 0 måneder", ikke fejlagtigt "3 år og 12 måneder".

**Ikke testet i denne omgang (kræver rigtig browser, jf. jeres egen testliste i opgaven):** faktisk brug af 4-trins-guiden i browseren, mobil-/tablet-/desktopvisning, PDF-udskrift med Business Case-sektionen, Excel-/CSV-åbning med det nye "BusinessCase"-ark/-fil, samt en reel gem-og-genåbn-tur i browseren (kun logikken er verificeret, ikke den fulde brugerrejse).

## 7. Bekræftelse: eksisterende funktionalitet

Diff-verificeret uændret: canvas-motor, analysemotor, KPI-dashboard/heatmap, truckbibliotek, layers, måleværktøjer, rapportmotor (Sprint 5), demo-projekt og startskærm (v1.0). Ældre projekter uden Business Case-data indlæses uden fejl og får automatisk en tom, sikker Business Case-struktur.

---
**Anbefaling:** Kør jeres testliste igennem i en rigtig browser (især punkt 1-4 og 10-14 fra opgavens afsnit 15), og sig til når Sprint 6 er godkendt — så fortsætter vi efter jeres plan med Sprint 7 (FlowScan) og Sprint 8 (AI Layout Optimizer), én ad gangen.
