# Lock-Free Programming

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/04-lock-free-programming.md`

---

# 📚 Sadržaj

- Šta je Lock-Free Programming
- Zašto postoje lock-free algoritmi
- Blocking vs Non-Blocking
- Osnovni pojmovi
- Kada koristiti lock-free pristup
- Prednosti i ograničenja

---

# Uvod

Već smo naučili da:

```go
mu.Lock()

// kritična sekcija

mu.Unlock()
```

rešava problem konkurentnog pristupa.

Ali postoji mana.

Dok jedna Goroutine drži lock:

```
ostale čekaju
```

To se naziva:

```
blocking
```

---

# Šta znači "blocking"?

Ako Goroutine ne može da nastavi dok druga ne oslobodi resurs:

```
Lock()

↓

čekanje

↓

Unlock()

↓

nastavak rada
```

to je **blocking** pristup.

---

# Zašto je to problem?

U sistemima sa velikim brojem Goroutines:

```
100

↓

1 000

↓

10 000

↓

100 000
```

veliki broj Goroutines može čekati isti lock.

To može dovesti do:

- contention-a,
- povećane latencije,
- manjeg iskorišćenja procesora.

---

# Ideja Lock-Free Programming-a

Cilj je:

> **Napredovati bez korišćenja međusobnog blokiranja.**

Umesto:

```
čekaj lock
```

koristi se:

```
pokušaj

↓

ako ne uspe

↓

pokušaj ponovo
```

Najčešće uz pomoć:

```
CAS
```

(*Compare-And-Swap*).

---

# Osnovni princip

Klasičan pristup:

```
Lock

↓

Modify

↓

Unlock
```

Lock-Free pristup:

```
Load

↓

Compute

↓

CAS

↓

Success?

↓

DA → gotovo

↓

NE → pokušaj ponovo
```

---

# Šta je CAS?

CAS znači:

```
Compare-And-Swap
```

Radi tri koraka:

1. Uporedi trenutnu vrednost.
2. Ako je ista kao očekivana, zameni je novom.
3. Ako nije, prijavi neuspeh.

Na primer:

```
counter = 10
```

Želimo:

```
10

↓

11
```

Ako je neko drugi već promenio vrednost:

```
10

↓

15
```

CAS neće izvršiti zamenu.

---

# Vizuelni primer

Dve Goroutines:

```
G1

Load()

↓

10

-------------------

G2

Load()

↓

10
```

G1 uspe:

```
CAS

10

↓

11
```

G2 pokušava:

```
CAS

10

↓

11
```

Ali trenutna vrednost više nije `10`, pa operacija ne uspeva.

G2 ponavlja postupak sa novom vrednošću.

---

# Blocking vs Lock-Free

## Blocking

```
Lock()

↓

čekanje

↓

Unlock()
```

---

## Lock-Free

```
Load()

↓

CAS

↓

Uspeh?

↓

Ne

↓

Retry
```

---

# Šta znači "progress"?

Kod lock-free algoritama važan je pojam:

```
progress
```

Sistem kao celina nastavlja da napreduje.

Čak i ako pojedinačna Goroutine mora da ponovi pokušaj, neka druga Goroutine je napravila napredak.

---

# Gde se koriste?

Lock-Free algoritmi se često nalaze u:

- scheduler-ima,
- runtime sistemima,
- mrežnim bibliotekama,
- visokoperformantnim redovima (queues),
- concurrent stack-ovima,
- memory allocator-ima.

---

# Da li ih treba koristiti svuda?

Ne.

Za većinu poslovnih aplikacija:

```go
Mutex
```

je:

- jednostavniji,
- čitljiviji,
- lakši za održavanje.

Lock-Free algoritmi imaju smisla tek kada postoji jasno izmeren problem sa contention-om.

---

# Prednosti

✅ Nema čekanja na lock.

✅ Manji contention.

✅ Dobra skalabilnost pri velikom broju niti ili Goroutines.

✅ Mogu smanjiti latenciju u specifičnim scenarijima.

---

# Ograničenja

❌ Složeniji dizajn.

❌ Teže dokazivanje ispravnosti.

❌ Veća mogućnost suptilnih bagova.

❌ Problemi kao što je ABA (obrađujemo kasnije).

❌ Često zahtevaju više pokušaja (retry).

---

# Kada koristiti?

Razmisli o lock-free pristupu kada:

- benchmark pokaže ozbiljan contention,
- kritični deo koda dominira ukupnim vremenom izvršavanja,
- jednostavnija rešenja nisu dovoljna.

Za većinu CRUD aplikacija, web servisa i poslovne logike, `Mutex` ili `RWMutex` su sasvim odgovarajući izbor.

---

# Best Practices

✅ Prvo napiši ispravno rešenje.

✅ Izmeri performanse.

✅ Tek onda razmatraj lock-free algoritme.

✅ Koristi standardnu biblioteku kada god je moguće.

---

# Mentalni model

Nemoj razmišljati:

```
Zaključaj

↓

Izmeni

↓

Otključaj
```

Razmišljaj:

```
Pročitaj

↓

Pokušaj izmenu

↓

Ako ne uspe

↓

Pokušaj ponovo
```

To je osnova većine lock-free algoritama.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Lock-Free Programming

✅ razliku između blocking i non-blocking pristupa

✅ osnovnu ideju CAS-a

✅ zašto postoje lock-free algoritmi

✅ njihove prednosti i ograničenja

✅ kada imaju smisla u produkcionim sistemima

---

# Lock-Free Programming — Blocking, Lock-Free i Wait-Free

---

# 📚 Sadržaj

- Progress Guarantees
- Blocking
- Lock-Free
- Wait-Free
- Poređenje
- Prednosti i mane
- Production primeri

---

# Uvod

Do sada smo naučili:

```
Mutex

↓

Blocking
```

i

```
CAS

↓

Lock-Free
```

Ali postoji još jedan važan nivo:

```
Wait-Free
```

Razlika između ova tri modela nije u API-ju, već u tome **kakvu garanciju napretka daju konkurentnom programu**.

---

# Šta su Progress Guarantees?

Progress Guarantee odgovara na pitanje:

> **Šta sistem garantuje kada više niti ili Goroutines istovremeno pokušava da izvrši operaciju?**

Drugim rečima:

```
Da li će operacija završiti?

↓

Kada?

↓

Pod kojim uslovima?
```

---

# 1. Blocking

Kod:

```go
mu.Lock()

// kritična sekcija

mu.Unlock()
```

Ako druga Goroutine drži lock:

```
čekaj
```

Napredak zavisi od druge Goroutine.

---

## Vizuelno

```
G1

Lock()

↓

Radi

--------------------

G2

Lock()

↓

Čeka

↓

Čeka

↓

Čeka
```

Ako G1 nikada ne oslobodi lock:

```
G2

↓

Nikada ne nastavlja.
```

---

# Garancija

```
Nema garancije
napretka.
```

Sve zavisi od vlasnika lock-a.

---

# Prednosti

✅ Jednostavan za razumevanje.

✅ Lak za implementaciju.

✅ Odličan za većinu aplikacija.

---

# Mane

❌ Contention.

❌ Blokiranje.

❌ Mogućnost deadlock-a.

---

# 2. Lock-Free

Lock-Free algoritmi koriste CAS petlje.

Primer:

```
Load()

↓

CAS

↓

Uspeh?

↓

NE

↓

Retry
```

Ako jedna Goroutine izgubi trku:

```
ponavlja pokušaj
```

Ali neka druga Goroutine je uspela.

---

## Vizuelno

```
G1

CAS

↓

Success

------------------

G2

CAS

↓

Fail

↓

Retry

↓

Success
```

---

# Garancija

```
Sistem

kao celina

uvek napreduje.
```

Ne garantuje se da će baš svaka pojedinačna Goroutine brzo završiti, ali se garantuje da će **neka** operacija uspeti.

---

# Prednosti

✅ Nema deadlock-a zbog zaključavanja.

✅ Veoma dobra skalabilnost.

✅ Dobar izbor za visoku konkurentnost.

---

# Mane

❌ Neke Goroutines mogu više puta izgubiti trku.

❌ Moguća pojava "starvation"-a pojedinačne Goroutine.

❌ Složeniji algoritmi.

---

# 3. Wait-Free

Wait-Free algoritam daje najjaču garanciju.

Svaka Goroutine završava svoju operaciju u ograničenom broju koraka.

Bez obzira šta rade ostale Goroutines.

---

## Vizuelno

```
G1

↓

Finish


G2

↓

Finish


G3

↓

Finish
```

Niko ne čeka beskonačno.

---

# Garancija

```
Svaka

pojedinačna

Goroutine

napreduje.
```

Ovo je mnogo jača garancija od lock-free pristupa.

---

# Zašto je Wait-Free težak?

Zato što algoritam mora obezbediti:

```
bez blokiranja

+

bez beskonačnih retry petlji

+

bez međusobnog ometanja
```

To je izuzetno teško za složene strukture podataka.

---

# Poređenje

| Osobina | Blocking | Lock-Free | Wait-Free |
|----------|----------|-----------|-----------|
| Koristi lock | ✅ | ❌ | ❌ |
| Može blokirati | ✅ | ❌ | ❌ |
| Deadlock moguć | ✅ | ❌ | ❌ |
| Retry petlje | ❌ | ✅ | Ponekad (ali sa garantovanim završetkom) |
| Garancija napretka sistema | ❌ | ✅ | ✅ |
| Garancija napretka svake Goroutine | ❌ | ❌ | ✅ |
| Složenost implementacije | Niska | Visoka | Veoma visoka |

---

# Production primeri

## Blocking

- poslovna logika,
- HTTP serveri,
- baze podataka,
- većina CRUD aplikacija.

---

## Lock-Free

- work-stealing scheduler-i,
- concurrent queue,
- memory allocator,
- runtime sistemi.

---

## Wait-Free

- real-time sistemi,
- avionika,
- medicinski uređaji,
- sistemi sa strogim vremenskim ograničenjima.

U Go aplikacijama je wait-free algoritme retko potrebno implementirati ručno.

---

# Najčešće zablude

## Zabluda #1

"Lock-Free znači da niko nikada ne ponavlja operaciju."

Netačno.

Retry petlje su tipične za lock-free algoritme.

---

## Zabluda #2

"Lock-Free znači da će moja Goroutine sigurno završiti."

Ne.

Garantuje se napredak sistema, ne nužno svake pojedinačne Goroutine.

---

## Zabluda #3

"Wait-Free je uvek najbolji."

Ne.

Cena implementacije i održavanja često prevazilazi korist.

Za većinu Go aplikacija to nije opravdano.

---

# Best Practices

✅ Počni sa `Mutex` ili `RWMutex`.

✅ Pređi na lock-free tek kada benchmark pokaže da je contention problem.

✅ Ne implementiraj wait-free algoritme bez vrlo dobrog razloga.

✅ Fokusiraj se na jednostavnost i ispravnost pre optimizacije.

---

# Mentalni model

```
Blocking

↓

Možda čekam drugu Goroutine.

----------------------------

Lock-Free

↓

Možda ponavljam operaciju,
ali sistem napreduje.

----------------------------

Wait-Free

↓

Svaka Goroutine završava
u ograničenom broju koraka.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su Progress Guarantees

✅ razliku između Blocking, Lock-Free i Wait-Free modela

✅ koje garancije svaki model pruža

✅ gde se koriste u praksi

✅ njihove prednosti i ograničenja

---

# Lock-Free Programming — Compare-And-Swap (CAS) Loop

---

# 📚 Sadržaj

- Šta je Compare-And-Swap (CAS)
- Kako radi CAS
- CAS Loop
- Retry mehanizam
- Backoff strategije
- Prednosti i ograničenja
- Primeri u Go-u

---

# Uvod

Kod sa `Mutex`-om izgleda ovako:

```go
mu.Lock()

counter++

mu.Unlock()
```

Kod sa CAS-om izgleda drugačije.

Ne zaključavamo pristup.

Umesto toga:

1. Pročitamo trenutnu vrednost.
2. Izračunamo novu vrednost.
3. Pokušamo da je upišemo.
4. Ako je neko drugi promenio stanje, pokušamo ponovo.

---

# Šta je Compare-And-Swap?

CAS je atomska operacija koja radi tri koraka:

1.

```
Compare
```

Da li je trenutna vrednost jednaka očekivanoj?

---

2.

Ako jeste:

```
Swap
```

Upiši novu vrednost.

---

3.

Ako nije:

```
Fail
```

Ne radi ništa.

Vrati informaciju da operacija nije uspela.

---

# Vizuelni primer

Početno stanje:

```
counter = 10
```

Želimo:

```
10

↓

11
```

CAS proverava:

```
Da li je

counter == 10 ?

↓

DA

↓

counter = 11
```

---

# Ako neko drugi promeni vrednost

Početno stanje:

```
counter = 10
```

Druga Goroutine uradi:

```
counter = 15
```

Naš CAS sada proverava:

```
counter == 10 ?

↓

NE
```

Operacija ne uspeva.

Ništa se ne menja.

---

# Zašto je ovo važno?

Bez CAS-a mogli bismo imati:

```
Read

↓

Modify

↓

Write
```

Između `Read` i `Write` druga Goroutine može promeniti vrednost.

CAS proverava da li je početno stanje i dalje isto.

---

# CAS Loop

Najčešći obrazac izgleda ovako:

```text
Load

↓

Compute

↓

CAS

↓

Success?

↓

YES

↓

Finish

↓

NO

↓

Retry
```

To se naziva:

```
CAS Loop
```

---

# Primer u Go-u

```go
for {
	old := atomic.LoadInt64(&counter)
	newValue := old + 1

	if atomic.CompareAndSwapInt64(
		&counter,
		old,
		newValue,
	) {
		break
	}
}
```

Koraci:

1. Učitaj trenutnu vrednost.
2. Izračunaj novu.
3. Pokušaj zamenu.
4. Ako zamena ne uspe, ponovi ceo postupak.

---

# Zašto mora ponovo?

Zamisli dve Goroutines.

Obe učitaju:

```
100
```

Prva uspešno izvrši:

```
100

↓

101
```

Druga pokušava:

```
100

↓

101
```

Ali trenutna vrednost više nije `100`.

CAS vraća:

```
false
```

Druga Goroutine ponavlja ceo proces sa novom vrednošću.

---

# Vizuelni tok

```
Load

↓

100

↓

Compute

↓

101

↓

CAS

↓

false

↓

Load

↓

101

↓

Compute

↓

102

↓

CAS

↓

true
```

---

# Retry nije greška

Mnogi početnici misle:

```
CAS failed

↓

Problem
```

Naprotiv.

Retry je očekivano ponašanje lock-free algoritama.

Neuspešan CAS samo znači da je neka druga Goroutine u međuvremenu napravila napredak.

---

# Šta je Backoff?

Ako mnogo Goroutines istovremeno pokušava isti CAS:

```
CAS

↓

Fail

↓

Retry

↓

Fail

↓

Retry
```

može doći do velikog broja neuspelih pokušaja.

Backoff znači da Goroutine kratko odloži sledeći pokušaj kako bi smanjila contention.

---

# Vrste Backoff-a

## Constant Backoff

Uvek isto kratko čekanje.

---

## Exponential Backoff

Svaki sledeći neuspeh povećava vreme čekanja.

Na primer:

```
1 μs

↓

2 μs

↓

4 μs

↓

8 μs
```

---

## Random Backoff

Dodaje slučajno kašnjenje kako bi se smanjila verovatnoća da više Goroutines ponovo pokuša u istom trenutku.

---

# Kada koristiti Backoff?

Kod veoma velikog contention-a.

Za mali broj Goroutines često nije potreban.

Važno je naglasiti da Go standardna biblioteka ne nameće jednu univerzalnu strategiju — izbor zavisi od konkretnog algoritma i rezultata merenja.

---

# Prednosti CAS-a

✅ Nema zaključavanja.

✅ Veoma brz u slučaju male konkurencije.

✅ Dobro se skalira.

✅ Osnova većine lock-free algoritama.

---

# Ograničenja

❌ Retry petlje mogu trošiti CPU.

❌ Veliki contention može značajno povećati broj neuspelih pokušaja.

❌ Ne rešava ABA problem (obrađujemo u sledećim lekcijama).

❌ Složeniji je za razumevanje od `Mutex`-a.

---

# Najčešće greške

## Greška #1

Pretpostaviti da će CAS uspeti iz prvog pokušaja.

Ne mora.

---

## Greška #2

Zaboraviti da se nova vrednost mora ponovo izračunati nakon neuspelog CAS-a.

Stara vrednost više nije važeća.

---

## Greška #3

Koristiti CAS za složene operacije koje obuhvataju više međusobno povezanih promenljivih.

Jedan CAS tipično štiti jednu atomsku vrednost ili pokazivač. Za složenije stanje često je prikladniji `Mutex`.

---

# Best Practices

✅ Uvek učitaj trenutno stanje neposredno pre `CompareAndSwap`.

✅ Nakon neuspelog CAS-a ponovo učitaj stanje i izračunaj novu vrednost.

✅ Ne pretpostavljaj da će retry biti redak u uslovima velikog contention-a.

✅ Benchmark-uj pre nego što zameniš `Mutex` CAS petljom.

---

# Mentalni model

Nemoj razmišljati:

```
Zaključaj

↓

Izmeni

↓

Otključaj
```

Razmišljaj:

```
Učitaj

↓

Izračunaj

↓

Pokušaj zamenu

↓

Ako ne uspe

↓

Ponovi
```

To je osnovni obrazac većine lock-free algoritama.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Compare-And-Swap (CAS)

✅ kako funkcioniše CAS operacija

✅ kako izgleda tipična CAS petlja

✅ zašto su retry petlje normalne

✅ šta su backoff strategije

✅ prednosti i ograničenja CAS pristupa

---

# Lock-Free Programming — ABA Problem

---

# 📚 Sadržaj

- Šta je ABA problem
- Kako nastaje
- Zašto CAS ne može da ga otkrije
- Primer sa pokazivačima
- Posledice
- Tehnike za ublažavanje ABA problema
- Odnos prema Go-u

---

# Uvod

CAS proverava:

```
Da li je vrednost ista?
```

Ali ne proverava:

```
Da li se vrednost
u međuvremenu menjala?
```

To je ključ ABA problema.

---

# Jednostavan primer

Početno stanje:

```
A
```

Druga Goroutine uradi:

```
A

↓

B

↓

A
```

Prva Goroutine kasnije izvrši:

```
CAS(A → C)
```

CAS vidi:

```
A
```

i uspešno izvrši zamenu.

Ne zna da je stanje bilo:

```
A

↓

B

↓

A
```

---

# Vizuelni tok

```
G1

Load

↓

A

--------------------

G2

A

↓

B

↓

A

--------------------

G1

CAS(A → C)

↓

Success
```

CAS smatra da je sve u redu.

Ali stanje sistema možda više nije isto kao u trenutku prvog čitanja.

---

# Zašto je ovo problem?

Kod običnih brojeva često nije.

Ali kod pokazivača može biti veoma opasno.

Na primer:

```
Node A
```

može biti uklonjen iz liste, zatim vraćen ili ponovo iskorišćen.

Adresa može ostati ista, ali značenje objekta više nije.

---

# Primer sa stack-om

Pretpostavimo lock-free stack.

Početak:

```
Top

↓

A

↓

B

↓

C
```

Goroutine G1 učita:

```
Top = A
```

Pre nego što izvrši CAS:

G2 uradi:

```
Pop(A)

↓

Pop(B)

↓

Push(A)
```

Novo stanje:

```
Top

↓

A

↓

C
```

G1 vidi:

```
Top == A
```

CAS uspe.

Ali struktura liste više nije ista kao kada je G1 započela operaciju.

To može dovesti do logičkih grešaka.

---

# Zašto CAS ne vidi problem?

CAS proverava samo:

```
Expected

==

Current
```

Ne proverava istoriju promena.

Ne zna da je bilo:

```
A

↓

B

↓

A
```

---

# Gde se ABA javlja?

Najčešće kod:

- lock-free stack-ova,
- lock-free queue struktura,
- freelist-i,
- memory allocator-a,
- hazardnih pokazivača.

Kod prostih atomskih brojača ABA obično nije problem.

---

# Kako se rešava?

Ne postoji jedno univerzalno rešenje.

Najčešće tehnike uključuju sledeće.

---

# 1. Version Tag (Tagged Pointer)

Umesto da se poredi samo pokazivač:

```
Pointer
```

poredi se:

```
Pointer

+

Version
```

Na primer:

```
(A,1)

↓

(A,2)

↓

(A,3)
```

Iako je pokazivač isti:

```
A
```

broj verzije se promenio.

CAS više neće pogrešno zaključiti da je stanje isto.

---

# 2. Hazard Pointers

Goroutine označava da trenutno koristi određeni objekat.

Dok postoji hazard pokazivač:

```
Objekat

↓

ne sme biti oslobođen
```

Ovo sprečava prerano oslobađanje i ponovno korišćenje memorije.

---

# 3. Epoch-Based Reclamation (EBR)

Memorija se ne oslobađa odmah.

Objekti se odlažu za kasnije uklanjanje.

Na taj način se smanjuje mogućnost da ista memorijska lokacija bude prerano ponovo iskorišćena.

---

# 4. Reference Counting

Objekat se oslobađa tek kada broj aktivnih referenci padne na nulu.

Ovaj pristup može biti koristan u određenim sistemima, ali donosi dodatni trošak i složenost.

---

# ABA i Go

Go runtime sa garbage collector-om smanjuje rizik od nekih klasičnih ABA scenarija koji nastaju zbog ručnog upravljanja memorijom.

Međutim:

- ABA kao logički problem i dalje može postojati u lock-free algoritmima.
- Garbage collector **ne rešava automatski ABA problem**.
- Ako razvijaš napredne lock-free strukture podataka, i dalje moraš voditi računa o ABA scenarijima.

---

# Kada ćeš se susresti sa ABA?

Većina Go programera nikada neće direktno implementirati algoritam u kome ABA predstavlja problem.

Ali ako radiš na:

- scheduler-u,
- memory pool-u,
- lock-free kolekcijama,
- runtime komponentama,
- bibliotekama za veoma visoke performanse,

onda je razumevanje ABA problema neophodno.

---

# Najčešće zablude

## Zabluda #1

"CAS rešava sve konkurentne probleme."

Ne.

CAS rešava atomsku zamenu vrednosti, ali ne prepoznaje istoriju promena.

---

## Zabluda #2

"Garbage Collector eliminiše ABA."

Ne.

GC rešava upravljanje memorijom.

ABA je prvenstveno problem logike i sinhronizacije.

---

## Zabluda #3

"ABA postoji samo u C/C++."

Netačno.

Mnogo je češći u jezicima sa ručnim upravljanjem memorijom, ali koncept nije ograničen na njih.

---

# Best Practices

✅ Koristi proverene lock-free algoritme umesto da razvijaš sopstvene, osim ako za to postoji jasan razlog.

✅ Ako implementiraš lock-free strukture sa pokazivačima, razmisli o mehanizmima poput verzionisanja ili drugih tehnika za bezbedno upravljanje životnim ciklusom objekata.

✅ Za većinu poslovnih aplikacija, `Mutex` ili `RWMutex` predstavljaju jednostavnije i bezbednije rešenje.

---

# Mentalni model

Nemoj razmišljati:

```
A

↓

A

↓

Isto stanje
```

Razmišljaj:

```
A

↓

B

↓

A

↓

Možda

nije

isto stanje
```

To je suština ABA problema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je ABA problem

✅ zašto CAS ne može da otkrije promene tipa `A → B → A`

✅ kako ABA utiče na lock-free strukture podataka

✅ najčešće tehnike za ublažavanje ABA problema

✅ zašto Garbage Collector ne rešava automatski ABA problem

---

# Lock-Free Programming — Lock-Free strukture podataka

---

# 📚 Sadržaj

- Lock-Free Counter
- Lock-Free Pointer Update
- Lock-Free Stack (koncept)
- Lock-Free Queue (koncept)
- Poređenje sa `Mutex`
- Kada koristiti lock-free pristup

---

# Uvod

Većina lock-free algoritama koristi isti obrazac:

```
Load

↓

Compute

↓

Compare-And-Swap

↓

Success?

↓

YES → Finish

↓

NO → Retry
```

Razlikuju se samo:

- šta se učitava,
- šta se menja,
- šta predstavlja "novo stanje".

---

# Primer 1 — Lock-Free Counter

Najjednostavniji primer je brojač.

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {
	var counter int64
	var wg sync.WaitGroup

	for i := 0; i < 100; i++ {
		wg.Add(1)

		go func() {
			defer wg.Done()

			atomic.AddInt64(&counter, 1)
		}()
	}

	wg.Wait()

	fmt.Println(counter)
}
```

Ovde ne koristimo `Mutex`.

`atomic.AddInt64` internim mehanizmima obezbeđuje atomsku izmenu vrednosti.

---

# Zašto ne CAS petlja?

Brojači su toliko čest slučaj da standardna biblioteka već nudi specijalizovane operacije:

```go
atomic.AddInt64(...)
```

One su jednostavnije i efikasnije od ručno napisane CAS petlje.

---

# Primer 2 — Ručna CAS petlja

```go
for {
	old := atomic.LoadInt64(&counter)
	newValue := old + 1

	if atomic.CompareAndSwapInt64(
		&counter,
		old,
		newValue,
	) {
		break
	}
}
```

Ovo radi isto kao:

```go
atomic.AddInt64(...)
```

ali služi kao dobar primer obrasca koji koriste složeniji lock-free algoritmi.

---

# Primer 3 — Atomska zamena pokazivača

Ponekad nije potrebno menjati broj, već referencu na objekat.

Na primer:

```go
type Config struct {
	Timeout int
}
```

Možemo atomično zameniti pokazivač na novu konfiguraciju korišćenjem odgovarajućih atomskih tipova ili operacija iz paketa `sync/atomic`.

Koncept je:

```
Old Config

↓

New Config
```

Reader-i uvek vide ili staru ili novu konfiguraciju, ali ne delimično izmenjeno stanje.

Ovaj obrazac je poznat kao **Copy-On-Write** za retke izmene i mnogo čitanja.

---

# Primer 4 — Lock-Free Stack (koncept)

Stack:

```
Top

↓

A

↓

B

↓

C
```

Push:

```
New

↓

A

↓

B

↓

C
```

Koraci:

1. Učitaj trenutni `Top`.
2. Novi čvor neka pokazuje na stari `Top`.
3. CAS pokušava da postavi novi čvor kao `Top`.
4. Ako CAS ne uspe, ponovi postupak.

---

# Vizuelno

```
Load Top

↓

Prepare Node

↓

CAS

↓

Success?

↓

Finish

↓

Retry
```

---

# Pop operacija

```
Top

↓

A

↓

B
```

Koraci:

1. Učitaj `Top`.
2. Zapamti sledeći čvor.
3. CAS pokušava da pomeri `Top` na sledeći čvor.
4. Ako ne uspe, ponovi.

Upravo kod ovakvih algoritama ABA problem postaje posebno važan.

---

# Primer 5 — Lock-Free Queue (koncept)

Queue ima:

```
Head

Tail
```

Dodavanje i uklanjanje elemenata zahteva koordinisano ažuriranje više pokazivača.

Zbog toga su lock-free redovi znatno složeniji od lock-free stack-ova i često koriste više CAS operacija i pažljivo definisana pravila.

---

# Poređenje sa `Mutex`

## `Mutex`

```
Lock

↓

Modify

↓

Unlock
```

Prednosti:

- jednostavno,
- pregledno,
- lako za održavanje.

---

## Lock-Free

```
Load

↓

CAS

↓

Retry
```

Prednosti:

- nema blokiranja zbog zaključavanja,
- dobra skalabilnost pri određenim obrascima konkurentnosti.

Nedostaci:

- složeniji algoritmi,
- teže dokazivanje ispravnosti,
- retry petlje i ABA problem.

---

# Da li je Lock-Free uvek brži?

Ne.

Ako je konkurencija mala:

```
Mutex

↓

često sasvim dovoljan
```

Ako postoji veliki contention:

```
Lock-Free

↓

može biti bolji izbor
```

Bez merenja performansi ne treba pretpostavljati koji pristup je brži.

---

# Gde se koriste?

Lock-free strukture podataka često se nalaze u:

- Go runtime-u,
- scheduler-ima,
- memory pool-ovima,
- mrežnim bibliotekama,
- sistemima sa veoma visokim zahtevima za performansama.

Većina poslovnih aplikacija nema potrebu za njihovom ručnom implementacijom.

---

# Najčešće greške

## Greška #1

Pretpostaviti da lock-free znači "uvek najbrže".

Ne.

Performanse zavise od obrasca pristupa, contention-a i vrste operacija.

---

## Greška #2

Pisati sopstvenu lock-free strukturu bez dobrog razumevanja CAS-a i ABA problema.

Takve greške su često retke, teško reproduktivne i veoma zahtevne za otklanjanje.

---

## Greška #3

Zameniti čitav sistem lock-free algoritmima bez opravdanog razloga.

Složenost održavanja može biti veća od potencijalne koristi.

---

# Best Practices

✅ Koristi gotove primitivne operacije iz `sync/atomic` kada odgovaraju problemu.

✅ Za složene strukture podataka prednost daj jednostavnijim rešenjima (`Mutex`, `RWMutex`) osim ako merenja pokažu potrebu za drugačijim pristupom.

✅ Ako uvodiš lock-free algoritam, dokumentuj njegove garancije i način na koji rešava konkurentne scenarije.

✅ Benchmark i profilisanje treba da prethode optimizaciji.

---

# Mentalni model

```
Mutex

↓

Zaštiti kritičnu sekciju

------------------------

Lock-Free

↓

Zaštiti

promenu stanja
```

Lock-free algoritmi ne štite kod, već pokušavaju da atomično promene stanje sistema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako izgleda lock-free brojač

✅ kako funkcioniše CAS petlja u praksi

✅ koncept lock-free stack-a

✅ koncept lock-free queue-a

✅ razliku između lock-free i `Mutex` pristupa

✅ kada lock-free algoritmi imaju smisla u produkcionom kodu

---

# Lock-Free Programming — Production Guidelines i završni rezime

---

# 📚 Sadržaj

- Kako izabrati odgovarajuću konkurentnu primitivu
- Hijerarhija složenosti
- Production checklist
- Najčešće greške
- Mentalni model
- Završni rezime

---

# Uvod

Posle ovog modula ne bi trebalo da se pitaš:

> "Kako da napišem lock-free algoritam?"

Već:

> "Da li mi je lock-free algoritam uopšte potreban?"

To je mnogo važnije pitanje.

---

# Hijerarhija izbora

Prilikom dizajna konkurentnog sistema, razmišljaj ovim redosledom.

---

## 1. Nema deljenog stanja

Najbolje rešenje.

Primer:

```
Worker

↓

lokalni podaci

↓

rezultat
```

Nema potrebe za sinhronizacijom.

---

## 2. Channels

Ako problem prirodno predstavlja razmenu podataka između Goroutines:

```
Producer

↓

Channel

↓

Consumer
```

Channels često pružaju najjasniji model.

---

## 3. `Mutex`

Ako više Goroutines deli složeno mutable stanje:

```go
type Cache struct {
	mu sync.RWMutex
	m  map[string]string
}
```

`Mutex` ili `RWMutex` su najčešće pravi izbor.

---

## 4. `sync/atomic`

Ako se radi o jednoj vrednosti ili pokazivaču:

```go
counter

flag

pointer
```

Atomske operacije mogu biti jednostavnije i efikasnije od zaključavanja.

---

## 5. Lock-Free algoritmi

Koristi ih tek kada:

- benchmark pokaže ozbiljan contention,
- postoje jasni zahtevi za vrlo niskom latencijom,
- jednostavnija rešenja nisu dovoljna.

---

# Vizuelna hijerarhija

```
No Shared State

↓

Channels

↓

Mutex

↓

Atomic

↓

Lock-Free
```

Kako se spuštaš niz ovu listu:

- raste složenost,
- raste odgovornost,
- raste mogućnost suptilnih grešaka.

---

# Kako odlučiti?

## Pitanje 1

Da li postoji deljeno mutable stanje?

```
NE

↓

Nema problema.
```

---

## Pitanje 2

Da li se problem prirodno rešava razmenom poruka?

```
DA

↓

Channel
```

---

## Pitanje 3

Da li štitiš složenu strukturu?

```
DA

↓

Mutex
```

---

## Pitanje 4

Da li menjaš jednu atomsku vrednost?

```
DA

↓

sync/atomic
```

---

## Pitanje 5

Da li benchmark pokazuje da je contention usko grlo?

```
DA

↓

Razmotri lock-free algoritam.
```

---

# Production Checklist

Pre nego što uvedeš lock-free pristup, proveri:

✅ Da li je problem pravilno izmeren (`benchmark`, `pprof`)?

✅ Da li je `Mutex` zaista usko grlo?

✅ Da li je algoritam dokazano ispravan?

✅ Da li su razmotreni ABA scenariji (ako su relevantni)?

✅ Da li tim može dugoročno održavati ovakav kod?

Ako je odgovor na više pitanja "ne", verovatno je bolje ostati pri jednostavnijem rešenju.

---

# Tipične greške Junior programera

❌ Korišćenje `sync/atomic` umesto `Mutex` samo zato što deluje "brže".

❌ Mešanje atomskih i neatomskih pristupa istom podatku.

❌ Pretpostavka da lock-free automatski znači bolje performanse.

---

# Tipične greške Medior programera

❌ Pisanje sopstvenih lock-free kolekcija bez dovoljno testiranja.

❌ Ignorisanje složenosti održavanja.

❌ Optimizacija bez merenja.

---

# Kako razmišlja Senior Go programer?

Ne pita:

> "Kako da izbegnem `Mutex`?"

Pita:

- Ko poseduje podatke?
- Da li postoji jednostavniji dizajn?
- Da li su performanse zaista problem?
- Mogu li dokazati ispravnost algoritma?
- Hoće li tim razumeti ovaj kod za godinu dana?

---

# Decision Tree

```
Shared State?

↓

NO

↓

No synchronization

-------------------------

YES

↓

Message passing?

↓

YES

↓

Channels

-------------------------

NO

↓

Complex structure?

↓

YES

↓

Mutex

-------------------------

NO

↓

Single atomic value?

↓

YES

↓

sync/atomic

-------------------------

Performance problem?

↓

YES

↓

Lock-Free
```

---

# Mentalni model

Nemoj razmišljati:

```
Šta je najbrže?
```

Razmišljaj:

```
Šta je

najjednostavnije

+

ispravno

+

dovoljno brzo?
```

U produkcionom razvoju to je gotovo uvek bolje pitanje.

---

# Ceo put kroz Modul #4.4

```
Blocking

↓

Lock-Free

↓

Wait-Free

↓

CAS

↓

Retry

↓

ABA

↓

Lock-Free Structures

↓

Production Decision
```

Ovaj sled tema gradi razumevanje od osnovnih pojmova do praktičnog donošenja odluka.

---

# Šta nosiš iz ovog modula?

Trebalo bi da možeš da objasniš:

- razliku između blocking, lock-free i wait-free pristupa,
- kako funkcioniše CAS petlja,
- zašto dolazi do ABA problema,
- kada koristiti `sync/atomic`,
- kada koristiti `Mutex`,
- kada lock-free pristup ima smisla,
- zašto je jednostavnije rešenje često bolje.

---

# 📋 Rezime Modula #4.4

U ovom modulu naučili smo:

✅ šta je Lock-Free Programming

✅ razliku između Blocking, Lock-Free i Wait-Free modela

✅ kako funkcioniše Compare-And-Swap (CAS)

✅ kako izgledaju CAS petlje i retry logika

✅ šta je ABA problem

✅ koncept lock-free struktura podataka

✅ kako izabrati između `channels`, `Mutex`, `sync/atomic` i lock-free pristupa

---

### ➡️ Sledeća lekcija **[**Worker Pools**](05-worker-pools.md)**

Obuhvatiće:

- zašto koristiti Worker Pool
- ograničavanje konkurentnosti (*bounded concurrency*)
- dizajn radnih redova (*job queues*)
- dinamički i statički broj workera
- graceful shutdown
- backpressure
- production obrasci za obradu velikog broja zadataka
