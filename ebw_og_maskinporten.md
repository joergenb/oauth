Mønstre

```
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW[Virksomhetslommebok]
end

subgraph Felles [Fellesløysingar]
  MP[Maskinporten]
  SA[(tilganger)]
end

subgraph TE [Tenesteeigar]
  A[API]
end

TE -->|konfigurerer|SA
F -->|1.ber om token| BW
BW -->|2. presenterer bevis| MP
MP -->|3. sjekker|SA
F ----->|3. sender request til|A
```
