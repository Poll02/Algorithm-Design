# Algorithm Design - Mock Exam 3

Questo mock exam è stato strutturato sul modello delle prove d'esame ufficiali dei proff. Anagnostopoulos, Leonardi e Fioravanti. Presenta 4 esercizi inediti che coprono i quattro pilastri del corso: **Flussi di Rete**, **Programmazione Dinamica**, **NP-Completezza** e **Algoritmi di Approssimazione**.

---

## Problem 1: Emergency Medical Drone Dispatch (Network Flow)

### Problem Statement
A humanitarian organization must coordinate the distribution of emergency medical supplies to $n$ remote villages after an earthquake. 
* Each village $i \in \{1, \dots, n\}$ requires an exact amount of $d_i \in \mathbb{N}^+$ blood units.
* There are $m$ drone launch bases available. Each base $j \in \{1, \dots, m\}$ has a storage capacity of at most $s_j \in \mathbb{N}^+$ blood units.
* Due to drone battery ranges and topographical obstacles (such as mountains), base $j$ can only reach a specific subset of villages $V(j) \subseteq \{1, \dots, n\}$.
* Furthermore, to avoid dangerous mid-air collisions over the local airspace of village $i$, base $j$ is not allowed to send more than $c_{ji} \in \mathbb{N}^+$ total units to village $i$.

1. Model the problem of determining whether all villages can receive their required supplies as a **Maximum Flow** problem. Specify the vertices, directed edges, and capacities of the network.
2. Formulate the exact condition on the maximum flow value that determines if a feasible distribution exists, and prove the correctness of the reduction in both directions ($\implies$ and $\impliedby$).
3. Discuss the computational running time of the procedure using an efficient max-flow algorithm.

---

### Solution

#### 1. Costruzione della Rete di Flusso $G = (V, E)$
Costruiamo un grafo orientato con sorgente $S$ e pozzo $T$:

* **Nodi ($V$):**
  * La sorgente $S$ e il pozzo $T$.
  * Un nodo $B_j$ per ogni base di droni, per $j \in \{1, \dots, m\}$.
  * Un nodo $U_i$ per ogni villaggio, per $i \in \{1, \dots, n\}$.
  * Totale vertici: $|V| = m + n + 2$.

* **Archi e Capacità ($E$):**
  1. **Dalla Sorgente alle Basi:** Aggiungiamo un arco orientato $(S, B_j)$ per ciascuna base $j$, con capacità pari alle scorte della base:
     $$c(S, B_j) = s_j \quad \forall j \in \{1, \dots, m\}$$
  2. **Dalle Basi ai Villaggi Raggiungibili:** Per ogni base $j$ e per ciascun villaggio $i \in V(j)$ raggiungibile dalla base, aggiungiamo un arco orientato $(B_j, U_i)$ con capacità limitata dal tetto sullo spazio aereo:
     $$c(B_j, U_i) = c_{ji} \quad \forall j \in \{1, \dots, m\}, \, i \in V(j)$$
  3. **Dai Villaggi al Pozzo:** Aggiungiamo un arco orientato $(U_i, T)$ per ciascun villaggio $i$, con capacità pari alla domanda richiesta:
     $$c(U_i, T) = d_i \quad \forall i \in \{1, \dots, n\}$$

```text
            (Cap: s_1)                 (Cap: c_ji)                (Cap: d_1)
         /---> [Base B_1] ---+--------------------------+---> [Villaggio U_1] ---\
        /                     \                        /                          \
      (S)                      X (connessioni basi-   X                            (T)
        \                     /   villaggi validi)   /                            /
         \---> [Base B_m] ---+--------------------------+---> [Villaggio U_n] ---/
            (Cap: s_m)                 (Cap: c_ji)                (Cap: d_n)
```

---

#### 2. Condizione sul Flusso Massimo e Dimostrazione di Correttezza

**Condizione:** Esiste un piano di distribuzione ammissibile se e solo se il valore del flusso massimo $F^*$ nella rete soddisfa:
$$F^* = \sum_{i=1}^n d_i$$

**Dimostrazione:**

* **($\implies$) Se esiste una distribuzione ammissibile, allora $F^* = \sum_{i=1}^n d_i$:**
  Sia $x_{ji} \ge 0$ il numero di unità inviate dalla base $j$ al villaggio $i$.
  Definiamo il flusso $f$ su $G$:
  * $f(B_j, U_i) = x_{ji}$ per ogni $i \in V(j)$. Poiché la distribuzione rispetta i vincoli di traffico aereo, $f(B_j, U_i) \le c_{ji}$.
  * $f(S, B_j) = \sum_{i \in V(j)} x_{ji}$. Poiché la base $j$ non invia più delle sue scorte disponibili, $f(S, B_j) \le s_j = c(S, B_j)$. Inoltre, la conservazione del flusso nel nodo $B_j$ è rispettata.
  * $f(U_i, T) = \sum_{j : i \in V(j)} x_{ji} = d_i = c(U_i, T)$, poiché ogni villaggio riceve esattamente la domanda richiesta. La conservazione in $U_i$ è garantita e l'arco è saturo.
  Tutti i vincoli di capacità e conservazione sono soddisfatti. Il valore totale del flusso è:
  $$|f| = \sum_{i=1}^n f(U_i, T) = \sum_{i=1}^n d_i$$
  Poiché la capacità del taglio $(V \setminus \{T\}, \{T\})$ è $\sum_{i=1}^n c(U_i, T) = \sum_{i=1}^n d_i$, per la Weak Duality il flusso non può superare questo valore. Dunque $F^* = \sum_{i=1}^n d_i$.

* **($\impliedby$) Se $F^* = \sum_{i=1}^n d_i$, allora esiste una distribuzione ammissibile:**
  Tutte le capacità sono intere, quindi per il *Teorema dell'Integralità* esiste un flusso massimo $f^*$ a valori interi.
  * Poiché il valore totale del flusso è $\sum_{i=1}^n d_i$ e la somma delle capacità degli archi entranti in $T$ è $\sum_{i=1}^n c(U_i, T) = \sum_{i=1}^n d_i$, **tutti gli archi $(U_i, T)$ devono essere saturi**:
    $$f^*(U_i, T) = d_i \quad \forall i \in \{1, \dots, n\}$$
  * Per la conservazione del flusso nel nodo $U_i$:
    $$\sum_{j : i \in V(j)} f^*(B_j, U_i) = f^*(U_i, T) = d_i$$
    Quindi il villaggio $i$ riceve esattamente $d_i$ unità complessive dalle basi collegate.
  * Per ogni arco intermedio, $f^*(B_j, U_i) \le c_{ji}$, rispettando il vincolo di collisione aerea.
  * Per la conservazione nel nodo $B_j$:
    $$\sum_{i \in V(j)} f^*(B_j, U_i) = f^*(S, B_j) \le c(S, B_j) = s_j$$
    Quindi nessuna base supera le proprie scorte $s_j$.
  L'assegnazione data da $x_{ji} = f^*(B_j, U_i)$ è pertanto ammissibile e soddisfa la richiesta di tutti i villaggi.

---

#### 3. Complessità Computazionale
* **Dimensioni della Rete:**
  * $|V| = m + n + 2 = O(m + n)$.
  * $|E| = m + n + \sum_{j=1}^m |V(j)| \le m + n + m \cdot n = O(m \cdot n)$.
* **Algoritmo di Risoluzione:**
  * Usando **Edmonds-Karp** ($O(|V| \cdot |E|^2)$), la complessità è strettamente polinomiale:
    $$O((m + n) \cdot (m \cdot n)^2)$$
  * Usando **Dinic**, la complessità su reti con queste caratteristiche scende a $O(|V|^2 |E|) = O((m+n)^2 \cdot mn)$.

---
---

## Problem 2: Cloud Server Autoscaling with Cooldown (Dynamic Programming)

### Problem Statement
A cloud service provider needs to schedule the execution mode of a server over a planning horizon of $n$ consecutive hours $t = 1, 2, \dots, n$. For each hour $t$:
* Running in **High-Performance Mode** yields a revenue of $H_t \in \mathbb{R}^+$.
* Running in **Low-Power Mode** yields a revenue of $L_t \in \mathbb{R}^+$ (with $L_t < H_t$).
* Keeping the server **Off** yields a revenue of $0$.

**Thermal Cooldown Constraint:** Running in High-Performance mode causes high hardware heating. Consequently, whenever the server operates in High-Performance mode at hour $t$, it is physically prohibited from operating in High-Performance mode for the next $k$ consecutive hours (that is, during hours $t+1, t+2, \dots, t+k$ the server can only operate in Low-Power mode or be Off). Here $k \ge 1$ is a fixed integer constant.

1. Let $OPT[t]$ be the maximum revenue achievable over the first $t$ hours. Formulate a **recursive formula** for $OPT[t]$, clearly defining the base cases.
2. Provide an efficient implementation (pseudocode) that computes the maximum revenue and reconstructs the optimal schedule of operations.
3. Discuss the time and space complexity of your solution.

---

### Solution

#### 1. Formula Ricorsiva
Definiamo $OPT[t]$ come il massimo guadagno ottenibile considerando le prime $t$ ore (da $1$ a $t$).

Allo slot orario $t$, abbiamo tre scelte mutuamente esclusive per il server:
1. **Tenere il server SPENTO all'ora $t$:**
   Guadagno 0 per l'ora $t$, il profitto totale è $OPT[t-1]$.
2. **Eseguire in LOW-POWER all'ora $t$:**
   Guadagniamo $L_t$, e poiché la modalità a basso consumo non causa stress termico, non pone vincoli sull'ora precedente: il profitto totale è $L_t + OPT[t-1]$.
3. **Eseguire in HIGH-PERFORMANCE all'ora $t$:**
   Guadagniamo $H_t$. A causa del vincolo di cooldown di $k$ ore, se usiamo la modalità High all'ora $t$, le precedenti $k$ ore $(t-k, \dots, t-1)$ **non potevano essere in modalità High**, ma potevano solo essere in modalità Low-Power (o Off).
   Poiché $L_\tau > 0$, per massimizzare il profitto durante le $k$ ore di cooldown obbligato, la scelta migliore è eseguire tutte quelle ore in Low-Power!
   Quindi il guadagno totale è:
   $$H_t + \sum_{j=1}^{\min(k, t-1)} L_{t-j} + OPT[t - k - 1]$$
   (dove per convenzione $OPT[\le 0] = 0$).

La formula ricorsiva globale per $t \ge 1$ è:
$$OPT[t] = \max \begin{cases}
OPT[t-1] & \text{(Off all'ora } t) \\
OPT[t-1] + L_t & \text{(Low-Power all'ora } t) \\
OPT[\max(0, t - k - 1)] + H_t + \sum_{j=1}^{\min(k, t-1)} L_{t-j} & \text{(High-Performance all'ora } t)
\end{cases}$$

*(Nota: Poiché $L_t > 0$, la scelta "Low-Power" domina sempre la scelta "Off", quindi il primo termine può essere omesso).*

**Casi Base:**
* $OPT[0] = 0$.

---

#### 2. Pseudocodice

Per ottenere un tempo lineare $O(n)$, precalcoliamo le somme prefisse dell'array $L$ per valutare $\sum L$ in tempo $O(1)$:

```text
Algorithm OptimalServerSchedule(n, k, H, L):
    // Precalcolo delle somme prefisse di L per calcoli O(1)
    PrefL = array of size n+1, PrefL[0] = 0
    For t = 1 to n:
        PrefL[t] = PrefL[t-1] + L[t]

    OPT = array of size n+1, OPT[0] = 0
    Choice = array of size n+1  // Memorizza la scelta ottima per il backtracking

    For t = 1 to n:
        // Opzione 1: Low-Power all'ora t
        opt_low = OPT[t-1] + L[t]

        // Opzione 2: High all'ora t (con k ore precedenti forzate a Low)
        start_cool = max(1, t - k)
        cooldown_gain = PrefL[t-1] - PrefL[start_cool - 1]
        prev_opt_index = max(0, t - k - 1)
        opt_high = H[t] + cooldown_gain + OPT[prev_opt_index]

        // Scelta del massimo
        If opt_high > opt_low:
            OPT[t] = opt_high
            Choice[t] = "HIGH"
        Else:
            OPT[t] = opt_low
            Choice[t] = "LOW"

    // Ricostruzione della soluzione (Backtracking)
    Schedule = array of size n
    curr = n
    While curr >= 1:
        If Choice[curr] == "LOW":
            Schedule[curr] = "LOW"
            curr = curr - 1
        Else:
            Schedule[curr] = "HIGH"
            // Le k ore precedenti a curr erano in cooldown forzato a Low
            For j = 1 to min(k, curr - 1):
                Schedule[curr - j] = "LOW"
            curr = curr - k - 1

    Return OPT[n], Schedule
```

---

#### 3. Complessità
* **Tempo:**
  * Il precalcolo delle somme prefisse richiede $O(n)$ tempo.
  * Il ciclo principale esegue $n$ iterazioni, e ogni iterazione compie operazioni aritmetiche in tempo costante $O(1)$ grazie alle somme prefisse.
  * La fase di backtracking percorre la lista all'indietro in al più $n$ passi.
  * Complessità temporale totale: **$O(n)$**, ottimale.
* **Spazio:**
  * Utilizziamo array di dimensione $n+1$ per `OPT`, `PrefL` e `Choice`.
  * Complessità spaziale totale: **$O(n)$**.

---
---

## Problem 3: The Hitting Set Problem (NP-Completeness)

### Problem Statement
In the **Hitting Set** problem, we are given:
* A ground set of $n$ elements $U = \{1, 2, \dots, n\}$.
* A collection of $m$ subsets of $U$, denoted by $\mathcal{S} = \{S_1, S_2, \dots, S_m\}$, where $S_j \subseteq U$ for all $j \in \{1, \dots, m\}$.
* A positive integer $k \le n$.

The question is whether there exists a subset $H \subseteq U$ with size $|H| \le k$ such that $H$ "hits" every set in $\mathcal{S}$, meaning that $H \cap S_j \neq \emptyset$ for every $j \in \{1, \dots, m\}$.

1. Prove that **Hitting Set** is in **NP**.
2. Prove that **Hitting Set** is **NP-complete** by giving a polynomial-time reduction from **Vertex Cover**.
3. Prove the correctness of the reduction in both directions ($\implies$ and $\impliedby$).

---

### Solution

#### 1. Appartenenza a NP
* **Certificato (Witness):** Un sottoinsieme $H \subseteq U$. La sua taglia è al più $|U| = n$ elementi, quindi la sua codifica ha dimensione polinomiale rispetto all'input.
* **Verificatore Polinomiale:**
  1. Controlla che $|H| \le k$ e $H \subseteq U$ in tempo $O(|H|) \le O(n)$.
  2. Crea un vettore caratteristico booleano `In_H` di dimensione $n$ inizializzato con i nodi di $H$ in tempo $O(n)$.
  3. Per ciascun sottoinsieme $S_j \in \mathcal{S}$ ($j = 1 \dots m$), scorre gli elementi $u \in S_j$ e verifica se almeno uno ha `In_H[u] == true`. Se per un insieme nessuno appartiene a $H$, restituisce `NO`.
  4. Se tutti gli $m$ sottoinsiemi sono intersecati, restituisce `YES`.
* **Costo:** $O(n + \sum_{j=1}^m |S_j|)$, lineare nella dimensione dell'input. Pertanto, **$\text{Hitting Set} \in \text{NP}$**.

---

#### 2. Riduzione Polinomiale da Vertex Cover ($\text{Vertex Cover} \le_P \text{Hitting Set}$)

* **Problema di partenza (Vertex Cover):** Dato un grafo non orientato $G = (V, E)$ e un intero $k$, esiste un sottoinsieme di vertici $C \subseteq V$ con $|C| \le k$ tale che ogni arco $e = (u, v) \in E$ abbia almeno un estremo in $C$?
* **Costruzione dell'istanza di Hitting Set $(U, \mathcal{S}, k')$:**
  1. **Universo ($U$):** Poniamo l'universo pari all'insieme dei vertici del grafo:
     $$U = V$$
  2. **Collezione di Sottoinsiemi ($\mathcal{S}$):** Per ciascun arco $e = (u, v) \in E$, creiamo un sottoinsieme di 2 elementi contenente i suoi estremi:
     $$S_e = \{u, v\}$$
     La collezione è formata da $|E|$ sottoinsiemi: $\mathcal{S} = \{S_e \mid e \in E\}$.
  3. **Parametro ($k'$):** Manteniamo la stessa soglia numerica:
     $$k' = k$$

La costruzione associa ogni arco a un sottoinsieme di due vertici, richiedendo tempo $O(|V| + |E|)$, pienamente polinomiale.

---

#### 3. Dimostrazione di Correttezza ($\iff$)

* **($\implies$) Se $G$ ammette un Vertex Cover di taglia $\le k$, allora esiste un Hitting Set di taglia $\le k'$:**
  Sia $C \subseteq V$ un Vertex Cover con $|C| \le k$.
  Scegliamo come candidato Hitting Set esattamente $H = C \subseteq U$:
  * La cardinalità rispetta il limite: $|H| = |C| \le k = k'$.
  * Per definizione di Vertex Cover, ogni arco $e = (u, v) \in E$ ha almeno un estremo in $C$, il che significa che almeno uno tra $u$ e $v$ appartiene a $C$.
  * Di conseguenza, per ogni sottoinsieme $S_e = \{u, v\} \in \mathcal{S}$, abbiamo:
    $$H \cap S_e = C \cap \{u, v\} \neq \emptyset$$
  * Dunque $H$ interseca ("hits") tutti i sottoinsiemi di $\mathcal{S}$.

* **($\impliedby$) Se esiste un Hitting Set di taglia $\le k'$, allora $G$ ammette un Vertex Cover di taglia $\le k$:**
  Sia $H \subseteq U$ un Hitting Set con $|H| \le k'$.
  Poniamo $C = H \subseteq V$:
  * La cardinalità è $|C| = |H| \le k' = k$.
  * Poiché $H$ è un Hitting Set valido, per ciascun sottoinsieme $S_e = \{u, v\} \in \mathcal{S}$ deve valere $H \cap S_e \neq \emptyset$.
  * Questo significa che almeno un elemento tra $u$ e $v$ appartiene a $H = C$.
  * Poiché questo è vero per ogni arco $e = (u, v) \in E$, ogni arco del grafo ha almeno un estremo in $C$.
  * Dunque $C$ è un Vertex Cover valido di dimensione al più $k$.

**Conclusione:** Poiché Vertex Cover è NP-completo e appartiene a NP, **Hitting Set è NP-Completo**.

---
---

## Problem 4: Maximum 3-Way Cut (Randomized Approximation & Derandomization)

### Problem Statement
Consider the **Max-3-Cut** problem: given an undirected graph $G = (V, E)$ with non-negative edge weights $w_e \ge 0$ for each $e \in E$, the goal is to partition the vertex set $V$ into **three disjoint subsets** $(V_1, V_2, V_3)$ (such that $V_1 \cup V_2 \cup V_3 = V$) to maximize the total weight of edges that cross between different sets:
$$w(V_1, V_2, V_3) = \sum_{e = (u, v) \in E, \, u \in V_a, v \in V_b, \, a \neq b} w_e$$

1. Design a simple randomized approximation algorithm that independently assigns each vertex $v \in V$ to one of the three sets $V_1, V_2, V_3$ uniformly at random with probability $\frac{1}{3}$.
2. Prove that the expected weight of the cut produced by this randomized algorithm achieves an **expected approximation ratio of $\frac{2}{3}$**.
3. Show how to **derandomize** this algorithm using the **Method of Conditional Expectations** to obtain a deterministic greedy algorithm that guarantees a cut of weight at least $\frac{2}{3} OPT$ in time $O(|V| + |E|)$.

---

### Solution

#### 1. Algoritmo Randomizzato
Per ogni vertice $v \in V$, scegliamo indipendentemente un indice $k \in \{1, 2, 3\}$ in modo uniforme, ponendo $v \in V_k$ con probabilità:
$$\Pr[v \in V_k] = \frac{1}{3} \quad \text{per } k \in \{1, 2, 3\}$$

---

#### 2. Dimostrazione dell'Expected Approximation Ratio ($\ge \frac{2}{3}$)
Consideriamo un generico arco $e = (u, v) \in E$.
L'arco $e$ **non viene tagliato** se e solo se entrambi gli estremi $u$ e $v$ vengono assegnati allo stesso identico insieme:
$$\Pr[u \text{ e } v \text{ nello stesso insieme}] = \sum_{k=1}^3 \Pr[u \in V_k \land v \in V_k]$$
Poiché le assegnazioni dei vertici sono stocasticamente indipendenti:
$$\Pr[u \text{ e } v \text{ nello stesso insieme}] = \sum_{k=1}^3 \left(\frac{1}{3} \cdot \frac{1}{3}\right) = 3 \cdot \frac{1}{9} = \frac{1}{3}$$

La probabilità che l'arco $e$ **venga tagliato** (i suoi estremi finiscano in insiemi diversi) è l'evento complementare:
$$\Pr[e \text{ è tagliato}] = 1 - \frac{1}{3} = \frac{2}{3}$$

Definiamo la variabile indicatrice $X_e$ che vale $1$ se $e$ è tagliato e $0$ altrimenti. Il valore atteso del peso del taglio è:
$$\mathbb{E}[ALG] = \sum_{e \in E} w_e \cdot \mathbb{E}[X_e] = \sum_{e \in E} w_e \cdot \Pr[e \text{ è tagliato}] = \frac{2}{3} \sum_{e \in E} w_e$$

Sia $W = \sum_{e \in E} w_e$ il peso totale di tutti gli archi del grafo. Poiché una soluzione ottima $OPT$ non può tagliare un peso superiore a $W$ ($OPT \le W$):
$$\mathbb{E}[ALG] = \frac{2}{3} W \ge \frac{2}{3} OPT$$

L'algoritmo raggiunge un expected approximation ratio di **$\frac{2}{3}$**.

---

#### 3. Derandomizzazione (Metodo delle Aspettative Condizionate)

Fissiamo un ordinamento arbitrario dei vertici: $v_1, v_2, \dots, v_n$.
Assegniamo i vertici uno alla volta. Supponiamo di aver già assegnato i vertici $v_1, \dots, v_{i-1}$ ai rispettivi insiemi $V_1, V_2, V_3$.

Quando dobbiamo posizionare $v_i$, per ciascuna delle tre possibili scelte $k \in \{1, 2, 3\}$:
* Gli archi tra $v_i$ e i vertici futuri $\{v_{i+1}, \dots, v_n\}$ avranno probabilità di essere tagliati pari a $\frac{2}{3}$, indipendentemente dalla partizione scelta per $v_i$.
* La scelta di $v_i$ influisce unicamente sugli archi incidenti verso i vertici **già posizionati** $\{v_1, \dots, v_{i-1}\}$:
  * Se assegniamo $v_i$ a $V_k$, tagliamo tutti gli archi che collegano $v_i$ ai nodi già posizionati nelle **altre due partizioni** diverse da $k$.
  * Definiamo $w(v_i, V_k)$ la somma dei pesi degli archi tra $v_i$ e i vicini già assegnati all'insieme $V_k$.
  * Il peso degli archi tagliati verso i nodi precedenti scegliendo l'insieme $k$ è:
    $$\text{Cut\_Gain}(k) = \sum_{j \neq k} w(v_i, V_j) = \left( \sum_{j=1}^3 w(v_i, V_j) \right) - w(v_i, V_k)$$

Per massimizzare l'aspettativa condizionata, dobbiamo massimizzare $\text{Cut\_Gain}(k)$, il che equivale a **minimizzare $w(v_i, V_k)$** (assegnare $v_i$ all'insieme con cui ha il minor peso di connessioni già presenti).

```text
Algorithm DeterministicMax3Cut(G = (V, E), w):
    V_1 = empty set, V_2 = empty set, V_3 = empty set

    For each vertex v in V:
        // Calcola il peso dei vicini già assegnati a ciascuno dei 3 insiemi
        w1 = sum of w(v, u) for all neighbors u of v already in V_1
        w2 = sum of w(v, u) for all neighbors u of v already in V_2
        w3 = sum of w(v, u) for all neighbors u of v already in V_3

        // Scegli l'insieme k che minimizza il peso verso la stessa partizione
        // (massimizzando gli archi tagliati verso le altre due)
        k = argmin(w1, w2, w3)
        Add v to V_k

    Return (V_1, V_2, V_3)
```

**Garanzia di Approssimazione:**
A ogni passo $i$, poiché scegliamo il minimo tra tre valori, vale:
$$w(v_i, V_k) \le \frac{w_1 + w_2 + w_3}{3}$$
Quindi gli archi tagliati verso i nodi precedenti sono almeno:
$$\text{Cut\_Gain}(k) = (w_1 + w_2 + w_3) - w(v_i, V_k) \ge \frac{2}{3} (w_1 + w_2 + w_3)$$
Sommando su tutti gli $n$ passi, ogni arco viene considerato una sola volta, garantendo:
$$ALG_{\text{det}} \ge \frac{2}{3} \sum_{e \in E} w_e \ge \frac{2}{3} OPT$$

**Complessità:**
Ogni vertice viene esaminato una volta. Ciascun arco $e = (u, v)$ viene controllato una volta quando si posiziona il secondo estremo, calcolando i pesi in tempo proporzionale al grado del vertice.
Tempo di esecuzione totale: **$O(|V| + |E|)$**, lineare.

