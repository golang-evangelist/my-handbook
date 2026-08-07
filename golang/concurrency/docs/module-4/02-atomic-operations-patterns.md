# Atomic Operations Patterns — Atomic Pointer

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/02-atomic-operations-patterns.md`

---

# 📚 Sadržaj

- Zašto postoji `atomic.Pointer`
- Problem deljenja pokazivača
- Atomic zamena objekta
- Read-Mostly sistemi
- Prednosti u odnosu na Mutex
- Ograničenja

---

# Uvod

U prethodnom modulu koristili smo atomic operacije nad:

- `int32`
- `int64`
- `uint64`

Ali šta ako želimo da bezbedno delimo čitav objekat?

Na primer:

```go
type Config struct {
	MaxConnections int
	Timeout        time.Duration
}
```

Više Goroutines želi da čita konfiguraciju, dok je druga povremeno menja.

Kako to rešiti?

---

# Klasično rešenje

Koristimo `sync.RWMutex`.

```go
type ConfigStore struct {
	mu     sync.RWMutex
	config *Config
}
```

Čitanje:

```go
store.mu.RLock()
cfg := store.config
store.mu.RUnlock()
```

Pisanje:

```go
store.mu.Lock()
store.config = newConfig
store.mu.Unlock()
```

Ovo je potpuno ispravno.

Ali kod sistema sa ogromnim brojem čitanja može postojati bolje rešenje.

---

# Atomic Pointer

Go uvodi generički tip:

```go
atomic.Pointer[T]
```

On omogućava:

- atomic učitavanje pokazivača,
- atomic zamenu pokazivača,
- bez korišćenja `Mutex`-a za samo čitanje.

---

# Primer

```go
type Config struct {
	MaxConnections int
}

var config atomic.Pointer[Config]
```

Postavljanje početne vrednosti:

```go
config.Store(
	&Config{
		MaxConnections: 100,
	},
)
```

Čitanje:

```go
cfg := config.Load()

fmt.Println(cfg.MaxConnections)
```

---

# Šta se zapravo menja?

Važno je razumeti da se **ne menja sadržaj objekta**.

Menja se:

```
pokazivač

↓

novi objekat
```

Drugim rečima:

```
stari Config

↓

novi Config
```

Sve Goroutines koje pozovu `Load()` dobiće pokazivač na jednu konzistentnu verziju konfiguracije.

---

# Read-Mostly Pattern

`atomic.Pointer` je odličan kada imamo:

```
mnogo čitanja

↓

vrlo malo pisanja
```

Primeri:

- konfiguracija aplikacije,
- feature flags,
- routing tabela,
- cache metapodaci.

---

# Prednosti

✅ Čitanje ne koristi lock.

✅ Reader-i se međusobno ne blokiraju.

✅ Writer menja samo pokazivač.

✅ Svaki reader vidi konzistentan objekat.

---

# Važno ograničenje

Ovo nije bezbedno:

```go
cfg := config.Load()

cfg.MaxConnections++
```

Zašto?

Zato što sada menjamo objekat na koji pokazivač pokazuje.

`atomic.Pointer` štiti samo zamenu pokazivača, **ne i mutaciju objekta**.

---

# Ispravan pristup

Napravimo novu vrednost:

```go
old := config.Load()

newCfg := &Config{
	MaxConnections: old.MaxConnections + 100,
}

config.Store(newCfg)
```

Stari objekat ostaje nepromenjen.

Svi novi reader-i dobijaju novu verziju.

---

# Mentalni model

Umesto:

```
izmeni postojeći objekat
```

razmišljaj:

```
napravi novi objekat

↓

zameni pokazivač
```

To je osnova mnogih visoko konkurentnih sistema.

---

# Kada koristiti `atomic.Pointer`?

Odličan izbor za:

- immutable konfiguracije,
- read-mostly podatke,
- snapshot objekte,
- globalna podešavanja.

Nije dobar izbor za:

- objekte koji se često menjaju,
- kompleksne transakcije,
- delimične izmene velikih struktura.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je `atomic.Pointer[T]`

✅ kako radi `Load()` i `Store()`

✅ zašto je pogodan za read-mostly sisteme

✅ razliku između zamene pokazivača i izmene objekta

✅ osnovu immutable snapshot pristupa

---

# Atomic Operations Patterns — atomic.Value

---

# 📚 Sadržaj

- Šta je `atomic.Value`
- Kada koristiti `atomic.Value`
- `Load()` i `Store()`
- Pravila korišćenja
- `atomic.Value` vs `atomic.Pointer`
- Production primeri
- Najčešće greške

---

# Uvod

Do sada smo koristili:

```go
atomic.Pointer[T]
```

On atomski menja:

```
pokazivač
```

Sada upoznajemo:

```go
atomic.Value
```

On atomski menja:

```
celu vrednost
```

---

# Šta je `atomic.Value`?

`atomic.Value` predstavlja kontejner koji omogućava:

- atomic upis vrednosti
- atomic čitanje vrednosti

pri čemu su svi reader-i potpuno bezbedni i ne koriste `Mutex`.

---

Model:

```
Store()

↓

nova vrednost

↓

Load()

↓

trenutna verzija
```

---

# Prvi primer

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {

	var v atomic.Value

	v.Store("Hello")

	fmt.Println(v.Load())

}
```

Rezultat:

```
Hello
```

---

# Čuvanje strukture

```go
type Config struct {
	Timeout int
	MaxConn int
}
```

Čuvanje:

```go
var config atomic.Value

config.Store(
	Config{
		Timeout: 30,
		MaxConn: 100,
	},
)
```

Čitanje:

```go
cfg :=
	config.Load().(Config)

fmt.Println(cfg.Timeout)
```

---

# Type Assertion

`Load()` vraća:

```go
any
```

Zato je potrebno:

```go
cfg := value.Load().(Config)
```

ili:

```go
cfg, ok := value.Load().(Config)
```

---

# Pravilo #1

Prva vrednost određuje tip.

Primer:

```go
var value atomic.Value

value.Store(100)
```

Od tog trenutka:

```
tip = int
```

---

Ispravno:

```go
value.Store(200)
```

---

Pogrešno:

```go
value.Store("hello")
```

Rezultat:

```
panic
```

---

# Pravilo #2

Ne sme se pozvati:

```go
Load()
```

pre nego što postoji prvi:

```go
Store()
```

Uvek prvo inicijalizuj vrednost.

---

# Zašto postoji `atomic.Value`?

Zamisli aplikaciju sa:

```
1000 reader-a

↓

1 writer
```

Writer povremeno ažurira konfiguraciju.

Reader-i samo čitaju.

`Mutex` bi radio ispravno.

Ali:

```
svako čitanje

↓

RLock()

↓

RUnlock()
```

Kod `atomic.Value`:

```
Load()

↓

gotovo
```

Nema zaključavanja.

---

# Production primer

Konfiguracija:

```go
type Config struct {
	LogLevel string
	Timeout  int
}
```

Globalna vrednost:

```go
var config atomic.Value
```

Pokretanje:

```go
config.Store(
	Config{
		LogLevel: "INFO",
		Timeout: 30,
	},
)
```

Reader:

```go
cfg := config.Load().(Config)

fmt.Println(cfg.LogLevel)
```

Writer:

```go
config.Store(
	Config{
		LogLevel: "DEBUG",
		Timeout: 60,
	},
)
```

Reader-i odmah vide novu verziju.

---

# `atomic.Pointer` vs `atomic.Value`

## `atomic.Pointer`

Radi sa:

```go
*Config
```

Prednosti:

- nema type assertion
- generički tip (`atomic.Pointer[T]`)
- veoma efikasan

---

## `atomic.Value`

Radi sa:

```go
any
```

Prednosti:

- može čuvati različite složenije vrednosti jednog istog tipa
- jednostavan API

Mana:

```
Load()

↓

type assertion
```

---

# Kada koristiti koji?

Koristi:

```go
atomic.Pointer[T]
```

kada:

- radiš sa pokazivačima
- želiš generičku sigurnost tipova
- koristiš Go 1.19+

---

Koristi:

```go
atomic.Value
```

kada:

- želiš atomski menjati celu vrednost
- radiš sa immutable objektima
- održavaš kompatibilnost sa starijim kodom

---

# Najčešće greške

## Greška #1

Menjanje objekta nakon `Load()`.

Loše:

```go
cfg := config.Load().(Config)

cfg.Timeout++
```

Ako je u pitanju pokazivač ili deljena mutabilna struktura, ovakva izmena može dovesti do race condition-a.

Bolji pristup:

- napravi kopiju,
- izmeni kopiju,
- pozovi `Store()` sa novom vrednošću.

---

## Greška #2

Promena tipa.

Loše:

```go
value.Store(100)

value.Store("Go")
```

Rezultat:

```
panic
```

---

## Greška #3

Pozivanje:

```go
Load()
```

pre prvog:

```go
Store()
```

Uvek inicijalizuj `atomic.Value` pre nego što ga druge Goroutine koriste.

---

# Tipični production slučajevi

`atomic.Value` se često koristi za:

- konfiguraciju aplikacije,
- routing tabele,
- ACL (Access Control List),
- feature flags,
- read-mostly cache,
- snapshot konfiguracije.

---

# Best Practices

✅ Koristi immutable objekte.

✅ Menjaj celu vrednost pomoću `Store()`.

✅ Ne menjaj sadržaj već učitane vrednosti ako je deljena.

✅ Nakon prvog `Store()` koristi uvek isti tip.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je `atomic.Value`

✅ kako rade `Store()` i `Load()`

✅ pravila tipova

✅ razliku u odnosu na `atomic.Pointer`

✅ tipične production primere

✅ najčešće greške

---

# Atomic Operations Patterns — Immutable Snapshot Pattern

---

# 📚 Sadržaj

- Šta je Immutable Snapshot Pattern
- Mutable vs Immutable objekti
- Snapshot pristup
- Versioning
- Atomic Pointer + Snapshot
- Prednosti i mane
- Production primeri

---

# Uvod

Jedan od najčešćih izvora race condition-a nije:

- `Mutex`
- `atomic`
- `channels`

već:

```
deljenje promenljivih objekata
```

Ako više Goroutines istovremeno menja isti objekat, sinhronizacija postaje neophodna.

Ali postoji drugačiji pristup.

---

# Osnovna ideja

Umesto:

```
izmeni postojeći objekat
```

radi se:

```
napravi novi objekat

↓

zameni referencu

↓

stari objekat ostaje nepromenjen
```

Ovo se naziva:

```
Immutable Snapshot Pattern
```

---

# Mutable objekat

Primer:

```go
type Config struct {
	Timeout int
	LogLevel string
}
```

Loš primer:

```go
cfg.Timeout = 60
cfg.LogLevel = "DEBUG"
```

Ako druga Goroutine istovremeno čita `cfg`, može nastati race condition.

---

# Immutable objekat

Umesto izmene:

```go
cfg.Timeout = 60
```

pravimo novu vrednost:

```go
newCfg := Config{
	Timeout: 60,
	LogLevel: cfg.LogLevel,
}
```

Zatim:

```go
config.Store(newCfg)
```

Stari objekat ostaje netaknut.

---

# Snapshot

Snapshot predstavlja:

```
trenutno stanje sistema
```

Primer:

```
Config v1

↓

Config v2

↓

Config v3
```

Svaka verzija je kompletna i ne menja se.

---

# Vizuelni model

```
Reader #1

↓

Config v1

-------------------

Reader #2

↓

Config v1

-------------------

Writer

↓

Config v2

↓

Store()
```

Novi reader-i dobijaju:

```
Config v2
```

Stari reader-i bezbedno nastavljaju da koriste:

```
Config v1
```

---

# Atomic Pointer + Snapshot

Primer:

```go
type Config struct {
	MaxConn int
	Timeout int
}

var config atomic.Pointer[Config]
```

Inicijalizacija:

```go
config.Store(
	&Config{
		MaxConn: 100,
		Timeout: 30,
	},
)
```

Čitanje:

```go
cfg := config.Load()

fmt.Println(cfg.Timeout)
```

---

# Ažuriranje

Umesto:

```go
cfg.Timeout = 60
```

radi se:

```go
old := config.Load()

newCfg := &Config{
	MaxConn: old.MaxConn,
	Timeout: 60,
}

config.Store(newCfg)
```

Stari objekat ostaje nepromenjen.

---

# Zašto je ovo važno?

Reader nikada ne vidi:

```
polovično ažuriran objekat
```

On vidi ili:

```
stari snapshot
```

ili:

```
novi snapshot
```

Nikada stanje između.

---

# Versioning

Mnogi sistemi čuvaju i verziju:

```go
type Config struct {
	Version int64
	Timeout int
}
```

Nova konfiguracija:

```text
Version 1

↓

Version 2

↓

Version 3
```

To olakšava:

- debugging,
- rollback,
- audit,
- sinhronizaciju između komponenti.

---

# Production primer

Zamislimo API Gateway.

Hiljade request-a u sekundi čitaju routing tabelu.

Administrator menja konfiguraciju jednom u nekoliko minuta.

Umesto zaključavanja:

```
Reader

↓

RLock()

↓

Read()

↓

RUnlock()
```

koristi se:

```
Load()

↓

Snapshot
```

Administrator napravi novu routing tabelu:

```text
Route Table v2
```

i zatim:

```go
routes.Store(newTable)
```

Novi zahtevi koriste novu tabelu.

Stari nastavljaju rad sa starom.

---

# Prednosti

✅ Reader-i nikada ne čekaju.

✅ Nema contention-a među reader-ima.

✅ Svaki reader vidi konzistentan objekat.

✅ Jednostavnije razmišljanje o konkurentnosti.

---

# Mane

## Veća potrošnja memorije

Pri svakoj izmeni pravi se novi objekat.

Ako je objekat veoma veliki, to može povećati broj alokacija.

---

## Potrebno kopiranje

Kod složenih struktura potrebno je pažljivo napraviti novu verziju.

Na primer:

```go
type Config struct {
	Routes map[string]string
}
```

Ne treba kopirati samo pokazivač na mapu, već po potrebi napraviti novu mapu i kopirati njen sadržaj kako bi novi snapshot bio zaista nezavisan.

---

# Kada koristiti?

Odličan izbor za:

- konfiguraciju,
- routing tabele,
- feature flags,
- ACL,
- read-mostly cache,
- metadata.

---

Nije dobar izbor za:

- objekte koji se menjaju hiljadama puta u sekundi,
- veoma velike mutable strukture,
- podatke kod kojih je svaka izmena mala, ali veoma česta.

---

# Best Practices

✅ Tretiraj snapshot kao **read-only**.

✅ Posle `Load()` nemoj menjati objekat.

✅ Za svaku izmenu napravi novu verziju.

✅ Koristi `atomic.Pointer` ili `atomic.Value` za atomsku zamenu snapshot-a.

---

# Mentalni model

Umesto:

```
izmeni objekat
```

razmišljaj:

```
napravi novu verziju

↓

objavi novu verziju

↓

stara verzija ostaje dostupna
```

To je jedan od najčešćih obrazaca u visoko konkurentnim Go servisima.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Immutable Snapshot Pattern

✅ razliku između mutable i immutable objekata

✅ kako funkcioniše snapshot pristup

✅ zašto je važan versioning

✅ kako se koristi zajedno sa `atomic.Pointer`

✅ prednosti i ograničenja ovog obrasca

---

# Atomic Operations Patterns — Copy-On-Write (COW)

---

# 📚 Sadržaj

- Šta je Copy-On-Write
- Veza sa Immutable Snapshot Pattern-om
- Kako funkcioniše COW
- Primer sa map-om
- Primer sa slice-om
- Prednosti i mane
- Kada koristiti COW

---

# Uvod

U prethodnoj lekciji videli smo:

```
Reader

↓

Snapshot
```

Ali ostaje pitanje:

> Kako napraviti novi snapshot?

Odgovor je:

```
Copy-On-Write
```

---

# Šta je Copy-On-Write?

Copy-On-Write (COW) znači:

> **Ne menjaj postojeći objekat.**

Umesto toga:

```
1. Napravi kopiju.

↓

2. Izmeni kopiju.

↓

3. Objavi novu verziju.
```

Stari objekat ostaje nepromenjen.

---

# Vizuelni model

Početno stanje:

```
Config v1
```

Writer želi izmenu:

```
Config v1

↓

Copy

↓

Config v2

↓

Modify

↓

Store()
```

Reader-i:

```
Reader A

↓

Config v1


------------------

Reader B

↓

Config v2
```

Oba rade bezbedno.

---

# Primer sa map-om

Početna konfiguracija:

```go
type Config struct {
	Routes map[string]string
}
```

Loš pristup:

```go
cfg := config.Load()

cfg.Routes["/admin"] = "admin-service"
```

Ovde menjamo mapu koju možda koriste druge Goroutines.

To može dovesti do:

- race condition-a,
- nekonzistentnog stanja,
- panike zbog konkurentnog pristupa mapi.

---

# Ispravan COW pristup

Prvo učitamo trenutni snapshot:

```go
old := config.Load()
```

Napravimo novu mapu:

```go
routes := make(
	map[string]string,
	len(old.Routes),
)

for k, v := range old.Routes {
	routes[k] = v
}
```

Dodamo novu rutu:

```go
routes["/admin"] = "admin-service"
```

Kreiramo novu konfiguraciju:

```go
newCfg := &Config{
	Routes: routes,
}
```

Objavimo novu verziju:

```go
config.Store(newCfg)
```

---

# Zašto kopiramo mapu?

Mapa je referentni tip.

Ovo:

```go
newCfg := &Config{
	Routes: old.Routes,
}
```

ne pravi novu mapu.

Dobijamo:

```
stara konfiguracija

↓

ista mapa

↓

nova konfiguracija
```

Obe konfiguracije dele istu mapu.

To nije immutable snapshot.

---

# Primer sa slice-om

Početno:

```go
type Config struct {
	Servers []string
}
```

Ne raditi:

```go
cfg.Servers = append(
	cfg.Servers,
	"server-4",
)
```

Ako je `cfg` deljen između Goroutines, ovo može izazvati race condition.

---

# COW za slice

Napravimo kopiju:

```go
servers := append(
	[]string(nil),
	old.Servers...,
)
```

Dodamo novi element:

```go
servers = append(
	servers,
	"server-4",
)
```

Nova konfiguracija:

```go
newCfg := &Config{
	Servers: servers,
}
```

Objavimo:

```go
config.Store(newCfg)
```

---

# Zašto ovo radi?

Reader-i koriste:

```
Servers v1
```

Writer pravi:

```
Servers v2
```

Ne postoji deljenje mutable slice-a.

---

# Prednosti

## Reader-i nikada ne čekaju

Nema:

```go
RLock()

RUnlock()
```

---

## Jednostavno razmišljanje

Reader vidi:

- staru verziju ili
- novu verziju.

Nikada nešto između.

---

## Nema race condition-a

Ako se svi pridržavaju pravila da se snapshot ne menja nakon objavljivanja.

---

# Mane

## Veće alokacije

Svaka izmena pravi novu kopiju.

---

## Veća potrošnja memorije

Kratko vreme postoje:

```
Config v1

+

Config v2
```

Garbage Collector će kasnije osloboditi staru verziju kada više nema referenci na nju.

---

## Skupo za velike objekte

Ako konfiguracija ima:

- desetine hiljada ruta,
- velike mape,
- velike slice-ove,

kopiranje može biti značajan trošak.

---

# Kada koristiti COW?

Odličan izbor za:

- konfiguraciju,
- routing tabele,
- read-mostly cache,
- ACL,
- feature flags,
- metadata.

---

# Kada ga izbegavati?

Loš izbor za:

- objekte koji se menjaju veoma često,
- velike mutable kolekcije,
- sisteme sa mnogo writer-a.

---

# Snapshot + COW

Ova dva obrasca gotovo uvek idu zajedno:

```
Writer

↓

Copy-On-Write

↓

Atomic Store

↓

Reader

↓

Immutable Snapshot
```

To je jedan od najčešćih obrazaca u konkurentnim Go servisima.

---

# Best Practices

✅ Kopiraj sve referentne tipove (`map`, `slice`) koje želiš da izmeniš.

✅ Ne menjaj snapshot nakon što je objavljen.

✅ Reader-i treba da tretiraju snapshot kao **read-only**.

✅ Koristi `atomic.Pointer` ili `atomic.Value` za objavljivanje nove verzije.

---

# Mentalni model

Nemoj razmišljati:

```
izmeni postojeći objekat
```

Razmišljaj:

```
napravi kopiju

↓

izmeni kopiju

↓

objavi novu verziju
```

Time dobijaš:

- jednostavnije razmišljanje,
- manje contention-a,
- bezbedno konkurentno čitanje.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Copy-On-Write

✅ razliku između COW i Immutable Snapshot obrasca

✅ kako pravilno kopirati map-e

✅ kako pravilno kopirati slice-ove

✅ prednosti i ograničenja COW pristupa

✅ kada koristiti ovaj obrazac

---

# Atomic Operations Patterns — Read-Mostly Systems

---

# 📚 Sadržaj

- Šta je Read-Mostly sistem
- Read-Heavy vs Write-Heavy
- Izbor mehanizma sinhronizacije
- `RWMutex` vs `atomic.Pointer` vs `atomic.Value`
- Production primeri
- Prednosti i ograničenja

---

# Uvod

Ne ponašaju se svi sistemi isto.

Neki imaju:

```
10 čitanja

↓

1000 pisanja
```

Drugi imaju:

```
1 pisanje

↓

1 000 000 čitanja
```

Ta razlika potpuno menja izbor mehanizma sinhronizacije.

---

# Šta znači Read-Mostly?

Read-Mostly sistem je sistem u kome:

```
broj čitanja

≫

broj pisanja
```

("≫" znači "mnogo veći od".)

Primer:

```
1 update / minut

↓

500 000 read operacija
```

---

# Tipični primeri

- konfiguracija aplikacije,
- routing tabele,
- feature flags,
- ACL,
- DNS cache,
- CDN konfiguracija,
- metadata.

---

# Read-Heavy vs Write-Heavy

## Read-Heavy

Primer:

```
Reads:

999 999

Writes:

1
```

Ovde želimo da čitanje bude što je moguće jeftinije.

---

## Write-Heavy

Primer:

```
Reads:

100

Writes:

50 000
```

Ovde je važnije optimizovati upis.

---

# Zašto je ovo važno?

Ako koristimo:

```go
RWMutex
```

svako čitanje radi:

```go
RLock()

...

RUnlock()
```

To je brzo.

Ali nije besplatno.

---

Kod:

```go
atomic.Pointer
```

reader radi:

```go
Load()
```

i nastavlja dalje.

Nema zaključavanja.

---

# Poređenje

## `RWMutex`

Reader:

```
RLock()

↓

Read()

↓

RUnlock()
```

---

Writer:

```
Lock()

↓

Write()

↓

Unlock()
```

---

Prednost:

jednostavna implementacija.

---

# `atomic.Pointer`

Reader:

```
Load()

↓

Read
```

---

Writer:

```
Copy-On-Write

↓

Store()
```

---

Prednost:

reader nikada ne čeka.

---

# `atomic.Value`

Veoma sličan pristup:

```
Load()

↓

Snapshot
```

Razlika je u API-ju i načinu rada sa tipovima.

---

# Vizuelno poređenje

## RWMutex

```
Reader A

↓

RLock


Reader B

↓

RLock


Writer

↓

čeka
```

Writer mora da sačeka da svi reader-i završe.

---

## Atomic Snapshot

```
Reader A

↓

Config v1


Reader B

↓

Config v1


Writer

↓

Config v2

↓

Store()
```

Writer ne blokira postojeće reader-e.

Novi reader-i odmah dobijaju novu verziju.

---

# Production primer 1

## HTTP konfiguracija

Hiljade request-a u sekundi čitaju:

```go
cfg.Timeout
```

Administrator jednom dnevno promeni timeout.

Idealno rešenje:

```
atomic.Pointer
```

---

# Production primer 2

## Feature Flags

Svaki request proverava:

```go
feature.Enabled
```

Promene su retke.

Reader-a ima mnogo.

Odličan izbor:

```
atomic.Value

ili

atomic.Pointer
```

---

# Production primer 3

## Routing tabela

API Gateway.

Svaki request:

```
Load()

↓

Route Lookup
```

Nova tabela se objavljuje:

```
Store()
```

Nema zaključavanja tokom čitanja.

---

# Kada RWMutex ostaje bolji izbor?

Ako se stanje često menja.

Primer:

```
1000 writes/s

1000 reads/s
```

Pravljenje kopije cele strukture za svaku izmenu može biti skuplje od korišćenja `RWMutex`.

---

# Decision Matrix

| Scenario | Preporuka |
|----------|-----------|
| Jedan brojač | `atomic.Add` |
| Status ili flag | `atomic.Load` / `Store` |
| Read-mostly konfiguracija | `atomic.Pointer` |
| Immutable snapshot | `atomic.Pointer` ili `atomic.Value` |
| Česta izmena složenih struktura | `RWMutex` |
| Deljena mapa | `RWMutex` |
| Worker komunikacija | `channels` |

---

# Trade-off

## `RWMutex`

Prednosti:

- jednostavan
- poznat
- odličan za mutable stanje

Mane:

- reader-i i writer-i međusobno utiču jedni na druge.

---

## `atomic.Pointer`

Prednosti:

- veoma brzo čitanje
- nema zaključavanja reader-a
- snapshot arhitektura

Mane:

- potrebno kopiranje pri svakoj izmeni
- više alokacija
- zahteva immutable pristup

---

# Kako razmišljati?

Nemoj prvo pitati:

> "Šta je brže?"

Prvo pitaj:

> "Koliko često čitam, a koliko često pišem?"

Odgovor na to pitanje često određuje najbolji izbor mehanizma.

---

# Best Practices

✅ Ako imaš mnogo više čitanja nego pisanja, razmotri snapshot pristup.

✅ Ako su izmene česte i male, `RWMutex` je često jednostavnije i efikasnije rešenje.

✅ Nemoj koristiti Copy-On-Write za velike objekte koji se neprestano menjaju.

✅ Profiliši aplikaciju pre optimizacije.

---

# Mentalni model

```
Read-Heavy

↓

Atomic Snapshot

↓

Load()

----------------------------

Write-Heavy

↓

RWMutex

↓

Lock()
```

Ne postoji univerzalno najbolje rešenje.

Najbolje rešenje zavisi od obrasca pristupa podacima.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Read-Mostly sistem

✅ razliku između Read-Heavy i Write-Heavy scenarija

✅ kada koristiti `RWMutex`

✅ kada koristiti `atomic.Pointer`

✅ kada koristiti `atomic.Value`

✅ kako izabrati odgovarajući obrazac za produkcioni sistem

---
# Atomic Operations Patterns — Best Practices i Production smernice

---

# 📚 Sadržaj

- Atomic mindset
- Decision checklist
- Najčešće greške
- Production preporuke
- Performance smernice
- Decision tree
- Završni rezime

---

# Uvod

Posle dva modula o `sync/atomic`, trebalo bi da razmišljaš ovako:

```
Problem

↓

Ko poseduje podatke?

↓

Koliko ima reader-a?

↓

Koliko ima writer-a?

↓

Da li postoji deljeno mutable stanje?

↓

Izbor odgovarajuće sinhronizacije
```

Atomic operacije nisu cilj.

One su samo jedan od alata.

---

# Atomic Mindset

Umesto pitanja:

> "Kako da izbacim Mutex?"

postavi pitanje:

> "Da li uopšte moram da delim mutable stanje?"

Ako je odgovor:

```
Ne.
```

onda razmisli o:

- Channels
- Ownership Pattern
- Immutable Snapshot

Ako je odgovor:

```
Da.
```

onda biraj između:

- Atomic
- Mutex
- RWMutex

---

# Decision Checklist

Pre nego što napišeš konkurentni kod, postavi sebi sledeća pitanja.

---

## 1. Da li delim stanje?

```
NE

↓

Ne treba sinhronizacija.
```

---

## 2. Da li delim jednu vrednost?

Primer:

```
counter

flag

status
```

Koristi:

```
atomic
```

---

## 3. Da li delim kompleksan objekat?

Primer:

```go
User

Cache

Session

Account
```

Koristi:

```
Mutex

ili

RWMutex
```

---

## 4. Da li imam mnogo više čitanja nego pisanja?

Ako:

```
Reads

≫

Writes
```

razmotri:

```
Immutable Snapshot

+

Copy-On-Write

+

atomic.Pointer

ili

atomic.Value
```

---

## 5. Da li Goroutines treba da razmenjuju posao?

Ako je odgovor:

```
DA
```

koristi:

```
channels
```

---

# Najčešće greške

## Greška #1

Korišćenje `atomic` za kompleksne operacije.

Loše:

```go
balance += amount

history = append(history, tx)
```

Ove dve operacije čine jednu logičku celinu.

Ovde je potreban `Mutex`.

---

## Greška #2

Menjanje immutable snapshot-a.

Loše:

```go
cfg := config.Load()

cfg.Timeout = 60
```

Time narušavaš pretpostavku da je snapshot nepromenljiv.

---

## Greška #3

Deljenje mutable `map` ili `slice` bez zaštite.

Loše:

```go
sharedMap[key] = value
```

Ako više Goroutines pristupa istoj mapi, potreban je odgovarajući mehanizam sinhronizacije ili drugačiji dizajn.

---

## Greška #4

Optimizacija bez merenja.

Pretpostavka:

```
Atomic je sigurno brži.
```

Nije uvek tačno.

Prvo meri.

Tek onda optimizuj.

---

## Greška #5

Prerana optimizacija.

Mnogo puta:

```go
Mutex
```

je:

- jednostavniji,
- čitljiviji,
- lakši za održavanje.

---

# Production preporuke

## Koristi `atomic`

Za:

- metrics,
- counters,
- feature flags,
- state machine,
- immutable snapshot.

---

## Koristi `Mutex`

Za:

- više povezanih promenljivih,
- transakcije,
- kompleksne objekte,
- održavanje invarianti.

---

## Koristi `RWMutex`

Za:

- deljene mape,
- cache,
- read-heavy mutable podatke.

---

## Koristi `channels`

Za:

- Worker Pool,
- Pipeline,
- Fan-In,
- Fan-Out,
- Producer-Consumer,
- Event Processing.

---

# Performance smernice

Pre optimizacije proveri:

```
Da li postoji contention?

↓

Da li postoji race?

↓

Da li postoji bottleneck?

↓

Da li benchmark potvrđuje problem?
```

Tek tada razmišljaj o prelasku sa `Mutex` na `atomic` ili o promeni arhitekture.

---

# Decision Tree

```
Deljeno stanje?

│
├── NE
│     │
│     └── Bez sinhronizacije
│
└── DA
      │
      ▼

Jedna vrednost?

│
├── DA
│     │
│     └── atomic
│
└── NE
      │
      ▼

Kompleksan objekat?

│
├── DA
│     │
│     └── Mutex / RWMutex
│
└── NE
      │
      ▼

Komunikacija između Goroutines?

│
├── DA
│     │
│     └── Channels
│
└── Read-Mostly?

      │
      ├── DA
      │     │
      │     └── Snapshot + Copy-On-Write
      │
      └── NE
            │
            └── RWMutex
```

---

# Mentalni model

Zapamti sledeće:

```
Atomic

↓

štiti jednu vrednost


----------------------------

Mutex

↓

štiti konzistentnost stanja


----------------------------

Channels

↓

prenose vlasništvo nad podacima


----------------------------

Snapshot

↓

eliminiše mutable deljenje
```

Svaki od ovih pristupa rešava drugačiji problem.

---

# Šta očekivati od Senior Go programera?

Senior Go programer:

✅ ne bira `atomic` zato što je "brži"

✅ ne bira `channels` zato što su "Go način"

✅ ne bira `Mutex` zato što je "najlakši"

Već:

- analizira obrazac pristupa podacima,
- razume odnose između reader-a i writer-a,
- bira najjednostavnije rešenje koje zadovoljava zahteve za ispravnošću, performansama i održavanjem.

---

# 📋 Rezime Modula #4.2

U ovoj lekciji naučili smo:

✅ `atomic.Pointer`

✅ `atomic.Value`

✅ Immutable Snapshot Pattern

✅ Copy-On-Write Pattern

✅ Read-Mostly arhitekture

✅ production smernice

✅ decision tree za izbor odgovarajućeg mehanizma sinhronizacije

---

### ➡️ Sledeća lekcija **[**Go Memory Model**](03-go-memory-model.md)**

[!NOTE] Sledeći modul je jedan od najvažnijih u celom kursu:

Obuhvatiće:

- šta znači "memory visibility",
- šta znači "ordering",
- kako CPU i kompajler mogu da promene redosled izvršavanja,
- memory barriers,
- happens-before kao formalni model,
- zašto konkurentan kod može izgledati ispravno, a ipak biti pogrešan.

Ovaj modul predstavlja teorijsku osnovu koja objašnjava **zašto** `Mutex`, `atomic` i `channels` rade ispravno, a ne samo **kako** se koriste.