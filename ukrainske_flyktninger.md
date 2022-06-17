# API, opprette eID for ukrainske flyktninger

## Føremål
- at ein flyktning skal ha både eID og Digital Postkasse tilgjengeleg so tidleg som mogeleg etter ankomst til Noreg
- at prosessen skal vere enkel og brukarvenleg for både flyktningen sjølv og personellet som skal utføre

## Føresetnader:
- I første omgang skal berre deler av brukerreisa under realiserast, dvs. direkte-integrasjon UDI-Buypass for å opprette eID.
- Må vere enkelt for å utvide med full "flyktningeorkestrator"-funksjonalitet på eit seinare tidspunkt


## Brukerreise

1. Flyktning innstallerer eID-app (Buypass eller MinID, primært  på forhånd, når de er i «ventesona» før ankomstregistrering)

1. PU-saksbehandler gjennomfører ID-kontroll og trykker på «bestill» knapp i eget fagsystem (DUF)
  1. Sender API-kall til Digdir “flyktninge-orkestrator” med nødvendig informasjon
  1. Orkestrator sender opprettelses-kall til aktuell eID og får aktiveringskode i retur
    1. Dersom flyktning har tilstrekkeleg ID-dokument, vert Buypass valgt. Dersom ikkje, fallback til MinID.
  1. Orkestrator oppretter flyktning i KRR.
  1. Orkestrator oppretter digital postkasse hjå postkasse-leverandør
  1. Orkestrator sender ut ein mail på ukrainsk/russisk (og ein digital post) til flyktning med informasjon om eID og postkasse, og informasjon om korleis ein kan reservere seg mot digital post / fjerne postkasse / slette seg frå KRR.
  1. Orkestrator returnerer aktiveringskode til DUF

1. Fagsystemet viser aktiveringskoden til PU-saksbehandler

1. PU-saksbehandler verifiserer at flyktning taster inn aktiveringskoden, og at eid’en blir aktivert.




## API-design

Hovedelement:
- APIet er sikra med bearer tokens frå Maskinporten
  - dedikert oauth2-scope for formålet (`idporten:create_eid`) som Digdir kontrollerer tilgang til
  - konfidensialitet og integritet er elles sikra med vanleg TLS (med "god" cipher)

- vilkår for å få Buypass er at request inneheld naudsynt info om id-dokument i `evidence`-strukturen.    
    - Dersom dette manglar, vil orkestrator ustede MinID istaden.

- for å kunne opprette i KRR og digital postkasse (og MinID) må minst eitt kontaktinformasjon vere oppgitt

- forslag om å bruke [OpenID for Identity Aassurance](https://openid.net/specs/openid-connect-4-identity-assurance-1_0.html)-spesifikasjonen for å kode informasjon om id-dokumentet
  - alternativt så kan vi bruke "flate claims"


#### request

```
POST /eid
Host: eitelleranna.buypass.no
Authorization: Bearer xxxxxxx                                   //maskinporten-token med "idporten:opprett"

{
  "pid": "01010199999"

  "evidence": [                                                 //ID-dokument, ihht. OpenID for Identity Assurance
        {
           "type": "document",
           "method": "pipp",                                    // pipp = personleg oppmøte
           "verifier": {
             "organization": "Politiets Utlendingsenhet",       // navn eller organisasjonsnummer
             "pid" : "02020288888"                              // fødselsnummer til politimann/dame.
             "txn": "1aa05779-0775-470f-a5c4-9f1f5e56cf06"      // sporbar transaksjonsid i DUF sine systemer
            },

            "time": "2021-04-09T14:12Z",                        // tidspunkt for ID-kontroll / opprettelsesforespørsel  (bruke epoch istaden ?)

            "document_details": {
              "type": "idcard",
              "issuer": {
                "name": "Stadt Augsburg",
                "country": "DE"
              },
            "document_number": "53554554",
            "date_of_issuance": "2010-03-23",
            "date_of_expiry": "2020-03-22"

            }
          }
        ]

   "mobile": "+4799998888",                           // obligatorisk for å få postkasse (eller MinID)
   "email": "test@example.com"                        // valgfri
}
```


#### respons

200 OK  med aktiveringskode i json-body
400 bad reqeuest med error_description
401 utgått maskinporten-token
403 feil scope eller mismatch pid i token og request body


#### alterantiv utan OpenID IdAA-spec:


```
POST /eid
Host: eitelleranna.buypass.no
Authorization: Bearer xxxxxxx                        //maskinporten-token med "idporten:opprett"

{
  "pid": "01010199999"                               // fødselsnummer til flyktning
  "operator" : "02020288888"                         // fødselsnummer til politimann/dame.
  "txn": "1aa05779-0775-470f-a5c4-9f1f5e56cf06"      // sporbar transaksjonsid i DUF sine systemer
  "time": "2021-04-09T14:12Z",                       // tidspunkt for ID-kontroll / opprettelsesforespørsel  (bruke epoch istaden ?)
  "type": "idcard",                                  // trenger ubypass vite om det er pass eller idkort
  "document_number": "53554554",
  "date_of_issuance": "2010-03-23",
  "date_of_expiry": "2020-03-22"

  "mobile": "+4799998888",                           // obligatorisk for å få postkasse (eller MinID)
  "email": "test@example.com"                        // valgfri
}
