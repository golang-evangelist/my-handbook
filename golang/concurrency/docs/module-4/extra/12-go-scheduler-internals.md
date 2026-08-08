# Go Scheduler Internals

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Go Scheduler Internals
>
> Level: Advanced / Senior

> Razumevanje načina na koji Go Runtime izvršava goroutine predstavlja jedan od najvažnijih koraka ka pisanju visokoperformantnih konkurentnih aplikacija.

---

# 📚 Sadržaj

- Šta je Go Scheduler?
- Zašto Scheduler postoji?
- Istorijski razvoj
- G–M–P model
- Životni ciklus goroutine-a
- Scheduler ciljevi
- Osnovna terminologija

---

# 1. Šta Je Go Scheduler?

Go Scheduler je deo Go Runtime-a koji odlučuje:

- koja goroutine će se izvršavati
- kada će se izvršavati
- na kom operativnom thread-u će se izvršavati
- kada će biti pauzirana
- kada će biti nastavljena

---

Bez Scheduler-a:

```
goroutine

↓

ne bi postojale
```

---

Runtime automatski upravlja:

- thread-ovima
- raspoređivanjem
- prebacivanjem između goroutine-a
- balansiranjem opterećenja

---

# 2. Zašto Scheduler Postoji?

Operativni sistemi raspoređuju:

```
OS Thread
```

---

Go raspoređuje:

```
Goroutine
```

---

To znači da:

```
Više hiljada goroutine-a

↓

može koristiti

↓

mali broj OS thread-ova
```

---

Bez ovakvog modela:

- svaka goroutine bi zahtevala poseban thread
- memorijska potrošnja bi bila ogromna
- context switch bi bio skup

---

# 3. Thread vs Goroutine

OS Thread:

- upravlja ga operativni sistem
- skup za kreiranje
- veliki stack
- sporiji context switch

---

Goroutine:

- upravlja Go Runtime
- veoma jeftina za kreiranje
- mali početni stack
- brz scheduler switch

---

Poređenje:

| Karakteristika | OS Thread | Goroutine |
|----------------|-----------|-----------|
| Kreiranje | Skupo | Veoma jeftino |
| Početni stack | Veliki | Mali |
| Upravljanje | OS | Go Runtime |
| Broj instanci | Hiljade | Stotine hiljada ili milioni |

---

# 4. Ciljevi Scheduler-a

Scheduler pokušava da postigne:

- maksimalno iskorišćenje CPU-a
- minimalan broj context switch-eva
- fer raspodelu izvršavanja
- malu latenciju
- visoku propusnost (throughput)

---

Drugim rečima:

```
više posla

↓

manje čekanja

↓

manje overhead-a
```

---

# 5. Istorijski Razvoj

Prve verzije Go-a koristile su jednostavniji scheduler.

---

Kasnije su uvedeni:

- G–M–P model
- work stealing
- asinhrona preempcija
- poboljšano balansiranje

---

Današnji scheduler predstavlja rezultat dugogodišnje optimizacije.

---

# 6. G–M–P Model

Srce Go Scheduler-a čine tri osnovna pojma:

```
G

↓

M

↓

P
```

---

Njihova značenja su:

```
G = Goroutine

M = Machine (OS Thread)

P = Processor
```

---

Ovo je model koji Go Runtime koristi za izvršavanje goroutine-a.

---

# 7. Šta Predstavlja G?

G označava:

```
Goroutine
```

---

Sadrži:

- stack
- programski brojač (Program Counter)
- stanje izvršavanja
- informacije potrebne scheduler-u

---

Jedna goroutine:

```
nije

OS Thread
```

---

Već:

```
lagana jedinica izvršavanja.
```

---

# 8. Šta Predstavlja M?

M označava:

```
Machine
```

što zapravo predstavlja:

```
OS Thread
```

---

M izvršava goroutine.

---

Ali:

```
M

ne može raditi

bez P.
```

---

# 9. Šta Predstavlja P?

P označava:

```
Processor
```

---

Važno:

Processor nije fizički CPU.

---

On predstavlja:

```
dozvolu

za izvršavanje Go koda.
```

---

P poseduje:

- lokalni run queue
- scheduler podatke
- cache za izvršavanje

---

Bez slobodnog P:

```
goroutine

ne mogu da se izvršavaju.
```

---

# 10. Kako G, M i P Sarađuju?

Najjednostavniji prikaz:

```
      G
      │
      ▼
+------------+
|     M      |
+------------+
      ▲
      │
+------------+
|     P      |
+------------+
```

---

Odnos:

```
P

dodeljuje posao

↓

M

izvršava

↓

G
```

---

# 11. Jedan Primer

Pretpostavimo:

```
8 CPU jezgara
```

---

Go obično kreira:

```
8 Processor-a (P)
```

---

Tokom rada može postojati:

```
50000 goroutine-a

↓

8 P

↓

10-20 OS thread-ova
```

---

Scheduler neprestano raspoređuje posao.

---

# 12. Životni Ciklus Goroutine-a

Pojednostavljeno:

```
Create

↓

Runnable

↓

Running

↓

Waiting

↓

Runnable

↓

Finished
```

---

Scheduler stalno pomera goroutine između ovih stanja.

---

# 13. Scheduler Kao Dirigent

Dobra analogija:

```
Orkestar
```

---

Instrumenti:

```
goroutine
```

---

Muzičari:

```
OS thread
```

---

Dirigent:

```
Go Scheduler
```

---

Dirigent određuje:

```
ko svira

i

kada.
```

---

# 14. Šta Ćemo Naučiti U Ovom Modulu?

U narednim delovima detaljno ćemo obraditi:

- G–M–P arhitekturu
- run queue
- local i global queue
- work stealing
- scheduler loop
- preemption
- netpoller
- syscall handling
- scheduler tracing
- scheduler optimizacije

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Go Scheduler

✅ zašto postoji

✅ razliku između thread-a i goroutine-a

✅ ciljeve scheduler-a

✅ istorijski razvoj

✅ osnovni G–M–P model

✅ životni ciklus goroutine-a

---

# Go Scheduler Internals

## Deo #2 — G–M–P Architecture Deep Dive

---

# 📚 Sadržaj

- Detaljna arhitektura G
- Detaljna arhitektura M
- Detaljna arhitektura P
- Scheduler state machine
- Veza između G, M i P
- Životni ciklus izvršavanja

---

# 1. G – Goroutine

G predstavlja osnovnu jedinicu izvršavanja u Go-u.

Jedna G struktura sadrži mnogo više od same funkcije.

Pojednostavljeno:

```
Goroutine

↓

Function

↓

Stack

↓

Registers

↓

Status

↓

Scheduler Metadata
```

---

Najvažnije informacije koje G čuva:

- početak izvršavanja funkcije
- trenutni Program Counter (PC)
- pokazivač na stack
- informacije za scheduler
- informacije za garbage collector

---

# 2. Goroutine Stack

Svaka goroutine poseduje sopstveni stack.

Za razliku od OS thread-a:

```
Thread

↓

veliki fiksni stack
```

---

Go koristi:

```
mali početni stack

↓

automatski raste

↓

automatski se smanjuje
```

---

Prednosti:

- veoma mala početna potrošnja memorije
- veliki broj goroutine-a
- efikasnije korišćenje RAM-a

---

# 3. Status Goroutine-a

Tokom života goroutine menja stanje.

Najčešća stanja:

```
Runnable

Running

Waiting

Dead
```

---

Runnable:

```
spremna

čeka izvršavanje
```

---

Running:

```
trenutno koristi CPU
```

---

Waiting:

```
čeka:

channel

mutex

I/O

timer
```

---

Dead:

```
izvršavanje završeno
```

---

# 4. M – Machine

M predstavlja:

```
OS Thread
```

---

Njegov zadatak je jednostavan:

```
izvršava Go kod
```

---

Jedan M:

- poseduje OS thread
- izvršava jednu goroutine u datom trenutku
- sarađuje sa jednim P

---

Može postojati više M nego P.

---

# 5. Zašto Više M?

Razmotrimo situaciju:

Jedan thread izvršava:

```
network read
```

---

Ako thread blokira:

```
OS blokira thread
```

---

Ali Go želi:

```
ostale goroutine

da nastave rad.
```

---

Rešenje:

Scheduler kreira ili koristi drugi M.

---

# 6. P – Processor

P je najvažniji deo scheduler-a.

On predstavlja:

```
execution context
```

---

P sadrži:

- lokalni run queue
- scheduler cache
- GC pomoćne informacije
- lokalne resurse runtime-a

---

Bez P:

```
M

ne može

izvršavati Go kod.
```

---

# 7. Lokalni Run Queue

Svaki P poseduje:

```
Local Run Queue
```

---

Primer:

```
P0

↓

G1

G2

G3

G4
```

---

Scheduler prvo uzima goroutine:

iz lokalnog reda.

---

To smanjuje:

- contention
- zaključavanje
- pristup globalnim strukturama

---

# 8. Global Run Queue

Pored lokalnih redova postoji:

```
Global Queue
```

---

Koristi se kada:

- lokalni red ostane prazan
- kreira se veliki broj novih goroutine-a
- vrši se balansiranje opterećenja

---

Pojednostavljeno:

```
        Global Queue

        /    |    \

      P0    P1    P2
```

---

Scheduler preferira:

```
Local Queue

↓

Global Queue
```

---

zbog boljih performansi.

---

# 9. Veza Između G, M i P

Kompletna slika:

```
        G1

        G2

        G3

         │

         ▼

+-----------------+

Local Run Queue

+-----------------+

         │

         ▼

+--------P--------+

         │

         ▼

+--------M--------+

         │

         ▼

      CPU Core
```

---

P bira:

```
sledeću goroutine
```

---

M izvršava.

---

# 10. Scheduler State Machine

Pojednostavljen tok:

```
Create G

↓

Runnable

↓

Local Queue

↓

Running

↓

Waiting

↓

Runnable

↓

Finished
```

---

Svaki prelaz kontroliše scheduler.

---

# 11. Primer Izvršavanja

Program:

```go
go A()

go B()

go C()
```

---

Scheduler:

```
G(A)

↓

Queue

↓

Running
```

---

Zatim:

```
G(B)

↓

Queue

↓

Running
```

---

Pa:

```
G(C)

↓

Queue

↓

Running
```

---

Redosled:

```
nije garantovan.
```

---

# 12. Zašto Je G–M–P Efikasan?

Prednosti:

✅ malo thread-ova

✅ mnogo goroutine-a

✅ mali stack

✅ lokalni redovi

✅ mali scheduler overhead

---

Rezultat:

```
visok throughput

+

niska latencija
```

---

# 13. Česte Zablude

❌ Jedna goroutine = jedan thread

Netačno.

---

❌ Jedan P = jedan CPU

Netačno.

P je runtime koncept.

---

❌ Goroutine odmah kreće sa izvršavanjem

Netačno.

Prvo ulazi u scheduler.

---

❌ Scheduler koristi samo jedan red

Netačno.

Postoje:

- local queue
- global queue

---

# 14. Mentalni Model

Najlakše je razmišljati ovako:

```
G

=

Job


P

=

Dispatcher


M

=

Worker
```

---

Dispatcher:

```
bira sledeći posao.
```

---

Worker:

```
izvršava posao.
```

---

# 15. Šta Sledi?

Sada kada poznajemo:

- G
- M
- P

možemo razumeti:

```
kako scheduler

pronalazi posao

kada nema lokalnog posla.
```

---

To nas vodi do:

```
Work Stealing
```

i:

```
Run Queue algoritama.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ unutrašnju strukturu G

✅ ulogu M

✅ ulogu P

✅ lokalni run queue

✅ globalni run queue

✅ scheduler state machine

✅ zašto je G–M–P model efikasan

---

# Go Scheduler Internals

## Deo #3 — Run Queues i Work Stealing

---

# 📚 Sadržaj

- Šta je Run Queue?
- Local Run Queue
- Global Run Queue
- Work Stealing
- Load Balancing
- Scheduler Fairness
- Tipični scenariji izvršavanja

---

# 1. Šta Je Run Queue?

Run Queue predstavlja red goroutine-a koje su:

```
Runnable
```

odnosno:

```
spremne za izvršavanje

ali trenutno ne koriste CPU.
```

---

Scheduler bira goroutine upravo iz ovih redova.

---

Pojednostavljeno:

```
Runnable Goroutines

↓

Run Queue

↓

Scheduler

↓

CPU
```

---

# 2. Zašto Postoji Local Run Queue?

Svaki P poseduje:

```
svoj

Local Run Queue
```

---

Primer:

```
P0

↓

G1

G2

G3

G4
```

---

Ako P ima spremne goroutine:

```
uzima posao

iz svog reda.
```

---

Prednosti:

- nema globalnog zaključavanja
- manje contention-a
- bolji CPU cache locality
- veći throughput

---

# 3. Global Run Queue

Pored lokalnih redova postoji i:

```
Global Run Queue
```

---

Koristi se kao zajednički rezervoar poslova.

---

Primer:

```
             Global Queue

           G21

           G22

           G23

        /     |      \

      P0     P1      P2
```

---

Scheduler poseže za globalnim redom kada:

- lokalni red ostane prazan
- potrebno je balansirati opterećenje
- određene goroutine nisu dodeljene nijednom P

---

# 4. Prioritet Prilikom Izbora Posla

Scheduler ne bira goroutine nasumično.

Pojednostavljen redosled:

```
1. Local Run Queue

↓

2. Global Run Queue

↓

3. Work Stealing

↓

4. Netpoller

↓

5. Idle
```

---

Ovim redosledom se smanjuje overhead i povećava efikasnost.

---

# 5. Šta Je Work Stealing?

Work Stealing je mehanizam kojim jedan P:

```
"krade"

goroutine

od drugog P.
```

---

Primer:

```
P0

G1

G2

G3

G4

G5
```

---

```
P1

(empty)
```

---

Umesto da P1 miruje:

```
uzima deo posla

od P0.
```

---

# 6. Zašto Je Work Stealing Važan?

Bez ovog mehanizma:

```
P0

preopterećen

↓

P1

ne radi ništa
```

---

Sa Work Stealing-om:

```
P0

↓

G1 G2

P1

↓

G3 G4 G5
```

---

Rezultat:

- bolja iskorišćenost CPU-a
- manje čekanja
- ravnomernije opterećenje

---

# 7. Kako Funkcioniše Work Stealing?

Pojednostavljen algoritam:

```
Local Queue prazna?

↓

DA

↓

Pokušaj krađe

↓

Uspeh?

↓

DA → izvršavanje

NE → Global Queue
```

---

Ako ni tamo nema posla:

```
P postaje idle.
```

---

# 8. Koliko Posla Se Krade?

Scheduler obično:

```
ne uzima

ceo red.
```

---

Već:

```
deo goroutine-a
```

---

Na taj način:

- oba Processor-a ostaju zauzeta
- izbegava se često ponovno balansiranje

---

# 9. Load Balancing

Cilj scheduler-a:

```
da nijedan P

ne bude

preopterećen

ili

potpuno neaktivan.
```

---

Primer:

Loše:

```
P0

100 goroutine
```

```
P1

0
```

---

Dobro:

```
P0

50
```

```
P1

50
```

---

# 10. Scheduler Fairness

Scheduler pokušava da bude:

```
fer
```

---

To znači:

nijedna runnable goroutine ne bi trebalo da:

```
čeka beskonačno.
```

---

Važno:

Fairness nije isto što i:

```
deterministički redosled.
```

---

Go ne garantuje:

```
G1

↓

G2

↓

G3
```

---

Redosled zavisi od:

- scheduler-a
- blokiranja
- preempcije
- dostupnih P
- sistemskog opterećenja

---

# 11. Primer Izvršavanja

Program:

```go
go taskA()
go taskB()
go taskC()
```

---

Jedan mogući raspored:

```
P0

↓

taskA

↓

taskC
```

---

```
P1

↓

taskB
```

---

Drugi raspored:

```
P0

↓

taskB
```

---

```
P1

↓

taskA

↓

taskC
```

---

Oba su potpuno ispravna.

---

# 12. Kada Nastaje Neravnomerna Raspodela?

Primer:

```
Worker A

100 ms
```

---

```
Worker B

2 ms
```

---

Ako svi dugi poslovi završe na jednom P:

dobijamo:

```
imbalance
```

---

Scheduler pokušava da ga ispravi pomoću:

```
Work Stealing
```

---

# 13. Kada Work Stealing Nije Dovoljan?

Ako su sve goroutine:

```
Waiting
```

na primer:

- čekaju mrežu
- čekaju disk
- čekaju mutex

---

Nema:

```
Runnable

goroutine
```

za krađu.

---

Tada scheduler čeka:

- završetak I/O operacije
- otključavanje mutex-a
- signal sa channel-a
- istek tajmera

---

# 14. Saveti Za Pisanje Efikasnog Koda

✔ Nemoj praviti jednu ogromnu goroutine koja radi sav posao.

---

✔ Posao podeli na manje nezavisne zadatke.

---

✔ Izbegavaj nepotrebno blokiranje.

---

✔ Ne drži mutex duže nego što je potrebno.

---

✔ Koristi worker pool kada ima mnogo sličnih zadataka.

---

Takav kod scheduler može mnogo bolje da rasporedi.

---

# 15. Mentalni Model

Zamisli više blagajni u supermarketu.

```
Blagajna 1

↓↓↓

Kupci
```

---

Ako druga blagajna ostane prazna:

```
kupci prelaze

na slobodnu blagajnu.
```

---

To je veoma slično:

```
Work Stealing
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Run Queue

✅ Local Run Queue

✅ Global Run Queue

✅ Work Stealing

✅ Load Balancing

✅ Scheduler Fairness

✅ tipične scenarije raspoređivanja goroutine-a

---

# Go Scheduler Internals

## Deo #4 — Scheduler Loop, Blocking i Netpoller

---

# 📚 Sadržaj

- Scheduler Loop
- Kako scheduler bira sledeću goroutine
- Blocking operacije
- Syscalls
- Netpoller
- Buđenje goroutine-a
- Povratak u Run Queue

---

# 1. Scheduler Loop

Scheduler ne radi samo jednom.

On neprekidno izvršava glavnu petlju koja bira sledeću goroutine za izvršavanje.

Pojednostavljeno:

```
Pronađi Runnable Goroutine

↓

Izvrši

↓

Blokirana?

↓

DA → Waiting

↓

NE → Nastavi

↓

Završen posao?

↓

DA → Ukloni G

↓

NE → Scheduler ponovo odlučuje
```

---

Tokom rada scheduler stalno proverava:

- lokalni run queue
- globalni run queue
- work stealing
- netpoller
- sistemske događaje

---

# 2. Kako Scheduler Bira Sledeću Goroutine?

Pojednostavljeni algoritam izgleda ovako:

```
1.

Local Queue
```

↓

Ako nema posla:

```
2.

Global Queue
```

↓

Ako nema posla:

```
3.

Work Stealing
```

↓

Ako nema posla:

```
4.

Netpoller
```

↓

Ako ni tamo nema posla:

```
Processor postaje idle.
```

---

Ovaj redosled minimizuje overhead i poboljšava lokalnost podataka (cache locality).

---

# 3. Šta Znači Blocking?

Goroutine nije uvek aktivna.

Može biti blokirana dok čeka:

- prijem sa channel-a
- slanje na channel
- `sync.Mutex`
- `sync.RWMutex`
- `sync.Cond`
- mrežni I/O
- fajl I/O
- timer
- `context.Done()`

---

Primer:

```go
msg := <-ch
```

Ako kanal nema vrednost:

```
goroutine

↓

Waiting
```

---

Scheduler tada bira neku drugu runnable goroutine.

---

# 4. Šta Se Dešava Tokom Blokiranja?

Pretpostavimo:

```
G1

↓

čeka channel
```

---

Scheduler radi sledeće:

```
G1

↓

Waiting
```

↓

```
P

↓

uzima sledeću Runnable Goroutine
```

---

CPU ne ostaje besposlen.

---

To je jedna od najvećih prednosti Go scheduler-a.

---

# 5. Syscall Blokiranje

Neke operacije blokiraju OS thread.

Na primer:

- određeni sistemski pozivi (syscalls)
- deo disk I/O operacija
- određene C funkcije preko cgo

---

Scenario:

```
G1

↓

Syscall

↓

M blokiran
```

---

Ako bi scheduler čekao isti thread:

ceo sistem bi usporio.

---

# 6. Kako Scheduler Rešava Syscall?

Kada M uđe u blokirajući syscall:

```
M

↓

blokiran
```

---

Scheduler može:

```
odvojiti P

↓

dodeliti ga drugom M
```

---

Rezultat:

```
ostale goroutine

nastavljaju rad.
```

---

Pojednostavljeno:

```
Pre:

P0

↓

M0

↓

Syscall
```

---

Posle:

```
P0

↓

M1

↓

nastavlja izvršavanje
```

---

# 7. Šta Je Netpoller?

Veliki broj mrežnih operacija ne koristi poseban thread po konekciji.

Umesto toga Go Runtime koristi:

```
Netpoller
```

---

Njegov zadatak je:

- praćenje mrežnih događaja
- praćenje spremnosti socket-a
- buđenje odgovarajućih goroutine-a

---

To omogućava hiljade ili desetine hiljada istovremenih mrežnih konekcija bez istog broja OS thread-ova.

---

# 8. Kako Radi Netpoller?

Primer:

```
HTTP Request

↓

Read()

↓

Socket nije spreman
```

---

Goroutine prelazi u:

```
Waiting
```

---

Netpoller prati socket.

---

Kada podaci stignu:

```
Socket Ready

↓

Netpoller

↓

Runnable

↓

Run Queue
```

---

Scheduler zatim može ponovo izvršiti goroutine.

---

# 9. Buđenje Goroutine-a

Goroutine može postati Runnable kada:

- channel primi ili pošalje vrednost
- mutex bude otključan
- istekne timer
- `context` bude otkazan
- završi se syscall
- Netpoller prijavi mrežni događaj

---

Pojednostavljeno:

```
Waiting

↓

Event

↓

Runnable

↓

Scheduler

↓

Running
```

---

# 10. Povratak u Run Queue

Važno je razumeti:

goroutine koja se probudi:

```
ne nastavlja

odmah

sa izvršavanjem.
```

---

Prvo postaje:

```
Runnable
```

---

Zatim ulazi u:

```
Run Queue
```

---

Tek kada scheduler izabere tu goroutine:

```
Running
```

---

# 11. Zašto Je Ovo Efikasno?

Bez scheduler-a:

```
Thread

↓

čeka mrežu

↓

CPU ne radi ništa
```

---

Sa Go Runtime-om:

```
Goroutine čeka

↓

druga goroutine koristi CPU
```

---

Rezultat:

- veća iskorišćenost procesora
- više paralelnog rada
- manje neaktivnog vremena

---

# 12. Primer sa Više Goroutine-a

Pretpostavimo tri goroutine:

```
G1

↓

Network Read
```

---

```
G2

↓

CPU Work
```

---

```
G3

↓

Channel Receive
```

---

Tok događaja:

1. G1 čeka mrežu.
2. G2 koristi CPU.
3. G3 čeka channel.
4. Stižu mrežni podaci.
5. Netpoller označava G1 kao Runnable.
6. Scheduler bira G1 kada dođe na red.

---

# 13. Najčešće Zablude

❌ Blokirana goroutine blokira ceo program.

Netačno.

---

❌ Svaka mrežna konekcija koristi jedan OS thread.

Netačno.

---

❌ Runnable znači da se goroutine odmah izvršava.

Netačno.

Runnable znači samo:

```
spremna za izvršavanje.
```

---

❌ Scheduler aktivno čeka (busy waiting).

U najvećem broju slučajeva ne.

Koristi događaje, redove izvršavanja i mehanizme operativnog sistema kako bi izbegao nepotrebno trošenje CPU vremena.

---

# 14. Mentalni Model

Zamisli pozivni centar.

```
Operater

↓

čeka poziv
```

---

Dok nema poziva:

drugi operateri rade.

---

Kada zazvoni telefon:

```
Poziv

↓

Operater postaje aktivan
```

---

To je veoma slično radu:

- Netpoller-a
- Scheduler-a
- Run Queue mehanizma

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako radi scheduler loop

✅ kako scheduler bira sledeću goroutine

✅ šta znači blocking

✅ kako se obrađuju syscalls

✅ šta je Netpoller

✅ kako se goroutine budi

✅ kako se vraća u Run Queue

---

# Go Scheduler Internals

## Deo #5 — Preemption, Scheduler Tracing i Runtime Dijagnostika

---

# 📚 Sadržaj

- Šta je Preemption?
- Cooperative vs Asynchronous Preemption
- Zašto je Preemption važna?
- Scheduler Tracing
- GODEBUG promenljive
- Runtime Trace
- Dijagnostika Scheduler-a
- Najbolje prakse

---

# 1. Šta Je Preemption?

Preemption predstavlja mogućnost da Go Scheduler prekine izvršavanje jedne goroutine kako bi omogućio drugim goroutine-ama da koriste procesor.

Pojednostavljeno:

```
G1

↓

koristi CPU veoma dugo

↓

Scheduler prekida G1

↓

pokreće G2
```

---

Bez preemption-a, jedna goroutine bi mogla dugo da zauzima CPU i uspori izvršavanje ostalih goroutine-a.

---

# 2. Zašto Je Preemption Važna?

Pretpostavimo sledeći kod:

```go
func busyLoop() {
	for {
	}
}
```

---

Ako bi scheduler dozvolio da ova goroutine radi beskonačno:

```
G1

↓

∞
```

ostale goroutine bi mogle da čekaju veoma dugo.

---

Preemption omogućava da scheduler vrati kontrolu nad izvršavanjem.

---

# 3. Cooperative Preemption

U ranim verzijama Go-a scheduler se oslanjao uglavnom na cooperative preemption.

To znači da je goroutine ustupala kontrolu scheduler-u na određenim bezbednim tačkama (safe points), kao što su:

- pozivi funkcija
- pojedine runtime operacije
- određene sinhronizacione operacije

---

Ako goroutine nije dolazila do tih tačaka, mogla je dugo da zadrži procesor.

---

# 4. Asynchronous Preemption

Novije verzije Go-a uvode asynchronous preemption.

Scheduler može prekinuti izvršavanje goroutine čak i kada ona nije eksplicitno ustupila kontrolu, pod uslovom da je prekid bezbedan.

---

Rezultat:

- bolja raspodela CPU vremena
- manja latencija
- pravednije izvršavanje goroutine-a

---

# 5. Safe Points

Scheduler ne može prekinuti izvršavanje na potpuno proizvoljnom mestu.

Prekid mora biti bezbedan.

Takve lokacije nazivaju se:

```
Safe Points
```

---

Na njima runtime zna:

- stanje registra
- stanje stack-a
- stanje objekata koje prati Garbage Collector

---

To omogućava bezbedan nastavak izvršavanja.

---

# 6. Scheduler Tracing

Go Runtime može prikazati informacije o radu scheduler-a.

Jedan od načina je korišćenje promenljive:

```bash
GODEBUG=schedtrace=1000
```

---

Ona ispisuje stanje scheduler-a u zadatom intervalu (u milisekundama).

---

Primer:

```text
SCHED 1000ms:
gomaxprocs=8
idleprocs=2
threads=10
spinningthreads=1
idlethreads=3
runqueue=12
```

---

Ovakvi podaci pomažu u razumevanju ponašanja aplikacije.

---

# 7. Korisne GODEBUG Opcije

Neke često korišćene opcije:

```bash
GODEBUG=schedtrace=1000
```

Prikazuje stanje scheduler-a.

---

```bash
GODEBUG=schedtrace=1000,scheddetail=1
```

Prikazuje detaljnije informacije o scheduler-u.

---

Druge korisne opcije (u zavisnosti od verzije Go-a):

- GC dijagnostika
- inicijalizacija runtime-a
- određeni eksperimenti i razvojne opcije

---

# 8. Runtime Trace

Go omogućava veoma detaljno praćenje izvršavanja programa.

Primer:

```bash
go test -trace trace.out
```

---

Pregled:

```bash
go tool trace trace.out
```

---

Trace prikazuje:

- raspoređivanje goroutine-a
- blokiranja
- mrežne događaje
- Garbage Collector
- sistemske pozive

---

To je jedan od najmoćnijih alata za analizu konkurentnih programa.

---

# 9. Šta Možemo Videti U Trace-u?

Primer događaja:

```
Goroutine Created

↓

Runnable

↓

Running

↓

Blocked

↓

Runnable

↓

Running

↓

Finished
```

---

Takođe možemo analizirati:

- koliko dugo goroutine čeka
- koliko traje izvršavanje
- kada dolazi do blokiranja
- koliko traje Garbage Collection

---

# 10. Kada Koristiti Scheduler Tracing?

Scheduler tracing je koristan kada:

- CPU nije dovoljno iskorišćen
- postoji veliki broj goroutine-a
- dolazi do neočekivanih zastoja
- sumnjamo na contention
- želimo bolje razumeti ponašanje scheduler-a

---

# 11. Česte Zablude

❌ Scheduler bira goroutine potpuno nasumično.

Netačno.

Koristi više heuristika i prioriteta.

---

❌ Preemption se dešava posle tačno određenog vremena.

Netačno.

To zavisi od runtime-a, safe point-ova i trenutnog stanja sistema.

---

❌ Trace značajno menja ponašanje aplikacije.

Trace uvodi određeni overhead, ali je namenjen upravo analizi i dijagnostici.

---

# 12. Najbolje Prakse

✔ Koristi `go tool trace` kada želiš detaljno razumeti ponašanje konkurentnog programa.

---

✔ Koristi `GODEBUG=schedtrace=...` za brz pregled rada scheduler-a.

---

✔ Kombinuj trace sa:

- benchmark testovima
- CPU profilima
- Memory profilima

---

✔ Ne donosi zaključke na osnovu jednog izvršavanja.

Posmatraj više pokretanja i različita opterećenja.

---

# 13. Mentalni Model

Zamisli aerodrom.

```
Avioni

↓

Kontrola leta

↓

Pista

↓

Poletanje
```

---

Scheduler:

- odlučuje ko dobija pistu
- prati stanje saobraćaja
- sprečava zastoje
- održava red

---

Trace predstavlja:

```
snimak rada

kontrole leta.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je preemption

✅ cooperative i asynchronous preemption

✅ safe point-ove

✅ scheduler tracing

✅ `GODEBUG=schedtrace`

✅ `scheddetail`

✅ `runtime/trace`

✅ najbolje prakse za dijagnostiku scheduler-a

---

# Go Scheduler Internals

## Deo #6 — Scheduler Performance, Best Practices i Završni Rezime

---

# 📚 Sadržaj

- Scheduler i performanse
- Najčešće greške
- Scheduler-friendly kod
- Production preporuke
- Dijagnostika
- Završna kontrolna lista
- Dalje učenje

---

# 1. Kako Scheduler Utiče na Performanse?

Go Scheduler je optimizovan da efikasno raspoređuje veliki broj goroutine-a na mali broj OS thread-ova.

Ipak, način na koji pišemo kod direktno utiče na njegovu efikasnost.

Na primer:

```
100 000 kratkih zadataka

↓

veoma dobro
```

---

Nasuprot tome:

```
1 goroutine

↓

sav posao

↓

slabo iskorišćen CPU
```

---

Scheduler najbolje radi kada postoji dovoljno nezavisnih jedinica posla koje mogu da se rasporede između dostupnih `P` instanci.

---

# 2. Nemoj Praviti Previše Goroutine-a

Goroutine jesu jeftine, ali nisu besplatne.

Svaka goroutine ima:

- stack
- scheduler metadata
- GC overhead

---

Loš primer:

```go
for _, item := range items {
	go process(item)
}
```

Ako `items` sadrži milion elemenata:

```
1 000 000 goroutine-a
```

nije dobra ideja.

---

Bolje rešenje:

```
Worker Pool

↓

ograničen broj worker-a

↓

Queue
```

---

# 3. Izbegavaj Dugo Držanje Lock-a

Loše:

```go
mu.Lock()

time.Sleep(time.Second)

mu.Unlock()
```

---

Za to vreme:

```
ostale goroutine

↓

Waiting
```

---

Bolje:

- zaključati što kasnije
- otključati što ranije
- kritičnu sekciju držati što kraćom

---

# 4. Izbegavaj Nepotrebno Blokiranje

Primer:

```go
<-ch
```

Ako nema pošiljaoca:

```
Waiting
```

---

Ako se ovakav obrazac često pojavljuje:

scheduler provodi više vremena upravljajući blokiranim goroutine-ama nego izvršavajući koristan posao.

---

# 5. Koristi Context za Lifecycle

Dugotrajne goroutine treba da imaju jasan način završetka.

Primer:

```go
select {
case <-ctx.Done():
	return
default:
	work()
}
```

---

Bez toga lako nastaju:

- goroutine leak
- curenje memorije
- nepotrebno zauzimanje resursa

---

# 6. Ne Oslanjaj Se na Redosled Izvršavanja

Pogrešna pretpostavka:

```go
go A()
go B()
```

↓

```
A

pa

B
```

---

Go to ne garantuje.

Ispravno razmišljanje:

```
Scheduler

odlučuje.
```

---

Ako je redosled važan:

koristi:

- channel
- `sync.WaitGroup`
- `sync.Cond`
- druge sinhronizacione mehanizme

---

# 7. Optimizuj Granularnost Posla

Premali zadaci:

```
više scheduler overhead-a
```

---

Preveliki zadaci:

```
loš load balancing
```

---

Cilj:

```
umerena veličina task-a
```

koja omogućava dobro iskorišćenje procesora uz mali overhead.

---

# 8. Prati Performanse

Nemoj optimizovati naslepo.

Koristi dostupne alate:

```bash
go test -bench=.
```

---

```bash
go test -benchmem
```

---

```bash
go test -cpuprofile cpu.out
```

---

```bash
go test -memprofile mem.out
```

---

```bash
go test -trace trace.out
```

---

Kombinovanjem benchmark-a, profila i trace-a dobija se realna slika ponašanja aplikacije.

---

# 9. Production Saveti

✔ Koristi worker pool za veliki broj kratkih zadataka.

---

✔ Izbegavaj globalne mutex-e kada postoji velika konkurentnost.

---

✔ Koristi `context.Context` za kontrolu životnog ciklusa goroutine-a.

---

✔ Ograniči broj paralelnih operacija kada koristiš spoljne resurse (baze podataka, HTTP servise, fajlove).

---

✔ Dodaj metrike:

- broj goroutine-a
- dužinu redova
- latenciju
- iskorišćenost worker-a
- broj grešaka

---

# 10. Najčešće Greške

❌ Previše goroutine-a.

---

❌ Goroutine bez izlaznog uslova.

---

❌ Dugo držanje lock-a.

---

❌ Pretpostavljanje redosleda izvršavanja.

---

❌ Ignorisanje `context` otkazivanja.

---

❌ Neprofilisanje aplikacije.

---

❌ Optimizacija bez merenja.

---

# 11. Scheduler-Friendly Checklist

Pre objavljivanja aplikacije proveri:

```
□ nema race condition-a

□ nema deadlock-a

□ nema goroutine leak-a

□ kritične sekcije su kratke

□ koristi se context

□ worker pool je ograničen

□ postoji graceful shutdown

□ postoje benchmark testovi

□ postoji profiling

□ postoji observability
```

---

# 12. Kako Dalje Učiti?

Preporučeni izvori:

- Go Memory Model
- Go Runtime
- Garbage Collector
- `runtime` paket
- `runtime/metrics`
- `runtime/trace`
- `sync` i `sync/atomic`
- izvorni kod Go Runtime-a

---

Posebno je korisno proučiti implementacije:

- scheduler-a
- channel-a
- mutex-a
- garbage collector-a

---

# 13. Završni Mentalni Model

Najjednostavnije je razmišljati ovako:

```
Goroutine

=

Posao
```

↓

```
Processor (P)

=

Dispečer
```

↓

```
Machine (M)

=

Radnik
```

↓

```
Scheduler

=

Koordinator
```

---

Koordinator:

- raspoređuje posao
- vodi računa o opterećenju
- reaguje na blokiranja
- održava sistem efikasnim

---

# 14. Završni Rezime Modula

Tokom svih šest delova naučili smo:

✅ šta je Go Scheduler

✅ G–M–P model

✅ životni ciklus goroutine-a

✅ lokalne i globalne run queue

✅ work stealing

✅ scheduler loop

✅ blokiranje i čekanje

✅ syscalls

✅ Netpoller

✅ preemption

✅ scheduler tracing

✅ `GODEBUG`

✅ `runtime/trace`

✅ scheduler-friendly dizajn

✅ production preporuke

---

# 15. Ključne Lekcije

Najvažnije stvari koje treba zapamtiti:

- Goroutine nisu OS thread-ovi.
- Scheduler upravlja raspoređivanjem goroutine-a.
- `P` predstavlja izvršni kontekst (execution context), a ne fizičko CPU jezgro.
- Scheduler preferira lokalne redove izvršavanja zbog boljih performansi.
- Work stealing poboljšava balans opterećenja.
- Blokirana goroutine ne blokira ceo program.
- `context.Context` je osnovni mehanizam za kontrolu životnog ciklusa goroutine-a.
- Profilisanje i merenje performansi treba da prethode optimizaciji.

---

# 🎓 Završetak Modula

Čestitamo!

Uspešno si završio modul **Go Scheduler Internals**.

Sada poseduješ čvrstu osnovu za razumevanje načina na koji Go Runtime izvršava konkurentne programe, što će ti pomoći pri projektovanju brzih, skalabilnih i pouzdanih Go aplikacija.

---

# 📋 Završni Rezime Dokumenta

Ovaj dokument obuhvatio je:

✅ arhitekturu G–M–P modela

✅ lokalne i globalne run queue

✅ work stealing algoritam

✅ scheduler loop

✅ blocking i syscalls

✅ Netpoller

✅ preemption

✅ scheduler tracing i `runtime/trace`

✅ performance preporuke

✅ scheduler-friendly dizajn

---

### ➡️ Sledeća lekcija **[**Common Concurrency Mistakes**](13-common-concurrency-mistakes.md)**

Obuhvatiće:

- najčešće greške pri radu sa goroutine-ama
- pogrešnu upotrebu channel-a
- race condition-e
- deadlock scenarije
- goroutine leak-ove
- pogrešnu upotrebu `context.Context`
- greške sa `sync.WaitGroup`, `Mutex`, `RWMutex` i `sync.Once`
- primere iz produkcionih sistema i preporučene načine rešavanja.