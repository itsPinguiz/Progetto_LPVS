# Sistema RFID con Risoluzione Collisioni non Deterministica

Il presente codice NuSMV modella un sistema RFID composto da 3 tag RFID e un reader RFID. L'obiettivo del sistema è gestire la collisione tra i tag quando più di uno risponde contemporaneamente al reader. Per risolvere questo problema, sono stati utilizzati metodi non deterministici.

## Struttura del Codice

Il sistema è suddiviso in tre moduli principali: `main`, `Tag`, e `Reader`.

### `main` Module

Il modulo principale (`main`) contiene la definizione dei tag (`tag1`, `tag2`, `tag3`) e del reader (`reader`). Inoltre, sono dichiarate le variabili necessarie per generare numeri casuali.

#### Proprietà Verificate

Il sistema verifica alcune proprietà, sia in logica LTL che CTL, per garantire il corretto funzionamento del sistema RFID.

- **Raggiungibilità Composta**: Il reader entra nello stato di lettura e almeno un tag entra nello stato di successo.
```nusmv
LTLSPEC
    F((reader.status = reading) & X(tag1.status = success | tag2.status = success | tag3.status = success))
```
- **Liveness**: Se nessun tag è stato letto, prima o poi almeno un tag verrà letto.
```nusmv
LTLSPEC
    G((reader.read_tags = 0) -> F(reader.read_tags = 1))
```
- **Liveness Condizionata al Round X**: Se tutti i tag sono stati letti, il reader termina.
```nusmv
CTLSPEC
    AG((reader.read_tags = 3) -> AX(reader.status = terminated))
```
- **Raggiungibilità**: Prima o poi, se un tag risponde, sarà considerato come letto dal reader.
```nusmv
CTLSPEC 
    EF((tag1.status = responding) -> (tag1.status = success))
```
- **Liveness**: Se tutti i tag sono stati letti, il reader termina.
```nusmv
CTLSPEC 
    AG(((tag1.status = success) & (tag2.status = success) & (tag3.status = success) -> AF((reader.status = terminated))))
```

### `Tag` Module

Il modulo `Tag` rappresenta un singolo tag RFID e definisce il suo comportamento in base allo stato e alle operazioni del reader.

```nusmv
-- Esempio di codice nel modulo Tag
MODULE Tag(reader,rnd)
    VAR
        status : {idle, responding, colliding, success};
        id : unsigned word[10];
        
    ASSIGN
        -- Logica assegnamenti...
```

### `Reader` Module
Il modulo `Reader` rappresenta il reader RFID e gestisce la segnalazione, la lettura degli ID e la risoluzione delle collisioni.
```nusmv
-- Esempio di codice nel modulo Reader
MODULE Reader(tag1,tag2,tag3)
    VAR 
        status : {idle, signaling, waiting, resolving_collision, reading, terminated};
        reading_id : unsigned word[10];
        accepted_id : word[10];
        read_tags : 0..3;
        tree_level : 0..10;
        actual_bit : unsigned word[1];
    DEFINE
        -- Definizione...
```

## Come Funziona il Sistema
Quando un tag riceve un segnale dal reader, genera un bit in modo pseudocasuale. Il comportamento del tag varia a seconda del numero di tag che rispondono al reader:

- Se un solo tag risponde, il reader legge il suo ID e lo comunica al tag;
- Se più tag rispondono, il reader risolve la collisione chiedendo di generare un nuovo bit;
- Se nessun tag risponde, il reader torna in idle e controlla quelli che hanno generato 1;
- Una volta che nessuno di quelli che hanno generato 1 risponde, risale l'albero al nodo padre;
- Ripete finché eventualmente non risponde piú nessun tag.

In questo modo, il sistema gestisce in maniera non deterministica la collisione tra i tag.




**Note:** La scelta casuale degli ID é stata resa semi-deterministica a causa di una generazione pseudocasuale del numero.
