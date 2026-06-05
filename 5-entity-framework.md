# Entity Framework Core

Questa dispensa introduce Entity Framework Core, il sistema di accesso ai dati raccomandato nell'ecosistema .NET.
L'obiettivo è capire come funziona, perché è utile, come si combina con LINQ e qual è la differenza concettuale rispetto a lavorare con semplici liste in memoria.

---

## Capitolo 1 — Cos'è Entity Framework Core

### Il problema senza un ORM

Senza strumenti dedicati, per leggere dati da un database relazionale si scrive SQL direttamente nel codice C#:

```csharp
using var connection = new SqlConnection(connectionString);
connection.Open();

var command = new SqlCommand("SELECT Id, Nome, Prezzo FROM Prodotti WHERE Prezzo > 10", connection);
var reader = command.ExecuteReader();

while (reader.Read())
{
    int id = reader.GetInt32(0);
    string nome = reader.GetString(1);
    decimal prezzo = reader.GetDecimal(2);

    Console.WriteLine($"{id} - {nome} - {prezzo}");
}
```

Questo approccio funziona, ma presenta diversi problemi:

- il codice SQL è una stringa, quindi nessun controllo del compilatore
- i tipi devono essere gestiti manualmente con `GetInt32`, `GetString` ecc.
- aggiungere una colonna richiede di aggiornare sia il database sia ogni query manuale
- il codice è difficile da testare in isolamento

### Cos'è un ORM

Un ORM, acronimo di Object-Relational Mapper, è uno strato software che traduce automaticamente le classi C# in tabelle del database e viceversa.

Con un ORM scrivi in C# e il framework genera il SQL per te.

### Entity Framework Core

Entity Framework Core, spesso abbreviato in EF Core, è l'ORM ufficiale di Microsoft per .NET.
Permette di:

- mappare classi C# su tabelle del database
- scrivere query in C# con LINQ invece di SQL
- gestire le modifiche alla struttura del database tramite le **migration**
- lavorare con diversi database cambiando solo la configurazione

```csharp
// Con EF Core la stessa query diventa:
IList<Prodotto> cari = dbContext.Prodotti
    .Where(p => p.Prezzo > 10)
    .ToList();
```

Il compilatore verifica i tipi, la query è espressa in C# e la classe `Prodotto` è un normale oggetto C#.

### Code First e Database First

EF Core supporta due approcci principali:

- **Code First**: si definiscono prima le classi C# e da esse EF genera il database tramite migration
- **Database First**: si parte da un database esistente e EF genera le classi corrispondenti

Il modo più comune nei nuovi progetti è **Code First**, perché lascia il controllo della struttura dati nel codice sorgente.

---

## Capitolo 2 — DbContext e DbSet

### DbContext

Il `DbContext` è la classe centrale di EF Core.
Rappresenta una sessione con il database: gestisce la connessione, traccia le modifiche agli oggetti e le invia al database quando chiami `SaveChanges`.

Si crea una classe che estende `DbContext`:

```csharp
using Microsoft.EntityFrameworkCore;

class NegozioContext : DbContext
{
    public DbSet<Prodotto> Prodotti { get; set; }
    public DbSet<Categoria> Categorie { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlite("Data Source=negozio.db");
    }
}
```

### DbSet\<T\>

`DbSet<T>` rappresenta una tabella del database come se fosse una collezione C#.
Implementa `IQueryable<T>`, che estende `IEnumerable<T>`.

```csharp
using var db = new NegozioContext();

// Legge tutti i prodotti
IList<Prodotto> tutti = db.Prodotti.ToList();

// Filtra
IList<Prodotto> economici = db.Prodotti
    .Where(p => p.Prezzo < 5)
    .ToList();
```

### La differenza cruciale: List\<T\> vs DbSet\<T\>

Questa è la distinzione più importante da capire prima di usare EF con LINQ.

**Con una List\<T\> in memoria:**

```csharp
List<Prodotto> prodotti = new List<Prodotto>
{
    new Prodotto { Id = 1, Nome = "Pane", Prezzo = 1.5m },
    new Prodotto { Id = 2, Nome = "Latte", Prezzo = 1.2m }
};

// Where viene eseguito in memoria: tutti i prodotti sono già stati caricati
IEnumerable<Prodotto> filtrati = prodotti.Where(p => p.Prezzo > 1);
```

La lista è già in memoria. `Where` scorre gli oggetti C# uno per uno.

**Con un DbSet\<T\>:**

```csharp
using var db = new NegozioContext();

// Where NON viene eseguito subito: costruisce una query SQL
IQueryable<Prodotto> query = db.Prodotti.Where(p => p.Prezzo > 1);

// Il SQL viene generato ed eseguito solo qui
IList<Prodotto> filtrati = query.ToList();
```

Il SQL generato da EF sarà qualcosa come:

```sql
SELECT * FROM Prodotti WHERE Prezzo > 1
```

Solo i record che soddisfano la condizione vengono trasferiti dal database al programma.

### Tabella di confronto

| Aspetto                     | `List<T>`                          | `DbSet<T>`                         |
|-----------------------------|------------------------------------|------------------------------------|
| Dove vivono i dati          | In memoria                         | Nel database                       |
| Tipo base                   | `IEnumerable<T>`                   | `IQueryable<T>`                    |
| Quando viene eseguito Where | Subito, in memoria                 | Quando si materializza la query    |
| Traduzione automatica       | Nessuna                            | LINQ → SQL                         |
| Persistenza                 | Dura finché il programma è in esecuzione | Persistente                  |

### IEnumerable vs IQueryable

Chiamare `AsEnumerable()` nel mezzo di una pipeline è un errore comune: interrompe la traduzione SQL e forza l'esecuzione in memoria da quel punto in poi.

```csharp
// IQueryable: il filtro avviene nel database
IQueryable<Prodotto> queryDb = db.Prodotti.Where(p => p.Prezzo > 10);

// IEnumerable: scarica tutto in memoria, poi filtra in C#
IEnumerable<Prodotto> inMemoria = db.Prodotti.AsEnumerable().Where(p => p.Prezzo > 10);
```

**Regola pratica**: non chiamare mai `AsEnumerable()` o `ToList()` prima di aver applicato tutti i filtri.

```csharp
// Corretto: prima filtra, poi materializza
IList<Prodotto> risultato = db.Prodotti
    .Where(p => p.Prezzo > 10)
    .OrderBy(p => p.Nome)
    .ToList();

// Sbagliato: carica tutto in memoria, poi filtra
IList<Prodotto> sbagliato = db.Prodotti
    .ToList()
    .Where(p => p.Prezzo > 10)
    .ToList();
```

---

## Capitolo 3 — Modellare i dati

### Le entity classes

Una entity class è una classe C# che corrisponde a una tabella del database.
Per convenzione EF usa il nome della classe come nome della tabella e il nome delle proprietà come nomi delle colonne.

```csharp
class Prodotto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public decimal Prezzo { get; set; }
}
```

EF riconosce automaticamente `Id` come chiave primaria.

### Data Annotations per EF

Gli stessi attributi di validazione usati in ASP.NET hanno un significato anche per EF nel momento in cui genera il database.

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

class Prodotto
{
    [Key]
    public int Id { get; set; }

    [Required]
    [MaxLength(200)]
    public string Nome { get; set; }

    [Column(TypeName = "decimal(10,2)")]
    public decimal Prezzo { get; set; }
}
```

| Attributo              | Effetto nel database             |
|------------------------|----------------------------------|
| `[Key]`                | Chiave primaria                  |
| `[Required]`           | Colonna NOT NULL                 |
| `[MaxLength(n)]`       | VARCHAR(n)                       |
| `[Column(TypeName)]`   | Tipo SQL specifico               |
| `[NotMapped]`          | Proprietà ignorata da EF         |

### Relazioni tra entità

#### Uno-a-molti

Un `Ordine` ha molti `RigaOrdine`.

```csharp
class Ordine
{
    public int Id { get; set; }
    public string Cliente { get; set; }

    // Navigation property: EF capisce che Ordine ha molte RigheOrdine
    public List<RigaOrdine> Righe { get; set; } = new List<RigaOrdine>();
}

class RigaOrdine
{
    public int Id { get; set; }
    public string Prodotto { get; set; }
    public int Quantita { get; set; }

    // Chiave esterna
    public int OrdineId { get; set; }

    // Navigation property inversa
    public Ordine Ordine { get; set; }
}
```

EF riconosce `OrdineId` come chiave esterna grazie alla convenzione sul nome.

#### Molti-a-molti

```csharp
class Studente
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public List<Corso> Corsi { get; set; } = new List<Corso>();
}

class Corso
{
    public int Id { get; set; }
    public string Titolo { get; set; }
    public List<Studente> Studenti { get; set; } = new List<Studente>();
}
```

EF Core 5+ genera automaticamente la tabella di join senza doverla dichiarare.

---

## Capitolo 4 — Migration

### Cos'è una migration

Una migration è un file C# generato automaticamente da EF che descrive come trasformare il database da uno stato a un altro.
Ogni volta che modifichi una entity class, generi una nuova migration che aggiorna lo schema del database.

### Perché esistono

Senza migration l'unica opzione è cancellare e ricreare il database ogni volta che cambia la struttura.
Le migration registrano la storia delle modifiche e permettono di applicarle in modo incrementale, anche su database di produzione.

### Comandi principali

Questi comandi si eseguono dal terminale nella cartella del progetto con i tool EF installati.

```bash
# Genera una nuova migration con il nome dato
dotnet ef migrations add AggiuntaColonnaPrezzo

# Applica tutte le migration pendenti al database
dotnet ef database update

# Torna alla migration precedente
dotnet ef database update NomeMigrationPrecedente
```

### Cosa contiene un file di migration

```csharp
public partial class AggiuntaColonnaPrezzo : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<decimal>(
            name: "Prezzo",
            table: "Prodotti",
            nullable: false,
            defaultValue: 0m);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(name: "Prezzo", table: "Prodotti");
    }
}
```

`Up` applica la modifica. `Down` la annulla.
EF genera questo codice automaticamente: in genere non devi scriverlo a mano.

---

## Capitolo 5 — LINQ con EF Core

### Come LINQ viene tradotto in SQL

Quando scrivi una query LINQ su un `DbSet<T>`, EF costruisce internamente un albero di espressioni che poi traduce in SQL prima di inviarlo al database.

```csharp
// Notazione metodo
IList<Prodotto> risultato = db.Prodotti
    .Where(p => p.Prezzo > 10 && p.Nome.StartsWith("L"))
    .OrderBy(p => p.Nome)
    .Take(5)
    .ToList();
```

SQL generato da EF:

```sql
SELECT TOP 5 Id, Nome, Prezzo
FROM Prodotti
WHERE Prezzo > 10 AND Nome LIKE 'L%'
ORDER BY Nome
```

Il filtro, l'ordinamento e il limite vengono eseguiti direttamente nel database.

### Include: caricare le relazioni

Per default EF non carica automaticamente le navigation properties. Questo si chiama **lazy loading disabilitato per default**.
Per caricare le relazioni insieme all'entità principale si usa `Include`.

```csharp
// Senza Include: Righe sarà null o vuota
Ordine ordine = db.Ordini.First(o => o.Id == 1);

// Con Include: Righe viene caricata con una JOIN
Ordine ordine = db.Ordini
    .Include(o => o.Righe)
    .First(o => o.Id == 1);
```

SQL generato con `Include`:

```sql
SELECT o.Id, o.Cliente, r.Id, r.Prodotto, r.Quantita
FROM Ordini o
LEFT JOIN RigheOrdine r ON r.OrdineId = o.Id
WHERE o.Id = 1
```

**Senza Include (comportamento meno adatto)**

```csharp
// Carichi ogni ordine e poi ogni riga con una query separata: problema N+1
foreach (Ordine o in db.Ordini.ToList())
{
    foreach (RigaOrdine r in db.RigheOrdine.Where(r => r.OrdineId == o.Id).ToList())
    {
        Console.WriteLine(r.Prodotto);
    }
}
```

Se hai 100 ordini, esegui 1 query per gli ordini più 100 query per le righe: 101 query totali.
Con `Include` è una sola query con JOIN.

### ThenInclude: relazioni annidate

```csharp
db.Ordini
    .Include(o => o.Righe)
        .ThenInclude(r => r.Prodotto)
    .ToList();
```

### AsNoTracking: lettura senza tracciamento

Per default EF tiene traccia degli oggetti letti per rilevarne le modifiche.
Se stai solo leggendo dati senza modificarli, `AsNoTracking` migliora le prestazioni.

```csharp
IList<Prodotto> prodotti = db.Prodotti.AsNoTracking().ToList();
```

### Select proiettato: non caricare tutto

Puoi usare `Select` per proiettare solo i campi che ti servono.

```csharp
// Notazione query
var query =
    from p in db.Prodotti
    select new { p.Nome, p.Prezzo };

var proiettati = query.ToList();
```

Genera:

```sql
SELECT Nome, Prezzo FROM Prodotti
```

---

## Capitolo 6 — Operazioni CRUD

### Create

```csharp
using var db = new NegozioContext();

Prodotto nuovo = new Prodotto
{
    Nome = "Pasta",
    Prezzo = 1.30m
};

db.Prodotti.Add(nuovo);
db.SaveChanges();

Console.WriteLine($"Id assegnato: {nuovo.Id}");
```

`SaveChanges` invia il SQL `INSERT` al database e aggiorna la proprietà `Id` con il valore generato.

### Read

```csharp
// Tutti
IList<Prodotto> tutti = db.Prodotti.ToList();

// Per chiave primaria (metodo ottimizzato)
Prodotto? trovato = db.Prodotti.Find(3);

// Con condizione
Prodotto? primo = db.Prodotti.FirstOrDefault(p => p.Nome == "Pasta");
```

`Find` usa prima la cache interna del contesto, poi il database: è più efficiente di `FirstOrDefault` per la chiave primaria.

### Update

```csharp
Prodotto? prodotto = db.Prodotti.Find(3);

if (prodotto is not null)
{
    prodotto.Prezzo = 1.50m;
    db.SaveChanges();
}
```

EF rileva che `Prezzo` è cambiato e genera solo:

```sql
UPDATE Prodotti SET Prezzo = 1.50 WHERE Id = 3
```

Non serve chiamare metodi speciali: basta modificare l'oggetto e chiamare `SaveChanges`.

### Delete

```csharp
Prodotto? prodotto = db.Prodotti.Find(3);

if (prodotto is not null)
{
    db.Prodotti.Remove(prodotto);
    db.SaveChanges();
}
```

### Async: non bloccare il thread

In applicazioni ASP.NET è sempre preferibile usare le versioni asincrone per non bloccare il thread del server.

```csharp
// Lettura async
IList<Prodotto> prodotti = await db.Prodotti.ToListAsync();

// Salvataggio async
db.Prodotti.Add(nuovo);
await db.SaveChangesAsync();
```

---

## Capitolo 7 — EF Core con ASP.NET

### Registrare il DbContext nel container DI

In un progetto ASP.NET il DbContext si registra in `Program.cs` e viene iniettato automaticamente nei controller o nelle Minimal API.

```csharp
builder.Services.AddDbContext<NegozioContext>(options =>
    options.UseSqlite("Data Source=negozio.db"));
```

### Uso in una Minimal API

```csharp
app.MapGet("/prodotti", async (NegozioContext db) =>
{
    return await db.Prodotti.AsNoTracking().ToListAsync();
});

app.MapPost("/prodotti", async (Prodotto prodotto, NegozioContext db) =>
{
    db.Prodotti.Add(prodotto);
    await db.SaveChangesAsync();
    return Results.Created($"/prodotti/{prodotto.Id}", prodotto);
});
```

### Uso in un Controller MVC

```csharp
[Route("prodotti")]
class ProdottiController : Controller
{
    private readonly NegozioContext _db;

    public ProdottiController(NegozioContext db)
    {
        _db = db;
    }

    [HttpGet]
    public async Task<IActionResult> Index()
    {
        IList<Prodotto> prodotti = await _db.Prodotti.AsNoTracking().ToListAsync();
        return View(prodotti);
    }
}
```

---

## Esercizi guidati

### 1. Definire una entity class

**Traccia**

Definisci una classe `Libro` con id, titolo, autore e prezzo, pronta per essere usata con EF.

**Come ragionare**

1. `Id` è la chiave primaria per convenzione.
2. Aggiungi `[Required]` e `[MaxLength]` dove utile.
3. Usa `decimal` per il prezzo con il tipo SQL esplicito.

**Soluzione finale**

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

class Libro
{
    public int Id { get; set; }

    [Required]
    [MaxLength(300)]
    public string Titolo { get; set; }

    [Required]
    [MaxLength(150)]
    public string Autore { get; set; }

    [Column(TypeName = "decimal(10,2)")]
    public decimal Prezzo { get; set; }
}
```

### 2. Definire un DbContext

**Traccia**

Crea un `DbContext` per una libreria che espone libri e categorie.

**Come ragionare**

1. Estendi `DbContext`.
2. Aggiungi un `DbSet<T>` per ogni entità.
3. Configura la connessione in `OnConfiguring`.

**Soluzione finale**

```csharp
using Microsoft.EntityFrameworkCore;

class LibreriaContext : DbContext
{
    public DbSet<Libro> Libri { get; set; }
    public DbSet<Categoria> Categorie { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlite("Data Source=libreria.db");
    }
}
```

### 3. LINQ su DbSet: filtrare e ordinare

**Traccia**

Leggi tutti i libri con prezzo inferiore a 20 euro, ordinati per titolo.

**Come ragionare**

1. Usa `Where` prima di `OrderBy`.
2. Chiama `ToList()` solo alla fine per materializzare.
3. Usa `AsNoTracking` perché è solo lettura.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

IList<Libro> economici = db.Libri
    .AsNoTracking()
    .Where(l => l.Prezzo < 20)
    .OrderBy(l => l.Titolo)
    .ToList();
```

### 4. Differenza List vs DbSet

**Traccia**

Mostra la differenza tra filtrare una `List<Libro>` e filtrare un `DbSet<Libro>`.

**Come ragionare**

1. Con la lista il filtro avviene in memoria dopo che tutti i dati sono già stati caricati.
2. Con il DbSet il filtro viene tradotto in SQL e inviato al database.

**Soluzione finale**

```csharp
// Lista in memoria: carica tutto, poi filtra in C#
List<Libro> listaLocale = new List<Libro>
{
    new Libro { Id = 1, Titolo = "Clean Code", Prezzo = 35m },
    new Libro { Id = 2, Titolo = "Il Signore degli Anelli", Prezzo = 15m }
};

IEnumerable<Libro> filtroInMemoria = listaLocale.Where(l => l.Prezzo < 20);

// DbSet: genera SQL WHERE Prezzo < 20, porta in memoria solo i risultati
using var db = new LibreriaContext();
IList<Libro> filtroSuDb = db.Libri.Where(l => l.Prezzo < 20).ToList();
```

### 5. Creare un record

**Traccia**

Aggiungi un nuovo libro al database.

**Come ragionare**

1. Crea l'oggetto senza impostare `Id`.
2. Usa `Add` e `SaveChanges`.
3. Dopo `SaveChanges` l'`Id` è valorizzato dal database.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

Libro nuovo = new Libro
{
    Titolo = "Il nome della rosa",
    Autore = "Umberto Eco",
    Prezzo = 14.90m
};

db.Libri.Add(nuovo);
db.SaveChanges();

Console.WriteLine($"Id assegnato: {nuovo.Id}");
```

### 6. Aggiornare un record

**Traccia**

Trova un libro per id e aggiorna il prezzo.

**Come ragionare**

1. Usa `Find` per recuperare il libro.
2. Modifica la proprietà.
3. Chiama `SaveChanges`: EF genera solo l'UPDATE necessario.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

Libro? libro = db.Libri.Find(1);

if (libro is not null)
{
    libro.Prezzo = 12.50m;
    db.SaveChanges();
}
```

### 7. Eliminare un record

**Traccia**

Rimuovi un libro dal database dato il suo id.

**Come ragionare**

1. Recupera il libro con `Find`.
2. Se esiste, usa `Remove` e `SaveChanges`.
3. Se non esiste, non fare nulla.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

Libro? libro = db.Libri.Find(1);

if (libro is not null)
{
    db.Libri.Remove(libro);
    db.SaveChanges();
}
```

### 8. Include: caricare una relazione

**Traccia**

Leggi tutti gli ordini includendo le righe di ogni ordine.

**Come ragionare**

1. Usa `Include` per caricare la navigation property.
2. Senza `Include` la proprietà sarebbe null o vuota.
3. Con `Include` EF genera una JOIN.

**Soluzione finale**

```csharp
using var db = new NegozioContext();

IList<Ordine> ordini = db.Ordini
    .Include(o => o.Righe)
    .AsNoTracking()
    .ToList();

foreach (Ordine ordine in ordini)
{
    Console.WriteLine($"Ordine {ordine.Id} - {ordine.Cliente}");

    foreach (RigaOrdine riga in ordine.Righe)
    {
        Console.WriteLine($"  {riga.Prodotto} x{riga.Quantita}");
    }
}
```

### 9. LINQ proiettato: Select su DbSet

**Traccia**

Leggi solo titolo e prezzo di tutti i libri senza caricare l'intera entità.

**Come ragionare**

1. Usa `Select` prima di `ToList`.
2. Il SQL generato includerà solo le colonne richieste.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

var proiettati = db.Libri
    .AsNoTracking()
    .Select(l => new { l.Titolo, l.Prezzo })
    .ToList();

foreach (var libro in proiettati)
{
    Console.WriteLine($"{libro.Titolo}: {libro.Prezzo} €");
}
```

### 10. Esercizio complesso: report aggregato con LINQ e EF

**Traccia**

Hai un `DbContext` con libri e categorie.
Devi:

1. trovare il prezzo medio dei libri per categoria
2. stampare solo le categorie con almeno 2 libri
3. ordinare per prezzo medio decrescente
4. stampare nome categoria, numero libri e prezzo medio

**Come ragionare passo per passo**

1. Usa `Include` per caricare i libri dentro ogni categoria.
2. Applica `Where` per filtrare le categorie con almeno 2 libri.
3. Usa `Select` per proiettare un tipo anonimo con nome, conteggio e media.
4. Applica `OrderByDescending` sulla media.
5. Materializza con `ToList` solo alla fine.
6. Usa `AsNoTracking` perché è solo lettura.

**Soluzione finale**

```csharp
using var db = new LibreriaContext();

var report = db.Categorie
    .AsNoTracking()
    .Include(c => c.Libri)
    .Where(c => c.Libri.Count >= 2)
    .Select(c => new
    {
        c.Nome,
        NumeroLibri = c.Libri.Count,
        PrezzoMedio = c.Libri.Average(l => l.Prezzo)
    })
    .OrderByDescending(r => r.PrezzoMedio)
    .ToList();

foreach (var voce in report)
{
    Console.WriteLine($"{voce.Nome}: {voce.NumeroLibri} libri, media {voce.PrezzoMedio:F2} €");
}
```

Questo esercizio mette insieme il modello relazionale, `Include`, `Where`, `Select`, `Average`, `OrderByDescending` e la comprensione di `IQueryable`: tutte le query vengono tradotte in un unico SQL ottimizzato, senza caricare dati inutili in memoria.
