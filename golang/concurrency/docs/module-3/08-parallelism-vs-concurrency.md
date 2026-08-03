# Parallelism vs Concurrency — Uvod u Koncepte

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Zašto razlikovati concurrency i parallelism
- Definicija concurrency-ja
- Definicija parallelism-a
- Istorijski kontekst
- Mentalni modeli
- Veza sa Go programiranjem
- Prvi primeri

---

# Uvod

U prethodnim lekcijama obrađivali smo:

```
Goroutines

Channels

Scheduler

GOMAXPROCS
```

---

Svi ovi koncepti vode ka jednom važnom pitanju:

> Da li se više stvari izvršava istovremeno ili samo upravljamo njihovim tokom?

---

Odgovor zahteva razlikovanje:

```
Concurrency

vs

Parallelism
```

---

# Zašto je ova razlika važna?

U svakodnevnom govoru često kažemo:

> "Program radi više stvari odjednom."

---

Ali to može značiti dve potpuno različite stvari.

---

Primer:

Aplikacija:

```
Prima HTTP request

Čita iz baze

Piše log

Šalje email
```

---

Može raditi:

```
concurrently
```

---

ali možda nema:

```
parallel execution
```

---

# Concurrency — definicija

Concurrency znači:

> Dizajniranje programa tako da više poslova može biti u toku u isto vreme.

---

Ključna reč:

```
u toku
```

---

Ne znači obavezno:

```
izvršavaju se u istom trenutku
```

---

# Primer concurrency-ja

Imamo jednu osobu:

```
Programer
```

---

Zadatke:

```
Task A

Task B

Task C
```

---

Radi:

```
A malo

B malo

C malo

A malo

B malo
```

---

Rezultat:

Svi poslovi napreduju.

---

Ali:

```
jedna osoba
```

---

Nema pravog paralelizma.

---

# Računarski primer

Imamo:

```
CPU = 1 core
```

---

I:

```
Goroutine A

Goroutine B

Goroutine C
```

---

Scheduler radi:

```
A

↓

B

↓

C

↓

A
```

---

Program je:

```
concurrent
```

---

Ali nije:

```
parallel
```

---

# Parallelism — definicija

Parallelism znači:

> Izvršavanje više zadataka u isto vreme koristeći više izvršnih jedinica.

---

Ključna reč:

```
istovremeno
```

---

Primer:

Imamo:

```
4 CPU cores
```

---

Izvršavanje:

```
Core 1 → Task A

Core 2 → Task B

Core 3 → Task C

Core 4 → Task D
```

---

To je:

```
parallel execution
```

---

# Concurrency bez parallelism-a

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

Go može upravljati:

```
1000 poslova
```

---

Ali:

samo jedan:

```
execution slot
```

---

Model:

```
G1

↓

G2

↓

G3
```

---

To je:

```
Concurrency
```

---

# Concurrency sa parallelism-om

Primer:

```
GOMAXPROCS = 4
```

---

Imamo:

```
G1

G2

G3

G4
```

---

Moguće:

```
G1 → CPU1

G2 → CPU2

G3 → CPU3

G4 → CPU4
```

---

Dobijamo:

```
Concurrency

+

Parallelism
```

---

# Analogija sa restoranom

## Concurrency

Jedan kuvar.

Ima:

```
porudžbina A

porudžbina B

porudžbina C
```

---

Kuvar:

```
isecka povrće

stavi meso

pripremi salatu

vrati se na meso
```

---

Sve porudžbine napreduju.

---

Ali:

```
jedan kuvar
```

---

---

## Parallelism

Tri kuvara:

```
Kuvar 1 → A

Kuvar 2 → B

Kuvar 3 → C
```

---

Tri stvari se rade:

```
istovremeno
```

---

# Veza sa Go-om

Go je dizajniran prvenstveno za:

```
Concurrency
```

---

Glavni alati:

```go
goroutines

channels

select
```

---

Primer:

```go
go fetchUser()

go fetchOrders()

go fetchPayments()
```

---

Program kaže:

```
ovi poslovi mogu napredovati nezavisno
```

---

---

# Da li ovo garantuje parallelism?

Ne.

---

Zavisi od:

```
GOMAXPROCS
```

---

Ako:

```
GOMAXPROCS = 1
```

---

Scheduler radi:

```
switch između Goroutines
```

---

Ako:

```
GOMAXPROCS > 1
```

---

Može koristiti:

```
više CPU core-ova
```

---

# Primer

Kod:

```go
go calculateA()

go calculateB()
```

---

Moguće izvršavanje:

---

## Scenario 1

```
CPU = 1
```

Rezultat:

```
A

B
```

---

Concurrency.

---

## Scenario 2

```
CPU = 2
```

Rezultat:

```
Core1 → A

Core2 → B
```

---

Concurrency + Parallelism.

---

# Važna razlika

Concurrency je:

```
struktura programa
```

---

Parallelism je:

```
način izvršavanja
```

---

Jednostavno:

```
Concurrency

=

kako organizujemo posao


Parallelism

=

kako hardver izvršava posao
```

---

# Veza sa Scheduler-om

Prethodne lekcije:

```
Goroutine

↓

Scheduler

↓

P

↓

M

↓

CPU
```

---

Scheduler omogućava:

```
concurrency
```

---

GOMAXPROCS omogućava:

```
parallel execution capacity
```

---

# Česte zablude

---

## Zabluda 1

"Concurrency znači paralelno."

Netačno.

---

Concurrency može postojati bez:

```
više CPU jezgara
```

---

---

## Zabluda 2

"Ako koristim goroutines, program je parallel."

Netačno.

---

Goroutines daju:

```
concurrency
```

---

---

## Zabluda 3

"Parallelism rešava sve probleme."

Netačno.

---

Više paralelizma može doneti:

- race condition,
- lock contention,
- kompleksnost.

---

# Mentalni model

Zapamti:

```
Concurrency

=
više stvari se obrađuje


Parallelism

=
više stvari se izvršava u istom trenutku
```

---

Još kraće:

```
Concurrency = structure

Parallelism = execution
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto razlikujemo concurrency i parallelism,
- šta znači concurrency,
- šta znači parallelism,
- kako concurrency radi bez parallelism-a,
- kako Go koristi concurrency,
- kako GOMAXPROCS utiče na parallel execution.

---

# Parallelism vs Concurrency — Modeli Izvršavanja

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Sequential execution model
- Concurrent execution model
- Parallel execution model
- Concurrent + Parallel model
- Vizuelna poređenja
- Veza sa Go Scheduler-om
- Primeri iz realnih sistema

---

# Uvod

U prethodnom delu definisali smo:

```
Concurrency

=

upravljanje sa više poslova


Parallelism

=

istovremeno izvršavanje više poslova
```

---

Sada ćemo pogledati kako ovi modeli izgledaju u praksi.

---

# 1. Sequential Execution

Sequential znači:

```
jedan posao

pa sledeći posao

pa sledeći
```

---

Model:

```
Time →

Task A █████

Task B       █████

Task C             █████
```

---

Svaki zadatak čeka prethodni.

---

Primer:

```go
func main() {

	taskA()

	taskB()

	taskC()

}
```

---

Izvršavanje:

```
A

↓

B

↓

C
```

---

Karakteristike:

✅ jednostavno

✅ predvidljivo

❌ sporo za nezavisne poslove

---

# Primer

Imamo:

```go
func loadUser(){

}

func loadOrders(){

}

func loadPayments(){

}
```

---

Sequential:

```go
loadUser()

loadOrders()

loadPayments()
```

---

Vreme:

```
100ms

+

200ms

+

150ms

=

450ms
```

---

---

# 2. Concurrent Execution

Concurrent znači:

više poslova je:

```
aktivno u isto vreme
```

---

Ali ne mora biti:

```
izvršeno u istom trenutku
```

---

Model:

```
Time →

Task A ████      ███

Task B     ████      ███

Task C        ████
```

---

Poslovi se prepliću.

---

Primer u Go-u:

```go
go loadUser()

go loadOrders()

go loadPayments()
```

---

Scheduler upravlja:

```
Goroutine A

Goroutine B

Goroutine C
```

---

# Concurrency na jednom CPU core-u

Imamo:

```
CPU = 1
```

---

Izvršavanje:

```
A

B

C

A

B

C
```

---

Izgleda kao:

```
istovremeno
```

---

Ali CPU radi:

```
jednu stvar u trenutku
```

---

Ovo je:

```
Concurrency bez Parallelism-a
```

---

# 3. Parallel Execution

Parallel znači:

više poslova se stvarno izvršava:

```
u istom trenutku
```

---

Potrebno:

```
više execution jedinica
```

---

Najčešće:

```
više CPU core-ova
```

---

Model:

```
Time →

Core 1: Task A █████

Core 2: Task B █████

Core 3: Task C █████
```

---

---

# Primer

Imamo:

```
CPU = 4 cores
```

---

Program:

```go
calculateA()

calculateB()

calculateC()

calculateD()
```

---

Ako ih rasporedimo:

```
Core1 → A

Core2 → B

Core3 → C

Core4 → D
```

---

Svi rade:

```
u istom trenutku
```

---

# 4. Concurrent + Parallel Execution

Ovo je najčešći model modernih sistema.

---

Imamo:

```
Concurrency

+

Parallelism
```

---

Primer:

```
10000 Goroutines

+

8 CPU cores
```

---

Go scheduler radi:

```
Goroutines

↓

P

↓

M

↓

CPU
```

---

Rezultat:

neke Goroutines:

```
stvarno rade paralelno
```

dok druge:

```
čekaju
```

---

# Vizuelno poređenje

## Sequential

```
CPU:

AAAA BBBB CCCC
```

---

## Concurrent

```
CPU:

AA BB CC AA BB CC
```

---

## Parallel

```
CPU1:

AAAA


CPU2:

BBBB


CPU3:

CCCC
```

---

## Concurrent + Parallel

```
CPU1:

A B A C


CPU2:

B C A B


CPU3:

C A B C
```

---

# Go Scheduler perspektiva

Veza sa prethodnim lekcijama:

```
Goroutines
     |
     v
Scheduler
     |
     v
P
     |
     v
M
     |
     v
CPU
```

---

Scheduler rešava:

```
koja Goroutine ide sledeća
```

---

GOMAXPROCS određuje:

```
koliko P može biti aktivno
```

---

# Primer: HTTP Server

Zamislimo:

```
1000 korisnika
```

---

Svaki request:

```
čitaju bazu

obrađuju podatke

vraćaju odgovor
```

---

Go:

```
request 1 → goroutine

request 2 → goroutine

request 3 → goroutine
```

---

To je:

```
Concurrency
```

---

Ako server ima:

```
8 CPU cores
```

---

I:

```
GOMAXPROCS = 8
```

---

Može dobiti:

```
Concurrency

+

Parallelism
```

---

# Primer: Database Queries

Imamo:

```
User service
```

---

Potrebno:

```
load profile

load orders

load settings
```

---

Sequential:

```
profile

↓

orders

↓

settings
```

---

Concurrent:

```
profile ─┐

orders ──┼── wait

settings ─┘
```

---

Dobijamo:

manju ukupnu latenciju.

---

# Primer: CPU Compression

Imamo:

```
100 slika
```

---

Compression:

```
CPU-heavy
```

---

Sequential:

```
slika1

slika2

slika3
```

---

Parallel:

```
Core1 → slika1

Core2 → slika2

Core3 → slika3
```

---

Dobijamo:

```
veći throughput
```

---

# Kako prepoznati koji model ti treba?

Postavi pitanje:

---

## Da li poslovi čekaju?

Primer:

- network
- database
- file IO

---

Koristi:

```
Concurrency
```

---

---

## Da li poslovi troše CPU?

Primer:

- encoding
- hashing
- matematika

---

Koristi:

```
Parallelism
```

---

---

# Važna veza

Concurrency rešava:

```
organizaciju rada
```

---

Parallelism rešava:

```
brzinu izvršavanja
```

---

Ne zamenjuju jedan drugog.

---

# Mentalni model

```
Sequential

jedan posao


Concurrent

više poslova u toku


Parallel

više poslova istovremeno


Concurrent + Parallel

više poslova + više CPU resursa
```

---

# 📋 Rezime

U ovom delu naučili smo:

- razliku između sequential, concurrent i parallel execution,
- kako izgleda izvršavanje na jednom i više CPU jezgara,
- kako Go Scheduler omogućava concurrency,
- kako GOMAXPROCS utiče na parallel execution,
- praktične primere iz servera i CPU obrade.

---

# Parallelism vs Concurrency — Go Concurrency Model

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Go pristup concurrency-ju
- Goroutines kao concurrency primitive
- Channels kao komunikacioni mehanizam
- CSP model
- Shared memory vs Message Passing
- Kako Go kombinuje concurrency i parallelism
- Primeri dizajna

---

# Uvod

Do sada smo videli:

```
Concurrency

=

više poslova koji napreduju nezavisno
```

---

Go je napravljen oko ideje:

> "Ne komuniciraj deljenjem memorije; deli memoriju komunikacijom."

Ovo je jedna od najvažnijih filozofija Go concurrency modela.

---

# Go Concurrency model

Go koristi nekoliko osnovnih mehanizama:

```
Goroutines

+

Channels

+

Select

+

Scheduler
```

---

Oni zajedno omogućavaju:

```
lakše pisanje concurrent programa
```

---

# Goroutines

Goroutine je:

> Laka izvršna jedinica kojom upravlja Go runtime.

---

Primer:

```go
func process() {

	fmt.Println("processing")

}

func main() {

	go process()

}
```

---

Ključna reč:

```go
go
```

---

Kreira:

```
nova Goroutine
```

---

# Goroutine nije Thread

Česta greška:

```
1 Goroutine = 1 OS Thread
```

---

Netačno.

---

Odnos:

```
10000 Goroutines


        ↓


nekoliko OS Threads
```

---

Go scheduler mapira:

```
Goroutines

↓

OS Threads
```

---

Preko GMP modela:

```
G

↓

P

↓

M
```

---

# Goroutines daju concurrency

Primer:

```go
go downloadFile()

go processImage()

go saveResult()
```

---

Program kaže:

```
ovi poslovi mogu raditi nezavisno
```

---

To je:

```
Concurrency
```

---

Ali:

da li postoji parallelism?

---

Zavisi od:

```
GOMAXPROCS
```

---

# Primer bez parallelism-a

Imamo:

```
GOMAXPROCS = 1
```

---

Kod:

```go
go taskA()

go taskB()
```

---

Scheduler:

```
taskA

↓

taskB

↓

taskA
```

---

Rezultat:

```
Concurrency

bez

Parallelism-a
```

---

# Primer sa parallelism-om

Imamo:

```
GOMAXPROCS = 2
```

---

Kod:

```go
go taskA()

go taskB()
```

---

Moguće:

```
CPU 1 → taskA

CPU 2 → taskB
```

---

Rezultat:

```
Concurrency

+

Parallelism
```

---

# Channels

Goroutines same po sebi nisu dovoljne.

---

Pitanje:

Kako komuniciraju?

---

Go koristi:

```go
channel
```

---

Primer:

```go
messages := make(chan string)
```

---

Slanje:

```go
messages <- "hello"
```

---

Primanje:

```go
msg := <-messages
```

---

---

# Channel kao komunikacioni kanal

Model:

```
Goroutine A

      |

      v

 Channel

      |

      v

Goroutine B
```

---

Podaci se prenose:

```
između Goroutines
```

---

# CSP model

Go concurrency je inspirisan konceptom:

```
Communicating Sequential Processes
(CSP)
```

---

Ideja:

> Ne deliš memoriju između procesa; oni komuniciraju porukama.

---

Model:

```
Process A

   |
 message

   |

Process B
```

---

---

# Shared Memory pristup

Tradicionalni model:

```
Thread A

       \
        \
       Shared Memory
        /
       /

Thread B
```

---

Problem:

više izvršavalaca pristupa istim podacima.

---

Potrebni:

- mutex,
- locks,
- atomic operations.

---

Primer:

```go
counter++

```

---

Rizik:

```
race condition
```

---

# Message Passing pristup

Go preferira:

```
message passing
```

---

Primer:

```go
jobs <- job
```

---

Umesto:

```
svi pristupaju istoj memoriji
```

---

Koristi se:

```
kanal komunikacije
```

---

# Poređenje modela

## Shared Memory

```
Thread A

   |
   |
 Shared Variable
   |
   |
Thread B
```

---

Potrebno:

```
lock
```

---

## Message Passing

```
Goroutine A

   |
 channel

   |

Goroutine B
```

---

Potrebna:

```
sinhronizovana komunikacija
```

---

# Primer Worker Pattern-a

Imamo:

```
Producer

↓

Channel

↓

Workers
```

---

Kod:

```go
jobs := make(chan int)

go worker(jobs)

jobs <- 10
```

---

Tok:

```
Main

↓

job

↓

channel

↓

worker
```

---

Ovo je tipičan Go concurrency pattern.

---

# Select statement

Kada imamo više kanala:

```go
select {

case msg := <-ch1:

case msg := <-ch2:

}
```

---

Go bira:

```
koji događaj je spreman
```

---

Koristi se za:

- timeout,
- cancellation,
- multiple events.

---

# Kako Go kombinuje concurrency i parallelism?

Cela slika:

```
Goroutines

      ↓

Go Scheduler

      ↓

GOMAXPROCS

      ↓

P

      ↓

M

      ↓

CPU cores
```

---

Goroutines daju:

```
Concurrency
```

---

Scheduler + CPU daju:

```
Parallelism
```

---

# Primer HTTP servera

Request:

```
GET /users
```

---

Go kreira:

```
goroutine
```

---

Ta goroutine:

- čita bazu,
- šalje odgovor.

---

10000 korisnika:

```
10000 Goroutines
```

---

CPU:

```
8 cores
```

---

Dobijamo:

```
Concurrency

+

Parallelism
```

---

# Go filozofija

Poznata Go ideja:

```
Do not communicate by sharing memory;
share memory by communicating.
```

---

Značenje:

Bolje:

```
channel

↓

poruka

↓

druga Goroutine
```

---

nego:

```
globalna promenljiva

+

mutex
```

---

Ali:

nije apsolutno pravilo.

---

# Kada koristiti Mutex?

Shared memory može biti pravi izbor.

Primer:

```go
cache map[string]string
```

---

Mutex:

```go
mu.Lock()

cache[key] = value

mu.Unlock()
```

---

Za male kritične sekcije:

mutex je često jednostavniji.

---

# Senior perspektiva

Go concurrency nije:

```
samo pravljenje Goroutines
```

---

Pravo pitanje je:

```
Kako organizovati tok podataka?

Ko poseduje stanje?

Kako komponente komuniciraju?
```

---

# Mentalni model

Zapamti:

```
Goroutines

=

konkurentni izvršioci


Channels

=

komunikacija


Scheduler

=

upravljanje izvršavanjem


GOMAXPROCS

=

potencijalni parallelism
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako Go implementira concurrency model,
- ulogu Goroutines,
- razliku između Goroutine i Thread-a,
- ulogu Channels,
- CSP princip,
- Shared Memory vs Message Passing,
- kako Go kombinuje concurrency i parallelism.

---

# Parallelism vs Concurrency — Kada Koristiti Concurrency, a Kada Parallelism

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Kako izabrati pravi pristup
- IO-bound workload
- CPU-bound workload
- Latency optimizacija
- Throughput optimizacija
- Concurrency patterns
- Parallel processing patterns
- Arhitektonske odluke

---

# Uvod

Do sada smo naučili:

```
Concurrency

=

upravljanje većim brojem nezavisnih poslova
```

i:

```
Parallelism

=

korišćenje više CPU resursa istovremeno
```

---

Sada dolazi praktično pitanje:

> Kada koristiti koji pristup?

---

Odgovor zavisi prvenstveno od:

- tipa workload-a,
- cilja optimizacije,
- dostupnih resursa.

---

# Prvo pitanje: Šta je usko grlo?

Pre nego što dodamo:

```go
go
```

pitamo:

```
Šta čekamo?
```

---

Ako čekamo:

```
network

database

disk

external API
```

---

problem je:

```
IO latency
```

---

Ako radimo:

```
računanje

transformaciju podataka

algoritam
```

---

problem je:

```
CPU capacity
```

---

# IO-bound workload

IO-bound znači:

program provodi mnogo vremena čekajući.

---

Primeri:

- HTTP pozivi,
- database query,
- čitanje fajlova,
- message queue,
- RPC pozivi.

---

Tok:

```
Request

↓

čekanje

↓

obrada

↓

čekanje
```

---

CPU često nije maksimalno iskorišćen.

---

# Rešenje: Concurrency

Kod:

```go
go fetchUser()

go fetchOrders()

go fetchPayments()
```

---

Dok jedna Goroutine čeka:

```
database response
```

druga može raditi:

```
drugi posao
```

---

Dobijamo:

```
bolje iskorišćenje vremena
```

---

# Primer: API agregator

Imamo servis:

```
/dashboard
```

Treba:

```
User Service

Order Service

Payment Service
```

---

## Sequential

```
User

↓

Orders

↓

Payments
```

---

Vreme:

```
100ms

+

200ms

+

150ms

=

450ms
```

---

## Concurrent

```
User
 \
  \
Orders -----> wait
  /
 /
Payments
```

---

Vreme:

```
max(100,200,150)

=

200ms
```

---

Dobit:

```
manja latency
```

---

# CPU-bound workload

CPU-bound znači:

program troši vreme na računanje.

---

Primeri:

- image processing,
- compression,
- encryption,
- video encoding,
- machine learning,
- matematika.

---

Primer:

```go
func calculate(){

	for i:=0;i<1000000000;i++{

	}

}
```

---

Ovde nema:

```
čekanja
```

---

CPU radi konstantno.

---

# Rešenje: Parallelism

Ako imamo:

```
8 CPU cores
```

možemo:

```
8 zadataka
```

istovremeno.

---

Primer:

```
Image 1 → CPU1

Image 2 → CPU2

Image 3 → CPU3

Image 4 → CPU4
```

---

Dobijamo:

```
veći throughput
```

---

# Primer: Obrada slika

Imamo:

```
10000 slika
```

---

Sekvencijalno:

```
image1

image2

image3
...
```

---

Parallel:

```
Worker 1 → image1

Worker 2 → image2

Worker 3 → image3
```

---

Broj worker-a:

često:

```
≈ broj CPU core-ova
```

---

# Concurrency nije zamena za Parallelism

Pogrešan pristup:

```
Imam spor CPU task

dodam 10000 Goroutines
```

---

Rezultat:

```
više scheduler posla
```

---

Ne:

```
više brzine
```

---

Primer:

```go
go calculate()

go calculate()

go calculate()

...
```

---

Ako:

```
GOMAXPROCS = 1
```

---

Sve se i dalje izvršava:

```
jednim CPU slotom
```

---

# Parallelism nije zamena za Concurrency

Suprotan problem:

Imamo:

```
10000 HTTP request-a
```

---

Nećemo:

```
10000 CPU thread-ova
```

---

Jer problem nije CPU.

---

Problem je:

```
čekanje IO-a
```

---

Potrebna je:

```
Concurrency
```

---

# Decision Matrix

| Scenario | Pristup |
|---|---|
| HTTP server | Concurrency |
| Database access | Concurrency |
| File IO | Concurrency |
| CPU rendering | Parallelism |
| Image processing | Parallelism |
| Compression | Parallelism |
| Batch jobs | Zavisi |
| Streaming pipeline | Concurrency + Parallelism |

---

# Kombinovanje oba pristupa

Moderni sistemi često koriste:

```
Concurrency

+

Parallelism
```

---

Primer:

Video processing servis.

---

Imamo:

## Concurrency deo

Primanje fajlova:

```
Upload requests

↓

Goroutines
```

---

## Parallelism deo

Obrada:

```
Workers

↓

CPU cores
```

---

Arhitektura:

```
Client

↓

API Goroutines

↓

Job Queue

↓

Worker Pool

↓

CPU Processing
```

---

# Worker Pool pattern

Čest Go pattern:

```
Jobs

↓

Channel

↓

Workers

↓

Results
```

---

Primer:

```go
jobs := make(chan Job)

results := make(chan Result)
```

---

Workers:

```go
for job := range jobs {

	process(job)

}
```

---

Broj worker-a:

CPU task:

```
≈ GOMAXPROCS
```

---

IO task:

```
može biti veći
```

---

# Concurrency za skaliranje

Concurrency često povećava:

```
capacity
```

---

Primer:

HTTP server:

Bez concurrency:

```
jedan request po vremenu
```

---

Sa concurrency:

```
hiljade aktivnih request-a
```

---

---

# Parallelism za ubrzanje

Parallelism povećava:

```
brzinu jednog velikog posla
```

---

Primer:

Jedna slika:

```
10 sekundi
```

---

Sa 4 CPU-a:

```
3 sekunde
```

---

---

# Performance perspektiva

Concurrency optimizuje:

```
waiting time
```

---

Parallelism optimizuje:

```
compute time
```

---

Formula:

```
IO problem

↓

Concurrency


CPU problem

↓

Parallelism
```

---

# Senior mentalni model

Pre nego što napišeš:

```go
go something()
```

pitaj:

```
Da li ovaj posao čeka?

ili

Da li ovaj posao računa?
```

---

Ako čeka:

```
Concurrency
```

---

Ako računa:

```
Parallelism
```

---

Ako radi oba:

```
kombinuj pristupe
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako birati između concurrency i parallelism,
- razliku IO-bound i CPU-bound problema,
- kada koristiti Goroutines,
- kada koristiti Worker Pool,
- kako kombinovati oba pristupa,
- kako donositi arhitektonske odluke.

---

# Parallelism vs Concurrency — Performance Aspekti i Praktični Dizajn

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Cena concurrency-ja
- Cena parallelism-a
- Scheduler overhead
- Context switching
- Memory i cache efekti
- Race conditions
- Lock contention
- False sharing
- Praktični dizajn primeri

---

# Uvod

Do sada smo naučili:

```
Concurrency

↓

bolje upravljanje velikim brojem poslova


Parallelism

↓

iskorišćenje više CPU resursa
```

---

Ali oba pristupa imaju cenu.

---

Važno pitanje:

> Da li dodatna kompleksnost donosi realnu korist?

---

# Cena concurrency-ja

Concurrency nije besplatan.

---

Svaka Goroutine ima:

- stack memoriju,
- scheduler state,
- runtime metadata.

---

Iako su Goroutines jeftine:

```
1000000 Goroutines
```

nije uvek dobra ideja.

---

# Goroutine overhead

Primer:

```go
for i := 0; i < 1000000; i++ {

	go process()

}
```

---

Problem:

kreiranje velikog broja Goroutines može izazvati:

- memory pressure,
- scheduler overhead,
- GC opterećenje.

---

Bolji pristup:

```
kontrolisan broj radnika
```

---

Primer:

```
Worker Pool
```

---

# Scheduler overhead

Go scheduler mora upravljati:

```
Goroutines

↓

P

↓

M
```

---

Ako imamo:

```
malo Goroutines
```

---

Scheduler radi malo.

---

Ako imamo:

```
milione Goroutines
```

---

Više posla:

- raspoređivanje,
- prebacivanje,
- balansiranje.

---

# Context switching

Kod concurrency-ja može postojati:

```
switch
```

između poslova.

---

Primer:

```
G1

↓

G2

↓

G3
```

---

Svaki prelazak ima cenu.

---

Kod Go-a je mnogo jeftinije nego kod OS thread-ova.

---

Ali:

```
nije nula
```

---

# Cena parallelism-a

Parallelism koristi više CPU resursa.

---

Ali donosi:

- koordinaciju,
- komunikaciju,
- sinhronizaciju.

---

Primer:

Imamo:

```
4 CPU workers
```

---

Ako svi rade nezavisno:

odlično.

---

Ako svi čekaju isti lock:

problem.

---

# Lock Contention

Primer:

```go
var counter int

var mu sync.Mutex
```

---

Više Goroutines:

```
G1
G2
G3
G4
```

---

Sve žele:

```go
counter++
```

---

Rezultat:

```
čekanje na mutex
```

---

Više CPU-a ne pomaže.

---

# Loš dizajn

Primer:

```go
for i:=0;i<100;i++{

	go func(){

		mu.Lock()

		counter++

		mu.Unlock()

	}()

}
```

---

Problem:

mnogo konkurencije:

za veoma mali posao.

---

---

# Bolji dizajn

Umesto:

```
100 Goroutines

+

jedan lock
```

---

Koristiti:

```
local aggregation
```

---

Primer:

Svaka Goroutine:

```
svoj counter
```

---

Na kraju:

```
merge rezultata
```

---

Smanjuje:

```
contention
```

---

# Race Condition

Concurrency donosi opasnost:

```
više izvršilaca

+

deljeno stanje
```

---

Primer:

```go
counter++
```

---

Izgleda kao jedna operacija.

---

Ali zapravo:

```
READ

+

ADD

+

WRITE
```

---

Dve Goroutines:

```
G1 READ 0

G2 READ 0

G1 WRITE 1

G2 WRITE 1
```

---

Rezultat:

```
izgubljen update
```

---

# Rešenje

Opcije:

## 1. Mutex

```go
mu.Lock()

counter++

mu.Unlock()
```

---

## 2. Atomic

```go
atomic.AddInt64()
```

---

## 3. Channel ownership

```
jedna Goroutine poseduje stanje
```

---

# Memory Effects

Parallelism utiče na memoriju.

---

CPU ima:

```
cache
```

---

Ako više CPU core-ova radi:

```
isti podatak
```

---

dolazi do:

```
cache coherence
```

---

---

# False Sharing

Napredni problem.

---

Primer:

CPU1 koristi:

```
var A
```

CPU2 koristi:

```
var B
```

---

Ali A i B su u istoj:

```
cache line
```

---

CPU-i stalno sinhronizuju cache.

---

Rezultat:

```
sporije
```

---

# Memory Locality

Bolji parallel dizajn:

```
svaki worker

ima svoj deo podataka
```

---

Primer:

```
Worker1 → data1

Worker2 → data2

Worker3 → data3
```

---

Manje:

- lockova,
- cache invalidacija.

---

# Pipeline Dizajn

Čest Go pattern:

```
Stage 1

↓

Stage 2

↓

Stage 3
```

---

Primer:

```
Read files

↓

Parse

↓

Process

↓

Save
```

---

Svaki stage:

može biti:

```
concurrent
```

---

A unutra:

```
parallel workers
```

---

# Primer arhitekture

Veliki sistem:

```
HTTP Requests

        |

        v

Ingress Goroutines

        |

        v

Job Queue

        |

        v

Worker Pool

        |

        v

CPU Processing

        |

        v

Storage
```

---

Ovde koristimo:

```
Concurrency

+

Parallelism
```

---

# Kada concurrency pogoršava performanse?

Primer:

CPU task:

```
calculate()
```

---

Pogrešno:

```go
for i:=0;i<100000;i++{

	go calculate()

}
```

---

Problem:

više vremena odlazi na:

```
scheduler
```

nego na:

```
računanje
```

---

# Kada parallelism pogoršava performanse?

Primer:

```
16 workers

na

2 CPU cores
```

---

Dobijamo:

- scheduling overhead,
- contention,
- cache probleme.

---

---

# Performance pravilo

Ne optimizovati:

```
broj Goroutines
```

---

Optimizovati:

```
arhitekturu toka podataka
```

---

# Senior pitanja

Pre concurrency dizajna pitati:

```
Ko poseduje podatke?

Ko ih menja?

Koliko često?

Da li je komunikacija skuplja od računanja?
```

---

# Mentalni model

Zapamti:

```
Concurrency

+

bez kontrole

=

kompleksnost


Parallelism

+

loš dizajn

=

sporiji program
```

---

Dobro dizajniran sistem:

```
pravi broj Goroutines

+

pravi broj workers

+

minimalna sinhronizacija
```

---

# 📋 Rezime

U ovom delu naučili smo:

- cenu concurrency-ja,
- cenu parallelism-a,
- scheduler overhead,
- context switching,
- race conditions,
- lock contention,
- cache probleme,
- kako dizajnirati efikasan concurrent sistem.

---

# Parallelism vs Concurrency — Praktični Zadaci i Završni Mentalni Model

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 8/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/08-parallelism-vs-concurrency.md`

---

# 📚 Sadržaj ovog dela

- Praktični zadaci
- Analiza scenarija
- Dizajn odluke
- Concurrency checklist
- Parallelism checklist
- Senior mentalni model
- Završni rezime lekcije

---

# Uvod

U prethodnim delovima naučili smo:

```
Concurrency

=

organizacija više poslova


Parallelism

=

izvršavanje više poslova istovremeno
```

---

Sada ćemo znanje primeniti kroz praktične situacije.

---

# Zadatak #1 — HTTP API Aggregator

## Scenario

Imamo endpoint:

```
GET /profile
```

---

Potrebno je dohvatiti:

```
User data

Orders

Payments
```

---

## Loš dizajn

```go
user := getUser()

orders := getOrders()

payments := getPayments()
```

---

Problem:

```
latency = suma svih poziva
```

---

---

## Bolji dizajn

Koristiti concurrency:

```go
go getUser()

go getOrders()

go getPayments()
```

---

Zašto?

Zato što su operacije:

```
nezavisne

+

IO-bound
```

---

Pravi izbor:

```
Concurrency
```

---

# Zadatak #2 — Image Processing

## Scenario

Imamo:

```
10000 slika
```

---

Svaka slika:

- resize,
- filter,
- compression.

---

Karakteristika:

```
CPU-heavy
```

---

Rešenje:

```
Worker Pool
```

---

Primer:

```
Workers ≈ CPU cores
```

---

Pravi izbor:

```
Parallelism
```

---

# Zadatak #3 — Web Crawler

## Scenario

Crawler posećuje:

```
1 000 000 URL-ova
```

---

Većina vremena:

```
čekanje HTTP response-a
```

---

CPU nije problem.

---

Rešenje:

```
Goroutines

+

Channels

+

Rate limiting
```

---

Pravi izbor:

```
Concurrency
```

---

# Zadatak #4 — Video Encoding

## Scenario

Jedan video:

```
4K resolution
```

---

Obrada:

```
CPU intensive
```

---

Moguće particionisanje:

```
frame 1

frame 2

frame 3
```

---

Svaki worker:

```
obrađuje deo
```

---

Pravi izbor:

```
Parallelism
```

---

# Zadatak #5 — Streaming Pipeline

## Scenario

Sistem:

```
Read events

↓

Parse

↓

Validate

↓

Store
```

---

Svaka faza može raditi nezavisno.

---

Dizajn:

```
Stage 1

↓

channel

↓

Stage 2

↓

channel

↓

Stage 3
```

---

Pravi izbor:

```
Concurrency

+

Parallelism
```

---

# Decision Tree

Pre nego što dodaš concurrency:

---

## Pitanje 1

Da li posao čeka?

```
Network?

Database?

Disk?
```

---

DA:

```
Concurrency
```

---

NE:

nastavi.

---

## Pitanje 2

Da li posao troši CPU?

```
Calculation?

Encoding?

Compression?
```

---

DA:

```
Parallelism
```

---

---

## Pitanje 3

Da li ima oba?

Primer:

```
API

+

CPU processing
```

---

Koristi:

```
kombinaciju
```

---

# Concurrency Checklist

Pre implementacije proveriti:

---

## 1. Ko poseduje stanje?

Primer:

```go
map[string]User
```

---

Ko je vlasnik?

---

---

## 2. Da li delimo memoriju?

Ako DA:

razmotriti:

- Mutex,
- RWMutex,
- Atomic,
- Channel ownership.

---

---

## 3. Da li postoji ograničenje?

Primer:

```
10000 requests
```

---

Ne:

```
10000 goroutines bez kontrole
```

---

Koristiti:

- worker pool,
- semaphore,
- rate limiter.

---

---

## 4. Kako završavamo posao?

Pitanja:

- ko čeka?
- ko cancel-uje?
- ko zatvara channel?

---

---

# Parallelism Checklist

---

## 1. Koliko CPU resursa postoji?

Proveriti:

```go
runtime.NumCPU()
```

---

---

## 2. Koliko workers koristiti?

CPU workload:

```
≈ GOMAXPROCS
```

---

---

## 3. Da li postoji contention?

Proveriti:

- mutex,
- atomic,
- shared data.

---

---

## 4. Da li delimo podatke?

Ako možemo:

bolje:

```
partition data
```

nego:

```
shared state
```

---

# Najčešće greške

---

# Greška #1

"Više Goroutines = brži program"

Netačno.

---

Dobijamo možda:

```
više overhead-a
```

---

# Greška #2

"Parallelism rešava IO"

Netačno.

---

IO problem:

```
Concurrency
```

---

# Greška #3

"Mutex je uvek loš"

Netačno.

---

Mutex je odličan kada:

- kritična sekcija mala,
- stanje jednostavno,
- nema velikog contention-a.

---

# Greška #4

"Channel uvek bolji od Mutex-a"

Netačno.

---

Izbor zavisi od:

- dizajna,
- vlasništva podataka,
- kompleksnosti.

---

# Senior mentalni model

Senior Go developer ne pita:

> "Koliko Goroutines da napravim?"

---

Već pita:

```
Koji problem rešavam?

IO čekanje?

CPU ograničenje?

Skaliranje?

Latency?

Throughput?
```

---

Zatim bira alat:

```
IO

↓

Concurrency


CPU

↓

Parallelism


Kompleksan sistem

↓

Concurrency + Parallelism
```

---

# Cela slika modula #3

Do sada smo povezali:

```
Mutex

↓

RWMutex

↓

Once

↓

Timeout

↓

Cancellation

↓

Scheduler

↓

GOMAXPROCS

↓

Concurrency vs Parallelism
```

---

Sve zajedno:

```
Go Runtime

       |

       v

Scheduler

       |

       v

Goroutines

       |

       v

Synchronization

       |

       v

CPU execution
```

---

# 📋 Završni rezime lekcije

U kompletnoj lekciji:

# Parallelism vs Concurrency

naučili smo:

✅ razliku između concurrency i parallelism  
✅ sequential vs concurrent vs parallel execution  
✅ Go concurrency model  
✅ Goroutines i Channels  
✅ CSP filozofiju  
✅ IO-bound i CPU-bound odluke  
✅ Worker Pool dizajn  
✅ Performance implikacije  
✅ Race condition i contention probleme  
✅ Kako dizajnirati concurrent sistem  

---

# 🎯 Ključna rečenica

Zapamti:

```
Concurrency rešava kako organizujemo više poslova.

Parallelism rešava kako ubrzavamo izvršavanje korišćenjem više resursa.
```

---

# Kraj lekcije

Završen:

```
Modul #3.8/9

Parallelism vs Concurrency
```

---

### ➡️ Sledeća lekcija **[**Modul #3 - Sumiranje i Zadaci**](09-module-3-summary-and-exercises.md)**

Obuhvatiće:

- kompletan pregled Modula #3,
- povezivanje svih koncepata,
- praktične projekte,
- zadatke po nivoima:
  - Junior,
  - Medior,
  - Senior.
