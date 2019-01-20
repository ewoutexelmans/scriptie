# Testen

Bij het evolueren van de applicatie is er vrij vlug een nood ontstaan voor een robuust test framework. Logica welke een tastbaar resultaat geeft zoals validatie en berekeningen kunnen vrij eenvoudig in **unit tests** automatisch getest worden. Er moet altijd zekerheid bestaan dat deze cruciale stukken functionaliteit het verwachte resultaat kunnen opleveren. Anderzijds wordt het bij het groeien van de applicatie ook altijd arbeidsintensiever om manueel features af te gaan en te bekijken of alles nog naar verwachting werkt. Aangezien dit in principe bij het ontwikkelen van elke nieuwe feature moet gedaan worden, wordt dit vlug een tijdrovende bezigheid. Tot het automatiseren van dit proces kunnen we **integratie tests** gebruiken. Er moet ook gezorgd worden dat dit op een constructieve manier wordt toegevoegd aan het project, wat wilt zeggen dat de tests per project opgedeeld worden in een nieuw, bijhorend test project.

## FluentAssertions {#fluent}

In het algemeen is er zoveel mogelijk gebruik gemaakt van zelf-beschrijvende tests. Vanuit een design perspectief is er zoveel mogelijk uit het principe gewerkt dat tests ook als documentatie moeten kunnen dienen. Neem bijvoorbeeld het valideren van een email input. Bij het ingeven van een lege email moet dit een 'Invalid' respons teruggeven. Dan wordt er bij de benaming van klasse en methode geschreven:

```cs
[TestClass]
public class WhenValidatingEmail
{

    [TestMethod]
    public void GivenEmptyEmailItShouldBeInvalid
    {
        ...
    }
}
```

Om verder te gaan met dit principe van documenterende tests te schrijven, is er gebruik gemaakt van **FluentAssertions**. Dit is een bibliotheek dat een alternatieve werkwijze aanbiedt tegenover het UnitTesting framework van Microsoft zelf, en is ook ontwikkeld geweest met het doel van zo leesbaar mogelijk te zijn. Daarbovenop is het beschikbaar met een groot aantal extensies om complexere tests te ontwikkelen.

De superieure leesbaarheid van FluentAssertions syntax kan vlug aangetoond worden:

```cs
string email = "valid@email.com";
var ok = EmailValidator.Validate(email);
Assert.AreEqual(true, ok);
```

wordt:

```cs
string email = "valid@email.com";
var ok = EmailValidator.Validate(email);
ok.Should().BeTrue();
```

Bij het eerste voorbeeld moet men toch een moment nemen om te begrijpen wat de beschreven verwachting juist is, terwijl bij het FluentAssertions voorbeeld dit in menselijkere taal vlugger overkomt.

## Unit Tests {#unit}
  
### API {#unitapi}

De eerste reeks unit tests zijn geschreven voor de reeks formulier validaties welke gebruikt worden aan het API einde. Dit zijn eenvoudige unit tests die met FluentAssertions asserteren of de juiste boolean wordt teruggegeven bij de input van een bepaalde string. Bijvoorbeeld, bij het valideren van een email moet bij het toesturen van een simpele string met geen formattering False geasserteerd worden.

### Domein {#domein}

In het domein kunnen de entiteitsrelaties getest worden, aangezien dit tijdens runtime in principe op volledig programmatisch niveau gebeurd. Er is doorheen de ontwikkeling van de applicatie zoveel mogelijk van deze interacties op niveau van domein gehouden om deze testbaarheid te verhogen (wanneer een entiteit in een databank query wordt gecontroleerd of veranderd, wordt het veel complexer om een test uit te voeren op deze functionaliteit door de nood voor een databank mock). Dus in plaats van te controleren op de IsActive flag van employees bij de database query, is er een extra verzameling 'EmployedEmployees' toegevoegd onder Company om deze check volledig op domein niveau te houden en bijgevolg testbaarder te maken.

Als voorbeeld van een dergelijke domein test, neem een bedrijf, en een nieuw aangemaakte werknemer. Dit 'employee' object wordt onder company.AddEmployee(newEmployee) toegevoegd. Dan kunnen we met FluentAssertions op hetzelfde Company object `company.Employees.Contains(employee).Should().BeTrue();` aanroepen.

### Automatisatie

Het uitvoeren van de unit tests is geïntegreerd in het deployment proces, en wordt bij de eerste stap in de continuous integration (build) bij elke repository push automatisch uitgevoerd. Zodoende kan men bij elke building log een extra tabblad 'Tests' bekijken waarin de resultaten van de unit tests altijd zullen gepubliceerd worden.

Het instellen van dit automatisch testen is bereikt door de implementatie van een extra stap `dotnet test ...` in de beschrijving van de YAML-file van het project waarvoor de tests zijn aangemaakt. Het publiceren van de resultaten gebeurt in een bijkomend geschreven Task 'PublishTestResults' welke de logs in een folder publiceren welke Azure DevOps dan kan gebruiken om ze op te halen.

## Integratietesten {#integratietesten}
Integratie testen zijn testen die worden uitgevoerd om de samenwerking tussen verschillende modules te testen. Verschillende modules van de software worden samengenomen en in hun geheel getest. In het test proces worden deze testen uitgevoerd na de unit tests. 

### API  {#integratieapi}
 
Er zijn in de backend voor het API ook integratie tests voorzien, welke functionaliteit op de API controllers vanuit een breder perspectief van een volwaardige API consumer kunnen controleren. Om dit te bereiken diende een soort van mock gemaakt te worden voor een API client. Hier is als eerste stap NSwag gebruikt, een tool vergelijkbaar met ng-swagger-gen dat ook clients voor gebruik in .NET Core kan generen uit het swagger.json bestand. Hier is ook een script voor geschreven om elke keer bij het aanpassen van een API controller gemakkelijker een nieuwe client te genereren. Echter kan deze gegenereerde client niet meteen gebruikt worden, aangezien er een geldig Auth0 token nodig is om met de API te mogen communiceren. Er is tot dit doeleinde middleware geschreven, 'AuthorizedClientFactory', welke bij elke integratie test een voor-geautoriseerde client kan produceren. Dit stuk middleware post eerst een verzoek naar het Auth0 domein en vraagt zo een token aan, en voegt deze token toe in de headers van een nieuw httpClient, welke vervolgens als basis dient voor een nieuwe instantie van de klasse die nswag heeft gegenereerd. Dit maakt op een zeer praktische manier op verzoek voorgeautoriseerde clients beschikbaar die met het API mogen communiceren.

Met deze client kan dan bv. een employee aangemaakt worden op de API met een POST, en met een GET meteen gecontroleerd worden of deze employee nu bestaat. FluentAssertsions laat ook toe om acties op te zetten, en de uitvoering hiervan op verwachte errors of exceptions te controleren. Bijvoorbeeld:

```cs
public class WhenDeactivatingEmployee
{
    ...
    
    public void ShouldReturnNotFound()
    {
    var brainChainClient = AuthorizedClientFactory.CreateClient();
    Func<Task> action = async () => await brainChainClient.DeactivateEmployeeAsync(NONEXISTENT_EMPLOYEE_ID);
    action.Should().Throw<SwaggerException>().And.StatusCode.Should().Be(404);
    }
}
```

### Integratietesten client {#integratietesten_client}
Client side worden er aan integratie tests gedaan om te verzekeren dat de functionaliteit van het softwaresysteem voldoet aan de vereisten

Client side testen besparen tijd aangezien de functionaliteit van het systeem niet meer handmatig moeten worden getest op de test server
Client side integratie testen worden in de release planning uitgevoerd na het deployen van de software op de test server.
Er wordt onder andere getest of werknemers die worden toegevoegd effectief verschijnen op de pagina.
Het integratietest project draait op Node.js en is volledig in typescript geschreven.
Client side integratie tests worden uitgevoerd met behulp van twee library’s, jest en puppeteer. Jest is een open-source library ontwikkeld door werknemers van Facebook. Het framework wordt gebruikt om javascript te testen en ondersteund ook typescript.
Puppeteer is een library die controle geeft aan google chrome en chromium via de Devtools Protocol. Puppeteer kan zowat alle handeling uitvoeren die iemand handmatig in een browser kan doen. Het is ontwikkeld door ontwikkelaars van google chrome. Met puppeteer kunnen google chrome en chromium “headless” worden gestart. Dit betekent dat er een browser wordt geopend zonder grafische gebruikersinterface.
De testomgeving wordt als volgt opgezet. 
In jest zijn er drie configuratieopties waar gebruik van wordt gemaakt in ons project. GlobalSetup, globalTeardown en testEnvironment zorgen ervoor dat jest gebruik maakt van custom modules, zodat er vlot gebruik gemaakt kan worden van puppeteer voor het effectieve testen.
 De setup module exporteert een asynchrone functie die eenmalig wordt uitgevoerd voor alle testen. Hierin wordt er een een globale instantie van de headless browser gecreëerd met door puppeteer, er wordt ingelogd in de client side van de applicatie. Puppeteer creëert bij de opstart van de globale browser een websocket endpoint waar de browser met verbonden wordt. De verbindingsdetails van het websocket endpoint worden tijdelijk naar een lokaal bestand geschreven.
De environment module exporteert een klasse. Deze klasse creëert een omgeving waarin de testen worden uitgevoerd. Puppeteer zorgt ervoor dat een nieuwe browserinstantie wordt aangemaakt voor de testomgeving, die verbinding maakt met het websocket endpoint, opgezet in globalSetup. De testen draaien dus in de browserinstantie aangemaakt in testEnvironment.
De teardown module exporteert dan weer een asynchrone functie die eenmalig wordt uitgevoerd nadat alle testen zijn afgelopen, ofschoon deze geslaagd zijn of niet. De teardown module sluit de globale browser die werd aangemaakt in de setup module. Hierdoor sluit ook het websocket endpoint.
Tussen enkele stappen van het testen door, worden er screenshots genomen van de browser. Deze screenshots worden geüpload naar een virtuele opslagruimte in de azure cloudomgeving. Op die manier kunnen de verschillende stappen worden opgevolgd en gecontroleerd wanneer de testen falen.
