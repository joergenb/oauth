Flyt:

```mermaid
sequenceDiagram;

   actor U as user
   participant W as wallet
   participant B as system <br/>browser
   participant C as NO PID Issuer <br/> (Credential Issuer)
   participant I as ID-porten <br/> (Authorization Server)
   participant F as Folkeregisteret
   participant WB as Wallet Backend
   participant T as NOBID Trust List

   
   W->>W: Generate key pair
   opt
      W->>+WB: initialize, request attestation (public key)
      WB-->>-W: key attestation JWT
   end
   U-->>W: Initiate enrollment
   W->>+T: GET issuers (country="NO", type="PID")
   T-->>-W: list of URLs of approved Credential Issuers
   U-->>W: selects issuer
   W->>+C: retrieve OpenID4VCI metadata
   C-->>-W: metadata, incl. IDP issuer url
   W->>W: can the issuer provide a PID in a supported format?
   opt If Identity Provider is different from Credential Issuer
       W-->>+I: retrieve OIDC metadata
       I-->>-W: metadata
   end
   
   W-->>B: initiate redirect
   B->>I: /authorize  (scope=eudiw:eu.europa.ec.eudiw.pid.no.1)
   
   note over U,I: user authenticates and consents
 
   I->>B: redirect with authorization code
   B->>W: claimed url with code
   
   W->>+I: /token (code, key attestation JWT, ?DPoP proof)
   opt If attestation-based client-auth was used 
     I-->>+T: verify attestation was signed by a trusted wallet (backend)
     T-->>-I: OK
   end
   I-->>-W: access_token 
   
   W->>+C: /credentials request (access-token, key proof, ?DPoP proof)
   C->>+F: lookup (fødselsnummer)
   F-->>-C: persondata
   C->>C: build the VC / mdoc
   C-->>-W: return the PID
```
