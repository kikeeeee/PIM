# PIM-DL: Espandere l'Applicabilità dei DRAM-PIM Commodity per il Deep Learning mediante Co-Ottimizzazione Algoritmo-Sistema

## Informazioni Generali

**Autori**: Cong Li (Peking University), Zhe Zhou (Peking University), Yang Wang (Microsoft Research), Fan Yang (Nankai University), Ting Cao (Microsoft Research), Mao Yang (Microsoft Research), Yun Liang (Peking University), Guangyu Sun (Peking University)

**Conferenza**: ASPLOS '24 (29th ACM International Conference on Architectural Support for Programming Languages and Operating Systems), 27 Aprile - 1 Maggio 2024, La Jolla, CA, USA

**DOI**: https://doi.org/10.1145/3620665.3640376

---

## Abstract e Contributi Principali

Il paper introduce **PIM-DL**, un framework innovativo progettato per espandere l'applicabilità dei DRAM-PIM (Processing-In-Memory) commodity nel contesto del deep learning. Il problema fondamentale affrontato è che i DRAM-PIM attuali, pur essendo commercialmente disponibili (come UPMEM PIM-DIMM, Samsung HBM-PIM, SK-Hynix AiM), hanno capacità computazionali limitate e possono gestire efficientemente solo operatori memory-bound come element-wise e operazioni GEMV (matrix-vector). Questi operatori, tuttavia, rappresentano meno del 15% del tempo totale di esecuzione nei workload DNN (Deep Neural Networks) tipici.

### Contributi Principali:

1. **Framework PIM-DL**: Primo framework completo di deep learning progettato specificamente per DRAM-PIM commodity, basato su un nuovo paradigma di deep learning basato su LUT (Lookup Table).

2. **Algoritmo eLUT-NN (Enhanced LUT-NN)**: Algoritmo migliorato per la calibrazione del modello che sostituisce tutte le operazioni GEMM lineari con lookup table, ottenendo accuratezza più alta con dati di calibrazione 100× minori rispetto al metodo baseline.

3. **Mapping Efficiente e Auto-Tuner**: Progettazione completa del mapping di LUT-NN su DRAM-PIM, modellazione quantitativa del dataflow, e framework di auto-tuning per ottimizzare i parametri di mapping su diverse piattaforme DRAM-PIM.

### Risultati di Performance:

- Confronto con GEMM-based inference su DRAM-PIMs: **22.6×-37.1× speedup**
- Confronto con CPU/GPU baselines: **fino a 3.54×/1.20× speedup**

---

## 1. Introduzione e Motivazione

### Il Problema del "Memory Wall"

I DRAM-PIM sono architetture che incorporano unità di processing vicino ai banchi di memoria per affrontare il problema del "Memory Wall" - la crescente discrepanza tra velocità di accesso alla memoria e capacità computazionale.

Recentemente, vari DRAM-PIM sono entrati in fase commerciale:
- **UPMEM PIM-DIMM**: Basato su DDR4, include core RISC programmabili vicino ai banchi DRAM, offrendo 8× di aumento della bandwidth totale
- **Samsung HBM-PIM**: Basato su HBM2, equipaggiato con unità vettoriali dedicate per le operazioni BLAS, 2 TB/s di bandwidth ma solo 1.2 TFLOP/s per cubo
- **SK-Hynix AiM**: Basato su GDDR6, raggiunge circa 1 TFLOP/s per chip

### Limitazioni Computazionali

La limitazione principale è che i DRAM-PIM implementano le unità di calcolo usando il processo DRAM, il che comporta:
- **Transistor 3× più lenti** rispetto alla CMOS nello stesso nodo tecnologico
- **Densità logica ridotta** di diversi fattori
- **Minore densità di routing** dovuta a meno strati metallici

Di conseguenza:
- Peak computazionale di UPMEM PIM-DIMM: solo 43.8 GOP/s per DIMM
- Le operazioni GEMM (ampiamente usate nei DNN) richiedono solitamente più di 10 TOP/s per garantire sufficiente throughput

I DRAM-PIMs sono quindi **estremamente compute-bound** e applicabili solo a operatori memory-bound.

### Soluzione Proposta: LUT-NN

Per affrontare questa limitazione, il paper propone di sostituire le operazioni GEMM compute-heavy con **Lookup Tables (LUT)**. Questo paradigma, basato sull'algoritmo LUT-NN emergente, consente di ridurre drasticamente le moltiplicazioni necessarie nell'inferenza DNN.

La filosofia di PIM-DL è trasformare il sistema da "compute-centric" (architettura tradizionale con heavy host) a "memory-centric" (offloading della maggior parte degli operatori ai DRAM-PIM, con un host "wimpy" per il resto).

---

## 2. Paradigma di Deep Learning Basato su LUT

### 2.1 Panoramica di LUT-NN

L'idea fondamentale di LUT-NN è che gli elementi di attivazione di matrice di input hanno **similarità semantica block-wise**. Ciò permette di approssimare i valori originali con pochi "centroidi" (valori tipici).

**Procedura Generale**:
1. Convertire le operazioni GEMM tra matrici di input e matrice di peso in moltiplicazioni tra centroidi e matrice di peso
2. Pre-calcolare e memorizzare i "partial-sum" tra centroidi e matrice di peso in lookup table
3. Durante l'inferenza, recuperare e accumulare i dati pre-calcolati dalle LUT in base agli indici dei centroidi più vicini agli input

### 2.2 Conversione LUT-NN

Come mostrato in **Figura 2-(b)**, il processo di conversione trasforma la matrice di peso originale (matrice verde) in lookup table e centroidi:

1. **Step ❶ - Clustering**: Ogni matrice di attivazione M×H viene divisa in sub-vettori 1×V lungo la dimensione H, creando H/V colonne. Per ogni colonna, si genera un codebook contenente CT centroidi (es. CT=4). Ogni centroide è un vettore 1×V ottenuto tramite clustering K-means.

2. **Step ❷ - Inner Product**: La matrice di peso F×H viene anch'essa divisa in sub-vettori 1×V, e si eseguono inner product con i codebook.

3. **Step ❸ - Generazione LUT**: Si derivano CT lookup table, ognuna con forma F × H/V.

**Notazione**:
- N = numero di righe
- H = dimensione della feature di attivazione
- F = numero di feature di output
- V = lunghezza del sub-vettore
- CT = numero di centroidi per codebook
- CB = H/V = numero di codebook

### 2.3 Inferenza LUT-NN

Come mostrato in **Figura 2-(c)**, la procedura di inferenza:

Data una matrice di attivazione di input N×H:

1. **Step ❹ - Calcolo Distanza**: La matrice di input viene divisa in tile 1×V. Ogni tile viene confrontato con il codebook della colonna corrispondente tramite inner-product.

2. **Step ❺ - Ricerca Centroide**: Si determina il centroide con distanza L2 minima (argmin dei risultati di inner-product).

3. **Step ❻ - Accesso Lookup Table**: In base all'indice del centroide migliore, si recupera una colonna di dati dalla lookup table indicizzata.

4. **Step ❼ - Accumulazione**: Per ogni riga della matrice di attivazione, i vettori F×1 cercati vengono accumulati per generare i risultati finali.

5. **Step ❽ - Output Finale**: Dopo il processing di tutte le N righe, si forma la matrice di risultati F×N.

### 2.4 Analisi di Riduzione della Computazione

**Figura 3** mostra l'analisi della riduzione dei FLOP:

Per GEMM tradizionale con input shape N×H e H×F: necessari **2×N×H×F operazioni** (metà moltiplicazioni).

Per LUT-NN con CT centroidi e lunghezza sub-vettore V:
- **3×N×H×CT operazioni** per il calcolo dell'indice
- **N×F×H/V operazioni** per l'accumulazione dei risultati
- Solo **N×H×CT moltiplicazioni** per il calcolo dell'indice

Risultato: LUT-NN riduce significativamente i FLOP (3.66×-18.29×) e le moltiplicazioni rappresentano solo il 2.9%-14.3% delle operazioni totali.

### 2.5 Affinità di LUT-NN con DRAM-PIM

**Figura 4** presenta l'analisi Roofline:

1. **Riduzione Computazionale**: Come mostrato, LUT-NN riduce enormemente la computazione necessaria.

2. **Natura Memory-Intensive**: L'analisi Roofline rivela che tutti gli operatori LUT hanno intensità aritmetica nel range 0.204-0.288, rientrando completamente nella **memoria-bound region** della CPU. Ciò li rende ideali per DRAM-PIMs, che hanno alta bandwidth.

---

## 3. Sfide nell'Adozione di LUT-NN e Soluzioni

### 3.1 Sfida 1: Accuratezza Insoddisfacente (C1)

L'algoritmo LUT-NN originale ha difficoltà a garantire accuratezza soddisfacente durante la conversione.

**Limitazione**: 
- Il lavoro [9] sostituisce solo il GEMM nel layer di classificazione finale
- Il lavoro [84] non riesce a sostituire completamente tutti gli operatori lineari (es. solo 6 su 12 layer in BERT-base)
- Se si prova a sostituire tutti i layer con LUT, l'accuratezza del modello diventa inaccettabile

### 3.2 Sfida 2: Mancanza di Framework (C2)

Non esistono framework di deep learning che supportano DRAM-PIMs commodity come backend (es. UPMEM PIM-DIMM).

**Necessità**: Un framework di serving inference che:
- Supporti sia CPU/GPU che DRAM-PIMs come backend
- Offloadi le operazioni PIM-friendly (soprattutto table lookup) ai PIM
- Gestisca il resto degli operatori sul processore host

### 3.3 Sfida 3: Performance Tuning (C3)

Anche se LUT-NN offre benefici di riduzione computazionale, tradurli in speedup reale su DRAM-PIMs è complesso a causa delle **limitazioni architetturali**:
- Comunicazione host-PIM vincolata
- Nessun datapath diretto inter-PE
- Problemi di load-balancing

---

## 4. Framework PIM-DL

### 4.1 Panoramica Generale

**Figura 5** mostra lo stack software di PIM-DL:

Il framework consiste di tre componenti principali:

1. **LUT-NN Converter**: Converte un modello DNN pre-addestrato in LUT-NN tramite calibrazione congiunta di centroidi e pesi
   - Introduce l'algoritmo **eLUT-NN (Enhanced LUT-NN)**
   - Riesce a sostituire tutti i layer feed-forward mantenendo alta accuratezza

2. **Inference Engine**: Implementa operatori LUT-NN basati su librerie host e PIM
   - Frontend framework con operatori host e PIM
   - Backend library con librerie host e runtime PIM

3. **Auto-Tuner**: Analizza le forme del modello LUT-NN e genera parametri di mapping ottimizzati per la piattaforma target

### 4.2 Algoritmo eLUT-NN

L'algoritmo eLUT-NN introduce due nuove tecniche di calibrazione per correllare gli errori di approssimazione con aggiornamenti parametrici minori:

#### Reconstruction Loss per Approssimazione Computazionale

La loss di ricostruzione accumula gli errori in tutti i layer sostituiti:

$$L = \text{Model Loss} + \beta \sum_{l \in L} ||\hat{A}_l W - A_l W||^2$$

Dove:
- A = matrice di attivazione originale
- Â = matrice di attivazione approssimata (sostituendo sub-vettori con i centroidi più vicini)
- H(·) = funzione di sostituzione centroide più vicino, tale che Â_l = H(A_l)
- β = termine di penalità per bilanciare i due termini di loss

**Benefici**:
1. **Gradient Propagation Diretto**: A differenza del lavoro precedente [84] che aggiorna i centroidi tramite backpropagation layer-by-layer, la reconstruction loss deriva direttamente i gradienti dei centroidi, superando il problema di gradient vanishing.

2. **Convergenza Accelerata**: Introducendo errori di computazione nella funzione di loss, i centroidi imparano rappresentazioni più accurate delle attivazioni.

#### Straight Through Estimator (STE) per Gradient Propagation

Poiché gli operatori di clustering e table-lookup non sono continuamente differenziabili, si utilizza la tecnica **Straight Through Estimator**:

$$\frac{\partial L}{\partial F} = \frac{\partial L}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial \hat{A}} \cdot \frac{\partial \hat{A}}{\partial A} \cdot \frac{\partial A}{\partial F}$$

Con STE: $$\frac{\partial \hat{A}}{\partial A} \approx I$$ (matrice identità)

Questo permette il passaggio dei gradienti e abilita la backpropagation.

**Vantaggi di eLUT-NN rispetto al Baseline**:

1. **A1 - Alta Data Efficiency**: 
   - Baseline [84] richiede il 100% del training set per calibrazione
   - eLUT-NN richiede meno dell'1% del dataset pre-training
   - Convergenza più rapida

2. **A2 - Alta Accuratezza**:
   - Sostituisce tutti i layer feed-forward con LUT mantenendo accuratezza sostanzialmente più alta
   - Il baseline LUT-NN ha degradazione di accuratezza inaccettabile quando applica sostituzione completa

### 4.3 PIM-DL Engine

Come mostrato in **Figura 6-(a)**, il motore PIM-DL comprende:

**Frontend Framework**:
- **Host Operators**: Implementati con librerie tensor ad alte prestazioni su CPU/GPU (MKL, cuBLAS, OneDNN, cuDNN)
- **PIM Operators**: 
  - PIM kernel sul host che attiva i moduli PIM
  - PIM binary sui moduli che descrive il workload offloaded

**Backend Library**:
- Host backend
- PIM runtime e PIM driver
- Comunicazione tramite DRAM-PIMs

**Caso Studio - Transformer** (**Figura 6-(b)**):

Considerato che i Transformer sono l'approccio de-facto in NLP e CV:

- **Layer Convertibili a LUT**: QKV Projection, O (Output) Projection, FFN1, FFN2 (layer lineari)
- **Layer Non Convertibili**: Attention operator (contiene GEMM, eseguito su host)
- **Layer Element-wise**: Add, Norm, GeLU (offloading dipende da supporto PIM)

---

## 5. Mapping Hardware e Ottimizzazione

### 5.1 Astrazione dei DRAM-PIM Commodity

**Figura 7** mostra l'astrazione architetturale:

Un sistema DRAM-PIM è composto da:
- **Host Processor**: CPU, GPU o FPGA
- **PIM Modules**: Collegati ai canali di memoria dell'host, uno per channel
- **Computation Nodes**: Distribuiti in ogni modulo PIM, condividono lo stesso data bus esterno
- **Processing Engine (PE)**: Ogni nodo contiene un PE e banchi di memoria locale
  - PE può essere un core CPU generale o un'unità di calcolo specializzata
  - Può contenere buffer on-chip o registri

**Modello di Esecuzione**: Offloading-based execution

Tre step per guidare i moduli PIM:
1. ❶ Host processor prepara i dati di input e li invia ai moduli PIM
2. ❷ Host lancia il kernel PIM (codice in ISA PIM o sequenza di comandi di memoria specifici)
3. ❸ Dopo che tutti i PE terminano l'esecuzione, l'host recupera i risultati

### 5.2 Limitazioni Architetturali

#### L1: Comunicazione Host-PIM Vincolata

In ogni modulo PIM, i PE condividono un bus di memoria comune:
- Esempio UPMEM: ogni PE accede a un datapath 8-bit; 8 PE in una rank formano un datapath 64-bit
- L'host deve trasferire dati a tutti i PE in una rank simultaneamente per utilizzare pienamente la bandwidth
- Il **broadcasting** da host a PIM raggiunge la più alta bandwidth (evita cache miss al lato host)

#### L2: Nessun Datapath Diretto Inter-PE

A causa della scarsità di risorse di routing on-chip in DRAM-PIM:
- UPMEM PIM-DIMM e HBM-PIM non implementano datapath per comunicazione inter-PE
- I PE devono fare affidamento sull'inoltro host per scambio di dati (load da PE a host cache, poi store a PE destinazione)
- La comunicazione inter-PE può diventare facilmente il collo di bottiglia della performance

#### L3: Problema di Load-Balancing

- I kernel PIM vengono distribuiti ed eseguiti in migliaia di PE
- Il PE più lento determina il tempo di completamento
- È necessario un efficiente load balancing

### 5.3 Dataflow di Inferenza LUT-NN su DRAM-PIMs

**Figura 8-(a)** mostra il mapping degli operatori:

- **Closest Centroid Search (CCS)**: Step ❹-❺ in Figura 2 - Eseguito su **HOST**
  - Calcola la distanza tra matrice di attivazione e centroidi
  - Genera la matrice di indici
  - Non adatto a DRAM-PIM per via della dipendenza da GEMM

- **Table Lookup (LUT)**: Step ❻-❼ in Figura 2 - Offloaded a **PIM**
  - Recupera lookup table e accumula i dati fetchati
  - Adatto a DRAM-PIM

### 5.4 Strategia di Partizione Sub-LUT (Step-1)

**Figura 8-(a)** illustra lo schema di partizione:

La matrice di indice e le LUT vengono divise:
- **Matrice di indice**: Divisa lungo la dimensione di riga N
- **LUT**: Divisa lungo la dimensione di feature F
- **Altre dimensioni**: Non tiling

I PE vengono logicamente distribuiti in **multiple groups**:
- PE nel gruppo i-esimo sono responsabili del calcolo dei risultati del tile di indice i-esimo
- PE j-esimo in ogni gruppo è responsabile della table lookup tra il suo index tile e il j-esimo LUT tile

**Vantaggi di questo Schema**:

1. **Per L1 (Comunicazione Host-PIM Vincolata)**:
   - Riuso di tile migliora la temporal locality
   - La dimensione codebook (CB) non è divisa tra PE, assicurando che i PE calcolino risultati completi di tile di output distinti
   - Evita extra overhead di lettura partial-sum e merging

2. **Per L2 (Nessun Datapath Inter-PE)**:
   - La dimensione centroidi (CT) non è divisa tra PE
   - No comunicazione inter-PE necessaria quando si recuperano LUT in base agli indici

3. **Per L3 (Load-Balancing)**:
   - Tiling uniforme dei tensori assicura che il workload di ogni PE sia identico
   - Poiché tutti i PE eseguono lo stesso micro kernel, il load balancing è garantito

**Modello Analitico**:

Assumendo le latenze di invio input, invio LUT e fetchata output sono t^sub_index, t^sub_lut, t^sub_output rispettivamente, l'overhead della partizione sub-LUT è:

$$t_{sub-lut} = t^{sub}_{index} + t^{sub}_{lut} + t^{sub}_{output}$$

Dove ogni termine può essere stimato usando la dimensione di trasferimento e la bandwidth:

$$t^{sub}_{x} = \frac{STileSize_x \times \#PE}{BW^{host}_x}, \quad x \in \{index, lut, output\}$$

Il vincolo sulla partizione è:

$$\#PE = \frac{N}{N_{s-tile}} \times \frac{F}{F_{s-tile}}$$

### 5.5 Esecuzione Micro Kernel (Step-2)

**Figura 8-(b)** mostra la strategia di tiling del micro kernel:

Dopo la partizione sub-LUT, i tile size in ogni PE sono:
- Index matrix: (N_s-tile, CB)
- Lookup table: (CB, CT, F_s-tile)
- Output result: (N_s-tile, F_s-tile)

Per utilizzare pienamente il buffer on-chip del PE (es. 64KB in UPMEM PIM-DIMM), si fa ulteriore tiling lungo le dimensioni (N_s-tile, F_s-tile, CB) con tiling factor (N_m-tile, F_m-tile, CB_m-tile).

**Procedura di Esecuzione**:

1. Il PE carica un index micro tile (MTile) e il corrispondente output MTile
2. Recupera le LUT per calcolare i risultati
3. Per ogni output MTile, traversa tutti gli index MTile nella stessa N_m-tile per ridurre i dati in tutti i codebook e ottenere risultati completi

**Modello Analitico**:

La latenza del micro kernel è la somma della latenza di trasferimento della memoria e della latenza di riduzione:

$$t_{micro-kernel} = t_{transfer} + t_{reduce}$$

Latenza di trasferimento:

$$t_{transfer} = t^{ld}_{index} + t^{ld}_{lut} + t^{ld}_{output} + t^{st}_{output}$$

Dove la latenza di load/store per singolo tile è:

$$t^{ld}_{x} = \frac{LCount_x \times MTileSize_x}{BW^{pim}_x}$$

$$t^{st}_{x} = \frac{SCount_x \times MTileSize_x}{BW^{pim}_x}$$

Latenza di riduzione:

$$t_{reduce} = RCount \times t_{single-reduce}$$

### 5.6 Spazio di Ricerca e Auto-Tuning

**Figura 9** illustra i diversi schemi di caricamento LUT.

Lo spazio di ricerca dell'auto-tuner include quattro tipi di parametri di mapping:

#### P1: Sub-LUT Tiling Factors
- (N_s-tile, F_s-tile)
- Affettano non solo la dimensione dei tile assegnati a ogni PE, ma anche il pattern di comunicazione di ogni tensore
- Permette di sfruttare diversi trade-off

#### P2: Micro Kernel Tiling Factors
- (N_m-tile, F_m-tile, CB_m-tile)
- Permettono di regolare l'allocazione del buffer tra MTile di indice e output

#### P3: Tile Traversal Order
- Permutare l'ordine di traversal dei MTile factors
- Cambia il pattern di riuso dei tile
- Offre opportunità di sfruttare pattern di riuso diversi

#### P4: LUT Load Scheme
Tre schemi di caricamento LUT (illustrati in Figura 9):

1. **Static Load**: Se il LUT MTile size in ogni PE è inferiore al buffer on-chip, tutto il LUT viene caricato staticamente on-chip
   - Necessario memorizzare CB_s-tile × CT × F_s-tile elementi LUT
   - Il LUT viene caricato una volta e riutilizzato
   - Buffer allocation: CB_s-tile × CT × F_s-tile

2. **Coarse-grain Load**: Dal momento che ogni indice seleziona l'elemento target da ogni CT candidati, si caricano questi CT elementi on-chip
   - Buffer size per caricamento: CB_load-tile × CT × F_load-tile
   - Gli elementi rimangono bufferizzati finché i codebook corrispondenti non sono stati ridotti

3. **Fine-grain Load**: Caricamento on-demand degli elementi LUT
   - Buffer size per slot parallelo: F_load-tile elementi LUT
   - Particolarmente utile se il PE può emettere multiple read request in parallelo (es. UPMEM con hardware threads multipli)

### 5.7 Auto-Tuner Workflow

**Algoritmo 1** descrive il workflow di auto-tuning:

```
Input: Workload Size (N, CB, CT, F)
Cost = MaxValue
MappingParams = {}

for each legal (N_s-tile, F_s-tile) do
    // Stima overhead della partizione sub-LUT
    t_sub-lut = t^sub_index + t^sub_lut + t^sub_output
    
    // Ricerca il micro-kernel ottimale
    t*_micro-kernel, Kernel* = KernelSearch(N_s-tile, F_s-tile, CB, CT)
    
    // Aggiorna i parametri ottimali
    if t_sub-lut + t*_micro-kernel < Cost then
        Cost = t_sub-lut + t*_micro-kernel
        MappingParams = {N_s-tile, F_s-tile, Kernel*}

return Cost, MappingParams
```

**Nota**: Per un dato modello, l'auto-tuner cerca i parametri ottimali di tutti i kernel LUT offline, con overhead minuscolo (~1s/modello su dual-socket Intel Xeon 4210 CPUs) rispetto alla latenza di inferenza del modello (decine di secondi/modello su piattaforma UPMEM).

---

## 6. Valutazione Sperimentale

### 6.1 Setup Sperimentale

#### Modelli Evaluati:

**Per Validazione di Accuratezza**:
- **NLP**: BERT-base e BERT-large su benchmark GLUE
- **CV**: ViT-base e ViT-huge su dataset CIFAR-10 e CIFAR-100

**Per Confronto Throughput**:
- BERT-base (768 hidden dim)
- BERT-large (1024 hidden dim)
- ViT-huge (1280 hidden dim)

#### Piattaforme Testate:

**Tabella 3** fornisce i dettagli di configurazione:

| **Piattaforma** | **Host** | **DRAM-PIM** | **PE Count** | **Memoria** |
|---|---|---|---|---|
| **DDR4-PIM** | Xeon 4210 CPU × 2 | UPMEM PIM-DIMM × 8 | 1024 | 64 GB DDR4 |
| **HBM-PIM (Sim)** | NVIDIA A2 GPU × 1 | Samsung HBM-PIM Cube × 4 | 512 | 8 GB HBM2 |
| **AiM (Sim)** | NVIDIA A2 GPU × 1 | SK-Hynix AiM Chip × 16 | 512 | 16 GB GDDR6 |

**Note**:
- Esperimenti principali su piattaforma UPMEM reale
- Samsung HBM-PIM e SK-Hynix AiM valutati tramite simulazione (non disponibili yet)
- Operatori host: C++/OpenMP, GGML per CPU
- Operatori PIM: UPMEM SDK (Version 2021.3.0)

#### Baselines:

- **eLUT-NN vs Baseline LUT-NN**: Confronto di accuratezza
- **Inferenza su CPU**: GGML-based transformer su CPU server (dual-socket Intel Xeon Gold 5218, FP32/INT8)
- **Inferenza su GPU**: DGX-1 workstation con NVIDIA V100 GPU (FP32)
- **GEMM-based PIM**: Offload di tutti i linear layer a DRAM-PIM con inferenza standard

### 6.2 Accuratezza del Modello

**Tabella 4 - Accuratezza NLP**:

| **Modello** | **Setting** | **MNLI** | **QQP** | **QNLI** | **SST-2** | **CoLA** | **STS-B** | **MRPC** | **RTE** | **Avg** |
|---|---|---|---|---|---|---|---|---|---|---|
| **BERT-base** | Original | 83.4 | 71.2 | 90.5 | 93.5 | 52.1 | 85.8 | 88.9 | 66.4 | **79.0** |
| | LUT-NN | 35.5 | 63.2 | 50.6 | 49.3 | 0.0 | 1.36 | 31.6 | 52.7 | **35.5** |
| | **eLUT-NN** | **79.9** | **69.6** | **87.4** | **92.4** | **51.2** | **83.2** | **87.1** | **64.7** | **76.9** |
| **BERT-large** | Original | 85.9 | 72.1 | 92.7 | 94.9 | 60.5 | 86.5 | 89.3 | 70.1 | **81.5** |
| | LUT-NN | 34.7 | 62.7 | 51.3 | 52.2 | 0.0 | 4.40 | 38.7 | 50.5 | **36.8** |
| | **eLUT-NN** | **82.1** | **71.0** | **90.2** | **93.1** | **56.8** | **86.7** | **86.1** | **68.4** | **79.3** |

**Analisi**:
- LUT-NN baseline ha degradazione catastrofica quando applica sostituzione completa (drop media 90.44% su NLP)
- eLUT-NN recupera notevolmente l'accuratezza (miglioramento medio 88.09% su NLP)
- Drop di accuratezza di eLUT-NN rispetto all'originale: ~2.25% media su NLP

**Tabella 5 - Accuratezza Vision**:

| **Modello** | **Setting** | **CIFAR-10** | **CIFAR-100** |
|---|---|---|---|
| **ViT-base** | Original | 98.5 | 91.4 |
| | LUT-NN | 10.1 | 1.07 |
| | **eLUT-NN** | **96.3** | **89.1** |
| **ViT-huge** | Original | 99.5 | 94.55 |
| | LUT-NN | 10.0 | 1.01 |
| | **eLUT-NN** | **97.8** | **91.32** |

**Analisi**:
- LUT-NN baseline ha accuratezza praticamente inutilizzabile su CV (drop media 44.10%)
- eLUT-NN raggiunge drop di soli ~2.36% rispetto all'originale su CV
- Convergenza: i modelli eLUT-NN raggiungono convergenza dopo non più di 100k iterazioni
- Data Efficiency: eLUT-NN consuma solo ~0.78% del dataset di training per calibrazione, vs 100% per il baseline

### 6.3 Performance End-to-End

#### Throughput

**Figura 10-(a)** mostra il confronto di performance:

**Configurazione Test**:
- BERT-base/large: sequence length 512, batch size 64
- ViT-huge: input image 224×224×3, batch size 128, patch size 14×14
- Quantizzazione INT8 sui LUT (drop di accuratezza ≤0.1%)
- PIM-DL: CT=16, V=2 o V=4

**Risultati di Speedup**:

Confronto con **CPU Server (FP32/INT8)**:
- V=2/CT=16: **2.05×/1.14× geomean speedup**
- V=4/CT=16: **3.07×/1.71× geomean speedup**

Confronto con **GEMM-based su PIM**:
- V=2/CT=16: **12.61× geomean speedup**
- V=4/CT=16: **18.91× geomean speedup**

#### Energy Efficiency

**Figura 10-(b)** mostra il consumo energetico normalizzato a FP32 CPU baseline:

**Metodologia di Misura**:
- Intel RAPL per misurare energia CPU
- Tool dpu-diag di UPMEM SDK per stima energia PIM-DIMM (~13.92W/DIMM@350MHz, power statico di core e banchi PIM)
- Energia accesso memoria CPU-PIM tramite MSR_DRAM_ENERGY_STATUS di Intel RAPL

**Risultati di Efficienza Energetica**:

Confronto con CPU baseline (FP32/INT8):
- V=2/CT=16: **2.95×/1.65× higher energy efficiency (geomean)**
- V=4/CT=16: **4.42×/2.46× higher energy efficiency (geomean)**

Confronto con inferenza GEMM su PIM:
- V=2/CT=16: **11.16× higher energy efficiency (geomean)**
- V=4/CT=16: **16.74× higher energy efficiency (geomean)**

### 6.4 Analisi di Performance

#### Latency Breakdown

**Figura 11-(a)** mostra la scomposizione della latenza:

- **Inferenza LUT-NN**: 73.73% ~ 79.39% della latenza totale
  - Operatore LUT: 69.88% ~ 76.10% della latenza LUT-NN (51.52% ~ 60.41% della latenza totale)
  - Operatore CCS (Closest Centroid Search): eseguito su host
  - Altri operatori: element-wise, normalization, etc.

Questo dimostra che PIM-DL riesce a offloadare una porzione molto più grande dei workload di deep learning rispetto ai sistemi PIM-enabled esistenti.

#### Confronto Layer-wise

**Figura 11-(b)** analizza lo speedup di ogni layer lineare:

Confronto tra inferenza LUT-NN (V=4/CT=16) e inferenza GEMM-based INT8 su CPU:

| **Layer** | **Speedup Geomean** |
|---|---|
| **QKV Projection** | 1.61× |
| **O (Output) Projection** | 0.99× |
| **FFN1** | 1.78× |
| **FFN2** | 2.38× |
| **Totale** | **1.81×** |

**Osservazioni**:
- FFN2 guadagna il miglioramento più alto (layer più grande con largest inner dim in GEMM)
- QKV e FFN1 guadagnano benefici significativi (large output feature dims)
- O projection mantiene performance comparabile anche con layer più piccoli

### 6.5 Analisi di Sensibilità

**Figura 12** mostra l'analisi di sensibilità variando quattro parametri (normalizzati a CPU INT8 baseline):

#### Sub-vector Length (V)

**Figura 12-(a)**:
- Sub-vector length maggiore → performance migliore (più grande V riduce il numero di codebook, shrinking LUT size)
- Miglioramento tende a convergere perché la bandwidth di UPMEM decresce quando transfer size shrinks

#### Centroid Number (CT)

**Figura 12-(b)**:
- Centroid number minore → performance migliore (più piccolo CT riduce memory footprint delle LUT)
- Anch'esso mostra lieve tendenza di convergenza

#### Batch Size

**Figura 12-(c)**:
- Batch size piccolo: CPU server supera PIM-DL (comunicazione host-PIM diventa collo di bottiglia con kernel piccoli)
- Batch size grande: PIM-DL mostra vantaggi significativi
- Questo è dovuto alla scarsa larghezza di banda host-PIM

#### Hidden Dimension

**Figura 12-(d)**:
- Vari hidden dimension comuni: 1024, 2048, 2560, 4096, 5120
- PIM-DL raggiunge **2.44× geomean speedup** su CPU
- A hidden dim 4096, PIM-DL guadagna performance molto migliore (CPU server ha scalabilità peggiore)

### 6.6 Visualizzazione dello Spazio di Mapping

**Figura 13** visualizza lo spazio di mapping di LUT-NN su UPMEM PIM-DIMM (case study: BERT-large FFN1 layer):

**Workload shape**: (N, CB, CT, F) = (32768, 256, 16, 4096)
**Sub-LUT tiling**: (N_s-tile, F_s-tile) = (16384, 8) per static scheme, (512, 256) per altri scheme

#### Sub-LUT Tiling Factors

**Figura 13-(d)**:
- Gap di performance fino a **1.91×** cambiando (N_s-tile, F_s-tile)
- Quando N_s-tile o F_s-tile è grande, lo skew della dimensione del tile porta a overhead di comunicazione host-PIM più alto

#### Micro Kernel Tile Size

- **Coarse-grain/Fine-grain LUT load**: Gap di performance medio **1.04×**
- **Static LUT load**: Gap fino a **1.74×**
  - Sotto static scheme, F_m-tile è limitato da F_s-tile
  - La bandwidth DRAM del PE non satura, aumentando rapidamente sotto F_m-tile bassa

#### Tile Traversal Order

- Gap di performance minimo dal cambio di traversal order
- Ciò è dovuto alla scarsa capacità computazionale dei PE in UPMEM: latenza di accumulazione domina latenza totale del micro kernel, diminuendo il vantaggio del data reuse

#### LUT Load Scheme

**Figura 13-(a)-(b)**:
- Gap considerevole dal cambio di load tile size
- Poiché gli offset dei tile on-chip sono computati dal PE, e la bandwidth del buffer on-chip è correlata al numero di istruzioni, è necessario impostare load tile size moderato per utilizzare pienamente la bandwidth

#### Analisi Auto-Tuner

- **Degradazione di Performance**: Parametri forniti da PIM-DL Auto-Tuner hanno ≤6% performance degradation rispetto all'ottimo
- **Errore di Stima**: Error medio 3.44%, error massimo 13.73%
- L'auto-tuner riesce a trovare automaticamente parametri near-optimal per workload dati

### 6.7 PIM-DL su HBM-PIM e AiM

Valutazione tramite simulazione (producti non ancora disponibili):

**Setup**: Sequence length 128, batch size 1-8, hidden dimensions variabili

#### Confronto con Inferenza GEMM-based su PIM

**Figura 14**:
- **HBM-PIM**: **23.94× geomean speedup**
- **AiM**: **19.06× geomean speedup**

Quando batch size aumenta, speedup di PIM-DL aumenta fino a **2.23×** (batch size maggiore è unfavorable per questi prodotti). Quando hidden dim aumenta, speedup di PIM-DL shrinks leggermente (dataflow di HBM-PIM e AiM è optimizzato per matrici con shape piatte).

#### Confronto con GPU (NVIDIA V100)

**Figura 15**:
- **AiM-based PIM-DL**: **fino a 1.20× speedup** vs NVIDIA V100 (geomean)
- **HBM-PIM-based PIM-DL**: Solo **39% di V100 performance** (geomean)

**Spiegazione**: 
- Gap di capacità computazionale enorme tra V100 GPU (130 TFLOPS) e HBM-PIM (4.8 TFLOPS)
- Per AiM, capacità computazionale molto più alta che HBM-PIM (16 TFLOPS), permette performance comparabile a V100

---

## 7. Discussione e Futuri Lavori

Il paper identifica alcuni implicazioni architettoniche che potrebbero migliorare ulteriormente la performance di PIM-DL:

### Adder-Only PIM Design

Dato che LUT-NN rimuove tutte le moltiplicazioni negli operatori LUT lato PIM, si potrebbe equipaggiare i PE con soli adder invece di adder+multiplier:
- Gli adder hanno overhead hardware molto minore rispetto ai moltiplicatori
- Sotto gli stessi constraint di area/power, si potrebbero equipaggiare molti più adder
- PIM-DL su DRAM-PIM adder-only raggiungerebbe performance significativamente più alta

### On-chip Buffer Management Support

Il buffer on-chip del PE non riesce a sfruttare il data reuse a causa dell'overhead severo nell'implementare un meccanismo di caching.
- Attualmente si usano solo 3 semplici schemi di caricamento LUT
- Con migliore supporto di on-chip buffer management, si potrebbe sfruttare il data reuse (soprattutto se LUT access distribution ha "hot" items)
- Questo porterebbe migliore performance a PIM-DL

---

## 8. Lavori Correlati

Il paper cita estesamente i lavori correlati su:

### DRAM-PIMs Accademici

Proposte di DRAM-PIM dai laboratori di ricerca per una decade, usati per:
- Graph processing (GraphP, GraphQ, etc.)
- Machine learning
- General-purpose applications

**Categorie**:
1. **Die-stacking memory DRAM-PIMs** (es. Hybrid Memory Cube - HMC)
2. **DIMM-based DRAM-PIMs** (es. TensorDIMM, RecNMP, DIMM-Link)

### DRAM-PIMs Commerciali

- UPMEM PIM-DIMM
- Samsung HBM-PIM
- SK-Hynix AiM

### Applicazioni su Commodity DRAM-PIMs

Vari lavori hanno customizzato applicazioni real-world su commodity DRAM-PIMs, ma **nessuno ha potuto processare efficientemente DNN mainstream** come Transformer.

### LUT-based Approaches

Lavori precedenti hanno implementato operazioni basate su LUT nei circuiti DRAM, ma:
- Diversi da PIM-DL (che adotta LUT a livello algoritmo)
- Non direttamente utilizzabili per accelerare GEMM
- TransPimLib implementa funzioni transcendentali basate su LUT su UPMEM, ma non può essere usato per accelerare GEMM

**Differenziazione di PIM-DL**:
- Primo framework di deep learning full-stack per commodity DRAM-PIMs
- Introduce innovazioni algoritmiche (eLUT-NN) per mantenere accuratezza quando si sostituisce GEMM con LUT-NN
- Contiene strategie efficienti di mapping e auto-tuning per boost della performance dell'inferenza

---

## 9. Conclusioni

**PIM-DL** è il primo framework completo progettato per espandere l'applicabilità dei DRAM-PIM commodity nel deep learning. Il framework affrontail tre sfide fondamentali:

1. **C1 (Accuratezza)**: Tramite algoritmo eLUT-NN che sostituisce tutti i layer feed-forward con LUT mantenendo alta accuratezza (drop solo ~2.25% su NLP, ~2.36% su CV)

2. **C2 (Framework)**: Tramite PIM-DL Engine che implementa operatori LUT-NN su backend host e PIM

3. **C3 (Performance Tuning)**: Tramite Auto-Tuner che ottimizza automaticamente i parametri di mapping hardware

### Risultati di Performance Sintetici:

- **vs GEMM-based inference su DRAM-PIMs**: 22.6×-37.1× speedup
- **vs CPU inference**: fino a 3.54× speedup
- **vs GPU inference**: fino a 1.20× speedup (su AiM)

### Software

PIM-DL è open-sourced a: https://github.com/leesou/PIM-DL-ASPLOS

---

## Tabelle e Figure Riassuntive

### Tabella Commodity DRAM-PIMs Confronto

La **Tabella 1** nel paper confronta tre prodotti DRAM-PIM commodity:

| **Prodotto** | **Tecnologia** | **PIM Units** | **Peak Bandwidth** | **Peak Throughput** |
|---|---|---|---|---|
| **PIM-DIMM** | DDR4 | RISC Cores | 80.4 GB/s per DIMM | 43.8 GOP/s per DIMM |
| **HBM-PIM** | HBM2 | FP16 MAC | 2 TB/s per cube | 1.2 TFLOPS |
| **AiM** | GDDR6 | BF16 MAC | 1 TB/s per chip | 1 TFLOPS |

### Notazioni Importanti (Tabella 2)

Il paper usa estensamente le seguenti notazioni per l'auto-tuner:

| **Notazione** | **Descrizione** |
|---|---|
| N | Row count dell'indice di input |
| CB | Numero di codebook (H/V) |
| CT | Numero di centroidi |
| F | Lunghezza della feature di output |
| X_s-tile | Tiling factor nella partizione sub-LUT |
| X_m-tile | Tiling factor nel micro kernel |
| X_load-tile | Load factor negli schemi non-static |
| BW_host^x | Bandwidth host-PIM |
| BW_pim^x | Bandwidth della memoria locale |
| #PE | Numero di PE usati |

---

## Note Finali per Studio

Questo riassunto copre completamente il contenuto del paper PIM-DL, mantenendo tutti gli argomenti principali senza eccessiva sintesi. Sono inclusi:

✓ **Introduzione e Motivazione**: Problem statement, limitazioni DRAM-PIMs
✓ **Background Tecnico**: Paradigma LUT-NN, Conversion/Inference, Analisi affinity
✓ **Sfide e Soluzioni**: C1/C2/C3 e come PIM-DL le affronta
✓ **Algoritmo eLUT-NN**: Reconstruction Loss, Straight Through Estimator
✓ **Framework PIM-DL**: Converter, Engine, Auto-Tuner
✓ **Hardware Mapping**: Abstrazione architettonica, limitazioni, dataflow a 2 step
✓ **Ottimizzazione**: Spazio di ricerca, schemi di caricamento LUT, workflow auto-tuning
✓ **Valutazione Completa**: Setup, accuratezza, performance, energy efficiency, analisi di sensibilità
✓ **Risultati su Diverse Piattaforme**: UPMEM (reale), HBM-PIM, AiM (simulati)
✓ **Discussione e Future Work**: Implicazioni architettoniche

Il documento è pronto per lo studio approfondito del framework PIM-DL e della sua innovativa approccio al deep learning su DRAM-PIMs commodity.