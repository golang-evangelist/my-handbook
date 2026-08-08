# Mastering Go Concurrency — Final Architecture & Best Practices

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 1)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Concurrency mentalni model
- Go concurrency filozofija
- Goroutine lifecycle
- Ownership model
- Communication vs Synchronization
- Shared State vs Message Passing
- Senior concurrency razmišljanje

---

# Uvod

Concurrency nije samo:

```go
go function()
```

---

Prava concurrency arhitektura uključuje:

- lifecycle,
- ownership,
- komunikaciju,
- failure handling,
- resource control.

---

# Go Concurrency Filozofija

Poznata Go filozofija:

> "Do not communicate by sharing memory; share memory by communicating."

---

Značenje:

Umesto:

```text
više goroutines

↓

isti memory

↓

mutex
```

preferirati:

```text
goroutine

↓

channel

↓

poruka
```

---

# Dva osnovna modela

Postoje dva pristupa.

---

# 1. Shared Memory Model

Više goroutines deli stanje.

---

Primer:

```go
counter++

```

---

Zaštita:

```go
sync.Mutex
```

---

Model:

```text
G1

 |

 ↓

Memory

 ↑

 |

G2
```

---

Prednosti:

- brz,
- jednostavan za neke slučajeve.

---

Mane:

- race condition,
- kompleksnost.

---

# 2. Message Passing Model

Goroutines komuniciraju porukama.

---

Primer:

```go
jobs <- job
```

---

Model:

```text
G1

↓

Channel

↓

G2
```

---

Prednosti:

- jasnije vlasništvo,
- manje lock-ova.

---

# Kada koristiti koji model?

## Shared State

Koristi kada:

- stanje je malo,
- pristup je jednostavan,
- performance je kritičan.

---

Primer:

```go
sync.Mutex
```

za:

- cache,
- counters,
- registries.

---

## Message Passing

Koristi kada:

- postoji tok podataka,
- postoji pipeline,
- postoji ownership.

---

Primer:

```go
worker <- job
```

---

# Goroutine Lifecycle

Svaka Goroutine mora imati:

---

# Start

Ko je kreira?

---

Primer:

```go
go worker()
```

---

Pitanje:

```
Ko je odgovoran za nju?
```

---

# Running

Šta radi?

---

Treba znati:

- input,
- output,
- state.

---

# Stop

Kako završava?

---

Primer:

```go
<-ctx.Done()
```

---

# Cleanup

Šta oslobađa?

- resources,
- connections,
- memory.

---

# Goroutine Ownership

Jedan od najvažnijih senior principa.

---

Pitanje:

> Ko poseduje ovu Goroutine?

---

Loše:

```go
go doSomething()
```

---

Niko ne zna:

- kada staje,
- ko je gasi,
- šta ako pukne.

---

Bolje:

```go
worker.Start()

worker.Stop()
```

---

Worker poseduje:

- lifecycle,
- channels,
- state.

---

# Concurrency Ownership Model

Dobar dizajn:

```text
Component A

owns:

- goroutines
- channels
- state
```

---

Drugi delovi sistema:

```
komuniciraju preko API-ja
```

---

Ne pristupaju direktno stanju.

---

# Structured Concurrency

Ideja:

> Sve Goroutines pripadaju nekom roditeljskom scope-u.

---

Primer:

```text
Request

↓

Handler

↓

Workers

↓

Cleanup
```

---

Ako request završi:

deca treba da završe.

---

Go koristi:

```go
context.Context
```

za ovaj model.

---

# Context kao Lifecycle Signal

Context prenosi:

- cancellation,
- timeout,
- deadline.

---

Primer:

```go
func process(
	ctx context.Context,
)
```

---

Svaka dublja funkcija dobija:

```go
ctx
```

---

Model:

```text
Parent Context

      |

      ↓

 Child Goroutine

      |

      ↓

 Cleanup
```

---

# Concurrency vs Parallelism

Važna razlika.

---

Concurrency:

```
više zadataka napreduje
```

---

Parallelism:

```
više zadataka izvršava se istovremeno
```

---

Primer:

Jedan CPU:

```
Concurrency
```

---

Više CPU core:

```
Parallelism
```

---

Go podržava oba.

---

# Scheduler Mental Model

Go runtime ima:

```
G

↓

M

↓

P
```

---

G:

Goroutine

---

M:

OS Thread

---

P:

Processor

---

Scheduler raspoređuje:

```text
Goroutines

↓

Threads
```

---

# Zašto ovo treba znati?

Jer:

previše Goroutines:

- memory overhead,
- scheduling overhead.

---

Premalo:

- CPU idle,
- slab throughput.

---

# Senior Decision Process

Pre nego što napišeš:

```go
go func()
```

pitaj:

---

## 1. Koji problem rešavam?

- latency?
- throughput?
- isolation?

---

## 2. Ko poseduje lifecycle?

---

## 3. Kako se gasi?

---

## 4. Kako se propagira error?

---

## 5. Šta se dešava pod overload-om?

---

# Concurrency Design Levels

## Level 1

```text
Goroutine
```

---

Početnik:

"Pokrenuo sam paralelno."

---

## Level 2

```text
Channel
```

---

Medior:

"Kontrolišem komunikaciju."

---

## Level 3

```text
Pattern
```

---

Senior:

"Biramo pravi model."

---

## Level 4

```text
Architecture
```

---

Expert:

"Dizajniram sistem koji preživljava failure."

---

# Najčešće Greške

---

## Fire-and-forget Goroutine

```go
go sendEmail()
```

---

Problem:

- leak,
- nema kontrole.

---

## Shared state bez ownership-a

---

Problem:

race condition.

---

## Nema cancellation-a

---

Problem:

beskonačne Goroutines.

---

## Nema backpressure-a

---

Problem:

memory explosion.

---

# Finalni Mentalni Model

Dobar Go concurrency sistem:

```text
Clear Ownership

        +

Controlled Lifecycle

        +

Message Passing

        +

Bounded Resources

        +

Failure Handling

        +

Observability
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ concurrency mentalni model

✅ Go concurrency filozofiju

✅ shared memory vs message passing

✅ goroutine lifecycle

✅ ownership model

✅ structured concurrency

✅ senior decision process

---

# Mastering Go Concurrency — Pattern Selection Guide

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 2)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Problem → Pattern razmišljanje
- Mutex izbor
- Channel izbor
- Worker Pool izbor
- Pipeline izbor
- Fan-out/Fan-in izbor
- Semaphore izbor
- Rate Limiter izbor
- Actor Model izbor
- Decision Matrix

---

# Osnovni Princip

Ne počinji sa:

```go
go func()
```

---

Počni sa pitanjem:

```
Koji problem pokušavam da rešim?
```

---

# Problem 1

## Više Goroutines pristupa istom stanju

Primer:

```go
counter++
```

---

Potrebno:

```
Synchronization
```

---

Izbor:

# Mutex

---

Model:

```text
G1

 |

Mutex

 |

Shared State

 |

Mutex

 |

G2
```

---

Primer:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Koristi:

- cache,
- counters,
- maps,
- registries.

---

# Mutex Decision

Koristi Mutex kada:

✅ postoji shared state

✅ operacije su kratke

✅ stanje pripada jednoj komponenti

---

Ne koristi kada:

❌ želiš pipeline

❌ želiš distribuiranu obradu

---

# Problem 2

## Goroutines treba da komuniciraju

Primer:

```text
Worker

↓

Result

↓

Another Worker
```

---

Izbor:

# Channel

---

Model:

```text
Producer

↓

Channel

↓

Consumer
```

---

Primer:

```go
results <- value
```

---

Koristi za:

- signalizaciju,
- prenos podataka,
- ownership transfer.

---

# Channel Decision

Koristi Channel kada:

✅ podaci putuju između goroutines

✅ postoji producer/consumer odnos

✅ želiš jasnu komunikaciju

---

Ne koristi kada:

❌ samo štitiš jednu promenljivu

---

# Problem 3

## Imaš mnogo poslova

Primer:

```text
100000 tasks
```

---

Ne želiš:

```text
100000 Goroutines
```

---

Izbor:

# Worker Pool

---

Model:

```text
Jobs

↓

Workers

↓

Results
```

---

Primer:

```text
10 workers

10000 jobs
```

---

Koristi za:

- batch processing,
- CPU tasks,
- API calls,
- file processing.

---

# Worker Pool Decision

Koristi kada:

✅ broj poslova je velik

✅ želiš limit concurrency

✅ svaki posao je nezavisan

---

# Problem 4

## Obrada ima više faza

Primer:

```text
Download

↓

Parse

↓

Validate

↓

Store
```

---

Izbor:

# Pipeline

---

Model:

```text
Stage 1

↓

Stage 2

↓

Stage 3
```

---

Svaka faza:

ima:

- input,
- output.

---

Koristi za:

- stream processing,
- ETL,
- data processing.

---

# Pipeline Decision

Koristi kada:

✅ postoji prirodan tok

✅ faze mogu paralelno

---

# Problem 5

## Paralelno pozivanje više servisa

Primer:

Search:

```text
Database

Cache

External API
```

---

Izbor:

# Fan-out/Fan-in

---

Model:

```text
        W1

Input   W2

        W3


          ↓


       Merge
```

---

Koristi za:

- parallel requests,
- aggregation,
- search.

---

# Problem 6

## Želiš limit aktivnih operacija

Primer:

API dozvoljava:

```
100 concurrent requests
```

---

Izbor:

# Semaphore

---

Model:

```text
Tokens:

1 2 3 4 5

↓

Workers
```

---

Koristi za:

- resource limiting,
- DB connections,
- external APIs.

---

# Problem 7

## Želiš ograničiti brzinu

Primer:

API:

```
1000 requests/min
```

---

Izbor:

# Rate Limiter

---

Model:

```text
Requests

↓

Limiter

↓

System
```

---

Koristi za:

- APIs,
- protection,
- fairness.

---

# Problem 8

## Svaki entitet ima svoje stanje

Primer:

Game server:

```
Player A

Player B

Player C
```

---

Svaki ima:

- state,
- messages.

---

Izbor:

# Actor Model

---

Model:

```text
Message

↓

Actor

↓

Private State
```

---

Koristi za:

- stateful systems,
- simulations,
- distributed processing.

---

# Problem 9

## Više servisa komunicira

Primer:

```text
Order

↓

Payment

↓

Inventory
```

---

Izbor:

# Event-driven Architecture

---

Model:

```text
Event

↓

Consumers
```

---

Koristi za:

- microservices,
- asynchronous workflows.

---

# Decision Matrix

| Problem | Pattern |
|-|-|
| Shared state | Mutex |
| Communication | Channel |
| Many jobs | Worker Pool |
| Multi-step processing | Pipeline |
| Parallel requests | Fan-out/Fan-in |
| Limit concurrency | Semaphore |
| Limit rate | Rate Limiter |
| Stateful entities | Actor |
| Service communication | Events |

---

# Kombinovanje Pattern-a

Najčešće production kombinacije:

---

## HTTP Service

```text
HTTP

↓

Rate Limiter

↓

Worker Pool

↓

Pipeline
```

---

## Data Processing

```text
Reader

↓

Pipeline

↓

Workers

↓

Writer
```

---

## Microservices

```text
API

↓

Event Bus

↓

Consumers

↓

Workers
```

---

# Anti-Pattern

## Pattern overuse

Loše:

```
Svuda Channel
```

---

Primer:

Za counter:

```go
channel <- 1
```

---

Bolje:

```go
atomic.AddInt64()
```

ili:

```go
Mutex
```

---

# Complexity Ranking

Od jednostavnog ka kompleksnom:

```
Atomic

↓

Mutex

↓

Channel

↓

Worker Pool

↓

Pipeline

↓

Distributed Events

↓

Consensus
```

---

# Senior Rule

Najjednostavniji model koji:

- rešava problem,
- ima jasan lifecycle,
- ima kontrolu resursa.

---

# Concurrency Review Checklist

Pre merge-a:

---

## Lifecycle

```
Kako se završava?
```

---

## Ownership

```
Ko poseduje state?
```

---

## Limits

```
Postoji li granica?
```

---

## Failure

```
Šta ako worker padne?
```

---

## Testing

```
Kako testirati race?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako birati concurrency pattern

✅ kada koristiti Mutex

✅ kada koristiti Channel

✅ kada koristiti Worker Pool

✅ kada koristiti Pipeline

✅ kada koristiti Fan-out/Fan-in

✅ kada koristiti Semaphore

✅ kada koristiti Event-driven model

---

# Mastering Go Concurrency — Testing Concurrent Code

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 3)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Zašto je concurrency testing težak
- Race Detector
- Deterministic Testing
- Testing Goroutines
- Testing Channels
- Testing Timeouts
- Testing Worker Pool-a
- Concurrency Test Strategije

---

# Zašto je Concurrent Testing Težak?

Sekvencijalni kod:

```text
A

↓

B

↓

C
```

---

Rezultat je predvidljiv.

---

Concurrent kod:

```text
        G1

       /


Main


       \

        G2
```

---

Redosled može biti:

```
G1 → G2

ili

G2 → G1
```

---

Test ne sme zavisiti od slučajnog rasporeda.

---

# Osnovni Princip

Loš test:

```go
time.Sleep(time.Second)
```

---

Zašto?

Zato što vreme nije garancija.

---

Bolje:

čekati signal.

---

Primer:

```go
done := make(chan bool)

go func(){

	work()

	done <- true

}()

<-done
```

---

# Race Detector

Najvažniji alat.

Go ima ugrađen:

```bash
go test -race
```

---

Detektuje:

- data race,
- konkurentni pristup memoriji,
- nekontrolisane izmene.

---

Primer problema:

```go
var counter int


go func(){
	counter++
}()

go func(){
	counter++
}()
```

---

Problem:

dve Goroutines menjaju isti podatak.

---

Race detector:

```bash
go test -race
```

---

Prikazuje:

```
WARNING: DATA RACE
```

---

# Race Detector Pravila

Koristi:

svaki put kada testiraš:

- Mutex kod,
- shared state,
- caches,
- workers.

---

Komanda:

```bash
go test -race ./...
```

---

# Deterministic Testing

Najvažniji princip:

> Test mora kontrolisati događaje.

---

Loše:

```go
time.Sleep(100*time.Millisecond)
```

---

Dobro:

```go
select {

case result := <-results:

	// verify

case <-time.After(time.Second):

	t.Fatal("timeout")

}
```

---

# Testing Goroutines

Problem:

Kako znati da je Goroutine završila?

---

Rešenje:

## WaitGroup

---

Primer:

```go
var wg sync.WaitGroup


wg.Add(1)

go func(){

	defer wg.Done()

	work()

}()


wg.Wait()
```

---

Test zna:

```
Goroutine completed
```

---

# Testing Goroutine Leak

Problem:

Goroutine ostane aktivna.

---

Primer:

```go
go worker()
```

---

Nema:

- stop,
- cancellation.

---

Test može proveriti:

```go
runtime.NumGoroutine()
```

---

Primer:

```go
before :=
	runtime.NumGoroutine()


// run test


after :=
	runtime.NumGoroutine()
```

---

Veliki rast:

mogući leak.

---

# Testing Channels

Channel test mora proveriti:

- vrednosti,
- zatvaranje,
- blokiranje.

---

Primer:

```go
func TestWorker(t *testing.T){

	ch := make(chan int)

	go func(){

		ch <- 42

		close(ch)

	}()

	value := <-ch

	if value != 42 {

		t.Fail()

	}

}
```

---

# Testing Channel Close

Važno:

Da li consumer može završiti?

---

Primer:

```go
for v := range ch {

}
```

---

Ako nema:

```go
close(ch)
```

može večno čekati.

---

Test:

```go
_, ok := <-ch

if ok {

	t.Error(
		"channel not closed",
	)

}
```

---

# Testing Timeouts

Svaki concurrent test treba imati timeout.

---

Nikada:

```go
result := <-ch
```

---

Bolje:

```go
select {

case result := <-ch:

	// success


case <-time.After(time.Second):

	t.Fatal(
		"timeout",
	)

}
```

---

Zašto?

Ako bug blokira:

test ne visi zauvek.

---

# Testing Worker Pool

Primer sistem:

```text
Jobs

↓

Workers

↓

Results
```

---

Test treba proveriti:

---

## 1. Broj obrađenih poslova

```go
expected == processed
```

---

## 2. Nema izgubljenih poslova

---

## 3. Worker shutdown

---

## 4. Error handling

---

Primer:

```go
jobs := []Job{
	1,2,3,4,5,
}

results :=
	runWorkers(jobs)

if len(results)!=5 {

	t.Fail()

}
```

---

# Testing Pipeline

Pipeline testira:

---

Input:

```text
Stage 1
```

---

Output:

```text
Final Stage
```

---

Primer:

```go
out :=
	pipeline(
		input,
	)

for result := range out {

	validate(result)

}
```

---

Proverava:

- transformaciju,
- ordering,
- shutdown.

---

# Testing Context Cancellation

Veoma važno.

---

Primer:

```go
ctx,cancel :=
	context.WithCancel(
		context.Background(),
	)

cancel()
```

---

Test:

worker treba da završi.

---

Model:

```go
select {

case <-ctx.Done():

	return

}
```

---

# Testing Retry Logic

Testirati:

---

Prvi pokušaj:

```
fail
```

---

Drugi:

```
success
```

---

Primer:

```text
Attempt 1 ❌

Attempt 2 ❌

Attempt 3 ✅
```

---

Proveriti:

- broj pokušaja,
- delay,
- finalni rezultat.

---

# Testing Concurrency Limits

Primer:

Semaphore:

```
max 5 workers
```

---

Test:

pokrenuti:

```
100 tasks
```

---

Pratiti:

```
active workers
```

---

Nikad ne sme:

```
> 5
```

---

# Fuzz Testing Concurrent Code

Go podržava:

```bash
go test -fuzz
```

---

Koristi se za:

- random input,
- edge cases.

---

Kod concurrency-ja:

korisno za:

- race scenarije,
- deadlock situacije.

---

# Test Strategije

## 1. Unit Test

Mala komponenta.

Primer:

```
worker function
```

---

## 2. Integration Test

Više komponenti.

Primer:

```
queue + workers
```

---

## 3. Stress Test

Veliko opterećenje.

Primer:

```
1M jobs
```

---

## 4. Race Test

```bash
-race
```

---

# Production Concurrency Testing

Pipeline:

```text
Unit Tests

↓

Race Detector

↓

Integration Tests

↓

Stress Tests

↓

Production Monitoring
```

---

# Najčešće Greške u Testovima

---

## Sleep Based Testing

Loše:

```go
Sleep(1s)
```

---

## Nema Timeout-a

Test može večno trajati.

---

## Ignorisanje Race Detector-a

Kod može izgledati ispravno.

---

## Testira se samo Happy Path

Ne testiraju se:

- cancellation,
- failures,
- retries.

---

# Senior Concurrency Test Checklist

Pre produkcije:

✅ `go test -race`

✅ nema goroutine leak-a

✅ svi testovi imaju timeout

✅ cancellation testiran

✅ failure scenario testiran

✅ load scenario testiran

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto je concurrency testing težak

✅ race detector

✅ deterministic testing

✅ testing goroutines

✅ testing channels

✅ testing timeouts

✅ testing worker pool-a

✅ concurrency test strategije

---

# Mastering Go Concurrency — Performance Optimization i Profiling

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 4)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Performance mentalni model
- Benchmarking concurrent koda
- CPU Profiling
- Memory Profiling
- Mutex Profiling
- Block Profiling
- Scheduler analiza
- Concurrency optimizacija

---

# Performance Mentalni Model

Concurrent sistem ima više dimenzija performansi.

---

# Throughput

Koliko posla sistem završava.

Primer:

```
jobs/sec
requests/sec
events/sec
```

---

# Latency

Koliko traje jedna operacija.

Primer:

```
request = 50ms
```

---

# Resource Usage

Koliko koristi:

- CPU,
- memory,
- goroutines,
- locks.

---

# Scalability

Kako se ponaša kada:

```
10 users

↓

10000 users
```

---

# Performance nije samo brzina

Loš dizajn može biti:

brz pod malim load-om,

ali:

```
pada pod velikim load-om
```

---

# Benchmarking Concurrent Koda

Go koristi:

```bash
go test -bench
```

---

Primer:

```go
func BenchmarkWorker(
	b *testing.B,
){

	for i:=0;i<b.N;i++{

		process()

	}

}
```

---

Pokretanje:

```bash
go test -bench=.
```

---

# Benchmark Goroutines

Primer:

```go
func BenchmarkParallel(
	b *testing.B,
){

	b.RunParallel(
		func(pb *testing.PB){

			for pb.Next(){

				work()

			}

		},
	)

}
```

---

Koristi kada testiraš:

- thread safety,
- parallel throughput.

---

# Benchmark Metrics

Rezultat:

```
1000000 ns/op
```

---

Znači:

prosečno vreme operacije.

---

Drugi parametri:

```
allocs/op
```

---

Broj alokacija.

---

Važno za:

- GC pressure,
- memory optimization.

---

# CPU Profiling

Pitanje:

> Gde CPU troši vreme?

---

Komanda:

```bash
go test \
-cpuprofile cpu.out
```

---

Analiza:

```bash
go tool pprof cpu.out
```

---

Tražimo:

- hot functions,
- expensive operations,
- unnecessary work.

---

# CPU Hotspot Primer

Profil kaže:

```
80% CPU

↓

JSON encoding
```

---

Optimizujemo:

ne:

```
goroutine tuning
```

nego:

```
serialization
```

---

# Memory Profiling

Pitanje:

> Ko troši memoriju?

---

Komanda:

```bash
go test \
-memprofile mem.out
```

---

Analiza:

```bash
go tool pprof mem.out
```

---

Tražimo:

- velike alokacije,
- memory leaks,
- objekte koji dugo žive.

---

# Goroutine Leak Profiling

Simptom:

```
memory raste

↓

goroutines rastu
```

---

Analiza:

```bash
pprof goroutine
```

---

Tražimo:

- blokirane goroutines,
- zaboravljene workere,
- channel leak.

---

# Mutex Profiling

Problem:

Previše lock contention-a.

---

Primer:

```go
mutex.Lock()

heavyWork()

mutex.Unlock()
```

---

Svi čekaju.

---

Rezultat:

```
CPU idle

Latency raste
```

---

Profil:

```bash
-mutexprofile
```

---

Tražimo:

- koji lock blokira,
- koliko dugo.

---

# Block Profiling

Pokazuje:

gde goroutines čekaju.

---

Primeri:

- channel receive,
- channel send,
- mutex,
- select.

---

Aktivacija:

```go
runtime.SetBlockProfileRate(1)
```

---

Analiza:

```bash
go tool pprof
```

---

# Scheduler Analiza

Go runtime scheduler upravlja:

```
G

M

P
```

---

Problemi:

---

## Previše Goroutines

Simptom:

```
scheduler overhead
```

---

## Premalo Goroutines

Simptom:

```
CPU idle
```

---

## Blocking Operations

Simptom:

```
thread starvation
```

---

# Go Trace

Najmoćniji alat.

---

Komanda:

```bash
go test \
-trace trace.out
```

---

Analiza:

```bash
go tool trace trace.out
```

---

Prikazuje:

- goroutine scheduling,
- blocking,
- network wait,
- GC events.

---

# Concurrency Bottleneck Primeri

---

# Problem 1

Previše Mutex-a

Simptom:

```
high contention
```

Rešenje:

- smanjiti critical section,
- koristiti atomic,
- sharding.

---

# Problem 2

Preveliki Channel Buffer

Simptom:

```
memory growth
```

Rešenje:

- bounded queue,
- backpressure.

---

# Problem 3

Premalo Workera

Simptom:

```
queue raste
```

Rešenje:

povećati concurrency.

---

# Problem 4

Previše Workera

Simptom:

```
CPU contention
```

Rešenje:

smanjiti broj.

---

# Worker Pool Tuning

Nema univerzalne vrednosti.

---

Zavisi od:

## CPU Bound

Primer:

compression.

---

Pravilo:

```
workers ≈ CPU cores
```

---

## I/O Bound

Primer:

HTTP calls.

---

Može biti:

```
više workers
```

---

# Channel Performance

Unbuffered:

```go
make(chan Job)
```

---

Sinhronizacija:

```
send

↓

receive
```

---

Buffered:

```go
make(chan Job,100)
```

---

Omogućava:

```
producer burst
```

---

Ali:

prevelik buffer:

- skriva probleme,
- povećava memory.

---

# Atomic vs Mutex

Atomic:

```go
atomic.AddInt64()
```

---

Koristi za:

- counters,
- flags.

---

Mutex:

```go
mu.Lock()
```

---

Koristi za:

- kompleksno stanje.

---

# Memory Optimization

Concurrency često povećava:

- allocations,
- GC pressure.

---

Optimizacije:

- reuse objekata,
- sync.Pool,
- smanji kopiranja,
- koristi vrednosti gde ima smisla.

---

# Production Profiling Workflow

Proces:

```text
Measure

↓

Profile

↓

Find Bottleneck

↓

Change

↓

Benchmark Again
```

---

Nikada:

```
optimize first
```

---

# Senior Performance Checklist

Pre optimizacije:

✅ postoji benchmark

✅ postoji profil

✅ poznat bottleneck

✅ promena je merena

---

# Finalni Mentalni Model

Dobar concurrent sistem:

```
Correct

↓

Observable

↓

Measured

↓

Optimized
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ performance metrike

✅ benchmark concurrency koda

✅ CPU profiling

✅ memory profiling

✅ mutex profiling

✅ block profiling

✅ scheduler analizu

✅ production optimization workflow

---

# Mastering Go Concurrency — Security i Production Pitfalls

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 5)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Concurrency Failure Model
- Deadlock analiza
- Race Condition obrasci
- Goroutine Leak prevencija
- Resource Exhaustion
- Channel problemi
- Production Anti-Patterns
- Security principi

---

# Concurrency Failure Model

Concurrent sistem može pasti na više načina:

```text
Race

↓

Deadlock

↓

Leak

↓

Resource Exhaustion

↓

System Failure
```

---

# 1. Deadlock

Deadlock znači:

> Goroutines čekaju jedna drugu zauvek.

---

Primer:

```go
mu1.Lock()

mu2.Lock()
```

Druga:

```go
mu2.Lock()

mu1.Lock()
```

---

Tok:

```text
G1

drži mu1

čeka mu2


G2

drži mu2

čeka mu1
```

---

Rezultat:

```
BLOCK FOREVER
```

---

# Deadlock Prevencija

## 1. Uvek isti redosled lock-ova

Dobro:

```text
mu1

↓

mu2
```

Svuda.

---

Loše:

```text
jedan deo:

mu1 → mu2


drugi:

mu2 → mu1
```

---

## 2. Kratak Critical Section

Loše:

```go
Lock()

networkCall()

Unlock()
```

---

Problem:

držiš lock dok čekaš mrežu.

---

Bolje:

```go
data := prepare()

Lock()

update(data)

Unlock()
```

---

## 3. Timeout mehanizmi

Za external resurse:

```go
context.WithTimeout()
```

---

# 2. Race Condition

Race nastaje kada:

više goroutines pristupa istom podatku bez zaštite.

---

Primer:

```go
balance += 100
```

---

Istovremeno:

```text
G1 read

G2 read

G1 write

G2 write
```

---

Jedna promena se izgubi.

---

# Race Prevencija

Opcije:

---

## Mutex

```go
mu.Lock()

balance++

mu.Unlock()
```

---

## Atomic

```go
atomic.AddInt64()
```

---

## Ownership

Jedna goroutine poseduje state.

---

# Ownership Model

Najsigurniji pristup:

```text
State Owner

      |

      ↓

Channel Messages
```

---

Primer:

```text
Account Actor

prima:

Deposit

Withdraw
```

---

Niko drugi direktno ne menja state.

---

# 3. Goroutine Leak

Jedan od najčešćih problema.

---

Primer:

```go
go func(){

	for {

		value := <-ch

		process(value)

	}

}()
```

---

Problem:

Ako:

```
ch nikada nije zatvoren
```

goroutine ostaje zauvek.

---

# Leak Simptomi

Vidimo:

- memory growth,
- broj goroutines raste,
- GC pritisak.

---

Monitoring:

```go
runtime.NumGoroutine()
```

---

# Goroutine Leak Prevencija

## Context Cancellation

```go
select {

case <-ctx.Done():

	return

case job := <-jobs:

	process(job)

}
```

---

## Owner Shutdown

Komponenta mora imati:

```text
Start()

Stop()
```

---

## WaitGroup

Čekanje završetka.

---

# 4. Resource Exhaustion

Concurrency može napraviti previše svega.

---

Primer:

```go
for {

	go process()

}
```

---

Rezultat:

```
milioni goroutines
```

---

Problemi:

- memory exhaustion,
- scheduler overload,
- OOM kill.

---

# Resource Limiting

Koristi:

---

## Worker Pool

Ograniči broj izvršavanja.

---

## Semaphore

Ograniči resurs.

---

## Queue Limit

Ograniči čekanje.

---

# 5. Channel Problemi

Channels su moćni, ali mogu biti opasni.

---

# Problem: Send Blocking

```go
ch <- value
```

---

Ako nema receiver-a:

```
goroutine blokirana
```

---

Rešenje:

```go
select {

case ch <- value:

case <-ctx.Done():

}
```

---

# Problem: Forgotten Close

Consumer:

```go
for v := range ch {

}
```

---

Ako producer nikada ne zatvori:

```
beskonačno čeka
```

---

# Problem: Closing Channel Ownership

Pravilo:

> Samo sender treba da zatvara channel.

---

Loše:

više goroutines:

```go
close(ch)
```

---

Može:

```
panic: close of closed channel
```

---

# 6. Panic u Goroutine

Problem:

Panic u goroutine ne sme biti ignorisan.

---

Primer:

```go
go func(){

	panic("error")

}()
```

---

Rezultat:

program može završiti.

---

Rešenje:

recover na granici:

```go
func worker(){

	defer func(){

		if r := recover(); r != nil {

		}

	}()

}
```

---

# 7. Unbounded Concurrency

Najopasniji anti-pattern.

---

Loše:

```go
for _, item := range items {

	go process(item)

}
```

---

Za:

```
1 000 000 itema
```

---

Dobijaš:

```
1 000 000 goroutines
```

---

Bolje:

```text
Queue

↓

Fixed Workers
```

---

# 8. Blocking Operations

Problem:

Goroutine radi:

```go
database.Call()
```

bez timeout-a.

---

Ako DB stane:

sve visi.

---

Obavezno:

```go
context.WithTimeout()
```

---

# Production Anti-Patterns

---

# Fire and Forget

```go
go sendEmail()
```

---

Bez:

- ownership,
- retry,
- error handling.

---

# Global Mutable State

```go
var cache map[string]string
```

---

Bez zaštite:

race.

---

# Infinite Retry

```text
fail

↓

retry

↓

fail

↓

retry forever
```

---

Rešenje:

- max attempts,
- DLQ.

---

# Huge Channel Buffer

```go
make(chan Job,10000000)
```

---

Problem:

sakriva overload.

---

# Ignoring Errors

Concurrent error može nestati.

---

Loše:

```go
go func(){

	doWork()

}()
```

---

Gde ide error?

---

# Security Perspective

Concurrency problemi mogu postati:

---

## Availability Problem

Primer:

goroutine leak.

---

## Resource Attack

Primer:

neograničeni request-i.

---

## Data Integrity Problem

Primer:

race condition.

---

# Production Concurrency Defense

Sistem treba imati:

---

## Limits

- workers,
- queues,
- connections.

---

## Timeouts

Svaki external poziv.

---

## Cancellation

Kontrola lifecycle-a.

---

## Observability

- metrics,
- logs,
- traces.

---

## Testing

- race detector,
- stress tests.

---

# Senior Checklist

Pre produkcije:

## Goroutines

✅ ko ih kreira?

✅ ko ih gasi?

---

## Channels

✅ ko šalje?

✅ ko zatvara?

---

## Locks

✅ redosled poznat?

✅ critical section mala?

---

## Resources

✅ postoje limiti?

---

## Errors

✅ gde završavaju?

---

# Finalni Mentalni Model

Siguran concurrent sistem:

```text
Bounded Resources

        +

Clear Ownership

        +

Controlled Lifecycle

        +

Failure Handling

        +

Observability
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ deadlock prevenciju

✅ race condition zaštitu

✅ goroutine leak prevenciju

✅ resource limiting

✅ channel sigurnost

✅ panic handling

✅ production anti-patterns

---

# Mastering Go Concurrency — Final Architecture Blueprint

> **Modul:** #4 — Advanced Go Concurrency
>
> **Lekcija:** 13/13 (Deo 6)
>
> **Fajl:** `docs/module-4/13-mastering-go-concurrency.md`

---

# 📚 Sadržaj

- Finalni concurrency mentalni model
- Production architecture blueprint
- Component ownership
- Lifecycle management
- Error handling
- Scalability principi
- Senior concurrency checklist
- Završni projekat Modula #4

---

# Finalni Concurrency Mentalni Model

Senior Go developer ne razmišlja:

```
Kako napraviti Goroutine?
```

---

Već:

```
Kako kontrolisati tok rada,
resurse i failure scenarije?
```

---

Concurrent sistem je kombinacija:

```
Ownership

+

Communication

+

Synchronization

+

Lifecycle

+

Failure Handling

+

Observability
```

---

# Finalna Go Concurrency Arhitektura

Production servis:

```text
                    Client

                      |

                      ↓

                 API Layer

                      |

                      ↓

              Application Service

                      |

        +-------------+-------------+

        |                           |

   Worker Pool                 Pipeline


        |                           |

        +-------------+-------------+

                      |

                  Storage


                      |

              Background Workers


                      |

              Metrics / Logs / Trace
```

---

# Component Ownership

Svaka komponenta mora posedovati:

- state,
- goroutines,
- channels,
- lifecycle.

---

Primer:

```go
type Worker struct {

	ctx context.Context

	cancel context.CancelFunc

	jobs chan Job

	wg sync.WaitGroup

}
```

---

Worker zna:

- kada startuje,
- kada staje,
- kako čisti resurse.

---

# Lifecycle Pattern

Dobar servis:

```go
Start()

↓

Run()

↓

Stop()
```

---

Primer:

```go
worker.Start()

defer worker.Stop()
```

---

Stop mora:

- zatvoriti workere,
- čekati završetak,
- osloboditi resurse.

---

# Graceful Shutdown

Production servis ne sme samo:

```text
kill process
```

---

Tok:

```text
SIGTERM

↓

Stop accepting traffic

↓

Cancel contexts

↓

Finish active jobs

↓

Close resources

↓

Exit
```

---

# Context Hierarchy

Dobar dizajn:

```text
Application Context

        |

        +---- HTTP Handler

        |

        +---- Worker

        |

        +---- Background Task
```

---

Cancellation se propagira niz stablo.

---

# Error Handling Architecture

Concurrent sistem mora imati definisan tok greške.

---

Loše:

```go
go process()
```

---

Gde ide error?

---

Bolje:

```text
Worker

↓

Error Channel

↓

Supervisor

↓

Decision
```

---

# Supervisor Pattern

Supervisor prati child komponente.

---

Model:

```text
Supervisor

     |

 +---+---+

 W1  W2  W3
```

---

Ako worker padne:

Supervisor odlučuje:

- restart,
- shutdown,
- alert.

---

# Bounded Concurrency

Jedno od najvažnijih pravila.

---

Nikada:

```go
for {

	go task()

}
```

---

Bolje:

```text
Input

↓

Queue

↓

Fixed Workers

↓

Output
```

---

Prednost:

sistem ima granice.

---

# Backpressure

Kada producer radi brže nego consumer.

---

Primer:

```
Producer:

10000/sec


Consumer:

1000/sec
```

---

Bez backpressure:

```
memory raste
```

---

Sa backpressure:

```
producer usporava
```

---

# Concurrency Pattern Map

## Simple State

Koristi:

```
Mutex
Atomic
```

---

## Communication

Koristi:

```
Channels
```

---

## Many Jobs

Koristi:

```
Worker Pool
```

---

## Multi Stage

Koristi:

```
Pipeline
```

---

## Parallel Requests

Koristi:

```
Fan-out/Fan-in
```

---

## Resource Limit

Koristi:

```
Semaphore
```

---

## Distributed Processing

Koristi:

```
Events + Queues
```

---

# Production Checklist

## Goroutines

✅ imaju owner

✅ imaju shutdown

✅ nemaju leak

---

## Channels

✅ poznat owner

✅ pravilno zatvaranje

✅ nema beskonačnog čekanja

---

## Locks

✅ mali critical section

✅ nema lock ordering problema

---

## Errors

✅ propagacija postoji

✅ retry postoji gde treba

---

## Resources

✅ ograničen broj workers

✅ bounded queues

---

## Testing

✅ unit tests

✅ integration tests

✅ race detector

---

## Observability

✅ metrics

✅ logs

✅ traces

---

# Finalni Senior Decision Framework

Kada dobiješ problem:

---

## 1. Da li mi treba concurrency?

Ako ne:

ne koristi ga.

---

## 2. Da li delim state?

Ako da:

razmisli o:

- Mutex,
- Atomic,
- Ownership.

---

## 3. Da li prenosim podatke?

Koristi:

Channel.

---

## 4. Da li imam mnogo poslova?

Koristi:

Worker Pool.

---

## 5. Da li sistem raste preko jednog procesa?

Koristi:

Distributed Patterns.

---

# Završni Projekat Modula #4

## Distributed Task Processing System

Implementirati:

---

# Core

Go service:

```text
API

↓

Task Queue

↓

Workers

↓

Storage
```

---

# Concurrency

Koristiti:

- goroutines,
- channels,
- worker pool,
- context.

---

# Reliability

Dodati:

- retry,
- timeout,
- idempotency,
- graceful shutdown.

---

# Testing

Implementirati:

- unit tests,
- race tests,
- benchmarks.

---

# Observability

Dodati:

- metrics,
- logs,
- tracing.

---

# Napredna verzija

Dodati:

- message broker,
- event-driven workflow,
- distributed workers.

---

# Putanja Znanja

Kompletan razvoj:

```text
Junior

Razume Goroutines


↓

Medior

Dizajnira Patterns


↓

Senior

Dizajnira Systems


↓

Expert

Dizajnira Distributed Architecture
```

---

# 📋 Završni Rezime Modula #4

Naučili smo:

## Osnove

✅ Goroutines

✅ Channels

✅ Select

✅ Context


## Synchronization

✅ Mutex

✅ RWMutex

✅ Atomic

✅ WaitGroup


## Patterns

✅ Worker Pool

✅ Pipeline

✅ Fan-out/Fan-in

✅ Semaphore

✅ Rate Limiter


## Advanced

✅ Scheduler

✅ Memory model

✅ Race detection

✅ Profiling


## Distributed

✅ Message Queue

✅ Event-driven systems

✅ Saga

✅ Outbox

✅ Coordination


## Production

✅ Reliability

✅ Observability

✅ Architecture

---

# 🎓 Kraj Modula #4

Kompletiran je:

```
Module #4

Advanced Go Concurrency
```

---

Stečeno znanje:

✅ Good Go Concurrency

✅ Correctness  

✅ Ownership

✅ Lifecycle

✅ Bounded Resources

✅ Failure Handling

✅ Observability

---

Sledeći nivo:

```
Module #4
```

će nadograditi ovo znanje kroz:

- Advanced Channels
- Advanced Context patterns
- Atomic operations
- Lock-free programming
- Memory synchronization
- Profiling
- Production concurrency architecture
