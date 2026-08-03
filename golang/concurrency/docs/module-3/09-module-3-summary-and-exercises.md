# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj

- Pregled Modula #3
- Go concurrency mentalni model
- Synchronization primitives
- Scheduler i runtime
- GOMAXPROCS
- Concurrency vs Parallelism
- Projekti
- Zadaci po nivoima
- Završni checklist

---

# Uvod

Modul #3 je bio fokusiran na:

```
Go Concurrency Internals

+

Synchronization

+

Runtime Execution Model
```

---

Cilj nije bio samo naučiti:

```go
go func()
```

---

Već razumeti:

```
Kako Go runtime izvršava konkurentne programe?
```

---

# Kompletna mapa Modula #3

```
Concurrency
    |
    |
    +-- Synchronization
    |
    +-- Scheduling
    |
    +-- Execution
    |
    +-- Performance
```

---

Detaljnije:

```
sync.Mutex

      ↓

sync.RWMutex

      ↓

sync.Once

      ↓

Timeouts

      ↓

Cancellation

      ↓

Scheduler

      ↓

GOMAXPROCS

      ↓

Parallelism vs Concurrency
```

---

# 1. Synchronization Primitives

## sync.Mutex

Koristi se za:

```
zaštitu deljenog stanja
```

---

Primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Mentalni model:

```
jedan vlasnik u trenutku
```

---

---

## sync.RWMutex

Omogućava:

```
više čitalaca

jedan pisac
```

---

Model:

```
Readers

↓

Shared access


Writer

↓

Exclusive access
```

---

Koristi se kod:

- cache,
- konfiguracije,
- read-heavy struktura.

---

---

## sync.Once

Garantuje:

```
izvrši jednom
```

---

Primer:

```go
once.Do(initConfig)
```

---

Tipične upotrebe:

- singleton inicijalizacija,
- lazy loading,
- globalni resursi.

---

# 2. Cancellation i Timeout

Moderni Go sistemi moraju znati:

```
kada stati
```

---

## Timeout

Znači:

```
ograničeno vreme čekanja
```

---

Primer:

```go
context.WithTimeout()
```

---

---

## Cancellation

Znači:

```
prekid rada koji više nije potreban
```

---

Primer:

HTTP request završen:

```
prekini database query
```

---

Model:

```
Parent

↓

Context

↓

Child Goroutines
```

---

# 3. Go Scheduler

Go runtime koristi:

```
GMP Model
```

---

G:

```
Goroutine
```

---

M:

```
Machine

(OS Thread)
```

---

P:

```
Processor

(execution resource)
```

---

Model:

```
G

↓

P

↓

M

↓

CPU
```

---

Scheduler rešava:

```
koja Goroutine se izvršava
```

---

# 4. GOMAXPROCS

GOMAXPROCS određuje:

```
broj aktivnih P objekata
```

---

Ne znači:

```
broj Goroutines
```

---

Ne znači:

```
broj Threads
```

---

Znači:

```
koliko Goroutines može paralelno koristiti CPU resurse
```

---

Primer:

```
GOMAXPROCS = 4
```

---

Moguće:

```
4 aktivna execution slot-a
```

---

# 5. Concurrency vs Parallelism

Najvažnija razlika:

---

## Concurrency

```
više poslova je u toku
```

---

Primer:

```
1000 HTTP request-a
```

---

---

## Parallelism

```
više poslova se izvršava istovremeno
```

---

Primer:

```
4 CPU core-a
```

---

---

Kratko:

```
Concurrency

=

organization


Parallelism

=

execution
```

---

# Go Runtime Mentalni Model

Cela slika:

```
Application

    |

Goroutines

    |

Scheduler

    |

GMP

    |

OS Threads

    |

CPU
```

---

Dodajemo:

```
Channels

Mutex

Context

Atomic

WaitGroup
```

---

Za kontrolu:

```
komunikacije

+

sinhronizacije
```

---

# Najvažniji principi

---

## Princip #1

Ne pravi Goroutines bez kontrole.

---

Loše:

```go
for {

	go process()

}
```

---

Bolje:

```
worker pool
```

---

---

## Princip #2

Izbegavaj nepotrebno deljeno stanje.

---

Bolje:

```
message passing
```

nego:

```
shared mutable state
```

---

---

## Princip #3

Meri pre optimizacije.

---

Koristi:

```
benchmark

pprof

trace
```

---

---

## Princip #4

Concurrency povećava kompleksnost.

---

Više Goroutines:

```
više mogućnosti

+

više problema
```

---

# Mini projekti

---

# Projekat #1 — Concurrent HTTP Client

Napraviti:

```
URL fetcher
```

---

Zahtevi:

- Goroutines,
- Channels,
- Timeout,
- Cancellation.

---

Cilj:

naučiti:

```
IO concurrency
```

---

# Projekat #2 — Worker Pool

Napraviti:

```
Job queue

+

Workers

+

Results
```

---

Koristiti:

- Channels,
- WaitGroup,
- Context.

---

Cilj:

```
kontrolisan concurrency
```

---

# Projekat #3 — Thread-safe Cache

Napraviti:

```
In-memory cache
```

---

Koristiti:

- RWMutex,
- TTL,
- Cleanup goroutine.

---

Cilj:

```
shared state synchronization
```

---

# Projekat #4 — Parallel Image Processor

Napraviti:

```
Image pipeline
```

---

Koristiti:

- Worker pool,
- GOMAXPROCS,
- Benchmark.

---

Cilj:

```
CPU parallelism
```

---

# 📋 Rezime

U ovom delu smo:

✅ povezali sve Module #3 koncepte  
✅ napravili kompletan runtime mentalni model  
✅ ponovili synchronization primitives  
✅ povezali Scheduler i GOMAXPROCS  
✅ definisali concurrency vs parallelism razliku  
✅ pripremili osnovu za praktične projekte  

---

# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj ovog dela

- Kompletan Go runtime model
- Lifecycle jedne Goroutine
- Od `go func()` do CPU izvršavanja
- GMP model u praksi
- Scheduler odluke
- Blocking i prebacivanje izvršavanja
- Veza sa synchronization alatima

---

# Uvod

Da bismo razumeli concurrency na višem nivou, potrebno je povezati sve delove.

Jedan mali primer:

```go
go process()
```

izgleda jednostavno.

---

Ali iza njega se dešava veliki broj runtime operacija:

```
Kreiranje Goroutine

↓

Dodavanje scheduler-u

↓

Dodela Processor-a (P)

↓

Mapiranje na Thread (M)

↓

CPU izvršavanje
```

---

# Lifecycle jedne Goroutine

Goroutine prolazi kroz nekoliko faza.

---

## 1. Kreiranje

Primer:

```go
go worker()
```

---

Go runtime kreira:

```
G strukturu
```

koja predstavlja Goroutine.

---

G sadrži informacije:

- stack,
- instruction pointer,
- stanje izvršavanja,
- reference.

---

Početni stack je mali.

Za razliku od OS thread-a:

```
Goroutine stack

≈ nekoliko KB
```

---

# 2. Dodavanje Scheduler-u

Nakon kreiranja:

Goroutine nije odmah na CPU-u.

---

Ona ide u:

```
scheduler queue
```

---

Model:

```
New Goroutine

        ↓

Runnable queue
```

---

Znači:

```
spremna za izvršavanje
```

---

# 3. Dodela Processor-a (P)

Scheduler bira:

```
koji P će izvršavati Goroutine
```

---

Podsećanje:

```
P = execution context
```

---

GOMAXPROCS određuje:

```
koliko P postoji
```

---

Primer:

```text
GOMAXPROCS = 4
```

---

Runtime ima:

```
P0

P1

P2

P3
```

---

---

# 4. Povezivanje sa Machine (M)

P zatim koristi:

```
M
```

---

M predstavlja:

```
OS Thread
```

---

Veza:

```
G

↓

P

↓

M

↓

CPU
```

---

# 5. CPU izvršavanje

Kada scheduler odluči:

Goroutine prelazi u:

```
Running
```

---

Primer:

```go
func worker(){

	fmt.Println("work")

}
```

---

CPU izvršava instrukcije funkcije.

---

# Stanja Goroutine

Goroutine se najčešće nalazi u jednom od stanja:

```
Runnable

Running

Waiting
```

---

# Runnable

Znači:

```
spremna za CPU
```

---

Primer:

```
G1 čeka

G2 trenutno radi
```

---

G1:

```
Runnable
```

---

# Running

Znači:

```
trenutno izvršavanje
```

---

Primer:

```
CPU

↓

G2
```

---

# Waiting

Znači:

```
blokirana
```

---

Primeri:

- channel receive,
- mutex lock,
- network IO,
- timer.

---

Primer:

```go
value := <-ch
```

---

Ako nema podatka:

```
Goroutine čeka
```

---

# Scheduler prebacivanje

Go scheduler često radi:

```
G1

↓

G2

↓

G3
```

---

Razlozi:

- Goroutine blokira,
- previše vremena koristi CPU,
- runtime balansira posao.

---

# Cooperative + Preemptive Scheduling

Moderni Go scheduler koristi:

```
preemption
```

---

Znači:

runtime može prekinuti Goroutine.

---

Primer:

Jedna Goroutine:

```go
for {

}
```

---

Bez preemption-a:

mogla bi blokirati druge.

---

Sa preemption-om:

scheduler može reći:

```
dosta, izvrši nekog drugog
```

---

# Blocking primer

Kod:

```go
func worker(ch chan int){

	x := <-ch

	fmt.Println(x)

}
```

---

Ako nema vrednosti:

```
Goroutine blokira
```

---

Šta radi scheduler?

Ne drži CPU besposlenim.

---

Umesto:

```
CPU čeka
```

radi:

```
druga Goroutine
```

---

# Mutex primer

Kod:

```go
mu.Lock()

criticalSection()

mu.Unlock()
```

---

Ako je lock zauzet:

Goroutine ide u:

```
Waiting
```

---

Scheduler bira:

```
drugu Goroutine
```

---

# Channel primer

Kod:

```go
ch <- data
```

---

Ako nema primaoca:

slanje može blokirati.

---

Stanje:

```
Running

↓

Waiting
```

---

Kada druga strana primi:

```
Waiting

↓

Runnable
```

---

# Veza sa Context Cancellation

Primer:

```go
select {

case <-ctx.Done():

	return

case data := <-ch:

	process(data)

}
```

---

Goroutine ima mogućnost:

```
kontrolisanog završetka
```

---

Bez toga:

može ostati:

```
leaked Goroutine
```

---

# Goroutine Leak

Primer:

```go
func worker(){

	select{}

}
```

---

Ova Goroutine:

nikada ne završava.

---

Problem:

- zauzima memoriju,
- povećava runtime overhead,
- može degradirati servis.

---

Rešenje:

uvek planirati:

```
start

↓

work

↓

stop
```

---

# Kako se sve povezuje?

Cela slika:

```
Application Code

        |

        v

go func()

        |

        v

Goroutine (G)

        |

        v

Scheduler

        |

        v

Processor (P)

        |

        v

Machine (M)

        |

        v

CPU
```

---

A kada postoji sinhronizacija:

```
Mutex

Channel

Context

Atomic

WaitGroup
```

kontrolišu:

```
kretanje i koordinaciju Goroutines
```

---

# Senior perspektiva

Kada vidiš:

```go
go someFunction()
```

nemoj razmišljati:

"kreirao sam thread".

---

Razmišljaj:

```
Kreirao sam task

koji će scheduler rasporediti

kada resursi budu dostupni.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ lifecycle jedne Goroutine  
✅ kreiranje G strukture  
✅ scheduler queue  
✅ vezu G-P-M modela  
✅ stanje Runnable/Running/Waiting  
✅ blocking ponašanje  
✅ scheduler prebacivanje  
✅ goroutine leak problem  

---

# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj ovog dela

- Go concurrency patterns pregled
- Worker Pool pattern
- Pipeline pattern
- Fan-Out pattern
- Fan-In pattern
- Semaphore pattern
- Rate Limiting pattern
- Kada koristiti koji pattern

---

# Uvod

Do sada smo naučili:

```
Goroutines

↓

Scheduler

↓

Synchronization

↓

Runtime Execution
```

---

Sada prelazimo na praktične arhitektonske obrasce.

---

Concurrency pattern predstavlja:

> Ponovljivo rešenje za organizaciju više konkurentnih izvršavanja.

---

Najčešći Go concurrency pattern-i:

```
Worker Pool

Pipeline

Fan-Out

Fan-In

Semaphore

Rate Limiting
```

---

# 1. Worker Pool Pattern

## Problem

Imamo veliki broj poslova:

```
100000 tasks
```

---

Ne želimo:

```
100000 Goroutines
```

---

Potrebna je kontrola.

---

Rešenje:

```
ograničen broj worker-a
```

---

Model:

```
Jobs

 |

 v

Workers

 |

 v

Results
```

---

# Osnovna implementacija

```go
jobs := make(chan int)

results := make(chan int)
```

---

Worker:

```go
func worker(
	jobs <-chan int,
	results chan<- int,
) {

	for job := range jobs {

		results <- job * 2

	}

}
```

---

Pokretanje:

```go
for i := 0; i < 4; i++ {

	go worker(
		jobs,
		results,
	)

}
```

---

Imamo:

```
4 worker Goroutines
```

---

# Kada koristiti Worker Pool?

Odlično za:

- CPU processing,
- batch obradu,
- kontrolisani concurrency,
- ograničenje resursa.

---

Primeri:

- resize slika,
- generisanje report-a,
- obrada fajlova.

---

# CPU Worker Pool

Za CPU-bound posao:

```
workers ≈ GOMAXPROCS
```

---

Primer:

```
CPU cores = 8

workers = 8
```

---

Zašto?

Jer:

```
više worker-a

ne znači

više CPU kapaciteta
```

---

# IO Worker Pool

Za IO-bound posao:

može biti:

```
više worker-a
```

---

Primer:

```
HTTP downloader
```

---

Jer većina vremena:

```
worker čeka
```

---

# 2. Pipeline Pattern

Pipeline deli obradu u faze.

---

Model:

```
Stage 1

↓

Stage 2

↓

Stage 3
```

---

Svaki stage:

- prima podatke,
- obrađuje,
- šalje dalje.

---

# Primer

Obrada brojeva:

```
Generate

↓

Square

↓

Print
```

---

Generator:

```go
func generate() <-chan int {

	out := make(chan int)

	go func(){

		for i := 0; i < 10; i++ {

			out <- i

		}

		close(out)

	}()

	return out

}
```

---

Processing stage:

```go
func square(
	in <-chan int,
) <-chan int {

	out := make(chan int)

	go func(){

		for n := range in {

			out <- n*n

		}

		close(out)

	}()

	return out

}
```

---

Upotreba:

```go
numbers := generate()

squares := square(numbers)
```

---

Dobijamo:

```
stream processing
```

---

# Kada koristiti Pipeline?

Idealno za:

- streaming,
- data processing,
- ETL sisteme,
- event obradu.

---

Primer:

```
Read file

↓

Parse

↓

Validate

↓

Store
```

---

# 3. Fan-Out Pattern

Fan-Out znači:

jedan izvor:

```
više worker-a
```

---

Model:

```
          Worker 1

Input ---- Worker 2

          Worker 3
```

---

Primer:

Jedan channel:

```go
jobs
```

---

Više worker-a:

```go
go worker(jobs)

go worker(jobs)

go worker(jobs)
```

---

Svi čitaju:

```
isti input
```

---

# Zašto Fan-Out?

Povećava:

```
throughput
```

---

Primer:

Jedan parser:

```
1000 records/sec
```

---

Sa 4 worker-a:

```
4000 records/sec
```

(moguće, zavisi od workload-a)

---

# 4. Fan-In Pattern

Fan-In je suprotno.

---

Više izvora:

```
Input 1

Input 2

Input 3
```

---

Jedan izlaz:

```
Output
```

---

Model:

```
Source1 \
Source2 ---> Merge ---> Output
Source3 /
```

---

Primer:

```go
func merge(
	channels ...<-chan int,
) <-chan int
```

---

Koristi se kada:

želimo:

```
kombinovati rezultate
```

---

Primer:

```
3 API poziva

↓

jedan rezultat
```

---

# Fan-Out + Fan-In

Često idu zajedno.

---

Model:

```
          Worker1
         /
Input --- Worker2 ---- Merge ---- Result
         \
          Worker3
```

---

Primer:

MapReduce stil:

```
Map

↓

Process

↓

Reduce
```

---

# 5. Semaphore Pattern

Semaphore ograničava:

```
koliko operacija sme biti aktivno
```

---

Primer:

Imamo:

```
10000 HTTP request-a
```

---

Ali želimo:

```
100 aktivnih
```

---

Implementacija:

```go
sem := make(chan struct{}, 100)
```

---

Pre rada:

```go
sem <- struct{}{}
```

---

Posle:

```go
<-sem
```

---

Rezultat:

maksimalno:

```
100 concurrent operacija
```

---

# Kada koristiti Semaphore?

Za:

- API limite,
- database konekcije,
- external services,
- memory zaštitu.

---

# 6. Rate Limiting Pattern

Rate limiter kontroliše:

```
koliko zahteva po vremenu
```

---

Primer:

```
100 requests/sec
```

---

Za razliku od Semaphore:

Semaphore:

```
koliko istovremeno
```

---

Rate Limiter:

```
koliko u vremenu
```

---

Primer:

```
10 req/s
```

---

Koristi se za:

- API protection,
- throttling,
- fairness.

---

# Poređenje pattern-a

| Pattern | Problem koji rešava |
|---|---|
| Worker Pool | kontrola broja worker-a |
| Pipeline | obrada kroz faze |
| Fan-Out | povećanje obrade |
| Fan-In | spajanje rezultata |
| Semaphore | ograničenje paralelnih operacija |
| Rate Limiter | ograničenje brzine |

---

# Kako izabrati pattern?

---

## Mnogo nezavisnih poslova

Koristi:

```
Worker Pool
```

---

## Obrada kroz korake

Koristi:

```
Pipeline
```

---

## Jedan input, više procesora

Koristi:

```
Fan-Out
```

---

## Više izvora, jedan rezultat

Koristi:

```
Fan-In
```

---

## Ograničeni resursi

Koristi:

```
Semaphore
```

---

## Zaštita od preopterećenja

Koristi:

```
Rate Limiter
```

---

# Senior mentalni model

Concurrency pattern nije cilj sam po sebi.

Ne pitaš:

> "Koji pattern da koristim?"

---

Prvo pitaš:

```
Koji problem imam?
```

---

Zatim:

```
Koji pattern rešava taj problem?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Worker Pool  
✅ Pipeline  
✅ Fan-Out  
✅ Fan-In  
✅ Semaphore  
✅ Rate Limiting  
✅ kada koristiti svaki pattern  
✅ kako kombinovati pattern-e  

---

# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj ovog dela

- Praktični projekti za Modul #3
- Projekat #1 — Concurrent HTTP Client
- Projekat #2 — Worker Pool System
- Projekat #3 — Thread-safe Cache
- Projekat #4 — Parallel Data Processor
- Projekat #5 — Concurrent Pipeline
- Arhitektonski principi
- Testiranje concurrent sistema

---

# Uvod

Teorija concurrency-ja postaje korisna tek kada možemo da je primenimo u realnim sistemima.

U ovom delu povezujemo:

```
Goroutines

+

Channels

+

Synchronization

+

Context

+

Scheduler

+

Performance
```

---

# Projekat #1 — Concurrent HTTP Client

## Cilj

Napraviti klijenta koji paralelno poziva veliki broj HTTP endpoint-a.

---

## Scenario

Imamo:

```
1000 URL-ova
```

---

Potrebno:

- poslati zahteve,
- obraditi rezultate,
- ograničiti broj aktivnih request-a,
- podržati cancellation.

---

# Arhitektura

```
URLs

 |

 v

Worker Pool

 |

 v

HTTP Requests

 |

 v

Results
```

---

# Zahtevi

Koristiti:

- Goroutines,
- Channels,
- Context,
- Semaphore.

---

Primer toka:

```
main()

↓

create context

↓

start workers

↓

send jobs

↓

collect results
```

---

# Problemi koje rešava

Uči:

- IO concurrency,
- kontrolu resursa,
- timeout,
- graceful shutdown.

---

# Projekat #2 — Worker Pool System

## Cilj

Implementirati generički sistem za obradu poslova.

---

Model:

```
Producer

    |

    v

Jobs Channel

    |

    v

Workers

    |

    v

Results
```

---

# Primer

Input:

```text
job1
job2
job3
```

---

Workers:

```
Worker 1

Worker 2

Worker 3
```

---

Output:

```text
result1
result2
result3
```

---

# Zahtevi

Implementirati:

- configurable broj worker-a,
- graceful shutdown,
- error handling,
- cancellation.

---

# Naučeni koncepti

Koristi:

```
Worker Pool

Fan-Out

Fan-In
```

---

# Projekat #3 — Thread-safe Cache

## Cilj

Napraviti in-memory cache koji podržava concurrent pristup.

---

Primer:

```go
cache.Set(
	"user:1",
	value,
)
```

---

i:

```go
cache.Get(
	"user:1",
)
```

---

# Interna struktura

Primer:

```go
type Cache struct {

	mu sync.RWMutex

	data map[string]interface{}

}
```

---

# Operacije

## Read

Više čitalaca:

```
RLock()
```

---

## Write

Jedan pisac:

```
Lock()
```

---

# Dodatni zahtevi

Dodati:

- TTL expiration,
- cleanup goroutine,
- metrics.

---

# Naučeni koncepti

Koristi:

```
RWMutex

Goroutines

Timers

Cancellation
```

---

# Projekat #4 — Parallel Data Processor

## Cilj

Napraviti sistem za CPU-intensive obradu.

---

Primer:

Input:

```
1 000 000 brojeva
```

---

Operacija:

```
transformacija

filter

agregacija
```

---

# Arhitektura

```
Input

 |

Fan-Out

 |

Workers

 |

Fan-In

 |

Result
```

---

# Worker broj

Testirati:

```
workers = 1

workers = 2

workers = 4

workers = 8
```

---

Meriti:

- vreme,
- CPU usage,
- throughput.

---

# Benchmark

Koristiti:

```bash
go test -bench .
```

---

Analizirati:

```
pre

vs

posle
```

---

# Projekat #5 — Concurrent Pipeline

## Cilj

Napraviti realni data processing pipeline.

---

Primer:

```
Read Files

↓

Parse Data

↓

Validate

↓

Transform

↓

Save
```

---

Svaka faza:

```
posebna Goroutine
```

---

Komunikacija:

```
Channels
```

---

# Zahtevi

Dodati:

- cancellation,
- error propagation,
- backpressure.

---

# Backpressure

Problem:

Brzi producer:

```
10000 events/sec
```

---

Spor consumer:

```
100 events/sec
```

---

Rešenje:

kontrola toka.

---

Primer:

buffered channel:

```go
make(chan Event, 100)
```

---

# Testiranje Concurrent Koda

Concurrent kod zahteva posebne testove.

---

# 1. Race Detector

Najvažniji alat:

```bash
go test -race ./...
```

---

Otkriva:

- data races,
- unsafe memory access.

---

# 2. Stress Testing

Pokrenuti:

```
mnogo iteracija
```

---

Primer:

```go
for i:=0;i<10000;i++ {

	testScenario()

}
```

---

Cilj:

pronaći:

```
retke probleme
```

---

# 3. Benchmark Testing

Meriti:

- throughput,
- latency,
- memory usage.

---

Komanda:

```bash
go test -bench .
```

---

# 4. Goroutine Leak Testing

Proveriti:

pre:

```go
runtime.NumGoroutine()
```

---

posle:

```go
runtime.NumGoroutine()
```

---

Ako broj stalno raste:

moguć leak.

---

# Arhitektonski principi

---

# Princip #1

Svaka Goroutine mora imati vlasnika.

---

Pitanje:

```
Ko je pokreće?

Ko je zaustavlja?
```

---

# Princip #2

Svaki channel mora imati jasnu odgovornost.

---

Pitanja:

```
Ko šalje?

Ko prima?

Ko zatvara?
```

---

# Princip #3

Cancellation mora da se propagira.

---

Model:

```
Parent Context

↓

Child Goroutines
```

---

# Princip #4

Ograniči concurrency.

---

Ne:

```
beskonačan broj Goroutines
```

---

Koristi:

- worker pool,
- semaphore,
- rate limiter.

---

# Senior dizajn pitanja

Pre implementacije pitati:

```
Da li je problem IO ili CPU?

Koji je bottleneck?

Koliko concurrency-ja je potrebno?

Kako završavaju Goroutines?

Kako testirati race uslove?
```

---

# 📋 Rezime

U ovom delu smo:

✅ napravili praktične projekte  
✅ povezali concurrency primitive  
✅ naučili arhitekturu realnih sistema  
✅ obradili testiranje concurrent koda  
✅ definisali production principe  

---

# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj ovog dela

- Junior zadaci
- Medior zadaci
- Senior zadaci
- Evaluacija znanja
- Očekivani nivo nakon Modula #3

---

# Uvod

Nakon završetka Modula #3 developer treba da razume:

```
kako Go izvršava konkurentne programe

+

kako kontrolisati konkurentnost

+

kako dizajnirati skalabilne sisteme
```

---

Nije cilj samo znati API:

```go
sync.Mutex
context.Context
go func()
```

---

Cilj je razumeti:

```
kada

zašto

i kako

koristiti određeni mehanizam
```

---

# 🟢 Junior nivo

## Cilj

Junior developer treba sigurno da koristi osnovne concurrency primitive.

---

# Zadatak J1 — Counter sa Mutex-om

Napraviti:

```go
type Counter struct {
	value int
}
```

---

Dodati metode:

```go
Increment()

Value()
```

---

Zahtevi:

- više Goroutines povećava counter,
- nema race condition-a.

---

Koristiti:

```
sync.Mutex
```

---

Provera:

```bash
go test -race
```

---

# Zadatak J2 — Goroutine Synchronization

Napraviti program:

```
5 Goroutines

↓

svaka radi posao

↓

main čeka završetak
```

---

Koristiti:

```go
sync.WaitGroup
```

---

Cilj:

razumeti:

```
start

↓

wait

↓

finish
```

---

# Zadatak J3 — Context Timeout

Napraviti funkciju:

```go
Process(ctx context.Context)
```

---

Ako posao traje duže:

```
prekini
```

---

Koristiti:

```go
context.WithTimeout()
```

---

# Zadatak J4 — Channel Communication

Napraviti:

```
Producer

↓

Channel

↓

Consumer
```

---

Producer šalje:

```text
brojeve 1-100
```

---

Consumer obrađuje.

---

Cilj:

razumeti:

```
message passing
```

---

# Zadatak J5 — Race Detection

Napraviti namerno:

```go
counter++
```

iz više Goroutines.

---

Pokrenuti:

```bash
go test -race
```

---

Analizirati problem.

---

# 🟡 Medior nivo

## Cilj

Medior developer treba da dizajnira concurrent komponente.

---

# Zadatak M1 — Worker Pool

Implementirati:

```
Job Queue

+

N Workers

+

Results
```

---

Zahtevi:

- konfigurabilan broj worker-a,
- graceful shutdown,
- error handling.

---

Koristiti:

- Channels,
- WaitGroup,
- Context.

---

# Zadatak M2 — Thread-safe Cache

Napraviti:

```
Concurrent Cache
```

---

Podržati:

```go
Get()

Set()

Delete()
```

---

Koristiti:

```
sync.RWMutex
```

---

Dodati:

- TTL,
- cleanup goroutine.

---

# Zadatak M3 — Pipeline Processing

Implementirati:

```
Generate

↓

Transform

↓

Filter

↓

Save
```

---

Svaki stage:

```
posebna Goroutine
```

---

Komunikacija:

```
Channels
```

---

# Zadatak M4 — Rate Limited API Client

Napraviti klijenta koji:

- šalje HTTP zahteve,
- ima maksimalan broj request-a/sec,
- podržava cancellation.

---

Koristiti:

- Context,
- Semaphore,
- Timer.

---

# Zadatak M5 — Parallel Processor

Obraditi veliki dataset.

---

Uporediti:

```
single-thread

vs

parallel workers
```

---

Meriti:

- execution time,
- memory,
- CPU.

---

Koristiti:

```bash
go test -bench .
```

---

# 🔴 Senior nivo

## Cilj

Senior developer treba da dizajnira production concurrency sisteme.

---

# Zadatak S1 — Production Worker Pool

Napraviti sistem koji podržava:

- dynamic worker count,
- cancellation,
- retry,
- timeout,
- metrics,
- graceful shutdown.

---

Arhitektura:

```
API

↓

Queue

↓

Workers

↓

Storage
```

---

# Zadatak S2 — Concurrent In-Memory Database

Napraviti:

```
Key-Value Store
```

---

Podržati:

```go
Get()

Set()

Delete()

List()
```

---

Zahtevi:

- thread safety,
- locking strategy,
- performance test.

---

Analizirati:

```
Mutex

vs

RWMutex

vs

Atomic
```

---

# Zadatak S3 — Event Processing System

Napraviti:

```
Event Stream

↓

Consumers

↓

Processors

↓

Storage
```

---

Dodati:

- backpressure,
- retry,
- dead letter queue.

---

# Zadatak S4 — Concurrent Web Crawler

Implementirati crawler koji:

- posećuje URL-ove,
- ograničava concurrency,
- sprečava duplikate,
- podržava cancellation.

---

Koristiti:

- Worker Pool,
- Semaphore,
- Context.

---

# Zadatak S5 — Performance Analysis

Napraviti tri implementacije:

```
Sequential

Concurrent

Parallel
```

---

Analizirati:

- latency,
- throughput,
- CPU,
- memory,
- scheduler behavior.

---

# Evaluacija znanja

## Junior

Developer zna:

✅ koristiti Goroutines  
✅ koristiti Channels  
✅ koristiti WaitGroup  
✅ koristiti Mutex  
✅ koristiti Context  
✅ razumeti race condition  

---

## Medior

Developer zna:

✅ dizajnirati Worker Pool  
✅ koristiti Pipeline  
✅ koristiti RWMutex  
✅ kontrolisati concurrency  
✅ raditi graceful shutdown  
✅ pisati concurrent testove  

---

## Senior

Developer zna:

✅ dizajnirati concurrency arhitekturu  
✅ analizirati bottleneck-e  
✅ optimizovati scheduler usage  
✅ birati synchronization strategiju  
✅ raditi performance tuning  
✅ sprečiti goroutine leak-ove  

---

# Finalni checklist

Pre završetka Modula #3 proveriti:

---

## Runtime

- [ ] Razumem GMP model
- [ ] Razumem scheduler
- [ ] Razumem GOMAXPROCS

---

## Synchronization

- [ ] Znam kada koristiti Mutex
- [ ] Znam kada koristiti RWMutex
- [ ] Razumem Once

---

## Context

- [ ] Znam timeout
- [ ] Znam cancellation
- [ ] Znam propagation

---

## Patterns

- [ ] Worker Pool
- [ ] Pipeline
- [ ] Fan-In
- [ ] Fan-Out
- [ ] Semaphore
- [ ] Rate Limiting

---

## Performance

- [ ] Razumem race condition
- [ ] Razumem contention
- [ ] Znam koristiti race detector
- [ ] Znam koristiti benchmark

---

# 📋 Završni zaključak Modula #3

Nakon ovog modula developer više ne vidi:

```go
go func()
```

kao samo:

"pokreni paralni kod".

---

Već razume:

```
Goroutine

↓

Scheduler

↓

Synchronization

↓

Execution

↓

Performance
```

---

To predstavlja osnovu za:

- backend servise,
- distribuirane sisteme,
- high-load aplikacije,
- cloud-native Go aplikacije.

---

# Module #3 — Summary and Exercises

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 9/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/09-module-3-summary-and-exercises.md`

---

# 📚 Sadržaj ovog dela

- Finalni integracioni projekat
- Production-grade Concurrent Job Processing System
- Arhitektura sistema
- Zahtevi implementacije
- Faze razvoja
- Evaluacioni kriterijumi
- Modul #3 završna provera znanja

---

# Finalni projekat Modula #3

# 🚀 Concurrent Job Processing System

## Cilj projekta

Napraviti kompletan sistem za izvršavanje poslova koji demonstrira sve ključne koncepte iz Modula #3:

```
Goroutines

+

Channels

+

Mutex/RWMutex

+

Context

+

Timeout

+

Cancellation

+

Worker Pool

+

Pipeline

+

Rate Limiting

+

Graceful Shutdown
```

---

# Scenario

Zamisli backend servis koji prima zahteve za obradu različitih poslova:

Primeri:

- generisanje izveštaja,
- obrada fajlova,
- slanje email-ova,
- resize slika,
- eksport podataka.

---

Sistem mora:

1. Primiti posao.
2. Staviti ga u queue.
3. Obraditi ga preko worker-a.
4. Vratiti rezultat.
5. Omogućiti kontrolisano gašenje.

---

# Visok nivo arhitekture

```
                Client
                  |
                  v
            Job Submit API
                  |
                  v
              Job Queue
                  |
                  v
          +---------------+
          | Worker Pool   |
          +---------------+
            |     |     |
            v     v     v
          W1     W2    W3

                  |
                  v

             Result Store
```

---

# Komponente sistema

---

# 1. Job Model

Definisati:

```go
type Job struct {

	ID string

	Type string

	Payload any

}
```

---

Rezultat:

```go
type Result struct {

	JobID string

	Output any

	Error error

}
```

---

# 2. Job Queue

Zadatak:

čuvanje poslova koji čekaju obradu.

---

Implementacija:

```go
jobs chan Job
```

---

Zahtevi:

- buffered channel,
- kontrola kapaciteta,
- backpressure.

---

Primer:

```go
make(chan Job, 100)
```

---

# 3. Worker Pool

Implementirati:

```
N worker-a
```

---

Primer:

```
Worker 1

Worker 2

Worker 3

Worker 4
```

---

Svaki worker:

radi:

```go
for {

	select {

	case job := <-jobs:

		process(job)

	case <-ctx.Done():

		return

	}

}
```

---

# 4. Context Management

Sistem mora podržati:

## Timeout

Primer:

```
Job maksimalno 30 sekundi
```

---

## Cancellation

Primer:

Administrator gasi servis:

```
stop processing
```

---

Model:

```
Root Context

      |

      v

Worker Contexts

      |

      v

Job Contexts
```

---

# 5. Graceful Shutdown

Pravilno gašenje:

```
Receive shutdown signal

↓

Stop accepting new jobs

↓

Finish active jobs

↓

Close workers

↓

Exit
```

---

Nikada:

```
kill process odmah
```

---

# 6. Retry mehanizam

Dodati:

```
failed job

↓

retry

↓

success/fail
```

---

Primer:

```go
maxRetries = 3
```

---

Voditi računa:

Retry ne sme napraviti:

```
beskonačnu petlju
```

---

# 7. Rate Limiting

Ograničiti:

koliko poslova može krenuti.

---

Primer:

```
100 jobs/min
```

---

Implementacija:

- ticker,
- semaphore,
- token bucket.

---

# 8. Metrics

Pratiti:

## Active workers

```
koliko worker-a radi
```

---

## Queue size

```
koliko poslova čeka
```

---

## Processing time

```
koliko traje obrada
```

---

## Failed jobs

```
broj grešaka
```

---

# Faze implementacije

---

# Faza 1 — Osnovni sistem

Implementirati:

✅ Job  
✅ Queue  
✅ Worker  
✅ Result  

---

Cilj:

razumeti:

```
Producer → Consumer
```

---

# Faza 2 — Concurrency kontrola

Dodati:

✅ Worker Pool  
✅ WaitGroup  
✅ Channels  

---

Cilj:

kontrolisan concurrency.

---

# Faza 3 — Error handling

Dodati:

✅ Error propagation  
✅ Retry  
✅ Timeout  

---

Cilj:

production ponašanje.

---

# Faza 4 — Shutdown

Dodati:

✅ Context  
✅ Cancellation  
✅ Graceful shutdown  

---

Cilj:

bez goroutine leak-a.

---

# Faza 5 — Performance

Meriti:

- throughput,
- latency,
- memory.

---

Koristiti:

```bash
go test -bench .
```

---

Race provera:

```bash
go test -race ./...
```

---

# Evaluacija projekta

---

# Junior nivo

Sistem mora imati:

✅ Goroutines  
✅ Channels  
✅ WaitGroup  
✅ Mutex  
✅ Context  
✅ Osnovni error handling  

---

# Medior nivo

Dodati:

✅ Worker Pool  
✅ Retry  
✅ Timeout  
✅ Graceful shutdown  
✅ Testove  
✅ Benchmark  

---

# Senior nivo

Dodati:

✅ Dynamic worker scaling  
✅ Metrics  
✅ Backpressure  
✅ Rate limiting  
✅ Performance tuning  
✅ Arhitektonsku dokumentaciju  

---

# Finalna pitanja za samoprocenu

Ako znaš odgovor na sledeća pitanja, Modul #3 je usvojen.

---

## Runtime

### 1.

Šta se dešava nakon:

```go
go function()
```

?

---

### 2.

Koja je razlika:

```
G

M

P
```

?

---

### 3.

Šta kontroliše:

```go
GOMAXPROCS
```

?

---

# Synchronization

### 4.

Kada koristiš:

```
Mutex
```

a kada:

```
RWMutex
```

?

---

### 5.

Zašto postoji:

```
sync.Once
```

?

---

# Context

### 6.

Koja je razlika između:

```
Timeout

Cancellation
```

?

---

### 7.

Kako sprečiti:

```
goroutine leak
```

?

---

# Patterns

### 8.

Kada koristiš:

```
Worker Pool
```

?

---

### 9.

Koja je razlika:

```
Fan-In

Fan-Out
```

?

---

### 10.

Kada koristiš:

```
Semaphore
```

?

---

# Performance

### 11.

Zašto:

```
više Goroutines

≠

uvek brži program
```

?

---

### 12.

Šta je:

```
Lock Contention
```

?

---

# Završni mentalni model

Posle Modula #3 treba da razmišljaš ovako:

```
Problem

↓

IO ili CPU?

↓

Concurrency ili Parallelism?

↓

Koji pattern?

↓

Kako kontrolisati lifecycle?

↓

Kako meriti performanse?
```

---

# 🎓 Kraj Modula #3

Kompletiran je:

```
Module #3

Advanced Go Concurrency Fundamentals
```

---

Stečeno znanje:

✅ Synchronization  
✅ Scheduler  
✅ Runtime model  
✅ Context management  
✅ Concurrency patterns  
✅ Performance analiza  
✅ Production dizajn  

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
