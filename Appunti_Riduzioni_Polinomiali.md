# Appunti sulle Riduzioni Polinomiali

Esistono tre strategie di base per effettuare riduzioni polinomiali e dimostrare l'intrattabilità di un problema[cite: 8].

---

## 1. Strategia 1: Semplice Equivalenza (Simple Equivalence)

Questa strategia si applica quando due problemi sono essenzialmente lo stesso problema visto da prospettive diverse[cite: 8].

* **Definizione di Independent Set:** Dato un grafo $G=(V, E)$ e un intero $k$, si chiede se esista un sottoinsieme $S \subseteq V$ con $|S| \ge k$ tale che nessun paio di vertici in $S$ condivida un arco[cite: 8].
* **Definizione di Vertex Cover:** Dato un grafo $G=(V, E)$ e un intero $k$, si chiede se esista un sottoinsieme $S \subseteq V$ con $|S| \le k$ tale che ogni arco abbia almeno un estremo in $S$[cite: 8].
* **Teorema:** VERTEX-COVER $\equiv_P$ INDEPENDENT-SET[cite: 8].
* **Dimostrazione:** $S$ è un independent set se e solo se $V \setminus S$ è un vertex cover[cite: 8].
  * Se $S$ è un independent set, non esiste alcun arco che abbia entrambi gli estremi in $S$[cite: 8]. Quindi, ogni arco deve avere almeno un estremo in $V \setminus S$, rendendo quest'ultimo un vertex cover[cite: 8].
  * Viceversa, se $V \setminus S$ è un vertex cover, esso copre tutti gli archi, il che significa che non possono esistere archi con entrambi gli estremi nell'insieme rimanente $S$[cite: 8]. Di conseguenza, $S$ è un independent set[cite: 8].

---

## 2. Strategia 2: Da Caso Speciale a Caso Generale (Special Case to General Case)

* **Definizione di Set Cover:** Dato un universo $U$ di elementi, una collezione di sottoinsiemi $S_1, S_2, \dots, S_m \subseteq U$, e un intero $k$, si chiede se esista una collezione di al massimo $k$ insiemi la cui unione sia esattamente $U$[cite: 8].
* **Teorema:** VERTEX-COVER $\le_P$ SET-COVER[cite: 8].
* **Costruzione:** Data un'istanza di VERTEX-COVER definita da $\langle G=(V,E), k angle$, creiamo un'istanza di SET-COVER impostando $k' = k$ e $U = E$[cite: 8]. Per ogni vertice $v$, creiamo un insieme $S_v = \{e \in E : e 	ext{ è incidente a } v\}$[cite: 8].
* **Correttezza:** Un insieme di vertici copre tutti gli archi se e solo se i corrispondenti insiemi scelti coprono tutti gli elementi dell'universo[cite: 8].

---

## 3. Strategia 3: Codifica con i Gadget (Encoding with Gadgets)

Questa strategia è usata per tradurre la logica di un problema nei vincoli strutturali di un altro[cite: 8].

* **Definizione di 3-SAT:** È una variante del problema SAT in cui ogni clausola della formula CNF possiede esattamente 3 letterali[cite: 8].
* **Teorema:** 3-SAT $\le_P$ INDEPENDENT-SET[cite: 8].
* **Costruzione:** Data un'istanza 3-SAT con $k$ clausole, costruiamo un grafo $G$[cite: 8].
  * Per ogni clausola, creiamo 3 vertici (uno per ciascun letterale) e li connettiamo formando un triangolo[cite: 8]. Questa struttura impedisce di scegliere più di un vertice per clausola all'interno di un independent set[cite: 8].
  * Aggiungiamo dei "conflict edges": colleghiamo ogni vertice letterale a tutti i vertici che rappresentano la sua negazione nelle altre clausole[cite: 8].
* **Correttezza:** Il grafo $G$ possiede un independent set di dimensione $k$ se e solo se la formula $\Phi$ è soddisfacibile[cite: 8].
  * Un independent set di dimensione $k$ deve scegliere esattamente un vertice per ogni triangolo, garantendo che ogni clausola sia soddisfatta[cite: 8]. I conflict edges impediscono l'assegnazione di valori contraddittori, assicurando che la soluzione formi un'assegnazione di verità coerente[cite: 8].

---

## 4. Transitività delle Riduzioni

* **Teorema della Transitività:** Se $X \le_P Y$ e $Y \le_P Z$, allora $X \le_P Z$[cite: 8].
* Questo teorema ci permette di concatenare le riduzioni tra loro[cite: 8]. Un esempio chiave derivante dalle riduzioni precedenti è la seguente catena:
  3-SAT $\le_P$ INDEPENDENT-SET $\le_P$ VERTEX-COVER $\le_P$ SET-COVER[cite: 8].
