# Interview Questions — Junior

> **Fajl:** `extras/01-interview-questions/module-1/02-junior.md`
>
> **Nivo:** Junior
>
> **Oblast:** #1 — Concurrency Fundamentals

---

## Sadržaj

1. [Koja je suštinska razlika između buffered i unbuffered channel-a?](#pitanje-01--koja-je-suštinska-razlika-između-buffered-i-unbuffered-channel-a)
2. [Da li buffered channel znači da je komunikacija asinhrona?](#pitanje-02--da-li-buffered-channel-znači-da-je-komunikacija-asinhrona)
3. [Kada send na unbuffered channel-u blokira?](#pitanje-03--kada-send-na-unbuffered-channel-u-blokira)
4. [Kada receive na unbuffered channel-u blokira?](#pitanje-04--kada-receive-na-unbuffered-channel-u-blokira)
5. [Kada send na buffered channel-u blokira?](#pitanje-05--kada-send-na-buffered-channel-u-blokira)
6. [Kada receive na buffered channel-u blokira?](#pitanje-06--kada-receive-na-buffered-channel-u-blokira)
7. [Kako možemo saznati kapacitet channel-a?](#pitanje-07--kako-možemo-saznati-kapacitet-channel-a)
8. [Zašto postoje channel directions?](#pitanje-08--zašto-postoje-channel-directions)
9. [Da li bidirectional channel možemo proslediti funkciji koja očekuje send-only channel?](#pitanje-09--da-li-bidirectional-channel-možemo-proslediti-funkciji-koja-očekuje-send-only-channel)
10. [Šta je bolje: `chan int` ili `<-chan int` kada funkcija samo prima podatke?](#pitanje-10--šta-je-bolje-chan-int-ili--chan-int-kada-funkcija-samo-prima-podatke)
11. [Kako `range` funkcioniše kada se koristi nad channel-om?](#pitanje-11--kako-range-funkcioniše-kada-se-koristi-nad-channel-om)
12. [Da li `range` automatski zatvara channel?](#pitanje-12--da-li-range-automatski-zatvara-channel)
13. [Zašto se channel zatvara?](#pitanje-13--zašto-se-channel-zatvara)
14. [Šta se dešava kada pokušamo da primimo vrednost sa zatvorenog channel-a?](#pitanje-14--šta-se-dešava-kada-pokušamo-da-primimo-vrednost-sa-zatvorenog-channel-a)
15. [Da li `close(ch)` odmah odbacuje vrednosti koje se već nalaze u buffered channel-u?](#pitanje-15--da-li-closech-odmah-odbacuje-vrednosti-koje-se-već-nalaze-u-buffered-channel-u)
16. [Ko je odgovoran za zatvaranje channel-a?](#pitanje-16--ko-je-odgovoran-za-zatvaranje-channel-a)
17. [Zašto receiver uglavnom ne treba da zatvara channel?](#pitanje-17--zašto-receiver-uglavnom-ne-treba-da-zatvara-channel)
18. [Koji praktični princip možemo koristiti kada odlučujemo ko zatvara channel?](#pitanje-18--koji-praktični-princip-možemo-koristiti-kada-odlučujemo-ko-zatvara-channel)
19. [Šta znači channel ownership?](#pitanje-19--šta-znači-channel-ownership)
20. [Zašto vraćati `<-chan T` umesto `chan T`?](#pitanje-20--zašto-vraćati--chan-t-umesto-chan-t)
21. [Koji je idiomatski način da producer signalizira consumer-u da je završio proizvodnju?](#pitanje-21--koji-je-idiomatski-način-da-producer-signalizira-consumer-u-da-je-završio-proizvodnju)
22. [Šta se dešava ako koristimo `range` nad channel-om koji nikada neće biti zatvoren?](#pitanje-22--šta-se-dešava-ako-koristimo-range-nad-channel-om-koji-nikada-neće-biti-zatvoren)
23. [Šta se dešava ako dva različita dela programa pokušaju da zatvore isti channel?](#pitanje-23--šta-se-dešava-ako-dva-različita-dela-programa-pokušaju-da-zatvore-isti-channel)
24. [Kada send operacija na channel-u blokira goroutine?](#pitanje-24--kada-send-operacija-na-channel-u-blokira-goroutine)
25. [Kada receive operacija blokira goroutine?](#pitanje-25--kada-receive-operacija-blokira-goroutine)
26. [Da li je svako blokiranje goroutine deadlock?](#pitanje-26--da-li-je-svako-blokiranje-goroutine-deadlock)
27. [Šta će se desiti u programu sa send-om na unbuffered channel bez receiver-a?](#pitanje-27--šta-će-se-desiti-u-programu-sa-send-om-na-unbuffered-channel-bez-receiver-a)
28. [Kako možemo popraviti deadlock izazvan slanjem na unbuffered channel?](#pitanje-28--kako-možemo-popraviti-deadlock-izazvan-slanjem-na-unbuffered-channel)
29. [Može li receive sam po sebi izazvati deadlock?](#pitanje-29--može-li-receive-sam-po-sebi-izazvati-deadlock)
30. [Zašto redosled send/receive operacija može biti važan?](#pitanje-30--zašto-redosled-sendreceive-operacija-može-biti-važan)
31. [Šta znači da unbuffered channel predstavlja rendezvous između goroutines?](#pitanje-31--šta-znači-da-unbuffered-channel-predstavlja-rendezvous-između-goroutines)
32. [Šta se događa kada buffered channel postane pun?](#pitanje-32--šta-se-događa-kada-buffered-channel-postane-pun)
33. [Da li povećavanje kapaciteta buffered channel-a rešava problem sporog consumer-a?](#pitanje-33--da-li-povećavanje-kapaciteta-buffered-channel-a-rešava-problem-sporog-consumer-a)
34. [Kako dve goroutines mogu međusobno da čekaju jedna drugu?](#pitanje-34--kako-dve-goroutines-mogu-međusobno-da-čekaju-jedna-drugu)
35. [Šta je circular wait u concurrency programu?](#pitanje-35--šta-je-circular-wait-u-concurrency-programu)
36. [Koja je razlika između deadlock-a i starvation-a?](#pitanje-36--koja-je-razlika-između-deadlock-a-i-starvation-a)
37. [Da li je goroutine leak isto što i deadlock?](#pitanje-37--da-li-je-goroutine-leak-isto-što-i-deadlock)
38. [Šta znači zatvoriti channel u Go-u?](#pitanje-38--šta-znači-zatvoriti-channel-u-go-u)
39. [Da li receiver treba da zatvara channel?](#pitanje-39--da-li-receiver-treba-da-zatvara-channel)
40. [Šta se događa ako pokušamo da pošaljemo vrednost na zatvoren channel?](#pitanje-40--šta-se-događa-ako-pokušamo-da-pošaljemo-vrednost-na-zatvoren-channel)
41. [Šta se događa kada pokušamo da primimo vrednost sa zatvorenog channel-a?](#pitanje-41--šta-se-događa-kada-pokušamo-da-primimo-vrednost-sa-zatvorenog-channel-a)
42. [Kako možemo proveriti da li je channel zatvoren?](#pitanje-42--kako-možemo-proveriti-da-li-je-channel-zatvoren)
43. [Šta se događa ako zatvorimo buffered channel koji još ima podatke?](#pitanje-43--šta-se-događa-ako-zatvorimo-buffered-channel-koji-još-ima-podatke)
44. [Kako `range` radi sa channel-om?](#pitanje-44--kako-range-radi-sa-channel-om)
45. [Šta se događa ako channel nikada nije zatvoren pri korišćenju `range`?](#pitanje-45--šta-se-događa-ako-channel-nikada-nije-zatvoren-pri-korišćenju-range)
46. [Zašto producer često treba da zatvori channel kada završi proizvodnju?](#pitanje-46--zašto-producer-često-treba-da-zatvori-channel-kada-završi-proizvodnju)
47. [Da li `close` služi samo za sprečavanje novih send operacija?](#pitanje-47--da-li-close-služi-samo-za-sprečavanje-novih-send-operacija)
48. [Zašto se za signalizaciju često koristi `chan struct{}`?](#pitanje-48--zašto-se-za-signalizaciju-često-koristi-chan-struct)
49. [Šta se događa ako više goroutines čeka na channel koji se zatvori?](#pitanje-49--šta-se-događa-ako-više-goroutines-čeka-na-channel-koji-se-zatvori)
50. [Da li zatvaranje channel-a automatski prekida goroutine?](#pitanje-50--da-li-zatvaranje-channel-a-automatski-prekida-goroutine)
51. [Koje je praktično pravilo za ownership channel-a?](#pitanje-51--koje-je-praktično-pravilo-za-ownership-channel-a)
52. [Čemu služi `select` u Go-u?](#pitanje-52--čemu-služi-select-u-go-u)
53. [Šta se događa ako je više `case` grana spremno istovremeno?](#pitanje-53--šta-se-događa-ako-je-više-case-grana-spremno-istovremeno)
54. [Šta se događa kada nijedan `case` nije spreman u `select`-u bez `default`?](#pitanje-54--šta-se-događa-kada-nijedan-case-nije-spreman-u-select-u-bez-default)
55. [Čemu služi `default` unutar `select` statement-a?](#pitanje-55--čemu-služi-default-unutar-select-statement-a)
56. [Kako možemo pokušati da primimo vrednost bez blokiranja?](#pitanje-56--kako-možemo-pokušati-da-primimo-vrednost-bez-blokiranja)
57. [Šta je busy loop i zašto je problematičan?](#pitanje-57--šta-je-busy-loop-i-zašto-je-problematičan)
58. [Kako možemo implementirati timeout za channel operaciju?](#pitanje-58--kako-možemo-implementirati-timeout-za-channel-operaciju)
59. [Da li je `time.After` uvek najbolji izbor za timeout?](#pitanje-59--da-li-je-timeafter-uvek-najbolji-izbor-za-timeout)
60. [Da li timeout automatski prekida goroutine koja radi posao?](#pitanje-60--da-li-timeout-automatski-prekida-goroutine-koja-radi-posao)
61. [Kako možemo omogućiti worker-u da reaguje na signal za završetak?](#pitanje-61--kako-možemo-omogućiti-worker-u-da-reaguje-na-signal-za-završetak)
62. [Šta se događa ako je jedan od channel-a u `select`-u zatvoren?](#pitanje-62--šta-se-događa-ako-je-jedan-od-channel-a-u-select-u-zatvoren)
63. [Zašto zatvoreni channel može izazvati problem u `select` petlji?](#pitanje-63--zašto-zatvoreni-channel-može-izazvati-problem-u-select-petlji)
64. [Zašto se `select` često opisuje kao mehanizam za multipleksiranje channel događaja?](#pitanje-64--zašto-se-select-često-opisuje-kao-mehanizam-za-multipleksiranje-channel-događaja)
65. [Da li se svi `case` blokovi izvršavaju ako su spremni?](#pitanje-65--da-li-se-svi-case-blokovi-izvršavaju-ako-su-spremni)
66. [Možemo li jednostavnim redosledom `case` grana dati prioritet jednom channel-u?](#pitanje-66--možemo-li-jednostavnim-redosledom-case-grana-dati-prioritet-jednom-channel-u)
67. [Kada je dovoljan običan receive, a kada je potreban `select`?](#pitanje-67--kada-je-dovoljan-običan-receive-a-kada-je-potreban-select)

---

## Pitanje 01 — Koja je suštinska razlika između buffered i unbuffered channel-a?

### Odgovor

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

### Šta interviewer očekuje?

* Razliku između `make(chan int)` i `make(chan int, N)`.
* Da unbuffered channel zahteva istovremeni send i receive.
* Da buffered channel dozvoljava asinhronost do kapaciteta bafera.
* Da pun buffered channel i dalje blokira sender-a.

### Česta greška

Nije precizno reći:

> "Buffered channel nikada ne blokira."

Preciznije je:

> "Buffered channel blokira kada se buffer napuni. Buffering menja trenutak blokiranja, ali ga ne eliminiše."

---

## Pitanje 02 — Da li buffered channel znači da je komunikacija asinhrona?

### Odgovor

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

### Šta interviewer očekuje?

* Da buffered channel ne eliminiše blokiranje.
* Da send blokira kada je bafer pun.
* Da je buffered channel ograničeni queue, a ne potpuno asinhrona komunikacija.

### Česta greška

Nije precizno reći:

> "Koristim buffered channel da se nikad ne brinem o blokiranju."

Preciznije je:

> "Buffered channel odlaže blokiranje, ali ga ne eliminiše. Pun buffer i dalje blokira sender-a."

---

## Pitanje 03 — Kada send na unbuffered channel-u blokira?

### Odgovor

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

### Šta interviewer očekuje?

* Da send na unbuffered channel blokira dok nema odgovarajućeg receiver-a.
* Da unbuffered channel predstavlja tačku susreta — rendezvous.
* Da je komunikacija istovremeno prenos podataka i sinhronizacija.

### Česta greška

Nije precizno reći:

> "Send na channel uvek odmah završi."

Preciznije je:

> "Send na unbuffered channel blokira dok receiver nije spreman. Nema odlaganja — nema buffer-a."

---

## Pitanje 04 — Kada receive na unbuffered channel-u blokira?

### Odgovor

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

### Šta interviewer očekuje?

* Da receive bez sender-a blokira goroutine.
* Da blokiranje bez mogućnosti napretka dovodi do deadlock-a.
* Da je odgovornost za obe strane komunikacije važan deo dizajna.

### Česta greška

Nije precizno reći:

> "Receive sa praznog channel-a odmah vraća zero value."

Preciznije je:

> "Receive na unbuffered channel-u bez sender-a blokira zauvek. Zero value se dobija samo pri receive-u sa zatvorenog channel-a."

---

## Pitanje 05 — Kada send na buffered channel-u blokira?

### Odgovor

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

### Šta interviewer očekuje?

* Da send na buffered channel blokira kada je buffer pun.
* Da buffer popunjava i prazni po FIFO redosledu.
* Da je blokiranje privremeno — čeka se dok se buffer ne oslobodi.

### Česta greška

Nije precizno reći:

> "Buffered channel ne blokira nikada."

Preciznije je:

> "Buffered channel blokira send kada je buffer pun. Kapacitet buffer-a samo definiše koliko send-ova može da se izvrši pre blokiranja."

---

## Pitanje 06 — Kada receive na buffered channel-u blokira?

### Odgovor

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
send    → blokira kada je buffer pun
receive → blokira kada je buffer prazan
```

Ovo je jedan od najvažnijih mentalnih modela za razumevanje channel-a.

### Šta interviewer očekuje?

* Da receive blokira kada je buffer prazan i channel nije zatvoren.
* Da obe granice — pun buffer i prazan buffer — mogu izazvati blokiranje.
* Da je ovo ključni mentalni model za razumevanje channel-a.

### Česta greška

Nije precizno reći:

> "Receive na buffered channel-u nikada ne blokira."

Preciznije je:

> "Receive na buffered channel-u blokira kada je buffer prazan i nema aktivnog sender-a koji bi poslao vrednost."

---

## Pitanje 07 — Kako možemo saznati kapacitet channel-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da `cap(ch)` vraća ukupni kapacitet bafera.
* Da `len(ch)` vraća broj trenutno dostupnih elemenata.
* Da ove vrednosti nisu bezbedne za synchronization logiku u concurrent kodu.

### Česta greška

Nije precizno reći:

> "Koristim `len(ch)` da znam da li mogu da pošaljem bez blokiranja."

Preciznije je:

> "U concurrent kodu, `len(ch)` može biti zastarela informacija čim je pročitana. Nije zamena za pravilnu koordinaciju."

---

## Pitanje 08 — Zašto postoje channel directions?

### Odgovor

Go omogućava da channel u funkcionalnom API-ju ograničimo na određeni smer komunikacije.

Tri oblika su:

```go
chan T
chan<- T
<-chan T
```

`chan T` je bidirectional channel. Može da šalje i prima:

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

### Šta interviewer očekuje?

* Poznavanje sva tri tipa direction-a: `chan T`, `chan<- T`, `<-chan T`.
* Da direction ograničava kako se channel može koristiti unutar funkcije.
* Da je direction deo API dizajna, a ne samo konvencija.

### Česta greška

Nije precizno reći:

> "Channel direction je samo hint za programera — compiler ih ionako ne proverava."

Preciznije je:

> "Channel direction je deo type sistema. Compiler odbija pogrešnu upotrebu — send na receive-only channel je compile error."

---

## Pitanje 09 — Da li bidirectional channel možemo proslediti funkciji koja očekuje send-only channel?

### Odgovor

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

### Šta interviewer očekuje?

* Da bidirectional channel može biti implicitno konvertovan u directional channel pri pozivu funkcije.
* Da ovo nije eksplicitni cast, već automatska konverzija u skladu sa tipom.
* Da se na ovaj način izražava ownership kroz tip sistem.

### Česta greška

Nije precizno reći:

> "Bidirectional i send-only channel nisu kompatibilni tipovi — ne mogu se koristiti zajedno."

Preciznije je:

> "Bidirectional channel (`chan T`) može biti prosleđen tamo gde se očekuje `chan<- T` ili `<-chan T`. Obrnuto nije dozvoljeno."

---

## Pitanje 10 — Šta je bolje: `chan int` ili `<-chan int` kada funkcija samo prima podatke?

### Odgovor

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

### Šta interviewer očekuje?

* Da direction komunicira nameru funkcije čitaocu koda.
* Da compiler proverava ispravnost i sprečava greške.
* Da je precizna direction bolja praksa od bidirectional-a kada je uloga funkcije jasna.

### Česta greška

Nije precizno reći:

> "Svejedno je da li koristim `chan int` ili `<-chan int` — rezultat je isti."

Preciznije je:

> "Korišćenjem `<-chan int` jasno komuniciramo da funkcija samo prima. Compiler tada može sprečiti slučajni send unutar te funkcije."

---

## Pitanje 11 — Kako `range` funkcioniše kada se koristi nad channel-om?

### Odgovor

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

### Šta interviewer očekuje?

* Sintaksu `for value := range ch`.
* Da `range` blokira između vrednosti dok channel nije zatvoren.
* Da `range` završava tek kada je channel eksplicitno zatvoren sa `close` i sve vrednosti pročitane.
* Veza između `range` i `close`.

### Česta greška

Nije precizno reći:

> "`range` nad channel-om završava kada channel ostane prazan."

Preciznije je:

> "`range` završava tek kada je channel eksplicitno zatvoren sa `close`. Bez `close` petlja ostaje blokirana zauvek."

---

## Pitanje 12 — Da li `range` automatski zatvara channel?

### Odgovor

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

### Šta interviewer očekuje?

* Da `range` samo čita — ne zatvara channel.
* Da `close` mora biti eksplicitno pozvan od strane sender-a.
* Da sender koji kontroliše kraj slanja obično zatvara channel.

### Česta greška

Nije precizno reći:

> "Receiver zatvara channel kada završi sa čitanjem pomoću `range`."

Preciznije je:

> "`range` ne zatvara channel. Sender koji zna da je slanje završeno je odgovoran za `close`."

---

## Pitanje 13 — Zašto se channel zatvara?

### Odgovor

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

`close` nije poruka koja predstavlja jednu dodatnu poslovnu vrednost. On menja stanje channel-a i omogućava receiver-u da sazna da je stream završen.

### Šta interviewer očekuje?

* Da `close` signalizira kraj toka podataka.
* Da `close` nije vrednost — već promena stanja channel-a.
* Da receiver može pouzdano detektovati zatvaranje kroz `range` ili `value, ok := <-ch`.

### Česta greška

Nije precizno reći:

> "Channel se zatvara da bi se oslobodila memorija."

Preciznije je:

> "`close` je signal receiver-ima da više neće biti novih vrednosti. On nije mehanizam za oslobađanje resursa — to radi garbage collector."

---

## Pitanje 14 — Šta se dešava kada pokušamo da primimo vrednost sa zatvorenog channel-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da receive sa zatvorenog i praznog channel-a ne izaziva panic.
* Da se vraća zero value uz `ok == false`.
* Da `value, ok := <-ch` omogućava detekciju zatvorenog channel-a.
* Da `range` automatski hendluje ovo.

### Česta greška

Nije precizno reći:

> "Receive sa zatvorenog channel-a izaziva panic."

Preciznije je:

> "Receive sa zatvorenog i praznog channel-a vraća zero value i `false` za `ok` — bez panica. Panic nastaje samo pri send-u na zatvoreni channel."

---

## Pitanje 15 — Da li `close(ch)` odmah odbacuje vrednosti koje se već nalaze u buffered channel-u?

### Odgovor

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
        ┌────────────────┐
        │ channel closed │
        └───────┬────────┘
                │
    postoje buffered vrednosti?
          /           \
        DA             NE
        │               │
        ▼               ▼
     čitaj ih      receive dobija
     normalno    zero value + false
```

Ovo je posebno važno kod producer/consumer obrazaca.

### Šta interviewer očekuje?

* Da `close` ne briše buffered vrednosti.
* Da buffered vrednosti mogu biti pročitane i nakon `close`.
* Da zero value uz `ok == false` dolazi tek kada su sve buffered vrednosti iscrpljene.

### Česta greška

Nije precizno reći:

> "`close(ch)` odmah onemogućava sve receive operacije."

Preciznije je:

> "`close(ch)` samo signalizira kraj slanja. Sve prethodno buffered vrednosti mogu se normalno pročitati. Tek kada se buffer isprazni, receive vraća zero value uz `ok == false`."

---

## Pitanje 16 — Ko je odgovoran za zatvaranje channel-a?

### Odgovor

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

Consumer nema potrebu da zatvara channel. Ovakav model smanjuje mogućnost greške jer je ownership jasan.

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

## Pitanje 17 — Zašto receiver uglavnom ne treba da zatvara channel?

### Odgovor

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

### Šta interviewer očekuje?

* Da receiver najčešće ne zna kada su svi sender-i završili.
* Da zatvaranje channel-a od strane receiver-a može izazvati panic u aktivnom sender-u.
* Da ownership mora biti jasan pre nego što se donese odluka o zatvaranju.

### Česta greška

Nije precizno reći:

> "Receiver može zatvoriti channel kada mu više ne trebaju podaci."

Preciznije je:

> "Ako receiver zatvori channel dok sender još šalje, sender će doživeti panic. Receiver uglavnom nije u poziciji da zna kada su svi sender-i završili."

---

## Pitanje 18 — Koji praktični princip možemo koristiti kada odlučujemo ko zatvara channel?

### Odgovor

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

### Šta interviewer očekuje?

* Princip: sender zatvara channel.
* Da ownership treba biti jasan pre nego što se donese odluka o `close`.
* Da `defer close(ch)` u producer-u je uobičajen idiom.

### Česta greška

Nije precizno reći:

> "Ko god završi prvi — on zatvori channel."

Preciznije je:

> "Channel treba da zatvori onaj ko kontroliše tok podataka i zna kada je slanje završeno. Najčešće je to producer."

---

## Pitanje 19 — Šta znači channel ownership?

### Odgovor

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

Ovo je snažan API dizajn jer ownership nije samo dokumentovan komentarom — on je delimično izražen kroz Go type sistem.

### Šta interviewer očekuje?

* Da ownership obuhvata kreiranje, slanje, primanje i zatvaranje channel-a.
* Da se ownership može delimično izraziti kroz directional types.
* Da nejasan ownership vodi ka greškama kao što su double-close ili send na zatvoren channel.

### Česta greška

Nije precizno reći:

> "Ownership channel-a nije bitan — svako može da uradi šta treba."

Preciznije je:

> "Nejasan channel ownership je čest uzrok panic-a i goroutine leak-ova. Ownership treba biti eksplicitno definisan i po mogućnosti izražen kroz type sistem."

---

## Pitanje 20 — Zašto vraćati `<-chan T` umesto `chan T`?

### Odgovor

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

### Šta interviewer očekuje?

* Da `<-chan T` ograničava caller-a na receive operacije.
* Da ovo sprečava slučajno slanje ili zatvaranje channel-a od strane caller-a.
* Da je ovo primer principa minimalnih privilegija u API dizajnu.

### Česta greška

Nije precizno reći:

> "Svejedno je da li vraćamo `chan T` ili `<-chan T` — caller može ionako uraditi šta hoće."

Preciznije je:

> "Vraćanjem `<-chan T` compiler sprečava caller-a da pošalje ili zatvori channel. To su compile error-i, ne runtime greške."

---

## Pitanje 21 — Koji je idiomatski način da producer signalizira consumer-u da je završio proizvodnju?

### Odgovor

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

### Šta interviewer očekuje?

* Da `defer close(ch)` u producer-u je standardni obrazac.
* Da `range` na strani consumer-a automatski završava po `close`.
* Da je ownership i lifecycle jasan: producer kreira, šalje i zatvara.

### Česta greška

Nije precizno reći:

> "Consumer treba da provjeri svaku vrednost da vidi da li je signal za kraj."

Preciznije je:

> "Idiomatski Go koristi `close(ch)` kao signal završetka i `range` za automatsku detekciju. Nema potrebe za sentinel vrednostima."

---

## Pitanje 22 — Šta se dešava ako koristimo `range` nad channel-om koji nikada neće biti zatvoren?

### Odgovor

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

* blokiranih goroutines;
* goroutine leak-a;
* deadlock-a u zavisnosti od ostatka programa;
* testova koji vise;
* problema pri graceful shutdown-u.

### Šta interviewer očekuje?

* Da `range` bez `close` može ostati blokiran zauvek.
* Da je ovo jedan od najčešćih uzroka goroutine leak-a.
* Da producer mora eksplicitno pozvati `close` kada je završio.

### Česta greška

Nije precizno reći:

> "`range` završava kada producer prestane da šalje vrednosti."

Preciznije je:

> "`range` završava samo kada je channel zatvoren. Pauza u slanju nije ista stvar kao zatvaranje — `range` čeka zauvek ako channel nije zatvoren."

---

## Pitanje 23 — Šta se dešava ako dva različita dela programa pokušaju da zatvore isti channel?

### Odgovor

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

### Šta interviewer očekuje?

* Da dvostruko zatvaranje channel-a izaziva panic.
* Da ownership nad channel lifecycle-om mora biti jedinstven.
* Da kod više sender-a treba koristiti koordinatora (npr. `sync.WaitGroup`).

### Česta greška

Nije precizno reći:

> "Drugi `close` je bezopasan — channel je već zatvoren."

Preciznije je:

> "Drugi poziv `close` na već zatvorenom channel-u izaziva panic. Zato ownership i odgovornost za `close` moraju biti jasni."

---

## Pitanje 24 — Kada send operacija na channel-u blokira goroutine?

### Odgovor

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

### Šta interviewer očekuje?

* Da send blokira kada channel ne može prihvatiti vrednost.
* Razliku između unbuffered i buffered ponašanja.
* Da buffer kapacitet određuje koliko send-ova može proći bez blokiranja.

### Česta greška

Nije precizno reći:

> "Send uvek odmah završi."

Preciznije je:

> "Send blokira kada unbuffered channel nema receiver-a, ili kada je buffer buffered channel-a pun."

---

## Pitanje 25 — Kada receive operacija blokira goroutine?

### Odgovor

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

### Šta interviewer očekuje?

* Da receive blokira kada nema dostupne vrednosti.
* Razliku između unbuffered (čeka sender-a) i buffered (čeka dok buffer nije prazan).
* Da receive sa zatvorenog i praznog channel-a ne blokira — vraća zero value.

### Česta greška

Nije precizno reći:

> "Receive uvek odmah dobija vrednost."

Preciznije je:

> "Receive blokira dok vrednost ne postane dostupna. Na unbuffered channel-u to znači čekanje na sender-a."

---

## Pitanje 26 — Da li je svako blokiranje goroutine deadlock?

### Odgovor

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

Deadlock nastaje kada goroutines čekaju događaj koji se više nikada neće dogoditi.

Dakle:

```text
blokiranje
    │
    ├── očekivano i privremeno → normalno
    │
    └── niko ne može napraviti
        očekivani progress → deadlock
```

Ovo je jedna od najvažnijih razlika koju treba razumeti.

### Šta interviewer očekuje?

* Da blokiranje nije isto što i deadlock.
* Da je privremeno blokiranje normalan deo concurrency modela.
* Da deadlock nastaje kada nijedna goroutine ne može napredovati.

### Česta greška

Nije precizno reći:

> "Svako blokiranje je deadlock."

Preciznije je:

> "Blokiranje je privremeno čekanje — goroutine čeka uslov koji će biti ispunjen. Deadlock nastaje kada taj uslov nikada neće biti ispunjen."

---

## Pitanje 27 — Šta će se desiti u programu sa send-om na unbuffered channel bez receiver-a?

### Odgovor

Program će deadlock-ovati.

Razlog je što je `ch` unbuffered channel, a `main` goroutine pokušava da pošalje vrednost bez postojanja receiver-a.

```go
func main() {
	ch := make(chan int)

	ch <- 42

	fmt.Println("done")
}
```

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

Go runtime detektuje situaciju u kojoj su sve goroutines blokirane i može prijaviti:

```text
fatal error: all goroutines are asleep - deadlock!
```

### Šta interviewer očekuje?

* Da send na unbuffered channel bez receiver-a blokira goroutine.
* Da blokiranje bez mogućnosti napretka dovodi do deadlock-a.
* Da Go runtime detektuje i prijavljuje ovaj deadlock.

### Česta greška

Nije precizno reći:

> "Program nastavlja jer `main` nije blokiran."

Preciznije je:

> "`main` goroutine blokira na `ch <- 42`. Nema druge goroutine koja bi izvršila receive, pa runtime detektuje deadlock."

---

## Pitanje 28 — Kako možemo popraviti deadlock izazvan slanjem na unbuffered channel?

### Odgovor

Jedna mogućnost je pokretanje receiver-a u drugoj goroutine:

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

### Šta interviewer očekuje?

* Da pokretanje receiver-a u goroutine rešava problem.
* Da buffered channel može eliminisati rendezvous zahtev.
* Da buffered channel nije generalno rešenje za sve deadlock situacije.

### Česta greška

Nije precizno reći:

> "Uvek koristi buffered channel da izbegneš deadlock."

Preciznije je:

> "Buffered channel menja uslove blokiranja, ali ne rešava fundamentalne probleme dizajna. Deadlock može nastati i sa buffered channel-om ako je buffer pun i nema receiver-a."

---

## Pitanje 29 — Može li receive sam po sebi izazvati deadlock?

### Odgovor

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

Ovo je suprotna varijanta deadlock-a na send:

```text
send bez receiver-a
        ↓
    deadlock

receive bez sender-a
        ↓
    deadlock
```

### Šta interviewer očekuje?

* Da receive bez sender-a blokira goroutine.
* Da blokiranje bez izlaza dovodi do deadlock-a.
* Da obe strane komunikacije moraju biti prisutne.

### Česta greška

Nije precizno reći:

> "Receive nikada ne može izazvati deadlock — on samo čeka."

Preciznije je:

> "Receive bez sender-a blokira zauvek. Ako je to jedina aktivna goroutine, runtime detektuje deadlock."

---

## Pitanje 30 — Zašto redosled send/receive operacija može biti važan?

### Odgovor

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

### Šta interviewer očekuje?

* Da sekvencijalni send i receive na unbuffered channel-u u istoj goroutine dovode do deadlock-a.
* Da sender mora biti u zasebnoj goroutine da bi rendezvous bio moguć.
* Da redosled operacija mora biti dizajniran u skladu sa semantikom channel-a.

### Česta greška

Nije precizno reći:

> "Send i receive mogu biti bilo gde u kodu — Go to automatski sinhronizuje."

Preciznije je:

> "Send i receive moraju biti u stanju da se izvrše istovremeno. Ako su sekvencijalni u istoj goroutine, send blokira i receive se nikada ne dostiže."

---

## Pitanje 31 — Šta znači da unbuffered channel predstavlja rendezvous između goroutines?

### Odgovor

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
* koordinaciju između goroutines.

### Šta interviewer očekuje?

* Da rendezvous znači da sender i receiver moraju biti istovremeno aktivni.
* Da unbuffered channel kombinuje prenos podataka i sinhronizaciju.
* Da je ovo fundamentalna razlika od buffered channel-a.

### Česta greška

Nije precizno reći:

> "Unbuffered channel je isti kao buffered sa kapacitetom 0 — samo sporiji."

Preciznije je:

> "Unbuffered channel nije sporiji buffered channel. On ima drugačiju semantiku — send i receive su tačka susreta, a sama komunikacija je akt sinhronizacije."

---

## Pitanje 32 — Šta se događa kada buffered channel postane pun?

### Odgovor

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
│    buffer   │
│   [1] [2]   │
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

### Šta interviewer očekuje?

* Da pun buffer blokira sender-a.
* Pojam backpressure-a — prirodno ograničavanje brzine producer-a.
* Da buffer nije beskonačan red.

### Česta greška

Nije precizno reći:

> "Ako buffer postane pun, vrednosti se gube."

Preciznije je:

> "Pun buffer blokira sender-a — vrednosti se ne gube, sender čeka dok se ne oslobodi prostor."

---

## Pitanje 33 — Da li povećavanje kapaciteta buffered channel-a rešava problem sporog consumer-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da veći buffer odlaže, ali ne eliminiše blokiranje.
* Da je backpressure — ne buffer size — pravo rešenje za sporog consumer-a.
* Da buffer treba biti dimenzionisan na osnovu konkretnih potreba, ne kao "fail-safe".

### Česta greška

Nije precizno reći:

> "Dovoljno veliki buffer uvek rešava problem sporog consumer-a."

Preciznije je:

> "Veći buffer samo odlaže blokiranje. Ako producer trajno nadmašuje consumer, buffer će se popuniti bez obzira na veličinu."

---

## Pitanje 34 — Kako dve goroutines mogu međusobno da čekaju jedna drugu?

### Odgovor

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

Ali obe goroutines pokušavaju prvo da izvrše send.

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

### Šta interviewer očekuje?

* Razumevanje circular wait-a kao uzroka deadlock-a.
* Da oba send-a blokiraju jer nema odgovarajućih receive-ova.
* Da redosled operacija mora biti dizajniran da izbegne cikličnu zavisnost.

### Česta greška

Nije precizno reći:

> "Dve goroutines koje šalju jednovremeno nikad ne mogu deadlock-ovati."

Preciznije je:

> "Ako goroutines šalju na unbuffered channel-e pri čemu svaka čeka drugu da uradi receive, nastaje circular wait i deadlock."

---

## Pitanje 35 — Šta je circular wait u concurrency programu?

### Odgovor

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

Kod channel-a ovo može nastati kada goroutines imaju pogrešno definisan redosled komunikacije.

Kod većih sistema circular wait može biti mnogo teže uočiti jer se lanac proteže kroz više komponenti.

### Šta interviewer očekuje?

* Definiciju circular wait-a.
* Da circular wait je jedan od uzroka deadlock-a.
* Da se teže detektuje u složenim sistemima sa više komponenti.

### Česta greška

Nije precizno reći:

> "Circular wait može uvek da se reši bez promene dizajna."

Preciznije je:

> "Circular wait zahteva promenu redosleda operacija ili uvođenje posrednika koji narušava krug zavisnosti."

---

## Pitanje 36 — Koja je razlika između deadlock-a i starvation-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da deadlock znači da nijedna goroutine ne može napredovati.
* Da starvation znači da jedna goroutine sistemski ne dobija resurse.
* Da su to različiti problemi sa različitim rešenjima.

### Česta greška

Nije precizno reći:

> "Deadlock i starvation su isti problem."

Preciznije je:

> "Deadlock je potpuna blokada sistema. Starvation je nefer raspodela resursa gde jedna goroutine trajno ne dobija šansu, ali sistem u celini napreduje."

---

## Pitanje 37 — Da li je goroutine leak isto što i deadlock?

### Odgovor

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

### Šta interviewer očekuje?

* Da goroutine leak i deadlock nisu sinonimi.
* Da goroutine leak može postojati bez da program deadlock-uje.
* Da lifecycle goroutine mora biti eksplicitno definisan.

### Česta greška

Nije precizno reći:

> "Goroutine leak uvek uzrokuje deadlock."

Preciznije je:

> "Goroutine leak znači da goroutine troši resurse duže nego što treba. Program može nastaviti da radi — leak ne mora odmah uzrokovati deadlock."

---

## Pitanje 38 — Šta znači zatvoriti channel u Go-u?

### Odgovor

Channel se zatvara pomoću ugrađene funkcije:

```go
close(ch)
```

Zatvaranje channel-a znači da sender signalizira:

> **"Više neće biti novih vrednosti koje će biti poslate kroz ovaj channel."**

Važno je razumeti da `close`:

* ne briše već poslate vrednosti;
* ne zaustavlja automatski receiver-e;
* ne prekida goroutines;
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

### Šta interviewer očekuje?

* Da `close` signalizira kraj slanja, ne briše vrednosti.
* Da buffered vrednosti ostaju dostupne i posle `close`.
* Da `close` ne prekida goroutines — one moraju same proveriti signal.

### Česta greška

Nije precizno reći:

> "`close(ch)` briše sve vrednosti i onemogućava dalji pristup channel-u."

Preciznije je:

> "`close(ch)` samo signalizira kraj slanja. Sve prethodno buffered vrednosti se mogu normalno pročitati."

---

## Pitanje 39 — Da li receiver treba da zatvara channel?

### Odgovor

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

### Šta interviewer očekuje?

* Da receiver uglavnom ne treba da zatvara channel.
* Da sender ima informaciju o tome kada je slanje završeno.
* Da zatvaranje od strane receiver-a može izazvati panic u sender-u.

### Česta greška

Nije precizno reći:

> "Receiver treba da zatvori channel kada ne želi više vrednosti."

Preciznije je:

> "Receiver uglavnom ne treba da zatvara channel. To je odgovornost sender-a koji zna kada je tok podataka završen."

---

## Pitanje 40 — Šta se događa ako pokušamo da pošaljemo vrednost na zatvoren channel?

### Odgovor

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

Ako više goroutines može da zatvara isti channel, postoji rizik od:

```text
panic: close of closed channel
```

Zbog toga ownership nad channel lifecycle-om mora biti jasan.

### Šta interviewer očekuje?

* Da send na zatvoren channel izaziva panic — runtime greška.
* Da double-close isto izaziva panic.
* Da jasan ownership sprečava ove greške.

### Česta greška

Nije precizno reći:

> "Send na zatvoren channel vraća grešku."

Preciznije je:

> "Send na zatvoren channel izaziva panic — ne vraća grešku. Zato ownership i životni ciklus channel-a moraju biti jasni."

---

## Pitanje 41 — Šta se događa kada pokušamo da primimo vrednost sa zatvorenog channel-a?

### Odgovor

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

Za `int` je zero value `0`. Za `string`: `""`. Za pointer: `nil`.

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

### Šta interviewer očekuje?

* Da receive sa zatvorenog channel-a ne izaziva panic.
* Da se vraća zero value odgovarajućeg tipa.
* Da sama zero value vrednost ne govori da li je channel zatvoren.

### Česta greška

Nije precizno reći:

> "Receive sa zatvorenog channel-a izaziva panic."

Preciznije je:

> "Receive sa zatvorenog i praznog channel-a vraća zero value i `false` za `ok` — bez panica. Panic nastaje samo pri send-u na zatvoreni channel."

---

## Pitanje 42 — Kako možemo proveriti da li je channel zatvoren?

### Odgovor

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
ok    = false
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

### Šta interviewer očekuje?

* Sintaksu `value, ok := <-ch`.
* Da `ok == false` znači channel je zatvoren i ispražnjen.
* Da zero value sama po sebi nije signal zatvaranja.

### Česta greška

Nije precizno reći:

> "Proverim da li je vrednost zero value da bih znao da je channel zatvoren."

Preciznije je:

> "Zero value može biti legitimna poslovna vrednost. Jedini pouzdan način detekcije je `value, ok := <-ch` i provjera `ok`."

---

## Pitanje 43 — Šta se događa ako zatvorimo buffered channel koji još ima podatke?

### Odgovor

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

### Šta interviewer očekuje?

* Da `close` na buffered channel-u sa podacima ne briše te podatke.
* Da receiver može normalno pročitati sve buffered vrednosti posle `close`.
* Da `range` završava tek kada su i channel zatvoren i buffer prazan.

### Česta greška

Nije precizno reći:

> "`close` odmah čini channel nedostupnim za čitanje."

Preciznije je:

> "`close` samo označava kraj slanja. Buffered vrednosti ostaju dostupne i mogu se normalno pročitati."

---

## Pitanje 44 — Kako `range` radi sa channel-om?

### Odgovor

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

### Šta interviewer očekuje?

* Sintaksu `for value := range ch`.
* Da `range` blokira između vrednosti.
* Da `range` završava kada je channel zatvoren i sve vrednosti pročitane.

### Česta greška

Nije precizno reći:

> "`range` nad channel-om završava kada channel ostane prazan."

Preciznije je:

> "`range` završava tek kada je channel eksplicitno zatvoren sa `close`. Bez `close` petlja ostaje blokirana zauvek."

---

## Pitanje 45 — Šta se događa ako channel nikada nije zatvoren pri korišćenju `range`?

### Odgovor

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

### Šta interviewer očekuje?

* Da `range` bez `close` može ostati blokiran zauvek.
* Da je ovo jedan od najčešćih uzroka goroutine leak-a.
* Da producer mora eksplicitno pozvati `close` kada je završio.

### Česta greška

Nije precizno reći:

> "`range` završava kada producer prestane da šalje vrednosti."

Preciznije je:

> "`range` završava samo kada je channel zatvoren. Tišina na channel-u nije ista stvar kao `close`."

---

## Pitanje 46 — Zašto producer često treba da zatvori channel kada završi proizvodnju?

### Odgovor

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

### Šta interviewer očekuje?

* Da producer ima informaciju o kraju toka podataka.
* Da `defer close(out)` je standardni idiom u producer funkcijama.
* Da `close` od strane producer-a omogućava `range` termination na strani consumer-a.

### Česta greška

Nije precizno reći:

> "Consumer može sam da zaključi kada je producer završio."

Preciznije je:

> "Consumer ne zna kada je producer završio osim ako producer eksplicitno ne pozove `close`. Zato je producer odgovoran za `close`."

---

## Pitanje 47 — Da li `close` služi samo za sprečavanje novih send operacija?

### Odgovor

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

Ovaj obrazac je koristan kada više goroutines treba da bude obavešteno o završetku određenog događaja.

### Šta interviewer očekuje?

* Da `close` može biti korišćen kao broadcast signal.
* Obrazac `done := make(chan struct{})` + `close(done)` + `<-done`.
* Da channel služi i kao sinhronizacioni mehanizam, ne samo za prenos vrednosti.

### Česta greška

Nije precizno reći:

> "`close` se koristi samo da zaustavi `range` petlje."

Preciznije je:

> "`close` je opštiji signal završetka. Može biti korišćen i bez `range` — kao broadcast signal svim goroutines koje čekaju na tom channel-u."

---

## Pitanje 48 — Zašto se za signalizaciju često koristi `chan struct{}`?

### Odgovor

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

### Šta interviewer očekuje?

* Da `struct{}` zauzima nula bajtova memorije.
* Da `chan struct{}` semantički jasno komunicira da je channel signal, a ne prenos podataka.
* Da je ovo standardni idiom za signalizaciju u Go-u.

### Česta greška

Nije precizno reći:

> "Koristim `chan bool` za signalizaciju jer `struct{}` izgleda čudno."

Preciznije je:

> "`chan struct{}` je idiomatski Go za signalizaciju. `struct{}` zauzima 0 bajtova i jasno komunicira namenu — signal, ne podatak."

---

## Pitanje 49 — Šta se događa ako više goroutines čeka na channel koji se zatvori?

### Odgovor

Sve goroutines koje čekaju receive mogu biti probuđene kada channel bude zatvoren.

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

Jedan send ne znači automatski da će sve čekajuće goroutines dobiti isti signal.

`close(done)` može signalizirati završetak svim receiver-ima koji čekaju na taj channel.

### Šta interviewer očekuje?

* Da `close` na channel-u deluje kao broadcast signal.
* Da slanje jedne vrednosti nije broadcast — prima je samo jedna goroutine.
* Da je ovo fundamentalna razlika između `close` i `send`.

### Česta greška

Nije precizno reći:

> "Slanjem vrednosti na channel obavešćavamo sve goroutines."

Preciznije je:

> "Jedan send prima samo jedna goroutine. `close` je broadcast — sve goroutines koje čekaju receive biće probuđene."

---

## Pitanje 50 — Da li zatvaranje channel-a automatski prekida goroutine?

### Odgovor

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
3. izlaznu putanju iz goroutine.

U većim sistemima ovo se često rešava pomoću `context.Context`.

### Šta interviewer očekuje?

* Da `close` ne prekida goroutine automatski.
* Da goroutine mora aktivno proveravati signal kroz `select` ili `<-done`.
* Da je `context.Context` standardni mehanizam za cancellation u produkcijskom kodu.

### Česta greška

Nije precizno reći:

> "Pozivom `close(done)` goroutine se odmah prekida."

Preciznije je:

> "`close(done)` šalje signal. Goroutine sama mora proveriti taj signal i odlučiti da završi. Prekid nije trenutan."

---

## Pitanje 51 — Koje je praktično pravilo za ownership channel-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da ownership obuhvata kreiranje, slanje, primanje i zatvaranje.
* Da directional types pomažu da se ownership izrazi kroz type sistem.
* Da jasno definisan ownership smanjuje mogućnost grešaka kao double-close ili send na zatvoren channel.

### Česta greška

Nije precizno reći:

> "Ownership channel-a nije bitan — samo treba paziti da se ne zatvori dvaput."

Preciznije je:

> "Jasan ownership je jedini način da se garantovano izbegnu double-close i send-after-close greške. Type sistem (directional channels) je alat, ali nije dovoljan sam po sebi."

---

## Pitanje 52 — Čemu služi `select` u Go-u?

### Odgovor

`select` omogućava goroutini da čeka na više channel operacija istovremeno.

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

## Pitanje 53 — Šta se događa ako je više `case` grana spremno istovremeno?

### Odgovor

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

### Šta interviewer očekuje?

* Da `select` bira pseudo-nasumično kada je više case-ova spremno.
* Da redosled case-ova ne garantuje prioritet.
* Da program ne sme zavisiti od pretpostavljenog redosleda selekcije.

### Česta greška

Nije precizno reći:

> "`select` uvek bira case koji je naveden prvi."

Preciznije je:

> "Kada je više case-ova istovremeno dostupno, `select` bira nasumično. Redosled case-ova ne implicira prioritet."

---

## Pitanje 54 — Šta se događa kada nijedan `case` nije spreman u `select`-u bez `default`?

### Odgovor

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

### Šta interviewer očekuje?

* Da `select` bez `default` blokira kada nijedan case nije spreman.
* Da je ovo namerno ponašanje — goroutine čeka na događaj.
* Razliku između blokirajućeg i neblokrajućeg `select`-a.

### Česta greška

Nije precizno reći:

> "`select` bez `default` izaziva grešku ako nijedan case nije spreman."

Preciznije je:

> "`select` bez `default` legitimno blokira goroutine dok neki case ne postane spreman. To je normalan i željeni mehanizam čekanja."

---

## Pitanje 55 — Čemu služi `default` unutar `select` statement-a?

### Odgovor

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

### Šta interviewer očekuje?

* Da `default` omogućava non-blocking select.
* Da se `default` izvršava kada nijedan drugi case nije spreman.
* Da `default` u beskonačnoj petlji može izazvati busy loop.

### Česta greška

Nije precizno reći:

> "`default` se izvršava ako su svi case-ovi blokiranji duže od nekog vremena."

Preciznije je:

> "`default` se izvršava odmah ako nijedan case nije spreman u tom trenutku. Nema nikakve pauze ili čekanja."

---

## Pitanje 56 — Kako možemo pokušati da primimo vrednost bez blokiranja?

### Odgovor

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

### Šta interviewer očekuje?

* Da `select` sa `default` omogućava non-blocking receive.
* Kada je ovakav obrazac koristan i kada nije.
* Rizik od busy loop-a u beskonačnoj petlji.

### Česta greška

Nije precizno reći:

> "Non-blocking receive uvek je bolja opcija od blokirajućeg."

Preciznije je:

> "Non-blocking receive ima smisla kada goroutine ima šta da radi i bez podataka. U beskonačnoj petlji bez alternativnog rada postaje busy loop koji troši CPU."

---

## Pitanje 57 — Šta je busy loop i zašto je problematičan?

### Odgovor

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

### Šta interviewer očekuje?

* Da busy loop nepotrebno troši CPU jer ne postoji tačka blokiranja.
* Da `default` u petlji bez alternativnog rada izaziva busy loop.
* Da blokirajući `select` je ispravniji izbor kada goroutine čeka na događaj.

### Česta greška

Nije precizno reći:

> "Busy loop je uvek brži jer nema čekanja."

Preciznije je:

> "Busy loop troši CPU i može usporiti ostatak sistema. Blokirajući `select` je efikasniji jer goroutine prepušta CPU scheduler-u dok čeka."

---

## Pitanje 58 — Kako možemo implementirati timeout za channel operaciju?

### Odgovor

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

### Šta interviewer očekuje?

* Obrazac `select` + `time.After` za timeout.
* Da `time.After` vraća channel koji prima vrednost nakon isteka vremena.
* Da je timeout nezavisan od bilo koje goroutine koja radi posao.

### Česta greška

Nije precizno reći:

> "Timeout automatski prekida goroutine koja radi posao."

Preciznije je:

> "Timeout samo prekida čekanje u `select`-u. Worker goroutine može nastaviti da radi — timeout i cancellation su različite stvari."

---

## Pitanje 59 — Da li je `time.After` uvek najbolji izbor za timeout?

### Odgovor

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

### Šta interviewer očekuje?

* Da `time.After` je prikladan za jednokratni timeout.
* Da `time.NewTimer` daje eksplicitniju kontrolu i mogućnost `Stop`.
* Da u petljama `time.After` može akumulirati timer goroutines.

### Česta greška

Nije precizno reći:

> "`time.After` je uvek ekvivalent `time.NewTimer`."

Preciznije je:

> "`time.After` kreira timer koji se ne može zaustaviti pre isteka. U petljama ovo može akumulirati neopozive timer goroutines. `time.NewTimer` sa `defer timer.Stop()` je precizniji."

---

## Pitanje 60 — Da li timeout automatski prekida goroutine koja radi posao?

### Odgovor

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

### Šta interviewer očekuje?

* Da timeout i cancellation su različiti koncepti.
* Da timeout samo prekida čekanje, ne worker goroutine.
* Da je `context.Context` standardni mehanizam za kombinovanje timeout-a i cancellation-a.

### Česta greška

Nije precizno reći:

> "Timeout automatski otkazuje sve operacije koje traju predugo."

Preciznije je:

> "Timeout samo prekida čekanje u `select`-u. Worker goroutine nastavlja da radi ako nema eksplicitnog cancellation signala."

---

## Pitanje 61 — Kako možemo omogućiti worker-u da reaguje na signal za završetak?

### Odgovor

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

### Šta interviewer očekuje?

* Obrazac `select` sa `jobs` i `done` channel-om.
* Da worker mora aktivno proveravati `done` signal.
* Da `close(done)` je broadcast signal svim radnicima koji dele isti `done` channel.

### Česta greška

Nije precizno reći:

> "Worker se automatski zaustavlja kada nema posla."

Preciznije je:

> "Worker blokira dok ne dobije posao ili signal za završetak. Bez eksplicitnog `done` signala, worker može ostati blokiran zauvek."

---

## Pitanje 62 — Šta se događa ako je jedan od channel-a u `select`-u zatvoren?

### Odgovor

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

### Šta interviewer očekuje?

* Da receive sa zatvorenog channel-a je odmah spreman u `select`-u.
* Da ovo može izazvati unexpected ponašanje u `select` petljama.
* Da treba koristiti comma-ok oblik za detekciju zatvorenog channel-a.

### Česta greška

Nije precizno reći:

> "Zatvoren channel u `select`-u se ignoriše."

Preciznije je:

> "Receive sa zatvorenog channel-a je odmah spreman. U petlji sa `select`-om, zatvoren channel može dominirati jer je uvek spreman."

---

## Pitanje 63 — Zašto zatvoreni channel može izazvati problem u `select` petlji?

### Odgovor

Posmatramo:

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

### Šta interviewer očekuje?

* Da zatvoren channel u `select` petlji postaje stalno spreman.
* Da bez comma-ok provere petlja može procesirati zero value vrednosti beskonačno.
* Da ispravna obrada zahteva eksplicitnu provjeru `ok`.

### Česta greška

Nije precizno reći:

> "Zatvoren channel u petlji automatski završava petlju."

Preciznije je:

> "Zatvoren channel u `select` petlji postaje stalno aktivan — petlja nastavlja da ga bira i dobija zero value. Bez `ok` provjere, ovo je beskonačna petlja nula."

---

## Pitanje 64 — Zašto se `select` često opisuje kao mehanizam za multipleksiranje channel događaja?

### Odgovor

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

### Šta interviewer očekuje?

* Da `select` kombinuje više izvora događaja u jednoj goroutine.
* Da `select` je osnova za izgradnju strukturisane konkurentnosti u Go-u.
* Tipični primeri: posao + cancellation + timer.

### Česta greška

Nije precizno reći:

> "`select` je samo `switch` za channels."

Preciznije je:

> "`select` blokira dok jedan od case-ova ne postane moguć, a ako je više case-ova istovremeno dostupno, bira nasumično. `switch` nema ovu semantiku."

---

## Pitanje 65 — Da li se svi `case` blokovi izvršavaju ako su spremni?

### Odgovor

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

### Šta interviewer očekuje?

* Da `select` izvršava tačno jedan case po izvršavanju.
* Da `for` petlja oko `select`-a je standardni obrazac za kontinuirano slušanje.
* Da višestruka selekcija zahteva petlju.

### Česta greška

Nije precizno reći:

> "`select` izvršava sve case-ove koji su spremni."

Preciznije je:

> "`select` bira tačno jedan case i izvršava ga. Ostali case-ovi se ignorišu u tom prolazu."

---

## Pitanje 66 — Možemo li jednostavnim redosledom `case` grana dati prioritet jednom channel-u?

### Odgovor

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

### Šta interviewer očekuje?

* Da redosled case-ova ne definiše prioritet u `select`-u.
* Da eksplicitni prioritet zahteva poseban dizajn.
* Da program ne sme zavisiti od pretpostavljenog redosleda selekcije.

### Česta greška

Nije precizno reći:

> "Stavim `urgent` case prvi u `select`-u i on će uvek biti prioritetan."

Preciznije je:

> "Redosled case-ova nema uticaj na selekciju. Kada su više case-ova istovremeno dostupni, `select` bira nasumično."

---

## Pitanje 67 — Kada je dovoljan običan receive, a kada je potreban `select`?

### Odgovor

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

### Šta interviewer očekuje?

* Razliku između `<-ch` (jedan channel) i `select` (više channels).
* Da `select` reaguje na prvi dostupan case.
* Primeri upotrebe: timeout, cancellation, višestruki izvori.

### Česta greška

Nije precizno reći:

> "`select` je uvek bolji od običnog receive — koristim ga uvek."

Preciznije je:

> "Obični receive je jasniji kada čekamo samo jedan event. `select` dodaje kompleksnost koja ima smisla samo kada je potrebna koordinacija više događaja."

---

[Prelazak na **Junior → Medior — Interview Questions**](03-junior-to-medior.md)