# Ontwikkeling
## Authenticatie en Autorisatie {#auth}
Authenticatie en autorisatie in onze applicatie wordt afgehandeld door auth0. Auth0 is een management platform voor authenticatie en autorisatie. Via het online platform wordt de autorisatie van de applicatie en de gebruikers geconfigureerd. Vanuit de applicatie zelf kan de configuratie aangepast worden met behulp van de management API van auth0.

Gebruikers worden geauthenticeerd met de universele login van auth0. Wanneer gebruikers een authenticatie request triggeren worden ze doorverwezen naar de universele login pagina van auth0. Deze pagina wordt enkel gebruikt voor inloggen en authenticatie, en toont het lock widget van auth0. Andere logica is niet beschikbaar op deze pagina.
De hoofdreden waarom er gebruik gemaakt wordt van universele login, is veiligheid. De universele login zorgt ervoor dat de applicatie geen gebruik maakt van cross-origin authenticatie. Cross-origin authenticatie is in se gevaarlijker en heeft een grotere kans om het slachtoffer te zijn van, onder andere, een man-in-the-middle aanval (Auth0 Documentatie, s.d.).

Wanneer de authenticatie geslaagd is, wordt het access token en de datum en tijd wanneer het token vervalt, ontvangen van auth0, opgeslagen in de browser cache via de localStorage tool van Angular. Deze tokens vervallen na tien uur.

De routes in de client side van de applicatie worden beschermd door de authGuard service. De service kijkt of er zich een accestoken bevindt in de local storage, en of dit token nog niet vervallen is. Zo ja, wordt de gebruiker weer doorverwezen naar de login pagina. Aangezien er maar één soort gebruiker toegang heeft tot de client side van de applicatie, namelijk de beheerder, heeft een geauthenticeerde gebruiker toegang tot alle pagina’s van de website.

In de backend van de applicatie worden alle controllers, buiten de Slack controller, in de API beschermd door autorisatie. Er kunnen dus geen ongeautoriseerde calls gemaakt worden vanuit de client. Calls die worden gemaakt vanuit de frontend worden in de client onderschept door een HttpInterceptor. Deze service zorgt ervoor dat het accestoken als bearer token wordt meegegeven aan de header van de call, alvorens ze naar de backend worden gestuurd. De backend checkt dan met behulp van de Authorization API van auth0 of de gebruiker deze call mag maken. In de huidige iteratie van het project, heeft een geauthenticeerde gebruiker toegang tot alle methodes van elke controller, buiten de Slack controller.

## Slack Integratie {#slack}
Om data te verkrijgen van medewerkers van het bedrijf hebben is er een Slack applicatie gecreëerd als onderdeel van het project.

Slack laat zijn gebruikers toe om applicatie toe te voegen aan de workspace. Zo kan men de functionaliteit van een workspace uitbreiden en veranderen.
In de Slack applicatie is er een bot user gebundeld. Deze bot zorgt ervoor dat medewerkers via conversaties kunnen interageren met de Slack app.
De Slack maakt gebruik van twee Slack API’s om interacties met medewerkers af te handelen, de Events API en de Conversations API.
Events API
De Events API zorgt ervoor dat een Slack app zich kan abonneren op events die gebeuren in Slack. In de Slack app geven we een request URL mee.
Om te verifiëren of deze URL geldig is stuurt Slack POST request met als type: url_verification en een willekeurige challenge string. Om de verificatie te voltooien moet de web API met een OK-response antwoorden, met een response body die de challenge string bevat.
Wanneer er een event gebeurt waarop we geabonneerd zijn, stuurt Slack een POST request naar de URL. De requests zijn JSON requests. De request URL komt overeen met een methode in de Slack controller van de web API. Slack verwacht dat er binnen de drie seconden gereageerd wordt met een 2XX response code. Lukt dit niet, zal de Events API maximaal drie keer proberen om de request naar de request-URL te sturen. Het is dus belangrijk dat de web API zo snel mogelijk antwoord op een request. Hierom moet er zoveel mogelijk vermeden worden dat het verwerken van een event en het beantwoorden op een event in hetzelfde proces gebeuren.
De Slack app kan zich op twee verschillende soorten events abonneren., Workspace Events en Bot Events.
Workspace Events zijn alle soorten events die kunnen plaatsnemen in een Slack Workspace. Deze events zijn echter niet relevant voor deze applicatie.
Bot Events zijn alle events die gerelateerd zijn aan kanalen en conversaties waar de bot deel van uitmaakt. Het enige event waarop de Slack app geabonneerd is, is het Bot Event: message.im. Dit event ontstaat wanneer er een bericht wordt gestuurd in een direct message kanaal, waar de bot deel van uitmaakt.
De web API checkt het type van de request die gestuurd is naar de request URL, om te weten of de request al dan niet verwerkt moet worden. De API is enkel in staat om requests met als type: url_verification (zoals hierboven geschreven wordt dit gebruikt om de request URL te verifiëren), en requests die een event hebben met als channel type: im (dit toont aan dat het event afkomstig is van een direct message channel), te verwerken. Als de controller een request ontvangt dat niet verwerkt kan worden, zal de methode een exception throwen.
Conversations API
## Beheerdersprofiel {#admin}
Choose language
Bij het injecteren van de translate service in de Appcomponent geven we mee voor welke talen we een json file hebben aangemaakt. Als standaardtaal gebruikt de client Engels.
De client checkt of de taalcode in de browser overeenkomt met een van de talen die beschikbaar zijn (bijvoorbeeld: nl voor Nederlands). Zo ja wordt die taal gebruikt voor de lokalisatie, zo nee wordt Engels gebruikt.
De translate service heeft een methode om de lijst van beschikbare talen te verkrijgen. Deze lijst wordt in de header weergegeven, zodat een gebruiker op die manier de taal kan aanpassen.
## Medewerkers {#medewerker}
CSV Upload
De gebruiker van de client applicatie moet op een gemakkelijke manier alle medewerkers van zijn bedrijf kunnen toevoegen. Daarom is er een optie voorzien in het beheerscherm van de medewerkers om een CSV-bestand te uploaden.
Dit bestand wordt met een request in een POST request doorgestuurd naar de web API. In de backend wordt het bestand met behulp van de .NET library CsvHelper omgezet naar een lijst van medewerkers. CsvHelper heeft een methode om data in een CSV-bestand lijn per lijn uit te lezen en te mappen naar een DTO, onder voorwaarde dat de titel van elke kolom gelijk is aan de property’s van de DTO. Is dit niet het geval, wordt er een foutmelding gegenereerd dat de kolommen in het bestand niet voldoen aan de verwachte property’s.
De lijst van DTO’s gegenereerd door de service wordt vervolgens gevalideerd. Ieder item in de lijst wordt gecontroleerd of het voldoet aan de voorwaarden die nodig zijn om een medewerker aan te maken (bijvoorbeeld de naam mag geen lege string zijn). Als er zich ergens in het bestand foute data bevindt, zal de gebruiker een foutmelding krijgen met het adres van de foute data (rijnummer en kolomnaam).
Slechts als de lijst gevalideerd is zal er een request worden gemaakt om een lijst van medewerkers te creëren in de databank.

Filter
De nieuwe filter moet ervoor zorgen dat er gefilterd kan worden op 
De filter is een domme component, die bestaat uit een enkel tekstveld. Wanneer de tekst in het tekstveld verandert, wordt het input event afgevuurd. De component bevat een subject, een speciaal type van observable. De waarde van het subject wordt de waarde van het tekstveld, telkens wanneer het input event wordt afgevuurd.
De filtercomponent heeft als output een observable die de waarde van het subject publiceert, zolang deze waarde een halve seconde onveranderd is gebleven én de waarde verschillend is van de vorige gepubliceerde waarde. Dit zorgt ervoor dat, als een gebruiker in het tekstveld aan het typen is, de waarde niet verandert met elke toetsaanslag. Zo wordt er enkel een request voor gefilterde data gestuurd wanneer de gebruiker even niet meer getypt heeft.
Een bovenliggende slimme component luistert naar het outputevent van de filtercomponent. De waarde die gebruiker intypte in het tekstveld is het keyword dat als parameter wordt gestuurd naar het filter subject in de dataservice. Aangezien een subject een soort observable is, kunnen we een functie creëren die een call doet naar de backend wanneer de waarde van het subject geüpdatet wordt.


## Technologieën {#technologie}
Technologies module
De client heeft een pagina voor het beheer van de technologieën en vaardigheden die de medewerkers kunnen bezitten.
De pagina heeft een overzicht met alle technologieën die de beheerder relevant vindt voor het bedrijf. Vanuit deze pagina kan de beheerder nieuwe technologieën toevoegen, oude technologieën updaten en irrelevante technologieën verwijderen. Per technologie kan de beheerder bekijken welke werknemers er vaardig zijn in de technologie.

## Vragen en Antwoorden {#qna}
Open question qna rounds
De Slackbot is in staat om open vragen te stellen via een direct message aan alle medewerkers. Dankzij de Events API kunnen we ervoor zorgen dat Slack de antwoorden doorstuurt naar de backend van de applicatie. Dit veroorzaakt een echter een probleem: Slack zal een request sturen naar de backend, elke keer iemand een direct message stuurt naar de bot.
Er moet dus voor gezorgd worden de backend enkel de direct messages van de medewerkers verwerkt wanneer er effectief een vraag is gesteld door de bot. Ook moet ervoor gezorgd worden dat de bot reageert op een direct message van een medewerker wanneer er geen vraag gesteld is of wanneer de medewerker al een antwoord heeft gegeven. Het is beter voor de gebruikservaring dat de bot elke input van een medewerker erkent. Zo kan er geen verwarring ontstaan over het feit of de bot al dan niet beschikbaar is, en waartoe de bot in staat is.
Als oplossing voor dit probleem is er een ronde systeem geïntroduceerd in de applicatie.
QnaRound is een entiteit die informatie bevat over het onderwerp van de vraag die moet gesteld worden tijdens de ronde, de vraag die effectief gesteld is tijdens de ronde en of de ronde al dan niet nog actief is.
De scheduler is verantwoordelijk voor het starten van een nieuwe ronde. Wanneer het tijd is om de medewerkers te bevragen over een nieuw onderwerp, stuurt de scheduler een request naar de business logica om een nieuwe ronde te creëren. Een ronde moet een onderwerp meekrijgen. Dit onderwerp kan bijvoorbeeld de werkplaats of een vaardigheid zijn. Aan de hand van het onderwerp van de ronde wordt dan bepaald welke vraag er overeenkomt met dat onderwerp.
Bij het creëren van een nieuwe ronde, wordt de nieuwe ronde als actief beschouwd en wordt de vorige ronde op non-actief gezet. Zo zal er altijd slechts één ronde actief zijn.
Zodra het creëren van een nieuwe ronde geslaagd is, stuurt de scheduler een request om de ronde-vraag te stellen.
De API heeft nu een manier om te bepalen hoe een direct message event moet worden afgehandeld. 
De flow ziet eruit als volgt.
De Slack Events API stuurt een request naar de Slack controller van de web API. De web API controleert het type van de request (de twee verwachte types zijn url_verification en event_callback). Als het gaat om een direct message request, wordt er een service aangesproken die de direct message kan afhandelen. De web API stuurt twee requests naar de business logica. Een om informatie te verschaffen over de actieve ronde, en een tweede om te bepalen of de medewerker die de request getriggerd heeft, al dan niet geantwoord heeft op de vraag van de huidige ronde. Als de medewerker al heeft geantwoord heeft, of er geen ronde actief is, wordt er via de bot een bericht gestuurd naar de medewerker dat er op dit moment geen vragen zijn die beantwoord kunnen worden. Wanneer er wel een ronde actief is, én de medewerker nog geen antwoord heeft gegeven op de vraag van die ronde, wordt de request afgehandeld naargelang het onderwerp van de ronde.

 
## Heatmap {#heatmap}
Heatmap
De heatmap is een visuele weergave van het aantal vaardigheden die de medewerkers van een bedrijf hebben.
De beheerder moet in een oogopslag kunnen zien met welke vaardigheden de medewerkers overweg kunnen. De heatmap wordt daarom weergegeven als een bubble chart. Elke cirkel komt overeen met een specifieke vaardigheid. Hoe groter de cirkel, hoe meer medewerkers aangeven dat ze ervaring hebben met die vaardigheid. De cirkels krijgen als titel de naam van de vaardigheid mee, en als ondertitel het aantal medewerkers die bekwaam zijn in die vaardigheid.
Om ervoor te zorgen dat de cirkels op een intuïtieve manier te vergelijken zijn, is er wiskunde nodig. Wanneer de ene vaardigheid dubbel zoveel medewerkers bevat als de andere, moet de oppervlakte van de ene cirkel dubbel zo groot zijn als de andere. Als de straal of de diameter van de ene cirkel dubbel zo groot zou zijn als de andere, merkt een menselijk oog gauw op dat de kleine cirkel niet juist twee keer in de grote past. Als er een één op één relatie zou zijn tussen de straal en het aantal vaardigheden, zou, bij een verdubbeling in vaardigheden, de cirkel vier keer zo groot lijken.
De relatie tussen de vaardigheden en de medewerkers is een veel op veel relatie. In de backend kan er met een eenvoudige query het aantal medewerkers per vaardigheid worden opgehaald. De client ontvangt de naam en het aantal in een DTO.
De opbouw van de heatmap gebeurt met behulp van de d3.js library. D3.js is een javascript library die documenten manipuleert op basis van data. We creëren een service, waarin d3 de heatmap gaat opbouwen. De service bouwt met de ontvangen data SVG-elementen op; voor elke vaardigheid een. Het DOM-element waar de heatmap moet in getoond worden krijgt als id: “charts” mee. Wanneer de we de methode van de heatmap service oproepen in een component, vult d3 het DOM-element met de juiste id op met de gecreëerde SVG-cirkels. De cirkels zijn gesorteerd op willekeurige volgorde en krijgen een willekeurige kleur.

