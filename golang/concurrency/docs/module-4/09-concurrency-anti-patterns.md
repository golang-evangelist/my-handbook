# Concurrency Anti-Patterns

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/09-concurrency-anti-patterns.md`

---

# 📚 Sadržaj

- Šta su concurrency anti-patterns
- Zašto su opasni
- Goroutine explosion
- Nekontrolisano kreiranje Goroutines
- Nedostatak backpressure-a
- Production posledice

---

# Uvod

Concurrency omogućava:

```text
više posla istovremeno
```

Ali bez kontrole može dovesti do:

- prevelike potrošnje memorije,
- preopterećenja CPU-a,
- blokiranih sistema,
- curenja resursa.

---

# Šta je Anti-Pattern?

Anti-pattern je:

> Rešenje koje izgleda ispravno, ali dugoročno proizvodi probleme.

---

Primer:

Početnik vidi:

```go
go process()
```

i razmišlja:

```
Brže!
```

---

Ali sistemsko pitanje je:

```
Koliko Goroutines?

Ko ih kontroliše?

Kada završavaju?
```

---

# Anti-Pattern #1

# Goroutine Explosion

Najpoznatija greška.

---

Primer:

```go
for _, item := range items {

	go process(item)

}
```

---

Na prvi pogled:

Elegantno.

---

Problem:

Ako:

```
items = 10 miliona
```

dobijamo:

```
10 miliona Goroutines
```

---

# Šta se dešava?

Svaka Goroutine ima:

- stack memoriju,
- scheduling overhead,
- runtime metadata.

---

Rezultat:

```text
Memory ↑

Scheduler overhead ↑

Latency ↑
```

---

# Loš dizajn

```text
1000000 Jobs

↓

1000000 Goroutines

↓

Database
```

---

Problem:

Database ne može obraditi toliko zahteva.

---

# Bolji dizajn

Koristimo:

## Worker Pool

```text
1000000 Jobs

↓

Queue

↓

20 Workers

↓

Database
```

---

Sistem ima kontrolu.

---

# Anti-Pattern #2

# Goroutine bez vlasnika

Primer:

```go
func start() {

	go func(){

		for {

			doWork()

		}

	}()

}
```

---

Pitanje:

Ko zaustavlja ovu Goroutine?

Odgovor:

Niko.

---

Rezultat:

```
Goroutine Leak
```

---

# Goroutine Leak

To znači:

Goroutine više nije korisna, ali i dalje postoji.

---

Primer:

```text
Request završen

↓

Goroutine ostala

↓

Čeka zauvek
```

---

Posledice:

- memorija se ne oslobađa,
- broj Goroutines raste,
- servis degradira.

---

# Pravilno upravljanje lifecycle-om

Koristi:

```go
context.Context
```

Primer:

```go
for {

	select {

	case <-ctx.Done():
		return

	default:
		work()

	}

}
```

---

Sada postoji:

```
Start

↓

Run

↓

Cancel

↓

Exit
```

---

# Anti-Pattern #3

# Fire-and-forget Goroutines

Primer:

```go
go sendEmail()
```

---

Problem:

Šta ako:

- email padne?
- aplikacija se ugasi?
- error se izgubi?

---

Ovakav kod:

```go
go operation()
```

nema:

- kontrolu,
- rezultat,
- error handling.

---

Bolje:

```go
resultCh := make(chan error)

go func(){

	resultCh <- operation()

}()
```

---

Ili:

koristiti:

- Worker Pool,
- errgroup,
- Queue sistem.

---

# Anti-Pattern #4

# Nema Backpressure-a

Problem:

Producer je brži od Consumer-a.

---

Primer:

```text
Producer

10000 events/sec


Consumer

100 events/sec
```

---

Rezultat:

```text
Buffer raste

↓

Memory raste

↓

Crash
```

---

# Loš primer

```go
ch := make(chan Event, 1000000)
```

---

Veliki buffer nije rešenje.

Samo odlaže problem.

---

# Bolji pristup

Koristiti:

- ograničene buffere,
- Worker Pool,
- Semaphore,
- Rate Limiter.

---

# Production primer

HTTP servis:

Loše:

```text
Request

↓

go process()

↓

Database
```

---

Dobro:

```text
Request

↓

Rate Limiter

↓

Worker Pool

↓

Semaphore

↓

Database
```

---

# Najvažnije pitanje

Za svaku Goroutine:

Pitaj:

## Ko je vlasnik?

```
Ko je kreirao?
```

---

## Ko je zaustavlja?

```
Kako završava?
```

---

## Kako javlja grešku?

```
Kako znamo da je pukla?
```

---

Ako nema odgovora:

verovatno postoji problem.

---

# Best Practices

✅ Ograniči broj aktivnih Goroutines.

✅ Svaka Goroutine mora imati lifecycle.

✅ Koristi `context.Context`.

✅ Ne ignoriši error-e.

✅ Uvedi backpressure.

---

# Mentalni model

Nemoj razmišljati:

```text
Goroutine je besplatna
```

Razmišljaj:

```text
Goroutine je resurs
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su concurrency anti-patterns

✅ šta je Goroutine Explosion

✅ kako nastaje Goroutine Leak

✅ zašto je fire-and-forget opasan

✅ zašto je backpressure važan

---

# Concurrency Anti-Patterns — Channel problemi

---

# 📚 Sadržaj

- Channel ownership pravilo
- Nikada zatvoren kanal
- Pogrešno zatvaranje kanala
- Send on closed channel
- Deadlock preko kanala
- Nil channel problemi
- Neograničeni channel buffer
- Best practices

---

# Uvod

Channel je komunikacioni mehanizam.

Primer:

```go
ch := make(chan int)

ch <- 10

value := <-ch
```

---

Ali channel nije samo cev.

On ima:

- vlasnika,
- lifecycle,
- pravila zatvaranja.

---

# Channel Ownership

Najvažnije pravilo:

> Goroutine koja šalje podatke je odgovorna za zatvaranje kanala.

---

Primer:

```text
Producer

↓

Channel

↓

Consumer
```

---

Producer zna:

```
Kada više nema podataka
```

Zato on zatvara:

```go
close(ch)
```

---

# Pravilan primer

```go
func producer() <-chan int {

	ch := make(chan int)

	go func(){

		defer close(ch)

		for i := 0; i < 10; i++ {

			ch <- i

		}

	}()

	return ch
}
```

---

Consumer:

```go
for value := range producer(){

	fmt.Println(value)

}
```

---

# Anti-Pattern #1

# Kanal nikada nije zatvoren

Primer:

```go
func producer() <-chan int {

	ch := make(chan int)

	go func(){

		for i := 0; i < 10; i++ {

			ch <- i

		}

	}()

	return ch
}
```

---

Problem:

Consumer:

```go
for v := range ch {

}
```

nikada ne završava.

---

Rezultat:

```text
Consumer čeka zauvek
```

---

# Simptom

Program izgleda:

```
nema error-a

ali ne završava
```

---

# Anti-Pattern #2

# Consumer zatvara channel

Loš primer:

```go
for value := range ch {

	process(value)

}

close(ch)
```

---

Zašto je loše?

Consumer ne zna:

```
Da li producer još šalje?
```

---

Ako producer pošalje:

```go
ch <- value
```

dobijamo:

```
panic:
send on closed channel
```

---

# Anti-Pattern #3

# Više sendera zatvara isti kanal

Primer:

```text
Producer 1

↓

Channel

↑

Producer 2
```

---

Ako oba rade:

```go
close(ch)
```

dobijamo:

```
panic:
close of closed channel
```

---

# Rešenje

Jedan vlasnik zatvaranja.

Ako ima više proizvođača:

koristi:

- koordinaciju,
- WaitGroup,
- poseban closer.

---

Primer:

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {

	wg.Add(1)

	go producer(&wg, ch)

}


go func(){

	wg.Wait()

	close(ch)

}()
```

---

# Anti-Pattern #4

# Send on closed channel

Primer:

```go
close(ch)

ch <- 10
```

---

Rezultat:

```text
panic:
send on closed channel
```

---

Prevencija:

- jasno ownership pravilo,
- dokumentovan lifecycle.

---

# Anti-Pattern #5

# Deadlock preko kanala

Primer:

```go
ch := make(chan int)

ch <- 10

fmt.Println("done")
```

---

Problem:

Unbuffered channel čeka receiver.

---

Tok:

```text
Sender

↓

čeka

↓

Nema receiver-a
```

---

Rezultat:

```
fatal error:
all goroutines are asleep - deadlock!
```

---

# Rešenje #1

Dodaj receiver:

```go
go func(){

	fmt.Println(<-ch)

}()
```

---

# Rešenje #2

Buffered channel:

```go
ch := make(chan int, 1)

ch <- 10
```

---

Ali:

Buffer nije univerzalno rešenje.

---

# Anti-Pattern #6

# Nil Channel

Primer:

```go
var ch chan int

ch <- 10
```

---

`ch` je:

```go
nil
```

---

Operacije:

| Operacija | Rezultat |
|-|-|
| Send | blokira zauvek |
| Receive | blokira zauvek |
| Close | panic |

---

# Poseban slučaj

Nil channel u select-u:

```go
select {

case value := <-ch:

	fmt.Println(value)

}
```

Ako je:

```go
ch == nil
```

case je deaktiviran.

---

Ovo se nekad koristi namerno.

---

# Anti-Pattern #7

# Ogroman buffered channel

Primer:

```go
make(chan Event, 10000000)
```

---

Izgleda kao:

```
više performansi
```

Ali:

Problem:

```
Producer brži

↓

memorija raste
```

---

Buffer samo skriva problem.

---

# Backpressure pristup

Bolje:

```text
Producer

↓

Limited Channel

↓

Consumer
```

---

Ako je channel pun:

producer čeka.

To je prirodni backpressure.

---

# Channel Design Rules

## Pravilo #1

Sender zatvara.

---

## Pravilo #2

Receiver nikada ne zatvara.

---

## Pravilo #3

Jedan vlasnik lifecycle-a.

---

## Pravilo #4

Channel treba imati jasnu svrhu.

---

# Loš dizajn

```text
10 Goroutines

↓

jedan globalni channel

↓

svi rade sve
```

---

Problem:

- teško pratiti,
- teško gasiti,
- teško testirati.

---

# Bolji dizajn

```text
Producer

↓

Dedicated Channel

↓

Consumer
```

---

# Production Checklist

Pre korišćenja channel-a:

Pitaj:

## Ko šalje?

```
Owner?
```

---

## Ko zatvara?

```
Owner?
```

---

## Kako završava?

```
Close?
Context?
```

---

## Šta ako consumer kasni?

```
Backpressure?
```

---

# Mentalni model

Channel nije:

```text
Queue bez pravila
```

---

Channel je:

```text
Dogovor između Goroutines
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ channel ownership pravilo

✅ zašto producer zatvara kanal

✅ probleme sa zatvaranjem kanala

✅ send on closed channel grešku

✅ deadlock preko kanala

✅ nil channel ponašanje

✅ probleme velikih buffer-a

---

# Concurrency Anti-Patterns — Mutex problemi

---

# 📚 Sadržaj

- Preširoke kritične sekcije
- Zaboravljen Unlock
- Mutex kopiranje
- Deadlock sa više lock-ova
- Pogrešna upotreba RWMutex-a
- Mutex kao globalna sinhronizacija
- Atomic vs Mutex
- Best practices

---

# Uvod

Mutex rešava problem:

```text
Više Goroutines

↓

Isti podatak

↓

Kontrolisan pristup
```

---

Primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Ali pitanje nije:

> "Da li mogu staviti Mutex?"

nego:

> "Da li je ovo pravo mesto za Mutex?"

---

# Anti-Pattern #1

# Preširoka kritična sekcija

Najčešća greška.

---

Loš primer:

```go
mu.Lock()

loadData()

processData()

saveData()

mu.Unlock()
```

---

Problem:

Dok traje:

```
loadData()
processData()
saveData()
```

svi ostali čekaju.

---

Mutex štiti previše.

---

# Bolje

Zaštiti samo stanje:

```go
data := calculate()

mu.Lock()

cache[key] = data

mu.Unlock()
```

---

Pravilo:

> Kritična sekcija treba da bude što kraća.

---

# Anti-Pattern #2

# Zaboravljen Unlock

Loš primer:

```go
mu.Lock()

update()

return
```

---

Rezultat:

Mutex ostaje zaključan.

---

Drugi thread:

```go
mu.Lock()
```

čeka zauvek.

---

Rešenje:

Koristi:

```go
defer
```

---

Primer:

```go
mu.Lock()

defer mu.Unlock()

update()
```

---

# Anti-Pattern #3

# Kopiranje Mutex-a

Mutex ne sme da se kopira nakon korišćenja.

---

Loš primer:

```go
type Counter struct {

	mu sync.Mutex

	value int

}


func copy(c Counter){

}
```

---

Problem:

Dobijaš dve kopije lock-a.

---

Jedan štiti:

```
value A
```

Drugi:

```
value B
```

---

Rezultat:

Nema stvarne sinhronizacije.

---

# Pravilo

Strukture koje sadrže Mutex:

koristi kao pointer.

---

Dobro:

```go
func update(c *Counter)
```

---

Loše:

```go
func update(c Counter)
```

---

# Anti-Pattern #4

# Deadlock sa više Mutex-a

Primer:

Imamo:

```go
mutexA

mutexB
```

---

Goroutine 1:

```text
Lock A

↓

Lock B
```

---

Goroutine 2:

```text
Lock B

↓

Lock A
```

---

Rezultat:

```text
Deadlock
```

---

# Rešenje

Uvek isti redosled zaključavanja.

Primer:

Sistem:

```
A

↓

B
```

svuda koristi:

```go
Lock A

Lock B
```

---

# Anti-Pattern #5

# Pogrešna upotreba RWMutex-a

`RWMutex` omogućava:

- više reader-a,
- jedan writer.

---

Primer:

```go
mu.RLock()

read()

mu.RUnlock()
```

---

Problem nastaje kada:

većina operacija piše.

---

Primer:

```
90% Write

10% Read
```

---

RWMutex može biti sporiji od Mutex-a.

---

# Zašto?

Zbog dodatne koordinacije:

- reader tracking,
- writer waiting,
- scheduling.

---

# Pravilo

Koristi RWMutex kada:

```
mnogo više čitanja nego pisanja
```

---

# Anti-Pattern #6

# Mutex kao globalna sinhronizacija

Primer:

```go
var globalMu sync.Mutex
```

---

Sve funkcije:

```text
Lock

↓

rad

↓

Unlock
```

---

Problem:

Ceo sistem postaje:

```
jedna traka
```

---

Rezultat:

- manji concurrency,
- bottleneck,
- teško skaliranje.

---

# Bolje

Granularni lock:

Primer:

```go
User A

↓

Lock A


User B

↓

Lock B
```

---

# Anti-Pattern #7

# Mutex umesto Channel-a

Ponekad se koristi:

```go
Mutex + Shared State
```

za problem koji je zapravo:

```
Message Passing
```

---

Primer:

Queue sistem.

Loše:

```text
Global Slice

↓

Mutex

↓

Workers
```

---

Bolje:

```text
Channel

↓

Workers
```

---

# Atomic vs Mutex

Česta dilema.

---

# Atomic

Za jednostavne vrednosti:

- counter,
- flag,
- state.

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

# Mutex

Za kompleksno stanje:

```go
struct {
	users map[string]User
	cache []Item
}
```

---

Pravilo:

```text
Jedna promenljiva

↓

Atomic
```

---

```text
Više povezanih podataka

↓

Mutex
```

---

# Production pravila

✅ Kritične sekcije drži kratkim.

✅ Koristi `defer Unlock()`.

✅ Nikada ne kopiraj Mutex.

✅ Definiši redosled lock-ovanja.

✅ Ne koristi globalni Mutex bez potrebe.

✅ Razmisli o Atomic ili Channel pristupu.

---

# Mentalni model

Loše:

```text
Mutex

↓

ceo program
```

---

Dobro:

```text
Mutex

↓

mali deo deljenog stanja
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ preširoke kritične sekcije

✅ zaboravljen Unlock

✅ probleme kopiranja Mutex-a

✅ deadlock sa više lock-ova

✅ kada RWMutex nije dobar izbor

✅ Mutex vs Atomic

✅ Mutex vs Channel

---

# Concurrency Anti-Patterns — WaitGroup problemi

---

# 📚 Sadržaj

- Uloga WaitGroup-a
- Add/Done problemi
- WaitGroup Add unutar Goroutine
- Zaboravljen Done
- Kopiranje WaitGroup-a
- WaitGroup za rezultate
- WaitGroup vs errgroup
- Best practices

---

# Uvod

`sync.WaitGroup` rešava problem:

```text
Pokreni više Goroutines

↓

Sačekaj da sve završe
```

---

Osnovni primer:

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

Tok:

```text
Add

↓

Start Goroutine

↓

Done

↓

Wait završava
```

---

# Anti-Pattern #1

# Add() unutar Goroutine

Vrlo česta greška.

---

Loš primer:

```go
for i := 0; i < 10; i++ {

	go func(){

		wg.Add(1)

		defer wg.Done()

		work()

	}()

}

wg.Wait()
```

---

Problem:

`Wait()` može da se izvrši pre:

```go
wg.Add(1)
```

---

Mogući rezultat:

```
program završi prerano
```

---

# Pravilno

`Add()` mora biti pre pokretanja Goroutine.

---

Dobro:

```go
for i := 0; i < 10; i++ {

	wg.Add(1)

	go func(){

		defer wg.Done()

		work()

	}()

}
```

---

Pravilo:

> Broj poslova mora biti poznat pre čekanja.

---

# Anti-Pattern #2

# Zaboravljen Done()

Primer:

```go
wg.Add(1)

go func(){

	work()

}()
```

---

Problem:

Nikada se ne izvrši:

```go
wg.Done()
```

---

Rezultat:

```go
wg.Wait()
```

čeka zauvek.

---

Rešenje:

Uvek:

```go
defer wg.Done()
```

---

# Zašto defer?

Loše:

```go
work()

wg.Done()
```

Ako:

```go
work()
```

izazove panic ili raniji return:

Done se ne izvrši.

---

Bolje:

```go
defer wg.Done()
```

---

# Anti-Pattern #3

# WaitGroup za komunikaciju

Neki pokušavaju:

```go
wg.Wait()
```

da dobiju rezultat.

---

Primer:

```text
Worker

↓

rezultat

↓

WaitGroup
```

---

Problem:

WaitGroup samo kaže:

```
gotovo
```

Ne daje:

```
šta je rezultat
```

---

Za rezultate koristi:

- channel,
- result struct,
- errgroup.

---

# Pravi model

WaitGroup:

```text
Koordinacija
```

---

Channel:

```text
Komunikacija
```

---

# Anti-Pattern #4

# Kopiranje WaitGroup-a

Primer:

```go
type Job struct {

	wg sync.WaitGroup

}
```

---

Zatim:

```go
job2 := job1
```

---

Problem:

Dobijaš dve kopije internog stanja.

---

Rezultat:

nepredvidivo ponašanje.

---

Pravilo:

Objekte koji sadrže:

- Mutex,
- WaitGroup,
- Cond

ne kopirati.

---

Koristi:

```go
*Job
```

---

# Anti-Pattern #5

# WaitGroup bez lifecycle kontrole

Primer:

```go
for {

	wg.Add(1)

	go process()

}
```

---

Problem:

Broj Goroutines raste beskonačno.

---

WaitGroup ne ograničava:

- broj Goroutines,
- memoriju,
- opterećenje.

---

Za ograničenje koristi:

- Worker Pool,
- Semaphore,
- Rate Limiter.

---

# Anti-Pattern #6

# Ignorisanje error-a

Primer:

```go
wg.Add(1)

go func(){

	defer wg.Done()

	process()

}()
```

---

Ako:

```go
process()
```

vrati error:

ko ga dobija?

---

Odgovor:

Niko.

---

WaitGroup ne podržava error propagation.

---

# WaitGroup vs errgroup

Za ozbiljniji rad često se koristi:

:contentReference[oaicite:0]{index=0}

---

WaitGroup:

```text
Sačekaj sve
```

---

errgroup:

```text
Sačekaj

+

uhvati error

+

cancel ostale
```

---

Primer koncepta:

```text
Worker 1

↓

error

↓

Cancel Context

↓

Worker 2 stop
```

---

# Kada koristiti WaitGroup?

Dobro za:

- čekanje više nezavisnih operacija,
- jednostavnu koordinaciju.

Primer:

```text
3 cache refresh operacije

↓

čekaj sve
```

---

# Kada koristiti errgroup?

Bolje za:

- request obradu,
- paralelne API pozive,
- taskove gde jedan failure menja sve.

---

# Anti-Pattern #7

# WaitGroup umesto Worker Pool-a

Loš dizajn:

```go
for _, job := range jobs {

	wg.Add(1)

	go process(job)

}
```

---

Ako ima:

```
milion poslova
```

dobijaš:

```
milion Goroutines
```

---

Bolje:

```text
Jobs Channel

↓

Workers

↓

WaitGroup
```

---

WaitGroup može biti deo Worker Pool-a.

---

# Production pravila

✅ `Add()` pre `go`.

✅ `Done()` pomoću `defer`.

✅ Ne koristi WaitGroup za rezultate.

✅ Ne kopiraj WaitGroup.

✅ Koristi errgroup za error propagation.

✅ Kombinuj sa Worker Pool-om kada broj poslova nije mali.

---

# Mentalni model

WaitGroup nije:

```text
kontroler Goroutines
```

---

WaitGroup jeste:

```text
brojač završetka
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako pravilno koristiti WaitGroup

✅ zašto Add mora biti pre Goroutine

✅ zašto Done mora biti siguran

✅ zašto WaitGroup ne prenosi rezultate

✅ WaitGroup vs errgroup

✅ kada koristiti Worker Pool umesto čistog WaitGroup pristupa

---

# Concurrency Anti-Patterns — Context problemi

---

# 📚 Sadržaj

- Uloga context-a
- Context kao globalni storage
- Zaboravljen cancel
- Pogrešan timeout dizajn
- Context bez propagacije
- Context u struct-ovima
- Pogrešni Context ključevi
- Best practices

---

# Uvod

`context.Context` omogućava:

```text
Parent operation

↓

Child operations
```

da dele:

- cancellation signal,
- deadline,
- request metadata.

---

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	time.Second,
)

defer cancel()
```

---

Tok:

```text
Request

↓

Context

↓

Goroutines

↓

Cancel
```

---

# Anti-Pattern #1

# Zaboravljen cancel()

Vrlo česta greška.

---

Loš primer:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)

go worker(ctx)
```

---

Problem:

Niko nikada ne poziva:

```go
cancel()
```

---

Rezultat:

- resursi ostaju aktivni,
- timer ostaje,
- child goroutines mogu nastaviti rad.

---

Pravilno:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)

defer cancel()
```

---

# Pravilo

> Ako kreiraš Context sa cancel funkcijom, ti poseduješ odgovornost da pozoveš cancel.

---

# Anti-Pattern #2

# Context kao globalni storage

Loš primer:

```go
ctx = context.WithValue(
	ctx,
	"user",
	user,
)
```

i onda:

```go
user := ctx.Value("user")
```

svuda u aplikaciji.

---

Problem:

Context postaje:

```
globalna mapa
```

---

Posledice:

- skriveni dependency-i,
- teže testiranje,
- loša čitljivost.

---

# Context Value pravilo

Koristi za:

- request ID,
- trace ID,
- auth metadata,
- deadline informacije.

---

Ne koristi za:

- database konekcije,
- servise,
- konfiguraciju,
- poslovne objekte.

---

# Anti-Pattern #3

# Context u struct-u

Loš primer:

```go
type Service struct {

	ctx context.Context

}
```

---

Zašto je problem?

Context ima lifecycle.

Service obično živi duže.

---

Primer:

HTTP request:

```
5 sekundi
```

Service:

```
24 sata
```

---

Context može isteći dok service postoji.

---

Bolje:

```go
func (s *Service) Run(
	ctx context.Context,
)
```

---

Context treba da bude:

```
argument funkcije
```

a ne:

```
polje strukture
```

---

# Anti-Pattern #4

# Ignorisanje Context-a u pozvanoj funkciji

Primer:

```go
func Process(
	ctx context.Context,
){
	doWork()
}
```

---

Ali:

```go
doWork()
```

nema:

```go
ctx
```

---

Rezultat:

Cancel signal se gubi.

---

Bolje:

```go
func doWork(
	ctx context.Context,
)
```

---

I dalje:

```go
select {

case <-ctx.Done():
	return

default:
	work()

}
```

---

# Anti-Pattern #5

# Pogrešan timeout dizajn

Loše:

Svaki sloj pravi svoj timeout.

---

Primer:

HTTP:

```
30s
```

Service:

```
30s
```

Database:

```
30s
```

---

Ukupno:

```
90 sekundi
```

---

Problem:

Timeout nije koordinisan.

---

Bolje:

Jedan parent deadline:

```text
HTTP Request

30s

↓

Service

↓

Database
```

---

Svi koriste isti context.

---

# Anti-Pattern #6

# Korišćenje Background svuda

Primer:

```go
context.Background()
```

u svakoj funkciji.

---

Problem:

Gubi se:

- cancellation,
- deadline,
- tracing.

---

Loše:

```go
func Save(){

	ctx := context.Background()

	db.Query(ctx)

}
```

---

Bolje:

```go
func Save(
	ctx context.Context,
){

	db.Query(ctx)

}
```

---

# Anti-Pattern #7

# Pogrešna upotreba Context ključeva

Loše:

```go
ctx.Value("user")
```

---

Problem:

String collision.

---

Bolje:

poseban tip:

```go
type contextKey string

const userKey contextKey = "user"
```

---

# Anti-Pattern #8

# Ignorisanje ctx.Done()

Loš worker:

```go
for {

	process()

}
```

---

Ako parent otkaže:

worker nastavlja.

---

Bolje:

```go
for {

	select {

	case <-ctx.Done():
		return

	default:
		process()

	}

}
```

---

# Context i Goroutine lifecycle

Svaka dugotrajna Goroutine treba:

```text
Start

↓

Run

↓

Listen Context

↓

Stop
```

---

Primer:

```text
Application

↓

Worker

↓

Context

↓

Shutdown
```

---

# Production pravila

✅ Context je prvi argument funkcije.

Primer:

```go
func Handle(
	ctx context.Context,
	req Request,
)
```

---

✅ Ne čuvaj Context u struct.

---

✅ Uvek pozovi cancel.

---

✅ Propagiraj Context kroz call chain.

---

✅ Ne koristi Context kao dependency container.

---

# Mentalni model

Context nije:

```text
globalna konfiguracija
```

---

Context jeste:

```text
signal za životni ciklus operacije
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto je cancel važan

✅ zašto Context nije storage

✅ zašto Context ne treba u struct-u

✅ kako pravilno propagirati cancellation

✅ timeout hijerarhiju

✅ pravilnu upotrebu context vrednosti

---

# Concurrency Anti-Patterns — Race Condition problemi

---

# 📚 Sadržaj

- Šta je Race Condition
- Data Race
- Lost Update problem
- Deljenje memorije bez zaštite
- Pogrešna upotreba Atomic-a
- Race kroz map-e
- Race detector
- Završni pregled Anti-Patterns

---

# Uvod

Race Condition nastaje kada rezultat programa zavisi od:

```
redosleda izvršavanja Goroutines
```

---

Primer:

```go
counter++
```

Izgleda kao jedna operacija.

Ali zapravo:

```text
1. Read counter

2. Add 1

3. Write counter
```

---

Ako dve Goroutines rade istovremeno:

```text
Goroutine A

Read 0


Goroutine B

Read 0
```

---

Obe upisuju:

```
1
```

umesto:

```
2
```

---

# Data Race

Data Race postoji kada:

1. dve Goroutines pristupaju istom podatku,

2. bar jedna operacija je write,

3. nema sinhronizacije.

---

Primer:

```go
var total int


go func(){

	total++

}()

go func(){

	total++

}()
```

---

Problem:

Obe menjaju:

```go
total
```

---

# Anti-Pattern #1

# Deljena promenljiva bez zaštite

Loš primer:

```go
type Counter struct {

	value int

}
```

---

Više Goroutines:

```go
counter.value++
```

---

Nema:

- Mutex,
- Atomic,
- Channel.

---

Rezultat:

nepredvidivo stanje.

---

# Rešenje #1

Mutex:

```go
mu.Lock()

counter.value++

mu.Unlock()
```

---

# Rešenje #2

Atomic:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

# Anti-Pattern #2

# Race kroz map-e

Go map nije thread-safe.

---

Loše:

```go
var cache = map[string]string{}
```

---

Goroutine 1:

```go
cache["a"] = "value"
```

---

Goroutine 2:

```go
fmt.Println(cache["a"])
```

---

Rezultat:

```
fatal error:
concurrent map read and map write
```

---

# Rešenja

## Mutex

```go
mu.Lock()

cache[key] = value

mu.Unlock()
```

---

## sync.Map

Za specifične slučajeve:

```go
var cache sync.Map
```

---

Ali:

`sync.Map` nije zamena za običnu mapu.

---

# Anti-Pattern #3

# Check-then-act Race

Vrlo česta greška.

---

Loš primer:

```go
if !exists(key) {

	create(key)

}
```

---

Problem:

Dve Goroutines:

```text
G1:

proveri → nema


G2:

proveri → nema
```

---

Obe kreiraju.

---

Rešenje:

Atomizovati operaciju.

Primer:

```go
sync.Once
```

ili:

```go
Mutex
```

---

# Anti-Pattern #4

# Pogrešna upotreba Atomic-a

Atomic nije magični lock.

---

Dobro:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Loše:

```go
atomic.StoreInt64()

atomic.LoadInt64()

calculate()

atomic.StoreInt64()
```

---

Problem:

Cela logika nije atomarna.

---

Primer:

```text
Read

↓

Calculate

↓

Write
```

mora biti jedna zaštićena operacija.

---

# Anti-Pattern #5

# Race kroz slice

Slice nije automatski thread-safe.

---

Loše:

```go
items = append(items, value)
```

iz više Goroutines.

---

Problem:

Interno:

- menja length,
- može realocirati backing array.

---

Rešenje:

Mutex:

```go
mu.Lock()

items = append(items, value)

mu.Unlock()
```

---

Ili:

jedan owner Goroutine:

```text
Workers

↓

Channel

↓

Collector

↓

Slice
```

---

# Anti-Pattern #6

# Ignorisanje Race Detector-a

Go ima ugrađeni alat:

```bash
go test -race
```

---

Primer:

```bash
go test -race ./...
```

---

On detektuje:

- data race,
- problematične pristupe memoriji.

---

Primer izlaza:

```
WARNING:
DATA RACE
```

---

# Kada koristiti Race Detector?

Obavezno:

- CI pipeline,
- concurrency testovi,
- kritični servisi.

---

# Race Detector ograničenja

Ne nalazi:

- logičke race probleme,
- deadlock,
- pogrešan dizajn.

---

Primer:

```text
Nema data race-a

ali sistem pogrešno radi
```

---

# Završni pregled svih Anti-Patterns

---

# Goroutines

❌ Nekontrolisan broj

❌ Goroutine leak

❌ Fire-and-forget bez kontrole

---

# Channels

❌ Nema close

❌ Pogrešan owner

❌ Deadlock

❌ Ogromni buffer-i

---

# Mutex

❌ Preširoke kritične sekcije

❌ Deadlock

❌ Kopiranje lock-a

---

# WaitGroup

❌ Add posle starta

❌ Zaboravljen Done

❌ Korišćen kao result collector

---

# Context

❌ Zaboravljen cancel

❌ Globalni storage

❌ Nema propagacije

---

# Race Condition

❌ Deljeno stanje bez zaštite

❌ Pogrešan Atomic

❌ Concurrent map/slice pristup

---

# Senior Concurrency Checklist

Pre merge-a pitaj:

---

## Lifecycle

```
Kako se završava Goroutine?
```

---

## Ownership

```
Ko poseduje stanje?
```

---

## Backpressure

```
Šta kada producer ubrza?
```

---

## Cancellation

```
Kako se prekida rad?
```

---

## Synchronization

```
Da li je deljeno stanje zaštićeno?
```

---

## Testing

```
Da li je pokrenut -race?
```

---

# Finalni mentalni model

Loš concurrency:

```text
Više Goroutines

↓

Deljena memorija

↓

Problemi
```

---

Dobar concurrency:

```text
Jasan ownership

↓

Kontrolisana komunikacija

↓

Predvidiv lifecycle
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Race Condition

✅ razliku između Race Condition i Data Race

✅ probleme sa mapama i slice-ovima

✅ kada koristiti Mutex, Atomic ili Channel

✅ kako koristiti Race Detector

✅ kompletan pregled concurrency anti-pattern-a

---

# 🎯 Modul #4.9 završen

Prešli smo:

1. Goroutine Anti-Patterns  
2. Channel Anti-Patterns  
3. Mutex Anti-Patterns  
4. WaitGroup Anti-Patterns  
5. Context Anti-Patterns  
6. Race Condition Anti-Patterns  

---

### ➡️ Sledeća lekcija **[**Performance & Profiling**](10-performance-and-profiling.md)**

Obuhvatiće:

- CPU profiling,
- Memory profiling,
- Goroutine profiling,
- pprof,
- benchmark analiza,
- tracing,
- pronalaženje bottleneck-a.