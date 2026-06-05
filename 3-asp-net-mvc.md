# ASP.NET MVC

Questa dispensa introduce ASP.NET MVC, il modello di ASP.NET pensato per applicazioni web con interfaccia utente server-side.
Il focus e sulla struttura Model-View-Controller, sulle convenzioni del framework e sul flusso completo richiesta -> controller -> view.

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

## Capitolo 4 — Setup di un progetto MVC

Il setup minimo di MVC in `Program.cs` e questo:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

Con `AddControllersWithViews()` abiliti sia i controller sia il rendering Razor.

---

## Capitolo 5 — Dependency Injection in MVC

La registrazione dei servizi avviene in `Program.cs`:

```csharp
builder.Services.AddSingleton<IProdottoRepository, ProdottoRepository>();
```

Nel controller usi la constructor injection:

```csharp
class ProdottiController : Controller
{
    private readonly IProdottoRepository _repo;

    public ProdottiController(IProdottoRepository repo)
    {
        _repo = repo;
    }

    public IActionResult Index()
    {
        return View(_repo.GetAll());
    }
}
```

---

## Capitolo 6 — Organizzare un'app MVC reale

Per mantenere il progetto leggibile:

- tieni i controller sottili e sposta la logica di business in servizi
- usa view model dedicati per la UI invece di passare sempre le entity
- valida sempre i dati in input con Data Annotations e `ModelState`
- usa `RedirectToAction` dopo una POST valida (pattern Post/Redirect/Get)

---

## Esercizi guidati (solo MVC)

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

### 6. MVC: action che restituisce JSON

**Traccia**

Crea una action MVC che risponde a `GET /saluto` con un oggetto JSON.

**Come ragionare**

1. Crea un controller dedicato.
2. Definisci una action GET.
3. Restituisci `Json(...)`.

**Soluzione finale**

```csharp
[Route("saluto")]
class SalutoController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        return Json(new { Messaggio = "Ciao dal server" });
    }
}
```

### 7. MVC: parametro di rotta

**Traccia**

Crea una action che riceve un `id` dall'URL e restituisce un oggetto con quell'id.

**Come ragionare**

1. Usa `{id}` nella rotta.
2. Aggiungi `int id` come parametro del metodo.
3. Restituisci `Json(...)`.

**Soluzione finale**

```csharp
[Route("prodotti")]
class ProdottiController : Controller
{
    [HttpGet("{id}")]
    public IActionResult Dettaglio(int id)
    {
        return Json(new { Id = id, Nome = "Prodotto esempio" });
    }
}
```

### 8. MVC: POST con body e redirect

**Traccia**

Crea una action POST che riceve un model dal form, valida e poi reindirizza.

**Come ragionare**

1. Ricevi il model come parametro.
2. Controlla `ModelState.IsValid`.
3. Reindirizza a `Index` se valido.

**Soluzione finale**

```csharp
[HttpPost]
public IActionResult Crea(Prodotto prodotto)
{
    if (!ModelState.IsValid)
    {
        return View(prodotto);
    }

    return RedirectToAction("Index");
}
```

### 9. MVC: proteggere area admin con Authorize

**Traccia**

Crea un controller admin accessibile solo da utenti autenticati.

**Come ragionare**

1. Applica `[Authorize]` al controller.
2. Definisci una action `Dashboard`.

**Soluzione finale**

```csharp
using Microsoft.AspNetCore.Authorization;

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

### 10. Esercizio complesso: mini CRUD MVC prodotti

**Traccia**

Progetta un piccolo flusso MVC prodotti con queste action:

1. `GET /prodotti` — mostra lista
2. `GET /prodotti/crea` — mostra form
3. `POST /prodotti/crea` — valida e salva
4. `GET /prodotti/dettaglio/{id}` — mostra dettaglio o 404

**Come ragionare passo per passo**

1. Definisci il model `Prodotto` con Data Annotations.
2. Prepara una lista in memoria come repository temporaneo.
3. Implementa `Index`, `Crea` GET/POST e `Dettaglio`.
4. In POST usa `ModelState.IsValid` e pattern Post/Redirect/Get.
5. Per il dettaglio inesistente restituisci `NotFound()`.

**Soluzione finale**

```csharp
using Microsoft.AspNetCore.Mvc;

[Route("prodotti")]
class ProdottiController : Controller
{
    private static readonly List<Prodotto> _prodotti =
    [
        new Prodotto { Id = 1, Nome = "Pane", Prezzo = 1.50m },
        new Prodotto { Id = 2, Nome = "Latte", Prezzo = 1.20m }
    ];

    [HttpGet]
    public IActionResult Index()
    {
        return View(_prodotti);
    }

    [HttpGet("crea")]
    public IActionResult Crea()
    {
        return View();
    }

    [HttpPost("crea")]
    public IActionResult Crea(Prodotto prodotto)
    {
        if (!ModelState.IsValid)
        {
            return View(prodotto);
        }

        prodotto.Id = _prodotti.Max(p => p.Id) + 1;
        _prodotti.Add(prodotto);
        return RedirectToAction("Index");
    }

    [HttpGet("dettaglio/{id}")]
    public IActionResult Dettaglio(int id)
    {
        var trovato = _prodotti.FirstOrDefault(p => p.Id == id);

        if (trovato is null)
        {
            return NotFound();
        }

        return View(trovato);
    }
}
```

Questo esercizio mette insieme routing, binding, validazione, Razor views e gestione del flusso MVC in una mini applicazione completa.
