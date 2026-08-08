# Concurrency Testing Strategies

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Concurrency Testing Strategies
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Zašto je testiranje konkurentnog koda teško?
- Determinističko vs nedeterminističko ponašanje
- Tipovi concurrency testova
- Unit testovi i goroutine
- Race Detection
- Stress Testing
- Deadlock Testing
- Fuzz Testing
- Benchmark Testing
- Production-grade strategija

---

# 1. Zašto Je Concurrency Testing Težak?

Kod sekvencijalnog programa:

```
A

↓

B

↓

C
```

redosled je predvidiv.

---

Kod konkurentnog programa:

```
G1

G2

G3
```

redosled izvršavanja zavisi od:

- Scheduler-a
- CPU opterećenja
- vremena
- sistemskih događaja

---

Isti test može:

```
proći 100 puta

↓

pasti 101. put
```

---

Ovakvi problemi se nazivaju:

```
flaky concurrency bugs
```

---

# 2. Nedeterminističko Ponašanje

Primer:

```go
go worker()

result := read()
```

---

Problem:

Ne znamo da li je:

```
worker()

završio
```

pre:

```
read()
```

---

Test može slučajno proći.

---

Ali ne proverava stvarni uslov.

---

# 3. Cilj Concurrency Testiranja

Dobar concurrency test treba da proveri:

- pravilnu sinhronizaciju
- odsustvo data race-a
- korektnu komunikaciju
- pravilno gašenje goroutine-a
- ponašanje pod opterećenjem

---

Ne treba samo proveravati:

```
da li radi jednom.
```

---

Već:

```
da li radi u svim mogućim rasporedima.
```

---

# 4. Tipovi Concurrency Testova

Najvažnije kategorije:

---

## 1. Functional Tests

Proveravaju:

```
da li rezultat odgovara očekivanju.
```

---

## 2. Race Tests

Proveravaju:

```
postoji li data race.
```

---

## 3. Stress Tests

Proveravaju:

```
ponašanje pod opterećenjem.
```

---

## 4. Deadlock Tests

Proveravaju:

```
da li program može ostati blokiran.
```

---

## 5. Fuzz Tests

Proveravaju:

```
veliki broj različitih ulaza i scenarija.
```

---

# 5. Osnovni Concurrency Unit Test

Primer:

```go
func TestCounter(t *testing.T) {

	counter := NewCounter()

	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {

		wg.Add(1)

		go func() {
			defer wg.Done()

			counter.Increment()
		}()
	}

	wg.Wait()

	if counter.Value() != 100 {
		t.Fatal("wrong value")
	}
}
```

---

Test proverava:

- više goroutine-a
- sinhronizaciju
- konačan rezultat

---

# 6. Problem Sa Vremenskim Čekanjem

Loš test:

```go
go worker()

time.Sleep(time.Second)

check()
```

---

Zašto?

Zato što:

```
vreme ≠ sinhronizacija
```

---

Na sporijem računaru:

```
1 sekunda možda nije dovoljna.
```

---

Na brzom:

```
test nepotrebno čeka.
```

---

# 7. Bolji Pristup

Koristiti:

- `sync.WaitGroup`
- channels
- context
- synchronization primitives

---

Primer:

```go
done := make(chan struct{})

go func() {
	work()

	close(done)
}()

<-done
```

---

Ovde test čeka:

```
stvarni događaj
```

---

# 8. Race Detector

Jedan od najvažnijih alata:

```bash
go test -race
```

---

Detektuje:

- istovremeni pristup memoriji
- write/write konflikte
- read/write konflikte

---

Primer problema:

```go
counter++

```

iz više goroutine-a.

---

Bez zaštite:

```
data race
```

---

# 9. Šta Race Detector Radi?

Runtime instrumentacija prati:

- memorijske pristupe
- goroutine interakcije
- sinhronizaciju

---

Kada pronađe konflikt:

prijavljuje:

- lokaciju prvog pristupa
- lokaciju drugog pristupa
- stack trace

---

# 10. Ograničenja Race Detector-a

Race detector nije magično rešenje.

---

Ne pronalazi:

❌ deadlock

❌ logičke greške

❌ pogrešnu sinhronizaciju bez race-a

❌ performance probleme

---

Primer:

Dve goroutine mogu biti savršeno sinhronizovane,

ali rezultat može biti pogrešan.

---

# 11. Concurrency Test Pyramid

Slično klasičnom testiranju:

```
        Stress Tests

            ↑

      Integration Tests

            ↑

       Unit Tests
```

---

Najviše treba imati:

```
brzih unit testova
```

---

Manje:

```
sporih stress testova
```

---

# 12. Senior Pravilo

Concurrency test ne treba da testira:

```
tajming
```

---

Treba da testira:

```
garancije sinhronizacije.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto je concurrency testing težak

✅ determinističko i nedeterminističko ponašanje

✅ tipove concurrency testova

✅ osnovne obrasce testiranja

✅ zašto `time.Sleep` nije dobar

✅ korišćenje WaitGroup i channel-a

✅ race detector osnove

---

# Concurrency Testing Strategies

## Deo #2 — Race Detection Deep Dive

---

# 📚 Sadržaj

- Šta je Data Race?
- Kako Race Detector radi?
- ThreadSanitizer osnova
- `go test -race`
- Tipični Race scenariji
- Race u strukturama podataka
- Race sa mapama
- Race sa slice-ovima
- False Positive situacije
- Race Prevention Patterns
- CI integracija

---

# 1. Šta Je Data Race?

Data race nastaje kada:

```
dve ili više goroutine-a

istovremeno pristupaju istoj memoriji

i najmanje jedna operacija je write.
```

---

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

---

Problem:

Obe goroutine menjaju:

```
istu memorijsku lokaciju.
```

---

# 2. Zašto Je Data Race Opasan?

Data race može izazvati:

- pogrešne rezultate
- nekonzistentno stanje
- teško ponovljive bugove
- različito ponašanje između izvršavanja

---

Primer:

Očekujemo:

```
counter = 2000
```

---

Dobijemo:

```
counter = 1734
```

---

Problem može izgledati potpuno nasumično.

---

# 3. Kako Race Detector Radi?

Go race detector koristi:

```
dynamic analysis
```

tokom izvršavanja programa.

---

Prati:

- memorijske pristupe
- goroutine odnose
- synchronization events

---

Kada pronađe konflikt:

analizira:

```
ko je čitao

ko je pisao

kada

i gde
```

---

# 4. ThreadSanitizer

Go race detector je zasnovan na:

```
ThreadSanitizer (TSan)
```

---

TSan je alat razvijen za detekciju data race problema u konkurentnim programima.

---

Go runtime integriše ovu tehnologiju kroz:

```bash
-race
```

---

# 5. Pokretanje Race Detector-a

Testovi:

```bash
go test -race ./...
```

---

Aplikacija:

```bash
go run -race main.go
```

---

Build:

```bash
go build -race
```

---

Najčešća upotreba:

```
CI pipeline
```

---

# 6. Primer Race Problema

Kod:

```go
type Counter struct {
	value int
}

func (c *Counter) Increment() {
	c.value++
}
```

---

Test:

```go
for i := 0; i < 100; i++ {

	go counter.Increment()

}
```

---

Problem:

```
value++

nije atomska operacija.
```

---

Interno:

```
READ value

+

ADD 1

+

WRITE value
```

---

Dve goroutine mogu preklopiti ove korake.

---

# 7. Race Sa Mapama

Go mape nisu thread-safe.

---

Problem:

```go
data := map[string]int{}

go func() {
	data["a"] = 1
}()

go func() {
	data["b"] = 2
}()
```

---

Rezultat:

```
concurrent map writes
```

---

Rešenja:

- Mutex
- RWMutex
- sync.Map
- channel ownership

---

# 8. Race Sa Slice-ovima

Slice ima:

```
pointer

length

capacity
```

---

Problem:

```go
items := []int{}

go func() {
	items = append(items, 1)
}()

go func() {
	items = append(items, 2)
}()
```

---

`append` može menjati:

- backing array
- length
- capacity

---

Zato postoji race.

---

# 9. Race Sa Pointerima

Primer:

```go
var user *User

go func() {
	user = loadUser()
}()

go func() {
	fmt.Println(user.Name)
}()
```

---

Problem:

Jedna goroutine piše pointer,

druga čita.

---

Rešenja:

- channel
- mutex
- atomic pointer

---

# 10. Race i Memory Visibility

Race nije samo problem vrednosti.

---

Problem je i:

```
kada jedna goroutine vidi promenu druge.
```

---

Bez synchronization:

nema garantovanog:

```
happens-before
```

odnosa.

---

Primer:

```go
ready = true
```

---

Druga goroutine možda nikada ne vidi:

```
ready == true
```

bez pravilne sinhronizacije.

---

# 11. False Positive Situacije

Race detector uglavnom daje tačne rezultate.

---

Ali postoje situacije gde:

- kod koristi unsafe
- ručno implementirani synchronization
- specijalni runtime obrasci

mogu otežati analizu.

---

Primer:

```go
unsafe.Pointer
```

---

Takav kod zahteva dodatnu pažnju.

---

# 12. Race Detector Ne Pronalazi Sve

Važno:

Race-free program nije automatski ispravan.

---

Race detector ne otkriva:

❌ deadlock

❌ starvation

❌ pogrešan redosled događaja

❌ logičke greške

---

Primer:

Dve goroutine koriste Mutex pravilno,

ali zaključavanje je pogrešno dizajnirano.

---

Nema race-a,

ali program je pogrešan.

---

# 13. Race Prevention Patterns

## Pattern 1 — Mutex Protection

```go
mu.Lock()

counter++

mu.Unlock()
```

---

## Pattern 2 — Atomic Operations

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

## Pattern 3 — Channel Ownership

Umesto:

```
više goroutine-a menja podatke
```

koristi:

```
jedna goroutine poseduje stanje.
```

---

# 14. CI Integracija

Preporučena praksa:

```yaml
test:
  script:
    - go test -race ./...
```

---

Race testovi treba da budu deo:

- pull request provere
- nightly build-a
- release pipeline-a

---

# 15. Race Testing Strategija

Dobar pristup:

```
Unit Tests

↓

Race Detector

↓

Stress Test

↓

Production Monitoring
```

---

Svaki sloj hvata drugačiji tip problema.

---

# 16. Senior Pravila

✔️ Svaki concurrent kod treba testirati sa `-race`.

---

✔️ Race detector treba koristiti često, ne samo kada postoji bug.

---

✔️ Race-free nije isto što i correctness.

---

✔️ Sinhronizacija mora biti deo dizajna.

---

✔️ Najbolji race fix je često bolja arhitektura.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je data race

✅ kako race detector radi

✅ ThreadSanitizer osnove

✅ `go test -race`

✅ race sa mapama

✅ race sa slice-ovima

✅ race sa pointerima

✅ false positive situacije

✅ race prevention patterns

✅ CI integraciju

---

# Concurrency Testing Strategies

## Deo #3 — Deterministic Concurrency Testing

---

# 📚 Sadržaj

- Problem nedeterminističkih testova
- Determinističko izvršavanje
- Synchronization Points
- Test Harness dizajn
- Channels kao test alat
- Kontrola goroutine lifecycle-a
- Eliminacija `time.Sleep`
- Reproducibilni concurrency testovi
- Primeri dobrih obrazaca

---

# 1. Problem Nedeterminističkih Testova

Najveći problem kod concurrency testova:

```
redosled izvršavanja
nije garantovan.
```

---

Primer:

```go
go process()

assert(result)
```

---

Test pretpostavlja:

```
process()

će završiti
```

pre:

```
assert()
```

---

Ali Go scheduler ne garantuje taj redosled.

---

# 2. Šta Je Deterministički Test?

Deterministički test je test koji:

```
uvek daje isti rezultat

bez obzira na scheduler.
```

---

Dobijamo:

```
isti ulaz

+

isti synchronization događaji

=

isti rezultat
```

---

# 3. Loš Pattern — `time.Sleep`

Primer:

```go
go worker()

time.Sleep(time.Second)

check()
```

---

Problem:

Test zavisi od:

- brzine CPU-a
- opterećenja sistema
- CI okruženja
- garbage collector-a

---

Na lokalnoj mašini:

```
prođe
```

---

U CI:

```
flaky fail
```

---

# 4. Pravilno Čekanje

Umesto vremena:

koristiti:

- channel
- WaitGroup
- context
- condition signal

---

Primer:

```go
done := make(chan struct{})

go func() {

	work()

	close(done)

}()

<-done
```

---

Sada čekamo:

```
stvarni događaj
```

---

# 5. Synchronization Points

Synchronization point je mesto gde test eksplicitno kontroliše tok.

---

Primer:

```
Test

↓

start goroutine

↓

čekaj signal

↓

dozvoli nastavak

↓

proveri rezultat
```

---

Ovo uklanja slučajnost.

---

# 6. Testiranje Channel Komunikacije

Channel je prirodan alat za kontrolu.

---

Primer:

```go
func worker(
	start <-chan struct{},
	done chan<- bool,
) {

	<-start

	process()

	done <- true
}
```

---

Test:

```go
start := make(chan struct{})
done := make(chan bool)

go worker(start, done)

close(start)

<-done
```

---

Test kontroliše:

```
kada worker počinje.
```

---

# 7. Kontrola Goroutine Lifecycle-a

Dobar test mora znati:

- kada goroutine kreće
- kada završava
- kako se gasi

---

Loš dizajn:

```go
go backgroundTask()
```

---

bez:

- cancel mehanizma
- done signala
- context-a

---

---

# 8. Testiranje Shutdown-a

Primer:

```go
ctx, cancel := context.WithCancel(
	context.Background(),
)

go worker(ctx)

cancel()

<-done
```

---

Test proverava:

```
da li goroutine pravilno završava.
```

---

# 9. Test Harness

Test harness je pomoćna struktura koja:

- kreira goroutine
- kontroliše događaje
- skuplja rezultate
- proverava stanje

---

Primer:

```go
type Harness struct {
	start chan struct{}
	done  chan struct{}
}
```

---

Koristi se za složenije concurrency scenarije.

---

# 10. Simulacija Različitih Redosleda

Neki bugovi se pojavljuju samo u određenom rasporedu.

---

Možemo forsirati redosled:

```
G1 start

↓

G1 pause

↓

G2 start

↓

G1 continue
```

---

Primer:

```go
pause := make(chan struct{})

<-pause
```

---

Test postaje kontrolisan.

---

# 11. Testing Race Conditions Deterministički

Primer:

Imamo:

```go
balance -= amount
```

---

Želimo da obe goroutine:

pročitaju staru vrednost pre izmene.

---

Test može koristiti:

```
barrier
```

---

Tok:

```
G1 READ

↓

G2 READ

↓

G1 WRITE

↓

G2 WRITE
```

---

Sada možemo reprodukovati problem.

---

# 12. Barrier Pattern

Barrier omogućava:

```
više goroutine-a čeka

dok svi ne stignu do iste tačke.
```

---

Primer:

```go
barrier := make(chan struct{})

worker1()

worker2()

close(barrier)
```

---

Obe goroutine nastavljaju zajedno.

---

# 13. Semaphore Pattern u Testovima

Semaphore može kontrolisati broj aktivnih goroutine-a.

---

Primer:

```go
sem := make(chan struct{}, 2)
```

---

Dozvoljava:

```
maksimalno 2 aktivne goroutine.
```

---

Koristan za:

- load test
- resource control

---

# 14. Eliminacija Flaky Testova

Najčešći uzroci:

❌ `time.Sleep`

❌ globalno stanje

❌ nedeterministički redosled

❌ zavisnost od CPU vremena

❌ nedostatak cleanup-a

---

Rešenja:

✔️ eksplicitna sinhronizacija

✔️ izolacija testova

✔️ kontrolisan lifecycle

✔️ deterministic scheduler points

---

# 15. Reproducibilni Concurrency Test

Dobar test ima:

```
setup

↓

controlled execution

↓

assertion

↓

cleanup
```

---

Primer:

```go
setup()

startWorkers()

triggerEvent()

waitForCompletion()

verify()

cleanup()
```

---

# 16. Testing Goroutine Cleanup

Važno je proveriti:

```
da li je broj goroutine-a isti
nakon testa.
```

---

Primer:

```go
before :=
	runtime.NumGoroutine()

runTest()

after :=
	runtime.NumGoroutine()
```

---

Ako:

```
after >> before
```

mogući leak.

---

# 17. Senior Pravila

✔️ Test ne treba da se oslanja na vreme.

---

✔️ Test treba da kontroliše događaje.

---

✔️ Channel je alat za koordinaciju, ne samo transport podataka.

---

✔️ Svaka goroutine treba imati jasan lifecycle.

---

✔️ Reproduktivnost je važnija od slučajnog pokrivanja.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ determinističko concurrency testiranje

✅ problem `time.Sleep`

✅ synchronization points

✅ channel kao test alat

✅ goroutine lifecycle kontrolu

✅ test harness dizajn

✅ barrier pattern

✅ eliminaciju flaky testova

---

# Concurrency Testing Strategies

## Deo #4 — Stress Testing i Load Testing Concurrent Systems

---

# 📚 Sadržaj

- Šta je Concurrency Stress Testing?
- Razlika između Load i Stress Testing-a
- Ciljevi stress testiranja
- Race Amplification
- Random Scheduling
- Long-running Tests
- Goroutine Explosion Testing
- Memory Pressure Testing
- Production-like Scenarios
- Stress Testing Workflow

---

# 1. Šta Je Concurrency Stress Testing?

Stress testing proverava:

```
kako se sistem ponaša

pod ekstremnim uslovima.
```

---

Za razliku od običnog testa:

```
1 goroutine

10 operacija
```

---

Stress test:

```
10000 goroutine-a

milioni operacija
```

---

Cilj nije samo:

```
da li rezultat postoji
```

---

Već:

```
da li sistem ostaje korektan pod pritiskom.
```

---

# 2. Load vs Stress Testing

Važno razlikovati.

---

## Load Testing

Pita:

```
Kako sistem radi

sa očekivanim opterećenjem?
```

---

Primer:

```
1000 request/s
```

---

## Stress Testing

Pita:

```
Šta se dešava

kada pređemo granice?
```

---

Primer:

```
100000 request/s
```

---

# 3. Ciljevi Stress Testiranja

Concurrency stress test proverava:

- race uslove
- deadlock situacije
- memory leak
- goroutine leak
- starvation
- degradaciju performansi

---

# 4. Osnovni Stress Test Primer

Primer:

```go
func TestStressCounter(t *testing.T) {

	counter := NewCounter()

	var wg sync.WaitGroup

	for i := 0; i < 10000; i++ {

		wg.Add(1)

		go func() {
			defer wg.Done()

			counter.Increment()
		}()
	}

	wg.Wait()

	if counter.Value() != 10000 {
		t.Fatal("invalid result")
	}
}
```

---

Ovaj test povećava verovatnoću:

```
otkrivanja race problema.
```

---

# 5. Race Amplification

Neki bugovi su retki.

---

Primer:

```
1 / 100000 izvršavanja
```

---

Stress test povećava broj pokušaja.

---

Umesto:

```
100 operacija
```

radimo:

```
10 miliona operacija
```

---

Verovatnoća pronalaženja problema raste.

---

# 6. Random Scheduling

Go scheduler može izvršavati goroutine različitim redosledom.

---

Stress test može koristiti:

- random input
- random delay
- random operation order

---

Primer:

```go
time.Sleep(
	time.Duration(rand.Intn(10)) *
	time.Millisecond,
)
```

---

Cilj:

```
što više različitih interleaving scenarija.
```

---

Napomena:

Random ne treba koristiti umesto sinhronizacije.

Koristi se za:

```
povećanje pokrivenosti.
```

---

# 7. Long-running Concurrency Testovi

Neki problemi se pojavljuju tek posle vremena.

---

Primer:

```
prvih 5 minuta

sve radi
```

---

Posle:

```
memory raste

goroutine raste
```

---

Long-running test može trajati:

```
10 min

1 sat

preko noći
```

---

# 8. Primer Long-running Testa

```go
func TestLongRunning(t *testing.T) {

	timeout :=
		time.After(30 * time.Minute)

	for {
		select {

		case <-timeout:
			return

		default:
			runScenario()
		}
	}
}
```

---

Koristi se za:

- leak detection
- stability testing

---

# 9. Goroutine Explosion Testing

Jedan od čestih problema:

```
neograničeno kreiranje goroutine-a.
```

---

Primer:

```go
for {
	go handleRequest()
}
```

---

Stress test proverava:

- maksimalan broj goroutine-a
- ponašanje pod opterećenjem
- cleanup

---

Merenje:

```go
runtime.NumGoroutine()
```

---

# 10. Memory Pressure Testing

Concurrency problemi često izgledaju kao memory problemi.

---

Testira se:

- veliki broj objekata
- veliki payload-i
- paralelne alokacije

---

Primer:

```
10000 goroutine

×

kreiraju velike strukture
```

---

Posmatramo:

- heap rast
- GC aktivnost
- OOM rizik

---

# 11. Testing Worker Pool-a

Primer pitanja:

```
Šta se dešava

kada broj taskova eksplodira?
```

---

Test:

```
workers = 10

tasks = 1 000 000
```

---

Proveravamo:

- queue ponašanje
- backpressure
- cleanup
- latency

---

# 12. Testing Channel Overflow-a

Buffered channel:

```go
jobs :=
	make(chan Job, 100)
```

---

Testira se:

šta se dešava kada:

```
producer

brži od

consumer-a
```

---

Mogući problemi:

- blokiranje
- memory growth
- starvation

---

# 13. Production-like Scenario

Dobar stress test imitira produkciju.

---

Primer:

API server:

```
Requests

↓

Handlers

↓

Workers

↓

Database calls
```

---

Test treba uključiti:

- realne veličine podataka
- realan broj korisnika
- realne timeout-e

---

# 14. Stress Testing Workflow

```
Definiši cilj

↓

Kreiraj scenario

↓

Povećaj concurrency

↓

Pokreni dugo

↓

Prati metrike

↓

Analiziraj problem

↓

Optimizuj
```

---

# 15. Metrike Koje Treba Pratiti

Tokom stress testa:

## Goroutines

```
runtime.NumGoroutine()
```

---

## Memory

- heap size
- allocations
- GC cycles

---

## CPU

- usage
- scheduler behavior

---

## Latency

- p50
- p95
- p99

---

# 16. Najčešće Greške

❌ testira se samo mali broj goroutine-a

❌ test traje prekratko

❌ ignoriše se cleanup

❌ meri se samo rezultat

❌ ne prati se memorija

---

# 17. Senior Pravila

✔️ Stress test nije zamena za unit test.

---

✔️ Cilj je pronaći granice sistema.

---

✔️ Uvek pratiti resurse, ne samo correctness.

---

✔️ Dugi testovi otkrivaju drugačije probleme.

---

✔️ Production workload je najbolji model.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ concurrency stress testing

✅ load vs stress razliku

✅ race amplification

✅ random scheduling

✅ long-running testove

✅ goroutine explosion testiranje

✅ memory pressure testove

✅ production-like scenarije

---

# Concurrency Testing Strategies

## Deo #5 — Deadlock Testing, Goroutine Leak Detection i Flaky Test Prevention

---

# 📚 Sadržaj

- Šta je Deadlock Testing?
- Detekcija blokiranih testova
- Timeout strategije
- Goroutine Leak Detection
- Goroutine Snapshot analiza
- Cleanup Patterns
- Flaky Test Prevention
- CI Stabilnost
- Production-grade test principi

---

# 1. Šta Je Deadlock Testing?

Deadlock test proverava:

```
da li konkurentni sistem

može ostati zauvek blokiran.
```

---

Deadlock nastaje kada:

```
Goroutine A čeka B

B čeka A
```

---

Primer:

```
Lock A

↓

čekaj Lock B


Lock B

↓

čekaj Lock A
```

---

Nijedna goroutine ne može nastaviti.

---

# 2. Zašto Deadlock Testiranje?

Deadlock problemi su opasni zato što:

- aplikacija ne mora crash-ovati
- CPU može biti nizak
- logovi često ne pokazuju uzrok
- pojavljuju se samo u određenom rasporedu

---

Primer:

```
99.99% vremena radi

0.01% vremena zamrzne
```

---

# 3. Timeout kao Zaštita

Svaki concurrency test treba imati:

```
maksimalno vreme izvršavanja.
```

---

Loš test:

```go
func TestWorker(t *testing.T) {

	runWorker()

}
```

---

Ako postoji deadlock:

```
test nikada ne završava.
```

---

Bolje:

```go
func TestWorker(t *testing.T) {

	done := make(chan struct{})

	go func() {

		runWorker()

		close(done)

	}()

	select {

	case <-done:
		// success

	case <-time.After(time.Second):
		t.Fatal("deadlock detected")
	}
}
```

---

# 4. `testing.T.Deadline`

Go testing framework već ima deadline podršku.

---

Primer:

```go
deadline, ok :=
	t.Deadline()
```

---

Možemo koristiti:

```
preostalo vreme testa
```

za cleanup i kontrolu.

---

# 5. Testing Mutex Deadlock-a

Primer problema:

```go
mu1.Lock()

mu2.Lock()
```

---

Druga goroutine:

```go
mu2.Lock()

mu1.Lock()
```

---

Test strategija:

forsirati:

```
G1 dođe do Lock 1

G2 dođe do Lock 2

nastaviti obe
```

---

Tako reprodukujemo problem.

---

# 6. Channel Deadlock Testiranje

Čest problem:

```go
ch := make(chan int)

ch <- 1
```

---

Unbuffered channel zahteva:

```
receiver
```

---

Ako receiver ne postoji:

```
blokada zauvek.
```

---

Test treba proveriti:

- ko šalje
- ko prima
- kada se channel zatvara

---

# 7. Goroutine Leak Detection

Goroutine leak znači:

```
goroutine nastane

ali nikada ne završi.
```

---

Primer:

```go
go func() {

	for {
		select {

		case <-data:
		
		}

	}

}()
```

---

Bez shutdown mehanizma:

```
goroutine ostaje zauvek.
```

---

# 8. Osnovna Leak Detekcija

Pre testa:

```go
before :=
	runtime.NumGoroutine()
```

---

Posle:

```go
after :=
	runtime.NumGoroutine()
```

---

Ako:

```
after > before
```

postoji sumnja.

---

# 9. Problem Sa Jednostavnim Brojanjem

Broj goroutine-a nije dovoljan.

---

Razlog:

Go runtime ima:

- garbage collection goroutine
- runtime goroutine
- background taskove

---

Zato treba:

- čekati stabilizaciju
- ponavljati merenje
- analizirati stack trace

---

# 10. Goroutine Snapshot

Najbolji način:

```
goroutine dump
```

---

Primer:

```go
buf := make([]byte, 1<<20)

n :=
	runtime.Stack(
		buf,
		true,
	)

fmt.Println(
	string(buf[:n]),
)
```

---

Dobijamo:

```
svaki aktivni stack.
```

---

Možemo pronaći:

```
koja goroutine nikada nije završila.
```

---

# 11. Cleanup Pattern

Svaka goroutine treba imati:

```
start

↓

work

↓

shutdown
```

---

Primer:

```go
func worker(
	ctx context.Context,
) {

	for {

		select {

		case <-ctx.Done():
			return

		default:
			process()

		}

	}
}
```

---

Test:

```go
cancel()
```

---

i proverava:

```
worker završava.
```

---

# 12. Flaky Test Prevention

Flaky test:

```
nekad prolazi

nekad pada
```

---

Kod concurrency testova najčešći uzroci:

- race
- sleep zavisnost
- globalno stanje
- random redosled
- nedostatak cleanup-a

---

# 13. Kako Ukloniti Flaky Testove?

## Korak 1

Identifikovati:

```
šta zavisi od vremena.
```

---

## Korak 2

Zameniti:

```
time.Sleep
```

sa:

```
event synchronization
```

---

## Korak 3

Dodati:

```
determinističku kontrolu
```

---

# 14. CI Stabilnost

Concurrency testovi u CI treba da budu:

- ponovljivi
- izolovani
- ograničenog trajanja
- bez spoljašnjih zavisnosti

---

Preporuka:

```bash
go test -race ./...
```

---

uz:

```bash
go test -count=100 ./...
```

---

za hvatanje retkih problema.

---

# 15. Reproducibility Strategy

Kada test padne:

sačuvati:

- random seed
- input
- konfiguraciju
- log događaja

---

Cilj:

```
ponovo reprodukovati isti scenario.
```

---

# 16. Finalni Concurrency Test Checklist

Pre merge-a:

```
□ nema data race

□ nema goroutine leak

□ nema deadlock rizika

□ nema time.Sleep zavisnosti

□ cleanup postoji

□ test je determinističan

□ CI prolazi stabilno
```

---

# 17. Senior Pravila

✔️ Test koji može da visi je loš test.

---

✔️ Svaka goroutine mora imati vlasnika lifecycle-a.

---

✔️ Cleanup je deo testa, ne opcija.

---

✔️ Flaky concurrency test često ukazuje na dizajn problem.

---

✔️ Reproducibilnost je ključna.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ deadlock testing

✅ timeout zaštitu

✅ channel deadlock analizu

✅ goroutine leak detection

✅ goroutine snapshot

✅ cleanup patterns

✅ flaky test prevenciju

✅ CI stabilnost

---

# Concurrency Testing Strategies

## Deo #6 — Advanced Concurrency Testing Architecture — Final Review

---

# 📚 Sadržaj

- Advanced Concurrency Test Architecture
- Integration Concurrency Testing
- Fuzz Testing Concurrent Systems
- Benchmark Testing Strategy
- Production-grade Test Pipeline
- Final Concurrency Testing Checklist
- Senior Mindset

---

# 1. Advanced Concurrency Test Architecture

Kod jednostavnih sistema dovoljan je:

```
unit test

+

race detector
```

---

Ali kompleksni sistemi zahtevaju više nivoa.

---

Primer:

```
                 Production Simulation

                         ↑

              Integration Tests

                         ↑

               Concurrency Unit Tests

                         ↑

                 Race Detection
```

---

Svaki nivo otkriva različite probleme.

---

# 2. Concurrency Unit Tests

Unit test treba proveriti:

- pojedinačnu komponentu
- synchronization pravila
- lifecycle goroutine-a
- channel ponašanje

---

Primer:

Worker:

```
input channel

↓

processing

↓

output channel
```

---

Testira se:

- pravilna obrada
- zatvaranje channel-a
- cancel ponašanje

---

# 3. Integration Concurrency Testing

Integration test proverava:

```
više komponenti

koje rade zajedno.
```

---

Primer:

```
HTTP Server

↓

Worker Pool

↓

Queue

↓

Database
```

---

Problemi koji se pojavljuju:

- race između komponenti
- pogrešan shutdown
- backpressure problemi
- resource exhaustion

---

# 4. Testiranje Shutdown Sekvence

Produkcioni sistemi moraju pravilno da se gase.

---

Scenario:

```
Application receives SIGTERM

↓

stop accepting requests

↓

finish active work

↓

shutdown workers

↓

close resources
```

---

Test treba proveriti:

- nema izgubljenih taskova
- nema zaglavljenih goroutine-a
- nema curenja resursa

---

# 5. Fuzz Testing Concurrent Systems

Fuzz testing istražuje:

```
veliki broj neočekivanih ulaza.
```

---

Kod concurrency sistema može testirati:

- različite sekvence događaja
- različite veličine podataka
- različite timing scenarije

---

Primer:

```
send

receive

cancel

retry

timeout

close
```

---

Različite kombinacije mogu otkriti:

- deadlock
- race
- invalid state transition

---

# 6. Fuzzing Event Sequence-a

Kod konkurentnih sistema često nije problem:

```
koji podatak
```

---

nego:

```
kojim redosledom događaji nastaju.
```

---

Primer:

Normalno:

```
Start

↓

Process

↓

Stop
```

---

Problematično:

```
Start

↓

Stop

↓

Process
```

---

Fuzzing može generisati ovakve sekvence.

---

# 7. Benchmark Strategija

Concurrency benchmark meri:

- throughput
- latency
- contention
- scalability

---

Primer:

```go
func BenchmarkWorkerPool(
	b *testing.B,
) {

	for i := 0; i < b.N; i++ {

		process()

	}

}
```

---

Pokretanje:

```bash
go test -bench=.
```

---

# 8. Benchmark Sa Race Detector-om

Korisno za sigurnost:

```bash
go test \
-race \
-bench=.
```

---

Ali:

race detector dodaje overhead.

---

Zato rezultate treba interpretirati pažljivo.

---

# 9. Scalability Testing

Pitanje:

```
Kako sistem raste?
```

---

Testirati:

```
1 worker

↓

10 workers

↓

100 workers

↓

1000 workers
```

---

Meriti:

- throughput
- latency
- memory
- CPU

---

# 10. Production-grade Concurrency Pipeline

Profesionalni CI pipeline:

```
Commit

↓

Unit Tests

↓

Race Detector

↓

Integration Tests

↓

Stress Tests

↓

Benchmarks

↓

Production Monitoring
```

---

# 11. Recommended Commands

Osnovni testovi:

```bash
go test ./...
```

---

Race:

```bash
go test -race ./...
```

---

Ponovljeno izvršavanje:

```bash
go test -count=100 ./...
```

---

Benchmark:

```bash
go test -bench=. -benchmem
```

---

Profilisanje:

```bash
go test -trace trace.out
```

---

# 12. Final Concurrency Testing Checklist

## Correctness

```
□ rezultat je tačan

□ nema race-a

□ nema deadlock-a
```

---

## Lifecycle

```
□ goroutine završavaju

□ cleanup postoji

□ shutdown radi
```

---

## Performance

```
□ nema nepotrebnog contention-a

□ throughput je prihvatljiv

□ memory usage je stabilan
```

---

## Reliability

```
□ testovi nisu flaky

□ CI je stabilan

□ problemi mogu da se reprodukuju
```

---

# 13. Najčešći Anti-pattern-i

❌ `time.Sleep` kao sinhronizacija

❌ ignorisanje race detector-a

❌ beskonačne goroutine

❌ testiranje samo happy path-a

❌ benchmark bez profilisanja

❌ ignorisanje cleanup-a

---

# 14. Senior Concurrency Testing Mindset

Junior pristup:

```
Da li kod radi?
```

---

Senior pristup:

```
Kako kod reaguje

kada se sve desi istovremeno?
```

---

Pitanja koja senior postavlja:

```
Ko poseduje stanje?

Ko kontroliše lifecycle?

Kako se sistem gasi?

Šta se dešava pod pritiskom?

Kako dokazujemo korektnost?
```

---

# 15. Završni Pregled Modula

U ovom modulu naučili smo:

---

## Race Detection

✅ data race

✅ ThreadSanitizer

✅ `-race`

---

## Deterministic Testing

✅ synchronization points

✅ channels

✅ lifecycle kontrola

---

## Stress Testing

✅ load vs stress

✅ race amplification

✅ long-running tests

---

## Reliability Testing

✅ deadlock detection

✅ goroutine leak detection

✅ flaky prevention

---

## Advanced Testing

✅ integration testing

✅ fuzz testing

✅ benchmarks

✅ CI pipeline

---

# 16. Šta Sada Znaš?

Nakon ovog modula možeš:

- dizajnirati concurrency test arhitekturu
- testirati goroutine lifecycle
- pronaći race probleme
- sprečiti deadlock situacije
- pisati stabilne CI testove
- meriti concurrency performanse
- pripremiti Go aplikaciju za production workload

---

### ➡️ Sledeća lekcija **[**Concurrency Exercises**](11-concurrency-exercises.md)**

Obuhvatiće:

- praktične zadatke
- goroutine vežbe
- channel probleme
- synchronization izazove
- worker pool implementacije
- pipeline zadatke
- race debugging vežbe
- senior-level concurrency probleme