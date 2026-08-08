# Lock-Free Programming

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Lock-Free Programming
>
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Uvod u Lock-Free Programming
- Zašto postoji lock-free pristup
- Problem tradicionalnih lock-ova
- Lock-Based vs Lock-Free pristup
- Progress Guarantees
- Wait-Free
- Lock-Free
- Obstruction-Free
- Non-Blocking algoritmi
- Atomic primitives kao osnova

---

# 1. Uvod u Lock-Free Programming

Lock-Free Programming je oblast konkurentnog programiranja koja pokušava da izbegne tradicionalne:

```go
sync.Mutex
```

mehanizme.

---

Klasičan pristup:

```
Goroutine A

    |
    |
  Lock

    |
  Critical Section

    |
 Unlock
```

---

Lock-free pristup:

```
Goroutine A

    |
 Atomic Operation

    |
 Continue
```

---

Cilj:

```
više paralelizma

+

manje blokiranja
```

---

# 2. Zašto Postoji Lock-Free?

Mutex je veoma koristan.

Ali ima cenu:

- blokiranje goroutines
- scheduler overhead
- contention
- priority inversion
- deadlock rizik

---

Primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Ako 1000 goroutines pokušava:

```
1000 goroutines
        |
        v
      mutex
```

---

Samo jedna napreduje.

---

Ostale:

```
čekaju
```

---

# 3. Problem Tradicionalnih Lock-ova

## 3.1 Contention

Primer:

```go
mu.Lock()

update()

mu.Unlock()
```

---

Ako:

```
kritična sekcija kratka
```

mutex je odličan.

---

Ali:

```
mnogo goroutines
+
isti lock
```

može postati usko grlo.

---

# 4. Lock-Based Programiranje

Tradicionalni model:

```
Shared State

      |
      |

Synchronization Lock

      |
      |

Multiple Goroutines
```

---

Primer:

```go
type Counter struct {

	mu sync.Mutex

	value int

}


func (c *Counter) Increment(){

	c.mu.Lock()

	c.value++

	c.mu.Unlock()

}
```

---

Prednosti:

✅ jednostavno

✅ lako za razumevanje

✅ lako za održavanje

---

Mane:

❌ blokiranje

❌ contention

❌ moguće deadlock situacije

---

# 5. Lock-Free Programiranje

Lock-free pristup:

ne koristi:

```go
Lock()
Unlock()
```

---

Umesto toga koristi:

- atomic operations
- CAS
- memory ordering

---

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Nema:

```
waiting queue
```

---

Nema:

```
blocked goroutine
```

---

# 6. Atomic Kao Osnova Lock-Free Dizajna

Najvažnija operacija:

## Compare-And-Swap

CAS:

```
ako je vrednost očekivana

promeni je

inače neuspeh
```

---

Model:

```text
CAS(
 address,
 old,
 new
)
```

---

Primer:

Početno:

```
counter = 10
```

---

Goroutine:

```
ako counter == 10

postavi 11
```

---

Ako druga goroutine promeni:

```
counter = 20
```

CAS neuspeva.

---

# 7. CAS Primer u Go-u

Sa atomic paketom:

```go
var value int64


ok :=
atomic.CompareAndSwapInt64(
	&value,
	0,
	1,
)
```

---

Značenje:

```
ako je value == 0

postavi value = 1
```

---

Rezultat:

```go
ok == true
```

ili:

```go
ok == false
```

---

# 8. Lock-Free Counter Primer

Mutex verzija:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Lock-free verzija:

```go
type Counter struct {

	value atomic.Int64

}
```

---

Increment:

```go
func (c *Counter) Inc(){

	c.value.Add(1)

}
```

---

Nema lock-a.

---

# 9. Ali Lock-Free Ne Znači Brže

Česta zabluda:

```
lock-free = faster
```

---

Nije uvek tačno.

---

Lock-free može biti sporiji zbog:

- CAS retry loop-a
- cache invalidacije
- memory barriers

---

Primer:

1000 goroutines:

```
CAS contention
```

može biti veliki.

---

# 10. Progress Guarantees

Kod konkurentnih algoritama postoje različiti nivoi garancija.

---

Hijerarhija:

```
Wait-Free

    ↓

Lock-Free

    ↓

Obstruction-Free

    ↓

Blocking
```

---

Jača garancija:

više kompleksnosti.

---

# 11. Blocking Algoritam

Primer:

```go
mu.Lock()

operation()

mu.Unlock()
```

---

Ako goroutine drži lock:

ostali čekaju.

---

Garancija:

nema.

---

# 12. Obstruction-Free

Najslabija non-blocking garancija.

---

Znači:

Ako jedna goroutine radi sama:

ona će završiti.

---

Ali:

druge goroutines mogu stalno ometati.

---

Primer:

```
G1 pokušava

G2 menja stanje

G1 retry
```

---

# 13. Lock-Free

Lock-free garantuje:

> Sistem kao celina napreduje.

---

Ne mora svaka goroutine završiti.

---

Primer:

```
G1 retry

G2 uspe

```

---

Postoji napredak:

```
progress happened
```

---

# 14. Wait-Free

Najjača garancija.

---

Svaka operacija završava u ograničenom broju koraka.

---

Primer:

```
G1 završi

G2 završi

G3 završi
```

---

Nema starvation-a.

---

# 15. Lock-Free Mentalni Model

Mutex:

```
čekaj dok lock nije slobodan
```

---

Lock-Free:

```
probaj

ako ne uspe

ponovi
```

---

Primer:

```
CAS

↓

uspe?

↓

da -> gotovo

ne -> retry
```

---

# 16. Kada Koristiti Lock-Free?

Dobri slučajevi:

✅ counters

✅ metrics

✅ statistics

✅ state flags

✅ atomic snapshots

---

Loši slučajevi:

❌ kompleksni objekti

❌ više povezanih vrednosti

❌ poslovna logika

---

# 17. Senior Pravilo

Lock-free nije zamena za mutex.

---

Pravo pitanje nije:

```
Kako ukloniti lock?
```

---

Nego:

```
Koji synchronization model najbolje odgovara problemu?
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je lock-free programming

✅ razliku lock-based i lock-free

✅ atomic osnove

✅ CAS koncept

✅ progress guarantees

✅ wait-free

✅ obstruction-free

✅ kada koristiti lock-free

---

# Lock-Free Programming

## Deo #2 — Compare-And-Swap (CAS) Deep Dive

---

# 📚 Sadržaj

- Šta je Compare-And-Swap
- CAS operacija korak po korak
- CAS kao atomarni primitiv
- CAS retry loop
- Optimistic concurrency
- CAS vs Mutex
- Atomic primitives u Go-u
- Lock-Free counter implementacija
- Lock-Free state update
- CAS ograničenja

---

# 1. Šta je Compare-And-Swap?

`Compare-And-Swap` (CAS) je jedna od najvažnijih operacija u lock-free programiranju.

---

CAS radi tri stvari:

1. pročita trenutnu vrednost

2. uporedi je sa očekivanom vrednošću

3. zameni je novom vrednošću ako se poklapa

---

Pseudo kod:

```text
CAS(
    address,
    expected,
    new
)
```

---

Značenje:

```
ako je *address == expected

    *address = new

inače

    ništa
```

---

# 2. CAS Korak po Korak

Početno stanje:

```
counter = 10
```

---

Goroutine A želi:

```
counter = 11
```

---

Radi:

```text
CAS(
 counter,
 10,
 11
)
```

---

Provera:

```
counter == 10 ?
```

---

Da:

```
counter = 11
```

---

Rezultat:

```
success
```

---

# 3. Konflikt Dve Goroutines

Početno:

```
counter = 10
```

---

Goroutine A:

```
CAS(10 -> 11)
```

---

Goroutine B:

```
CAS(10 -> 20)
```

---

Ko prva uspe:

```
pobeđuje
```

---

Primer:

A:

```
counter = 11
```

---

B proverava:

```
counter == 10 ?
```

---

Ne.

---

B dobija:

```
false
```

---

Mora retry.

---

# 4. CAS Retry Loop

Najčešći lock-free obrazac:

```go
for {

	old :=
	atomic.LoadInt64(
		&value,
	)


	new :=
		old + 1


	if atomic.CompareAndSwapInt64(
		&value,
		old,
		new,
	){

		break

	}

}
```

---

Algoritam:

```
READ

↓

CALCULATE

↓

CAS

↓

success?

    yes -> finish

    no -> retry
```

---

# 5. Zašto Retry Radi?

CAS je optimistic.

---

Pretpostavka:

```
verovatno niko neće promeniti vrednost
```

---

Ako se promena desi:

```
detektuj konflikt
```

---

Umesto:

```
čekaj lock
```

radi:

```
probaj ponovo
```

---

# 6. Optimistic Concurrency

CAS koristi:

```
optimistic locking
```

---

Za razliku od:

```
pessimistic locking
```

---

## Pessimistic

Mutex:

```
zaključaj pre promene
```

---

## Optimistic

CAS:

```
promeni ako je stanje očekivano
```

---

Primer iz baze:

```
UPDATE users

SET name='A'

WHERE version=5
```

---

Ako version nije:

```
5
```

update propada.

---

Isti princip.

---

# 7. CAS i Atomicity

CAS mora biti:

```
jedna nedeljiva operacija
```

---

Ne sme postojati:

```
prozor između:

read

i

write
```

---

Zato CPU ima:

```
atomic CAS instruction
```

---

Na modernim procesorima:

- x86
- ARM

postoji hardverska podrška.

---

# 8. Go Atomic CAS API

Package:

```go
sync/atomic
```

---

Primer:

```go
var value int64


success :=
atomic.CompareAndSwapInt64(
	&value,
	0,
	1,
)
```

---

Ako:

```go
value == 0
```

rezultat:

```go
value = 1
```

---

Ako:

```go
value == 5
```

rezultat:

```go
false
```

---

# 9. Modern Go Atomic Types

Od Go 1.19:

postoje typed atomics.

---

Primer:

```go
var counter atomic.Int64
```

---

Increment:

```go
counter.Add(1)
```

---

Load:

```go
value :=
counter.Load()
```

---

Store:

```go
counter.Store(100)
```

---

Prednosti:

- type safety
- manje grešaka
- čistiji kod

---

# 10. Lock-Free Counter

Mutex verzija:

```go
type Counter struct {

	mu sync.Mutex

	value int64

}
```

---

Lock-free:

```go
type Counter struct {

	value atomic.Int64

}
```

---

Metoda:

```go
func (c *Counter) Inc(){

	c.value.Add(1)

}
```

---

Nema:

```
Lock

Unlock
```

---

# 11. Lock-Free Boolean State

Čest pattern:

```go
var running atomic.Bool
```

---

Start:

```go
running.Store(true)
```

---

Stop:

```go
running.Store(false)
```

---

Check:

```go
if running.Load(){

	work()

}
```

---

Primer:

- shutdown flag
- feature flag
- worker status

---

# 12. Lock-Free Pointer Update

Go podržava:

```go
atomic.Pointer[T]
```

---

Primer:

```go
type Config struct {

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

Dobijamo:

```
atomic snapshot
```

---

# 13. CAS State Machine Pattern

Primer:

stanja:

```
STOPPED

RUNNING

FAILED
```

---

Atomic state:

```go
var state atomic.Int32
```

---

Promena:

```go
CAS(
	STOPPED,
	RUNNING,
)
```

---

Samo jedna goroutine može izvršiti tranziciju.

---

# 14. CAS vs Mutex

## Mutex

Prednosti:

✅ jednostavan

✅ podržava kompleksne operacije

---

Mane:

❌ blocking

❌ contention

---

## CAS

Prednosti:

✅ non-blocking

✅ skalabilan za male operacije

---

Mane:

❌ kompleksniji

❌ retry overhead

---

# 15. Kada CAS Nije Dobar Izbor?

Primer:

```go
bankAccount.Transfer()
```

---

Operacija:

```
remove money

+

add money
```

---

Više promenljivih:

```
balance A

balance B
```

---

CAS postaje komplikovan.

---

Mutex je često bolji.

---

# 16. CAS Performance Problem

Ako mnogo goroutines radi:

```
CAS retry
```

---

Dobijamo:

```
contention loop
```

---

Primer:

```
G1 retry

G2 retry

G3 retry

G4 success
```

---

CPU troši vreme na:

```
failed attempts
```

---

# 17. Senior Pravilo

CAS je odličan za:

```
jednu memorijsku lokaciju

+

jednostavnu tranziciju
```

---

Za kompleksan state:

koristi:

```
mutex

ili

ownership model
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ CAS koncept

✅ CAS korak po korak

✅ retry loop

✅ optimistic concurrency

✅ atomic CAS API

✅ typed atomics

✅ lock-free counter

✅ atomic pointer

✅ state machine pattern

✅ CAS ograničenja

---

# Lock-Free Programming

## Deo #3 — Atomic Operations i Lock-Free Building Blocks

---

# 📚 Sadržaj

- Atomic operacije kao building blocks
- Load i Store operacije
- Add operacije
- Swap operacije
- Compare-And-Swap recap
- `atomic.Value`
- `atomic.Pointer`
- Memory ordering
- Lock-free state patterns
- Praktične komponente

---

# 1. Atomic Operacije kao Building Blocks

Lock-free algoritmi se grade pomoću malog broja osnovnih operacija:

```
Load

Store

Add

Swap

Compare-And-Swap
```

---

Ove operacije omogućavaju:

```
sigurnu izmenu memorije

bez mutex lock-a
```

---

# 2. Atomic Load Operacija

`Load` čita vrednost atomarno.

---

Primer:

```go
var counter atomic.Int64
```

---

Čitanje:

```go
value :=
counter.Load()
```

---

Garantuje:

```
read nije prekinut
```

---

Bez atomica:

```go
value = counter
```

može biti deo:

```
data race
```

---

# 3. Atomic Store Operacija

`Store` upisuje vrednost atomarno.

---

Primer:

```go
var running atomic.Bool
```

---

Postavljanje:

```go
running.Store(true)
```

---

Druga goroutine:

```go
if running.Load(){

	start()

}
```

---

Koristi se za:

- flags
- status
- lifecycle state

---

# 4. Load/Store Pattern

Čest pattern:

```
writer

Store(new state)


reader

Load(state)
```

---

Primer:

```go
type Service struct {

	ready atomic.Bool

}
```

---

Start:

```go
service.ready.Store(true)
```

---

Request:

```go
if service.ready.Load(){

	handle()

}
```

---

Nema:

```
mutex
```

---

# 5. Atomic Add Operacija

Koristi se za:

- counters
- metrics
- statistics

---

Primer:

```go
var requests atomic.Int64
```

---

Increment:

```go
requests.Add(1)
```

---

Decrement:

```go
requests.Add(-1)
```

---

Prednost:

```
više goroutines

+

jedan counter

+

bez lock-a
```

---

# 6. Atomic Swap Operacija

`Swap`:

```
zameni vrednost

i vrati staru
```

---

Primer:

```go
old :=
state.Swap(
	newState,
)
```

---

Pre:

```
state = A
```

---

Posle:

```
state = B
```

---

Rezultat:

```
old == A
```

---

# 7. Swap kao State Transition

Primer:

```go
var status atomic.Int32
```

---

Reset:

```go
previous :=
status.Swap(
	0,
)
```

---

Koristi se za:

- reset counters
- state replacement
- ownership transfer

---

# 8. Compare-And-Swap Rekapitulacija

CAS:

```
conditional update
```

---

Model:

```go
CAS(
	old,
	new,
)
```

---

Primer:

```go
changed :=
state.CompareAndSwap(
	0,
	1,
)
```

---

Znači:

```
ako je state == 0

pređi u state == 1
```

---

# 9. atomic.Value

Pre typed atomics:

```go
atomic.Value
```

je korišćen za:

```
atomic snapshot
```

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
		Port:8080,
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

Prednost:

može čuvati:

```
bilo koji isti tip
```

---

# 10. atomic.Value Pravila

Prvo:

```go
Store()
```

definiše tip.

---

Primer:

```go
config.Store(
	Config{},
)
```

---

Kasnije:

mora biti isti tip:

```go
Config{}
```

---

Nije dozvoljeno:

```go
config.Store(
	"string",
)
```

---

# 11. atomic.Value Configuration Pattern

Primer:

```go
type Config struct {

	Host string

	Port int

}
```

---

Storage:

```go
var currentConfig atomic.Value
```

---

Update:

```go
currentConfig.Store(
	Config{
		Host:"localhost",
		Port:8080,
	},
)
```

---

Read:

```go
cfg :=
currentConfig.Load().
(Config)
```

---

Rezultat:

```
race-free snapshot
```

---

# 12. atomic.Pointer

Modern Go pristup.

---

Primer:

```go
type Config struct {

	Port int

}
```

---

Deklaracija:

```go
var config atomic.Pointer[Config]
```

---

Store:

```go
config.Store(
	&Config{
		Port:8080,
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

Prednosti:

- type safe
- nema cast-a
- efikasan

---

# 13. Atomic Pointer kao Immutable Pattern

Važan koncept:

Ne menjaj objekat.

---

Loše:

```go
cfg.Port = 9000
```

---

Bolje:

```go
newCfg :=
&Config{
	Port:9000,
}


config.Store(newCfg)
```

---

Stari snapshot ostaje:

```
netaknut
```

---

# 14. Memory Ordering

Atomic operacije nisu samo:

```
thread-safe read/write
```

---

One obezbeđuju:

```
memory visibility
```

---

Primer:

Goroutine A:

```go
data = 42

ready.Store(true)
```

---

Goroutine B:

```go
if ready.Load(){

	use(data)

}
```

---

Atomic operacija pravi:

```
visibility boundary
```

---

# 15. Lock-Free Flag Pattern

Primer:

```go
type Worker struct {

	stopped atomic.Bool

}
```

---

Stop:

```go
worker.stopped.Store(true)
```

---

Loop:

```go
for {

	if worker.stopped.Load(){

		return

	}

	work()

}
```

---

Koristi se za:

- shutdown
- cancellation
- feature flags

---

# 16. Lock-Free Metrics Pattern

Primer:

```go
type Metrics struct {

	requests atomic.Int64

	errors atomic.Int64

}
```

---

Request:

```go
metrics.requests.Add(1)
```

---

Error:

```go
metrics.errors.Add(1)
```

---

Nema:

```
mutex contention
```

---

# 17. Atomic Operacije Nisu Dovoljne za Sve

Primer:

```go
balanceA -= 100

balanceB += 100
```

---

Dve operacije:

```
nije jedna atomic jedinica
```

---

Potrebno:

- mutex
- transaction
- ownership

---

# 18. Senior Pravilo

Atomic koristi za:

```
small independent state
```

---

Mutex koristi za:

```
complex invariants
```

---

Channel koristi za:

```
ownership transfer
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ atomic Load

✅ atomic Store

✅ atomic Add

✅ atomic Swap

✅ CAS

✅ atomic.Value

✅ atomic.Pointer

✅ memory ordering

✅ immutable snapshots

✅ lock-free building blocks

---

# Lock-Free Programming

## Deo #4 — Lock-Free Data Structures

---

# 📚 Sadržaj

- Uvod u lock-free data structures
- Zašto su strukture podataka izazov
- Lock-free Stack
- Lock-free Queue
- Michael-Scott Queue koncept
- Linked-list algoritmi
- Atomic pointer veze
- ABA problem uvod
- Praktični dizajn principi

---

# 1. Uvod u Lock-Free Data Structures

Do sada smo koristili atomic operacije za:

- counter
- flag
- snapshot

---

Ali realni sistemi često zahtevaju:

```
kolekcije podataka
```

kao što su:

- stack
- queue
- linked list
- cache
- pool

---

Problem:

Jedna atomic promenljiva nije dovoljna.

---

Primer:

```go
queue.Push(item)

queue.Pop()
```

---

Potrebno je menjati:

```
više memorijskih lokacija
```

---

# 2. Zašto su Strukture Podataka Izazov?

Mutex pristup:

```go
mu.Lock()

queue.Push(item)

mu.Unlock()
```

---

Jednostavno.

---

Lock-free pristup mora rešiti:

- konkurentne izmene
- konzistentnost strukture
- memory reclamation
- ABA problem

---

Zato su lock-free strukture mnogo kompleksnije.

---

# 3. Osnovni Princip

Lock-free strukture često koriste:

```
atomic pointer

+

CAS

+

retry loop
```

---

Opšti obrazac:

```
Read current state

        |

Calculate new state

        |

CAS update

        |

Success?

    yes -> done

    no -> retry
```

---

# 4. Lock-Free Stack

Stack radi po principu:

```
LIFO

Last In First Out
```

---

Primer:

```
Push A

Push B

Push C


Stack:

C

B

A
```

---

Pop:

```
C
```

---

# 5. Stack Node Struktura

Primer:

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

	head atomic.Pointer[Node]

}
```

---

`head` pokazuje na vrh stack-a.

---

# 6. Lock-Free Push Algoritam

Cilj:

dodati novi node na početak.

---

Pre:

```
head

 |

 A
 |
 B
```

---

Novi:

```
C
```

---

Želimo:

```
head

 |

 C
 |
 A
 |
 B
```

---

Algoritam:

1. pročitaj head

2. postavi new.next

3. CAS zameni head

---

# 7. Push Pseudo Kod

```go
func (s *Stack) Push(value int){

	node :=
	&Node{
		value:value,
	}


	for {

		old :=
		s.head.Load()


		node.next =
			old


		if s.head.CompareAndSwap(
			old,
			node,
		){

			return

		}

	}

}
```

---

Ako druga goroutine promeni:

```
CAS fail
```

---

Retry.

---

# 8. Lock-Free Pop Algoritam

Pre:

```
head

 |

C

|

A
```

---

Želimo:

```
head

 |

A
```

---

Koraci:

1. pročitaj head

2. uzmi next

3. CAS pomeri head

---

# 9. Pop Pseudo Kod

```go
func (s *Stack) Pop() *Node {

	for {

		old :=
		s.head.Load()


		if old == nil {

			return nil

		}


		next :=
		old.next


		if s.head.CompareAndSwap(
			old,
			next,
		){

			return old

		}

	}

}
```

---

# 10. Lock-Free Queue

Queue:

```
FIFO

First In First Out
```

---

Primer:

```
Push A

Push B

Push C
```

---

Red:

```
A -> B -> C
```

---

Pop:

```
A
```

---

# 11. Queue Problem

Stack je jednostavniji.

Queue ima:

```
head

+

tail
```

---

Potrebno održavati:

```
front

+

back
```

---

Dve goroutines mogu menjati:

```
različite krajeve
```

---

# 12. Michael-Scott Queue

Jedan od najpoznatijih lock-free queue algoritama.

Autori:

```
Michael & Scott
```

---

Koristi:

- linked list
- atomic pointers
- CAS

---

Struktura:

```
Head

 |

Dummy Node

 |

A

 |

B

 |

Tail
```

---

# 13. Dummy Node Koncept

Queue koristi sentinel node.

---

Zašto?

Da bi:

```
head

i

tail
```

uvek imali validnu referencu.

---

Pojednostavljuje:

- enqueue
- dequeue

---

# 14. Enqueue Operacija

Dodavanje:

```
tail -> new node
```

---

Koraci:

1. pročitaj tail

2. pročitaj tail.next

3. CAS dodaj node

4. pomeri tail

---

Ako neko drugi promeni stanje:

```
retry
```

---

# 15. Dequeue Operacija

Uklanjanje:

```
head -> next
```

---

Koraci:

1. pročitaj head

2. pročitaj tail

3. pročitaj next

4. CAS pomeri head

---

Ako nema elemenata:

```
empty
```

---

# 16. Atomic Pointer Veze

Lock-free strukture često imaju:

```go
atomic.Pointer[Node]
```

---

Primer:

```go
type Node struct {

	next atomic.Pointer[Node]

	value int

}
```

---

Prednost:

promena veze:

```
next pointer
```

je sigurna.

---

# 17. Problem Memory Reclamation

Veliki problem:

Ako uklonimo node:

```go
old.next
```

---

Ko ga briše?

---

Druga goroutine možda još radi:

```go
old.value
```

---

Potrebni sistemi:

- hazard pointers
- epoch reclamation
- garbage collector

---

# 18. Go i Garbage Collector

Go ima prednost:

```
GC automatski upravlja memorijom
```

---

Zato je lakše praviti:

lock-free strukture.

---

Ali:

GC ne rešava:

- logičku konzistentnost
- ABA problem

---

# 19. ABA Problem — Uvod

Jedan od najpoznatijih problema.

---

Scenario:

Početno:

```
pointer = A
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
A -> B

B -> A
```

---

Goroutine 1 vidi:

```
A
```

---

Misli:

```
ništa se nije promenilo
```

---

Ali jeste.

---

# 20. Lock-Free Pravilo

Atomic pointer nije dovoljan.

---

Potrebno razmišljati o:

```
state transition

+

lifetime

+

versioning
```

---

# 21. Kada Koristiti Lock-Free Structures?

Dobri slučajevi:

✅ ekstreman concurrency

✅ veoma kratke operacije

✅ visok performance zahtev

---

Loši slučajevi:

❌ poslovna logika

❌ kompleksni invarianti

❌ timovi bez iskustva

---

# 22. Senior Perspektiva

Većina Go aplikacija neće pisati:

```
custom lock-free queue
```

---

Ali senior developer mora razumeti:

- kako rade
- kada postoje
- koje probleme rešavaju

---

# 📋 Rezime

U ovom delu naučili smo:

✅ lock-free data structures

✅ lock-free stack

✅ CAS retry pattern

✅ lock-free queue

✅ Michael-Scott queue koncept

✅ atomic pointers

✅ memory reclamation

✅ ABA problem uvod

---

# Lock-Free Programming

## Deo #5 — ABA Problem i Memory Reclamation

---

# 📚 Sadržaj

- ABA problem detaljna analiza
- Zašto CAS ne rešava sve
- Primer realnog ABA scenarija
- Version tagging
- Tagged pointers koncept
- Memory reclamation problem
- Hazard pointers
- Epoch-Based Reclamation
- Go Garbage Collector i lock-free strukture
- Production preporuke

---

# 1. ABA Problem — Detaljna Analiza

ABA problem je jedan od najtežih problema u lock-free programiranju.

---

Nastaje kada:

```
vrednost izgleda isto

ali istorija promene nije ista
```

---

Drugim rečima:

```
A

postane

B

pa se vrati

A
```

---

CAS vidi:

```
A == A
```

---

Ali realno:

```
A nije isti A
```

---

# 2. Zašto CAS Ne Rešava Sve?

CAS radi:

```text
compare old value

then swap
```

---

Primer:

```go
CAS(
	oldPointer,
	newPointer,
)
```

---

CAS proverava samo:

```
trenutnu vrednost
```

---

Ne zna:

```
šta se desilo između
```

---

# 3. Klasičan ABA Scenario

Imamo stack:

```
Head

 |

A

 |

B
```

---

## Goroutine 1

Čita:

```
head = A
```

---

Planira:

```
head -> B
```

---

Ali pre CAS-a:

---

## Goroutine 2

Pop:

```
A uklonjen
```

---

Stack:

```
B
```

---

Zatim:

Goroutine 2:

vrati A nazad:

```
A -> B
```

---

Sada:

```
head = A
```

---

## Goroutine 1

Radi:

```text
CAS(A, new)
```

---

CAS uspe.

---

Ali struktura je promenjena između.

---

# 4. Problem sa "Istom" Vrednošću

Za CAS:

```
A == A
```

---

Za algoritam:

```
A1 != A2
```

---

Jer:

```
A1

ima staru istoriju
```

---

dok:

```
A2

ima novu istoriju
```

---

# 5. Rešenje: Version Tagging

Ideja:

ne čuvati samo vrednost.

---

Umesto:

```
pointer
```

čuvamo:

```
pointer

+

version
```

---

Primer:

```
(A,1)
```

---

Posle promene:

```
(A,2)
```

---

Pointer isti.

---

Ali verzija drugačija.

---

# 6. Tagged Pointer Koncept

Normalan CAS:

```text
CAS(pointerA, pointerB)
```

---

Tagged CAS:

```text
CAS(
(pointerA, version1),

(pointerB, version2)
)
```

---

Sada:

ABA postaje:

```
A1

↓

B2

↓

A3
```

---

CAS vidi:

```
A1 != A3
```

---

Promena je detektovana.

---

# 7. Version Counter Primer

Model:

```go
type TaggedPointer struct {

	ptr unsafe.Pointer

	version uint64

}
```

---

Svaka promena:

uvećava:

```go
version++
```

---

Rezultat:

svaka verzija je jedinstvena.

---

# 8. Memory Reclamation Problem

Lock-free strukture imaju još jedan problem:

```
kada obrisati memoriju?
```

---

Primer:

Thread A:

```go
node := head.Load()
```

---

Thread B:

```go
remove(node)
```

---

Pitanje:

da li Thread A još koristi node?

---

# 9. Zašto Free Memorije Može Biti Opasan?

Klasičan primer:

```
G1:

čita node


G2:

ukloni node


G2:

oslobodi memoriju


G1:

pristupa node-u
```

---

Rezultat:

```
use-after-free
```

---

U jezicima bez GC:

ovo je kritičan problem.

---

# 10. Hazard Pointer Koncept

Hazard pointer znači:

```
"ova memorija mi trenutno treba"
```

---

Svaka goroutine registruje:

```
koji objekat koristi
```

---

Primer:

```
G1

hazard -> Node A
```

---

Druga goroutine:

ne sme obrisati:

```
Node A
```

---

dok postoji hazard referenca.

---

# 11. Hazard Pointer Algoritam

Koraci:

1. pročitaj pointer

2. objavi da ga koristiš

3. proveri da se nije promenio

4. koristi objekat

5. ukloni zaštitu

---

Model:

```
Load

↓

Protect

↓

Validate

↓

Use

↓

Release
```

---

# 12. Epoch-Based Reclamation

Drugi pristup:

```
epoch sistem
```

---

Ideja:

memorija se ne briše odmah.

---

Čeka se:

```
sve goroutines pređu određenu epoch granicu
```

---

Primer:

```
Epoch 1

Node uklonjen


Epoch 2

niko ga više ne koristi


Delete
```

---

# 13. Go Garbage Collector i ABA

Go ima GC.

---

Prednost:

nema klasičnog:

```
free()
```

---

GC sprečava:

```
use-after-free
```

---

Ali ne rešava:

```
ABA logiku
```

---

Primer:

pointer može:

```
A

B

A
```

---

GC ne zna:

da je algoritam pogrešan.

---

# 14. Go Lock-Free i GC Interakcija

Kod Go-a često:

nije potrebno implementirati:

- hazard pointers
- manual reclamation

---

Ali potrebno je:

razumeti:

- lifetime objekata
- reference
- atomic pristup

---

# 15. Atomic Pointer i GC

Primer:

```go
var config atomic.Pointer[Config]
```

---

Store:

```go
config.Store(
	&Config{},
)
```

---

GC vidi:

```
reference postoji
```

---

Objekat ostaje živ.

---

# 16. Production Pristup u Go-u

U većini slučajeva:

koristi:

```
atomic.Pointer

+

immutable objects
```

---

Primer:

```
old config

       |
       |

new config snapshot

       |
       |

atomic swap
```

---

Jednostavnije:

od:

```
custom lock-free structure
```

---

# 17. Kada Razmišljati o ABA Problemu?

Ako pišeš:

- lock-free stack
- lock-free queue
- memory pool
- custom scheduler
- runtime komponentu

---

Ne za:

- counters
- flags
- configs

---

# 18. Senior Pravilo

CAS rešava:

```
atomic update
```

---

Ne rešava:

```
istoriju promena

+

lifetime objekta

+

validnost reference
```

---

Zato lock-free algoritmi zahtevaju:

```
CAS

+

versioning

+

memory management
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ ABA problem

✅ zašto CAS nije dovoljan

✅ version tagging

✅ tagged pointers

✅ memory reclamation problem

✅ hazard pointers

✅ epoch reclamation

✅ Go GC ograničenja

✅ production pristup

---

# Lock-Free Programming

## Deo #6 — Lock-Free Programming u Go-u — Production Patterns

---

# 📚 Sadržaj

- Lock-free u realnim Go sistemima
- Kada koristiti lock-free
- Kada koristiti mutex
- Kada koristiti channel
- Atomic vs Mutex vs Channel
- Performance analiza
- Benchmark pristup
- Common mistakes
- Production checklist
- Završni pregled Lock-Free Programming modula

---

# 1. Lock-Free u Realnim Go Sistemima

Lock-free programiranje je najčešće prisutno u:

- runtime sistemima
- high-performance bibliotekama
- observability alatima
- networking komponentama
- scheduler sistemima

---

U standardnim poslovnim aplikacijama:

najčešće se koristi kombinacija:

```
Mutex

+

Channels

+

Atomic primitives
```

---

Cilj nije:

```
ukloniti svaki lock
```

---

Cilj je:

```
izabrati pravi synchronization model
```

---

# 2. Kada Koristiti Lock-Free?

Lock-free je dobar izbor kada imamo:

---

## 2.1 Jednu nezavisnu vrednost

Primer:

```go
var requests atomic.Int64
```

---

Operacija:

```go
requests.Add(1)
```

---

Idealno.

---

## 2.2 State Flag

Primer:

```go
var stopped atomic.Bool
```

---

Upotreba:

```go
stopped.Store(true)
```

---

Čitanje:

```go
if stopped.Load(){

	return

}
```

---

## 2.3 Read-Mostly Configuration

Primer:

```
mnogo čitalaca

malo pisaca
```

---

Pattern:

```
immutable snapshot

+

atomic pointer
```

---

# 3. Kada Ne Koristiti Lock-Free?

Lock-free nije dobar za:

---

## Kompleksan State

Primer:

```go
type Account struct {

	balance int

	history []Transaction

	owner string

}
```

---

Promena zahteva:

više invarianti.

---

Bolje:

```go
sync.Mutex
```

---

# 4. Kada Koristiti Mutex?

Mutex je pravi izbor kada:

imaš:

```
shared mutable state
```

---

Primer:

```go
type Cache struct {

	mu sync.RWMutex

	items map[string]string

}
```

---

Read:

```go
mu.RLock()

value := items[key]

mu.RUnlock()
```

---

Write:

```go
mu.Lock()

items[key]=value

mu.Unlock()
```

---

Prednosti:

✅ jednostavno

✅ čitljivo

✅ sigurno

---

# 5. Kada Koristiti Channel?

Channel je najbolji kada želimo:

```
ownership transfer
```

---

Primer:

worker sistem:

```
Producer

   |

 channel

   |

Worker
```

---

Producer ne menja worker state.

---

Šalje:

```go
Job
```

---

Worker poseduje:

```
obradu
```

---

# 6. Atomic vs Mutex vs Channel

| Primitive | Koristi se za |
|---|---|
| Atomic | Jedna vrednost |
| Mutex | Deljeni mutable state |
| RWMutex | Read-heavy state |
| Channel | Komunikaciju i ownership |
| sync.Once | Jednokratnu inicijalizaciju |

---

# 7. Decision Tree

Pitanje 1:

```
Da li postoji shared mutable state?
```

---

Ne:

```
nema synchronization
```

---

Da:

---

Pitanje 2:

```
Jedna mala vrednost?
```

---

Da:

```
Atomic
```

---

Ne:

---

Pitanje 3:

```
Jedna goroutine može posedovati state?
```

---

Da:

```
Channel
```

---

Ne:

```
Mutex
```

---

# 8. Performance Analiza

Lock-free može smanjiti:

- blocking
- scheduler waiting
- lock contention

---

Ali uvodi:

- CAS retry
- CPU cache invalidaciju
- kompleksnost

---

Nije automatski brže.

---

# 9. Cache Coherence Problem

Moderni CPU imaju:

```
L1 cache

L2 cache

L3 cache
```

---

Ako više goroutines menja isti atomic:

```
cache line bouncing
```

---

Primer:

```
CPU 1

counter


CPU 2

counter
```

---

Cache mora stalno da se sinhronizuje.

---

# 10. False Sharing

Primer:

```go
type Counters struct {

	a atomic.Int64

	b atomic.Int64

}
```

---

Ako su:

```
a

i

b
```

u istoj cache liniji:

može nastati:

```
cache contention
```

---

Rešenje:

padding:

```go
type Counter struct {

	value atomic.Int64

	_ [56]byte

}
```

---

# 11. Benchmark Pristup

Nikada ne pretpostavljati.

---

Meriti:

```bash
go test -bench=.
```

---

Sa race proverom:

```bash
go test -race ./...
```

---

Meriti:

- throughput
- latency
- allocations
- CPU usage

---

# 12. Primer Benchmark Poređenja

Mutex:

```go
func BenchmarkMutex(
	b *testing.B,
){

	var mu sync.Mutex

	var counter int


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

Atomic:

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

Rezultat zavisi od:

- broja goroutines
- CPU arhitekture
- contention nivoa

---

# 13. Common Lock-Free Greške

## Greška 1

"Atomic rešava sve"

---

Netačno.

Atomic rešava:

```
pojedinačne operacije
```

---

Ne rešava:

```
kompleksnu logiku
```

---

# Greška 2

Previše CAS loop-ova.

---

Problem:

```
spin retry
```

---

CPU radi mnogo:

```
failed attempts
```

---

# Greška 3

Ignorisanje lifecycle-a.

---

Primer:

goroutine nema:

```
shutdown signal
```

---

Rezultat:

leak.

---

# 14. Production Checklist

Pre korišćenja lock-free pristupa:

---

## Design

✅ Da li stvarno postoji bottleneck?

✅ Da li je mutex problem?

---

## Correctness

✅ Postoji li race?

✅ Postoji li ABA rizik?

✅ Postoji li jasan ownership?

---

## Performance

✅ Benchmark postoji?

✅ Merena latencija?

✅ Testirano pod opterećenjem?

---

# 15. Finalni Mentalni Model

Zapamti:

```
Atomic

=

jedna memorijska lokacija
```

---

```
Mutex

=

zaštita kompleksnog state-a
```

---

```
Channel

=

komunikacija i ownership
```

---

```
Lock-Free

=

specijalizovan alat
```

---

# 16. Lock-Free Programming Formula

Dobar lock-free dizajn:

```
Minimal Shared State

+

Atomic Primitives

+

Clear Ownership

+

Correct Memory Model

=

Safe Non-Blocking System
```

---

# 17. Završetak Modula

Završili smo:

```
docs/module-4/extras/

└── 04-lock-free-programming.md
```

---

Obrađene teme:

✅ Lock-Free osnove

✅ CAS algoritam

✅ Atomic primitives

✅ Lock-free data structures

✅ ABA problem

✅ Memory reclamation

✅ Production patterns

✅ Performance tuning

---

# 🎯 Senior Lekcija

Najbolji Go concurrency kod nije onaj koji koristi:

```
najnapredniji alat
```

---

Već onaj koji ima:

```
najjednostavniji ispravan model
```

---

Ako je:

```
atomic dovoljan
```

koristi atomic.

---

Ako je:

```
mutex jasniji
```

koristi mutex.

---

Ako:

```
ownership može biti prenet
```

koristi channel.

---

### ➡️ Sledeća lekcija **[**Channels Internals**](05-channels-internals.md)**

Obuhvatiće:

- channel runtime struktura
- hchan
- send/receive operacije
- sudog
- wait queues
- buffered channel internals
- scheduler interakcija
- performance analiza
```