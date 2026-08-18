# Interview Questions — Beginner

> **Fajl:** `extras/01-interview-questions/module-1/01-beginner.md`
>
> **Nivo:** Beginner
>
> **Oblast:** #1 — Concurrency Fundamentals

---

## Sadržaj

1. [Šta je goroutine?](#1-šta-je-goroutine)
2. [Po čemu se goroutine razlikuje od OS thread-a?](#2-po-čemu-se-goroutine-razlikuje-od-os-thread-a)
3. [Kako se pokreće goroutine?](#3-kako-se-pokreće-goroutine)
4. [Da li `go` garantuje da će goroutine završiti pre `main()` funkcije?](#4-da-li-go-garantuje-da-će-goroutine-završiti-pre-main-funkcije)
5. [Šta je channel i čemu služi?](#5-šta-je-channel-i-čemu-služi)
6. [Kako se šalje vrednost kroz channel?](#6-kako-se-šalje-vrednost-kroz-channel)
7. [Kako se prima vrednost iz channel-a?](#7-kako-se-prima-vrednost-iz-channel-a)
8. [Šta je osnovna razlika između buffered i unbuffered channel-a?](#8-šta-je-osnovna-razlika-između-buffered-i-unbuffered-channel-a)
9. [Šta se dešava kada se šalje vrednost u unbuffered channel?](#9-šta-se-dešava-kada-se-šalje-vrednost-u-unbuffered-channel)
10. [Šta znači channel direction?](#10-šta-znači-channel-direction)
11. [Čemu služe `range` i `close` zajedno sa channel-ima?](#11-čemu-služe-range-i-close-zajedno-sa-channel-ima)
12. [Ko treba da zatvori channel?](#12-ko-treba-da-zatvori-channel)
13. [Šta se dešava ako program pokuša da primi podatak iz zatvorenog channel-a?](#13-šta-se-dešava-ako-program-pokuša-da-primi-podatak-iz-zatvorenog-channel-a)
14. [Šta je deadlock u kontekstu goroutines i channel-a?](#14-šta-je-deadlock-u-kontekstu-goroutines-i-channel-a)
15. [Zašto `time.Sleep()` nije dobar način sinhronizacije goroutines?](#15-zašto-timesleep-nije-dobar-način-sinhronizacije-goroutines)

---

# 1. Šta je goroutine?

## Odgovor

Goroutine je lagana jedinica konkurentnog izvršavanja kojom upravlja **Go runtime**, a ne direktno operativni sistem.

Pokreće se pomoću ključne reči `go`:

```go
go doWork()
```

Kada Go izvrši ovu naredbu, `doWork()` se ne izvršava kao običan sinhroni poziv funkcije. Umesto toga, Go runtime kreira novu goroutine i omogućava scheduler-u da je rasporedi za izvršavanje.

Na primer:

```go
package main

import "fmt"

func printMessage() {
	fmt.Println("Hello from goroutine")
}

func main() {
	go printMessage()

	fmt.Println("Hello from main")
}
```

Ovde postoje najmanje dve goroutine:

* glavna goroutine u kojoj se izvršava `main()`;
* goroutine pokrenuta naredbom `go printMessage()`.

Važno je razumeti da **goroutine nije isto što i thread**.

Go runtime može imati veliki broj goroutines koje se izvršavaju preko relativno malog broja OS thread-ova.

Konceptualno:

```text
Goroutines
    │
    ▼
Go Scheduler
    │
    ▼
OS Threads
    │
    ▼
CPU
```

Zbog toga je goroutine mnogo jeftinija za kreiranje i upravljanje od klasičnog OS thread-a.

### Šta interviewer očekuje od Beginner kandidata?

Beginner treba da zna:

* šta je goroutine;
* kako se pokreće;
* da je goroutine konkurentna jedinica izvršavanja;
* da njome upravlja Go runtime;
* da goroutine nije isto što i OS thread.

### Česta greška

Nije precizno reći:

> "Goroutine je Go thread."

Preciznije je:

> "Goroutine je lagana jedinica izvršavanja kojom upravlja Go runtime scheduler i koja se izvršava na OS thread-ovima."

---

# 2. Po čemu se goroutine razlikuje od OS thread-a?

## Odgovor

Najvažnija razlika je u tome **ko upravlja izvršavanjem** i **koliko je jedinica izvršavanja skupa**.

OS thread kreira i kontroliše operativni sistem.

Goroutine kreira Go runtime.

Pojednostavljeno:

| OS Thread                      | Goroutine                                   |
| ------------------------------ | ------------------------------------------- |
| Upravlja OS                    | Upravlja Go runtime                         |
| Teži za kreiranje              | Veoma lagana                                |
| Veći overhead                  | Mali overhead                               |
| Scheduler je deo OS-a          | Go ima sopstveni scheduler                  |
| Relativno mali broj thread-ova | Može postojati veoma veliki broj goroutines |

Jedna od važnih karakteristika goroutines je njihov stack.

Goroutine počinje sa relativno malim stack-om koji može da raste kada je potrebno.

Zbog toga je moguće imati veliki broj goroutines bez potrebe da se za svaku kreira veliki OS thread stack.

Međutim, ne treba iz ovoga zaključiti:

> "Goroutines su besplatne."

Nisu.

Svaka goroutine i dalje koristi memoriju i runtime resurse. Ako aplikacija nekontrolisano kreira goroutines koje nikada ne završavaju, može doći do **goroutine leak-a** i ozbiljne potrošnje resursa.

### Interview odgovor u jednoj rečenici

> Goroutine je lagana konkurentna jedinica izvršavanja kojom upravlja Go runtime, dok je OS thread jedinica izvršavanja kojom upravlja operativni sistem.

---

# 3. Kako se pokreće goroutine?

## Odgovor

Goroutine se pokreće pomoću ključne reči `go`.

Osnovni oblik je:

```go
go function()
```

Na primer:

```go
func worker() {
	fmt.Println("working")
}

func main() {
	go worker()
}
```

Moguće je pokrenuti i anonimnu funkciju:

```go
go func() {
	fmt.Println("working")
}()
```

Moguće je proslediti argumente:

```go
func greet(name string) {
	fmt.Println("Hello,", name)
}

func main() {
	go greet("Marko")
}
```

Isto važi za anonimne funkcije:

```go
go func(name string) {
	fmt.Println("Hello,", name)
}("Marko")
```

Bitno je razumeti da:

```go
go worker()
```

nije isto što i:

```go
worker()
```

Kod običnog poziva:

```go
worker()
```

trenutna goroutine čeka da se `worker()` završi.

Kod:

```go
go worker()
```

Go pokreće novu goroutine i trenutna goroutine može odmah da nastavi sa izvršavanjem.

Na primer:

```go
func main() {
	go worker()

	fmt.Println("main continues")
}
```

`main` ne čeka automatski da `worker()` završi.

To je jedna od najvažnijih stvari koju kandidat mora da razume.

---

# 4. Da li `go` garantuje da će goroutine završiti pre `main()` funkcije?

## Odgovor

**Ne.**

Pokretanje goroutine ne znači da će ona sigurno završiti pre nego što se `main()` završi.

Na primer:

```go
package main

import "fmt"

func worker() {
	fmt.Println("worker")
}

func main() {
	go worker()

	fmt.Println("main")
}
```

Moguće je da program ispiše samo:

```text
main
```

a da:

```text
worker
```

nikada ne bude ispisan.

Zašto?

Zato što se program završava kada se završi glavna goroutine, odnosno kada se završi `main()` funkcija.

Ako je u trenutku završetka `main()` goroutine `worker` još uvek aktivna, program se završava.

Drugim rečima:

```text
main goroutine
      │
      ├── go worker()
      │
      ├── nastavlja izvršavanje
      │
      └── main() završava
              │
              ▼
         program završava
```

Čak i ako `worker` još nije izvršen.

---

## Kako se početnici često pokušavaju zaštititi?

Jedan od najčešćih primera je:

```go
func main() {
	go worker()

	time.Sleep(time.Second)
}
```

Ovo može izgledati kao da rešava problem, ali nije dobra metoda sinhronizacije.

Problem je što ne znaš da li je jedna sekunda:

* previše;
* dovoljno;
* premalo.

Na sporijem sistemu worker možda neće završiti za jednu sekundu.

Na brzom sistemu program možda nepotrebno čeka.

Zbog toga:

```go
time.Sleep(...)
```

nije generalno mehanizam za koordinaciju goroutines.

U kasnijim modulima koriste se pravi mehanizmi sinhronizacije, kao što su:

* `sync.WaitGroup`;
* channels;
* `context`;
* drugi synchronization primitives.

### Ključna interview poruka

Ako interviewer pita:

> "Da li `go worker()` garantuje da će worker završiti?"

Odgovor je:

> Ne. `go` samo pokreće novu goroutine. Ne postoji implicitno čekanje na njen završetak. Ako je potrebno sačekati goroutine, mora se koristiti odgovarajući mehanizam koordinacije.

---

# 5. Šta je channel i čemu služi?

## Odgovor

Channel je Go mehanizam za **komunikaciju i koordinaciju između goroutines**.

Možeš ga posmatrati kao kanal kroz koji goroutines šalju i primaju vrednosti.

Primer:

```go
ch := make(chan int)
```

Ovim se kreira channel koji prenosi vrednosti tipa `int`.

Slanje:

```go
ch <- 42
```

Primanje:

```go
value := <-ch
```

Konceptualno:

```text
Goroutine A
     │
     │  42
     ▼
  channel
     │
     ▼
Goroutine B
```

Jedna od ključnih ideja Go concurrency modela jeste:

> Goroutines mogu komunicirati razmenom podataka preko channels.

Umesto da više goroutines direktno manipuliše zajedničkim stanjem, često se može dizajnirati sistem u kome jedna goroutine proizvodi podatke, a druga ih prima preko channel-a.

Na primer:

```go
func producer(ch chan int) {
	ch <- 42
}

func consumer(ch chan int) {
	value := <-ch
	fmt.Println(value)
}
```

Ovde channel predstavlja komunikacionu granicu između producenta i konzumenta.

---

## Channel nije samo "queue"

Početnik često kaže:

> "Channel je queue."

To nije potpuno precizno.

Channel može imati buffering i u tom slučaju zaista poseduje buffer koji čuva određeni broj vrednosti.

Ali **unbuffered channel** nema takav buffer.

Kod unbuffered channel-a slanje i primanje su direktno povezani:

```go
ch := make(chan int)
```

Pošiljalac:

```go
ch <- 42
```

ne može normalno da nastavi samo zato što je izraz izvršen.

Mora postojati odgovarajući receiver.

Zbog toga channels nisu samo struktura podataka, već i **mehanizam sinhronizacije**.

---

## Šta interviewer očekuje?

Beginner treba da zna:

* šta je channel;
* da je tipiziran;
* kako se kreira;
* kako se šalje vrednost;
* kako se prima vrednost;
* da channel može služiti i za komunikaciju i za koordinaciju goroutines.

Osnovna sintaksa:

```go
ch := make(chan int)

ch <- 10

value := <-ch
```

---

# Modul 1 — Interview Questions: Beginner

Ovaj dokument predstavlja početni nivo pitanja za proveru razumevanja osnova konkurentnog programiranja u Go-u.

Pitanja su fokusirana na fundamentalne koncepte obrađene u **Module 1**: goroutine, osnovnu komunikaciju preko kanala i osnovne principe slanja i prijema vrednosti.

## 1. Goroutine kao osnovna jedinica konkurentnosti

### Pitanje 1

Šta je goroutine i po čemu se razlikuje od klasičnog OS threada?

**Odgovor:**

Goroutine je lagana izvršna jedinica kojom upravlja Go runtime. Pokreće se pomoću `go` naredbe:

```go
go doWork()
```

Za razliku od OS threada, goroutine nije direktno mapiran 1:1 na jedan sistemski thread. Go runtime koristi scheduler koji raspoređuje veliki broj goroutine-a preko manjeg ili odgovarajućeg broja OS threadova.

Zbog toga je kreiranje goroutine-a relativno jeftino i moguće je imati veliki broj istovremeno aktivnih goroutine-a.

---

### Pitanje 2

Šta se dešava kada napišemo:

```go
go process()
```

**Odgovor:**

Go pokreće `process` kao novu goroutine. Pozivajuća goroutine ne čeka da se `process` završi, već nastavlja sa svojim izvršavanjem.

To znači da:

```go
go process()
fmt.Println("done")
```

ne garantuje da će `process()` završiti pre nego što se ispiše `"done"`.

Scheduler odlučuje kada će nova goroutine dobiti priliku za izvršavanje.

---

### Pitanje 3

Da li je izvršavanje goroutine-a determinističko?

**Odgovor:**

Ne. Redosled izvršavanja konkurentnih goroutine-a generalno nije deterministički.

Na primer:

```go
go fmt.Println("A")
go fmt.Println("B")
```

ne garantuje da će izlaz biti:

```text
A
B
```

Može biti i:

```text
B
A
```

Program ne treba da zavisi od slučajnog redosleda izvršavanja goroutine-a.

---

## 2. Osnovna komunikacija preko kanala

### Pitanje 4

Šta je channel u Go-u?

**Odgovor:**

Channel je mehanizam za komunikaciju i sinhronizaciju između goroutine-a.

Kanal možemo kreirati pomoću:

```go
ch := make(chan int)
```

Jedna goroutine može poslati vrednost:

```go
ch <- 42
```

dok druga može primiti vrednost:

```go
value := <-ch
```

Osnovna ideja Go concurrency modela jeste da goroutine-i mogu koordinisati rad tako što komuniciraju preko kanala.

---

### Pitanje 5

Zašto se channels često opisuju kao mehanizam za komunikaciju između goroutine-a?

**Odgovor:**

Zato što channel omogućava jednoj goroutine-i da preda podatak drugoj goroutine-i bez potrebe da obe direktno manipulišu istom promenljivom.

Na primer:

```go
func main() {
    ch := make(chan int)

    go func() {
        ch <- 42
    }()

    value := <-ch

    fmt.Println(value)
}
```

Jedna goroutine proizvodi vrednost, dok druga prima tu vrednost.

Channel istovremeno predstavlja i mehanizam sinhronizacije jer slanje i prijem mogu blokirati izvršavanje.

---

## 3. Send i receive operacije

### Pitanje 6

Šta znači izraz:

```go
ch <- value
```

**Odgovor:**

To je send operacija.

Vrednost `value` se šalje u channel `ch`:

```go
ch <- value
```

Kod nebufferovanog kanala send operacija može blokirati dok druga goroutine ne bude spremna da primi vrednost.

---

### Pitanje 7

Šta znači izraz:

```go
value := <-ch
```

**Odgovor:**

To je receive operacija.

Program pokušava da primi jednu vrednost iz kanala `ch`:

```go
value := <-ch
```

Ako vrednost trenutno nije dostupna, receive operacija može blokirati dok druga goroutine ne pošalje vrednost.

---

## 4. Sinhronizacija preko kanala

### Pitanje 8

Kako channel može da se koristi za sinhronizaciju?

**Odgovor:**

Channel može da primora jednu goroutine-u da sačeka drugu.

Na primer:

```go
done := make(chan struct{})

go func() {
    // neki posao

    close(done)
}()

<-done
```

Glavna goroutine blokira na:

```go
<-done
```

dok druga goroutine ne završi posao i zatvori channel.

Na taj način channel nije samo sredstvo za prenos podataka već i alat za koordinaciju izvršavanja.

---

### Pitanje 9

Šta znači da je channel operacija blokirajuća?

**Odgovor:**

To znači da goroutine koja izvršava operaciju može biti zaustavljena dok se ne ispuni uslov potreban za nastavak.

Kod nebufferovanog kanala:

```go
ch := make(chan int)
```

send:

```go
ch <- 10
```

čeka odgovarajući receive, dok receive:

```go
value := <-ch
```

čeka odgovarajući send.

Ovo ponašanje omogućava prirodnu sinhronizaciju između goroutine-a.

---

## 5. Osnovni concurrency model

### Pitanje 10

Zašto se u Go-u ne preporučuje posmatranje goroutine-a samo kao „jeftinog threada“?

**Odgovor:**

Zato što goroutine predstavlja deo šireg Go concurrency modela.

Važni elementi tog modela su:

* goroutine,
* channel,
* scheduler,
* blokirajuće operacije,
* komunikacija,
* sinhronizacija,
* ownership podataka.

Samo kreiranje velikog broja goroutine-a nije dovoljno za pravilan concurrency dizajn. Potrebno je razumeti kako goroutine-i komuniciraju, kada blokiraju i kako se njihov životni ciklus završava.

---

## Beginner — ključne stvari koje kandidat treba da zna

Na Beginner nivou kandidat bi trebalo da ume da objasni:

* šta je goroutine;
* kako se goroutine pokreće;
* zašto redosled izvršavanja nije garantovan;
* šta je channel;
* kako se channel kreira;
* kako se šalje vrednost;
* kako se prima vrednost;
* šta znači blokirajuća send/receive operacija;
* kako channel može da posluži za osnovnu sinhronizaciju;
* zašto konkurentni program ne treba da zavisi od slučajnog redosleda goroutine-a.

### Minimalni praktični primer

```go
package main

import "fmt"

func main() {
    ch := make(chan string)

    go func() {
        ch <- "hello from goroutine"
    }()

    message := <-ch

    fmt.Println(message)
}
```

Mentalni model ovog primera je jednostavan:

```text
main goroutine
      │
      │ start
      ▼
worker goroutine
      │
      │ send
      ▼
   channel
      │
      │ receive
      ▼
main goroutine
```

Beginner kandidat ne mora još da razume interne detalje Go scheduler-a. Međutim, mora da razume osnovnu relaciju:

**goroutine + channel + send/receive + blocking = osnovni temelj Go concurrency modela.**

---

# 01 — Beginner: Osnovna pitanja o gorutinama i kanalima

## 4. Goroutine scheduling i osnovna komunikacija

### Pitanje 16

**Da li pokretanje gorutine garantuje da će se njeno izvršavanje desiti pre nego što se trenutna funkcija nastavi?**

**Odgovor:**

Ne.

Poziv:

```go
go doWork()
```

samo omogućava Go runtime-u da zakaže izvršavanje `doWork` funkcije kao gorutine. Ne postoji garancija da će nova gorutina odmah početi sa izvršavanjem.

Na primer:

```go
func main() {
    go fmt.Println("goroutine")

    fmt.Println("main")
}
```

Mogući izlaz je:

```text
main
```

a moguće je i:

```text
main
goroutine
```

U ovom primeru program može završiti pre nego što gorutina dobije priliku da izvrši `fmt.Println`.

**Ključna stvar:**

`go` statement ne znači:

> "Izvrši ovo odmah paralelno."

On znači:

> "Pokreni ovu funkciju kao gorutinu čije će izvršavanje Go scheduler rasporediti."

---

### Pitanje 17

**Šta se dešava sa gorutinama kada `main` funkcija završi?**

**Odgovor:**

Kada `main` gorutina završi izvršavanje, proces se završava.

Preostale gorutine se ne čekaju automatski.

Na primer:

```go
func main() {
    go func() {
        time.Sleep(time.Second)
        fmt.Println("done")
    }()
}
```

Program može završiti odmah, bez ispisa:

```text
done
```

Razlog je to što `main` ne čeka novu gorutinu.

Za koordinaciju se koriste mehanizmi kao što su:

* `sync.WaitGroup`,
* channels,
* `context`,
* drugi mehanizmi za sinhronizaciju.

Za osnovni slučaj možemo koristiti `sync.WaitGroup`:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    fmt.Println("done")
}()

wg.Wait()
```

Ovde `main` gorutina čeka da radna gorutina završi.

---

### Pitanje 18

**Da li dve gorutine automatski izvršavaju kod paralelno?**

**Odgovor:**

Ne nužno.

Treba razlikovati:

* **concurrency** — više jedinica rada može biti u toku;
* **parallelism** — više jedinica rada se zaista izvršava istovremeno na različitim CPU jezgrima.

Go omogućava konkurentno izvršavanje pomoću gorutina, ali stvarni paralelizam zavisi od runtime-a, raspoloživih CPU resursa i podešavanja izvršavanja.

Primer:

```go
go taskA()
go taskB()
```

znači da su `taskA` i `taskB` pokrenuti kao gorutine.

Ne znači da će CPU nužno izvršavati:

```text
CPU 1 → taskA
CPU 2 → taskB
```

istovremeno.

Go runtime scheduler odlučuje kada i gde će se gorutine izvršavati.

---

### Pitanje 19

**Da li redosled izvršavanja gorutina može da se predvidi?**

**Odgovor:**

Ne treba pretpostavljati određeni redosled izvršavanja gorutina.

Na primer:

```go
func main() {
    go func() {
        fmt.Println("A")
    }()

    go func() {
        fmt.Println("B")
    }()
}
```

Ne treba pisati program koji zavisi od toga da li će rezultat biti:

```text
A
B
```

ili:

```text
B
A
```

Scheduler odlučuje kada će pojedina gorutina dobiti priliku za izvršavanje.

Ako redosled predstavlja poslovni zahtev, potrebno je eksplicitno uvesti mehanizam koordinacije.

Na primer, channel može izraziti zavisnost:

```go
done := make(chan struct{})

go func() {
    fmt.Println("A")
    close(done)
}()

<-done

fmt.Println("B")
```

Sada postoji eksplicitna veza:

```text
A završava
    ↓
close(done)
    ↓
main nastavlja
    ↓
B
```

Ovo je mnogo pouzdanije nego oslanjanje na slučajni scheduling.

---

### Pitanje 20

**Šta znači da je channel blokirajući mehanizam?**

**Odgovor:**

Send i receive operacije nad channel-om mogu blokirati gorutinu dok se ne ispune odgovarajući uslovi.

Kod nebaferovanog channel-a:

```go
ch := make(chan int)
```

slanje:

```go
ch <- 42
```

ne može da završi dok druga gorutina ne bude spremna da primi vrednost.

Analogno tome:

```go
value := <-ch
```

čeka dok vrednost ne bude poslata.

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch

fmt.Println(value)
```

Ovde channel predstavlja mehanizam komunikacije i koordinacije između dve gorutine.

Važno je razumeti da channel nije samo "cev" za prenos podataka.

On istovremeno može da predstavlja **sinhronizacionu tačku** između gorutina.

---

### Pitanje 21

**Šta se dešava kada šaljemo vrednost na nebaferovani channel, a niko trenutno ne prima tu vrednost?**

**Odgovor:**

Gorutina koja pokušava da pošalje vrednost blokira se.

Na primer:

```go
ch := make(chan int)

ch <- 10

fmt.Println("done")
```

Ako ne postoji druga gorutina koja prima vrednost sa `ch`, izvršavanje se zaustavlja na:

```go
ch <- 10
```

`fmt.Println("done")` se neće izvršiti.

Ako nema druge gorutine koja može da omogući nastavak izvršavanja, program može završiti sa:

```text
fatal error: all goroutines are asleep - deadlock!
```

Ovo je jedan od osnovnih oblika deadlock-a u Go concurrency modelu.

---

### Pitanje 22

**Šta se dešava kada primamo vrednost sa nebaferovanog channel-a, a niko ne šalje vrednost?**

**Odgovor:**

Receive operacija blokira.

Na primer:

```go
ch := make(chan int)

value := <-ch

fmt.Println(value)
```

Ako nijedna druga gorutina ne izvršava:

```go
ch <- value
```

receive operacija nema odakle da dobije vrednost.

Zato gorutina ostaje blokirana.

Ako su sve gorutine u stanju čekanja i ne postoji mogućnost napretka, runtime detektuje deadlock.

---

### Pitanje 23

**Zašto se channel često koristi kao mehanizam koordinacije, a ne samo za prenos podataka?**

**Odgovor:**

Zato što komunikacija preko channel-a može istovremeno da uspostavi odnos između trenutka kada je jedna gorutina proizvela podatak i trenutka kada druga gorutina može da nastavi.

Na primer:

```go
done := make(chan struct{})

go func() {
    doWork()
    close(done)
}()

<-done
```

Ovde channel ne prenosi konkretan poslovni podatak.

Njegova svrha je signalizacija:

```text
worker
  │
  ├── doWork()
  │
  └── close(done)
           │
           ▼
        receiver
           │
           └── nastavlja
```

Ovakav obrazac je veoma čest u Go concurrency kodu.

Channel može predstavljati:

* signal završetka;
* dostupnost posla;
* rezultat rada;
* zahtev;
* događaj;
* koordinaciju između gorutina.

Zbog toga je razumevanje channel-a mnogo šire od razumevanja same sintakse:

```go
ch <- value
```

i:

```go
value := <-ch
```

---

### Pitanje 24

**Da li je dobro koristiti `time.Sleep` da bismo "sačekali" drugu gorutinu?**

**Odgovor:**

U produkcionom kodu uglavnom nije.

Na primer:

```go
go doWork()

time.Sleep(time.Second)
```

ne garantuje da je `doWork` završen nakon jedne sekunde.

Može završiti:

* pre isteka jedne sekunde;
* tačno nakon jedne sekunde;
* posle više od jedne sekunde.

`Sleep` predstavlja vremensko čekanje, a ne sinhronizaciju.

Ako želimo da čekamo završetak rada, treba koristiti odgovarajući mehanizam koordinacije.

Na primer:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()
    doWork()
}()

wg.Wait()
```

Ovde program čeka **stvarni završetak rada**, a ne proizvoljni vremenski interval.

---

### Pitanje 25

**Koja je osnovna razlika između "čekanja određeno vreme" i "čekanja događaja"?**

**Odgovor:**

`time.Sleep` čeka vreme:

```go
time.Sleep(time.Second)
```

Dok channel može čekati događaj:

```go
<-done
```

To je fundamentalna razlika.

Kod `Sleep` kažemo:

> "Sačekaj jednu sekundu."

Kod channel-a kažemo:

> "Sačekaj dok se ne dogodi određeni događaj."

U concurrency kodu drugo je obično ispravan model kada je potrebno koordinisati gorutine.

Na primer:

```go
done := make(chan struct{})

go func() {
    doWork()
    close(done)
}()

<-done
```

Ovde vreme potrebno za `doWork` nije unapred poznato niti je bitno.

Bitno je samo da se događaj:

```text
doWork completed
```

dogodi pre nego što receiver nastavi.

---

## Ključne lekcije ovog dela

Nakon ovog dela treba razumeti sledeće:

1. `go` statement ne garantuje trenutno izvršavanje nove gorutine.
2. Završetak `main` goroutine završava proces.
3. Goroutine scheduling ne treba tretirati kao deterministički.
4. Concurrency nije isto što i parallelism.
5. Redosled izvršavanja gorutina ne treba pretpostavljati bez eksplicitne sinhronizacije.
6. Nebaferovani channel može blokirati sender dok receiver nije spreman.
7. Receive može blokirati dok ne postoji odgovarajući send.
8. Channel može služiti kao mehanizam koordinacije, a ne samo prenosa podataka.
9. `time.Sleep` nije zamena za pravilnu sinhronizaciju.
10. U concurrency kodu treba čekati događaj ili stanje, a ne proizvoljni vremenski interval, kada god je to moguće.

---

# Interview Questions — Beginner

## Deo #4/5

### 4. Osnovna komunikacija između goroutina

#### Pitanje 4.1

**Šta se dešava kada goroutina pošalje vrednost na nebufferizovani kanal?**

Kod nebufferizovanog kanala (`unbuffered channel`), slanje ne može da se završi samo smeštanjem vrednosti u interni bafer, zato što takav bafer ne postoji.

Pošiljalac:

```go
ch <- value
```

blokira se dok druga goroutina ne bude spremna da primi vrednost:

```go
value := <-ch
```

Primer:

```go
package main

import "fmt"

func main() {
	ch := make(chan int)

	go func() {
		ch <- 42
	}()

	value := <-ch

	fmt.Println(value)
}
```

Ovde goroutina koja izvršava:

```go
ch <- 42
```

čeka dok `main` goroutina ne izvrši:

```go
value := <-ch
```

Tek tada može da se izvrši komunikacija između dve goroutine.

Ovo je jedna od osnovnih karakteristika nebufferizovanih kanala:

> **Slanje i prijem predstavljaju koordinisanu tačku susreta između goroutina.**

---

#### Pitanje 4.2

**Šta se dešava kada goroutina primi vrednost sa nebufferizovanog kanala, ali trenutno nema pošiljaoca?**

Ako niko trenutno ne šalje vrednost na kanal:

```go
value := <-ch
```

operacija prijema blokira izvršavanje goroutine.

Na primer:

```go
ch := make(chan int)

value := <-ch
fmt.Println(value)
```

Ako se nikada ne pojavi pošiljalac:

```go
ch <- 42
```

goroutina ostaje blokirana.

Ako je to jedina aktivna goroutina, program može završiti sa:

```text
fatal error: all goroutines are asleep - deadlock!
```

Zato kod rada sa kanalima moramo razumeti ne samo **šta se šalje i prima**, već i **ko je odgovoran da druga strana komunikacije zaista postoji**.

---

### 5. Buffering i osnovna razlika između kanala

#### Pitanje 5.1

**Koja je osnovna razlika između buffered i unbuffered kanala?**

Nešto poput:

```go
ch := make(chan int)
```

kreira nebufferizovani kanal.

Dok:

```go
ch := make(chan int, 3)
```

kreira kanal kapaciteta `3`.

Kod nebufferizovanog kanala, slanje zavisi od toga da li postoji odgovarajući prijem.

Kod bufferizovanog kanala, slanje može da se završi dok god u bufferu postoji slobodan prostor.

Primer:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20
```

Oba slanja mogu da se izvrše bez trenutnog prijemnika zato što kanal može da zadrži dve vrednosti.

Ali:

```go
ch <- 30
```

blokiraće se dok neko ne preuzme jednu od prethodno poslatih vrednosti.

---

#### Pitanje 5.2

**Da li buffered channel znači da slanje nikada neće blokirati?**

Ne.

Buffer samo omogućava određenu količinu asinhronog slanja.

Ako je:

```go
ch := make(chan int, 2)
```

i buffer već sadrži:

```text
[10, 20]
```

sledeće:

```go
ch <- 30
```

blokiraće se dok se ne oslobodi prostor u bufferu.

Dakle:

```text
unbuffered channel
    ↓
send zahteva receiver

buffered channel
    ↓
send može koristiti slobodan buffer
    ↓
kada se buffer napuni → send blokira
```

Važno je razumeti da buffering **ne uklanja potrebu za koordinacijom**. On samo menja trenutak u kojem može doći do blokiranja.

---

### 6. Channel directions

#### Pitanje 6.1

**Šta predstavljaju channel directions u Go-u?**

Channel može biti:

* bidirectional — može da šalje i prima;
* send-only — može samo da šalje;
* receive-only — može samo da prima.

Bidirectional kanal:

```go
chan int
```

Send-only:

```go
chan<- int
```

Receive-only:

```go
<-chan int
```

Primer:

```go
func producer(ch chan<- int) {
	ch <- 42
}
```

Funkcija `producer` može da šalje vrednosti, ali ne može da ih prima.

S druge strane:

```go
func consumer(ch <-chan int) {
	value := <-ch
	fmt.Println(value)
}
```

`consumer` može da prima vrednosti, ali ne može da ih šalje.

---

#### Pitanje 6.2

**Zašto su channel directions korisni?**

Channel directions omogućavaju da API jasno definiše odgovornost funkcije.

Na primer:

```go
func producer(ch chan<- int)
```

govori čitaocu:

> Ova funkcija proizvodi podatke i šalje ih u kanal.

Dok:

```go
func consumer(ch <-chan int)
```

govori:

> Ova funkcija konzumira podatke iz kanala.

Time se smanjuje mogućnost greške i povećava čitljivost koda.

Još važnije, compiler može da proveri da li pokušavamo da koristimo kanal na način koji nije dozvoljen njegovim tipom.

Na primer:

```go
func consumer(ch <-chan int) {
	ch <- 10
}
```

nije dozvoljeno, jer je `ch` receive-only kanal.

---

### 7. `range` nad kanalom

#### Pitanje 7.1

**Kako se koristi `range` za čitanje vrednosti iz kanala?**

Kanal možemo koristiti kao izvor vrednosti u `for range` petlji:

```go
for value := range ch {
	fmt.Println(value)
}
```

Petlja nastavlja da prima vrednosti sve dok kanal ne bude zatvoren.

Primer:

```go
ch := make(chan int)

go func() {
	ch <- 1
	ch <- 2
	ch <- 3

	close(ch)
}()

for value := range ch {
	fmt.Println(value)
}
```

Rezultat:

```text
1
2
3
```

`range` se završava nakon što je kanal zatvoren i nakon što su sve vrednosti koje su već poslate pročitane.

---

#### Pitanje 7.2

**Zašto je `close` važan kada koristimo `range` nad kanalom?**

`range` mora imati način da zna da više neće biti novih vrednosti.

Zatvaranje kanala predstavlja signal:

> Više neće biti novih send operacija.

Zato je tipičan obrazac:

```go
go func() {
	defer close(ch)

	for _, value := range values {
		ch <- value
	}
}()
```

A receiver:

```go
for value := range ch {
	process(value)
}
```

Ovde sender poseduje lifecycle kanala i odgovoran je za njegovo zatvaranje.

Važno:

**Receiver uglavnom ne treba da zatvara kanal koji nije njegov.**

Praktično pravilo je:

> **Goroutina koja proizvodi vrednosti i zna da je proizvodnja završena obično je odgovorna za `close`.**

---

### 8. Osnovni `select`

#### Pitanje 8.1

**Čemu služi `select` u Go concurrency modelu?**

`select` omogućava goroutini da čeka više channel operacija istovremeno.

Primer:

```go
select {
case value := <-ch1:
	fmt.Println("ch1:", value)

case value := <-ch2:
	fmt.Println("ch2:", value)
}
```

Ako je jedna od operacija spremna, `select` može da je izabere.

Ako nijedna nije spremna, `select` blokira dok neka od navedenih operacija ne postane moguća.

---

#### Pitanje 8.2

**Koja je osnovna razlika između `select` i običnog channel receive-a?**

Kod:

```go
value := <-ch
```

čekamo jednu konkretnu operaciju.

Kod:

```go
select {
case value := <-ch1:
	// ...
case value := <-ch2:
	// ...
}
```

čekamo više mogućih događaja.

To omogućava obrasce kao što su:

* čekanje na više izvora podataka;
* timeout;
* cancellation;
* koordinacija više goroutina;
* reagovanje na prvi kanal koji postane spreman.

`select` je zato jedan od osnovnih mehanizama za izgradnju strukturisane konkurentnosti u Go-u.

---

## Ključne činjenice za Beginner nivo

Na ovom nivou kandidat treba da razume sledeće:

1. Slanje na nebufferizovani kanal može da blokira.
2. Prijem sa kanala može da blokira.
3. Buffered channel ima ograničen kapacitet.
4. Pun buffered channel može da blokira sender.
5. Channel može biti bidirectional, send-only ili receive-only.
6. `range` omogućava kontinuirano čitanje iz kanala.
7. `close` signalizira da više neće biti novih vrednosti.
8. `range` nad kanalom završava kada je kanal zatvoren i potrošene su sve preostale vrednosti.
9. `select` omogućava čekanje na više channel operacija.
10. Odgovornost za lifecycle kanala treba jasno definisati, naročito kada postoji više goroutina.

---

# Interview Questions — Beginner

## Deo #5/5

### 9. Goroutine lifecycle

#### Pitanje 9.1

**Da li `go` ključna reč garantuje da će goroutina završiti svoj posao pre nego što se program završi?**

Ne.

Pokretanje goroutine:

```go
go doWork()
```

ne znači da će `main` čekati da `doWork` završi.

Na primer:

```go
func main() {
	go func() {
		fmt.Println("hello")
	}()

	fmt.Println("main finished")
}
```

Program može završiti pre nego što goroutina dobije priliku da izvrši `Println`.

`main` goroutina predstavlja početnu tačku programa. Kada se `main` završi, proces se završava, zajedno sa svim preostalim goroutinama.

Zato mora postojati eksplicitna koordinacija ako je rezultat goroutine važan.

Na primer, u jednostavnom primeru možemo koristiti kanal:

```go
done := make(chan struct{})

go func() {
	defer close(done)

	doWork()
}()

<-done
```

U realnom kodu često ćemo koristiti `sync.WaitGroup`, `context.Context` ili kombinaciju više concurrency mehanizama, u zavisnosti od problema koji rešavamo.

---

### 10. Da li je pokretanje goroutine skupo?

#### Pitanje 10.1

**Da li je goroutine isto što i OS thread?**

Ne.

Goroutine je Go-ov concurrency primitive kojim upravlja Go runtime, dok je OS thread resurs kojim upravlja operativni sistem.

Jedan OS thread može izvršavati Go kod za više različitih goroutina tokom vremena.

Pojednostavljeno:

```text
Process
  │
  ├── OS Thread
  │     ├── goroutine A
  │     ├── goroutine B
  │     └── goroutine C
  │
  └── OS Thread
        ├── goroutine D
        └── goroutine E
```

Go runtime raspoređuje goroutine na raspoložive execution resources.

Zbog toga Go može da podrži veliki broj goroutina bez potrebe da svaka goroutina direktno odgovara jednom OS thread-u.

---

#### Pitanje 10.2

**Da li zbog toga treba pokretati goroutine za svaku sitnicu?**

Ne.

Goroutine je relativno lagan concurrency primitive, ali nije besplatan.

Svaka goroutine zahteva određene runtime resurse i može imati indirektne troškove kroz:

* scheduling;
* memoriju;
* channel komunikaciju;
* synchronization;
* contention;
* garbage collection;
* lifecycle management.

Najvažnije pitanje nije:

> "Da li mogu da koristim goroutine?"

nego:

> "Da li ovaj posao zaista zahteva konkurentno izvršavanje?"

---

### 11. Data race

#### Pitanje 11.1

**Šta je data race?**

Data race nastaje kada više goroutina istovremeno pristupa istoj memorijskoj lokaciji, najmanje jedan pristup je upis, a pristupi nisu pravilno sinhronizovani.

Na primer:

```go
var counter int

go func() {
	counter++
}()

go func() {
	counter++
}()
```

Obe goroutine pristupaju promenljivoj `counter`.

Operacija:

```go
counter++
```

nije konceptualno jedna nedeljiva operacija. Ona uključuje čitanje, izmenu i upis vrednosti.

Bez odgovarajuće sinhronizacije rezultat nije bezbedno definisan.

Go pruža race detector koji možemo koristiti tokom razvoja i testiranja:

```bash
go test -race ./...
```

ili:

```bash
go run -race .
```

---

### 12. Deadlock

#### Pitanje 12.1

**Šta je deadlock?**

Deadlock nastaje kada goroutine ili grupa goroutina čeka uslov koji nikada neće biti ispunjen.

Jedan od najjednostavnijih primera:

```go
func main() {
	ch := make(chan int)

	ch <- 42

	fmt.Println("done")
}
```

Ovde `main` pokušava da pošalje vrednost na nebufferizovani kanal.

Ali nema druge goroutine koja prima tu vrednost.

Zato se `main` blokira.

Pošto nema druge goroutine koja može da nastavi komunikaciju, runtime detektuje deadlock.

---

### 13. Goroutine leak

#### Pitanje 13.1

**Šta je goroutine leak?**

Goroutine leak nastaje kada goroutine ostane aktivna ili blokirana duže nego što je predviđeno, zato što nema način da završi svoj posao.

Primer:

```go
func worker(ch <-chan int) {
	for {
		value := <-ch
		fmt.Println(value)
	}
}
```

Ako više niko nikada ne šalje vrednosti i nema cancellation mehanizma, ova goroutine može ostati blokirana neograničeno dugo.

Problem postaje ozbiljniji kada aplikacija kontinuirano kreira takve goroutine.

Na primer:

```text
request 1 → goroutine leak
request 2 → goroutine leak
request 3 → goroutine leak
...
request N → goroutine leak
```

Vremenom broj goroutina može rasti i negativno uticati na aplikaciju.

Zbog toga je lifecycle goroutine jedan od ključnih aspekata production concurrency koda.

---

### 14. Osnovni cancellation obrazac

#### Pitanje 14.1

**Kako možemo omogućiti goroutini da zna kada treba da prekine rad?**

Jedan od osnovnih pristupa jeste `context.Context`.

Na primer:

```go
func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			return

		default:
			// work
		}
	}
}
```

Pozivalac može otkazati rad:

```go
ctx, cancel := context.WithCancel(context.Background())

go worker(ctx)

cancel()
```

Goroutine tada dobija signal kroz:

```go
<-ctx.Done()
```

Ovaj obrazac je posebno važan kod servera, request lifecycle-a i background workera.

---

### 15. Ko treba da zatvara kanal?

#### Pitanje 15.1

**Da li receiver treba da zatvori kanal kada završi sa čitanjem?**

Uobičajeno — ne.

Najčešće kanal zatvara strana koja proizvodi vrednosti i zna da više nema šta da pošalje.

Na primer:

```go
func producer(ch chan<- int) {
	defer close(ch)

	for i := 0; i < 10; i++ {
		ch <- i
	}
}
```

Receiver:

```go
for value := range ch {
	fmt.Println(value)
}
```

Ovde producer poseduje lifecycle toka podataka.

Ako receiver zatvori kanal dok producer još pokušava da šalje:

```go
close(ch)
```

kasniji send:

```go
ch <- value
```

može izazvati:

```text
panic: send on closed channel
```

Zato je ownership kanala važan deo API dizajna.

---

### 16. Šta se dešava kada primimo vrednost sa zatvorenog kanala?

#### Pitanje 16.1

**Da li receive sa zatvorenog kanala izaziva panic?**

Ne.

Ako je kanal zatvoren i više nema vrednosti u njemu:

```go
value, ok := <-ch
```

dobijamo:

```text
value = zero value
ok = false
```

Primer:

```go
ch := make(chan int)

close(ch)

value, ok := <-ch

fmt.Println(value)
fmt.Println(ok)
```

Rezultat će biti:

```text
0
false
```

Ovo je veoma važno jer omogućava bezbednu detekciju kraja toka podataka.

Zbog toga možemo koristiti:

```go
for {
	value, ok := <-ch

	if !ok {
		break
	}

	process(value)
}
```

ili jednostavnije:

```go
for value := range ch {
	process(value)
}
```

---

### 17. Najvažniji Beginner mentalni model

Kandidat koji završi Beginner nivo ne treba samo da zna sintaksu:

```go
go ...
ch <- value
value := <-ch
close(ch)
select { ... }
```

Već treba da počne da razmišlja u terminima:

```text
Goroutine
    │
    │ lifecycle
    ▼
Channel
    │
    ├── send
    ├── receive
    └── close
    │
    ▼
Synchronization
    │
    ├── blocking
    ├── coordination
    └── cancellation
```

Drugim rečima, concurrency nije samo pitanje:

> "Kako da pokrenem više stvari?"

nego:

> "Kako da bezbedno koordiniram njihov rad, lifecycle i komunikaciju?"

To je ključna razlika između početnog poznavanja Go concurrency modela i stvarnog razumevanja problema koje concurrency rešava.

---

## Beginner — završna lista pitanja

Na kraju ovog nivoa kandidat treba da bude sposoban da odgovori na pitanja kao što su:

1. Šta je goroutine?
2. Kako se pokreće goroutine?
3. Da li `go` čeka da funkcija završi?
4. Šta se dešava kada `main` završi?
5. Šta je channel?
6. Kako se šalje vrednost kroz kanal?
7. Kako se prima vrednost iz kanala?
8. Koja je razlika između buffered i unbuffered kanala?
9. Kada send blokira?
10. Kada receive blokira?
11. Šta su channel directions?
12. Šta predstavljaju `chan<-` i `<-chan`?
13. Kako funkcioniše `range` nad kanalom?
14. Čemu služi `close`?
15. Ko bi trebalo da zatvori kanal?
16. Šta se događa pri receive-u sa zatvorenog kanala?
17. Šta je `select`?
18. Šta je deadlock?
19. Šta je data race?
20. Šta je goroutine leak?
21. Kako se osnovno implementira cancellation?
22. Zašto je lifecycle goroutine važan?
23. Zašto je channel ownership važan?
24. Kako `go test -race` pomaže u otkrivanju concurrency problema?

---

## Kriterijum za prelazak na Junior nivo

Beginner nivo je uspešno savladan kada kandidat može samostalno da:

* kreira i pokrene goroutine;
* koristi channel za komunikaciju;
* razlikuje buffered i unbuffered channel;
* koristi send-only i receive-only channel;
* koristi `range` i `close`;
* koristi osnovni `select`;
* prepozna jednostavan deadlock;
* objasni osnovni goroutine leak;
* objasni osnovni data race;
* razume zašto goroutine mora imati jasan lifecycle;
* razume osnovni princip cancellation-a;
* objasni ko je odgovoran za zatvaranje kanala.

Sledeći nivo uvodi složenije probleme i zahteva da kandidat ne samo koristi concurrency primitive, već razume njihove posledice i pravilno ih primenjuje u konkretnim scenarijima.

---

[Prelazak na **Junior — Interview Questions**](02-junior.md)