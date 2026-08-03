# Shared Memory i uvod u `sync.Mutex`

> **Modul:** #3 — Srednje
>
> **Lekcija:** 1/9 (Deo 1)
>
> **Fajl:** `docs/module-3/01-mutex.md`

> **Napomena:** Ovo je prvi deo lekcije o `sync.Mutex`. U narednim delovima ćemo postepeno graditi razumevanje sve do production primera.

---

# 📚 Sadržaj ovog dela

- Uvod
- Dva načina komunikacije između Goroutines
- Go filozofija
- Šta je Shared Memory?
- Zašto Shared Memory predstavlja problem?
- Šta je Critical Section?
- Šta je Mutual Exclusion?
- Šta je `sync.Mutex`?
- Prvi intuitivni primer
- Vizuelni dijagrami

---

# 🎯 Cilj lekcije

Nakon ovog dela znaćeš:

- šta predstavlja **Shared Memory**,
- zašto više Goroutines može napraviti problem,
- šta je **Critical Section**,
- zašto je potreban **Mutex**,
- zbog čega Go uopšte poseduje `sync.Mutex`.

---

# Uvod

Do sada smo učili kako Goroutines komuniciraju preko **Channel-a**.

To je idiomatski način rada u Go-u.

Na primer:

```text
Goroutine A

↓

Channel

↓

Goroutine B
```

Ovde se podaci prenose.

Nema deljenja iste promenljive.

---

# Go filozofija

Jedna od najpoznatijih rečenica iz Go dokumentacije glasi:

> **Don't communicate by sharing memory; share memory by communicating.**

U slobodnom prevodu:

> **Nemoj komunicirati tako što više Goroutines deli istu memoriju. Umesto toga, razmenjuj podatke preko Channel-a.**

---

# Šta ova rečenica znači?

Pretpostavimo da dve Goroutines koriste istu promenljivu.

```go
counter++
```

Obe žele da je promene.

Obe pristupaju istoj memorijskoj lokaciji.

To nazivamo:

> **Shared Memory**

---

# Šta je Shared Memory?

Shared Memory znači:

> Više Goroutines može da pristupi istoj promenljivoj u memoriji.

Na primer:

```go
var counter int
```

Imamo:

```text
        counter

           ▲

      ┌────┴────┐

      │         │

 Goroutine1  Goroutine2
```

Obe koriste isti objekat.

---

# Da li je Shared Memory loša?

Ne.

Sama po sebi nije problem.

Problem nastaje kada:

- jedna Goroutine piše,
- druga piše,
- ili jedna čita dok druga piše.

---

# Primer

```go
var balance = 100
```

Jedna Goroutine:

```go
balance += 50
```

Druga:

```go
balance -= 20
```

Obe rade istovremeno.

Da li će rezultat biti:

```
130
```

?

Možda.

Možda neće.

---

# Zašto?

Jer izvršavanje nije sekvencijalno.

Go Scheduler odlučuje:

- koja Goroutine radi,
- koliko dugo radi,
- kada će biti prekinuta.

---

# Vizuelni prikaz

```text
Goroutine A

↓

balance += 50

------------

Goroutine B

↓

balance -= 20
```

Njihovo izvršavanje može biti isprepletano.

---

# Šta znači "istovremeno"?

Ne mora značiti da rade na dva različita CPU jezgra.

Dovoljno je da Scheduler prebacuje izvršavanje između njih.

Na primer:

```text
A

↓

B

↓

A

↓

A

↓

B
```

ili

```text
B

↓

A

↓

B

↓

A
```

Redosled nije garantovan.

---

# Jedan važan detalj

Mnogi početnici zamišljaju:

```go
counter++
```

kao jednu operaciju.

U stvarnosti to nije jedna operacija.

Konceptualno izgleda ovako:

```text
1.

Učitaj vrednost

↓

2.

Povećaj

↓

3.

Upiši nazad
```

Između ovih koraka Scheduler može promeniti aktivnu Goroutine.

---

# Primer

Pretpostavimo:

```go
counter = 5
```

Goroutine A:

```text
učitaj 5
```

Pre nego što upiše:

Scheduler prebacuje izvršavanje.

---

Goroutine B:

```text
učitaj 5

↓

uvećaj

↓

upiši 6
```

Scheduler ponovo vraća Goroutine A.

Ona nastavlja:

```text
uvećaj

↓

upiši 6
```

Konačna vrednost:

```
6
```

Očekivali smo:

```
7
```

Jedno povećanje je izgubljeno.

---

# Vizuelni prikaz

```text
Counter = 5

↓

A čita 5

↓

B čita 5

↓

B upisuje 6

↓

A upisuje 6
```

Rezultat:

```text
6
```

Umesto:

```text
7
```

---

# Zašto se ovo dešava?

Zato što obe Goroutines menjaju istu memoriju.

Bez koordinacije.

---

# Critical Section

Dolazimo do veoma važnog pojma.

> **Critical Section**

---

# Šta je Critical Section?

Critical Section je deo koda koji koristi deljeni resurs.

Na primer:

```go
counter++
```

ili

```go
users[id] = user
```

ili

```go
balance += amount
```

Sve ovo predstavlja Critical Section.

---

# Zašto "Critical"?

Zato što istovremeni pristup može dovesti do pogrešnih rezultata.

---

# Vizuelni prikaz

```text
Critical Section

┌────────────────────┐

counter++

└────────────────────┘
```

Idealno:

u jednom trenutku

↓

jedna Goroutine.

---

# Šta želimo?

Želimo sledeće pravilo.

```text
Ako je jedna Goroutine
ušla u Critical Section,

nijedna druga
ne sme ući.
```

---

# Mutual Exclusion

Ovo pravilo ima ime.

> **Mutual Exclusion**

Skraćeno:

> **Mutex**

---

# Šta znači Mutual Exclusion?

Doslovno:

> međusobno isključivanje.

Drugim rečima:

Ako jedna Goroutine koristi kritični deo koda,

ostale moraju da sačekaju.

---

# Vizuelni prikaz

```text
          Lock

            │

            ▼

      Critical Section

            ▲

            │

         Unlock
```

---

# Šta je `sync.Mutex`?

`sync.Mutex` je tip iz standardnog paketa:

```go
sync
```

Njegova jedina svrha je:

> da obezbedi da samo jedna Goroutine u jednom trenutku koristi Critical Section.

---

# Kako radi?

Pre ulaska:

```go
Lock()
```

Po izlasku:

```go
Unlock()
```

Vizuelno:

```text
Lock

↓

Critical Section

↓

Unlock
```

---

# Intuitivna analogija

Zamisli jedno kupatilo.

Postoji samo jedan ključ.

```text
Ključ

↓

Kupatilo
```

Ako ga jedna osoba uzme,

ostali čekaju.

Tek kada vrati ključ,

sledeća osoba može da uđe.

`Mutex` je upravo taj ključ.

---

# Još jedna analogija

Jedna kasa u prodavnici.

```text
Kupac A

↓

Kasa

Kupac B čeka

Kupac C čeka
```

Samo jedan kupac može biti na kasi u datom trenutku.

---

# Da li Mutex ubrzava program?

Ne.

Ovo je česta zabluda.

Mutex ne postoji zbog brzine.

Postoji zbog:

> **ispravnosti programa.**

Ako pritom performanse ostanu dobre — odlično.

Ali primarni cilj je tačnost.

---

# Kada nam Mutex uopšte treba?

Kada više Goroutines:

- menja isti `int`,
- koristi istu `map`,
- koristi isti `slice`,
- pristupa istoj strukturi,
- menja stanje istog objekta.

Ako nema deljenog stanja, često nema potrebe za `Mutex`-om.

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Shared Memory,
- zašto više Goroutines može izazvati problem,
- šta predstavlja Critical Section,
- šta znači Mutual Exclusion,
- zbog čega postoji `sync.Mutex`,
- da `Mutex` štiti ispravnost programa, a ne služi prvenstveno za povećanje performansi.

---

---

# Prvi problem bez `sync.Mutex`

Do sada smo razumeli teoriju.

Sada ćemo videti kako problem izgleda u stvarnom Go programu.

---

# Primer

```go
package main

import (
	"fmt"
	"sync"
)

func main() {

	var (
		counter int
		wg sync.WaitGroup
	)

	for i := 0; i < 1000; i++ {

		wg.Add(1)

		go func() {

			defer wg.Done()

			counter++

		}()

	}

	wg.Wait()

	fmt.Println(counter)

}
```

---

# Šta očekujemo?

Pokrećemo:

```
1000
```

Goroutines.

Svaka izvršava:

```go
counter++
```

Logično očekivanje je:

```text
1000
```

---

# Da li će rezultat uvek biti 1000?

Ne.

Možemo dobiti:

```text
998
```

ili

```text
995
```

ili

```text
991
```

ili čak neku još manju vrednost.

Na nekim pokretanjima može se pojaviti i:

```text
1000
```

To ne znači da je program ispravan.

To samo znači da se problem nije manifestovao u tom izvršavanju.

---

# Zašto?

Zato što:

```go
counter++
```

nije atomska operacija.

Sastoji se od više koraka.

---

# Konceptualni prikaz

```text
1.

Read

↓

2.

Increment

↓

3.

Write
```

Tri odvojene operacije.

---

# Šta Scheduler može da uradi?

Pretpostavimo:

```text
counter = 10
```

---

## Goroutine A

Korak 1

```text
Read

↓

10
```

Scheduler prekida izvršavanje.

---

## Goroutine B

Korak 1

```text
Read

↓

10
```

Korak 2

```text
Increment

↓

11
```

Korak 3

```text
Write

↓

11
```

Scheduler vraća Goroutine A.

---

## Goroutine A nastavlja

Korak 2

```text
Increment

↓

11
```

Korak 3

```text
Write

↓

11
```

---

# Konačan rezultat

Umesto:

```text
12
```

dobijamo:

```text
11
```

Jedno povećanje je izgubljeno.

---

# Vizuelni prikaz

```text
Counter

10

│

├── Goroutine A čita 10

│

├── Scheduler

│

├── Goroutine B čita 10

├── Goroutine B upisuje 11

│

├── Scheduler

│

└── Goroutine A upisuje 11
```

---

# Ovo nazivamo...

> **Lost Update**

Jedna izmena je prebrisala drugu.

---

# Da li je `WaitGroup` kriv?

Ne.

Ovo je veoma česta greška početnika.

`WaitGroup` radi tačno ono za šta je namenjen.

On čeka da:

```
1000
```

Goroutines završi rad.

Ništa više.

Ne štiti podatke.

---

# Šta `WaitGroup` garantuje?

Garantuje:

```text
Sve Goroutines su završile.
```

Ne garantuje:

```text
Podaci su ispravni.
```

To su dve potpuno različite stvari.

---

# Još jedan primer

```go
var balance = 100
```

Dve Goroutines.

Prva:

```go
balance += 50
```

Druga:

```go
balance -= 20
```

Očekujemo:

```text
130
```

?

Ne nužno.

Možemo dobiti i druge rezultate.

---

# Zašto?

Obe menjaju istu memoriju.

Bez ikakve koordinacije.

---

# Shared Resource

Sve što više Goroutines koristi predstavlja:

> **Shared Resource**

Na primer:

- `int`
- `map`
- `slice`
- `struct`
- cache
- konfiguracija
- brojač
- stanje konekcije

---

# Da li je čitanje problem?

Ako više Goroutines samo čita istu promenljivu,

najčešće nema problema.

Na primer:

```go
fmt.Println(config.Port)
```

Ako niko ne menja:

```go
config.Port
```

onda je paralelno čitanje bezbedno.

Problem nastaje kada postoji upis.

---

# Read + Write

Jedna Goroutine:

```go
counter++
```

Druga:

```go
fmt.Println(counter)
```

Sada jedna piše,

druga čita.

Rezultat može biti nepredvidiv.

---

# Write + Write

Još gore.

```go
counter++
```

i

```go
counter--
```

Obe menjaju stanje.

---

# Zašto se problem ne pojavljuje svaki put?

Scheduler nije deterministički.

Ne postoji unapred definisan raspored izvršavanja.

Na primer:

Pokretanje broj 1

```text
1000
```

Pokretanje broj 2

```text
997
```

Pokretanje broj 3

```text
992
```

Pokretanje broj 4

```text
999
```

Isti program.

Različiti rezultati.

---

# Šta znači "nedeterministički"?

To znači da:

> Ne možemo pouzdano predvideti redosled izvršavanja Goroutines.

Zbog toga su concurrency bagovi često teški za reprodukciju.

---

# Zašto su ovakvi bagovi opasni?

Zamisli bankarski sistem.

Dve uplate stižu istovremeno.

Ako jedna "nestane",

stanje računa postaje pogrešno.

---

# Ili internet prodavnicu

Poslednji proizvod na stanju.

Dva kupca ga kupuju u isto vreme.

Oba zahteva prođu.

Na kraju imamo:

```text
Stanje:

-1
```

To je logička greška izazvana konkurentnim pristupom.

---

# Kako rešiti problem?

Potreban nam je mehanizam koji kaže:

```text
Sačekaj.

Neko već koristi ovu promenljivu.
```

To je upravo posao:

```go
sync.Mutex
```

---

# Ideja Mutex-a

Bez Mutex-a:

```text
G1

↓

Counter

↑

G2
```

Obe Goroutines pristupaju istovremeno.

---

Sa Mutex-om:

```text
G1

↓

Lock()

↓

Counter

↓

Unlock()

↓

G2 ulazi tek sada
```

U jednom trenutku,

samo jedna Goroutine koristi kritični deo koda.

---

# Da li Mutex rešava sve concurrency probleme?

Ne.

On rešava samo jedan veoma važan problem:

> Bezbedan pristup deljenoj memoriji.

Ne rešava:

- Deadlock
- Livelock
- Starvation
- Pogrešan dizajn programa
- Lošu arhitekturu

Njih ćemo obraditi u narednim modulima.

---

# Mentalni model

Zapamti sledeće pravilo.

Ako vidiš:

```text
više Goroutines

+

ista promenljiva

+

bar jedna piše
```

odmah razmišljaj:

> **Da li mi je potreban Mutex ili neki drugi mehanizam sinhronizacije?**

To je jedno od najvažnijih pitanja u Go Concurrency-ju.

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto `counter++` nije atomska operacija,
- kako nastaje **Lost Update** problem,
- zašto `sync.WaitGroup` ne štiti podatke,
- šta predstavlja **Shared Resource**,
- zašto su concurrency bagovi nedeterministički,
- zašto je potreban `sync.Mutex`.

---

---

# Prvi program sa `sync.Mutex`

U prethodnom delu videli smo da sledeći program nije bezbedan:

```go
counter++
```

Razlog je što više Goroutines istovremeno menja istu promenljivu.

Sada ćemo rešiti taj problem pomoću `sync.Mutex`.

---

# Uvoz paketa

`Mutex` se nalazi u standardnom paketu:

```go
import "sync"
```

---

# Deklaracija Mutex-a

Najjednostavniji način je:

```go
var mu sync.Mutex
```

ili zajedno sa promenljivom koju štiti:

```go
var (
	counter int
	mu      sync.Mutex
)
```

---

# Šta predstavlja `mu`?

Možeš ga zamisliti kao:

```text
Ključ

ili

Bravu
```

Ko poseduje ključ,

jedini može da uđe u Critical Section.

---

# Prvi ispravan primer

```go
package main

import (
	"fmt"
	"sync"
)

func main() {

	var (
		counter int
		mu      sync.Mutex
		wg      sync.WaitGroup
	)

	for i := 0; i < 1000; i++ {

		wg.Add(1)

		go func() {

			defer wg.Done()

			mu.Lock()

			counter++

			mu.Unlock()

		}()

	}

	wg.Wait()

	fmt.Println(counter)

}
```

---

# Očekivani rezultat

Sada će rezultat biti:

```text
1000
```

pri svakom pokretanju programa.

---

# Šta se promenilo?

Dodali smo samo dve linije:

```go
mu.Lock()
```

i

```go
mu.Unlock()
```

Sve ostalo je ostalo isto.

---

# Šta radi `Lock()`?

Poziv:

```go
mu.Lock()
```

znači:

> "Želim ekskluzivan pristup Critical Section-u."

Ako niko drugi ne koristi Mutex,

Goroutine odmah nastavlja izvršavanje.

---

# Šta ako je Mutex zauzet?

Pretpostavimo:

```text
Goroutine A

↓

Lock()
```

Dok je još u Critical Section-u,

dolazi:

```text
Goroutine B

↓

Lock()
```

Šta se događa?

---

# Odgovor

Goroutine B ne može da nastavi.

Ona čeka.

Vizuelno:

```text
           Mutex

        +---------+

G1 ---> | LOCKED  |

G2 ---> | WAITING |

G3 ---> | WAITING |

        +---------+
```

---

# Šta radi `Unlock()`?

Kada Goroutine završi posao:

```go
mu.Unlock()
```

Mutex postaje slobodan.

Jedna od čekajućih Goroutines može da nastavi.

---

# Životni ciklus

```text
Lock()

↓

Critical Section

↓

Unlock()
```

To je osnovni obrazac korišćenja `Mutex`-a.

---

# Korak po korak analiza

Pretpostavimo:

```go
counter = 0
```

Pokreću se dve Goroutines.

---

## Goroutine A

```go
mu.Lock()
```

Mutex je slobodan.

A ulazi.

---

Stanje:

```text
Counter = 0

Mutex = Locked
```

---

A izvršava:

```go
counter++
```

Rezultat:

```text
Counter = 1
```

---

Za kraj:

```go
mu.Unlock()
```

Mutex postaje slobodan.

---

## Goroutine B

U međuvremenu je pozvala:

```go
mu.Lock()
```

Ali je čekala.

Čim A pozove:

```go
Unlock()
```

B ulazi.

---

B izvršava:

```go
counter++
```

Rezultat:

```text
Counter = 2
```

---

Na kraju:

```go
mu.Unlock()
```

---

# Vizuelni prikaz

```text
Goroutine A

↓

Lock

↓

counter++

↓

Unlock

----------------------

Goroutine B

↓

čeka

↓

Lock

↓

counter++

↓

Unlock
```

---

# Šta je zaštićeno?

Samo kod između:

```go
Lock()
```

i

```go
Unlock()
```

To nazivamo:

> **Critical Section**

---

# Šta treba zaključavati?

Samo ono što zaista koristi deljeni resurs.

Na primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

Nije potrebno zaključati ostatak programa.

---

# Što je Critical Section manja...

...to je bolje.

Primer:

```go
mu.Lock()

counter++

mu.Unlock()

fmt.Println("Done")
```

Ovde:

```go
fmt.Println()
```

nije deo Critical Section-a.

Ne treba ga zaključavati.

---

# Loš primer

```go
mu.Lock()

counter++

time.Sleep(time.Second)

fmt.Println(counter)

mu.Unlock()
```

Ovde Mutex ostaje zaključan:

```
1 sekundu
```

Sve ostale Goroutines čekaju.

---

# Bolji primer

```go
mu.Lock()

counter++

value := counter

mu.Unlock()

time.Sleep(time.Second)

fmt.Println(value)
```

Sada je zaključavanje mnogo kraće.

To poboljšava propusnost programa.

---

# Jedan Mutex može štititi više promenljivih

Na primer:

```go
type BankAccount struct {

	mu sync.Mutex

	balance int

	transactions int

}
```

Jedan Mutex štiti oba polja.

---

# Ili svako polje može imati svoj Mutex

```go
type Cache struct {

	dataMu sync.Mutex

	statsMu sync.Mutex

}
```

O ovome ćemo detaljnije govoriti kada budemo obrađivali **Fine-grained** i **Coarse-grained Locking**.

---

# Da li treba imati jedan globalni Mutex?

Najčešće:

Ne.

Bolje je da svaki objekat štiti sopstveno stanje.

Na primer:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

Na taj način odgovornost ostaje lokalizovana.

---

# Mentalni model

Zamisli biblioteku.

Postoji jedna retka knjiga.

Samo jedna osoba sme da je koristi u čitaonici.

```text
Čitalac A

↓

Knjiga

↓

Vrati knjigu

↓

Čitalac B
```

`Mutex` obezbeđuje upravo takvo ponašanje.

---

# Šta se dešava unutar Runtime-a?

Kada Goroutine pozove:

```go
mu.Lock()
```

i Mutex je zauzet,

Runtime neće dozvoliti ulazak u Critical Section.

Goroutine prelazi u stanje čekanja.

Kasnije Scheduler može da izvršava druge spremne Goroutines.

Kada se pozove:

```go
mu.Unlock()
```

jedna od čekajućih Goroutines dobija priliku da nastavi izvršavanje.

Važno je razumeti da **nije garantovano** koja će tačno čekajuća Goroutine biti sledeća.

---

# Da li `Mutex` garantuje redosled?

Ne.

Na primer, ako čekaju:

```text
G2

G3

G4
```

ne treba pretpostaviti da će se probuditi baš tim redom.

Redosled nije deo API ugovora.

Program ne sme da zavisi od toga.

---

# Channel ili Mutex?

Početnici često pitaju:

> "Da li treba uvek koristiti Channel umesto Mutex-a?"

Odgovor je:

Ne.

Go preporučuje komunikaciju preko Channel-a kada je to prirodno rešenje.

Ali kada više Goroutines deli stanje koje treba zaštititi, `sync.Mutex` je često jednostavnije, efikasnije i idiomatsko rešenje.

Pravilo nije:

> Channel **ili** Mutex.

Već:

> Koristi alat koji najbolje odgovara problemu.

---

# 🚨 Najčešće greške

## 1. Zaboravljen `Unlock()`

```go
mu.Lock()

counter++
```

Ako nema:

```go
mu.Unlock()
```

ostale Goroutines mogu zauvek čekati.

---

## 2. Zaključavanje prevelikog dela koda

Što je duže zaključavanje,

to je veće čekanje drugih Goroutines.

---

## 3. Više različitih Critical Section-a pod jednim velikim Mutex-om

Program radi,

ali često gubi na performansama.

Kasnije ćemo videti kako se ovo rešava finijim zaključavanjem.

---

# ✅ Best Practices

- Zaključavaj samo ono što mora biti zaštićeno.
- Critical Section neka bude što kraća.
- Drži `Mutex` što bliže podacima koje štiti.
- Ne oslanjaj se na redosled buđenja Goroutines.
- Razmišljaj o ispravnosti programa pre optimizacije.

---

# 📋 Rezime

U ovom delu naučili smo:

- kako se koristi `sync.Mutex`,
- šta rade `Lock()` i `Unlock()`,
- kako `Mutex` sprečava istovremeni pristup Critical Section-u,
- zašto zaključavanje treba da bude kratko,
- kako Runtime blokira Goroutines koje pokušaju da zaključaju zauzet `Mutex`,
- kada je `Mutex` prikladnije rešenje od Channel-a.

---

# Praktična upotreba `sync.Mutex` — `defer Unlock()` i zaštita deljenih struktura

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 1/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/01-mutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto koristiti `defer mu.Unlock()`
- Razlika između ručnog i odloženog Unlock-a
- `defer` i bezbednost koda
- Šta se dešava kod `panic`
- Zaštita `map` pomoću `Mutex`-a
- Zašto Go `map` nije concurrency-safe
- Implementacija bezbednog pristupa `map`-i
- `ConcurrentMap` obrazac
- Najčešće greške

---

# Uvod

U prethodnom delu naučili smo osnovni obrazac:

```go
mu.Lock()

// Critical Section

mu.Unlock()
```

Ovo funkcioniše.

Međutim, u realnom kodu postoji problem.

Šta ako se između:

```go
Lock()
```

i:

```go
Unlock()
```

dogodi greška?

---

# Problem sa ručnim `Unlock()`

Pogledajmo primer:

```go
func increment() {

	mu.Lock()

	counter++

	doSomething()

	mu.Unlock()

}
```

Na prvi pogled izgleda ispravno.

Ali šta ako:

```go
doSomething()
```

izazove:

```text
panic
```

?

---

# Izvršavanje

```text
Lock()

↓

counter++

↓

doSomething()

↓

PANIC
```

Program prekida tok.

Linija:

```go
mu.Unlock()
```

nikada se ne izvršava.

---

# Posledica

Mutex ostaje zaključan:

```text
Mutex = Locked
```

Sve druge Goroutines koje pokušaju:

```go
mu.Lock()
```

će čekati zauvek.

---

# Rezultat

Dobijamo:

> **Deadlock**

kasnije ćemo ga detaljno obraditi.

---

# Rešenje

Go idiomatski koristi:

```go
defer
```

---

# `defer mu.Unlock()`

Primer:

```go
func increment() {

	mu.Lock()

	defer mu.Unlock()

	counter++

	doSomething()

}
```

---

# Šta se ovde dešava?

Redosled:

```text
Lock()

↓

Registruj Unlock()

↓

Izvrši ostatak funkcije

↓

Automatski Unlock()
```

---

# Važno pravilo

`defer` funkcije se izvršavaju:

> kada trenutna funkcija završi.

Bez obzira kako završi:

- normalan return,
- greška,
- panic.

---

# Primer

```go
func example() {

	defer fmt.Println("Kraj")

	fmt.Println("Početak")

}
```

Rezultat:

```text
Početak

Kraj
```

---

# Sa Mutex-om

```go
func update() {

	mu.Lock()

	defer mu.Unlock()

	data++

}
```

Možemo biti sigurni:

```go
Unlock()
```

će biti pozvan.

---

# Ručni Unlock vs defer Unlock

## Ručni način

```go
mu.Lock()

counter++

mu.Unlock()
```

Prednost:

- malo brži,
- nema dodatnog `defer` poziva.

Mana:

- lako se zaboravi,
- manje bezbedan kod kompleksnih funkcija.

---

## `defer` način

```go
mu.Lock()

defer mu.Unlock()

counter++
```

Prednosti:

- sigurniji,
- čitljiviji,
- standardni Go stil.

Mana:

- minimalan overhead.

---

# Da li je overhead važan?

U većini aplikacija:

Ne.

Razlika je veoma mala.

Mnogo veći problem je:

- bug zbog zaboravljenog `Unlock()`,
- deadlock,
- teško održavanje koda.

---

# Production pravilo

U većini slučajeva:

```go
mu.Lock()

defer mu.Unlock()
```

je preporučeni obrazac.

---

# Zaštita `map` pomoću Mutex-a

Sada prelazimo na veoma čest slučaj.

Go `map`.

---

# Problem

Imamo:

```go
users := make(map[string]int)
```

Jedna Goroutine:

```go
users["Marko"] = 25
```

Druga:

```go
users["Ana"] = 30
```

Obe pišu istovremeno.

---

# Da li je bezbedno?

Ne.

Go mapa nije concurrency-safe za konkurentni upis.

---

# Primer problema

```go
package main

import (
	"sync"
)

func main() {

	users := make(map[string]int)

	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {

		wg.Add(1)

		go func(id int) {

			defer wg.Done()

			users["user"] = id

		}(i)

	}

	wg.Wait()

}
```

---

# Mogući rezultat

Program može završiti sa:

```text
fatal error:

concurrent map writes
```

---

# Zašto?

Interna struktura mape se menja.

Go runtime ne može bez zaštite da dozvoli:

```text
Goroutine A

↓

menja bucket

----------------

Goroutine B

↓

menja isti bucket
```

---

# Rešenje

Dodajemo Mutex.

---

# ConcurrentMap obrazac

```go
type SafeMap struct {

	mu sync.Mutex

	data map[string]int

}
```

---

# Inicijalizacija

```go
m := SafeMap{

	data: make(map[string]int),

}
```

---

# Bezbedan Write

```go
func (m *SafeMap) Set(
	key string,
	value int,
) {

	m.mu.Lock()

	defer m.mu.Unlock()

	m.data[key] = value

}
```

---

# Bezbedan Read

```go
func (m *SafeMap) Get(
	key string,
) int {

	m.mu.Lock()

	defer m.mu.Unlock()

	return m.data[key]

}
```

---

# Zašto i Read zaključavamo?

Zato što obična mapa nije bezbedna ni kada se čita dok neko drugi piše.

Problem nije samo:

```text
Write + Write
```

nego i:

```text
Read + Write
```

---

# Vizuelno

Bez Mutex-a:

```text
          map

        ▲     ▲

        │     │

       G1    G2
```

Obe Goroutines pristupaju direktno.

---

Sa Mutex-om:

```text
          Mutex

            │

            ▼

           map

            ▲

            │

        samo jedna
```

---

# Poboljšanje

Možemo koristiti:

```go
sync.RWMutex
```

kada imamo mnogo čitanja.

Njega ćemo detaljno obraditi u sledećoj lekciji.

---

# Alternativno rešenje

Go ima:

```go
sync.Map
```

koji je specijalizovana concurrency-safe mapa.

Ali:

Ne treba automatski koristiti `sync.Map`.

Običan:

```go
map + Mutex
```

je često:

- jednostavniji,
- čitljiviji,
- brži.

---

# Kada koristiti `sync.Map`?

Tipični slučajevi:

- mnogo read operacija,
- malo write operacija,
- ključevi se dodaju jednom i uglavnom čitaju.

Za većinu poslovnog koda:

```go
map + Mutex
```

je bolji izbor.

---

# Najčešće greške

## ❌ Zaboravljen Unlock

```go
mu.Lock()

data++

```

---

## ❌ Unlock na pogrešnom mestu

```go
mu.Lock()

mu.Unlock()

data++
```

Ovde podatak više nije zaštićen.

---

## ❌ Zaključavanje samo Write operacije

```go
mu.Lock()

map[key] = value

mu.Unlock()
```

ali:

```go
value := map[key]
```

bez zaštite.

To može izazvati problem.

---

## ❌ Kopiranje strukture koja sadrži Mutex

Loše:

```go
copy := original
```

Ako struktura sadrži:

```go
sync.Mutex
```

ne treba je kopirati.

---

# Mentalni model

Zapamti:

```text
Shared Data

+

više Goroutines

+

upis

=

potreban synchronization mechanism
```

Najčešće:

```go
sync.Mutex
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto koristiti `defer mu.Unlock()`,
- kako `defer` sprečava ostavljanje Mutex-a zaključanim,
- zašto je `map` problematična u konkurentnom okruženju,
- kako zaštititi `map` pomoću `Mutex`-a,
- kako izgleda `ConcurrentMap` obrazac,
- kada razmišljati o `sync.Map`.

---

# Praktična upotreba `sync.Mutex` — zaštita `slice`-a i dizajn struktura sa Mutex-om

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 1/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/01-mutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto `slice` nije concurrency-safe
- Problem konkurentnog pristupa `slice`-u
- Zaštita `slice`-a pomoću `Mutex`-a
- Dizajn strukture koja poseduje svoj `Mutex`
- Encapsulation i zaštita internog stanja
- Pointer receiver vs value receiver
- Zašto se `Mutex` ne kopira
- Embedded `Mutex`
- Best Practices

---

# Zaštita `slice`-a pomoću `sync.Mutex`

U prethodnom delu videli smo da `map` nije bezbedna za konkurentni upis.

Ista ideja važi i za:

```go
slice
```

---

# Primer

Imamo:

```go
numbers := []int{}
```

Više Goroutines dodaje elemente:

```go
numbers = append(numbers, value)
```

---

# Problem

Na prvi pogled deluje jednostavno:

```go
numbers = append(numbers, 10)
```

Ali `append` nije samo:

```text
dodaj element
```

Interno može uključivati:

1. proveru kapaciteta,
2. alokaciju novog niza,
3. kopiranje postojećih elemenata,
4. promenu pokazivača,
5. promenu dužine.

---

# Vizuelno

Pre:

```text
slice

len = 3
cap = 4

[1][2][3][ ]
```

Dodavanje:

```go
append(slice, 4)
```

može zahtevati:

```text
alokaciju

↓

kopiranje

↓

promenu strukture slice-a
```

---

# Šta ako dve Goroutines rade isto?

Goroutine A:

```go
append(numbers, 10)
```

Goroutine B:

```go
append(numbers, 20)
```

---

# Mogući problem

Obe vide:

```text
len = 3
```

Obe pokušavaju da dodaju element.

Rezultat može biti:

```text
[1][2][3][10]
```

umesto:

```text
[1][2][3][10][20]
```

Jedna promena je izgubljena.

---

# Nezaštićen primer

```go
package main

import (
	"sync"
)

func main() {

	var numbers []int

	var wg sync.WaitGroup

	for i := 0; i < 1000; i++ {

		wg.Add(1)

		go func(value int) {

			defer wg.Done()

			numbers = append(numbers, value)

		}(i)

	}

	wg.Wait()

}
```

---

# Problem

Ovaj kod nema garantovan rezultat.

Može:

- izgubiti elemente,
- izazvati race condition,
- dati neočekivano stanje.

---

# Rešenje

Koristimo:

```go
Mutex
```

---

# Zaštićen primer

```go
package main

import (
	"sync"
)

func main() {

	var (
		numbers []int
		mu sync.Mutex
		wg sync.WaitGroup
	)

	for i := 0; i < 1000; i++ {

		wg.Add(1)

		go func(value int) {

			defer wg.Done()

			mu.Lock()

			numbers = append(numbers, value)

			mu.Unlock()

		}(i)

	}

	wg.Wait()

}
```

---

# Još bolji stil

Koristimo:

```go
defer
```

---

```go
mu.Lock()

defer mu.Unlock()

numbers = append(numbers, value)
```

---

# Zašto je ovo bolje?

Ako kasnije dodamo:

```go
validate()

calculate()

update()
```

ne moramo ručno da proveravamo svaki izlaz iz funkcije.

---

# Dizajn strukture sa sopstvenim Mutex-om

U realnim aplikacijama ne želimo:

```go
globalni Mutex
```

za sve podatke.

Bolje je:

> Objekat poseduje svoj Mutex i štiti svoje stanje.

---

# Primer

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

# Ovaj objekat sada sam kontroliše pristup:

```text
Counter

├── value

└── Mutex
```

---

# Metoda za povećavanje

```go
func (c *Counter) Increment() {

	c.mu.Lock()

	defer c.mu.Unlock()

	c.value++

}
```

---

# Metoda za čitanje

```go
func (c *Counter) Value() int {

	c.mu.Lock()

	defer c.mu.Unlock()

	return c.value

}
```

---

# Zašto je ovo dobar dizajn?

Zato što korisnik objekta ne mora znati:

- da postoji Mutex,
- kada treba zaključati,
- kada otključati.

On samo koristi:

```go
counter.Increment()
```

---

# Loš dizajn

```go
counter.mu.Lock()

counter.value++

counter.mu.Unlock()
```

Zašto?

Zato što spoljašnji kod sada zna internu implementaciju.

Krši se:

> Encapsulation

---

# Encapsulation

Dobra praksa:

```text
Objekat

↓

štiti svoje stanje
```

Ne:

```text
Spoljašnji kod

↓

menja unutrašnje stanje direktno
```

---

# Pointer receiver vs value receiver

Veoma važna tema.

Pogledaj:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

# Pogrešno

```go
func (c Counter) Increment() {

	c.mu.Lock()

	defer c.mu.Unlock()

	c.value++

}
```

---

# Zašto?

Zato što:

```go
Counter
```

biva kopiran.

Dobijamo:

```text
Original

Counter
 |
 ├── Mutex
 └── value


Kopija

Counter
 |
 ├── drugi Mutex
 └── druga vrednost
```

---

# Posledica

Metoda zaključava kopirani Mutex.

Ne originalni.

---

# Ispravno

Koristimo:

```go
pointer receiver
```

---

```go
func (c *Counter) Increment() {

	c.mu.Lock()

	defer c.mu.Unlock()

	c.value++

}
```

---

# Zašto?

Sada metoda radi nad originalnim objektom:

```text
Counter

↓

isti Mutex

↓

isto stanje
```

---

# Pravilo

Ako struktura sadrži:

```go
sync.Mutex
```

najčešće koristi:

```go
pointer receiver
```

---

# Zašto se Mutex ne sme kopirati?

`sync.Mutex` ima interno stanje.

On zna:

- da li je zaključan,
- ko čeka,
- runtime informacije.

Kopiranjem dobijamo:

```text
Mutex A

i

Mutex B
```

koji više nisu povezani.

---

# Primer problema

```go
type Store struct {

	mu sync.Mutex

	data map[string]string

}
```

Ako uradimo:

```go
copy := original
```

dobili smo:

```text
original.mu

copy.mu
```

Dve različite brave.

---

# Go upozorenje

Go dokumentacija jasno kaže:

> A Mutex must not be copied after first use.

Drugim rečima:

Nakon što je korišćen,

ne sme se kopirati.

---

# Embedded Mutex

Možemo napisati:

```go
type Cache struct {

	sync.Mutex

	data map[string]string

}
```

---

# Tada možemo:

```go
cache.Lock()

cache.data["key"] = "value"

cache.Unlock()
```

---

# Da li je preporučljivo?

Zavisi.

Prednost:

- kraći kod.

Mana:

- Mutex postaje deo javnog API-ja.

Često je eksplicitno bolje:

```go
type Cache struct {

	mu sync.Mutex

	data map[string]string

}
```

---

# Preporučeni stil

U većini slučajeva:

```go
type Object struct {

	mu sync.Mutex

	state State

}
```

i metode:

```go
func (o *Object) Update()
func (o *Object) Read()
```

---

# Mentalni model

Dobra concurrency struktura izgleda ovako:

```text
        Object

     ┌───────────┐
     │           │
     │  Mutex    │
     │           │
     │  State    │
     │           │
     └───────────┘
```

Objekat sam čuva pravila pristupa.

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto `slice` nije bezbedan za konkurentni upis,
- kako zaštititi `slice` pomoću `Mutex`-a,
- kako dizajnirati strukture koje poseduju sopstveni Mutex,
- zašto je važna enkapsulacija,
- zašto se koristi pointer receiver,
- zašto se `Mutex` ne sme kopirati,
- razliku između embedded i privatnog Mutex-a.

---

# Napredni aspekti `sync.Mutex` — Locking strategije, performanse i ograničenja

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 1/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/01-mutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto osnovno korišćenje `Mutex`-a nije dovoljno
- Lock Contention
- Coarse-grained Locking
- Fine-grained Locking
- Trade-off između jednostavnosti i performansi
- Koliko dugo držati Lock?
- Mutex i performanse
- Kada Mutex postaje usko grlo
- Kada ne koristiti Mutex
- Mutex vs Channel
- Production smernice

---

# Zašto napredni aspekti?

Do sada smo naučili:

```go
mu.Lock()

// Critical Section

mu.Unlock()
```

Tehnički, program radi.

Ali u realnim sistemima postavljaju se pitanja:

- Koliko dugo treba držati Lock?
- Da li jedan Mutex štiti previše podataka?
- Da li više Goroutines previše čeka?
- Da li možemo povećati paralelizam?

---

# Lock Contention

Jedan od najvažnijih pojmova kod Mutex-a je:

> **Lock Contention**

---

# Šta je Lock Contention?

Lock Contention nastaje kada veliki broj Goroutines pokušava da dobije isti Mutex.

Na primer:

```text
             Mutex

              |
              |
     +--------+--------+
     |        |        |
    G1       G2       G3
     |
   LOCKED
```

Jedna Goroutine radi.

Ostale čekaju.

---

# Primer

Imamo:

```go
type Bank struct {

	mu sync.Mutex

	balance int

}
```

Metoda:

```go
func (b *Bank) Update() {

	b.mu.Lock()

	defer b.mu.Unlock()

	// kompleksan rad

}
```

Ako:

- 1000 Goroutines,
- isti Bank objekat,
- isti Mutex,

dobijamo:

```text
1000 Goroutines

↓

1 Mutex

↓

čekanje
```

---

# Posledica

Program i dalje radi ispravno.

Ali:

- mnogo Goroutines blokira,
- CPU vreme se troši na čekanje,
- paralelizam se smanjuje.

---

# Važno pravilo

Mutex rešava:

```
ispravnost
```

ali može ograničiti:

```
paralelizam
```

---

# Coarse-grained Locking

Prva strategija.

---

# Šta znači?

Koristimo jedan veliki Mutex koji štiti veliki deo stanja.

Primer:

```go
type Database struct {

	mu sync.Mutex

	users map[string]User

	orders map[string]Order

	products map[string]Product

}
```

Jedan Mutex štiti sve.

---

# Vizuelno

```text
             Database

        ┌───────────────┐
        │               │
        │    Mutex      │
        │               │
        ├───────────────┤
        │ Users         │
        │ Orders        │
        │ Products      │
        └───────────────┘
```

---

# Prednosti

Jednostavno:

```text
jedan Lock

jedna pravila
```

Manje šanse za greške.

---

# Mane

Ako jedna Goroutine radi:

```go
updateUser()
```

druga ne može:

```go
readProduct()
```

iako nisu povezani.

---

# Fine-grained Locking

Druga strategija.

---

# Ideja

Umesto jednog velikog Mutex-a:

imamo više manjih.

---

Primer:

```go
type Database struct {

	usersMu sync.Mutex

	ordersMu sync.Mutex

	productsMu sync.Mutex

	users map[string]User

	orders map[string]Order

	products map[string]Product

}
```

---

# Vizuelno

```text
Database

 ┌─────────┐
 │ Users   │
 │ Mutex   │
 └─────────┘


 ┌─────────┐
 │ Orders  │
 │ Mutex   │
 └─────────┘


 ┌──────────┐
 │Products  │
 │ Mutex    │
 └──────────┘
```

---

# Prednost

Veći paralelizam.

Primer:

```text
G1

↓

Users Lock


G2

↓

Products Lock
```

Obe mogu raditi istovremeno.

---

# Mana

Veća kompleksnost.

Sada postoji mogućnost:

```text
Lock A

↓

Lock B
```

i druga Goroutine:

```text
Lock B

↓

Lock A
```

---

# Rezultat?

Deadlock.

Ovo ćemo detaljno obraditi kasnije.

---

# Poređenje

| Strategija | Prednost | Mana |
|---|---|---|
| Coarse-grained | Jednostavna | Manji paralelizam |
| Fine-grained | Bolje performanse | Kompleksnija |

---

# Koliko dugo držati Lock?

Veoma važno pravilo:

> Drži Mutex zaključan što kraće.

---

# Loš primer

```go
mu.Lock()

loadData()

calculate()

saveData()

mu.Unlock()
```

Problem:

Mutex je zaključan tokom:

- čitanja,
- računanja,
- I/O operacija.

---

# Bolji primer

```go
data := loadData()

result := calculate(data)

mu.Lock()

save(result)

mu.Unlock()
```

---

# Zašto?

Samo deo koji menja deljeno stanje treba zaštitu.

---

# Critical Section treba biti:

- kratka,
- jasna,
- predvidiva.

---

# Mutex i I/O

Jedna od najčešćih grešaka:

```go
mu.Lock()

database.Query()

mu.Unlock()
```

---

# Zašto loše?

Database poziv može trajati:

- milisekunde,
- sekunde,
- mnogo duže.

Za to vreme:

sve ostale Goroutines čekaju.

---

# Pravilo

Ne drži Mutex tokom:

- HTTP poziva,
- database query-ja,
- file I/O,
- sleep-a,
- čekanja na Channel.

---

# Mutex i CPU performanse

Česta zabluda:

> "Dodavanje Mutex-a usporava program."

Tačno.

Ali pitanje je:

koliko?

---

Mutex uvodi:

- Lock overhead,
- Unlock overhead,
- čekanje.

Ali često je cena mala u odnosu na:

- database,
- mrežu,
- disk,
- kompleksne kalkulacije.

---

# Kada Mutex postaje bottleneck?

Primer:

```text
10000 Goroutines

        |

        |

     jedan Mutex

        |

     jedna operacija
```

Ako svi čekaju isti Lock,

program postaje skoro sekvencijalan.

---

# Kako rešiti?

Opcije:

## 1. Smanjiti Critical Section

Najčešće prvo rešenje.

---

## 2. Koristiti više Lock-ova

Fine-grained locking.

---

## 3. Promeniti dizajn

Na primer:

umesto:

```text
100 Goroutines

↓

jedan shared counter
```

možda:

```text
100 Goroutines

↓

lokalni rezultati

↓

merge
```

---

# Mutex vs Channel

Veoma česta dilema.

---

# Mutex

Koristi:

kada deliš stanje.

Primer:

```go
counter++
```

ili:

```go
cache[key]=value
```

---

# Channel

Koristi:

kada prenosiš podatke.

Primer:

```text
Producer

↓

Channel

↓

Consumer
```

---

# Primer sa Mutex-om

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

# Primer sa Channel-om

```go
jobs := make(chan Job)

jobs <- job
```

---

# Praktično pravilo

Ako pitaš:

> "Ko sme da menja ovaj podatak?"

verovatno razmišljaš o:

```text
Mutex
```

Ako pitaš:

> "Kako da prenesem podatak između Goroutines?"

verovatno razmišljaš o:

```text
Channel
```

---

# Kada Mutex nije dobar izbor?

Izbegavati kada:

## 1. Nema deljenog stanja

Primer:

```go
goroutine(input)
```

---

## 2. Podaci prirodno prolaze kroz pipeline

Primer:

```text
Stage 1

↓

Stage 2

↓

Stage 3
```

Bolji su Channels.

---

## 3. Critical Section je ogromna

Ako zaključavaš pola programa,

problem je dizajn.

---

# Production smernice

## Pravilo 1

Mutex neka bude vlasništvo podatka.

Dobro:

```go
type Cache struct {

	mu sync.Mutex

	data map[string]string

}
```

---

## Pravilo 2

Ne izlaži internu sinhronizaciju korisniku.

Dobro:

```go
cache.Set()
```

Loše:

```go
cache.mu.Lock()
```

---

## Pravilo 3

Ne optimizuj prerano.

Počni sa:

```go
jednostavan Mutex
```

Optimizuj tek kada merenje pokaže problem.

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Lock Contention,
- razliku između Coarse-grained i Fine-grained Locking-a,
- kako Mutex utiče na performanse,
- zašto Critical Section treba biti kratka,
- zašto ne treba držati Lock tokom I/O operacija,
- kada koristiti Mutex, a kada Channel,
- kako dizajnirati production kod sa Mutex-om.

---

# Problemi i debugging kod `sync.Mutex`-a — Deadlock, Double Lock, Lock Ordering i Starvation

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 1/9 (Deo 7)  
>
> **Fajl:** `docs/module-3/01-mutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto `Mutex` može biti opasan
- Deadlock
- Double Lock
- Unlock bez Lock-a
- Lock Ordering problem
- Starvation
- Kako nastaju problemi sa Mutex-om
- Kako pronaći problem
- Debugging tehnike
- Best Practices za izbegavanje grešaka

---

# Uvod

Do sada smo naučili da:

```go
mu.Lock()

// Critical Section

mu.Unlock()
```

štiti deljeni podatak.

Međutim:

> `Mutex` rešava problem konkurentnog pristupa, ali uvodi novu grupu mogućih problema.

Najpoznatiji su:

- Deadlock
- Double Lock
- Pogrešan redosled zaključavanja
- Zaboravljen Unlock
- Starvation

---

# 1. Deadlock

Deadlock je jedan od najpoznatijih concurrency problema.

---

# Definicija

> Deadlock nastaje kada dve ili više Goroutines čekaju jedna drugu i nijedna ne može da nastavi izvršavanje.

Drugim rečima:

```text
A čeka B

B čeka A
```

i program stoji zauvek.

---

# Najjednostavniji primer

Imamo dva Mutex-a:

```go
var (
	mutexA sync.Mutex
	mutexB sync.Mutex
)
```

---

# Goroutine 1

```go
mutexA.Lock()

mutexB.Lock()
```

---

# Goroutine 2

```go
mutexB.Lock()

mutexA.Lock()
```

---

# Šta se događa?

Početno stanje:

```text
Mutex A = slobodan
Mutex B = slobodan
```

---

## Goroutine 1

Uzima:

```text
Lock A
```

Stanje:

```text
A = locked
B = free
```

---

## Goroutine 2

Uzima:

```text
Lock B
```

Stanje:

```text
A = locked
B = locked
```

---

Sada:

Goroutine 1 želi:

```go
mutexB.Lock()
```

Ali:

```text
B je zauzet
```

---

Goroutine 2 želi:

```go
mutexA.Lock()
```

Ali:

```text
A je zauzet
```

---

# Rezultat

```text
G1

drži A

čeka B


G2

drži B

čeka A
```

Niko ne može dalje.

---

# Vizuelni prikaz

```text
        Mutex A

          ▲
          |
          |
G2 čeka  |  G1 drži


        Mutex B

          ▲
          |
          |
G1 čeka  |  G2 drži
```

---

# Program je zaglavljen

Nema:

- panic-a,
- greške,
- automatskog oporavka.

Samo:

```text
čekanje zauvek
```

---

# 2. Double Lock

Double Lock znači:

> Ista Goroutine pokušava da zaključa isti Mutex dva puta.

---

# Primer

```go
func update() {

	mu.Lock()

	defer mu.Unlock()


	mu.Lock()

}
```

---

# Šta se događa?

Prvi:

```go
mu.Lock()
```

uspeva.

Mutex je:

```text
Locked
```

---

Drugi:

```go
mu.Lock()
```

čeka.

Ali ko treba da ga oslobodi?

Ista Goroutine.

---

# Problem

Go `sync.Mutex` nije reentrant.

To znači:

Ista Goroutine ne može ponovo zaključati isti Mutex.

---

# Rezultat

Deadlock.

---

# Pogrešna pretpostavka

Početnici često misle:

> "Ako sam ja zaključao, mogu opet."

Ne.

Mutex ne zna za "vlasnika".

On samo zna:

```text
slobodan

ili

zaključan
```

---

# 3. Unlock bez Lock-a

Suprotan problem.

---

# Primer

```go
var mu sync.Mutex


mu.Unlock()
```

---

# Rezultat

Program dobija:

```text
fatal error:
sync: unlock of unlocked mutex
```

---

# Zašto?

Pokušavamo da otvorimo bravu koja nije zaključana.

---

# 4. Zaboravljen Unlock

Veoma čest problem.

---

# Primer

```go
func update() {

	mu.Lock()

	counter++

}
```

---

# Problem

Nema:

```go
mu.Unlock()
```

---

# Posledica

Prva Goroutine radi.

Sve sledeće:

```go
mu.Lock()
```

čekaju.

---

# Zato koristimo:

```go
mu.Lock()

defer mu.Unlock()
```

---

# 5. Lock Ordering Problem

Ovo je jedan od najvažnijih naprednih problema.

---

# Scenario

Imamo:

```go
Mutex A

Mutex B
```

Pravilo:

Svi moraju uzimati:

```text
A pa B
```

---

# Dobro

```go
muA.Lock()

muB.Lock()
```

---

Druga funkcija:

```go
muA.Lock()

muB.Lock()
```

---

Nema problema.

---

# Loše

Jedna funkcija:

```go
muA.Lock()

muB.Lock()
```

---

Druga:

```go
muB.Lock()

muA.Lock()
```

---

# Rezultat

Mogući deadlock.

---

# Pravilo

U velikim sistemima definiši:

> Globalni redosled zaključavanja.

Na primer:

```text
1. User Mutex

2. Account Mutex

3. Transaction Mutex
```

Nikada obrnuto.

---

# 6. Starvation

Starvation znači:

> Jedna Goroutine dugo vremena ne dobija priliku da izvrši svoj posao.

---

# Primer

Imamo:

```text
G1
G2
G3
G4
```

Sve žele:

```go
mu.Lock()
```

---

Ako neke Goroutines stalno dobijaju Mutex,

jedna može dugo čekati.

---

# Vizuelno

```text
Mutex

↓

G1

↓

G2

↓

G3

↓

G1

↓

G2

```

A:

```text
G4
```

nikada ne dolazi na red.

---

# Da li Go garantuje fer raspodelu?

Ne treba pisati program koji zavisi od toga.

Scheduler i runtime pokušavaju da omoguće napredak,

ali redosled nije API ugovor.

---

# Kako pronaći Mutex probleme?

---

# 1. Race Detector

Najvažniji alat:

```bash
go test -race
```

ili:

```bash
go run -race main.go
```

---

# Šta pronalazi?

Otkriva:

- Data Race,
- konkurentne pristupe bez zaštite.

Ne rešava automatski deadlock.

---

# 2. Stack Trace

Ako program visi:

```bash
Ctrl + \
```

na Unix sistemima.

Može pokazati:

- gde Goroutines čekaju,
- koji Lock pokušavaju.

---

# 3. Logging

Primer:

```go
fmt.Println("trying lock")

mu.Lock()

fmt.Println("got lock")
```

Može pokazati gde program staje.

---

# 4. Profiling

Go ima:

```text
pprof
```

koji može pomoći kod:

- blokiranja,
- contention problema.

---

# Best Practices

## 1. Koristi `defer Unlock`

Dobro:

```go
mu.Lock()

defer mu.Unlock()
```

---

## 2. Drži Lock kratko

Loše:

```go
mu.Lock()

databaseCall()

mu.Unlock()
```

---

## 3. Ne zaključavaj nepotrebno

Ako nema deljenog stanja:

nema potrebe za Mutex-om.

---

## 4. Definiši redosled Lock-ova

Kod više Mutex-a:

```text
uvek isti redosled
```

---

## 5. Ne kopiraj Mutex

Nikada:

```go
copy := object
```

ako sadrži:

```go
sync.Mutex
```

---

# Mentalni model

Mutex je alat.

Kao nož.

Može rešiti problem.

Ali nepravilna upotreba pravi novi problem.

---

# Pravilo za pamćenje

```text
Jedan podatak

+

više Goroutines

+

kontrolisan pristup

=

Mutex
```

Ali:

```text
više Mutex-a

+

različit redosled

=

oprez
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Deadlock,
- kako nastaje Double Lock,
- zašto Unlock bez Lock-a pravi grešku,
- kako nastaje Lock Ordering problem,
- šta je Starvation,
- kako koristiti `-race`,
- kako debugovati Mutex probleme,
- koje prakse sprečavaju concurrency bugove.

---

# `sync.Mutex` — Rezime, kada koristiti, kada izbegavati i praktični zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 1/9 (Deo 8)  
>
> **Fajl:** `docs/module-3/01-mutex.md`

---

# 📚 Sadržaj ovog dela

- Kompletan rezime `sync.Mutex`
- Mentalna mapa
- Mutex filozofija
- Kada koristiti `Mutex`
- Kada izbegavati `Mutex`
- `Mutex` vs Channel
- Najčešće greške
- Production checklist
- Pitanja za proveru znanja
- Praktični zadaci

---

# Kompletan rezime `sync.Mutex`

Tokom ove lekcije prošli smo ceo životni ciklus korišćenja `sync.Mutex`-a:

Od:

```text
Zašto postoji Mutex?
```

do:

```text
Kako ga koristiti u production sistemima?
```

---

# 1. Problem koji Mutex rešava

Problem:

Više Goroutines pristupa istom podatku.

Primer:

```go
counter++
```

---

Bez zaštite:

```text
Goroutine A

        ↘

         counter

        ↗

Goroutine B
```

Rezultat:

- izgubljene izmene,
- nekonzistentno stanje,
- race problemi.

---

# 2. Critical Section

Critical Section je deo programa koji pristupa deljenom resursu.

Primer:

```go
counter++
```

ili:

```go
users[id] = user
```

---

Pravilo:

U jednom trenutku:

```text
samo jedna Goroutine
```

sme izvršavati Critical Section.

---

# 3. Mutual Exclusion

Mutual Exclusion znači:

> Istovremeni pristup je zabranjen.

Od toga dolazi naziv:

```text
Mutex
```

---

# 4. Osnovni obrazac

Svaki Mutex kod prati:

```go
mu.Lock()

// Critical Section

mu.Unlock()
```

---

Idiomski Go stil:

```go
mu.Lock()

defer mu.Unlock()

// Critical Section
```

---

# 5. Shared Memory

Mutex koristimo kada imamo:

```text
više Goroutines

+

isti podatak

+

bar jedna menja podatak
```

---

Primeri:

- counter,
- cache,
- mapa,
- slice,
- stanje objekta,
- konfiguracija koja se menja.

---

# 6. Mutex unutar strukture

Najbolji dizajn:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Objekat:

- poseduje podatke,
- poseduje pravila pristupa.

---

# 7. Pointer Receiver

Ako struktura sadrži:

```go
sync.Mutex
```

koristimo:

```go
func (c *Counter) Increment()
```

a ne:

```go
func (c Counter) Increment()
```

---

Razlog:

Value receiver pravi kopiju.

A Mutex se ne sme kopirati.

---

# 8. `map` i `slice`

Go kolekcije:

```go
map
slice
```

nisu automatski bezbedne za konkurentne izmene.

---

Potrebno:

```go
Mutex

+

kolekcija
```

Primer:

```go
type Cache struct {

	mu sync.Mutex

	data map[string]string

}
```

---

# 9. Lock Contention

Ako mnogo Goroutines čeka isti Mutex:

```text
1000 Goroutines

        |

        |

     jedan Mutex
```

dobijamo:

- čekanje,
- slabiji paralelizam,
- moguće usko grlo.

---

# 10. Coarse vs Fine-grained Locking

## Coarse-grained

Jedan veliki Mutex:

```text
jedan Lock

↓

ceo objekat
```

Prednost:

- jednostavno.

Mana:

- manje paralelizma.

---

## Fine-grained

Više manjih Mutex-a:

```text
User Mutex

Order Mutex

Cache Mutex
```

Prednost:

- više paralelizma.

Mana:

- veća kompleksnost.

---

# 11. Mutex problemi

Najvažniji:

---

## Deadlock

Dve Goroutines čekaju jedna drugu.

```text
A čeka B

B čeka A
```

---

## Double Lock

Ista Goroutine:

```go
Lock()

Lock()
```

---

## Unlock bez Lock-a

```go
Unlock()
```

bez prethodnog:

```go
Lock()
```

---

## Lock Ordering

Različit redosled uzimanja Lock-ova.

---

## Starvation

Jedna Goroutine dugo ne dobija pristup.

---

# Mentalna mapa

```text
                sync.Mutex

                    |
                    |

        +-----------+-----------+

        |                       |

 Shared Memory          Critical Section


        |                       |

        |                       |

    Data Protection       Lock / Unlock


        |

        |

 Performance


        |

        |

 Contention / Deadlock
```

---

# Kada koristiti `sync.Mutex`?

Koristi Mutex kada:

---

## 1. Imaš deljeno stanje

Primer:

```go
type Cache struct {

	mu sync.Mutex

	items map[string]Item

}
```

---

## 2. Potreban ti je jednostavan lock

Primer:

```go
counter++
```

---

## 3. Objekt poseduje interno stanje

Primer:

```go
type Queue struct {

	mu sync.Mutex

	items []Task

}
```

---

## 4. Potrebna je mala kritična sekcija

Primer:

```go
mu.Lock()

state.update()

mu.Unlock()
```

---

# Kada izbegavati `Mutex`?

---

# 1. Kada nema deljene memorije

Primer:

```text
Goroutine A

↓

Channel

↓

Goroutine B
```

Nema potrebe za Mutex-om.

---

# 2. Kada je problem zapravo prenos podataka

Primer:

Producer:

```go
jobs <- job
```

Consumer:

```go
job := <-jobs
```

Bolji su:

```text
Channels
```

---

# 3. Kada Mutex zaključava veliki deo sistema

Ako vidiš:

```go
mu.Lock()

100 linija koda

mu.Unlock()
```

verovatno postoji problem dizajna.

---

# 4. Kada postoji prirodan pipeline

Primer:

```text
Input

↓

Parser

↓

Processor

↓

Writer
```

Koristi:

```text
Channels
```

---

# Mutex vs Channel

Jedno od najvažnijih pitanja.

---

# Mutex

Model:

```text
Delim memoriju

i

štitim pristup
```

---

Primer:

```go
counter++
```

---

# Channel

Model:

```text
Prenosim vlasništvo nad podatkom
```

---

Primer:

```go
jobs <- task
```

---

# Poređenje

| | Mutex | Channel |
|-|-|-|
| Glavna svrha | zaštita stanja | komunikacija |
| Model | shared memory | message passing |
| Najbolji za | cache, state | pipeline |
| Kompleksnost | manja | zavisi od dizajna |
| Performanse | često brže za mali state | odlične za tok podataka |

---

# Pravilo za izbor

Postavi sebi pitanje:

## Pitanje 1

"Da li više Goroutines menjaju isti podatak?"

Ako da:

```text
Mutex
```

---

## Pitanje 2

"Da li jedna Goroutine šalje podatke drugoj?"

Ako da:

```text
Channel
```

---

# Production Checklist

Pre korišćenja Mutex-a proveri:

---

## ✅ Da li postoji shared state?

Ako ne:

ne treba Mutex.

---

## ✅ Da li je Critical Section mala?

Ako nije:

refaktorisati.

---

## ✅ Da li se koristi pointer receiver?

Za struct sa Mutex-om:

uglavnom da.

---

## ✅ Da li postoji mogućnost Deadlock-a?

Proveriti:

- redosled Lock-ova,
- dužinu zaključavanja.

---

## ✅ Da li je Mutex vlasništvo objekta?

Bolje:

```go
object.mu
```

nego:

```go
globalMu
```

---

# Pitanja za proveru znanja

## 1.

Zašto:

```go
counter++
```

nije bezbedan?

---

## 2.

Koja je razlika između:

```go
WaitGroup
```

i:

```go
Mutex
```

---

## 3.

Zašto `Mutex` ne treba kopirati?

---

## 4.

Zašto koristimo pointer receiver?

---

## 5.

Kada bi izabrao Channel umesto Mutex-a?

---

## 6.

Šta je Lock Contention?

---

## 7.

Kako nastaje Deadlock?

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnove

## Zadatak 1

Napravi thread-safe counter:

Zahtevi:

- 100 Goroutines,
- svaka povećava vrednost 100 puta,
- rezultat mora biti tačan.

---

## Zadatak 2

Napraviti:

```go
SafeCounter
```

koji ima:

```go
Increment()

Value()
```

---

# 🟡 Nivo 2 — Kolekcije

## Zadatak 3

Napraviti:

```go
SafeMap
```

koji podržava:

```go
Set()

Get()

Delete()
```

---

## Zadatak 4

Napraviti thread-safe queue:

```go
Push()

Pop()
```

---

# 🟠 Nivo 3 — Dizajn

## Zadatak 5

Napraviti:

```go
Cache
```

koji koristi:

```go
map

+

Mutex
```

---

## Zadatak 6

Dodati statistiku:

```go
hits

misses
```

i zaštititi je.

---

# 🔴 Nivo 4 — Napredno

## Zadatak 7

Implementirati Bank sistem:

Operacije:

```go
Deposit()

Withdraw()

Transfer()
```

Zaštititi od konkurentnih pristupa.

---

## Zadatak 8

Napraviti sistem sa dva Mutex-a.

Namerno izazvati:

```text
Deadlock
```

zatim ga popraviti.

---

# Veliki projekat

## Thread-safe In-memory Database

Implementirati:

```text
Database

├── Users

├── Products

├── Orders

└── Cache
```

Zahtevi:

- konkurentni read,
- konkurentni write,
- zaštita podataka,
- testiranje sa:

```bash
go test -race
```

---

# Završna poruka

`sync.Mutex` je jedan od osnovnih alata Go Concurrency modela.

Najvažnije je razumeti:

```text
Mutex nije za pravljenje paralelizma.

Mutex je za kontrolu paralelnog pristupa.
```

Kada više Goroutines deli stanje:

```text
Mutex štiti podatke.
```

Kada Goroutines razmenjuju informacije:

```text
Channels povezuju tok podataka.
```

Pravi Go programi često koriste oba:

```text
Channels za komunikaciju

+

Mutex za interno stanje
```

---

### ➡️ Sledeća lekcija **[**sync.RWMutex**](02-rwmutex.md)**

obradiće:

- Read Lock,
- Write Lock,
- razliku između `Mutex` i `RWMutex`,
- kada `RWMutex` ubrzava program,
- kada ga zapravo usporava,
- production obrasce korišćenja.
