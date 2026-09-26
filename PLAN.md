# Enroute: de routeplanner die meebeweegt met wat er speelt

## Context

Karin wil een nieuwe fietsroute-app bouwen voor sportieve fietsers. Wat haar stoort aan het huidige aanbod:

- **Het is niet live.** Een route wordt vooraf gemaakt en blijft daarna dom. Een afgesloten weg zie je pas als je ervoor staat. Garmin, Wahoo en Hammerhead herberekenen alleen als je al van de route af bent.
- **Populaire routes zijn niet de beste routes.** Komoot en Strava sturen je over veelgereden wegen. Die lopen vaak dwars door dorps- en stadskernen: stoplichten, drempels, rotondes, winkelend publiek, scholieren.
- **Wind is grof.** Garmin kent alleen N/O/Z/W en geen windverwachting per uur.
- **Rondjes zijn lastig.** Google Maps plant alleen van A naar B.

Doel: een product voor de markt. Eerst een webapp voor Nederland, daarna een telefoon-app die live meedenkt, daarna op de fietscomputer, en op termijn eventueel eigen hardware.

## De belofte

> Jij zegt: "70 km, vanaf huis, zaterdag 9:00". Enroute bedenkt het beste rondje voor dát moment, en past het onderweg aan als er iets verandert.

Je hoeft geen filters aan te vinken. Je kiest hooguit een profiel (racefiets, gravel, toer).

## Wat maakt het anders dan Komoot

| | Komoot | Enroute |
|---|---|---|
| Hoe een route ontstaat | Je klikt punten aan, of volgt "highlights" en populaire wegen | Je geeft afstand en vertrektijd, de planner kiest zelf |
| Dorpskernen | Juist vaak erdoorheen, want daar rijden veel mensen | Actief vermijden, tenzij er geen alternatief is |
| Wind | Weerbericht langs de route, maar route past zich niet aan | Rondje wordt gebouwd rond de wind op het uur dat je daar fietst |
| Afsluitingen en evenementen | Niet | Actuele Nederlandse data, vooraf en tijdens de rit |
| Tijdens de rit | Statische route, herberekenen pas als je van de route af bent | Stelt onderweg een betere route voor en stuurt die naar je fietscomputer |
| Voor wie | Wandelaars, toerfietsers, mountainbikers, iedereen | Specifiek sportieve fietsers die door willen rijden |

Kort gezegd: Komoot helpt je een mooie route te vinden die anderen al reden. Enroute bedenkt de beste route voor nu en blijft meedenken.

## Wat de planner meeweegt

Het belangrijkste doel is **doorrijden**: zo weinig mogelijk remmen, stoppen en optrekken.

| Factor | Bron | Gewicht |
|---|---|---|
| Dorps- en stadskernen | OSM bebouwde kom, woongebieden, `maxspeed` 30, `place` | Hoog |
| Stoplichten | OSM `highway=traffic_signals` | Hoog |
| Drempels, rotondes, wegversmallingen | OSM `traffic_calming`, `junction=roundabout` | Middel |
| Wind in graden en kracht, per uur | Open-Meteo, KNMI HARMONIE | Hoog. Tegenwind vooral in de eerste helft. |
| Wegdek | OSM `surface`, `smoothness` | Hoog voor racefiets |
| Verkeersdrukte | OSM wegklasse, NDW-intensiteiten | Middel |
| Drukke fietspaden met bromfietsers | OSM `cycleway`, `moped` | Middel |
| Beweegbare bruggen, spoorwegovergangen, ponten | OSM `bridge:movable`, `railway=level_crossing`, `ferry` | Middel |
| Voorrang en wegen van rechts | OSM wegklasse, `give_way`, `stop` | Laag, als bijkomend voordeel |
| Hoogte | AHN, SRTM | Afhankelijk van wens |
| Populariteit bij sportieve fietsers | Eigen ritdata van gebruikers, alleen snelle ritten | Later, licht |

Populariteit telt bewust weinig. En als hij meetelt, dan alleen ritten van sportieve fietsers, niet van woon-werkverkeer.

## Live: wat "meebewegen" betekent

| Wat er speelt | Bron | Wat Enroute doet |
|---|---|---|
| Wegafsluitingen en werkzaamheden | NDW open data, Melvin | Vermijden bij planning. Onderweg omleiden als er iets nieuws opduikt. |
| Evenementen, markten, wielerwedstrijden | Melvin, gemeentelijke vergunningen | Idem |
| Draaiende windrichting | Open-Meteo, elk kwartier | Voorstel om het rondje om te draaien of in te korten |
| Buien | Buienradar, KNMI radar | Waarschuwen en een route voorstellen die de bui ontwijkt |
| Meldingen van andere fietsers | Eigen app | "Weg opgebroken", "gravel na regen modderig" |
| Openstaande brug, pont uit de vaart | NDW brugopeningen, vaartijden | Omleiden of wachttijd meerekenen |

Het werkt zo: de telefoon kent je positie en de rest van je route. Elke paar minuten kijkt de server of er iets verandert op je resterende route. Zo ja, dan komt er een voorstel. Jij accepteert met één tik, of het gebeurt automatisch.

## Op de fietscomputer

Er is een realistische route naar live op het stuur, zonder meteen eigen hardware te maken.

1. **Hammerhead Karoo.** Draait op Android en heeft een open uitbreidings-SDK. Enroute kan daar als volwaardige app op draaien, met live herplannen. Beste eerste kandidaat.
2. **Garmin Edge.** Connect IQ laat eigen apps toe. Bestaande apps zoals dwMap bewijzen dat eigen navigatie mogelijk is. Beperkter dan Karoo, maar Garmin is de grootste markt.
3. **Wahoo.** Geen apps op het apparaat. Alleen routes synchroniseren via de cloud. Live aanpassen gaat daar via de telefoon.
4. **Alle apparaten.** GPX-export en koppeling met Garmin Connect, Wahoo en Hammerhead voor vooraf geplande routes.

### Eigen hardware

Kan, maar pas als er bewezen vraag is. Wat het vraagt:

- Een Android-gebaseerde fietscomputer via een fabrikant die apparaten onder eigen merk levert. Zo is Karoo ook begonnen.
- Hardwareontwerp, CE-certificering, batterijveiligheid, voorraad, garantie, klantenservice.
- Grofweg een investering van enkele tonnen tot meer dan een miljoen euro en een eigen hardwareteam.

Advies: bouw de software zo dat hij op Android draait. Dan werkt dezelfde app op telefoon, Karoo en later eigen hardware. De hardwarebeslissing kan dan wachten tot er duizenden betalende gebruikers zijn.

## Techniek

- **Routering:** zelf gehoste GraphHopper met OSM-extract van Nederland. Ingebouwd rondje-algoritme en per verzoek aanpasbare kostenmodellen.
- **Planner-service:** TypeScript (Node). Maakt kandidaat-rondjes, haalt wind en afsluitingen op, scoort.
- **Live-service:** houdt actieve ritten bij, controleert de resterende route tegen nieuwe afsluitingen, wind en buien, en stuurt voorstellen.
- **Webapp:** Next.js met MapLibre-kaart.
- **Mobiele app:** Expo/React Native met achtergrond-GPS. Android-basis maakt de stap naar Karoo klein.
- **Data:** OSM, Open-Meteo, NDW, Melvin, Buienradar.

## Rondjes genereren

1. Startpunt, afstand of duur, vertrektijd en profiel.
2. Windverwachting ophalen voor gebied en ritduur.
3. 20 tot 50 kandidaten maken: GraphHopper `round_trip` in verschillende richtingen in graden, plus varianten via tussenpunten op een cirkel rond de start.
4. Elke kandidaat scoren, met wind op het verwachte tijdstip per stuk.
5. De beste 3 tonen die onderling verschillen, met uitleg: "3 km door bebouwde kom, 2 stoplichten, wind mee op de terugweg".

## Fasering

**Fase 0: bewijzen dat het beter is**
- GraphHopper lokaal met NL-data en een racefietsmodel dat kernen, stoplichten en slecht wegdek vermijdt.
- Script dat voor een paar startpunten rondjes maakt en vergelijkt met een Komoot-rondje van dezelfde lengte: km door kernen, stoplichten, stops, km tegenwind.

**Fase 1: webapp**
- Startpunt, afstand, vertrektijd kiezen. Windbewuste rondjes met uitleg. Afsluitingen uit NDW en Melvin meegenomen bij planning. GPX-export.

**Fase 2: telefoon-app met live herplannen**
- Navigatie in de app. Live controle op afsluitingen, wind en buien. Voorstel om te herplannen tijdens de rit.
- Accounts, routes bewaren, koppeling met Garmin Connect, Wahoo en Hammerhead voor vooraf geplande routes.

**Fase 3: live op de fietscomputer**
- Karoo-app met live herplannen.
- Garmin Connect IQ-app.

**Fase 4: slimmer en groter**
- Leren van gereden ritten van gebruikers.
- Clubs, groepsritten, trainingsdoelen.
- Beslissing over eigen hardware.

## Validatie en verdienmodel

- **Testgroep vanaf fase 0.** 10 tot 20 sportieve fietsers rijden voorgestelde rondjes naast hun gebruikelijke Komoot- of Garmin-route en geven een oordeel.
- **Meetbaar bewijs.** Per rit: minuten stilstand, km door kernen, stoplichten, km tegenwind. Dat wordt ook het marketingverhaal.
- **Freemium.** Gratis: rondjes plannen en GPX-export. Betaald, ongeveer 5 euro per maand: live herplannen, windtiming per uur, fietscomputer-apps.
- **Later B2B.** Clubs, evenementen, fietsvakanties.
- **Juridisch.** OSM vraagt bronvermelding. AVG-conforme opslag van ritdata vanaf fase 2. Strava-data mag volgens hun voorwaarden niet worden gebruikt om een concurrerende dienst te trainen, dus we leren alleen van eigen gebruikersdata.

## Risico's

- **Kernen vermijden kan omwegen opleveren.** Soms is een kern de enige oversteek over een kanaal of spoor. Het model straft kernen, maar verbiedt ze niet.
- **Afsluitingsdata is niet compleet.** NDW en Melvin dekken veel, maar niet alles. Meldingen van gebruikers vullen aan.
- **Garmin Connect IQ is beperkt.** Daarom Karoo eerst.
- **Komoot kan wind of live toevoegen.** Voorsprong zit in de combinatie van Nederlandse live-data, doorrijden als ontwerpprincipe en live op de fietscomputer.

## Eerste bouwstap in deze repository

Fase 0 als proof of concept:

- `routing/`: Docker Compose voor GraphHopper, NL-extract van Geofabrik, custom model voor racefiets.
- `planner/`: TypeScript-module die kandidaat-rondjes maakt, wind ophaalt bij Open-Meteo, afsluitingen bij NDW, en scoort.
- `web/`: eenvoudige kaartpagina die de 3 beste rondjes toont met uitleg.
- `README.md`: uitleg en opstartinstructies.

## Verificatie

- GraphHopper start en levert een rondje binnen 10% van de gevraagde afstand.
- Voor een paar testlocaties telt een script km door bebouwde kom en aantal stoplichten op de Enroute-route en op een standaard fietsroute van GraphHopper. Enroute moet duidelijk lager scoren.
- Bij harde westenwind begint het beste rondje aantoonbaar tegen de wind in.
- Een weg die in de NDW-data als afgesloten staat, komt niet in de voorgestelde routes voor.
- Webpagina getest met Playwright: kaart laadt, 3 routes verschijnen, GPX-download werkt.
