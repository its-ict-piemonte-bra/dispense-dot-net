# dispense-dot-net

Dispense per il corso di Programmazione .NET / C#.
Ogni dispensa include spiegazioni progressive, esempi di codice commentati e 10 esercizi guidati con soluzione.

---

## Indice

### [0 — Introduzione a .NET e C#](0-introduzione.md)

- Cos'è .NET: storia e motivazioni
- Il linguaggio C#
- Buone pratiche iniziali: commenti, convenzioni e nomenclatura
- Tipi primitivi e derivati: interi, decimali, stringhe, bool
- Vettori e matrici
- Controllo del flusso: `if`, `switch`, `while`, `do...while`, `for`
- Metodo di lavoro sugli esercizi
- 10 esercizi guidati

### [1 — Programmazione orientata agli oggetti in C#](1-oop.md)

- Perché usare l'OOP
- Classe e oggetto
- Attributi e metodi
- Incapsulamento
- Costruttori
- Proprietà
- Accessibilità: `public`, `private`, `protected`, `internal`
- Metodi: firma, overloading, parametri opzionali
- Statico e istanza
- Ereditarietà
- `protected internal`, `sealed`, `volatile`
- Polimorfismo: `virtual`, `override`, `abstract`
- Interfacce
- Generici: `List<T>`, `Dictionary<K,V>`, vincoli di tipo
- Come pensare un problema OOP
- 10 esercizi guidati

### [2 — LINQ e le collezioni in C#](2-linq.md)

- Cap. 1 — Le interfacce delle collezioni: `IEnumerable<T>`, `ICollection<T>`, `IList<T>`, `ISet<T>`, `IDictionary<K,V>`, `IQueryable<T>`
- Cap. 2 — Prerequisiti: delegate, `Func`/`Action`, lambda, predicati
- Cap. 3 — Cos'è LINQ: deferred execution, sintassi query, sintassi metodo
- Cap. 4 — Operatori principali: `Where`, `Select`, `OrderBy`, `GroupBy`, `First`/`FirstOrDefault`, `Any`/`All`, aggregazioni, `Distinct`, `Take`/`Skip`, `Join`, `ToList`/`ToArray`
- 10 esercizi guidati

### [3 — ASP.NET MVC](3-asp-net-mvc.md)

- Cap. 1 — Prerequisito: gli attributi in C#
- Cap. 2 — Cos'è ASP.NET: ciclo richiesta/risposta, metodi HTTP, status code, routing
- Cap. 3 — ASP.NET MVC: pattern, controller, routing, model, Data Annotations, Razor views, `IActionResult`
- Cap. 4 — Setup progetto MVC e routing convenzionale
- Cap. 5 — Dependency Injection e organizzazione applicativa
- 10 esercizi guidati

### [4 — ASP.NET Web API con ApiController](4-asp-net-api.md)

- Cap. 1 — Obiettivo: progettare endpoint REST con ApiController
- Cap. 2 — Setup base: `AddControllers` e `MapControllers`
- Cap. 3 — Modello e validazione con Data Annotations
- Cap. 4-7 — Controller API: GET/POST/PUT/DELETE e codici HTTP corretti
- Cap. 8 — Autorizzazione con `[Authorize]`
- Cap. 9 — Dependency Injection nei controller API
- Errori comuni + esercizio guidato

### [5 — Entity Framework Core](5-entity-framework.md)

- Cap. 1 — Cos'è EF Core: il problema senza ORM, Code First vs Database First
- Cap. 2 — DbContext e DbSet: differenza tra `List<T>` e `DbSet<T>`, `IEnumerable<T>` vs `IQueryable<T>`
- Cap. 3 — Modellare i dati: entity class, Data Annotations, relazioni uno-a-molti e molti-a-molti
- Cap. 4 — Migration: `dotnet ef migrations add`, `dotnet ef database update`
- Cap. 5 — LINQ con EF Core: traduzione in SQL, `Include`, `ThenInclude`, `AsNoTracking`, `Select` proiettato
- Cap. 6 — Operazioni CRUD: `Add`, `Find`, `Remove`, `SaveChanges`, versioni async
- Cap. 7 — EF Core con ASP.NET: registrazione DI, uso in Minimal API e in Controller MVC
- 10 esercizi guidati