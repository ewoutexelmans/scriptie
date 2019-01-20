# Project Management
## Agile Ontwikkeling {#agile_ontwikkeling}
## Source Controle {#source_controle}
## Projectstructuur {#projectstructuur}
### Client-side
#### Structuur
De client-side van het project is opgebouwd in Angular. Sinds oktober 2018 maakt het project gebruik van Angular versie 7.

De Angular CLI wordt gebruikt om de client en zijn componenten, services, ... te genereren. Dit zorgt ervoor dat de structuur van de client zich houdt aan de best practices, aangeraden door de officiële documentatie van Angular (Angular Docs, s.d.).
De folder waarin het client-side project zich bevindt, is hernoemd naar *Involved.BrainChain.Client* om consequent te zijn met de naamgevingsregels van de backend projecten.

Om ervoor te zorgen dat de implementatie van features, zowel op korte termijn als op lange termijn, helder en overzichtelijk kan gebeuren, passen we in de client applicatie het volgende principe toe; Feature- , shared- en coremodules (Angular Docs, s.d.).

**Featuremodules** zijn modules gecreëerd voor een specifieke toepassing van de applicatie. 
Het is de bedoeling dat bestanden die gerelateerd zijn aan een zekere functionaliteit, in één folder terechtkomen. De totale inhoud van de folder wordt dan gehangen aan de featuremodule. Dit betekent concreet, dat de featuremodules afgestemd zijn op de businesslogica van onze applicatie. Bijvoorbeeld, alle client-side logica die een beheerder nodig heeft om medewerkers van het bedrijf te beheren, wordt gebundeld in de medewerkers featuremodule.
Elke featuremodule heeft een routingmodule. Op deze manier heeft elke featuremodule een relatie met een specifieke link in de client applicatie. Featuremodules zijn lazy-loaded. Wanneer de gebruiker surft naar de link van een featuremodule Dit zorgt ervoor dat de modules enkel geladen worden wanneer ze beschikbaar moeten zijn voor de gebruiker.
In dit project bevatten de featuremodules voornamelijk slimme- en domme componenten, en services.

De **sharedmodule** bevat delen van de applicatie die in meerdere featuremodules gebruikt worden.
De sharedmodule bevat voornamelijk directives, pipes, componenten,.... Componenten die gebruikt worden in meer dan twee featuremodules, worden als vuistregel gedeclareerd in de sharedmodule. Deze componenten zijn altijd domme componenten.
Services horen niet thuis in de sharedmodule. Dit voorkomt dat er meerdere instanties van een singleton service worden gecreëerd.
De sharedmodule wordt geïmporteerd in de featuremodules die toegang nodig hebben tot de shared componenten, directives, enzovoorts.

De **coremodule** bevat alle services en componenten die de client moet laden bij het opstarten van de applicatie.
In het algemeen zijn het meestal services die gedeclareerd worden in de coremodule.
De coremodule wordt geïmporteerd in de root AppModule, en enkel daar.

Alle services in de client zijn singelton services. De coremodule zorgt ervoor dat de services geprovided worden in de root van de applicatie. Dit maakt dat de client meer *tree shakable*[^1] is.
Services die gebruikt worden in slechts één featuremodule, worden mee in de folder van die featuremodule geplaatst. Ook deze services worden geprovided in de root.
De services die gegeneerd worden met behulp van Swagger, zijn voorbeelden van services die in meerdere featuremodules gebruikt kunnen worden.[^2] Deze services horen dus in de folder van de coremodule.

De componenten die aanwezig zijn in de featuremodules, zijn opgedeeld in domme en slimme componenten. De relatie tussen de slimme en domme componenten is meestal een parent-child relatie.
Slimme componenten staan in voor de werking van de client en het verwerken van data. Domme componenten tonen de data aan de gebruiker, en zorgt ervoor dat de gebruiker kan interageren met de data (Arnold, 2017). Bijvoorbeeld, data die verwerkt werd in de slimme component en getoond moet worden aan de gebruiker, wordt doorgespeeld naar een domme component. Het is dan de taak van die domme component om de data te tonen. Als een gebruiker interactie moet kunnen hebben met een domme component, wordt dit niet afgehandeld door de domme component. De domme component stuurt de interactie van de gebruiker naar zijn ouder, de slimme component. De slimme component zal dan de interactie van de gebruiker afhandelen.
Als gevolg zullen services nooit geïnjecteerd worden in domme componenten, enkel in de slimme ouder.
In de featuremodules wordt het aantal slimme componenten zo klein mogelijk gehouden. Concreet zullen de verschillende routes van de client altijd verwijzen naar de slimme componenten. De DOM-elementen die getoond moeten worden zijn dan opgebouwd met de HTML-selectors van de domme componenten.

####  Programmeerconcepten
##### Reactive programming
Reactive programmeren is een ontwikkelingsmodel gestructureerd rond asynchrone datastreams. Reactive Extensions, ofte Rx, is de API die het meeste gebruikt wordt om reactive te programmeren. In de client wordt er gebruik gemaakt van de RxJS, de Rx library voor javascript.
Rx is opgebouwd rond het gedachtegoed van het Observable Pattern, het Iterator Pattern en Functional Programming.
In de client wordt er voornamelijk gebruik gemaakt van Observables en Observers. Observers kunnen zich abonneren (subscriben) op een Observable. Wanneer Observables datastreams de wereld insturen, zullen de geabonneerde Observers luisteren naar en reageren op de datastreams. De kracht van Rx zit in het feit dat de API de client toegang geeft tot verschillende Operators. Deze operators zorgen ervoor dat we datastreams kunnen transformeren, combineren, manipuleren, enzovoort… (Bhuvan, 2018).
Rx wordt in het project gebruikt om op een vlotte en asynchrone manier dataflow tussen de verschillende componenten, services, directives, enzovoort… te voorzien. De belangrijkste plek waar Rx gebruikt wordt in het project is bij de communicatie tussen de client en de backend.
De web API maakt gebruik van Swashbuckle om een swagger file te generen. In de client wordt er gebruik gemaakt van de swagger-gen library om de services aan te maken die de effectieve REST calls naar de web API gaan uitvoeren. Alle methodes in de gegenereerde retourneren Observables. De data die de client toon moet altijd de meest recente zijn die in de databank aanwezig is. Daarom steekt er een service als buffer tussen de slimme component die de data nodig heeft, en de service die de calls naar de web API maakt.
De werking van de buffer service wordt aan de hand van de medewerkers module uitgelegd. Op het beheerscherm voor de medewerkers kan de beheerder een lijst van medewerkers raadplegen en de details van elke medewerker te zien krijgen. Op deze lijst van medewerkers kunnen er CRUD-operaties worden uitgevoerd.
De buffer service heeft een Observable property genaamd employees. Employees heeft als type een lijst van de medewerkers. Employees wordt gecreëerd door combineLatest, een van de methodes van Rx. CombineLatest verwacht als parameters meerdere Observables, in dit geval refreshSubject en filterSubject. Subjects zijn een type van Observables. Zodra een van de twee Subjects een nieuwe waarde uitzendt, zal employees door combineLatest zijn nieuwe waarde uitzenden. De Rx operator, mergeMap, wordt toegepast op combineLatest om ervoor te zorgen dat de waarde die employees zal uitzenden de lijst van medewerkers is. MergeMap spreekt de methode uit de gegenereerde service aan, die de GET-request naar de web API doet voor de lijst van medewerkers. MergeMap zorgt er ook voor dat employees gelijk is aan de Observable die geretourneerd wordt door de get methode van de gegenereerde service. Wanneer de client een CRUD-operatie wil uitvoeren op de lijst van medewerkers, wordt ervoor gezorgd dat refresSubject een waarde uitzendt (deze waarde mag altijd null zijn). Zo bevat employees altijd de meest recente lijst.


## Continue Integratie {#continue_integratie}
Azure pipelines yml files
Continue integratie en continue deployment
Azure pipelines is een tool dat gebruikt wordt om te helpen bij continue integratie en continue deployment.
Voor continue integratie maken we gebruik van de build pipeline. De build pipeline detecteert wanneer er veranderingen zijn aangebracht op de branches van onze repository. De configuratie van de build pipeline wordt beschreven in yml files.
De yml file die zich bevindt in de root folder van de repository vertelt de build pipeline wanneer de build pipeline getriggerd moet worden. Het project heeft dus één build pipeline die automatisch wordt uitgevoerd zodra er een commit wordt gepusht naar een van de branches, die beschreven staat in de yml file, in de repository. Ook bevat deze een verwijzing naar andere yml files die dan weer beschrijven welke projecten moeten worden gebuild en getest.
De bestanden, die gegenereerd worden door het buildproces, worden gekopieerd naar een zogenaamd artifact. Deze artifacts worden gebruikt door de release pipeline.
In de build pipeline worden ook de unittests automatisch uitgevoerd.
Wanneer zowel het builden als de test geslaagd zijn, gaat de release pipeline van start.
De release pipeline staat in voor de continue deployment van de applicatie. Het project heeft twee release pipelines. Artifacts die gegenereerd worden door alle branches, buiten de master branch, triggeren de eerste pipeline. Artifacts die gegeneerd worde door de master branch triggeren de tweede pipeline.
Aan een release pipeline kunnen variabelen worden vastgehangen. Omdat deze variabelen niet in de source controle terechtkomen, kunnen de release variabelen gebruikt worden om gevoelige informatie te bewaren.
In het geval van het Angular project zorgen we ervoor dat de environment variabelen, die applicatie gebruikt in de productie omgeving, vervangen worden door de juiste release variabelen. Dit gebeurt met behulp van een powershell script.
In het geval van .NET Core, kan de release pipeline automatisch de variabelen die zich bevinden in de appsettings.json files vervangen. In dat geval moet de naam van de release variabele voldoen aan de correcte benaming.
De eerste release pipeline zorgt ervoor dat de applicatie automatisch gedeployed wordt naar de test servers.
De tweede release pipeline zorgt ervoor dat de applicatie automatisch gedeployed wordt naar de acceptatie server. De tweede release pipeline kan de applicatie ook deployen naar de productieserver. Dit moet echter manueel getriggerd worden.
Beide release pipelines zorgen ervoor dat de migraties die nodig zijn, worden uitgevoerd op de respectievelijke databanken.
Nadat de applicatie gedeployed is op de testservers, worden er clientside integratietesten uitgevoerd
## Continue Implementatie {#continue_implementatie}
## Persistence {#persistence}
Fluent migrations
De migraties van de databank worden uitgevoerd met behulp van Fluent Migrator. Fluent Migrator is een framework ontwikkeld voor .NET en .NET Core. Het staat toe om, op een gestructureerde en heldere manier, database schema’s aan te passen en te creëren. De migraties worden beschreven in klasses, geschreven in C#.
Er zijn twee redenen waarom de voorkeur uit ging naar Fluent Migrator boven de Code First Migrations van Entity Framework zelf. 
In het project is er een .NET Core console applicatie aanwezig dat ervoor zorg dat de migraties worden uitgevoerd. De console applicatie aanvaard een connectionstring van een databank als argument.
De console applicatie wordt ook gebruikt om de lokale databank en de databank op de testserver, te seeden met testdata. Fluent Migrator laat het toe om via de runner SQL-query’s uit te voeren op de databank.
In de build pipeline wordt de console applicatie gebuild. De DLL-bestanden die het resultaat zijn van de build worden gekopieerd naar het build artifact. In de release pipeline worden de connectionstrings van de databanken opgeslagen. De migraties worden in de release planning als eerste stap uitgevoerd. Via een command line script in de wordt de console applicatie uitgevoerd, met de juist connectionstring verworven uit de release pipeline.

## Lokale Ontwikkelingsomgeving {#lokale_ontwikkelingsomgeving}
Local scripts
Nadat wij gestopt zijn met werken aan de applicatie, wil Involved dat er nog verder ontwikkeld kan worden aan de applicatie. Daarom zijn er, op vraag van de opdrachtgever, enkele scripts gecreëerd die helpen bij het opzetten van de lokale ontwikkelingsomgeving.
Iemand die de repository clonet, kan het bash script: build.sh uitvoeren. Dit script zorgt ervoor dat dat de API, scheduler en angular projecten worden gebuild met dotnet en npm. Er wordt een database gecreëerd op de lokale SQL-server en de console applicatie die de migraties van de database afhandelt wordt gebuild en gerund.

[^1]: *Tree shaking* is een term die, in de context van JavaScript, gebruikt wordt om het elimineren van dode code te beschrijven. Ongebruikte modules worden tijdens het buildproces niet gebundeld door *tree shaking* (Bachuk, 2017).
[^2]: De services die gegeneerd worden door Swagger zorgen ervoor dat de client calls kan doen naar de backend.