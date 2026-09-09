# Quaderno delle Riduzioni NP-Complete

Questo documento raccoglie tutte le riduzioni discusse, strutturate in base alla definizione dei problemi, l'algoritmo di riduzione (con pseudocodice) e gli esempi intuitivi affrontati.

---

## 1. VERTEX COVER = INDEPENDENT SET (IS)

**I Problemi:**
*   **Independent Set (IS):** Dato un grafo $G=(V,E)$ e un intero $k$, esiste un sottoinsieme di nodi $S \subseteq V$ di dimensione $\ge k$ tale che nessun arco colleghi due nodi in $S$?
*   **Vertex Cover (VC):** Dato un grafo $G=(V,E)$ e un intero $k$, esiste un sottoinsieme di nodi $C \subseteq V$ di dimensione $\le k$ tale che ogni arco abbia almeno un estremo in $C$?

**L'Algoritmo (Equivalenza):**
Un insieme $S$ è un Independent Set se e solo se il suo complemento $V \setminus S$ è un Vertex Cover.
```text
Funzione Riduci_VC_a_IS(Grafo G=(V,E), intero k):
    Ritorna istanza IS: (G, |V| - k)
```

**Esempio Intuitivo:**
Un grafo quadrato $C_4$ con nodi $1, 2, 3, 4$. 
*   L'insieme $S = \{1, 3\}$ è un Independent Set di taglia 2 (non ci sono archi tra 1 e 3).
*   Il complemento $V \setminus S = \{2, 4\}$ è un Vertex Cover di taglia 2 (copre tutti gli archi del quadrato).

---

## 2. VERTEX COVER $	o$ SET COVER

**Il Problema (Set Cover):**
Dato un universo $U$ di elementi, una famiglia $F$ di sottoinsiemi di $U$ e un intero $k$, esiste una sottofamiglia di $k$ insiemi che uniti contengono tutti gli elementi di $U$?

**L'Algoritmo:**
Trasformiamo i nodi in insiemi e gli archi negli elementi da coprire.
```text
Funzione Riduci_VC_a_SC(Grafo G=(V,E), intero k):
    Universo U = E (l'insieme degli archi)
    Per ogni nodo v in V:
        Crea un insieme S_v contenente tutti gli archi incidenti a v
    Ritorna istanza SC: (U, {S_v per ogni v}, k)
```

**Esempio Intuitivo:**
Un grafo a stella con un nodo centrale (A) collegato a tre foglie (B, C, D). 
Per coprire gli archi, basta prendere l'insieme associato al nodo A, poiché contiene tutti gli archi del grafo. Vertex Cover (taglia 1) equivale a Set Cover (taglia 1).

---

## 3. 3-SAT $	o$ INDEPENDENT SET

**Il Problema (3-SAT):**
Data una formula logica in forma normale congiuntiva (AND di OR) in cui ogni clausola ha esattamente 3 letterali, esiste un'assegnazione di verità che rende vera la formula?

**L'Algoritmo:**
Creiamo un grafo in cui i nodi rappresentano i letterali, forzando la scelta di un solo letterale vero per clausola e impedendo contraddizioni logiche.
```text
Funzione Riduci_3SAT_a_IS(Formula F con k clausole):
    Per ogni clausola C_j = (L_1 v L_2 v L_3):
        Crea un triangolo con 3 nodi (uno per ogni letterale)
    Per ogni coppia di nodi che rappresentano letterali opposti (es. x e not x):
        Aggiungi un arco tra di loro (arco di conflitto)
    Ritorna istanza IS: (Grafo creato, k)
```

**Esempio Intuitivo:**
Per la formula $(x_1 \lor x_2 \lor x_3) \land (
eg x_1 \lor x_4 \lor x_5)$.
Creiamo due triangoli. Scegliere un Independent Set di taglia $k=2$ significa pescare un nodo da ogni triangolo senza mai scegliere contemporaneamente $x_1$ e $
eg x_1$ (che sono collegati da un arco di conflitto).

---

## 4. DIR-HAM-CYCLE $	o$ HAM-CYCLE

**I Problemi:**
*   **DIR-HAM-CYCLE:** Ciclo che visita ogni nodo una sola volta in un grafo *diretto*.
*   **HAM-CYCLE:** Ciclo che visita ogni nodo una sola volta in un grafo *non diretto*.

**L'Algoritmo:**
Si sdoppia ogni nodo in tre sottomodi per preservare la direzionalità degli archi nel grafo non diretto.
```text
Funzione Riduci_DHC_a_HC(Grafo Diretto G=(V,E)):
    Per ogni nodo v in V:
        Crea tre nodi: v_in, v_mid, v_out
        Aggiungi archi non diretti (v_in, v_mid) e (v_mid, v_out)
    Per ogni arco diretto (u, w) in E:
        Aggiungi un arco non diretto (u_out, w_in)
    Ritorna istanza HC: Grafo non diretto creato
```

**Esempio Intuitivo:**
Immagina un incrocio autostradale (nodo $A$). Nel grafo non diretto, potresti attraversarlo al contrario. Dividendo $A$ in corsia d'ingresso ($A_{in}$), centro ($A_{mid}$) e uscita ($A_{out}$), chiunque entri in $A_{in}$ è fisicamente obbligato a passare per $A_{mid}$ e uscire da $A_{out}$, costringendo il ciclo a rispettare il senso di marcia originale.

---

## 5. 3-SAT $	o$ DIR-HAM-CYCLE

**L'Algoritmo:**
Si converte la scelta Vero/Falso nella scelta della direzione in cui percorrere una catena di nodi.
```text
Funzione Riduci_3SAT_a_DHC(Formula F):
    Per ogni variabile x:
        Crea una riga di nodi con archi bidirezionali (Sx -> Dx = Vero, Dx -> Sx = Falso)
    Per ogni clausola C:
        Crea un "nodo isola" isolato
    Per ogni letterale nella formula:
        Collega il nodo isola alla riga della variabile corrispondente 
        orientando gli ingressi/uscite in base a Vero/Falso
    Collega inizio e fine delle righe per chiudere il ciclo
    Ritorna istanza DHC sul grafo creato
```

**Esempio Intuitivo:**
Se la variabile $x$ è vera, percorro il suo binario da sinistra a destra. Questo mi abilita a usare le "rampe di uscita" che deviano verso i nodi clausola che richiedono $x=	ext{Vero}$. Se passo nella direzione sbagliata, le rampe sono in contromano e non posso visitare i nodi clausola.

---

## 6. HAM-CYCLE $	o$ TSP

**Il Problema (TSP Decisionale):**
Dato un grafo pesato, esiste un tour che visita tutte le città (nodi) con un costo totale $\le D$?

**L'Algoritmo:**
Si crea un grafo completo assegnando pesi bassi agli archi che esistono nel grafo originale, e pesi alti agli archi mancanti.
```text
Funzione Riduci_HC_a_TSP(Grafo G=(V,E)):
    Crea un grafo completo K con gli stessi nodi V
    Per ogni coppia di nodi u, v:
        Se l'arco (u,v) esiste in E: Peso = 1
        Altrimenti: Peso = 2
    Ritorna istanza TSP: (K, target D = |V|)
```

**Esempio Intuitivo:**
Hai una mappa di strade esistenti e vuoi sapere se puoi fare un giro passando per tutte le città. Disegni una mappa dove puoi volare ovunque: guidare su una strada vera costa 1, usare l'elicottero (strada falsa) costa 2. Se riesci a fare il giro intero spendendo esattamente quanto il numero di città, significa che non hai mai preso l'elicottero.

---

## 7. 3-SAT $	o$ 3D MATCHING

**Il Problema (3D Matching):**
Dati tre insiemi $X,Y,Z$ disgiunti e di uguale cardinalità, e un insieme di triplette $T \subseteq X 	imes Y 	imes Z$, esiste un sottoinsieme di triplette che copre ogni elemento esattamente una volta?

**L'Algoritmo:**
Usa elementi "core" e "tip" per le variabili, permettendo la scelta esclusiva.
```text
Funzione Riduci_3SAT_a_3DM(Formula F):
    Per ogni variabile x:
        Crea un core (A, B) e punte P_vero, P_falso
        Aggiungi triplette per le due assegnazioni
    Per ogni clausola C:
        Crea un core (C_x, C_y)
        Aggiungi triplette agganciandole alle punte che soddisfano la clausola
    Aggiungi elementi "slack/cleanup" per coprire le punte non usate
    Ritorna istanza 3DM
```

**Esempio Intuitivo:**
Se $x=	ext{Vero}$, usi il core della variabile per "coprire" la punta Falsa, lasciando scoperta la punta Vera. Il "core" della clausola ha disperatamente bisogno di incastrarsi con una punta libera: troverà la punta Vera e la clausola sarà soddisfatta!

---

## 8. 3-SAT $	o$ 3-COLORING

**Il Problema (3-COLOR):**
È possibile colorare i nodi di un grafo con 3 colori in modo che due nodi collegati non abbiano mai lo stesso colore?

**L'Algoritmo:**
```text
Funzione Riduci_3SAT_a_3COL(Formula F):
    Crea la Tavolozza (3 nodi collegati a triangolo: T, F, B)
    Per ogni variabile x:
        Crea nodi (x) e (not x) collegati tra loro e collegati a (B)
    Per ogni clausola C = (L1 v L2 v L3):
        Crea l'imbuto (6 nodi intermedi, 2 mini-OR)
        Collega gli ingressi a L1, L2, L3
        Collega l'uscita a (F) e (B)
    Ritorna istanza 3-COLOR: Grafo creato
```

**Esempio Intuitivo:**
Il nodo Uscita di una clausola è forzato dal grafo a essere per forza colorato di "Vero" (poiché è collegato a Falso e Base). L'imbuto è costruito in modo che, se tutti e tre i letterali in ingresso sono colorati di "Falso", la regola dei colori si rompe e il grafo si "inceppa". Serve almeno un ingresso "Vero" per colorare il grafo con successo.

---

## 9. 3-SAT $	o$ SUBSET SUM

**Il Problema (Subset Sum):**
Dato un insieme di interi, esiste un sottoinsieme la cui somma è esattamente uguale a un target $W$?

**L'Algoritmo:**
Cambio di base (base 10) per trasformare logica e insiemi in somma algebrica senza riporto.
```text
Funzione Riduci_3SAT_a_SS(Formula F con N var e K clausole):
    Genera un target W composto da: 
        N cifre "1" (per le variabili) seguite da K cifre "3" (per le clausole)
    Per ogni variabile:
        Genera due numeri (Vero/Falso).
        Hanno un "1" nella loro colonna var, e un "1" nelle clausole che soddisfano.
    Per ogni clausola:
        Genera due numeri "slack" che valgono 1 solo nella colonna di quella clausola.
    Ritorna istanza SS: (Insieme dei numeri generati, target W)
```

**Esempio Intuitivo:**
Il target finisce con "...3". Puoi prendere al massimo due numeri slack (che aggiungono $1+1=2$). Per raggiungere il 3, sei costretto ad aver pescato almeno una variabile che fornisce un "1" nella colonna di quella specifica clausola, il che equivale matematicamente a soddisfarla.
