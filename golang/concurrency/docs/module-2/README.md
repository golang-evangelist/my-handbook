# Module 2 — Coordination & Concurrency Patterns

Module 2 predstavlja prirodan nastavak Module 1.

U Module 1 naučili smo kako:

* pokrenuti goroutine;
* komunicirati preko channel-a;
* razlikovati unbuffered i buffered channels;
* koristiti directional channels;
* iterirati preko channel-a;
* zatvoriti channel;
* definisati ownership i lifecycle;
* prepoznati osnovne blocking, deadlock i goroutine-leak probleme.

Module 2 prelazi sa pitanja:

> **Kako goroutine-i komuniciraju?**

na mnogo važnije pitanje:

> **Kako koordinisati veći broj concurrent aktivnosti tako da sistem bude korektan, kontrolisan i održiv?**

---

# Module 2 at a Glance

Struktura modula u repozitorijumu je:

```text
module-2/
│
├── README.md
├── 01-select.md
├── 02-select-default.md
├── 03-waitgroup.md
├── 04-worker-pools.md
├── 05-pipelines.md
├── 06-fan-out.md
├── 07-fan-in.md
└── 08-module-2-summary-and-exercises.md
```

Konceptualni progression:

```text
                 Module 2
                     │
                     ▼
                 select
                     │
                     ▼
              select + default
                     │
                     ▼
                 WaitGroup
                     │
                     ▼
                Worker Pool
                     │
                     ▼
                  Pipeline
                     │
              ┌──────┴──────┐
              ▼             ▼
           Fan-Out        Fan-In
              │             │
              └──────┬──────┘
                     ▼
              Coordinated
              Concurrency
```

---

# Module Goal

Cilj Module 2 je da čitalac nauči da od osnovnih concurrency primitives napravi **composable concurrency systems**.

Na kraju modula trebalo bi da bude moguće dizajnirati sistem kao:

```text
                         Input
                           │
                           ▼
                    ┌─────────────┐
                    │ Dispatcher  │
                    └──────┬──────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Worker A      Worker B      Worker C
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                       Results
                           │
                           ▼
                       Consumer
```

i da se pritom precizno zna:

* ko pokreće koju goroutine;
* ko šalje;
* ko prima;
* ko čeka;
* ko zatvara channel;
* kada se sistem završava;
* kako se sprečava prerano završavanje;
* kako se sprečava goroutine leak;
* kako se kontroliše concurrency level.

---

# Learning Objectives

Po završetku Module 2 čitalac treba da bude sposoban da:

## `select`

* koristi `select` za čekanje na više channel operations;
* razume kada je case ready;
* razume kako `select` bira između više spremnih case-ova;
* koristi `select` za koordinaciju više concurrent događaja;
* razume blocking behavior `select` statement-a.

## `select` + `default`

* razume razliku između blocking i non-blocking select-a;
* koristi `default` za non-blocking channel operations;
* prepozna kada `default` može dovesti do busy-loop-a;
* razume kada je polling opravdan, a kada nije.

## `WaitGroup`

* koristi `sync.WaitGroup`;
* razume relationship između `Add`, `Done` i `Wait`;
* čeka završetak grupe goroutine-a;
* izbegava tipične lifecycle greške;
* razume zašto `WaitGroup` nije zamena za channel communication.

## Worker Pools

* implementira bounded worker pool;
* razume producer/worker/result arhitekturu;
* kontroliše broj aktivnih worker-a;
* koristi channel kao work queue;
* razume backpressure;
* pravilno završava worker pool.

## Pipelines

* dizajnira višestepeni pipeline;
* povezuje stages preko channels;
* razume lifecycle svakog stage-a;
* propagira completion;
* identifikuje potencijalne goroutine leaks.

## Fan-Out

* distribuira posao na više worker-a;
* razume konkurentnu obradu;
* razlikuje fan-out od worker-pool modela;
* razume ordering implications.

## Fan-In

* kombinuje više input channels;
* objedini rezultate u jedan output channel;
* koordinira završetak više producer-a;
* razume kako `WaitGroup` i channel closing mogu raditi zajedno.

---

# Module 2 Mental Model

Module 1 je imao osnovni model:

```text
Goroutine
    │
    ▼
Channel
    │
    ▼
Goroutine
```

Module 2 uvodi multiplicitet:

```text
                 ┌──► Goroutine A ──┐
                 │                  │
Channel ─────────┼──► Goroutine B ──┼──► Channel
                 │                  │
                 └──► Goroutine C ──┘
```

Sada više nije dovoljno znati samo:

```text
send
receive
close
```

Potrebno je znati:

```text
select
coordination
fan-out
fan-in
bounded concurrency
completion
```

---

# 1. `select`

Prva lekcija:

```text
01-select.md
```

uvodi `select`, jedan od centralnih mehanizama Go concurrency modela.

Osnovni oblik:

```go
select {
case value := <-ch1:
    process(value)

case value := <-ch2:
    process(value)
}
```

`select` omogućava goroutine-i da čeka na više communication operations.

Mentalni model:

```text
                   ┌──► receive ch1
                   │
select ────────────┼──► receive ch2
                   │
                   └──► send ch3
```

Umesto da goroutine bude vezana za jednu operaciju:

```go
value := <-ch
```

može istovremeno da čeka više mogućnosti.

---

# Why `select` Matters

Bez `select`:

```text
Goroutine
    │
    ▼
wait for ch1
```

Ako `ch1` nikada ne dobije vrednost, goroutine čeka.

Sa `select`:

```text
             ┌── ch1
             │
Goroutine ───┼── ch2
             │
             └── ch3
```

goroutine može reagovati na prvi relevantan događaj koji postane spreman.

Ovo je osnova za:

* multiplexing;
* cancellation;
* timeouts;
* event loops;
* multiple input sources;
* coordination.

---

# `select` as Multiplexer

`select` se može posmatrati kao concurrency multiplexer:

```text
Input A ──┐
          │
Input B ──┼──► select ──► Handler
          │
Input C ──┘
```

Umesto da ručno proveravamo svaki channel:

```text
check A
check B
check C
```

`select` omogućava runtime-u da čeka relevantne operations.

---

# Ready Cases

Važno je razumeti termin **ready**.

Case je ready kada njegova channel operation može da se izvrši bez dodatnog blocking-a.

Na primer, ako:

```go
ch := make(chan int, 1)
ch <- 42
```

onda je:

```go
case value := <-ch:
```

ready.

Ako je drugi channel takođe spreman:

```text
ch1 → ready
ch2 → ready
```

`select` može izabrati bilo koji od spremnih case-ova.

Ne treba pretpostavljati da će uvek biti izabran prvi case.

---

# No Ready Case

Ako nijedan case nije ready:

```go
select {
case <-ch1:
case <-ch2:
}
```

goroutine blokira dok jedan od case-ova ne postane ready.

Mentalni model:

```text
ch1 ── not ready
ch2 ── not ready
          │
          ▼
       BLOCK
          │
          ▼
   one becomes ready
          │
          ▼
      continue
```

Ovo je fundamentalna razlika između običnog `select`-a i `select`-a sa `default`.

---

# Multiple Ready Cases

Ako je više case-ova spremno:

```text
ch1 ── ready
ch2 ── ready
ch3 ── not ready
```

`select` bira jedan od ready case-ova.

Zbog toga kod ne treba da zavisi od pretpostavke:

```text
"ch1 always wins because it is first."
```

To nije validan concurrency design assumption.

---

# `select` and Channel Directions

Directional channels se prirodno kombinuju sa `select`.

Na primer:

```go
func consume(
    data <-chan int,
    done <-chan struct{},
) {
    select {
    case value := <-data:
        process(value)

    case <-done:
        return
    }
}
```

API jasno pokazuje da funkcija:

* čita data;
* čita shutdown signal;
* ne šalje na te channels.

---

# `select` and Lifecycle

Jedna od najvažnijih upotreba `select`-a jeste lifecycle control.

Na primer:

```go
select {
case value := <-jobs:
    process(value)

case <-done:
    return
}
```

Worker sada može istovremeno:

```text
receive work
       OR
receive shutdown
```

To je važan korak dalje od jednostavnog:

```go
value := <-jobs
```

koji nema ugrađen način da reaguje na drugi događaj.

---

# `select` and Cancellation Preview

Kasnije ćemo koristiti:

```go
select {
case <-ctx.Done():
    return

case value := <-jobs:
    process(value)
}
```

Ovaj obrazac je jedan od najvažnijih idiomatskih concurrency patterns u Go-u.

Module 2 ga uvodi kroz `select`, dok će cancellation i `context` biti obrađeni dublje u kasnijim delovima curriculum-a.

---

# 2. `select` and Send Operations

`select` ne služi samo za receive.

Može čekati i send:

```go
select {
case ch <- value:
    // send succeeded

case <-done:
    return
}
```

Ovo omogućava producer-u da kaže:

```text
send value
    OR
stop
```

Mentalni model:

```text
                ┌──► send value
Producer ───────┤
                └──► shutdown
```

Ovo je veoma važno za izbegavanje producer goroutine leak-a.

---

# `select` With Multiple Communication Directions

Jedan `select` može kombinovati:

```text
receive
receive
send
receive
```

Na primer:

```go
select {
case value := <-input:
    process(value)

case resultCh <- result:
    result = nil

case <-done:
    return
}
```

Ovo već liči na event-driven concurrency component.

---

# `select` Is Not a General Synchronization Primitive

`select` omogućava izbor između channel operations.

Ne treba ga posmatrati kao univerzalnu zamenu za:

* mutex;
* `WaitGroup`;
* atomic operations;
* `context`.

Svaki primitive ima drugačiju semantiku.

`select` je prvenstveno:

```text
channel operation coordination
```

---

# 3. `select` Fairness and Scheduling

Kada više case-ova može da se izvrši, ne treba pisati logiku koja zavisi od njihovog relativnog položaja.

Na primer:

```go
select {
case <-highPriority:
    handleHigh()

case <-lowPriority:
    handleLow()
}
```

Ne treba automatski zaključiti:

```text
highPriority always wins
```

Samo zato što se nalazi iznad `lowPriority`.

Ako je sistemu potrebna stroga prioritizacija, mora se koristiti eksplicitniji design.

---

# Priority Is a Design Problem

Ako postoji zahtev:

```text
high priority work must be handled first
```

običan:

```go
select {
case high := <-high:
case low := <-low:
}
```

nije dovoljan kao formalna priority guarantee.

Moguće je implementirati priority-aware logic, ali to treba biti eksplicitno dizajnirano.

Ovo je važan concurrency design principle:

> Ne pretpostavljaj scheduling guarantee tamo gde API definiše samo concurrency semantics.

---

# 4. `select` and Closed Channels

Closed channel može učiniti receive case ready odmah.

Na primer:

```go
close(done)
```

nakon toga:

```go
case <-done:
```

može odmah biti izabran.

Zbog toga je closed channel veoma pogodan za broadcast-style shutdown signals.

Mentalni model:

```text
             close(done)
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
       G1        G2        G3
        │         │         │
        ▼         ▼         ▼
      stop      stop      stop
```

---

# Closed Channel as a Permanent Signal

Za razliku od slanja jedne vrednosti:

```go
done <- struct{}{}
```

close:

```go
close(done)
```

može omogućiti da više receiver-a detektuje isti događaj.

To je ključna razlika:

```text
send one value
```

nasuprot:

```text
close channel
```

koji signalizira stanje:

```text
"this event has happened"
```

---

# State vs Event

Ovo je koristan način razmišljanja.

Slanje vrednosti često predstavlja:

```text
EVENT
```

dok zatvoren signalni channel može predstavljati:

```text
STATE:
the event has happened
```

Na primer:

```text
done <- value
```

može signalizirati jedan occurrence.

Ali:

```text
close(done)
```

omogućava svakom zainteresovanom receiver-u da vidi da je stanje prešlo u:

```text
done
```

---

# 5. `select` and Time

`select` može čekati i na time-based channel operations.

Na primer:

```go
select {
case value := <-ch:
    process(value)

case <-time.After(time.Second):
    handleTimeout()
}
```

Konceptualno:

```text
                 ┌──► data available
Goroutine ───────┤
                 └──► timeout elapsed
```

Time-based coordination će biti detaljnije obrađena u okviru cancellation/timeout tema, ali `select` je mehanizam koji ih povezuje sa channel modelom.

---

# Timeout Semantics

Timeout treba razumeti kao:

```text
wait for operation
OR
stop waiting after deadline
```

To je fundamentalno drugačije od:

```text
sleep and hope operation finished
```

`select` omogućava upravo prvi model.

---

# 6. `select` and Non-Blocking Operations

Druga lekcija:

```text
02-select-default.md
```

uvodi `default`.

Primer:

```go
select {
case value := <-ch:
    process(value)

default:
    // nothing available
}
```

Ako receive nije ready, `default` se izvršava odmah.

Za razliku od:

```go
select {
case value := <-ch:
    process(value)
}
```

koji bi blokirao.

---

# Blocking vs Non-Blocking Select

Bez `default`:

```text
no case ready
      │
      ▼
    BLOCK
```

Sa `default`:

```text
no case ready
      │
      ▼
  default
      │
      ▼
 continue immediately
```

To je osnovni non-blocking channel operation pattern.

---

# Non-Blocking Receive

Primer:

```go
select {
case value := <-ch:
    fmt.Println(value)

default:
    fmt.Println("no value available")
}
```

Ovaj kod ne čeka.

On odgovara na pitanje:

> "Da li je vrednost trenutno dostupna?"

a ne:

> "Sačekaj dok vrednost ne postane dostupna."

Ta razlika je ključna.

---

# Non-Blocking Send

Isti model važi za send:

```go
select {
case ch <- value:
    fmt.Println("sent")

default:
    fmt.Println("channel not ready")
}
```

Ako channel trenutno ne može da prihvati vrednost, `default` se izvršava.

Kod buffered channel-a to znači da buffer može biti pun.

Kod unbuffered channel-a znači da receiver trenutno nije spreman.

---

# Non-Blocking Does Not Mean Asynchronous

Važno:

```text
non-blocking
```

ne znači:

```text
asynchronous
```

Non-blocking select samo znači:

> Operacija neće čekati ako trenutno nije ready.

To ne znači da će se operacija automatski izvršiti kasnije.

Ako case nije ready:

```text
operation skipped
```

a `default` se izvršava.

---

# 7. Polling

`default` može biti koristan za polling:

```go
for {
    select {
    case value := <-ch:
        process(value)

    default:
        doOtherWork()
    }
}
```

Ali ovaj pattern treba koristiti pažljivo.

Ako `doOtherWork()` ne blokira i loop se brzo izvršava:

```text
iteration
iteration
iteration
iteration
iteration
...
```

može doći do visokog CPU usage-a.

---

# Busy Loop Anti-Pattern

Loš obrazac:

```go
for {
    select {
    case value := <-ch:
        process(value)

    default:
    }
}
```

Ako nema vrednosti:

```text
default
  ↓
loop
  ↓
default
  ↓
loop
  ↓
default
```

goroutine može potrošiti CPU bez korisnog rada.

To je **busy waiting**.

---

# Avoiding Busy Waiting

Ako nije potrebno aktivno polling ponašanje, bolje je koristiti blocking:

```go
select {
case value := <-ch:
    process(value)

case <-done:
    return
}
```

ili odgovarajući synchronization primitive.

Princip:

> Ako možeš da čekaš na događaj, nemoj neprekidno proveravati da li se događaj dogodio.

---

# `default` as Opportunistic Work

Postoji legitimna upotreba:

```go
select {
case value := <-ch:
    process(value)

default:
    performFallback()
}
```

Ovo znači:

```text
if work is available:
    do work
else:
    do something else
```

To može biti validan non-blocking design.

Ali treba jasno znati da se work neće čekati.

---

# 8. `select` State Machine

`select` se može posmatrati kao malu state machine.

Na primer:

```text
                 START
                   │
                   ▼
              WAITING
              /   |   \
             /    |    \
            ▼     ▼     ▼
         DATA   TIMEOUT STOP
            │     │      │
            ▼     ▼      ▼
         PROCESS HANDLE  EXIT
```

Ovaj mentalni model postaje izuzetno koristan kada se concurrency component ponaša kao event-driven worker.

---

# Worker State Machine

Worker može imati:

```text
RUNNING
   │
   ├── job available ──► PROCESSING
   │                         │
   │                         ▼
   │                       RUNNING
   │
   └── shutdown ─────────► STOPPED
```

`select` može biti mehanizam koji povezuje ove transitions.

---

# 9. `WaitGroup`

Treća lekcija:

```text
03-waitgroup.md
```

uvodi:

```go
sync.WaitGroup
```

`WaitGroup` rešava drugačiji problem od channel-a.

Channel rešava:

```text
communication
```

dok `WaitGroup` rešava:

```text
completion coordination
```

Osnovni model:

```text
             Main
              │
              ▼
         WaitGroup
        /    │    \
       ▼     ▼     ▼
      G1     G2    G3
       │     │     │
       ▼     ▼     ▼
      Done  Done  Done
        \     │     /
         \    │    /
          ▼   ▼   ▼
            Wait
              │
              ▼
           continue
```

---

# `WaitGroup` Lifecycle

Tipičan obrazac:

```go
var wg sync.WaitGroup

wg.Add(3)

go func() {
    defer wg.Done()
    workA()
}()

go func() {
    defer wg.Done()
    workB()
}()

go func() {
    defer wg.Done()
    workC()
}()

wg.Wait()
```

Mentalni model:

```text
Add(3)
  │
  ├── worker A ── Done()
  ├── worker B ── Done()
  └── worker C ── Done()
             │
             ▼
          Wait()
             │
             ▼
          continue
```

---

# `Add`

`Add` povećava counter.

```go
wg.Add(3)
```

znači:

```text
3 units of work are outstanding
```

Kada worker završi:

```go
wg.Done()
```

counter se smanjuje.

---

# `Done`

Idiomatic pattern:

```go
go func() {
    defer wg.Done()

    doWork()
}()
```

Prednost `defer` pristupa jeste što se `Done()` izvršava kada goroutine napusti funkciju, uključujući normalan return iz funkcije.

Ipak, `Done()` mora biti pravilno vezan za odgovarajući `Add()`.

---

# `Wait`

```go
wg.Wait()
```

blokira dok counter ne postane:

```text
0
```

Ako je:

```text
counter = 3
```

`Wait()` čeka.

Ako:

```text
counter = 0
```

`Wait()` može nastaviti.

---

# `WaitGroup` Is Not a Channel

Važno je razlikovati:

```text
WaitGroup
```

od:

```text
Channel
```

`WaitGroup` ne prenosi rezultate.

Ne može odgovoriti:

```text
"What value did worker produce?"
```

Njegova osnovna uloga je:

```text
"Are all tracked goroutines finished?"
```

Ako treba prenos podataka:

```text
worker → result channel
```

Ako treba čekanje na završetak:

```text
WaitGroup
```

Često se koriste zajedno.

---

# `WaitGroup` + Channel

Primer konceptualne arhitekture:

```text
               Jobs
                │
                ▼
       ┌────────┼────────┐
       ▼        ▼        ▼
      G1       G2       G3
       │        │        │
       └────────┼────────┘
                │
                ▼
             Results
```

`WaitGroup` može pratiti:

```text
G1
G2
G3
```

dok channel prenosi:

```text
results
```

Ovo je fundamentalni building block worker pool-a.

---

# 10. Worker Pool

Četvrta lekcija:

```text
04-worker-pools.md
```

uvodi worker pool.

Problem:

```text
N jobs
```

ne znači automatski da želimo:

```text
N goroutines
```

Ako imamo:

```text
1,000,000 jobs
```

pokretanje milion goroutine-a može biti loš design.

Worker pool uvodi bounded concurrency:

```text
                  Jobs
                   │
                   ▼
             ┌───────────┐
             │ Job Queue │
             └─────┬─────┘
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Worker 1 Worker 2 Worker 3
```

Broj worker-a je kontrolisan.

---

# Bounded Concurrency

Ako imamo:

```go
workerCount := 8
```

onda sistem može imati najviše približno:

```text
8 active workers
```

koji procesiraju work items u datom pool-u.

To omogućava kontrolu:

* CPU usage;
* memory usage;
* downstream pressure;
* external API concurrency;
* database connection usage.

---

# Worker Pool Architecture

Tipičan model:

```text
                     Producer
                        │
                        ▼
                  ┌───────────┐
                  │ jobs chan  │
                  └─────┬─────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Worker 1      Worker 2      Worker 3
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                  results chan
                        │
                        ▼
                    Consumer
```

Worker pool uvodi tri odvojena concerns:

```text
work distribution
+
bounded execution
+
result collection
```

---

# Worker Pool and Backpressure

Ako je:

```text
producer rate > worker processing rate
```

jobs channel počinje da se puni.

Ako je buffered:

```text
[ job ][ job ][ job ][ job ]
```

kada postane pun:

```text
producer
   │
   ▼
FULL JOB QUEUE
   │
   ▼
BLOCK
```

Na ovaj način worker pool prirodno propagira backpressure.

---

# Worker Count Is a Capacity Decision

Broj worker-a nije samo:

```text
"how many goroutines can I create?"
```

To je pitanje:

```text
"What level of concurrent work can the system safely sustain?"
```

Za CPU-bound posao worker count često zavisi od CPU parallelism-a.

Za I/O-bound posao može biti veći, ali zavisi od:

* downstream capacity;
* latency;
* connection pools;
* rate limits;
* memory;
* external service constraints.

Detaljnija performance analiza dolazi kasnije.

---

# Worker Pool Shutdown

Worker pool mora imati jasan shutdown model.

Na primer:

```text
Producer
   │
   ▼
jobs channel
   │
   ├── Worker A
   ├── Worker B
   └── Worker C
```

Kada nema više poslova:

```text
close(jobs)
```

Workers koriste:

```go
for job := range jobs {
    process(job)
}
```

i završavaju kada:

```text
jobs closed
+
buffer drained
```

Ovo direktno koristi Module 1 znanje o channel lifecycle-u.

---

# Coordinating Worker Completion

Ako imamo više worker-a:

```text
Worker A ──┐
Worker B ──┼──► all workers done
Worker C ──┘
```

treba način da se zna kada su svi završili.

Tu se pojavljuje:

```text
WaitGroup
```

Konceptualno:

```text
close(jobs)
     │
     ▼
workers finish
     │
     ▼
wg.Done()
     │
     ▼
wg.Wait()
     │
     ▼
all workers complete
```

Ovaj pattern predstavlja osnovu za bezbedno zatvaranje result channel-a.

---

# Result Channel Lifecycle

Jedan od važnih worker-pool patterns:

```text
workers
   │
   │ results
   ▼
results channel
```

Ali:

> Ko zatvara `results`?

Ne consumer.

Ne pojedinačni worker.

Potrebna je koordinacija:

```text
Worker A ──┐
Worker B ──┼──► WaitGroup
Worker C ──┘
               │
               ▼
          all workers done
               │
               ▼
        close(results)
```

To je jedna od ključnih praktičnih primena `WaitGroup` + channel kombinacije.

---

# Module 2 Core Relationship

Do ove tačke Module 2 uvodi sledeću arhitekturu:

```text
              Communication
                    │
                 Channel
                    │
                    ▼
               Coordination
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       select              WaitGroup
          │                   │
          └─────────┬─────────┘
                    ▼
              Worker Pool
```

`select` rešava:

```text
Which event can I react to?
```

`WaitGroup` rešava:

```text
When are all tracked goroutines finished?
```

Worker pool koristi oba koncepta zajedno sa channels.

---

# 11. Pipeline Architecture

Peta lekcija:

```text
05-pipelines.md
```

uvodi pipeline.

Pipeline predstavlja niz processing stages povezanih channels.

```text
Stage 1
   │
   ▼
Channel
   │
   ▼
Stage 2
   │
   ▼
Channel
   │
   ▼
Stage 3
```

Svaki stage može biti:

```text
goroutine + input channel + output channel
```

---

# Pipeline Stage Contract

Tipičan stage:

```go
func stage(in <-chan int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for value := range in {
            out <- transform(value)
        }
    }()

    return out
}
```

Ovo daje jasan contract:

```text
input:
<-chan int

output:
<-chan int
```

Stage:

```text
receives
transforms
sends
closes output
```

---

# Pipeline Composition

Stages se mogu povezivati:

```go
stage3(stage2(stage1(input)))
```

Konceptualno:

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

Ovo predstavlja veoma elegantan oblik concurrency composition-a.

---

# Pipeline Benefits

Pipeline omogućava:

* separation of concerns;
* streaming;
* concurrent processing;
* stage-level isolation;
* composability;
* backpressure propagation.

Na primer:

```text
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

Svaki stage može imati sopstveni concurrency model.

---

# Pipeline Backpressure

Ako Stage 3 postane spor:

```text
Stage 1 → Stage 2 → Stage 3
                     ▲
                     │
                  slow
```

problem se može propagirati unazad:

```text
Stage 3
   ▲
   │
Stage 2 blocks
   ▲
   │
Stage 1 slows
```

Ovo je važna osobina channel-based pipeline-a.

Backpressure sprečava nekontrolisano gomilanje work-a kada downstream ne može da prati upstream.

---

# Pipeline Lifecycle

Svaki stage mora imati jasan lifecycle:

```text
start
  │
  ▼
receive
  │
  ▼
process
  │
  ▼
send
  │
  ▼
close output
  │
  ▼
exit
```

Ako jedan stage nikada ne zatvori output:

```text
downstream range
```

može ostati blokiran.

Ako downstream prestane da prima:

```text
upstream send
```

može ostati blokiran.

Zato pipeline lifecycle mora biti projektovan end-to-end.

---

# Pipeline Cancellation Preview

Pipeline sistemi posebno dobro pokazuju problem early termination-a.

```text
Stage 1 → Stage 2 → Stage 3
                         │
                         ▼
                    consumer exits
```

Ako Stage 2 i Stage 1 to ne saznaju:

```text
Stage 1
   │
   ▼
blocked send
```

Ovo je jedan od razloga zbog kojih će cancellation biti centralna tema kasnijeg concurrency dizajna.

---

# 12. Fan-Out

Šesta lekcija:

```text
06-fan-out.md
```

uvodi fan-out.

Fan-out znači da jedan stream work-a obrađuje više concurrent consumers.

```text
                  ┌──► Worker A
                  │
Jobs ─────────────┼──► Worker B
                  │
                  └──► Worker C
```

Svi workers mogu čitati iz istog jobs channel-a.

---

# Fan-Out Mechanics

Ako imamo:

```go
jobs := make(chan Job)
```

i:

```text
Worker A ── receives jobs
Worker B ── receives jobs
Worker C ── receives jobs
```

svaki work item se dobija od jednog receiver-a.

Konceptualno:

```text
Job 1 → Worker A
Job 2 → Worker C
Job 3 → Worker B
Job 4 → Worker A
```

Distribucija zavisi od runtime scheduling i channel readiness, a ne od unapred definisanog round-robin plana.

---

# Fan-Out Is Not Broadcast

Veoma važna razlika.

Fan-out:

```text
Job
 │
 └──► ONE worker
```

Broadcast:

```text
Event
 ├──► Worker A
 ├──► Worker B
 └──► Worker C
```

Kod fan-out-a više workers dele posao.

Kod broadcast-a svaki subscriber dobija event.

Običan Go channel sa više receiver-a prirodno podržava fan-out semantics, ne broadcast semantics.

---

# Fan-Out Use Cases

Fan-out je koristan kada:

* work items mogu da se obrađuju nezavisno;
* želimo povećati throughput;
* posao je dovoljno granularan;
* želimo bounded concurrency;
* downstream processing može biti paralelizovan.

Primer:

```text
1000 URLs
     │
     ▼
 URL jobs
     │
     ├──► Worker 1
     ├──► Worker 2
     ├──► Worker 3
     └──► Worker 4
```

---

# Ordering in Fan-Out

Fan-out često narušava originalni ordering.

Input:

```text
1
2
3
4
```

može dati:

```text
3
1
4
2
```

ako rezultate skupljamo po završetku.

Ako je ordering važan, mora postojati dodatni design:

```text
sequence number
+
reordering stage
```

Ne treba pretpostaviti da concurrency automatski čuva input ordering.

---

# 13. Fan-In

Sedma lekcija:

```text
07-fan-in.md
```

uvodi fan-in.

Fan-in kombinuje više input streams u jedan output stream.

```text
Worker A ──┐
Worker B ──┼──► Results
Worker C ──┘
```

To je suprotno od fan-out-a.

```text
Fan-Out:
1 → N

Fan-In:
N → 1
```

---

# Fan-In Architecture

Na primer:

```text
Input
  │
  ├──► Worker A ──┐
  │               │
  ├──► Worker B ──┼──► merged results
  │               │
  └──► Worker C ──┘
```

Consumer sada ima samo jedan output channel:

```go
for result := range results {
    process(result)
}
```

Umesto da prati tri odvojena channels.

---

# Fan-In Requires Completion Coordination

Najvažnije pitanje:

> Kada je bezbedno zatvoriti merged output channel?

Ne kada prvi worker završi.

Ne kada consumer odluči da je dosta.

Tek kada su svi upstream producers završili.

```text
Worker A ── done ──┐
Worker B ── done ──┼──► all done
Worker C ── done ──┘
                       │
                       ▼
                 close(results)
```

Ovo je prirodna tačka za `WaitGroup`.

---

# Fan-In + WaitGroup

Konceptualni pattern:

```text
Worker A ──┐
Worker B ──┼──► WaitGroup
Worker C ──┘
              │
              ▼
           Wait()
              │
              ▼
       close(results)
```

A result forwarding može biti:

```text
Worker A ──► results
Worker B ──► results
Worker C ──► results
```

Kada svi worker-i završe:

```text
close(results)
```

Consumer može bezbedno:

```go
for result := range results {
    process(result)
}
```

---

# Fan-Out + Fan-In

Najvažniji kombinovani pattern:

```text
                       ┌──► Worker A ──┐
                       │               │
Input ─────► Fan-Out ──┼──► Worker B ──┼──► Fan-In ──► Output
                       │               │
                       └──► Worker C ──┘
```

Ovaj pattern omogućava:

```text
parallel processing
+
centralized result stream
```

To je jedan od najčešćih obrazaca za concurrent data processing.

---

# 14. Complete Module 2 Architecture

Kada se svi koncepti spoje:

```text
                              Input
                                │
                                ▼
                           ┌─────────┐
                           │ Channel │
                           └────┬────┘
                                │
                           Fan-Out
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                 Worker A    Worker B    Worker C
                    │           │           │
                    └───────────┼───────────┘
                                │
                              Fan-In
                                │
                                ▼
                         ┌─────────────┐
                         │   Results   │
                         └──────┬──────┘
                                │
                                ▼
                             Consumer
```

Koordinacija:

```text
             ┌──────── select ────────┐
             │                         │
             ▼                         ▼
          data event              shutdown
             │                         │
             └──────────┬──────────────┘
                        │
                        ▼
                   Worker lifecycle
                        │
                        ▼
                   WaitGroup
                        │
                        ▼
                    completion
```

Ovo je suština Module 2.

---

# 15. Module 2 Design Principles

## Principle 1 — Coordinate Explicitly

Nemoj se oslanjati na slučajan scheduler ordering.

Koristi:

* channels;
* `select`;
* `WaitGroup`;
* odgovarajući lifecycle mechanisms.

---

## Principle 2 — Bound Concurrency

Nemoj automatski kreirati jednu goroutine po work item-u ako broj work items može biti veliki.

Koristi worker pool kada je potreban kontrolisan concurrency level.

---

## Principle 3 — Separate Communication from Completion

Channel može prenositi:

```text
data
```

dok `WaitGroup` može pratiti:

```text
completion
```

Kombinovanje ova dva mehanizma često daje čistiji dizajn.

---

## Principle 4 — Close Output at the Right Boundary

Ako više workers šalje rezultate:

```text
workers ──► results
```

output channel treba zatvoriti tek kada je poznato da više nema sender-a.

To zahteva koordinaciju.

---

## Principle 5 — Preserve Backpressure

Nemoj bez razloga uvoditi ogromne buffers.

Ako downstream ne može da prati upstream, sistem treba da ima definisano ponašanje.

---

## Principle 6 — Lifecycle Is End-to-End

Ne projektuj samo:

```text
start
```

Projektuj:

```text
start
  │
  ▼
process
  │
  ▼
stop
  │
  ▼
cleanup
```

za svaku concurrent component-u.

---

# 16. Module 2 Failure Modes

Najvažniji failure modes:

## `select` without a useful exit path

Worker može ostati blokiran:

```text
waiting forever
```

ako nijedan case nikada ne postane ready.

---

## Busy Loop with `default`

```go
for {
    select {
    case value := <-ch:
        process(value)
    default:
    }
}
```

može proizvesti visok CPU usage.

---

## Incorrect `WaitGroup.Add`

Ako broj `Add()` operacija ne odgovara broju `Done()` poziva, lifecycle može biti neispravan.

---

## Missing `Done`

Worker koji nikada ne pozove:

```go
wg.Done()
```

može ostaviti:

```go
wg.Wait()
```

blokiran zauvek.

---

## Closing Results Too Early

Ako result channel zatvoriš dok workers još šalju:

```text
send on closed channel
```

---

## Never Closing Results

Ako consumer koristi:

```go
for result := range results
```

a result channel se nikada ne zatvori:

```text
consumer waits forever
```

---

## Pipeline Stage Leak

Ako downstream završi, upstream stage može ostati blokiran na send-u.

---

## Fan-Out Ordering Assumption

Concurrent workers ne garantuju output ordering.

---

## Fan-In Completion Bug

Ako se output zatvori nakon što samo jedan worker završi:

```text
remaining workers
       │
       ▼
send on closed channel
```

---

# 17. Module 2 Competency Model

Module 2 treba završiti sa sledećim nivoom razumevanja:

```text
                 Primitive
                    │
                    ▼
                 Pattern
                    │
                    ▼
              Coordination
                    │
                    ▼
           Concurrent System
```

Čitalac više ne treba samo da zna:

```go
select {}
```

već da zna:

```text
Why is select needed here?
What events are being coordinated?
What happens if no event occurs?
How does the component shut down?
```

Isto važi za `WaitGroup`:

```text
Why are we waiting?
Who is being tracked?
Who calls Done?
When is it safe to continue?
```

---

# 18. Module 2 Exit Criteria

Module 2 može se smatrati završenim kada čitalac može samostalno da implementira:

```text
Input
  │
  ▼
Bounded Worker Pool
  │
  ├── Worker 1
  ├── Worker 2
  ├── Worker 3
  └── Worker 4
  │
  ▼
Merged Results
  │
  ▼
Consumer
```

uz:

* `select` za relevantne events;
* `WaitGroup` za completion;
* channels za communication;
* `range` za stream consumption;
* pravilno zatvaranje channels;
* bounded concurrency;
* jasan ownership;
* kontrolisan lifecycle.

---

# 19. Module 2 Navigation

|  # | Lesson                                                        | Focus                           |
| -: | ------------------------------------------------------------- | ------------------------------- |
|  1 | [Select](./01-select.md)                                      | Multiplexing channel operations |
|  2 | [Select + Default](./02-select-default.md)                    | Non-blocking operations         |
|  3 | [WaitGroup](./03-waitgroup.md)                                | Completion coordination         |
|  4 | [Worker Pools](./04-worker-pools.md)                          | Bounded concurrency             |
|  5 | [Pipelines](./05-pipelines.md)                                | Concurrent processing stages    |
|  6 | [Fan-Out](./06-fan-out.md)                                    | Work distribution               |
|  7 | [Fan-In](./07-fan-in.md)                                      | Result aggregation              |
|  8 | [Summary & Exercises](./08-module-2-summary-and-exercises.md) | Consolidation                   |

---

# 20. Transition from Module 1 to Module 2

Module 1:

```text
Goroutine
    │
    ▼
Channel
    │
    ▼
Goroutine
```

Module 2:

```text
                       ┌──► Worker
                       │
Input ──► Channel ─────┼──► Worker ──► Results
                       │
                       └──► Worker
```

Module 1 je učio:

```text
communication
```

Module 2 uči:

```text
coordination
```

To je ključna razlika.

---

# 21. What Comes Next

Nakon Module 2, concurrency model će biti dovoljno bogat da možemo preći na probleme koji zahtevaju:

```text
shared state
synchronization
timeouts
cancellation
scheduler behavior
parallelism
```

Tu dolaze:

```text
Mutex
RWMutex
Once
Timeouts
Cancellation
Go Scheduler
GOMAXPROCS
Parallelism vs Concurrency
```

Drugim rečima:

```text
Module 1
Communication
      │
      ▼
Module 2
Coordination
      │
      ▼
Module 3
Synchronization + Runtime Behavior
      │
      ▼
Module 4
Advanced / Expert Concurrency
```

---

# Module 2 — Coordination & Concurrency Patterns

## 22. Detailed Concept Map

Module 2 nije skup izolovanih lekcija. Njegov pravi cilj je da pokaže kako se osnovni concurrency primitives kombinuju u veće, kontrolisane sisteme.

Celokupna konceptualna mapa izgleda ovako:

```text
                         CHANNELS
                            │
                            ▼
                     COMMUNICATION
                            │
                            ▼
                        SELECT
                     ┌──────┴──────┐
                     │             │
                     ▼             ▼
                  blocking     non-blocking
                     │             │
                     │          default
                     │
                     ▼
                COORDINATION
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     WAITGROUP               CHANNELS
          │                     │
          └──────────┬──────────┘
                     ▼
              WORKER POOLS
                     │
             ┌───────┴───────┐
             ▼               ▼
          FAN-OUT          FAN-IN
             │               │
             └───────┬───────┘
                     ▼
                 PIPELINES
                     │
                     ▼
          COMPOSABLE CONCURRENCY
```

Ovaj progression je važan zato što svaka nova tema rešava problem koji prethodna tema sama ne rešava.

---

# 23. `select` Solves Choice

Channel sam po sebi omogućava komunikaciju:

```go
value := <-jobs
```

Ali ova operacija predstavlja samo jednu mogućnost:

```text
wait for jobs
```

U realnom sistemu worker često mora da reaguje na više događaja:

```text
                 ┌──► new job
                 │
Worker ──────────┼──► shutdown
                 │
                 └──► timeout
```

`select` uvodi upravo ovaj oblik koordinacije.

```go
select {
case job := <-jobs:
    process(job)

case <-done:
    return

case <-timeout:
    handleTimeout()
}
```

Zato se `select` može posmatrati kao **event coordination primitive**.

---

# 24. `default` Solves Immediate Choice

Običan `select` kaže:

> Čekaj dok jedna od operacija ne bude moguća.

`select` sa `default` kaže:

> Ako nijedna operacija trenutno nije moguća, nastavi odmah.

```go
select {
case value := <-ch:
    process(value)

default:
    doSomethingElse()
}
```

To daje dva fundamentalna concurrency modela:

```text
Blocking:
    wait until ready

Non-blocking:
    try now, otherwise continue
```

Oba mogu biti korisna, ali imaju potpuno drugačiju semantiku.

---

# 25. `WaitGroup` Solves Completion

`select` odgovara na:

```text
"What event should I react to?"
```

`WaitGroup` odgovara na:

```text
"When have all tracked goroutines finished?"
```

To je druga vrsta koordinacije.

```text
G1 ──► Done()
G2 ──► Done()
G3 ──► Done()
          │
          ▼
       Wait()
          │
          ▼
       continue
```

Zato se `select` i `WaitGroup` ne takmiče jedan sa drugim.

Oni rešavaju različite probleme.

---

# 26. Worker Pool Solves Capacity

Ako imamo veliki broj work items:

```text
Job 1
Job 2
Job 3
...
Job 1,000,000
```

nije nužno dobro imati:

```text
1,000,000 goroutines
```

Worker pool uvodi ograničenje:

```text
                Jobs
                  │
                  ▼
             Job Queue
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Worker 1   Worker 2   Worker 3
```

Sistem sada ima eksplicitni concurrency budget:

```text
workerCount = 3
```

To je **bounded concurrency**.

---

# 27. Pipeline Solves Processing Composition

Worker pool organizuje:

```text
many jobs
    ↓
many workers
```

Pipeline organizuje:

```text
stage
  ↓
stage
  ↓
stage
```

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

Svaki stage može biti concurrent component.

To omogućava da se sistem modeluje kao niz nezavisnih processing stages.

---

# 28. Fan-Out Solves Work Distribution

Fan-out rešava:

> Kako jedan work stream distribuirati između više concurrent consumers?

```text
                 ┌──► Worker A
                 │
Jobs ────────────┼──► Worker B
                 │
                 └──► Worker C
```

Jedan job ide jednom worker-u.

Cilj je povećanje throughput-a kroz konkurentnu obradu.

---

# 29. Fan-In Solves Result Aggregation

Fan-in rešava suprotan problem:

> Kako rezultate iz više concurrent sources predstaviti kao jedan stream?

```text
Worker A ──┐
Worker B ──┼──► Results
Worker C ──┘
```

Consumer tada ne mora da zna koliko upstream producers postoji.

On samo radi:

```go
for result := range results {
    consume(result)
}
```

---

# 30. The Core Composition

Kada se sve spoji, dobijamo jedan od najvažnijih concurrency modela u Go-u:

```text
                         INPUT
                           │
                           ▼
                     ┌───────────┐
                     │ jobs chan │
                     └─────┬─────┘
                           │
                       FAN-OUT
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Worker A      Worker B      Worker C
             │             │             │
             └─────────────┼─────────────┘
                           │
                         FAN-IN
                           │
                           ▼
                   ┌──────────────┐
                   │ results chan │
                   └──────┬───────┘
                          │
                          ▼
                       OUTPUT
```

A lifecycle coordination:

```text
Worker A ──┐
Worker B ──┼──► WaitGroup ──► close(results)
Worker C ──┘
```

Dok svaki worker može imati:

```go
select {
case job := <-jobs:
    process(job)

case <-done:
    return
}
```

Sada imamo:

```text
communication
+
selection
+
completion
+
bounded concurrency
+
parallel work distribution
+
result aggregation
```

To je srž Module 2.

---

# 31. Communication Topology

Jedan od najvažnijih skill-ova koji Module 2 razvija jeste sposobnost čitanja concurrency topology-ja.

Na primer:

```text
Producer
   │
   ▼
 jobs
   │
   ├────────────┐
   ▼            ▼
 Worker A     Worker B
   │            │
   └──────┬─────┘
          ▼
       results
          │
          ▼
       Consumer
```

Potrebno je moći odgovoriti na:

1. Ko je producer?
2. Ko je consumer?
3. Ko poseduje `jobs` channel?
4. Ko ga zatvara?
5. Ko poseduje `results` channel?
6. Ko ga zatvara?
7. Koliko workers postoji?
8. Kako se workers završavaju?
9. Šta se događa ako consumer ode ranije?
10. Šta se događa ako producer stane?
11. Gde postoji mogućnost blocking-a?
12. Gde postoji mogućnost goroutine leak-a?

Ova pitanja su važnija od samog poznavanja syntax-e.

---

# 32. Channel Ownership

Module 2 dodatno učvršćuje koncept channel ownership-a.

Dobro projektovan sistem treba jasno da zna ko je odgovoran za lifecycle channel-a.

Tipičan producer:

```go
func produce() <-chan Job {
    jobs := make(chan Job)

    go func() {
        defer close(jobs)

        // produce jobs
    }()

    return jobs
}
```

Caller dobija:

```go
<-chan Job
```

i može samo da prima.

Producer zadržava ownership nad close operation-om.

---

# 33. Why Directional Channels Matter More in Module 2

U jednostavnom primeru:

```go
func worker(jobs chan Job)
```

nije jasno da li worker:

* šalje;
* prima;
* zatvara;
* radi sve navedeno.

Sa directional channel-om:

```go
func worker(jobs <-chan Job)
```

contract je eksplicitan:

```text
worker receives jobs
```

Isto:

```go
func merge(inputs []<-chan Result) <-chan Result
```

govori mnogo više o dizajnu API-ja.

Directional channels zato nisu samo syntax convenience.

Oni su alat za izražavanje concurrency ownership-a i communication direction-a.

---

# 34. Producer Contract

Producer tipično:

```text
create
   │
   ▼
send values
   │
   ▼
close output
```

Na primer:

```go
func producer() <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 10; i++ {
            out <- i
        }
    }()

    return out
}
```

Producer contract je:

```text
"I produce values and eventually close my output."
```

---

# 35. Consumer Contract

Consumer tipično:

```text
receive
   │
   ▼
process
   │
   ▼
stop when closed
```

Na primer:

```go
for value := range input {
    process(value)
}
```

Ovaj pattern implicitno zavisi od producer lifecycle-a.

Ako producer nikada ne zatvori `input`, consumer može čekati beskonačno.

---

# 36. Worker Contract

Worker pool worker ima nešto složeniji contract:

```text
receive jobs
    │
    ▼
process jobs
    │
    ▼
emit results
    │
    ▼
stop
```

Tipično:

```go
func worker(
    jobs <-chan Job,
    results chan<- Result,
) {
    for job := range jobs {
        results <- process(job)
    }
}
```

Worker ne treba da zatvara `jobs` zato što ga ne poseduje.

Worker takođe ne treba automatski da zatvara `results` ako postoje drugi workers koji još šalju rezultate.

---

# 37. Ownership Matrix

Kod većih concurrency sistema korisno je napraviti ownership matricu.

| Resource  | Creator              | Sender   | Receiver | Closer      |
| --------- | -------------------- | -------- | -------- | ----------- |
| `jobs`    | producer/coordinator | producer | workers  | producer    |
| `results` | coordinator          | workers  | consumer | coordinator |
| `done`    | coordinator          | —        | workers  | coordinator |

Ovakva tabela može veoma brzo otkriti lifecycle greške.

---

# 38. Closing Rule

Jedno od najvažnijih pravila Go concurrency programming-a:

> Channel treba da zatvori strana koja zna da više neće biti novih send operations.

To je posebno važno kod fan-in sistema.

Ako imamo:

```text
Worker A ──┐
Worker B ──┼──► results
Worker C ──┘
```

nijedan pojedinačni worker ne zna nužno da li su ostali završili.

Zato je centralna coordinator goroutine prirodnije mesto za:

```go
close(results)
```

nakon:

```go
wg.Wait()
```

---

# 39. Completion vs Closure

Važno je razlikovati:

```text
worker finished
```

od:

```text
results channel is closed
```

Worker može završiti:

```text
Worker A → done
```

dok:

```text
Worker B → still running
Worker C → still running
```

Zato:

```text
one worker finished
```

nije isto što i:

```text
no more results will ever arrive
```

Tek drugo stanje opravdava:

```go
close(results)
```

---

# 40. Completion Graph

Kod fan-in-a možemo ga predstaviti kao dependency graph:

```text
Worker A ──► Done A ──┐
                      │
Worker B ──► Done B ──┼──► All workers done
                      │
Worker C ──► Done C ──┘
                              │
                              ▼
                       close(results)
```

`WaitGroup` predstavlja praktičan mehanizam za ovu dependency relationship.

---

# 41. Concurrency vs Parallelism in Module 2

Module 2 pre svega uči **concurrency structure**, a ne garantovanu parallel execution.

Na primer:

```text
Worker A
Worker B
Worker C
```

znači da imamo više concurrent activities.

To ne garantuje da se sve tri fizički izvršavaju istovremeno na različitim CPU cores.

Razlika:

```text
Concurrency
    =
multiple tasks can make progress independently
```

nasuprot:

```text
Parallelism
    =
multiple tasks execute simultaneously
```

Runtime scheduler i `GOMAXPROCS` određuju detalje fizičkog execution model-a.

---

# 42. Why Worker Pools Are Not Automatically Faster

Povećanje broja workers:

```text
2 → 4 → 8 → 16 → 32
```

ne garantuje:

```text
throughput:
100 → 200 → 400 → 800 → 1600
```

Realni sistem može imati:

```text
CPU contention
lock contention
I/O saturation
database limits
network limits
memory pressure
scheduler overhead
```

Zato worker pool predstavlja **capacity control**, a ne automatsku performance optimization.

---

# 43. CPU-Bound Work

Za CPU-heavy workload:

```text
calculate
compress
hash
encode
transform
```

previše workers može proizvesti contention.

Primer:

```text
CPU cores = 8

workers:
8      → potentially useful
80     → not automatically better
800    → likely excessive
```

Tačan optimalni broj zavisi od workload-a i runtime/system characteristics.

---

# 44. I/O-Bound Work

Za I/O-bound workload:

```text
HTTP requests
database calls
filesystem
RPC
```

worker pool može imati više workers nego CPU cores zato što workers značajan deo vremena provode čekajući.

Ali i ovde postoji granica.

Na primer:

```text
HTTP API allows:
100 concurrent requests
```

worker pool od:

```text
1000 workers
```

ne znači automatski da će sistem biti bolji.

Može samo povećati:

* queueing;
* memory usage;
* downstream pressure;
* timeout frequency;
* rate-limit violations.

---

# 45. Queue Capacity

Buffered jobs channel predstavlja queue capacity.

Na primer:

```go
jobs := make(chan Job, 100)
```

znači da channel može zadržati do određenog broja queued values bez blokiranja sender-a, pod odgovarajućim uslovima.

Mentalni model:

```text
Producer
   │
   ▼
┌─────────────────────┐
│ job │ job │ job │...│
└─────────────────────┘
          100
```

Buffer nije besplatna memorija.

Veći buffer znači potencijalno više queued work-a.

---

# 46. Buffer Is Not Backpressure Elimination

Buffered channel:

```text
producer → [buffer] → workers
```

može odložiti blocking.

Ali ne uklanja capacity mismatch.

Ako:

```text
producer rate = 10,000 jobs/s
worker rate   = 1,000 jobs/s
```

queue će se eventualno napuniti.

```text
producer
   │
   ▼
buffer
██████████████████
        │
        ▼
      workers
```

Kada se capacity iscrpi, producer mora čekati ili sistem mora imati drugo definisano ponašanje.

---

# 47. Backpressure as a System Property

Backpressure nije samo channel behavior.

To je property celog sistema.

```text
External Input
      │
      ▼
Ingress
      │
      ▼
Queue
      │
      ▼
Workers
      │
      ▼
Database
```

Ako database postane spor:

```text
Database slows
      │
      ▼
Workers slow
      │
      ▼
Queue grows
      │
      ▼
Ingress blocks/slows
```

Dobro projektovan sistem dozvoljava da se pressure propagira kontrolisano.

---

# 48. Select and Backpressure

`select` može eksplicitno izraziti izbor između slanja i shutdown-a:

```go
select {
case jobs <- job:
    // accepted
case <-done:
    return
}
```

Producer tada neće zauvek čekati na pun queue ako postoji shutdown signal.

Ovo je važan pattern:

```text
             ┌──► queue accepts job
Producer ────┤
             └──► shutdown
```

---

# 49. Non-Blocking Queue Submission

Sa `default`:

```go
select {
case jobs <- job:
    accepted()

default:
    rejected()
}
```

Ovo može implementirati:

```text
try enqueue
```

semantics.

To može biti korisno kada sistem želi:

* reject;
* drop;
* fallback;
* metrics;
* alternative processing.

Ali nije isto što i:

```text
wait until queue has capacity
```

---

# 50. Queue Full Is a Business Decision

Ako je queue pun, sistem mora imati definisano ponašanje.

Moguće strategije:

```text
BLOCK
DROP
REJECT
RETRY
DEFER
SHED LOAD
```

Concurrency primitive samo omogućava implementaciju.

Sam poslovni zahtev određuje koja strategija je ispravna.

---

# 51. Structured Concurrency Perspective

Go nema jednu centralnu primitive koja automatski predstavlja kompletan structured-concurrency tree.

Zbog toga programer mora eksplicitno organizovati:

```text
parent
  │
  ├── child goroutine
  ├── child goroutine
  └── child goroutine
```

i lifecycle:

```text
start children
      │
      ▼
coordinate
      │
      ▼
wait
      │
      ▼
stop / cleanup
```

`WaitGroup`, channels i `select` su osnovni building blocks za takav dizajn.

Kasniji `context` mehanizmi dodatno unapređuju cancellation propagation.

---

# 52. Goroutine Lifecycle Contract

Za svaku goroutine treba moći odgovoriti:

```text
Who starts me?
Who can stop me?
What am I waiting for?
What happens if my input closes?
What happens if my consumer exits?
Who waits for me?
```

Ako na ova pitanja nema odgovora, concurrency design verovatno nije dovoljno precizan.

---

# 53. Goroutine Leak Analysis

Razmotrimo:

```go
func worker(in <-chan int) {
    for value := range in {
        process(value)
    }
}
```

Ako `in` nikada nije zatvoren:

```text
worker
  │
  ▼
range
  │
  ▼
wait forever
```

To možda nije leak ako je channel namerno dugotrajan.

Ali ako je lifecycle trebalo da se završi, onda imamo leak.

Zato:

> Goroutine leak nije "goroutine postoji dugo"; leak je goroutine koja ostaje aktivna bez legitimnog lifecycle razloga.

---

# 54. Leak Through Blocked Send

Drugi čest scenario:

```go
func stage(out chan<- int) {
    for {
        out <- compute()
    }
}
```

Ako consumer prestane:

```text
Stage
  │
  ▼
out <- value
  │
  ▼
BLOCK FOREVER
```

Ako nema cancellation path-a, stage može ostati zauvek blokiran.

---

# 55. Leak Through Blocked Receive

Isto važi za:

```go
value := <-jobs
```

Ako producer nikada ne šalje niti zatvara channel:

```text
Worker
  │
  ▼
receive
  │
  ▼
BLOCK FOREVER
```

Zbog toga lifecycle channels mora biti definisan.

---

# 56. Leak Through `WaitGroup`

`WaitGroup` može prikriti leak ako coordinator čeka worker-e koji nikada neće završiti:

```text
Main
 │
 ▼
wg.Wait()
 │
 ▼
WAIT FOREVER
```

Ali pravi problem je često dublje:

```text
Worker
 │
 ▼
blocked send
 │
 ▼
never calls Done()
```

Zato `WaitGroup` nije zaštita od lifecycle bug-ova.

On samo koordinira završetak ako se završetak zaista dogodi.

---

# 57. The "Who Closes?" Question

Kod svakog channel-based design-a treba postaviti:

```text
Who creates this channel?
Who sends?
Who receives?
Who closes?
When is it closed?
What happens after closure?
```

Ako je odgovor:

```text
"Everyone"
```

design treba ponovo razmotriti.

Channel ownership treba biti što jasniji.

---

# 58. The "Who Waits?" Question

Za `WaitGroup`:

```text
Who calls Add?
Who calls Done?
Who calls Wait?
```

Tipičan model:

```text
Coordinator
    │
    ├── Add
    │
    ├── start workers
    │
    └── Wait
          │
          ▼
       completion
```

Worker:

```text
worker
  │
  └── Done
```

Ova separation of responsibilities smanjuje mogućnost lifecycle grešaka.

---

# 59. The "Who Owns Shutdown?" Question

Shutdown takođe treba imati owner-a.

Na primer:

```text
Coordinator
     │
     ▼
close(done)
     │
     ├──► Worker A stops
     ├──► Worker B stops
     └──► Worker C stops
```

Ako više goroutine-a pokušava:

```go
close(done)
```

može doći do:

```text
panic: close of closed channel
```

Zato shutdown ownership mora biti eksplicitan.

---

# 60. Module 2 as a Design Vocabulary

Na kraju ovog modula programer dobija vocabulary za opisivanje concurrency arhitekture:

```text
goroutine
channel
select
blocking
non-blocking
WaitGroup
worker pool
bounded concurrency
pipeline
stage
fan-out
fan-in
backpressure
completion
ownership
shutdown
goroutine leak
```

Ovo je mnogo važnije od memorisanja pojedinačnih code snippets.

---

# 61. Example: Concurrent File Processing System

Pretpostavimo sistem koji obrađuje veliki broj fajlova.

Pipeline:

```text
File Paths
    │
    ▼
Discovery
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
Write
```

Možemo koristiti:

```text
Fan-Out
```

za parallel parsing:

```text
                 ┌──► Parser 1
                 │
Files ───────────┼──► Parser 2
                 │
                 └──► Parser 3
```

I:

```text
Fan-In
```

za rezultate:

```text
Parser 1 ──┐
Parser 2 ──┼──► results
Parser 3 ──┘
```

`WaitGroup` koordinira završetak parser-a.

`select` može omogućiti shutdown.

---

# 62. Example: HTTP Request Processing

Drugi primer:

```text
Incoming requests
       │
       ▼
     Queue
       │
       ▼
┌──────┼──────┐
▼      ▼      ▼
W1     W2     W3
│      │      │
└──────┼──────┘
       ▼
  Result/Response
```

Worker pool omogućava:

```text
maximum N concurrent requests
```

`select` može omogućiti:

```text
request
OR
shutdown
OR
timeout
```

`WaitGroup` može omogućiti graceful worker completion.

---

# 63. Example: Concurrent API Aggregation

Pretpostavimo da servis mora pozvati tri nezavisna API-ja:

```text
             Request
          /     |     \
         ▼      ▼      ▼
       API A  API B  API C
         │      │      │
         └──────┼──────┘
                ▼
             Combine
```

Ovo je concurrency fan-out/fan-in scenario.

Ali postoji važna razlika:

```text
API A
API B
API C
```

mogu imati različite latency characteristics.

`select` i kasniji cancellation mechanisms omogućavaju da sistem ne čeka neograničeno na jednu dependency.

---

# 64. Example: Streaming Transformation

Pipeline:

```text
Input Stream
     │
     ▼
Decode
     │
     ▼
Validate
     │
     ▼
Transform
     │
     ▼
Encode
     │
     ▼
Output Stream
```

Prednost nije samo parallelism.

Pipeline omogućava **streaming**.

Dok se prvi item obrađuje u Stage 3:

```text
Stage 1 → Item 2
Stage 2 → Item 1
Stage 3 → Item 0
```

više stages može istovremeno raditi na različitim items.

To povećava resource utilization kada je workload odgovarajuć.

---

# 65. Throughput vs Latency

Concurrency design mora razlikovati:

```text
throughput
```

od:

```text
latency
```

Worker pool često povećava:

```text
throughput
```

jer više work items može biti obrađeno concurrent.

Ali povećanje concurrency-a ne mora smanjiti latency pojedinačnog item-a.

Na primer:

```text
one request:
100 ms
```

ne postaje automatski:

```text
10 ms
```

samo zato što imamo 10 workers.

Concurrency uglavnom utiče na broj work items koji mogu biti obrađeni u određenom periodu.

---

# 66. Amdahl's Law Perspective

Ako je deo sistema serijalan:

```text
Serial ───────────────►
       │
       ▼
Parallel Work
       │
       ▼
Serial ───────────────►
```

ukupni speedup ima ograničenje.

Ako samo 80% workload-a može biti paralelizovano, beskonačno povećanje broja workers neće proizvesti beskonačan speedup.

Zato concurrency architecture treba da identifikuje:

```text
parallelizable work
```

i:

```text
serialization points
```

---

# 67. Contention

Više workers može konkurisati za isti resource:

```text
Worker A ──┐
Worker B ──┼──► Database
Worker C ──┘
```

Ako database podržava samo ograničen broj connections, dodatni workers neće nužno povećati throughput.

Slični problemi postoje sa:

* mutex-ima;
* filesystem resources;
* network sockets;
* external APIs;
* CPU;
* memory bandwidth.

Zato concurrency treba posmatrati kao sistemski resource-management problem.

---

# 68. Observability

Concurrent systems su teži za debugging.

Dobro dizajniran worker pool treba imati mogućnost da prati:

```text
active workers
queued jobs
completed jobs
failed jobs
processing latency
queue latency
worker utilization
shutdown duration
```

Na primer:

```text
jobs queued       = 42
workers active   = 8
jobs completed   = 913
jobs failed      = 7
```

Bez ovakvih podataka teško je proceniti da li je concurrency model zaista efikasan.

---

# 69. Error Propagation

Module 2 prvenstveno obrađuje concurrency coordination, ali error propagation je neodvojiv deo realnih sistema.

Ako worker pronađe grešku:

```text
Worker A ──► ERROR
```

sistem mora odlučiti:

```text
continue?
retry?
stop worker?
stop whole pipeline?
cancel siblings?
report error?
```

Channel može preneti rezultat:

```go
type Result struct {
    Value any
    Err   error
}
```

Ali za kompleksnije cancellation semantics biće potreban napredniji lifecycle model.

---

# 70. Errors Are Part of Concurrency Protocol

Ako worker vraća:

```text
result
```

ali error ignorišemo:

```text
Worker
  │
  └── error lost
```

možemo dobiti sistem koji izgleda uspešno iako je deo work-a propao.

Concurrency protocol zato treba da definiše:

```text
success
failure
shutdown
completion
```

a ne samo:

```text
data flow
```

---

# 71. Panic and Worker Lifecycle

Ako worker panic-uje, očekivani lifecycle može biti prekinut.

Na primer:

```go
go func() {
    defer wg.Done()
    process()
}()
```

Ako `process()` panic-uje, `defer wg.Done()` će se izvršiti dok panic propagira, ali sama goroutine neće nastaviti normalno.

U realnom sistemu treba eksplicitno definisati:

```text
panic policy
```

posebno za long-running workers.

Ovo je jedna od tema koje treba detaljnije obraditi kada se concurrency sistemi približe production-grade nivou.

---

# 72. Testing Module 2 Systems

Concurrency code zahteva drugačiji testing mindset.

Test ne treba samo proveriti:

```text
expected output
```

već i:

```text
completion
shutdown
ordering assumptions
race behavior
leaks
timeouts
```

Na primer, worker pool test treba proveriti:

```text
all jobs processed
```

ali i:

```text
all workers terminate
```

---

# 73. Determinism

Concurrent execution nije deterministička.

Ako imamo:

```text
Worker A
Worker B
Worker C
```

redosled može varirati između test runs.

Zato testovi ne bi trebalo da zavise od:

```text
"Worker A must finish before Worker B"
```

ako to nije deo contract-a.

Umesto toga treba testirati invariant:

```text
all expected results eventually appear
```

---

# 74. Race Detection

Concurrent code treba testirati sa Go race detector-om:

```text
go test -race ./...
```

Race detector je posebno važan kada se Module 2 patterns kombinuju sa shared state-om.

Channel-based communication može smanjiti potrebu za shared mutable state-om, ali ga ne eliminiše automatski.

---

# 75. Benchmarking Worker Pools

Za worker pool je korisno meriti:

```text
throughput
latency
CPU
memory
allocations
```

i menjati:

```text
worker count
buffer size
batch size
```

Na primer:

```text
workers = 1
workers = 2
workers = 4
workers = 8
workers = 16
```

Rezultat može pokazati da optimum nije tamo gde se očekuje.

---

# 76. Do Not Optimize by Intuition Alone

Concurrency code često izgleda "brže" zato što ima više goroutine-a.

To nije dovoljan dokaz.

Validan zaključak zahteva:

```text
benchmark
+
profiling
+
realistic workload
```

Concurrency architecture treba da rešava konkretan problem, a ne samo da koristi više goroutine-a zato što je to moguće.

---

# 77. Production Readiness Checklist

Pre nego što concurrency component smatramo production-ready, treba proveriti:

### Lifecycle

* [ ] Ko pokreće goroutine?
* [ ] Ko je zaustavlja?
* [ ] Ko čeka njen završetak?
* [ ] Da li postoji shutdown path?

### Channels

* [ ] Ko kreira channel?
* [ ] Ko šalje?
* [ ] Ko prima?
* [ ] Ko zatvara?
* [ ] Da li se channel zatvara tačno jednom?

### `select`

* [ ] Da li postoji mogućnost beskonačnog blocking-a?
* [ ] Da li `default` pravi busy loop?
* [ ] Da li shutdown može prekinuti blocking operation?

### `WaitGroup`

* [ ] Da li svaki `Add` ima odgovarajući `Done`?
* [ ] Da li je `Wait` pozvan na odgovarajućem mestu?
* [ ] Da li worker može ostati blokiran pre `Done`?

### Worker Pool

* [ ] Da li je concurrency bounded?
* [ ] Da li je queue bounded?
* [ ] Šta se dešava kada je queue pun?
* [ ] Da li se workers pravilno gase?

### Pipeline

* [ ] Da li svaki stage zatvara svoj output?
* [ ] Šta se dešava ako downstream prestane?
* [ ] Da li postoji cancellation strategy?
* [ ] Da li postoji goroutine leak?

### Fan-Out / Fan-In

* [ ] Da li je ordering relevantan?
* [ ] Da li svi workers završavaju?
* [ ] Da li se merged output zatvara tek nakon svih senders?
* [ ] Da li consumer može završiti prerano?

---

# 78. Common Anti-Patterns

## Anti-Pattern 1 — Goroutine per Item Without a Bound

```go
for _, job := range jobs {
    go process(job)
}
```

Ovo može biti potpuno validno za mali i kontrolisan workload.

Ali za potencijalno neograničen input može napraviti:

```text
unbounded concurrency
```

---

## Anti-Pattern 2 — Infinite Polling

```go
for {
    select {
    case job := <-jobs:
        process(job)
    default:
    }
}
```

Ako nema drugih aktivnosti:

```text
CPU usage ↑
```

---

## Anti-Pattern 3 — Consumer Closes Producer Channel

Consumer:

```go
close(jobs)
```

dok producer još šalje:

```go
jobs <- job
```

može izazvati panic.

Consumer ne treba da zatvara channel samo zato što ga prima.

---

## Anti-Pattern 4 — Multiple Closers

```text
Worker A ── close(results)
Worker B ── close(results)
```

Drugi close izaziva panic.

Channel lifecycle treba da ima jasan owner.

---

## Anti-Pattern 5 — Waiting Before Starting Consumers

Ako se radi:

```text
produce
   │
   ▼
wait for producers
   │
   ▼
start consumer
```

a producer šalje na unbuffered ili pun buffered channel:

```text
producer blocks
wait blocks
consumer never starts
```

Dobijamo deadlock.

---

# 79. Start Consumers Before Waiting When Necessary

Kod streaming systems često treba:

```text
start producers
start consumers
wait for completion
```

a ne:

```text
start producers
wait
start consumers
```

Jer consumers mogu biti potrebni da bi producers mogli da nastave.

Ovo je odličan primer kako concurrency topology direktno utiče na correctness.

---

# 80. Deadlock Analysis

Kod svakog concurrent system-a korisno je nacrtati dependency graph.

Na primer:

```text
Producer
   │
   ▼
jobs channel
   │
   ▼
Worker
   │
   ▼
results channel
   │
   ▼
Consumer
```

Ako bilo koja tačka čeka drugu tačku koja čeka nju:

```text
A waits for B
B waits for A
```

imamo deadlock cycle.

Module 2 treba da nauči čitaoca da takve cikluse prepoznaje pre runtime-a.

---

# 81. Blocking Graph

Jedan koristan model:

```text
Producer
   │
   │ send
   ▼
 jobs
   │
   │ receive
   ▼
Worker
   │
   │ send
   ▼
results
   │
   │ receive
   ▼
Consumer
```

Za svaku strelicu pitamo:

```text
Can this operation block?
Who unblocks it?
What happens if the other side disappears?
```

Ovo je praktičan način za concurrency code review.

---

# 82. Concurrency Code Review Questions

Senior-level review treba da postavi pitanja kao:

1. Koji je lifecycle svakog goroutine-a?
2. Koji je owner svakog channel-a?
3. Ko zatvara channel?
4. Da li postoji bounded concurrency?
5. Da li postoji backpressure?
6. Da li postoji cancellation?
7. Da li postoji possibility of leak?
8. Da li postoji possibility of deadlock?
9. Da li ordering matters?
10. Da li errors propagate?
11. Da li shutdown može da se završi deterministički?
12. Da li test pokriva concurrent edge cases?
13. Da li je worker count opravdan?
14. Da li je channel buffer opravdan?
15. Da li concurrency zapravo rešava problem?

---

# 83. Module 2 — Practical Design Heuristic

Kada dobiješ concurrency problem, nemoj odmah krenuti sa:

```go
go func() {}
```

Prvo definiši:

```text
1. What is the work?
2. What can execute independently?
3. What must be coordinated?
4. What is the communication topology?
5. What is the concurrency limit?
6. What is the lifecycle?
7. What is the shutdown mechanism?
8. What is the completion condition?
9. What happens under overload?
10. What happens when a consumer disappears?
```

Tek nakon toga izaberi primitives.

---

# 84. Choosing the Primitive

Koristan mentalni decision tree:

```text
Need to communicate data?
        │
        └──► Channel

Need to wait for multiple goroutines?
        │
        └──► WaitGroup

Need to wait for one of multiple events?
        │
        └──► select

Need immediate/non-blocking attempt?
        │
        └──► select + default

Need bounded concurrent processing?
        │
        └──► Worker Pool

Need staged processing?
        │
        └──► Pipeline

Need distribute work?
        │
        └──► Fan-Out

Need merge streams?
        │
        └──► Fan-In
```

Ovo nije apsolutna lista pravila, ali je odlična početna heuristika.

---

# 85. From Primitive to Architecture

Najvažnija lekcija Module 2 je prelazak sa:

```text
"Kako radi channel?"
```

na:

```text
"Kako od channels napraviti sistem?"
```

Na nivou primitive:

```go
ch <- value
```

Na nivou architecture:

```text
producer
   │
   ▼
queue
   │
   ├──► workers
   │
   ▼
results
   │
   ▼
consumer
```

To je veliki skok u concurrency reasoning-u.

---

# 86. Module 2 Knowledge Layers

Module 2 može se posmatrati kroz četiri nivoa.

## Level 1 — Syntax

Čitalac zna:

```go
select {}
wg.Add()
wg.Done()
wg.Wait()
```

---

## Level 2 — Semantics

Čitalac razume:

```text
blocking
ready
completion
closure
```

---

## Level 3 — Patterns

Čitalac zna:

```text
worker pool
pipeline
fan-out
fan-in
```

---

## Level 4 — Design

Čitalac može odlučiti:

```text
which primitive?
why?
where?
who owns lifecycle?
what happens under failure?
```

Pravi cilj modula je Level 4.

---

# 87. Senior-Level Perspective

Senior Go developer ne treba samo da zna kako da napiše worker pool.

Treba da zna kada **ne treba** da ga napiše.

Na primer:

```text
Problem:
10 jobs, each 5ms
```

Worker pool može dodati nepotrebnu kompleksnost.

Ali:

```text
Problem:
10,000,000 jobs
external API
maximum 50 concurrent requests
```

bounded worker pool ima jasnu vrednost.

Concurrency architecture treba da bude proporcionalna problemu.

---

# 88. Complexity Budget

Svaka concurrency abstraction uvodi complexity:

```text
goroutine
channel
select
WaitGroup
worker pool
pipeline
fan-out
fan-in
```

što više primitives kombinujemo, to više lifecycle state-ova imamo.

Zato cilj nije:

```text
maximum concurrency
```

nego:

```text
minimum necessary concurrency complexity
```

koja zadovoljava requirements.

---

# 89. Simplicity Rule

Ako problem može biti korektno rešen:

```go
for _, item := range items {
    process(item)
}
```

nema automatskog razloga da se uvode:

```text
goroutines
channels
worker pools
fan-in
fan-out
```

Concurrency je alat.

Nije cilj sam po sebi.

---

# 90. When Concurrency Is Justified

Concurrency je posebno opravdana kada postoji:

```text
independent work
+
waiting
+
streaming
+
throughput requirement
+
latency requirement
+
resource utilization requirement
```

Ali svaka od ovih tvrdnji treba da bude vezana za konkretan requirement.

---

# 91. Module 2 Final Mental Model

Do kraja modula mentalni model treba da izgleda ovako:

```text
                    CONCURRENCY SYSTEM
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
      Communication   Coordination     Capacity
            │              │              │
            ▼              ▼              ▼
         Channels        select       Worker Pool
                           │              │
                           ▼              ▼
                      WaitGroup      Bounded Work
            │
            ▼
      Processing Topology
            │
      ┌─────┴─────┐
      ▼           ▼
   Fan-Out      Fan-In
      │           │
      └─────┬─────┘
            ▼
         Pipeline
            │
            ▼
       Full System
```

---

# 92. Module 2 Completion Checklist

Pre prelaska na Module 3, čitalac bi trebalo da može da označi:

### `select`

* [ ] Razumem kada je case ready.
* [ ] Razumem blocking behavior.
* [ ] Razumem multiple ready cases.
* [ ] Umem da kombinujem receive i send operations.
* [ ] Umem da koristim shutdown channel.
* [ ] Razumem `select` + timeout model.

### `default`

* [ ] Razumem non-blocking receive.
* [ ] Razumem non-blocking send.
* [ ] Prepoznajem busy loops.
* [ ] Znam kada je polling opravdan.

### `WaitGroup`

* [ ] Razumem `Add`.
* [ ] Razumem `Done`.
* [ ] Razumem `Wait`.
* [ ] Umem da koordiniram više workers.
* [ ] Razumem zašto `WaitGroup` ne prenosi podatke.

### Worker Pools

* [ ] Umem da implementiram bounded worker pool.
* [ ] Razumem queue capacity.
* [ ] Razumem backpressure.
* [ ] Umem da koordiniram worker shutdown.
* [ ] Umem da zatvorim result channel na pravom mestu.

### Pipelines

* [ ] Razumem stage model.
* [ ] Umem da povežem stages.
* [ ] Razumem lifecycle output channels.
* [ ] Prepoznajem pipeline leaks.

### Fan-Out

* [ ] Razumem work distribution.
* [ ] Razumem razliku između fan-out i broadcast.
* [ ] Razumem ordering implications.

### Fan-In

* [ ] Umem da merge-ujem više streams.
* [ ] Razumem completion coordination.
* [ ] Umem da bezbedno zatvorim merged output.

### System Design

* [ ] Umem da definišem channel ownership.
* [ ] Umem da definišem goroutine lifecycle.
* [ ] Umem da identifikujem blocking points.
* [ ] Umem da identifikujem leak paths.
* [ ] Umem da identifikujem deadlock dependencies.
* [ ] Umem da objasnim zašto je određeni concurrency pattern potreban.

---

# 93. Recommended Study Order

Iako fajlovi imaju numerički redosled, preporučeni mentalni progression je:

```text
01-select
    │
    ▼
02-select-default
    │
    ▼
03-waitgroup
    │
    ▼
04-worker-pools
    │
    ▼
05-pipelines
    │
    ├──────────────┐
    ▼              ▼
06-fan-out       07-fan-in
    │              │
    └──────┬───────┘
           ▼
08-summary-and-exercises
```

Ovaj redosled prati povećanje kompleksnosti:

```text
choice
  ↓
non-blocking choice
  ↓
completion
  ↓
bounded concurrency
  ↓
composition
  ↓
distribution
  ↓
aggregation
  ↓
integration
```

---

# 94. Relation to Module 3

Module 2 se završava na nivou:

```text
coordination through channels
```

Module 3 prelazi na situacije gde channels nisu dovoljne ili nisu najbolji abstraction.

Na primer:

```text
shared mutable state
       │
       ▼
     Mutex
       │
       ▼
    RWMutex
       │
       ▼
      Once
```

zatim:

```text
timeouts
cancellation
scheduler
GOMAXPROCS
parallelism
```

Dakle:

```text
Module 1
    Communication

Module 2
    Coordination

Module 3
    Synchronization + Runtime

Module 4
    Advanced Concurrency
```

---

# 95. Module 2 Summary

Module 2 uvodi ključne mehanizme za koordinaciju goroutine-a.

`select` omogućava čekanje i izbor između više channel operations:

```go
select {
case value := <-ch1:
case value := <-ch2:
}
```

`default` omogućava non-blocking behavior:

```go
select {
case value := <-ch:
default:
}
```

`WaitGroup` omogućava koordinaciju completion-a:

```go
wg.Add(n)

go func() {
    defer wg.Done()
    work()
}()

wg.Wait()
```

Worker pool uvodi bounded concurrency:

```text
jobs
  │
  ├──► worker
  ├──► worker
  └──► worker
```

Pipeline uvodi staged processing:

```text
stage → stage → stage
```

Fan-out distribuira work:

```text
1 → N
```

Fan-in kombinuje results:

```text
N → 1
```

A kombinacija svega omogućava izgradnju ozbiljnijih concurrent systems:

```text
                 INPUT
                   │
                   ▼
                 QUEUE
                   │
             ┌─────┼─────┐
             ▼     ▼     ▼
            W1    W2    W3
             │     │     │
             └─────┼─────┘
                   ▼
                RESULTS
                   │
                   ▼
                OUTPUT
```

Najvažnija poruka Module 2 je:

> **Concurrency primitives nisu krajnji cilj. Pravi cilj je sposobnost da se od njih izgradi sistem sa jasnim communication, coordination, capacity i lifecycle pravilima.**

---

# 96. Transition to Module 3

Nakon što su communication i coordination patterns savladani, ostaje sledeće fundamentalno pitanje:

> Šta radimo kada više goroutine-a mora da pristupi istom shared state-u?

Na primer:

```text
Worker A ──┐
Worker B ──┼──► shared state
Worker C ──┘
```

Channels nisu uvek najbolji odgovor.

Tada ulazimo u:

```text
Mutex
RWMutex
Once
```

a zatim u:

```text
Timeouts
Cancellation
Go Scheduler
GOMAXPROCS
Parallelism
```

To predstavlja prirodan nastavak concurrency modela:

```text
                 Go Concurrency
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
     Communication              Coordination
          │                         │
       Module 1                  Module 2
                                    │
                                    ▼
                              Synchronization
                                    │
                                    ▼
                                Module 3
                                    │
                                    ▼
                              Advanced Model
                                    │
                                    ▼
                                Module 4
```

---

# 97. Final Module 2 Principle

Ako bi čitalac trebalo da zapamti samo jedan princip iz ovog modula, to bi bio:

> **Svaka concurrent goroutine mora imati jasan razlog za postojanje, jasan communication contract i jasan lifecycle.**

Iz tog principa proizlaze:

```text
channel ownership
channel direction
select-based coordination
WaitGroup completion
bounded worker pools
pipeline lifecycle
fan-out distribution
fan-in aggregation
backpressure
shutdown
leak prevention
```

Kada su ove stvari eksplicitno definisane, concurrency kod prestaje da bude skup nasumičnih goroutine-a i postaje **projektovan concurrent system**.

---

# Module 2 — Coordination & Concurrency Patterns

## 98. Exercises & Practical Challenges

Ovaj modul treba završiti praktičnim zadacima koji proveravaju da li čitalac zaista razume concurrency patterns, a ne samo njihovu sintaksu.

Zadaci su organizovani progresivno:

```text
Level 1
    ↓
select
    ↓
Level 2
    ↓
WaitGroup
    ↓
Level 3
    ↓
Worker Pools
    ↓
Level 4
    ↓
Pipelines
    ↓
Level 5
    ↓
Fan-Out / Fan-In
    ↓
Level 6
    ↓
Production-Style Systems
```

---

# 99. Exercises — `select`

## Exercise 1 — Two Channel Sources

Napraviti program koji čeka vrednost sa dva različita kanala:

```text
channel A
channel B
```

Program treba da ispiše sa kog kanala je vrednost stigla.

Cilj:

* `select`;
* multiple receive cases;
* nondeterministic choice kada su oba case-a ready.

---

## Exercise 2 — Multiplexing

Napraviti funkciju:

```go
func merge(a, b <-chan int) <-chan int
```

koja vraća jedan channel sa vrednostima iz oba input channel-a.

Ne koristiti `reflect.Select`.

Cilj:

* `select`;
* channel lifecycle;
* pravilno zatvaranje output channel-a.

---

## Exercise 3 — Select Loop

Napraviti worker koji konstantno čeka:

```text
job
shutdown
```

Koristiti:

```go
select
```

Worker mora završiti kada primi shutdown signal.

---

## Exercise 4 — Multiple Event Sources

Napraviti goroutine koja istovremeno reaguje na:

```text
jobs
control messages
shutdown
```

Ovaj zadatak treba da natera čitaoca da razmišlja o event-driven concurrency modelu.

---

# 100. Exercises — `select` + `default`

## Exercise 5 — Non-Blocking Receive

Napraviti funkciju koja pokušava da pročita vrednost iz channel-a, ali ne sme da blokira.

Očekivani model:

```text
value available
    ↓
return value

otherwise
    ↓
return "not ready"
```

---

## Exercise 6 — Non-Blocking Send

Napraviti funkciju:

```go
func trySend(ch chan<- int, value int) bool
```

koja vraća:

```text
true  → value accepted
false → send would block
```

---

## Exercise 7 — Avoid Busy Loop

Analizirati sledeći pattern:

```go
for {
    select {
    case value := <-ch:
        process(value)
    default:
    }
}
```

Objasniti:

1. zašto može koristiti CPU;
2. kada je takav pattern opravdan;
3. kako bi se implementacija mogla poboljšati.

Cilj nije samo napisati kod već razumeti runtime behavior.

---

# 101. Exercises — `WaitGroup`

## Exercise 8 — Parallel Tasks

Pokrenuti `N` goroutine-a i sačekati da sve završe.

Svaka goroutine treba da:

```text
sleep
perform work
print completion
```

Glavni goroutine ne sme završiti pre svih workers-a.

---

## Exercise 9 — Worker Completion

Napraviti:

```go
func runWorkers(n int)
```

koja:

1. pokreće `n` workers;
2. svaki worker obrađuje određeni posao;
3. koristi `sync.WaitGroup`;
4. vraća tek kada svi workers završe.

---

## Exercise 10 — WaitGroup Failure Analysis

Analizirati sledeći kod:

```go
var wg sync.WaitGroup

for i := 0; i < 10; i++ {
    go func() {
        defer wg.Done()
        work()
    }()
}

wg.Wait()
```

Identifikovati problem.

Objasniti zašto:

```go
wg.Add(10)
```

mora biti izvršen pre čekanja.

Cilj:

* `WaitGroup` lifecycle;
* `Add`;
* `Done`;
* `Wait`;
* coordination correctness.

---

# 102. Exercises — Worker Pool

## Exercise 11 — Basic Worker Pool

Implementirati worker pool sa:

```text
1 producer
N workers
1 consumer
```

Topology:

```text
             Jobs
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
      W1     W2     W3
       │      │      │
       └──────┼──────┘
              ▼
           Results
```

---

## Exercise 12 — Configurable Worker Count

Napraviti:

```go
func WorkerPool(
    jobs <-chan Job,
    workers int,
) <-chan Result
```

Broj workers-a treba da bude konfigurabilan.

Testirati:

```text
1 worker
2 workers
4 workers
8 workers
```

---

## Exercise 13 — Bounded Concurrency

Implementirati sistem koji garantuje:

```text
maximum N jobs concurrently
```

Broj ukupnih jobs može biti proizvoljno velik.

Cilj je razlikovati:

```text
total work
```

od:

```text
concurrent work
```

---

## Exercise 14 — Queue Capacity

Eksperimentisati sa:

```text
unbuffered channel
buffer = 1
buffer = 10
buffer = 100
```

Posmatrati:

* blocking;
* throughput;
* memory usage;
* producer behavior.

Zaključiti zašto buffer size predstavlja architectural decision, a ne samo performance tweak.

---

# 103. Exercises — Worker Pool Lifecycle

## Exercise 15 — Graceful Shutdown

Implementirati worker pool koji može primiti:

```text
jobs
shutdown
```

Worker treba da prestane da prihvata novi posao kada sistem uđe u shutdown.

---

## Exercise 16 — Drain Before Shutdown

Napraviti dva režima:

### Immediate shutdown

```text
stop accepting
stop workers
discard remaining jobs
```

### Graceful shutdown

```text
stop accepting
finish queued jobs
wait for workers
close results
```

Objasniti razliku između ova dva lifecycle modela.

---

## Exercise 17 — Shutdown While Blocked

Napraviti worker koji može biti blokiran na:

```go
jobs <- result
```

ili:

```go
job := <-jobs
```

i omogućiti shutdown koji može prekinuti čekanje.

Cilj:

* `select`;
* cancellation signal;
* goroutine lifecycle.

---

# 104. Exercises — Pipeline

## Exercise 18 — Generator Pipeline

Napraviti pipeline:

```text
Generate
   ↓
Square
   ↓
Consume
```

Primer:

```text
1 → 1
2 → 4
3 → 9
4 → 16
```

Svaki stage treba da bude zasebna goroutine.

---

## Exercise 19 — Multi-Stage Pipeline

Implementirati:

```text
Generate
   ↓
Filter
   ↓
Transform
   ↓
Validate
   ↓
Output
```

Svaki stage treba da ima jasno definisan:

```text
input
output
ownership
shutdown
```

---

## Exercise 20 — Pipeline Closure

Za svaki stage odrediti:

```text
Who closes input?
Who closes output?
When?
```

Cilj je sprečiti:

```text
deadlock
goroutine leak
send on closed channel
```

---

# 105. Exercises — Fan-Out

## Exercise 21 — Parallel Processing

Imati jedan input channel:

```text
jobs
```

i tri workers-a:

```text
W1
W2
W3
```

Svaki job treba da bude obrađen tačno jednom.

---

## Exercise 22 — Fan-Out With Ordering

Implementirati fan-out sistem koji ipak mora vratiti rezultate u originalnom redosledu.

Input:

```text
A
B
C
D
```

Workers mogu završiti:

```text
C
A
D
B
```

ali finalni output mora biti:

```text
A
B
C
D
```

Cilj je razumeti da:

```text
parallel processing
```

i:

```text
ordered output
```

nisu ista stvar.

---

# 106. Exercises — Fan-In

## Exercise 23 — Merge Multiple Workers

Imati:

```text
worker1 → channel1
worker2 → channel2
worker3 → channel3
```

Napraviti:

```go
merge(
    channel1,
    channel2,
    channel3,
)
```

koji vraća:

```text
single output channel
```

---

## Exercise 24 — Correct Fan-In Closure

Osigurati da merged channel bude zatvoren tek kada svi input channels završe.

Ne sme se dogoditi:

```text
close(results)
```

dok neki producer još može izvršiti:

```go
results <- value
```

---

# 107. Exercises — Combined Patterns

## Exercise 25 — Worker Pool + Fan-In

Implementirati:

```text
Jobs
 │
 ├──► Worker 1 ──► Results 1
 ├──► Worker 2 ──► Results 2
 └──► Worker 3 ──► Results 3
                       │
                       ▼
                     Fan-In
                       │
                       ▼
                    Consumer
```

Cilj:

* worker pool;
* fan-in;
* WaitGroup;
* channel ownership.

---

## Exercise 26 — Fan-Out + Fan-In

Napraviti sistem koji:

1. generiše jobs;
2. distribuira ih workers-ima;
3. paralelno obrađuje;
4. skuplja rezultate;
5. zatvara output tek kada svi workers završe.

---

## Exercise 27 — Pipeline + Worker Pool

Napraviti:

```text
Input
  ↓
Parse
  ↓
Worker Pool
  ↓
Transform
  ↓
Output
```

Parse i transform su pipeline stages, dok je centralna obrada bounded worker pool.

---

# 108. Exercises — Backpressure

## Exercise 28 — Producer Faster Than Consumers

Simulirati:

```text
producer = fast
workers  = slow
```

Na primer:

```text
producer: 1000 jobs/s
workers:  100 jobs/s
```

Posmatrati queue behavior.

---

## Exercise 29 — Bounded Queue

Implementirati bounded queue pomoću buffered channel-a.

Kada je queue pun, definisati jedno od ponašanja:

```text
block
reject
drop
```

Uporediti sva tri pristupa.

---

## Exercise 30 — Backpressure Propagation

Napraviti pipeline:

```text
Producer
   ↓
Stage A
   ↓
Stage B
   ↓
Slow Consumer
```

Namerno usporiti poslednji stage.

Posmatrati kako se blocking propagira prema upstream stages.

---

# 109. Exercises — Error Handling

## Exercise 31 — Result + Error

Definisati:

```go
type Result struct {
    Value any
    Err   error
}
```

Worker treba da šalje oba rezultata:

```text
success
```

ili:

```text
failure
```

Consumer treba da obradi oba slučaja.

---

## Exercise 32 — First Error

Napraviti worker pool koji završava obradu kada se pojavi prva greška.

Potrebno je definisati:

```text
error propagation
shutdown
worker termination
result cleanup
```

Ovaj zadatak je namerno složeniji jer zahteva koordinaciju više lifecycle događaja.

---

# 110. Exercises — Goroutine Leak Detection

## Exercise 33 — Find the Leak

Analizirati program u kojem jedna goroutine ostaje blokirana na send-u.

Pronaći:

```text
blocked goroutine
```

i objasniti:

```text
why it cannot finish
```

---

## Exercise 34 — Fix the Leak

Dodati lifecycle mechanism koji omogućava worker-u da napusti blocking operation.

Moguća rešenja treba analizirati kroz:

```text
select
shutdown channel
context
```

Napomena: `context` će detaljnije biti obrađen u narednim modulima.

---

# 111. Exercises — Deadlock Analysis

## Exercise 35 — Identify the Cycle

Dat je concurrency graph:

```text
Goroutine A
    │
    ▼
channel X
    │
    ▼
Goroutine B
    │
    ▼
channel Y
    │
    ▼
Goroutine A
```

Objasniti kako može nastati deadlock.

---

## Exercise 36 — Buffered vs Unbuffered

Implementirati isti communication pattern koristeći:

```text
unbuffered channel
```

zatim:

```text
buffered channel
```

Uporediti behavior.

Posebno objasniti zašto buffer može promeniti scheduling behavior bez rešavanja underlying lifecycle problema.

---

# 112. Exercises — Production Simulation

## Exercise 37 — Concurrent File Processor

Napraviti sistem koji:

1. prima listu fajlova;
2. čita ih;
3. parsira sadržaj;
4. transformiše podatke;
5. vraća rezultate.

Koristiti:

```text
worker pool
pipeline
fan-out
fan-in
WaitGroup
```

---

## Exercise 38 — Concurrent HTTP Fetcher

Napraviti HTTP fetcher koji prima veliki broj URL-ova.

Zahtevi:

```text
bounded concurrency
result collection
error handling
graceful shutdown
```

Ne dozvoliti neograničen broj istovremenih HTTP requests.

---

## Exercise 39 — Concurrent Job Processor

Implementirati servis sa sledećim modelom:

```text
                    Jobs
                     │
                     ▼
                   Queue
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Worker      Worker     Worker
          │          │          │
          └──────────┼──────────┘
                     ▼
                  Results
                     │
                     ▼
                  Storage
```

Requirements:

* configurable worker count;
* bounded queue;
* graceful shutdown;
* error propagation;
* metrics;
* no goroutine leaks.

---

# 113. Capstone Project

## Concurrent Data Processing Engine

Završni zadatak Module 2 treba da objedini sve patterns iz modula.

Napraviti engine koji prihvata veliki broj work items:

```text
Input
  │
  ▼
Producer
  │
  ▼
Bounded Queue
  │
  ▼
Worker Pool
  │
  ▼
Processing
  │
  ▼
Fan-In
  │
  ▼
Result Consumer
```

Sistem mora podržavati:

```text
+ configurable workers
+ bounded queue
+ concurrent processing
+ result aggregation
+ graceful shutdown
+ error propagation
+ completion detection
+ backpressure
```

---

# 114. Capstone Architecture

Pre implementacije nacrtati architecture diagram.

Minimalni zahtev:

```text
                    ┌───────────────┐
                    │    Producer   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   jobs chan   │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
         ┌────────┐    ┌────────┐    ┌────────┐
         │Worker 1│    │Worker 2│    │Worker N│
         └───┬────┘    └───┬────┘    └───┬────┘
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    ┌───────────────┐
                    │ results chan  │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │    Consumer    │
                    └───────────────┘
```

Na posebnom mestu treba prikazati shutdown path:

```text
Shutdown
   │
   ├──► Producer
   ├──► Worker 1
   ├──► Worker 2
   └──► Worker N
```

---

# 115. Capstone Requirements

## Functional Requirements

Sistem mora:

* prihvatati jobs;
* obrađivati jobs concurrent;
* ograničiti broj aktivnih workers-a;
* emitovati rezultate;
* propagirati errors;
* pravilno završiti processing.

## Concurrency Requirements

Sistem mora:

* koristiti goroutines;
* koristiti channels;
* koristiti `select`;
* koristiti `WaitGroup`;
* koristiti bounded worker pool;
* demonstrirati fan-out;
* demonstrirati fan-in.

## Lifecycle Requirements

Mora postojati jasan:

```text
startup
processing
shutdown
completion
```

model.

## Safety Requirements

Sistem ne sme imati:

```text
data races
send on closed channel
double close
deadlock
goroutine leaks
```

---

# 116. Capstone Review

Nakon implementacije treba odgovoriti na sledeća pitanja.

### Ownership

```text
Who owns jobs?
Who owns results?
Who closes jobs?
Who closes results?
```

### Workers

```text
How many?
Why this number?
What happens if one worker fails?
```

### Backpressure

```text
What happens when jobs queue is full?
```

### Shutdown

```text
What happens to queued jobs?
What happens to active jobs?
```

### Errors

```text
Does one error stop the system?
Or does processing continue?
```

### Completion

```text
How does the consumer know
that no more results will arrive?
```

### Leaks

```text
Can any goroutine remain blocked forever?
```

Ako ova pitanja nisu jasno odgovorena, implementation nije kompletan.

---

# 117. Performance Investigation

Capstone treba testirati sa različitim konfiguracijama:

```text
workers = 1
workers = 2
workers = 4
workers = 8
workers = 16
```

I različitim queue sizes:

```text
buffer = 0
buffer = 1
buffer = 10
buffer = 100
buffer = 1000
```

Merenja treba da obuhvate:

```text
throughput
latency
CPU
memory
allocations
```

Cilj nije pronaći univerzalno "najbolji" broj.

Cilj je naučiti da se concurrency configuration mora zasnivati na workload-u.

---

# 118. Race Detection

Concurrency exercises treba pokretati sa:

```bash
go test -race ./...
```

Ako projekat sadrži executable:

```bash
go run -race .
```

Race detector treba da bude deo normalnog development workflow-a za concurrency-heavy kod.

---

# 119. Testing Completion

Testovi treba da proveravaju ne samo rezultate već i lifecycle.

Na primer:

```text
jobs submitted
    ↓
all jobs completed
    ↓
results closed
    ↓
workers exited
```

Posebno je važno testirati:

```text
zero jobs
one job
many jobs
more workers than jobs
worker failure
shutdown during processing
full queue
empty queue
```

---

# 120. Advanced Review Challenge

Nakon svih exercises, čitalac treba da uzme bilo koji worker-pool implementation i pokuša da ga refaktoriše tako da može odgovoriti na:

```text
1. Who owns every goroutine?
2. Who owns every channel?
3. Who closes every channel?
4. What is the maximum concurrency?
5. What causes blocking?
6. What causes completion?
7. What causes shutdown?
8. What happens on error?
9. What happens on overload?
10. What prevents goroutine leaks?
```

Ako se na bilo koje pitanje ne može odgovoriti precizno, concurrency design treba dodatno analizirati.

---

# 121. What You Should Be Able to Build

Nakon završetka Module 2, čitalac bi trebalo samostalno da može implementirati:

```text
✓ basic multiplexers
✓ non-blocking channel operations
✓ coordinated goroutine groups
✓ worker pools
✓ bounded job queues
✓ processing pipelines
✓ fan-out systems
✓ fan-in systems
✓ concurrent aggregators
✓ graceful shutdown mechanisms
✓ basic backpressure systems
✓ concurrent processing engines
```

Ali još važnije, trebalo bi da može da objasni **zašto** je izabrani pattern odgovarajući.

---

# 122. What You Should Be Able to Debug

Čitalac treba da može da prepozna:

```text
✓ blocked send
✓ blocked receive
✓ forgotten channel close
✓ premature channel close
✓ multiple channel close
✓ WaitGroup misuse
✓ deadlock cycle
✓ goroutine leak
✓ unbounded concurrency
✓ uncontrolled queue growth
✓ busy loop
✓ nondeterministic ordering
✓ incorrect fan-in closure
```

Ovo predstavlja važniji praktični rezultat od samog pisanja patterns-a.

---

# 123. What Module 2 Does Not Yet Cover

Module 2 namerno ne pokušava da pokrije sve concurrency teme.

Posebno dublje teme ostaju za naredne module:

```text
Mutex
RWMutex
Once
Condition variables
Timeout architecture
Cancellation
Go scheduler
GOMAXPROCS
Parallelism
Atomic operations
Advanced synchronization
Memory model
Lock-free programming
```

Razlog je separation of concerns.

Module 2 treba da izgradi snažan mentalni model **coordination through channels and concurrency patterns** pre nego što se uvede shared-memory synchronization.

---

# 124. Module 2 — Final Knowledge Map

Celokupan modul može se sažeti ovako:

```text
                    MODULE 2
                        │
                        ▼
                  COORDINATION
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
     SELECT          WAITGROUP       CHANNELS
        │               │                │
        ▼               ▼                ▼
    choice          completion       communication
        │                                │
        ▼                                ▼
   DEFAULT                          WORKER POOL
        │                                │
        ▼                                ▼
 non-blocking                      bounded work
                                         │
                              ┌──────────┴──────────┐
                              ▼                     ▼
                           FAN-OUT                FAN-IN
                              │                     │
                              └──────────┬──────────┘
                                         ▼
                                      PIPELINE
                                         │
                                         ▼
                                   BACKPRESSURE
                                         │
                                         ▼
                                  SYSTEM DESIGN
```

---

# 125. Final Principles

### Principle 1 — Communication Has Ownership

Channels nisu samo cevi za podatke.

Oni predstavljaju communication contract.

---

### Principle 2 — Completion Is Different From Communication

To što je jedan worker završio ne znači da je ceo system završio.

Completion mora biti eksplicitno koordinisan.

---

### Principle 3 — Concurrency Should Be Bounded

Neograničena concurrency često samo pomera problem iz application layer-a u:

```text
scheduler
memory
CPU
network
database
external dependencies
```

---

### Principle 4 — Buffering Changes Behavior

Buffer može smanjiti blocking, ali ne rešava:

```text
lifecycle
backpressure
capacity mismatch
shutdown
```

---

### Principle 5 — Every Goroutine Needs an Exit Path

Za svaku dugotrajnu goroutine treba znati:

```text
How does it start?
How does it work?
How does it stop?
Who waits for it?
```

---

### Principle 6 — Closing Is a Protocol

`close(channel)` nije samo cleanup.

On predstavlja informaciju:

```text
"No more values will be sent."
```

Zato close mora biti deo dizajna communication protocol-a.

---

### Principle 7 — Patterns Are Composable

Najveća vrednost Module 2 nije pojedinačan pattern.

Vrednost je sposobnost kombinovanja:

```text
select
+
WaitGroup
+
worker pool
+
fan-out
+
fan-in
+
pipeline
```

u jedan coherent system.

---

### Principle 8 — Concurrency Is a Design Problem

Najbolji concurrency code ne počinje sa:

```go
go func() {}
```

Već sa pitanjima:

```text
What is independent?
What must communicate?
What must be synchronized?
What is the capacity?
What is the lifecycle?
What happens on failure?
What happens during shutdown?
```

Tek nakon toga dolazi izbor primitives.

---

# 126. Module 2 Completion Statement

Ako su svi prethodni concepts i exercises savladani, čitalac više ne bi trebalo da posmatra goroutines kao izolovane execution units.

Treba da ih posmatra kao komponente većeg sistema:

```text
             GOROUTINES
                  │
                  ▼
            COMMUNICATION
                  │
                  ▼
             COORDINATION
                  │
                  ▼
          CONCURRENCY PATTERNS
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Workers   Pipelines  Fan-In
        │         │         │
        └─────────┼─────────┘
                  ▼
            SYSTEM DESIGN
                  │
                  ▼
          CORRECT LIFECYCLE
                  │
                  ▼
          PRODUCTION SYSTEM
```

To je stvarni cilj ovog modula.

---

# 127. Transition to Module 3

Sa Module 1 čitalac je naučio osnovnu komunikaciju između goroutines.

Sa Module 2 naučio je kako da od tih primitives napravi koordinisane concurrency patterns.

Sledeći problem nastaje kada više goroutines ne razmenjuje samo poruke, već pristupa **istom shared state-u**:

```text
             Shared State
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Goroutine  Goroutine  Goroutine
        A         B         C
```

Tada channels nisu jedina opcija.

Module 3 uvodi:

```text
Mutex
RWMutex
Once
```

za shared-memory synchronization, a zatim prelazi na:

```text
timeouts
cancellation
Go scheduler
GOMAXPROCS
parallelism vs concurrency
```

Time se concurrency model proširuje sa:

```text
communication + coordination
```

na:

```text
synchronization + runtime behavior
```

---

# 128. Module 2 — Final Takeaway

> **Dobro projektovan concurrent system nije onaj koji koristi najviše goroutines, već onaj koji ima jasno definisanu komunikaciju, ograničenu concurrency, predvidiv lifecycle, kontrolisan shutdown i eksplicitnu koordinaciju između svih concurrent components.**

Ako čitalac razume ovu ideju, onda su:

```text
select
default
WaitGroup
worker pools
pipelines
fan-out
fan-in
```

više od pojedinačnih Go API-ja.

Postaju building blocks za projektovanje concurrent software-a.

---

## Module 2 — End

```text
Module 1
    │
    │ Communication
    ▼
Module 2
    │
    │ Coordination & Patterns
    ▼
Module 3
    │
    │ Synchronization & Runtime
    ▼
Module 4
    │
    │ Advanced Concurrency
    ▼
Go Concurrency Mastery
```