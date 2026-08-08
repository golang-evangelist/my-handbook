# Atomic Operations Patterns

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Atomic Operations Patterns
>
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Uvod u atomic operacije
- Zašto postoje atomic operacije?
- Problem sa non-atomic operacijama
- Hardware atomicity
- Go atomic package
- Atomic vs Mutex
- Basic atomic operations
- Memory ordering
- Kada koristiti atomic

---

# 1. Uvod u Atomic Operacije

Atomic operacije predstavljaju operacije koje se izvršavaju kao:

```
nedeljiva celina
```

Drugim rečima:

nema međustanja koje druga goroutine može videti.

---

Primer obične operacije:

```go
counter++
```

izgleda kao jedna operacija.

Ali interno:

```
READ

↓

ADD

↓

WRITE
```

---

Druga goroutine može upasti između:

```
READ

        G2

ADD

        G2

WRITE
```

---

Atomic operacija rešava ovaj problem.

---

# 2. Zašto Postoje Atomic Operacije?

Concurrency problem:

Imamo shared variable:

```go
var counter int64
```

i više goroutines:

```go
go increment()

go increment()

go increment()
```

---

Bez zaštite:

```
Race Condition
```

---

Moguća greška:

```
Expected:

100000


Got:

87342
```

---

Rešenja:

1. Mutex

2. Channel

3. Atomic

---

# 3. Atomicity Koncept

Atomic znači:

Operacija se izvršava:

```
ALL

ili

NOTHING
```

---

Primer:

Atomic increment:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Nema:

```
half completed state
```

---

Druga goroutine vidi:

pre:

```
10
```

ili posle:

```
11
```

---

Ne vidi:

```
10.5
```

---

# 4. Hardware Atomicity

Atomic operacije nisu samo Go koncept.

One koriste CPU instrukcije.

Primer:

x86:

```
LOCK prefix
```

---

ARM:

```
Load-Acquire

Store-Release
```

---

CPU obezbeđuje:

- atomic read
- atomic write
- synchronization ordering

---

Go runtime mapira:

```go
sync/atomic
```

na odgovarajuće CPU primitive.

---

# 5. Go Atomic Package

Glavni package:

```go
sync/atomic
```

---

Omogućava:

- atomic load
- atomic store
- atomic add
- compare-and-swap
- atomic swap

---

Import:

```go
import "sync/atomic"
```

---

# 6. Atomic Load

Čitanje vrednosti:

```go
atomic.LoadInt64(
	&counter,
)
```

---

Primer:

```go
var counter int64

value :=
atomic.LoadInt64(
	&counter,
)

fmt.Println(value)
```

---

Garancija:

read operacija je atomic.

---

# 7. Atomic Store

Upis:

```go
atomic.StoreInt64(
	&counter,
	100,
)
```

---

Primer:

```go
atomic.StoreInt64(
	&status,
	1,
)
```

---

Tipična upotreba:

- flags
- state markers
- configuration pointer

---

# 8. Atomic Add

Najčešća operacija:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Primer:

```go
func Increment(){

	atomic.AddInt64(
		&counter,
		1,
	)

}
```

---

Bezbedno iz više goroutines.

---

# 9. Atomic Swap

Swap menja vrednost i vraća staru.

Primer:

```go
old :=
atomic.SwapInt64(
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

Rezultat:

```
old == 1
```

---

Koristi se za:

- state transition
- ownership transfer

---

# 10. Compare-And-Swap (CAS)

Jedna od najvažnijih atomic operacija.

Format:

```go
CompareAndSwap(
	address,
	old,
	new,
)
```

---

Značenje:

```
Ako je trenutna vrednost == old

promeni u new
```

---

Primer:

```go
atomic.CompareAndSwapInt64(
	&state,
	0,
	1,
)
```

---

Ako:

```
state == 0
```

rezultat:

```
state = 1
```

---

Ako:

```
state == 2
```

ništa se ne menja.

---

# 11. Atomic vs Mutex

## Atomic

Prednosti:

- veoma brz
- nema lock contention
- mali overhead

---

Mane:

- ograničena logika
- teže čitljiv kod
- lako napraviti grešku

---

## Mutex

Prednosti:

- jednostavan
- štiti kompleksan state
- lak za održavanje

---

Mane:

- lock overhead
- contention

---

# 12. Kada Koristiti Atomic?

Dobri kandidati:

---

## Counter

```go
requests++
```

---

## Flags

```go
running = true
```

---

## Statistics

```go
totalRequests
```

---

## State Machine

```go
STARTING

RUNNING

STOPPED
```

---

## Pointer Publication

```go
atomic.Pointer[T]
```

---

# 13. Kada NE Koristiti Atomic?

Primer:

```go
account.balance += payment
```

---

Ovo nije jedna atomic operacija.

Potrebno je:

```
read balance

+

calculate

+

write balance
```

---

Bolje:

```go
mu.Lock()

balance += payment

mu.Unlock()
```

---

# 14. Senior Mental Model

Atomic nije:

```
"brži Mutex"
```

---

Atomic je:

```
alat za veoma male, precizno definisane promene stanja
```

---

Ako problem zahteva:

- invariant
- više promenljivih
- kompleksnu logiku

koristi:

```
Mutex
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su atomic operacije

✅ zašto postoje

✅ CPU atomicity

✅ sync/atomic package

✅ Load

✅ Store

✅ Add

✅ Swap

✅ Compare-And-Swap

✅ Atomic vs Mutex

---

# Atomic Operations Patterns

## Deo #2 — Compare-And-Swap (CAS), Lock-Free Algoritmi i ABA Problem

---

# 📚 Sadržaj

- Uvod u Compare-And-Swap
- Kako CAS funkcioniše
- CAS algoritam
- CAS retry loop
- Lock-free programming koncept
- Atomic state machines
- ABA problem
- Rešavanje ABA problema
- Praktični Go primeri

---

# 1. Uvod u Compare-And-Swap (CAS)

`Compare-And-Swap` je jedna od najvažnijih atomic operacija.

Ideja:

> Promeni vrednost samo ako je ona još uvek ona koju očekujemo.

---

Standardni oblik:

```go
CAS(address, old, new)
```

---

Značenje:

```
Ako:

current value == old


onda:

current value = new
```

---

U suprotnom:

```
ne radi ništa
```

---

# 2. Problem Koji CAS Rešava

Pretpostavimo:

```go
var counter int64
```

Imamo dve goroutines:

```
G1

counter++

G2

counter++
```

---

Bez zaštite:

```
READ 0

READ 0

WRITE 1

WRITE 1
```

---

Rezultat:

```
1
```

umesto:

```
2
```

---

CAS radi drugačije.

---

# 3. CAS Algoritam

Pretpostavimo:

```
counter = 0
```

---

Goroutine 1:

pokušava:

```go
CAS(
	counter,
	0,
	1,
)
```

---

CPU proverava:

```
counter == 0 ?
```

Da:

```
counter = 1
```

---

Goroutine 2:

pokušava:

```go
CAS(
	counter,
	0,
	1,
)
```

---

CPU proverava:

```
counter == 0 ?
```

Ne:

```
counter == 1
```

---

Rezultat:

```
FAIL
```

---

Nema izgubljene promene.

---

# 4. CAS Retry Loop

CAS se često koristi u petlji.

Primer:

```go
func Increment(){

	for {

		old :=
			atomic.LoadInt64(
				&counter,
			)

		newValue :=
			old + 1


		if atomic.CompareAndSwapInt64(
			&counter,
			old,
			newValue,
		){

			return

		}

	}

}
```

---

Algoritam:

```
1. Pročitaj staru vrednost

2. Izračunaj novu

3. Pokušaj CAS

4. Ako neuspeh:

   ponovi
```

---

# 5. Zašto CAS Može da Padne?

Primer:

Početno:

```
counter = 10
```

---

G1:

```
read 10
```

---

G2:

```
read 10
```

---

G2:

```
CAS 10 -> 11

SUCCESS
```

---

G1:

pokušava:

```
CAS 10 -> 11
```

---

Ali sada:

```
counter == 11
```

---

Rezultat:

```
FAIL
```

---

G1 mora ponovo.

---

# 6. Lock-Free Programming

Lock-free znači:

Program napreduje bez tradicionalnih lock-ova.

---

Tradicionalni pristup:

```go
mu.Lock()

update()

mu.Unlock()
```

---

Lock-free:

```go
CAS loop
```

---

Primer:

```
Thread A

   |
   CAS


Thread B

   |
   CAS
```

---

Neko će uspeti.

---

Nema:

```
waiting on mutex
```

---

# 7. Lock-Free Nivoi

Postoje različiti nivoi:

---

## Lock-Free

Garantuje:

neko će napredovati.

---

## Wait-Free

Garantuje:

svaka operacija završava u ograničenom broju koraka.

---

## Obstruction-Free

Garantuje:

operacija uspeva ako radi sama.

---

CAS algoritmi često ciljaju:

```
lock-free
```

---

# 8. Atomic State Machine

Čest production pattern.

Primer:

```go
type State int64
```

---

Stanja:

```go
const (

	StateInit int64 = iota

	StateRunning

	StateStopped

)
```

---

Atomic promena:

```go
atomic.CompareAndSwapInt64(
	&state,
	StateInit,
	StateRunning,
)
```

---

Značenje:

```
Ako je sistem INIT

pređi u RUNNING
```

---

# 9. Primer Lifecycle Controller-a

```go
type Worker struct {

	state int64

}
```

---

Start:

```go
func (w *Worker) Start() bool {

	return atomic.CompareAndSwapInt64(
		&w.state,
		StateInit,
		StateRunning,
	)

}
```

---

Stop:

```go
func (w *Worker) Stop() bool {

	return atomic.CompareAndSwapInt64(
		&w.state,
		StateRunning,
		StateStopped,
	)

}
```

---

Garancija:

samo jedna goroutine može promeniti stanje.

---

# 10. ABA Problem

Jedan od najpoznatijih CAS problema.

---

Primer:

Početno:

```
A
```

---

Goroutine 1:

čita:

```
A
```

---

Goroutine 2:

menja:

```
A → B
```

zatim:

```
B → A
```

---

Goroutine 1 sada vidi:

```
A
```

---

CAS proverava:

```
Da li je i dalje A?
```

Odgovor:

```
DA
```

---

Ali stanje se promenilo.

---

Problem:

```
A

↓

B

↓

A
```

izgleda isto.

---

# 11. Zašto je ABA Problem Opasan?

CAS proverava samo:

```
vrednost
```

---

Ali ne proverava:

```
istoriju promena
```

---

Primer:

Stack:

```
Top = Node A
```

---

Thread 1:

uzima pointer A.

---

Thread 2:

uklanja A.

dodaje A ponovo.

---

Thread 1:

CAS uspe.

---

Ali struktura možda više nije ista.

---

# 12. Rešavanje ABA Problema

## 1. Version Counter

Umesto:

```
A
```

koristi:

```
(A, version)
```

---

Primer:

```
A,1

↓

B,2

↓

A,3
```

---

Vrednost izgleda isto:

```
A
```

ali verzija nije.

---

## 2. Tagged Pointer

Pointer + metadata:

```
address

+

counter
```

---

## 3. Hazard Pointers

Koristi se u kompleksnim lock-free strukturama.

---

# 13. Atomic Pointer u Go-u

Go podržava:

```go
atomic.Pointer[T]
```

---

Primer:

```go
type Config struct {

	Timeout int

}


var config atomic.Pointer[Config]
```

Store:

```go
config.Store(
	&Config{
		Timeout:10,
	},
)
```

Load:

```go
cfg :=
config.Load()
```

---

Prednost:

```
lock-free read
```

---

# 14. Kada Koristiti CAS?

Dobri slučajevi:

✅ state transitions

✅ counters

✅ flags

✅ statistics

✅ lock-free structures

---

Loši slučajevi:

❌ kompleksni business rules

❌ više povezanih vrednosti

❌ transakcije

---

# 15. Senior Pravilo

CAS kod izgleda jednostavno:

```go
CAS(old,new)
```

---

Ali realna pitanja su:

```
Šta ako CAS fail-uje?

Koliko retry pokušaja?

Da li postoji starvation?

Da li postoji ABA problem?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Compare-And-Swap

✅ CAS retry loop

✅ lock-free koncept

✅ state machines

✅ ABA problem

✅ versioning

✅ atomic pointers

✅ kada koristiti CAS

---

# Atomic Operations Patterns

## Deo #3 — Atomic Patterns u Realnim Go Sistemima

---

# 📚 Sadržaj

- Atomic counters pattern
- Metrics pattern
- Request tracking
- Feature flags
- Health state pattern
- Rate limiter pattern
- Lock-free cache pattern
- Atomic configuration pattern
- Production guidelines

---

# 1. Atomic Patterns u Production Sistemima

U realnim Go servisima atomic operacije se najčešće koriste za:

- metrike,
- brojače,
- state tracking,
- feature flags,
- lightweight synchronization.

---

Važno pravilo:

Atomic je najbolji kada imamo:

```
mala promena stanja

+

jasna semantika
```

---

Primer:

```text
request_count += 1
```

---

Nije idealan za:

```text
update kompleksnog objekta
```

---

# 2. Atomic Counter Pattern

Najčešći slučaj:

brojanje događaja.

Primer:

```go
type Metrics struct {

	requests int64

}
```

---

Increment:

```go
func (m *Metrics) IncRequests(){

	atomic.AddInt64(
		&m.requests,
		1,
	)

}
```

---

Read:

```go
func (m *Metrics) Requests() int64 {

	return atomic.LoadInt64(
		&m.requests,
	)

}
```

---

Prednost:

više goroutines može paralelno povećavati counter.

---

# 3. HTTP Request Metrics Primer

Primer server metrike:

```go
type ServerMetrics struct {

	totalRequests int64

	successes int64

	failures int64

}
```

---

Handler:

```go
func handler(){

	atomic.AddInt64(
		&metrics.totalRequests,
		1,
	)

}
```

---

Na kraju:

```go
fmt.Println(
	atomic.LoadInt64(
		&metrics.totalRequests,
	),
)
```

---

Koristi se za:

- Prometheus metrics
- monitoring
- observability

---

# 4. Atomic Gauge Pattern

Counter samo raste.

Gauge može da raste i pada.

Primer:

broj aktivnih konekcija.

---

Povećanje:

```go
atomic.AddInt64(
	&connections,
	1,
)
```

---

Smanjenje:

```go
atomic.AddInt64(
	&connections,
	-1,
)
```

---

Primer:

```
connections = 150
```

---

Disconnect:

```
connections = 149
```

---

# 5. Active Worker Tracking

Primer:

Worker pool:

```go
var activeWorkers int64
```

---

Worker start:

```go
atomic.AddInt64(
	&activeWorkers,
	1,
)
```

---

Worker stop:

```go
atomic.AddInt64(
	&activeWorkers,
	-1,
)
```

---

Monitoring:

```go
current :=
atomic.LoadInt64(
	&activeWorkers,
)
```

---

# 6. Atomic Boolean Pattern

Go ima:

```go
atomic.Bool
```

---

Primer:

```go
var running atomic.Bool
```

---

Start:

```go
running.Store(true)
```

---

Check:

```go
if running.Load(){

	serve()

}
```

---

Koristi se za:

- lifecycle state
- shutdown flags
- feature switches

---

# 7. Shutdown Signal Pattern

Primer:

```go
type Worker struct {

	stopped atomic.Bool

}
```

---

Run loop:

```go
for {

	if w.stopped.Load(){

		return

	}

	process()

}
```

---

Stop:

```go
w.stopped.Store(true)
```

---

Prednost:

nema mutex-a.

---

Ali:

za kompleksan shutdown:

koristi:

```
context.Context
```

---

# 8. Feature Flag Pattern

Primer:

```go
type FeatureFlags struct {

	newAlgorithm atomic.Bool

}
```

---

Enable:

```go
flags.newAlgorithm.Store(true)
```

---

Usage:

```go
if flags.newAlgorithm.Load(){

	useNewAlgorithm()

}else{

	useOldAlgorithm()

}
```

---

Prednost:

runtime promena bez restartovanja.

---

# 9. Atomic Configuration Pattern

Čest pattern:

read-heavy configuration.

---

Problem:

Config se retko menja.

Ali se mnogo čita.

---

Mutex pristup:

```
svako čitanje

↓

lock

↓

unlock
```

---

Atomic pointer:

```
read

↓

Load pointer

↓

use config
```

---

Primer:

```go
type Config struct {

	Timeout time.Duration

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
		Timeout:5*time.Second,
	},
)
```

---

Read:

```go
cfg :=
config.Load()

fmt.Println(
	cfg.Timeout,
)
```

---

# 10. Rate Limiter Counter Pattern

Jednostavan limiter:

```go
var requests int64
```

---

Svaki zahtev:

```go
current :=
atomic.AddInt64(
	&requests,
	1,
)
```

---

Provera:

```go
if current > limit {

	return errors.New(
		"rate exceeded",
	)

}
```

---

Napomena:

Za ozbiljan distributed rate limiter koristi:

- token bucket
- Redis
- distributed counter

---

# 11. Lock-Free Cache Pattern

Scenario:

mnogo čitanja.

malo promena.

---

Struktura:

```
Readers

   |

atomic.Pointer

   |

Cache snapshot
```

---

Primer:

```go
type Cache struct {

	Items map[string]string

}
```

---

Read:

```go
cache :=
current.Load()

value :=
cache.Items[key]
```

---

Update:

kreira novu verziju:

```go
newCache := clone(old)

newCache.Items[key] = value

current.Store(newCache)
```

---

Ovo je:

```
copy-on-write
```

---

# 12. Atomic State Machine u Servisima

Primer:

```go
const (

	StateCreated int64 = iota

	StateRunning

	StateStopped

)
```

---

Transition:

```go
atomic.CompareAndSwapInt64(
	&state,
	StateCreated,
	StateRunning,
)
```

---

Garantuje:

samo jedan start.

---

Koristi se kod:

- servers
- workers
- background jobs
- lifecycle managers

---

# 13. Atomic Statistics Object

Primer:

```go
type Stats struct {

	Requests atomic.Int64

	Errors atomic.Int64

}
```

---

Usage:

```go
stats.Requests.Add(1)

stats.Errors.Add(1)
```

---

Moderni Go stil:

koristi typed atomic types:

```go
atomic.Int64
```

umesto:

```go
atomic.AddInt64()
```

---

# 14. Atomic Pattern Limitacije

Atomic nije rešenje za:

---

## Više povezanih vrednosti

Primer:

```go
balance

+

transactions
```

---

## Invarijante

Primer:

```go
if stock > 0 {

	stock--

}
```

---

## Kompleksne promene

Primer:

```go
updateUserProfile()
```

---

Koristi:

```
Mutex

ili

Transaction model
```

---

# 15. Production Guidelines

Koristi atomic kada:

✅ jedna promenljiva

✅ jednostavna tranzicija

✅ visok broj čitanja

✅ nizak contention

---

Izbegavaj kada:

❌ logika postaje kompleksna

❌ potrebno je više lock koraka

❌ teško je dokazati ispravnost

---

# 16. Senior Mentalni Model

Atomic rešava:

```
Kako promeniti jednu vrednost bez race condition-a?
```

---

Ne rešava:

```
Kako dizajnirati concurrent sistem?
```

---

Dobar concurrency dizajn počinje sa:

```
ownership

↓

communication

↓

synchronization
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ atomic counters

✅ metrics pattern

✅ gauges

✅ worker tracking

✅ atomic bool

✅ shutdown flags

✅ feature flags

✅ atomic config

✅ rate limiter pattern

✅ lock-free cache

✅ state machines

---

# Atomic Operations Patterns

## Deo #4 — Atomic Types u Modernom Go-u

---

# 📚 Sadržaj

- Evolucija Go atomic API-ja
- Legacy atomic funkcije
- Typed atomic types
- atomic.Int32
- atomic.Int64
- atomic.Uint32
- atomic.Uint64
- atomic.Bool
- atomic.Pointer[T]
- atomic.Value
- Migracija na moderni API
- Best practices

---

# 1. Evolucija Go Atomic API-ja

Go je dugo koristio funkcionalni atomic API:

```go
sync/atomic
```

---

Primer starog stila:

```go
var counter int64


atomic.AddInt64(
	&counter,
	1,
)
```

---

Problem:

- lako pogrešiti tip
- potrebno voditi računa o pointerima
- manje čitljiv kod

---

Moderni Go uvodi typed atomic types.

---

Od Go 1.19:

```go
atomic.Int64
```

---

Primer:

```go
var counter atomic.Int64

counter.Add(1)
```

---

# 2. Legacy Atomic API

Stari API:

```go
atomic.LoadInt64()

atomic.StoreInt64()

atomic.AddInt64()

atomic.CompareAndSwapInt64()

atomic.SwapInt64()
```

---

Primer:

```go
var value int64

atomic.StoreInt64(
	&value,
	100,
)

result :=
atomic.LoadInt64(
	&value,
)
```

---

Radi ispravno.

Ali postoji veća mogućnost greške.

---

# 3. Typed Atomic Types

Novi API koristi strukture.

Package:

```go
sync/atomic
```

---

Dostupni tipovi:

```text
atomic.Bool

atomic.Int32

atomic.Int64

atomic.Uint32

atomic.Uint64

atomic.Uintptr

atomic.Pointer[T]

atomic.Value
```

---

Prednost:

state je enkapsuliran.

---

# 4. atomic.Int64

Najčešće korišćen tip.

---

Primer:

```go
type Metrics struct {

	requests atomic.Int64

}
```

---

Increment:

```go
metrics.requests.Add(1)
```

---

Read:

```go
count :=
metrics.requests.Load()
```

---

Reset:

```go
metrics.requests.Store(0)
```

---

# 5. Counter Primer sa atomic.Int64

Pre:

```go
var requests int64


func Inc(){

	atomic.AddInt64(
		&requests,
		1,
	)

}
```

---

Posle:

```go
var requests atomic.Int64


func Inc(){

	requests.Add(1)

}
```

---

Kod je:

- kraći
- sigurniji
- čitljiviji

---

# 6. atomic.Bool

Za boolean state.

---

Primer:

```go
var running atomic.Bool
```

---

Set:

```go
running.Store(true)
```

---

Read:

```go
if running.Load(){

	process()

}
```

---

Swap:

```go
old :=
running.Swap(false)
```

---

CAS:

```go
running.CompareAndSwap(
	false,
	true,
)
```

---

# 7. Lifecycle Primer sa atomic.Bool

Primer:

```go
type Worker struct {

	stopped atomic.Bool

}
```

---

Run:

```go
func (w *Worker) Run(){

	for {

		if w.stopped.Load(){

			return

		}

		work()

	}

}
```

---

Stop:

```go
func (w *Worker) Stop(){

	w.stopped.Store(true)

}
```

---

Jednostavan lifecycle signal.

---

# 8. atomic.Pointer[T]

Jedan od najmoćnijih modernih atomic tipova.

---

Koristi se za:

- immutable snapshots
- configuration
- caches
- routing tables

---

Primer:

```go
type Config struct {

	Timeout int

}
```

---

Pointer:

```go
var config atomic.Pointer[Config]
```

---

Store:

```go
config.Store(
	&Config{
		Timeout:10,
	},
)
```

---

Load:

```go
cfg :=
config.Load()
```

---

# 9. Atomic Configuration Pattern

Scenario:

```
90% reads

10% updates
```

---

Mutex:

```
Read

↓

Lock

↓

Unlock
```

---

Atomic pointer:

```
Read

↓

Load pointer
```

---

Primer:

```go
func GetConfig() *Config {

	return config.Load()

}
```

---

Update:

```go
func UpdateConfig(
	cfg *Config,
){

	config.Store(cfg)

}
```

---

# 10. atomic.Value

Pre typed atomic types postojao je:

```go
atomic.Value
```

---

Omogućava čuvanje bilo kog tipa.

---

Primer:

```go
var config atomic.Value
```

---

Store:

```go
config.Store(
	Config{
		Timeout:10,
	},
)
```

---

Load:

```go
cfg :=
config.Load()
```

---

# 11. atomic.Value Pravila

Postoje važna pravila.

---

Prvo:

tip mora biti isti.

---

Ispravno:

```go
config.Store(
	Config{},
)

config.Store(
	Config{},
)
```

---

Pogrešno:

```go
config.Store(
	Config{},
)

config.Store(
	OtherConfig{},
)
```

---

Runtime panic.

---

# 12. atomic.Pointer vs atomic.Value

## atomic.Pointer[T]

Prednosti:

- type safe
- compile-time provera
- bolji API

---

Primer:

```go
atomic.Pointer[Config]
```

---

## atomic.Value

Prednosti:

- fleksibilnost
- legacy kompatibilnost

---

Primer:

```go
atomic.Value
```

---

Moderna preporuka:

koristi:

```
atomic.Pointer[T]
```

kada je moguće.

---

# 13. Non-Copy Rule

Atomic objekti ne smeju da se kopiraju nakon prve upotrebe.

---

Loše:

```go
type Counter struct {

	value atomic.Int64

}
```

---

Kopiranje:

```go
copy := original
```

---

Problem:

dobija se kopirani atomic state.

---

Pravilo:

```
Do not copy atomic values.
```

---

# 14. Zero Value

Typed atomic types imaju koristan zero value.

---

Primer:

```go
var counter atomic.Int64
```

---

Početna vrednost:

```
0
```

---

Može odmah:

```go
counter.Add(1)
```

---

Nema:

```go
NewAtomicInt64()
```

---

# 15. Migracija Legacy → Modern API

Pre:

```go
var count int64


atomic.AddInt64(
	&count,
	1,
)
```

---

Posle:

```go
var count atomic.Int64


count.Add(1)
```

---

Pre:

```go
atomic.LoadInt64(
	&state,
)
```

---

Posle:

```go
state.Load()
```

---

# 16. Best Practices

Koristi:

## atomic.Int64

za:

- counters
- metrics
- statistics

---

## atomic.Bool

za:

- flags
- lifecycle state

---

## atomic.Pointer[T]

za:

- immutable snapshots
- configs
- caches

---

## atomic.Value

za:

- legacy code
- dynamic values

---

# 17. Senior Pravilo

Modern atomic API rešava:

```
kako bezbednije napisati atomic operaciju
```

---

Ali ne rešava:

```
da li atomic treba koristiti
```

---

Pre upotrebe pitaj:

```
Da li je ovo zaista jedna nezavisna promenljiva?
```

Ako nije:

koristi drugi concurrency model.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ legacy atomic API

✅ typed atomic types

✅ atomic.Int64

✅ atomic.Bool

✅ atomic.Pointer[T]

✅ atomic.Value

✅ zero value

✅ non-copy rule

✅ migration strategy

---

# Atomic Operations Patterns

## Deo #5 — Lock-Free Data Structures u Go-u

---

# 📚 Sadržaj

- Uvod u lock-free strukture
- Zašto koristiti lock-free strukture?
- Lock-free Stack
- CAS bazirani algoritmi
- Lock-free Queue koncept
- Memory reclamation problem
- ABA problem u strukturama podataka
- Ograničenja lock-free pristupa
- Kada koristiti lock-free

---

# 1. Uvod u Lock-Free Data Structures

Tradicionalne strukture podataka koriste lock:

Primer:

```text
Goroutine

   |
   Lock

   |
 Queue

   |
 Unlock
```

---

Lock-free strukture koriste:

```
atomic operations

+

CAS
```

---

Ideja:

Umesto:

```
čekaj lock
```

koristi:

```
pokušaj promenu

ako neuspeh:

retry
```

---

# 2. Zašto Koristiti Lock-Free Strukture?

Prednosti:

## 1. Manji contention

Nema:

```
goroutine waiting
```

---

## 2. Bolji throughput

Više goroutines može pokušavati paralelno.

---

## 3. Predvidljivije ponašanje

Nema:

- lock convoy
- priority inversion

---

Ali:

cena je:

```
kompleksnost
```

---

# 3. Lock-Free Stack Koncept

Stack:

```
LIFO
```

---

Primer:

```
TOP

 |
 A

 |
 B

 |
 C
```

---

Operacije:

Push:

```
dodaj element na vrh
```

Pop:

```
uzmi element sa vrha
```

---

# 4. Klasičan Stack sa Mutex-om

Primer:

```go
type Stack struct {

	mu sync.Mutex

	items []int

}
```

---

Push:

```go
func (s *Stack) Push(v int){

	s.mu.Lock()

	defer s.mu.Unlock()

	s.items =
		append(
			s.items,
			v,
		)

}
```

---

Jednostavno.

Ali:

svaka operacija čeka lock.

---

# 5. Lock-Free Stack Struktura

Koristimo linked list:

```go
type Node struct {

	value int

	next *Node

}
```

---

Stack:

```go
type Stack struct {

	top atomic.Pointer[Node]

}
```

---

Stanje:

```
top

 |

Node

 |

Node

 |
nil
```

---

# 6. Lock-Free Push

Ideja:

1. Kreiraj novi node

2. Pročitaj trenutni top

3. Postavi next

4. CAS promeni top

---

Kod:

```go
func (s *Stack) Push(value int){

	node :=
	&Node{
		value:value,
	}


	for {

		oldTop :=
			s.top.Load()


		node.next =
			oldTop


		if s.top.CompareAndSwap(
			oldTop,
			node,
		){

			return

		}

	}

}
```

---

Ako neko drugi promeni top:

CAS failuje.

---

Retry.

---

# 7. Lock-Free Pop

Algoritam:

1. Pročitaj top

2. Uzmi next

3. CAS promeni top

---

Kod:

```go
func (s *Stack) Pop() *Node {

	for {

		oldTop :=
			s.top.Load()


		if oldTop == nil {

			return nil

		}


		next :=
			oldTop.next


		if s.top.CompareAndSwap(
			oldTop,
			next,
		){

			return oldTop

		}

	}

}
```

---

# 8. Problem: ABA u Stack-u

Naizgled:

```
Top = A
```

---

Thread 1:

čita:

```
A
```

---

Thread 2:

pop:

```
A
```

---

Thread 2:

push:

```
A
```

---

Thread 1:

CAS:

```
A -> B
```

uspeva.

---

Ali struktura više nije ista.

---

Rešenje:

version counter.

---

# 9. Tagged Pointer Pattern

Umesto:

```text
pointer
```

koristimo:

```
(pointer, version)
```

---

Primer:

```
Node A

version 1
```

---

Posle:

```
Node A

version 3
```

---

Pointer isti.

Ali stanje drugačije.

---

# 10. Lock-Free Queue Koncept

Queue:

```
FIFO
```

---

Primer:

```
HEAD

↓

A

↓

B

↓

C

↓

TAIL
```

---

Najpoznatiji algoritam:

```
Michael-Scott Queue
```

---

Koristi:

- atomic pointers
- CAS
- dummy node

---

# 11. Michael-Scott Queue Ideja

Struktura:

```go
type Node struct {

	value any

	next atomic.Pointer[Node]

}
```

---

Queue:

```go
type Queue struct {

	head atomic.Pointer[Node]

	tail atomic.Pointer[Node]

}
```

---

Operacije:

Enqueue:

```
dodaj iza tail
```

---

Dequeue:

```
uzmi iza head
```

---

# 12. Zašto je Queue Teža od Stack-a?

Stack menja:

```
jedan pointer
```

---

Queue menja:

```
head

+

tail

+

next pointer
```

---

Više atomic tranzicija.

---

Veća mogućnost greške.

---

# 13. Memory Reclamation Problem

Veliki problem lock-free struktura.

---

Primer:

Node:

```
A
```

---

Thread 1:

drži pointer na A.

---

Thread 2:

ukloni A.

---

Ko oslobađa memoriju?

---

U jezicima bez GC:

problem:

```
use-after-free
```

---

Primeri rešenja:

- hazard pointers
- epoch reclamation
- reference counting

---

# 14. Go i Memory Reclamation

Go ima:

```
Garbage Collector
```

---

Zato nema klasičan:

```
free()
```

problem.

---

Ali i dalje postoji problem:

```
logička bezbednost
```

---

Primer:

node može biti:

- uklonjen iz strukture
- ali i dalje referenciran

---

GC rešava memoriju.

Ne rešava algoritamsku ispravnost.

---

# 15. Lock-Free Ne Znači Brže

Česta zabluda:

```
lock-free = faster
```

---

Nije uvek tačno.

---

CAS retry može izazvati:

```
high CPU usage
```

---

Mutex može biti brži kada:

- contention je nizak
- kritična sekcija mala

---

# 16. Kada Koristiti Lock-Free Strukture?

Koristi kada:

✅ ekstreman throughput

✅ visok contention

✅ poznat algoritam

✅ performance kritičan sistem

---

Ne koristi kada:

❌ jednostavan mutex rešava problem

❌ tim ne razume algoritam

❌ nema potrebe za optimizacijom

---

# 17. Production Pravilo

Prvo:

```
napravi korektan kod
```

zatim:

```
izmeri performance
```

zatim:

```
optimizuj
```

---

Lock-free je:

```
poslednji alat

ne prvi izbor
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ lock-free koncept

✅ CAS algoritme

✅ lock-free stack

✅ lock-free queue koncept

✅ ABA problem

✅ tagged pointer

✅ memory reclamation

✅ ograničenja lock-free pristupa

---

# Atomic Operations Patterns

## Deo #6 — Performance, Benchmarking i Production Tuning

---

# 📚 Sadržaj

- Atomic performance model
- Atomic vs Mutex benchmark
- Contention analiza
- Cache line efekat
- False sharing
- Memory ordering overhead
- CPU cache coherency
- Performance tuning strategije
- Kada optimizovati

---

# 1. Atomic Performance Model

Atomic operacije izgledaju jednostavno:

```go
counter.Add(1)
```

---

Ali ispod postoji:

```
Goroutine

↓

Compiler barrier

↓

CPU atomic instruction

↓

Cache coherency protocol

↓

Memory update
```

---

Atomic nije običan:

```go
x++
```

---

CPU mora garantovati:

```
niko drugi ne menja vrednost istovremeno
```

---

# 2. Atomic vs Mutex

Poređenje:

| | Atomic | Mutex |
|-|-|-|
| Granularnost | Jedna vrednost | Kritična sekcija |
| Lock | Ne | Da |
| Overhead | Mali | Veći |
| Kompleksnost | Veća | Manja |
| Debugging | Teži | Lakši |
| Fleksibilnost | Mala | Velika |

---

Primer:

Counter:

```go
counter++
```

Atomic:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Mutex:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

# 3. Benchmark Primer

Primer testiranja:

```go
func BenchmarkAtomic(
	b *testing.B,
){

	var counter atomic.Int64


	b.RunParallel(
		func(pb *testing.PB){

			for pb.Next(){

				counter.Add(1)

			}

		},
	)

}
```

---

Mutex verzija:

```go
func BenchmarkMutex(
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

Pokretanje:

```bash
go test -bench=. -cpu=8
```

---

# 4. Rezultat Benchmark-a

Nije uvek:

```
Atomic > Mutex
```

---

Zavisi od:

- broja goroutines
- contention nivoa
- dužine kritične sekcije
- CPU arhitekture

---

Primer:

Mali workload:

```
Mutex ≈ Atomic
```

---

Veliki contention:

```
Atomic često bolji
```

---

Kompleksna operacija:

```
Mutex bolji
```

---

# 5. Contention Analiza

Contention znači:

više goroutines pokušava isti resurs.

---

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

1000 goroutines:

```
G1
 |
G2
 |
G3
 |
...
 |
counter
```

---

Svi čekaju CPU cache ownership.

---

Problem više nije lock.

Problem je:

```
cache contention
```

---

# 6. CPU Cache Coherency

Moderni CPU ima:

```
L1 Cache

L2 Cache

L3 Cache

RAM
```

---

Ako jedna jezgra menja vrednost:

```
core 1 cache
```

---

druga jezgra mora dobiti novu vrednost:

```
invalidate cache line
```

---

Atomic operacije zahtevaju:

```
cache synchronization
```

---

To ima cenu.

---

# 7. Cache Line

CPU ne učitava jednu promenljivu.

Učitava:

```
cache line
```

---

Tipično:

```
64 bytes
```

---

Primer:

```go
type Counters struct {

	a int64

	b int64

}
```

---

Moguće:

```
| a | b |
```

ista cache line.

---

Problem:

promena `a` invalidira cache i za `b`.

---

# 8. False Sharing

False sharing:

dve nezavisne promenljive dele istu cache line.

---

Primer:

```go
type Data struct {

	counter1 atomic.Int64

	counter2 atomic.Int64

}
```

---

Thread 1:

menja:

```
counter1
```

---

Thread 2:

menja:

```
counter2
```

---

Iako nisu povezani:

CPU stalno sinhronizuje cache.

---

Rezultat:

performanse padaju.

---

# 9. Padding Pattern

Rešenje:

odvojiti cache line.

---

Primer:

```go
type Counter struct {

	value atomic.Int64

	_ [56]byte

}
```

---

Ideja:

```
counter

64 bytes

counter
```

---

Smanjuje false sharing.

---

# 10. Sharded Atomic Counters

Još bolje rešenje:

ne koristi jedan counter.

---

Umesto:

```
counter
```

koristi:

```
counter[0]

counter[1]

counter[2]

counter[3]
```

---

Svaka goroutine koristi svoj shard.

---

Primer:

```go
type ShardedCounter struct {

	shards []atomic.Int64

}
```

---

Read:

saberi sve:

```
total =
sum(shards)
```

---

Prednost:

manji contention.

---

# 11. Memory Ordering Overhead

Atomic operacije imaju ordering pravila.

---

CPU može optimizovati:

normalne operacije.

---

Ali atomic:

zahteva:

```
visibility guarantee
```

---

Cena:

manje optimizacije.

---

Zato:

ne koristiti atomic svuda.

---

# 12. Benchmark Greške

Česta greška:

benchmark nije realan.

---

Primer:

```go
for i:=0;i<N;i++{

	counter++

}
```

---

Ovo ne simulira:

```
multiple goroutines
```

---

Bolje:

```go
b.RunParallel()
```

---

# 13. Production Tuning Strategija

Redosled:

---

## 1. Izmeri

Koristi:

```bash
go test -bench
```

---

## 2. Profiling

CPU:

```bash
go tool pprof
```

---

Memory:

```bash
go tool pprof
```

---

## 3. Analiza

Pitaj:

```
Da li je contention problem?
```

---

## 4. Optimizacija

Opcije:

- atomic
- sharding
- batching
- lock reduction

---

# 14. Kada Atomic Ne Pomaže?

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Ali:

100 miliona poziva.

---

Problem:

nije atomic.

Problem:

```
algoritam
```

---

Bolje:

batch update:

```
local counter

↓

periodic flush

↓

global counter
```

---

# 15. Senior Performance Pravilo

Najbrži concurrent kod često nije:

```
bolji lock
```

---

Nego:

```
manje deljenog state-a
```

---

Dizajn:

loše:

```
100 goroutines

↓

jedna promenljiva
```

---

bolje:

```
100 goroutines

↓

lokalni state

↓

merge
```

---

# 16. Finalna Checklista

Pre atomic optimizacije:

✅ postoji benchmark

✅ postoji profil

✅ potvrđen contention

✅ razumemo memory model

✅ razumemo cache efekte

---

# 📋 Rezime

U ovom delu naučili smo:

✅ atomic performance model

✅ atomic vs mutex

✅ benchmarking

✅ contention

✅ cache line

✅ false sharing

✅ padding

✅ sharded counters

✅ memory ordering

✅ production tuning

---

# 🎯 Kraj fajla

Završili smo:

```
docs/module-4/extras/
└── 02-atomic-operations-patterns.md
```

Obuhvaćeno:

- Atomic primitives
- CAS
- Lock-free koncepti
- Atomic types
- Production patterns
- Performance tuning

---

### ➡️ Sledeća lekcija **[**Data Races Deep Dive**](03-data-races-deep-dive.md)**

Obuhvatiće:

- race condition vs data race
- Go race detector
- happens-before analiza
- compiler reorderings
- CPU memory effects
- real-world race bugovi
