# Algorithm Design - Mock Exam 4

Questo mock exam è modellato sulle prove d'esame ufficiali dei proff. Anagnostopoulos, Leonardi e Fioravanti. Presenta 4 esercizi inediti con soluzioni complete, dimostrazioni formali e pseudocodici dettagliati.

---

## Problem 1: Autonomous Rover Fleet Evacuation (Network Flow with Node Capacities)

### Problem Statement
A team of $k$ autonomous exploration rovers is operating inside an $N \times M$ grid-shaped hazardous terrain. 
* Each rover $r \in \{1, \dots, k\}$ is currently located at a distinct initial grid cell $(x_r, y_r)$.
* The rovers must evacuate to the safety boundary of the grid (i.e., any cell lying on the outer perimeter of the $N \times M$ grid).
* Rovers can move between adjacent grid cells (up, down, left, right).
* **Collision and Interference Constraint:** To prevent physical collisions and avoid mutual sensor jamming, **no two rovers are allowed to pass through the same grid cell at any point** during the evacuation (meaning that the paths taken by the rovers from their starting cells to the boundary must be completely **vertex-disjoint**).

1. Formulate the problem of finding a simultaneous evacuation plan for all $k$ rovers as a **Maximum Flow** problem. In particular, explain how to handle cell/vertex capacities using the **node-splitting technique**, specifying the network's vertices, directed edges, and capacities.
2. State the condition on the maximum flow value that determines if a valid evacuation plan exists, and prove the correctness of the reduction in both directions ($\implies$ and $\impliedby$).
3. Analyze the computational complexity of the procedure.

---

### Solution

#### 1. Modellazione della Rete di Flusso con Node-Splitting

Il vincolo fondamentale impone che ciascuna cella della griglia possa essere attraversata da **al più un solo rover** (cammini disgiunti sui vertici). 
Nelle reti di flusso standard le capacità sono definite sugli archi e non sui nodi. Utilizziamo quindi la tecnica canonica del **node-splitting (sdoppiamento dei nodi)**:

* **Sdoppiamento di ciascuna cella $u$:**
  Per ogni cella $u = (x, y)$ della griglia $N \times M$, creiamo due nodi nella rete:
  * Un nodo di ingresso $u_{in}$.
  * Un nodo di uscita $u_{out}$.
  * Un **arco interno orientato** $(u_{in}, u_{out})$ con **capacità unitaria**:
    $$c(u_{in}, u_{out}) = 1$$
    Questo arco fa da collo di bottiglia: garantisce che al massimo 1 unità di flusso (cioè al più un rover) possa transitare attraverso la cella $u$.

* **Costruzione completa della rete $G' = (V', E')$:**
  1. **Nodi ($V'$):**
     * Una sorgente $S$ e un pozzo $T$.
     * Per ogni cella $u$, la coppia $(u_{in}, u_{out})$. Totale nodi: $|V'| = 2NM + 2$.
  2. **Dalla Sorgente alle Posizioni Iniziali:**
     * Per ciascun rover $r \in \{1, \dots, k\}$ con cella di partenza $s_r$, aggiungiamo un arco orientato:
       $$(S, (s_r)_{in}) \quad \text{con capacità } c = 1$$
  3. **Archi di Adiacenza nella Griglia:**
     * Se due celle $u$ e $v$ sono adiacenti nella griglia, un rover può muoversi da $u$ a $v$ o viceversa. Aggiungiamo quindi gli archi orientati tra l'uscita di una cella e l'ingresso della cella adiacente:
       $$((u)_{out}, (v)_{in}) \quad \text{con capacità } c = 1$$
       $$((v)_{out}, (u)_{in}) \quad \text{con capacità } c = 1$$
  4. **Dalle Celle di Confine al Pozzo:**
     * Per ogni cella $b$ che si trova sul perimetro esterno della griglia $N \times M$, aggiungiamo un arco orientato verso il pozzo:
       $$((b)_{out}, T) \quad \text{con capacità } c = 1$$

```text
       (Cap: 1)         (Cap: 1)           (Cap: 1)         (Cap: 1)           (Cap: 1)
 (S) -----------> [u_in] -----> [u_out] ------------> [v_in] -----> [v_out] -----------> (T)
                 \____________________/              \____________________/
                        Cella u                             Cella v
```

---

#### 2. Condizione sul Flusso Massimo e Dimostrazione ($\iff$)

**Condizione:** Esiste un piano di evacuazione simultaneo per tutti i $k$ rover lungo cammini vertice-disgiunti se e solo se il valore del flusso massimo $F^*$ nella rete soddisfa:
$$F^* = k$$

**Dimostrazione:**

* **($\implies$) Se esiste un piano di evacuazione valido, allora $F^* = k$:**
  Supponiamo che esistano $k$ cammini vertice-disgiunti $P_1, \dots, P_k$ nella griglia che collegano ciascuna posizione iniziale $s_r$ a una cella di confine $b_r$.
  Definiamo un flusso $f$ su $G'$ inviando 1 unità di flusso lungo ciascun cammino $P_r$:
  * L'arco $(S, (s_r)_{in})$ trasporta 1 unità.
  * Per ogni cella $u$ attraversata dal cammino $P_r$, l'arco interno $(u_{in}, u_{out})$ trasporta 1 unità (rispettando la capacità $c=1$).
  * Gli archi tra celle consecutive $((u)_{out}, (v)_{in})$ trasportano 1 unità (rispettando la capacità $c=1$).
  * L'arco finale $((b_r)_{out}, T)$ trasporta 1 unità verso $T$.
  Poiché i cammini sono disgiunti sui vertici nella griglia, nessuna cella viene visitata da più di un cammino. Di conseguenza, nessun arco interno $(u_{in}, u_{out})$ riceve più di 1 unità di flusso, e le capacità non vengono mai violate.
  La conservazione del flusso è rispettata in ogni nodo intermedio.
  Il flusso totale uscente da $S$ è pari al numero di rover: $|f| = k$.
  Poiché la capacità uscente da $S$ è $\sum_{r=1}^k c(S, (s_r)_{in}) = k$, questo flusso è massimo, quindi $F^* = k$.

* **($\impliedby$) Se $F^* = k$, allora esiste un piano di evacuazione valido:**
  Tutte le capacità della rete sono intere ($1$). Per il *Teorema dell'Integralità*, esiste un flusso massimo $f^*$ a valori interi, quindi su ogni arco il flusso vale $0$ o $1$.
  * Il flusso totale è $k$. Poiché escono esattamente $k$ archi da $S$ (ciascuno con capacità 1 verso una partenza $(s_r)_{in}$), tutti i $k$ archi $(S, (s_r)_{in})$ devono essere saturi: $f^*(S, (s_r)_{in}) = 1$.
  * Per il teorema di decomposizione dei flussi (Flow Decomposition Theorem), un flusso intero da $S$ a $T$ senza cicli negativi può essere decomposto esattamente in $k$ cammini diretti disgiunti sugli archi da $S$ a $T$.
  * Ciascun cammino parte da un rover distinto $(s_r)_{in}$ e arriva a $T$ passando per una sequenza di nodi $u_{in} \to u_{out}$.
  * Poiché l'arco interno $(u_{in}, u_{out})$ ha capacità 1, il flusso totale che lo attraversa può essere al più 1. Dunque **nessun nodo intermedio $u$ può essere condiviso da due cammini diversi**.
  I $k$ cammini estratti dal flusso corrispondono quindi esattamente a $k$ traiettorie di evacuazione mutuamente disgiunte sui vertici nella griglia.

---

#### 3. Complessità Computazionale
* **Dimensioni della Rete:**
  * $|V'| = 2NM + 2 = O(NM)$.
  * Ciascuna cella ha al più 4 vicini, quindi il numero di archi di adiacenza è al più $4NM$. In totale: $|E'| = k + NM + 4NM + 4(N+M) = O(NM)$.
* **Algoritmo:**
  Poiché tutte le capacità interne dei nodi sono binarie ($c=1$), la rete è una rete a capacità unitarie. L'algoritmo di **Dinic** su reti unitarie opera in tempo:
  $$O(|E'| \sqrt{|V'|}) = O(NM \sqrt{NM}) = O((NM)^{3/2})$$
  In alternativa, con **Edmonds-Karp** la complessità è $O(|V'| \cdot |E'|^2) = O((NM)^3)$, mentre con **Ford-Fulkerson** (dato che $F^* = k \le NM$) il tempo è $O(|E'| \cdot k) = O(k \cdot NM)$.

---
---

## Problem 2: Optimal Data Stream Compression & Partitioning (Dynamic Programming)

### Problem Statement
An edge computing device collects a continuous temporal stream of $n$ integer sensor measurements $A = [a_1, a_2, \dots, a_n]$.
To transmit these data over a bandwidth-constrained network, the device must partition the sequence into contiguous chunks of blocks $C_1, C_2, \dots, C_m$.

For each chunk spanning from index $i$ to index $j$ (with $1 \le i \le j \le n$):
* There is a fixed transmission overhead and protocol header cost $F > 0$.
* The compression cost of the chunk is proportional to its internal variability:
  $$\text{Var}(i, j) = \max_{i \le \ell \le j} a_\ell - \min_{i \le \ell \le j} a_\ell$$
* Total transmission cost of the chunk $A[i \dots j]$:
  $$\text{Cost}(i, j) = F + \text{Var}(i, j)$$
* **Hardware Buffer Constraint:** The device's internal buffer cannot compress more than $K$ consecutive samples in a single chunk, meaning that any chunk must satisfy:
  $$j - i + 1 \le K$$

1. Formulate a dynamic programming **recursive equation** to find the minimum total cost to partition the entire array $A[1 \dots n]$. Clearly state the base cases.
2. Provide pseudocode for the algorithm that computes the optimal cost and reconstructs the optimal chunk boundaries.
3. Analyze the time and space complexity of your algorithm.

---

### Solution

#### 1. Equazione Ricorsiva
Definiamo $OPT[i]$ come il **minimo costo totale** per trasmettere l'intero prefisso di misurazioni $A[1 \dots i]$.

Per calcolare $OPT[i]$, consideriamo tutte le possibili scelte per l'**ultimo blocco contiguo** della sequenza, che terminerà all'indice $i$ e sarà iniziato a un certo indice $j \le i$:
* Il blocco $A[j \dots i]$ ha lunghezza $i - j + 1$.
* Per il vincolo sul buffer, la lunghezza del blocco non può superare $K$, quindi $i - j + 1 \le K \implies j \ge \max(1, i - K + 1)$.
* Il costo dell'ultimo blocco è $\text{Cost}(j, i) = F + (\max_{j \le \ell \le i} a_\ell - \min_{j \le \ell \le i} a_\ell)$.
* Il costo totale associato a questa scelta è il costo del blocco corrente sommato alla soluzione ottima per il prefisso rimanente, ovvero:
  $$OPT[j-1] + \text{Cost}(j, i)$$

La formula ricorsiva per $1 \le i \le n$ è quindi:
$$OPT[i] = \min_{\max(1, \, i - K + 1) \le j \le i} \Big( OPT[j-1] + F + \max_{j \le \ell \le i} a_\ell - \min_{j \le \ell \le i} a_\ell \Big)$$

**Caso Base:**
* $OPT[0] = 0$ (nessun costo per trasmettere 0 elementi).

---

#### 2. Pseudocodice

Possiamo calcolare il massimo e il minimo dinamicamente scorrendo $j$ all'indietro da $i$ fino a $i - K + 1$, mantenendo il massimo e il minimo correnti in tempo $O(1)$ per ogni passo:

```text
Algorithm MinStreamCompression(A[1..n], F, K):
    OPT = array of size n+1 initialized to +infinity
    OPT[0] = 0
    Split = array of size n+1 initialized to 0  // Per memorizzare il punto di split ottimo

    For i = 1 to n:
        curr_max = A[i]
        curr_min = A[i]
        
        // Iteriamo a ritroso sull'inizio del blocco j
        For j = i down to max(1, i - K + 1):
            curr_max = max(curr_max, A[j])
            curr_min = min(curr_min, A[j])
            
            chunk_cost = F + (curr_max - curr_min)
            total_cost = OPT[j - 1] + chunk_cost
            
            If total_cost < OPT[i]:
                OPT[i] = total_cost
                Split[i] = j

    // Ricostruzione dei blocchi ottimi (Backtracking)
    Chunks = empty list
    curr = n
    While curr > 0:
        start_chunk = Split[curr]
        Chunks.append((start_chunk, curr))
        curr = start_chunk - 1

    Reverse Chunks
    Return OPT[n], Chunks
```

---

#### 3. Complessità Computazionale
* **Complessità Temporale:**
  * Il ciclo esterno viene eseguito $n$ volte (per $i = 1 \dots n$).
  * Per ciascun $i$, il ciclo interno itera all'indietro su $j$ per al più $K$ passi.
  * Ad ogni passo, l'aggiornamento di `curr_max`, `curr_min` e il confronto richiedono tempo $O(1)$.
  * Il tempo per riempire l'array $OPT$ è quindi:
    $$\sum_{i=1}^n O(K) = \mathbf{O(n \cdot K)}$$
  * La fase di backtracking esegue al più $n$ salti all'indietro, impiegando $O(n)$ tempo.
  * Complessità temporale totale: **$O(n \cdot K)$**. Se $K$ è una costante di sistema, il tempo è perfettamente **lineare $O(n)$**.
* **Complessità Spaziale:**
  * Utilizziamo gli array `OPT` e `Split` di dimensione $n+1$.
  * Complessità spaziale totale: **$O(n)$**.

---
---

## Problem 3: The Set Packing Problem (NP-Completeness)

### Problem Statement
Consider the **Set Packing** decision problem:
* **Input:** A ground set (universe) $U$, a collection of subsets $\mathcal{S} = \{S_1, S_2, \dots, S_m\}$ where each $S_i \subseteq U$, and a positive integer $k \le m$.
* **Question:** Does there exist a subcollection of at least $k$ subsets $\mathcal{S}' \subseteq \mathcal{S}$ with $|\mathcal{S}'| \ge k$ such that all subsets in $\mathcal{S}'$ are **mutually disjoint** (meaning that for any pair $S_i, S_j \in \mathcal{S}'$ with $i \neq j$, $S_i \cap S_j = \emptyset$)?

1. (2 points) Prove that **Set Packing** is in **NP**.
2. (8 points) Prove that **Set Packing** is **NP-complete** by providing a polynomial-time reduction from the **Independent Set** problem. Provide the formal proof in both directions ($\implies$ and $\impliedby$).

---

### Solution

#### 1. Appartenenza a NP
* **Witness (Certificato):** Un sottoinsieme di indici $\mathcal{S}' \subseteq \{1, \dots, m\}$ di sottoinsiemi selezionati. La dimensione del witness è al più $m \log m$ bit, chiaramente polinomiale rispetto all'input.
* **Verificatore Polinomiale:**
  1. Controlla che $|\mathcal{S}'| \ge k$ in tempo $O(|\mathcal{S}'|) \le O(m)$.
  2. Crea un array di conteggio o marcatori di appartenenza per ciascun elemento dell'universo $U$, inizializzato a $0$.
  3. Per ciascun sottoinsieme $S_i \in \mathcal{S}'$, scorre tutti i suoi elementi $u \in S_i$:
     * Se l'elemento $u$ è già stato marcato in precedenza da un altro sottoinsieme, significa che due insiemi in $\mathcal{S}'$ condividono l'elemento $u$ (non sono disgiunti): il verificatore rifiuta (`NO`).
     * Altrimenti, marca $u$ come visto.
  4. Se tutti gli elementi di tutti i sottoinsiemi in $\mathcal{S}'$ vengono visitati senza collisioni, il verificatore accetta (`YES`).
* **Costo:** $O(|U| + \sum_{S_i \in \mathcal{S}'} |S_i|)$, lineare rispetto all'input. Pertanto, **$\text{Set Packing} \in \text{NP}$**.

---

#### 2. NP-Hardness: Riduzione da Independent Set ($\text{Independent Set} \le_P \text{Set Packing}$)

* **Problema di partenza (Independent Set):** Dato un grafo non orientato $G = (V, E)$ e un intero $k$, esiste un sottoinsieme di vertici $I \subseteq V$ con $|I| \ge k$ tale che nessun paio di vertici in $I$ sia collegato da un arco?

* **Costruzione dell'istanza di Set Packing $(U, \mathcal{S}, k')$:**
  1. **Universo ($U$):** L'universo di elementi coincide con l'**insieme degli archi** del grafo:
     $$U = E$$
  2. **Collezione di Sottoinsiemi ($\mathcal{S}$):** Per ciascun vertice $v \in V$, creiamo un sottoinsieme $S_v$ contenente tutti gli archi incidenti su $v$:
     $$S_v = \{e \in E \mid e \text{ è incidente su } v\}$$
     La collezione contiene un sottoinsieme per ciascun vertice: $\mathcal{S} = \{S_v \mid v \in V\}$, quindi $|\mathcal{S}| = |V| = n$.
  3. **Parametro ($k'$):** Poniamo la stessa soglia richiesta per la dimensione:
     $$k' = k$$

* **Proprietà fondamentale della costruzione:**
  Due sottoinsiemi $S_u$ e $S_v$ (con $u \neq v$) sono **disgiunti** se e solo se non condividono alcun arco. Ma l'unico arco che $u$ e $v$ possono condividere è l'arco diretto $(u, v)$ che li unisce. Dunque:
  $$S_u \cap S_v = \emptyset \iff (u, v) \notin E$$
  *(Due vertici non sono adiacenti se e solo se i rispettivi insiemi di archi incidenti sono disgiunti!)*

La costruzione richiede semplicemente di elencare gli archi incidenti per ogni nodo, eseguibile in tempo $O(|V| + |E|)$, ampiamente polinomiale.

---

#### 3. Dimostrazione di Equivalenza ($\iff$)

* **($\implies$) Se $G$ ammette un Independent Set di taglia $k$, allora $\mathcal{S}$ ammette un Set Packing di taglia $k$:**
  Sia $I \subseteq V$ un Independent Set di taglia $k$.
  Selezioniamo la sotto-collezione di insiemi corrispondente:
  $$\mathcal{S}' = \{S_v \mid v \in I\}$$
  * La cardinalità è $|\mathcal{S}'| = |I| = k$.
  * Consideriamo due qualsiasi sottoinsiemi distinti $S_u, S_v \in \mathcal{S}'$ (con $u, v \in I$). Poiché $I$ è un Independent Set, non esiste alcun arco tra $u$ e $v$ nel grafo ($ (u, v) \notin E $).
  * Di conseguenza, $S_u$ e $S_v$ non hanno alcun arco in comune: $S_u \cap S_v = \emptyset$.
  * Tutti i $k$ insiemi in $\mathcal{S}'$ sono mutuamente disgiunti, formando un Set Packing valido di taglia $k$.

* **($\impliedby$) Se $\mathcal{S}$ ammette un Set Packing di taglia $k$, allora $G$ ammette un Independent Set di taglia $k$:**
  Sia $\mathcal{S}' \subseteq \mathcal{S}$ una famiglia di almeno $k$ sottoinsiemi mutuamente disgiunti.
  Definiamo il corrispondente insieme di vertici nel grafo:
  $$I = \{v \in V \mid S_v \in \mathcal{S}'\}$$
  * La cardinalità è $|I| = |\mathcal{S}'| \ge k$.
  * Consideriamo due vertici arbitrari $u, v \in I$ con $u \neq v$. Poiché i sottoinsiemi $S_u, S_v \in \mathcal{S}'$ appartengono a un Set Packing, essi sono disgiunti: $S_u \cap S_v = \emptyset$.
  * Se esistesse un arco $e = (u, v) \in E$, tale arco apparterrebbe per definizione sia a $S_u$ che a $S_v$, implicando $e \in S_u \cap S_v \neq \emptyset$, assurdo.
  * Dunque tra $u$ e $v$ non può esserci alcun arco.
  * Poiché questo vale per ogni coppia in $I$, $I$ è un Independent Set valido di cardinalità almeno $k$.

**Conclusione:** Avendo dimostrato che $\text{Set Packing} \in \text{NP}$ e $\text{Independent Set} \le_P \text{Set Packing}$, **Set Packing è NP-Completo**.

---
---

## Problem 4: Vertex Cover via Maximal Matching (Approximation Algorithms)

### Problem Statement
Recall the **Vertex Cover** problem: given an undirected graph $G = (V, E)$, find a subset of vertices $C \subseteq V$ of minimum cardinality such that every edge $e = (u, v) \in E$ has at least one endpoint in $C$.

Consider the following simple greedy strategy:
1. Compute a **maximal matching** $M \subseteq E$ in $G$ (a matching $M$ is *maximal* if no other edge in $E \setminus M$ can be added to $M$ without violating the matching property).
2. Construct the vertex cover $C$ by selecting **both endpoints** of every edge in the matching $M$:
   $$C = \bigcup_{e = (u, v) \in M} \{u, v\}$$

1. Prove that the constructed set $C$ is a **valid vertex cover** for $G$.
2. Prove that the optimal vertex cover $C^*$ must satisfy $|C^*| \ge |M|$.
3. Prove that this algorithm achieves an **approximation ratio of 2**, that is, $|C| \le 2 \cdot |C^*|$.
4. Show a family of graphs where this algorithm achieves an approximation ratio exactly equal to 2 (tightness of the bound).

---

### Solution

#### 1. Dimostrazione che $C$ è un Vertex Cover valido
Procediamo per assurdo.
Supponiamo che $C$ non sia un vertex cover valido per $G$.
* Allora deve esistere almeno un arco $e' = (x, y) \in E$ tale che **nessuno dei suoi due estremi appartiene a $C$** ($x \notin C$ e $y \notin C$).
* Poiché $C$ contiene entrambi gli estremi di tutti gli archi presenti nel matching $M$, il fatto che $x \notin C$ e $y \notin C$ implica che:
  * Il vertice $x$ non è incidente a nessun arco in $M$.
  * Il vertice $y$ non è incidente a nessun arco in $M$.
* Di conseguenza, l'arco $e' = (x, y)$ non condivide alcun estremo con nessun arco del matching $M$.
* Ma allora potremmo aggiungere l'arco $e'$ al matching $M$, ottenendo un matching più grande $M \cup \{e'\}$.
* Questo è in palese contraddizione con l'ipotesi che $M$ sia un matching **massimale** (per definizione, un matching è massimale se non è contenuto propriamente in nessun altro matching).
* Dunque non può esistere alcun arco scoperto, e $C$ è un Vertex Cover valido.

---

#### 2. Dimostrazione del Lower Bound sull'Ottimo ($|C^*| \ge |M|$)
Sia $M$ un qualsiasi matching in $G$ e sia $C^*$ un vertex cover ottimo.
* Per definizione di matching, tutti gli archi in $M$ sono **a due a due disgiunti sui vertici** (nessun paio di archi in $M$ condivide un estremo comune).
* Per essere un vertex cover valido, $C^*$ deve coprire tutti gli archi del grafo, e in particolare deve coprire tutti gli archi di $M$.
* Poiché nessun vertice può coprire più di un arco di $M$ contemporaneamente (essendo gli archi disgiunti), $C^*$ deve contenere **almeno un vertice distinto per ciascun arco di $M$**.
* Ne consegue direttamente il lower bound:
  $$|C^*| \ge |M|$$

---

#### 3. Dimostrazione del Rapporto di Approssimazione ($ALG \le 2 \cdot OPT$)
* Per costruzione dell'algoritmo, il set $C$ include entrambi gli estremi di ciascun arco in $M$.
* Poiché gli archi di $M$ sono disgiunti sui vertici, ogni arco in $M$ contribuisce con esattamente 2 vertici distinti a $C$:
  $$|C| = 2 \cdot |M|$$
* Utilizzando il lower bound ricavato al punto precedente ($|M| \le |C^*|$):
  $$|C| = 2 \cdot |M| \le 2 \cdot |C^*|$$
* Dividendo per la dimensione dell'ottimo $|C^*|$:
  $$\frac{|C|}{|C^*|} \le 2$$
L'algoritmo garantisce una **2-approssimazione deterministica** nel caso peggiore.

---

#### 4. Tightness del Bound (Esempio in cui il rapporto vale esattamente 2)
Consideriamo un grafo $G$ formato da $k$ copie disgiunte del grafo completo $K_2$ (ovvero $k$ archi indipendenti senza vertici in comune):
$$E = \{(u_1, v_1), (u_2, v_2), \dots, (u_k, v_k)\}$$
* Il matching massimale $M$ include necessariamente tutti e $k$ gli archi: $|M| = k$.
* L'algoritmo seleziona entrambi gli estremi di ogni arco:
  $$|C| = 2k$$
* D'altra parte, per coprire ciascun arco indipendente $(u_i, v_i)$ basta selezionare un solo estremo (ad esempio tutti i vertici $u_i$):
  $$|C^*| = k$$
* Il rapporto di approssimazione per questa istanza è:
  $$\frac{|C|}{|C^*|} = \frac{2k}{k} = 2$$
Questo dimostra che il fattore 2 è stretto (*tight*).

