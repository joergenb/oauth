# Abstract

This document defines an extention on top of OpenID Connect and Oauth2 which let relying parties share session-related context attributes between them, intermediated by an OpenID Provider. 

# 1. Introduction

This mechanism is intended for use cases where the end-user journey spans two relying parties, where those RPs typically belong to different organizations (eg: security domains). Both RPs rely on the same OP. For the journey to be successful, parts of the context at the first RP need to be carried over to the second RP. This context can comprise both of attributes which are OP-controlled (ie: standard claims in an id_token from that OP), but also of RP-specific attributes unknown to the OP. 

For the user journey to be seamless, the user should preferably not need to re-authenticate when transfering to the second RP. Note that such decision is subject to OP policy and out-of-scope for this document.

*Example use case*: The user chooses to represent a company at the first RP, and this representation should be carried over when the user moves to the second RP.

See appendix A for more thourough examples.

# 2. Terminology

The terms client/relying party is used interchangably throughout the text.

This document defines the following terms:

* Context Transfer: an OP-controlled transfer of session-related claims/attributes between two Relying Parties.

* Receiving client: the destination relying party which receives the context.

* Originating client:  the relying party which initiates the context transfer.

* Contect Transfer Authentication:  The authentication process performed by the Receiving Client to complete the Context Transfer process.


# 3. Overview

There are two clients, A (originating) and B (receiving). They have up-front agreed on a certain user journey which spans both clients.  Both clients rely on the same OP for end-user authentication. All three parties operate in separate security domains (f.ex belong to different organizations).

The end-user log in at client A, establishing a central session at OP, and a local session at client A.  The end-user makes some actions at client A which establishes a specific *context*. Client A decides that the End-User must go to client B to proceed with his user journey. Client A requests a context transfer from the OP, then redirects the End-user to client B with the context transfer reference. Client B initates an authorization request towards the OP and includes the context transfer reference. Provided that the request as well as the validitity of the End-user session at the OP can meet policy requirements, the OP will grant the request and proceed with the flow without end-user interaction. The transferred context will be made available to the client B in the id_token and/or the userinfo endpoint, per OP policy. 

```mermaid
sequenceDiagram
  participant A as Originating client
  participant OP as OpenID Provider
  participant B as Receiving client


  note over A, OP: End-user authenticates 
  A-->>A: establish local context
  A->>+OP: (1) Context transfer request
  OP-->-A: context reference URI
  A-->>B: reference URI is transferred
  B->>+OP: Context Transfer Authentication
  OP-->>-B: id_token with context
  note over B: end-user is logged in with context

```
The core of this extension is to introduce a new Context Transfer endpoint at the OP, which is used by client A to initiate the process. This is descibed in chapter 5.  Also, a new value `ctx_transfer` for the `prompt` -parameter is introduced to be used by client B to complete the process. This is described in chapter 6.

# 4. Scope

TBD

# 5. Context Transfer Endpoint

The Context Transfer endpoint is used by clients to initiate a context transfer. A client POSTs its context content to the endpoint, and receives a context transfer reference in return. 

Client Authentication must be used at the Context Transfer Endpoint.  The OP must ensure a proper binding of the context transfer between the two clients and the End-user browser session.

## 5.1 Context Transfer request

The request is a set of claims:

* `client_context`: JSON object containing the client-controlled claims that the originating client wants to transfer to the receiving client
* `target_client_id`: Client identifier for the receiving client.
* `id_token_hint`: the id_token associated with the current end-user session at the originating client.  It will include any OP-controlled session claims, which should be carried over to the receiving client.
* `sub`: The subject value of the End-User. Note: value might be pairwise for the originating client. 

Either `id_token_hint` or `sub` must be present in the request, to ensure a proper binding between the End-user and the two clients.

Note that the originating client should not include OP-controlled session parameteres from the id_token as part of the client_context.  OP should remove any claims from the context which it deems are OP-controlled.

```
POST /client_context_endpoint
Host: op.example.com
Content-Type: Application/JSON

{
    "target_client_id": "xxxxxx",
    "id_token_hint": "eY...",
    "client_context": {
        "selected_company": "yyyyy",
        "process": "apply for building permit"
    }
}
```

## 5.2 Context Transfer response

The response contains two claims:

* `context_transfer_uri`:  An URI referencing this specific context
* `context_expires_in`: A JSON number that represents the lifetime of the context transfer URI

The context transfer URI should be kept relatively short (typically a few seconds).  The authentication process at client B must be completed within this expiry.

## 5.3 Unsuccessful responses

If the OP decides to reject the context transfer request, it returns error responses with the same format as for the token endpoint.

`no valid session`: the enduser assosiated with the sub or id_token does not have a valid session at the OP.
`transfer not allowed`: originating client is not allowed to perform context transfers.
`session not suitable for receiving client`: some of the OP-controlled session claims are not allowed to be transferred to the receiving client.
`session not suitable for transfer`: other characteristicas of the end-user session prohibits context transfer, per OP policy.


## 5.4 Transporting the context reference URI

The means of how the originating client transports the context tranfer URI to the receiving client is out-of-scope for this extension. 
Typically, a link will redirect the browser of the end-user to some dedicated URL which enables the receiving client to understand that context transfer authentication should be initiated.  The context transfer URI could be included as a fragment or query parameter on that URL.


# 6. Authorization at receiving client

To complete the context transfer, the receiving client performs a slightly modified OpenID Connect authentication flow towards the OP.  The mechanism is specified for the Authentication Code flow, and should work with both basic redirected flows, as well pushed authorization requests (PAR) or request objects based requests.  As the receiving client needs to perform client authentication before the OP can validate the binding, use of PAR or Request Objects are strongly recommended.

## 6.1 Authorization request

In the authorization request, the following parameters must be set to trigger context transfer authentation:

* `prompt` must be set to `context_transfer`
* `login_hint` must contain the Context Tranfer URI

There are no other restriction put on other OpenID Connect and Oauth2 authorization request parameters.

The OP must evaluate if the end-user session at the OP is valid and eligble for context transfer.  If client authentication is performed, the binding between the parties must be validated now.

The receiving client can request other scopes than the origiting client.  If those scopes require end-user approval, the OP must show a dialogue to gather such consent.

The following is an example request, where the authorization request is pushed:
```
POST /par HTTP/1.1
Host: idporten.no
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0Mzo3RmpmcDBaQnIxS3REUmJuZlZkbUl3

  prompt=context_transfer&
  login_hint=urn:idporten:ctx:jkj4kj342234n3434nnmnhasdf&
  response_type=code&
  state=af0ifjsldkj&
  nonce=asdfds234552&
  client_id=s6BhdRkqt3&
  redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb&
  code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bww-uCHaoeK1t8U&
  code_challenge_method=S256&
  scope=openid

```


## 6.1 Authorization request

The response is a normal response according the requested grant type, ie. a redirect back to the redirect_uri with a `code` value, which is used at the token endpoint to exchange for id_token and access_token. This concludes the process.

The id_token will contain the `client_context` from the originating client.  Note: the OP may remove individual claims from the JSON object per OP policy.

The access_token can, if relevant for the requested scopes, also contain the `client_context`, or parts thereof.




# XXX. Alternative solutions considered

can the use case be solved by other means,  existing standards, etc ?  


### Could CIBA be used instead ?

client a could send its id_token to client B, and then client b includes it in a ciba-flow towards the OP.
session parameters could be transfered directly between clients.

drawback: OP cannot verify the end-user session in the browser.


##### just use login_hint 

client A can send its id_token to client B, and then client_b includes it as a normal id_token_hint (or login_hint)

no way for the OP to know if this is a leak, an attack or a valid transfer.



#### use first-party apps draft

kinda similar mechanism, but assumes that clients belong to the same security domain, and often one of the cliens is a mobile app (public client)




# Appenidix A: examples

The End-user is filing his tax declaration online at the Norwegian Tax Administration (Client A). He notices an error about one of the buildings/properties he owns. To fix this, the information must be updated at the Authentic Source for properties, ie. The Norwegian Mapping Authorithy.  The user, still in tax declaration webservice, selects the property, and clicks "change".  The Tax Authority sends the user to the Norwegian Mapping Authorithy, where he is automatically signed in, the chosen property is pre-chosen  the property in question  
(ex. choosing a company to represent, selecting an item) 

Another example of context transfer which will be needed in Norway is representation - ex. the user choose a company to represent at client A, and must continue to represent this company at client B.   Or 


# Proposed solutions

# 0: bilateral solutions

To just use links between the two webservices will be open for both leakage of information, as well as open for a number of attacks.  There is need for both integritety and potentially confidentiality in the context which must be transfered from client A to B.

The two organizations can of course decide on a proprietary protocol and linking between themselves. However, if authentication and session managmement is not properly handled, they might unknowningly introduce security risks.  Therefore, it would be better building upon well-establised standards.


# old


graph LR
  A[Originating client]
  OP[OpenID Provider]
  B[Receiving client]

  A-. (1) authenticates .-> OP 
  A--. (2) POST context --> OP
  OP--. context reference UDI --> A
  A -. link .-> B
  B --. (3) authorization request --> OP
  