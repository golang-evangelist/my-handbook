# Performance i Profiling

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/10-performance-and-profiling.md`

---

# 📚 Sadržaj

- Šta je profiling
- Zašto je profiling važan
- Performance mindset
- Go runtime observability
- Vrste profiling-a
- Benchmark vs Profiling
- pprof uvod

---

# Uvod

Performance problem može izgledati ovako:

```text
API spor

↓

CPU visok

↓

Memory raste

↓

Latency raste
```

---

Ali pitanje je:

```
Zašto?
```

---

Mogući uzroci:

- CPU bottleneck,
- GC overhead,
- memory allocation,
- lock contention,
- goroutine blocking,
- I/O čekanje.

---

Bez merenja:

```text
optimizacija = pretpostavka
```

---

Sa merenjem:

```text
optimizacija = odluka zasnovana na podacima
```

---

# Performance Mindset

Loš pristup:

> "Ovaj kod izgleda spor."

---

Bolji pristup:

> "Profil pokazuje da 70% vremena odlazi u ovu funkciju."

---

Primer:

Imamo API:

```
1000 requests/sec
```

Latency:

```
500ms
```

---

Profiling pokaže:

```
Database query:
60%

JSON encoding:
30%

Business logic:
10%
```

---

Ne optimizujemo business logic.

---

# Šta je Profiling?

Profiling je proces:

> Sistematsko prikupljanje podataka o izvršavanju programa.

---

Profil pokazuje:

- gde CPU troši vreme,
- gde memorija odlazi,
- koje Goroutines postoje,
- gde se program blokira.

---

# Vrste Profiling-a

Go runtime pruža nekoliko važnih profila.

---

# 1. CPU Profile

Odgovara na pitanje:

```
Gde program troši CPU vreme?
```

---

Primer:

```text
Function A

40%

Function B

30%

Function C

20%
```

---

Koristan za:

- algoritme,
- hot path,
- optimizaciju funkcija.

---

# 2. Memory Profile (Heap)

Odgovara:

```
Ko alocira memoriju?
```

---

Pokazuje:

- broj alokacija,
- veličinu objekata,
- zadržanu memoriju.

---

Koristan za:

- GC probleme,
- memory leak,
- previše allocation-a.

---

# 3. Goroutine Profile

Odgovara:

```
Koliko Goroutines postoji?
```

---

Pomaže za:

- goroutine leak,
- blokirane Goroutines,
- deadlock analizu.

---

Primer:

```text
100000 goroutines

↓

problem
```

---

# 4. Block Profile

Pokazuje:

```
Gde Goroutines čekaju?
```

---

Primer:

Čekanje na:

- channel,
- mutex,
- synchronization primitive.

---

# 5. Mutex Profile

Pokazuje:

```
Gde postoji lock contention?
```

---

Primer:

```text
Mutex A

80% waiting
```

---

Znači:

Mutex je bottleneck.

---

# Benchmark vs Profiling

Često se mešaju.

---

# Benchmark

Odgovara:

```
Koliko brzo radi ovaj deo koda?
```

---

Primer:

```go
BenchmarkEncode
```

---

Meri:

- ns/op,
- allocs/op,
- bytes/op.

---

# Profiling

Odgovara:

```
Zašto je sistem spor?
```

---

Meri:

- CPU usage,
- memory usage,
- blocking.

---

# Primer razlike

Benchmark:

```
JSON Encode:

200ns
```

---

Profil produkcije:

```
JSON Encode:

40% CPU
```

---

Benchmark kaže:

```
brzo je
```

---

Profil kaže:

```
radi milion puta
```

---

# Go Runtime Observability

Go runtime već daje mnogo informacija.

---

Važni alati:

```bash
go test
```

```bash
go tool pprof
```

```bash
go tool trace
```

```bash
runtime/pprof
```

```bash
runtime/trace
```

---

# pprof

`pprof` je standardni Go alat za profiling.

---

Može analizirati:

- CPU,
- heap,
- goroutines,
- mutex,
- block.

---

Model:

```text
Application

↓

Collect Profile

↓

pprof

↓

Analysis
```

---

# Osnovni tok rada

## 1. Problem

Primer:

```
API latency raste
```

---

## 2. Merenje

Pokrenemo:

```
CPU profile
```

---

## 3. Analiza

Pronađemo:

```
hot function
```

---

## 4. Izmena

Optimizujemo.

---

## 5. Ponovno merenje

Potvrdimo rezultat.

---

# Najčešće greške kod optimizacije

## Greška #1

Optimizovanje bez merenja.

---

## Greška #2

Fokus na mali deo koda.

---

Primer:

Optimizujemo:

```
1% vremena
```

---

A problem je:

```
Database 80%
```

---

## Greška #3

Ignorisanje allocation-a.

---

Kod može biti:

```
CPU brz

Memory spor
```

---

# Performance Golden Rules

✅ Measure first.

✅ Optimize bottleneck.

✅ Benchmark change.

✅ Profiling u realnim uslovima.

✅ Ne žrtvuj čitljivost bez razloga.

---

# Mentalni model

Junior:

```text
Mislim da je sporo
```

---

Medior:

```text
Izmerio sam vreme
```

---

Senior:

```text
Profil pokazuje bottleneck
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je profiling

✅ zašto je važan

✅ CPU, Memory, Goroutine, Block i Mutex profile

✅ razliku između benchmark-a i profiling-a

✅ uvod u pprof

---

# Performance i Profiling — Go Benchmarking

---

# 📚 Sadržaj

- Šta je Go Benchmark
- `testing.B`
- `b.N`
- Benchmark struktura
- Benchmark pravila
- Memory benchmark
- Allocs/op
- Benchmark concurrency koda
- Česte greške

---

# Uvod

Go ima ugrađenu podršku za benchmark testove kroz:

```go
testing
```

paket.

---

Benchmark funkcija:

```go
func BenchmarkXxx(
	b *testing.B,
)
```

---

Za razliku od običnog testa:

```go
func TestXxx(
	t *testing.T,
)
```

---

Benchmark meri performanse.

---

# Osnovni Benchmark

Primer:

```go
func BenchmarkAdd(
	b *testing.B,
){

	for i := 0; i < b.N; i++ {

		add(10,20)

	}

}
```

---

Pokretanje:

```bash
go test -bench=.
```

---

Rezultat:

```
BenchmarkAdd-8

1000000000

1.2 ns/op
```

---

# Šta je b.N?

Najvažniji koncept.

---

`b.N` nije fiksan broj.

Go runtime ga automatski određuje.

---

Cilj:

dobiti stabilno merenje.

---

Primer:

Prvi pokušaj:

```
N = 1
```

---

Ako je brzo:

```
N = 1000
```

---

Ako je i dalje brzo:

```
N = 1000000
```

---

Go povećava N dok ne dobije dovoljno precizan rezultat.

---

# Benchmark Lifecycle

Go radi:

```text
Pokreni benchmark

↓

Podesi N

↓

Izvrši N puta

↓

Izračunaj rezultat
```

---

# Benchmark Pravilo #1

Kod koji meriš mora biti unutar:

```go
for i := 0; i < b.N; i++
```

---

Loše:

```go
result := expensive()

for i := 0; i < b.N; i++ {

}
```

---

Ovo meri:

```
praznu petlju
```

---

# Benchmark Pravilo #2

Setup ne sme biti deo merenja.

---

Loše:

```go
for i := 0; i < b.N; i++ {

	data := createData()

	process(data)

}
```

---

Ako meriš:

```
process()
```

onda setup kvari rezultat.

---

Bolje:

```go
data := createData()

b.ResetTimer()

for i := 0; i < b.N; i++ {

	process(data)

}
```

---

# b.ResetTimer()

Koristi se kada postoji priprema.

---

Primer:

```go
setup()

b.ResetTimer()

benchmarkCode()
```

---

Timer počinje od:

```
ResetTimer()
```

---

# Benchmark Memory Allocation

Go benchmark može meriti memoriju.

---

Pokretanje:

```bash
go test \
-bench=. \
-benchmem
```

---

Primer:

```
BenchmarkJSON

500 ns/op

200 B/op

3 allocs/op
```

---

Značenje:

## ns/op

Vreme po operaciji.

---

## B/op

Broj bajtova alociranih.

---

## allocs/op

Broj heap alokacija.

---

# Zašto su alokacije važne?

Go ima Garbage Collector.

Više alokacija znači:

```
više GC rada
```

---

Primer:

Kod A:

```
100 ns/op

10 allocs/op
```

---

Kod B:

```
120 ns/op

0 allocs/op
```

---

U produkciji B može biti bolji.

---

# Allocs/op analiza

Primer:

```go
func BenchmarkString(
	b *testing.B,
){

	for i := 0; i < b.N; i++ {

		_ = fmt.Sprintf(
			"hello %d",
			i,
		)

	}

}
```

---

Mogući rezultat:

```
5 allocs/op
```

---

Optimizacija:

```go
strconv.AppendInt()
```

može smanjiti:

```
allocs/op
```

---

# Benchmark sa ResetTimer + Cleanup

Primer:

```go
func BenchmarkWorker(
	b *testing.B,
){

	resource := setup()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {

		use(resource)

	}

	b.StopTimer()

	cleanup(resource)

}
```

---

# Benchmark Concurrent koda

Concurrency benchmark zahteva pažnju.

---

Primer:

```go
func BenchmarkChannel(
	b *testing.B,
){

	ch := make(chan int, 100)

	b.RunParallel(
		func(pb *testing.PB){

			for pb.Next(){

				ch <- 1

			}

		},
	)

}
```

---

# b.RunParallel()

Koristi se za:

- više Goroutines,
- paralelne operacije.

---

Model:

```text
Goroutine 1

Goroutine 2

Goroutine 3

        ↓

    Benchmark
```

---

# GOMAXPROCS i Benchmark

Concurrency benchmark zavisi od:

```bash
GOMAXPROCS
```

---

Možemo testirati:

```bash
GOMAXPROCS=1 go test -bench=.
```

---

i:

```bash
GOMAXPROCS=8 go test -bench=.
```

---

Razlika pokazuje skaliranje.

---

# Benchmark Channel vs Mutex

Primer pitanja:

```
Da li je Channel brži od Mutex-a?
```

---

Ne postoji univerzalni odgovor.

Mora se meriti.

---

Primer:

Benchmark:

```
Mutex Counter

50 ns/op
```

---

Channel:

```
200 ns/op
```

---

Za ovaj slučaj:

Mutex je bolji.

---

Ali za:

```
Pipeline
```

Channel može biti pravi dizajn.

---

# Česte Benchmark greške

---

## Greška #1

Merenje samo jednom.

---

Loše:

```go
start := time.Now()

function()

elapsed := time.Since(start)
```

---

Bolje:

```bash
go test -bench
```

---

## Greška #2

Compiler optimizacija ukloni kod.

---

Primer:

```go
calculate()
```

rezultat se ne koristi.

---

Rešenje:

koristi globalni sink:

```go
var result int
```

---

## Greška #3

Benchmark nije realan.

---

Primer:

Testiraš:

```
10 elemenata
```

Produkcija:

```
milion elemenata
```

---

# Benchmark Checklist

Pre benchmark-a:

✅ Šta merim?

✅ Da li setup utiče?

✅ Da li koristim realne podatke?

✅ Da li gledam allocs/op?

✅ Da li proveravam više GOMAXPROCS vrednosti?

---

# Performance Workflow

```text
Benchmark

↓

Profiling

↓

Optimizacija

↓

Benchmark ponovo
```

---

# Mentalni model

Benchmark odgovara:

```text
Koliko brzo?
```

---

Profiling odgovara:

```text
Zašto sporo?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ strukturu Go benchmark-a

✅ kako radi `b.N`

✅ `ResetTimer`

✅ `benchmem`

✅ `allocs/op`

✅ concurrency benchmark

✅ `RunParallel`

✅ najčešće benchmark greške

---

# Performance i Profiling — CPU Profiling

---

# 📚 Sadržaj

- Šta je CPU profiling
- Kako CPU profiler radi
- `runtime/pprof`
- CPU profile lifecycle
- `go tool pprof`
- Flame Graph
- Flat vs Cumulative time
- Hot path analiza
- Production pristup

---

# Uvod

CPU profiling odgovara na pitanje:

```
Gde program troši CPU vreme?
```

---

Primer problema:

API:

```
Latency ↑
```

CPU:

```
100%
```

---

Mogući uzroci:

- loš algoritam,
- beskonačna petlja,
- previše serializacije,
- kompleksna obrada,
- nepotrebne kalkulacije.

---

Bez CPU profiling-a:

```
nagađanje
```

---

Sa CPU profiling-om:

```
dokaz
```

---

# Kako CPU profiler radi?

Go runtime periodično uzima uzorke izvršavanja.

---

Model:

```text
Program radi

↓

Profiler uzima stack sample

↓

Analiza

↓

Koje funkcije troše vreme?
```

---

Važno:

CPU profiler ne meri svaki poziv.

On radi:

```
sampling
```

---

# Sampling koncept

Primer:

Program:

```
100 sekundi rada
```

Profiler uzme:

```
hiljade uzoraka
```

---

Rezultat:

```
function A

60%

function B

20%

function C

10%
```

---

# CPU Profile preko runtime/pprof

Primer:

```go
package main

import (
	"os"
	"runtime/pprof"
)

func main(){

	file, _ :=
		os.Create(
			"cpu.prof",
		)

	pprof.StartCPUProfile(file)

	defer pprof.StopCPUProfile()


	runApplication()

}
```

---

Tok:

```text
Start

↓

Application

↓

Stop

↓

cpu.prof
```

---

# Analiza profila

Pokretanje:

```bash
go tool pprof cpu.prof
```

---

Dobijamo:

```
(pprof)
```

prompt.

---

Primer:

```bash
top
```

---

Rezultat:

```
flat   cum

40%    40%    parseJSON

20%    60%    handleRequest

10%    70%    validate
```

---

# Flat vs Cumulative vreme

Veoma važan koncept.

---

# Flat

Vreme direktno u funkciji.

Primer:

```
parse()

20%
```

---

Znači:

Sama funkcija troši CPU.

---

# Cumulative

Vreme funkcije + pozvane funkcije.

Primer:

```
handleRequest()

80%
```

---

Možda:

```
handleRequest

↓

parse

↓

validate
```

---

# Primer

Kod:

```go
func process(){

	parse()

	validate()

}
```

---

Profil:

```
process

cum: 90%

flat: 5%
```

---

Znači:

`process` samo poziva druge.

---

# Hot Path

Hot path je:

> Deo programa koji se izvršava najčešće ili najduže traje.

---

Primer:

Request:

```text
HTTP

↓

Handler

↓

JSON

↓

Business Logic

↓

DB
```

---

CPU profil:

```
JSON decode 50%

Business 20%

Handler 10%
```

---

Optimizujemo:

```
JSON decode
```

---

Ne:

```
Handler
```

---

# Flame Graph

Jedan od najpopularnijih prikaza.

---

Izgleda:

```text
main

████████████████

 handler

 ███████████

  parseJSON

  ███████

   decode

   ████
```

---

Širina:

```
CPU vreme
```

---

Dubina:

```
call stack
```

---

Šira funkcija:

```
veći problem
```

---

# pprof komande

## top

Prikaz najskupljih funkcija.

```bash
top
```

---

## list

Prikaz linija koda.

```bash
list FunctionName
```

---

Primer:

```bash
list processRequest
```

---

Dobijamo:

```
linija 45

cost: 30%
```

---

## web

Generiše graf.

```bash
web
```

---

Potrebno:

Graphviz.

---

# CPU Profiling kroz HTTP server

U praksi se često koristi:

:contentReference[oaicite:0]{index=0}

---

Primer:

```go
import _ "net/http/pprof"
```

---

Pokretanje:

```go
go func(){

	http.ListenAndServe(
		":6060",
		nil,
	)

}()
```

---

Dobijamo:

```
/debug/pprof
```

---

CPU profile:

```bash
go tool pprof \
http://localhost:6060/debug/pprof/profile
```

---

# Production CPU Profiling

U produkciji:

NE radimo:

```
stalno profiling
```

---

Bolje:

```
kratki interval

↓

analiza

↓

isključiti
```

---

Primer:

```
30 sekundi CPU profile
```

---

# CPU Bottleneck primeri

---

## Primer #1

Problem:

```
CPU 100%
```

Profil:

```
regexp.Match

70%
```

---

Rešenje:

- cache regex,
- precompile.

---

## Primer #2

Problem:

```
Latency visok
```

Profil:

```
json.Marshal

50%
```

---

Rešenje:

- optimizovati strukture,
- smanjiti encoding.

---

## Primer #3

Problem:

```
Worker Pool spor
```

Profil:

```
mutex.Lock

60%
```

---

Rešenje:

- smanjiti contention,
- drugačiji dizajn.

---

# CPU Profiling i Concurrency

Kod concurrent sistema gledamo:

- lock contention,
- scheduler overhead,
- goroutine switching,
- channel overhead.

---

Primer:

Imamo:

```
1000 workers
```

ali CPU profil:

```
runtime.gopark

40%
```

---

Znači:

mnogo čekanja.

---

# Greške kod CPU profiling-a

## Greška #1

Profiling premalog perioda.

---

## Greška #2

Profiling nereprezentativnog workload-a.

---

## Greška #3

Optimizacija funkcije koja nije bottleneck.

---

# Performance Workflow

```text
Problem

↓

Benchmark

↓

CPU Profile

↓

Hot Path

↓

Optimization

↓

Benchmark Again
```

---

# Mentalni model

Benchmark:

```
Koliko traje?
```

---

CPU Profile:

```
Gde odlazi vreme?
```

---

Flame Graph:

```
Kako je vreme raspoređeno?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je CPU profiling

✅ kako radi sampling

✅ `runtime/pprof`

✅ `go tool pprof`

✅ flat vs cumulative vreme

✅ flame graph

✅ hot path analizu

✅ production CPU profiling pristup

---

# Performance i Profiling — Memory Profiling

---

# 📚 Sadržaj

- Šta je Memory Profiling
- Heap Profiling
- Allocation Profiling
- `runtime/pprof`
- `go tool pprof`
- In-use vs Alloc
- GC i Memory Profiling
- Escape Analysis povezivanje
- Memory leak analiza

---

# Uvod

Memory profiling odgovara na pitanje:

```
Gde program troši memoriju?
```

---

Primer problema:

Aplikacija:

```
RAM ↑
```

posle nekoliko sati rada.

---

Mogući uzroci:

- memory leak,
- previše cache podataka,
- velike strukture,
- nepotrebne alokacije,
- zadržavanje referenci.

---

Bez profiling-a:

```
"verovatno je cache"
```

---

Sa profiling-om:

```
ova funkcija zadržava 500MB
```

---

# Heap Memory

Go memorija ima:

```
Stack

↓

Heap
```

---

Većina dugovečnih objekata završava na:

```
Heap
```

---

Heap profiling pokazuje:

- koji objekti postoje,
- ko ih je kreirao,
- koliko memorije zauzimaju.

---

# Heap Profile

Primer:

```go
import (
	"os"
	"runtime/pprof"
)

func writeHeapProfile(){

	file, _ :=
		os.Create(
			"heap.prof",
		)

	pprof.WriteHeapProfile(file)

}
```

---

Dobijamo:

```
heap.prof
```

---

Analiza:

```bash
go tool pprof heap.prof
```

---

# In-use vs Alloc

Jedan od najvažnijih koncepata.

---

## In-use

Pita:

```
Šta je trenutno živo?
```

---

Primer:

```
cache.Map

500MB
```

---

Koristi se za:

- memory leak,
- zadržane objekte.

---

## Alloc

Pita:

```
Šta je ukupno alocirano?
```

---

Primer:

Tokom rada:

```
10GB allocations
```

ali trenutno:

```
200MB
```

---

Koristi se za:

- GC pressure,
- allocation optimization.

---

# Primer razlike

Aplikacija:

```
RAM stabilan
```

ali:

```
GC CPU visok
```

---

Heap:

```
In-use:

200MB
```

---

Alloc:

```
50GB
```

---

Problem:

Previše kreiranja kratkoživećih objekata.

---

# Allocation Profiling

Odgovara:

```
Ko pravi najviše objekata?
```

---

Primer:

Profil:

```
json.Marshal

40%

fmt.Sprintf

30%

append

20%
```

---

Zaključak:

Problem nisu objekti koji ostaju.

Problem su:

```
stalne nove alokacije
```

---

# GC Veza

Go Garbage Collector radi:

```
Allocation

↓

Heap raste

↓

GC start

↓

Mark

↓

Sweep
```

---

Više allocation-a znači:

```
češći GC
```

---

Posledice:

- CPU overhead,
- latency spike,
- veći tail latency.

---

# Memory Profiling Workflow

```text
Problem

↓

Heap Profile

↓

Pronađi allocator

↓

Analiza

↓

Optimizacija

↓

Ponovo meri
```

---

# Primer #1

## Memory Leak

Simptom:

```
RAM raste svaki sat
```

---

Heap profile:

```
map[string]*Session

5GB
```

---

Analiza:

Sessions nikada nisu uklonjene.

---

Rešenje:

- TTL,
- cleanup,
- bounded cache.

---

# Primer #2

## Previše Allocation-a

Simptom:

```
CPU visok

GC visok
```

---

Profil:

```
fmt.Sprintf

50%
```

---

Problem:

String kreiranje.

---

Optimizacija:

```go
strconv.AppendInt()
```

ili:

```go
strings.Builder
```

---

# Primer #3

## Veliki objekti

Kod:

```go
data := make(
	[]byte,
	1024*1024,
)
```

---

Profil:

```
large byte slice

2GB
```

---

Analiza:

Nepotrebno držanje podataka.

---

# Memory Profile preko HTTP pprof

Koristi se:

:contentReference[oaicite:0]{index=0}

---

Heap:

```bash
go tool pprof \
http://localhost:6060/debug/pprof/heap
```

---

Allocation:

```bash
go tool pprof \
http://localhost:6060/debug/pprof/allocs
```

---

# Escape Analysis + Profiling

Escape analysis kaže:

```
Stack ili Heap?
```

---

Komanda:

```bash
go test -gcflags="-m"
```

---

Primer:

```go
func create() *User {

	u := User{}

	return &u

}
```

---

Compiler:

```
moved to heap
```

---

Memory profile:

pokazuje:

```
User allocations
```

---

Kombinacija:

```
Escape Analysis

+

Heap Profile
```

daje kompletnu sliku.

---

# Memory Profiling i Concurrency

Concurrency može povećati memory problem.

---

Primer:

```text
10000 Goroutines

↓

svaka drži buffer

↓

RAM raste
```

---

Profil:

```
[]byte

80%
```

---

Problem:

Previše paralelnog rada.

---

Rešenje:

- Worker Pool,
- limit concurrency,
- backpressure.

---

# Česte greške

## Greška #1

Gledati samo trenutni RAM.

---

Problem:

Ne vidi:

```
allocation rate
```

---

## Greška #2

Smanjivati GC bez razloga.

---

GC nije problem.

Često je simptom.

---

## Greška #3

Optimizovati male alokacije.

---

Prvo:

```
najveći allocator
```

---

# Memory Optimization Pravila

✅ Smanji nepotrebne alokacije.

✅ Reuse objekte kada ima smisla.

✅ Koristi pool za specifične slučajeve.

✅ Ograniči cache.

✅ Prati allocation rate.

---

# Mentalni model

Heap profile:

```
Šta postoji?
```

---

Allocation profile:

```
Šta stalno nastaje?
```

---

Escape analysis:

```
Zašto je na heap-u?
```

---

GC:

```
Koliko to košta?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Memory Profiling

✅ Heap profiling

✅ In-use vs Alloc

✅ allocation profiling

✅ GC vezu

✅ escape analysis povezivanje

✅ memory leak analizu

---

# Performance i Profiling — Concurrency Profiling

> **Modul:** #4 — Advanced Go Concurrency

---

# 📚 Sadržaj

- Goroutine Profiling
- Goroutine leak analiza
- Block Profiling
- Mutex Profiling
- Scheduler analiza
- `runtime/trace`
- Concurrency bottleneck analiza

---

# Uvod

Concurrency problem često izgleda ovako:

```
CPU nije 100%

Memory je normalan

Ali aplikacija je spora
```

---

Zašto?

Zato što Goroutines mogu biti:

- blokirane,
- čekati lock,
- čekati channel,
- loše raspoređene.

---

# Goroutine Profiling

Odgovara na pitanje:

```
Koje Goroutines postoje i gde čekaju?
```

---

Dobijamo informacije o:

- broju Goroutines,
- stack trace-u,
- mestu blokiranja.

---

# Osnovni primer

Import:

```go
import _ "net/http/pprof"
```

---

Pokrenemo server:

```go
http.ListenAndServe(
	":6060",
	nil,
)
```

---

Goroutine profil:

```bash
go tool pprof \
http://localhost:6060/debug/pprof/goroutine
```

---

# Goroutine Leak

Najčešći concurrency problem.

---

Primer:

```go
func worker(){

	ch := make(chan int)

	go func(){

		for {

			value := <-ch

			process(value)

		}

	}()

}
```

---

Problem:

Goroutine čeka zauvek.

---

Nema:

- cancel,
- close,
- shutdown signal.

---

Rezultat:

```
Goroutines ↑
```

---

# Simptom Goroutine Leak-a

Vremenom:

```
100

↓

1000

↓

10000 Goroutines
```

---

Posledice:

- veća memorija,
- scheduler overhead,
- sporiji sistem.

---

# Pronalaženje Leak-a

Koraci:

---

## 1. Broj Goroutines

```go
runtime.NumGoroutine()
```

---

Primer:

```go
fmt.Println(
	runtime.NumGoroutine(),
)
```

---

Ako konstantno raste:

problem.

---

## 2. Goroutine Profile

Tražimo:

```
vekoma iste stack trace-ove
```

---

Primer:

```
worker()

↓

chan receive
```

---

Znači:

Goroutine čeka.

---

# Rešenje Goroutine Lifecycle-a

Svaka Goroutine treba:

```
Start

↓

Work

↓

Receive cancel

↓

Exit
```

---

Primer:

```go
func worker(
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

# Block Profiling

Block profile pokazuje:

```
Gde Goroutines čekaju?
```

---

Čekanja:

- channel receive,
- channel send,
- Mutex,
- WaitGroup,
- Cond.

---

Omogućavanje:

```go
runtime.SetBlockProfileRate(1)
```

---

Analiza:

```bash
go tool pprof \
block.prof
```

---

# Primer Block problema

Imamo:

```go
result := <-ch
```

---

Profil:

```
chan receive

80%
```

---

Zaključak:

Goroutine više vremena čeka nego što radi.

---

# Mutex Profiling

Mutex profile pokazuje:

```
Koji lock pravi problem?
```

---

Omogućavanje:

```go
runtime.SetMutexProfileFraction(1)
```

---

Analiza:

```bash
go tool pprof \
mutex.prof
```

---

# Primer Mutex Contention-a

Kod:

```go
mu.Lock()

update()

mu.Unlock()
```

---

Mnogo Goroutines:

```
G1
G2
G3
G4
```

čekaju:

```
isti Mutex
```

---

Profil:

```
sync.(*Mutex).Lock

70%
```

---

Problem:

Lock contention.

---

# Kako rešiti Mutex Contention?

Opcije:

---

## 1. Smanjiti kritičnu sekciju

Loše:

```go
Lock()

databaseCall()

Unlock()
```

---

Bolje:

```go
databaseCall()

Lock()

update()

Unlock()
```

---

## 2. Granularni lock

Loše:

```text
jedan globalni Mutex
```

---

Bolje:

```text
više manjih lock-ova
```

---

## 3. Channel ownership

Umesto:

```
shared state + Mutex
```

nekad:

```
owner Goroutine + Channel
```

---

# Scheduler Profiling

Go scheduler odlučuje:

```
Koja Goroutine ide na koji thread?
```

---

Problemi:

- previše Goroutines,
- previše context switching-a,
- blokiranje.

---

Za detaljnu analizu koristi se:

```go
runtime/trace
```

---

# runtime/trace

Omogućava:

- scheduler događaje,
- Goroutine lifecycle,
- GC događaje,
- network blocking.

---

Primer:

```go
trace.Start(file)

defer trace.Stop()
```

---

Analiza:

```bash
go tool trace trace.out
```

---

Dobijamo:

```
Timeline
```

---

# Trace prikazuje

## Goroutines

```
start

run

block

unblock
```

---

## Scheduler

```
P

↓

M

↓

G
```

---

## GC

```
GC start

mark

sweep
```

---

# Primer: Worker Pool problem

Simptom:

```
100 workers

ali spor sistem
```

---

Profil:

```
goroutines blocked

channel receive
```

---

Zaključak:

Nema dovoljno poslova.

---

Drugi slučaj:

```
workers rade

CPU visok
```

---

Profil:

```
scheduler overhead
```

---

Zaključak:

Previše konkurentnosti.

---

# Concurrency Bottleneck Workflow

```text
Problem

↓

Goroutine count

↓

Goroutine profile

↓

Block profile

↓

Mutex profile

↓

Trace

↓

Optimization
```

---

# Production Monitoring

Korisne metrike:

---

## Goroutines

```go
runtime.NumGoroutine()
```

---

## Memory

- heap size,
- allocations.

---

## GC

- pause time,
- frequency.

---

## Locks

- contention.

---

# Senior Concurrency Checklist

Pitaj:

---

## Lifecycle

```
Da li se Goroutines završavaju?
```

---

## Blocking

```
Gde čekaju?
```

---

## Locking

```
Koji lock usporava?
```

---

## Scheduling

```
Da li imamo previše konkurentnosti?
```

---

# Mentalni model

CPU Profile:

```
Gde radimo?
```

---

Memory Profile:

```
Šta kreiramo?
```

---

Concurrency Profile:

```
Zašto čekamo?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Goroutine profiling

✅ goroutine leak analizu

✅ Block profiling

✅ Mutex profiling

✅ scheduler analizu

✅ runtime trace

✅ concurrency bottleneck workflow

---

# Performance i Profiling — Production Workflow

---

# 📚 Sadržaj

- Performance debugging workflow
- Problem → merenje → optimizacija
- Kombinovanje benchmark-a i profiling-a
- Production profiling strategija
- Case study optimizacije
- Performance checklist

---

# Uvod

Najveća greška kod optimizacije:

```
Pretpostavka
```

---

Ispravan proces:

```
Observe

↓

Measure

↓

Analyze

↓

Optimize

↓

Verify
```

---

# Performance Investigation Workflow

Kompletan tok:

```text
Problem

↓

Metrike

↓

Benchmark

↓

Profiling

↓

Root Cause

↓

Optimization

↓

Regression Test
```

---

# Korak #1

# Identifikacija problema

Primer:

API latency raste.

Metrike:

```
p50: 50ms

p95: 500ms

p99: 2s
```

---

Pitanje:

```
Gde nastaje problem?
```

---

Ne krećemo odmah u kod.

---

# Korak #2

# Posmatranje sistema

Pratimo:

## CPU

```
Da li je CPU visok?
```

---

## Memory

```
Da li heap raste?
```

---

## Goroutines

```
Da li broj raste?
```

---

## GC

```
Da li GC troši vreme?
```

---

# Korak #3

# Izbor pravog alata

---

## CPU problem

Koristimo:

```
CPU Profile
```

---

Tražimo:

```
hot path
```

---

## Memory problem

Koristimo:

```
Heap Profile
```

---

Tražimo:

```
allocator
```

---

## Goroutine problem

Koristimo:

```
Goroutine Profile
```

---

Tražimo:

```
leak/block
```

---

## Lock problem

Koristimo:

```
Mutex Profile
```

---

# Performance Matrix

| Simptom | Alat | Tražimo |
|---|---|---|
| CPU visok | CPU Profile | Hot functions |
| RAM raste | Heap Profile | Retained objects |
| GC visok | Alloc Profile | Allocation rate |
| Spor concurrency | Block Profile | Waiting |
| Lock problem | Mutex Profile | Contention |
| Čudno ponašanje | Trace | Timeline |

---

# Benchmark + Profiling kombinacija

Benchmark:

```text
Da li je promena bolja?
```

---

Profiling:

```text
Zašto je bolja?
```

---

Primer:

Pre:

```
Benchmark:

1000 ns/op
```

---

Profil:

```
json.Marshal 70%
```

---

Optimizacija:

```
custom encoder
```

---

Posle:

```
400 ns/op
```

---

# Case Study #1

# CPU Bottleneck

Problem:

```
API spor
```

---

Metrike:

```
CPU 95%
```

---

CPU Profile:

```
regexp.Match

60%
```

---

Analiza:

Regex se kompajlira svaki request.

---

Pre:

```go
regexp.MatchString(
	pattern,
	input,
)
```

---

Posle:

```go
var re =
	regexp.MustCompile(
		pattern,
	)
```

---

Rezultat:

```
CPU ↓

Latency ↓
```

---

# Case Study #2

# Memory Pressure

Problem:

```
GC CPU visok
```

---

Heap:

```
normalan
```

---

Alloc profile:

```
fmt.Sprintf

50%
```

---

Problem:

Mnogo privremenih stringova.

---

Optimizacija:

```go
strings.Builder
```

---

Rezultat:

```
allocs/op ↓

GC ↓
```

---

# Case Study #3

# Goroutine Leak

Problem:

Posle 24h:

```
memory raste
```

---

Metrika:

```
goroutines:

100

↓

50000
```

---

Goroutine profile:

```
worker()

chan receive
```

---

Problem:

Worker nikada ne dobija shutdown signal.

---

Rešenje:

```go
select {

case <-ctx.Done():
	return

case job := <-jobs:
	process(job)

}
```

---

# Production Profiling Strategija

Profiling ne treba stalno uključivati.

---

Bolji pristup:

## On-demand profiling

```
Problem

↓

Aktiviraj profile

↓

Analiza

↓

Isključi
```

---

# Profiling u CI/CD

Koristi:

## Benchmark regression

Primer:

```
pre:

500 ns/op


posle:

700 ns/op
```

---

Alarm:

```
40% sporije
```

---

# Performance Testing Levels

---

# Unit Benchmark

Meri:

```
jednu funkciju
```

---

# Micro Benchmark

Meri:

```
algoritam
```

---

# Load Test

Meri:

```
ceo servis
```

---

# Production Profiling

Meri:

```
realan workload
```

---

# Senior Optimization Rules

---

## Pravilo #1

Ne optimizuj bez merenja.

---

## Pravilo #2

Najveći bottleneck prvo.

---

## Pravilo #3

Performance nije samo brzina.

Gledamo:

- latency,
- throughput,
- memory,
- GC,
- concurrency.

---

## Pravilo #4

Optimizacija mora biti proverena.

```
Before

↓

Change

↓

After
```

---

# Finalni Performance Model

Senior Go developer razmišlja:

```text
Da li je spor?

↓

Izmeri

↓

Zašto je spor?

↓

Profil

↓

Koji je bottleneck?

↓

Promena

↓

Dokaz
```

---

# 📋 Rezime Modula #4.10

Naučili smo:

✅ Go benchmark sistem

✅ CPU profiling

✅ Memory profiling

✅ Goroutine profiling

✅ Block profiling

✅ Mutex profiling

✅ runtime trace

✅ production workflow

---

# 🎯 Modul #4.10 završen

Prešli smo:

```
Modul #4.9
Concurrency Anti-Patterns

        ↓

Modul #4.10
Performance & Profiling
```

Sledeći nivo:

```
Kako dizajnirati
production-grade
Go concurrency sisteme
```

---

### ➡️ Sledeća lekcija **[**Advanced Concurrency Architecture**](11-advanced-concurrency-architecture.md)**

Obuhvatiće:

- Worker Pool dizajn
- Pipeline arhitekturu
- Fan-in/Fan-out napredni obrasci
- Backpressure strategije
- Rate limiting
- Production concurrency patterns