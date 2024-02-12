# Progetto_LPVS

Questo modello rappresenta un sistema composto da 3 Tag RFID e un reader RFID.
Il problema che si é affrontato é la collisione tra i tag, ovvero la situazione in cui piú tag rispondono al reader contemporaneamente. 
Per risolvere questo problema sono stati utilizzati metodi non deterministici, come da specifica.

Quando un tag riceve un segnale dal reader, genera un bit in maniera pseudocasuale.
In base a quanti tag rispondono al reader, puó comportarsi in maniera diversa:
 - se un solo tag risponde, allora il reader legge il suo id e lo comunica al tag;
 - se piú tag rispondono, allora il reader risolve la collisione, ovvero chiede di generare un nuovo bit;
 - se nessun tag risponde, il reader torna in idle e controlla quelli che hanno generato 1.
In questo modo, il reader riesce a leggere tutti i tag in maniera non deterministica.
