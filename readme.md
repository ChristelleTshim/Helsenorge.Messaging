# Helsenorge Messaging

## Oversikt

Et av integrasjonalternativene mot Helsenorge er � sende XML meldinger via et sett med k�er. Denne pakken gj�r det lettere � integrere seg via denne kanalen. 
Denne pakken h�ndterer f�lgende funksjoner:

### Sending og mottak av meldinger
Pakken st�tter det som kalles asynkrone og synkrone meldinger. Asynkrone meldinger sendes og en gang i fremtiden s� kan det komme et svar. 
Synkrone meldinger er teknisk sett asynkrone, men funksjonelt synkrone; man blokkerer inntil man f�r et svar eller timer ut.

Pakken st�tter b�de mottak og sending av meldinger. Noen ganger s� kan man tenke seg at man bare trenger sending eller mottak, men i praksis s� trenger man begge. 
Ofte s� vil applikasjonslaget kreve kvitteringmeldinger som skal mottas dersom man sender, eller sendes dersom man mottar. 

Pakken er bygd slik at man kan registrer callbacks n�r man mottar de forskjellige meldingstypene. Applikasjonslaget st�r da fritt til � gj�re det den vil med meldingen.

Dersom en melding feiler i mottak, s� blir meldingen liggende igjen p� k�en og vi pr�ver igjen (maks 10 ganger). Unntaket er feil som skal varsles til avsender. 

### Registre
For � kunne kommunisere med andre parter, s� trenger vi � vite hvor meldinger skal sendes. Dette er informasjon som hentes ut fra forskjellige registre.
Pakken er avhengig av disse registerintegrasjonene.

### Kryptering og signering
Dataene som ligger i meldingene blir f�rst signert, s� kryptert. Sertifikatinfrastrukturen baserer seg p� informasjon i adresseregisteret samt private sertifikater.
Pakken st�tter kryptering, dekryptering, signering, og signatur validering.

### Feilh�ndtering
Pakken har bred st�tte rundt feilh�ndtering. 
- Alle kommunikasjons parter har en error-k� der meldinger som feilet blir sendt. Ekstra data om feilen blir inkludert i meldings headeren.
- Avsender blir varslet dersom sertifikatene de benytter ikke lenger er gyldige. 
- Applikasjonslaget kan varsle avsender dersom XML ikke validerer
- Applikasjonslaget kan varsle avsender dersom den f�r data som ikke stemmer. f.eks. forskjell mellom det som kommer i header og det som ligger i XML.
- Applikasjonslaget kan varsle avsender om mere generelle feil.

Feil som mottas p� v�r error-k� varsles via en egen callback. Hva systemet �nsker � gj�re med disse kan variere. Helsenorge logger disse slik at driftspersonell kan feils�ke.

### Logging
All kommunikasjon med k�er og registre logges. Feil blir ogs� logget med egen definert EventId.

## Forutsetninger

F�r du kan ta i bruk denne pakke, s� er det en del forutsetninger som m� v�re p� plass. 

- L�sningen din m� st�tte .NET 4.6
- Din organisasjon m� v�re registrert i adresseregisteret.
- Du m� ha brukernavn og passord til adresseregisteret og CPA tjenesten
- Du m� ha brukernavn og passord til k� systemet, samt connection string.
- Du m� vite hva din her-id er for noe
- Du m� ha private sertifikater for kryptering og signering. 
- Systemet ditt m� v�re p� helsenettet. 
- Det m� ha blitt opprettet et sett med k�er for deg. Dersom du skal sende synkrone meldinger, s� m� hver mottagene server ha sin egen k�.  

Adresseregisteret, CPA tjenesten og k� systemet driftes av Norsk Helse Nett. 

## Integrering med din kode

### Eksterne pakkeavhengigheter

Koden i denne pakken benytter 
- Microsoft.Extensions.Caching.Abstractions.IDistributedCache
- Microsoft.Extensions.Logging.Abstractions.ILogger

Disse tilbyr generelle grensesnitt for logging og caching. Pakkene er en del av den nye ASP.NET Core stacken, men fungerer fint med .NET 4.6. 
For faktisk implementasjon av disse grensesnittene s� kan man enten bruke noe som allerede er laget, eller benytte en egen implementasjon.

### Register integrasjon

F�r man kan sette opp infrastrukturen for meldinger, s� m� man ha register integrasjonen p� plass. 
Denne koden bruker klasser fra andre Microsoft.Extensions.* pakkker.

```cs
var loggerFactory = new LoggerFactory();
loggerFactory.AddConsole();
var logger = loggerFactory.CreateLogger<Program>();

var distributedCache = new MemoryDistributedCache(new MemoryCache(new MemoryCacheOptions()));

var addressRegistrySettings = new AddressRegistrySettings();
addressRegistrySettings.UserName = "user name";
addressRegistrySettings.Password = "password";
addressRegistrySettings.WcfConfiguration = ConfigurationManager.OpenExeConfiguration(ConfigurationUserLevel.None);
addressRegistrySettings.CachingInterval = TimeSpan.FromHours(12);			
			
var addressRegistry = new AddressRegistry(addressRegistrySettings, distributedCache);

var collaborationProtocolRegistrySettings = new CollaborationProtocolRegistrySettings();
collaborationProtocolRegistrySettings.MyHerId = "1234";
collaborationProtocolRegistrySettings.UserName = "user name";
collaborationProtocolRegistrySettings.Password = "password";
collaborationProtocolRegistrySettings.WcfConfiguration = ConfigurationManager.OpenExeConfiguration(ConfigurationUserLevel.None);
collaborationProtocolRegistrySettings.CachingInterval = TimeSpan.FromHours(12);			
			
var collaborationProtocolRegistry = new CollaborationProtocolRegistry(collaborationProtocolRegistrySettings, distributedCache);
```

### Sending av meldinger

MessagingClient klassen er designet for � v�re singleton. Dersom den ikke er det, s� vil det skape problemer med synkrone meldinger.

```cs
var messagingSettings = new MessagingSettings(); // denne har mange verdier satt som standard som man kan overstyre
messagingSettings.MyHerId = "1234";			
messagingSettings.DecryptionCertificate = "aaaaabbbbbbb";
messagingSettings.SigningCertificate = "ccccccddddd";
messagingSettings.ServiceBus.Synchronous.ReplyQueue = "MyReplyQueue";
messagingSettings.ServiceBus.ConnectionString = "connection string";

var client = new MessagingClient(messagingSettings, collaborationProtocolRegistry, addressRegistry);

var outgoingMessage = new OutgoingMessage()
{
	ToHerId = 789,
	CpaId = Guid.Empty,
	Payload = new XDocument(),
	MessageFunction = "DUMMY_MESSAGE_FUNCTION",
	MessageId = Guid.NewGuid().ToString("D"),
	ScheduledSendTimeUtc = DateTime.Now,
	PersonalId = "12345",
};

// for asynkrone meldinger
client.SendAndContinue(logger, outgoingMessage);

// for synkrone meldinger
client.SendAndWait(logger, outgoingMessage);
```

### Mottak av meldinger

```cs
var server = new MessagingServer(messagingSettings, logger, collaborationProtocolRegistry, addressRegistry);

server.RegisterAsynchronousMessageReceivedStartingCallback((message) => /* do something */ );
server.RegisterAsynchronousMessageReceivedCallback((message) => /* do something */ );
server.RegisterAsynchronousMessageReceivedCompletedCallback((message) => /* do something */ );

server.RegisterSynchronousMessageReceivedStartingCallback((message) => /* do something */ );
server.RegisterSynchronousMessageReceivedCallback((message) => /* do something */ { return new XDocument();} );
server.RegisterSynchronousMessageReceivedCompletedCallback((message) => /* do something */ );
```

### Feilh�ndtering

```cs
server.RegisterErrorMessageReceivedCallback((message) => /* do something */ );
server.RegisterUnhandledExceptionCallback((message, exception) => /* do something */ );
server.RegisterHandledExceptionCallback((message, exception) => /* do something */ );

 // avsender blir varslet om xml feil. errorCondition = transport:not-well-formed-xml
throw new XmlSchemaValidationException("XML-Error");

// avsender blir varslet om at dataene som kom ikke stemte. errorCondition = transport:invalid-field-value
throw new ReceivedDataMismatchException("Mismatch") { ExpectedValue = "Expected", ReceivedValue = "Received"};

// avsender blir varslet om annen feil. errorCondition = transport:internal-error 
throw new NotifySenderException("NotifySender");

// avsender blir ikke varslet, og meldingen blir liggende p� k�en og vi pr�ver p� nytt
throw new ArgumentOutOfRangeException();
```

### Logging
Det er verdt � nevne litt rundt logging. N�r man ser p� ASP.NET core koden, s� blir det opprettet en ILogger instans n�r en controller blir instansiert. 
ILogger instansen representerer hvor i koden man begynte, og f�r et navn basert p� dette. For v�r MessagingServer, s� har hver tr�d sin egen ILogger instans.

Disse instansene kan v�re kortlevd, og annen informasjon p� nettet indikerer at disse trenger ikke v�re thread safe. https://msdn.microsoft.com/magazine/mt694089

Siden MessagingClient og MessagingServer er singleton, s� er det ikke naturlig � bruke en logger instans for alle requester. 
Derfor er det veldig mange metoder som tar inn en ILogger referanse slik at alt som skjer knyttet til en request havner i samme kategori.

N�r man begynner � tenke p� correlation id'er s� er det noe ILogger systemet ikke har st�tte for. Man kan tenke seg at man bruker 
RegisterAsynchronousMessageReceivedStartingCallback til � sette en correlation id som tagger alle logginnslagene knyttet til en spesifik melding. 

NLog st�tter b�de ILogger og har st�tte for AsyncLocal verdier via Mapped Diagnostic Logical Context.

### Eksempler
Det finnes to konsoll applikasjoner som viser en klient og server komponent. 
Kliententen sender xml filer som ligger p� disk, og serveren skriver mottatte meldinger til disk. 

Helsenorge.Messaging.Client
Helsenorge.Messaging.Server

