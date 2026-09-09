# Capitolo 7 — NP e Intrattabilità Computazionale
### Schema di studio completo

---

## PARTE 1 — INQUADRAMENTO

### 1.1 Design patterns (problemi "facili")

| Pattern | Esempio | Complessità |
|---|---|---|
| Greedy | Interval scheduling | O(n log n) |
| Divide-and-conquer | FFT | O(n log n) |
| Programmazione dinamica | Edit distance | O(n²) |
| Dualità | Bipartite matching | O(n³) |
| Riduzioni, local search, randomizzazione | — | — |

### 1.2 Anti-patterns (problemi "difficili")

| Anti-pattern | Significato |
|---|---|
| **NP-completezza** | Un algoritmo O(n^k) è molto improbabile |
| **PSPACE-completezza** | È improbabile persino un *certificatore* O(n^k) |
| **Indecidibilità** | Nessun algoritmo esiste, punto |

> Nota sulla gerarchia: NP-completo = "difficile da **risolvere**". PSPACE-completo = "difficile persino da **verificare**" (es. giochi a due giocatori, quantificatori alternati). Indecidibile = "impossibile" (es. halting problem).

### 1.3 La tabella "sì / probabilmente no"

Coppie di problemi quasi identici nell'enunciato, ma su lati opposti della barriera:

| Risolvibile in poly-time | Probabilmente NO |
|---|---|
| Shortest path | Longest path |
| Matching | 3D-matching |
| Min cut | Max cut |
| 2-SAT | 3-SAT |
| Planar 4-color | Planar 3-color |
| Bipartite vertex cover | Vertex cover |
| Primality testing | Factoring |

**Morale:** una modifica minima dell'enunciato (2 → 3 letterali, grafo bipartito → grafo generale) può far saltare da P a NP-completo. Non fidarsi dell'intuizione.

---

## PARTE 2 — RIDUZIONI POLINOMIALI (§8.1)

### 2.1 Definizione 24 — Riduzione polinomiale

> **X ≤P Y** ("X si riduce polinomialmente a Y") se **ogni istanza di X** può essere risolta usando:
> - un numero **polinomiale** di passi computazionali standard, **più**
> - un numero **polinomiale** di chiamate a un **oracolo** che risolve Y.

**Nota importante:** le istanze di Y costruite devono avere **dimensione polinomiale** — le paghiamo per scriverle.

**Lettura intuitiva:** `X ≤P Y` significa **"Y è almeno difficile quanto X"** (a meno di fattori polinomiali). Il simbolo ≤ è orientato come la difficoltà: la cosa *piccola* (facile) sta a sinistra.

### 2.2 A cosa servono le riduzioni

| Uso | Enunciato | Direzione del ragionamento |
|---|---|---|
| **Progettare algoritmi** | Se X ≤P Y e Y ∈ P, allora X ∈ P | Positivo: eredito la facilità da destra |
| **Provare intrattabilità** | Se X ≤P Y e X non ha algoritmo poly, allora nemmeno Y | Contronominale: eredito la difficoltà da sinistra |
| **Provare equivalenza** | Se X ≤P Y **e** Y ≤P X, scrivo X ≡P Y | Stessa difficoltà |

> ⚠️ **Errore classico d'esame.** Per dimostrare che *Y è difficile* devo ridurre **DA** un problema noto difficile **A** Y (`X_noto ≤P Y`). Se faccio `Y ≤P X_noto` non ho dimostrato niente sulla difficoltà di Y — ho solo detto che Y è *al massimo* difficile quanto X.

### 2.3 Le tre strategie di riduzione

1. **Equivalenza semplice** → i due problemi sono lo stesso problema riformulato. *(es. VERTEX-COVER ≡P INDEPENDENT-SET)*
2. **Caso speciale → caso generale** → X è un caso particolare di Y. *(es. VERTEX-COVER ≤P SET-COVER)*
3. **Encoding con gadget** → costruisco pezzi di grafo/formula che "simulano" la logica di X. *(es. 3-SAT ≤P INDEPENDENT-SET)*

### 2.4 Teorema 27 — Transitività

> Se X ≤P Y e Y ≤P Z, allora X ≤P Z.

Questo permette di **concatenare** le riduzioni, ed è il motore di tutta la teoria:

```
3-SAT ≤P INDEPENDENT-SET ≤P VERTEX-COVER ≤P SET-COVER
```

---

## PARTE 3 — LE CLASSI P, NP, EXP (§8.3)

### 3.1 Problemi di decisione

Un **problema di decisione** X è un **insieme di stringhe**.
Un algoritmo A **risolve** X se: `A(s) = yes ⟺ s ∈ X`.

**Tempo polinomiale:** A gira in poly-time se per ogni input s, A(s) termina in ≤ p(|s|) passi, con p polinomio e |s| = **lunghezza** della stringa.

> ⚠️ Polinomiale nella **lunghezza dell'input**, non nel suo **valore**. Un algoritmo che testa la primalità di *n* provando tutti i divisori fino a √n è esponenziale in |s| ≈ log₂ n.

### 3.2 Definizione 29 — Classe P

> **P** = insieme dei problemi di decisione risolvibili da un algoritmo in tempo polinomiale.

| Problema | Domanda | Algoritmo |
|---|---|---|
| MULTIPLE | x è multiplo di y? | Divisione delle elementari |
| RELPRIME | x e y sono coprimi? | Algoritmo di Euclide |
| PRIMES | x è primo? | Algoritmo AKS |
| EDIT-DISTANCE | La edit distance tra x e y è < 5? | Programmazione dinamica |
| LSOLVE | Ax = b ha soluzione? | Eliminazione di Gauss-Edmonds |

**Esempio PRIMES:**
- Input: s = 53, cioè la stringa binaria `110101`, |s| = 6 bit.
- Output: `yes`.
- Tempo: O(|s|⁸) = O(6⁸), polinomiale **nella lunghezza** (6), non nel valore (53).

### 3.3 Definizione 30 — Certificatore

**Intuizione:** invece di *trovare* una soluzione, riesco a *verificarla* in fretta?

> Un algoritmo **C(s, t)** è un **certificatore** per X se per ogni stringa s:
> `s ∈ X ⟺ ∃ t tale che C(s, t) = yes`

La stringa **t** si chiama **certificato** (o *witness*, o *testimone*).

### 3.4 Definizione 31 — Classe NP

> **NP** = insieme dei problemi di decisione per cui esiste un certificatore C(s,t) **poly-time**, con **|t| ≤ p(|s|)** per qualche polinomio p.

NP = **N**ondeterministic **P**olynomial-time.

> ⚠️ NP **non** vuol dire "non polinomiale". Vuol dire "verificabile in tempo polinomiale".
> Entrambi i vincoli sono necessari: il certificato deve essere **corto** (poly) e la verifica **veloce** (poly).

#### Esempio A — COMPOSITES ∈ NP
- **Problema:** dato s, è composto (non primo)?
- **Certificato t:** un fattore non banale di s, cioè 1 < t < s con t | s.
- **Certificatore:**
```c
boolean C(s, t) {
    if (t <= 1 or t >= s) return false;
    if (s mod t == 0)     return true;
    else                  return false;
}
```
- **Istanza concreta:** s = 437.669 → certificato t = 541 (oppure 809).
  Verifica: 437.669 = 541 × 809 ✓
- **Perché |t| ≤ |s|?** Ogni fattore di s è ≤ s, quindi ha al più lo stesso numero di bit.

#### Esempio B — SAT ∈ NP
- **Problema:** data una formula CNF Φ, esiste un assegnamento che la soddisfa?
- **Certificato:** un assegnamento di verità alle n variabili booleane.
- **Certificatore:** controlla che ogni clausola contenga almeno un letterale `true`.
- **Istanza concreta:**
  Φ = (x₁ ∨ x̄₂ ∨ x̄₃) ∧ (x̄₁ ∨ x₂ ∨ x̄₃) ∧ (x̄₁ ∨ x̄₂ ∨ x₄) ∧ (x₁ ∨ x̄₃ ∨ x̄₄)
  Certificato: x₁=1, x₂=1, x₃=0, x₄=1.
  Clausola 1: 1 ∨ 0 ∨ 1 = 1 ✓ — Clausola 2: 0 ∨ 1 ∨ 1 = 1 ✓ — (tutte soddisfatte)

#### Esempio C — HAM-CYCLE ∈ NP
- **Problema:** dato G = (V,E) non diretto, esiste un ciclo **hamiltoniano** (ciclo semplice che visita ogni nodo esattamente una volta)?
- **Certificato:** una permutazione di tutti gli n nodi.
- **Certificatore:**
  1. La permutazione contiene ogni nodo di V esattamente una volta?
  2. Esiste un arco tra ogni coppia di nodi adiacenti nella permutazione?
  3. Esiste un arco tra l'ultimo e il primo nodo (per chiudere il ciclo)?
- Il certificatore gira in **O(n)** → polinomiale.

### 3.5 Definizione 32 — Classe EXP

> **EXP** = insieme dei problemi di decisione risolvibili da un algoritmo in tempo **esponenziale**.

### 3.6 Teorema 28 — Catena di inclusioni

> **P ⊆ NP ⊆ EXP**

**Prova di P ⊆ NP:**
Se A(s) risolve X in poly-time, uso il certificato **vuoto** ε e definisco C(s, ε) = A(s).
C è poly-time e |ε| ≤ p(|s|) banalmente ⟹ X ∈ NP. □

**Prova di NP ⊆ EXP:**
Sia X ∈ NP con certificatore poly-time C(s,t).
Per risolvere l'input s: eseguo C(s,t) su **tutte** le stringhe t con |t| ≤ p(|s|).
Ce ne sono al più **2^p(|s|)** — esponenzialmente tante, ma **finite**.
Restituisco `yes` se almeno una chiamata dà `yes`. È un algoritmo esponenziale. □

### 3.7 La domanda P vs NP

La più grande domanda aperta dell'informatica: **P = NP?**

| Se **P = NP** | Se **P ≠ NP** |
|---|---|
| Esistono algoritmi efficienti per 3-SAT, TSP, 3-COLOR, FACTOR… | Nessun algoritmo efficiente per i problemi NP-completi |
| Romperebbe RSA e potenzialmente farebbe collassare l'economia | Il mondo resta com'è |
| EXP ⊃ P = NP (P e NP collassano insieme) | EXP ⊃ NP ⊃ P (tre classi distinte) |

**Consenso:** la maggior parte dei ricercatori crede che **P ≠ NP**.

---

## PARTE 4 — NP-COMPLETEZZA (§8.4)

### 4.1 Due tipi di riduzione

| Tipo | Definizione | Chiamate all'oracolo |
|---|---|---|
| **Riduzione di Cook** (≤P) | Risolvi X usando un numero polinomiale di chiamate a un oracolo per Y | Polinomialmente molte, ovunque nel calcolo |
| **Trasformazione di Karp** | Dato x input di X, costruisci y in poly-time tale che `x ∈ X ⟺ y ∈ Y` | **Una sola**, alla fine |

Quasi tutte le riduzioni che si vedono in pratica sono **trasformazioni di Karp**. È **aperto** se le due definizioni siano equivalenti.

### 4.2 Definizione 33 — NP-completo

> Un problema **Y è NP-completo** se:
> 1. **Y ∈ NP**, e
> 2. **per ogni X ∈ NP: X ≤P Y**.
>
> Cioè: *"Y può risolvere qualsiasi problema in NP"*.

Y è quindi uno dei problemi **più difficili** di NP: se sai risolvere lui, sai risolvere tutti.

### 4.3 Teorema 29 — Teorema centrale

> Sia Y NP-completo. Allora: **Y è risolvibile in poly-time ⟺ P = NP**.

**Prova (⇐):** se P = NP, siccome Y ∈ NP allora Y ∈ P, quindi Y è risolvibile in poly-time.

**Prova (⇒):** supponiamo Y risolvibile in poly-time. Sia X un problema qualsiasi in NP.
Siccome X ≤P Y (definizione di NP-completo), posso risolvere X in poly-time ⟹ **NP ⊆ P**.
Sappiamo già che P ⊆ NP, quindi **P = NP**. □

**Conseguenza pratica:** dimostrare che un problema è NP-completo è la giustificazione formale per smettere di cercare un algoritmo esatto efficiente e passare ad approssimazioni, euristiche o casi speciali.

### 4.4 Ricetta per provare che Y è NP-completo

1. **Mostra Y ∈ NP** → descrivi un certificato e un certificatore poly-time.
2. **Scegli un problema X già noto NP-completo** (es. CIRCUIT-SAT, 3-SAT).
3. **Prova X ≤P Y** → costruisci una trasformazione poly-time (+ dimostra `x ∈ X ⟺ y ∈ Y`, entrambe le direzioni!).

**Giustificazione:** se W ∈ NP, allora W ≤P X ≤P Y per transitività, quindi Y è NP-completo.

> Il punto 3 richiede sempre **due direzioni** della dimostrazione: (⇒) se x è sì allora y è sì; (⇐) se y è sì allora x è sì. Dimenticare la (⇐) è l'errore più frequente.

---

## PARTE 5 — CATALOGO DEI PROBLEMI NP-COMPLETI

### 5.0 La catena dei domino

```
                        CIRCUIT-SAT            ← Cook-Levin (dalla definizione)
                             ↓
                          3-SAT
        ┌────────────┬────────┴──────┬──────────────┐
        ↓            ↓               ↓              ↓
 INDEPENDENT-SET  DIR-HAM-CYCLE  GRAPH 3-COLOR  SUBSET-SUM
        ↓            ↓               ↓              ↓
   VERTEX-COVER   HAM-CYCLE     PLANAR 3-COLOR  SCHEDULING
        ↓            ↓
    SET-COVER       TSP
```

Una volta stabilito CIRCUIT-SAT, tutti gli altri **cadono come tessere del domino**.

> **Problemi in NP né in P né NP-completi (per quanto si sappia):** FACTORING, isomorfismo di grafi, equilibrio di Nash. Sono le eccezioni notevoli: quasi tutti gli altri problemi NP naturali sono o in P o NP-completi.

---

### 5.1 CIRCUIT-SAT — il primo problema NP-completo

**Definizione 34.** Dato un circuito combinatorio costruito con porte **AND, OR, NOT**, dove alcuni input sono **fissati** (hard-coded a 0 o 1) e altri sono **liberi**: possiamo settare gli input liberi in modo che l'output del circuito sia **1**?

**Esempio:** possiamo scegliere x₂, x₃, x₄ ∈ {0,1} in modo che l'output sia 1?
Risposta: x₂ = 1, x₃ = 0, x₄ = 1 produce output 1 ✓

**Teorema 30 (Cook-Levin, 1971/1973): CIRCUIT-SAT è NP-completo.**

**Sketch di dimostrazione:**

*(a) CIRCUIT-SAT ∈ NP:*
- Certificato = un settaggio degli input liberi.
- Certificatore = simula il circuito, O(n).

*(b) Ogni problema X ∈ NP si riduce a CIRCUIT-SAT.* Dato X con certificatore C(s,t):
1. Vedi C(s,t) come un algoritmo su **|s| + p(|s|) bit di input** (s fissato, t libero).
2. Siccome C gira in poly-time, può essere **convertito in un circuito K di dimensione polinomiale** (è il punto cruciale: ogni computazione poly-time è simulabile da un circuito poly-size).
3. **Hard-codifica** i primi |s| bit con l'input s.
4. I restanti p(|s|) bit rappresentano il certificato t → sono gli **input liberi**.
5. Quindi: `K è soddisfacibile ⟺ C(s,t) = yes per qualche t ⟺ s ∈ X`.

**Esempio concreto — il certificatore di INDEPENDENT-SET come circuito.**
Per G = (V,E) con n = 3 (nodi u, v, w) e k = 2:
- Il circuito ha **n = 3 input liberi**: "quali nodi includo nell'insieme?"
- Gli **input hard-coded** (C(n,2) = 3 bit) codificano la struttura del grafo: quali archi esistono.
- Il circuito calcola in parallelo: *(i)* "entrambi gli estremi di qualche arco sono stati scelti?" (negato) **AND** *(ii)* "l'insieme ha dimensione ≥ 2?"
- Output = 1 **se e solo se** i nodi scelti formano un independent set di dimensione 2.

---

### 5.2 3-SAT

**Definizione 28 — SAT e 3-SAT**
- Un **letterale** è una variabile booleana xᵢ o la sua negazione x̄ᵢ.
- Una **clausola** è una disgiunzione (OR) di letterali: Cⱼ = x₁ ∨ x̄₂ ∨ x₃.
- Una formula in **Forma Normale Congiuntiva (CNF)** è una congiunzione (AND) di clausole: Φ = C₁ ∧ C₂ ∧ C₃ ∧ C₄.
- **SAT:** data Φ in CNF, esiste un assegnamento di verità che la soddisfa?
- **3-SAT:** SAT in cui **ogni clausola ha esattamente 3 letterali** (ciascuno relativo a una variabile diversa).

**Esempio:** (x̄₁ ∨ x₂ ∨ x₃) ∧ (x₁ ∨ x̄₂ ∨ x₃) ∧ (x₂ ∨ x₃) ∧ (x̄₁ ∨ x̄₂ ∨ x̄₃)
Soddisfacibile con x₁ = true, x₂ = true, x₃ = false.

**In NP:** certificato = assegnamento di verità; certificatore = controlla ogni clausola. ✓

**Teorema 31 — 3-SAT è NP-completo**
Basta mostrare **CIRCUIT-SAT ≤P 3-SAT**.

**Costruzione.** Dato un circuito K, crea una variabile 3-SAT xᵢ per **ogni elemento** (porta o filo) i. Aggiungi clausole che **forzano il calcolo corretto**:

| Tipo di porta | Vincolo | Clausole aggiunte |
|---|---|---|
| NOT | x₂ = ¬x₃ | (x₂ ∨ x₃) ∧ (x̄₂ ∨ x̄₃) — 2 clausole |
| OR | x₁ = x₄ ∨ x₅ | (x̄₁ ∨ x₄ ∨ x₅) ∧ (x₁ ∨ x̄₄) ∧ (x₁ ∨ x̄₅) — 3 clausole |
| AND | x₀ = x₁ ∧ x₂ | (x₀ ∨ x̄₁ ∨ x̄₂) ∧ (x̄₀ ∨ x₁) ∧ (x̄₀ ∨ x₂) — 3 clausole |
| Input hard-coded a 0 | x₅ = 0 | (x̄₅) — 1 clausola |
| Output = 1 | x₀ = 1 | (x₀) — 1 clausola |

**Padding:** le clausole corte (lunghezza 1 o 2) vengono portate a lunghezza esattamente 3 usando **nuove variabili dummy**. Es. la clausola (x₀) diventa:
(x₀ ∨ z₁ ∨ z₂) ∧ (x₀ ∨ z̄₁ ∨ z₂) ∧ (x₀ ∨ z₁ ∨ z̄₂) ∧ (x₀ ∨ z̄₁ ∨ z̄₂)
— soddisfatte tutte e quattro **solo se** x₀ = true, qualunque sia il valore di z₁, z₂.

**Risultato:** la formula è soddisfacibile ⟺ il circuito è soddisfacibile. □

---

### 5.3 INDEPENDENT-SET

**Definizione 25.** Dato un grafo G = (V,E) e un intero k: esiste un sottoinsieme S ⊆ V con **|S| ≥ k** tale che **nessuna coppia di vertici in S condivide un arco**?

**Esempio** (grafo a 8 nodi delle dispense): esiste un independent set di dimensione ≥ 6? **Sì**. Di dimensione ≥ 7? **No**.

**In NP:** certificato = l'insieme S; certificatore = controlla |S| ≥ k e che nessuna coppia in S sia collegata (O(n²)). ✓

**Teorema — 3-SAT ≤P INDEPENDENT-SET (Claim 6)**

**Costruzione.** Data un'istanza 3-SAT Φ con **k clausole**, costruisci il grafo G:
1. Per **ogni clausola**, crea **3 vertici** (uno per letterale) e collegali in un **triangolo**.
2. Collega ogni vertice-letterale a ogni vertice che rappresenta la sua **negazione** in altre clausole (**conflict edges**).

Poni la soglia dell'independent set = **k** (numero di clausole). Il grafo ha **3k** vertici.

**Perché questa costruzione (l'idea chiave):**
- Vogliamo un independent set di dimensione k. Ogni clausola è un **triangolo**, e da un triangolo posso prendere **al massimo un vertice**. Con k triangoli e soglia k, sono **costretto a sceglierne esattamente uno per clausola** → un letterale "vero" per ogni clausola.
- I **conflict edges** mi impediscono di scegliere due letterali che si contraddicono (es. x₁ e x̄₁) → l'assegnamento risultante è **coerente**.

**Prova:** G ha un independent set di dimensione k ⟺ Φ è soddisfacibile.
- **(⇒)** Un independent set di dimensione k prende esattamente un vertice per triangolo (uno per clausola). I triangoli impediscono di sceglierne due dalla stessa clausola; i conflict edges impediscono assegnamenti contraddittori. Ponendo quei letterali a `true` si soddisfa ogni clausola in modo coerente.
- **(⇐)** Dato un assegnamento soddisfacente, prendi **un letterale vero per clausola**. Questi formano un independent set di dimensione k: non ce ne sono due nello stesso triangolo, e non ce ne sono due in conflitto (sono tutti veri nello stesso assegnamento). □

**Esempio concreto:**
Φ = (x̄₁ ∨ x₂ ∨ x₃) ∧ (x₁ ∨ x̄₂ ∨ x₃) ∧ (x̄₁ ∨ x₂ ∨ x₄)
- k = 3 clausole → serve un independent set di dimensione 3.
- G ha 3 × 3 = 9 vertici (3 triangoli).
- Archi: quelli dei triangoli + i conflict edges (es. x₁ nella clausola 2 è collegato a x̄₁ nelle clausole 1 e 3).
- Scelta: **{x₃ (da C₁), x₁ (da C₂), x₄ (da C₃)}** → uno per triangolo, nessun conflitto ⟹ independent set di dimensione 3 ⟹ Φ soddisfacibile con x₁ = x₃ = x₄ = true (x₂ arbitrario).

---

### 5.4 VERTEX-COVER

**Definizione 26.** Dato G = (V,E) e un intero k: esiste S ⊆ V con **|S| ≤ k** tale che **ogni arco ha almeno un estremo in S**?

*In parole: il minimo numero di nodi che coprono tutti gli archi.*

**Esempio** (stesso grafo a 8 nodi): esiste un vertex cover di dimensione ≤ 4? **Sì**. Di dimensione ≤ 3? **No**.

**In NP:** certificato = l'insieme S; certificatore = per ogni arco controlla se almeno un estremo è in S (O(m)). ✓

**Claim 4 — VERTEX-COVER ≡P INDEPENDENT-SET**

> **S è un independent set ⟺ V \ S è un vertex cover.**

**Prova (⇒):** sia S un independent set. Prendi un arco qualsiasi (u,v). Siccome S è indipendente, u ∉ S **oppure** v ∉ S, quindi u ∈ V\S oppure v ∈ V\S. Dunque V\S copre ogni arco.

**Prova (⇐):** sia V\S un vertex cover. Prendi due nodi qualsiasi u, v ∈ S. L'arco (u,v) **non può esistere**, perché V\S dovrebbe coprirlo, ma né u né v stanno in V\S. Quindi nessuna coppia di nodi in S è collegata ⟹ S è indipendente. □

**Conseguenza operativa (la formula da ricordare):**
> G ha un independent set di dimensione **≥ k** ⟺ G ha un vertex cover di dimensione **≤ n − k**.

Questa è una **riduzione per equivalenza semplice** (Strategia 1): l'oracolo si chiama una sola volta, cambiando solo il parametro k → n − k, e la risposta si riporta identica.

**Esempio minimale:** grafo a cammino `a — b — c`, n = 3.
- Independent set {a, c}, dimensione k = 2 ✓ (a e c non sono adiacenti)
- Complemento V \ {a,c} = {b}, dimensione n − k = 1, ed è un vertex cover ✓ (b copre sia (a,b) sia (b,c))

---

### 5.5 SET-COVER

**Definizione 27.** Dato un universo U di elementi, una collezione di sottoinsiemi S₁, S₂, …, S_m ⊆ U e un intero k: esiste una collezione di **≤ k insiemi la cui unione è U**?

**Esempio:**
U = {1,2,3,4,5,6,7}, k = 2
S₁ = {3,7}, S₂ = {3,4,5,6}, S₃ = {1}, S₄ = {2,4}, S₅ = {5}, S₆ = {1,2,6,7}
**Risposta: Sì** — scegli S₂ e S₆: la loro unione è {1,2,3,4,5,6,7} = U.

**Applicazione reale:** hai m pacchetti software, ognuno fornisce un insieme Sᵢ di funzionalità. Puoi coprire tutte le n funzionalità richieste con ≤ k pacchetti?

**In NP:** certificato = i k indici degli insiemi scelti; certificatore = calcola l'unione e confrontala con U. ✓

**Claim 5 — VERTEX-COVER ≤P SET-COVER**

**Costruzione.** Data l'istanza di VERTEX-COVER ⟨G = (V,E), k⟩, crea l'istanza SET-COVER:
```
k' = k
U  = E                                  (gli elementi da coprire sono gli ARCHI)
Sᵥ = { e ∈ E : e è incidente al vertice v }    (un insieme per ogni VERTICE)
```

**Perché funziona:** un insieme di vertici copre tutti gli archi **se e solo se** i corrispondenti insiemi Sᵥ coprono tutti gli elementi. Quindi:
> esiste un vertex cover di dimensione ≤ k ⟺ esiste un set cover di dimensione ≤ k.

**Perché è "caso speciale → caso generale" (Strategia 2):** VERTEX-COVER è esattamente il caso particolare di SET-COVER in cui **ogni elemento appartiene a esattamente 2 insiemi** (un arco ha esattamente 2 estremi). SET-COVER è più generale, quindi almeno altrettanto difficile.

**Esempio (dalle dispense):** grafo con vertici {a,b,c,d,e,f} e archi {e₁,…,e₇}, k = 2.
Il vertex cover {b, e} corrisponde agli insiemi:
- S_b = {e₁, e₂, e₃, e₄}
- S_e = {e₄, e₅, e₆, e₇}

La loro unione è {e₁,…,e₇} = U ✓ — due insiemi coprono tutto, esattamente come due vertici coprivano tutti gli archi.

---

### 5.6 DIR-HAM-CYCLE

**Problema.** Dato un grafo **diretto** G, esiste un **ciclo diretto semplice** che contiene ogni nodo di V?

**In NP:** certificato = permutazione dei nodi; certificatore = controlla che ogni arco consecutivo esista **nella direzione giusta**. ✓

**Teorema 33 — 3-SAT ≤P DIR-HAM-CYCLE**

**Sketch di costruzione.** Costruiamo un grafo con **2ⁿ cicli hamiltoniani**, in corrispondenza biunivoca con i **2ⁿ assegnamenti di verità**:
- Per ogni variabile xᵢ creiamo una **"riga" di nodi**, che può essere attraversata:
  - da **sinistra a destra** → xᵢ = 1
  - da **destra a sinistra** → xᵢ = 0
- Per ogni clausola Cⱼ creiamo un **"nodo clausola"**.
- Colleghiamo i nodi della riga al nodo clausola in modo che il ciclo possa fare la **deviazione (detour)** attraverso il nodo clausola **solo se** la variabile viene attraversata nella direzione "corretta", cioè quella che **soddisfa** la clausola.

**Risultato:** il ciclo hamiltoniano deve visitare tutti i nodi clausola ⟹ ogni clausola deve essere soddisfatta da almeno una variabile ⟹ esiste un ciclo hamiltoniano ⟺ Φ è soddisfacibile.

---

### 5.7 HAM-CYCLE

**Problema.** Dato un grafo **non diretto** G, esiste un ciclo semplice che contiene ogni nodo di V?

**In NP:** vedi §3.4 esempio C. ✓

**Teorema 32 — DIR-HAM-CYCLE ≤P HAM-CYCLE**

**Costruzione.** Trasformiamo il grafo diretto G in un grafo non diretto G′:
- Per ogni nodo v di G creiamo **3 nodi** in G′: **v_in, v, v_out**, collegati in linea:
  `v_in — v — v_out`
- Ogni arco diretto (u, v) diventa un arco non diretto tra **u_out** e **v_in**.

**Perché funziona:** il nodo centrale **v** ha grado 2 (solo v_in e v_out), quindi un ciclo hamiltoniano in G′ è **costretto** ad attraversare la tripletta in linea: `v_in → v → v_out`. Questo impone un **verso di percorrenza uniforme** su tutto il ciclo, che mima perfettamente un ciclo diretto in G.

```
G  (diretto):     u ──────→ v

G' (non diretto): u_in — u — u_out ── v_in — v — v_out
```

---

### 5.8 TSP (Traveling Salesperson Problem)

**Problema.** Dato un insieme di **n città** e le distanze d(u,v), esiste un **tour** (che visita tutte le città e torna al punto di partenza) di lunghezza **≤ D**?

**In NP:** certificato = l'ordine di visita delle città; certificatore = somma le distanze e confronta con D. ✓

**Teorema 34 — HAM-CYCLE ≤P TSP**

**Costruzione.** Data un'istanza G di HAM-CYCLE, crea n città (una per nodo) e poni:
```
d(u,v) = 1   se (u,v) è un arco di G
d(u,v) = 2   se (u,v) NON è un arco di G
D = n
```

**Perché funziona:** ogni tour visita esattamente n archi. La lunghezza minima possibile è n (tutti gli archi di costo 1). Quindi:
> esiste un tour di lunghezza ≤ n ⟺ **tutti** gli archi usati hanno distanza 1 ⟺ tutti gli archi usati sono archi di G ⟺ G ha un ciclo hamiltoniano.

**Esempio:** G = quadrato a—b—c—d—a (senza diagonali).
- Tour a→b→c→d→a: costo 1+1+1+1 = 4 = n ✓ → ciclo hamiltoniano trovato.
- Tour a→c→b→d→a: usa le diagonali (a,c) e (b,d), costo 2+1+2+1 = 6 > 4 ✗.

---

### 5.9 3D-MATCHING (§8.6 — problemi di partizionamento)

**Problema.** Dati tre insiemi disgiunti X, Y, Z, ciascuno di dimensione n, e un insieme di **triple** T ⊆ X × Y × Z: esiste un insieme di **n triple tale che ogni elemento sia coperto esattamente una volta**?

*Esempio applicativo:* accoppiare docenti, corsi e orari — ogni docente insegna un corso in un orario, senza sovrapposizioni.

**In NP:** certificato = le n triple scelte; certificatore = controlla che coprano ogni elemento esattamente una volta. ✓

**Teorema 35 — 3-SAT ≤P 3D-MATCHING**

**Sketch di costruzione.** Si usano tre famiglie di gadget:
- **Gadget "core" e "tip"** per ogni variabile: rappresentano la scelta True/False (una "ruota" di triple in cui si può prendere o il gruppo pari o il gruppo dispari — le due configurazioni corrispondono ai due valori di verità).
- **Gadget "clause"**: garantiscono che almeno un letterale per clausola sia soddisfatto (la clausola può essere coperta solo pescando da un "tip" lasciato libero da un letterale vero).
- **Gadget "cleanup"**: coprono i "tip" rimasti inutilizzati, in modo che il matching risulti perfetto.

---

### 5.10 3-COLOR (§8.7 — colorazione di grafi)

**Problema.** Dato un grafo non diretto G, è possibile colorare i nodi con **3 colori** (Rosso, Verde, Blu) in modo che **nessuna coppia di nodi adiacenti abbia lo stesso colore**?

*Applicazione:* **register allocation** nei compilatori (i nodi sono le variabili, gli archi indicano "vive contemporaneamente", i colori sono i registri disponibili).

**In NP:** certificato = la colorazione; certificatore = controlla ogni arco. ✓

**Teorema 36 — 3-SAT ≤P 3-COLOR**

**Costruzione.**
1. Crea un **triangolo di nodi**: **True (T)**, **False (F)**, **Base (B)**. Essendo un triangolo, i tre nodi ricevono tre colori diversi: fissiamo così il significato dei colori.
2. Per ogni letterale xᵢ, crea due nodi xᵢ e x̄ᵢ, **collegali tra loro** e **collegali entrambi a B**.
   → Effetto: non possono essere colorati come B, e devono avere colori diversi tra loro ⟹ uno è colorato **T** e l'altro **F**. Questo codifica un **assegnamento di verità coerente**.
3. Per ogni clausola, aggiungi un **gadget a 6 nodi** collegato ai tre letterali della clausola e ai nodi **B** e **F**.
   → Effetto: il gadget è 3-colorabile **se e solo se** almeno uno dei tre letterali è colorato **T**.

**Risultato:** G è 3-colorabile ⟺ Φ è soddisfacibile.

---

### 5.11 SUBSET-SUM (§8.8 — problemi numerici)

**Problema.** Dati numeri naturali w₁, …, wₙ e un target W: esiste un sottoinsieme che sommi **esattamente** a W?

**In NP:** certificato = il sottoinsieme; certificatore = somma e confronta con W. ✓

**Teorema 37 — 3-SAT ≤P SUBSET-SUM**

**Sketch di costruzione.** Si costruiscono interi scritti in **base 10**, dove le **cifre** rappresentano:
- una colonna per ogni **variabile**,
- una colonna per ogni **clausola**.

I numeri sono scelti in modo che ottenere la somma esatta W richieda:
1. selezionare **o xᵢ o x̄ᵢ** (mai entrambi, mai nessuno) → assegnamento di verità coerente;
2. **coprire ogni colonna-clausola** → ogni clausola soddisfatta.

Si aggiungono numeri **"dummy"** per gestire i riporti nelle colonne delle clausole (permettono di completare la colonna quando 1, 2 o 3 letterali sono veri).

**Esempio concreto.** Φ = (x̄₁ ∨ x₂ ∨ x₃) ∧ (x₁ ∨ x̄₂ ∨ x₃)

Colonne: `x₁ x₂ x₃ | C₁ C₂`

| Numero | x₁ | x₂ | x₃ | C₁ | C₂ | Valore |
|---|---|---|---|---|---|---|
| x₁ | 1 | 0 | 0 | 0 | 1 | 10001 |
| x̄₁ | 1 | 0 | 0 | 1 | 0 | 10010 |
| x₂ | 0 | 1 | 0 | 1 | 0 | 01010 |
| x̄₂ | 0 | 1 | 0 | 0 | 1 | 01001 |
| x₃ | 0 | 0 | 1 | 1 | 1 | 00111 |
| x̄₃ | 0 | 0 | 1 | 0 | 0 | 00100 |
| dummy C₁ | 0 | 0 | 0 | 1 / 2 | 0 | 00010 / 00020 |
| dummy C₂ | 0 | 0 | 0 | 0 | 1 / 2 | 00001 / 00002 |
| **Target W** | **1** | **1** | **1** | **4** | **4** | **11144** |

- Le colonne-variabile devono valere **1** ⟹ scelgo esattamente uno tra xᵢ e x̄ᵢ.
- Le colonne-clausola devono valere **4**: i letterali contribuiscono da 1 a 3 (se la clausola è soddisfatta) o 0 (se non lo è); i dummy possono aggiungere 0, 1, 2 o 3 ⟹ il totale 4 è raggiungibile **se e solo se** almeno un letterale è vero.

*Verifica con x₁ = x₂ = x₃ = true:* 10001 + 01010 + 00111 = 11122, poi + 00020 (dummy C₁) + 00002 (dummy C₂) = **11144 = W** ✓

---

### 5.12 SCHEDULE-RELEASE-TIMES

**Problema.** Possiamo schedulare n job, dove il job i ha tempo di processamento **tᵢ**, **release time rᵢ** (non può iniziare prima) e **deadline dᵢ** (deve finire entro)?

**In NP:** certificato = lo scheduling (tempo di inizio di ogni job); certificatore = controlla vincoli e sovrapposizioni. ✓

**Teorema 38 — SUBSET-SUM ≤P SCHEDULE-RELEASE-TIMES**

**Costruzione.** Data l'istanza SUBSET-SUM {w₁,…,wₙ}, W:
- Crea n job con **lunghezza tᵢ = wᵢ**, release time 0 e deadline 1 + Σwᵢ.
- Crea **un job speciale "job 0"** con: **lunghezza 1**, **release time W**, **deadline W + 1**.

**Perché funziona:**
- Il job 0 ha una finestra di ampiezza esattamente 1 e lunghezza 1 ⟹ è **costretto a girare esattamente nell'intervallo [W, W+1]**.
- Il tempo totale disponibile è esattamente 1 + Σwᵢ, quindi la macchina **non può mai restare inattiva**.
- Gli altri job devono quindi riempire **esattamente** [0, W] e **esattamente** [W+1, 1 + Σwᵢ].
- Riempire esattamente [0, W] significa trovare un sottoinsieme di pesi che somma esattamente a **W** → mima perfettamente SUBSET-SUM.

```
tempo:  0 ─────────── W ── W+1 ─────────── 1+Σwᵢ
        │ sottoinsieme │job│   il resto    │
        │  somma = W   │ 0 │ somma = Σwᵢ−W │
```

---

## PARTE 6 — RIEPILOGO DELLE RIDUZIONI

| # | Riduzione | Strategia | Idea in una riga |
|---|---|---|---|
| 1 | Ogni X ∈ NP ≤P **CIRCUIT-SAT** | Cook-Levin | Il certificatore poly-time diventa un circuito poly-size |
| 2 | CIRCUIT-SAT ≤P **3-SAT** | Gadget | Una variabile per filo/porta + clausole che forzano il calcolo |
| 3 | 3-SAT ≤P **INDEPENDENT-SET** | Gadget | Triangolo per clausola + conflict edges; k = n° clausole |
| 4 | INDEPENDENT-SET ≡P **VERTEX-COVER** | Equivalenza | S indipendente ⟺ V\S vertex cover; k ↔ n−k |
| 5 | VERTEX-COVER ≤P **SET-COVER** | Caso speciale | U = archi, Sᵥ = archi incidenti a v |
| 6 | 3-SAT ≤P **DIR-HAM-CYCLE** | Gadget | Riga per variabile (verso = valore) + nodo per clausola |
| 7 | DIR-HAM-CYCLE ≤P **HAM-CYCLE** | Gadget | v → (v_in, v, v_out) forza il verso di percorrenza |
| 8 | HAM-CYCLE ≤P **TSP** | Gadget | d = 1 sugli archi, 2 altrove, D = n |
| 9 | 3-SAT ≤P **3D-MATCHING** | Gadget | core/tip per variabile, clause gadget, cleanup |
| 10 | 3-SAT ≤P **3-COLOR** | Gadget | Triangolo T/F/B + gadget a 6 nodi per clausola |
| 11 | 3-SAT ≤P **SUBSET-SUM** | Gadget numerico | Cifre = colonne variabili + clausole, dummy per i riporti |
| 12 | SUBSET-SUM ≤P **SCHEDULE-RELEASE-TIMES** | Gadget | Job 0 di lunghezza 1 bloccato in [W, W+1] |

---

## PARTE 7 — CHECKLIST E TRAPPOLE D'ESAME

**Da saper fare a memoria:**
- [ ] Definizione di ≤P, di certificatore, di NP, di NP-completo
- [ ] Prove di P ⊆ NP e NP ⊆ EXP
- [ ] Prova del Teorema centrale (Y NP-completo risolvibile in poly ⟺ P = NP)
- [ ] La ricetta in 3 passi per provare la NP-completezza
- [ ] Claim 4 (IS ≡P VC) con **entrambe** le direzioni — è la prova più richiesta
- [ ] La costruzione 3-SAT → INDEPENDENT-SET e perché triangoli + conflict edges
- [ ] La catena completa dei domino

**Trappole ricorrenti:**

| Trappola | Chiarimento |
|---|---|
| "NP significa non-polinomiale" | **No**: significa *Nondeterministic Polynomial*, cioè **verificabile** in poly-time |
| Ridurre nella direzione sbagliata | Per provare Y difficile: **X_noto ≤P Y**, mai il contrario |
| Dimostrare una sola direzione | Serve sempre `x ∈ X ⟺ y ∈ Y`, entrambi i versi |
| Dimenticare il punto 1 della ricetta | NP-completo richiede **anche** Y ∈ NP, non solo la riduzione |
| Confondere polinomiale nel valore e nella lunghezza | Poly nella **lunghezza** dell'input (PRIMES: O(\|s\|⁸), non O(s)) |
| Dimenticare che l'istanza costruita deve essere di dimensione polinomiale | Fa parte della definizione di riduzione |

**Terminologia extra utile (non nelle dispense, ma spesso chiesta):**
- **NP-hard**: soddisfa solo la condizione 2 (ogni X ∈ NP si riduce a Y), **senza** richiedere Y ∈ NP. Quindi: *NP-completo = NP-hard **e** in NP*. Un problema NP-hard può essere anche più difficile (es. indecidibile).
