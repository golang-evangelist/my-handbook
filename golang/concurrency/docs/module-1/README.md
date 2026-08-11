# Module 1 — Concurrency Fundamentals

Module 1 predstavlja temelj celog **Go Concurrency** curriculum-a.

Cilj ovog modula nije da čitalac samo nauči kako da napiše:

```go
go worker()
```

već da razume šta goroutine predstavlja, kako goroutine-i komuniciraju, šta znači blocking, kako funkcionišu unbuffered i buffered channels, kako se definiše channel direction i kako se kontroliše channel lifecycle.

Modul uvodi najvažnije primitive Go concurrency modela i postavlja mentalni model koji će se koristiti u svim narednim modulima.

---

# Module 1 at a Glance

```text
Module 1
│
├── Goroutines
│
├── Channels
│
├── Unbuffered Channels
│
├── Buffered Channels
│
├── Channel Directions
│
├── Range Over Channels
│
├── Close Channel
│
└── Summary & Exercises
```

Struktura modula:

```text
module-1/
│
├── README.md
├── 01-goroutines.md
├── 02-channels.md
├── 03-unbuffered-channels.md
├── 04-buffered-channels.md
├── 05-channel-directions.md
├── 06-range-over-channels.md
├── 07-close-channel.md
└── 08-module-1-summary-and-exercises.md
```

---

# Cilj modula

Nakon završetka Module 1 čitalac treba da razume osnovni Go concurrency model:

```text
Goroutine
    │
    ▼
Communication
    │
    ▼
Channel
    │
    ▼
Another Goroutine
```

Ali ovaj model treba razumeti mnogo dublje od same sintakse.

Potrebno je znati:

* šta se dešava kada se goroutine pokrene;
* kako goroutine lifecycle izgleda;
* šta znači da je operacija blocking;
* kada send blokira;
* kada receive blokira;
* kako unbuffered channel ostvaruje synchronization;
* kako buffered channel menja blocking behavior;
* šta znači channel direction;
* ko treba da zatvori channel;
* kako receiver zna da više neće biti vrednosti;
* kako `range` funkcioniše nad channel-om;
* šta se dešava sa zatvorenim channel-om;
* kako sprečiti goroutine leaks;
* kako prepoznati osnovne deadlock scenarije.

---

# Learning Objectives

Po završetku modula trebalo bi da budeš sposoban da:

## Goroutines

* pokreneš goroutine;
* razumeš razliku između regularnog function call-a i goroutine invocation-a;
* razumeš da `go` statement ne predstavlja sinhrono izvršavanje;
* objasniš lifecycle goroutine-a;
* razumeš zašto `main` može završiti pre drugih goroutine-a;
* prepoznaš osnovne lifecycle probleme.

## Channels

* kreiraš channel;
* šalješ vrednost kroz channel;
* primaš vrednost sa channel-a;
* razumeš blocking behavior;
* koristiš channel za komunikaciju između goroutine-a;
* razlikuješ send i receive operacije.

## Unbuffered Channels

* objasniš rendezvous semantics;
* razumeš kada send blokira;
* razumeš kada receive blokira;
* koristiš unbuffered channel kao synchronization point.

## Buffered Channels

* kreiraš buffered channel;
* razumeš kapacitet i trenutno stanje buffer-a;
* razlikuješ buffer capacity od queue length-a;
* razumeš kada send blokira;
* razumeš kada receive blokira;
* razumeš trade-off buffered channels.

## Channel Directions

* koristiš bidirectional channel;
* koristiš send-only channel;
* koristiš receive-only channel;
* koristiš directional channels u function signatures;
* razumeš kako directional channels izražavaju API contract.

## Range

* iteriraš preko channel-a;
* razumeš kada `range` završava;
* povežeš `range` sa channel closing semantics.

## Close

* pravilno zatvoriš channel;
* razumeš šta `close` znači;
* proveriš da li je channel zatvoren;
* razumeš zero-value behavior nakon zatvaranja;
* prepoznaš pogrešno zatvaranje channel-a.

---

# Prerequisites

Pre početka Module 1 preporučuje se poznavanje:

* Go syntax;
* variables;
* functions;
* function calls;
* pointers;
* structs;
* interfaces;
* slices;
* maps;
* basic error handling;
* packages;
* osnovnog rada sa `go run` i `go test`.

Prethodno iskustvo sa concurrency-jem nije potrebno.

---

# Centralni Mentalni Model

Module 1 uvodi tri osnovna koncepta:

```text
Execution
    │
    ▼
Goroutines

Communication
    │
    ▼
Channels

Lifecycle
    │
    ▼
Close / Range
```

Ova tri koncepta treba posmatrati zajedno.

Na primer:

```text
Producer Goroutine
       │
       │ send()
       ▼
    Channel
       │
       │ receive()
       ▼
Consumer Goroutine
```

Ovo je jedan od najvažnijih obrazaca u Go concurrency-ju.

Ali odmah se pojavljuje pitanje:

> Šta se dešava ako producer šalje, a consumer još nije spreman?

Odgovor zavisi od toga da li je channel:

```text
unbuffered
```

ili:

```text
buffered
```

Zbog toga se u modulu buffered i unbuffered channels obrađuju odvojeno.

---

# 1. Goroutines

Prva lekcija:

```text
01-goroutines.md
```

uvodi osnovni execution model.

Goroutine predstavlja funkciju koja se izvršava concurrent u odnosu na pozivajući kod.

Primer:

```go
go worker()
```

Za razliku od:

```go
worker()
```

`go` statement omogućava da poziv bude pokrenut kao goroutine.

Mentalni model:

```text
Regular Call

main
 │
 ▼
worker()
 │
 ▼
return
```

nasuprot:

```text
Goroutine

main
 │
 ├──────────────► worker()
 │
 ▼
continues
```

Ovo uvodi prvu važnu činjenicu:

> Pokretanje goroutine-a ne znači da pozivajući goroutine čeka da se nova goroutine završi.

---

## Goroutine Lifecycle

Goroutine može biti posmatrana kroz lifecycle:

```text
Created
   │
   ▼
Runnable
   │
   ▼
Running
   │
   ├── Blocked
   │     │
   │     └── Runnable
   │
   ▼
Completed
```

U Module 1 fokus nije na internals scheduler-a.

Za sada je dovoljno razumeti da goroutine:

* može biti pokrenuta;
* može biti runnable;
* može blokirati;
* može nastaviti izvršavanje;
* može završiti.

Detaljna analiza scheduler-a biće obrađena kasnije u Module 3 i Module 4.

---

# `main` Goroutine

Posebno važan koncept:

```go
func main() {
    go worker()
}
```

Ne postoji implicitna garancija da će `worker()` završiti pre nego što `main` završi.

Ako `main` završi:

```text
main goroutine
      │
      ▼
   returns
      │
      ▼
process exits
```

preostale goroutine-e ne dobijaju priliku da nastave kao nezavisni procesi.

Zato:

```text
go worker()
```

nije isto što i:

```text
wait for worker
```

Ova razlika će kasnije biti rešavana pomoću različitih synchronization mehanizama.

---

# Goroutines Are Not OS Processes

Goroutine nije:

* OS process;
* OS thread koji developer ručno upravlja;
* potpuno nezavisni program.

Go runtime upravlja goroutine-ama.

Pojednostavljen model:

```text
Many Goroutines
       │
       ▼
  Go Runtime
       │
       ▼
   Scheduler
       │
       ▼
  OS Threads
```

U Module 1 nije potrebno ulaziti u detalje scheduler-a.

Bitno je razumeti:

> Developer opisuje concurrent work, a Go runtime upravlja njegovim izvršavanjem.

---

# Goroutine Communication

Samo pokretanje više goroutine-a nije dovoljno.

Ako goroutine-i treba da sarađuju, moraju imati način komunikacije ili synchronization-a.

Module 1 uvodi channels kao centralni communication mechanism.

```text
Producer
    │
    ▼
 Channel
    │
    ▼
Consumer
```

Primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

Ovde postoje dve operacije:

```text
ch <- 42
```

send

i:

```text
value := <-ch
```

receive.

---

# 2. Channels

Druga lekcija:

```text
02-channels.md
```

uvodi channels kao typed communication mechanism.

Channel se kreira pomoću:

```go
ch := make(chan int)
```

Tip channel-a je:

```text
chan int
```

što znači da channel prenosi vrednosti tipa `int`.

Drugi primeri:

```go
chan string
chan bool
chan MyStruct
chan *User
```

Channel je typed.

To znači da compiler može proveriti da li se šalje odgovarajuća vrsta vrednosti.

---

# Send Operation

Send se zapisuje:

```go
ch <- value
```

Na primer:

```go
ch <- 42
```

Mentalni model:

```text
Sender
   │
   │ 42
   ▼
Channel
```

---

# Receive Operation

Receive:

```go
value := <-ch
```

Mentalni model:

```text
Channel
   │
   │ 42
   ▼
Receiver
```

Može se koristiti i bez čuvanja rezultata:

```go
<-ch
```

---

# Channel as Communication Boundary

Channel treba posmatrati kao boundary između concurrent activities.

```text
Goroutine A
     │
     │ send()
     ▼
  CHANNEL
     │
     │ receive()
     ▼
Goroutine B
```

Ovo je veoma važan mentalni model zato što omogućava razdvajanje:

* producer-a;
* consumer-a;
* ownership-a;
* processing-a;
* lifecycle-a.

Na primer:

```text
Producer
  │
  ├── creates work
  │
  ▼
Channel
  │
  ├── transports work
  │
  ▼
Worker
  │
  └── processes work
```

---

# Blocking Semantics

Najvažniji koncept kod channels je blocking.

Channel operacija nije samo:

```text
send data
```

ili:

```text
receive data
```

Ona može predstavljati i synchronization point.

Kod unbuffered channel-a:

```go
ch <- value
```

može blokirati dok receiver nije spreman.

A:

```go
value := <-ch
```

može blokirati dok sender nije spreman.

To znači da channel istovremeno predstavlja:

```text
Communication
+
Synchronization
```

Ova osobina postaje posebno važna kod unbuffered channels.

---

# 3. Unbuffered Channels

Treća lekcija:

```text
03-unbuffered-channels.md
```

uvodi channels bez buffer-a.

Kreiranje:

```go
ch := make(chan int)
```

Channel ima:

```text
capacity = 0
```

To znači da ne postoji prostor u koji send može samo da smesti vrednost i odmah nastavi.

Umesto toga, sender i receiver moraju da se usklade.

Mentalni model:

```text
Sender
   │
   │ send(42)
   │
   ▼
 rendezvous
   ▲
   │ receive(42)
   │
Receiver
```

Ovo se često opisuje kao **rendezvous**.

---

# Unbuffered Send

Ako sender izvrši:

```go
ch <- 42
```

a receiver još nije spreman:

```text
Sender
  │
  │ send(42)
  ▼
BLOCKED
```

Sender nastavlja tek kada odgovarajući receive može da se izvrši.

---

# Unbuffered Receive

Ako receiver izvrši:

```go
value := <-ch
```

a nema dostupnog sender-a:

```text
Receiver
   │
   │ receive
   ▼
 BLOCKED
```

Receiver nastavlja kada sender pošalje vrednost.

---

# Unbuffered Channel as Synchronization

Ovo je jedna od ključnih osobina:

```text
Sender
   │
   │ send
   ▼
Channel
   │
   │ receive
   ▼
Receiver
```

Send i receive moraju biti koordinisani.

Zato unbuffered channel može predstavljati vrlo precizan synchronization point.

Na primer:

```go
done := make(chan struct{})

go func() {
    doWork()
    done <- struct{}{}
}()

<-done
```

Glavna goroutine čeka dok worker ne pošalje signal.

Ovde channel ne prenosi "koristan podatak".

On prenosi **signal**.

---

# Zero-Sized Signaling

Za signalizaciju se često koristi:

```go
chan struct{}
```

jer `struct{}` nema podatke koje treba preneti.

Primer:

```go
done := make(chan struct{})

go func() {
    defer close(done)

    doWork()
}()
```

Receiver:

```go
<-done
```

U ovom primeru channel može predstavljati completion signal.

Kasnije će biti detaljnije obrađeni cancellation i lifecycle patterns.

---

# 4. Buffered Channels

Četvrta lekcija:

```text
04-buffered-channels.md
```

uvodi buffered channels.

Kreiranje:

```go
ch := make(chan int, 3)
```

Channel sada ima:

```text
capacity = 3
```

Mentalni model:

```text
     Sender
       │
       ▼
┌───────────────┐
│ 42 │ 43 │ 44  │
└───────────────┘
       │
       ▼
    Receiver
```

Sender ne mora odmah imati receiver-a sve dok postoji slobodan prostor u buffer-u.

---

# Buffered Send

Ako je buffer:

```text
capacity = 3
length = 0
```

onda:

```go
ch <- 42
```

može odmah da se izvrši.

Stanje postaje:

```text
capacity = 3
length = 1
```

Ako se pošalju još dve vrednosti:

```text
capacity = 3
length = 3
```

buffer je pun.

Sledeći send:

```go
ch <- 45
```

moraće da čeka dok receiver ne oslobodi prostor.

---

# Buffered Receive

Ako buffer sadrži:

```text
[42, 43, 44]
```

receive:

```go
value := <-ch
```

uzima sledeću vrednost.

Nakon toga:

```text
[43, 44]
```

Buffer ima slobodan prostor i blocked sender može eventualno nastaviti.

---

# Capacity vs Length

Važno je razlikovati:

```text
capacity
```

od:

```text
current number of buffered elements
```

Za:

```go
ch := make(chan int, 10)
```

možemo imati:

```text
cap(ch) == 10
len(ch) == 0
```

nakon jednog send-a:

```text
cap(ch) == 10
len(ch) == 1
```

nakon pet send-ova:

```text
cap(ch) == 10
len(ch) == 5
```

`len(ch)` predstavlja trenutno stanje buffer-a, dok `cap(ch)` predstavlja njegov maksimalni kapacitet.

---

# Buffered vs Unbuffered

Najvažnija razlika:

| Property               |                   Unbuffered |                     Buffered |
| ---------------------- | ---------------------------- | ---------------------------- |
| Capacity               |                          `0` |                        `> 0` |
| Internal queue         |                           Ne |                           Da |
| Send without receiver  |                           Ne |      Da, dok buffer nije pun |
| Receive without sender |                           Ne |   Da, dok buffer nije prazan |
| Synchronization        |                     Direktna | Delimično odvojena buffer-om |
| Typical use            | Rendezvous / synchronization |        Queueing / decoupling |
| ---------------------- | ---------------------------- | ---------------------------- |

Ne treba zaključiti:

```text
buffered = better
```

ili:

```text
unbuffered = better
```

Izbor zavisi od communication i lifecycle modela.

---

# Buffering as Decoupling

Buffered channel može da odvoji producer i consumer u vremenu.

Bez buffer-a:

```text
Producer ───────── Consumer
        synchronization
```

Sa buffer-om:

```text
Producer
   │
   ▼
Buffer
   │
   ▼
Consumer
```

Producer može privremeno da nastavi čak i ako consumer trenutno nije spreman.

To je korisno, ali uvodi dodatna pitanja:

* koliki buffer treba?
* šta ako se buffer napuni?
* da li producer treba da blokira?
* da li se gubi backpressure?
* da li buffer samo skriva downstream problem?
* koliko memorije buffer zauzima?

Zbog toga buffer size treba tretirati kao deo dizajna, a ne kao proizvoljan broj.

---

# 5. Channel Directions

Peta lekcija:

```text
05-channel-directions.md
```

uvodi directional channels.

Postoje tri relevantna oblika:

```text
chan T
<-chan T
chan<- T
```

---

## Bidirectional Channel

```go
chan int
```

omogućava:

```text
send
receive
```

---

## Receive-Only Channel

```go
<-chan int
```

omogućava samo:

```text
receive
```

Primer:

```go
func consume(ch <-chan int) {
    for value := range ch {
        fmt.Println(value)
    }
}
```

Unutar funkcije ne možemo poslati vrednost na `ch`.

---

## Send-Only Channel

```go
chan<- int
```

omogućava samo:

```text
send
```

Primer:

```go
func produce(ch chan<- int) {
    ch <- 42
}
```

Unutar funkcije ne možemo primiti vrednost sa `ch`.

---

# Directional Channels as API Contracts

Directional channel-i nisu samo compiler convenience.

Oni predstavljaju **API contract**.

Na primer:

```go
func producer() <-chan int
```

jasno govori:

> Ova funkcija vraća channel iz kojeg caller treba da čita.

I:

```go
func worker(jobs <-chan Job)
```

govori:

> Worker prima posao; nije odgovoran za slanje novih poslova na ovaj channel.

Directional channels tako izražavaju nameru direktno u type system-u.

---

# Conversion from Bidirectional to Directional

Bidirectional channel može biti prosleđen funkciji koja zahteva užu sposobnost.

Na primer:

```go
ch := make(chan int)

produce(ch)
consume(ch)
```

gde:

```go
func produce(ch chan<- int) {
    ch <- 42
}
```

i:

```go
func consume(ch <-chan int) {
    fmt.Println(<-ch)
}
```

Caller poseduje bidirectional channel:

```text
chan int
```

dok funkcije dobijaju samo capability koji im je potreban:

```text
producer → chan<- int
consumer → <-chan int
```

Ovo je koristan design pattern za smanjenje mogućnosti greške.

---

# 6. Range Over Channels

Šesta lekcija:

```text
06-range-over-channels.md
```

uvodi iteraciju preko channel-a.

Primer:

```go
for value := range ch {
    process(value)
}
```

Ovaj oblik je posebno koristan za stream processing.

Mentalni model:

```text
Producer
   │
   ├── value
   ├── value
   ├── value
   └── close
        │
        ▼
Consumer
   │
   └── range
```

`range` nastavlja da prima vrednosti sve dok channel nije zatvoren i dok nema više vrednosti za čitanje.

---

# Range Requires Lifecycle Semantics

Ako consumer koristi:

```go
for value := range ch {
    // ...
}
```

postavlja se pitanje:

> Ko zatvara `ch`?

Ako producer nikada ne zatvori channel, consumer može ostati da čeka zauvek kada više nema novih vrednosti.

Zato:

```text
range
  +
close
```

treba posmatrati kao lifecycle pair.

Primer:

```go
func producer() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 10; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Consumer:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Ovde je lifecycle jasan:

```text
create
  ↓
send values
  ↓
close
  ↓
range terminates
```

---

# 7. Close Channel

Sedma lekcija:

```text
07-close-channel.md
```

obrađuje channel closing semantics.

Channel se zatvara:

```go
close(ch)
```

Osnovno značenje:

> Na channel više neće biti poslato novih vrednosti.

To je signal receiver-u da je stream završen.

---

# Close Is a Sender-Side Responsibility

Uobičajeni ownership princip:

```text
Producer
   │
   ├── creates channel
   ├── sends values
   └── closes channel
            │
            ▼
        Consumer
```

Consumer obično ne zatvara channel koji nije njegov za ownership.

Posebno treba izbegavati situaciju:

```text
Producer A ──┐
Producer B ──┼── Channel
Producer C ──┘
```

gde više producer-a pokušava da odlučuje kada je channel završen.

To može dovesti do:

```text
panic: close of closed channel
```

ili do slanja na već zatvoren channel-a:

```text
panic: send on closed channel
```

Zbog toga channel ownership mora biti eksplicitno dizajniran.

---

# Receiving from a Closed Channel

Kada je channel zatvoren i više nema buffered vrednosti, receive može da završi odmah sa zero value-om i indikatorom da je channel zatvoren.

Primer:

```go
value, ok := <-ch

if !ok {
    // channel is closed
}
```

Za channel tipa:

```go
chan int
```

zero value je:

```text
0
```

Zato je bitan drugi rezultat:

```text
ok == false
```

Bez njega nije moguće razlikovati:

```text
received actual 0
```

od:

```text
channel is closed and drained
```

---

# Range and Closed Channels

`range` sakriva eksplicitnu proveru `ok`.

Umesto:

```go
for {
    value, ok := <-ch

    if !ok {
        break
    }

    process(value)
}
```

može se koristiti:

```go
for value := range ch {
    process(value)
}
```

To je jedan od najčešćih idiomatskih oblika channel consumption-a u Go-u.

---

# Closed Channel vs Nil Channel

Važno je razlikovati:

```text
nil channel
```

od:

```text
closed channel
```

To nisu ista stanja.

Konceptualno:

```text
nil channel
    │
    └── channel value does not reference an initialized channel

closed channel
    │
    └── initialized channel whose send side is permanently closed
```

Njihov blocking behavior se razlikuje i ova razlika postaje veoma važna kasnije kada se koristi `select`.

---

# Module 1 Design Principles

Tokom celog modula treba imati sledeća pravila na umu.

## Principle 1 — Goroutines Have Lifetimes

Svaka goroutine treba da ima jasan odgovor na:

```text
Who starts me?
Who stops me?
What do I wait for?
What happens on failure?
```

---

## Principle 2 — Channels Have Owners

Za svaki channel treba biti jasno:

```text
Who creates it?
Who sends?
Who receives?
Who closes?
```

---

## Principle 3 — Blocking Is a Design Decision

Ako operacija može da blokira, treba znati:

```text
Why can it block?
For how long?
Who unblocks it?
What happens if the other side never arrives?
```

---

## Principle 4 — Buffer Size Is Architecture

Buffered channel:

```go
make(chan Job, 100)
```

nije samo optimizacioni detalj.

Kapacitet može uticati na:

* throughput;
* latency;
* memory usage;
* backpressure;
* failure behavior.

---

## Principle 5 — Directional Channels Express Intent

Koristi:

```go
<-chan T
```

i:

```go
chan<- T
```

kada API treba da ograniči capability caller-a.

---

# Module 1 Conceptual Progression

Kompletan progression izgleda ovako:

```text
01 Goroutines
      │
      ▼
02 Channels
      │
      ▼
03 Unbuffered Channels
      │
      ▼
04 Buffered Channels
      │
      ▼
05 Channel Directions
      │
      ▼
06 Range
      │
      ▼
07 Close
      │
      ▼
08 Summary & Exercises
```

Svaka tema rešava novi deo problema.

```text
Goroutines
    │
    └── How do we execute concurrently?
             │
             ▼
Channels
    │
    └── How do goroutines communicate?
             │
             ▼
Unbuffered
    │
    └── How do sender and receiver synchronize?
             │
             ▼
Buffered
    │
    └── How can communication be temporarily decoupled?
             │
             ▼
Directions
    │
    └── How do we constrain channel capabilities?
             │
             ▼
Range
    │
    └── How do we consume a stream?
             │
             ▼
Close
    │
    └── How does the stream terminate?
```

---

# What You Should Be Able to Explain

Pre prelaska na Module 2 trebalo bi da možeš bez gledanja dokumentacije da objasniš:

### Goroutines

> Šta se dešava kada napišemo `go f()`?

### Channels

> Šta predstavljaju send i receive operacije?

### Unbuffered

> Zašto unbuffered send može da blokira?

### Buffered

> Kada buffered send može da nastavi bez receiver-a?

### Direction

> Koja je razlika između `chan T`, `<-chan T` i `chan<- T`?

### Range

> Kada `for range` preko channel-a završava?

### Close

> Šta `close(ch)` zapravo znači?

### Lifecycle

> Ko je odgovoran za završetak channel stream-a?

Ako na ova pitanja možeš precizno odgovoriti, imaš osnovu potrebnu za sledeći nivo concurrency-ja.

---

# Module 1 → Module 2

Module 1 završava osnovni communication model:

```text
Goroutine
   │
   ▼
Channel
   │
   ▼
Communication
   │
   ▼
Lifecycle
```

Module 2 će na ovaj model dodati:

```text
select
WaitGroup
Worker Pools
Pipelines
Fan-Out
Fan-In
```

što vodi ka:

```text
Multiple Goroutines
        │
        ▼
Coordination
        │
        ▼
Concurrency Patterns
```

Drugim rečima:

> **Module 1 uči kako goroutine-i komuniciraju; Module 2 uči kako veliki broj goroutine-a koordinisati.**

---

## Navigation

|  # | Lesson                                                        | Focus                        |
| -: | ------------------------------------------------------------- | ---------------------------- |
|  1 | [Goroutines](./01-goroutines.md)                              | Concurrent execution         |
|  2 | [Channels](./02-channels.md)                                  | Goroutine communication      |
|  3 | [Unbuffered Channels](./03-unbuffered-channels.md)            | Rendezvous & synchronization |
|  4 | [Buffered Channels](./04-buffered-channels.md)                | Buffering & decoupling       |
|  5 | [Channel Directions](./05-channel-directions.md)              | Type-level API constraints   |
|  6 | [Range Over Channels](./06-range-over-channels.md)            | Stream consumption           |
|  7 | [Close Channel](./07-close-channel.md)                        | Channel lifecycle            |
|  8 | [Summary & Exercises](./08-module-1-summary-and-exercises.md) | Consolidation                |
| -: | ------------------------------------------------------------- | ---------------------------- |

---

# Module 1 — Concurrency Fundamentals

## 8. Deep Conceptual Model

Module 1 treba posmatrati kao jedan povezan sistem, a ne kao skup nepovezanih API-ja.

Osnovni elementi su:

```text
┌─────────────┐
│  Goroutine  │
└──────┬──────┘
       │
       │ communication()
       ▼
┌─────────────┐
│   Channel   │
└──────┬──────┘
       │
       │ communication()
       ▼
┌─────────────┐
│  Goroutine  │
└─────────────┘
```

Ali channel nije samo transport vrednosti.

U zavisnosti od konfiguracije i korišćenja, channel može predstavljati:

* communication mechanism;
* synchronization point;
* work queue;
* completion signal;
* stream;
* ownership boundary;
* lifecycle signal.

Zbog toga razumevanje channel-a zahteva razumevanje njegovog ponašanja u različitim stanjima.

---

# Channel State Model

Channel treba posmatrati kroz nekoliko mogućih stanja:

```text
                    ┌────────────┐
                    │    nil     │
                    └─────┬──────┘
                          │ make
                          ▼
                    ┌────────────┐
                    │   Open     │
                    └─────┬──────┘
                          │ close
                          ▼
                    ┌────────────┐
                    │   Closed   │
                    └────────────┘
```

Za buffered channel postoji dodatna dimenzija:

```text
Open Channel
     │
     ├── Empty
     │
     ├── Partially Full
     │
     └── Full
```

Blocking behavior zavisi od kombinacije:

```text
channel state
+
buffer capacity
+
buffer occupancy
+
operation
```

---

# Send Semantics

Za send:

```go
ch <- value
```

treba postaviti sledeća pitanja:

1. Da li je channel `nil`?
2. Da li je channel otvoren?
3. Da li postoji receiver?
4. Ako je buffered, da li postoji slobodan prostor?
5. Ako ne postoji uslov za nastavak, ko će omogućiti nastavak?

Ovo daje sledeći konceptualni model:

```text
                  SEND
                    │
                    ▼
              Is channel nil?
                /       \
              yes        no
               │          │
               ▼          ▼
            blocks    Is channel closed?
                          /       \
                        yes        no
                         │          │
                         ▼          ▼
                       panic    Can send proceed?
                                  /      \
                                yes       no
                                 │        │
                                 ▼        ▼
                                send    blocks
```

Ovaj model je mnogo važniji od memorisanja pojedinačnih pravila.

---

# Receive Semantics

Za receive:

```go
value := <-ch
```

postavljamo slična pitanja:

```text
                    RECEIVE
                       │
                       ▼
                 Is channel nil?
                   /       \
                 yes        no
                 │           │
                 ▼           ▼
              blocks    Are values available?
                              /       \
                            yes         no
                             │           │
                             ▼           ▼
                          receive    Is channel closed?
                                       /       \
                                     yes        no
                                     │           │
                                     ▼           ▼
                               zero value    blocks
                               + ok=false
```

Ovo je osnovni model koji će biti posebno koristan kada se kasnije uvede `select`.

---

# Nil Channels

Nil channel nastaje kada je channel deklarisan bez inicijalizacije:

```go
var ch chan int
```

Tada:

```go
ch == nil
```

Send:

```go
ch <- 42
```

blokira.

Receive:

```go
value := <-ch
```

takođe blokira.

To znači da nil channel nije isto što i:

```go
make(chan int)
```

i nije isto što i zatvoren channel.

---

# Nil Channel vs Open Channel vs Closed Channel

| State            | Send                            | Receive                                   | Close      |
| ---------------- | ------------------------------- | ----------------------------------------- | ---------- |
| `nil`            | blocks                          | blocks                                    | panic      |
| open, unbuffered | može blokirati                  | može blokirati                            | dozvoljeno |
| open, buffered   | blokira samo kada je buffer pun | blokira samo kada nema dostupne vrednosti | dozvoljeno |
| closed           | panic                           | zero value + `ok=false` kada je drained   | panic      |
| ---------------- | ------------------------------- | ----------------------------------------- | ---------- |

Ova tabela je jedna od najvažnijih referenci u Module 1.

---

# Closed Buffered Channel

Važna nijansa:

**zatvaranje channel-a ne briše postojeće buffered vrednosti.**

Na primer:

```go
ch := make(chan int, 3)

ch <- 10
ch <- 20

close(ch)
```

Channel je sada:

```text
closed
buffer = [10, 20]
```

Receiver i dalje može dobiti:

```text
10
20
```

Tek nakon što su sve buffered vrednosti pročitane:

```text
receive → zero value
ok == false
```

Zato se može dogoditi:

```text
closed
+
buffer still contains values
```

Ovo je izuzetno važno za stream semantics.

---

# Channel Closing Does Not Mean "Stop Immediately"

`close(ch)` ne znači:

```text
"Delete everything in channel."
```

Ne znači ni:

```text
"Interrupt every receiver."
```

Umesto toga znači:

> "Nove vrednosti više neće biti poslate."

Ako buffered vrednosti postoje, one ostaju dostupne receiver-ima.

Mentalni model:

```text
Producer
   │
   ├── send A
   ├── send B
   ├── send C
   │
   └── close
         │
         ▼
Channel
┌────────────────┐
│ A │ B │ C │ X  │
└────────────────┘
       │
       ▼
Consumer
```

`X` predstavlja činjenicu da više novih vrednosti neće biti dodato.

---

# The `ok` Result

Receive može imati dva rezultata:

```go
value, ok := <-ch
```

`ok` predstavlja informaciju o tome da li je vrednost uspešno primljena iz otvorenog channel-a.

Primer:

```go
value, ok := <-ch

if !ok {
    return
}
```

Ovo je posebno korisno kada zero value predstavlja validnu poslovnu vrednost.

Na primer:

```text
0
```

može biti validan rezultat.

Zato:

```go
value == 0
```

nije dovoljan dokaz da je channel zatvoren.

Potrebno je proveriti:

```go
ok == false
```

---

# Comma-Ok Pattern

Tipičan channel consumption pattern:

```go
for {
    value, ok := <-ch

    if !ok {
        break
    }

    process(value)
}
```

Equivalent idiom:

```go
for value := range ch {
    process(value)
}
```

Drugi oblik je često čitljiviji kada je jedini interes obrada svih vrednosti dok stream traje.

---

# Channel Ownership

Jedan od najvažnijih design concepts u Module 1 jeste **ownership**.

Za channel treba imati jasan odgovor na:

```text
Who creates it?
Who sends?
Who receives?
Who closes?
```

Tipičan pattern:

```text
              OWNER
                │
                ▼
          creates channel
                │
                ▼
             producer
                │
              sends
                │
                ▼
             channel
                │
             receives
                │
                ▼
             consumer
```

U mnogim idiomatskim Go programima producer je odgovoran za close.

---

# Ownership Pattern

Primer:

```go
func generate() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 10; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Caller:

```go
for value := range generate() {
    fmt.Println(value)
}
```

Ovde je odgovornost jasna:

```text
generate()
   │
   ├── creates()
   ├── sends()
   └── closes()
        │
        ▼
caller
   │
   ├── receives()
   └── ranges
```

Ovo je odličan osnovni pattern jer ownership nije implicitno raspodeljen između više komponenti.

---

# Why Consumers Usually Don't Close

Ako consumer zatvori channel dok producer još može slati:

```text
Producer
   │
   │ send()
   ▼
Channel ← closed by consumer
```

producer može izazvati:

```text
panic: send on closed channel
```

Zbog toga consumer obično nije odgovoran za closing channel-a koji producer kontroliše.

Praktično pravilo:

> **The sender owns the decision to close.**

Kada postoji više sender-a, problem postaje složeniji i zahteva eksplicitno definisanje ownership-a.

---

# Multiple Producers

Razmotrimo:

```text
Producer A ──┐
Producer B ──┼──► Channel ──► Consumer
Producer C ──┘
```

Ko zatvara channel?

Ne treba da svaki producer radi:

```go
close(ch)
```

jer će drugi producer-i možda pokušati da šalju nakon zatvaranja.

Tipičan obrazac zahteva koordinaciju producer-a:

```text
Producer A ──┐
Producer B ──┼──► Channel ──► Consumer
Producer C ──┘
      │
      ▼
 all producers done
      │
      ▼
 close channel
```

Detaljna koordinacija više goroutine-a biće obrađena u Module 2 pomoću `WaitGroup` i drugih patterns.

---

# Channel as a Stream

Channel je često najbolje razumeti kao stream:

```text
value
  ↓
value
  ↓
value
  ↓
value
  ↓
EOF
```

U Go-u channel close igra ulogu sličnu end-of-stream signal-u.

Producer:

```text
send
send
send
send
close
```

Consumer:

```text
receive
receive
receive
receive
EOF
```

`range` daje prirodan način da se ovaj stream konzumira.

---

# Producer / Consumer Pattern

Jedan od centralnih obrazaca Module 1:

```text
             Producer
                │
                │ values
                ▼
          ┌───────────┐
          │  Channel  │
          └─────┬─────┘
                │
                │ values
                ▼
             Consumer
```

Producer je odgovoran za:

* proizvodnju podataka;
* slanje podataka;
* signalizaciju kraja stream-a.

Consumer je odgovoran za:

* primanje podataka;
* obradu podataka;
* reakciju na završetak stream-a.

Ova podela odgovornosti postaje temelj za pipeline architecture.

---

# Pipeline Preview

Iako će pipeline detaljno biti obrađen tek u Module 2, Module 1 treba da pripremi mentalni model:

```text
Stage 1
  │
  ▼
Channel
  │
  ▼
Stage 2
  │
  ▼
Channel
  │
  ▼
Stage 3
```

Svaki stage može biti goroutine.

```text
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Stage 1 │───►│ Stage 2 │───►│ Stage 3 │
└─────────┘    └─────────┘    └─────────┘
```

Channels predstavljaju granice između stage-ova.

---

# Backpressure

Buffered i unbuffered channels imaju direktnu vezu sa konceptom **backpressure**.

Kod unbuffered channel-a:

```text
Producer
   │
   ▼
Receiver
```

Producer ne može nastaviti sa send-om bez odgovarajućeg receiver-a.

To predstavlja vrlo direktan oblik backpressure-a.

Kod buffered channel-a:

```text
  Producer
     │
     ▼
┌───────────┐
│  Buffer   │
└───────────┘
     │
     ▼
  Consumer
```

Producer može da "pretekne" consumer-a dok se buffer ne napuni.

Kada se buffer napuni:

```text
Producer
   │
   ▼
FULL BUFFER
   │
   ▼
BLOCK
```

Backpressure se ponovo propagira ka producer-u.

---

# Buffering Trade-Off

Buffer može pomoći kada postoji kratkotrajni mismatch između producer-a i consumer-a:

```text
Producer:  ███████████
Consumer:  ██████
```

Buffer apsorbuje kratkoročni burst:

```text
Producer
   │
   ▼
[buffer]
   │
   ▼
Consumer
```

Ali buffer ne rešava trajni throughput mismatch.

Ako:

```text
producer rate > consumer rate
```

tokom dovoljno dugog perioda, buffer će na kraju postati pun.

Zato:

```text
Buffering ≠ unlimited scalability
```

i:

```text
Buffering ≠ performance guarantee
```

---

# Blocking as Backpressure

Blocking može biti korisna osobina.

Na primer:

```go
jobs <- job
```

može blokirati kada worker sistem ne može da prihvati dodatni posao.

To može sprečiti:

```text
unbounded memory growth
```

jer producer ne može beskonačno da generiše work items.

Dakle, blocking nije automatski bug.

Treba razlikovati:

```text
intentional blocking
```

od:

```text
accidental blocking
```

---

# Intentional vs Accidental Blocking

### Intentional

```text
Producer waits because consumer is not ready.
```

To može biti validan synchronization ili backpressure mechanism.

### Accidental

```text
Producer waits forever because consumer terminated.
```

To može predstavljati goroutine leak ili deadlock.

Razlika nije u tome da li goroutine blokira.

Razlika je:

> **Da li je blocking behavior deo dizajna i da li postoji garantovan način da se stanje promeni?**

---

# Goroutine Leak

Goroutine leak može nastati kada goroutine ostane blokirana bez mogućnosti da završi.

Primer:

```go
func worker(ch chan int) {
    value := <-ch
    fmt.Println(value)
}
```

Ako nikada niko ne pošalje vrednost:

```text
worker
  │
  ▼
receive
  │
  ▼
blocked forever
```

Ako owner sistema više nema potrebu za tom goroutine-om, ona je leaked.

---

# Leak Through Send

Isto važi za send:

```go
func worker(ch chan int) {
    ch <- 42
}
```

Ako receiver nikada neće doći:

```text
worker
  │
  ▼
send
  │
  ▼
blocked forever
```

Kod unbuffered channel-a ovo je naročito lako napraviti.

---

# Leak Through Range

Još jedan čest slučaj:

```go
for value := range ch {
    process(value)
}
```

Ako producer nikada ne zatvori channel:

```text
Consumer
   │
   ▼
range
   │
   ▼
waiting for next value
   │
   ▼
forever
```

Zbog toga channel lifecycle mora biti deo dizajna.

---

# Deadlock vs Goroutine Leak

Ova dva koncepta nisu identična.

## Deadlock

Program ili relevantan execution path ne može da napreduje jer svi potrebni učesnici čekaju jedni druge.

Tipičan primer:

```text
Goroutine A
   │
   ▼
wait for B

Goroutine B
   │
   ▼
wait for A
```

## Goroutine Leak

Jedna ili više goroutine-a ostaju aktivne ili blokirane iako više nisu potrebne.

Program možda nastavi da radi.

```text
Main ───────────────► continues

Leaked Goroutine ──► blocked forever
```

Deadlock i leaks mogu imati zajedničke uzroke, ali predstavljaju različite failure modes.

---

# Common Module 1 Failure Modes

Tokom praktičnog rada treba posebno tražiti:

## 1. Send Without Receiver

```go
ch := make(chan int)
ch <- 42
```

Ako nema receiver-a, send blokira.

Ako se to desi u `main` goroutine-i i nema drugog goroutine-a koji može da primi:

```text
fatal error: all goroutines are asleep - deadlock!
```

---

## 2. Receive Without Sender

```go
ch := make(chan int)
value := <-ch
```

Ako nema sender-a, receive blokira.

---

## 3. Send on Closed Channel

```go
close(ch)
ch <- 42
```

Rezultat je panic.

---

## 4. Close Closed Channel

```go
close(ch)
close(ch)
```

Rezultat je panic.

---

## 5. Close Nil Channel

```go
var ch chan int
close(ch)
```

Rezultat je panic.

---

## 6. Never Closing a Stream

```go
for value := range ch {
    process(value)
}
```

Ako channel nikada nije zatvoren i nema više vrednosti:

```text
consumer waits forever
```

---

## 7. Consumer Terminates Too Early

Producer može ostati blokiran:

```text
Producer
   │
   ▼
send
   │
   ▼
Consumer exited
   │
   ▼
blocked forever
```

Ovo je posebno važno kada consumer odluči da obradi samo deo stream-a.

---

# Early Consumer Exit

Razmotrimo:

```go
for value := range ch {
    if value == 10 {
        return
    }

    process(value)
}
```

Ako producer nastavi da šalje:

```text
Producer
   │
   ├── send 1
   ├── send 2
   ├── ...
   └── send 11
             │
             ▼
       no receiver
             │
             ▼
          BLOCKED
```

Consumer je završio, ali producer nije obavešten.

Ovo je jedan od razloga zbog kojih će cancellation biti veoma važan u kasnijim modulima.

---

# Lifecycle Must Be Bidirectional

Nije dovoljno definisati samo:

```text
Producer → Consumer
```

Potrebno je razmišljati i o:

```text
Consumer → Producer
```

u situacijama kada consumer više nije zainteresovan.

Idealni lifecycle u složenijim sistemima često izgleda ovako:

```text
Producer
   │
   │ data
   ▼
Channel
   │
   ▼
Consumer
   │
   │ cancellation / stop signal
   ▼
Producer
```

Module 1 uvodi problem.

Module 2 i Module 3 uvode naprednije mehanizme za njegovo rešavanje.

---

# Channel Direction and Ownership

Directional channels mogu dodatno pojasniti ownership.

Na primer:

```go
func generate() <-chan int
```

signalizira:

```text
caller receives
generator owns production
```

Dok:

```go
func consume(ch <-chan int)
```

signalizira:

```text
function only consumes
```

A:

```go
func produce(ch chan<- int)
```

signalizira:

```text
function only produces
```

Ovo smanjuje broj dozvoljenih operacija i time smanjuje prostor za grešku.

---

# API Design Example

Lošiji API:

```go
func process(ch chan int)
```

Ako funkcija samo čita:

```text
caller cannot know from type
whether process sends or receives
```

Precizniji API:

```go
func process(ch <-chan int)
```

Type sada dokumentuje intent.

Isto važi za producer:

```go
func produce(ch chan<- int)
```

---

# Module 1 and Type Safety

Directional channels predstavljaju primer kako Go type system može pomoći concurrency design-u.

Compiler može sprečiti:

```go
func consume(ch <-chan int) {
    ch <- 42
}
```

i:

```go
func produce(ch chan<- int) {
    value := <-ch
}
```

Greška nije samo runtime problem.

API je dizajniran tako da pogrešna operacija bude nemoguća u datom kontekstu.

---

# Concurrency Without Channels

Važno je ne zaključiti da:

```text
Go concurrency = channels only
```

Go ima više concurrency primitives i patterns:

```text
Goroutines
Channels
Mutexes
RWMutex
WaitGroup
Once
Atomics
Context
```

Module 1 namerno počinje channels pristupom jer je communication model jedan od ključnih elemenata Go concurrency-ja.

Shared-state synchronization biće obrađen u Module 3.

---

# Communication vs Shared Memory

Dva osnovna načina koordinacije su:

```text
1. Message Passing
2. Shared Memory + Synchronization
```

Channels predstavljaju prvi pristup:

```text
Goroutine A
    │
    │ message
    ▼
 Channel
    │
    │ message
    ▼
Goroutine B
```

Mutex predstavlja drugi:

```text
Goroutine A ──┐
              ▼
          Shared State
              ▲
              │
Goroutine B ──┘
       +
     Mutex
```

Module 1 se fokusira na message-passing model.

---

# Don't Use Concurrency by Default

Još jedan važan princip:

> Concurrency nije cilj sama po sebi.

Ako je posao:

```text
simple
small
sequential
```

dodavanje goroutine-a i channel-a može povećati:

* complexity;
* synchronization overhead;
* failure surface;
* debugging difficulty.

Primer:

```go
result := calculate()
```

ne mora postati:

```go
go calculate()
```

samo zato što Go omogućava concurrency.

Concurrent design treba da postoji zato što rešava konkretan problem.

---

# Concurrency vs Parallelism

Module 1 uvodi concurrency, ali ne treba mešati:

```text
Concurrency
```

sa:

```text
Parallelism
```

Concurrency opisuje strukturu sistema u kojem postoji više aktivnosti koje mogu da napreduju nezavisno.

Parallelism opisuje stvarno istovremeno izvršavanje više work units-a.

Pojednostavljeno:

```text
Concurrency

A ─────┐   ┌────
       └───┘
B ─────────────
```

nasuprot:

```text
Parallelism

CPU 1: A ─────────
CPU 2: B ─────────
```

Detaljna analiza scheduler-a i parallel execution-a dolazi u Module 3.

---

# Module 1 Practical Competency

Nakon praktičnog rada sa svim lekcijama, trebalo bi da možeš da implementiraš sledeće obrasce.

## Simple Producer

```go
func generate() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 10; i++ {
            ch <- i
        }
    }()

    return ch
}
```

---

## Consumer

```go
func consume(ch <-chan int) {
    for value := range ch {
        fmt.Println(value)
    }
}
```

---

## Producer → Consumer

```go
func main() {
    values := generate()

    consume(values)
}
```

Mentalni model:

```text
generate()
    │
    ▼
goroutine
    │
    ▼
channel
    │
    ▼
consume()
    │
    ▼
range
    │
    ▼
close
```

Ovaj obrazac je jedan od osnovnih building blocks za kasnije pipeline patterns.

---

# Exercises Philosophy

Završna lekcija:

```text
08-module-1-summary-and-exercises.md
```

ne treba da bude samo zbir trivijalnih pitanja.

Vežbe treba da testiraju:

```text
syntax
   +
semantics
   +
blocking behavior
   +
lifecycle
   +
ownership
   +
failure analysis
```

Na primer, zadatak nije dovoljno formulisati samo kao:

> "Napravi goroutine."

Bolji zadatak je:

> "Napravi producer goroutine koja generiše vrednosti, šalje ih kroz channel, pravilno završava stream i omogućava consumer-u da se završi bez goroutine leak-a."

Drugi nivo može zahtevati:

> "Objasni šta se dešava ako consumer završi pre nego što producer pošalje sve vrednosti."

Treći nivo:

> "Redizajniraj sistem tako da early consumer termination ne ostavi producer blokiran."

Takvi zadaci grade stvarno concurrency razumevanje.

---

# Suggested Exercise Categories

Module 1 exercises treba da pokriju:

| Category        | Examples                |
| --------------- | ----------------------- |
| Goroutines      | lifecycle, ordering     |
| Channels        | send/receive            |
| Blocking        | blocked send/receive    |
| Unbuffered      | rendezvous              |
| Buffered        | capacity                |
| Direction       | API contracts           |
| Range           | stream consumption      |
| Close           | lifecycle               |
| Ownership       | producer responsibility |
| Deadlock        | detection               |
| Goroutine leaks | diagnosis               |
| Design          | producer/consumer       |
| Debugging       | explain execution       |
| Refactoring     | improve unsafe code     |
| --------------- | ----------------------- |

---

# Knowledge Check

Pre prelaska na Module 2, proveri da li možeš da odgovoriš na sledeća pitanja.

### Fundamental

1. Šta je goroutine?
2. Šta se dešava kada se `main` završi?
3. Da li `go f()` čeka `f()`?
4. Šta je channel?
5. Šta znači send?
6. Šta znači receive?

### Blocking

7. Kada unbuffered send blokira?
8. Kada unbuffered receive blokira?
9. Kada buffered send blokira?
10. Kada buffered receive blokira?

### Lifecycle

11. Šta znači `close(ch)`?
12. Ko treba da zatvara channel?
13. Šta se dešava kada se primi vrednost sa closed channel-a?
14. Šta predstavlja `ok`?
15. Zašto `range` može ostati blokiran?

### Design

16. Ko je owner channel-a?
17. Zašto directional channels poboljšavaju API?
18. Kada je buffered channel koristan?
19. Kada buffering može sakriti problem?
20. Kako nastaje goroutine leak?

Ako odgovori nisu precizni, potrebno je vratiti se na odgovarajuću lekciju pre prelaska dalje.

---

# Module 1 Completion Criteria

Module 1 može se smatrati završenim kada čitalac može da:

```text
Create goroutine
        │
        ▼
Create channel
        │
        ▼
Send values
        │
        ▼
Receive values
        │
        ▼
Choose buffering model
        │
        ▼
Restrict channel direction
        │
        ▼
Consume stream
        │
        ▼
Close channel correctly
        │
        ▼
Explain blocking behavior
        │
        ▼
Identify lifecycle failures
```

Ali to još nije dovoljno za advanced concurrency.

Potrebno je razumeti i kako koordinisati **više** concurrent components.

---

# Module 1 Exit Test

Pre prelaska na Module 2 trebalo bi da bude moguće samostalno implementirati:

```text
Producer
   │
   ├── generates N values
   │
   ├── sends values through channel
   │
   └── closes channel
            │
            ▼
Consumer
   │
   ├── receives using range
   │
   └── terminates cleanly
```

Bez:

* sleep-based synchronization;
* global mutable state;
* nepotrebnih mutex-a;
* busy waiting-a;
* goroutine leak-a;
* send-on-closed panic-a;
* deadlock-a.

---

# Anti-Patterns to Avoid

## Sleep as Synchronization

Ne treba koristiti:

```go
time.Sleep(time.Second)
```

kao zamenu za pravilnu synchronization.

Sleep ne garantuje da je druga goroutine završila.

On samo uvodi vremensko čekanje.

---

## Busy Waiting

Izbegavati:

```go
for !ready {
}
```

To troši CPU i ne predstavlja kvalitetan synchronization mechanism.

---

## Unbounded Goroutines

Ne treba automatski raditi:

```go
for _, job := range jobs {
    go process(job)
}
```

ako broj `jobs` može biti veoma veliki.

To može proizvesti ogroman broj goroutine-a.

Bounded concurrency biće obrađen u Module 2.

---

## Arbitrary Buffer Sizes

Ne treba nasumično koristiti:

```go
make(chan Job, 10000)
```

samo zato što "veći buffer rešava blocking".

Buffer treba imati semantičko i operativno opravdanje.

---

## Closing from the Receiver

Ne treba zatvarati channel samo zato što receiver želi da prestane da čita.

To može ugroziti producer lifecycle.

---

# Summary

Module 1 je izgradio osnovni vocabulary Go concurrency-ja:

```text
Goroutine
Channel
Send
Receive
Blocking
Unbuffered
Buffered
Direction
Range
Close
Ownership
Lifecycle
```

Sada imamo osnovni communication model:

```text
              Goroutine A
                   │
                   │ send()
                   ▼
              ┌─────────┐
              │ Channel │
              └────┬────┘
                   │
                   │ receive()
                   ▼
              Goroutine B
```

i lifecycle model:

```text
create()
  │
  ▼
send()
  │
  ▼
receive()
  │
  ▼
close()
  │
  ▼
range terminates()
```

Takođe razumemo da svaki blocking operation mora imati smislen lifecycle:

```text
BLOCK
  │
  ▼
What unblocks me?
```

Ako odgovor ne postoji, postoji mogućnost:

```text
deadlock
```

ili:

```text
goroutine leak
```

---

# What Module 1 Does Not Yet Cover

Module 1 namerno ne obrađuje detaljno:

* `select`;
* `select` sa `default`;
* `WaitGroup`;
* worker pools;
* pipelines;
* fan-out;
* fan-in;
* mutexes;
* scheduler internals;
* `GOMAXPROCS`;
* atomic operations;
* Go Memory Model;
* happens-before;
* lock-free algorithms;
* advanced race analysis;
* concurrency profiling;
* advanced performance engineering.

Ove teme dolaze kasnije.

---

# Transition to Module 2

Module 1 je rešio problem:

```text
How can concurrent activities communicate?
```

Odgovor:

```text
Goroutines + Channels
```

Ali sledeći problem je mnogo veći:

```text
How do we coordinate many concurrent activities?
```

Na primer:

```text
             ┌── Worker A ──┐
             │              │
Input ───────┼── Worker B ──┼────► Results
             │              │
             └── Worker C ──┘
```

Sada moramo rešiti:

* ko čeka koga;
* kako reagujemo na više channels;
* kako ograničavamo broj worker-a;
* kako distribuiramo work;
* kako skupljamo rezultate;
* kako povezujemo više processing stage-ova;
* kako završavamo ceo sistem.

To je predmet **Module 2 — Coordination & Concurrency Patterns**.

---

## Navigation

|  # | Lesson                                                        | Focus                  |
| -: | ------------------------------------------------------------- | ---------------------- |
|  1 | [Goroutines](./01-goroutines.md)                              | Concurrent execution   |
|  2 | [Channels](./02-channels.md)                                  | Communication          |
|  3 | [Unbuffered Channels](./03-unbuffered-channels.md)            | Rendezvous             |
|  4 | [Buffered Channels](./04-buffered-channels.md)                | Buffering              |
|  5 | [Channel Directions](./05-channel-directions.md)              | Capability restriction |
|  6 | [Range Over Channels](./06-range-over-channels.md)            | Stream consumption     |
|  7 | [Close Channel](./07-close-channel.md)                        | Lifecycle              |
|  8 | [Summary & Exercises](./08-module-1-summary-and-exercises.md) | Consolidation          |
| -: | ------------------------------------------------------------- | ---------------------- |

---

# Module 1 — Concurrency Fundamentals

## 9. Practical Concurrency Patterns

Module 1 ne treba završiti samo poznavanjem pojedinačnih channel operacija.

Cilj je da se primitive:

```text
goroutine
channel
send
receive
close
range
```

spoje u nekoliko osnovnih concurrency patterns.

Ovi patterns predstavljaju prelaz između:

```text
language primitives
```

i:

```text
concurrency design
```

---

# Pattern 1 — Producer

Producer je komponenta koja proizvodi vrednosti i šalje ih downstream consumer-u.

Osnovni oblik:

```go
func producer() <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 10; i++ {
            out <- i
        }
    }()

    return out
}
```

Consumer:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Lifecycle:

```text
create
  │
  ▼
start goroutine
  │
  ▼
produce
  │
  ▼
send
  │
  ▼
close
```

Ovaj pattern će se koristiti kroz veliki deo kasnijeg concurrency curriculum-a.

---

# Pattern 2 — Consumer

Consumer preuzima ownership nad obradom vrednosti koje dolaze kroz channel.

```go
func consume(in <-chan int) {
    for value := range in {
        process(value)
    }
}
```

Ovde su dve stvari posebno važne:

1. consumer dobija `<-chan int`;
2. lifecycle je definisan kroz `range`.

Funkcija ne može slučajno da šalje na input channel.

---

# Pattern 3 — Producer → Consumer

Najjednostavniji kompletan model:

```text
Producer
    │
    │ values
    ▼
 Channel
    │
    │ values
    ▼
Consumer
```

Kod:

```go
func main() {
    values := producer()
    consume(values)
}
```

Ovaj pattern predstavlja osnovni building block za pipeline systems.

---

# Pattern 4 — Signal Channel

Channel ne mora prenositi poslovne podatke.

Može prenositi signal:

```go
done := make(chan struct{})
```

Na primer:

```go
go func() {
    doWork()
    close(done)
}()

<-done
```

Ovde `done` predstavlja:

```text
work completed
```

a ne podatak koji treba procesirati.

---

# Why `close` Can Be a Broadcast Signal

Jedna od korisnih osobina closed channel-a jeste da receive nakon zatvaranja više ne čeka.

To omogućava da jedan close signalizuje više receiver-a.

Konceptualno:

```text
              close(done)
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Worker A  Worker B  Worker C
        │         │         │
        ▼         ▼         ▼
      stop      stop      stop
```

Ovaj obrazac će postati naročito važan kada se uvedu cancellation i `context`.

---

# Pattern 5 — Channel as Work Queue

Buffered channel može predstavljati work queue:

```text
              Jobs
               │
               ▼
        ┌─────────────┐
        │ Job Channel │
        └──────┬──────┘
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
    Worker  Worker  Worker
```

Primer:

```go
jobs := make(chan Job, 100)
```

Producer šalje:

```go
jobs <- job
```

Worker prima:

```go
job := <-jobs
```

Detaljan worker-pool pattern biće obrađen u Module 2.

---

# Pattern 6 — Stream Processing

Channel može predstavljati stream:

```text
item
  │
  ▼
item
  │
  ▼
item
  │
  ▼
close
```

Consumer:

```go
for item := range stream {
    process(item)
}
```

Ovaj način rada je naročito koristan kada:

* broj elemenata nije poznat unapred;
* producer generiše podatke tokom vremena;
* consumer može obrađivati elemente nezavisno;
* želimo streaming umesto učitavanja svega u memoriju.

---

# Pattern 7 — Typed Pipeline Boundary

Directional channels omogućavaju da svaki stage pipeline-a ima jasan contract.

Na primer:

```go
func stage(in <-chan int) <-chan int
```

Ova funkcija:

* prima podatke;
* proizvodi novi stream;
* ne šalje na input;
* caller čita output.

Mentalni model:

```text
<-chan int
    │
    ▼
┌─────────────┐
│    stage    │
└──────┬──────┘
       │
       ▼
<-chan int
```

Ovo predstavlja čist concurrency API.

---

# Pattern 8 — Fan-Out Preview

Fan-out znači distribuiranje rada između više consumer-a.

```text
                 ┌──► Worker A
                 │
Jobs ────────────┼──► Worker B
                 │
                 └──► Worker C
```

Jedan channel može imati više receiver-a.

Svaki receive uzima jednu dostupnu vrednost.

To omogućava konkurentnu obradu.

Detaljan fan-out pattern biće obrađen u Module 2.

---

# Pattern 9 — Fan-In Preview

Fan-in predstavlja objedinjavanje više input streams u jedan output stream.

```text
Worker A ──┐
Worker B ──┼──► Results
Worker C ──┘
```

Ovaj obrazac je prirodna posledica channel-based composition-a.

Module 1 treba samo da uvede koncept.

Implementacija sa više goroutine-a i koordinacija njihovog završetka biće deo Module 2.

---

# 10. Reading Concurrency Code

Kada čitaš Go concurrency kod, nemoj početi od pitanja:

> "Šta radi ova linija?"

Počni od:

> "Koje goroutine postoje i kako komuniciraju?"

Napravi mentalni graf:

```text
G1 ──► CH1 ──► G2
G2 ──► CH2 ──► G3
G3 ──► CH3 ──► G4
```

Zatim za svaki channel utvrdi:

```text
creator
sender
receiver
closer
buffer size
lifecycle
```

---

# Concurrency Code Review Checklist

Kod review-a concurrency koda proveri:

### Goroutines

* Ko pokreće goroutine?
* Koliko goroutine-a može biti kreirano?
* Kada goroutine završava?
* Šta se dešava ako dependency nikada ne odgovori?

### Channels

* Ko kreira channel?
* Da li je buffered ili unbuffered?
* Zašto je izabrana ta konfiguracija?
* Ko šalje?
* Ko prima?
* Ko zatvara?

### Blocking

* Gde se može blokirati?
* Da li je blocking nameran?
* Šta garantuje progress?

### Lifecycle

* Šta signalizira completion?
* Šta se dešava ako consumer završi ranije?
* Šta se dešava ako producer završi ranije?

### Errors

* Kako se propagiraju greške?
* Da li goroutine može završiti zbog error-a?
* Šta se dešava sa ostalim goroutine-ama?

Poslednje teme biće detaljnije obrađene u narednim modulima.

---

# 11. Reasoning About Blocking

Jedna od najvažnijih veština u concurrency programming-u jeste sposobnost da predvidiš gde će goroutine stati.

Razmotri:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

Analiza:

```text
Goroutine A
    │
    │ send
    ▼
  waits
    │
    │ receiver available
    ▼
  continues

Main
    │
    │ receive
    ▼
  receives 42
```

Nema problema.

---

# Blocking Timeline

Concurrency kod je lakše analizirati pomoću timeline-a:

```text
Time ───────────────────────────────►

Main:     create ───── receive ───────── done
                         ▲
                         │
Worker:   start ────── send ───────────── done
```

Za buffered channel:

```go
ch := make(chan int, 1)

go func() {
    ch <- 42
}()
```

worker može završiti send pre nego što main izvrši receive:

```text
Time ───────────────────────────────►

Worker:     start ───── send ───── done
                         │
Buffer:                 [42]
                         │
Main:     create ───── receive ─ done
```

Ovo je fundamentalna razlika između buffered i unbuffered channel-a.

---

# 12. Determinism and Scheduling

Concurrency kod ne treba analizirati pretpostavljajući određeni execution order.

Na primer:

```go
go func() {
    fmt.Println("A")
}()

fmt.Println("B")
```

Nije garantovano:

```text
A
B
```

niti:

```text
B
A
```

Scheduling određuje kada će goroutine dobiti priliku za izvršavanje.

Ako je redosled važan, potrebno je uvesti synchronization.

```text
Wrong assumption:
    "goroutine started first, therefore it finishes first"

Correct model:
    "goroutine is concurrent; execution order must not be assumed
     unless synchronization establishes it."
```

Ovo je ključna mentalna promena kod učenja Go concurrency-ja.

---

# Synchronization Establishes Ordering

Na primer:

```go
done := make(chan struct{})

go func() {
    work()
    close(done)
}()

<-done
useResult()
```

Sada postoji jasno definisan dependency:

```text
work()
   │
   ▼
close(done)
   │
   ▼
<-done
   │
   ▼
useResult()
```

Dakle, `useResult()` ne zavisi od slučajnog scheduler ordering-a.

Zavisi od eksplicitnog synchronization-a.

---

# 13. Concurrency Invariants

Kod kompleksnijeg concurrency koda korisno je definisati invariants.

Na primer:

```text
Invariant:
Only the producer closes the channel.
```

ili:

```text
Invariant:
Every value sent to jobs is eventually consumed.
```

ili:

```text
Invariant:
No goroutine remains blocked after shutdown.
```

Ovo pretvara concurrency debugging iz:

```text
"Why does this sometimes hang?"
```

u:

```text
"Which invariant was violated?"
```

To je mnogo korisniji način razmišljanja.

---

# Example: Channel Ownership Invariant

Pretpostavimo:

```text
Producer → jobs → Workers
```

Invariant:

```text
Producer is the sole owner responsible for closing jobs.
```

Ako se u kodu pojavi:

```go
worker1 := func() {
    close(jobs)
}
```

odmah postoji sumnja da je ownership model narušen.

---

# Example: Progress Invariant

Pretpostavimo:

```text
Producer
   │
   ▼
Channel
   │
   ▼
Consumer
```

Progress invariant:

```text
Every send must eventually have a receiver.
```

Ako consumer može završiti pre producer-a, invariant više nije garantovan.

To može proizvesti blocked sender.

---

# 14. Debugging Channel Problems

Kada program "visi", prvo traži blocking points.

Tipična pitanja:

```text
Where is the goroutine blocked?
Why is it blocked?
Who should unblock it?
Can that goroutine still run?
```

Za channel:

```text
send
receive
range
```

proveri drugu stranu komunikacije.

---

# Debugging a Blocked Send

Ako imaš:

```go
ch <- value
```

i goroutine je blokirana:

1. Ko je receiver?
2. Da li receiver još postoji?
3. Da li receiver može doći do receive operation-a?
4. Ako je buffered, da li je buffer pun?
5. Da li je consumer prerano završio?

---

# Debugging a Blocked Receive

Ako imaš:

```go
value := <-ch
```

proveri:

1. Ko šalje?
2. Da li producer još postoji?
3. Da li producer može doći do send-a?
4. Da li je producer već završio?
5. Ako se koristi `range`, da li će channel biti zatvoren?

---

# Debugging `range`

Za:

```go
for value := range ch {
    process(value)
}
```

proveri:

```text
Who sends?
Who closes?
Can sender terminate?
Can close always be reached?
Can consumer terminate early?
```

Ako odgovor na "Who closes?" nije jasan, lifecycle dizajn je nepotpun.

---

# 15. Concurrency Design Questions

Pre implementacije concurrency sistema treba postaviti nekoliko pitanja.

## Execution

```text
Which operations can run concurrently?
```

## Communication

```text
How do concurrent components exchange data?
```

## Synchronization

```text
Where must ordering be guaranteed?
```

## Ownership

```text
Who owns each channel?
```

## Backpressure

```text
What happens when consumers are slower than producers?
```

## Shutdown

```text
How does each goroutine know it should stop?
```

## Failure

```text
What happens if one goroutine fails?
```

## Resource Lifetime

```text
Who releases the resources?
```

U Module 1 odgovori na poslednja pitanja mogu biti jednostavni.

U narednim modulima postaće mnogo važniji.

---

# 16. When to Choose Unbuffered Channels

Unbuffered channel je dobar kandidat kada je potrebna direktna koordinacija.

Tipični slučajevi:

```text
handoff
synchronization
completion signal
strict producer-consumer coordination
```

Na primer:

```go
ready := make(chan struct{})

go func() {
    initialize()
    close(ready)
}()

<-ready
```

Ovde je važniji signal nego throughput.

---

# 17. When to Choose Buffered Channels

Buffered channel je kandidat kada postoji opravdana potreba za:

```text
temporary decoupling
work queue
burst absorption
bounded buffering
```

Na primer:

```go
jobs := make(chan Job, workerCount)
```

Ali kapacitet mora biti deo dizajna.

Ne treba koristiti buffer samo da bi se "uklonio deadlock" bez razumevanja uzroka.

---

# Buffer Can Hide Bugs

Pretpostavimo da postoji bug:

```text
Producer never has a receiver.
```

Dodavanje:

```go
make(chan int, 100)
```

može privremeno omogućiti programu da radi.

Ali ako producer pošalje 101 vrednost:

```text
buffer = full
```

problem se vraća.

Zato:

```text
Buffering can delay a blocking failure.
```

Ne mora ga rešiti.

---

# 18. Module 1 Architecture

Kompletan module architecture:

```text
                    MODULE 1
                        │
        ┌───────────────┼───────────────────┐
        │               │                   │
        ▼               ▼                   ▼
   Execution       Communication         Lifecycle
        │               │                   │
        ▼               ▼                   ▼
   Goroutines        Channels             Close
                        │                   │
             ┌──────────┴──────────┐        │
             │                     │        │
             ▼                     ▼        ▼
        Unbuffered             Buffered   Range
             │                     │
             └──────────┬──────────┘
                        │
                        ▼
                 Channel Directions
                        │
                        ▼
                  API / Ownership
```

Ovo predstavlja conceptual map celog modula.

---

# 19. From Primitive to Pattern

Learning progression treba da bude:

```text
Syntax
  │
  ▼
Semantics
  │
  ▼
Blocking behavior
  │
  ▼
Lifecycle
  │
  ▼
Ownership
  │
  ▼
Pattern
  │
  ▼
Architecture
```

Nije dovoljno znati:

```go
ch <- value
```

Potrebno je znati:

```text
Who receives it?
When?
What if nobody receives it?
Who closes the channel?
What if the consumer exits?
What happens under load?
```

To je razlika između poznavanja syntax-a i razumevanja concurrency-ja.

---

# 20. Module 1 Final Checklist

Pre nego što pređeš na Module 2, proveri sledeću listu.

## Goroutines

* [ ] Znam šta `go` statement radi.
* [ ] Razumem goroutine lifecycle.
* [ ] Ne oslanjam se na slučajan execution order.
* [ ] Razumem da završetak `main` procesa prekida program.

## Channels

* [ ] Znam da kreiram channel.
* [ ] Razumem send.
* [ ] Razumem receive.
* [ ] Razumem blocking semantics.

## Unbuffered

* [ ] Razumem rendezvous.
* [ ] Znam kada send blokira.
* [ ] Znam kada receive blokira.
* [ ] Mogu koristiti unbuffered channel za synchronization.

## Buffered

* [ ] Razumem capacity.
* [ ] Razumem current buffer occupancy.
* [ ] Znam kada send blokira.
* [ ] Znam kada receive blokira.
* [ ] Razumem backpressure implications.

## Direction

* [ ] Razumem `chan T`.
* [ ] Razumem `<-chan T`.
* [ ] Razumem `chan<- T`.
* [ ] Mogu koristiti directional channels u API dizajnu.

## Range

* [ ] Mogu koristiti `range` nad channel-om.
* [ ] Razumem kada `range` završava.
* [ ] Razumem vezu između `range` i `close`.

## Close

* [ ] Znam šta `close` znači.
* [ ] Razumem sender ownership.
* [ ] Znam šta se dešava pri receive-u sa closed channel-a.
* [ ] Razumem `value, ok := <-ch`.
* [ ] Znam da se closed channel ne sme ponovo zatvarati.

## Failure Modes

* [ ] Prepoznajem deadlock.
* [ ] Prepoznajem goroutine leak.
* [ ] Prepoznajem send on closed channel.
* [ ] Prepoznajem close of closed channel.
* [ ] Prepoznajem blocking send/receive.
* [ ] Razumem problem early consumer exit-a.

---

# 21. Final Mental Model

Na kraju Module 1 treba da postoji sledeća mentalna slika:

```text
                       GO CONCURRENCY
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
             Goroutines                Channels
                 │                         │
                 │                         │
                 ▼                         ▼
             Execution              Communication
                                           │
                              ┌────────────┴────────────┐
                              │                         │
                              ▼                         ▼
                         Unbuffered                 Buffered
                              │                         │
                              └────────────┬────────────┘
                                           │
                                           ▼
                                   Blocking Semantics
                                           │
                                           ▼
                                    Channel Direction
                                           │
                                           ▼
                                     Stream / Range
                                           │
                                           ▼
                                         Close
                                           │
                                           ▼
                                       Lifecycle
                                           │
                                           ▼
                                       Ownership
                                           │
                                           ▼
                                  Concurrency Patterns
```

Ovaj model treba koristiti kao osnovu za sve naredne module.

---

# 22. Module 1 in One Sentence

Ako bi ceo Module 1 morao da se sažme u jednu ideju:

> **Goroutines omogućavaju concurrent execution, channels omogućavaju komunikaciju i synchronization između njih, a pravilno definisan ownership i lifecycle određuju da li će taj concurrency sistem završavati korektno.**

---

# 23. Preparation for Module 2

Nakon što je ovaj modul savladan, sledeći prirodan korak je koordinacija više concurrent aktivnosti.

Module 2 će nadograditi:

```text
Goroutines
+
Channels
+
Blocking
+
Lifecycle
```

sa:

```text
select
WaitGroup
Worker Pools
Pipelines
Fan-Out
Fan-In
```

što će omogućiti izgradnju realnijih concurrency sistema:

```text
                     Input
                       │
                       ▼
                  ┌─────────┐
                  │ Workers │
                  └────┬────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Worker A     Worker B     Worker C
          │            │            │
          └────────────┼────────────┘
                       ▼
                    Results
```

Tu se prelazi sa:

```text
"How do channels work?"
```

na:

```text
"How do I design a concurrent system?"
```

---

# Module 1 Completion

Module 1 je završen kada čitalac više ne posmatra:

```go
go worker()
```

kao samo "pokretanje funkcije u pozadini",

već razmišlja o:

```text
goroutine lifetime
        │
        ▼
  communication
        │
        ▼
    blocking
        │
        ▼
    ownership
        │
        ▼
    shutdown
        │
        ▼
   correctness
```

To je fundamentalna osnova potrebna za razumevanje naprednijeg Go concurrency-ja.

---

## Navigation

|  # | Lesson                                                        | Focus                        |
| -: | ------------------------------------------------------------- | ---------------------------- |
|  1 | [Goroutines](./01-goroutines.md)                              | Concurrent execution         |
|  2 | [Channels](./02-channels.md)                                  | Communication                |
|  3 | [Unbuffered Channels](./03-unbuffered-channels.md)            | Rendezvous & synchronization |
|  4 | [Buffered Channels](./04-buffered-channels.md)                | Buffering & backpressure     |
|  5 | [Channel Directions](./05-channel-directions.md)              | Capability restriction       |
|  6 | [Range Over Channels](./06-range-over-channels.md)            | Stream consumption           |
|  7 | [Close Channel](./07-close-channel.md)                        | Lifecycle & ownership        |
|  8 | [Summary & Exercises](./08-module-1-summary-and-exercises.md) | Consolidation                |
| -: | ------------------------------------------------------------- | ---------------------------- |

