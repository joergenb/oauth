
#Digdir-camp-oppgåve om egen-kontrollert identitet og digital lommebok


## Motivasjon

Egen-kontrollert identitet (selv-sovereign identity, SSI) er en motreaksjon til store platformer (som facebook, twitter, ...) som samler inn masse data om oss, data som vi ikke har kontroll over hvordan blir brukt.

Grunntanken bak SSI er at brukeren selv skal ha kontroll på sin identitet og sine data.  SSI er altså et **tankesett**, en samfunnstrend, og ikke en teknisk løsning.  

For **kontroll på egen identitet** ser SSI-tilhengere for seg at istedet for “log in med facebook”, så skal brukeren ha en digital “lommebok”,  der en kan opprette en drøss med “sub-identiteter”, hver til sitt bruk, feks. en id for å logge inn på facebook, en annen mot twitter, en tredje mot offentlig sektor, egne id'er for hver nettbutikk, etc.

For **kontroll med egne data** ser man videre for seg å putte "bevis" inn i lommeboka. Bevis utstedt av ulike utstedere. For eksempel kan et universitet utstede et vitnemål til lommeboka til en student.    Brukeren  bestemmer selv, ved hjelp av lommeboka, hvis og når beviset skal deles med andre aktører.


## Oppgåve

Oppgåve er i stort å utforske økosystemet som SSI tilbyr, finne ut kor modent ulike deler av det er, og kome med anbefalingar på kva Digdir generelt og ID-porte spesielt bør gjere på dette feltet vidare.

Primært tenkjer me at oppgåva vert løyst i form av utvikling av ein proof-of-concept/pilot, samt dokumentasjon av funn og anbefalingar.

PoC bør ta utgangspunkt i eit konkret use-case for å demonstrere heile verdikjeda med bevis i digital lommebok, dvs. frå utstedelse av bevis til bruk av beviset opp mot ei digital teneste.  Mogelege kandidatar for use-casekan vere:


1. Ein kommune utsteder kortlevde bevis på at "lommebok-innehaver er sjukepleier", som Folkehelseinstituttet (FHI) kan stole på til å utlevere data om pasienter.

2. Alderskontroll: Folkeregisteret utsteder bevis på at bruker er myndig og beviset kan verifiserast av Vinmonopolet til å la brukar kjøpe øl / sprit.

3. Bevis på norsk statsborgerskap

4.



### Konkretisering av del-oppgåver:




#### 1: Lag ein utsteder

Lag ein web-applikasjon som utsteder bevis i form av VC.  

####

#### 2: Lag ein tjeneste

Tjenesten skal lese beviset som brukaren legg fram,  skal validere mot tillitsinfrastrukturen at utsteder er til å stole på.







ID-porten si primære rolle i denne arkitekturen må bli som utsteder av VC for sjølve «grunn-identiteten».
Vi kan lage en onboarding via to metodar,
vanleg innlogging med eID
passleser-funksjonaliteten frå minid-passport
Kan også vurdere å hente litt folkeregisterdata for å berike ID, evt. definere nokre Verifialble Presentations for «er over 18», «er norsk statsborger»
Det vil vere veldig nyttig for oss å forstå betre korleis brukeren av lommeboka kan bevise sin identitet, slik at vi kan utstede denne VC
Kva protokoll(er) finst mellom lommebok og id-porten ?
Og då korleis vi kan stole på at innbyggeren har valgt ei lommebok som er trygg&sikker.

Vi treng eit Verifiabla Data Registry
På sikt vil vel EU drifte dette, tenkjer eg,  og kvart medlemsland melder inn godkjente issuere.
Som sommarprosjekt må det vel vere tilstrekkeleg å berre ha ein skydatabase med public-nøkler til issuerene og brukerne
Dersom dere har noko frå før kan vi bruke den, men studentene våre får sikkert raskt opp noko her.

Vi treng ei lommebok !
Dette er det mest usikre punktet for meg - finst det nokon der ute som er VC-kompatibelt & klart?
Kva Data Registries støttes allereie ?
Kva bruker dere i aksjeeierbok-case ?
Og so hadde det vore kult med ein teneste som konsumerer desse VCene, då..




## Relevant bakgrunn

**Sylferskt:** EU vil utvikle en europeisk identitet basert på lommebok-tankegangen:   https://digital-strategy.ec.europa.eu/en/news/commission-proposes-trusted-and-secure-digital-identity-all-europeans

Norsk initiativ: https://www.din.foundation/

Stor internasjonal aktør: https://identity.foundation/

Pluss ein drøss andre aktører,  mange som skal "gjere noko med blokk-kjede eller kryptovaluta..."


## Spesifikasjoner, definisjoner

https://en.wikipedia.org/wiki/Self-sovereign_identity

https://en.wikipedia.org/wiki/Decentralized_identifiers

https://www.w3.org/TR/vc-data-model/

https://www.w3.org/TR/did-core/Overview.html

Merk at DID'er og VC'er ikkje er det same, at det KAN brukast saman, men ein kan fint bruke dei kvar for seg.


## Tekniske lenker, implementasjoner


https://docs.microsoft.com/en-us/azure/active-directory/verifiable-credentials/decentralized-identifier-overview

https://www.youtube.com/watch?v=62IYP1XtTYU
kan merke oss at her bruke dei *ikkje* DIDs og/eller blokkkjeder, men heller tradisjonell PKI til å etablere tillit mellom issuer og verifier.



https://medium.com/decentralized-identity/using-openid-connect-with-decentralized-identifiers-24733f6fa636

Dersom ein VERKELEG vil gå djupt, sjå proceedings frå førre Internet Identity Workshop: https://github.com/windley/IIW_homepage/raw/gh-pages/assets/proceedings/IIW_32_Book_of_Proceedings_Final%20A%201.pdf



# Mogelege oppgåver:

### Kva er vitkig for Digdir?

1. Korleis kan vi utstede VC?
  * Kva VC kan det vere aktuelt at Digdir utsteder?
2. Kva lommebøker finst, og korleis kan vi stole på at innbyggeren har valgt ei "trygg" lommebok ?
3. Korleis logger ein brukar inn med ei lommebok?
4. Korleis utvekslar vi data via ei lommebok i praksis ?



### Mogeleg use-case

## Oppgåver

1.


 FHI
