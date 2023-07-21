# Istruzioni per lo script:

### Lo script richiede le seguenti informazioni:

- `picture_folder`: Percorso della cartella in cui si trovano le immagini
- `pixel_config`: Percorso del file di configurazione in formato TOML, che specifica quale immagine deve essere collocata
- `--config`: Crea una configurazione, può essere utilizzato più volte. (anche in caso di upgrade)

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

`""` segnane uno "parametri vuoti" (viene quindi ignorato, cucchiaio: `10;250;;/tmp/prio.png;/tmp/json.png;1;1;1;1`
oder `;;;/tmp/prio.png;;/tmp/json.png;1;1;1;1` non genererebbe un file png, ma il file prio)  
Esempio:  
`20;200;/tmp/png.png;/tmp/prio.png;;/tmp/json.json;0;0;0;0`  
`20;200;;/tmp/prio.png;;/tmp/json.json;1;0;0;0`  
`20;200;/tmp/png.png;;/tmp/picture_prio.png;/tmp/json.json;1;1;0;0`  
`20;200;/tmp/png.png;/tmp/prio.png;;/tmp/json.json;0;0;0;0`


------
File toml:

`ignore_colors`: qualsiasi colore da ignorare. I colori sono espressi in esadecimale ma SENZA il # iniziale.  
`width`: Larghezza dell'immagine generata 
`height`: Altezza dell'immagine generata
`add-x`: Offset x (reddit utilizza coordinate negative); sommandola alla larghezza, la coordinata più piccola deve essere 0!
`add-y`: Offset y (reddit utilizza coordinate negative); sommandola all' altezza, la coordinata più piccola deve essere 0!
`default_prio`: Priorità predefinita per tutte le immagini  
`structure' (Molteplici)

Ogni `structure` ha i seguenti valori:

|   Parameter   |       Esempio       | Facoltativo |                               Beschreibung                               |
|:-------------:|:-------------------:|:-----------:|:------------------------------------------------------------------------:|
|     name      |     flagge-ost      |             |                          Nome della struttura, deve essere univoco                            |
|     file      |   flagge-ost.png    |             |                            Nome file, relativo a `picture_folder`                             |
| priority_file | flagge-ost-prio.png |      X      |               Nome file per il file di priorità, relativo a `picture_folder`                  |
|    startx     |         100         |             |  x (da sinistra a destra) valore iniziale in corrispondenza del quale impostare la struttura  |
|    starty     |         100         |             |          y (dall'alto verso il basso) Valore iniziale in cui impostare la struttura           |
|   priority    |         127         |      X      |            Priorità per i pixel dell'immagine che non hanno una propria priorità              |

255 è la priorità più alta
Le priorità dei pixel sono calcolate come segue (ultima corrispondenza)
1. Priorità default
2. Priorità della struttura, se assegnata
3. Canale alfa del pixel se l'immagine ne ha uno
4. Valore del canale rosso del pixel corrispondente nel PNG prio, se esiste un PNG prio (gli altri canali vengono ignorati)


### Dependency

- Pillow~=9.4.0
- toml~=0.10.2
- python~=3.10

Installation:
`sudo apt install python3.11-full`
`python3.11 -m ensurepip --upgrade`
`python3.11 -m pip install --upgrade pip`
`python3.11 -m pip install toml`
`python3.11 -m pip install pillow`
