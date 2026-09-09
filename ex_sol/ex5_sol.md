# Practical Session 5 - NP-Completeness (Solutions)

## Problem 1
**Problem Statement:**
Consider the following decision problem. Clique: Given an undirected graph $G=(V,E)$ and an integer $k$, does there exist a subset of vertices $S \subseteq V$ of size $|S| \ge k$ that induces a complete subgraph i.e. a clique?
Prove that Clique is NP-complete via a reduction from 3-SAT.

**Solution:**
Per dimostrare che Clique è NP-completo, dobbiamo dimostrare due proprietà:

1. **Appartenenza a NP (Clique $\in$ NP):**
   Un certificato (witness) per questo problema è un sottoinsieme di nodi $t \subseteq V$. Il verificatore algoritmico esegue due controlli:
   * Verifica la cardinalità: controlla che $|t| \ge k$. Richiede tempo asintotico $O(n)$.
   * Verifica la completezza: per ogni coppia di nodi $u, v \in t$, controlla che l'arco $(u, v)$ appartenga ad $E$. Ci sono al massimo $O(n^2)$ coppie da controllare.
   Entrambi i controlli avvengono in tempo polinomiale, quindi il problema appartiene a NP.

2. **NP-Hardness (Riduzione 3-SAT $\le_p$ Clique):**
   Data una formula 3-SAT con $m$ clausole, costruiamo un'istanza di Clique nel seguente modo:
   * **Nodi:** Creiamo un nodo per ogni occorrenza di un letterale nella formula (totale $3m$ nodi).
   * **Archi:** Inseriamo un arco tra il nodo $u$ e il nodo $v$ se e solo se sono rispettate entrambe queste condizioni:
     * *Condizione strutturale:* I nodi appartengono a clausole diverse.
     * *Condizione logica:* I nodi non rappresentano letterali opposti (es. vietato collegare $x_1$ e $
eg x_1$).
   * **Target:** Impostiamo $k = m$ (il numero di clausole).
   
   *Dimostrazione costruttiva:* Una Clique di dimensione $m$ impone la scelta di esattamente un nodo (letterale) da ciascuna delle $m$ clausole (grazie alla regola strutturale), e garantisce che l'assegnazione sia logicamente valida e senza contraddizioni (grazie alla regola logica), soddisfacendo così l'intera formula.

---

## Problem 2
**Problem Statement:**
Prove that Clique $\le_{p}$ IndependentSet.

**Solution:**
La riduzione sfrutta la complementarità geometrica tra i due problemi:
* Una Clique è un insieme di nodi in cui *tutti* gli archi possibili sono presenti.
* Un Independent Set è un insieme di nodi in cui *nessun* arco è presente.

L'algoritmo di riduzione (eseguibile in tempo polinomiale) è il seguente:
1. L'input per Clique è un'istanza $(G, k)$, dove $G=(V,E)$.
2. Costruiamo il **Grafo Complemento** $\overline{G} = (V, \overline{E})$. Il grafo mantiene gli stessi nodi, ma inverte tutti gli archi: un arco $(u, v)$ appartiene a $\overline{G}$ se e solo se **non** appartiene a $G$.
3. Restituiamo in output l'istanza tradotta per Independent Set: $(\overline{G}, k)$.

*Dimostrazione costruttiva:* Un sottoinsieme di nodi $S$ di dimensione $k$ forma una Clique nel grafo originale $G$ se e solo se quello stesso sottoinsieme $S$ forma un Independent Set nel grafo complemento $\overline{G}$. Questo perché, rimuovendo da $G$ gli archi che rendevano quei nodi completamente connessi, gli stessi nodi rimangono totalmente scollegati in $\overline{G}$.

---

## Problem 3
**Problem Statement:**
Consider the following decision problem. 4D-Matching: Given $X, Y, Z, W$ disjoint sets of cardinality $n$ and a set of quadruples $Q \subseteq X 	imes Y 	imes Z 	imes W$, is there a subset $Q' \subseteq Q$ such that each element of $X \cup Y \cup Z \cup W$ appears exactly in one tuple in $Q'$?
Prove that 4D-Matching is NP-complete.

**Solution:**
Per dimostrare che 4D-Matching è NP-completo, dobbiamo dimostrare due proprietà:

1. **Appartenenza a NP (4D-Matching $\in$ NP):**
   Il certificato (witness) è un sottoinsieme $t \subseteq Q$. Affinché sia valido, la sua cardinalità deve essere esattamente $n$ (poiché ci sono in totale $4n$ elementi da coprire e ogni tupla ne copre $4$). Il verificatore controlla che ogni elemento dell'unione $X \cup Y \cup Z \cup W$ compaia in esattamente una tupla di $t$. Poiché il controllo avviene su un massimo di $n$ tuple, l'algoritmo termina in tempo polinomiale.

2. **NP-Hardness (Riduzione 3D-Matching $\le_p$ 4D-Matching):**
   Fornita in input un'istanza del 3D-Matching (tre insiemi $X, Y, Z$ di dimensione $n$ e un insieme di triplette $T$), costruiamo l'istanza del 4D-Matching in questo modo:
   * **Insiemi:** Manteniamo identici $X, Y, Z$ e creiamo un nuovo insieme fantasma $W = \{w_1, w_2, \dots, w_n\}$ di cardinalità $n$.
   * **Quadruple ($Q$):** Per ogni tripletta $t = (x, y, z) \in T$, generiamo $n$ nuove quadruple unendo $t$ con tutti i possibili elementi di $W$. In termini formali: $Q = T 	imes W$. La costruzione richiede $O(|T| \cdot n)$ operazioni, quindi avviene in tempo polinomiale.
   
   *Dimostrazione costruttiva:* 
   * **($\implies$)** Se esiste un 3D-Matching perfetto, esso contiene $n$ triplette che coprono interamente $X, Y$ e $Z$. Per formare un 4D-Matching, basterà assegnare a ciascuna di queste $n$ triplette un elemento distinto di $W$ (es. la prima tripletta con $w_1$, la seconda con $w_2$, ecc.). Poiché le combinazioni esistono in $Q$ per costruzione, avremo ottenuto un 4D-Matching perfetto.
   * **($\impliedby$)** Se esiste un 4D-Matching perfetto $Q' \subseteq Q$, esso conterrà esattamente $n$ quadruple che coprono tutti gli insiemi senza sovrapposizioni. Se ignoriamo (cancelliamo) l'ultima coordinata (quella dell'insieme $W$) da ciascuna di queste quadruple, otterremo $n$ triplette originarie di $T$ che coprono perfettamente $X, Y$ e $Z$, rivelando la soluzione del 3D-Matching.
