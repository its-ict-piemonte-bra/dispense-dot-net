# ASP.NET — MVC e Minimal API

Questa dispensa introduce ASP.NET, la piattaforma .NET per costruire applicazioni web.
Si concentra su due modelli principali: ASP.NET MVC, pensato per applicazioni web con interfaccia utente, e ASP.NET Minimal API, pensato per servizi leggeri che espongono dati.

Prima di entrare in ASP.NET è necessario capire gli attributi, uno strumento del linguaggio C# usato ovunque nel framework.

---

## Capitolo 1 — Prerequisito: gli attributi

### Cos'è un attributo

Un attributo è un'annotazione che si aggiunge a una classe, un metodo, una proprietà o un parametro per fornire informazioni aggiuntive al compilatore, al runtime o al framework.
In pratica è metadata: non cambia la logica del codice, ma dice al sistema come deve trattarlo.

La sintassi è semplice: si scrive tra `[` e `]` sopra all'elemento che si vuole annotare.

```csharp
[Obsolete("Usa il metodo NuovoMetodo invece.")]
public void VecchioMetodo()
{
    Console.WriteLine("Metodo deprecato");
}
```

Qui `Obsolete` è un attributo che segnala al compilatore di avvisare chiunque chiami questo metodo.

**Esempio meno adatto**

```csharp
// Senza attributo: nessun avviso, il metodo sembra normale
public void VecchioMetodo()
{
    Console.WriteLine("Metodo deprecato");
}
```

Senza l'attributo, chi usa il metodo non sa che esiste un'alternativa migliore.

### Attributi comuni in C#

```csharp
// Segnala che il metodo è deprecato
[Obsolete("Usare NuovoMetodo")]
public void VecchioMetodo() { }

// Indica che la classe può essere serializzata
[Serializable]
class Configurazione { }

// Impedisce la serializzazione di un campo
[NonSerialized]
private int _cache;
```

### Attributi con parametri

Gli attributi possono ricevere valori, sia posizionali sia nominati.

```csharp
[Obsolete("Usa NuovoMetodo", error: true)]
public void MetodoRimosso() { }
```

Il secondo parametro `error: true` trasforma l'avviso in un errore di compilazione.

### Attributi personalizzati

Puoi definire i tuoi attributi creando una classe che estende `Attribute`.

```csharp
class AutoreAttribute : Attribute
{
    public string Nome { get; }
    public int Anno { get; }

    public AutoreAttribute(string nome, int anno)
    {
        Nome = nome;
        Anno = anno;
    }
}

[Autore("Luca Rossi", 2025)]
class MioServizio { }
```

Gli attributi personalizzati sono molto usati per aggiungere metadati leggibili a runtime tramite reflection.

### Perché ASP.NET li usa ovunque

ASP.NET legge gli attributi a runtime per decidere:

- quale metodo risponde a quale URL
- quale metodo HTTP è accettato (`GET`, `POST`, ecc.)
- chi è autorizzato ad accedere a una risorsa
- come validare i dati in ingresso

Senza capire gli attributi, il codice ASP.NET sembra pieno di annotazioni misteriose.

---

## Capitolo 2 — Cos'è ASP.NET

ASP.NET è il framework .NET per costruire applicazioni web, API e servizi.
Viene eseguito su un server e risponde alle richieste HTTP che arrivano dai browser o da altri client.

### Il ciclo richiesta-risposta

Ogni interazione web segue lo stesso schema:

1. il client (browser o app) invia una **richiesta HTTP**
2. il server riceve la richiesta, la elabora
3. il server invia una **risposta HTTP**

```
Client  ─── GET /prodotti ──────→  Server
Client  ←── 200 OK + JSON ──────  Server
```

### Metodi HTTP principali

| Metodo   | Uso tipico                             |
|----------|----------------------------------------|
| `GET`    | Leggere una risorsa                    |
| `POST`   | Creare una nuova risorsa               |
| `PUT`    | Sostituire una risorsa esistente       |
| `PATCH`  | Modificare parzialmente una risorsa    |
| `DELETE` | Eliminare una risorsa                  |

### Codici di stato HTTP

| Codice | Significato                        |
|--------|------------------------------------|
| `200`  | OK — richiesta riuscita            |
| `201`  | Created — risorsa creata           |
| `400`  | Bad Request — dati non validi      |
| `401`  | Unauthorized — non autenticato     |
| `403`  | Forbidden — non autorizzato        |
| `404`  | Not Found — risorsa non trovata    |
| `500`  | Internal Server Error — errore server |

### Routing

Il routing è il meccanismo che mappa un URL a un pezzo di codice specifico.

```
GET /prodotti          →  leggi tutti i prodotti
GET /prodotti/5        →  leggi il prodotto con id 5
POST /prodotti         →  crea un nuovo prodotto
DELETE /prodotti/5     →  elimina il prodotto con id 5
```

In ASP.NET il routing è configurato tramite attributi o convenzioni.

---

## Capitolo 3 — ASP.NET MVC

### Il pattern MVC

MVC è un pattern architetturale che divide un'applicazione in tre responsabilità:

- **Model** — i dati e la logica di business
- **View** — la presentazione, cioè l'HTML che viene mostrato all'utente
- **Controller** — riceve la richiesta, usa il Model, decide quale View restituire

```
Richiesta HTTP
     ↓
Controller  ──→  Model (dati)
     ↓
   View  ──→  Risposta HTML
```

Il vantaggio è la separazione chiara delle responsabilità: il controller non sa come vengono mostrati i dati, la view non sa dove vengono presi.

### Struttura di un progetto MVC

Un progetto ASP.NET MVC ha convenzionalmente questa struttura:

```
Controllers/
    ProdottiController.cs
Models/
    Prodotto.cs
Views/
    Prodotti/
        Index.cshtml
        Dettaglio.cshtml
Program.cs
```

Le cartelle non sono obbligatorie, ma la convenzione le rende attese dal framework.

### Convention over Configuration

"Convention over Configuration" (spesso abbreviato CoC) è un principio di progettazione software secondo cui il framework adotta comportamenti predefiniti basati su convenzioni, riducendo la quantità di configurazione esplicita necessaria.

In pratica: se rispetti le convenzioni, le cose funzionano senza configurazione. Se vuoi un comportamento diverso, lo devi specificare esplicitamente.

ASP.NET MVC applica questo principio in diversi punti:

- una classe chiamata `ProdottiController` viene riconosciuta automaticamente come controller per la risorsa "prodotti"
- il metodo `Index()` di `ProdottiController` cerca automaticamente la view in `Views/Prodotti/Index.cshtml`
- i parametri del metodo vengono legati automaticamente ai dati della richiesta HTTP (model binding)

**Esempio: la view trovata per convenzione**

```csharp
class ProdottiController : Controller
{
    public IActionResult Index()
    {
        return View(); // ASP.NET cerca Views/Prodotti/Index.cshtml senza che tu lo scriva
    }
}
```

Il framework costruisce il percorso della view dal nome del controller e dal nome dell'action. Non serve specificarlo.

**Esempio meno adatto**

```csharp
public IActionResult Index()
{
    return View("~/Views/Prodotti/Index.cshtml"); // percorso esplicito: ridondante
}
```

Il percorso esplicito funziona, ma è configurazione in più che non aggiunge nulla se già rispetti la convenzione.

Il vantaggio principale di CoC è la riduzione del codice di configurazione: meno cose da scrivere, meno file da mantenere, meno errori dovuti a configurazioni errate.

### Il Controller

Un controller è una classe che estende `Controller` e contiene metodi che gestiscono le richieste.
Ogni metodo pubblico è detto **action** e corrisponde a un URL.

```csharp
using Microsoft.AspNetCore.Mvc;

class ProdottiController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Dettaglio(int id)
    {
        return View(id);
    }
}
```

**Esempio meno adatto**

```csharp
class ProdottiController
{
    public void Index() { }
}
```

Un controller non esteso da `Controller` non riceve il contesto HTTP e non può usare i metodi del framework come `View()` o `Json()`.

### Attributi di routing sul controller

Gli attributi definiscono quale URL risponde a quale action e quale metodo HTTP è accettato.

```csharp
[Route("prodotti")]
class ProdottiController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        return View();
    }

    [HttpGet("{id}")]
    public IActionResult Dettaglio(int id)
    {
        return View();
    }

    [HttpPost]
    public IActionResult Crea(Prodotto prodotto)
    {
        return RedirectToAction("Index");
    }
}
```

`[HttpGet]` significa che il metodo risponde solo a richieste GET.
`[HttpGet("{id}")]` significa che l'URL include un parametro, per esempio `/prodotti/5`.

### Il Model

Il model è una classe C# che rappresenta i dati.
In MVC viene passato dalla controller alla view per essere visualizzato.

```csharp
class Prodotto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public decimal Prezzo { get; set; }
}
```

### Data Annotations: validare il Model

Gli attributi di validazione si chiamano Data Annotations e si trovano in `System.ComponentModel.DataAnnotations`.
Vengono controllati automaticamente da ASP.NET prima che il metodo del controller venga eseguito.

```csharp
using System.ComponentModel.DataAnnotations;

class Prodotto
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Il nome è obbligatorio")]
    [MaxLength(100)]
    public string Nome { get; set; }

    [Range(0.01, 10000)]
    public decimal Prezzo { get; set; }
}
```

Nel controller puoi verificare la validità con `ModelState.IsValid`:

```csharp
[HttpPost]
public IActionResult Crea(Prodotto prodotto)
{
    if (!ModelState.IsValid)
    {
        return View(prodotto);
    }

    // salva e reindirizza
    return RedirectToAction("Index");
}
```

### La View con Razor

Le view usano Razor, una sintassi che mescola HTML e C#.
Il file ha estensione `.cshtml`.

```html
@model Prodotto

<h1>@Model.Nome</h1>
<p>Prezzo: @Model.Prezzo €</p>
```

Il simbolo `@` introduce il codice C# dentro l'HTML.
`@model` in cima al file dichiara il tipo del modello passato dal controller.

**Esempio meno adatto**

```html
<h1>Prodotto</h1>
<p>Prezzo: ???</p>
```

Senza il modello tipizzato, la view non può accedere ai dati in modo sicuro.

### IActionResult e i tipi di risposta

Un controller può restituire tipi diversi in base alla situazione.

```csharp
// Restituisce una view HTML
return View(prodotto);

// Reindirizza a un'altra action
return RedirectToAction("Index");

// Restituisce 404
return NotFound();

// Restituisce dati JSON
return Json(prodotto);

// Restituisce un codice di stato personalizzato
return StatusCode(500);
```

Usare `IActionResult` come tipo di ritorno permette di restituire uno qualsiasi di questi valori senza cambiare la firma del metodo.

### Attributo Authorize

`[Authorize]` blocca l'accesso a una action o a un intero controller se l'utente non è autenticato.

```csharp
[Authorize]
[Route("admin")]
class AdminController : Controller
{
    [HttpGet]
    public IActionResult Dashboard()
    {
        return View();
    }
}
```

**Esempio meno adatto**

```csharp
public IActionResult Dashboard()
{
    if (!User.Identity.IsAuthenticated)
    {
        return RedirectToAction("Login", "Account");
    }

    return View();
}
```

Il controllo manuale funziona, ma va ripetuto in ogni metodo. `[Authorize]` centralizza la protezione in modo dichiarativo.

---

## Capitolo 4 — ASP.NET Minimal API

### Cos'è e perché è stata introdotta

Minimal API è un modello alternativo introdotto in .NET 6 per scrivere API HTTP in modo più semplice e con meno codice.
Non usa controller, non usa view e non segue il pattern MVC.

**Perché è stata introdotta**
MVC nasce per applicazioni web complesse con interfaccia HTML.
Per un servizio che restituisce solo JSON — come un'API usata da un'app mobile — MVC porta con sé strutture non necessarie.
Minimal API riduce tutto al minimo indispensabile.

### Struttura di un progetto Minimal API

Tutto il codice può stare in `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Ciao dal server");

app.Run();
```

Questo è un server HTTP funzionante. Risponde a `GET /` con la stringa `"Ciao dal server"`.

### Gestire i metodi HTTP

```csharp
app.MapGet("/prodotti", () =>
{
    return new[] { "Pane", "Latte", "Burro" };
});

app.MapPost("/prodotti", (Prodotto prodotto) =>
{
    // ricevi e salva
    return Results.Created($"/prodotti/{prodotto.Id}", prodotto);
});

app.MapDelete("/prodotti/{id}", (int id) =>
{
    return Results.NoContent();
});
```

### Parametri di rotta

I parametri nell'URL vengono estratti automaticamente:

```csharp
app.MapGet("/prodotti/{id}", (int id) =>
{
    return $"Prodotto con id {id}";
});
```

`{id}` nell'URL viene legato automaticamente al parametro `int id` della lambda.

**Esempio meno adatto**

```csharp
app.MapGet("/prodotti/{id}", (HttpContext context) =>
{
    string? idStr = context.Request.RouteValues["id"]?.ToString();
    int id = int.Parse(idStr ?? "0");
    return $"Prodotto con id {id}";
});
```

Il binding automatico è più sicuro e leggibile. La lettura manuale dal contesto è inutilmente verbosa.

### Restituire JSON

Minimal API serializza automaticamente gli oggetti C# in JSON:

```csharp
app.MapGet("/prodotti", () =>
{
    var prodotti = new[]
    {
        new { Id = 1, Nome = "Pane", Prezzo = 1.50 },
        new { Id = 2, Nome = "Latte", Prezzo = 1.20 }
    };

    return prodotti;
});
```

Il client riceve:

```json
[
  { "id": 1, "nome": "Pane", "prezzo": 1.50 },
  { "id": 2, "nome": "Latte", "prezzo": 1.20 }
]
```

### Results: restituire risposte strutturate

`Results` è una classe helper che permette di restituire risposte HTTP con codice di stato esplicito.

```csharp
app.MapGet("/prodotti/{id}", (int id) =>
{
    if (id <= 0)
    {
        return Results.BadRequest("Id non valido");
    }

    var prodotto = new { Id = id, Nome = "Pane" };
    return Results.Ok(prodotto);
});

app.MapPost("/prodotti", (Prodotto prodotto) =>
{
    return Results.Created($"/prodotti/{prodotto.Id}", prodotto);
});
```

| Metodo                   | Codice HTTP |
|--------------------------|-------------|
| `Results.Ok(...)`        | 200         |
| `Results.Created(...)`   | 201         |
| `Results.NoContent()`    | 204         |
| `Results.BadRequest(...)` | 400        |
| `Results.NotFound(...)`  | 404         |

### Dependency Injection nelle Minimal API

I servizi registrati nel container DI vengono iniettati automaticamente come parametri:

```csharp
builder.Services.AddSingleton<IProdottoRepository, ProdottoRepository>();

app.MapGet("/prodotti", (IProdottoRepository repo) =>
{
    return repo.GetAll();
});
```

### Raggruppare le rotte

Per organizzare meglio le rotte si può usare `MapGroup`:

```csharp
var prodotti = app.MapGroup("/prodotti");

prodotti.MapGet("/", () => "Lista prodotti");
prodotti.MapGet("/{id}", (int id) => $"Prodotto {id}");
prodotti.MapPost("/", (Prodotto p) => Results.Created($"/prodotti/{p.Id}", p));
```

### Attributo RequireAuthorization

Nelle Minimal API la protezione si applica in modo fluente:

```csharp
app.MapGet("/admin", () => "Area riservata")
   .RequireAuthorization();
```

---

## Capitolo 5 — MVC vs Minimal API: quando usare l'uno o l'altro

### Confronto diretto

| Caratteristica           | ASP.NET MVC                         | Minimal API                        |
|--------------------------|-------------------------------------|------------------------------------|
| Struttura                | Controller, View, Model             | Solo `Program.cs` (o file organizzati) |
| Interfaccia utente HTML  | Sì, con Razor                       | No                                 |
| Risposta tipica          | HTML o JSON                         | JSON                               |
| Routing                  | Attributi o convenzione             | `MapGet`, `MapPost`, ecc.          |
| Verbosità                | Più strutturata                     | Minima                             |
| Scenari ideali           | Web app con pagine HTML             | API REST, microservizi             |
| Introdotto in            | ASP.NET (storico)                   | .NET 6                             |

### Quando scegliere MVC

- stai costruendo un sito web con pagine HTML
- hai bisogno di form, validazione lato server e view Razor
- il progetto è grande e beneficia della struttura a cartelle

### Quando scegliere Minimal API

- stai costruendo un'API che restituisce solo JSON
- il progetto è piccolo o un microservizio
- vuoi meno codice e meno file da mantenere
- consumi l'API da un frontend separato (React, Angular, app mobile)

### Possono coesistere

In .NET è possibile avere MVC e Minimal API nello stesso progetto.
In pratica si usa MVC per le pagine web e Minimal API per gli endpoint REST.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();
app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");

// Endpoint Minimal API accanto ai controller MVC
app.MapGet("/api/stato", () => Results.Ok(new { Stato = "OK" }));

app.Run();
```

---

## Esercizi guidati

### 1. Attributo personalizzato: annotare una classe

**Traccia**

Definisci un attributo `Versione` con un numero di versione e applicalo a una classe.

**Come ragionare**

1. Crea una classe che estende `Attribute`.
2. Aggiungi una proprietà per il numero di versione.
3. Applica l'attributo.

**Soluzione finale**

```csharp
class VersioneAttribute : Attribute
{
    public string Numero { get; }

    public VersioneAttribute(string numero)
    {
        Numero = numero;
    }
}

[Versione("1.0")]
class MioServizio { }
```

### 2. Controller MVC: action GET di base

**Traccia**

Scrivi un controller MVC con una action che restituisce una view.

**Come ragionare**

1. Estendi `Controller`.
2. Usa `[HttpGet]` sulla action.
3. Restituisci `View()`.

**Soluzione finale**

```csharp
using Microsoft.AspNetCore.Mvc;

[Route("home")]
class HomeController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        return View();
    }
}
```

### 3. Controller MVC: action con parametro

**Traccia**

Scrivi una action che riceve un `id` dall'URL e restituisce JSON con quel valore.

**Come ragionare**

1. Usa `[HttpGet("{id}")]`.
2. Aggiungi `int id` come parametro del metodo.
3. Usa `Json(...)` per rispondere.

**Soluzione finale**

```csharp
[Route("prodotti")]
class ProdottiController : Controller
{
    [HttpGet("{id}")]
    public IActionResult Dettaglio(int id)
    {
        return Json(new { Id = id, Nome = "Esempio" });
    }
}
```

### 4. Model con Data Annotations

**Traccia**

Definisci un model `Ordine` con nome obbligatorio e quantità tra 1 e 100.

**Come ragionare**

1. Usa `[Required]` per il nome.
2. Usa `[Range]` per la quantità.

**Soluzione finale**

```csharp
using System.ComponentModel.DataAnnotations;

class Ordine
{
    [Required(ErrorMessage = "Il nome è obbligatorio")]
    public string Nome { get; set; }

    [Range(1, 100, ErrorMessage = "La quantità deve essere tra 1 e 100")]
    public int Quantita { get; set; }
}
```

### 5. Controller MVC: POST con validazione

**Traccia**

Scrivi una action POST che valida il model e restituisce errori o successo.

**Come ragionare**

1. Ricevi il model come parametro.
2. Controlla `ModelState.IsValid`.
3. Restituisci la view con gli errori oppure reindirizza.

**Soluzione finale**

```csharp
[HttpPost]
public IActionResult Crea(Ordine ordine)
{
    if (!ModelState.IsValid)
    {
        return View(ordine);
    }

    return RedirectToAction("Index");
}
```

### 6. Minimal API: endpoint GET che restituisce JSON

**Traccia**

Crea un endpoint Minimal API che risponde a `GET /saluto` con un oggetto JSON.

**Come ragionare**

1. Usa `app.MapGet`.
2. Restituisci un tipo anonimo.

**Soluzione finale**

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/saluto", () => new { Messaggio = "Ciao dal server" });

app.Run();
```

### 7. Minimal API: parametro di rotta

**Traccia**

Crea un endpoint che riceve un `id` dall'URL e restituisce un oggetto con quell'id.

**Come ragionare**

1. Usa `{id}` nella rotta.
2. Aggiungi `int id` come parametro della lambda.

**Soluzione finale**

```csharp
app.MapGet("/prodotti/{id}", (int id) =>
{
    return Results.Ok(new { Id = id, Nome = "Prodotto esempio" });
});
```

### 8. Minimal API: POST con body JSON

**Traccia**

Crea un endpoint POST che riceve un oggetto dal body e risponde con `201 Created`.

**Come ragionare**

1. Il parametro della lambda riceve il body deserializzato automaticamente.
2. Usa `Results.Created` per la risposta.

**Soluzione finale**

```csharp
app.MapPost("/prodotti", (Prodotto prodotto) =>
{
    return Results.Created($"/prodotti/{prodotto.Id}", prodotto);
});
```

### 9. Minimal API: validazione e risposta 400

**Traccia**

Crea un endpoint GET che restituisce `400 Bad Request` se il parametro non è valido.

**Come ragionare**

1. Controlla il valore del parametro.
2. Usa `Results.BadRequest` per il caso non valido.
3. Usa `Results.Ok` per il caso valido.

**Soluzione finale**

```csharp
app.MapGet("/prodotti/{id}", (int id) =>
{
    if (id <= 0)
    {
        return Results.BadRequest("L'id deve essere maggiore di zero.");
    }

    return Results.Ok(new { Id = id, Nome = "Prodotto" });
});
```

### 10. Esercizio complesso: API REST per una lista di prodotti

**Traccia**

Progetta una piccola API Minimal con queste rotte:

1. `GET /prodotti` — restituisce tutti i prodotti
2. `GET /prodotti/{id}` — restituisce un prodotto per id o 404
3. `POST /prodotti` — aggiunge un prodotto e risponde 201
4. `DELETE /prodotti/{id}` — rimuove un prodotto o risponde 404

**Come ragionare passo per passo**

1. Definisci un record o una classe `Prodotto` semplice.
2. Usa una lista in memoria come fonte dati temporanea.
3. Implementa ogni rotta separatamente.
4. Per `GET {id}` e `DELETE {id}` usa `FirstOrDefault` per cercare il prodotto.
5. Restituisci sempre il codice HTTP corretto con `Results`.
6. Raggruppa le rotte con `MapGroup` per tenere il codice ordinato.

**Soluzione finale**

```csharp
using System.Collections.Generic;
using System.Linq;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var prodotti = new List<Prodotto>
{
    new Prodotto(1, "Pane", 1.50m),
    new Prodotto(2, "Latte", 1.20m)
};

var gruppo = app.MapGroup("/prodotti");

gruppo.MapGet("/", () =>
{
    return Results.Ok(prodotti);
});

gruppo.MapGet("/{id}", (int id) =>
{
    Prodotto? trovato = prodotti.FirstOrDefault(p => p.Id == id);

    if (trovato is null)
    {
        return Results.NotFound($"Prodotto con id {id} non trovato.");
    }

    return Results.Ok(trovato);
});

gruppo.MapPost("/", (Prodotto prodotto) =>
{
    prodotti.Add(prodotto);
    return Results.Created($"/prodotti/{prodotto.Id}", prodotto);
});

gruppo.MapDelete("/{id}", (int id) =>
{
    Prodotto? trovato = prodotti.FirstOrDefault(p => p.Id == id);

    if (trovato is null)
    {
        return Results.NotFound($"Prodotto con id {id} non trovato.");
    }

    prodotti.Remove(trovato);
    return Results.NoContent();
});

app.Run();

record Prodotto(int Id, string Nome, decimal Prezzo);
```

Questo esercizio mette insieme attributi, routing, metodi HTTP, codici di stato, LINQ e i concetti di Minimal API in un servizio minimo ma funzionante.
