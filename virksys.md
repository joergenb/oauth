# Virksomhetssystem

Eit virksomheitssystem er ein spesiell type Maskinporten-integrasjon som er tilgjengeleggjort i Altinn3-platformen. Bruksområdet er tiltenkt "målretta" delegering i kunde-leverandør-forhold, typisk, men ikkje begrensa til, scenario der det inngår tre partar:  ein "part" - ein "hjelper" - ein systemleverandør.


# Bruker-reise

1. Systemleverandør tilbyr rekneskapsprogram ("Turboskatt") for rekneskapsbyråer.  Systemleverandøren oppretter eit (evt. fleire) virksomheitsystem ("Turboskatt landbruk", "Turboskatt VVS") gjennom Altinn for dette programmet, og gir systemet naudsynte API-tilgangar (vanlege oauth2 maskinporten scopes, t.d `skatteetaten:mva`)

2. Eit regnskapsbyrå (="hjelper") ynskjer so å ta i bruk Turboskatt for eit subsett av eigne kundar.  Rekneskapesbyrået oppretter ei eiga "kundemappe" i Altinn for dette. Ein bemyndiga hjå regnskapsbyrået koblar so kundemappa til virksomheitssystemet til Systemleverandøren. I denne prosessen vert det vist ei "samtykke"-side som informerer regnskapsbyrået om kva koblinga innebærer. (Systemleverandøren kan ogso "be om tilgang" og førehands-opprette ei kundemappe for regnskapsbyrået, dersom dette er meir hensiktsmessig).

3. Ein kunde-administrator hjå regnskapsbyrået **videre-delegerer** so aktuelle rettar "R1", "R2", etc... som regnskapsbyrået har fått frå eigne kundar til kundemappa.   (Til dømes skal "Reidun Røyrleggjar AS" og "Vidar Ventilasjon AS" puttast i kundemappa for VVS, medan "Beate Bonde" skal i landbruks-kundemappa.

4. Virksomheitssystemet skal so gjere eit API-kall som omhandlar data tilhøyrande Reidun Røyrlegger.   Det autentiserer seg som Systemleverandør opp mot Maskinporten for å hente eit token.  To ulike mønster er her tenkt støtta:

  * 4a. **Virksomheitstoken**.   Dette er ein ny token-type i Maskinporten.  Det fortel at det autentiserte virksomheitsystemet "Turboskatt" har - gjennom "hjelperen" - fått videredelegert rettigheita "R1" tilhøyrande Kunde ("Reidun Røyrleggar")

  * 4b. **Tynne token**. Dette er eit vanleg Maskinporten-token der berre id'en på virksomheitssystemet inngår. Når det aktuell APIet mottek tokenet, må det gjere eit kall til Altinn Autorisasjon for spørje om det aktuelle virksomheitssystemet har lov til å gjere aktuell API-handling.

  Dette mønsteret er mest passande der legacy-systemer går over til å bruke virksomheitssystem, eller der autorisasjonsreglane er sopassa komplekse at standard "virksomheitstoken" ikkje er tilstrekkeleg.

5. Dersom Regnskapsbyrået vil bytte systemleverandør, endrar dei berre kundemappa til å peike på den nye leverandøren sitt virksomheitssystem.  Gamal leverandør vil umiddelbart då misse alle rettar.

6. Dersom Kunden (Reidun Røyrleggjar) vil bytte regnskapsbyrå frå Øvre Toten Rekneskapstenester til Nedre Toten Rekneskapstenester, so endrar ho anten i Enhetsregisteret, eller bytter delegeringa av "X" i Altinn.  Då kaskade-slettast retten frå kundemappa til Øvre Toten.  Kunde-ansvarleg hjå Nedre Toten må - anten manuelt eller automatisert gjennom "innsalgsprosessen" -  videre-delegere rettane til eiga kundemappe.

# Eigenskapar

* Systemleverandør velger fritt om dei vil opprette eitt felles virksomheitssystem for alle rekneskapsbyrå, eller om dei føretrekk å ha ei virksomheitssystem-registrering per byrå.
* Kundemappe kan berre peike på 1 virksomheitssystem
* Regnskapsbyrå kan opprette so mange kundemapper dei finn føremålsteneleg.
* Konseptet vil virke fint også innad i same verksemd
  * I store verksemder vil dei truleg ikkje ynskjer at HR-systemet skal kunne ha ret
* ID på virksomheitssystemet er lik `client_id` i Maskinporten.  
* Virksomheitssystem er ein ny `integration_type` i Maskinporten.
* Virksomheitssystem MÅ bruke ein spesifikk nøkkel til klient-autentisering i Maskinporten
  * dvs. vilkårleg virksomheitssertifikat er ikkje tillatt
  * men det er ikkje forbudt å bruke eitt - og berre eitt - virksomhetssertifikat
* Det er mogeleg å inkludere fleire organisasjonar / rettar i samme virksomheitstoken
* Merk at "scope-delegering" IKKJE kan støttast for denne typen Maskinporten-integrasjonar.


# Arkiktektur



Virksomheitsystemet er altså mappa 1:1 til ein spesifikk Maskinporten-integrasjon. For å skape ein abstraksjon mellom virksomheitssytemet, og dei rettane/fullmaktene/delegeringane som systemet skal ha tilgang til, vert det innført eit konsept med "kundemappe" (eller "delegeringsmappe" el.)


# Teknisk løysing

Klikk på lenke for å sjå detaljar for kvart steg i prosessen:

* [Opprette virksomheitssystem](virksys - opprette.html)
