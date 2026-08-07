# Go Memory Model — Uvod

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/03-go-memory-model.md`

---

# 📚 Sadržaj

- Šta je Go Memory Model
- Zašto postoji Memory Model
- Problem sa pretpostavkama
- Memory Visibility
- CPU, Compiler i Runtime
- Zašto konkurentan kod može biti pogrešan

---

# Uvod

Već smo naučili:

- Goroutines
- Channels
- Mutex
- RWMutex
- sync/atomic

Sada dolazimo do pitanja:

> **Zašto oni uopšte rade?**

Odgovor daje:

```
Go Memory Model
```

---

# Šta je Memory Model?

Memory Model definiše:

> **pravila po kojima jedna Goroutine vidi promene koje je napravila druga Goroutine.**

Drugim rečima:

Memory Model odgovara na pitanje:

```
Kada je upisana vrednost
vidljiva drugim Goroutines?
```

---

# Zašto je potreban?

Pretpostavimo:

```go
var ready bool

func writer() {
	ready = true
}

func reader() {
	if ready {
		fmt.Println("Ready!")
	}
}
```

Mnogi očekuju:

```
writer()

↓

reader()

↓

Ready!
```

Ali Go Memory Model kaže:

> Ovo **nije garantovano** ako ne postoji odgovarajuća sinhronizacija.

---

# Najveća zabluda

Početnici često razmišljaju:

```
Promenio sam promenljivu.

↓

Druga Goroutine će odmah videti novu vrednost.
```

To nije pravilo koje garantuje Go.

Bez sinhronizacije:

```
vidljivost nije garantovana
```

---

# Šta znači "vidljivost"?

Primer:

Goroutine A:

```go
x = 42
```

Goroutine B:

```go
fmt.Println(x)
```

Pitanje nije:

> Da li će B pročitati `42`?

Pitanje je:

> **Da li Go garantuje da će B videti `42`?**

Bez sinhronizacije:

```
NE
```

---

# Zašto?

Zato što između koda koji pišemo i izvršavanja postoji više slojeva.

```
Go kod

↓

Compiler

↓

CPU Instructions

↓

CPU Cache

↓

RAM
```

Svaki od ovih slojeva može optimizovati izvršavanje.

---

# Uloga kompajlera

Compiler može:

- promeniti redosled instrukcija,
- ukloniti nepotrebna čitanja,
- zadržati vrednost u registru,
- izvršiti druge optimizacije koje ne menjaju ponašanje **jedne** Goroutine.

Ali u konkurentnom programu te optimizacije mogu promeniti ono što druga Goroutine vidi.

---

# Uloga procesora

Savremeni procesori:

- koriste više jezgara,
- imaju privatne cache memorije,
- izvršavaju instrukcije van redosleda (Out-of-Order Execution),
- koriste write buffer-e.

To znači da:

```
upis

↓

nije nužno odmah vidljiv
```

drugim jezgrima.

---

# Vizuelni model

```
CPU Core 1

↓

L1 Cache

↓

L2 Cache

↓

RAM

------------------------

CPU Core 2

↓

L1 Cache

↓

L2 Cache

↓

RAM
```

Ako Core 1 promeni vrednost, Core 2 možda još neko vreme vidi staru vrednost.

---

# Go ne zabranjuje optimizacije

Važno pravilo:

Go želi da:

- kompajler bude brz,
- procesor bude efikasan,
- program bude konkurentan.

Zbog toga ne zabranjuje optimizacije.

Umesto toga definiše:

```
kada su promene
garantovano vidljive
```

---

# Šta daje garanciju?

Go Memory Model kaže da određene operacije uspostavljaju odnos koji garantuje vidljivost.

Na primer:

- `Mutex`
- `RWMutex`
- `Channel`
- `sync.Once`
- `WaitGroup` (u odgovarajućim obrascima korišćenja)
- `sync/atomic`

Sve ove operacije stvaraju jasno definisane tačke sinhronizacije.

---

# Šta ako nema sinhronizacije?

Primer:

```go
var x int

go func() {
	x = 100
}()

fmt.Println(x)
```

Rezultat može biti:

```
0
```

ili:

```
100
```

Program sadrži **data race**, pa Go Memory Model više ne daje garancije o ispravnom ponašanju.

---

# Šta zapravo garantuje Go?

Go ne garantuje:

```
šta ćeš videti
```

Go garantuje:

```
ako koristiš
ispravnu sinhronizaciju,

onda će sve Goroutines
videti konzistentno stanje
```

To je ključna razlika.

---

# Mentalni model

Nemoj razmišljati:

```
Upisao sam vrednost.

↓

Sigurno je svi vide.
```

Razmišljaj:

```
Upisao sam vrednost.

↓

Da li postoji
sinhronizacija?

↓

Ako postoji,

Go garantuje
vidljivost.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Go Memory Model

✅ šta znači memory visibility

✅ zašto optimizacije kompajlera i CPU-a utiču na konkurentni kod

✅ zašto običan upis u promenljivu nije dovoljan

✅ koje vrste sinhronizacije uspostavljaju garancije vidljivosti

---

# Go Memory Model — Memory Visibility

---

# 📚 Sadržaj

- Šta je Memory Visibility
- Putanja podataka kroz sistem
- CPU registri
- CPU cache
- Write buffers
- Zašto Goroutines mogu videti različite vrednosti
- Kako sinhronizacija rešava problem

---

# Uvod

U prethodnom delu naučili smo da:

> Upis u promenljivu **nije isto što i garantovana vidljivost tog upisa**.

Sada ćemo videti zašto.

---

# Šta je Memory Visibility?

Memory Visibility odgovara na pitanje:

> **Kada promena koju napravi jedna Goroutine postaje vidljiva drugim Goroutines?**

Naglasak je na reči:

```
vidljiva
```

Nije dovoljno da je vrednost upisana.

Potrebno je da druga Goroutine može pouzdano da je pročita.

---

# Kako izgleda putanja podataka?

Kada napišemo:

```go
counter++
```

ne znači da procesor odmah upisuje novu vrednost u RAM.

Pojednostavljen prikaz:

```
Go kod

↓

Compiler

↓

CPU registri

↓

Write Buffer

↓

CPU Cache

↓

RAM
```

Na svakom koraku mogu postojati optimizacije ili privremeno zadržavanje podataka.

---

# CPU registri

Registri su najbrža memorija procesora.

Na primer:

```go
x := counter
```

Procesor može zadržati `counter` u registru umesto da ga ponovo čita iz memorije.

To ubrzava izvršavanje, ali znači da druga jezgra ne vide automatski tu vrednost.

---

# CPU Cache

Svako jezgro obično ima sopstveni cache.

Primer:

```
Core 1

↓

L1 Cache

↓

L2 Cache

↓

RAM

------------------------

Core 2

↓

L1 Cache

↓

L2 Cache

↓

RAM
```

Ako Core 1 promeni vrednost u svom cache-u, Core 2 može još neko vreme koristiti staru kopiju.

---

# Write Buffer

Moderni procesori koriste write buffer.

To znači da se upis može privremeno zadržati pre nego što postane vidljiv ostatku sistema.

Pojednostavljeno:

```
counter = 100

↓

Write Buffer

↓

CPU Cache

↓

RAM
```

Druga Goroutine možda još ne vidi novu vrednost.

---

# Primer bez sinhronizacije

```go
var ready bool

func writer() {
	ready = true
}

func reader() {
	if ready {
		fmt.Println("Ready")
	}
}
```

Intuitivno očekujemo:

```
writer()

↓

ready = true

↓

reader()

↓

Ready
```

Ali bez sinhronizacije Go ne garantuje da će `reader()` videti `true`.

---

# Dve Goroutine

```
Goroutine A

↓

ready = true

------------------------

Goroutine B

↓

if ready
```

Ako ne postoji sinhronizacija:

```
?

```

Rezultat nije definisan Memory Model-om.

---

# Šta radi sinhronizacija?

Operacije kao što su:

- `Mutex`
- `atomic`
- slanje i prijem preko `channel`-a

uspostavljaju tačke na kojima Go obezbeđuje da prethodni upisi postanu vidljivi drugim Goroutines.

Na konceptualnom nivou:

```
Writer

↓

Write

↓

Synchronization

↓

Reader

↓

Read
```

---

# Primer sa Mutex-om

```go
mu.Lock()
ready = true
mu.Unlock()
```

Druga Goroutine:

```go
mu.Lock()

fmt.Println(ready)

mu.Unlock()
```

Ovde Go Memory Model garantuje da će druga Goroutine videti promene koje su se dogodile pre `Unlock()`.

---

# Primer sa atomic

```go
atomic.StoreUint32(&ready, 1)
```

Čitanje:

```go
if atomic.LoadUint32(&ready) == 1 {
	fmt.Println("Ready")
}
```

`Store()` i `Load()` nisu samo atomske operacije.

One obezbeđuju i odgovarajuću vidljivost memorije prema Go Memory Model-u.

---

# Primer sa channel-om

```go
done := make(chan struct{})

go func() {
	ready = true
	done <- struct{}{}
}()

<-done

fmt.Println(ready)
```

Prijem sa kanala garantuje da su promene koje su prethodile slanju vidljive nakon uspešnog prijema.

---

# Zašto je ovo važno?

Bez pravila o vidljivosti mogli bismo imati situaciju:

```
Writer:

x = 42

↓

Reader:

x == 0
```

Iako je `writer` već izvršio upis.

Memory Model precizno definiše kada takva situacija **ne može** da se dogodi.

---

# Najčešće zablude

## Zabluda #1

"Program radi na mom računaru."

To ne znači da je ispravan.

Možda se race condition jednostavno još nije ispoljio.

---

## Zabluda #2

"x86 uvek radi kako očekujem."

Neke arhitekture imaju jači model memorije od drugih, ali Go Memory Model definiše pravila na nivou jezika.

Ne treba pisati kod koji zavisi od ponašanja određenog procesora.

---

## Zabluda #3

"Dovoljno je da koristim `volatile`."

Go nema `volatile` ključnu reč.

Za sinhronizaciju koristi:

- `sync`
- `sync/atomic`
- `channels`

---

# Best Practices

✅ Svaki deljeni podatak treba da ima jasno definisan mehanizam sinhronizacije.

✅ Nemoj pretpostavljati da će običan upis odmah biti vidljiv drugim Goroutines.

✅ Koristi `atomic` samo za jednostavne deljene vrednosti.

✅ Za složenije stanje koristi `Mutex`, `RWMutex` ili arhitekturu zasnovanu na kanalima.

---

# Mentalni model

Nemoj razmišljati:

```
Write

↓

Read
```

Razmišljaj:

```
Write

↓

Synchronization

↓

Memory Visibility

↓

Read
```

Sinhronizacija je ono što obezbeđuje da drugi učesnici vide tvoje promene.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta znači memory visibility

✅ kako CPU registri, cache i write buffer utiču na vidljivost podataka

✅ zašto običan upis nije dovoljan

✅ kako `Mutex`, `atomic` i `channels` obezbeđuju vidljivost

✅ najčešće zablude vezane za konkurentni pristup memoriji

---

# Go Memory Model — Instruction Reordering

---

# 📚 Sadržaj

- Šta je Instruction Reordering
- Compiler Reordering
- CPU Out-of-Order Execution
- Zašto se instrukcije preuređuju
- Kada je to dozvoljeno
- Kada nije
- Kako sinhronizacija sprečava pogrešan redosled

---

# Uvod

Pogledaj sledeći kod:

```go
x = 10
y = 20
```

Većina programera zamišlja:

```
1.

x = 10

↓

2.

y = 20
```

Ali stvarni redosled izvršavanja može biti drugačiji.

---

# Šta je Instruction Reordering?

Instruction Reordering znači da:

```
Compiler

ili

CPU
```

mogu promeniti redosled izvršavanja instrukcija.

Važno:

Program mora ostati ispravan za **jednu Goroutine**.

---

# Zašto se to radi?

Glavni cilj je:

```
veće performanse
```

Procesor pokušava da:

- smanji čekanje,
- bolje iskoristi izvršne jedinice,
- poveća broj instrukcija izvršenih u jedinici vremena.

---

# Compiler Reordering

Kompajler može promeniti redosled nezavisnih instrukcija.

Primer:

```go
a = 1
b = 2
```

Ako između njih nema zavisnosti, kompajler ih može reorganizovati.

Sa stanovišta jedne Goroutine rezultat ostaje isti.

---

# CPU Out-of-Order Execution

Savremeni procesori mogu izvršavati instrukcije van njihovog redosleda u programu.

Na primer:

```text
Instrukcije:

1
2
3
4
```

Procesor može interno izvršiti:

```text
1
3
2
4
```

ako ne postoje međusobne zavisnosti.

Na kraju obezbeđuje da rezultat odgovara pravilima arhitekture i jezika.

---

# Primer

Kod:

```go
a = 1
b = 2
```

Moguće interno izvršavanje:

```
b = 2

↓

a = 1
```

Ako nijedna druga Goroutine ne posmatra ove promenljive bez sinhronizacije, to ne menja ponašanje programa.

---

# Problem u konkurentnom programu

Pretpostavimo:

```go
data = 42
ready = true
```

Druga Goroutine radi:

```go
if ready {
	fmt.Println(data)
}
```

Intuitivno očekujemo:

```
data = 42

↓

ready = true

↓

reader vidi data = 42
```

Ali bez sinhronizacije Go Memory Model to **ne garantuje**.

Ako program sadrži data race, ne treba se oslanjati na bilo kakav redosled.

---

# Šta radi Memory Model?

Go Memory Model ne zabranjuje optimizacije.

On definiše:

```
kada određeni redosled

mora

biti vidljiv drugim Goroutines
```

To se postiže sinhronizacionim operacijama.

---

# Memory Barrier (koncept)

Sinhronizacione operacije deluju kao granice preko kojih se ne sme narušiti potrebna vidljivost.

Konceptualno:

```
Write

↓

Memory Barrier

↓

Read
```

Ne moraš znati kako je to implementirano na svakoj arhitekturi, ali je važno razumeti ulogu.

---

# Primer sa Mutex-om

```go
mu.Lock()

data = 42
ready = true

mu.Unlock()
```

Druga Goroutine:

```go
mu.Lock()

fmt.Println(data)
fmt.Println(ready)

mu.Unlock()
```

`Unlock()` i naredni uspešan `Lock()` uspostavljaju odnos koji garantuje da će druga Goroutine videti prethodne upise.

---

# Primer sa atomic

```go
atomic.StoreInt32(&ready, 1)
```

Kasnije:

```go
if atomic.LoadInt32(&ready) == 1 {
	// ...
}
```

Atomic operacije obezbeđuju potrebnu sinhronizaciju prema Go Memory Model-u.

---

# Šta nije dozvoljeno?

Ako postoji odgovarajuća sinhronizacija, kompajler i procesor ne smeju narušiti garancije koje Memory Model daje.

Drugim rečima, optimizacije su dozvoljene samo dok ne promene spolja vidljivo ponašanje programa koje je definisano Memory Model-om.

---

# Najčešće zablude

## Zabluda #1

"Procesor izvršava kod odozgo nadole."

Ne nužno.

Bitan je rezultat koji je dozvoljen pravilima arhitekture i Go Memory Model-a.

---

## Zabluda #2

"Ako radi bez `Mutex`-a, onda je ispravno."

Može raditi hiljadama izvršavanja, a zatim jednom dati pogrešan rezultat.

To je tipično ponašanje programa sa data race-om.

---

## Zabluda #3

"Reordering je greška."

Naprotiv.

To je jedna od najvažnijih optimizacija modernih procesora i kompajlera.

Problem nastaje kada se piše konkurentan kod bez odgovarajuće sinhronizacije.

---

# Best Practices

✅ Nemoj se oslanjati na redosled izvršavanja koji vidiš u izvornom kodu ako postoji data race.

✅ Koristi `Mutex`, `atomic` ili `channels` da uspostaviš potrebne garancije.

✅ Piši kod koji je ispravan prema Go Memory Model-u, a ne prema ponašanju jednog procesora ili jedne Go verzije.

---

# Mentalni model

Nemoj razmišljati:

```
Kod

↓

CPU izvršava identično
```

Razmišljaj:

```
Kod

↓

Compiler optimizuje

↓

CPU optimizuje

↓

Memory Model određuje
šta mora ostati garantovano
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Instruction Reordering

✅ zašto kompajler i CPU preuređuju instrukcije

✅ šta je Out-of-Order Execution

✅ zašto je to bezbedno za jednu Goroutine

✅ kako Go Memory Model ograničava optimizacije u prisustvu sinhronizacije

---

# Go Memory Model — Happens-Before Relation

---

# 📚 Sadržaj

- Šta je Happens-Before
- Zašto je važan
- Formalna ideja
- Happens-Before i Memory Visibility
- Happens-Before pravila
- Primeri sa `Mutex`
- Primeri sa `channels`
- Primeri sa `sync/atomic`
- Primeri sa `sync.Once`

---

# Uvod

Do sada smo naučili:

- Memory Visibility
- CPU Cache
- Instruction Reordering

Sada možemo odgovoriti na najvažnije pitanje:

> **Kada Go garantuje da će jedna Goroutine videti promene druge Goroutine?**

Odgovor je:

```
Kada između njih postoji
Happens-Before relacija.
```

---

# Šta je Happens-Before?

Happens-Before (HB) je logički odnos između dve operacije.

Ako:

```
A

happens-before

B
```

onda Go garantuje da:

- svi upisi koji su se dogodili pre A,
- biće vidljivi operaciji B.

HB nije isto što i:

```
vremenski redosled
```

već:

```
garantovani redosled
```

definisan Memory Model-om.

---

# Vizuelni model

Bez HB:

```
Write

      ?

Read
```

Nema garancije.

---

Sa HB:

```
Write

↓

Synchronization

↓

Read
```

Sada postoji garancija vidljivosti.

---

# Najvažnija ideja

Nemoj razmišljati:

```
Prvo se izvršilo A.

↓

Zato će B videti rezultat.
```

Razmišljaj:

```
Da li postoji

Happens-Before?

↓

DA

↓

Vidljivost je garantovana.
```

---

# Primer bez HB

```go
var x int

go func() {
	x = 100
}()

fmt.Println(x)
```

Ovde:

```
HB

↓

ne postoji
```

Program ima data race.

Go ne daje garancije.

---

# Mutex i Happens-Before

Primer:

```go
mu.Lock()

x = 100

mu.Unlock()
```

Druga Goroutine:

```go
mu.Lock()

fmt.Println(x)

mu.Unlock()
```

Go garantuje:

```
Unlock()

↓

happens-before

↓

sledeći uspešan Lock()
```

Zbog toga druga Goroutine vidi:

```
100
```

---

# Vizuelno

```
Writer

Lock()

↓

x = 100

↓

Unlock()

====================

Reader

Lock()

↓

x == 100
```

Linija između `Unlock()` i narednog uspešnog `Lock()` predstavlja HB relaciju.

---

# Channels i Happens-Before

Primer:

```go
done := make(chan struct{})

go func() {

	x = 100

	done <- struct{}{}

}()
```

Reader:

```go
<-done

fmt.Println(x)
```

Go garantuje:

```
Send

↓

HB

↓

Receive
```

Sve što se dogodilo pre slanja postaje vidljivo nakon prijema.

---

# Atomic i Happens-Before

Primer:

```go
atomic.StoreInt64(&counter, 10)
```

Reader:

```go
v := atomic.LoadInt64(&counter)
```

Kada `Load()` pročita vrednost koju je zapisao odgovarajući `Store()`, Go Memory Model obezbeđuje potrebnu vidljivost za tu sinhronizaciju.

Atomic operacije nisu samo atomske.

One su i sinhronizacione operacije.

---

# sync.Once

Primer:

```go
var once sync.Once

once.Do(initConfig)
```

Sve Goroutines koje se vrate iz:

```go
once.Do(...)
```

garantovano vide sve promene koje je napravila funkcija prosleđena `Do`.

To omogućava bezbednu jednokratnu inicijalizaciju.

---

# WaitGroup

`WaitGroup` služi za koordinaciju završetka Goroutines.

Kada se koristi prema dokumentovanom obrascu (svaka Goroutine poziva `Done()`, a druga čeka `Wait()`), može se koristiti za bezbedno čekanje završetka rada.

Ipak, `WaitGroup` nije opšti mehanizam za zaštitu deljenog mutable stanja.

Za deljene podatke i dalje su potrebni odgovarajući mehanizmi sinhronizacije (`Mutex`, `atomic`, `channels` i sl.).

---

# Šta uspostavlja Happens-Before?

Najčešće:

- `Mutex`
- `RWMutex`
- `Channel send/receive`
- `Channel close`
- `sync.Once`
- `sync.Cond`
- `sync/atomic`
- druge sinhronizacione operacije definisane Go Memory Model-om

---

# Šta NE uspostavlja HB?

Običan upis:

```go
x = 100
```

Obično čitanje:

```go
fmt.Println(x)
```

Obična promenljiva:

```go
ready = true
```

Bez sinhronizacije:

```
HB

↓

ne postoji
```

---

# Mentalni model

Svaki put kada vidiš:

```
jedna Goroutine piše

↓

druga čita
```

postavi pitanje:

```
Gde je Happens-Before?
```

Ako ne možeš da pokažeš HB relaciju:

verovatno postoji problem u konkurentnom kodu.

---

# Najčešće greške

## Greška #1

Pretpostaviti da:

```
goroutine A

↓

goroutine B
```

automatski znači da će B videti sve izmene A.

Ne postoji takva garancija bez odgovarajuće sinhronizacije.

---

## Greška #2

Mešati:

```
redosled izvršavanja

i

happens-before
```

HB je formalna garancija Memory Model-a.

Nije isto što i hronološki redosled.

---

## Greška #3

Koristiti deljene promenljive bez jasne HB relacije.

To vodi ka:

- data race,
- nepredvidivom ponašanju,
- teško reproduktivnim bagovima.

---

# Best Practices

✅ Za svaki deljeni podatak identifikuj HB relaciju.

✅ Koristi sinhronizacione primitive koje eksplicitno uspostavljaju HB.

✅ Ako ne možeš da objasniš zašto postoji HB između writer-a i reader-a, preispitaj dizajn.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Happens-Before

✅ razliku između vremenskog i garantovanog redosleda

✅ kako `Mutex` uspostavlja HB

✅ kako `channels` uspostavljaju HB

✅ kako `sync/atomic` uspostavlja HB

✅ kako `sync.Once` uspostavlja HB

✅ zašto je HB centralni koncept Go Memory Model-a

---

# Go Memory Model — Happens-Before pravila

---

# 📚 Sadržaj

- Happens-Before pravila
- Pokretanje Goroutine
- `Mutex`
- `RWMutex`
- `channels`
- `close(channel)`
- `sync.Once`
- `sync/atomic`
- Praktični primeri

---

# Uvod

Do sada smo naučili:

```
Write

↓

Synchronization

↓

Read
```

Sada ćemo videti **koje konkretne operacije predstavljaju sinhronizaciju**.

---

# Pravilo 1 — Pokretanje Goroutine

Primer:

```go
x := 42

go func() {
	fmt.Println(x)
}()
```

Pokretanje Goroutine preko `go` naredbe garantuje da se sve što je izvršeno **pre** `go` naredbe vidi u novoj Goroutine.

Dakle:

```go
x = 42

↓

go func()
```

Nova Goroutine će videti vrednost `42`.

---

# Šta ovo NE znači?

Ako kasnije uradimo:

```go
go worker()

x = 100
```

to **ne znači** da će `worker()` videti `100`.

Za promene koje se dese nakon pokretanja Goroutine potrebna je dodatna sinhronizacija.

---

# Pravilo 2 — `Mutex`

Pisac:

```go
mu.Lock()

x = 100

mu.Unlock()
```

Čitalac:

```go
mu.Lock()

fmt.Println(x)

mu.Unlock()
```

Go garantuje:

```
Unlock()

↓

HB

↓

sledeći uspešan Lock()
```

---

# Pravilo 3 — `RWMutex`

Za `RWMutex` važe ista osnovna pravila vidljivosti.

Writer:

```go
rw.Lock()

x = 100

rw.Unlock()
```

Reader:

```go
rw.RLock()

fmt.Println(x)

rw.RUnlock()
```

Ako `RLock()` uspe nakon odgovarajućeg `Unlock()`, reader vidi prethodne izmene.

---

# Pravilo 4 — Slanje preko channel-a

Primer:

```go
done := make(chan struct{})

go func() {

	x = 100

	done <- struct{}{}

}()
```

Reader:

```go
<-done

fmt.Println(x)
```

Garancija:

```
Send

↓

HB

↓

Receive
```

Sve što je prethodilo slanju vidljivo je nakon prijema.

---

# Pravilo 5 — Zatvaranje channel-a

Primer:

```go
x = 100

close(done)
```

Druga Goroutine:

```go
<-done

fmt.Println(x)
```

Uspešan prijem koji saznaje da je kanal zatvoren (`ok == false`, ili prijem sa već zatvorenog kanala) vidi promene koje su se dogodile pre `close()`.

Zbog toga se `close()` često koristi kao signal da je određena faza rada završena.

---

# Pravilo 6 — `sync.Once`

```go
once.Do(initConfig)
```

Sve Goroutines koje prođu kroz:

```go
once.Do(...)
```

garantovano vide efekte koje je napravila funkcija `initConfig`.

---

# Pravilo 7 — `sync/atomic`

Primer:

```go
atomic.StoreUint64(&counter, 100)
```

Kasnije:

```go
v := atomic.LoadUint64(&counter)
```

Kada `Load()` pročita vrednost koju je zapisao odgovarajući `Store()`, Go garantuje potrebnu vidljivost memorije.

---

# Vizuelni pregled

```
go statement

↓

HB

↓

nova Goroutine


---------------------------

Unlock

↓

HB

↓

Lock


---------------------------

Send

↓

HB

↓

Receive


---------------------------

close(channel)

↓

HB

↓

Receive closed


---------------------------

Store

↓

HB

↓

Load


---------------------------

sync.Once

↓

HB

↓

ostale Goroutines
```

---

# Primer bez HB

```go
var ready bool

go func() {
	ready = true
}()

for !ready {
}
```

Ovde:

```
HB

↓

ne postoji
```

Program ima data race.

Petlja može da se ponaša nepredvidivo.

---

# Primer sa HB

```go
done := make(chan struct{})

go func() {

	ready = true

	close(done)

}()

<-done

fmt.Println(ready)
```

Ovde postoji jasno definisana Happens-Before relacija.

---

# Najčešće greške

## Greška #1

Pretpostaviti da:

```go
go worker()
```

automatski sinhronizuje sve buduće izmene.

Ne sinhronizuje.

---

## Greška #2

Koristiti običnu promenljivu kao signal.

Loše:

```go
ready = true
```

Bez odgovarajuće sinhronizacije nema garancije vidljivosti.

---

## Greška #3

Zaboraviti da `channel` nije samo mehanizam za prenos podataka.

On je i mehanizam za sinhronizaciju.

---

# Best Practices

✅ Kada koristiš deljene podatke, identifikuj koja operacija uspostavlja Happens-Before.

✅ Nemoj se oslanjati na redosled izvršavanja bez sinhronizacije.

✅ Kanali mogu služiti i kao signal za završetak rada, ne samo za prenos vrednosti.

✅ Za jednostavne signale često je jasnije koristiti `close(done)` nego deljenu `bool` promenljivu.

---

# Mentalni model

Svaki konkurentni program možeš analizirati ovako:

```
Writer

↓

Koja operacija
uspostavlja HB?

↓

Reader
```

Ako nema odgovora na to pitanje, potrebno je dodati odgovarajuću sinhronizaciju.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Happens-Before pravila za `go` naredbu

✅ pravila za `Mutex` i `RWMutex`

✅ pravila za `channels` i `close(channel)`

✅ pravila za `sync.Once`

✅ pravila za `sync/atomic`

✅ kako prepoznati da li između dve Goroutine postoji garantovana vidljivost

---

# Go Memory Model — Production analiza i završni rezime

---

# 📚 Sadržaj

- Kako analizirati konkurentan kod
- Studije slučaja
- Tipične greške
- Checklist za Go Memory Model
- Mentalni model
- Završni rezime

---

# Uvod

Posle ovog modula ne treba da razmišljaš:

> "Da li će ovo raditi?"

Već:

> "Da li mogu da dokažem da je ovaj kod ispravan prema Go Memory Model-u?"

To je način razmišljanja koji se očekuje od iskusnog Go programera.

---

# Metodologija analize

Za svaki konkurentni kod postavi sledeća pitanja.

---

## 1. Ko piše podatke?

Primer:

```go
config.Timeout = 30
```

Koja Goroutine ovo radi?

---

## 2. Ko čita podatke?

Primer:

```go
fmt.Println(config.Timeout)
```

Koliko Goroutines čita istu vrednost?

---

## 3. Da li postoji deljeno stanje?

Ako ne postoji:

```
Nema problema.
```

Ako postoji:

```
Potrebna je sinhronizacija.
```

---

## 4. Gde je Happens-Before?

Najvažnije pitanje.

Možeš li pokazati:

```
Writer

↓

HB

↓

Reader
```

Ako ne možeš:

Postoji velika verovatnoća da kod nije ispravan.

---

## 5. Ko garantuje vidljivost?

To može biti:

- `Mutex`
- `RWMutex`
- `channel`
- `sync.Once`
- `sync/atomic`

Ako odgovor glasi:

```
"Niko."
```

potrebno je promeniti dizajn.

---

# Studija slučaja #1

```go
var ready bool

go func() {
	ready = true
}()

for !ready {
}
```

Analiza:

```
Writer

↓

ready = true

-------------------

Reader

↓

for !ready
```

HB?

```
NE
```

Zaključak:

```
Data Race
```

---

# Studija slučaja #2

```go
done := make(chan struct{})

go func() {

	ready = true

	close(done)

}()

<-done

fmt.Println(ready)
```

Analiza:

```
Writer

↓

close(done)

↓

HB

↓

Receive

↓

Reader
```

Zaključak:

```
Ispravno
```

---

# Studija slučaja #3

```go
mu.Lock()

value = 100

mu.Unlock()
```

Reader:

```go
mu.Lock()

fmt.Println(value)

mu.Unlock()
```

Analiza:

```
Unlock

↓

HB

↓

Lock
```

Zaključak:

```
Ispravno
```

---

# Studija slučaja #4

```go
atomic.StoreInt64(&counter, 100)

fmt.Println(
	atomic.LoadInt64(&counter),
)
```

Analiza:

```
Store

↓

HB

↓

Load
```

Zaključak:

```
Ispravno
```

---

# Tipične greške Junior programera

❌ Deljenje promenljivih bez sinhronizacije.

❌ Pretpostavka da Goroutine odmah vidi promene.

❌ Korišćenje `time.Sleep()` kao mehanizma sinhronizacije.

❌ Mešanje `Mutex` i `atomic` nad istim podatkom bez pažljivo definisanog dizajna.

❌ Ignorisanje rezultata `go test -race`.

---

# Tipične greške Medior programera

❌ Prerana optimizacija prelaskom na `atomic`.

❌ Korišćenje Copy-On-Write za često menjane velike strukture.

❌ Nepotpuno kopiranje referentnih tipova (`map`, `slice`) pri pravljenju snapshot-a.

❌ Oslanjanje na ponašanje određene CPU arhitekture umesto na Go Memory Model.

---

# Kako razmišlja Senior Go programer?

Ne pita:

> "Da li radi?"

Pita:

- Ko poseduje podatke?
- Ko ih menja?
- Ko ih čita?
- Gde je Happens-Before?
- Da li postoji data race?
- Da li je dizajn jednostavan za održavanje?

---

# Go Memory Model Checklist

Pre nego što završiš implementaciju konkurentnog koda, proveri:

✅ Da li postoji deljeno mutable stanje?

✅ Da li je za svaki deljeni podatak definisan vlasnik ili odgovarajuća zaštita?

✅ Da li postoji jasno definisana Happens-Before relacija između upisa i čitanja?

✅ Da li koristiš odgovarajući mehanizam (`Mutex`, `RWMutex`, `channels`, `sync/atomic`, `sync.Once`)?

✅ Da li je kod prošao `go test -race`?

✅ Da li su optimizacije uvedene tek nakon merenja performansi?

---

# Mentalni model

Nemoj razmišljati:

```
Goroutine A

↓

Goroutine B
```

Razmišljaj:

```
Writer

↓

Synchronization

↓

Happens-Before

↓

Memory Visibility

↓

Reader
```

To je suština Go Memory Model-a.

---

# Kako povezati sve što smo naučili?

```
Compiler

↓

Instruction Reordering

↓

CPU Cache

↓

Memory Visibility

↓

Synchronization

↓

Happens-Before

↓

Correct Concurrent Program
```

Svaka karika u ovom lancu je važna.

---

# Šta nosiš iz ovog modula?

Trebalo bi da možeš da objasniš:

- zašto običan upis nije dovoljan,
- zašto postoji Memory Visibility,
- šta je Instruction Reordering,
- šta je Happens-Before,
- kako `Mutex`, `channels` i `atomic` obezbeđuju potrebne garancije,
- kako analizirati ispravnost konkurentnog programa.

Ako to možeš, razumeš suštinu Go Memory Model-a.

---

# 📋 Rezime Modula #4.3

U ovom modulu naučili smo:

✅ šta je Go Memory Model

✅ šta znači Memory Visibility

✅ kako CPU cache i optimizacije utiču na konkurentni kod

✅ šta je Instruction Reordering

✅ šta je Happens-Before

✅ pravila za `Mutex`, `RWMutex`, `channels`, `sync.Once` i `sync/atomic`

✅ metodologiju za analizu konkurentnog programa

---

### ➡️ Sledeća lekcija **[**Lock-Free Programming**](04-lock-free-programming.md)**

Obuhvatiće:

- šta znači *lock-free* algoritam,
- razlika između *blocking*, *lock-free* i *wait-free* pristupa,
- ABA problem,
- CAS petlje (*Compare-And-Swap loops*),
- implementacija lock-free struktura podataka,
- prednosti, ograničenja i kada ih koristiti u praksi.

Ovaj modul predstavlja ulazak u napredne tehnike konkurentnog programiranja koje se koriste u sistemima sa veoma visokim zahtevima za performansama.