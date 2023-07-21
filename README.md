# Istruzioni per lo script pixel:

### Lo script richiede le seguenti informazioni:

- `picture_folder`: Percorso della cartella in cui si trovano le immagini
- `pixel_config`: Percorso del file di configurazione in formato TOML, che specifica quale immagine deve essere collocata
- `--config`: Può essere utilizzato più volte. Una configurazione valida anche in caso di upgrade

La configurazione del generatore ha il seguente formato:  
`--config min_prio;max_prio;png_path;prio_path;png_prio_path;json_path;io;ip;ao;cmp`

Spiegazionne dei percorsi:  
`png_path`: L'immagine generata
`prio_path`: La maschera di priorità generata in scala di grigi (l nero è la massima priorità). è solo considerato,solose 'ip' è '0'. 
`png_prio_path`: L'immagine generata con colore e priorità in un PNG (La priorità è il canale alfa). è solo considerato,solose 'ip' è '0'.  
`json_path`: Il file json generato, accettato dallo script overlay o dallo script placer.

I singoli parametri:

| Parameter |                Valori consentiti             |   Default   |                                                                                                            Descrizione                                                                                                              |
|:---------:|:-------------------------------------------:|:------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| min_prio  |                0 <= x <= 255                |      10      |                                                                                      Tutti i pixel con una priorità inferiore vengono ignorati.                                                                                     |
| max_prio  |                0 <= x <= 255                |     250      |                                                                                      Tutti i pixel con una priorità più alta vengono ignorati.                                                                                      |
|  *_path   | percorso file o<br/> Testo libero <br/> `""` |      -       | ,Il risultato corrispondente viene emesso come base64 in stdout se `base64:` è all'inizio. Quindi il testo libero con due punti viene scritto davanti all'output<br/>Altrimenti l'output viene memorizzato nel percorso appropriato.|
|    io     |                `0` oppure `1`               |      -       |                                                                                      Genera l'output come sovrapposizione (c'è un pixel vuoto tra ogni due pixel                                                                    |
|    ip     |                `0` oppure `1`               |      -       |                                                                                      Ignora completamente le priorità; tutti i pixel hanno priorità 255                                                                              |
|    ao     |                `0` oppure `1`               |      -       |                                                                                      Consenti o non consentire la sovrascrittura dei pixel con pixel con priorità più alta                                                          |
|    cmp    |                `0` oppure `1`               |      -       |                                                                                      Tutti i pixel che hanno una priorità più alta di quella specificata da `max_prio' sono impostati su questo valore                              |

`""` kennzeichnen einen "leeren Parameter" (wird dann ignoriert, Bsp: `10;250;;/tmp/prio.png;/tmp/json.png;1;1;1;1`
oder `;;;/tmp/prio.png;;/tmp/json.png;1;1;1;1` würde keine png Datei generieren, aber die Prio Datei)  
Beispiele:  
`20;200;/tmp/png.png;/tmp/prio.png;;/tmp/json.json;0;0;0;0`  
`20;200;;/tmp/prio.png;;/tmp/json.json;1;0;0;0`  
`20;200;/tmp/png.png;;/tmp/picture_prio.png;/tmp/json.json;1;1;0;0`  
`20;200;/tmp/png.png;/tmp/prio.png;;/tmp/json.json;0;0;0;0`


------
toml Datei:

`ignore_colors`: alle Farben, die ignoriert werden sollen. Die Farben werden in Hex aber OHNE führendes # angegeben.  
`width`: Breite des generierten Bildes  
`height`: Höhe des generierten Bildes  
`add-x`: Offset x (reddit nutzt negative Koordinaten); nach Addition muss kleinste Koordinate 0 sein!
`add-y`: Offset y (reddit nutzt negative Koordinaten); nach Addition muss kleinste Koordinate 0 sein!
`default_prio`: Default Priorität für alle Bilder  
`structure` (Liste)

Jedes `structure` hat folgende Werte:

|   Parameter   |      Beispiel       | Optional |                               Beschreibung                               |
|:-------------:|:-------------------:|:--------:|:------------------------------------------------------------------------:|
|     name      |     flagge-ost      |    N     |                  Name der Struktur, muss eindeutig sein                  |
|     file      |   flagge-ost.png    |    N     |                  Dateiname, relativ zu `picture_folder`                  |
| priority_file | flagge-ost-prio.png |    J     |         Dateiname für die Priodatei, relativ zu `picture_folder`         |
|    startx     |         100         |    N     | x (links-nach-rechts) Startwert, an den die Struktur gesetzt werden soll |
|    starty     |         100         |    N     |  y (oben-nach-unten) Startwert, an den die Struktur gesetzt werden soll  |
|   priority    |         127         |    J     |     Priorität für Pixel des Bildes, die keine eigene Priorität haben     |

255 ist die höchste Priorität.
Die Prioritäten der Pixel werden wie folgt berechnet (last match):
1. Default prio
2. Prio der Struktur, falls gegeben
3. Alpha Channel des Pixels, falls das Bild einen solchen hat
4. Wert des roten Kanals des entsprechenden Pixels im Prio PNG, falls es ein Prio PNG gibt (die anderen Kanäle werden ignoriert)
