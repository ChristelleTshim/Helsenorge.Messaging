# Helsenorge Messaging Feilkoder

I utgangspunktet s� pr�ver vi � gi relevante feilmeldinger som et menneske klarer � forst�; disse kan endre seg over tid. 
For feilmeldinger s� legger vi ved en ID som ikke endrer seg; denne kan da benyttes av maskiner for � identifisere en gitt feilmelding. 

Drift kan da benytte disse feilkodene til � bestemme hva slag varsling som man anser som n�dvendig. 

F�lgende feilkoder kan rapporteres. 

## Helsenorge.Registries

### REG-1
Problemer med � hente kommunikasjons detaljer.
 
### REG-2
Problemer med � finne CPP for en motpart

### REG-3
Problemer med � finne CPA for en motpart.

## Helsenorge.Messaging

### MUG-1
Generell feil med � motta meldinger.

### MUG-2
Generell feil med � sende meldinger.

### MUG-3
K�navnet er tomt.

### MUG-4
Avsender mangler i adresseregisteret.

### MUG-10
Mer enn en feil med avsenders sertifikat.

### MUG-11
Avsenders sertifikat har ugyldig start dato.

### MUG-12
Avsenders sertifikat har ugyldig slutt dato.

### MUG-13
Avsenders sertifikat har blitt revokert.

### MUG-14
Avsenders sertifikat har ugyldig type. f.eks. signeringssertifikat som brukes for kryptering.

### MUG-15
Mer enn en feil med lokalt sertifikat.

### MUG-16
Lokalt sertifikat har ugyldig start dato.

### MUG-17
Lokalt sertifikat har ugyldig slutt dato.

### MUG-18
Lokalt sertifikat har blitt revokert.

### MUG-19
Lokalt sertifikat har ugyldig type. f.eks. signeringssertifikat som brukes for kryptering.

### MUG-20
Mottatt melding er ikke XML.

### MUG-21
Mottatt melding mangler data i AMQP header.

### MUG-22
Mottatt melding har feil data i header kontra det som ligger i meldingen. Denne brukes av hodemelding for � sjekke at avsender id i fagmeldingen stemmer med det som st�r i AMQP header. 

### MUG-23
Meldingsmottaket har rapportert en feil som skal sendes til avsender. 

### MUG-30
Avsender svarte ikke p� en synkron melding innen en gitt tid (timeout).

### MUG-31
Avsender svarte p� en synkron melding etter at tiden gikk ut. Meldingen har ikke blitt prosessert.

### MUG-33
Ugyldig meldingsfunksjon.

### MUG-34
Feil som avsender har rapportert. Ting som kommer inn p� error k�en.

### MUG-35
Ukjent feil har oppst�tt.
