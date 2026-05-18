# Introduzione a .NET e C#

Questa dispensa introduce i concetti fondamentali della piattaforma .NET e del linguaggio C#.
L'obiettivo è fornire una base solida per iniziare a scrivere programmi semplici, leggere codice esistente e affrontare con metodo i primi esercizi.

I temi trattati in questa sezione sono:

- cos'è .NET, come si è evoluto nel tempo e perché è nato
- buone pratiche iniziali di programmazione, come commenti, convenzioni e nomenclature
- cenni storici sul linguaggio C#
- tipi primitivi e tipi derivati, inclusi vettori, matrici e struct, con inizializzazione esplicita in C#
- costrutti condizionali come `if` e `switch`
- costrutti iterativi come `do...while`, `while` e `for`
- esempi guidati di difficoltà crescente, con suggerimenti per arrivare a una soluzione in modo ordinato

## Un esempio iniziale

Prima di entrare nei dettagli, confrontiamo due versioni dello stesso piccolo frammento.

```csharp
// Codice poco chiaro
int x = 0;
if (x > 0)
{
	Console.WriteLine("OK");
}
```

```csharp
// Codice più leggibile
int numeroDiElementi = 0;
if (numeroDiElementi > 0)
{
	Console.WriteLine("La collezione contiene elementi.");
}
```

La logica è identica, ma il secondo esempio è più facile da capire perché il nome della variabile dice cosa rappresenta davvero.

## Obiettivi della dispensa

Alla fine di questa introduzione dovresti essere in grado di:

1. riconoscere il ruolo di .NET come piattaforma di sviluppo
2. distinguere tra concetti di linguaggio e concetti di piattaforma
3. leggere e scrivere piccoli programmi in C# con una struttura chiara
4. scegliere il costrutto più adatto tra le alternative di base
5. impostare un ragionamento corretto di fronte a un esercizio nuovo

Un obiettivo importante, spesso trascurato, è imparare a distinguere il codice che funziona dal codice che è anche chiaro e mantenibile.

## Cos'è .NET

.NET è una piattaforma di sviluppo pensata per creare applicazioni di vario tipo: console, desktop, web, servizi e molto altro.
Non è solo un linguaggio, ma un insieme di strumenti, librerie e runtime che semplificano la costruzione e l'esecuzione dei programmi.

Nel tempo .NET si è evoluto da un ecosistema fortemente legato a Windows a una piattaforma moderna, multipiattaforma e open source.
Questo passaggio ha reso possibile usare gli stessi strumenti su sistemi operativi diversi e adottare un modello di sviluppo più uniforme.

Un programma .NET di base può essere molto semplice:

```csharp
Console.WriteLine("Ciao da .NET");
```

Questo è un esempio utile per ricordare che l'obiettivo iniziale non è scrivere molto codice, ma capire bene cosa succede riga per riga.

## Perché nasce

L'idea alla base di .NET è ridurre la complessità dello sviluppo software fornendo:

- una base comune per eseguire applicazioni scritte in linguaggi compatibili
- librerie standard per evitare di riscrivere funzioni già note
- un ambiente di esecuzione controllato e più sicuro
- strumenti per organizzare meglio il codice e renderlo più manutenibile

In pratica, .NET aiuta lo sviluppatore a concentrarsi sulla logica applicativa invece che sui dettagli di basso livello.

Per esempio, se vuoi leggere un numero e fare una verifica, ti interessa soprattutto la logica:

```csharp
int eta = 20;

if (eta >= 18)
{
	Console.WriteLine("Maggiore età");
}
```

Non devi preoccuparti di come il sistema operativo gestisce internamente la stampa a video: la piattaforma ti mette a disposizione gli strumenti già pronti.

## Il linguaggio C#

C# è il linguaggio più diffuso nell'ecosistema .NET.
Nasce con l'obiettivo di offrire un linguaggio moderno, leggibile e adatto sia a piccoli programmi sia ad applicazioni complesse.

Tra le sue caratteristiche più importanti troviamo:

- sintassi chiara e orientata alla leggibilità
- tipizzazione forte
- ampia integrazione con le librerie della piattaforma

Per questo motivo C# è adatto a introdurre concetti di programmazione che torneranno utili anche in altri linguaggi.

Un esempio di codice scritto bene in C# è leggibile anche senza spiegazioni esterne:

```csharp
int punteggio = 28;

if (punteggio >= 18)
{
	Console.WriteLine("Esame superato");
}
else
{
	Console.WriteLine("Esame non superato");
}
```

Un esempio peggiore usa nomi vaghi e valori non spiegati:

```csharp
int a = 28;

if (a >= 18)
{
	Console.WriteLine("OK");
}
else
{
	Console.WriteLine("NO");
}
```

Il codice funziona, ma comunica meno informazioni a chi lo legge.

## Buone pratiche iniziali

Quando si comincia a programmare, il codice non dovrebbe essere solo corretto, ma anche comprensibile.
Le prime abitudini importanti riguardano i commenti, le convenzioni e la scelta dei nomi.

In C# le convenzioni più comuni aiutano a rendere il codice uniforme e facile da leggere:

- tipi, metodi e costanti usano in genere `PascalCase`
- variabili locali e parametri usano in genere `camelCase`
- campi privati usano spesso `camelCase` con prefisso `_`
- i nomi devono descrivere il contenuto o il comportamento, non essere generici

Esempio corretto:

```csharp
const int MaxTentativi = 3;
int tentativiEffettuati = 0;
string nomeUtente = "Luca";
```

Esempio meno adatto:

```csharp
const int max_tentativi = 3;
int x = 0;
string s = "Luca";
```

Nel secondo caso il codice è ancora valido, ma i nomi non seguono le convenzioni di C# e rendono più difficile capire il significato dei valori.

I nomi di variabili, metodi, parametri e costanti dovrebbero descrivere chiaramente il loro significato.
Un codice leggibile è più facile da correggere, estendere e spiegare.

I commenti vanno usati con criterio: servono a chiarire intenzioni non ovvie, non a ripetere ciò che il codice dice già da solo.

### Commenti: quando servono e quando no

Commento inutile:

```csharp
int eta = 18; // assegna 18 alla variabile eta
```

Commento utile:

```csharp
// Il requisito richiede la maggiore età per accedere alla funzione.
int eta = 18;
```

In generale, se un commento descrive solo ciò che il codice già dice chiaramente, è meglio migliorare il codice invece del commento.

### Nomi: cattivi e buoni esempi

```csharp
int n = 5;
int c = 0;
```

```csharp
int numeroElementi = 5;
int elementiStampati = 0;
```

Nel secondo caso i nomi aiutano a capire il ruolo delle variabili, soprattutto quando il programma cresce.

## Tipi e strutture fondamentali

Uno dei primi passi nello studio di C# è capire come rappresentare i dati.
La dispensa affronta i tipi primitivi, i tipi derivati e le strutture dati più semplici.

Tra gli argomenti principali ci sono:

- numeri interi e decimali
- valori booleani
- stringhe
- vettori e matrici con sintassi esplicita di inizializzazione in C#
- struct come contenitori di dati correlati

Capire questi elementi è essenziale per costruire programmi che leggono, elaborano e producono informazioni.

### Tipi primitivi

Esempio di valori di base:

```csharp
int eta = 21;
double prezzo = 19.99;
bool isAttivo = true;
char iniziale = 'A';
string nome = "Luca";
```

Un uso poco chiaro potrebbe mescolare tipi e significati senza criterio:

```csharp
int x = 21;
double y = 19.99;
bool z = true;
```

I tipi sono corretti, ma i nomi non raccontano nulla.

### Vettori

Un vettore contiene più valori dello stesso tipo.
In C# si dichiara con le parentesi quadre nel tipo, per esempio `int[]`, e in questa dispensa lo inizializziamo sempre in modo esplicito con `new int[] { ... }`.

```csharp
int[] voti = new int[] { 18, 24, 30 };
// oppure int[] voti = [ 18, 24, 30 ];
// oppure ancora int[] voti = new int[3]; // poi inizializzo i singoli elementi
Console.WriteLine(voti[0]);
```

Esempio meno adatto:

```csharp
int[] a = new int[] { 18, 24, 30 };
Console.WriteLine(a[0]);
```

Il problema non è l'array, ma il nome troppo generico.

### Matrici

Una matrice è utile quando i dati hanno righe e colonne.
In C# si dichiara con `[,]`, per esempio `int[,]`, e in questa dispensa la inizializziamo sempre con `new int[,] { ... }`.

```csharp
int[,] tabella = new int[,]
{
	{ 1, 2, 3 },
	{ 4, 5, 6 }
};

// oppure
tabella = [
    [1, 2, 3],
    [4, 5, 6]
]

Console.WriteLine(tabella[1, 2]);
```

Se la struttura dei dati non è davvero bidimensionale, una matrice può essere una scelta inutile rispetto a un vettore inizializzato con `new int[] { ... }`.

### Struct

Una struct è utile quando vuoi raggruppare dati correlati.

```csharp
struct Punto
{
	public int X;
	public int Y;
}

Punto p;
p.X = 3;
p.Y = 7;
```

Un approccio meno chiaro sarebbe tenere i valori separati senza significato comune:

```csharp
int x = 3;
int y = 7;
```

Se i due valori rappresentano sempre la stessa entità logica, raggrupparli migliora la comprensione.

## Controllo del flusso

Un programma non esegue sempre le istruzioni nello stesso ordine.
I costrutti condizionali e iterativi permettono di prendere decisioni e ripetere operazioni in modo controllato.

La dispensa introduce:

- `if` per eseguire blocchi diversi in base a una condizione
- `switch` per gestire più casi alternativi
- `while` e `do...while` per ripetere finché una condizione è valida
- `for` per cicli con contatore

Questi strumenti sono la base della logica di programmazione nei primi esercizi.

### if

Esempio chiaro:

```csharp
int eta = 17;

if (eta >= 18)
{
	Console.WriteLine("Accesso consentito");
}
else
{
	Console.WriteLine("Accesso negato");
}
```

Esempio meno buono, perché usa una condizione negativa non necessaria:

```csharp
int eta = 17;

if (!(eta < 18))
{
	Console.WriteLine("Accesso consentito");
}
else
{
	Console.WriteLine("Accesso negato");
}
```

La seconda versione funziona, ma si legge più lentamente.

### switch

```csharp
string giorno = "lunedi";

switch (giorno)
{
	case "lunedi":
		Console.WriteLine("Inizio settimana");
		break;
	case "venerdi":
		Console.WriteLine("Fine settimana vicina");
		break;
	default:
		Console.WriteLine("Giorno normale");
		break;
}
```

Se i casi sono molti e tutti confrontano la stessa variabile, `switch` è spesso più leggibile di una lunga catena di `if`.

### while

```csharp
int contatore = 0;

while (contatore < 3)
{
	Console.WriteLine(contatore);
	contatore++;
}
```

Un errore comune è dimenticare l'aggiornamento della variabile di controllo:

```csharp
int contatore = 0;

while (contatore < 3)
{
	Console.WriteLine(contatore);
}
```

In questo caso il ciclo non termina mai.

### do...while

```csharp
string risposta;

do
{
	Console.Write("Scrivi sì o no: ");
	risposta = Console.ReadLine();
}
while (risposta != "sì" && risposta != "no");
```

Questo costrutto è utile quando il blocco deve essere eseguito almeno una volta.

### for

```csharp
for (int i = 0; i < 5; i++)
{
	Console.WriteLine(i);
}
```

Esempio meno adatto per un ciclo con contatore:

```csharp
int i = 0;
while (i < 5)
{
	Console.WriteLine(i);
	i++;
}
```

La seconda soluzione non è sbagliata, ma `for` esprime meglio l'idea di ciclo controllato da un indice.

## Metodo di lavoro sugli esercizi

Negli esempi finali la priorità non è solo arrivare alla risposta, ma imparare un metodo.
Di fronte a un problema conviene sempre:

1. leggere bene la traccia
2. individuare input, output e vincoli
3. scomporre il problema in passaggi semplici
4. scrivere una prima soluzione anche se minimale
5. verificare il risultato con casi di prova

Questo approccio è più utile della ricerca della soluzione immediata, soprattutto nelle fasi iniziali dell'apprendimento.

Esempio di ragionamento su un esercizio semplice:

```csharp
int numero = 6;

if (numero % 2 == 0)
{
	Console.WriteLine("Numero pari");
}
else
{
	Console.WriteLine("Numero dispari");
}
```

Prima di scrivere il codice, conviene chiedersi:

- qual è il dato in ingresso?
- qual è la regola da applicare?
- qual è l'output atteso?

Se queste risposte sono chiare, il codice diventa molto più semplice da costruire.

## Come usare questa dispensa

Questa introduzione vuole essere un punto di partenza, non un riferimento esaustivo.
L'idea è affrontare un concetto alla volta, costruendo gradualmente familiarità con la sintassi e con il ragionamento algoritmico.

Se alcuni argomenti risultano inizialmente poco chiari, è normale: le sezioni successive riprendono gli stessi temi con esempi concreti e con un livello di dettaglio maggiore.

In pratica, questa introduzione deve essere letta come una guida operativa: osserva gli esempi, confronta il codice buono con quello meno chiaro e prova a riscrivere i frammenti con nomi migliori o con una struttura più leggibile.

## Esercizi guidati

Gli esercizi che seguono sono pensati per essere letti in ordine di difficoltà crescente.
Per ogni traccia trovi un percorso operativo, cioè i passaggi mentali da seguire prima di scrivere il codice, e poi la soluzione finale.

### 1. if semplice: verificare la maggiore età

**Traccia**

Ricevi in ingresso un'età e stampa se la persona è maggiorenne oppure no.

**Come ragionare**

1. Identifica il dato: un numero intero chiamato `eta`.
2. Definisci la regola: la persona è maggiorenne se `eta >= 18`.
3. Scegli il costrutto: basta un `if` semplice con `else`.
4. Scrivi prima il messaggio e poi la condizione.

**Soluzione finale**

```csharp
int eta = 17;

if (eta >= 18)
{
	Console.WriteLine("Maggiorenne");
}
else
{
	Console.WriteLine("Minorenne");
}
```

### 2. if con condizione booleana composta: accesso a una promozione

**Traccia**

Una persona può accedere a una promozione solo se ha la tessera attiva e ha speso almeno 50 euro.

**Come ragionare**

1. Usa una variabile booleana per lo stato della tessera.
2. Usa una variabile numerica per la spesa.
3. Combina le due condizioni con `&&`.
4. Stampa un messaggio chiaro per i due casi.

**Soluzione finale**

```csharp
bool tesseraAttiva = true;
double spesaTotale = 52.5;

if (tesseraAttiva && spesaTotale >= 50)
{
	Console.WriteLine("Promozione valida");
}
else
{
	Console.WriteLine("Promozione non valida");
}
```

### 3. switch: tradurre il numero del mese

**Traccia**

Dato un numero da 1 a 3, stampa il nome del mese corrispondente nel trimestre iniziale dell'anno.

**Come ragionare**

1. La variabile in ingresso è un intero chiamato `mese`.
2. I casi possibili sono pochi e distinti.
3. `switch` è più leggibile di una catena di `if`.
4. Inserisci un `default` per i valori non previsti.

**Soluzione finale**

```csharp
int mese = 2;

switch (mese)
{
	case 1:
		Console.WriteLine("Gennaio");
		break;
	case 2:
		Console.WriteLine("Febbraio");
		break;
	case 3:
		Console.WriteLine("Marzo");
		break;
	default:
		Console.WriteLine("Mese non valido per questo esercizio");
		break;
}
```

### 4. do...while: inserire un voto valido

**Traccia**

Continua a chiedere un voto finché non viene inserito un valore compreso tra 0 e 30.

**Come ragionare**

1. Il ciclo deve partire almeno una volta, quindi usa `do...while`.
2. La variabile da controllare è il voto.
3. Dopo ogni inserimento verifica il range.
4. Esci solo quando il voto è valido.

**Soluzione finale**

```csharp
int voto;

do
{
	Console.Write("Inserisci un voto da 0 a 30: ");
	string input = Console.ReadLine() ?? "";

	if (!int.TryParse(input, out voto))
	{
		voto = -1;
	}
}
while (voto < 0 || voto > 30);

Console.WriteLine($"Voto accettato: {voto}");
```

### 5. while: contare fino a un limite

**Traccia**

Stampa i numeri da 1 a 5 usando un ciclo `while`.

**Come ragionare**

1. Inizializza il contatore a 1.
2. Continua finché il contatore è minore o uguale a 5.
3. Stampa il valore corrente.
4. Incrementa il contatore alla fine del ciclo.

**Soluzione finale**

```csharp
int contatore = 1;

while (contatore <= 5)
{
	Console.WriteLine(contatore);
	contatore++;
}
```

### 6. for: sommare i primi numeri naturali

**Traccia**

Calcola e stampa la somma dei numeri da 1 a 10.

**Come ragionare**

1. Usa una variabile accumulatore inizializzata a 0.
2. Scorri i numeri da 1 a 10 con un `for`.
3. A ogni passo aggiungi il valore corrente alla somma.
4. Alla fine stampa il risultato.

**Soluzione finale**

```csharp
int somma = 0;

for (int i = 1; i <= 10; i++)
{
	somma += i;
}

Console.WriteLine($"Somma finale: {somma}");
```

### 7. if + ciclo: stampare i numeri pari fino a un limite

**Traccia**

Dato un limite, stampa solo i numeri pari compresi tra 1 e il limite.

**Come ragionare**

1. Usa un ciclo per attraversare tutti i numeri.
2. Dentro il ciclo controlla la parità con `if`.
3. Stampa solo quando il numero è divisibile per 2.
4. Il resto dei numeri va ignorato.

**Soluzione finale**

```csharp
int limite = 10;

for (int i = 1; i <= limite; i++)
{
	if (i % 2 == 0)
	{
		Console.WriteLine(i);
	}
}
```

### 8. while + condizione: cercare un valore in un intervallo

**Traccia**

Conta quanti numeri da 1 a 20 sono multipli di 3.

**Come ragionare**

1. Parti da 1 con un contatore.
2. Usa un secondo contatore per i multipli trovati.
3. Ogni volta che trovi un multiplo di 3 incrementa il secondo contatore.
4. Al termine stampa il totale.

**Soluzione finale**

```csharp
int numero = 1;
int multipliDiTre = 0;

while (numero <= 20)
{
	if (numero % 3 == 0)
	{
		multipliDiTre++;
	}

	numero++;
}

Console.WriteLine($"Multipli di 3 trovati: {multipliDiTre}");
```

### 9. for + condizione: calcolare solo i voti sufficienti

**Traccia**

Data una serie di voti, conta quanti sono sufficienti, cioè maggiori o uguali a 18.

**Come ragionare**

1. Metti i voti in un vettore.
2. Usa un `for` per scorrere tutti gli elementi.
3. Per ogni voto applica una condizione `if`.
4. Incrementa il contatore solo quando il voto è sufficiente.

**Soluzione finale**

```csharp
int[] voti = new int[] { 15, 18, 21, 12, 30 };
int promossi = 0;

for (int i = 0; i < voti.Length; i++)
{
	if (voti[i] >= 18)
	{
		promossi++;
	}
}

Console.WriteLine($"Voti sufficienti: {promossi}");
```

### 10. Esercizio complesso: gestione di una lista di esami

**Traccia**

Hai una lista di voti di esame. Devi:

1. contare quanti voti sono sufficienti
2. contare quanti voti sono insufficienti
3. calcolare la somma dei voti sufficienti
4. stampare la media dei soli voti sufficienti, se ce n'è almeno uno

**Come ragionare passo per passo**

1. La struttura di base è un vettore di voti, perché i dati sono una sequenza lineare.
2. Servono tre variabili di lavoro: un contatore per i sufficienti, uno per gli insufficienti e una somma parziale.
3. Scorri il vettore con un `for`, perché devi visitare ogni elemento una sola volta.
4. Per ogni voto, applica una condizione: se è almeno 18, aggiorna i dati dei sufficienti; altrimenti aggiorna quelli degli insufficienti.
5. Dopo il ciclo, verifica se hai trovato almeno un voto sufficiente prima di dividere la somma.
6. Stampa i risultati in modo separato e chiaro.

**Soluzione finale**

```csharp
int[] voti = new int[] { 18, 24, 30, 17, 12, 21, 19 };
int votiSufficienti = 0;
int votiInsufficienti = 0;
int sommaSufficienti = 0;

for (int i = 0; i < voti.Length; i++)
{
	int voto = voti[i];

	if (voto >= 18)
	{
		votiSufficienti++;
		sommaSufficienti += voto;
	}
	else
	{
		votiInsufficienti++;
	}
}

Console.WriteLine($"Voti sufficienti: {votiSufficienti}");
Console.WriteLine($"Voti insufficienti: {votiInsufficienti}");

if (votiSufficienti > 0)
{
	double mediaSufficienti = (double)sommaSufficienti / votiSufficienti;
	Console.WriteLine($"Media dei voti sufficienti: {mediaSufficienti}");
}
else
{
	Console.WriteLine("Nessun voto sufficiente, media non calcolabile.");
}
```

Questo esercizio riassume il metodo usato nella dispensa: analisi dei dati, scelta del costrutto giusto, separazione delle responsabilità e controllo finale del risultato.
