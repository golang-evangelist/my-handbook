# Cancellation — Kontrolisani prekid rada u Go Concurrency modelu

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- Šta je cancellation
- Zašto je cancellation važan
- Razlika između timeout-a i cancellation-a
- Problem nekontrolisanih Goroutines
- Goroutine lifecycle
- Manualni stop signal
- Uvod u `context` cancellation
- Mentalni model

---

# Uvod

U prethodnoj lekciji naučili smo:

```text
Timeout
```

---

Timeout rešava problem:

> Koliko dugo smo spremni da čekamo?

---

Ali postoji još jedno pitanje:

> Šta ako više ne želimo da se operacija izvršava?

---

Primeri:

- korisnik je zatvorio browser,
- stigao je novi zahtev,
- servis se gasi,
- rezultat više nije potreban,
- korisnik je kliknuo Cancel.

---

U tim situacijama koristimo:

```text
Cancellation
```

---

# Šta je cancellation?

Cancellation znači:

> Slanje signala da neka operacija treba da prestane sa radom.

---

Važno:

Cancellation nije:

```
force kill
```

---

Nije:

```
prekini thread nasilno
```

---

U Go-u:

cancellation je:

```
kooperativni proces
```

---

To znači:

```
jedna strana šalje signal

druga strana odlučuje kada će završiti
```

---

# Timeout vs Cancellation

Veoma važna razlika.

---

# Timeout

Sistem kaže:

```
vreme je isteklo
```

---

Primer:

```text
HTTP request traje 10s

timeout = 2s

prekini čekanje
```

---

Izvor:

```go
context.DeadlineExceeded
```

---

# Cancellation

Neko eksplicitno kaže:

```
prekini rad
```

---

Primer:

```
User klikne Cancel

↓

cancel()

↓

worker završava
```

---

Izvor:

```go
context.Canceled
```

---

# Poređenje

| | Timeout | Cancellation |
|-|-|-|
| Razlog | vreme | odluka |
| Automatski | da | ne uvek |
| API | WithTimeout | WithCancel |
| Primer | request predugo traje | user odustao |
| Error | DeadlineExceeded | Canceled |

---

# Zašto je cancellation važan?

Go omogućava veoma lako kreiranje Goroutines:

```go
go process()
```

---

Ali problem nastaje:

Kako zaustaviti:

```go
process()
```

?

---

Primer:

```go
func process(){

	for {

		doWork()

	}

}
```

---

Ova Goroutine:

- radi zauvek,
- nema kontrolu,
- ne može elegantno da se ugasi.

---

Rezultat:

```
Goroutine leak
```

---

# Goroutine lifecycle

Dobra Goroutine ima:

```
START

 |

RUNNING

 |

STOP SIGNAL

 |

CLEANUP

 |

EXIT
```

---

Loša:

```
START

 |

RUNNING

 |

???

 |

nikada ne završava
```

---

# Primer problema

Zamislimo:

Server pokrene:

```go
go generateReport()
```

---

Korisnik zatvori stranicu.

---

Report više nije potreban.

---

Ali:

```go
generateReport()
```

nastavlja:

```
CPU radi

memorija zauzeta

database upiti traju
```

---

Bolje:

poslati:

```
cancel signal
```

---

# Manualni stop signal

Pre context-a često se koristio poseban channel.

---

Primer:

```go
stop := make(chan struct{})
```

---

Worker:

```go
func worker(
	stop <-chan struct{},
){

	for {

		select {

		case <-stop:

			return


		default:

			work()

		}

	}

}
```

---

Start:

```go
go worker(stop)
```

---

Stop:

```go
close(stop)
```

---

Ovo radi.

---

Ali ima ograničenja.

---

# Problemi custom stop channel-a

---

## 1. Teže propagiranje

Ako imamo:

```
Handler

 |

Service

 |

Repository
```

---

Moramo svuda prosleđivati:

```go
stop chan
```

---

---

## 2. Timeout nije ugrađen

Moramo sami dodavati:

```go
timer
```

---

---

## 3. Standardne biblioteke koriste context

Primer:

- HTTP
- database/sql
- gRPC
- veliki broj biblioteka

---

Zato Go koristi:

```go
context.Context
```

---

# Uvod u context cancellation

Osnovni API:

```go
context.WithCancel()
```

---

Primer:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)


defer cancel()
```

---

Dobijamo:

```
ctx

+

cancel function
```

---

Kada pozovemo:

```go
cancel()
```

---

context šalje signal:

```go
ctx.Done()
```

---

# Prvi cancellation primer

```go
package main

import (
	"context"
	"fmt"
	"time"
)


func worker(
	ctx context.Context,
){

	for {

		select {

		case <-ctx.Done():

			fmt.Println(
				"worker stopped",
			)

			return


		default:

			fmt.Println(
				"working",
			)

			time.Sleep(
				500*time.Millisecond,
			)

		}

	}

}


func main(){

	ctx, cancel :=
		context.WithCancel(
			context.Background(),
		)


	go worker(ctx)


	time.Sleep(
		2*time.Second,
	)


	cancel()


	time.Sleep(
		500*time.Millisecond,
	)

}
```

---

Tok:

```
main

 |

create context

 |

start worker

 |

worker radi

 |

cancel()

 |

ctx.Done()

 |

worker return
```

---

# Važna osobina

Jedan `cancel()` može obavestiti:

```
više Goroutines
```

---

Primer:

```
          cancel()

             |

    -----------------

    |       |       |

   G1      G2      G3
```

---

Zato je context odličan za:

- worker pool,
- server shutdown,
- request propagation.

---

# Cancellation nije prekid

Ovo je najvažniji koncept.

---

Poziv:

```go
cancel()
```

ne radi:

```
kill worker
```

---

Radi:

```
signal worker-u
```

---

Worker mora imati:

```go
select {

case <-ctx.Done():

	return

}
```

---

Ovo se zove:

```text
cooperative cancellation
```

---

# Mentalni model

Zapamti:

```
Timeout

=
vreme je isteklo


Cancellation

=
neko je odlučio da završi
```

---

A:

```
Context

=
kanal za prenos te odluke
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je cancellation,
- zašto je potreban,
- razliku između timeout-a i cancellation-a,
- problem nekontrolisanih Goroutines,
- lifecycle Goroutine-a,
- manualni stop channel pristup,
- zašto se koristi `context`,
- osnovni `WithCancel` pattern,
- cooperative cancellation koncept.

---

# Cancellation — `context.WithCancel` i osnovni cancellation obrasci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- `context.WithCancel`
- `CancelFunc`
- Kako context propagira cancellation signal
- Parent-child context odnos
- Više Goroutines sa istim context-om
- `ctx.Done()`
- `ctx.Err()`
- Osnovni cancellation pattern-i

---

# Uvod

U prethodnom delu videli smo:

```go
context.WithCancel()
```

koji omogućava:

```
kreiranje cancellation signala
```

---

Osnovna ideja:

```
kreiramo context

↓

prosledimo ga Goroutines

↓

pozovemo cancel()

↓

sve Goroutines dobijaju signal
```

---

# `context.WithCancel`

Osnovni API:

```go
func WithCancel(
	parent Context,
) (
	ctx Context,
	cancel CancelFunc,
)
```

---

Primer:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)
```

---

Dobijamo dva elementa:

---

## 1. Context

```go
ctx
```

Sadrži:

- cancellation signal,
- deadline informacije,
- request metadata.

---

## 2. CancelFunc

```go
cancel
```

Funkcija koja šalje signal:

```text
prekini rad
```

---

# `CancelFunc`

Primer:

```go
cancel()
```

---

Kada se pozove:

```
ctx.Done()
```

postaje spreman za čitanje.

---

Primer:

```go
select {

case <-ctx.Done():

	fmt.Println("cancelled")

}
```

---

Rezultat:

```
cancelled
```

---

# Zašto se koristi `defer cancel()`?

Standardni obrazac:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)

defer cancel()
```

---

Zašto?

Zato što:

- oslobađa resurse,
- prekida child context-e,
- sprečava nepotrebno zadržavanje objekata.

---

Primer:

```go
func process(){

	ctx, cancel :=
		context.WithCancel(
			context.Background(),
		)

	defer cancel()


	doWork(ctx)

}
```

---

Bez obzira kako funkcija završi:

- normalno,
- sa error-om,
- panic recovery,

cancel se poziva.

---

# `ctx.Done()`

Najvažniji deo context cancellation-a.

---

Metod:

```go
ctx.Done()
```

vraća:

```go
<-chan struct{}
```

---

To je read-only channel.

---

Kada nema cancellation-a:

```
channel otvoren
```

---

Kada se pozove:

```go
cancel()
```

---

Dobijamo:

```
channel zatvoren
```

---

# Zašto close channel-a?

Go channel model omogućava:

```
jedan signal

↓

više receiver-a
```

---

Primer:

```
             close(done)

                  |

       --------------------

       |         |        |

      G1        G2       G3
```

---

Sve Goroutines mogu detektovati:

```go
<-ctx.Done()
```

---

# Više Goroutines sa jednim context-om

Primer:

```go
func worker(
	id int,
	ctx context.Context,
){

	for {

		select {

		case <-ctx.Done():

			fmt.Println(
				"worker",
				id,
				"stopped",
			)

			return


		default:

			fmt.Println(
				"worker",
				id,
				"working",
			)

		}

	}

}
```

---

Pokretanje:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)


for i:=1;i<=3;i++{

	go worker(i,ctx)

}
```

---

Imamo:

```
G1

G2

G3
```

---

Svi slušaju:

```go
ctx.Done()
```

---

Stop:

```go
cancel()
```

---

Rezultat:

```
worker 1 stopped

worker 2 stopped

worker 3 stopped
```

---

# Parent-child context odnos

Context-i formiraju hijerarhiju.

---

Primer:

```
Background

    |
    |
 WithCancel

    |
    |
 Child
```

---

Ako parent dobije cancellation:

```
parent cancelled

        |

        v

child cancelled
```

---

Ali:

```
child cancelled

        X

parent ostaje aktivan
```

---

Cancellation ide:

```
odozgo nadole
```

---

Ne ide:

```
odozdo nagore
```

---

# Primer child context-a

```go
parent :=
	context.Background()


ctx, cancel :=
	context.WithCancel(
		parent,
	)

defer cancel()
```

---

Sada:

```go
child := ctx
```

---

Kada:

```go
cancel()
```

---

child dobija:

```go
ctx.Done()
```

---

# `ctx.Err()`

Kada je context završen:

možemo proveriti razlog.

---

Primer:

```go
err := ctx.Err()
```

---

Mogući rezultati:

---

## Cancellation

```go
context.Canceled
```

---

Značenje:

```
neko je pozvao cancel()
```

---

## Timeout

```go
context.DeadlineExceeded
```

---

Značenje:

```
rok je istekao
```

---

Primer:

```go
if ctx.Err() ==
	context.Canceled {

	fmt.Println(
		"manual cancel",
	)

}
```

---

# Osnovni cancellation pattern

Standardni obrazac:

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

Ovaj pattern treba koristiti za:

- background worker-e,
- infinite loop,
- polling,
- monitoring,
- queue consumer-e.

---

# Cancellation sa rezultat channel-om

Primer:

```go
func worker(
	ctx context.Context,
	result chan<- int,
){

	value := calculate()


	select {

	case result <- value:

	case <-ctx.Done():

		return

	}

}
```

---

Zašto?

Jer ako je caller odustao:

```
nema receiver-a
```

---

Worker ne sme ostati blokiran.

---

# Cancellation tokom blokirajuće operacije

Loše:

```go
for {

	data := <-channel

}
```

---

Problem:

Ako nema podataka:

```
nikada neće proveriti cancel
```

---

Bolje:

```go
for {

	select {

	case data := <-channel:

		process(data)


	case <-ctx.Done():

		return

	}

}
```

---

# Česte greške

---

## Greška 1

Kreirati novi context umesto prosleđivanja.

Loše:

```go
func Service(){

	ctx := context.Background()

}
```

---

Gubi se:

- timeout,
- cancellation.

---

---

## Greška 2

Ignorisati:

```go
ctx.Done()
```

---

Context postoji, ali niko ga ne sluša.

---

---

## Greška 3

Ne pozvati:

```go
cancel()
```

---

Može dovesti do:

- zadržavanja resursa,
- child context leak-a.

---

# Mentalni model

Zapamti:

```
WithCancel

=

kreiraj signal za prekid


cancel()

=

pošalji signal


ctx.Done()

=

slušaj signal
```

---

Još kraće:

```
cancel()

↓

Done()

↓

return
```

---

# 📋 Rezime

U ovom delu naučili smo:

- detaljno `context.WithCancel`,
- ulogu `CancelFunc`,
- kako radi `ctx.Done()`,
- kako se cancellation propagira,
- parent-child context hijerarhiju,
- korišćenje jednog context-a za više Goroutines,
- `ctx.Err()`,
- osnovne cancellation pattern-e.

---

# Cancellation — Cooperative Cancellation i dizajn funkcija koje poštuju Context

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- Šta je cooperative cancellation
- Zašto Go nema force-kill Goroutine mehanizam
- Cancellation-aware funkcije
- Pravilan dizajn funkcija sa `context.Context`
- Propagacija context-a kroz slojeve aplikacije
- Cancellation tokom CPU-bound i I/O operacija
- Česte greške
- Best practices

---

# Uvod

U prethodnim delovima naučili smo:

```go
cancel()
```

šalje signal:

```text
prekini rad
```

---

Ali postavlja se važno pitanje:

> Ko zapravo zaustavlja Goroutine?

---

Odgovor:

**Sama Goroutine.**

---

Go runtime ne prekida proizvoljno izvršavanje:

```go
go process()
```

---

Ne postoji:

```go
kill(goroutine)
```

---

Razlog:

Go želi da izbegne:

- ostavljene resurse,
- nekonzistentno stanje,
- prekinute kritične sekcije,
- oštećene podatke.

---

Zato Go koristi:

```text
cooperative cancellation
```

---

# Šta je cooperative cancellation?

Cooperative cancellation znači:

> Jedna komponenta šalje zahtev za prekid, a druga komponenta proverava signal i sama završava rad.

---

Model:

```
Controller

   |
   |
 cancel()

   |
   v

Worker

   |
   |
 ctx.Done()

   |
   v

return
```

---

Nema nasilnog prekida.

---

# Zašto Go nema force kill?

Zamislimo:

```go
go updateDatabase()
```

---

U sredini operacije:

```go
BEGIN TRANSACTION

UPDATE users

???
```

---

Ako runtime prekine Goroutine:

```
database stanje?
mutex?
file handle?
memory?
```

---

Ne postoji bezbedan trenutak za prekid.

---

Zato programer odlučuje:

```go
if cancelled {

	cleanup()

	return

}
```

---

# Cancellation-aware funkcija

Dobra concurrency funkcija treba da prihvata:

```go
context.Context
```

---

Primer:

```go
func Process(
	ctx context.Context,
) error {

}
```

---

Zašto?

Zato što caller može kontrolisati:

- timeout,
- cancellation,
- shutdown.

---

# Loš dizajn

Primer:

```go
func Process() error {

	for {

		doWork()

	}

}
```

---

Problem:

Nema način da kaže:

```
stani
```

---

Ako se pokrene:

```go
go Process()
```

---

Nema kontrole.

---

# Bolji dizajn

```go
func Process(
	ctx context.Context,
) error {

	for {

		select {

		case <-ctx.Done():

			return ctx.Err()


		default:

			doWork()

		}

	}

}
```

---

Sada funkcija:

- poštuje cancellation,
- završava kontrolisano,
- vraća razlog prekida.

---

# Pravilo: Context ide prvi argument

Go konvencija:

```go
func DoSomething(
	ctx context.Context,
	arg string,
) error
```

---

Ne:

```go
func DoSomething(
	arg string,
	ctx context.Context,
)
```

---

Standard:

```go
ctx prvi
```

---

# Context se ne čuva u struct-u

Česta greška:

```go
type Service struct {

	ctx context.Context

}
```

---

Zašto je loše?

Context predstavlja:

```
jedan request ili operaciju
```

---

Nije:

```
životni vek objekta
```

---

Bolje:

```go
type Service struct{}


func (s Service)
	Process(
		ctx context.Context,
	){

}
```

---

# Propagacija context-a kroz slojeve

Tipična arhitektura:

```
HTTP Handler

      |
      v

Service Layer

      |
      v

Repository

      |
      v

Database
```

---

Svaki sloj:

prima:

```go
context.Context
```

---

i prosleđuje dalje.

---

Primer:

## Handler

```go
func Handler(
	ctx context.Context,
){

	service.CreateUser(ctx)

}
```

---

## Service

```go
func CreateUser(
	ctx context.Context,
){

	repository.Save(ctx)

}
```

---

## Repository

```go
func Save(
	ctx context.Context,
){

	db.ExecContext(
		ctx,
		query,
	)

}
```

---

Rezultat:

Jedan cancellation signal kontroliše ceo lanac.

---

# Cancellation u CPU-bound operacijama

Primer:

teška računica:

```go
func Calculate(
	ctx context.Context,
) {

	for i:=0;i<1000000000;i++ {

		compute()

	}

}
```

---

Problem:

Nema blokiranja.

---

Nema:

```go
channel receive
```

---

Zato moramo sami proveravati:

```go
func Calculate(
	ctx context.Context,
){

	for i:=0;i<1000000000;i++ {


		if i%1000==0 {

			select {

			case <-ctx.Done():

				return


			default:

			}

		}


		compute()

	}

}
```

---

Periodično proveravanje.

---

# Cancellation u I/O operacijama

Kod I/O operacija lakše:

Primer:

```go
select {

case data := <-ch:

	process(data)


case <-ctx.Done():

	return

}
```

---

Isto važi za:

- HTTP,
- database,
- network,
- channels.

---

# Cancellation sa više faza

Primer:

```
Download

   |

Process

   |

Save

   |

Notify
```

---

Svaka faza:

```go
ctx
```

---

Ako download otkaže:

sve ostalo staje.

---

# Greška: Ignorisanje error-a context-a

Loše:

```go
case <-ctx.Done():

	return nil
```

---

Problem:

Caller ne zna:

- timeout?
- manual cancel?

---

Bolje:

```go
case <-ctx.Done():

	return ctx.Err()
```

---

# Greška: Provera context-a samo jednom

Loše:

```go
if ctx.Err()!=nil {

	return

}


for {

	work()

}
```

---

Ako rad traje dugo:

cancel može doći posle provere.

---

Bolje:

periodično proveravati.

---

# Greška: Kreiranje context.Background() unutra

Loše:

```go
func Save(){

	ctx :=
		context.Background()

}
```

---

Time prekidamo lanac.

---

Gubimo:

- parent timeout,
- parent cancellation.

---

# Best practices

---

## 1. Svaka dugoživeća funkcija prima context

Dobro:

```go
func Worker(ctx context.Context)
```

---

---

## 2. Context se prosleđuje, ne skladišti

---

---

## 3. Proveravati `ctx.Done()`

Kod:

- loop-ova,
- Goroutines,
- čekanja.

---

---

## 4. Vraćati `ctx.Err()`

Omogućava dijagnostiku.

---

---

## 5. Cleanup pre return-a

Primer:

```go
case <-ctx.Done():

	close(file)

	releaseLock()

	return ctx.Err()
```

---

# Mentalni model

Zapamti:

```
Cancellation

nije prekid

nego zahtev za prekid
```

---

Dobra Goroutine:

```
start

 |

work

 |

ctx.Done()

 |

cleanup

 |

return
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je cooperative cancellation,
- zašto Go nema force kill Goroutines,
- kako dizajnirati cancellation-aware funkcije,
- pravilnu upotrebu `context.Context`,
- propagaciju context-a kroz slojeve,
- razliku između CPU-bound i I/O cancellation-a,
- best practices za production kod.

---

# Cancellation — Worker Shutdown Pattern i Graceful Shutdown

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- Šta je graceful shutdown
- Problem naglog gašenja aplikacije
- Worker lifecycle
- Worker shutdown pattern
- Shutdown signal
- Context-based worker shutdown
- Više worker-a i zajednički cancellation
- Cleanup resursa
- Graceful shutdown HTTP servera
- Best practices

---

# Uvod

U prethodnim delovima naučili smo kako:

```go
cancel()
```

šalje signal jednoj ili više Goroutines.

---

Ali u realnoj aplikaciji imamo mnogo više komponenti:

```
HTTP Server

     |

Background Workers

     |

Message Consumers

     |

Database Connections

     |

Cache

```

---

Kada aplikacija treba da se ugasi:

ne želimo:

```
kill process
```

---

Želimo:

```
signal

↓

zaustavi nove poslove

↓

završi postojeće

↓

oslobodi resurse

↓

exit
```

---

To nazivamo:

```text
Graceful Shutdown
```

---

# Šta je graceful shutdown?

Graceful shutdown znači:

> Kontrolisano gašenje aplikacije bez gubitka podataka i bez ostavljanja resursa u lošem stanju.

---

Primer:

Imamo:

```
100 HTTP zahteva
```

---

Server dobije:

```
shutdown signal
```

---

Dobar sistem:

```
ne prima nove zahteve

+

završi postojeće

+

zatvara konekcije
```

---

Loš sistem:

```
kill

↓

prekinute operacije

↓

izgubljeni podaci
```

---

# Problem naglog gašenja

Primer:

```go
func main(){

	go processPayments()

	select{}

}
```

---

Ako proces bude ugašen:

```
processPayments()
```

može biti:

- usred transakcije,
- usred upisa,
- usred slanja poruke.

---

Rezultat:

```
nekonzistentno stanje
```

---

# Worker lifecycle

Dobar worker ima životni ciklus:

```
CREATE

  |

START

  |

WAIT JOB

  |

PROCESS

  |

STOP SIGNAL

  |

FINISH CURRENT JOB

  |

CLEANUP

  |

EXIT
```

---

# Osnovni worker pattern

Primer:

```go
func worker(
	ctx context.Context,
	jobs <-chan int,
){

	for {

		select {

		case <-ctx.Done():

			return


		case job := <-jobs:

			process(job)

		}

	}

}
```

---

Worker ima dve mogućnosti:

---

## 1. Dobija posao

```go
case job := <-jobs:
```

---

## 2. Dobija shutdown signal

```go
case <-ctx.Done():
```

---

# Pokretanje worker-a

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)


defer cancel()


jobs :=
	make(chan int)


go worker(
	ctx,
	jobs,
)
```

---

Shutdown:

```go
cancel()
```

---

Tok:

```
main

 |

cancel()

 |

ctx.Done()

 |

worker return
```

---

# Više worker-a

Realni sistemi često imaju:

```
Worker 1

Worker 2

Worker 3

Worker 4
```

---

Svi dele:

```go
ctx
```

---

Primer:

```go
for i:=0;i<4;i++{

	go worker(
		ctx,
		jobs,
	)

}
```

---

Shutdown:

```go
cancel()
```

---

Rezultat:

```
Worker 1 stopped

Worker 2 stopped

Worker 3 stopped

Worker 4 stopped
```

---

# Problem: Worker može biti usred posla

Primer:

```
Worker

 |

processing payment

 |

cancel()
```

---

Pitanje:

Da li odmah prekidamo?

---

Zavisi od sistema.

---

Opcije:

---

## Option 1 — Immediate shutdown

Odmah:

```go
return
```

---

Koristi se za:

- cache refresh,
- metrike,
- monitoring.

---

---

## Option 2 — Finish current job

Često bolje:

```
trenutni posao završi

novi poslovi se ne uzimaju
```

---

Primer:

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

---

Ako cancellation stigne tokom:

```go
process(job)
```

---

process treba sam da podrži context.

---

# Context-aware job processing

Loše:

```go
func process(job Job){

	time.Sleep(
		10*time.Second,
	)

}
```

---

Ne može se prekinuti.

---

Bolje:

```go
func process(
	ctx context.Context,
	job Job,
) error {

	select {

	case <-time.After(
		10*time.Second,
	):

		return nil


	case <-ctx.Done():

		return ctx.Err()

	}

}
```

---

Sada posao može biti otkazan.

---

# Worker + WaitGroup pattern

Čest obrazac:

```
Context

+

WaitGroup
```

---

Primer:

```go
var wg sync.WaitGroup


for i:=0;i<4;i++{

	wg.Add(1)


	go func(){

		defer wg.Done()

		worker(ctx)

	}()

}
```

---

Shutdown:

```go
cancel()


wg.Wait()
```

---

Tok:

```
cancel

↓

workers stop

↓

WaitGroup čeka

↓

program exit
```

---

# HTTP Server graceful shutdown

Go standardna biblioteka podržava:

```go
server.Shutdown(ctx)
```

---

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		5*time.Second,
	)

defer cancel()


server.Shutdown(ctx)
```

---

Značenje:

```
prestani primati nove konekcije

+

sačekaj postojeće
```

---

# Signal handling

Operativni sistemi šalju:

```
SIGINT

SIGTERM
```

---

Primer:

```go
signal.Notify(
	stop,
	os.Interrupt,
	syscall.SIGTERM,
)
```

---

Čekanje:

```go
<-stop
```

---

Zatim:

```go
cancel()
```

---

Arhitektura:

```
OS Signal

    |

 main

    |

 cancel()

    |

 workers

    |

 shutdown
```

---

# Cleanup resursa

Graceful shutdown treba zatvoriti:

---

## Database

```go
db.Close()
```

---

## Files

```go
file.Close()
```

---

## Channels

```go
close(ch)
```

---

## Network

```go
listener.Close()
```

---

# Redosled gašenja

Preporučeni redosled:

```
1. Zaustavi prihvatanje novih zahteva

2. Pošalji cancellation signal

3. Sačekaj aktivne poslove

4. Zatvori resurse

5. Exit
```

---

# Česte greške

---

## Greška 1

Ignorisanje shutdown-a.

Loše:

```go
for {

	work()

}
```

---

---

## Greška 2

Force kill procesa.

Loše:

```
os.Exit()
```

bez cleanup-a.

---

---

## Greška 3

Nema timeout-a za shutdown.

Loše:

```go
wg.Wait()
```

zauvek.

---

Bolje:

```go
context.WithTimeout()
```

---

# Production pattern

Najčešći oblik:

```
main

 |

create root context

 |

start services

 |

wait signal

 |

cancel()

 |

WaitGroup.Wait()

 |

cleanup()

 |

exit
```

---

# Mentalni model

Zapamti:

```
Graceful shutdown

=

signal

+

cancellation

+

wait

+

cleanup
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je graceful shutdown,
- worker lifecycle,
- worker shutdown pattern,
- korišćenje context-a za gašenje worker-a,
- rad sa više worker-a,
- kombinaciju `context + WaitGroup`,
- HTTP server shutdown,
- signal handling,
- pravilni redosled gašenja aplikacije.

---

# Cancellation — HTTP Serveri, Background Task-ovi i Production Servisi

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- Cancellation u HTTP serverima
- Request lifecycle
- Client disconnect scenario
- Background task cancellation
- Long-running procesi
- Service lifecycle management
- Context u mikroservisima
- Production cancellation arhitektura

---

# Uvod

U prethodnim delovima naučili smo:

- kako kreirati cancellation signal,
- kako zaustaviti worker-e,
- kako implementirati graceful shutdown.

---

Sada prelazimo na realne production scenarije:

```
HTTP Request

Background Jobs

Message Consumers

Microservices

Schedulers
```

---

# HTTP Request lifecycle

Svaki HTTP zahtev ima svoj životni ciklus:

```
Client

   |
   |
Request

   |
   v

Handler

   |
   v

Service

   |
   v

Database

   |
   v

Response
```

---

Go automatski kreira:

```go
context.Context
```

za svaki request.

---

Primer:

```go
func handler(
	w http.ResponseWriter,
	r *http.Request,
){

	ctx := r.Context()

}
```

---

Ovaj context sadrži:

- cancellation signal,
- deadline,
- request metadata.

---

# Client disconnect scenario

Zamislimo:

Korisnik šalje zahtev:

```
GET /report
```

---

Server:

```
generiše veliki report
```

---

Korisnik:

```
zatvara browser
```

---

Šta treba da se desi?

---

Loš sistem:

```
Client odlazi

↓

Server nastavlja rad

↓

CPU troši resurse
```

---

Dobar sistem:

```
Client odlazi

↓

Request context cancelled

↓

Service prekida rad
```

---

# HTTP context cancellation primer

```go
func reportHandler(
	w http.ResponseWriter,
	r *http.Request,
){

	ctx := r.Context()


	err :=
		generateReport(ctx)


	if err != nil {

		http.Error(
			w,
			err.Error(),
			500,
		)

		return

	}

}
```

---

Service:

```go
func generateReport(
	ctx context.Context,
) error {


	for i:=0;i<100;i++{


		select {

		case <-ctx.Done():

			return ctx.Err()


		default:

			createPart()

		}

	}


	return nil

}
```

---

Ako klijent prekine:

```
ctx.Done()
```

se aktivira.

---

# Database cancellation u HTTP request-u

Čest lanac:

```
HTTP Request

      |

Service

      |

Repository

      |

Database
```

---

Handler:

```go
ctx := r.Context()
```

---

Repository:

```go
db.QueryContext(
	ctx,
	query,
)
```

---

Ako korisnik ode:

```
HTTP context cancelled

        |

Database query cancelled
```

---

Jedan signal kontroliše ceo lanac.

---

# Background task-ovi

Nisu svi poslovi vezani za HTTP.

---

Primeri:

- email sender,
- cleanup job,
- cache refresh,
- scheduler,
- queue consumer.

---

Oni često rade:

```
24/7
```

---

Zato moraju imati:

```
shutdown mehanizam
```

---

# Loš background worker

Primer:

```go
func startCleanup(){

	go func(){

		for {

			cleanup()

			time.Sleep(
				time.Hour,
			)

		}

	}()

}
```

---

Problem:

Nema:

- stop signal,
- cleanup,
- kontrolu.

---

# Bolji background worker

```go
func cleanupWorker(
	ctx context.Context,
){

	ticker :=
		time.NewTicker(
			time.Hour,
		)

	defer ticker.Stop()


	for {

		select {

		case <-ticker.C:

			cleanup()


		case <-ctx.Done():

			return

		}

	}

}
```

---

Sada imamo:

```
start

 |

run

 |

cancel

 |

stop
```

---

# Long-running procesi

Primer:

```
Kafka consumer

Queue listener

Event processor
```

---

Arhitektura:

```
main

 |

context

 |

consumer loop

 |

message processing
```

---

Primer:

```go
func consumer(
	ctx context.Context,
	messages <-chan Message,
){

	for {

		select {

		case <-ctx.Done():

			return


		case msg := <-messages:

			handle(msg)

		}

	}

}
```

---

# Cancellation kod message processing-a

Važno:

Nije dovoljno zaustaviti consumer.

---

Može postojati:

```
trenutni message processing
```

---

Primer:

```
Message A

processing...

cancel()
```

---

Dizajn:

```go
func handle(
	ctx context.Context,
	msg Message,
) error
```

---

Ako obrada traje dugo:

```go
select {

case <-ctx.Done():

	return ctx.Err()


default:

	processStep()

}
```

---

# Service lifecycle management

Veće aplikacije imaju više servisa:

```
HTTP Server

Worker Pool

Metrics

Queue Consumer

Scheduler
```

---

Svaki servis treba:

```go
Start(ctx)

Stop(ctx)
```

---

Primer:

```go
type Service interface {

	Start(
		ctx context.Context,
	)

	Stop(
		ctx context.Context,
	)

}
```

---

# Primer service manager-a

```go
func RunServices(
	ctx context.Context,
	services []Service,
){

	for _,s :=
		range services {

		go s.Start(ctx)

	}

}
```

---

Shutdown:

```go
cancel()
```

---

Svi servisi dobijaju:

```go
ctx.Done()
```

---

# Context u mikroservisima

U distributed sistemima:

```
Service A

      |

Service B

      |

Service C
```

---

Request deadline treba da se propagira.

---

Primer:

```
User request

timeout 10s
```

---

Service A:

```
ostalo 7s
```

---

Service B:

```
ostalo 4s
```

---

Database:

```
ostalo 2s
```

---

Bez propagacije:

svaki servis radi nezavisno.

---

Sa context propagation:

ceo sistem poštuje isti rok.

---

# Production arhitektura

Tipičan Go servis:

```
main

 |

root context

 |

--------------------------------

HTTP Server

Worker Pool

Consumer

Scheduler

Metrics

--------------------------------

 |

cancel()

 |

shutdown

 |

cleanup
```

---

# Cancellation granice

Dobar sistem ima jasne granice:

---

## Request granica

```text
HTTP context
```

---

## Application granica

```text
root context
```

---

## Worker granica

```text
child context
```

---

# Česte greške

---

## Greška 1

Pokretanje background task-a bez context-a.

Loše:

```go
go task()
```

---

Bolje:

```go
go task(ctx)
```

---

## Greška 2

Ignorisanje request cancellation-a.

Loše:

```go
process()
```

---

Bolje:

```go
process(
	r.Context(),
)
```

---

## Greška 3

Mešanje request context-a i globalnog context-a.

Request:

```
trajanje sekunde
```

---

Application:

```
trajanje meseci
```

---

Ne treba ih mešati.

---

# Mentalni model

Zapamti:

```
HTTP request

=

privremeni životni vek


Application context

=

životni vek procesa
```

---

Cancellation povezuje:

```
signal

↓

komponente

↓

kontrolisan završetak
```

---

# 📋 Rezime

U ovom delu naučili smo:

- cancellation u HTTP serverima,
- request lifecycle,
- client disconnect scenario,
- korišćenje `r.Context()`,
- cancellation database poziva,
- background task shutdown,
- long-running worker-e,
- service lifecycle management,
- context propagation u mikroservisima.

---

# Cancellation — Napredni pattern-i, best practices i praktični zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 5/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/05-cancellation.md`

---

# 📚 Sadržaj ovog dela

- Context hijerarhija u velikim sistemima
- Cancellation tree model
- Multiple cancellation sources
- Cancellation propagation patterns
- Shutdown orchestration
- Cancellation anti-pattern-i
- Production checklist
- Pitanja za proveru znanja
- Praktični zadaci

---

# Uvod

Do sada smo naučili:

- kako kreirati cancellation signal,
- kako proslediti context kroz aplikaciju,
- kako zaustaviti worker-e,
- kako implementirati graceful shutdown.

---

Sada ćemo spojiti sve koncepte u:

```
production concurrency dizajn
```

---

# Context hijerarhija

Go context formira stablo.

Primer:

```
context.Background()

          |
          |
      Application

          |
    ----------------

    |              |

 HTTP Request    Worker

    |

 Database
```

---

Cancellation ide:

```
parent

  ↓

children

  ↓

grandchildren
```

---

Ako se parent otkaže:

svi potomci dobijaju signal.

---

# Cancellation tree model

Zamisli aplikaciju:

```
Root Context

      |
      |
 Application

      |
 ----------------------

 |          |          |

HTTP     Worker     Consumer

 |
 |
Database
```

---

Poziv:

```go
rootCancel()
```

---

Rezultat:

```
HTTP stopped

Worker stopped

Consumer stopped

Database operations cancelled
```

---

Jedan signal kontroliše ceo sistem.

---

# Multiple cancellation sources

U realnom sistemu cancellation može doći iz više razloga.

---

Primer:

```
                 User Cancel

                      |

                      |

Timeout -------- Context -------- Shutdown Signal

                      |

                      |

                 Worker Stop
```

---

Izvori:

- timeout,
- user cancellation,
- OS signal,
- server shutdown,
- parent service failure.

---

# Primer: kombinovanje izvora

Često imamo:

```go
ctx, cancel :=
	context.WithCancel(
		parent,
	)
```

---

Jedan izvor:

```go
cancel()
```

---

Ali context može biti otkazan i od parent-a:

```
parent cancelled

        |

        v

child cancelled
```

---

---

# Cancellation propagation pattern

Najčešći production pattern:

```
main()

 |

create root context

 |

start components

 |

wait signal

 |

cancel()

 |

wait cleanup

 |

exit
```

---

Primer:

```go
func main(){

	ctx, cancel :=
		context.WithCancel(
			context.Background(),
		)

	defer cancel()


	startWorkers(ctx)


	waitShutdownSignal()

}
```

---

---

# Shutdown orchestration

U većim sistemima ne želimo:

```go
cancel()
```

i ništa više.

---

Potrebno je:

```
koordinisano gašenje
```

---

Primer:

```
Shutdown Manager


1. Stop HTTP server

2. Stop accepting jobs

3. Cancel workers

4. Flush queues

5. Close database

6. Exit
```

---

# Primer shutdown manager-a

```go
type Manager struct {

	services []Service

}
```

---

Shutdown:

```go
func (m *Manager)
Shutdown(ctx context.Context)
{

	for _,service :=
		range m.services {

		service.Stop(ctx)

	}

}
```

---

---

# Cancellation i WaitGroup

Čest pattern:

```
Context

+

WaitGroup
```

---

Primer:

```go
var wg sync.WaitGroup


for i:=0;i<5;i++{

	wg.Add(1)


	go func(){

		defer wg.Done()

		worker(ctx)

	}()

}
```

---

Shutdown:

```go
cancel()

wg.Wait()
```

---

Tok:

```
cancel

↓

workers exit

↓

Done()

↓

program završava
```

---

# Cancellation i channel close

Ponekad se koristi:

```go
close(channel)
```

za signal.

---

Primer:

```go
done := make(chan struct{})
```

---

Stop:

```go
close(done)
```

---

Ali:

context je pogodniji kada imamo:

- više slojeva,
- timeout,
- standardne biblioteke.

---

# Context vs Stop Channel

| | Context | Stop Channel |
|-|-|-|
| Cancellation | ✅ | ✅ |
| Timeout | ✅ | ❌ |
| Deadline | ✅ | ❌ |
| HTTP podrška | ✅ | ❌ |
| Database podrška | ✅ | ❌ |
| Standardni Go pattern | ✅ | delimično |

---

# Cancellation anti-pattern-i

---

# ❌ 1. Ignorisanje context-a

Loše:

```go
func process(){

	for {

		work()

	}

}
```

---

Problem:

nema kontrole.

---

---

# ❌ 2. Čuvanje context-a u struct-u

Loše:

```go
type Worker struct {

	ctx context.Context

}
```

---

Context pripada operaciji.

---

Ne objektu.

---

---

# ❌ 3. Kreiranje novog Background context-a

Loše:

```go
func Save(){

	ctx :=
		context.Background()

}
```

---

Gubi se:

- timeout,
- cancellation.

---

---

# ❌ 4. Ignorisanje ctx.Err()

Loše:

```go
case <-ctx.Done():

	return nil
```

---

Bolje:

```go
case <-ctx.Done():

	return ctx.Err()
```

---

---

# ❌ 5. Prekid bez cleanup-a

Loše:

```go
case <-ctx.Done():

	return
```

---

Ako postoji:

- file,
- lock,
- connection,

mora cleanup.

---

# Production checklist

Pre produkcije proveriti:

---

## Context dizajn

✅ Svaka dugoživeća funkcija prima:

```go
context.Context
```

---

## Propagacija

✅ Context ide kroz:

```
handler

service

repository
```

---

## Goroutines

✅ Svaka Goroutine ima:

```
stop mehanizam
```

---

## Workers

✅ Imaju:

```
ctx.Done()
```

---

## Shutdown

✅ Postoji:

```
graceful shutdown
```

---

## Cleanup

✅ Resursi se zatvaraju:

- database,
- files,
- connections,
- channels.

---

# Pitanja za proveru znanja

---

## 1.

Zašto Go nema:

```text
kill goroutine
```

?

---

## 2.

Koja je razlika:

```go
context.Canceled
```

i:

```go
context.DeadlineExceeded
```

?

---

## 3.

Zašto context treba proslediti kroz funkcije?

---

## 4.

Zašto nije dobro čuvati context u struct-u?

---

## 5.

Koja je uloga:

```go
ctx.Done()
```

?

---

## 6.

Šta znači:

```text
cooperative cancellation
```

?

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnovni

Napraviti worker:

```go
func worker(
	ctx context.Context,
)
```

---

Zahtevi:

- radi u petlji,
- proverava cancellation,
- završava pravilno.

---

# 🟡 Nivo 2 — Worker Pool

Napraviti:

```
5 worker-a

+

jobs channel
```

---

Zahtevi:

- zajednički context,
- graceful shutdown,
- WaitGroup.

---

# 🟠 Nivo 3 — HTTP Service

Napraviti:

```
HTTP Handler

↓

Service

↓

Repository
```

---

Zahtevi:

- propagacija context-a,
- database timeout,
- client disconnect handling.

---

# 🔴 Nivo 4 — Senior

Napraviti servis:

```
HTTP Server

Worker Pool

Background Scheduler

Database

Message Consumer
```

---

Implementirati:

- root context,
- signal handling,
- graceful shutdown,
- cleanup,
- timeout shutdown.

---

# Završni mentalni model

Zapamti:

```
Context

=

životni vek operacije


Cancellation

=

signal za završetak


Goroutine

=

mora sarađivati


Graceful shutdown

=

signal

+

čekanje

+

cleanup
```

---

### ➡️ Sledeća lekcija **[**Go Scheduler**](06-go-scheduler.md)**

Obradićemo:

- kako Go runtime raspoređuje Goroutines,
- G-M-P model,
- work stealing,
- scheduler lifecycle,
- preemption,
- veze između concurrency i scheduler-a.
