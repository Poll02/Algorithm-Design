# Quaderno di Preparazione - Algorithm Design

## Esercizio 1: Rod Cutting (Capitolo 4, Problem 2)
### Problema
Data un'asta di lunghezza intera $L$ e una funzione di valore $v(l)$ per $0 \le l \le L$, trovare il partizionamento in pezzi interi che massimizza la somma dei valori dei pezzi ottenuti.

### 1. Definizione dello Stato
$OPT(i)$: massimo valore ricavabile dal taglio ottimale di un'asta di lunghezza $i$ ($0 \le i \le L$).

### 2. Equazione di Ricorrenza e Caso Base (Multiway Choice)
- **Caso Base:** $OPT(0) = 0$
- **Passo Ricorsivo:** per ogni $1 \le i \le L$:
  $$OPT(i) = \max_{1 \le j \le i} \{ v(j) + OPT(i - j) \}$$
  Dove $j$ rappresenta la lunghezza del primo pezzo staccato (venduto a $v(j)$) e $OPT(i-j)$ è il sottoproblema ottimo sulla barra rimanente.

### 3. Implementazione Bottom-Up (Estesa per ricostruzione tagli)
```python
def extended_rod_cutting(v, L):
    OPT = [0] * (L + 1)
    S = [0] * (L + 1)  # memorizza la dimensione del primo taglio ottimo
    
    for i in range(1, L + 1):
        q = float('-inf')
        for j in range(1, i + 1):
            if v[j] + OPT[i - j] > q:
                q = v[j] + OPT[i - j]
                S[i] = j
        OPT[i] = q
        
    return OPT, S

def get_optimal_cuts(S, L):
    cuts = []
    while L > 0:
        cuts.append(S[L])
        L = L - S[L]
    return cuts
```

### 4. Complessità
- **Tempo:** $O(L^2)$ (sommatoria $\sum_{j=1}^L j = \frac{L(L+1)}{2}$)
- **Spazio:** $O(L)$ per memorizzare `OPT` ed `S`.

---

## Esercizio 2: Maximum Contiguous Subarray Sum (Esame 16-01-2025, Problem 2)
### Problema
Dato un array $A[1 \dots n]$ di numeri interi relativi, determinare il sottoarray contiguo che massimizza la somma dei suoi elementi in tempo al più $O(n)$.

### 1. Definizione dello Stato
$OPT[i]$: somma del sottoarray contiguo ottimo che **termina obbligatoriamente all'indice $i$**.

### 2. Equazione di Ricorrenza e Caso Base (Binary Choice)
- **Caso Base:** $OPT[1] = A[1]$
- **Passo Ricorsivo:** per ogni $2 \le i \le n$:
  $$OPT[i] = \max \{ A[i], \, A[i] + OPT[i-1] \}$$
  - **Scelta 1 ($A[i]$):** ricominciare una nuova sequenza contigua a partire da $A[i]$ (se $OPT[i-1] \le 0$).
  - **Scelta 2 ($A[i] + OPT[i-1]$):** estendere il blocco contiguo precedente con l'elemento corrente.

La massima somma globale è: $\max_{1 \le i \le n} OPT[i]$.

### 3. Implementazione Completa e Ricostruzione del Sottoarray
```python
def max_subarray_with_indices(A):
    n = len(A)
    OPT = [0] * n
    
    OPT[0] = A[0]
    max_sum = A[0]
    
    best_start = 0
    best_end = 0
    curr_start = 0
    
    for i in range(1, n):
        if A[i] > A[i] + OPT[i - 1]:
            OPT[i] = A[i]
            curr_start = i  # Si azzera e ricomincia da qui
        else:
            OPT[i] = A[i] + OPT[i - 1]
            
        if OPT[i] > max_sum:
            max_sum = OPT[i]
            best_start = curr_start
            best_end = i
            
    return max_sum, best_start, best_end, A[best_start : best_end + 1]
```

### 4. Complessità
- **Tempo:** $O(n)$ — un singolo ciclo lineare con operazioni a tempo costante.
- **Spazio:** $O(n)$ mantenendo l'array `OPT` (riducibile a $O(1)$ tenendo traccia solo dell'ultimo valore).

---

## Esercizio 3: Max-Weight Independent Set on Line Graphs (Esame 12-02-2025, Problem 2 / Note Cap. 4 Problem 1)
### Problema
Dato un grafo a linea non orientato $G=(V, E)$ di $n$ vertici con pesi non negativi $w(v)$, trovare un insieme indipendente $S \subseteq V$ (nessun arco tra nodi scelti) che massimizzi il peso totale $\sum_{v \in S} w(v)$.

### 1. Definizione dello Stato
$OPT[i]$: peso massimo di un independent set formato considerando solo i primi $i$ nodi della linea ($v_1, \dots, v_i$).

### 2. Equazione di Ricorrenza e Casi Base (Binary Choice)
- **Casi Base:**
  - $OPT[0] = 0$
  - $OPT[1] = w_1$
- **Passo Ricorsivo:** per ogni $i \ge 2$:
  $$OPT[i] = \max \{ OPT[i-1], \, OPT[i-2] + w_i \}$$
  - **Scelta 1 ($OPT[i-1]$):** non includere il nodo $v_i$.
  - **Scelta 2 ($OPT[i-2] + w_i$):** includere il nodo $v_i$ (guadagnando $w_i$), escludere obbligatoriamente il nodo adiacente $v_{i-1}$ e ottimizzare sui primi $i-2$ nodi.

### 3. Implementazione Bottom-Up (Calcolo valore e Ricostruzione)
```python
def max_weight_independent_set(w):
    n = len(w)
    if n == 0:
        return 0, []
    if n == 1:
        return w[0], [0]
        
    OPT = [0] * (n + 1)
    OPT[0] = 0
    OPT[1] = w[0]
    
    for i in range(2, n + 1):
        OPT[i] = max(OPT[i - 1], OPT[i - 2] + w[i - 1])
        
    # Ricostruzione della soluzione (backtracking)
    selected_nodes = []
    i = n
    while i >= 1:
        if i == 1:
            selected_nodes.append(0)
            break
        if OPT[i - 2] + w[i - 1] > OPT[i - 1]:
            selected_nodes.append(i - 1)
            i -= 2
        else:
            i -= 1
            
    selected_nodes.reverse()
    return OPT[n], selected_nodes
```

### 4. Complessità
- **Tempo:** $O(n)$ — sia il ciclo di riempimento della tabella DP che la fase di backtracking visitano ogni elemento al più una volta.
- **Spazio:** $O(n)$ per memorizzare la tabella `OPT` (o $O(1)$ se si richiede solo il valore massimo).

---

## Esercizio 4: Longest Common Subsequence di Tre Stringhe (Esame 15-11-2024, Problem 1)
### Problema
Date tre sequenze di caratteri $x[1 \dots n]$, $y[1 \dots m]$ e $z[1 \dots l]$, trovare la lunghezza e la stringa effettiva della sottosequenza comune più lunga (LCS).

### 1. Definizione dello Stato
$OPT[i, j, k]$: lunghezza della più lunga sottosequenza comune tra i prefissi $x[1 \dots i]$, $y[1 \dots j]$ e $z[1 \dots k]$.

### 2. Equazione di Ricorrenza e Casi Base
- **Casi Base:** $OPT[i, j, k] = 0$ se $i = 0$, $j = 0$ oppure $k = 0$.
- **Passo Ricorsivo:**
  - Se $x[i] == y[j] == z[k]$:
    $$OPT[i, j, k] = 1 + OPT[i-1, j-1, k-1]$$
  - Altrimenti:
    $$OPT[i, j, k] = \max \{ OPT[i-1, j, k], \, OPT[i, j-1, k], \, OPT[i, j, k-1] \}$$

### 3. Implementazione Bottom-Up e Ricostruzione della Stringa
```python
def lcs_3_strings(x, y, z):
    n, m, l = len(x), len(y), len(z)
    
    # Tabella 3D (n+1) x (m+1) x (l+1)
    OPT = [[[0] * (l + 1) for _ in range(m + 1)] for _ in range(n + 1)]
    
    # Riempimento della tabella
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            for k in range(1, l + 1):
                if x[i - 1] == y[j - 1] == z[k - 1]:
                    OPT[i][j][k] = 1 + OPT[i - 1][j - 1][k - 1]
                else:
                    OPT[i][j][k] = max(
                        OPT[i - 1][j][k],
                        OPT[i][j - 1][k],
                        OPT[i][j][k - 1]
                    )
                    
    # Ricostruzione LCS (Backtracking)
    lcs_chars = []
    i, j, k = n, m, l
    while i > 0 and j > 0 and k > 0:
        if x[i - 1] == y[j - 1] == z[k - 1]:
            lcs_chars.append(x[i - 1])
            i -= 1
            j -= 1
            k -= 1
        elif OPT[i][j][k] == OPT[i - 1][j][k]:
            i -= 1
        elif OPT[i][j][k] == OPT[i][j - 1][k]:
            j -= 1
        else:
            k -= 1
            
    lcs_chars.reverse()
    return OPT[n][m][l], "".join(lcs_chars)
```

### 4. Complessità
- **Tempo:** $O(n \cdot m \cdot l)$ — la tabella ha $(n+1)(m+1)(l+1)$ celle e ciascuna richiede $O(1)$ operazioni. Il backtracking compie al più $n + m + l$ passi.
- **Spazio:** $O(n \cdot m \cdot l)$ per allocare la matrice tridimensionale.

## Esercizio: Optimal Bracket Sum to Minimize Numerical Overflow (Esame 26 Gennaio 2022)

### Testo del Problema
Dato un array $A = [A_1, A_2, \dots, A_n]$ di interi positivi e negativi, calcolare la somma di tutti gli elementi $\sum_{i=1}^n A_i$ parentesizzando i termini in modo da **minimizzare il valore massimo** raggiunto durante l'intera sequenza di calcoli intermedi.

1. Dare la formulazione ricorsiva usata in Programmazione Dinamica ($OPT[i, j]$ calcolato da $OPT[i, k]$ e $OPT[k+1, j]$).
2. Dimostrare la correttezza della formulazione ricorsiva.
3. Argomentare l'implementazione e la complessità temporale.

---

### 1. Definizione dello Stato e Formulazione Ricorsiva

#### Intuizione
* **Invarianza della somma:** Per la proprietà associativa, la somma finale di un intervallo $A[i \dots j]$ è fissa e non dipende dalle parentesi:
  $$S[i, j] = \sum_{m=i}^j A_m$$
* **Ultima operazione di unione:** L'albero di calcolo divide l'intervallo $[i, j]$ in un blocco sinistro $[i, k]$ e un blocco destro $[k+1, j]$ con $i \le k < j$. L'operazione finale esegue l'addizione $S[i, k] + S[k+1, j] = S[i, j]$.
* **Picco generato:** Il valore massimo toccato scegliendo un taglio $k$ è il massimo tra il picco a sinistra, il picco a destra e il risultato dell'unione:
  $$\max \{ OPT[i, k], \, OPT[k+1, j], \, S[i, j] \}$$

#### Definizione dello Stato
Sia $OPT[i, j]$ il minimo valore di picco necessario per calcolare la somma del sottoarray $A[i \dots j]$.

#### Equazione di Ricorrenza
* **Caso base ($i = j$):**
  $$OPT[i, i] = A_i \quad \forall i \in \{1, \dots, n\}$$
* **Passo ricorsivo ($i < j$):**
  $$OPT[i, j] = \min_{i \le k < j} \max \{ OPT[i, k], \, OPT[k+1, j], \, S[i, j] \}$$


---

### 2. Dimostrazione di Correttezza (Optimal Substructure)

**Tesi:** Sia $T$ l'albero di parentesizzazione ottimo per $[i, j]$ con ultima unione in $k$. I sotto-alberi $T_L$ (per $[i, k]$) e $T_R$ (per $[k+1, j]$) devono essere alberi ottimi per i rispettivi sotto-intervalli.

**Dimostrazione (per assurdo / Cut-and-Paste):**
Supponiamo che $T_L$ non sia ottimo per $[i, k]$. Allora esiste un albero alternativo $T'_L$ tale che:
$$P(T'_L) < P(T_L)$$
Poiché la somma algebrica finale è invariante ($S(T'_L) = S(T_L) = S[i, k]$), possiamo sostituire $T_L$ con $T'_L$ dentro $T$. Il picco del nuovo albero $T'$ risulterà:
$$P(T') = \max \{ P(T'_L), \, P(T_R), \, S[i, j] \} \le \max \{ P(T_L), \, P(T_R), \, S[i, j] \} = P(T)$$
Il picco globale non peggiora e diventa strettamente inferiore qualora il massimo fosse determinato da $P(T_L)$. L'algoritmo prova tutti i possibili punti di split $k \in \{i, \dots, j-1\}$, garantendo di trovare la parentesizzazione globalmente ottimale.

---

### 3. Pseudocodice e Ricostruzione della Soluzione

```python
def optimal_bracket_sum(A):
    n = len(A)
    # Precalcolo somme prefisse per avere S[i, j] in O(1)
    P = [0] * (n + 1)
    for m in range(1, n + 1):
        P[m] = P[m - 1] + A[m - 1]

    def S(i, j):
        return P[j] - P[i - 1]

    # Tabella DP e tabella per memorizzare i tagli ottimi
    OPT = [[0] * (n + 1) for _ in range(n + 1)]
    split = [[0] * (n + 1) for _ in range(n + 1)]

    # Caso base: blocchi di lunghezza 1
    for i in range(1, n + 1):
        OPT[i][i] = A[i - 1]

    # Riempimento bottom-up per lunghezze crescenti L
    for L in range(2, n + 1):
        for i in range(1, n - L + 2):
            j = i + L - 1
            best_val = float('inf')
            best_k = i
            sum_ij = S(i, j)

            for k in range(i, j):
                peak_k = max(OPT[i][k], OPT[k + 1][j], sum_ij)
                if peak_k < best_val:
                    best_val = peak_k
                    best_k = k

            OPT[i][j] = best_val
            split[i][j] = best_k

    return OPT[1][n], split
```

---

### 4. Analisi della Complessità

* **Complessità Temporale:**
  * Precalcolo delle somme prefisse: $O(n)$.
  * Ci sono $\Theta(n^2)$ sotto-intervalli $(i, j)$.
  * Per un intervallo di lunghezza $L$, si testano $L - 1$ possibili valori di $k$. Ciascun test richiede $O(1)$ operazioni grazie al precalcolo di $S[i, j]$.
  * Tempo totale:
    $$\sum_{L=2}^n (n - L + 1)(L - 1) = \Theta(n^3)$$
* **Complessità Spaziale:** $\Theta(n^2)$ per memorizzare le matrici `OPT` e `split`.