# LINQ e le collezioni in C#

Questa dispensa affronta due argomenti strettamente legati: le interfacce delle collezioni e LINQ.
Prima di usare LINQ in modo consapevole è fondamentale capire quali strutture dati lo supportano, come funzionano le espressioni lambda e perché esistono tante interfacce diverse.

---

## Capitolo 1 — Le interfacce delle collezioni

In C# le collezioni non sono un'unica cosa: esistono molti tipi diversi di lista, insieme, dizionario e sequenza.
Tutte queste strutture condividono alcune caratteristiche di base, e quelle caratteristiche comuni sono modellate tramite interfacce.

Il punto centrale è questo: se scrivi il tuo codice lavorando con un'interfaccia invece che con un tipo concreto, puoi cambiare la struttura dati interna senza dover riscrivere il codice che la usa.

### IEnumerable\<T\>

È l'interfaccia più basilare di tutta la gerarchia delle collezioni.
Dice una sola cosa: "questa sequenza può essere attraversata con un ciclo `foreach`".
Non dice quanti elementi ci sono, non permette di aggiungerne, non dà accesso per indice.

**Perché è stata introdotta**
Serviva un contratto minimo per permettere a qualsiasi tipo di sequenza, da un array a un file CSV letto riga per riga, di essere trattata in modo uniforme.
LINQ si appoggia quasi interamente su `IEnumerable<T>`.

```csharp
IEnumerable<int> numeri = new int[] { 1, 2, 3, 4 };

foreach (int n in numeri)
{
    Console.WriteLine(n);
}
```

**Esempio meno adatto**

```csharp
int[] numeri = new int[] { 1, 2, 3, 4 };
```

Usare direttamente il tipo concreto `int[]` lega il codice a quella struttura specifica. Se in futuro vuoi passare a una lista o a un generatore pigro, devi cambiare tutto il codice che lo usa. Con `IEnumerable<T>` non devi.

### IEnumerator\<T\>

È il meccanismo interno che `IEnumerable<T>` usa per camminare sulla sequenza.
Espone tre membri: `Current` (l'elemento corrente), `MoveNext()` (avanza) e `Reset()`.

**Perché è stata introdotta**
Separare il "cursore" dalla collezione permette di avere più cursori attivi sulla stessa sequenza contemporaneamente, ognuno nella propria posizione.

```csharp
IEnumerable<string> nomi = new string[] { "Alice", "Bob", "Carlo" };
IEnumerator<string> cursore = nomi.GetEnumerator();

while (cursore.MoveNext())
{
    Console.WriteLine(cursore.Current);
}
```

In pratica non usi `IEnumerator<T>` direttamente quasi mai: è `foreach` a chiamarlo per te.

### ICollection\<T\>

Estende `IEnumerable<T>` aggiungendo la capacità di modificare la collezione e di sapere quanti elementi contiene.
I membri principali sono `Count`, `Add`, `Remove`, `Contains` e `Clear`.

**Perché è stata introdotta**
`IEnumerable<T>` non basta quando hai bisogno di sapere la dimensione o di aggiungere elementi.
`ICollection<T>` è il livello minimo per gestire una collezione mutabile.

```csharp
ICollection<string> nomi = new List<string>();
nomi.Add("Alice");
nomi.Add("Bob");

Console.WriteLine(nomi.Count);
Console.WriteLine(nomi.Contains("Bob"));
```

**Esempio meno adatto**

```csharp
List<string> nomi = new List<string>();
nomi.Add("Alice");
```

Usare `ICollection<T>` come tipo della variabile è più corretto quando non ti serve l'accesso per indice: comunica che il codice richiede solo la capacità di aggiungere, rimuovere e contare.

### IList\<T\>

Estende `ICollection<T>` aggiungendo l'accesso agli elementi per indice con `[i]`, e metodi come `IndexOf`, `Insert` e `RemoveAt`.

**Perché è stata introdotta**
Non sempre serve l'accesso per indice, ma quando serve è importante averlo esplicitamente nel contratto.
`IList<T>` dice: "questa collezione mantiene un ordine e permette di accedere agli elementi tramite posizione".

```csharp
IList<string> nomi = new List<string> { "Alice", "Bob", "Carlo" };

Console.WriteLine(nomi[1]);

nomi.Insert(0, "Daniele");
Console.WriteLine(nomi[0]);
```

**Esempio meno adatto**

```csharp
// Errore concettuale: IEnumerable non ha l'accesso per indice
IEnumerable<string> nomi = new List<string> { "Alice", "Bob" };
// Console.WriteLine(nomi[1]); -- non compila
```

Usare `IEnumerable<T>` quando hai bisogno dell'indice è un errore concettuale: scegli il contratto che rispecchia davvero le operazioni che intendi fare.

### ISet\<T\>

Modella un insieme: una collezione di elementi unici senza duplicati e con operazioni insiemistiche come unione, intersezione e differenza.
I metodi principali sono `UnionWith`, `IntersectWith`, `ExceptWith` e `IsSubsetOf`.

**Perché è stata introdotta**
Un insieme è concettualmente diverso da una lista: l'ordine non importa e i duplicati non esistono.
`ISet<T>` esprime questa semantica in modo esplicito.

```csharp
ISet<string> gruppoA = new HashSet<string> { "Alice", "Bob" };
ISet<string> gruppoB = new HashSet<string> { "Bob", "Carlo" };

gruppoA.IntersectWith(gruppoB);

foreach (string nome in gruppoA)
{
    Console.WriteLine(nome);
}
```

**Esempio meno adatto**

```csharp
IList<string> gruppoA = new List<string> { "Alice", "Bob", "Bob" };
```

Una lista non garantisce l'unicità degli elementi. Se hai bisogno di escludere i duplicati per definizione, una lista è la struttura sbagliata.

### IDictionary\<TKey, TValue\>

Modella un dizionario: una struttura dati che associa chiavi a valori.
I membri principali sono `Add`, `Remove`, `ContainsKey`, `TryGetValue`, `Keys` e `Values`.

**Perché è stata introdotta**
Associare un valore a una chiave è un'operazione così comune che merita un'interfaccia dedicata.
Permette di trattare qualsiasi dizionario concreto, come `Dictionary<K,V>` o `SortedDictionary<K,V>`, con lo stesso contratto.

```csharp
IDictionary<string, int> voti = new Dictionary<string, int>();
voti.Add("Alice", 28);
voti.Add("Bob", 21);

if (voti.TryGetValue("Alice", out int votoAlice))
{
    Console.WriteLine($"Alice: {votoAlice}");
}
```

**Esempio meno adatto**

```csharp
IList<string> voti = new List<string> { "Alice:28", "Bob:21" };
```

Codificare coppie chiave-valore dentro stringhe di una lista è fragile e difficile da interrogare. `IDictionary` rende l'associazione esplicita e sicura.

### IReadOnlyCollection\<T\> e IReadOnlyList\<T\>

Sono versioni in sola lettura di `ICollection<T>` e `IList<T>`.
Non espongono `Add`, `Remove` o `Clear`: permettono solo di leggere.

**Perché sono state introdotte**
Quando vuoi esporre una collezione all'esterno senza permetterne la modifica, restituire `IReadOnlyList<T>` comunica questa intenzione in modo netto. Chi riceve il valore sa che non deve modificare la struttura.

```csharp
class Registro
{
    private readonly List<string> _voci = new List<string>();

    public void Aggiungi(string voce)
    {
        _voci.Add(voce);
    }

    public IReadOnlyList<string> Voci => _voci;
}
```

**Esempio meno adatto**

```csharp
class Registro
{
    public List<string> Voci = new List<string>();
}
```

Esporre la lista interna direttamente consente al codice esterno di aggiungere, rimuovere e svuotare i dati senza che la classe ne sappia nulla.

### IQueryable\<T\>

Estende `IEnumerable<T>` e rappresenta una query che non viene eseguita subito, ma tradotta in un'altra forma, come SQL, ed eseguita su una sorgente remota.

**Perché è stata introdotta**
Con `IEnumerable<T>` un filtro come `Where` carica tutti i dati in memoria e poi li filtra.
Con `IQueryable<T>` la query viene trasformata in un'istruzione SQL e il filtro avviene direttamente sul database, senza caricare dati inutili.

```csharp
// IQueryable - la query viene eseguita sul database
// IQueryable<string> nomiDalDb = dbContext.Nomi.Where(n => n.StartsWith("A"));

// IEnumerable - la query viene eseguita in memoria dopo aver caricato tutto
IEnumerable<string> nomi = new string[] { "Alice", "Anna", "Bob" };
IEnumerable<string> nomiInMemoria = nomi.Where(n => n.StartsWith("A"));
```

`IQueryable<T>` è usato principalmente con Entity Framework o con altri ORM, ed è approfondito nella dispensa su ASP.NET.

### Gerarchia riepilogativa

```
IEnumerable<T>
    └── ICollection<T>
            ├── IList<T>
            └── ISet<T>

IEnumerable<T>
    └── IQueryable<T>

IEnumerable<KeyValuePair<TKey, TValue>>
    └── ICollection<KeyValuePair<TKey, TValue>>
            └── IDictionary<TKey, TValue>

IEnumerable<T>
    └── IReadOnlyCollection<T>
            └── IReadOnlyList<T>
```

Capire questa gerarchia significa scegliere sempre il tipo più adatto alle operazioni che vuoi fare.

---

## Capitolo 2 — Prerequisiti: lambda e predicati

Prima di affrontare LINQ è necessario capire due strumenti su cui si appoggia quasi ogni operatore: le espressioni lambda e i predicati.
Senza di essi il codice LINQ è illeggibile.

### Delegati

Un delegato è un tipo che rappresenta un riferimento a un metodo.
In pratica è una variabile che contiene una funzione, non un valore.

```csharp
// Dichiarazione di un delegato: firma = prende un int, restituisce bool
delegate bool Controllo(int numero);

// Metodo compatibile con la firma
bool EPositivo(int n)
{
    return n > 0;
}

// Assegnazione
Controllo c = EPositivo;
Console.WriteLine(c(5));  // true
Console.WriteLine(c(-1)); // false
```

**Perché esistono**
I delegati permettono di passare comportamenti come parametri. Questo è esattamente ciò che fanno gli operatori LINQ: ricevono una funzione e la applicano a ogni elemento della sequenza.

### Func e Action

C# fornisce delegati generici già pronti per i casi più comuni, così non devi dichiararne di nuovi ogni volta.

`Func<TInput, TOutput>` rappresenta un metodo che prende un valore e ne restituisce un altro.
`Action<T>` rappresenta un metodo che prende un valore e non restituisce nulla.

```csharp
Func<int, bool> ePositivo = EPositivo;

Console.WriteLine(ePositivo(5));  // true

Action<string> stampa = Console.WriteLine;
stampa("Ciao");
```

La maggior parte degli operatori LINQ accetta `Func<T, bool>` o `Func<T, TResult>` come parametro.

### Espressioni lambda

Un'espressione lambda è una funzione anonima: una funzione senza nome, scritta direttamente nel punto in cui serve.
La sintassi è: `parametri => corpo`.

```csharp
// Funzione anonima che prende un int e restituisce true se è positivo
Func<int, bool> ePositivo = n => n > 0;

Console.WriteLine(ePositivo(5));  // true
Console.WriteLine(ePositivo(-1)); // false
```

Quando il corpo è più complesso, si usano le graffe:

```csharp
Func<int, string> classifica = voto =>
{
    if (voto >= 18)
    {
        return "Promosso";
    }

    return "Rimandato";
};

Console.WriteLine(classifica(24)); // Promosso
Console.WriteLine(classifica(12)); // Rimandato
```

**Esempio meno adatto**

```csharp
// Senza lambda: serve un metodo separato anche per logiche banali
static bool EPositivo(int n)
{
    return n > 0;
}

Func<int, bool> controllo = EPositivo;
```

Le lambda eliminano la necessità di definire un metodo separato per ogni piccola operazione, rendendo il codice più compatto e leggibile nel contesto in cui viene usato.

### Predicati

Un predicato è una funzione che prende un valore e restituisce `true` o `false`.
In C# è semplicemente un `Func<T, bool>`.

```csharp
Func<int, bool> ePari = n => n % 2 == 0;
Func<string, bool> eCorto = s => s.Length <= 3;

Console.WriteLine(ePari(4));       // true
Console.WriteLine(eCorto("Ciao")); // false
```

I predicati sono usati ovunque in LINQ: `Where`, `Any`, `All`, `Count`, `First` e molti altri operatori accettano un predicato per decidere su quali elementi agire.

### Lambda con più parametri

Alcune operazioni LINQ usano lambda con due parametri, per esempio `Join` o certi overload di `Select`.

```csharp
Func<int, int, int> somma = (a, b) => a + b;
Console.WriteLine(somma(3, 4)); // 7
```

### Riepilogo visivo

```
n => n > 0
^    ^^^^^
|      |
|   corpo della funzione (espressione o blocco)
|
parametro (il nome è libero)
```

```
(a, b) => a + b
^^^^^^    ^^^^^
  |         |
più        corpo
parametri
```

Tenendo a mente questo schema, ogni operatore LINQ diventa leggibile: `Where(n => n > 2)` significa "filtra tenendo solo gli elementi per cui `n > 2` è vero".

---

## Capitolo 3 — Cos'è LINQ

LINQ è l'acronimo di Language Integrated Query.
È un insieme di estensioni del linguaggio C# che permette di interrogare, filtrare e trasformare sequenze di dati usando una sintassi integrata nel codice.

La caratteristica più importante è che funziona su qualsiasi `IEnumerable<T>`, quindi su array, liste, dizionari, set e qualsiasi altra struttura supporti quell'interfaccia.

### Deferred execution

La maggior parte degli operatori LINQ non esegue immediatamente l'operazione.
Costruisce una rappresentazione della query e la esegue solo quando i risultati vengono effettivamente richiesti, per esempio con `foreach`, `ToList()` o `Count()`.

```csharp
IEnumerable<int> numeri = new int[] { 1, 2, 3, 4, 5 };

// La query non viene eseguita qui...
IEnumerable<int> query = numeri.Where(n => n > 2);

// ...ma solo qui, quando iteriamo
foreach (int n in query)
{
    Console.WriteLine(n);
}
```

**Esempio meno adatto**

```csharp
IEnumerable<int> numeri = new int[] { 1, 2, 3, 4, 5 };

// Qui la query viene materializzata subito e i risultati restano fissi
List<int> risultati = numeri.Where(n => n > 2).ToList();
```

Capire quando la query viene eseguita è fondamentale per evitare comportamenti inattesi, soprattutto quando la sorgente dati è un database.

### Sintassi query

C# offre una sintassi simile a SQL per scrivere query LINQ.

```csharp
int[] voti = new int[] { 15, 18, 21, 30, 12, 24 };

IEnumerable<int> votiSufficienti =
    from v in voti
    where v >= 18
    orderby v descending
    select v;

foreach (int v in votiSufficienti)
{
    Console.WriteLine(v);
}
```

### Sintassi metodo (fluent)

La stessa query scritta come concatenazione di metodi:

```csharp
int[] voti = new int[] { 15, 18, 21, 30, 12, 24 };

IEnumerable<int> votiSufficienti = voti
    .Where(v => v >= 18)
    .OrderByDescending(v => v);

foreach (int v in votiSufficienti)
{
    Console.WriteLine(v);
}
```

Le due sintassi producono lo stesso risultato.
La sintassi metodo è generalmente preferita perché è più flessibile e funziona anche in contesti dove la sintassi query non è disponibile.

---

## Capitolo 4 — Gli operatori principali

### Where

Filtra gli elementi in base a una condizione.

```csharp
int[] numeri = new int[] { 1, 2, 3, 4, 5, 6 };

IEnumerable<int> pari = numeri.Where(n => n % 2 == 0);
```

**Esempio meno adatto**

```csharp
List<int> pari = new List<int>();

for (int i = 0; i < numeri.Length; i++)
{
    if (numeri[i] % 2 == 0)
    {
        pari.Add(numeri[i]);
    }
}
```

Il ciclo manuale è più verboso e sposta l'attenzione su come si fa, non su cosa si vuole ottenere.

### Select

Trasforma ogni elemento applicando una funzione.

```csharp
string[] nomi = new string[] { "alice", "bob", "carlo" };

IEnumerable<string> nomiMaiuscoli = nomi.Select(n => n.ToUpper());
```

**Esempio meno adatto**

```csharp
for (int i = 0; i < nomi.Length; i++)
{
    nomi[i] = nomi[i].ToUpper();
}
```

Il ciclo modifica la sorgente originale. Con `Select` la sorgente resta invariata e si ottiene una nuova sequenza.

### OrderBy e OrderByDescending

Ordina la sequenza in modo crescente o decrescente.

```csharp
int[] voti = new int[] { 18, 30, 22, 15 };

IEnumerable<int> crescente = voti.OrderBy(v => v);
IEnumerable<int> decrescente = voti.OrderByDescending(v => v);
```

### GroupBy

Raggruppa gli elementi in base a una chiave comune.

```csharp
string[] nomi = new string[] { "Alice", "Anna", "Bob", "Beatrice", "Carlo" };

IEnumerable<IGrouping<char, string>> perIniziale = nomi.GroupBy(n => n[0]);

foreach (IGrouping<char, string> gruppo in perIniziale)
{
    Console.WriteLine($"Iniziale: {gruppo.Key}");

    foreach (string nome in gruppo)
    {
        Console.WriteLine($"  {nome}");
    }
}
```

### First, FirstOrDefault, Single, SingleOrDefault

Restituiscono uno o il primo elemento che soddisfa una condizione.

```csharp
int[] numeri = new int[] { 1, 3, 5, 6, 7 };

// Primo pari, eccezione se non esiste
int primoPari = numeri.First(n => n % 2 == 0);

// Primo pari, valore di default se non esiste
int primoPariSicuro = numeri.FirstOrDefault(n => n % 2 == 0);
```

`First` lancia un'eccezione se non trova nulla. `FirstOrDefault` restituisce il valore di default del tipo, quindi `0` per gli interi e `null` per i tipi di riferimento.
`Single` è come `First` ma verifica anche che esista esattamente un solo elemento corrispondente.

**Esempio meno adatto**

```csharp
int primoPari = 0;

for (int i = 0; i < numeri.Length; i++)
{
    if (numeri[i] % 2 == 0)
    {
        primoPari = numeri[i];
        break;
    }
}
```

### Any e All

`Any` verifica se esiste almeno un elemento che soddisfa la condizione.
`All` verifica se tutti gli elementi la soddisfano.

```csharp
int[] voti = new int[] { 18, 22, 30 };

bool haInsufficienze = voti.Any(v => v < 18);
bool tuttiSufficienti = voti.All(v => v >= 18);
```

### Count, Sum, Average, Min, Max

Operatori di aggregazione che restituiscono un singolo valore.

```csharp
int[] voti = new int[] { 18, 22, 30, 24 };

int totale = voti.Count();
int somma = voti.Sum();
double media = voti.Average();
int minimo = voti.Min();
int massimo = voti.Max();
```

Puoi anche passare una condizione a `Count`:

```csharp
int sufficienti = voti.Count(v => v >= 18);
```

### Distinct

Rimuove i duplicati dalla sequenza.

```csharp
int[] numeri = new int[] { 1, 2, 2, 3, 3, 3 };

IEnumerable<int> unici = numeri.Distinct();
```

### Take e Skip

`Take` prende i primi N elementi. `Skip` ne salta N e restituisce il resto.
Sono spesso usati insieme per implementare la paginazione.

```csharp
int[] numeri = new int[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

IEnumerable<int> primiTre = numeri.Take(3);
IEnumerable<int> saltaPrimiTre = numeri.Skip(3);

// Seconda pagina da 3 elementi
IEnumerable<int> pagina = numeri.Skip(3).Take(3);
```

### Join

Mette in corrispondenza elementi di due sequenze in base a una chiave comune.

```csharp
var studenti = new[]
{
    new { Id = 1, Nome = "Alice" },
    new { Id = 2, Nome = "Bob" },
    new { Id = 3, Nome = "Carlo" }
};

var votiStudenti = new[]
{
    new { IdStudente = 1, Voto = 28 },
    new { IdStudente = 2, Voto = 21 },
    new { IdStudente = 3, Voto = 30 }
};

var risultati = studenti.Join(
    votiStudenti,
    s => s.Id,
    v => v.IdStudente,
    (s, v) => new { s.Nome, v.Voto }
);

foreach (var r in risultati)
{
    Console.WriteLine($"{r.Nome}: {r.Voto}");
}
```

### ToList e ToArray

Materializzano la query: eseguono immediatamente la sequenza e restituiscono una lista o un array.

```csharp
int[] numeri = new int[] { 1, 2, 3, 4, 5 };

List<int> listaFiltrata = numeri.Where(n => n > 2).ToList();
int[] arrayFiltrato = numeri.Where(n => n > 2).ToArray();
```

---

## Esercizi guidati

Gli esercizi seguono una difficoltà progressiva.
Per ogni traccia trovi una guida operativa e la soluzione finale.

### 1. IEnumerable: iterare una sequenza

**Traccia**

Stampa tutti i numeri di un array usando `IEnumerable<int>` come tipo della variabile.

**Come ragionare**

1. Dichiara la variabile con `IEnumerable<int>`.
2. Assegna un array.
3. Itera con `foreach`.

**Soluzione finale**

```csharp
IEnumerable<int> numeri = new int[] { 10, 20, 30, 40 };

foreach (int n in numeri)
{
    Console.WriteLine(n);
}
```

### 2. ICollection: aggiungere e contare

**Traccia**

Crea una collezione di nomi, aggiungine tre e stampa il numero totale.

**Come ragionare**

1. Usa `ICollection<string>` come tipo.
2. Istanzia con `new List<string>()`.
3. Usa `Add` e `Count`.

**Soluzione finale**

```csharp
ICollection<string> nomi = new List<string>();
nomi.Add("Alice");
nomi.Add("Bob");
nomi.Add("Carlo");

Console.WriteLine($"Elementi: {nomi.Count}");
```

### 3. Where: filtrare i voti sufficienti

**Traccia**

Da un array di voti, seleziona solo quelli maggiori o uguali a 18.

**Come ragionare**

1. Usa `Where` con una lambda.
2. Itera sul risultato.

**Soluzione finale**

```csharp
int[] voti = new int[] { 15, 18, 22, 10, 30 };

IEnumerable<int> sufficienti = voti.Where(v => v >= 18);

foreach (int v in sufficienti)
{
    Console.WriteLine(v);
}
```

### 4. Select: trasformare nomi in maiuscolo

**Traccia**

Data una lista di nomi in minuscolo, producine una nuova con tutti i caratteri in maiuscolo.

**Come ragionare**

1. Usa `Select` con `ToUpper()`.
2. Non modificare la sorgente originale.

**Soluzione finale**

```csharp
string[] nomi = new string[] { "alice", "bob", "carlo" };

IEnumerable<string> maiuscoli = nomi.Select(n => n.ToUpper());

foreach (string nome in maiuscoli)
{
    Console.WriteLine(nome);
}
```

### 5. OrderBy + Where: voti sufficienti in ordine decrescente

**Traccia**

Filtra i voti sufficienti e stampali dal più alto al più basso.

**Come ragionare**

1. Prima applica `Where`.
2. Poi applica `OrderByDescending`.
3. L'ordine degli operatori conta.

**Soluzione finale**

```csharp
int[] voti = new int[] { 15, 28, 22, 10, 30, 18 };

IEnumerable<int> risultato = voti
    .Where(v => v >= 18)
    .OrderByDescending(v => v);

foreach (int v in risultato)
{
    Console.WriteLine(v);
}
```

### 6. Count, Any, All: statistiche su una lista

**Traccia**

Verifica se tutti i voti sono sufficienti, se almeno uno è 30, e quanti sono sufficienti.

**Come ragionare**

1. Usa `All` per il controllo globale.
2. Usa `Any` per cercare un singolo caso.
3. Usa `Count` con condizione.

**Soluzione finale**

```csharp
int[] voti = new int[] { 18, 22, 30, 25 };

bool tuttiSufficienti = voti.All(v => v >= 18);
bool haTrenta = voti.Any(v => v == 30);
int quanti = voti.Count(v => v >= 18);

Console.WriteLine($"Tutti sufficienti: {tuttiSufficienti}");
Console.WriteLine($"Ha 30: {haTrenta}");
Console.WriteLine($"Sufficienti: {quanti}");
```

### 7. GroupBy: raggruppare per iniziale

**Traccia**

Raggruppa una lista di nomi per la loro iniziale e stampa ogni gruppo.

**Come ragionare**

1. Usa `GroupBy` con `n[0]`.
2. Itera sui gruppi con `foreach` annidato.

**Soluzione finale**

```csharp
string[] nomi = new string[] { "Alice", "Anna", "Bob", "Bruno", "Carlo" };

IEnumerable<IGrouping<char, string>> gruppi = nomi.GroupBy(n => n[0]);

foreach (IGrouping<char, string> gruppo in gruppi)
{
    Console.WriteLine($"Iniziale {gruppo.Key}:");

    foreach (string nome in gruppo)
    {
        Console.WriteLine($"  {nome}");
    }
}
```

### 8. Take e Skip: paginazione manuale

**Traccia**

Data una lista di 10 numeri, stampa la seconda pagina di 3 elementi.

**Come ragionare**

1. Salta i primi 3 con `Skip(3)`.
2. Prendi i successivi 3 con `Take(3)`.

**Soluzione finale**

```csharp
int[] numeri = new int[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

IEnumerable<int> pagina = numeri.Skip(3).Take(3);

foreach (int n in pagina)
{
    Console.WriteLine(n);
}
```

### 9. IDictionary + LINQ: media per studente

**Traccia**

Hai un dizionario che associa il nome di uno studente a una lista di voti.
Calcola la media per ogni studente e stampa il risultato.

**Come ragionare**

1. Itera sul dizionario con `foreach`.
2. Per ogni coppia chiave-valore usa `Average()` sulla lista.
3. Stampa nome e media.

**Soluzione finale**

```csharp
using System.Collections.Generic;

IDictionary<string, int[]> registro = new Dictionary<string, int[]>
{
    { "Alice", new int[] { 18, 24, 30 } },
    { "Bob", new int[] { 22, 21, 19 } },
    { "Carlo", new int[] { 15, 18, 20 } }
};

foreach (KeyValuePair<string, int[]> voce in registro)
{
    double media = voce.Value.Average();
    Console.WriteLine($"{voce.Key}: media {media:F1}");
}
```

### 10. Esercizio complesso: analisi di un registro voti

**Traccia**

Hai una lista di oggetti con nome e voto.
Devi:

1. filtrare solo i promossi
2. ordinarli per voto decrescente
3. stampare il nome del primo classificato
4. calcolare la media dei voti dei promossi
5. stampare quanti sono stati rimandati

**Come ragionare passo per passo**

1. Definisci la struttura dei dati con un array di tipi anonimi.
2. Applica `Where` per separare promossi e rimandati.
3. Usa `OrderByDescending` per ordinare i promossi.
4. Usa `First` per il primo classificato.
5. Usa `Average` e `Count` per le statistiche.
6. Materializza solo quando serve con `ToList()`.
7. Separa ogni risposta in un passaggio distinto per chiarezza.

**Soluzione finale**

```csharp
var studenti = new[]
{
    new { Nome = "Alice", Voto = 28 },
    new { Nome = "Bob", Voto = 17 },
    new { Nome = "Carlo", Voto = 30 },
    new { Nome = "Daniele", Voto = 15 },
    new { Nome = "Elena", Voto = 22 }
};

var promossi = studenti
    .Where(s => s.Voto >= 18)
    .OrderByDescending(s => s.Voto)
    .ToList();

var primoClassificato = promossi.First();
double mediaPromossi = promossi.Average(s => s.Voto);
int rimandati = studenti.Count(s => s.Voto < 18);

Console.WriteLine($"Primo classificato: {primoClassificato.Nome} ({primoClassificato.Voto})");
Console.WriteLine($"Media promossi: {mediaPromossi:F1}");
Console.WriteLine($"Rimandati: {rimandati}");
```

Questo esercizio mette insieme i concetti fondamentali: interfacce come sorgente dati, LINQ per interrogarla, e ragionamento progressivo per rispondere a più domande con un'unica pipeline.
