# Mønstre

## 0: lommebok-til-lommebok

Dette er anteke det normale bruksmønstere for virksomheitslommeboka.  Ingen behov for Maskinporten.

```mermaid
graph LR

subgraph C [Konsument]
  F[Fagsystem]
  BW1[Virksomhetslommebok]
end


subgraph TE [Tenesteeigar]
  BW2[Virksomhetslommebok]
  F[Fagsystem]
end

subgraph EU [EU-kommisjonen]
  EDD[European Digital Directory]
end


F -->|1. 'send noko til nokon'| BW1
BW1 -->|2. 'finn lommebok-adresse'| EDD
BW1 --->|3. presenter bevis|BW2
```


## 1: Tradisjonell API-bruk 

Tenesteeigar har eit eksisterande API som er sikra med Maskinporten, ynskjer å kunne dele også med "virksomhetslommebok".

```mermaid
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


utfordringar:
- kva bevis er det som mappar til tenesteeigar sitt scope ?
- skal scope-tilganger framleis ligge i Maskinporten?

- har også ein variant #1b,  der fagsystemet snakkar først med lommmeboka for å få ein presetnasjon, for deretter å videresende denne til Maskinporten.  Litt rar?

## 2: Tradisjonell API-bruk 

Variant der fagsystemet Tenesteeigar har eit eksisterande API som er sikra med Maskinporten, ynskjer å kunne dele også med "virksomhetslommebok".

```mermaid
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
