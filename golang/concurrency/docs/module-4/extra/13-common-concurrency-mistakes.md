# Common Concurrency Mistakes

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Common Concurrency Mistakes
> 
> Level: Intermediate → Advanced

---

# 📚 Sadržaj

- Zašto su concurrency greške teške?
- Kategorije grešaka
- Race Condition
- Data Race
- Race vs Data Race
- Deadlock
- Livelock
- Starvation
- Opšti principi

---

# 1. Zašto Su Concurrency Greške Posebne?

> Pisanje konkurentnih programa nije teško zato što su goroutine ili channel-i komplikovani, već zato što su greške često nedeterminističke. Program može raditi hiljadama puta ispravno, a zatim se iznenada srušiti ili zaglaviti zbog jednog loše sinhronizovanog pristupa.

Kod sekvencijalnog programa izvršavanje je uglavnom predvidivo.

```
A

↓

B

↓

C
```

---

Kod konkurentnog programa:

```
A

↘

 B

↗

C
```

---

Redosled izvršavanja zavisi od:

- Go Scheduler-a
- operativnog sistema
- dostupnih CPU jezgara
- vremena izvršavanja
- blokiranja
- spoljašnjih događaja

---

Zbog toga iste greške nije uvek lako reprodukovati.

---

# 2. Najčešće Kategorije Grešaka

U praksi se najčešće sreću:

- Race Condition
- Data Race
- Deadlock
- Livelock
- Starvation
- Goroutine Leak
- Channel Misuse
- Pogrešna sinhronizacija
- Pogrešna upotreba `context.Context`

---

Svaka od njih ima drugačiji uzrok i drugačiji način rešavanja.

---

# 3. Race Condition

Race Condition predstavlja logičku grešku.

Program zavisi od toga:

```
ko

će

prvi

stići.
```

---

Primer:

```go
if !exists(id) {
	create(id)
}
```

---

Ako dve goroutine izvrše isti kod istovremeno:

```
obe mogu zaključiti

da objekat ne postoji.
```

---

Rezultat:

duplirani podaci ili nekonzistentno stanje.

---

# 4. Data Race

Data Race predstavlja istovremeni pristup istoj memorijskoj lokaciji bez odgovarajuće sinhronizacije, pri čemu je bar jedan pristup upis.

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

Obe goroutine menjaju:

```
counter
```

bez zaštite.

---

Rezultat je:

```
nedefinisano ponašanje.
```

---

# 5. Race Condition vs Data Race

Važno je razlikovati ova dva pojma.

Race Condition:

- logički problem
- može postojati i bez data race-a

---

Data Race:

- problem pristupa memoriji
- otkriva ga:

```bash
go test -race
```

---

Moguće je imati:

✔ Race Condition bez Data Race-a.

---

I:

✔ Data Race bez očiglednog logičkog problema.

---

# 6. Deadlock

Deadlock nastaje kada goroutine međusobno čekaju resurse i nijedna ne može da nastavi izvršavanje.

Primer:

```
G1

↓

čeka G2
```

---

```
G2

↓

čeka G1
```

---

Rezultat:

```
beskonačno čekanje.
```

---

Go Runtime često prijavljuje:

```text
fatal error:
all goroutines are asleep - deadlock!
```

---

# 7. Livelock

Kod livelock-a goroutine nisu blokirane.

One rade.

Ali:

```
nikada

ne završavaju

koristan posao.
```

---

Primer:

Dve goroutine stalno ustupaju prednost jedna drugoj.

```
A

↓

B

↓

A

↓

B
```

---

Rezultat:

```
0 napretka.
```

---

# 8. Starvation

Starvation znači:

jedna goroutine veoma dugo ili zauvek ne dobija priliku da izvrši posao.

Primer:

```
G1

uvek dobija lock
```

---

```
G2

uvek čeka
```

---

Scheduler pokušava da umanji ovakve situacije, ali ih loš dizajn aplikacije može izazvati.

---

# 9. Zašto Se Ove Greške Teško Otkrivaju?

Jer često zavise od:

- vremena izvršavanja
- opterećenja sistema
- broja CPU jezgara
- scheduler odluka

---

Program može:

```
1000 puta

raditi ispravno

↓

1001. put

pasti.
```

---

# 10. Najvažniji Alati

Za otkrivanje problema koriste se:

```bash
go test -race
```

---

```bash
go tool trace
```

---

```bash
go tool pprof
```

---

Takođe:

- `runtime.Stack()`
- `runtime.NumGoroutine()`
- `net/http/pprof`

---

# 11. Opšti Principi

Prilikom pisanja konkurentnog koda:

✔ Jasno definiši vlasništvo nad podacima.

---

✔ Izbegavaj deljenje promenljivih kada je moguće.

---

✔ Komuniciraj preko channel-a kada odgovara problemu.

---

✔ Koristi `Mutex` kada više goroutine mora deliti stanje.

---

✔ Dodaj mogućnost otkazivanja pomoću `context.Context`.

---

✔ Uvek planiraj kako će se goroutine završiti.

---

# 12. Mentalni Model

Zamisli raskrsnicu bez semafora.

Ako više automobila istovremeno pokušava da prođe:

- može doći do sudara
- može doći do zastoja
- neko može zauvek čekati
- svi mogu stalno da se pomeraju bez prolaska

---

To odgovara:

- Data Race
- Deadlock
- Starvation
- Livelock

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto su concurrency greške teške

✅ razliku između Race Condition i Data Race

✅ šta su Deadlock, Livelock i Starvation

✅ zašto su ove greške često nedeterminističke

✅ koje alate koristiti za analizu

---

# Common Concurrency Mistakes

## Deo #2 — Race Condition i Data Race u Dubinu

---

# 📚 Sadržaj

- Šta je Data Race?
- Tipični uzroci
- Race nad promenljivama
- Race nad pokazivačima
- Race nad map-ama
- Atomic vs Mutex
- Kako sprečiti Data Race?

---

# 1. Šta Je Data Race?

Data Race nastaje kada su ispunjena sva tri uslova:

1. Dve ili više goroutine pristupaju istoj memorijskoj lokaciji.
2. Najmanje jedan pristup je upis (`write`).
3. Pristupi nisu pravilno sinhronizovani.

---

Vizuelno:

```
        Goroutine A
             │
             ▼
        shared memory
             ▲
             │
        Goroutine B
```

Ako nema mehanizma sinhronizacije, rezultat je **nedefinisano ponašanje**.

---

# 2. Najjednostavniji Primer

```go
package main

import "fmt"

var counter int

func main() {
	go func() {
		counter++
	}()

	go func() {
		counter++
	}()

	fmt.Println(counter)
}
```

---

Problem:

```
Read

↓

Modify

↓

Write
```

Operacija `counter++` nije atomska.

---

Mogući rezultat:

- `0`
- `1`
- `2`

ili potpuno nepredvidivo ponašanje.

---

# 3. Kako Radi `counter++`?

Izraz:

```go
counter++
```

nije jedna instrukcija.

Pojednostavljeno:

```
LOAD counter

↓

ADD 1

↓

STORE counter
```

---

Ako dve goroutine izvršavaju isti niz instrukcija istovremeno, može doći do gubitka jednog ažuriranja (*lost update*).

---

# 4. Race Nad Pokazivačem

Primer:

```go
type User struct {
	Name string
}

var u = &User{}
```

---

Dve goroutine:

```go
go func() {
	u.Name = "Alice"
}()

go func() {
	u.Name = "Bob"
}()
```

---

Obe menjaju isto polje iste strukture.

Rezultat zavisi od redosleda izvršavanja.

---

# 5. Race Nad Struct Poljima

```go
type Counter struct {
	Value int
}

var c Counter
```

---

```go
go func() {
	c.Value++
}()

go func() {
	c.Value++
}()
```

---

Iako se pristupa polju strukture, problem je isti:

```
shared memory

↓

write

↓

race
```

---

# 6. Race Nad Map-ama

Jedna od najčešćih grešaka u Go-u.

```go
cache := map[string]int{}
```

---

Istovremeno:

```go
go func() {
	cache["A"] = 1
}()

go func() {
	cache["B"] = 2
}()
```

---

ili:

```go
go func() {
	cache["A"] = 1
}()

go func() {
	_ = cache["A"]
}()
```

---

Ugrađene mape (**built-in maps**) nisu bezbedne za istovremeni pristup.

Go može prijaviti:

```text
fatal error:
concurrent map writes
```

ili:

```text
fatal error:
concurrent map read and map write
```

---

# 7. Kako Rešiti Race Nad Map-om?

Najčešći pristupi:

### `sync.Mutex`

```go
var mu sync.Mutex

mu.Lock()
cache["A"] = 1
mu.Unlock()
```

---

### `sync.RWMutex`

Omogućava više istovremenih čitalaca i jednog pisca.

Pogodno kada je broj čitanja znatno veći od broja upisa.

---

### `sync.Map`

Koristan za specifične obrasce pristupa, naročito kada postoji mnogo paralelnih čitanja i retke izmene.

Nije univerzalna zamena za običnu mapu.

---

# 8. Atomic vs Mutex

Ako delimo samo jedan broj:

```go
var counter int64
```

možemo koristiti atomske operacije.

Primer:

```go
atomic.AddInt64(&counter, 1)
```

---

Ako delimo složenije stanje:

```go
type Account struct {
	Balance int
	Owner   string
}
```

bolji izbor je:

```go
sync.Mutex
```

---

Pravilo:

- jednostavna atomska vrednost → `sync/atomic`
- složen objekat → `sync.Mutex` ili drugi odgovarajući mehanizam

---

# 9. Kako Otkriti Data Race?

Najvažniji alat:

```bash
go test -race
```

---

Primer:

```bash
go test -race ./...
```

---

Race detector može prijaviti:

- mesto čitanja
- mesto upisa
- stack trace obe goroutine
- lokaciju kreiranja goroutine-a

---

To značajno olakšava pronalaženje problema.

---

# 10. Kako Sprečiti Data Race?

Najčešće strategije:

✔ Ne deliti promenljive između goroutine-a.

---

✔ Koristiti channel za prenos vlasništva nad podacima.

---

✔ Koristiti `sync.Mutex` ili `sync.RWMutex`.

---

✔ Koristiti atomske operacije za jednostavne brojače i zastavice.

---

✔ Jasno definisati koja goroutine poseduje određeno stanje.

---

# 11. Česte Zablude

❌ "Race detector pronalazi sve concurrency greške."

Netačno.

On otkriva **data race**, ali ne i sve logičke greške, poput deadlock-a ili race condition-a bez deljenja memorije.

---

❌ "`counter++` je jedna operacija."

Netačno.

Sastoji se od više koraka (`load → add → store`).

---

❌ "`sync.Map` je uvek brži."

Netačno.

Performanse zavise od obrasca pristupa podacima.

---

# 12. Mentalni Model

Zamisli zajedničku svesku.

Dvoje ljudi istovremeno pokušava da upiše novi broj na isto mesto.

Bez dogovora:

```
Osoba A

↓

piše

↑

Osoba B
```

Rezultat može biti:

- izgubljen upis
- prepisan podatak
- nekonzistentno stanje

---

To je suština **Data Race** problema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ uslove za nastanak Data Race-a

✅ zašto `counter++` nije atomska operacija

✅ race nad pokazivačima

✅ race nad struct poljima

✅ race nad map-ama

✅ razliku između `sync/atomic` i `sync.Mutex`

✅ kako koristiti `go test -race`

---

# Common Concurrency Mistakes

## Deo #3 — Deadlock, Livelock i Starvation

---

# 📚 Sadržaj

- Šta je Deadlock?
- Channel Deadlock
- Mutex Deadlock
- Circular Wait
- Livelock
- Starvation
- Kako ih sprečiti?
- Dijagnostika

---

# 1. Šta Je Deadlock?

Deadlock nastaje kada dve ili više goroutine međusobno čekaju resurse i nijedna ne može da nastavi izvršavanje.

Vizuelno:

```
G1

↓

čeka G2

↑

↓

G2

↓

čeka G1
```

---

Rezultat:

```
niko

ne može

da nastavi.
```

---

Ako nijedna goroutine nije u stanju da napreduje, Go Runtime često prijavljuje:

```text
fatal error:
all goroutines are asleep - deadlock!
```

> **Napomena:** Ova poruka se javlja kada runtime zaključi da nijedna goroutine ne može nastaviti izvršavanje. Nije svaki deadlock nužno prijavljen ovom porukom, naročito u složenijim aplikacijama sa spoljnim događajima ili cgo kodom.

---

# 2. Channel Deadlock

Najjednostavniji primer:

```go
func main() {
	ch := make(chan int)

	ch <- 42
}
```

---

Šta se dešava?

```
Sender

↓

čeka Receiver

↓

Receiver ne postoji
```

---

Program ostaje blokiran.

---

Rešenje:

```go
go func() {
	ch <- 42
}()

fmt.Println(<-ch)
```

---

Ili koristiti baferovani kanal kada to odgovara problemu:

```go
ch := make(chan int, 1)
```

---

# 3. Deadlock Prilikom Primanja

Sličan problem:

```go
ch := make(chan int)

fmt.Println(<-ch)
```

---

Situacija:

```
Receiver

↓

čeka Sender

↓

Sender ne postoji
```

---

Rezultat:

beskonačno čekanje.

---

# 4. Deadlock sa `for range`

Primer:

```go
for v := range ch {
	fmt.Println(v)
}
```

---

Ako kanal nikada nije zatvoren:

```
Sender završio

↓

channel ostaje otvoren

↓

Receiver čeka zauvek
```

---

Ispravno:

```go
close(ch)
```

nakon što su sve vrednosti poslate.

---

# 5. Mutex Deadlock

Primer:

```go
mu.Lock()

mu.Lock()
```

---

`sync.Mutex` nije rekurzivan.

Drugi `Lock()` čeka da prvi bude otključan.

Pošto se to nikada ne dogodi:

```
Deadlock
```

---

Uvek upari:

```go
mu.Lock()
defer mu.Unlock()
```

kada je to prikladno.

---

# 6. Circular Wait

Primer:

Goroutine A:

```go
muA.Lock()
muB.Lock()
```

---

Goroutine B:

```go
muB.Lock()
muA.Lock()
```

---

Vizuelno:

```
G1

↓

muA

↓

čeka muB

↑

↓

G2

↓

muB

↓

čeka muA
```

---

Klasičan deadlock.

---

Rešenje:

Uvek zaključavaj više mutex-a istim redosledom u celom programu.

Na primer:

```
muA

↓

muB
```

nikada obrnuto.

---

# 7. Livelock

Kod livelock-a goroutine nisu blokirane.

Naprotiv:

one stalno rade.

Ali ne ostvaruju napredak.

---

Primer:

Dve goroutine pokušavaju da budu "ljubazne":

```
A se pomera

↓

B se pomera

↓

A ponovo

↓

B ponovo
```

---

Rezultat:

```
beskonačno kretanje

↓

bez završetka posla
```

---

# 8. Starvation

Starvation znači da jedna goroutine veoma dugo ili nikada ne dobije pristup resursu.

Primer:

```
Worker 1

uvek uzima posao
```

---

```
Worker 2

uvek ostaje bez posla
```

---

Ili:

```
Mutex

↓

uvek dobija ista goroutine
```

---

Posledica:

neke goroutine praktično ne napreduju.

---

# 9. Kako Sprečiti Ove Probleme?

### Za Deadlock

✔ Jasno definiši vlasništvo nad resursima.

✔ Zaključavaj više mutex-a uvek istim redosledom.

✔ Ne zaboravi `Unlock()`.

✔ Ne šalji niti primaj sa kanala bez odgovarajuće druge strane.

---

### Za Livelock

✔ Izbegavaj beskonačne pokušaje bez promene strategije.

✔ Uvedi nasumično kašnjenje (*randomized backoff*) ili drugačiji mehanizam odlučivanja kada više učesnika stalno odustaje jedan zbog drugog.

---

### Za Starvation

✔ Drži kritične sekcije kratkim.

✔ Izbegavaj nepotrebno dugo zadržavanje resursa.

✔ Ravnomerno rasporedi posao između worker-a.

---

# 10. Kako Dijagnostikovati?

Korisni alati:

```go
runtime.Stack()
```

Prikazuje gde su goroutine blokirane.

---

```go
runtime.NumGoroutine()
```

Prikazuje ukupan broj aktivnih goroutine-a.

---

Profilisanje:

```bash
go tool trace
```

---

I:

```bash
go tool pprof
```

---

Za mrežne servise može biti korisno uključiti:

```go
import _ "net/http/pprof"
```

---

# 11. Česte Zablude

❌ "Deadlock se javlja samo kod mutex-a."

Netačno.

Može nastati i zbog channel-a, `WaitGroup`-a ili drugih mehanizama sinhronizacije.

---

❌ "Livelock je isto što i Deadlock."

Netačno.

- Deadlock → ništa se ne izvršava.
- Livelock → izvršavanje postoji, ali nema korisnog napretka.

---

❌ "Starvation znači da je program stao."

Netačno.

Program može normalno raditi, dok jedna ili više goroutine gotovo nikada ne dobiju priliku za izvršavanje.

---

# 12. Mentalni Model

Zamisli uzak most.

### Deadlock

Dva automobila ulaze sa suprotnih strana.

Nijedan ne može da prođe.

---

### Livelock

Oba vozača stalno pokušavaju da se sklone jedan drugom:

```
levo

↓

desno

↓

levo

↓

desno
```

Ali i dalje blokiraju prolaz.

---

### Starvation

Jedan smer neprekidno dobija zeleno svetlo.

Drugi smer čeka veoma dugo.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je deadlock

✅ channel deadlock

✅ mutex deadlock

✅ circular wait

✅ razliku između deadlock-a i livelock-a

✅ šta je starvation

✅ kako dijagnostikovati i sprečiti ove probleme

---

# Common Concurrency Mistakes

## Deo #4 — Goroutine Leaks i Pogrešna Upotreba `context.Context`

---

# 📚 Sadržaj

- Šta je Goroutine Leak?
- Kako nastaje?
- Najčešći uzroci
- Beskonačne goroutine
- Worker leak
- Channel leak
- `context.Context`
- Cancellation
- Timeout
- Najbolje prakse

---

# 1. Šta Je Goroutine Leak?

Goroutine Leak nastaje kada goroutine više ne obavlja koristan posao, ali i dalje postoji i zauzima resurse.

Takva goroutine može ostati aktivna:

- minutima
- satima
- danima

ako ne postoji mehanizam za njen završetak.

---

Za razliku od memory leak-a:

```
goroutine

↓

i dalje postoji

↓

koristi memoriju

↓

scheduler je prati

↓

GC je ne može ukloniti
```

---

# 2. Zašto Su Goroutine Leak-ovi Opasni?

Jedna goroutine zauzima relativno malo memorije.

Ali:

```
10

↓

100

↓

1 000

↓

100 000
```

---

Veliki broj "zaboravljenih" goroutine-a može dovesti do:

- povećane potrošnje memorije
- većeg scheduler overhead-a
- većeg GC overhead-a
- degradacije performansi

---

# 3. Beskonačna Goroutine

Primer:

```go
go func() {
	for {
		work()
	}
}()
```

---

Ako ne postoji izlaz iz petlje:

```
goroutine

↓

nikada

ne završava.
```

---

Bolje:

```go
for {
	select {
	case <-ctx.Done():
		return
	default:
		work()
	}
}
```

---

# 4. Leak Zbog Channel-a

Primer:

```go
go func() {
	<-ch
}()
```

---

Ako niko nikada ne pošalje vrednost:

```
goroutine

↓

Waiting

↓

zauvek
```

---

Rešenje:

- poslati vrednost
- zatvoriti kanal ako je to odgovarajući signal završetka
- koristiti `context.Context` za otkazivanje čekanja

---

# 5. Worker Leak

Primer worker-a:

```go
for job := range jobs {
	process(job)
}
```

---

Ako:

```
jobs

nikada

nije zatvoren
```

worker:

```
čeka zauvek.
```

---

Rešenje:

Pošiljalac treba da zatvori kanal kada više neće slati nove poslove.

---

# 6. `context.Context`

`context.Context` predstavlja standardni način za:

- cancellation
- timeout
- deadline
- prenos vrednosti vezanih za zahtev (*request-scoped values*)

---

Najčešće se koristi za kontrolu životnog ciklusa operacija i goroutine-a.

---

# 7. Cancellation

Primer:

```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

---

Goroutine:

```go
select {
case <-ctx.Done():
	return
}
```

---

Kada se pozove:

```go
cancel()
```

↓

```
Done()

↓

goroutine završava.
```

---

# 8. Timeout

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	5*time.Second,
)
defer cancel()
```

---

Nakon isteka vremena:

```
ctx.Done()

↓

zatvara se
```

---

Sve goroutine koje osluškuju `ctx.Done()` mogu bezbedno da završe rad.

---

# 9. Najčešće Greške sa `context.Context`

❌ Ignorisanje `ctx.Done()`.

---

❌ Kreiranje `WithCancel()` bez poziva `cancel()` kada više nije potreban.

---

❌ Prosleđivanje `nil` umesto validnog konteksta.

---

❌ Čuvanje `Context` u struct polju za dugotrajno korišćenje.

Preporuka je da se `Context` prosleđuje kao prvi argument funkcije kojoj je potreban.

---

❌ Korišćenje `context.Background()` duboko u poslovnoj logici umesto prosleđivanja postojećeg konteksta.

---

# 10. Kako Otkriti Goroutine Leak?

Korisni alati:

```go
runtime.NumGoroutine()
```

---

Ako broj goroutine-a:

```
stalno raste

↓

bez razloga
```

postoji mogućnost curenja.

---

Dalje:

```go
runtime.Stack()
```

---

ili:

```bash
go tool pprof
```

---

Za server aplikacije često se koristi:

```go
import _ "net/http/pprof"
```

kako bi se analizovalo stanje goroutine-a u radu.

---

# 11. Najbolje Prakse

✔ Svaka goroutine treba da ima jasan izlazni uslov.

---

✔ Worker-i treba da završe rad kada više nema posla ili kada je operacija otkazana.

---

✔ Koristi `context.Context` za dugotrajne operacije.

---

✔ Pozovi `cancel()` kada više nije potreban izvedeni (`derived`) kontekst.

---

✔ Jasno definiši ko zatvara channel.

---

✔ Ne pokreći goroutine bez plana kako će biti zaustavljena.

---

# 12. Mentalni Model

Zamisli fabriku.

Radnici:

```
goroutine
```

---

Ako fabrika prestane sa radom:

radnici treba da dobiju signal:

```
zatvaramo

↓

idite kući
```

---

Ako signal nikada ne stigne:

radnici ostaju da čekaju zauvek.

To je analogija za:

```
goroutine leak.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je goroutine leak

✅ kako nastaje

✅ leak zbog channel-a

✅ worker leak

✅ ulogu `context.Context`

✅ cancellation

✅ timeout

✅ kako otkriti i sprečiti goroutine leak-ove

---

# Common Concurrency Mistakes

## Deo #5 — Najčešće Greške sa `sync` Paketom

---

# 📚 Sadržaj

- Greške sa `sync.WaitGroup`
- Greške sa `sync.Mutex`
- Greške sa `sync.RWMutex`
- Greške sa `sync.Once`
- Greške sa `sync.Cond`
- Greške sa `sync/atomic`
- Najbolje prakse

---

# 1. Greške sa `sync.WaitGroup`

`sync.WaitGroup` služi za čekanje da grupa goroutine-a završi izvršavanje.

Najčešće greške nisu u samom tipu, već u njegovoj nepravilnoj upotrebi.

---

# 2. Zaboravljen `Done()`

Primer:

```go
var wg sync.WaitGroup

wg.Add(1)

go func() {
	defer fmt.Println("finished")
	// nedostaje wg.Done()
}()

wg.Wait()
```

---

Rezultat:

```
Wait()

↓

čeka zauvek
```

---

Ispravno:

```go
go func() {
	defer wg.Done()
}()
```

---

# 3. Pozivanje `Add()` Nakon Pokretanja Goroutine

Loš primer:

```go
go worker()

wg.Add(1)
```

---

Goroutine može završiti pre nego što se brojač poveća.

To može dovesti do nekorektnog ponašanja ili panike.

---

Pravilan redosled:

```go
wg.Add(1)

go worker()
```

---

# 4. Negativan Brojač

Primer:

```go
wg.Add(1)

wg.Done()

wg.Done()
```

---

Rezultat:

```text
panic:
sync: negative WaitGroup counter
```

---

Broj poziva `Done()` mora odgovarati broju poziva `Add()`.

---

# 5. Kopiranje `WaitGroup`

`WaitGroup` se ne sme kopirati nakon prve upotrebe.

Loš primer:

```go
wg2 := wg
```

---

Obe promenljive više ne predstavljaju isto stanje sinhronizacije.

---

Preporuka:

Prosleđuj pokazivač:

```go
func worker(wg *sync.WaitGroup)
```

ili deli istu instancu.

---

# 6. Greške sa `sync.Mutex`

Najčešće greške:

❌ Zaboravljen `Unlock()`.

---

❌ Dvostruki `Lock()` iste goroutine.

---

❌ Zaključavanje velikih kritičnih sekcija.

---

Primer:

```go
mu.Lock()

time.Sleep(time.Second)

mu.Unlock()
```

---

Za to vreme:

```
ostale goroutine

↓

Waiting
```

---

Kritične sekcije treba da budu što kraće.

---

# 7. Greške sa `sync.RWMutex`

Česta zabluda:

```
RWMutex

↓

uvek je brži
```

---

Nije tačno.

Ako postoji mnogo upisa, dodatni overhead može učiniti `RWMutex` sporijim od običnog `Mutex`-a.

---

Koristi `RWMutex` kada:

- ima mnogo paralelnih čitanja
- upisi su relativno retki

---

# 8. Greške sa `sync.Once`

Primer:

```go
var once sync.Once

once.Do(initDB)
```

---

Greške:

❌ Očekivanje da se funkcija može ponovo izvršiti.

---

❌ Menjanje logike unutar `Do()` u zavisnosti od poziva.

---

Važno:

`Do()` garantuje da će funkcija biti izvršena najviše jednom po instanci `Once`.

---

# 9. Greške sa `sync.Cond`

`sync.Cond` zahteva pravilnu koordinaciju između goroutine-a.

Česta greška:

```go
cond.Wait()
```

bez provere uslova.

---

Ispravno:

```go
mu.Lock()
for !ready {
	cond.Wait()
}
mu.Unlock()
```

---

Razlog:

Nakon buđenja nije garantovano da je uslov i dalje ispunjen.

Petlja ponovo proverava stanje.

---

# 10. Greške sa `sync/atomic`

Primer:

```go
counter++
```

---

Ako se deli između goroutine-a:

```
Data Race
```

---

Bolje:

```go
atomic.AddInt64(&counter, 1)
```

---

Ali:

`sync/atomic` nije zamena za `Mutex` kada više promenljivih mora biti ažurirano kao jedna logička celina.

---

# 11. Česte Zablude

❌ "`Mutex` je spor, zato ga treba izbegavati."

Netačno.

U mnogim slučajevima `Mutex` je jednostavniji i sasvim dovoljno efikasan.

---

❌ "`RWMutex` je uvek bolji."

Netačno.

Zavisi od odnosa broja čitanja i upisa.

---

❌ "`sync.Once` mogu resetovati."

Netačno.

Za novu inicijalizaciju potrebna je nova instanca `sync.Once`.

---

❌ "`WaitGroup` mogu bezbedno kopirati."

Netačno.

To može dovesti do ozbiljnih problema u sinhronizaciji.

---

# 12. Najbolje Prakse

✔ Pozovi `Add()` pre pokretanja goroutine.

---

✔ U goroutine koristi:

```go
defer wg.Done()
```

na početku funkcije, kada je moguće.

---

✔ Kritične sekcije drži kratkim.

---

✔ Biraj između `Mutex` i `RWMutex` na osnovu merenja, a ne pretpostavki.

---

✔ Koristi `sync.Once` samo za jednokratnu inicijalizaciju.

---

✔ Koristi `sync/atomic` samo za jednostavne atomske operacije.

---

# 13. Mentalni Model

Zamisli gradilište.

- `WaitGroup` → spisak radnika koji moraju završiti posao.
- `Mutex` → ključ od jedne prostorije.
- `RWMutex` → čitaonica u kojoj više ljudi može čitati, ali samo jedna osoba može menjati sadržaj.
- `Once` → svečano presecanje vrpce koje se obavlja samo jednom.
- `Cond` → zvono koje obaveštava radnike da se stanje promenilo.
- `atomic` → brojčanik koji se može bezbedno povećati ili smanjiti jednom operacijom.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ najčešće greške sa `WaitGroup`

✅ greške sa `Mutex` i `RWMutex`

✅ pravilnu upotrebu `sync.Once`

✅ obrasce za korišćenje `sync.Cond`

✅ ograničenja `sync/atomic`

✅ preporučene prakse pri radu sa `sync` paketom

---

# Common Concurrency Mistakes

## Deo #6 — Production Best Practices i Završni Rezime

---

# 📚 Sadržaj

- Production principi
- Checklist za konkurentni kod
- Kada koristiti koji mehanizam?
- Dijagnostika
- Alati
- Najčešće preporuke
- Završni rezime

---

# 1. Production Principi

Najbolji konkurentni kod nije onaj koji koristi najviše goroutine-a.

Najbolji kod je:

- jednostavan
- predvidiv
- lako testabilan
- lako održiv

---

Najvažnije pravilo:

```
Prvo ispravnost.

↓

Tek onda performanse.
```

---

# 2. Checklist Pre Produkcije

Pre nego što aplikacija ode u produkciju proveri:

```
□ nema Data Race

□ nema Deadlock

□ nema Goroutine Leak

□ koristi Context

□ ima Timeout

□ koristi Cancellation

□ nema beskonačnih worker-a

□ nema beskonačnih retry petlji

□ svi channel-i imaju vlasnika

□ svaki Mutex se otključava

□ WaitGroup brojač je ispravan
```

---

# 3. Kada Koristiti Koji Mehanizam?

### Channel

Koristi kada želiš:

- prenos podataka
- signalizaciju
- pipeline
- worker pool

---

### `sync.Mutex`

Koristi kada više goroutine deli isto stanje.

---

### `sync.RWMutex`

Koristi kada:

```
Read

≫≫≫

Write
```

odnosno kada čitanja značajno preovlađuju nad upisima.

---

### `sync/atomic`

Koristi za:

- brojače
- zastavice (*flags*)
- jednostavne atomske vrednosti

---

### `context.Context`

Koristi za:

- cancellation
- timeout
- deadline
- vrednosti vezane za zahtev (*request-scoped values*)

---

# 4. Ne Mešaj Nepotrebno Primitive

Loš primer:

```
Mutex

+

Channel

+

Atomic

+

Cond
```

za rešavanje jednog jednostavnog problema.

---

Bolje:

izaberi jedan odgovarajući mehanizam kada je to moguće.

Jednostavniji kod je lakši za razumevanje i održavanje.

---

# 5. Testiranje Konkurentnog Koda

Redovno koristi:

```bash
go test
```

---

Za otkrivanje data race-a:

```bash
go test -race
```

---

Za benchmark:

```bash
go test -bench=.
```

---

Za merenje alokacija:

```bash
go test -benchmem
```

---

Za trace:

```bash
go test -trace trace.out
```

---

Za CPU profil:

```bash
go test -cpuprofile cpu.out
```

---

Za memorijski profil:

```bash
go test -memprofile mem.out
```

---

# 6. Posmatraj Produkciju

Pored testova, prati i ponašanje aplikacije.

Korisne metrike:

- broj goroutine-a
- latencija zahteva
- broj timeout-a
- broj otkazanih (`cancelled`) operacija
- iskorišćenost worker-a
- dužina redova (`queue length`)
- stopa grešaka

---

Ako broj goroutine-a neprekidno raste bez opravdanog razloga, to može biti znak curenja goroutine-a.

---

# 7. Najčešće Greške U Produkciji

❌ Pokretanje neograničenog broja goroutine-a.

---

❌ Ignorisanje `ctx.Done()`.

---

❌ Deljenje mapa bez zaštite.

---

❌ Pretpostavljanje redosleda izvršavanja.

---

❌ Dugo držanje `Mutex`-a.

---

❌ Nepozivanje `cancel()` za izvedene (`derived`) kontekste kada više nisu potrebni.

---

❌ Nedostatak timeout-a pri radu sa mrežom ili bazom podataka.

---

# 8. Zlatna Pravila

✔ Ne komuniciraj preko deljene memorije kada prenos vlasništva preko channel-a prirodno rešava problem.

✔ Ne deli stanje bez jasnog plana sinhronizacije.

✔ Svaka goroutine treba da ima definisan način završetka.

✔ Kritične sekcije neka budu kratke.

✔ Profiliši i meri pre optimizacije.

✔ Piši konkurentni kod koji će drugi lako razumeti.

---

# 9. Alati Koje Treba Poznavati

### Testiranje

```bash
go test
```

---

### Race Detector

```bash
go test -race
```

---

### Benchmark

```bash
go test -bench=.
```

---

### Trace

```bash
go tool trace
```

---

### Profilisanje

```bash
go tool pprof
```

---

### Runtime

- `runtime.NumGoroutine()`
- `runtime.Stack()`
- `runtime/trace`
- `runtime/pprof`
- `net/http/pprof`

---

# 10. Mentalni Model

Zamisli orkestar.

- Scheduler je dirigent.
- Goroutine su muzičari.
- Channel-i su note koje se razmenjuju.
- Mutex je ključ od prostorije sa instrumentima.
- Context je znak dirigenta da se koncert završava.
- WaitGroup je spisak svih izvođača koji moraju završiti nastup pre spuštanja zavese.

Ako jedan deo ne funkcioniše pravilno, ceo orkestar može zvučati loše, čak i kada svaki pojedinačni muzičar svira dobro.

---

# 11. Završni Rezime Modula

Tokom svih šest delova naučili smo:

✅ razliku između Race Condition i Data Race

✅ kako nastaju deadlock, livelock i starvation

✅ kako sprečiti goroutine leak-ove

✅ pravilnu upotrebu `context.Context`

✅ najčešće greške sa `sync` paketom

✅ production obrasce za konkurentni kod

✅ alate za testiranje, profilisanje i dijagnostiku

---

# 12. Ključne Lekcije

Zapamti sledeće principe:

- Goroutine su jeftine, ali nisu besplatne.
- Svaka goroutine treba da ima jasan životni ciklus.
- `go test -race` treba redovno koristiti tokom razvoja.
- `context.Context` je standardni način za otkazivanje dugotrajnih operacija.
- Nemoj pretpostavljati redosled izvršavanja goroutine-a.
- Biraj najjednostavniji mehanizam sinhronizacije koji rešava problem.
- Optimizacija bez merenja često vodi do složenijeg i manje pouzdanog koda.

---

# 🎓 Završetak Extra Modula

Čestitamo!

Završio si svih **13 Extra modula** iz oblasti konkurentnosti u Go-u.

Obrađene teme uključuju:

- napredne concurrency obrasce
- `context.Context`
- worker pool dizajn
- pipeline obrasce
- fan-in i fan-out
- scheduler internals
- tipične konkurentne greške
- production best practices

Ovi moduli predstavljaju čvrstu osnovu za razvoj visoko konkurentnih, skalabilnih i pouzdanih Go aplikacija.

---

# 🚀 Preporučeni Sledeći Koraci

Nakon ovog modula preporučuje se produbljivanje znanja kroz:

1. **Go Memory Model**
2. **Garbage Collector Internals**
3. **Escape Analysis**
4. **Memory Allocation**
5. **Runtime Source Code**
6. **Standard Library Internals**
7. **Performance Optimization**
8. **Profiling i Benchmarking u produkcionim sistemima**

---

# 📋 Završni Rezime Dokumenta

Ovaj dokument obuhvatio je:

✅ Race Condition i Data Race

✅ Deadlock, Livelock i Starvation

✅ Goroutine Leak

✅ pravilnu upotrebu `context.Context`

✅ najčešće greške sa `sync` paketom

✅ production preporuke

✅ testiranje i dijagnostiku

✅ najbolje prakse za pisanje konkurentnog Go koda

---

# 🏁 Završetak Celine "Extras"

Uspešno je završen kompletan skup dodatnih modula iz konkurentnosti.

Stečeno znanje predstavlja odličnu osnovu za razumevanje rada Go Runtime-a, projektovanje konkurentnih sistema i rešavanje problema koji se javljaju u realnim produkcionim okruženjima.