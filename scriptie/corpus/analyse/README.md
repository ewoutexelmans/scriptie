# Technische Analyse {#analyse}
 
De opdrachtomschrijving omvat een reeks functionaliteitsvereisten zonder een vooraf bestaande 'as-is'-situatie: dit wilt zeggen dat de applicatie volledig van de 'grond op' ontwikkeld zal worden, waarbij er veel waarde gehecht wordt aan de juiste keuzes betreffende technologie, ontwikkelingsomgeving, design, externe afhankelijkheden, test framework, en opleveringssystemen.
 
## Technologie {#tech}
 
Het project is voorgesteld als een full-stack .NET-gebaseerde applicatie. Aangezien dit een volledig nieuwe applicatie is, is het aangewezen om in de backend met .NET Core aan de slag te gaan. Deze keuze wordt voornamelijk door Microsoft zelf gemotiveerd: de afgelopen jaren heeft Microsoft ingezet op .NET Core als de sneller bewegende, 'bleeding-edge' technologie vergeleken tegenover het vrij oude .NET Framework, met een geleidelijke overdracht van de vele features in het .NET Framework naar .NET Core. In de aankondiging van .NET Core 3.0 liet Microsoft zelfs weten dat deze versie alleen nog maar op het .NET Core framework zal werken, waar in voorgaande versies dit nog op het standaard .NET Framework kon gecompileerd worden (Scott H, 2018).
 
Voor het admin-paneel zal er een  frontend framework gebruikt worden . Hier kiezen we voor Angular (7), waarbij de voornaamste van de motiverende factoren de design goals van Angular zelf zijn: het is gericht op een enterprise context, heeft een ontwerp gebaseerd op het Model-View-Controller (MVC) design dat .NET Core ook deelt, en het gebruikt TypeScript, wat in grote mate de syntax van C# reflecteert. Deze eigenschappen maken  Angular een populaire keuze bij veel programmeurs in het .NET landschap.
 
## Ontwikkelingsomgeving {#env}
 
De vereisten van de ontwikkelingsomgeving bevatten als minimum volgende zaken:
 
* ASP.NET Core 2.2 (Ten tijde van start project) ontwikkelingsomgeving
* npm package manager
* Angular CLI
* SQL Server Express
* Toegang tot een slack workspace en bijhorende slackbot
 
Ten slotte dienden er ook automatiserende (shell en SQL) scripts meegeleverd te worden voor het compileren van de projecten, de aanmaak en migraties van de database, en invoegen van omgevingsvariabelen. Dit vergemakkelijkt het potentiële proces van de opname van nieuwe ontwikkelaars in het ontwikkelingsproces of de overdracht van het project.
 
## Design {#design}
 
Het juiste design voor het project werd gekozen na evaluatie van de vereisten van ons project, waaronder het potentiële volume, de vorm en de kost van communicatie met de backend (queries, writes, enz.), eventueel hergebruik of verdere ontwikkeling alsook moderne software development architectuur- en designpatronen.
 
 
**Vorm, volume en kost van interacties**: de hoofdzakelijke manier van communicatie en vergaren van data input zal via een chatbot gebeuren. Dit stelt al meteen een potentieel grote volume en asynchrone manier van dataverzending voor. De applicatie moet per bedrijf en per werknemer interacties met de chatbot tegelijk kunnen verwerken die eerst nog via het externe systeem van Slack zelf passeren. Deze verzoeken moeten door onze backend verwerkt worden, en meteen moet er al (sorterings-)logica toegepast worden: mag dit verzoek verwerkt worden? Wat voor verzoek is het en naar welk deel van de business logica hoort het te gaan? Pas dan wordt mogelijk nog onze interne logica toegepast, en nog eventuele database operations. Daarenboven moet er ook steeds een bijbehorend antwoord gestuurd worden. De gebruiker van een bot hoort immers altijd te beschikken over kennis van de status van de chatbot (bv. meldingen dat bepaalde inputs niet herkend werden, succes-bevestigende antwoorden of verder aftakkende vragen van de chatbot). Bijgevolg zijn dit vaak vrij kostelijke operaties. We zitten bovendien ook met een meer traditionele CRUD-service voor ons admin-paneel, dat in feite dient om de data-input van de chatbot te overschrijven (bv. het aanpassen van verkeerde antwoorden), en zodus ook met meerdere command- en query bronnen.
 
 
**Project vereisten**: hier worden de concrete verwachtingen van het project op in brede lijnen opgesomd, gedetailleerde feature-vereisten zullen bij 'requirements' opgelijst worden.
 
* De applicatie moet over een database beschikken.
* De applicatie moet met de database integreren.
* De applicatie moet met interne business logica de databases wijzigen zoals nodig.
* De applicatie moet toegang voorzien tot deze business logica via API interfaces.
* De applicatie moet integreren met Slack API.
* De applicatie moet via de Slack integratie de chatbot op tijdschema's kunnen aansturen.
 
**Hergebruik en toekomstige ontwikkeling**: vanuit de beschrijving en besprekingen met Involved werd er afgesproken dat het project moet worden aangezien als iets wat mogelijk nog verder wordt ontwikkeld, of waarvan delen zullen hergebruikt worden.
 
**Software Architectuur**: in het verleden werden applicaties voornamelijk als zogenoemde "monolithische" applicaties gebouwd, 'single-tiered' software programma's waarin user interface, data toegang en business logica in één groot platform worden gecombineerd. In modernere software development trends wordt de nadruk eerder op modulariteit gelegd, waarbij gebruik gemaakt wordt van 'loosely-coupled', multi-tiered aparte componenten in een zogenoemde microservice-gebaseerde architectuur. De voordelen van deze microservices liggen in het feit dat individuele componenten (of services) veel meer herbruikbaar worden, ze aangemoedigd worden om klein te blijven, een klein aantal ontwikkelaars per service aanmoedigt, ze worden apart opgeleverd bij een deployment proces (en kunnen dus apart schaalbaar gemaakt worden), en dependencies kunnen opgesplitst worden per component ipv voor de hele applicatie. De nadelen zijn dat er over het algemeen meer complexiteit wordt geïntroduceerd, wat het noodzakelijk maakt om goede documentatie, goeie DevOps en een (ideaal zelf-documenterende) testing framework te voorzien, en het kan ook moeilijk worden om de afhankelijkheden die **wel** bestaan bij te houden, dus ook dit moet goed worden gedocumenteerd en onder toezicht gehouden.
 
**Software Design Patterns**: Verder bekijken we de de potentiële complexiteit en volume van de communicatie van onze applicatie en welke design patronen hier een oplossing kunnen vormen. In feite zit de applicatie meteen met een complexiteit aan queries die van meerdere services data moeten kunnen ophalen, en dat in een microservice architectuur. Hier kunnen we 2 design patronen voor vinden als oplossing. **Command Query Responsibility Segregation** (CQRS) en **API Composition**.
 
Bij CQRS worden lees-acties (*queries*) en schrijf-acties (*commands*) gescheiden met een query-model en een command-model, respectief. Het voordeel van dit design wordt duidelijk als men volgende situatie neemt: een administrator voert een query uit op een werknemer detail pagina, en wilt ook een eigenschap aanpassen (een command). Tegelijkertijd beantwoordt deze werknemer een vraag van de slackbot welke deze zelfde eigenschap moet updaten (ook een command). Bij een traditionele CRUD-controller zou dit al meteen voor problemen kunnen zorgen, want er worden 2 commands en een query voor dezelfde data verstuurd, welke door dezelfde interface moeten geïnterpreteerd worden. Bij een CQRS-implementatie worden alledrie zorgvuldig gescheiden. Het nadeel hierbij is dat voor deze 3 aparte stukken functionaliteit ook logica dient te bestaan, wat het risico voor code-duplicatie en onnodige complexiteit verhoogt. (Fowler, M. 2011)
 
Bij data-driven API Composition wordt er een soort van 'API gateway' geïmplementeerd in het backend systeem welke automatisch een reeks RESTful APIs bundelt en beschikbaar maakt op één portal aan externe consumenten. Dit is een vrij arbeidsintensief proces waar betrokken ontwikkelaars manueel de data modellen van alle kandidaat API's moet bekijken, de bijbehorende concepten matchen, een nieuw globaal model opstellen en deze publiceren. Eén opkomend praktische uitvoering hiervan is het bundelen van meerdere gepubliceerde Web APIs van bedrijven of regeringen om nieuwe informatie te genereren en publiceren. (Ed-douibi, H. 2018)
 
**Conclusie:** daar hier gewerkt wordt met een ‘agile’ manier van ontwikkeling, en design patronen vaak pas worden geïmplementeerd na het herhaaldelijk tegenkomen van een probleem (design patronen zijn ideaal om te implementeren bij ‘refactoring’, in tegenstelling tot de core architectuur van het project), kunnen niet vooraf alle design patronen worden bepaald. Echter kan wel de conclusie gevormd worden dat voor het grondwerk een **loosely-coupled microservice-architectuur** het voordeligst is, dit met de 'losse' stukken vereiste functionaliteit in acht genomen: zij kunnen immers afgesplitst worden van het project (bv. als een ander project slack integratie nodig heeft, kan onze implementatie gemakkelijk hergebruikt worden).Om loose coupling te helpen bereiken in een project, kunnen we het dependency injection patroon gebruiken. Voor de concrete implementatie wordt er voorlopig gekozen voor de dependency injection container van .NET Core zelf, om in het begin het aantal externe afhankelijkheden laag te houden, en omdat .NET Core out-of-the-box het DI patroon volledig ondersteunt. (Smith, S. Addie, S. Latham, L. 2018) Een voorbeeld van een bibliotheek die dit ook kan oplossen is **Autofac**, welke voornamelijk uitgebreidere levensduur opties toelaat voor de afhankelijkheden, maar deze wordt bij het moment van schrijven als niet nodig geacht daar wij onze afhankelijkheden alleen per verzoek hoeven te injecteren.
 
Verder zal er als oplossing om overzicht en structuur te behouden in de verschillende vormen van communicatie en data sources in de applicatie gebruik gemaakt worden van CQRS. Het vormt een meer rechtstreekse oplossing voor de voorgestelde problemen in dit project, en de meer abstracte manier van werken via de Slack-integratie kan beter geïmplementeerd worden in deze vorm dan wanneer er getracht wordt ze op te nemen in een samengestelde API portal volgens de werkwijze van API Composition. De gekozen technologie om dit design te helpen implementeren is MediatR. MediatR is op zich een implementatie van het mediator design patroon, een zogenoemde ‘event-driven pattern’ waar een extra ‘service layer’ interacties toelaat tussen objecten in de applicatie en dusdanig de nood om van elkaars bestaan af te weten vermindert. Dat brengt meteen het grote voordeel van deze applicatie met zich mee, namelijk dat het nog eens in grote mate de loose-coupling ervan bevordert. (Powell-Morse, A. 2017) De werkwijze van MediatR laat ons dan ook toe om een gemakkelijkere gebruik van CQRS te bereiken. Het design van MediatR centreert zich rond zogenoemde 'handlers', die in de business/domain layer van onze applicatie kunnen aangeroepen worden door berichten vanuit een presentatie layer (dit gebruik en aanroepen van handlers vormt dan samen de onzichtbare ‘service layer’ van MediatR). Dit is een ideale werkwijze om CQRS op een overzichtelijke manier te implementeren en commands en queries gescheiden te houden. (Gordon, S. 2016)
 
## Databank {#db}
 
**Technologie**: voor de database van het project is er besloten om te werken met Microsoft SQL Server. Motiverende factoren zijn de voorgaande ervaring van betrokken ontwikkelaars, de bestaande werkwijze van het bedrijf (integratie met Azure cloud services biedt ondersteuning aan voor een relatief klein aantal database technologieën) en de algemene bestaande associatie met het .NET development ecosysteem van Microsoft. De gekozen manier van data access is het toepassen van object-relational mapping met het bijhorende Entity Framework (Core), welke ons zal toelaten om alle data manipulatie op niveau van de applicatie zelf behouden zonder gebruik van breekbare SQL query strings in code.
 
**Design**: er is op basis van de initieel opgestelde requirements en user stories een eerste domain model mock opgesteld, hieronder afgebeeld, waaruit ontwikkeling zal beginnen. Echter is deze ver van finaal daar er een agile development werkwijze van kracht is, en er kan verwacht worden dat de benodigdheden en vorm van de domain models tijdens de duur van het project vaak zullen veranderen. Om ons te assisteren met deze eisen van een vaak veranderende en evoluerende database zullen we **FluentMigrator** gebruiken in de applicatie. FluentMigrator is een .NET bibliotheek dat ons toelaat om onze database schema op een constructieve en vlotte manier te initialiseren in de applicatie zelf met C# syntax en aan te passen waar nodig, in plaats van kostbare ontwikkelingstijd te verliezen met het (her)schrijven van meerdere SQL scripts die iedere keer moeten uitgevoerd worden per ontwikkelingsomgeving. De keuze om niet Code-First te werken en zodus de migraties van Entity Framework zelf te gebruiken is gedreven door het *Separation of Concerns* principe: aangezien we vanuit een domain-driven design perspectief werken, houden we de zaken domein en database gescheiden door ze apart te implementeren.
 
![Tweede versie Database Model Mock](../../../Documentatie/DatabaseModel&#32;Mockv2.png)

*Fig. 1: Tweede versie database model mock*
 
## Requirements {#req}
 
Hier wordt bij de analyse de reeks technische benodigdheden in de vorm van verwachte features om een Minimum Viable Product (MVP) te bekomen opgelijst, afgeleid van de rubriek 'functionaliteit' in de originele opdrachtomschrijving en verder verfijnd op basis van uiteindelijke epics in het agile development systeem:
 
* Het beheren van medewerkers via een client applicatie (toevoegen, opvragen, verwijderen, aanpassen).
* Het beheren van technologieën via een client applicatie (toevoegen, opvragen, verwijderen, aanpassen).
* Een slackbot applicatie dat via een slack workspace volgende functionaliteit kan toepassen:
** Vragen aan relevante werknemers waar hun huidige werkplaats is, en antwoorden hierop verwerken.
** Vragen aan relevante werknemers met welke technologieën hij of zij werkt, en antwoorden hierop verwerken.
** Op een slimme wijze antwoorden groeperen: bv. aNgulr wordt Angular.
** Ja/Nee vragen stellen aan medewerkers: bv. 'Heeft u al eens met technologie X gewerkt?' en antwoorden accepteren op basis van de selectiekeuze.
* Lijsten tonen van alle medewerkers en hun technologieën, en technologieën met hun relevante medewerkers.
* Deze data kunnen tonen in andere vormen, zoals een geïntegreerde heatmap.
 
Bijkomend worrden er nog requirements opgelijst voor de oplevering van de app:
 
* De applicatie moet i18n (internationalisatienorm) ondersteunen.
* Er moet een test, acceptatie en productieomgeving zijn.
* Naar deze omgevingen worden updates automatisch gepushed via een continuous integration pipeline, het stagebedrijf gebruikt Azure DevOps (bij start van dit project nog bekend als 'VSTS') en dus dient deze integratie hieronder te gebeuren.