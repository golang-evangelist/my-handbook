# Interview Questions — Junior

## Deo #1/7

## 1. Buffered vs. Unbuffered Channels

### Pitanje 1.1

**Koja je suštinska razlika između buffered i unbuffered channel-a?**

Osnovna razlika je u tome što buffered channel poseduje interni bafer za određeni broj elemenata, dok unbuffered channel nema prostor za skladištenje vrednosti.

Unbuffered channel:

```go
ch := make(chan int)
```

Buffered channel:

```go
ch := make(chan int, 3)
```

Kod unbuffered channel-a:

```go
ch <- value
```

send mora da se uskladi sa odgovarajućim receive-om.

Kod buffered channel-a, send može da završi bez trenutnog receiver-a sve dok u baferu postoji slobodan kapacitet.

Na primer:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20
```

Oba send-a mogu da se izvrše bez druge goroutine koja trenutno prima vrednosti.

Treći send:

```go
ch <- 30
```

blokiraće dok se iz kanala ne pročita najmanje jedna vrednost.

---

### Pitanje 1.2

**Da li buffered channel znači da je komunikacija asinhrona?**

Ne nužno.

Buffered channel omogućava određeni stepen decoupling-a između sender-a i receiver-a, ali ne uklanja blocking semantics.

Na primer:

```go
ch := make(chan int, 2)

ch <- 1
ch <- 2
```

sender može nastaviti jer bafer ima slobodan prostor.

Ali:

```go
ch <- 3
```

blokira kada je bafer pun.

Dakle, buffered channel možemo posmatrati kao ograničeni queue:

```text
Sender
   │
   ▼
┌───────────────┐
│ 1 │ 2 │   │   │
└───────────────┘
       ▲
       │
    Receiver
```

Bafer ne znači "nikada ne blokiraj".

Bafer znači:

> "Dozvoli određen broj vrednosti da čeka između sender-a i receiver-a."

---

## 2. Blocking Semantics

### Pitanje 2.1

**Kada send na unbuffered channel-u blokira?**

Send na unbuffered channel-u blokira dok druga goroutine ne bude spremna da primi vrednost.

Na primer:

```go
ch := make(chan int)

ch <- 42
```

Ako nema receiver-a, trenutna goroutine čeka.

Sa druge strane:

```go
go func() {
	value := <-ch
	fmt.Println(value)
}()

ch <- 42
```

ovde sender i receiver mogu da se sinhronizuju.

Konceptualno:

```text
Sender                     Receiver
   │                          │
   │────── 42 ───────────────>│
   │                          │
   ▼                          ▼
nastavlja                nastavlja
```

Kod unbuffered channel-a komunikacija predstavlja istovremeno:

1. prenos podatka;
2. synchronization point.

---

### Pitanje 2.2

**Kada receive na unbuffered channel-u blokira?**

Receive blokira kada nema vrednosti koja može biti primljena.

Na primer:

```go
ch := make(chan int)

value := <-ch
```

Ako druga goroutine ne izvrši:

```go
ch <- value
```

receiver ostaje blokiran.

Zato channel operacije treba posmatrati kao deo synchronization modela, a ne samo kao način prenosa podataka.

---

## 3. Blocking kod Buffered Channels

### Pitanje 3.1

**Kada send na buffered channel-u blokira?**

Send blokira kada je bafer pun.

Na primer:

```go
ch := make(chan int, 2)

ch <- 1
ch <- 2
```

Bafer je sada pun:

```text
┌───────┬───────┐
│   1   │   2   │
└───────┴───────┘
       full
```

Sledeći send:

```go
ch <- 3
```

mora da sačeka dok receiver ne oslobodi prostor.

Ako druga goroutine izvrši:

```go
value := <-ch
```

jedna vrednost napušta bafer i sender može da nastavi.

---

### Pitanje 3.2

**Kada receive na buffered channel-u blokira?**

Receive blokira kada nema vrednosti u kanalu.

Na primer:

```go
ch := make(chan int, 2)

value := <-ch
```

Bafer je prazan, pa receiver čeka.

Ako druga goroutine izvrši:

```go
ch <- 100
```

receive može da se nastavi.

Dakle, kod buffered channel-a postoje dve osnovne granice:

```text
send  → blokira kada je buffer pun
receive → blokira kada je buffer prazan
```

Ovo je jedan od najvažnijih mentalnih modela za razumevanje channel-a.

---

## 4. Channel Capacity

### Pitanje 4.1

**Kako možemo saznati kapacitet channel-a?**

Koristimo `cap`:

```go
ch := make(chan int, 5)

fmt.Println(cap(ch))
```

Rezultat:

```text
5
```

Za broj trenutno dostupnih elemenata koristimo `len`:

```go
fmt.Println(len(ch))
```

Na primer:

```go
ch := make(chan int, 5)

ch <- 10
ch <- 20

fmt.Println(len(ch))
fmt.Println(cap(ch))
```

Rezultat:

```text
2
5
```

Dakle:

```text
len(ch) = broj trenutno dostupnih elemenata
cap(ch) = ukupni kapacitet bafera
```

Važno je razumeti da `len(ch)` i `cap(ch)` nisu zamena za pravilnu synchronization logiku.

Posebno u concurrent kodu, stanje koje pročitate može se promeniti odmah nakon tog čitanja.

---

## 5. Channel Directions

### Pitanje 5.1

**Zašto postoje channel directions?**

Go omogućava da channel u funkcionalnom API-ju ograničimo na određeni smer komunikacije.

Tri oblika su:

```go
chan T
chan<- T
<-chan T
```

`chan T` je bidirectional channel.

Može da šalje i prima:

```go
func process(ch chan int) {
	ch <- 10
	value := <-ch

	_ = value
}
```

`chan<- T` je send-only channel:

```go
func produce(ch chan<- int) {
	ch <- 10
}
```

`<-chan T` je receive-only channel:

```go
func consume(ch <-chan int) {
	value := <-ch

	fmt.Println(value)
}
```

Ovo je važno za API dizajn zato što funkciji možemo eksplicitno dati samo operacije koje su joj potrebne.

---

### Pitanje 5.2

**Da li bidirectional channel možemo proslediti funkciji koja očekuje send-only channel?**

Da.

Na primer:

```go
ch := make(chan int)

produce(ch)
```

ako je funkcija:

```go
func produce(ch chan<- int) {
	ch <- 10
}
```

Go dozvoljava korišćenje bidirectional channel-a tamo gde je potreban send-only channel.

Analogno tome, bidirectional channel možemo proslediti funkciji koja očekuje receive-only channel:

```go
func consume(ch <-chan int) {
	fmt.Println(<-ch)
}
```

Ovo omogućava da se channel ownership i dozvoljene operacije izraze kroz tip sistema.

---

## 6. Zašto je Directional Channel važan u API dizajnu?

### Pitanje 6.1

**Šta je bolje?**

```go
func worker(ch chan int)
```

ili:

```go
func worker(ch <-chan int)
```

ako `worker` samo prima podatke?

Druga verzija je preciznija:

```go
func worker(ch <-chan int)
```

Ona dokumentuje nameru funkcije i sprečava slučajan send unutar funkcije.

Isto važi za producer:

```go
func producer(ch chan<- int)
```

Time API postaje samodokumentujući:

```text
producer
    │
    │ chan<-
    ▼
 channel
    │
    │ <-chan
    ▼
consumer
```

Ovo je posebno korisno kada concurrency kod postane veći i kada više komponenti deli iste channel-e.

---

## 7. Junior mentalni model

Na Junior nivou kandidat više ne treba samo da zna:

```go
go worker()
```

ili:

```go
ch <- value
```

Već treba da razume šta se dešava kada se izvršavanje susretne sa channel operacijom.

Osnovni model treba da bude:

```text
                 CHANNEL
                    │
          ┌─────────┴─────────┐
          │                   │
       unbuffered           buffered
          │                   │
          │              ┌────┴────┐
          │              │         │
      sender/receiver   capacity   queue
       synchronization
```

I za svaku channel operaciju treba postaviti pitanje:

> **Da li ova operacija može da blokira? Ako može — pod kojim uslovom?**

To pitanje predstavlja osnovu za razumevanje složenijih concurrency problema.

---

# Interview Questions — Junior

## Deo #2/7

## 8. `range` nad Channel-om

### Pitanje 8.1

**Kako `range` funkcioniše kada se koristi nad channel-om?**

`range` omogućava da goroutine uzima vrednosti iz channel-a sve dok channel ne bude zatvoren.

Na primer:

```go
for value := range ch {
	fmt.Println(value)
}
```

Ovo je konceptualno ekvivalentno obrascu:

```go
for {
	value, ok := <-ch
	if !ok {
		break
	}

	fmt.Println(value)
}
```

Kod channel-a je važno razumeti da `range` ne završava samo zato što trenutno nema vrednosti.

Ako je channel otvoren i prazan, `range` čeka na sledeću vrednost.

Zato:

```go
for value := range ch {
	// ...
}
```

može da blokira sve dok:

1. ne stigne nova vrednost;
2. ili channel ne bude zatvoren.

---

### Pitanje 8.2

**Da li `range` automatski zatvara channel?**

Ne.

`range` samo čita iz channel-a.

Ne izvršava:

```go
close(ch)
```

Kada `range` završi, to znači da je channel zatvoren i da više nema vrednosti za čitanje.

Na primer:

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

Nakon `close(ch)`, `range` može da završi jer zna da više neće biti novih vrednosti.

---

## 9. `close` Channel-a

### Pitanje 9.1

**Zašto se channel zatvara?**

`close` signalizira receiver-ima:

> **"Više neće biti poslatih vrednosti na ovaj channel."**

Na primer:

```go
func producer(ch chan<- int) {
	defer close(ch)

	ch <- 10
	ch <- 20
	ch <- 30
}
```

Consumer može koristiti:

```go
for value := range ch {
	fmt.Println(value)
}
```

Channel close ovde predstavlja signal završetka toka podataka.

Važno je razlikovati:

```text
slanje vrednosti
        ↓
"evo podatka"

zatvaranje channel-a
        ↓
"nema više podataka"
```

`close` nije poruka koja predstavlja jednu dodatnu poslovnu vrednost.

On menja stanje channel-a i omogućava receiver-u da sazna da je stream završen.

---

### Pitanje 9.2

**Šta se dešava kada pokušamo da primimo vrednost sa zatvorenog channel-a?**

Ako je channel zatvoren i više nema buffered vrednosti, receive operacija odmah vraća zero value odgovarajućeg tipa.

Na primer:

```go
ch := make(chan int)

close(ch)

value := <-ch

fmt.Println(value)
```

Rezultat je:

```text
0
```

Zbog toga postoji drugi oblik receive operacije:

```go
value, ok := <-ch
```

Ako je channel zatvoren i ispražnjen:

```text
value = zero value
ok    = false
```

Na primer:

```go
value, ok := <-ch

if !ok {
	fmt.Println("channel je zatvoren")
}
```

Ovo je važno zato što sama vrednost ne govori nužno da li je channel zatvoren.

Na primer, `0` može biti sasvim legitimna vrednost koju je sender poslao.

---

## 10. Closed Channel i Buffered Values

### Pitanje 10.1

**Da li `close(ch)` odmah odbacuje vrednosti koje se već nalaze u buffered channel-u?**

Ne.

Ako buffered channel sadrži vrednosti, one mogu i dalje biti pročitane nakon `close`.

Na primer:

```go
ch := make(chan int, 3)

ch <- 10
ch <- 20
ch <- 30

close(ch)
```

Receiver i dalje može da pročita:

```go
fmt.Println(<-ch) // 10
fmt.Println(<-ch) // 20
fmt.Println(<-ch) // 30
```

Tek nakon što se buffered vrednosti potroše, dalji receive vraća zero value uz `ok == false`.

Dakle:

```text
           close(ch)
               │
               ▼
        ┌───────────────┐
        │ channel closed│
        └───────┬───────┘
                │
        postoje buffered
           vrednosti?
          /           \
        DA             NE
        │               │
        ▼               ▼
    čitaj ih        receive dobija
    normalno        zero value + false
```

Ovo je posebno važno kod producer/consumer obrazaca.

---

## 11. Ko treba da zatvori Channel?

### Pitanje 11.1

**Ko je odgovoran za zatvaranje channel-a?**

Najčešće channel treba da zatvori komponenta koja je odgovorna za slanje vrednosti i zna da više neće slati.

Drugim rečima:

> **Sender koji poseduje tok podataka obično poseduje i odgovornost za `close`.**

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

Consumer samo čita:

```go
for value := range producer() {
	fmt.Println(value)
}
```

Consumer nema potrebu da zatvara channel.

Ovakav model smanjuje mogućnost greške jer je ownership jasan.

---

### Pitanje 11.2

**Zašto receiver uglavnom ne treba da zatvara channel?**

Receiver ne mora nužno znati da li postoje drugi sender-i koji još uvek šalju vrednosti.

Ako receiver zatvori channel dok drugi sender još uvek pokušava:

```go
ch <- value
```

sender može izazvati panic:

```text
panic: send on closed channel
```

Zato je veoma opasan obrazac:

```go
func consumer(ch chan int) {
	defer close(ch) // potencijalno pogrešno
}
```

Ako consumer nije vlasnik send lifecycle-a, on ne treba da zatvara channel.

---

## 12. "Closing Principle"

### Pitanje 12.1

**Koji praktični princip možemo koristiti kada odlučujemo ko zatvara channel?**

Koristan princip je:

> **The sender should close the channel.**

Preciznije:

> **Komponenta koja kontroliše kraj slanja treba da kontroliše i zatvaranje channel-a.**

Na primer:

```go
func producer() <-chan int {
	ch := make(chan int)

	go func() {
		defer close(ch)

		for i := 0; i < 5; i++ {
			ch <- i
		}
	}()

	return ch
}
```

Ovde je ownership vrlo jasan:

```text
producer
   │
   ├── šalje
   ├── odlučuje kada završava
   └── zatvara channel
```

Consumer:

```go
func consumer(ch <-chan int) {
	for value := range ch {
		fmt.Println(value)
	}
}
```

samo konzumira stream.

---

## 13. Channel Ownership

### Pitanje 13.1

**Šta znači channel ownership?**

Channel ownership predstavlja odgovornost za lifecycle channel-a i tok podataka koji preko njega prolazi.

U praksi ownership obuhvata pitanja:

* ko kreira channel;
* ko šalje;
* ko prima;
* ko odlučuje kada je slanje završeno;
* ko zatvara channel;
* ko je odgovoran za eventualnu koordinaciju gašenja.

Na primer:

```go
func generate() <-chan int {
	ch := make(chan int)

	go func() {
		defer close(ch)

		ch <- 1
		ch <- 2
		ch <- 3
	}()

	return ch
}
```

Funkcija `generate` je owner producer lifecycle-a.

Spoljašnji kod dobija:

```go
<-chan int
```

što znači da može samo da prima.

Ovo je snažan API dizajn jer ownership nije samo dokumentovan komentarom — on je delimično izražen kroz Go type system.

---

## 14. Zašto vraćati `<-chan T` umesto `chan T`?

### Pitanje 14.1

Posmatrajmo:

```go
func generate() chan int
```

i:

```go
func generate() <-chan int
```

Zašto je druga verzija često bolja?

Zato što caller-u ne treba omogućiti operacije koje nisu deo njegovog ugovora.

Ako funkcija proizvodi podatke:

```go
func generate() <-chan int
```

caller može:

```go
for value := range generate() {
	fmt.Println(value)
}
```

ali ne može:

```go
generate() <- 10
```

Niti može:

```go
close(generate())
```

Time se smanjuje površina za greške.

Ovo je primer principa:

> **Expose only the capabilities that the caller needs.**

Directional channels zato nisu samo sintaktička pogodnost, već alat za dizajn concurrency API-ja.

---

## 15. `range` + `close` kao standardni producer/consumer obrazac

### Pitanje 15.1

**Koji je idiomatski način da producer signalizira consumer-u da je završio proizvodnju?**

Producer zatvara channel:

```go
func producer() <-chan int {
	ch := make(chan int)

	go func() {
		defer close(ch)

		for i := 1; i <= 5; i++ {
			ch <- i
		}
	}()

	return ch
}
```

Consumer koristi `range`:

```go
for value := range producer() {
	fmt.Println(value)
}
```

Tok izgleda ovako:

```text
Producer
   │
   │ 1
   │ 2
   │ 3
   │ 4
   │ 5
   ▼
 channel
   │
   │ close
   ▼
Consumer
   │
   └── range završava
```

Ovaj obrazac je jedan od osnovnih building block-ova Go concurrency modela.

---

## 16. Tipične greške Junior developera

### Pitanje 16.1

**Šta se dešava ako koristimo `range` nad channel-om koji nikada neće biti zatvoren?**

Ako nema više vrednosti, `range` može ostati blokiran zauvek.

Na primer:

```go
func producer(ch chan<- int) {
	ch <- 1
	ch <- 2
	// channel nikada nije zatvoren
}
```

Consumer:

```go
for value := range ch {
	fmt.Println(value)
}
```

Nakon vrednosti `1` i `2`, consumer čeka sledeću vrednost.

Ako je producer završio i niko više neće slati:

```text
consumer
   │
   ▼
range
   │
   ▼
čeka zauvek
```

Ovo može dovesti do:

* blokiranih goroutine-a;
* goroutine leak-a;
* deadlock-a u zavisnosti od ostatka programa;
* testova koji vise;
* problema pri graceful shutdown-u.

---

### Pitanje 16.2

**Šta se dešava ako dva različita dela programa pokušaju da zatvore isti channel?**

Drugi `close` izaziva panic:

```text
panic: close of closed channel
```

Zbog toga je važno imati jedno jasno mesto koje kontroliše close lifecycle.

Loš dizajn:

```text
producer A ──┐
             ├── channel ── consumer
producer B ──┘

A može close
B može close
```

Ovakav dizajn stvara nejasan ownership.

Bolji dizajn je centralizovati odgovornost:

```text
producer(s)
     │
     ▼
 coordinator
     │
     ├── zna kada je slanje završeno
     │
     └── close(channel)
```

Kod više sender-a pitanje zatvaranja postaje posebno važno i zahteva eksplicitnu koordinaciju.

---

## Junior checklist

Na ovom nivou kandidat treba pouzdano da razume sledeće:

* `range` nad channel-om čita dok channel ne bude zatvoren;
* `range` sam ne zatvara channel;
* `close` signalizira da više neće biti novih send-ova;
* zatvoren buffered channel može još sadržati vrednosti;
* receive sa zatvorenog i ispražnjenog channel-a vraća zero value;
* `value, ok := <-ch` omogućava detekciju zatvorenog channel-a;
* sender koji kontroliše kraj slanja obično zatvara channel;
* receiver uglavnom ne treba da zatvara channel;
* directional channels izražavaju dozvoljeni smer komunikacije;
* `<-chan T` može ograničiti caller-a na receive;
* `chan<- T` može ograničiti caller-a na send;
* `range` nad channel-om koji nikada neće biti zatvoren može ostati blokiran;
* višestruko zatvaranje istog channel-a izaziva panic.

# Interview Questions — Junior

## Deo #3/7

## 17. Blokiranje pri slanju na Channel

### Pitanje 17.1

**Kada send operacija na channel-u blokira goroutine?**

Send operacija:

```go
ch <- value
```

blokira sender-a kada channel u tom trenutku ne može da prihvati vrednost.

Kod **unbuffered channel-a**, send može da se izvrši kada postoji receiver koji je spreman da primi vrednost.

Na primer:

```go
ch := make(chan int)

ch <- 42
```

Ako ne postoji druga goroutine koja prima:

```go
<-ch
```

send će blokirati.

Kod buffered channel-a ponašanje zavisi od kapaciteta.

```go
ch := make(chan int, 2)

ch <- 1
ch <- 2
```

Oba send-a mogu da se izvrše bez receiver-a zato što buffer ima prostora.

Treći send:

```go
ch <- 3
```

blokira dok se ne oslobodi mesto u buffer-u.

---

## 18. Blokiranje pri Receive operaciji

### Pitanje 18.1

**Kada receive operacija blokira goroutine?**

Receive:

```go
value := <-ch
```

blokira kada nema vrednosti koju može da primi.

Kod unbuffered channel-a:

```go
ch := make(chan int)

value := <-ch
```

receiver čeka dok neka druga goroutine ne izvrši:

```go
ch <- 42
```

Kod buffered channel-a receiver blokira kada je buffer prazan i channel još nije zatvoren.

Na primer:

```go
ch := make(chan int, 2)

value := <-ch
```

Ako niko nije poslao vrednost, goroutine čeka.

---

## 19. Blokiranje nije isto što i Deadlock

### Pitanje 19.1

**Da li je svako blokiranje goroutine-a deadlock?**

Ne.

Blokiranje je normalan deo concurrency modela.

Na primer:

```go
ch := make(chan int)

go func() {
	ch <- 42
}()

value := <-ch

fmt.Println(value)
```

Main goroutine može kratko čekati na receive, ali druga goroutine šalje vrednost.

Čim komunikacija postane moguća, blokiranje se završava.

Deadlock nastaje kada goroutine-i čekaju događaj koji se više nikada neće dogoditi.

Dakle:

```text
blokiranje
    │
    ├── očekivano i privremeno → normalno
    │
    └── niko ne može napraviti
        očekivani progress → deadlock
```

Ovo je jedna od najvažnijih razlika koju Junior developer mora da razume.

---

## 20. Najjednostavniji Deadlock

### Pitanje 20.1

**Šta će se desiti u sledećem programu?**

```go
func main() {
	ch := make(chan int)

	ch <- 42

	fmt.Println("done")
}
```

Program će deadlock-ovati.

Razlog je što je `ch` unbuffered channel, a `main` goroutine pokušava da pošalje vrednost bez postojanja receiver-a.

Tok:

```text
main
 │
 │ ch <- 42
 ▼
BLOCKED
```

Niko ne izvršava:

```go
<-ch
```

Zato nema mogućnosti za nastavak.

Go runtime detektuje situaciju u kojoj su sve goroutine blokirane i može prijaviti:

```text
fatal error: all goroutines are asleep - deadlock!
```

---

## 21. Kako rešiti ovaj Deadlock?

### Pitanje 21.1

**Kako možemo popraviti prethodni program?**

Jedna mogućnost je pokretanje receiver-a u drugoj goroutine-i:

```go
func main() {
	ch := make(chan int)

	go func() {
		fmt.Println(<-ch)
	}()

	ch <- 42
}
```

Sada postoji goroutine koja može da primi vrednost.

Druga mogućnost je korišćenje buffered channel-a:

```go
func main() {
	ch := make(chan int, 1)

	ch <- 42

	fmt.Println(<-ch)
}
```

Buffer omogućava prvom send-u da se izvrši bez istovremenog receiver-a.

Ali važno je razumeti:

> **Buffered channel nije univerzalno rešenje za deadlock.**

On samo menja uslove pod kojima send može da napreduje.

---

## 22. Deadlock sa Receive operacijom

### Pitanje 22.1

**Može li receive sam po sebi izazvati deadlock?**

Da.

Na primer:

```go
func main() {
	ch := make(chan int)

	value := <-ch

	fmt.Println(value)
}
```

Main goroutine čeka vrednost.

Ali niko nikada ne šalje:

```go
ch <- value
```

Zato nema progress-a.

Program završava u deadlock-u.

Ovo je suprotna varijanta prethodnog primera:

```text
send bez receiver-a
        ↓
    deadlock

receive bez sender-a
        ↓
    deadlock
```

---

## 23. Deadlock zbog Pogrešnog Redosleda Operacija

### Pitanje 23.1

**Zašto redosled send/receive operacija može biti važan?**

Kod unbuffered channel-a send i receive moraju da se "sretnu".

Na primer:

```go
ch := make(chan int)

ch <- 1
fmt.Println(<-ch)
```

ovo deadlock-uje jer se prvi send nikada ne završava.

Program nikada ne stigne do receive operacije.

Ako promenimo strukturu:

```go
ch := make(chan int)

go func() {
	ch <- 1
}()

fmt.Println(<-ch)
```

sada sender može da čeka dok receiver ne bude spreman.

Ovo je tipičan primer **rendezvous** semantike unbuffered channel-a.

---

## 24. Unbuffered Channel kao Rendezvous

### Pitanje 24.1

**Šta znači da unbuffered channel predstavlja rendezvous između goroutine-a?**

Kod unbuffered channel-a nema prostora za skladištenje vrednosti.

Send i receive moraju da se sinhronizuju.

Konceptualno:

```text
Sender                      Receiver
   │                           │
   │       ch <- value         │
   │──────────────────────────>│
   │                           │
   │       synchronization     │
   │<─────────────────────────>│
```

Send ne znači samo:

> "Stavi vrednost negde."

Već praktično:

> "Predaj ovu vrednost receiver-u."

Zato unbuffered channel istovremeno predstavlja:

* komunikaciju;
* sinhronizaciju;
* koordinaciju između goroutine-a.

---

## 25. Buffered Channel i Backpressure

### Pitanje 25.1

**Šta se događa kada buffered channel postane pun?**

Kada je buffer pun, sledeći send blokira.

Na primer:

```go
ch := make(chan int, 2)

ch <- 1
ch <- 2
```

Buffer je sada pun.

Sledeće:

```go
ch <- 3
```

blokira dok neki receiver ne uradi:

```go
<-ch
```

Ovo omogućava prirodan oblik **backpressure-a**.

```text
Producer
   │
   ▼
┌─────────────┐
│   buffer    │
│  [1] [2]    │
└──────┬──────┘
       │
       ▼
    Consumer

buffer pun
     │
     ▼
Producer BLOCKED
```

Producer tako ne može beskonačno brzo da proizvodi podatke ako consumer ne može da ih obrađuje dovoljno brzo.

---

## 26. Kapacitet Buffer-a nije "Brzina"

### Pitanje 26.1

**Da li povećavanje kapaciteta buffered channel-a rešava problem sporog consumer-a?**

Ne nužno.

Na primer:

```go
ch := make(chan Job, 100000)
```

može omogućiti producer-u da dugo nastavi da šalje bez blokiranja.

Ali ako producer proizvodi mnogo brže nego consumer što obrađuje:

```text
Producer
  │
  │ 1000 jobs/s
  ▼
Buffer
  │
  │ 100 jobs/s
  ▼
Consumer
```

buffer će se vremenom popuniti.

Veći buffer samo odlaže trenutak kada producer počinje da blokira.

Zato buffer treba posmatrati kao deo concurrency dizajna, a ne kao zamenu za kontrolu protoka.

---

## 27. Deadlock zbog Očekivanja Dve Strane

### Pitanje 27.1

**Kako dve goroutine-e mogu međusobno da čekaju jedna drugu?**

Na primer:

```go
ch1 := make(chan int)
ch2 := make(chan int)

go func() {
	ch1 <- 1
	<-ch2
}()

go func() {
	ch2 <- 2
	<-ch1
}()
```

Prva goroutine pokušava:

```text
ch1 <- 1
```

a druga:

```text
ch2 <- 2
```

Obe su na unbuffered channel-ima.

Nijedna ne može da završi svoj send dok druga ne izvrši odgovarajući receive.

Ali obe goroutine pokušavaju prvo da izvrše send.

Dobijamo:

```text
Goroutine A                  Goroutine B

ch1 <- 1                     ch2 <- 2
   │                             │
   ▼                             ▼
 WAIT                          WAIT
   │                             │
   └───────────┬─────────────────┘
               │
               ▼
            DEADLOCK
```

Ovo je klasičan **circular wait**.

---

## 28. Circular Wait

### Pitanje 28.1

**Šta je circular wait u concurrency programu?**

Circular wait nastaje kada svaki učesnik čeka resurs ili događaj koji zavisi od drugog učesnika u istom ciklusu.

Konceptualno:

```text
A čeka B
B čeka A
```

ili:

```text
A → čeka B
B → čeka C
C → čeka A
```

Kod channel-a ovo može nastati kada goroutine-i imaju pogrešno definisan redosled komunikacije.

Kod većih sistema circular wait može biti mnogo teže uočiti jer se lanac proteže kroz više komponenti.

---

## 29. Deadlock vs. Starvation

### Pitanje 29.1

**Koja je razlika između deadlock-a i starvation-a?**

Kod **deadlock-a** sistem više ne može da napravi progress zbog međusobnog čekanja.

Kod **starvation-a** neka goroutine ili task može teoretski da napreduje, ali u praksi dugo ili beskonačno ne dobija potrebnu priliku.

Pojednostavljeno:

```text
Deadlock:
A čeka B
B čeka A
→ niko ne napreduje

Starvation:
A stalno dobija resurs
B stalno čeka
→ B ne dobija fer priliku
```

Ovo su različiti concurrency problemi i zahtevaju različite dijagnostičke pristupe.

---

## 30. Deadlock vs. Goroutine Leak

### Pitanje 30.1

**Da li je goroutine leak isto što i deadlock?**

Ne.

Goroutine leak znači da goroutine ostane aktivna ili blokirana duže nego što je očekivano, bez mogućnosti ili mehanizma da se pravilno završi.

Na primer:

```go
func worker(ch <-chan int) {
	for {
		value := <-ch
		fmt.Println(value)
	}
}
```

Ako niko više ne šalje niti zatvara channel, worker može ostati blokiran zauvek.

To je potencijalni goroutine leak.

Deadlock je širi problem koji se odnosi na odsustvo mogućnosti napretka potrebnog za nastavak sistema.

Moguće je imati:

* deadlock bez dugoročnog leak-a;
* goroutine leak bez globalnog deadlock-a;
* i oba problema istovremeno.

---

## Junior mentalni model

Kod channel concurrency-ja treba uvek postaviti sledeća pitanja:

```text
1. Ko šalje?
2. Ko prima?
3. Da li je channel buffered?
4. Koliki je buffer?
5. Kada send može da blokira?
6. Kada receive može da blokira?
7. Ko zatvara channel?
8. Kada se channel zatvara?
9. Šta se dešava ako producer završi?
10. Šta se dešava ako consumer završi prvi?
11. Može li neka goroutine ostati blokirana?
12. Postoji li circular wait?
```

Ako na ova pitanja ne postoji jasan odgovor, concurrency dizajn verovatno još nije dovoljno precizan.

---

## Ključne lekcije ovog dela

* Send na unbuffered channel-u zahteva odgovarajući receive.
* Receive sa praznog otvorenog channel-a blokira.
* Send na pun buffered channel blokira.
* Blokiranje samo po sebi nije deadlock.
* Deadlock nastaje kada potrebni progress više nije moguć.
* Unbuffered channel predstavlja rendezvous između sender-a i receiver-a.
* Buffered channel može obezbediti određeni stepen decoupling-a.
* Buffer uvodi mogućnost backpressure-a kada se napuni.
* Veći buffer ne rešava fundamentalno sporog consumer-a.
* Circular wait je čest uzrok deadlock-a.
* Goroutine leak i deadlock nisu sinonimi.
* Concurrency dizajn mora eksplicitno definisati ownership, lifecycle i blocking ponašanje.

# Interview Questions — Junior

## Deo #4/7

## 31. Zatvaranje Channel-a

### Pitanje 31.1

**Šta znači zatvoriti channel u Go-u?**

Channel se zatvara pomoću ugrađene funkcije:

```go
close(ch)
```

Zatvaranje channel-a znači da sender signalizira:

> **"Više neće biti novih vrednosti koje će biti poslate kroz ovaj channel."**

Važno je razumeti da `close`:

* ne briše već poslate vrednosti;
* ne zaustavlja automatski receiver-e;
* ne prekida goroutine;
* ne znači da channel više ne može biti pročitan.

Ako channel sadrži buffered vrednosti, one i dalje mogu biti primljene nakon `close`.

Na primer:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20

close(ch)

fmt.Println(<-ch)
fmt.Println(<-ch)
```

Output:

```text
10
20
```

Channel je zatvoren, ali postojeće vrednosti su i dalje dostupne.

---

## 32. Ko treba da zatvori Channel?

### Pitanje 32.1

**Da li receiver treba da zatvara channel?**

Uobičajeno pravilo je:

> **Goroutine koja je odgovorna za slanje podataka treba da bude odgovorna i za zatvaranje channel-a.**

Na primer:

```go
func producer(out chan<- int) {
	defer close(out)

	for i := 0; i < 10; i++ {
		out <- i
	}
}
```

Receiver:

```go
func consumer(in <-chan int) {
	for value := range in {
		fmt.Println(value)
	}
}
```

Producer zna kada više neće slati podatke.

Consumer to najčešće ne zna.

Zbog toga producer ima informaciju potrebnu za donošenje odluke o zatvaranju.

---

## 33. Šta se događa kada se šalje na zatvoren Channel?

### Pitanje 33.1

**Šta se događa ako pokušamo da pošaljemo vrednost na zatvoren channel?**

Program će izazvati panic.

Na primer:

```go
ch := make(chan int)

close(ch)

ch <- 42
```

Rezultat je panic:

```text
panic: send on closed channel
```

Zato `close` nije operacija koju treba izvršavati proizvoljno.

Ako više goroutine-a može da zatvara isti channel, postoji rizik od:

```text
panic: close of closed channel
```

Zbog toga ownership nad channel lifecycle-om mora biti jasan.

---

## 34. Šta se događa pri Receive-u sa zatvorenog Channel-a?

### Pitanje 34.1

**Šta se događa kada pokušamo da primimo vrednost sa zatvorenog channel-a?**

Receive sa zatvorenog channel-a je validan.

Ako su sve prethodno poslate vrednosti već pročitane, receive vraća **zero value** odgovarajućeg tipa.

Na primer:

```go
ch := make(chan int)

close(ch)

value := <-ch

fmt.Println(value)
```

Output:

```text
0
```

Za `int` je zero value:

```go
0
```

Za `string`:

```go
""
```

Za pointer:

```go
nil
```

Zbog toga samo:

```go
value := <-ch
```

nije dovoljno da razlikujemo:

```text
channel je zatvoren
```

od:

```text
channel je otvoren, ali je primljena zero value vrednost
```

Za to postoji drugi oblik receive operacije.

---

## 35. Comma-ok idiom

### Pitanje 35.1

**Kako možemo proveriti da li je channel zatvoren?**

Koristimo oblik:

```go
value, ok := <-ch
```

`ok` predstavlja informaciju o tome da li je vrednost primljena iz otvorenog channel-a.

Na primer:

```go
value, ok := <-ch

if !ok {
	fmt.Println("channel je zatvoren")
	return
}

fmt.Println(value)
```

Ako je channel zatvoren i više nema vrednosti:

```text
value = zero value
ok = false
```

Ako je vrednost uspešno primljena:

```text
ok = true
```

Ovo je posebno važno kada je zero value legitimna poslovna vrednost.

Na primer, `0` može biti potpuno validan podatak, pa ne možemo koristiti:

```go
if value == 0 {
	// channel je zatvoren
}
```

jer to nije tačno.

---

## 36. Receive iz Buffered Closed Channel-a

### Pitanje 36.1

**Šta se događa ako zatvorimo buffered channel koji još ima podatke?**

Podaci se i dalje mogu primiti.

Na primer:

```go
ch := make(chan int, 3)

ch <- 10
ch <- 20
ch <- 30

close(ch)
```

Možemo uraditi:

```go
for value := range ch {
	fmt.Println(value)
}
```

Dobijamo:

```text
10
20
30
```

Tek kada se postojeći podaci potroše, `range` završava.

Konceptualno:

```text
Before close:

┌───────────────┐
│ 10 │ 20 │ 30  │
└───────────────┘

close(ch)

┌───────────────┐
│ 10 │ 20 │ 30  │
└───────────────┘
       ↓
receiver čita postojeće vrednosti
       ↓
buffer prazan
       ↓
channel closed
       ↓
range završava
```

`close` dakle ne znači:

> "Obriši sve."

Već:

> "Neće biti novih send operacija."

---

## 37. `range` preko Channel-a

### Pitanje 37.1

**Kako `range` radi sa channel-om?**

Možemo iterirati preko channel-a:

```go
for value := range ch {
	fmt.Println(value)
}
```

Ova petlja prima vrednosti sve dok channel ne bude zatvoren i dok se sve njegove vrednosti ne potroše.

Ekvivalentan obrazac je:

```go
for {
	value, ok := <-ch

	if !ok {
		break
	}

	fmt.Println(value)
}
```

Zato je:

```go
for value := range ch
```

idiomatski način za obradu stream-a vrednosti čiji je kraj eksplicitno označen zatvaranjem channel-a.

---

## 38. Šta ako `range` koristi otvoren Channel?

### Pitanje 38.1

**Šta se događa ako channel nikada nije zatvoren?**

Ako nema više vrednosti, `range` će čekati sledeću vrednost.

Na primer:

```go
ch := make(chan int)

go func() {
	ch <- 1
	ch <- 2
}()

for value := range ch {
	fmt.Println(value)
}
```

Consumer će dobiti:

```text
1
2
```

Ali nakon toga nema više vrednosti.

Pošto channel nije zatvoren:

```go
close(ch)
```

`range` ne zna da je producer završio.

Zato receiver ostaje blokiran.

Ovo može dovesti do situacije u kojoj program ne može normalno da završi.

---

## 39. Producer određuje kraj Stream-a

### Pitanje 39.1

**Zašto producer često treba da zatvori channel kada završi proizvodnju?**

Zato što producer zna kada više nema podataka.

Na primer:

```go
func producer(out chan<- int) {
	defer close(out)

	for i := 1; i <= 5; i++ {
		out <- i
	}
}
```

Consumer:

```go
func consumer(in <-chan int) {
	for value := range in {
		fmt.Println(value)
	}
}
```

Tok je:

```text
Producer
   │
   ├── 1
   ├── 2
   ├── 3
   ├── 4
   ├── 5
   │
   └── close
          │
          ▼
       Consumer
          │
          └── range završava
```

Ovo daje jasnu lifecycle granicu između producer-a i consumer-a.

---

## 40. Channel Close kao Signal

### Pitanje 40.1

**Da li `close` služi samo za sprečavanje novih send operacija?**

Ne.

`close` je istovremeno i **signal**.

Može se koristiti za označavanje:

> "Događaj je završen."

Na primer:

```go
done := make(chan struct{})

go func() {
	doWork()
	close(done)
}()

<-done

fmt.Println("work completed")
```

Ovde ne šaljemo nikakvu konkretnu vrednost.

Samo zatvaranje channel-a predstavlja signal da je posao završen.

Ovaj obrazac je koristan kada više goroutine-a treba da bude obavešteno o završetku određenog događaja.

---

## 41. Zašto se često koristi `chan struct{}`?

### Pitanje 41.1

**Zašto se za signalizaciju često koristi `chan struct{}`?**

Zato što nam nije potrebna stvarna vrednost.

Koristimo:

```go
done := make(chan struct{})
```

a zatim:

```go
close(done)
```

Receiver može da čeka:

```go
<-done
```

`struct{}` nema podatke koje treba skladištiti.

Semantički je jasno da channel predstavlja signal, a ne stream poslovnih podataka.

Primer:

```go
func worker(done chan<- struct{}) {
	// posao
	close(done)
}
```

Consumer:

```go
<-done

fmt.Println("worker finished")
```

Za ozbiljnije cancellation obrasce koristiće se `context.Context`, ali je `close` nad signalnim channel-om važan osnovni concurrency obrazac.

---

## 42. Može li više goroutine-a primiti signal nakon `close`?

### Pitanje 42.1

**Šta se događa ako više goroutine-a čeka na channel koji se zatvori?**

Sve goroutine koje čekaju receive mogu biti probuđene kada channel bude zatvoren.

Na primer:

```go
done := make(chan struct{})

for i := 0; i < 3; i++ {
	go func(id int) {
		<-done
		fmt.Println("worker", id, "finished")
	}(i)
}

close(done)
```

Zatvaranje channel-a predstavlja broadcast signal.

To je fundamentalno drugačije od slanja jedne vrednosti:

```go
done <- struct{}{}
```

Jedan send ne znači automatski da će sve čekajuće goroutine dobiti isti signal.

`close(done)` može signalizirati završetak svim receiver-ima koji čekaju na taj channel.

---

## 43. `close` nije Cancellation mehanizam sam po sebi

### Pitanje 43.1

**Da li zatvaranje channel-a automatski prekida goroutine?**

Ne.

Na primer:

```go
func worker(done <-chan struct{}) {
	for {
		select {
		case <-done:
			return
		default:
			// work
		}
	}
}
```

Ovde goroutine eksplicitno proverava signal.

Samo postojanje channel-a ne znači da će goroutine automatski biti zaustavljena.

Zato cancellation zahteva:

1. signal;
2. kod koji taj signal posmatra;
3. izlaznu putanju iz goroutine-e.

U većim sistemima ovo se često rešava pomoću `context.Context`.

---

## 44. Pravilo Ownership-a

### Pitanje 44.1

**Koje je praktično pravilo za ownership channel-a?**

Najbezbednije pravilo je:

> **Onaj ko proizvodi i kontroliše životni ciklus podataka treba da kontroliše i zatvaranje channel-a.**

Ako imamo:

```go
func producer(out chan<- int)
```

producer ima send-only pristup:

```go
out <- value
```

i može zatvoriti channel:

```go
close(out)
```

Consumer ima:

```go
func consumer(in <-chan int)
```

i može samo da prima:

```go
value := <-in
```

Ovakva API struktura dokumentuje nameru i smanjuje mogućnost pogrešne upotrebe.

---

## 45. Junior mentalni model: Channel Lifecycle

Kod svakog channel-a treba moći jasno da odgovorimo na sledeća pitanja:

```text
1. Ko ga kreira?
2. Ko šalje?
3. Ko prima?
4. Ko ga zatvara?
5. Kada se zatvara?
6. Šta znači close u ovom konkretnom API-ju?
7. Da li channel ima buffer?
8. Šta se događa kada je buffer pun?
9. Šta se događa kada je channel prazan?
10. Šta se događa nakon close?
11. Koji goroutine-i zavise od njegovog zatvaranja?
12. Može li receiver ostati zauvek u range petlji?
13. Može li neko poslati nakon close?
14. Može li više goroutine-a pokušati da zatvori isti channel?
```

Ako odgovori na ova pitanja nisu jasni, lifecycle channel-a nije dovoljno dobro definisan.

---

## Ključne lekcije ovog dela

* `close(ch)` označava da više neće biti novih send operacija.
* Zatvaranje channel-a ne briše već poslate vrednosti.
* Buffered channel može nastaviti da vraća vrednosti i nakon `close`.
* Send na zatvoren channel izaziva panic.
* Receive sa zatvorenog channel-a je validan.
* `value, ok := <-ch` omogućava detekciju zatvorenog channel-a.
* `range` završava kada je channel zatvoren i kada su sve vrednosti pročitane.
* `range` nad otvorenim channel-om može ostati blokiran zauvek.
* Producer obično treba da zatvori channel kada završi proizvodnju.
* `close` može služiti kao broadcast signal.
* `chan struct{}` je čest obrazac za signalizaciju bez podataka.
* Zatvaranje channel-a samo po sebi ne prekida goroutine.
* Channel ownership treba jasno definisati.
* Channel lifecycle je sastavni deo concurrency API dizajna.

# Interview Questions — Junior

## Deo #5/7

## 46. `select` Statement

### Pitanje 46.1

**Čemu služi `select` u Go-u?**

`select` omogućava goroutine-i da čeka na više channel operacija istovremeno.

Najjednostavniji primer:

```go
select {
case value := <-ch1:
	fmt.Println("primljeno sa ch1:", value)

case value := <-ch2:
	fmt.Println("primljeno sa ch2:", value)
}
```

Goroutine čeka dok najmanje jedna od navedenih komunikacionih operacija ne postane moguća.

Možemo ga posmatrati kao concurrency ekvivalent konceptu:

> "Čekaj bilo koji od ovih događaja."

Za razliku od običnog:

```go
value := <-ch
```

koji čeka samo jedan channel, `select` omogućava koordinaciju više mogućih događaja.

---

## 47. Kako `select` bira Case?

### Pitanje 47.1

**Šta se događa ako je više `case` grana spremno istovremeno?**

Ako je više channel operacija spremno, Go runtime bira jednu od spremnih opcija.

Na primer:

```go
select {
case <-ch1:
	fmt.Println("ch1")

case <-ch2:
	fmt.Println("ch2")
}
```

Ako su oba channel-a spremna za receive, ne treba računati na određeni redosled.

Izbor nije nešto što aplikacija treba da koristi kao poslovnu logiku.

Zato ne treba pisati kod koji pretpostavlja:

```text
ch1 će uvek biti izabran pre ch2
```

Ako su oba slučaja spremna, izbor je nedeterministički iz perspektive programa.

---

## 48. `select` bez `default`

### Pitanje 48.1

**Šta se događa kada nijedan `case` nije spreman?**

Ako `select` nema `default`, goroutine blokira dok jedan od `case`-ova ne postane spreman.

Na primer:

```go
select {
case value := <-ch:
	fmt.Println(value)

case <-done:
	fmt.Println("finished")
}
```

Ako:

* `ch` nema vrednost;
* `done` nije zatvoren;

goroutine čeka.

To je normalno ponašanje.

Konceptualno:

```text
                 ┌── ch spreman ──→ case 1
select ──────────┤
                 └── done spreman → case 2

nijedan spreman
       │
       ▼
    BLOCK
```

---

## 49. `select` sa `default`

### Pitanje 49.1

**Čemu služi `default` unutar `select` statement-a?**

`default` omogućava **non-blocking** pokušaj channel operacije.

Na primer:

```go
select {
case value := <-ch:
	fmt.Println(value)

default:
	fmt.Println("nema podataka")
}
```

Ako `ch` trenutno nema vrednost, `default` se izvršava odmah.

Dakle:

```text
case spreman
    ↓
izvrši komunikaciju

case nije spreman
    ↓
default
    ↓
nastavi odmah
```

Ovo je fundamentalno drugačije od `select`-a bez `default`, koji čeka.

---

## 50. Non-blocking Receive

### Pitanje 50.1

**Kako možemo pokušati da primimo vrednost bez blokiranja?**

Koristimo:

```go
select {
case value := <-ch:
	fmt.Println("received:", value)

default:
	fmt.Println("nothing available")
}
```

Ako je vrednost dostupna, prima se.

Ako nije, program odmah nastavlja.

Ovaj obrazac može biti koristan kada je čekanje neprihvatljivo.

Ali treba biti oprezan sa njegovom upotrebom.

Ako se ovakav kod izvršava u beskonačnoj petlji:

```go
for {
	select {
	case value := <-ch:
		process(value)

	default:
	}
}
```

možemo dobiti **busy loop** koji nepotrebno troši CPU.

---

## 51. Busy Loop

### Pitanje 51.1

**Šta je busy loop i zašto je problematičan?**

Busy loop nastaje kada goroutine neprekidno proverava stanje bez blokiranja ili čekanja.

Na primer:

```go
for {
	select {
	case value := <-ch:
		process(value)

	default:
	}
}
```

Ako nema podataka, goroutine ne blokira.

Ona neprekidno izvršava:

```text
select
select
select
select
select
...
```

To može dovesti do visokog CPU utilization-a.

Bolji pristup zavisi od konkretnog problema.

Na primer, ako je čekanje legitimno, možemo koristiti blocking `select`:

```go
select {
case value := <-ch:
	process(value)
}
```

ili kombinovati `select` sa `time.After`, `time.NewTimer` ili drugim signalom.

---

## 52. Timeout pomoću `select`

### Pitanje 52.1

**Kako možemo implementirati timeout za channel operaciju?**

Možemo kombinovati channel operaciju sa timer channel-om.

Jedan jednostavan obrazac je:

```go
select {
case value := <-ch:
	fmt.Println("received:", value)

case <-time.After(time.Second):
	fmt.Println("timeout")
}
```

Ako vrednost stigne pre isteka jedne sekunde, prvi case se izvršava.

Ako ne stigne, timer case postaje spreman.

Konceptualno:

```text
                 ┌── value arrives ──→ success
select ──────────┤
                 └── timeout ─────────→ timeout
```

Ovo je važan obrazac za mrežne pozive, RPC operacije, worker-e i druge concurrency scenarije.

---

## 53. `time.After` i ponovljena upotreba

### Pitanje 53.1

**Da li je `time.After` uvek najbolji izbor za timeout?**

Ne.

Za jednostavan, jednokratni timeout:

```go
select {
case <-ch:
case <-time.After(time.Second):
}
```

može biti sasvim odgovarajući.

Ali u dugotrajnim petljama ili visokofrekventnom kodu treba razumeti lifecycle timer-a i razmotriti eksplicitni:

```go
timer := time.NewTimer(...)
```

kada je potrebno preciznije upravljanje timer resursom.

Na primer:

```go
timer := time.NewTimer(time.Second)
defer timer.Stop()

select {
case value := <-ch:
	fmt.Println(value)

case <-timer.C:
	fmt.Println("timeout")
}
```

Ovaj obrazac daje eksplicitniju kontrolu nad timer-om.

---

## 54. Timeout ne znači Cancellation

### Pitanje 54.1

**Da li timeout automatski prekida goroutine koja radi posao?**

Ne.

Na primer:

```go
select {
case result := <-resultCh:
	fmt.Println(result)

case <-time.After(time.Second):
	fmt.Println("timeout")
}
```

Ako timeout nastupi, samo goroutine koja izvršava `select` nastavlja drugim putem.

Worker koji možda i dalje radi:

```go
go doWork()
```

ne mora biti zaustavljen.

To može dovesti do goroutine leak-a ako worker nema sopstveni mehanizam za prekid.

Zato treba razlikovati:

```text
timeout
   │
   └── prestani da čekaš

cancellation
   │
   └── signaliziraj worker-u da prekine posao
```

U naprednijim programima timeout i cancellation često se kombinuju preko `context.Context`.

---

## 55. `select` sa `done` Channel-om

### Pitanje 55.1

**Kako možemo omogućiti worker-u da reaguje na signal za završetak?**

Tipičan obrazac:

```go
func worker(done <-chan struct{}, jobs <-chan Job) {
	for {
		select {
		case job := <-jobs:
			process(job)

		case <-done:
			return
		}
	}
}
```

Worker sada može da čeka:

* novi posao;
* signal za završetak.

Ako se izvrši:

```go
close(done)
```

`case <-done` postaje spreman.

Worker može da završi svoju goroutine.

Ovo je osnovni model graceful cancellation-a.

---

## 56. `select` nad zatvorenim Channel-om

### Pitanje 56.1

**Šta se događa ako je jedan od channel-a u `select`-u zatvoren?**

Receive iz zatvorenog channel-a je odmah spreman.

Na primer:

```go
ch := make(chan int)
close(ch)

select {
case value := <-ch:
	fmt.Println(value)

default:
	fmt.Println("default")
}
```

Prvi case je spreman jer je receive sa zatvorenog channel-a validan.

Dobićemo zero value i `ok == false` ako koristimo comma-ok oblik:

```go
value, ok := <-ch
```

Zbog toga kod `select` petlji treba voditi računa o zatvorenim channel-ima.

Ako channel ostane u `select`-u nakon što je zatvoren, njegov receive case može biti spreman praktično odmah.

---

## 57. Zašto zatvoreni Channel može izazvati problem u `select` petlji?

### Pitanje 57.1

Posmatrajmo:

```go
for {
	select {
	case value := <-ch:
		process(value)

	case <-done:
		return
	}
}
```

Ako se `ch` zatvori, receive case postaje stalno spreman.

Ako ne proverimo `ok`:

```go
value := <-ch
```

dobićemo zero value nakon što se postojeće vrednosti potroše.

Petlja može nastaviti da obrađuje zero value vrednosti.

Ispravniji obrazac je:

```go
for {
	select {
	case value, ok := <-ch:
		if !ok {
			return
		}

		process(value)

	case <-done:
		return
	}
}
```

Time eksplicitno obrađujemo lifecycle channel-a.

---

## 58. `select` kao Concurrency Multiplexer

### Pitanje 58.1

**Zašto se `select` često opisuje kao mehanizam za multipleksiranje channel događaja?**

Zato što jedna goroutine može koordinisati više nezavisnih izvora događaja.

Na primer:

```go
select {
case job := <-jobs:
	process(job)

case <-ctx.Done():
	return

case <-ticker.C:
	emitMetrics()
}
```

Jedna goroutine sada reaguje na:

1. poslovni posao;
2. cancellation;
3. periodični timer događaj.

`select` time postaje centralna tačka koordinacije različitih concurrency događaja.

---

## 59. Više `select` grana ne znači paralelno izvršavanje

### Pitanje 59.1

**Da li se svi `case` blokovi izvršavaju ako su spremni?**

Ne.

Jedan `select` statement bira jednu spremnu granu.

Na primer:

```go
select {
case <-ch1:
	handle1()

case <-ch2:
	handle2()

case <-ch3:
	handle3()
}
```

Ako su sva tri channel-a spremna, izvršava se samo jedan `case`.

Nakon završetka `select` statement-a, program nastavlja dalje.

Ako želimo ponavljanje:

```go
for {
	select {
	case <-ch1:
		handle1()

	case <-ch2:
		handle2()

	case <-ch3:
		handle3()
	}
}
```

onda se `select` ponavlja kroz petlju.

---

## 60. `select` i Prioritet

### Pitanje 60.1

**Možemo li jednostavnim redosledom `case` grana dati prioritet jednom channel-u?**

Ne treba računati na to.

Na primer:

```go
select {
case <-urgent:
	handleUrgent()

case <-normal:
	handleNormal()
}
```

Ne znači da će `urgent` uvek biti izabran ako su oba spremna.

Ako aplikacija zahteva eksplicitnu prioritizaciju, potrebno je dizajnirati drugačiju logiku.

Na primer, može se prvo proveriti prioritetni channel non-blocking pristupom, a zatim izvršiti blocking select, ali takav obrazac treba koristiti samo kada stvarno postoji poslovna potreba.

---

## 61. Channel komunikacija ili `select`?

### Pitanje 61.1

**Kada je dovoljan običan receive, a kada je potreban `select`?**

Ako goroutine čeka samo jedan događaj:

```go
value := <-ch
```

običan receive je jednostavniji.

Ako mora da reaguje na više mogućih događaja:

```go
select {
case value := <-ch:
	process(value)

case <-ctx.Done():
	return
}
```

`select` je prirodniji izbor.

Praktično pravilo:

```text
jedan događaj
    ↓
receive

više mogućih događaja
    ↓
select
```

---

## 62. Junior mentalni model za `select`

Pre upotrebe `select` treba znati:

```text
1. Koji channel-i učestvuju?
2. Koji case-ovi su send, a koji receive?
3. Da li neki case može ostati zauvek blokiran?
4. Da li postoji default?
5. Ako postoji default, da li može nastati busy loop?
6. Šta se dešava kada se channel zatvori?
7. Da li postoji timeout?
8. Da li timeout samo prekida čekanje ili zaista otkazuje posao?
9. Koji signal završava worker?
10. Da li postoji mogućnost goroutine leak-a?
```

---

## Ključne lekcije ovog dela

* `select` čeka na više channel operacija.
* Ako je više operacija spremno, bira se jedna spremna grana.
* `select` bez `default` može blokirati.
* `default` omogućava non-blocking ponašanje.
* `default` u beskonačnoj petlji može izazvati busy loop.
* `select` se često koristi za timeout.
* Timeout i cancellation nisu ista stvar.
* `done` channel može služiti kao signal za završetak.
* Receive sa zatvorenog channel-a je odmah spreman.
* Zatvoren channel može ostati stalno spreman u `select` petlji.
* Zato treba proveravati `ok` kada channel lifecycle zahteva detekciju zatvaranja.
* `select` izvršava jednu izabranu granu, ne sve spremne grane.
* Redosled `case` grana ne treba koristiti kao implicitni prioritet.
* `select` je osnovni alat za kompoziciju concurrency događaja.

````markdown
# Interview Questions — Junior

## 6. Napredniji modeli komunikacije kanalima

U prethodnim pitanjima fokus je bio na osnovnoj upotrebi kanala, razlikama između buffered i unbuffered kanala, channel directions, kao i pravilima vezanim za `range` i `close`.

Na Junior nivou potrebno je otići korak dalje: razumeti **šta se zapravo dešava kada goroutine šalje ili prima podatke preko kanala i kada operacija može da blokira**.

---

### Pitanje 6.1 — Kada send na kanal blokira?

Send operacija:

```go
ch <- value
````

može da blokira trenutnu goroutine dok se ne ispuni uslov za slanje.

Kod **unbuffered** kanala:

```go
ch := make(chan int)

ch <- 42
```

send blokira dok druga goroutine ne izvrši receive:

```go
value := <-ch
```

Primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch

fmt.Println(value)
```

Ovde sender i receiver moraju da se "sretnu" u trenutku komunikacije.

Kod **buffered** kanala:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20
```

send može da se izvrši bez trenutnog receiver-a sve dok buffer nije pun.

Treći send:

```go
ch <- 30
```

blokiraće dok se iz kanala ne pročita najmanje jedna vrednost.

**Ključna ideja:**

> Send na kanalu blokira kada kanal u tom trenutku ne može da prihvati novu vrednost.

---

### Pitanje 6.2 — Kada receive sa kanala blokira?

Receive:

```go
value := <-ch
```

blokira ako nema dostupne vrednosti koju može da primi.

Kod unbuffered kanala:

```go
ch := make(chan int)

value := <-ch
```

receive čeka sender-a.

Kod buffered kanala:

```go
ch := make(chan int, 2)

ch <- 10

value := <-ch
```

receive se izvršava odmah jer buffer sadrži vrednost.

Ako je buffer prazan:

```go
ch := make(chan int, 2)

value := <-ch
```

receive blokira dok neka goroutine ne pošalje vrednost.

---

### Pitanje 6.3 — Šta se dešava kada se zatvoren kanal čita?

Ako je kanal zatvoren:

```go
close(ch)
```

receive ne blokira nakon što su sve prethodno poslate vrednosti pročitane.

Na primer:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20

close(ch)

fmt.Println(<-ch)
fmt.Println(<-ch)
fmt.Println(<-ch)
```

Ispis će biti:

```text
10
20
0
```

Treći receive dobija **zero value** za tip kanala.

Zbog toga se često koristi forma:

```go
value, ok := <-ch
```

gde:

* `ok == true` znači da je vrednost regularno primljena;
* `ok == false` znači da je kanal zatvoren i da više nema vrednosti.

Primer:

```go
value, ok := <-ch

if !ok {
    fmt.Println("channel closed")
    return
}

fmt.Println(value)
```

---

### Pitanje 6.4 — Zašto je `range` nad kanalom koristan?

Umesto ručnog proveravanja:

```go
for {
    value, ok := <-ch

    if !ok {
        break
    }

    fmt.Println(value)
}
```

možemo koristiti:

```go
for value := range ch {
    fmt.Println(value)
}
```

`range` nastavlja da prima vrednosti sve dok kanal ne bude zatvoren i dok se ne potroše sve vrednosti koje su eventualno ostale u bufferu.

Zbog toga je veoma čest obrazac:

```go
func worker(ch <-chan int) {
    for value := range ch {
        process(value)
    }
}
```

Producer je odgovoran za završetak toka:

```go
close(ch)
```

a consumer može jednostavno da radi:

```go
for value := range ch {
    process(value)
}
```

---

### Pitanje 6.5 — Ko bi trebalo da zatvori kanal?

Praktično pravilo je:

> Kanal treba da zatvori goroutine koja zna da više neće biti slanja na taj kanal.

Najčešće je to **sender/producer**, a ne receiver.

Primer:

```go
func producer() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 5; i++ {
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

Producer zna kada je proizvodnja završena, pa zato on zatvara kanal.

Receiver uglavnom nema dovoljno informacija da zna da li će neki drugi producer kasnije poslati novu vrednost.

---

## 7. Blocking kao deo dizajna konkurentnog programa

Blokiranje samo po sebi nije greška.

Ovo je veoma važna razlika.

Na primer:

```go
value := <-ch
```

može legitimno da čeka jer program očekuje da će druga goroutine poslati vrednost.

Problem nastaje kada **blokiranje nije deo očekivanog protokola**.

Na primer:

```go
ch := make(chan int)

ch <- 10

fmt.Println("done")
```

Ako nema druge goroutine koja prima vrednost, program može završiti u deadlock-u.

Kod `main` goroutine, runtime može detektovati situaciju u kojoj nijedna goroutine ne može da nastavi izvršavanje.

---

## 8. Blocking i buffered kanali

Buffered kanal može smanjiti direktnu zavisnost između sender-a i receiver-a.

Na primer:

```go
ch := make(chan int, 3)

ch <- 1
ch <- 2
ch <- 3
```

sva tri send-a mogu da se izvrše bez receiver-a.

Ali:

```go
ch <- 4
```

blokira jer je buffer pun.

To znači da buffered kanal **ne uklanja blokiranje**.

On samo omogućava određenu količinu asinhronog rada.

---

## 9. Tipično pitanje na intervjuu

### "Da li buffered channel znači da send nikada neće blokirati?"

**Ne.**

Send na buffered channel-u blokira kada je buffer pun i nema receiver-a koji bi oslobodio prostor.

Na primer:

```go
ch := make(chan int, 2)

ch <- 1
ch <- 2

ch <- 3 // blocks
```

Kapacitet kanala je:

```text
2
```

Nakon dva send-a buffer je pun.

Treći send mora da čeka.

---

## 10. Važna mentalna slika

Kod konkurentnog Go programa treba razmišljati o kanalu kao o delu **sinhronizacionog protokola**, a ne samo kao o strukturi za skladištenje podataka.

Na primer:

```text
Producer
   |
   | send
   v
Channel
   |
   | receive
   v
Consumer
```

Kod unbuffered kanala:

```text
Producer ──────┐
               ├── rendezvous ──> Consumer
               └─────────────────
```

Kod buffered kanala:

```text
Producer
   |
   v
┌───────────────┐
│ Channel       │
│ [1] [2] [3]   │
└───────────────┘
   |
   v
Consumer
```

Buffer predstavlja ograničenu količinu prostora između proizvođača i potrošača.

Kada se prostor popuni, producer mora da čeka.

Kada se prostor isprazni, consumer može da mora da čeka.

---

## Šta Junior treba da razume

Nakon ovog nivoa, kandidat treba da bude sposoban da objasni:

* kada send blokira;
* kada receive blokira;
* razliku između blokiranja na unbuffered i buffered kanalima;
* šta se dešava pri receive-u sa zatvorenog kanala;
* kako `value, ok := <-ch` funkcioniše;
* zašto se koristi `range` nad kanalom;
* ko bi trebalo da zatvara kanal;
* zašto buffered channel ne znači "non-blocking";
* kako kapacitet buffera utiče na producer/consumer odnos;
* zašto je blocking deo concurrency dizajna;
* kako pogrešno dizajniran channel protocol može dovesti do deadlock-a.

Ovo je osnova za sledeći nivo razumevanja: **lifecycle goroutine-a, ownership kanala i sprečavanje goroutine leak-ova**.