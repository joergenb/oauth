# Abstract

This document describes a profile (extention?) of OpenID for Verifiable Presentations for use cases where there is no human in the loop. 

# 1. Introduction.

A Business Wallet is a kind of wallet where the Holder is an organization, and not a human being.  Compared to wallets for natural persons, buiness wallets are cloud-based software, and not apps running on mobile devices.    

OpenID4VP is based on an assumption that the same Holder is interacting with both the Verifier and the Wallet during presentation, and that the (human) Holder is using his web browser to orchestrate the presentation.  This assumption is not true for Buiness Wallets, which typically operate as automated systems without human involvement, thus there is no need for a browser-based flow. 

The profile described herein therefore skips the browser-based parts of openid4vp, by introducing a token endpoint for Wallets, from where Verifiers can request vp_tokens directly. A new grant_type for this purpose is also introduced. 

There might be presentation scenarios between Business Wallets where there is a need to get human involvement, for example getting human approval before presenting busines-critical credentials. This document defines processing rules for such scenarios. 


# 2. Terminology 

TBD

# 3. Overview

Both the Verifier and the Wallet publish metadata documents [RFC8414] containing their respective endpoint locations and capabilites. 
The means by which the Verifier learns the metadata location for a Wallet is out of scope.


The Verifies requests credentitals by POSTing a token request using the `urn:openid:vprequest` grant type, to the token endpoint of the Wallet.  If the Wallet contains the requested credentials, and Wallet policy allows for automated presentation, a `vp_token` containing the presentations is returned directly.   


```
 +--------------+                                    +--------------+
 |   Verifier   |                                    |    Wallet    |
 +--------------+                                    +--------------+
        |                                                   |
        |                                                   |
        |  (1) Token Request                                |
        |  (DCQL query)                                     |
        |-------------------------------------------------->|
        |                                                   |
        |                                                   |
        |  (2)   Token Response                             |
        |  (VP Token with Presentation(s))                  |
        |<--------------------------------------------------|
```

If the presentation request needs human intervention, the Wallet will inform the Verifer that the vp_token may be available at a later point in time,  and how the Verifier eventually will get hold of it.



# 4. Scope

This profile extends openid4vp and the underlying specifications in the following ways:

- mandates supporting wallets to offer a token endpoint
- introduces a new grant type `urn:openid:vprequest` for requesting presentations. See Section #5 for details.
- introduces new error codes and processing rules for the token endpoint 

# 5. Presentation Request grant

The Presentation Request grant is an extension of the the JWT authorization grant as specified by [RFC7523](https://datatracker.ietf.org/doc/html/rfc7523#section-2.1)

The value of the `grant_ype` is `urn:openid:vprequest`. 

The content of the grant is a presentation request as specified by the baseline openid4vp protocol, e.g it contains DCQL request, transaction data, client metadata, etc, with some additional processing rules:

- `response_type` should be only `vp_token`
- `redirect_uri` should not be present.  if provided, it should be ignored by the wallet
- client identifier prefix using redirect_uri should result in 400 bad request


The following is an example for the payload of a Presentation Request grant:
```
{
  "iss": "verifier_attestation:NOFOR.123456789",
  "aud": "https://businesswallet.example.com/SEBOLREG.987654321",
  "response_type": "vp_token",
  "client_id": "verifier_attestation:NOFOR.123456789",
  "dcql_query": {
    "credentials": [
      {
        "id": "some_identity_credential",
        "format": "dc+sd-jwt",
        "meta": {
          "vct_values": [ "https://credentials.example.com/identity_credential" ]
        },
        "claims": [
            {"path": ["family_name"]},
            {"path": ["given_name"]}
        ]
      }
    ]
  },
  "nonce": "n-0S6_WzA2Mj"
}
```


# 6. Token endpoint

Supporting wallets must offer a token endpoint.  The token endpoint must at least support the Presentation Request grant defined in Section 5.1.   Other grant types may be supported.  

Note:  Wallet implementations must ensure that the token endpoint can not be misused by Verifiers to buypass end-user authorization in deployments where also natural person wallets are supported.


Example of token request:
```
     POST /token HTTP/1.1
     Host: wallet.example.com
     Content-Type: application/x-www-form-urlencoded

     grant_type=urn%3Aopenid%3Avp_request
     &assertion=eyJhbGciOiJFUzI1[...omitted for brevity...]
```
where the payload of the JWT grant could look like example 5.1

## 6.1 Synchronous response

If the Wallet contains the requested credentials, and Wallet policy allows for automated presentation, a `vp_token` containing the presentations is returned directly in response to the presentation request.

If the Wallet policy prohibits this specific Verifer from being presented the requested credentials, TBD: ("dont try again"). 

## 6.2: Asynchoronous response

If the presentation request needs human intervention, the processing rules are TBD.  Some potential options are described below:

#### Option 1:  Direct post

If the presentation request needs human intervention, the Wallet will inform the Verifer that the vp_token may be posted back at a later stage.  To support this, the Verifier must include support for the `direct_post` mode. 

Example:
```
{
  "iss": "verifier_attestation:NOFOR.123456789",
  "aud": "https://businesswallet.example.com/SEBOLREG.987654321",
  "response_type": "vp_token",
  "response_mode": "vp_token direct_post"
  "response_uri": "https://some_other_wallet_provider.example.com/NOFOR.123456789/direct_post"
```

Note: In baseline VP, the current `direct_post` response mode instructs the Wallet to always post back, even when a synchronous response could be  possible.  To allow for a more flexible approach, we could either specify written processing rules here (eg: "if the wallet can directly respond, prefer that over a direct_post-response") Or we can introduce protocol signalling, fex. a new response_mode `vp_token` for syncronous operation, and let Verifier 

 (note 2: The definition of response-mode is tightly linked to the Authorization Endpoint, which is not in our case, problematic? 

> "The Response Mode request parameter response_mode informs the Authorization Server of the mechanism to be used for returning Authorization Response parameters from the Authorization Endpoint.

#### Option 2: Deferred presentation request

If the presentation request needs human intervention, the Wallet will inform the Verifer can get it a later point in time, but then from the Deferred Presentation endpoint of the Wallet.

The following parameters are used in the Deferred Credential Request:

transaction_id: REQUIRED. String identifying a Deferred Issuance transaction.
credential_response_encryption: OPTIONAL. as defined in Section 8.2 of OpenID4VP.

```
{
  "iss": "verifier_attestation:NOFOR.123456789",
  "aud": "https://businesswallet.example.com/SEBOLREG.987654321",
  "response_type": "vp_token",
  "transaction_id" : "1234555ndfhfdn",
}
```





# 7: Wallet-initiated presentation

