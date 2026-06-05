# ASP.NET Web API con ApiController

Questa dispensa introduce ASP.NET Web API usando l'approccio con ApiController.
L'obiettivo e costruire endpoint HTTP chiari, tipizzati e manutenibili per restituire dati JSON.

---

## Capitolo 1 - Obiettivo

Costruire una piccola API prodotti con questi endpoint:

1. GET /api/prodotti
2. GET /api/prodotti/{id}
3. POST /api/prodotti
4. PUT /api/prodotti/{id}
5. DELETE /api/prodotti/{id}

---

## Capitolo 2 - Setup di base

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();

app.Run();
```

Con questo setup ASP.NET cerca automaticamente i controller API e mappa le rotte dichiarate con gli attributi.

---

## Capitolo 3 - Abilitare supporto XML

Per default una Web API ASP.NET restituisce soprattutto JSON.
Se vuoi supportare anche XML, devi aggiungere i formatter XML nella configurazione MVC.

### Configurazione in Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddControllers()
    .AddXmlSerializerFormatters();

var app = builder.Build();

app.MapControllers();
app.Run();
```

Se il metodo non e disponibile nel tuo progetto, aggiungi il pacchetto `Microsoft.AspNetCore.Mvc.Formatters.Xml`.

### Come funziona la risposta JSON/XML

ASP.NET usa la content negotiation in base all'header `Accept` del client:

- `Accept: application/json` -> risposta JSON
- `Accept: application/xml` -> risposta XML

Esempio di test con curl:

```bash
curl -H "Accept: application/xml" https://localhost:5001/api/prodotti
```

### Quando conviene abilitarlo

- integrazione con sistemi legacy che si aspettano XML
- interoperabilita con client enterprise non JSON-first
- migrazioni graduali da API storiche

---

## Capitolo 4 - Attributi principali usati in Web API

Questo capitolo elenca gli attributi principali che userai in questa dispensa e il loro significato.

### Attributi di controller e routing

- `[ApiController]`: abilita convenzioni API (binding piu prevedibile, validazione automatica, risposte 400 automatiche su model non valido)
- `[Route("api/[controller]")]`: definisce la rotta base del controller; `[controller]` usa il nome della classe senza suffisso `Controller`
- `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, `[HttpPatch]`: legano una action a un metodo HTTP
- `[HttpGet("{id:int}")]`: aggiunge template di rotta e vincoli (qui `id` deve essere intero)

### Attributi di binding dei parametri

- `[FromRoute]`: prende il valore dal percorso URL
- `[FromQuery]`: prende il valore dalla query string (`?pagina=2`)
- `[FromBody]`: prende il valore dal corpo della richiesta (tipicamente JSON)

### Attributi di sicurezza

- `[Authorize]`: richiede autenticazione/autorizzazione secondo policy o ruoli configurati
- `[AllowAnonymous]`: eccezione puntuale a `[Authorize]`

### Attributi di validazione model (Data Annotations)

- `[Required]`: campo obbligatorio
- `[MaxLength(n)]`: lunghezza massima stringa o array
- `[Range(min, max)]`: vincolo numerico
- `[EmailAddress]`, `[Phone]`, `[StringLength]`, `[RegularExpression]`: validazioni comuni aggiuntive

### Attributi utili per documentazione risposta

- `[ProducesResponseType(typeof(T), 200)]`: documenta tipo e status code atteso (utile per OpenAPI/Swagger)

---

## Capitolo 5 - Cosa fanno [ApiController], [Route], [Authorize]

### [ApiController]

Con `[ApiController]` il framework applica comportamenti tipici API:

1. inferenza delle fonti di binding
2. validazione automatica del model
3. ritorno automatico di `400 Bad Request` se il model non e valido
4. payload di errore standard `ValidationProblemDetails`

### [Route]

`[Route]` decide come costruire l'URL.

Esempio:

```csharp
[Route("api/[controller]")]
public class ProdottiController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult GetById(int id) => Ok();
}
```

Se il controller e `ProdottiController`, l'endpoint diventa `GET /api/prodotti/{id}`.

### [Authorize]

`[Authorize]` limita accesso a utenti autenticati o autorizzati.

```csharp
[Authorize]
[HttpGet("riservato")]
public IActionResult AreaRiservata() => Ok();
```

Puoi usarlo anche con ruoli o policy:

```csharp
[Authorize(Roles = "Admin")]
public IActionResult SoloAdmin() => Ok();
```

---

## Capitolo 6 - FromRoute, FromQuery, FromBody: cosa sono e quando usarli

### [FromRoute]

Legge dati dal percorso URL.

```csharp
[HttpGet("{id:int}")]
public IActionResult GetById([FromRoute] int id)
{
    return Ok(new { Id = id });
}
```

Request esempio: `GET /api/prodotti/10`

### [FromQuery]

Legge dati dalla query string.

```csharp
[HttpGet]
public IActionResult Search([FromQuery] string? nome, [FromQuery] int pagina = 1)
{
    return Ok(new { Nome = nome, Pagina = pagina });
}
```

Request esempio: `GET /api/prodotti?nome=pane&pagina=2`

### [FromBody]

Legge dati dal corpo JSON.

```csharp
[HttpPost]
public IActionResult Create([FromBody] ProdottoCreateRequest request)
{
    return CreatedAtAction(nameof(GetById), new { id = 1 }, request);
}
```

Request body esempio:

```json
{
  "nome": "Pane",
  "prezzo": 1.50
}
```

### Nota pratica

Con `[ApiController]`, spesso non serve specificare sempre `[FromRoute]`/`[FromQuery]`/`[FromBody]`.
Metterli esplicitamente, pero, rende il codice piu leggibile e didattico.

---

## Capitolo 7 - Modello e validazione request in dettaglio

```csharp
using System.ComponentModel.DataAnnotations;

public class ProdottoCreateRequest
{
    [Required(ErrorMessage = "Il nome e obbligatorio")]
    [MaxLength(100)]
    public string Nome { get; set; } = string.Empty;

    [Range(0.01, 10000, ErrorMessage = "Prezzo non valido")]
    public decimal Prezzo { get; set; }
}
```

Come funziona la validazione con ApiController:

1. ASP.NET deserializza il JSON in un oggetto C# (`ProdottoCreateRequest`)
2. Esegue le Data Annotations (`Required`, `Range`, ecc.)
3. Popola `ModelState` con gli errori
4. Se il controller ha `[ApiController]`, restituisce automaticamente `400 Bad Request` prima di entrare nell'action
5. Il payload di errore e tipicamente `ValidationProblemDetails`

Esempio action:

```csharp
[HttpPost]
public ActionResult<Prodotto> Create([FromBody] ProdottoCreateRequest request)
{
    // Se request e invalida, questa riga non viene raggiunta con [ApiController].
    var creato = new Prodotto { Id = 10, Nome = request.Nome, Prezzo = request.Prezzo };
    return CreatedAtAction(nameof(GetById), new { id = creato.Id }, creato);
}
```

Se vuoi comportamento personalizzato sulle risposte di validazione, puoi configurare `ApiBehaviorOptions`.

---

## Capitolo 8 - Tutti i modi comuni per restituire status code

In Web API puoi restituire uno status code in modi diversi.

### 1. Restituzione implicita (200) con tipo normale

```csharp
[HttpGet("ping")]
public string Ping()
{
    return "ok"; // 200 OK implicito
}
```

### 2. `ActionResult<T>` (misto tra valore e risultato HTTP)

```csharp
[HttpGet("{id:int}")]
public ActionResult<Prodotto> GetById(int id)
{
    var prodotto = Prodotti.FirstOrDefault(p => p.Id == id);
    if (prodotto is null) return NotFound();
    return prodotto; // 200 OK implicito
}
```

### 3. `IActionResult` con helper dedicati

```csharp
return Ok(dati);                  // 200
return CreatedAtAction(...);      // 201
return Accepted();                // 202
return NoContent();               // 204
return BadRequest("errore");     // 400
return Unauthorized();            // 401
return Forbid();                  // 403
return NotFound();                // 404
return Conflict("duplicato");    // 409
return UnprocessableEntity();     // 422
return Problem("errore server"); // 500 (o custom)
```

### 4. `StatusCode(int)` esplicito

```csharp
return StatusCode(418, "I'm a teapot");
```

### 5. `ObjectResult` / `new JsonResult(...)`

```csharp
return new ObjectResult(new { Messaggio = "Custom" }) { StatusCode = 202 };
```

### 6. Documentare status code con attributi

```csharp
[ProducesResponseType(typeof(Prodotto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[HttpGet("{id:int}")]
public ActionResult<Prodotto> GetById(int id) { ... }
```

Questo non cambia il runtime da solo, ma documenta chiaramente il contratto API.

---

## Capitolo 9 - Struttura di un controller API

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class ProdottiController : ControllerBase
{
}
```

`ControllerBase` fornisce helper come `Ok`, `NotFound`, `BadRequest`, `CreatedAtAction`.

---

## Capitolo 10 - GET lista e GET per id

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdottiController : ControllerBase
{
    private static readonly List<Prodotto> Prodotti = new()
    {
        new Prodotto { Id = 1, Nome = "Pane", Prezzo = 1.50m },
        new Prodotto { Id = 2, Nome = "Latte", Prezzo = 1.20m }
    };

    [HttpGet]
    public ActionResult<IEnumerable<Prodotto>> GetAll()
    {
        return Ok(Prodotti);
    }

    [HttpGet("{id:int}")]
    public ActionResult<Prodotto> GetById(int id)
    {
        var trovato = Prodotti.FirstOrDefault(p => p.Id == id);

        if (trovato is null)
        {
            return NotFound($"Prodotto con id {id} non trovato");
        }

        return Ok(trovato);
    }
}
```

---

## Capitolo 11 - POST con validazione automatica

```csharp
[HttpPost]
public ActionResult<Prodotto> Create([FromBody] ProdottoCreateRequest request)
{
    // Con [ApiController], se il model e invalido ASP.NET risponde 400 automaticamente.

    var nuovoId = Prodotti.Count == 0 ? 1 : Prodotti.Max(p => p.Id) + 1;
    var prodotto = new Prodotto
    {
        Id = nuovoId,
        Nome = request.Nome,
        Prezzo = request.Prezzo
    };

    Prodotti.Add(prodotto);

    return CreatedAtAction(nameof(GetById), new { id = prodotto.Id }, prodotto);
}
```

CreatedAtAction restituisce 201 Created e aggiunge l'header Location con URL della nuova risorsa.

---

## Capitolo 12 - PUT e DELETE

```csharp
[HttpPut("{id:int}")]
public IActionResult Update([FromRoute] int id, [FromBody] ProdottoCreateRequest request)
{
    var esistente = Prodotti.FirstOrDefault(p => p.Id == id);

    if (esistente is null)
    {
        return NotFound();
    }

    esistente.Nome = request.Nome;
    esistente.Prezzo = request.Prezzo;

    return NoContent();
}

[HttpDelete("{id:int}")]
public IActionResult Delete(int id)
{
    var esistente = Prodotti.FirstOrDefault(p => p.Id == id);

    if (esistente is null)
    {
        return NotFound();
    }

    Prodotti.Remove(esistente);
    return NoContent();
}
```

---

## Capitolo 13 - Autorizzazione

```csharp
using Microsoft.AspNetCore.Authorization;

[ApiController]
[Route("api/admin")]
[Authorize]
public class AdminController : ControllerBase
{
    [HttpGet("dashboard")]
    public IActionResult Dashboard()
    {
        return Ok(new { Stato = "Area riservata" });
    }
}
```

Questo endpoint richiede un utente autenticato secondo lo schema configurato nell'applicazione.

---

## Capitolo 14 - Dependency Injection

Registrazione servizio:

```csharp
builder.Services.AddSingleton<IProdottoRepository, ProdottoRepository>();
```

Iniezione nel controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProdottiController : ControllerBase
{
    private readonly IProdottoRepository _repo;

    public ProdottiController(IProdottoRepository repo)
    {
        _repo = repo;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(_repo.GetAll());
    }
}
```

---

## Capitolo 15 - Errori comuni da evitare

- Dimenticare `[ApiController]` e perdere validazione automatica e convenzioni API
- Usare `Controller` invece di `ControllerBase` in un controller solo API
- Restituire sempre `200` anche quando servono `201`, `204`, `400` o `404`
- Non tipizzare le rotte (esempio `{id:int}`) e accettare input non coerenti
- Mettere troppa logica di business nel controller invece che in servizi dedicati

---

## Esercizio guidato

### Traccia

Implementa una API ordini con:

1. GET /api/ordini
2. GET /api/ordini/{id}
3. POST /api/ordini
4. DELETE /api/ordini/{id}

### Passi consigliati

1. Definisci il model Ordine con Data Annotations
2. Crea OrdiniController con [ApiController] e [Route("api/[controller]")]
3. Implementa gli endpoint con codici di stato corretti
4. Usa CreatedAtAction nella POST
5. Usa `[FromRoute]`, `[FromQuery]`, `[FromBody]` dove opportuno
6. Usa NotFound nella GET per id e nella DELETE

---

## Conclusione

ApiController e l'approccio consigliato quando vuoi API REST strutturate in ASP.NET:

- controller chiari e organizzati
- validazione automatica del model
- codici HTTP espliciti e coerenti
- piena integrazione con Dependency Injection, autenticazione e autorizzazione

---

## Appendix - Model di esempio completo

```csharp
public class Prodotto
{
    public int Id { get; set; }
    public string Nome { get; set; } = string.Empty;
    public decimal Prezzo { get; set; }
}

public class ProdottoCreateRequest
{
    [Required(ErrorMessage = "Il nome e obbligatorio")]
    [MaxLength(100)]
    public string Nome { get; set; } = string.Empty;

    [Range(0.01, 10000, ErrorMessage = "Prezzo non valido")]
    public decimal Prezzo { get; set; }
}
```
