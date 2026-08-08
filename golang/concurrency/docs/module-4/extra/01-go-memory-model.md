# Go Memory Model

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Go Memory Model
>
> Level: Advanced / Senior
>
> Official Reference: https://go.dev/ref/mem

---

# 📚 Sadržaj

- Uvod u Go Memory Model
- Zašto postoji Memory Model?
- CPU, Compiler i Runtime optimizacije
- Problem vidljivosti memorije
- Sequential Consistency
- Happens-Before koncept (uvod)
- Data Race koncept (uvod)
- Synchronization primitives
- Best practices

---

# 1. Uvod u Go Memory Model

Go Memory Model definiše:

> uslove pod kojima jedna goroutine može videti promene koje je napravila druga goroutine.

Drugim rečima:

Ako imamo:

```go
goroutine A
```

koja menja podatak:

```go
x = 10
```

i:

```go
goroutine B
```

koja čita:

```go
fmt.Println(x)
```

pitanje je:

```
Da li goroutine B sigurno vidi novu vrednost?
```

---

Odgovor nije automatski:

```
DA
```

---

# 2. Zašto nam treba Memory Model?

Naivna pretpostavka:

```
Program promeni memoriju

↓

Druga goroutine pročita memoriju
```

Ali moderni sistemi imaju više slojeva:

```
Go Code

↓

Compiler

↓

CPU Instructions

↓

CPU Cache

↓

Main Memory
```

---

Svaki sloj može optimizovati izvršavanje.

---

# 3. Problem Vidljivosti Memorije

Primer:

```go
var ready bool
var data int
```

Goroutine 1:

```go
data = 42
ready = true
```

Goroutine 2:

```go
for !ready {

}

fmt.Println(data)
```

---

Intuitivno očekujemo:

```
42
```

---

Ali bez synchronization-a:

nije garantovano.

---

Zašto?

Compiler ili CPU mogu promeniti redosled:

Original:

```
data = 42

ready = true
```

---

Moguće optimizovano:

```
ready = true

data = 42
```

---

Druga goroutine može videti:

```
ready == true

data == 0
```

---

# 4. Compiler Optimizations

Compiler nije obavezan da izvrši kod tačno tekstualnim redosledom.

Primer:

```go
x++

y++
```

---

Ako nema zavisnosti:

compiler može reorganizovati instrukcije.

---

Cilj:

```
brži program
```

---

Ali kod concurrency-ja:

redosled može biti važan.

---

# 5. CPU Cache Problem

Moderni CPU ima cache hijerarhiju:

```
CPU Core 1

L1 Cache

L2 Cache

L3 Cache

Main Memory
```

i:

```
CPU Core 2

L1 Cache

L2 Cache

L3 Cache

Main Memory
```

---

Primer:

Core 1:

```go
counter = 100
```

promena možda prvo ide u:

```
Core 1 Cache
```

---

Core 2 može još uvek videti:

```
counter = 0
```

---

Zato postoji potreba za:

```
Memory Synchronization
```

---

# 6. Šta Memory Model Garantuje?

Go Memory Model definiše pravila:

Kada:

```
write
```

postaje vidljiv za:

```
read
```

drugoj goroutine.

---

Bez ovih pravila:

svaki concurrent program bi bio nepredvidiv.

---

# 7. Sequential Consistency

Jedan idealni model kaže:

Sve operacije izgledaju kao da se izvršavaju:

```
jednim globalnim redom
```

---

Primer:

Dve goroutines:

```
G1:
x = 1


G2:
y = 1
```

---

Sequential consistency bi dala:

Mogući redovi:

```
x=1

y=1
```

ili:

```
y=1

x=1
```

---

Ali ne:

```
delimično izvršavanje
```

---

Go ne garantuje potpunu sequential consistency za obične memorijske operacije.

---

# 8. Synchronization Primitives

Go daje mehanizme koji stvaraju memorijske garancije.

Najvažniji:

---

## Mutex

```go
mu.Lock()

value++

mu.Unlock()
```

---

Unlock jedne goroutine:

happens-before

sledeći Lock druge goroutine.

---

## Channels

Primer:

```go
ch <- value
```

i:

```go
v := <-ch
```

---

Slanje kroz channel:

happens-before

primanje.

---

## WaitGroup

Primer:

```go
wg.Done()
```

pre:

```go
wg.Wait()
```

---

## Atomic Operations

Primer:

```go
atomic.StoreInt64()
```

i:

```go
atomic.LoadInt64()
```

---

# 9. Data Race — Uvod

Data race postoji kada:

- dve goroutines pristupaju istom podatku
- najmanje jedan pristup je write
- nema synchronization-a

---

Primer:

```go
var counter int


go func(){

	counter++

}()

go func(){

	counter++

}()
```

---

Problem:

obe goroutines menjaju:

```
isti memory location
```

---

Pokretanje:

```bash
go test -race
```

otkriva problem.

---

# 10. Memory Model Mentalni Model

Razmišljaj ovako:

```
Normalna promena

        ❓

Druga goroutine
```

nije dovoljna.

---

Potrebno je:

```
Write

↓

Synchronization Event

↓

Read
```

---

Primer:

```
goroutine A

write data

↓

channel send

↓

channel receive

↓

goroutine B

read data
```

---

Sada postoji garancija.

---

# 11. Najvažnije Pravilo

Nemoj razmišljati:

> "Promenljiva je globalna, svi je vide."

---

Razmišljaj:

> "Koji synchronization mechanism garantuje vidljivost?"

---

# 12. Senior Perspective

Razumevanje Memory Model-a objašnjava:

- zašto Mutex radi
- zašto Channel radi
- zašto atomic postoji
- zašto race detector postoji
- zašto concurrent kod može biti pogrešan i kada "izgleda ispravno"

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Go Memory Model

✅ zašto postoji

✅ odnos compiler / CPU / memory

✅ problem vidljivosti promena

✅ uvod u happens-before

✅ uvod u data race

✅ ulogu synchronization primitive-a

---

# Go Memory Model

## Deo #2 — Memory Ordering i Visibility Guarantees

---

# 📚 Sadržaj

- Memory Ordering koncept
- Program Order vs Execution Order
- Compiler Reordering
- CPU Reordering
- Visibility Guarantees
- Synchronization Boundaries
- Happens-Before uvod
- Šta Go garantuje
- Šta Go ne garantuje

---

# 1. Memory Ordering Koncept

U sekvencijalnom programu očekujemo:

```go
a = 1

b = 2

c = 3
```

redosled:

```
a

↓

b

↓

c
```

---

Kod concurrent programa:

taj redosled nije uvek vidljiv drugim goroutines.

---

Postoje tri nivoa redosleda:

```
Source Code Order

        ↓

Compiler Order

        ↓

CPU Execution Order
```

---

# 2. Program Order vs Execution Order

## Program Order

Redosled koji developer napiše.

Primer:

```go
x = 1

y = 2
```

---

## Execution Order

Redosled kojim se instrukcije stvarno izvršavaju.

Može biti:

```
y = 2

x = 1
```

ako nema zavisnosti.

---

Za single-thread program:

ovo često nije problem.

---

Za concurrent program:

može promeniti rezultat.

---

# 3. Compiler Reordering

Compiler optimizuje kod.

Cilj:

```
isti rezultat

+

brže izvršavanje
```

---

Primer:

```go
func calculate(){

	a := 10

	b := 20

	use(a,b)

}
```

---

Compiler može promeniti:

- raspored instrukcija
- ukloniti nepotrebne operacije
- držati vrednosti u registrima

---

Kod bez concurrency-ja:

sigurno.

---

Kod shared memory:

može biti problem.

---

# 4. Primer Compiler Reordering Problema

Kod:

```go
var initialized bool
var value int
```

Writer:

```go
value = 100

initialized = true
```

Reader:

```go
if initialized {

	fmt.Println(value)

}
```

---

Developer očekuje:

```
initialized == true

value == 100
```

---

Ali bez synchronization:

moguće je:

```
initialized == true

value == 0
```

---

Zašto?

Compiler nije dužan da čuva ovaj redosled između goroutines.

---

# 5. CPU Reordering

Moderni CPU izvršava instrukcije agresivno.

Koristi:

- pipelines
- out-of-order execution
- store buffers
- cache coherence

---

Primer:

CPU vidi:

```text
Instruction A

Instruction B
```

---

Može izvršiti:

```text
B

A
```

ako smatra da je bezbedno.

---

Za single thread:

CPU održava iluziju ispravnog redosleda.

---

Za više thread-ova:

potrebni su memory barriers.

---

# 6. Memory Barrier Koncept

Memory barrier je mehanizam koji kaže:

```
Ne pomeraj operacije preko ove granice.
```

---

Primer:

```
Write data

     |
     |
 Memory Barrier

     |
     |

Read data
```

---

U Go-u developer uglavnom ne koristi direktno barrier.

Umesto toga koristi:

- Mutex
- Channels
- Atomic
- WaitGroup

---

# 7. Synchronization Boundary

Synchronization događaj predstavlja tačku gde Go garantuje vidljivost.

---

Primer:

```go
data = 42

ch <- true
```

---

Druga goroutine:

```go
<-ch

fmt.Println(data)
```

---

Redosled:

```
write data

↓

channel send

↓

channel receive

↓

read data
```

---

Channel predstavlja:

```
visibility boundary
```

---

# 8. Mutex Visibility Guarantee

Primer:

```go
var counter int

var mu sync.Mutex
```

Writer:

```go
mu.Lock()

counter = 10

mu.Unlock()
```

Reader:

```go
mu.Lock()

fmt.Println(counter)

mu.Unlock()
```

---

Garancija:

```
Unlock()

        happens-before

Lock()
```

---

Reader vidi:

```
counter == 10
```

---

# 9. Atomic Visibility Guarantee

Primer:

```go
atomic.StoreInt64(
	&value,
	100,
)
```

---

Druga goroutine:

```go
v :=
atomic.LoadInt64(
	&value,
)
```

---

Atomic operacije daju:

- atomicity
- ordering guarantees

---

Ali:

atomic nije zamena za kompleksnu sinhronizaciju.

---

# 10. Šta Go Garantuje?

Go garantuje:

---

## 1. Single Goroutine Order

Unutar jedne goroutine:

```go
A()

B()

C()
```

izvršavanje ima definisan redosled.

---

## 2. Synchronization Ordering

Preko:

- channel
- mutex
- atomic
- WaitGroup

---

## 3. Data Race Detection

Ako postoji race:

```bash
go test -race
```

može ga otkriti.

---

# 11. Šta Go NE Garantuje?

Bez synchronization:

Go ne garantuje:

---

## Vidljivost

Primer:

```go
x = 10
```

druga goroutine ne mora odmah videti.

---

## Redosled

Primer:

```go
a = 1

b = 2
```

druga goroutine ne mora videti tim redom.

---

## Atomicnost kompleksnih operacija

Primer:

```go
counter++
```

nije jedna operacija.

---

Interno:

```
READ

+

ADD

+

WRITE
```

---

# 12. Common Developer Mistake

Kod:

```go
done := false


go func(){

	doWork()

	done = true

}()


for !done {

}
```

---

Izgleda jednostavno.

Ali:

problemi:

- data race
- compiler optimizations
- CPU visibility

---

Ispravno:

```go
done := make(chan struct{})


go func(){

	doWork()

	close(done)

}()

<-done
```

---

# 13. Mentalni Model

Nemoj razmišljati:

```
RAM je zajednički
```

---

Bolje:

```
Svaka goroutine ima svoj pogled.

Synchronization povezuje te poglede.
```

---

# 14. Senior Pravilo

Ako dve goroutines dele podatak:

pitaj:

```
Koji synchronization mechanism garantuje visibility?
```

Ako nema odgovora:

kod verovatno nije bezbedan.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ memory ordering

✅ program vs execution order

✅ compiler reordering

✅ CPU reordering

✅ memory barriers

✅ synchronization boundaries

✅ visibility guarantees

✅ šta Go garantuje

✅ šta Go ne garantuje

---

# Go Memory Model

## Deo #3 — Happens-Before Relationship

---

# 📚 Sadržaj

- Formalni model izvršavanja
- Sequenced Before
- Synchronized Before
- Happens-Before definicija
- Transitive ordering
- Memory visibility kroz Happens-Before
- Praktični primeri
- Zašto je Happens-Before važan

---

# 1. Formalni Model Go Memory Model-a

Go Memory Model koristi nekoliko ključnih pojmova:

```
Sequenced Before

+

Synchronized Before

=

Happens Before
```

---

Ova relacija određuje:

> Kada je jedna memorijska operacija garantovano vidljiva drugoj goroutine.

---

# 2. Sequenced Before

`Sequenced Before` opisuje redosled operacija unutar jedne goroutine.

Primer:

```go
func worker() {

	a := 10

	b := 20

	fmt.Println(a + b)

}
```

---

Redosled:

```
a := 10

↓

b := 20

↓

Print
```

---

Ovo je prirodan redosled jedne goroutine.

---

# 3. Synchronized Before

`Synchronized Before` nastaje preko synchronization mehanizma.

Primer:

Channel:

```go
ch <- value
```

i:

```go
v := <-ch
```

---

Postoji odnos:

```
send

↓

receive
```

---

Receive vidi sve promene koje su bile pre send operacije.

---

# 4. Happens-Before Definicija

Formalno:

Ako:

```
A happens-before B
```

onda:

```
B može videti efekte A
```

---

Drugim rečima:

Ako goroutine A uradi:

```go
x = 100
```

i postoji:

```
A

happens-before

B
```

onda B može bezbedno čitati:

```go
x
```

kao:

```
100
```

---

# 5. Kombinovanje Pravila

Happens-before nastaje kombinacijom:

```
Sequenced Before

+

Synchronized Before
```

---

Primer:

```go
x = 10

ch <- true
```

---

Druga goroutine:

```go
<-ch

print(x)
```

---

Imamo:

## Goroutine 1

```
x = 10

↓

send
```

---

## Channel

```
send

↓

receive
```

---

## Goroutine 2

```
receive

↓

read x
```

---

Kompletan lanac:

```
x = 10

↓

send

↓

receive

↓

read x
```

---

Zato:

```
write x

happens-before

read x
```

---

# 6. Transitive Ordering

Jedna od najvažnijih osobina.

Ako:

```
A happens-before B
```

i:

```
B happens-before C
```

onda:

```
A happens-before C
```

---

Primer:

```
Goroutine 1

write data

↓

channel send


Goroutine 2

channel receive

↓

write result

↓

channel send


Goroutine 3

channel receive

↓

read result
```

---

Lanac:

```
data write

↓

send

↓

receive

↓

result write

↓

send

↓

receive

↓

read
```

---

Sve promene putuju kroz lanac.

---

# 7. Mutex Happens-Before

Primer:

```go
var mu sync.Mutex

var value int
```

Writer:

```go
mu.Lock()

value = 42

mu.Unlock()
```

Reader:

```go
mu.Lock()

fmt.Println(value)

mu.Unlock()
```

---

Relacija:

```
Unlock()

↓

Lock()
```

---

Rezultat:

Reader vidi:

```text
value == 42
```

---

# 8. WaitGroup Happens-Before

Primer:

```go
var wg sync.WaitGroup

var result int
```

Worker:

```go
result = calculate()

wg.Done()
```

Main:

```go
wg.Wait()

fmt.Println(result)
```

---

Redosled:

```
write result

↓

Done()

↓

Wait()

↓

read result
```

---

`Wait()` garantuje završetak worker-a.

---

# 9. Once Happens-Before

Primer:

```go
var once sync.Once

var config Config
```

Initialization:

```go
once.Do(func(){

	config = loadConfig()

})
```

---

Sve goroutines koje pozovu:

```go
once.Do(...)
```

vide završenu inicijalizaciju.

---

# 10. Atomic Happens-Before

Primer:

```go
atomic.StoreInt64(
	&ready,
	1,
)
```

Reader:

```go
if atomic.LoadInt64(&ready)==1 {

	useData()

}
```

---

Atomic operacije omogućavaju koordinaciju.

---

Ali pažnja:

```go
atomic.StoreInt64(&ready,1)
```

ne znači automatski da su svi drugi podaci bezbedni ako nisu pravilno povezani.

---

# 11. Primer Bez Happens-Before

Loš kod:

```go
var data int

var ready bool


go func(){

	data = 100

	ready = true

}()


go func(){

	if ready {

		fmt.Println(data)

	}

}()
```

---

Ne postoji:

```
write

↓

synchronization

↓

read
```

---

Problem:

```
data race
```

---

# 12. Zašto je Happens-Before Važan?

Bez njega ne možemo dokazati:

- ispravnost programa,
- vidljivost podataka,
- redosled događaja.

---

Senior concurrency analiza se često svodi na pitanje:

```
Postoji li Happens-Before relacija?
```

---

Ako postoji:

```
verovatno bezbedno
```

Ako ne postoji:

```
potencijalni bug
```

---

# 13. Happens-Before kao Graph

Možemo zamisliti program kao graf:

```
        write A

           |
           ↓

       channel send

           |
           ↓

       channel receive

           |
           ↓

        read A
```

---

Ako postoji put:

```
write → read
```

postoji ordering.

---

Ako nema puta:

```
write     read

  ?        ?
```

nema garancije.

---

# 14. Senior Mentalni Model

Concurrency problem nije:

> "Imam dve goroutines."

---

Pravi problem je:

> "Mogu li dokazati redosled memorijskih operacija?"

---

Memory Model daje alat:

```
Happens-Before
```

za taj dokaz.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Sequenced Before

✅ Synchronized Before

✅ Happens-Before

✅ transitive ordering

✅ channel ordering

✅ mutex ordering

✅ WaitGroup ordering

✅ atomic ordering

✅ kako dokazati memory visibility

---

# Go Memory Model

## Deo #4 — Synchronization Primitives i njihove Guarantees

---

# 📚 Sadržaj

- Uvod u synchronization guarantees
- Channel memory guarantees
- Mutex memory guarantees
- RWMutex memory guarantees
- WaitGroup memory guarantees
- Once memory guarantees
- Atomic memory guarantees
- Poređenje mehanizama
- Izbor pravog mehanizma

---

# 1. Uvod u Synchronization Guarantees

U prethodnim delovima videli smo:

```
Write

↓

Synchronization

↓

Read
```

---

Synchronization mehanizam stvara:

```
Happens-Before relationship
```

---

Go ima nekoliko glavnih mehanizama:

```text
Channels

Mutex

RWMutex

WaitGroup

Once

Atomic
```

---

Svaki ima drugačiji model korišćenja.

---

# 2. Channel Memory Guarantees

Channels su jedan od najvažnijih Go mehanizama.

---

Primer:

```go
var data int

ch := make(chan struct{})
```

Writer:

```go
data = 100

ch <- struct{}{}
```

Reader:

```go
<-ch

fmt.Println(data)
```

---

Garancija:

```
data = 100

↓

channel send

↓

channel receive

↓

read data
```

---

Dakle:

```
send happens-before receive
```

---

# 3. Unbuffered Channel Guarantee

Kod:

```go
ch := make(chan int)
```

---

Send:

```go
ch <- value
```

čeka receiver.

---

Relacija:

```
send

⇄

receive
```

---

Ovo daje veoma jaku koordinaciju.

---

Tipična upotreba:

- signalizacija
- ownership transfer
- synchronization point

---

# 4. Buffered Channel Guarantee

Kod:

```go
ch := make(chan int, 10)
```

---

Send ne mora odmah čekati:

```go
ch <- value
```

---

Ali:

poruka u channel-u i dalje nosi:

```
happens-before guarantee
```

---

Primer:

```go
data = 42

ch <- true
```

---

Receive:

```go
<-ch

fmt.Println(data)
```

---

Bezbedno.

---

# 5. Channel Close Guarantee

Poseban slučaj:

```go
close(ch)
```

---

Garancija:

Operacija pre:

```go
close(ch)
```

vidljiva je nakon:

```go
value, ok := <-ch
```

gde je:

```go
ok == false
```

---

Primer:

```go
result = "done"

close(done)
```

---

Consumer:

```go
<-done

fmt.Println(result)
```

---

Vidi:

```
result == "done"
```

---

# 6. Mutex Memory Guarantees

Mutex štiti shared memory.

---

Primer:

```go
var mu sync.Mutex

var counter int
```

Writer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Reader:

```go
mu.Lock()

fmt.Println(counter)

mu.Unlock()
```

---

Garancija:

```
Unlock

↓

Lock
```

---

Sve promene pre `Unlock()` vidljive su posle `Lock()`.

---

# 7. RWMutex Guarantees

`RWMutex` ima dva tipa zaključavanja:

```go
Lock()

Unlock()
```

i:

```go
RLock()

RUnlock()
```

---

Writer:

```go
mu.Lock()

data = value

mu.Unlock()
```

---

Reader:

```go
mu.RLock()

read(data)

mu.RUnlock()
```

---

Garancija:

Writer:

```
Unlock

↓

sledeći Lock/RLock
```

---

# 8. Mutex vs Channel Memory Model

## Mutex

Model:

```
Shared Memory

+

Protection
```

---

Primer:

```text
goroutine
    |
    |
 mutex
    |
    |
 shared data
```

---

## Channel

Model:

```
Communication

+

Synchronization
```

---

Primer:

```
goroutine

↓

channel

↓

goroutine
```

---

Praktično pravilo:

Koristi Mutex kada deliš state.

Koristi Channel kada prenosiš ownership.

---

# 9. WaitGroup Memory Guarantees

WaitGroup koordinira završetak goroutines.

---

Primer:

```go
var wg sync.WaitGroup

var result int
```

Worker:

```go
result = calculate()

wg.Done()
```

Main:

```go
wg.Wait()

fmt.Println(result)
```

---

Redosled:

```
write result

↓

Done()

↓

Wait()

↓

read result
```

---

`Wait()` garantuje završetak.

---

# 10. sync.Once Guarantees

`sync.Once` garantuje:

jedna inicijalizacija.

---

Primer:

```go
var once sync.Once

var config Config
```

---

Kod:

```go
once.Do(func(){

	config = load()

})
```

---

Ostale goroutines vide:

```
potpuno inicijalizovan config
```

---

Tipična upotreba:

- singleton
- lazy initialization
- global config

---

# 11. Atomic Memory Guarantees

Atomic operacije rade direktno nad memorijom.

Primer:

```go
atomic.StoreInt64(
	&counter,
	100,
)
```

Reader:

```go
v :=
atomic.LoadInt64(
	&counter,
)
```

---

Garancije:

## Atomicity

Operacija se izvršava nedeljivo.

---

## Visibility

Druge goroutines vide promenu.

---

## Ordering

Operacije imaju definisana pravila redosleda.

---

# 12. Atomic Nije Zamena za Mutex

Primer:

```go
counter++
```

može zahtevati:

```
read

+

modify

+

write
```

---

Atomic može rešiti:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Ali ne rešava:

```go
account.balance += transaction.amount
```

---

Za kompleksan state:

koristi Mutex.

---

# 13. Poređenje Synchronization Mehanizama

| Mehanizam | Glavna namena | Memory Guarantee |
|---|---|---|
| Channel | komunikacija | send → receive |
| close(channel) | signal | close → receive |
| Mutex | shared state | Unlock → Lock |
| RWMutex | read/write state | Unlock → Lock/RLock |
| WaitGroup | lifecycle | Done → Wait |
| Once | initialization | Do → Do |
| Atomic | simple values | Store → Load |

---

# 14. Kako Izabrati Pravi Mehanizam?

Pitanje:

## Prenosim podatke?

Koristi:

```
Channel
```

---

## Delim memoriju?

Koristi:

```
Mutex
```

---

## Brojač / flag?

Koristi:

```
Atomic
```

---

## Čekam završetak?

Koristi:

```
WaitGroup
```

---

## Jednokratna inicijalizacija?

Koristi:

```
sync.Once
```

---

# 15. Senior Pravilo

Najčešća greška:

```
Dodaj Mutex svuda
```

---

Bolje:

Prvo definiši:

```
Ownership model
```

zatim:

```
Communication model
```

zatim:

```
Synchronization
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ channel guarantees

✅ close guarantees

✅ mutex ordering

✅ RWMutex ordering

✅ WaitGroup guarantees

✅ Once guarantees

✅ Atomic guarantees

✅ razliku između mehanizama

---

# Go Memory Model

## Deo #5 — Practical Examples i Common Pitfalls

---

# 📚 Sadržaj

- Uvod u praktične probleme
- Broken synchronization primeri
- Data visibility problemi
- Race condition primeri
- Loop variable problem
- Unsafe publication
- Double-check locking problem
- Ispravni obrasci rešavanja
- Senior debugging pristup

---

# 1. Uvod u Praktične Probleme

Go concurrency bugovi su često opasni zato što:

- kod izgleda ispravno,
- radi lokalno,
- prolazi testove,
- pada samo pod opterećenjem.

---

Najčešći razlog:

```
Nema Happens-Before relacije
```

---

Pitanje koje uvek postavljamo:

```
Ko garantuje da ova vrednost postoji u drugoj goroutine?
```

---

# 2. Problem: Shared Variable Without Synchronization

Primer:

```go
package main

import (
	"fmt"
	"time"
)

var counter int

func main() {

	go func() {
		counter++
	}()

	go func() {
		counter++
	}()

	time.Sleep(time.Second)

	fmt.Println(counter)

}
```

---

Developer očekuje:

```
2
```

---

Ali moguće:

```
1
```

ili drugačiji rezultat.

---

Zašto?

`counter++` nije jedna operacija.

Interno:

```
READ counter

+

ADD 1

+

WRITE counter
```

---

Mogući scenario:

```
G1              G2

READ 0

                READ 0

ADD 1

                ADD 1

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

Rešenje:

```go
var mu sync.Mutex

mu.Lock()

counter++

mu.Unlock()
```

---

ili:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

# 3. Problem: Visibility Without Synchronization

Kod:

```go
var ready bool
var data int
```

Writer:

```go
go func(){

	data = 100

	ready = true

}()
```

Reader:

```go
go func(){

	for !ready {

	}

	fmt.Println(data)

}()
```

---

Izgleda logično.

Ali nema:

```
write

↓

synchronization

↓

read
```

---

Problemi:

- compiler optimizacija
- CPU cache
- data race

---

Rešenje:

Channel:

```go
done := make(chan struct{})
```

Writer:

```go
data = 100

close(done)
```

Reader:

```go
<-done

fmt.Println(data)
```

---

# 4. Problem: Loop Variable Capture

Vrlo čest goroutine bug.

Kod:

```go
for i := 0; i < 5; i++ {

	go func(){

		fmt.Println(i)

	}()

}
```

---

Očekivanje:

```
0
1
2
3
4
```

---

Ali rezultat može biti:

```
5
5
5
5
5
```

---

Zašto?

Sve goroutines koriste istu promenljivu:

```
i
```

---

Ispravno:

```go
for i := 0; i < 5; i++ {

	i := i

	go func(){

		fmt.Println(i)

	}()

}
```

---

ili:

```go
for i := 0; i < 5; i++ {

	go func(v int){

		fmt.Println(v)

	}(i)

}
```

---

# 5. Problem: Unsafe Publication

Primer:

```go
type Config struct {

	URL string

}
```

---

Globalna promenljiva:

```go
var config *Config
```

---

Initialization:

```go
config = &Config{
	URL:"localhost",
}
```

---

Reader:

```go
fmt.Println(config.URL)
```

---

Problem:

Druga goroutine može videti:

```
config != nil
```

ali objekat nije potpuno inicijalizovan.

---

Rešenja:

## sync.Once

```go
once.Do(func(){

	config = loadConfig()

})
```

---

ili:

## Channel synchronization

```go
ready <- config
```

---

# 6. Problem: Double-Checked Locking

Pokušaj optimizacije:

```go
if instance == nil {

	mu.Lock()

	if instance == nil {

		instance = create()

	}

	mu.Unlock()

}
```

---

U mnogim jezicima problematično.

Zašto?

Jer:

```
pointer assignment
```

i:

```
object initialization
```

nisu nužno isti trenutak.

---

U Go-u:

najbolje koristiti:

```go
sync.Once
```

---

Primer:

```go
once.Do(func(){

	instance = create()

})
```

---

# 7. Problem: Closing Channel From Wrong Side

Loš dizajn:

```go
func worker(ch chan int){

	close(ch)

}
```

---

Problem:

Worker ne poseduje channel.

---

Pravilo:

```
Sender owns closing
```

---

Primer:

```go
func producer() <-chan int {

	ch := make(chan int)

	go func(){

		defer close(ch)

		ch <- 1

	}()

	return ch
}
```

---

Ownership je jasan.

---

# 8. Problem: Goroutine Leak

Primer:

```go
func process(ch <-chan int){

	go func(){

		for {

			value := <-ch

			fmt.Println(value)

		}

	}()

}
```

---

Ako niko više ne šalje:

goroutine ostaje zauvek.

---

Posledica:

```
memory leak

+

resource leak
```

---

Rešenje:

Context cancellation:

```go
select {

case value := <-ch:

case <-ctx.Done():

	return

}
```

---

# 9. Problem: Copying Mutex

Loše:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Ako kopiraš:

```go
copy := original
```

kopiraš i mutex.

---

Rezultat:

nepredvidivo ponašanje.

---

Pravilo:

```
Do not copy sync primitives
```

---

# 10. Problem: Misusing Atomic

Primer:

```go
atomic.StoreInt64(
	&balance,
	100,
)
```

---

Ali operacija:

```go
balance += payment
```

zahteva:

```
read

+

calculate

+

write
```

---

Atomic nije dovoljan.

---

Koristi:

```go
mu.Lock()

balance += payment

mu.Unlock()
```

---

# 11. Debugging Concurrency Bugova

Senior pristup:

---

## 1. Race Detector

```bash
go test -race ./...
```

---

## 2. Goroutine Dump

```go
runtime.NumGoroutine()
```

---

## 3. Profiling

Koristi:

```
pprof
```

---

## 4. Trace

Koristi:

```
go tool trace
```

---

# 12. Mentalni Model za Debug

Kada vidiš bug:

Nemoj pitati:

```
Zašto je vrednost pogrešna?
```

---

Pitaj:

```
Koji događaj garantuje visibility?
```

---

Ako odgovor ne postoji:

verovatno postoji concurrency problem.

---

# 13. Senior Checklist

Pre nego što pustiš concurrent kod:

✅ Svaka goroutine ima owner-a

✅ Svaka goroutine ima shutdown mehanizam

✅ Shared state ima synchronization

✅ Channel ownership je jasan

✅ Nema kopiranja mutex-a

✅ Race detector prolazi

✅ Nema goroutine leak-a

---

# 📋 Rezime

U ovom delu naučili smo:

✅ realne concurrency bugove

✅ shared memory probleme

✅ visibility probleme

✅ loop capture bug

✅ unsafe publication

✅ double-check locking problem

✅ channel ownership

✅ goroutine leak

✅ atomic ograničenja

---

# Go Memory Model

## Deo #6 — Advanced Cases i Design Principles

---

# 📚 Sadržaj

- Zero values i concurrency
- Package initialization ordering
- Interface values i memory visibility
- Slices u concurrent okruženju
- Maps u concurrent okruženju
- Immutable data patterns
- Copy-on-write pristup
- Thread-safe API dizajn
- Senior concurrency principi

---

# 1. Zero Values i Concurrency

Go ima veoma važan princip:

> Zero value treba da bude korisna.

Primer:

```go
var mu sync.Mutex
```

Odmah je spreman za korišćenje:

```go
mu.Lock()

// critical section

mu.Unlock()
```

---

Isto važi za:

```go
var wg sync.WaitGroup
```

i:

```go
var once sync.Once
```

---

Prednost:

nema potrebe za:

```go
NewMutex()
```

ili:

```go
Initialize()
```

---

# 2. Zero Value Ne Znači Thread Safe

Česta greška:

```go
type Counter struct {

	value int

}
```

---

Instanca:

```go
var c Counter
```

ima validan zero value.

Ali:

```go
go c.Increment()

go c.Increment()
```

nije bezbedno.

---

Potrebno:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Zero value omogućava:

```
lakšu inicijalizaciju
```

ali ne garantuje:

```
concurrency safety
```

---

# 3. Package Initialization Ordering

Go garantuje redosled inicijalizacije paketa.

Primer:

```go
var config = loadConfig()
```

---

Redosled:

```
import dependencies

        ↓

initialize variables

        ↓

init()

        ↓

main()
```

---

Ovo je:

```
single-thread initialization
```

---

Zbog toga je package-level initialization često bezbedan.

---

Primer:

```go
var defaultConfig = Config{

	Timeout: 10,

}
```

---

Nakon:

```go
main()
```

sve goroutines vide:

```
potpuno inicijalizovanu vrednost
```

---

# 4. Interface Values i Memory Visibility

Interface u Go-u nije samo vrednost.

Interno:

```
interface

+

type information

+

data pointer
```

---

Primer:

```go
var value interface{}
```

---

Assignment:

```go
value = User{

	Name:"Mark",

}
```

---

Concurrent read:

```go
fmt.Println(value)
```

---

Bez synchronization:

problem.

---

Zašto?

Interface promena uključuje:

- promenu type pointer-a
- promenu data pointer-a

---

To nisu nužno jedna nedeljiva operacija.

---

Rešenje:

Mutex:

```go
mu.Lock()

value = newValue

mu.Unlock()
```

---

ili:

Atomic pointer:

```go
atomic.Pointer[T]
```

---

# 5. Slices i Concurrency

Slice je struktura:

```
Slice Header

|

├── Pointer

├── Length

└── Capacity
```

---

Primer:

```go
items := []int{}
```

---

Problem:

goroutine 1:

```go
items = append(
	items,
	1,
)
```

---

goroutine 2:

```go
fmt.Println(items)
```

---

Append menja:

- pointer
- length
- underlying array

---

Zato:

nije thread safe.

---

# 6. Bezbedni Slice Pattern

## Pattern 1 — Ownership Transfer

Jedna goroutine poseduje slice.

```
Producer

↓

Consumer
```

---

## Pattern 2 — Copy

```go
copyItems :=
append(
	[]int(nil),
	items...,
)
```

---

## Pattern 3 — Mutex

```go
mu.Lock()

items = append(items, value)

mu.Unlock()
```

---

# 7. Maps i Concurrency

Go map nije thread-safe.

---

Loše:

```go
m := make(map[string]int)
```

---

Goroutine 1:

```go
m["a"] = 1
```

---

Goroutine 2:

```go
fmt.Println(m["a"])
```

---

Rezultat:

runtime panic:

```
concurrent map read and map write
```

---

# 8. Thread-Safe Map Pattern

## sync.Mutex

```go
type SafeMap struct {

	mu sync.Mutex

	data map[string]int

}
```

---

## sync.RWMutex

Za mnogo čitanja:

```go
mu.RLock()

value := data[key]

mu.RUnlock()
```

---

## sync.Map

Za specifične slučajeve:

```go
var m sync.Map
```

---

# 9. Immutable Data Pattern

Jedan od najjačih concurrency principa:

```
Ne deli memoriju.

Deli kopije.
```

---

Primer:

```go
type Config struct {

	Timeout int

}
```

---

Umesto:

```go
config.Timeout = 20
```

---

Koristi:

```go
newConfig := Config{

	Timeout:20,

}
```

---

Stari objekat ostaje nepromenjen.

---

# 10. Copy-On-Write Pattern

Ideja:

Čitanje:

```
bez lock-a
```

Pisanje:

```
kreiraj novu verziju
```

---

Primer:

```
Config v1

Readers

        ↓

Config v2

Writer
```

---

Često se koristi za:

- configuration
- routing tables
- feature flags

---

# 11. Atomic Pointer Pattern

Go podržava:

```go
atomic.Pointer[T]
```

---

Primer:

```go
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

read bez mutex-a.

---

# 12. Thread-Safe API Dizajn

Dobar API jasno definiše ownership.

---

Loše:

```go
func GetUsers() []User
```

---

Ko poseduje slice?

Ko sme da menja?

---

Bolje:

```go
func GetUsers() []User {

	return clone(users)

}
```

---

---

# 13. Concurrency Contract

Svaki public API treba da definiše:

## Ownership

Ko poseduje podatak?

---

## Mutation

Da li je dozvoljena promena?

---

## Synchronization

Ko kontroliše pristup?

---

## Lifetime

Kada objekat može biti uništen?

---

# 14. Senior Pravila

## Pravilo 1

Preferiraj:

```
share memory by communicating
```

umesto:

```
communicate by sharing memory
```

---

## Pravilo 2

Immutable objekti smanjuju probleme.

---

## Pravilo 3

Ownership je važniji od lock-a.

---

## Pravilo 4

Ako API skriva concurrency model:

imaš potencijalni problem.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zero values

✅ package initialization

✅ interface visibility

✅ slice concurrency

✅ map concurrency

✅ immutable patterns

✅ copy-on-write

✅ atomic pointer

✅ thread-safe API dizajn

---

# 🎯 Kraj fajla

`01-go-memory-model.md` je završio uvod u:

- Memory Visibility
- Ordering
- Happens-Before
- Synchronization Guarantees
- Practical Concurrency Problems
- Advanced Design Patterns

---

### ➡️ Sledeća lekcija **[**Atomic Operations Patterns**](02-atomic-operations-patterns.md)**

Obuhvatiće:

- atomic primitives
- Compare-And-Swap
- lock-free counters
- atomic state machines
- performance tradeoffs
- production patterns
