# Project Management
## Project Management Systeem {#management_system}
Het projectmanagement systeem dat gebruikt wordt voor dit project is Azure Devops. Azure Devops, ook wel bekend onder de naam Visual Studio Online, is ontwikkeld door Microsoft.

Azure Devops biedt verschillende services aan die helpen bij het managen van het project.
De volgende services worden gebruikt voor het managen van dit project.

###### Azure Boards
Azure Boards is het eigenlijke projectmanagement systeem van Azure Devops. Gebruikers kunnen hier features, user story's en verschillende andere work items aan de backlog toevoegen.
Aan de user story's kunnen taken worden toegevoegd. Deze taken moeten afgewerkt worden om ervoor te zorgen dat user story als compleet te beschouwen.
Met Azure Boards kunnen sprints gepland en opgevolgd worden.

###### Azure Repos
Azure Repos is de git repository geïntegreerd in Azure Devops. Azure Repos biedt een manier aan om code reviews uit te voeren op pull requests.

###### Azure Pipelines
Met Azure pipelines kan de continue integratie en continue implementatie van het project worden geautomatiseerd.
Deze pipelines kunnen worden geconfigureerd zodat het builden en het releasen van het project automatisch gebeurd.

Azure Devops zorgt voor een nauwe samenwerking tussen deze drie services.
Aan user story's kunnen feature- of bugfix branches worden gehangen. Door de nummer van een work item te vermelden in een commit message, kunnen de commit's gelinkt worden aan specifieke taken die bij een user story horen.
Wanneer er een pull request wordt gecreëerd voor een branch, worden de gelinkte work items getoond.
Bij elke pull request kan commentaar gezet worden met een korte verwijzing naar de code. Wanneer er dan aanpassingen in dat gedeelte van de code komen, wordt er in de pull request een vergelijking getoond tussen de oude code en de geüpdatete code.
Wanneer de feature afgewerkt is en de pull request gemerged wordt, worden de work items en user story's, die gelinkt waren aan de branch, automatisch afgesloten.
De Azure Pipelines worden geconfigureerd zodat ze automatisch builden en releasen zodra er nieuwe code wordt gepushed op een branch.

## Agile Ontwikkeling {#agile_ontwikkeling}
Het project is ontwikkeld volgens de Agile Development. Wekelijks wordt er samengezeten met de stagementor voor de sprint retrospective en de sprint planning.

De sprint retrospective begint altijd met een demo van de applicatie zoals ze draait op de acceptatie omgeving. Alle functionaliteit die in de voorbije sprint is toegevoegd wordt getoond.
Er wordt besproken hoe de sprint verlopen is. Er wordt tijd gemaakt om te bespreken wat er niet goed is verlopen, maar ook wat er wel goed is verlopen. Ook wordt er aangehaald wat er verbeterd kan worden met zicht op de komende sprint.

Tijdens de sprintplanning wordt er bepaald welke user stories de komende week worden opgenomen. De user stories worden ingeschat met story points en vervolgens verdeeld naar gelang het aantal story points.

## Source Controle {#source_controle}
Dit is een gedetailleerde beschrijving van het source controle systeem toegepast in dit project.

Voor elke user story die gepland is in een sprint wordt er een branch gecreëerd met het formaat: feature/(nummer en naam van de user story).

Commits die gebeuren op de feature branches gebeuren moeten in hun message een nummer meekrijgen. Deze nummers komen overeen met de taken van een user story.

Voor elke branch wordt er een pull request gecreëerd. De stagaires en stagementor reviewen de code in de pull request. Wanneer de user story afgewerkt is, en de code is gecontroleerd en goedgekeurd op zijn kwaliteit, kan de pull request gemerged worden.

Voor het mergen worden alle commits op de branch gesquashed tot één commit.
Dit zorgt ervoor dat elke feature als één commit op de master branch terechtkomt

## Continue Integratie {#continue_integratie}
Voor de implementatie van de continue integratie wordt er gebruik gemaakt van de build pipeline. Het project heeft één build pipeline.

De build pipeline wordt geconfigureerd met behulp van yml files die mee in de source controle worden ingecheckt. De yml file die zich bevindt in de root folder van de repository vertelt de build pipeline wanneer hij getriggerd moet worden. De build pipeline creëert zo automatisch een nieuwe build, zodra er een commit wordt gepusht naar een van de branches die beschreven staat in de root yml file.
De root yml file bevat verwijzingen naar andere yml files die zich in de root folders bevinden van de verschillende frontend- en backend projecten.

De yml files die zich bevinden in de folders van de projecten, hebben drie taken:
1. Ze zorgen ervoor dat de build pipeline de juiste projecten build.
2. Ze zorgen ervoor dat de bestanden, die gegenereerd worden tijdens het buildproces, gebundeld worden. Deze bundel wordt gezipt in een zogenaamd artifact.
3. Ze zorgen ervoor dat de unit testen worden uitgevoerd als het builden geslaagd is.

Wanneer deze drie stappen geslaagd zijn, begint automatisch de de continue implementatie.

## Continue Implementatie {#continue_implementatie}
De release pipelines staan in voor de continue implementatie.

Het project heeft twee release pipelines. De eerste pipeline zorgt voor implementatie op de test omgeving. Deze pipeline gaat van start wanneer het builden en testen van een feature- of bugfix branch geslaagd is. De andere pipeline zorgt voor de implementatie op de acceptatie- en release omgeving. Deze pipeline gaat van start wanneer het builden en testen van de master branch geslaagd is.

De pipelines downloaden de artifacts die gecreëerd werden in de build pipeline.
Beide pipelines voeren vervolgens dire taken uit, om de release te doen slagen:
1. De variabelen die afhankelijk zijn van de specifieke omgeving worden gedeclareerd. Dit zijn bijvoorbeeld de connectionstrings voor de databanken, of de URL's van de API's.
2. De migraties worden uitgevoerd op de databanken.
3. De client-, api en scheduler applications worden gedeployed.

Aan een release pipeline kunnen variabelen worden vastgehangen. Omdat deze variabelen niet in de source controle terechtkomen, kunnen de release variabelen gebruikt worden om gevoelige informatie te bewaren.
In het geval van het Angular project zorgen we ervoor dat de environment variabelen, die applicatie gebruikt in de productie omgeving, vervangen worden met behulp van een powershell script.
In het geval van .NET Core, kan de release pipeline automatisch de variabelen die zich bevinden in de appsettings.json files vervangen. In dat geval moet de naam van de release variabele gelijk zijn aan het adres van de variabele in de appsettings.json file.

De eerste release pipeline zorgt ervoor dat de applicatie automatisch gedeployed wordt naar de test omgeving op Azure.
De tweede release pipeline zorgt ervoor dat de applicatie automatisch gedeployed wordt naar de acceptatie omgeving. De tweede release pipeline kan de applicatie ook deployen naar de productieserver. Dit moet echter manueel getriggerd worden.

Als het deployen naar de testomgeving geslaagd is, kunnen er client-side integratietesten op worden uitgevoerd. Op dit moment kan dit echter nog niet automatisch via de release pipeline.

## Lokale Ontwikkelingsomgeving {#lokale_ontwikkelingsomgeving}
Als er nieuwe ontwikkelaars aan het project beginnen werken, moet ervoor gezorgd worden dat zij op een vlotte manier kunnen start.
Daarom zijn er enkele scripts gecreëerd die helpen bij het opzetten van de lokale ontwikkelingsomgeving.

Iemand die de repository clonet, kan het bash script: build.sh uitvoeren. Dit script zorgt ervoor dat dat de API, scheduler en angular projecten worden gebuild met dotnet en npm. Er wordt een database gecreëerd op de lokale SQL-server en de console applicatie die de migraties van de database afhandelt wordt gebuild en gerund.
