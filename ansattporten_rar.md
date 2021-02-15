# RAR i ID-porten

# Kva er RAR ?

RAR er ein ny Oauth2-utvidelse for transaksjonsspesifikke autorisasjonar:

* Publiserte draft:  https://tools.ietf.org/html/draft-ietf-oauth-rar-03
* Arbeidsdokument : https://github.com/oauthstuff/draft-oauth-rar/blob/master/main.md

Der "basic" Oauth2 kun gir tilgang til eit såkalt "scope" (tekst-streng), opnar RAR for tilgang til meir utvida datamodeller i form av **autorisasjonstyper**.

Dette blir utlevert i token som eit nytt fleir-nivå claim kalla `authorization_details` som igjen er ein array av autorisasjonsobjekter, der kvart objekt består av:
- standardiserte felt:
  - type (påkrevd felt, definerer den aktuelle autorisasjonstypen)
  - action
  - locations (tiltenkt mottakar =audience for tokenet)
  - identifier  (kan peike på ein konkret ressurs hjå RS)
  - datatypes (ein array med datatyper klient ønsker å få frå RS)
- eigendefinerte felt,
  - til ein gitt `type` vil det normalt vere definert og dokumentert ein tilhøyrande gyldig datamodell


MERK: Autorisasjonsserver skal avvise dersom klient førespur autorisasjonsobjekt med syntaks-feil eller ukjende element.

MERK 2: Autorisasjonsserver kan endre autorisasjonsobjekta i responsen, basert på brukaren sine handlingar i samtykke-dialogen.

**Døme på request:**
```
GET /authorize?response_type=code
      &client_id=s6BhdRkqt3
      &state=af0ifjsldkj
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
      &code_challenge_method=S256
      &code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bwc-uCHaoeK1t8U
      &authorization_details=%5B%7B%22type%22%3A%22account%5Finformati
      on%22%2C%22actions%22%3A%5B%22list%5Faccounts%22%2C%22read%5Fbal
      ances%22%2C%22read%5Ftransactions%22%5D%2C%22locations%22%3A%5B%
      22https%3A%2F%2Fexample%2Ecom%2Faccounts%22%5D%7D%5D HTTP/1.1
   Host: server.example.com
```
Ein skal helst ikkje bruke `scope` og `authorization_details` samstundes.

# Bruk av RAR i ansattporten

Me tenkjer at RAR er ein god underliggande protokoll for å løyse fleire av dei behova som er spelt inn til Ansattporten.  

Me ser for oss å definere eit antal autorisasjonstyper for dei ulike brukerreisene som er identifisert i prosjektet.  Dersom ein slik type er tilstades i autentiseringsforespørselen, så trigger dette ansattport-funksjonalitet.  Kvar autorisasjonstype vil føre til at det blir vist ein nærare definert "avgiver-velger" etter at innlogga brukar har autentisert seg.  

Me kan sjå for oss generiske ansattport-dialogar,  og kanskje også tenesteeigar-spesifikke dialoger.

# 1: Generisk ansattpålogging med Altinn

**Bruksmønster:** Tenesteeigar tilbyr ei nettside, der ein kun ønskjer at pålogging frå personar som har "ei bestemt rolle/representasjon" i Altinn, for ein organisasjon (=avgiver, normalt vil dette vere ein annan org en tenesteeigar sjølv).
* Definerer ein generisk autorisasjonstype `ansattporten:altinnressurs`
* Definerer ein URN-syntaks som identifiserer den etterspurte ressrurs/"rolla", alt etter om desse er Altinn2.0 eller 3.0

Ein minimums-førespurnad blir då slik:

```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:role:rolletypekode"
  }
]
```

der `ressurs` må følgje desse reglane:

|Ressurs-identifikator| Beskrivelse|Eksempel|
|-|-|-|
|urn:altinn:role:{rolletypekode} | Kan velge fra allowlist av [Altinn-rollene](https://www.altinn.no/api/metadata/roledefinitions?language=1044). | altinn:role:siskd
|urn:altinn:resource:{tjenestekode}:{tjenesteutgave} | Altinn 2 [tenestekode/utgåve](https://www.altinn.no/api/metadata?language=1044) | altinn:resource:3906:141205
|urn:altinn:resource:{org}:{appname} | Altinn 3 [org/app](https://www.altinn.no/api/metadata?language=1044) | altinn:resource:skd:sirius


> **Mange av dagens standard Altinn-roller gir veldig breie tilganger ("Post/arkiv", "Utfyller/innsender").**  Dette er problematisert med at de ikkje følger gode dataminimeringsprinsipp, og vanskeliggjør det å skulle holde oversikt over hva en gitt rolle faktisk gir tilgang til.  Derfor bør ein kanskje ikkje tilby rolle som muligheit i det heile, men heller kreve at det defineres tjenester som det spørres på. Ein mellomting er å berre godkjenna førespurnader på eit sterkt avgrensa sett med Altinn-roller.

Ansattporten vil så vise en organisasjonsvelger, som lister opp alle de organisasjoner der innlogget bruker har tilgang til den aktuelle ressursen i Altinn.  Bruker kan så velge en - og bare en - organisasjon.   Det resulterende id_token vil se slik ut:

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
    "avgiver": [{
        "Authority": "iso6523-actorid-upis",
        "ID": "0192:999888777"  // org.no til arbeidsgiveren som den innlogga brukeren har valgt i org.velger
    }]
  }
]
```

For å forbedre brukervenligheten, bør det være mulig å sende en rikere forespørsel som støtter forhåndsvalgt avgiver (dersom klient vet dette fra før) eller begrense avgiver-typer, slik at ikke bruker velger en person som avgiver når tjenesten bare gir mening for foretaks-representasjon for den gitte ressursen. Videre bør det (kanskje?) åpnes for at bruker kan kunne velge flere avgivere i samme autorisajon. Den fulle datamodellen for "generisk ansattpålogging"-forespørsel blir da:
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:[resource eller role]:[identifikator]",
    "mulige_avgivertyper": [ "foretak", "bedrift", "person"] // default: foretak
    "avgiver_hint": { [ avgiver i iso6523] }
    "tillat_flervalg": false //
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

### Avgiver-begresning

Kan vere hensiktsmessig å la klient åpne for mulighet til å velge flere representasjonsforhold, men viktig at brukeren har kontroll og får bestemme selv om dette faktisk skal skje.
```
    "tillat_flervalg": true //
```

Se eksempel på mulig avgiver-dialog her: https://app.moqups.com/Lj7L3ahE5T/view/page/ad64222d5

### Lokalt hurtigbytte av avgiver

Formål: gjere det enkelt for proff-brukere å kunne bytte avgiver raskt.  Slike brukere har mange (potensielle) avgivere, og det vil være uhengsiktmessig å redirect hele browseren til Ansattporten hele tiden (spesielt viss klienten er en desktop-applikasjon eller SPA.)

Istedet får brukeren mulighet til å velge  i et lokal GUI kven av dei andre mulege avgiverne ein ønskjer å representere.

Virkemåte:

#### 1. Førstegangs pålogging.
  * Ansattporten viser liste med potensielle representasjonsforhold (=kandidatliste).  
  * Bruker velger organisasjon på vanlig måte. Klient får token.
  * Ansattporten cacher hele kandidatliste knyttet til SSO-sesjon/autorisasjonen.

#### 2. Klient kaller dedikert endepunkt med id_token for hente kandidatliste

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

#### 3. Bruker velger ny kandidat lokalt
Avgiverlista presenteres for bruker i lokal GUI. Bruker velger en av kandidatene.

#### 4. Token-exchange
Brukers valgt blir inkludert som `ny_avgiver`-claim i et tokenexchange-kall ihht. [RFC8693](https://tools.ietf.org/html/rfc8693) mot Ansattporten.  

```
POST /tokenexchange
Host: ansattporten.no

  grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Atoken-exchange
  &subject_token_type=urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aid_token
  &subject_token=<opprinnelig id_token>
  &ny_avgiver=<iso6523-resprentasjon av valgt avgiver>

```
Dersom ny_avgiver finst i kandidatlista utsteder AP nytt token (samme autorisasjon, ny autorisajon ?)  kandidatliste oppfriskes mot Altinn dersom eldre enn x sec.  1 versjon:  rein stateless, kaller Altinn kvar gong kandidateendepunkt eller exchange-endepunkt


"korps kontra equinor-styreleder"-problmatikken -> bør vere eigen autorisasjontype.  eller brukerstyr swithc "ønsker å representere alle disse"  (kun en,  alle,  noen utvalgte)


Klienten må bruke samme klient-autentiseringsmetode mot /tokenexchange som mot /token.


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

### 2: Generisk datautvekling

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

#### Klient foreslår avgivere (var "Skattemelding næring"):

Fagsystemet kjenner sjølv avgivere kan bruke `avgiver_hint` som f.eks påvirker sortering/forhåndsvalg etc i brukerdialogen. Dette kan også komme med i `/tokeninfo/avgiverliste` slik at lokal aktørvelger kan identifisere de mest relevante aktørene.

### 3:
