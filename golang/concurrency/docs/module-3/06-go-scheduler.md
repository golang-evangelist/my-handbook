# Go Scheduler — Uvod u Go Runtime Scheduler

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- Šta je scheduler
- Zašto Go ima sopstveni scheduler
- Odnos Goroutines i OS Threads
- Concurrency pre Go modela
- Problem klasičnih thread modela
- Go pristup konkurentnosti
- Mentalni model Go Scheduler-a

---

# Uvod

Do sada smo naučili:

```go
go function()
```

kreira novu Goroutine.

---

Primer:

```go
go process()
```

---

Ali postavlja se pitanje:

> Ko izvršava tu Goroutine?

---

Odgovor:

```text
Go Scheduler
```

---

Go Scheduler je deo:

```text
Go Runtime-a
```

---

Njegov posao je:

- kreiranje rasporeda izvršavanja,
- dodeljivanje Goroutines,
- upravljanje OS thread-ovima,
- balansiranje rada.

---

# Šta je scheduler?

Scheduler je komponenta operativnog sistema ili runtime-a koja odlučuje:

```
koji zadatak

kada

na kom procesoru

će se izvršavati
```

---

Primer:

Imamo:

```
1000 Goroutines
```

---

Ali:

```
8 CPU jezgara
```

---

Neko mora odlučiti:

```
koje Goroutines trenutno rade
```

---

To radi scheduler.

---

# Tradicionalni model: OS Threads

Pre Go-a, mnogi jezici su koristili:

```
1 task

=

1 OS Thread
```

---

Primer:

```
Thread 1 → Request A

Thread 2 → Request B

Thread 3 → Request C
```

---

OS scheduler upravlja thread-ovima.

---

# Problem OS Thread modela

OS thread je relativno težak resurs.

---

Svaki thread ima:

- stack memoriju,
- kernel strukture,
- context switch troškove.

---

Primer:

```
100000 korisnika

↓

100000 thread-ova
```

---

Problem:

- memorija,
- scheduling overhead,
- sporiji context switch.

---

# Go pristup

Go uvodi:

```
Goroutine
```

---

Goroutine je:

```
laganiji izvršni kontekst
```

---

Model:

```
Goroutines

      ↓

Go Scheduler

      ↓

OS Threads

      ↓

CPU
```

---

---

# Goroutine vs OS Thread

| | Goroutine | OS Thread |
|-|-|-|
| Kreira | Go runtime | OS |
| Memorija | mala | veća |
| Broj | milioni | hiljade |
| Scheduling | Go runtime | OS |
| Context switch | jeftiniji | skuplji |

---

# Primer

Možemo imati:

```go
for i:=0;i<100000;i++{

	go task(i)

}
```

---

Dobijamo:

```
100000 Goroutines
```

---

Ali ne:

```
100000 OS Threads
```

---

Go runtime ih raspoređuje.

---

# Zašto Go ima svoj scheduler?

Glavni razlog:

```text
efikasno upravljanje velikim brojem Goroutines
```

---

Go zna više nego OS:

---

OS vidi:

```
Thread radi

Thread čeka
```

---

Go vidi:

```
Goroutine čeka channel

Goroutine čeka network

Goroutine spremna za CPU
```

---

Go runtime ima više informacija.

---

# Blocking problem

Primer:

```go
data := <-channel
```

---

Goroutine čeka.

---

Klasični model:

```
Thread blokiran
```

---

Go model:

```
Goroutine parkirana

OS Thread slobodan
```

---

Rezultat:

drugi poslovi mogu nastaviti.

---

# Goroutine lifecycle

Goroutine prolazi kroz stanja:

```
CREATED

    |

RUNNABLE

    |

RUNNING

    |

WAITING

    |

TERMINATED
```

---

Scheduler pomera Goroutine između ovih stanja.

---

# Runnable vs Running

Važna razlika.

---

## Runnable

Znači:

```
spremna za izvršavanje
```

---

Ali:

trenutno nema CPU.

---

---

## Running

Znači:

```
trenutno koristi CPU
```

---

Primer:

Imamo:

```
100 runnable Goroutines
```

---

CPU može izvršavati samo određeni broj.

---

Scheduler bira:

```
koja ide sledeća
```

---

# Concurrency i Scheduler

Concurrency znači:

```
više poslova napreduje istovremeno
```

---

Scheduler omogućava:

```
deljenje CPU vremena
```

---

Primer:

```
G1

G2

G3
```

---

CPU:

```
G1
G2
G3
G1
G2
...
```

---

Ovo daje osećaj paralelnog rada.

---

# Concurrency nije isto što i Parallelism

Važna razlika.

---

Concurrency:

```
više zadataka

jedan CPU

prebacivanje između njih
```

---

Parallelism:

```
više zadataka

više CPU jezgara

istovremeno izvršavanje
```

---

Primer:

CPU:

```
1 core
```

---

Može imati:

```
1000 Goroutines
```

---

To je:

```
Concurrency
```

---

CPU:

```
8 cores
```

---

Može imati:

```
8 Goroutines
```

istovremeno.

---

To je:

```
Parallelism
```

---

# Go Scheduler mentalni model

Zapamti:

```
Goroutines

      ↓

Go Scheduler

      ↓

OS Threads

      ↓

CPU
```

---

Go runtime odlučuje:

```
ko radi

kada radi

gde radi
```

---

# Zašto je scheduler važan Go developeru?

Zato što direktno utiče na:

- performanse,
- broj Goroutines,
- blokiranje,
- CPU utilization,
- concurrency dizajn.

---

Bez razumevanja scheduler-a teško je razumeti:

- deadlock,
- starvation,
- race condition,
- worker pool tuning,
- GOMAXPROCS.

---

# Česte zablude

---

## Zabluda 1

"Goroutine je isto što i thread."

Nije.

---

Goroutine:

```
runtime-managed
```

---

Thread:

```
OS-managed
```

---

## Zabluda 2

"Više Goroutines znači više paralelizma."

Ne nužno.

---

Više Goroutines znači:

```
više concurrent poslova
```

---

---

## Zabluda 3

"Go runtime uvek koristi sve CPU jezgre."

Ne.

---

Kontroliše ga:

```go
GOMAXPROCS
```

---

O tome detaljno kasnije.

---

# Mentalni model

Zapamti:

```
Goroutine

=

lagani zadatak


Scheduler

=

menadžer zadataka


Thread

=

izvršna jedinica


CPU

=

fizički resurs
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Go Scheduler,
- zašto postoji,
- razliku između Goroutines i OS Threads,
- zašto Go koristi runtime scheduler,
- kako se Goroutines izvršavaju,
- odnos concurrency i parallelism,
- osnovni mentalni model scheduler-a.

---

# Go Scheduler — G-M-P Model (Goroutine, Machine, Processor)

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- Šta je G-M-P model
- G — Goroutine
- M — Machine (OS Thread)
- P — Processor
- Veza između G, M i P
- Zašto je uveden Processor koncept
- Scheduler hijerarhija
- Primer izvršavanja Goroutine-a
- Mentalni model GMP arhitekture

---

# Uvod

U prethodnom delu naučili smo:

```
Goroutines

      ↓

Go Scheduler

      ↓

OS Threads

      ↓

CPU
```

---

Ali ova slika je pojednostavljena.

---

Interna arhitektura Go runtime scheduler-a koristi model:

```
G-M-P
```

---

GMP model je jedna od najvažnijih stvari za razumevanje Go concurrency-ja.

---

# Šta je G-M-P model?

GMP predstavlja tri ključne komponente:

```
G = Goroutine

M = Machine

P = Processor
```

---

Možemo ga predstaviti:

```
        G

        |
        |
        v

        M

        |
        |
        v

        P

        |
        |
        v

       CPU
```

---

Preciznije:

```
Goroutines (G)

        |

Scheduler

        |

Machines (M)

        |

Processors (P)

        |

CPU cores
```

---

# 1. G — Goroutine

`G` predstavlja jednu Goroutine.

---

Primer:

```go
go process()
```

---

Go runtime kreira:

```
G objekat
```

---

On sadrži informacije:

- stack,
- instruction pointer,
- status izvršavanja,
- reference na funkciju.

---

Primer:

Imamo:

```go
go worker()
go logger()
go consumer()
```

---

Interno:

```
G1 → worker()

G2 → logger()

G3 → consumer()
```

---

# Goroutine stanje

Goroutine može biti:

---

## Runnable

Spremna za rad:

```
čeka CPU
```

---

## Running

Trenutno izvršava instrukcije.

---

## Waiting

Čeka:

- channel,
- mutex,
- network,
- timer.

---

## Dead

Završila izvršavanje.

---

Primer:

```
G1

RUNNING

↓

WAITING

↓

RUNNABLE

↓

RUNNING

↓

DONE
```

---

# 2. M — Machine

`M` predstavlja:

```
OS Thread
```

---

To je stvarni sistemski thread.

---

M izvršava Go kod.

---

Primer:

```
M1

↓

CPU
```

---

Go runtime kreira i upravlja M objektima.

---

Važno:

```
Jedan M nije jedna Goroutine
```

---

Jedan M može izvršavati:

```
G1

zatim

G2

zatim

G3
```

---

Scheduler prebacuje posao.

---

# Context switching

Primer:

M1:

```
trenutno radi G1
```

---

G1 čeka:

```go
<-channel
```

---

Scheduler:

```
skida G1

stavlja G1 u waiting

uzima G2
```

---

M1:

```
radi G2
```

---

OS thread nije blokiran.

---

# 3. P — Processor

Ovo je najvažniji deo GMP modela.

---

`P` nije CPU.

---

P je:

```
runtime scheduling resource
```

---

Možemo ga zamisliti kao:

```
dozvola za izvršavanje Go koda
```

---

M može izvršavati Go kod samo ako poseduje P.

---

Model:

```
M + P = mogućnost izvršavanja
```

---

Bez P:

```
M ne može izvršavati Go kod
```

---

# Zašto postoji P?

Pre GMP modela:

```
G direktno vezan za M
```

---

Problem:

Scheduler je imao komplikovano upravljanje.

---

Sa P:

```
G

↓

P

↓

M
```

---

P poseduje:

- lokalni queue Goroutines,
- scheduling state,
- memory cache reference.

---

# Lokalni run queue

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
 G2,
 G3
]
```

---

M uzima posao:

```
P1 → M1 → CPU
```

---

---

# Više Processor-a

Primer:

CPU:

```
4 cores
```

---

GOMAXPROCS:

```
4
```

---

Go kreira:

```
P1
P2
P3
P4
```

---

Model:

```
P1 → M1 → CPU Core 1

P2 → M2 → CPU Core 2

P3 → M3 → CPU Core 3

P4 → M4 → CPU Core 4
```

---

To omogućava:

```
pravi parallelism
```

---

# Kompletan GMP odnos

Najjednostavnije:

```
          Goroutines

        G1 G2 G3 G4 G5


             ↓


        Processor


          P1 P2


             ↓


        Machine


          M1 M2


             ↓


            CPU
```

---

---

# Primer izvršavanja

Kod:

```go
go task1()

go task2()

go task3()
```

---

Runtime kreira:

```
G1

G2

G3
```

---

Scheduler:

stavlja ih u queue:

```
P1:

[
 G1,
 G2,
 G3
]
```

---

M dobija P:

```
M1 + P1
```

---

M izvršava:

```
G1
```

---

Ako G1 blokira:

```
G1 → waiting
```

---

Scheduler bira:

```
G2
```

---

---

# GMP i Channels

Primer:

```go
data := <-ch
```

---

Goroutine:

```
G1
```

blokira.

---

Scheduler:

```
G1 → waiting
```

---

M ne mora čekati.

---

Dobija:

```
G2
```

---

Zato Go može imati veliki broj Goroutines.

---

# GMP i Mutex

Slično:

```go
mutex.Lock()
```

---

Ako lock nije dostupan:

```
G1 čeka
```

---

Scheduler može:

```
G1 → waiting

G2 → running
```

---

---

# GMP i CPU bound posao

Primer:

```go
for {

	calculate()

}
```

---

Goroutine stalno koristi CPU.

---

Ako nema preemption:

```
G blokira druge
```

---

Go scheduler ima mehanizme da je prekine.

---

O tome detaljno u narednim delovima.

---

# GOMAXPROCS i P

Kasnije ćemo detaljno:

```go
runtime.GOMAXPROCS()
```

---

Ali osnovna ideja:

```
GOMAXPROCS

=

broj aktivnih P
```

---

Primer:

```go
GOMAXPROCS(4)
```

---

Znači:

```
4 P objekta
```

---

Maksimalno:

```
4 Goroutines
```

mogu istovremeno izvršavati Go kod.

---

# Česte zablude

---

## Zabluda 1

"P je CPU."

Nije.

---

P je:

```
scheduler resurs
```

---

---

## Zabluda 2

"Jedna Goroutine ima svoj Thread."

Nema.

---

Više Goroutines dele Threads.

---

---

## Zabluda 3

"Broj Goroutines = broj CPU poslova."

Nije.

---

Scheduler ih raspoređuje.

---

# Mentalni model

Zapamti:

```
G

=

šta treba izvršiti


P

=

gde se raspoređuje


M

=

ko izvršava
```

---

Još kraće:

```
G = Task

P = Permission

M = Worker
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta predstavlja GMP model,
- ulogu Goroutine (`G`),
- ulogu Machine (`M`),
- ulogu Processor (`P`),
- zašto P postoji,
- kako scheduler povezuje G, M i P,
- kako se Goroutines izvršavaju,
- osnovnu vezu sa `GOMAXPROCS`.

---

# Go Scheduler — Scheduler Lifecycle i Run Queue

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- Životni ciklus Goroutine-a
- Kreiranje nove Goroutine
- Scheduler lifecycle
- Runnable queue
- Local run queue
- Global run queue
- Prebacivanje Goroutines između stanja
- Kako scheduler bira sledeći posao
- Scheduler decision points

---

# Uvod

U prethodnom delu naučili smo:

```
G = Goroutine

M = OS Thread

P = Scheduler Processor
```

---

Ali ostaje važno pitanje:

> Kako scheduler odlučuje koja Goroutine će sledeća raditi?

---

Odgovor:

Go runtime koristi:

- run queue,
- stanje Goroutine-a,
- dostupne Processor-e,
- scheduler algoritme.

---

# Životni ciklus Goroutine-a

Svaka Goroutine prolazi kroz više stanja:

```
CREATED

   |

   v

RUNNABLE

   |

   v

RUNNING

   |

   +------------+

   |            |

   v            v

WAITING      DONE

```

---

# 1. CREATED stanje

Kada napišemo:

```go
go process()
```

---

Go runtime:

1. kreira Goroutine strukturu,
2. priprema stack,
3. dodaje je scheduler-u.

---

Primer:

```go
func main(){

	go worker()

}
```

---

Interno:

```
kreiran G1
```

---

Ali:

```
još nije izvršen
```

---

Nalazi se u stanju:

```
CREATED
```

---

# 2. RUNNABLE stanje

Nakon kreiranja:

```
G1 → runnable
```

---

Znači:

```
spremna za CPU
```

---

Ali trenutno:

```
nema procesorsko vreme
```

---

Primer:

Imamo:

```go
go task1()
go task2()
go task3()
```

---

Scheduler vidi:

```
Runnable:

G1

G2

G3
```

---

---

# 3. RUNNING stanje

Kada scheduler odluči:

```
uzmi G1
```

---

Dobijamo:

```
P

+

M

+

G1
```

---

Izvršavanje:

```
G1 → CPU
```

---

Samo ograničen broj Goroutines može biti:

```
RUNNING
```

---

Primer:

```
GOMAXPROCS = 4
```

---

Moguće:

```
G1 running

G2 running

G3 running

G4 running
```

---

---

# 4. WAITING stanje

Goroutine može čekati.

---

Primer:

```go
data := <-channel
```

---

Ako nema podatka:

```
G1 blokira
```

---

Scheduler:

```
G1 → waiting
```

---

CPU:

```
slobodan
```

---

Druga Goroutine:

```
G2 → running
```

---

---

# 5. DONE stanje

Kada funkcija završi:

```go
func worker(){

	fmt.Println("done")

}
```

---

Goroutine:

```
RUNNING

↓

DONE
```

---

Runtime oslobađa resurse.

---

# Kreiranje Goroutine-a

Primer:

```go
go calculate()
```

---

Proces:

```
1. Kreiraj G

2. Postavi stack

3. Dodaj u queue

4. Scheduler bira kada radi
```

---

Važno:

Kreiranje Goroutine-a:

ne znači:

```
odmah izvršavanje
```

---

Znači:

```
dodavanje kandidata scheduler-u
```

---

# Run Queue

Scheduler mora čuvati:

```
ko čeka izvršavanje
```

---

To se radi pomoću:

```
run queue
```

---

Postoje:

```
Local Run Queue

Global Run Queue
```

---

# Local Run Queue

Svaki `P` ima svoju queue.

---

Primer:

```
P1

[
 G1,
 G2,
 G3
]
```

---

Prednost:

Brz pristup.

---

Nema potrebe za globalnim lock-om.

---

---

# Global Run Queue

Postoji zajednička queue:

```
Global Queue

[
 G100,
 G101,
 G102
]
```

---

Koristi se kada:

- nema lokalnog posla,
- kreirano je mnogo Goroutines,
- scheduler treba balansiranje.

---

# Zašto postoje dve queue?

Performanse.

---

Loše:

```
sve Goroutines

↓

jedna globalna queue

↓

mutex

↓

sporije
```

---

Bolje:

```
P1 → lokalna queue

P2 → lokalna queue

P3 → lokalna queue
```

---

Veći paralelizam.

---

# Primer raspoređivanja

Imamo:

```go
go A()
go B()
go C()
```

---

Scheduler:

```
P1:

[
 A,
 B,
 C
]
```

---

M1 dobija P1:

```
M1 + P1
```

---

Izvršava:

```
A
```

---

A blokira:

```
channel receive
```

---

Stanje:

```
A → WAITING
```

---

Scheduler uzima:

```
B
```

---

Rezultat:

```
CPU nije blokiran
```

---

# Scheduler decision points

Scheduler donosi odluke kada:

---

## 1. Goroutine blokira

Primer:

```go
<-channel
```

---

---

## 2. Goroutine završi

Primer:

```go
return
```

---

---

## 3. Sistemski poziv

Primer:

```go
file.Read()
```

---

---

## 4. Timer događaj

Primer:

```go
time.Sleep()
```

---

---

## 5. Preemption

Kada Goroutine dugo koristi CPU.

---

---

# `time.Sleep` i scheduler

Primer:

```go
time.Sleep(time.Second)
```

---

Ne znači:

```
blokiraj thread
```

---

Go radi:

```
Goroutine → waiting

Thread → slobodan
```

---

Posle:

```
timer expires

↓

Goroutine runnable
```

---

# Primer sa više Goroutines

Kod:

```go
func main(){

	for i:=0;i<5;i++{

		go worker(i)

	}


	time.Sleep(time.Second)

}
```

---

Možemo imati:

```
G1
G2
G3
G4
G5
```

---

Ali scheduler bira:

```
ko kada radi
```

---

Ne postoji garancija:

```
G1 pa G2 pa G3
```

---

# Važna osobina

Go scheduler nije:

```
FIFO scheduler
```

---

Redosled izvršavanja nije garantovan.

---

Zato:

Loše:

```go
go print("A")
go print("B")
```

---

Ne očekivati:

```
A

B
```

---

Može biti:

```
B

A
```

---

# Scheduler i determinističnost

Concurrency kod često nije determinističan.

---

Isti program:

```
Run 1:

G1 G2 G3


Run 2:

G2 G3 G1
```

---

Zato postoje:

- channels,
- synchronization primitives,
- mutex.

---

# Mentalni model

Zapamti:

```
go func()

↓

CREATE G

↓

RUNNABLE queue

↓

Scheduler izbor

↓

RUNNING

↓

WAITING ili DONE
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kompletan lifecycle Goroutine-a,
- razliku između CREATED, RUNNABLE, RUNNING i WAITING,
- kako rade run queue,
- lokalni i globalni queue,
- kada scheduler donosi odluke,
- kako blocking operacije oslobađaju CPU,
- zašto redosled Goroutines nije garantovan.

---

# Go Scheduler — Work Stealing i Balansiranje Goroutines

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- Problem nebalansiranog rada
- Zašto je potreban Work Stealing
- Kako Work Stealing funkcioniše
- Processor balansiranje
- Lokalni queue između Processor-a
- Global queue kao fallback
- Primer neravnomerne distribucije
- Prednosti Work Stealing algoritma

---

# Uvod

U prethodnom delu naučili smo:

```
svaki P ima svoj Local Run Queue
```

---

Primer:

```
P1

[
 G1,
 G2,
 G3,
 G4
]


P2

[
 G5
]
```

---

Problem:

P1 ima mnogo posla.

P2 skoro ništa.

---

Dobijamo:

```
P1 overloaded

P2 idle
```

---

To nije efikasno.

---

Go runtime rešava ovaj problem pomoću:

```
Work Stealing
```

---

# Šta je Work Stealing?

Work Stealing je algoritam gde:

> Slobodan Processor uzima posao od drugog Processor-a koji ima višak posla.

---

Model:

```
P1

[
 G1,
 G2,
 G3,
 G4
]


P2

[]

```

---

P2 nema posla.

---

P2 kaže:

```
Mogu da pomognem.
```

---

Uzima deo:

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

Sada:

```
oba rade
```

---

# Zašto je Work Stealing potreban?

Bez njega:

```
P1:

100 Goroutines


P2:

0 Goroutines
```

---

CPU:

```
50% iskorišćenost
```

---

Sa Work Stealing:

```
P1:

50 Goroutines


P2:

50 Goroutines
```

---

CPU:

```
bolje iskorišćen
```

---

# Kako izgleda scheduler balansiranje?

Imamo:

```
Goroutines

       ↓

Local Queues

       ↓

Processor-i

       ↓

OS Threads

       ↓

CPU
```

---

Scheduler stalno proverava:

```
ko ima posao?

ko je slobodan?
```

---

# Primer

Imamo:

```go
for i:=0;i<1000;i++{

	go process(i)

}
```

---

Kreirano:

```
1000 Goroutines
```

---

Moguće stanje:

```
P1

[
G1...G800
]


P2

[
G801...G900
]


P3

[
G901...G1000
]
```

---

Ali posle nekog vremena:

```
P1:

800 poslova


P2:

završen


P3:

završen
```

---

P2 i P3 mogu pomoći P1.

---

# Stealing pravilo

Slobodan P:

1. proverava svoj queue,
2. proverava global queue,
3. pokušava da ukrade posao od drugog P.

---

Pojednostavljeno:

```
Local Queue

↓

Global Queue

↓

Steal from another P
```

---

# Zašto ne uzima ceo queue?

Primer:

P1:

```
[
G1,
G2,
G3,
G4
]
```

---

P2 uzima sve:

```
P1 []

P2 [G1,G2,G3,G4]
```

---

Problem:

P1 opet nema ništa.

---

Zato se obično uzima deo posla.

---

Primer:

```
P1

[
G1,
G2,
G3,
G4
]


P2 steals:

[
G3,
G4
]
```

---

Rezultat:

```
P1:

G1,G2


P2:

G3,G4
```

---

# Veza sa GMP modelom

Prethodno:

```
G = posao

P = scheduling resurs

M = izvršilac
```

---

Work stealing radi između:

```
P objekata
```

---

Primer:

```
P1

     steals

P2
```

---

Ne između:

```
Threads
```

---

# Local Queue prednost

Local queue omogućava:

```
brz pristup
```

---

Bez nje:

svaka operacija:

```
global lock

↓

global queue

↓

sporije
```

---

Sa local queue:

```
P1 → svoje Goroutines
```

---

Veoma brzo.

---

# Global Queue uloga

Global queue nije glavni put.

---

Koristi se kao:

```
rezervni izvor posla
```

---

Primer:

Novi posao:

```
go task()
```

---

Može završiti:

```
local queue

ili

global queue
```

---

# Work Stealing i CPU utilizacija

Cilj:

```
maksimalno iskoristiti CPU
```

---

Bez balansiranja:

```
CPU1 100%

CPU2 0%

CPU3 0%

CPU4 0%
```

---

Sa work stealing:

```
CPU1 80%

CPU2 80%

CPU3 80%

CPU4 80%
```

---

---

# Work Stealing i Concurrency Patterns

Ovo direktno utiče na:

- worker pools,
- fan-out,
- parallel processing,
- batch processing.

---

Primer:

Worker pool:

```
Jobs

 |

Workers

 |

CPU
```

---

Scheduler dodatno balansira:

```
Workers

↓

P

↓

M

↓

CPU
```

---

# Primer CPU-bound workload-a

Kod:

```go
func calculate(){

	for i:=0;i<1000000000;i++{

	}

}
```

---

Ako imamo:

```go
go calculate()
go calculate()
go calculate()
go calculate()
```

---

Scheduler raspoređuje:

```
P1 → G1

P2 → G2

P3 → G3

P4 → G4
```

---

---

# Work Stealing nije magija

Važno:

Work stealing ne rešava:

- race condition,
- deadlock,
- loš dizajn,
- blokiranje.

---

On rešava samo:

```
balansiranje scheduling posla
```

---

# Česte zablude

---

## Zabluda 1

"Scheduler raspoređuje Goroutines direktno na CPU."

Nije potpuno tačno.

---

Putanja:

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

---

## Zabluda 2

"Svaka Goroutine ima svoj Thread."

Nema.

---

---

## Zabluda 3

"Work stealing znači kopiranje Goroutines."

Ne.

---

To znači:

```
preuzimanje scheduling posla
```

---

# Mentalni model

Zapamti:

```
P1 ima posao

P2 nema posao


↓

P2 krade deo posla


↓

CPU bolje iskorišćen
```

---

Još kraće:

```
Idle P

+

Busy P

=

Work Stealing
```

---

# 📋 Rezime

U ovom delu naučili smo:

- problem nebalansiranog rada između Processor-a,
- šta je Work Stealing,
- kako slobodan Processor preuzima posao,
- odnos Work Stealing-a i GMP modela,
- ulogu Local i Global Run Queue,
- kako scheduler povećava CPU iskorišćenost.

---

# Go Scheduler — System Calls, Blocking Operations i Preemption

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- Problem blokiranja OS Thread-a
- Syscall operacije
- M izlazi iz scheduler sistema
- Hand-off mehanizam
- Network polling
- Goroutine parking
- CPU-bound Goroutines
- Cooperative preemption
- Asynchronous preemption
- Kako Go sprečava blokiranje programa

---

# Uvod

Do sada smo naučili:

```
G → P → M → CPU
```

---

Ali šta se dešava kada Goroutine uradi nešto što blokira?

Primeri:

```go
file.Read()

network.Call()

syscall()

time.Sleep()
```

---

Ako bi se ceo OS Thread blokirao:

```
M blokiran

↓

CPU izgubljen
```

---

To bi bilo veoma neefikasno.

---

Zato Go scheduler ima mehanizme:

- parking Goroutines,
- oslobađanje Processor-a,
- kreiranje novih Machine objekata,
- preemption.

---

# Blocking operacije

Postoje različiti tipovi čekanja.

---

## 1. Goroutine waiting

Primer:

```go
value := <-channel
```

---

Goroutine čeka podatak.

---

Go može:

```
G → waiting

M → slobodan

P → nastavlja rad
```

---

---

## 2. Timer waiting

Primer:

```go
time.Sleep(time.Second)
```

---

Go ne radi:

```
blokiraj thread
```

---

Nego:

```
G → sleep state
```

---

Thread može izvršavati druge Goroutines.

---

---

## 3. Syscall blocking

Primer:

```go
syscall.Read()
```

---

Ovo je komplikovanije.

---

# Šta je syscall?

Syscall je:

```
poziv između user programa i kernel-a
```

---

Primeri:

- file operations,
- network,
- device access.

---

Tok:

```
Go program

↓

OS kernel

↓

rezultat
```

---

# Problem syscall-a

Primer:

```
M1 izvršava G1


G1 radi syscall


M1 blokiran
```

---

Sada:

```
P nema M
```

---

CPU može ostati neiskorišćen.

---

# Hand-off mehanizam

Go runtime rešava ovo:

Pre:

```
P1

 |

M1

 |

G1 syscall
```

---

G1 blokira.

---

Runtime:

```
M1 ide u syscall

P1 se odvaja

P1 dobija novi M
```

---

Novo stanje:

```
M1

↓

syscall


P1

↓

M2

↓

drugi G
```

---

Rezultat:

```
program nastavlja rad
```

---

# Grafički prikaz

Pre:

```
P1

 |

M1

 |

G1
```

---

Tokom syscall-a:

```
G1

↓

M1

↓

Kernel
```

---

Scheduler:

```
P1

↓

M2

↓

G2
```

---

---

# Network polling

Go ima poseban sistem:

```
network poller
```

---

Za network operacije:

- TCP,
- HTTP,
- sockets.

---

Primer:

```go
conn.Read(buffer)
```

---

Ne mora blokirati OS Thread.

---

Go može:

```
G1 čeka network

P1 radi G2
```

---

---

# Goroutine parking

Kada Goroutine ne može nastaviti:

Go je:

```
parkira
```

---

Primer:

```go
<-ch
```

---

Ako nema podatka:

```
G1 parked
```

---

Kada podatak stigne:

```
G1 runnable
```

---

Scheduler je ponovo uzima.

---

# CPU-bound Goroutine problem

Sada drugi slučaj.

---

Primer:

```go
func busy(){

	for {

		calculate()

	}

}
```

---

Ova Goroutine:

- ne blokira,
- ne čeka,
- stalno koristi CPU.

---

Problem:

```
G1

↓

CPU zauzet
```

---

Kako druga Goroutines dobijaju šansu?

---

Odgovor:

```
Preemption
```

---

# Šta je preemption?

Preemption znači:

> Scheduler može privremeno prekinuti Goroutine i dati CPU drugoj Goroutine.

---

Model:

```
G1 radi

↓

scheduler interrupt

↓

G1 pauza

↓

G2 radi
```

---

---

# Cooperative preemption

Stariji Go modeli koristili su:

```
cooperative preemption
```

---

Goroutine je morala doći do:

- function call,
- safe point,
- runtime check.

---

Problem:

Beskonačna petlja:

```go
for {

}
```

---

mogla je dugo blokirati scheduler.

---

# Asynchronous preemption

Moderni Go runtime koristi:

```
asynchronous preemption
```

---

Scheduler može:

```
prekinuti Goroutine nezavisno
```

---

Primer:

```go
for {

	calculate()

}
```

---

Druge Goroutines ipak dobijaju CPU vreme.

---

# Preemption i fairness

Cilj:

```
nijedna Goroutine ne dominira CPU-om
```

---

Primer:

Imamo:

```
G1 CPU-heavy

G2 HTTP request

G3 Logger
```

---

Scheduler obezbeđuje:

```
G1 ne blokira G2 i G3
```

---

# Scheduler i time slice

Scheduler koristi vremenski kvant.

---

Pojednostavljeno:

```
G1

radi malo


↓

G2

radi malo


↓

G3

radi malo
```

---

Ovo daje:

```
fair scheduling
```

---

# Blocking vs Non-blocking primer

## Loše

```go
func worker(){

	for {

		heavyCalculation()

	}

}
```

---

---

## Bolje

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

---

# Veza sa Concurrency Patterns

Scheduler direktno utiče na:

- worker pools,
- pipelines,
- fan-out/fan-in,
- parallel processing.

---

Primer:

Worker pool:

```
100 workers

↓

scheduler

↓

CPU
```

---

Scheduler odlučuje:

```
koji worker radi sada
```

---

# Česte zablude

---

## Zabluda 1

"Sleep blokira thread."

Ne.

---

Obično:

```
Goroutine čeka

Thread slobodan
```

---

---

## Zabluda 2

"Channel blokira CPU."

Ne.

---

Blokira:

```
Goroutine
```

---

Ne nužno:

```
Thread
```

---

---

## Zabluda 3

"Jedna beskonačna petlja uvek blokira ceo program."

Ne u modernom Go runtime-u.

---

Preemption omogućava:

```
scheduler kontrolu
```

---

# Mentalni model

Zapamti:

```
Blocking Goroutine

↓

park


Blocking syscall

↓

detach P


CPU-heavy Goroutine

↓

preemption
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta se dešava tokom blokirajućih operacija,
- razliku između Goroutine čekanja i syscall blokiranja,
- hand-off mehanizam,
- network polling,
- Goroutine parking,
- cooperative i asynchronous preemption,
- kako scheduler održava fairness.

---

# Go Scheduler — Tuning, Best Practices i Praktični Zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 6/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/06-go-scheduler.md`

---

# 📚 Sadržaj ovog dela

- GOMAXPROCS i scheduler
- Scheduler tuning
- Kada menjati GOMAXPROCS
- Scheduler observability
- Debugging scheduler problema
- Najčešće greške
- Best practices
- Praktični zadaci

---

# Uvod

Do sada smo prošli kompletnu arhitekturu:

```
Goroutines

      ↓

G-M-P Scheduler

      ↓

OS Threads

      ↓

CPU
```

---

Sada prelazimo na praktičnu stranu:

- kako kontrolisati scheduler,
- kako posmatrati ponašanje,
- kako izbeći loš dizajn.

---

# GOMAXPROCS

Jedan od najvažnijih scheduler parametara je:

```go
runtime.GOMAXPROCS()
```

---

On određuje:

```
koliko Processor objekata (P)
može biti aktivno
```

---

Primer:

```go
runtime.GOMAXPROCS(4)
```

---

Znači:

```
P1

P2

P3

P4
```

---

Maksimalno:

```
4 Goroutines
```

mogu izvršavati Go kod paralelno.

---

# GOMAXPROCS nije broj Goroutines

Važna razlika.

---

Primer:

```go
go worker()
```

---

Možemo imati:

```
100000 Goroutines
```

---

Ali:

```go
GOMAXPROCS = 8
```

---

Znači:

```
8 aktivnih izvršavanja
```

---

Ostale Goroutines čekaju scheduler.

---

# Default GOMAXPROCS

U modernom Go runtime-u:

default je:

```
broj dostupnih CPU logical cores
```

---

Primer:

Mašina:

```
8 CPU cores
```

---

Go obično koristi:

```
GOMAXPROCS = 8
```

---

---

# Kada menjati GOMAXPROCS?

U većini aplikacija:

```
NE menjati
```

---

Go runtime već dobro bira vrednost.

---

Ali postoje situacije:

- container limit,
- CPU quota,
- specijalizovani workload.

---

# Primer container problema

Imamo:

Host:

```
32 CPU cores
```

---

Container limit:

```
2 CPU cores
```

---

Ako runtime vidi:

```
32
```

može napraviti previše aktivnih P objekata.

---

Rezultat:

- scheduling overhead,
- contention,
- lošije performanse.

---

---

# CPU-bound vs IO-bound workload

Scheduler ponašanje zavisi od tipa posla.

---

# CPU-bound

Primer:

```go
calculateHash()
compressData()
imageProcessing()
```

---

Karakteristike:

- koristi CPU,
- malo čeka.

---

Potrebno:

```
dobar broj P
```

---

---

# IO-bound

Primer:

```go
HTTP request

Database query

File read
```

---

Karakteristike:

- mnogo čekanja,
- malo CPU rada.

---

Možemo imati:

```
mnogo Goroutines
```

---

Scheduler ih parkira dok čekaju.

---

# Scheduler tuning princip

Ne optimizovati:

```
pre merenja
```

---

Prvo:

```
measure

↓

profile

↓

optimize
```

---

Koristiti:

- benchmarks,
- CPU profile,
- trace.

---

# Scheduler Trace

Go ima runtime trace alat.

---

Primer:

```bash
go test -trace trace.out
```

---

Prikazuje:

- Goroutine scheduling,
- blocking,
- network wait,
- GC interakcije.

---

Analiza:

```bash
go tool trace trace.out
```

---

---

# GODEBUG scheduler informacije

Go runtime omogućava debug informacije.

Primer:

```bash
GODEBUG=schedtrace=1000
```

---

Dobijamo:

```
scheduler state
```

svakih:

```
1000ms
```

---

Možemo videti:

- broj Goroutines,
- idle processors,
- running threads.

---

Primer:

```bash
GODEBUG=schedtrace=1000 ./app
```

---

# Scheduler i Garbage Collector

Scheduler sarađuje sa GC-om.

---

GC može:

- zaustaviti deo runtime-a,
- zahtevati safe points.

---

Zato:

loš concurrency dizajn može uticati i na GC.

---

# Scheduler best practices

---

## 1. Ne kreirati nepotrebno mnogo Goroutines

Loše:

```go
for {

	go expensiveTask()

}
```

---

Bolje:

```
worker pool
```

---

---

## 2. Ne blokirati dugo CPU

Loše:

```go
for {

	heavyCalculation()

}
```

---

Bolje:

- podeliti posao,
- koristiti worker-e,
- omogućiti cancellation.

---

---

## 3. Koristiti channels za koordinaciju

Dobro:

```go
jobs <- task
```

---

Bolje nego:

- manuelni polling,
- sleep loop.

---

---

## 4. Paziti na shared state

Scheduler povećava concurrency.

---

Ali ne rešava:

- race condition,
- data corruption.

---

Potrebni su:

- Mutex,
- channels,
- atomic.

---

# Debugging concurrency problema

Kada program ima problem:

---

## Problem:

```
CPU 100%
```

Proveriti:

- infinite loops,
- CPU-bound Goroutines,
- lock contention.

---

## Problem:

```
malo CPU utilization
```

Proveriti:

- blocking,
- IO wait,
- channel deadlock.

---

## Problem:

```
spor sistem
```

Proveriti:

- previše Goroutines,
- scheduler overhead,
- GC pressure.

---

# Race detector

Za concurrency bugove:

```bash
go test -race
```

---

Otkriva:

- data races,
- unsafe shared memory.

---

# Benchmark scheduler ponašanja

Primer:

```go
go test -bench .
```

---

Merimo:

- throughput,
- latency,
- overhead.

---

# Praktični primer

Imamo:

```go
for i:=0;i<1000;i++{

	go process(i)

}
```

---

Pitanja:

1. Koliko Goroutines?

```
1000
```

---

2. Koliko CPU izvršava?

Zavisi od:

```
GOMAXPROCS
```

---

3. Ko raspoređuje?

```
Go Scheduler
```

---

# Senior mentalni model

Kada vidiš:

```go
go func()
```

razmišljaj:

```
kreiran G

↓

queue

↓

P bira posao

↓

M izvršava

↓

CPU radi
```

---

Kada vidiš:

```go
<-channel
```

razmišljaj:

```
G blokira

↓

scheduler oslobađa resurse

↓

druga G radi
```

---

Kada vidiš:

```go
CPU heavy loop
```

razmišljaj:

```
preemption
```

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Scheduler osnove

Napraviti program:

- kreira 100 Goroutines,
- svaka ispisuje svoj ID,
- posmatrati redosled izvršavanja.

Cilj:

razumeti nedeterminističnost.

---

# 🟡 Nivo 2 — CPU workload

Napraviti:

```go
worker()
```

koji radi:

- intenzivan CPU posao.

Eksperiment:

menjati:

```go
GOMAXPROCS
```

i meriti vreme.

---

# 🟠 Nivo 3 — Blocking workload

Napraviti:

- 1000 Goroutines,
- svaka čeka channel.

Posmatrati:

- broj thread-ova,
- CPU utilization.

---

# 🔴 Nivo 4 — Senior

Napraviti servis:

```
HTTP API

+

Worker Pool

+

Background Scheduler

+

Database simulation
```

Implementirati:

- context cancellation,
- worker pool,
- graceful shutdown,
- profiling.

---

# Završni rezime

Go Scheduler omogućava:

```
milioni Goroutines

↓

efikasno raspoređivanje

↓

mali broj OS Threads

↓

visoke performanse
```

---

Najvažniji koncepti:

```
G = Goroutine

M = OS Thread

P = Scheduler resource

Run Queue

Work Stealing

Preemption

GOMAXPROCS
```

---

### ➡️ Sledeća lekcija **[**GOMAXPROCS**](07-gomaxprocs.md)**

Obradićemo:

- detaljnu internu ulogu GOMAXPROCS-a,
- odnos sa CPU core-ovima,
- runtime.GOMAXPROCS,
- container okruženja,
- tuning strategije.
