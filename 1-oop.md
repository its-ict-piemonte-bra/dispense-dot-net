# Programmazione orientata agli oggetti in C#

Questa dispensa introduce la programmazione orientata agli oggetti, spesso indicata come OOP, nel contesto di C# e di .NET.
L'obiettivo non è solo imparare la sintassi, ma capire perché questo modo di organizzare il codice è utile quando un programma cresce.

L'idea centrale è semplice: invece di scrivere solo sequenze di istruzioni, si modellano entità del problema con dati e comportamenti raccolti in modo coerente.

## Perché usare l'OOP

L'OOP aiuta a dividere un programma in parti più piccole e più comprensibili.
Questo rende più semplice:

- ragionare su dati e comportamenti insieme
- evitare codice disperso e difficile da mantenere
- riusare soluzioni già definite
- modificare una parte del programma senza toccare tutto il resto

Un esempio non orientato agli oggetti tende a separare troppo i dati dalle operazioni:

```csharp
string nome = "Luca";
int eta = 20;

Console.WriteLine(nome + " ha " + eta + " anni");
```

Un approccio più coerente raggruppa dati e comportamento in un'unica entità:

```csharp
Studente studente = new Studente("Luca", 20);
studente.Presentati();
```

In questo caso il codice comunica meglio l'intenzione: stiamo lavorando con uno studente, non con valori scollegati.

## Classe e oggetto

La classe è il modello, cioè la definizione di una nuova struttura dati con i relativi comportamenti.
L'oggetto è un'istanza concreta di quella classe.

Se la classe è il progetto di un edificio, l'oggetto è l'edificio costruito seguendo quel progetto.

### Esempio corretto

```csharp
class Studente
{
	public string Nome { get; }
	public int Eta { get; }

	public Studente(string nome, int eta)
	{
		Nome = nome;
		Eta = eta;
	}

	public void Presentati()
	{
		Console.WriteLine($"Mi chiamo {Nome} e ho {Eta} anni.");
	}
}

Studente studente = new Studente("Luca", 20);
studente.Presentati();
```

### Esempio meno chiaro

```csharp
class A
{
	public string n;
	public int e;

	public A(string x, int y)
	{
		n = x;
		e = y;
	}

	public void P()
	{
		Console.WriteLine(n + " " + e);
	}
}
```

Il secondo esempio funziona ancora, ma i nomi non aiutano a capire il ruolo dei dati e del metodo.

## Attributi e metodi

Gli attributi rappresentano lo stato dell'oggetto.
I metodi rappresentano le azioni che l'oggetto può compiere.

Un attributo risponde alla domanda: che cosa sa l'oggetto?
Un metodo risponde alla domanda: che cosa fa l'oggetto?

### Esempio

```csharp
class Auto
{
	public string Marca { get; }
	public int Velocita { get; private set; }

	public Auto(string marca)
	{
		Marca = marca;
		Velocita = 0;
	}

	public void Accelera(int incremento)
	{
		Velocita += incremento;
	}
}
```

### Esempio meno adatto

```csharp
class Auto
{
	public string a;
	public int b;

	public void c(int d)
	{
		b += d;
	}
}
```

Il problema non è solo estetico: con nomi troppo brevi o generici, il significato si perde rapidamente.

## Incapsulamento

L'incapsulamento consiste nel nascondere i dettagli interni e nel mostrare solo ciò che serve usare dall'esterno.
In pratica, evitiamo che il codice esterno modifichi liberamente lo stato interno dell'oggetto.

### Esempio corretto

```csharp
class ContoBancario
{
	private decimal _saldo;

	public ContoBancario(decimal saldoIniziale)
	{
		_saldo = saldoIniziale;
	}

	public decimal Saldo => _saldo;

	public void Versa(decimal importo)
	{
		if (importo > 0)
		{
			_saldo += importo;
		}
	}
}
```

### Esempio meno adatto

```csharp
class ContoBancario
{
	public decimal saldo;
}
```

Nel secondo caso chiunque può modificare il saldo senza controllo.
Questo rende il programma più fragile e più difficile da correggere.

## Costruttori

Il costruttore è un metodo speciale che viene eseguito quando si crea un nuovo oggetto.
Serve a inizializzare lo stato iniziale in modo coerente.

### Esempio corretto

```csharp
class Libro
{
	public string Titolo { get; }
	public string Autore { get; }

	public Libro(string titolo, string autore)
	{
		Titolo = titolo;
		Autore = autore;
	}
}
```

### Esempio meno adatto

```csharp
class Libro
{
	public string Titolo;
	public string Autore;

	public void ImpostaDati(string titolo, string autore)
	{
		Titolo = titolo;
		Autore = autore;
	}
}
```

Questo secondo approccio sposta l'inizializzazione in un metodo separato, quindi l'oggetto può esistere per un po' in uno stato incompleto.

## Proprietà

Le proprietà sono un modo ordinato per leggere e, quando serve, modificare i dati di un oggetto.
Sono molto utili perché espongono un'interfaccia semplice verso l'esterno senza rinunciare al controllo interno.

### Esempio corretto

```csharp
class Prodotto
{
	public string Nome { get; }
	public decimal Prezzo { get; private set; }

	public Prodotto(string nome, decimal prezzo)
	{
		Nome = nome;
		Prezzo = prezzo;
	}

	public void ApplicaSconto(decimal sconto)
	{
		if (sconto > 0 && sconto < Prezzo)
		{
			Prezzo -= sconto;
		}
	}
}
```

### Esempio meno adatto

```csharp
class Prodotto
{
	public string Nome;
	public decimal Prezzo;
}
```

Qui i dati sono esposti direttamente e possono essere modificati senza alcuna regola.

## Accessibilità

In C# i modificatori di accesso definiscono chi può vedere o usare membri di una classe.
I più importanti per iniziare sono `public` e `private`.

- `public`: accessibile dall'esterno
- `private`: accessibile solo all'interno della classe

### Esempio

```csharp
class Persona
{
	private int _eta;

	public Persona(int eta)
	{
		_eta = eta;
	}

	public bool EMaggioreEta()
	{
		return _eta >= 18;
	}
}
```

### Esempio meno adatto

```csharp
class Persona
{
	public int eta;
}
```

Il campo pubblico espone troppo lo stato interno e lascia poco controllo sulla validità dei dati.

## Metodi

I metodi rappresentano comportamenti o operazioni eseguibili sull'oggetto.
Un buon metodo fa una sola cosa in modo chiaro.

### Esempio corretto

```csharp
class Rettangolo
{
	public double Base { get; }
	public double Altezza { get; }

	public Rettangolo(double baseRettangolo, double altezza)
	{
		Base = baseRettangolo;
		Altezza = altezza;
	}

	public double CalcolaArea()
	{
		return Base * Altezza;
	}
}
```

### Esempio meno adatto

```csharp
class Rettangolo
{
	public double Base;
	public double Altezza;

	public void FaiTutto()
	{
		Console.WriteLine(Base * Altezza);
	}
}
```

Il nome `FaiTutto` non dice nulla e il metodo mescola calcolo e stampa, rendendo il comportamento meno riusabile.

## Statico e istanza

Un membro di istanza appartiene a un oggetto specifico.
Un membro statico appartiene alla classe nel suo insieme.

### Esempio di istanza

```csharp
class Contatore
{
	public int Valore { get; private set; }

	public void Incrementa()
	{
		Valore++;
	}
}

Contatore primo = new Contatore();
Contatore secondo = new Contatore();
```

### Esempio statico

```csharp
class Calcoli
{
	public static int Quadrato(int numero)
	{
		return numero * numero;
	}
}

int risultato = Calcoli.Quadrato(5);
```

### Esempio meno adatto

```csharp
class Calcoli
{
	public int Quadrato(int numero)
	{
		return numero * numero;
	}
}
```

Se un'operazione non usa lo stato dell'oggetto, spesso è più chiaro renderla `static`.

## Ereditarietà

L'ereditarietà permette di creare una classe a partire da un'altra, riusando e specializzando il comportamento.
È utile quando esiste una relazione chiara di tipo generale/specifico.

### Esempio corretto

```csharp
class Veicolo
{
	public string Marca { get; }

	public Veicolo(string marca)
	{
		Marca = marca;
	}

	public void Avvia()
	{
		Console.WriteLine("Veicolo avviato");
	}
}

class Auto : Veicolo
{
	public Auto(string marca) : base(marca)
	{
	}
}
```

### Esempio meno adatto

```csharp
class Auto
{
	public string Marca { get; }
}

class Motocicletta
{
	public string Marca { get; }
}
```

Se due classi condividono davvero lo stesso concetto base, può essere più utile un modello comune invece di duplicare la stessa struttura.

## Protected internal

Il modificatore `protected internal` rende un membro accessibile sia alle classi derivate sia al codice che si trova nello stesso assembly.
Si usa quando vuoi esporre un dettaglio a chi estende la classe, ma senza renderlo completamente pubblico.

### Esempio corretto

```csharp
class Animale
{
	protected internal string Nome { get; }

	protected internal Animale(string nome)
	{
		Nome = nome;
	}
}

class Cane : Animale
{
	public Cane(string nome) : base(nome)
	{
	}
}
```

### Esempio meno adatto

```csharp
class Animale
{
	public string Nome { get; }
}
```

Se un membro serve solo a chi deriva dalla classe o al codice interno dell'assembly, esporlo come `public` allarga troppo la superficie d'uso.

## Sealed

Il modificatore `sealed` impedisce di derivare ulteriormente una classe.
È utile quando una classe è già completa e non vuoi che venga specializzata oltre.

### Esempio corretto

```csharp
sealed class UtenteSemplificato
{
	public string Nome { get; }

	public UtenteSemplificato(string nome)
	{
		Nome = nome;
	}
}
```

### Esempio meno adatto

```csharp
class UtenteSemplificato
{
	public string Nome { get; }

	public UtenteSemplificato(string nome)
	{
		Nome = nome;
	}
}
```

Se non vuoi che una classe venga usata come base per altre classi, `sealed` lo dichiara in modo esplicito.

## Volatile

Il modificatore `volatile` si applica ai campi e segnala che il valore può cambiare in modo asincrono, per esempio in scenari concorrenti.
Non è un sostituto della sincronizzazione, ma uno strumento per casi specifici.

### Esempio corretto

```csharp
class Segnalatore
{
	private volatile bool _attivo;

	public void Attiva()
	{
		_attivo = true;
	}

	public bool EAttivo()
	{
		return _attivo;
	}
}
```

### Esempio meno adatto

```csharp
class Segnalatore
{
	private bool _attivo;
}
```

Se il valore viene letto e scritto da contesti diversi, la sola variabile normale potrebbe non essere sufficiente a descrivere chiaramente l'intenzione.

## Polimorfismo

Il polimorfismo consente di trattare oggetti diversi tramite un'interfaccia comune.
In pratica, chi usa il codice lavora con un comportamento generale, mentre ogni classe fornisce la propria versione.

### Esempio

```csharp
class Forma
{
	public virtual double CalcolaArea()
	{
		return 0;
	}
}

class Quadrato : Forma
{
	public double Lato { get; }

	public Quadrato(double lato)
	{
		Lato = lato;
	}

	public override double CalcolaArea()
	{
		return Lato * Lato;
	}
}

class Cerchio : Forma
{
	public double Raggio { get; }

	public Cerchio(double raggio)
	{
		Raggio = raggio;
	}

	public override double CalcolaArea()
	{
		return Math.PI * Raggio * Raggio;
	}
}
```

### Esempio meno adatto

```csharp
class Forme
{
	public double AreaQuadrato(double lato)
	{
		return lato * lato;
	}

	public double AreaCerchio(double raggio)
	{
		return Math.PI * raggio * raggio;
	}
}
```

Questo approccio raggruppa casi diversi in un'unica classe senza una struttura comune davvero utile.

## Interfacce

Un'interfaccia definisce un contratto: dice quali membri una classe deve offrire, ma non come li implementa.
È utile quando più classi diverse devono esporre lo stesso comportamento.

### Esempio

```csharp
interface IStampabile
{
	void Stampa();
}

class Fattura : IStampabile
{
	public void Stampa()
	{
		Console.WriteLine("Stampa della fattura");
	}
}

class Ricevuta : IStampabile
{
	public void Stampa()
	{
		Console.WriteLine("Stampa della ricevuta");
	}
}
```

### Esempio meno adatto

```csharp
class Fattura
{
	public void StampaFattura()
	{
	}
}

class Ricevuta
{
	public void StampaRicevuta()
	{
	}
}
```

Qui il comportamento comune esiste, ma non è modellato in modo uniforme.

## Generici

I generici permettono di scrivere classi, interfacce e metodi che funzionano con qualsiasi tipo, senza doverli riscrivere per ogni caso specifico.
Il tipo concreto viene specificato solo nel momento in cui si usa la classe o il metodo.

Il parametro di tipo si scrive tra `<` e `>` e per convenzione si chiama `T`, ma può avere qualsiasi nome.

### Perché sono stati introdotti

Senza i generici, per avere una lista di stringhe e una lista di interi servirebbero due classi distinte, oppure si usava `object` come tipo generico, perdendo la verifica del tipo da parte del compilatore.

**Approccio senza generici (meno sicuro)**

```csharp
class ContenitoreNumero
{
    private int _valore;

    public ContenitoreNumero(int valore)
    {
        _valore = valore;
    }

    public int GetValore()
    {
        return _valore;
    }
}

class ContenitoreStringa
{
    private string _valore;

    public ContenitoreStringa(string valore)
    {
        _valore = valore;
    }

    public string GetValore()
    {
        return _valore;
    }
}
```

Stesso codice, duplicato due volte solo per cambiare il tipo. Se la logica cambia, va aggiornata in entrambi i posti.

**Approccio con i generici (corretto)**

```csharp
class Contenitore<T>
{
    private T _valore;

    public Contenitore(T valore)
    {
        _valore = valore;
    }

    public T GetValore()
    {
        return _valore;
    }
}

Contenitore<int> numero = new Contenitore<int>(42);
Contenitore<string> testo = new Contenitore<string>("Ciao");

Console.WriteLine(numero.GetValore());
Console.WriteLine(testo.GetValore());
```

Una sola classe, usata con tipi diversi. Il compilatore verifica che i tipi siano coerenti a ogni utilizzo.

### Generici nei metodi

I generici si applicano anche ai singoli metodi, non solo alle classi.

```csharp
class Utilita
{
    public static void Stampa<T>(T valore)
    {
        Console.WriteLine(valore);
    }
}

Utilita.Stampa<int>(10);
Utilita.Stampa<string>("Ciao");
```

Il compilatore spesso riesce a dedurre il tipo dal parametro, quindi si può omettere:

```csharp
Utilita.Stampa(10);
Utilita.Stampa("Ciao");
```

### Vincoli sul tipo generico

A volte vuoi limitare quali tipi possono essere passati come `T`.
Il vincolo si scrive con la parola chiave `where`.

```csharp
class ContenitoreComparabile<T> where T : IComparable<T>
{
    private T _valore;

    public ContenitoreComparabile(T valore)
    {
        _valore = valore;
    }

    public bool EMaggiore(T altro)
    {
        return _valore.CompareTo(altro) > 0;
    }
}

ContenitoreComparabile<int> c = new ContenitoreComparabile<int>(10);
Console.WriteLine(c.EMaggiore(5));
```

Il vincolo `where T : IComparable<T>` garantisce che il compilatore accetti solo tipi che implementano il confronto, evitando errori a runtime.

### Generici e interfacce

La maggior parte delle interfacce delle collezioni in C# è generica: `IEnumerable<T>`, `IList<T>`, `ICollection<T>` e così via.
Questo significa che una lista di interi è un `IList<int>`, una lista di stringhe è un `IList<string>`, e il compilatore verifica che i tipi siano usati in modo coerente.

```csharp
IList<int> numeri = new List<int>();
numeri.Add(1);
numeri.Add(2);

IList<string> nomi = new List<string>();
nomi.Add("Alice");
// nomi.Add(1); -- errore di compilazione: il tipo non corrisponde
```

Capire i generici è fondamentale per leggere e usare LINQ, che si basa interamente su `IEnumerable<T>` e su operatori generici come `Where<T>`, `Select<T>` e così via.

## Come pensare un problema OOP

Quando devi progettare una soluzione orientata agli oggetti, conviene seguire un ordine preciso:

1. individua le entità del problema
2. per ogni entità scrivi i dati importanti
3. per ogni entità scrivi le azioni davvero utili
4. separa ciò che deve restare interno da ciò che deve essere visibile
5. crea oggetti coerenti e usa nomi espliciti

Se il codice è ben progettato, dovresti riuscire a leggerlo come una descrizione del problema.

## Esercizi guidati

Gli esercizi seguenti introducono i concetti di OOP in modo progressivo.
Per ogni traccia trovi una breve guida operativa e poi una soluzione finale.

### 1. classe semplice: rappresentare un libro

**Traccia**

Definisci una classe `Libro` con titolo e autore.

**Come ragionare**

1. Identifica i dati: titolo e autore.
2. Scrivi un costruttore che li inizializza.
3. Aggiungi proprietà in sola lettura.

**Soluzione finale**

```csharp
class Libro
{
	public string Titolo { get; }
	public string Autore { get; }

	public Libro(string titolo, string autore)
	{
		Titolo = titolo;
		Autore = autore;
	}
}
```

### 2. oggetto: usare la classe Libro

**Traccia**

Crea un oggetto `Libro` e stampa i suoi dati.

**Come ragionare**

1. Istanzia la classe con valori concreti.
2. Leggi le proprietà.
3. Stampa il risultato.

**Soluzione finale**

```csharp
Libro libro = new Libro("Il nome della rosa", "Umberto Eco");

Console.WriteLine($"{libro.Titolo} - {libro.Autore}");
```

### 3. incapsulamento: proteggere un saldo

**Traccia**

Crea una classe `Portafoglio` che consenta solo di aggiungere denaro.

**Come ragionare**

1. Rendi privato il saldo.
2. Espandi solo i metodi necessari.
3. Impedisci importi negativi.

**Soluzione finale**

```csharp
class Portafoglio
{
	private decimal _saldo;

	public Portafoglio(decimal saldoIniziale)
	{
		_saldo = saldoIniziale;
	}

	public decimal Saldo => _saldo;

	public void Aggiungi(decimal importo)
	{
		if (importo > 0)
		{
			_saldo += importo;
		}
	}
}
```

### 4. costruttore: inizializzare un punto

**Traccia**

Definisci una classe `Punto` con coordinate X e Y.

**Come ragionare**

1. Le coordinate devono essere presenti fin dalla creazione.
2. Usa un costruttore per inizializzarle.
3. Espone solo lettura se i dati non devono cambiare.

**Soluzione finale**

```csharp
class Punto
{
	public int X { get; }
	public int Y { get; }

	public Punto(int x, int y)
	{
		X = x;
		Y = y;
	}
}
```

### 5. metodo: calcolare un'area

**Traccia**

Definisci una classe `Cerchio` con un metodo che calcola l'area.

**Come ragionare**

1. Conserva il raggio come dato dell'oggetto.
2. Scrivi un metodo che restituisce un numero.
3. Non stampare dentro il metodo: restituisci il risultato.

**Soluzione finale**

```csharp
class Cerchio
{
	public double Raggio { get; }

	public Cerchio(double raggio)
	{
		Raggio = raggio;
	}

	public double CalcolaArea()
	{
		return Math.PI * Raggio * Raggio;
	}
}
```

### 6. statico: funzione di supporto

**Traccia**

Scrivi un metodo statico che verifica se un numero è positivo.

**Come ragionare**

1. Il metodo non deve dipendere da uno stato interno.
2. Rendilo statico.
3. Restituisci un valore booleano.

**Soluzione finale**

```csharp
class Verifiche
{
	public static bool EPositivo(int numero)
	{
		return numero > 0;
	}
}
```

### 7. ereditarietà: distinguere veicolo e auto

**Traccia**

Crea una classe base `Veicolo` e una classe derivata `Auto`.

**Come ragionare**

1. Individua ciò che è comune.
2. Metti il comune nella classe base.
3. Lascia nella classe derivata solo ciò che la specializza.

**Soluzione finale**

```csharp
class Veicolo
{
	public string Marca { get; }

	public Veicolo(string marca)
	{
		Marca = marca;
	}
}

class Auto : Veicolo
{
	public Auto(string marca) : base(marca)
	{
	}
}
```

### 8. polimorfismo: più forme, stessa chiamata

**Traccia**

Definisci due forme diverse e calcola la loro area con lo stesso metodo.

**Come ragionare**

1. Crea una base comune con un metodo virtuale.
2. Override nelle classi derivate.
3. Usa la stessa chiamata per tutte le forme.

**Soluzione finale**

```csharp
Forma[] forme = new Forma[]
{
	new Quadrato(4),
	new Cerchio(3)
};

for (int i = 0; i < forme.Length; i++)
{
	Console.WriteLine(forme[i].CalcolaArea());
}
```

### 9. interfaccia: oggetti stampabili

**Traccia**

Definisci un contratto comune per elementi stampabili.

**Come ragionare**

1. Individua il comportamento comune.
2. Scrivi l'interfaccia.
3. Implementala in classi diverse.

**Soluzione finale**

```csharp
interface IStampabile
{
	void Stampa();
}

class Fattura : IStampabile
{
	public void Stampa()
	{
		Console.WriteLine("Fattura stampata");
	}
}
```

### 10. esercizio complesso: gestione di una biblioteca

**Traccia**

Progetta una piccola biblioteca con queste esigenze:

1. ogni libro ha titolo, autore e stato di disponibilità
2. la biblioteca può aggiungere libri
3. la biblioteca può prestare un libro se è disponibile
4. la biblioteca può restituire un libro
5. la biblioteca può mostrare quanti libri sono disponibili

**Come ragionare in modo completo**

1. Individua le entità principali: `Libro` e `Biblioteca`.
2. Il libro deve conoscere i suoi dati e il proprio stato di disponibilità.
3. La biblioteca deve contenere una collezione di libri e gestire le operazioni.
4. La disponibilità non deve essere modificata liberamente dall'esterno.
5. Per le operazioni usa metodi brevi e chiari: aggiungi, presta, restituisci, conta.
6. Se un libro non esiste o non è disponibile, il metodo deve dirlo chiaramente.
7. Prima costruisci la struttura minima, poi aggiungi i controlli.

**Soluzione finale**

```csharp
using System.Collections.Generic;

class Libro
{
	public string Titolo { get; }
	public string Autore { get; }
	public bool Disponibile { get; private set; }

	public Libro(string titolo, string autore)
	{
		Titolo = titolo;
		Autore = autore;
		Disponibile = true;
	}

	public bool Presta()
	{
		if (!Disponibile)
		{
			return false;
		}

		Disponibile = false;
		return true;
	}

	public void Restituisci()
	{
		Disponibile = true;
	}
}

class Biblioteca
{
	private readonly List<Libro> _libri = new List<Libro>();

	public void AggiungiLibro(Libro libro)
	{
		_libri.Add(libro);
	}

	public bool PrestaLibro(string titolo)
	{
		for (int i = 0; i < _libri.Count; i++)
		{
			if (_libri[i].Titolo == titolo)
			{
				return _libri[i].Presta();
			}
		}

		return false;
	}

	public bool RestituisciLibro(string titolo)
	{
		for (int i = 0; i < _libri.Count; i++)
		{
			if (_libri[i].Titolo == titolo)
			{
				_libri[i].Restituisci();
				return true;
			}
		}

		return false;
	}

	public int ContaDisponibili()
	{
		int disponibili = 0;

		for (int i = 0; i < _libri.Count; i++)
		{
			if (_libri[i].Disponibile)
			{
				disponibili++;
			}
		}

		return disponibili;
	}
}

Biblioteca biblioteca = new Biblioteca();
biblioteca.AggiungiLibro(new Libro("Il nome della rosa", "Umberto Eco"));
biblioteca.AggiungiLibro(new Libro("1984", "George Orwell"));

biblioteca.PrestaLibro("1984");
Console.WriteLine($"Libri disponibili: {biblioteca.ContaDisponibili()}");
```

Questo esercizio mette insieme i concetti principali della dispensa: classe, oggetto, incapsulamento, metodi, collezione e ragionamento ordinato sul problema.
