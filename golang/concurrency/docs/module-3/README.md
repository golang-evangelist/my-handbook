# Module 3 — Synchronization, Cancellation & Go Runtime

## 1. Module Overview

Module 3 predstavlja sledeći veliki korak u razumevanju concurrency-ja u Go-u.

U prethodnim modulima fokus je bio prvenstveno na:

```text
Goroutines
    ↓
Channels
    ↓
Communication
    ↓
Coordination
    ↓
Concurrency Patterns
```

Module 1 je postavio temelje goroutines i channel komunikacije, dok je Module 2 pokazao kako se ti building blocks kombinuju u:

* `select`-based coordination;
* `WaitGroup` coordination;
* worker pools;
* pipelines;
* fan-out;
* fan-in;
* bounded concurrency;
* backpressure;
* graceful shutdown patterns.

Module 3 proširuje model concurrency-ja u dva pravca:

```text
                    MODULE 3
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
   Synchronization              Runtime Control
          │                           │
          ▼                           ▼
       Mutex                      Timeouts
      RWMutex                   Cancellation
        Once                     Go Scheduler
          │                       GOMAXPROCS
          │               Parallelism vs Concurrency
          └─────────────┬─────────────┘
                        ▼
                Production Concurrency
```

Drugim rečima, Module 3 prelazi sa pitanja:

> **Kako goroutines komuniciraju?**

na šira pitanja:

> **Kako goroutines bezbedno pristupaju shared state-u, kako kontrolišemo njihov lifecycle i kako Go runtime izvršava concurrent program?**

---

# 2. Why Module 3 Matters

Channel-based communication nije jedini način rešavanja concurrency problema.

Postoje sistemi u kojima više goroutines mora pristupati istom shared state-u:

```text
                 Shared State
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Goroutine A Goroutine B Goroutine C
```

Na primer:

```go
type Counter struct {
    value int
}
```

Ako više goroutines istovremeno izvršava:

```go
counter.value++
```

problem nije u komunikaciji između goroutines.

Problem je u **shared memory access-u**.

Tada dolazimo do synchronization primitives kao što su:

```text
sync.Mutex
sync.RWMutex
sync.Once
```

Međutim, synchronization nije jedini problem.

Concurrent system mora imati i kontrolisan lifecycle:

```text
start
  ↓
running
  ↓
timeout / cancellation / completion
  ↓
shutdown
```

Zato Module 3 uvodi i:

```text
timeouts
cancellation
```

Na kraju, kada concurrency sistem postane dovoljno složen, potrebno je razumeti šta se dešava ispod application-level primitives:

```text
goroutine
    ↓
scheduler
    ↓
P
    ↓
M
    ↓
OS thread
    ↓
CPU
```

Zbog toga Module 3 završava sa:

```text
Go scheduler
GOMAXPROCS
parallelism vs concurrency
```

---

# 3. Learning Objectives

Po završetku Module 3 čitalac treba da bude sposoban da:

* koristi `sync.Mutex` za zaštitu shared state-a;
* razume razliku između `Mutex` i `RWMutex`;
* koristi `sync.Once` za one-time initialization;
* razume blocking behavior synchronization primitives;
* kombinuje locks i channels;
* implementira timeout mehanizme;
* dizajnira cancellation-aware goroutines;
* spreči goroutine leaks tokom cancellation-a;
* razume osnovni Go scheduler model;
* razume odnos između goroutines, Ps, Ms i OS threads;
* objasni ulogu `GOMAXPROCS`;
* razlikuje concurrency od parallelism;
* razume kada povećanje broja goroutines povećava throughput, a kada samo povećava contention;
* analizira concurrency architecture sa runtime perspective.

---

# 4. Prerequisites

Pre početka ovog modula potrebno je dobro poznavanje:

### Go fundamentals

* functions;
* methods;
* structs;
* interfaces;
* pointers;
* slices;
* maps;
* errors.

### Concurrency fundamentals

* goroutines;
* channels;
* buffered/unbuffered channels;
* directional channels;
* channel closing;
* `range` over channels;
* `select`.

### Coordination patterns

* `WaitGroup`;
* worker pools;
* pipelines;
* fan-out;
* fan-in;
* graceful shutdown basics;
* bounded concurrency;
* backpressure.

Bez ovih temelja teško je razumeti zašto se synchronization primitives koriste i kada je bolje koristiti channels.

---

# 5. Module Structure

Module 3 se sastoji od sledećih tema:

```text
module-3/
│
├── README.md
│
├── 01-mutex.md
├── 02-rwmutex.md
├── 03-once.md
├── 04-timeouts.md
├── 05-cancellation.md
├── 06-go-scheduler.md
├── 07-gomaxprocs.md
├── 08-parallelism-vs-concurrency.md
│
└── 09-module-3-summary-and-exercises.md
```

Redosled nije slučajan.

Teme su organizovane tako da se mentalni model postepeno proširuje:

```text
Mutex
  ↓
RWMutex
  ↓
Once
  ↓
Timeouts
  ↓
Cancellation
  ↓
Go Scheduler
  ↓
GOMAXPROCS
  ↓
Parallelism vs Concurrency
```

---

# 6. Topic 01 — Mutex

Fajl:

```text
01-mutex.md
```

`sync.Mutex` predstavlja osnovni mutual-exclusion primitive u Go-u.

Njegova osnovna svrha je zaštita critical section-a:

```text
          Shared State
               │
               ▼
          ┌─────────┐
          │  Mutex  │
          └────┬────┘
               │
          ┌────▼────┐
          │ Critical│
          │ Section │
          └─────────┘
```

Čitalac treba da razume:

* šta je mutual exclusion;
* šta je critical section;
* kako `Lock` i `Unlock` funkcionišu;
* šta znači da je mutex locked/unlocked;
* kako mutex sprečava concurrent access;
* šta se dešava kada više goroutines čeka isti mutex;
* zašto `Unlock` treba koristiti sa `defer`;
* kako nastaje deadlock zbog pogrešnog lock ordering-a;
* zašto prevelika critical section degradira concurrency.

Posebna pažnja treba da bude posvećena razlici između:

```text
protecting data
```

i:

```text
protecting code
```

Mutex nije sam po sebi "zaštita funkcije".

On štiti **invariant shared state-a** ako svi pristupi tom state-u poštuju isti synchronization protocol.

---

# 7. Mutex Mental Model

Osnovni model:

```text
Goroutine A
     │
     ▼
 Lock()
     │
     ▼
 Critical Section
     │
     ▼
 Unlock()
```

Ako u međuvremenu Goroutine B pokuša:

```text
Lock()
```

dok je mutex već zaključan:

```text
Goroutine B
     │
     ▼
   Lock()
     │
     X
   BLOCK
```

Nakon što A izvrši:

```text
Unlock()
```

B može nastaviti.

Ovaj model uvodi fundamentalni koncept:

> **Concurrency control through mutual exclusion.**

---

# 8. Mutex and Shared State

Tipičan pattern:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()

    c.value++
}
```

Bitno je razumeti da synchronization primitive i protected state čine jednu logičku celinu.

U praksi je često dobro držati mutex blizu podataka koje štiti:

```text
┌─────────────────────┐
│ Counter             │
│                     │
│ mu                  │
│ value               │
└─────────────────────┘
```

umesto da synchronization bude rasut po application code-u.

---

# 9. Critical Section Design

Jedna od važnijih tema nije samo:

> "Kako koristiti Mutex?"

već:

> **Šta tačno treba da bude unutar critical section-a?**

Preširok critical section:

```go
mu.Lock()

expensiveOperation()
networkCall()
databaseCall()
fileOperation()

mu.Unlock()
```

može praktično serijalizovati sistem.

Preuzak critical section:

```go
value := sharedValue
mu.Unlock()

modify(value)

mu.Lock()
sharedValue = value
mu.Unlock()
```

može dovesti do race condition-a ako je shared invariant narušen.

Zato critical section treba projektovati prema **invariantima podataka**, a ne prema proizvoljnim granicama funkcije.

---

# 10. Mutex and `defer`

Preporučeni osnovni pattern:

```go
mu.Lock()
defer mu.Unlock()
```

Prednost je lifecycle safety.

Ako se unutar critical section-a dogodi:

```text
return
panic
error path
```

`defer` osigurava da se `Unlock` izvrši.

Međutim, čitalac treba da razume da `defer` nije zamena za pravilno projektovanje critical section-a.

---

# 11. Common Mutex Mistakes

Tema mora obuhvatiti tipične greške:

```text
✓ forgetting Unlock
✓ double Unlock
✓ copying a Mutex after first use
✓ locking the wrong Mutex
✓ protecting only some accesses
✓ holding lock too long
✓ nested locks
✓ inconsistent lock ordering
✓ calling blocking operations while locked
✓ exposing protected state without synchronization
```

Posebno treba razumeti da mutex može biti tehnički ispravno korišćen, a da architecture i dalje bude loša.

---

# 12. Mutex vs Channels

Module 3 treba da uspostavi važnu granicu:

```text
Channels
    ↓
communication / ownership transfer

Mutex
    ↓
shared state synchronization
```

Ovo nije apsolutno pravilo.

Channels mogu zaštititi state kroz ownership model, dok mutex može biti deo kompleksnog communication architecture-a.

Ali kao početni mentalni model:

```text
"Who owns the data?"
        │
        ├── ownership transfer → channel
        │
        └── shared ownership → synchronization
```

predstavlja korisnu heuristiku.

---

# 13. Topic 02 — RWMutex

Fajl:

```text
02-rwmutex.md
```

`sync.RWMutex` proširuje mutual exclusion model sa dve vrste access-a:

```text
Read lock
Write lock
```

Mentalni model:

```text
                RWMutex
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
     Readers               Writer
        │                     │
   concurrent             exclusive
```

Više readers može istovremeno pristupati protected state-u, dok writer zahteva exclusive access.

---

# 14. Reader vs Writer Access

Osnovni API:

```go
RLock()
RUnlock()

Lock()
Unlock()
```

Conceptualno:

```text
RLock()
   ↓
multiple readers allowed
```

dok:

```text
Lock()
   ↓
no other reader/writer allowed
```

Čitalac treba da razume kada je ovaj model koristan, a kada običan `Mutex` predstavlja jednostavniji i bolji izbor.

---

# 15. RWMutex Trade-Offs

`RWMutex` ne znači automatski:

```text
RWMutex = faster
```

Performance zavisi od:

* read/write ratio;
* duration of critical section;
* contention;
* scheduler behavior;
* workload;
* goroutine count;
* frequency of lock acquisition.

Na primer, ako je workload:

```text
95% reads
5% writes
```

RWMutex može imati smisla.

Ali ako je workload:

```text
50% reads
50% writes
```

ili critical sections veoma kratke, dodatna kompleksnost možda ne daje korist.

Zato treba meriti, a ne pretpostavljati.

---

# 16. RWMutex Design Questions

Za svaki `RWMutex` usage potrebno je pitati:

```text
1. Is shared state actually read-heavy?
2. Are reads long enough to overlap meaningfully?
3. Are writes rare?
4. Is contention measurable?
5. Does RWMutex simplify or complicate the design?
```

Ovo sprečava cargo-cult korišćenje `RWMutex`.

---

# 17. Topic 03 — Once

Fajl:

```text
03-once.md
```

`sync.Once` rešava specifičan concurrency problem:

> **Kako garantovati da se neka inicijalizacija izvrši najviše jednom, čak i kada više goroutines pokušava da je pokrene?**

Model:

```text
Goroutine A ──┐
Goroutine B ──┼──► sync.Once ──► initialization
Goroutine C ──┘
```

Tipičan pattern:

```go
var once sync.Once

once.Do(func() {
    initialize()
})
```

---

# 18. Why `sync.Once` Exists

Naivna implementacija:

```go
if resource == nil {
    resource = initialize()
}
```

nije dovoljna ako više goroutines može istovremeno izvršiti ovaj kod.

Moguće je:

```text
Goroutine A → sees nil
Goroutine B → sees nil

A → initialize
B → initialize
```

`sync.Once` uvodi one-time execution semantics.

---

# 19. Once Semantics

Važno je razumeti da `sync.Once` nije isto što i:

```text
"execute if successful"
```

Već predstavlja određenu once-execution semantiku.

Čitalac treba pažljivo da prouči:

* kada se funkcija smatra izvršenom;
* šta se dešava ako funkcija panics;
* kako concurrent callers čekaju;
* zašto `sync.Once` nije general-purpose state machine;
* zašto se ne treba koristiti kao zamena za mutex.

---

# 20. Once and Initialization

Tipične primene:

```text
lazy initialization
one-time setup
global resource initialization
singleton-like initialization
configuration loading
expensive setup
```

Ali `sync.Once` treba koristiti kada zaista postoji invariant:

```text
"This initialization must happen only once."
```

Ako je potreban lifecycle:

```text
initialize
reset
reinitialize
shutdown
initialize again
```

`sync.Once` nije odgovarajući primitive.

---

# 21. Topic 04 — Timeouts

Fajl:

```text
04-timeouts.md
```

Timeout predstavlja vremensko ograničenje operation-a.

Mentalni model:

```text
Operation
   │
   ├──── success ────► result
   │
   └──── timeout ────► fallback / cancellation
```

U Go-u timeout se često implementira pomoću:

```go
select {
case result := <-resultCh:
    return result
case <-time.After(timeout):
    return errTimeout
}
```

Međutim, timeout nije samo convenience feature.

On predstavlja **lifecycle boundary**.

---

# 22. Why Timeouts Matter

Bez timeout-a concurrent operation može čekati neograničeno:

```text
request
   │
   ▼
goroutine
   │
   ▼
blocked operation
   │
   X
 forever
```

Sa timeout-om:

```text
request
   │
   ▼
operation
   │
   ├── completed
   │
   └── deadline exceeded
```

Ovo je naročito važno kod:

* network operations;
* RPC;
* database calls;
* channel operations;
* external services;
* distributed systems.

---

# 23. Timeout Is Not Automatically Cancellation

Ovo je jedna od ključnih razlika Module 3.

Ako caller prestane da čeka:

```text
caller
  │
  ├── timeout
  │
  ▼
returns
```

to ne znači automatski da underlying goroutine prestaje:

```text
caller ──► returns
             │
             X
         worker still running
```

Dakle:

```text
timeout
    ≠
cancellation
```

Timeout kontroliše koliko dugo caller čeka.

Cancellation kontroliše da li ongoing operation treba da prestane.

Ova razlika biće centralna u sledećoj temi.

---

# 24. Timeout Design

Prilikom dizajniranja timeout-a potrebno je definisati:

```text
What exactly is timed?
Who owns the timer?
What happens after timeout?
Does the underlying operation stop?
What happens to partial work?
What happens to the result?
```

Bez ovih odgovora timeout može samo sakriti lifecycle problem.

---

# 25. Topic 05 — Cancellation

Fajl:

```text
05-cancellation.md
```

Cancellation predstavlja mehanizam kojim se concurrent operation-u signalizira:

```text
"Stop what you are doing and exit as soon as it is safe to do so."
```

Osnovni model:

```text
                Cancellation Signal
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Worker A     Worker B     Worker C
          │            │            │
          ▼            ▼            ▼
        stop         stop         stop
```

Ovo je fundamentalno za production concurrency systems.

---

# 26. Cancellation Is Cooperative

Go cancellation nije preemptive application-level kill.

Caller ne "ubija" goroutine.

Umesto toga:

```text
cancel signal
     │
     ▼
worker observes signal
     │
     ▼
worker cleans up
     │
     ▼
worker returns
```

Zato cancellation mora biti deo dizajna same goroutine.

---

# 27. Cancellation With `select`

Tipičan model:

```go
for {
    select {
    case job := <-jobs:
        process(job)

    case <-done:
        return
    }
}
```

Worker sada ima dva moguća lifecycle događaja:

```text
job available
```

ili:

```text
shutdown requested
```

Ovaj pattern prirodno povezuje Module 2 `select` knowledge sa Module 3 cancellation architecture.

---

# 28. Cancellation Propagation

U realnom sistemu cancellation često mora propagirati kroz više layers:

```text
Request
   │
   ▼
Service
   │
   ▼
Worker Pool
   │
   ▼
Worker
   │
   ▼
External Operation
```

Ako se request otkaže, idealno:

```text
Request cancellation
       ↓
Service cancellation
       ↓
Worker cancellation
       ↓
Underlying operation cancellation
```

Ako samo gornji layer prestane da čeka, moguće je ostaviti goroutines koje nastavljaju rad.

---

# 29. Cancellation and Goroutine Leaks

Jedan od najvažnijih ciljeva cancellation-a je sprečavanje:

```text
goroutine leak
```

Problem:

```text
caller exits
    │
    ▼
worker remains blocked
    │
    ▼
worker never exits
```

Ako se ovo ponavlja za veliki broj requests:

```text
request 1 → leaked goroutine
request 2 → leaked goroutine
request 3 → leaked goroutine
...
```

sistem može vremenom potrošiti:

* memory;
* scheduler resources;
* file descriptors;
* connections;
* other resources.

Zato cancellation treba posmatrati kao deo resource management-a.

---

# 30. Timeout + Cancellation

Najvažniji kombinovani pattern:

```text
                 Operation
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Success             Timeout
                              │
                              ▼
                         Cancellation
                              │
                              ▼
                       Worker exits
```

Idealno je da timeout ne bude samo:

```text
"caller stopped waiting"
```

već:

```text
"operation should stop because its deadline has expired."
```

Ovaj princip predstavlja osnovu modernih Go services.

---

# 31. Module 3 — First Half Mental Model

Prvih pet tema mogu se povezati u jedan model:

```text
                 CONCURRENT SYSTEM
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Shared State      Lifecycle         Runtime
        │                │
        ▼                ▼
      Mutex          Timeout
        │                │
        ▼                ▼
    RWMutex       Cancellation
        │
        ▼
      Once
```

`Mutex`, `RWMutex` i `Once` rešavaju synchronization problems.

`Timeouts` i `Cancellation` rešavaju lifecycle problems.

Druga polovina modula zatim prelazi na runtime:

```text
Synchronization
       +
Lifecycle
       +
Runtime Execution
```

---

# 32. Synchronization vs Communication

Do ovog trenutka concurrency primitives mogu se klasifikovati približno ovako:

```text
┌──────────────────────────────────────────────┐
│                CONCURRENCY                   │
├──────────────────────────────────────────────┤
│                                              │
│ Communication                                │
│   ├── channels                               │
│   └── select                                 │
│                                              │
│ Coordination                                 │
│   └── WaitGroup                              │
│                                              │
│ Synchronization                              │
│   ├── Mutex                                  │
│   ├── RWMutex                                │
│   └── Once                                   │
│                                              │
│ Lifecycle                                    │
│   ├── timeouts                               │
│   └── cancellation                           │
│                                              │
│ Runtime                                      │
│   ├── scheduler                              │
│   ├── GOMAXPROCS                             │
│   └── parallelism                            │
│                                              │
└──────────────────────────────────────────────┘
```

Ovo predstavlja jedan od glavnih mentalnih modela celog concurrency tutoriala.

---

# 33. When to Use Which Primitive?

Kada se pojavi concurrency problem, ne treba automatski posegnuti za prvim primitive-om koji je poznat.

Postaviti pitanje:

```text
What problem am I solving?
```

### Communication

Ako treba preneti ownership/data:

```text
channel
```

### Coordination

Ako treba sačekati završetak grupe goroutines:

```text
WaitGroup
```

### Mutual exclusion

Ako više goroutines pristupa shared mutable state-u:

```text
Mutex
```

### Read-heavy shared state

Ako postoji opravdana potreba za concurrent reads:

```text
RWMutex
```

### One-time initialization

Ako operation mora biti izvršen jednom:

```text
Once
```

### Time boundary

Ako operation ne sme trajati neograničeno:

```text
timeout
```

### Lifecycle termination

Ako worker treba dobiti signal da prestane:

```text
cancellation
```

---

# 34. The Critical Design Question

Najvažnije pitanje Module 3 nije:

> "Koji API treba da pozovem?"

Već:

> **"Koji concurrency invariant pokušavam da održim?"**

Primer:

```text
Counter must never be concurrently mutated.
```

→ `Mutex`

Primer:

```text
Initialization must happen exactly once according to Once semantics.
```

→ `sync.Once`

Primer:

```text
Request should not wait longer than 2 seconds.
```

→ timeout/deadline

Primer:

```text
All request-scoped workers must stop when the request is cancelled.
```

→ cancellation propagation

Ovaj način razmišljanja treba da postane osnova za production-level Go concurrency design.

---

# 35. Transition to Runtime

Nakon synchronization i lifecycle primitives dolazimo do sledećeg pitanja:

```text
What actually executes these goroutines?
```

Application code vidi:

```go
go worker()
```

ali Go runtime mora da odluči:

```text
When should worker run?
On which execution resource?
How should runnable goroutines be distributed?
When should one goroutine yield?
How many can execute simultaneously?
```

Odgovor se nalazi u Go scheduler-u.

Zato druga polovina Module 3 prelazi sa:

```text
application-level concurrency
```

na:

```text
runtime-level concurrency
```

---

# 36. Module 3 Learning Progression

Do kraja ovog modula mentalni model treba da izgleda ovako:

```text
                Go Concurrency
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
    Application Model        Runtime Model
          │                       │
    ┌─────┼─────┐           ┌─────┼─────┐
    ▼     ▼     ▼           ▼     ▼     ▼
  Mutex  Once  Cancel    Scheduler P   M
    │     │     │                 │
    └─────┼─────┘                 ▼
          ▼                   OS Threads
       Lifecycle                 │
          │                      ▼
          ▼                     CPU
      Correctness
```

Ovo je trenutak kada Go concurrency prestaje da bude samo skup language/library primitives i počinje da se posmatra kao **runtime-backed execution model**.

---

# 37. Module 3 Completion Criteria

Po završetku celog modula, čitalac treba da bude sposoban da:

### Synchronization

* objasni mutual exclusion;
* koristi `Mutex`;
* koristi `RWMutex`;
* koristi `Once`;
* prepozna lock-related deadlocks;
* minimizuje critical sections;
* proceni synchronization overhead.

### Lifecycle

* implementira timeout;
* razlikuje timeout od cancellation-a;
* implementira cooperative cancellation;
* propagira cancellation kroz concurrent components;
* dizajnira goroutine exit paths;
* spreči goroutine leaks.

### Runtime

* objasni osnovni scheduler model;
* objasni odnos G/M/P;
* razume `GOMAXPROCS`;
* razlikuje runnable od running goroutines;
* razlikuje concurrency od parallelism;
* razume kada workload postaje CPU-bound ili I/O-bound.

---

# 38. Module 3 Philosophy

Centralna ideja ovog modula može se sažeti u jednu rečenicu:

> **Concurrent program nije samo skup goroutines koje rade paralelno; to je sistem sa shared state-om, synchronization pravilima, lifecycle-om i runtime execution modelom.**

Zato dobar concurrent design mora odgovoriti na četiri osnovna pitanja:

```text
1. Who owns the state?
2. Who synchronizes access?
3. How does the work stop?
4. How is the work actually scheduled?
```

Module 3 sistematski odgovara na sva četiri.

---

# Module 3 — Synchronization, Cancellation & Go Runtime

## 39. Topic 06 — Go Scheduler

Fajl:

```text
06-go-scheduler.md
```

Go scheduler je runtime komponenta odgovorna za upravljanje izvršavanjem goroutines.

Na application nivou developer piše:

```go
go worker()
```

ali runtime mora da odluči:

```text
kada će worker biti izvršen
na kom execution resource-u
koliko dugo će izvršavanje trajati
kada treba izvršiti drugu goroutine
kako rasporediti runnable goroutines
```

Zbog toga razumevanje scheduler-a postaje važno kada se prelazi sa osnovnog korišćenja concurrency primitives na performance engineering.

---

# 40. Goroutine Is Not an OS Thread

Jedna od najvažnijih činjenica:

```text
goroutine ≠ OS thread
```

Go runtime koristi veliki broj goroutines i raspoređuje ih preko manjeg ili odgovarajućeg broja OS threads.

Konceptualno:

```text
        Goroutines
     ┌────┬────┬────┬────┬────┐
     │ G1 │ G2 │ G3 │ G4 │ G5 │
     └─┬──┴─┬──┴─┬──┴─┬──┴─┬──┘
       │    │    │    │    │
       └────┴────┼────┴────┘
                  ▼
              Scheduler
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
        Thread  Thread  Thread
          1       2       3
```

Ovo je jedan od razloga zbog kojih Go može efikasno da podrži veoma veliki broj goroutines.

---

# 41. The G-M-P Model

Za razumevanje Go scheduler-a koristi se model:

```text
G = Goroutine
M = Machine / OS Thread
P = Processor
```

Konceptualno:

```text
                 P
                 │
                 ▼
              run queue
                 │
          ┌──────┼──────┐
          ▼      ▼      ▼
          G1     G2     G3
                 │
                 ▼
                 M
                 │
                 ▼
             OS Thread
```

Ova terminologija je ključna za razumevanje runtime scheduler-a.

---

# 42. G — Goroutine

`G` predstavlja runtime representation goroutine-a.

Goroutine poseduje state potreban da runtime može da:

* započne izvršavanje;
* suspenduje execution;
* nastavi execution;
* čuva execution context;
* blokira;
* postane runnable;
* završi.

Conceptual lifecycle:

```text
created
   │
   ▼
runnable
   │
   ▼
running
   │
   ├──► blocked
   │       │
   │       ▼
   │    runnable
   │
   └──► finished
```

Važno je razlikovati:

```text
created
runnable
running
blocked
dead
```

jer scheduler ne izvršava sve goroutines istovremeno.

---

# 43. M — Machine

`M` predstavlja OS thread kojim runtime izvršava Go code.

Simplifikovano:

```text
M
│
└── OS thread
```

Jedan M može izvršavati goroutine kada ima odgovarajući P.

Ali:

```text
goroutines ≠ Ms
```

Broj goroutines može biti ogroman u poređenju sa brojem OS threads.

---

# 44. P — Processor

`P` predstavlja runtime execution resource potreban da M izvršava Go code.

P sadrži runtime state potreban za scheduling, uključujući lokalnu runnable queue.

Konceptualno:

```text
P0
 │
 ├── local run queue
 │      ├── G1
 │      ├── G2
 │      └── G3
 │
 └── M
```

Broj P-ova je povezan sa:

```text
GOMAXPROCS
```

što će biti detaljno obrađeno u sledećoj temi.

---

# 45. Local Run Queue

Scheduler ne mora imati jednu globalnu queue za sve goroutines.

Svaki P ima lokalnu runnable queue.

Conceptual model:

```text
P0                    P1
│                     │
├── G1                ├── G7
├── G2                ├── G8
├── G3                └── G9
└── G4
```

Ovakva organizacija omogućava scheduler-u da smanji contention oko jedne globalne queue.

---

# 46. Global Run Queue

Pored lokalnih queues postoji i globalna runnable queue.

Conceptualno:

```text
             Global Run Queue
          ┌────────────────────┐
          │ G10 G11 G12 G13    │
          └─────────┬──────────┘
                    │
             ┌──────┴──────┐
             ▼             ▼
            P0             P1
        local queue     local queue
```

Scheduler može prebacivati work između globalnog i lokalnih queues.

Detalji implementacije mogu se menjati između Go verzija, zato mentalni model treba posmatrati kao runtime architecture, a ne kao immutable implementation contract.

---

# 47. Work Stealing

Jedan od važnih scheduler concepts je **work stealing**.

Pretpostavimo:

```text
P0                    P1
│                     │
├── G1                │
├── G2                │
├── G3                │
├── G4                │
└── G5                │
```

dok je P1 idle.

Scheduler može omogućiti P1 da preuzme deo runnable work-a:

```text
P0                    P1
│                     │
├── G1                ├── G4
├── G2                └── G5
└── G3
```

Cilj:

```text
load balancing
```

a ne striktno očuvanje ownership-a određene goroutine queue.

---

# 48. Why Work Stealing Matters

Bez load balancing-a moguće je:

```text
P0 → overloaded
P1 → idle
P2 → idle
P3 → idle
```

što znači da raspoloživi CPU resources nisu optimalno iskorišćeni.

Work stealing omogućava scheduler-u da redistribuira runnable work.

To je posebno relevantno kod:

```text
CPU-bound workloads
large number of goroutines
uneven task duration
parallel workloads
```

---

# 49. Goroutine States and Scheduler

Za mentalni model dovoljno je razlikovati nekoliko važnih stanja:

```text
                ┌───────────┐
                │  Created  │
                └─────┬─────┘
                      ▼
                ┌───────────┐
                │ Runnable  │
                └─────┬─────┘
                      ▼
                ┌───────────┐
                │  Running  │
                └──┬────┬───┘
                   │    │
             block │    │ finish
                   ▼    ▼
               Blocked  Dead
                   │
                   ▼
                Runnable
```

Scheduler uglavnom upravlja tranzicijama između:

```text
runnable
running
blocked
```

dok runtime lifecycle upravlja završetkom goroutine-a.

---

# 50. Blocking Operations

Goroutine može postati blocked zbog:

```text
channel receive
channel send
mutex acquisition
I/O
sleep
select
network wait
other synchronization
```

Primer:

```go
value := <-ch
```

Ako nema dostupne vrednosti:

```text
G
│
▼
blocked
```

Scheduler može dati execution opportunity drugoj runnable goroutine.

Ovo je jedan od ključnih razloga zbog kojih goroutine nije isto što i OS thread.

---

# 51. Blocking Does Not Necessarily Mean CPU Waste

Važno je razlikovati:

```text
goroutine blocked
```

od:

```text
CPU actively spinning
```

Ako goroutine čeka channel:

```go
value := <-ch
```

ona ne mora trošiti CPU dok čeka.

Scheduler može izvršavati druge goroutines.

Suprotan primer je busy loop:

```go
for {
    select {
    default:
    }
}
```

koji može stalno biti runnable i trošiti CPU.

---

# 52. Preemption

Moderni Go runtime poseduje mehanizme koji omogućavaju scheduler-u da prekine dugotrajno izvršavanje goroutine-a i omogući drugim goroutines da dobiju execution time.

Conceptualno:

```text
G1
│
├── execute
├── execute
├── execute
│
▼
preempt
│
▼
G2
```

Ovo je posebno važno za CPU-bound code.

Bez preemption-a jedna goroutine koja dugo izvršava CPU work mogla bi značajno otežati scheduling drugih goroutines.

---

# 53. Cooperative and Runtime Scheduling

Historijski se scheduling ponašanje Go runtime-a menjalo kroz verzije.

Zato je važno razlikovati:

```text
language semantics
```

od:

```text
runtime implementation details
```

Go program ne treba da zavisi od pretpostavke:

```text
"goroutine X će sigurno izvršiti pre goroutine Y"
```

osim kada program koristi eksplicitnu synchronization.

Scheduler nije deterministički API.

---

# 54. Scheduler Is Not a Synchronization Primitive

Nikada ne treba koristiti pretpostavku:

```go
go func() {
    value = 42
}()

fmt.Println(value)
```

i očekivati da scheduler garantuje da je goroutine završila.

Scheduling:

```text
does not imply synchronization
```

Ako je potreban ordering, koristiti odgovarajući synchronization mechanism:

```text
channel
WaitGroup
Mutex
Once
atomic
```

u zavisnosti od problema.

---

# 55. `runtime.Gosched`

Go runtime pruža:

```go
runtime.Gosched()
```

koji omogućava goroutine-i da dobrovoljno prepusti execution opportunity.

Conceptualno:

```text
G1
 │
 ├── work
 │
 └── Gosched()
       │
       ▼
     G2
```

Međutim, `Gosched` nije normalan replacement za:

```text
channel synchronization
Mutex
condition waiting
proper coordination
```

Ako program zahteva `Gosched` da bi "radio", vrlo često postoji dublji synchronization design problem.

---

# 56. Scheduler Fairness

Scheduler treba da omogući napredak različitim runnable goroutines.

Ali ne treba pretpostavljati strogu fairness garanciju kao application-level contract.

Ne treba pisati:

```text
G1 must run before G2
```

samo zato što je:

```text
go G1()
go G2()
```

napisano tim redom.

Ako je ordering važan:

```text
G1 → signal → G2
```

treba ga eksplicitno modelovati.

---

# 57. Scheduler and Channels

Channels imaju direktnu vezu sa scheduler behavior-om.

Kada goroutine izvrši:

```go
value := <-ch
```

a nema value-a:

```text
G
│
▼
blocked
```

Kada drugi goroutine izvrši:

```go
ch <- value
```

runtime može omogućiti waiting goroutine-i da nastavi.

Conceptualno:

```text
G1                  G2
│                   │
receive             send
│                   │
▼                   ▼
blocked         value available
│                   │
└─────────┬─────────┘
          ▼
       runnable
```

Zbog toga channel communication nije samo application-level abstraction, već je duboko povezana sa runtime scheduling-om.

---

# 58. Scheduler and Mutex

Sličan model postoji kod mutex contention-a.

Ako goroutine pokušava:

```go
mu.Lock()
```

dok je mutex zauzet, ona može čekati umesto da aktivno izvršava CPU spin forever.

Conceptualno:

```text
G1
│
├── Lock()
├── critical section
└── Unlock()

G2
│
└── Lock()
      │
      ▼
    waiting
```

Kada lock postane dostupan, G2 može nastaviti.

Zato synchronization primitives treba razumeti i kao mechanisms koji utiču na scheduling behavior.

---

# 59. Scheduler and I/O

Kod I/O-bound workload-a veliki broj goroutines može biti veoma koristan.

Primer:

```text
G1 → network wait
G2 → network wait
G3 → database wait
G4 → file wait
G5 → CPU work
```

Scheduler može omogućiti G5 da koristi CPU dok ostale goroutines čekaju.

Conceptualno:

```text
I/O waits
   │
   ├── G1
   ├── G2
   ├── G3
   └── G4
        │
        ▼
     CPU work
        │
        ▼
       G5
```

Ovo je jedan od razloga zašto Go concurrency model dobro odgovara server-side I/O workloads.

---

# 60. Scheduler and CPU-Bound Work

Kod CPU-bound workload-a situacija je drugačija.

Ako imamo:

```text
100000 goroutines
```

koje sve rade CPU-intensive calculations, problem više nije waiting za I/O.

Problem postaje:

```text
CPU capacity
scheduler overhead
cache locality
contention
parallelism
```

Tada je potrebno razumeti `GOMAXPROCS` i razliku između concurrency i parallelism.

---

# 61. Topic 07 — GOMAXPROCS

Fajl:

```text
07-gomaxprocs.md
```

`GOMAXPROCS` određuje maksimalan broj procesora koje Go scheduler može koristiti za istovremeno izvršavanje Go code-a.

Conceptualno:

```text
GOMAXPROCS = N
```

znači da runtime ima do `N` P execution contexts dostupnih za concurrent execution of Go code.

Simplifikovano:

```text
GOMAXPROCS = 4

P0    P1    P2    P3
│     │     │     │
▼     ▼     ▼     ▼
CPU   CPU   CPU   CPU
```

Ovo nije isto što i:

```text
number of goroutines
```

niti je isto što i:

```text
number of OS threads
```

---

# 62. GOMAXPROCS vs Goroutine Count

Primer:

```text
1000 goroutines
GOMAXPROCS = 4
```

ne znači da se 1000 goroutines izvršava istovremeno na CPU-u.

Već konceptualno:

```text
1000 Goroutines
       │
       ▼
Scheduler
       │
       ▼
4 execution contexts
```

Runnable goroutines se raspoređuju preko raspoloživih Ps.

---

# 63. GOMAXPROCS vs OS Threads

Takođe:

```text
GOMAXPROCS = 4
```

ne znači:

```text
exactly 4 OS threads
```

Go runtime može koristiti više OS threads u različitim situacijama.

Zato treba razlikovati:

```text
Goroutines
Ps / GOMAXPROCS
OS Threads
```

---

# 64. Why GOMAXPROCS Matters

Kod CPU-bound workload-a:

```text
GOMAXPROCS
```

direktno utiče na količinu CPU parallelism-a koju runtime može ostvariti.

Primer:

```text
GOMAXPROCS = 1
```

konceptualno:

```text
G1 → G2 → G3 → G4
```

jedan execution context.

Sa:

```text
GOMAXPROCS = 4
```

moguće je:

```text
G1 ──► CPU 1
G2 ──► CPU 2
G3 ──► CPU 3
G4 ──► CPU 4
```

ako workload i hardware omogućavaju stvarnu parallel execution.

---

# 65. `runtime.GOMAXPROCS`

Go pruža API:

```go
runtime.GOMAXPROCS(n)
```

koji može postaviti GOMAXPROCS i vraća prethodnu vrednost.

Postoji i način da se pročita runtime configuration kroz:

```go
runtime.GOMAXPROCS(0)
```

pri čemu se ne menja vrednost.

Međutim, application code uglavnom ne treba nasumično da menja GOMAXPROCS.

---

# 66. Why Manual GOMAXPROCS Tuning Is Dangerous

Naivna optimizacija:

```go
runtime.GOMAXPROCS(1)
```

ili:

```go
runtime.GOMAXPROCS(runtime.NumCPU())
```

nije automatski dobra ideja za svaki workload.

Performance zavisi od:

```text
CPU topology
workload type
I/O
cgo
blocking
contention
cache behavior
runtime version
container limits
application architecture
```

Zato tuning treba raditi na osnovu:

```text
measurement
profiling
benchmarking
production characteristics
```

---

# 67. CPU-Bound Example

Pretpostavimo:

```go
func expensiveWork() {
    for i := 0; i < 1_000_000_000; i++ {
        // CPU-intensive work
    }
}
```

Ako pokrenemo mnogo goroutines:

```go
for i := 0; i < 100; i++ {
    go expensiveWork()
}
```

100 goroutines ne znači 100-way parallelism.

Stvarni parallelism zavisi od:

```text
available CPUs
GOMAXPROCS
runtime scheduling
workload
```

---

# 68. I/O-Bound Example

Kod:

```text
HTTP requests
database queries
network calls
file operations
```

veliki broj goroutines može biti koristan čak i kada je broj istovremeno izvršavanih CPU contexts relativno mali.

Primer:

```text
1000 requests
     │
     ▼
goroutines
     │
     ├── waiting
     ├── waiting
     ├── waiting
     ├── CPU work
     └── waiting
```

Većina goroutines nije CPU-active u svakom trenutku.

---

# 69. GOMAXPROCS Is About Parallelism

Važno:

```text
GOMAXPROCS
```

je mnogo bliže konceptu:

```text
parallel execution capacity
```

nego:

```text
concurrency capacity
```

Možemo imati:

```text
1,000,000 goroutines
```

sa:

```text
GOMAXPROCS = 4
```

i dalje imati ogromnu concurrency.

Ali CPU parallelism je ograničen brojem execution contexts.

---

# 70. Topic 08 — Parallelism vs Concurrency

Fajl:

```text
08-parallelism-vs-concurrency.md
```

Ovo je jedna od najčešće pogrešno shvaćenih tema.

### Concurrency

Concurrency znači da više tasks može biti **in progress** i da se njihovo izvršavanje može preplitati.

```text
Time ─────────────────────►

Task A: ███     ███
Task B:    ███     ███
Task C:       ███
```

### Parallelism

Parallelism znači da se više tasks zaista izvršava **istovremeno** na različitim execution resources.

```text
CPU 1: █████████████
CPU 2: █████████████
CPU 3: █████████████
```

---

# 71. Concurrency Without Parallelism

Moguće je imati concurrency bez stvarnog parallelism-a.

Primer:

```text
GOMAXPROCS = 1

G1 ──work──wait──work──
G2 ─────work────wait──
G3 ─────────work──────
```

Tasks su concurrent, ali u datom trenutku samo jedan execution context radi Go code.

---

# 72. Parallelism Implies Concurrent Work

Ako imamo:

```text
CPU 1 → G1
CPU 2 → G2
```

onda se G1 i G2 zaista izvršavaju istovremeno.

To je:

```text
parallel execution
```

i ujedno predstavlja concurrent workload.

Ali concurrency ne zahteva parallelism.

---

# 73. Concurrency Is a Program Structure

Concurrency je često architectural property.

Na primer:

```text
HTTP request handler
     │
     ├── database operation
     ├── cache operation
     └── external service
```

Tasks mogu biti concurrent čak i ako se ne izvršavaju fizički istovremeno.

---

# 74. Parallelism Is an Execution Property

Parallelism zavisi od:

```text
hardware
runtime
GOMAXPROCS
scheduler
workload
```

Dakle:

```text
Concurrency
    ↓
how work is structured

Parallelism
    ↓
how work is physically executed
```

Ovo je izuzetno važna distinkcija.

---

# 75. CPU-Bound vs I/O-Bound

Za concurrency architecture treba razlikovati:

### CPU-bound

```text
work
work
work
work
```

CPU je bottleneck.

### I/O-bound

```text
CPU work
   ↓
I/O wait
   ↓
CPU work
   ↓
I/O wait
```

External system je često bottleneck.

Concurrency može pomoći I/O-bound workloads tako što CPU može obrađivati druge tasks dok jedan task čeka.

---

# 76. CPU-Bound Parallelism

Kod CPU-bound workload-a concurrency sama po sebi nije dovoljna.

Ako imamo:

```text
1000 CPU-bound goroutines
```

ali:

```text
GOMAXPROCS = 1
```

ne dobijamo 1000-way CPU parallelism.

Ako hardware ima više CPU cores i workload je pogodan, povećanje parallelism-a može smanjiti ukupno vreme izvršavanja do određene granice.

---

# 77. More Parallelism Is Not Always Better

Pretpostavimo:

```text
CPU cores = 8
```

Nije automatski optimalno imati:

```text
GOMAXPROCS = 100
```

za CPU-bound workload.

Više runnable work-a može dovesti do:

```text
scheduling overhead
cache contention
synchronization contention
context switching
memory pressure
```

Zato postoji optimalna tačka koja zavisi od workload-a.

---

# 78. Amdahl's Law Intuition

Ako deo programa mora biti izvršen sekvencijalno, postoji granica koliko ubrzanje može biti ostvareno povećanjem parallelism-a.

Conceptualno:

```text
Total Work
├───────────────┬───────────────┐
│ Parallel Part │ Serial Part   │
└───────────────┴───────────────┘
```

Ako je serial portion velik:

```text
more CPUs
    ↓
limited speedup
```

Concurrency design zato mora analizirati i:

```text
serialization points
```

kao što su:

* mutex;
* single-threaded consumer;
* shared resource;
* database bottleneck;
* external API;
* sequential pipeline stage.

---

# 79. Mutex and Parallelism

Mutex može smanjiti parallelism.

Na primer:

```text
G1 ──Lock──critical────Unlock
G2 ──wait────────────────────Lock
G3 ──wait────────────────────Lock
G4 ──wait────────────────────Lock
```

Ako je critical section veliki, veliki broj goroutines postaje:

```text
waiting
```

i CPU resources mogu ostati nedovoljno iskorišćeni.

Zato synchronization correctness i performance nisu potpuno odvojeni problemi.

---

# 80. False Sharing and Cache Effects

Na naprednijem nivou, čak i kada goroutines mogu fizički da se izvršavaju paralelno, shared memory architecture može ograničiti performance.

Primer:

```text
CPU 1 → variable A
CPU 2 → variable B
```

Ako se A i B nalaze na istoj cache line-i, izmene jednog podatka mogu uticati na cache coherence druge CPU jedinice.

Ovaj problem se naziva:

```text
false sharing
```

Ova tema pripada dubljem performance/memory delu concurrency-ja, ali je korisna kao uvod u činjenicu da:

> Parallel execution ne znači automatski linearno ubrzanje.

---

# 81. Scheduler + GOMAXPROCS + Parallelism

Ove tri teme treba posmatrati zajedno:

```text
Goroutines
     │
     ▼
Scheduler
     │
     ▼
G/M/P
     │
     ▼
GOMAXPROCS
     │
     ▼
Execution capacity
     │
     ▼
Parallelism
```

Scheduler određuje **ko će dobiti execution time**.

GOMAXPROCS određuje **koliko P execution contexts može istovremeno izvršavati Go code**.

Hardware određuje **koliko stvarnog CPU parallelism-a postoji**.

---

# 82. A Complete Runtime Mental Model

Jedan od glavnih ciljeva Module 3 jeste izgradnja sledećeg modela:

```text
                        Application
                             │
                             ▼
                        Goroutines
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          Channels         Mutex          Once
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                         Scheduler
                             │
                      ┌──────┼──────┐
                      ▼      ▼      ▼
                     P0     P1     P2
                      │      │      │
                      ▼      ▼      ▼
                     M0     M1     M2
                      │      │      │
                      ▼      ▼      ▼
                    OS Threads
                      │      │      │
                      └──────┼──────┘
                             ▼
                           CPUs
```

Ovaj diagram povezuje application-level concurrency sa runtime-level execution.

---

# 83. Runtime Details vs Language Guarantees

Veoma je važno da napredni Go developer razlikuje:

```text
What Go guarantees
```

od:

```text
How current runtime happens to implement it
```

Na primer, scheduler može menjati interne algoritme između Go verzija.

Application ne treba da zavisi od:

```text
exact scheduling order
exact goroutine placement
exact queue behavior
exact preemption timing
```

osim kada je određeno ponašanje eksplicitno garantovano API-jem ili memory model-om.

---

# 84. Practical Performance Investigation

Kada concurrency system ima performance problem, ne treba odmah menjati:

```text
GOMAXPROCS
```

ili dodavati:

```text
more goroutines
```

Prvo treba utvrditi bottleneck.

Tipičan proces:

```text
Measure
   ↓
Profile
   ↓
Identify bottleneck
   ↓
Form hypothesis
   ↓
Change one variable
   ↓
Benchmark
   ↓
Compare
```

Potential bottlenecks:

```text
CPU
memory
allocation
GC
mutex contention
channel contention
I/O
network
database
scheduler
```

---

# 85. Benchmarking Concurrency

Concurrency optimization treba testirati benchmark-om.

Primer:

```bash
go test -bench=. -benchmem ./...
```

Za različite konfiguracije:

```text
workers = 1
workers = 2
workers = 4
workers = 8
workers = 16
```

i različite workload types.

Ne treba zaključiti:

```text
"8 workers is faster"
```

na osnovu jednog pokretanja.

Potrebno je meriti:

```text
throughput
latency
allocations
CPU utilization
contention
```

---

# 86. Race Detector vs Performance

Race detector:

```bash
go test -race ./...
```

je fundamentalni alat za correctness.

Ali race detector menja runtime characteristics i nije performance benchmark.

Zato:

```text
Race testing
```

i:

```text
Performance testing
```

treba tretirati kao odvojene aktivnosti.

---

# 87. Mutex Profiling

Kada sumnjamo da mutex predstavlja bottleneck, treba istražiti:

```text
lock contention
critical section duration
number of acquisitions
```

Problem nije nužno:

```text
"Mutex is slow"
```

već često:

```text
"Too much work is serialized behind this mutex."
```

To je mnogo važnija architectural dijagnoza.

---

# 88. Blocked Goroutines as a Diagnostic Signal

Ako veliki broj goroutines čeka:

```text
mutex
channel
I/O
```

to ne znači automatski da postoji bug.

Ali veliki broj blocked goroutines može biti signal:

```text
resource bottleneck
backpressure
lock contention
slow downstream dependency
incorrect lifecycle
```

Zato goroutine count sam po sebi nije dovoljan metric.

---

# 89. Goroutine Count Is Not Throughput

Možemo imati:

```text
10 goroutines → high throughput
```

i:

```text
10,000 goroutines → low throughput
```

ako je bottleneck:

```text
database
mutex
CPU
network
external API
```

Concurrency treba meriti kroz workload-level metrics, ne samo kroz broj goroutines.

---

# 90. Production Concurrency Model

Na production nivou architecture često izgleda:

```text
                 Incoming Work
                       │
                       ▼
                 Request Layer
                       │
                 ┌─────┴─────┐
                 ▼           ▼
              Service A   Service B
                 │           │
                 ▼           ▼
             Worker Pool  Worker Pool
                 │           │
                 └─────┬─────┘
                       ▼
                  Shared State
                       │
                     Mutex
                       │
                       ▼
                   Storage
```

Istovremeno lifecycle može biti:

```text
Request
   │
   ▼
Context / Cancellation
   │
   ├── Worker A
   ├── Worker B
   └── Worker C
```

Runtime:

```text
Goroutines
    ↓
Scheduler
    ↓
P/M
    ↓
OS Threads
    ↓
CPU
```

Module 3 sada povezuje sva tri nivoa:

```text
application
lifecycle
runtime
```

---

# 91. Common Design Mistakes

## Mistake 1 — Using Mutex Everywhere

Mutex nije univerzalno rešenje.

Ako problem zapravo predstavlja:

```text
ownership transfer
```

channel može dati jednostavniji model.

---

## Mistake 2 — Using Channels Everywhere

Suprotno tome, channel nije uvek bolji.

Ako postoji mali shared mutable state sa jasnim invariantom:

```text
Mutex
```

može biti jednostavniji i efikasniji.

---

## Mistake 3 — Timeout Without Cancellation

```text
select {
case result := <-ch:
case <-time.After(timeout):
}
```

može vratiti caller-a dok worker i dalje radi.

Ako worker mora da bude zaustavljen, potrebna je cancellation path.

---

## Mistake 4 — Assuming Scheduler Ordering

Ne treba pretpostaviti:

```text
go A()
go B()

A always runs first
```

Scheduling order nije synchronization contract.

---

## Mistake 5 — Increasing GOMAXPROCS Blindly

Veći broj P-ova ne garantuje veći throughput.

---

## Mistake 6 — Increasing Goroutine Count Blindly

Više goroutines može povećati:

```text
contention
memory
scheduling overhead
downstream pressure
```

umesto throughput-a.

---

# 92. Design Review Checklist

Za svaki production concurrency component postaviti:

### Shared State

```text
What state is shared?
```

### Synchronization

```text
Who protects it?
```

### Critical Section

```text
How large is it?
```

### Communication

```text
Should ownership be transferred instead?
```

### Lifecycle

```text
How does the goroutine stop?
```

### Timeout

```text
What is the maximum acceptable wait?
```

### Cancellation

```text
How does cancellation propagate?
```

### Scheduling

```text
Is the workload CPU-bound or I/O-bound?
```

### Parallelism

```text
How much actual parallel execution is useful?
```

### Capacity

```text
What is the maximum concurrent work?
```

### Failure

```text
What happens when one component fails?
```

---

# 93. Module 3 — Integrated Architecture

Sada možemo spojiti sve teme:

```text
                          REQUEST
                             │
                             ▼
                       Cancellation
                             │
                             ▼
                       Timeout/Deadline
                             │
                             ▼
                       Service Layer
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
             Worker A     Worker B     Worker C
                │            │            │
                └────────────┼────────────┘
                             ▼
                       Shared State
                             │
                       ┌─────┴─────┐
                       ▼           ▼
                     Mutex      RWMutex
                       │
                       ▼
                     Once
                       │
                       ▼
                    Storage
                             │
                             ▼
                         Runtime
                             │
                      ┌──────┼──────┐
                      ▼      ▼      ▼
                     P0     P1     P2
                      │      │      │
                      └──────┼──────┘
                             ▼
                         OS Threads
                             │
                             ▼
                            CPU
```

Ovo predstavlja production-oriented mental model Module 3.

---

# 94. Key Distinctions

Module 3 treba da ostavi čitaocu jasne granice između sledećih pojmova:

| Pojam        | Osnovna uloga                           |
| ------------ | --------------------------------------- |
| `Mutex`      | mutual exclusion                        |
| `RWMutex`    | concurrent reads + exclusive writes     |
| `Once`       | one-time execution                      |
| Timeout      | vremensko ograničenje čekanja/operacije |
| Cancellation | signal za prekid rada                   |
| Goroutine    | lightweight concurrent execution unit   |
| Scheduler    | raspoređivanje goroutines               |
| P            | runtime execution context               |
| M            | OS thread representation                |
| GOMAXPROCS   | broj P execution contexts               |
| Concurrency  | više tasks in progress                  |
| Parallelism  | stvarno istovremeno izvršavanje         |

---

# 95. Module 3 — What Has Changed

Na kraju prethodnog modula model je bio:

```text
Goroutines
    ↓
Channels
    ↓
Coordination
    ↓
Concurrency Patterns
```

Sada je:

```text
Goroutines
    │
    ├── Communication
    │      └── Channels
    │
    ├── Coordination
    │      └── WaitGroup
    │
    ├── Synchronization
    │      ├── Mutex
    │      ├── RWMutex
    │      └── Once
    │
    ├── Lifecycle
    │      ├── Timeout
    │      └── Cancellation
    │
    └── Runtime
           ├── Scheduler
           ├── G/M/P
           ├── GOMAXPROCS
           └── Parallelism
```

Ovo je značajan prelaz od basic concurrency programming-a ka ozbiljnom concurrency engineering-u.

---

# 96. Module 3 — Practical Engineering Principle

Najvažniji engineering principle ovog dela modula:

> **Ne optimizuj concurrency model prema broju goroutines. Optimizuj ga prema bottleneck-u, workload-u i lifecycle zahtevima sistema.**

To znači da pre bilo kakvog tuning-a treba znati:

```text
What is the workload?
```

```text
Where is the bottleneck?
```

```text
What is the synchronization cost?
```

```text
What is the desired concurrency?
```

```text
What is the useful parallelism?
```

```text
What is the shutdown behavior?
```

---

# 97. Transition to Final Module 3 Section

Preostaje još završni deo Module 3.

On treba da objedini:

```text
Mutex
RWMutex
Once
Timeouts
Cancellation
Scheduler
GOMAXPROCS
Parallelism
```

u praktičan knowledge map.

Završni deo će takođe obuhvatiti:

```text
summary
design principles
debugging scenarios
performance scenarios
exercises
advanced exercises
capstone challenge
completion criteria
```

Poseban fokus treba da bude na tome da čitalac ne samo prepozna concurrency primitive, već da može da izabere odgovarajući primitive na osnovu konkretnog problema.

---

# 98. Module 3 — Final Mental Transition

Do ovog trenutka concurrency može da se posmatra kroz tri nivoa:

```text
                 CONCURRENCY
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
 Communication   Synchronization   Lifecycle
       │              │              │
   Channels        Mutex/Once    Timeout/Cancel
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                 Go Runtime
                      │
              ┌───────┼───────┐
              ▼       ▼       ▼
          Scheduler   P/M   GOMAXPROCS
              │
              ▼
          Parallelism
```

Sledeći korak je da se ovaj model proveri kroz konkretne failure modes, performance problems i production scenarios.

---

# Module 3 — Synchronization, Cancellation & Runtime

## Module Summary

Module 3 predstavlja prelaz sa osnovnih concurrency primitives i concurrency patterns ka ozbiljnom upravljanju:

* shared state;
* synchronization;
* goroutine lifecycle;
* cancellation;
* timeout-ima;
* runtime scheduling-u;
* CPU parallelism-u;
* production concurrency behavior-u.

Nakon prethodnih modula, čitalac već razume kako da kreira goroutines, koristi channels i konstruiše osnovne concurrency patterns.

Sada je potrebno razumeti **kada goroutines treba da čekaju, kako se štiti shared state, kako se prekida njihov rad i kako Go runtime fizički raspoređuje concurrency workload**.

---

# 99. Module 3 Knowledge Map

Celokupan Module 3 može se posmatrati kroz sledeću strukturu:

```text
Module 3
│
├── Synchronization
│   ├── Mutex
│   ├── RWMutex
│   └── Once
│
├── Lifecycle Control
│   ├── Timeouts
│   └── Cancellation
│
├── Runtime
│   ├── Go Scheduler
│   ├── G / M / P
│   └── GOMAXPROCS
│
└── Execution Model
    └── Concurrency vs Parallelism
```

Ovo nisu izolovane teme.

One formiraju jedan execution model:

```text
Goroutine
    │
    ├── communicates
    │
    ├── accesses shared state
    │
    ├── blocks
    │
    ├── gets cancelled
    │
    ├── gets scheduled
    │
    └── executes in parallel
```

---

# 100. Synchronization Decision Tree

Kada postoji shared state, prvo pitanje nije:

> "Da li treba Mutex?"

Već:

> "Kako treba organizovati ownership i access prema ovom state-u?"

Praktičan decision tree:

```text
Shared state?
    │
    ├── No
    │    └── No synchronization required
    │
    └── Yes
         │
         ├── Can ownership be transferred?
         │      └── Consider channels
         │
         └── Shared access required
                │
                ├── Simple exclusive access
                │      └── Mutex
                │
                ├── Many readers / few writers
                │      └── RWMutex
                │
                └── One-time initialization
                       └── Once
```

Ovo nije apsolutno pravilo, već početna heuristika.

---

# 101. Mutex — Engineering Rule

`sync.Mutex` treba koristiti kada invariant zahteva:

```text
exactly one goroutine at a time
```

Primer:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()

    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()

    return c.value
}
```

Ključni element nije sam `Mutex`.

Ključni element je:

```text
ownership of invariant
```

`Mutex` štiti state i pravila koja važe nad tim state-om.

---

# 102. Critical Section Design

Critical section treba da bude:

```text
as small as practical
```

Loš primer:

```go
mu.Lock()

result := expensiveOperation()

saveToDatabase(result)

sendNetworkRequest(result)

mu.Unlock()
```

Ovde mutex potencijalno štiti mnogo više nego što je potrebno.

Bolji dizajn često razdvaja:

```text
protected state manipulation
```

od:

```text
expensive external work
```

Na primer:

```go
mu.Lock()
value := state.value
mu.Unlock()

result := expensiveOperation(value)
```

Naravno, ovo je ispravno samo ako semantika programa dozvoljava da se state promeni između ta dva trenutka.

---

# 103. RWMutex — When It Helps

`RWMutex` može biti koristan kada postoji:

```text
many concurrent readers
few writers
```

Primer:

```text
Reader 1 ─┐
Reader 2 ─┼──► RLock
Reader 3 ─┘

Writer ───────► Lock
```

Readers mogu međusobno da se izvršavaju konkurentno.

Writer zahteva ekskluzivan access.

Ali `RWMutex` nije automatski brži od `Mutex`.

Ako:

```text
writes are frequent
critical sections are tiny
contention is low
```

običan `Mutex` može biti jednostavniji i efikasniji.

---

# 104. Once — Initialization Boundary

`sync.Once` treba koristiti kada postoji requirement:

```text
initialize exactly once
```

Primer:

```go
var (
    once   sync.Once
    client *Client
)

func GetClient() *Client {
    once.Do(func() {
        client = newClient()
    })

    return client
}
```

Mentalni model:

```text
First caller
     │
     ▼
 initialization
     │
     ▼
 completed
     │
     ▼
all future callers
     │
     ▼
reuse initialized state
```

`Once` nije general-purpose locking primitive.

Njegova semantika je mnogo preciznija:

> izvrši određenu funkciju jednom.

---

# 105. Synchronization and Memory Visibility

Synchronization nije samo pitanje:

```text
"Ko sme da pristupi podatku?"
```

Veoma je važno i:

```text
"When is another goroutine guaranteed to observe the state?"
```

Concurrency correctness zahteva pravilno ordering i visibility ponašanje.

Zato su:

```text
Mutex
channels
atomic operations
WaitGroup
Once
```

više od mehanizama za "čekanje".

Oni predstavljaju synchronization relationships između goroutines.

---

# 106. Timeout — Engineering Meaning

Timeout nije isto što i cancellation.

Timeout odgovara na pitanje:

> "Koliko dugo ćemo čekati?"

Cancellation odgovara na pitanje:

> "Da li treba prekinuti rad?"

Ova razlika je fundamentalna.

```text
Timeout
   │
   ▼
stop waiting / deadline exceeded
```

dok:

```text
Cancellation
   │
   ▼
tell work to stop
```

Jedna operacija može koristiti oba.

---

# 107. Timeout + Cancellation

Production pattern često izgleda:

```text
Request
   │
   ▼
Deadline
   │
   ▼
Context
   │
   ├── Worker A
   ├── Worker B
   └── Worker C
```

Kada deadline istekne:

```text
context.Done()
       │
       ├── A stops
       ├── B stops
       └── C stops
```

Ovo sprečava da request završi dok njegovi child goroutines nastavljaju da rade bez razloga.

---

# 108. Timeout Is a Boundary

Timeout treba posmatrati kao protection boundary.

Na primer:

```text
HTTP request
     │
     ▼
External API
     │
     ├── 50ms
     ├── 100ms
     ├── 200ms
     └── timeout
```

Bez timeout-a jedan spor downstream dependency može zadržati resource-e neograničeno dugo.

Posledice mogu biti:

```text
goroutine accumulation
connection exhaustion
queue growth
memory growth
latency amplification
```

---

# 109. Cancellation Must Propagate

Cancellation ima vrednost samo ako downstream components poštuju cancellation signal.

Loš model:

```text
Request cancelled
      │
      ▼
handler exits

worker continues forever
```

Bolji model:

```text
Request cancelled
      │
      ▼
context cancelled
      │
      ├── worker stops
      ├── DB operation stops
      └── external operation stops
```

Zato cancellation mora biti deo architecture-a, a ne samo dodat `context.Context` argument.

---

# 110. Context as Lifecycle Contract

`context.Context` treba koristiti za:

```text
cancellation
deadlines
request-scoped values
```

ali naročito je važan kao lifecycle propagation mechanism.

Tipičan tok:

```text
main
 │
 ▼
HTTP handler
 │
 ▼
service
 │
 ▼
repository
 │
 ▼
external call
```

Context treba da prati lifecycle:

```text
ctx
 │
 ├── handler
 ├── service
 ├── repository
 └── external call
```

Ako root operation bude cancelled:

```text
ctx.Done()
```

signal propagira niz dependency chain.

---

# 111. Goroutine Leak

Jedan od najvažnijih problema Module 3 jeste **goroutine leak**.

Primer:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        process(value)
    }
}
```

Ako producer nikada više ne šalje niti zatvara channel:

```text
worker
   │
   ▼
waiting forever
```

Goroutine ostaje živa.

To može izgledati bezazleno u malom programu, ali u serveru može dovesti do:

```text
memory growth
resource retention
scheduler overhead
blocked shutdown
```

---

# 112. Goroutine Leak Prevention

Za svaki spawned goroutine treba moći odgovoriti:

```text
How does it start?
How does it receive work?
How does it stop?
Who owns it?
What happens on error?
What happens on cancellation?
```

Ako na pitanje:

```text
"How does it stop?"
```

nema odgovora, architecture verovatno ima lifecycle problem.

---

# 113. Worker Lifecycle

Production worker često treba da ima:

```text
start
   │
   ▼
process
   │
   ├── work available
   │
   └── cancellation
          │
          ▼
        cleanup
          │
          ▼
        exit
```

Primer:

```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return

        case job, ok := <-jobs:
            if !ok {
                return
            }

            process(job)
        }
    }
}
```

Ovde postoje dve eksplicitne lifecycle paths:

```text
context cancellation
channel closure
```

---

# 114. Structured Concurrency Mindset

Go ne nameće striktan structured concurrency model kao jezičku konstrukciju, ali production Go code treba težiti sličnom principu:

```text
parent
 ├── child
 ├── child
 └── child
```

Parent treba da zna:

```text
who children are
when they finish
how they fail
how they stop
```

Drugim rečima:

> Goroutine ownership treba da bude eksplicitan.

---

# 115. WaitGroup and Lifecycle

`WaitGroup` rešava jednu specifičnu potrebu:

```text
wait until a group of goroutines finishes
```

Primer:

```go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)

    go func() {
        defer wg.Done()
        work()
    }()
}

wg.Wait()
```

Ali `WaitGroup` sam po sebi ne rešava:

```text
cancellation
error propagation
timeouts
worker shutdown
```

Zato ga treba kombinovati sa odgovarajućim lifecycle mechanisms.

---

# 116. Error Propagation

Production concurrency design mora odgovoriti na:

```text
What happens when one worker fails?
```

Naivni model:

```text
Worker A ── success
Worker B ── success
Worker C ── ERROR
Worker D ── success
```

može biti pogrešan ako failure C čini rezultat cele operacije nevalidnim.

Tada architecture često treba:

```text
C fails
 │
 ├── cancel siblings
 ├── stop accepting work
 └── return error
```

Ovo je jedan od razloga za korišćenje strukturiranih concurrency abstractions iz standardne biblioteke ili x/sync ekosistema kada je odgovarajuće.

---

# 117. Scheduler — What Developers Should Assume

Developer treba da pretpostavi:

```text
goroutines may run in any order
goroutines may be preempted
goroutines may block
goroutines may resume later
execution timing is nondeterministic
```

Developer ne treba da pretpostavi:

```text
goroutine A starts before B
goroutine A finishes before B
same goroutine always runs on same OS thread
specific scheduler queue ordering
specific preemption timing
```

Ovaj princip eliminiše veliki broj concurrency bugs.

---

# 118. Scheduler and Correctness

Correctness ne sme zavisiti od:

```text
timing
sleep
Gosched
"it usually runs first"
```

Na primer, ovo nije validna synchronization strategy:

```go
go update()

time.Sleep(time.Millisecond)

read()
```

Sleep može promeniti verovatnoću događaja, ali ne predstavlja formalnu synchronization relationship.

Ispravan dizajn koristi:

```text
channel
WaitGroup
Mutex
Once
atomic
context
```

u skladu sa problemom.

---

# 119. Scheduler and Performance

Scheduler overhead postaje relevantan kada program ima:

```text
extremely high goroutine churn
tiny tasks
heavy synchronization
massive runnable queues
```

Međutim, ne treba optimizovati scheduler pre nego što je potvrđeno da je scheduler bottleneck.

Pravilna procedura:

```text
measure
   ↓
profile
   ↓
identify scheduler-related cost
   ↓
optimize
```

a ne:

```text
assume scheduler is slow
   ↓
rewrite architecture
```

---

# 120. GOMAXPROCS — Practical Rule

U većini normalnih aplikacija:

```text
do not tune GOMAXPROCS blindly
```

Prvo razmotriti:

```text
CPU limits
container environment
workload characteristics
Go version
runtime defaults
benchmark results
```

Ako performance tuning zahteva promenu GOMAXPROCS-a, promena mora biti zasnovana na merenju.

---

# 121. Concurrency vs Parallelism — Interview-Level Distinction

### Concurrency

```text
dealing with multiple things at once
```

Program organizuje više independent tasks.

### Parallelism

```text
doing multiple things at exactly the same time
```

Hardware/runtime izvršava tasks simultaneously.

Kratka formula:

```text
Concurrency = structure
Parallelism = execution
```

Ovo je jedna od najvažnijih mentalnih formula celog concurrency tutorial-a.

---

# 122. Scenario 1 — HTTP Service

Pretpostavimo:

```text
1000 concurrent HTTP requests
```

Svaki request:

```text
parse
   ↓
database
   ↓
external API
   ↓
response
```

Praktičan model:

```text
1000 requests
      │
      ▼
1000-ish goroutines
      │
      ├── many waiting for I/O
      └── some doing CPU work
```

Ne znači:

```text
1000 CPU cores required
```

Concurrency omogućava efikasno korišćenje CPU-a dok drugi requests čekaju I/O.

---

# 123. Scenario 2 — CPU Image Processing

Pretpostavimo:

```text
1000 images
```

Svaka obrada je CPU-heavy.

Tada:

```text
1000 goroutines
```

nije automatski optimalno.

Bolji model može biti bounded worker pool:

```text
Jobs
 │
 ▼
Queue
 │
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

Broj workers treba prilagoditi workload-u i hardware-u.

---

# 124. Scenario 3 — Shared Cache

Pretpostavimo:

```go
type Cache struct {
    mu sync.RWMutex
    data map[string]Value
}
```

Ako:

```text
reads >> writes
```

`RWMutex` može imati smisla.

Ali ako je workload:

```text
reads ≈ writes
```

ili je contention minimalan:

```text
Mutex
```

može biti jednostavniji izbor.

Zaključak:

> Primitive treba birati prema access pattern-u, ne prema tome šta zvuči "naprednije".

---

# 125. Scenario 4 — Lazy Initialization

Ako expensive resource treba inicijalizovati jednom:

```text
first request
    │
    ▼
initialize
    │
    ▼
all future requests
    │
    ▼
reuse
```

`sync.Once` predstavlja prirodan primitive.

---

# 126. Scenario 5 — Downstream Timeout

Service A poziva Service B.

Ako B ne odgovara:

```text
A
│
▼
B
│
└── hangs
```

bez deadline-a može doći do accumulation-a.

Sa timeout-om:

```text
A
│
▼
B
│
└── deadline exceeded
        │
        ▼
      cancel
```

ovo štiti resource-e A.

---

# 127. Scenario 6 — Request Cancellation

Korisnik prekida HTTP request.

Ako backend nastavi:

```text
database query
external API
worker computation
```

nakon što rezultat više nije potreban, resursi se bespotrebno troše.

Bolji model:

```text
client disconnect
      │
      ▼
request context cancelled
      │
      ├── DB
      ├── external API
      └── workers
            │
            ▼
          stop
```

---

# 128. Scenario 7 — Graceful Shutdown

Production service treba da ima shutdown lifecycle:

```text
SIGTERM
  │
  ▼
stop accepting new work
  │
  ▼
cancel background work
  │
  ▼
wait for active work
  │
  ▼
cleanup resources
  │
  ▼
exit
```

Concurrency primitives Module 3 direktno učestvuju u ovom procesu.

---

# 129. Scenario 8 — Deadlock

Classic example:

```go
mu.Lock()
defer mu.Unlock()

anotherOperation()
```

Ako `anotherOperation()` pokušava da uzme isti non-reentrant mutex:

```text
G
│
├── Lock
│
└── Lock again
       │
       ▼
     deadlock
```

Ovo pokazuje zašto ownership i lock hierarchy moraju biti jasni.

---

# 130. Scenario 9 — Lock Ordering

Ako postoje:

```text
muA
muB
```

i:

```text
G1:
Lock(A)
Lock(B)

G2:
Lock(B)
Lock(A)
```

moguće je:

```text
G1 holds A
G2 holds B

G1 waits B
G2 waits A
```

što formira deadlock cycle.

Production systems zato često definišu:

```text
global lock ordering
```

---

# 131. Scenario 10 — Goroutine Leak

Ako worker:

```go
go worker(jobs)
```

nema:

```text
close(jobs)
```

ili:

```text
ctx.Done()
```

može ostati blocked forever.

Zato svaki background goroutine mora imati lifecycle owner-a.

---

# 132. Scenario 11 — Unbounded Concurrency

Naivni server:

```go
for _, job := range jobs {
    go process(job)
}
```

može kreirati:

```text
1,000 jobs  → 1,000 goroutines
1,000,000 jobs → 1,000,000 goroutines
```

ako input nema prirodno ograničenje.

Problem može biti:

```text
memory
downstream overload
scheduler pressure
file descriptors
connections
CPU
```

Rešenje često uključuje:

```text
bounded worker pool
semaphore
queue
backpressure
```

---

# 133. Scenario 12 — Backpressure

Production concurrency system treba da odgovori:

> Šta se dešava kada producer proizvodi work brže nego što consumer može da obradi?

Bez backpressure-a:

```text
Producer
   │
   ▼
unbounded queue
   │
   ▼
memory growth
```

Sa bounded capacity:

```text
Producer
   │
   ▼
bounded queue
   │
   ├── capacity available
   │
   └── capacity exhausted
            │
            ▼
        slow producer
```

Backpressure je ključan deo stabilnog concurrent system design-a.

---

# 134. Module 3 — Debugging Checklist

Kada concurrency bug postoji, prvo pitati:

### 1. Da li postoji data race?

```bash
go test -race ./...
```

### 2. Da li goroutine može da ostane blocked forever?

```text
channel?
mutex?
I/O?
select?
```

### 3. Ko je owner goroutine-e?

```text
who starts it?
who stops it?
```

### 4. Da li postoji cancellation path?

```text
ctx.Done()
```

### 5. Da li postoji timeout?

```text
deadline?
```

### 6. Da li postoji deadlock?

```text
lock ordering?
channel cycle?
```

### 7. Da li je concurrency bounded?

```text
worker pool?
semaphore?
queue capacity?
```

### 8. Gde je bottleneck?

```text
CPU?
memory?
lock?
channel?
I/O?
database?
```

---

# 135. Module 3 — Performance Checklist

Pre optimizacije:

```text
[ ] benchmark exists
[ ] workload is representative
[ ] bottleneck identified
[ ] CPU profile considered
[ ] memory profile considered
[ ] mutex/block profile considered
[ ] goroutine behavior understood
```

Tek onda:

```text
[ ] adjust worker count
[ ] reduce contention
[ ] reduce allocations
[ ] improve batching
[ ] tune queue size
[ ] evaluate GOMAXPROCS
```

---

# 136. Exercises — Level 1: Synchronization

### Exercise 1

Implement a thread-safe counter using `sync.Mutex`.

Requirements:

* `Inc()`;
* `Dec()`;
* `Value()`.

Run 100 goroutines that increment the counter.

Verify correctness using:

```bash
go test -race
```

---

### Exercise 2

Replace the implementation with `sync.RWMutex`.

Measure:

```text
read-heavy workload
write-heavy workload
```

Compare results.

---

### Exercise 3

Implement lazy initialization using `sync.Once`.

Verify that initialization executes exactly once even when 100 goroutines call the getter concurrently.

---

# 137. Exercises — Level 2: Lifecycle

### Exercise 4

Implement a worker that exits when:

```text
jobs channel closes
```

and when:

```text
context is cancelled
```

---

### Exercise 5

Create a worker pool with:

```text
N workers
bounded jobs channel
context cancellation
WaitGroup
```

Requirements:

* no goroutine leaks;
* graceful shutdown;
* workers stop on cancellation.

---

### Exercise 6

Add timeout behavior.

If a worker cannot complete its task before the deadline:

```text
cancel
return error
```

---

# 138. Exercises — Level 3: Runtime

### Exercise 7

Create a CPU-bound benchmark.

Compare:

```text
GOMAXPROCS = 1
GOMAXPROCS = 2
GOMAXPROCS = 4
GOMAXPROCS = 8
```

Record:

```text
execution time
CPU utilization
allocations
```

---

### Exercise 8

Create an I/O-bound benchmark.

Compare:

```text
1 worker
2 workers
4 workers
8 workers
16 workers
32 workers
```

Explain why the optimal number differs from the CPU-bound benchmark.

---

### Exercise 9

Create a workload with intentional mutex contention.

Measure how throughput changes when:

```text
critical section
```

becomes progressively larger.

---

# 139. Exercises — Level 4: Production Architecture

### Exercise 10 — Graceful Shutdown

Build a service with:

```text
HTTP server
background worker
job queue
context cancellation
WaitGroup
```

Implement:

```text
SIGTERM
   ↓
stop accepting work
   ↓
cancel workers
   ↓
wait
   ↓
shutdown
```

---

### Exercise 11 — Bounded Concurrency

Implement an API that processes arbitrary incoming jobs but never allows more than:

```text
N concurrent jobs
```

Compare:

```text
unbounded goroutines
```

against:

```text
bounded concurrency
```

---

### Exercise 12 — Backpressure

Build:

```text
producer
   ↓
bounded channel
   ↓
workers
```

Make the producer significantly faster than consumers.

Observe:

```text
queue saturation
producer blocking
throughput
latency
```

Explain how the bounded queue stabilizes the system.

---

# 140. Exercises — Level 5: Senior-Level Analysis

### Exercise 13 — Deadlock Analysis

Create two goroutines that acquire two mutexes in opposite order.

Identify:

```text
wait-for graph
deadlock cycle
```

Then fix the problem by establishing deterministic lock ordering.

---

### Exercise 14 — Goroutine Leak Detection

Create an intentionally leaking worker.

Use:

```text
runtime.NumGoroutine()
```

and profiling tools to identify the leak.

Then fix it using:

```text
context cancellation
channel closure
```

---

### Exercise 15 — Concurrency vs Parallelism

Create a workload that demonstrates:

```text
concurrency without parallelism
```

and another that demonstrates:

```text
actual parallelism
```

Explain what changes when:

```text
GOMAXPROCS
```

changes.

---

# 141. Capstone — Production Worker System

Build a production-oriented worker system with:

```text
                    Producer
                       │
                       ▼
                Bounded Job Queue
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Worker 1  Worker 2  Worker N
             │         │         │
             └─────────┼─────────┘
                       ▼
                    Results
                       │
                       ▼
                    Caller
```

The system must support:

* bounded concurrency;
* graceful shutdown;
* cancellation;
* deadlines;
* worker lifecycle;
* error propagation;
* synchronization;
* metrics;
* no goroutine leaks.

---

# 142. Capstone Requirements

The implementation should answer all of the following:

```text
How many workers exist?
```

```text
Who owns the workers?
```

```text
How are workers stopped?
```

```text
What happens when the queue is full?
```

```text
What happens when one worker fails?
```

```text
What happens when the caller cancels?
```

```text
What happens during shutdown?
```

```text
Can the system leak goroutines?
```

```text
Can shared state race?
```

```text
Can workers deadlock?
```

```text
What is the maximum concurrency?
```

---

# 143. Module 3 Completion Criteria

Čitalac je spreman za sledeći modul kada može samostalno da objasni:

### Synchronization

* razliku između `Mutex` i `RWMutex`;
* kada koristiti `Once`;
* šta predstavlja critical section;
* kako lock contention utiče na performance;
* kako nastaje deadlock.

### Lifecycle

* razliku između timeout-a i cancellation-a;
* kako `context.Context` propagira cancellation;
* kako sprečiti goroutine leak;
* kako implementirati graceful shutdown.

### Runtime

* šta predstavlja goroutine;
* šta predstavljaju G, M i P;
* kako scheduler raspoređuje runnable goroutines;
* šta predstavlja GOMAXPROCS;
* kako work stealing doprinosi load balancing-u.

### Execution

* razliku između concurrency i parallelism;
* CPU-bound vs I/O-bound workloads;
* zašto više goroutines ne znači automatski veći throughput;
* zašto veći GOMAXPROCS ne znači automatski bolje performanse.

### Engineering

* kako ograničiti concurrency;
* kako implementirati backpressure;
* kako dijagnostikovati contention;
* kako koristiti race detector;
* kako meriti concurrency performance.

---

# 144. Senior-Level Questions

Pre prelaska na sledeći modul čitalac treba da može da odgovori na sledeća pitanja bez oslanjanja na memorisane definicije.

### Question 1

Zašto `Mutex` nije samo performance cost, već i memory synchronization mechanism?

### Question 2

Kada je `RWMutex` zapravo lošiji izbor od `Mutex`?

### Question 3

Zašto timeout bez cancellation-a može ostaviti goroutine koja nastavlja rad?

### Question 4

Kako goroutine leak može nastati iako nema data race-a?

### Question 5

Zašto `time.Sleep()` nije synchronization primitive?

### Question 6

Šta znači da je scheduler nondeterministic?

### Question 7

Koja je razlika između:

```text
G
M
P
```

### Question 8

Kako GOMAXPROCS utiče na parallelism?

### Question 9

Zašto 1000 goroutines ne znači 1000-way parallel execution?

### Question 10

Zašto CPU-bound i I/O-bound workloads zahtevaju različit concurrency tuning?

---

# 145. Module 3 — Final Principles

Module 3 treba zapamtiti kroz sledeće principe:

## Principle 1

> **Shared mutable state requires explicit synchronization or ownership discipline.**

---

## Principle 2

> **A goroutine needs an explicit lifecycle.**

---

## Principle 3

> **Timeout answers "how long?", cancellation answers "should this work stop?".**

---

## Principle 4

> **Scheduler behavior must never be used as an application-level ordering guarantee.**

---

## Principle 5

> **More goroutines do not automatically mean more throughput.**

---

## Principle 6

> **More parallelism does not automatically mean better performance.**

---

## Principle 7

> **GOMAXPROCS should be tuned based on measurements, not intuition.**

---

## Principle 8

> **Bounded concurrency is usually safer than unbounded concurrency.**

---

## Principle 9

> **Every background goroutine should have an owner and a shutdown path.**

---

## Principle 10

> **Concurrency correctness comes before concurrency optimization.**

---

# 146. Module 3 — Final Architecture

Na kraju Module 3, kompletan concurrency model treba da izgleda ovako:

```text
                           APPLICATION
                                │
                                ▼
                           Goroutines
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
     Communication        Synchronization        Lifecycle
          │                     │                     │
      Channels             Mutex/RWMutex        Context
          │                  Once                  │
          │                     │              Timeout
          │                     │                  │
          └─────────────────────┼──────────────────┘
                                │
                                ▼
                           Go Runtime
                                │
                       ┌────────┼────────┐
                       ▼        ▼        ▼
                      G        M        P
                       │        │        │
                       └────────┼────────┘
                                ▼
                           Scheduler
                                │
                                ▼
                          GOMAXPROCS
                                │
                                ▼
                           Parallelism
                                │
                                ▼
                             CPUs
```

Ovaj model predstavlja osnovu za razumevanje ozbiljnih Go concurrent sistema.

---

# 147. Transition to Module 4

Module 3 je završio prelaz:

```text
basic concurrency
        │
        ▼
synchronization
        │
        ▼
lifecycle management
        │
        ▼
runtime understanding
        │
        ▼
parallel execution
```

Sledeći nivo treba da objedini sve prethodne koncepte u širi concurrency engineering model.

Module 4 treba da ide dalje od osnovnih primitives i da uvede:

```text
advanced concurrency architecture
memory model
happens-before
data races
lock-free programming
channel internals
scheduler internals
high-performance patterns
production failure modes
```

Na taj način Module 4 predstavlja prirodan prelaz sa:

```text
"How do I use Go concurrency?"
```

na:

```text
"How does Go concurrency actually work,
and how do I engineer it correctly at scale?"
```

---

# 148. End of Module 3

Module 3 je time kompletiran.

Obrađene oblasti:

```text
01. Mutex
02. RWMutex
03. Once
04. Timeouts
05. Cancellation
06. Go Scheduler
07. GOMAXPROCS
08. Parallelism vs Concurrency
```

Glavni cilj ovog modula nije memorisanje API-ja.

Cilj je razvoj sposobnosti da se concurrency problem posmatra kroz:

```text
state
ownership
synchronization
lifecycle
cancellation
scheduling
parallelism
performance
```

Tek kada su ti aspekti jasno definisani, moguće je napraviti concurrency architecture koja je istovremeno:

```text
correct
safe
bounded
observable
maintainable
performant
```