# ID-porten utland -  utkast til protokoll


## Kva er idporten-utland ?

Det opprettes en ny nasjonal fellestjeneste "idporten-utland" som beriker google- eller apple-innlogginger med norske sektor-spesifikke identifikatorer, som feks Hjelpenummer.


Arkitekturmessig er løysinga basert på eit digdir-internt register (utlendingsregisteret), der ein elektronisk id kan linkast til forskjellige sektorspesifikke/nasjonale identifiktatorer, til verifiseringsinformasjon (som passlesing, eller sterk eID), og lagring av tilhøyrande attributter.  


## Highlights:

* Sjølsvtendig OIDC server, (feks issuer= https://utland.idporten.no/)
* Bruker OIDC med oauth2.1 i botn,  dvs authorization-code-flow med  PKCE påkrevd for “alt”.
  * ikkje støtte for PAR i første versjon

* klienter bruker primært scopes for å få de “ekstra-identifikatorene” de ønsker

    * t.d `idporten:utland:fhnummer` dersom klienten ønsker Nasjonalt Felles Hjelpenummer fra Personregisteret til Norsk Helsenett

    * dersom ikkje forespurt informasjon finst tilknytta brukar-id'en frå før, so vert det rekvirert frå autorativ kjelde der&da.

    * `openid profile` er ein gyldig kombinasjon, dvs. lov å logge inn utan noko beriking eller kobling


* klienter må håndere et bredt mulighetsrom av “sikkerhetsnivå”,  dvs. en innlogget bruker kan ha:

    * svak eller sterk identitetskontroll

    * svake eller sterke autentiseringsmekanismer

    * svake eller sterke koblinger til nasjonale/sektorvise norske identifikatorer




## Koding av id_token

Denne variasjonen i styrke og kilde for opplysninger må kodes ned i token på ein-eller-annan måte.

Dette kapittelet viser nokre døme.  

#### Alt 1 : minimumsløsying som berre dekkar "korona-innreise"

Flat struktur:

```
    {
      "sub" : "pairwise sub basert på intern bruker-id",
      "iss" : "https://c2id-demo.westeurope.cloudapp.azure.com/c2id",
      "aud" : "b7198ea6-14f0-488f-bb9a-206993ad28bc",
      "iat" : 1616765456,
      "exp" : 1616765576,
      "auth_time" : 1616765455,
      "nonce" : "qDHodUys-yzbA961atMXwfk3gBefUn27i7m4m-bEc_4",
      "sid" : "n3ss5nXYYkLRm8J_KZqP-pjsPli5AuKGDiAwRAn5nxE"
      "locale" : "nb",

      "acr" : "Level1",
      "amr" : [ "Apple-ID" ],  
      "fhnummer": "17012054321",

    }
```     

#### Alt 2: Flat struktur når bruker-id har mange attributter tilknytta

her er døme der helsepersonell har gjort ein fysisk id-kontroll (pipp) ved utdeling av FH-nummer på grensa:

```
    {
      "sub" : "pairwise sub basert på intern bruker-id",
      "iss" : "https://c2id-demo.westeurope.cloudapp.azure.com/c2id",
      "aud" : "b7198ea6-14f0-488f-bb9a-206993ad28bc",
      "iat" : 1616765456,
      "exp" : 1616765576,
      "auth_time" : 1616765455,
      "nonce" : "qDHodUys-yzbA961atMXwfk3gBefUn27i7m4m-bEc_4",
      "sid" : "n3ss5nXYYkLRm8J_KZqP-pjsPli5AuKGDiAwRAn5nxE"
      "locale" : "nb",

      "acr" : "Level1",
      "amr" : [ "Apple-ID" ],  
      "fhnummer": "17012054321",

      "id_document:number": "123456",
      "id_document:type": "passport",
      "id_document:verification": "pipp"
      "id_document:verification_location": "Gardermoen"
      "id_document:given_name": "Navn",
      "id_document:family_name": "Navnesen",

      "email": "epost@norge.no",
      "email_source": "self-registered"

    }
```     

Me ser at det blir utfordrande å kode "kilde" og "styrke" for ulike claims i ein strengt flat struktur.  

#### Alt 3: OpenID Identify Assuranse-struktur

Istadenfor å finne opp vår eigne hierarkiske struktur for "styrke" og "kilde" til claims, så er det truleg betre å ta i bruk [OpenID for Identity Assurance-spesifikasjonen](https://openid.net/specs/openid-connect-4-identity-assurance-1_0.html)

Men denne blir kanskje litt overkill for _alle_ attributtar?  Under eit døme der brukeren har logga inn med AppleID, fått FH-nummer frå NHN, har oppgitt nokre opplysningar sjølv, og har verifisert identiteten sin ved å scanne passet.)

```
{
  "sub" : "pairwise sub basert på intern bruker-id",
  "iss" : "https://c2id-demo.westeurope.cloudapp.azure.com/c2id",
  "aud" : "b7198ea6-14f0-488f-bb9a-206993ad28bc",
  "iat" : 1616765456,
  "exp" : 1616765576,
  "auth_time" : 1616765455,
  "nonce" : "qDHodUys-yzbA961atMXwfk3gBefUn27i7m4m-bEc_4",
  "sid" : "n3ss5nXYYkLRm8J_KZqP-pjsPli5AuKGDiAwRAn5nxE"
  "locale" : "nb",

  "acr" : "Level1",
  "amr" : [ "Apple-ID" ],  

  "verified_claims"; [
      {
        "verification": {
          "trust_framework": "norsk_helsenett"
        }
        "claims": {
          "fhnummer": "17012054321",          
        }
      },

      {
        "verification": {
          "trust_framework": "self-registered"
        },
        "claims": {
          "email": "epost@norge.no",
          "mobile": "+4799998888"
          "given_name": "Navn"
          "family_name": "Navnesen"
        }
      },

      {
        "verification": {
          "trust_framework": "idp"
          "evidence":
             "source": "https://appleid.apple.com."
        },
        "claims": {
          "sub": "WVWfdPmvXUvaBAyktZcmzEMhK",
          "email":      "enannenepost@icloud.com",
          "email_verified": "true"
          ""
        }
      },

      {          
        "verification": {
          "trust_framework": "passport-scan",
          "evidence": [ {
            "type": "id_document",
            "method": "epipp",  //
            "time": "2021-06-01T13:14:15TZD",
            "document": {
              "type": "passport",
              "number": "123456"
            }
          } ]  
        },
          "claims": {
            "name1": "NAVNESEN",
            "name2": "NAVN MELLOMNAVN"
          }
        }      
      }  
    }
```

#### Alt 4: Hybrid-løsning, IdAA for "sterke ting" og flatt for resten:

"Sterke" ting er då ting som passlesning som her:

```
    {
      "sub" : "pairwise sub basert på intern bruker-id",
      "iss" : "https://c2id-demo.westeurope.cloudapp.azure.com/c2id",
      "aud" : "b7198ea6-14f0-488f-bb9a-206993ad28bc",
      "iat" : 1616765456,
      "exp" : 1616765576,
      "auth_time" : 1616765455,
      "nonce" : "qDHodUys-yzbA961atMXwfk3gBefUn27i7m4m-bEc_4",
      "sid" : "n3ss5nXYYkLRm8J_KZqP-pjsPli5AuKGDiAwRAn5nxE"
      "locale" : "nb",

      "acr" : "Level1",
      "amr" : [ "Apple-ID" ],  
      "fhnummer": "17012054321",

      "given_name": "Navn",
      "family_name": "Navnesen",
      "email": "epost@norge.no",
      "mobile": "+4799998888"

      "verified_claims": { [     
        {
          "verification": {
            "trust_framework": "passport-scan",
            "evidence": [ {
              "type": "id_document",
              "method": "epipp",
              "time": "2021-06-01T13:14:15TZD",
              "document": {
                "type": "passport",
                "number": "PN123456",
                "issuer": {
                  "country": "CH"
                  "date_of_expiry": "2022-08-15"
                }
              }
            } ]  
          },
          "claims": {
            "primary_identifier": "NAVNESEN",   // kan vi alltid mappe primary til given_name ?
            "secondary_identifier": "NAVN",
            "personal_number": "01010154321",
            "nationalities": [ "CHN" ],
            "birthdate": "1999-01-01"
          }
        }
      ]
    }
```

Dersom brukeren over seinare skulle få D-nummer av FREG,  så er det ein teoretisk mogelegheit å også inkludere D-nummer i responsen, men det føreset at "personal_number" frå passet også vert registrert i FREG, og at idporten-utland skjønar på magisk vis at den må slå opp i FREG for denne brukaren.  


Ein annan variant er innlogging med nivå-høgt eID gjennom eIDAS:

```
    {
      "sub" : "pairwise sub basert på intern bruker-id",
      "iss" : "https://utland.idporten.no/c2id",
      "aud" : "min_client_id",
      "iat" : 1616765456,
      "exp" : 1616765576,
      "auth_time" : 1616765455,
      "nonce" : "qDHodUys-yzbA961atMXwfk3gBefUn27i7m4m-bEc_4",
      "sid" : "n3ss5nXYYkLRm8J_KZqP-pjsPli5AuKGDiAwRAn5nxE"
      "locale" : "nb",

      "acr" : "Level4",
      "amr" : [ "eidas" ],

      "fhnummer": "17012054321",
      "pid": "17012056789"

      "email": "epost@norge.no",
      "mobile": "+4799998888"

      "verified_claims": { [     
        {
          "verification": {
            "trust_framework": "eidas_ial_high"
          },
          "claims": {
            "familiy_name": "NAVNSEN",
            "given_name": "NAV",
            "eidas_identifier": "SE/NO/0110sdfbd",
            "birthdate": "1999-01-01"
          }
        }
      ]

    }
```


### mapping scopes - claims

UTKAST !!!

|scope|claims|verified-struktur|Eksempel|Navn|Forklaring|
|-|-|-|-|-|-|
|idporten:utland:fhnummer|fhnummer|Nei|01010154321|Nasjonalt Felles Hjelpenummer|FH-nummer finnes i Personregisteret til Norsk Helsenett. Utlendingsporten vil rekvirere et nytt FH-nummer for hver ny eID som logger på.|
|idporten:utland:kontaktinfo|email|Nei||Spør brukeren om å oppgi epost og mobilnummer. Lagres i u-reg.  Kan være forskjellig frå det som ligg i apple/google-id'en|
|| mobile|||
|idporten:utland:pass|claims ihht. ICAO9303 primary_identifier, secondary_identifier, personal_number, etc...|Ja||Trigger ein pass-scanning. Todo om det skal vere mogeleg å skilje på berre NFC-lesing, eller om ein også skal inkludere video-selfie|

TODO: bør bruke OIDC standard claims namn der det er mulig:

https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims

og so er det eit par nye med IdentityAssurance-spec’en ogos:

https://openid.net/specs/openid-connect-4-identity-assurance-1_0.html#name-additional-claims-about-end
