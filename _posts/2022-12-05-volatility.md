#Volatility
**Volatility ha bisogno di profili per funzionare**
Quando lanciamo un comando per analizzare la memoria, volatility identifica anche il sistema operativo, la versione e l'architettura.
Con il comando `volatility -f memdump.mem imageingo`, il tool, oltre ad eseguire l'istruzione, ci suggerirà il profilo da passare come parametro. 
Il profile è strettamente necesasrio per lanciare i successivi comandi del tool.

### Command List

**`volatility -f memdump.mem --profile=PROFILE pslist`** --> prende come parametro un'immagine, incluso il profilo del sistema, ed usa il plugin **pslist** per stampare a video la lista dei processi. 

**`volatility -f memdump.mem --profile=PROFILE pstree`** --> stampa un albero dei processi.

**`volatility -f memdump.mem --profile=PROFILE psscan`** --> stampa tutti i processi disponibili, inclusi quelli nascosti.

**`volatility -f memdump.mem --profile=PROFILE psxview`** --> stampa i processi aspettati e nascosti (combinazione tra **pslist** e **psscan**).

**`volatility -f memdump.mem --profile=PROFILE netscan`** --> identifica connessioni attive e chiuse.

**`volatility -f memdump.mem --profile=PROFILE timeliner`** --> crea una timeline degli eventi dall'immagine della memoria.

**`volatility -f memdump.mem --profile=PROFILE iehistory`** --> history del browser.

**`volatility -f memdump.mem --profile=PROFILE filescan`** --> identifica ogni file presente in memoria.  

**`volatility -f memdump.mem --profile=PROFILE dumpfiles -n --dump-dir=./`** --> dump dei file presenti in memoria nella cartella locale (path ./). 

A good list of volatility cheet sheet --> https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet

Approach to memory investigation --> https://medium.com/@zemelusa/first-steps-to-volatile-memory-analysis-dcbd4d2d56a1

