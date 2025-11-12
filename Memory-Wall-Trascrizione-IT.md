# Il Muro della Memoria: Passato, Presente e Futuro della DRAM

## Vincitori e Vinti nella Rivoluzione DRAM 3D

**Autori**: Dylan Patel, Jeff Koch, Tanj  
**Data**: 3 settembre 2024

---

## Introduzione: La Morte Silenziosa della Legge di Moore

Il mondo continua a discutere sulla morte della Legge di Moore, ma la vera tragedia è che essa è già morta più di un decennio fa senza alcun clamore o titolo nei giornali. L'attenzione si concentra generalmente sulla logica, ma la Legge di Moore si è sempre applicata anche alla DRAM.

**[IMMAGINE 1: Le Leggi di Scaling Originali - Fonte: 1965 The Future of Integrated Electronics - Gordon Moore]**

La DRAM non scala più. Nei giorni gloriosi, la densità di bit di memoria raddoppiava ogni 18 mesi, superando persino la logica. Questo si traduce in un aumento di densità di poco più di 100x ogni decennio. Ma nell'ultimo decennio, lo scaling si è rallenrato così tanto che la densità è aumentata solo di 2x.

**[IMMAGINE 2: Andamento della Densità DRAM - Fonte: SemiAnalysis]**

Ora con l'esplosione dell'IA, l'equilibrio dell'industria è stato alterato ancora di più. Mentre i chip logici hanno migliorato significativamente sia in densità che in costo per transistor nel tempo, i miglioramenti nelle velocità della DRAM sono stati lenti.

Nonostante le paure significative, il costo per transistor continua a scendere sui nodi 3nm e 2nm di TSMC. Con la memoria, invece, l'aumento della larghezza di banda è guidato da tecniche di packaging eroiche e costose.

**[IMMAGINE 3: Velocità vs Costo della Memoria - Fonte: Nvidia, SemiAnalysis]**

La memoria ad alta larghezza di banda (HBM), la spina dorsale della memoria degli acceleratori, costa 3x o più per GB rispetto alla DDR5 standard. I clienti sono costretti ad accettare questo poiché c'è poca alternativa se vogliono realizzare un pacchetto acceleratore competitivo. Questo equilibrio è instabile – le future generazioni di HBM continuano a diventare ancora più complesse con conteggi di strati più elevati. I bisogni di memoria per l'IA stanno esplodendo poiché i soli pesi del modello si avvicinano alla scala multi-TB.

Per H100, ~50%+ del costo di produzione è attribuito a HBM e con Blackwell, questo sale a ~60%+.

L'industria DRAM, in altre parole, ha colpito un muro. I miglioramenti del calcolo, sebbene rallentino, superano di gran lunga la memoria. Come può il ritmo dell'innovazione nella DRAM riaccelerarsi – e quali innovazioni possono essere sfruttate per migliorare la larghezza di banda, la capacità, il costo e il consumo di energia in futuro?

Ci sono molte possibili soluzioni. Con centinaia di miliardi in capex per l'IA sul tavolo, c'è un forte incentivo per l'industria a spingere queste soluzioni in avanti. Iniziando con un'introduzione sulla storia e il background della DRAM, copriremo ogni problema che compone il moderno "muro della memoria" e le possibili soluzioni. Discuteremo di idee relativamente più semplici e a breve termine come l'estensione della roadmap HBM e di opzioni più complesse e a lungo termine come il compute-in-memory (CIM), nuovi tipi di memoria come FeRAM (Ferroelectric RAM) o MRAM (Magnetic RAM), e l'imminente arrivo della DRAM 4F2 e DRAM 3D.

---

## Sezione 1: Primer sulla DRAM

### Working Memory (Memoria di Lavoro)

Esistono diversi tipi di memoria utilizzati in un computer. La più veloce è la SRAM (Static Random Access Memory), che è compatibile con le tecnologie di processo logico e si trova sulla CPU o GPU. Poiché si trova su un die logico, la SRAM è anche il tipo di memoria più costoso – circa 100x+ più costosa per byte rispetto alla memoria ad accesso casuale dinamico (DRAM) – e quindi viene utilizzata solo in piccole quantità.

L'estremità opposta dello spettro include le unità SSD NAND non volatili, i dischi rigidi e i nastri magnetici. Questi sono economici ma troppo lenti per molti compiti. La DRAM si trova nella zona "Riccioli d'Oro" tra SRAM e Flash – abbastanza veloce, abbastanza economica.

**[IMMAGINE 4: La Gerarchia della Memoria - Fonte: Enfabrica]**

La DRAM può costituire fino alla metà del costo di un sistema server non-AI. Eppure negli ultimi 10 anni è stata la più lenta a scalare di tutta la logica e la memoria principale. I chip DRAM da 16Gb sono stati resi disponibili ad alto volume 8 anni fa ma rimangono i più comuni oggi; quando sono stati introdotti costavano circa $3 per gigabyte e hanno raggiunto un picco di quasi $5 prima di scendere alla gamma di $3 negli ultimi 12 mesi. Le velocità sono, se possibile, leggermente più lente. La potenza ha visto il miglior progresso principalmente grazie all'aumento della LPDDR, un cambiamento di packaging che utilizza fili più corti e più efficienti, ma qui l'asticella è bassa. La mancanza di progresso nel scaling della DRAM è un collo di bottiglia di performance e economico che frena il calcolo.

### Architettura Base della DRAM

In linea di principio, la DRAM è semplice. Comprende un array di celle di memoria disposte in una griglia, ciascuna che memorizza un bit di informazione. Tutta la DRAM moderna utilizza una cella 1T1C, indicando 1 transistor e 1 condensatore. Il transistor controlla l'accesso alla cella, e il condensatore memorizza l'informazione sotto forma di una piccola carica elettrica.

**[IMMAGINE 5: Circuito DRAM Base - Un array di celle di memoria connesse con una wordline lungo ogni riga, bitline lungo ogni colonna. L'attivazione di 1 wordline e 1 bitline permette la lettura o la scrittura della cella dove si intersecano]**

Le wordline (WL) connettono tutte le celle in una singola riga; controllano il transistor di accesso per ogni cella. Le bitline (BL) connettono tutte le celle in una singola colonna; si connettono alla sorgente del transistor di accesso. Quando una wordline è energizzata, i transistor di accesso per tutte le celle nella riga si aprono e permettono il flusso di corrente dalla bitline nella cella (quando si scrive nella cella) o dalla cella alla BL (quando si legge dalla cella). Solo 1 wordline e 1 bitline saranno attive contemporaneamente, il che significa che solo la 1 cella dove si intersecano la wordline e bitline attive sarà scritta o letta.

**[IMMAGINE 6: Il Flusso di Carica - La carica è permessa di fluire dalla bitline al condensatore o viceversa quando il transistor di accesso è attivato dalla wordline - Fonte: Branch Education]**

La DRAM è una tecnologia di memoria volatile: i condensatori di archiviazione perdono carica, e quindi richiedono refresh frequenti (spesso ogni ~32 millisecondi) per mantenere i dati memorizzati. Ogni refresh legge il contenuto di una cella, aumenta la tensione sulla bitline a un livello ideale, e lascia che questo valore rinfrescato fluisca di nuovo nel condensatore. I refresh avvengono interamente all'interno del chip DRAM, senza dati che fluiscono dentro o fuori dal chip. Questo minimizza la potenza sprecata, ma i refresh possono ancora rappresentare il 10%+ del consumo totale di potenza della DRAM.

I condensatori, proprio come i transistor, sono stati ridotti a larghezze nanometriche ma anche con rapporti di aspetto estremi ~1.000nm di altezza ma solo decine di nm di diametro – i rapporti di aspetto si stanno avvicinando a 100:1, con capacità nell'ordine di 6-7 fF (femto-Farad). Ogni condensatore memorizza una carica estremamente piccola, circa 40.000 elettroni quando appena scritto.

La cella deve ricevere e inviare elettroni attraverso la bitline, ma la tensione posta sulla bitline è diluita da tutte le altre celle attaccate alla stessa bitline. La capacità totale della bitline può superare 30fF – una diluizione di 5x. La bitline è anche molto sottile il che rallenta gli elettroni. Infine, la cella potrebbe essersi scaricata significativamente se non è stata rinfrescata di recente, quindi ha solo una frazione di carica da consegnare.

Tutti questi fattori significano che scaricare una cella per leggerne il valore può risultare in un segnale molto debole che deve essere amplificato. A tal fine, gli amplificatori di sensazione (SA) sono attaccati alla fine di ogni bitline per rilevare le cariche estremamente piccole lette dalle celle di memoria e amplificare il segnale a una forza utile. Questi segnali più forti possono quindi essere letti altrove nel sistema come un binario 1 o 0.

L'amplificatore di sensazione ha un design di circuito intelligente: confronta la bitline attiva con un vicino corrispondente che non è in uso, iniziando con entrambe le linee portate a una tensione simile. La tensione sulla bitline attiva sarà confrontata con il vicino inattivo, spostando l'amplificatore di sensazione fuori dall'equilibrio e causandogli di amplificare la differenza di nuovo in quella bitline attiva, sia amplificando il segnale che guidando un valore pieno fresco, alto o basso, di nuovo nella cella che rimane aperta alla bitline. È una situazione "due piccioni con una fava": la cella viene letta e rinfrescata contemporaneamente.

Dopo aver letto/rinfrescato la cella attiva, il valore può essere copiato fuori dal chip o sovrascritto da un'operazione di scrittura. Una scrittura ignora il valore rinfrescato e usa un segnale più forte per forzare la bitline a corrispondere al nuovo valore. Quando la lettura o la scrittura è terminata, le wordline sono disabilitate, spegnendo i transistor di accesso e intrappolando così qualsiasi carica residente nei condensatori di archiviazione.

La DRAM moderna è resa possibile da due invenzioni separate e complementari: la cella di memoria 1T1C e l'amplificatore di sensazione.

### La Storia della DRAM (Quando la DRAM Ancora Scalava)

La cella di memoria 1T1C è stata inventata nel 1967 presso l'IBM dal Dr. Robert Dennard, ben noto anche per la sua legge di scaling dei transistor MOS. Sia la DRAM che lo scaling si basano su transistor MOS (metal oxide silicon, i strati nel gate del transistor).

**[IMMAGINE 7: Patent Originale di Dennard per l'Architettura di Cella di Memoria 1T1C - Fonte: U.S. Patent 3,387,286]**

Nonostante l'invenzione della struttura di cella di memoria 1T1C, la DRAM iniziale spedita da Intel nel 1973 utilizzava 3 transistor per cella con il gate sul transistor centrale che agiva come condensatore di archiviazione. Questo era una "gain cell" dove il transistor centrale e finale fornivano guadagno per amplificare la carica molto piccola sul gate centrale, abilitando la cella ad essere letta facilmente e senza disturbare il valore memorizzato.

Una cella 1T1C è teoricamente migliore: meno dispositivi, più semplice da collegare insieme, e più piccola. Perché non è stata immediatamente adottata? Non era ancora pratico leggere la cella.

Al momento dell'invenzione, la piccola capacità della cella 1T1C la rendeva non fattibile per operare. Una seconda invenzione chiave era necessaria: l'amplificatore di sensazione.

**[IMMAGINE 8: Prime Sperimentazioni dell'Amplificatore di Sensazione]**

Il primo amplificatore di sensazione moderno è stato sviluppato nel 1971 da Karl Stein presso Siemens, presentato a una conferenza in California, e completamente trascurato. L'architettura 1T1C non era ampiamente adottata a quel punto e Siemens non aveva idea di cosa fare con questa invenzione. Stein è stato spostato ad un altro incarico dove ha avuto una carriera di successo non correlata alla DRAM.

**[IMMAGINE 9: Patent Originale dell'Amplificatore di Sensazione di Stein - Fonte: U.S. Patent 3,774,176]**

Questo design era ben abbinato alla spaziatura delle bitline ed è stato in grado di scalare più piccolo per stare al passo con la dimensione della cella. L'amplificatore di sensazione è completamente spento quando non in uso il che consente di averne milioni su un chip senza drenare la potenza. Sono stati un piccolo miracolo.

Ci vollero più di 5 anni perché il momento dell'amplificatore di sensazione arrivasse. Robert Proebsting presso Mostek in modo indipendente (ri)scoprì il concetto e entro il 1977 il loro DRAM da 16kb con architettura 1T1C + SA divenne il leader di mercato. Questa formula vincente rimase – l'architettura DRAM è fondamentalmente la stessa quasi 5 decenni dopo.

### Quando la DRAM Ha Smesso di Scalare

Nel ventesimo secolo, la Legge di Moore e lo scaling di Dennard governavano l'industria dei semiconduttori. Al suo apice, gli aumenti di densità DRAM superavano la logica. La capacità DRAM per chip raddoppiava ogni 18 mesi, alimentando l'ascesa dei fab giapponesi (che hanno superato per la prima volta la quota di mercato statunitense nel 1981 e hanno raggiunto il picco di circa l'80% nel 1987) e successivamente le società coreane (la cui quota di mercato ha superato quella del Giappone nel 1998). La rapida sostituzione generazionale dei fab su un processo relativamente semplice ha creato opportunità per i nuovi entranti con i fondi per costruire il prossimo fab di generazione.

**[IMMAGINE 10: Prezzo per Bit nel Tempo - Fonte: Lee, K.H., A Strategic Analysis of the DRAM Industry After the Year 2000]**

Il prezzo per bit si è ridotto di 3 ordini di grandezza in 20 anni in un'"età d'oro" dello scaling DRAM.

Questo ritmo non era fattibile a lungo, e dalla fine del ventesimo secolo all'inizio del ventunesimo, la logica ha superato di gran lunga lo scaling della memoria. Lo scaling logico recente si è rallenrato a un ritmo di miglioramenti di densità del 30-40% ogni 2 anni. Ma questo è ancora buono in confronto alla DRAM che è approssimativamente un ordine di grandezza più lenta del suo picco, ora richiedendo 10 anni per un aumento di densità di 2x.

**[IMMAGINE 11: Cicli di Memoria nel Tempo - "It's different this time": no, i cicli di memoria sono stati parte dell'industria per 50 anni - Fonte: Lee, K.H., A Strategic Analysis of the DRAM Industry After the Year 2000]**

Questo rallentamento dello scaling ha avuto effetti a cascata sulla dinamica dei prezzi della DRAM. Mentre la memoria è stata tradizionalmente un'industria ciclica, lo scaling di densità lento ha significato una riduzione dei costi molto inferiore per attenuare gli aumenti di prezzo quando l'offerta è limitata. L'unico modo per aumentare l'offerta DRAM è costruire nuovi fab. Le selvagge oscillazioni di prezzo e il CAPEX elevato significano che solo le più grandi società sopravvivono: più di 20 produttori producevano DRAM a metà degli anni '90, con l'80% di quota di mercato distribuito tra i primi 10. Ora i primi 3 fornitori possiedono più del 95% del mercato.

Poiché la DRAM è commoditizzata, i fornitori sono intrinsecamente molto più suscettibili alle fluttuazioni di prezzo (in contrasto con la logica o l'analogico) e devono competere principalmente sui prezzi grezzi delle loro merci quando il mercato è basso. La logica ha mantenuto solo la Legge di Moore con costi crescenti, la DRAM non ha questo lusso. Il costo della DRAM è semplice da misurare, $/Gb. Relativo ai periodi precedenti, gli ultimi 10 anni hanno visto un lento calo dei prezzi – solo 1 ordine di grandezza in un decennio quando ci voleva metà di quel tempo. Il caratteristico comportamento di picchi e valli della DRAM è evidente anche.

**[IMMAGINE 12: Densità DRAM e Prezzo nel Tempo - La densità DRAM scala a 2x per decennio, mentre il prezzo è guidato da effetti ciclici - Fonte: DRAMExchange, SemiAnalysis]**

Da quando è entrato nei nodi a 10nm, la densità di bit DRAM si è stagnata. Anche l'aggiunta di EUV nei nodi 1z di Samsung e 1a di SK Hynix non ha aumentato significativamente la densità. Due sfide notevoli sono nei condensatori e negli amplificatori di sensazione.

I condensatori sono difficili sotto molti aspetti. In primo luogo, il patterning è impegnativo poiché i fori devono essere strettamente imballati con molto buon controllo della dimensione critica (CD) e controllo di sovrapposizione, per contattare i transistor di accesso sottostanti e evitare il bridging o altri difetti. I condensatori hanno un rapporto di aspetto molto alto e incidere un profilo di foro dritto e stretto è eccezionalmente difficile, ulteriormente aggravato dalla necessità di una maschera più spessa per abilitare un'incisione più profonda poiché una maschera più spessa richiede un fotoresist più spesso che è più difficile da disegnare.

Successivamente, i molteplici strati privi di difetti di pochi nm di spessore devono essere depositati sulle pareti di tutto il profilo del foro per formare il condensatore. Quasi ogni passo mette a dura prova i limiti della moderna tecnologia di elaborazione.

**[IMMAGINE 13: Struttura del Condensatore DRAM - Richiede molti strati esquisiti formati in un foro con rapporto di aspetto 100:1 (non in scala – i veri condensatori potrebbero essere 10x più alti di quanto mostrato) - Fonte: Applied Materials]**

Gli amplificatori di sensazione sono una storia simile agli interconnettori logici. Una volta una considerazione secondaria, sono ora di difficoltà uguale o anche maggiore rispetto alle caratteristiche "principali" (transistor logici e celle di memoria). Sono schiacciati da più lati. Lo scaling dell'area deve essere fatto per corrispondere al restringimento della bitline, con gli amplificatori di sensazione che diventano meno sensibili e più propensi a variazioni e perdite poiché vengono resi più piccoli. Allo stesso tempo, i condensatori più piccoli immagazzinano meno carica, quindi il requisito di sensazione per leggerli diventa più difficile.

Ci sono altre sfide pure, con il risultato netto che scalare la DRAM in modo economico è sempre più difficile usando approcci tradizionali. La porta è aperta per nuove idee – esploriamone alcune...

---

## Sezione 2: Roadmap a Breve Termine – 4F2 e Transistor a Canale Verticale

Nel breve termine, lo scaling della DRAM continuerà lungo la sua roadmap tradizionale. I cambiamenti architetturali più grandi e fondamentali richiederanno anni per essere sviluppati e implementati. Nel frattempo, l'industria deve rispondere alla necessità di migliori prestazioni, anche se solo con miglioramenti marginali.

La roadmap a breve termine ha 2 innovazioni: il layout di cella 4F2 e i transistor a canale verticale (VCT).

**[IMMAGINE 14: Roadmap DRAM di Samsung - Fonte: Samsung Memcon 2024, originariamente pubblicato da SemiEngineering]**

Nota che alcune società, inclusa Samsung nella loro roadmap, mettono VCT sotto il banner "3D". Mentre tecnicamente vero, questo è un po' fuorviante poiché VCT è distinto da quello che è comunemente chiamato "DRAM 3D".

### Architettura 4F2 vs 6F2

**[IMMAGINE 15: Layout Standard 6F2 vs 4F2 con Transistor a Canale Verticale - Fonte: CXMT IEDM 2023]**

4F2 descrive l'area della cella di memoria in termini della dimensione della caratteristica minima F, simile alla metrica della traccia per l'altezza della cella logica standard, ad esempio una "cella 6T". La dimensione della caratteristica minima è generalmente la larghezza della linea o dello spazio, nella DRAM sarà la larghezza della wordline o della bitline. È un modo semplice per indicare la densità di un layout di cella e rende il confronto facile – una cella 4F2 è solo i 2/3 della dimensione di una cella 6F2, offrendo un aumento teorico di densità del 30% senza scalare la dimensione della caratteristica minima. Nota che il puro layout della cella non è l'unico limite allo scaling della densità, quindi i benefici reali saranno probabilmente inferiori al caso ideale del 30%.

4F2 è il limite teorico per una cella di bit singolo. Ricorda che la dimensione della caratteristica è la larghezza della linea o dello spazio (cioè il semi-pitch), quindi un pattern di linea + spazio avrà un pitch di 2F, non F, e quindi la dimensione minima possibile della cella è 4F2 non solo F2. Quindi una volta che questa architettura è raggiunta, l'unico percorso per lo scaling orizzontale è scalare F stesso – qualcosa che sta diventando rapidamente impratico, se non addirittura impossibile.

La DRAM ha utilizzato un layout 6F2 dal 2007, con 8F2 prima di questo (nota interessante: la NAND moderna utilizza già una cella 4F2 ma con una dimensione di caratteristica F significativamente più grande. La SRAM è nell'ordine di 120 F2, 20x meno densa!)

Una notevole eccezione è CXMT, un fornitore cinese che ha utilizzato VCT e un layout 4F2 nella loro DRAM a 18nm che ha aggirato le sanzioni, dimostrata alla fine del 2023. Poiché Samsung, SK Hynix e Micron erano in grado di scalare le celle, non sono stati costretti ad adottare queste architetture allo stesso modo in cui CXMT è stata. L'implicazione della prima adozione di CXMT è anche importante – è probabile che stiano avendo difficoltà a scalare F poiché hanno optato per il cambiamento più drastico in architetture di cella e transistor.

### Transistor a Canale Verticale (VCT)

L'abilitatore chiave per le celle 4F2 è il transistor a canale verticale. È necessario semplicemente perché il transistor deve scalare per stare nella cella e entrambi i contatti – a bitline e a condensatore – devono anche stare in quell'impronta, quindi, una linea verticale. A queste scale diventa necessario costruire il transistor verticalmente invece che orizzontalmente, riducendo la sua impronta a circa 1F, più o meno corrispondendo al condensatore sopra di esso, pur mantenendo una lunghezza di canale sufficiente per il transistor per operare efficacemente.

La DRAM attuale utilizza canali orizzontali e source/drain con separazione orizzontale. Queste sono un'architettura matura e ben compresa. I VCT impilano una sorgente (connessa al BL sotto di essa), un canale (circondato da gate & la wordline che controlla il gate), e uno scarico (connesso al condensatore sopra) in sequenza. Ci sono compromessi nella fabbricazione dove alcuni passaggi diventano più facili e altri più difficili, ma nel complesso i VCT sono più difficili da produrre.

Il processo di Samsung è notevole per l'uso del wafer bonding. In un processo simile alla distribuzione di potenza sul retro per la logica, i transistor di accesso della cella sono fabbricati con bitline formate in cima prima di capovolgere il wafer e legarlo a un wafer di supporto, quindi la bitline è ora sepolta. Interessantemente, la base legata non sembra avere bisogno di un allineamento accurato con i VCT sebbene la divulgazione non spieghi se la CMOS periferica sarà sul chip capovolto, o nella base appena legata. Il lato superiore è assottigliato per esporre l'altro capo dei transistor così che i condensatori di archiviazione possono essere costruiti sopra di essi. EVG e TEL faranno guadagni da questa nuova necessità incrementale di strumenti di bonding dei wafer.

---

## Sezione 3: Varianti Attuali di DRAM

La DRAM viene in molte varietà, ognuna ottimizzata per diversi obiettivi. I sapori di generazione più recente rilevanti sono DDR5, LPDDR5X, GDDR6X e HBM3/E. Le differenze tra loro risiedono quasi interamente nei circuiti periferici. Le celle di memoria stesse sono simili attraverso le varietà e i metodi di fabbricazione sono ampiamente simili per tutti i tipi. Introduciamo brevemente i vari sapori di DRAM e il ruolo di ognuno.

### DDR5

DDR5 (Double Data Rate gen. 5) fornisce la più alta capacità di memoria in quanto è imballato in moduli di memoria in linea doppia (DIMM). La DRAM DDR5 è lo standard per i server e i sistemi desktop tradizionali, offrendo ampie capacità a un costo ragionevole. Tuttavia, la larghezza di banda è limitata dalle connessioni del bus condiviso e dalle velocità di comunicazione ridotte, rendendo questo il fondamentale "Goldilocks" della memoria.

### LPDDR5X

LPDDR5X (Low Power DDR5 con X significando migliorata) fornisce operazione a basso consumo ma richiede connessioni a distanze più corte e bassa capacità alla CPU che limitano la capacità, quindi viene utilizzato in telefoni cellulari e laptop dove il basso consumo è desiderabile e i vincoli di layout tollerabili.

Più recentemente abbiamo visto packaging di capacità più elevata per LPDDR usati in alcuni acceleratori AI, workstation professionali di Apple, e CPU alimentatori di AI come Grace. Questi nuovi usi sono guidati dalla ricerca di trasferimenti dati efficienti energeticamente e ad alta larghezza di banda.

Negli acceleratori, LPDDR è emerso come l'opzione migliore per un "secondo livello" di memoria che fornisce capacità più economica a un livello inferiore (più lento) rispetto a HBM costoso. Non è all'altezza nel costruire capacità massime e nelle caratteristiche di affidabilità, ma batte i DIMM DDR5 in quanto consuma un ordine di grandezza meno energia per bit di throughput.

Il packaging LPDDR5X sale fino a 480GB disponibile sul processore Nvidia Grace, che è circa 10x il limite di capacità per le configurazioni GDDR (che sono limitate dalle regole di layout dei circuiti stampati e imballaggio dei chip richiesti per soddisfare i segnali nei sistemi di gioco consumer), e nello stesso intervallo di configurazioni DDR server medio.

La capacità LPDDR5X più grande è possibile utilizzando R-DIMM di dimensioni superiori a 128GB, sebbene costosa a causa della complessità dell'imballaggio e dei Registri aggiuntivi (una sorta di chip buffer) sui DIMM.

LPDDR5X ha un grande vantaggio nel consumo di potenza vs. DDR e nel costo vs. HBM, ma l'energia per bit non può sfidare HBM e richiede molti corsie (connessioni alla CPU) che affollano i layout dei circuiti stampati a capacità maggiori. Ha anche una storia debole sulla correzione degli errori (ECC) che diventa più importante a capacità maggiori poiché c'è una maggiore probabilità di un errore. Per compensare, una certa capacità deve essere dirottata a supportare ECC extra. Ad esempio, la CPU Grace ha 512GB di LPDDR5X per vassoio di calcolo ma sembra riservare 32GB per le caratteristiche di affidabilità, lasciando 480GB disponibile per l'uso.

Lo standard LPDDR6 imminente mostra poco miglioramento, mantenendo conteggi di corsie elevati per chip e aumenti di velocità relativamente lievi insieme a supporto limitato per la correzione degli errori. LPDDR6 non fornirà un concorrente HBM.

### GDDR6X

GDDR6X (G per Graphics) è focalizzato sulle applicazioni grafiche, offrendo alta larghezza di banda a basso costo ma con latenza più elevata e consumo di potenza più elevato. Sebbene utile nelle GPU di gioco, è stato progettato con limiti di capacità a livello di circuito stampato e livelli di potenza che limitano la dimensione delle applicazioni AI che possono usarlo.

### HBM3E

Poi c'è HBM3E (High Bandwidth Memory gen. 3, con una versione "E" migliorata). Dà priorità alla larghezza di banda e all'efficienza energetica ma è molto costoso. Le 2 caratteristiche definenti di HBM sono la larghezza di bus molto più ampia e lo stacking verticale della memoria. I singoli die HBM hanno 256 bit ciascuno di I/O, 16x più della LPDDR che ha una larghezza di bus di solo 16 bit per chip. I die sono impilati verticalmente, tipicamente 8 o più, con I/O raggruppati per ogni 4 die; in totale il pacchetto può fornire 1024 bit di larghezza di banda. In HBM4 questo raddoppierà a 2048 bit. Per trarre il massimo da HBM è meglio co-imballato accanto al motore di calcolo per ridurre la latenza e l'energia per bit. Per espandere la capacità mantenendo una connessione breve al calcolo, più die devono essere aggiunti allo stack.

L'alto costo di HBM è principalmente guidato da questa necessità di stacking dei die. In uno stack HBM tipico, 8 o 12 die DRAM (con 16 e oltre sulla roadmap) sono impilati uno sopra l'altro, con alimentazione e segnale instradati da Via Attraverso Silicio (TSV) in ogni die. I TSV sono fili che passano direttamente attraverso il chip, che permettono la connessione tra i chip. I TSV sono molto più densi, più performanti, e più costosi rispetto ai metodi di wire-bonding più vecchi usati per connettere i chip impilati. Più di 1.200 fili di segnale devono essere instradati tramite TSV in uno stack HBM. Un'area significativa deve essere dedicata a loro, rendendo ogni die HBM DRAM il doppio della dimensione di un die DDR standard per la stessa capacità. Questo significa anche requisiti di binning più elevati per la performance elettrica e termica per il die DRAM.

Questa complessità sottrae dal rendimento. Ad esempio, gli errori di progettazione DRAM di Samsung e il loro uso di un trailing node 1α stanno contribuendo ai loro rendimenti HBM scioccantemente scarsi. L'imballaggio è l'altra sfida maggiore. L'allineamento corretto di 8+ die con migliaia di connessioni ciascuno è difficile e quindi costoso a causa dei rendimenti relativamente bassi. Al momento questo è uno dei principali differenziatori tra i fornitori HBM, poiché SK Hynix può produrre con successo HBM3E con il loro imballaggio MR-MUF mentre Samsung fatica a rendere il loro prodotto. Micron ha una soluzione fattibile, ma ha bisogno di scalare significativamente la produzione.

Nonostante i costi elevati e le sfide di rendimento, HBM3E è, per ora, il prodotto più prezioso e ad alto margine che l'industria della memoria abbia mai avuto. Questo è principalmente perché per i grandi acceleratori AI di modello, nessun altro sapore di DRAM è un'alternativa fattibile. Mentre i margini probabilmente si eroderanno poiché Samsung migliora il rendimento, e Micron mentre scalano la produzione, l'appetito della memoria degli acceleratori AI continuerà a crescere – in una certa misura compensando il beneficio di questa nuova offerta.

**[IMMAGINE 16: Confronto HBM - HBM domina in larghezza di banda e densità di imballaggio molto alta. Fonte: SemiAnalysis]**

In breve, l'alta larghezza di banda e la densità di imballaggio molto alta insieme alla migliore energia per bit e vera capacità ECC rendono HBM3E il chiaro vincitore, per ora, per gli Acceleratori AI. Questo è il motivo per cui prodotti come Nvidia H100 e AMD MI300X lo usano. GDDR6/X arriva in secondo lontano dalle stesse metriche sebbene con capacità minuscola. LPDDR5 e DDR5 sono ancora peggio, nessuno dei due è adatto alle esigenze dell'acceleratore.

---

## Sezione 4: Roadmap HBM

La soluzione HBM attuale è costosa e sarà sempre più difficile da scalare. Come siamo finiti in questa situazione?

HBM è una soluzione di imballaggio costruita intorno a idee DRAM legacy, ma imballata con densità e adiacenza per provare a risolvere i problemi di larghezza di banda e potenza per l'AI e altre forme di calcolo ad alte prestazioni.

**[IMMAGINE 17: Stack HBM e Configurazione - Tutti i principali GPU AI ora usano HBM come loro memoria. I piani per il 2025 hanno HBM3e a 12-Hi con chip da 32 Gb per un totale di 48 GB per stack, con velocità dati a 8 Gbps per filo]**

Nei server GPU il primo versione di memoria unificata con CPU di supporto ha lanciato con AMD MI300A e Nvidia Grace Hopper. La CPU Grace ha capacità LPDDR5X ad alta capacità, mentre la GPU ha HBM3 ad alta larghezza di banda. Tuttavia, la CPU e la GPU sono su pacchetti separati, connessi su NVLink-C2C a 900 GB/s. Questo modello è più semplice da integrare ma più difficile dal lato software. La latenza della memoria connessa all'altro chip è molto più alta e potrebbe influenzare un numero significativo di workload. Come tale, la memoria non è del tutto uniforme e viene con i suoi propri problemi.

**[IMMAGINE 18: HBM4 e Roadmap Futura - Fonte: Samsung, Micron]**

HBM4 è a pochi anni di distanza, con Samsung e Micron che affermano che sarà fino a 16-Hi con 1.5 TB/s per stack. Questa è più che il doppio della larghezza di banda di quello che abbiamo oggi a solo 1.3-1.5x della potenza, ma questo scaling non è sufficiente, poiché il consumo di potenza della memoria continua ad aumentare complessivamente. HBM4 cambierà anche a larghezza di 2048-bit per stack, riducendo i tassi dati di un piccolo importo a 7.5 Gbps, aiutando con il consumo di potenza e l'integrità del segnale. È probabile che i tassi dati aumenteranno ai livelli di HBM3E con HBM4E o qualcosa di simile.

L'altro cambiamento significativo è nel die base HBM. Il die base sarà fabbricato su processi FinFET al contrario della tecnologia CMOS planare usata ora. Per Micron e SK Hynix che non hanno questa capacità logica, il die base sarà fabbricato da una fonderia con TSMC che già fa annunci che saranno il partner per SK Hynix. Inoltre, ci sarà personalizzazione del die base per clienti individuali.

Abbiamo un rapporto separato sulla personalizzazione HBM in arrivo, ma un veloce primer qui: gli annunci HBM4 predicono che almeno 2 diverse forme di chip base saranno in uso, permettendo all'interfaccia della memoria di essere ottimizzata per diverse velocità e lunghezze. È probabile che la funzionalità che controlla la macchina a stati DRAM si sposti sul chip base per controllare più efficientemente i chip DRAM, e le connessioni solo verticali possono permettere all'energia per bit di essere ridotta.

L'HBM personalizzato può abilitare molteplici architetture di pacchetto altre fuori dagli assemblaggi convenzionali basati su CoWoS che vediamo oggi. Potrebbe esserci PHY ripetitore per daisy chain molteplici file di HBM, sebbene qualsiasi cosa oltre 2 gradi vedrebbe rendimenti decrescenti.

**[IMMAGINE 19: Configurazione HBM Personalizzato - Fonte: SK Hynix]**

Con HBM4 e successori, il passaggio al bonding ibrido è stato suggerito. Questo permetterà stack HBM più sottili poiché il gap del bump è rimosso, e dissipazione del calore migliorata. Inoltre, permetterà altezze dello stack di 16-20+ strati. Potrebbe anche ridurre il consumo di potenza di un piccolo importo poiché la distanza fisica che i segnali percorreranno sarà ridotta. Le sfide sono sostanziali però – rendere uno stack incollato di 16+ die, nessuno perfettamente piatto, non è facile – nessuno è vicino a una soluzione di produzione ad alto volume pronta al momento.

Tutto HBM4 iniziale non userà il bonding ibrido, e ci aspettiamo che rimanga vero per molto più a lungo di quanto la maggior parte spererebbe.

---

## Sezione 5: Compute In Memory (CIM)

### Il Problema: Architettura "Dumb" DRAM

La DRAM è stata ostacolata dall'inizio dalla sua architettura. È una semplice macchina a stati senza alcuna logica di controllo, il che aiuta a mantenere il costo basso, ma significa che dipende dall'host (CPU) per controllarla.

Questo paradigma è fermamente radicato: i processi di fabbricazione DRAM moderni sono così pesantemente ottimizzati e specializzati che non possono realisticamente produrre logica di controllo. Il gruppo industriale JEDEC (Joint Electron Devices Engineering Council) applica anche intrusioni minime dalla logica quando sviluppa nuovi standard.

**[IMMAGINE 20: DRAM "Dumb" - La logica di controllo è separata dalla memoria, quindi i comandi devono passare attraverso un'interfaccia lenta e inefficiente - Fonte: SemiAnalysis]**

Il chip DRAM è totalmente dipendente dall'host: tutti i comandi sono incanalati attraverso un'unica interfaccia condivisa per molteplici banche nella memoria, per conto di molteplici thread nell'host. Ogni comando richiede 4 o più passaggi per essere emesso con temporizzazione precisa per mantenere il corretto funzionamento della DRAM. I chip DRAM non hanno nemmeno la logica per evitare i conflitti.

Questo è esacerbato dall'uso di un'antica interfaccia half-duplex: un chip DRAM può leggere o scrivere dati ma non entrambi contemporaneamente. L'host ha un modello esatto della DRAM e deve prevedere se l'interfaccia dovrebbe essere impostata per leggere o scrivere per ogni ciclo di clock. I comandi e i dati sono inviati su fili separati, il che riduce la complessità della temporizzazione ma aumenta i conteggi di filo e l'"affollamento della spiaggia" sulla GPU o CPU. Nel complesso, l'interfaccia di memoria ha sceso un ordine di grandezza sotto i tassi di bit, la densità della spiaggia, e l'efficienza dei PHY alternativi usati dai chip logici.

L'esito di questi svantaggi è che i DIMM DDR5, i più comuni sui server, spendono più del 99% dell'energia di lettura o scrittura nel controller host e nell'interfaccia. Altre varianti sono leggermente migliori – l'uso dell'energia HBM è approssimativamente 95% interfaccia, 5% lettura/scrittura della cella di memoria – ma ancora da nessuna parte vicino al potenziale completo della DRAM.

La funzionalità è semplicemente nel posto sbagliato. Naturalmente, la soluzione è spostare quella nel posto corretto: la logica di controllo dovrebbe essere su-chip con la memoria. Questo è Compute in Memory (CIM).

### Il Potenziale: Sbloccare i Bank DRAM

**[IMMAGINE 21: Potenziale dei Bank DRAM - Le banche DRAM hanno un potenziale di performance incredibile che va quasi completamente sprecato a causa delle interfacce - Fonte: SemiAnalysis]**

I bank sono l'unità base della costruzione DRAM. Comprendono 8 sub-bank ciascuno con 64Mb (8k righe x 8k bit) di memoria. Il bank attiva e rinfrescia 1 riga di 8k bit contemporaneamente ma trasferisce solo 256 di loro dentro o fuori in qualsiasi operazione di I/O. Questa limitazione è dovuta alle connessioni esterne dagli amplificatori di sensazione: mentre la riga è supportata da 8k amplificatori di sensazione, solo 1 in 32 amplificatori di sensazione (256) sono connessi fuori dal sub-bank, il che significa che le operazioni di lettura o scrittura sono limitate a 256 bit.

**[IMMAGINE 22: Dettagli del Bank DRAM - (a) La densa stuoia di condensatori alti limita l'accesso agli amplificatori di sensazione - Fonte: SemiAnalysis. (b) Una dissezione di microscopia ionica focalizzata [FIB] della regione dell'amplificatore di sensazione di una DRAM DDR4 - Fonte: Marazzi et al. "HiFi-DRAM: Enabling High-Fidelity DRAM Research by Uncovering Sense Amplifiers with IC Imaging", ISCA 2024 (c) Un grafico del bordo della regione Mat in una DRAM 1β - Fonte: Micron]**

Gli amplificatori di sensazione sono in un canyon circondato da condensatori alti. Nel teardown FIB sopra da ETH Zurich puoi vedere che c'è cablaggio ai livelli più alti che ha bisogno di via alti che si estendono per fare contatti agli amplificatori di sensazione.

Anche con questa interfaccia limitata, 1 in 32 accessibile in qualsiasi momento, la capacità di picco di lettura/scrittura di un bank è approssimativamente 256Gb/s, con una media più vicina a 128 Gb/s poiché almeno il 50% del tempo è usato nel passaggio a una nuova riga attiva. Con 32 bank per chip da 16Gb il potenziale completo di un chip è 4TB/s.

Più in alto nella gerarchia, i bank sono connessi in gruppi di bank, che a loro volta si connettono all'interfaccia fuori dal chip DRAM. In HBM, ogni die ha 256 linee dati con un throughput di picco di 256 GB/s per die. Questo collo di bottiglia può utilizzare solo 1/16 del potenziale sottostante dei bank.

**[IMMAGINE 23: Bottleneck della Larghezza di Banda - Visualizzazione dei livelli multipli di collo di bottiglia - Fonte: SemiAnalysis]**

Ad aggravare la situazione, 2pJ di energia sono necessari per trasferire un singolo bit fuori dal chip, 20x più di quanto ci è voluto per muoverlo dentro o fuori della cella. La maggior parte di questo accade alle due interfacce su ogni estremità dei fili DQ (Data Question-mark, una linea dati che è usata sia per lettura che per scrittura), e nella logica del controller sull'host.

Con un'architettura così dispendiosa, è inevitabile che gli sforzi saranno fatti per accedere a più del potenziale di performance.

### Il Pieno Potenziale della DRAM

**[IMMAGINE 24: Potenziale Completo - Anche semplici esempi teorici mostrano c'è un potenziale massivo in offerta - Fonte: SemiAnalysis]**

Anche semplici esempi teorici mostrano che c'è un potenziale massivo in offerta qui. Implementare lo standard UCIe (Universal Chiplet Interconnect) permetterebbe un throughput di 11 Tbps per mm di bordo – quasi 12x migliore di HBM3E. L'energia per bit andrebbe giù di un ordine di grandezza da 2pJ a 0.25pJ. E UCIe non è nemmeno l'ultima soluzione... lo standard proprietario Nulink di Eliyan, per prendere solo un esempio, rivendica anche miglioramenti più grandi.

**[IMMAGINE 25: Miglioramenti Potenziali della Larghezza di Banda - Fonte: Tom's Hardware]**

Il caveat qui è che se la fabric dell'host è estesa attraverso l'interfaccia, allora un sottoinsieme del comando fabric deve essere gestito dal lato DRAM. Ogni bank avrebbe bisogno di implementare la macchina a stati (pre-charge, selezione indirizzo, attiva, lettura/scrittura, chiudi, ecc.) localmente. Questo richiede logica (relativamente) complessa fabbricata su-chip con la DRAM.

Aggiungere logica a un chip DRAM non è, naturalmente, un compito semplice. La buona notizia è che HBM include un chip base CMOS, e quando la DRAM 3D arriva c'è una certezza virtuale che la buona logica CMOS è legata sopra o sotto lo stack di memoria. In altre parole, l'architettura è amenable all'incluzione di un po' di calcolo dentro la memoria, e i produttori di chip saranno incentivati a farlo.

### Il Cammino in Avanti

Ci sono frutti bassi qui: considera cosa potrebbe essere fatto se HBM adottasse il tasso GDDR7 di 32Gbps per filo dati. GDDR7 dimostra che i transistor abbastanza veloci possono essere fatti sui chip DRAM, e la distanza verticale attraverso i TSV al stack base è sotto 1mm il che dovrebbe mantenere l'energia per bit nella gamma 0.25pJ/bit. Solleva la domanda: perché JEDEC non si appoggerebbe a uno standard migliorato qui?

Le interfacce esterne sul chip base potrebbero essere sostanzialmente aggiornate ai design moderni offrendo più di un terabyte/sec per mm di bordo, a frazioni di pJ di energia per bit. Qualcuno vincerà grosso nelle guerre di IP. Mentre è possibile che JEDEC adotti una scelta come standard, è più probabile che sia fatto da coppie di vendor memoria / GPU che si muovono più veloce, poiché JEDEC di solito impiega anni.

**[IMMAGINE 26: Architettura CIM Futura - Visualizzazione della potenziale evoluzione con compute integrato - Fonte: SemiAnalysis]**

Vediamo già il cambiamento reale possibile in HBM4 con l'accettazione di chip base di terze parti, il che è destinato a scatenare esperimenti. Probabilmente vedremo controllo di canale offloadato, pura estensione di fabric su l'interconnessione, energia ridotta per bit su centimetri di distanza, e daisy chaining ad altre file di HBM più lontane dall'host, o a memoria di secondo livello come banche di LPDDR.

In questo modo i design possono aggirare i limiti di potenza di provare a fare il calcolo dentro lo stack di memoria e invece usare un'interfaccia modernizzata sul chip base per permettere ai chip vicini la larghezza di banda e bassa energia per bit per il calcolo come-se-in-memoria.

---

## Sezione 6: Memoria Emergente

### FeRAM (Ferroelectric RAM)

Per così lungo quanto DRAM e NAND sono stati di riferimento, c'è stata ricerca in migliori alternative. Il termine ombrello per queste è "memorie emergenti". È un po' un termine sbagliato poiché, finora, nessuno di loro è riuscito a "emergere" in un prodotto ad alto volume. Dato i nuovi sfide e incentivi circondando l'IA però, meritano almeno una breve discussione.

La memoria più promettente per le applicazioni discrete è FeRAM. Invece di usare un dielettrico (materiale isolante) nel condensatore di archiviazione, usano un ferroelettrico (un materiale che si polarizza in un campo elettrico). Questi hanno la caratteristica desiderabile di essere non-volatile, cioè possono immagazzinare dati quando spento e non sprecare energia o tempo su refresh.

Micron ha mostrato risultati promettenti a IEDM 2023 con densità comparabile al loro DRAM D1β insieme con buone performance di resistenza e ritenzione. In altre parole un buon candidato per uso AI/ML se non fosse per un problema: il costo. È complesso da produrre e fa più uso di materiali esotici rispetto alla DRAM convenzionale, al punto che semplicemente non è competitivo al presente.

### MRAM (Magnetic RAM)

MRAM è un'altra area di ricerca promettente. Invece di usare cariche elettriche, i dati sono immagazzinati per mezzo magnetico. La maggior parte dei design usa giunzioni tunnel magnetiche (MTJ) come cella di archiviazione di bit.

**[IMMAGINE 27: MRAM - Magnetic Tunnel Junction RAM, usando il meccanismo magnetico piuttosto che elettrico - Fonte: SK Hynix]**

A IEDM 2022, SK Hynix e Kioxia hanno mostrato una cella MTJ a 1-selettore con pitch a 45nm e dimensione critica a 20nm. Insieme, hanno raggiunto la più alta densità MRAM a oggi di 0.49 Gb/mm2, maggiore del DRAM D1β di Micron che ha una densità di 0.435 Gb/mm2. La cella caratterizza persino un design 4F2. Il loro obiettivo è productizzare in pacchetti discreti come alternativa a DRAM.

Al presente nessuna delle memorie alternative è ben posizionata per sfidare DRAM. Alcuni hanno celle più grandi o più lente. Alcuni hanno processi più costosi. La maggior parte ha resistenza limitata. Alcuni hanno basso rendimento. In pratica, i prodotti spediti per memorie magnetiche o a cambio di fase sono dimensionati in MB non GB. Questo potrebbe cambiare, c'è un sacco di soldi in gioco e una combinazione vincente potrebbe esistere in segreto, ma c'è un sacco di lavoro sia sui dispositivi che sulla scala della produzione da fare.

---

## Sezione 7: DRAM 3D - Basics (Contenuto a Pagamento)

[Nota: Questa sezione è riservata ai sottoscrittori paganti secondo il documento originale]

---

## Conclusione: Il Futuro della Memoria

La DRAM è in una fase critica di transizione. Mentre la Legge di Moore e i metodi di scaling tradizionali rallentano, l'industria deve innovare per rispondere alla crescente domanda di memoria per l'IA e i sistemi ad alte prestazioni.

Il breve termine offre marginalità attraverso 4F2, VCT e ottimizzazioni HBM. Nel medio termine, le soluzioni di Compute in Memory, le interfacce modernizzate e i chip base personalizzati promettono miglioramenti significativi in larghezza di banda e efficienza energetica. Nel lungo termine, la DRAM 3D, le memorie emergenti come FeRAM e MRAM, e paradigmi completamente nuovi potrebbero trasformare il paesaggio della memoria.

Con centinaia di miliardi in capex per l'IA sul tavolo, i vincitori di questa rivoluzione della memoria guadagneranno posizioni competitive massiccie. Le aziende che innovano oggi nella memoria definiranno il futuro del calcolo domani.

---

## Referenze Immagini Completa

1. **Immagine 1**: Le Leggi di Scaling Originali - Fonte: 1965 The Future of Integrated Electronics – Gordon Moore
2. **Immagine 2**: Andamento della Densità DRAM - Fonte: SemiAnalysis
3. **Immagine 3**: Velocità vs Costo della Memoria - Fonte: Nvidia, SemiAnalysis
4. **Immagine 4**: La Gerarchia della Memoria - Fonte: Enfabrica
5. **Immagine 5**: Circuito DRAM Base - Array di celle di memoria con wordline e bitline
6. **Immagine 6**: Il Flusso di Carica - Transistor di accesso e condensatore - Fonte: Branch Education
7. **Immagine 7**: Patent di Dennard 1T1C - U.S. Patent 3,387,286
8. **Immagine 8**: Prime Sperimentazioni dell'Amplificatore di Sensazione
9. **Immagine 9**: Patent dell'Amplificatore di Sensazione di Stein - U.S. Patent 3,774,176
10. **Immagine 10**: Prezzo per Bit nel Tempo - Fonte: Lee, K.H., A Strategic Analysis of the DRAM Industry After the Year 2000
11. **Immagine 11**: Cicli di Memoria nel Tempo - Fonte: Lee, K.H., A Strategic Analysis of the DRAM Industry After the Year 2000
12. **Immagine 12**: Densità DRAM e Prezzo nel Tempo - Fonte: DRAMExchange, SemiAnalysis
13. **Immagine 13**: Struttura del Condensatore DRAM - Fonte: Applied Materials
14. **Immagine 14**: Roadmap DRAM di Samsung - Fonte: Samsung Memcon 2024, SemiEngineering
15. **Immagine 15**: Layout 6F2 vs 4F2 - Fonte: CXMT IEDM 2023
16. **Immagine 16**: Confronto HBM - Fonte: SemiAnalysis
17. **Immagine 17**: Stack HBM e Configurazione - Fonte: Samsung, Micron
18. **Immagine 18**: HBM4 e Roadmap Futura - Fonte: Samsung, Micron
19. **Immagine 19**: Configurazione HBM Personalizzato - Fonte: SK Hynix
20. **Immagine 20**: DRAM "Dumb" - Fonte: SemiAnalysis
21. **Immagine 21**: Potenziale dei Bank DRAM - Fonte: SemiAnalysis
22. **Immagine 22**: Dettagli del Bank DRAM - Fonte: SemiAnalysis, Micron, ISCA 2024
23. **Immagine 23**: Bottleneck della Larghezza di Banda - Fonte: SemiAnalysis
24. **Immagine 24**: Potenziale Completo - Fonte: SemiAnalysis
25. **Immagine 25**: Miglioramenti Potenziali della Larghezza di Banda - Fonte: Tom's Hardware
26. **Immagine 26**: Architettura CIM Futura - Fonte: SemiAnalysis
27. **Immagine 27**: MRAM - Fonte: SK Hynix

---

**Fine della Trascrizione**

*Documento originale: "The Memory Wall: Past, Present, and Future of DRAM" - Dylan Patel, Jeff Koch, Tanj - SemiAnalysis - 3 Settembre 2024*
