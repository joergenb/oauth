# RAR i ID-porten

## Kva er RAR ?

Erstatter (helst) `scope` med ein transaksjonspesifikk/fingranulert "samtykke" til ein `authorization_details` som er ein array av **autorisasjonsobjekter** som kvart består av:
- standardiserte felt:
  - type (påkrevd felt)
  - action
  - locations (samme som aud, stort sett)
  - identifier  (peikar på ressursen hjå RS)
  - datatypes (peiker på kva datatyper klient ønsker å ha frå RS)
- eigendefinerte felt,
  - ein `type` vil normal ha definert ein tilhøyrande gyldig datamodell


MERK: AS skal avvise dersom eit av autorisasjonsobjekta har syntaks-feil eller element som er ukjende.

MERK 2: AS kan endre authorisasjonsobjekta i responsen, basert på brukaren sine handlingar i samtykke-dialogen.


## Mogelege typer:

### Basic ansattporten
vil kun ha pålogging av "nokon med ei viss rolle/representasjon". Klient kjenner ikkje org. frå før:

Request blir lik som dagens, med følgjande tillegg:
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:role:rolletypekode"
  }
]
```


Full datamodell
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:[resource eller role]:[identifikator]",
    "mulige_avgivertyper": [ "foretak", "bedrift", "person"] // default: foretak
    "avgiver_hint": { [ avgiver i iso6523] } // trengs det noko hint på brukerstyrt begrensning
  }
]
```

Bør kun vere Altinn-3.0-kompatible ressurser.

|Ressurs-identifikator| Beskrivelse|Eksempel|
|-|-|-|
|urn:altinn:role:{rolletypekode} | Kan velge fra whitelist av [Altinn-rollene](https://www.altinn.no/api/metadata/roledefinitions?language=1044). | altinn:role:siskd
|urn:altinn:resource:{tjenestekode}:{tjenesteutgave} | Altinn 2 [tenestekode/utgåve](https://www.altinn.no/api/metadata?language=1044) | altinn:resource:3906:141205
|urn:altinn:resource:{org}:{appname} | Altinn 3 [org/app](https://www.altinn.no/api/metadata?language=1044) | altinn:resource:skd:sirius

For Enhetsregisteret er det typisk nokon med nøkkelroller / prokura/ (signeringsrett) som kan vere aktuelt å ha

> **Mange av dagens standard Altinn-roller gir veldig breie tilganger ("Post/arkiv", "Utfyller/innsender").  Dette er problematisert med at de ikkje følger gode dataminimeringsprinsipp, og vanskeliggjør det å skulle holde oversikt over hva en gitt rolle faktisk gir tilgang til.  Derfor bør ein kanskje ikkje tilby rolle som muligheit i det heile, men heller kreve at det defineres tjenester som det spørres på. Ein mellomting er å berre godkjenna førespurnader på eit sterkt avgrensa sett med Altinn-roller.**

Bruker velger så en - og bare en - organisasjon i organisasjonsvelger.
Respons i id_token

```
"sub": "WE0DjFv9ygb2rjS7s_tXsg-fez2Co3Q8oxUmcvQ0mzQ=",
"iss": "https://oidc.difi.no/idporten-oidc-provider/",
"pid": "<fnr sluttbruker>",
...
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:role:siskd"
    "ressurs_name": "Begrenset signeringsrett",
    "avgiver": {
        "Authority": "iso6523-actorid-upis",
        "ID": "0192:999888777"  // org.no til arbeidsgiveren som den innlogga brukeren har valgt i org.velger
    }
  }
]
```

### Flere søke-kriterier

Dersom det er naturlig at flere roller skal ha adgang til tjenestene, sender klient inn flere autorisasjonsobjekter.

```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:role:a0237"  // Ansvarlig revisor
  },
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:role:a0239"  // Regnskapsfører med signeringsrett
  }
]
```

Organisasjonsvelger bør opplyse noe om dette, t.d.
> Denne tjenesten støtter kun pålogging av personer som har rollene
> * Ansvarlig revisor
> * Regnskapsfører med signeringsrett


Bruker kan fremdeles bare velge en organisasjon.  Dersom bruker har flere av de forespurte rollene i valgt organisasjon, vil responsen inneholde flere autorisasjonsobjekter.

### Lokal hurtigbytte av avgiver

Formål: gjere det enkelt for proff-bukrere å kunne bytte avgiver raskt.
bruker kan velge i lokal GUI kven av dei andre mulege avgiverne ein ønskjer å representere

1. Bruker velger org.  AP lagrer kandidatliste på autorisasjonen. Klient får token.
2. Klient kaller dedikert endepunkt med token for hente kandidatliste
3. Bruker velger ny kandidat i klient.
4. Klient utfører tokenexchange mot AP, dersom target-reportee finst i kandidatlista utsteder AP nytt token (samme autorisasjon, ny autorisajon ?)  kandidatliste oppfriskes mot Altinn dersom eldre enn x sec.  1 versjon:  rein stateless, kaller Altinn kvar gong kandidateendepunkt eller exchange-endepunkt


"korps kontra equinor-styreleder"-problmatikken -> bør vere eigen autorisasjontype.  eller brukerstyr swithc "ønsker å representere alle disse"  (kun en,  alle,  noen utvalgte)

```
GET /tokeninfo/avgiverliste 
Authorization: Bearer <token>
```

gir respons:

```
[
  {
      "Authority": "iso6523-actorid-upis",
      "ID": "0192:999888777",
      "avgiver_hintet": true
  },
  {
      "Authority": "iso6523-actorid-upis",
      "ID": "0192:987654321"
  },
  {
      "Authority": "norwegian-population-register", // Finnes det noe standardisert her? En ISO/UPIS for fysiske personer?
      "ID": "11028034569"
  },
  {
      "Authority": "norwegian-population-register", 
      "ID": "411028034569" // D-nummer ..?
  }
]
```
*TODO! Identifikatorer for personer. Kan vi eksponere en liste med fnr her ..?*

Denne lista kan da presenteres for sluttbruker i portalen, og valg

```
POST /tokenexchange?ny_avgiver=0192:987654321
Authorization: Bearer <token>
```

gir respons

```
eyJhbGciOiJSUzI1NiIsImtpZCI6I... <nytt id_token>
```


### Tilgangsstyring på tjenestenivå

Fremfor å spørre på standardrollene er det i de fleste tilfeller å foretrekke at tjenesten oppretter en egen spesialisert autorisasjonsressurs i form av en lenketjeneste i Altinn. Tilgang til denne tjenesten kan (men må ikke) forhåndstildeles til et valgt sett med roller, f.eks. "Daglig leder". All annen tilgang må eksplisitt delegeres.


Request omtrent som for rolle:
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:resource:5129:1"
  }
]
```

### Basic datautvekling

Dersom man ønsker datautveksling med innlogget ansatt, har vi flere valg:

1. API-tilbyder kan stole på det generiske "ansattporten:altinnressurs"-objektet
  - må validere på "ressurs"
  - bør kreve audience-begrensning, enten via tradisjonell `aud` eller ved `locations`-felt i autorisasjonsobjektet.

2. API-tilbyder ønsker å tilgangstyre hvilke klienter/konsumenter som skal kunne bruke APIet
  - må lage eget autorisasjonsobjekt, må lage enkel datastruktur som Ansattporten kan validere mot
  - tilgangstyring på samme måten som scopes idag
  
3. API-tilbyder ønsker å tilgangstyre hvilke bruker-valgte organisasjoner som skal bruke APIet
  - kan bruke [tjenesteeierstyrt rettighetsregister (SRR)](https://altinn.github.io/docs/api/tjenesteeiere/funksjonelle-scenario/#tjenesteeierstyrt-rettighetsregister) i Altinn 
    - Personer vil kun få organisasjoner/personer i sin avgiverliste som er eksplisitt gitt tilgang til tjenesten i SRR 
    - Ikke mulig for tilgangstyrer i virksomhet A å delegere tilgang til en SRR-styrt tjeneste hvis ikke A er gitt tilgang av i SRR av tjenesteeier
    - Eksisterende delegeringer slettes hvis tjenesteeier fjerner tilgang gitt i SRR  
  
For å utstede access_token vil inneholde samme struktur, men her kreves `locations`-claim i tillegg dersom den generiske "ansattporten"-prefixet skal trustes av APIet (*TODO! Hvorfor kreves locations/aud?*)

### Klient foreslår avgivere (var "Skattemelding næring"):

Fagsystemet kjenner sjølv avgivere kan bruke `avgiver_hint` som f.eks påvirker sortering/forhåndsvalg etc i brukerdialogen. Dette kan også komme med i `/tokeninfo/avgiverliste` slik at lokal aktørvelger kan identifisere de mest relevante aktørene.

