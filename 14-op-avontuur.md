**Inhoud**

- [(terug naar het begin)](index.md)
- [8 - Waarmee wil je verder?](8-waarmee-verder.md)
- [9 - Spelers met karakter](9-spelers-met-karakter.md)

# 14 - Op avontuur!

In het [vorige hoofdstuk](13-grote-levels-editor.md) heb je gezien hoe je grotere levels kunt maken. Maar zo'n wijde wereld moet natuurlijk wel gevuld worden met interessante mogelijkheden. Daar gaan we nu mee aan de slag.

Je kunt <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=637f03e3c4899dec47f2d98b868a80db'>dit spel </a> met meerdere 'kamers' bijvoorbeeld als beginpunt gebruiken.

## Gesprekjes

Heel belangrijk in een avonturenspel is dat je andere wezens tegenkomt waar je mee kunt praten.

Misschien komen we wel een konijn tegen:

```
Konijn K
Gray Black
.0.0.
00000
01010
.000.
00000
```

Vergeet niet om het `Konijn` ook toe te voegen aan de `Object` groep onder `LEGEND`, en natuurlijk ergens in je wereld een konijn te plaatsen (met de letter `K` in de code, of via de editor).

Het konijn doet natuurlijk nog niets tot we er een regel voor aanmaken. Voeg een `RULE` toe zodat je een gesprekje kunt voeren door tegen het konijn aan te lopen:

    [ > Speler | Konijn ] -> message Ik ben Flappie, wie ben jij?

Als het goed is, stelt Flappie zich nu voor. Het is wel leuk als het konijn meer dan een ding zegt. Dat kunt doen door meerdere konijn-objecten te maken:

```
Konijn1 K
Gray Black
.0.0.
00000
01010
.000.
00000

Konijn2
Gray Black
.0.0.
00000
01010
.000.
00000

Konijn3
Gray Black
.0.0.
00000
01010
.000.
00000
```

Maak in het `LEGEND` gedeelte een groep `Konijn` aan waar al deze objecten in zitten:

    Konijn = Konijn1 or Konijn2 or Konijn3

Pas nu de regel die hierboven hadden gemaakt als volgt aan:

    [ > Speler | Konijn1 ] -> [ Speler | Konijn2 ] message Ik ben Flappie, wie ben jij?

Als je het spel draait, zal het konijn de eerste keer dat je tegen 'm aan loopt iets zeggen, maar daarna niet meer. Dat komt doordat de regel hierboven er een `Konijn2` van heeft gemaakt. Je moet dus nog regel(s) toevoegen die zorgt dat `Konijn2` ook iets terugzegt. Lukt dat?

Oplossing:

    [ > Speler | Konijn2 ] -> [ Speler | Konijn3 ] message Ik heb zo'n trek!
    [ > Speler | Konijn3 ] -> [ Speler | Konijn1 ] message Heb je misschien een wortel voor me?

Zoals je ziet maken we bij de tweede regel van `Konijn2` een `Konijn3` en bij de derde regel maken we er weer een `Konijn1` van. Hierdoor kun je het gesprekje nog eens herhalen als je vergeten was wat het konijn ook alweer precies zei.

Hier zie je het <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=4c7c6ed93d844f701f8eecc462d5c189'>resultaat</a>.

## Voorwerpen

Natuurlijk is het leuk als je voorwerpen kunt vinden die je verder kunt helpen.

### Duwen

De simpelste manier om in PuzzleScript iets te doen met voorwerpen is door het zo te maken dat je het voorwerp kunt duwen, net als de blokjes in vorige hoofdstukken.

Laten we een wortel en een spruitje voor het konijn toevoegen. We maken ook meteen een extra versie van het konijn, `GevoerdKonijn`, zodat we kunnen 'onthouden' wanneer het konijn iets gegeten heeft.

```
Wortel W
Orange Green
...11
..001
..00.
.00..
0....

Spruitje P
Darkgreen Green
.....
.010.
.111.
.100.
.....

GevoerdKonijn
Gray Black
.0.0.
00000
01010
.000.
00000
```

Maak bij `LEGEND` een groep `Groente` en voeg die toe aan `Objects`:

```
Groente = Wortel or Spruitje
Objects = Muur or Speler or Konijn or Groente
```

Plaats nu een wortel en spruitje in je wereld (met de letters `W` en `P`). Zorg dat je ze naar het konijn kan duwen.

Met deze regels kunnen we de wortel en het spruitje naar het konijn duwen:

```
(Groente duwen)
[ > Speler | Groente ] -> [ > Speler | > Groente ]
```

Misschien is het leuk als het konijn de wortel wel wil maar het spruitje niet:

```
(Konijn houdt niet van spruitjes)
[ > Spruitje | Konijn ] -> [ Spruitje | Konijn ] message Ik hou niet van spruitjes!

(Konijn eet wortel op)
[ > Wortel | Konijn ] -> [    | GevoerdKonijn ]
```

Hoe zou je het zo kunnen maken dat het konijn je bedankt als je nog eens met hem praat nadat je 'm gevoerd hebt?

```
(Konijn bedankt de speler voor de wortel)
[ > Speler | GevoerdKonijn ] -> message Dankjewel, dat was lekker!
```

En kun je zorgen dat het konijn genoeg heeft aan 1 wortel en "nee dank je" zegt als je er nog een aanbiedt?

```
(Konijn hoeft maar 1 wortel)
[ > Wortel | GevoerdKonijn ] -> message Ik heb genoeg gehad!
```

Hier is een <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=4c7c6ed93d844f701f8eecc462d5c189'>werkend voorbeeld</a>.

### Niet duwen maar dragen

Wat als we voorwerpen niet in het rond willen duwen, maar willen dat de speler het voorwerp oppakt en met zich mee draagt? Dat is wat lastiger, omdat we in PuzzleScript niet zomaar een lijst voorwerpen kunnen bijhouden die de speler bij zich draagt.

We kunnen wel een extra 'laag' (`COLLISIONLAYER`) gebruiken in PuzzleScript waar 'draagbare' voorwerpen zoals de wortel en het spruitje op staan:

```
================
COLLISIONLAYERS
================

(De 'grond')
Background

(Alle gewone spelobjecten: speler, konijn, muren)
Object

(Door de speler te dragen objecten, in dit geval de groente)
Groente
```

Doordat de groente nu op een aparte laag staat, is het mogelijk dat de speler en een van de groentes een vakje delen.

We moeten nu wel onze regels aanpassen zodat we voorwerpen dragen in plaats van duwen:

```
(Zodra de speler op het vakje van een draagbaar voorwerp staat, moet dat voorwerp met de speler meebewegen)
[ > Speler Groente ] -> [ > Speler > Groente ]
```

Als je wilt dat de speler een gedragen voorwerp ook weer neer kan leggen door op `x` te drukken, voeg dan deze regel toe:

```
(Zet draagbaar voorwerp neer op een lege plek naast speler)
[ ACTION Speler Groente | no Object no Groente ] -> [ Speler | Groente ]
```

`ACTION` betekent hier dat er net op `x` is gedrukt. PuzzleScript zal in dat geval zoeken naar een aangrenzend leeg vakje (geen `Object`, geen `Groente`) en zal daar de groente neerzetten.

We moeten ook onze regels voor het voeren van het konijn iets aanpassen, omdat de speler en het spruitje tegelijkertijd richting het konijn bewegen. Als je dit niet doet, komen de regels voor het voeren in de war met de regels voor het praten met het konijn:

```
(Konijn houdt niet van spruitjes)
[ > Speler > Spruitje | Konijn ] -> [ Speler Spruitje | Konijn ] message Ik hou niet van spruitjes!

(Konijn eet wortel op)
[ > Speler > Wortel | Konijn no GevoerdKonijn ] -> [ Speler | GevoerdKonijn ]

(Konijn hoeft maar 1 wortel)
[ > Speler > Wortel | GevoerdKonijn ] -> [ Speler Wortel | GevoerdKonijn ] message Ik heb genoeg gehad!

(Gevoerd konijn bedankt speler)
[ > Speler | GevoerdKonijn ] -> [ Speler | GevoerdKonijn ] message Dat was lekker, dankjewel!
```

Als het niet lukt, kijk dan hier voor een <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=b6c1928a337a957321bcfd6a36d3abc4'>werkend voorbeeld</a>.

In een echt spel wil je natuurlijk dat het `Konijn` je iets geeft als dank voor de wortel, zodat je verder kan komen in het spel. Bijvoorbeeld een sleutel. Kun je bedenken hoe je dat doet?

Maak daarvoor een `OBJECT` `Sleutel` aan, voeg 'm toe aan `Objects` (onder `LEGEND`) en pas de regel voor het geven van de wortel aan:

    [ > Speler > Wortel | Konijn no GevoerdKonijn ] -> [ Speler Sleutel | GevoerdKonijn ]

## Geheime deuren, knoppen

Als we een deur willen maken die opengaat als je een knop indrukt, bijvoorbeeld door er een kistje op te zetten, heb je eerst een aantal `OBJECTS` nodig:

```
Kistje @
(Een kistje dat de speler kan duwen)
white gray darkgray
00001
01112
01112
01112
12222

Deur D
(De deur die je kunt openen met de knop)
brown darkbrown
.101.
01010
01010
01010
01010

DeurOpen O
(De deur als-ie geopend is)
brown
.....
0....
0....
0....
0....

Knop K
(De knop waarmee je de deur kunt openen)
blue
.....
.000.
.000.
.000.
.....
```

Je kunt de deur natuurlijk ook een geheime deur maken! Pas daarvoor het plaatje aan zodat het er (bijna) hetzelfde uitziet als een muur.

Je kunt zelf kiezen waar je deze zaken in je wereld plaatst. Maar we moeten wel de `LEGEND` en `COLLISIONLAYERS` aanpassen. Bij `LEGEND`:

```
Vloer = Knop or DeurOpen

(Dit zijn al onze voorwerpen)
Voorwerp = Muur or Speler or Kistje or Muntje or Deur
```

Bij `COLLISIONLAYERS`:

```
================
COLLISIONLAYERS
================

Achtergrond

(We hebben een extra laag voor de knop en de open deur, omdat de speler of een kistje tegelijk op hetzelfde vakje moeten kunnen staan)
Vloer

Voorwerp
```

Je hebt natuurlijk een regel nodig zodat de Speler een kistje kan duwen:

```
(De Speler kan Kistjes duwen)
[ > Speler | Kistje ] -> [ > Speler | > Kistje ]
```

Om de deur open te laten gaan als er iets op de knop staat:

```
(Open deur als er een voorwerp op de knop staat, sluit 'm anders)
late [ Knop Voorwerp ] [ Deur ] -> [ Knop Voorwerp ] [ DeurOpen ]
late [ Knop no Voorwerp ] [ DeurOpen ] -> [ Knop ] [ Deur ]
```

Dit werkt ook als je meerdere knoppen en deuren hebt. Als alle knoppen ingedrukt worden, gaan de deuren open. Weet je hoe het komt dat het zo werkt?

Antwoord: de eerste regel opent alle deuren als tenminste 1 knop ingedrukt wordt, maar de tweede regel gooit alle deuren weer dicht als tenminste 1 knop niet ingedrukt wordt. Dus tot alle knoppen ingedrukt zijn, blijven alle deuren dicht.

Hier is een <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=d745b4dd69fc65d639036b265c50a90e'>werkend voorbeeld</a>.

Natuurlijk kun je ook een valstrik maken met een (bijna) onzichtbare knop waar de speler juist niet op moet gaan staan, omdat er dan bijvoorbeeld ergens een deur dichtgaat en de speler opgesloten zit.

## Magische drankjes

Magie is natuurlijk altijd leuk in een avonturenspel. Bijvoorbeeld een drankje waardoor je elastisch wordt en je je jezelf uit een gevangenis kan bevrijden door je door tralies heen te wringen!

We hebben nu twee spelerobjecten nodig: een `GewoneSpeler` en een `MagischeSpeler`:

```
GewoneSpeler S
(Het figuurtje dat je bestuurt)
darkgray brown lightblue blue
.000.
.111.
22222
.333.
.3.3.

MagischeSpeler
(De speler, nadat die het drankje gedronken heeft)
darkgray black
.000.
.000.
00000
.000.
.0.0.

Drankje D
(Een magisch drankje)
purple
..0..
..0..
.000.
.000.
.000.

Tralies T
white
0.0.0
0.0.0
0.0.0
0.0.0
0.0.0
```

Maak in `LEGEND` de groep `Speler` aan:

```
Speler = GewoneSpeler or MagischeSpeler
Player = Speler
```

(`Player = Speler` staat er alleen zodat PuzzleScript, dat overal Engels gebruikt, weet wat onze speler is)

We zetten de `Tralies` op een eigen `COLLISIONLAYER`, omdat de `MagischeSpeler` er straks doorheen moet kunnen lopen:

```
================
COLLISIONLAYERS
================
Achtergrond
Voorwerp
Tralies
```

Als je het drankje drinkt, willen we natuurlijk een magisch geluid laten horen:

```
=======
SOUNDS
=======
Drankje DESTROY 80302508
```

Deze regels heb je nodig:

```
(Als Speler het Drankje drinkt wordt die magisch)
[ > Speler | Drankje ] -> [ > MagischeSpeler |          ]

(Gewone speler kan zich niet door tralies heen wringen)
[ > GewoneSpeler | Tralies ] -> CANCEL

(Als je door de tralies heen gaat, verlies je je magie weer)
[ > MagischeSpeler Tralies |    ] -> [ Tralies | GewoneSpeler ]
```

Hier is het <a target='_blank' href='https://www.puzzlescript.net/editor.html?hack=1921649f0225820ff3dd08a1da3111a1'>werkende voorbeeld</a>.
