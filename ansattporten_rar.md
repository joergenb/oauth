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
    "ressurs": "urn:altinn:rolle:prokura"
  }
]
```


Full datamodell
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:rolle:prokura",
    "mulige_avgivertyper": [ "foretak", "underenhet" "person"] //default:foretak
    "avgiver_hint": { [ avgiver i iso6523] } //
    // trengs det noko hint på brukerstyrt begrensning
  }
]
```

Bør kun vere Altinn-3.0-kompatible ressurser.

|Ressurs-identifikator| Beskrivelse|
|-|-|
|urn:altinn:rolle:{rolletypekode} | Kan velge fra whitelist (ligg i Ansattporten) av Enhetsregistert og Altinn-rollene  (aktuelle er regnskapsfører). Dvs berre dei sentrale rollane som ikkje har for vide tilganger. |
|


For Enhetsregisteret er det typisk nokon med nøkkelroller / prokura/ (signeringsrett) som kan vere aktuelt å ha

Ein del av dagens standard altinn-roller gir veldig breie tilganger.  Dette er problematisert med at de ikkje følger gode dataminimeringsprinsipp.  Derfor bør ein kanskje ikkje tilby rolle som muligheit i det heile, men heller kreve at Arbeidsgiver tjenester



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
    "ressurs": "urn:altinn:rolle:prokura"
    "ressurs_name": "Noen med prokura",
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
    "ressurs": "urn:altinn:enhetsregisterrolle:prokura"
  },
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:enhetsregisterrolle:dagligleder"
  }
]
```

Organisasjonsvelger bør opplyse noe om dette, t.d.
> Denne tjenesten støtter kun pålogging av personer som har rollene
> * Lønn og personalmedarbeider
> * Daglig leder




Bruker kan fremdeles bare velge en organisasjon.  Dersom bruker har flere av de forespurte rollene i valgt organisasjon, vil responsen inneholde flere autorisasjonsobjekter.


TODO: dette er ganske Altinn-spesifikt,  er det mulig å lage ein noko meir generell modell ?   Eller blir det berre overengineering....
```
"authorization_details": [
  {
    "type": "ansattporten:generisk_rolle",
    "delegation_source": "https://www.altinn.no/"
    "RoleId": 38069107
  }
]
```

### Lokal hurtigbytte av avgiver

Formål: gjere det enkelt for proff-bukrere å kunne bytte avgire raskt.
bruker kan velge i lokal GUI kven av dei andre tokenene

1. Bruker velger org.  AP lagrer kandidatliste på autorisasjonen. Klient får token.
2. Klient kaller dedikert endepunkt for hente kandidatliste
3. Bruker velger ny kandidat i klient.
4. Klient utfører tokenexchange mot AP, dersom target-reportee finst i kandidatlista utsteder AP nytt token (samme autorisasjon, ny autorisajon ?)  kandidatliste oppfriskes mot Altinn dersom eldre enn x sec.  1 versjon:  rein stateless, kaller Altinn kvar gong kandidateendepunkt eller exchange-endepunkt


"korps kontra equinor-styreleder"-problmatikken -> bør vere eigen autorisasjontype.  eller brukerstyr swithc "ønsker å representere alle disse"  (kun en,  alle,  noen utvalgte)

```
[10:37] Langfors, Bjørn Dybvik
/tokeninfo/reporteecandidates
/tokenexchange?token=<token>&reportee=<fra kandidatliste>
```




### Tjeneste-spesifikke "roller"

Dersom standardrollene ikke er formålstjenelig, må tjenesten opprette en egen "rolle" i form av en lenketjeneste i Altinn. (link til dokumentasjon).  


Request omtrent som for rolle:
```
"authorization_details": [
  {
    "type": "ansattporten:tjeneste",
    "ServiceCode": 9107
    "ServiceEditionCode": 1
  }
]
```

### Basic datautvekling

Dersom man ønsker datautveksling med innlogget ansatt, har vi flere valg:

1. API-tilbyder kan stole på det generiske "ansattporten:rolle"-objektet
  - bør validere på RoleID
  - bør kreve audience-begrensning, enten via tradisjonell `aud` eller ved `locations`-felt i autorisasjonsobjektet.
2. API-tilbyder ønsker å tilgangstyre hvilke klienter som skal kunne bruke APIet
  - må lage eget autorisasjonsobjekt, må lage enkel datastruktur som Ansattporten kan validere mot
  - tilgangstyring på samme måten som scopes idag
3. API-tilbyder ønsker å tilgangstyre hvilke bruker-valgte organisasjoner som skal bruke APIet
  - altså: klient har anna orgno enn det som den innlogga brukeren velger i org.velger
  - sært tilfelle ?  eller meir normalt enn 2.
  -

For å utstede access_tokenvil inneholde samme struktur, men her kreves `locations`-claim i tillegg dersom den generiske "ansattporten"-prefixet skal trustes av APIet

### Person-til-person representasjonsforhold

"logg inn på vegne av noen andre", eigen auth-type, for å trigge eige GUI i rar-dialog.

Er dette eigen autorisasjonstype, eller bør det vere eit felt om "type avgivere som eg er interessert i"

Request blir lik som dagens, med følgjande tillegg:
```
"authorization_details": [
  {
    "type": "ansattporten:representasjon",
    ""
  }
]
```

Respons i id_token

```
"sub": "WE0DjFv9ygb2rjS7s_tXsg-fez2Co3Q8oxUmcvQ0mzQ=",
"iss": "https://oidc.difi.no/idporten-oidc-provider/",
"pid": "<fnr sluttbruker>",
...
"authorization_details": [
  {
    "type": "ansattporten:representasjon",
    "RoleId": 38069107
    "påvegneav": {
        "Authority": "norsk_fødselsnummer",
        "ID": "31129900000"  // fødselsnummer til den som innlogga bruker ønsker å representere (og har rettighet til å representere).
    }
  }
]
```

### Skattemelding næring:

Framlegget sendt til Skatteetaten.
Fagsystemet kjenner sjølv avgivere,  bruker seier "yay" eller "nay" til enkelt-element.

```
[
  {
    "type": "skatt:skattemelding_naering",
    "locations": [
      "https://altinn.no"
    ],
    "actions": [
      "read",
      "write"
    ],
    "avgiver": "01028012345",
  },
  {
    "type": "skatt:skattemelding_naering",
    "locations": [
      "https://altinn.no"
    ],
    "actions": [
      "read"
    ],
    "avgiver": "987654321",
  }
]
```
