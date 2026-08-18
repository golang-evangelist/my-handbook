# Interview Questions — Senior

> **Fajl:** `extras/01-interview-questions/module-1/06-senior.md`
> 
> **Nivo:** Senior
> 
> **Oblast:** #1 — Concurrency Fundamentals

---

Na **Senior** nivou više nije dovoljno znati sintaksu:

```go
go worker()
```

ili:

```go
ch <- value
```

Senior developer mora da razume **šta ovakav kod znači u kontekstu celog sistema**.

To podrazumeva razumevanje odnosa između:

* gorutina;
* Go runtime-a;
* scheduler-a;
* channel-a;
* send/receive operacija;
* blokiranja;
* sinhronizacije;
* ownership-a;
* lifecycle-a;
* memorije;
* konkurentnosti i paralelizma.

U osnovnom delu Module #1 repo obrađuje gorutine, njihove životne cikluse, scheduler, stack i osnovni model izvršavanja, a zatim prelazi na channels i njihovu ulogu u komunikaciji između gorutina.

Na Senior nivou pitanje više nije:

> "Šta je gorutina?"

nego:

> "Koje garancije i koje mogućnosti concurrency modela koristiš kada projektuješ sistem koji mora biti korektan pod konkurentnim izvršavanjem?"

---

# 1.1. Šta zapravo znači da Go podržava concurrency?

Concurrency znači da sistem može imati više nezavisnih jedinica rada koje **napreduju preklapajući se u vremenu**.

Na primer:

```go
go processPayment()
go updateMetrics()
go sendNotification()
```

Ove tri gorutine predstavljaju tri potencijalno konkurentna toka izvršavanja.

Ali concurrency ne znači automatski:

```text
tri CPU jezgra
+
tri gorutine
=
tri stvari se istovremeno izvršavaju
```

To bi bio problem paralelizma.

Možemo napraviti osnovnu razliku:

```text
Concurrency
    ↓
više poslova može napredovati nezavisno

Parallelism
    ↓
više poslova se fizički izvršava istovremeno
```

Go concurrency model omogućava da programer kreira veliki broj gorutina, dok Go runtime scheduler odlučuje kako će ih rasporediti na OS thread-ove i CPU.

---

# 1.2. Da li je concurrency isto što i parallelism?

Ne.

Ovo je jedna od ključnih razlika koju Senior developer mora jasno da objasni.

Pretpostavimo:

```go
go A()
go B()
```

To znači da su `A` i `B` konkurentni.

Ne znači nužno da se:

```text
A ──────────────►
B ──────────────►
```

izvršavaju fizički u isto vreme.

Ako postoji samo jedno dostupno izvršavajuće CPU mesto, scheduler može naizmenično izvršavati:

```text
A A A B B A B B A ...
```

To je concurrency.

Ako postoje odgovarajući CPU resursi i scheduler ih koristi:

```text
CPU 1: A A A A
CPU 2: B B B B
```

onda imamo i parallelism.

Zato:

> **Concurrency je struktura programa, dok je parallelism svojstvo izvršavanja.**

---

# 1.3. Šta se događa kada napišemo `go f()`?

Naivna interpretacija je:

> "Pokreni funkciju na novom thread-u."

To nije precizno.

Kada napišemo:

```go
go f()
```

Go runtime kreira novu gorutinu koja treba da izvrši `f`.

Dalje scheduler upravlja njenim izvršavanjem.

Konceptualno:

```text
go f()
   │
   ▼
new goroutine
   │
   ▼
Go runtime
   │
   ▼
scheduler
   │
   ▼
OS thread
   │
   ▼
CPU
```

Važna posledica jeste da programer ne dobija direktnu kontrolu nad konkretnim OS thread-om na kome će se gorutina izvršavati.

Repo upravo naglašava ovu razliku između gorutine i OS thread-a i činjenicu da Go runtime scheduler upravlja njihovim izvršavanjem.

---

# 1.4. Da li `go` garantuje da će gorutina biti izvršena?

Ne.

Ovo je izuzetno važno.

Kod:

```go
func main() {
    go worker()
}
```

ne postoji garancija da će `worker()` završiti pre nego što se program završi.

Ako `main` završi:

```text
main
 │
 ├── go worker()
 │
 └── return
        │
        ▼
   program exits
```

proces se završava.

Preostale gorutine ne nastavljaju život nezavisno od procesa.

Zato:

```go
go worker()
```

nije mehanizam za:

> "Pokreni posao koji će sigurno biti završen."

To je samo:

> "Pokreni gorutinu čiji lifecycle sada mora biti pravilno kontrolisan."

Repo eksplicitno obrađuje ovu situaciju i pokazuje da `main()` može završiti pre nego što nova gorutina dobije priliku da izvrši svoj kod.

---

# 1.5. Zašto `time.Sleep()` nije concurrency mehanizam?

Početnik može napisati:

```go
func main() {
    go worker()

    time.Sleep(time.Second)
}
```

i dobiti očekivani rezultat.

Ali ovaj kod ne predstavlja pravilnu sinhronizaciju.

Zašto?

Zato što:

```text
sleep duration ≠ completion guarantee
```

Ako je posao gotov za:

```text
10 ms
```

čekamo nepotrebno.

Ako traje:

```text
2 seconds
```

program može završiti prerano.

Dakle:

```go
time.Sleep(time.Second)
```

ne odgovara na pitanje:

> "Da li je gorutina završila?"

Odgovara samo na:

> "Da li je prošla jedna sekunda?"

To su potpuno različite stvari.

Repo upravo zbog toga `time.Sleep()` tretira kao demonstraciono, a ne production rešenje za sinhronizaciju.

---

# 1.6. Šta Senior developer treba da pita umesto "koliko gorutina mogu da pokrenem?"

Pogrešno pitanje:

> Koliko gorutina Go može da ima?

Mnogo bolje pitanje:

> Koliko gorutina moj sistem može bezbedno da održava za konkretan workload?

Broj gorutina zavisi od:

* njihove memorijske potrošnje;
* količine stack-a;
* lifetime-a;
* količine rada;
* blokiranja;
* I/O operacija;
* shared state-a;
* scheduler overhead-a;
* memory pressure-a;
* downstream kapaciteta.

Na primer, ovo:

```go
for {
    go process()
}
```

nije dobar concurrency model samo zato što su gorutine "jeftine".

Ako producer generiše posao mnogo brže nego što sistem može da ga obradi:

```text
jobs ↑
goroutines ↑
memory ↑
latency ↑
```

Concurrency mora biti **bounded** kada workload zahteva granicu.

---

# 1.7. Da li je veliki broj gorutina sam po sebi problem?

Ne.

Ovo je važna nijansa.

Go je upravo dizajniran tako da gorutine budu veoma lagane u poređenju sa OS thread-ovima.

Ali:

```text
lightweight ≠ free
```

Svaka gorutina ipak ima:

* runtime metadata;
* stack;
* lifecycle;
* scheduler overhead;
* potencijalno blokiranje;
* reference na objekte;
* moguće dependency-je na druge gorutine.

Ako imate:

```text
100 goroutines
```

to može biti potpuno normalno.

Ako imate:

```text
1,000,000 goroutines
```

to može biti opravdano u određenom workload-u, ali zahteva vrlo pažljivu analizu.

Ne treba postavljati proizvoljnu granicu:

> "Više od X gorutina je loše."

Pravilnije pitanje je:

> "Zašto postoji toliko gorutina i u kakvom se stanju nalaze?"

---

# 1.8. Goroutine lifecycle kao Senior-level problem

Gorutinu ne treba posmatrati samo kao:

```text
start → execute → finish
```

U realnom sistemu ona može:

```text
Created
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
Blocked
   ↓
Running
   ↓
Finished
```

Na primer, gorutina može biti blokirana na:

```go
ch <- value
```

ili:

```go
value := <-ch
```

ili:

```go
select {
case ...
}
```

ili na sinhronizacionom mehanizmu koji ćemo detaljnije obrađivati u kasnijim modulima.

Zato je za debugging često važnije pitanje:

> "Na čemu gorutina čeka?"

nego:

> "Koliko gorutina imamo?"

---

# 1.9. Kako channel menja concurrency model?

Channel uvodi **komunikacionu granicu** između gorutina.

Na primer:

```go
jobs := make(chan Job)

go producer(jobs)
go consumer(jobs)
```

Sada imamo:

```text
Producer
   │
   │ Job
   ▼
Channel
   │
   │ Job
   ▼
Consumer
```

Channel nije samo struktura podataka.

On uvodi određenu sinhronizacionu semantiku.

Kod unbuffered channel-a send:

```go
jobs <- job
```

ne može jednostavno da se posmatra kao:

> "Dodaj vrednost u kolekciju."

Potrebna je odgovarajuća komunikacija sa receiver-om.

Zbog toga channel može istovremeno predstavljati:

* komunikacioni mehanizam;
* koordinacioni mehanizam;
* synchronization point;
* mehanizam za backpressure.

---

# 1.10. Zašto Senior developer mora razumeti blocking semantics?

Razmotrimo:

```go
ch := make(chan int)

ch <- 42
```

Ako nema gorutine koja prima:

```go
value := <-ch
```

send blokira.

Konceptualno:

```text
Producer
   │
   │ send 42
   ▼
Channel
   │
   │ nema receiver-a
   ▼
BLOCKED
```

Ovo nije greška samo po sebi.

Blocking je deo dizajna channel-a.

Problem nastaje kada blocking nije deo nameravane arhitekture.

Na primer:

```text
Producer
   ↓
Channel
   ↓
Consumer
   ↓
Channel
   ↓
Consumer
   ↓
...
```

Ako bilo koja komponenta prestane da napreduje, downstream blocking može propagirati kroz ceo sistem.

Senior developer mora razumeti **blocking dependency graph**.

---

# 1.11. Da li je blocking uvek loš?

Ne.

Ovo je veoma važna razlika.

Blocking može biti poželjan.

Na primer, unbuffered channel može namerno koristiti blocking da bi obezbedio rendezvous između producer-a i consumer-a.

```text
Producer
   │
   │ send
   │
   ├──── waits ────┐
   │               │
   │               ▼
   │           Consumer
   │               │
   └───────────────┘
```

Problem nije:

> "gorutina je blokirana."

Problem je:

> "gorutina je blokirana bez validnog puta ka nastavku ili završetku."

To je ključna razlika između **intentional blocking** i **accidental blocking**.

---

# 1.12. Kako prepoznati potencijalni deadlock?

Jedan od najjednostavnijih primera:

```go
func main() {
    ch := make(chan int)

    ch <- 42
}
```

Ovde nema receiver-a.

Dakle:

```text
main goroutine
     │
     ▼
send
     │
     ▼
channel
     │
     X
 no receiver
```

Program nema gorutinu koja može omogućiti nastavak send operacije.

Rezultat je deadlock.

Ali Senior developer treba da razume širi princip:

> Deadlock nije samo "channel bez receiver-a".

Deadlock je stanje u kome sistem više ne može da napreduje zato što konkurentne komponente čekaju jedna drugu.

Na primer:

```text
G1 waits for G2
G2 waits for G3
G3 waits for G1
```

Dobijamo ciklus:

```text
G1 → G2 → G3 → G1
```

i nijedna komponenta ne može da napreduje.

---

# 1.13. Šta znači "progress" u concurrent sistemu?

Senior-level analiza concurrency sistema mora posmatrati **progress**.

Pitanje nije samo:

> "Da li je kod thread-safe?"

Nego i:

> "Da li sistem garantovano može da napreduje?"

Na primer, sistem može biti memory-safe, ali potpuno beskoristan ako:

```text
svi worker-i čekaju
```

ili:

```text
producer čeka consumer-a
consumer čeka producer-a
```

Zato concurrency correctness obuhvata više dimenzija:

```text
Correctness
├── safety
├── liveness
├── progress
├── lifecycle
└── resource bounds
```

U praksi treba razmišljati i o:

* deadlock-u;
* starvation-u;
* livelock-u;
* goroutine leak-u;
* neograničenom queue-u;
* neograničenom broju gorutina.

---

# 1.14. Kako biste objasnili razliku između safety i liveness?

**Safety** se odnosi na ono što se ne sme dogoditi.

Na primer:

```text
"Ne smeju postojati dva konkurentna upisa u state bez odgovarajuće zaštite."
```

**Liveness** se odnosi na ono što se mora dogoditi.

Na primer:

```text
"Svaki prihvaćeni posao mora na kraju biti obrađen ili eksplicitno otkazan."
```

Moguće je imati safety bez liveness-a.

Na primer:

```text
nema data race-a
```

ali:

```text
sve gorutine su deadlocked
```

Sistem je možda race-free, ali nije funkcionalan.

Senior developer zato ne sme concurrency correctness svesti samo na:

```text
"nema race detector warning-a"
```

---

# 1.15. Senior interview pitanje

**Pitanje:**

> Imaš Go servis sa 20.000 gorutina. Nema data race-a, ali latency povremeno eksplodira. Da li možeš zaključiti da concurrency model radi ispravno?

**Odgovor:**

Ne.

Odsustvo data race-a samo govori da određeni oblik konkurentnog pristupa memoriji nije detektovan kao data race.

Ne govori ništa direktno o:

* deadlock-u;
* starvation-u;
* goroutine leak-u;
* contention-u;
* prevelikom broju gorutina;
* backpressure-u;
* queue growth-u;
* scheduler overhead-u;
* sporom downstream servisu;
* timeout-u;
* resource exhaustion-u.

Senior analiza mora posmatrati ceo concurrency sistem.

---

# 1.16. Mentalni model Senior developera

Kada vidiš:

```go
go worker()
```

nemoj razmišljati samo:

```text
"Pokrećem gorutinu."
```

Razmišljaj:

```text
Ko je owner?
     ↓
Koliko dugo treba da živi?
     ↓
Ko je zaustavlja?
     ↓
Na čemu može blokirati?
     ↓
Koji resurs koristi?
     ↓
Kako komunicira?
     ↓
Kako dobija grešku?
     ↓
Kako se propagira cancellation?
     ↓
Šta se događa pod overload-om?
     ↓
Šta se događa tokom shutdown-a?
     ↓
Kako dokazujem da radi?
```

To je prelazak sa:

```text
syntax-level concurrency
```

na:

```text
system-level concurrency reasoning
```

---

# 1.17. Završna lekcija ovog dela

Module #1 počinje veoma jednostavno:

```go
go worker()
```

Ali iza ove jedne ključne reči postoji čitav runtime model.

Senior developer mora razumeti da concurrency nije samo sposobnost pokretanja više funkcija.

To je projektovanje sistema u kome više nezavisnih tokova:

* izvršava posao;
* komunicira;
* blokira;
* nastavlja rad;
* završava;
* otkazuje se;
* reaguje na overload;
* koristi ograničene resurse.

Najvažnija mentalna transformacija je:

```text
Junior:
"Kako da pokrenem gorutinu?"

Medior:
"Kako da gorutine komuniciraju?"

Senior:
"Kako da garantujem da ceo concurrent sistem
ostane korektan, ograničen, predvidiv i sposoban
da napreduje pod realnim opterećenjem?"
```

---

### Pitanje 6

**Šta podrazumevaš pod životnim ciklusom goroutine-a i zašto je njegovo eksplicitno upravljanje važno u produkcionom Go programu?**

### Odgovor

Goroutine ima životni ciklus koji možemo konceptualno posmatrati kroz nekoliko faza:

1. kreiranje,
2. izvršavanje,
3. blokiranje ili čekanje,
4. nastavak izvršavanja,
5. završetak.

Najvažnija karakteristika je da goroutine nema ugrađeni mehanizam kojim drugi goroutine može proizvoljno da ga "ubije".

Na primer:

```go
go func() {
    for {
        doWork()
    }
}()
```

Ovaj goroutine će nastaviti da postoji sve dok njegova funkcija ne završi ili dok ceo proces ne bude terminiran.

Zbog toga je odgovornost autora programa da definiše **uslov završetka goroutine-a**.

Tipičan obrazac je korišćenje channel-a:

```go
func worker(stop <-chan struct{}) {
    for {
        select {
        case <-stop:
            return
        default:
            doWork()
        }
    }
}
```

Ovde postoji eksplicitni signal koji worker-u govori da treba da završi.

Još češći obrazac u realnim aplikacijama jeste kombinovanje rada sa `context.Context`:

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

Na ovaj način životni ciklus goroutine-a postaje deo dizajna sistema, a ne slučajna posledica implementacije.

---

### Pitanje 7

**Da li završetak funkcije koja je pokrenula goroutine automatski završava i goroutine koji je ta funkcija pokrenula?**

### Odgovor

Ne.

Goroutine koji poziva funkciju i goroutine koji je pokrenut pomoću `go` naredbe imaju nezavisne tokove izvršavanja.

Na primer:

```go
func main() {
    go func() {
        time.Sleep(time.Second)
        fmt.Println("worker finished")
    }()

    fmt.Println("main finished")
}
```

Kada `main` funkcija završi, završava se i ceo proces.

Zbog toga možda nikada nećemo videti:

```text
worker finished
```

Problem ovde nije u tome što je goroutine automatski prekinut zato što je njegov "parent" goroutine završio.

Pravi razlog je što je **proces završio svoje izvršavanje**.

Ovo je fundamentalna razlika.

Go nema klasičan model:

```text
parent goroutine
        |
        +---- child goroutine
```

u kome završetak parent-a automatski znači završetak child-a.

Ako je potrebno kontrolisati završetak goroutine-a, aplikacija mora eksplicitno implementirati taj mehanizam.

Na primer:

```go
func main() {
    done := make(chan struct{})

    go func() {
        defer close(done)

        time.Sleep(time.Second)
        fmt.Println("worker finished")
    }()

    <-done
}
```

Ovde `main` čeka da worker završi.

Za kompleksnije sisteme mnogo prikladniji mehanizam je često `context.Context` u kombinaciji sa koordinacionim mehanizmom poput `sync.WaitGroup`.

---

### Pitanje 8

**Kako nastaje goroutine leak i zašto je posebno opasan u dugotrajnom serveru?**

### Odgovor

Goroutine leak nastaje kada goroutine više nije koristan, ali nikada ne dobije uslov koji mu omogućava da završi izvršavanje.

Jedan jednostavan primer:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        process(value)
    }
}
```

Ako više niko nikada ne šalje vrednosti kroz `ch`, worker može ostati blokiran na:

```go
value := <-ch
```

Ako više ne postoji način da se taj goroutine legitimno iskoristi, imamo potencijalni leak.

Kod jednog goroutine-a posledice možda nisu značajne.

Ali zamislimo HTTP server koji za svaki request pokreće goroutine:

```go
func handler(w http.ResponseWriter, r *http.Request) {
    go processRequest(r)
}
```

Ako `processRequest` u određenim situacijama ostane trajno blokiran, svaki takav request može ostaviti goroutine iza sebe.

Ako server obrađuje:

```text
10 request/s
```

i leak-uje samo:

```text
1 goroutine / 1000 request-a
```

problem možda neće biti vidljiv odmah.

Ali kod dugotrajnog procesa čak i mali leak rate može vremenom dovesti do velikog broja goroutine-a.

Posledice mogu uključivati:

* povećanu potrošnju memorije,
* povećanje scheduler overhead-a,
* veći GC pritisak,
* povećanje broja aktivnih stack-ova,
* zadržavanje objekata kroz goroutine stack-ove,
* degradaciju performansi,
* eventualno iscrpljivanje resursa procesa.

Zato senior-level concurrency dizajn ne treba da odgovori samo na pitanje:

> "Kako pokrenuti goroutine?"

već prvenstveno:

> "Koji je njegov životni ciklus i pod kojim uslovima garantovano završava?"

---

### Pitanje 9

**Kako bi dizajnirao goroutine koji može istovremeno da prima posao i da bude prekinut?**

### Odgovor

Tipičan obrazac je `select` sa work channel-om i cancellation signalom:

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

Ovaj dizajn rešava dva različita načina završetka:

### 1. Eksplicitno otkazivanje

```go
case <-ctx.Done():
    return
```

Worker završava kada je context otkazan.

### 2. Završetak input stream-a

```go
case job, ok := <-jobs:
    if !ok {
        return
    }
```

Ako producer zatvori `jobs`, worker može kontrolisano da završi.

Time dobijamo jasnu lifecycle semantiku:

```text
            ┌───────────────┐
            │    Worker     │
            └───────┬───────┘
                    │
          ┌─────────┴─────────┐
          │                   │
      jobs channel        ctx.Done()
          │                   │
       work arrives        cancel
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
                 return
```

Važna stvar je da cancellation mora biti propagiran do svih komponenti koje mogu da blokiraju.

Nije dovoljno samo otkazati context ako worker nakon toga ostane blokiran na nekoj drugoj operaciji koja ne reaguje na cancellation.

---

### Pitanje 10

**Zašto je "pokreni goroutine i zaboravi na njega" problematičan obrazac u produkcionom kodu?**

### Odgovor

Sam obrazac nije automatski pogrešan.

Postoje legitimne situacije u kojima možemo pokrenuti background goroutine čiji životni vek odgovara životnom veku procesa.

Problem nastaje kada nema odgovora na sledeća pitanja:

* Ko ga zaustavlja?
* Kada se zaustavlja?
* Ko zna da li je završio?
* Šta ako zavisnost koju koristi postane nedostupna?
* Šta ako dobije cancellation?
* Šta ako dođe do greške?
* Ko upravlja njegovim resursima?
* Kako proveravamo da nije leak-ovan?

Na primer:

```go
func startWorker() {
    go func() {
        for {
            doWork()
        }
    }()
}
```

API:

```go
startWorker()
```

skriva činjenicu da je pokrenut dugotrajan proces.

Pozivalac nema način da ga:

* zaustavi,
* sačeka,
* proveri,
* restartuje,
* dobije informaciju o grešci.

Bolji dizajn često eksplicitno predstavlja lifecycle:

```go
type Worker struct {
    // ...
}

func (w *Worker) Start() {
    // ...
}

func (w *Worker) Stop() {
    // ...
}

func (w *Worker) Wait() {
    // ...
}
```

ili koristi context:

```go
func Run(ctx context.Context) error {
    // worker lifecycle
}
```

Ovakav API čini concurrency ponašanje vidljivim korisniku komponente.

To je važna osobina kvalitetnog concurrent API dizajna.

---

### Pitanje 11

**Da li je svaki goroutine leak rezultat greške u channel komunikaciji?**

### Odgovor

Ne.

Channel je jedan od najčešćih uzroka, ali goroutine može biti leak-ovan zbog različitih vrsta blokiranja.

Na primer:

#### Čekanje na channel

```go
<-ch
```

#### Slanje na channel bez primaoca

```go
ch <- value
```

#### Čekanje na mutex

```go
mu.Lock()
```

#### Čekanje na `WaitGroup`

```go
wg.Wait()
```

#### Čekanje na I/O

Na primer, goroutine može ostati zarobljen u operaciji koja nema odgovarajući timeout ili cancellation mehanizam.

Zato je korisnije razmišljati o problemu kao o:

> **goroutine koji nema garantovani završetak**

umesto samo kao o:

> **goroutine koji čeka na channel-u**

Senior developer treba da posmatra lifecycle cele konkurentne komponente, uključujući:

```text
goroutine
   │
   ├── channels
   ├── mutexes
   ├── wait groups
   ├── context
   ├── timers
   ├── I/O
   └── external dependencies
```

Svaka od tih zavisnosti može uticati na mogućnost završetka goroutine-a.

---

### Pitanje 12

**Kako bi objasnio razliku između "goroutine je blokiran" i "goroutine je leak-ovan"?**

### Odgovor

Blokiranje samo po sebi nije problem.

Blokiranje je normalan deo konkurentnog programa.

Na primer:

```go
value := <-ch
```

može legitimno blokirati goroutine dok ne stigne nova vrednost.

To je očekivano ponašanje.

Problem nastaje kada goroutine ostane blokiran **bez mogućnosti ili namere da se ikada nastavi ili završi**.

Dakle:

```text
Blocked goroutine
       │
       ├── postoji validan uslov nastavka
       │       └── normalno stanje
       │
       └── ne postoji validan uslov nastavka
               └── potencijalni leak
```

Ovo je važna razlika u dijagnostici.

Na primer, worker koji čeka novi posao može biti blokiran sat vremena i to može biti potpuno normalno.

S druge strane, worker koji čeka channel čiji producer više ne postoji verovatno predstavlja leak.

Zato sama činjenica:

> "Imamo goroutine koji je u `chan receive` stanju."

nije dovoljna da zaključimo da postoji bug.

Moramo razumeti **ownership, lifecycle i očekivani završetak** tog goroutine-a.

---

### Pitanje 13

**Koja je razlika između blokiranja goroutine-a i deadlock-a?**

### Odgovor

**Blokiranje (blocking)** znači da goroutine trenutno ne može da nastavi izvršavanje dok se ne ispuni određeni uslov.

Na primer:

```go
value := <-ch
```

Ako nema vrednosti dostupne na `ch`, goroutine će biti blokiran dok neka druga goroutine ne pošalje vrednost ili dok se channel ne zatvori.

Samo blokiranje nije problem. Ono je fundamentalni mehanizam kojim Go omogućava koordinaciju goroutine-a.

**Deadlock** je mnogo ozbiljnije stanje: predstavlja situaciju u kojoj konkurentne komponente međusobno čekaju uslove koji se više nikada neće ispuniti.

Na primer:

```go
func main() {
    ch := make(chan int)

    ch <- 42

    fmt.Println(<-ch)
}
```

Slanje na unbuffered channel blokira dok ne postoji receiver.

Međutim, receiver se u ovom slučaju nikada neće izvršiti, jer je isti goroutine blokiran na send operaciji.

Rezultat je deadlock.

Možemo ga predstaviti:

```text
main goroutine
      │
      ▼
    send
      │
      ▼
   BLOCKED
      │
      X
receiver nikada nije pokrenut
```

Važna senior-level distinkcija je:

> **Blocking je stanje; deadlock je sistemsko stanje bez mogućeg napretka.**

Drugim rečima, možemo imati veliki broj blokiranih goroutine-a u potpuno zdravom sistemu.

Na primer:

```text
worker-1 → čeka posao
worker-2 → čeka posao
worker-3 → čeka posao
```

Ako producer može u budućnosti poslati posao, nema deadlock-a.

Ali:

```text
worker-1 → čeka worker-2
worker-2 → čeka worker-1
```

predstavlja klasičan ciklus čekanja.

---

### Pitanje 14

**Kako nastaje deadlock u sistemu sa više channel-a?**

### Odgovor

Deadlock se može pojaviti kada goroutine-i formiraju ciklus međusobnog čekanja.

Na primer:

```go
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        value := <-ch1
        ch2 <- value
    }()

    ch1 <- 42
}
```

Ovaj konkretan primer nije deadlock zato što `main` šalje vrednost kroz `ch1`, a worker je spreman da je primi.

Ali ako promenimo koordinaciju:

```go
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        value := <-ch1
        ch2 <- value
    }()

    <-ch2
}
```

Sada `main` čeka vrednost sa `ch2`.

Worker čeka vrednost sa `ch1`.

Niko ne šalje na `ch1`.

Dobijamo:

```text
main
 │
 └── receive ch2
          │
          ▼
       BLOCKED


worker
 │
 └── receive ch1
          │
          ▼
       BLOCKED
```

Nema goroutine-a koji može da napravi progres.

To je deadlock.

Kod kompleksnijih sistema isti problem može biti mnogo manje očigledan:

```text
Producer
   │
   ▼
 channel A
   │
   ▼
Worker A
   │
   ▼
 channel B
   │
   ▼
Worker B
   │
   ▼
 channel C
   │
   ▼
Worker C
   │
   └──────────────► channel A
```

Ako svaki element pipeline-a čeka sledeći element u ciklusu, sistem može prestati da napreduje.

Zato senior analiza concurrency sistema mora da posmatra **graf zavisnosti i komunikacije**, a ne samo pojedinačne funkcije.

---

### Pitanje 15

**Šta je ciklus čekanja (wait cycle) i kako ga prepoznaješ?**

### Odgovor

Wait cycle nastaje kada A čeka B, dok B direktno ili indirektno čeka A.

Najjednostavniji oblik:

```text
A → čeka B
B → čeka A
```

Ali u realnom sistemu ciklus može imati više koraka:

```text
A → B
B → C
C → D
D → A
```

Na primer:

```text
goroutine A
    │
    ▼
 mutex X

goroutine B
    │
    ▼
 mutex Y
```

Ako A drži X i čeka Y:

```text
A:
X.Lock()
Y.Lock()
```

dok B drži Y i čeka X:

```text
B:
Y.Lock()
X.Lock()
```

dobijamo:

```text
A ──wait──> B
B ──wait──> A
```

Kod channel-a se ista ideja pojavljuje kada goroutine-i čekaju send/receive operacije koje međusobno zavise jedna od druge.

Jedan od korisnih načina razmišljanja jeste da concurrency sistem posmatramo kao graf:

```text
Node = goroutine / resource
Edge = "čeka na"
```

Ako graf sadrži ciklus u kome nijedna komponenta ne može da napravi progres bez druge, postoji potencijal za deadlock.

Ovakav način razmišljanja je posebno važan kod:

* kompleksnih pipeline-ova,
* worker pool-ova,
* kombinovanja channel-a i mutex-a,
* višestepenih sistema,
* shutdown procedura,
* servisnih komponenti koje međusobno čekaju.

---

### Pitanje 16

**Zašto kombinovanje mutex-a i channel-a zahteva posebno pažljiv dizajn?**

### Odgovor

Zato što možemo napraviti situaciju u kojoj jedna goroutine drži mutex dok čeka channel operaciju, dok druga goroutine pokušava da uzme isti mutex kako bi proizvela vrednost na tom channel-u.

Na primer, konceptualno:

```go
mu.Lock()

ch <- value

mu.Unlock()
```

Ako receiver mora da uzme isti mutex pre nego što može da obradi vrednost:

```go
mu.Lock()

value := <-ch

mu.Unlock()
```

možemo dobiti:

```text
Goroutine A
    │
    ├── Lock(mu)
    │
    └── send(ch)
             │
             ▼
          BLOCKED


Goroutine B
    │
    ├── receive(ch)
    │
    └── Lock(mu)
             │
             ▼
          BLOCKED
```

A drži mutex i čeka receiver-a.

B čeka A da oslobodi mutex.

To je deadlock.

Zbog toga je opšti princip:

> **Ne držati lock dok se obavlja operacija koja može neograničeno blokirati, osim ako je takvo ponašanje eksplicitno deo dizajna i dokazano bezbedno.**

Ovo nije apsolutno pravilo — postoje legitimni dizajni gde je lock namerno zadržan tokom određene operacije — ali takav dizajn zahteva jasno obrazloženje.

Posebno treba biti oprezan sa:

* channel send/receive,
* I/O operacijama,
* network pozivima,
* `WaitGroup.Wait()`,
* drugim mutex-ima,
* dugotrajnim callback-ovima.

---

### Pitanje 17

**Kako bi analizirao situaciju u kojoj program ima veliki broj goroutine-a, ali nijedan očigledan deadlock?**

### Odgovor

Prvo ne bih pretpostavio da veliki broj goroutine-a znači deadlock.

Moguće je da sistem legitimno koristi veliki broj goroutine-a.

Analiza bi počela utvrđivanjem stanja goroutine-a.

Pitanja koja bih postavio su:

1. Koliko goroutine-a postoji?
2. Koliko dugo postoje?
3. U kom stanju se nalaze?
4. Na čemu su blokirani?
5. Da li se broj goroutine-a kontinuirano povećava?
6. Da li se nakon završetka posla smanjuje?
7. Da li postoje goroutine-i koji čekaju channel?
8. Da li čekaju mutex?
9. Da li čekaju I/O?
10. Da li čekaju `WaitGroup`?
11. Da li postoji očekivani shutdown signal?

Posebno je važna dinamika:

```text
broj goroutine-a
      │
      │        /\
      │       /  \
      │      /    \
      │_____/      \____
      │
      └────────────────── time
```

Ako broj goroutine-a raste tokom opterećenja, a zatim se vraća na normalan nivo, to može biti potpuno očekivano.

Nasuprot tome:

```text
broj goroutine-a
      │
      │          /
      │         /
      │        /
      │       /
      │      /
      │_____/
      │
      └────────────────── time
```

može ukazivati na goroutine leak.

U praktičnoj analizi korisni su:

* goroutine profile,
* stack trace-ovi,
* runtime metrike,
* `pprof`,
* tracing,
* application-level metrike.

Na primer:

```go
import "runtime"

fmt.Println(runtime.NumGoroutine())
```

može dati osnovnu informaciju o trenutnom broju goroutine-a.

Ali sam broj nije dovoljan.

Senior analiza mora odgovoriti na pitanje:

> **Zašto postoje baš ti goroutine-i i da li je njihovo trenutno stanje očekivano?**

---

### Pitanje 18

**Kako bi dizajnirao shutdown za sistem koji ima više međusobno povezanih goroutine-a?**

### Odgovor

Shutdown treba da bude deo arhitekture sistema, a ne naknadno dodat mehanizam.

Tipičan pristup je:

```text
                Context
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Worker A  Worker B  Worker C
          │        │        │
          └────────┼────────┘
                   ▼
                 Wait
```

Context predstavlja signal:

> "Sistem treba da prestane sa radom."

Svaka komponenta treba da reaguje na taj signal.

Na primer:

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

Na višem nivou možemo imati:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

var wg sync.WaitGroup

wg.Add(3)

go func() {
    defer wg.Done()
    workerA(ctx)
}()

go func() {
    defer wg.Done()
    workerB(ctx)
}()

go func() {
    defer wg.Done()
    workerC(ctx)
}()

// ...
cancel()

wg.Wait()
```

Ovde imamo jasnu separaciju odgovornosti:

* `context` — signalizira cancellation,
* goroutine-i — odgovaraju na cancellation,
* `WaitGroup` — omogućava pozivaocu da sačeka završetak.

Međutim, kod ozbiljnijih sistema treba razmotriti i redosled gašenja.

Na primer:

```text
1. Stop accepting new work
        ↓
2. Signal cancellation
        ↓
3. Finish/abort in-flight work
        ↓
4. Close internal resources
        ↓
5. Wait for workers
        ↓
6. Exit
```

Ako se channel-i, producer-i i consumer-i gase u pogrešnom redosledu, shutdown sam može postati izvor:

* deadlock-a,
* panic-a,
* izgubljenih podataka,
* goroutine leak-a.

Zato je **shutdown path** jednako važan kao i normalni execution path.

---

### Pitanje 19

**Zašto je lifecycle management posebno važan kod dugotrajnih Go servisa?**

### Odgovor

Kod kratkotrajnog CLI programa goroutine leak često može proći neprimećeno jer proces ubrzo završi.

Kod dugotrajnog servisa situacija je potpuno drugačija.

Na primer:

```text
HTTP request
     │
     ▼
start goroutine
     │
     ▼
external operation
     │
     X
   leak
```

Ako servis radi danima ili mesecima, čak i mali broj leak-ovanih goroutine-a može akumulirati značajnu količinu resursa.

To može uticati na:

* memoriju,
* scheduler,
* GC,
* file descriptors,
* network connections,
* timers,
* queues,
* downstream servise.

Zbog toga je concurrency lifecycle deo operativne pouzdanosti sistema.

Senior developer treba da razmišlja ne samo o:

```text
"Da li kod radi?"
```

već i o:

```text
"Da li kod nastavlja da radi ispravno
nakon miliona operacija?"
```

To je jedna od ključnih razlika između lokalno funkcionalnog concurrency koda i production-grade concurrency arhitekture.

---

### Pitanje 20

**Kako bi razlikovao privremeno povećanje broja goroutine-a od sistemskog goroutine leak-a?**

### Odgovor

Posmatrao bih ponašanje kroz vreme, pod kontrolisanim opterećenjem.

Na primer, ako sistem dobija više posla:

```text
load ↑
goroutines ↑
```

to samo po sebi nije problem.

Ako se nakon smanjenja opterećenja broj goroutine-a vrati:

```text
load ↑
goroutines ↑

load ↓
goroutines ↓
```

ponašanje je verovatno očekivano.

Kod leak-a često dobijamo:

```text
request volume
████████████████████

goroutines
▂▃▄▅▆▇██████████████
```

Broj goroutine-a nastavlja da raste ili ostaje trajno iznad očekivanog baseline-a.

Još važnije, potrebno je identifikovati **koji tip goroutine-a raste**.

Na primer:

```text
worker goroutines     → stabilno
HTTP handlers         → stabilno
background pollers    → stabilno
blocked receivers     → kontinuirani rast
```

To je mnogo korisnija informacija od same metrike:

```text
runtime.NumGoroutine()
```

U produkcionom sistemu zato ima smisla kombinovati:

* broj goroutine-a,
* goroutine profile,
* latency,
* memory usage,
* GC metrike,
* queue depth,
* request rate,
* error rate.

Time možemo povezati lifecycle problem sa njegovim sistemskim posledicama.

---

## Senior-level zaključak

Kod senior-level concurrency pitanja nije dovoljno znati sintaksu:

```go
go func() {}
```

ili:

```go
select {}
```

Potrebno je razumeti **napredak sistema kroz vreme**.

Ključna pitanja su:

```text
Ko pokreće goroutine?
        ↓
Ko mu daje posao?
        ↓
Na čemu može da blokira?
        ↓
Ko ga može zaustaviti?
        ↓
Kako zna da treba da završi?
        ↓
Ko čeka njegov završetak?
        ↓
Šta se dešava ako završi neočekivano?
        ↓
Šta se dešava tokom shutdown-a?
```

Ako na ova pitanja ne postoji jasan odgovor, concurrency dizajn verovatno još nije dovoljno robustan za production environment.

---

### Pitanje 21

**Šta znači "ownership" channel-a u Go concurrency dizajnu?**

### Odgovor

Ownership channel-a znači da je jasno definisano:

* ko kreira channel,
* ko šalje vrednosti,
* ko prima vrednosti,
* ko odlučuje kada više neće biti novih vrednosti,
* ko zatvara channel,
* ko upravlja životnim ciklusom komunikacionog toka.

Ovo je veoma važan koncept jer veliki broj concurrency problema nastaje upravo zbog nejasnog ownership-a.

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

Ovde je ownership relativno jasan.

`producer`:

1. kreira channel,
2. jedini šalje vrednosti,
3. zatvara channel kada završi produkciju.

Consumer samo prima:

```go
for value := range producer() {
    fmt.Println(value)
}
```

Consumer nema razlog da zatvara channel.

Ovakav dizajn možemo opisati kao:

```text
Producer
   │
   │ owns
   ▼
channel
   │
   │ read-only
   ▼
Consumer
```

Ownership je posebno važan kod zatvaranja channel-a.

Praktično pravilo je:

> **Komponenta koja je odgovorna za produkciju vrednosti i zna da više neće biti vrednosti obično treba da bude komponenta koja zatvara channel.**

Consumer uglavnom ne treba da zatvara channel koji nije njegov.

---

### Pitanje 22

**Zašto je "receiver should not close the channel" korisno pravilo?**

### Odgovor

Zato što receiver često ne zna da li postoje drugi producer-i koji još uvek šalju vrednosti.

Na primer:

```go
func producer(id int, ch chan<- int) {
    for i := 0; i < 10; i++ {
        ch <- i
    }
}
```

Ako imamo više producer-a:

```go
ch := make(chan int)

go producer(1, ch)
go producer(2, ch)
go producer(3, ch)
```

Consumer:

```go
for value := range ch {
    process(value)
}
```

Consumer ne zna kada je poslednji producer završio.

Ako bi consumer proizvoljno uradio:

```go
close(ch)
```

producer koji još uvek pokušava:

```go
ch <- value
```

može izazvati panic:

```text
panic: send on closed channel
```

Zato ownership mora biti definisan na višem nivou.

Kod više producer-a često imamo poseban koordinacioni goroutine koji prati njihove završetke:

```text
Producer A ──┐
Producer B ──┼──> channel ──> Consumer
Producer C ──┘
       │
       ▼
   coordinator
       │
       ▼
   close(channel)
```

Na taj način channel se zatvara tek kada je garantovano da više neće biti send operacija.

---

### Pitanje 23

**Kako channel directions (`chan<-` i `<-chan`) doprinose bezbednosti concurrency API-ja?**

### Odgovor

Channel directions omogućavaju da API eksplicitno ograniči šta određena komponenta sme da radi sa channel-om.

Imamo:

```go
chan T
```

što znači da je channel bidirekcionalan.

Zatim:

```go
chan<- T
```

što znači:

> send-only channel

i:

```go
<-chan T
```

što znači:

> receive-only channel.

Na primer:

```go
func producer(out chan<- int) {
    out <- 42
}
```

Unutar `producer` funkcije ne možemo da primamo:

```go
value := <-out
```

jer je `out` send-only.

Slično:

```go
func consumer(in <-chan int) {
    value := <-in
    fmt.Println(value)
}
```

Consumer ne može da šalje:

```go
in <- 42
```

Ovo je važno jer type system sada izražava concurrency contract.

Umesto:

```go
func process(ch chan int)
```

možemo imati:

```go
func producer(out chan<- int)
func consumer(in <-chan int)
```

što odmah dokumentuje nameru.

To smanjuje mogućnost grešaka i čini API lakšim za razumevanje.

Senior developer bi trebalo da razmišlja o channel direction-u kao o delu **API contract-a**, a ne samo kao o sintaktičkoj mogućnosti Go jezika.

---

### Pitanje 24

**Kako bi dizajnirao funkciju koja proizvodi podatke bez izlaganja internog channel-a kao send/receive bidirekcionalnog resursa?**

### Odgovor

Tipičan obrazac je da funkcija vrati receive-only channel:

```go
func generate() <-chan int {
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

Pozivalac dobija:

```go
values := generate()
```

i može samo da prima:

```go
for value := range values {
    fmt.Println(value)
}
```

Ne može da uradi:

```go
values <- 100
```

niti:

```go
close(values)
```

jer njegov tip nije bidirekcionalni channel.

Ovo ima važnu posledicu za enkapsulaciju:

```text
                        internal
                     ┌─────────────┐
                     │  producer   │
                     │     │       │
                     │     ▼       │
                     │  chan int   │
                     └─────┬───────┘
                           │
                           │ <-chan
                           ▼
                       consumer
```

Interna komponenta kontroliše produkciju i lifecycle.

Spoljni kod dobija samo mogućnost čitanja.

Ovo je jedan od najjednostavnijih načina da se concurrency contract učini eksplicitnim.

---

### Pitanje 25

**Da li channel treba tretirati kao ownership mehanizam, queue, synchronization primitive ili sve navedeno?**

### Odgovor

Može biti sve navedeno, ali senior developer mora jasno razlikovati **koju ulogu channel ima u konkretnom dizajnu**.

Channel može služiti kao:

### 1. Communication mechanism

```go
ch <- value
```

Jedna goroutine prosleđuje podatak drugoj.

### 2. Synchronization mechanism

Na primer:

```go
done := make(chan struct{})

go func() {
    doWork()
    close(done)
}()

<-done
```

Ovde channel prvenstveno predstavlja signalizaciju.

### 3. Work queue

```go
jobs := make(chan Job, 100)
```

Producer-i ubacuju poslove, worker-i ih preuzimaju.

### 4. Backpressure mechanism

Buffered channel može ograničiti koliko posla može da bude akumulirano:

```go
jobs := make(chan Job, 100)
```

Kada je buffer pun, producer može biti blokiran.

To omogućava sistemu da prirodno uspori producer-a kada consumer-i ne mogu da obrade posao dovoljno brzo.

### 5. Lifecycle signal

Na primer:

```go
<-ctx.Done()
```

iako ovde zapravo koristimo channel koji izlaže `Context`.

Zbog toga nije dobro reći:

> "Channel je samo queue."

Channel je generalniji concurrency primitive.

Njegova konkretna semantika zavisi od arhitekture u kojoj se koristi.

---

### Pitanje 26

**Kada channel predstavlja dobar API boundary između komponenti?**

### Odgovor

Channel je dobar API boundary kada komunikacija prirodno predstavlja **tok događaja, poslova ili rezultata kroz vreme**.

Na primer:

```text
HTTP ingestion
      │
      ▼
   jobs chan
      │
      ▼
 worker pool
      │
      ▼
 results chan
      │
      ▼
 aggregator
```

Ovde channel jasno predstavlja tok podataka.

Dobar je i kada želimo:

* producer/consumer model,
* pipeline,
* streaming,
* fan-out/fan-in,
* event processing,
* asinhronu obradu.

Međutim, channel nije automatski najbolji API boundary.

Ako funkcija treba samo da zaštiti stanje:

```go
type Counter struct {
    mu sync.Mutex
    n  int
}
```

channel može biti nepotrebna komplikacija.

Na primer, nije nužno dobro napraviti:

```text
goroutine
   ↓
request channel
   ↓
state owner goroutine
   ↓
response channel
```

samo da bismo zaštitili jedan integer.

U tom slučaju mutex može biti jednostavniji i direktniji.

Senior-level pitanje nije:

> "Možemo li ovo rešiti channel-om?"

nego:

> **"Da li channel predstavlja prirodan model komunikacije između ovih komponenti?"**

---

### Pitanje 27

**Kako prepoznaješ da je channel-based dizajn postao previše komplikovan?**

### Odgovor

Jedan od signala je kada moramo pratiti veliki broj implicitnih pravila:

```text
channel A zatvara B
channel C mora biti zatvoren pre D
worker E mora završiti pre F
producer G ne sme slati posle H
```

Ako je lifecycle teško objasniti jednostavnim dijagramom, moguće je da je concurrency model previše složen.

Drugi signali su:

* veliki broj međusobno povezanih channel-a,
* veliki broj goroutine-a sa malim odgovornostima,
* kompleksan shutdown,
* teško testiranje,
* potreba za mnogim `select` granama,
* teško utvrđivanje ownership-a,
* channel koji služi za više potpuno različitih namena,
* callback + channel + mutex kombinacije,
* skriveno blokiranje.

Na primer:

```text
A → ch1 → B → ch2 → C
↑                  ↓
└────── ch5 ← D ← ch3
         ↑
        ch4
```

Možda je takav dizajn opravdan.

Ali možda postoji jednostavnija arhitektura.

Senior developer mora biti sposoban da proceni **trade-off**, a ne samo da implementira concurrency primitive.

---

### Pitanje 28

**Koji je osnovni princip za određivanje ownership-a u producer/consumer sistemu?**

### Odgovor

Najvažnije pitanje je:

> **Ko ima dovoljno informacija da zna kada je stream završen?**

Ta komponenta je najčešće kandidat za ownership lifecycle-a channel-a.

Na primer:

```go
func produce(out chan<- Event) {
    defer close(out)

    for _, event := range events {
        out <- event
    }
}
```

Producer zna da je završio listu događaja.

Zato on zatvara channel.

Consumer:

```go
func consume(in <-chan Event) {
    for event := range in {
        handle(event)
    }
}
```

samo čita dok stream ne bude zatvoren.

Ovakav model je čist:

```text
Producer
   │
   │ owns production
   │
   ▼
Channel
   │
   │ lifecycle controlled by producer
   ▼
Consumer
```

Kod kompleksnijih sistema ownership može biti izdvojen u coordinator komponentu.

Bitno je da postoji **jedan jasan autoritet za lifecycle**.

---

## Senior-level princip

Dobar concurrency API ne definiše samo tipove podataka.

On definiše i:

* ko proizvodi,
* ko konzumira,
* ko blokira,
* ko otkazuje,
* ko zatvara,
* ko čeka,
* ko poseduje lifecycle.

Možemo ga posmatrati kao ugovor:

```text
             CONCURRENCY CONTRACT

       ┌───────────────────────────┐
       │       Component A         │
       │                           │
       │  owns → channel           │
       │  sends → values           │
       │  closes → channel         │
       └─────────────┬─────────────┘
                     │
                     ▼
              ┌─────────────┐
              │   Channel   │
              └──────┬──────┘
                     │
                     ▼
       ┌───────────────────────────┐
       │       Component B         │
       │                           │
       │  receives → values        │
       │  cannot close             │
       │  cannot send              │
       └───────────────────────────┘
```

Kada je ovaj ugovor jasan, concurrency kod je značajno lakše analizirati, testirati i održavati.

---

### Pitanje 29

**Šta je backpressure u concurrent sistemu i kako ga Go channel može implementirati?**

### Odgovor

**Backpressure** je mehanizam kojim sporiji deo sistema može da ograniči brzinu kojom brži deo sistema proizvodi posao.

Problem možemo predstaviti ovako:

```text
Producer
   │
   │ 10.000 jobs/s
   ▼
 Queue
   │
   │ 1.000 jobs/s
   ▼
Consumer
```

Ako producer kontinuirano proizvodi deset puta više posla nego što consumer može da obradi, sistem mora imati neku strategiju.

Bez kontrole, queue može neograničeno da raste.

To dovodi do:

* rasta memorije,
* povećanja latencije,
* GC pritiska,
* eventualnog OOM-a,
* cascading failure-a.

Buffered channel može predstavljati ograničeni buffer:

```go
jobs := make(chan Job, 100)
```

Kada postoji prostor:

```text
Producer
   │
   ▼
┌─────────────┐
│ Job buffer  │
│  capacity   │
│     100     │
└─────────────┘
   │
   ▼
Consumer
```

Producer može da nastavi da šalje dok se buffer ne popuni.

Kada se popuni:

```text
Producer
   │
   ▼
████████████████████
      FULL
        │
        ▼
     BLOCKED
```

Producer tada mora da sačeka dok consumer ne oslobodi prostor.

To je oblik backpressure-a.

Važna ideja je:

> **Bounded queue pretvara neograničeni rast rada u kontrolisano usporavanje producer-a.**

---

### Pitanje 30

**Zašto buffered channel nije isto što i "uvek bolje performanse"?**

### Odgovor

Buffered channel može smanjiti broj trenutnih blokiranja producer-a, ali to ne znači automatski da će sistem biti brži.

Na primer:

```go
jobs := make(chan Job, 10)
```

i:

```go
jobs := make(chan Job, 100000)
```

imaju potpuno različitu semantiku.

Veliki buffer može:

* apsorbovati burst saobraćaja,
* smanjiti trenutno blokiranje,
* povećati throughput u određenim scenarijima.

Ali takođe može:

* povećati memorijsku potrošnju,
* sakriti problem sporog consumer-a,
* povećati queueing latency,
* odložiti trenutak kada sistem primeni backpressure.

Na primer:

```text
Producer: 10.000 jobs/s
Consumer:  1.000 jobs/s
```

Sa bufferom od `100000`, sistem može dugo prihvatati višak.

Ali matematički problem ostaje:

```text
production rate > consumption rate
```

Buffer ne rešava osnovni throughput mismatch.

On samo odlaže njegovu manifestaciju.

Možemo to predstaviti:

```text
bez buffera:

Producer ──BLOCK──> Consumer


sa malim bufferom:

Producer ──> [██████] ──> Consumer
                 │
                 └── eventualni BLOCK


sa ogromnim bufferom:

Producer ──> [████████████████████████████] ──> Consumer
                       │
                       └── problem samo kasnije postaje vidljiv
```

Senior developer zato ne bira buffer veličinu samo na osnovu pitanja:

> "Koliko je veće brže?"

Već razmatra workload, burstiness, latency, memoriju i failure semantics.

---

### Pitanje 31

**Kako određuješ odgovarajuću veličinu buffered channel-a?**

### Odgovor

Ne postoji univerzalna optimalna vrednost.

Veličina buffer-a treba da proizilazi iz očekivanog ponašanja sistema.

Treba razmotriti:

1. prosečan throughput,
2. peak throughput,
3. burst duration,
4. consumer processing time,
5. dozvoljenu latenciju,
6. memorijski budžet,
7. prirodu posla,
8. šta treba da se desi kada je buffer pun.

Na primer, ako imamo kratke burst-ove:

```text
normal:
100 jobs/s

burst:
500 jobs/s
trajanje:
2 sekunde
```

buffer može apsorbovati deo razlike između producer-a i consumer-a.

Ali ako je producer dugoročno brži:

```text
Producer = 500 jobs/s
Consumer = 100 jobs/s
```

buffer će se pre ili kasnije napuniti.

Tada moramo odlučiti šta sistem radi:

```text
buffer full
    │
    ├── block producer
    ├── reject job
    ├── drop job
    ├── retry later
    ├── persist externally
    └── scale consumers
```

To je arhitektonska odluka.

Zato je pitanje:

> "Da li channel treba da ima buffer 100 ili 1000?"

manje važno od:

> **"Koja je semantika sistema kada je buffer pun?"**

---

### Pitanje 32

**Koja je razlika između backpressure-a i rate limiting-a?**

### Odgovor

To su povezani, ali različiti koncepti.

**Backpressure** obično nastaje kao reakcija downstream komponente na njenu trenutnu sposobnost obrade.

Na primer:

```text
Consumer spor
     ↓
Queue full
     ↓
Producer blocked
```

**Rate limiting** namerno ograničava brzinu pristizanja ili obrade prema unapred definisanom pravilu.

Na primer:

```text
API
 │
 ▼
Rate limiter
 │
 ├── 100 requests/s
 │
 ▼
Workers
```

Rate limiter kaže:

> "Dozvoljeno je najviše N operacija u jedinici vremena."

Backpressure kaže:

> "Downstream trenutno ne može da obradi dodatni posao, zato upstream mora da uspori ili promeni ponašanje."

U realnom sistemu mogu postojati zajedno:

```text
Client
   │
   ▼
Rate Limiter
   │
   ▼
Bounded Queue
   │
   ▼
Worker Pool
   │
   ▼
Database
```

Rate limiting štiti sistem od prevelike ulazne brzine.

Backpressure štiti sistem kada downstream ne može da prati trenutnu brzinu obrade.

---

### Pitanje 33

**Šta se dešava ako consumer prestane da obrađuje vrednosti, a producer nastavi da šalje na buffered channel?**

### Odgovor

Ako je channel bounded:

```go
jobs := make(chan Job, 100)
```

producer može poslati određeni broj vrednosti:

```text
Producer
   │
   ▼
┌───────────────────┐
│ 100 buffered jobs │
└───────────────────┘
```

Kada se buffer napuni:

```go
jobs <- job
```

će blokirati.

Ako consumer nikada više ne pročita vrednost, producer ostaje blokiran.

Ako je producer kritičan za shutdown ili sistemsku koordinaciju, moguće je da će i druge goroutine zavisiti od njega.

Tada možemo dobiti cascading blockage:

```text
Consumer
   X
   │
   ▼
Queue full
   │
   ▼
Producer blocked
   │
   ▼
Upstream blocked
   │
   ▼
Request processing degraded
```

Zato buffered channel ne uklanja potrebu za lifecycle management-om.

Ako consumer može prestati sa radom, producer mora imati način da sazna da više nema smisla slati.

Čest obrazac je:

```go
select {
case jobs <- job:
    // job accepted
case <-ctx.Done():
    // stop producing
}
```

Time producer nije beskonačno vezan za channel send.

---

### Pitanje 34

**Zašto je `select` važan kada channel send može blokirati?**

### Odgovor

Zato što omogućava da goroutine ne čeka samo jednu operaciju.

Umesto:

```go
jobs <- job
```

možemo imati:

```go
select {
case jobs <- job:
    // job sent
case <-ctx.Done():
    return
}
```

Sada postoje dva moguća ishoda:

```text
                ┌── jobs available ──> SEND
                │
select ─────────┤
                │
                └── cancellation ───> RETURN
```

Ovo je posebno važno kod dugotrajnih worker sistema.

Bez cancellation grane:

```text
producer
   │
   ▼
send(job)
   │
   ▼
BLOCKED FOREVER
```

Sa cancellation signalom:

```text
producer
   │
   ▼
select
 ┌─┴───────────────┐
 ▼                 ▼
send           ctx.Done()
 │                 │
 ▼                 ▼
continue          return
```

Time channel operacija dobija lifecycle escape path.

To je jedan od ključnih obrazaca production-grade Go concurrency koda.

---

### Pitanje 35

**Kako backpressure utiče na latency?**

### Odgovor

Backpressure može biti koristan za zaštitu sistema, ali može povećati latency.

Pretpostavimo:

```text
Producer
   │
   ▼
Queue
   │
   ▼
Worker
```

Ako queue počne da se puni, novi posao više ne može odmah da bude obrađen.

Vreme čekanja raste:

```text
latency =
    queue wait
  + processing time
```

Ako queue postane veoma dugačak:

```text
job 1 → █
job 2 → ██
job 3 → ███
...
job N → █████████████████
```

poslednji posao može čekati veoma dugo.

Zato veliki queue nije nužno znak dobrog sistema.

U nekim sistemima je bolje odbiti posao:

```text
queue full
    │
    ▼
reject
```

nego ga prihvatiti i obraditi mnogo sekundi ili minuta kasnije.

Ovo je posebno važno kod:

* real-time sistema,
* payment processing-a,
* betting sistema,
* trading sistema,
* notification sistema,
* HTTP API-ja.

Semantika sistema mora definisati šta je prihvatljiva latencija i šta treba uraditi kada sistem dostigne kapacitet.

---

### Pitanje 36

**Šta znači da sistem treba da bude "bounded"?**

### Odgovor

Bounded concurrency znači da sistem ima eksplicitno ograničene resurse ili količinu posla koji može istovremeno da obrađuje.

Na primer:

```go
jobs := make(chan Job, 100)
```

ima bounded queue.

Worker pool:

```text
            ┌── Worker 1
            ├── Worker 2
Jobs ───────┼── Worker 3
            ├── Worker 4
            └── Worker 5
```

ima bounded broj worker-a.

To je važno jer bez ograničenja možemo imati:

```go
go process(job)
```

za svaki incoming job.

Ako stigne milion poslova:

```text
1.000.000 jobs
       │
       ▼
1.000.000 goroutines
```

Go goroutine-i jesu relativno jeftini, ali "jeftino" nije isto što i "beskonačno".

Bounded concurrency omogućava sistemu da kontroliše:

* CPU,
* memoriju,
* broj konekcija,
* downstream pressure,
* queue depth,
* latency,
* ukupan broj aktivnih operacija.

---

### Pitanje 37

**Zašto bounded concurrency često daje stabilniji sistem od neograničenog kreiranja goroutine-a?**

### Odgovor

Zato što sistem može imati kontrolisan maksimum aktivnog rada.

Na primer:

```text
incoming requests
       │
       ▼
┌────────────────┐
│ bounded queue  │
└───────┬────────┘
        │
   ┌────┼────┐
   ▼    ▼    ▼
 worker worker worker
```

Ako imamo 10 worker-a:

```text
max active processing ≈ 10
```

U zavisnosti od arhitekture i dodatnih goroutine-a, ovo daje predvidljiviji resource envelope.

Nasuprot tome:

```text
request
   │
   ▼
go process()
```

može proizvesti proizvoljan broj aktivnih operacija.

To može dovesti do:

```text
incoming load ↑
     ↓
goroutines ↑
     ↓
memory ↑
     ↓
GC pressure ↑
     ↓
latency ↑
     ↓
timeouts ↑
     ↓
retries ↑
     ↓
load ↑
```

Ovo je klasičan oblik **feedback loop-a** koji može izazvati cascading failure.

Bounded concurrency zato nije samo performance optimization.

To je često **stability mechanism**.

---

## Senior-level zaključak

Kod concurrency sistema nije dovoljno pitati:

> "Da li možemo da obradimo ovaj posao?"

Potrebno je pitati:

> **"Šta se dešava kada posao stiže brže nego što sistem može da ga obradi?"**

Production-grade dizajn mora definisati ponašanje za:

```text
normal load
    ↓
burst
    ↓
queue growth
    ↓
queue full
    ↓
downstream slowdown
    ↓
timeout
    ↓
cancellation
    ↓
shutdown
```

Najvažniji koncepti koje treba povezati su:

```text
Buffered Channels
       │
       ▼
Bounded Queues
       │
       ▼
Backpressure
       │
       ▼
Bounded Concurrency
       │
       ▼
Resource Protection
       │
       ▼
System Stability
```

Ako sistem nema definisano ponašanje kada dostigne kapacitet, onda concurrency dizajn nije kompletan.

---

### Pitanje 38

**Koja je razlika između deadlock-a, livelock-a i starvation-a?**

### Odgovor

Sva tri problema predstavljaju različite oblike neuspeha concurrent sistema, ali njihova priroda je različita.

#### Deadlock

Kod **deadlock-a**, goroutine-i čekaju jedni druge i nijedna ne može da nastavi.

Na primer:

```text
Goroutine A
    │
    └── čeka B

Goroutine B
    │
    └── čeka A
```

Sistem je potpuno blokiran.

Tipičan primer sa mutex-ima:

```go
mu1.Lock()
defer mu1.Unlock()

mu2.Lock()
defer mu2.Unlock()
```

dok druga goroutine radi:

```go
mu2.Lock()
defer mu2.Unlock()

mu1.Lock()
defer mu1.Unlock()
```

Ako obe goroutine istovremeno uzmu prvi mutex:

```text
G1 → owns mu1 → waits for mu2
G2 → owns mu2 → waits for mu1
```

dobijamo circular wait.

---

#### Livelock

Kod **livelock-a**, goroutine-i nisu blokirani u klasičnom smislu.

One aktivno izvršavaju kod, ali sistem ne napreduje.

Na primer:

```text
G1 → pokušava
   → odustaje
   → pokušava ponovo

G2 → pokušava
   → odustaje
   → pokušava ponovo
```

Obe goroutine su aktivne, ali nijedna ne ostvaruje korisni progress.

Možemo ga zamisliti kao dve osobe koje pokušavaju da se mimoiđu:

```text
A → ← B

A ide levo
B ide levo

A ide desno
B ide desno

A ide levo
B ide levo
```

Sistem "radi", ali nema progress.

---

#### Starvation

Kod **starvation-a**, jedna goroutine ili grupa goroutine-a ne dobija dovoljno pristupa resursu zato što druge goroutine kontinuirano dobijaju prednost.

Na primer:

```text
G1 ─┐
G2 ─┤
G3 ─┤──> resource
G4 ─┘

G5 ───────> čeka veoma dugo
```

G5 možda nikada ne dobije dovoljno CPU vremena, lock ownership-a ili pristupa drugom resursu.

Dakle:

| Problem    | Goroutine aktivna? |  Progress? |
| ---------- | -----------------: | ---------: |
| Deadlock   |                 Ne |         Ne |
| Livelock   |                 Da |         Ne |
| Starvation |              Možda | Nedovoljan |

Senior developer mora umeti da razlikuje ova tri problema jer zahtevaju različite strategije rešavanja.

---

### Pitanje 39

**Kako može nastati deadlock korišćenjem kanala?**

### Odgovor

Najjednostavniji primer je slanje na unbuffered channel bez consumer-a:

```go
ch := make(chan int)

ch <- 42
```

Send čeka receiver-a.

Ali receiver ne postoji.

Program nema goroutine koja može da izvrši:

```go
<-ch
```

Zato nastaje deadlock.

Tipičan scenario:

```text
main goroutine
     │
     ▼
ch <- 42
     │
     ▼
BLOCKED
```

Go runtime može detektovati situaciju u kojoj su sve goroutine blokirane i prijaviti:

```text
fatal error: all goroutines are asleep - deadlock!
```

Važna stvar je da channel sam po sebi nije "deadlock-safe".

Channel samo definiše određeni communication mechanism.

Programer mora obezbediti validan lifecycle producer-a i consumer-a.

---

### Pitanje 40

**Kako buffered channel može sprečiti jedan deadlock, ali istovremeno sakriti drugi problem?**

### Odgovor

Posmatrajmo:

```go
ch := make(chan int, 1)

ch <- 42
```

Ovo neće blokirati jer channel ima kapacitet `1`.

Međutim:

```go
ch <- 42
ch <- 43
```

drugi send blokira jer je buffer pun.

Dakle:

```text
capacity = 1

send 42 → OK

buffer:
[42]

send 43 → BLOCK
```

Buffer može odložiti blokiranje, ali ne mora ukloniti njegov uzrok.

To je veoma važno prilikom debugging-a.

Na primer, program može raditi bez problema sa:

```go
make(chan Job, 100)
```

tokom testiranja.

Ali pod production load-om:

```text
100 jobs
101st job
   ↓
BLOCK
```

Problem postaje vidljiv tek kada sistem dostigne određeni nivo opterećenja.

Zato buffered channel ne treba posmatrati kao "lek protiv deadlock-a".

On samo menja tačku na kojoj se komunikacija blokira.

---

### Pitanje 41

**Šta je circular wait i zašto je važan za deadlock?**

### Odgovor

**Circular wait** znači da postoji ciklus zavisnosti između goroutine-a i resursa.

Na primer:

```text
G1 → čeka R2
R2 → pripada G2

G2 → čeka R1
R1 → pripada G1
```

Graf izgleda:

```text
G1
 │
 ▼
R2
 │
 ▼
G2
 │
 ▼
R1
 │
 └──────> G1
```

Postoji ciklus.

Kod mutex-a:

```text
G1:
Lock(A)
Lock(B)

G2:
Lock(B)
Lock(A)
```

Ako se izvršavanje interleaving-uje ovako:

```text
G1: Lock(A)  ✓
G2: Lock(B)  ✓
G1: Lock(B)  BLOCK
G2: Lock(A)  BLOCK
```

nijedna goroutine ne može da nastavi.

Jedan od standardnih načina prevencije je **globalno definisan ordering resursa**.

Na primer:

```text
uvek prvo A
zatim B
zatim C
```

Tada sve goroutine moraju poštovati:

```go
Lock(A)
Lock(B)
```

a nikada:

```go
Lock(B)
Lock(A)
```

Time se uklanja mogućnost određenih circular-wait scenarija.

---

### Pitanje 42

**Kako možeš sprečiti deadlock kada više goroutine-a zaključava više mutex-a?**

### Odgovor

Jedna od najvažnijih tehnika je **consistent lock ordering**.

Pretpostavimo:

```go
type Account struct {
    mu      sync.Mutex
    balance int
}
```

i transfer između dva account-a.

Loš dizajn može dozvoliti:

```text
Transfer A → B

Lock(A)
Lock(B)
```

dok druga goroutine radi:

```text
Transfer B → A

Lock(B)
Lock(A)
```

To može proizvesti:

```text
G1: owns A → waits B
G2: owns B → waits A
```

Bolji pristup je definisati deterministički ordering.

Na primer:

```text
lock account sa manjim ID-em
zatim account sa većim ID-em
```

Dakle, čak i kada transfer ide:

```text
A → B
```

ili:

```text
B → A
```

lock ordering ostaje:

```text
min(ID) → max(ID)
```

Time se izbegava circular dependency.

Ovo je tipičan primer gde senior developer ne rešava samo konkretan deadlock, već uvodi **invariant koji sprečava čitavu klasu deadlock-a**.

---

### Pitanje 43

**Šta je starvation u Go concurrency sistemu?**

### Odgovor

Starvation nastaje kada goroutine praktično ne dobija priliku da napreduje zato što druge goroutine kontinuirano koriste resurs.

Resurs može biti:

* mutex,
* CPU,
* channel,
* worker slot,
* connection pool,
* queue capacity,
* drugi shared resource.

Na primer:

```text
         ┌── G1 ──┐
         ├── G2 ──┤
Resource ┼── G3 ──┤
         ├── G4 ──┤
         └── G5 ──┘

G6 → čeka
```

Ako G1–G5 kontinuirano monopolizuju resurs, G6 može imati veoma loš progress.

Važno je razlikovati starvation od deadlock-a.

Kod deadlock-a imamo:

```text
nema progress-a jer postoji ciklus zavisnosti
```

Kod starvation-a:

```text
neke goroutine napreduju,
ali druge praktično ne napreduju.
```

Dakle, sistem kao celina može izgledati "živ", dok pojedinačne operacije imaju neprihvatljivu latenciju.

To je naročito opasno u sistemima sa SLA/SLO zahtevima.

---

### Pitanje 44

**Kako livelock može nastati u retry mehanizmu?**

### Odgovor

Pretpostavimo dve goroutine koje pokušavaju da izvrše operaciju.

Obe koriste logiku:

```text
pokušaj
ako ne možeš:
    oslobodi resurs
    retry
```

Ako obe goroutine rade potpuno istu stvar u istom trenutku:

```text
G1: try → fail → retry
G2: try → fail → retry

G1: try → fail → retry
G2: try → fail → retry
```

sistem je aktivan, ali nema progress-a.

To je livelock.

Jedna od strategija je uvođenje:

* random jitter-a,
* exponential backoff-a,
* prioriteta,
* drugačije scheduling politike.

Na primer:

```text
retry delay =
    exponential backoff
    +
    random jitter
```

Jitter je važan jer sprečava da veliki broj goroutine-a istovremeno izvršava identičan retry pattern.

Bez jitter-a:

```text
G1 ───── retry ───── retry ─────
G2 ───── retry ───── retry ─────
G3 ───── retry ───── retry ─────
```

Sa jitter-om:

```text
G1 ─── retry ───────── retry ───

G2 ────── retry ─── retry ──────

G3 ── retry ───────────── retry ─
```

Time se smanjuje verovatnoća sinhronizovanog konflikta.

---

### Pitanje 45

**Šta je data race i kako se razlikuje od deadlock-a?**

### Odgovor

**Data race** nastaje kada dve ili više goroutine-a pristupaju istoj memorijskoj lokaciji konkurentno, pri čemu je najmanje jedan pristup write, a pristupi nisu pravilno sinhronizovani.

Primer:

```go
var counter int

go func() {
    counter++
}()

go func() {
    counter++
}()
```

Obe goroutine menjaju:

```text
counter
```

bez odgovarajuće sinhronizacije.

To je data race.

Deadlock je potpuno drugačiji problem.

Kod deadlock-a:

```text
goroutine → čeka
goroutine → čeka
```

Kod data race-a:

```text
goroutine → pristupa memoriji
goroutine → istovremeno pristupa istoj memoriji
```

Dakle:

| Problem    | Osnovni problem                               |
| ---------- | --------------------------------------------- |
| Data race  | Neispravan konkurentni pristup memoriji       |
| Deadlock   | Cikličko čekanje                              |
| Livelock   | Aktivnost bez progress-a                      |
| Starvation | Jedna goroutine ne dobija dovoljno progress-a |

Ova četiri problema ne treba mešati.

Jedan concurrent sistem može istovremeno imati više njih.

---

### Pitanje 46

**Kako bi dijagnostikovao da li je problem deadlock ili data race?**

### Odgovor

Prvo posmatram simptom.

Ako program:

```text
visi
ne napreduje
CPU je nizak
goroutine-i čekaju
```

deadlock je jedan od prvih kandidata.

Ako program:

```text
ponekad daje različite rezultate
ponašanje zavisi od timing-a
testovi povremeno padaju
```

data race je ozbiljan kandidat.

Za race detection u Go-u koristi se race detector:

```bash
go test -race ./...
```

ili, zavisno od aplikacije:

```bash
go run -race .
```

Za deadlock je često korisno analizirati goroutine dump.

Na primer:

```text
goroutine 12 [chan send]:
...
goroutine 13 [chan receive]:
...
```

ili:

```text
goroutine 7 [sync.Mutex.Lock]:
...
```

Takav stack može pokazati gde goroutine trenutno čeka.

Senior developer zato ne zaključuje:

> "Program visi, sigurno je race."

Naprotiv, prvo klasifikuje simptom, zatim prikuplja runtime evidence i tek onda određuje uzrok.

---

## Senior-level princip

Kod concurrency bug-a najvažnije je razumeti **progress**.

Za svaku goroutine treba moći odgovoriti:

1. Koji posao ona obavlja?
2. Na čemu može da blokira?
3. Ko je oslobađa?
4. Koji događaj omogućava njen nastavak?
5. Kako može da se prekine?
6. Šta se dešava ako druga strana nikada ne odgovori?
7. Kako se garantuje da sistem napreduje?

Dobar concurrency dizajn ne opisuje samo:

```text
"goroutine šalje podatke"
```

nego i:

```text
ko šalje
ko prima
koliko dugo može da čeka
šta ako nema receiver-a
šta ako nema više producer-a
šta ako consumer umre
šta ako queue bude pun
šta ako dođe cancellation
šta ako dođe shutdown
```

Upravo ta pitanja razlikuju jednostavan concurrency kod od production-grade concurrency arhitekture.

---

### Pitanje 47

**Zašto je lifecycle goroutine-a važan deo concurrency dizajna?**

### Odgovor

Goroutine nije samo funkcija koja se pokrene pomoću `go` ključne reči.

U production sistemu svaka goroutine ima određeni **lifecycle**:

```text
START
  │
  ▼
RUNNING
  │
  ├── BLOCKED
  │     │
  │     └── RUNNING
  │
  ▼
STOPPING
  │
  ▼
TERMINATED
```

Senior developer mora znati:

* ko pokreće goroutine,
* šta goroutine radi,
* koliko dugo treba da postoji,
* šta je završni uslov,
* ko signalizira shutdown,
* da li goroutine može biti otkazana,
* šta se dešava ako zavisnost nikada ne odgovori,
* kako se proverava da je goroutine zaista završila.

Loš concurrency dizajn često izgleda ovako:

```go
go worker()
```

i tu se lifecycle završava.

Nema:

* cancellation mehanizma,
* ownership-a,
* shutdown procedure,
* error propagation-a,
* čekanja na završetak.

To može dovesti do **goroutine leak-a**.

---

### Pitanje 48

**Šta je goroutine leak i kako nastaje?**

### Odgovor

Goroutine leak nastaje kada goroutine ostane aktivna ili blokirana duže nego što je predviđeno, a više ne postoji realna potreba da ona nastavi da postoji.

Jedan od najčešćih primera:

```go
func worker(ch <-chan int) {
    for {
        value := <-ch
        process(value)
    }
}
```

Ako producer prestane da šalje podatke, worker može ostati blokiran zauvek.

Još problematičniji primer je kada goroutine čeka na channel koji nikada neće biti zatvoren:

```text
worker
  │
  ▼
<-ch
  │
  ▼
BLOCKED FOREVER
```

Ako se takve goroutine pokreću često:

```text
request 1 → goroutine leak
request 2 → goroutine leak
request 3 → goroutine leak
...
request N → goroutine leak
```

broj goroutine-a može kontinuirano rasti.

Posledice mogu biti:

* povećana memorijska potrošnja,
* povećan scheduler overhead,
* nepotrebno zadržavanje objekata,
* zadržavanje channel-a i drugih referenci,
* degradacija performansi,
* eventualno iscrpljivanje resursa.

Zato senior concurrency dizajn mora imati eksplicitno definisan **termination path**.

---

### Pitanje 49

**Kako `context.Context` pomaže u lifecycle management-u goroutine-a?**

### Odgovor

`context.Context` omogućava da komponenta dobije signal da treba da prekine svoj rad.

Tipičan obrazac:

```go
func worker(ctx context.Context, jobs <-chan Job) {
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

Sada worker ima dva moguća događaja:

```text
             ┌── job received ──> process
             │
worker ──────┤
             │
             └── ctx.Done() ────> return
```

To je mnogo robusnije od:

```go
for {
    job := <-jobs
}
```

jer postoji eksplicitna mogućnost prekida.

`context.Context` je posebno koristan kada lifecycle goroutine-a treba povezati sa:

* HTTP request-om,
* RPC pozivom,
* timeout-om,
* cancellation-om,
* shutdown-om servisa,
* parent goroutine-om ili višom komponentom sistema.

Važno je razumeti da context sam po sebi ne "ubija" goroutine.

On šalje **signal za cancellation**.

Goroutine mora taj signal da poštuje.

---

### Pitanje 50

**Šta se dešava ako goroutine ignoriše cancellation signal?**

### Odgovor

Ako goroutine ne proverava cancellation, poziv:

```go
cancel()
```

ne garantuje da će goroutine završiti.

Na primer:

```go
func worker(ctx context.Context) {
    for {
        doWork()
    }
}
```

Čak i ako postoji:

```go
ctx, cancel := context.WithCancel(context.Background())

cancel()
```

worker nema nikakav kod koji reaguje na:

```go
ctx.Done()
```

Zato može nastaviti da radi.

Ispravan dizajn zahteva **cooperative cancellation**:

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

Međutim, čak i ovo može biti problematično ako je:

```go
doWork()
```

dugotrajan ili blokirajući poziv.

Zato cancellation mora biti propagiran i kroz niže slojeve kada je moguće.

Na primer:

```go
worker
  ↓
service
  ↓
repository
  ↓
HTTP client / DB
```

Ako je moguće, context treba proslediti kroz ceo pozivni lanac:

```go
service.Do(ctx)
repository.Find(ctx)
client.Do(ctx)
```

Time cancellation može da propagira od spoljnog događaja do stvarnog blocking operation-a.

---

### Pitanje 51

**Kako bi dizajnirao graceful shutdown za worker pool?**

### Odgovor

Pretpostavimo worker pool:

```text
             ┌── Worker 1
Jobs ────────┼── Worker 2
             ├── Worker 3
             └── Worker 4
```

Graceful shutdown znači da sistem ne prekida rad proizvoljno, već kontroliše završetak.

Jedan mogući lifecycle je:

```text
RUNNING
   │
   │ shutdown signal
   ▼
STOP ACCEPTING NEW WORK
   │
   ▼
FINISH / CANCEL CURRENT WORK
   │
   ▼
WAIT FOR WORKERS
   │
   ▼
TERMINATED
```

Na primer:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

var wg sync.WaitGroup

for i := 0; i < 4; i++ {
    wg.Add(1)

    go func() {
        defer wg.Done()
        worker(ctx, jobs)
    }()
}
```

Kada dođe shutdown:

```go
cancel()
wg.Wait()
```

Ali pravi dizajn zavisi od semantike sistema.

Ako posao mora biti završen:

```text
shutdown
    ↓
stop accepting new jobs
    ↓
finish queued jobs
    ↓
workers exit
```

Ako posao može biti prekinut:

```text
shutdown
    ↓
cancel context
    ↓
workers stop
```

Senior developer mora definisati ovu semantiku pre implementacije.

---

### Pitanje 52

**Zašto je `close(channel)` deo lifecycle dizajna, a ne samo tehnička operacija?**

### Odgovor

Zatvaranje channel-a predstavlja važnu informaciju:

> "Ovaj communication stream više neće imati novih vrednosti."

Na primer:

```go
for value := range ch {
    process(value)
}
```

petlja završava kada channel bude zatvoren i kada se potroše sve preostale vrednosti.

Zato:

```text
producer
   │
   │ send values
   ▼
channel
   │
   │ close
   ▼
consumer
   │
   ▼
exit
```

`close` je posebno važan kada consumer treba da zna da više neće biti podataka.

Ali treba pažljivo definisati ownership.

Uobičajeno pravilo je:

> **sender koji poseduje stream najčešće je odgovoran za njegovo zatvaranje.**

Consumer ne bi trebalo proizvoljno da zatvara channel koji nije njegov za upravljanje.

Loš dizajn može dovesti do:

```text
consumer closes channel
producer sends
       ↓
panic: send on closed channel
```

Zato channel ownership treba biti deo API dizajna.

---

### Pitanje 53

**Da li `close(channel)` zaustavlja goroutine koja šalje podatke?**

### Odgovor

Ne.

`close` menja stanje channel-a.

Ne postoji automatska mehanika koja kaže:

```text
close(ch)
   ↓
kill producer goroutine
```

Ako producer nastavi:

```go
ch <- value
```

nakon zatvaranja channel-a, dobiće:

```text
panic: send on closed channel
```

Zato lifecycle mora biti koordinisan.

Na primer:

```text
producer
   │
   ├── proizvodi
   │
   ├── dobija cancellation
   │
   ▼
prestaje sa radom
   │
   ▼
close(ch)
```

Dakle, `close` je deo protocol-a, a ne mehanizam za ubijanje goroutine-a.

---

### Pitanje 54

**Ko treba da bude vlasnik goroutine-a u dobro dizajniranom sistemu?**

### Odgovor

Treba jasno definisati **ownership**.

Ako komponenta pokrene goroutine:

```go
func StartWorker() {
    go worker()
}
```

postavlja se pitanje:

> Ko je odgovoran za njen završetak?

Ako odgovor nije jasan, postoji rizik od lifecycle problema.

Bolji API može eksplicitno predstaviti lifecycle:

```go
worker := NewWorker(...)
worker.Start()

// ...

worker.Stop()
worker.Wait()
```

ili:

```go
ctx, cancel := context.WithCancel(...)
defer cancel()

go worker.Run(ctx)
```

U oba slučaja postoji jasniji odnos:

```text
owner
  │
  ├── starts goroutine
  │
  ├── controls lifecycle
  │
  └── waits for termination
```

Ovo je veoma važan senior-level koncept.

**Ako pokrećeš goroutine, moraš znati ko je odgovoran za njen završetak.**

---

### Pitanje 55

**Kako bi otkrio goroutine leak u production sistemu?**

### Odgovor

Prvo bih posmatrao broj goroutine-a kroz vreme.

Ako sistem stabilno radi:

```text
100
102
101
103
100
```

to može biti normalno.

Ali ako imamo:

```text
100
500
1,000
5,000
10,000
```

bez proporcionalnog povećanja legitimnog workload-a, postoji ozbiljna sumnja na leak.

U Go-u se broj goroutine-a može pratiti preko:

```go
runtime.NumGoroutine()
```

Za detaljniju analizu mogu se koristiti runtime profileri i goroutine dump-ovi.

Ključna pitanja su:

1. Koje goroutine postoje?
2. Na čemu čekaju?
3. Koliko dugo čekaju?
4. Ko ih je pokrenuo?
5. Koji događaj treba da ih završi?
6. Zašto taj događaj nije nastupio?

Na primer, ako veliki broj goroutine-a čeka:

```text
<-jobs
```

treba proveriti:

* da li producer i dalje postoji,
* da li je channel ikada zatvoren,
* da li postoji cancellation,
* da li je worker lifecycle pravilno definisan.

Senior debugging nije samo:

> "Imamo mnogo goroutine-a."

nego:

> "Koji lifecycle invariant je prekršen i zašto te goroutine više nemaju validan termination path?"

---

## Senior-level princip

Concurrency sistem treba posmatrati kao skup **lifecycle protokola**.

Za svaku goroutine i svaki channel treba moći definisati:

```text
Who starts it?
Who owns it?
Who feeds it?
Who stops it?
Who closes it?
Who waits for it?
What happens on cancellation?
What happens on error?
What happens on timeout?
```

Ako na neka od ovih pitanja nema jasnog odgovora, concurrency dizajn verovatno još nije dovoljno precizan za production okruženje.

Posebno je važno izbeći obrazac:

```go
go something()
```

bez jasnog odgovora na pitanje:

```text
"Kako i kada ova goroutine završava?"
```

To pitanje je jedan od najvažnijih kriterijuma za procenu zrelosti concurrent Go koda.

---

### Pitanje 56

**Šta podrazumeva channel ownership u Go concurrency dizajnu?**

### Odgovor

Channel ownership predstavlja jasno definisanje odgovornosti nad životnim ciklusom channel-a.

Kod svakog channel-a treba znati:

* ko ga kreira,
* ko šalje podatke,
* ko ih prima,
* ko ga zatvara,
* kada se zatvara,
* šta znači njegovo zatvaranje,
* ko je odgovoran za koordinaciju producer-a i consumer-a.

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

Ovde je ownership relativno jasan:

```text
producer
   │
   ├── kreira channel
   ├── šalje podatke
   └── zatvara channel

consumer
   │
   └── prima podatke
```

Consumer ne mora da zna kako producer implementira channel.

On samo zna:

```go
for value := range producer() {
    process(value)
}
```

Ovakav dizajn smanjuje mogućnost pogrešne upotrebe.

---

### Pitanje 57

**Zašto je `chan<- T` često koristan za API dizajn?**

### Odgovor

Directional channel omogućava API-ju da eksplicitno izrazi nameru.

Na primer:

```go
func produce(out chan<- int) {
    out <- 42
}
```

Funkcija može samo da šalje.

Ne može da radi:

```go
value := <-out
```

Isto važi za:

```go
func consume(in <-chan int) {
    value := <-in
    process(value)
}
```

Consumer može samo da prima.

Ovo je više od sintaksne pogodnosti.

Directional channel predstavlja **compile-time constraint**.

Umesto:

```go
func process(ch chan int)
```

API može izraziti:

```go
func process(in <-chan int)
```

što dokumentuje:

> Ova komponenta je consumer.

I:

```go
func process(out chan<- int)
```

što dokumentuje:

> Ova komponenta je producer.

Time se smanjuje prostor za grešku i jasnije se definiše concurrency boundary.

---

### Pitanje 58

**Kako bi dizajnirao funkciju koja vraća read-only channel?**

### Odgovor

Tipičan obrazac je:

```go
func generate() <-chan int {
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

Spoljašnji caller dobija:

```go
ch := generate()
```

ali njegov tip je:

```go
<-chan int
```

Zato može:

```go
value := <-ch
```

ali ne može:

```go
ch <- 42
```

niti:

```go
close(ch)
```

To je veoma koristan oblik enkapsulacije.

Interni implementation poseduje puni channel:

```text
inside:
chan int

outside:
<-chan int
```

API time skriva operacije koje caller ne treba da izvršava.

---

### Pitanje 59

**Šta je concurrency boundary?**

### Odgovor

Concurrency boundary predstavlja granicu između komponenata na kojoj se definiše način konkurentne komunikacije i koordinacije.

Na primer:

```text
HTTP Handler
     │
     │
     ▼
Service
     │
     │ channel
     ▼
Worker Pool
```

Granica između Service-a i Worker Pool-a može biti:

```go
jobs chan<- Job
```

Service proizvodi poslove:

```go
jobs <- job
```

Worker pool ih konzumira:

```go
job := <-jobs
```

Dobro definisana granica treba jasno da definiše:

* ownership,
* lifecycle,
* blocking behavior,
* cancellation,
* backpressure,
* error propagation,
* shutdown semantics.

Loš concurrency boundary može izgledati ovako:

```go
func Process(
    ctx context.Context,
    jobs chan Job,
    results chan Result,
    workers *sync.WaitGroup,
)
```

Caller sada mora razumeti previše internih detalja.

Bolji API može sakriti implementaciju:

```go
func (p *Processor) Process(ctx context.Context, input Input) (Output, error)
```

Interno processor može koristiti:

* goroutine-e,
* channels,
* worker pool,
* mutex,
* atomic operations.

Ali caller ne mora znati kako je concurrency implementiran.

To je ključna ideja:

> Concurrency implementation treba biti enkapsulirana kada calleru nije potrebna direktna kontrola.

---

### Pitanje 60

**Kada channel nije dobar izbor za komunikaciju između komponenti?**

### Odgovor

Channel nije automatski najbolji concurrency primitive.

Ako je problem jednostavno zaštita shared state-a:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}
```

mutex može biti jednostavniji od channel-based actor modela.

Na primer, nema potrebe uvoditi:

```text
goroutine
    ↓
channel
    ↓
counter owner goroutine
    ↓
result channel
```

ako je jedina operacija:

```go
counter++
```

Channel može biti opravdan kada postoji:

* stream podataka,
* producer/consumer odnos,
* pipeline,
* ownership transfer,
* event processing,
* fan-in/fan-out,
* asynchronous work queue.

Ali ako channel samo komplikuje jednostavan shared-state problem, možda nije pravi alat.

Senior developer bira primitive prema problemu, a ne prema popularnosti primitive-a.

---

### Pitanje 61

**Kako biraš između channel-a i mutex-a?**

### Odgovor

Jedna korisna mentalna podela je:

### Channel

Koristiš kada želiš da modeluješ:

```text
communication
```

odnosno:

> Jedna goroutine proizvodi podatke, druga ih konzumira.

Na primer:

```text
Producer → Channel → Consumer
```

### Mutex

Koristiš kada želiš da modeluješ:

```text
shared state protection
```

odnosno:

> Više goroutine-a pristupa istom stanju, ali pristup mora biti koordinisan.

Na primer:

```text
G1 ─┐
G2 ─┼──> Mutex ──> Shared State
G3 ─┘
```

Nije apsolutno pravilo.

Channel može zaštititi ownership nad state-om, a mutex može biti deo kompleksnog communication sistema.

Ali kao početna heuristika:

```text
communication → channel
shared mutable state → mutex
```

je veoma korisna.

---

### Pitanje 62

**Zašto "share memory by communicating" nije pravilo da treba koristiti channel za sve?**

### Odgovor

Go-ova filozofija često se sažima kroz ideju:

> "Do not communicate by sharing memory; share memory by communicating."

Međutim, ovo nije zabrana korišćenja mutex-a ili shared memory modela.

Go standardna biblioteka direktno pruža:

```go
sync.Mutex
sync.RWMutex
sync.Once
sync.Cond
sync/atomic
```

Zato je pogrešno zaključiti:

```text
channel = good
mutex = bad
```

Pravi cilj je odabrati primitive koji najbolje predstavlja problem.

Na primer:

```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]Value
}
```

može biti mnogo jednostavniji i efikasniji dizajn od pokušaja da se ceo cache pretvori u channel-driven actor.

S druge strane, za pipeline:

```text
input
  ↓
parse
  ↓
validate
  ↓
transform
  ↓
output
```

channels mogu biti prirodniji model.

Senior developer treba da razume **trade-off**, a ne da prati slogan mehanički.

---

### Pitanje 63

**Šta znači da concurrency API treba da definiše blocking semantics?**

### Odgovor

Ako funkcija može da blokira, caller treba da zna pod kojim uslovima.

Na primer:

```go
func Submit(job Job) error
```

Ako interno radi:

```go
jobs <- job
```

onda `Submit` može blokirati kada je queue pun.

Caller mora znati:

```text
Da li Submit:
- blokira?
- ima timeout?
- vraća odmah?
- vraća ErrQueueFull?
- poštuje context?
```

API može biti mnogo eksplicitniji:

```go
func Submit(ctx context.Context, job Job) error
```

Sada postoji mogućnost:

```go
select {
case jobs <- job:
    return nil

case <-ctx.Done():
    return ctx.Err()
}
```

Semantika je:

```text
job accepted
      ili
context cancelled
```

Bez ovakvog ugovora caller može slučajno napraviti:

```text
HTTP request
    ↓
Submit()
    ↓
blocks
    ↓
worker unavailable
    ↓
request hangs
```

Zato blocking behavior nije implementation detail.

On je deo API contract-a.

---

### Pitanje 64

**Kako channel capacity utiče na API semantiku?**

### Odgovor

Kapacitet channel-a određuje koliko poruka može biti privremeno zadržano bez consumer-a.

Za:

```go
make(chan Job)
```

kapacitet je:

```text
0
```

Producer i consumer moraju se sinhronizovati direktno.

Za:

```go
make(chan Job, 100)
```

producer može privremeno biti ispred consumer-a do određenog nivoa.

To uvodi **buffering**.

Ali capacity nije samo performance parameter.

On može menjati sistemsku semantiku.

Na primer:

```text
capacity = 0
```

znači:

```text
producer mora da ima consumer-a
```

dok:

```text
capacity = 1000
```

omogućava:

```text
producer može kratkoročno nadmašiti consumer
```

Ali kada buffer postane pun:

```text
producer
   │
   ▼
[1000 jobs]
   │
   ▼
BLOCK
```

Zato capacity predstavlja deo **backpressure modela**.

Ako je queue prevelik, sistem može akumulirati ogromnu količinu posla.

Ako je premali, producer može previše često blokirati.

Senior developer zato ne bira:

```go
make(chan Job, 100)
```

nasumično.

Kapacitet treba povezati sa:

* expected throughput-om,
* consumer capacity-jem,
* latency zahtevima,
* memory budget-om,
* burst behavior-om,
* backpressure strategijom.

---

### Pitanje 65

**Zašto je channel-based API često bolji kada se modeluje stream, a ne pojedinačna operacija?**

### Odgovor

Channel prirodno predstavlja tok vrednosti:

```text
v1 → v2 → v3 → v4 → v5
```

Na primer:

```go
func generate() <-chan Event
```

consumer može da radi:

```go
for event := range events {
    process(event)
}
```

Ovo je prirodan stream abstraction.

Međutim, ako funkcija vraća samo jednu vrednost:

```go
func GetUser(ctx context.Context, id string) (User, error)
```

uvođenje channel-a:

```go
func GetUser(ctx context.Context, id string) <-chan User
```

može nepotrebno zakomplikovati API.

Sada caller mora da razume:

* channel lifecycle,
* close,
* blocking,
* cancellation,
* goroutine ownership,
* moguće leak-ove.

Za single-result operation često je jednostavnije koristiti:

```go
(value, error)
```

ili:

```go
(value, error)
```

uz `context.Context`.

Channel treba uvoditi kada njegova semantika zaista dodaje vrednost.

---

## Senior-level princip

Dobar concurrency API ne izlaže samo podatke.

On izlaže i **concurrency contract**.

Za svaki concurrency-related API treba razmotriti:

```text
Ownership
Blocking
Cancellation
Timeout
Capacity
Backpressure
Shutdown
Error propagation
Ordering
Lifecycle
```

Na primer, API:

```go
func Submit(ctx context.Context, job Job) error
```

može implicitno imati mnogo precizniji ugovor od:

```go
func Submit(job Job)
```

jer prvi omogućava jasno definisanje šta se dešava kada:

* queue nema kapacitet,
* caller otkaže operaciju,
* worker pool nije dostupan,
* shutdown je u toku.

Na senior nivou channel više nije samo:

```text
"cevast način za slanje vrednosti između goroutine-a."
```

On postaje deo **API contract-a i arhitekture sistema**.

---

### Pitanje 66

**Šta je backpressure i zašto je važan u concurrent sistemima?**

### Odgovor

**Backpressure** je mehanizam kojim sporiji deo sistema signalizira bržem delu sistema da mora usporiti ili prestati sa prihvatanjem novog rada.

Pretpostavimo:

```text
Producer
   │
   │ 10.000 jobs/s
   ▼
Queue
   │
   │ 1.000 jobs/s
   ▼
Workers
```

Ako producer konstantno proizvodi više posla nego što workers mogu da obrade, sistem mora imati odgovor na pitanje:

> Šta se dešava sa viškom posla?

Bez backpressure-a, queue može neograničeno rasti:

```text
100
1.000
10.000
100.000
1.000.000
...
```

što može dovesti do:

* velike potrošnje memorije,
* povećanja latency-ja,
* GC pressure-a,
* OOM situacije,
* cascading failure-a.

Backpressure omogućava sistemu da kontroliše brzinu prihvatanja posla.

---

### Pitanje 67

**Kako buffered channel može implementirati jednostavan oblik backpressure-a?**

### Odgovor

Pretpostavimo:

```go
jobs := make(chan Job, 100)
```

Producer može da šalje dok buffer ne postane pun:

```text
Producer
   │
   ▼
┌──────────────┐
│ 100 jobs     │
└──────────────┘
   │
   ▼
Workers
```

Kada je buffer pun:

```go
jobs <- job
```

blokira dok worker ne oslobodi mesto.

To automatski usporava producer-a.

Model postaje:

```text
worker capacity
      ↓
channel capacity
      ↓
producer blocking
```

To je jednostavan oblik backpressure-a.

Međutim, senior developer mora razumeti da buffered channel nije kompletan backpressure strategy.

U zavisnosti od sistema možda je potrebno:

* odbacivanje posla,
* timeout,
* retry,
* rate limiting,
* queue sa persistent storage-om,
* load shedding,
* prioritetizacija posla.

---

### Pitanje 68

**Kako bi implementirao bounded concurrency?**

### Odgovor

Bounded concurrency znači da ograničimo broj istovremeno aktivnih operacija.

Na primer, želimo maksimalno 10 concurrent jobs:

```go
sem := make(chan struct{}, 10)
```

Pre pokretanja posla:

```go
sem <- struct{}{}
```

nakon završetka:

```go
<-sem
```

Model:

```text
             ┌── Job
             ├── Job
             ├── Job
Semaphore ───┼── Job
(cap = 10)   ├── ...
             └── Job
```

Najviše 10 goroutine-a može istovremeno zauzeti slotove.

Ali postoji i problematičan obrazac:

```go
for _, job := range jobs {
    sem <- struct{}{}

    go func() {
        defer func() {
            <-sem
        }()

        process(job)
    }()
}
```

Iako je broj aktivnih poslova ograničen, broj kreiranih goroutine-a može biti veoma veliki ako je `jobs` ogroman.

Bolji dizajn može kombinovati bounded concurrency sa eksplicitnim worker pool-om.

---

### Pitanje 69

**Koja je razlika između worker pool-a i semaphore pattern-a?**

### Odgovor

Oba mogu ograničiti concurrency, ali modeluju različite stvari.

### Semaphore

Semaphore prvenstveno odgovara na pitanje:

> Koliko operacija sme istovremeno da bude aktivno?

Na primer:

```text
max concurrency = 10
```

### Worker pool

Worker pool odgovara na pitanje:

> Koje goroutine-e kontinuirano obrađuju posao iz queue-a?

Model:

```text
             ┌── Worker 1
Jobs ────────┼── Worker 2
             ├── Worker 3
             └── Worker 4
```

Worker pool ima eksplicitne workers.

Semaphore često samo ograničava broj aktivnih operacija.

Na primer:

```text
10.000 jobs
     │
     ▼
10 concurrent operations
```

može biti implementirano semaphore-om bez 10.000 persistent worker goroutine-a.

Izbor zavisi od workload-a i lifecycle zahteva.

---

### Pitanje 70

**Zašto "spawn goroutine per request" može postati problematično?**

### Odgovor

Model:

```go
for {
    request := acceptRequest()

    go handle(request)
}
```

može biti potpuno validan ako je workload kontrolisan.

Problem nastaje kada broj request-a nije ograničen.

Ako sistem primi:

```text
100 requests
1.000 requests
10.000 requests
100.000 requests
```

može nastati:

```text
100.000 goroutines
```

Goroutine je relativno jeftina, ali nije besplatna.

Svaka goroutine zahteva runtime bookkeeping i početni stack prostor, a workload koji drži goroutine-e blokiranim može zadržavati i druge resurse.

Ako svaka goroutine dodatno drži:

* request data,
* buffers,
* channels,
* database resources,
* network connections,

ukupna potrošnja može brzo postati problem.

Zato production sistemi često uvode:

```text
bounded concurrency
+
backpressure
+
timeouts
+
cancellation
```

---

### Pitanje 71

**Kako bi dizajnirao concurrency limit za pristup eksternom servisu?**

### Odgovor

Pretpostavimo da servis poziva eksterni API:

```text
Go service
    │
    ├── request 1 ──┐
    ├── request 2 ──┤
    ├── request 3 ──┤
    ├── ...
    └── request N ──┘
             │
             ▼
       External API
```

Ako eksterni sistem može da obradi najviše 50 concurrent request-a, lokalni servis ne bi trebalo nekontrolisano da generiše stotine ili hiljade concurrent poziva.

Može se uvesti concurrency limit:

```go
sem := make(chan struct{}, 50)
```

uz cancellation-aware acquisition:

```go
select {
case sem <- struct{}{}:
    defer func() {
        <-sem
    }()

    return callExternalService(ctx)

case <-ctx.Done():
    return ctx.Err()
}
```

Ovim dobijamo:

```text
             max 50
               │
               ▼
Go Service ──────────> External Service
```

To štiti i lokalni sistem i downstream dependency.

Ali u realnom sistemu treba razmotriti i:

* rate limit,
* timeout,
* retries,
* exponential backoff,
* circuit breaker,
* connection pool,
* downstream quotas.

Concurrency limit sam po sebi ne rešava sve probleme.

---

### Pitanje 72

**Zašto retry u concurrent sistemu može pogoršati problem?**

### Odgovor

Pretpostavimo da downstream servis počne da bude spor.

Početni workload:

```text
100 requests
```

Ako svaki request ima 3 retry pokušaja, workload može postati:

```text
100 × 3 = 300 attempts
```

Ako retries krenu dok je downstream već preopterećen:

```text
load
  ↓
failure
  ↓
retry
  ↓
more load
  ↓
more failure
  ↓
more retry
```

nastaje **retry storm**.

To može pretvoriti lokalni problem u cascading failure.

Zato retry strategija mora biti concurrency-aware.

Tipične zaštite uključuju:

* bounded retries,
* exponential backoff,
* jitter,
* timeout,
* cancellation,
* concurrency limits,
* circuit breaker.

Senior developer mora posmatrati retry kao dodatni workload, a ne kao "besplatni pokušaj".

---

### Pitanje 73

**Kako timeout utiče na resource management?**

### Odgovor

Timeout omogućava da operacija ne ostane aktivna neograničeno.

Na primer:

```go
ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()
```

Ako downstream ne odgovori u roku od dve sekunde:

```text
request
   │
   ├── waiting
   │
   ├── waiting
   │
   └── timeout
         ↓
      cancellation
```

To je posebno važno za goroutine lifecycle.

Bez timeout-a možemo imati:

```text
request 1 → waiting
request 2 → waiting
request 3 → waiting
...
request N → waiting
```

Sa timeout-om:

```text
request
   ↓
bounded lifetime
   ↓
success OR cancellation
```

Ali timeout mora biti propagiran do stvarne blocking operacije.

Nije dovoljno imati timeout samo na najvišem sloju ako unutrašnji kod ignoriše context.

---

### Pitanje 74

**Šta je load shedding i kada bi ga koristio?**

### Odgovor

**Load shedding** znači namerno odbacivanje ili odbijanje dela workload-a kada sistem više nema dovoljno kapaciteta.

Pretpostavimo:

```text
Incoming traffic
       │
       ▼
┌───────────────┐
│ System limit  │
│ 1000 jobs     │
└───────────────┘
       │
       ├── accepted
       │
       └── rejected
```

Umesto da dozvolimo da latency beskonačno raste, možemo vratiti grešku:

```text
503 Service Unavailable
```

ili aplikacionu grešku poput:

```text
ErrOverloaded
```

Ovo je često zdravije od:

```text
accept everything
      ↓
queue everything
      ↓
memory exhaustion
      ↓
system crash
```

Load shedding je posebno relevantan u sistemima gde je bolje odbiti novi zahtev nego degradirati ceo servis.

---

### Pitanje 75

**Kako bi povezao backpressure, bounded concurrency i graceful shutdown u jednom sistemu?**

### Odgovor

Senior-level dizajn treba da poveže ove mehanizme.

Na primer:

```text
                Incoming Requests
                       │
                       ▼
                ┌─────────────┐
                │ Admission   │
                │ Control     │
                └─────────────┘
                       │
                 bounded queue
                       │
                       ▼
                ┌─────────────┐
                │ Worker Pool │
                │   N = 20    │
                └─────────────┘
                       │
                       ▼
                Downstream API
```

Pri normalnom radu:

```text
requests
   ↓
bounded queue
   ↓
20 workers
   ↓
downstream
```

Ako downstream uspori:

```text
workers slow
   ↓
queue fills
   ↓
backpressure
   ↓
producer blocks/rejects
```

Ako dođe shutdown:

```text
shutdown signal
       ↓
stop accepting new work
       ↓
cancel / drain according to policy
       ↓
workers terminate
       ↓
wait
       ↓
process exits
```

To je kompletan lifecycle model.

---

## Senior-level princip

Concurrent sistem mora imati kontrolu nad **brzinom**, **količinom** i **životnim vekom** posla.

Tri ključna pitanja su:

```text
How much work can enter?
How much work can run concurrently?
How long can work remain alive?
```

Na njih odgovaraju različiti mehanizmi:

```text
Backpressure
    ↓
kontroliše priliv posla

Bounded concurrency
    ↓
kontroliše broj aktivnih operacija

Timeout / cancellation
    ↓
kontroliše lifetime operacije

Graceful shutdown
    ↓
kontroliše završetak sistema
```

Kada se ovi mehanizmi kombinuju, concurrency sistem postaje predvidljiviji i otporniji na overload.

Posebno u production sistemima nije dovoljno pitati:

> "Da li kod radi kada je workload normalan?"

Senior pitanje je:

> "Šta se dešava kada workload postane 10× veći, downstream postane 10× sporiji ili jedan dependency prestane da odgovara?"

Tu se vidi kvalitet concurrency arhitekture.

---

### Pitanje 76

**Koje su najčešće failure modes u Go concurrent sistemima?**

### Odgovor

Senior developer treba da prepoznaje concurrency probleme ne samo na nivou sintakse, već kao sistemske failure modes.

Najčešći su:

```text
Goroutine leak
Data race
Deadlock
Livelock
Starvation
Unbounded concurrency
Unbounded queue growth
Blocked producer
Blocked consumer
Channel misuse
Premature channel close
Missing cancellation
Missing timeout
Retry storm
Cascading failure
```

Ovi problemi često nisu izolovani.

Na primer:

```text
Downstream slowdown
       ↓
Workers block
       ↓
Queue fills
       ↓
Producers block
       ↓
Requests accumulate
       ↓
Goroutines accumulate
       ↓
Memory pressure
       ↓
Latency increases
       ↓
Timeouts
       ↓
Retries
       ↓
More load
```

Zato concurrency failure treba analizirati kao **lanac događaja**, a ne samo kao pojedinačni bug.

---

### Pitanje 77

**Koja je razlika između deadlock-a, livelock-a i starvation-a?**

### Odgovor

### Deadlock

Goroutine-e čekaju jedna na drugu i nijedna ne može da nastavi.

Na primer:

```text
G1 → čeka G2
G2 → čeka G1
```

Sistem je blokiran.

### Livelock

Goroutine-e nisu blokirane, ali kontinuirano rade bez korisnog napretka.

Na primer, dve komponente konstantno menjaju stanje kako bi izbegle konflikt:

```text
G1 → pokušaj
G2 → pokušaj
G1 → povuci se
G2 → povuci se
G1 → pokušaj
G2 → pokušaj
...
```

CPU može biti aktivan, ali posao ne napreduje.

### Starvation

Jedna goroutine ili grupa goroutine-a ne dobija dovoljno pristupa resursu da bi napredovala.

Na primer:

```text
G1 ─┐
G2 ─┤
G3 ─┤── resource
G4 ─┤
G5 ─┘

G5 nikada ne dobija priliku.
```

Sva tri problema zahtevaju drugačiji debugging pristup.

---

### Pitanje 78

**Kako bi dijagnostikovao deadlock u Go programu?**

### Odgovor

Prvi korak je identifikovati goroutine-e koje su blokirane.

Tipičan simptom može biti:

```text
fatal error: all goroutines are asleep - deadlock!
```

Ali production deadlock ne mora uvek izgledati tako jednostavno.

Može postojati samo deo sistema koji je blokiran.

Treba analizirati:

* goroutine dump,
* stack trace-ove,
* channel send/receive operacije,
* mutex lock/unlock,
* `WaitGroup.Wait`,
* `select`,
* network I/O,
* database I/O.

Mentalni model treba da bude:

```text
Goroutine
   │
   ▼
šta čeka?
   │
   ▼
ko treba da proizvede signal?
   │
   ▼
da li taj producer može da nastavi?
```

Na primer:

```text
G1
 │
 └── waits on ch1

G2
 │
 └── waits on ch2

G3
 │
 └── waits on G1
```

Potrebno je rekonstruisati dependency graph.

Ako postoji ciklus:

```text
G1 → G2 → G3 → G1
```

postoji ozbiljna sumnja na deadlock.

---

### Pitanje 79

**Kako runtime i pprof mogu pomoći pri analizi goroutine problema?**

### Odgovor

Go runtime omogućava posmatranje stanja goroutine-a, a `pprof` pruža profiling infrastrukturu.

Posebno je koristan **goroutine profile**.

On može pokazati gde goroutine-e trenutno provode vreme ili na čemu čekaju.

Mentalni model:

```text
Application
     │
     ▼
Runtime
     │
     ▼
Goroutine profile
     │
     ├── blocked on channel
     ├── waiting on mutex
     ├── waiting on select
     ├── network I/O
     └── other states
```

Ako veliki broj goroutine-a ima isti stack trace:

```text
worker()
   ↓
select
   ↓
<-jobs
```

to može biti očekivano.

Ali ako imamo:

```text
10.000 goroutines
   ↓
same blocked stack
   ↓
same channel
```

to je jak signal da lifecycle ili shutdown protocol nije ispravan.

Važno je razlikovati:

> veliki broj goroutine-a

od:

> abnormalan broj goroutine-a koji imaju isti nevalidan lifecycle.

Broj sam po sebi nije dovoljan za zaključak.

---

### Pitanje 80

**Kako bi koristio race detector za concurrency probleme?**

### Odgovor

Go ima ugrađen race detector.

Tipično se koristi:

```bash
go test -race ./...
```

ili:

```bash
go run -race .
```

Race detector pokušava da otkrije određene slučajeve **data race-a** pri izvršavanju programa.

Data race nastaje kada:

1. više goroutine-a pristupa istoj memorijskoj lokaciji,
2. najmanje jedan pristup je write,
3. pristupi nisu pravilno sinhronizovani.

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

Ovo nema validnu sinhronizaciju.

Race detector može pomoći da se takav problem pronađe.

Međutim, važno je razumeti ograničenje:

> Race detector ne dokazuje da program nema race.

On pronalazi race-ove koji se manifestuju tokom konkretnog izvršavanja.

Zato test coverage i workload imaju veliki uticaj na njegovu efikasnost.

---

### Pitanje 81

**Da li program bez data race-a automatski ima ispravan concurrency dizajn?**

### Odgovor

Ne.

Ovo je veoma važna distinkcija.

Program može biti race-free, ali i dalje imati:

* deadlock,
* goroutine leak,
* starvation,
* livelock,
* pogrešan shutdown,
* pogrešnu ordering semantiku,
* unbounded queue,
* nekontrolisanu concurrency,
* pogrešnu cancellation logiku.

Na primer:

```go
mu.Lock()
defer mu.Unlock()

value++
```

može biti potpuno race-free.

Ali ako goroutine nikada ne može da dobije mutex:

```text
G1
 │
 └── holds lock forever

G2
 │
 └── waits forever
```

problem nije data race.

Zato:

```text
Race-free
    ≠
Concurrency-correct
```

Senior developer mora analizirati širu concurrency semantiku.

---

### Pitanje 82

**Kako bi instrumentirao concurrent sistem za production observability?**

### Odgovor

Concurrency problemi često zahtevaju observability signale koji omogućavaju da se vidi šta se dešava pod opterećenjem.

Korisne metrike uključuju:

```text
goroutine count
queue depth
active workers
job processing latency
job wait time
job success/failure count
timeouts
cancellations
retries
dropped jobs
rejected requests
downstream latency
```

Na primer:

```text
Queue depth
    │
    │        /\
    │       /  \
    │      /    \
    │_____/      \____
    └──────────────────> time
```

Ako queue depth kontinuirano raste, moguće je da:

```text
incoming rate > processing rate
```

To je mnogo korisnija informacija od samog broja goroutine-a.

Slično tome, ako:

```text
active workers = 100%
queue depth ↑
latency ↑
timeouts ↑
```

sistem verovatno radi na ili iznad svog kapaciteta.

Observability omogućava da concurrency problem posmatramo kao sistemski problem.

---

### Pitanje 83

**Zašto je queue depth važnija metrika od samog broja goroutine-a u worker pool-u?**

### Odgovor

Broj goroutine-a govori koliko concurrency jedinica postoji.

Queue depth govori koliko posla čeka.

Pretpostavimo:

```text
workers = 20
queue = 0
```

sistem može biti potpuno zdrav.

Ako imamo:

```text
workers = 20
queue = 10.000
```

očigledno postoji problem sa odnosom priliva i obrade.

Možemo posmatrati:

```text
arrival rate
      vs
processing rate
```

Ako je:

```text
arrival rate > processing rate
```

queue će rasti.

Ako je:

```text
arrival rate < processing rate
```

queue će se smanjivati ili ostajati prazan.

Zato je queue depth važan signal za:

* capacity planning,
* overload detection,
* autoscaling,
* backpressure,
* latency analysis.

---

### Pitanje 84

**Kako bi pronašao uzrok povećanja latency-ja u concurrent worker pool-u?**

### Odgovor

Ne bih odmah povećavao broj worker-a.

Prvo bih posmatrao:

```text
1. incoming rate
2. queue depth
3. worker utilization
4. job processing time
5. downstream latency
6. error rate
7. retry rate
8. timeout rate
9. resource utilization
```

Na primer:

```text
queue depth ↑
worker utilization = 100%
job duration ↑
downstream latency ↑
```

Uzrok možda nije nedovoljno worker-a.

Može biti downstream dependency.

Ako samo povećamo:

```text
workers: 20 → 100
```

možemo napraviti još veći pritisak na downstream:

```text
100 workers
      ↓
100 concurrent calls
      ↓
downstream overload
      ↓
more latency
      ↓
more timeout
      ↓
more retry
```

To može pogoršati problem.

Senior pristup je:

> Identifikuj bottleneck pre nego što povećaš concurrency.

---

### Pitanje 85

**Kako concurrency bug može izazvati cascading failure?**

### Odgovor

Zamislimo servis A koji poziva servis B.

Normalno:

```text
A
│
├── 10 concurrent requests
│
▼
B
```

Servis B uspori.

Sada A ima:

```text
10 requests waiting
```

Ako sistem nema bounded concurrency, request-i počinju da se akumuliraju:

```text
100
1.000
10.000
```

Istovremeno mogu nastati:

```text
more goroutines
more memory
more queues
more timeouts
more retries
```

Retry dodatno povećava traffic prema B:

```text
A
│
├── original requests
│
└── retries
        │
        ▼
        B
```

B postaje još sporiji.

To dalje povećava broj blokiranih goroutine-a u A.

Dobijamo:

```text
B slowdown
   ↓
A queue growth
   ↓
goroutine growth
   ↓
resource pressure
   ↓
timeouts
   ↓
retries
   ↓
more traffic
   ↓
B becomes even slower
```

Ovo je klasičan primer cascading failure-a.

Zato concurrency control nije samo performance optimization.

On je deo **reliability architecture**.

---

## Senior-level princip

Debugging concurrent sistema zahteva kombinaciju tri nivoa analize.

### 1. Local correctness

Da li je konkretna operacija bezbedna?

```text
mutex
atomic
channel
synchronization
```

### 2. Lifecycle correctness

Da li komponente pravilno počinju i završavaju?

```text
start
run
cancel
shutdown
wait
terminate
```

### 3. System-level behavior

Kako sistem reaguje na:

```text
load
latency
failure
overload
timeouts
retries
dependency failure
```

Zreo concurrency dizajn mora biti ispravan na sva tri nivoa.

Možemo imati:

```text
Local correctness ✓
Lifecycle correctness ✓
System behavior ✗
```

i sistem će i dalje biti nebezbedan u production-u.

Zato senior-level concurrency engineering nije samo pitanje:

> "Da li postoji race?"

nego:

> "Da li sistem ostaje predvidljiv kada se workload, latency i failure conditions promene?"

Ovaj završni deo treba da poveže prethodne koncepte u jednu celinu.

Na senior nivou se od kandidata više ne očekuje samo poznavanje pojedinačnih Go concurrency mehanizama. Očekuje se sposobnost da proceni **arhitekturu concurrent sistema**, identifikuje failure modes, izabere odgovarajući concurrency model i objasni trade-off-e.

---

### Pitanje 86

**Dobijaš postojeći concurrent sistem. Kako bi pristupio njegovom code review-u?**

### Odgovor

Ne bih odmah krenuo od pitanja:

> "Da li koristi mutex ili channel?"

Prvo bih pokušao da razumem concurrency model.

Analiza bi išla približno ovim redom:

```text
1. Ko proizvodi posao?
2. Ko obrađuje posao?
3. Gde se posao skladišti?
4. Koliko concurrency jedinica postoji?
5. Kako se concurrency ograničava?
6. Kako se propagira cancellation?
7. Kako se radi shutdown?
8. Ko poseduje resurse?
9. Kako se obrađuju greške?
10. Kako sistem reaguje na overload?
```

Tek nakon toga bih analizirao konkretne primitive.

Na primer:

```text
Producer
   │
   ▼
Queue
   │
   ▼
Workers
   │
   ▼
External Dependency
```

Za svaku komponentu bih proverio:

* lifecycle,
* ownership,
* synchronization,
* blocking behavior,
* failure behavior,
* cancellation,
* observability.

Posebno bih tražio implicitne pretpostavke.

Na primer:

```go
go process(request)
```

odmah otvara pitanja:

* Ko kontroliše broj goroutine-a?
* Ko čeka njihov završetak?
* Šta se dešava ako `process` blokira?
* Ko otkazuje operaciju?
* Šta se dešava pri shutdown-u?
* Kako se propagira greška?

Senior code review treba da posmatra **lifecycle**, a ne samo implementaciju pojedinačne funkcije.

---

### Pitanje 87

**Kada bi izabrao channel, a kada mutex?**

### Odgovor

Ne postoji pravilo:

> "Channels su za concurrency, mutexi su loši."

Oba su validna concurrency mehanizma.

Mutex je često prirodan izbor kada više goroutine-a štiti isti mutable state:

```go
type Counter struct {
    mu    sync.Mutex
    value int
}
```

Model:

```text
G1 ─┐
G2 ─┼──> shared state
G3 ─┘
      ▲
      │
    mutex
```

Channel je često prirodniji kada goroutine-e komuniciraju ili prenose ownership:

```text
Producer
    │
    │ Job
    ▼
 Channel
    │
    ▼
Consumer
```

Model decision-a može biti:

```text
Shared mutable state?
        │
       yes
        │
        ▼
     Mutex?
```

ili:

```text
Need to transfer work/data/ownership?
        │
       yes
        │
        ▼
     Channel?
```

Ali to nije apsolutno pravilo.

Senior developer bira mehanizam prema **semantici problema**, a ne prema popularnosti određenog concurrency pattern-a.

---

### Pitanje 88

**Kada bi koristio atomic operacije umesto mutex-a?**

### Odgovor

Atomic je koristan kada imamo veoma jednostavno stanje koje zahteva atomic read-modify-write operacije.

Na primer:

```go
atomic.AddInt64(&counter, 1)
```

Ovo je prirodno za:

* counters,
* flags,
* state transitions,
* lock-free metadata,
* neke performance-sensitive strukture.

Ali atomic ne treba koristiti samo zato što je "brži".

Ako imamo kompleksno invariantno stanje:

```text
balance
reserved
available
limit
```

koje mora biti konzistentno kao jedna celina, mutex može biti mnogo jasniji.

Na primer:

```text
balance >= reserved
```

Ako više vrednosti mora istovremeno da zadovoljava invariant, nekoliko nepovezanih atomic operacija može napraviti komplikovan i teško proverljiv dizajn.

Senior kriterijum je:

> Koji mehanizam najjasnije izražava invariant koji štitimo?

---

### Pitanje 89

**Kako bi dizajnirao concurrency model za sistem koji obrađuje veliki broj nezavisnih poslova?**

### Odgovor

Prvi kandidat je worker pool.

Model:

```text
                    ┌── Worker 1
                    ├── Worker 2
Jobs ──> Queue ─────┼── Worker 3
                    ├── Worker 4
                    └── Worker N
```

Ali worker pool ne bih dizajnirao bez ograničenja.

Potrebno je definisati:

```text
queue capacity
worker count
job timeout
shutdown policy
error policy
retry policy
overload policy
```

Na primer:

```text
Queue capacity = 1,000
Workers        = 20
Job timeout    = 5s
Retries        = 2
```

Ako queue dostigne maksimum, moramo znati šta se dešava:

```text
reject
block
drop
persist
prioritize
```

To je deo arhitekture.

---

### Pitanje 90

**Kako bi pristupio izboru broja worker-a?**

### Odgovor

Ne postoji univerzalna vrednost:

```text
workers = 10
```

Broj worker-a zavisi od karakteristika workload-a.

Za CPU-bound posao concurrency je povezan sa dostupnim CPU resursima.

Za I/O-bound posao možemo imati više concurrent operacija jer workers često čekaju:

```text
network
database
disk
external API
```

Ali veći broj worker-a nije automatski bolji.

Ako povećamo:

```text
10 → 100 → 1.000 workers
```

možemo dobiti:

* contention,
* downstream overload,
* memory pressure,
* scheduler overhead,
* queue contention,
* veći latency zbog resursne konkurencije.

Zato worker count treba određivati merenjem.

Tipičan pristup je:

```text
baseline
   ↓
benchmark
   ↓
profile
   ↓
increase concurrency
   ↓
measure again
```

Concurrency tuning je empirijski problem.

---

### Pitanje 91

**Kako bi dizajnirao graceful shutdown za worker pool?**

### Odgovor

Jedan mogući lifecycle:

```text
Running
   │
   ▼
Shutdown signal
   │
   ▼
Stop accepting new work
   │
   ▼
Cancel/drain according to policy
   │
   ▼
Workers finish/terminate
   │
   ▼
Wait
   │
   ▼
Exit
```

Na primer:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

Workers mogu koristiti:

```go
select {
case job := <-jobs:
    process(ctx, job)

case <-ctx.Done():
    return
}
```

A lifecycle worker-a treba eksplicitno kontrolisati.

Posebno treba definisati razliku između:

### Stop accepting

Ne prihvatamo nove poslove.

### Drain

Dozvoljavamo postojećim poslovima da se završe.

### Cancel

Prekidamo operacije koje podržavaju cancellation.

### Force termination

Prekidamo proces bez čekanja.

Senior dizajn mora jasno definisati koju od ovih semantika sistem zahteva.

---

### Pitanje 92

**Šta bi uradio ako jedan worker može da blokira neograničeno dugo?**

### Odgovor

Ne bih dozvolio da worker lifecycle zavisi od operacije bez definisanog lifetime-a.

Koristio bih:

* context,
* timeout,
* cancellation-aware API,
* bounded resources.

Na primer:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

err := process(ctx, job)
```

Ako `process` propagira context do blocking dependency-ja:

```text
worker
   │
   ▼
operation
   │
   ├── success
   │
   └── timeout/cancel
```

worker može nastaviti sa sledećim poslom.

Ako dependency ignoriše cancellation:

```text
worker
   │
   ▼
unbounded blocking call
```

onda timeout na spoljašnjem nivou možda neće osloboditi stvarni resurs.

Zato je cancellation propagation važnija od samog postojanja `context.Context` parametra.

---

### Pitanje 93

**Kako bi razlikovao correctness problem od performance problema u concurrent sistemu?**

### Odgovor

**Correctness problem** znači da sistem može proizvesti pogrešan rezultat ili pogrešno ponašanje.

Primer:

```text
counter expected = 100
counter actual   = 73
```

**Performance problem** znači da sistem daje ispravan rezultat, ali ne zadovoljava potrebne performance karakteristike.

Primer:

```text
correct result
latency = 4 seconds
SLO     = 500ms
```

Međutim, problemi mogu biti povezani.

Na primer:

```text
too much contention
      ↓
latency
      ↓
timeouts
      ↓
retries
      ↓
more contention
```

Zato senior analiza treba da razlikuje:

```text
correctness
performance
reliability
```

ali i da razume njihove međusobne veze.

---

### Pitanje 94

**Kako bi procenio da li je concurrent dizajn production-ready?**

### Odgovor

Ne bih se zadovoljio time da:

```text
go test ./...
```

prođe.

Production readiness zahteva više dimenzija.

### Correctness

```text
No known data races
No obvious deadlocks
Correct synchronization
Correct ownership
```

### Lifecycle

```text
Start
Cancel
Timeout
Shutdown
Cleanup
```

### Capacity

```text
Bounded concurrency
Bounded queues
Known resource limits
```

### Failure handling

```text
Retries
Timeouts
Cancellation
Partial failure
Dependency failure
Overload
```

### Observability

```text
Metrics
Logs
Tracing
Profiles
Goroutine visibility
```

### Testing

```text
Race detector
Stress tests
Load tests
Failure injection
Timeout scenarios
Shutdown scenarios
```

Production-ready concurrency sistem treba da ima odgovor na pitanje:

> "Šta se dešava kada sve radi normalno?"

ali i:

> "Šta se dešava kada ništa ne radi normalno?"

---

## Pitanje 95

**Dobijaš sledeći zahtev: "Implementiraj concurrent processing sistema koji mora da obradi veliki broj događaja, ne sme da ostane bez kontrole pod load-om i mora da se ugasi bez gubitka već prihvaćenih događaja." Kako bi ga dizajnirao?**

### Odgovor

Ovo je tipičan senior-level problem jer ne postoji jedna concurrency primitive koja ga rešava.

Prvo bih definisao sistemske garancije.

```text
1. Koliko događaja možemo prihvatiti?
2. Koliko događaja možemo istovremeno obraditi?
3. Šta znači "accepted"?
4. Kada događaj može biti izgubljen?
5. Koliko dugo događaj sme da čeka?
6. Šta se dešava ako downstream padne?
7. Šta se dešava pri shutdown-u?
```

Zatim bih dizajnirao bounded pipeline:

```text
                Event Producers
                       │
                       ▼
                ┌─────────────┐
                │ Admission   │
                │ Control     │
                └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │ Bounded     │
                │ Queue       │
                └─────────────┘
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Worker     Worker    Worker
             │         │         │
             └─────────┼─────────┘
                       ▼
                Downstream
```

Za svaki deo definišem lifecycle.

### Producer

Može:

```text
accept
block
reject
```

u zavisnosti od overload policy-ja.

### Queue

Mora biti bounded.

Ne bih koristio neograničeno akumuliranje u memoriji za sistem koji treba da bude otporan na overload.

### Workers

Imali bi eksplicitno ograničen broj:

```text
N workers
```

### Job processing

Svaki job dobija:

```text
context
timeout
error handling
```

### Shutdown

Prilikom shutdown-a:

```text
1. stop accepting new events
2. stop producers
3. preserve accepted events
4. drain queue
5. finish active workers
6. wait for workers
7. release resources
8. terminate
```

Ako zahtev kaže da **prihvaćeni događaji ne smeju biti izgubljeni**, in-memory channel možda nije dovoljan.

Tada bih razmotrio durable queue ili persistent event broker.

To je veoma važna architectural boundary:

```text
In-memory queue
      ≠
Durable queue
```

Ako proces padne:

```text
in-memory queue
      ↓
events lost
```

dok persistent queue može omogućiti recovery.

Senior developer mora razlikovati:

> concurrency guarantee

od:

> durability guarantee.

---

# Završni senior mentalni model

Go concurrency treba posmatrati kao sistem od nekoliko međusobno povezanih slojeva:

```text
┌───────────────────────────────────────┐
│          Business Semantics           │
├───────────────────────────────────────┤
│       Reliability / Failure Model     │
├───────────────────────────────────────┤
│     Backpressure / Capacity Control   │
├───────────────────────────────────────┤
│       Cancellation / Timeouts         │
├───────────────────────────────────────┤
│       Concurrency Architecture        │
├───────────────────────────────────────┤
│ Worker Pools / Pipelines / Fan-In-Out │
├───────────────────────────────────────┤
│      Channels / Mutex / Atomic        │
├───────────────────────────────────────┤
│         Go Runtime / Scheduler        │
└───────────────────────────────────────┘
```

Senior developer ne bira concurrency primitive izolovano.

Prvo definiše:

```text
ownership
lifecycle
capacity
ordering
failure semantics
```

a zatim bira mehanizme koji te zahteve implementiraju.

---

[Prelazak na **Senior+ — Interview Questions**](07-senior-plus.md)