# Module #1 — Interview Questions: Medior

> **Fajl:** `extras/01-interview-questions/module-1/04-medior.md`
>
> **Nivo:** Medior
>
> **Oblast:** #1 — Concurrency Fundamentals

---

## 1. Uvod

Na **Medior** nivou očekuje se da kandidat više ne posmatra goroutine i channel kao izolovane Go konstrukcije.

Junior kandidat treba da zna:

> „Kako se pokreće goroutine?“

Medior kandidat treba da zna:

> „Kako će se ova goroutine ponašati tokom čitavog svog životnog ciklusa, gde može da blokira, ko je odgovoran za njen završetak i šta se dešava ako komunikaciona strana sistema prestane da funkcioniše?“

Zato se pitanja ovog nivoa fokusiraju na:

* ponašanje goroutines tokom izvršavanja,
* blocking i synchronization semantics,
* ownership channel-a,
* producer/consumer odnose,
* lifecycle management,
* sprečavanje goroutine leak-ova,
* izbor odgovarajućeg komunikacionog modela,
* posledice pogrešnog dizajna.

---

# 2. Goroutine Lifecycle

## Pitanje 1

**Opiši životni ciklus goroutine od trenutka kada je kreirana do trenutka kada se završi.**

### Odgovor

Kada se izvrši:

```go
go process()
```

Go runtime kreira novu goroutine i stavlja je u sistem izvršavanja kojim upravlja Go scheduler.

Pojednostavljeno, lifecycle izgleda ovako:

```text
Creation
   │
   ▼
Runnable
   │
   ▼
Running
   │
   ├──────────────► Blocked
   │                  │
   │                  ▼
   │               Runnable
   │
   ▼
Finished
```

Goroutine može prelaziti između različitih stanja.

Na primer, goroutine može:

1. biti kreirana,
2. čekati da bude raspoređena,
3. početi izvršavanje,
4. blokirati se na channel operaciji,
5. kasnije postati ponovo runnable,
6. nastaviti izvršavanje,
7. završiti izvršavanje.

Važna stvar je da **goroutine nije isto što i OS thread**.

Go runtime upravlja goroutines, dok scheduler odlučuje kada će određena goroutine dobiti mogućnost izvršavanja na dostupnom OS thread-u.

---

## Pitanje 2

**Da li `go foo()` znači da će `foo()` odmah početi da se izvršava?**

### Odgovor

Ne.

```go
go foo()
```

znači da se `foo()` izvršava kao nova goroutine, ali ne znači da će njeno izvršavanje početi odmah.

Na primer:

```go
func main() {
	go foo()

	fmt.Println("main")
}
```

Ne postoji garancija da će `foo()` početi pre:

```go
fmt.Println("main")
```

Scheduler odlučuje kada će goroutine biti izvršena.

Zbog toga nikada ne treba pisati konkurentni kod koji zavisi od pretpostavke:

> „Pošto sam napisao `go foo()`, `foo()` će sigurno biti izvršena pre sledeće linije.“

To nije concurrency guarantee.

---

# 3. Blocking kao deo modela izvršavanja

## Pitanje 3

**Šta znači da je goroutine blokirana?**

### Odgovor

Blokirana goroutine trenutno ne može da nastavi izvršavanje jer čeka neki događaj ili resurs.

U Module #1 najvažniji primer je channel komunikacija.

Na primer:

```go
ch := make(chan int)

value := <-ch
```

Ako trenutno nema goroutine koja će poslati vrednost:

```go
ch <- value
```

receiver će čekati.

Dakle:

```text
Receiver
   │
   ▼
<- ch
   │
   │ nema podatka
   ▼
BLOCKED
```

Kada druga goroutine pošalje vrednost:

```text
Sender
   │
   ▼
ch <- value
   │
   ▼
Receiver se budi
   │
   ▼
nastavlja izvršavanje
```

Blocking sam po sebi nije greška.

Naprotiv, **blocking je fundamentalni deo Go concurrency modela**.

Problem nastaje kada goroutine blokira u situaciji iz koje više nikada ne može da izađe.

Tada govorimo o problemima kao što su:

* deadlock,
* goroutine leak,
* indefinite blocking.

---

# 4. Da li je blocking uvek loš?

## Pitanje 4

**Da li treba izbegavati blocking operacije u Go concurrency kodu?**

### Odgovor

Ne.

To bi bilo pogrešno shvatanje concurrency modela.

Na primer:

```go
value := <-ch
```

je potpuno legitimna operacija.

Ako goroutine treba da čeka rezultat druge goroutine, blocking receive može biti upravo ono što želimo.

Problem nije:

> „Goroutine blokira.“

Problem je:

> „Goroutine blokira bez validnog mehanizma kojim će se njeno čekanje završiti.“

Na primer:

```go
ch := make(chan int)

func worker() {
	value := <-ch

	fmt.Println(value)
}
```

Ako nikada ne postoji sender:

```go
ch <- value
```

goroutine će ostati blokirana.

Zato Medior developer treba da postavi dodatno pitanje:

> **Koji događaj omogućava ovoj goroutine da nastavi ili završi?**

To je mnogo važnije od samog pitanja da li operacija blokira.

---

# 5. Unbuffered Channel kao synchronization point

## Pitanje 5

**Zašto unbuffered channel možemo posmatrati kao synchronization point?**

### Odgovor

Kod unbuffered channel-a nema prostora za skladištenje vrednosti.

```go
ch := make(chan int)
```

Sender i receiver moraju međusobno da se usklade.

Na primer:

```go
go func() {
	ch <- 42
}()

value := <-ch
```

Sender:

```go
ch <- 42
```

ne može jednostavno da ostavi `42` u bufferu.

Mora postojati odgovarajući receiver.

Zbog toga unbuffered channel istovremeno predstavlja:

1. mehanizam prenosa podataka,
2. synchronization mechanism.

Možemo ga konceptualno posmatrati kao:

```text
Sender
   │
   │ 42
   ▼
Synchronization point
   │
   ▼
Receiver
```

Ovo je jedna od ključnih razlika u odnosu na buffered channel.

---

# 6. Buffered Channel i promena blocking semantics

## Pitanje 6

**Kako buffer menja ponašanje channel-a?**

### Odgovor

Ako kreiramo:

```go
ch := make(chan int, 3)
```

channel ima kapacitet `3`.

Sender može poslati do tri vrednosti bez neposrednog receiver-a:

```go
ch <- 1
ch <- 2
ch <- 3
```

Tek sledeći send:

```go
ch <- 4
```

blokira ako u međuvremenu niko nije pročitao neku od vrednosti.

Konceptualno:

```text
        Buffer
┌───────────────────┐
│ 1 │ 2 │ 3 │       │
└───────────────────┘
          ▲
          │
       sender
```

Buffer zato razdvaja producer i consumer u određenoj meri.

Ali važno:

> **Buffered channel ne uklanja synchronization.**

On samo omogućava određenu količinu asinhronog napretka između sender-a i receiver-a.

Kada se buffer napuni:

```text
producer
   │
   ▼
[1][2][3]
         │
         │ full
         ▼
      BLOCKED
```

producer ponovo mora da čeka.

---

# 7. Kako Medior bira između buffered i unbuffered channel-a?

## Pitanje 7

**Kada bi izabrao unbuffered, a kada buffered channel?**

### Odgovor

Ne treba birati buffer samo zato što:

> „Buffered channel je brži.“

To je previše pojednostavljeno.

Izbor treba da proizlazi iz semantics sistema.

### Unbuffered channel

Koristan je kada želimo direktnu koordinaciju između sender-a i receiver-a.

Na primer:

```go
done := make(chan struct{})
```

ili kada želimo da slanje predstavlja synchronization point.

### Buffered channel

Koristan je kada želimo da producer može određeno vreme da napreduje nezavisno od consumer-a.

Na primer:

```go
jobs := make(chan Job, 100)
```

To može predstavljati queue između:

```text
Producers
    │
    ▼
┌───────────────┐
│ jobs channel  │
│ capacity 100  │
└───────────────┘
    │
    ▼
Workers
```

Ali veličina buffera mora imati smisao.

Buffer od:

```go
100
```

nije automatski bolji od:

```go
10
```

ili:

```go
1000
```

Veličina buffera predstavlja deo concurrency dizajna i treba je posmatrati kroz:

* throughput,
* latency,
* burst behavior,
* memory consumption,
* backpressure,
* producer/consumer brzinu.

---

# 8. Channel ownership

## Pitanje 8

**Šta podrazumevamo pod ownership-om channel-a?**

### Odgovor

Ownership predstavlja pitanje:

> **Ko je odgovoran za kreiranje, slanje, zatvaranje i prosleđivanje channel-a?**

Na primer:

```go
func producer() <-chan int {
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

Ovde producer:

* kreira channel,
* šalje podatke,
* zatvara channel.

Caller dobija samo receive-only channel:

```go
<-chan int
```

Caller može da radi:

```go
for value := range producer() {
	fmt.Println(value)
}
```

ali ne može da:

```go
close(ch)
```

niti:

```go
ch <- value
```

Ovakav dizajn jasno definiše ownership.

To smanjuje mogućnost da više komponenti pokušava da upravlja istim channel-om.

---

# 9. Ko treba da zatvori channel?

## Pitanje 9

**Ko bi trebalo da zatvori channel?**

### Odgovor

Opšte pravilo je:

> **Sender, odnosno komponenta koja zna da više neće biti poslatih vrednosti, treba da zatvori channel.**

Na primer:

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

Producer zna kada je završio proizvodnju.

Zato producer radi:

```go
close(ch)
```

Consumer ne treba proizvoljno da zatvara channel koji mu je dat za čitanje.

Ovo je posebno važno kada postoji više učesnika.

Pogrešan ownership može dovesti do:

```text
send on closed channel
```

panic-a.

---

# 10. Ključna Medior perspektiva

Kod Junior nivoa dovoljno je razumeti:

```text
goroutine
channel
send
receive
close
```

Na Medior nivou moramo razumeti odnose između njih:

```text
                 ┌───────────────┐
                 │   Goroutine   │
                 └───────┬───────┘
                         │
                         │ send
                         ▼
                 ┌───────────────┐
                 │    Channel    │
                 └───────┬───────┘
                         │
                         │ receive
                         ▼
                 ┌───────────────┐
                 │   Goroutine   │
                 └───────────────┘
```

A zatim postaviti pitanja:

* Ko poseduje channel?
* Ko ga zatvara?
* Ko šalje?
* Ko prima?
* Ko može da blokira?
* Šta se dešava kada nema receiver-a?
* Šta se dešava kada nema sender-a?
* Ko označava kraj stream-a?
* Šta se dešava ako consumer prestane da čita?
* Da li neka goroutine može ostati zauvek blokirana?

To je prelazak sa **syntax-level knowledge** na **concurrency design reasoning**.

---

# 11. Šta treba zapamtiti

Medior kandidat treba da bude sposoban da objasni sledeće:

1. Goroutine može biti runnable, running, blocked ili završena.
2. `go f()` ne garantuje trenutno izvršavanje funkcije.
3. Blocking nije sam po sebi greška.
4. Problem nastaje kada blocking nema validan izlaz.
5. Unbuffered channel predstavlja snažnu synchronization tačku.
6. Buffered channel omogućava određeni stepen odvajanja producer-a i consumer-a.
7. Buffer ne uklanja potrebu za pravilnim concurrency dizajnom.
8. Channel ownership treba jasno definisati.
9. Komponenta koja proizvodi podatke obično treba da zatvori channel.
10. Lifecycle goroutine-a mora biti deo dizajna sistema, a ne naknadna briga.

---

Na medior nivou više nije dovoljno znati da kanal omogućava komunikaciju između goroutine-a. Potrebno je razumeti **kada i zašto određena operacija blokira**, kakve posledice to ima na sistem i kako dizajnirati komunikacioni protokol koji neće dovesti do deadlock-a, goroutine leak-a ili nekontrolisanog čekanja.

---

## Pitanje 2: Šta se dešava kada goroutine pošalje vrednost na kanal, a u tom trenutku nema spremnog receiver-a?

Odgovor zavisi od toga da li je kanal **unbuffered** ili **buffered**.

Kod nebaferovanog kanala:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()
```

Send operacija:

```go
ch <- 42
```

ne može da završi dok druga goroutine ne izvrši odgovarajući receive:

```go
value := <-ch
```

Dakle, kod unbuffered kanala komunikacija predstavlja direktnu sinhronizaciju između pošiljaoca i primaoca.

Možemo konceptualno posmatrati operaciju kao:

```text
Sender
   │
   │ ch <- value
   ▼
[ čeka receiver ]
   │
   │ receiver prihvata value
   ▼
nastavlja izvršavanje
```

Kod baferovanog kanala ponašanje je drugačije.

Ako postoji slobodan prostor u buffer-u:

```go
ch := make(chan int, 10)

ch <- 42
```

send može odmah da se završi bez postojanja receiver-a.

Vrednost se smešta u buffer:

```text
Sender
   │
   │ ch <- 42
   ▼
┌───────────────┐
│ Channel       │
│ buffer        │
│               │
│ [42]          │
└───────────────┘
```

Tek kada je buffer pun, sledeći send mora da čeka receiver-a koji će osloboditi prostor.

---

## Pitanje 3: Da li je `send` na buffered channel-u uvek neblokirajući?

Ne.

Ovo je jedna od čestih pogrešnih pretpostavki.

Ako imamo:

```go
ch := make(chan int, 2)
```

možemo poslati dve vrednosti bez receiver-a:

```go
ch <- 1
ch <- 2
```

Ali treći send:

```go
ch <- 3
```

blokiraće dok neka druga goroutine ne izvrši receive.

Model:

```text
capacity = 2

┌─────────────────┐
│ Channel buffer  │
├─────────────────┤
│ 1               │
│ 2               │
└─────────────────┘
      FULL

ch <- 3

      │
      ▼

   BLOCKED
```

Kada receiver izvrši:

```go
value := <-ch
```

jedna pozicija u buffer-u se oslobađa i sender može da nastavi.

Zato buffered channel ne uklanja blokiranje.

On samo **pomera granicu na kojoj blokiranje nastaje**.

---

## Pitanje 4: Koja je praktična razlika između ova dva dizajna?

```go
make(chan int)
```

i:

```go
make(chan int, 100)
```

Prvi kanal zahteva direktnu koordinaciju sender-a i receiver-a.

Drugi dozvoljava određenu količinu asinhronog rada.

To znači da izbor kapaciteta predstavlja deo dizajna sistema.

Na primer:

```go
jobs := make(chan Job)
```

može značiti:

> Nemoj dozvoliti producer-u da napreduje dok worker nije spreman da prihvati posao.

Dok:

```go
jobs := make(chan Job, 100)
```

može značiti:

> Dozvoli producer-u da privremeno proizvede do 100 poslova bez čekanja worker-a.

To može biti korisno, ali uvodi dodatnu memorijsku potrošnju i može prikriti problem ako producer sistematski proizvodi podatke brže nego što consumer može da ih obradi.

---

## Pitanje 5: Kako buffering može da prikrije problem u concurrency dizajnu?

Pretpostavimo:

```go
jobs := make(chan Job, 1000)
```

Producer proizvodi:

```text
1000 jobs/sec
```

dok worker-i obrađuju:

```text
500 jobs/sec
```

Na početku sistem može izgledati potpuno stabilno.

Buffer apsorbuje razliku:

```text
Producer
  │
  │ 1000/sec
  ▼
┌─────────────────┐
│ buffer          │
│                 │
│ jobs accumulate │
└─────────────────┘
  │
  │ 500/sec
  ▼
Workers
```

Ali buffer ima konačan kapacitet.

Kada se napuni:

```text
Producer → BUFFER FULL → BLOCKED
                         │
                         ▼
                     backpressure
```

Zato buffering ne rešava fundamentalni throughput problem.

On samo omogućava sistemu da **privremeno apsorbuje neravnomeran workload**.

---

## Pitanje 6: Šta je backpressure u channel-based sistemu?

**Backpressure** je mehanizam kojim sporiji deo sistema ograničava brzinu kojom brži deo sistema može da proizvodi posao.

Na primer:

```text
Producer
   │
   │ jobs
   ▼
Channel
   │
   ▼
Workers
```

Ako workers ne mogu dovoljno brzo da obrade poslove, channel će se popunjavati.

Kada više nema prostora:

```text
Producer
   │
   ▼
[ FULL BUFFER ]
      │
      ▼
   BLOCKED
```

Blokiranje producer-a predstavlja oblik prirodnog backpressure-a.

To je često poželjno ponašanje.

Umesto da sistem neograničeno akumulira posao u memoriji:

```text
RAM
████████████████████████████████████
```

producer se usporava.

Ovo je posebno važno kod sistema sa velikim prometom, gde nekontrolisana akumulacija može dovesti do:

* povećanja latency-ja,
* rasta memorijske potrošnje,
* povećanog GC pritiska,
* goroutine blokiranja,
* timeout-a,
* cascading failure-a.

---

## Pitanje 7: Kako `select` menja ponašanje blokirajućih channel operacija?

`select` omogućava goroutine-u da čeka na više channel operacija istovremeno.

Na primer:

```go
select {
case job := <-jobs:
    process(job)

case <-ctx.Done():
    return
}
```

Ova goroutine ne čeka samo na posao.

Ona istovremeno čeka i na signal za cancellation.

To je fundamentalno važan concurrency obrazac.

Bez `select`:

```go
job := <-jobs
```

goroutine može ostati blokirana sve dok neko ne pošalje vrednost.

Sa cancellation granom:

```go
select {
case job := <-jobs:
    process(job)

case <-ctx.Done():
    return
}
```

goroutine može napustiti čekanje.

Konceptualno:

```text
                 ┌── jobs ───────► process
                 │
goroutine ─ select
                 │
                 └── ctx.Done() ─► return
```

Ovo je jedan od osnovnih mehanizama za kontrolisanje lifecycle-a concurrent komponenti.

---

## Ključne lekcije

Na medior nivou treba razumeti sledeće:

1. **Unbuffered channel** zahteva koordinaciju sender-a i receiver-a.
2. **Buffered channel** omogućava određenu količinu asinhronog rada.
3. Buffered channel i dalje može blokirati kada se napuni.
4. Kapacitet buffera je deo arhitektonskog dizajna.
5. Buffer ne rešava throughput problem.
6. Channel može prirodno implementirati backpressure.
7. `select` omogućava kombinovanje komunikacije sa cancellation-om i drugim događajima.
8. Blokiranje mora biti posmatrano u kontekstu lifecycle-a cele goroutine-e, a ne samo pojedinačne channel operacije.

---

Na medior nivou očekuje se da developer ne samo da zna šta je deadlock, već da ume da **prepozna uslove koji do njega dovode**, da analizira komunikacioni protokol između goroutine-a i da razlikuje deadlock od drugih problema poput goroutine leak-a ili običnog sporog izvršavanja.

---

## Pitanje 8: Šta je deadlock u Go concurrency sistemu?

**Deadlock** nastaje kada goroutine-e ili drugi concurrent entiteti čekaju jedni na druge na način iz kojeg više ne mogu da napreduju.

Jedan od najjednostavnijih primera sa kanalom:

```go
func main() {
    ch := make(chan int)

    ch <- 42

    fmt.Println("done")
}
```

Ovde `main` goroutine pokušava da pošalje vrednost na unbuffered channel:

```go
ch <- 42
```

Ali ne postoji druga goroutine koja bi izvršila receive:

```go
<-ch
```

Zbog toga `main` blokira.

Pošto nema druge goroutine koja može da omogući nastavak izvršavanja, program završava u deadlock-u.

Konceptualno:

```text
main
 │
 │ ch <- 42
 ▼
BLOCKED
 │
 └── nema receiver-a
```

Go runtime u određenim situacijama može detektovati da su sve goroutine-e blokirane i prijaviti:

```text
fatal error: all goroutines are asleep - deadlock!
```

---

## Pitanje 9: Zašto je deadlock sa buffered channel-om drugačiji?

Kod buffered channel-a send može da napreduje sve dok buffer ima slobodnog prostora.

Na primer:

```go
ch := make(chan int, 1)

ch <- 42
```

ne blokira.

Ali:

```go
ch <- 100
```

blokira jer je buffer pun.

Dakle:

```text
capacity = 1

┌─────────┐
│   42    │
└─────────┘
    FULL

ch <- 100
    │
    ▼
 BLOCKED
```

Ako ne postoji receiver koji će ukloniti vrednost iz channel-a, goroutine ostaje blokirana.

Zbog toga se deadlock može pojaviti i sa buffered kanalima.

Buffer samo menja trenutak kada nastaje blokiranje.

---

## Pitanje 10: Kako nastaje circular wait između goroutine-a?

Jedan klasičan scenario je kada dve goroutine-e čekaju jedna drugu.

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

Obe send operacije zahtevaju odgovarajućeg receiver-a.

Prva goroutine radi:

```go
ch1 <- 1
```

a druga:

```go
ch2 <- 2
```

Ako obe goroutine stignu do send operacija pre nego što bilo koja izvrši receive, obe mogu ostati blokirane.

Model:

```text
Goroutine A
    │
    │ send ch1
    ▼
  WAIT
    ▲
    │
receiver mora biti B


Goroutine B
    │
    │ send ch2
    ▼
  WAIT
    ▲
    │
receiver mora biti A
```

Nijedna goroutine ne može da nastavi do receive operacije.

To je circular dependency.

---

## Pitanje 11: Koja četiri uslova karakterišu klasičan deadlock?

Klasična analiza deadlock-a koristi četiri Coffman uslova:

1. **Mutual exclusion**
2. **Hold and wait**
3. **No preemption**
4. **Circular wait**

Kod Go concurrency sistema posebno je koristan koncept **circular wait**.

Na primer:

```text
Goroutine A
    │
    └── čeka B

Goroutine B
    │
    └── čeka A
```

Međutim, kod channel-based concurrency-ja treba razmišljati šire.

Ne mora postojati mutex da bi postojao deadlock.

Isti princip može nastati kroz:

* send operacije,
* receive operacije,
* `select`,
* `WaitGroup`,
* `Mutex`,
* `RWMutex`,
* `Cond`,
* kombinaciju više synchronization primitives.

Zato deadlock nije problem koji pripada samo `sync` paketu.

---

## Pitanje 12: Da li svaki blokirani channel znači deadlock?

Ne.

Ovo je veoma važna razlika.

Blokiranje samo po sebi nije greška.

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

Sender kratko blokira dok receiver ne bude spreman.

To je očekivano ponašanje.

Deadlock postoji kada **blokiranje predstavlja stanje iz kojeg sistem više ne može da napreduje**.

Možemo razlikovati:

```text
Blocking
   │
   ├── očekivano
   │
   ├── privremeno
   │
   └── deadlock
```

Dakle, pitanje nije:

> "Da li goroutine blokira?"

nego:

> "Da li postoji validan događaj koji će omogućiti goroutine-i da nastavi?"

---

## Pitanje 13: Kako razlikovati deadlock od goroutine leak-a?

**Deadlock** znači da sistem ne može da napravi potreban napredak zbog međusobnog čekanja ili nedostupnog synchronization događaja.

**Goroutine leak** znači da goroutine ostane aktivna ili blokirana iako više ne postoji realna potreba da nastavi da postoji.

Na primer:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        process(value)
    }
}
```

Ako se worker pokrene:

```go
go worker(ch)
```

a ostatak sistema kasnije prestane da koristi `ch`, worker može zauvek ostati blokiran na:

```go
value := <-ch
```

To može predstavljati goroutine leak.

Sistem možda nastavlja normalno da radi:

```text
main ───────────────► continues

worker
  │
  ▼
<-ch
  │
  └── blocked forever
```

Za razliku od klasičnog deadlock-a, ne mora cela aplikacija biti zaglavljena.

---

## Pitanje 14: Kako `select` može pomoći u sprečavanju goroutine leak-a?

Umesto:

```go
func worker(ch <-chan Job) {
    for {
        job := <-ch
        process(job)
    }
}
```

možemo koristiti cancellation signal:

```go
func worker(
    ctx context.Context,
    ch <-chan Job,
) {
    for {
        select {
        case job := <-ch:
            process(job)

        case <-ctx.Done():
            return
        }
    }
}
```

Sada worker ima eksplicitnu izlaznu putanju.

```text
                ┌── job ──► process
                │
worker ─ select ┤
                │
                └── cancel ──► return
```

Ovo je jedan od najvažnijih principa robustnog concurrent dizajna:

> Goroutine koja može da čeka mora imati jasno definisan način da prestane da čeka.

Naravno, konkretan protokol može biti drugačiji, ali lifecycle mora biti eksplicitno dizajniran.

---

## Pitanje 15: Kako analizirati deadlock u složenom sistemu?

Kada se pojavi deadlock, korisno je napraviti **wait-for graph**.

Na primer:

```text
G1 ──waits for──► G2
▲                  │
│                  │
└────waits for─────┘
```

Ako postoji ciklus:

```text
G1 → G2 → G3 → G1
```

postoji ozbiljna sumnja na circular wait.

Kod channel sistema možemo posmatrati i resurse:

```text
G1 ──send──► ch1 ──requires──► G2
G2 ──send──► ch2 ──requires──► G3
G3 ──send──► ch3 ──requires──► G1
```

Ovakav model je često mnogo korisniji od prostog gledanja source code-a liniju po liniju.

Kod produkcionih sistema treba identifikovati:

* koja goroutine čeka,
* na kojoj operaciji čeka,
* ko treba da je probudi,
* da li taj entitet postoji,
* da li je taj entitet takođe blokiran,
* da li postoji circular dependency.

---

## Medior perspektiva

Kod ozbiljnog concurrency koda nije dovoljno pitati:

> "Da li ovaj channel radi?"

Potrebno je pitati:

> "Ko poseduje channel, ko šalje, ko prima, kada se channel zatvara, šta se dešava kada producer završi, šta se dešava kada consumer završi i šta se dešava ako jedna strana više nije dostupna?"

To predstavlja prelaz od **korišćenja concurrency primitive** ka **dizajniranju concurrency protokola**.

### Ključne lekcije

* Deadlock nije isto što i privremeno blokiranje.
* Buffered channels mogu takođe dovesti do deadlock-a.
* Circular wait je jedan od najvažnijih obrazaca deadlock-a.
* Deadlock ne zahteva mutex.
* Goroutine leak može postojati čak i kada ostatak aplikacije normalno radi.
* `select` sa cancellation signalom često predstavlja važan mehanizam za kontrolu lifecycle-a.
* Wait-for analiza je koristan mentalni model za složene deadlock scenarije.
* Svaka blokirajuća operacija treba da ima jasno definisan protokol koji obezbeđuje nastavak ili prekid čekanja.

---

Na medior nivou očekuje se razumevanje da pokretanje goroutine-e nije samo pitanje dodavanja `go` ključne reči. Potrebno je razmišljati o njenom **životnom ciklusu (lifecycle)**:

```text
creation
   ↓
running
   ↓
waiting / blocked
   ↓
running
   ↓
termination
```

Najčešći problemi u realnim sistemima nastaju kada developer razmišlja samo o početku goroutine-e, a ne o njenom završetku.

---

## Pitanje 16: Šta određuje životni vek goroutine-e?

Goroutine postoji dok njen entry function ne završi.

Na primer:

```go
func worker() {
    fmt.Println("working")
}

func main() {
    go worker()

    time.Sleep(time.Second)
}
```

Goroutine `worker` počinje izvršavanje kada runtime dobije priliku da je rasporedi, a završava se kada:

```go
worker()
```

dođe do kraja.

Nema eksplicitne operacije:

```go
goroutine.Destroy()
```

Go runtime automatski uklanja goroutine kada ona završi izvršavanje.

Međutim, **upravljanje lifecycle-om goroutine-e ostaje odgovornost aplikacije**.

---

## Pitanje 17: Da li `main` čeka da sve goroutine-e završe?

Ne.

Ovo je jedna od osnovnih stvari koju developer mora da razume.

```go
func main() {
    go func() {
        fmt.Println("worker")
    }()

    fmt.Println("main")
}
```

Kada `main` goroutine završi, proces se završava.

Preostale goroutine-e ne nastavljaju život nezavisno od procesa.

Zbog toga je pogrešno razmišljati:

> "Pokrenuo sam goroutine i Go će se pobrinuti da ona završi."

Go runtime upravlja scheduling-om, ali ne upravlja poslovnim lifecycle-om goroutine-e umesto aplikacije.

Ako je potrebno čekati završetak worker-a, mora postojati eksplicitan synchronization mehanizam.

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

Ovde je lifecycle eksplicitno modelovan:

```text
main
 │
 ├── start worker
 │
 │      worker
 │        │
 │        └── Done()
 │
 └── Wait()
        │
        ▼
     continue
```

---

## Pitanje 18: Zašto je `time.Sleep` loš način za čekanje goroutine-e?

Možemo napisati:

```go
go worker()

time.Sleep(time.Second)
```

ali `Sleep` ne predstavlja synchronization protocol.

On samo kaže:

> "Pretpostavi da će worker završiti za jednu sekundu."

To je fundamentalno drugačije od:

```go
wg.Wait()
```

koji kaže:

> "Čekaj dok worker zaista ne signalizira da je završio."

`Sleep` je problematičan jer vreme izvršavanja nije determinističko.

Worker može:

* završiti za 10 ms,
* trajati 500 ms,
* trajati 2 sekunde,
* biti blokiran zauvek.

Zbog toga:

```go
time.Sleep(time.Second)
```

nije pouzdan način upravljanja lifecycle-om.

---

## Pitanje 19: Koji su tipični načini za kontrolu završetka goroutine-e?

Najčešći mehanizmi su:

### `WaitGroup`

Za čekanje da grupa goroutine-a završi:

```go
var wg sync.WaitGroup

wg.Add(2)

go func() {
    defer wg.Done()
    worker1()
}()

go func() {
    defer wg.Done()
    worker2()
}()

wg.Wait()
```

---

### `context.Context`

Za cancellation:

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            doWork()
        }
    }
}
```

---

### Channel

Channel može služiti kao signal:

```go
done := make(chan struct{})

go func() {
    defer close(done)

    doWork()
}()

<-done
```

Ovde zatvaranje channel-a predstavlja signal:

```text
worker
  │
  ├── doWork()
  │
  └── close(done)
          │
          ▼
        waiter
```

---

### Kombinacija mehanizama

U realnim sistemima često se kombinuju:

```text
context
   │
   ├── cancellation
   │
   ▼
workers
   │
   └── WaitGroup
           │
           ▼
        shutdown
```

`context` kontroliše **kada treba stati**, dok `WaitGroup` kontroliše **kada su svi zaista stali**.

---

## Pitanje 20: Koja je razlika između cancellation-a i completion-a?

Ovo je veoma važna razlika.

**Cancellation** znači:

> "Nemoj više da nastavljaš posao."

**Completion** znači:

> "Posao je završen."

Na primer:

```go
ctx, cancel := context.WithCancel(context.Background())

var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    worker(ctx)
}()

cancel()

wg.Wait()
```

Ovde:

```go
cancel()
```

ne znači da je worker već završio.

On samo šalje zahtev:

```text
STOP WORK
```

Worker mora da obradi cancellation i izađe.

Tek nakon:

```go
wg.Wait()
```

možemo znati da je worker završio.

Dakle:

```text
cancel()
   │
   ▼
"treba da stane"
   │
   ▼
worker observes cancellation
   │
   ▼
worker returns
   │
   ▼
wg.Done()
   │
   ▼
wg.Wait() unblocks
```

Ovo je veoma važan obrazac za graceful shutdown.

---

## Pitanje 21: Šta znači da goroutine ima "owner-a"?

U ozbiljnom concurrent sistemu korisno je imati jasan odgovor na pitanje:

> Ko je odgovoran za ovu goroutine-u?

Owner bi trebalo da zna:

* zašto goroutine postoji,
* kada se pokreće,
* šta radi,
* šta joj omogućava da napreduje,
* kada treba da se završi,
* kako se zaustavlja,
* ko čeka njen završetak.

Na primer:

```go
type WorkerPool struct {
    ctx    context.Context
    cancel context.CancelFunc
    wg     sync.WaitGroup
}
```

Pool može biti owner svojih worker-a.

```text
WorkerPool
    │
    ├── Worker 1
    ├── Worker 2
    ├── Worker 3
    │
    ├── cancellation
    │
    └── WaitGroup
```

Ovakav dizajn je mnogo sigurniji od proizvoljnog pokretanja goroutine-a kroz različite delove sistema.

---

## Pitanje 22: Zašto je "fire-and-forget" goroutine potencijalno problematična?

Primer:

```go
func process(req Request) {
    go sendNotification(req)
}
```

Na prvi pogled izgleda jednostavno.

Ali postavljaju se pitanja:

* Ko čeka `sendNotification`?
* Šta ako proces bude ugašen?
* Šta ako `sendNotification` blokira?
* Šta ako napravi error?
* Ko upravlja retry-em?
* Da li goroutine može da procuri?
* Koliko ovakvih goroutine-a može postojati istovremeno?

Ako se ista funkcija pozove 100.000 puta:

```text
process()
   │
   ├── go notification()
   ├── go notification()
   ├── go notification()
   ├── ...
   └── go notification()
```

možemo dobiti nekontrolisan broj goroutine-a.

Zato "fire-and-forget" nije automatski loš, ali mora biti **namerno dizajniran concurrency model**.

---

## Pitanje 23: Kako dizajnirati goroutine tako da njen lifecycle bude eksplicitan?

Jedan robustan obrazac je:

```go
type Worker struct {
    ctx    context.Context
    cancel context.CancelFunc
    wg     sync.WaitGroup
}
```

Pokretanje:

```go
func (w *Worker) Start() {
    w.wg.Add(1)

    go func() {
        defer w.wg.Done()

        w.run()
    }()
}
```

Zaustavljanje:

```go
func (w *Worker) Stop() {
    w.cancel()
    w.wg.Wait()
}
```

Lifecycle postaje:

```text
             Start()
                │
                ▼
        ┌──────────────┐
        │    Worker    │
        │   running    │
        └──────┬───────┘
               │
          cancel()
               │
               ▼
        ┌──────────────┐
        │  stopping    │
        └──────┬───────┘
               │
           Done()
               │
               ▼
        ┌──────────────┐
        │  terminated  │
        └──────────────┘
```

Ovakav dizajn razdvaja tri različite odgovornosti:

1. **Start** — pokretanje rada.
2. **Cancel** — zahtev za prekid rada.
3. **Wait** — potvrda da je rad zaista završen.

---

## Medior perspektiva

Kod jednostavnog programa dovoljno je znati kako da se pokrene goroutine:

```go
go worker()
```

Kod produkcionog sistema pitanje postaje mnogo ozbiljnije:

> Ko upravlja njenim životnim ciklusom?

Dobar concurrent dizajn treba da omogući da se za svaku važnu goroutine-u odgovori na sledeća pitanja:

```text
Who starts it?
Who owns it?
What does it wait for?
How does it stop?
Who waits for termination?
What happens on failure?
```

Ako na ova pitanja ne postoji jasan odgovor, postoji rizik od:

* goroutine leak-a,
* nekontrolisanog rasta broja goroutine-a,
* nepotpunog shutdown-a,
* izgubljenih rezultata,
* izgubljenih error-a,
* deadlock-a,
* nedeterminističkog ponašanja.

### Ključne lekcije

* Završetak goroutine-e je određen završetkom njene funkcije.
* `main` ne čeka automatski ostale goroutine-e.
* `time.Sleep` nije synchronization primitive.
* `WaitGroup` služi za praćenje completion-a.
* `context` služi za cancellation propagation.
* Cancellation i completion nisu ista stvar.
* Goroutine treba da ima jasan lifecycle.
* Važne goroutine-e treba da imaju jasno definisanog owner-a.
* "Fire-and-forget" mora biti svesna arhitektonska odluka.
* `Start → Cancel → Wait` predstavlja veoma koristan obrazac za lifecycle management.

---

Na medior nivou više nije dovoljno znati samo sintaksu:

```go
ch := make(chan int)
ch <- value
value := <-ch
```

Potrebno je razumeti **zašto je channel uveden u dizajn**, kakvu ulogu ima u koordinaciji goroutine-a i koje posledice ima njegov izbor.

Channel nije samo "cev" kroz koju prolaze podaci.

On istovremeno može predstavljati:

* komunikacioni mehanizam,
* synchronization primitive,
* ownership boundary,
* lifecycle signal,
* backpressure mehanizam,
* deo concurrency protokola.

Zbog toga izbor channel-a predstavlja dizajnersku odluku.

---

## Pitanje 24: Kada koristiti channel, a kada `sync.Mutex`?

Ovo je jedno od najvažnijih pitanja za medior Go developera.

Ne postoji pravilo:

> "Channels su za concurrency, mutex je za obične stvari."

Oba mehanizma služe za koordinaciju concurrent izvršavanja, ali rešavaju različite probleme.

### Channel

Channel je prirodan kada goroutine-e treba da:

> komuniciraju i prenose ownership nad podacima.

Na primer:

```go
jobs := make(chan Job)

go worker(jobs)

jobs <- job
```

Ovde postoji tok podataka:

```text
producer
   │
   │ Job
   ▼
channel
   │
   ▼
worker
```

### Mutex

Mutex je prirodan kada više goroutine-a treba da pristupi:

> zajedničkom stanju.

Na primer:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.value++
    c.mu.Unlock()
}
```

Ovde nema potrebe za komunikacionim pipeline-om.

Postoji shared state:

```text
        ┌───────────┐
Worker1 │           │
        │  Counter  │
Worker2 │           │
        └───────────┘
             ▲
             │
           Mutex
```

Praktično pravilo:

> Ako problem izgleda kao "pošalji posao/podatak drugoj goroutine-i", channel je često prirodan izbor.

> Ako problem izgleda kao "zaštiti ovo zajedničko stanje", mutex je često prirodniji izbor.

---

## Pitanje 25: Zašto buffered i unbuffered channel predstavljaju različite synchronization modele?

Unbuffered channel:

```go
ch := make(chan int)
```

ima kapacitet:

```text
0
```

Send i receive moraju da se usklade.

```go
ch <- 42
```

ne može normalno da završi dok druga strana ne primi vrednost.

Možemo ga posmatrati kao:

```text
sender
   │
   │ rendezvous
   ▼
receiver
```

To predstavlja veoma jak synchronization point.

---

Buffered channel:

```go
ch := make(chan int, 10)
```

omogućava do 10 vrednosti da bude smešteno bez trenutnog receiver-a.

```text
producer
   │
   ▼
┌───────────────┐
│ buffer        │
│ [job][job]... │
└───────┬───────┘
        │
        ▼
      worker
```

Time se producer i consumer delimično razdvajaju u vremenu.

Ali buffered channel ne znači:

> "problem sa concurrency-jem je rešen."

Buffer samo uvodi određenu količinu međuspremnika.

---

## Pitanje 26: Kako veličina buffer-a utiče na sistem?

Pretpostavimo:

```go
jobs := make(chan Job, 100)
```

Broj `100` nije samo tehnički detalj.

On određuje koliko posla može privremeno da bude prihvaćeno bez blokiranja sender-a.

Ako producer radi brže od consumer-a:

```text
producer rate  >  consumer rate
```

buffer će se postepeno puniti:

```text
0%
   ↓
25%
   ↓
50%
   ↓
75%
   ↓
100%
```

Kada se napuni:

```go
jobs <- job
```

počinje da blokira.

To može biti korisno.

Zašto?

Zato što channel tada prirodno uvodi **backpressure**.

```text
producer
   │
   ▼
┌─────────────┐
│   buffer    │
│ ██████████  │ ← full
└──────┬──────┘
       │
       ▼
    consumer
```

Producer više ne može neograničeno da proizvodi posao.

Zbog toga buffer može biti deo stabilizacionog mehanizma sistema.

---

## Pitanje 27: Šta je backpressure?

Backpressure je mehanizam kojim sporiji deo sistema ograničava brzinu kojom brži deo sistema može da proizvodi posao.

Na primer:

```text
Producer
  10k jobs/s
      │
      ▼
   channel
      │
      ▼
Consumer
   2k jobs/s
```

Ako nema ograničenja, queue može neograničeno rasti.

Sa bounded channel-om:

```go
jobs := make(chan Job, 1000)
```

sistem dobija granicu:

```text
maximum queued jobs ≈ 1000
```

Kada se granica dostigne, producer mora da:

* čeka,
* odbaci posao,
* vrati grešku,
* primeni timeout,
* izvrši neki fallback.

To je mnogo zdravije od neograničenog gomilanja rada.

---

## Pitanje 28: Da li buffered channel garantuje bolju performansu?

Ne.

To je česta pogrešna pretpostavka.

Buffered channel može:

* smanjiti broj trenutnih blokiranja,
* omogućiti burst handling,
* povećati throughput u određenim workload-ima,
* razdvojiti producer i consumer u vremenu.

Ali može takođe:

* povećati memorijsku potrošnju,
* sakriti problem sporog consumer-a,
* povećati latency,
* omogućiti gomilanje stale posla,
* odložiti pojavu backpressure-a.

Na primer:

```go
jobs := make(chan Job, 1_000_000)
```

nije automatski bolji dizajn od:

```go
jobs := make(chan Job, 100)
```

Veliki buffer može samo sakriti fundamentalni problem:

```text
producer >>> consumer
```

Ako consumer trajno ne može da prati producer-a, buffer će se na kraju napuniti.

---

## Pitanje 29: Šta znači da channel definiše ownership boundary?

Razmotrimo:

```go
jobs := make(chan Job)

go worker(jobs)
```

Sada možemo definisati odgovornost:

```text
Producer
   │
   │ owns Job before send
   ▼
Channel
   │
   │ transfers Job
   ▼
Worker
   │
   │ owns Job after receive
   ▼
Processing
```

Ovakav način razmišljanja je veoma važan.

Kada se podatak pošalje drugoj goroutine-i, treba razumeti:

* ko ga dalje menja,
* ko je odgovoran za lifecycle,
* da li se objekat dalje deli,
* da li postoji concurrent access,
* da li je potrebna dodatna zaštita.

Channel može da uvede jasnu granicu između dva concurrent dela sistema.

---

## Pitanje 30: Zašto je channel direction važan u API dizajnu?

Umesto:

```go
func worker(ch chan Job)
```

možemo napisati:

```go
func worker(ch <-chan Job)
```

što jasno govori:

> ova funkcija samo prima.

Ili:

```go
func producer(ch chan<- Job)
```

što govori:

> ova funkcija samo šalje.

Time compiler može da spreči određene greške.

Na primer:

```go
func worker(ch <-chan Job) {
    ch <- Job{} // compile error
}
```

ili:

```go
func producer(ch chan<- Job) {
    close(ch)

    // <-ch // compile error
}
```

Directional channels zato nisu samo sintaktička pogodnost.

Oni predstavljaju deo **API contract-a**.

---

## Pitanje 31: Ko treba da zatvori channel?

Opšte pravilo:

> Goroutine koja je odgovorna za proizvodnju i zna da više neće biti vrednosti često je prirodni owner za `close`.

Na primer:

```go
func producer(out chan<- int) {
    defer close(out)

    for i := 0; i < 10; i++ {
        out <- i
    }
}
```

Consumer:

```go
func consumer(in <-chan int) {
    for value := range in {
        process(value)
    }
}
```

Lifecycle izgleda:

```text
producer
   │
   ├── send
   ├── send
   ├── send
   │
   └── close(out)
          │
          ▼
       consumer
          │
          └── range ends
```

Ovo daje vrlo čist protocol:

```text
values
   +
termination signal
```

---

## Pitanje 32: Zašto `close` ne znači "uništi channel"?

`close(ch)` ne uništava channel.

On signalizira:

> "Više vrednosti neće biti poslato."

Na primer:

```go
ch := make(chan int)

close(ch)

value, ok := <-ch
```

dobijamo:

```text
value == 0
ok == false
```

Zato je drugi rezultat veoma važan:

```go
value, ok := <-ch
```

`ok == false` označava da je channel zatvoren i da više nema dostupnih vrednosti.

Kod:

```go
for value := range ch {
    process(value)
}
```

`range` interno prati upravo taj lifecycle.

---

## Pitanje 33: Koji su tipični problemi lošeg channel dizajna?

Loš dizajn može dovesti do:

### Deadlock-a

```go
ch := make(chan int)

ch <- 42
```

Ako nema receiver-a, goroutine blokira.

---

### Panic-a zbog slanja na zatvoren channel

```go
close(ch)

ch <- 42
```

rezultira panic-om.

---

### Double close-a

```go
close(ch)
close(ch)
```

takođe izaziva panic.

---

### Goroutine leak-a

```go
func worker(ch <-chan Job) {
    job := <-ch
    process(job)
}
```

Ako posao nikada ne stigne, worker može ostati blokiran neograničeno dugo.

---

### Neograničenog queue-a

Ako se channel zameni nekontrolisanim mehanizmom za gomilanje posla, sistem može imati:

```text
memory growth
     ↓
latency growth
     ↓
resource exhaustion
```

---

## Pitanje 34: Kako proceniti da li je channel dobar API?

Postavi sledeća pitanja:

1. Ko šalje?
2. Ko prima?
3. Ko zatvara?
4. Kada se zatvara?
5. Šta predstavlja zatvoren channel?
6. Da li je channel buffered?
7. Zašto baš taj kapacitet?
8. Šta se dešava kada je buffer pun?
9. Šta se dešava kada nema podataka?
10. Kako se worker zaustavlja?
11. Ko je owner goroutine-e?
12. Kako se propagiraju greške?
13. Kako se sprečava goroutine leak?
14. Kako se ponaša sistem pod load-om?

Ako odgovor na ova pitanja nije jasan, channel API verovatno još nije dovoljno dobro definisan.

---

## Medior zaključak

Na medior nivou channel treba posmatrati kao **protokol**, a ne samo kao strukturu podataka.

Dobro dizajniran channel sistem definiše:

```text
DATA FLOW
    +
SYNCHRONIZATION
    +
OWNERSHIP
    +
LIFECYCLE
    +
BACKPRESSURE
```

Najvažnije je razumeti da izbor:

```go
make(chan T)
```

nasuprot:

```go
make(chan T, N)
```

nije samo pitanje performansi.

On menja ponašanje sistema.

Isto tako:

```go
chan T
```

nasuprot:

```go
<-chan T
chan<- T
```

nije samo pitanje sintakse.

On definiše granice odgovornosti unutar API-ja.

### Ključne lekcije

* Channel i mutex rešavaju različite klase problema.
* Unbuffered channel predstavlja snažan synchronization rendezvous.
* Buffered channel uvodi vremensko razdvajanje producer-a i consumer-a.
* Kapacitet buffera utiče na backpressure i memory usage.
* Veći buffer ne znači automatski bolje performanse.
* Channel može predstavljati ownership boundary.
* Directional channels poboljšavaju API contract.
* Producer je često prirodni owner zatvaranja output channel-a.
* `close` označava kraj slanja, a ne "uništavanje" channel-a.
* Channel dizajn mora uključiti lifecycle, failure i backpressure ponašanje.

---

Jedna od ključnih razlika između junior i medior nivoa u Go concurrency-ju jeste razumevanje da pokretanje goroutine-e nije problem.

Problem je njeno **upravljanje tokom celog lifecycle-a**.

Kod jednostavnog primera:

```go
go process()
```

goroutine je pokrenuta.

Ali medior developer mora da odgovori na pitanja:

* Kada se goroutine završava?
* Šta ako `process()` blokira?
* Šta ako rezultat više nikome nije potreban?
* Šta ako downstream komponenta prestane da prima?
* Kako goroutine reaguje na cancellation?
* Ko je odgovoran za njeno gašenje?
* Kako znamo da nijedna goroutine nije ostala blokirana?

Concurrency dizajn zato treba posmatrati kroz lifecycle:

```text
START
  │
  ▼
RUNNING
  │
  ├──────────────┐
  │              │
  ▼              ▼
BLOCKED       CANCELLING
  │              │
  └──────┬───────┘
         ▼
      STOPPING
         │
         ▼
     TERMINATED
```

---

## Pitanje 35: Šta je goroutine leak?

Goroutine leak nastaje kada goroutine ostane aktivna ili blokirana iako više nema legitimnu svrhu da nastavi sa radom.

Na primer:

```go
func worker(ch <-chan Job) {
    job := <-ch
    process(job)
}
```

Ako niko nikada ne pošalje `Job`:

```go
go worker(jobs)
```

goroutine ostaje blokirana na:

```go
job := <-ch
```

Ako je aplikacija dugovečna i ovakve goroutine-e se stalno kreiraju, broj goroutine-a može rasti.

Tipičan obrazac:

```text
request
   │
   ▼
goroutine
   │
   ▼
wait forever
```

Ako se ovo ponavlja:

```text
request 1 → leaked goroutine
request 2 → leaked goroutine
request 3 → leaked goroutine
...
request N → leaked goroutine
```

dobijamo resource leak.

---

## Pitanje 36: Da li goroutine leak predstavlja memory leak?

Ne u tradicionalnom smislu.

Goroutine leak je pre svega **lifecycle/resource-management problem**.

Ali leaked goroutine može indirektno izazvati:

* povećanu memorijsku potrošnju,
* rast broja stack-ova,
* zadržavanje objekata dostupnih preko goroutine stack-a,
* povećanje scheduler overhead-a,
* povećanje GC pritiska,
* blokiranje drugih komponenti,
* iscrpljivanje drugih resursa.

Zato je pogrešno posmatrati goroutine kao "besplatnu" jedinicu rada.

Primer:

```text
100 goroutines
    ↓
1,000 goroutines
    ↓
10,000 goroutines
    ↓
100,000 goroutines
```

Čak i ako pojedinačna goroutine koristi relativno malo resursa, sistemski efekat može biti ozbiljan.

---

## Pitanje 37: Kako se goroutine najčešće zaštiti od beskonačnog čekanja?

Jedan od osnovnih mehanizama je `select` sa cancellation signalom.

Na primer:

```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case job := <-jobs:
            process(job)

        case <-ctx.Done():
            return
        }
    }
}
```

Lifecycle sada izgleda:

```text
                  ┌──────────────┐
                  │    worker    │
                  └──────┬───────┘
                         │
                       select
                     /       \
                    /         \
                   ▼           ▼
                 jobs       ctx.Done()
                  │             │
                  ▼             ▼
               process        return
```

Goroutine više nije vezana isključivo za dolazak posla.

Ima definisan cancellation path.

---

## Pitanje 38: Zašto je cancellation path jednako važan kao success path?

Loš concurrency dizajn često razmatra samo:

```text
SUCCESS
```

Na primer:

```go
job := <-jobs
result := process(job)
results <- result
```

Ali production sistem mora da razmotri i:

```text
SUCCESS
ERROR
TIMEOUT
CANCELLATION
SHUTDOWN
DOWNSTREAM FAILURE
```

Ako je `results <- result` blokirajući send, a downstream više ne prima rezultate, goroutine može ostati zauvek blokirana.

Bolji dizajn:

```go
select {
case results <- result:
case <-ctx.Done():
    return
}
```

Sada postoji izlaz i kada normalan tok više nije moguć.

---

## Pitanje 39: Zašto cancellation mora da propagira kroz ceo concurrency graph?

Pretpostavimo sistem:

```text
HTTP request
     │
     ▼
 handler
     │
     ▼
 producer
     │
     ▼
 worker pool
     │
     ▼
 database
```

Ako request bude otkazan, nema smisla da:

```text
worker
producer
database operation
```

nastave da rade beskonačno dugo na rezultatu koji više nikome nije potreban.

Idealno:

```text
request cancelled
       │
       ▼
context cancelled
       │
       ├── producer stops
       ├── workers stop
       └── downstream operations stop
```

Cancellation zato treba da bude deo arhitekture, a ne naknadno dodat `if` uslov.

---

## Pitanje 40: Ko treba da kreira context?

Tipično, najviši sloj koji poseduje lifecycle operacije kreira ili dobija `context.Context`, a niži slojevi ga primaju i propagiraju.

Na primer:

```go
func Handle(ctx context.Context) error {
    return process(ctx)
}
```

zatim:

```go
func process(ctx context.Context) error {
    return worker(ctx)
}
```

i dalje:

```go
func worker(ctx context.Context) error {
    // ...
}
```

Niži sloj ne bi trebalo proizvoljno da kreira potpuno nezavisan context ako treba da poštuje cancellation roditeljske operacije.

Loš obrazac:

```go
func worker(ctx context.Context) {
    ctx = context.Background()
    // ...
}
```

Time se prekida cancellation chain.

---

## Pitanje 41: Šta je razlika između `context.Background()` i `context.TODO()`?

`context.Background()` predstavlja osnovni, prazan root context.

Tipično se koristi na granicama sistema gde zaista nema roditeljskog context-a.

Na primer:

```go
ctx := context.Background()
```

`context.TODO()` se koristi kao privremena oznaka kada još nije jasno koji context treba proslediti.

Na primer:

```go
func legacyFunction() {
    ctx := context.TODO()
    // ...
}
```

`TODO()` nije mehanizam cancellation-a.

Niti predstavlja alternativu za pravilno prosleđivanje context-a.

U production kodu `TODO()` može biti koristan kao signal da neki deo API dizajna još treba refaktorisati.

---

## Pitanje 42: Kako timeout sprečava goroutine leak?

Pretpostavimo operaciju koja može dugo da čeka:

```go
select {
case result := <-results:
    return result

case <-time.After(time.Second):
    return ErrTimeout
}
```

Time se definiše vremenska granica.

Međutim, važno je razumeti jednu stvar:

> Timeout na receiver strani sam po sebi ne garantuje da je producer goroutine zaustavljen.

Na primer:

```go
go func() {
    result := expensiveOperation()
    results <- result
}()
```

Ako caller istekne:

```text
caller
   │
   ├── timeout
   ▼
return
```

producer može i dalje pokušavati:

```text
producer
   │
   ▼
results <- result
   │
   X
no receiver
```

Zato cancellation treba da bude propagiran i producer-u:

```go
go func() {
    result := expensiveOperation(ctx)

    select {
    case results <- result:
    case <-ctx.Done():
        return
    }
}()
```

---

## Pitanje 43: Zašto je `time.After` u loop-u potencijalno loš obrazac?

Kod:

```go
for {
    select {
    case job := <-jobs:
        process(job)

    case <-time.After(time.Second):
        // timeout
    }
}
```

kreira novi timer za svaku iteraciju.

Ako je timeout deo dugotrajnog loop-a, često je bolje koristiti `time.NewTimer` ili `time.NewTicker`, u zavisnosti od semantike koju želimo.

Na primer:

```go
timer := time.NewTimer(time.Second)
defer timer.Stop()

for {
    select {
    case job := <-jobs:
        process(job)

    case <-timer.C:
        return
    }
}
```

Ovde eksplicitnije upravljamo lifecycle-om timer-a.

---

## Pitanje 44: Kako `close` može pomoći u lifecycle management-u?

Ako jedna goroutine proizvodi podatke:

```go
func producer(out chan<- int) {
    defer close(out)

    for i := 0; i < 10; i++ {
        out <- i
    }
}
```

consumer može da završi:

```go
func consumer(in <-chan int) {
    for value := range in {
        process(value)
    }
}
```

Kada producer završi:

```text
producer
   │
   ├── value
   ├── value
   ├── value
   │
   └── close
         │
         ▼
      consumer
         │
         └── range terminates
```

Channel closure ovde predstavlja **termination protocol**.

To je mnogo bolje nego da consumer mora da nagađa:

> "Da li će još neka vrednost stići?"

---

## Pitanje 45: Šta je problem sa goroutine-om koja nema eksplicitan termination condition?

Na primer:

```go
func worker(ch <-chan Job) {
    for {
        job := <-ch
        process(job)
    }
}
```

Ova goroutine nema definisan način da se zaustavi osim ako se kanal ne zatvori — a kod direktnog receive-a zatvaranje channel-a daje zero value, što može biti pogrešno obrađeno ako se ne proverava `ok`.

Bolje:

```go
func worker(ch <-chan Job) {
    for job := range ch {
        process(job)
    }
}
```

ili:

```go
func worker(ctx context.Context, ch <-chan Job) {
    for {
        select {
        case job, ok := <-ch:
            if !ok {
                return
            }

            process(job)

        case <-ctx.Done():
            return
        }
    }
}
```

Druga varijanta eksplicitno podržava oba termination razloga:

```text
channel closed
      OR
context cancelled
      ↓
    return
```

---

## Pitanje 46: Kako prepoznati goroutine leak u realnom sistemu?

Prvi signal često je neočekivan rast:

```go
runtime.NumGoroutine()
```

Na primer:

```go
fmt.Println(runtime.NumGoroutine())
```

Ako broj goroutine-a kontinuirano raste pod ponavljanjem iste operacije:

```text
request 1 → 20 goroutines
request 2 → 25 goroutines
request 3 → 31 goroutines
request 4 → 40 goroutines
...
```

to zahteva istragu.

Ali sam broj goroutine-a nije dokaz leak-a.

Aplikacija može legitimno imati veliki broj aktivnih goroutine-a.

Potrebno je posmatrati:

* lifecycle,
* goroutine stack traces,
* blocking points,
* ownership,
* cancellation,
* workload,
* stabilnost broja goroutine-a nakon završetka operacije.

---

## Pitanje 47: Koji alat je posebno koristan za analizu goroutine leak-ova?

Go runtime profiler, odnosno `pprof`, može prikazati goroutine profile.

Na primer, kroz HTTP pprof endpoint možemo analizirati goroutine stack-ove i videti gde se goroutine-e nalaze.

Tipičan signal:

```text
goroutine 123 [chan receive]
goroutine 124 [select]
goroutine 125 [IO wait]
```

Ako veliki broj goroutine-a ima isti stack i ostaje u istom stanju, to može ukazivati na lifecycle problem.

Važno je analizirati **zašto** su goroutine-e tamo, a ne samo koliko ih postoji.

---

## Pitanje 48: Kako izgleda dobar worker lifecycle?

Dobar worker obično ima jasno definisane faze:

```text
START
  │
  ▼
RUN
  │
  ├── receive job
  ├── process job
  ├── publish result
  │
  ├──────────────┐
  │              │
  ▼              ▼
job channel    cancellation
closed            │
  │               │
  └──────┬────────┘
         ▼
       STOP
```

Na primer:

```go
func worker(
    ctx context.Context,
    jobs <-chan Job,
    results chan<- Result,
) {
    for {
        select {
        case <-ctx.Done():
            return

        case job, ok := <-jobs:
            if !ok {
                return
            }

            result := process(job)

            select {
            case results <- result:
            case <-ctx.Done():
                return
            }
        }
    }
}
```

Ovakav worker ima jasno definisan izlaz:

1. context cancellation,
2. zatvaranje input channel-a,
3. cancellation tokom slanja rezultata.

To je znatno robusnije od:

```go
for {
    job := <-jobs
    results <- process(job)
}
```

---

## Medior zaključak

Concurrency sistem nije dobro dizajniran samo zato što uspešno kreira goroutine-e.

Mora biti jasno definisano:

```text
WHO STARTS?
WHO OWNS?
WHO CANCELS?
WHO STOPS?
WHEN DOES IT STOP?
WHAT HAPPENS IF DOWNSTREAM DISAPPEARS?
```

Najvažniji koncepti ovog nivoa su:

* goroutine lifecycle,
* cancellation propagation,
* timeout,
* graceful termination,
* channel closure,
* goroutine leak prevention,
* observability.

Posebno treba zapamtiti:

> **Svaka dugovečna goroutine-a treba da ima jasan razlog zbog kojeg postoji i jasan način na koji prestaje da postoji.**

Ako ne možemo da odgovorimo na pitanje:

> "Kako ova goroutine-a završava?"

onda concurrency dizajn još nije kompletan.

---

Na medior nivou više nije dovoljno posmatrati jednu goroutine-u ili jedan channel izolovano.

Realni Go sistemi sastoje se od više concurrent komponenti:

```text
        ┌──────────────┐
        │   Producer   │
        └──────┬───────┘
               │
               ▼
        ┌──────────────┐
        │    Queue     │
        │   channel    │
        └──────┬───────┘
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
 Worker 1   Worker 2  Worker 3
       │       │       │
       └───────┼───────┘
               ▼
        ┌──────────────┐
        │    Results   │
        └──────┬───────┘
               ▼
            Consumer
```

Svaka komponenta pojedinačno može biti ispravna, a da njihov **sistem kao celina** ipak bude neispravan.

Zbog toga medior developer mora da razume **composition**:

> Kako se više concurrency primitive-a kombinuje u jedan predvidiv sistem?

---

## Pitanje 49: Šta znači composition u Go concurrency-ju?

Concurrency composition predstavlja kombinovanje više concurrent komponenti tako da zajedno implementiraju širi workflow.

Na primer:

```text
producer
   │
   ▼
worker pool
   │
   ▼
aggregator
   │
   ▼
consumer
```

Svaka komponenta može koristiti:

* goroutine,
* channel,
* `select`,
* `context`,
* `sync`,
* timeout,
* cancellation.

Problem nastaje kada njihove pretpostavke nisu kompatibilne.

Na primer:

```text
 producer
    │
    ▼
   jobs
    │
    ▼
 workers
    │
    ▼
 results
    │
    ▼
aggregator
```

Ako aggregator prestane da prima rezultate, worker može ostati blokiran.

Ako worker prestane, producer može blokirati.

Ako producer ne zatvori `jobs`, worker može čekati zauvek.

Dakle, correctness mora da se posmatra kroz **ceo graph**, a ne samo kroz pojedinačne funkcije.

---

## Pitanje 50: Šta je concurrency protocol?

Concurrency protocol predstavlja skup pravila koja određuju kako concurrent komponente međusobno komuniciraju i kako upravljaju lifecycle-om.

Na primer:

```text
Producer:
    šalje Job
    zatvara jobs

Worker:
    prima Job
    obrađuje Job
    šalje Result
    završava na cancellation

Aggregator:
    prima Result
    završava kada su workers završeni
```

To je implicitni protocol.

Možemo ga formalizovati:

```text
Producer
    │
    │ send Job
    ▼
jobs
    │
    │ receive Job
    ▼
Worker
    │
    │ send Result
    ▼
results
```

A lifecycle:

```text
Producer
   │
   └── close(jobs)
            │
            ▼
        Workers stop
            │
            ▼
      results closed
            │
            ▼
       Aggregator stop
```

Bez ovakvog protokola lako nastaju deadlock-ovi i leak-ovi.

---

## Pitanje 51: Kako se detektuje potencijalni deadlock analizom sistema?

Ne treba tražiti samo:

```go
ch <- value
```

ili:

```go
<-ch
```

nego analizirati **ko čeka koga**.

Na primer:

```text
Producer
   │
   ▼
jobs
   │
   ▼
Worker
   │
   ▼
results
   │
   ▼
Aggregator
```

Ako aggregator čeka producer-a:

```text
Aggregator → Producer
```

dok producer čeka worker-a:

```text
Producer → Worker
```

a worker čeka aggregator-a:

```text
Worker → Aggregator
```

dobijamo ciklus:

```text
Producer
   ↓
Worker
   ↓
Aggregator
   ↓
Producer
```

To je potencijalni deadlock cycle.

Ključna ideja:

> Deadlock nije problem jedne linije koda. To je problem ciklusa čekanja između concurrent komponenti.

---

## Pitanje 52: Šta je "failure mode" concurrency sistema?

Failure mode opisuje način na koji sistem može da prestane da funkcioniše ispravno.

Kod concurrency sistema tipični failure modes su:

### Deadlock

```text
A waits for B
B waits for A
```

### Goroutine leak

```text
goroutine waits forever
```

### Starvation

Jedna goroutine praktično ne dobija priliku da izvrši potreban posao.

### Unbounded queue growth

```text
producer > consumer
```

duže vreme.

### Race condition

Rezultat zavisi od interleaving-a concurrent operacija.

### Data race

Concurrent pristup istoj memorijskoj lokaciji bez odgovarajuće sinhronizacije, pri čemu je najmanje jedan pristup write.

### Premature shutdown

Jedna komponenta se ugasi dok druge još zavise od nje.

### Lost cancellation

Cancellation se ne propagira do komponente koja je blokirana ili izvršava dugotrajnu operaciju.

### Partial failure

Jedan deo sistema otkaže, dok ostali delovi nastavljaju da rade.

---

## Pitanje 53: Šta je starvation?

Starvation nastaje kada goroutine ostane bez adekvatnog pristupa resursu koji joj je potreban, iako sistem kao celina nastavlja da radi.

Na primer:

```text
Worker A ───────────────► CPU
Worker B ───────────────► CPU

Worker C
   │
   └──── čeka ───────────────►
```

Starvation nije isto što i deadlock.

Kod deadlock-a:

```text
niko ne može da napreduje
```

Kod starvation-a:

```text
neki deo sistema napreduje,
ali određena goroutine ne dobija fer priliku
```

U praksi starvation može biti povezan sa:

* lock contention-om,
* nefer raspodelom rada,
* prioritetnim workload-om,
* neadekvatnim scheduling modelom,
* beskonačnim dominantnim workload-om.

---

## Pitanje 54: Kako bounded concurrency utiče na sistem?

Pretpostavimo da aplikacija prima:

```text
100,000 requests
```

i za svaki request pokreće neograničen broj concurrent operacija.

Naivni model:

```text
request
   │
   ├── goroutine
   ├── goroutine
   ├── goroutine
   ├── goroutine
   └── ...
```

može dovesti do prevelikog pritiska na:

* CPU,
* memoriju,
* database connections,
* network connections,
* downstream services.

Bounded concurrency uvodi granicu.

Na primer:

```text
             ┌──────────────┐
requests ───►│ worker pool  │
             │ max = 100    │
             └──────────────┘
```

Sada je maksimalan broj aktivnih worker-a ograničen.

To je posebno važno kod sistema koji komuniciraju sa ograničenim downstream resursima.

---

## Pitanje 55: Da li broj goroutine-a treba uvek ograničiti?

Ne.

Go je dizajniran da podrži veliki broj goroutine-a.

Problem nije:

> "Imamo mnogo goroutine-a."

Problem je:

> "Nemamo kontrolu nad količinom concurrent rada."

Na primer, mnogo goroutine-a koje većinom čekaju na I/O može biti sasvim normalno.

Ali:

```text
1,000,000 goroutines
       │
       ▼
1,000,000 database operations
```

može biti katastrofalno za database.

Zato treba razlikovati:

```text
goroutine count
```

od:

```text
concurrency of expensive work
```

To su dve različite stvari.

---

## Pitanje 56: Kako se sprečava prevelika konkurentnost?

Jedan pristup je worker pool:

```go
const workers = 10

jobs := make(chan Job)

for i := 0; i < workers; i++ {
    go worker(jobs)
}
```

Time dobijamo:

```text
             ┌── worker 1
             ├── worker 2
jobs ────────┼── worker 3
             ├── ...
             └── worker 10
```

Drugi pristup je semaphore pattern:

```go
sem := make(chan struct{}, 10)
```

Pre operacije:

```go
sem <- struct{}{}
```

Nakon operacije:

```go
<-sem
```

Sada najviše 10 operacija može biti aktivno.

---

## Pitanje 57: Kada worker pool može biti bolji od "goroutine per task" pristupa?

Worker pool je posebno koristan kada:

* broj poslova može biti veoma veliki,
* posao je CPU-intensive,
* downstream resurs je ograničen,
* želimo bounded concurrency,
* želimo kontrolisati queue,
* želimo stabilniji memory footprint.

Na primer:

```text
10,000 jobs
     │
     ▼
   queue
     │
     ├── worker 1
     ├── worker 2
     ├── worker 3
     └── worker 4
```

umesto:

```text
10,000 jobs
     │
     ├── goroutine
     ├── goroutine
     ├── goroutine
     ├── ...
     └── 10,000 goroutines
```

Ali worker pool nije univerzalno bolji.

Za jednostavne, kratke i I/O-bound operacije `goroutine-per-task` može biti sasvim odgovarajući model.

---

## Pitanje 58: Kako concurrency sistem treba da reaguje kada downstream komponenta postane spora?

Pretpostavimo:

```text
Producer
  10,000/s
      │
      ▼
Worker
      │
      ▼
Database
  1,000/s
```

Ako nema kontrole:

```text
queue
████████████████████████
████████████████████████
████████████████████████
```

raste dok sistem ne ostane bez resursa.

Robusniji sistem mora da ima strategiju:

* bounded queue,
* backpressure,
* timeout,
* cancellation,
* rate limiting,
* retry policy,
* rejection,
* load shedding.

Na primer:

```text
Producer
   │
   ▼
bounded queue
   │
   ├── capacity available → accept
   │
   └── full → reject / wait / timeout
```

Ovo je veoma važno u production sistemima.

---

## Pitanje 59: Zašto retry može biti opasan u concurrent sistemu?

Pretpostavimo da downstream servis sporadično vraća grešku.

Naivno:

```go
for {
    err := call()
    if err == nil {
        break
    }

    retry()
}
```

Ako veliki broj goroutine-a istovremeno radi retry:

```text
100 requests
    │
    ▼
100 failures
    │
    ▼
100 retries
    │
    ▼
100 failures
    │
    ▼
100 retries
```

dobijamo **retry storm**.

To dodatno opterećuje već problematičan downstream sistem.

Zbog toga retry u concurrent sistemima često treba kombinovati sa:

* exponential backoff,
* jitter,
* maksimalnim brojem pokušaja,
* timeout-om,
* cancellation-om,
* circuit breaker-om ili sličnom zaštitom.

---

## Pitanje 60: Šta je graceful shutdown kod concurrent sistema?

Graceful shutdown znači da sistem ne prekida aktivan rad proizvoljno, već kontroliše:

```text
STOP ACCEPTING NEW WORK
        ↓
FINISH / CANCEL EXISTING WORK
        ↓
RELEASE RESOURCES
        ↓
EXIT
```

Na primer:

```text
HTTP server
    │
    ├── stop accepting requests
    │
    ▼
workers
    │
    ├── finish current jobs
    │
    ▼
close resources
    │
    ▼
process exits
```

Bez graceful shutdown-a moguće su situacije kao:

* izgubljeni poslovi,
* prekinute transakcije,
* nedovršeni network writes,
* napuštene goroutine-e,
* nekonzistentno stanje.

---

## Pitanje 61: Kako se može koordinirati graceful shutdown?

Jedan čest model koristi:

```text
context
+
WaitGroup
+
channel lifecycle
```

Na primer:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    worker(ctx, jobs)
}()

// shutdown
cancel()
wg.Wait()
```

Context signalizira:

```text
STOP
```

a `WaitGroup` omogućava da main/controller sačeka:

```text
ALL WORKERS TERMINATED
```

To su dve različite odgovornosti.

---

## Pitanje 62: Zašto `WaitGroup` nije cancellation primitive?

`sync.WaitGroup` služi za čekanje da grupa goroutine-a završi.

Na primer:

```go
var wg sync.WaitGroup

wg.Add(2)

go func() {
    defer wg.Done()
    work()
}()

go func() {
    defer wg.Done()
    work()
}()

wg.Wait()
```

`Wait()` kaže:

> "Sačekaj da se sve završi."

Ali ne kaže:

> "Zaustavi sve."

Za cancellation je potreban drugi mehanizam, tipično `context.Context` ili drugi eksplicitni signal.

Zato:

```text
WaitGroup
   ↓
WAIT

Context
   ↓
CANCEL
```

često rade zajedno.

---

## Pitanje 63: Kako prepoznati da je concurrency API previše komplikovan?

Jedan od signala je kada korisnik API-ja mora da zna previše internih detalja.

Na primer, ako caller mora ručno da:

```text
start producer
start workers
create channels
close input
close output
cancel context
wait workers
drain results
```

da bi koristio jednostavnu funkcionalnost, abstraction boundary je verovatno loše postavljen.

Dobar API bi trebalo da sakrije deo lifecycle kompleksnosti.

Na primer:

```go
type Processor struct {
    // internal concurrency state
}

func (p *Processor) Process(ctx context.Context, jobs []Job) error
```

Caller ne mora da zna:

* koliko worker-a postoji,
* koliko channel-a postoji,
* kako su worker-i povezani,
* kada se interni channel-i zatvaraju.

Concurrency complexity treba da bude kapsulirana tamo gde je moguće.

---

## Pitanje 64: Koji je najvažniji princip kod composition-a concurrent sistema?

Najvažniji princip je:

> **Svaka komponenta mora imati jasno definisan ownership i lifecycle, a granice između komponenti moraju imati eksplicitan concurrency protocol.**

Možemo ga prikazati:

```text
┌──────────────┐
│  Component A │
│              │
│ ownership    │
│ lifecycle    │
└──────┬───────┘
       │
       │ protocol
       │
       ▼
┌──────────────┐
│  Component B │
│              │
│ ownership    │
│ lifecycle    │
└──────────────┘
```

Ako nije jasno:

* ko šalje,
* ko prima,
* ko zatvara,
* ko cancel-uje,
* ko čeka,
* ko prijavljuje grešku,

sistem će biti teško predvidiv.

---

## Medior mentalni model

Za ozbiljan concurrency dizajn treba razmišljati kroz sledeći model:

```text
             ┌──────────────┐
             │   Producer   │
             └──────┬───────┘
                    │
                DATA FLOW
                    │
                    ▼
             ┌──────────────┐
             │    Queue     │
             └──────┬───────┘
                    │
            CONCURRENCY LIMIT
                    │
                    ▼
          ┌───────────────────┐
          │    Worker Pool    │
          └─────────┬─────────┘
                    │
               RESULT FLOW
                    │
                    ▼
             ┌──────────────┐
             │   Consumer   │
             └──────────────┘

Cancellation ─────────────────► ALL COMPONENTS

Errors ────────────────────────► CONTROL PLANE

Shutdown ──────────────────────► ALL LIFECYCLES

Backpressure ──────────────────► PRODUCER RATE
```

Ovo je mnogo korisniji mentalni model od pukog poznavanja individualnih concurrency primitive-a.

### Ključne lekcije

* Concurrent komponente moraju biti analizirane kao sistem.
* Channel-i predstavljaju deo concurrency protocol-a.
* Deadlock može nastati kroz ciklus čekanja između više komponenti.
* Failure modes moraju biti deo dizajna.
* Bounded concurrency štiti ograničene resurse.
* Worker pool nije automatski bolji od goroutine-per-task modela.
* Backpressure je važan kada producer može biti brži od consumer-a.
* Retry može izazvati retry storm.
* Graceful shutdown zahteva kontrolisan lifecycle.
* `context` i `WaitGroup` rešavaju različite probleme.
* Dobar concurrency API kapsulira internu kompleksnost.

---

[Prelazak na **Medior → Senior — Interview Questions**](../05-medior-to-senior.md)