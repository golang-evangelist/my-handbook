# Senior+ — Interview Questions

# Interview Questions — Senior

> **Fajl:** `extras/01-interview-questions/module-1/07-senior-plus.md`
> 
> **Nivo:** Senior+
> 
> **Oblast:** #1 — Concurrency Fundamentals

---

## 1. Concurrency architecture i sistemsko razmišljanje

Na **Senior+** nivou pitanja više nisu usmerena samo na poznavanje `goroutine`, `channel`, `sync.Mutex` ili `select`.

Očekuje se sposobnost da kandidat:

* modeluje concurrency problem,
* identifikuje invariants,
* definiše ownership,
* projektuje lifecycle,
* kontroliše concurrency,
* uvede backpressure,
* predvidi failure modes,
* proceni trade-off između throughput-a, latency-ja i reliability-ja,
* i objasni zašto je određena arhitektura bolja od alternativa.

Senior+ kandidat ne treba samo da zna **kako** nešto implementirati, već i **zašto je određeni concurrency model ispravan za dati sistem**.

---

### Pitanje 1

**Kako bi definisao concurrency model sistema pre nego što napišeš prvi red Go koda?**

### Odgovor

Prvo bih definisao sistemske invariants i lifecycle, a tek onda izabrao Go concurrency primitive.

Minimalna analiza bi obuhvatila:

```text
1. Ko proizvodi posao?
2. Ko poseduje posao?
3. Ko obrađuje posao?
4. Koji delovi sistema mogu raditi paralelno?
5. Koji delovi moraju biti serijalizovani?
6. Koji je maksimalni concurrency?
7. Gde se posao bufferyzuje?
8. Ko kontroliše backpressure?
9. Kako se propagira cancellation?
10. Kako se sistem gasi?
11. Šta se dešava pri failure-u?
12. Koje garancije sistem daje?
```

Na primer, za payment processing servis:

```text
Request
   │
   ▼
Validation
   │
   ▼
Admission Control
   │
   ▼
Queue
   │
   ▼
Workers
   │
   ▼
Payment Provider
   │
   ▼
Result
```

Pre implementacije moramo znati:

```text
ordering?
at-most-once?
at-least-once?
idempotency?
bounded concurrency?
timeout?
retry?
durability?
shutdown semantics?
```

Tek nakon toga biramo:

```text
channels
mutexes
atomics
worker pools
context
queues
```

Drugim rečima:

> **Concurrency primitive je implementacioni detalj sistema čiji model mora biti definisan pre same primitive.**

---

### Pitanje 2

**Koje concurrency invariants bi smatrao najvažnijim u production sistemu?**

### Odgovor

Invariant predstavlja uslov koji mora ostati tačan tokom celog lifecycle-a sistema.

Na primer:

```text
activeWorkers <= maxWorkers
```

ili:

```text
queueLength <= queueCapacity
```

ili:

```text
closed(jobChannel) => no producer may send
```

Kod finansijskih sistema možemo imati mnogo važnije poslovne invariants:

```text
accountBalance >= reservedAmount
```

ili:

```text
transactionID is processed at most once
```

ako sistem zahteva at-most-once semantiku.

Kod concurrent sistema posebno bih proveravao:

### Ownership invariant

Jedan resurs mora imati jasno definisanog owner-a.

### Lifecycle invariant

Svaka goroutine mora imati definisan način završetka.

### Capacity invariant

Concurrency i queue kapacitet moraju biti ograničeni.

### Synchronization invariant

Shared mutable state mora imati definisan synchronization mechanism.

### Ordering invariant

Ako redosled ima poslovni značaj, mora biti eksplicitno garantovan.

Senior+ developer ne bi trebalo da kaže samo:

> "Ovo je thread-safe."

Bolje pitanje je:

> "Koji invariant ovaj synchronization mechanism štiti?"

---

### Pitanje 3

**Zašto je ownership važniji od same upotrebe concurrency primitive?**

### Odgovor

Zato što concurrency primitive može sprečiti data race, ali ne može sama definisati ko je odgovoran za lifecycle i semantiku resursa.

Na primer:

```go
jobs := make(chan Job)

go producer(jobs)
go worker(jobs)
```

Odmah se postavlja pitanje:

> Ko zatvara `jobs`?

Ako producer-i nisu jasno definisani kao owner-i:

```text
Producer A ─┐
Producer B ─┼──> jobs
Producer C ─┘
```

onda proizvoljno zatvaranje channel-a može izazvati:

```text
panic: send on closed channel
```

Još važnije, ownership određuje:

```text
ko kreira
ko menja
ko zatvara
ko otkazuje
ko čeka
ko oslobađa
```

Dobro dizajniran concurrent sistem ima eksplicitnu ownership mapu.

Na primer:

```text
Component         Owns
────────────────────────────
Producer          input lifecycle
Queue             queued jobs
Worker pool       worker lifecycle
Worker            active job
Coordinator       shutdown
```

Ovakav model značajno smanjuje broj implicitnih concurrency pretpostavki.

---

### Pitanje 4

**Kako bi objasnio razliku između concurrency modela i concurrency primitive?**

### Odgovor

Concurrency model opisuje **kako sistem organizuje paralelni rad**.

Primitive predstavljaju konkretne mehanizme pomoću kojih taj model implementiramo.

Na primer:

```text
Concurrency Model
       │
       ├── Worker Pool
       ├── Pipeline
       ├── Fan-Out/Fan-In
       ├── Actor-like ownership
       └── Shared-state coordination
                │
                ▼
         Go Primitives
                │
       ├── goroutines
       ├── channels
       ├── mutex
       ├── atomic
       ├── context
       └── select
```

Worker pool nije isto što i channel.

Možemo implementirati worker pool pomoću channel-a:

```text
jobs channel
    ↓
workers
```

ali channel sam po sebi ne predstavlja worker pool.

Isto tako, mutex nije concurrency model.

Mutex je mehanizam za koordinaciju pristupa shared state-u.

Senior+ kandidat mora razlikovati ove nivoe apstrakcije.

---

### Pitanje 5

**Kako bi procenio da li sistem treba concurrency ili parallelism?**

### Odgovor

Prvo bih identifikovao prirodu workload-a.

### I/O-bound workload

Na primer:

```text
HTTP requests
database queries
external APIs
network operations
```

Može imati smisla koristiti visok nivo concurrency-ja jer goroutine-e često čekaju.

Model:

```text
G1 ── waiting ──┐
G2 ── waiting ──┤
G3 ── waiting ──┤── I/O
G4 ── running ──┘
```

### CPU-bound workload

Na primer:

```text
encryption
compression
image processing
complex calculations
```

Ovde je relevantniji broj CPU execution resources.

Ako imamo:

```text
CPU cores = 8
```

nije automatski korisno pokrenuti:

```text
1000 CPU-heavy workers
```

Jer će veliki broj njih konkurisati za ograničen CPU kapacitet.

Senior+ procena zato mora uzeti u obzir:

```text
workload type
CPU capacity
I/O latency
memory
contention
downstream limits
```

---

### Pitanje 6

**Zašto "više goroutine-a" nije validna strategija za povećanje throughput-a?**

### Odgovor

Zato što throughput često ograničava downstream bottleneck.

Pretpostavimo:

```text
Workers = 10
DB capacity = 100 concurrent queries
```

Povećamo:

```text
Workers = 1,000
```

ali DB ostaje:

```text
DB capacity = 100
```

Dobijamo:

```text
1,000 workers
      │
      ▼
100 DB operations
      │
      ▼
900 waiting
```

Povećanjem concurrency-ja možemo dobiti:

* više contention-a,
* veći memory footprint,
* veći scheduling overhead,
* veći queueing delay,
* veći downstream pressure,
* više timeout-a.

Još gori scenario:

```text
more workers
    ↓
more requests
    ↓
downstream overload
    ↓
higher latency
    ↓
timeouts
    ↓
retries
    ↓
even more requests
```

Zato je cilj:

> **optimalan concurrency, a ne maksimalan concurrency.**

---

## Pitanje 7

**Kako bi projektovao bounded concurrency za servis koji poziva eksterni API?**

### Odgovor

Ne bih dozvolio da svaki incoming request direktno kreira nekontrolisanu goroutine koja poziva eksterni API.

Umesto toga bih uveo concurrency limit.

Na primer:

```text
Incoming Requests
       │
       ▼
Admission Control
       │
       ▼
Concurrency Limit = N
       │
       ▼
External API
```

Jedan model može koristiti buffered channel kao semaphore:

```go
sem := make(chan struct{}, 20)
```

Pre poziva:

```go
sem <- struct{}{}
defer func() {
    <-sem
}()
```

Time ograničavamo broj istovremenih operacija.

Ali senior+ analiza ide dalje.

Moramo definisati:

```text
Šta se dešava kada je limit dostignut?
```

Opcije su:

```text
block
timeout
reject
queue
shed load
```

Ako incoming traffic može biti neograničen, samo semaphore nije dovoljan.

Možemo dobiti:

```text
10,000 requests
       │
       ▼
10 active API calls
       │
       ▼
9,990 waiting goroutines
```

Zato treba kontrolisati i:

```text
concurrency
queue capacity
request lifetime
```

Concurrency limit bez bounded waiting prostora može samo pomeriti bottleneck.

---

### Pitanje 8

**Kako bi objasnio razliku između backpressure-a i load shedding-a?**

### Odgovor

**Backpressure** znači da sistem usporava upstream kada downstream ne može da obradi više posla.

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

Kada se queue napuni:

```text
Producer
   │
   X
   │
Queue full
```

producer može biti blokiran.

To je backpressure.

**Load shedding** znači da sistem namerno odbacuje deo posla kada nema dovoljno kapaciteta.

Na primer:

```text
Queue full
    │
    ▼
Reject request
```

ili:

```text
Drop low-priority job
```

Backpressure:

```text
"Uspori."
```

Load shedding:

```text
"Ne možemo da prihvatimo ovaj posao."
```

U production sistemima često su potrebna oba.

---

### Pitanje 9

**Zašto je unbounded concurrency architectural smell?**

### Odgovor

Zato što sistem tada nema jasno definisanu gornju granicu potrošnje resursa.

Na primer:

```go
for _, req := range requests {
    go process(req)
}
```

Ako `requests` sadrži:

```text
100 requests
```

dobijamo približno:

```text
100 goroutines
```

Ako sadrži:

```text
1,000,000 requests
```

potencijalno dobijamo:

```text
1,000,000 goroutines
```

Čak i ako pojedinačna goroutine nije skupa, ukupna količina može postati problem.

Još važnije:

```text
goroutine count
     ↓
downstream requests
     ↓
connections
     ↓
memory
     ↓
queues
```

ne postoji izolovan concurrency budget.

Production sistem treba da ima eksplicitno definisane granice.

Na primer:

```text
max workers      = 100
max queue        = 10,000
max DB requests  = 50
max API requests = 20
```

To omogućava predvidljiv behavior pod load-om.

---

### Pitanje 10

**Kako bi napravio concurrency budget za production servis?**

### Odgovor

Concurrency budget bih posmatrao kao skup ograničenja kroz ceo dependency graph.

Na primer:

```text
HTTP Service
     │
     ├── DB
     ├── Redis
     └── Payment API
```

Možemo imati:

```text
Service concurrency = 500
DB concurrency      = 100
Redis concurrency   = 200
Payment API         = 20
```

Ne znači da svaki request sme da koristi sve resurse.

Ako svaki request istovremeno zahteva payment API:

```text
500 requests
    │
    ▼
Payment API
    │
    ▼
limit = 20
```

dobijamo queueing.

Zato concurrency budget mora pratiti dependency constraints.

Idealno:

```text
service budget
     │
     ├── endpoint A
     ├── endpoint B
     └── endpoint C
             │
             ▼
        dependency budget
```

Posebno bih razmotrio različite prioritete.

Na primer:

```text
Critical payment operation
        >
Analytics operation
        >
Background maintenance
```

Ako sistem uđe u overload, ne bi trebalo da sve vrste posla budu tretirane identično.

---

### Pitanje 11

**Šta znači da je concurrency sistem "bounded", i zašto je to važno?**

### Odgovor

Bounded sistem ima eksplicitno ograničene resurse i količinu rada koja može istovremeno biti aktivna.

Na primer:

```text
workers ≤ 50
queue ≤ 5,000
DB connections ≤ 100
external API calls ≤ 20
```

To omogućava da sistem ima predvidljivo ponašanje.

Nasuprot tome:

```text
incoming requests
       │
       ▼
go process()
       │
       ▼
unbounded goroutines
       │
       ▼
unbounded resource consumption
```

može dovesti do:

```text
memory exhaustion
connection exhaustion
queue explosion
timeouts
cascading failures
```

Bounded concurrency nije samo optimization.

To je **stability mechanism**.

---

### Pitanje 12

**Kako bi objasnio razliku između throughput-a, latency-ja i concurrency-ja?**

### Odgovor

To su povezane, ali različite metrike.

### Concurrency

Koliko operacija je trenutno aktivno.

```text
10 active requests
```

### Throughput

Koliko operacija sistem obrađuje u jedinici vremena.

```text
1,000 requests/sec
```

### Latency

Koliko traje pojedinačna operacija.

```text
P95 latency = 200ms
```

Povećanje concurrency-ja može povećati throughput samo do određene granice.

Nakon saturation point-a:

```text
concurrency ↑
      │
      ▼
contention ↑
      │
      ▼
latency ↑
      │
      ▼
throughput stagnira ili opada
```

Zato optimalna tačka često izgleda ovako:

```text
                saturation
                   │
throughput         │
   ▲          ┌────┘
   │        /
   │      /
   │    /
   │  /
   └──────────────────> concurrency
```

Senior+ developer treba da zna da **više concurrency-ja ne znači linearno veći throughput**.

---

### Pitanje 13

**Kako bi identifikovao saturation point concurrent sistema?**

### Odgovor

Eksperimentalno.

Povećavao bih concurrency postepeno i merio:

```text
throughput
P50 latency
P95 latency
P99 latency
CPU
memory
GC
queue depth
error rate
timeouts
downstream latency
```

Na primer:

```text
Workers    Throughput    P99
--------------------------------
10         5k/s          40ms
20         9k/s          50ms
40         16k/s         70ms
80         17k/s         150ms
160        16k/s         500ms
```

Ovde se vidi da:

```text
80 → 160 workers
```

ne povećava throughput, ali dramatično povećava latency.

Saturation point je približno oko:

```text
40–80 workers
```

zavisno od SLO-a.

Ovakva odluka treba da bude zasnovana na merenju, a ne na intuiciji.

---

### Pitanje 14

**Šta bi smatrao ozbiljnim signalom da concurrent sistem nije dobro dizajniran?**

### Odgovor

Neki od najvažnijih signala su:

```text
goroutine count constantly increasing
queue depth constantly increasing
unbounded goroutine creation
frequent timeout spikes
retry storms
high mutex contention
workers permanently blocked
shutdown hangs
large number of leaked goroutines
race detector findings
high P99 under moderate load
downstream overload caused by upstream concurrency
```

Posebno je opasan sistem koji radi dobro pod normalnim load-om, ali se raspada pri overload-u.

Na primer:

```text
Normal:
100 req/s → healthy

Overload:
150 req/s → latency explodes
```

To znači da sistem nema adekvatan overload policy.

Robustan sistem bi trebalo da ima definisano ponašanje:

```text
capacity reached
      │
      ├── queue
      ├── throttle
      ├── reject
      └── shed
```

a ne da samo nastavi da prihvata posao dok ne ostane bez resursa.

---

## Senior+ princip

Najvažniji pomak od **Senior** ka **Senior+** nivou je prelazak sa:

> "Kako da napišem concurrent kod?"

na:

> "Kako da projektujem concurrent sistem koji ostaje korektan, bounded, observable i predvidljiv pod normalnim radom, overload-om i failure-om?"

U sledećim delovima ćemo ovaj model produbiti kroz:

* concurrency contracts,
* ordering guarantees,
* delivery semantics,
* idempotency,
* cancellation propagation,
* structured concurrency,
* failure isolation,
* distributed concurrency,
* i production-grade concurrency architecture.

---

Na Senior+ nivou concurrency se više ne posmatra samo kroz pitanje:

> "Da li je pristup memoriji thread-safe?"

Potrebno je definisati **šta sistem garantuje korisniku ili drugim komponentama**.

To znači da concurrency dizajn mora imati eksplicitni contract koji određuje:

* ownership,
* ordering,
* visibility,
* delivery semantics,
* cancellation,
* retry behavior,
* idempotency,
* failure semantics.

---

### Pitanje 15

**Šta je concurrency contract i zašto je važan?**

### Odgovor

Concurrency contract definiše koje garancije komponenta daje kada je koristi više concurrent izvršnih tokova.

Na primer, API može garantovati:

```text
- bezbedan concurrent pristup
- očuvanje redosleda
- at-most-once obradu
- thread-safe read operacije
- concurrent write operacije
```

Ali sama tvrdnja:

> "Ovaj tip je thread-safe."

nije dovoljno precizna.

Na primer:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}
```

Možemo napraviti:

```go
func (c *Counter) Inc()
func (c *Counter) Value() int
```

i garantovati da su pojedinačne operacije thread-safe.

Ali to ne znači da je sledeći kod atomic kao celina:

```go
if counter.Value() < limit {
    counter.Inc()
}
```

Između `Value()` i `Inc()` može doći drugi goroutine.

Dakle, contract pojedinačnih metoda nije isto što i atomicity kompletnog workflow-a.

Senior+ kandidat mora razlikovati:

```text
operation-level safety
        ≠
workflow-level correctness
```

---

### Pitanje 16

**Kako bi dokumentovao concurrency contract jednog Go tipa?**

### Odgovor

Dokumentacija bi trebalo eksplicitno da kaže:

```text
1. Da li je tip bezbedan za concurrent upotrebu?
2. Koje metode mogu biti pozivane concurrent?
3. Da li return vrednosti predstavljaju snapshot?
4. Da li caller dobija ownership nad vraćenim podacima?
5. Da li redosled ima garanciju?
6. Da li metode mogu blokirati?
7. Kako se radi shutdown?
8. Da li metoda podržava cancellation?
```

Na primer:

```go
// Cache is safe for concurrent use.
//
// Get may be called concurrently with Set.
//
// Close releases internal resources and must be called once.
//
// After Close returns, no further calls to Get or Set are permitted.
type Cache struct {
    ...
}
```

Ovo je mnogo korisnije od komentara:

```go
// Thread-safe.
```

Jer prvi komentar definiše **contract**, dok drugi samo daje nepreciznu tvrdnju.

---

### Pitanje 17

**Šta znači ordering guarantee u concurrent sistemu?**

### Odgovor

Ordering guarantee definiše kojim redosledom se događaji moraju posmatrati ili obrađivati.

Na primer, imamo:

```text
A → B → C
```

Ako sistem garantuje FIFO ordering, obrada mora poštovati:

```text
A
↓
B
↓
C
```

Ali concurrent sistem može prirodno proizvesti:

```text
A ────────┐
          ├──> processing
B ──┐     │
    └─────┘
C ───────────────>
```

pa rezultat može biti:

```text
B
A
C
```

Ako poslovna logika zahteva ordering:

```text
A before B
```

onda paralelna obrada mora biti ograničena ili mora postojati mehanizam za reorderovanje.

---

### Pitanje 18

**Da li channel u Go automatski garantuje ordering?**

### Odgovor

Ne treba iz toga izvesti opštu tvrdnju da channel garantuje ordering celog sistema.

Ako jedan producer šalje:

```go
jobs <- A
jobs <- B
jobs <- C
```

receiver koji prima iz tog channel-a dobija vrednosti redom kojim su poslate:

```text
A → B → C
```

Ali čim imamo više concurrent producer-a:

```text
Producer A ─┐
            ├──> channel
Producer B ─┘
```

redosled između producer-a može zavisiti od scheduling-a i trenutka slanja.

Još važnije, čak i ako channel redom isporuči:

```text
A
B
C
```

više worker-a može obrađivati:

```text
Worker 1 → A
Worker 2 → B
Worker 3 → C
```

i završiti:

```text
B
C
A
```

Dakle:

```text
channel receive ordering
        ≠
processing completion ordering
```

Ovo je veoma važna razlika u production sistemima.

---

### Pitanje 19

**Kako bi implementirao concurrent obradu uz očuvanje redosleda rezultata?**

### Odgovor

Jedan pristup je da svakom poslu dodelimo sequence number:

```text
Job A → sequence 0
Job B → sequence 1
Job C → sequence 2
```

Workers mogu raditi paralelno:

```text
Worker 1 → A
Worker 2 → B
Worker 3 → C
```

ali rezultat se može reorderovati:

```text
Result 2
Result 0
Result 1
```

u:

```text
Result 0
Result 1
Result 2
```

Model:

```text
             ┌── Worker ── Result 2
Jobs ────────┼── Worker ── Result 0
             └── Worker ── Result 1
                           │
                           ▼
                    Reordering Buffer
                           │
                           ▼
                    Ordered Output
```

Ovo omogućava:

```text
parallel execution
+
ordered publication
```

ali uvodi dodatnu kompleksnost:

* memory usage,
* buffering,
* head-of-line blocking,
* timeout handling,
* missing result handling.

Na primer, ako rezultat `sequence=0` nikada ne stigne:

```text
Result 1
Result 2
Result 3
```

možda ne mogu biti objavljeni jer čekamo:

```text
Result 0
```

Zato ordering ima cenu.

---

### Pitanje 20

**Šta je head-of-line blocking u concurrent sistemu?**

### Odgovor

Head-of-line blocking nastaje kada jedan spor ili zaglavljen element sprečava napredovanje drugih elemenata koji su već spremni.

Na primer:

```text
Queue:

A   B   C   D
│   │   │   │
└── slow
```

Ako sistem mora očuvati strogi ordering:

```text
A → B → C → D
```

a `A` traje 10 sekundi:

```text
A = 10s
B = 10ms
C = 10ms
D = 10ms
```

onda:

```text
B, C, D
```

moraju čekati.

Iako su njihovi rezultati već dostupni.

To predstavlja trade-off:

```text
strict ordering
       vs
parallel progress
```

Senior+ kandidat treba da zna da ordering nije besplatna garancija.

---

### Pitanje 21

**Koje delivery semantics postoje u distributed/concurrent sistemima?**

### Odgovor

Najčešće se govori o:

### At-most-once

Poruka se obrađuje najviše jednom.

```text
0 ili 1 obrada
```

Ako processing padne:

```text
message
   ↓
attempt
   ↓
failure
   ↓
message lost
```

Prednost je izbegavanje duplikata, ali cena može biti gubitak poruke.

---

### At-least-once

Poruka se pokušava obraditi najmanje jednom.

Ako postoji failure:

```text
message
   ↓
processing
   ↓
failure
   ↓
retry
```

može nastati:

```text
processing #1
processing #2
```

Dakle, moguće su duplikate.

Zato je potrebna idempotency strategija.

---

### Exactly-once

Exactly-once semantika je mnogo kompleksnija od:

```text
"pozovemo funkciju samo jednom"
```

U distributed sistemu moramo razmotriti:

```text
processing
commit
acknowledgement
retry
crash
network partition
```

Na primer:

```text
Worker
  │
  ▼
Process payment
  │
  ▼
Payment succeeds
  │
  X
  │
Worker crashes before ACK
```

Sistem ne zna da li treba ponovo da obradi operaciju.

Ako ponovi:

```text
duplicate payment
```

Ako ne ponovi:

```text
possible lost operation
```

Zato "exactly once" često zahteva kombinaciju:

```text
idempotency
+
durability
+
transactional boundaries
+
deduplication
```

---

### Pitanje 22

**Zašto je at-least-once delivery često praktičniji od exactly-once pristupa?**

### Odgovor

Zato što je često jednostavnije garantovati:

```text
message will be retried
```

nego:

```text
message will be processed exactly once
```

Ako imamo:

```text
at-least-once
+
idempotent consumer
```

možemo bezbedno obraditi duplicate delivery.

Na primer:

```text
event ID = 12345
```

Consumer proverava:

```text
already processed 12345?
```

Ako jeste:

```text
ignore duplicate
```

Ako nije:

```text
process
record 12345
```

Tako dobijamo praktičan model:

```text
at-least-once delivery
        +
idempotent processing
        =
effectively safe retries
```

Ali treba biti veoma oprezan sa načinom čuvanja idempotency state-a.

Ako imamo:

```text
process event
save event ID
```

i proces padne između:

```text
process event
      ↓
CRASH
      ↓
save event ID
```

event može biti ponovo obrađen.

Zato idempotency record i poslovna operacija često moraju imati odgovarajuću atomicnost ili transactional boundary.

---

### Pitanje 23

**Šta je idempotency i zašto je posebno važna u concurrent sistemima?**

### Odgovor

Operacija je idempotentna ako njeno ponavljanje ne menja konačni rezultat nakon prvog uspešnog izvršenja.

Formalno:

```text
f(f(x)) = f(x)
```

U distributed sistemima ovo je posebno važno zbog:

```text
retries
timeouts
duplicate delivery
network failures
worker crashes
```

Na primer:

```text
POST /payment
Idempotency-Key: abc123
```

Ako klijent pošalje zahtev:

```text
Request #1
```

i ne dobije odgovor zbog network timeout-a, može poslati:

```text
Request #2
```

Server mora moći da zaključi:

```text
abc123 already processed
```

umesto da izvrši payment drugi put.

Concurrent obrada dodatno komplikuje stvar.

Dva goroutine-a mogu istovremeno dobiti isti zahtev:

```text
G1 ──> key abc123
G2 ──> key abc123
```

Ako oba provere:

```text
not processed
```

pre nego što bilo koji zapiše stanje, oba mogu nastaviti.

Zato idempotency check mora imati odgovarajuću atomicnost.

---

### Pitanje 24

**Kako bi zaštitio idempotency check od race condition-a?**

### Odgovor

Naivni pristup:

```go
if !store.Exists(key) {
    process()
    store.Save(key)
}
```

nije bezbedan.

Možemo imati:

```text
G1                         G2

Exists(key) = false       Exists(key) = false
       │                         │
       ▼                         ▼
    process()                 process()
       │                         │
       ▼                         ▼
   Save(key)                  Save(key)
```

Oba goroutine-a su prošla proveru.

Potrebna je atomicna operacija:

```text
check + reserve
```

na nivou odgovarajućeg storage-a.

Na primer konceptualno:

```text
INSERT idempotency_key
IF key already exists
    reject duplicate
```

ili:

```text
SETNX key
```

pa tek onda processing.

Ali i dalje ostaje pitanje šta se dešava ako:

```text
reserve key
      ↓
process
      ↓
CRASH
```

Zato idempotency state često ima lifecycle:

```text
PENDING
   ↓
COMPLETED
   ↓
EXPIRED
```

i recovery semantics.

---

### Pitanje 25

**Kako bi razlikovao thread-safety od linearizability-ja?**

### Odgovor

Thread-safe ili concurrent-safe API može garantovati da nema data race-a i da je interna struktura konzistentna.

Ali linearizability je jača semantička garancija.

Linearizabilna operacija se ponaša kao da se svaka concurrent operacija izvršila u jednom tačno određenom trenutku između njenog poziva i završetka.

Na primer:

```text
G1: Set(10)
G2: Get()
```

Ako `Get()` počne nakon što je `Set(10)` završio, linearizabilan sistem mora vratiti:

```text
10
```

Ali za concurrent overlap:

```text
G1: ───── Set(10) ─────
G2:    ─── Get() ──────
```

mora postojati konzistentan linearni redosled koji objašnjava rezultat.

Ovo je mnogo jači koncept od:

```text
"nema data race-a"
```

Data-race-free program i linearizabilan API nisu ista stvar.

---

### Pitanje 26

**Zašto data-race-free program može i dalje biti logički neispravan?**

### Odgovor

Zato što synchronization može zaštititi pojedinačne operacije, ali ne i kombinaciju operacija.

Na primer:

```go
if balance >= amount {
    balance -= amount
}
```

Ako je pristup `balance` pravilno zaključan pojedinačno, možemo izbeći data race.

Ali ako se lock uzima odvojeno:

```go
mu.Lock()
ok := balance >= amount
mu.Unlock()

if ok {
    mu.Lock()
    balance -= amount
    mu.Unlock()
}
```

dva goroutine-a mogu oba dobiti:

```text
ok = true
```

i oba izvršiti withdrawal.

Nema nužno data race-a.

Ali poslovni invariant:

```text
balance >= 0
```

može biti prekršen.

Dakle:

```text
data-race-free
        ≠
business-correct
```

Senior+ developer mora analizirati **atomicity poslovne operacije**, a ne samo memorijsku bezbednost.

---

### Pitanje 27

**Kako bi definisao atomicity boundary u concurrent business operation-u?**

### Odgovor

Atomicity boundary treba da obuhvati sve operacije koje moraju izgledati kao jedna nedeljiva poslovna akcija.

Na primer:

```text
Check balance
      +
Reserve amount
```

moraju biti jedna atomicna operacija ako dva concurrent zahteva ne smeju rezervisati isti novac.

Model:

```text
┌───────────────────────────┐
│ Atomic business operation │
│                           │
│  check balance            │
│  reserve amount           │
│  update state             │
└───────────────────────────┘
```

Ako granicu postavimo pogrešno:

```text
check
  ↓
unlock
  ↓
update
```

drugi goroutine može promeniti state između tih koraka.

Senior+ kandidat treba da pita:

> "Koje poslovne operacije moraju biti linearizovane?"

a ne samo:

> "Gde ćemo staviti mutex?"

---

### Pitanje 28

**Kako bi objasnio odnos između concurrency-ja i business correctness-a u payment sistemu?**

### Odgovor

Pretpostavimo da korisnik ima:

```text
balance = 100
```

i dva concurrent payment zahteva:

```text
Payment A = 80
Payment B = 80
```

Ako oba goroutine-a pročitaju:

```text
balance = 100
```

pre nego što bilo koji izvrši update:

```text
G1 → sees 100
G2 → sees 100
```

oba mogu zaključiti:

```text
100 >= 80
```

i oba odobriti payment.

Rezultat:

```text
160 spent
```

iako je bilo samo:

```text
100 available
```

Ovo nije samo concurrency bug.

To je **business invariant violation**.

Zato fintech concurrency sistemi zahtevaju da developer razume:

```text
concurrency
+
transactions
+
locking
+
idempotency
+
ordering
+
state transitions
```

---

### Pitanje 29

**Da li bi za svaki business invariant koristio mutex?**

### Odgovor

Ne.

Mutex štiti memory state unutar procesa.

Ako imamo više instanci servisa:

```text
Service A ─┐
Service B ─┼──> Database
Service C ─┘
```

lokalni mutex u Service A ne štiti isti state od Service B.

Zato je granica koordinacije ključna.

```text
single process
    ↓
mutex may be sufficient
```

ali:

```text
multiple processes
    ↓
distributed coordination required
```

Moguća rešenja uključuju:

* database transaction,
* row-level locking,
* optimistic concurrency control,
* atomic update,
* distributed lock kada je zaista potreban,
* serialization kroz queue/partition.

Senior+ developer mora znati gde concurrency control treba da živi.

---

### Pitanje 30

**Kako bi odlučio da li concurrency treba kontrolisati u aplikaciji ili u storage sloju?**

### Odgovor

Zavisi od invariant-a.

Ako želimo ograničiti broj concurrent external API poziva:

```text
application layer
```

je prirodno mesto:

```text
Service
   │
   ▼
Semaphore / Worker Pool
   │
   ▼
External API
```

Ako želimo zaštititi database invariant:

```text
account balance
transaction state
inventory quantity
```

storage/transaction layer može biti autoritativan:

```text
Service
   │
   ▼
Transaction
   │
   ▼
Database invariant
```

Za shared business state ne treba se oslanjati samo na lokalni Go mutex ako više procesa može menjati isti podatak.

Praktično pravilo:

> **Concurrency control treba postaviti što bliže resursu čiji invariant štitimo, dok se workload-level concurrency može kontrolisati na application nivou.**

---

## Ključna Senior+ pitanja nakon ovog dela

Nakon ovog dela kandidat treba da ume da razmišlja o:

```text
Concurrency contract
        ↓
Ownership
        ↓
Ordering
        ↓
Delivery semantics
        ↓
Idempotency
        ↓
Atomicity
        ↓
Business invariants
        ↓
Distributed coordination
```

To predstavlja prelazak sa lokalnog Go concurrency-ja na **sistemsko concurrency razmišljanje**.

# `extras/01-interview-questions/module-1/07-senior-plus.md` — Deo #3/12

## 3. Backpressure, bounded concurrency i kontrola resursa

Kod ozbiljnih concurrent sistema nije dovoljno omogućiti paralelno izvršavanje. Potrebno je kontrolisati **koliko** posla sistem može istovremeno da prihvati i obradi.

Ako producer može da generiše posao brže nego što workers mogu da ga obrade, sistem mora imati strategiju za:

* backpressure,
* bounded queues,
* bounded concurrency,
* overload handling,
* admission control,
* cancellation,
* timeouts,
* graceful degradation.

---

### Pitanje 31

**Šta je backpressure i zašto je važan u concurrent sistemima?**

### Odgovor

Backpressure je mehanizam kojim sporiji deo sistema signalizira bržem delu da mora usporiti produkciju ili prihvat novih poslova.

Bez backpressure-a možemo imati:

```text
Producer
   │
   │  10.000 jobs/s
   ▼
Queue
   │
   │  1.000 jobs/s
   ▼
Workers
```

Ako producer konstantno generiše:

```text
10.000 jobs/s
```

a sistem obrađuje:

```text
1.000 jobs/s
```

queue će rasti:

```text
1s  →  9.000 pending
2s  → 18.000 pending
3s  → 27.000 pending
...
```

Pre ili kasnije dolazimo do:

```text
memory exhaustion
```

ili do neprihvatljivo velikog latency-ja.

Backpressure menja ponašanje producer-a:

```text
Producer
   │
   ▼
Admission Control
   │
   ▼
Bounded Queue
   │
   ▼
Workers
```

Kada sistem dostigne svoj kapacitet, producer mora:

* čekati,
* odbiti posao,
* vratiti grešku,
* blokirati,
* smanjiti rate,
* ili primeniti neku drugu degradation strategiju.

---

### Pitanje 32

**Koja je razlika između bounded i unbounded queue pristupa?**

### Odgovor

Unbounded queue konceptualno dozvoljava da broj pending poslova raste bez unapred definisane granice.

To izgleda jednostavno:

```text
Producer → Queue → Workers
```

ali predstavlja opasan failure mode.

Ako je:

```text
arrival rate > service rate
```

queue će neograničeno rasti.

Kod bounded queue-a postoji eksplicitna granica:

```text
capacity = 1000
```

Kada je queue pun, sistem mora odlučiti šta radi sa novim poslom.

Na primer:

```text
queue full
   │
   ├── block producer
   ├── reject request
   ├── drop oldest
   ├── drop newest
   └── return overload error
```

Prednost bounded pristupa je što memorijska potrošnja i workload mogu imati predvidljivije granice.

---

### Pitanje 33

**Kako bi implementirao bounded worker pool u Go-u?**

### Odgovor

Jedan jednostavan model koristi bounded jobs channel:

```go
jobs := make(chan Job, 100)
```

i ograničen broj worker-a:

```go
const workers = 10
```

Arhitektura:

```text
             ┌── Worker 1
             ├── Worker 2
Producer ───>│
             ├── ...
             └── Worker 10
```

`jobs` channel predstavlja bounded buffer, dok broj goroutine-a definiše maksimalni broj concurrent processing operacija.

Važna osobina je da sistem ne kreira novi goroutine za svaki posao bez granice:

```go
go process(job)
```

za proizvoljan broj poslova može proizvesti ogroman broj goroutine-a.

Umesto toga:

```text
N jobs
   ↓
bounded queue
   ↓
M workers
```

gde je:

```text
M = maksimalan broj concurrent processing operacija
```

Međutim, worker pool sam po sebi nije kompletan backpressure sistem.

Mora se definisati šta se dešava kada je:

```text
jobs channel full
```

---

### Pitanje 34

**Da li buffered channel automatski rešava backpressure?**

### Odgovor

Ne.

Buffered channel samo omogućava određenu količinu privremenog buffering-a.

Na primer:

```go
jobs := make(chan Job, 100)
```

daje:

```text
capacity = 100
```

Ako producer proizvodi brže od worker-a:

```text
Producer
   ↓
[100 slots]
   ↓
Workers
```

prvih 100 poslova može biti prihvaćeno bez blokiranja.

Ali kada se queue napuni:

```text
[████████████████████] full
```

sledeći send može blokirati.

To jeste oblik backpressure-a, ali samo ako je blokiranje prihvatljivo za producer-a.

U nekim sistemima blokiranje HTTP request handler-a nije poželjno. Umesto toga može biti bolje:

```go
select {
case jobs <- job:
    // accepted
default:
    // overloaded
}
```

i vratiti odgovarajuću overload grešku.

Dakle:

```text
buffering
    ≠
complete backpressure strategy
```

---

### Pitanje 35

**Kada bi koristio blocking, a kada rejecting backpressure?**

### Odgovor

Zavisi od prirode workload-a.

### Blocking

Producer čeka dok se ne oslobodi kapacitet.

```text
request
   ↓
queue full
   ↓
wait
   ↓
capacity available
```

Dobro funkcioniše kada:

* caller može da čeka,
* workload nije latency-sensitive,
* gubitak posla nije prihvatljiv,
* upstream prirodno podržava usporavanje.

Ali može izazvati **resource exhaustion**.

Na primer, ako HTTP handler-i čekaju:

```text
10.000 requests
      ↓
10.000 blocked goroutines
```

sistem može postati još sporiji.

---

### Rejecting

Ako nema kapaciteta:

```text
request
   ↓
queue full
   ↓
reject
```

Prednost je što sistem ostaje responsive pod overload-om.

Možemo vratiti:

```text
HTTP 429 Too Many Requests
```

ili internu grešku tipa:

```text
ErrOverloaded
```

Ovo je često zdravije za systems koji moraju imati **bounded resource usage**.

---

### Pitanje 36

**Šta je admission control?**

### Odgovor

Admission control je odluka da li sistem uopšte treba da prihvati novi posao.

Umesto:

```text
accept everything
```

sistem procenjuje:

```text
Can I safely accept this work?
```

Na primer:

```text
Incoming request
       │
       ▼
Admission Control
       │
   ┌───┴────┐
   │        │
 accept   reject
   │        │
   ▼        ▼
 queue     429
```

Admission control može koristiti:

* queue capacity,
* active request count,
* CPU utilization,
* memory pressure,
* downstream capacity,
* rate limits,
* tenant quotas,
* deadlines.

To je važan mehanizam zaštite sistema od overload-a.

---

### Pitanje 37

**Zašto unlimited goroutine creation može predstavljati problem ako su goroutine-i "jeftini"?**

### Odgovor

Goroutine-i jesu relativno lagani u odnosu na OS thread-ove, ali nisu besplatni.

Svaki goroutine zahteva:

* stack space,
* scheduler metadata,
* execution resources,
* memoriju koju koriste njegove lokalne promenljive,
* eventualne objekte koje drži živim.

Ako aplikacija uradi:

```go
for _, job := range jobs {
    go process(job)
}
```

za veoma veliki broj poslova, može doći do ogromnog broja aktivnih goroutine-a.

Još važnije, broj goroutine-a može biti potpuno nepovezan sa kapacitetom downstream sistema.

Na primer:

```text
1.000.000 requests
        ↓
1.000.000 goroutines
        ↓
Database
```

Database možda može obraditi samo:

```text
500 concurrent operations
```

Ostatak predstavlja konkurenciju koju sistem nije u stanju korisno da obradi.

Zato pitanje nije:

> "Koliko goroutine-a Go može da pokrene?"

nego:

> "Koliko concurrent work-a ovaj sistem može bezbedno da obradi?"

---

### Pitanje 38

**Da li broj worker-a treba uvek da bude jednak broju CPU core-ova?**

### Odgovor

Ne.

Optimalan broj worker-a zavisi od karaktera workload-a.

Za CPU-bound posao:

```text
CPU
 ↓
workers
```

previše worker-a može dovesti do nepotrebnog scheduling overhead-a i contention-a.

Za I/O-bound posao:

```text
Worker
   ↓
network I/O
   ↓
wait
```

worker može provoditi veliki deo vremena blokiran na I/O-u, pa može biti korisno imati više concurrent operacija nego CPU core-ova.

Na primer:

```text
CPU-bound:
workers ≈ CPU parallelism

I/O-bound:
workers > CPU count
```

ali ne postoji univerzalna formula.

Potrebno je meriti:

* throughput,
* latency,
* CPU utilization,
* memory,
* downstream saturation,
* queue depth,
* contention.

---

### Pitanje 39

**Šta je resource amplification i kako concurrency može da ga izazove?**

### Odgovor

Resource amplification nastaje kada jedan spoljašnji zahtev proizvodi više internih resursa ili operacija.

Na primer:

```text
1 HTTP request
   ↓
10 goroutines
   ↓
10 DB queries
   ↓
10 external API calls
```

Ako dobijemo:

```text
1.000 HTTP requests
```

možemo proizvesti:

```text
10.000 goroutines
10.000 DB queries
10.000 external API calls
```

Ako svaki sloj sistema dodatno multiplicira concurrency:

```text
Request
   ↓ ×10
Service
   ↓ ×5
Worker
   ↓ ×N
Downstream
```

mali broj ulaznih zahteva može proizvesti ogromno opterećenje.

Zato concurrency limit treba posmatrati kroz **ceo dependency graph**, a ne izolovano po komponentama.

---

### Pitanje 40

**Kako bi sprečio da jedan tenant ili korisnik potroši sav concurrency kapacitet sistema?**

### Odgovor

Globalni limit:

```text
100 concurrent jobs
```

nije dovoljan ako jedan tenant može zauzeti svih 100:

```text
Tenant A → 100
Tenant B → 0
Tenant C → 0
```

Možemo kombinovati:

```text
Global limit
+
Per-tenant limit
```

Na primer:

```text
Global = 100
Tenant = max 10
```

Dobijamo:

```text
Tenant A → 10
Tenant B → 10
Tenant C → 10
...
```

Ovo predstavlja oblik **fairness control-a**.

U naprednijim sistemima mogu se koristiti:

* weighted quotas,
* priority queues,
* token buckets,
* fair scheduling,
* per-tenant worker pools.

Cilj nije samo maksimalan throughput, već i sprečavanje **noisy neighbor** efekta.

---

### Pitanje 41

**Kako deadline propagation utiče na backpressure?**

### Odgovor

Ako request ima deadline:

```text
deadline = T
```

nema smisla beskonačno čekati na queue.

Na primer:

```go
select {
case jobs <- job:
    return nil

case <-ctx.Done():
    return ctx.Err()
}
```

Ako je queue puna, producer čeka samo dok:

```text
queue capacity
```

ili:

```text
context deadline
```

ne nastupi.

To sprečava situaciju:

```text
request
   ↓
queue full
   ↓
wait
   ↓
wait
   ↓
deadline already expired
```

Ako posao više nema korisnu vrednost, njegovo prihvatanje ili obrada samo troši resurse.

Zato su:

```text
backpressure
+
context cancellation
+
deadlines
```

često nerazdvojivi elementi production concurrency dizajna.

---

### Pitanje 42

**Šta je overload protection i zašto je bolje odbiti deo posla nego dozvoliti sistemski kolaps?**

### Odgovor

U overload stanju sistem ima manje kapaciteta nego što workload zahteva:

```text
demand > capacity
```

Ako pokušamo obraditi sve:

```text
queue grows
   ↓
latency grows
   ↓
timeouts
   ↓
retries
   ↓
more load
   ↓
more timeouts
```

Dobijamo **retry storm** i potencijalni cascading failure.

Bolji pristup može biti:

```text
reject excess work
        ↓
preserve capacity
        ↓
existing requests complete
```

To je princip:

> **Controlled rejection is often safer than uncontrolled degradation.**

Sistem možda neće obraditi 100% zahteva, ali može ostati stabilan i obraditi prihvaćeni workload sa predvidljivim latency-jem.

---

## Ključni Senior+ koncepti

Kod dizajniranja concurrent sistema treba razmišljati kroz sledeći model:

```text
Incoming Work
      │
      ▼
Admission Control
      │
      ▼
Bounded Queue
      │
      ▼
Bounded Concurrency
      │
      ▼
Downstream Resource
      │
      ▼
Completion / Cancellation
```

A za svaki korak treba definisati:

```text
capacity
ordering
timeout
cancellation
rejection
retry
fairness
failure semantics
```

Najvažnija ideja ovog dela je:

> **Concurrency bez ograničenja nije skalabilnost.**

Pravi cilj nije maksimalan broj concurrent goroutine-a, već **kontrolisan throughput uz predvidljivu potrošnju resursa i stabilno ponašanje pod overload-om**.

---

Kod ozbiljnih concurrent sistema nije dovoljno znati kako pokrenuti goroutine. Potrebno je znati kako ga **kontrolisano zaustaviti**.

Production sistem mora imati jasno definisan lifecycle:

```text
start
  ↓
running
  ↓
degraded / draining
  ↓
shutdown
  ↓
terminated
```

Posebno je važno da goroutine-i ne ostanu da žive nakon što njihov posao više nije potreban.

---

### Pitanje 43

**Zašto je cancellation važan u concurrent Go aplikacijama?**

### Odgovor

Goroutine koji više nema razloga da nastavi rad predstavlja potrošnju resursa.

Može držati:

* memoriju,
* channel reference,
* database connection,
* network connection,
* timer,
* file descriptor,
* druge objekte koje sprečava od GC-a.

Na primer:

```go
func worker(jobs <-chan Job) {
    for job := range jobs {
        process(job)
    }
}
```

Ako `jobs` nikada ne bude zatvoren, worker može ostati živ neograničeno.

Problem postaje još ozbiljniji kada worker čeka na više izvora:

```text
jobs
  │
  ├── database
  │
  ├── network
  │
  └── timer
```

Cancellation omogućava da lifecycle bude eksplicitno kontrolisan:

```text
work
 ↓
cancel
 ↓
stop waiting
 ↓
release resources
 ↓
exit
```

---

### Pitanje 44

**Koja je razlika između `close(channel)` i `context cancellation`?**

### Odgovor

To su različiti mehanizmi sa različitim semantikama.

`close(channel)` signalizira:

> **Nema više vrednosti koje će biti poslate kroz ovaj channel.**

Na primer:

```go
close(jobs)
```

receiver može završiti:

```go
for job := range jobs {
    process(job)
}
```

kada se channel isprazni.

Nasuprot tome, `context cancellation` signalizira:

> **Operacija više nije potrebna i treba prekinuti rad što je pre moguće.**

Na primer:

```go
select {
case job := <-jobs:
    process(job)

case <-ctx.Done():
    return
}
```

Dakle:

```text
close(channel)
    ↓
producer lifecycle / end of stream

context cancellation
    ↓
consumer/work lifecycle / abort
```

Channel close se prvenstveno odnosi na **stream podataka**, dok context cancellation predstavlja **signal za prekid operacije i propagaciju lifecycle-a**.

---

### Pitanje 45

**Ko treba da zatvara channel?**

### Odgovor

Uobičajeno pravilo je:

> **Sender je odgovoran za zatvaranje channel-a kada zna da više neće biti vrednosti za slanje.**

Na primer:

```go
func producer(out chan<- Job) {
    defer close(out)

    for _, job := range jobs {
        out <- job
    }
}
```

Consumer:

```go
func consumer(in <-chan Job) {
    for job := range in {
        process(job)
    }
}
```

Consumer ne treba proizvoljno da zatvara channel koji mu je dat kao input.

Ako consumer uradi:

```go
close(in)
```

dok producer još pokušava:

```go
in <- job
```

producer može dobiti:

```text
panic: send on closed channel
```

Zato ownership mora biti jasan.

---

### Pitanje 46

**Kako bi dizajnirao worker koji podržava i normalan završetak i cancellation?**

### Odgovor

Worker treba da reaguje na oba događaja:

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

            if err := process(ctx, job); err != nil {
                // handle error
            }
        }
    }
}
```

Postoje dva različita razloga za izlazak:

```text
channel closed
    ↓
normal completion

ctx.Done()
    ↓
cancellation
```

Ovo je važna razlika.

Ako je producer završio:

```text
jobs closed
```

worker može završiti normalno nakon obrade preostalih poslova.

Ako je request otkazan:

```text
ctx.Done()
```

worker može napustiti obradu.

---

### Pitanje 47

**Zašto cancellation mora da bude propagiran kroz call chain?**

### Odgovor

Pretpostavimo:

```text
HTTP request
    ↓
Service
    ↓
Repository
    ↓
Database
```

Ako HTTP request dobije cancellation:

```text
client disconnect
       ↓
request context cancelled
```

a service nastavi da radi:

```text
Service
   ↓
Database query
   ↓
External API
```

sistem troši resurse na posao čiji rezultat više nije potreban.

Ispravan model je:

```text
Request context
       │
       ▼
Service
       │
       ▼
Repository
       │
       ▼
Database / network operation
```

i svaki sloj treba da propagira context gde je relevantno.

Na primer:

```go
func (s *Service) Process(
    ctx context.Context,
    id string,
) error {
    data, err := s.repo.Get(ctx, id)
    if err != nil {
        return err
    }

    return s.external.Call(ctx, data)
}
```

Time cancellation postaje deo lifecycle-a cele operacije.

---

### Pitanje 48

**Da li `context.Context` treba čuvati u strukturi?**

### Odgovor

Uobičajeno ne.

Context predstavlja lifecycle konkretne operacije i treba ga prosleđivati eksplicitno:

```go
func (s *Service) Process(
    ctx context.Context,
    req Request,
) error
```

umesto:

```go
type Service struct {
    ctx context.Context
}
```

Context koji predstavlja jedan request ne treba postati implicitno stanje servisa koji živi duže od tog request-a.

Praktično pravilo:

```text
Context pripada operaciji.
```

Ne treba ga koristiti kao generički dependency container ili mesto za čuvanje proizvoljnog state-a.

---

### Pitanje 49

**Šta je goroutine leak?**

### Odgovor

Goroutine leak nastaje kada goroutine ostane živ iako više ne može ili ne treba da obavlja koristan posao.

Tipičan primer:

```go
func start() {
    ch := make(chan int)

    go func() {
        value := <-ch
        fmt.Println(value)
    }()
}
```

Ako niko nikada ne pošalje:

```go
ch <- value
```

i channel ne dobije drugi način završetka, goroutine ostaje blokiran.

Ako se `start()` poziva veliki broj puta:

```text
start()
start()
start()
...
```

dobijamo:

```text
goroutine
goroutine
goroutine
...
```

koji ostaju blokirani.

Goroutine leak je posebno opasan kada je lifecycle goroutine-a vezan za:

* HTTP request,
* websocket connection,
* subscription,
* queue consumer,
* timer,
* external resource.

---

### Pitanje 50

**Kako bi detektovao goroutine leak?**

### Odgovor

Prvi signal može biti rast broja goroutine-a:

```go
runtime.NumGoroutine()
```

ali sam broj nije dovoljan.

Potrebno je posmatrati:

```text
goroutine count
goroutine profiles
heap
blocked goroutines
channel activity
request lifecycle
```

Go profiler može pokazati gde goroutine-i čekaju.

Na primer, ako veliki broj goroutine-a čeka na:

```text
chan receive
```

ili:

```text
sync.Mutex.Lock
```

to može ukazivati na problem u lifecycle-u ili contention-u.

Važno je razlikovati:

```text
legitimate long-lived goroutines
```

od:

```text
leaked goroutines
```

Server može namerno imati goroutine koji traje tokom celog procesa.

Problem je kada goroutine treba da završi, ali nema putanju ka završetku.

---

### Pitanje 51

**Šta je graceful shutdown?**

### Odgovor

Graceful shutdown znači da sistem prestaje da prihvata novi posao, omogućava postojećem poslu da se završi u dozvoljenom roku i zatim oslobađa resurse.

Tipičan lifecycle:

```text
Running
   │
   ▼
Stop accepting new work
   │
   ▼
Drain existing work
   │
   ▼
Cancel remaining work
   │
   ▼
Close resources
   │
   ▼
Exit
```

Na primer, HTTP servis može prvo prestati da prihvata nove request-e.

Existing request-i dobijaju određeni grace period:

```text
shutdown deadline = 30s
```

Nakon toga:

```text
remaining work
       ↓
cancel
       ↓
force termination
```

Graceful shutdown nije samo:

```go
os.Exit(0)
```

To je koordinisan lifecycle svih concurrent komponenti.

---

### Pitanje 52

**Kako bi koordinisao shutdown više goroutine-a?**

### Odgovor

Jedan čest pristup koristi `context` zajedno sa `sync.WaitGroup`.

Konceptualno:

```text
             shutdown signal
                    │
                    ▼
                 context
              ┌─────┼─────┐
              ▼     ▼     ▼
           worker  worker  worker
              │     │     │
              └─────┼─────┘
                    ▼
               WaitGroup
                    │
                    ▼
                 shutdown
```

Primer:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

var wg sync.WaitGroup

for i := 0; i < workers; i++ {
    wg.Add(1)

    go func() {
        defer wg.Done()
        worker(ctx)
    }()
}

cancel()
wg.Wait()
```

`context` signalizira:

```text
"prestanite sa radom"
```

dok `WaitGroup` omogućava:

```text
"sačekaj da svi zaista završe"
```

To su dve različite odgovornosti.

---

### Pitanje 53

**Zašto je `WaitGroup` sam po sebi nedovoljan za graceful shutdown?**

### Odgovor

`WaitGroup` samo omogućava čekanje da goroutine-i završe.

On ne govori goroutine-u:

```text
"treba da završiš."
```

Ako worker radi:

```go
func worker(wg *sync.WaitGroup) {
    defer wg.Done()

    for {
        doWork()
    }
}
```

onda:

```go
wg.Wait()
```

može čekati beskonačno.

Potrebna je kombinacija:

```text
signal termination
        +
wait for termination
```

Tipično:

```text
context / channel
        +
WaitGroup
```

---

### Pitanje 54

**Kako bi rešio graceful shutdown kada worker trenutno radi dugo-running operaciju?**

### Odgovor

Idealno, sama operacija treba da podržava cancellation.

Umesto:

```go
process(job)
```

bolje:

```go
process(ctx, job)
```

pa unutar operacije:

```go
select {
case <-ctx.Done():
    return ctx.Err()

case result := <-externalOperation:
    return handle(result)
}
```

Ako je operacija database ili network call koji podržava context, context treba proslediti direktno.

Ako je operation neprekidiva:

```text
shutdown
   ↓
worker waits
   ↓
operation completes
   ↓
worker exits
```

potrebno je imati maksimalni shutdown deadline.

Ne treba dozvoliti da graceful shutdown postane:

```text
shutdown
   ↓
wait forever
```

---

### Pitanje 55

**Koja je razlika između cancellation propagation i cancellation enforcement?**

### Odgovor

Cancellation propagation znači da signal o otkazivanju putuje kroz sistem:

```text
parent context
      ↓
service
      ↓
repository
      ↓
external operation
```

Cancellation enforcement znači da komponenta zaista reaguje na taj signal.

Na primer:

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            doWork()
        }
    }
}
```

Ako `doWork()` traje 60 sekundi i ne proverava context, cancellation može biti propagiran, ali nije efektivno primenjen tokom te operacije.

Zato:

```text
propagation
    ≠
enforcement
```

Sistem mora imati oba.

---

### Pitanje 56

**Kako bi sprečio da shutdown deadlock-uje zbog pogrešnog ownership-a channel-a?**

### Odgovor

Pretpostavimo:

```text
Producer → jobs → Worker
```

Ako worker očekuje:

```go
for job := range jobs {
    process(job)
}
```

onda mora postojati jasan owner koji će zatvoriti `jobs`.

Ako producer nikada ne zatvori channel:

```text
worker
   ↓
range jobs
   ↓
wait forever
```

Ako više producer-a šalje u isti channel, prerano zatvaranje je opasno:

```text
Producer A ─┐
Producer B ─┼──> jobs
Producer C ─┘
```

Jedan producer ne sme zatvoriti channel dok ostali još mogu slati.

Mogući model je centralizovani producer coordinator:

```text
Producer A ─┐
Producer B ─┼──> Coordinator ──> jobs
Producer C ─┘
                           │
                           ▼
                         close
```

Coordinator zna kada su svi producer-i završili i tada zatvara output channel.

---

### Pitanje 57

**Kako bi dizajnirao shutdown za pipeline sa više faza?**

### Odgovor

Pretpostavimo pipeline:

```text
Input
  ↓
Parse
  ↓
Validate
  ↓
Transform
  ↓
Persist
```

Svaka faza može imati svoje goroutine-e.

Potrebno je obezbediti da cancellation propagira kroz ceo pipeline:

```text
                context
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
     Parse      Validate    Transform
                               │
                               ▼
                            Persist
```

Svaka faza treba:

1. da prestane da prihvata novi posao kada je cancelled,
2. da zatvori svoje output channel-e kada završi,
3. da sačeka svoje worker-e,
4. da ne ostavi downstream consumer-e blokirane,
5. da ne zatvara channel koji nije njen ownership.

Kod pipeline-a je posebno važno sprečiti situaciju:

```text
Stage A stopped
      ↓
Stage B waits forever
      ↓
Stage C waits forever
```

Zato shutdown mora biti deo arhitekture pipeline-a, a ne naknadno dodat feature.

---

### Pitanje 58

**Kako bi razlikovao graceful shutdown od graceful degradation?**

### Odgovor

Graceful shutdown znači:

> Sistem se namerno gasi.

Graceful degradation znači:

> Sistem ostaje aktivan, ali pod opterećenjem smanjuje funkcionalnost ili throughput da bi očuvao stabilnost.

Na primer:

```text
Normal:
payments + reporting + analytics

Degraded:
payments
       +
limited reporting
       +
analytics disabled
```

Dok:

```text
Shutdown:
stop accepting work
       ↓
drain
       ↓
terminate
```

Oba koncepta koriste:

* cancellation,
* bounded resources,
* prioritization,
* timeouts,

ali imaju različite ciljeve.

---

## Senior+ perspektiva

Na ovom nivou cancellation se više ne posmatra kao:

```go
ctx.Done()
```

jedna izolovana Go tehnika.

Posmatra se kao deo **sistemskog lifecycle management-a**:

```text
Request lifecycle
       │
       ▼
Context propagation
       │
       ▼
Concurrent work
       │
       ▼
Backpressure
       │
       ▼
Cancellation
       │
       ▼
Drain
       │
       ▼
Resource cleanup
       │
       ▼
Termination
```

Dobro dizajniran concurrent sistem mora imati odgovor na pitanje:

> **"Šta se dešava sa svakim goroutine-om, channel-om i external resource-om kada posao više nije potreban?"**

Ako na to pitanje ne postoji precizan odgovor, concurrency dizajn nije kompletan.

---

Jedan od važnih koraka od Senior ka Senior+ nivou jeste prelazak sa razmišljanja:

> „Pokrenuću goroutine i on će završiti posao.“

na razmišljanje:

> „Ko je vlasnik ovog concurrent task-a, ko upravlja njegovim životnim ciklusom, kako se propagira greška i kako garantujemo da je task završen ili otkazan?“

Ovo je suština **structured concurrency** pristupa.

Go nema jedan jedinstveni `structured concurrency` primitiv, ali kombinacijom:

* `context.Context`,
* `sync.WaitGroup`,
* `errgroup`,
* channel-a,
* cancellation-a,
* jasnog ownership-a,

možemo izgraditi vrlo robustan model.

---

## Pitanje 59

**Šta podrazumevamo pod structured concurrency?**

### Odgovor

Structured concurrency je način organizovanja concurrent task-ova tako da njihov životni ciklus bude jasno vezan za životni ciklus parent operacije.

Umesto:

```text
request
   │
   ├── goroutine ──────────────── ?
   ├── goroutine ─────── ?
   └── goroutine ─────────────── ?
```

želimo:

```text
request
   │
   ├── task A
   ├── task B
   └── task C
        │
        ▼
   all tasks complete
        │
        ▼
request completes
```

Parent zna:

* koje child task-ove je pokrenuo,
* kada su završili,
* da li je neki task otkazao,
* da li failure jednog task-a treba da otkaže ostale,
* kada je bezbedno osloboditi resurse.

Fundamentalni princip je:

```text
child task lifetime
        ⊆
parent task lifetime
```

Drugim rečima, child goroutine ne bi trebalo proizvoljno da nadživi operaciju koja ga je pokrenula.

---

## Pitanje 60

**Koji problem rešava "fire-and-forget" goroutine?**

### Odgovor

Naizgled jednostavan kod:

```go
go sendMetrics(event)
```

može biti potpuno opravdan u određenim situacijama.

Ali postavlja ozbiljna pitanja:

```text
Ko čeka ovaj goroutine?
Ko hvata njegovu grešku?
Šta ako aplikacija počne shutdown?
Šta ako request više ne postoji?
Šta ako se resource zatvori?
Šta ako goroutine nikada ne završi?
```

Ako je odgovor na sva ova pitanja:

```text
"Ne znam."
```

onda imamo ne-strukturisanu konkurentnost.

Problem nije samo tehnički.

Na primer:

```text
HTTP request
   │
   ├── database update
   │
   └── go publishEvent()
             │
             └── request returns
```

Ako process odmah nakon toga dobije shutdown signal:

```text
publishEvent()
     X
process terminates
```

event može biti izgubljen.

Senior+ kandidat zato mora razlikovati:

### Background work koji je namerno nezavisan

od:

### Child task-a koji predstavlja deo trenutne operacije.

Samo drugi slučaj zahteva strogu parent-child strukturu.

---

## Pitanje 61

**Da li svaki goroutine mora imati parent goroutine?**

### Odgovor

Ne u bukvalnom tehničkom smislu.

Production servis može imati dugovečne goroutine-e:

```text
main
 ├── HTTP server
 ├── metrics collector
 ├── queue consumer
 └── scheduler
```

Ti goroutine-i predstavljaju **application-level components**.

Ali i oni treba da imaju jasno definisan lifecycle.

Na primer:

```text
application
     │
     ├── HTTP server
     ├── worker pool
     └── metrics collector
```

Application lifecycle može biti njihov logički parent:

```text
application start
      ↓
components start
      ↓
running
      ↓
shutdown signal
      ↓
components stop
      ↓
application exit
```

Dakle, structured concurrency ne znači nužno:

> „Svaki goroutine mora direktno biti child drugog goroutine-a.“

Znači da concurrent aktivnosti imaju **jasno definisan ownership i lifecycle**.

---

## Pitanje 62

**Kako `errgroup` pomaže u implementaciji structured concurrency-ja?**

### Odgovor

`errgroup` omogućava da grupa goroutine-a bude posmatrana kao jedna logička celina.

Konceptualno:

```text
Parent operation
       │
       ├── Task A
       ├── Task B
       └── Task C
```

Ako jedan task vrati error:

```text
Task A → nil
Task B → error
Task C → cancellation
```

grupa može:

```text
error
  ↓
cancel context
  ↓
stop remaining tasks
  ↓
wait
  ↓
return error
```

To je upravo model koji želimo kod mnogih request-scoped concurrent operacija.

Na primer:

```go
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error {
    return loadUser(ctx)
})

g.Go(func() error {
    return loadOrders(ctx)
})

g.Go(func() error {
    return loadRecommendations(ctx)
})

if err := g.Wait(); err != nil {
    return err
}
```

Ovde parent operacija ne završava dok child task-ovi ne završe ili budu otkazani.

---

## Pitanje 63

**Koja je prednost `errgroup.WithContext` u odnosu na običan `sync.WaitGroup`?**

### Odgovor

`WaitGroup` prvenstveno rešava:

```text
"Sačekaj da svi goroutine-i završe."
```

Ne rešava sam po sebi:

```text
"Šta ako jedan goroutine doživi error?"
```

Sa `errgroup.WithContext` dobijamo:

```text
task failure
    ↓
context cancellation
    ↓
remaining tasks stop
    ↓
Wait
    ↓
return error
```

To omogućava koordinaciju failure-a.

Na primer:

```text
Task A ─────── success
Task B ── error
Task C ──────────────── cancelled
```

Umesto da `Task C` nastavi da troši resurse iako je rezultat parent operacije već neupotrebljiv.

---

## Pitanje 64

**Da li `errgroup` automatski prekida svaki goroutine kada jedan task vrati grešku?**

### Odgovor

Ne.

`errgroup` može otkazati context koji je kreiran preko:

```go
errgroup.WithContext(...)
```

ali svaki goroutine mora **kooperativno reagovati na cancellation**.

Na primer:

```go
g.Go(func() error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()

        case item := <-items:
            process(item)
        }
    }
})
```

Ako goroutine ignoriše context:

```go
g.Go(func() error {
    for {
        doUninterruptibleWork()
    }
})
```

cancellation neće magično prekinuti njegovu izvršnu funkciju.

Dakle:

```text
errgroup
   ↓
cancellation signal
   ↓
goroutine must cooperate
```

Go nema mehanizam kojim `context.CancelFunc` nasilno ubija proizvoljan goroutine.

---

## Pitanje 65

**Zašto je goroutine cancellation kooperativna?**

### Odgovor

Go namerno ne pruža opšti mehanizam za bezbedno:

```text
kill goroutine
```

Jer goroutine može u trenutku prekida:

* držati mutex,
* menjati shared state,
* imati otvoren resource,
* izvršavati critical section,
* biti usred state transition-a.

Nasilno zaustavljanje može ostaviti sistem u nekonzistentnom stanju.

Zato se koristi kooperativni model:

```text
cancel signal
      ↓
goroutine observes cancellation
      ↓
cleanup
      ↓
return
```

Na primer:

```go
select {
case <-ctx.Done():
    return ctx.Err()

case result := <-operation:
    return handle(result)
}
```

Ovaj model zahteva da API-ji i funkcije budu projektovani sa cancellation-om na umu.

---

## Pitanje 66

**Kako bi dizajnirao funkciju koja je cancellation-aware?**

### Odgovor

Prvo, funkcija treba da prihvati context ako njen posao može trajati duže vreme ili zavisi od cancellable operacije:

```go
func Process(ctx context.Context, job Job) error
```

Zatim treba proveravati cancellation na odgovarajućim granicama.

Na primer:

```go
func Process(ctx context.Context, jobs []Job) error {
    for _, job := range jobs {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }

        if err := processOne(ctx, job); err != nil {
            return err
        }
    }

    return nil
}
```

Još bolje je da i `processOne` propagira context:

```go
func processOne(ctx context.Context, job Job) error {
    // network/database/etc.
}
```

Ako postoji veliki CPU-bound loop, potrebno je povremeno proveravati context:

```text
iteration
iteration
iteration
   ↓
check ctx
   ↓
iteration
iteration
```

Ne treba proveravati context na svakom mogućem mestu bez razloga, ali cancellation granice moraju biti dovoljno česte da sistem može reagovati u prihvatljivom vremenu.

---

## Pitanje 67

**Šta je cancellation latency?**

### Odgovor

Cancellation latency je vreme između:

```text
cancel()
```

i trenutka kada concurrent task zaista prestane sa radom.

Na primer:

```text
T0: cancel()
T1: worker observes cancellation
T2: worker cleanup
T3: worker returns
```

Ako je:

```text
T3 - T0 = 5 seconds
```

cancellation latency je približno 5 sekundi.

Ovo može biti problem ako je shutdown deadline:

```text
2 seconds
```

a worker reaguje tek nakon:

```text
5 seconds
```

Cancellation latency zavisi od:

* blocking operacija,
* polling intervala,
* network timeout-a,
* database timeout-a,
* CPU-bound rada,
* channel communication-a,
* cleanup-a.

Senior+ kandidat treba da razmišlja o cancellation-u ne samo kao boolean signal-u, već i kao **latency contract-u**.

---

## Pitanje 68

**Kako bi dizajnirao bounded concurrent task group?**

### Odgovor

Ako imamo veliki broj task-ova:

```text
100.000 jobs
```

ne želimo nužno:

```text
100.000 goroutines
```

Možemo imati:

```text
100.000 jobs
      ↓
bounded concurrency = 20
      ↓
20 active tasks
```

Jedan model je worker pool:

```text
                 ┌── Worker 1
                 ├── Worker 2
Jobs ────────────┼── ...
                 └── Worker 20
```

Drugi model je semaphore:

```text
acquire
   ↓
run task
   ↓
release
```

Konceptualno:

```go
sem := make(chan struct{}, 20)

for _, job := range jobs {
    sem <- struct{}{}

    go func(job Job) {
        defer func() {
            <-sem
        }()

        process(job)
    }(job)
}
```

Ali ovde i dalje možemo kreirati veoma veliki broj goroutine-a ako producer brzo prolazi kroz `jobs`.

Bolji dizajn može uključivati i bounded input ili worker pool:

```text
bounded queue
+
bounded workers
```

Dakle, **concurrency limit** i **goroutine count** nisu nužno ista stvar.

---

## Pitanje 69

**Koja je razlika između concurrency limit-a i queue limit-a?**

### Odgovor

Concurrency limit određuje koliko task-ova može biti **aktivno u istom trenutku**.

Na primer:

```text
concurrency = 20
```

znači:

```text
max 20 active tasks
```

Queue limit određuje koliko task-ova može čekati:

```text
queue = 100
```

Arhitektura:

```text
Incoming
   │
   ▼
Queue [100]
   │
   ▼
Workers [20]
```

Ukupan broj accepted-but-not-completed poslova može biti približno:

```text
100 queued
+
20 active
```

Ako se queue napuni, admission control odlučuje šta dalje.

Zato ova dva limita rešavaju različite probleme:

```text
Concurrency limit
    ↓
CPU / I/O / downstream pressure

Queue limit
    ↓
memory / latency / backlog pressure
```

---

## Pitanje 70

**Kako bi rešio failure jednog task-a kada ostali task-ovi zavise od njega?**

### Odgovor

Prvo treba identifikovati dependency graph.

Na primer:

```text
        A
       / \
      B   C
       \ /
        D
```

Ako `A` ne uspe:

```text
A → failure
```

nema smisla pokretati:

```text
B
C
D
```

Ako `B` failuje, možda `C` i dalje može raditi, ali `D` ne može.

To znači da structured concurrency nije samo:

```text
start tasks
wait tasks
```

već zahteva modelovanje dependency-ja.

Mogući pristupi su:

```text
fail-fast
```

gde failure prekida celu grupu,

ili:

```text
partial failure
```

gde se nezavisne operacije nastavljaju.

Izbor zavisi od semantike sistema.

---

## Pitanje 71

**Šta je fail-fast concurrency model?**

### Odgovor

Fail-fast model znači:

> Kada jedna kritična concurrent operacija ne uspe, ostatak operacije se otkazuje što je pre moguće.

Na primer:

```text
Load user       ─── success
Load account    ─── error
Load analytics  ─── cancellation
```

Ako bez account-a ceo request nema smisla, nema razloga nastaviti analytics task.

Model:

```text
Task A ── success
Task B ── ERROR
             │
             ▼
         cancellation
          /      \
         ▼        ▼
      Task C    Task D
       stop      stop
```

Prednosti:

* manja potrošnja resursa,
* manji latency,
* jednostavnija failure semantika.

Ali nije dobar izbor kada task-ovi mogu nezavisno da daju korisne rezultate.

---

## Pitanje 72

**Kada fail-fast nije dobar izbor?**

### Odgovor

Ako imamo nezavisne operacije:

```text
fetch profile
fetch recommendations
fetch ads
```

failure jedne možda ne treba da prekine ostale.

Na primer:

```text
profile       → success
recommendations → error
ads           → success
```

Možemo vratiti:

```text
profile + ads
```

uz degradirane recommendations.

To je model:

```text
partial success
```

Tada je bolje imati granularniju cancellation i error handling strategiju.

Senior+ developer mora prvo definisati **business semantics**, pa tek onda izabrati concurrency primitive.

---

## Pitanje 73

**Kako bi testirao structured concurrency?**

### Odgovor

Testovi treba da proveravaju lifecycle, a ne samo rezultat.

Na primer:

```text
1. taskovi se pokreću
2. parent dobija cancellation
3. child taskovi reaguju
4. resources se oslobađaju
5. svi child taskovi završavaju
6. parent se završava
```

Posebno treba testirati:

```text
normal completion
cancellation
timeout
child failure
multiple child failures
shutdown
blocked worker
queue full
downstream failure
```

Važan kriterijum je:

```text
goroutine count after test
```

ne bi trebalo da raste kroz ponovljena izvršavanja zbog leaked goroutine-a.

---

## Senior+ princip

Structured concurrency uvodi veoma važan mentalni model:

```text
                 Parent
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
      Task A     Task B     Task C
        │          │          │
        └──────────┼──────────┘
                   ▼
             All completed
                   │
                   ▼
                Parent
```

Uz failure:

```text
Task B
  │
  ▼
error
  │
  ▼
cancel context
  │
  ├── Task A → stop
  └── Task C → stop
  │
  ▼
wait
  │
  ▼
return error
```

Suština je da concurrent rad ne bude skup nepovezanih goroutine-a, već **strukturisana hijerarhija task-ova sa jasno definisanim ownership-om, lifecycle-om, cancellation-om i failure semantikom**.

---

Na Senior+ nivou potrebno je razumeti da concurrency nije samo pitanje broja goroutine-a. Kada veliki broj goroutine-a konkuriše za CPU, lock, channel, I/O ili downstream resurs, sistem mora da ima određenu **scheduling** i **fairness** semantiku.

Posebno je važno razumeti razliku između:

* concurrency,
* parallelism,
* scheduling,
* fairness,
* starvation,
* priority,
* contention,
* throughput,
* latency.

---

## Pitanje 74

**Šta scheduler zapravo radi u Go runtime-u?**

### Odgovor

Go runtime scheduler odlučuje **koji goroutine će biti izvršavan na kom izvršnom resursu i kada**.

Go koristi model:

```text
G = Goroutine
M = OS Thread
P = Processor
```

odnosno poznati:

```text
G-M-P model
```

Pojednostavljeno:

```text
              Go Scheduler
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
       G1         G2         G3
        │          │          │
        └──────┬───┴──────┬───┘
               ▼          ▼
               P          P
               │          │
               ▼          ▼
               M          M
```

`P` predstavlja runtime resource potreban da bi Go kod mogao da se izvršava.

`M` je OS thread.

`G` je goroutine.

Scheduler pokušava da održi dobru iskorišćenost procesora, omogući napredak goroutine-a i izbegne nepotrebno blokiranje.

Međutim, scheduler nije application-level scheduler.

On ne zna da li je:

```text
job A
```

važniji od:

```text
job B
```

u poslovnom smislu.

Ako aplikacija zahteva prioritizaciju, fairness ili quotas, to se najčešće mora implementirati na višem nivou.

---

## Pitanje 75

**Da li Go garantuje da će goroutine-i biti izvršavani fer?**

### Odgovor

Ne treba pretpostavljati apsolutnu fairness garanciju.

Scheduler nastoji da omogući napredak goroutine-a i da spreči određene oblike starvation-a, ali aplikacija ne treba da zasniva correctness na pretpostavci:

> „Svaki goroutine će dobiti tačno jednak CPU time.“

Na primer:

```text
G1 ────────────────
G2 ─────
G3 ─────────
```

ne znači da će G1, G2 i G3 dobiti identičan broj CPU ciklusa.

Na runtime ponašanje utiču:

* broj procesora,
* `GOMAXPROCS`,
* blocking operacije,
* syscall-ovi,
* network poller,
* scheduler state,
* mutex contention,
* channel operations,
* workload.

Zato treba razlikovati:

```text
scheduler fairness
```

od:

```text
application fairness
```

Ako zahtev glasi:

> „Svaki tenant mora dobiti približno jednak deo kapaciteta.“

to nije problem koji treba prepustiti Go scheduler-u.

---

## Pitanje 76

**Šta je starvation?**

### Odgovor

Starvation nastaje kada goroutine ili task praktično ne dobija mogućnost da napreduje jer drugi učesnici konstantno zauzimaju potreban resurs.

Na primer:

```text
G1 ──┐
G2 ──┤
G3 ──┤──> shared resource
G4 ──┘
```

Ako `G1` konstantno uspeva da dobije resource:

```text
G1 → resource
G1 → resource
G1 → resource
G1 → resource
...
```

dok `G4` nikada ne dobija priliku:

```text
G4 → waiting...
```

onda `G4` može biti starved.

Starvation može nastati zbog:

* lock contention-a,
* lošeg scheduling-a,
* agresivnog retry-ja,
* neograničenog priority workload-a,
* pogrešnog worker-pool dizajna,
* nefer queue strukture.

Važno je da starvation nije isto što i deadlock.

Kod deadlock-a:

```text
niko ne napreduje
```

Kod starvation-a:

```text
neki napreduju
drugi ne
```

---

## Pitanje 77

**Koja je razlika između starvation-a i deadlock-a?**

### Odgovor

### Deadlock

Sistem ulazi u stanje gde učesnici međusobno čekaju i niko ne može nastaviti.

Na primer:

```text
G1 → čeka Lock B
G2 → čeka Lock A
```

dok:

```text
G1 drži Lock A
G2 drži Lock B
```

Dobijamo:

```text
G1 ──wait──> B
 ↑           │
 │           ↓
 A <──wait── G2
```

Niko ne napreduje.

### Starvation

Neko napreduje, ali određeni task praktično nikada ne dobija priliku.

```text
G1 → progress
G2 → progress
G3 → progress
G4 → waiting forever
```

Dakle:

```text
deadlock
    ↓
globalni nedostatak napretka

starvation
    ↓
nedostatak napretka određenog učesnika
```

---

## Pitanje 78

**Kako može nastati starvation u worker pool-u?**

### Odgovor

Pretpostavimo priority queue:

```text
High priority
   ↓
workers
```

Ako stalno stižu high-priority poslovi:

```text
H H H H H H H H H ...
```

low-priority posao:

```text
L
```

može ostati u queue-u veoma dugo:

```text
H → H → H → H → H → ...
L → waiting
```

Ako sistem nikada ne uzme `L`, imamo starvation.

Jedno rešenje je **aging**:

```text
waiting time ↑
    ↓
effective priority ↑
```

Tako low-priority posao vremenom postaje dovoljno prioritetan da bude obrađen.

Drugi pristup je weighted scheduling:

```text
High: 80%
Low:  20%
```

što omogućava low-priority workload-u minimalni garantovani kapacitet.

---

## Pitanje 79

**Kako implementirati fairness između različitih workload-a?**

### Odgovor

Jedan od pristupa je odvojeni queue po tenant-u:

```text
Tenant A → Queue A ─┐
Tenant B → Queue B ─┼── Scheduler → Workers
Tenant C → Queue C ─┘
```

Scheduler može koristiti:

* round-robin,
* weighted round-robin,
* fair queueing,
* priority + aging.

Na primer:

```text
A → B → C → A → B → C
```

umesto:

```text
A → A → A → A → A → A
```

ako A konstantno generiše workload.

Fairness je posebno važan u multi-tenant sistemima.

Bez njega jedan korisnik može postati:

```text
noisy neighbor
```

i degradirati performanse ostalih.

---

## Pitanje 80

**Šta je priority inversion?**

### Odgovor

Priority inversion nastaje kada high-priority task indirektno čeka low-priority task koji drži potreban resource.

Primer:

```text
High priority H
       │
       ▼
    waits
       │
       ▼
Low priority L
       │
       ▼
     holds
     Mutex
```

Ali između njih se nalazi medium-priority workload:

```text
H → waits for L
M → consumes CPU
L → cannot run
```

Dobijamo:

```text
High priority
     ↓
 waiting
     ↓
Low priority
     ↑
blocked by
     │
Medium priority
```

Ovo može ozbiljno narušiti latency očekivanja.

U klasičnim real-time sistemima priority inversion je posebno ozbiljan problem.

U Go aplikacijama ne treba automatski pretpostaviti da možemo dobiti real-time scheduling semantiku samo zato što smo definisali neki application-level priority.

---

## Pitanje 81

**Kako bi dizajnirao priority-aware worker pool?**

### Odgovor

Jednostavan worker pool:

```text
jobs channel
   ↓
workers
```

ne daje direktnu podršku za prioritete.

Za priority scheduling možemo imati:

```text
High Queue
Medium Queue
Low Queue
```

i scheduler:

```text
           ┌── High
           │
Scheduler ├── Medium
           │
           └── Low
```

Naivni scheduler:

```text
always choose High
```

može izazvati starvation.

Bolji pristup može kombinovati:

```text
priority
+
fairness
+
aging
```

Na primer:

```text
effective priority =
base priority + waiting time
```

Tako se balansira između:

```text
latency za kritične poslove
```

i:

```text
garantovanog napretka ostalih poslova
```

---

## Pitanje 82

**Šta je work conservation?**

### Odgovor

Work-conserving scheduler pokušava da ne ostavi worker idle ako postoji posao koji može biti izvršen.

Na primer:

```text
Workers: 4

W1 → busy
W2 → busy
W3 → idle
W4 → busy

Queue:
job available
```

Ako `W3` može da preuzme posao, work-conserving sistem pokušava da ga aktivira.

Nasuprot tome, sistem sa nepotrebnim restrikcijama može imati:

```text
CPU idle
+
pending work
```

što smanjuje throughput.

Ali work conservation ne znači:

> „Uvek pokreni maksimalan broj task-ova.“

Ako je downstream resource već saturiran:

```text
DB max connections = 20
```

pokretanje:

```text
1.000 DB operations
```

ne povećava nužno throughput.

Može ga čak smanjiti zbog contention-a.

---

## Pitanje 83

**Zašto više concurrency-ja ne znači nužno veći throughput?**

### Odgovor

Postoji granica nakon koje dodatna konkurentnost povećava overhead umesto korisnog rada.

Na primer:

```text
Concurrency
   │
   │          ______
   │        /
   │      /
   │    /
   │  /
   └──────────────────
             Throughput
```

U početku:

```text
concurrency ↑
throughput ↑
```

ali nakon saturacije:

```text
concurrency ↑
throughput ≈ constant
```

ili čak:

```text
concurrency ↑
throughput ↓
```

zbog:

* lock contention-a,
* scheduler overhead-a,
* cache contention-a,
* context switching-a,
* memory bandwidth-a,
* downstream saturation-a,
* connection pool contention-a.

Zato se concurrency tuning mora zasnivati na merenjima.

---

## Pitanje 84

**Šta je contention i kako ga prepoznaješ?**

### Odgovor

Contention nastaje kada više concurrent task-ova konkuriše za isti resurs.

Tipični resursi su:

```text
Mutex
RWMutex
channel
database connection
CPU
memory bandwidth
file
network connection
```

Primer:

```text
G1 ──┐
G2 ──┤
G3 ──┼──> Mutex
G4 ──┤
G5 ──┘
```

Ako većina goroutine-a provodi vreme čekajući mutex, povećanje broja worker-a neće pomoći.

Možemo dobiti:

```text
workers ↑
contention ↑
throughput ↓
```

Contention treba analizirati pomoću profiling i tracing alata, a ne pretpostavkom.

---

## Pitanje 85

**Kako bi razlikovao CPU saturation od lock contention-a?**

### Odgovor

Kod CPU saturation-a CPU je zaista zauzet korisnim ili scheduler radom:

```text
CPU utilization ≈ high
```

Kod lock contention-a možemo imati:

```text
goroutines = high
CPU utilization = moderate
lock wait = high
```

Na primer:

```text
G1 → lock → work
G2 → wait
G3 → wait
G4 → wait
G5 → wait
```

Većina goroutine-a ne radi korisni posao.

Kod CPU-bound sistema povećanje concurrency-ja može biti korisno do određene tačke.

Kod lock-heavy sistema može napraviti situaciju:

```text
more workers
    ↓
more contention
    ↓
less throughput
```

Zato profiling treba da pokaže gde vreme zaista odlazi.

---

## Pitanje 86

**Zašto worker pool može povećati latency iako smanjuje broj goroutine-a?**

### Odgovor

Worker pool uvodi queue.

Ako imamo:

```text
100 requests
```

a samo:

```text
10 workers
```

90 zahteva čeka:

```text
queue
```

Dakle:

```text
request
  ↓
queue wait
  ↓
worker execution
```

Ukupan latency je:

```text
queueing latency
+
processing latency
```

Ako je worker pool premali:

```text
queue latency ↑
```

Ako je prevelik:

```text
contention ↑
resource pressure ↑
```

Zato worker pool predstavlja trade-off između:

```text
bounded resource usage
```

i:

```text
queueing latency
```

Optimalna veličina mora biti određena workload-om i merenjima.

---

## Pitanje 87

**Šta je Little's Law i kako može pomoći pri analizi concurrency sistema?**

### Odgovor

Little's Law je:

```text
L = λW
```

gde je:

```text
L = prosečan broj elemenata u sistemu
λ = prosečan arrival rate
W = prosečno vreme provedeno u sistemu
```

U praktičnom concurrency sistemu možemo ga interpretirati kao:

```text
concurrency ≈ throughput × latency
```

Na primer, ako sistem obrađuje:

```text
λ = 100 requests/s
```

a prosečan request traje:

```text
W = 0.2 s
```

onda je prosečan broj request-a u sistemu približno:

```text
L = 100 × 0.2
  = 20
```

Ovo je veoma korisno za reasoning o:

* worker pool veličini,
* queue depth-u,
* concurrency limitima,
* throughput-u,
* latency-ju.

Ali Little's Law ne određuje sam po sebi optimalan dizajn. To je alat za analizu sistema.

---

## Pitanje 88

**Kako bi odredio početnu veličinu worker pool-a?**

### Odgovor

Ne postoji univerzalna vrednost:

```text
workers = 10
```

koja je ispravna za svaki sistem.

Početna procena zavisi od:

### CPU-bound workload-a

Početi približno oko raspoloživog CPU parallelism-a, zatim benchmark-ovati.

### I/O-bound workload-a

Možemo koristiti veći concurrency jer task-ovi provode vreme čekajući I/O.

Ali downstream sistem postavlja granicu.

Na primer:

```text
application workers = 100
database connections = 20
```

Ako svih 100 worker-a istovremeno pokušavaju DB operaciju, database connection pool postaje bottleneck.

Zato treba posmatrati:

```text
CPU
memory
queue
workers
connections
downstream latency
error rate
```

i meriti sistem pod realističnim workload-om.

---

## Pitanje 89

**Kako bi prepoznao da je concurrency limit prenizak?**

### Odgovor

Tipični simptomi:

```text
queue depth ↑
worker utilization = high
downstream utilization = low
throughput = unexpectedly low
```

Na primer:

```text
Workers = 10
Queue = 10.000
DB utilization = 20%
CPU = 30%
```

Moguće je da je concurrency limit prenizak.

Ali ne treba automatski povećati worker count.

Možda postoji skriveni bottleneck:

```text
mutex
channel
connection pool
rate limiter
```

Zato treba identifikovati **bottleneck resource**.

---

## Pitanje 90

**Kako bi prepoznao da je concurrency limit previsok?**

### Odgovor

Mogući simptomi:

```text
CPU ↑
context switching ↑
lock contention ↑
memory ↑
downstream errors ↑
latency ↑
throughput ↓
```

Na primer:

```text
workers = 20
throughput = 1.000 req/s

workers = 100
throughput = 900 req/s
```

Ako latency istovremeno raste:

```text
20 workers → 50 ms
100 workers → 300 ms
```

jasno je da dodatna konkurentnost nije pomogla.

Mogući uzrok je:

```text
resource saturation
```

ili:

```text
contention
```

---

## Pitanje 91

**Kako concurrency control utiče na tail latency?**

### Odgovor

Prosečan latency može izgledati dobro dok tail latency postaje katastrofalan.

Na primer:

```text
p50 = 20 ms
p95 = 100 ms
p99 = 2 s
```

Velika concurrency može povećati queueing i contention.

To često posebno pogađa:

```text
p95
p99
p99.9
```

Ako svaki request čeka shared resource:

```text
Request
   ↓
Queue
   ↓
Lock
   ↓
DB
```

mala varijacija u jednom sloju može propagirati kroz ceo sistem.

Zato concurrency tuning ne treba optimizovati samo prema:

```text
average latency
```

već i prema:

```text
tail latency
```

---

## Pitanje 92

**Zašto je fairness ponekad važniji od maksimalnog throughput-a?**

### Odgovor

Pretpostavimo dva tenant-a:

```text
Tenant A → 10.000 requests/s
Tenant B → 100 requests/s
```

Ako scheduler uvek obrađuje A jer queue nikada nije prazan:

```text
A A A A A A A A ...
B B B → waiting
```

ukupan throughput može izgledati odlično.

Ali B može imati neprihvatljiv latency.

Fair scheduler može malo smanjiti maksimalan throughput:

```text
A → 8.000/s
B → 100/s
```

ali obezbediti mnogo bolju izolaciju.

U production sistemima često je važnije:

```text
predictability
+
fairness
+
isolation
```

nego apsolutni maksimum throughput-a.

---

## Pitanje 93

**Kako bi dizajnirao concurrency sistem koji garantuje minimalni kapacitet za kritične poslove?**

### Odgovor

Možemo koristiti odvojene resource pool-ove:

```text
                Workers
              /         \
             /           \
      Critical          Normal
        Pool              Pool
```

Na primer:

```text
Critical = 20 workers
Normal   = 80 workers
```

Tako normalni workload ne može zauzeti svih 100 worker-a.

Alternativno možemo koristiti reservation:

```text
total = 100
reserved critical = 20
general capacity = 80
```

Time critical workload ima garantovan minimum.

Ovaj princip se često koristi za:

* control-plane traffic,
* health checks,
* payment operations,
* security operations,
* administrative traffic.

Ideja je:

> **Critical work ne sme zavisiti od toga da li je best-effort workload iscrpeo sve resurse.**

---

## Senior+ zaključak

Scheduling i concurrency control treba posmatrati kao sistemski problem:

```text
                Incoming Work
                      │
                      ▼
               Admission Control
                      │
                      ▼
                Priority/Fairness
                      │
                      ▼
                 Queue Limit
                      │
                      ▼
              Concurrency Limit
                      │
                      ▼
                  Workers
                      │
                      ▼
                Downstream
```

Na svakom nivou postoje potencijalni failure modes:

```text
queue overflow
starvation
priority inversion
contention
resource saturation
tail latency
```

Najvažniji Senior+ mentalni model je:

> **Ne optimizuje se broj goroutine-a. Optimizuje se protok korisnog rada kroz ograničene resurse uz kontrolisan latency i garantovan napredak važnih workload-a.**

---

Na Senior+ nivou više nije dovoljno znati da `Mutex` štiti kritičnu sekciju ili da channel omogućava komunikaciju između goroutine-a.

Potrebno je razumeti **zašto** određeni concurrent program jeste ili nije korektan.

Ključni koncepti su:

* Go Memory Model,
* visibility,
* ordering,
* synchronization,
* happens-before,
* data race,
* atomicity,
* synchronization events,
* channel communication,
* mutex unlock/lock ordering,
* goroutine creation,
* goroutine termination,
* `sync/atomic`.

Osnovno pitanje više nije:

> „Da li ovo radi na mojoj mašini?“

već:

> „Da li Go Memory Model garantuje da ovo ponašanje može da se dogodi na svim dozvoljenim izvršenjima?“

---

## Pitanje 94

**Šta je Go Memory Model?**

### Odgovor

Go Memory Model definiše pravila prema kojima se određuju:

* koji rezultati concurrent programa su dozvoljeni,
* kada je write jednog goroutine-a vidljiv drugom,
* koje synchronization operacije uspostavljaju ordering,
* šta program sme da pretpostavlja o memorijskim pristupima.

Veoma važna razlika je između:

```text
program executes on my machine
```

i:

```text
program execution is guaranteed by the language/runtime model
```

Na primer:

```go
var x int

func writer() {
    x = 42
}

func reader() {
    fmt.Println(x)
}
```

Ako se `writer` i `reader` izvršavaju concurrent bez synchronization-a, ne smemo pretpostaviti da reader pouzdano vidi:

```text
42
```

Samo zato što je write izvršen „ranije“.

Potrebno je postojanje odgovarajuće **happens-before** relacije.

---

## Pitanje 95

**Šta znači da je jedan događaj happens-before drugog?**

### Odgovor

`happens-before` je relacija koja određuje ordering između događaja u concurrent programu.

Ako:

```text id="g0d3jr"
A happens-before B
```

onda program može da se osloni na određene garantovane efekte A pre B.

Happens-before relacija nastaje kombinovanjem:

```text id="7fh2jp"
sequenced-before / program order
+
synchronization
```

Na primer:

```go
x = 42
ready = true
```

samo po sebi ne daje drugom goroutine-u dovoljno informacija.

Ali:

```go
mu.Lock()
x = 42
mu.Unlock()
```

i:

```go
mu.Lock()
fmt.Println(x)
mu.Unlock()
```

uvode synchronization odnos između odgovarajućih lock/unlock operacija.

Conceptualno:

```text id="6l9q1f"
G1
 │
 │ x = 42
 │
 ▼
Unlock
 │
 │ happens-before
 ▼
Lock
 │
 ▼
G2
 │
 │ read x
 ▼
42
```

To je fundamentalni mehanizam kojim concurrent program dobija **ordering i visibility guarantees**.

---

## Pitanje 96

**Da li program order sam po sebi garantuje visibility drugom goroutine-u?**

### Odgovor

Ne.

Ako goroutine A uradi:

```go
x = 100
```

a goroutine B zatim pročita:

```go
fmt.Println(x)
```

bez synchronization-a, činjenica da je A izvršio write pre B-ovog read-a u realnom vremenu nije dovoljna garancija.

Problem je:

```text id="j25w4a"
G1:
x = 100

G2:
read x
```

Ne postoji odgovarajuća synchronization relacija.

Drugim rečima:

```text id="l0o8qg"
wall-clock order
```

nije isto što i:

```text id="d9m5qk"
memory-model ordering
```

Ovo je jedna od najvažnijih razlika koju Senior+ kandidat mora razumeti.

---

## Pitanje 97

**Šta je data race?**

### Odgovor

Data race nastaje kada dva ili više goroutine-a concurrent pristupaju istoj memorijskoj lokaciji, pri čemu:

* najmanje jedan pristup predstavlja write,
* pristupi nisu pravilno sinhronizovani.

Na primer:

```go
var counter int

go func() {
    counter++
}()

go func() {
    counter++
}()
```

`counter++` nije jedna nedeljiva operacija.

Konceptualno može biti:

```text id="w2dbjk"
read counter
add 1
write counter
```

Dva goroutine-a mogu izvršavati ove operacije interleaved.

Na primer:

```text id="l7xqj4"
G1: read 0
G2: read 0
G1: write 1
G2: write 1
```

Rezultat može biti:

```text id="4c6y0j"
1
```

umesto očekivanih:

```text id="g6f7hf"
2
```

Ali problem nije samo izgubljeni increment.

Data race znači da program krši potrebne synchronization uslove i ponašanje nije nešto na šta treba računati.

---

## Pitanje 98

**Da li je svaki data race samo problem sa atomicity-jem?**

### Odgovor

Ne.

Atomicity i data race su povezani, ali nisu isti koncept.

Na primer:

```go
counter++
```

može biti problem jer operacija nije atomarna.

Ali i:

```go
ready = true
```

može predstavljati data race ako jedan goroutine piše `ready`, a drugi ga čita bez odgovarajuće synchronization relacije.

Ovde možda nema složenog read-modify-write problema.

Postoji problem:

```text id="0ubj5d"
visibility + ordering
```

Dakle:

```text id="8gq8iz"
atomicity
≠
data-race freedom
```

Možemo imati potrebu za synchronization-om čak i kada je pojedinačni memory access tehnički jednostavan.

---

## Pitanje 99

**Da li je `int` read/write automatski thread-safe zato što je mali tip?**

### Odgovor

Ne treba mešati:

```text
machine-level atomicity
```

sa:

```text
Go-level synchronization guarantees
```

Čak i ako je određeni read/write fizički izveden jednom instrukcijom na konkretnoj arhitekturi, to ne znači da je concurrent program korektan bez synchronization-a.

Na primer:

```go
var flag bool

go func() {
    flag = true
}()

go func() {
    if flag {
        doWork()
    }
}()
```

Ne treba ovaj kod opravdavati argumentom:

> „`bool` je samo jedan byte.“

Potrebno je definisati synchronization.

Ako je `flag` signal između goroutine-a, možemo koristiti:

```text id="1q5p4h"
channel
Mutex
atomic
```

u zavisnosti od semantike problema.

---

## Pitanje 100

**Kako channel komunikacija utiče na happens-before relaciju?**

### Odgovor

Channel operacije mogu uspostaviti synchronization odnos između goroutine-a.

Na primer:

```go
ch := make(chan int)

go func() {
    x := 42
    ch <- x
}()

value := <-ch
```

Conceptualno:

```text id="2p1r0a"
G1
 │
 │ x = 42
 │
 ▼
send
 │
 │ synchronization
 ▼
receive
 │
 ▼
G2
 │
 │ use value
 ▼
42
```

Zbog toga channel nije samo mehanizam za prenos podataka.

On može biti i **synchronization boundary**.

Ovo je važna karakteristika Go concurrency modela:

> Komunikacija između goroutine-a može istovremeno prenositi podatke i uspostavljati ordering.

---

## Pitanje 101

**Da li slanje na buffered channel ima potpuno isti synchronization efekat kao slanje na unbuffered channel?**

### Odgovor

Ne treba ih tretirati kao identične operacije.

Kod unbuffered channel-a send i receive direktno učestvuju u handoff-u:

```text id="yjkn9c"
Sender
   │
   │ send
   ▼
Receiver
```

Kod buffered channel-a postoji međuspremnik:

```text id="m3z1cw"
Sender
   │
   ▼
[ buffer ]
   │
   ▼
Receiver
```

Send može završiti bez trenutnog receive-a dok god buffer ima kapacitet.

To menja blocking behavior i određene synchronization odnose.

Zato izbor:

```text id="pr4t2q"
buffered
vs
unbuffered
```

nije samo performance tuning.

On menja semantiku koordinacije.

---

## Pitanje 102

**Šta znači da je channel close synchronization događaj?**

### Odgovor

Zatvaranje channel-a ima synchronization semantiku u Go memory modelu.

Ako jedan goroutine uradi:

```go
x = 42
close(ch)
```

a drugi:

```go
<-ch
```

nakon što je channel zatvoren, postoji relevantan ordering između close događaja i receive-a koji vidi closed state.

Konceptualno:

```text id="90u4o0"
G1
 │
 │ x = 42
 │
 ▼
close(ch)
 │
 │ synchronization
 ▼
receive from closed ch
 │
 ▼
G2
 │
 │ read x
```

Zato close može biti korišćen kao signal završetka.

Međutim, treba paziti da se channel koristi u skladu sa svojim ownership modelom.

Najčešće je sender odgovoran za close:

```text id="rv4r2m"
producer
   │
   ├── send
   ├── send
   └── close
```

dok consumer može:

```text id="p8b8m4"
range ch
```

---

## Pitanje 103

**Kako goroutine creation utiče na memory ordering?**

### Odgovor

Kada se goroutine kreira:

```go
x = 42

go func() {
    fmt.Println(x)
}()
```

postoji ordering između događaja koji prethode `go` statement-u i početka izvršavanja nove goroutine u smislu Go memory modela.

Conceptualno:

```text id="70pvpu"
Parent
 │
 │ x = 42
 │
 ▼
go statement
 │
 ▼
Child goroutine
 │
 ▼
read x
```

To znači da je veoma različita situacija od:

```text id="x6f6px"
G1 writes x
G2 independently starts
```

Bez odgovarajuće synchronization relacije.

Goroutine creation predstavlja jednu od runtime/language-level tačaka koja utiče na ordering.

---

## Pitanje 104

**Da li završetak goroutine-a automatski sinhronizuje sve sa drugim goroutine-ima?**

### Odgovor

Ne treba pretpostaviti da:

```text id="5c5f3j"
goroutine finished
```

automatski znači:

```text id="b9z0kc"
every other goroutine sees all its writes
```

Ako parent želi da zna da je child završio, koristi se eksplicitna synchronization mehanizacija.

Najčešće:

```text id="qjy6p0"
WaitGroup
channel
errgroup
```

Na primer:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()
    doWork()
}()

wg.Wait()
```

`Wait()` predstavlja koordinacionu tačku.

Ne treba implementirati completion protokol preko pretpostavke:

```text id="8p0f1r"
"goroutine je verovatno završio do sada"
```

---

## Pitanje 105

**Zašto je "sleep and hope" anti-pattern u concurrent kodu?**

### Odgovor

Kod:

```go
go doWork()

time.Sleep(time.Second)

useResult()
```

ne definiše correctness condition.

On samo kaže:

```text id="8q5w8p"
"nadam se da je task završio za jednu sekundu."
```

Ako task traje:

```text id="6ah5q2"
100 ms
```

čekamo nepotrebno.

Ako traje:

```text id="nq3h5e"
2 seconds
```

program može pristupiti stanju prerano.

Pravilno je koristiti synchronization:

```go
done := make(chan struct{})

go func() {
    defer close(done)
    doWork()
}()

<-done
```

ili:

```go
wg.Wait()
```

ili:

```go
errgroup.Wait()
```

depending on semantics.

Concurrency correctness treba biti definisana događajima, ne vremenom spavanja.

---

## Pitanje 106

**Kada koristiti Mutex, a kada atomic operaciju?**

### Odgovor

`Mutex` je pogodan kada štitimo **invariant ili kompleksno stanje**.

Na primer:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}
```

Ako operacija uključuje više polja:

```text id="6ylq5p"
read A
read B
modify A
modify B
```

i ta dva polja moraju ostati konzistentna, mutex je često prirodniji izbor.

Atomic operacije su pogodne za jednostavne state transitions:

```text id="2a7j5q"
counter
flag
state
pointer
```

na primer:

```go
atomic.AddInt64(&counter, 1)
```

Ali atomic API ne treba koristiti samo zato što je:

> „brži od mutex-a.“

Atomic design može biti mnogo teži za reasoning.

Prvo treba definisati invariant i synchronization protocol.

Tek onda izabrati primitive.

---

## Pitanje 107

**Da li atomic operacije rešavaju sve probleme sa shared memory?**

### Odgovor

Ne.

Atomic može garantovati atomicity konkretne operacije, ali ne mora rešiti širi invariant.

Na primer:

```text id="5n6wsl"
balance
available
status
```

Ako sva tri polja moraju predstavljati konzistentno stanje, tri nezavisna atomic-a možda nisu dovoljna.

Možemo dobiti:

```text id="0t7c4g"
balance = new
available = old
status = old
```

što predstavlja nevalidnu kombinaciju.

Tada je možda potreban:

```text id="j1w7f4"
Mutex
```

ili potpuno drugačiji state representation.

Senior+ kandidat mora razmišljati o **state invariant-u**, a ne samo o pojedinačnim memory access-ima.

---

## Pitanje 108

**Šta znači "data-race-free" program?**

### Odgovor

Program je data-race-free kada concurrent pristupi shared memory-u poštuju potrebne synchronization odnose.

Tipičan pristup je:

```text id="f9p1sl"
shared state
     │
     ▼
synchronization
     │
     ├── Mutex
     ├── Atomic
     ├── Channel
     └── other synchronization
```

Data-race-free program je mnogo lakši za reasoning jer memory model daje jače garancije.

To ne znači automatski da je program logički korektan.

Možemo imati potpuno race-free program sa:

```text id="4n8u3v"
deadlock
starvation
livelock
wrong ordering
lost business events
```

Dakle:

```text id="g8ux0q"
race-free
≠
bug-free
```

Ali data-race freedom je fundamentalni preduslov za pouzdanu concurrent implementaciju.

---

## Pitanje 109

**Kako Race Detector pomaže u analizi concurrency problema?**

### Odgovor

Go race detector može detektovati određene data race situacije tokom izvršavanja.

Tipično se pokreće:

```text id="9w9qpc"
go test -race ./...
```

ili odgovarajućim `-race` režimom za druge Go komande.

Ako postoji problem, race detector može prijaviti concurrent access pattern.

Ali važno je razumeti ograničenje:

```text id="f3kq9n"
race detector
     ↓
observed executions
```

On ne dokazuje da race ne postoji samo zato što test nije prijavio race.

Ako test nikada ne izvrši problematičan interleaving:

```text id="f9r1hl"
race exists
     ↓
test misses execution
     ↓
no report
```

Zato je `-race` veoma moćan alat, ali nije formalna verifikacija kompletnog concurrent programa.

---

## Pitanje 110

**Kako bi analizirao concurrency bug koji se javlja samo jednom u milion izvršavanja?**

### Odgovor

Prvo bih izbegao pristup:

```text id="0u4r8h"
"stavimo Sleep i probaj ponovo."
```

Umesto toga:

### 1. Identifikovati shared state

```text id="i4n8f1"
Koji podaci se dele?
```

### 2. Identifikovati synchronization boundaries

```text id="i2z7z0"
Gde se garantuje ordering?
```

### 3. Pokrenuti race detector

```text id="4t3v3s"
go test -race
```

### 4. Povećati broj iteracija

```text id="6zq8j1"
for i := 0; i < N; i++ {
    runScenario()
}
```

### 5. Smanjiti nondeterminism gde je moguće

Test treba da kontroliše:

* start barrier,
* goroutine koordinaciju,
* channel handoff,
* cancellation.

### 6. Dodati observability

Na primer:

```text id="2d4b6e"
request ID
task ID
goroutine role
state transition
timing
```

### 7. Analizirati state transitions

Ne samo:

```text id="1o5p0j"
"šta je finalna vrednost?"
```

već:

```text id="1a8j5k"
ko je promenio stanje
kada
pod kojim synchronization uslovom
```

Senior+ debugging concurrency problema je često **forenzička analiza execution ordering-a**.

---

## Pitanje 111

**Zašto "radi na x86" nije dovoljan argument za concurrent kod?**

### Odgovor

Arhitektura procesora i Go Memory Model nisu ista stvar.

Možemo imati program koji:

```text id="42o9qs"
na mojoj mašini
```

izgleda stabilno:

```text id="w7q9sl"
Linux
x86-64
Go version X
```

ali to ne znači da je ponašanje garantovano na:

```text id="h22g8q"
ARM64
drugi compiler/runtime
drugi optimization
drugo scheduling ponašanje
```

Još važnije:

Ako program ima data race, problem postoji bez obzira na to što se u praksi:

```text id="7i6q1a"
"nikad nije desio."
```

Senior+ kandidat ne treba da pita:

> „Da li CPU trenutno radi ono što očekujemo?“

već:

> „Da li language/runtime model garantuje ovo ponašanje?“

---

## Pitanje 112

**Kako bi objasnio razliku između ordering-a i visibility-ja?**

### Odgovor

**Ordering** govori o tome kojim redosledom se događaji mogu posmatrati u okviru garantovanog synchronization modela.

**Visibility** govori o tome da li jedan concurrent execution context može pouzdano da posmatra rezultat drugog.

Na primer:

```text id="uv7z1m"
G1:
x = 42
ready = true
```

G2:

```text id="jqv5ro"
if ready {
    fmt.Println(x)
}
```

Intuitivno očekujemo:

```text id="6u1phf"
ready == true
⇒ x == 42
```

Ali bez odgovarajućeg synchronization protocol-a, ovaj zaključak nije validan.

Potrebno je uspostaviti:

```text id="o1t4wr"
write x
   ↓
synchronization
   ↓
read x
```

Zato synchronization nije samo način da sprečimo simultaneous writes.

On definiše **ordering i visibility contract** između concurrent task-ova.

---

## Senior+ mentalni model

Concurrent memory reasoning možemo predstaviti ovako:

```text id="m6s5sy"
          Shared State
               │
               ▼
       ┌─────────────────┐
       │ Synchronization │
       └─────────────────┘
          │    │    │
          ▼    ▼    ▼
       Mutex Atomic Channel
          │    │    │
          └────┼────┘
               ▼
         Happens-Before
               │
               ▼
        Visibility/Ordering
               │
               ▼
        Correct Execution
```

Bez jasnog synchronization protokola, reasoning se često svodi na pretpostavke o:

* scheduler-u,
* CPU-u,
* cache-u,
* compiler-u,
* timing-u.

To nije dovoljno za production-grade concurrent software.

Senior+ nivo podrazumeva sposobnost da se concurrency correctness obrazloži kroz **memory model, synchronization i happens-before**, a ne kroz empirijsko posmatranje nekoliko uspešnih izvršavanja.

---

Na Senior+ nivou potrebno je razumeti da uklanjanje klasičnog deadlock-a ne znači automatski da je concurrent sistem korektan ili efikasan.

Postoje najmanje četiri različita problema napretka:

```text
Deadlock
    ↓
niko ne napreduje jer se međusobno čeka

Starvation
    ↓
određeni task ne dobija resurs

Livelock
    ↓
taskovi se stalno izvršavaju, ali ne ostvaruju napredak

Contention
    ↓
taskovi napreduju, ali uz visoku cenu čekanja
```

Kod naprednog concurrent dizajna često se koriste:

* `sync/atomic`,
* CAS (`Compare-And-Swap`),
* lock-free strukture,
* wait-free algoritmi,
* retry loops,
* backoff strategije.

Ovi pristupi mogu dati odlične performanse, ali povećavaju kompleksnost dokazivanja korektnosti.

---

## Pitanje 113

**Šta je livelock?**

### Odgovor

Livelock je stanje u kome concurrent task-ovi **nisu blokirani**, stalno izvršavaju neku akciju, ali sistem ipak ne ostvaruje koristan napredak.

Kod deadlock-a:

```text
G1 → waiting
G2 → waiting
```

Kod livelock-a:

```text
G1 → work → retry → work → retry → ...
G2 → work → retry → work → retry → ...
```

CPU može biti aktivan:

```text
CPU usage ↑
```

ali korisni posao ne napreduje:

```text
progress ≈ 0
```

Tipičan primer je retry protokol koji uvek reaguje na isti način i zbog toga se dve konkurentne operacije međusobno stalno „izbegavaju“.

---

## Pitanje 114

**Kako može nastati livelock u concurrent algoritmu?**

### Odgovor

Zamislimo dve goroutine koje pokušavaju da zauzmu dva resursa:

```text
G1 → A
G2 → B
```

Obe pokušaju da uzmu drugi:

```text
G1 → B → fail
G2 → A → fail
```

Umesto da blokiraju, obe odmah oslobode prvi resource:

```text
G1 → release A
G2 → release B
```

i pokušaju ponovo:

```text
G1 → A
G2 → B
```

Ponovo:

```text
G1 → B → fail
G2 → A → fail
```

Dobijamo:

```text
retry
  ↓
collision
  ↓
retry
  ↓
collision
  ↓
...
```

Nema klasičnog deadlock-a jer niko ne čeka zauvek.

Ali nema ni korisnog napretka.

---

## Pitanje 115

**Kako se livelock razlikuje od busy waiting-a?**

### Odgovor

Busy waiting znači da task aktivno proverava uslov umesto da blokira.

Na primer:

```go
for !ready.Load() {
}
```

Ovo može biti busy wait.

Ako `ready` nikada ne postane `true`, goroutine troši CPU bez napretka.

Livelock je širi koncept.

Kod livelock-a taskovi mogu aktivno menjati stanje:

```text
A → B → A → B → A
```

ali nikada ne dođu do korisnog završnog stanja.

Dakle:

```text
busy waiting
    ↓
jedan oblik aktivnog čekanja

livelock
    ↓
sistem aktivno radi
ali ne ostvaruje progress
```

Busy waiting može biti deo livelock-a, ali nije svaki busy wait livelock.

---

## Pitanje 116

**Kako sprečiti livelock?**

### Odgovor

Najčešće tehnike su:

### 1. Randomized backoff

Umesto:

```text
retry
retry
retry
```

koristimo:

```text
retry
 ↓
random delay
 ↓
retry
```

Ako se dva task-a sudare, mala je verovatnoća da će se ponovo savršeno sinhronizovati.

### 2. Exponential backoff

Na primer:

```text
10µs
20µs
40µs
80µs
160µs
...
```

uz maksimalni limit.

### 3. Deterministički ordering

Ako više task-ova treba da zauzme više resursa, definišemo globalni redosled:

```text
Lock A → Lock B → Lock C
```

svi task-ovi poštuju isti redosled.

### 4. Bounded retries

Ne treba dozvoliti:

```text
retry forever
```

bez jasne strategije.

### 5. Fallback

Nakon određenog broja neuspeha:

```text
retry
  ↓
retry
  ↓
retry
  ↓
fallback / error
```

---

## Pitanje 117

**Šta znači da je algoritam lock-free?**

### Odgovor

Lock-free algoritam garantuje da će **sistem kao celina ostvariti napredak** u konačnom broju koraka, čak i ako pojedinačni thread/goroutine bude ometen.

To ne znači:

```text
svaki goroutine će sigurno napredovati
```

Već:

```text
neki goroutine će napredovati
```

Drugim rečima:

```text
lock-free
    ↓
system-wide progress
```

dok:

```text
wait-free
    ↓
per-operation bounded progress
```

Ova razlika je veoma važna.

---

## Pitanje 118

**Šta znači da je algoritam wait-free?**

### Odgovor

Wait-free algoritam daje jaču garanciju:

> Svaki poziv operacije završava u ograničenom broju koraka, nezavisno od ponašanja drugih učesnika.

Conceptualno:

```text
Lock-free:

G1 ────────────── success
G2 ─ retry ─ retry ─ retry ─ ...

System progresses.
```

Wait-free:

```text
G1 ───── success
G2 ───── success
G3 ───── success
```

uz garantovanu gornju granicu broja koraka za svaku operaciju.

Wait-free je teže implementirati i dokazati.

Zato:

```text
wait-free
```

nije automatski bolji izbor za production application code.

---

## Pitanje 119

**Šta je obstruction-free?**

### Odgovor

Obstruction-free je slabija progress garancija od lock-free.

Algoritam garantuje da će operacija završiti ako određeni thread/goroutine može da izvršava dovoljno dugo bez konkurencije sa drugim operacijama.

Hijerarhijski:

```text
Wait-free
    ↓
Lock-free
    ↓
Obstruction-free
```

Uobičajena interpretacija:

### Wait-free

Svaki task ima garantovan bounded progress.

### Lock-free

Sistem kao celina ima garantovan progress.

### Obstruction-free

Jedna izolovana operacija može završiti ako nema konkurencije.

Ovi termini opisuju **progress guarantees**, a ne samo odsustvo mutex-a.

---

## Pitanje 120

**Da li lock-free znači da nema nikakvog čekanja?**

### Odgovor

Ne.

Ovo je jedna od najčešćih zabluda.

Lock-free algoritam može imati:

```text
CAS failure
    ↓
retry
    ↓
CAS failure
    ↓
retry
```

Dakle goroutine može čekati indirektno kroz retry.

Razlika je u tome što nema klasičnog:

```text
Mutex.Lock()
    ↓
blocked
```

koji bi mogao dovesti do deadlock-a.

Lock-free algoritam može trošiti CPU na retry i može imati visok contention.

Na primer:

```text
G1 → CAS success
G2 → CAS fail
G3 → CAS fail
G2 → retry
G3 → retry
```

Ako jedan goroutine konstantno uspeva, drugi mogu imati vrlo lošu individualnu progress semantiku.

Zato:

```text
lock-free
≠
wait-free
```

i:

```text
lock-free
≠
zero waiting
```

---

## Pitanje 121

**Šta je CAS?**

### Odgovor

CAS znači:

**Compare-And-Swap**

Konceptualno:

```text
CAS(address, old, new)
```

znači:

```text
ako je *address == old
    postavi *address = new
    success
inače
    failure
```

Na primer:

```text
counter = 10
```

Goroutine pokušava:

```text
CAS(counter, 10, 11)
```

Ako je counter i dalje `10`:

```text
10 → 11
success
```

Ako je drugi goroutine već promenio vrednost:

```text
10 → 12
```

CAS:

```text
expected = 10
actual   = 12
```

ne uspeva.

Task zatim može pokušati ponovo:

```text
read
 ↓
calculate
 ↓
CAS
 ↓
failure?
 ↓
retry
```

---

## Pitanje 122

**Zašto je CAS osnova mnogih lock-free algoritama?**

### Odgovor

CAS omogućava atomic state transition:

```text
old state
    │
    │ CAS
    ▼
new state
```

bez klasičnog mutex-a.

Na primer, state machine može imati:

```text
OPEN
 ↓
CLOSING
 ↓
CLOSED
```

Task može pokušati:

```text
CAS(state, OPEN, CLOSING)
```

Samo jedan goroutine može uspešno izvršiti tranziciju.

Ostali dobijaju:

```text
CAS failure
```

i mogu:

```text
retry
```

ili:

```text
observe state
```

Ovo je veoma moćan obrazac za implementaciju:

* counters,
* flags,
* state machines,
* reference counts,
* lock-free queues,
* once-like state transitions.

---

## Pitanje 123

**Zašto CAS retry loop može biti skup?**

### Odgovor

Pretpostavimo:

```text
100 goroutines
```

koje pokušavaju da izmene jednu atomic promenljivu.

Možemo dobiti:

```text
G1 → CAS success
G2 → CAS fail
G3 → CAS fail
...
G100 → CAS fail
```

Zatim svi ponovo pokušavaju.

To znači:

```text
contention ↑
CAS failures ↑
CPU work ↑
```

i veliki deo CPU vremena može otići na neuspešne pokušaje.

Na kraju:

```text
lock-free algorithm
```

može imati lošije performanse od:

```text
simple mutex
```

ako je contention dovoljno visok.

Zato se lock-free dizajn ne uvodi bez benchmark-a.

---

## Pitanje 124

**Kako izgleda tipičan atomic CAS loop u Go-u?**

### Odgovor

Konceptualno:

```go
for {
    old := atomic.LoadInt64(&value)
    newValue := old + 1

    if atomic.CompareAndSwapInt64(&value, old, newValue) {
        break
    }
}
```

Semantika:

```text
read current value
       ↓
calculate new value
       ↓
CAS
   ┌───┴───┐
 success  failure
   │         │
   ▼         ▼
  done      retry
```

Važno je razumeti da između:

```text
Load
```

i:

```text
CAS
```

drugi goroutine može promeniti vrednost.

Zato CAS mora proveriti da li je state i dalje onaj koji smo očekivali.

---

## Pitanje 125

**Da li `atomic.AddInt64` može biti bolji izbor od ručno implementiranog CAS loop-a?**

### Odgovor

Da.

Ako problem predstavlja samo atomic increment:

```go
atomic.AddInt64(&counter, 1)
```

je jednostavniji i manje podložan greškama od:

```go
for {
    old := atomic.LoadInt64(&counter)

    if atomic.CompareAndSwapInt64(
        &counter,
        old,
        old+1,
    ) {
        break
    }
}
```

CAS je potreban kada imamo custom state transition.

Ako standardna atomic operacija već izražava potreban invariant, treba je preferirati.

Senior+ princip:

> Ne implementiraj složeniji synchronization protocol kada postojeći primitive direktno izražava problem.

---

## Pitanje 126

**Kada je mutex bolji od lock-free algoritma?**

### Odgovor

U velikom broju production situacija — često.

Mutex može biti bolji kada:

* state ima više međusobno zavisnih polja,
* critical section je kratka,
* contention nije ekstreman,
* correctness je važnija od mikro-optimizacije,
* kod treba lako održavati,
* latency zahtevi nisu ekstremni.

Na primer:

```go
type Cache struct {
    mu    sync.Mutex
    items map[string]Value
}
```

je često bolji dizajn od komplikovanog lock-free cache-a.

Lock-free implementacija može imati:

```text
više retry-ja
više edge case-ova
teže reasoning
teže debugging
teže testiranje
```

Ako benchmark ne pokazuje realan benefit:

```text
simple mutex
```

je često bolji izbor.

---

## Pitanje 127

**Zašto je lock-free kod teži za dokazivanje korektnosti?**

### Odgovor

Kod mutex pristupa imamo relativno jednostavan model:

```text
Lock
  ↓
critical section
  ↓
Unlock
```

Možemo definisati invariant:

```text
dok držimo lock
state je konzistentan
```

Kod lock-free pristupa imamo mnogo mogućih interleaving-a:

```text
G1 → Load
G2 → Load
G3 → CAS
G1 → CAS fail
G2 → CAS fail
G1 → Load
...
```

Potrebno je dokazati:

* atomicity,
* ordering,
* ABA behavior,
* memory reclamation,
* progress guarantee,
* linearizability,
* state invariants.

To je mnogo zahtevnije.

Zato Senior+ kandidat treba da bude sposoban da objasni ne samo:

> „Ovo je lock-free.“

već:

> „Koju progress garanciju daje, gde je linearization point i kako dokazujemo da nijedna dozvoljena interleaving sekvenca ne narušava invariant?“

---

## Pitanje 128

**Šta je linearizability?**

### Odgovor

Linearizability je correctness property concurrent objekta.

Intuicija je:

> Svaka concurrent operacija treba da izgleda kao da se dogodila atomarno u nekoj tački između svog poziva i završetka.

Ta tačka se naziva:

```text
linearization point
```

Na primer:

```text
G1: Add(10)
        │
        ├──────────────┐
        │              │
G2: Add(20)            │
        │              │
        └──────────────┘
```

Iako se operacije preklapaju u realnom vremenu, observable rezultat treba da bude ekvivalentan nekom validnom sekvencijalnom redosledu:

```text
Add(10)
Add(20)
```

ili:

```text
Add(20)
Add(10)
```

u zavisnosti od konkretne implementacije.

Linearizability je posebno važna kod:

* concurrent queues,
* stacks,
* maps,
* counters,
* state machines.

---

## Pitanje 129

**Da li je svaki lock-free algoritam linearizable?**

### Odgovor

Ne.

Lock-free opisuje:

```text
progress guarantee
```

Linearizability opisuje:

```text
correctness guarantee
```

To su različite dimenzije.

Možemo imati algoritam koji:

```text
lock-free
```

ali:

```text
logically incorrect
```

Ako state transitions nisu pravilno dizajnirane, algoritam može garantovati da neko napreduje, ali dati pogrešan rezultat.

Zato concurrency algoritam treba analizirati najmanje kroz:

```text
Correctness
+
Memory ordering
+
Progress
+
Performance
```

---

## Pitanje 130

**Šta je ABA problem?**

### Odgovor

ABA problem nastaje kada thread pročita vrednost:

```text
A
```

zatim drugi thread promeni:

```text
A → B → A
```

a prvi thread zatim izvrši CAS koji proverava samo:

```text
expected = A
```

i zaključi:

```text
"A se nije promenio."
```

Primer:

```text
T1:
read A

T2:
A → B
B → A

T1:
CAS(A, new)
```

T1 ne zna da je state prošao kroz:

```text
A → B → A
```

To može biti problem ako sama vrednost nije dovoljna da reprezentuje istoriju ili validnost state-a.

Rešenja zavise od strukture problema, na primer:

```text
version/tag
```

uz state:

```text
(A, version=1)
```

pa:

```text
A → B → A
```

postaje:

```text
(A,1)
→ (B,2)
→ (A,3)
```

CAS tada može detektovati da se state ipak promenio.

---

## Pitanje 131

**Zašto memory reclamation predstavlja problem kod lock-free struktura?**

### Odgovor

Pretpostavimo lock-free linked list:

```text
A → B → C
```

Goroutine G1 pročita:

```text
B
```

Dok G1 radi, G2 ukloni B:

```text
A → C
```

i oslobodi/reclaim-uje B.

Ako G1 zatim koristi stari pointer:

```text
G1 → B
```

možemo imati problem sa životnim vekom memorije.

U jezicima sa garbage collector-om deo problema je drugačiji nego u manuelno upravljanim memorijskim sistemima, ali logical lifetime i object reachability i dalje moraju biti pravilno dizajnirani.

Kod kompleksnih lock-free struktura moramo razmišljati o:

```text
publication
ownership
reachability
state transition
reclamation
```

Ne treba pretpostaviti:

> „GC znači da memory reclamation problem ne postoji.“

GC rešava određenu klasu memory lifetime problema, ali ne rešava automatski sve correctness probleme concurrent pointer-based strukture.

---

## Pitanje 132

**Zašto exponential backoff može poboljšati lock-free algoritam?**

### Odgovor

Ako veliki broj goroutine-a konstantno izvršava:

```text
CAS
 ↓
failure
 ↓
CAS
 ↓
failure
```

CPU može biti preplavljen retry pokušajima.

Backoff uvodi razmak između pokušaja:

```text
CAS fail
   ↓
wait
   ↓
retry
```

Na primer:

```text
1st failure → 10µs
2nd failure → 20µs
3rd failure → 40µs
4th failure → 80µs
```

Randomization dodatno smanjuje verovatnoću da svi goroutine-i ponovo pokušaju istovremeno.

Efekat:

```text
CAS contention ↓
CPU waste ↓
successful operations ↑
```

Ali prevelik backoff može povećati latency.

Zato je potrebno benchmark-ovati:

```text
throughput
p50
p95
p99
CPU
CAS failures
```

---

## Pitanje 133

**Zašto lock-free nije sinonim za visok performans?**

### Odgovor

Performanse zavise od workload-a.

Lock-free algoritam može biti odličan kada:

```text
contention = low/moderate
operation = very short
latency requirement = strict
```

Ali kod ekstremnog contention-a:

```text
100 goroutines
      ↓
same atomic variable
```

može doći do ogromnog broja CAS failure-a.

Mutex može tada imati bolju praktičnu efikasnost:

```text
goroutine
   ↓
sleep/block
   ↓
resource available
   ↓
continue
```

umesto:

```text
goroutine
   ↓
CAS
   ↓
fail
   ↓
CAS
   ↓
fail
   ↓
CAS
```

Zato je izbor:

```text
Mutex vs Atomic vs Lock-free
```

performance pitanje tek **nakon** što je correctness definisana i nakon što benchmark pokaže bottleneck.

---

## Senior+ princip

Napredni concurrency algoritmi mogu se posmatrati kroz četiri dimenzije:

```text
                 Concurrent Algorithm
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    Correctness       Progress        Performance
        │                │                │
        ▼                ▼                ▼
 Linearizability   Lock-free/Wait-free  Contention
 Memory ordering   Starvation           Latency
 ABA               Livelock             Throughput
```

Najvažnije je ne pomešati ove koncepte.

```text
lock-free
    ≠
deadlock-free only

lock-free
    ≠
linearizable

atomic
    ≠
whole-state correctness

CAS
    ≠
automatically faster

wait-free
    ≠
automatically better
```

Senior+ concurrency inženjer ne bira lock-free algoritam zato što zvuči naprednije.

Bira ga kada konkretni workload, progress zahtev i performance karakteristike opravdavaju dodatnu kompleksnost — i kada može jasno da obrazloži **correctness, memory ordering, progress guarantee i performance trade-off**.

---

Senior+ concurrency dizajn ne podrazumeva samo pokretanje goroutine-a i njihovu koordinaciju.

Jednako važno pitanje je:

> **Kako goroutine završava?**

Production sistem mora imati jasno definisan lifecycle:

```text
Start
  ↓
Running
  ↓
Completed
```

ali i:

```text
Start
  ↓
Running
  ↓
Cancellation
  ↓
Cleanup
  ↓
Exit
```

Ako cancellation nije deo dizajna od početka, vrlo lako nastaju:

* goroutine leaks,
* blokirani goroutine-i,
* neoslobođeni resursi,
* nepotrebni network requests,
* beskonačno čekanje,
* cascading failures,
* nepredvidivo gašenje servisa.

U Go-u je `context.Context` centralni mehanizam za propagaciju:

* cancellation,
* deadline-a,
* timeout-a,
* request-scoped metadata.

Važno je, međutim, razumeti da `context` **ne prekida goroutine nasilno**.

On daje signal:

> „Treba da prekineš rad čim bezbedno možeš.“

---

# Pitanje 134

**Šta je cancellation u concurrent sistemu?**

### Odgovor

Cancellation je mehanizam kojim jedan deo sistema signalizira drugom delu:

```text
"Više nema potrebe da nastavljaš ovaj posao."
```

Na primer:

```text
HTTP request
    ↓
service
    ↓
database query
    ↓
external API
```

Ako klijent prekine HTTP request:

```text
client
   ↓
disconnect
```

nema smisla da backend nastavi:

```text
database query
       +
external API call
       +
CPU-intensive processing
```

Ako je rezultat tog rada namenjen samo prekinutom request-u.

Cancellation omogućava:

```text
request cancelled
      ↓
context cancelled
      ↓
downstream work stops
      ↓
resources released
```

To je važan deo lifecycle management-a.

---

# Pitanje 135

**Da li `context.Context` prekida goroutine?**

### Odgovor

Ne.

Ovo je fundamentalno pravilo.

`context.Context` ne poseduje mehanizam kojim može nasilno da ubije goroutine.

Umesto toga goroutine mora eksplicitno proveravati cancellation signal:

```go
select {
case <-ctx.Done():
    return
default:
}
```

ili:

```go
select {
case <-ctx.Done():
    return ctx.Err()

case result := <-results:
    return process(result)
}
```

Conceptualno:

```text
Context
   │
   │ cancellation signal
   ▼
goroutine
   │
   ├── observe
   ├── cleanup
   └── return
```

Dakle:

```text
context cancellation
        ≠
goroutine termination
```

Već:

```text
context cancellation
        ↓
cooperative cancellation
        ↓
goroutine decides to exit
```

---

# Pitanje 136

**Zašto je cooperative cancellation bolji model od prisilnog prekida goroutine-a?**

### Odgovor

Goroutine možda trenutno drži:

* mutex,
* file descriptor,
* database transaction,
* network connection,
* temporary resource,
* partially updated state.

Nasilno prekidanje bi moglo ostaviti sistem u nekonzistentnom stanju.

Cooperative cancellation omogućava:

```text
       receive cancellation
               ↓
    stop accepting new work
               ↓
finish / rollback critical operation
               ↓
        release resources
               ↓
            return
```

Na primer:

```go
func worker(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()

        case job := <-jobs:
            if err := process(ctx, job); err != nil {
                return err
            }
        }
    }
}
```

Worker sam kontroliše gde je bezbedno da prekine posao.

---

# Pitanje 137

**Koja je razlika između `context.WithCancel`, `context.WithTimeout` i `context.WithDeadline`?**

### Odgovor

`WithCancel` omogućava eksplicitno otkazivanje:

```go
ctx, cancel := context.WithCancel(parent)
defer cancel()
```

Cancellation se dešava kada pozovemo:

```go
cancel()
```

`WithTimeout` definiše relativno vremensko ograničenje:

```go
ctx, cancel := context.WithTimeout(parent, 2*time.Second)
defer cancel()
```

Kada prođu dve sekunde:

```text
deadline reached
      ↓
context cancelled
```

`WithDeadline` koristi apsolutni trenutak:

```go
ctx, cancel := context.WithDeadline(parent, deadline)
defer cancel()
```

Conceptualno:

```text
WithCancel
    ↓
explicit cancellation

WithTimeout
    ↓
now + duration

WithDeadline
    ↓
specific instant
```

Sva tri mehanizma propagiraju cancellation downstream.

---

# Pitanje 138

**Zašto treba pozvati `cancel()` čak i kada timeout još nije istekao?**

### Odgovor

Tipičan kod:

```go
ctx, cancel := context.WithTimeout(
    parent,
    5*time.Second,
)
defer cancel()
```

`defer cancel()` je važan zato što omogućava oslobađanje resursa povezanih sa child context-om čim funkcija završi.

Ako operacija uspe nakon:

```text
100ms
```

nema razloga da context ostane aktivan do:

```text
5s
```

Dakle:

```text
operation completed
      ↓
cancel()
      ↓
cleanup
```

je bolji lifecycle.

Čak i kada je cancellation već automatski izazvan deadline-om, pozivanje `cancel()` je standardan i bezbedan obrazac kada smo dobili cancel function.

---

# Pitanje 139

**Šta znači propagacija cancellation-a?**

### Odgovor

Ako imamo:

```text
Request
   ↓
Service A
   ↓
Service B
   ↓
Database
```

parent context može propagirati cancellation:

```text
Request cancelled
      ↓
ctx.Done()
      ↓
Service A
      ↓
Service B
      ↓
Database operation
```

Ako svaki sloj prihvata:

```go
ctx context.Context
```

možemo održati cancellation chain.

Na primer:

```go
func Handle(ctx context.Context) error {
    return service.Process(ctx)
}

func (s *Service) Process(ctx context.Context) error {
    return s.repo.Load(ctx)
}

func (r *Repository) Load(ctx context.Context) error {
    return r.db.QueryContext(ctx, query)
}
```

Cancellation se tako propagira kroz ceo execution graph.

---

# Pitanje 140

**Zašto nije dobro kreirati novi `context.Background()` unutar funkcije koja već dobija context?**

### Odgovor

Na primer:

```go
func Process(ctx context.Context) error {
    ctx = context.Background()

    return doWork(ctx)
}
```

ovim prekidamo parent-child cancellation chain.

Ako je parent context otkazan:

```text
request cancelled
      ↓
parent ctx cancelled
```

novi:

```text
context.Background()
```

ne zna ništa o tome.

Dobijamo:

```text
Request
   X
   │
   │ cancellation
   ▼
Service

new Background()
   │
   ▼
work continues
```

To može izazvati:

* goroutine leak,
* unnecessary work,
* request cancellation ignorisanje,
* nepotrebno opterećenje downstream servisa.

`context.Background()` je tipično root context, a ne nešto što treba proizvoljno koristiti unutar request lifecycle-a.

---

# Pitanje 141

**Zašto `context` treba prosleđivati eksplicitno kao prvi argument funkcije?**

### Odgovor

Idiomatic Go API obično izgleda:

```go
func DoWork(ctx context.Context, arg Arg) error
```

a ne:

```go
func DoWork(arg Arg) error
```

gde se context krije negde u:

```text
global variable
struct field
package state
```

Eksplicitni `ctx` pokazuje da operacija pripada određenom lifecycle-u.

Na primer:

```go
func FetchUser(
    ctx context.Context,
    id string,
) (User, error)
```

jasno pokazuje:

```text
caller owns lifecycle
function observes lifecycle
```

To poboljšava:

* testiranje,
* composability,
* cancellation propagation,
* API transparentnost.

---

# Pitanje 142

**Da li context treba čuvati u struct-u?**

### Odgovor

Uobičajeno — ne.

Loš obrazac:

```go
type Service struct {
    ctx context.Context
}
```

pa:

```go
service.ctx = ctx
```

Context je vezan za određenu operaciju, request ili lifecycle.

Service objekat često živi mnogo duže od jednog request-a:

```text
Service lifetime
────────────────────────────────────────>

Request 1
   │
Request 2
   │
Request 3
   │
Request 4
```

Ako context stavimo u service:

```text
Service
   │
   └── Context(Request 1)
```

lifecycle postaje pogrešno modelovan.

Bolje:

```go
func (s *Service) Process(ctx context.Context) error
```

Context pripada operaciji.

---

# Pitanje 143

**Šta je goroutine leak?**

### Odgovor

Goroutine leak nastaje kada goroutine ostane aktivan duže nego što je potrebno i nema realnu mogućnost da završi.

Tipičan primer:

```go
func worker() {
    for {
        job := <-jobs
        process(job)
    }
}
```

Ako `jobs` nikada neće biti zatvoren niti će worker dobiti cancellation signal:

```text
worker
  ↓
waiting forever
```

Svaki poziv koji kreira takvog workera može doprineti rastu broja goroutine-a:

```text
request 1 → +1 goroutine
request 2 → +1 goroutine
request 3 → +1 goroutine
...
```

Posledica može biti:

```text
goroutines ↑
memory ↑
scheduler overhead ↑
resource usage ↑
```

Goroutine leak je lifecycle bug.

---

# Pitanje 144

**Kako `context` može pomoći u sprečavanju goroutine leak-a?**

### Odgovor

Umesto:

```go
for {
    job := <-jobs
    process(job)
}
```

worker može imati cancellation path:

```go
for {
    select {
    case <-ctx.Done():
        return

    case job := <-jobs:
        process(job)
    }
}
```

Lifecycle sada ima eksplicitan exit:

```text
worker starts
    ↓
wait
    ↓
work
    ↓
wait
    ↓
ctx.Done()
    ↓
return
```

Ovo je veoma važan pattern za dugotrajne goroutine-e.

Svaki goroutine koji može blokirati neograničeno dugo treba imati jasno definisan način završetka.

---

# Pitanje 145

**Kako bi detektovao goroutine leak u production sistemu?**

### Odgovor

Prvo bih posmatrao:

```text
runtime.NumGoroutine()
```

i trend:

```text
goroutine count
    time →
```

Ako broj kontinuirano raste bez očekivanog razloga:

```text
100
120
150
200
300
500
```

to je signal za istragu.

Dalje bih koristio:

* pprof goroutine profile,
* goroutine dumps,
* tracing,
* metrics,
* structured logging.

Goroutine profile može pokazati gde goroutine-i trenutno čekaju.

Na primer veliki broj goroutine-a na:

```text
chan receive
```

može ukazivati na channel lifecycle problem.

Veliki broj na:

```text
sync.Mutex.Lock
```

može ukazivati na contention.

Važno je posmatrati **trend**, a ne samo apsolutni broj.

Veliki broj goroutine-a sam po sebi nije automatski leak.

---

# Pitanje 146

**Šta je timeout, a šta deadline?**

### Odgovor

Timeout predstavlja relativno trajanje:

```text
"operacija sme trajati najviše 2 sekunde."
```

Deadline predstavlja apsolutni trenutak:

```text
"operacija mora biti završena do 12:00:05."
```

Primer:

```go
context.WithTimeout(ctx, 2*time.Second)
```

znači:

```text
now + 2s
```

dok:

```go
context.WithDeadline(ctx, deadline)
```

znači:

```text
specific point in time
```

Deadline je posebno koristan kada imamo lanac poziva.

Na primer:

```text
incoming request deadline
          ↓
service
          ↓
database
          ↓
external API
```

Svaki downstream sloj treba da poštuje preostali budžet.

---

# Pitanje 147

**Zašto svaki downstream poziv ne treba da dobije novi fiksni timeout?**

### Odgovor

Pretpostavimo da request ima:

```text
deadline = 2s
```

Service A kaže:

```text
database timeout = 2s
```

zatim:

```text
external API timeout = 2s
```

Ako se pozivi izvršavaju sekvencijalno:

```text
DB → 2s
API → 2s
```

ukupno možemo čekati:

```text
4s
```

iako je originalni request imao samo:

```text
2s
```

Zato je bolje propagirati postojeći context:

```go
db.QueryContext(ctx, ...)
```

i dozvoliti da downstream operacija poštuje parent deadline.

Conceptualno:

```text
Request budget: 2s
      │
      ├── DB: remaining budget
      │
      └── API: remaining budget
```

Timeout treba posmatrati kao **latency budget**, a ne samo kao lokalnu konfiguracionu vrednost.

---

# Pitanje 148

**Šta znači cancellation-aware I/O?**

### Odgovor

Cancellation-aware I/O znači da I/O operacija može biti prekinuta kada context bude otkazan.

Na primer, database API često ima:

```go
QueryContext(ctx, query)
```

umesto:

```go
Query(query)
```

Ako request bude otkazan:

```text
ctx.Done()
    ↓
database operation cancelled
```

Bez cancellation-aware API-ja možemo imati:

```text
request cancelled
      ↓
handler exits
      ↓
database query continues
```

što je nepotreban rad.

Kod network operacija treba koristiti API-je koji podržavaju context ili eksplicitne deadlines/timeouts.

---

# Pitanje 149

**Kako cancellation utiče na worker pool?**

### Odgovor

Worker pool treba da ima definisan shutdown protocol.

Na primer:

```text
        jobs
         │
   ┌─────┼─────┐
   ▼     ▼     ▼
 worker worker worker
   │     │     │
   └─────┼─────┘
         ▼
       results
```

Kada context bude otkazan:

```text
ctx.Done()
   │
   ├── worker 1 → exit
   ├── worker 2 → exit
   └── worker 3 → exit
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

            process(ctx, job)
        }
    }
}
```

Worker treba da reaguje i na:

```text
channel closed
```

i na:

```text
context cancellation
```

u zavisnosti od lifecycle modela.

---

# Pitanje 150

**Šta je graceful shutdown?**

### Odgovor

Graceful shutdown znači da sistem prestaje da prihvata novi rad, ali kontrolisano završava ili otkazuje postojeći rad.

Tipičan servis lifecycle:

```text
Running
   │
   │ shutdown signal
   ▼
Stop accepting new requests
   │
   ▼
Cancel / drain work
   │
   ▼
Wait for workers
   │
   ▼
Close resources
   │
   ▼
Exit
```

Na primer:

```text
HTTP server
    ↓
stop accepting new connections
    ↓
existing handlers finish
    ↓
background workers stop
    ↓
DB connections close
    ↓
process exits
```

Bez graceful shutdown-a možemo imati:

* prekinute request-e,
* izgubljene poruke,
* nepotpune transakcije,
* goroutine leak,
* corrupted application-level state.

---

# Pitanje 151

**Kako bi dizajnirao shutdown sequence za Go servis?**

### Odgovor

Jedan robustan pristup:

### 1. Primiti shutdown signal

Na primer:

```text
SIGTERM
```

### 2. Zaustaviti prijem novog rada

Na primer:

```text
HTTP server shutdown
```

### 3. Cancel root context

```go
cancel()
```

### 4. Propagirati cancellation

```text
root context
      ↓
workers
      ↓
downstream operations
```

### 5. Sačekati goroutine-e

```go
wg.Wait()
```

### 6. Osloboditi resurse

Na primer:

```text
database
connections
files
listeners
```

### 7. Završiti proces

Idealni lifecycle:

```text
SIGTERM
  ↓
stop accepting
  ↓
cancel
  ↓
drain/cleanup
  ↓
wait
  ↓
exit
```

Bitno je da shutdown bude **determinističan**, a ne samo:

```go
time.Sleep(5 * time.Second)
os.Exit(0)
```

---

# Pitanje 152

**Šta je problem sa `os.Exit()` tokom graceful shutdown-a?**

### Odgovor

`os.Exit()` trenutno završava proces.

To znači da normalan Go cleanup mehanizam ne dobija priliku da završi svoj posao na isti način kao kod normalnog povratka iz funkcije.

Ako uradimo:

```go
os.Exit(0)
```

pre nego što:

```text
workers finish
transactions complete
buffers flush
connections close
```

možemo izgubiti rad.

Zato production shutdown treba prvo da izvrši:

```text
stop
cancel
drain
wait
cleanup
```

a tek onda dozvoli proces exit.

---

# Pitanje 153

**Kako sprečiti da cancellation ostavi sistem u nekonzistentnom stanju?**

### Odgovor

Cancellation ne treba posmatrati kao:

```text
"prekini sve odmah"
```

već:

```text
"nemoj započinjati novi nepotreban rad i završi trenutnu atomicnu jedinicu rada na bezbedan način."
```

Na primer, ako worker izvršava:

```text
BEGIN transaction
    ↓
update A
    ↓
update B
    ↓
COMMIT
```

nije dobro proizvoljno prekidati između:

```text
update A
```

i:

```text
update B
```

ako cancellation semantics baze ne garantuju rollback.

Bolje:

```text
ctx cancelled
    ↓
do not start next transaction
    ↓
finish/rollback current transaction
    ↓
return
```

Cancellation mora biti usklađen sa **transaction boundaries**.

---

# Pitanje 154

**Da li treba proveravati `ctx.Err()` ili `<-ctx.Done()`?**

### Odgovor

Zavisi od potrebe.

Ako želimo blokirajuće čekanje:

```go
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-results:
    return result
}
```

`Done()` daje channel koji signalizira cancellation.

Ako želimo samo da proverimo trenutno stanje:

```go
if err := ctx.Err(); err != nil {
    return err
}
```

`Err()` vraća:

```text
context.Canceled
```

ili:

```text
context.DeadlineExceeded
```

ili:

```text
nil
```

zavisno od stanja.

Tipičan obrazac:

```go
select {
case <-ctx.Done():
    return ctx.Err()
default:
}
```

koristan je kada želimo non-blocking cancellation check.

---

# Pitanje 155

**Zašto cancellation treba proveravati na pravim granicama, a ne samo jednom na početku funkcije?**

### Odgovor

Ako funkcija traje dugo:

```go
func Process(ctx context.Context) error {
    if err := ctx.Err(); err != nil {
        return err
    }

    expensiveStep1()
    expensiveStep2()
    expensiveStep3()

    return nil
}
```

cancellation može nastupiti tokom:

```text
step1
```

a funkcija neće proveriti context do samog kraja.

Bolje je imati cancellation-aware granice:

```text
step1
  ↓
check
  ↓
step2
  ↓
check
  ↓
step3
  ↓
check
```

Još bolje je da same dugotrajne operacije podržavaju context:

```go
expensiveStep(ctx)
```

Tada cancellation može prekinuti čekanje bez potrebe za ručnim polling-om.

---

# Pitanje 156

**Šta je structured concurrency i kako se uklapa u Go?**

### Odgovor

Structured concurrency je princip prema kome je lifecycle concurrent task-a vezan za lifecycle njegovog parent scope-a.

Conceptualno:

```text
Parent operation
│
├── Child task A
├── Child task B
└── Child task C
```

Parent treba da zna:

* koje child task-ove je kreirao,
* kada su završili,
* da li je neki failovao,
* kako se cancellation propagira,
* kada je bezbedno da se parent završi.

Go nema jedan jedini primitive koji sve ovo rešava, ali kombinacija:

```text
context
WaitGroup
errgroup
channels
```

omogućava implementaciju structured concurrency obrazaca.

Na primer:

```text
request
   │
   ├── worker A
   ├── worker B
   └── worker C
```

Ako B failuje:

```text
B → error
   ↓
cancel context
   ↓
A stops
C stops
   ↓
wait
   ↓
request returns error
```

Ovakav model je često mnogo pouzdaniji od slobodnog kreiranja goroutine-a bez ownership-a.

---

# Pitanje 157

**Zašto svaki goroutine treba da ima jasno definisanog owner-a?**

### Odgovor

Ako funkcija kreira goroutine:

```go
go backgroundWork()
```

treba da znamo:

```text
Ko ga zaustavlja?
Ko čeka da završi?
Ko obrađuje njegov error?
Šta se dešava ako parent request bude otkazan?
```

Ako na ova pitanja nema odgovora, lifecycle je verovatno loše dizajniran.

Dobro pravilo je:

```text
creator
   ↓
owns lifecycle
```

Owner zna:

```text
start
cancel
wait
cleanup
error propagation
```

Na primer:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()
    worker(ctx)
}()

// owner eventually:
wg.Wait()
```

Goroutine ne treba da postane „nečiji tuđi problem“.

---

# Pitanje 158

**Kako bi dizajnirao API koji garantuje da goroutine ne može ostati zauvek blokiran?**

### Odgovor

Prvo bih identifikovao sve blocking points:

```text
channel receive
channel send
mutex
I/O
condition
external call
```

Zatim bih za svaki definisao exit path.

Na primer:

```go
func worker(ctx context.Context, jobs <-chan Job) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()

        case job, ok := <-jobs:
            if !ok {
                return nil
            }

            if err := process(ctx, job); err != nil {
                return err
            }
        }
    }
}
```

Sada worker ima najmanje dva završna puta:

```text
jobs closed
    ↓
return

context cancelled
    ↓
return
```

Ako `process` podržava context, imamo i cancellation tokom samog rada.

Ovo je mnogo pouzdanije od:

```go
for job := range jobs {
    process(job)
}
```

kada `jobs` nema garantovan lifecycle.

---

# Pitanje 159

**Šta je cancellation storm i kako može nastati?**

### Odgovor

Cancellation storm nastaje kada cancellation jednog događaja izazove veliki broj istovremenih cancellation reakcija.

Na primer:

```text
Root request cancelled
        │
        ├── 100 workers
        ├── 500 downstream tasks
        ├── 1000 pending operations
        └── many cleanup handlers
```

Svi mogu istovremeno početi:

```text
cleanup
retry cancellation
close resources
emit logs
```

Ako cleanup sam generiše značajan workload, sistem može dobiti dodatni spike.

Zato cancellation path treba biti:

* idempotentan,
* brz,
* predvidljiv,
* bez nepotrebnog retry-ja,
* bez blokiranja na resurse koji se više ne očekuju.

Cancellation treba smanjiti workload, a ne proizvesti novi workload storm.

---

# Pitanje 160

**Kako razlikovati `context.Canceled` od `context.DeadlineExceeded`?**

### Odgovor

Ove greške nose različitu semantiku.

`context.Canceled` znači da je context eksplicitno otkazan.

Na primer:

```go
cancel()
```

`context.DeadlineExceeded` znači da je dostignut deadline.

Na primer:

```go
context.WithTimeout(ctx, time.Second)
```

Ako operacija traje predugo:

```text
deadline
   ↓
context.DeadlineExceeded
```

Ova razlika može biti važna za observability i business logic.

Na primer:

```text
Canceled
    ↓
request no longer needed

DeadlineExceeded
    ↓
operation too slow
```

U production sistemu nije dobro sve context greške pretvoriti u generičko:

```text
"operation failed"
```

jer se gubi važna informacija o uzroku.

---

# Pitanje 161

**Da li treba retry-ovati operaciju nakon `context.Canceled`?**

### Odgovor

U pravilu ne.

Ako je caller rekao:

```text
cancel
```

retry može biti direktno suprotan njegovoj nameri.

Na primer:

```text
HTTP request cancelled
      ↓
DB call canceled
      ↓
retry DB call
      ↓
wasted work
```

Tipično:

```text
context.Canceled
    ↓
stop
```

Kod:

```text
context.DeadlineExceeded
```

situacija može biti složenija.

Ponekad je deadline signal da je operation predugo trajala i retry takođe nema smisla.

Ali retry odluka mora zavisiti od:

* operation semantics,
* idempotency,
* remaining deadline,
* error type,
* retry budget.

Nikada ne treba slepo implementirati:

```go
for {
    err := operation()
    if err != nil {
        retry()
    }
}
```

---

# Pitanje 162

**Kako cancellation utiče na retry logic?**

### Odgovor

Retry loop mora biti cancellation-aware.

Loš primer:

```go
for attempts := 0; attempts < 10; attempts++ {
    err := operation()

    if err == nil {
        return nil
    }

    time.Sleep(time.Second)
}
```

Ako je context otkazan:

```text
ctx cancelled
     ↓
sleep continues
     ↓
retry continues
```

To je nepotreban rad.

Bolje:

```go
for attempts := 0; attempts < 10; attempts++ {
    err := operation(ctx)

    if err == nil {
        return nil
    }

    select {
    case <-ctx.Done():
        return ctx.Err()

    case <-time.After(backoff):
    }
}
```

Još bolje je izbeći kreiranje novih timer-a u tight loop-u i pažljivo upravljati timer lifecycle-om.

Princip je:

```text
retry
  ↓
check cancellation
  ↓
backoff
  ↓
retry
```

a ne:

```text
retry forever regardless of lifecycle
```

---

# Pitanje 163

**Kako bi objasnio cancellation budget u distribuiranom sistemu?**

### Odgovor

Pretpostavimo da incoming request ima:

```text
deadline = 2 seconds
```

On poziva:

```text
Service A
    ↓
Service B
    ↓
Service C
    ↓
Database
```

Ne može svaki servis da dobije novi nezavisan timeout od:

```text
2 seconds
```

jer bi ukupni latency mogao eksplodirati.

Umesto toga imamo jedan lifecycle budget:

```text
2s total
│
├── A
│
├── B
│
├── C
│
└── DB
```

Kako vreme prolazi:

```text
remaining budget ↓
```

Downstream operacije treba da poštuju taj budget.

Ovo je fundamentalni princip za kontrolu:

* tail latency,
* cascading delays,
* resource exhaustion,
* request amplification.

---

# Pitanje 164

**Kako cancellation može sprečiti cascading failure?**

### Odgovor

Bez cancellation-a:

```text
Service A
   ↓
Service B
   ↓
Service C
```

Ako Service A više ne treba rezultat:

```text
client disconnects
```

ali B i C nastavljaju rad:

```text
B → continues
C → continues
DB → continues
```

Veliki broj prekinutih request-a može tako proizvoditi ogromnu količinu nepotrebnog downstream rada.

Sa cancellation propagation:

```text
client disconnect
       ↓
A cancels
       ↓
B cancels
       ↓
C cancels
       ↓
DB cancels
```

Sistem brzo oslobađa resurse.

Cancellation je zato jedan od važnih mehanizama **load shedding-a i failure containment-a**.

---

# Pitanje 165

**Koje osobine treba da ima dobar cancellation protocol?**

### Odgovor

Dobar cancellation protocol treba da bude:

### 1. Propagatable

Signal treba da može da putuje kroz call graph.

### 2. Cooperative

Task sam odlučuje kako bezbedno završava.

### 3. Prompt

Dugotrajni posao treba relativno brzo da reaguje.

### 4. Idempotentan

Višestruki cancellation/cleanup pozivi ne smeju izazvati korupciju.

### 5. Resource-safe

Cancellation mora voditi ka:

```text
cleanup
```

a ne ka:

```text
leak
```

### 6. Observable

Treba znati:

```text
što je otkazano
zašto
koliko često
koliko dugo cleanup traje
```

### 7. Composable

Treba raditi kroz:

```text
service
repository
worker
I/O
external calls
```

bez ručnog pravljenja potpuno različitih cancellation mehanizama.

---

# Senior+ mentalni model

Lifecycle concurrent sistema možemo predstaviti ovako:

```text
                    START
                      │
                      ▼
                  RUNNING
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       SUCCESS      ERROR     CANCELLATION
          │           │           │
          └───────────┼───────────┘
                      ▼
                   CLEANUP
                      │
                      ▼
                    WAIT
                      │
                      ▼
                    EXIT
```

Dobar Go concurrency dizajn mora odgovoriti na sva pitanja:

```text
Ko kreira goroutine?
Ko je owner?
Ko ga otkazuje?
Kako dobija cancellation?
Na kojim tačkama može blokirati?
Kako se propagira deadline?
Kako se oslobađaju resursi?
Ko čeka završetak?
Kako se propagiraju error-i?
Šta se dešava tokom shutdown-a?
```

Najvažniji Senior+ princip je:

> **Goroutine nije samo jedinica izvršavanja; ona je resurs sa lifecycle-om koji mora imati jasan ownership, cancellation i termination protocol.**

Ako je odgovor na pitanje:

```text
"Kako se ova goroutine završava?"
```

nejasan, postoji realna mogućnost da je concurrent dizajn nepotpun.

---

Na Senior+ nivou nije dovoljno znati da channel služi za komunikaciju između goroutine-a.

Potrebno je razumeti channel kao **sinhronizacioni primitive sa preciznim lifecycle-om, ownership modelom i kapacitetom koji direktno utiče na ponašanje sistema pod opterećenjem**.

Kod ozbiljnog concurrent sistema pitanja više nisu samo:

```text
"Kako da pošaljem vrednost kroz channel?"
```

već:

```text
Ko poseduje channel?
Ko ga zatvara?
Šta predstavlja buffer?
Šta se dešava kada consumer zaostaje?
Da li sistem ima backpressure?
Šta se dešava kada producer otkaže rad?
Kako sprečiti deadlock?
Da li channel uopšte treba koristiti?
```

---

# Pitanje 166

**Ko treba da zatvori channel?**

### Odgovor

Opšte pravilo je:

> **Goroutine koja je owner producer strane i koja zna da više nikada neće poslati vrednost treba da zatvori channel.**

Na primer:

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

Producer zna:

```text
"Ja sam završio slanje."
```

zato producer zatvara channel.

Consumer:

```go
for value := range producer() {
    fmt.Println(value)
}
```

ne zatvara channel.

Model ownership-a:

```text
producer
   │
   │ owns sending lifecycle
   ▼
channel
   │
   ▼
consumer
```

Važno je razlikovati:

```text
send ownership
```

od:

```text
receive ownership
```

i:

```text
close ownership
```

---

# Pitanje 167

**Šta se dešava ako consumer zatvori channel koji producer još koristi?**

### Odgovor

Producer može pokušati:

```go
ch <- value
```

nakon što je consumer uradio:

```go
close(ch)
```

što izaziva:

```text
panic: send on closed channel
```

Zbog toga je ovo opasan obrazac:

```go
go producer(ch)

for value := range ch {
    if shouldStop {
        close(ch)
        break
    }

    process(value)
}
```

Consumer možda misli:

```text
"Ne treba mi više channel."
```

ali producer možda još uvek ima validne vrednosti za slanje.

Bolji dizajn koristi odvojeni cancellation signal:

```text
consumer
   │
   │ cancel
   ▼
context
   │
   ▼
producer
   │
   └── stops sending
```

Dakle:

```text
close(channel)
```

nije generalni mehanizam za:

```text
"Molim te, prestani da šalješ."
```

Za to je često pogodniji:

```text
context cancellation
```

---

# Pitanje 168

**Da li `close(channel)` treba posmatrati kao signal producer-a ili consumer-a?**

### Odgovor

Najčešće kao signal producer-a:

```text
producer
   │
   │ "Nema više vrednosti."
   ▼
close(ch)
   │
   ▼
consumer
```

Consumer tada može detektovati završetak:

```go
value, ok := <-ch

if !ok {
    // channel closed
}
```

ili:

```go
for value := range ch {
    process(value)
}
```

`range` završava kada channel bude zatvoren i kada se isprazni eventualni buffer.

Semantika je:

```text
close
  ↓
no more sends allowed
  ↓
existing buffered values remain readable
  ↓
eventually receive returns zero value + ok=false
```

---

# Pitanje 169

**Da li zatvaranje buffered channel-a odbacuje vrednosti koje se nalaze u buffer-u?**

### Odgovor

Ne.

Na primer:

```go
ch := make(chan int, 3)

ch <- 10
ch <- 20
ch <- 30

close(ch)
```

Consumer i dalje može dobiti:

```text
10
20
30
```

Tek nakon pražnjenja buffera:

```go
value, ok := <-ch
```

dobija:

```text
value = 0
ok = false
```

Dakle:

```text
close(ch)
    │
    ├── buffered values remain
    │
    ▼
buffer drained
    │
    ▼
receive → zero value, false
```

Ovo je važno kod graceful completion semantics-a.

---

# Pitanje 170

**Šta znači backpressure u concurrent sistemu?**

### Odgovor

Backpressure znači da sporiji downstream consumer ograničava brzinu kojom upstream producer može da generiše ili šalje work.

Na primer:

```text
Producer
   │
   │ 1000 jobs/s
   ▼
Channel
   │
   ▼
Consumer
   │
   │ 100 jobs/s
```

Ako nema dovoljno buffer-a ili drugog mehanizma, producer će na kraju morati da čeka.

Kod unbuffered channel-a:

```go
ch := make(chan Job)
```

send:

```go
ch <- job
```

čeka dok receiver ne preuzme vrednost.

To predstavlja prirodni oblik backpressure-a.

---

# Pitanje 171

**Zašto buffered channel može smanjiti backpressure, ali ga ne uklanja?**

### Odgovor

Ako imamo:

```go
ch := make(chan Job, 100)
```

producer može privremeno da generiše više posla nego što consumer trenutno obrađuje.

Na primer:

```text
Producer: 1000 jobs/s
Consumer: 100 jobs/s
Buffer:   100 jobs
```

Buffer samo omogućava određenu količinu akumulacije.

Kada se napuni:

```text
buffer full
   ↓
producer blocks
```

Dakle:

```text
buffer
  ↓
absorbs temporary bursts
```

ali ne rešava fundamentalni throughput mismatch.

Ako producer trajno proizvodi:

```text
1000 jobs/s
```

a consumer trajno obrađuje:

```text
100 jobs/s
```

onda će backlog pre ili kasnije postati problem.

---

# Pitanje 172

**Zašto veliki channel buffer nije automatski dobro rešenje za performance problem?**

### Odgovor

Veliki buffer može samo odložiti manifestaciju problema.

Na primer:

```go
make(chan Job, 1_000_000)
```

može omogućiti ogromnu akumulaciju posla.

Ali ako:

```text
producer throughput > consumer throughput
```

onda backlog raste.

Posledice mogu biti:

* velika potrošnja memorije,
* duže vreme čekanja,
* stale work,
* povećan latency,
* memory pressure,
* GC pressure,
* teže predviđanje sistema.

Umesto:

```text
"Stavi veći buffer."
```

Senior+ inženjer treba da pita:

```text
Zašto consumer ne može da prati producer?
Da li work treba odbacivati?
Da li treba limitirati concurrency?
Da li treba batching?
Da li treba load shedding?
Da li treba povećati consumer throughput?
Da li je red čekanja uopšte potreban?
```

Buffer size je **design parameter**, a ne magična performance optimizacija.

---

# Pitanje 173

**Kako channel može implementirati semaphore?**

### Odgovor

Buffered channel može predstavljati ograničen broj dozvola.

Na primer:

```go
sem := make(chan struct{}, 10)
```

Pre ulaska u critical concurrent operation:

```go
sem <- struct{}{}
```

Po završetku:

```go
<-sem
```

Ako je svih 10 slotova zauzeto:

```text
10 active operations
      ↓
next acquire blocks
```

Conceptualno:

```text
capacity = 10

┌──────────────────────────┐
│ permits                  │
│ [ ][ ][ ][ ][ ][ ] ...   │
└──────────────────────────┘
```

Time ograničavamo maksimalni broj istovremenih operacija.

Na primer:

```go
func process(ctx context.Context, job Job) error {
    select {
    case sem <- struct{}{}:
        defer func() {
            <-sem
        }()

        return execute(ctx, job)

    case <-ctx.Done():
        return ctx.Err()
    }
}
```

Ovde semaphore ima i cancellation-aware acquisition.

---

# Pitanje 174

**Koja je razlika između worker pool-a i semaphore-a?**

### Odgovor

Semaphore ograničava **koliko operacija sme istovremeno da se izvršava**.

Worker pool definiše **skup goroutine-a koji preuzimaju poslove**.

Semaphore:

```text
jobs
 │
 ├── acquire permit
 ├── execute
 └── release permit
```

Worker pool:

```text
             jobs
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
   worker  worker  worker
```

Worker pool obično daje:

* kontrolisan broj worker goroutine-a,
* centralizovan job queue,
* stabilan concurrency model.

Semaphore može biti jednostavniji kada već imamo direktne taskove i samo želimo:

```text
"maksimalno N istovremenih operacija."
```

Senior+ dizajn ne treba da bira primitive samo zato što je poznat, već prema lifecycle-u i ownership modelu.

---

# Pitanje 175

**Kada je channel bolji od mutex-a?**

### Odgovor

Channel je često dobar kada problem prirodno izgleda kao:

```text
producer → consumer
```

ili:

```text
stage A → stage B
```

ili:

```text
worker pool
```

Mutex je često bolji kada više goroutine-a deli state:

```text
shared state
    ↑
 ┌──┼──┐
 │  │  │
G1 G2 G3
```

i treba nam:

```text
mutual exclusion
```

Na primer:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}
```

Ako je cilj samo zaštita:

```text
n++
```

channel može biti nepotrebno komplikovan.

Go concurrency dizajn nije:

```text
"Channels everywhere."
```

nego:

```text
"Use the primitive that best expresses the synchronization model."
```

---

# Pitanje 176

**Kada channel nije dobar izbor za shared state?**

### Odgovor

Ako imamo jednostavan state:

```go
type Cache struct {
    mu sync.RWMutex
    data map[string]string
}
```

i operacije:

```text
Get
Set
Delete
```

channel-based ownership model može izgledati kao:

```text
request channel
     ↓
single owner goroutine
     ↓
map
```

To može biti validno, ali uvodi:

* dodatne goroutine-e,
* message passing,
* serialization,
* lifecycle complexity,
* potencijalni bottleneck.

Ako je problem jednostavno:

```text
"Zaštiti mapu od concurrent access-a."
```

mutex može biti direktnije rešenje.

---

# Pitanje 177

**Šta znači "Do not communicate by sharing memory; share memory by communicating" i da li to znači da mutex ne treba koristiti?**

### Odgovor

Ne.

To je često pogrešno interpretirano.

Poruka promoviše model u kome se ownership state-a može preneti komunikacijom, ali ne predstavlja zabranu mutex-a.

Ako imamo:

```text
goroutine A owns state
```

i druga goroutine šalje request:

```text
request → A
```

A može ostati jedini owner state-a.

Ali ako imamo shared state koji je prirodno modelovan kao concurrent data structure, mutex može biti jednostavniji i efikasniji.

Dakle:

```text
channels
```

i:

```text
mutexes
```

nisu međusobno suprotstavljene filozofije.

Oba su synchronization primitives.

---

# Pitanje 178

**Kako prepoznati channel koji je postao hidden queue?**

### Odgovor

Ako sistem koristi:

```go
jobs := make(chan Job, 100000)
```

i producer stalno puni channel dok consumer pokušava da sustigne backlog, channel praktično predstavlja queue.

Tada treba analizirati:

```text
queue depth
enqueue rate
dequeue rate
processing latency
drop rate
```

Ako je:

```text
arrival rate > service rate
```

queue će rasti.

Channel možda radi tehnički ispravno, ali arhitektura ima throughput problem.

Senior+ analiza zato ne treba da stane na:

```text
"nema deadlock-a."
```

nego treba da proveri:

```text
Da li sistem može stabilno da radi pod opterećenjem?
```

---

# Pitanje 179

**Šta je bounded concurrency i zašto je važan?**

### Odgovor

Bounded concurrency znači da sistem eksplicitno ograničava broj istovremenih operacija.

Na primer:

```text
max concurrency = 32
```

umesto:

```go
for _, job := range jobs {
    go process(job)
}
```

što može kreirati:

```text
100
1000
10000
100000
```

goroutine-a ako input eksplodira.

Bounded concurrency štiti:

* CPU,
* memoriju,
* database connections,
* external APIs,
* file descriptors,
* scheduler,
* downstream services.

Tipični mehanizmi:

```text
worker pool
semaphore
rate limiter
bounded queue
```

---

# Pitanje 180

**Zašto `go func()` unutar petlje može biti opasan pattern iako sam po sebi nije greška?**

### Odgovor

Problem nije samo u closure semantics-u, već i u concurrency explosion-u.

Na primer:

```go
for _, job := range jobs {
    go process(job)
}
```

Ako `jobs` ima:

```text
1,000,000
```

elemenata, program može pokušati da kreira ogroman broj goroutine-a.

To može dovesti do:

```text
memory pressure
scheduler pressure
downstream overload
connection exhaustion
```

Bolji model može biti:

```text
jobs
 │
 ▼
bounded queue
 │
 ├── worker
 ├── worker
 ├── worker
 └── worker
```

sa unapred definisanim concurrency limitom.

Senior+ pitanje nije:

```text
"Može li Go da kreira milion goroutine-a?"
```

nego:

```text
"Da li je neograničena konkurentnost validan production design?"
```

---

# Pitanje 181

**Šta je fan-out/fan-in obrazac?**

### Odgovor

Fan-out znači da jedan stream posla distribuiramo na više worker-a:

```text
             jobs
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
      W1     W2     W3
```

Fan-in znači da rezultate više worker-a spajamo:

```text
      W1 ──┐
      W2 ──┼──→ results
      W3 ──┘
```

Kombinacija:

```text
producer
    ↓
fan-out
    ↓
workers
    ↓
fan-in
    ↓
consumer
```

je veoma čest concurrency pattern.

Ali Senior+ dizajn mora rešiti:

```text
Ko zatvara results?
Šta ako worker failuje?
Šta ako consumer ode?
Kako se propagira cancellation?
Kako se sprečava blokiranje fan-in sender-a?
Kako se čeka na sve workere?
```

Pattern bez lifecycle semantics-a nije kompletan design.

---

# Pitanje 182

**Kako fan-in može izazvati goroutine leak?**

### Odgovor

Pretpostavimo:

```text
W1 ──┐
W2 ──┼──→ results
W3 ──┘
```

Ako downstream consumer prestane da čita:

```text
consumer exits
```

worker koji pokušava:

```go
results <- value
```

može zauvek čekati.

Tako dobijamo:

```text
worker
   ↓
send results
   ↓
blocked forever
```

Rešenje je cancellation-aware send:

```go
select {
case results <- value:
case <-ctx.Done():
    return
}
```

Sada:

```text
consumer stops
    ↓
ctx cancelled
    ↓
worker unblocks
    ↓
worker exits
```

Ovo je tipičan primer gde channel sam nije dovoljan; potreban je lifecycle protocol.

---

# Pitanje 183

**Kako bi dizajnirao cancellation-aware producer?**

### Odgovor

Producer ne treba slepo da šalje:

```go
ch <- value
```

ako receiver može nestati.

Bolji obrazac:

```go
select {
case ch <- value:
    // value sent
case <-ctx.Done():
    return ctx.Err()
}
```

Time producer ima dva izlaza:

```text
send succeeds
      │
      ▼
continue

OR

ctx cancelled
      │
      ▼
return
```

Ovaj pattern je naročito važan u:

* pipelines,
* fan-out,
* fan-in,
* worker pools,
* streaming systems.

---

# Pitanje 184

**Šta znači channel ownership u ozbiljnom concurrent API-ju?**

### Odgovor

Ownership znači da je jasno definisano:

```text
Ko kreira channel?
Ko šalje?
Ko prima?
Ko zatvara?
Ko čeka završetak?
```

Na primer:

```go
func Generate(ctx context.Context) <-chan Item
```

API jasno kaže:

```text
caller receives
implementation owns sending
implementation owns close
```

Directional channel tipovi dodatno dokumentuju ovaj contract:

```go
chan<- Item
```

znači:

```text
send-only
```

dok:

```go
<-chan Item
```

znači:

```text
receive-only
```

To smanjuje broj mogućih lifecycle grešaka.

---

# Pitanje 185

**Zašto je `chan<-` / `<-chan` više od compile-time convenience-a?**

### Odgovor

Directional channel tip predstavlja deo API contract-a.

Ako funkcija prima:

```go
func producer(out chan<- Item)
```

caller ne može unutar te funkcije koristiti:

```go
<-out
```

Slično:

```go
func consumer(in <-chan Item)
```

ne može slati:

```go
in <- item
```

Time compiler proverava deo ownership modela.

Conceptualno:

```text
producer
   │
   │ chan<- Item
   ▼
channel
   │
   │ <-chan Item
   ▼
consumer
```

Ovo je posebno korisno u većim codebase-ovima gde concurrency contracts treba da budu vidljivi iz API-ja.

---

# Pitanje 186

**Da li channel buffer treba birati proizvoljno?**

### Odgovor

Ne.

Buffer size treba da proizilazi iz konkretne concurrency semantike.

Mogući razlozi:

```text
1. absorb burst
2. decouple producer/consumer
3. avoid unnecessary blocking
4. batch work
5. bound queue
```

Ali treba razumeti šta kapacitet znači.

Na primer:

```go
make(chan Job, 10)
```

nije isto što i:

```go
make(chan Job, 10000)
```

jer drugi omogućava mnogo veću akumulaciju rada.

Kod production sistema treba meriti:

```text
queue depth
blocking time
throughput
latency
memory
```

i tek onda podešavati buffer.

---

# Pitanje 187

**Kako bi testirao backpressure behavior?**

### Odgovor

Ne bih testirao samo:

```text
"Da li rezultat odgovara?"
```

već i concurrency behavior.

Na primer:

```text
producer rate = 1000/s
consumer rate = 100/s
buffer = 10
```

Treba proveriti:

* da li producer blokira,
* koliko dugo blokira,
* da li cancellation prekida blocking send,
* da li queue ostaje bounded,
* da li se rad gubi ili čuva,
* kako se sistem ponaša kada consumer postane nedostupan.

Posebno važan test:

```text
consumer stops
      ↓
cancel context
      ↓
producer must exit
```

Ako test nikada ne završi:

```text
timeout
```

to može otkriti lifecycle bug.

---

# Pitanje 188

**Kako bi razlikovao throughput, latency i concurrency u channel-based pipeline-u?**

### Odgovor

Tri različite metrike:

### Throughput

Koliko posla sistem obrađuje po jedinici vremena.

```text
jobs/sec
```

### Latency

Koliko dugo jedan job čeka/prolazi kroz sistem.

```text
request → result
```

### Concurrency

Koliko je operacija istovremeno aktivno.

```text
active workers
```

Na primer možemo povećati concurrency:

```text
4 workers → 32 workers
```

ali dobiti lošiji latency ako downstream database postane bottleneck.

Isto tako možemo povećati channel buffer i smanjiti kratkoročno blocking producer-a, ali povećati queue latency.

Senior+ analiza zato ne optimizuje samo:

```text
"više goroutine-a"
```

već ceo performance envelope.

---

# Pitanje 189

**Šta je head-of-line blocking u concurrent pipeline-u?**

### Odgovor

Head-of-line blocking nastaje kada jedan spor element na početku reda sprečava obradu drugih elemenata.

Na primer:

```text
Queue:

[slow job] [fast job] [fast job] [fast job]
     │
     ▼
processing
```

Ako postoji samo jedan worker:

```text
slow job
   ↓
fast jobs čekaju
```

Čak i ako su fast jobs nezavisni.

Fan-out sa više worker-a može ublažiti problem:

```text
             jobs
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
     W1       W2       W3
    slow     fast     fast
```

Ali tada treba analizirati:

* ordering requirements,
* fairness,
* resource contention,
* downstream capacity.

Ne treba uvoditi paralelizaciju ako poslovna semantika zahteva striktan ordering.

---

# Pitanje 190

**Kada je neophodno očuvati ordering, a kada je bezbedno koristiti paralelizaciju?**

### Odgovor

Ako posao ima zavisnosti:

```text
A → B → C
```

ne možemo proizvoljno izvršiti:

```text
C → A → B
```

Ako su poslovi nezavisni:

```text
A
B
C
D
```

možemo ih paralelizovati.

Senior+ dizajn treba eksplicitno definisati:

```text
ordering guarantee
```

na primer:

```text
input order preserved
```

ili:

```text
completion order undefined
```

Ako ordering nije potreban, njegovo očuvanje može nepotrebno smanjiti concurrency.

Ako ordering jeste potreban, fan-out/fan-in možda zahteva sequence number:

```go
type Result struct {
    Index int
    Value Value
}
```

i kasniju rekonstrukciju redosleda.

---

# Pitanje 191

**Šta je load shedding i kako se razlikuje od backpressure-a?**

### Odgovor

Backpressure pokušava da uspori producer kada downstream ne može da prati.

Load shedding znači da sistem namerno odbacuje deo rada kada je preopterećen.

Na primer:

```text
incoming requests
        │
        ▼
queue full
        │
        ├── reject
        └── accept
```

Umesto:

```text
queue → grows forever
```

sistem može imati:

```text
bounded queue
      ↓
queue full
      ↓
reject / drop / degrade
```

Load shedding može biti neophodan kada je važnije:

```text
protect system
```

nego:

```text
process every request
```

Ovo je posebno važno u distributed systems i high-load servisima.

---

# Pitanje 192

**Kako bi napravio bounded worker system koji podržava cancellation i backpressure?**

### Odgovor

Jedan mogući model:

```text
             Producer
                │
                ▼
        bounded jobs channel
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
       W1      W2      W3
        │       │       │
        └───────┼───────┘
                ▼
             Results
```

Producer:

```go
select {
case jobs <- job:
case <-ctx.Done():
    return ctx.Err()
}
```

Worker:

```go
select {
case <-ctx.Done():
    return ctx.Err()

case job := <-jobs:
    return process(ctx, job)
}
```

Lifecycle:

```text
ctx cancellation
      ↓
producer stops
      ↓
workers stop
      ↓
results stop
      ↓
wait
```

Ključna svojstva:

```text
bounded
cancellation-aware
owned channels
explicit shutdown
```

---

# Pitanje 193

**Koje concurrency primitive bi izabrao za sledeći problem: "Imamo 10.000 poslova, ali najviše 20 sme da se izvršava istovremeno"?**

### Odgovor

Najprirodnije opcije su:

```text
worker pool
```

ili:

```text
semaphore
```

Izbor zavisi od toga da li želimo:

### Worker pool

Ako želimo centralni job queue:

```text
10,000 jobs
     ↓
queue
     ↓
20 workers
```

### Semaphore

Ako caller već ima konkretne taskove:

```text
for each job
    acquire
    execute
    release
```

Ako je workload velik, worker pool često daje bolji lifecycle model jer concurrency i queue imaju jasnog owner-a.

---

# Pitanje 194

**Kako bi procenio da li je concurrency architecture zaista bolja od sekvencijalne implementacije?**

### Odgovor

Ne bih pretpostavio da je concurrent verzija automatski brža.

Merio bih:

```text
throughput
latency
CPU
memory
allocations
GC
lock/channel contention
downstream utilization
error rate
```

Zatim bih uporedio:

```text
sequential baseline
```

sa:

```text
concurrent implementation
```

Posebno bih proverio da li je bottleneck:

```text
CPU
I/O
database
network
serialization
lock contention
external service
```

Ako je bottleneck database sa maksimalno 20 konekcija:

```text
1000 goroutines
```

ne znači:

```text
1000x throughput
```

Može značiti samo:

```text
1000 goroutines
      ↓
20 DB connections
      ↓
queueing
```

Concurrency treba uskladiti sa stvarnim bottleneck-om sistema.

---

# Senior+ mentalni model: channel kao contract

Channel treba posmatrati kroz četiri dimenzije:

```text
             CHANNEL
                │
      ┌─────────┼─────────┐
      │         │         │
   Ownership  Capacity  Lifecycle
      │         │         │
   Who sends?  Buffer   Who closes?
   Who reads?  Backlog   When stops?
```

i petu:

```text
             Semantics
                │
       ┌────────┼────────┐
       │        │        │
    ordering  blocking  cancellation
```

Zreo concurrency dizajn ne završava sa:

```go
ch := make(chan T)
```

nego definiše kompletan contract:

```text
Who creates it?
Who owns it?
Who sends?
Who receives?
Who closes?
What does buffering mean?
What happens when buffer is full?
What happens when receiver disappears?
How does cancellation propagate?
How is completion observed?
What happens on error?
```

Ako ovo nije jasno, channel architecture još nije dovoljno definisana.

---

Na Senior+ nivou pitanja o goroutine-ama i channel-ima prelaze iz domena pojedinačnih language feature-a u domen **dizajna concurrent sistema**.

Nije dovoljno znati da:

```go
go worker()
```

pokreće goroutine, ili da:

```go
select {
case <-ctx.Done():
}
```

omogućava cancellation.

Potrebno je razumeti kompletan lifecycle:

```text
creation
   ↓
coordination
   ↓
work
   ↓
failure / cancellation
   ↓
draining
   ↓
shutdown
```

i obezbediti da svaka goroutine ima jasan razlog zbog kojeg postoji i jasan uslov pod kojim prestaje da postoji.

---

# Pitanje 195

**Šta znači da goroutine ima definisan lifecycle?**

### Odgovor

Goroutine lifecycle treba da ima jasno definisane faze:

```text
start
  ↓
work
  ↓
stop condition
  ↓
cleanup
  ↓
exit
```

Na primer:

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

Ova goroutine ima dva eksplicitna načina završetka:

```text
context cancelled
```

ili:

```text
jobs channel closed
```

To je mnogo sigurnije od goroutine-a koji nema definisan termination path.

---

# Pitanje 196

**Šta je goroutine leak?**

### Odgovor

Goroutine leak nastaje kada goroutine ostane aktivna iako više nema korisnu funkciju, zato što je blokirana ili joj nije omogućen izlazak.

Tipičan primer:

```go
func worker(ch <-chan Job) {
    for {
        job := <-ch
        process(job)
    }
}
```

Ako niko više nikada ne šalje:

```text
worker
  │
  ▼
receive from channel
  │
  ▼
blocked forever
```

Goroutine može ostati živa do kraja procesa.

Problem može biti i indirektan:

```text
producer
   ↓
channel
   ↓
consumer exits
   ↓
producer blocks forever
```

Zato je pitanje koje Senior+ inženjer postavlja:

> **Kako svaka goroutine zna da treba da završi?**

---

# Pitanje 197

**Da li broj goroutine-a sam po sebi predstavlja problem?**

### Odgovor

Ne.

Go je projektovan tako da podrži veliki broj goroutine-a.

Problem nije:

```text
"Imamo mnogo goroutine-a."
```

nego:

```text
"Imamo više goroutine-a nego što workload i architecture opravdavaju."
```

Treba posmatrati:

* goroutine lifecycle,
* memory footprint,
* blocking,
* scheduler overhead,
* workload,
* downstream capacity,
* concurrency limits.

Na primer, 100.000 kratkotrajnih goroutine-a koje brzo završe mogu biti sasvim validne.

Ali 100.000 goroutine-a koje čekaju:

```text
database
channel
network
mutex
timer
```

mogu predstavljati ozbiljan problem.

Broj goroutine-a je metrika za analizu, a ne samostalni dokaz greške.

---

# Pitanje 198

**Kako bi pronašao goroutine leak u production-like okruženju?**

### Odgovor

Prvi korak je posmatranje broja goroutine-a kroz vreme.

Ako:

```text
requests ↑
goroutines ↑
requests ↓
goroutines remain ↑
```

to može ukazivati na leak.

Go omogućava profilisanje goroutine stack-ova preko `pprof`.

Na primer:

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go http.ListenAndServe(":6060", nil)
}
```

Goroutine profile može pokazati gde su goroutine-e blokirane.

Tražimo obrasce poput:

```text
chan receive
chan send
sync.Mutex.Lock
select
IO wait
```

Ključno pitanje nije samo:

```text
"Koliko goroutine-a postoji?"
```

već:

```text
"Zašto ove goroutine-e još uvek postoje?"
```

---

# Pitanje 199

**Kako context cancellation treba da propagira kroz concurrency pipeline?**

### Odgovor

Idealno:

```text
root context
      │
      ▼
producer
      │
      ├─────────┐
      ▼         ▼
   worker 1  worker 2
      │         │
      └────┬────┘
           ▼
        consumer
```

Kada se context otkaže:

```text
ctx.Done()
   │
   ├── producer stops
   ├── worker 1 stops
   ├── worker 2 stops
   └── consumer stops
```

Svaka faza treba da poštuje cancellation.

Na primer:

```go
select {
case job := <-jobs:
    process(job)

case <-ctx.Done():
    return
}
```

Ali cancellation ne treba posmatrati samo kao signal.

Treba definisati i:

```text
šta se dešava sa već primljenim poslom?
šta se dešava sa rezultatima?
da li se queue drainuje?
da li se work rollback-uje?
```

To je pitanje poslovne semantike, a ne samo Go syntax-e.

---

# Pitanje 200

**Koja je razlika između graceful shutdown i immediate shutdown?**

### Odgovor

### Graceful shutdown

Sistem prestaje da prihvata novi posao, ali pokušava da završi postojeći.

```text
stop accepting
      ↓
drain existing work
      ↓
cleanup
      ↓
exit
```

### Immediate shutdown

Sistem pokušava da prekine rad što je pre moguće.

```text
cancel
  ↓
stop workers
  ↓
cleanup
  ↓
exit
```

Na primer, HTTP server može pri shutdown-u:

```text
new requests → rejected
existing requests → allowed to finish
```

dok worker pool može:

```text
stop producer
   ↓
close/drain queue
   ↓
wait workers
```

Izbor zavisi od toga da li je gubitak work-a prihvatljiv.

---

# Pitanje 201

**Kako bi dizajnirao graceful shutdown worker pool-a?**

### Odgovor

Jedan robustan model je:

```text
             producer
                │
                ▼
          jobs channel
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
       W1      W2      W3
        │       │       │
        └───────┼───────┘
                ▼
             results
```

Shutdown:

```text
1. Stop producer
2. Close jobs
3. Workers drain jobs
4. Workers exit
5. Wait for workers
6. Close results
7. Consumer exits
```

Na primer:

```go
close(jobs)
wg.Wait()
close(results)
```

Ali redosled mora biti pažljivo definisan.

Ako zatvorimo `results` prerano:

```text
worker
   ↓
results <- value
   ↓
panic: send on closed channel
```

Zato je owner svakog channel-a ključan.

---

# Pitanje 202

**Zašto `WaitGroup` ne rešava lifecycle problem sam po sebi?**

### Odgovor

`WaitGroup` rešava koordinaciju čekanja:

```text
"Sačekaj da goroutine-e završe."
```

Ali ne govori goroutine-ama:

```text
"Završite."
```

Na primer:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    for {
        work()
    }
}()

wg.Wait()
```

Ovo nikada ne završava ako `work()` nema termination condition.

Potrebno je kombinovati:

```text
cancellation / close / condition
```

sa:

```text
WaitGroup
```

Conceptualno:

```text
signal stop
    ↓
goroutines exit
    ↓
WaitGroup.Wait()
```

a ne:

```text
WaitGroup.Wait()
    ↓
somehow goroutines will stop
```

---

# Pitanje 203

**Koja je razlika između signalizacije završetka i čekanja završetka?**

### Odgovor

To su dva različita problema.

### Signalizacija

```text
"Treba da staneš."
```

Može biti:

```text
context cancellation
channel close
done channel
```

### Čekanje

```text
"Čekam dok ne završiš."
```

Može biti:

```text
WaitGroup
channel receive
```

Na primer:

```go
cancel()

wg.Wait()
```

Prva linija:

```text
cancel()
```

signalizira.

Druga:

```text
wg.Wait()
```

čeka.

Ovo razdvajanje je veoma važno za ispravan lifecycle dizajn.

---

# Pitanje 204

**Da li `context.Context` treba koristiti kao general-purpose data container?**

### Odgovor

Ne.

Context je namenjen za:

* cancellation,
* deadlines,
* request-scoped values.

Ne treba koristiti:

```go
ctx = context.WithValue(ctx, "user", user)
```

kao zamenu za normalan dependency passing ili strukturu podataka.

Context treba da putuje kroz call chain:

```text
request
   ↓
service
   ↓
repository
   ↓
external API
```

kada svaki sloj treba da poštuje:

```text
deadline
cancellation
request-scoped metadata
```

Ali poslovni podaci treba da budu eksplicitni argumenti.

Loš dizajn:

```go
func Process(ctx context.Context) {
    user := ctx.Value("user")
}
```

Bolji:

```go
func Process(ctx context.Context, user User) {
}
```

---

# Pitanje 205

**Kako timeout utiče na concurrent operation?**

### Odgovor

Timeout definiše maksimalni period tokom kojeg je operacija validna.

Na primer:

```go
ctx, cancel := context.WithTimeout(
    parent,
    2*time.Second,
)
defer cancel()
```

Sada downstream operation treba da poštuje:

```go
select {
case result := <-resultCh:
    return result

case <-ctx.Done():
    return ctx.Err()
}
```

Ako downstream ne poštuje context:

```text
caller timeout
      ↓
caller returns
      ↓
downstream goroutine continues
```

što može izazvati leak ili nepotrebnu potrošnju resursa.

Timeout je zato koristan samo ako se propagira kroz celu operaciju.

---

# Pitanje 206

**Šta je cancellation leak?**

### Odgovor

Cancellation leak nastaje kada caller prekine operaciju, ali deo sistema nastavi da radi.

Na primer:

```text
request
   │
   ▼
service
   │
   ▼
goroutine
   │
   ▼
external operation
```

Caller:

```text
cancel
```

ali goroutine ne proverava:

```go
ctx.Done()
```

i nastavlja:

```text
work
  ↓
work
  ↓
work
```

Iako rezultat više nikome nije potreban.

To troši:

* CPU,
* memoriju,
* network,
* connections,
* downstream capacity.

---

# Pitanje 207

**Kako sprečiti goroutine da ostane blokirana na send operaciji?**

### Odgovor

Umesto:

```go
ch <- value
```

koristiti cancellation-aware send:

```go
select {
case ch <- value:
case <-ctx.Done():
    return
}
```

To je naročito važno ako receiver može prestati da postoji.

Model:

```text
producer
   │
   ├── send succeeds
   │
   └── cancellation
          ↓
        exit
```

Bez cancellation case-a:

```text
producer
   ↓
send
   ↓
receiver gone
   ↓
blocked forever
```

---

# Pitanje 208

**Kako sprečiti goroutine da ostane blokirana na receive operaciji?**

### Odgovor

Umesto:

```go
value := <-ch
```

možemo koristiti:

```go
select {
case value := <-ch:
    process(value)

case <-ctx.Done():
    return
}
```

Ako je channel zatvoren:

```go
value, ok := <-ch

if !ok {
    return
}
```

Dakle receive lifecycle treba da pokrije najmanje:

```text
value available
channel closed
context cancelled
```

U zavisnosti od arhitekture, ne moraju sva tri slučaja biti potrebna, ali moraju biti svesno razmotrena.

---

# Pitanje 209

**Kako bi dizajnirao pipeline koji ima više faza i podržava cancellation?**

### Odgovor

Na primer:

```text
Input
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

Svaka faza može biti:

```go
func stage(
    ctx context.Context,
    in <-chan Input,
) <-chan Output
```

Unutar stage-a:

```go
for {
    select {
    case <-ctx.Done():
        return

    case item, ok := <-in:
        if !ok {
            return
        }

        result := transform(item)

        select {
        case out <- result:
        case <-ctx.Done():
            return
        }
    }
}
```

Sada cancellation može propagirati kroz celu pipeline strukturu.

Ključna pravila:

```text
input close → stage finishes
context cancel → stage finishes
output send → cancellation-aware
```

---

# Pitanje 210

**Kako error treba da utiče na ostatak concurrent sistema?**

### Odgovor

Ne postoji univerzalno pravilo.

Postoje najmanje tri modela.

### Fail-fast

Jedna greška prekida ceo operation:

```text
worker 1 → success
worker 2 → ERROR
worker 3 → cancel
worker 4 → cancel
```

Koristan kada rezultat nema smisla bez svih komponenti.

### Best-effort

Greška jednog worker-a ne zaustavlja ostale:

```text
worker 1 → success
worker 2 → ERROR
worker 3 → success
worker 4 → success
```

### Error aggregation

Svi worker-i završe, a greške se prikupe.

Senior+ dizajn treba eksplicitno da definiše error semantics.

---

# Pitanje 211

**Zašto je concurrent error handling teži od sequential error handling-a?**

### Odgovor

Sekvencijalno:

```go
result, err := operation()
if err != nil {
    return err
}
```

Greška prirodno prekida flow.

Kod concurrency-ja može postojati:

```text
W1 → success
W2 → error
W3 → success
W4 → still running
```

Sada treba odlučiti:

```text
Da li W4 treba da se prekine?
Da li čekamo W1/W3?
Da li rezultat W1 ostaje validan?
Da li vraćamo jednu ili više grešaka?
Ko zatvara channels?
```

Zbog toga concurrent error handling mora biti deo architecture design-a, a ne naknadni dodatak.

---

# Pitanje 212

**Šta je fail-fast concurrency model?**

### Odgovor

Fail-fast model znači:

```text
prva relevantna greška
        ↓
cancel shared context
        ↓
ostale goroutine-e prekidaju rad
        ↓
wait
        ↓
return error
```

Conceptualno:

```text
          ┌── W1 → success
          │
cancel ←──┼── W2 → ERROR
          │
          ├── W3 → stop
          │
          └── W4 → stop
```

Ovaj model je veoma koristan za operation-level concurrency gde rezultat nema smisla ako bilo koji critical stage failuje.

---

# Pitanje 213

**Zašto nije dovoljno samo pozvati `cancel()` kada worker prijavi grešku?**

### Odgovor

Zato što ostale goroutine-e moraju biti dizajnirane tako da poštuju cancellation.

Ako worker radi:

```go
for {
    expensiveOperation()
}
```

bez:

```go
select {
case <-ctx.Done():
    return
}
```

onda:

```text
cancel()
```

ne znači automatski:

```text
goroutine stopped
```

Cancellation je cooperative mechanism.

To znači:

```text
signal
   ↓
goroutine observes signal
   ↓
goroutine cleans up
   ↓
goroutine exits
```

---

# Pitanje 214

**Šta znači "structured concurrency" u kontekstu Go-a?**

### Odgovor

Structured concurrency podrazumeva da concurrent tasks imaju jasan parent-child odnos i lifecycle.

Umesto:

```text
function()
  ├── go task()
  ├── go task()
  └── go task()
```

gde child goroutine-e mogu preživeti parent function, želimo:

```text
parent operation
      │
      ├── child 1
      ├── child 2
      └── child 3
      │
      ▼
wait
      │
      ▼
return
```

To znači:

```text
start
  ↓
run
  ↓
cancel if needed
  ↓
wait
  ↓
return
```

Go nema jedan jedini language construct koji garantuje structured concurrency za sve slučajeve, pa se ovaj model gradi kombinacijom:

```text
context
WaitGroup
channels
error propagation
```

i pažljivog API dizajna.

---

# Pitanje 215

**Zašto je "fire-and-forget" goroutine problematičan u production kodu?**

### Odgovor

Pattern:

```go
go sendMetrics()
```

može biti validan za određene pomoćne operacije.

Ali ako je operation critical:

```go
go persistData()
return nil
```

onda caller nema garanciju da je data persistence završena.

Problemi:

```text
caller returns
   ↓
goroutine still running
   ↓
process crashes
```

ili:

```text
request cancelled
   ↓
goroutine continues
```

ili:

```text
shutdown
   ↓
goroutine never awaited
```

Fire-and-forget treba koristiti samo kada je:

```text
failure acceptable
completion irrelevant
lifecycle controlled
```

---

# Pitanje 216

**Kako bi review-ovao concurrent funkciju kao Senior+ inženjer?**

### Odgovor

Ne bih počeo od pitanja:

```text
"Da li radi?"
```

nego od sledećih kategorija.

### Ownership

```text
Ko poseduje state?
Ko poseduje channel?
Ko zatvara channel?
```

### Lifecycle

```text
Kako goroutine završava?
```

### Cancellation

```text
Šta se dešava kada caller otkaže operation?
```

### Error propagation

```text
Kako se greška propagira?
```

### Backpressure

```text
Šta se dešava kada consumer zaostane?
```

### Resource limits

```text
Da li je concurrency bounded?
```

### Shutdown

```text
Da li se sve goroutine-e mogu deterministički ugasiti?
```

### Observability

```text
Kako ćemo otkriti leak, contention ili backlog?
```

Tek nakon toga bih analizirao mikro-optimizacije.

---

# Pitanje 217

**Koji su najčešći anti-pattern-i u Go concurrency kodu?**

### Odgovor

Najčešći su:

### 1. Goroutine bez termination path-a

```go
go func() {
    for {
        work()
    }
}()
```

### 2. Consumer zatvara producer-ov channel

```go
close(ch)
```

sa strane koja ne poseduje send lifecycle.

### 3. Neograničen broj goroutine-a

```go
for _, item := range items {
    go process(item)
}
```

### 4. Send bez cancellation-a

```go
ch <- result
```

kada receiver može nestati.

### 5. Ignorisanje channel close-a

```go
value := <-ch
```

kada je zatvaranje deo contract-a.

### 6. Preveliki buffers

```go
make(chan Item, 1_000_000)
```

kao zamena za rešavanje throughput problema.

### 7. Mutex/channel overengineering

Korišćenje kompleksnog message-passing sistema za jednostavan shared state.

### 8. Fire-and-forget critical work

```go
go persist()
```

bez čekanja.

### 9. Nedeterministički shutdown

Program završava samo zato što `main()` izađe, a ne zato što su goroutine-e pravilno ugašene.

---

# Pitanje 218

**Kako bi objasnio razliku između "concurrency correctness" i "concurrency performance"?**

### Odgovor

Concurrency correctness znači:

```text
Nema data race-a.
Nema deadlock-a.
Nema unsafe lifecycle-a.
Nema izgubljenih rezultata koji nisu očekivani.
Nema send-on-closed-channel panic-a.
```

Concurrency performance znači:

```text
Dovoljna throughput.
Prihvatljiv latency.
Kontrolisana memory usage.
Dobar CPU utilization.
Efikasno korišćenje downstream resursa.
```

Sistem može biti:

```text
correct + slow
```

ili:

```text
fast + incorrect
```

Production kvalitet zahteva oba.

---

# Pitanje 219

**Kako bi dokazao da concurrent sistem nema goroutine leak?**

### Odgovor

Ne bih se oslonio samo na code inspection.

Kombinovao bih:

```text
static reasoning
+
tests
+
timeouts
+
goroutine profiling
+
load testing
```

Test može imati:

```text
start baseline
    ↓
run operation
    ↓
cancel
    ↓
wait
    ↓
verify goroutine count stabilizes
```

Još važnije, testovi treba da proveravaju termination contract.

Na primer:

```go
select {
case <-done:
    // expected
case <-time.After(timeout):
    t.Fatal("goroutine did not terminate")
}
```

Timeout nije dokaz razloga, ali je odličan detector da lifecycle nije zatvoren.

---

# Pitanje 220

**Koje pitanje bi Senior+ kandidat trebalo sebi da postavi pre nego što uvede concurrency?**

### Odgovor

Najvažnije pitanje nije:

```text
"Kako da ovo paralelizujem?"
```

nego:

> **"Da li ovaj problem zaista zahteva concurrency i koji konkretan bottleneck pokušavam da rešim?"**

Zatim:

```text
Šta je jedinica rada?
Šta može da se izvršava nezavisno?
Šta je shared state?
Koji su dependencies?
Koji je concurrency limit?
Šta je backpressure model?
Kako se propagira cancellation?
Kako se propagira error?
Ko zatvara resources?
Kako se sistem gasi?
Kako ćemo meriti da je concurrency poboljšao sistem?
```

Tek nakon toga biramo primitive:

```text
goroutine
channel
mutex
RWMutex
atomic
WaitGroup
context
worker pool
semaphore
```

Senior+ nivo ne znači:

```text
"znam više concurrency API-ja."
```

nego:

```text
"znam da modelujem lifecycle, ownership,
failure i performance concurrent sistema."
```

---

# Senior+ zaključak

Na najvišem nivou Go concurrency znanje prestaje da bude kolekcija syntax pravila.

Potrebno je razumeti sistem kao skup konkurentnih komponenti:

```text
                  ┌───────────────┐
                  │    Context    │
                  └───────┬───────┘
                          │
                cancellation / deadline
                          │
                          ▼
┌──────────┐       ┌──────────────┐       ┌───────────┐
│ Producer │ ───→  │ Bounded Queue│ ───→  │  Workers  │
└──────────┘       └──────────────┘       └─────┬─────┘
                                                │
                                                ▼
                                         ┌────────────┐
                                         │  Results   │
                                         └─────┬──────┘
                                               │
                                               ▼
                                            Consumer
```

Svaka komponenta mora imati definisan:

```text
ownership
lifecycle
capacity
cancellation
error behavior
shutdown behavior
```

A ceo sistem mora imati odgovor na pitanje:

> **Šta se dešava kada bilo koji deo sistema postane spor, otkaže, blokira ili prestane da postoji?**

To je jedna od ključnih razlika između poznavanja Go concurrency API-ja i stvarnog senior-level concurrency engineering-a.

---

Ovaj poslednji deo prelazi sa pojedinačnih concurrency mehanizama na **holističku procenu concurrent sistema**.

Na Senior+ intervjuu kandidat više ne treba samo da objasni kako `goroutine`, `channel`, `mutex`, `context` ili `WaitGroup` funkcionišu.

Potrebno je da pokaže sposobnost da:

* modeluje concurrency problem,
* prepozna rizike,
* izabere odgovarajuću primitivu,
* definiše ownership,
* kontroliše concurrency,
* propagira cancellation,
* upravlja greškama,
* dizajnira graceful shutdown,
* analizira performance,
* razmišlja o observability-ju,
* i obrazloži trade-off između više validnih rešenja.

---

# Pitanje 221

**Dobijaš production servis koji koristi 5000 goroutine-a, 1000 pending jobs i CPU utilization od samo 15%. Da li je to problem?**

### Odgovor

Ne može se zaključiti samo iz tih metrika.

Broj goroutine-a:

```text
5000
```

nije sam po sebi dovoljan dokaz problema.

Potrebno je utvrditi gde goroutine-e provode vreme.

Mogu biti:

```text
I/O waiting
network waiting
database waiting
channel waiting
timer waiting
mutex waiting
```

ili mogu nepotrebno biti aktivne.

Analiza treba da uključi:

```text
goroutine profile
CPU profile
mutex profile
block profile
memory profile
queue depth
request latency
throughput
error rate
```

Na primer:

```text
5000 goroutines
    │
    ├── 4500 waiting for I/O
    ├── 400 waiting on channels
    ├── 50 running
    └── 50 blocked on mutex
```

može biti sasvim drugačiji problem od:

```text
5000 goroutines
    │
    └── repeatedly scheduling useless work
```

Senior+ kandidat prvo prikuplja dokaze, a tek onda zaključuje.

---

# Pitanje 222

**Servis ima 32 worker-a, ali povećanje broja worker-a na 64 ne povećava throughput. Šta bi istraživao?**

### Odgovor

To najčešće znači da worker pool više nije bottleneck.

Mogući bottleneck-i:

```text
database
network
external API
CPU
mutex contention
channel contention
connection pool
disk I/O
rate limiter
```

Na primer:

```text
64 workers
     │
     ▼
DB connection pool = 10
     │
     ▼
10 concurrent queries
```

Povećanje worker-a sa 32 na 64 ne može automatski povećati throughput ako database već predstavlja limiting factor.

Treba meriti:

```text
worker utilization
DB utilization
connection pool wait
CPU utilization
network latency
queue depth
```

Ovo je klasičan primer da:

> **više concurrency-ja ne znači automatski više throughput-a.**

---

# Pitanje 223

**Kako bi odredio optimalan broj worker-a?**

### Odgovor

Ne postoji univerzalna vrednost.

Zavisi od workload-a.

Za CPU-bound workload concurrency se često približava broju raspoloživih CPU resursa, uz dodatne faktore.

Za I/O-bound workload može biti znatno veća:

```text
workers >> CPUs
```

ali samo dok downstream resursi mogu da podrže taj concurrency.

Eksperimentalno bih testirao:

```text
N = 1
N = 2
N = 4
N = 8
N = 16
N = 32
N = 64
...
```

i merio:

```text
throughput
p50 latency
p95 latency
p99 latency
CPU
memory
downstream saturation
error rate
```

Optimalna vrednost je ona koja zadovoljava production requirements, a ne ona koja izgleda lepo kao broj.

---

# Pitanje 224

**Šta je concurrency budget?**

### Odgovor

Concurrency budget je eksplicitno ograničenje koliko istovremenog rada sistem dozvoljava.

Na primer:

```text
HTTP requests:        200
DB queries:            50
external API calls:    20
background workers:    10
```

Ovo sprečava situaciju:

```text
incoming traffic
      ↓
unbounded goroutines
      ↓
unbounded downstream pressure
```

Umesto toga:

```text
incoming traffic
      ↓
bounded concurrency
      ↓
controlled resource usage
```

Concurrency budget može postojati na više nivoa:

```text
request
service
worker pool
database
external API
tenant
```

Ovo je posebno važno u sistemima sa više downstream dependencies.

---

# Pitanje 225

**Kako bi sprečio da jedan tenant iscrpi ceo concurrency budget?**

### Odgovor

Globalni limit:

```text
max concurrency = 100
```

nije uvek dovoljan.

Ako jedan tenant pošalje:

```text
100 concurrent jobs
```

može blokirati sve ostale.

Moguće rešenje je per-tenant limit:

```text
Global: 100

Tenant A: max 20
Tenant B: max 20
Tenant C: max 20
...
```

Model:

```text
              Global Budget
                  100
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
   Tenant A    Tenant B    Tenant C
      20           20          20
```

Ovo je concurrency isolation.

U zavisnosti od sistema mogu se koristiti:

```text
semaphore
worker pools
rate limiting
weighted queues
fair scheduling
```

---

# Pitanje 226

**Šta je starvation u concurrent sistemu?**

### Odgovor

Starvation nastaje kada jedna goroutine ili klasa posla dugo ne dobija potrebne resurse zato što ih druge goroutine-e stalno preuzimaju.

Na primer:

```text
Worker pool
    │
    ├── Tenant A → continuous work
    ├── Tenant A → continuous work
    ├── Tenant A → continuous work
    └── Tenant B → waiting
```

Ako scheduler/queue nema fairness, Tenant B može dugo čekati.

Starvation se razlikuje od deadlock-a.

### Deadlock

Sistem više ne može da napreduje zbog međusobnog čekanja.

### Starvation

Sistem napreduje, ali određena komponenta praktično ne dobija priliku da napreduje.

To je posebno važno kod:

```text
queues
locks
worker pools
resource pools
priority scheduling
```

---

# Pitanje 227

**Šta je priority inversion?**

### Odgovor

Priority inversion nastaje kada high-priority task čeka resurs koji drži low-priority task, dok medium-priority task sprečava low-priority task da završi.

Conceptualno:

```text
High priority
      │
      ▼
   waiting
      │
      ▼
Low priority ── owns resource
      │
      ▼
Medium priority ── consumes execution time
```

High-priority posao ne može da nastavi dok low-priority posao ne oslobodi resource.

U Go aplikacijama ovo se može pojaviti u kompleksnim scheduling/locking sistemima.

Senior+ kandidat treba da razmišlja o:

```text
priority
fairness
resource ownership
lock duration
queue ordering
```

---

# Pitanje 228

**Kako bi dizajnirao sistem koji mora da zaštiti downstream servis od overload-a?**

### Odgovor

Ne bih samo povećao broj worker-a.

Uveo bih kombinaciju:

```text
bounded concurrency
+
bounded queue
+
timeout
+
rate limiting
+
backpressure
+
load shedding
```

Na primer:

```text
                Requests
                    │
                    ▼
              Rate Limiter
                    │
                    ▼
              Bounded Queue
                    │
                    ▼
            Worker Concurrency
                    │
                    ▼
              Downstream API
```

Ako queue dostigne limit:

```text
queue full
   ↓
reject / shed load
```

Ako downstream postane spor:

```text
timeout
   ↓
release worker
```

Cilj nije maksimalno koristiti downstream servis.

Cilj je:

> **Održati sistem stabilnim pod opterećenjem.**

---

# Pitanje 229

**Šta znači "stability under load" u concurrent sistemu?**

### Odgovor

Sistem je stabilan ako povećanje opterećenja ne dovodi do nekontrolisanog rasta:

```text
latency
queue depth
memory
goroutines
errors
connections
```

Na primer:

```text
load ↑
throughput ↑
latency stable
memory stable
queue bounded
```

je poželjan scenario.

Nasuprot tome:

```text
load ↑
queue ↑
latency ↑↑
goroutines ↑↑
memory ↑↑
errors ↑
```

ukazuje na overload cascade.

Senior+ inženjer treba da razmišlja o **stabilnosti sistema**, a ne samo o nominalnom throughput-u.

---

# Pitanje 230

**Šta je cascading failure u concurrent sistemu?**

### Odgovor

Primer:

```text
Service A
   │
   ▼
Service B
   │
   ▼
Database
```

Database postane spor.

Zatim:

```text
DB latency ↑
    ↓
B requests remain active longer
    ↓
B concurrency ↑
    ↓
B connection pool exhausted
    ↓
A requests block
    ↓
A goroutines ↑
    ↓
A memory ↑
```

Jedan bottleneck propagira problem kroz ceo sistem.

To je cascading failure.

Zaštita može uključiti:

```text
timeouts
bounded concurrency
circuit breakers
bulkheads
load shedding
backpressure
resource isolation
```

---

# Pitanje 231

**Šta je bulkhead pattern?**

### Odgovor

Bulkhead izoluje resource budget-e kako failure jedne kategorije workload-a ne bi srušio ceo sistem.

Na primer:

```text
                 Service
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Payments             Reports
      budget=20             budget=5
```

Ako Reports postane spor:

```text
Reports → exhausted
```

Payments i dalje imaju svoj concurrency budget.

U Go-u se ovo može implementirati pomoću:

```text
semaphore
worker pool
bounded queue
per-resource limits
```

Bulkhead je posebno koristan kada više workload-a deli isti proces.

---

# Pitanje 232

**Kako bi objasnio razliku između timeout-a, cancellation-a i deadline-a?**

### Odgovor

### Timeout

Relativno ograničenje:

```text
"Dozvoli najviše 2 sekunde."
```

### Deadline

Apsolutna vremenska granica:

```text
"Operacija mora završiti do 14:32:10."
```

### Cancellation

Signal:

```text
"Operacija više nije potrebna; prekini je."
```

U Go-u sva tri mogu biti predstavljena kroz `context.Context`.

Važno je da downstream operacije poštuju context.

---

# Pitanje 233

**Da li cancellation garantuje da će operation odmah stati?**

### Odgovor

Ne.

Context cancellation je cooperative.

Ako goroutine radi:

```go id="9f4r2x"
for {
    compute()
}
```

i nikada ne proverava context:

```text id="2ovx4n"
cancel()
   ↓
goroutine continues
```

Ako operation može dugo da traje, mora imati cancellation point ili koristiti API koji podržava context.

Na primer:

```go id="2tqk2x"
select {
case <-ctx.Done():
    return ctx.Err()
default:
}
```

ili context-aware I/O API.

Dakle:

```text id="c8o6mb"
cancel ≠ forced thread termination
```

---

# Pitanje 234

**Kako bi objasnio trade-off između graceful shutdown-a i data durability-ja?**

### Odgovor

Pretpostavimo da worker obrađuje:

```text
payment
```

Ako shutdown odmah prekine worker:

```text
payment processing
       ↓
process killed
```

možemo izgubiti posao.

Ako čekamo beskonačno:

```text
graceful shutdown
       ↓
worker never finishes
       ↓
deployment blocked
```

Potrebno je definisati:

```text
shutdown deadline
```

i recovery model.

Na primer:

```text
stop accepting new jobs
      ↓
finish critical jobs
      ↓
deadline reached
      ↓
persist/requeue unfinished jobs
      ↓
exit
```

To je već distributed-systems problem, ne samo Go concurrency problem.

---

# Pitanje 235

**Kako bi dizajnirao durable worker queue?**

### Odgovor

Ako gubitak in-memory channel sadržaja nije prihvatljiv, običan:

```go
chan Job
```

nije dovoljan kao durable queue.

In-memory channel:

```text
process memory
     ↓
crash
     ↓
queue lost
```

Za durability je potreban persistent storage:

```text
producer
   ↓
durable queue
   ↓
workers
```

Na primer, arhitektura može koristiti:

```text
database
message broker
persistent log
```

Channel onda može služiti kao lokalni concurrency primitive između komponenti procesa, ali ne kao jedini source of truth.

---

# Pitanje 236

**Zašto channel nije zamena za message broker?**

### Odgovor

Channel je in-process primitive.

Njegov scope je:

```text
jedan Go proces
```

Message broker može obezbediti:

```text
durability
retries
consumer groups
delivery semantics
replay
cross-process communication
scaling
```

Channel:

```text
G1 → channel → G2
```

Broker:

```text
Service A
    │
    ▼
Message Broker
    │
    ├── Service B
    ├── Service C
    └── Service D
```

Channel i broker rešavaju različite probleme.

---

# Pitanje 237

**Kako bi dizajnirao retry sistem za concurrent operation?**

### Odgovor

Retry nije samo:

```go id="c2c6w5"
for i := 0; i < 3; i++ {
    err = operation()
}
```

Potrebno je razmotriti:

```text
idempotency
backoff
jitter
maximum attempts
deadline
cancellation
error classification
downstream load
```

Na primer:

```text
attempt 1
   ↓
failure
   ↓
backoff
   ↓
attempt 2
   ↓
failure
   ↓
backoff
   ↓
attempt 3
```

Ako hiljade goroutine-a istovremeno retry-uju nakon istog failure-a, mogu napraviti:

```text
retry storm
```

Zato retry mora biti usklađen sa concurrency i rate limiting modelom.

---

# Pitanje 238

**Šta je retry storm?**

### Odgovor

Retry storm nastaje kada veliki broj klijenata istovremeno ponavlja neuspele operacije.

Na primer:

```text
1000 requests
     ↓
downstream fails
     ↓
1000 retries
     ↓
downstream still overloaded
     ↓
1000 more retries
```

Sistem sam pojačava overload.

Zaštita uključuje:

```text
exponential backoff
jitter
retry budgets
timeouts
circuit breakers
rate limits
```

Senior+ kandidat treba da prepozna da retry može povećati problem koji pokušava da reši.

---

# Pitanje 239

**Kako concurrency utiče na idempotency?**

### Odgovor

Ako isti posao može biti izvršen više puta:

```text
Job A
   ├── attempt 1
   └── attempt 2
```

onda operation mora biti bezbedna za duplicate execution ili mora imati deduplication mechanism.

Na primer:

```text
charge customer
```

nije isto što i:

```text
calculate hash
```

Kod payment-like operacija potreban je idempotency key:

```text
request ID
     ↓
deduplication
     ↓
operation
```

Concurrency povećava mogućnost da isti posao bude obrađen:

```text
concurrently
retries
duplicate delivery
timeout + late completion
```

Zato concurrency design i business correctness ne mogu biti potpuno odvojeni.

---

# Pitanje 240

**Kako bi odgovorio na pitanje: "Channels or mutexes — šta je bolje?"**

### Odgovor

Odgovor:

> Ni jedno nije univerzalno bolje.

Treba prvo identifikovati problem.

Ako imamo:

```text
shared mutable state
```

mutex je često prirodan izbor.

Ako imamo:

```text
pipeline
producer-consumer
work distribution
ownership transfer
```

channel može biti prirodniji.

Primer:

```text
shared counter
    ↓
Mutex
```

Nasuprot:

```text
producer
    ↓
channel
    ↓
worker
```

Pravi odgovor na intervjuu nije:

```text
"Channels su Go način."
```

nego:

```text
"Biramo primitive prema synchronization semantics problem-a."
```

---

# Pitanje 241

**Kako bi objasnio razliku između lock-based i message-passing arhitekture?**

### Odgovor

### Lock-based

Više goroutine-a pristupa zajedničkom state-u:

```text
        shared state
       /     |     \
      G1    G2     G3
       \     |     /
        Mutex
```

### Message-passing

Jedna komponenta poseduje state, a ostale komuniciraju sa njom:

```text
G1 ──┐
G2 ──┼──→ channel → owner → state
G3 ──┘
```

Prvi model često ima:

```text
shared memory
mutual exclusion
```

Drugi:

```text
ownership
message passing
```

Ni jedan nije univerzalno superioran.

---

# Pitanje 242

**Kako bi odlučio da li concurrency complexity opravdava benefit?**

### Odgovor

Procena treba da uključi:

```text
performance gain
implementation complexity
failure modes
testing complexity
operational complexity
maintenance cost
```

Ako concurrent implementation daje:

```text
+2% throughput
```

ali uvodi:

```text
3 channels
5 goroutines
complex shutdown
multiple cancellation paths
```

verovatno nije vredna.

Ako daje:

```text
10x throughput
```

uz kontrolisan lifecycle, verovatno jeste.

Senior+ inženjer optimizuje:

```text
system value
```

a ne:

```text
goroutine count
```

---

# Pitanje 243

**Kako bi testirao concurrent architecture deterministički?**

### Odgovor

Concurrency testovi često pate od:

```text
timing dependence
race conditions
flaky scheduling
```

Loš test:

```go id="4m6y6f"
time.Sleep(100 * time.Millisecond)

if result != expected {
    t.Fatal(...)
}
```

Bolje je koristiti explicit synchronization:

```text
done channel
WaitGroup
condition
controlled test hooks
```

Na primer:

```go id="x9ktcd"
done := make(chan struct{})

go func() {
    defer close(done)
    work()
}()

select {
case <-done:
    // finished
case <-time.After(time.Second):
    t.Fatal("timeout")
}
```

Test treba da čeka na **event**, a ne na proizvoljan vremenski period.

---

# Pitanje 244

**Kako race detector pomaže, a šta ne može da dokaže?**

### Odgovor

`go test -race` može otkriti određene data race situacije tokom izvršavanja testova.

Ali:

```text
race detector
```

ne dokazuje:

```text
"program nema nijedan mogući concurrency bug."
```

Može otkriti samo race koji je zapravo manifestovan tokom execution-a.

Ne rešava automatski:

```text
deadlock
goroutine leak
wrong ownership
incorrect cancellation
logical race
incorrect shutdown
```

Zato race detector treba kombinovati sa:

```text
tests
load tests
profiling
static reasoning
code review
```

---

# Pitanje 245

**Koja je razlika između data race-a i logical race-a?**

### Odgovor

Data race je problem konkurentnog pristupa memoriji bez odgovarajuće sinhronizacije.

Logical race može postojati i kada je svaki pristup memoriji tehnički sinhronizovan.

Na primer:

```text
G1:
check balance

G2:
check balance

G1:
withdraw

G2:
withdraw
```

Svaka operacija može biti pojedinačno zaštićena mutex-om, ali kompletna business transaction može biti pogrešno modelovana.

Dakle:

```text
data race
```

je memory synchronization problem.

```text
logical race
```

je correctness problem u interleaving-u događaja.

Senior+ inženjer mora razumeti oba.

---

# Pitanje 246

**Kako bi objasnio deadlock na arhitektonskom nivou?**

### Odgovor

Deadlock nije samo:

```text
Mutex A → Mutex B
Mutex B → Mutex A
```

Može nastati kroz različite primitive.

Na primer:

```text
G1
 │
 ▼
send ch1
 │
 ▼
wait ch2

G2
 │
 ▼
send ch2
 │
 ▼
wait ch1
```

Obe goroutine-e čekaju jedna drugu.

Formalno, deadlock nastaje kada postoji ciklus zavisnosti:

```text
G1 waits for G2
G2 waits for G1
```

ili kompleksniji:

```text
G1 → G2 → G3 → G1
```

Senior+ analiza treba da traži **dependency cycles**, a ne samo mutex locking.

---

# Pitanje 247

**Kako bi dizajnirao concurrency architecture za sistem sa CPU-bound i I/O-bound workload-ima?**

### Odgovor

Ne bih ih automatski stavio u isti worker pool.

Na primer:

```text
             Incoming work
                   │
          ┌────────┴────────┐
          ▼                 ▼
      CPU-bound          I/O-bound
       workers             workers
          │                 │
      bounded            bounded
      by CPU           by downstream
```

CPU-bound concurrency treba uskladiti sa CPU kapacitetom.

I/O-bound concurrency treba uskladiti sa:

```text
connection pools
remote API limits
latency
memory
```

Odvajanje workload-a sprečava da jedan tip posla iscrpi resurse potrebne drugom.

---

# Pitanje 248

**Kako bi procenio concurrency architecture nakon implementacije?**

### Odgovor

Pratio bih najmanje:

```text
Throughput
Latency p50
Latency p95
Latency p99
Error rate
Queue depth
Queue wait time
Active workers
Goroutine count
CPU
Memory
GC
Mutex contention
Channel blocking
Downstream saturation
```

I posmatrao bih ponašanje kroz različita opterećenja:

```text
10%
25%
50%
75%
100%
125%
150%
200%
```

Posebno je važan overload region.

Nije dovoljno znati:

```text
"Radi dobro na 50% load-a."
```

Potrebno je znati:

```text
"Kako sistem degradira kada pređe kapacitet?"
```

---

# Pitanje 249

**Šta očekuješ od Senior+ kandidata kada dobije nepoznat concurrency problem?**

### Odgovor

Ne očekuje se da kandidat odmah napiše kod.

Očekuje se da prvo postavi pitanja.

Na primer:

```text
1. Koji je workload?
2. Da li je CPU ili I/O bound?
3. Koji je očekivani throughput?
4. Koji je latency requirement?
5. Da li je ordering potreban?
6. Da li je work idempotentan?
7. Šta se dešava ako worker failuje?
8. Šta se dešava ako consumer nestane?
9. Da li je work durable?
10. Koliki concurrency downstream podržava?
11. Da li moramo podržati graceful shutdown?
12. Da li smemo odbaciti work?
13. Kako propagiramo cancellation?
14. Kako merimo overload?
15. Kako testiramo failure scenarios?
```

Tek nakon razumevanja requirements-a bira se implementation.

---

# Pitanje 250

**Koji bi bio tvoj kompletan mentalni model za dizajn Go concurrent sistema?**

### Odgovor

Senior+ kandidat treba da razmišlja kroz sledeći redosled.

### 1. Workload

```text
Šta se izvršava?
CPU?
I/O?
Mixed?
```

### 2. Dependencies

```text
Šta zavisi od čega?
```

### 3. Ownership

```text
Ko poseduje state?
```

### 4. Concurrency

```text
Šta može paralelno?
Šta mora sekvencijalno?
```

### 5. Capacity

```text
Koliko rada sme biti aktivno?
```

### 6. Backpressure

```text
Šta se dešava kada downstream zaostane?
```

### 7. Failure

```text
Šta ako komponenta failuje?
```

### 8. Cancellation

```text
Kako se prekida work?
```

### 9. Shutdown

```text
Kako se sistem gasi?
```

### 10. Durability

```text
Da li sme work biti izgubljen?
```

### 11. Observability

```text
Kako znamo da sistem radi dobro?
```

### 12. Testing

```text
Kako dokazujemo correctness pod različitim interleavings i load-ovima?
```

### 13. Performance

```text
Da li concurrency zaista donosi benefit?
```

---

# Senior+ scenario: projektovanje production worker sistema

Pretpostavimo sledeći zahtev:

```text
Imamo API koji prima 100.000 poslova.
Svaki posao poziva eksterni HTTP servis.
Eksterni servis dozvoljava maksimalno 50 concurrent requests.
Posao traje između 50 ms i 5 s.
Korisnik može otkazati operation.
Sistem mora podržati graceful shutdown.
Neuspešni poslovi treba da budu retry-ovani.
```

Senior+ kandidat ne bi počeo ovako:

```go id="yqtb2b"
for _, job := range jobs {
    go process(job)
}
```

Umesto toga, model bi mogao izgledati:

```text id="xmj0qm"
                         API
                          │
                          ▼
                   bounded admission
                          │
                          ▼
                    Job Queue
                          │
                          ▼
                  Worker Pool / Limit
                          │
                   max 50 concurrent
                          │
                          ▼
                 External HTTP API
                          │
                    ┌─────┴─────┐
                    │           │
                    ▼           ▼
                  success     failure
                                │
                                ▼
                              retry
                                │
                                ▼
                         backoff + jitter
```

Uz:

```text id="yl53y5"
context cancellation
timeouts
bounded concurrency
bounded queue
retry budget
graceful shutdown
metrics
tracing
```

Ako external service postane spor:

```text
latency ↑
    ↓
workers remain occupied longer
    ↓
queue wait ↑
    ↓
latency ↑
```

Zato timeout mora sprečiti beskonačno zauzimanje worker-a.

Ako external service potpuno padne:

```text
requests fail
    ↓
retry
    ↓
backoff
    ↓
retry budget
```

a ne:

```text
requests fail
    ↓
100.000 immediate retries
```

Ako korisnik otkaže operation:

```text
context cancellation
       ↓
queue admission stops
       ↓
workers cancel
       ↓
HTTP requests cancel where supported
       ↓
resources released
```

Ako proces dobije shutdown signal:

```text
stop accepting new work
        ↓
stop producers
        ↓
drain or persist queued work
        ↓
wait workers
        ↓
cleanup
        ↓
exit
```

Ovo je razlika između:

```text
"znam Go concurrency"
```

i:

```text
"znam da dizajniram production concurrency system."
```

---

# Senior+ završna evaluacija

Kandidat na ovom nivou treba da bude sposoban da objasni ne samo **kako** nešto radi, već i:

```text
ZAŠTO
KADA
KADA NE
KOJI JE TRADE-OFF
KAKO SE TESTIRA
KAKO SE POSMATRA
KAKO SE GASI
KAKO SE OPORAVLJA OD GREŠKE
```

Posebno treba da ume da poveže:

```text
goroutines
    ↓
channels
    ↓
synchronization
    ↓
ownership
    ↓
backpressure
    ↓
bounded concurrency
    ↓
cancellation
    ↓
error propagation
    ↓
graceful shutdown
    ↓
observability
    ↓
performance
    ↓
production reliability
```

Najvažnija Senior+ sposobnost nije poznavanje najvećeg broja concurrency API-ja.

To je sposobnost da se **konkurentni sistem modeluje kao skup jasno definisanih lifecycle-a, ownership odnosa, resource budget-a i failure scenarija**.

---

# Kriterijumi za Senior+ nivo

Kandidat je blizu Senior+ nivoa ako bez značajne pomoći može da:

* dizajnira bounded worker pool,
* objasni channel ownership,
* spreči goroutine leak,
* dizajnira cancellation propagation,
* implementira graceful shutdown,
* razlikuje backpressure od load shedding-a,
* definiše concurrency budget,
* prepozna starvation i deadlock,
* dizajnira retry sa backoff-om i jitter-om,
* razume idempotency u concurrent sistemima,
* izoluje workload-e pomoću bulkhead principa,
* analizira throughput i latency,
* koristi profiling za concurrency probleme,
* razume razliku između data race-a i logical race-a,
* dizajnira testove bez oslanjanja na proizvoljne `Sleep` pozive,
* prepozna kada channel nije pravi primitive,
* objasni kada je mutex jednostavniji,
* dizajnira failure propagation,
* razume graceful degradation,
* i obrazloži concurrency trade-off na arhitektonskom nivou.

---

# Završna poruka

Go concurrency nije samo:

```go
go func() {}
```

i:

```go
select {}
```

Ozbiljan concurrency engineering podrazumeva upravljanje:

```text
                 CONCURRENCY
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
    Ownership      Lifecycle      Capacity
       │              │              │
       ▼              ▼              ▼
    Channels       Context        Backpressure
    Mutexes        Shutdown       Limits
    Atomics        Errors         Queues
       │              │              │
       └──────────────┼──────────────┘
                      ▼
                Reliability
                      │
                      ▼
                 Production
```

Kada kandidat može da sagleda sve ove dimenzije zajedno i da donese obrazloženu odluku između više validnih arhitektura, tada više ne govori samo o Go concurrency mehanizmima — već pokazuje **Senior+ nivo concurrency engineering-a**.