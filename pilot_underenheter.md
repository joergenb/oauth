# Pilot på adressering til underenhet

## Bakgrunn og formål

Vergemålsnemnda er en enhet som ikke oppfyller kravene til å bli registert i Enhetsregisteret. De kan derfor ikke få organisasjonnummer, og da heller ikke virksomhetssertifikat.  Juridisk er vergemålsnemnda underlagt Statens Sivilrettsforvaltning.  Se også [Digdirs veileder for virksomhetsautentisering](https://www.digdir.no/digitalisering-og-samordning/veileder-virksomhetsautentisering/2435).

Formålet med piloten er å teste ut hvordan Vergemålsnemnda skal kunne inngå i meldingsuteksling gjennom eFormidling.  Nærmere bestemt følgende to bruksområder:
* Vergemålsnemnda skal kunne signere en melding, sende til en mottaker, og mottaker skal kunne validere signaturen
* Mottaker skal kunne svare tilbake til Vergemålsnemnda. Svaret skal være kryptert slik at kun Vergemålsnemnda skal kunne åpne meldingen.


## Løsningsforslag:

* Integrasjonspunktet for eFormidling er "ytterpunkt" for konfidensialitet- og integritetsbeskyttelsen:
  * Integrasjonspunktet hos mottaker validerer signatur på mottatt melding, og sender usignert melding videre til fagsystem.
  * Integrasjonspunktet hos mottaker får svaret i klartekst fra fagsystem, finner Vergemålsnemnda sitt krypteringssertifikat, krypterer meldingen og sender til Vergemålsnemnda sitt integrasjonspunkt

* Tilgjengeliggjøring av nøkkel
  * I piloten bruker ikke Vergemålsnmenda et virksomhetssertifkat, men isteden en asymmetrisk nøkkel tilknyttet registreringen i Maskinporten.
  * Samme nøkkel benyttes både til signering og kryptering.
  * Nøkkelen må tilgjengliggjøres TBD (SR?, BCP?, annen plass ?)
  * Nøkkelen opprettes av Vergemålsnemnda selv, direkte på serveren som kjører Vergemålsnemnda sitt integrasjonspunkt.  Public-delen sendes Digdir som legger den manuelt inn i Maskinporten

Alternative løsningsforslag kunne vært å fått utstedt spesielle virksomhetssertikat med underenhet (OU-felt, se feks. [seid.2, leveranse 1, kap 6.2 ](https://www.nkom.no/internett/elektronisk-id-og-tillitstjenester/seid-prosjektet/_/attachment/download/9fe6dc19-74a8-44c1-99b6-652503d04e96:0ccc1b50d45e1141302df8f2f0e5d11c473f57b1/SEID%20Leveranse%201.pdf) og [info hos buypass](https://buypassdev.atlassian.net/wiki/spaces/BCA/pages/2182479873/Endringer+i+Buypass+personsertifikater+og+virksomhetssertifikater) )  og få registrert dette i BCP/BPL.



## Utviklingsoppgaver

### Maskinporten
Maskinpoten må utvides med "manuell" støtte for underenheter
  * Manuelt (via SQL) registere at en klient tilhører en underenhet
  * Kreve at klient med underenhet bruker asymmetrisk nøkkel til klient-autentisering
  * Bruke utvidet identifikator (`0192:986186999:MP//vergemålsnemnda:1`) i access_token
  * Det lages ingen støtte for selvbetjening av underenhet, eller tilgangstyring av scopes på underenhetsnivå som del av piloten

### Integrasjonpunktet

### Fagsystem hos mottakere

Ønske: ingen endringer.

Saksbehandler må kunne velge Vergemålsnemnda som mottaker.  Det er p.t. uklart om fagstystem støtter - eller ønsker å ha -  forhold hovedenhet-underenhet - eller om Vergemålsnemnda er en mottaker på "samme nivå" som andre.

 Enten så må fagsystemet klare å håndtere ISO6523-identifikatorer direkte, eller så må det på plass en mapping, enten fagsystemet selv, eller i Integrasjonspunktet. Mapping kan variere mellom ulike fagsystemleverandører.

### Fagsystem hos Vergemålsnmenda

Ingen endringer ?

### Andre applikasjoner/systemer
