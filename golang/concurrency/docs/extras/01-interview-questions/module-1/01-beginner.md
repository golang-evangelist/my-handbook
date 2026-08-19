# Interview Questions — Beginner

> **Fajl:** `extras/01-interview-questions/module-1/01-beginner.md`
>
> **Nivo:** Beginner
>
> **Oblast:** #1 — Concurrency Fundamentals

---

## Sadržaj

1. [Šta je goroutine?](#pitanje-01--šta-je-goroutine)
2. [Po čemu se goroutine razlikuje od OS thread-a?](#pitanje-02--po-čemu-se-goroutine-razlikuje-od-os-thread-a)
3. [Kako se pokreće goroutine?](#pitanje-03--kako-se-pokreće-goroutine)
4. [Da li `go` garantuje da će goroutine završiti pre `main()` funkcije?](#pitanje-04--da-li-go-garantuje-da-će-goroutine-završiti-pre-main-funkcije)
5. [Šta je channel i čemu služi?](#pitanje-05--šta-je-channel-i-čemu-služi)
6. [Šta je goroutine i po čemu se razlikuje od klasičnog OS thread-a?](#pitanje-06--šta-je-goroutine-i-po-čemu-se-razlikuje-od-klasičnog-os-thread-a)
7. [Šta se dešava kada napišemo `go process()`?](#pitanje-07--šta-se-dešava-kada-napišemo-go-process)
8. [Da li je izvršavanje goroutines determinističko?](#pitanje-08--da-li-je-izvršavanje-goroutines-determinističko)
9. [Šta je channel u Go-u?](#pitanje-09--šta-je-channel-u-go-u)
10. [Zašto se channels često opisuju kao mehanizam za komunikaciju između goroutines?](#pitanje-10--zašto-se-channels-često-opisuju-kao-mehanizam-za-komunikaciju-između-goroutines)
11. [Šta znači izraz `ch <- value`?](#pitanje-11--šta-znači-izraz-ch---value)
12. [Šta znači izraz `value := <-ch`?](#pitanje-12--šta-znači-izraz-value---ch)
13. [Kako channel može da se koristi za sinhronizaciju?](#pitanje-13--kako-channel-može-da-se-koristi-za-sinhronizaciju)
14. [Šta znači da je channel operacija blokirajuća?](#pitanje-14--šta-znači-da-je-channel-operacija-blokirajuća)
15. [Zašto se u Go-u ne preporučuje posmatranje goroutines samo kao „jeftinih thread-ova"?](#pitanje-15--zašto-se-u-go-u-ne-preporučuje-posmatranje-goroutines-samo-kao-jeftinih-thread-ova)
16. [Da li pokretanje goroutine garantuje da će se njeno izvršavanje desiti pre nego što se trenutna funkcija nastavi?](#pitanje-16--da-li-pokretanje-goroutine-garantuje-da-će-se-njeno-izvršavanje-desiti-pre-nego-što-se-trenutna-funkcija-nastavi)
17. [Šta se dešava sa goroutines kada `main` funkcija završi?](#pitanje-17--šta-se-dešava-sa-goroutines-kada-main-funkcija-završi)
18. [Da li dve goroutines automatski izvršavaju kod paralelno?](#pitanje-18--da-li-dve-goroutines-automatski-izvršavaju-kod-paralelno)
19. [Da li redosled izvršavanja goroutines može da se predvidi?](#pitanje-19--da-li-redosled-izvršavanja-goroutines-može-da-se-predvidi)
20. [Šta znači da je channel blokirajući mehanizam?](#pitanje-20--šta-znači-da-je-channel-blokirajući-mehanizam)
21. [Šta se dešava kada šaljemo vrednost na unbuffered channel, a niko trenutno ne prima tu vrednost?](#pitanje-21--šta-se-dešava-kada-šaljemo-vrednost-na-unbuffered-channel-a-niko-trenutno-ne-prima-tu-vrednost)
22. [Šta se dešava kada primamo vrednost sa unbuffered channel-a, a niko ne šalje vrednost?](#pitanje-22--šta-se-dešava-kada-primamo-vrednost-sa-unbuffered-channel-a-a-niko-ne-šalje-vrednost)
23. [Zašto se channel često koristi kao mehanizam koordinacije, a ne samo za prenos podataka?](#pitanje-23--zašto-se-channel-često-koristi-kao-mehanizam-koordinacije-a-ne-samo-za-prenos-podataka)
24. [Da li je dobro koristiti `time.Sleep` da bismo „sačekali" drugu goroutine?](#pitanje-24--da-li-je-dobro-koristiti-timesleep-da-bismo-sačekali-drugu-goroutine)
25. [Koja je osnovna razlika između „čekanja određeno vreme" i „čekanja događaja"?](#pitanje-25--koja-je-osnovna-razlika-između-čekanja-određeno-vreme-i-čekanja-događaja)
26. [Šta se dešava kada goroutine pošalje vrednost na unbuffered channel?](#pitanje-26--šta-se-dešava-kada-goroutine-pošalje-vrednost-na-unbuffered-channel)
27. [Šta se dešava kada goroutine primi vrednost sa unbuffered channel-a, ali trenutno nema pošiljaoca?](#pitanje-27--šta-se-dešava-kada-goroutine-primi-vrednost-sa-unbuffered-channel-a-ali-trenutno-nema-pošiljaoca)
28. [Koja je osnovna razlika između buffered i unbuffered channel-a?](#pitanje-28--koja-je-osnovna-razlika-između-buffered-i-unbuffered-channel-a)
29. [Da li buffered channel znači da slanje nikada neće blokirati?](#pitanje-29--da-li-buffered-channel-znači-da-slanje-nikada-neće-blokirati)
30. [Šta predstavljaju channel directions u Go-u?](#pitanje-30--šta-predstavljaju-channel-directions-u-go-u)
31. [Zašto su channel directions korisni?](#pitanje-31--zašto-su-channel-directions-korisni)
32. [Kako se koristi `range` za čitanje vrednosti iz channel-a?](#pitanje-32--kako-se-koristi-range-za-čitanje-vrednosti-iz-channel-a)
33. [Zašto je `close` važan kada koristimo `range` nad channel-om?](#pitanje-33--zašto-je-close-važan-kada-koristimo-range-nad-channel-om)
34. [Čemu služi `select` u Go concurrency modelu?](#pitanje-34--čemu-služi-select-u-go-concurrency-modelu)
35. [Koja je osnovna razlika između `select` i običnog channel receive-a?](#pitanje-35--koja-je-osnovna-razlika-između-select-i-običnog-channel-receive-a)
36. [Da li `go` ključna reč garantuje da će goroutine završiti posao pre nego što se program završi?](#pitanje-36--da-li-go-ključna-reč-garantuje-da-će-goroutine-završiti-posao-pre-nego-što-se-program-završi)
37. [Da li je goroutine isto što i OS thread?](#pitanje-37--da-li-je-goroutine-isto-što-i-os-thread)
38. [Da li zbog toga treba pokretati goroutine za svaku sitnicu?](#pitanje-38--da-li-zbog-toga-treba-pokretati-goroutine-za-svaku-sitnicu)
39. [Šta je data race?](#pitanje-39--šta-je-data-race)
40. [Šta je deadlock?](#pitanje-40--šta-je-deadlock)
41. [Šta je goroutine leak?](#pitanje-41--šta-je-goroutine-leak)
42. [Kako možemo omogućiti goroutini da zna kada treba da prekine rad?](#pitanje-42--kako-možemo-omogućiti-goroutini-da-zna-kada-treba-da-prekine-rad)
43. [Ko treba da zatvori channel?](#pitanje-43--ko-treba-da-zatvori-channel)
44. [Da li receive sa zatvorenog channel-a izaziva panic?](#pitanje-44--da-li-receive-sa-zatvorenog-channel-a-izaziva-panic)

---

## Pitanje 01 — Šta je goroutine?

### Odgovor

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

Ovde postoje najmanje dve goroutines:

* glavna goroutine u kojoj se izvršava `main()`;
* goroutine pokrenuta naredbom `go printMessage()`.

Važno je razumeti da **goroutine nije isto što i thread**.

Go runtime može imati veliki broj goroutines koje se izvršavaju preko relativno malog broja OS thread-ova:

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

### Šta interviewer očekuje?

* Šta je goroutine.
* Kako se pokreće.
* Da je goroutine konkurentna jedinica izvršavanja.
* Da njome upravlja Go runtime.
* Da goroutine nije isto što i OS thread.

### Česta greška

Nije precizno reći:

> "Goroutine je Go thread."

Preciznije je:

> "Goroutine je lagana jedinica izvršavanja kojom upravlja Go runtime scheduler i koja se izvršava na OS thread-ovima."

---

## Pitanje 02 — Po čemu se goroutine razlikuje od OS thread-a?

### Odgovor

Najvažnija razlika je u tome **ko upravlja izvršavanjem** i **koliko je jedinica izvršavanja skupa**.

OS thread kreira i kontroliše operativni sistem. Goroutine kreira Go runtime.

| OS Thread | Goroutine |
| --- | --- |
| Upravlja OS | Upravlja Go runtime |
| Teži za kreiranje | Veoma lagana |
| Veći overhead | Mali overhead |
| Scheduler je deo OS-a | Go ima sopstveni scheduler |
| Relativno mali broj thread-ova | Može postojati veoma veliki broj goroutines |

Jedna od važnih karakteristika goroutines je njihov stack. Goroutine počinje sa relativno malim stack-om koji može da raste kada je potrebno. Zbog toga je moguće imati veliki broj goroutines bez potrebe da se za svaku kreira veliki OS thread stack.

Međutim, ne treba iz ovoga zaključiti:

> "Goroutines su besplatne."

Nisu. Svaka goroutine i dalje koristi memoriju i runtime resurse. Ako aplikacija nekontrolisano kreira goroutines koje nikada ne završavaju, može doći do **goroutine leak-a** i ozbiljne potrošnje resursa.

### Šta interviewer očekuje?

* Ko upravlja OS thread-om, a ko goroutine.
* Da goroutine ima dinamički stack koji raste prema potrebi.
* Da je goroutine jeftinija od OS thread-a, ali nije besplatna.
* Pojam goroutine leak-a kao posledice nekontrolisanog kreiranja.

### Česta greška

Nije precizno reći:

> "Goroutines su besplatne — mogu ih kreirati koliko god hoću bez posledica."

Preciznije je:

> "Goroutines su jeftine, ali ne besplatne. Svaka koristi memoriju i runtime resurse. Nekontrolisano kreiranje može dovesti do goroutine leak-a."

---

## Pitanje 03 — Kako se pokreće goroutine?

### Odgovor

Goroutine se pokreće pomoću ključne reči `go`.

Osnovni oblik:

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

Bitno je razumeti da `go worker()` nije isto što i `worker()`.

Kod običnog poziva `worker()` trenutna goroutine čeka da se `worker` završi. Kod `go worker()` Go pokreće novu goroutine i trenutna goroutine može odmah da nastavi sa izvršavanjem:

```go
func main() {
	go worker()

	fmt.Println("main continues")
}
```

`main` ne čeka automatski da `worker` završi. To je jedna od najvažnijih stvari koju kandidat mora da razume.

### Šta interviewer očekuje?

* Sintaksa `go function()`.
* Da se može koristiti i sa imenovanim i anonimnim funkcijama.
* Da se mogu proslediti argumenti.
* Razlika između `go worker()` i `worker()`.

### Česta greška

Nije precizno reći:

> "`go worker()` radi isto što i `worker()`, samo brže."

Preciznije je:

> "`go worker()` pokreće novu goroutine i ne čeka njen završetak. `worker()` blokira trenutnu goroutine dok se `worker` ne završi."

---

## Pitanje 04 — Da li `go` garantuje da će goroutine završiti pre `main()` funkcije?

### Odgovor

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

a da `worker` nikada ne bude ispisan. Zašto? Zato što se program završava kada se završi glavna goroutine — odnosno kada se završi `main()` funkcija:

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

Jedan od najčešćih primera pokušaja zaštite od ovog problema je korišćenje `time.Sleep`:

```go
func main() {
	go worker()

	time.Sleep(time.Second)
}
```

Ovo može izgledati kao da rešava problem, ali nije dobar mehanizam sinhronizacije. Problem je što ne znaš da li je jedna sekunda previše, dovoljno ili premalo. Na sporijem sistemu `worker` možda neće završiti za jednu sekundu. Na brzom sistemu program možda nepotrebno čeka.

U kasnijim modulima koriste se pravi mehanizmi sinhronizacije, kao što su:

* `sync.WaitGroup`;
* channels;
* `context`.

### Šta interviewer očekuje?

* Da `go` samo pokreće goroutine, ne čeka je.
* Da program završava kada `main` goroutine završi.
* Da `time.Sleep` nije mehanizam koordinacije.
* Da se za čekanje goroutines koriste odgovarajući mehanizmi.

### Česta greška

Nije precizno reći:

> "Go automatski čeka sve goroutines pre izlaska iz programa."

Preciznije je:

> "Program se završava kada završi `main` goroutine, bez obzira na stanje ostalih goroutines. `go` ne znači čekanje."

---

## Pitanje 05 — Šta je channel i čemu služi?

### Odgovor

Channel je Go mehanizam za **komunikaciju i koordinaciju između goroutines**.

Možeš ga posmatrati kao kanal kroz koji goroutines šalju i primaju vrednosti.

Kreiranje:

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

Jedna od ključnih ideja Go concurrency modela jeste da goroutines mogu komunicirati razmenom podataka preko channels, umesto direktne manipulacije zajedničkim stanjem:

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

Channel nije samo struktura podataka. Kod unbuffered channel-a slanje i primanje su direktno koordinisani, što ga čini i **mehanizmom sinhronizacije**.

### Šta interviewer očekuje?

* Šta je channel.
* Da je tipiziran.
* Kako se kreira.
* Kako se šalje i prima vrednost.
* Da channel može služiti i za komunikaciju i za koordinaciju goroutines.

### Česta greška

Nije precizno reći:

> "Channel je queue — pošiljalac može uvek da stavi vrednost i nastavi."

Preciznije je:

> "Unbuffered channel nema interni buffer. Send blokira dok receiver nije spreman, što channel čini mehanizmom sinhronizacije, a ne samo prenosom podataka."

---

## Pitanje 06 — Šta je goroutine i po čemu se razlikuje od klasičnog OS thread-a?

### Odgovor

Goroutine je lagana izvršna jedinica kojom upravlja Go runtime. Pokreće se pomoću `go` naredbe:

```go
go doWork()
```

Za razliku od OS thread-a, goroutine nije direktno mapirana 1:1 na jedan sistemski thread. Go runtime koristi scheduler koji raspoređuje veliki broj goroutines preko manjeg ili odgovarajućeg broja OS thread-ova.

Zbog toga je kreiranje goroutine relativno jeftino i moguće je imati veliki broj istovremeno aktivnih goroutines.

### Šta interviewer očekuje?

* Definiciju goroutine kao izvršne jedinice kojom upravlja Go runtime.
* Da goroutine nije direktno mapirana 1:1 na OS thread.
* Da Go scheduler raspoređuje goroutines na OS thread-ove.
* Da je kreiranje goroutine jeftinije od kreiranja OS thread-a.

### Česta greška

Nije precizno reći:

> "Goroutine je samo lakši thread — inače rade isto."

Preciznije je:

> "Goroutine i OS thread nemaju odnos 1:1. Go scheduler može rasporediti mnogo goroutines na mali broj OS thread-ova, što daje drugačiji model upravljanja konkurentnošću."

---

## Pitanje 07 — Šta se dešava kada napišemo `go process()`?

### Odgovor

Go pokreće `process` kao novu goroutine. Pozivajuća goroutine ne čeka da se `process` završi, već nastavlja sa svojim izvršavanjem.

To znači da:

```go
go process()
fmt.Println("done")
```

ne garantuje da će `process()` završiti pre nego što se ispiše `"done"`.

Scheduler odlučuje kada će nova goroutine dobiti priliku za izvršavanje.

### Šta interviewer očekuje?

* Da `go process()` pokreće goroutine, a ne da čeka njen završetak.
* Da pozivajuća goroutine odmah nastavlja izvršavanje.
* Da scheduler odlučuje o rasporedu izvršavanja.

### Česta greška

Nije precizno reći:

> "`go process()` znači: izvrši `process` odmah, u pozadini, i onda nastavi."

Preciznije je:

> "`go process()` pokreće goroutine i prepušta scheduler-u kada će ona dobiti CPU vreme. Nema garancije o redosledu ni trenutku izvršavanja."

---

## Pitanje 08 — Da li je izvršavanje goroutines determinističko?

### Odgovor

Ne. Redosled izvršavanja konkurentnih goroutines generalno nije deterministički.

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

Program ne treba da zavisi od slučajnog redosleda izvršavanja goroutines.

Ako redosled predstavlja poslovni zahtev, potrebno je eksplicitno uvesti mehanizam koordinacije.

### Šta interviewer očekuje?

* Da redosled goroutines nije garantovan.
* Da program ne sme da zavisi od slučajnog scheduling redosleda.
* Da je za garantovani redosled potrebna eksplicitna sinhronizacija.

### Česta greška

Nije precizno reći:

> "Goroutines se uvek izvršavaju u redosledu u kome su pokrenute."

Preciznije je:

> "Redosled izvršavanja goroutines kontroliše scheduler i nije deterministički. Program koji zavisi od pretpostavljenog redosleda ima potencijalni bug."

---

## Pitanje 09 — Šta je channel u Go-u?

### Odgovor

Channel je mehanizam za komunikaciju i sinhronizaciju između goroutines.

Kanal se kreira pomoću:

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

Osnovna ideja Go concurrency modela jeste da goroutines mogu koordinisati rad tako što komuniciraju preko channels, umesto da direktno dele zajedničko stanje.

### Šta interviewer očekuje?

* Definiciju channel-a kao mehanizma komunikacije i sinhronizacije.
* Sintaksu kreiranja: `make(chan T)`.
* Sintaksu slanja i primanja.
* Da channels smanjuju potrebu za direktnim deljenjem memorije.

### Česta greška

Nije precizno reći:

> "Channel je samo način da se prosledi vrednost između funkcija."

Preciznije je:

> "Channel je mehanizam za komunikaciju i sinhronizaciju između goroutines — ne samo prenos vrednosti, već i koordinacija toka izvršavanja."

---

## Pitanje 10 — Zašto se channels često opisuju kao mehanizam za komunikaciju između goroutines?

### Odgovor

Zato što channel omogućava jednoj goroutine da preda podatak drugoj goroutine bez potrebe da obe direktno manipulišu istom promenljivom.

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

Jedna goroutine proizvodi vrednost, dok druga prima tu vrednost. Channel istovremeno predstavlja i mehanizam sinhronizacije jer slanje i prijem mogu blokirati izvršavanje — goroutine se koordinišu samim činom komunikacije.

### Šta interviewer očekuje?

* Da channel prenosi podatke između goroutines bez direktnog deljenja promenljive.
* Da send i receive operacije imaju efekat sinhronizacije.
* Razliku između komunikacije i deljenja memorije.

### Česta greška

Nije precizno reći:

> "Channels su alternativa za globalne promenljive."

Preciznije je:

> "Channels nisu samo alternativa deljenim promenljivima — oni su mehanizam komunikacije koji ujedno koordiniše redosled izvršavanja goroutines kroz prirodu blokirajućih operacija."

---

## Pitanje 11 — Šta znači izraz `ch <- value`?

### Odgovor

To je **send operacija**.

Vrednost `value` se šalje u channel `ch`:

```go
ch <- value
```

Kod unbuffered channel-a send operacija može blokirati dok druga goroutine ne bude spremna da primi vrednost:

```go
ch := make(chan int)

go func() {
	ch <- 42  // blokira dok main ne uradi receive
}()

value := <-ch
fmt.Println(value)
```

### Šta interviewer očekuje?

* Da je `ch <- value` send operacija.
* Da send može blokirati kod unbuffered channel-a.
* Da mora postojati odgovarajući receiver da bi se send nastavio.

### Česta greška

Nije precizno reći:

> "`ch <- value` uvek odmah završi."

Preciznije je:

> "`ch <- value` može blokirati ako je channel unbuffered i nema goroutine koja čeka da primi vrednost."

---

## Pitanje 12 — Šta znači izraz `value := <-ch`?

### Odgovor

To je **receive operacija**.

Program pokušava da primi jednu vrednost iz channel-a `ch`:

```go
value := <-ch
```

Ako vrednost trenutno nije dostupna, receive operacija može blokirati dok druga goroutine ne pošalje vrednost:

```go
ch := make(chan int)

go func() {
	ch <- 42
}()

value := <-ch  // čeka dok goroutine ne pošalje
fmt.Println(value)
```

### Šta interviewer očekuje?

* Da je `value := <-ch` receive operacija.
* Da receive može blokirati.
* Razlika između `ch <- value` (send) i `value := <-ch` (receive).

### Česta greška

Nije precizno reći:

> "`value := <-ch` uvek odmah dobija vrednost."

Preciznije je:

> "`value := <-ch` blokira dok vrednost ne postane dostupna na channel-u."

---

## Pitanje 13 — Kako channel može da se koristi za sinhronizaciju?

### Odgovor

Channel može da primora jednu goroutine da sačeka drugu.

Na primer:

```go
done := make(chan struct{})

go func() {
	// neki posao

	close(done)
}()

<-done
```

Glavna goroutine blokira na `<-done` dok druga goroutine ne završi posao i zatvori channel.

Na taj način channel nije samo sredstvo za prenos podataka, već i alat za koordinaciju izvršavanja:

```text
worker goroutine
    │
    ├── posao
    │
    └── close(done)
              │
              ▼
         main se odblokira
              │
              └── nastavlja
```

Ovakav obrazac je veoma čest u Go concurrency kodu.

### Šta interviewer očekuje?

* Da channel može blokirati goroutine dok se ne ispuni uslov.
* Obrazac `done := make(chan struct{})` + `close(done)` + `<-done`.
* Da channel služi i kao sinhronizacioni mehanizam, ne samo za prenos vrednosti.

### Česta greška

Nije precizno reći:

> "Channel se koristi samo za slanje i primanje podataka."

Preciznije je:

> "Channel se koristi i za koordinaciju — blokiranje jedne goroutine dok druga ne završi određeni posao, bez potrebe da se prenos konkretnih vrednosti vrši."

---

## Pitanje 14 — Šta znači da je channel operacija blokirajuća?

### Odgovor

To znači da goroutine koja izvršava operaciju može biti zaustavljena dok se ne ispuni uslov potreban za nastavak.

Kod unbuffered channel-a:

```go
ch := make(chan int)
```

Send:

```go
ch <- 10
```

čeka odgovarajući receive. Receive:

```go
value := <-ch
```

čeka odgovarajući send.

Ovo ponašanje omogućava prirodnu sinhronizaciju između goroutines. Goroutine se susreću na channel-u — jedna čeka da pošalje, druga da primi — i nastavljaju tek kada su obe strane spremne.

### Šta interviewer očekuje?

* Definiciju blokirajuće operacije u kontekstu channel-a.
* Da send čeka receiver i obratno kod unbuffered channel-a.
* Da blokirajuće operacije omogućavaju prirodnu sinhronizaciju.

### Česta greška

Nije precizno reći:

> "Blokirajuća operacija znači da program stoji i ne radi ništa."

Preciznije je:

> "Blokirajuća operacija zaustavlja konkretnu goroutine dok se ne ispuni uslov. Ostale goroutines mogu nastaviti da se izvršavaju."

---

## Pitanje 15 — Zašto se u Go-u ne preporučuje posmatranje goroutines samo kao „jeftinih thread-ova"?

### Odgovor

Zato što goroutine predstavlja deo šireg Go concurrency modela.

Važni elementi tog modela su:

* goroutine,
* channel,
* scheduler,
* blokirajuće operacije,
* komunikacija,
* sinhronizacija,
* ownership podataka.

Samo kreiranje velikog broja goroutines nije dovoljno za pravilan concurrency dizajn. Potrebno je razumeti kako goroutines komuniciraju, kada blokiraju i kako se njihov životni ciklus završava.

Goroutine koja nema jasnu exit strategiju može postati goroutine leak. Goroutine koja nekontrolisano pristupa deljenim podacima može izazvati data race. Goroutines koje čekaju jedna drugu mogu izazvati deadlock.

Zbog svega toga, goroutine je mnogo više od „jeftinog thread-a".

### Šta interviewer očekuje?

* Da goroutine nije samo jeftini thread.
* Da goroutine nosi sa sobom odgovornost za komunikaciju, lifecycle i sinhronizaciju.
* Da je razumevanje scheduler-a, channel-a i koordinacije neophodan deo concurrency znanja.

### Česta greška

Nije precizno reći:

> "Goroutines su kao thread-ovi, samo jeftiniji — koristim ih isto."

Preciznije je:

> "Goroutines su deo Go concurrency modela koji uključuje scheduler, channels i sinhronizacione primitive. Tretiranje goroutines samo kao jeftinijih thread-ova propušta ključne aspekte njihovog pravilnog korišćenja."

---

## Pitanje 16 — Da li pokretanje goroutine garantuje da će se njeno izvršavanje desiti pre nego što se trenutna funkcija nastavi?

### Odgovor

Ne.

Poziv:

```go
go doWork()
```

samo omogućava Go runtime-u da zakaže izvršavanje `doWork` funkcije kao goroutine. Ne postoji garancija da će nova goroutine odmah početi sa izvršavanjem.

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

U ovom primeru program može završiti pre nego što goroutine dobije priliku da izvrši `fmt.Println`.

`go` statement ne znači:

> "Izvrši ovo odmah paralelno."

On znači:

> "Pokreni ovu funkciju kao goroutine čije će izvršavanje Go scheduler rasporediti."

### Šta interviewer očekuje?

* Da `go` ne garantuje trenutno izvršavanje nove goroutine.
* Da scheduler odlučuje kada će goroutine dobiti CPU vreme.
* Da program može završiti pre nego što nova goroutine počne da se izvršava.

### Česta greška

Nije precizno reći:

> "Goroutine se odmah počinje izvršavati paralelno sa ostatkom koda."

Preciznije je:

> "Goroutine se stavlja u red za scheduler. Kada će tačno početi da se izvršava zavisi od rasporeda koji scheduler odredi."

---

## Pitanje 17 — Šta se dešava sa goroutines kada `main` funkcija završi?

### Odgovor

Kada `main` goroutine završi izvršavanje, proces se završava. Preostale goroutines se ne čekaju automatski.

Na primer:

```go
func main() {
	go func() {
		time.Sleep(time.Second)
		fmt.Println("done")
	}()
}
```

Program može završiti odmah, bez ispisa `done`. Razlog je to što `main` ne čeka novu goroutine.

Za koordinaciju se koriste mehanizmi kao što su `sync.WaitGroup`, channels i `context`.

Primer sa `sync.WaitGroup`:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
	defer wg.Done()

	fmt.Println("done")
}()

wg.Wait()
```

Ovde `main` goroutine čeka da radna goroutine završi.

### Šta interviewer očekuje?

* Da završetak `main` goroutine završava proces.
* Da preostale goroutines bivaju prekinute bez čekanja.
* Poznavanje bar jednog mehanizma koordinacije.

### Česta greška

Nije precizno reći:

> "Go čeka da sve goroutines završe pre nego što program izađe."

Preciznije je:

> "Go ne čeka automatski. Završetak `main` goroutine odmah završava proces i sve preostale goroutines bivaju prekinute."

---

## Pitanje 18 — Da li dve goroutines automatski izvršavaju kod paralelno?

### Odgovor

Ne nužno.

Treba razlikovati:

* **concurrency** — više jedinica rada može biti u toku;
* **parallelism** — više jedinica rada se zaista izvršava istovremeno na različitim CPU jezgrima.

Go omogućava konkurentno izvršavanje pomoću goroutines, ali stvarni paralelizam zavisi od runtime-a, raspoloživih CPU resursa i podešavanja izvršavanja.

Primer:

```go
go taskA()
go taskB()
```

znači da su `taskA` i `taskB` pokrenuti kao goroutines. Ne znači da će CPU nužno izvršavati:

```text
CPU 1 → taskA
CPU 2 → taskB
```

istovremeno. Go runtime scheduler odlučuje kada i gde će se goroutines izvršavati.

### Šta interviewer očekuje?

* Razlika između concurrency i parallelism.
* Da goroutines obezbeđuju concurrency, ali ne garantuju parallelism.
* Da scheduler i dostupni CPU resursi određuju stvarno izvršavanje.

### Česta greška

Nije precizno reći:

> "Dve goroutines uvek rade paralelno — svaka na svom CPU jezgru."

Preciznije je:

> "Dve goroutines su konkurentne — mogu se izvršavati naizmenično na istom CPU jezgru. Paralelno izvršavanje zavisi od broja dostupnih jezgara i scheduler-a."

---

## Pitanje 19 — Da li redosled izvršavanja goroutines može da se predvidi?

### Odgovor

Ne treba pretpostavljati određeni redosled izvršavanja goroutines.

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

Scheduler odlučuje kada će pojedina goroutine dobiti priliku za izvršavanje.

Ako redosled predstavlja poslovni zahtev, potrebno je eksplicitno uvesti mehanizam koordinacije. Na primer, channel može izraziti zavisnost:

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

### Šta interviewer očekuje?

* Da redosled goroutines nije garantovan bez eksplicitne sinhronizacije.
* Da program ne sme zavisiti od pretpostavljenog scheduling redosleda.
* Da channel ili drugi mehanizmi mogu uvesti garantovani redosled.

### Česta greška

Nije precizno reći:

> "Goroutines se uvek izvršavaju u redosledu u kome su pokrenute, pa mogu zavisiti od tog redosleda."

Preciznije je:

> "Scheduling redosled nije garantovan. Program koji zavisi od toga uvodi nedeterminizam i potencijalne bugove."

---

## Pitanje 20 — Šta znači da je channel blokirajući mehanizam?

### Odgovor

Send i receive operacije nad channel-om mogu blokirati goroutine dok se ne ispune odgovarajući uslovi.

Kod unbuffered channel-a:

```go
ch := make(chan int)
```

slanje:

```go
ch <- 42
```

ne može da završi dok druga goroutine ne bude spremna da primi vrednost. Analogno tome:

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

Ovde channel predstavlja mehanizam komunikacije i koordinacije između dve goroutines. Channel nije samo „cev" za prenos podataka — on istovremeno može da predstavlja **sinhronizacionu tačku** između goroutines.

### Šta interviewer očekuje?

* Da send blokira dok nema receivera (unbuffered).
* Da receive blokira dok nema sende-ra.
* Da blokirajuća priroda channel-a čini ga sinhronizacionim mehanizmom.

### Česta greška

Nije precizno reći:

> "Blokirajuće znači da channel usporava program."

Preciznije je:

> "Blokirajuće ponašanje channel-a je korisna osobina — goroutines se prirodno koordinišu bez eksplicitnih lock-ova."

---

## Pitanje 21 — Šta se dešava kada šaljemo vrednost na unbuffered channel, a niko trenutno ne prima tu vrednost?

### Odgovor

Goroutine koja pokušava da pošalje vrednost blokira se.

Na primer:

```go
ch := make(chan int)

ch <- 10

fmt.Println("done")
```

Ako ne postoji druga goroutine koja prima vrednost sa `ch`, izvršavanje se zaustavlja na `ch <- 10`. `fmt.Println("done")` se neće izvršiti.

Ako nema druge goroutine koja može da omogući nastavak izvršavanja, program može završiti sa:

```text
fatal error: all goroutines are asleep - deadlock!
```

Ovo je jedan od osnovnih oblika deadlock-a u Go concurrency modelu.

### Šta interviewer očekuje?

* Da send na unbuffered channel bez receivera blokira goroutine.
* Da blokiranje bez mogućnosti napretka dovodi do deadlock-a.
* Da runtime detektuje ovaj deadlock.

### Česta greška

Nije precizno reći:

> "Vrednost se čuva u channel-u dok neko ne primi."

Preciznije je:

> "Unbuffered channel nema memoriju za čuvanje vrednosti. Send blokira sve dok receiver nije spreman. Nema odlaganja."

---

## Pitanje 22 — Šta se dešava kada primamo vrednost sa unbuffered channel-a, a niko ne šalje vrednost?

### Odgovor

Receive operacija blokira.

Na primer:

```go
ch := make(chan int)

value := <-ch

fmt.Println(value)
```

Ako nijedna druga goroutine ne izvršava send na `ch`, receive operacija nema odakle da dobije vrednost. Goroutine ostaje blokirana.

Ako su sve goroutines u stanju čekanja i ne postoji mogućnost napretka, runtime detektuje deadlock:

```text
fatal error: all goroutines are asleep - deadlock!
```

Zbog toga kod rada sa channels moramo razumeti ne samo šta se šalje i prima, već i **ko je odgovoran da druga strana komunikacije zaista postoji**.

### Šta interviewer očekuje?

* Da receive bez sender-a blokira goroutine.
* Da blokiranje bez mogućnosti napretka dovodi do deadlock-a.
* Da je odgovornost za obe strane komunikacije važan deo dizajna.

### Česta greška

Nije precizno reći:

> "Receive sa praznog channel-a vraća nulu."

Preciznije je:

> "Receive sa unbuffered channel-a bez sender-a blokira zauvek. Nula vrednost se dobija samo pri receive-u sa zatvorenog channel-a."

---

## Pitanje 23 — Zašto se channel često koristi kao mehanizam koordinacije, a ne samo za prenos podataka?

### Odgovor

Zato što komunikacija preko channel-a može istovremeno da uspostavi odnos između trenutka kada je jedna goroutine završila posao i trenutka kada druga goroutine može da nastavi.

Na primer:

```go
done := make(chan struct{})

go func() {
	doWork()
	close(done)
}()

<-done
```

Ovde channel ne prenosi konkretan poslovni podatak. Njegova svrha je signalizacija:

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

Channel može predstavljati:

* signal završetka;
* dostupnost posla;
* rezultat rada;
* zahtev;
* događaj;
* koordinaciju između goroutines.

Zbog toga je razumevanje channel-a mnogo šire od razumevanja same sintakse slanja i primanja.

### Šta interviewer očekuje?

* Da channel može koordinisati bez prenosa konkretnih poslovnih podataka.
* Obrazac signal završetka: `done := make(chan struct{})` + `close(done)` + `<-done`.
* Da channel prenosi i informacije i timing — kada se nešto desilo.

### Česta greška

Nije precizno reći:

> "Channel se koristi samo kada treba preneti vrednost između goroutines."

Preciznije je:

> "Channel se koristi i za čistu koordinaciju — signal da je goroutine završila rad, bez ikakvog prenosa podataka."

---

## Pitanje 24 — Da li je dobro koristiti `time.Sleep` da bismo „sačekali" drugu goroutine?

### Odgovor

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

`Sleep` predstavlja vremensko čekanje, a ne sinhronizaciju. Ako `doWork` traje duže od jedne sekunde, program nastavlja prerano. Ako traje kraće, program nepotrebno gubi vreme.

Ako želimo da čekamo završetak rada, treba koristiti odgovarajući mehanizam koordinacije:

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

### Šta interviewer očekuje?

* Da `time.Sleep` nije mehanizam sinhronizacije.
* Da `Sleep` čeka vreme, ne događaj.
* Da `sync.WaitGroup`, channels ili `context` treba koristiti za koordinaciju.

### Česta greška

Nije precizno reći:

> "`time.Sleep(time.Second)` je siguran način da sačekam goroutine — sekunda je dovoljno."

Preciznije je:

> "`time.Sleep` ne garantuje da je goroutine završila rad. To je vremensko čekanje, ne sinhronizacija. Koristi se `sync.WaitGroup` ili channel."

---

## Pitanje 25 — Koja je osnovna razlika između „čekanja određeno vreme" i „čekanja događaja"?

### Odgovor

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

Na primer:

```go
done := make(chan struct{})

go func() {
	doWork()
	close(done)
}()

<-done
```

Ovde vreme potrebno za `doWork` nije unapred poznato niti je bitno. Bitno je samo da se događaj `doWork` završio dogodi pre nego što receiver nastavi.

U concurrency kodu čekanje događaja je obično ispravan model kada je potrebno koordinisati goroutines.

### Šta interviewer očekuje?

* Razlika između vremenskog čekanja i čekanja događaja.
* Da channel čeka događaj, a `Sleep` čeka vreme.
* Da čekanje događaja ne zavisi od trajanja posla.

### Česta greška

Nije precizno reći:

> "Svejedno je da li čekam vreme ili događaj — rezultat je isti."

Preciznije je:

> "Čekanje vremena je uvek pogrešna procena: ili predugo ili prekratko. Čekanje događaja je tačno — program nastavlja tačno kada treba."

---

## Pitanje 26 — Šta se dešava kada goroutine pošalje vrednost na unbuffered channel?

### Odgovor

Kod unbuffered channel-a, slanje ne može da se završi samo smeštanjem vrednosti u interni buffer, zato što takav buffer ne postoji.

Pošiljalac:

```go
ch <- value
```

blokira se dok druga goroutine ne bude spremna da primi vrednost:

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

Ovde goroutine koja izvršava `ch <- 42` čeka dok `main` goroutine ne izvrši `value := <-ch`. Tek tada može da se izvrši komunikacija između dve goroutines.

Ovo je jedna od osnovnih karakteristika unbuffered channel-a:

> **Slanje i prijem predstavljaju koordinisanu tačku susreta između goroutines.**

### Šta interviewer očekuje?

* Da send na unbuffered channel blokira dok receiver nije spreman.
* Da send i receive predstavljaju sinhronizacionu tačku susreta.
* Da unbuffered channel nema interni buffer.

### Česta greška

Nije precizno reći:

> "Goroutine šalje vrednost i odmah nastavlja."

Preciznije je:

> "Kod unbuffered channel-a goroutine koja šalje blokira dok goroutine koja prima nije spremna. Tek tada obe nastavljaju."

---

## Pitanje 27 — Šta se dešava kada goroutine primi vrednost sa unbuffered channel-a, ali trenutno nema pošiljaoca?

### Odgovor

Ako niko trenutno ne šalje vrednost na channel:

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

goroutine ostaje blokirana. Ako je to jedina aktivna goroutine, program može završiti sa:

```text
fatal error: all goroutines are asleep - deadlock!
```

Zato kod rada sa channels moramo razumeti ne samo šta se šalje i prima, već i **ko je odgovoran da druga strana komunikacije zaista postoji**.

### Šta interviewer očekuje?

* Da receive bez sender-a blokira goroutine.
* Da nema vrednosti koja se „privremeno čuva" — unbuffered channel je samo rendezvous.
* Da blokiranje bez izlaza dovodi do deadlock-a.

### Česta greška

Nije precizno reći:

> "Receive na praznom channel-u odmah vraća zero value."

Preciznije je:

> "Receive na unbuffered channel-u bez sender-a blokira zauvek. Zero value se dobija samo sa zatvorenog channel-a."

---

## Pitanje 28 — Koja je osnovna razlika između buffered i unbuffered channel-a?

### Odgovor

**Unbuffered channel** (`make(chan int)`) nema interni buffer. Slanje i primanje moraju se desiti istovremeno — pošiljalac blokira dok primalac nije spreman, i obratno.

**Buffered channel** (`make(chan int, N)`) ima buffer kapaciteta N. Slanje može da se završi bez čekanja receivera, sve dok u bufferu postoji slobodan prostor.

Primer sa buffered channel-om kapaciteta 2:

```go
ch := make(chan int, 2)

ch <- 10  // ne blokira, buffer [10]
ch <- 20  // ne blokira, buffer [10, 20]
```

Oba slanja mogu da se izvrše bez trenutnog receivera zato što channel može da zadrži dve vrednosti.

Ali:

```go
ch <- 30  // blokira — buffer pun
```

blokiraće se dok neko ne preuzme jednu od prethodno poslatih vrednosti.

### Šta interviewer očekuje?

* Razlika između `make(chan int)` i `make(chan int, N)`.
* Da unbuffered zahteva istovremeni send i receive.
* Da buffered dozvoljava asinhronost do kapaciteta.
* Da pun buffered channel i dalje blokira.

### Česta greška

Nije precizno reći:

> "Buffered channel nikada ne blokira."

Preciznije je:

> "Buffered channel blokira kad se buffer napuni. Buffering menja trenutak blokiranja, ali ga ne eliminiše."

---

## Pitanje 29 — Da li buffered channel znači da slanje nikada neće blokirati?

### Odgovor

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

Konceptualno:

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

Važno je razumeti da buffering **ne uklanja potrebu za koordinacijom**. On samo menja trenutak u kome može doći do blokiranja.

### Šta interviewer očekuje?

* Da buffered channel blokira kada je buffer pun.
* Da buffering ne uklanja potrebu za koordinacijom.
* Da se buffer popunjava i praznenja po FIFO redosledu.

### Česta greška

Nije precizno reći:

> "Koristim buffered channel da se nikad ne brinem o blokiranju."

Preciznije je:

> "Buffered channel odlaže blokiranja, ali ga ne eliminiše. Pun buffer i dalje blokira sender."

---

## Pitanje 30 — Šta predstavljaju channel directions u Go-u?

### Odgovor

Channel direction definiše da li channel može samo da šalje, samo da prima, ili oboje.

**Bidirectional** (podrazumevani):

```go
chan int
```

**Send-only:**

```go
chan<- int
```

**Receive-only:**

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

```go
func consumer(ch <-chan int) {
	value := <-ch
	fmt.Println(value)
}
```

`consumer` može da prima vrednosti, ali ne može da ih šalje.

### Šta interviewer očekuje?

* Poznavanje sva tri tipa direction-a.
* Sintaksu za svaki tip.
* Da direction ograničava kako se channel može koristiti unutar funkcije.

### Česta greška

Nije precizno reći:

> "Channel direction je samo hint za programera."

Preciznije je:

> "Channel direction je deo type sistema. Compiler odbija pogrešnu upotrebu — send na receive-only channel je compile error."

---

## Pitanje 31 — Zašto su channel directions korisni?

### Odgovor

Channel directions omogućavaju da API jasno definiše odgovornost funkcije.

Na primer:

```go
func producer(ch chan<- int)
```

govori čitaocu:

> Ova funkcija proizvodi podatke i šalje ih u channel.

Dok:

```go
func consumer(ch <-chan int)
```

govori:

> Ova funkcija konzumira podatke iz channel-a.

Time se smanjuje mogućnost greške i povećava čitljivost koda.

Još važnije, compiler može da proveri da li pokušavamo da koristimo channel na način koji nije dozvoljen njegovim tipom:

```go
func consumer(ch <-chan int) {
	ch <- 10  // compile error: send on receive-only channel
}
```

Ovo nije dozvoljeno, jer je `ch` receive-only channel.

### Šta interviewer očekuje?

* Da direction komunicira nameru funkcije čitaocu koda.
* Da compiler proverava ispravnost i sprečava greške.
* Da direction je deo API dizajna, ne samo konvencija.

### Česta greška

Nije precizno reći:

> "Channel directions su samo za dokumentaciju — compiler ih ionako ne proverava."

Preciznije je:

> "Compiler aktivno proverava direction. Pokušaj send-a na receive-only channel ili receive-a na send-only channel je compile error."

---

## Pitanje 32 — Kako se koristi `range` za čitanje vrednosti iz channel-a?

### Odgovor

Kanal možemo koristiti kao izvor vrednosti u `for range` petlji:

```go
for value := range ch {
	fmt.Println(value)
}
```

Petlja nastavlja da prima vrednosti sve dok channel ne bude zatvoren.

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

`range` se završava nakon što je channel zatvoren i nakon što su sve vrednosti koje su već poslate pročitane.

### Šta interviewer očekuje?

* Sintaksa `for value := range ch`.
* Da `range` blokira između vrednosti.
* Da `range` završava kada je channel zatvoren i sve vrednosti pročitane.
* Veza između `range` i `close`.

### Česta greška

Nije precizno reći:

> "`range` nad channel-om završava kada channel ostane prazan."

Preciznije je:

> "`range` završava tek kada je channel eksplicitno zatvoren sa `close`. Bez `close` petlja ostaje blokirana zauvek."

---

## Pitanje 33 — Zašto je `close` važan kada koristimo `range` nad channel-om?

### Odgovor

`range` mora imati način da zna da više neće biti novih vrednosti.

Zatvaranje channel-a predstavlja signal:

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

Ovde sender poseduje lifecycle channel-a i odgovoran je za njegovo zatvaranje. Kada sender zatvori channel, `range` petlja u receiver-u se završava.

Važno: **Receiver uglavnom ne treba da zatvara channel koji nije njegov.**

Pravilo je:

> Goroutine koja proizvodi vrednosti i zna da je proizvodnja završena obično je odgovorna za `close`.

### Šta interviewer očekuje?

* Da `close` signalizira kraj toka podataka za `range`.
* Da bez `close` petlja nikada ne završava.
* Da sender uglavnom poseduje odgovornost za `close`.

### Česta greška

Nije precizno reći:

> "Receiver zatvara channel kada završi sa čitanjem."

Preciznije je:

> "Receiver uglavnom ne zatvara channel. Sender koji zna da je slanje završeno je odgovoran za `close`."

---

## Pitanje 34 — Čemu služi `select` u Go concurrency modelu?

### Odgovor

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

Ako je jedna od operacija spremna, `select` može da je izabere. Ako nijedna nije spremna, `select` blokira dok neka od navedenih operacija ne postane moguća.

### Šta interviewer očekuje?

* Da `select` čeka više channel operacija istovremeno.
* Da blokira dok nijedna case nije dostupna.
* Da se koristi za timeout, cancellation i čekanje višestrukih izvora podataka.

### Česta greška

Nije precizno reći:

> "`select` izvršava prvu `case` opciju koja je navedena."

Preciznije je:

> "Ako je više `case` opcija istovremeno dostupno, `select` bira pseudo-nasumično. Redosled case-ova ne garantuje redosled izvršavanja."

---

## Pitanje 35 — Koja je osnovna razlika između `select` i običnog channel receive-a?

### Odgovor

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
* koordinacija više goroutines;
* reagovanje na prvi channel koji postane spreman.

`select` je zato jedan od osnovnih mehanizama za izgradnju strukturisane konkurentnosti u Go-u.

Tipičan primer sa timeoutom:

```go
select {
case value := <-ch:
	process(value)

case <-time.After(time.Second):
	fmt.Println("timeout")
}
```

Tipičan primer sa cancellation-om:

```go
select {
case value := <-work:
	process(value)

case <-ctx.Done():
	return
}
```

### Šta interviewer očekuje?

* Razlika između `<-ch` (jedan channel) i `select` (više channels).
* Da `select` reaguje na prvi dostupan case.
* Primeri upotrebe: timeout, cancellation, višestruki izvori.

### Česta greška

Nije precizno reći:

> "`select` je samo `switch` za channels."

Preciznije je:

> "`select` blokira dok jedan od case-ova ne postane moguć, a ako je više case-ova istovremeno dostupno, bira nasumično. `switch` nema ovu semantiku."

---

## Pitanje 36 — Da li `go` ključna reč garantuje da će goroutine završiti posao pre nego što se program završi?

### Odgovor

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

Program može završiti pre nego što goroutine dobije priliku da izvrši `Println`.

`main` goroutine predstavlja početnu tačku programa. Kada se `main` završi, proces se završava, zajedno sa svim preostalim goroutines.

Zato mora postojati eksplicitna koordinacija ako je rezultat goroutine važan. Na primer:

```go
done := make(chan struct{})

go func() {
	defer close(done)

	doWork()
}()

<-done
```

U realnom kodu često se koriste `sync.WaitGroup`, `context.Context` ili kombinacija više concurrency mehanizama.

### Šta interviewer očekuje?

* Da `go` ne implicira čekanje.
* Da završetak `main` goroutine završava proces.
* Da je eksplicitna koordinacija neophodna ako rezultat goroutine nije nebitan.

### Česta greška

Nije precizno reći:

> "Program čeka sve pokrenute goroutines pre nego što izađe."

Preciznije je:

> "Program izlazi kada `main` goroutine završi, bez obzira na stanje ostalih goroutines."

---

## Pitanje 37 — Da li je goroutine isto što i OS thread?

### Odgovor

Ne.

Goroutine je Go-ov concurrency primitive kojim upravlja Go runtime, dok je OS thread resurs kojim upravlja operativni sistem.

Jedan OS thread može izvršavati Go kod za više različitih goroutines tokom vremena.

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

Go runtime raspoređuje goroutines na raspoložive OS thread-ove. Zbog toga Go može da podrži veliki broj goroutines bez potrebe da svaka goroutine direktno odgovara jednom OS thread-u.

### Šta interviewer očekuje?

* Da goroutine i OS thread nisu isto.
* Da Go runtime mapira goroutines na OS thread-ove (N:M odnos).
* Da je jedan OS thread može izvršavati više goroutines naizmenično.

### Česta greška

Nije precizno reći:

> "Svaka goroutine ima svoj OS thread."

Preciznije je:

> "Go koristi N:M threading model — mnogo goroutines se raspoređuje na manji broj OS thread-ova."

---

## Pitanje 38 — Da li zbog toga treba pokretati goroutine za svaku sitnicu?

### Odgovor

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

Pokretanje goroutine ima smisla kada:

* posao može biti izvršen konkurentno sa nečim drugim;
* posao je I/O-bound i može čekati bez blokiranja ostatka programa;
* posao traje dovoljno dugo da overhead goroutine bude zanemarljiv.

### Šta interviewer očekuje?

* Da goroutine nije besplatna.
* Da treba imati razlog za pokretanje goroutine.
* Da nekontrolisano kreiranje goroutines dovodi do problema.

### Česta greška

Nije precizno reći:

> "Goroutines su jeftine — pokretam ih za svaki mali zadatak jer nema loše posledice."

Preciznije je:

> "Goroutines imaju troškove: memorija, scheduling, lifecycle management. Treba ih koristiti kada konkurentnost zaista donosi vrednost."

---

## Pitanje 39 — Šta je data race?

### Odgovor

Data race nastaje kada više goroutines istovremeno pristupa istoj memorijskoj lokaciji, najmanje jedan pristup je upis, a pristupi nisu pravilno sinhronizovani.

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

Obe goroutines pristupaju promenljivoj `counter`. Operacija `counter++` nije atomarna — uključuje čitanje, izmenu i upis vrednosti. Bez odgovarajuće sinhronizacije rezultat nije bezbedno definisan.

Go pruža race detector koji možemo koristiti tokom razvoja i testiranja:

```bash
go test -race ./...
```

ili:

```bash
go run -race .
```

### Šta interviewer očekuje?

* Definicija data race-a.
* Primer data race-a.
* Da `counter++` nije atomarna operacija.
* Da Go ima race detector.
* Da data race može dovesti do nedefinisanog ponašanja.

### Česta greška

Nije precizno reći:

> "Ako program ne pada, nema data race-a."

Preciznije je:

> "Data race može da postoji a da program izgleda ispravno. Efekti su nedeterministički i zavisni od timing-a i hardvera."

---

## Pitanje 40 — Šta je deadlock?

### Odgovor

Deadlock nastaje kada goroutine ili grupa goroutines čeka uslov koji nikada neće biti ispunjen.

Jedan od najjednostavnijih primera:

```go
func main() {
	ch := make(chan int)

	ch <- 42

	fmt.Println("done")
}
```

Ovde `main` pokušava da pošalje vrednost na unbuffered channel. Ali nema druge goroutine koja prima tu vrednost. Zato se `main` blokira.

Pošto nema druge goroutine koja može da nastavi komunikaciju, runtime detektuje deadlock:

```text
fatal error: all goroutines are asleep - deadlock!
```

### Šta interviewer očekuje?

* Definicija deadlock-a.
* Primer deadlock-a sa channel-om.
* Da Go runtime detektuje deadlock u jednostavnim slučajevima.
* Da Go runtime ne rešava deadlock — program se prekida.

### Česta greška

Nije precizno reći:

> "Go automatski rešava deadlock i nastavlja izvršavanje."

Preciznije je:

> "Go runtime može detektovati i prijaviti deadlock, ali ga ne rešava. Program se prekida sa greškom."

---

## Pitanje 41 — Šta je goroutine leak?

### Odgovor

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

Problem postaje ozbiljniji kada aplikacija kontinuirano kreira takve goroutines:

```text
request 1 → goroutine leak
request 2 → goroutine leak
request 3 → goroutine leak
...
request N → goroutine leak
```

Vremenom broj goroutines može rasti i negativno uticati na aplikaciju. Zbog toga je lifecycle goroutine jedan od ključnih aspekata production concurrency koda.

### Šta interviewer očekuje?

* Definicija goroutine leak-a.
* Da svaka goroutine, čak i blokirana, troši resurse.
* Da lifecycle goroutine mora biti jasno definisan.
* Da je cancellation osnova za sprečavanje leak-ova.

### Česta greška

Nije precizno reći:

> "Goroutine koja čeka i ne radi ništa ne troši resurse."

Preciznije je:

> "Svaka goroutine, čak i blokirana, zauzima memoriju i runtime resurse. Akumulacija leak-ova može ozbiljno uticati na performanse aplikacije."

---

## Pitanje 42 — Kako možemo omogućiti goroutini da zna kada treba da prekine rad?

### Odgovor

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

Goroutine tada dobija signal kroz `<-ctx.Done()` i završava rad pozivanjem `return`.

Ovaj obrazac je posebno važan kod servera, request lifecycle-a i background workera.

### Šta interviewer očekuje?

* Da `context.Context` nosi cancellation signal.
* Da goroutine mora aktivno proveravati `ctx.Done()`.
* Da `cancel()` pozivalac poziva kada rad više nije potreban.
* Da `select` kombinuje cancellation sa normalnim radom.

### Česta greška

Nije precizno reći:

> "Pozivom `cancel()` goroutine se odmah prekida."

Preciznije je:

> "`cancel()` šalje signal kroz `ctx.Done()`. Goroutine sama mora proveriti taj signal i odlučiti da završi. Prekid nije trenutan."

---

## Pitanje 43 — Ko treba da zatvori channel?

### Odgovor

Uobičajeno: strana koja **šalje vrednosti** i zna da je slanje završeno.

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

Ako receiver zatvori channel dok producer još pokušava da šalje:

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

Zato je ownership channel-a važan deo API dizajna.

### Šta interviewer očekuje?

* Da sender uglavnom zatvara channel.
* Da receiver uglavnom ne treba da zatvara tuđi channel.
* Da send na zatvoreni channel izaziva panic.
* Pojam channel ownership-a.

### Česta greška

Nije precizno reći:

> "Receiver treba da zatvori channel kada završi sa čitanjem."

Preciznije je:

> "Receiver uglavnom ne zatvara channel. To je odgovornost sender-a koji kontroliše lifecycle toka podataka."

---

## Pitanje 44 — Da li receive sa zatvorenog channel-a izaziva panic?

### Odgovor

Ne.

Ako je channel zatvoren i više nema vrednosti u njemu:

```go
value, ok := <-ch
```

dobijamo:

```text
value = zero value
ok    = false
```

Primer:

```go
ch := make(chan int)

close(ch)

value, ok := <-ch

fmt.Println(value) // 0
fmt.Println(ok)    // false
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

### Šta interviewer očekuje?

* Da receive sa zatvorenog channel-a ne izaziva panic.
* Vrednosti `value` i `ok` pri tom receive-u.
* Kako `ok` pattern detektuje zatvoreni channel.
* Da `range` automatski hendluje ovo.

### Česta greška

Nije precizno reći:

> "Receive sa zatvorenog channel-a izaziva panic."

Preciznije je:

> "Receive sa zatvorenog i praznog channel-a vraća zero value i `false` za `ok` — bez panica. Panic nastaje samo pri send-u na zatvoreni channel."

---

[Prelazak na **Junior — Interview Questions**](02-junior.md)
