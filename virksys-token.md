# 4: Virksomhetssystem ber om token

Me ser for oss å støtte to mønster:

* Virksomheitstoken
* "tynne" token

## Virksomheitstoken

Virksomheitstoken er ein ny, "ferdigbakt" tokentype, som inneheld basis autorisasjonsinformasjon om kva rettar virksomheitssystemet har fått frå ein Part (som oftast gjennom ein "hjelper")

Flyten er slik:

```mermaid
sequenceDiagram;
  participant VS as Virksomhetssystem
  participant MP as Maskinporten
  participant AA as Altinn Autorisasjon
  participant API

  VS->>MP: førespør token (part, rettigheit?, hjelper?)
  note over MP: virksomhetsautentisering + scope-validering

  MP->>AA: kva RETTIGHEITER har virksys for PART ?<br/>evt: har virksys. tilgang til RETTIGHEIT for PART ?  <br/>(evt. via HJELPER)
  note over AA: Autorisasjonbeslutning
  AA->>MP: ja
  MP->>VS: virksomheitstoken
  VS->>API: førespør data med virksomheitstoken
  API->>VS:  returner data
```


Virksomheitssystemet må spesifisere at det ynskjer eit slikt type token ved å oppgje RAR-typen `urn:altinn:virksys` i requesten, og vidare kva **part** som token-requesten gjeld, typisk:
```
{
  "aud": "https://maskinporten.no/",
  "iss": <client_id for virksys.>,

  "scope": "skatteetaten:mva"

  "authorization_details": [ {
     "type": "urn:altinn:virksys",
     "part": "0192:999888777",       //orgno til parten  (forenkla, skal vere full iso6523-syntax)
   }
  ]
```

Maskinporten vil så spørje Altinn3 om dette representasjonsforholdet er tilatt, og dersom "ja", utstede eit virksomheitstoken, som er nesten identisk med requesten, men berika då med rettigheit(er) og hjelper (for sporingsformål hjå api-tilbydar):


```
{
  "iss": "https://maskinporten.no/",
  "client_id": <client_id for virksys.>,
  client_name": "Turboskatt for Øvre Toten Regnskapstjenester"

  "scope": "skatteetaten:mva"

  "authorization_details": [ {
     "type": "urn:altinn:virksys",
     "part": "0192:999888777",       //orgno til parten  
     "hjelper": "0192:888666555",    //orgno til hjelperen ?
     "rettigheit": [ "merverdiavgift", "skatt" ]              // TODO, kva skal dette eigentleg vere... ?
   }
  ]
```

TODO:  Datamodell for request og respons må drøftast nærare.

Det bør vere mogeleg for virksomheitssystemet å spørje både heile spissa på kombinasjonen part-rettigheit-hjelper" eller "åpent", dvs "kva rettigheiter har eg pva. denne parten", eller "gje meg X  partar som eg har rettigheit R for".


## Tynne token

Tynne token er heilt ordinære Maskinporten-tokens.   Men sidan me føl konvensjonen om at identifikator for virksomheitssystemet = client_id i Maskinporten,  so kan API-tilbydar ved mottak av tokenet gjere eit oppslag mot Altinn Autorisasjon for å sjekke om virksomheitssystemet er autorisert for aktuell handling.  Dette openar for at API-tilbydar kan sende meir komplekse autorisasjons-spørringar enn det standard-tokenet kan (og bør) støtte.


Flyten er slik:

```mermaid
sequenceDiagram;
  participant VS as Virksomhetssystem
  participant MP as Maskinporten
  participant AA as Altinn Autorisasjon
  participant API

  VS->>MP: førespør token
  note over MP: virksomhetsautentisering + scope-validering

  MP->>VS: vanleg maskinporten-token
  VS->>API: førespør data med virksomheitstoken
  API->>AA: autorisasjons-førespurnad (m/ virksys.id=client_id)
  AA->>API: Autorisasjonbeslutning

  API->>VS:  returner data
```

### Andre eigenskapar:

* MERK:  Systemleverandør kan IKKJE videre-delegere API-scopet til  driftsleverandør/databehandlar for denne token-typen, sidan klienten då er registrert på Driftsleverandør og det er ingen kobling til Systemleverandør.
    * Dersom Visma har fått tildelt skatteetaten:mva og delegerer dette vidare til Basefarm, betyr det samtundes at Basefarm skal kunne opptre som ALLE Visma sine virksomheitssystem?

* Det bør kanskje vere  mogeleg å inkludere fleire organisasjonar / rettar i samme virksomheitstoken for å unngå for mange nettverkskall. Dog risiko for veldig store tokens som kan gje problem med HTTP header størrelse, kan evt. vurder om CWT kan støttast i tillegg til JWT.
