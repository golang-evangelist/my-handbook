
# Module 4 — Advanced Go Concurrency

Module 4 predstavlja **napredni nivo Go concurrency-ja**.

Dok su prethodni moduli izgradili osnovno razumevanje goroutines, channels, synchronization, cancellation i Go scheduler-a, Module 4 prelazi na probleme koji se pojavljuju kada concurrency sistem treba da bude:

* visokih performansi;
* skalabilan;
* otporan na contention;
* bez data race-ova;
* pravilno sinhronizovan;
* memory-model correct;
* bounded;
* observability-friendly;
* spreman za production workload;
* sposoban da funkcioniše i u distribuiranom sistemu.

Struktura ovog modula je zasnovana na postojećoj organizaciji repozitorijuma, koja obuhvata `sync/atomic`, atomic patterns, Go Memory Model, lock-free programming, worker pools, pipelines, semaphores, concurrency patterns, anti-patterns, performance/profiling, advanced architecture, distributed concurrency i završni pregled Go concurrency-ja.

---

# 1. Position of Module 4

Module 4 nije zamišljen kao još jedan skup API primera.

Njegov cilj je da čitalac razume **što se događa ispod concurrency abstractions** i da na osnovu toga donosi engineering odluke.

Prethodni moduli su prvenstveno odgovorili na:

```text
How do I write concurrent Go code?
```

Module 4 prelazi na:

```text
Why is this concurrent design correct?

Why is it performant?

Why does it scale?

What guarantees does the Go memory model provide?

When should I use atomic operations instead of locks?

When is lock-free programming justified?

How do I diagnose contention?

How do I design concurrency boundaries in production?

What changes when concurrency crosses process boundaries?
```

---

# 2. Module 4 Structure

Module 4 sadrži sledeće glavne oblasti:

```text
module-4/
│
├── 01-sync-atomic.md
├── 02-atomic-operations-patterns.md
├── 03-go-memory-model.md
├── 04-lock-free-programming.md
├── 05-worker-pools.md
├── 06-pipelines.md
├── 07-semaphore-pattern.md
├── 08-concurrency-patterns.md
├── 09-concurrency-anti-patterns.md
├── 10-performance-and-profiling.md
├── 11-advanced-concurrency-architecture.md
├── 12-distributed-concurrency.md
├── 13-mastering-go-concurrency.md
│
├── extra/
│
└── README.md
```

Ovakva struktura omogućava da se tema razvija od **low-level synchronization primitives** ka **system-level concurrency architecture**.

---

# 3. Learning Progression

Redosled nije slučajan.

Može se predstaviti kao:

```text
Atomic Operations
       │
       ▼
Memory Model
       │
       ▼
Lock-Free Programming
       │
       ▼
Concurrency Patterns
       │
       ▼
Bounded Concurrency
       │
       ▼
Performance
       │
       ▼
Architecture
       │
       ▼
Distributed Concurrency
       │
       ▼
Concurrency Engineering
```

Drugim rečima:

```text
primitive
   ↓
semantics
   ↓
pattern
   ↓
performance
   ↓
architecture
   ↓
distributed system
```

---

# 4. Module 4 Learning Objectives

Po završetku modula čitalac treba da bude sposoban da:

* koristi `sync/atomic` kada je atomic synchronization odgovarajuća apstrakcija;
* razume razliku između atomicity, visibility i ordering;
* interpretira osnovne guarantee-e Go Memory Model-a;
* prepozna kada program ima data race;
* razume koncept lock-free algoritama;
* razlikuje lock-free, wait-free i obstruction-free pristupe;
* konstruiše bounded worker pool;
* konstruiše pipeline sa pravilnim lifecycle management-om;
* koristi semaphore pattern za ograničavanje concurrency-ja;
* bira odgovarajući concurrency pattern;
* prepozna concurrency anti-pattern;
* profilira CPU, memory, goroutine, blocking i mutex behavior;
* analizira contention;
* dizajnira concurrency boundary-je na nivou sistema;
* razume razliku između local concurrency i distributed concurrency;
* projektuje concurrent system koji ima jasne ownership i lifecycle granice.

---

# 5. From Synchronization to Memory Semantics

U prethodnim modulima `Mutex` je predstavljen kao synchronization primitive.

Module 4 ide korak dublje.

Potrebno je razumeti da concurrent program ne zavisi samo od:

```text
who executes?
```

nego i od:

```text
what values can another goroutine observe?
```

To nas vodi ka:

```text
Atomicity
   +
Visibility
   +
Ordering
   =
Concurrency correctness
```

Ovo je osnova za razumevanje `sync/atomic` i Go Memory Model-a.

---

# 6. Atomic Operations

Prva oblast Module 4 je:

```text
01-sync-atomic.md
```

Tema je `sync/atomic`.

Atomic operations omogućavaju da određene operacije nad shared state-om budu izvršene bez eksplicitnog `Mutex`-a.

Tipični primeri uključuju:

```text
atomic load
atomic store
atomic add
atomic swap
atomic compare-and-swap
```

Mentalni model:

```text
ordinary read/write
       │
       ▼
possible race
```

naspram:

```text
atomic operation
       │
       ▼
well-defined synchronization semantics
```

Međutim:

> Atomic operation nije magično rešenje za svaki shared-state problem.

---

# 7. Atomicity Is Not the Whole Story

Naivno razmišljanje:

> "Ako koristim atomic, moj program je thread-safe."

nije dovoljno precizno.

Potrebno je analizirati:

```text
What state is atomic?

What invariant spans multiple variables?

What ordering is required?

What visibility guarantee is needed?

Can the entire operation be expressed as one atomic transition?
```

Na primer, ako invariant zahteva istovremenu promenu:

```text
balance
+
transactionState
+
timestamp
```

jedan atomic counter možda nije dovoljan.

Tada `Mutex` ili druga synchronization abstraction može biti prikladnija.

---

# 8. Atomic Counters

Najjednostavniji atomic pattern je counter.

Konceptualno:

```text
Goroutine A ─┐
Goroutine B ─┼──► atomic increment
Goroutine C ─┘
```

Za razliku od običnog:

```go
counter++
```

koji predstavlja read-modify-write sequence, atomic increment pruža jednu atomic operation.

To je naročito korisno za:

* metrics;
* counters;
* sequence numbers;
* state flags;
* lightweight coordination.

---

# 9. Compare-and-Swap

Jedan od najvažnijih atomic primitives je:

```text
Compare-And-Swap
```

Mentalni model:

```text
if current == old {
    current = new
}
```

kao jedna atomic transition.

Vizuelno:

```text
Current State
     │
     ▼
compare(old)
     │
 ┌───┴────┐
 │        │
match    mismatch
 │        │
 ▼        ▼
update   retry/fail
```

CAS je važan zato što predstavlja osnovu velikog broja lock-free algoritama.

---

# 10. Atomic State Machines

Atomic operations mogu predstavljati state transitions.

Na primer:

```text
NEW
 │
 ▼
RUNNING
 │
 ├──► FAILED
 │
 └──► COMPLETED
```

Ako više goroutines pokušava da promeni state:

```text
G1 ─┐
G2 ─┼──► state transition
G3 ─┘
```

CAS može obezbediti da samo validna tranzicija uspe.

Ovo je fundamentalni pattern za:

* lifecycle state;
* initialization state;
* shutdown state;
* connection state;
* one-way transitions.

---

# 11. Atomic Flags

Jednostavan atomic flag može predstavljati stanje:

```text
running = true
```

ili:

```text
stopped = true
```

Ali kod složenijih lifecycle state-ova treba izbegavati gomilu nezavisnih boolean vrednosti:

```text
started
stopping
stopped
failed
closed
```

jer mogu nastati nemoguće kombinacije.

Bolji model može biti explicit state machine:

```text
0 = NEW
1 = RUNNING
2 = STOPPING
3 = STOPPED
4 = FAILED
```

uz strogo definisane transitions.

---

# 12. Atomic Operations and Invariants

Najvažnije pitanje pre korišćenja atomics-a:

> Koji invariant pokušavamo da zaštitimo?

Primer:

```text
count >= 0
```

može biti lako predstavljen atomic counter-om.

Ali invariant:

```text
items + reserved == capacity
```

zahteva koordinaciju između više vrednosti.

Ako se te vrednosti moraju menjati zajedno, atomic primitive nad jednom vrednošću nije dovoljan.

Tada je često:

```text
Mutex + invariant
```

jednostavniji i sigurniji dizajn.

---

# 13. Atomic vs Mutex

Praktična heuristika:

| Problem                                | Tipičan izbor      |
| -------------------------------------- | ------------------ |
| Simple counter                         | Atomic             |
| Boolean/state flag                     | Atomic             |
| Sequence number                        | Atomic             |
| CAS state transition                   | Atomic             |
| Multiple related fields                | Mutex              |
| Complex invariant                      | Mutex              |
| Long critical section                  | Mutex              |
| Complex mutation logic                 | Mutex              |
| Read/write shared structure            | Mutex/RWMutex      |
| High-performance specialized algorithm | Atomic / lock-free |

Ovo nije hard rule.

Benchmark i correctness analiza imaju poslednju reč.

---

# 14. Atomic Operations Patterns

Druga oblast:

```text
02-atomic-operations-patterns.md
```

prelazi sa API-ja na reusable patterns.

Relevantni patterns uključuju:

```text
atomic counter
atomic flag
atomic state machine
CAS loop
copy-on-write
immutable snapshot
publication
lock-free coordination
```

Cilj nije memorisanje pojedinačnih funkcija.

Cilj je prepoznati problem:

```text
"Da li je ovo state koji mogu bezbedno predstaviti jednom atomic transition-om?"
```

---

# 15. CAS Loop

Tipičan CAS loop ima oblik:

```text
load current
    │
    ▼
calculate next
    │
    ▼
CAS(current, next)
    │
 ┌──┴──┐
 │     │
success failure
 │     │
 ▼     └────► retry
done
```

Pseudo-code:

```go
for {
    old := load()

    next := compute(old)

    if CAS(old, next) {
        break
    }
}
```

CAS loop je moćan, ali može imati problem sa contention-om.

Ako veliki broj goroutines stalno pokušava da promeni isti state:

```text
G1 ─┐
G2 ─┤
G3 ─┼──► CAS contention
G4 ─┤
G5 ─┘
```

može doći do mnogo failed retries.

---

# 16. Retry and Contention

Lock-free ne znači:

```text
no contention
```

Naprotiv, lock-free algoritam može imati ekstreman contention na jednoj atomic lokaciji.

Primer:

```text
             ┌── G1 retry
             │
             ├── G2 retry
             │
shared state ┼── G3 success
             │
             ├── G4 retry
             │
             └── G5 retry
```

Zato treba razlikovati:

```text
blocking
```

od:

```text
contention
```

Lock-free algoritam može eliminisati blocking, ali i dalje imati ozbiljan contention.

---

# 17. Copy-on-Write

Jedan koristan pattern jeste:

```text
readers
   │
   ▼
immutable snapshot
```

Writer:

```text
old state
   │
   ▼
copy
   │
   ▼
modify copy
   │
   ▼
publish new state
```

Readers ne moraju da zaključavaju svaki read.

Ovaj pristup je posebno koristan kada:

```text
reads >> writes
```

i state može relativno jeftino da se kopira.

---

# 18. Immutable State

Jedan od najjačih concurrency patterns je:

> Ne sinhronizuj shared mutable state ako shared mutable state možeš eliminisati.

Umesto:

```text
many goroutines
      │
      ▼
shared mutable object
      │
      ▼
many locks
```

može se koristiti:

```text
immutable snapshot
      │
 ┌────┼────┐
 ▼    ▼    ▼
G1   G2   G3
```

Ovo smanjuje synchronization complexity.

---

# 19. Publication Pattern

Kada se novi state pripremi:

```text
build new state
       │
       ▼
publish atomically
       │
       ▼
readers see complete snapshot
```

Ključna ideja:

> Readers nikada ne treba da vide parcijalno konstruisan state.

Ovo zahteva pravilno korišćenje synchronization semantics, ne samo običan pointer assignment.

---

# 20. Go Memory Model

Treća oblast:

```text
03-go-memory-model.md
```

predstavlja jednu od najvažnijih tema celog tutorial-a.

Memory Model odgovara na pitanje:

> Koje ponašanje concurrent Go programa je garantovano, a koje nije?

Ovo je mnogo dublje od:

```text
"race detector kaže da nema race-a."
```

Race detector je alat za pronalaženje problema.

Memory Model definiše semantiku.

---

# 21. Data Race

Data race postoji kada:

```text
two goroutines
access same memory location
concurrently
```

i:

```text
at least one access is a write
```

bez odgovarajuće synchronization.

Konceptualno:

```text
G1: write(x)
        │
        │
        ▼
       x
        ▲
        │
        │
G2: read(x)
```

bez synchronization relationship-a.

---

# 22. Race-Free Is Necessary, Not Sufficient

Važna engineering lekcija:

```text
No data race
```

ne znači automatski:

```text
Correct application
```

Program može biti race-free, ali logički pogrešan.

Na primer:

```text
G1 reads state
G2 changes state
G1 makes decision based on stale state
```

može biti legalno sa aspekta data race-a, ali pogrešno sa aspekta poslovne logike.

Zato concurrency correctness ima više nivoa:

```text
Memory safety
      ↓
Race freedom
      ↓
Synchronization correctness
      ↓
Invariant correctness
      ↓
Application correctness
```

---

# 23. Happens-Before

Jedan od centralnih pojmova Memory Model-a je:

```text
happens-before
```

Ako događaj A happens-before događaja B, program ima definisanu ordering relationship između njih.

Konceptualno:

```text
A
│
│ synchronization
▼
B
```

To nije isto što i:

```text
A happened earlier according to wall-clock time
```

Radi se o **formalnoj ordering relationship**.

---

# 24. Synchronization Creates Ordering

Odgovarajuće synchronization mechanisms mogu uspostaviti ordering.

Na primer:

```text
G1
 │
 ▼
write(x)
 │
 ▼
send(ch)
 │
 ▼
receive(ch)
 │
 ▼
G2
 │
 ▼
read(x)
```

Channel communication nije samo prenos vrednosti.

Ona predstavlja synchronization event.

Slično, određene operacije nad:

```text
Mutex
WaitGroup
Once
atomic
```

imaju synchronization semantics.

---

# 25. Visibility

Concurrent programming mora rešiti pitanje:

```text
When is a write observable by another goroutine?
```

Bez odgovarajućeg synchronization:

```text
G1 writes x
     │
     ▼
    x
     ▲
     │
G2 reads x
```

program ne sme da se oslanja na pretpostavku da će G2 "sigurno odmah videti" G1 write.

Synchronization establishes the required ordering and visibility guarantees.

---

# 26. Sequential Consistency Intuition

Za veliki broj praktičnih atomic use cases korisno je razmišljati kroz jednostavan model:

```text
operations appear in a single consistent order
```

Ali precizno razumevanje Memory Model-a zahteva čitanje formalnih guarantees, posebno kada se kombinuju:

```text
ordinary memory accesses
atomic operations
channels
locks
goroutine creation
goroutine termination
```

Cilj ovog modula je da čitalac ne koristi intuitivno pravilo tamo gde je potrebna formalna guarantee.

---

# 27. Goroutine Creation and Synchronization

Pokretanje goroutine-a takođe predstavlja važnu concurrency boundary.

Mentalni model:

```text
parent goroutine
      │
      ▼
   go f()
      │
      ▼
child goroutine
```

Ali ne treba pretpostaviti proizvoljan execution order između parent-a i child-a samo zato što je `go` statement izvršen.

Ako redosled ima semantički značaj:

```text
explicit synchronization
```

mora da ga uspostavi.

---

# 28. Goroutine Completion

Slično važi i za završetak goroutine-a.

Ako jedna goroutine treba da sačeka da druga završi:

```text
worker
  │
  ▼
complete
  │
  ▼
WaitGroup / channel / other synchronization
  │
  ▼
caller continues
```

Ne treba koristiti:

```go
time.Sleep(...)
```

kao zamenu za lifecycle synchronization.

---

# 29. Memory Model and API Design

Memory Model nije samo tema za compiler/runtime eksperte.

Direktno utiče na API design.

Na primer:

```text
Who owns this object?

Can callers mutate it concurrently?

Does this method require external synchronization?

Does this API publish state safely?

Can this object be reused after Close()?

```

Concurrent API treba da ima jasno definisane ownership i synchronization semantics.

---

# 30. What Module 4 Changes

Na kraju prve faze Module 4, concurrency više ne posmatramo samo kao:

```text
goroutines + channels
```

nego kao:

```text
             Concurrent System
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Execution    Synchronization  State
       │            │            │
   Scheduler      Atomics       Ownership
       │            │            │
   Parallelism    Mutexes       Immutability
                    │
                    ▼
              Memory Model
                    │
                    ▼
             Correctness
```

Ovo predstavlja osnovu za sledeće oblasti Module 4:

```text
lock-free programming
worker pools
pipelines
semaphores
advanced patterns
anti-patterns
performance
architecture
distributed concurrency
```

---

# 31. Module 4 — Core Mental Model

Do kraja ovog dela čitalac treba da počne da razmišlja u sledećem obliku:

```text
                    CONCURRENCY PROBLEM
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
            State         Lifecycle      Workload
              │              │              │
              ▼              ▼              ▼
         Ownership       Cancellation    CPU / I/O
              │              │              │
              ▼              ▼              ▼
       Synchronization     Timeout       Boundedness
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                        Architecture
                             │
                             ▼
                       Measurement
                             │
                             ▼
                         Production
```

Ovo je centralna perspektiva Module 4.

---

# 32. Key Principles

### Principle 1

> **Atomic operations are synchronization primitives, not universal replacements for locks.**

### Principle 2

> **Choose synchronization based on the invariant you need to protect.**

### Principle 3

> **Lock-free does not mean contention-free.**

### Principle 4

> **Race freedom does not automatically imply application correctness.**

### Principle 5

> **Correct concurrent programs require explicit synchronization relationships.**

### Principle 6

> **Do not rely on scheduler timing as a synchronization mechanism.**

### Principle 7

> **Immutable state can eliminate synchronization rather than merely optimize it.**

### Principle 8

> **Concurrent API design must define ownership, lifecycle and synchronization expectations.**

---

# 33. Preparation for the Next Part

Sledeći deo će produbiti temu **lock-free programming**.

Biće obrađeni:

```text
lock-free vs blocking
wait-free vs lock-free
obstruction-free algorithms
CAS loops
ABA problem
contention
progress guarantees
atomic state transitions
lock-free data structures
trade-offs između Mutex i atomics
```

Poseban fokus biće na pitanju:

```text
When does eliminating a lock actually improve a system,
and when does it merely make the system harder to reason about?
```

To je ključna granica između:

```text
using atomics
```

i:

```text
engineering lock-free concurrency
```

---

# 34. Lock-Free Programming

Četvrta ključna oblast ovog modula je:

```text
04-lock-free-programming.md
```

Lock-free programming predstavlja pokušaj da se concurrency algoritmi konstruišu bez tradicionalnih blocking locks.

Umesto:

```text
Goroutine
   │
   ▼
Lock()
   │
   ├── critical section
   │
   ▼
Unlock()
```

koristi se model:

```text
Goroutine
   │
   ▼
read state
   │
   ▼
compute new state
   │
   ▼
CAS
   │
 ┌─┴──────────────┐
 │                │
success          failure
 │                │
 ▼                ▼
done             retry
```

Ovo može omogućiti veoma performantne algoritme, ali uz značajno veću kompleksnost.

---

# 35. Blocking vs Lock-Free

Blocking algoritam može da dovede goroutine u stanje:

```text
RUNNING
   │
   ▼
WAITING
   │
   ▼
LOCK AVAILABLE
   │
   ▼
RUNNING
```

Lock-free algoritam pokušava da izbegne takvo čekanje.

Kod CAS pristupa:

```text
G1 ───────► success
G2 ──► retry
G3 ──► retry
G4 ──► retry
```

neka goroutine može neuspešno pokušavati više puta, ali sistem kao celina nastavlja da ostvaruje progress.

Ovo vodi ka važnom konceptu:

> **Progress guarantee.**

---

# 36. Progress Guarantees

Concurrency algoritmi mogu imati različite nivoe garancije napretka.

Najvažniji termini su:

```text
obstruction-free
lock-free
wait-free
```

Hijerarhijski:

```text
                    Stronger guarantee
                           ▲
                           │
                     Wait-Free
                           │
                     Lock-Free
                           │
                  Obstruction-Free
                           │
                     No Guarantee
```

Što je guarantee jači, implementacija je često kompleksnija.

---

# 37. Obstruction-Free

Algoritam je obstruction-free ako:

> pojedinačna goroutine može završiti operaciju u konačnom broju koraka kada nema konkurencije.

Drugim rečima:

```text
G1
 │
 ▼
alone
 │
 ▼
progress
```

Ali ako se pojave druge goroutines:

```text
G1 ─┐
G2 ─┼──► contention
G3 ─┘
```

nema nužno garancije da će određena goroutine završiti.

---

# 38. Lock-Free

Lock-free algoritam pruža jaču garanciju:

> sistem kao celina uvek ostvaruje progress.

To ne znači:

```text
every goroutine progresses
```

već:

```text
at least one goroutine
eventually completes
```

Primer:

```text
G1 ── retry ── retry ── success
G2 ── retry ── success
G3 ── retry
```

Iako G3 možda dugo ne napreduje, sistem nije potpuno blokiran.

---

# 39. Wait-Free

Wait-free predstavlja još jaču garanciju.

Svaka operacija mora završiti u ograničenom broju koraka.

Konceptualno:

```text
G1 ─────────────► done
G2 ─────────────► done
G3 ─────────────► done
```

nema starvation-a u smislu neograničenog broja retry-ja.

Wait-free algoritmi su izuzetno zahtevni za projektovanje.

Zbog toga:

```text
wait-free
```

nije default izbor za običan Go application code.

---

# 40. Why Lock-Free Is Difficult

Na prvi pogled:

```text
CAS loop
```

deluje jednostavno.

U realnosti mora se razmišljati o:

```text
ABA
memory reclamation
contention
starvation
false sharing
ordering
progress guarantees
invariants
```

Zbog toga lock-free programming treba posmatrati kao **specijalizovanu engineering discipline**, a ne kao jednostavnu zamenu za `sync.Mutex`.

---

# 41. The ABA Problem

Jedan od poznatijih problema CAS algoritama je:

```text
ABA problem
```

Pretpostavimo da goroutine pročita:

```text
A
```

zatim druga goroutine promeni:

```text
A → B → A
```

Prva goroutine sada vidi:

```text
A
```

i može pogrešno zaključiti:

> "State nije promenjen."

Vizuelno:

```text
G1: read A
        │
        │
        ▼
       A
        ▲
        │
G2: A → B → A
        │
        ▼
G1: CAS(A, C)
```

Problem je što je vrednost ponovo `A`, ali istorija state-a nije ista.

---

# 42. ABA Prevention

Mogući pristupi zavise od algoritma.

Jedan koncept je dodavanje version/tag information:

```text
(value, version)
```

umesto:

```text
value
```

Na primer:

```text
(A, 1)
   │
   ▼
(B, 2)
   │
   ▼
(A, 3)
```

Iako je vrednost ponovo `A`, verzija više nije ista.

Tada CAS može otkriti da je state u međuvremenu menjan.

---

# 43. Lock-Free Data Structures

Tipični lock-free data structures uključuju:

```text
lock-free stack
lock-free queue
ring buffer
atomic state machine
single-producer/single-consumer queue
```

Njihova implementacija često koristi:

```text
atomic load
atomic store
CAS
```

ali correctness zavisi od mnogo više od samog CAS-a.

---

# 44. Lock-Free Stack — Conceptual Model

Stack ima:

```text
head
 │
 ▼
Node
 │
 ▼
Node
 │
 ▼
Node
```

Push:

```text
new node
    │
    ▼
new.next = oldHead
    │
    ▼
CAS(head, oldHead, new)
```

Ako CAS ne uspe:

```text
another goroutine changed head
```

algoritam ponavlja pokušaj.

---

# 45. Lock-Free Queue

Queue je složeniji.

Konceptualno:

```text
head ───────────────►
                     Node → Node → Node
                                      ◄──────── tail
```

Producer i consumer mogu istovremeno da menjaju različite delove strukture.

Ali correctness zahteva precizno definisanje:

```text
head semantics
tail semantics
publication
node visibility
empty-state detection
memory reclamation
```

Zbog toga production lock-free queue ne treba implementirati "od nule" bez vrlo jasnog razloga.

---

# 46. Memory Reclamation

Jedan od najtežih problema lock-free struktura nije samo:

```text
How do we update a pointer atomically?
```

već:

```text
When is it safe to reclaim the old object?
```

Primer:

```text
G1 ──► reads node A
              │
G2 ──► removes A
              │
              ▼
          reclaim A
              │
              ▼
G1 ──► accesses A
```

Ako se objekat prerano oslobodi ili reciklira, nastaje ozbiljan correctness problem.

Go garbage collector značajno menja prirodu ovog problema u odnosu na manual-memory-management jezike, ali ne uklanja sve moguće lifetime i logical-reclamation probleme.

---

# 47. Lock-Free Does Not Mean Faster

Jedna od najvažnijih lekcija:

```text
lock-free ≠ faster
```

Lock-free implementacija može biti sporija zbog:

```text
CAS retries
cache-line bouncing
memory ordering
contention
false sharing
complexity
```

Na primer:

```text
Mutex
   │
   ▼
short critical section
   │
   ▼
very low contention
```

može biti mnogo brži od:

```text
CAS loop
   │
   ▼
thousands of retries
```

---

# 48. Mutex vs Atomic — Correct Decision

Praktični decision tree:

```text
Need shared state?
       │
       ├── No ──► no synchronization
       │
       └── Yes
            │
            ├── Simple scalar state?
            │       │
            │       └──► Atomic may fit
            │
            └── Complex invariant?
                    │
                    └──► Mutex may fit
```

Zatim:

```text
Benchmark
   │
   ▼
Does synchronization dominate?
   │
   ├── No ──► keep simpler design
   │
   └── Yes
         │
         ▼
   investigate alternatives
```

---

# 49. Worker Pools

Sledeća velika oblast je:

```text
05-worker-pools.md
```

Worker pool rešava problem:

> Kako ograničiti broj istovremeno aktivnih workers?

Umesto:

```text
jobs
 │
 ├──► goroutine
 ├──► goroutine
 ├──► goroutine
 ├──► ...
 └──► unlimited
```

koristi se:

```text
jobs
 │
 ▼
queue
 │
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker N
```

---

# 50. Why Worker Pools Exist

Worker pool može kontrolisati:

```text
CPU usage
memory usage
database connections
HTTP connections
downstream load
goroutine count
```

Najvažnija osobina je:

```text
bounded concurrency
```

Ako imamo:

```text
N = 16
```

onda sistem ne treba da ima više od 16 aktivnih workers koji izvršavaju dati workload.

---

# 51. Worker Pool Components

Tipičan worker pool ima:

```text
                 Producer
                    │
                    ▼
              Jobs Channel
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Worker 1    Worker 2    Worker N
        │           │           │
        └───────────┼───────────┘
                    ▼
                 Results
```

Komponente:

```text
job source
job queue
workers
result handling
cancellation
shutdown
error propagation
```

---

# 52. Worker Pool Lifecycle

Worker pool mora imati jasan lifecycle:

```text
START
  │
  ▼
RUNNING
  │
  ├── jobs available
  │
  ├── cancellation
  │
  ├── shutdown
  │
  └── fatal error
          │
          ▼
       STOPPING
          │
          ▼
        STOPPED
```

Najveća greška je napraviti worker pool koji zna samo:

```text
start
```

ali ne zna:

```text
stop
```

---

# 53. Worker Pool Ownership

Treba jasno definisati:

```text
Who creates workers?
Who owns the jobs channel?
Who closes the jobs channel?
Who cancels the workers?
Who waits for completion?
Who consumes results?
```

Jedan mogući ownership model:

```text
Pool
 │
 ├── owns workers
 ├── owns cancellation
 ├── owns WaitGroup
 └── owns queue
```

Korisnik pool-a tada koristi jednostavan API:

```text
Submit
Wait
Shutdown
```

---

# 54. Bounded Queue

Worker pool može koristiti bounded channel:

```go
jobs := make(chan Job, 100)
```

što daje:

```text
capacity = 100
```

Ako je queue puna:

```text
producer
   │
   ▼
queue full
   │
   ▼
block / reject / drop / backpressure
```

Ovo je važna architectural odluka.

---

# 55. Queue Full Policies

Kada je queue puna, sistem može:

### Block

```text
producer waits
```

Prednost:

* prirodan backpressure.

Mana:

* producer može dugo čekati.

### Reject

```text
return ErrQueueFull
```

Prednost:

* latency je bounded.

Mana:

* work se odbacuje.

### Drop

```text
discard job
```

Može biti prihvatljivo za:

* metrics;
* telemetry;
* best-effort events.

### Spill

```text
queue full
   │
   ▼
external durable queue
```

korisno je kada work mora biti sačuvan.

---

# 56. Worker Pool Sizing

Broj workers ne treba birati proizvoljno.

Za CPU-bound workload:

```text
workers ≈ available CPU parallelism
```

može biti dobra početna tačka.

Za I/O-bound workload:

```text
workers > CPU count
```

može imati smisla jer workers često čekaju I/O.

Ali stvarni optimalni broj zavisi od:

```text
latency
CPU cost
I/O wait
downstream limits
memory
contention
batch size
queueing
```

---

# 57. Worker Pool Failure Modes

Tipični problemi:

```text
goroutine leak
dead worker
blocked submitter
unconsumed results
unbounded queue
worker panic
shutdown race
lost error
```

Posebno opasan je:

```text
result channel
```

koji niko ne čita.

Primer:

```text
worker
  │
  ▼
send result
  │
  ▼
blocked forever
```

Ako je channel unbuffered i consumer je nestao, worker može ostati zauvek blokiran.

---

# 58. Worker Panic

Production worker pool treba da ima jasno definisanu politiku za panic.

Moguće strategije:

```text
panic
 │
 ├── recover locally
 ├── log
 ├── report error
 └── restart worker
```

ili:

```text
panic
 │
 ▼
fail entire operation
```

Izbor zavisi od semantics-a workload-a.

Nije univerzalno ispravno "uvek recover-ovati".

---

# 59. Pipelines

Sledeća oblast:

```text
06-pipelines.md
```

Pipeline predstavlja obradu kroz više concurrency stages:

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

Svaki stage može biti implementiran kao jedna ili više goroutines.

---

# 60. Pipeline Example

Konceptualno:

```text
Files
  │
  ▼
Read
  │
  ▼
Parse
  │
  ▼
Transform
  │
  ▼
Persist
```

Concurrency može biti:

```text
Read workers
    │
    ▼
Parse workers
    │
    ▼
Transform workers
    │
    ▼
Persist workers
```

---

# 61. Pipeline Backpressure

Ako:

```text
Stage 1 = 1000 ops/sec
Stage 2 = 100 ops/sec
```

onda će queue između njih rasti.

Bez bounded queue-a:

```text
Stage 1
  │
  ▼
unbounded buffer
  │
  ▼
memory growth
```

Sa bounded queue-om:

```text
Stage 1
  │
  ▼
bounded buffer
  │
  ▼
Stage 2
```

Stage 1 će na kraju biti throttled.

To je backpressure.

---

# 62. Pipeline Cancellation

Svaki pipeline stage mora znati kada da prestane.

Tipičan model:

```text
ctx
 │
 ├── stage 1
 ├── stage 2
 ├── stage 3
 └── stage 4
```

Ako stage 3 više ne može da nastavi:

```text
stage 3 error
     │
     ▼
cancel context
     │
 ├── stage 1 stops
 ├── stage 2 stops
 └── stage 4 stops
```

Bez ovoga pipeline može ostaviti upstream goroutines blocked.

---

# 63. Pipeline Channel Ownership

Važno pravilo:

> Onaj ko proizvodi vrednosti najčešće je odgovoran za zatvaranje output channel-a.

Tipična struktura:

```text
stage
 │
 ├── receive input
 ├── produce output
 └── close output
```

Consumer:

```text
for value := range output {
    ...
}
```

može tada znati kada je stage završen.

---

# 64. Pipeline Error Propagation

Pipeline mora definisati:

```text
What happens if one stage fails?
```

Moguće:

```text
Stage 2
   │
   ▼
error
   │
   ├── cancel pipeline
   ├── drain / stop upstream
   └── report error
```

Bez definisane error propagation strategije može doći do:

```text
goroutine leaks
blocked channels
partial results
silent failures
```

---

# 65. Fan-Out / Fan-In

Napredniji pipeline pattern:

```text
                 ┌── Worker 1 ──┐
                 │              │
Input ───────────┼── Worker 2 ──┼──► Output
                 │              │
                 └── Worker 3 ──┘
```

Ovo kombinuje:

```text
fan-out
```

i:

```text
fan-in
```

Fan-out:

```text
one input stream
       │
       ├── worker 1
       ├── worker 2
       └── worker 3
```

Fan-in:

```text
worker 1 ─┐
worker 2 ─┼──► one output stream
worker 3 ─┘
```

---

# 66. Fan-Out Correctness

Fan-out mora rešiti:

```text
Who owns input?
Who closes input?
How are workers stopped?
What happens if one worker fails?
```

Fan-in mora rešiti:

```text
Who closes output?
When are all workers finished?
How are results synchronized?
```

Čest pattern je:

```text
worker WaitGroup
       │
       ▼
all workers done
       │
       ▼
close merged output
```

---

# 67. Semaphore Pattern

Sledeća oblast:

```text
07-semaphore-pattern.md
```

Semaphore ograničava broj concurrent operations.

Konceptualno:

```text
capacity = 3

┌───────────────┐
│ token │ token │ token │
└───────────────┘
```

Goroutine mora uzeti token:

```text
acquire
   │
   ▼
work
   │
   ▼
release
```

---

# 68. Semaphore vs Worker Pool

Ova dva pattern-a rešavaju sličan problem, ali nisu identični.

### Worker Pool

```text
fixed workers
     │
     ▼
workers pull jobs
```

### Semaphore

```text
many goroutines
      │
      ▼
limited concurrent section
```

Primer:

```text
for _, item := range items {
    go func() {
        acquire()
        defer release()

        process(item)
    }()
}
```

Ovde može postojati mnogo goroutines, ali samo ograničen broj istovremeno izvršava critical workload.

---

# 69. Semaphore Use Cases

Semaphore je koristan za ograničavanje:

```text
database queries
HTTP calls
file operations
CPU-heavy sections
external API requests
resource acquisition
```

Na primer:

```text
API limit = 20 concurrent requests
```

može se predstaviti kao:

```text
semaphore capacity = 20
```

---

# 70. Semaphore and Resource Limits

Concurrency limit treba često vezati za stvarni bottleneck.

Ako downstream database podržava:

```text
max 100 concurrent queries
```

a application kreira:

```text
10,000 concurrent queries
```

to nije efikasan concurrency model.

Semaphore može napraviti:

```text
10,000 logical tasks
        │
        ▼
semaphore(100)
        │
        ▼
max 100 active DB operations
```

---

# 71. Concurrency Patterns

Sledeća oblast:

```text
08-concurrency-patterns.md
```

objedinjuje reusable patterns.

Najvažniji:

```text
worker pool
pipeline
fan-out
fan-in
semaphore
bounded concurrency
producer-consumer
broadcast
orchestration
scatter-gather
```

Pattern nije cilj sam po sebi.

Pattern je odgovor na određenu strukturu problema.

---

# 72. Producer-Consumer

Osnovni model:

```text
Producer
   │
   ▼
Queue
   │
   ▼
Consumer
```

Concurrency nastaje zato što producer i consumer mogu napredovati nezavisno.

Ako queue ima capacity:

```text
Producer
   │
   ▼
[ 1 ][ 2 ][ 3 ][ 4 ]
               │
               ▼
            Consumer
```

queue predstavlja određeni nivo decoupling-a.

---

# 73. Scatter-Gather

Scatter-gather predstavlja:

```text
Request
   │
   ├──► Service A
   ├──► Service B
   └──► Service C
        │
        ▼
     gather
        │
        ▼
     response
```

Ovaj pattern je čest u service orchestration-u.

Ali mora rešiti:

```text
timeout
partial failure
cancellation
result aggregation
concurrency limit
```

---

# 74. Partial Failure

Ako:

```text
A = success
B = success
C = timeout
```

šta se dešava?

Moguće semantics:

```text
fail entire request
```

ili:

```text
return partial result
```

ili:

```text
fallback
```

ili:

```text
retry C
```

Concurrency architecture ne može biti odvojena od business semantics-a.

---

# 75. Broadcast

Broadcast pattern šalje isti event ka više consumers:

```text
                 ┌──► Consumer A
                 │
Event ───────────┼──► Consumer B
                 │
                 └──► Consumer C
```

Ovo nije isto što i običan channel sa više receivers, gde svaki element obično konzumira samo jedan receiver.

Broadcast zahteva drugačiju architecture.

---

# 76. Broadcast Lifecycle

Broadcast system mora rešiti:

```text
subscriber registration
subscriber removal
slow subscriber
buffering
backpressure
shutdown
```

Posebno je opasan slow consumer:

```text
Producer
   │
   ▼
Broadcast
 ┌─┼───────────────┐
 ▼ ▼               ▼
Fast Fast           Slow
                    │
                    ▼
                 blocks
```

Jedna od mogućih politika je izolovanje sporog subscriber-a.

---

# 77. Concurrency Anti-Patterns

Sledeća oblast:

```text
09-concurrency-anti-patterns.md
```

je podjednako važna kao patterns.

Najčešći anti-pattern-i:

```text
unbounded goroutines
time.Sleep synchronization
shared mutable state everywhere
overuse of Mutex
overuse of atomics
goroutine without owner
ignored cancellation
ignored errors
unbounded queues
channel misuse
premature parallelism
```

---

# 78. Anti-Pattern — Sleep as Synchronization

Loše:

```go
go doWork()

time.Sleep(time.Second)

readResult()
```

Problem:

```text
maybe enough
maybe too short
maybe unnecessarily long
```

Ispravno:

```text
WaitGroup
channel
context
explicit synchronization
```

---

# 79. Anti-Pattern — Goroutine per Item

Naivni pattern:

```go
for _, item := range items {
    go process(item)
}
```

Može biti sasvim validan za mali, bounded workload.

Ali ako je:

```text
items = unbounded
```

onda:

```text
goroutines = unbounded
```

što može destabilizovati sistem.

Bolje je često:

```text
bounded worker pool
```

ili:

```text
semaphore
```

---

# 80. Anti-Pattern — Global Mutex

Ako cela aplikacija koristi:

```text
global.Lock()
```

za skoro svaki shared operation:

```text
G1 ─┐
G2 ─┤
G3 ─┼──► global mutex
G4 ─┤
G5 ─┘
```

sistem može izgubiti concurrency.

Global lock može pretvoriti:

```text
concurrent program
```

u:

```text
serialized program
```

---

# 81. Anti-Pattern — Atomic Everything

Suprotna greška je:

```text
"Mutex is slow, use atomic everywhere."
```

Posledice:

```text
complex invariants
CAS loops
hard-to-review code
subtle ordering bugs
difficult debugging
```

Jednostavan `Mutex` je često bolji engineering choice.

---

# 82. Anti-Pattern — Forgotten Cancellation

Background worker:

```text
go worker()
```

bez cancellation:

```text
request ends
   │
   ▼
worker continues
```

Repeated thousands of times:

```text
worker
worker
worker
worker
...
```

može dovesti do goroutine leak-a.

---

# 83. Anti-Pattern — Ignoring Backpressure

Ako producer radi:

```text
10,000 jobs/sec
```

a consumer:

```text
100 jobs/sec
```

onda:

```text
queue growth = 9,900 jobs/sec
```

Ako queue nije bounded, memorija može rasti bez ograničenja.

Backpressure mora biti deo architecture-a.

---

# 84. Anti-Pattern — Premature Parallelism

Loš optimization strategy:

```text
sequential code
    │
    ▼
add goroutines everywhere
    │
    ▼
hope for performance
```

Ispravno:

```text
measure
   │
   ▼
identify bottleneck
   │
   ▼
parallelize suitable work
   │
   ▼
benchmark
   │
   ▼
verify
```

---

Do ove tačke Module 4 je povezao:

```text
Atomic Operations
       │
       ▼
CAS
       │
       ▼
Lock-Free Algorithms
       │
       ▼
Progress Guarantees
       │
       ▼
Worker Pools
       │
       ▼
Pipelines
       │
       ▼
Semaphores
       │
       ▼
Concurrency Patterns
       │
       ▼
Anti-Patterns
```

Glavna ideja je:

> **Concurrency architecture treba da kontroliše state, work, lifecycle i resource consumption.**

---

# 86. Advanced Concurrency Decision Tree

Kada dizajniraš novi concurrent subsystem, možeš početi sledećim redosledom:

```text
                    Concurrent Work
                          │
                          ▼
                 Is work bounded?
                    │           │
                   No          Yes
                    │           │
                    ▼           ▼
              Add bounds     continue
                    │
                    ▼
              Shared state?
                │       │
               No      Yes
                │       │
                ▼       ▼
              continue  invariant?
                         │
                    ┌────┴────┐
                    ▼         ▼
                 simple     complex
                    │         │
                    ▼         ▼
                 atomic     mutex
```

Zatim:

```text
Need multiple stages?
        │
        ├── Yes → pipeline
        │
        └── No
             │
             ▼
Need fixed workers?
        │
        ├── Yes → worker pool
        │
        └── No
             │
             ▼
Need concurrency limit?
        │
        ├── Yes → semaphore
        │
        └── No → direct concurrency
```

---

# 87. Senior-Level Questions

Pre nastavka, čitalac treba da može da odgovori:

### Question 1

Koja je razlika između:

```text
obstruction-free
lock-free
wait-free
```

### Question 2

Zašto lock-free algoritam može imati visok contention?

### Question 3

Šta je ABA problem?

### Question 4

Zašto CAS nije dovoljan za kompleksan invariant koji obuhvata više nezavisnih vrednosti?

### Question 5

Zašto worker pool predstavlja bounded concurrency?

### Question 6

Koja je razlika između worker pool-a i semaphore pattern-a?

### Question 7

Šta se dešava kada jedan pipeline stage radi sporije od prethodnog?

### Question 8

Ko zatvara channel u pipeline-u i zašto?

### Question 9

Kako cancellation treba da propagira kroz pipeline?

### Question 10

Zašto unbounded concurrency može oboriti downstream dependency čak i kada application CPU nije zasićen?

---

# 88. Engineering Principles

### Principle 9

> **Lock-free is a progress guarantee, not a performance guarantee.**

### Principle 10

> **A simpler lock-based design is often preferable to a complex lock-free implementation.**

### Principle 11

> **Bound concurrency around the resource that actually limits the system.**

### Principle 12

> **Every pipeline stage needs a lifecycle and failure strategy.**

### Principle 13

> **Backpressure is a correctness and stability mechanism, not merely a performance optimization.**

### Principle 14

> **Concurrency patterns must be selected according to workload semantics.**

### Principle 15

> **Every concurrent abstraction needs an explicit shutdown story.**

### Principle 16

> **Never introduce parallelism merely because the language makes goroutines cheap.**

---

# 89. Transition

Sledeći deo će nastaviti od concurrency patterns ka **performance engineering-u i naprednoj concurrency arhitekturi**.

Biće obrađene sledeće oblasti:

```text
10-performance-and-profiling.md
11-advanced-concurrency-architecture.md
12-distributed-concurrency.md
```

Fokus će biti na:

```text
CPU profiling
memory profiling
goroutine profiling
mutex profiling
block profiling
contention
benchmarking
latency
throughput
work stealing
cache effects
false sharing
system architecture
distributed locks
distributed coordination
idempotency
deduplication
```

Posebna pažnja biće posvećena prelazu:

```text
local concurrency
       │
       ▼
process-level concurrency
       │
       ▼
distributed concurrency
```

jer synchronization primitives poput `Mutex`, `atomic` i channels više nisu dovoljne kada se state nalazi na više procesâ ili mašina.

---

# Module 4 — Advanced Go Concurrency

## Performance Engineering, Profiling, Architecture & Distributed Concurrency

> **Continuation of:** `golang/concurrency/docs/module-4/README.md - Deo #2/5`

---

# 90. Concurrency Performance Engineering

Sledeća velika oblast Module 4 je:

```text
10-performance-and-profiling.md
```

Do sada smo se bavili pitanjem:

```text
Is the concurrent program correct?
```

Sada prelazimo na:

```text
Is the concurrent program efficient?
```

A zatim na još važnije pitanje:

```text
Where is the actual bottleneck?
```

Concurrent program može biti:

* race-free;
* deadlock-free;
* logically correct;

a ipak biti veoma spor.

Zbog toga correctness i performance moraju biti analizirani odvojeno.

---

# 91. Concurrency Performance Model

Performanse concurrent sistema mogu se posmatrati kroz nekoliko ključnih dimenzija:

```text
                    Performance
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Throughput      Latency       Resource
          │              │              │
          ▼              ▼              ▼
       work/sec        p50/p99        CPU
                                      Memory
                                      Goroutines
                                      Locks
                                      I/O
```

Jedna optimizacija može poboljšati jednu dimenziju, a pogoršati drugu.

Na primer:

```text
more workers
    │
    ├──► higher throughput
    │
    └──► higher contention
              │
              ▼
        worse latency
```

Zato se concurrency ne optimizuje jednom metrikom.

---

# 92. Throughput vs Latency

**Throughput** predstavlja količinu rada koju sistem završi u određenom vremenu.

Primer:

```text
10,000 requests / second
```

**Latency** predstavlja koliko dugo jedan request traje.

Primer:

```text
p50 = 10 ms
p95 = 25 ms
p99 = 80 ms
```

Sistem može imati:

```text
high throughput
+
high tail latency
```

ili:

```text
low latency
+
low throughput
```

Zavisi od workload-a i arhitekture.

---

# 93. Tail Latency

Concurrent systems često imaju problem sa:

```text
p95
p99
p99.9
```

jer mali broj sporih operations može značajno uticati na korisničko iskustvo.

Na primer:

```text
p50 = 10ms
p90 = 15ms
p95 = 20ms
p99 = 150ms
p99.9 = 800ms
```

Average latency:

```text
≈ 15ms
```

može izgledati odlično.

Ali realni korisnici mogu povremeno doživeti:

```text
800ms
```

Zbog toga concurrent systems treba analizirati kroz distribuciju latency-ja, a ne samo kroz prosečnu vrednost.

---

# 94. Amdahl's Law

Parallelism ima fundamentalno ograničenje.

Ako deo programa mora biti izvršen sekvencijalno:

```text
serial work
+
parallel work
```

dodavanje više CPU cores neće beskonačno povećavati performanse.

Amdahl's Law konceptualno kaže:

```text
speedup = 1 / (S + P/N)
```

gde su:

```text
S = serial fraction
P = parallel fraction
N = number of processors
```

Ako je:

```text
S = 0.10
P = 0.90
```

čak i sa ogromnim brojem procesora speedup je ograničen.

To je razlog zašto:

> "Dodaj još goroutines" nije performance strategy.

---

# 95. Concurrency vs Parallelism

Važno je razlikovati:

```text
Concurrency
```

od:

```text
Parallelism
```

Concurrency:

```text
multiple tasks in progress
```

Parallelism:

```text
multiple tasks executing simultaneously
```

Na jednom CPU core-u može postojati:

```text
concurrency
```

bez stvarnog:

```text
parallel execution
```

Na više cores moguće je imati oba.

---

# 96. Goroutines Are Not Free

Goroutines su veoma jeftine u odnosu na OS threads, ali nisu besplatne.

Svaka goroutine zahteva:

```text
stack
runtime bookkeeping
scheduler state
synchronization
```

Veliki broj goroutines može povećati:

```text
memory usage
scheduler overhead
GC pressure
contention
latency
```

Zato:

```text
cheap ≠ free
```

---

# 97. Measuring Before Optimizing

Pravilo:

```text
measure → identify → change → measure
```

ne:

```text
guess → optimize → guess again
```

Tipičan workflow:

```text
1. Define performance target
2. Build representative benchmark
3. Measure baseline
4. Profile
5. Identify bottleneck
6. Change implementation
7. Benchmark again
8. Verify correctness
```

---

# 98. Go Benchmarking

Go benchmark može meriti:

```text
ns/op
B/op
allocs/op
```

što omogućava poređenje različitih concurrency implementations.

Na primer, korisno je porediti:

```text
Mutex
vs
Atomic
vs
Channel
```

ali samo za isti workload i iste correctness guarantees.

---

# 99. Benchmark Pitfalls

Benchmark može biti pogrešan.

Problemi uključuju:

```text
compiler optimizations
dead-code elimination
unrealistic workload
too-small input
too-large input
missing contention
missing parallelism
CPU frequency variation
GC effects
synchronization artifacts
```

Posebno kod concurrent benchmarks-a potrebno je obezbediti da benchmark zaista generiše workload koji želimo da merimo.

---

# 100. `go test -bench`

Osnovni benchmark workflow:

```text
go test -bench=.
```

Za memory allocation:

```text
go test -bench=. -benchmem
```

Za konkurentni benchmark:

```text
b.RunParallel(...)
```

može simulirati konkurentne pozive.

Ali broj goroutines i workload treba pažljivo kontrolisati.

---

# 101. Profiling

Go ekosistem pruža veoma snažne profiling mogućnosti.

Najvažniji profili uključuju:

```text
CPU
heap
allocations
goroutine
mutex
block
threadcreate
```

Mentalni model:

```text
Application
    │
    ▼
Observe
    │
    ├── CPU
    ├── Memory
    ├── Goroutines
    ├── Blocking
    └── Mutex contention
```

---

# 102. CPU Profiling

CPU profiler odgovara na pitanje:

> Gde program troši CPU vreme?

Možemo otkriti:

```text
hot functions
hot call paths
unexpected CPU work
scheduler overhead
serialization bottlenecks
```

Ako concurrency optimizacija smanji lock time, ali CPU usage eksplodira zbog CAS retries, CPU profile može to pokazati.

---

# 103. Goroutine Profiling

Goroutine profile pomaže da otkrijemo:

```text
goroutine leaks
blocked goroutines
unexpected goroutine growth
waiting states
```

Mentalni model:

```text
goroutines
   │
   ├── running
   ├── runnable
   ├── waiting
   ├── blocked
   └── leaked
```

Ako broj goroutines kontinuirano raste:

```text
100
 ↓
500
 ↓
2,000
 ↓
10,000
```

to je signal da lifecycle nije pravilno kontrolisan.

---

# 104. Block Profiling

Block profile je posebno važan za concurrency.

On pomaže da se pronađu mesta gde goroutines provode vreme čekajući.

Tipični izvori:

```text
channel operations
sync primitives
select
network waits
other blocking operations
```

Ako sistem ima visoku latency, a CPU nije zasićen:

```text
CPU = 30%
latency = high
```

blocking/queueing može biti jedan od kandidata.

---

# 105. Mutex Profiling

Mutex profile pomaže u otkrivanju contention-a.

Primer:

```text
G1 ───── Lock ───────────── Unlock
G2 ── waiting ────────────────┘
G3 ── waiting ────────────────┘
G4 ── waiting ────────────────┘
```

Ako mnogo goroutines čeka isti lock:

```text
contention hotspot
```

može biti dominantan bottleneck.

---

# 106. Contention

Contention nastaje kada veliki broj execution units pokušava da koristi isti resource.

Primer:

```text
              shared resource
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
       G1          G2          G3
        │           │           │
        └──────── lock ─────────┘
```

Više workers ne znači nužno veći throughput.

Može doći do:

```text
more workers
   │
   ▼
more contention
   │
   ▼
less useful work
```

---

# 107. Contention Collapse

Jedan od opasnijih fenomena:

```text
load increases
     │
     ▼
more goroutines
     │
     ▼
more contention
     │
     ▼
less throughput
     │
     ▼
more queueing
     │
     ▼
higher latency
     │
     ▼
more concurrent requests
```

Sistem može ući u negativnu povratnu spregu.

Ovo je razlog zašto bounded concurrency i backpressure imaju stabilizacionu ulogu.

---

# 108. False Sharing

Napredna performance tema je:

```text
false sharing
```

Dve nezavisne vrednosti mogu slučajno deliti isti CPU cache line.

Konceptualno:

```text
Cache Line
┌─────────────────────────────┐
│ counter A │ counter B       │
└─────────────────────────────┘
```

Ako:

```text
G1 frequently writes A
G2 frequently writes B
```

cache coherence traffic može biti nepotrebno visok iako logički nema shared state-a između A i B.

---

# 109. Cache-Line Contention

Na modernim multicore sistemima cache coherence može biti veoma značajan.

Model:

```text
CPU 1
 │
 ▼
cache line
 │
 ▼
write A

CPU 2
 │
 ▼
same cache line
 │
 ▼
write B
```

Iako su:

```text
A != B
```

CPU caches moraju koordinisati ownership nad cache line-om.

Rezultat može biti:

```text
cache-line bouncing
```

što degradira performance.

---

# 110. Atomics and Cache Traffic

Atomic operations nisu samo logička synchronization primitives.

One imaju fizičku posledicu:

```text
atomic operation
      │
      ▼
cache coherence
      │
      ▼
memory subsystem
```

Na hot shared variable:

```text
atomic counter
```

može postati bottleneck.

Zato je ponekad bolji pattern:

```text
per-worker local counter
       │
       ▼
periodic aggregation
```

nego:

```text
all workers
      │
      ▼
one global atomic counter
```

---

# 111. Sharded State

Sharding može smanjiti contention.

Umesto:

```text
             one lock
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      G1       G2       G3
```

koristimo:

```text
        shard 1   shard 2   shard 3
           │         │         │
          G1        G2        G3
```

State se deli na više nezavisnih partitions.

To može povećati concurrency.

---

# 112. Lock Striping

Sličan koncept je:

```text
lock striping
```

Umesto jednog lock-a:

```text
globalLock
```

koristimo:

```text
lock[0]
lock[1]
lock[2]
...
lock[N]
```

Key se mapira na određeni lock:

```text
hash(key) % N
```

Tada unrelated keys mogu biti obrađeni paralelno.

---

# 113. Advanced Concurrency Architecture

Sledeća oblast je:

```text
11-advanced-concurrency-architecture.md
```

Ovde concurrency više nije samo pitanje individualnog goroutine-a.

Posmatramo ceo sistem:

```text
                 System
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Ingress      Workers     Storage
        │           │           │
        ▼           ▼           ▼
      Queue      Pipeline    Database
```

Svaka komponenta ima:

```text
capacity
latency
failure mode
ownership
lifecycle
```

---

# 114. Concurrency Boundary

Dobar architecture definiše gde concurrency počinje i završava.

Na primer:

```text
HTTP Request
     │
     ▼
Handler
     │
     ▼
Service
     │
     ▼
Worker Pool
     │
     ▼
Repository
     │
     ▼
Database
```

Nije potrebno da svaka layer sama kreira goroutines.

Concurrency ownership treba biti eksplicitan.

---

# 115. Structured Concurrency

Važan architectural princip jeste:

> Concurrent work treba da ima jasan parent-child lifecycle.

Konceptualno:

```text
Parent Task
    │
    ├── Child A
    ├── Child B
    └── Child C
```

Ako parent završi ili otkaže operation:

```text
cancel parent
      │
      ├── cancel A
      ├── cancel B
      └── cancel C
```

Go `context.Context` je centralni mehanizam za propagaciju cancellation-a kroz API granice.

---

# 116. Context Is Not a General State Container

`context.Context` treba koristiti za:

```text
cancellation
deadlines
request-scoped values
```

Ne treba ga koristiti kao:

```text
global dependency container
mutable application state
configuration store
random parameter bag
```

Concurrency architecture zahteva jasnu separation of concerns.

---

# 117. Admission Control

Jedan od najvažnijih production pattern-a je:

```text
admission control
```

Sistem ne treba prihvatiti više work-a nego što može stabilno da obradi.

Model:

```text
Incoming Load
      │
      ▼
Admission Control
      │
 ┌────┴────┐
 ▼         ▼
Accept    Reject
 │
 ▼
Bounded System
```

Moguće politike:

```text
queue
reject
shed load
rate limit
priority
timeout
```

---

# 118. Load Shedding

Ako sistem postane overloaded:

```text
incoming work
      │
      ▼
capacity exceeded
      │
      ▼
load shedding
```

bolje je odbiti deo work-a nego dozvoliti:

```text
queue growth
   ↓
memory growth
   ↓
latency growth
   ↓
timeouts
   ↓
retry storm
   ↓
system failure
```

Ovo je centralni concept resilient concurrency architecture-a.

---

# 119. Retry Storm

Concurrency i retries mogu proizvesti veoma opasan feedback loop.

Primer:

```text
Service A
   │
   ▼
Service B overloaded
   │
   ▼
timeout
   │
   ▼
retry
   │
   ▼
more requests to B
   │
   ▼
more overload
```

To može dovesti do:

```text
retry storm
```

Zato retries treba kombinovati sa:

```text
timeouts
backoff
jitter
retry limits
circuit breakers
load shedding
```

---

# 120. Concurrency and Backpressure

Backpressure treba da postoji između svih relevantnih stages:

```text
Producer
   │
   ▼
Queue
   │
   ▼
Worker
   │
   ▼
Downstream
```

Ako downstream uspori:

```text
downstream slower
      │
      ▼
worker throughput decreases
      │
      ▼
queue grows
      │
      ▼
producer slows/rejects
```

To je stabilan feedback mechanism.

---

# 121. Distributed Concurrency

Sledeća oblast:

```text
12-distributed-concurrency.md
```

uvodi fundamentalnu promenu.

Do sada imamo:

```text
single process
```

gde možemo koristiti:

```text
Mutex
atomic
channel
WaitGroup
```

U distribuiranom sistemu imamo:

```text
Machine A          Machine B
    │                  │
    └──── Network ─────┘
```

Nema shared memory.

Zato:

```text
sync.Mutex
```

ne može zaštititi state između dve mašine.

---

# 122. Local vs Distributed Synchronization

Lokalno:

```text
G1 ─┐
G2 ─┼──► Mutex ──► shared memory
G3 ─┘
```

Distribuirano:

```text
Process A
    │
    ▼
 Network
    │
    ▼
Process B
```

Potrebni su drugi mehanizmi:

```text
distributed lock
lease
consensus
database transaction
atomic storage operation
message broker
coordination service
```

---

# 123. Distributed Lock

Distributed lock omogućava koncept:

```text
resource
   │
   ▼
distributed lock
   │
 ┌─┴───────────┐
 ▼             ▼
Node A       Node B
```

Ali distributed lock je mnogo složeniji od lokalnog `Mutex`.

Problemi uključuju:

```text
network partitions
process crashes
timeouts
lease expiration
clock issues
split brain
stale ownership
lock recovery
```

---

# 124. Lease

Umesto permanentnog lock-a, sistem može koristiti:

```text
lease
```

Koncept:

```text
Acquire lease
     │
     ▼
valid until T
     │
     ├── renew
     │
     ▼
expiration
```

Ako owner nestane:

```text
lease expires
      │
      ▼
another node may acquire
```

Ovo omogućava recovery nakon process failure-a.

---

# 125. Distributed Lock Is Not Magic

Ako Node A dobije lock:

```text
A owns lock
```

ali izgubi network connectivity:

```text
A ──X── coordinator
```

drugi node može zaključiti da je lease istekao:

```text
B acquires lock
```

A možda i dalje nastavi da radi.

Tada imamo:

```text
A thinks: "I own it"
B thinks: "I own it"
```

što je oblik split-brain / stale ownership problema.

Zbog toga distributed locking mora biti projektovan sa jasnim safety semantics.

---

# 126. Idempotency

Jedna od najvažnijih tehnika u distributed concurrency sistemima je:

```text
idempotency
```

Ako request može biti izvršen više puta:

```text
request
   │
   ├── attempt 1
   ├── retry
   └── attempt 2
```

system treba sprečiti neželjeni višestruki efekat.

Primer:

```text
payment_id = 12345
```

umesto:

```text
charge()
charge()
```

može se koristiti:

```text
charge(payment_id)
```

gde je `payment_id` idempotency key.

---

# 127. Deduplication

Distributed workers mogu dobiti isti posao više puta:

```text
Queue
 │
 ├── Worker A → job 42
 │
 └── Worker B → job 42
```

Sistem mora odlučiti:

```text
Can job 42 execute twice?
```

Ako ne:

```text
deduplication
```

mora postojati.

Mogući mehanizmi:

```text
unique database constraint
idempotency key
deduplication table
atomic state transition
distributed coordination
```

---

# 128. Exactly-Once Processing

"Exactly once" je veoma snažna tvrdnja.

U distribuiranom sistemu treba razlikovati:

```text
at-most-once
at-least-once
effectively-once
exactly-once
```

### At-most-once

```text
0 or 1 execution
```

### At-least-once

```text
1 or more executions
```

### Exactly-once

```text
exactly 1 execution
```

U praksi je često lakše postići:

```text
at-least-once delivery
+
idempotent processing
```

nego garantovati literalni exactly-once execution.

---

# 129. Distributed Work Queue

Tipična arhitektura:

```text
                  Queue
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Worker A  Worker B  Worker C
          │         │         │
          └─────────┼─────────┘
                    ▼
                 Storage
```

Problemi:

```text
worker crash
message visibility
acknowledgement
retry
duplicate delivery
poison message
dead-letter queue
```

To je distributed verzija worker pool problema.

---

# 130. Distributed Cancellation

Lokalno:

```text
context.Cancel()
```

može propagirati cancellation kroz process.

Distribuirano:

```text
Node A
  │
  ▼
Network
  │
  ▼
Node B
```

cancellation može biti:

```text
delayed
duplicated
lost
```

zbog network failure-a.

Zato distributed cancellation treba tretirati kao **best-effort signal**, a correctness ne sme zavisiti samo od toga da je cancellation stigao na vreme.

---

# 131. Timeouts Are Mandatory Boundaries

Distributed operations treba da imaju:

```text
deadline
```

jer network može:

```text
delay
drop
partition
stall
```

Bez timeout-a:

```text
request
   │
   ▼
network
   │
   X
   │
   ▼
wait forever
```

Sa deadline-om:

```text
request
   │
   ▼
deadline
   │
   ▼
bounded wait
```

---

# 132. Distributed Concurrency Failure Model

U lokalnom sistemu često razmatramo:

```text
goroutine
mutex
channel
panic
```

U distribuiranom sistemu dodajemo:

```text
process crash
machine failure
network partition
message duplication
message loss
delayed message
stale state
clock uncertainty
partial failure
```

Zbog toga distributed concurrency predstavlja zaseban nivo kompleksnosti.

---

# 133. Concurrency Architecture — Full Model

Module 4 sada može biti posmatran kroz:

```text
                      SYSTEM
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
           Local               Distributed
              │                   │
       ┌──────┼──────┐      ┌─────┼─────┐
       ▼      ▼      ▼      ▼     ▼     ▼
     Mutex  Atomic  Chan   Queue Lease Retry
       │      │      │      │     │     │
       └──────┼──────┘      └─────┼─────┘
              ▼                   ▼
          Memory Model        Failure Model
              │                   │
              └─────────┬─────────┘
                        ▼
                  Architecture
                        │
                        ▼
                   Performance
```

---

# 134. Performance + Architecture

Performance engineering ne treba posmatrati izolovano.

Na primer:

```text
more workers
```

može povećati:

```text
throughput
```

ali istovremeno:

```text
contention
memory
queueing
GC pressure
downstream load
```

Zato je cilj:

> **Optimal concurrency, not maximum concurrency.**

---

# 135. Capacity Planning

Concurrent system treba imati poznate granice.

Na primer:

```text
CPU:              8 cores
DB connections:   100
HTTP concurrency: 500
Queue capacity:   10,000
Workers:          32
```

Ovo omogućava:

```text
load
 │
 ▼
admission
 │
 ▼
bounded execution
 │
 ▼
bounded resource consumption
```

Umesto:

```text
load
 │
 ▼
unbounded goroutines
 │
 ▼
resource exhaustion
```

---

# 136. Concurrency Budget

Korisna architectural ideja je concurrency budget.

Na primer:

```text
service
 │
 ├── DB = 50
 ├── external API = 20
 ├── CPU workers = 8
 └── background jobs = 10
```

Svaki resource ima svoju granicu.

Ovo sprečava da jedna vrsta workload-a potroši sve dostupne resources.

---

# 137. Resource Isolation

Ako system ima:

```text
user requests
background jobs
maintenance tasks
```

ne treba nužno da svi koriste isti worker pool.

Loš model:

```text
                 one pool
              ┌────┴────┐
              ▼         ▼
           requests   jobs
```

Background jobs mogu zauzeti sve workers.

Bolji model:

```text
             service
          ┌─────┴─────┐
          ▼           ▼
      request pool   job pool
```

To predstavlja concurrency isolation.

---

# 138. Priority and Fairness

Ako postoji više workload classes:

```text
high priority
normal priority
low priority
```

sistem može koristiti:

```text
priority queues
separate pools
weighted scheduling
```

Ali treba paziti na starvation:

```text
high priority
high priority
high priority
...
```

koji može sprečiti low-priority posao da ikada dođe na red.

---

# 139. Starvation

Starvation nastaje kada goroutine ili task praktično ne dobija dovoljno prilike za progress.

Primer:

```text
G1 ────────────────►
G2 ─ wait ─ wait ─ wait ─ ...
```

Za razliku od deadlock-a:

```text
deadlock = nobody progresses
```

starvation može biti:

```text
system progresses
but one participant does not
```

To je posebno relevantno kod kompleksnih lock-free i scheduling algoritama.

---

# 140. Fairness vs Throughput

Scheduler ili synchronization mechanism može optimizovati:

```text
throughput
```

na račun:

```text
fairness
```

ili obrnuto.

Ne postoji univerzalno najbolja strategija.

Na primer:

```text
high throughput
```

može zahtevati batch processing.

Ali batch processing može povećati latency pojedinačnih tasks.

Architecture mora definisati cilj.

---

# 141. Observability

Production concurrency system treba imati observability.

Relevantne metrike:

```text
goroutines
queue depth
queue wait time
worker utilization
job duration
job failures
retry count
lock contention
blocked time
CPU
memory
GC
```

Posebno korisna metrika:

```text
queue depth over time
```

jer pokazuje da li workload raste brže nego što sistem može da ga obradi.

---

# 142. Queueing Theory Intuition

Ako je:

```text
arrival rate > service rate
```

queue će rasti.

Formalno:

```text
λ > μ
```

gde je:

```text
λ = arrival rate
μ = service rate
```

Ako:

```text
λ < μ
```

sistem može stabilno da obrađuje workload, pod pretpostavkom odgovarajućih uslova.

Ovo je fundamentalna osnova za:

```text
worker pool sizing
queue sizing
backpressure
capacity planning
```

---

# 143. The Real Goal of Concurrency

Cilj nije:

```text
more goroutines
```

Cilj nije ni:

```text
more CPU utilization
```

Cilj je:

```text
stable useful work
within acceptable latency
under expected load
```

Odnosno:

```text
                    Production Goal
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Correctness       Capacity         Latency
          │                │                │
          ▼                ▼                ▼
       No races        Bounded load      SLO/SLA
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                       Reliability
```

---

# 144. Senior-Level Questions

Čitalac sada treba da može da odgovori:

### Question 1

Zašto povećavanje broja workers ne garantuje veći throughput?

### Question 2

Koja je razlika između throughput-a i latency-ja?

### Question 3

Zašto je tail latency važniji od average latency-ja u mnogim production sistemima?

### Question 4

Šta može pokazati goroutine profile?

### Question 5

Kada koristiti mutex profile?

### Question 6

Kako false sharing može degradirati performance bez logičkog shared state-a?

### Question 7

Zašto sharding smanjuje contention?

### Question 8

Šta je admission control?

### Question 9

Zašto load shedding može povećati system reliability?

### Question 10

Koja je razlika između lokalnog `Mutex`-a i distributed lock-a?

### Question 11

Zašto distributed systems moraju računati na partial failure?

### Question 12

Zašto su idempotency i deduplication važni kod at-least-once delivery-ja?

### Question 13

Zašto "exactly once" treba koristiti veoma oprezno kao architectural claim?

### Question 14

Šta se dešava kada arrival rate postane veći od service rate-a?

### Question 15

Kako concurrency budget doprinosi stabilnosti sistema?

---

# 145. Engineering Principles

### Principle 17

> **Measure concurrency performance; do not infer it from code structure.**

### Principle 18

> **Optimize bottlenecks, not abstractions.**

### Principle 19

> **More concurrency can reduce performance.**

### Principle 20

> **Bound every resource whose exhaustion can destabilize the system.**

### Principle 21

> **Distributed synchronization requires explicit failure semantics.**

### Principle 22

> **Timeouts are architectural boundaries, not merely defensive programming.**

### Principle 23

> **Design distributed operations to tolerate retries and duplicate execution.**

### Principle 24

> **A stable system needs admission control, backpressure and load shedding.**

### Principle 25

> **Concurrency should be observable in production.**

---

# 146. Transition

Sledeći deo će se fokusirati na završni nivo Module 4:

```text
13-mastering-go-concurrency.md
```

i na završno objedinjavanje svih concurrency concepts.

Biće obrađeno:

```text
concurrency design methodology
correctness checklist
race detection strategy
deadlock analysis
goroutine lifecycle
ownership
cancellation
backpressure
bounded concurrency
performance analysis
profiling workflow
API design
testing concurrent systems
production readiness
code review
architectural trade-offs
```

Poseban fokus biće na pitanju:

```text
How do you review a concurrent Go system
as a senior/staff-level engineer?
```

Nakon toga će završni deo obuhvatiti:

```text
Mastery Checklist
Advanced Exercises
Architecture Scenarios
Interview-Level Questions
Production Checklist
```

---

# Module 4 — Advanced Go Concurrency

## Mastery, Design Methodology, Testing, Code Review & Production Readiness

> **Continuation of:** `golang/concurrency/docs/module-4/README.md - Deo #3/5`

---

# 147. Concurrency Mastery

Module 4 predstavlja prelazak sa:

```text
learning concurrency primitives
```

na:

```text
engineering concurrent systems
```

Senior-level Go developer ne treba samo da zna:

```text
goroutine
channel
mutex
atomic
context
select
```

Već treba da razume:

```text
ownership
lifecycle
ordering
coordination
backpressure
failure propagation
resource limits
observability
performance
testing
```

Drugim rečima:

> **Concurrency mastery is primarily about reasoning, not syntax.**

---

# 148. Concurrency Design Methodology

Za svaki novi concurrent subsystem može se koristiti sistematski proces.

```text
                    Problem
                       │
                       ▼
                Define ownership
                       │
                       ▼
               Define lifecycle
                       │
                       ▼
             Define synchronization
                       │
                       ▼
              Define cancellation
                       │
                       ▼
             Define backpressure
                       │
                       ▼
             Define failure modes
                       │
                       ▼
               Define observability
                       │
                       ▼
                  Benchmark
                       │
                       ▼
                    Test
                       │
                       ▼
                  Production
```

Ovo sprečava da concurrency bude dodat naknadno kao ad-hoc optimizacija.

---

# 149. Step 1 — Define the Work

Pre nego što se uvede goroutine, potrebno je definisati:

```text
What is the unit of work?
```

Na primer:

```text
ProcessRequest
ProcessJob
WriteRecord
FetchPage
TransformItem
```

Ako nije jasno šta predstavlja jedan task, teško je pravilno definisati concurrency boundary.

---

# 150. Step 2 — Define Ownership

Za svaki mutable state treba odgovoriti:

```text
Who owns this state?
```

Na primer:

```text
Worker A owns partition A
Worker B owns partition B
```

ili:

```text
One goroutine owns state
Other goroutines communicate through channels
```

Ownership često eliminiše potrebu za kompleksnom synchronization logikom.

---

# 151. Ownership as a Concurrency Primitive

Umesto:

```text
10 goroutines
     │
     ▼
shared map
     │
     ▼
global mutex
```

može se koristiti:

```text
          Owner goroutine
                │
          ┌─────┴─────┐
          ▼           ▼
       request      request
          │           │
          ▼           ▼
        state       state
```

Samo owner menja state.

Ostali goroutines komuniciraju sa owner-om.

Ovo je često poznato kao:

> **Share memory by communicating.**

---

# 152. Step 3 — Define Lifecycle

Svaka goroutine treba da ima odgovor na pitanja:

```text
When does it start?
Who owns it?
What makes it stop?
Who waits for it?
What happens if its work fails?
```

Loš design:

```text
func start() {
    go worker()
}
```

bez jasnog lifecycle-a.

Bolji design eksplicitno definiše:

```text
start
run
cancel
stop
wait
```

---

# 153. Goroutine Lifecycle Contract

Dobar concurrent component ima contract:

```text
Start
  │
  ▼
Running
  │
  ├── normal completion
  ├── cancellation
  ├── error
  └── shutdown
  │
  ▼
Stopped
  │
  ▼
Wait returns
```

Ako component može ostati između:

```text
Running
```

i:

```text
Stopped
```

bez mogućnosti kontrole, postoji potencijal za goroutine leak.

---

# 154. Step 4 — Define Synchronization

Tek nakon ownership i lifecycle analize bira se synchronization mechanism.

Moguće opcije:

```text
Mutex
RWMutex
Atomic
Channel
Once
WaitGroup
Cond
Context
```

Pitanje nije:

> "Koji primitive je najmoderniji?"

Već:

> "Koji primitive najjasnije izražava required synchronization semantics?"

---

# 155. Synchronization Decision Matrix

| Problem                 | Tipično rešenje                       |
| ----------------------- | ------------------------------------- |
| Mutual exclusion        | `sync.Mutex`                          |
| Read-heavy shared state | `sync.RWMutex`                        |
| Simple atomic state     | `sync/atomic`                         |
| Task communication      | channel                               |
| One-time initialization | `sync.Once`                           |
| Wait for workers        | `sync.WaitGroup`                      |
| Cancellation            | `context.Context`                     |
| Condition waiting       | `sync.Cond`                           |
| Bounded concurrency     | semaphore / worker pool               |
| Coordination            | channels / synchronization primitives |

Tabela nije zakon.

Ako design zahteva kombinovanje više primitives, to je normalno.

---

# 156. Step 5 — Define Cancellation

Svaki long-running concurrent operation treba imati cancellation strategy.

Pitanja:

```text
What cancels the operation?
What happens to queued work?
What happens to running work?
What happens to downstream operations?
Who observes cancellation?
```

Tipičan model:

```text
Parent Context
      │
      ▼
Service
      │
      ▼
Worker Pool
      │
      ▼
External I/O
```

Cancellation treba propagirati niz ceo execution graph.

---

# 157. Cancellation Is Cooperative

Go cancellation nije:

```text
kill goroutine
```

već:

```text
signal goroutine
```

Goroutine mora sarađivati.

Primer koncepta:

```text
for {
    select {
    case <-ctx.Done():
        return
    case item := <-jobs:
        process(item)
    }
}
```

Ako `process(item)` nikada ne proverava cancellation i ne koristi cancellable I/O, cancellation može biti odložena.

---

# 158. Cancellation Budget

Concurrent operation može imati više nivoa timeout-a:

```text
request deadline
       │
       ▼
service deadline
       │
       ▼
database deadline
```

Važno je da child operation ne bude duža od parent deadline-a.

Idealno:

```text
child deadline <= parent deadline
```

inače child može nastaviti da troši resources nakon što je parent već istekao.

---

# 159. Step 6 — Define Backpressure

Svaki producer/consumer system treba odgovoriti:

```text
What happens when consumers are slower?
```

Opcije:

```text
block producer
buffer
drop
reject
sample
batch
scale workers
```

Na primer:

```text
Producer
   │
   ▼
Bounded Queue
   │
   ▼
Workers
```

Ako je queue puna:

```text
enqueue
   │
   ▼
FULL
```

mora postojati definisana politika.

---

# 160. Unbounded Queues Are Dangerous

Konceptualno:

```text
Producer rate = 10,000/s
Consumer rate = 5,000/s
```

onda queue raste:

```text
5k
10k
15k
20k
...
```

Ako queue nema granicu:

```text
memory usage → ∞
```

u teorijskom modelu.

U praksi sistem može pasti mnogo ranije.

Zato je bounded queue često safety mechanism, a ne samo performance optimization.

---

# 161. Step 7 — Define Failure Semantics

Concurrent operation može imati:

```text
success
error
timeout
cancellation
panic
partial failure
```

Treba definisati:

```text
What happens to siblings?
What happens to queued work?
What happens to completed work?
Can work be retried?
Is retry safe?
```

---

# 162. Fail-Fast vs Best-Effort

Dva različita modela.

### Fail-fast

```text
Task A ── success
Task B ── error
             │
             ▼
        cancel siblings
```

Koristan kada rezultat nema smisla bez svih tasks.

### Best-effort

```text
Task A ── success
Task B ── error
Task C ── success
Task D ── success
```

Sistem nastavlja sa delimičnim rezultatima.

Izbor zavisi od domain semantics.

---

# 163. Error Propagation

Ako concurrent operation ima više child tasks:

```text
Parent
 ├── A
 ├── B
 ├── C
 └── D
```

treba definisati:

```text
Which error reaches the caller?
```

Mogući modeli:

```text
first error
all errors
aggregated errors
partial result + errors
```

Go standard library i moderni concurrency patterns omogućavaju različite načine modelovanja ovih semantics.

---

# 164. Structured Error Handling

Loš pattern:

```text
go func() {
    if err := work(); err != nil {
        log.Println(err)
    }
}()
```

Caller možda nikada neće znati da je work failed.

Bolji architecture omogućava:

```text
worker
   │
   ▼
error
   │
   ▼
coordinator
   │
   ▼
caller
```

Error treba biti deo concurrency design-a, ne samo logging side effect.

---

# 165. Panic in Concurrent Code

`panic` unutar goroutine-a nije automatski uhvaćen od strane parent goroutine-a.

Mentalni model:

```text
Parent goroutine
      │
      └── child goroutine
                │
                └── panic
```

Parent neće automatski dobiti:

```text
panic
```

kao regularan error.

Zbog toga long-running worker systems moraju eksplicitno definisati panic policy.

---

# 166. Production Panic Policy

Moguće strategije:

```text
panic crashes process
```

ili:

```text
recover at controlled boundary
```

ili:

```text
worker supervisor restarts worker
```

Ali `recover` ne treba koristiti kao zamenu za correctness.

Ako worker panic-uje, potrebno je razumeti:

```text
What state was partially modified?
Was work acknowledged?
Should the job retry?
Can retry duplicate side effects?
```

---

# 167. Step 8 — Define Resource Boundaries

Concurrent components troše:

```text
CPU
memory
goroutines
file descriptors
connections
queue slots
database connections
external API capacity
```

Svaki resource treba imati kontrolu.

Na primer:

```text
workers <= DB connections
```

ako svaki worker zahteva DB connection.

U suprotnom:

```text
workers = 1,000
DB pool = 50
```

većina workers će samo čekati.

---

# 168. Concurrency Is a Resource Allocation Problem

Možemo posmatrati concurrency kao:

```text
Available resources
        │
        ▼
Concurrency budget
        │
        ├── CPU
        ├── memory
        ├── DB
        ├── network
        └── downstream
```

Optimalan broj workers zavisi od bottleneck-a.

CPU-bound workload:

```text
workers ≈ CPU capacity
```

I/O-bound workload:

```text
workers > CPU count
```

ali samo do tačke gde downstream resources ostaju stabilni.

---

# 169. Step 9 — Define Observability

Concurrent component treba imati:

```text
metrics
logs
traces
profiles
```

Posebno treba biti moguće odgovoriti:

```text
How many tasks are running?
How many are queued?
How long do they wait?
How long do they execute?
How many fail?
How many retry?
How many goroutines exist?
Where is contention?
```

---

# 170. Correlation IDs

Kod concurrent systems jedan request može pokrenuti:

```text
Request
 ├── Task A
 ├── Task B
 └── Task C
```

Observability treba omogućiti da se svi eventi povežu sa istim logical operation.

Tipično:

```text
request_id
trace_id
span_id
job_id
```

Ovo je posebno važno kada se logovi više goroutines interleavuju.

---

# 171. Step 10 — Test the Model

Concurrent testing ne treba samo proveravati:

```text
expected result
```

već i:

```text
lifecycle
ordering
cancellation
race safety
deadlock safety
resource bounds
failure behavior
```

---

# 172. Race Detection

Go race detector je jedan od ključnih alata:

```text
go test -race ./...
```

Može otkriti određene:

```text
data races
```

ali ne dokazuje:

```text
no races exist
```

u svim mogućim execution paths.

Race detector je runtime analysis tool, ne formal proof system.

---

# 173. Race Detector Limitations

Race detector može propustiti problem ako:

```text
problematic code path
```

nikada nije izvršen tokom testa.

Na primer:

```text
if rareCondition {
    concurrentAccess()
}
```

Ako test nikada ne aktivira `rareCondition`, detector ne može analizirati taj execution.

Zato je potrebno kombinovati:

```text
race detector
+
stress tests
+
unit tests
+
integration tests
+
code review
```

---

# 174. Stress Testing

Concurrent bugs često zavise od:

```text
timing
scheduling
load
CPU count
GC
contention
```

Zbog toga treba koristiti:

```text
repeated execution
high concurrency
randomized scheduling
long-running tests
```

Primer koncepta:

```text
run test
100
1,000
10,000
100,000
times
```

Cilj je povećati verovatnoću da se problematičan interleaving pojavi.

---

# 175. Deterministic Testing

Idealno concurrent code treba dizajnirati tako da se što više ponašanja može testirati deterministički.

Na primer, umesto direktnog oslanjanja na:

```text
time.Sleep(...)
```

korisnije je imati:

```text
explicit synchronization
controlled signals
test hooks
fake clocks
bounded channels
```

`time.Sleep` nije synchronization primitive.

---

# 176. Why `time.Sleep` Is a Bad Synchronization Mechanism

Kod:

```text
go startWorker()

time.Sleep(100 * time.Millisecond)

assert(...)
```

test implicitno pretpostavlja:

```text
worker will finish within 100ms
```

Ali to zavisi od:

```text
CPU load
scheduler
machine
CI environment
GC
timing
```

Bolje:

```text
workerDone <- struct{}{}
```

pa:

```text
<-workerDone
```

ili drugi eksplicitni synchronization mechanism.

---

# 177. Testing Cancellation

Cancellation test treba proveriti:

```text
cancel()
```

i zatim:

```text
worker stops
```

Ne samo:

```text
function returns
```

Veoma je važno proveriti da nema:

```text
goroutine leak
```

nakon cancellation-a.

---

# 178. Testing Timeouts

Timeout test mora razlikovati:

```text
operation finished
```

od:

```text
operation was cancelled by deadline
```

Treba testirati i:

```text
timeout before work starts
timeout while waiting
timeout during I/O
timeout during retry
```

---

# 179. Testing Worker Pools

Worker pool testovi treba da pokriju:

```text
zero jobs
one job
many jobs
more jobs than workers
worker failure
queue full
cancellation
shutdown
repeated start/stop
```

Posebno:

```text
jobs == workers
jobs < workers
jobs > workers
```

mogu otkriti različite lifecycle bugove.

---

# 180. Testing Pipelines

Pipeline:

```text
Stage A
   │
   ▼
Stage B
   │
   ▼
Stage C
```

treba testirati kada:

```text
A fails
B fails
C fails
consumer stops
producer stops
context cancelled
channel closed
downstream slow
```

Najčešći bug je da upstream stage nastavi da šalje podatke nakon što downstream više ne prima.

---

# 181. Goroutine Leak Testing

Jedna moguća strategija je posmatranje:

```text
runtime.NumGoroutine()
```

pre i nakon testa.

Ali ovo je samo signal.

Broj goroutines može varirati zbog runtime internals i drugih aktivnosti.

Bolji pristup je:

```text
explicit lifecycle ownership
+
wait for shutdown
+
bounded completion
```

a broj goroutines koristiti kao dodatnu proveru.

---

# 182. Fuzzing Concurrent Systems

Fuzzing može pomoći kod concurrency APIs gde postoji veliki broj input kombinacija.

Može otkriti:

```text
unexpected state transitions
invalid inputs
edge cases
```

Ali fuzzing sam po sebi nije zamena za:

```text
race testing
stress testing
load testing
```

---

# 183. Model-Based Testing

Za kompleksne state machines korisno je definisati:

```text
State
  +
Allowed transition
  +
Invariant
```

Na primer:

```text
Open
 │
 ├── Close → Closed
 └── Cancel → Cancelled
```

Ako concurrent implementation proizvede:

```text
Closed → Open
```

test može otkriti violation.

---

# 184. Concurrency Invariants

Senior-level testing često proverava invariants, ne samo outputs.

Primer:

```text
Invariant:
activeWorkers <= maxWorkers
```

Drugi:

```text
Invariant:
closed queue never accepts new work
```

Treći:

```text
Invariant:
job cannot be acknowledged twice
```

Četvrti:

```text
Invariant:
cancelled context eventually stops worker
```

Invariants su veoma snažan način razmišljanja o correctness-u.

---

# 185. Code Review Methodology

Prilikom review-a concurrent Go koda, ne treba početi od pitanja:

```text
"Da li ovde treba Mutex?"
```

Bolji redosled:

```text
1. What state exists?
2. Who owns it?
3. Who can mutate it?
4. What are the concurrent actors?
5. What ordering is required?
6. How do they communicate?
7. How does work stop?
8. What happens on error?
9. What happens under overload?
10. What happens during shutdown?
```

Tek nakon toga analiziramo primitive.

---

# 186. Concurrency Code Review Checklist

### Ownership

* [ ] Svaki mutable state ima owner-a.
* [ ] Ownership je dokumentovan ili očigledan iz API-ja.
* [ ] Nema nepotrebnog shared mutable state-a.

### Synchronization

* [ ] Svaki shared state ima definisanu synchronization strategiju.
* [ ] Lock scope je minimalan.
* [ ] Nema lock-order inversion-a.
* [ ] Atomic operations imaju jasan memory-ordering requirement.

### Lifecycle

* [ ] Svaka goroutine ima stop condition.
* [ ] Postoji način da caller sačeka shutdown.
* [ ] Nema implicitnih background goroutines.

### Cancellation

* [ ] Context se propagira.
* [ ] Long-running operations proveravaju cancellation.
* [ ] I/O operacije imaju deadlines.

### Backpressure

* [ ] Queue capacity je definisana.
* [ ] Producer behavior pri overload-u je definisan.
* [ ] Nema nekontrolisanog rasta work-a.

### Errors

* [ ] Worker errors nisu izgubljeni.
* [ ] Panic policy je definisan.
* [ ] Retry semantics su jasne.

### Testing

* [ ] Testovi se izvršavaju sa `-race`.
* [ ] Cancellation je testirana.
* [ ] Shutdown je testiran.
* [ ] Overload je testiran.
* [ ] Failure paths su testirani.

---

# 187. Common Senior-Level Mistakes

Čak i iskusan Go developer može napraviti sledeće greške.

### Mistake 1 — Goroutine Everywhere

```text
go doSomething()
```

se koristi bez lifecycle ownership-a.

---

### Mistake 2 — Global Mutex

Jedan lock štiti ceo sistem:

```text
globalMu.Lock()
...
globalMu.Unlock()
```

što može pretvoriti concurrent system u serial bottleneck.

---

### Mistake 3 — Unbounded Workers

```text
for _, job := range jobs {
    go process(job)
}
```

može proizvesti:

```text
N jobs
=
N goroutines
```

bez resource control-a.

---

### Mistake 4 — Sleep-Based Synchronization

```text
time.Sleep(...)
```

se koristi za čekanje concurrency events.

---

### Mistake 5 — Ignored Errors

```text
go func() {
    _ = process()
}()
```

može izgubiti failure signal.

---

### Mistake 6 — Missing Cancellation

Background worker nema:

```text
ctx.Done()
```

i može živeti koliko i proces.

---

### Mistake 7 — Closing Channels Incorrectly

Više goroutines pokušava da zatvori isti channel:

```text
close(ch)
```

što može izazvati:

```text
panic: close of closed channel
```

---

### Mistake 8 — Retry Without Idempotency

```text
retry()
```

može proizvesti duplicate side effects.

---

### Mistake 9 — Optimizing Without Profiling

Developer menja:

```text
Mutex → Atomic
```

bez dokaza da je mutex bottleneck.

---

### Mistake 10 — Confusing Race-Free With Correct

Program može biti:

```text
race-free
```

a ipak imati:

```text
deadlock
livelock
starvation
incorrect ordering
lost updates
incorrect cancellation
```

---

# 188. Concurrency Anti-Patterns

Najvažniji anti-pattern-i:

```text
goroutine leaks
unbounded concurrency
unbounded queues
global locks
shared mutable state everywhere
sleep-based synchronization
fire-and-forget work
ignored errors
ignored cancellation
incorrect channel ownership
retry storms
lock contention
busy waiting
premature atomics
premature lock-free algorithms
```

---

# 189. Concurrency Design Smells

Code smell:

```text
go go go
```

ako nema jasnog ownership-a.

Code smell:

```text
mutex everywhere
```

može značiti da ownership model nije dobro definisan.

Code smell:

```text
select {
case <-time.After(...):
}
```

na mnogo mesta može ukazivati na timing-based coordination umesto explicit synchronization-a.

Code smell:

```text
make(chan T, 1000000)
```

može ukazivati na pokušaj skrivanja backpressure problema velikim bufferom.

---

# 190. Production Readiness

Pre deployment-a concurrent component treba proći kroz:

```text
Correctness
   │
   ▼
Race Safety
   │
   ▼
Lifecycle
   │
   ▼
Cancellation
   │
   ▼
Backpressure
   │
   ▼
Failure Handling
   │
   ▼
Observability
   │
   ▼
Performance
   │
   ▼
Capacity
   │
   ▼
Load Testing
```

Tek tada concurrency implementation može biti smatran production-ready.

---

# 191. Production Concurrency Checklist

## Correctness

* [ ] Nema poznatih data races.
* [ ] Invariants su definisani.
* [ ] Ordering requirements su jasni.
* [ ] Shared state ima ownership model.

## Lifecycle

* [ ] Sve goroutines imaju lifecycle.
* [ ] Shutdown je determinističan.
* [ ] Worker pool može bezbedno da se ugasi.
* [ ] Nema poznatih goroutine leak-ova.

## Failure

* [ ] Errors se propagiraju.
* [ ] Panic behavior je definisan.
* [ ] Retry behavior je definisan.
* [ ] Duplicate work je bezbedan ili sprečen.

## Capacity

* [ ] Worker count je bounded.
* [ ] Queue size je bounded.
* [ ] Connection pools su bounded.
* [ ] Memory growth je kontrolisan.

## Cancellation

* [ ] Context se propagira.
* [ ] Deadlines postoje za remote I/O.
* [ ] Cancellation se poštuje.

## Performance

* [ ] Baseline benchmark postoji.
* [ ] Bottleneck je profilisan.
* [ ] Contention je analiziran.
* [ ] Memory allocations su analizirane.

## Observability

* [ ] Queue depth se meri.
* [ ] Worker utilization se meri.
* [ ] Error rate se meri.
* [ ] Latency se meri.
* [ ] Goroutine count je observable.

---

# 192. Senior Architecture Exercise

Zamisli sistem:

```text
HTTP API
   │
   ▼
Job Submission
   │
   ▼
Queue
   │
   ▼
Worker Pool
   │
   ▼
External API
```

Requirements:

```text
10,000 requests/sec
500 workers maximum
external API = 200 concurrent requests
queue = 5,000 jobs
request timeout = 2s
job timeout = 5s
```

Potrebno je definisati:

```text
worker count
queue behavior
backpressure
timeout propagation
retry policy
idempotency
shutdown
metrics
failure handling
```

Ovo predstavlja tipičan senior-level concurrency design problem.

---

# 193. Architecture Reasoning

Prvo identifikujemo resource bottleneck:

```text
External API
    │
    ▼
200 concurrent requests
```

Zato:

```text
worker count = 500
```

ne znači:

```text
500 concurrent external calls
```

potrebno je imati dodatni limit:

```text
500 workers
      │
      ▼
200 external-call permits
      │
      ▼
External API
```

To pokazuje važnu razliku između:

```text
worker concurrency
```

i:

```text
downstream concurrency
```

---

# 194. Graceful Shutdown

Production system mora imati definisan shutdown sequence.

Tipičan model:

```text
Receive SIGTERM
      │
      ▼
Stop accepting new work
      │
      ▼
Cancel/close admission
      │
      ▼
Drain or cancel queued work
      │
      ▼
Wait for active workers
      │
      ▼
Close resources
      │
      ▼
Exit
```

Redosled je važan.

Ako se prvo zatvori database connection:

```text
workers still running
       │
       ▼
DB closed
       │
       ▼
worker failures
```

shutdown postaje nekontrolisan.

---

# 195. Graceful Shutdown Semantics

Potrebno je definisati da li sistem:

```text
drains all work
```

ili:

```text
cancels remaining work
```

ili koristi:

```text
deadline-bounded drain
```

Na primer:

```text
shutdown timeout = 30s
```

model:

```text
Shutdown
   │
   ▼
Drain for 30s
   │
   ├── completed
   │
   └── remaining → cancel
```

Ovo je često praktičan production compromise.

---

# 196. Zero-Downtime Deployment

Kod rolling deployment-a:

```text
Old Instance
     │
     ├── stop admission
     ├── drain
     └── terminate

New Instance
     │
     └── accept traffic
```

Concurrent lifecycle postaje deo deployment architecture-a.

---

# 197. Concurrency and SLOs

Sistem može imati SLO:

```text
99% requests < 200ms
```

Concurrency design treba biti analiziran prema tom target-u.

Na primer:

```text
average latency = 100ms
```

nije dovoljno ako:

```text
p99 = 2s
```

SLO mora biti povezan sa:

```text
queueing
worker utilization
downstream latency
timeouts
retries
```

---

# 198. Queueing and Tail Latency

Ako queue postane pun:

```text
request
  │
  ▼
queue
  │
  ▼
wait
```

latency raste čak i ako processing time ostaje isti.

Dakle:

```text
latency
=
queue wait
+
execution
+
downstream wait
```

Ovo je veoma važan model za performance analysis.

---

# 199. Concurrency Is a System Property

Nije dovoljno reći:

```text
"This function is concurrent."
```

Concurrency se prostire kroz sistem:

```text
HTTP
 │
 ▼
Service
 │
 ▼
Worker
 │
 ▼
Database
 │
 ▼
External API
```

Ako jedan downstream resource nije concurrent-safe ili ima mali capacity:

```text
whole system concurrency
```

mora biti prilagođena tom bottleneck-u.

---

# 200. Final Mental Model

Senior Go engineer treba da razmišlja ovako:

```text
                     WORK
                      │
                      ▼
                 Concurrency
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Ownership   Lifecycle   Capacity
          │           │           │
          ▼           ▼           ▼
      Synchronize  Cancel      Backpressure
          │           │           │
          └───────────┼───────────┘
                      ▼
                   Failure
                      │
                      ▼
                 Observability
                      │
                      ▼
                  Profiling
                      │
                      ▼
                   Testing
                      │
                      ▼
                 Production
```

Ovo je suština Module 4.

---

# 201. Module 4 Mastery Criteria

Nakon završetka Module 4 čitalac bi trebalo da može da:

* dizajnira concurrent subsystem od nule;
* definiše ownership model;
* bira odgovarajući synchronization primitive;
* dizajnira worker pool;
* dizajnira pipeline;
* implementira fan-out/fan-in;
* uvede bounded concurrency;
* uvede backpressure;
* implementira cancellation;
* dizajnira graceful shutdown;
* analizira goroutine lifecycle;
* otkriva goroutine leak;
* analizira data race;
* razlikuje deadlock, livelock i starvation;
* koristi atomic operations kada imaju smisla;
* razume memory-ordering posledice;
* profilira contention;
* profilira CPU i memory usage;
* benchmark-uje concurrent code;
* testira concurrency pod load-om;
* dizajnira retry semantics;
* razume idempotency;
* projektuje distributed work processing;
* razume partial failure;
* razume distributed locking limitations;
* dizajnira production-grade concurrency architecture.

---

# 202. Staff-Level Perspective

Na najvišem nivou pitanje više nije:

> "Kako da napišem ovaj concurrent code?"

Već:

> "Kako da ovaj sistem ostane correct, bounded, observable and recoverable kada workload, latency and failures promene uslove?"

To zahteva razmišljanje u terminima:

```text
state
ownership
invariants
ordering
capacity
failure
recovery
observability
```

a ne samo:

```text
goroutines
channels
mutexes
```

---

# 203. Final Principle

Najvažniji princip celog naprednog concurrency materijala:

> **Concurrency is not about making more things happen at once. It is about controlling simultaneous work safely and predictably.**

Dobar concurrent system je:

```text
correct
+
bounded
+
cancelable
+
observable
+
testable
+
recoverable
+
performant
```

Ako nedostaje jedna od ovih osobina, concurrency design može biti funkcionalan u idealnim uslovima, ali nepouzdan pod production load-om.

---

# 204. Transition

Preostaje još završni deo:

```text
golang/concurrency/docs/module-4/README.md - Deo #5/5
```

Finalni deo će objediniti ceo Module 4 kroz:

```text
Module 4 Mastery Checklist
Advanced Exercises
Concurrency Case Studies
Architecture Challenges
Senior/Staff Interview Questions
Production Scenarios
Final Review
Recommended Learning Path
Module 4 Completion Criteria
```

Poseban fokus biće na praktičnoj sintezi:

```text
Goroutines
    +
Channels
    +
Synchronization
    +
Atomics
    +
Context
    +
Scheduler
    +
Memory Model
    +
Performance
    +
Testing
    +
Distributed Concurrency
    =
Production-Grade Go Concurrency
```

# Module 4 — Advanced Go Concurrency

## Mastery Checklist, Case Studies, Exercises, Interview Questions & Completion Criteria

> **Continuation of:** `golang/concurrency/docs/module-4/README.md - Deo #4/5`

---

# 205. Module 4 — Final Integration

Module 4 je završna faza concurrency putanje.

Prethodni moduli su izgradili znanje o pojedinačnim mehanizmima:

```text
goroutines
channels
select
WaitGroup
Mutex
RWMutex
Once
Context
timeouts
cancellation
scheduler
GOMAXPROCS
parallelism
```

Module 4 ih posmatra kao međusobno povezane delove jednog sistema.

Cilj više nije samo:

```text
"Can I use a channel?"
```

nego:

```text
"Can I design a concurrent system whose
correctness, lifecycle, resource usage,
failure behavior and performance I can explain?"
```

---

# 206. The Complete Concurrency Model

Go concurrency može se posmatrati kroz nekoliko fundamentalnih dimenzija:

```text
                    CONCURRENT SYSTEM
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Execution        Coordination      State
          │                │                │
          ▼                ▼                ▼
      Goroutines        Channels          Ownership
      Scheduler         Mutexes           Immutability
      Parallelism       Atomics           Isolation
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                       Lifecycle
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Cancellation   Timeout      Shutdown
              │            │            │
              └────────────┼────────────┘
                           ▼
                        Capacity
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          Backpressure   Queues      Limits
                           │
                           ▼
                        Failure
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Errors        Retry       Recovery
                           │
                           ▼
                     Observability
                           │
                           ▼
                       Production
```

Ovaj model predstavlja osnovu za dizajniranje ozbiljnih concurrent sistema.

---

# 207. Mastery Checklist — Goroutines

Čitalac treba da razume:

* [ ] šta goroutine predstavlja;
* [ ] kako goroutine lifecycle funkcioniše;
* [ ] kako goroutine završava;
* [ ] kako goroutine može procuriti;
* [ ] kako scheduler raspoređuje goroutines;
* [ ] razliku između goroutine-a i OS thread-a;
* [ ] razliku između concurrency i parallelism;
* [ ] kada kreiranje goroutine-a ima smisla;
* [ ] kada goroutine predstavlja nepotrebni overhead;
* [ ] kako strukturirati goroutine ownership.

Posebno mora biti jasno:

```text
goroutine != free resource
```

Iako su goroutines relativno jeftine, njihov broj ipak mora biti kontrolisan.

---

# 208. Mastery Checklist — Channels

Čitalac treba da može da objasni:

* [ ] unbuffered channel;
* [ ] buffered channel;
* [ ] send semantics;
* [ ] receive semantics;
* [ ] blocking;
* [ ] channel directions;
* [ ] `close`;
* [ ] receive from closed channel;
* [ ] `range` over channel;
* [ ] `select`;
* [ ] nil channels;
* [ ] channel ownership;
* [ ] producer/consumer patterns;
* [ ] fan-in;
* [ ] fan-out;
* [ ] pipelines.

Posebno:

> Channel nije samo data structure. Channel je synchronization mechanism.

---

# 209. Mastery Checklist — Synchronization

Čitalac treba da zna kada koristiti:

```text
sync.Mutex
sync.RWMutex
sync.WaitGroup
sync.Once
sync.Cond
sync/atomic
channels
context
```

i, još važnije, kada **ne koristiti** određeni primitive.

Primer:

Ako je problem:

```text
wait until all workers finish
```

prirodnije rešenje je:

```text
sync.WaitGroup
```

nego:

```text
channel + custom counter + mutex
```

Jednostavniji synchronization model je obično bolji synchronization model.

---

# 210. Mastery Checklist — Atomics

Čitalac mora razumeti da atomics nisu:

```text
"faster mutexes"
```

Atomics su primitive za specifične state transitions.

Primeri:

```text
counter
flag
state machine
pointer publication
statistics
lock-free coordination
```

Ali:

```text
multiple related fields
complex invariants
large critical sections
```

često zahtevaju drugačiji model.

---

# 211. Mastery Checklist — Context

Čitalac treba da razume:

```text
context.WithCancel
context.WithTimeout
context.WithDeadline
context.WithValue
ctx.Done()
ctx.Err()
```

i posebno:

> `context.Context` propagira cancellation i request-scoped metadata; ne treba ga koristiti kao generički container za application state.

---

# 212. Mastery Checklist — Scheduler

Treba razumeti konceptualni model:

```text
G = goroutine
M = machine / OS thread
P = processor
```

i odnos:

```text
G
│
▼
P
│
▼
M
│
▼
OS CPU
```

Ne mora se memorisati svaki runtime implementation detail.

Važno je razumeti:

```text
runnable
running
blocked
waiting
syscall
preemption
work stealing
```

i njihov uticaj na performance.

---

# 213. Mastery Checklist — Memory Model

Napredni concurrency developer mora razumeti:

```text
happens-before
synchronization
visibility
ordering
data race
atomicity
```

Posebno:

> Race-free program nije automatski logički korektan program.

Može postojati:

```text
no data race
+
wrong synchronization semantics
```

što i dalje daje incorrect behavior.

---

# 214. Mastery Checklist — Lifecycle

Za svaku goroutine treba moći odgovoriti:

```text
Who starts me?
Who owns me?
What work do I perform?
What can block me?
What stops me?
Who waits for me?
What happens if I fail?
```

Ako nema odgovora:

```text
Who stops me?
```

treba posumnjati na goroutine leak.

---

# 215. Mastery Checklist — Backpressure

Čitalac mora razumeti:

```text
producer rate
consumer rate
queue capacity
worker capacity
downstream capacity
```

i njihovu međusobnu vezu.

Na primer:

```text
producer = 100k jobs/s
consumer = 20k jobs/s
```

ne rešava se jednostavno sa:

```text
make(chan Job, 1000000)
```

Jer time samo pomeramo problem:

```text
rate mismatch
       │
       ▼
memory growth
       │
       ▼
resource exhaustion
```

---

# 216. Mastery Checklist — Failure Handling

Concurrent system mora definisati:

```text
error
panic
timeout
cancellation
retry
partial failure
shutdown
```

Posebno treba razumeti da retry povećava concurrency load.

Ako downstream sistem ima problem:

```text
failure
   │
   ▼
retry
   │
   ▼
more requests
   │
   ▼
more load
   │
   ▼
more failures
```

ovo može proizvesti:

> **retry storm**

---

# 217. Mastery Checklist — Testing

Treba koristiti:

```text
go test
go test -race
benchmarks
profiling
stress tests
fuzzing
integration tests
load tests
```

ali svaki alat rešava drugačiji problem.

| Tool              | Primarna svrha                |
| ----------------- | ----------------------------- |
| `go test`         | correctness                   |
| `go test -race`   | race detection                |
| benchmark         | performance                   |
| CPU profile       | CPU bottleneck                |
| memory profile    | allocation/memory analysis    |
| goroutine profile | goroutine state/leaks         |
| block profile     | blocking                      |
| mutex profile     | lock contention               |
| stress test       | rare scheduling failures      |
| load test         | system behavior under load    |
| fuzz test         | input/state-space exploration |

---

# 218. Mastery Checklist — Performance

Ne treba optimizovati:

```text
goroutine
vs
thread
```

ili:

```text
channel
vs
mutex
```

samo na osnovu intuicije.

Potrebno je meriti:

```text
throughput
latency
p50
p95
p99
CPU
memory
allocations
contention
blocking
```

Performance engineering počinje merenjem.

---

# 219. Advanced Case Study #1 — Concurrent HTTP Fetcher

Pretpostavimo:

```text
1000 URLs
```

Potrebno je fetch-ovati sve URL-ove, ali:

```text
maximum concurrency = 20
```

Naivno:

```text
for _, url := range urls {
    go fetch(url)
}
```

stvara:

```text
1000 goroutines
```

Bolji model:

```text
URLs
 │
 ▼
Jobs
 │
 ▼
Bounded Worker Pool
 │
 ├── Worker 1
 ├── Worker 2
 ├── ...
 └── Worker 20
 │
 ▼
Results
```

Concurrency limit je:

```text
20
```

a ne:

```text
1000
```

---

# 220. HTTP Fetcher — Failure Semantics

Potrebno je definisati:

```text
What happens if one URL fails?
```

Moguće:

```text
fail-fast
```

ili:

```text
collect all results
```

Ako se koristi best-effort model:

```text
Result {
    URL
    Value
    Error
}
```

omogućava da se uspešni i neuspešni rezultati obrade zajedno.

---

# 221. HTTP Fetcher — Cancellation

Ako caller otkaže request:

```text
ctx.Done()
```

treba da utiče na:

```text
job submission
workers
HTTP requests
result collection
```

Idealni lifecycle:

```text
Request cancelled
       │
       ▼
Stop submitting jobs
       │
       ▼
Cancel active requests
       │
       ▼
Workers exit
       │
       ▼
Function returns
```

---

# 222. Advanced Case Study #2 — Bounded Worker Pool

Problem:

```text
10,000 jobs
```

sa:

```text
50 workers
```

Architecture:

```text
                 ┌─────────────┐
Jobs ───────────►│   Queue     │
                 └──────┬──────┘
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
         Worker       Worker      Worker
            │           │           │
            └───────────┼───────────┘
                        ▼
                     Results
```

Treba definisati:

```text
queue capacity
worker count
shutdown behavior
error behavior
retry policy
```

---

# 223. Worker Pool — Shutdown

Graceful shutdown može biti:

```text
Stop producer
     │
     ▼
Close jobs
     │
     ▼
Workers drain jobs
     │
     ▼
Workers exit
     │
     ▼
WaitGroup.Wait()
```

Bitno:

> Worker pool treba da ima jasno definisanog vlasnika nad `jobs` channel-om.

---

# 224. Advanced Case Study #3 — Fan-Out/Fan-In

Input:

```text
1000 records
```

Processing:

```text
parse
validate
transform
```

Architecture:

```text
             ┌── Worker A ──┐
             │              │
Input ───────┼── Worker B ──┼──► Merge
             │              │
             └── Worker C ──┘
```

Fan-out:

```text
one stream
   ↓
multiple workers
```

Fan-in:

```text
multiple workers
   ↓
one result stream
```

Ovaj pattern je veoma koristan za CPU-bound ili I/O-bound workloads koji imaju nezavisne tasks.

---

# 225. Advanced Case Study #4 — Pipeline

Pipeline:

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
Write
```

Svaka faza može imati:

```text
one goroutine
```

ili:

```text
multiple workers
```

zavisno od bottleneck-a.

Na primer:

```text
Read       = 1
Parse      = 4
Validate   = 8
Transform  = 8
Write      = 2
```

Ovo je oblik stage-specific concurrency.

---

# 226. Pipeline Backpressure

Ako `Write` stage obrađuje samo:

```text
1000 items/s
```

dok `Transform` proizvodi:

```text
5000 items/s
```

onda:

```text
Transform
    │
    ▼
queue
    │
    ▼
Write
```

mora usporiti.

Pipeline bez backpressure mehanizma može dovesti do:

```text
unbounded memory
```

---

# 227. Advanced Case Study #5 — Rate-Limited API

Pretpostavimo:

```text
external API:
100 requests/sec
```

Naš sistem ima:

```text
500 workers
```

Worker count ne sme automatski značiti:

```text
500 requests/sec
```

Potrebna je rate-limiting policy.

Konceptualno:

```text
Workers
   │
   ▼
Rate Limiter
   │
   ▼
External API
```

Concurrency limit i rate limit nisu ista stvar.

---

# 228. Concurrency Limit vs Rate Limit

### Concurrency limit

Kontroliše:

```text
How many operations are active?
```

Na primer:

```text
20 concurrent requests
```

### Rate limit

Kontroliše:

```text
How many operations occur per unit of time?
```

Na primer:

```text
100 requests/sec
```

Sistem može zahtevati oba:

```text
max concurrent = 20
max rate = 100/sec
```

---

# 229. Advanced Case Study #6 — Database Workers

Pretpostavimo:

```text
100 workers
```

ali:

```text
DB connection pool = 20
```

Ako svaki worker koristi connection:

```text
100 workers
     │
     ▼
20 DB connections
```

80 workers može samo čekati.

Bolji design može uskladiti:

```text
worker concurrency
```

sa:

```text
DB capacity
```

ili eksplicitno kontrolisati database concurrency.

---

# 230. Advanced Case Study #7 — Cache Refresh

Cache ima background refresh:

```text
Request
  │
  ▼
Cache
  │
  ├── hit → return
  │
  └── miss
       │
       ▼
   refresh
```

Ako 10,000 requests istovremeno detektuje cache miss:

```text
10,000 requests
      │
      ▼
10,000 refreshes
```

dobijamo:

> cache stampede

Potrebna je koordinacija.

Mogući pristup:

```text
singleflight-style deduplication
```

gde samo jedna goroutine izvršava refresh, dok ostale čekaju rezultat.

---

# 231. Advanced Case Study #8 — Graceful Service Shutdown

Service:

```text
HTTP Server
     │
     ├── Worker Pool
     │
     ├── Background Consumer
     │
     └── Metrics Reporter
```

Shutdown:

```text
SIGTERM
  │
  ▼
Stop HTTP admission
  │
  ▼
Cancel root context
  │
  ├── Workers stop
  ├── Consumer stop
  └── Metrics reporter stop
  │
  ▼
Wait
  │
  ▼
Close resources
  │
  ▼
Exit
```

Ovo predstavlja production lifecycle.

---

# 232. Advanced Case Study #9 — Deadlock Investigation

Pretpostavimo:

```text
Goroutine A
    Lock(A)
    Lock(B)

Goroutine B
    Lock(B)
    Lock(A)
```

Graph:

```text
A waits for B
B waits for A
```

ciklus:

```text
A → B → A
```

što predstavlja deadlock.

Prvi korak u rešavanju nije dodavanje timeout-a.

Prvi korak je:

> Razumeti lock dependency graph.

---

# 233. Lock Ordering

Jedna od standardnih strategija:

```text
Always acquire locks in the same order.
```

Na primer:

```text
A < B < C
```

svaka goroutine mora:

```text
A → B → C
```

a nikada:

```text
C → B → A
```

Time se smanjuje mogućnost circular wait-a.

---

# 234. Advanced Case Study #10 — Lock Contention

Pretpostavimo:

```text
100 goroutines
        │
        ▼
single mutex
        │
        ▼
tiny critical section
```

Ako profil pokaže visoku contention:

```text
mutex wait
```

moguća rešenja:

```text
reduce critical section
shard state
partition data
reduce shared state
use atomics where appropriate
change ownership model
```

Ali:

> Ne treba automatski zameniti mutex atomikom.

---

# 235. Advanced Case Study #11 — Sharded State

Umesto:

```text
              Global Map
                  │
              Global Mutex
```

može:

```text
Shard 0 → Mutex
Shard 1 → Mutex
Shard 2 → Mutex
...
Shard N → Mutex
```

Key se mapira na shard:

```text
hash(key) % N
```

Time se contention može smanjiti.

Cena:

```text
more complexity
```

Zato je potrebno benchmark-ovati.

---

# 236. Advanced Case Study #12 — Atomic State Machine

Neki state machine-i mogu koristiti atomic state:

```text
Created
   │
   ▼
Running
   │
   ▼
Stopping
   │
   ▼
Stopped
```

Atomic CAS može omogućiti:

```text
Created → Running
```

samo jednom caller-u.

Ali svaki transition mora imati jasno definisane invariants.

---

# 237. Advanced Case Study #13 — Distributed Work

U distributed system-u:

```text
Node A
Node B
Node C
```

mogu obrađivati isti posao.

Sada lokalni:

```text
Mutex
```

nije dovoljan.

Potrebni su distributed coordination mechanisms:

```text
queue
lease
database lock
distributed lock
leader election
idempotency key
```

Ovde concurrency prelazi granice jednog Go procesa.

---

# 238. Local vs Distributed Concurrency

Local:

```text
goroutines
channels
mutexes
atomics
```

Distributed:

```text
network
timeouts
retries
leases
duplicates
partial failure
clock uncertainty
partitions
```

Distributed concurrency je fundamentalno teži problem.

---

# 239. Idempotency

Ako job može biti izvršen dva puta:

```text
Job
 │
 ├── attempt 1
 │
 └── attempt 2
```

system mora moći bezbedno da obradi duplicate.

Primer:

```text
payment_id = 123
```

umesto:

```text
charge()
```

bez identiteta operacije, koristi se:

```text
charge(payment_id)
```

i downstream može detektovati duplicate.

---

# 240. Exactly-Once Illusion

U distributed systems treba biti oprezan sa terminom:

```text
exactly once
```

Mnogi sistemi praktično implementiraju:

```text
at-least-once delivery
```

što znači:

```text
job may execute more than once
```

Zato su:

```text
idempotency
deduplication
transaction boundaries
```

ključni.

---

# 241. Advanced Exercises — Level 1

### Exercise 1 — Goroutine Lifecycle

Implementirati worker koji:

```text
starts
processes jobs
responds to cancellation
returns
```

Requirements:

* nema goroutine leak;
* worker se može testirati;
* shutdown je determinističan.

---

### Exercise 2 — Bounded Worker Pool

Implementirati:

```text
NewWorkerPool(n)
Submit(job)
Shutdown()
Wait()
```

Requirements:

```text
max workers = n
```

i:

```text
Submit
```

mora imati definisano ponašanje tokom shutdown-a.

---

### Exercise 3 — Fan-In

Implementirati funkciju:

```text
merge(channels ...<-chan T) <-chan T
```

Requirements:

* svi input channels se obrađuju;
* output se zatvara tek kada svi input-i završe;
* nema goroutine leak-a.

---

### Exercise 4 — Fan-Out

Implementirati worker distribution:

```text
jobs → N workers
```

uz:

```text
bounded concurrency
```

---

### Exercise 5 — Cancellation

Implementirati pipeline koji se kompletno zaustavlja kada:

```text
ctx.Done()
```

postane ready.

---

# 242. Advanced Exercises — Level 2

### Exercise 6 — Bounded Pipeline

Napraviti:

```text
producer
→ parser
→ transformer
→ consumer
```

sa bounded channels.

Testirati behavior kada je consumer spor.

---

### Exercise 7 — Retry With Backoff

Implementirati retry:

```text
attempt 1
   │
   ▼
backoff
   │
   ▼
attempt 2
   │
   ▼
backoff
   │
   ▼
attempt 3
```

Requirements:

* maximum attempts;
* context cancellation;
* maximum delay;
* error propagation.

---

### Exercise 8 — Rate Limiter

Implementirati rate limiter koji dozvoljava:

```text
100 operations/sec
```

i podržava cancellation.

---

### Exercise 9 — Concurrent Cache

Implementirati thread-safe cache.

Requirements:

* concurrent reads;
* concurrent writes;
* race-free behavior;
* bounded lifecycle;
* benchmarks.

---

### Exercise 10 — Singleflight Refresh

Implementirati cache refresh deduplication:

```text
100 callers
    │
    ▼
one refresh
    │
    ▼
100 callers receive result
```

---

# 243. Advanced Exercises — Level 3

### Exercise 11 — Graceful Worker Pool

Implementirati worker pool koji podržava:

```text
Submit
Cancel
Shutdown
Wait
```

uz:

```text
drain timeout
```

---

### Exercise 12 — Priority Queue Workers

Implementirati bounded priority work queue.

Requirements:

```text
high priority
medium priority
low priority
```

i definisati starvation policy.

---

### Exercise 13 — Dynamic Worker Pool

Implementirati pool koji može povećavati/smanjivati broj workers prema:

```text
queue depth
```

ali mora imati:

```text
min workers
max workers
cooldown
```

---

### Exercise 14 — Circuit Breaker

Implementirati state machine:

```text
Closed
  │
  ▼
Open
  │
  ▼
Half-Open
  │
  └── success → Closed
```

Testirati concurrent calls.

---

### Exercise 15 — Concurrent Batch Processor

Implementirati processor koji:

```text
collects items
   │
   ▼
forms batches
   │
   ▼
processes batches concurrently
```

uz:

```text
maximum batch size
maximum wait time
maximum concurrency
```

---

# 244. Advanced Exercises — Level 4

### Exercise 16 — Concurrent HTTP Aggregator

Jedan request poziva:

```text
Service A
Service B
Service C
```

paralelno.

Requirements:

* shared context;
* timeout;
* cancellation;
* error propagation;
* partial failure policy;
* bounded concurrency.

---

### Exercise 17 — Resilient API Client

Implementirati:

```text
timeout
retry
backoff
rate limit
circuit breaker
```

ali bez stvaranja retry storm-a.

---

### Exercise 18 — Distributed Job Worker

Dizajnirati worker koji može da obrađuje jobs sa:

```text
at-least-once delivery
```

i mora sprečiti neželjene duplicate side effects.

---

### Exercise 19 — Graceful HTTP Server

Implementirati server sa:

```text
HTTP handlers
background workers
queue
database
shutdown
```

i graceful shutdown tokom:

```text
SIGTERM
```

---

### Exercise 20 — Production Concurrency System

Projektovati kompletan sistem:

```text
HTTP API
    │
    ▼
Job Queue
    │
    ▼
Worker Pool
    │
    ▼
External API
```

Requirements:

```text
10k requests/sec
500 workers maximum
200 downstream concurrent calls
5k queue capacity
2s request timeout
5s job timeout
retry
backpressure
metrics
graceful shutdown
```

Potrebno je napisati:

```text
architecture
invariants
failure model
concurrency model
capacity model
test strategy
benchmark strategy
observability strategy
```

---

# 245. Senior-Level Interview Questions

## Goroutines

1. Kako biste pronašli goroutine leak?
2. Kako definišete lifecycle goroutine-a?
3. Kada je goroutine pool bolji od `go func()` za svaki task?
4. Da li veći broj goroutines automatski povećava throughput?
5. Kako scheduler utiče na latency?

---

# 246. Channels

1. Koja je razlika između buffered i unbuffered channel-a?
2. Šta se dešava kada šaljete u nil channel?
3. Šta se dešava kada primate iz closed channel-a?
4. Ko treba da zatvori channel?
5. Kada channel nije dobro rešenje?

---

# 247. Synchronization

1. Kada koristiti Mutex umesto channel-a?
2. Kada koristiti RWMutex?
3. Kada atomic?
4. Kako nastaje deadlock?
5. Kako sprečiti lock-order inversion?
6. Kako smanjiti lock contention?

---

# 248. Memory Model

1. Šta je happens-before?
2. Šta predstavlja data race?
3. Da li race-free znači correct?
4. Zašto običan read/write nije dovoljan za synchronization?
5. Kako synchronization primitive utiče na visibility?

---

# 249. Scheduler

1. Šta predstavljaju G, M i P?
2. Kako goroutine prelazi u waiting state?
3. Šta je work stealing?
4. Kako preemption utiče na fairness?
5. Kada `GOMAXPROCS` ima performance impact?

---

# 250. Worker Pools

1. Zašto worker pool?
2. Kako određujete broj workers?
3. Kako sprečavate unbounded queue?
4. Kako worker pool reaguje na cancellation?
5. Kako worker pool reaguje na panic?
6. Kako garantujete graceful shutdown?

---

# 251. Context

1. Ko treba da kreira root context?
2. Zašto context treba prosleđivati kao prvi argument?
3. Kada koristiti `WithCancel`?
4. Kada `WithTimeout`?
5. Zašto ne treba koristiti context kao generički state container?
6. Kako cancellation propagira kroz call graph?

---

# 252. Performance Questions

1. Kako biste dokazali da je mutex bottleneck?
2. Kako biste merili channel contention?
3. Kako biste analizirali goroutine blocking?
4. Kako biste odredili optimalan worker count?
5. Kako razlikujete CPU-bound i I/O-bound workload?

---

# 253. Production Questions

1. Kako dizajnirate graceful shutdown?
2. Šta se dešava sa in-flight requests?
3. Šta se dešava sa queued jobs?
4. Kako sprečavate retry storm?
5. Kako implementirate backpressure?
6. Kako merite queue saturation?
7. Kako detektujete goroutine leaks u production-u?

---

# 254. Architecture Interview Challenge

### Scenario

Imate:

```text
API
 ↓
Queue
 ↓
Workers
 ↓
Database
 ↓
External API
```

System mora podržati:

```text
50,000 requests/sec
```

External API podržava:

```text
500 concurrent requests
```

Database:

```text
100 connections
```

Queue:

```text
10,000 jobs
```

Pitanje:

> Kako biste dizajnirali concurrency model?

Ne postoji samo jedan tačan odgovor.

Potrebno je objasniti:

```text
worker count
DB concurrency
external API concurrency
queue behavior
backpressure
timeout
retry
cancellation
shutdown
observability
```

---

# 255. Architecture Reasoning Example

Mogući model:

```text
                 HTTP
                  │
                  ▼
             Admission
                  │
                  ▼
            Bounded Queue
                  │
                  ▼
             Worker Pool
                  │
          ┌───────┴────────┐
          ▼                ▼
       DB Limit       API Limit
       = 100          = 500
```

Ovde postoje najmanje tri različita concurrency budgets:

```text
HTTP concurrency
worker concurrency
downstream concurrency
```

To je mnogo realističniji model od:

```text
worker count = system capacity
```

---

# 256. Final Practical Project

Za kompletiranje Module 4 preporučuje se izrada jednog production-style projekta.

## Project — Concurrent Job Processing Platform

Architecture:

```text
                  ┌───────────────┐
                  │   HTTP API    │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │ Admission     │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │ Bounded Queue  │
                  └───────┬───────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │   Worker Pool    │
                 └────────┬─────────┘
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Database     External API   Cache
             │            │            │
             └────────────┼────────────┘
                          ▼
                     Result Store
```

Required features:

```text
bounded concurrency
backpressure
cancellation
timeouts
retry
idempotency
graceful shutdown
metrics
race-free state
tests
benchmarks
profiling
```

---

# 257. Project — Correctness Requirements

System mora garantovati:

```text
No data races
No double acknowledgement
No work after shutdown
No unbounded queue growth
No uncontrolled goroutine creation
No retry after cancellation
No resource usage beyond configured limits
```

Ovo su system invariants.

---

# 258. Project — Performance Requirements

Treba meriti:

```text
throughput
p50 latency
p95 latency
p99 latency
queue wait time
worker utilization
DB wait time
external API wait time
CPU
memory
allocations
goroutine count
mutex contention
blocking
```

---

# 259. Project — Failure Scenarios

Obavezno testirati:

```text
database unavailable
external API unavailable
external API slow
queue full
worker panic
context cancellation
request timeout
shutdown during active work
shutdown with queued work
repeated retries
duplicate job
```

---

# 260. Module 4 Completion Criteria

Module 4 može se smatrati završenim kada čitalac može samostalno da:

### Understand

* [ ] objasni Go concurrency model;
* [ ] objasni scheduler;
* [ ] objasni memory-model implications;
* [ ] objasni synchronization semantics.

### Design

* [ ] dizajnira worker pool;
* [ ] dizajnira pipeline;
* [ ] dizajnira fan-in/fan-out;
* [ ] dizajnira bounded concurrency;
* [ ] dizajnira backpressure;
* [ ] dizajnira cancellation;
* [ ] dizajnira graceful shutdown.

### Implement

* [ ] implementira race-free concurrent state;
* [ ] implementira cancellable workers;
* [ ] implementira bounded queues;
* [ ] implementira retry sa backoff-om;
* [ ] implementira rate limiting;
* [ ] implementira concurrent aggregation.

### Debug

* [ ] koristi race detector;
* [ ] analizira deadlock;
* [ ] analizira goroutine leak;
* [ ] analizira lock contention;
* [ ] koristi goroutine/block/mutex profile.

### Optimize

* [ ] benchmark-uje concurrency design;
* [ ] pronalazi bottleneck;
* [ ] određuje concurrency limits;
* [ ] razlikuje CPU i I/O bottleneck.

### Production

* [ ] definiše failure semantics;
* [ ] definiše backpressure;
* [ ] definiše retry policy;
* [ ] definiše shutdown semantics;
* [ ] uvodi observability;
* [ ] razume capacity planning.

---

# 261. Final Module 4 Checklist

```text
[ ] Goroutines
[ ] Goroutine lifecycle
[ ] Scheduler
[ ] G/M/P
[ ] Preemption
[ ] Work stealing
[ ] Channels
[ ] Channel ownership
[ ] Select
[ ] Fan-in
[ ] Fan-out
[ ] Pipelines
[ ] Mutex
[ ] RWMutex
[ ] WaitGroup
[ ] Once
[ ] Cond
[ ] Atomics
[ ] Context
[ ] Cancellation
[ ] Timeouts
[ ] Deadlocks
[ ] Livelocks
[ ] Starvation
[ ] Data races
[ ] Happens-before
[ ] Memory visibility
[ ] Worker pools
[ ] Bounded concurrency
[ ] Backpressure
[ ] Rate limiting
[ ] Retry
[ ] Idempotency
[ ] Circuit breakers
[ ] Graceful shutdown
[ ] Goroutine leak detection
[ ] Race testing
[ ] Stress testing
[ ] Benchmarks
[ ] Profiling
[ ] Load testing
[ ] Observability
[ ] Production architecture
[ ] Distributed concurrency
```

---

# 262. The Final Mental Model

Kada se sve prethodno znanje objedini, concurrent Go system treba posmatrati ovako:

```text
                           SYSTEM
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
         Execution         Shared State      Work
             │                │                │
             ▼                ▼                ▼
        Goroutines        Ownership        Queue
        Scheduler         Isolation        Pipeline
        Parallelism       Synchronization  Workers
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                         Coordination
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                 Channels   Mutex     Atomic
                    │         │         │
                    └─────────┼─────────┘
                              ▼
                         Lifecycle
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                Cancel     Timeout    Shutdown
                              │
                              ▼
                           Capacity
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
               Backpressure Limits  Rate Limit
                              │
                              ▼
                           Failure
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                  Error     Retry     Recovery
                              │
                              ▼
                        Observability
                              │
                              ▼
                         Performance
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                 Measure   Profile    Benchmark
                              │
                              ▼
                           Testing
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                   Race     Stress    Load
                              │
                              ▼
                         Production
```

Ovo je kompletan conceptual model Go concurrency-ja na naprednom nivou.

---

# 263. Final Principle

Najvažnija lekcija Module 4 nije:

```text
"Learn more concurrency primitives."
```

Već:

> **Design concurrency around ownership, lifecycle, synchronization, capacity, cancellation and failure semantics.**

Dobro napisan concurrent program nije onaj koji koristi mnogo goroutines.

Dobro napisan concurrent program je onaj čije ponašanje možemo objasniti.

Možemo odgovoriti:

```text
Who owns the state?
Who performs the work?
Who coordinates the workers?
Who can block?
Who can cancel?
Who can fail?
Who closes?
Who waits?
How much work can exist?
What happens under overload?
What happens during shutdown?
How do we know it is correct?
How do we know it is fast enough?
```

Ako na ova pitanja postoje jasni odgovori, concurrency architecture je verovatno dobro postavljena.

---

# 264. End of Module 4

Ovim se završava:

```text
golang/concurrency/docs/module-4/README.md
```

Module 4 je predstavljao završni, napredni sloj Go concurrency-ja:

```text
Basic Concurrency
        │
        ▼
Coordination
        │
        ▼
Synchronization
        │
        ▼
Cancellation
        │
        ▼
Scheduler & Parallelism
        │
        ▼
Memory Model
        │
        ▼
Advanced Patterns
        │
        ▼
Performance
        │
        ▼
Testing
        │
        ▼
Production Architecture
```

Kroz sva četiri modula, cilj je bio da se concurrency ne posmatra kao kolekcija izolovanih API-ja, već kao disciplina koja obuhvata:

```text
execution
coordination
state management
synchronization
lifecycle
capacity
failure handling
observability
performance
correctness
```

---

# 265. Concurrency Learning Path — Completed

Kompletna putanja:

```text
Module 1
   │
   ├── Goroutines
   ├── Channels
   ├── Channel Directions
   ├── Range
   └── Close
   │
   ▼
Module 2
   │
   ├── Select
   ├── WaitGroup
   ├── Worker Pools
   ├── Pipelines
   ├── Fan-Out
   └── Fan-In
   │
   ▼
Module 3
   │
   ├── Mutex
   ├── RWMutex
   ├── Once
   ├── Timeouts
   ├── Cancellation
   ├── Scheduler
   ├── GOMAXPROCS
   └── Parallelism
   │
   ▼
Module 4
   │
   ├── Advanced Concurrency
   ├── Memory Model
   ├── Atomics
   ├── Lock-Free Concepts
   ├── Advanced Patterns
   ├── Backpressure
   ├── Performance
   ├── Testing
   ├── Production Design
   └── Distributed Concurrency
```

---

# 266. End State

Nakon završetka ovog materijala, čitalac više ne bi trebalo da razmišlja samo:

```text
"How do I start a goroutine?"
```

nego:

```text
"What concurrency model does this system require?"
```

To predstavlja prelazak sa:

```text
Go concurrency user
```

na:

```text
Go concurrency engineer
```

i predstavlja završetak:

```text
golang/concurrency/
```
