# Scenario

A Business Wallet Instance A presents credentials to another Business Wallet Instance B

# Prerequisites

1. The presentation is automatic without human control; it can however be triggered by an authorized employee logging into the EBW and performing some GUI action, or the employee may have set up automation rules earlier which triggers automated presentation at a later stage.
2. How employees log in to the EBW is out-of-scope for the VP protocol, and is up to the discretion of the EBW-provider.  It can f.ex be based on presenting a PoR from their identity wallet.
3. The EBW initiating the protocol uses the European Digital Directory to find the endpoint (`iss` value) of the other EBW.

There are 2 variants of this scenario to consider:

# Variant 1: Verifier-initiated flow

This variant is similar to regular VP, which is always Verifier-initiated. Ie: EBW B asks EBW A for a credential.

1. The Business Wallet B fetches the client metadata of the Business Wallet A from the well-known endpoint, based on `iss`.
2. The Business Wallet B creates a presentation request (ie. authorization request) towards A's authorization endpoint.
3. Business Wallet A validates the request and returns a presentation response (vp_token)

The flow is a simplied openid4vp flow where the end-user/browser parts are omitted, since there is no need to ensure that a human is in the loop to collect consent and exercise "sole control" according to eIDAS2.

```mermaid
sequenceDiagram
    participant EBWA as Business Wallet A
    participant EBWB as Business Wallet B
    EBWB->>+EBWA: get metadata
    EBWA-->>-EBWB: response
    EBWB->>+EBWA: presentation request
    EBWA-->>-EBWB: presentation (vp_token)
```

Using this approach we can have the same mental model,  as well as protocol exchange, for issuing credentials both to Identity and Business wallets.

Comparing against EUDIW, this draft proposes the following changes:
- the authorization endpoint and credential offers are not used as they are not needed
- we foresee no or minimal changes to the credential endpoint
- the JWT that the EBW use to authenticate itself towards the token endpoint must be profiled/standardized

# Variant 2: Wallet-initiated flow

# Authentication of the wallet

The main open question in this proposal is how the business wallet instance authenicates itself towards the issuer.  Here, eventually, guidance from the coming Regulation and implementing acts needs to be considered, in order to create high-level requirement in a coming ARF, which then can be used for protocol design.

Nevertheless, some options are discussed below:

### Option 1: Present the EBWOID

The wallet authenticates by presenting its EBWOID to the Issuer. 
```
POST /token

grant_type=verifiable_presentation
&scope=reqeusted_credentials
&vp_token=["eyJhbGciOiJSU...", "other presentations", ...]
```
Here, a new grant_type is introduced, profiling which claims that must be in the request. The current proposal only intoduces a `vp_token` claim, where the wallet must include the presentation of credential(s) needed to satisfy issuer requirements. Processing rules for `vp_token` should be identical to its use in Openid4VP. 

In the normal case, the vp_token will contain an EBWOID in SD-JWT VC-format, including a key binding JWT which demonstrates that the Holder has proof-of-possession of the EBWOID key material.

### Option 2: WUA

Here, the Wallet instance includes a WUA from the Wallet Provider, attesting that the request comes from a valid EBW-instance. The request should also identify the EBW owner, which could be either by reference, or by also  proving posession of the EBWOID.

### Option 3: Using WRPAC

Another option might be to leverage Wallet-Relying Parties Access Certificates.  As the EBW probably need to be provisioned with WRPACs to act as a Verifier during VP-flows, it can then be beneficial to also use the WRPAC for authentication when the EBW is acting as a Holder.



# Motivation:

- The Oauth2 and OpenID protocols have already well-established features for machine-to-machine authorization, with large deployed ecosystems within e.x. OpenBanking and eHealth ecosystems around the world.   These features can easily be applied also to VP and VCI, yielding simplified protocols, which is what we propose in this draft.

- This proposal avoids workarounds seen in the wild trying to automate (and side-step...) the end-user consent and sole control happening in the browser.  Instead, we get a dedicated, simpler protocol variant for automated issuance.  This also gives Credential Issuers the to force business wallets to involve a human in scenarios where this is required, for instance for issueing certain high-value credentials.
  

# Open issues:

- Since all EBWs are online services with resolvable DNS (no mobile app), it seems beneficial that they also host their own client metadata, so that the other party can query their capabilities before initiating protocol exchange.
- 

- **Holder binding:**  For EUDIW, there is a significant complexity coming from privacy protections, where every credential is issued batches where each credential issue must have unique key material.  For EBW, can we instead bind all credentials to the same EBWOID key ?  Do we need batch issuance ?

- We assume that WRPAC and WRPRC are used by the Issuer in the same manner as for EUDIW (ie: put WRPRC inside metadata and sign it using WRPAC)

  
