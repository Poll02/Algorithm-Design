# Algorithm Design - Mock Exam 2

## Problem 1: Professor-Group Assignment (Max-Flow)

**Problem Statement:**
A university has $n$ student project groups $g_1, g_2, \dots, g_n$ and $m$ professors $p_1, p_2, \dots, p_m$. Each group $g_i$ has a predetermined list of professors $P_i \subseteq \{p_1, \dots, p_m\}$ qualified to evaluate their work. 

To ensure fair grading and workload balance:
* Each group $g_i$ must be evaluated by exactly $k$ different qualified professors.
* No professor $p_j$ can evaluate more than $c_j \in \mathbb{N}^+$ total groups.
* A professor cannot evaluate the same group more than once.

1. Model the problem of deciding whether a valid assignment exists as a **Max-Flow** problem. Detail the construction of the flow network: nodes, directed edges, and capacities.
2. State the condition on the maximum flow value that determines if a valid assignment exists, and prove the correctness of the reduction in both directions ($\implies$ and $\impliedby$).
3. Analyze the computational complexity of determining the existence of a valid assignment using an appropriate max-flow algorithm.

---

### Solution

**1. Algorithm Design (Graph Construction)**
Costruiamo una rete di flusso orientata $G = (V, E)$ nel seguente modo:
* **Nodi:** 
  * Un nodo sorgente $S$.
  * Un nodo pozzo $T$.
  * Un nodo per ogni gruppo di studenti $g_i$, per $i \in \{1, \dots, n\}$.
  * Un nodo per ogni professore $p_j$, per $j \in \{1, \dots, m\}$.
* **Archi dalla Sorgente ai Gruppi:** 
  * Per ogni gruppo $g_i$, aggiungiamo un arco orientato $(S, g_i)$ con capacità:
    $$c(S, g_i) = k$$
    Questo assicura che ogni gruppo riceva esattamente $k$ valutatori.
* **Archi tra Gruppi e Professori Qualificati:**
  * Per ogni gruppo $g_i$ e per ciascun professore qualificato $p_j \in P_i$, aggiungiamo un arco orientato $(g_i, p_j)$ con capacità unitaria:
    $$c(g_i, p_j) = 1$$
    La capacità unitaria garantisce che un docente possa valutare lo stesso gruppo al massimo una volta.
* **Archi dai Professori al Pozzo:**
  * Per ogni professore $p_j$, aggiungiamo un arco orientato $(p_j, T)$ con capacità pari al suo limite di carico:
    $$c(p_j, T) = c_j$$

**Condizione di Esistenza:**
Un'assegnazione valida esiste se e solo se il valore del flusso massimo $F^*$ nella rete soddisfa:
$$F^* = k \cdot n$$

---

**2. Proof of Correctness**

* **$(\implies)$ Se esiste un'assegnazione valida, allora il flusso massimo è $F^* = k \cdot n$:**
  Supponiamo esista un'assegnazione valida che rispetta tutti i vincoli. Costruiamo il flusso $f$ come segue:
  * Per ogni gruppo $g_i$, poniamo $f(S, g_i) = k$.
  * Per ogni coppia $(g_i, p_j)$, poniamo $f(g_i, p_j) = 1$ se il professore $p_j$ valuta il gruppo $g_i$, e $0$ altrimenti. Poiché ogni gruppo ha esattamente $k$ valutatori distinti, escono esattamente $k$ unità di flusso da ogni nodo $g_i$, rispettando la conservazione del flusso. Inoltre, la capacità unitaria $c(g_i, p_j) = 1$ non viene mai violata.
  * Per ogni professore $p_j$, poniamo $f(p_j, T)$ pari al numero totale di gruppi che valuta. Poiché l'assegnazione è valida, tale numero è $\le c_j = c(p_j, T)$, rispettando la capacità dell'arco.
  * La conservazione del flusso è soddisfatta su tutti i nodi intermedi.
  * Il valore totale del flusso uscente dalla sorgente è:
    $$\sum_{i=1}^n f(S, g_i) = \sum_{i=1}^n k = k \cdot n$$
  Poiché la capacità totale uscente da $S$ è $\sum c(S, g_i) = k \cdot n$, questo flusso è anche massimo ($F^* = k \cdot n$).

* **$(\impliedby)$ Se il flusso massimo è $F^* = k \cdot n$, allora esiste un'assegnazione valida:**
  Supponiamo che il valore del flusso massimo sia $F^* = k \cdot n$.
  * **Interezza:** Tutte le capacità della rete ($k$, $1$, $c_j$) sono intere. Per l'Integrality Theorem, esiste un flusso massimo $f$ a valori interi su tutti gli archi.
  * **Saturazione:** La capacità di taglio $( \{S\}, V \setminus \{S\} )$ è $\sum_{i=1}^n c(S, g_i) = k \cdot n$. Affinché il flusso totale raggiunga $k \cdot n$, tutti gli archi $(S, g_i)$ devono essere saturi: $f(S, g_i) = k$ per ogni $i \in \{1, \dots, n\}$.
  * **Assegnazione:** Poiché gli archi intermedi hanno capacità $1$ e il flusso è intero, deve valere $f(g_i, p_j) \in \{0, 1\}$. Assegniamo il professore $p_j$ al gruppo $g_i$ se e solo se $f(g_i, p_j) = 1$.
  * **Verifica dei vincoli:**
    1. Per ogni gruppo $g_i$, la conservazione impone che $\sum_{p_j \in P_i} f(g_i, p_j) = f(S, g_i) = k$. Poiché i flussi sono in $\{0, 1\}$, ogni gruppo riceve esattamente $k$ docenti qualificati distinti.
    2. La presenza di un solo arco orientato con capacità $1$ per coppia previene duplicati.
    3. Per ogni docente $p_j$, il numero di gruppi assegnati corrisponde al flusso totale in ingresso $\sum_{g_i} f(g_i, p_j) = f(p_j, T) \le c(p_j, T) = c_j$, rispettando il tetto massimo di valutazioni.

---

**3. Computational Complexity**

* **Dimensioni della Rete:**
  * Vertici: $|V| = n + m + 2 = O(n + m)$ (gruppi, professori, sorgente e pozzo).
  * Archi: $|E| = n + m + \sum_{i=1}^n |P_i| \le n + m + n \cdot m = O(n \cdot m)$ nel caso peggiore in cui ogni docente sia qualificato per ogni gruppo.
* **Algoritmo di Risoluzione:**
  Utilizzando l'algoritmo di **Edmonds-Karp** (implementazione BFS di Ford-Fulkerson):
  * Complessità canonica: $O(|V| \cdot |E|^2)$.
  * Sostituendo le dimensioni ricavate:
    $$O((n + m) \cdot |E|^2)$$
    Nel caso peggiore in cui $|E| = O(n \cdot m)$, la complessità temporale è:
    $$O((n + m) \cdot (n \cdot m)^2)$$
  *(In alternativa, l'algoritmo di Dinic con capacità residue intere o di rete unitaria permette di risolverlo in $O(|E| \sqrt{|V|})$).*
* **Complessità Spaziale:**
  $O(|V| + |E|) = O(n + m + |E|)$ per memorizzare il grafo tramite liste di adiacenza e il grafo residuo.
