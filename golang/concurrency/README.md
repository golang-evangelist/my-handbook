# Go Concurrency

Go concurrency tutorial koji vodi čitaoca od osnovnog razumevanja goroutine-a i channel-a, preko synchronization primitives i klasičnih concurrency patterns, do naprednih tema kao što su Go memory model, happens-before odnosi, data races, lock-free programming, scheduler internals, atomic operations, profiling i projektovanje visokopropusnih concurrent sistema.

Ovaj deo handbook-a nije zamišljen kao zbir izolovanih primera. Cilj je da čitalac postepeno izgradi **mentalni model Go concurrency-ja** — kako goroutine-i izvršavaju kod, kako međusobno komuniciraju, kako se usklađuju, gde nastaju race conditions i deadlock-i, kako runtime upravlja izvršavanjem i kako dizajnirati concurrent sistem koji je istovremeno korektan, skalabilan i održiv.

---

## Cilj tutoriala

Go je od samog početka dizajniran sa concurrency-jem kao jednom od svojih centralnih karakteristika.

Najpoznatiji elementi Go concurrency modela su:

* goroutines
* channels
* `select`
* `sync` package
* `sync/atomic`
* `context`
* Go scheduler
* Go memory model
* race detector
* concurrency patterns
* parallel execution preko više procesorskih jezgara

Međutim, poznavanje sintakse samo po sebi nije dovoljno za ozbiljan concurrency development.

Senior Go developer mora da razume ne samo **kako** napisati concurrent kod, već i:

* zašto je određeni concurrency model izabran;
* koji goroutine poseduje određeni podatak;
* gde se odvija komunikacija između goroutine-a;
* gde postoji synchronization boundary;
* koji događaji imaju happens-before odnos;
* da li postoji data race;
* kako može nastati deadlock;
* kako ograničiti concurrency;
* kako kontrolisati lifecycle goroutine-a;
* kako implementirati cancellation;
* kako razlikovati concurrency od parallelism-a;
* kako scheduler utiče na ponašanje programa;
* kada koristiti channels, a kada locks;
* kada koristiti atomics;
* kako meriti i profilisati concurrent sistem;
* kako izbeći goroutine leaks;
* kako projektovati sistem koji ostaje razumljiv pod velikim opterećenjem.

Zbog toga je ovaj tutorial organizovan kao progresivni learning path, a ne kao običan API reference.

---

## Šta ćeš naučiti

Po završetku kompletnog tutoriala trebalo bi da budeš sposoban da:

### Osnovni nivo

* kreiraš i kontrolišeš goroutine-e;
* razumeš lifetime goroutine-a;
* koristiš unbuffered channels;
* koristiš buffered channels;
* definišeš channel direction;
* šalješ i primaš vrednosti preko channels;
* zatvaraš channels;
* iteriraš preko channels pomoću `range`;
* razumeš blocking behavior channel operacija;
* prepoznaš osnovne concurrency probleme.

### Srednji nivo

* koristiš `select`;
* implementiraš timeout obrasce;
* koristiš `WaitGroup`;
* konstruišeš worker pool;
* implementiraš pipelines;
* koristiš fan-out/fan-in obrasce;
* kontrolišeš broj concurrent workers;
* implementiraš cancellation;
* koristiš `Mutex` i `RWMutex`;
* koristiš `sync.Once`;
* razumeš razliku između shared-state i message-passing pristupa;
* pravilno upravljaš lifecycle-om goroutine-a.

### Napredni nivo

* razumeš Go scheduler;
* razumeš ulogu `GOMAXPROCS`;
* razlikuješ concurrency od parallelism-a;
* koristiš `sync/atomic`;
* razumeš Go memory model;
* razumeš happens-before odnose;
* analiziraš data races;
* koristiš race detector;
* razumeš lock-free pristupe;
* biraš između channels, mutex-a i atomics na osnovu karakteristika problema;
* razumeš contention;
* razumeš backpressure;
* analiziraš goroutine leaks;
* dizajniraš robustne cancellation i shutdown mehanizme.

### Expert nivo

* analiziraš concurrency probleme na nivou runtime-a;
* razumeš scheduler behavior;
* razumeš interne karakteristike channels;
* analiziraš synchronization i memory-ordering probleme;
* koristiš profiling za pronalaženje concurrency bottleneck-a;
* dizajniraš visokopropusne concurrent komponente;
* prepoznaješ concurrency anti-patterns;
* razumeš trade-off između correctness, throughput, latency i complexity;
* projektuješ concurrency architecture za production sisteme;
* kritički ocenjuješ postojeći concurrent Go kod.

---

## Centralni princip

Osnovni koncept ovog tutoriala može se sažeti ovako:

> **Concurrency nije isto što i pokretanje više goroutine-a.**

Goroutine je samo jedan od mehanizama pomoću kojih Go omogućava concurrent execution.

Pravi problem je **koordinacija izvršavanja i pristupa podacima**.

Na primer, sledeći kod može pokrenuti više goroutine-a:

```go
go worker()
go worker()
go worker()
```

Ali time još nismo rešili:

* kako worker-i dobijaju posao;
* kako prijavljuju rezultat;
* kako se zna kada su završili;
* kako se zaustavljaju;
* šta se dešava ako jedan worker blokira;
* šta se dešava ako producer proizvodi brže nego što consumer obrađuje;
* kako se deli state;
* da li postoji data race;
* kako se vrši graceful shutdown.

Zbog toga tutorial progresivno prelazi sa:

```text
Goroutines
    ↓
Channels
    ↓
Select
    ↓
Synchronization
    ↓
Concurrency Patterns
    ↓
Cancellation / Lifecycle
    ↓
Scheduler / Parallelism
    ↓
Memory Model
    ↓
Atomics / Lock-Free
    ↓
Internals / Performance
    ↓
Production Architecture
```

---

## Osnovni mentalni model

Go concurrency treba posmatrati kroz nekoliko međusobno povezanih slojeva.

### 1. Execution

Ko izvršava kod?

```text
Goroutine
    ↓
Go Runtime
    ↓
Scheduler
    ↓
OS Threads
    ↓
CPU Cores
```

Ovaj sloj obuhvata goroutine lifecycle, scheduling i parallel execution.

---

### 2. Communication

Kako goroutine-i razmenjuju podatke?

```text
Producer
    │
    ▼
 Channel
    │
    ▼
Consumer
```

Channels predstavljaju jedan od centralnih mehanizama Go concurrency modela.

---

### 3. Synchronization

Kako koordinisati shared state?

```text
Goroutine A ──┐
              ├── Mutex ── Shared State
Goroutine B ──┘
```

Ovde ulaze:

* `sync.Mutex`
* `sync.RWMutex`
* `sync.Once`
* `sync.WaitGroup`
* `sync.Cond`
* `sync/atomic`

---

### 4. Memory Ordering

Kada je efekat jedne goroutine vidljiv drugoj?

Ovo pitanje vodi direktno ka:

* Go memory model-u;
* happens-before odnosima;
* synchronization events;
* atomic operations;
* data races.

Ovaj nivo je posebno važan kada se prelazi sa praktičnog korišćenja concurrency primitives na razumevanje njihovih semantičkih garancija.

---

### 5. Lifecycle

Kako concurrent komponenta počinje i završava rad?

Tipičan lifecycle izgleda ovako:

```text
Start
  │
  ▼
Running
  │
  ├── Work
  │
  ├── Wait
  │
  ├── Block
  │
  └── Error
  │
  ▼
Cancellation
  │
  ▼
Shutdown
  │
  ▼
Exit
```

Bez pravilno definisanog lifecycle-a, concurrent sistem može imati:

* goroutine leaks;
* blokirane goroutine-e;
* nepotrebno zauzimanje memorije;
* neograničeno čekanje;
* nepotpun shutdown;
* izgubljene rezultate.

---

## Concurrency i parallelism

Jedna od najvažnijih tema kroz ceo tutorial biće razlikovanje ova dva koncepta.

### Concurrency

Concurrency opisuje strukturu programa:

> više poslova može biti u toku i njihov napredak može biti koordinisan.

### Parallelism

Parallelism opisuje istovremeno izvršavanje:

> više operacija se zaista izvršava u istom trenutku na različitim CPU execution resources.

Pojednostavljeno:

```text
Concurrency

Task A ────────┐
               ├── interleaving
Task B ────────┘
```

nasuprot:

```text
Parallelism

CPU 1:  Task A ─────────────

CPU 2:  Task B ─────────────
```

Go omogućava oba modela, ali njihovo ponašanje zavisi od runtime scheduler-a, `GOMAXPROCS`, dostupnih CPU resursa i prirode samog workload-a.

---

## Communication vs Shared Memory

Jedna od ključnih odluka pri dizajniranju concurrent Go programa jeste način upravljanja state-om.

Postoje dva osnovna pristupa.

### Shared memory + synchronization

Više goroutine-a pristupa istom state-u, uz synchronization primitives.

```go
var mu sync.Mutex
var counter int

func increment() {
    mu.Lock()
    counter++
    mu.Unlock()
}
```

### Message passing

Goroutine-i razmenjuju podatke preko channels.

```go
jobs := make(chan int)

go worker(jobs)

jobs <- 42
```

Ni jedan pristup nije univerzalno bolji.

Dobro dizajniran Go kod treba da bira mehanizam prema problemu koji rešava.

---

## Kome je tutorial namenjen

Tutorial je namenjen čitaocima koji žele da pređu od osnovnog poznavanja Go concurrency-ja ka ozbiljnom production-level razumevanju.

Posebno je koristan za:

* Go Junior developere koji žele da izgrade čvrstu osnovu;
* Go Medior developere koji žele da razumeju concurrency patterns;
* Senior Go developere koji žele dublje razumevanje runtime-a i memory model-a;
* developere koji rade na high-throughput servisima;
* developere koji rade sa worker pool-ovima i pipelines;
* developere koji optimizuju concurrent workloads;
* developere koji analiziraju race conditions i deadlocks;
* developere koji projektuju concurrent libraries i infrastructure components.

---

## Preduslovi

Pre početka preporučuje se dobro poznavanje:

* Go syntax;
* variables i constants;
* functions;
* methods;
* structs;
* interfaces;
* slices i maps;
* pointers;
* error handling;
* packages;
* modules;
* basic testing;
* osnovnog korišćenja standardne biblioteke.

Nije neophodno prethodno duboko poznavanje concurrency-ja.

Tutorial je konstruisan tako da se concurrency koncepti uvode postepeno.

---

## Organizacija tutoriala

Kompletan materijal je podeljen na četiri modula.

```text
golang/concurrency/
│
├── README.md
│
└── docs/
    │
    ├── module-1/
    │   └── README.md
    │
    ├── module-2/
    │   └── README.md
    │
    ├── module-3/
    │   └── README.md
    │
    └── module-4/
        └── README.md
```

Svaki modul predstavlja zasebnu fazu u učenju.

---

## Learning Path

Preporučeni redosled je:

```text
Module 1
Fundamentals
    │
    ▼
Module 2
Coordination & Patterns
    │
    ▼
Module 3
Synchronization & Runtime
    │
    ▼
Module 4
Advanced Concurrency
```

Ne preporučuje se preskakanje ranijih modula.

Napredne teme kao što su memory model, lock-free programming i scheduler internals mnogo su lakše za razumevanje kada čitalac već ima čvrst mentalni model goroutine-a, channels i synchronization primitives.

---

## Šta ovaj tutorial nije

Ovaj tutorial nije:

* samo `goroutine` cheat sheet;
* samo pregled `sync` package-a;
* samo lista concurrency patterns;
* zamena za Go specification;
* samo zbir gotovih snippets-a;
* vodič koji podrazumeva da je više goroutine-a automatski bolje.

Fokus je na **razumevanju modela, trade-off-a i correctness-a**.

Kod concurrent programa, kod koji "radi" nije nužno kod koji je korektan.

Može:

* raditi u lokalnom testu;
* raditi pod malim opterećenjem;
* raditi većinu vremena;
* proći unit testove;

a ipak sadržati:

* data race;
* deadlock;
* starvation;
* livelock;
* goroutine leak;
* unbounded concurrency;
* nepredvidiv shutdown;
* memory-ordering problem.

Zbog toga će se kroz tutorial posebna pažnja posvećivati upravo tim problemima.

---

## Filosofija učenja

Svaka ozbiljna concurrency tema treba da se posmatra kroz najmanje četiri pitanja:

### 1. Correctness

Da li je program semantički korektan?

### 2. Safety

Da li može doći do:

* data race-a;
* invalidnog state-a;
* deadlock-a;
* neželjenog shared-state pristupa?

### 3. Liveness

Da li sistem nastavlja da napreduje?

Ovde se pojavljuju problemi poput:

* deadlock-a;
* starvation-a;
* livelock-a;
* blocking-a;
* goroutine leak-a.

### 4. Performance

Kako se sistem ponaša pod opterećenjem?

Posmatraju se:

* throughput;
* latency;
* contention;
* CPU utilization;
* memory usage;
* scheduling overhead;
* synchronization overhead.

Tek kada su ova četiri aspekta razmatrana zajedno, možemo govoriti o kvalitetnom concurrent dizajnu.

---

## Napomena o terminologiji

U ovom tutorialu će se koristiti originalna Go terminologija zajedno sa odgovarajućim objašnjenjima:

| Termin             | Značenje                                                                                |
| ------------------ | --------------------------------------------------------------------------------------- |
| Goroutine          | lagana concurrent execution jedinica kojom upravlja Go runtime                          |
| Channel            | typed communication mechanism između goroutine-a                                        |
| Buffered channel   | channel sa internim bufferom                                                            |
| Unbuffered channel | channel bez internog buffer-a                                                           |
| Select             | multiplexing mechanism za channel operacije                                             |
| Mutex              | mutual exclusion synchronization primitive                                              |
| RWMutex            | reader/writer mutual exclusion primitive                                                |
| WaitGroup          | primitive za čekanje grupe goroutine-a                                                  |
| Atomic operation   | operacija sa atomicity garancijom                                                       |
| Data race          | konkurentni pristup istoj memorijskoj lokaciji bez odgovarajuće synchronization zaštite |
| Deadlock           | stanje u kojem goroutine-i međusobno čekaju i sistem ne može da napreduje               |
| Goroutine leak     | goroutine koji ostaje aktivan iako više nije potreban                                   |
| Scheduler          | Go runtime komponenta koja raspoređuje goroutine-e za izvršavanje                       |
| Memory model       | pravila koja definišu vidljivost i ordering memorijskih operacija                       |
| Happens-before     | ordering odnos između događaja relevantan za synchronization i memory visibility        |
| Contention         | konkurencija više izvršilaca za isti resource                                           |
| Backpressure       | mehanizam kojim sporiji consumer ograničava producer                                    |
| Lock-free          | algoritamski pristup koji izbegava tradicionalne locks uz određene progress guarantees  |

---

## Glavni cilj

Na kraju ovog tutoriala cilj nije da znaš napamet:

```go
go func() {}
```

ili:

```go
select {
case <-ch:
}
```

Pravi cilj je da možeš da pogledaš concurrent sistem i odgovoriš na pitanja:

```text
Ko radi?
Ko poseduje state?
Ko šalje podatke?
Ko prima podatke?
Ko blokira?
Ko koga čeka?
Šta garantuje synchronization?
Šta se dešava ako producer bude brži?
Šta se dešava ako consumer nestane?
Kako se sistem gasi?
Postoji li data race?
Postoji li deadlock?
Koliko concurrency-ja je dozvoljeno?
Da li je workload CPU-bound ili I/O-bound?
Da li je problem concurrency ili parallelism?
Gde je contention?
Kako ćemo dokazati da je sistem korektan?
Kako ćemo dokazati da je dovoljno brz?
```

Ako možeš sistematski da odgovoriš na ova pitanja, onda concurrency više nije samo skup Go API-ja, već deo tvog inženjerskog mentalnog modela.

---

# Go Concurrency

## Arhitektura tutoriala

Kompletan concurrency curriculum organizovan je kao progresija od osnovnih mehanizama ka dubokom razumevanju runtime-a, memorijske vidljivosti i production-level concurrency dizajna.

```text
                    Go Concurrency
                          │
          ┌───────────────┴───────────────┐
          │                               │
     Communication                  Synchronization
          │                               │
      Channels                         sync
          │                               │
          └───────────────┬───────────────┘
                          │
                    Coordination
                          │
             ┌────────────┴────────────┐
             │                         │
          Patterns                 Lifecycle
             │                         │
       Worker Pools              Cancellation
       Pipelines                 Timeouts
       Fan-Out/Fan-In             Shutdown
             │                         │
             └────────────┬────────────┘
                          │
                     Runtime Model
                          │
              Scheduler / Parallelism
                          │
                          ▼
                  Memory Semantics
                          │
              Memory Model / Atomic
                          │
                          ▼
                 Advanced Concurrency
                          │
             Lock-Free / Internals
                          │
                          ▼
             Performance & Production
```

Ovakva organizacija je namerna.

Čitalac prvo uči **kako goroutine-i komuniciraju**, zatim **kako se koordiniraju**, nakon toga **kako runtime izvršava concurrent program**, a tek onda prelazi na detalje memory model-a, atomics, lock-free algoritama i performance engineering-a.

---

# Moduli

## Module 1 — Concurrency Fundamentals

Prvi modul predstavlja temelj čitavog tutoriala.

U njemu se uvode osnovni building blocks Go concurrency modela:

* goroutines;
* channels;
* unbuffered channels;
* buffered channels;
* channel directions;
* `range` preko channels;
* channel closing;
* osnovna pravila ownership-a i lifecycle-a.

Struktura:

```text
module-1/
│
├── 01-goroutines.md
├── 02-channels.md
├── 03-unbuffered-channels.md
├── 04-buffered-channels.md
├── 05-channel-directions.md
├── 06-range-over-channels.md
├── 07-close-channel.md
└── 08-module-1-summary-and-exercises.md
```

### Glavno pitanje modula

> Kako pokrenuti više goroutine-a i omogućiti im da bezbedno komuniciraju?

Modul počinje sa najjednostavnijim execution modelom:

```go
go worker()
```

i zatim uvodi channels kao komunikacioni mehanizam:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

Nakon toga se postepeno uvode:

```text
unbuffered
    ↓
buffered
    ↓
directional
    ↓
range
    ↓
close
```

Posebna pažnja treba da bude posvećena činjenici da `close` nije isto što i "brisanje channel-a", niti je zatvaranje channel-a način da se automatski zaustave goroutine-i.

Channel lifecycle je odgovornost dizajna.

---

# Module 2 — Coordination and Concurrency Patterns

Drugi modul prelazi sa osnovnih building blocks na koordinaciju većeg broja goroutine-a.

Struktura:

```text
module-2/
│
├── 01-select.md
├── 02-select-default.md
├── 03-waitgroup.md
├── 04-worker-pools.md
├── 05-pipelines.md
├── 06-fan-out.md
├── 07-fan-in.md
└── 08-module-2-summary-and-exercises.md
```

Centralne teme su:

* `select`;
* non-blocking channel operations;
* `WaitGroup`;
* worker pools;
* pipelines;
* fan-out;
* fan-in.

### Glavno pitanje modula

> Kako koordinisati veliki broj concurrent aktivnosti?

Ako Module 1 pokazuje kako napraviti osnovnu komunikaciju:

```text
Goroutine A ── Channel ──> Goroutine B
```

Module 2 prelazi na strukture poput:

```text
                    ┌── Worker 1 ──┐
                    │              │
Producer ── Jobs ───┼── Worker 2 ──┼── Results
                    │              │
                    └── Worker 3 ──┘
```

ili:

```text
Input
  │
  ▼
Stage 1
  │
  ▼
Stage 2
  │
  ▼
Stage 3
  │
  ▼
Output
```

Ovo je trenutak kada concurrency prestaje da bude samo pitanje "kako pokrenuti goroutine" i postaje pitanje **arhitekture sistema**.

---

## `select` kao concurrency multiplexer

`select` omogućava goroutine-u da čeka više mogućih channel operacija.

Konceptualno:

```text
                 ┌── channel A
                 │
Goroutine ───────┼── channel B
                 │
                 └── channel C
```

To omogućava implementaciju:

* multiplexing-a;
* cancellation-a;
* timeout-a;
* event loop-ova;
* graceful shutdown-a;
* non-blocking operations.

Posebno je važno razumeti razliku između:

```go
select {
case value := <-ch:
    // ...
}
```

i:

```go
select {
case value := <-ch:
    // ...
default:
    // do not block
}
```

`default` menja blocking semantics kompletnog `select` izraza i zato zahteva pažljivu upotrebu.

---

# Module 3 — Synchronization and Runtime

Treći modul uvodi dublji synchronization model i osnovne runtime koncepte.

Struktura:

```text
module-3/
│
├── 01-mutex.md
├── 02-rwmutex.md
├── 03-once.md
├── 04-timeouts.md
├── 05-cancellation.md
├── 06-go-scheduler.md
├── 07-gomaxprocs.md
├── 08-parallelism-vs-concurrency.md
└── 09-module-3-summary-and-exercises.md
```

Ovde se prelazi sa:

```text
communication
```

na:

```text
coordination + shared state + execution model
```

### Glavno pitanje modula

> Šta se dešava kada goroutine-i dele state ili kada njihovo izvršavanje mora precizno da se koordinira?

Uvod u `Mutex` menja perspektivu.

Umesto:

```text
Goroutine → Channel → Goroutine
```

dobijamo:

```text
Goroutine A ──┐
              │
              ▼
           Shared State
              ▲
              │
Goroutine B ──┘
```

uz synchronization:

```text
Goroutine A ── Lock ── Shared State ── Unlock
Goroutine B ── Lock ── Shared State ── Unlock
```

---

# Synchronization primitives

U ovom sloju čitalac uči kada i zašto koristiti:

```text
Mutex
  │
  ├── mutual exclusion
  │
  └── protection of shared state


RWMutex
  │
  ├── multiple readers
  │
  └── exclusive writer


Once
  │
  └── one-time initialization


WaitGroup
  │
  └── lifecycle coordination
```

Bitno je razlikovati **mutual exclusion** od **coordination**.

Na primer, `Mutex` odgovara na pitanje:

> Ko sme trenutno da pristupi ovom shared state-u?

Dok `WaitGroup` odgovara na pitanje:

> Da li je grupa goroutine-a završila?

To nisu isti problemi i ne treba ih rešavati istim primitive-om.

---

# Timeouts i Cancellation

Concurrent sistemi moraju imati kontrolisan lifecycle.

Sistem koji može da pokrene goroutine-e, ali ne može pouzdano da ih zaustavi, nije kompletno projektovan concurrent sistem.

Tipičan lifecycle:

```text
Start
  │
  ▼
Work
  │
  ├───────────────┐
  │               │
  ▼               ▼
Success         Cancellation
  │               │
  │               ▼
  │            Cleanup
  │               │
  └───────┬───────┘
          ▼
         Exit
```

Timeout predstavlja vremensko ograničenje operacije.

Cancellation predstavlja signal da operacija više nije potrebna ili da treba da bude prekinuta.

U realnim sistemima ova dva koncepta često rade zajedno.

---

# Go Scheduler

Nakon što čitalac savlada osnovne synchronization primitive, tutorial prelazi na pitanje:

> Ko zapravo odlučuje kada će se određena goroutine izvršavati?

Odgovor je Go runtime scheduler.

Pojednostavljeni model:

```text
G = Goroutine
M = OS Thread
P = Processor / execution context
```

Možemo ga predstaviti kao:

```text
        Go Runtime Scheduler

             Global / Local Queues
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Goroutine   Goroutine   Goroutine
          │           │           │
          └───────────┼───────────┘
                      ▼
                     P
                      │
                      ▼
                     M
                      │
                      ▼
                     CPU
```

Ovo predstavlja uvod u dublji runtime model.

Čitalac mora razumeti da:

```go
go fn()
```

ne znači:

```text
create OS thread
```

niti znači:

```text
execute immediately
```

Goroutine se predaje runtime scheduler-u, koji zatim odlučuje kada i gde će biti izvršena.

---

# `GOMAXPROCS`

`GOMAXPROCS` uvodi dodatni nivo razumevanja.

Potrebno je razlikovati:

```text
Goroutines
```

od:

```text
OS threads
```

i:

```text
logical processors available to Go scheduler
```

Konceptualno:

```text
Many Goroutines
       │
       ▼
   Scheduler
       │
       ▼
   P contexts
       │
       ▼
   OS Threads
       │
       ▼
     CPUs
```

Zbog toga broj goroutine-a i nivo stvarnog CPU parallelism-a nisu ista stvar.

Može postojati:

```text
100,000 goroutines
```

a samo ograničen broj goroutine-a može u datom trenutku biti aktivno izvršavan na CPU-u.

---

# Module 4 — Advanced Go Concurrency

Četvrti modul predstavlja najdublji sloj tutoriala.

Za razliku od prethodnih modula, ovde fokus više nije samo na pravilnom korišćenju concurrency API-ja.

Fokus se pomera na:

```text
Semantics
Internals
Memory
Atomics
Correctness
Performance
Architecture
```

Core sadržaj je organizovan oko naprednih concurrency tema, dok dodatni `extra/` materijal produbljuje pojedine oblasti do nivoa runtime-a i performance engineering-a.

Konceptualno:

```text
                 Module 4
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    Memory        Atomics       Internals
       │             │             │
       ▼             ▼             ▼
   Ordering       CAS/etc.      Channels
   Visibility     Lock-free      Scheduler
   Happens-before               Runtime
       │             │             │
       └─────────────┼─────────────┘
                     │
                     ▼
               Performance
                     │
                     ▼
              Production Design
```

---

# Napredni nivo razmišljanja

Na ovom nivou nije dovoljno reći:

> "Koristi mutex."

Potrebno je pitati:

```text
Koliko često se state menja?
Koliko često se čita?
Koliko goroutine-a konkuriše?
Koliko dugo traje critical section?
Da li postoji contention?
Da li je lock coarse-grained ili fine-grained?
Da li je channel prirodniji abstraction?
Da li atomic operation rešava problem?
Da li je lock-free pristup opravdan?
Koje su progress guarantees?
Kako izgleda failure mode?
Kako izgleda shutdown?
Kakav je throughput?
Kakva je latency distribucija?
```

Ovo je razlika između **korišćenja concurrency API-ja** i **concurrency engineering-a**.

---

# Memory Model

Jedna od najvažnijih naprednih tema je Go memory model.

Concurrent program može imati više goroutine-a koje izvršavaju:

```text
Read
Write
Read
Write
```

ali sama činjenica da su operacije napisane određenim redosledom u source code-u ne znači da druga goroutine automatski ima odgovarajuću synchronization garanciju.

Zato je potrebno razumeti:

* memory visibility;
* synchronization;
* ordering;
* happens-before;
* atomicity;
* data races.

Mentalni model:

```text
Goroutine A
    │
    │ Write
    ▼
  Memory
    │
    │ synchronization
    ▼
Goroutine B
    │
    │ Read
    ▼
  Observed State
```

Bez odgovarajućeg synchronization odnosa ne treba pretpostavljati da će konkurentni pristup state-u imati željene semantičke garancije.

---

# Happens-Before

Happens-before predstavlja jedan od ključnih koncepata za formalno razumevanje concurrency correctness-a.

Umesto da razmišljamo samo:

```text
A happened first
B happened second
```

potrebno je razmišljati:

```text
A happens-before B
```

jer je upravo ordering relationship između događaja osnova za reasoning o vidljivosti memorijskih efekata.

Konceptualno:

```text
Event A
   │
   │ happens-before
   ▼
Event B
```

Kasnije teme koriste ovaj model za objašnjenje:

* channels;
* locks;
* atomic operations;
* synchronization;
* data races;
* visibility guarantees.

---

# Data Races

Data race nastaje kada konkurentne goroutine-e pristupaju istoj memorijskoj lokaciji, pri čemu najmanje jedan pristup predstavlja write, a između pristupa ne postoji odgovarajuća synchronization garancija.

Konceptualno:

```text
Goroutine A              Goroutine B
     │                        │
     │ Write(x)               │
     │                        │
     │                  Read(x)
     │                        │
     └──────────┬─────────────┘
                │
             Race?
```

Problem data race-a nije samo "pogrešna vrednost".

Race znači da program krši concurrency assumptions na nivou memory semantics-a.

Zato će advanced deo posebno razlikovati:

```text
Race condition
```

od:

```text
Data race
```

To nisu sinonimi.

---

# Atomic Operations

Atomics predstavljaju još jedan nivo synchronization-a.

Umesto:

```text
Lock
  ↓
Read
  ↓
Modify
  ↓
Write
  ↓
Unlock
```

određene operacije mogu biti izvedene pomoću atomic primitives.

Konceptualno:

```text
Atomic Read
Atomic Write
Atomic Add
Compare-And-Swap
```

Ali atomic operations nisu automatska zamena za mutex-e.

Njihova upotreba zahteva razumevanje:

* atomicity;
* memory ordering;
* synchronization;
* alignment;
* contention;
* invariants;
* correctness proofs.

---

# Lock-Free Programming

Lock-free algoritmi predstavljaju napredniji pristup concurrency design-u.

Tipičan obrazac koristi compare-and-swap:

```text
Read current value
       │
       ▼
Compute new value
       │
       ▼
CAS(old, new)
   │         │
success    failure
   │         │
   ▼         └── retry
Done
```

Ovaj model može smanjiti određene probleme povezane sa locks, ali povećava kompleksnost algoritma.

Zato:

```text
Lock-free ≠ automatically faster
```

i:

```text
Lock-free ≠ automatically better
```

U production kodu correctness i maintainability često imaju veću vrednost od marginalnog smanjenja synchronization overhead-a.

---

# Performance Engineering

Advanced concurrency ne može biti kompletan bez merenja.

Intuicija često nije dovoljna.

Concurrent sistem može imati:

* više goroutine-a;
* više workers;
* manje locks;

a ipak biti sporiji.

Razlozi mogu uključivati:

```text
Contention
    ↓
Cache effects
    ↓
Synchronization overhead
    ↓
Scheduling overhead
    ↓
Excessive allocations
    ↓
Poor workload partitioning
```

Zato performance treba posmatrati kroz merljive karakteristike:

| Metrika         | Pitanje                                               |
| --------------- | ----------------------------------------------------- |
| Throughput      | Koliko posla sistem završi u jedinici vremena?        |
| Latency         | Koliko dugo traje pojedinačna operacija?              |
| CPU utilization | Koliko CPU resursa se koristi?                        |
| Contention      | Koliko vremena goroutine-i provode čekajući resource? |
| Memory          | Koliko memorije concurrent workload zahteva?          |
| Goroutine count | Koliko goroutine-a postoji i koliko ih je aktivno?    |
| Blocking        | Gde goroutine-i čekaju?                               |
| Scalability     | Kako sistem reaguje na povećanje workload-a?          |

---

# Production Concurrency

Krajnji cilj nije samo napisati concurrent program.

Cilj je napisati concurrent program koji se može:

* testirati;
* profilisati;
* debagovati;
* održavati;
* skalirati;
* bezbedno zaustaviti;
* razumeti nakon nekoliko meseci;
* koristiti pod production workload-om.

Zbog toga se advanced concurrency mora posmatrati kroz tri dimenzije:

```text
              Concurrent System
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Correctness   Performance   Operability
        │            │            │
     No races     Throughput    Observability
     No leaks     Latency       Debuggability
     No deadlocks Scalability   Graceful shutdown
```

Dobro dizajniran concurrent sistem mora biti dobar u sve tri.

---

# Kako koristiti ovaj tutorial

Preporučeni način rada je:

```text
1. Read
   ↓
2. Understand the model
   ↓
3. Study the example
   ↓
4. Modify the example
   ↓
5. Create a failure
   ↓
6. Diagnose the failure
   ↓
7. Fix the design
   ↓
8. Measure
   ↓
9. Review the trade-offs
```

Posebno je korisno namerno napraviti:

* data race;
* deadlock;
* goroutine leak;
* blocked send;
* blocked receive;
* unbounded worker creation;
* incorrect channel closing;
* cancellation bug.

Zatim problem treba analizirati i popraviti.

To daje mnogo dublje razumevanje od pukog čitanja ispravnog primera.

---

# Pravilo za izbor concurrency mehanizma

Kroz tutorial se može koristiti sledeća heuristika kao početna tačka:

```text
Da li goroutine-i treba da razmenjuju podatke?
        │
       DA
        │
        ▼
     Channel
        │
       NE
        │
        ▼
Da li dele mutable state?
        │
       DA
        │
        ▼
   Synchronization
        │
        ├── Mutex / RWMutex
        │
        └── Atomic
```

Ali ovo je samo početna heuristika.

Prava odluka zavisi od:

* ownership model-a;
* lifetime-a state-a;
* access pattern-a;
* contention-a;
* latency zahteva;
* throughput zahteva;
* complexity-a;
* failure semantics;
* cancellation model-a.

---

# Završna perspektiva

Go concurrency treba posmatrati kao slojevit sistem:

```text
                    Go Concurrency
                         │
              ┌──────────┴──────────┐
              │                     │
          Goroutines             Channels
              │                     │
              └──────────┬──────────┘
                         │
                      Select
                         │
                         ▼
                 Coordination
                         │
                         ▼
              Synchronization
                         │
                         ▼
                Concurrency Patterns
                         │
                         ▼
                 Lifecycle Control
                         │
                         ▼
                   Go Scheduler
                         │
                         ▼
             Concurrency / Parallelism
                         │
                         ▼
                  Memory Model
                         │
                         ▼
                    Atomics
                         │
                         ▼
                  Lock-Free Code
                         │
                         ▼
                Runtime Internals
                         │
                         ▼
             Performance Engineering
                         │
                         ▼
             Production Architecture
```

Svaki sledeći modul koristi mentalne modele iz prethodnog.

Zbog toga se preporučuje linearno prolazak kroz:

```text
Module 1 → Module 2 → Module 3 → Module 4
```

a ne izolovano čitanje pojedinačnih tema.

Kompletna navigacija i detaljna struktura svakog modula biće predstavljeni u narednim sekcijama ovog handbook-a.

---

# Go Concurrency

## Concurrency Concepts Map

Da bi se Go concurrency pravilno razumeo, nije dovoljno posmatrati teme kao izolovane API-je. Svaka tema predstavlja deo šireg sistema.

Sledeća mapa predstavlja konceptualnu zavisnost između najvažnijih oblasti:

```text
                         Goroutines
                              │
                              ▼
                         Channels
                              │
                ┌─────────────┴─────────────┐
                │                           │
           Communication                Blocking
                │                           │
                ▼                           ▼
             Select                  Synchronization
                │                           │
                ▼                           ├── Mutex
          Coordination                      ├── RWMutex
                │                           ├── Once
                │                           ├── WaitGroup
                ▼                           └── Atomic
        Concurrency Patterns
                │
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
      Worker  Pipeline  Fan-Out/Fan-In
      Pools
        │       │        │
        └───────┼────────┘
                ▼
             Lifecycle
                │
       ┌────────┼─────────┐
       │        │         │
       ▼        ▼         ▼
   Timeout  Cancellation Shutdown
                │
                ▼
          Runtime Scheduler
                │
       ┌────────┴────────┐
       │                 │
       ▼                 ▼
   GOMAXPROCS       Parallelism
       │                 │
       └────────┬────────┘
                ▼
           Memory Model
                │
       ┌────────┴────────┐
       │                 │
       ▼                 ▼
 Happens-Before      Data Races
       │
       ▼
    Atomics
       │
       ▼
 Lock-Free Programming
       │
       ▼
 Internals & Performance
```

Ova mapa pokazuje važnu stvar:

> Napredni concurrency koncepti nisu odvojeni od osnovnih; oni su njihova posledica.

Na primer, nemoguće je kvalitetno razumeti lock-free programming bez razumevanja atomic operations, a atomic operations bez memory model-a.

Slično tome, nije moguće kvalitetno projektovati worker pool bez razumevanja channels, blocking behavior-a i lifecycle-a goroutine-a.

---

# Fundamental Concurrency Questions

Tokom rada kroz ceo tutorial, svaki concurrent dizajn treba posmatrati kroz nekoliko osnovnih pitanja.

## 1. Ko izvršava posao?

Identifikuj goroutine-e.

```text
Request
   │
   ▼
Goroutine A
   │
   ├── Goroutine B
   ├── Goroutine C
   └── Goroutine D
```

Treba znati:

* ko kreira goroutine;
* ko je njen owner;
* koliko dugo može da živi;
* šta je njen posao;
* pod kojim uslovima se završava.

Ako nije jasno ko je odgovoran za lifecycle goroutine-a, vrlo lako nastaje goroutine leak.

---

## 2. Ko poseduje state?

Za svaki mutable state treba definisati ownership.

Na primer:

```text
                ┌── Goroutine A owns state
                │
Shared State ────┼── Goroutine B accesses state
                │
                └── Goroutine C accesses state
```

Ako više goroutine-a direktno menja state, mora postojati jasan synchronization model.

Alternative mogu biti:

```text
Shared ownership + Mutex
```

ili:

```text
Single owner + Channels
```

Ownership je često važniji od samog izbora primitive-a.

---

## 3. Kako se prenose podaci?

Treba identifikovati communication path.

```text
Producer
   │
   ▼
Channel
   │
   ▼
Consumer
```

ili:

```text
Producer
   │
   ▼
Shared State
   │
   ▲
   │
Consumer
```

Prvi model favorizuje message passing.

Drugi model favorizuje shared memory + synchronization.

---

## 4. Ko koga čeka?

Svaka blocking operacija treba da ima jasno objašnjenje.

Na primer:

```go
ch <- value
```

može blokirati ako nema odgovarajućeg receiver-a ili slobodnog prostora u buffered channel-u.

Slično:

```go
value := <-ch
```

može blokirati dok vrednost ne postane dostupna.

Kod mutex-a:

```go
mu.Lock()
```

može blokirati dok drugi goroutine ne oslobodi lock.

Zato je važno mapirati:

```text
Waiter → Resource → Owner
```

Ako postoji ciklus:

```text
A waits for B
B waits for C
C waits for A
```

postoji potencijalni deadlock.

---

# Blocking Model

Blocking nije nužno problem.

Blocking je često sastavni deo ispravnog concurrency dizajna.

Problem nastaje kada blocking nije deo kontrolisanog lifecycle-a.

Na primer:

```text
Producer
   │
   ▼
 Channel
   │
   ▼
Consumer
```

Ako consumer privremeno nije spreman, producer može legitimno čekati.

Ali ako consumer nikada neće doći:

```text
Producer
   │
   ▼
 Channel
   │
   X
Consumer never starts
```

producer može ostati zauvek blokiran.

Zato je kod svakog blocking operation-a potrebno postaviti pitanje:

> Ko će omogućiti da se ova operacija završi?

---

# Goroutine Lifecycle

Jedan od najvažnijih design principa ovog tutoriala jeste da svaka goroutine treba da ima jasno definisan lifecycle.

Minimalni model:

```text
             Start
               │
               ▼
             Running
               │
        ┌──────┼──────┐
        │      │      │
        ▼      ▼      ▼
      Block   Work   Error
        │      │      │
        └──────┼──────┘
               ▼
          Cancellation
               │
               ▼
            Cleanup
               │
               ▼
             Exit
```

Treba znati:

* ko je pokreće;
* šta radi;
* na čemu može da blokira;
* kako prima cancellation signal;
* kako završava;
* ko čeka njen završetak.

Ovo je posebno važno kod servera, worker pool-ova, background processors i pipelines.

---

# Structured Concurrency

Go nema jedan centralni jezički construct koji sam po sebi nameće structured concurrency, ali principi structured concurrency mogu i treba da se primenjuju u dizajnu.

Ideja je da concurrent child operations imaju jasno definisan:

* owner;
* scope;
* lifetime;
* cancellation path;
* error propagation;
* shutdown behavior.

Umesto:

```text
Main
 │
 ├── goroutine
 ├── goroutine
 ├── goroutine
 ├── goroutine
 └── goroutine
```

bez jasnog lifecycle-a, cilj je:

```text
Parent Scope
 │
 ├── Child A
 ├── Child B
 └── Child C
      │
      └── all terminate with parent scope
```

Ovaj način razmišljanja je posebno važan u većim sistemima.

---

# Channel Ownership

Jedno od praktičnih pravila koje će se ponavljati kroz tutorial jeste:

> Onaj ko kreira i proizvodi vrednosti na channel-u najčešće treba da bude odgovoran za njegovo zatvaranje.

Primer:

```go
func producer() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 10; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Consumer samo čita:

```go
for value := range producer() {
    // use value
}
```

Ovde je lifecycle jasan:

```text
Producer
   │
   ├── creates channel
   ├── sends values
   └── closes channel
           │
           ▼
Consumer
   │
   └── ranges until closed
```

Ovaj princip smanjuje broj situacija u kojima više goroutine-a pokušava da upravlja istim channel lifecycle-om.

---

# Close Semantics

`close` treba razumeti kao signal:

> više vrednosti neće biti poslate.

Nije:

* signal za automatsko gašenje svih goroutine-a;
* signal da je channel "obrisan";
* način da se prekine receiver;
* mehanizam za cancellation.

Na primer:

```go
close(ch)
```

omogućava receiver-u da detektuje završetak stream-a:

```go
value, ok := <-ch

if !ok {
    // channel is closed and drained
}
```

ili:

```go
for value := range ch {
    // ...
}
```

`range` završava kada je channel zatvoren i kada su preostale vrednosti pročitane.

---

# Select kao Lifecycle Tool

`select` nije samo alat za čekanje na više channels.

On predstavlja jednu od ključnih komponenti za lifecycle control.

Na primer:

```go
select {
case value := <-jobs:
    process(value)

case <-done:
    return
}
```

Dobijamo:

```text
             ┌── jobs ───────► Process
             │
Worker ──────┤
             │
             └── done ───────► Exit
```

Worker sada ima dva moguća događaja:

1. novi posao;
2. signal za završetak.

Ovaj obrazac se pojavljuje u velikom broju production sistema.

---

# Backpressure

Backpressure je jedan od najvažnijih praktičnih concurrency koncepata.

Pretpostavimo:

```text
Producer: 100,000 jobs/s
Consumer: 10,000 jobs/s
```

Ako sistem nema ograničenje:

```text
Producer
   │
   ▼
Unlimited Queue
   │
   ▼
Slow Consumer
```

queue može nastaviti da raste.

Posledice mogu biti:

* povećanje memorijske potrošnje;
* povećanje latency-ja;
* eventualni OOM;
* sistemska degradacija.

Sa kontrolisanim bufferom:

```text
Producer
   │
   ▼
Bounded Buffer
   │
   ▼
Consumer
```

producer u nekom trenutku mora da čeka.

To uvodi prirodni backpressure.

---

# Bounded Concurrency

Jedan od najčešćih concurrency anti-pattern-a jeste kreiranje neograničenog broja goroutine-a.

Na primer:

```go
for _, job := range jobs {
    go process(job)
}
```

Ako `jobs` ima milion elemenata, pokušava se kreirati ogroman broj concurrent aktivnosti.

Bolji model može biti worker pool:

```text
Jobs
 │
 ▼
Bounded Queue
 │
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

Sistem tada eksplicitno definiše concurrency limit.

To omogućava kontrolu:

* CPU usage;
* memory usage;
* downstream pressure;
* number of connections;
* database load;
* API rate;
* latency.

---

# Fan-Out / Fan-In

Fan-out i fan-in predstavljaju osnovne concurrency composition patterns.

## Fan-Out

Jedan source distribuira posao na više workers:

```text
              ┌── Worker A ──┐
              │              │
Input ────────┼── Worker B ──┼── Results
              │              │
              └── Worker C ──┘
```

Cilj je povećanje throughput-a.

---

## Fan-In

Više izvora šalje rezultate u jedan output:

```text
Worker A ──┐
           │
Worker B ──┼──► Aggregator
           │
Worker C ──┘
```

Cilj je objedinjavanje rezultata.

---

## Combined

Najčešće se koriste zajedno:

```text
                   ┌── Worker A ──┐
                   │              │
Input ──► Fan-Out ─┼── Worker B ──┼──► Fan-In ──► Output
                   │              │
                   └── Worker C ──┘
```

Ovo predstavlja jednu od osnovnih arhitektonskih formi za concurrent pipelines.

---

# Pipeline Design

Pipeline deli obradu na stage-ove.

```text
Input
  │
  ▼
Stage A
  │
  ▼
Stage B
  │
  ▼
Stage C
  │
  ▼
Output
```

Svaki stage može imati:

* sopstvene goroutine-e;
* sopstveni channel;
* sopstveni concurrency limit;
* sopstveni cancellation behavior.

Na primer:

```text
Read
 │
 ▼
Parse
 │
 ▼
Validate
 │
 ▼
Transform
 │
 ▼
Persist
```

Prednost ovog modela je composability.

Problem nastaje kada jedan stage postane sporiji od ostalih.

Tada se ponovo pojavljuje backpressure.

---

# Error Propagation

Concurrent sistem mora imati definisan model za propagaciju grešaka.

Naivni model:

```text
Worker A ── error
Worker B ── success
Worker C ── success
```

postavlja pitanja:

* Da li treba zaustaviti B i C?
* Ko prijavljuje error?
* Ko čeka rezultate?
* Da li se radi retry?
* Da li se otkazuje kompletan operation?
* Kako se vrši cleanup?

Zato concurrent architecture mora definisati:

```text
Error
  │
  ▼
Propagation
  │
  ▼
Cancellation
  │
  ▼
Cleanup
  │
  ▼
Result
```

Ovo postaje posebno važno kod server-side Go aplikacija.

---

# Timeout kao Boundary

Timeout treba posmatrati kao **execution boundary**, a ne samo kao `time.After` trik.

Na primer:

```text
Request
  │
  ▼
Service
  │
  ├── Database
  ├── External API
  └── Cache
```

Ako request ima timeout:

```text
Request deadline
       │
       ▼
Service deadline
       │
       ├── DB operation
       ├── API operation
       └── Cache operation
```

Child operations treba da poštuju parent lifecycle gde god je moguće.

Na taj način sprečava se da request završi dok njegove child goroutine-e nastavljaju da rade nepotreban posao.

---

# Concurrency Failure Modes

Tutorial će posebnu pažnju posvetiti tipičnim failure mode-ovima.

## Deadlock

```text
Goroutine A waits for B
Goroutine B waits for A
```

Niko ne može da napreduje.

---

## Data Race

```text
Goroutine A ── Write ──┐
                       ├── same memory
Goroutine B ── Read ───┘
```

Bez odgovarajuće synchronization garancije.

---

## Goroutine Leak

```text
Parent exits
    │
    └────── Child goroutine remains blocked
```

Proces može ostati živ ili nepotrebno trošiti resurse.

---

## Starvation

Jedna goroutine ili grupa goroutine-a ne dobija dovoljno prilika za napredak.

```text
Resource
   │
   ├── Worker A ── repeatedly wins
   ├── Worker B ── waits
   └── Worker C ── waits
```

---

## Livelock

Goroutine-i aktivno rade, ali sistem ne ostvaruje koristan progress.

```text
A reacts to B
B reacts to A
A reacts to B
B reacts to A
...
```

CPU može biti zauzet, a posao ostaje nezavršen.

---

## Unbounded Concurrency

```text
Input
 │
 ├── goroutine
 ├── goroutine
 ├── goroutine
 ├── ...
 └── millions
```

Broj concurrent aktivnosti nema kontrolisanu granicu.

---

## Unbounded Buffering

```text
Fast Producer
      │
      ▼
Growing Queue
      │
      ▼
Slow Consumer
```

Memory usage može nastaviti da raste.

---

# Testing Concurrent Code

Concurrent kod je teži za testiranje zato što execution order nije uvek determinističan.

Test može proći:

```text
Run 1 ✓
Run 2 ✓
Run 3 ✓
Run 4 ✓
Run 5 ✓
```

a ipak sadržati race koji se pojavi tek pod drugačijim scheduling-om:

```text
Run 6 ✗
```

Zato testiranje concurrency-ja mora uključivati:

* race detection;
* stress tests;
* repeated execution;
* controlled synchronization;
* timeout protection;
* deterministic test hooks gde je moguće;
* benchmarks;
* profiling.

---

# Race Detector

Go ecosystem pruža race detector koji predstavlja jedan od ključnih alata za analizu concurrent programa.

Tipičan način pokretanja:

```bash
go test -race ./...
```

ili:

```bash
go run -race .
```

Race detector nije zamena za razumevanje memory model-a.

On je alat koji pomaže da se određene klase problema pronađu.

Važno je razumeti i njegova ograničenja:

```text
No race detected
        ≠
No possible race exists
```

Testirana execution paths moraju zapravo biti izvršene da bi alat mogao da ih analizira.

---

# Performance Investigation

Kada concurrent program radi sporo, ne treba automatski dodavati više goroutine-a.

Prvi korak treba da bude:

```text
Measure
   ↓
Identify bottleneck
   ↓
Form hypothesis
   ↓
Change design
   ↓
Measure again
```

Tipični problemi mogu biti:

```text
Too much contention
Too many goroutines
Too much synchronization
Too little parallelism
Too much parallelism
Blocking I/O
Allocation pressure
Poor batching
Unbounded queues
Scheduler overhead
```

Concurrency optimization bez merenja lako dovodi do kompleksnijeg, ali ne i bržeg sistema.

---

# Design Trade-Offs

Ne postoji universalno najbolji concurrency primitive.

Izbor treba posmatrati kroz trade-off-e.

| Problem                   | Mogući pristup                             |
| ------------------------- | ------------------------------------------ |
| Passing ownership         | Channel                                    |
| Event stream              | Channel                                    |
| Shared mutable state      | Mutex / RWMutex                            |
| Simple counter            | Atomic                                     |
| One-time initialization   | `sync.Once`                                |
| Waiting for workers       | `sync.WaitGroup`                           |
| Multiple channel events   | `select`                                   |
| Bounded processing        | Worker pool                                |
| Multi-stage processing    | Pipeline                                   |
| Parallel processing       | Fan-out                                    |
| Aggregation               | Fan-in                                     |
| Operation cancellation    | Context / channel signaling                |
| Time limit                | Context / timeout                          |
| High-contention primitive | Carefully chosen synchronization / atomics |

Tabela predstavlja početnu heuristiku, a ne strogo pravilo.

---

# Senior-Level Review Questions

Kod code review-a concurrent Go programa treba postavljati pitanja poput:

### Ownership

* Ko poseduje state?
* Ko može da ga menja?
* Da li ownership može biti pojednostavljen?

### Lifecycle

* Ko kreira goroutine?
* Ko je gasi?
* Šta se dešava pri error-u?
* Šta se dešava pri cancellation-u?

### Channels

* Ko kreira channel?
* Ko šalje?
* Ko prima?
* Ko ga zatvara?
* Može li send da blokira zauvek?

### Synchronization

* Zašto je potreban lock?
* Koliko dugo se drži?
* Da li postoji contention?
* Može li critical section biti smanjen?

### Errors

* Kako se propagira prvi error?
* Da li se ostale goroutine-e otkazuju?
* Da li postoji cleanup?

### Performance

* Da li je concurrency bounded?
* Da li postoji backpressure?
* Da li je worker pool opravdan?
* Gde je bottleneck?

### Correctness

* Postoji li data race?
* Postoji li deadlock?
* Postoji li starvation?
* Postoji li goroutine leak?

Ovo su pitanja koja razlikuju površno poznavanje concurrency-ja od ozbiljnog engineering pristupa.

---

# Curriculum Completion Criteria

Smatra se da je čitalac savladao osnovni concurrency nivo kada može samostalno da:

* kreira goroutine;
* koristi channels;
* razume blocking;
* koristi `select`;
* implementira worker pool;
* implementira pipeline;
* koristi fan-out/fan-in;
* koristi mutex;
* implementira cancellation;
* prepozna osnovne concurrency bug-ove.

Za napredni nivo potrebno je dodatno razumeti:

* scheduler;
* `GOMAXPROCS`;
* parallelism;
* memory model;
* happens-before;
* data races;
* atomics;
* lock-free pristupe;
* contention;
* performance profiling;
* runtime internals.

Za senior/expert nivo očekuje se sposobnost da se ne samo implementira već i **analizira concurrency architecture**.

---

# Završna mapa

Na kraju kompletnog curriculum-a čitalac treba da može da poveže sledeće koncepte:

```text
                    APPLICATION
                         │
                         ▼
                CONCURRENCY DESIGN
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      Goroutines      Channels      Shared State
          │              │              │
          │              ▼              ▼
          │           Select          Mutex
          │              │            RWMutex
          │              │            Atomic
          └──────────────┼──────────────┘
                         ▼
                  Coordination
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      Worker Pools    Pipelines     Fan-Out/In
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                   Lifecycle
                         │
                ┌────────┼────────┐
                ▼        ▼        ▼
             Timeout Cancellation Shutdown
                │        │        │
                └────────┼────────┘
                         ▼
                     Runtime
                         │
                ┌────────┴────────┐
                ▼                 ▼
             Scheduler        Parallelism
                │                 │
                └────────┬────────┘
                         ▼
                    Memory Model
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
       Happens-Before   Races      Atomics
             │                       │
             └───────────┬───────────┘
                         ▼
                  Lock-Free Design
                         │
                         ▼
                  Runtime Internals
                         │
                         ▼
              Performance Engineering
                         │
                         ▼
                 Production Systems
```

Ova mapa predstavlja krajnji mentalni model kojem tutorial vodi.

Ne uči se samo kako se koriste Go concurrency primitives.

Uči se kako se **razmišlja o concurrent sistemu**.

---

# Go Concurrency

## Curriculum Navigation

Ovaj README predstavlja početnu tačku za kompletan `Go Concurrency` curriculum.

Prethodne sekcije su definisale:

* ciljeve tutoriala;
* mentalne modele;
* concurrency terminology;
* progression između modula;
* communication i synchronization modele;
* lifecycle management;
* runtime i memory model perspektivu;
* production i performance perspektivu.

Ova završna sekcija daje praktičnu navigaciju kroz kompletan curriculum.

---

# Complete Curriculum

```text id="q8k8nz"
golang/
└── concurrency/
    │
    ├── README.md
    │
    └── docs/
        │
        ├── module-1/
        │   ├── README.md
        │   ├── 01-goroutines.md
        │   ├── 02-channels.md
        │   ├── 03-unbuffered-channels.md
        │   ├── 04-buffered-channels.md
        │   ├── 05-channel-directions.md
        │   ├── 06-range-over-channels.md
        │   ├── 07-close-channel.md
        │   └── 08-module-1-summary-and-exercises.md
        │
        ├── module-2/
        │   ├── README.md
        │   ├── 01-select.md
        │   ├── 02-select-default.md
        │   ├── 03-waitgroup.md
        │   ├── 04-worker-pools.md
        │   ├── 05-pipelines.md
        │   ├── 06-fan-out.md
        │   ├── 07-fan-in.md
        │   └── 08-module-2-summary-and-exercises.md
        │
        ├── module-3/
        │   ├── README.md
        │   ├── 01-mutex.md
        │   ├── 02-rwmutex.md
        │   ├── 03-once.md
        │   ├── 04-timeouts.md
        │   ├── 05-cancellation.md
        │   ├── 06-go-scheduler.md
        │   ├── 07-gomaxprocs.md
        │   ├── 08-parallelism-vs-concurrency.md
        │   └── 09-module-3-summary-and-exercises.md
        │
        └── module-4/
            ├── README.md
            ├── 01-sync-atomic.md
            ├── 02-memory-model.md
            ├── 03-happens-before.md
            ├── 04-data-races.md
            ├── 05-lock-free-programming.md
            ├── 06-concurrency-patterns.md
            ├── 07-concurrency-anti-patterns.md
            ├── 08-goroutine-leaks.md
            ├── 09-concurrency-testing.md
            ├── 10-concurrency-performance.md
            ├── 11-concurrency-profiling.md
            ├── 12-advanced-concurrency-architecture.md
            ├── 13-mastering-go-concurrency.md
            │
            └── extra/
                ├── 01-go-memory-model.md
                ├── 02-happens-before.md
                ├── 03-data-races-deep-dive.md
                ├── 04-lock-free-programming.md
                ├── 05-channels-internals.md
                ├── 06-goroutine-internals.md
                ├── 07-go-scheduler-internals.md
                ├── 08-sync-package-internals.md
                ├── 09-atomic-operations-internals.md
                ├── 10-concurrency-performance-tuning.md
                ├── 11-concurrency-profiling.md
                ├── 12-concurrency-testing-strategies.md
                └── 13-advanced-concurrency-case-studies.md
```

---

# Module 1 — Fundamentals

**Path:**

```text id="0a0m0w"
golang/concurrency/docs/module-1/
```

Module 1 je početna tačka za concurrency.

### Topics

|  # | Topic               | Core concept                        |
| -: | ------------------- | ----------------------------------- |
|  1 | Goroutines          | Concurrent execution                |
|  2 | Channels            | Communication                       |
|  3 | Unbuffered Channels | Synchronous communication           |
|  4 | Buffered Channels   | Asynchronous buffering              |
|  5 | Channel Directions  | API-level communication constraints |
|  6 | Range Over Channels | Stream consumption                  |
|  7 | Close Channel       | Stream lifecycle                    |
|  8 | Summary & Exercises | Consolidation                       |

### Learning outcome

Nakon Module 1 čitalac treba da razume osnovni model:

```text id="n8gq6w"
Goroutine
    │
    ▼
Channel
    │
    ▼
Another Goroutine
```

i da bude sposoban da napiše jednostavan concurrent program bez oslanjanja na nejasno ili implicitno ponašanje.

**Prelazak na Module 2:** tek nakon što su blocking semantics, channel ownership i lifecycle dovoljno jasni.

---

# Module 2 — Coordination & Patterns

**Path:**

```text id="5c3x4v"
golang/concurrency/docs/module-2/
```

Module 2 proširuje prethodni model.

### Topics

|  # | Topic                | Core concept            |
| -: | -------------------- | ----------------------- |
|  1 | `select`             | Multiplexing            |
|  2 | `select` + `default` | Non-blocking operations |
|  3 | `WaitGroup`          | Goroutine coordination  |
|  4 | Worker Pools         | Bounded concurrency     |
|  5 | Pipelines            | Multi-stage processing  |
|  6 | Fan-Out              | Parallel distribution   |
|  7 | Fan-In               | Result aggregation      |
|  8 | Summary & Exercises  | Consolidation           |

### Learning outcome

Čitalac treba da bude sposoban da transformiše:

```text id="kz7rpx"
Single Goroutine
```

u kontrolisanu concurrent architecture:

```text id="3sj7vf"
                  ┌── Worker 1 ──┐
                  │              │
Input ──► Queue ──┼── Worker 2 ──┼──► Results
                  │              │
                  └── Worker 3 ──┘
```

sa jasnim:

* concurrency limitom;
* lifecycle-om;
* communication path-om;
* completion model-om.

---

# Module 3 — Synchronization & Runtime

**Path:**

```text id="7x84k5"
golang/concurrency/docs/module-3/
```

Module 3 uvodi shared-state synchronization i runtime perspective.

### Topics

|  # | Topic                      | Core concept               |
| -: | -------------------------- | -------------------------- |
|  1 | `Mutex`                    | Mutual exclusion           |
|  2 | `RWMutex`                  | Read/write synchronization |
|  3 | `Once`                     | One-time initialization    |
|  4 | Timeouts                   | Temporal boundaries        |
|  5 | Cancellation               | Lifecycle control          |
|  6 | Go Scheduler               | Goroutine scheduling       |
|  7 | `GOMAXPROCS`               | Runtime parallel execution |
|  8 | Parallelism vs Concurrency | Execution model            |
|  9 | Summary & Exercises        | Consolidation              |

### Learning outcome

Nakon Module 3 čitalac treba da razume oba glavna concurrency modela:

```text id="3kkw5p"
Message Passing
       │
       ▼
    Channels
```

i:

```text id="e1w92d"
Shared State
       │
       ▼
Synchronization
       │
   ┌───┴────┐
   ▼        ▼
 Mutex    Atomic
```

kao i osnovni odnos između:

```text id="p4j8qg"
Goroutines
     │
     ▼
Scheduler
     │
     ▼
Processors / Threads
     │
     ▼
CPU
```

---

# Module 4 — Advanced Concurrency

**Path:**

```text id="6u1e2k"
golang/concurrency/docs/module-4/
```

Module 4 predstavlja završni core nivo.

On objedinjuje prethodne koncepte i uvodi teme koje zahtevaju mnogo dublje razumevanje concurrency-ja.

### Core Topics

|  # | Topic                             | Focus                        |
| -: | --------------------------------- | ---------------------------- |
|  1 | `sync/atomic`                     | Atomic synchronization       |
|  2 | Memory Model                      | Memory visibility & ordering |
|  3 | Happens-Before                    | Synchronization ordering     |
|  4 | Data Races                        | Race analysis                |
|  5 | Lock-Free Programming             | Non-blocking algorithms      |
|  6 | Concurrency Patterns              | Advanced composition         |
|  7 | Concurrency Anti-Patterns         | Failure-prone designs        |
|  8 | Goroutine Leaks                   | Lifecycle failures           |
|  9 | Concurrency Testing               | Correctness verification     |
| 10 | Concurrency Performance           | Throughput & latency         |
| 11 | Concurrency Profiling             | Runtime investigation        |
| 12 | Advanced Concurrency Architecture | System design                |
| 13 | Mastering Go Concurrency          | Final synthesis              |

---

# Module 4 — Extra Material

Pored core lekcija, Module 4 sadrži dodatni expert-oriented sloj:

```text id="s5m2z3"
module-4/
└── extra/
```

Ovaj deo treba koristiti kao **deep-dive/reference layer**.

### Extra Topics

|  # | Topic                             |
| -: | --------------------------------- |
|  1 | Go Memory Model                   |
|  2 | Happens-Before                    |
|  3 | Data Races Deep Dive              |
|  4 | Lock-Free Programming             |
|  5 | Channels Internals                |
|  6 | Goroutine Internals               |
|  7 | Go Scheduler Internals            |
|  8 | `sync` Package Internals          |
|  9 | Atomic Operations Internals       |
| 10 | Concurrency Performance Tuning    |
| 11 | Concurrency Profiling             |
| 12 | Concurrency Testing Strategies    |
| 13 | Advanced Concurrency Case Studies |

Extra materijal nije zamišljen kao obavezni prvi prolaz kroz curriculum.

Njegova uloga je da omogući čitaocu da ode dublje kada osnovni i advanced core concepts više nisu dovoljni.

---

# Recommended Learning Sequence

Preporučeni redosled učenja:

```text id="q8j24s"
                    START
                      │
                      ▼
              Module 1
              Fundamentals
                      │
                      ▼
              Module 1 Exercises
                      │
                      ▼
              Module 2
          Coordination & Patterns
                      │
                      ▼
              Module 2 Exercises
                      │
                      ▼
              Module 3
      Synchronization & Runtime
                      │
                      ▼
              Module 3 Exercises
                      │
                      ▼
              Module 4
        Advanced Concurrency
                      │
                      ▼
          Module 4 Core Exercises
                      │
                      ▼
             Extra Deep Dives
                      │
                      ▼
                    MASTER
```

Nije potrebno memorisati sve API-je.

Potrebno je razviti sposobnost da se izabere odgovarajući model za konkretan problem.

---

# Recommended Study Method

Za svaku temu preporučuje se sledeći proces.

## Step 1 — Understand

Prvo razumeti koncept bez optimizacije i bez preuranjenog fokusiranja na internals.

Na primer:

```text id="h74j4s"
What is a channel?
```

zatim:

```text id="xj90jz"
How does sending block?
```

pa:

```text id="zv9a5n"
What happens when the channel is closed?
```

---

## Step 2 — Implement

Napisati mali, izolovani primer.

Primer treba da bude dovoljno mali da se execution flow može mentalno pratiti.

---

## Step 3 — Break

Namerno napraviti grešku:

```text id="0f8t2u"
Deadlock
Race
Leak
Blocked send
Blocked receive
Incorrect close
Unbounded concurrency
```

---

## Step 4 — Diagnose

Koristiti odgovarajuće alate:

```text id="x5brm5"
go test -race
go test
go test -bench
pprof
trace
Delve
runtime diagnostics
```

Cilj nije samo popraviti kod.

Cilj je razumeti **zašto je kod bio pogrešan**.

---

## Step 5 — Measure

Ako je problem performance prirode:

```text id="mxxmnp"
Baseline
   ↓
Change
   ↓
Benchmark
   ↓
Profile
   ↓
Compare
```

Bez baseline-a ne postoji pouzdana procena optimizacije.

---

## Step 6 — Review

Na kraju treba postaviti pitanja:

* Da li je dizajn jednostavan?
* Da li je concurrency zaista potreban?
* Da li postoji bolji ownership model?
* Da li postoji bounded concurrency?
* Da li postoji backpressure?
* Da li lifecycle ima jasan završetak?
* Da li postoji cancellation?
* Da li postoji synchronization bottleneck?
* Da li je implementacija dovoljno laka za održavanje?

---

# Recommended Tooling

Za praktičan rad sa concurrency kodom posebno su važni Go alati.

## Build & Run

```bash id="n3e3b7"
go run .
```

```bash id="9q4g3n"
go build ./...
```

---

## Testing

```bash id="t0xj9m"
go test ./...
```

---

## Race Detection

```bash id="7b0uw1"
go test -race ./...
```

Race detector treba koristiti kao standardni deo razvoja concurrent koda, a ne samo kao poslednji korak kada se pojavi sumnja na race.

---

## Benchmarking

```bash id="d4s0tt"
go test -bench=. ./...
```

Benchmark treba koristiti za poređenje konkretnih implementacija i merenje efekta promena.

---

## Profiling

Advanced performance investigation može koristiti Go profiling tooling i `pprof`.

Tipičan proces:

```text id="o4xx7e"
Workload
   ↓
Profile
   ↓
Identify hotspot
   ↓
Understand contention
   ↓
Change implementation
   ↓
Profile again
```

---

# Concurrency Design Checklist

Pre nego što se concurrent component smatra završenim, korisno je proći kroz sledeću checklistu.

## Ownership

* [ ] Svaki mutable state ima jasno definisan ownership.
* [ ] Jasno je ko može da menja state.
* [ ] Shared state je minimalan gde god je moguće.

## Goroutines

* [ ] Svaka goroutine ima jasno definisan lifecycle.
* [ ] Jasno je ko je kreira.
* [ ] Jasno je kako se završava.
* [ ] Nema poznatih goroutine leak-ova.

## Channels

* [ ] Jasno je ko kreira channel.
* [ ] Jasno je ko šalje.
* [ ] Jasno je ko prima.
* [ ] Jasno je ko zatvara channel.
* [ ] Blocking behavior je nameran.
* [ ] Buffer size je opravdan.

## Synchronization

* [ ] Svaki lock ima jasno definisanu svrhu.
* [ ] Critical section je dovoljno mali.
* [ ] Lock ordering je poznat.
* [ ] Nema nepotrebnog contention-a.
* [ ] Atomic operations se koriste samo kada su semantički opravdane.

## Lifecycle

* [ ] Cancellation postoji gde je potrebna.
* [ ] Timeout postoji gde je potreban.
* [ ] Shutdown je definisan.
* [ ] Resources se pravilno oslobađaju.
* [ ] Child goroutine-i ne ostaju aktivni nakon završetka parent operation-a.

## Correctness

* [ ] Kod je testiran sa race detector-om.
* [ ] Deadlock scenariji su razmotreni.
* [ ] Error propagation je definisan.
* [ ] Failure scenarios su testirani.

## Performance

* [ ] Concurrency je bounded gde je potrebno.
* [ ] Backpressure postoji gde je potreban.
* [ ] Throughput je izmeren.
* [ ] Latency je izmerena.
* [ ] Contention je analiziran.
* [ ] Optimizacije su zasnovane na merenju.

---

# Production Readiness

Concurrent kod treba smatrati production-ready tek kada su poznati njegovi:

```text id="x6t6kp"
Correctness properties
        +
Failure modes
        +
Lifecycle behavior
        +
Resource limits
        +
Performance characteristics
        +
Observability requirements
```

Posebno treba izbegavati situaciju:

```text id="h9k4x7"
"It works on my machine."
```

Concurrent program može izgledati potpuno korektno tokom normalnog execution-a, a da se problem pojavi tek kada:

* workload poraste;
* scheduling bude drugačiji;
* latency downstream sistema poraste;
* jedan worker kasni;
* jedan channel ostane bez receiver-a;
* cancellation stigne u pogrešnom trenutku;
* više CPU jezgara izvršava kod istovremeno;
* sistem bude pod contention-om.

Zato je concurrency engineering disciplina u kojoj je **failure analysis** jednako važan kao i happy-path implementation.

---

# Final Learning Objectives

Nakon kompletnog curriculum-a čitalac treba da bude sposoban da:

### Understand

* objasni Go concurrency model;
* objasni goroutine lifecycle;
* objasni channel semantics;
* objasni synchronization;
* objasni scheduler;
* objasni memory model;
* objasni happens-before.

### Implement

* worker pool;
* pipeline;
* fan-out/fan-in;
* cancellation;
* timeout;
* synchronization;
* atomic operations;
* concurrent data processing.

### Analyze

* deadlocks;
* data races;
* goroutine leaks;
* starvation;
* livelock;
* contention;
* scheduler behavior;
* performance bottlenecks.

### Design

* bounded concurrency;
* backpressure;
* graceful shutdown;
* concurrent services;
* high-throughput pipelines;
* synchronization boundaries;
* ownership models.

### Optimize

* measure concurrency;
* benchmark;
* profile;
* identify contention;
* reduce unnecessary synchronization;
* izabrati odgovarajući concurrency primitive.

---

# The Core Principle

Kompletan tutorial može se svesti na jednu centralnu ideju:

```text id="r6t5h7"
Concurrent Programming
        │
        ▼
Multiple Activities
        │
        ▼
Coordination
        │
        ▼
Synchronization
        │
        ▼
Correctness
        │
        ▼
Performance
        │
        ▼
Reliable Systems
```

Goroutines daju mogućnost concurrent execution-a.

Channels omogućavaju communication.

Synchronization primitives omogućavaju kontrolu shared state-a.

Scheduler upravlja execution-om.

Memory model definiše semantičke granice.

Testing i race detection pomažu u proveri correctness-a.

Profiling i benchmarking omogućavaju performance analysis.

Architecture povezuje sve prethodno u production sistem.

Zbog toga je pravi cilj ovog curriculum-a mnogo širi od učenja nekoliko concurrency API-ja:

> **Cilj je razviti sposobnost projektovanja, implementacije, analize i optimizacije pouzdanih concurrent sistema u Go-u.**

---

# Start Here

Ako prvi put prolaziš kroz Go concurrency, počni od:

```text id="h7f9yq"
golang/concurrency/docs/module-1/README.md
```

a zatim nastavi redom:

```text id="1y6d0x"
Module 1
   ↓
Module 2
   ↓
Module 3
   ↓
Module 4
   ↓
Module 4 / extra
```

Ako već dobro poznaješ osnovne goroutine i channel patterns, Module 1 može poslužiti kao review, nakon čega možeš nastaviti ka Module 2 i Module 3.

Za duboko razumevanje Go concurrency-ja preporučuje se da se Module 4 ne posmatra samo kao završni "advanced" materijal, već kao mesto gde se svi prethodni koncepti povezuju u jedan koherentan model.

---

## Navigation

| Module                                | Focus                                                              | Level                   |
| ------------------------------------- | ------------------------------------------------------------------ | ----------------------- |
| [Module 1](./docs/module-1/README.md) | Goroutines, Channels, Channel Lifecycle                            | Beginner → Intermediate |
| [Module 2](./docs/module-2/README.md) | `select`, Coordination, Worker Pools, Pipelines, Fan-Out/Fan-In    | Intermediate            |
| [Module 3](./docs/module-3/README.md) | Synchronization, Cancellation, Scheduler, Parallelism              | Intermediate → Advanced |
| [Module 4](./docs/module-4/README.md) | Atomics, Memory Model, Races, Lock-Free, Performance, Architecture | Advanced → Expert       |

---

## Final Mental Model

```text id="l0x8hx"
                    GO CONCURRENCY
                         │
                         ▼
                  ┌─────────────┐
                  │ Goroutines  │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │  Channels   │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │   Select    │
                  └──────┬──────┘
                         │
                         ▼
               ┌───────────────────┐
               │   Coordination    │
               └─────────┬─────────┘
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
      Message Passing           Shared State
             │                       │
             ▼                       ▼
         Channels               Mutex/RWMutex
             │                       │
             └───────────┬───────────┘
                         ▼
                    Synchronization
                         │
                         ▼
                  Concurrency Patterns
                         │
                         ▼
                  Lifecycle Management
                         │
                         ▼
                    Go Scheduler
                         │
                         ▼
                Parallel Execution
                         │
                         ▼
                   Memory Model
                         │
                         ▼
                   Happens-Before
                         │
                         ▼
                     Atomics
                         │
                         ▼
                 Lock-Free Algorithms
                         │
                         ▼
                  Runtime Internals
                         │
                         ▼
               Performance Engineering
                         │
                         ▼
                  Production Systems
```

Ovo je mentalni model koji treba da ostane nakon završetka celog curriculum-a.

---

# Conclusion

Go concurrency je kombinacija:

* execution model-a;
* communication model-a;
* synchronization-a;
* memory semantics;
* lifecycle management-a;
* runtime behavior-a;
* performance engineering-a;
* system architecture-a.

Jednostavan concurrent program može početi sa:

```go id="v7k3cw"
go worker()
```

ali production-level concurrency zahteva mnogo više od toga.

Potrebno je razumeti:

```text id="y2z3wq"
Who runs?
Who owns?
Who communicates?
Who waits?
Who cancels?
Who closes?
Who synchronizes?
Who observes?
Who exits?
What happens under failure?
What happens under load?
```

Kada se ova pitanja mogu sistematski odgovoriti, concurrency postaje predvidiviji, testabilniji i lakši za projektovanje.

**Go Concurrency curriculum počinje sa goroutine-ama, ali se završava sposobnošću da se projektuju pouzdani concurrent sistemi.**

---



