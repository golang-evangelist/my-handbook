# Data Races Deep Dive

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Data Races Deep Dive
>
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Uvod u data race problem
- Race Condition vs Data Race
- Shared Memory model
- Zašto data race nastaje
- Primer jednostavnog race bug-a
- Instruction interleaving
- Compiler optimizations
- CPU reordering
- Go memory model i data races
- Osnovna pravila zaštite

---

# 1. Uvod u Data Race Problem

Concurrency programi često dele podatke između goroutines.

Primer:

```go
var counter int
```

---

Dve goroutines:

```go
go increment()

go increment()
```

---

Naizgled:

```
counter++

counter++
```

trebalo bi dati:

```
2
```

---

Ali u concurrent svetu:

rezultat nije garantovan.

---

Problem:

```
više izvršilaca pristupa istoj memoriji
```

---

# 2. Šta je Data Race?

Data race postoji kada:

imamo dve ili više goroutines koje:

1. pristupaju istoj memorijskoj lokaciji

2. najmanje jedna operacija je write

3. pristupi nisu pravilno sinhronizovani

---

Formalno:

```
Concurrent Access

+

Same Memory Location

+

At Least One Write

+

No Synchronization
```

=

```
DATA RACE
```

---

# 3. Race Condition vs Data Race

Ova dva termina se često mešaju.

---

## Data Race

Tehnički problem:

```
nezaštićen pristup memoriji
```

---

Primer:

```go
counter++
```

iz više goroutines.

---

## Race Condition

Širi koncept:

rezultat zavisi od redosleda izvršavanja.

---

Primer:

```go
if balance > 0 {

	withdraw()

}
```

---

Problem:

Druga goroutine može promeniti:

```
balance
```

između:

```
check

↓

use
```

---

# 4. Shared Memory Model

Go koristi:

```
goroutines

+

shared memory

+

synchronization primitives
```

---

Deljena memorija:

```go
var state State
```

---

Dve goroutines:

```
G1

 |

state


G2

 |
state
```

---

Bez komunikacije:

problem.

---

# 5. Najjednostavniji Race Primer

Kod:

```go
package main

import (
	"fmt"
	"sync"
)


var counter int


func increment(){

	counter++

}


func main(){

	var wg sync.WaitGroup


	for i:=0;i<1000;i++{

		wg.Add(1)

		go func(){

			defer wg.Done()

			increment()

		}()

	}


	wg.Wait()


	fmt.Println(counter)

}
```

---

Očekivanje:

```
1000
```

---

Mogući rezultat:

```
997
```

ili:

```
942
```

---

Zašto?

Jer:

```go
counter++
```

nije atomic.

---

# 6. Šta se Dešava Interno?

Linija:

```go
counter++
```

nije jedna CPU instrukcija.

---

Može izgledati ovako:

```
READ counter

ADD 1

WRITE counter
```

---

Dve goroutines:

```
G1                 G2


READ 0            


                   READ 0


ADD 1


                   ADD 1


WRITE 1


                   WRITE 1
```

---

Final:

```
1
```

umesto:

```
2
```

---

# 7. Instruction Interleaving

Scheduler može prebaciti izvršavanje u bilo kom trenutku.

---

Primer:

G1:

```
READ counter
```

---

Scheduler:

```
switch
```

---

G2:

```
READ counter
```

---

Scheduler:

```
switch
```

---

G1:

```
WRITE counter
```

---

G2:

```
WRITE counter
```

---

Redosled određuje rezultat.

---

# 8. Zašto Compiler Dodatno Komplikuje?

Programer vidi:

```go
x = x + 1
```

---

Ali compiler može:

- promeniti redosled instrukcija
- držati vrednost u registru
- optimizovati pristup memoriji

---

Primer:

```go
for running {

}
```

---

Programer očekuje:

svaki put čitaj:

```go
running
```

---

Compiler može optimizovati:

```
učitaj jednom
```

---

Zato bez synchronization nema garancije.

---

# 9. CPU Reordering

Moderni CPU nije jednostavan interpreter.

---

CPU može izvršavati:

```
out of order
```

---

Primer:

Kod:

```
A = 1

B = 2
```

---

CPU može interno izvršiti:

```
B = 2

A = 1
```

---

Ako nema memory barriers:

druga goroutine može videti neočekivan redosled.

---

# 10. Go Memory Model i Data Races

Go memory model definiše:

kada jedna goroutine garantovano vidi promene druge.

---

Bez:

- channel komunikacije
- mutex-a
- atomic operacija

nema garantovane vidljivosti.

---

Primer:

```go
data = 42
ready = true
```

---

Druga goroutine:

```go
if ready {

	fmt.Println(data)

}
```

---

Bez synchronization:

nije garantovano da vidi:

```
data == 42
```

---

# 11. Data Race Nije Uvek Očigledan

Primer:

```go
type Config struct {

	Timeout int

}
```

---

Jedna goroutine:

```go
config.Timeout = 10
```

---

Druga:

```go
fmt.Println(
	config.Timeout,
)
```

---

Izgleda bezazleno.

Ali:

```
read + write
```

bez synchronization:

=

```
data race
```

---

# 12. Zaštita Od Data Race-a

Glavni alati:

---

## Mutex

```go
mu.Lock()

value++

mu.Unlock()
```

---

## Atomic

```go
atomic.AddInt64(
	&value,
	1,
)
```

---

## Channels

```go
ch <- value
```

---

## Immutable Data

```go
newConfig := oldConfig
```

---

# 13. Osnovno Pravilo

Ako više goroutines pristupa istoj promenljivoj:

pitaj:

```
Ko poseduje podatak?
```

---

Ako odgovor nije:

```
jedna goroutine
```

potrebna je:

```
sinhronizacija
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je data race

✅ razliku race condition i data race

✅ shared memory problem

✅ zašto counter++ nije bezbedan

✅ instruction interleaving

✅ compiler optimizations

✅ CPU reordering

✅ Go memory model veze

---

# Data Races Deep Dive

## Deo #2 — Go Race Detector Deep Dive

---

# 📚 Sadržaj

- Uvod u Go Race Detector
- Zašto je race detector potreban
- `go test -race`
- `go run -race`
- `go build -race`
- Kako race detector radi
- ThreadSanitizer osnova
- Race report analiza
- Reading stack traces
- Ograničenja race detector-a
- Production workflow

---

# 1. Uvod u Go Race Detector

Go ima ugrađeni alat za pronalaženje data race problema:

```bash
-race
```

---

Primer:

```bash
go test -race ./...
```

---

Race detector analizira program tokom izvršavanja.

---

Ne proverava samo kod.

On prati:

```
memory access

+

goroutine scheduling

+

synchronization events
```

---

# 2. Zašto Je Race Detector Potreban?

Data race problemi su specifični:

- često se ne reprodukuju
- zavise od timing-a
- pojavljuju se samo pod opterećenjem

---

Primer:

```go
counter++
```

---

Može raditi:

```
1000 puta
```

bez greške.

---

Ali:

jedan drugačiji scheduling:

```
BUG
```

---

Manual debugging je veoma težak.

---

# 3. Osnovne Race Komande

## Testiranje

```bash
go test -race
```

---

Sa svim paketima:

```bash
go test -race ./...
```

---

## Pokretanje aplikacije

```bash
go run -race main.go
```

---

## Build

```bash
go build -race
```

---

## Benchmark

```bash
go test -race -bench=.
```

---

# 4. Prvi Race Detector Primer

Kod:

```go
package main


import (
	"fmt"
	"sync"
)


var counter int


func main(){

	var wg sync.WaitGroup


	for i:=0;i<10;i++{

		wg.Add(1)


		go func(){

			defer wg.Done()

			counter++

		}()

	}


	wg.Wait()


	fmt.Println(counter)

}
```

---

Pokretanje:

```bash
go run -race main.go
```

---

Rezultat:

```
WARNING: DATA RACE
```

---

# 5. Kako Race Detector Radi?

Go race detector je baziran na:

```
ThreadSanitizer (TSan)
```

---

TSan je dinamički analyzer.

---

Tokom izvršavanja prati:

- reads
- writes
- synchronization

---

Primer:

Goroutine 1:

```
WRITE x
```

---

Goroutine 2:

```
READ x
```

---

Ako nema:

```
happens-before relationship
```

prijavljuje race.

---

# 6. Dynamic Analysis

Race detector nije static analyzer.

---

Ne radi:

```
čitanje source koda
```

i zaključivanje.

---

Radi:

```
pokreni program

↓

posmatraj izvršavanje

↓

detektuj konflikt
```

---

Zato:

mora se izvršiti problematičan kod.

---

# 7. Primer Race Report-a

Tipičan izlaz:

```
==================
WARNING: DATA RACE


Write at 0x00c0000120f8

goroutine 7:

main.increment()


Previous read at 0x00c0000120f8

goroutine 8:

main.increment()


==================
```

---

Važni delovi:

1. Access type

```
Read / Write
```

---

2. Memory address

```
0x00c0000120f8
```

---

3. Goroutine stack

```
goroutine 7
```

---

# 8. Kako Čitati Race Report

Primer:

```
Write at ...
```

znači:

neka goroutine menja vrednost.

---

Primer:

```
Previous read at ...
```

znači:

druga goroutine čita istu memoriju.

---

Traži:

```
gde je prvi shared state
```

---

Ne fokusiraj se na:

```
gde je panic
```

---

Data race često nema panic.

---

# 9. Synchronization Events

Race detector zna za:

## Mutex

```go
mu.Lock()

mu.Unlock()
```

---

## Channels

```go
ch <- value

<-ch
```

---

## WaitGroup

```go
wg.Done()

wg.Wait()
```

---

## Atomic

```go
atomic.AddInt64()
```

---

Oni kreiraju:

```
happens-before relationship
```

---

# 10. Primer Popravke Race-a

Pre:

```go
var counter int


counter++
```

---

Posle:

```go
var counter int

var mu sync.Mutex


mu.Lock()

counter++

mu.Unlock()
```

---

Race detector:

pre:

```
WARNING: DATA RACE
```

---

posle:

```
clean
```

---

# 11. Race Detector i Atomics

Primer:

```go
var counter atomic.Int64


counter.Add(1)
```

---

Race detector razume:

```
atomic synchronization
```

---

Nema prijave.

---

Ali:

pogrešna kombinacija:

```go
counter.Add(1)

fmt.Println(counterValue)
```

gde `counterValue` nije atomic:

može biti race.

---

# 12. Ograničenja Race Detector-a

Race detector nije magični alat.

---

## 1. Kod mora biti izvršen

Ako bug putanja nije pokrenuta:

neće biti pronađena.

---

## 2. Runtime overhead

Program može biti:

```
5x - 10x sporiji
```

---

## 3. Više memorije

Race instrumentation zahteva dodatnu memoriju.

---

# 13. Race Detector u CI Pipeline-u

Preporučeni workflow:

---

Lokalno:

```bash
go test -race ./...
```

---

CI:

```bash
go test -race -cover ./...
```

---

Nightly:

```bash
go test -race ./...
```

sa:

- integration testovima
- load testovima

---

# 14. Race Detector i Production

Ne koristi se obično u produkciji.

Razlog:

- performance overhead
- memory overhead

---

Koristi se u:

- development
- CI
- staging

---

# 15. Testovi Moraju Biti Concurrent

Loš test:

```go
func TestCounter(t *testing.T){

	counter++

}
```

---

Nema konkurencije.

---

Bolje:

```go
func TestCounter(t *testing.T){

	for i:=0;i<100;i++{

		go increment()

	}

}
```

---

Sada race detector može videti problem.

---

# 16. Race Detector Kao Design Feedback

Ako race detector često prijavljuje probleme:

možda problem nije:

```
nedostaje mutex
```

---

Možda je problem:

```
loš ownership model
```

---

Pitanje:

```
Ko je vlasnik ovog podatka?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Go race detector

✅ `go test -race`

✅ `go run -race`

✅ ThreadSanitizer koncept

✅ dynamic analysis

✅ race report čitanje

✅ synchronization awareness

✅ CI upotrebu

✅ ograničenja alata

---

# Data Races Deep Dive

## Deo #3 — Happens-Before Analiza Data Race Problema

---

# 📚 Sadržaj

- Šta je Happens-Before odnos
- Go Memory Model
- Sequenced Before vs Happens Before
- Synchronization Edges
- Mutex Happens-Before
- Channel Happens-Before
- WaitGroup Happens-Before
- Atomic Visibility
- Race-Free dokazivanje
- Praktična analiza

---

# 1. Šta je Happens-Before?

`Happens-Before` je koncept koji definiše:

> Da li jedna memorijska operacija garantovano prethodi drugoj u smislu vidljivosti.

---

Jednostavno:

Ako:

```
A happens-before B
```

onda:

```
B garantovano vidi efekte A
```

---

Primer:

```go
x = 10

unlock()
```

---

druga goroutine:

```go
lock()

read x
```

---

Garantija:

```
read x == 10
```

---

# 2. Zašto je Happens-Before Važan?

Kod konkurentnog programa nije dovoljno pitati:

```
šta se izvršilo prvo?
```

---

Pravo pitanje:

```
da li postoji garantovana veza između operacija?
```

---

Primer:

```
G1

write data


G2

read data
```

---

Bez veze:

```
nema garancije
```

---

Sa synchronization:

```
postoji happens-before
```

---

# 3. Go Memory Model

Go Memory Model definiše:

kada jedna goroutine može videti promene druge goroutine.

---

Osnovna pravila:

```
Program Order

+

Synchronization

=

Visibility Guarantee
```

---

Bez synchronization:

```
write

???

read
```

---

Sa synchronization:

```
write

↓

synchronization

↓

read
```

---

# 4. Sequenced Before

Prvi nivo:

jedna goroutine.

---

Primer:

```go
x = 1

y = 2
```

---

U istoj goroutine:

```
x = 1

happens before

y = 2
```

---

To je:

```
sequenced-before
```

---

# 5. Happens-Before Između Goroutines

Problem nastaje kada imamo:

```
G1

write


G2

read
```

---

Potrebna je:

```
synchronization edge
```

---

Primeri:

- mutex
- channel
- atomic
- WaitGroup

---

# 6. Mutex Happens-Before Pravilo

Go garantuje:

Unlock:

```
happens-before
```

sledeći Lock.

---

Primer:

Goroutine 1:

```go
mu.Lock()

data = 42

mu.Unlock()
```

---

Goroutine 2:

```go
mu.Lock()

fmt.Println(data)

mu.Unlock()
```

---

Veza:

```
Unlock

↓

Lock

↓

Read
```

---

Rezultat:

```
data == 42
```

---

# 7. Channel Happens-Before Pravilo

Channel komunikacija takođe kreira happens-before.

---

Primer:

Sender:

```go
data = 100

ch <- true
```

---

Receiver:

```go
<-ch

fmt.Println(data)
```

---

Veza:

```
write data

↓

send

↓

receive

↓

read data
```

---

Garantovana vidljivost.

---

# 8. Unbuffered Channel

Kod:

```go
ch := make(chan bool)
```

---

Send:

```go
ch <- true
```

čeka receiver.

---

Odnos:

```
send

↓

receive
```

---

Jak synchronization point.

---

# 9. Buffered Channel

Primer:

```go
ch :=
make(chan int, 10)
```

---

Send:

```go
ch <- value
```

može završiti bez receiver-a.

---

Ali:

Go Memory Model definiše:

```
n-ti send

happens-before

n-ti receive
```

---

Važno kod dizajna.

---

# 10. WaitGroup Happens-Before

Primer:

```go
go worker()

wg.Wait()
```

---

Worker:

```go
result = 42

wg.Done()
```

---

Main:

```go
wg.Wait()

fmt.Println(result)
```

---

Veza:

```
Done

↓

Wait returns

↓

Read result
```

---

Bez ove veze:

data race.

---

# 11. Atomic Visibility

Atomic operacije nisu samo:

```
bezbedan increment
```

---

One daju:

```
memory synchronization
```

---

Primer:

```go
atomic.StoreInt64(
	&ready,
	1,
)
```

---

Druga goroutine:

```go
if atomic.LoadInt64(
	&ready,
)==1 {

	use(data)

}
```

---

Atomic operacije kreiraju vidljivost.

---

# 12. Primer Bez Happens-Before

Kod:

```go
var data int

var ready bool


func writer(){

	data = 42

	ready = true

}


func reader(){

	if ready {

		fmt.Println(data)

	}

}
```

---

Problem:

Nema:

```
synchronization
```

---

Moguće:

```
ready == true

data == 0
```

---

Zašto?

Jer:

```
visibility nije garantovana
```

---

# 13. Popravka sa Channel-om

```go
var data int


func writer(
	ch chan struct{},
){

	data = 42

	ch <- struct{}{}

}


func reader(
	ch chan struct{},
){

	<-ch

	fmt.Println(data)

}
```

---

Sada:

```
write

↓

send

↓

receive

↓

read
```

---

Race je uklonjen.

---

# 14. Dokazivanje Race-Free Koda

Senior pristup:

Ne pitamo:

```
da li radi?
```

---

Pitamo:

```
koja synchronization edge postoji?
```

---

Za svaki shared state:

definiši:

```
Owner

+

Access rule

+

Synchronization mechanism
```

---

# 15. Ownership Model

Primer:

Loše:

```
10 goroutines

↓

shared map
```

---

Bolje:

```
worker goroutine

↓

owns map
```

---

Ostali:

```
communicate preko channel-a
```

---

# 16. Happens-Before Mentalni Model

Za svaki podatak:

Pitaj:

```
Ko ga piše?
```

---

Zatim:

```
Ko ga čita?
```

---

Zatim:

```
Koja veza garantuje redosled?
```

---

Ako nema odgovora:

verovatno postoji race.

---

# 17. Senior Checklist

Za shared state:

✅ postoji owner

✅ postoji synchronization

✅ postoji happens-before edge

✅ race detector prolazi

✅ dizajn je jednostavan

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Happens-Before koncept

✅ Go Memory Model

✅ sequenced-before

✅ synchronization edges

✅ mutex visibility

✅ channel visibility

✅ WaitGroup ordering

✅ atomic visibility

✅ dokazivanje race-free koda

---

# Data Races Deep Dive

## Deo #4 — Realni Data Race Bugovi u Go Aplikacijama

---

# 📚 Sadržaj

- Data race u mapama
- Data race u structovima
- Configuration race
- Lazy initialization problem
- Cache race
- Slice race
- Global state race
- Production incident scenariji
- Kako dizajnirati race-free kod

---

# 1. Data Race u Go Mapama

Go mape nisu thread-safe.

---

Primer:

```go
var users = make(map[string]int)
```

---

Goroutine 1:

```go
users["marko"] = 100
```

---

Goroutine 2:

```go
fmt.Println(
	users["marko"],
)
```

---

Problem:

```
write

+

read

+

no synchronization
```

=

```
DATA RACE
```

---

# 2. Concurrent Map Write Problem

Još ozbiljniji slučaj:

```go
go func(){

	users["a"] = 1

}()


go func(){

	users["b"] = 2

}()
```

---

Rezultat:

mogući runtime panic:

```
fatal error:
concurrent map writes
```

---

Go runtime eksplicitno detektuje neke slučajeve.

---

# 3. Rešenje: Mutex Za Mapu

Primer:

```go
type SafeMap struct {

	mu sync.Mutex

	data map[string]int

}
```

---

Write:

```go
func (s *SafeMap) Set(
	key string,
	value int,
){

	s.mu.Lock()

	defer s.mu.Unlock()


	s.data[key] = value

}
```

---

Read:

```go
func (s *SafeMap) Get(
	key string,
) int {

	s.mu.Lock()

	defer s.mu.Unlock()


	return s.data[key]

}
```

---

# 4. RWMutex Za Read-Heavy Map

Ako imamo:

```
mnogo čitanja

malo pisanja
```

---

Koristi:

```go
sync.RWMutex
```

---

Read:

```go
mu.RLock()

value :=
data[key]

mu.RUnlock()
```

---

Write:

```go
mu.Lock()

data[key] = value

mu.Unlock()
```

---

# 5. sync.Map

Go ima:

```go
sync.Map
```

---

Primer:

```go
var cache sync.Map
```

---

Store:

```go
cache.Store(
	"user",
	"Marko",
)
```

---

Load:

```go
value, ok :=
cache.Load(
	"user",
)
```

---

Koristi kada:

✅ mnogo read operacija

✅ ključevi se retko menjaju

---

Ne koristi automatski svuda.

---

# 6. Data Race u Structovima

Primer:

```go
type User struct {

	Name string

	Age int

}
```

---

Global:

```go
var user User
```

---

Goroutine 1:

```go
user.Age = 30
```

---

Goroutine 2:

```go
fmt.Println(
	user.Age,
)
```

---

Problem:

```
write + read
```

---

# 7. Partial Struct Update Problem

Česta greška:

```go
type Config struct {

	Host string

	Port int

}
```

---

Update:

```go
config.Host = "localhost"

config.Port = 8080
```

---

Druga goroutine:

```go
startServer(config)
```

---

Može videti:

```
Host = localhost

Port = 0
```

---

State je nevalidan.

---

# 8. Atomic Pointer Configuration Pattern

Bolje:

```go
type Config struct {

	Host string

	Port int

}
```

---

Storage:

```go
var config atomic.Pointer[Config]
```

---

Update:

```go
config.Store(
	&Config{
		Host:"localhost",
		Port:8080,
	},
)
```

---

Read:

```go
cfg :=
config.Load()
```

---

Čitav snapshot je konzistentan.

---

# 9. Lazy Initialization Race

Primer:

```go
var instance *Database


func GetDB() *Database {

	if instance == nil {

		instance =
		newDatabase()

	}

	return instance

}
```

---

Problem:

Dve goroutines:

```
G1:
instance == nil


G2:
instance == nil
```

---

Obe kreiraju:

```
Database
```

---

# 10. Rešenje: sync.Once

```go
var once sync.Once

var instance *Database


func GetDB() *Database {

	once.Do(
		func(){

			instance =
			newDatabase()

		},
	)


	return instance

}
```

---

Garantuje:

```
exactly once initialization
```

---

# 11. Cache Race

Primer:

```go
type Cache struct {

	items map[string]string

}
```

---

Read:

```go
cache.items[key]
```

---

Update:

```go
cache.items[key] = value
```

---

Problem:

read/write race.

---

# 12. Copy-On-Write Cache Pattern

Za read-heavy cache:

```go
type Cache struct {

	items map[string]string

}
```

---

Update:

1. napravi kopiju

2. promeni kopiju

3. atomic swap

---

Primer:

```go
newCache := clone(old)

newCache[key] = value

current.Store(
	newCache,
)
```

---

Prednost:

read nema lock.

---

# 13. Slice Data Race

Slice:

```go
var values []int
```

---

Goroutine 1:

```go
values =
append(
	values,
	1,
)
```

---

Goroutine 2:

```go
len(values)
```

---

Problem:

slice header se menja.

---

Slice sadrži:

```
pointer

length

capacity
```

---

Nije atomic struktura.

---

# 14. Global Variable Race

Primer:

```go
var debug bool
```

---

Request handler:

```go
if debug {

	logDetails()

}
```

---

Admin endpoint:

```go
debug = true
```

---

Problem:

runtime konfiguracija bez synchronization.

---

Rešenje:

```go
var debug atomic.Bool
```

---

# 15. Production Incident Primer

Scenario:

HTTP servis.

---

Global cache:

```go
var config Config
```

---

Deploy:

nova konfiguracija.

---

Reload:

```go
config = newConfig
```

---

Request:

```go
use(config)
```

---

Rezultat:

neki request-i vide:

stari state.

---

Neki:

novi state.

---

Bug:

```
nekonzistentno ponašanje
```

---

# 16. Race-Free Dizajn Pravila

## Pravilo 1

Izbegavaj globalni mutable state.

---

## Pravilo 2

Preferiraj ownership.

---

## Pravilo 3

Koristi immutable snapshots.

---

## Pravilo 4

Jasno definiši synchronization.

---

## Pravilo 5

Pokreni:

```bash
go test -race ./...
```

---

# 17. Senior Mentalni Model

Najveći broj race bugova nastaje zbog:

```
shared mutable state
```

---

Najbolje rešenje često nije:

```
dodaj mutex
```

---

Nego:

```
ukloni deljenje state-a
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ map race probleme

✅ struct race probleme

✅ config race

✅ lazy initialization race

✅ sync.Once pattern

✅ cache race

✅ slice race

✅ global state race

✅ production scenarije

---

# Data Races Deep Dive

## Deo #5 — Advanced Race Patterns i Debugging Strategije

---

# 📚 Sadržaj

- Subtle race bugovi
- Interface race problemi
- Unsafe package i race rizici
- Race u error handling-u
- Goroutine leak i indirektni race problemi
- Debugging workflow
- Race prevention arhitektura
- Production best practices

---

# 1. Subtle Data Race Bugovi

Nisu svi race bugovi očigledni.

---

Jednostavan primer:

```go
counter++
```

je lako uočljiv.

---

Ali realni sistemi imaju:

- cache
- configuration
- callbacks
- state machine
- lifecycle state

---

Najopasniji race bugovi:

```
izgledaju kao validan kod
```

---

# 2. Race u Interface Vrednostima

Interface nije samo vrednost.

Sadrži:

```
type pointer

+

data pointer
```

---

Primer:

```go
var value interface{}
```

---

Goroutine 1:

```go
value = User{
	Name:"Marko",
}
```

---

Goroutine 2:

```go
fmt.Println(value)
```

---

Problem:

interface assignment nije nužno atomic.

---

# 3. Interface Snapshot Pattern

Loše:

```go
var config interface{}
```

---

Bolje:

```go
var config atomic.Value
```

---

Store:

```go
config.Store(
	Config{
		Port:8080,
	},
)
```

---

Load:

```go
cfg :=
config.Load().
(Config)
```

---

Dobija se:

```
atomic snapshot
```

---

# 4. Race sa Closure-ima

Čest problem:

```go
for i:=0;i<10;i++{

	go func(){

		fmt.Println(i)

	}()

}
```

---

Problem:

closure deli:

```
i
```

---

Sve goroutines pristupaju istoj promenljivoj.

---

Moderna verzija:

```go
for i:=0;i<10;i++{

	i := i

	go func(){

		fmt.Println(i)

	}()

}
```

---

Sada svaka goroutine ima svoj state.

---

# 5. Race u Callback Sistemima

Primer:

```go
type Server struct {

	handler func()

}
```

---

Thread 1:

```go
server.handler =
newHandler
```

---

Thread 2:

```go
server.handler()
```

---

Problem:

read/write race.

---

Rešenje:

atomic pointer:

```go
var handler atomic.Value
```

---

Update:

```go
handler.Store(
	newHandler,
)
```

---

Call:

```go
handler.Load().
(func())()
```

---

# 6. Unsafe Package i Race Rizici

Package:

```go
unsafe
```

zaobilazi Go sigurnosne mehanizme.

---

Primer:

```go
unsafe.Pointer
```

---

Problem:

compiler više nema iste garancije.

---

Kod:

```go
ptr =
unsafe.Pointer(data)
```

---

mora imati:

```
jasnu synchronization strategiju
```

---

# 7. Race sa Unsafe Pointer-ima

Loš primer:

```go
var ptr unsafe.Pointer


ptr = unsafe.Pointer(
	&config,
)
```

---

Druga goroutine:

```go
cfg :=
(*Config)(ptr)
```

---

Problem:

nema atomic exchange.

---

Bolje:

```go
var ptr atomic.Pointer[Config]
```

---

# 8. Error Handling Race

Primer:

```go
var lastError error
```

---

Worker:

```go
lastError = err
```

---

Monitor:

```go
fmt.Println(lastError)
```

---

Problem:

```
shared error state
```

---

Rešenje:

channel:

```go
errors <- err
```

---

ili:

```go
atomic.Value
```

---

# 9. Goroutine Leak kao Indirektan Problem

Goroutine leak:

nije direktno data race.

---

Ali može izazvati:

- stale state
- unexpected writes
- resource exhaustion

---

Primer:

```go
go func(){

	for {

		process()

	}

}()
```

---

Bez:

```
cancel signal
```

---

Goroutine nastavlja zauvek.

---

# 10. Context i Race Prevention

Ispravno:

```go
ctx, cancel :=
context.WithCancel(
	context.Background(),
)

defer cancel()
```

---

Worker:

```go
select {

case <-ctx.Done():

	return

default:

	work()

}
```

---

Prednost:

kontrolisan lifecycle.

---

# 11. Debugging Workflow

Kada postoji race:

---

## Korak 1

Pokreni:

```bash
go test -race ./...
```

---

## Korak 2

Pronađi:

```
shared variable
```

---

## Korak 3

Identifikuj:

```
writer
```

i:

```
reader
```

---

## Korak 4

Pronađi:

```
happens-before edge
```

---

Ako ne postoji:

problem.

---

# 12. Race Debugging Pitanja

Za svaku promenljivu:

---

## Ko je owner?

Primer:

```
worker goroutine
```

---

## Ko menja?

Primer:

```
HTTP handler
```

---

## Ko čita?

Primer:

```
monitor
```

---

## Kako komuniciraju?

Primer:

```
channel
mutex
atomic
```

---

# 13. Race Prevention Arhitektura

Najbolji pristup:

```
design out races
```

---

Primer:

Loše:

```
10 goroutines

↓

shared map
```

---

Bolje:

```
10 goroutines

↓

messages

↓

owner goroutine
```

---

# 14. Actor Model Pattern u Go-u

Go često koristi actor-like dizajn.

---

Jedna goroutine poseduje state.

---

Ostali šalju:

```go
commands
```

---

Primer:

```
Client

 |

channel

 |

State Owner

 |

Memory
```

---

Nema:

```
shared mutation
```

---

# 15. Immutable Design

Umesto:

```go
config.Port = 9000
```

---

Koristi:

```go
newConfig :=
Config{
	Port:9000,
}
```

---

Zameni ceo snapshot.

---

Prednost:

manje race mogućnosti.

---

# 16. Race Prevention Checklist

Pre merge-a:

✅ `go test -race ./...`

✅ nema global mutable state

✅ ownership definisan

✅ lifecycle kontrolisan

✅ immutable gde moguće

✅ synchronization dokumentovan

---

# 17. Senior Pravilo

Data race problem se retko rešava samo tehnikom:

```
dodaj mutex
```

---

Najbolji dizajn:

```
smanji deljeni state

↓

jasan ownership

↓

kontrolisana komunikacija
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ subtle race bugove

✅ interface race

✅ closure race

✅ callback race

✅ unsafe pointer rizike

✅ error state race

✅ goroutine leak veze

✅ debugging workflow

✅ actor model pristup

✅ race prevention arhitekturu

---

# Data Races Deep Dive

## Deo #6 — Data Race Prevention — Final Architecture Patterns

---

# 📚 Sadržaj

- Race-free service design
- Ownership modeli
- Concurrency boundaries
- State isolation
- Message passing arhitektura
- Immutable state patterns
- Synchronization decision matrix
- Code review checklist
- Final Data Race mentalni model

---

# 1. Race-Free Service Design

Cilj modernog concurrent sistema:

```
minimizovati shared mutable state
```

---

Najčešća greška:

```
više goroutines

        |
        v

jedan globalni state
```

---

Bolji dizajn:

```
goroutines

        |
        v

jasno definisani owner-i

        |
        v

kontrolisana komunikacija
```

---

# 2. Ownership Model

Jedno od najvažnijih pravila Go concurrency dizajna:

> Ko poseduje podatak, taj ga menja.

---

Primer:

Loše:

```go
type Cache struct {

	data map[string]string

}
```

---

Više goroutines:

```
G1 -> write

G2 -> write

G3 -> read
```

---

Problem:

```
shared ownership
```

---

Bolje:

```
Cache goroutine

       |
       |
     state

       |
       |
  channel commands
```

---

# 3. Single Owner Pattern

Primer:

```go
type Command struct {

	Key string

	Value string

}
```

---

Owner goroutine:

```go
func runCache(
	commands <-chan Command,
){

	cache :=
	make(map[string]string)


	for cmd :=
	range commands {

		cache[cmd.Key] =
			cmd.Value

	}

}
```

---

Ostale goroutines:

ne pristupaju:

```go
cache
```

direktno.

---

Komuniciraju:

```go
commands <- cmd
```

---

# 4. Concurrency Boundaries

Dobar sistem ima jasne granice.

---

Primer:

```
HTTP Layer

        |
        |
        v

Service Layer

        |
        |
        v

Worker Layer
```

---

Svaki sloj definiše:

- ko menja state
- kako se komunicira
- gde postoji synchronization

---

# 5. State Isolation Pattern

Umesto:

```go
var state State
```

---

Koristi:

```go
type Worker struct {

	state State

}
```

---

Svaka goroutine:

ima svoj:

```
local state
```

---

Rezultat:

manje:

```
locks

+

races
```

---

# 6. Message Passing Arhitektura

Go filozofija:

> Do not communicate by sharing memory; share memory by communicating.

---

Loše:

```
shared memory

+

locks
```

---

Bolje:

```
channels

+

messages
```

---

Primer:

```
Producer

    |
    |
 channel

    |
    |

Consumer
```

---

# 7. Immutable State Pattern

Mutable:

```go
config.Port = 8080
```

---

Problem:

svako može menjati.

---

Immutable:

```go
newConfig :=
Config{
	Port:8080,
}
```

---

Stari objekat:

```
ostaje isti
```

---

Novi:

```
novi snapshot
```

---

# 8. Atomic Snapshot Pattern

Za konfiguracije:

```go
var current atomic.Pointer[Config]
```

---

Update:

```go
current.Store(
	&Config{
		Port:8080,
	},
)
```

---

Read:

```go
cfg :=
current.Load()
```

---

Prednost:

```
lock-free reads
```

---

# 9. Synchronization Decision Matrix

## Mutex

Koristi kada:

```
više operacija

+

zajednički state
```

Primer:

```go
mu.Lock()

a++

b++

mu.Unlock()
```

---

---

## Atomic

Koristi kada:

```
jedna vrednost

+

jednostavna operacija
```

Primer:

```go
atomic.AddInt64()
```

---

---

## Channel

Koristi kada:

```
ownership transfer
```

Primer:

```go
worker <- job
```

---

---

## sync.Once

Koristi kada:

```
jednokratna inicijalizacija
```

Primer:

```go
once.Do(init)
```

---

# 10. Code Review Checklist

Pre prihvatanja concurrent koda:

---

## Shared State

Pitaj:

```
Da li postoji globalna mutable promenljiva?
```

---

Ako postoji:

zašto?

---

# Ownership

Pitaj:

```
Ko poseduje ovaj podatak?
```

---

Ako odgovor nije jasan:

problem.

---

# Synchronization

Pitaj:

```
Koja primitive štiti podatak?
```

---

Primer:

- mutex
- atomic
- channel

---

# Lifecycle

Pitaj:

```
Kako goroutine završava?
```

---

Treba postojati:

- context cancellation
- shutdown signal

---

# 11. Najčešći Anti-Patterni

## Global Mutable State

```go
var users map[string]User
```

---

Problem:

svako pristupa.

---

## Mutex Everywhere

Dodavanje mutex-a svuda:

nije dizajn.

---

Problem:

kompleksnost raste.

---

## Shared Slice

```go
items = append(
	items,
	x,
)
```

iz više goroutines.

---

Problem:

race.

---

## Hidden State

Primer:

```go
package variable
```

koju svi koriste.

---

Problem:

ownership ne postoji.

---

# 12. Production Architecture Primer

Primer web servisa:

```
                 HTTP Requests

                       |
                       |

                Request Handlers

                       |
                       |

              Command Channels

                       |
                       |

              Worker Goroutines

                       |
                       |

              Owned State

```

---

Nema:

```
global shared mutation
```

---

# 13. Testing Strategy

Za concurrent kod:

obavezno:

```bash
go test -race ./...
```

---

Dodati:

- stress testove
- parallel testove
- fuzz testing

---

Primer:

```go
t.Parallel()
```

---

Cilj:

povećati verovatnoću otkrivanja race-a.

---

# 14. Finalni Mentalni Model

Kada vidiš:

```go
var x T
```

pitaj:

---

## 1.

Ko piše?

```
writer
```

---

## 2.

Ko čita?

```
reader
```

---

## 3.

Kako komuniciraju?

```
mutex/channel/atomic
```

---

## 4.

Postoji li happens-before?

```
da/ne
```

---

Ako nema:

```
potencijalni race
```

---

# 15. Data Race Formula

Možemo zapamtiti:

```
Shared Mutable State

+

Multiple Goroutines

+

No Synchronization

=

Data Race
```

---

Prevencija:

```
Ownership

+

Synchronization

+

Clear Lifecycle

=

Race-Free System
```

---

# 📋 Završni Rezime Modula

Završili smo:

```
docs/module-4/extras/
└── 03-data-races-deep-dive.md
```

Obrađeno:

✅ Data race osnove

✅ Race Condition vs Data Race

✅ Go Race Detector

✅ ThreadSanitizer

✅ Happens-Before model

✅ Memory visibility

✅ Realni production bugovi

✅ Advanced race patterns

✅ Ownership modeli

✅ Race-free arhitektura

---

# 🎯 Ključna Senior Lekcija

Najbolji concurrent sistemi nisu oni koji imaju najviše:

```
mutex-a
```

---

Nego oni koji imaju najmanje:

```
shared mutable state
```

---

Dobro dizajniran Go concurrency sistem:

```
jasno vlasništvo

+

kontrolisana komunikacija

+

minimalna sinhronizacija
```

---

### ➡️ Sledeća lekcija **[**Lock-Free Programming**](04-lock-free-programming.md)**

Obuhvatiće:

- lock-free vs wait-free
- CAS algoritme
- atomic primitives
- lock-free data structures
- ABA problem
- memory reclamation
- production lock-free patterns
