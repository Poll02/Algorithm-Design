# Appunti su Network Flow: Teoria e Applicazioni (Versione Completa)

## 1. Definizioni Fondamentali
Una **rete di flusso** è un grafo orientato $G = (V, E)$ in cui:
* **Capacità $c(e)$**: ogni arco $e = (u, v)$ ha una capacità massima non negativa $c(e) \ge 0$.
* **Sorgente e Pozzo**: esistono due nodi speciali, la sorgente $s$ (senza archi entranti) e il pozzo $t$ (senza archi uscenti).

Un **Flusso Ammissibile (s-t flow)** è una funzione $f: E 	o \mathbb{R}_{\ge 0}$ che rispetta due vincoli:
1. **Vincolo di Capacità:** $0 \le f(e) \le c(e)$ per ogni arco $e \in E$.
2. **Conservazione del Flusso:** per ogni nodo $v \in V \setminus \{s, t\}$, la somma dei flussi entranti è uguale alla somma dei flussi uscenti:
   $$\sum_{(u, v) \in E} f(u, v) = \sum_{(v, w) \in E} f(v, w)$$

**Valore del flusso $v(f)$**: la quantità netta totale di flusso che esce dalla sorgente $s$:
$$v(f) = \sum_{(s, u) \in E} f(s, u)$$

---

## 2. Il Problema del Max-Flow e del Min-Cut
* **Max-Flow Problem:** trovare un flusso ammissibile $f$ che massimizzi il valore $v(f)$.
* **s-t Cut (Taglio):** una partizione dei nodi in due insiemi disgiunti $(A, B)$ tale che $s \in A$ e $t \in B$.
* **Capacità del Taglio $c(A, B)$**: somma delle capacità degli archi che attraversano il taglio da $A$ verso $B$:
  $$c(A, B) = \sum_{u \in A, \, v \in B} c(u, v)$$
* **Min-Cut Problem:** trovare un s-t cut di capacità minima.

---

## 3. L'Algoritmo di Ford-Fulkerson
L'approccio greedy semplice fallisce (es. rete a "diamante") perché una scelta avida può saturare un arco vitale inibendo cammini migliori, e non può tornare indietro. Ford-Fulkerson supera questo limite introducendo il **grafo residuo $G_f$**.

### Grafo Residuo $G_f$ e Backward Edges
Per ogni arco originario $(u, v)$ con capacità $c(u, v)$ e flusso $f(u, v)$:
* **Arco in avanti (Forward Edge):** se $f(u, v) < c(u, v)$, c'è spazio per altro flusso. In $G_f$ compare $(u, v)$ con capacità residua $c_f(u, v) = c(u, v) - f(u, v)$.
* **Arco all'indietro (Backward Edge):** se $f(u, v) > 0$, l'algoritmo permette di "redirezionare" questo flusso. In $G_f$ compare l'arco $(v, u)$ con capacità $c_f(v, u) = f(u, v)$.

### Algoritmo
1. Inizializza il flusso $f(e) = 0$ per ogni arco $e$.
2. Cerca un **cammino aumentante (s-t path)** in $G_f$.
3. Se trovato, determina il "bottleneck" (la capacità minima degli archi sul cammino) e incrementa il flusso globale di tale quantità.
4. Aggiorna $G_f$ e ripeti dal passo 2. Termina quando non ci sono più cammini.
* **Complessità:** $O(m \cdot C)$ dove $m = |E|$ e $C$ è il valore del flusso massimo.

---

## 4. Miglioramenti di Ford-Fulkerson
Per evitare casi pessimistici dove il bottleneck è sistematicamente piccolo, si usano euristiche di selezione del cammino:

1. **Capacity Scaling (Grafo $\Delta$-Residuo)**
   * Definisce una soglia $\Delta$ (inizialmente la max capacità uscente da $s$). Cerca cammini usando solo archi con capacità $\ge \Delta$.
   * Quando non ci sono più cammini, dimezza $\Delta = \Delta / 2$. Termina dopo la fase $\Delta = 1$.
   * **Complessità:** $O(m^2 \log C)$. È un algoritmo *debolmente polinomiale*.
2. **Edmonds-Karp (Shortest Path Augmenting)**
   * Utilizza la **BFS** per trovare sempre il cammino aumentante col minor numero di archi (più "corto" a livello topologico).
   * **Complessità:** $O(n \cdot m^2)$. È un algoritmo *fortemente polinomiale*.

---

## 5. Teorema Max-Flow Min-Cut
**Enunciato:** Il valore del flusso massimo $v(f)$ è esattamente uguale alla capacità del taglio minimo $c(A, B)$.
$$\max v(f) = \min c(A, B)$$

**Dimostrazione (Terminazione di Ford-Fulkerson $\implies$ Min-Cut):**
Quando l'algoritmo termina, non esiste alcun s-t path in $G_f$. 
1. Costruiamo l'insieme $A$ contenente tutti i nodi raggiungibili da $s$ in $G_f$, e $B$ contenente i nodi rimanenti. Poiché non c'è percorso, $s \in A$ e $t \in B$. Questo è un taglio valido.
2. Analizziamo gli archi originari al confine tra $A$ e $B$:
   * Ogni arco in avanti $(u, v)$ da $A$ a $B$ deve essere saturo ($f(u, v) = c(u, v)$). Se non lo fosse, l'arco esisterebbe in $G_f$ e $v$ sarebbe in $A$.
   * Ogni arco all'indietro $(v, u)$ con $v \in B$ e $u \in A$ deve avere flusso nullo ($f(v, u) = 0$). Se non lo fosse, esisterebbe un arco backward in $G_f$ verso $v$, portandolo in $A$.
3. Per la conservazione del flusso, il flusso netto dal taglio $(A, B)$ è:
   $$v(f) = \sum_{u \in A, \, v \in B} c(u, v) - 0 = c(A, B)$$
Questo dimostra che il flusso è massimo e il taglio è minimo.

---

## 6. Applicazioni di Base

### Bipartite Matching
* **Problema:** In un grafo diviso in candidati $L$ e lavori $R$, trovare il massimo numero di abbinamenti 1-a-1.
* **Riduzione:** Si aggiungono $s$ e $t$. Si collegano $s$ a tutti i nodi $L$, e tutti i nodi $R$ a $t$, con capacità unitaria ($1$). Si orientano gli archi tra $L$ ed $R$ imponendo capacità $1$.
* **Correttezza:** Grazie al *Teorema dell'Integralità*, il flusso massimo trovato sarà composto solo da interi (0 o 1). I flussi pari a 1 sugli archi $L 	o R$ formano un abbinamento perfetto senza sovrapposizioni.
* **Complessità:** $O(m \cdot n)$.

### Cammini Arco-Disgiunti (Edge-Disjoint Paths)
* **Problema:** Trovare il massimo numero di percorsi da $s$ a $t$ che non condividono nessun arco.
* **Riduzione:** Si assegna a tutti gli archi una capacità $1$ e si lancia il Max-Flow.
* **Correttezza:** Il flusso in uscita traccerà $v(f)$ percorsi. La capacità 1 impedisce accavallamenti.
* **Teorema di Menger:** Il numero max di cammini arco-disgiunti equivale al numero min di archi da rimuovere per disconnettere $s$ da $t$ (diretta applicazione del Min-Cut).

---

## 7. Circolazione e Varianti (Estensioni del Modello)

### Circulation with Demands (Circolazione con Domande)
* **Problema:** Scompare il concetto di un'unica sorgente e un unico pozzo. Ogni nodo $v$ ha una domanda $d(v)$.
  * Se $d(v) > 0$: il nodo è un pozzo locale (richiede flusso).
  * Se $d(v) < 0$: il nodo è una sorgente locale (produce flusso).
  * Se $d(v) = 0$: nodo di transito.
* **Condizione Necessaria:** La rete è in equilibrio solo se $\sum d(v) = 0$.
* **Riduzione a Max-Flow:** 
  1. Aggiungiamo una super-sorgente $s^*$ e un super-pozzo $t^*$.
  2. Colleghiamo $s^*$ a ogni nodo con $d(v) < 0$ tramite un arco di capacità $-d(v)$.
  3. Colleghiamo ogni nodo con $d(v) > 0$ a $t^*$ con capacità $d(v)$.
* **Correttezza:** Esiste una circolazione ammissibile se e solo se il calcolo del Max-Flow su $s^*-t^*$ satura *tutti* gli archi uscenti da $s^*$ (ovvero $v(f) = \sum_{v: d(v)>0} d(v)$).

### Circulation with Lower Bounds (Limiti Inferiori)
* **Problema:** Oltre alla capacità massima $c(e)$, ogni arco ha un vincolo minimo di flusso $l(e)$. Il flusso deve rispettare $l(e) \le f(e) \le c(e)$.
* **Riduzione a Circolazione Standard:**
  1. Forziamo il passaggio del minimo garantito: poniamo su ogni arco un flusso base $l(e)$. Questo violerà temporaneamente la conservazione del flusso nei nodi.
  2. Definiamo un "flusso residuo" che dobbiamo ancora instradare: $f'(e) = f(e) - l(e)$.
  3. Aggiorniamo le capacità: la nuova capacità a disposizione è $c'(e) = c(e) - l(e)$.
  4. Correggiamo le domande dei nodi per compensare il flusso base forzato: 
     $$d'(v) = d(v) + L_{in}(v) - L_{out}(v)$$
     *(dove $L_{in}$ e $L_{out}$ sono le somme dei lower bounds entranti e uscenti dal nodo $v$).*
* **Correttezza:** Si risolve il problema di Circolazione con Domande (usando $d'$ e $c'$). Se si trova una soluzione $f'$, il flusso reale ammissibile sarà $f(e) = f'(e) + l(e)$.

---

## 8. Applicazioni Avanzate

### Survey Design (Progettazione di Sondaggi)
* **Problema:** Vogliamo assegnare dei prodotti ai consumatori affinché li recensiscano.
  * Il consumatore $i$ può fare tra $c_i$ e $c'_i$ recensioni.
  * Il prodotto $j$ deve ricevere tra $p_j$ e $p'_j$ recensioni.
  * Un consumatore recensisce un prodotto solo se lo ha acquistato.
* **Riduzione:** Si modella come Circolazione con Lower Bounds.
  * Nodi: consumatori e prodotti. Super-sorgente $s$, super-pozzo $t$.
  * Arco da $s$ al consumatore $i$ con capacità $[c_i, c'_i]$.
  * Arco dal consumatore $i$ al prodotto $j$ (se acquistato) con capacità $[0, 1]$.
  * Arco dal prodotto $j$ a $t$ con capacità $[p_j, p'_j]$.
  * Arco di ritorno "infinito" da $t$ a $s$ per chiudere il ciclo e formare una circolazione.

### Airline Scheduling (Pianificazione dei Voli)
* **Problema:** Dato un elenco di voli (con aeroporti e orari di partenza/arrivo), assegnarli a degli aerei in modo che ogni volo venga coperto e gli aerei abbiano il tempo materiale di spostarsi da un arrivo alla partenza successiva.
* **Riduzione a Circolazione:** 
  * I voli diventano archi tra i nodi "partenza" e "arrivo" con un lower bound $l=1$ (ogni volo deve essere operato almeno/esattamente una volta).
  * Si inseriscono archi di "collegamento" $[0, 1]$ tra l'arrivo di un volo e la partenza di un altro se gli orari sono compatibili.
  * Aggiungendo un arco di ritorno da $t$ a $s$ e imponendo limiti sulla sua capacità, si può anche minimizzare o vincolare il numero totale di aerei impiegati dalla compagnia.

### Project Selection (Selezione di Progetti / Max-Weight Closure)
* **Problema:** Abbiamo un insieme di progetti. Ogni progetto $v$ ha un profitto netto $p_v$ (che può essere $>0$ se genera guadagno, o $<0$ se è un costo di investimento). I progetti hanno delle **dipendenze**: se decido di avviare il progetto A, sono obbligato ad avviare anche il progetto B (costruendo ad es. un'infrastruttura). Vogliamo scegliere il set di progetti che massimizza il profitto.
* **Riduzione a Min-Cut:**
  1. Creiamo una rete con $s$ (super-sorgente) e $t$ (super-pozzo).
  2. **Progetti in attivo ($p_v > 0$):** arco da $s$ al progetto $v$ con capacità $p_v$.
  3. **Progetti in perdita ($p_v < 0$):** arco dal progetto $v$ a $t$ con capacità $-p_v$ (costo positivo).
  4. **Dipendenze:** se A richiede B, inseriamo un arco da A a B con **capacità infinita** $\infty$ (questo impedisce fisicamente che un Min-Cut possa tagliare A separandolo da B senza pagare un costo infinito, garantendo la coerenza logica).
* **Soluzione:** Calcoliamo il Min-Cut tra $s$ e $t$. I progetti che terminano nel lato del taglio della sorgente $s$ costituiscono il nostro set ottimale da avviare.
  * **Profitto Max:** (Somma di tutti i profitti positivi) - (Capacità del Min-Cut).
