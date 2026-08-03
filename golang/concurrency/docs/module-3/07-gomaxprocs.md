# GOMAXPROCS — Uvod, koncept i veza sa Go Scheduler-om

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- Šta je GOMAXPROCS
- Zašto postoji GOMAXPROCS
- Veza sa Go Scheduler-om
- Veza sa Processor (`P`) komponentom
- Razlika između Goroutines i paralelnog izvršavanja
- Default vrednost
- Prvi eksperimenti

---

# Uvod

U prethodnoj lekciji obradili smo:

```
G-M-P model
```

gde smo naučili:

```
G = Goroutine

M = Machine (OS Thread)

P = Processor
```

---

Posebno je važan deo:

```
P = scheduling resource
```

---

Broj aktivnih `P` objekata kontroliše:

```go
GOMAXPROCS
```

---

# Šta je GOMAXPROCS?

`GOMAXPROCS` definiše:

> maksimalan broj Processor (`P`) objekata koji mogu istovremeno izvršavati Go kod.

---

Primer:

```go
runtime.GOMAXPROCS(4)
```

---

Go runtime ima:

```
P1

P2

P3

P4
```

---

To znači:

maksimalno:

```
4 Goroutines
```

mogu istovremeno izvršavati CPU instrukcije.

---

# Važna razlika

GOMAXPROCS nije:

```
broj Goroutines
```

---

Nije:

```
broj Threads
```

---

Nije:

```
broj aplikacionih taskova
```

---

On kontroliše:

```
broj aktivnih scheduler Processor-a
```

---

# Primer

Kod:

```go
for i := 0; i < 1000; i++ {

	go work()

}
```

---

Kreira:

```
1000 Goroutines
```

---

Ali ako imamo:

```go
GOMAXPROCS = 4
```

---

Istovremeno:

```
G1 → CPU

G2 → CPU

G3 → CPU

G4 → CPU
```

---

Ostale:

```
G5...G1000
```

čekaju scheduler.

---

# GOMAXPROCS i Go Scheduler

Veza:

```
Goroutines

      ↓

Scheduler

      ↓

P count

      ↓

OS Threads

      ↓

CPU
```

---

GOMAXPROCS utiče na:

```
koliko P postoji
```

---

Primer:

```text
GOMAXPROCS = 1
```

---

Model:

```
P1

 |

M1

 |

CPU
```

---

Samo jedna Goroutine može biti CPU-active.

---

# GOMAXPROCS = 4

Model:

```
P1 → M1 → CPU Core 1

P2 → M2 → CPU Core 2

P3 → M3 → CPU Core 3

P4 → M4 → CPU Core 4
```

---

Dobijamo:

```
pravi paralelizam
```

---

# Concurrency bez paralelizma

Primer:

```
GOMAXPROCS = 1
```

---

Imamo:

```
1000 Goroutines
```

---

Program radi.

---

Ali:

```
samo jedan CPU execution slot
```

---

To je:

```
Concurrency
```

---

Nije:

```
Parallelism
```

---

# Paralelizam

Primer:

```
GOMAXPROCS = 8
```

---

Imamo:

```
8 aktivnih P
```

---

CPU:

```
Core 1

Core 2

Core 3

Core 4

Core 5

Core 6

Core 7

Core 8
```

---

Više Goroutines može stvarno raditi istovremeno.

---

# Default GOMAXPROCS

Ako ništa ne podesimo:

```go
runtime.GOMAXPROCS()
```

---

Go runtime bira vrednost.

---

Uobičajeno:

```
broj dostupnih CPU logical cores
```

---

Primer:

Mašina:

```
8 logical CPUs
```

---

Default:

```
GOMAXPROCS = 8
```

---

# Čitanje trenutne vrednosti

Kod:

```go
package main

import (
	"fmt"
	"runtime"
)

func main(){

	fmt.Println(
		runtime.GOMAXPROCS(0),
	)

}
```

---

Rezultat:

primer:

```
8
```

---

# Zašto koristiti 0?

Poseban slučaj:

```go
runtime.GOMAXPROCS(0)
```

---

Znači:

```
vrati trenutnu vrednost

ne menjaj ništa
```

---

---

# Promena vrednosti

Primer:

```go
runtime.GOMAXPROCS(2)
```

---

Sada:

```
2 aktivna P
```

---

Preporuka:

ne menjati bez razloga.

---

# Eksperiment

Primer:

```go
package main

import (
	"fmt"
	"runtime"
)

func main(){

	fmt.Println(
		"CPU:",
		runtime.NumCPU(),
	)


	fmt.Println(
		"GOMAXPROCS:",
		runtime.GOMAXPROCS(0),
	)

}
```

---

Dobijamo:

```
CPU: 8

GOMAXPROCS: 8
```

---

# runtime.NumCPU()

Ovo vraća:

```
broj dostupnih CPU logical cores
```

---

Primer:

```go
runtime.NumCPU()
```

---

Ne znači:

```
broj aktivnih P
```

---

Za to:

```go
runtime.GOMAXPROCS(0)
```

---

# Česta greška

Pogrešno:

```go
goroutines := runtime.NumCPU()
```

---

Zašto?

Zato što:

```
broj Goroutines

nema veze sa brojem CPU
```

---

Možemo imati:

```
CPU = 8

Goroutines = 1 000 000
```

---

# Kada GOMAXPROCS utiče na performanse?

Najviše kod:

```
CPU-bound workload
```

---

Primer:

- matematički proračuni,
- kompresija,
- enkripcija,
- obrada slike,
- simulacije.

---

Manje kod:

```
IO-bound workload
```

---

Primer:

- HTTP,
- database,
- network.

---

# Mentalni model

Zapamti:

```
Goroutines

=

koliko poslova imamo


GOMAXPROCS

=

koliko poslova može fizički raditi paralelno
```

---

Još kraće:

```
G = Task

P = CPU execution slot
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je GOMAXPROCS,
- zašto postoji,
- kako se povezuje sa Scheduler-om,
- odnos sa `P` komponentom GMP modela,
- razliku između concurrency i parallelism,
- kako čitati i menjati GOMAXPROCS.

---

# GOMAXPROCS — Veza sa GMP modelom i Processor (`P`) komponentom

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- GOMAXPROCS unutar GMP arhitekture
- Detaljna uloga Processor (`P`) komponente
- Kako runtime kreira P objekte
- Odnos P, M i CPU core-ova
- Šta se dešava pri promeni GOMAXPROCS vrednosti
- Smanjivanje i povećavanje broja P objekata
- Scheduler ponašanje

---

# Uvod

U prethodnom delu naučili smo:

```
GOMAXPROCS

=

broj aktivnih Processor (`P`) objekata
```

---

Sada ćemo detaljnije analizirati:

```
GOMAXPROCS → P → M → CPU
```

---

# GMP model podsećanje

Go runtime scheduler koristi:

```
        G (Goroutine)

             |

             v

        P (Processor)

             |

             v

        M (Machine)

             |

             v

           CPU
```

---

Svaka komponenta ima svoju ulogu.

---

# G — Goroutine

Predstavlja:

```
šta treba izvršiti
```

---

Primer:

```go
go process()
```

---

Runtime kreira:

```
G1
```

---

Možemo imati:

```
milione G objekata
```

---

Ali oni ne rade svi istovremeno.

---

# M — Machine

Predstavlja:

```
OS Thread
```

---

M izvršava Go kod.

---

Ali postoji ograničenje:

```
M mora imati P
```

---

Bez P:

```
M ne može uzeti Goroutine
```

---

# P — Processor

P predstavlja:

```
scheduler resurs
```

---

P poseduje:

- local run queue,
- scheduling state,
- runtime podatke.

---

Najvažnije:

```
P omogućava M-u da izvršava G
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

# GOMAXPROCS i broj P objekata

Kada podesimo:

```go
runtime.GOMAXPROCS(4)
```

---

Runtime održava:

```
4 P objekta
```

---

Model:

```
P1

P2

P3

P4
```

---

Svaki P može imati:

```
jedan aktivan M
```

---

Dobijamo:

```
P1 → M1 → CPU

P2 → M2 → CPU

P3 → M3 → CPU

P4 → M4 → CPU
```

---

# Zašto P nije isto što i CPU?

Važno pitanje.

---

CPU je:

```
fizički hardver
```

---

P je:

```
Go runtime struktura
```

---

Primer:

Mašina:

```
16 CPU cores
```

---

Ali:

```go
GOMAXPROCS(4)
```

---

Rezultat:

```
16 CPU postoji

ali Go koristi 4 P
```

---

---

# Veza između P i CPU core-ova

Najčešći slučaj:

```
P ≈ CPU core
```

---

Primer:

```
CPU = 8

GOMAXPROCS = 8
```

---

Ali nije pravilo.

---

Možemo imati:

```
CPU = 8

GOMAXPROCS = 2
```

---

ili:

```
CPU = 8

GOMAXPROCS = 16
```

---

---

# Šta se dešava kada povećamo GOMAXPROCS?

Primer:

Pre:

```
GOMAXPROCS = 2
```

---

Imamo:

```
P1

P2
```

---

CPU:

```
2 aktivna execution slot-a
```

---

Promena:

```go
runtime.GOMAXPROCS(4)
```

---

Runtime kreira dodatne:

```
P3

P4
```

---

Novo stanje:

```
P1

P2

P3

P4
```

---

Moguće više paralelnog rada.

---

# Ali povećanje nije uvek bolje

Primer:

```
CPU = 4
```

---

Postavimo:

```go
GOMAXPROCS(100)
```

---

Dobijamo:

```
100 P objekata
```

---

Problem:

- više scheduling overhead-a,
- više context switching-a,
- veća konkurencija za resurse.

---

Više nije:

```
više P = više performansi
```

---

# Šta se dešava kada smanjimo GOMAXPROCS?

Primer:

Pre:

```
P1

P2

P3

P4
```

---

Promena:

```go
runtime.GOMAXPROCS(1)
```

---

Runtime smanjuje:

```
aktivan broj P
```

---

Rezultat:

```
P1
```

---

Ostali poslovi:

```
čekaju scheduler
```

---

# Primer CPU-bound workload-a

Kod:

```go
func calculate(){

	for i:=0;i<1_000_000_000;i++{

	}

}
```

---

Pokrenemo:

```go
go calculate()
go calculate()
go calculate()
go calculate()
```

---

## GOMAXPROCS = 1

```
G1
 |
CPU

G2
 |
wait

G3
 |
wait

G4
 |
wait
```

---

Nema pravog paralelizma.

---

## GOMAXPROCS = 4

```
G1 → CPU1

G2 → CPU2

G3 → CPU3

G4 → CPU4
```

---

Sada imamo:

```
parallel execution
```

---

# Primer IO-bound workload-a

Kod:

```go
func request(){

	http.Get(url)

}
```

---

Većinu vremena:

```
čekanje network-a
```

---

Goroutine:

```
WAITING
```

---

Scheduler može koristiti CPU za druge G.

---

Zato povećanje GOMAXPROCS često malo menja rezultat.

---

# GOMAXPROCS i Local Run Queue

Svaki P ima:

```
local queue
```

---

Primer:

```
P1

[
 G1,
 G2
]


P2

[
 G3,
 G4
]
```

---

Više P znači:

- više lokalnih queue-ova,
- više paralelnog rada,
- više scheduler koordinacije.

---

---

# Work Stealing i GOMAXPROCS

Iz prethodne lekcije:

```
P1 ima posao

P2 nema posao
```

---

Scheduler:

```
P2 steal-uje od P1
```

---

Ako povećamo broj P:

```
više potencijalnih stealera
```

---

Ali:

previše P može povećati overhead.

---

# GOMAXPROCS i Worker Pool

Primer:

Imamo:

```
1000 jobs
```

---

Worker pool:

```
8 workers
```

---

Ako:

```
GOMAXPROCS = 8
```

---

Dobro iskorišćen CPU.

---

Ali:

```
1000 workers

GOMAXPROCS = 2
```

---

Ne znači:

```
1000 paralelnih izvršavanja
```

---

Scheduler ih ograničava.

---

# Mentalni model

Zapamti:

```
GOMAXPROCS

↓

broj P

↓

broj aktivnih execution slot-ova

↓

koliko Goroutines može raditi paralelno
```

---

Cela slika:

```
Goroutines

     ↓

Processor (P)

     ↓

Machine (M)

     ↓

CPU
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako GOMAXPROCS funkcioniše unutar GMP modela,
- detaljnu ulogu `P`,
- razliku između P i CPU-a,
- kako povećanje i smanjenje GOMAXPROCS utiče na scheduler,
- odnos sa CPU-bound i IO-bound workload-ima,
- vezu sa Work Stealing algoritmom.

---

# GOMAXPROCS — CPU Paralelizam, Benchmark i Merenje Performansi

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- GOMAXPROCS i pravi paralelizam
- CPU-bound workload
- IO-bound workload
- Uticaj broja CPU jezgara
- Benchmark sa različitim GOMAXPROCS vrednostima
- Throughput vs latency
- Zašto veći GOMAXPROCS nije uvek bolji
- Kako meriti performanse

---

# Uvod

U prethodnom delu naučili smo:

```
GOMAXPROCS

↓

broj P objekata

↓

broj aktivnih execution slot-ova
```

---

Sada prelazimo na praktično pitanje:

> Kako GOMAXPROCS utiče na performanse programa?

---

Odgovor zavisi od:

- tipa workload-a,
- broja CPU jezgara,
- količine blokiranja,
- količine shared state-a.

---

# CPU paralelizam

CPU paralelizam znači:

```
više instrukcija

u istom trenutku

na različitim CPU jezgrima
```

---

Primer:

Imamo:

```
4 CPU cores
```

---

I:

```
GOMAXPROCS = 4
```

---

Moguće:

```
G1 → Core 1

G2 → Core 2

G3 → Core 3

G4 → Core 4
```

---

To je:

```
true parallelism
```

---

# CPU-bound workload

CPU-bound znači:

program većinu vremena:

```
računa
```

---

Primeri:

- hashing,
- encryption,
- compression,
- image processing,
- matematičke simulacije.

---

Primer:

```go
func calculate(){

	var x uint64

	for i := 0; i < 1_000_000_000; i++ {

		x += uint64(i)

	}

}
```

---

Ovde:

```
CPU radi konstantno
```

---

# Efekat GOMAXPROCS kod CPU-bound rada

Pretpostavimo:

```
CPU = 8 cores
```

---

## GOMAXPROCS = 1

Model:

```
G1

↓

CPU Core 1
```

---

Rezultat:

```
nema paralelizma
```

---

---

## GOMAXPROCS = 4

Model:

```
G1 → Core1

G2 → Core2

G3 → Core3

G4 → Core4
```

---

Rezultat:

```
veći throughput
```

---

---

## GOMAXPROCS = 8

Iskorišćeno:

```
svih 8 core-ova
```

---

Obično najbolji rezultat.

---

---

## GOMAXPROCS = 32

Ako imamo:

```
8 CPU cores
```

---

Dobijamo:

```
više scheduling overhead-a
```

---

Rezultat može biti:

```
sporije
```

---

# IO-bound workload

IO-bound znači:

program često čeka.

---

Primer:

```go
response, err :=
	http.Get(url)
```

---

Većina vremena:

```
čekanje network-a
```

---

Tok:

```
G1

↓

WAITING

↓

G2 radi
```

---

CPU nije glavni problem.

---

# Efekat GOMAXPROCS kod IO-bound rada

Primer:

```
10000 HTTP request Goroutines
```

---

Čak i:

```
GOMAXPROCS = 2
```

može biti dovoljno.

---

Zašto?

Zato što:

```
Goroutines provode vreme čekajući
```

---

---

# Throughput vs Latency

Važan koncept.

---

## Throughput

Koliko posla završimo u jedinici vremena.

Primer:

```
10000 request/sec
```

---

---

## Latency

Koliko traje jedan zahtev.

Primer:

```
50ms response time
```

---

Povećanje GOMAXPROCS može:

poboljšati:

```
throughput
```

---

ali pogoršati:

```
latency
```

---

# Primer contention problema

Imamo:

```go
var counter int
```

---

Mnogo Goroutines:

```
G1

G2

G3

G4
```

---

Sve rade:

```go
counter++
```

---

Potrebna zaštita:

```go
mutex.Lock()
```

---

Sada:

više P ne znači automatski:

```
više performansi
```

---

Jer postoji:

```
lock contention
```

---

# Benchmark primer

Kreiramo CPU test:

```go
func BenchmarkCPU(
	b *testing.B,
){

	for i:=0;i<b.N;i++{

		calculate()

	}

}
```

---

Pokretanje:

```bash
go test -bench .
```

---

---

# Testiranje različitih GOMAXPROCS vrednosti

Primer:

```bash
GOMAXPROCS=1 go test -bench .

GOMAXPROCS=2 go test -bench .

GOMAXPROCS=4 go test -bench .

GOMAXPROCS=8 go test -bench .
```

---

Dobijamo:

```
vreme izvršavanja
```

---

Upoređujemo:

```
1

2

4

8
```

---

# Benchmark rezultat (primer)

Mašina:

```
8 cores
```

---

Mogući rezultat:

```
GOMAXPROCS=1

10s


GOMAXPROCS=4

3s


GOMAXPROCS=8

2s


GOMAXPROCS=32

2.5s
```

---

Zaključak:

```
više nije uvek bolje
```

---

# Kako pronaći optimalnu vrednost?

Proces:

```
1. Identifikuj workload

2. Napravi benchmark

3. Testiraj vrednosti

4. Izmeri rezultat

5. Izaberi najbolju
```

---

Ne:

```
pogađati
```

---

# GOMAXPROCS i broj Goroutines

Česta zabuna:

```
100 Goroutines

=

100 CPU poslova
```

---

Netačno.

---

Primer:

```
GOMAXPROCS = 4

Goroutines = 100000
```

---

Samo:

```
4 mogu raditi istovremeno
```

---

Ostale:

```
scheduler queue
```

---

# GOMAXPROCS i Worker Pool dizajn

Primer:

```
Jobs

 |

Worker Pool

 |

CPU
```

---

CPU-bound:

Često:

```
broj worker-a ≈ GOMAXPROCS
```

---

Primer:

```
8 CPU cores

8 workers
```

---

Ali:

nije univerzalno pravilo.

---

IO-bound:

Može biti:

```
stotine worker-a
```

---

---

# Eksperiment

Primer:

```go
func worker(){

	for i:=0;i<1000000000;i++{

	}

}
```

---

Pokrenuti:

```go
for i:=0;i<8;i++{

	go worker()

}
```

---

Testirati:

```
GOMAXPROCS=1

GOMAXPROCS=2

GOMAXPROCS=4

GOMAXPROCS=8
```

---

Posmatrati:

- vreme,
- CPU usage,
- broj thread-ova.

---

# Monitoring

Korisni alati:

---

## CPU profiling

```bash
go test -cpuprofile cpu.out
```

---

---

## Trace

```bash
go test -trace trace.out
```

---

---

## Runtime metrics

Paket:

```go
runtime/metrics
```

---

Može pokazati:

- scheduler podatke,
- GC,
- memory.

---

# Česte zablude

---

## Zabluda 1

"Dupliranje GOMAXPROCS duplira brzinu."

Ne.

---

Zavisi od:

- workload-a,
- lockova,
- memorije.

---

---

## Zabluda 2

"Za server stavim maksimalni GOMAXPROCS."

Ne.

---

Default je često najbolji.

---

---

## Zabluda 3

"Više Goroutines znači više paralelizma."

Ne.

---

Goroutines daju:

```
concurrency
```

---

GOMAXPROCS omogućava:

```
parallel execution
```

---

# Mentalni model

Zapamti:

```
CPU-bound

↓

GOMAXPROCS važan


IO-bound

↓

scheduler + waiting važniji
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako GOMAXPROCS utiče na CPU paralelizam,
- razliku CPU-bound i IO-bound workload-a,
- throughput vs latency,
- kako benchmarkirati različite vrednosti,
- zašto veći GOMAXPROCS nije uvek bolji,
- kako meriti realne performanse.

---

# GOMAXPROCS — Container Okruženja, CPU Limits i Production Tuning

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- GOMAXPROCS na serverima
- Problem različitih CPU okruženja
- Docker CPU limits
- Kubernetes CPU requests i limits
- Zašto runtime može pogrešno proceniti CPU
- Uticaj na scheduler
- Production tuning strategije
- Automatsko podešavanje GOMAXPROCS

---

# Uvod

Do sada smo naučili:

```
GOMAXPROCS

↓

broj P objekata

↓

broj paralelnih Go execution slotova
```

---

Na lokalnoj mašini:

```
CPU = 8

GOMAXPROCS = 8
```

---

Ali production okruženje je često drugačije.

Primer:

```
Host machine:

32 CPU cores


Container:

2 CPU cores
```

---

Pitanje:

> Koliko CPU-a Go runtime vidi?

---

# Problem u virtualizovanim okruženjima

Tradicionalno:

```
Go runtime

↓

OS

↓

CPU informacije
```

---

Runtime pita:

```go
runtime.NumCPU()
```

---

Dobije:

```
koliko CPU resursa OS prijavljuje
```

---

Problem:

u container okruženju ta informacija može biti drugačija od realnog limita.

---

# Primer Docker okruženja

Host:

```
32 cores
```

---

Pokrenemo:

```bash
docker run my-app
```

---

Container vidi:

```
32 CPU
```

---

Go:

```
GOMAXPROCS = 32
```

---

Ali aplikacija realno ima:

```
2 CPU quota
```

---

Rezultat:

```
32 P objekta

za 2 CPU-a
```

---

# Posledice previsokog GOMAXPROCS-a

Previše aktivnih P objekata može izazvati:

---

## 1. Scheduling overhead

Više:

```
P

+

M

+

context switching
```

---

Runtime troši više vremena na organizaciju rada.

---

## 2. CPU contention

Više thread-ova želi:

```
isti CPU resurs
```

---

Rezultat:

```
borba za CPU
```

---

## 3. Lošija latency

Aplikacija može imati:

```
veći response time
```

---

Iako ima dovoljno Goroutines.

---

# Kubernetes primer

Imamo pod:

```yaml
resources:
  limits:
    cpu: "2"
```

---

Znači:

```
maksimalno 2 CPU cores
```

---

Ali node:

```
64 cores
```

---

Ako Go vidi node informacije:

može dobiti:

```
GOMAXPROCS = 64
```

---

Aplikacija realno:

```
2 CPU
```

---

Neusklađenost:

```
GOMAXPROCS ≠ realni CPU limit
```

---

# CPU request vs CPU limit

U Kubernetes-u postoje dva koncepta.

---

## CPU request

Garantovani resurs.

Primer:

```yaml
requests:
  cpu: "1"
```

---

Znači:

```
minimum 1 CPU
```

---

## CPU limit

Maksimum.

Primer:

```yaml
limits:
  cpu: "4"
```

---

Znači:

```
najviše 4 CPU
```

---

Za GOMAXPROCS je važniji:

```
CPU limit
```

---

# Rešenje problema

Postoje tri pristupa.

---

# Pristup 1 — Ručno podešavanje

Primer:

```go
runtime.GOMAXPROCS(2)
```

---

Dobro za:

- kontrolisana okruženja,
- specifične deployment-e.

---

Mana:

svaka promena zahteva novu konfiguraciju.

---

# Pristup 2 — Environment konfiguracija

Primer:

```bash
GOMAXPROCS=4
```

---

Aplikacija koristi:

```
deployment config
```

---

Prednost:

nema promene koda.

---

---

# Pristup 3 — Automatsko podešavanje

Moderne production aplikacije često koriste:

```
automatsko čitanje CPU limita
```

---

Princip:

```
container CPU limit

↓

izračunavanje

↓

GOMAXPROCS
```

---

Primer biblioteke:

:contentReference[oaicite:0]{index=0}

---

Ona omogućava:

```
automatsko usklađivanje
```

između:

```
container limita

i

Go scheduler-a
```

---

# Production preporuka

Za moderne cloud sisteme:

najčešće:

```
ne dirati GOMAXPROCS ručno
```

---

Bolji pristup:

```
pravilno definisati CPU limits

+

automatski tuning
```

---

# Primer servisnog workload-a

HTTP API:

```
10000 request/sec
```

---

CPU limit:

```
4 cores
```

---

Dobro:

```
GOMAXPROCS ≈ 4
```

---

Loše:

```
GOMAXPROCS = 64
```

---

---

# GOMAXPROCS i autoscaling

Važan scenario.

Imamo:

```
Kubernetes HPA
```

---

Poveća broj podova:

```
2 pods

↓

10 pods
```

---

Svaki pod ima:

```
CPU limit 2
```

---

Svaki pod treba:

```
GOMAXPROCS ≈ 2
```

---

Ne:

```
node CPU count
```

---

# Production debugging

Ako servis ima:

## Visoku CPU potrošnju

Proveriti:

```
GOMAXPROCS

CPU limits

thread count
```

---

---

## Lošu latency

Proveriti:

```
previše aktivnih threadova

scheduler contention
```

---

---

## Nestabilne performanse

Proveriti:

```
container throttling
```

---

# CPU throttling

Kubernetes može ograničiti CPU.

Primer:

limit:

```
2 CPU
```

---

Aplikacija pokušava:

```
4 CPU
```

---

Kernel throttluje:

```
proces
```

---

Rezultat:

- veća latency,
- sporiji request-i.

---

# GOMAXPROCS i realni CPU resurs

Mentalni model:

Ne gledaj:

```
koliko CPU host ima
```

---

Gledaj:

```
koliko CPU aplikacija sme da koristi
```

---

Formula:

```
Optimalni GOMAXPROCS

≈

CPU quota aplikacije
```

---

# Production checklist

Pre deployment-a proveriti:

✅ CPU limits definisan

✅ GOMAXPROCS odgovara limitu

✅ benchmark u realnom okruženju

✅ monitoring scheduler ponašanja

✅ latency merenje

---

# Česte zablude

---

## Zabluda 1

"Više CPU na serveru znači više CPU za container."

Nije uvek tačno.

---

---

## Zabluda 2

"Go automatski zna Kubernetes limit."

Zavisi od verzije runtime-a i okruženja.

---

---

## Zabluda 3

"Veći GOMAXPROCS povećava performanse."

Samo ako postoji realan CPU kapacitet.

---

# Mentalni model

Zapamti:

```
Host CPU

↓

Container limit

↓

GOMAXPROCS

↓

P

↓

M

↓

CPU
```

---

# 📋 Rezime

U ovom delu naučili smo:

- probleme GOMAXPROCS-a u container okruženjima,
- Docker i Kubernetes CPU limite,
- razliku između host CPU-a i dostupnog CPU-a aplikacije,
- posledice previsokog GOMAXPROCS-a,
- production tuning pristupe,
- važnost automatskog podešavanja.

---

# GOMAXPROCS — Advanced Tuning, Best Practices i Production Merenje

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- Kada menjati GOMAXPROCS
- Kada ne menjati GOMAXPROCS
- GOMAXPROCS tuning strategija
- Benchmark u realnim uslovima
- Scheduler metrics
- CPU utilization analiza
- Latency analiza
- Production best practices

---

# Uvod

Do sada smo naučili:

```
GOMAXPROCS

↓

broj P objekata

↓

broj paralelnih execution slotova
```

---

Takođe smo videli:

- lokalni razvoj,
- CPU workload,
- container okruženja.

---

Sada dolazimo do najvažnijeg production pitanja:

> Da li treba ručno menjati GOMAXPROCS?

---

# Pravilo broj 1

U većini aplikacija:

```
NE menjati GOMAXPROCS
```

---

Zašto?

Zato što Go runtime već radi dobar posao:

- detektuje CPU resurse,
- kreira odgovarajući broj P objekata,
- balansira Goroutines.

---

Default ponašanje je često:

```
najbolja početna vrednost
```

---

# Kada ima smisla menjati GOMAXPROCS?

Postoje određeni slučajevi.

---

## 1. CPU ograničeni workload

Primer:

- batch processing,
- data processing,
- image processing,
- scientific computing.

---

Imamo:

```
100% CPU workload
```

---

Možemo testirati:

```
GOMAXPROCS = 1

GOMAXPROCS = 2

GOMAXPROCS = 4

GOMAXPROCS = 8
```

---

Cilj:

pronaći:

```
najveći throughput
```

---

# 2. Container CPU quota

Primer:

Container:

```
CPU limit = 2
```

---

Host:

```
32 cores
```

---

Ako runtime pogrešno vidi:

```
32 CPU
```

---

Može imati smisla:

```
GOMAXPROCS = 2
```

---

---

# 3. Specijalizovani workload

Primer:

Aplikacija:

```
jedan veoma skup CPU task
```

---

Ponekad je bolje:

```
manje P

više predvidljivosti
```

---

---

# Kada NE menjati GOMAXPROCS?

---

## IO-heavy serveri

Primer:

HTTP API:

```
Database

↓

Network

↓

Response
```

---

Većinu vremena:

```
Goroutines čekaju
```

---

Promena GOMAXPROCS često neće pomoći.

---

---

## Aplikacije sa dobrim default ponašanjem

Primer:

standardni Go servis:

- REST API,
- gRPC servis,
- worker servis.

---

Bolje:

```
ostaviti runtime-u
```

---

# GOMAXPROCS tuning proces

Ne:

```
promeni vrednost

↓

nadati se boljem rezultatu
```

---

Ispravan proces:

```
1. Izmeri trenutno stanje

2. Napravi benchmark

3. Promeni jednu stvar

4. Ponovo meri

5. Uporedi rezultate
```

---

# Merenje performansi

Važni parametri:

---

## Throughput

Pitanje:

```
Koliko posla završimo?
```

---

Primer:

```
requests/sec
```

---

---

## Latency

Pitanje:

```
Koliko brzo odgovorimo?
```

---

Primer:

```
p95 = 50ms
```

---

---

## CPU usage

Pitanje:

```
Da li koristimo CPU efikasno?
```

---

---

## Scheduler overhead

Pitanje:

```
Koliko vremena runtime troši na scheduling?
```

---

# Benchmark primer

CPU workload:

```go
func BenchmarkWork(
	b *testing.B,
){

	for i := 0; i < b.N; i++ {

		calculate()

	}

}
```

---

Pokretanje:

```bash
GOMAXPROCS=1 go test -bench .
```

---

Zatim:

```bash
GOMAXPROCS=2 go test -bench .
```

---

I:

```bash
GOMAXPROCS=4 go test -bench .
```

---

Poređenje:

```
vreme

↓

throughput

↓

CPU usage
```

---

# Profiling

Benchmark nije dovoljan.

---

Koristiti:

```bash
go test -cpuprofile cpu.out
```

---

Analiza:

```bash
go tool pprof cpu.out
```

---

Tražiti:

- hot functions,
- lock contention,
- scheduler overhead.

---

# Runtime trace

Za scheduler probleme:

```bash
go test -trace trace.out
```

---

Pokretanje:

```bash
go tool trace trace.out
```

---

Možemo videti:

- Goroutine scheduling,
- blocking,
- network wait,
- GC događaje.

---

# Runtime metrics

Paket:

```go
runtime/metrics
```

---

Može pratiti:

- scheduler stanje,
- GC,
- memory,
- goroutine count.

---

Primer:

```go
metrics.All()
```

---

---

# GOMAXPROCS i lock contention

Važan slučaj.

Imamo:

```go
var mu sync.Mutex
```

---

Mnogo Goroutines:

```
G1
G2
G3
G4
```

---

Sve čekaju:

```
isti lock
```

---

Povećanje:

```
GOMAXPROCS 4 → 16
```

---

Može pogoršati:

```
lock contention
```

---

Jer:

više izvršilaca pokušava isti resurs.

---

# False assumption

Pogrešno:

```
više P

=

uvek brži program
```

---

Tačno:

```
više P

=

više mogućnosti za paralelizam
```

---

Ali rezultat zavisi od:

- algoritma,
- memorije,
- lockova,
- IO-a.

---

# Production preporučeni pristup

Za tipičan servis:

```
1. Koristi default

2. Postavi ispravne CPU limite

3. Prati metrike

4. Benchmarkiraj probleme

5. Menjaj samo ako merenje pokaže korist
```

---

# Primer production odluke

Situacija:

```
API servis

CPU limit: 4

GOMAXPROCS: 64
```

---

Problem:

- visoka latency,
- CPU throttling.

---

Rešenje:

```
GOMAXPROCS ≈ 4
```

---

Rezultat:

- manje scheduling overhead-a,
- stabilniji latency.

---

# Senior mentalni model

Kada vidiš:

```go
runtime.GOMAXPROCS(x)
```

pitaj:

```
Zašto menjamo?

Koji problem rešavamo?

Koje metrike potvrđuju promenu?
```

---

Nikada:

```
stavi najveću vrednost
```

---

---

# 📋 Rezime

U ovom delu naučili smo:

- kada menjati GOMAXPROCS,
- kada ostaviti default,
- kako raditi tuning proces,
- koje metrike pratiti,
- kako benchmarkirati promene,
- odnos GOMAXPROCS-a i lock contention-a,
- production best practices.

---

# GOMAXPROCS — Praktični Eksperimenti, Benchmark i Zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 7/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/07-gomaxprocs.md`

---

# 📚 Sadržaj ovog dela

- Eksperiment 1 — Posmatranje GOMAXPROCS
- Eksperiment 2 — CPU-bound paralelizam
- Eksperiment 3 — Poređenje vrednosti
- Eksperiment 4 — Worker pool tuning
- Eksperiment 5 — Scheduler analiza
- Praktični zadaci
- Završni rezime GOMAXPROCS lekcije

---

# Uvod

U prethodnim delovima smo prošli:

```
GOMAXPROCS

↓

P objekti

↓

Scheduler

↓

CPU paralelizam
```

---

Sada ćemo znanje primeniti kroz eksperimente.

---

# Eksperiment #1 — Čitanje trenutne vrednosti

Cilj:

razumeti razliku između:

- CPU broja,
- GOMAXPROCS vrednosti.

---

Kod:

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {

	fmt.Println(
		"CPU:",
		runtime.NumCPU(),
	)

	fmt.Println(
		"GOMAXPROCS:",
		runtime.GOMAXPROCS(0),
	)

}
```

---

Primer rezultat:

```
CPU: 8

GOMAXPROCS: 8
```

---

Objašnjenje:

```
NumCPU

=

koliko CPU runtime vidi


GOMAXPROCS

=

koliko P koristi
```

---

# Eksperiment #2 — CPU-bound paralelizam

Cilj:

videti uticaj GOMAXPROCS.

---

Kod:

```go
package main

import (
	"runtime"
	"sync"
)

func calculate() {

	var x uint64

	for i := 0; i < 1_000_000_000; i++ {

		x += uint64(i)

	}

}

func main() {

	runtime.GOMAXPROCS(4)

	var wg sync.WaitGroup

	for i := 0; i < 4; i++ {

		wg.Add(1)

		go func(){

			defer wg.Done()

			calculate()

		}()

	}

	wg.Wait()

}
```

---

Eksperiment:

Menjati:

```go
runtime.GOMAXPROCS(1)
```

zatim:

```go
runtime.GOMAXPROCS(2)
```

zatim:

```go
runtime.GOMAXPROCS(4)
```

---

Meriti:

- ukupno vreme,
- CPU usage.

---

# Očekivanje

Na mašini sa:

```
4 cores
```

---

Rezultat približno:

```
GOMAXPROCS=1

najsporije


GOMAXPROCS=4

najbrže
```

---

Ali:

nije garantovano.

---

Zavisi od:

- CPU-a,
- memorije,
- compiler optimizacija.

---

# Eksperiment #3 — Previše P objekata

Cilj:

videti overhead.

---

Test:

```go
runtime.GOMAXPROCS(100)
```

---

Ako imamo:

```
4 CPU cores
```

---

Pitanje:

Da li je:

```
100x brže?
```

---

Odgovor:

```
ne
```

---

Zašto?

Jer:

```
CPU ostaje 4
```

---

Ali scheduler ima:

```
više koordinacije
```

---

# Eksperiment #4 — Worker Pool tuning

Primer:

Imamo:

```
10000 poslova
```

---

Worker pool:

```go
workers := runtime.GOMAXPROCS(0)
```

---

Primer:

```go
for i := 0; i < workers; i++ {

	go worker()

}
```

---

Za CPU-bound workload:

dobar početak:

```
workers ≈ GOMAXPROCS
```

---

---

Za IO-bound:

Primer:

HTTP crawler:

```
workers može biti mnogo veći
```

---

Jer:

većinu vremena:

```
čekaju
```

---

# Eksperiment #5 — Scheduler posmatranje

Koristiti:

```bash
GODEBUG=schedtrace=1000
```

---

Primer:

```bash
GODEBUG=schedtrace=1000 ./app
```

---

Dobijamo informacije:

- broj P,
- idle P,
- running M,
- Goroutine stanje.

---

Primer konceptualnog izlaza:

```
gomaxprocs=4

idleprocs=1

threads=5

goroutines=100
```

---

# Eksperiment #6 — GOMAXPROCS i Mutex

Cilj:

videti contention.

---

Kod:

```go
var mu sync.Mutex

var counter int

func increment(){

	mu.Lock()

	counter++

	mu.Unlock()

}
```

---

Pokrenuti:

```
100000 Goroutines
```

---

Test:

```
GOMAXPROCS=1

GOMAXPROCS=8
```

---

Mogući rezultat:

veći GOMAXPROCS:

```
više čekanja na lock
```

---

Pouka:

```
paralelizam zahteva dobar dizajn
```

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnove

Napraviti program koji:

- ispisuje CPU broj,
- ispisuje GOMAXPROCS,
- menja GOMAXPROCS vrednost.

Cilj:

razumeti osnovni API.

---

# 🟡 Nivo 2 — CPU benchmark

Napraviti:

```
CPU intensive task
```

---

Testirati:

```
GOMAXPROCS = 1

GOMAXPROCS = 2

GOMAXPROCS = 4

GOMAXPROCS = 8
```

---

Zabeležiti:

- vreme,
- CPU usage.

---

# 🟠 Nivo 3 — Worker Pool

Napraviti:

```
Job queue

+

Worker pool
```

---

Eksperimentisati:

```
workers = 1

workers = GOMAXPROCS

workers = 2*GOMAXPROCS
```

---

Uporediti rezultate.

---

# 🔴 Nivo 4 — Production simulacija

Napraviti servis:

```
HTTP API

+

Worker pool

+

CPU task

+

Database simulation
```

---

Meriti:

- request latency,
- throughput,
- CPU usage.

---

Testirati:

različite:

```
GOMAXPROCS
```

---

# 🔥 Senior zadatak

Napraviti benchmark sistem:

Meri:

```
GOMAXPROCS

vs

Throughput

vs

Latency

vs

CPU utilization
```

---

Napraviti tabelu:

| GOMAXPROCS | Throughput | p95 latency | CPU |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 4 | | | |
| 8 | | | |

---

Zaključiti:

```
optimalna vrednost
```

---

# Najvažnije lekcije

---

## 1.

GOMAXPROCS nije:

```
broj Goroutines
```

---

## 2.

GOMAXPROCS nije:

```
broj Threads
```

---

## 3.

GOMAXPROCS kontroliše:

```
broj aktivnih P
```

---

## 4.

Više P nije uvek bolje.

---

## 5.

Benchmark je jedini pravi odgovor.

---

# Finalni mentalni model

Cela slika:

```
Goroutines

      ↓

Scheduler

      ↓

P (GOMAXPROCS)

      ↓

M (OS Threads)

      ↓

CPU cores
```

---

# 📋 Završni rezime lekcije

U kompletnoj lekciji **GOMAXPROCS** naučili smo:

✅ šta je GOMAXPROCS  
✅ kako radi unutar GMP modela  
✅ vezu sa Processor (`P`) komponentom  
✅ CPU paralelizam  
✅ CPU-bound i IO-bound razliku  
✅ container probleme  
✅ production tuning  
✅ benchmark metodologiju  
✅ praktične eksperimente  

---

### ➡️ Sledeća lekcija **[**Parallelism vs Concurrency**](08-parallelism-vs-concurrency.md)**

Obradićemo:

- razliku između ova dva koncepta,
- modele izvršavanja,
- Go pristup concurrency-ju,
- kada koristiti concurrency a kada parallelism,
- praktične primere.
