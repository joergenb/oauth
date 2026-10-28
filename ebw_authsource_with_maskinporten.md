# Scenario

Securing Authentic Source APIs (ETSI 119 478) using the existing Norwegian national Oauth2-infrastructure

# Background

- Many APIs from Norwegian public sector bodies (ie. authentic sources) are secured by an existing, centralized solution called Maskinporten.

    - 63 agencies offer APIs
    - montly 4000-5000 companies consume the their APIs
    
- Maskinporten is an oauth2-server stricly for machine-to-machine communication.  It perfoms:

    - authentication of businesses based on their (qualified) electronic certificates (or other trusted key material), ie. RFC 7523

    - centralized access management (API owners can curate per-API-allowlists of business identifiers)

    - ie. it basically transforms a certificate into an access token


# Problem statement 

Article 45e of the amended eIDAS-Regulation (EU) No 910/2014 [i.33] requires that the verification
requests are "at the request of the user" 


# Option 1: Using ID-porten as AS


```mermaid
sequenceDiagram
    EU as End User (wallet unit)
    Q as QTSP
    AS as Authentic Source
    IDP as Authorization Server (ID-porten)

    EU->>Q: I want a credential
    Q->>Q: discovers athAS+EBWA: get metadata
    EBWA-->>-EBWB: response
    EBWB->>+EBWA: presentation request
    EBWA-->>-EBWB: presentation (vp_token)
```





- korleis sleppe dynamic registration

treng kunne stole på redirect_uri, og sign. algoritmer og client_id,  korleis unngå det ?
- using JAR / request objects

- wrpac eller signeringssert ?




# Option 2:  Using Maskinporten as AS




# Problem 1:  avoid client registration



- korleis identifisere brukaren ?
- korleis samle consten ?

How can an Norwegian Authentic Source authenicate any QTSP 


rerequisites


1. The presentation is automatic without human control; it can however be triggered by an authorized employee logging into the EBW and performing some GUI action, or the employee may have set up automation rules earlier which triggers automated presentation at a later stage.
2. How employees log in to the EBW is out-of-scope for the VP protocol, and is up to the discretion of the EBW-provider.  It can f.ex be based on presenting a PoR from their identity wallet.
3. The EBW initiating the protocol uses the European Digital Directory to find the endpoint (`iss` value) of the other EBW.

There are 2 variants of this scenario to consider:

# Variant 1: Verifier-initiated flow

This variant is similar to regular VP, which is always Verifier-initiated. Ie: EBW B acts as Verifier and asks EBW A acting as Holder for a credential.

1. Verifier fetches the client metadata of the Holder EBW from the well-known endpoint, based on `iss`.
2. Verifier creates a presentation request (ie. authorization request) towards Holder EBW's authorization endpoint.
3. Holder EBW validates the request and returns a presentation response (vp_token)

The flow is a simplied openid4vp flow where the end-user/browser parts are omitted, since there is no need to ensure that a human is in the loop to collect consent and exercise "sole control" according to eIDAS2.

```mermaid
sequenceDiagram
    participant EBWA as Business Wallet A (Holder)
    participant EBWB as Business Wallet B (Verifier)
    EBWB->>+EBWA: get metadata
    EBWA-->>-EBWB: response
    EBWB->>+EBWA: presentation request
    EBWA-->>-EBWB: presentation (vp_token)
```

Comparing against EUDIW, this draft proposes the following changes:
- browser redirects and/or the DC API are not used as they are not needed
- business wallet instances should publish their [client metadata on a well-known endpoint](https://datatracker.ietf.org/doc/draft-ietf-oauth-client-id-metadata-document/), including signing keys.  The EBWOID public key could be included in the metadata.

# Variant 2: Wallet-initiated flow

This flow is simply the reverse of variant 1; ie Holder EBW A sends a presentation directly to the Verifier EBW B.

This variant could alternatively be solved by adding an initial step to Variant A, in which Holder EBW A firsts asks Verifier EBW B to "please, can you ask me to present this credential".  


# Authentication of the verifier

### Option 1: Using WRPAC

Here, EBW instances uses a WRPAC for authenticaing the presentation request, by selecting `x509_hash` as Client Identifier Prefix, similar to normal EUDIW operation.   It is TBD if WRPRC (registration certififates) are also needed, depending on the coming Regulation, but we think they can be skipped as there is not personal data requested and no need to registrer the intent according to GDPR.

For large organizations with multiple WRP instances or multiple EBWs, using WRPAC may be the most feasible option, as separate software system can be more isolated. Also, the EUDIW concept of delegated authority to Intermediaries can be leverange

In this option, the EBWOID is not used. 

### Option 2: Using EBWOID directly

This may be a simpler option because EBW instances doesn't need to also get WRPACs.  The authorization request is signed with the EBWOID key. 

The Client Identifier Prefix could be `verifier_attestation`.  Note that this option place some requirements on the EBWOIDs: they must be in SD-JWT format, and some claims should be non-selectively disclosurable, namely `sub` (which should be set to the EU-ID, and used as original client identifier), `iss` (which is a qualified EAA provider) and `cnf` for the holder binding.   Also; how `iss` values should correspond to the Trust Services (QEAA-providers issuing EBWOID) on the Trust List must be solved. 

### Option 3: Wallet-provider attestation

Same as option 2, only that instead of EBWOID, we use an attestation signed by the Business Wallet-provider.  Trust List resolution might be simpler here, as Wallet Providers shall be put on the new ETSI 119 602 (json-based) trust lists and not ETSI 119 612 (xml, already used by eidas1). Thus, it could potentially be easier to extend / profile ETSI 119 602 to also include `iss` values of Trusted Entities, in addidition to signing keys.



# Open issues:

- Since all EBWs are online services with resolvable DNS (no mobile app), it seems beneficial that they also host their own client metadata, so that the other party can query their capabilities before initiating protocol exchange.
