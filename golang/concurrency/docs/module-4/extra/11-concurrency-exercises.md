# Concurrency Exercises

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Concurrency Exercises
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Kako koristiti ove vežbe
- Nivoi težine
- Exercise metodologija
- Goroutine osnove
- Channel vežbe
- Synchronization problemi
- Debugging pristup
- Senior izazovi

---

# 1. Cilj Ovog Modula

Ovaj modul nije teorijski.

---

Cilj je:

```
primeniti concurrency koncepte

kroz praktične probleme.
```

---

Vežbe su dizajnirane da razviju:

- razumevanje Go scheduler-a
- pravilnu upotrebu goroutine-a
- channel dizajn
- synchronization obrasce
- debugging veštine

---

# 2. Kako Raditi Vežbe?

Za svaki zadatak:

## Korak 1

Razumi problem.

---

Pitaj:

```
Koji resurs deli više goroutine-a?
```

---

## Korak 2

Identifikuj:

- ownership
- lifecycle
- synchronization

---

## Korak 3

Implementiraj.

---

## Korak 4

Testiraj:

```bash
go test -race
```

---

## Korak 5

Analiziraj:

- performanse
- moguće leak-ove
- edge case scenarije

---

# 3. Nivoi Težine

Vežbe su podeljene:

---

# Level 1 — Beginner Concurrency

Fokus:

- goroutine
- channel
- WaitGroup

---

# Level 2 — Intermediate

Fokus:

- select
- cancellation
- worker pool

---

# Level 3 — Advanced

Fokus:

- pipelines
- fan-out/fan-in
- synchronization

---

# Level 4 — Senior

Fokus:

- performance
- debugging
- architecture

---

# 4. Exercise #1 — Prva Goroutine

## Cilj

Kreirati goroutine koja izvršava funkciju paralelno.

---

Zadatak:

Napiši program koji:

1. kreira goroutine
2. izvršava funkciju
3. čeka završetak

---

Primer očekivanog ponašanja:

```
main started

worker running

worker finished

main finished
```

---

## Zahtevi

Koristiti:

- goroutine
- WaitGroup

---

Ne koristiti:

```go
time.Sleep()
```

za čekanje.

---

# 5. Exercise #2 — Multiple Goroutines

## Cilj

Pokrenuti više goroutine-a.

---

Zadatak:

Kreirati:

```
10 worker goroutine-a
```

---

Svaka worker goroutine:

- dobija svoj ID
- ispisuje početak
- završava rad

---

Očekivanje:

```
worker 1 started

worker 2 started

...

worker 10 finished
```

---

Koristiti:

```go
sync.WaitGroup
```

---

# 6. Exercise #3 — Shared Counter

## Cilj

Razumeti data race.

---

Problem:

```go
counter++
```

iz više goroutine-a.

---

Zadatak:

Napraviti:

```
100 goroutine-a
```

---

Svaka:

```
povećava counter 1000 puta
```

---

Očekivanje:

```
counter == 100000
```

---

Prvo implementirati:

```
bez zaštite
```

---

Pokrenuti:

```bash
go test -race
```

---

Analizirati rezultat.

---

# 7. Exercise #4 — Mutex Counter

## Cilj

Popraviti prethodni problem.

---

Dodati:

```go
sync.Mutex
```

---

Implementirati:

```go
Increment()
```

i:

```go
Value()
```

---

Testirati:

```bash
go test -race
```

---

Očekivanje:

nema race-a.

---

# 8. Exercise #5 — Atomic Counter

## Cilj

Koristiti atomic operacije.

---

Zameniti mutex:

sa:

```go
sync/atomic
```

---

Koristiti:

```go
atomic.AddInt64()
```

---

Uporediti:

- Mutex verziju
- Atomic verziju

---

Analizirati:

- čitljivost
- performanse
- kompleksnost

---

# 9. Exercise #6 — Channel Communication

## Cilj

Naučiti osnovnu komunikaciju.

---

Napraviti:

```
producer

↓

channel

↓

consumer
```

---

Producer šalje:

```
1..10
```

---

Consumer:

sabira vrednosti.

---

Rezultat:

```
sum = 55
```

---

# 10. Exercise #7 — Channel Close

## Cilj

Razumeti zatvaranje channel-a.

---

Zadatak:

Producer treba:

- poslati podatke
- zatvoriti channel

---

Consumer koristi:

```go
range channel
```

---

Testirati:

šta se dešava ako:

```
channel nije zatvoren.
```

---

# 11. Exercise #8 — Buffered Channel

## Cilj

Razlika između:

```
buffered

vs

unbuffered
```

---

Napraviti:

```go
make(chan int, 5)
```

---

Eksperiment:

- promeniti veličinu buffera
- meriti ponašanje

---

# 12. Exercise #9 — Select

## Cilj

Koristiti:

```go
select
```

---

Napraviti dva izvora:

```
channel A

channel B
```

---

Consumer treba da primi:

```
prvi dostupan rezultat
```

---

# 13. Exercise #10 — Timeout Pattern

## Cilj

Dodati timeout.

---

Koristiti:

```go
time.After()
```

ili:

```go
context.WithTimeout()
```

---

Scenario:

```
worker traje predugo
```

---

Sistem treba:

```
prekinuti čekanje.
```

---

# 14. Exercise #11 — Goroutine Leak

## Cilj

Pronaći i popraviti leak.

---

Dato:

```go
func start() {

	go func() {

		for {
			<-ch
		}

	}()

}
```

---

Problem:

goroutine nikada ne završava.

---

Zadatak:

Dodati:

- cancellation
- cleanup

---

# 15. Senior Analiza

Za svaku vežbu odgovoriti:

```
Ko poseduje podatke?

Ko kreira goroutine?

Ko je gasi?

Kako znamo da je završila?

Šta ako dođe do greške?
```

---

# 📋 Rezime

U ovom delu urađeno:

✅ goroutine vežbe

✅ WaitGroup

✅ data race demonstracija

✅ Mutex zaštita

✅ Atomic operacije

✅ Channel komunikacija

✅ Channel close

✅ Buffered channel

✅ Select

✅ Timeout

✅ Goroutine leak analiza

---

# Concurrency Exercises

## Deo #2 — Intermediate Concurrency Exercises

---

# 📚 Sadržaj

- Worker Pool vežbe
- Pipeline implementacije
- Fan-Out / Fan-In zadaci
- Context Cancellation
- Semaphore Pattern
- Rate Limiter
- Concurrency kontrola
- Error propagation

---

# 1. Exercise #12 — Worker Pool Osnove

## Cilj

Implementirati klasičan worker pool.

---

Arhitektura:

```
Jobs

↓

Workers

↓

Results
```

---

Zadatak:

Napraviti:

```
5 worker goroutine-a
```

---

Input:

```go
jobs chan Job
```

---

Output:

```go
results chan Result
```

---

Svaki worker:

- prima posao
- obrađuje
- šalje rezultat

---

# Zahtevi

Implementirati:

```go
type Job struct {
	ID int
}
```

---

i:

```go
type Result struct {
	ID int
	Output int
}
```

---

Testirati:

```
100 jobs

5 workers
```

---

# 2. Exercise #13 — Worker Pool Shutdown

## Cilj

Pravilno gašenje worker pool-a.

---

Problem:

```
jobs channel zatvoren

workers ostaju aktivni
```

---

Zadatak:

Dodati:

- WaitGroup
- cleanup
- pravilno zatvaranje result channel-a

---

Očekivanje:

```
svi workers završeni

svi rezultati obrađeni
```

---

# 3. Exercise #14 — Worker Pool sa Context Cancellation

## Cilj

Prekid rada kada korisnik otkaže operaciju.

---

Dodati:

```go
context.Context
```

---

Worker:

```go
select {

case job := <-jobs:
	process(job)

case <-ctx.Done():
	return

}
```

---

Test:

Scenario:

```
10000 jobs

↓

cancel posle 100ms
```

---

Proveriti:

- nema leak-a
- workers završavaju

---

# 4. Exercise #15 — Pipeline Pattern

## Cilj

Implementirati concurrency pipeline.

---

Arhitektura:

```
Generator

↓

Processor

↓

Aggregator
```

---

Primer:

Generator šalje:

```
1 2 3 4 5
```

---

Processor:

```
x2
```

---

Aggregator:

```
sum
```

---

Rezultat:

```
30
```

---

# 5. Exercise #16 — Pipeline Cancellation

## Cilj

Dodati propagaciju otkazivanja.

---

Problem:

Ako downstream stane:

```
upstream nastavlja slanje
```

---

Rezultat:

```
goroutine leak
```

---

Dodati:

```go
ctx.Done()
```

na svaki stage.

---

# 6. Exercise #17 — Fan-Out Pattern

## Cilj

Paralelna obrada.

---

Arhitektura:

```
             Worker 1
           /
Input ---- Worker 2
           \
             Worker 3
```

---

Zadatak:

Jedan input channel.

Više consumer goroutine-a.

---

Meriti:

- vreme obrade
- broj workers

---

# 7. Exercise #18 — Fan-In Pattern

## Cilj

Spajanje više izvora.

---

Arhitektura:

```
Source A
        \
         Merge
        /
Source B
```

---

Implementirati:

```go
fanIn(
	ch1,
	ch2,
	ch3,
)
```

---

Rezultat:

jedan output channel.

---

# 8. Exercise #19 — Fan-Out / Fan-In Pipeline

## Cilj

Kombinovati obrasce.

---

Arhitektura:

```
Generator

↓

Fan-Out Workers

↓

Fan-In Collector

↓

Result
```

---

Scenario:

```
obrada velikog broja taskova
```

---

Analizirati:

- scalability
- memory usage
- latency

---

# 9. Exercise #20 — Semaphore Pattern

## Cilj

Ograničiti broj aktivnih goroutine-a.

---

Implementacija:

```go
sem :=
	make(chan struct{}, 3)
```

---

Zadatak:

Dozvoliti:

```
maksimalno 3 paralelna zadatka
```

---

Testirati:

```
100 taskova
```

---

# 10. Exercise #21 — Concurrent File Processing

## Cilj

Obrada velikog broja fajlova.

---

Scenario:

```
1000 fajlova

↓

workers

↓

analiza sadržaja
```

---

Zahtevi:

- ograničen broj workers
- error handling
- cleanup

---

# 11. Exercise #22 — Rate Limiter

## Cilj

Implementirati ograničenje brzine.

---

Primer:

Dozvoliti:

```
100 operacija / sekundi
```

---

Koristiti:

- ticker
- channel
- context

---

Primer:

```go
ticker :=
	time.NewTicker(
		time.Second/100,
	)
```

---

# 12. Exercise #23 — Concurrent Retry Pattern

## Cilj

Implementirati retry mehanizam.

---

Scenario:

```
request fail

↓

retry

↓

success
```

---

Dodati:

- maksimalan broj pokušaja
- backoff
- cancellation

---

# 13. Exercise #24 — Error Propagation

## Cilj

Pravilno prenositi greške.

---

Problem:

Jedna goroutine:

```
fail
```

---

Ostale:

```
nastavljaju rad
```

---

Rešenje:

koristiti:

- error channel
- context cancellation

---

Primer:

```
Worker 3 error

↓

cancel all workers
```

---

# 14. Exercise #25 — Concurrent Cache

## Cilj

Napraviti thread-safe cache.

---

Podržati:

```go
Get(key)

Set(key,value)
```

---

Implementacija:

Opcije:

- Mutex
- RWMutex
- sync.Map

---

Testirati:

```
100 readers

10 writers
```

---

# 15. Exercise #26 — Producer Consumer System

## Cilj

Implementirati klasičan obrazac.

---

Arhitektura:

```
Producer

↓

Queue

↓

Consumers
```

---

Testirati:

- brz producer
- spor consumer
- backpressure

---

# 16. Exercise #27 — Backpressure Handling

## Cilj

Kontrolisati overload.

---

Scenario:

```
producer >> consumer
```

---

Implementirati:

- buffered channel
- blocking
- dropping
- retry

---

Analizirati trade-off:

```
latency

vs

throughput
```

---

# 17. Senior Analiza

Za svaku vežbu odgovoriti:

```
Da li postoji ownership?

Ko zatvara channel?

Ko gasi goroutine?

Šta se dešava kada komponenta padne?

Postoji li backpressure?
```

---

# 📋 Rezime

U ovom delu urađeno:

✅ Worker Pool

✅ Worker shutdown

✅ Context cancellation

✅ Pipeline pattern

✅ Pipeline cancellation

✅ Fan-Out

✅ Fan-In

✅ Semaphore pattern

✅ Rate limiter

✅ Retry pattern

✅ Error propagation

✅ Concurrent cache

✅ Producer-consumer

✅ Backpressure

---

# Concurrency Exercises

## Deo #3 — Advanced Concurrency Exercises

---

# 📚 Sadržaj

- Advanced Synchronization
- Atomic algoritmi
- Lock-Free strukture
- Custom Synchronization Primitives
- Scheduler-aware problemi
- Performance tuning
- Memory visibility
- Contention analiza

---

# 1. Exercise #28 — Implementacija Thread-Safe Queue

## Cilj

Napraviti konkurentnu queue strukturu.

---

API:

```go
type Queue[T any] struct {
}
```

---

Metode:

```go
Push(value T)

Pop() (T, bool)

Size() int
```

---

Zahtevi:

Podržati:

- više producer-a
- više consumer-a

---

Implementacije:

Opcija 1:

```
Mutex + Slice
```

---

Opcija 2:

```
Channel-based queue
```

---

Test:

```
100 producers

100 consumers
```

---

# 2. Exercise #29 — Concurrent Ring Buffer

## Cilj

Implementirati bounded queue.

---

Struktura:

```
Fixed Array

+

Head pointer

+

Tail pointer
```

---

Podržati:

```go
Write()

Read()
```

---

Testirati:

- overflow
- underflow
- concurrent access

---

Analizirati:

```
lock contention
```

---

# 3. Exercise #30 — Atomic Counter sa Metrics

## Cilj

Napraviti high-performance counter.

---

Koristiti:

```go
sync/atomic
```

---

API:

```go
Increment()

Decrement()

Load()

Reset()
```

---

Test:

```
10000 goroutines

milioni operacija
```

---

Uporediti:

```
Mutex Counter

vs

Atomic Counter
```

---

Meriti:

- throughput
- latency
- CPU usage

---

# 4. Exercise #31 — Atomic State Machine

## Cilj

Implementirati stanje sistema.

---

Primer:

```go
type State int
```

---

Stanja:

```
Created

Running

Stopped

Failed
```

---

Koristiti:

```go
atomic.CompareAndSwap()
```

---

Dozvoljene tranzicije:

```
Created
   |
   v
Running
   |
   v
Stopped
```

---

Ne dozvoliti:

```
Stopped -> Running
```

---

# 5. Exercise #32 — Lock-Free Stack

## Cilj

Implementirati stack bez Mutex-a.

---

Struktura:

```
Node

↓

Next pointer
```

---

Operacije:

```go
Push()

Pop()
```

---

Koristiti:

```go
atomic.CompareAndSwapPointer()
```

---

Analizirati:

- ABA problem
- memory reclamation
- complexity

---

# 6. Exercise #33 — Read Heavy Cache

## Cilj

Optimizovati scenario:

```
mnogo čitanja

malo pisanja
```

---

Implementirati:

Varijanta 1:

```
Mutex
```

---

Varijanta 2:

```
RWMutex
```

---

Varijanta 3:

```
sync.Map
```

---

Benchmark:

```
90% reads

10% writes
```

---

# 7. Exercise #34 — Custom Once Primitive

## Cilj

Implementirati sopstveni:

```go
sync.Once
```

---

API:

```go
type Once struct {
}
```

---

Metoda:

```go
Do(func())
```

---

Zahtevi:

Funkcija se izvršava:

```
tačno jednom
```

---

Test:

```
100 goroutines

isti poziv
```

---

# 8. Exercise #35 — Custom Semaphore

## Cilj

Implementirati semaphore.

---

API:

```go
Acquire()

Release()
```

---

Primer:

Kapacitet:

```
5
```

---

Dozvoliti:

```
maksimalno 5 aktivnih operacija
```

---

Test:

```
1000 tasks
```

---

# 9. Exercise #36 — Context-aware Lock

## Cilj

Napraviti lock koji podržava:

```
timeout
```

---

API:

```go
Lock(ctx)

Unlock()
```

---

Scenario:

```
goroutine čeka lock

↓

context timeout

↓

odustaje
```

---

# 10. Exercise #37 — Deadlock Detector

## Cilj

Napraviti jednostavan alat za detekciju.

---

Pratiti:

```
ko čeka koji resurs
```

---

Model:

```
Goroutine

↓

Resource
```

---

Detektovati:

```
circular dependency
```

---

# 11. Exercise #38 — Scheduler Observation

## Cilj

Analizirati ponašanje scheduler-a.

---

Koristiti:

```go
runtime
```

---

Pratiti:

- broj goroutine-a
- GOMAXPROCS
- CPU utilization

---

Eksperiment:

Menjati:

```go
runtime.GOMAXPROCS()
```

---

Posmatrati:

- throughput
- latency

---

# 12. Exercise #39 — Work Stealing Experiment

## Cilj

Razumeti scheduler principe.

---

Napraviti:

```
više worker goroutine-a
```

---

Distribuirati:

```
različite količine posla
```

---

Analizirati:

- balansiranje
- idle workers
- throughput

---

# 13. Exercise #40 — Contention Benchmark

## Cilj

Meriti lock contention.

---

Scenario:

```
100 goroutines

isti mutex
```

---

Meriti:

- vreme čekanja
- throughput

---

Uporediti:

```
jedan globalni lock

vs

sharded locks
```

---

# 14. Exercise #41 — Sharded Counter

## Cilj

Smanjiti contention.

---

Umesto:

```
jedan counter
```

koristiti:

```
N counter shards
```

---

Primer:

```
CPU core count

↓

broj shard-ova
```

---

Final:

```
sum svih shard-ova
```

---

# 15. Exercise #42 — Memory Visibility Test

## Cilj

Razumeti:

```
happens-before
```

---

Scenario:

Jedna goroutine:

```go
data = 42
ready = true
```

---

Druga:

```go
if ready {
	print(data)
}
```

---

Eksperiment:

- bez synchronization
- sa channel-om
- sa mutex-om

---

Analizirati:

```
memory visibility
```

---

# 16. Exercise #43 — Lock Ordering Strategy

## Cilj

Sprečiti deadlock.

---

Problem:

```go
Lock(A)

Lock(B)
```

---

Druga:

```go
Lock(B)

Lock(A)
```

---

Rešenje:

Definisati:

```
globalni redosled zaključavanja
```

---

Testirati:

```
1000 paralelnih pokušaja
```

---

# 17. Exercise #44 — High Throughput Logger

## Cilj

Napraviti concurrent logger.

---

Zahtevi:

- više writer-a
- jedan disk writer
- buffering
- backpressure

---

Arhitektura:

```
Workers

↓

Channel

↓

Logger Goroutine
```

---

Analizirati:

- latency
- memory usage
- throughput

---

# 18. Senior Analiza

Kod svake vežbe analizirati:

```
Koji je synchronization model?

Koji je bottleneck?

Postoji li contention?

Može li lock biti uklonjen?

Kako scheduler utiče?
```

---

# 📋 Rezime

U ovom delu urađeno:

✅ Thread-safe Queue

✅ Ring Buffer

✅ Atomic Counter

✅ Atomic State Machine

✅ Lock-Free Stack

✅ Read-heavy Cache

✅ Custom Once

✅ Custom Semaphore

✅ Context Lock

✅ Deadlock Detector

✅ Scheduler eksperimenti

✅ Work Stealing analiza

✅ Contention Benchmark

✅ Sharded Counter

✅ Memory Visibility

✅ Lock Ordering

✅ High Throughput Logger

---

# Concurrency Exercises

## Deo #4 — Expert Concurrency Exercises

---

# 📚 Sadržaj

- Production-grade concurrency sistemi
- Fault tolerance
- Advanced cancellation
- Distributed concurrency
- Observability
- Resilience patterns
- Real-world Go architecture problemi

---

# 1. Exercise #45 — Production Worker Pool

## Cilj

Napraviti worker pool spreman za produkciju.

---

Sistem treba podržati:

- dinamičan broj taskova
- ograničen broj worker-a
- graceful shutdown
- error handling
- metrics

---

Arhitektura:

```
API

↓

Job Queue

↓

Worker Pool

↓

Result Handler
```

---

Implementirati:

```go
type WorkerPool struct {
}
```

---

Metode:

```go
Submit(job Job) error

Start()

Shutdown()

Stats()
```

---

Statistike:

```
active workers

completed jobs

failed jobs

queue size
```

---

# 2. Exercise #46 — Graceful Shutdown System

## Cilj

Implementirati kompletan shutdown lifecycle.

---

Scenario:

```
SIGTERM

↓

stop new requests

↓

finish active jobs

↓

close resources
```

---

Koristiti:

- context cancellation
- WaitGroup
- cleanup hooks

---

Testirati:

```
shutdown tokom aktivnog rada
```

---

Proveriti:

- nema izgubljenih poslova
- nema goroutine leak-a

---

# 3. Exercise #47 — Circuit Breaker Pattern

## Cilj

Implementirati zaštitu od neuspešnih servisa.

---

Stanja:

```
Closed

↓

Open

↓

Half Open
```

---

API:

```go
Call(fn)

State()
```

---

Pravila:

Ako broj grešaka pređe limit:

```
Closed → Open
```

---

Posle timeout-a:

```
Open → Half Open
```

---

# 4. Exercise #48 — Retry sa Exponential Backoff

## Cilj

Napraviti robustan retry mehanizam.

---

Podržati:

- maksimalan broj pokušaja
- exponential delay
- jitter
- cancellation

---

Primer:

```
Attempt 1

100ms

Attempt 2

200ms

Attempt 3

400ms
```

---

Testirati:

- uspeh posle retry-a
- permanent failure
- cancellation

---

# 5. Exercise #49 — Distributed Job Scheduler

## Cilj

Napraviti jednostavan scheduler.

---

Sistem:

```
Jobs

↓

Scheduler

↓

Workers
```

---

Podržati:

- delayed jobs
- retry
- cancellation
- priority

---

Primer:

```go
Schedule(
	job,
	time.Now().Add(time.Minute),
)
```

---

# 6. Exercise #50 — Rate Limited API Client

## Cilj

Napraviti concurrent API client.

---

Zahtevi:

- više request goroutine-a
- rate limit
- timeout
- retry

---

Arhitektura:

```
Requests

↓

Limiter

↓

HTTP Workers

↓

Response
```

---

Meriti:

- throughput
- latency
- error rate

---

# 7. Exercise #51 — Concurrent Batch Processor

## Cilj

Obrada velikih batch-eva.

---

Scenario:

```
1 000 000 records
```

---

Sistem:

```
Reader

↓

Workers

↓

Aggregator
```

---

Zahtevi:

- bounded memory
- backpressure
- cancellation

---

# 8. Exercise #52 — Event Processing System

## Cilj

Napraviti event-driven sistem.

---

Arhitektura:

```
Event Source

↓

Event Bus

↓

Handlers
```

---

Podržati:

- više subscriber-a
- ordering
- cancellation
- error isolation

---

# 9. Exercise #53 — Concurrent Cache sa Expiration

## Cilj

Napraviti napredni cache.

---

Podržati:

```go
Get()

Set()

Delete()

Expire()
```

---

Background goroutine:

```
cleanup expired entries
```

---

Zahtevi:

- thread-safe
- minimal contention
- graceful shutdown

---

# 10. Exercise #54 — Connection Pool

## Cilj

Implementirati resource pool.

---

Primer:

```
Database connections

↓

Pool

↓

Consumers
```

---

API:

```go
Acquire()

Release()

Close()
```

---

Testirati:

- max capacity
- timeout
- leak detection

---

# 11. Exercise #55 — Distributed Lock Simulation

## Cilj

Simulirati distributed lock.

---

Model:

```
Client A

↓

Lock Service

↓

Resource
```

---

Podržati:

- acquire
- release
- timeout
- ownership

---

Analizirati:

- race condition
- stale lock

---

# 12. Exercise #56 — Actor Model Implementation

## Cilj

Implementirati actor pristup.

---

Svaki actor:

```
state

+

mailbox

+

goroutine
```

---

API:

```go
Send(message)

Stop()
```

---

Pravila:

Actor sam upravlja stanjem.

---

# 13. Exercise #57 — Supervisor Pattern

## Cilj

Implementirati nadzor goroutine-a.

---

Supervisor:

prati:

```
child workers
```

---

Ako worker padne:

```
restart
```

---

Slično:

```
Erlang OTP supervision
```

---

# 14. Exercise #58 — Fault Injection Testing

## Cilj

Testirati otpornost sistema.

---

Namerno izazvati:

- timeout
- panic
- network failure
- worker crash

---

Proveriti:

```
da li sistem ostaje stabilan.
```

---

# 15. Exercise #59 — Observability Layer

## Cilj

Dodati metrike concurrency sistema.

---

Pratiti:

```
goroutine count

queue length

processing time

errors

latency
```

---

Implementirati:

```go
Metrics()
```

---

# 16. Exercise #60 — Production Pipeline System

## Cilj

Napraviti kompletan processing pipeline.

---

Arhitektura:

```
Input

↓

Validation

↓

Processing

↓

Storage

↓

Notification
```

---

Svaki stage:

- ima worker-e
- ima cancellation
- propagira greške

---

# 17. Expert Analiza

Za svaki sistem odgovoriti:

```
Kako sistem reaguje na failure?

Kako se vrši shutdown?

Kako se meri performansa?

Postoji li backpressure?

Kako se sprečava overload?

Kako se debug-uje problem?
```

---

# 📋 Rezime

U ovom delu urađeno:

✅ Production worker pool

✅ Graceful shutdown

✅ Circuit breaker

✅ Retry backoff

✅ Distributed scheduler

✅ Rate limited client

✅ Batch processing

✅ Event processing

✅ Expiring cache

✅ Connection pool

✅ Distributed lock

✅ Actor model

✅ Supervisor pattern

✅ Fault injection

✅ Observability

✅ Production pipeline

---

# Concurrency Exercises

## Deo #5 — Concurrency Debugging Exercises

---

# 📚 Sadržaj

- Race Condition Debugging
- Deadlock Debugging
- Goroutine Leak Analysis
- `pprof` Debugging
- `trace` Analiza
- Performance Debugging
- Scheduler Analysis
- Production Incident Simulation

---

# 1. Exercise #61 — Race Condition Detection

## Cilj

Pronaći i popraviti data race.

---

Dato:

```go
type Counter struct {
	value int
}

func (c *Counter) Increment() {
	c.value++
}
```

---

Test:

```go
for i := 0; i < 1000; i++ {

	go counter.Increment()

}
```

---

Pokrenuti:

```bash
go test -race
```

---

Zadatak:

Identifikovati:

- gde nastaje race
- zašto postoji
- kako ga ukloniti

---

Moguća rešenja:

- Mutex
- Atomic
- Channel ownership

---

# 2. Exercise #62 — Race u Map Strukturi

## Cilj

Debugovati concurrent map problem.

---

Kod:

```go
cache := map[string]string{}

go func() {

	cache["key"] = "value"

}()

go func() {

	fmt.Println(cache["key"])

}()
```

---

Zadatak:

Pronaći problem.

---

Implementirati tri rešenja:

1. Mutex

2. RWMutex

3. sync.Map

---

Uporediti:

- performanse
- kompleksnost
- čitljivost

---

# 3. Exercise #63 — Deadlock Debugging

## Cilj

Pronaći deadlock.

---

Kod:

```go
mu1.Lock()

mu2.Lock()
```

---

Druga goroutine:

```go
mu2.Lock()

mu1.Lock()
```

---

Zadatak:

Koristiti:

```bash
SIGQUIT
```

ili:

```go
runtime.Stack()
```

---

Analizirati:

```
ko čeka koji lock
```

---

Rešenje:

Definisati:

```
jedinstveni redosled zaključavanja
```

---

# 4. Exercise #64 — Channel Deadlock

## Cilj

Debugovati blokiran channel.

---

Kod:

```go
ch := make(chan int)

ch <- 10
```

---

Problem:

Nema receiver-a.

---

Zadatak:

Pronaći:

- gde se goroutine blokira
- zašto nema nastavka

---

Rešenja:

- dodati receiver
- koristiti buffered channel
- koristiti select timeout

---

# 5. Exercise #65 — Goroutine Leak Detection

## Cilj

Pronaći leak.

---

Kod:

```go
func StartWorker() {

	go func() {

		for {
			process()
		}

	}()

}
```

---

Problem:

Worker nema shutdown.

---

Zadatak:

Dodati:

- context
- cancel
- cleanup

---

Testirati:

```go
runtime.NumGoroutine()
```

---

# 6. Exercise #66 — Goroutine Stack Analysis

## Cilj

Analizirati aktivne goroutine.

---

Koristiti:

```go
runtime.Stack()
```

---

Zadatak:

Napraviti program koji:

- startuje više goroutine-a
- namerno ih blokira
- generiše dump

---

Analizirati:

- state
- blocking point
- stack trace

---

# 7. Exercise #67 — CPU Profiling

## Cilj

Pronaći CPU bottleneck.

---

Koristiti:

```bash
go test -cpuprofile cpu.out
```

---

Analizirati:

```bash
go tool pprof cpu.out
```

---

Pronaći:

- hot functions
- CPU intensive code
- contention

---

# 8. Exercise #68 — Memory Profiling

## Cilj

Pronaći memory problem.

---

Koristiti:

```bash
go test -memprofile mem.out
```

---

Analizirati:

```bash
go tool pprof mem.out
```

---

Pratiti:

- allocations
- retained memory
- leaks

---

# 9. Exercise #69 — Goroutine Profiling

## Cilj

Analizirati goroutine stanje.

---

HTTP server:

omogućiti:

```go
net/http/pprof
```

---

Endpoint:

```
/debug/pprof/goroutine
```

---

Analizirati:

- broj goroutine-a
- blokirane goroutine
- ponavljajuće stack trace-ove

---

# 10. Exercise #70 — Blocking Profile

## Cilj

Pronaći gde program čeka.

---

Omogućiti:

```go
runtime.SetBlockProfileRate()
```

---

Analizirati:

- channel blocking
- mutex waiting
- synchronization delays

---

# 11. Exercise #71 — Mutex Contention Analysis

## Cilj

Pronaći lock bottleneck.

---

Omogućiti:

```go
runtime.SetMutexProfileFraction()
```

---

Analizirati:

```
koji lock najviše blokira.
```

---

Optimizovati:

- smanjiti critical section
- koristiti RWMutex
- shardovati podatke

---

# 12. Exercise #72 — Execution Trace Analysis

## Cilj

Razumeti scheduler.

---

Generisati:

```bash
go test -trace trace.out
```

---

Analizirati:

```bash
go tool trace trace.out
```

---

Posmatrati:

- goroutine scheduling
- blocking events
- network events
- GC aktivnosti

---

# 13. Exercise #73 — Scheduler Experiment

## Cilj

Istražiti uticaj:

```go
GOMAXPROCS
```

---

Eksperiment:

```
GOMAXPROCS=1

↓

GOMAXPROCS=4

↓

GOMAXPROCS=8
```

---

Meriti:

- throughput
- latency
- CPU usage

---

# 14. Exercise #74 — Production Incident Simulation

## Cilj

Simulirati produkcioni problem.

---

Scenario:

```
API latency raste

↓

broj goroutine-a raste

↓

memory raste
```

---

Zadatak:

Koristiti:

- pprof
- trace
- logs
- metrics

---

Pronaći root cause.

---

# 15. Exercise #75 — Concurrency Performance Investigation

## Cilj

Optimizovati postojeći sistem.

---

Dato:

```
slow concurrent service
```

---

Analizirati:

- lock contention
- goroutine overhead
- channel bottleneck
- allocation rate

---

Predložiti:

- arhitektonske izmene
- optimizacije

---

# 16. Debugging Workflow

Standardni senior workflow:

```
Problem

↓

Reproduce

↓

Collect data

↓

Profile

↓

Analyze

↓

Fix

↓

Benchmark

↓

Verify
```

---

# 17. Senior Debugging Checklist

```
□ Da li postoji race?

□ Da li goroutine završavaju?

□ Gde se blokira sistem?

□ Ko drži lock?

□ Gde nastaje contention?

□ Da li postoji memory leak?

□ Kako scheduler utiče?
```

---

# 📋 Rezime

U ovom delu urađeno:

✅ Race debugging

✅ Map race analiza

✅ Deadlock debugging

✅ Channel deadlock

✅ Goroutine leak

✅ Stack dump analiza

✅ CPU profiling

✅ Memory profiling

✅ Goroutine profiling

✅ Blocking profile

✅ Mutex profile

✅ Execution trace

✅ Scheduler analiza

✅ Production incident debugging

---

# Concurrency Exercises

## Deo #6 — Final Concurrency Challenge Set

---

# 📚 Sadržaj

- Završni izazovi
- System Design zadaci
- Production Scenario
- Architecture Review
- Performance Review
- Debugging Review
- Final Checklist
- Dalji koraci

---

# 1. Final Challenge #1 — Distributed Task Processing System

## Cilj

Projektovati kompletan sistem za obradu zadataka.

---

Arhitektura:

```
               API

                │

                ▼

          Job Submission

                │

                ▼

          Buffered Queue

                │

                ▼

          Worker Pool

                │

                ▼

        Processing Pipeline

                │

                ▼

          Result Storage
```

---

Sistem mora podržati:

- graceful shutdown
- retry
- timeout
- cancellation
- metrics
- logging
- backpressure

---

# 2. Final Challenge #2 — Real-Time Event Processing

## Cilj

Napraviti sistem za obradu događaja u realnom vremenu.

---

Primer:

```
IoT uređaji

↓

Event Bus

↓

Workers

↓

Aggregation

↓

Storage

↓

Notification
```

---

Zahtevi:

- ordering
- fault tolerance
- bounded memory
- horizontal scalability

---

# 3. Final Challenge #3 — High Throughput Logger

## Cilj

Implementirati logger sposoban za veliki broj upisa.

---

Zahtevi:

- asinhrono zapisivanje
- batching
- flush interval
- graceful shutdown
- backpressure

---

Meriti:

- throughput
- latency
- memory usage

---

# 4. Final Challenge #4 — Concurrent Cache Server

## Cilj

Napraviti thread-safe cache server.

---

Podržati:

```
GET

SET

DELETE

EXPIRE

CLEAR
```

---

Implementirati:

- TTL
- cleanup worker
- metrics
- concurrent access

---

Benchmark:

```
1000 clients
```

---

# 5. Final Challenge #5 — Web Crawler

## Cilj

Napraviti konkurentni crawler.

---

Arhitektura:

```
Seed URLs

↓

Scheduler

↓

Workers

↓

HTTP Client

↓

Parser

↓

Queue
```

---

Podržati:

- maksimalan broj paralelnih zahteva
- deduplikaciju URL-ova
- timeout
- retry

---

# 6. Final Challenge #6 — Concurrent File Indexer

## Cilj

Indeksirati veliki broj fajlova.

---

Pipeline:

```
Scanner

↓

Reader

↓

Parser

↓

Indexer

↓

Storage
```

---

Optimizovati:

- memory usage
- CPU usage
- disk I/O

---

# 7. Final Challenge #7 — API Gateway Simulation

## Cilj

Simulirati API Gateway.

---

Tok:

```
Request

↓

Authentication

↓

Rate Limiter

↓

Worker Pool

↓

Backend Service

↓

Response
```

---

Implementirati:

- timeout
- retry
- circuit breaker
- metrics

---

# 8. Final Challenge #8 — Scheduler Simulator

## Cilj

Napraviti simulator raspoređivanja poslova.

---

Podržati:

- FIFO
- Priority Queue
- Round Robin

---

Meriti:

- waiting time
- turnaround time
- throughput

---

# 9. Final Challenge #9 — Monitoring Dashboard

## Cilj

Prikupiti informacije o radu sistema.

---

Prikazivati:

```
goroutine count

queue length

worker utilization

latency

errors

memory

CPU
```

---

Koristiti:

- `runtime`
- `runtime/metrics`
- `expvar`
- `pprof`

---

# 10. Final Challenge #10 — Failure Recovery

## Cilj

Sistem mora preživeti različite vrste grešaka.

---

Simulirati:

- worker panic
- timeout
- network failure
- slow consumer
- queue overflow

---

Sistem treba:

- nastaviti rad
- izolovati grešku
- prijaviti incident

---

# 11. Architecture Review Checklist

Za svaki projekat odgovoriti:

```
Ko poseduje podatke?

Ko poseduje goroutine?

Ko zatvara channel?

Ko kontroliše lifecycle?

Kako se propagiraju greške?

Kako se vrši shutdown?
```

---

# 12. Performance Review Checklist

Analizirati:

```
□ throughput

□ latency

□ CPU

□ memory

□ allocations

□ GC

□ lock contention

□ channel contention
```

---

Koristiti:

```bash
go test -bench=. -benchmem
```

---

Profilisati:

```bash
go test -cpuprofile cpu.out

go test -memprofile mem.out

go test -trace trace.out
```

---

# 13. Reliability Review Checklist

Proveriti:

```
□ nema race condition-a

□ nema deadlock-a

□ nema goroutine leak-a

□ nema memory leak-a

□ cleanup radi

□ shutdown radi

□ retry funkcioniše

□ timeout funkcioniše
```

---

# 14. Production Readiness Checklist

Sistem treba da ima:

- structured logging
- metrics
- profiling podršku
- health check
- configuration
- graceful shutdown
- observability
- testove
- benchmark-e
- dokumentaciju

---

# 15. Završni Projekat

Projektovati i implementirati kompletan servis koji koristi:

- goroutine
- channels
- `context`
- `sync`
- `sync/atomic`
- worker pool
- pipeline
- fan-out/fan-in
- rate limiter
- retry
- circuit breaker
- profiling
- benchmarking
- race detection

---

Minimalni zahtevi:

```
✔ najmanje 10 worker-a

✔ graceful shutdown

✔ metrics

✔ benchmark

✔ race-free

✔ production-ready struktura
```

---

# 16. Šta Treba Da Znaš Nakon Modula?

Po završetku ovog modula trebalo bi da možeš:

✅ dizajnirati konkurentne sisteme

✅ implementirati production-grade concurrency obrasce

✅ analizirati Go scheduler

✅ koristiti `sync` i `sync/atomic`

✅ primeniti Go Memory Model u praksi

✅ otkriti i otkloniti race condition-e

✅ sprečiti deadlock i starvation

✅ pronaći goroutine leak-ove

✅ koristiti `pprof` i `go tool trace`

✅ optimizovati throughput i latency

✅ napisati pouzdane concurrency testove

---

# 17. Završna Preporuka Za Dalje Učenje

Preporučeni redosled nakon ovog modula:

1. Ponovo rešiti sve vežbe bez gledanja rešenja.

2. Svaku vežbu pokrenuti uz:

```bash
go test -race
```

3. Napisati benchmark-e za ključne komponente.

4. Profilisati aplikaciju pomoću:

- `pprof`
- `go tool trace`

5. Implementirati jedan kompletan production-grade servis koristeći sve obrađene obrasce.

6. Analizirati izvorni kod Go runtime-a za:

- scheduler
- channels
- mutex
- garbage collector

---

# 🎓 Završetak Modula

Čestitamo!

Završio si kompletan skup vežbi za naprednu konkurentnost u Go-u.

Ove vežbe pokrivaju put od osnovnih goroutine-a i channel-a do projektovanja, testiranja, profilisanja i optimizacije production-grade konkurentnih sistema.

---

# 📋 Završni Rezime Modula

Tokom svih šest delova obradio si:

✅ 76+ praktičnih vežbi

✅ osnovne concurrency obrasce

✅ napredne synchronization tehnike

✅ worker pool, pipeline i fan-out/fan-in

✅ `context` i graceful shutdown

✅ atomic operacije i lock-free koncepte

✅ debugging i profiling

✅ performance tuning

✅ production arhitekturu

✅ završne sistemske izazove

---

### ➡️ Sledeća lekcija **[**Go Scheduler Internals**](12-go-scheduler-internals.md)**

Obuhvatiće:

- G–M–P model
- work stealing
- run queue
- scheduler loop
- preemption
- syscalls
- netpoller
- scheduler tracing
- scheduler optimizacije
- uticaj scheduler-a na performanse i dizajn konkurentnih aplikacija.