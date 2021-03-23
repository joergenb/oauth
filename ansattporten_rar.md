# Ansattporten og bruk av RAR

# Kva er RAR ?

Rich Authorization Requests (RAR) er ein ny Oauth2-utvidelse for transaksjonsspesifikke autorisasjonar:

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

### Forhold mellom scope og authorization_details

Ein skal helst ikkje bruke `scope` og `authorization_details` samstundes.

Dersom ein gjer det, må tildelt autorisasjon tolkast innskrenkande langs begge dimensjonar, slik at ikkje ein klient får rettar som brukaren ikkje meinte å gje den.

Dette vil i stor grad vere avhengig av god validering på API-sida.

# Bruk av RAR i ansattporten

Me tenkjer at RAR er ein god underliggande protokoll for å løyse fleire av dei behova som er spelt inn til Ansattporten.  

Me ser for oss å definere eit antal autorisasjonstyper for dei ulike brukerreisene som er identifisert i prosjektet.  Dersom ein slik type er tilstades i autentiseringsforespørselen, så trigger dette ansattport-funksjonalitet.  Kvar autorisasjonstype vil føre til at det blir vist ein nærare definert "avgiver-velger" etter at innlogga brukar har autentisert seg.

Me kan sjå for oss generiske ansattport-dialogar,  og kanskje også tenesteeigar-spesifikke dialoger.

# Brukerreise 1: Generisk ansattpålogging med Altinn

**Bruksmønster:** Tenesteeigar tilbyr ei nettside, der ein kun ønskjer at pålogging frå personar som har "ei bestemt rolle/representasjon" i Altinn, for ein organisasjon (=avgiver, normalt vil dette vere ein annan org enni tenesteeigar sjølv).

Brukerreise:

1. Bruker klikker login-knapp hos tjeneste.  Kallet til ansattporten inneholder informasjon om hvilket representasjonsforhold som tjenesten trenger
2. Bruker autentiserer seg med sterk eID.  Det opprettes ikke SSO-sesjon i Ansattporten.
3. Bruker vises en "avgiver-velger"-dialog, der hen kan velge hvilken avgiver (organisasjon, person) som denne innloggingen skal være på vegne av
4. Bruker blir sendt tilbake til tjenesten, med informasjon om valgt avgiver

Protokollmessig implementasjon:

* Definerer ein generisk autorisasjonstype `ansattporten:altinnressurs`
* Definerer ein URN-syntaks som identifiserer den etterspurte ressrurs/"rolla", alt etter om desse er Altinn2.0 eller 3.0

Ein minimums-førespurnad blir då slik:

```
GET /authorize

...vanlige oauth-paramtre (redirect_uri, client_id)...

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
"iss": "https://ansattporten.no/",
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

Merk at som default er ikke rettighet/operasjon-dimensjonen (les, skriv, ...) eksplisitt signallert, slik at de tjenester som trenger skille mellom ulike rettigheter, må be om dette eksplisitt. Se avsnitt nedenfor for hvordan dette blir løst.


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



### Tilgangsstyring på tjenestenivå

Fremfor å spørre på standardrollene er det i de fleste tilfeller å foretrekke at tjenesteeier oppretter en egen spesialisert autorisasjonsressurs i form av en lenketjeneste i Altinn. Tilgang til denne tjenesten kan (men må ikke) forhåndstildeles til et valgt sett med roller, f.eks. "Daglig leder". All annen tilgang må eksplisitt delegeres.


Request omtrent som for rolle:
```
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:resource:5129:1"
  }
]
```

### Flere søke-kriterier

Dersom det er naturlig at flere roller skal ha adgang til tjenesten, sender klient inn flere autorisasjonsobjekter.

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

I normaltilfellet bør bruker kun få lov til å bare velge en organisasjon.  Dersom bruker har flere av de forespurte rollene i valgt organisasjon, vil responsen inneholde flere autorisasjonsobjekter.

TODO: Avklare organisasjons-flervalg kombinert med multiple forespurte ressurser:  Det er andre brukstilfeller der man ønsker å representere mange avgivere (proffbrukere), såkalt "flervalg".  Dersom flervalg kombineres med multiple ressurser, blir det en utfordring å lage et forståelig design i avgiver-dialogen.   Ansattporten må passe på at det ikkje blir utlevert autorisasjons-objekter som er upresise, slik at mottaker misforstår hvilke rettigheter innlogga bruker faktisk har for de ulike avgiverne.

### Rettigheter/operasjoner

Altinn sin datamodell åpner for at en bruke kan tideles ulike rettigheter/opersjon på en ressurs.  Eksempel på slike er "les", "skriv", "signer".

For Ansattporten tror vi at de fleste tjenester ikke trenger å skille på denne dimensjonen, og har derfor valgt å ikke inkludere slik informasjon i protokollen som default etter følgende oppførsel: Dersom tjenesten forespør en ressurs som bruker rettigheter, så er det tilstrekkelig for Ansattporten at brukeren har 1 av disse satt for en avgiver, for at avgiveren havner i avgiverdialogen. I responsen vil Ansattporten kun utlevere ressursen, og ikke rettigheten(e).

Dersom tjenesten derimot eksplisitt spør etter rettigheter:
```
urn:altinn:resource:{tjenestekode}:{tjenesteutgave}:{rettighet}  
```
så vil avgiver-dialog populeres kun med avgivere som brukeren har den aktuelle rettigheten for,  og responsen vil også inkludere rettigheten:


```
"sub": "WE0DjFv9ygb2rjS7s_tXsg-fez2Co3Q8oxUmcvQ0mzQ=",
"iss": "https://ansattporten.no/",
"pid": "<fnr sluttbruker>",
...
"authorization_details": [
  {
    "type": "ansattporten:altinnressurs",
    "ressurs": "urn:altinn:3906:141205:les"
    "ressurs_name": "A01 a-melding",
    "avgiver": [{
        "Authority": "iso6523-actorid-upis",
        "ID": "0192:999888777"  // org.no til arbeidsgiveren som den innlogga brukeren har valgt i org.velger
    }]
  }
]
```

Tjenester bør unngå å spørre etter en kombinasjoner på "ressurs-nivå" og "rettighets-nivå", da dette trolig vil føre til utfordringer både i å lage en forståelig avgiver-dialog, men også i tolking av returnerte autorisasjonsobjekter.


### Flervalg og avgiver-begresning

Det kan vere hensiktsmessig å la klient åpne for mulighet til å velge flere representasjonsforhold, men viktig at brukeren har kontroll og får bestemme selv om dette faktisk skal skje.



```
    "tillat_flervalg": true     // kan velge flere avgivere i dialog
    "be_om_hurtigbytte": true  // be sluttbruker om å gi klienten lov til å bytte organisasjon (kan potensielt skje uten brukermedvirkning)
```

Se eksempel på mulig avgiver-dialog her: https://app.moqups.com/Lj7L3ahE5T/view/page/ad64222d5

TODO: her er en motsetning mellom bruksmønsteret for proff-brukere (regnskapsmedarbeidere) som kan representere opp til 100 organisasjoner på en dag, kontra privat-bruker (eksempelet med person som har signeringsrett BÅDE for stort foretak i jobbsammenheng OG for skolekorpset til eget barn,  og som selvsagt ikke ønsker at disse skal kunne blandes/misbrukes).  Det er kanskje mest realistisk å måtte innføre egne autorisasjonstype av type `ansattporten:proff` og ha egen avgiver-velger for dettte, enn å forsøke å konstruere EN dialog som skal håndtere begge typer? (men det hensyntar daikke privat-brukerens behov, dersom tjenesten valgte proff-bruker-dialog... )



### Lokalt hurtigbytte av avgiver

Formål: gjere det enkelt for proff-brukere å kunne bytte avgiver raskt.  Slike brukere har mange (potensielt flere hundre) avgivere, og det vil være uhengsiktmessig å redirecte hele browseren til Ansattporten hver gang brukeren skal bytte avgiver  (spesielt viss klienten er en desktop-applikasjon eller SPA.)

Istedet får brukeren mulighet til å velge  i et lokal GUI kven av dei andre mulege avgiverne ein ønskjer å representere.

Virkemåte:

#### 1. Førstegangs pålogging.
  * Ansattporten viser liste med potensielle representasjonsforhold (=kandidatliste).  
  * Bruker velger organisasjon på vanlig måte. Bruker må kanskje hake av for å bruke hurtigbytte for å hindre misbruk.  Klient får token.
  * Ansattporten cacher hele kandidatliste knyttet til autorisasjonen.

#### 2. Klient kaller dedikert endepunkt for hente kandidatliste

```
GET /tokeninfo/avgiverliste
Authorization: Bearer <aktivt access_token tilhørende bruker>
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
*TODO! Identifikatorer for personer. Kan vi eksponere en liste med fnr her sånn GDPR-messig ..?*

#### 3. Bruker velger ny kandidat lokalt
Avgiverlista presenteres for bruker i lokal GUI. Bruker velger en av kandidatene.

#### 4. Token-exchange
Brukers valgt blir inkludert som `ny_avgiver`-attributt i et tokenexchange-kall ihht. [RFC8693](https://tools.ietf.org/html/rfc8693) mot Ansattporten.  

```
POST /tokenexchange
Host: ansattporten.no

  grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Atoken-exchange
  &subject_token_type=urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aid_token
  &subject_token=<opprinnelig id_token>
  &requested_token_type=urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aid_token  // eller access_token
  &ny_avgiver=<iso6523-resprentasjon av valgt avgiver>

```
Dersom ny_avgiver finst i kandidatlista, og autorisasjonen tilknyttet `subject_token` fremdeles er aktiv, så utsteder AP nytt token i responsen:
```
HTTP 200 OK
Content-Type: application/json

{
    "access_token": "eyJ..""  <nytt_token, av forespurt type>,
    "issued_token_type": urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aid_token,  // eller access_token
    "token_type": "N_A",       // "Bearer" dersom issued_token_type=access_token
    "expires_in: 120           // Levetid, dersom issued_token_type=access_token
    "refresh_token": "sdc1..." // ?(er det behov for refresh)
}
```

Klienten må bruke samme klient-autentiseringsmetode mot /tokenexchange som mot /token.

Kandidatliste caches i Ansattporten i X minutter, oppfriskes mot Altinn dersom eldre. Oppfrisking kan kun redusere kandidatliste, ikke øke (for å hindre at klienten får tilgang til avgivere som brukeren ikke godtok i avgiverdialogen), dersom ikke kandidate-liste=<alle mine avgivere>.

Hurtigbytte må ikkje kunne misbrukast til å gje klienten tilgang på vegne av ein avgiver som sluttbrukar ikkje ønskjer (det er teknisk sett mogeleg for klient å gjere tokenexchange-kallet utan å faktisk vise lokal GUI til sluttbruker). Dette betyr (truleg) at klienten må aktivt forespørre støtte for hurtigbytte, og då må dette visast og aktiverast av sluttbruker i tilgangsdialog.





# Brukerreise 2: Generisk datautvekling

Dersom man ønsker datautveksling mellom virksomheter, der en API-tilbyder krever at det skal være en innlogget ansatt med eit visst representasjonsforhold hos konsumenten,  har vi flere valg:

1. API-tilbyder kan stole på det generiske "ansattporten:altinnressurs"-objektet
  - må validere på "ressurs"
  - bør kreve audience-begrensning, enten via tradisjonell `aud` eller ved `locations`-felt i autorisasjonsobjektet.

2. API-tilbyder ønsker å tilgangstyre hvilke klienter/konsumenter som skal kunne bruke APIet
  - må lage eget autorisasjonsobjekt, må lage enkel datastruktur som Ansattporten kan validere mot
  - TODO hvordan lage spesialiserte GUI
  - tilgangstyring på samme måten som scopes idag (via Samarbeidsportalen)

3. API-tilbyder ønsker å tilgangstyre hvilke bruker-valgte organisasjoner som skal bruke APIet
  - kan bruke [tjenesteeierstyrt rettighetsregister (SRR)](https://altinn.github.io/docs/api/tjenesteeiere/funksjonelle-scenario/#tjenesteeierstyrt-rettighetsregister) i Altinn
    - Personer vil kun få organisasjoner/personer i sin avgiverliste som er eksplisitt gitt tilgang til tjenesten i SRR
    - Ikke mulig for tilgangstyrer i virksomhet A å delegere tilgang til en SRR-styrt tjeneste hvis ikke A er gitt tilgang av i SRR av tjenesteeier
    - Eksisterende delegeringer slettes hvis tjenesteeier fjerner tilgang gitt i SRR  

For å utstede access_token vil inneholde samme struktur, men her kreves `locations`-claim i tillegg dersom den generiske "ansattporten"-prefixet skal trustes av APIet (*TODO! Hvorfor kreves locations/aud?*)



Dette mønster bør vere mogeleg å kombinere med pseudoymiserte tokens, tilsvarande slik dette er realisert med ID-porten idag.  Det velkjente eksemplet med "*mekaniker logger inn til bilverksted-fagsystem og kalle EU-kontroll-APIet til Statens Vegvesen*" er jo strengt tatt meir eit ansattport-scenario enn er ID-porten-scenarioet.




# Brukerreise 3: Informasjon fra AA-registeret

TBD

En ansatt er alltid knyttet til underenhet (virksomheten) og ikkje juridisk enhet (den opplysningspliktige).  
  - De orgnummerne som ID-porten/Maskinporten "kjenner", er normalt juridisk enhet (topp-nivå) - då det er desse som kan få utstedt virksomhetssertifikat.

Et arbeidsforhold har alltid bare yrkeskode, det er ikkje alltid intuitiv både kva kode som HR set på ein tilsett, det kan vere andre avhengigheiter som gjer at HR velger ein kode som ikkje er "passande" for ei tilganstyringsformål.  Då kan det bli utfordrande for tjenesteutviklere kven ein skal velge.  
  - kva skal skje om "forsker" blir daglig leder, då mister ein alle tilgangene?
  - men det er mogeleg å opprette *fleire* arbeidsforhold på same person (kan vere både "rektor" og "vaktmester" på samme skole)


Må ha en god innsikt i yrkes- og næringskoder-hierarkiet før en tar dette i bruk til tilgangstyringsbeslutninger.
    - Kan finne en "forsker" i en gren, og en annen "forsker" i en annen gren som du ikke forventa

Uklart korleis ein skal håndtere permisjoner.  (permisjons-id)

Det finst ogso eit felt "arbeidsforhold-id" som ikkje har noko med "ansatt-id" å gjere, kan t.d. bytte dersom bedriften bytter HR-system

AA-registeret (gjennom Altinn?) bør kanskje vere ein "opt-in", arbeidsgiver må bestemme om min org(hiererki) skal kunne brukes til tilgangsbeslutninger.


AA-registrert vil tilby oppslags-API.  Kanskje Ansattporten kan bruke dette direkte i starten.  På sikt vil aa-registeret vil tilby hendelseslister på individ-nivå.



TODO:
- videredistribusjon ?
- SAMTYKKE TIL Å UTLEVERE YRKESKODE  (og da må det være reelt samtykke, (du må kunne seie nei))







# Brukerreise 4: Punkt-innlogging uten SSO

I noen sektorer / arbeidssituasjoner vil det være av stor verdi bare det at Ansattporten kan tilby en "rein" punkt-autentisering som ikke fører til en SSO-sesjon.

Dette løses enkelt på protokoll-nivå ved at det tillates at klienter sender autentiseringsforespørsler uten `authorization_details` eller oauth2-scopes utover `openid`, at det da ikke vises noen organisasjonsvelger,  og at det da utleveres et "ID-porten-lignende"-id_token uten noen organisasjonsinformasjon, og da "https://ansattporten.no" som issuer.
