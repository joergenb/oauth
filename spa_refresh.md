# Problem

1. ID-porten tillet ikkje at public-klienter kan få refresh_token, sidan dei ikkje kan bevise kven dei er (klient-autentisering) ved forsøk på refresh.

2. ID-portens støttar ikkje "silent renewal" (prompt=none), pga. begresningar i produktet som ID-porten er basert på.

3. Browsere innfører strengare handtering av 3djeparts-cookies, som gjer at iframe-baserte metodar som "silent renewal" uansett ikkje vil  fungere i framtida
https://leastprivilege.com/2020/03/31/spas-are-dead/ :

> What this basically means is, that browser are getting more and more strict with how they handle their cookies. [...]  the immediate consequences will be:
>
> * front-channel logout notifications do not work anymore (used in pretty much every authentication protocol – like SAML, WS-Fed and OpenID Connect)
>* the OpenID Connect JavaScript session notifications don’t working anymore
>* the “silent renew” technique that was recommended so far to give your application session bound token refreshing don’t work anymore
>
> Safari and Brave are the first browser implementing those changes. Chrome will follow in 2022 (hopefully sooner) etc…
>
>Some things can be fixed, e.g. you can replace front-channel notifications with back-channel ones. Some people recommend replacing silent renew with refresh tokens. This is dangerous advice – even if your token service has implemented countermeasures.

#  Mogelege løysingar


### 1: Kunde må bruke backend-for-frontend arkitektur

Dominick Baier over er ganske klar på at det er dette han anbefalar...

Dvs. lage ein tynn backend mellom SPA og ID-porten. BFFen blir då ein confidential oauth-klient sett frå idporten og kan få refresh_token.  Transparent ruting av API-kall gjennom BFF-en.

Kunden må jo uansett tilby eit domene som SPAen skal lastast frå, så å utvide denne med ein tynn backend burde vere overkommeleg.  Fordelen er at ein får høgare sikkerheit kring tokens.  Men ulempa er at det medfører behov for forvaltningsregime rundt backend, behandling av personopplysningar når API-kall går gjennom BFF - men kunden slepp vel ikkje unna dette ansvaret sjølv om ved ein "rein" SPA.


### 2: Utlevere refresh_token til public klienter

ID-porten endrer oppførsel slik at dersom ein klient har `application_type=browser`, så får den også lov til å motta eit refresh_token.

Men for å ivareta tilstrekkelig sikkerheit, bør vi innføre tilleggstiltak i tråd med oppdaterte retningslinjer fra IETF, dvs.
* OAuth 2.0 Security Best Current Practice (https://tools.ietf.org/html/draft-ietf-oauth-security-topics-16#section-4.13.2)
* OAuth 2.0 for Browser-Based Apps (https://tools.ietf.org/html/draft-ietf-oauth-browser-based-apps-07#section-8)
* Den kommande Oauth 2.1-standarden (https://tools.ietf.org/html/draft-ietf-oauth-v2-1-00#section-6.1) (som igjen baserer seg på de to foregåande dokumentene)


Potensielle tilleggstiltak:

* **rotere refresh_token**  Kvar gong eit refresh_token blir brukt, blir det laga eit nytt refresh_token og det gamle invalidert.
* **Rotasjon med replay-deteksjon**  Som over, men gamle refresh_token blir lagra i ID-porten so lenge autorisasjonen varar.  Dersom gamle refresh_token blir forsøkt brukt, blir alt invalidert.
* **avsender-begrensa refresh_token** Binde refresh_tokenet til avsender slik at avsender må bevise besittelse av et privatnøkkel ved bruk, anten til TLS-laget gjennom https://tools.ietf.org/html/rfc8705,  eller på applikasjonslag med DPoP (https://tools.ietf.org/html/draft-ietf-oauth-dpop-02#section-5)
* **Invalidere autorisasjon ved inaktivet**  Dersom SPAen ikkje fornyer refresh_token innafor ein viss tid som er "rimeleg", (td. 50 ganger levetida til access_token), så blir heile autorisasjonen og tilhøyrande token invalidert av ID-porten.  (Døme: autorisasjon varar 24 timar, access_token er 30 sekund,  dersom refresh ikkje er brukt på 50*30 = 25 minutt, så blir autorisasjon ugyldig.).  Slik oppførsel kan ein få til ved å tune levetidsparametre idag, men API-tilbder kan ikkje tvinge det, så alternativt kan ein vurdere å innføre refresh_token_max_lifetime på scopet.

Dominick meiner på https://leastprivilege.com/2020/01/21/hardening-refresh-tokens/  at ein også bør:

* kreve samtykke-dialog til autorisasjonen før klient kan få refresh-token


### 3: Offline-access for å trigge refreshtoken

OIDC har definert eit eige scope for å trigge såkalla offline access.
https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess

  Det er uklart for oss korleis dette heng saman med silent renewal-metodene.  Bør offline_access vere det som trigger at klienten faktisk får eit refresh token ?

spec'messig så er det berre /userinfo-endepunktet som det er spesifisert at scopet skal gi tilgang til.

Det er også slik at i dei tilfella som har vore rapportert til oss, so har det alltid vore snakk om at brukaren faktisk er til stades i perioden.
