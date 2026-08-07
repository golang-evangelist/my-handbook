# sync/atomic — Uvod u Atomic Operations

> **Modul:** #4 — Advanced Go Concurrency  
>
> **Fajl:** `docs/module-4/01-sync-atomic.md`

---

# 📚 Sadržaj

- Šta su atomic operacije
- Zašto postoje atomic operacije
- Problem sa race condition
- Mutex vs Atomic
- CPU nivo izvršavanja
- Kada koristiti atomic
- Kada ne koristiti atomic

---

# Uvod

U prethodnom modulu naučili smo:

```
više Goroutines

+

deljeno stanje

=

potreba za sinhronizacijom
```

---

Najčešće rešenje:

```go
sync.Mutex
```

---

Primer:

```go
var counter int

var mu sync.Mutex


func Increment(){

	mu.Lock()

	counter++

	mu.Unlock()

}
```

---

Ovo radi.

Ali postoji pitanje:

> Da li nam je za svaku malu operaciju potreban Mutex?

---

Odgovor:

Ne uvek.

---

# Šta je Atomic operacija?

Atomic znači:

```
nedeljiva operacija
```

---

Drugim rečima:

Operacija se izvršava kao jedna celina.

---

Nema mogućnosti da druga Goroutine vidi:

```
pola izvršene operacije
```

---

Primer:

```go
counter++
```

Izgleda kao jedna operacija.

Ali CPU vidi:

```
READ

+

ADD

+

WRITE
```

---

Zato nije atomic.

---

# Problem bez sinhronizacije

Kod:

```go
counter++
```

iz više Goroutines.

---

Početno:

```
counter = 0
```

---

Goroutine 1:

```
READ 0
```

---

Goroutine 2:

```
READ 0
```

---

Goroutine 1:

```
WRITE 1
```

---

Goroutine 2:

```
WRITE 1
```

---

Rezultat:

```
counter = 1
```

---

Očekivanje:

```
counter = 2
```

---

Ovo je:

```
Data Race
```

---

# Atomic rešava problem

Atomic operacija garantuje:

```
READ + MODIFY + WRITE

kao jedna nedeljiva operacija
```

---

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Ako dve Goroutines izvrše:

```
G1 Add +1

G2 Add +1
```

---

Rezultat:

```
counter = 2
```

---

# Gde se Atomic izvršava?

Atomic operacije se oslanjaju na:

```
CPU instrukcije
```

---

Moderni procesori imaju podršku za:

- atomic read,
- atomic write,
- compare-and-swap.

---

Primer koncepta:

```
CPU Core 1

LOCK instruction

↓

Memory update

↓

Other cores see new value
```

---

# Atomic u Go-u

Paket:

```go
sync/atomic
```

---

Primer:

```go
import "sync/atomic"
```

---

Najčešće operacije:

```
Load

Store

Add

Swap

Compare-And-Swap
```

---

# Atomic nije zamena za Mutex

Česta greška:

> "Atomic je brži, koristiću ga svuda."

---

Nije tačno.

---

Atomic je odličan za:

- counter,
- flag,
- simple state.

---

Mutex je bolji za:

- kompleksno stanje,
- više promenljivih,
- strukture,
- map/slice operacije.

---

Primer:

Ovo nije dobar kandidat za Atomic:

```go
type User struct {

	Name string

	Age int

	Active bool

}
```

---

Zašto?

Jer menjamo:

```
više povezanih vrednosti
```

---

Bolje:

```go
mu.Lock()

user.Name = "Marko"
user.Age = 30

mu.Unlock()
```

---

# Atomic primeri iz realnih sistema

## 1. Metrics counter

Primer:

```
broj request-a
```

---

```go
atomic.AddInt64(
	&requests,
	1,
)
```

---

---

## 2. Feature flag

Primer:

```go
var enabled int32
```

---

Čitanje:

```go
atomic.LoadInt32(&enabled)
```

---

---

## 3. Status state

Primer:

```
Server running

Server stopping
```

---

```go
atomic.StoreInt32(
	&state,
	Stopping,
)
```

---

# Atomic prednosti

## 1. Nema lock-a

Nema:

```go
Lock()

Unlock()
```

---

## 2. Manji overhead

Za male operacije:

```
Atomic

>

Mutex
```

---

## 3. Bolja skalabilnost

Kod velikog broja čitanja:

```
više Goroutines

+

jednostavan state
```

---

# Atomic ograničenja

---

## 1. Samo jednostavne operacije

Dobro:

```go
counter++
```

---

Loše:

```go
updateUser()
```

---

---

## 2. Teže za čitanje

Mutex:

```go
mu.Lock()
```

je očigledan.

---

Atomic logika često zahteva:

dublje razumevanje.

---

---

## 3. Kompleksnost algoritma

Lock-free kod može biti:

```
teži za održavanje
```

---

# Mentalni model

Zapamti:

```
Mutex

=

zaštiti kritičnu sekciju


Atomic

=

zaštiti jednu malu vrednost
```

---

# Senior pravilo

Ako imaš:

```
jedan broj

jedan flag

jedan state
```

razmisli o:

```
Atomic
```

---

Ako imaš:

```
objekat

strukturu

više povezanih promena
```

koristi:

```
Mutex
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su atomic operacije  
✅ zašto counter++ nije atomic  
✅ kako nastaje race condition  
✅ kako atomic rešava problem  
✅ razliku Atomic vs Mutex  
✅ kada koristiti atomic  
✅ ograničenja atomic pristupa  

---

# sync/atomic — Osnovni API

---

# 📚 Sadržaj

- Paket `sync/atomic`
- Atomic tipovi
- Alignment pravila
- Load operacije
- Store operacije
- Add operacije
- Swap operacije
- Pravilan način korišćenja

---

# 1. Paket sync/atomic

Go standardna biblioteka obezbeđuje atomic operacije kroz:

```go
import "sync/atomic"
```

---

Paket omogućava:

- sigurno čitanje vrednosti,
- sigurno menjanje vrednosti,
- sinhronizaciju bez klasičnih lock-ova.

---

Glavne kategorije:

```
Load

Store

Add

Swap

Compare-And-Swap
```

---

# 2. Atomic tipovi u Go-u

Go podržava atomic operacije nad:

- `int32`
- `int64`
- `uint32`
- `uint64`
- `uintptr`
- pointers

---

Primer:

```go
var counter int64
```

---

Atomic operacija:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

# 3. Pravilo pointer argumenta

Atomic funkcije rade nad adresom vrednosti.

---

Ne:

```go
atomic.AddInt64(
	counter,
	1,
)
```

---

Ispravno:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Zašto?

Atomic mora direktno da pristupi memorijskoj lokaciji.

---

Model:

```
Variable

↓

Memory Address

↓

CPU Atomic Instruction
```

---

# 4. Memory Alignment

Atomic operacije imaju zahtev:

> Vrednost mora biti pravilno poravnata u memoriji.

---

Posebno važno za:

```go
int64
```

---

Primer:

```go
type Counter struct {

	value int64

}
```

---

Go compiler uglavnom obezbeđuje pravilan alignment.

---

Ali problem može nastati kod:

- unsafe koda,
- ručne memorije,
- deljenja struktura između arhitektura.

---

# 5. Atomic Load

## Šta radi?

Čita vrednost na atomic način.

---

Običan read:

```go
value := counter
```

---

Atomic read:

```go
value := atomic.LoadInt64(
	&counter,
)
```

---

Garantuje:

```
dobijamo celu vrednost
```

---

Bez:

```
partial read
```

---

# Primer

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main(){

	var counter int64 = 100

	value := atomic.LoadInt64(
		&counter,
	)

	fmt.Println(value)

}
```

---

Rezultat:

```
100
```

---

# Kada koristiti Load?

Najčešće za:

- state flag,
- metrics,
- konfiguracione vrednosti.

---

Primer:

```go
if atomic.LoadInt32(&running) == 1 {

	start()

}
```

---

# 6. Atomic Store

## Šta radi?

Upisuje novu vrednost atomic načinom.

---

Običan write:

```go
running = 1
```

---

Atomic write:

```go
atomic.StoreInt32(
	&running,
	1,
)
```

---

# Primer

```go
var enabled int32


atomic.StoreInt32(
	&enabled,
	1,
)
```

---

Druga Goroutine:

```go
if atomic.LoadInt32(&enabled)==1 {

	fmt.Println("enabled")

}
```

---

Dobijamo:

```
sigurnu komunikaciju između Goroutines
```

---

# 7. Atomic Add

Najčešće korišćena operacija.

---

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Radi:

```
counter = counter + 1
```

---

Ali atomically.

---

Podržava:

pozitivne:

```go
+1
```

---

negativne:

```go
-1
```

---

Primer:

```go
atomic.AddInt64(
	&activeUsers,
	-1,
)
```

---

# Realni primer

Brojanje HTTP request-a:

```go
var requests int64


func handler(){

	atomic.AddInt64(
		&requests,
		1,
	)

}
```

---

Rezultat:

```
broj request-a
```

bez Mutex-a.

---

# 8. Atomic Swap

Swap:

```
upiši novu vrednost

vrati staru vrednost
```

---

Primer:

```go
old := atomic.SwapInt32(
	&state,
	2,
)
```

---

Pre:

```
state = 1
```

---

Posle:

```
state = 2
```

---

Povratna vrednost:

```
old = 1
```

---

# Kada koristiti Swap?

Primer:

promena statusa:

```
Starting

↓

Running

↓

Stopping
```

---

Kod:

```go
previous :=
atomic.SwapInt32(
	&state,
	Stopping,
)
```

---

# 9. Atomic API pregled

| Operacija | Namena |
|-|-|
| Load | atomic read |
| Store | atomic write |
| Add | increment/decrement |
| Swap | zamena vrednosti |
| CAS | uslovna promena |

---

# 10. Atomic nije samo "brži Mutex"

Česta zabluda:

```
Atomic = brži Mutex
```

---

Tačnije:

```
Atomic

=

drugačiji alat
```

---

Mutex rešava:

```
kritičnu sekciju
```

---

Atomic rešava:

```
jednu memorijsku vrednost
```

---

# 11. Primer poređenja

## Mutex

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Štiti:

```
ceo blok koda
```

---

## Atomic

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Štiti:

```
jednu operaciju
```

---

# 12. Pravila dobre prakse

## Pravilo #1

Ne mešati običan pristup i atomic pristup.

Loše:

```go
atomic.StoreInt64(&value,1)

value = 2
```

---

Zašto?

Jer uvodiš race.

---

# Pravilo #2

Sve pristupe istoj promenljivoj zaštititi istim mehanizmom.

---

Dobro:

```go
Load

Store

Add
```

svuda atomic.

---

# Pravilo #3

Atomic koristiti za male state vrednosti.

---

Dobri kandidati:

```go
counter

flag

status
```

---

Loši kandidati:

```go
map

slice

complex struct
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ `sync/atomic` paket  
✅ podržane tipove  
✅ memory alignment osnovu  
✅ Atomic Load  
✅ Atomic Store  
✅ Atomic Add  
✅ Atomic Swap  
✅ pravila bezbednog korišćenja  

---

# sync/atomic — Praktična upotreba i performanse

---

# 📚 Sadržaj

- Atomic counter primer
- Mutex counter primer
- Concurrent pristup
- Benchmark poređenje
- Cache line i contention
- Kada Atomic daje prednost

---

# 1. Atomic Counter primer

Najčešći primer za atomic je:

```
shared counter
```

---

Problem:

Više Goroutines povećava istu vrednost.

---

Bez zaštite:

```go
counter++
```

dobijamo:

```
data race
```

---

Rešenje:

```go
atomic.AddInt64()
```

---

# Primer

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {

	var counter int64

	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {

		wg.Add(1)

		go func(){

			defer wg.Done()

			for j := 0; j < 1000; j++ {

				atomic.AddInt64(
					&counter,
					1,
				)

			}

		}()

	}

	wg.Wait()

	fmt.Println(counter)

}
```

---

Rezultat:

```
100000
```

---

Zašto?

Imamo:

```
100 Goroutines

×

1000 increments

=

100000
```

---

# 2. Isti primer sa Mutex-om

Sada isto rešavamo pomoću:

```go
sync.Mutex
```

---

Primer:

```go
package main

import (
	"fmt"
	"sync"
)

func main(){

	var counter int64

	var mu sync.Mutex

	var wg sync.WaitGroup


	for i:=0;i<100;i++{

		wg.Add(1)

		go func(){

			defer wg.Done()


			for j:=0;j<1000;j++{

				mu.Lock()

				counter++

				mu.Unlock()

			}

		}()

	}


	wg.Wait()

	fmt.Println(counter)

}
```

---

Rezultat:

```
100000
```

---

Oba rešenja su:

```
thread-safe
```

---

Ali imaju različitu internu implementaciju.

---

# 3. Atomic vs Mutex — šta se dešava?

## Mutex

Tok:

```
Goroutine

↓

Lock

↓

Scheduler

↓

Critical Section

↓

Unlock
```

---

Ako je lock zauzet:

Goroutine može:

```
čekati
```

ili biti:

```
parkirana
```

---

# Atomic

Tok:

```
Goroutine

↓

CPU atomic instruction

↓

Memory update
```

---

Nema:

```
Lock

Unlock
```

---

# 4. Benchmark primer

Benchmark omogućava poređenje.

---

Struktura:

```
counter_benchmark_test.go
```

---

## Mutex benchmark

```go
func BenchmarkMutexCounter(
	b *testing.B,
){

	var counter int64

	var mu sync.Mutex


	b.RunParallel(
		func(pb *testing.PB){

			for pb.Next(){

				mu.Lock()

				counter++

				mu.Unlock()

			}

		},
	)

}
```

---

## Atomic benchmark

```go
func BenchmarkAtomicCounter(
	b *testing.B,
){

	var counter int64


	b.RunParallel(
		func(pb *testing.PB){

			for pb.Next(){

				atomic.AddInt64(
					&counter,
					1,
				)

			}

		},
	)

}
```

---

Pokretanje:

```bash
go test -bench .
```

---

# 5. Očekivani rezultat

U jednostavnom counter primeru:

Atomic često ima prednost.

Primer:

```
Mutex:

100 ns/op


Atomic:

10-20 ns/op
```

---

Napomena:

Rezultat zavisi od:

- CPU arhitekture,
- broja Goroutines,
- contention nivoa,
- Go verzije.

---

# 6. Zašto je Atomic brži?

Zato što koristi:

```
CPU atomic instructions
```

---

Primer koncepta:

```
LOCK XADD
```

na x86 arhitekturi.

---

CPU sam obezbeđuje:

- atomic read,
- modify,
- write.

---

Nema potrebe za:

```
OS scheduling
```

---

# 7. Ali Atomic nije uvek brži

Važno:

```
Atomic != uvek brži
```

---

Kod velike konkurencije:

```
10000 Goroutines

+

jedna atomic promenljiva
```

može nastati:

```
contention
```

---

Svi CPU core-ovi pokušavaju:

```
izmeniti istu memorijsku lokaciju
```

---

Rezultat:

```
cache line invalidation
```

---

# 8. Cache Line problem

CPU ne čita pojedinačne bajtove.

Radi sa:

```
cache line
```

obično:

```
64 bytes
```

---

Primer:

```go
type Counter struct {

	a int64

	b int64

}
```

---

Ako različiti CPU core-ovi menjaju:

```
a

b
```

može doći do:

```
false sharing
```

---

Jer su u istoj cache liniji.

---

# 9. False Sharing

Primer:

```
CPU 1

a++

↓

cache invalidation


CPU 2

b++

↓

cache invalidation
```

---

Iako:

```
a

i

b
```

nisu ista promenljiva.

---

Problem:

dele isti cache prostor.

---

Rešenje:

- padding,
- razdvajanje struktura,
- sharding.

---

# 10. Kada Atomic daje najveću prednost?

Odličan za:

---

## Metrics

Primer:

```go
requestCount++
```

---

## Flags

Primer:

```go
serverRunning
```

---

## Counters

Primer:

```go
activeConnections
```

---

## Simple state machine

Primer:

```
Starting

Running

Stopped
```

---

# 11. Kada Atomic nije dobar izbor?

Primer:

```go
type Account struct {

	balance int64

	transactions []Transaction

}
```

---

Operacija:

```go
Transfer()
```

zahteva:

```
provera

+

izmena

+

dodavanje transakcije
```

---

Atomic nije dovoljan.

---

Bolje:

```go
mu.Lock()

Transfer()

mu.Unlock()
```

---

# 12. Pravilo za izbor

Koristi Atomic kada imaš:

```
jedna vrednost

jedna operacija

jedan invariant
```

---

Koristi Mutex kada imaš:

```
više vrednosti

više operacija

kompleksan invariant
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Atomic counter implementaciju  
✅ Mutex counter implementaciju  
✅ razliku u izvršavanju  
✅ benchmark poređenje  
✅ CPU atomic instrukcije  
✅ contention problem  
✅ cache line i false sharing  
✅ kada Atomic daje prednost  

---

# sync/atomic — Compare-And-Swap (CAS)

---

# 📚 Sadržaj

- Šta je Compare-And-Swap
- Problem klasičnog read-modify-write pristupa
- Atomic CAS operacija
- Optimistic concurrency
- Implementacija atomic state machine
- Lock-free counter primer
- Prednosti i ograničenja CAS-a

---

# 1. Šta je Compare-And-Swap?

Compare-And-Swap (CAS) je jedna od najvažnijih atomic operacija.

Njena ideja:

> Promeni vrednost samo ako je ona i dalje ista kao što očekujemo.

---

CAS radi tri koraka:

```
1. Proveri trenutnu vrednost

2. Uporedi sa očekivanom vrednošću

3. Ako se poklapa, izvrši zamenu
```

---

Model:

```
if value == old {

	value = new

}
```

---

Ali kao jedna atomic CPU operacija.

---

# 2. Problem običnog pristupa

Pretpostavimo:

```go
value := 10
```

Želimo:

```
ako je value == 10

postavi value = 20
```

---

Naivan kod:

```go
if value == 10 {

	value = 20

}
```

---

Problem:

Između:

```go
if
```

i:

```go
assignment
```

druga Goroutine može promeniti vrednost.

---

Primer:

Početno:

```
value = 10
```

---

Goroutine 1:

```
proveri value == 10
```

---

Goroutine 2:

```
value = 30
```

---

Goroutine 1:

```
value = 20
```

---

Dobijamo:

```
izgubljenu promenu
```

---

Ovo je:

```
race condition
```

---

# 3. CAS rešava problem

CAS kombinuje:

```
check

+

update
```

u jednu operaciju.

---

Primer:

```go
atomic.CompareAndSwapInt64(
	&value,
	10,
	20,
)
```

---

Značenje:

```
Ako je value == 10

onda postavi value = 20
```

---

Povratna vrednost:

```go
true
```

ako je zamena uspešna.

---

Ako nije:

```go
false
```

---

# 4. CAS API

Primer:

```go
swapped :=
atomic.CompareAndSwapInt64(
	&value,
	old,
	new,
)
```

---

Parametri:

| Parametar | Značenje |
|-|-|
| `&value` | memorijska lokacija |
| `old` | očekivana vrednost |
| `new` | nova vrednost |

---

# 5. Jednostavan primer

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main(){

	var state int32 = 0


	success :=
	atomic.CompareAndSwapInt32(
		&state,
		0,
		1,
	)


	fmt.Println(success)

	fmt.Println(state)

}
```

---

Rezultat:

```
true
1
```

---

Prva Goroutine:

```
0 → 1
```

uspešno.

---

Druga:

```
0 → 1
```

neće uspeti jer:

```
state više nije 0
```

---

# 6. CAS kao lock-free mehanizam

Mutex:

```
čekaj lock

↓

izmeni

↓

otpusti lock
```

---

CAS:

```
pokušaj promenu

↓

ako neuspeh

↓

ponovi
```

---

Primer:

```
Try

↓

Fail?

↓

Retry

↓

Success
```

---

Ovaj pristup se naziva:

```
Optimistic Concurrency
```

---

# 7. Lock-free counter

Primer:

```go
func increment(
	value *int64,
){

	for {

		old :=
		atomic.LoadInt64(value)


		new :=
		old + 1


		if atomic.CompareAndSwapInt64(
			value,
			old,
			new,
		){

			return

		}

	}

}
```

---

Šta se dešava?

---

Korak 1:

Čitamo:

```
old value
```

---

Korak 2:

Računamo:

```
new value
```

---

Korak 3:

Pokušamo:

```
old → new
```

---

Ako druga Goroutine promeni vrednost:

CAS vraća:

```
false
```

---

Ponovo pokušavamo.

---

# 8. Zašto je ovo lock-free?

Nema:

```go
Lock()
```

niti:

```go
Unlock()
```

---

Nijedna Goroutine ne blokira drugu.

---

Ako jedna čeka:

druga može napredovati.

---

Definicija:

```
Lock-free algoritam garantuje
da sistem kao celina napreduje.
```

---

# 9. Primer: Atomic State Machine

Čest production slučaj:

status servisa.

---

Stanja:

```go
const (

	Stopped int32 = iota

	Running

	Stopping

)
```

---

Promena:

```
Stopped

↓

Running
```

---

Kod:

```go
ok :=
atomic.CompareAndSwapInt32(
	&state,
	Stopped,
	Running,
)
```

---

Ako je neko već pokrenuo servis:

druga Goroutine dobija:

```
false
```

---

Nema duplog startovanja.

---

# 10. CAS i ABA problem

Jedan od najpoznatijih problema.

---

Primer:

Početno:

```
A
```

---

Goroutine 1 pročita:

```
A
```

---

Goroutine 2 promeni:

```
A → B → A
```

---

Goroutine 1 vidi:

```
A
```

---

CAS uspe.

---

Ali stanje nije isto.

---

Problem:

```
ista vrednost

ne znači

isto stanje
```

---

Rešenja:

- version counter,
- tagged pointer,
- dodatni metadata.

---

# 11. Kada koristiti CAS?

Dobri slučajevi:

## State transition

```
Created → Running
```

---

## One-time initialization

```
uninitialized → initialized
```

---

## Lock-free structures

- stack,
- queue,
- counters.

---

# 12. Kada ne koristiti CAS?

CAS nije jednostavniji Mutex.

---

Loš izbor:

- kompleksne transakcije,
- veliki objekti,
- više povezanih promena.

---

Primer:

```go
bankTransfer()
```

---

Bolje:

```go
Mutex
```

---

# 13. CAS mentalni model

Zapamti:

```
Mutex:

"čekaj dok ne dobiješ pravo"


CAS:

"probaj, pa proveri da li si uspeo"
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Compare-And-Swap  
✅ kako CAS rešava race problem  
✅ optimistic concurrency  
✅ lock-free pristup  
✅ CAS retry loop  
✅ atomic state machine  
✅ ABA problem  

---

# sync/atomic — Atomic Patterns

---

# 📚 Sadržaj

- Atomic Counter Pattern
- Atomic Flag Pattern
- Atomic Configuration Pattern
- Reference Counting Pattern
- Atomic State Machine
- Production primeri

---

# Uvod

Do sada smo naučili:

```
Atomic API

↓

Load / Store / Add / Swap

↓

Compare-And-Swap
```

---

Sada prelazimo na pitanje:

> Kako se atomic operacije koriste u realnim sistemima?

---

Najčešći atomic pattern-i:

```
Counter

Flag

Configuration

Reference Counting

State Machine
```

---

# 1. Atomic Counter Pattern

Najčešći atomic slučaj.

---

Primer:

HTTP server prati broj zahteva.

---

Bezbedna verzija:

```go
type Metrics struct {

	requests int64

}
```

---

Povećavanje:

```go
atomic.AddInt64(
	&metrics.requests,
	1,
)
```

---

Čitanje:

```go
total :=
atomic.LoadInt64(
	&metrics.requests,
)
```

---

Rezultat:

više Goroutines može:

```
increment

+

read
```

bez Mutex-a.

---

# Realni primeri

Atomic counter se koristi za:

- broj HTTP request-a,
- broj grešaka,
- aktivne konekcije,
- statistiku sistema.

---

Primer:

```text
requests_total = 153920
errors_total   = 42
```

---

# 2. Atomic Flag Pattern

Flag predstavlja:

```
ON / OFF

TRUE / FALSE

START / STOP
```

---

Primer:

server status:

```go
var running int32
```

---

Start:

```go
atomic.StoreInt32(
	&running,
	1,
)
```

---

Stop:

```go
atomic.StoreInt32(
	&running,
	0,
)
```

---

Provera:

```go
if atomic.LoadInt32(&running) == 1 {

	process()

}
```

---

# Zašto ne bool?

Često pitanje:

Zašto:

```go
bool
```

ne koristimo?

---

Zato što:

```go
bool

+
concurrent access

=
race condition
```

---

Atomic podržava:

```
int32

int64

uint32

uint64
```

---

Zato koristimo:

```go
0 = false

1 = true
```

---

# 3. Atomic Configuration Pattern

Scenario:

Aplikacija ima konfiguraciju koja se često čita.

Primer:

```go
type Config struct {

	MaxConnections int

	Timeout int

}
```

---

Problem:

Mnogo reader-a.

Malo update-a.

---

Opcija 1:

Mutex:

```
RLock

Read

RUnlock
```

---

Opcija 2:

Atomic pointer.

---

Primer:

```go
var config atomic.Pointer[Config]
```

---

Postavljanje:

```go
config.Store(
	&Config{
		MaxConnections:100,
	},
)
```

---

Čitanje:

```go
current :=
config.Load()
```

---

Prednost:

Reader nema lock.

---

Model:

```
Readers

↓

Atomic Load

↓

Config snapshot
```

---

Koristi se za:

- runtime configuration,
- feature flags,
- dynamic settings.

---

# 4. Reference Counting Pattern

Reference counting znači:

pratimo:

```
koliko korisnika koristi objekat
```

---

Primer:

Resource:

```
Connection Pool

File Handle

Memory Object
```

---

Counter:

```go
var refs int64
```

---

Dodavanje reference:

```go
atomic.AddInt64(
	&refs,
	1,
)
```

---

Uklanjanje:

```go
if atomic.AddInt64(
	&refs,
	-1,
)==0 {

	closeResource()

}
```

---

Značenje:

Ako nema više korisnika:

```
refs == 0
```

onda:

```
cleanup
```

---

# Primer

Početak:

```
refs = 1
```

---

Nova Goroutine:

```
refs = 2
```

---

Jedna završava:

```
refs = 1
```

---

Poslednja završava:

```
refs = 0
```

---

Resource se oslobađa.

---

# 5. Atomic State Machine Pattern

Vrlo čest production pattern.

---

Problem:

Objekat ima životni ciklus.

---

Primer:

Server:

```
Created

↓

Starting

↓

Running

↓

Stopping

↓

Stopped
```

---

Definicija:

```go
const (

	Created int32 = iota

	Running

	Stopping

	Stopped

)
```

---

State:

```go
var state int32
```

---

Promena:

```go
atomic.CompareAndSwapInt32(
	&state,
	Created,
	Running,
)
```

---

Značenje:

Samo jedan caller može izvršiti:

```
Created → Running
```

---

# Zašto CAS?

Bez CAS:

dve Goroutines:

```
start()

start()
```

obe vide:

```
Created
```

---

Obe pokreću server.

---

Sa CAS:

Prva:

```
Created → Running

true
```

---

Druga:

```
Created → Running

false
```

---

# 6. Atomic Statistics Pattern

Često u backend servisima.

---

Primer:

```go
type Statistics struct {

	success int64

	failed int64

}
```

---

Success:

```go
atomic.AddInt64(
	&stats.success,
	1,
)
```

---

Failure:

```go
atomic.AddInt64(
	&stats.failed,
	1,
)
```

---

Metrics endpoint:

```go
success :=
atomic.LoadInt64(
	&stats.success,
)
```

---

# 7. Atomic + Mutex kombinacija

Nije uvek:

```
Atomic ili Mutex
```

---

Često se kombinuju.

---

Primer:

```go
type Cache struct {

	size int64

	mu sync.Mutex

	items map[string]string

}
```

---

Atomic:

```
size counter
```

---

Mutex:

```
map access
```

---

Pravilo:

Koristi najbolji alat za svaki deo.

---

# 8. Atomic Pattern Decision

| Problem | Rešenje |
|-|-|
| Brojač | Atomic Add |
| Flag | Atomic Load/Store |
| Status | CAS |
| Config snapshot | Atomic Pointer |
| Reference lifecycle | Atomic Counter |
| Kompleksno stanje | Mutex |

---

# 9. Senior pravilo

Nemoj pitati:

> "Kako da izbacim Mutex?"

---

Bolje pitanje:

> "Koji deo stanja zahteva atomic zaštitu?"

---

Cilj nije:

```
što manje lock-ova
```

---

Cilj je:

```
ispravna i skalabilna sinhronizacija
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Atomic Counter pattern  
✅ Atomic Flag pattern  
✅ Atomic Configuration pattern  
✅ Reference Counting  
✅ Atomic State Machine  
✅ Atomic + Mutex kombinaciju  
✅ izbor pravog pattern-a  

---

# sync/atomic — Atomic vs Mutex vs Channels

---

# 📚 Sadržaj

- Tri različita pristupa sinhronizaciji
- Atomic vs Mutex
- Mutex vs Channels
- Atomic vs Channels
- Decision tree
- Production primeri
- Najčešće greške
- Best practices

---

# Uvod

Go nudi više načina za sinhronizaciju:

- `sync/atomic`
- `sync.Mutex`
- `channels`

Početnici često pitaju:

> "Koji je najbolji?"

Tačnije pitanje je:

> "Koji je najprikladniji za konkretan problem?"

Svaki od ovih mehanizama rešava drugačiju vrstu problema.

---

# 1. Atomic

Atomic štiti:

```
jednu vrednost
```

Primeri:

- brojač
- status
- flag
- pointer
- jednostavan state

Primer:

```go
atomic.AddInt64(&requests, 1)
```

---

# 2. Mutex

Mutex štiti:

```
kritičnu sekciju
```

odnosno skup operacija koje moraju biti izvršene kao jedna celina.

Primer:

```go
mu.Lock()

account.Balance -= amount
account.History = append(account.History, tx)

mu.Unlock()
```

Ovde nije problem samo jedna promenljiva, već održavanje konzistentnog stanja.

---

# 3. Channels

Channel nije prvenstveno alat za zaštitu memorije.

Njegova glavna uloga je:

```
komunikacija između Goroutines
```

Primer:

```go
jobs <- job
```

Ovde jedna Goroutine predaje posao drugoj.

---

# Go filozofija

Poznata preporuka iz Go dokumentacije glasi:

> "Do not communicate by sharing memory; instead, share memory by communicating."

To znači:

- kada je moguće, organizuj program tako da jedna Goroutine poseduje određeno stanje,
- ostale Goroutine komuniciraju sa njom putem channel-a.

Time se često eliminiše potreba za eksplicitnim lock-ovima.

---

# Atomic vs Mutex

## Atomic

Prednosti:

- veoma mali overhead
- nema `Lock()` / `Unlock()`
- odličan za jednostavne vrednosti

Nedostaci:

- radi samo nad malim skupom operacija
- algoritmi mogu postati komplikovani
- nije pogodan za kompleksno stanje

---

## Mutex

Prednosti:

- jednostavan za razumevanje
- štiti više promenljivih odjednom
- lakše održavanje

Nedostaci:

- može izazvati contention
- Goroutine može biti blokirana
- nepravilna upotreba može dovesti do deadlock-a

---

# Atomic vs Channels

Atomic:

```
shared state

↓

atomic update
```

Channels:

```
owner goroutine

↓

message passing
```

Ako više Goroutines deli jednu promenljivu:

Atomic može biti dobro rešenje.

Ako želiš da jedna Goroutine bude vlasnik podataka:

Channel je često bolji izbor.

---

# Mutex vs Channels

Česta dilema.

Primer:

```go
map[string]User
```

Imamo dve mogućnosti.

## Pristup 1

Jedna Goroutine poseduje mapu.

Sve izmene stižu preko channel-a.

Prednosti:

- nema race condition
- nema Mutex-a

Mana:

- vlasnička Goroutine može postati usko grlo.

---

## Pristup 2

Mapa je deljena.

Pristup je zaštićen:

```go
sync.RWMutex
```

Prednosti:

- jednostavnija implementacija
- dobro radi za mnogo čitanja

Mana:

- postoji contention pri pisanju.

---

# Decision Tree

```
Da li postoji deljeno stanje?

        │
        ▼

        DA
        │
        ▼

Da li je samo jedna vrednost?

        │
   ┌────┴────┐
   │         │
  DA        NE
   │         │
   ▼         ▼

Atomic    Mutex

```

---

Ako odgovor glasi:

```
Ne želim deljeno stanje.
```

onda razmisli o:

```
Channels
```

---

# Production primeri

## Atomic

- metrics
- telemetry
- counters
- feature flags
- state flags

---

## Mutex

- cache
- session storage
- LRU cache
- map
- slice
- kompleksni objekti

---

## Channels

- worker pool
- pipeline
- producer-consumer
- event processing
- actor-like modeli

---

# Kombinovanje mehanizama

Production sistemi retko koriste samo jedan pristup.

Primer:

```go
type Server struct {
	mu       sync.RWMutex
	clients  map[string]Client
	requests int64
	jobs      chan Job
}
```

Ovde imamo:

```
requests

↓

Atomic

----------------

clients

↓

RWMutex

----------------

jobs

↓

Channel
```

Svaki alat rešava drugačiji problem.

---

# Najčešće greške

## Greška 1

Koristiti Atomic za kompleksne strukture.

---

## Greška 2

Koristiti Mutex za običan brojač.

---

## Greška 3

Koristiti Channel samo zato što je "Go idiom".

Nekada je jednostavan `Mutex` mnogo jasnije rešenje.

---

## Greška 4

Mešati običan pristup i atomic pristup istoj promenljivoj.

Primer:

```go
atomic.StoreInt64(&counter, 1)

counter++
```

Ovo može dovesti do race condition-a.

---

# Pravila izbora

| Problem | Najčešći izbor |
|----------|----------------|
| Counter | Atomic |
| Flag | Atomic |
| Status | Atomic (CAS) |
| Mapa | Mutex / RWMutex |
| Slice | Mutex |
| Više povezanih promenljivih | Mutex |
| Worker Pool | Channels |
| Pipeline | Channels |
| Event processing | Channels |

---

# Best Practices

✅ Koristi `atomic` za jednostavne numeričke vrednosti.

✅ Koristi `Mutex` kada moraš zaštititi konzistentnost više povezanih podataka.

✅ Koristi `channels` kada Goroutines treba da razmenjuju informacije ili prenesu vlasništvo nad podacima.

✅ Ne biraj alat po performansama pre nego što razumeš problem koji rešavaš.

---

# 📋 Rezime Modula #4.1

U okviru lekcije **`01-sync-atomic.md`** naučili smo:

- ✅ šta su atomic operacije
- ✅ `Load`, `Store`, `Add`, `Swap`
- ✅ `Compare-And-Swap (CAS)`
- ✅ optimistic concurrency
- ✅ lock-free osnove
- ✅ najčešće atomic pattern-e
- ✅ razliku između `atomic`, `Mutex` i `channels`
- ✅ kako odabrati odgovarajući mehanizam sinhronizacije

Ovo predstavlja osnovu za razumevanje naprednih tema kao što su:

- Go Memory Model
- Happens-Before relacije
- Lock-Free algoritmi
- Internali Go runtime-a

---

### ➡️ Sledeća lekcija **[**Atomic Operations Patterns**](02-atomic-operations-patterns.md)**

Obuhvatiće:

- napredni obrasci korišćenja `sync/atomic`,
- implementacija lock-free struktura,
- kombinovanje više atomic operacija,
- praktični production primeri,
- najčešće greške i ograničenja atomic pristupa.
