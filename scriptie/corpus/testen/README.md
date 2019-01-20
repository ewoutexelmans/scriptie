# Testen
## Integratietesten {#integratietesten}
Integratie testen zijn testen die worden uitgevoerd om de samenwerking tussen verschillende modules te testen. Verschillende modules van de software worden samengenomen en in hun geheel getest. In het test proces worden deze testen uitgevoerd na de unit tests. (Ould, M. & Unwin, C. 1986)

### Integratietesten client {#integratietesten_client}
Client-side worden er aan integratie tests gedaan om te verzekeren dat de functionaliteit van het softwaresysteem voldoet aan de vereisten. Zo wordt er,bijvoorbeeld, onder andere getest of werknemers die worden toegevoegd effectief verschijnen op de pagina.

Client-side testen besparen tijd aangezien de functionaliteit van het systeem niet meer handmatig moeten worden getest op de test server
Het is uiteindelijk de bedoeling dat client-side integratie testen worden in de release planning uitgevoerd na het deployen van de software op de test server. Op dit moment worden moeten de integratie testen nog manueel worden gestart.


Het integratietest project draait op Node.js en is volledig in typescript geschreven.
Client side integratie tests worden uitgevoerd met behulp van twee library’s, jest en puppeteer.

Jest is een open-source library ontwikkeld door werknemers van Facebook. Het framework wordt gebruikt om javascript te testen en ondersteund ook typescript.

Puppeteer is een library die controle geeft aan google chrome en chromium via het Devtools Protocol. Puppeteer kan zowat alle handeling uitvoeren die iemand handmatig in een browser kan doen. Het is ontwikkeld door ontwikkelaars van google chrome. Met puppeteer kunnen google chrome en chromium “headless” worden gestart. Dit betekent dat er een browser wordt geopend zonder grafische gebruikersinterface. (Puppeteer Documentatie, s.d.)

De testomgeving wordt als volgt opgezet. 
In jest zijn er drie configuratieopties waar gebruik van wordt gemaakt in ons project. GlobalSetup, globalTeardown en testEnvironment zorgen ervoor dat jest gebruik maakt van custom modules, zodat er vlot gebruik gemaakt kan worden van puppeteer voor het effectieve testen.

De setup module exporteert een asynchrone functie die eenmalig wordt uitgevoerd voor alle testen. Hierin wordt er een een globale instantie van de headless browser gecreëerd met door puppeteer, er wordt ingelogd in de client side van de applicatie. Puppeteer creëert bij de opstart van de globale browser een websocket endpoint waar de browser met verbonden wordt. De verbindingsdetails van het websocket endpoint worden tijdelijk naar een lokaal bestand geschreven.

De environment module exporteert een klasse. Deze klasse creëert een omgeving waarin de testen worden uitgevoerd. Puppeteer zorgt ervoor dat een nieuwe browserinstantie wordt aangemaakt voor de testomgeving, die verbinding maakt met het websocket endpoint, opgezet in globalSetup. De testen draaien dus in de browserinstantie aangemaakt in testEnvironment.

De teardown module exporteert dan weer een asynchrone functie die eenmalig wordt uitgevoerd nadat alle testen zijn afgelopen, ofschoon deze geslaagd zijn of niet. De teardown module sluit de globale browser die werd aangemaakt in de setup module. Hierdoor sluit ook het websocket endpoint.

Tussen enkele stappen van het testen door, worden er screenshots genomen van de browser. Deze screenshots worden geüpload naar een virtuele opslagruimte in de azure cloudomgeving. Op die manier kunnen de verschillende stappen worden opgevolgd en gecontroleerd wanneer de testen falen.
