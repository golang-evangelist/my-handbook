# Module 1 — Interview Questions

> **Fajl:** `extras/01-interview-questions/module-1/03-junior-to-medior.md`
>
> **Nivo:** Junior → Medior
>
> **Oblast:** #1 — Concurrency Fundamentals

---

## 1. Goroutine lifecycle

### Pitanje

Objasni lifecycle jedne goroutine u Go-u.

Koji su osnovni događaji kroz koje goroutine može da prođe od trenutka kada je pokrenuta do trenutka kada završi izvršavanje?

### Odgovor

Goroutine je konkurentna jedinica izvršavanja kojom upravlja Go runtime.

Kada napišemo:

```go
go doWork()
```

Go runtime kreira novu goroutine koja će izvršiti `doWork`.

Važno je razumeti da poziv `go` **ne znači da će funkcija odmah početi da se izvršava**. On samo omogućava runtime-u da novu goroutine rasporedi za izvršavanje.

Pojednostavljeno, lifecycle možemo posmatrati kroz nekoliko stanja:

```text
Created
   │
   ▼
Runnable
   │
   ▼
Running
   │
   ├──────────────┐
   │              │
   ▼              ▼
Blocked        Runnable
   │
   │ unblock
   ▼
Runnable
   │
   ▼
Running
   │
   ▼
Terminated
```

Tipičan tok je:

1. goroutine je kreirana,
2. postaje runnable,
3. scheduler je raspoređuje na izvršavanje,
4. goroutine izvršava svoj kod,
5. može privremeno da bude blokirana,
6. nakon odblokiranja ponovo postaje runnable,
7. scheduler je ponovo pokreće,
8. kada funkcija završi, goroutine terminira.

---

## 2. Goroutine može da bude blokirana

Goroutine ne mora sve vreme da bude u stanju izvršavanja.

Na primer:

```go
ch := make(chan int)

go func() {
    value := <-ch
    fmt.Println(value)
}()
```

Ako niko ne pošalje vrednost u `ch`, goroutine će čekati na receive operaciji.

To znači da goroutine nije aktivno troši CPU dok čeka.

Kasnije:

```go
ch <- 42
```

receive se može kompletirati i goroutine može ponovo postati runnable.

Ovo je jedna od važnih karakteristika Go concurrency modela:

> **Blocking operation ne znači nužno da je ceo program blokiran.**

Blokirana je konkretna goroutine koja čeka određeni događaj.

---

## 3. Goroutine i channel komunikacija

Channels često predstavljaju mehanizam kojim se lifecycle goroutine povezuje sa drugim goroutine-ama.

Na primer:

```go
func worker(ch chan int) {
    value := <-ch
    fmt.Println("received:", value)
}

func main() {
    ch := make(chan int)

    go worker(ch)

    ch <- 10
}
```

Ovde imamo dve goroutine:

```text
main goroutine
      │
      │ send 10
      ▼
   channel
      │
      ▼
worker goroutine
      │
      │ receive
      ▼
    value
```

Worker ne mora da koristi polling:

```go
for {
    if dataAvailable() {
        // ...
    }
}
```

Umesto toga, može da čeka na channel operaciju:

```go
value := <-ch
```

To je mnogo prirodniji model za koordinaciju između goroutine-a.

---

## 4. Lifecycle problem: ko završava goroutine?

Jedno od važnih pitanja na Junior → Medior nivou nije samo:

> "Kako pokrenuti goroutine?"

nego:

> **"Kako znam kada i zašto će goroutine završiti?"**

Na primer, ovaj kod je problematičan:

```go
func startWorker() {
    ch := make(chan int)

    go func() {
        for {
            value := <-ch
            fmt.Println(value)
        }
    }()
}
```

Worker može ostati blokiran ili aktivan praktično neograničeno.

Ako više ne postoji način da dobije podatke ili signal za završetak, napravili smo goroutine čiji lifecycle nije jasno definisan.

To može dovesti do **goroutine leak-a**.

Zato prilikom dizajniranja concurrent koda treba postaviti pitanje:

```text
Ko pokreće goroutine?
        │
        ▼
Ko joj daje posao?
        │
        ▼
Ko zna kada je posao završen?
        │
        ▼
Ko joj govori da treba da završi?
        │
        ▼
Kako garantujemo da će zaista završiti?
```

Ovo je mnogo važnije od samog poznavanja sintakse `go`.

---

## 5. Goroutine lifecycle kao ownership problem

U ozbiljnijem kodu treba da postoji jasno definisan **ownership** goroutine-a.

Na primer:

```go
func startWorker(jobs <-chan Job) {
    go func() {
        for job := range jobs {
            process(job)
        }
    }()
}
```

Ovde `range jobs` implicitno definiše jedan deo lifecycle-a:

```text
jobs open
   │
   ▼
worker receives jobs
   │
   ▼
jobs closed
   │
   ▼
range terminates
   │
   ▼
worker goroutine returns
```

Ako je channel zatvoren:

```go
close(jobs)
```

`range` se završava kada su sve već poslate vrednosti pročitane, nakon čega funkcija može da se završi.

Ovo predstavlja jednostavan primer kako se **channel lifecycle** može koristiti za upravljanje **goroutine lifecycle-om**.

---

## 6. Šta interviewer očekuje od Junior → Medior kandidata?

Na početnom nivou dovoljno je znati:

> "Goroutine je lagana konkurentna jedinica izvršavanja."

Na Junior → Medior nivou očekuje se više.

Kandidat bi trebalo da razume:

* da `go` samo pokreće novu goroutine,
* da scheduler odlučuje kada će se izvršavati,
* da goroutine može biti runnable, running ili blokirana,
* da channel operacije mogu blokirati goroutine,
* da blokiranje jedne goroutine ne mora blokirati ceo program,
* da goroutine može ostati aktivna duže nego što je očekivano,
* šta predstavlja goroutine leak,
* zašto lifecycle mora biti eksplicitno dizajniran,
* kako channel može pomoći u kontrolisanom završavanju goroutine-a.

Drugim rečima:

> **Junior → Medior kandidat ne treba samo da zna kako da pokrene goroutine, već mora da razume ko kontroliše njen životni ciklus i kako ona završava.**

---

## 7. Tipična interview zamka

Interviewer može postaviti pitanje:

> "Ako pokrenemo goroutine pomoću `go`, da li će ona sigurno završiti pre `main` goroutine?"

Odgovor je:

**Ne.**

Na primer:

```go
func main() {
    go func() {
        fmt.Println("hello")
    }()
}
```

Nema garancije da će `main` goroutine sačekati novu goroutine.

Ako `main` završi, proces može završiti pre nego što nova goroutine dobije priliku da izvrši svoj kod.

Zato je pogrešno koristiti:

```go
time.Sleep(time.Second)
```

kao generalno rešenje za koordinaciju goroutine-a.

Za ozbiljnu koordinaciju treba koristiti odgovarajuće synchronization mehanizme, koje ćemo detaljnije obrađivati u kasnijim modulima.

---

## 8. Ključna lekcija

Najvažnija promena u razmišljanju između Junior i Junior → Medior nivoa jeste sledeća:

```text
Junior:

"Kako da pokrenem goroutine?"

        ↓

Junior → Medior:

"Kako da dizajniram njen lifecycle?"
```

Concurrency kod nije samo pitanje paralelnog izvršavanja.

Potrebno je razumeti:

* ko proizvodi podatke,
* ko ih konzumira,
* ko blokira,
* kada se blokiranje završava,
* ko zatvara channel,
* kada goroutine završava,
* šta se dešava ako consumer nestane,
* šta se dešava ako producer nastavi da šalje,
* i kako sprečiti da goroutine ostane zauvek aktivna.

To je osnova za prelazak sa korišćenja concurrency primitives na **dizajniranje pouzdanih concurrent sistema**.

---

U ovom delu prelazimo sa pitanja koja proveravaju **poznavanje osnovnih concurrency mehanizama** na pitanja koja proveravaju da li kandidat zaista razume **ponašanje konkurentnog programa**.

Fokus ovog nivoa je na:

* životnom ciklusu goroutine-a,
* blokiranju,
* međusobnoj zavisnosti goroutine-a,
* deadlock scenarijima,
* ownership modelu nad kanalima,
* pravilnom zatvaranju kanala,
* prepoznavanju goroutine leak-ova,
* razmišljanju o konkurentnom kodu kroz stanje sistema, a ne samo kroz pojedinačne linije koda.

---

# 1. Goroutine lifecycle

## Pitanje 1.v2

**Kada tačno goroutine počinje da postoji i kada se smatra završenom?**

### Odgovor

Goroutine se kreira izvršavanjem `go` naredbe:

```go
go process()
```

U tom trenutku Go runtime kreira goroutine koja će izvršiti funkciju `process`.

Važno je razlikovati **kreiranje goroutine-a** od **početka njenog stvarnog izvršavanja**.

Poziv:

```go
go process()
```

ne znači:

> "Odmah sada izvrši `process`."

On znači:

> "Zakaži izvršavanje `process` u novoj goroutine."

Runtime scheduler odlučuje kada će ta goroutine dobiti priliku da izvršava.

Na primer:

```go
func main() {
    go process()

    fmt.Println("main finished")
}
```

Moguće je da `process()` uopšte ne stigne da se izvrši pre nego što `main` goroutine završi.

Kada `main` goroutine završi izvršavanje, završava se i ceo Go program, bez obzira na to da li druge goroutine još postoje.

Zbog toga sledeći kod nije pouzdan način sinhronizacije:

```go
func main() {
    go process()

    time.Sleep(time.Second)
}
```

`Sleep` može privremeno da spreči završetak `main` goroutine-a, ali ne predstavlja mehanizam kojim se garantuje da je određena goroutine završila posao.

Za eksplicitno čekanje koristi se odgovarajući synchronization mechanism, na primer `sync.WaitGroup`:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    process()
}()

wg.Wait()
```

Ovde postoji jasna relacija:

```text
main goroutine
      │
      │ Add(1)
      ▼
worker goroutine
      │
      │ Done()
      ▼
Wait() se odblokira
```

### Ključna ideja

Goroutine ima životni ciklus koji možemo konceptualno predstaviti kao:

```text
CREATED
   │
   ▼
RUNNABLE
   │
   ▼
RUNNING
   │
   ├──────────────┐
   │              │
   ▼              ▼
BLOCKED         RUNNABLE
   │              │
   └──────►───────┘
          │
          ▼
      TERMINATED
```

Goroutine može više puta prelaziti između stanja u kojima može da izvršava i stanja u kojima čeka na neki događaj.

Na primer, goroutine može biti blokirana zbog:

* receive operacije nad kanalom,
* send operacije nad kanalom,
* `sync.Mutex` zaključavanja,
* `WaitGroup.Wait()`,
* `select` operacije,
* određenih I/O operacija,
* drugih synchronization mehanizama.

---

## Pitanje 2.v2

**Da li `go f()` garantuje da će `f()` biti izvršena pre završetka `main` funkcije?**

### Odgovor

Ne.

Primer:

```go
func main() {
    go func() {
        fmt.Println("hello")
    }()
}
```

Nema garancije da će se:

```text
hello
```

pojaviti na izlazu.

`main` može završiti pre nego što scheduler dobije priliku da izvrši novu goroutine.

Ako je potrebno garantovati izvršavanje i završetak goroutine-a, mora postojati eksplicitna koordinacija:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    fmt.Println("hello")
}()

wg.Wait()
```

Ovo je fundamentalna razlika između:

```go
go f()
```

i:

```go
start f concurrently
and establish a synchronization mechanism
that determines when the work is complete
```

---

# 2. Blocking behavior

## Pitanje 3.v2

**Šta znači da je goroutine blokirana?**

### Odgovor

Blokirana goroutine trenutno ne može da nastavi izvršavanje zato što čeka da se ispuni neki uslov.

Na primer:

```go
ch := make(chan int)

go func() {
    value := <-ch

    fmt.Println(value)
}()
```

Receive:

```go
value := <-ch
```

blokira goroutine dok druga goroutine ne pošalje vrednost:

```go
ch <- 42
```

Dakle:

```text
Worker goroutine
      │
      ▼
   <-ch
      │
      │ BLOCKED
      │
      ▼
   ch <- 42
      │
      ▼
  RUNNABLE
      │
      ▼
   continues
```

Blokiranje samo po sebi **nije problem**.

Concurrency programi često namerno koriste blocking operations.

Problem nastaje kada goroutine ostane blokirana **bez mogućnosti da se uslov ikada ispuni**.

Na primer:

```go
func worker(ch <-chan int) {
    value := <-ch

    fmt.Println(value)
}
```

Ako nijedna goroutine nikada ne pošalje vrednost:

```go
ch <- value
```

worker ostaje blokiran zauvek.

To može biti legitimno ako je to namerno ponašanje dugovečne goroutine.

Ali ako je goroutine trebalo da završi, tada govorimo o potencijalnom **goroutine leak-u**.

---

## Pitanje 4.v2

**Koja je razlika između blokiranja goroutine-a i blokiranja celog programa?**

### Odgovor

Jedna goroutine može biti blokirana dok druge goroutine nastavljaju da rade.

Na primer:

```go
func main() {
    ch := make(chan int)

    go func() {
        <-ch
    }()

    fmt.Println("main continues")
}
```

Worker goroutine je blokiran na:

```go
<-ch
```

ali `main` može nastaviti izvršavanje.

Međutim, ako **sve goroutine** u programu postanu blokirane i nema mogućnosti da se bilo koja od njih odblokira, runtime može detektovati deadlock.

Konceptualno:

```text
G1 ──blocked──► waiting for G2
G2 ──blocked──► waiting for G1
```

ili:

```text
G1 ──► channel receive
G2 ──► channel receive
G3 ──► Wait()
```

a nijedna goroutine ne može da proizvede događaj koji bi omogućio napredovanje.

To je fundamentalna razlika:

```text
blocked goroutine
        ≠
deadlock
```

Blokiranje je stanje pojedinačne goroutine.

Deadlock je stanje sistema u kome nema mogućeg napretka.

---

# 3. Deadlock reasoning

## Pitanje 5.v2

**Šta se dešava u sledećem programu?**

```go
func main() {
    ch := make(chan int)

    ch <- 42

    fmt.Println("done")
}
```

### Odgovor

Program će se blokirati na:

```go
ch <- 42
```

Zato što je `ch` **unbuffered channel**.

Kod unbuffered channel-a send operacija zahteva odgovarajući receive.

Potrebno je da postoji druga goroutine koja izvršava:

```go
<-ch
```

Na primer:

```go
func main() {
    ch := make(chan int)

    go func() {
        value := <-ch
        fmt.Println(value)
    }()

    ch <- 42
}
```

Sada postoji komunikacioni par:

```text
sender                    receiver

ch <- 42  ─────────────►  <-ch
```

Kod prvobitnog programa nema receiver-a:

```text
sender
  │
  ▼
ch <- 42
  │
  X
blocked forever
```

Pošto je `main` jedina aktivna goroutine i ona je blokirana, runtime može prijaviti deadlock.

---

# 4. Channel ownership

## Pitanje 6.v2

**Šta podrazumevamo pod ownership modelom nad kanalom?**

### Odgovor

Ownership model predstavlja način razmišljanja o tome:

> Ko je odgovoran za kreiranje, slanje, zatvaranje i životni ciklus kanala?

Ovo je posebno važno kod većih concurrency sistema.

Tipičan obrazac je:

```go
func generator() <-chan int {
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

Ovde `generator`:

1. kreira channel,
2. jedini proizvodi podatke,
3. zna kada je proizvodnja završena,
4. zatvara channel.

Consumer samo koristi channel:

```go
for value := range generator() {
    fmt.Println(value)
}
```

Consumer **ne mora** da zna kada producer završava.

To je veoma važna arhitektonska osobina.

Možemo je predstaviti ovako:

```text
Producer
   │
   │ owns
   ▼
channel
   │
   │ exposes receive-only view
   ▼
Consumer
```

Producer ima:

```go
chan int
```

dok consumer dobija:

```go
<-chan int
```

Time se ownership i prava pristupa jasno razdvajaju.

---

## Pitanje 7.v2

**Ko bi, po pravilu, trebalo da zatvori channel?**

### Odgovor

Najčešće:

> **Goroutine koja proizvodi vrednosti i zna da više neće biti novih vrednosti treba da zatvori channel.**

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

Ovde producer zna da je završio:

```go
for i := 0; i < 10; i++ {
    ch <- i
}
```

i zato izvršava:

```go
close(ch)
```

Consumer:

```go
for value := range producer() {
    fmt.Println(value)
}
```

ne zatvara channel.

Ovo sprečava česte greške kao što su:

```text
send on closed channel
```

ili:

```text
close of closed channel
```

Praktično pravilo je:

```text
Producer owns sending.
Producer usually owns closing.
Consumer usually only receives.
```

Ovo nije apsolutno pravilo za svaki mogući dizajn, ali predstavlja veoma dobar default za channel-based API-je.

---

## Pitanje 9 — Šta se dešava sa Goroutine-om nakon `go` naredbe?

Kada napišemo:

```go
go process()
```

Go ne izvršava funkciju sinhrono na mestu na kojem se nalazi poziv.

Go Runtime kreira novu Goroutine i predaje je Scheduler-u na izvršavanje.

Pojednostavljeno:

```text
go process()
     │
     ▼
Go Runtime
     │
     ▼
Goroutine
     │
     ▼
Scheduler
     │
     ▼
OS Thread
     │
     ▼
CPU
```

Istovremeno, Goroutine koja je izvršila `go process()` nastavlja sa svojim izvršavanjem.

Na primer:

```go
func main() {
    go process()

    fmt.Println("main continues")
}
```

`main` ne čeka automatski da `process()` završi.

Zbog toga redosled izvršavanja između dve Goroutines nije garantovan bez odgovarajuće koordinacije.

### Ključni odgovor na intervjuu

> `go` pokreće novu Goroutine i omogućava konkurentno izvršavanje. Pozivajuća Goroutine ne čeka automatski da nova Goroutine završi; Scheduler odlučuje kada će nova Goroutine dobiti priliku za izvršavanje.

---

## Pitanje 10 — Da li je pokretanje Goroutine-a isto što i kreiranje OS Thread-a?

Ne.

To je jedna od osnovnih stvari koju Go developer treba da razume.

Goroutine je runtime-managed jedinica izvršavanja, dok je OS Thread resurs kojim upravlja operativni sistem.

Pojednostavljena arhitektura izgleda ovako:

```text
                 Go Runtime
                     │
                 Scheduler
             ┌───────┴───────┐
             │               │
        Goroutine         Goroutine
             │               │
             └───────┬───────┘
                     │
                 OS Threads
                     │
                    CPU
```

Više Goroutines može biti raspoređeno preko manjeg broja OS Thread-ova.

Zbog toga Go program može imati veoma veliki broj Goroutines bez potrebe da za svaku Goroutine postoji poseban OS Thread.

### Šta je važno zapamtiti?

Ne treba razmišljati:

```text
1 Goroutine = 1 Thread
```

nego:

```text
many Goroutines
       ↓
Go Scheduler
       ↓
OS Threads
       ↓
CPU
```

---

## Pitanje 11 — Da li Go garantuje kada će određena Goroutine biti izvršena?

Ne.

Kada napišemo:

```go
go worker()
```

time ne garantujemo da će `worker()` odmah početi da se izvršava.

Garantujemo da je pokrenuta nova Goroutine i da Scheduler može da je rasporedi za izvršavanje.

Na primer:

```go
func main() {
    go func() {
        fmt.Println("worker")
    }()

    fmt.Println("main")
}
```

Mogući izlaz može biti:

```text
main
worker
```

ali program može završiti i pre nego što `worker` dobije priliku da se izvrši.

Zato nije ispravno oslanjati se na slučajni redosled izvršavanja.

---

## Pitanje 12 — Zašto sledeći program možda neće ispisati `Hello from Goroutine`?

```go
func main() {
    go func() {
        fmt.Println("Hello from Goroutine")
    }()

    fmt.Println("Hello from main")
}
```

Zato što se završetkom `main()` Goroutine završava i ceo Go program.

Ako Scheduler ne izvrši novu Goroutine pre završetka `main()`, program se završava.

Model:

```text
main
 │
 ├── go worker()
 │
 ├── nastavlja
 │
 └── završava
        │
        ▼
     program exit
```

`worker` nema poseban mehanizam koji bi sprečio završetak programa.

Zbog toga sledeće nije pouzdan način koordinacije:

```go
go worker()

// nadajmo se da će worker završiti
```

Program mora imati eksplicitni mehanizam koordinacije.

U kasnijim modulima za to ćemo koristiti mehanizme kao što su:

* `sync.WaitGroup`
* `context.Context`
* Channels
* drugi synchronization primitives

---

## Pitanje 13 — Zašto `time.Sleep()` nije dobar način čekanja na Goroutine?

Možemo napisati:

```go
go worker()

time.Sleep(time.Second)
```

ali ovo nije pravi mehanizam sinhronizacije.

Problem je što ne znamo koliko je vremena potrebno `worker` Goroutine-i.

Ako stavimo:

```go
time.Sleep(1 * time.Second)
```

možda će:

```text
worker završiti za 10 ms
```

pa smo nepotrebno čekali.

Ali može se dogoditi i:

```text
worker traje 2 sekunde
```

pa će `main` završiti prerano.

Dakle:

```text
Sleep = čekanje određenog vremena

Synchronization = čekanje određenog događaja/uslova
```

To su dve različite stvari.

### Loš pristup

```go
go worker()

time.Sleep(time.Second)
```

### Ispravan koncept

```text
pokreni worker
       ↓
čekaj da worker završi
       ↓
nastavi
```

U Go-u se za ovo koriste odgovarajući synchronization mehanizmi.

### Važna napomena

`time.Sleep()` nije "zabranjen".

Može biti koristan:

* u demonstracijama,
* testovima određenih vremenskih scenarija,
* simulaciji latencije,
* jednostavnim eksperimentima.

Ali ne treba ga koristiti kao zamenu za mehanizam koordinacije.

---

## Pitanje 14 — Šta znači da je Goroutine blokirana?

Goroutine je blokirana kada trenutno ne može da nastavi izvršavanje zato što čeka neki događaj ili resurs.

Na primer, kod Unbuffered Channel-a:

```go
ch := make(chan int)

go func() {
    value := <-ch
    fmt.Println(value)
}()
```

Ako nema vrednosti koju može da primi, Goroutine čeka.

Slično, Sender može biti blokiran:

```go
ch := make(chan int)

ch <- 42
```

Ako nema odgovarajućeg Receiver-a, slanje na Unbuffered Channel ne može da završi.

Možemo razmišljati ovako:

```text
Goroutine
    │
    ▼
čeka događaj
    │
    ▼
BLOCKED
    │
    ▼
događaj se dogodio
    │
    ▼
READY / RUNNABLE
    │
    ▼
ponovno izvršavanje
```

Ovo je veoma važan koncept jer concurrency problem često nije u samom `go` keyword-u, već u tome **na čemu Goroutine čeka**.

---

## Pitanje 15 — Kako Unbuffered Channel utiče na blokiranje Sender-a?

Unbuffered Channel nema prostor za skladištenje vrednosti.

Na primer:

```go
ch := make(chan int)
```

Ako imamo:

```go
ch <- 42
```

Sender mora da sačeka Receiver-a.

Drugim rečima:

```text
Sender
   │
   │  42
   ▼
Channel
   │
   │
   ▼
Receiver
```

Slanje može da završi tek kada postoji odgovarajuća komunikacija sa Receiver-om.

Zbog toga Unbuffered Channel predstavlja i komunikacioni i sinhronizacioni mehanizam.

### Mentalni model

```text
Unbuffered Channel

Sender ──────► Receiver
        čekanje
```

Za razliku od Buffered Channel-a:

```text
Sender ──────► [ buffer ] ──────► Receiver
```

gde Sender može da nastavi dok god ima prostora u baferu.

---

## Pitanje 16 — Kada će Receiver biti blokiran?

Receiver može biti blokiran kada pokušava da primi vrednost, a odgovarajuća vrednost nije dostupna.

Na primer:

```go
ch := make(chan int)

value := <-ch
```

Ako niko ne šalje vrednost:

```go
ch <- value
```

Receiver čeka.

Model:

```text
Receiver
   │
   ▼
<-ch
   │
   ▼
nema vrednosti
   │
   ▼
BLOCKED
```

Kada drugi učesnik pošalje vrednost:

```go
ch <- 42
```

komunikacija može da se nastavi.

---

## Pitanje 17 — Zašto je razumevanje blokiranja važno za debugging concurrent programa?

Zato što mnogi concurrency problemi zapravo izgledaju kao:

> "Program ništa ne radi."

Ali program možda nije "zaglavljen" u klasičnom smislu.

Možda Goroutine čeka:

* Channel receive,
* Channel send,
* zaključavanje resursa,
* drugi synchronization primitive,
* neki I/O događaj.

Kod Channel komunikacije treba postaviti pitanje:

```text
Ko šalje?
Ko prima?
Kada šalje?
Kada prima?
Da li se obe strane mogu susresti?
```

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()
```

Ako nema Receiver-a, Sender može ostati blokiran.

Ako se ostatak programa oslanja na završetak te Goroutine, problem se može proširiti na ceo sistem.

Zato je razumevanje lifecycle-a i blocking behavior-a osnova za kasnije teme kao što su:

* deadlock,
* Goroutine leak,
* cancellation,
* graceful shutdown,
* Worker Pool,
* Pipeline,
* Fan-in/Fan-out.

---

## Pitanje 18 — Koja je fundamentalna razlika između "Goroutine je završila" i "Goroutine je blokirana"?

To su potpuno različita stanja.

### Završena Goroutine

Goroutine je završila svoje izvršavanje:

```text
 RUNNING
    │
    ▼
  RETURN
    │
    ▼
   DONE
```

Nema više posla.

### Blokirana Goroutine

Goroutine još nije završila, ali trenutno ne može da nastavi:

```text
      RUNNING
        │
        ▼
WAITING / BLOCKED
        │
        ▼
      event
        │
        ▼
     RUNNABLE
        │
        ▼
     RUNNING
```

Ova razlika je ključna.

Ako imamo veliki broj Goroutines koje su završile, to nije isto što i veliki broj Goroutines koje su ostale blokirane.

Veliki broj blokiranih Goroutines može ukazivati na:

* deadlock,
* Goroutine leak,
* nedostajući Receiver,
* nedostajući Sender,
* pogrešan lifecycle management,
* nedostatak cancellation mehanizma.

---

## Ključne lekcije ovog dela

Za nivo **Junior → Medior** nije dovoljno znati samo:

```go
go worker()
```

Potrebno je razumeti šta se događa **nakon** toga.

Treba znati da:

1. `go` kreira novu Goroutine.
2. Go Scheduler odlučuje kada će ona biti izvršena.
3. Goroutine nije isto što i OS Thread.
4. `main()` ne čeka automatski druge Goroutines.
5. `time.Sleep()` nije mehanizam za pouzdanu sinhronizaciju.
6. Goroutine može biti `running`, `blocked/waiting` ili završena.
7. Channel komunikacija može blokirati Sender-a ili Receiver-a.
8. Unbuffered Channel prirodno predstavlja tačku sinhronizacije.
9. Razumevanje blokiranja je osnova za razumevanje deadlock-a i Goroutine leak-ova.

---

## Pitanje 19 — Koja je osnovna razlika između Unbuffered i Buffered Channel-a?

Osnovna razlika je u tome što **Unbuffered Channel nema kapacitet za skladištenje elemenata**, dok Buffered Channel ima unapred definisan kapacitet.

Unbuffered Channel:

```go
ch := make(chan int)
```

Buffered Channel:

```go
ch := make(chan int, 3)
```

Kod Unbuffered Channel-a komunikacija zahteva susret Sender-a i Receiver-a.

Kod Buffered Channel-a vrednost može biti smeštena u buffer sve dok buffer nije pun.

Pojednostavljeno:

```text
Unbuffered:

Sender ─────────────► Receiver
          direktna
         komunikacija
```

Nasuprot tome:

```text
Buffered:

Sender ───► [ 1 | 2 | 3 ] ───► Receiver
             buffer
```

To znači da Buffered Channel može privremeno da razdvoji tempo rada Producer-a i Consumer-a.

---

## Pitanje 20 — Kada se slanje na Unbuffered Channel blokira?

Slanje na Unbuffered Channel blokira se kada nema odgovarajućeg Receiver-a koji može da primi vrednost.

Na primer:

```go
ch := make(chan int)

ch <- 42
```

Ako se ova operacija izvršava u Goroutine-i koja nema odgovarajući Receiver:

```go
<-ch
```

Sender ne može da nastavi.

Mentalni model:

```text
             nema Receiver-a
                    │
                    ▼
Sender ───────► Channel
                    │
                    ▼
                 BLOCKED
```

Kada Receiver postane spreman:

```go
value := <-ch
```

komunikacija može da se izvrši.

Zbog toga Unbuffered Channel uvodi vrlo snažnu sinhronizacionu tačku između dve Goroutines.

---

## Pitanje 21 — Kada se slanje na Buffered Channel blokira?

Kod Buffered Channel-a Sender ne mora odmah da ima Receiver-a.

Na primer:

```go
ch := make(chan int, 3)

ch <- 10
ch <- 20
ch <- 30
```

Sve tri operacije mogu da se izvrše bez trenutnog Receiver-a.

Channel izgleda ovako:

```text
┌───────────────┐
│ 10 │ 20 │ 30  │
└───────────────┘
       full
```

Međutim, sledeće slanje:

```go
ch <- 40
```

blokiraće se dok se ne oslobodi prostor u bufferu.

Dakle:

```text
capacity = 3

send 10 → OK
send 20 → OK
send 30 → OK
send 40 → BLOCKED
```

Kada Receiver primi neku vrednost:

```go
value := <-ch
```

jedno mesto u bufferu se oslobađa i Sender može da nastavi.

---

## Pitanje 22 — Da li Buffered Channel znači da nema blokiranja?

Ne.

Ovo je veoma česta greška u razumevanju Channel-a.

Buffered Channel samo omogućava **određenu količinu asinkronog međuspremanja**.

On ne uklanja mogućnost blokiranja.

Ako imamo:

```go
ch := make(chan int, 2)
```

onda:

```go
ch <- 1
ch <- 2
```

mogu da završe bez Receiver-a.

Ali:

```go
ch <- 3
```

blokira se ako niko ne čita iz Channel-a.

Isto tako, Receiver može da se blokira kada je buffer prazan:

```go
value := <-ch
```

Ako nema vrednosti i nema Sender-a koji može da pošalje vrednost, Receiver čeka.

Dakle, Buffered Channel ima dva važna blocking condition-a:

```text
SEND:
buffer full
    ↓
BLOCKED

RECEIVE:
buffer empty
    ↓
BLOCKED
```

---

## Pitanje 23 — Kako kapacitet Buffered Channel-a utiče na concurrency?

Kapacitet određuje koliko vrednosti može privremeno da postoji između Producer-a i Consumer-a.

Na primer:

```go
ch := make(chan int, 100)
```

Producer može poslati do 100 vrednosti bez trenutnog Receiver-a.

To može biti korisno kada Producer povremeno radi brže od Consumer-a.

Na primer:

```text
Producer
   │
   │  1
   │  2
   │  3
   ▼
┌───────────────┐
│ 1 │ 2 │ 3 │...│
└───────────────┘
          │
          ▼
       Consumer
```

Ali buffer nije beskonačan.

Ako Consumer nastavi da bude spor, buffer će se vremenom napuniti.

Tada se backpressure prenosi nazad ka Producer-u:

```text
Producer
   │
   ▼
BUFFER FULL
   │
   ▼
Producer BLOCKED
```

Ovo je važan koncept u concurrent sistemima.

---

## Pitanje 24 — Šta je backpressure u kontekstu Channel-a?

**Backpressure** je situacija u kojoj sporiji Consumer ograničava tempo Producer-a.

Na primer:

```text
Producer:  ████████████████████
Consumer:  ███████
```

Producer generiše podatke mnogo brže nego što Consumer može da ih obradi.

Ako koristimo Buffered Channel:

```text
Producer
   │
   ▼
┌──────────────────┐
│  buffer          │
│ ████████████████ │
└──────────────────┘
   │
   ▼
Consumer
```

buffer će se postepeno puniti.

Kada se napuni:

```text
Producer
   │
   ▼
BUFFER FULL
   │
   ▼
BLOCKED
```

Na taj način Channel može prirodno da ograniči brzinu Producer-a.

To je često poželjno ponašanje.

Umesto da sistem neograničeno akumulira posao u memoriji, sporiji downstream komponent može da izvrši pritisak nazad ka upstream komponenti.

---

## Pitanje 25 — Zašto veliki Buffer nije automatski bolje rešenje?

Zato što veći buffer samo povećava količinu posla koja može da bude akumulirana.

Na primer:

```go
ch := make(chan Request, 1_000_000)
```

Ovo ne rešava problem ako Consumer ne može dovoljno brzo da obradi Request-ove.

Možemo dobiti:

```text
Producer
   │
   ▼
████████████████████████████
       veliki buffer
████████████████████████████
   │
   ▼
spor Consumer
```

Problemi mogu uključivati:

* povećanu potrošnju memorije,
* povećanu latenciju,
* akumulaciju zastarelih podataka,
* skrivanje problema u downstream komponenti,
* duže vreme do pojave backpressure-a,
* potencijalno gomilanje Goroutines.

Zato kapacitet Channel-a treba da bude rezultat dizajna sistema, a ne nasumičan broj.

---

## Pitanje 26 — Kako bi objasnio odnos između Buffer Capacity i Throughput-a?

Buffer može da pomogne da se Producer i Consumer ne moraju savršeno vremenski poklapati.

Na primer:

```text
Producer ───► Buffer ───► Consumer
```

Ako Producer radi u kratkim burst-ovima, buffer može da apsorbuje te burst-ove.

Bez buffera:

```text
Producer ───────► Consumer
        mora da čeka
```

Sa bufferom:

```text
Producer ───► [buffer] ───► Consumer
                 ↑
          privremena razlika
          u brzini izvršavanja
```

Ali buffer sam po sebi ne povećava fundamentalni processing capacity Consumer-a.

Ako je dugoročni odnos:

```text
Producer rate > Consumer rate
```

onda će buffer na kraju biti pun.

Dakle:

```text
kratkotrajni burst
        +
odgovarajući buffer
        =
manje neposrednog blokiranja
```

dok:

```text
Producer trajno brži od Consumer-a
        +
konačan buffer
        =
backpressure
```

---

## Pitanje 27 — Da li je Channel samo mehanizam za prenos podataka?

Ne.

Channel u Go-u može imati najmanje dve uloge:

1. **komunikacija između Goroutines**
2. **sinhronizacija između Goroutines**

Na primer:

```go
ch := make(chan struct{})

go func() {
    // neki posao
    close(ch)
}()

<-ch
```

Ovde Channel praktično služi kao signal.

Nije potrebno prenositi poslovne podatke.

Možemo koristiti:

```go
chan struct{}
```

kao signalni mehanizam.

Mentalni model:

```text
Goroutine A
    │
    │ signal
    ▼
 Channel
    │
    ▼
Goroutine B
```

Zato Channel treba posmatrati šire od "queue-a za podatke".

---

## Pitanje 28 — Zašto je `chan struct{}` često pogodan za signalizaciju?

`struct{}` je prazan tip koji ne zahteva skladištenje podataka.

Kada nam je potreban samo signal:

```go
done := make(chan struct{})
```

možemo koristiti:

```go
close(done)
```

kao broadcast signal.

Druga Goroutine može čekati:

```go
<-done
```

Nema potrebe da šaljemo stvarnu vrednost.

Ovo jasno izražava nameru:

```text
done
  │
  ▼
"signal je emitovan"
```

umesto:

```go
done <- true
```

gde bi vrednost `true` mogla izgledati kao poslovni podatak.

---

## Pitanje 29 — Šta se dešava kada zatvorimo Channel?

Kada izvršimo:

```go
close(ch)
```

Channel se označava kao zatvoren.

Važno je razumeti da `close` **ne uništava Channel** i ne znači:

> "Channel više ne postoji."

Umesto toga, zatvoren Channel više ne prihvata nove Send operacije.

Receiver-i i dalje mogu da primaju vrednosti koje su već u bufferu.

Na Buffered Channel-u:

```text
pre close:

┌───────────────┐
│ 10 │ 20 │ 30  │
└───────────────┘

close(ch)

┌───────────────┐
│ 10 │ 20 │ 30  │
└───────────────┘
       CLOSED
```

Receiver može nastaviti da čita postojeće vrednosti.

Nakon što se sve vrednosti potroše, receive sa zatvorenog Channel-a vraća zero value uz informaciju da je Channel zatvoren.

Na primer:

```go
value, ok := <-ch
```

Ako je Channel zatvoren i prazan:

```text
value = zero value
ok    = false
```

---

## Pitanje 30 — Ko bi trebalo da zatvara Channel?

Najčešće je dobro pravilo:

> **Goroutine koja je odgovorna za slanje podataka treba da bude odgovorna i za zatvaranje Channel-a.**

Drugim rečima, Producer koji zna da više neće biti novih vrednosti obično zatvara Channel.

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

Consumer samo čita:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Ovde je ownership jasan:

```text
Producer
   │
   ├── send
   ├── send
   ├── send
   │
   └── close
        │
        ▼


Consumer
   │
   └── range until closed
```

Ovaj model smanjuje rizik od pogrešnog zatvaranja Channel-a.

---

## Pitanje 31 — Šta se dešava ako pokušamo da pošaljemo vrednost na zatvoren Channel?

Program će izazvati panic.

Na primer:

```go
ch := make(chan int)

close(ch)

ch <- 42
```

rezultira:

```text
panic: send on closed channel
```

Zato zatvaranje Channel-a predstavlja ozbiljnu promenu njegovog lifecycle-a.

Ne treba zatvarati Channel proizvoljno.

Posebno je opasno kada više Goroutines pokušava da odlučuje kada Channel treba zatvoriti.

---

## Pitanje 32 — Šta se dešava ako dva dela programa pokušavaju da zatvore isti Channel?

Drugo zatvaranje istog Channel-a izaziva panic.

Na primer:

```go
close(ch)
close(ch)
```

rezultira:

```text
panic: close of closed channel
```

Zbog toga je ownership Channel-a važan.

Ako više Goroutines ima pravo da šalje podatke, često nije dobro da svaka od njih samostalno odlučuje:

```go
close(ch)
```

Potrebna je jasna koordinacija oko pitanja:

> Ko je odgovoran za završetak svih Sender-a i zatvaranje Channel-a?

Ovo postaje naročito važno kod:

* Fan-in obrazaca,
* Worker Pool-a,
* Pipeline-a,
* više Producer-a,
* graceful shutdown-a.

---

## Ključni koncepti ovog dela

Za prelaz sa Junior na Medior nivo treba razumeti da Channel nije samo:

```go
ch <- value
```

već sistem sa preciznim blocking semantics.

Najvažniji koncepti su:

### Unbuffered

```text
send ↔ receive
```

Komunikacija predstavlja direktnu sinhronizacionu tačku.

### Buffered

```text
send → buffer → receive
```

Buffer omogućava privremeno razdvajanje brzine Producer-a i Consumer-a.

### Full buffer

```text
send → BLOCKED
```

### Empty buffer

```text
receive → BLOCKED
```

### Backpressure

```text
spor Consumer
      ↓
buffer se puni
      ↓
buffer full
      ↓
Producer BLOCKED
```

### Channel ownership

Treba jasno definisati:

```text
Ko šalje?
Ko prima?
Ko zatvara?
Ko je odgovoran za lifecycle?
```

Upravo ova pitanja predstavljaju osnovu za prelazak sa jednostavnog korišćenja Channel-a na **concurrency design**.

---

## Pitanje 33 — Šta podrazumevamo pod Channel lifecycle-om?

**Channel lifecycle** predstavlja ceo životni ciklus Channel-a:

1. kreiranje,
2. prosleđivanje,
3. slanje i primanje,
4. eventualno zatvaranje,
5. završetak korišćenja.

Najjednostavniji lifecycle izgleda ovako:

```text
make(chan T)
     │
     ▼
Channel created
     │
     ▼
send / receive
     │
     ▼
   close
     │
     ▼
drain remaining values
     │
     ▼
no further communication
```

Važno je da Channel lifecycle bude jasno definisan.

Ako nije jasno ko je odgovoran za završetak komunikacije, mogu nastati:

* deadlock,
* Goroutine leak,
* panic zbog slanja na zatvoren Channel,
* panic zbog višestrukog `close`,
* beskonačno čekanje na `range`.

---

## Pitanje 34 — Da li svaki Channel mora da bude zatvoren?

Ne.

Ovo je jedna od važnijih stvari koje treba razumeti.

Channel se ne zatvara samo zato što je završena neka određena operacija.

Channel treba zatvoriti kada njegovo zatvaranje ima semantičko značenje:

> **"Više neće biti novih vrednosti."**

Na primer, Producer koji generiše konačan broj vrednosti može zatvoriti Channel:

```go
func numbers() <-chan int {
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

Ali Channel koji predstavlja dugotrajni event stream možda neće biti zatvoren sve dok komponenta koja ga poseduje ne završi svoj lifecycle.

Dakle:

```text
Nema novih vrednosti
        ↓
close može imati smisla
```

ali:

```text
"trenutno nemam vrednost"
        ↓
NE znači close
```

---

## Pitanje 35 — Zašto nije dobro zatvarati Channel samo zato što trenutno nema podataka?

Zato što `close` ne znači:

> "Trenutno nema podataka."

On znači:

> "Nikada više neće biti novih podataka."

To je fundamentalna razlika.

Na primer, ako Producer privremeno nema podatke:

```text
Producer
   │
   │ nema trenutno posla
   ▼
Channel
```

to ne znači da treba:

```go
close(ch)
```

Ako Producer kasnije pokuša:

```go
ch <- value
```

dobiće:

```text
panic: send on closed channel
```

Dakle, `close` predstavlja **terminal state** Channel-a.

---

## Pitanje 36 — Kako `range` radi sa Channel-om?

Možemo iterirati preko Channel-a:

```go
for value := range ch {
    fmt.Println(value)
}
```

`range` nastavlja da prima vrednosti sve dok Channel ne bude zatvoren.

Na primer:

```go
ch := make(chan int)

go func() {
    defer close(ch)

    for i := 0; i < 5; i++ {
        ch <- i
    }
}()

for value := range ch {
    fmt.Println(value)
}
```

Lifecycle je:

```text
Producer
   │
   ├── send 0
   ├── send 1
   ├── send 2
   ├── send 3
   ├── send 4
   │
   └── close(ch)
          │
          ▼
       Consumer
          │
          └── range završava
```

`range` koristi zatvaranje Channel-a kao signal da više nema vrednosti.

---

## Pitanje 37 — Šta se dešava ako koristimo `range` nad Channel-om koji nikada neće biti zatvoren?

Consumer može zauvek čekati.

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 1
}()
```

Ako imamo:

```go
for value := range ch {
    fmt.Println(value)
}
```

dobijamo:

```text
1
^
│
└── Channel nikada nije zatvoren
```

Nakon što se `1` primi, `range` pokušava sledeći receive.

Pošto nema novih vrednosti i Channel nije zatvoren:

```text
range
  │
  ▼
receive
  │
  ▼
BLOCKED FOREVER
```

Ovo je tipičan izvor Goroutine leak-a ili deadlock-a, zavisno od ostatka programa.

---

## Pitanje 38 — Kako se rešava problem kada postoji više Producer-a?

Kada više Goroutines šalje vrednosti u isti Channel, ne treba dozvoliti da svaka od njih samostalno zatvara Channel.

Na primer:

```text
Producer A ──┐
             │
Producer B ──┼──► Channel ───► Consumer
             │
Producer C ──┘
```

Problem je:

> Ko zatvara Channel?

Ako Producer A uradi:

```go
close(ch)
```

dok Producer B još šalje:

```go
ch <- value
```

Producer B može izazvati:

```text
panic: send on closed channel
```

Potrebna je koordinacija.

Jedan tipičan obrazac koristi `sync.WaitGroup`.

```go
var wg sync.WaitGroup

for i := 0; i < 3; i++ {
    wg.Add(1)

    go func(id int) {
        defer wg.Done()

        // send values
        ch <- id
    }(i)
}

go func() {
    wg.Wait()
    close(ch)
}()
```

Ovde je lifecycle mnogo jasniji:

```text
Producer A ──┐
Producer B ──┼──► Channel
Producer C ──┘
       │
       ▼
   WaitGroup
       │
       ▼
   svi Producer-i
   završili rad
       │
       ▼
    close(ch)
```

---

## Pitanje 39 — Zašto je ovaj obrazac sa `WaitGroup` važan?

Zato što odvaja dve odgovornosti:

**Producer-i**:

```text
proizvode podatke
```

**Coordinator**:

```text
čeka da svi Producer-i završe
```

**Coordinator zatim**:

```text
close(ch)
```

Na taj način se dobija jasna ownership struktura.

```text
             ┌── Producer A
             │
             ├── Producer B
             │
             └── Producer C
                    │
                    ▼
               WaitGroup
                    │
                    ▼
                 close
                    │
                    ▼
                 Consumer
```

Ovo je naročito korisno kod **Fan-in** obrazaca, gde više izvora šalje podatke u jedan zajednički Channel.

---

## Pitanje 40 — Šta znači Channel ownership?

Channel ownership predstavlja jasno definisanu odgovornost nad Channel-om.

Treba da znamo:

* ko ga kreira,
* ko šalje,
* ko prima,
* ko ga zatvara,
* ko odlučuje kada komunikacija završava.

Jedan veoma koristan mentalni model je:

```text
Creator
   │
   ▼
Producer ───► Channel ───► Consumer
   │
   └── ownership nad send side
```

Ako funkcija kreira Channel i vraća ga kao receive-only:

```go
func generate() <-chan int
```

onda API jasno govori:

> Pozivalac može da prima, ali ne treba da šalje niti da zatvara Channel.

To je mnogo sigurnije od:

```go
func generate() chan int
```

jer drugi kod tada dobija mogućnost da radi:

```go
ch <- value
close(ch)
```

čime se ownership granica gubi.

---

## Pitanje 41 — Kako Channel directions pomažu u ownership modelu?

Go omogućava:

```go
chan T
chan<- T
<-chan T
```

Tri oblika predstavljaju:

```text
chan T
  │
  ├── send
  └── receive
```

```text
chan<- T
  │
  └── send only
```

```text
<-chan T
  │
  └── receive only
```

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

Pozivalac može:

```go
for value := range producer() {
    fmt.Println(value)
}
```

ali ne može:

```go
close(ch)
```

ako ima samo `<-chan int`.

Ovo predstavlja oblik **compile-time enforcement-a ownership-a**.

---

## Pitanje 42 — Da li `close` treba posmatrati kao operaciju Producer-a ili Consumer-a?

U tipičnom Producer/Consumer modelu, `close` pripada strani koja zna da više neće biti novih vrednosti.

Najčešće je to Producer.

```text
Producer
   │
   ├── value
   ├── value
   ├── value
   │
   └── close
        │
        ▼
Consumer
   │
   └── range završava
```

Consumer obično nema dovoljno informacija da odluči da li će Producer kasnije poslati još nešto.

Zato Consumer najčešće treba da **reaguje na zatvaranje**, a ne da ga sam izaziva.

---

## Pitanje 43 — Kako se razlikuju "producer finished" i "consumer finished"?

To su dve potpuno različite stvari.

Producer završava kada:

```text
nema više vrednosti koje može da proizvede
```

Consumer završava kada:

```text
više nema potrebe da obrađuje vrednosti
```

Na primer:

```text
Producer:
1000 items
    │
    ▼
Channel
    │
    ▼
Consumer:
obrađuje samo prvih 100
```

Consumer može završiti pre Producer-a.

Tada nastaje problem:

```text
Consumer exits
     │
     ▼
Producer sends
     │
     ▼
nema Receiver-a
     │
     ▼
Producer BLOCKED
```

Ovo je jedan od važnih razloga zašto lifecycle Channel-a ne treba posmatrati izolovano.

Potrebno je razmišljati o lifecycle-u svih Goroutines koje učestvuju u komunikaciji.

---

## Pitanje 44 — Kako Consumer može da prekine rad bez ostavljanja Producer-a blokiranog?

Jedan od standardnih pristupa je uvođenje cancellation mehanizma.

Na primer, korišćenjem `context.Context`:

```go
func producer(ctx context.Context) <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; ; i++ {
            select {
            case ch <- i:
            case <-ctx.Done():
                return
            }
        }
    }()

    return ch
}
```

Sada Producer ne čeka samo:

```go
ch <- i
```

već može da reaguje i na cancellation:

```text
                 ┌── send ─────────► Channel
Producer ─ select
                 └── ctx.Done() ──► exit
```

Ovo sprečava situaciju u kojoj Consumer nestane, a Producer ostane zauvek blokiran.

---

## Pitanje 45 — Šta je Goroutine leak?

**Goroutine leak** nastaje kada Goroutine ostane aktivna i čeka resurs ili događaj koji se više nikada neće dogoditi.

Channel je jedan od čestih uzroka.

Na primer:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        fmt.Println(value)
    }
}
```

Ako niko više neće slati:

```text
worker
  │
  ▼
receive
  │
  ▼
BLOCKED
  │
  └── zauvek
```

Goroutine ostaje živa iako više nema korisnu funkciju.

Ako se takav obrazac ponavlja, broj Goroutines može kontinuirano rasti.

---

## Pitanje 46 — Kako `close` može sprečiti Goroutine leak kod Consumer-a?

Ako Consumer koristi:

```go
for value := range ch {
    process(value)
}
```

zatvaranje Channel-a predstavlja signal:

```text
Producer
   │
   └── close(ch)
          │
          ▼
Consumer range
          │
          ▼
       EXIT LOOP
          │
          ▼
     Goroutine ends
```

Zato je pravilno zatvaranje Channel-a često sastavni deo lifecycle management-a.

Ali samo `close` nije dovoljan za sve situacije.

Ako Consumer može završiti pre Producer-a, Producer-u je često potreban cancellation signal.

---

## Pitanje 47 — Koji je najvažniji princip za bezbedan Channel lifecycle?

Najvažniji princip je:

> **Channel mora imati jasno definisan ownership i lifecycle.**

Pre nego što napišemo:

```go
ch := make(chan T)
```

trebalo bi da znamo:

```text
1. Ko kreira Channel?
2. Ko šalje?
3. Ko prima?
4. Ko zatvara?
5. Ko odlučuje kada je slanje završeno?
6. Šta se dešava ako Consumer odustane?
7. Šta se dešava ako Producer odustane?
8. Kako se Goroutines završavaju?
```

Ako na ova pitanja nemamo jasan odgovor, concurrency dizajn verovatno još nije dovoljno definisan.

---

## Sažetak

Na Junior → Medior nivou Channel treba posmatrati kao komponentu sa jasno definisanim lifecycle-om.

Ključni koncepti su:

```text
Channel lifecycle
       │
       ├── creation
       ├── ownership
       ├── send
       ├── receive
       ├── close
       └── termination
```

Posebno treba razumeti:

* Channel ne mora uvek da bude zatvoren.
* `close` znači da više neće biti novih vrednosti.
* `range` očekuje završetak Channel lifecycle-a.
* Više Producer-a zahteva koordinaciju oko `close`.
* `WaitGroup` može koordinisati završetak više Producer-a.
* Channel directions mogu da ograniče ownership.
* Consumer koji prerano završi može ostaviti Producer-a blokiranog.
* Cancellation je često potreban za bezbedan lifecycle.
* Neodgovarajuće upravljanje Channel lifecycle-om može proizvesti Goroutine leak.

Najvažnije pitanje više nije samo:

> **"Kako koristim Channel?"**

nego:

> **"Ko je vlasnik komunikacije i kako se garantuje da će svaka Goroutine moći da završi svoj lifecycle?"**

---

## Pitanje 48 — Šta je deadlock u Go programu?

**Deadlock** nastaje kada Goroutines čekaju jedna na drugu na način iz kog nijedna ne može da nastavi izvršavanje.

Najjednostavniji primer:

```go
func main() {
    ch := make(chan int)

    ch <- 42

    fmt.Println("done")
}
```

Channel je nebufferovan.

Operacija:

```go
ch <- 42
```

zahteva odgovarajući Receiver.

Ali Receiver ne postoji.

Zato `main` blokira.

Program završava sa runtime porukom:

```text
fatal error: all goroutines are asleep - deadlock!
```

Mentalni model:

```text
main
 │
 └── send
      │
      ▼
   Channel
      │
      └── nema Receiver-a
              │
              ▼
           BLOCKED
```

---

## Pitanje 49 — Koja je razlika između blocking-a i deadlock-a?

**Blocking** znači da Goroutine trenutno ne može da nastavi.

To samo po sebi nije problem.

Na primer:

```go
value := <-ch
```

može legitimno da blokira dok Producer ne pošalje vrednost.

```text
Consumer
   │
   ▼
receive
   │
   ▼
WAIT
   │
   │ Producer šalje
   ▼
CONTINUE
```

To je normalna sinhronizacija.

**Deadlock** nastaje kada čekanje nema mogućnost da bude razrešeno.

```text
Consumer
   │
   ▼
receive
   │
   ▼
WAIT FOREVER
   ▲
   │
   └── nema Producer-a
```

Dakle:

> **Svaki deadlock uključuje blocking, ali svaki blocking nije deadlock.**

---

## Pitanje 50 — Kako izgleda deadlock sa nebufferovanim Channel-om?

Neurobufferovani Channel zahteva istovremeni handshake između Sender-a i Receiver-a.

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 10
}()

value := <-ch
```

Ovde nema problema.

Sender:

```text
ch <- 10
```

čeka dok Receiver:

```text
<-ch
```

ne bude spreman.

Ali ako imamo:

```go
ch := make(chan int)

go func() {
    <-ch
}()

ch <- 10
```

i dalje nema problema.

Sender i Receiver mogu da se sinhronizuju.

Problem nastaje kada postoji samo jedna strana:

```go
ch := make(chan int)

ch <- 10
```

Nema Receiver-a.

---

## Pitanje 51 — Može li buffered Channel da izazove deadlock?

Da.

Buffer ne uklanja mogućnost deadlock-a.

Na primer:

```go
ch := make(chan int, 1)

ch <- 10
ch <- 20
```

Prvi send uspeva:

```text
Buffer:
[10]
```

Drugi send mora da čeka jer je buffer pun:

```text
Buffer:
[10] ← full

ch <- 20
      │
      ▼
   BLOCKED
```

Ako nema Receiver-a koji će ukloniti `10`, druga operacija nikada neće moći da završi.

Dakle:

> Buffered Channel pomera trenutak blokiranja, ali ne garantuje odsustvo deadlock-a.

---

## Pitanje 52 — Kako izgleda deadlock kada su i Sender i Receiver blokirani?

Jedan primer je lanac Channel-a:

```text
Goroutine A
    │
    ▼
Channel 1
    │
    ▼
Goroutine B
    │
    ▼
Channel 2
    │
    ▼
Goroutine A
```

Ako A čeka Channel 1, a B čeka Channel 2, može nastati ciklus:

```text
A waits for B
     ▲       │
     │       ▼
B waits for A
```

Nijedna Goroutine ne može da nastavi.

To je **circular wait**.

---

## Pitanje 53 — Šta je circular wait?

**Circular wait** nastaje kada postoji ciklus u odnosima čekanja.

Na primer:

```text
Goroutine A
    │
    │ waits for
    ▼
Goroutine B
    │
    │ waits for
    ▼
Goroutine C
    │
    │ waits for
    ▼
Goroutine A
```

Sada imamo:

```text
A → B → C → A
```

Svaka Goroutine čeka nešto što zavisi od sledeće Goroutine.

Pošto se ciklus zatvara, niko ne može da napravi progress.

---

## Pitanje 54 — Kako se deadlock može javiti korišćenjem više Channel-a?

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

Na prvi pogled deluje da dve Goroutines međusobno komuniciraju.

Ali obe prvo pokušavaju send na nebufferovane Channel-e:

```text
Goroutine A ──send──► ch1
                         ▲
                         │
                    receiver?
```

i:

```text
Goroutine B ──send──► ch2
                         ▲
                         │
                    receiver?
```

Obe čekaju Receiver-a.

A Receiver se nalazi tek u drugom delu iste Goroutine.

Do njega se ne dolazi dok prvi send ne završi.

Rezultat:

```text
A waits on ch1
B waits on ch2

A cannot reach receive ch2
B cannot reach receive ch1
```

To je deadlock.

---

## Pitanje 55 — Kako prepoznati deadlock analizom Channel operacija?

Jedan praktičan pristup je da za svaku Channel operaciju napišemo:

```text
Goroutine
    │
    ├── SEND
    │
    └── RECEIVE
```

zatim utvrdimo ko predstavlja drugu stranu komunikacije.

Na primer:

```text
G1:
    send ch1
    receive ch2

G2:
    send ch2
    receive ch1
```

Zatim analiziramo prvu blocking operaciju:

```text
G1 → send ch1 → needs receiver
G2 → send ch2 → needs receiver
```

Ali:

```text
G1 receiver ch2
```

dolazi tek nakon `send ch1`.

I:

```text
G2 receiver ch1
```

dolazi tek nakon `send ch2`.

Dakle:

```text
G1
 │
 └── waits for G2

G2
 │
 └── waits for G1
```

Deadlock je vidljiv već iz redosleda operacija.

---

## Pitanje 56 — Da li `select` automatski sprečava deadlock?

Ne.

`select` omogućava da Goroutine čeka na više komunikacionih operacija, ali ne garantuje da će neka od njih ikada postati spremna.

Na primer:

```go
select {
case value := <-ch1:
    fmt.Println(value)

case value := <-ch2:
    fmt.Println(value)
}
```

Ako:

* `ch1` nikada ne dobije vrednost,
* `ch2` nikada ne dobije vrednost,

onda `select` čeka.

```text
          select
         /      \
        /        \
     ch1          ch2
      │            │
   blocked       blocked
        \        /
         \      /
          WAIT
```

Ako postoji mogućnost da nijedna grana nikada neće postati spremna, program i dalje može imati deadlock.

---

## Pitanje 57 — Čemu služi `default` u `select` konstrukciji?

`default` omogućava non-blocking ponašanje.

Na primer:

```go
select {
case value := <-ch:
    fmt.Println(value)

default:
    fmt.Println("nothing available")
}
```

Ako receive trenutno nije moguć:

```text
select
 │
 ├── receive unavailable
 │
 └── default
        │
        ▼
     continue
```

Goroutine ne blokira.

Ali `default` treba koristiti pažljivo.

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

može napraviti **busy loop**.

Goroutine tada stalno izvršava:

```text
select
select
select
select
select
...
```

i nepotrebno troši CPU.

Zato:

> Non-blocking nije isto što i efikasno.

---

## Pitanje 58 — Kako timeout može sprečiti beskonačno čekanje?

`select` se često kombinuje sa `time.After`:

```go
select {
case value := <-ch:
    fmt.Println(value)

case <-time.After(time.Second):
    fmt.Println("timeout")
}
```

Sada postoje dva ishoda:

```text
                select
               /      \
              /        \
          ch receive   timeout
             │           │
             ▼           ▼
          success      failure
```

Ako Channel ne postane spreman u definisanom vremenu, timeout grana omogućava Goroutine-i da nastavi.

Međutim, timeout ne rešava automatski sve lifecycle probleme.

Ako druga Goroutine nastavi da radi i nakon timeout-a, ona i dalje može ostati blokirana.

Zato timeout treba posmatrati kao deo šireg cancellation/lifecycle dizajna.

---

## Pitanje 59 — Da li timeout garantuje da neće biti Goroutine leak-a?

Ne.

Na primer:

```go
func startWorker() <-chan int {
    ch := make(chan int)

    go func() {
        time.Sleep(time.Hour)
        ch <- 42
    }()

    return ch
}
```

Consumer može uraditi:

```go
select {
case value := <-startWorker():
    fmt.Println(value)

case <-time.After(time.Second):
    fmt.Println("timeout")
}
```

Consumer završava posle jedne sekunde.

Ali Worker i dalje postoji.

Nakon sat vremena pokušava:

```go
ch <- 42
```

Ako više nema Receiver-a, Worker može ostati blokiran zauvek.

Dakle:

```text
Consumer
   │
   └── timeout
        │
        └── exits

Worker
   │
   └── still running
        │
        └── eventually blocks
```

Ovo je klasičan primer zašto **timeout ≠ cancellation**.

---

## Pitanje 60 — Koja je razlika između timeout-a i cancellation-a?

**Timeout** kaže:

> "Neću čekati duže od X vremena."

**Cancellation** kaže:

> "Prekini rad zato što više nema potrebe da nastavljaš."

Timeout:

```text
Consumer
   │
   ├── wait
   │
   └── timeout
```

Cancellation:

```text
Controller
   │
   └── cancel
        │
        ├── Producer
        ├── Worker
        └── Consumer
```

Kod pravilnog cancellation dizajna sve relevantne Goroutines dobijaju signal da treba da završe.

---

## Pitanje 61 — Kako izgleda tipičan Channel deadlock izazvan pogrešnim redosledom operacija?

Na primer:

```go
func main() {
    ch := make(chan int)

    go func() {
        ch <- 1
        ch <- 2
    }()

    value := <-ch
    fmt.Println(value)
}
```

Ovaj kod ne mora biti deadlock.

Consumer primi `1`:

```text
Producer ──1──► Consumer
```

Producer zatim pokušava:

```go
ch <- 2
```

Ako Consumer više ne prima, Producer ostaje blokiran.

Ali `main` može završiti pre nego što runtime prijavi deadlock, jer se program završava kada `main` završi.

Ovo pokazuje još jednu važnu stvar:

> Nije dovoljno analizirati samo da li je neka Goroutine blokirana. Moramo razumeti lifecycle celog programa.

---

## Pitanje 62 — Zašto je `main` poseban u concurrency analizi?

Zato što završetak `main` funkcije završava proces.

Na primer:

```go
func main() {
    go func() {
        for {
            fmt.Println("working")
        }
    }()
}
```

`main` može završiti gotovo odmah.

To znači da program ne čeka Worker Goroutine.

Dakle:

```text
main
 │
 └── exits
      │
      ▼
   process exits
      │
      └── worker stops
```

Ovo nije isto što i uredno gašenje svih Goroutines.

U ozbiljnim programima lifecycle aplikacije mora eksplicitno da definiše kako se Goroutines pokreću i završavaju.

---

## Pitanje 63 — Kako možemo sistematski analizirati deadlock?

Koristan postupak je sledeći.

### Korak 1 — Identifikuj sve Goroutines

```text
G1
G2
G3
...
```

### Korak 2 — Identifikuj operaciju na kojoj svaka može da blokira

Na primer:

```text
G1 → send ch1
G2 → receive ch1
G3 → receive ch2
```

### Korak 3 — Utvrdi šta svaka operacija zahteva

Na primer:

```text
send ch1
    ↓
requires receiver

receive ch2
    ↓
requires sender
```

### Korak 4 — Napravi dependency graph

```text
G1 ──waits for──► G2
G2 ──waits for──► G3
G3 ──waits for──► G1
```

### Korak 5 — Potraži ciklus

Ako postoji:

```text
G1 → G2 → G3 → G1
```

postoji potencijal za deadlock.

Ovaj način razmišljanja je mnogo korisniji od pokušaja da se deadlock rešava nasumičnim dodavanjem buffer-a, `Sleep` poziva ili `default` grana.

---

## Pitanje 64 — Zašto dodavanje buffer-a često samo prikrije problem?

Pretpostavimo:

```go
ch := make(chan int)
```

i program blokira na:

```go
ch <- value
```

Developer može promeniti:

```go
ch := make(chan int, 100)
```

Program možda više neće odmah blokirati.

Ali problem može ostati:

```text
Producer
   │
   ├── 1
   ├── 2
   ├── ...
   └── 101
         │
         ▼
      buffer full
         │
         ▼
       BLOCK
```

Buffer je promenio kapacitet sistema, ali nije nužno rešio lifecycle problem.

Zato buffer treba da bude deo dizajna protoka podataka, a ne "fix" za nejasan deadlock.

---

## Pitanje 65 — Šta je najvažnije pitanje kada vidiš blokiranu Goroutine?

Nemoj odmah pitati:

> "Kako da uklonim blocking?"

Prvo pitaj:

> **"Na koji događaj ova Goroutine čeka i ko garantuje da će se taj događaj dogoditi?"**

Na primer:

```text
Goroutine
   │
   ▼
receive ch
   │
   ▼
Čeka vrednost
   │
   ▼
Ko šalje?
   │
   ▼
Da li će taj Producer sigurno završiti send?
```

Ako odgovor glasi:

```text
"Nisam siguran."
```

onda je to potencijalni lifecycle problem.

---

## Sažetak

Na Junior → Medior nivou potrebno je razlikovati:

| Koncept        | Značenje                                                   |
| -------------- | ---------------------------------------------------------- |
| Blocking       | Goroutine trenutno čeka                                    |
| Deadlock       | Čekanje više ne može biti razrešeno                        |
| Circular wait  | Goroutines čekaju jedna drugu u ciklusu                    |
| Timeout        | Ograničava koliko dugo čekamo                              |
| Cancellation   | Signalizira da rad treba prekinuti                         |
| Buffer         | Omogućava određeni broj asinhronih send operacija          |
| Goroutine leak | Goroutine ostaje aktivna bez mogućnosti korisnog završetka |

Najvažniji mentalni model je:

```text
Blocking
   │
   ▼
Šta čekamo?
   │
   ▼
Ko treba da omogući progress?
   │
   ▼
Da li će se taj događaj sigurno dogoditi?
   │
   ├── DA ──► normalna sinhronizacija
   │
   └── NE ──► deadlock / leak / lifecycle problem
```

Dobro poznavanje Concurrency-ja počinje kada prestanemo da posmatramo `send` i `receive` kao izolovane operacije i počnemo da posmatramo **dependency graph između Goroutines**.

---

### 11. Kako biste dizajnirali funkciju koja pokreće goroutine, ali garantuje da će ta goroutine u nekom trenutku završiti?

Jedan od najvažnijih principa konkurentnog programiranja u Go-u jeste:

> **Kod koji pokrene goroutine mora imati jasan model njenog završetka.**

Samo pokretanje goroutine-a nije problem:

```go
go process()
```

Problem nastaje kada `process()` nema definisan način da prekine rad.

Na primer:

```go
func process(ch <-chan int) {
    for {
        value := <-ch
        fmt.Println(value)
    }
}
```

Ova goroutine može ostati blokirana zauvek ako niko više ne šalje podatke.

Bolji dizajn uvodi eksplicitni signal za prekid.

Jedan od klasičnih obrazaca jeste korišćenje `context.Context`:

```go
func process(ctx context.Context, ch <-chan int) {
    for {
        select {
        case <-ctx.Done():
            return

        case value := <-ch:
            fmt.Println(value)
        }
    }
}
```

Sada caller kontroliše životni ciklus:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go process(ctx, ch)
```

Kada se pozove:

```go
cancel()
```

goroutine dobija signal preko:

```go
<-ctx.Done()
```

i završava izvršavanje.

Važno je razumeti da `context` **ne ubija goroutine**. On samo omogućava da goroutine sazna da treba da prekine rad.

Odgovornost goroutine-a je da taj signal obradi.

---

### 12. Zašto je `for {}` bez cancellation mehanizma potencijalni concurrency problem?

Sledeći kod izgleda jednostavno:

```go
func worker(ch <-chan Job) {
    for {
        job := <-ch
        process(job)
    }
}
```

Ali postoji fundamentalni problem:

**Ko garantuje da će ova goroutine ikada završiti?**

Ako se `ch` nikada ne zatvori i više nema send operacija, goroutine ostaje blokirana na:

```go
job := <-ch
```

To može predstavljati goroutine leak.

Još ozbiljniji primer je:

```go
func worker(ch <-chan Job) {
    for {
        select {
        case job := <-ch:
            process(job)
        }
    }
}
```

Dodavanje `select` ovde nije rešilo problem.

Goroutine i dalje nema izlaznu granu.

Robusniji dizajn:

```go
func worker(ctx context.Context, ch <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return

        case job, ok := <-ch:
            if !ok {
                return
            }

            process(job)
        }
    }
}
```

Ovde postoje dva legitimna načina završetka:

1. cancellation preko `ctx`
2. zatvaranje input kanala

To je mnogo sigurniji lifecycle model.

---

### 13. Šta je goroutine leak?

**Goroutine leak** nastaje kada goroutine ostane aktivna ili blokirana iako više nema korisnog posla koji treba da obavlja.

Primer:

```go
func generator() <-chan int {
    ch := make(chan int)

    go func() {
        for i := 0; ; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Caller može da pročita samo nekoliko vrednosti:

```go
ch := generator()

fmt.Println(<-ch)
fmt.Println(<-ch)
fmt.Println(<-ch)
```

a zatim da prestane da čita.

Producer će tada vrlo verovatno ostati blokiran na:

```go
ch <- i
```

i goroutine neće moći da završi.

Problem nije samo "jedna goroutine koja je ostala".

U dugotrajnim servisima ponavljanje ovakvog obrasca može dovesti do:

* rasta broja goroutine-a,
* povećane potrošnje memorije,
* povećanog scheduling overhead-a,
* blokiranih channel operacija,
* degradacije performansi,
* eventualnog iscrpljivanja resursa.

---

### 14. Kako biste sprečili leak u prethodnom generator primeru?

Generator treba da ima način da dobije signal da consumer više nije zainteresovan.

Na primer:

```go
func generator(ctx context.Context) <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; ; i++ {
            select {
            case <-ctx.Done():
                return

            case ch <- i:
            }
        }
    }()

    return ch
}
```

Caller sada kontroliše lifecycle:

```go
ctx, cancel := context.WithCancel(context.Background())

ch := generator(ctx)

fmt.Println(<-ch)
fmt.Println(<-ch)
fmt.Println(<-ch)

cancel()
```

Kada consumer više nije zainteresovan:

```go
cancel()
```

producer dobija signal i završava.

Ovo je primer **structured lifecycle management-a**:

```text
caller
  │
  ├── kreira context
  │
  ├── pokreće goroutine
  │
  ├── koristi rezultat
  │
  └── cancellation
          │
          ▼
      goroutine
          │
          └── return
```

---

### 15. Da li je dovoljno zatvoriti channel da bi se sve goroutine-e završile?

Ne.

Ovo je jedna od čestih pogrešnih pretpostavki.

Zatvaranje kanala obaveštava **receiver-e** da više neće biti novih vrednosti.

Na primer:

```go
for value := range ch {
    process(value)
}
```

kada se `ch` zatvori, `range` se završava.

Međutim, to ne znači automatski da će svaka goroutine koja je povezana sa tim kanalom završiti.

Na primer, producer može biti blokiran na drugom kanalu:

```go
go func() {
    for {
        select {
        case out <- value:
        case other <- event:
        }
    }
}()
```

Zatvaranje `out` samo po sebi ne predstavlja univerzalni cancellation mehanizam.

Zato treba razlikovati:

* **channel close** — signal da više nema vrednosti,
* **context cancellation** — signal da treba prekinuti rad,
* **goroutine lifecycle** — kompletan model pokretanja, rada i završetka goroutine-a.

---

### 16. Ko treba da zatvara channel?

Opšte pravilo:

> **Sender koji poseduje channel lifecycle treba da ga zatvori.**

Receiver uglavnom ne bi trebalo da zatvara channel koji mu je prosleđen na čitanje.

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

Producer zna kada više nema podataka:

```go
defer close(ch)
```

Consumer samo čita:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Ovo jasno definiše ownership:

```text
Producer
   │
   │ owns close
   ▼
channel
   │
   ▼
Consumer
   │
   └── reads
```

Ovakva pravila ownership-a značajno smanjuju mogućnost:

* `panic: close of closed channel`
* slanja na zatvoren channel
* prerano zatvorenog kanala
* nejasnog lifecycle-a.

---

### Ključne stvari koje kandidat treba da zna

Kod pitanja o lifecycle-u goroutine-a, dobar odgovor ne treba da se zaustavi na:

> "Koristio bih `context`."

Potrebno je razumeti **zašto**.

Treba biti sposoban objasniti:

* ko pokreće goroutine,
* ko je odgovoran za njen završetak,
* šta se dešava kada nema više inputa,
* šta se dešava kada consumer prestane da čita,
* gde goroutine može da se blokira,
* kako se cancellation propagira,
* ko poseduje channel,
* ko ga zatvara,
* kako se sprečava goroutine leak,
* kako se proverava da li lifecycle zaista funkcioniše.

Kod ozbiljnijih sistema ovo prelazi iz pitanja sintakse u pitanje **ownership-a i lifecycle dizajna**.

---

Na Junior → Medior nivou očekuje se da više ne posmatraš gorutinu samo kao način da se neki posao izvršava „u pozadini“.

Potrebno je da razumeš **ko je odgovoran za njen životni ciklus**, kako gorutina završava, šta se dešava sa kanalima koje koristi i kako sprečiti da gorutina ostane aktivna duže nego što je potrebno.

---

### Pitanje 1

**Šta znači da gorutina ima vlasnika (owner-a)?**

### Odgovor

Vlasnik gorutine je komponenta koja je odgovorna za:

* pokretanje gorutine,
* definisanje njenog životnog ciklusa,
* obezbeđivanje načina za njen završetak,
* koordinaciju sa drugim gorutinama,
* i proveru da gorutina zaista može da se završi.

Primer:

```go
func startWorker(jobs <-chan Job) <-chan Result {
	results := make(chan Result)

	go func() {
		defer close(results)

		for job := range jobs {
			results <- process(job)
		}
	}()

	return results
}
```

Ovde funkcija koja pokreće worker praktično definiše njegov lifecycle.

Važno je da worker ne bude samo:

```go
go worker()
```

bez jasnog odgovora na pitanje:

> **Ko i pod kojim uslovima zaustavlja ovu gorutinu?**

Ako takav odgovor ne postoji, postoji rizik od goroutine leak-a.

---

### Pitanje 2

**Zašto je goroutine leak problem čak i kada gorutina ne koristi mnogo CPU-a?**

### Odgovor

Gorutina koja je blokirana i dalje zauzima runtime resurse.

Može da zadrži:

* memoriju,
* reference ka objektima,
* channel,
* context,
* mutex-related state,
* druge objekte koji zbog tih referenci ne mogu da budu oslobođeni.

Jedan leak možda nije problem.

Problem nastaje kada se gorutina kreira mnogo puta:

```go
for {
	go worker()
}
```

Ako worker nikada ne završava, broj gorutina može kontinuirano da raste.

To može dovesti do:

* povećanja memorijske potrošnje,
* većeg GC pritiska,
* degradacije performansi,
* iscrpljivanja resursa,
* i na kraju nestabilnosti servisa.

---

### Pitanje 3

**Kako channel može da dovede do goroutine leak-a?**

### Odgovor

Najčešći slučaj je gorutina koja čeka na channel koji više nikada neće dobiti podatke:

```go
func worker(ch <-chan int) {
	for {
		value := <-ch
		fmt.Println(value)
	}
}
```

Ako nijedna druga gorutina više ne šalje:

```go
ch <- value
```

worker ostaje blokiran na:

```go
value := <-ch
```

Ako niko ne zatvori channel i ne postoji drugi mehanizam za prekid rada, gorutina može ostati aktivna zauvek.

Bezbedniji dizajn često koristi `range`:

```go
func worker(ch <-chan int) {
	for value := range ch {
		fmt.Println(value)
	}
}
```

A producer ili owner zatvara channel kada je slanje završeno:

```go
close(ch)
```

Tada `range` završava i gorutina može normalno da se vrati.

---

### Pitanje 4

**Ko bi trebalo da zatvori channel?**

### Odgovor

Pravilo koje treba usvojiti je:

> **Sender je odgovoran za zatvaranje channel-a kada zna da više neće biti slanja.**

Primer:

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

Producer zna kada je završio slanje i zato zatvara channel.

Consumer ne bi trebalo proizvoljno da zatvara channel koji nije njegov za ownership.

Loš dizajn može dovesti do:

```text
producer ───────► channel ◄────── consumer
                       │
                       └── consumer closes channel
```

Ako producer nakon toga pokuša:

```go
channel <- value
```

dobija se panic:

```text
send on closed channel
```

Zbog toga ownership mora biti jasan.

---

### Pitanje 5

**Kako `context.Context` pomaže u upravljanju životnim ciklusom gorutine?**

### Odgovor

`context.Context` omogućava da gorutina dobije signal da treba da prekine rad.

Tipičan obrazac je:

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

Parent komponenta može otkazati context:

```go
ctx, cancel := context.WithCancel(context.Background())

go worker(ctx)

cancel()
```

Worker dobija signal preko:

```go
<-ctx.Done()
```

i završava se.

Ovo je posebno važno kod dugotrajnih gorutina koje ne mogu jednostavno da se oslanjaju na zatvaranje channel-a.

---

### Pitanje 6

**Da li `cancel()` automatski zaustavlja gorutinu?**

### Odgovor

Ne.

Ovo je veoma važno.

`cancel()` samo signalizira otkazivanje context-a.

Gorutina mora eksplicitno da proverava taj signal.

Na primer:

```go
func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			return

		case job := <-jobs:
			process(job)
		}
	}
}
```

Ako gorutina ignoriše:

```go
ctx.Done()
```

poziv:

```go
cancel()
```

neće sam po sebi prekinuti njeno izvršavanje.

Go nema mehanizam kojim `context` nasilno ubija proizvoljnu gorutinu.

Zato se cancellation zasniva na **cooperative cancellation** principu.

---

### Pitanje 7

**Koja je osnovna razlika između `close(channel)` i `cancel()`?**

### Odgovor

`close(channel)` govori:

> „Preko ovog channel-a više neće biti poslat nijedan element.“

Dok `cancel()` govori:

> „Komponenta treba da prekine ili otkaže svoj rad.“

Zato imaju različitu semantiku.

Channel close:

```go
close(jobs)
```

najčešće predstavlja završetak toka podataka.

Context cancellation:

```go
cancel()
```

predstavlja signal za prekid operacije ili životnog ciklusa.

Mogu se koristiti zajedno:

```go
func worker(ctx context.Context, jobs <-chan Job) {
	for {
		select {
		case <-ctx.Done():
			return

		case job, ok := <-jobs:
			if !ok {
				return
			}

			process(job)
		}
	}
}
```

Worker završava ako:

1. parent otkaže context, ili
2. `jobs` channel bude zatvoren.

Ovakav obrazac je osnova robusnijih concurrency sistema.

---

## Ključne lekcije

Na ovom nivou trebalo bi da možeš da odgovoriš na sledeća pitanja:

* Ko je owner gorutine?
* Ko odlučuje kada gorutina završava?
* Kako gorutina zna da treba da završi?
* Ko zatvara channel?
* Šta se dešava ako consumer čeka na channel koji više niko ne koristi?
* Kako nastaje goroutine leak?
* Koja je uloga `context.Context`?
* Da li `cancel()` nasilno prekida gorutinu?
* Kada koristiti channel close, a kada context cancellation?
* Kako kombinovati više mehanizama za kontrolu lifecycle-a?

Ovo je jedna od ključnih razlika između početnog korišćenja gorutina i **predvidivog concurrency dizajna**.

---

[Prelazak na **Medior — Interview Questions**](../04-medior.md)