# 1: Opprette virksomhetsystem

*Integrasjonsansvarleg* hjå Systemleverandør (Visma) logger på Altinn portal. Ønsker å opprette nytt system.

Hen får sjå liste over eksisterende virksomhetssytem tilhøyrande Visma, og scopes som er tilordna Visma, ved at Altinn kallar
`GET /clients?integration_type="altinn_virksys")` og `GET /scopes/access/for_consumer?consumer_org=99988877)`

Integrasjonsansvarleg fyller ut naudsynt informasjon og klikkar "opprett". Altinn vil då kalle Maskinporten-sjølvbetjenings-API:
```
POST /client
Authorization: Bearer <token utsted til Altinn med idporten:dcr.write og idporten:dcr.supplier>

{
  "client_name": "Turboskatt for Øvre Toten Regnskapstjenester"
  "Description": "Her kan det stå ein lengre tekst om du vil"

  "scope": [ "skatteetaten:mva" ] ,
  "client_orgno": "<visma sitt orgno>"
  "integration_type": "altinn_virksys",
  "application_type": "web"

  //standard oauth dcr-claims:
  "token_endpoint_auth_method": "private_key_jwt",
  "grant_types": [ "urn:ietf:params:oauth:grant-type:jwt-bearer" ],
  ...
}
```

Maskinporten-sjølvbetjenings-API vil svare med ein tildelt `client_id`, dette blir då identifikatoren til virksomheitssystemet.  

### Om client_id

Normalt anbefalar me ikkje å bruke client_id til tilgangstyring hjå API-tilbydar, då dette claimet ihht spec er eit forhold berre mellom IDP og klient-organisasjonen. Organisasjonen må kunne endre sine klienter ("orden i eige hus") utan at API-tilbydarar skal bli påverka.  Men i dette tilfellet so snakkar me om eit meir "lukka" økosystem, der API-tilbydarar uansett må forholde seg til Altinn sin Autorisasjonsmodell, samt at vedlikehaldet av maskinporten-integrasjonane skjer gjennom Altinn, og me vurderer det då som føremålsteneleg å nytte client_id på denne måten.

### Krav til klient-autentisering

Ein Maskinporten-klient oppretta på trandisjonell måte vil som default tillate klient-autentisering med alle virksomheitssertifikat tilhøyrande konsument-organisasjonen. Dette for at de skal vere lett å kome i gong med Maskinporten.

Men dette leier til "universalnøkkel"-problematikk, og kombinert med at me forventar mange slike virksomheitssystem-registreringar, gjer me ei tilpassing i Maskinporten slik at det - for `integration_type=altinn_virksys` -  MÅ vere registrert ein asymmetrisk nøkkel på klienten før den kan brukast.

Prosessen for å opprette nøkkelparet på ein måte som føl beste praksis kan bli ei utfordring - her kan me sjå for oss fleire alternativ:

1. Altinn tilbyr eit GUI der Integrasjonsansvarleg kan lime inn ein publicnøkkel på PEM eller jwk-format.

2. Altinn tilbyr javascript-støtte for generering av nøkkelpar i browser, og så blir public-delen av nøkkelen sendt til ID-porten medan integrasjonsansvarleg må sjølv frakte nøkkelparet på ein trygg måte frå brower til det faktiske systemet. Fordel: privatnøkkel er aldri tilgjengeleg for Digdir.

3. Maskinporten sjølvbetjening generer ei p12-fil beskytta med sterkt passord. Passord til fila blir sendt på sms til integrasjonsansvarleg. Passord blir aldri logga eller synleg i Digdir sine system, som betyr at sjølv om me midlertidig behandler ein kopi av systemleverandøren sin privatnøkkel, kan me ikkje misbruke den. 
