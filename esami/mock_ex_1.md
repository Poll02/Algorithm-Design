# Algorithm Design - Mock Exam 1

## Problem 1: Mining Operations (Max-Flow Min-Cut)

**Problem Statement:**
A mining company is evaluating a new open-pit mining site. The site is divided into $n$ blocks of earth. Each block $i \in \{1, \dots, n\}$ has a known net profit $p_i \in \mathbb{R}$, which can be positive (it contains valuable ore) or negative (it is just dirt and costs money to extract). 
Because of the physical structure of the terrain, you cannot extract a block $i$ unless you have also extracted a specific set of blocks lying immediately above it. These physical dependencies are given as a set of directed edges $D$, where a directed edge $(i, j) \in D$ means "extracting block $i$ requires extracting block $j$".
1. Model the problem of finding a valid set of blocks to extract that maximizes the total profit as a **Max-Flow Min-Cut** problem. Specify nodes, edges, and capacities clearly.
2. Formally justify why the minimum cut in your network corresponds to the maximum profit valid selection.
3. Discuss the computational complexity of your algorithm.

---

### Solution

**1. Algorithm Design (Graph Construction)**
Costruiamo un grafo orientato $G = (V, E)$ per modellare le dipendenze e i profitti dei blocchi:
* **Nodi:** Definiamo un nodo sorgente $S$, un nodo pozzo $T$, e un nodo per ogni blocco $i \in \{1, \dots, n\}$.
* **Archi dei Profitti Positivi:** Per ogni blocco $i$ tale che $p_i \ge 0$, aggiungiamo un arco dalla sorgente al blocco $(S, i)$ con capacità pari al profitto: $c(S, i) = p_i$.
* **Archi dei Costi (Profitti Negativi):** Per ogni blocco $j$ tale che $p_j < 0$, aggiungiamo un arco dal blocco al pozzo $(j, T)$ con capacità pari al costo assoluto: $c(j, T) = -p_j$.
* **Archi di Dipendenza:** Per ogni dipendenza fisica $(i, j) \in D$, aggiungiamo un arco dal blocco $i$ al blocco $j$ con capacità infinita: $c(i, j) = \infty$.

Calcoliamo il flusso massimo da $S$ a $T$ e troviamo il taglio minimo associato $(A, B)$ prendendo tutti i nodi raggiungibili da $S$ nel grafo residuo. I blocchi da estrarre corrispondono all'insieme $A \setminus \{S\}$.

**2. Proof of Correctness**
Dimostriamo che la capacità del taglio $(A, B)$ è legata al profitto dell'insieme estratto $A$.
Poiché il taglio ha capacità finita, nessun arco con capacità $\infty$ viene tagliato, garantendo che se un blocco $i \in A$, allora anche le sue dipendenze necessarie si troveranno in $A$ (la soluzione è fisicamente ammissibile).

La capacità del taglio è composta dagli archi recisi:
$$c(A, B) = \sum_{i \in B, p_i \ge 0} p_i + \sum_{j \in A, p_j < 0} (-p_j)$$

Sia $P^+ = \sum_{i=1}^n \max(0, p_i)$ la somma totale di tutti i profitti positivi possibili nella miniera. Possiamo scrivere:
$$P^+ = \sum_{i \in A, p_i \ge 0} p_i + \sum_{i \in B, p_i \ge 0} p_i \implies \sum_{i \in B, p_i \ge 0} p_i = P^+ - \sum_{i \in A, p_i \ge 0} p_i$$

Sostituendo nell'equazione del taglio otteniamo:
$$c(A, B) = P^+ - \sum_{i \in A, p_i \ge 0} p_i + \sum_{j \in A, p_j < 0} (-p_j)$$
$$c(A, B) = P^+ - \left( \sum_{i \in A, p_i \ge 0} p_i - \sum_{j \in A, p_j < 0} (-p_j) \right)$$

Il termine tra parentesi è il **Profitto Netto** (guadagni meno perdite) generato dall'estrazione dei blocchi in $A$. Quindi:
$$c(A, B) = P^+ - \text{Profitto}(A)$$
Poiché $P^+$ è una costante, trovare il taglio $(A, B)$ di capacità minima equivale matematicamente a massimizzare il $\text{Profitto}(A)$.

**3. Computational Complexity**
La rete ha $|V| = n + 2 = O(n)$ nodi.
Il numero di archi è al massimo $|E| = n + |D| = O(n + |D|)$ (un arco per ogni blocco verso $S$ o $T$, più gli archi di dipendenza).
Utilizzando l'algoritmo di Edmonds-Karp per calcolare il flusso massimo, la cui complessità è $O(|V| \cdot |E|^2)$, otteniamo un tempo di esecuzione di:
$$O(n \cdot (n + |D|)^2)$$

---

## Problem 2: 5G Antenna Placement (Dynamic Programming)

**Problem Statement:**
A telecommunication company needs to place new 5G antennas along a straight highway of length $L$ kilometers. They have identified $n$ candidate positions $x_1, x_2, \dots, x_n$ (where $0 < x_1 < x_2 < \dots < x_n < L$). 
Placing an antenna at position $x_i$ yields an expected revenue of $r_i > 0$. However, to avoid signal interference, no two placed antennas can be closer than $D$ kilometers to each other. In other words, if you select positions $x_i$ and $x_j$, it must be true that $|x_i - x_j| \ge D$.
1. Let $OPT[i]$ be the maximum revenue obtainable considering only a valid subset of the first $i$ candidate positions. Formulate a suitable **recursive formula** for this problem. Define clearly what the base cases are.
2. Provide an efficient implementation of the dynamic programming algorithm (pseudocode) to compute the maximum total revenue.
3. Discuss its space and time complexities.

---

### Solution

**1. Recursive Formula**
Definiamo $OPT[i]$ come il massimo ricavo ottenibile considerando solo un sottoinsieme valido delle prime $i$ antenne.
Per gestire correttamente il vincolo fisico della distanza, definiamo una funzione ausiliaria $prev(i)$ che restituisce l'indice della prima antenna (guardando da destra verso sinistra) compatibile con l'antenna $i$:
* $prev(i) = \max \{j < i \mid x_i - x_j \ge D\}$
* Se nessuna antenna precedente è compatibile, $prev(i) = 0$.

Per l'antenna $i$, abbiamo due scelte:
* **Includere $i$:** Otteniamo il ricavo $r_i$ più il valore ottimo considerando le antenne compatibili, ovvero $OPT[prev(i)]$.
* **Escludere $i$:** Il valore ottimo è semplicemente l'ottimo calcolato per le prime $i-1$ antenne, ovvero $OPT[i-1]$.

La formula ricorsiva è il massimo tra queste due scelte:
$$OPT[i] = \max \{ r_i + OPT[prev(i)], \ OPT[i-1] \}$$
**Base Case:** $OPT[0] = 0$.

**2. Pseudocode**
```text
Algorithm OptAntennasPlacement(X, R, D, n):
    // Precomputing the prev array
    prev = array of size n+1 initialized to 0
    For i = 1 to n:
        // Use Binary Search to find the largest index j < i where X[i] - X[j] >= D
        prev[i] = BinarySearch(X, 1, i-1, X[i] - D) 
        
    OPT = array of size n+1 initialized to 0
    OPT[0] = 0
    
    // DP Evaluation
    For i = 1 to n:
        if R[i] + OPT[prev[i]] > OPT[i-1]:
            OPT[i] = R[i] + OPT[prev[i]]
        else:
            OPT[i] = OPT[i-1]
            
    return OPT[n]
```

**3. Space and Time Complexities**
* **Time Complexity:** Il precalcolo dell'array `prev` può essere ottimizzato in quanto le posizioni $x_1 \dots x_n$ sono già ordinate in modo crescente. Per ogni indice $i$, la ricerca binaria (`BinarySearch`) per trovare l'elemento compatibile più vicino impiega tempo $O(\log n)$. Fare questo per tutti gli $n$ elementi richiede tempo $O(n \log n)$. Il ciclo successivo per calcolare la ricorrenza valuta $n$ stati, e ogni calcolo interno richiede $O(1)$ tempo, costando in totale $O(n)$. La complessità temporale asintotica totale è limitata dalla ricerca binaria, pari a **$O(n \log n)$**. *(Nota: sarebbe anche possibile utilizzare un approccio a due puntatori (sliding window) per precalcolare l'array `prev` in $O(n)$).*
* **Space Complexity:** L'algoritmo richiede spazio per memorizzare l'array `prev` e la tabella di programmazione dinamica `OPT`, ciascuno di dimensione $n+1$. Pertanto, la complessità spaziale è **$O(n)$**.

---

## Problem 3: The Partition Problem (NP-Completeness)

**Problem Statement:**
Consider the **Partition Problem**:
* **Input:** A multiset of positive integers $S = \{a_1, a_2, \dots, a_n\}$.
* **Question:** Can $S$ be partitioned into two disjoint subsets $S_1$ and $S_2$ such that the sum of the numbers in $S_1$ equals the sum of the numbers in $S_2$?

### Solution

**1. Proof of Membership in NP**
* **Witness (Certificate):** Given an instance of the Partition Problem with multiset $S = \{a_1, \dots, a_n\}$, a valid witness $t$ is a boolean array of length $n$, where $t[i] = 1$ indicates $a_i \in S_1$ and $t[i] = 0$ indicates $a_i \in S_2$. The size of this witness is exactly $n$ bits, which is strictly polynomial $O(n)$ with respect to the input size.
* **Verifier (Certifier):** An algorithm $C(s, t)$ iterates through the array $S$. It computes `sum1` by adding elements where $t[i] == 1$ and `sum2` by adding elements where $t[i] == 0$. Finally, it checks if `sum1 == sum2`. Since it performs $n$ additions, the verifier runs in polynomial time.
* **Conclusion:** Having a polynomial-size certificate and a polynomial-time verifier, the problem is in **NP**.

**2. NP-Hardness (Reduction from Subset Sum)**
Dimostriamo che $\text{Subset Sum} \le_p \text{Partition}$. 
Data un'istanza generica del Subset Sum composta da un insieme $W = \{w_1, \dots, w_m\}$ e un target $T$, calcoliamo la somma totale degli elementi $K = \sum_{i=1}^m w_i$.
Costruiamo un'istanza del Partition Problem creando un multinsieme $S$ che include tutti gli elementi di $W$, più due nuovi **elementi artificiali** $x$ e $y$ definiti come:
* $x = 2K - T$
* $y = K + T$

Quindi $S = W \cup \{x, y\}$. La costruzione avviene banalmente in tempo polinomiale. 
La somma totale del nuovo multinsieme $S$ è:
$$\text{Sum}(S) = K + (2K - T) + (K + T) = 4K$$
Il target richiesto per partizionare $S$ in due metà di ugual peso è esattamente $\frac{4K}{2} = 2K$.

**3. Dimostrazione (Se e Solo Se)**
* **($\implies$)** Supponiamo che esista una soluzione per il Subset Sum, ovvero un sottoinsieme $W' \subseteq W$ la cui somma è $T$. 
Costruiamo $S_1 = W' \cup \{x\}$ e $S_2 = (W \setminus W') \cup \{y\}$.
La somma di $S_1$ è: $T + x = T + (2K - T) = 2K$.
La somma di $S_2$ è: $(K - T) + y = (K - T) + (K + T) = 2K$.
Poiché le due metà sommano esattamente a $2K$, $S_1$ e $S_2$ formano una partizione valida per $S$.

* **($\impliedby$)** Supponiamo esista una soluzione per il Partition Problem, ovvero due insiemi $S_1, S_2$ disgiunti la cui somma è $2K$ ciascuno.
Sappiamo che la somma dei due elementi artificiali è $x + y = (2K - T) + (K + T) = 3K$. Poiché $3K > 2K$ (il target limite della singola partizione), $x$ e $y$ **non possono assolutamente trovarsi nello stesso sottoinsieme**. 
Senza perdita di generalità, assumiamo $x \in S_1$ e $y \in S_2$.
Affinché la somma di $S_1$ sia $2K$, gli altri elementi in $S_1$ (che per forza di cose provengono esclusivamente dall'insieme originale $W$) devono sommare a:
$2K - x = 2K - (2K - T) = T$.
Abbiamo quindi estratto un sottoinsieme degli elementi originali $W$ che somma esattamente a $T$, dimostrando l'esistenza di una soluzione valida per il Subset Sum.

---

## Problem 4: Weighted Vertex Cover (LP Rounding)

**Problem Statement:**
Consider the **Weighted Vertex Cover** problem. Given an undirected graph $G = (V, E)$ where each vertex $v \in V$ has a non-negative cost $w_v \ge 0$, find a vertex cover $C \subseteq V$ that minimizes the total cost $\sum_{v \in C} w_v$.
1. Formulate this problem as an **Integer Linear Program (ILP)**, clearly stating the decision variables, the objective function, and the constraints. Write its Linear Programming (LP) relaxation.
2. Design a deterministic rounding algorithm that takes an optimal fractional solution from the LP and produces a valid vertex cover.
3. Prove mathematically that your rounding algorithm guarantees a **2-approximation** factor with respect to the optimal integer solution.

### Solution

**1. ILP Formulation and LP Relaxation**
Definiamo una variabile decisionale $x_v$ per ciascun vertice $v \in V$:
$$x_v = \begin{cases} 1 & \text{se il vertice } v \text{ appartiene al cover } C \\ 0 & \text{altrimenti} \end{cases}$$

Formulazione Intera (ILP):
$$\min \sum_{v \in V} w_v x_v$$
soggetto a:
$$x_u + x_v \ge 1 \quad \forall (u, v) \in E$$
$$x_v \in \{0, 1\} \quad \forall v \in V$$

Il **rilassamento lineare (LP)** sostituisce il vincolo di discretezza con un vincolo di intervallo continuo:
$$x_v \ge 0 \quad \forall v \in V$$
*(Nota: il vincolo superiore $x_v \le 1$ è implicitamente soddisfatto all'ottimo poiché $w_v \ge 0$).*

**2. Deterministic Rounding Algorithm**
1. Risolviamo l'LP rilassato in tempo polinomiale per ottenere la soluzione ottima frazionaria $x^* = (x_1^*, \dots, x_n^*)$.
2. Definiamo la soluzione intera arrotondata $\hat{x}_v \in \{0, 1\}$ mediante soglia deterministica pari a $\frac{1}{2}$:
$$\hat{x}_v = \begin{cases} 1 & \text{se } x_v^* \ge \frac{1}{2} \\ 0 & \text{se } x_v^* < \frac{1}{2} \end{cases}$$
3. L'insieme dei vertici selezionati è $C = \{v \in V \mid \hat{x}_v = 1\}$.

**3. Proof of 2-Approximation**

* **Ammissibilità (Covering):**  
  Per ogni arco $(u, v) \in E$, la soluzione frazionaria dell'LP soddisfa il vincolo:
  $$x_u^* + x_v^* \ge 1$$
  Se la somma di due numeri non negativi è maggiore o uguale a 1, almeno uno di essi deve essere non inferiore alla loro media:
  $$\max(x_u^*, x_v^*) \ge \frac{1}{2}$$
  Dunque almeno una tra le due variabili soddisfa la soglia di arrotondamento e viene posta a $1$ ($\hat{x}_u = 1$ oppure $\hat{x}_v = 1$). Tutti gli archi del grafo risultano coperti, garantendo che $C$ sia un Vertex Cover ammissibile.

* **Fattore di Approssimazione:**  
  Per la regola di arrotondamento adottata, per ogni vertice $v \in V$ vale la disuguaglianza:
  $$\hat{x}_v \le 2 x_v^*$$
  Infatti:
  * Se $\hat{x}_v = 0$, $0 \le 2 x_v^*$ è sempre vera essendo $x_v^* \ge 0$.
  * Se $\hat{x}_v = 1$, allora $x_v^* \ge \frac{1}{2} \implies 2 x_v^* \ge 1 = \hat{x}_v$.

  Calcoliamo il costo totale della soluzione intera calcolata ($ALG$):
  $$ALG = \sum_{v \in V} w_v \hat{x}_v \le \sum_{v \in V} w_v (2 x_v^*) = 2 \sum_{v \in V} w_v x_v^* = 2 \cdot OPT_{LP}$$

  Poiché l'LP è un rilassamento dell'ILP originale, il valore ottimo frazionario costituisce un limite inferiore (*lower bound*) per il valore ottimo intero ($OPT_{LP} \le OPT$). Concludiamo che:
  $$ALG \le 2 \cdot OPT_{LP} \le 2 \cdot OPT$$
  L'algoritmo fornisce una **2-approssimazione**.
