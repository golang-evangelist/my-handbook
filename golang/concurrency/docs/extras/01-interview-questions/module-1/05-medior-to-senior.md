# Interview Questions — Medior → Senior

> **Fajl:** `extras/01-interview-questions/module-1/05-medior-to-senior.md`
> 
> **Nivo:** Medior → Senior
> 
> **Oblast:** #1 — Concurrency Fundamentals

---

## 1. Kako bi objasnio razliku između "pokrenuti Goroutine" i "uspostaviti konkurentni sistem"?

Pokretanje Goroutine-a je samo **mehanizam izvršavanja**.

Na primer:

```go
go process()
```

ovim smo samo rekli Go runtime-u da `process()` izvršava konkurentno.

Međutim, konkurentni sistem zahteva mnogo više:

* definisanje vlasništva nad podacima,
* komunikaciju između Goroutines,
* kontrolu životnog ciklusa,
* definisanje načina završavanja,
* sprečavanje blokiranja,
* sprečavanje Goroutine leak-ova,
* pravilno propagiranje grešaka,
* kontrolu količine konkurentnog rada.

Zato senior-level razmišljanje ne počinje pitanjem:

> "Gde mogu da stavim `go`?"

nego:

> "Koji deo sistema treba da izvršavam konkurentno i kako ću kontrolisati njegov životni ciklus?"

---

## 2. Zašto je nekontrolisano pokretanje Goroutines problem?

Na prvi pogled Goroutines deluju veoma jeftino:

```go
for i := 0; i < 100_000; i++ {
    go process(i)
}
```

Međutim, "jeftino" ne znači "besplatno".

Svaka Goroutine zahteva određene runtime resurse, a veliki broj aktivnih Goroutines može dovesti do:

* povećane memorijske potrošnje,
* većeg scheduler overhead-a,
* većeg broja blokiranih Goroutines,
* većeg broja network operacija,
* povećanog pritiska na downstream sisteme,
* težeg debugging-a,
* Goroutine leak-ova.

Posebno opasna situacija nastaje kada broj Goroutines raste proporcionalno broju spoljašnjih zahteva bez ikakve kontrole.

Na primer:

```text
HTTP request
     │
     ▼
go process()
     │
     ▼
database/API
```

Ako sistem primi veliki broj zahteva, možemo dobiti:

```text
10 requests
   ↓
10 Goroutines

10.000 requests
   ↓
10.000 Goroutines

1.000.000 requests
   ↓
1.000.000 Goroutines
```

Senior developer mora razmišljati o **backpressure-u, ograničavanju konkurentnosti i životnom ciklusu Goroutines**, a ne samo o tome da li kod tehnički može da pokrene Goroutine.

---

## 3. Da li je više Goroutines uvek bolje?

Ne.

Broj Goroutines treba da bude posledica arhitekture sistema, a ne cilj sam po sebi.

Na primer, ako imamo:

```go
for _, item := range items {
    go process(item)
}
```

to može biti sasvim prihvatljivo ako je:

* broj `items` mali,
* posao nezavisan,
* downstream sistem može da obradi takav concurrency,
* životni ciklus Goroutines je kontrolisan.

Ali ako `items` može sadržati milione elemenata, isti kod može postati ozbiljan problem.

Bolji dizajn može biti ograničena konkurentnost:

```text
           ┌── Worker 1
Jobs ──────┼── Worker 2
           ├── Worker 3
           └── Worker 4
```

umesto:

```text
Jobs
 │
 ├── Goroutine
 ├── Goroutine
 ├── Goroutine
 ├── Goroutine
 ├── ...
 └── Goroutine
```

Ključno pitanje je:

> Koliko konkurentnog rada sistem realno može da podnese?

---

## 4. Koja je veza između Goroutine-a i Channel-a?

Goroutine predstavlja **jedinicu konkurentnog izvršavanja**, dok channel predstavlja jedan od osnovnih mehanizama komunikacije između Goroutines.

Na primer:

```go
ch := make(chan int)

go func() {
    ch <- 42
}()

value := <-ch
```

Ovde imamo:

```text
Goroutine A
    │
    │ send
    ▼
 Channel
    │
    │ receive
    ▼
Goroutine B
```

Channel omogućava da konkurentne komponente razmenjuju podatke bez potrebe da direktno dele istu memoriju.

Međutim, channel sam po sebi ne rešava kompletan problem konkurentnosti.

I dalje moramo razmišljati o:

* tome ko šalje,
* tome ko prima,
* tome ko zatvara channel,
* mogućem blokiranju,
* životnom ciklusu Goroutines,
* mogućem deadlock-u,
* mogućem Goroutine leak-u.

---

## 5. Kako bi objasnio razliku između unbuffered i buffered channel-a sa aspekta dizajna sistema?

Kod unbuffered channel-a:

```go
ch := make(chan int)
```

send i receive imaju direktnu sinhronizaciju.

```text
Sender ─────► Channel ◄───── Receiver
          mora da se uskladi
```

Sender ne može uspešno da završi send dok odgovarajući receiver ne bude spreman.

Kod buffered channel-a:

```go
ch := make(chan int, 10)
```

channel može privremeno da zadrži određeni broj vrednosti.

```text
Producer
   │
   ▼
┌─────────────┐
│   buffer    │
│  1 2 3 4 5  │
└─────────────┘
   │
   ▼
Consumer
```

To uvodi mogućnost **asinkronije između proizvođača i potrošača**.

Ali buffer nije beskonačna memorija.

Kada se napuni:

```text
Producer
    │
    ▼
[1][2][3][4]
            ← buffer full
    │
    X
    │
 blocked
```

Producer ponovo blokira.

Zato veličina buffer-a predstavlja deo dizajna sistema, a ne samo tehnički detalj.

---

## 6. Šta bi smatrao dobrim razlogom za korišćenje buffered channel-a?

Buffered channel ima smisla kada želimo da dozvolimo određenu količinu nezavisnosti između sender-a i receiver-a.

Na primer:

```go
jobs := make(chan Job, 100)
```

može predstavljati ograničeni queue između producer-a i worker-a.

To može omogućiti:

```text
Producer
   │
   ▼
[ Queue ]
   │
   ├── Worker 1
   ├── Worker 2
   ├── Worker 3
   └── Worker 4
```

Međutim, buffered channel ne treba automatski koristiti samo zato što "program radi brže".

Moramo znati:

* zašto buffer postoji,
* zašto je baš te veličine,
* šta se dešava kada se napuni,
* koliko dugo producer može da radi bez consumer-a,
* šta se događa kada consumer uspori.

---

## 7. Kako bi prepoznao da channel predstavlja deo backpressure mehanizma?

Zamislimo:

```text
Producer
   │
   ▼
Buffered Channel
   │
   ▼
Consumer
```

Ako producer proizvodi podatke brže nego što consumer može da ih obradi:

```text
Producer rate: 1000/s
Consumer rate: 100/s
```

buffer će se vremenom napuniti.

Kada se to dogodi:

```text
Producer
   │
   ▼
[ FULL BUFFER ]
   │
   X
   │
 blocked
```

To je zapravo koristan signal.

Sistem je rekao:

> Consumer ne može da prati producer-a.

Umesto da nekontrolisano generišemo još Goroutines i još memorije, možemo koristiti blokiranje kao oblik **backpressure-a**.

Zato buffering mora biti posmatran kao deo sistema za kontrolu protoka, a ne samo kao optimizacija.

---

## 8. Zašto su channel ownership i odgovornost za `close` važni?

Jedan od najvažnijih dizajnerskih principa kod channel-a jeste jasno definisano vlasništvo.

Tipičan princip glasi:

> Goroutine koja proizvodi vrednosti i poseduje lifecycle producer-a treba da bude odgovorna za zatvaranje channel-a.

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

Consumer tada samo čita:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Consumer ne mora da zna:

* kako je channel kreiran,
* kada producer završava,
* kada treba zatvoriti channel.

Ovo je važan oblik enkapsulacije konkurentnog sistema.

---

## 9. Zašto je `close` signal završetka, a ne signal "više ne želim da koristim channel"?

`close(channel)` treba posmatrati kao informaciju:

> Više neće biti poslatih vrednosti.

Na primer:

```go
close(ch)
```

ne znači:

```text
"Channel više ne postoji."
```

već:

```text
"Producer je završio slanje."
```

Receiver može i dalje da primi preostale buffered vrednosti.

Na primer:

```go
ch := make(chan int, 2)

ch <- 10
ch <- 20

close(ch)
```

receiver i dalje može da dobije:

```text
10
20
```

a nakon toga:

```text
channel closed
```

Zato je `close` deo **protocol-a između producer-a i consumer-a**.

---

## 10. Šta je najvažnije pitanje koje senior developer treba da postavi kada vidi concurrency kod?

Ne:

> "Da li ovo radi?"

nego:

> "Kako se ovaj concurrency sistem ponaša u svim mogućim stanjima?"

Na primer:

### Normalan scenario

```text
Producer
   ↓
Channel
   ↓
Consumer
   ↓
Done
```

### Spor consumer

```text
Producer
   ↓
Channel
   ↓
slow Consumer
```

### Producer završi

```text
Producer
   ↓
close(channel)
   ↓
Consumer
```

### Consumer nestane

```text
Producer
   ↓
Channel
   X
Consumer gone
```

### Error

```text
Producer
   ↓
ERROR
   X
Consumer
```

### Cancellation

```text
Producer
   ↓
Cancellation
   ↓
Goroutine cleanup
```

Senior-level concurrency podrazumeva razmišljanje o **failure modes**, a ne samo o happy path-u.

---

## Ključne poruke ovog dela

* Goroutine je mehanizam izvršavanja, a ne kompletan concurrency dizajn.
* Veliki broj Goroutines nije automatski dobar dizajn.
* Channel je komunikacioni mehanizam, ali zahteva jasno definisan protocol.
* Buffered channel može predstavljati queue i deo backpressure mehanizma.
* Veličina buffer-a je arhitektonska odluka.
* `close` predstavlja signal da više neće biti novih vrednosti.
* Ownership nad channel-om treba da bude eksplicitan.
* Senior developer mora razmišljati o blokiranju, završetku, greškama i cleanup-u.
* Concurrency kod treba analizirati kroz normalne i failure scenarije.

---

## 11. Šta se dešava kada sender pošalje vrednost na channel, a receiver trenutno nije spreman?

Odgovor zavisi od toga da li je channel buffered ili unbuffered.

Kod unbuffered channel-a:

```go
ch := make(chan int)

ch <- 42
```

send će blokirati dok druga Goroutine ne izvrši odgovarajući receive:

```go
value := <-ch
```

Konceptualno:

```text
Sender
   │
   │ ch <- 42
   │
   X  blocked
   │
   │
Receiver
   │
   │ <-ch
   ▼
  42
```

Kada receiver postane spreman, komunikacija može da se nastavi.

Kod buffered channel-a:

```go
ch := make(chan int, 10)

ch <- 42
```

send može odmah da se završi ako buffer ima slobodan prostor.

```text
Sender
   │
   ▼
┌─────────────┐
│  42         │
│             │
│   buffer    │
└─────────────┘
```

Ali kada se buffer napuni, naredni send ponovo blokira.

---

## 12. Šta se dešava kada receiver čita iz channel-a, a trenutno nema dostupne vrednosti?

Kod običnog receive-a:

```go
value := <-ch
```

ako nema dostupne vrednosti, receiver blokira.

Na primer:

```go
ch := make(chan int)

go func() {
    time.Sleep(time.Second)
    ch <- 42
}()

value := <-ch
```

Receiver čeka dok sender ne pošalje vrednost.

Konceptualno:

```text
Receiver
   │
   │ <-ch
   │
   X blocked
   │
   │
   │       42
   │        │
   │        ▼
   └──── Channel
            ▲
            │
         Sender
```

Ovo blokiranje nije samo "čekanje".

Ono je deo sinhronizacionog protokola između konkurentnih komponenti.

---

## 13. Zašto blokiranje Goroutine-a samo po sebi nije problem?

Blokiranje je normalna karakteristika konkurentnog Go programa.

Problem nastaje kada blokiranje:

* nije očekivano,
* nema mogućnost završetka,
* nastavlja da traje neograničeno,
* sprečava druge Goroutines da završe,
* dovodi do deadlock-a,
* dovodi do Goroutine leak-a.

Na primer, ovo je validan oblik sinhronizacije:

```go
value := <-ch
```

ako znamo da će druga komponenta eventualno poslati vrednost.

Ali ako sender nikada neće izvršiti:

```go
ch <- value
```

receiver može ostati blokiran zauvek.

Zato pitanje nije:

> "Da li kod blokira?"

nego:

> "Da li je blokiranje deo namerno dizajniranog protokola i da li postoji garantovan način da se završi?"

---

## 14. Kako bi objasnio razliku između očekivanog blokiranja i Goroutine leak-a?

### Očekivano blokiranje

Goroutine čeka zato što čeka legitimni događaj:

```go
value := <-ch
```

i postoji jasno definisan put:

```text
Producer
   ↓
send
   ↓
Channel
   ↓
receive
   ↓
Consumer continues
```

### Goroutine leak

Goroutine čeka na događaj koji se nikada neće dogoditi.

Na primer:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        process(value)
    }
}
```

Ako niko više nikada ne šalje vrednosti i nema nikakvog mehanizma za završavanje worker-a, Goroutine može ostati zauvek blokirana.

Konceptualno:

```text
Worker
  │
  ▼
<-ch
  │
  X
  │
  │ forever
  │
  └─────────────► Goroutine leak
```

Zato senior developer mora analizirati **životni ciklus svake dugotrajne Goroutine**.

---

## 15. Kako bi sprečio Goroutine leak u consumer Goroutine-i?

Jedan od osnovnih pristupa je korišćenje zatvaranja channel-a.

Na primer:

```go
func worker(ch <-chan int) {
    for value := range ch {
        process(value)
    }
}
```

Producer:

```go
func producer(ch chan<- int) {
    defer close(ch)

    for i := 0; i < 10; i++ {
        ch <- i
    }
}
```

Kada producer završi:

```text
Producer
   │
   │ sends values
   ▼
Channel
   │
   │ close
   ▼
Worker
   │
   ▼
range terminates
   │
   ▼
return
```

Ovo je jednostavan i veoma čest lifecycle pattern.

Međutim, `close` nije dovoljan za svaki scenario.

Ako worker može biti blokiran na nekoj drugoj operaciji, potrebni su dodatni mehanizmi za cancellation ili timeout.

---

## 16. Zašto `range` preko channel-a zahteva zatvaranje channel-a?

Kod:

```go
for value := range ch {
    process(value)
}
```

petlja se završava kada channel bude zatvoren i kada budu pročitane sve preostale vrednosti.

Ako channel nikada nije zatvoren:

```text
for range ch
      │
      ▼
receive
      │
      ▼
receive
      │
      ▼
receive
      │
      ▼
...
```

i više nema novih vrednosti, Goroutine može ostati blokirana.

Zato producer koji koristi `range` consumer model mora imati jasno definisan trenutak završetka:

```go
defer close(ch)
```

ili drugi odgovarajući lifecycle mehanizam.

---

## 17. Da li receiver treba da zatvara channel kada više ne želi da prima podatke?

U pravilu — **ne**.

Ako imamo:

```text
Producer ─────► Channel ─────► Consumer
```

producer je odgovoran za činjenicu:

> "Više nema podataka."

Consumer može odlučiti:

> "Mene više ne zanimaju podaci."

To su dve različite informacije.

Zatvaranje channel-a od strane consumer-a može biti opasno jer producer možda i dalje pokušava da pošalje vrednost:

```go
ch <- value
```

što može dovesti do:

```text
send on closed channel
```

Zato je veoma važno razlikovati:

* **producer završava proizvodnju**,
* **consumer završava svoju obradu**.

Ako consumer želi da signalizira producer-u da više nije zainteresovan, bolji dizajn je eksplicitan cancellation signal ili `context.Context`, kada je odgovarajući.

---

## 18. Zašto je channel ownership važan kod većih sistema?

U malom programu možemo relativno lako videti:

```go
ch := make(chan int)
```

i pronaći:

* ko šalje,
* ko prima,
* ko zatvara.

U većem sistemu channel može prolaziti kroz više funkcija:

```text
API
 │
 ▼
Service
 │
 ▼
Worker Manager
 │
 ▼
Worker
 │
 ▼
Repository
```

Ako nije jasno ko poseduje channel, mogu nastati problemi:

```text
Who creates it?
Who sends?
Who receives?
Who closes?
Who decides when it is finished?
```

Zato dobar API treba da ograniči odgovornosti.

Na primer:

```go
func producer() <-chan Result
```

umesto:

```go
func producer() chan Result
```

Prva verzija jasno kaže:

> Pozivalac može da prima vrednosti, ali ne treba da šalje niti da upravlja producer-ovim slanjem.

To je oblik **compile-time dokumentovanja ownership-a**.

---

## 19. Kako channel direction doprinosi bezbednijem concurrency dizajnu?

Go omogućava:

```go
chan T
chan<- T
<-chan T
```

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

Ovim jasno izražavamo:

```text
producer
    │
    │ chan<- int
    ▼
 Channel
    │
    │ <-chan int
    ▼
consumer
```

Prednost nije samo dokumentaciona.

Compiler može sprečiti određene klase grešaka.

Na primer, producer ne može da izvrši:

```go
<-out
```

ako je `out` tipa:

```go
chan<- int
```

A consumer ne može da izvrši:

```go
in <- value
```

ako je `in` tipa:

```go
<-chan int
```

Dakle, channel direction pretvara deo concurrency contract-a u **statičku proveru tipova**.

---

## 20. Šta bi senior developer proverio kada dobije postojeći channel-based sistem na code review-u?

Ne bi proveravao samo da li se channel pravilno koristi u happy path-u.

Analiza bi obuhvatila najmanje:

### Ownership

* Ko kreira channel?
* Ko šalje?
* Ko prima?
* Ko zatvara?

### Lifecycle

* Kada producer počinje?
* Kada završava?
* Kada consumer završava?
* Šta se događa ako consumer nestane?

### Blocking

* Gde Goroutines mogu da blokiraju?
* Da li je blokiranje očekivano?
* Može li trajati neograničeno?

### Buffering

* Da li je channel buffered?
* Zašto?
* Zašto baš ta veličina?
* Šta se događa kada se buffer napuni?

### Failure handling

* Šta se događa ako producer dobije grešku?
* Kako consumer saznaje za grešku?
* Da li se ostale Goroutines završavaju?

### Cleanup

* Da li sve pokrenute Goroutines imaju put do završetka?
* Postoji li mogućnost Goroutine leak-a?

### API design

* Da li se koriste `<-chan T` i `chan<- T` gde imaju smisla?
* Da li je ownership dovoljno jasan?

### Scalability

* Šta se događa kada broj zahteva poraste 10×?
* Da li concurrency raste bez ograničenja?
* Gde se primenjuje backpressure?

Ovo je razlika između pitanja:

> "Da li program koristi Goroutines i channel-e?"

i ozbiljne analize:

> "Da li je concurrency model ovog sistema održiv pod realnim opterećenjem i failure scenarijima?"

---

## Ključne poruke ovog dela

* Channel send i receive mogu blokirati i to je normalan deo Go concurrency modela.
* Blokiranje samo po sebi nije problem; problem je **neograničeno ili neočekivano blokiranje**.
* Goroutine leak nastaje kada Goroutine nema realan put do završetka.
* `range` preko channel-a zahteva jasan signal završetka.
* Producer je tipično odgovoran za zatvaranje channel-a.
* Consumer koji više nije zainteresovan ne treba proizvoljno da zatvara producer-ov channel.
* Channel ownership treba da bude eksplicitan.
* Channel directions (`chan<-`, `<-chan`) omogućavaju compile-time proveru dela concurrency contract-a.
* Senior code review mora obuhvatiti ownership, lifecycle, blocking, buffering, failure handling, cleanup i scalability.

---

## 21. Kako bi dizajnirao pipeline zasnovan na channel-ima?

Pipeline predstavlja lanac konkurentnih stage-ova gde svaki stage:

1. prima podatke,
2. obrađuje ih,
3. prosleđuje rezultat sledećem stage-u.

Na primer:

```text
Input
  │
  ▼
Stage 1
  │
  ▼
Stage 2
  │
  ▼
Stage 3
  │
  ▼
Output
```

U Go-u svaki stage može biti predstavljen Goroutine-om i channel-om:

```go
func square(in <-chan int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for value := range in {
            out <- value * value
        }
    }()

    return out
}
```

Pipeline se zatim može kompoziciono povezati:

```go
stage1 := generate()
stage2 := square(stage1)
stage3 := square(stage2)

for value := range stage3 {
    fmt.Println(value)
}
```

Važna karakteristika ovakvog dizajna jeste da su stage-ovi relativno nezavisni.

Svaki stage ima jasno definisan:

* input,
* output,
* lifecycle,
* ownership.

---

## 22. Koji je najveći problem naivnog pipeline dizajna?

Najveći problem je **cancellation**.

Pretpostavimo:

```text
Generator
    ↓
Stage A
    ↓
Stage B
    ↓
Stage C
```

Ako `Stage C` prestane da čita zato što je pronašao rezultat koji mu je potreban, `Stage B` može ostati blokiran na:

```go
out <- value
```

A `Stage A` zatim može ostati blokiran na sopstvenom output channel-u.

Dobijamo:

```text
Stage A
   │
   X blocked
   │
Stage B
   │
   X blocked
   │
Stage C
   │
   └── stopped
```

Jedna Goroutine je prestala da učestvuje u pipeline-u, ali prethodni stage-ovi nisu dobili informaciju o tome.

Ovo je tipičan uzrok Goroutine leak-a.

Zato pipeline mora imati **cancellation propagation**.

---

## 23. Kako bi dodao cancellation u pipeline?

Jedan pristup je eksplicitni `done` channel:

```go
func square(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for {
            select {
            case <-done:
                return

            case value, ok := <-in:
                if !ok {
                    return
                }

                result := value * value

                select {
                case out <- result:
                case <-done:
                    return
                }
            }
        }
    }()

    return out
}
```

Sada svaki stage može reagovati na cancellation:

```text
             done
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
    Stage A  Stage B  Stage C
```

Kada se cancellation aktivira, svaki stage dobija mogućnost da prekine rad i oslobodi svoje resurse.

U modernom Go kodu, kada postoji request-scoped lifecycle, često je prirodnije koristiti `context.Context`.

---

## 24. Kada bi koristio channel, a kada `sync.Mutex`?

Ne postoji pravilo:

> "Channel je bolji od mutex-a."

To su različiti alati.

### Channel

Channel je naročito pogodan kada želimo:

* komunikaciju između Goroutines,
* ownership transfer,
* producer/consumer model,
* pipeline,
* event stream,
* cancellation signal.

Na primer:

```text
Producer → Channel → Consumer
```

### Mutex

Mutex je pogodan kada više Goroutines treba da pristupa **istom mutable state-u**.

Na primer:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    c.n++
    c.mu.Unlock()
}
```

Ovde channel nije nužno prirodniji.

Imamo jednostavan shared state:

```text
Goroutine A ─┐
Goroutine B ─┼──► Counter
Goroutine C ─┘
```

Mutex direktno štiti invariant tog state-a.

Senior developer bira alat na osnovu **modela problema**, a ne na osnovu opšte preferencije.

---

## 25. Šta znači "Do not communicate by sharing memory; share memory by communicating"?

Ova poznata Go ideja opisuje preferenciju ka modelu u kojem se ownership podataka prenosi kroz komunikaciju.

Umesto:

```text
Goroutine A
     │
     ▼
shared mutable state
     ▲
     │
Goroutine B
```

možemo imati:

```text
Goroutine A
     │
     │ data
     ▼
  Channel
     │
     ▼
Goroutine B
```

Ideja je da jedna Goroutine može biti vlasnik određenog state-a, dok druge Goroutines komuniciraju sa njom putem channel-a.

Na primer:

```go
type Request struct {
    ID    int
    Value int
}

requests := make(chan Request)
```

Jedna Goroutine može posedovati state:

```go
func worker(requests <-chan Request) {
    state := make(map[int]int)

    for request := range requests {
        state[request.ID] = request.Value
    }
}
```

Druge Goroutines ne moraju direktno pristupati `state` mapi.

To može smanjiti potrebu za lock-ovima.

Ali ovo **nije univerzalno pravilo**.

Channel-based ownership može biti kompleksniji od mutex-a ako je problem jednostavan.

---

## 26. Kada je channel-based state machine bolji izbor od shared state-a?

Channel-based state machine može biti koristan kada postoji jasno definisan **owner state-a**.

Na primer:

```text
             ┌─────────────┐
Request ────►│             │
Request ────►│ State Owner │
Request ────►│             │
             └──────┬──────┘
                    │
                    ▼
                 state
```

Samo jedna Goroutine menja state.

Ostale Goroutines šalju komande:

```go
type Command struct {
    Key   string
    Value int
}
```

Owner obrađuje komande sekvencijalno:

```go
for cmd := range commands {
    state[cmd.Key] = cmd.Value
}
```

Prednost je što nema potrebe za zaključavanjem `state` mape između više writer-a.

Ali cena može biti:

* dodatne Goroutines,
* dodatni channel-i,
* složeniji lifecycle,
* teže request/response semantike,
* potencijalni bottleneck jednog owner-a.

Zato treba analizirati workload.

---

## 27. Šta je backpressure i zašto je važan?

**Backpressure** je mehanizam kojim downstream komponenta ograničava brzinu kojom upstream komponenta može da proizvodi podatke.

Bez backpressure-a:

```text
Producer
  │
  │ 100k req/s
  ▼
Consumer
  │
  │ 10k req/s
  ▼
Processing
```

Producer može generisati više posla nego što sistem može da obradi.

Rezultat može biti:

* rast memorije,
* rast queue-a,
* povećanje latency-ja,
* timeout-i,
* OOM,
* cascade failure.

Buffered channel može delimično predstavljati backpressure:

```go
jobs := make(chan Job, 100)
```

Kada se buffer popuni, producer blokira:

```text
Producer
   │
   ▼
┌───────────────┐
│ 100 jobs      │
│ BUFFER FULL   │
└───────────────┘
       │
       X
    blocked
```

Na taj način sistem prirodno usporava producer-a.

---

## 28. Zašto veličina buffered channel-a ne treba da bude proizvoljna?

Kod:

```go
jobs := make(chan Job, 1000)
```

broj `1000` nije automatski dobar izbor.

Buffer predstavlja deo concurrency dizajna.

Treba analizirati:

* očekivani throughput,
* consumer processing time,
* burst behavior,
* memory footprint,
* latency,
* workload variability,
* failure scenarios.

Prevelik buffer može samo odložiti problem.

Na primer:

```text
Producer: 100k jobs
Consumer: 10k jobs/s
Buffer:   1000
```

Ako producer konstantno nadmašuje consumer, buffer će se samo ponovo napuniti.

Veliki buffer može sakriti overload dok sistem ne dođe do još većeg problema.

Zato buffered channel nije zamena za pravi capacity planning.

---

## 29. Kako bi razlikovao throughput, latency i concurrency?

### Throughput

Broj jedinica posla obrađenih u određenom vremenu.

Na primer:

```text
10,000 requests / second
```

### Latency

Vreme potrebno da se jedan zahtev obradi.

Na primer:

```text
P99 latency = 120 ms
```

### Concurrency

Broj operacija koje su u određenom trenutku aktivne/in-flight.

Na primer:

```text
500 requests in-flight
```

Ove metrike nisu međusobno iste.

Možemo povećati concurrency, a da throughput ne raste.

Možemo čak povećati concurrency i pogoršati latency zbog:

* contention-a,
* scheduler overhead-a,
* GC pritiska,
* lock contention-a,
* I/O saturation-a.

Senior concurrency dizajn zato mora razmatrati sistemske metrike, a ne samo broj Goroutines.

---

## 30. Kako bi procenio da li concurrency zaista poboljšava sistem?

Ne treba zaključivati na osnovu:

```go
go doWork()
```

ili broja pokrenutih Goroutines.

Potrebno je meriti sistem.

Tipična pitanja su:

```text
Da li je throughput veći?
Da li je latency manji?
Da li je CPU utilization prihvatljiv?
Da li je memory usage prihvatljiv?
Da li postoji contention?
Da li raste broj timeout-a?
Da li sistem ostaje stabilan pod overload-om?
```

Na primer, ako sekvencijalni sistem ima:

```text
Throughput:  1,000 req/s
P99:         20 ms
CPU:         40%
```

a konkurentna verzija:

```text
Throughput:  1,100 req/s
P99:         200 ms
CPU:         95%
```

nije očigledno da je druga verzija bolja.

Concurrency je sredstvo za postizanje sistemskog cilja, a ne cilj sam po sebi.

---

## Ključne poruke ovog dela

* Pipeline je kompozicija konkurentnih stage-ova povezanih channel-ima.
* Najveći problem pipeline-a je često lifecycle i cancellation.
* Ako downstream prestane da čita, upstream može ostati blokiran.
* Cancellation mora propagirati kroz ceo pipeline.
* Channel i mutex nisu međusobno zamenljivi alati.
* Channel je naročito koristan za komunikaciju i ownership transfer.
* Mutex je prirodan za zaštitu shared mutable state-a.
* Channel-based state ownership može eliminisati deo lock contention-a, ali uvodi druge troškove.
* Backpressure sprečava upstream da neograničeno proizvodi posao.
* Buffer veličina treba da bude rezultat workload analize, a ne proizvoljan broj.
* Throughput, latency i concurrency su različite metrike.
* Više Goroutines ne znači automatski bolji sistem.
* Concurrency optimizacije treba potvrditi merenjem.

---

## 31. Kako bi dizajnirao worker pool u Go-u?

Worker pool je concurrency pattern kojim ograničavamo broj istovremeno aktivnih worker Goroutines.

Osnovna struktura:

```text
                 ┌──────────────┐
                 │   Job Queue  │
                 └──────┬───────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Worker 1      Worker 2      Worker 3
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                     Results
```

Tipična implementacija:

```go
func worker(jobs <-chan Job, results chan<- Result) {
    for job := range jobs {
        results <- process(job)
    }
}
```

Broj worker-a je ograničen:

```go
for i := 0; i < workerCount; i++ {
    go worker(jobs, results)
}
```

Producer šalje poslove:

```go
for _, job := range jobsToProcess {
    jobs <- job
}
close(jobs)
```

Worker-i završavaju kada se `jobs` channel zatvori.

Važno je da worker pool ne posmatramo samo kao "N Goroutines".

On predstavlja **kontrolu concurrency-ja i sistemskog kapaciteta**.

---

## 32. Zašto worker pool može biti bolji od pokretanja nove Goroutine za svaki posao?

Go Goroutines su jeftine, ali "jeftino" ne znači "besplatno".

Ako imamo:

```go
for _, job := range jobs {
    go process(job)
}
```

i broj poslova može biti veoma velik, concurrency može nekontrolisano da raste.

Na primer:

```text
1,000 jobs   → 1,000 Goroutines
100,000 jobs → 100,000 Goroutines
1,000,000 jobs → 1,000,000 Goroutines
```

To može dovesti do:

* povećanog scheduler overhead-a,
* povećane memorijske potrošnje,
* prevelikog broja istovremenih I/O operacija,
* overload-a downstream servisa,
* većeg GC pritiska,
* povećanja latency-ja.

Worker pool uvodi eksplicitni limit:

```text
100,000 jobs
      │
      ▼
┌─────────────┐
│ jobs channel│
└──────┬──────┘
       │
       ▼
  20 workers
```

Samo 20 poslova se obrađuje konkurentno.

Ostali čekaju.

To predstavlja oblik **concurrency control-a**.

---

## 33. Da li broj worker-a treba da bude jednak broju CPU jezgara?

Ne nužno.

Za CPU-bound workload često ima smisla početi od vrednosti koja je povezana sa brojem dostupnih CPU resursa.

Na primer:

```go
runtime.GOMAXPROCS(0)
```

može dati trenutno podešavanje maksimalnog broja P konteksta koji mogu izvršavati Go kod paralelno.

Ali za I/O-bound workload:

```text
Worker
  │
  ▼
Network I/O
  │
  └── waiting
```

Goroutine veliki deo vremena ne koristi CPU.

Zato može imati smisla imati više worker-a od broja CPU jezgara.

Na primer:

```text
CPU-bound:
8 CPU → možda ~8 aktivnih workers

I/O-bound:
8 CPU → možda desetine/stotine workers
```

Ali ni jedna od ovih vrednosti nije univerzalno pravilo.

Optimalan broj zavisi od:

* workload-a,
* CPU utilization-a,
* I/O latency-ja,
* downstream capacity-ja,
* memory usage-a,
* contention-a,
* target latency-ja.

Najbolji pristup je merenje i load testing.

---

## 34. Kako bi sprečio da worker pool "proguta" neograničenu količinu poslova?

Potrebno je kontrolisati queue.

Ako imamo:

```go
jobs := make(chan Job, 1_000_000)
```

to ne znači da smo rešili overload.

Samo smo dozvolili da ogromna količina posla bude akumulirana u memoriji.

Bolji dizajn može koristiti ograničen buffer:

```go
jobs := make(chan Job, 100)
```

Kada se buffer napuni:

```text
Producer
   │
   ▼
┌────────────┐
│ 100 jobs   │
│ FULL       │
└─────┬──────┘
      │
      X
   blocked
```

Producer tada mora:

* da čeka,
* odbije posao,
* primeni timeout,
* vrati grešku,
* primeni retry policy,
* ili signalizira overload.

To je mnogo zdraviji model od neograničenog rasta queue-a.

---

## 35. Kako bi implementirao bounded concurrency bez klasičnog worker pool-a?

Možemo koristiti buffered channel kao semaphore:

```go
sem := make(chan struct{}, 10)
```

Pre pokretanja rada:

```go
sem <- struct{}{}

go func() {
    defer func() {
        <-sem
    }()

    process()
}()
```

Na ovaj način najviše 10 operacija može biti aktivno u datom trenutku.

Konceptualno:

```text
          Semaphore
       ┌──────────────┐
       │ 10 permits   │
       └──────┬───────┘
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
      Job    Job    Job
```

Ovo je korisno kada želimo:

* ograničiti broj concurrent requests,
* ograničiti database connections,
* ograničiti API calls,
* kontrolisati expensive operations.

Ali treba paziti na lifecycle i čekanje.

Ako je Goroutine već pokrenuta pre nego što dobije permit:

```go
go func() {
    sem <- struct{}{}
    ...
}()
```

možemo i dalje stvoriti veliki broj blokiranih Goroutines.

To nije isto što i bounded number of Goroutines.

---

## 36. Koja je razlika između ograničavanja broja Goroutines i ograničavanja broja aktivnih operacija?

Ovo je važna razlika.

### Bounded Goroutines

Imamo ograničen broj worker-a:

```text
20 workers
```

i oni uzimaju posao iz queue-a.

### Bounded operations

Možemo imati veliki broj request handler-a, ali samo određeni broj njih može ući u kritičnu operaciju:

```text
1000 Goroutines
      │
      ▼
┌─────────────┐
│  semaphore  │
│  20 permits │
└──────┬──────┘
       │
       ▼
  expensive work
```

Ostatak čeka.

Prvi model ograničava concurrency na nivou execution workers-a.

Drugi model ograničava concurrency određene operacije.

Senior dizajn zahteva da znamo **šta tačno ograničavamo**.

---

## 37. Kako bi rešio problem kada jedan worker dugo blokira?

Pretpostavimo worker pool:

```text
Worker 1 → 10ms
Worker 2 → 20ms
Worker 3 → 5s
Worker 4 → 15ms
```

Ako je broj worker-a mali, jedan spor posao može zauzeti značajan deo kapaciteta.

Ako svi worker-i dobiju dugotrajne poslove:

```text
Worker 1 → blocked
Worker 2 → blocked
Worker 3 → blocked
Worker 4 → blocked

Queue → growing
```

Potrebno je analizirati prirodu workload-a.

Moguća rešenja uključuju:

* timeout,
* context cancellation,
* odvajanje različitih tipova poslova,
* posebne worker pool-ove,
* priority queue,
* circuit breaker,
* bounded retries,
* workload-specific concurrency limits.

Na primer:

```go
ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()

result := process(ctx, job)
```

Sada posao ima definisan lifecycle.

---

## 38. Zašto retry može biti opasan u concurrency sistemima?

Pretpostavimo da downstream servis ima kapacitet:

```text
100 requests/s
```

Naš sistem šalje:

```text
100 requests/s
```

Ako downstream počne da vraća greške, naivni retry može proizvesti:

```text
100 original requests
+
100 retries
+
100 retry retries
+
...
```

Dobijamo **retry storm**.

```text
              ┌── retry ──┐
              │            ▼
Client ─────► Service ───► Dependency
  ▲             │
  │             │ error
  └─────────────┘
```

Concurrency sistemi moraju zato kombinovati:

* retry limit,
* exponential backoff,
* jitter,
* timeout,
* cancellation,
* concurrency limit,
* eventualno circuit breaker.

Retry bez ograničenja može povećati load upravo onda kada je dependency već preopterećen.

---

## 39. Kako bi dizajnirao cancellation za dugotrajnu operaciju?

Preferirani pristup je da cancellation signal bude deo API-ja.

Na primer:

```go
func process(ctx context.Context, job Job) error {
    select {
    case <-ctx.Done():
        return ctx.Err()

    default:
    }

    return doWork(ctx, job)
}
```

Još važnije je da downstream operacije takođe podržavaju context:

```go
func queryDatabase(ctx context.Context) error {
    // database operation using ctx
    return nil
}
```

Lifecycle postaje:

```text
Request
   │
   ▼
Context
   │
   ├──── Worker
   │       │
   │       └── DB
   │
   └──── HTTP call
```

Ako request bude otkazan:

```text
Context cancellation
        │
        ├── Worker stops
        ├── DB operation stops
        └── HTTP request stops
```

Time se sprečava da posao nastavi da troši resurse nakon što njegov rezultat više nije potreban.

---

## 40. Šta bi smatrao znakom da worker-pool dizajn nije dobar?

Nekoliko tipičnih simptoma:

### 1. Queue stalno raste

```text
Incoming: 10,000/s
Processing: 5,000/s
```

Sistem proizvodi više posla nego što može da obradi.

### 2. Worker count raste bez kontrole

Ako se worker-i dinamički dodaju bez jasne granice:

```text
100 → 1,000 → 10,000 workers
```

to može samo premestiti problem.

### 3. Visok latency

Veći queue znači da request možda dugo čeka pre nego što obrada uopšte počne.

### 4. Downstream overload

Previše concurrent workers može preopteretiti:

* bazu,
* HTTP dependency,
* filesystem,
* message broker.

### 5. Goroutine leaks

Worker-i koji ne mogu da završe ostaju u memoriji.

### 6. Teško gašenje

Ako nije jasno kako se:

```text
producer
worker
queue
consumer
```

zaustavljaju, shutdown postaje rizičan.

### 7. Nedostatak observability-ja

Bez metrika ne znamo:

* queue depth,
* worker utilization,
* processing latency,
* error rate,
* cancellation rate.

Dobar worker pool nije samo concurrency primitive.

On je **sistemski capacity-control mehanizam**.

---

## Ključne poruke ovog dela

* Worker pool ograničava concurrency i kontroliše sistemski kapacitet.
* Broj worker-a ne treba proizvoljno postavljati.
* CPU-bound i I/O-bound workload-i imaju različite karakteristike.
* Veliki buffered queue nije automatski dobro rešenje.
* Bounded concurrency treba jasno definisati: da li ograničavamo Goroutines ili konkretne operacije?
* Semaphore pattern može kontrolisati broj aktivnih operacija.
* Dugotrajne operacije treba da imaju jasan timeout/cancellation lifecycle.
* Retry bez backoff-a i ograničenja može izazvati retry storm.
* Worker pool treba posmatrati kroz throughput, latency, queue depth i downstream capacity.
* Dobar concurrency dizajn uključuje i shutdown i observability, a ne samo pokretanje Goroutines.

---

Na Medior → Senior nivou nije dovoljno znati kako da se pokrene goroutine. Potrebno je razumeti da **svaka goroutine predstavlja resurs čiji životni ciklus mora biti eksplicitno dizajniran**.

Osnovno pitanje više nije:

> „Kako da pokrenem goroutine?“

nego:

> „Ko je odgovoran za njeno pokretanje, kada ona treba da završi, šta se dešava ako njen posao ne može da se završi i kako garantujem da neće ostati zauvek aktivna?“

Dobar dizajn goroutine-a treba da odgovori na najmanje sledeća pitanja:

1. Ko je njen owner?
2. Koji događaj predstavlja normalan završetak?
3. Koji događaj predstavlja otkazivanje?
4. Kako goroutine saznaje da treba da završi?
5. Kako caller zna da je goroutine završila?
6. Šta se dešava ako downstream consumer prestane da prima podatke?
7. Da li goroutine može da blokira zauvek?
8. Ko čisti resurse koje goroutine koristi?

---

## 5.1. Goroutine treba da ima jasan owner

Jedan od najvažnijih principa je **goroutine ownership**.

Ako neka funkcija kreira goroutine:

```go
func StartWorker() {
    go worker()
}
```

postavlja se pitanje:

> Ko je odgovoran za završetak `worker` goroutine-a?

Ako odgovor nije jasan, dizajn je potencijalno problematičan.

Bolji API obično eksplicitno definiše životni ciklus:

```go
func StartWorker(ctx context.Context) error {
    go worker(ctx)

    return nil
}
```

Sada `ctx` predstavlja deo ugovora između owner-a i worker-a.

Worker može da prati cancellation:

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

Owner zatim kontroliše životni ciklus kroz context.

---

## 5.2. Goroutine treba da ima definisan izlaz

Goroutine koja nema jasno definisan način završetka predstavlja potencijalni leak.

Na primer:

```go
func worker(ch <-chan int) {
    go func() {
        for value := range ch {
            process(value)
        }
    }()
}
```

Ova goroutine će završiti samo ako se `ch` zatvori.

Ako se to nikada ne dogodi:

```text
producer
   │
   ▼
channel
   │
   ▼
worker goroutine
   │
   └── čeka zauvek
```

To može biti sasvim ispravan dizajn ako je goroutine namerno dugovečna.

Ali ako je worker trebalo da bude privremen, onda imamo potencijalni goroutine leak.

---

## 5.3. Cancellation treba da bude deo dizajna

Za goroutine koje obavljaju posao koji može biti prekinut, često je potrebno omogućiti cancellation.

Tipičan obrazac je:

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

Worker ima dva normalna načina završetka:

```text
             ┌──────────────┐
             │    worker    │
             └──────┬───────┘
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      jobs closed         ctx canceled
          │                   │
          └─────────┬─────────┘
                    ▼
                  return
```

Ovakav dizajn je mnogo robusniji od goroutine koja zavisi samo od jednog izvora podataka.

---

## 5.4. Cancellation nije dovoljan ako goroutine ignoriše signal

Sledeći kod izgleda kao da podržava cancellation:

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return

        default:
            expensiveOperation()
        }
    }
}
```

Ali ako `expensiveOperation()` traje veoma dugo, worker neće proveriti `ctx.Done()` dok se operacija ne završi.

Dakle:

```text
ctx.Cancel()
    │
    ▼
worker
    │
    ├── vidi cancellation?
    │       │
    │       └── NE
    │
    ▼
expensiveOperation()
    │
    ▼
tek nakon završetka proverava cancellation
```

Zbog toga je važno analizirati **granularnost cancellation-a**.

Ako posao uključuje blokirajuću operaciju, idealno je da i ta operacija podržava context:

```go
result, err := client.Do(ctx, request)
```

ili da se blokiranje ograniči timeout-om.

---

## 5.5. Ko čeka da goroutine završi?

Pokretanje goroutine-a:

```go
go worker()
```

ne daje caller-u nikakvu informaciju o njenom završetku.

Ako je potrebno sačekati završetak, mora postojati synchronization mechanism.

Jedan od najčešćih obrazaca koristi `sync.WaitGroup`:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
    defer wg.Done()

    worker()
}()

wg.Wait()
```

Sada postoji jasna veza:

```text
owner
  │
  ├── starts worker
  │
  └── waits
        │
        ▼
     worker
        │
        ▼
    wg.Done()
        │
        ▼
      owner
```

Međutim, `WaitGroup` sam po sebi ne predstavlja cancellation mechanism.

On odgovara na pitanje:

> „Da li su goroutine završile?“

Ne odgovara na:

> „Kako da im kažem da završe?“

Zato se u ozbiljnijim sistemima često kombinuju lifecycle kontrola i synchronization.

---

## 5.6. Goroutine leak često nastaje zbog blokiranog send-a

Jedan od klasičnih problema:

```go
func producer() <-chan int {
    ch := make(chan int)

    go func() {
        for i := 0; i < 100; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Ako consumer pročita samo nekoliko vrednosti:

```go
ch := producer()

fmt.Println(<-ch)
fmt.Println(<-ch)
```

producer može ostati blokiran na:

```go
ch <- i
```

Ako niko više ne prima sa kanala:

```text
producer
   │
   ▼
ch <- value
   │
   X
blocked forever
```

To je tipičan goroutine leak.

---

## 5.7. Cancellation treba da obuhvati i send

Robusniji dizajn:

```go
func producer(ctx context.Context) <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 0; i < 100; i++ {
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

Sada producer može da napusti blokirani send:

```text
                ┌── ch <- i
producer ───────┤
                └── ctx.Done()
```

Ako consumer nestane, owner može otkazati context.

---

## 5.8. Consumer takođe može biti izvor problema

Problemi nisu ograničeni na producer-e.

Na primer:

```go
func consumer(ch <-chan int) {
    for value := range ch {
        process(value)
    }
}
```

Ako producer nikada ne zatvori kanal:

```text
producer
   │
   ▼
channel
   │
   ▼
consumer
   │
   └── range čeka zauvek
```

Zato je ownership nad zatvaranjem kanala važan deo dizajna.

Generalno pravilo:

> **Sender je najčešće odgovoran za zatvaranje kanala kada zna da više neće slati vrednosti.**

Consumer ne treba proizvoljno da zatvara kanal koji nije njegov za ownership.

---

## 5.9. Senior kandidat treba da razmišlja o failure scenarijima

Na Senior nivou pitanje o goroutine lifecycle-u nije samo:

> „Da li ovaj kod radi?“

nego:

> „Šta se dešava kada sistem ne radi idealno?“

Treba razmotriti:

* consumer prestane da čita;
* producer prestane da šalje;
* kanal nikada nije zatvoren;
* goroutine blokira na send-u;
* goroutine blokira na receive-u;
* downstream servis postane nedostupan;
* request bude otkazan;
* timeout istekne;
* worker završi sa greškom;
* jedna goroutine završi, a druge ostanu aktivne;
* owner izgubi interesovanje za rezultat.

Ovo je razlika između **poznavanja goroutine syntax-e** i **dizajniranja pouzdanog concurrent sistema**.

---

## 5.10. Šta bih očekivao od dobrog odgovora na intervjuu?

Dobar Medior → Senior odgovor trebalo bi da pomene najmanje:

* goroutine ownership;
* definisan lifecycle;
* cancellation;
* `context.Context`;
* `select`;
* channel ownership;
* zatvaranje kanala;
* mogućnost blokiranja na send/receive;
* goroutine leak;
* `sync.WaitGroup` za čekanje završetka;
* timeout/deadline kada je relevantno;
* ponašanje sistema u failure scenarijima.

Još bolji odgovor bi napravio razliku između:

```text
Cancellation
    ↓
signal goroutine-i da završi

Synchronization
    ↓
čekanje da goroutine završi

Resource ownership
    ↓
određivanje ko upravlja lifecycle-om

Backpressure
    ↓
kontrola odnosa producer ↔ consumer
```

To pokazuje da kandidat ne posmatra concurrency kao skup izolovanih API-ja, već kao **problem upravljanja životnim ciklusom i koordinacijom konkurentnih komponenti**.

---

Na Medior → Senior nivou nije dovoljno znati kako da pokreneš goroutinu. Potrebno je razumeti **ko je odgovoran za njen životni ciklus, pod kojim uslovima se završava i kako sistem garantuje da goroutine neće ostati blokirana zauvek**.

---

### Pitanje 1: Šta je goroutine leak?

**Odgovor:**

Goroutine leak nastaje kada goroutina ostane aktivna iako više nema korisnu funkciju u sistemu, najčešće zato što je trajno blokirana na:

* channel send-u,
* channel receive-u,
* `select` izrazu bez odgovarajuće izlazne grane,
* čekanju na signal koji nikada neće stići,
* nekoj drugoj operaciji koja nema garantovan završetak.

Primer:

```go
func process(input <-chan int) {
    go func() {
        for value := range input {
            fmt.Println(value)
        }
    }()
}
```

Ako `input` nikada nije zatvoren i više niko neće slati podatke, goroutina može ostati zauvek blokirana na:

```go
for value := range input
```

Problem nije samo u tome što jedna goroutina ostaje aktivna. U realnom sistemu leak može nastajati pri svakom zahtevu, poruci ili transakciji, pa broj goroutina može kontinuirano rasti.

---

### Pitanje 2: Kako bi dizajnirao goroutinu tako da njen životni ciklus bude eksplicitno kontrolisan?

**Odgovor:**

Goroutina treba da ima jasno definisan:

1. razlog zbog kojeg je pokrenuta,
2. vlasnika koji je odgovoran za njen životni ciklus,
3. uslov pod kojim završava,
4. mehanizam za signalizaciju završetka,
5. način za propagaciju otkazivanja.

U savremenom Go kodu veoma čest obrazac je korišćenje `context.Context`:

```go
func worker(ctx context.Context, jobs <-chan int) {
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

Ovde postoje dva eksplicitna načina završetka:

```go
case <-ctx.Done():
    return
```

i:

```go
case job, ok := <-jobs:
    if !ok {
        return
    }
```

To je mnogo sigurnije od goroutine koja implicitno pretpostavlja da će joj neki događaj u budućnosti omogućiti završetak.

---

### Pitanje 3: Da li zatvaranje jednog channel-a automatski sprečava goroutine leak?

**Odgovor:**

Ne.

Zatvaranje channel-a sprečava blokiranje goroutina koje upravo čekaju na taj channel, ali sistem može imati druge goroutine koje čekaju na druge resurse ili događaje.

Na primer:

```go
func worker(
    jobs <-chan int,
    results chan<- int,
) {
    for job := range jobs {
        result := process(job)
        results <- result
    }
}
```

Ako je `results` blokiran zato što nema receiver-a, worker može ostati blokiran na:

```go
results <- result
```

čak i ako je `jobs` zatvoren.

Dakle:

> **Channel closure rešava samo one blokade koje su direktno povezane sa zatvorenim channel-om.**

Senior developer mora analizirati **ceo komunikacioni graf**, a ne samo pojedinačni channel.

---

### Pitanje 4: Kako nastaje goroutine leak u producer/consumer obrascu?

**Odgovor:**

Tipičan problem nastaje kada producer šalje podatke, consumer prestane da ih prima, a producer nema način da sazna da više nema downstream receiver-a.

Primer:

```go
func producer(out chan<- int) {
    for i := 0; ; i++ {
        out <- i
    }
}
```

Ako consumer prestane:

```go
for value := range in {
    if value > 10 {
        return
    }
}
```

producer može zauvek ostati blokiran na:

```go
out <- i
```

To je klasičan goroutine leak.

Robusniji dizajn koristi cancellation:

```go
func producer(ctx context.Context, out chan<- int) {
    for i := 0; ; i++ {
        select {
        case <-ctx.Done():
            return

        case out <- i:
        }
    }
}
```

Sada producer ima eksplicitan način da prekine rad kada downstream više nije zainteresovan za rezultate.

---

### Pitanje 5: Šta znači da goroutine ima "structured lifecycle"?

**Odgovor:**

To znači da je odnos između pokretanja, rada i završetka goroutine jasno definisan.

Dobar model izgleda ovako:

```text
parent operation
       │
       ├── starts goroutine
       │
       ├── goroutine performs work
       │
       ├── cancellation/error occurs
       │
       └── goroutine terminates
```

Problematičan model izgleda ovako:

```text
request
   │
   └── go func() {
           // radi nešto
           // nema jasnog vlasnika
           // nema cancellation-a
           // nema deadline-a
       }()
```

Kod drugog pristupa životni ciklus goroutine postaje implicitna stvar.

Senior-level concurrency design nastoji da **lifetime goroutine bude vezan za lifetime operacije kojoj pripada**.

---

### Pitanje 6: Kako bi detektovao goroutine leak u Go aplikaciji?

**Odgovor:**

Prvi korak je posmatranje broja goroutina.

Može se koristiti:

```go
runtime.NumGoroutine()
```

Na primer:

```go
before := runtime.NumGoroutine()

runOperation()

after := runtime.NumGoroutine()

fmt.Println("before:", before)
fmt.Println("after:", after)
```

Međutim, sama razlika u broju nije dokaz leak-a.

Broj goroutina može legitimno privremeno porasti.

Za ozbiljniju analizu koristi se goroutine profile preko `pprof`, odnosno:

```text
goroutine profile
```

koji omogućava da se vidi gde goroutine trenutno postoje i na kojim operacijama čekaju.

U testovima je korisno proveravati da li se broj goroutina vraća približno na očekivani nivo nakon završetka operacije.

Važno je izbegavati testove koji se oslanjaju samo na:

```go
time.Sleep(...)
```

jer takvi testovi često uvode race između samog testa i završetka goroutina.

---

### Senior-level zaključak

Kod concurrency sistema pitanje nije samo:

> "Da li goroutine radi?"

nego:

> "Ko je odgovoran za njen životni ciklus i kako dokazujemo da će se ona završiti?"

To je jedna od ključnih razlika između osnovnog korišćenja goroutina i ozbiljnog concurrency dizajna.

Robustan dizajn treba da omogući da se za svaku značajnu goroutinu odgovori na sledeća pitanja:

```text
Ko ju je pokrenuo?
        ↓
Zašto postoji?
        ↓
Koji resurs koristi?
        ↓
Koji događaj je završava?
        ↓
Kako se propagira cancellation?
        ↓
Kako znamo da je završila?
```

Ako na ova pitanja nema jasnog odgovora, postoji povećan rizik od goroutine leak-a, blokiranja i nekontrolisanog rasta resursa.

---

### 7. Kako biste analizirali situaciju u kojoj goroutine ostaje blokirana na channel operaciji?

Kada goroutine ostane blokirana na channel operaciji, prvo treba utvrditi **zašto druga strana komunikacije nikada ne može da završi odgovarajuću operaciju**.

Na primer:

```go
func worker(ch chan int) {
    value := <-ch
    fmt.Println(value)
}

func main() {
    ch := make(chan int)

    go worker(ch)

    time.Sleep(time.Second)
}
```

Goroutine `worker` čeka:

```go
value := <-ch
```

Pošto nema goroutine koja šalje vrednost:

```go
ch <- value
```

`worker` ostaje blokirana.

Međutim, samo konstatovanje da je goroutine blokirana nije dovoljno. Na Medior → Senior nivou potrebno je analizirati **lifecycle komunikacije**:

1. Ko je vlasnik channel-a?
2. Ko šalje?
3. Ko prima?
4. Ko odlučuje kada je komunikacija završena?
5. Da li će se sender ikada pojaviti?
6. Da li receiver može da čeka zauvek?
7. Da li postoji cancellation mehanizam?
8. Da li je moguće da jedna goroutine zavisi od druge koja je već blokirana?

---

### 7.1. Blokirani sender

Kod nebufferovanog channel-a sender mora da sačeka receiver-a:

```go
ch := make(chan int)

ch <- 42
```

Ako se ova operacija izvršava u goroutine-i:

```go
go func() {
    ch <- 42
}()
```

sender će ostati blokiran dok druga goroutine ne izvrši:

```go
value := <-ch
```

Ovo nije automatski problem.

Naprotiv, upravo takva sinhronizacija predstavlja jednu od osnovnih karakteristika nebufferovanih channel-a.

Problem nastaje kada sistem nema garantovanog receiver-a.

---

### 7.2. Blokirani receiver

Analogno tome:

```go
value := <-ch
```

može ostati blokiran ako nema sender-a.

Na primer:

```go
func worker(ch <-chan int) {
    value := <-ch
    fmt.Println(value)
}
```

Ako producer nikada ne pošalje vrednost, worker nema način da nastavi.

Zato je važno razlikovati:

> **"goroutine čeka"**

od:

> **"goroutine čeka na događaj za koji ne postoji garancija da će se dogoditi."**

Prvi slučaj može biti potpuno ispravan.

Drugi je potencijalni concurrency bug.

---

### 7.3. Blokiranje kao deo protokola

Channel komunikaciju treba posmatrati kao **protokol između goroutina**, a ne samo kao mehanizam prenosa vrednosti.

Na primer:

```text
Producer
   │
   │ data
   ▼
Channel
   │
   │ data
   ▼
Consumer
```

Ako consumer prestane da čita:

```text
Producer ──► Channel ──X──► Consumer
```

producer može ostati blokiran.

Ovo je naročito važno kod pipeline arhitekture.

Na primer:

```go
func producer(out chan<- int) {
    for i := 0; i < 1000; i++ {
        out <- i
    }
}
```

Ako downstream komponenta prestane da konzumira podatke, producer može ostati zauvek blokiran na:

```go
out <- i
```

---

### 7.4. Goroutine leak

Jedna od posledica ovakvog dizajna može biti **goroutine leak**.

Primer:

```go
func producer() <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 1000; i++ {
            out <- i
        }
    }()

    return out
}
```

Consumer može pročitati samo prvu vrednost:

```go
ch := producer()

fmt.Println(<-ch)
```

Nakon toga više niko ne čita iz `ch`.

Producer može ostati blokiran na sledećem:

```go
out <- i
```

Goroutine više nema mogućnost da završi svoj lifecycle.

To je tipičan primer goroutine leak-a.

---

### 7.5. Kako sprečiti leak?

Jedan od osnovnih mehanizama je cancellation.

Na primer:

```go
func producer(ctx context.Context) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 1000; i++ {
            select {
            case out <- i:
            case <-ctx.Done():
                return
            }
        }
    }()

    return out
}
```

Sada producer ne zavisi samo od toga da li consumer čita.

On ima još jedan način da završi:

```text
send successful
       │
       ├──► nastavi
       │
       └──► cancellation
                │
                ▼
              return
```

Ovo je značajna razlika između jednostavnog channel koda i production-grade concurrency koda.

---

### 7.6. Šta interviewer očekuje od Medior → Senior kandidata?

Nije dovoljno reći:

> "Channel blokira kada nema receiver-a."

To je tačno, ali predstavlja osnovno znanje.

Kvalitetniji odgovor treba da pokaže da kandidat razume **sistemsku posledicu blokiranja**:

* goroutine može ostati aktivna neograničeno dugo;
* sender može čekati consumer-a;
* consumer može čekati producer-a;
* pipeline može prestati da napreduje;
* upstream goroutine može biti blokirana zbog downstream komponente;
* goroutine leak može nastati bez klasičnog deadlock-a;
* cancellation treba da bude deo lifecycle dizajna kada je trajanje rada ograničeno;
* ownership channel-a mora biti jasno definisan.

Drugim rečima, senior kandidat ne posmatra samo:

```go
ch <- value
```

nego ceo lifecycle:

```text
creation
   ↓
communication
   ↓
blocking
   ↓
cancellation
   ↓
completion
   ↓
cleanup
```

---

### Ključna poruka

**Channel blocking nije sam po sebi bug.**

Blocking je fundamentalna karakteristika channel komunikacije.

Problem nastaje kada blocking više nije deo kontrolisanog concurrency protokola, već postane posledica neispravnog lifecycle-a, nepostojećeg consumer-a, prekinutog pipeline-a ili nedostatka cancellation mehanizma.

Na Medior → Senior nivou potrebno je zato analizirati **ko može da blokira, zbog koga, koliko dugo i na koji način se garantuje završetak goroutine-e**.

---

### 8. Ko treba da zatvori channel i zašto je ownership važan?

Jedno od važnijih pravila pri dizajniranju Go concurrency sistema jeste:

> **Goroutine koja proizvodi vrednosti i ima znanje da više neće biti novih vrednosti treba da bude odgovorna za zatvaranje channel-a.**

Drugim rečima, channel treba da zatvara strana koja ima odgovornost nad njegovim **send lifecycle-om**.

Tipičan obrazac izgleda ovako:

```go
func producer() <-chan int {
    ch := make(chan int)

    go func() {
        defer close(ch)

        for i := 1; i <= 10; i++ {
            ch <- i
        }
    }()

    return ch
}
```

Consumer dobija samo receive-only channel:

```go
ch := producer()

for value := range ch {
    fmt.Println(value)
}
```

Consumer nema mogućnost da zatvori channel:

```go
<-chan int
```

To nije samo pitanje sintakse.

To je način da API izrazi **ownership**.

---

## 8.1. Zašto consumer ne treba proizvoljno da zatvara channel?

Pretpostavimo:

```go
ch := make(chan int)

go func() {
    for i := 0; i < 10; i++ {
        ch <- i
    }
}()

go func() {
    close(ch)
}()
```

Ovde consumer ili druga goroutine može zatvoriti channel dok sender još uvek pokušava da pošalje vrednost.

Rezultat može biti:

```text
panic: send on closed channel
```

Dakle:

```go
close(ch)
```

nije neutralna operacija.

Ona menja stanje channel-a za sve goroutine koje ga koriste.

---

## 8.2. Close nije signal "meni više ne treba channel"

Ovo je česta greška u razumevanju.

`close(ch)` ne znači:

> "Ja više ne želim da primam podatke."

Već znači:

> **"Nijedna nova vrednost više neće biti poslata kroz ovaj channel."**

Zato je `close` signal o **završetku producer lifecycle-a**.

Na primer:

```go
func producer(ch chan<- int) {
    defer close(ch)

    for i := 0; i < 10; i++ {
        ch <- i
    }
}
```

Producer zna kada je završio slanje.

Consumer to može detektovati:

```go
for value := range ch {
    fmt.Println(value)
}
```

`range` će se završiti tek kada:

1. channel bude zatvoren;
2. i sve prethodno poslate vrednosti budu pročitane.

---

## 8.3. Channel ownership kao arhitektonsko pravilo

U ozbiljnijem kodu korisno je razmišljati o channel-u kao o resursu sa jasno definisanim vlasnikom.

Na primer:

```text
Producer
   │
   │ owns send lifecycle
   ▼
Channel
   │
   │ exposes receive
   ▼
Consumer
```

Producer odlučuje:

* kada počinje proizvodnja;
* koje vrednosti šalje;
* kada više nema vrednosti;
* kada treba zatvoriti channel.

Consumer odlučuje:

* kako obrađuje vrednosti;
* da li može da obradi rezultat;
* kada završava sopstveni posao.

Ali consumer ne treba da odlučuje kada producer više nema šta da pošalje.

To je odgovornost producer-a.

---

## 8.4. Directional channels kao mehanizam zaštite

Go omogućava da ownership dodatno izrazimo kroz tipove:

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

Povratna vrednost je:

```go
<-chan int
```

što znači:

> caller može da prima, ali ne može da šalje niti da zatvara channel.

S druge strane, producer može dobiti:

```go
chan<- int
```

što znači da funkcija može da šalje vrednosti.

Primer:

```go
func producer(out chan<- int) {
    defer close(out)

    for i := 0; i < 10; i++ {
        out <- i
    }
}
```

Ovakav API jasno komunicira nameru.

---

## 8.5. Šta directional channel rešava, a šta ne rešava?

Directional channel sprečava određene klase grešaka na nivou kompajlera.

Na primer:

```go
func consume(ch <-chan int) {
    close(ch)
}
```

nije dozvoljeno.

Isto važi za pokušaj slanja:

```go
func consume(ch <-chan int) {
    ch <- 42
}
```

Kompajler će prijaviti grešku.

Ali directional channel ne može da garantuje da će concurrency protokol biti ispravan.

Na primer:

```go
func consume(ch <-chan int) {
    value := <-ch
    fmt.Println(value)
}
```

Consumer i dalje može ostati blokiran zauvek ako producer nikada ne pošalje vrednost ili zatvori channel.

Dakle:

```text
Directional channel
        ↓
štiti API contract
        ↓
ali ne rešava
        ↓
lifecycle / cancellation / progress probleme
```

---

## 8.6. Ko zatvara channel u pipeline-u?

Kod pipeline-a ownership treba da prati tok podataka.

Na primer:

```go
func stage1() <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 10; i++ {
            out <- i
        }
    }()

    return out
}

func stage2(in <-chan int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for value := range in {
            out <- value * 2
        }
    }()

    return out
}
```

Ovde:

```text
stage1
   │
   │ owns out
   ▼
channel
   │
   ▼
stage2
   │
   │ owns its own out
   ▼
channel
   │
   ▼
consumer
```

Svaki stage je odgovoran za channel koji proizvodi.

To stvara veoma važnu invariantu:

> **Producer zatvara output channel kada završi sa slanjem.**

---

## 8.7. Šta se dešava ako consumer ranije odustane?

Ovde dolazimo do mnogo ozbiljnijeg problema.

Pretpostavimo:

```go
for value := range ch {
    if value == 5 {
        break
    }

    process(value)
}
```

Consumer je završio ranije.

Ali producer možda još uvek radi:

```go
for i := 0; i < 1000000; i++ {
    out <- i
}
```

Nakon što consumer prestane da čita, producer može ostati blokiran.

To je tipičan lifecycle problem.

Zato samo pravilno zatvaranje channel-a nije dovoljno.

Potrebno je razmotriti i **cancellation propagation**.

---

## 8.8. Ownership + cancellation

Robusniji pipeline može izgledati ovako:

```go
func producer(ctx context.Context) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)

        for i := 0; i < 1000000; i++ {
            select {
            case out <- i:
            case <-ctx.Done():
                return
            }
        }
    }()

    return out
}
```

Consumer sada može signalizirati da više ne želi podatke:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

ch := producer(ctx)

for value := range ch {
    if value == 5 {
        cancel()
        break
    }

    process(value)
}
```

Sada postoji eksplicitna veza između:

```text
consumer cancellation
        ↓
context
        ↓
producer
        ↓
producer terminates
        ↓
channel closes
```

To je mnogo pouzdaniji lifecycle model.

---

## 8.9. Senior-level pitanje: ko poseduje pravo da prekine sistem?

Kod kompleksnijih concurrency sistema ownership nije ograničen samo na pitanje:

> "Ko zatvara channel?"

Treba postaviti šira pitanja:

* Ko kreira goroutine?
* Ko je odgovoran za njeno završavanje?
* Ko kreira channel?
* Ko šalje?
* Ko prima?
* Ko zatvara?
* Ko može da otkaže operaciju?
* Kako cancellation propagira kroz sistem?
* Ko garantuje da goroutine neće ostati blokirana?
* Ko je odgovoran za cleanup?

Ovo je suština **lifecycle ownership-a**.

---

## 8.10. Pravilo koje treba zapamtiti

Za tipičan producer/consumer model:

```text
                PRODUCER
                   │
                   │ send
                   ▼
              CHANNEL
                   │
                   │ receive
                   ▼
               CONSUMER
```

Producer je vlasnik send lifecycle-a.

Zato producer, kada zna da više neće slati:

```go
close(ch)
```

Consumer obično samo:

```go
for value := range ch {
    process(value)
}
```

Ako consumer može da završi pre producer-a, potrebno je razmotriti:

```text
context
cancellation
select
lifecycle management
goroutine cleanup
```

Dakle, **channel ownership i goroutine ownership treba posmatrati zajedno**.

To je jedan od ključnih koraka od običnog korišćenja channel-a ka profesionalnom dizajnu concurrent sistema.

---

**Backpressure** je mehanizam kojim sporiji deo sistema signalizira bržem delu da mora da uspori proizvodnju ili da sačeka dok downstream komponenta ne bude spremna.

U Go concurrency sistemima backpressure se vrlo često pojavljuje prirodno kroz **blocking semantics channel-a**.

Na primer:

```go
ch := make(chan int)

go func() {
    for i := 0; i < 100; i++ {
        ch <- i
    }
}()
```

Ako consumer ne čita:

```go
for value := range ch {
    process(value)
}
```

producer će se zaustaviti na:

```go
ch <- i
```

jer je channel nebufferisan.

To znači da channel nije samo mehanizam za prenos podataka.

On može predstavljati i **mehanizam kontrole protoka**.

---

## 9.1. Unbuffered channel kao najjači oblik backpressure-a

Kod:

```go
ch := make(chan int)
```

send i receive moraju da se sinhronizuju.

Producer:

```go
ch <- value
```

ne može nastaviti dok receiver ne prihvati vrednost.

Možemo to predstaviti:

```text
Producer
   │
   │ send
   ▼
CHANNEL
   │
   │ mora postojati receiver
   ▼
Consumer
```

Ako consumer uspori:

```text
Consumer slows down
        ↓
Producer blocks
        ↓
Production slows down
```

Ovo je jednostavan i veoma koristan oblik backpressure-a.

---

## 9.2. Buffered channel menja dinamiku sistema

Kod:

```go
ch := make(chan int, 100)
```

producer može poslati do 100 vrednosti bez trenutnog receiver-a.

Dakle:

```text
Producer
   │
   ▼
┌───────────────┐
│ buffer        │
│ 1 2 3 4 ...   │
└───────────────┘
        │
        ▼
     Consumer
```

Producer neće odmah blokirati kada consumer uspori.

Ali kada se buffer napuni:

```text
buffer full
     ↓
send blocks
```

Zato buffered channel ne uklanja backpressure.

On ga samo **odlaže**.

---

## 9.3. Zašto je veličina buffer-a arhitektonska odluka?

Ovo:

```go
make(chan Job, 10)
```

nije samo tehnička konfiguracija.

Veličina buffer-a utiče na:

* latency;
* throughput;
* memory consumption;
* burst tolerance;
* broj istovremeno pending poslova;
* ponašanje producer-a;
* ponašanje consumer-a;
* recovery nakon kratkotrajnog usporavanja.

Na primer:

```go
make(chan Job, 1)
```

i:

```go
make(chan Job, 10000)
```

predstavljaju veoma različite concurrency sisteme.

Veći buffer može sakriti problem sporog consumer-a.

To može izgledati dobro u testovima, ali u produkciji može dovesti do:

```text
traffic spike
    ↓
buffer grows
    ↓
memory usage grows
    ↓
latency grows
    ↓
system becomes overloaded
```

---

## 9.4. Backpressure i memory consumption

Razmotrimo:

```go
jobs := make(chan Job, 1000000)
```

Ako producer generiše poslove mnogo brže nego što ih worker-i obrađuju, queue može postati veoma velika.

Na primer:

```text
Producer:
10,000 jobs/sec

Consumer:
1,000 jobs/sec
```

Razlika je:

```text
+9,000 pending jobs/sec
```

Ako se stanje nastavi dovoljno dugo, buffer će se napuniti.

Ako je buffer dovoljno velik, problem može dugo ostati neprimećen.

Zato veliki buffer nije automatski "bolje rešenje".

---

## 9.5. Kako worker pool koristi backpressure?

Tipičan worker pool:

```go
jobs := make(chan Job, 100)

for i := 0; i < workers; i++ {
    go worker(jobs)
}
```

Producer:

```go
for _, job := range input {
    jobs <- job
}
```

Ako svi worker-i rade:

```text
Producer
   │
   ▼
jobs channel
   │
   ├── Worker 1
   ├── Worker 2
   ├── Worker 3
   └── Worker 4
```

Kada worker-i postanu zasićeni:

```text
workers busy
    ↓
jobs buffer fills
    ↓
producer blocks
```

To je često poželjno ponašanje.

Sistem tada ne prihvata neograničen broj posla.

---

## 9.6. Zašto je "unbounded queue" opasan koncept?

Ako sistem dozvoljava da producer beskonačno brzo dodaje posao, a consumer ne može da ga obradi istom brzinom, dobija se:

```text
arrival rate > service rate
```

Ako je:

* λ = arrival rate
* μ = service rate

onda je problem kada dugoročno važi:

```text
λ > μ
```

Nijedan finite buffer ne može trajno rešiti ovaj problem.

Buffer samo omogućava sistemu da apsorbuje privremeni burst.

Ako je producer dugoročno brži od consumer-a:

```text
queue ↑
queue ↑
queue ↑
queue ↑
...
```

Na kraju dolazi do:

* blokiranja producer-a;
* odbacivanja posla;
* povećanja latency-ja;
* povećanja memory usage-a;
* timeout-a;
* cascading failure-a.

Senior developer treba da prepozna ovaj problem kao **capacity planning problem**, a ne samo kao channel problem.

---

## 9.7. Backpressure naspram drop strategije

Nisu svi sistemi dizajnirani da blokiraju producer-a.

Nekada je bolje odbaciti podatak.

Na primer:

```go
select {
case ch <- event:
    // poslato
default:
    // channel je trenutno pun
}
```

Ovo znači:

> Ako channel trenutno nije spreman za slanje, nemoj blokirati.

To može biti korisno za:

* telemetry;
* metrics;
* non-critical notifications;
* cache invalidation signals;
* best-effort događaje.

Ali može biti potpuno neprihvatljivo za:

* finansijske transakcije;
* ledger entries;
* payment commands;
* settlement events;
* account balance updates.

Dakle, izbor između:

```text
block
```

i:

```text
drop
```

je **business semantics**, a ne samo concurrency odluka.

---

## 9.8. Backpressure u fintech sistemu

Pretpostavimo da sistem prima payment commands:

```text
API
 ↓
Payment Producer
 ↓
Job Queue
 ↓
Payment Workers
 ↓
Database / External Provider
```

Ako payment provider postane sporiji:

```text
Payment Workers slow down
        ↓
Job Queue fills
        ↓
Producer blocks
```

To može biti poželjno jer sprečava da sistem prihvati neograničen broj zahteva koje ne može da obradi.

Ali sistem mora imati definisanu politiku:

* koliko dugo request sme da čeka;
* kada se vraća timeout;
* da li se posao retry-uje;
* gde se posao durable-uje;
* da li se koristi broker;
* da li se zahtev može bezbedno ponoviti;
* kako se sprečava duplicate processing.

Concurrency mehanizam sam po sebi ne rešava ove probleme.

---

## 9.9. Backpressure u iGaming sistemu

Zamislimo veliki broj betting events:

```text
Players
   │
   ▼
Event ingestion
   │
   ▼
Concurrent processors
   │
   ▼
Settlement / Wallet / Odds
```

Ako downstream komponenta uspori, ingestion sistem može početi da akumulira događaje.

Bez backpressure-a:

```text
events ↑
goroutines ↑
memory ↑
latency ↑
```

Na kraju može doći do:

```text
GC pressure
CPU pressure
timeouts
goroutine explosion
service instability
```

Sa kontrolisanim backpressure-om:

```text
downstream slows
      ↓
queue fills
      ↓
producer throttles
      ↓
system remains bounded
```

Ovo je važna razlika između:

> sistema koji može da obradi veliki throughput

i:

> sistema koji može da preživi overload.

---

## 9.10. Backpressure i goroutine explosion

Jedan od opasnih anti-pattern-a je:

```go
for {
    job := <-jobs

    go process(job)
}
```

Ako `process` traje dugo, broj goroutine-a može neograničeno rasti.

Na primer:

```text
100 jobs/sec
    ↓
100 goroutines/sec
```

Ako svaka goroutine traje 30 sekundi:

```text
~3000 concurrent goroutines
```

Ako workload dodatno poraste:

```text
1000 jobs/sec
    ↓
~30,000 concurrent goroutines
```

Problem nije nužno u samim goroutine-ima.

Problem je odsustvo **bounded concurrency**.

Worker pool uvodi ograničenje:

```text
N workers
```

i time kontroliše maksimalni nivo konkurentnosti.

---

## 9.11. Semaphore kao kontrola konkurentnosti

Jedan način da se ograniči broj aktivnih operacija:

```go
sem := make(chan struct{}, 10)
```

Pre pokretanja operacije:

```go
sem <- struct{}{}

go func() {
    defer func() {
        <-sem
    }()

    process()
}()
```

Na taj način najviše 10 operacija može biti aktivno.

Konceptualno:

```text
                 ┌── worker
                 ├── worker
                 ├── worker
Semaphore ───────┼── ...
                 └── max N
```

Ovo je drugačiji oblik backpressure-a:

```text
bounded concurrency
```

umesto:

```text
unbounded concurrency
```

---

## 9.12. Kako izabrati strategiju?

Senior-level razmišljanje ne počinje pitanjem:

> "Da li da koristim buffered channel?"

Već:

> "Šta treba da se desi kada producer postane brži od consumer-a?"

Moguće strategije su:

| Strategija    | Kada ima smisla                   |
| ------------- | --------------------------------- |
| Block         | posao ne sme biti izgubljen       |
| Bounded queue | želiš ograničenu memoriju         |
| Drop          | događaj nije kritičan             |
| Timeout       | čekanje ima vremensko ograničenje |
| Retry         | operacija je privremeno neuspešna |
| Worker pool   | želiš bounded concurrency         |
| Rate limiting | želiš kontrolu ulaznog protoka    |
| Durable queue | posao mora preživeti restart      |

Pravi izbor zavisi od semantike sistema.

---

## 9.13. Interview pitanje

**Pitanje:**

> Imaš producer koji generiše 10.000 događaja u sekundi i consumer koji može da obradi samo 1.000 događaja u sekundi. Da li bi povećao channel buffer sa 100 na 100.000?

**Loš odgovor:**

> Da, tako ćemo imati više prostora.

**Bolji odgovor:**

> Ne kao primarno rešenje. Buffer može apsorbovati kratkotrajni burst, ali ne rešava situaciju u kojoj je arrival rate dugoročno veći od service rate-a. Potrebno je definisati backpressure strategiju, bounded concurrency, rate limiting, eventualno durable queue, timeout/drop/rejection politiku i monitoring queue depth-a i processing latency-ja.

To je razlika između:

```text
"znam kako channel radi"
```

i:

```text
"znam kako concurrency sistem treba da se ponaša pod load-om"
```

---

## 9.14. Ključna mentalna mapa

Kod projektovanja concurrent sistema treba posmatrati:

```text
          INPUT RATE
              │
              ▼
          PRODUCER
              │
              ▼
        ┌───────────┐
        │   QUEUE   │
        └───────────┘
              │
              ▼
           WORKERS
              │
              ▼
        SERVICE RATE
```

Ako:

```text
input rate < service rate
```

sistem ima kapacitet.

Ako:

```text
input rate ≈ service rate
```

sistem radi blizu granice.

Ako:

```text
input rate > service rate
```

potrebna je eksplicitna overload strategija.

Ta strategija može biti:

```text
backpressure
rate limiting
load shedding
queueing
scaling
retry control
bounded concurrency
```

Zato je **backpressure jedan od centralnih koncepata production-grade concurrency dizajna**.

---

Cilj nije proveravanje pojedinačnih činjenica o gorutinama, channel-ima ili `select` konstrukciji, već sposobnosti da se više concurrency koncepata posmatra kao **jedan sistem**.

Na ovom nivou kandidat treba da ume da objasni:

* kako se concurrency modelira;
* gde se uvode granice konkurentnosti;
* kako se upravlja lifecycle-om gorutina;
* kako se propagira cancellation;
* kako se sprečavaju goroutine leak-ovi;
* kako se rešava backpressure;
* kako se definiše ownership;
* kako se prepoznaju failure modes;
* kako se bira odgovarajući concurrency pattern;
* kako se dizajnira concurrency API koji je bezbedan za korišćenje.

---

## 10.1. Kako bi projektovao production-grade concurrent pipeline?

Pretpostavimo sistem:

```text
Incoming Events
      │
      ▼
   Ingestion
      │
      ▼
    Queue
      │
      ▼
  Processing
      │
      ▼
 External Service
      │
      ▼
   Persistence
```

Naivna implementacija može izgledati ovako:

```go
for event := range events {
    go process(event)
}
```

Ali ovakav dizajn nema jasnu granicu konkurentnosti.

Senior-level pristup počinje definisanjem ograničenja.

Na primer:

```text
                  ┌──────────────┐
Events ──────────►│ bounded queue│
                  └──────┬───────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Worker 1   Worker 2   Worker N
              │          │          │
              └──────────┼──────────┘
                         ▼
                  External Service
```

Potrebno je odgovoriti na pitanja:

1. Koliko worker-a postoji?
2. Koliko događaja može čekati?
3. Šta se događa kada je queue puna?
4. Kako se zaustavlja pipeline?
5. Kako se propagira cancellation?
6. Kako se obrađuje greška?
7. Da li se posao retry-uje?
8. Da li je obrada idempotentna?
9. Šta se događa kada downstream servis postane spor?
10. Kako se meri queue depth?
11. Kako se meri processing latency?
12. Kako se otkriva goroutine leak?

Concurrency dizajn je kompletan tek kada su ova pitanja odgovorena.

---

# 10.2. Kako bi sprečio goroutine leak?

Goroutine leak nastaje kada gorutina ostane aktivna duže nego što je predviđeno zato što ne postoji putanja kojom može normalno da završi.

Na primer:

```go
func worker(ch <-chan Job) {
    for {
        job := <-ch
        process(job)
    }
}
```

Ako producer prestane da šalje podatke, worker ostaje blokiran.

Bolji dizajn može koristiti `range` i zatvaranje channel-a:

```go
func worker(ch <-chan Job) {
    for job := range ch {
        process(job)
    }
}
```

Sada:

```text
close(ch)
   ↓
range završava
   ↓
worker završava
```

Za dugotrajne komponente često je potrebna i eksplicitna cancellation kontrola:

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

Ovde postoje dva načina završetka:

```text
context cancellation
        ili
channel close
```

Senior developer treba da zna koji lifecycle događaj predstavlja svaki od njih.

---

# 10.3. Ko je vlasnik channel-a?

Jedno od najvažnijih pravila kod channel dizajna jeste:

> Komponenta koja kreira channel i ima odgovornost za njegove vrednosti najčešće treba da bude odgovorna i za njegovo zatvaranje.

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

Consumer:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Consumer ne zatvara channel.

Zašto?

Zato što consumer ne zna kada producer završava proizvodnju.

Ako consumer uradi:

```go
close(ch)
```

dok producer još pokušava:

```go
ch <- value
```

može nastati:

```text
panic: send on closed channel
```

Zato ownership nije samo pitanje stila.

On određuje:

* ko kontroliše lifecycle;
* ko sme da zatvori channel;
* ko garantuje završetak;
* ko je odgovoran za cleanup.

---

# 10.4. Kada bi koristio channel, a kada `sync.Mutex`?

Ovo je jedno od najvažnijih interview pitanja.

Ne postoji univerzalno pravilo:

> "Channels su bolji od mutex-a."

To nije tačno.

### Channel

Channel je prirodan kada želimo:

```text
communication
coordination
ownership transfer
pipelines
producer/consumer
```

Na primer:

```go
jobs <- job
```

### Mutex

Mutex je prirodan kada imamo shared state:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}
```

i:

```go
func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()

    c.value++
}
```

Ovde channel nije nužno bolji dizajn.

Možemo razmišljati:

```text
Channel:
"pošalji podatak drugoj gorutini"

Mutex:
"zaštiti ovaj shared state"
```

Senior developer bira mehanizam prema problemu, a ne prema preferenciji.

---

# 10.5. Kako bi rešio cascading failure?

Pretpostavimo:

```text
Service A
   ↓
Service B
   ↓
Service C
```

Ako C postane spor:

```text
C slows down
   ↓
B waits
   ↓
B queue grows
   ↓
A waits
   ↓
A queue grows
   ↓
system overload
```

Concurrency bez ograničenja može pogoršati situaciju.

Na primer:

```go
go callServiceC()
```

za svaki request može dovesti do ogromnog broja aktivnih gorutina.

Production-grade sistem može koristiti kombinaciju:

```text
timeouts
bounded concurrency
rate limiting
backpressure
circuit breaking
load shedding
cancellation
```

Posebno je važno da timeout ne bude samo na HTTP nivou.

Cancellation treba da se propagira kroz call chain:

```text
request
   │
   ▼
context.Context
   │
   ├── database
   ├── external API
   └── internal workers
```

Ako originalni request više nije relevantan, nema smisla da downstream operacije nastave beskonačno da rade.

---

# 10.6. Šta se događa kada parent request završi?

Pretpostavimo:

```go
func Handler(w http.ResponseWriter, r *http.Request) {
    go process(r.Context())
}
```

Ako je gorutina vezana za request lifecycle, `r.Context()` može biti otkazan kada request završi.

To znači da `process` mora poštovati cancellation:

```go
func process(ctx context.Context) {
    select {
    case <-ctx.Done():
        return

    case <-work:
        // obrada
    }
}
```

Ali postoji važna arhitektonska odluka.

Ako posao mora da nastavi da živi i nakon završetka HTTP request-a, onda taj posao ne bi trebalo automatski vezivati za request context.

Na primer:

```text
HTTP request
     │
     ├── request-scoped work
     │
     └── durable background job
```

To su dva različita lifecycle-a.

Senior developer mora jasno razlikovati:

```text
request lifetime
```

od:

```text
business operation lifetime
```

---

# 10.7. Kako bi istraživao production concurrency problem?

Pretpostavimo da servis povremeno ima:

```text
high latency
high CPU
goroutine count ↑
```

Ne treba odmah menjati concurrency kod.

Prvo treba prikupiti podatke.

Tipičan investigation workflow:

```text
Symptom
   ↓
Metrics
   ↓
Profiles
   ↓
Goroutine dump
   ↓
Trace
   ↓
Identify blocking point
   ↓
Identify contention
   ↓
Identify lifecycle problem
   ↓
Fix
   ↓
Benchmark / load test
```

Korisni alati i tehnike uključuju:

* goroutine profiles;
* CPU profiles;
* mutex profiles;
* block profiles;
* `pprof`;
* runtime tracing;
* race detector;
* benchmark testove;
* load testove;
* metrics i distributed tracing.

Na primer, ako goroutine count raste:

```text
goroutines
   │
   ├── waiting on channel
   ├── waiting on mutex
   ├── waiting on I/O
   ├── blocked in select
   └── running
```

Broj gorutina sam po sebi ne govori šta je problem.

Potrebno je razumeti **šta gorutine rade**.

---

# 10.8. Kako bi prepoznao da je concurrency dizajn previše komplikovan?

Jedan od znakova je veliki broj međusobno povezanih channel-a i gorutina:

```text
G1 ──► C1 ──► G2
 │             │
 ├──► C2 ──► G3
 │             │
 └──► C3 ──► G4
        │      │
        └──────┘
```

Ako developer ne može lako da odgovori na:

* ko kreira channel;
* ko ga zatvara;
* ko pokreće gorutinu;
* ko je zaustavlja;
* šta se događa na error;
* šta se događa na cancellation;
* šta se događa kada consumer nestane;

sistem je verovatno previše kompleksan.

Concurrency abstraction treba da smanji kompleksnost, a ne da je sakrije.

Ponekad je jednostavan:

```go
mu.Lock()
defer mu.Unlock()
```

bolji od kompleksnog network-a gorutina i channel-a.

---

# 10.9. Kako bi ocenio concurrency API?

Dobar concurrency API treba da jasno definiše:

### Ownership

Ko poseduje state?

### Lifecycle

Ko pokreće komponentu?

Ko je zaustavlja?

### Cancellation

Kako se prekida rad?

### Error propagation

Kako greške stižu do caller-a?

### Backpressure

Šta se događa kada sistem postane zasićen?

### Concurrency limit

Koliko operacija može biti aktivno?

### Shutdown

Da li se postojeći posao završava ili prekida?

### Safety

Da li API može lako da se koristi pogrešno?

---

# 10.10. Završno interview pitanje

**Pitanje:**

> Dobio si postojeći Go servis koji koristi 15 gorutina, 8 channel-a, nekoliko `select` blokova i worker pool. Povremeno dolazi do povećanja latency-ja i broja gorutina. Kako bi pristupio problemu?

Senior odgovor ne bi trebalo da bude:

> "Povećao bih broj worker-a."

Niti:

> "Dodao bih još jedan buffer."

Umesto toga:

```text
1. Reprodukovati problem
        ↓
2. Izmeriti latency
        ↓
3. Izmeriti goroutine count
        ↓
4. Analizirati goroutine dump
        ↓
5. Analizirati block/mutex profile
        ↓
6. Analizirati CPU/memory
        ↓
7. Pregledati channel ownership
        ↓
8. Pregledati cancellation
        ↓
9. Pregledati backpressure
        ↓
10. Identifikovati bottleneck
        ↓
11. Napraviti minimalnu korekciju
        ↓
12. Benchmark + load test
        ↓
13. Verifikovati production metrike
```

Najvažnija stvar je:

> **Concurrency problem se ne rešava nagađanjem. Rešava se kombinacijom modela, merenja i kontrolisanog eksperimenta.**

---

Poseban fokus bio je na:

* goroutine lifecycle-u;
* channel ownership-u;
* blocking semantics;
* backpressure-u;
* bounded concurrency;
* worker pool-ovima;
* cancellation-u;
* timeout-ima;
* goroutine leak-ovima;
* failure propagation-u;
* cascading failure-ima;
* concurrency API dizajnu;
* observability-ju;
* debugging-u;
* arhitektonskim trade-off-ima.

---

[Prelazak na **Senior — Interview Questions**](../06-senior.md)