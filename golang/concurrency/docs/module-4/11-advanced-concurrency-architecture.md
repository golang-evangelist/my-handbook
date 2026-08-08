# Advanced Concurrency Architecture

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/11-advanced-concurrency-architecture.md`

---

# 📚 Sadržaj

- Šta je concurrency architecture
- Od Goroutine do sistema
- Ownership model
- Lifecycle dizajn
- Worker Pool uvod
- Pipeline arhitektura uvod
- Backpressure koncept

---

# Uvod

Mali program:

```go
go process()
```

je jednostavan.

---

Ali produkcioni sistem:

```text
HTTP Requests

↓

Workers

↓

Database

↓

External APIs

↓

Shutdown
```

zahteva arhitekturu.

---

# Concurrency Architecture

Concurrency architecture definiše:

- ko kreira Goroutines,
- ko ih zaustavlja,
- ko poseduje stanje,
- kako se prenose podaci,
- kako se kontroliše opterećenje.

---

# Loš dizajn

Primer:

```go
func Handle(){

	go task1()

	go task2()

	go task3()

}
```

---

Problem:

Niko ne zna:

- kada završavaju,
- šta ako puknu,
- kako se gase,
- koliko ih postoji.

---

# Dobar dizajn

Imamo:

```text
Owner

↓

Lifecycle

↓

Workers

↓

Shutdown
```

---

# Ownership Model

Jedan od najvažnijih koncepata.

---

Pitanje:

```
Ko poseduje ovu Goroutine?
```

---

Ako odgovor nije jasan:

verovatno postoji problem.

---

Primer:

```go
worker := NewWorker()
worker.Start()
```

---

Worker poseduje:

- svoju Goroutine,
- channel,
- shutdown logiku.

---

# Goroutine Lifecycle

Svaka Goroutine treba imati:

## Start

```text
kreirana
```

---

## Running

```text
radi
```

---

## Stop

```text
dobija signal
```

---

## Exit

```text
oslobađa resurse
```

---

Model:

```text
Start

↓

Run

↓

Context Cancel

↓

Cleanup

↓

Exit
```

---

# Lifecycle Anti-Pattern

Loše:

```go
go func(){

	for {

		work()

	}

}()
```

---

Problem:

Nema:

- cancel,
- shutdown,
- kontrolu.

---

# Production model

Bolje:

```go
func Worker(
	ctx context.Context,
){

	for {

		select {

		case <-ctx.Done():
			return

		default:
			work()

		}

	}

}
```

---

# Sistem nivo

Concurrency sistem obično ima:

```text
Input

↓

Queue

↓

Workers

↓

Output
```

---

Primer:

HTTP servis:

```text
Request

↓

Job Queue

↓

Workers

↓

Response
```

---

# Problem neograničene konkurentnosti

Loš primer:

```go
for _, job := range jobs {

	go process(job)

}
```

---

Ako imamo:

```
1 000 000 jobs
```

dobijamo:

```
1 000 000 Goroutines
```

---

Problemi:

- memory,
- scheduler,
- GC,
- latency.

---

# Rešenje

Kontrolisana konkurentnost.

---

Primer:

```text
1000 jobs

↓

10 workers

↓

process
```

---

Ovo je:

# Worker Pool

---

# Worker Pool koncept

Worker Pool ima:

## Jobs

Podaci za obradu.

---

## Workers

Goroutines koje rade posao.

---

## Results

Rezultati.

---

Model:

```text
Jobs Channel

      ↓

Worker 1

Worker 2

Worker 3

      ↓

Results
```

---

# Prednosti Worker Pool-a

## 1. Kontrola resursa

Broj Goroutines je ograničen.

---

## 2. Stabilnost

Sistem ne eksplodira pod load-om.

---

## 3. Predvidivost

Lakše merenje performansi.

---

# Pipeline Architecture

Drugi važan obrazac.

---

Pipeline:

> Obrada podataka kroz više faza.

---

Primer:

```text
Input

↓

Parse

↓

Validate

↓

Transform

↓

Save
```

---

Svaka faza:

- ima svoje Goroutines,
- komunicira channel-ima.

---

# Pipeline primer

```go
numbers

↓

square

↓

sum
```

---

Faza 1:

```go
generate()
```

---

Faza 2:

```go
process()
```

---

Faza 3:

```go
consume()
```

---

# Problem Pipeline-a

Šta ako jedna faza radi sporije?

Primer:

```text
Producer

1000/s

↓

Consumer

100/s
```

---

Dolazi do:

```
buffer rasta
```

---

Potrebna je:

# Backpressure

---

# Backpressure koncept

Backpressure znači:

> Sistem kontroliše ulaz kada potrošač ne može da prati brzinu proizvodnje.

---

Bez backpressure:

```text
Input ↑

Memory ↑

Crash
```

---

Sa backpressure:

```text
Input

↓

Limit

↓

Processing
```

---

# Mehanizmi backpressure-a

## Buffered Channel

```go
make(chan Job,100)
```

---

## Semaphore

Ograničava broj aktivnih operacija.

---

## Rate Limiter

Ograničava brzinu.

---

## Queue Limit

Ograničava broj čekajućih poslova.

---

# Concurrency Architecture Principi

## 1. Jasno vlasništvo

Ko startuje?

Ko stopira?

---

## 2. Kontrolisana konkurentnost

Ne kreirati beskonačno Goroutines.

---

## 3. Propagacija Context-a

Svaka komponenta mora znati kada da stane.

---

## 4. Merljivost

Sistem mora imati:

- metrike,
- profile,
- logove.

---

# Mentalni model

Junior:

```
Pokreni Goroutine
```

---

Medior:

```
Koristi Channel
```

---

Senior:

```
Dizajniraj lifecycle,
ownership i backpressure
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je concurrency architecture

✅ ownership model

✅ Goroutine lifecycle

✅ zašto je potrebna kontrolisana konkurentnost

✅ uvod u Worker Pool

✅ uvod u Pipeline

✅ backpressure koncept

---

# Advanced Concurrency Architecture — Worker Pool

---

# 📚 Sadržaj

- Šta je Worker Pool
- Worker Pool komponente
- Job Queue dizajn
- Worker lifecycle
- Result handling
- Error propagation
- Shutdown strategije
- Dynamic worker scaling

---

# Šta je Worker Pool?

Worker Pool je concurrency obrazac gde:

```
Broj poslova

↓

ograničen broj Workers

↓

obrada
```

---

Umesto:

```go
for _, job := range jobs {

	go process(job)

}
```

koristimo:

```text
Jobs

↓

Worker Pool

↓

Results
```

---

# Osnovne komponente

Worker Pool ima četiri glavna dela:

---

## 1. Job Queue

Channel koji sadrži poslove.

Primer:

```go
jobs := make(
	chan Job,
	100,
)
```

---

## 2. Workers

Goroutines koje čitaju jobs.

Primer:

```text
Worker 1

Worker 2

Worker 3
```

---

## 3. Processor

Logika izvršavanja.

Primer:

```go
process(job)
```

---

## 4. Results

Mesto gde se šalju rezultati.

---

Model:

```text
             Jobs

              ↓

      +---------------+

      | Worker Pool   |

      |               |

      | W1 W2 W3 W4  |

      +---------------+

              ↓

          Results
```

---

# Osnovni Worker Pool primer

```go
type Job struct {
	ID int
}
```

---

Worker:

```go
func worker(
	id int,
	jobs <-chan Job,
){

	for job := range jobs {

		process(job)

	}

}
```

---

Start:

```go
for i := 0; i < 5; i++ {

	go worker(
		i,
		jobs,
	)

}
```

---

Producer:

```go
for i := 0; i < 100; i++ {

	jobs <- Job{
		ID:i,
	}

}
```

---

Close:

```go
close(jobs)
```

---

# Worker Lifecycle

Dobar Worker ima:

```
Start

↓

Wait for Job

↓

Process

↓

Handle Error

↓

Repeat

↓

Shutdown
```

---

Worker ne sme:

❌ da ostane blokiran zauvek

❌ da ignoriše cancel

❌ da guta greške

---

# Context-aware Worker

Production verzija:

```go
func worker(
	ctx context.Context,
	jobs <-chan Job,
){

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

---

Prednosti:

- graceful shutdown,
- timeout,
- kontrola lifecycle-a.

---

# Job Queue Dizajn

Najčešće pitanje:

```
Koliki buffer?
```

---

Premali:

```go
make(chan Job,10)
```

---

Problem:

producer često čeka.

---

Preveliki:

```go
make(chan Job,1000000)
```

---

Problem:

memory spike.

---

Buffer treba da bude:

```
ograničen
```

i zasnovan na:

- throughput-u,
- memory limitu,
- SLA zahtevima.

---

# Result Handling

Postoje dva česta modela.

---

# Model #1

## Shared Result Channel

```go
results := make(chan Result)
```

---

Workers:

```go
results <- result
```

---

Consumer:

```go
for r := range results {

	handle(r)

}
```

---

# Model #2

## Future pattern

Svaki job ima svoj rezultat.

Primer:

```go
type Result struct {

	JobID int

	Value string

	Error error

}
```

---

# Error Propagation

Jedna od najvažnijih stvari.

---

Loše:

```go
func worker(){

	process()

}
```

---

Ako proces pukne:

```
greška izgubljena
```

---

Bolje:

```go
result := Result{

	Error: err,

}
```

---

# errgroup pristup

Za grupisanje Goroutines koristi se:

:contentReference[oaicite:0]{index=0}

---

Model:

```text
Worker 1

Worker 2

Worker 3

↓

jedan error
```

---

Prednosti:

- cancel ostalih workers,
- centralno upravljanje greškama.

---

# Shutdown Strategije

Production sistem mora znati:

```
Kako se gasi?
```

---

## Hard Shutdown

Odmah prekid.

---

Problem:

gubi poslove.

---

## Graceful Shutdown

Proces:

```
Stop receiving

↓

Finish current jobs

↓

Exit
```

---

Najčešći izbor.

---

# Graceful Worker Pool Shutdown

Redosled:

```
1. Cancel context

2. Close jobs channel

3. Wait workers

4. Cleanup
```

---

Primer:

```go
cancel()

close(jobs)

wg.Wait()
```

---

# WaitGroup u Worker Pool-u

Koristi se za:

```
čekanje završetka workers
```

---

Primer:

```go
var wg sync.WaitGroup

wg.Add(workers)

go func(){

	defer wg.Done()

	worker()

}()
```

---

# Dynamic Worker Scaling

Fiksan broj workers nije uvek optimalan.

---

Primer:

Load:

```
nizak
```

Potrebno:

```
5 workers
```

---

Load:

```
visok
```

Potrebno:

```
50 workers
```

---

Dynamic scaling:

```text
Queue size

↓

Scale workers
```

---

# Scaling Metrics

Pratimo:

## Queue depth

```
koliko jobova čeka
```

---

## Processing latency

```
koliko traje posao
```

---

## Worker utilization

```
koliko rade
```

---

# Worker Pool Anti-Patterns

---

## Anti-pattern #1

Previše workers

```
100000 workers
```

---

Problem:

- scheduler overhead,
- memory.

---

## Anti-pattern #2

Beskonačan queue

---

Problem:

memory leak.

---

## Anti-pattern #3

Worker bez cancel-a

---

Problem:

goroutine leak.

---

## Anti-pattern #4

Ignorisanje error-a

---

Problem:

silent failure.

---

# Worker Pool Design Checklist

Pre implementacije pitati:

---

## Jobs

```
Koji je posao?
```

---

## Workers

```
Koliko paralelizma treba?
```

---

## Queue

```
Koliko memorije dozvoljavamo?
```

---

## Errors

```
Kako propagiramo greške?
```

---

## Shutdown

```
Kako završavamo?
```

---

# Mentalni model

Loš dizajn:

```
100000 jobs

↓

100000 Goroutines
```

---

Dobar dizajn:

```
100000 jobs

↓

20 Workers

↓

kontrolisan sistem
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Worker Pool arhitekturu

✅ Job Queue

✅ Worker lifecycle

✅ Context-aware workers

✅ Result handling

✅ Error propagation

✅ Graceful shutdown

✅ Dynamic scaling koncept

---

# Advanced Concurrency Architecture — Pipeline Architecture

---

# 📚 Sadržaj

- Šta je Pipeline Architecture
- Stage koncept
- Channel kao transport
- Generator pattern
- Fan-out u pipeline-u
- Fan-in u pipeline-u
- Cancellation propagation
- Backpressure
- Production pipeline dizajn

---

# Šta je Pipeline Architecture?

Pipeline predstavlja obradu podataka kroz niz faza.

---

Model:

```text
Input

↓

Stage 1

↓

Stage 2

↓

Stage 3

↓

Output
```

---

Svaka faza:

- prima podatke,
- obrađuje,
- šalje rezultat sledećoj fazi.

---

# Primer

Obrada fajlova:

```text
Read Files

↓

Parse

↓

Validate

↓

Transform

↓

Save
```

---

Svaka faza može biti:

- jedna Goroutine,
- grupa Goroutines,
- Worker Pool.

---

# Pipeline Stage

Stage je funkcija koja:

uzima:

```go
<-chan T
```

vraća:

```go
chan R
```

---

Primer:

```go
func square(
	in <-chan int,
) <-chan int {

	out := make(chan int)

	go func(){

		defer close(out)

		for n := range in {

			out <- n*n

		}

	}()

	return out
}
```

---

# Generator Pattern

Generator kreira podatke.

---

Primer:

```go
func generate(
	values ...int,
) <-chan int {

	out := make(chan int)

	go func(){

		defer close(out)

		for _, v := range values {

			out <- v

		}

	}()

	return out
}
```

---

Upotreba:

```go
nums := generate(
	1,2,3,4,
)

result := square(nums)
```

---

Tok:

```text
generate

↓

square

↓

consumer
```

---

# Pipeline Primer

Kompletan tok:

```go
nums :=
	generate(
		1,2,3,4,
	)

squares :=
	square(nums)

for value := range squares {

	fmt.Println(value)

}
```

---

Rezultat:

```
1

4

9

16
```

---

# Problem: Spora Faza

Primer:

```text
Stage 1

1000 items/sec


Stage 2

100 items/sec
```

---

Dolazi do:

```
queue raste
```

---

Posledice:

- memory growth,
- latency,
- crash.

---

Rešenje:

# Backpressure

---

# Backpressure u Pipeline-u

Backpressure znači:

> Sporija faza kontroliše brzinu ulaza.

---

Mehanizmi:

---

## 1. Unbuffered Channel

```go
make(chan T)
```

---

Producer čeka consumer.

---

## 2. Ograničen Buffer

```go
make(chan T,100)
```

---

Samo 100 elemenata može čekati.

---

## 3. Worker Limit

Ograničava paralelizam.

---

# Fan-Out

Jedna faza šalje posao više workers.

---

Primer:

```text
Input

  ↓

Worker 1

Worker 2

Worker 3

```

---

Koristi se kada:

```
jedna faza je bottleneck
```

---

Primer:

```text
Parse

↓

10 parsers

↓

Next Stage
```

---

# Fan-Out Implementacija

```go
for i := 0; i < workers; i++ {

	go worker(
		input,
	)

}
```

---

Svi workers čitaju:

```go
<-input
```

---

Channel distribuira posao.

---

# Fan-In

Suprotno od fan-out.

Više izvora:

```text
Worker 1

Worker 2

Worker 3

    ↓

 Output
```

---

Koristi se za:

- spajanje rezultata,
- agregaciju.

---

Primer:

```go
func merge(
	channels ...<-chan int,
) <-chan int
```

---

# Fan-In Implementacija

```go
func merge(
	cs ...<-chan int,
) <-chan int {

	out := make(chan int)

	var wg sync.WaitGroup


	for _, c := range cs {

		wg.Add(1)

		go func(){

			defer wg.Done()

			for v := range c {

				out <- v

			}

		}()

	}


	go func(){

		wg.Wait()

		close(out)

	}()


	return out
}
```

---

# Cancellation Propagation

Pipeline mora moći da se prekine.

---

Problem:

Consumer ode.

Producer nastavlja:

```text
Goroutine leak
```

---

Rešenje:

```go
context.Context
```

---

Primer:

```go
select {

case out <- value:

case <-ctx.Done():

	return

}
```

---

Sada svaki stage zna:

```
kada treba stati
```

---

# Pipeline sa Context-om

Model:

```text
Context

↓

Stage 1

↓

Stage 2

↓

Stage 3
```

---

Cancel:

```go
cancel()
```

---

Rezultat:

sve faze završavaju.

---

# Pipeline Error Handling

Pitanje:

```
Šta ako jedna faza pukne?
```

---

Loše:

```
greška se ignoriše
```

---

Bolje:

Rezultat nosi error:

```go
type Result struct {

	Value Data

	Err error

}
```

---

Ili:

koristiti:

```
errgroup
```

---

# Production Pipeline Primer

Sistem:

```
Events

↓

Decode

↓

Validate

↓

Enrich

↓

Persist
```

---

Decode:

10 workers

---

Validate:

5 workers

---

Persist:

3 workers

---

Zašto?

Jer svaka faza ima drugačiji cost.

---

# Pipeline Tuning

Ne povećavati workers svuda.

---

Primer:

```text
Decode:

CPU bound


Persist:

I/O bound
```

---

Optimalno:

```
Decode workers ↑

Persist workers ograničeni
```

---

# Pipeline Anti-Patterns

---

## Anti-pattern #1

Beskonačni buffer.

---

Problem:

Memory leak.

---

## Anti-pattern #2

Nema cancellation.

---

Problem:

Goroutine leak.

---

## Anti-pattern #3

Sve faze isti broj workers.

---

Problem:

Loše iskorišćenje resursa.

---

## Anti-pattern #4

Ignorisanje sporih consumer-a.

---

Problem:

Backpressure ne postoji.

---

# Pipeline vs Worker Pool

| Worker Pool | Pipeline |
|-|-|
| nezavisni poslovi | tok podataka |
| isti tip obrade | više faza |
| queue + workers | stage + channels |
| paralelizam | transformacija |

---

# Kombinovanje

Često se koriste zajedno:

```text
Pipeline Stage

↓

Worker Pool

↓

Pipeline Stage
```

---

Primer:

```text
Read

↓

Parse Workers

↓

Validate

↓

Save Workers
```

---

# Mentalni model

Worker Pool:

```
Kako obraditi mnogo poslova?
```

---

Pipeline:

```
Kako organizovati tok obrade?
```

---

Senior dizajn:

```
Pipeline

+

Worker Pool

+

Backpressure

+

Cancellation
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Pipeline architecture

✅ Stage koncept

✅ Generator pattern

✅ Fan-out

✅ Fan-in

✅ Cancellation propagation

✅ Backpressure

✅ Production pipeline dizajn

---

# Advanced Concurrency Architecture — Backpressure i Load Control

---

# 📚 Sadržaj

- Šta je Backpressure
- Zašto je potreban Load Control
- Bounded Concurrency
- Semaphore Pattern
- Queue Management
- Rate Limiting
- Admission Control
- Overload Protection

---

# Šta je Backpressure?

Backpressure je mehanizam kojim sporiji deo sistema signalizira:

```
Ne mogu više da primam posao.
```

---

Primer:

Producer:

```
10000 requests/sec
```

---

Consumer:

```
1000 requests/sec
```

---

Bez kontrole:

```
Queue:

1000

↓

10000

↓

100000

↓

Memory problem
```

---

Sa backpressure:

```
Queue limit

↓

uspori producer

↓

sistem ostaje stabilan
```

---

# Fundamentalni princip

Svaki sistem ima:

```
Kapacitet
```

---

Ako:

```
Input > Capacity
```

mora postojati odluka:

1. čekati,
2. odbiti,
3. usporiti,
4. degradirati funkcionalnost.

---

# Backpressure Strategije

Postoje četiri glavne strategije:

---

## 1. Blocking

Producer čeka.

---

Primer:

```go
jobs <- job
```

---

Ako je channel pun:

```
blokiranje
```

---

Prednost:

- jednostavno,
- nema gubitka podataka.

---

Mana:

- povećava latency.

---

# 2. Bounded Queue

Ograničena veličina queue-a.

---

Primer:

```go
jobs :=
	make(chan Job,1000)
```

---

Kada se napuni:

```
nema više prostora
```

---

Sistem mora odlučiti šta dalje.

---

# 3. Dropping

Odbacivanje posla.

---

Primer:

```go
select {

case jobs <- job:

default:

	return errors.New(
		"queue full",
	)

}
```

---

Koristi se za:

- metrics,
- logs,
- cache refresh.

---

# 4. Rate Limiting

Kontrola brzine ulaza.

---

Primer:

```
100 requests/sec
```

---

Više nije dozvoljeno.

---

# Bounded Concurrency

Jedan od najvažnijih pattern-a.

---

Problem:

```go
for {

	go request()

}
```

---

Nema limita.

---

Rešenje:

ograničiti broj aktivnih operacija.

---

Primer:

```
Maximum:

100 concurrent tasks
```

---

# Semaphore Pattern

Semaphore predstavlja:

```
ograničen broj dozvola
```

---

Primer:

```go
sem :=
	make(chan struct{},10)
```

---

Dobijanje dozvole:

```go
sem <- struct{}{}
```

---

Oslobađanje:

```go
<-sem
```

---

Model:

```text
10 tokens

↓

10 aktivnih operacija

↓

ostali čekaju
```

---

# Semaphore Primer

```go
sem :=
	make(chan struct{},5)


for _, item := range items {

	sem <- struct{}{}

	go func(){

		defer func(){

			<-sem

		}()

		process(item)

	}()

}
```

---

Rezultat:

Maksimalno:

```
5 Goroutines radi istovremeno
```

---

# Semaphore vs Worker Pool

Oba ograničavaju concurrency.

---

Worker Pool:

```
fiksni workers

+

queue
```

---

Semaphore:

```
ograničenje aktivnih operacija
```

---

Primer:

HTTP client:

Bolje:

```
Semaphore
```

---

Batch processing:

Bolje:

```
Worker Pool
```

---

# Queue Management

Queue nije beskonačan resurs.

---

Loše:

```go
make(chan Job,10000000)
```

---

Problem:

Ako svaki job ima:

```
10KB
```

onda:

```
10M × 10KB

≈ 100GB
```

---

# Queue Metrics

Production sistem prati:

## Queue Depth

Koliko čeka.

---

## Queue Age

Koliko dugo najstariji job čeka.

---

## Processing Time

Koliko traje obrada.

---

## Rejection Rate

Koliko poslova odbijamo.

---

# Rate Limiting

Rate limiter kontroliše:

```
koliko zahteva po vremenu
```

---

Primer:

API:

```
1000 req/min
```

---

Preko toga:

```
429 Too Many Requests
```

---

# Token Bucket Algorithm

Najčešći algoritam.

---

Model:

```
Bucket

+

Tokens

+

Refill
```

---

Primer:

```
100 tokens

+

10 token/sec refill
```

---

Request:

troši token.

---

Nema tokena:

```
čekaj ili odbij
```

---

# Rate Limiting u Go

Često se koristi:

:contentReference[oaicite:0]{index=0}

---

Primer koncepta:

```go
limiter.Allow()
```

---

Ako:

```go
false
```

request se odbija.

---

# Admission Control

Napredniji koncept.

---

Pitanje:

```
Da li ovaj posao sme u sistem?
```

---

Pre nego što prihvatimo:

proveravamo:

- trenutno opterećenje,
- queue size,
- deadline,
- prioritet.

---

Primer:

Normalan request:

```
accepted
```

---

Low priority:

```
rejected
```

---

# Priority Queue

Nisu svi poslovi jednaki.

---

Primer:

```
Payment

>

Email

>

Analytics
```

---

Sistem može koristiti:

```
priority scheduling
```

---

# Overload Protection

Cilj:

> Sistem mora ostati živ pod prevelikim opterećenjem.

---

Strategije:

---

## Timeout

```go
context.WithTimeout()
```

---

## Circuit Breaker

Zaštita od spoljašnjih servisa.

---

## Load Shedding

Namerno odbacivanje low priority posla.

---

## Bulkhead Pattern

Izolacija resursa.

---

# Bulkhead Pattern

Ideja:

Jedan problem ne sme srušiti sve.

---

Loše:

```
jedan globalni pool
```

---

Bolje:

```
Payment Pool

Email Pool

Report Pool
```

---

Ako:

```
Report pukne
```

Payment radi.

---

# Realni Production Primer

HTTP servis:

```text
Request

↓

Rate Limit

↓

Admission Control

↓

Worker Pool

↓

Database
```

---

Ako DB uspori:

```
Worker Pool pun

↓

Backpressure

↓

Reject novih zahteva
```

---

# Backpressure Checklist

Pitaj:

---

## Queue

```
Da li ima limit?
```

---

## Workers

```
Da li je broj ograničen?
```

---

## Timeout

```
Šta ako posao traje predugo?
```

---

## Failure

```
Šta ako završi kapacitet?
```

---

# Mentalni model

Bez kontrole:

```
više load-a

↓

više svega

↓

sistem pada
```

---

Sa backpressure:

```
više load-a

↓

kontrola

↓

stabilnost
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Backpressure koncept

✅ bounded concurrency

✅ semaphore pattern

✅ queue management

✅ rate limiting

✅ admission control

✅ overload protection

✅ bulkhead pattern

---

# Advanced Concurrency Architecture — Advanced Patterns

---

# 📚 Sadržaj

- Worker Pool + Pipeline kombinacije
- Scatter-Gather pattern
- Fan-out/Fan-in napredni dizajn
- Actor Model
- Event-driven concurrency
- Production concurrency patterns

---

# Uvod

Jedan concurrency pattern rešava jedan problem.

---

Primer:

Worker Pool:

```
kontrola broja izvršavanja
```

---

Pipeline:

```
organizacija toka podataka
```

---

Production sistemi kombinuju više obrazaca.

---

# Pattern kombinacija

Primer:

Image Processing sistem:

```text
Upload

↓

Decode Pipeline

↓

Worker Pool

↓

Resize

↓

Worker Pool

↓

Storage
```

---

Svaka komponenta ima svoju odgovornost.

---

# Worker Pool + Pipeline

Najčešća kombinacija.

---

Model:

```text
Stage 1

↓

Worker Pool

↓

Stage 2

↓

Worker Pool

↓

Stage 3
```

---

Primer:

Video processing:

```text
Download

↓

Decode Workers

↓

Transform Workers

↓

Encode Workers

↓

Save
```

---

# Zašto kombinovati?

Različite faze imaju različit cost.

---

Primer:

Decode:

```
CPU intensive
```

---

Save:

```
I/O intensive
```

---

Ne treba isti broj workers.

---

# Scatter-Gather Pattern

Scatter-Gather znači:

> Pošalji posao više izvršilaca i sakupi rezultate.

---

Model:

```text
Request

   ↓

Scatter

 / | \

W1 W2 W3

 \ | /

 Gather

   ↓

Response
```

---

Primer:

Search sistem.

---

Request:

```
find user
```

---

Parallel:

```
Database A

Database B

Cache

External API
```

---

Gather:

```
combine results
```

---

# Scatter-Gather primer

```go
type Result struct {

	Value string

	Error error

}
```

---

Pokrenemo:

```go
go queryDB()

go queryCache()

go queryAPI()
```

---

Sakupljamo:

```go
for i:=0;i<3;i++ {

	result := <-results

}
```

---

# Prednosti

Ako su pozivi nezavisni:

sekvencijalno:

```
100ms
+
100ms
+
100ms

=300ms
```

---

Paralelno:

```
max(100,100,100)

=100ms
```

---

# Problem Scatter-Gather-a

Jedan spor poziv može blokirati sve.

---

Primer:

```
DB A 50ms

DB B 60ms

API 5s
```

---

Rezultat:

```
čekamo 5s
```

---

Rešenje:

- timeout,
- context cancellation,
- partial results.

---

# Context u Scatter-Gather

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		time.Second,
	)

defer cancel()
```

---

Ako jedan servis kasni:

```
cancel
```

---

# Fan-Out / Fan-In Advanced

Osnovni model:

```text
Input

↓

Workers

↓

Merge
```

---

Napredni model:

```text
              Worker A

Input

              Worker B

              Worker C


              ↓


            Aggregator
```

---

# Aggregator Pattern

Jedna Goroutine upravlja stanjem.

---

Primer:

```text
Workers

↓

Events

↓

Aggregator

↓

State
```

---

Prednost:

nema:

```
shared memory + mutex
```

---

# Actor Model

Actor model zasniva se na:

```
Actor

+

Mailbox

+

Messages
```

---

Actor:

- ima svoje stanje,
- obrađuje poruke,
- nema direktan shared state.

---

Model:

```text
Message

↓

Actor

↓

State update
```

---

# Actor u Go

Go nema ugrađen Actor framework.

Ali može se napraviti:

```go
type Actor struct {

	mailbox chan Message

	state State

}
```

---

Actor loop:

```go
for msg := range mailbox {

	handle(msg)

}
```

---

# Prednosti Actor Model-a

Izbegava:

- data race,
- mutex kompleksnost,
- shared memory probleme.

---

Koristi se za:

- distributed systems,
- stateful services,
- event processing.

---

# Event-driven Concurrency

Moderan sistem često izgleda:

```text
Event

↓

Broker

↓

Consumers

↓

Workers
```

---

Primer:

```text
OrderCreated

↓

Payment Service

↓

Inventory Service

↓

Email Service
```

---

Svaki consumer:

ima:

- svoje workers,
- svoj lifecycle,
- svoj backpressure.

---

# Event Processing Pattern

Consumer:

```text
Receive Event

↓

Validate

↓

Process

↓

Ack
```

---

Ako greška:

```
Retry

↓

Dead Letter Queue
```

---

# Production Pattern: Retry Worker

Čest sistem:

```text
Job

↓

Worker

↓

Error?

↓

Retry Queue

↓

Worker
```

---

Mora imati:

- max retry,
- backoff,
- timeout.

---

# Exponential Backoff

Primer:

Retry:

```
1s

↓

2s

↓

4s

↓

8s
```

---

Zaštita:

- od preopterećenja,
- od retry storm-a.

---

# Production Pattern: Timeout Budget

Request ima:

```
1 sekunda
```

---

Ne treba:

Service A:

1s

+

Service B:

1s

---

Već:

```
deliti budget
```

---

Primer:

```text
API

500ms

DB

300ms

External

200ms
```

---

# Production Pattern: Bulkhead + Worker Pool

Primer:

```text
Payment Workers

10


Email Workers

20
```

---

Email problem:

ne utiče na:

```
Payment
```

---

# Production Pattern: Graceful Degradation

Ako sistem nije kompletno dostupan:

koristi:

```
fallback
```

---

Primer:

Cache postoji:

```
return cache
```

---

Database down:

```
partial response
```

---

# Senior Concurrency Design

Senior developer razmišlja:

---

Ne:

```
Koliko Goroutines da napravim?
```

---

Već:

```
Koji model izvršavanja odgovara problemu?
```

---

Pitanja:

```
Da li je posao nezavisan?

Da li postoji tok podataka?

Da li treba kontrola resursa?

Kako se sistem gasi?

Kako se ponaša pod overload-om?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Worker Pool + Pipeline kombinacije

✅ Scatter-Gather

✅ Advanced Fan-out/Fan-in

✅ Aggregator pattern

✅ Actor Model

✅ Event-driven concurrency

✅ Retry pattern

✅ Bulkhead i graceful degradation

---

# Advanced Concurrency Architecture — Production Case Study

---

# 📚 Sadržaj

- Production servis arhitektura
- HTTP layer
- Admission Control
- Worker Pool
- Pipeline obrada
- Context lifecycle
- Graceful shutdown
- Error handling
- Metrics i monitoring
- Finalni concurrency model

---

# Case Study

Dizajniramo servis:

```
Image Processing Service
```

Funkcionalnost:

Korisnik šalje:

```
Image Upload Request
```

Sistem:

- validira fajl,
- obrađuje sliku,
- generiše thumbnail,
- čuva rezultat.

---

# Problem

Naivna implementacija:

```go
func Upload(){

	go processImage()

}
```

---

Problemi:

- nema limita,
- nema shutdown-a,
- nema backpressure-a,
- nema kontrole memorije.

---

# Production Arhitektura

Kompletan tok:

```text
                HTTP Request

                     |

                     ↓

             Admission Control

                     |

                     ↓

                Job Queue

                     |

                     ↓

             Processing Workers

                     |

                     ↓

              Pipeline Stages

                     |

                     ↓

                 Storage

                     |

                     ↓

               Result Event
```

---

# 1. HTTP Layer

HTTP handler ne radi težak posao.

---

Loše:

```go
func handler(){

	process()

}
```

---

Problem:

Request drži konekciju dugo.

---

Bolje:

```go
func handler(){

	jobQueue <- job

	return

}
```

---

Handler odgovara:

```
accepted
```

---

Processing ide asinhrono.

---

# 2. Admission Control

Pre nego što prihvatimo posao:

proveravamo:

- queue size,
- dostupne workers,
- rate limit.

---

Model:

```text
Request

↓

Can Accept?

↓

YES / NO
```

---

Ako nema kapaciteta:

```
429 Too Many Requests
```

---

# 3. Rate Limiting

Zaštita od prevelikog broja zahteva.

---

Primer:

```
100 uploads/sec
```

---

Više:

```
reject
```

---

Prednost:

Sistem ostaje stabilan.

---

# 4. Job Queue

Ograničen buffer:

```go
jobs :=
	make(
		chan Job,
		1000,
	)
```

---

Nikada:

```go
chan Job, infinity
```

---

Queue predstavlja:

```
memory boundary
```

---

# 5. Worker Pool

Processing workers:

```text
Worker 1

Worker 2

Worker 3

Worker 4
```

---

Svaki worker:

uzima:

```
Job
```

radi:

```
process()
```

vraća:

```
Result
```

---

# Worker Lifecycle

Worker:

```text
Start

↓

Read Job

↓

Process

↓

Report Result

↓

Repeat

↓

Shutdown
```

---

# 6. Pipeline Stage

Obrada slike:

```text
Decode

↓

Resize

↓

Compress

↓

Save
```

---

Svaka faza:

- ima svoj workload,
- može imati svoje workers.

---

Primer:

Decode:

```
CPU bound
```

---

Save:

```
I/O bound
```

---

# 7. Backpressure

Scenario:

Upload:

```
10000/min
```

---

Processing:

```
1000/min
```

---

Bez zaštite:

```
queue raste
```

---

Sa backpressure:

```text
Queue full

↓

Reject

↓

System stable
```

---

# 8. Context Lifecycle

Svaki posao dobija context.

---

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		parent,
		time.Minute,
	)
```

---

Ako:

- timeout,
- shutdown,
- client cancel,

sve staje.

---

# 9. Error Handling

Greška ne sme nestati.

---

Rezultat:

```go
type Result struct {

	ID string

	Error error

}
```

---

Opcije:

- retry,
- dead letter queue,
- alert.

---

# Retry Strategy

Ne:

```text
retry forever
```

---

Bolje:

```
Attempt 1

↓

1s

↓

Attempt 2

↓

2s

↓

Attempt 3

↓

fail
```

---

# 10. Graceful Shutdown

Production servis mora pravilno da se ugasi.

---

Proces:

```text
SIGTERM

↓

Stop accepting jobs

↓

Finish active jobs

↓

Close resources

↓

Exit
```

---

Primer:

```go
cancel()

close(jobs)

wg.Wait()
```

---

# 11. Metrics

Bez metrika nema optimizacije.

---

Pratimo:

## Workers

```
active workers
```

---

## Queue

```
queue depth
```

---

## Latency

```
processing time
```

---

## Errors

```
failure rate
```

---

## Throughput

```
jobs/sec
```

---

# 12. Profiling

Kada postoji problem:

---

CPU:

```
pprof CPU
```

---

Memory:

```
heap profile
```

---

Concurrency:

```
goroutine/block/mutex profile
```

---

Trace:

```
scheduler analiza
```

---

# Finalna Arhitektura

```text
                 Client

                   |

                   ↓

             HTTP Handler

                   |

                   ↓

          Rate Limiter

                   |

                   ↓

        Admission Controller

                   |

                   ↓

              Job Queue

                   |

                   ↓

          Worker Pool

                   |

                   ↓

             Pipeline

                   |

                   ↓

             Storage

                   |

                   ↓

            Monitoring
```

---

# Design Decisions

## Zašto Worker Pool?

Da kontrolišemo:

```
broj paralelnih obrada
```

---

## Zašto Queue limit?

Da zaštitimo:

```
memoriju
```

---

## Zašto Context?

Da omogućimo:

```
cancellation
```

---

## Zašto Metrics?

Da znamo:

```
stvarno ponašanje sistema
```

---

# Senior Concurrency Checklist

Pre produkcije proveriti:

---

## Lifecycle

✅ Sve Goroutines imaju shutdown.

---

## Ownership

✅ Jasno ko poseduje stanje.

---

## Limits

✅ Queue i workers imaju granice.

---

## Failure

✅ Greške imaju strategiju.

---

## Observability

✅ Metrike i profiling postoje.

---

## Performance

✅ Benchmark i profiling potvrđuju dizajn.

---

# Junior vs Senior Perspektiva

Junior:

```
Kako pokrenuti Goroutine?
```

---

Medior:

```
Kako koristiti Channel?
```

---

Senior:

```
Kako dizajnirati sistem
koji ostaje stabilan pod load-om?
```

---

# 📋 Rezime Modula #4.11

Naučili smo:

✅ concurrency architecture principe

✅ Worker Pool dizajn

✅ Pipeline arhitekturu

✅ Backpressure

✅ Load Control

✅ Production patterns

✅ kompletan servis dizajn

---

# 🎯 Modul #4.11 završen

Prešli smo put:

```text
Goroutine

↓

Channel

↓

Pattern

↓

Architecture

↓

Production System
```

---

### ➡️ Sledeća lekcija **[**Distributed Concurrency & Go Systems**](12-distributed-concurrency.md)**

Obuhvatiće:

- concurrency preko granica procesa
- messaging sistemi
- event-driven arhitektura
- distributed workers
- idempotency
- retry sistemi
- consistency problemi
- production distributed patterns