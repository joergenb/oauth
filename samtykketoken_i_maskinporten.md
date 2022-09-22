# Maskinporten som token-utsteder for samtykkeløysinga?

Det er tre prosesser knytta til [Samtykkeløsningen](https://altinn.github.io/docs/utviklingsguider/samtykke/) der det kan vere aktuelt å la ID-porten / Maskinporten utføre dei "oauth2-liknande" oppgåvene som skjer i Altinn idag:

1. Sluttbruker inngår samtykke
2. Datakonsument henter token
3. Datakilde validerer token og utleverer data


Me ser først på nr 2 og 3:

## Prosess 2+3: Samtykke-token frå Maskinporten


```mermaid
graph LR;
  Datakonsument-->|henter token|Maskinporten
  Maskinporten-->|forespør samtykke|Altinn Autorisasjon
  Maskinporten-->|returnerer token|Datakonsument
  Datakonsument-->|henter data|API
```


### Request:

Følgjande scope er naudsynt for å få ut samtykketokens (dvs. gjenbruk av dagens scope)

|claim|skildring|
|-|-|
|scope|Må vere `altinn:consenttokens` |

I tilegg trengs det innførast samtykke-spesifikke element i førespurnaden. Dette kan løysast på tre ulike måtar:

#### Alt 1: nytt oauth2 grant

Maskinporten innfører "samtykke" som eit nytt sokalla oauth2-grant: `urn:maskinporten:samtykke`

```
POST /token.oauth2 HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Amaskinporten%3Asamtykke
&assertion=eyJhb.....
```

der grantet (=`assertion`) er ein jwt på same måte som vanlege JWT-grant i Maskinporten ihht RFC7523, der følgjande claims må vere oppgitt:

ANTEN:
|claim|skildring|
|-|-|
|consent_id|Ein unik identifikator på sjølve samtykket.  (Dette vart tidlegare kalla `AuthorizationCode`, men endrar namn for å unngå forvirring med standard oauth-claim)|


ELLER:  (er dette mogeleg eller ønskjeleg  å få til?)

Viss datakonsument har mista id'en til samtykke-instansen, kan dei nytta grantet som ein oppslagsmekanisme for å sjekka om dei har framleis har samtykke frå brukaren :
|claim|skildring|
|-|-|
|OfferedBy|Fødselsnummer til sluttbruker. |
|CoveredBy|Orgnummer til datakonsument  (TRENG ME DENNE, KUNNE VORE PLUKKA FRÅ KLIENTAUTNETISERING?|
|ServiceCodes| Array med tjenestekoder som samtykket skal gjelde for. Døme: `[ "4629_2", "4630_2"` |

**Døme på JWT-body:**

```
"aud":"https://maskinporten.no",
"iss": "some_client_id",
"iat": ..
"exp": ..
"jti": ..
"scope": "altinn:consenttokens",
"consent_id": "c7dbe642-0fc1-4c3b-8959-8a92e3e1f17d"
```


#### Alt 2: Innføre token-type i Maskinporten

Maskinporten innfører `maskinporten_token_type` som eit valfritt claim i vanlege JWT-grants.  Dette claimet trigger so spesifikk oppførsel alt etter kva type som er førespurt. For samtykkeføremål so er verdien `urn:altinn:samtykke`)

TBD: bør type-definisjonane matche utgåande RAR-type (dvs. tjenesteeierspeisifkk urn:altinn...) eller ikkje?  (urn:maskinporten...)

**Døme på JWT-body:**

```
"aud":"https://maskinporten.no",
"iss": "some_client_id",
"iat": ..
"exp": ..
"jti": ..
"maskinporten_token_type": "urn:altinn:samtykke",
"scope": "altinn:consenttokens",
"consent_id": "c7dbe642-0fc1-4c3b-8959-8a92e3e1f17d"
```

#### Alt 3: mapping mellom scopes og token-typar

Ein variant av alt 2 der Maskinporten trigger på `"scope": "altinn:consenttokens")`.  Fordel: slepp oppgi eit ekstra claim i request.  Ulempe:  hard binding mellom kunde sine scope-definisjonar og oppførsel i Maskinporten.

#### Alt 4: token-exchange?

Her innfører vi IKKJE nye token-typer eller grants i maskinporten.  Men "dyttar ut" til datakonsument å veksle eit ordinært Maskinporten-token inn i eit samtykke-token:

```
POST /token HTTP/1.1
Host: maskinporten.no
Authorization: Basic cnMwODpsb25nLXNlY3VyZS1yYW5kb20tc2VjcmV0
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Atoken-exchange
&resource=https%3A%2F%2Fwww.altinn.no%2F
&requested_token_type=urn%3Aaltinn%3Asamtykke
&subject_token=<maskinporten-token>
&subject_token_type=
     urn%3Aietf%3Aparams%3Aoauth%3Atoken-type%3Aaccess_token

```

fordel med dette mønstere er at me (kanskje) kan holde maskinporten lettare ved å ha eigne mikrotjenster for ulike token-typar, (men det er ingen eigen tokenexchange-url publisert på .well-known, so då trengs uansett ein rutings-mekanisme)
det er truleg sett på som ein stor ulempe for kundane at dei alltid må først få eitt token, og så veksle inn i eit anna token.

#### Felles

**Bruk av databehandler** er basert på den vanlege [delegering i eOppslag-funksjonaliteten i Maskinporten](https://docs.digdir.no/docs/Maskinporten/maskinporten_func_delegering) for kunde-leverandørforhold.

Dvs. databehandlar som skal hente samtykke-tokens på vegne av data-konsument (behandlingsansvarlig), må autentisere seg med eige virksomheitssertifikat/nøkkel mot Maskinporten og be om på-vegne-av-token.  registerere ein maskinporten-klient om brukar eige virksomhetssertifikat/nøkkel klienten som henter token vere registrert på leverandør (databehandler) og bruke eige virksert/nøkkelgrantet i tillegg innehalde:

|claim|skildring|
|-|-|
|consumer_org| Organisasjonsnummer på dataValgfri, trigger vanleg kontroll av delegering for aktuelt scope|

(ER DET SLIK AT SAMTYKKET ER GJEVE TIL DATABEHANLDAR?  SO VISS BANKEN BYTTAR DATABHENDALAR MÅ INNBYGGAR GJE SAMTYKKE PÅ NYTT?)


**Ved mottak av request** vil Maskinporten sende eit kall til Altinn Autorisasjon for å hente ut sjølve samtykket, validere reponse, og bygge ein samtykke-json-struktur.


Feilmeldinger:
400 - bad request
400 - bad request - "noe feil mot altinn"  (truleg feil i Maskinporten og ikkje hjå konsument)
404 - consent_id not found
404 - det finst ikkje noko samtykke for kombinasjonen av coveredby og offeredby og servicecode
403 - forbidden: konsument / leverandør har ikkje lov til å hente samtykketoken
40x - leverandør har ikkje lov til å opptre på vegne av konsument (for scope, dvs. hente samtykketokens)



### Respons

Responsen er er eit samtykke-token frå Maskinporten.  Denne modifisert ihht dagens modell, for å likne meir på RAR-spesifikasjonen.


```
// standard oauth claims:
"iss": "https://maskinporten.no",
"iat": 1503860317,
"exp": 1503860347,
"jti": "a9daa57c-0b68-4ba2-a953-7567a99bd0f0",

// Informasjon om klienten som ba om tokenet:
"client_id": "asdkjflkdsajfsadlf",
"client_amr": "private_key_jwt",
"token_type": "Bearer",
"consumer" : {
   "authority" : "iso6523-actorid-upis",
   "ID" : "0192:991825827"
 }
 "supplier" : {
    "authority" : "iso6523-actorid-upis",
    "ID" : "0192:991825827"
  },
"delegation_source": "https://altinn.no"

// Informasjon om sjølve samtykket:
"authorization_details": [
    {
      "type":"urn:altinn:samtykke",
      "service_code": 4629,
      "service_edition": 2,
      "year": 2016,

      "consent_id": "c7dbe642-0fc1-4c3b-8959-8a92e3e1f17d",
      "OfferedBy": "11025802170",
      "CoveredBy": "910514458",
      "DelegatedDate": 1503855661,
      "ValidToDate": 1506760200,
    },
    {
      "type":"urn:altinn:samtykke",
      "service_code": 4630,
      "service_edition": 2,
      "from": "2017-06",
      "to": "2017-08",

      "consent_id": "c7dbe642-0fc1-4c3b-8959-8a92e3e1f17d",
      "OfferedBy": "11025802170",
      "CoveredBy": "910514458",
      "DelegatedDate": 1503855661,
      "ValidToDate": 1506760200,
    }

]
```



# Prosess 1:  Innbyggar inngår samtykke


TBD
