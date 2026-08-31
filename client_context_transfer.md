Client Context Transfer

# Overview

There are two clients, A and B, belonging to different organizations (e.g. security domains).  They have up-front agreed on a certain user journey which spans both clients.
Both clients rely on the same AS/IDP, operated by a third organization (e.g. another security domain).

The end-user starts at client A, and makes some actions there which establishes a specific *context*. Client A decides that the End-User must go to client B to proceed with his actions to complete the journey. Client A sends the End-user to cliente B, leveraging the IDP to transfer the context between them, the end-user is automatically logged in at client B, and the context is automatically establied, so that the end-user does not have to repeat the same actions taken at A. 

# Example

The End-user is filing his tax declaration online at the Norwegian Tax Administration (Client A). He notices an error about one of the buildings/properties he owns. To fix this, the information must be updated at the Authentic Source for properties, ie. The Norwegian Mapping Authorithy.  The user, still in tax declaration webservice, selects the property, and clicks "change".  The Tax Authority sends the user to the Norwegian Mapping Authorithy, where he is automatically signed in, the chosen property is pre-chosen  the property in question  
(ex. choosing a company to represent, selecting an item) 

Another example of context transfer which will be needed in Norway is representation - ex. the user choose a company to represent at client A, and must continue to represent this company at client B.   Or 


# Proposed solutions

# 0: bilateral solutions

To just use links between the two webservices will be open for both leakage of information, as well as open for a number of attacks.  There is need for both integritety and potentially confidentiality in the context which must be transfered from client A to B.

The two organizations can of course decide on a proprietary protocol and linking between themselves. However, if authentication and session managmement is not properly handled, they might unknowningly introduce security risks.  Therefore, it would be better building upon well-establised standards.


# 1: context_transfer as a login_hint

Client A informs the IDP that it want to establish a context, by POSTing its content to a new Context-transfer endpoint.  To ensure a proper binding of the session, the `client_id` of the target client B must be included, as well as the `sub` of the user.
Upon successful validation, the IDP responds with a context identifier, `context_id` and an `context_expires_in` expiry timer.
Client A redirects the end-user to Client B, along with the context identifier.  
Client B performs an authorization request to the IDP, including the `context_id` as a `login_hint`.  The request must be pushed, or use JAR/request object, so the IDP can perform client authentication.  The IDP validates that the request comes from the correct target client for the specific `context_id`, that it still has a valid browser session with the end-user and that this user is the subject of the requested context, and finally that the context has not expired. 

