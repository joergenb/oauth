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

#### Basic ansattporten
vil kun ha pålogging av Lønn og personalmedarbeider,  klient kjenner ikkje org. frå før:

Request blir lik som dagens, med følgjande tillegg:
```
"authorization_details": [
  {
    "type": "ansattporten:rolle",
    "RoleId": 38069107
  }
]
```

Bruker velger så en - og bare en - organisasjon i organisasjonsvelger.
Respons i id_token

```
"sub": "WE0DjFv9ygb2rjS7s_tXsg-fez2Co3Q8oxUmcvQ0mzQ=",
"iss": "https://oidc.difi.no/idporten-oidc-provider/",
"pid": "<fnr sluttbruker>",
...
"authorization_details": [
  {
    "type": "ansattporten:rolle",
    "RoleId": 38069107,
    "RoleName": "L\u00f8nn og personalmedarbeider",
    "avgiver": {
        "Authority": "iso6523-actorid-upis",
        "ID": "0192:999888777"  // org.no til arbeidsgiveren som den innlogga brukeren har valgt i org.velger
    }
  }
]
```

Dersom det er naturlig at flere roller skal ha adgang til tjenestene, sender klient inn flere autorisasjonsobjekter.

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

#### Tjeneste-spesifikke "roller"

Dersom standardrollene ikke er formålstjenelig, må tjenesten opprette en egen "rolle" i form av en lenketjeneste i Altinn. (link til dokumentasjon).  


Request blir lik som dagens, med følgjande tillegg:
```
"authorization_details": [
  {
    "type": "ansattporten:tjeneste",
    "ServiceCode": 9107
    "ServiceEditionCode": 1
  }
]
```
Må opprettes som
#### Basic datautvekling

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

#### person-til-person representasjonsforhold

"logg inn på vegne av noen andre", eigen auth-type, for å trigge eige GUI i rar-dialog.


Request blir lik som dagens, med følgjande tillegg:
```
"authorization_details": [
  {
    "type": "ansattporten:representasjon",
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
        "Authority": "iso6523-actorid-upis",
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
