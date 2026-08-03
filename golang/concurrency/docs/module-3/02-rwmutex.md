# `sync.RWMutex` — Read/Write zaključavanje u Go Concurrency modelu

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto postoji `sync.RWMutex`
- Problem klasičnog `Mutex`-a
- Read-heavy sistemi
- Koncept Read Lock i Write Lock
- Razlika između čitanja i pisanja
- Kako radi `RWMutex`
- Mentalni model
- Kada razmišljati o `RWMutex`-u
- Prvi primer

---

# Uvod

U prethodnoj lekciji naučili smo:

```go
sync.Mutex
```

koji omogućava:

```text
jedna Goroutine u Critical Section-u
```

---

Primer:

```go
mu.Lock()

value++

mu.Unlock()
```

---

Ovo rešava problem:

```text
više Goroutines menjaju isti podatak
```

---

Ali postoji jedan problem.

---

# Problem običnog `Mutex`-a

Pretpostavimo da imamo podatak:

```go
config := Configuration{}
```

Većina operacija je:

```go
Read
```

Na primer:

```go
GetConfig()
```

---

Samo retko:

```go
UpdateConfig()
```

menja podatak.

---

# Scenario

Imamo:

```text
1000 Goroutines
```

koje čitaju:

```go
config.Timeout
```

i:

```text
1 Goroutine
```

koja menja:

```go
config.Timeout = 5
```

---

Sa običnim `Mutex`-om:

```go
mu.Lock()

read()

mu.Unlock()
```

---

Dobijamo:

```text
Reader 1

LOCK


Reader 2

ČEKA


Reader 3

ČEKA
```

---

# Problem

Čitanje ne menja podatak.

Tehnički:

više čitalaca može bezbedno da čita istovremeno.

---

Primer:

```text
Reader A

        \
         \
          Config
         /
        /

Reader B
```

Nema konflikta.

---

Ali `Mutex` ne zna razliku između:

```text
Read

i

Write
```

Za njega je sve:

```text
Exclusive access
```

---

# Rešenje

Go uvodi:

```go
sync.RWMutex
```

---

RW znači:

```text
Read / Write
```

---

Ideja:

Postoje dva tipa zaključavanja:

## Read Lock

```go
RLock()
```

više čitalaca može istovremeno.

---

## Write Lock

```go
Lock()
```

samo jedan pisac.

---

# Mentalni model

Običan Mutex:

```text
Mutex


Reader

Writer

Reader

Writer


svi čekaju
```

---

RWMutex:

```text
          RWMutex


Reader 1  \
Reader 2   >  mogu zajedno
Reader 3  /


Writer

čeka sve
```

---

# Pravila `RWMutex`-a

Postoje dve osnovne operacije.

---

# Čitanje

Koristi:

```go
RLock()
```

i:

```go
RUnlock()
```

Primer:

```go
rw.RLock()

value := data

rw.RUnlock()
```

---

# Pisanje

Koristi:

```go
Lock()
```

i:

```go
Unlock()
```

Primer:

```go
rw.Lock()

data = newValue

rw.Unlock()
```

---

# Važna razlika

`RWMutex` nije:

```text
brži Mutex
```

---

On je:

```text
specijalizovan Mutex
```

za slučajeve gde imamo:

```text
mnogo čitanja

malo pisanja
```

---

# Prvi primer

Pretpostavimo:

```go
type Config struct {

	rw sync.RWMutex

	value int

}
```

---

# Read metoda

```go
func (c *Config) Get() int {

	c.rw.RLock()

	defer c.rw.RUnlock()

	return c.value

}
```

---

# Write metoda

```go
func (c *Config) Set(value int) {

	c.rw.Lock()

	defer c.rw.Unlock()

	c.value = value

}
```

---

# Šta se sada događa?

Ako imamo:

```text
Reader 1

Reader 2

Reader 3
```

svi mogu:

```go
RLock()
```

istovremeno.

---

Ali kada dođe:

```text
Writer
```

on mora čekati:

```text
Reader 1 završava

Reader 2 završava

Reader 3 završava

↓

Writer dobija Lock
```

---

# Vizuelni prikaz

## Samo čitanje

```text
RWMutex

+----------------+

R1  R2  R3

+----------------+
```

---

## Pisanje

```text
RWMutex

+----------------+

        W

+----------------+
```

---

# Zašto ne koristiti uvek RWMutex?

Zato što ima dodatnu kompleksnost.

Običan:

```go
sync.Mutex
```

je često:

- jednostavniji,
- predvidljiviji,
- dovoljno brz.

---

# Primer lošeg izbora

Imamo:

```go
counter++
```

---

Operacije:

```text
50% read

50% write
```

---

`RWMutex` nema veliku korist.

---

Bolje:

```go
sync.Mutex
```

---

# Primer dobrog izbora

Cache:

```text
99% Get()

1% Set()
```

---

Tu:

```go
sync.RWMutex
```

ima smisla.

---

# Poređenje

| | Mutex | RWMutex |
|-|-|-|
| Read | Exclusive | Concurrent |
| Write | Exclusive | Exclusive |
| Kompleksnost | manja | veća |
| Najbolji slučaj | mešani pristup | mnogo čitanja |
| API | Lock/Unlock | RLock/RUnlock + Lock/Unlock |

---

# Mentalni model za pamćenje

`Mutex` kaže:

> Samo jedna Goroutine sme unutra.

---

`RWMutex` kaže:

> Više čitalaca sme unutra, ali pisac mora biti sam.

---

# Kada razmišljati o `RWMutex`?

Kada vidiš:

```text
Shared data

+

mnogo čitanja

+

malo promena
```

Primeri:

- konfiguracije,
- cache,
- registri,
- in-memory baze,
- metadata.

---

# Kada NE razmišljati o `RWMutex`?

Ako imaš:

```text
često pisanje
```

ili:

```text
mala Critical Section
```

običan:

```go
sync.Mutex
```

je često bolji.

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto postoji `sync.RWMutex`,
- koji problem rešava u odnosu na `Mutex`,
- razliku između Read i Write operacija,
- koncept `RLock()` i `Lock()`,
- kada RWMutex ima smisla,
- zašto RWMutex nije automatski bolji od Mutex-a.

---

# `sync.RWMutex` — Read Lock i Write Lock detaljno

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- `RLock()` — Read Lock
- `RUnlock()` — oslobađanje Read Lock-a
- `Lock()` — Write Lock
- `Unlock()` — oslobađanje Write Lock-a
- Kako Reader-i rade paralelno
- Kako Writer dobija ekskluzivan pristup
- Writer priority
- Primer izvršavanja
- Najčešće greške

---

# `RLock()` — Read Lock

Kada Goroutine samo čita podatak:

```go
rw.RLock()
```

---

Primer:

```go
func (c *Config) Get() int {

	c.rw.RLock()

	defer c.rw.RUnlock()

	return c.value

}
```

---

# Šta znači `RLock()`?

Poruka `RWMutex`-u:

> "Neću menjati podatak. Samo želim da ga pročitam."

---

# Više Reader-a istovremeno

Za razliku od `Mutex`-a:

```go
mu.Lock()
```

gde samo jedna Goroutine ulazi,

`RWMutex` dozvoljava:

```text
Reader 1

Reader 2

Reader 3

Reader 4
```

istovremeno.

---

# Primer

Imamo:

```go
config := Config{
	value: 10,
}
```

Tri Goroutines:

```go
go config.Get()

go config.Get()

go config.Get()
```

---

Izvršavanje:

```text
G1

RLock()
 |
 čita


G2

RLock()
 |
 čita


G3

RLock()
 |
 čita
```

---

Sve tri rade paralelno.

---

# `RUnlock()`

Kada završimo čitanje:

```go
rw.RUnlock()
```

oslobađamo Read Lock.

---

Primer:

```go
func (c *Config) Get() int {

	c.rw.RLock()

	defer c.rw.RUnlock()

	return c.value

}
```

---

# Zašto je potreban `RUnlock()`?

Zato što `RWMutex` mora znati:

koliko Reader-a trenutno koristi podatak.

---

Interno stanje:

```text
Reader count = 3
```

Kada:

```go
RUnlock()
```

pozove poslednji Reader:

```text
Reader count = 0
```

Writer može dobiti pristup.

---

# `Lock()` — Write Lock

Kada menjamo podatak:

```go
rw.Lock()
```

---

Primer:

```go
func (c *Config) Set(value int) {

	c.rw.Lock()

	defer c.rw.Unlock()

	c.value = value

}
```

---

# Šta znači `Lock()`?

Poruka:

> "Menjam podatak i niko drugi ne sme pristupiti dok završim."

---

# Writer je ekskluzivan

Dok Writer drži Lock:

```text
Writer

↓

podatak
```

nema:

- Reader-a,
- drugog Writer-a.

---

# Primer

Početno:

```text
RWMutex = slobodan
```

Dolazi:

```text
Writer
```

uzima:

```go
Lock()
```

---

Stanje:

```text
Writer

LOCKED

Reader ❌

Writer ❌
```

---

# Reader dok Writer radi

Ako Reader pozove:

```go
RLock()
```

dok Writer drži Lock:

ne može da nastavi.

Čeka.

---

# Primer

```text
Writer

Lock()

     |
     |
  menja podatak
     |
     |

Unlock()


Reader

čeka
```

---

# Writer dok Reader-i rade

Sada obrnuta situacija.

Imamo:

```text
Reader 1

Reader 2

Reader 3
```

svi imaju:

```go
RLock()
```

---

Dolazi:

```text
Writer
```

---

Writer poziva:

```go
Lock()
```

---

Rezultat:

Writer čeka.

---

Stanje:

```text
Reader 1  RUNNING

Reader 2  RUNNING

Reader 3  RUNNING


Writer    WAITING
```

---

Kada svi Reader-i završe:

```go
RUnlock()
```

Writer dobija pristup.

---

# Writer Priority

Važan detalj.

Stara implementacija mnogih Read/Write lock sistema imala je problem:

Ako stalno dolaze novi Reader-i:

```text
R1

R2

R3

R4

R5
```

Writer može čekati veoma dugo.

---

To se zove:

> Writer starvation

---

# Primer starvation problema

Zamislimo:

```text
Reader 1

Reader 2

Reader 3

Reader 4

...
```

uvek postoji bar jedan Reader.

Writer:

```text
čekam...
```

---

U modernom Go runtime-u `sync.RWMutex` ima ponašanje koje sprečava beskonačno gladovanje Writer-a.

Kada Writer čeka:

novi Reader-i ne dobijaju beskonačno prednost.

---

# Pojednostavljen model

Bez Writer prioriteta:

```text
Readers
Readers
Readers
Readers
Readers

Writer čeka zauvek
```

---

Sa Writer prioritizacijom:

```text
Readers

↓

Writer

↓

novi Readers
```

---

# Primer kompletnog tipa

```go
type Counter struct {

	rw sync.RWMutex

	value int

}
```

---

Read metoda:

```go
func (c *Counter) Value() int {

	c.rw.RLock()

	defer c.rw.RUnlock()

	return c.value

}
```

---

Write metoda:

```go
func (c *Counter) Increment() {

	c.rw.Lock()

	defer c.rw.Unlock()

	c.value++

}
```

---

# Pravilo za korišćenje

## Za čitanje:

Koristi:

```go
RLock()

RUnlock()
```

---

## Za promenu:

Koristi:

```go
Lock()

Unlock()
```

---

# Najčešće greške

---

# ❌ Greška 1

Koristiti:

```go
Lock()
```

za čitanje.

Primer:

```go
rw.Lock()

value := data

rw.Unlock()
```

Radi.

Ali gubiš prednost RWMutex-a.

---

# ❌ Greška 2

Koristiti:

```go
RLock()
```

za promenu.

Primer:

```go
rw.RLock()

data++

rw.RUnlock()
```

Ovo je pogrešno.

Više Reader-a može menjati isti podatak.

---

# ❌ Greška 3

Zaboravljen:

```go
RUnlock()
```

Rezultat:

Reader ostaje aktivan.

Writer može biti blokiran.

---

# ❌ Greška 4

Promena podatka unutar Read Lock-a

Loše:

```go
rw.RLock()

cache[key] = value

rw.RUnlock()
```

---

Ispravno:

```go
rw.Lock()

cache[key] = value

rw.Unlock()
```

---

# Mentalni model

Zapamti:

```text
RLock()

=
"gledam"


Lock()

=
"menjam"
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako rade `RLock()` i `RUnlock()`,
- kako rade `Lock()` i `Unlock()` kod `RWMutex`-a,
- zašto Reader-i mogu raditi paralelno,
- zašto Writer zahteva ekskluzivan pristup,
- kako izgleda Writer priority ponašanje,
- koje su najčešće greške.

---

# `sync.RWMutex` — Praktični obrasci upotrebe

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto se `RWMutex` najčešće koristi u realnim sistemima
- Thread-safe cache
- Zaštita konfiguracije
- Registry pattern
- In-memory storage
- Analiza dizajna
- Kada `RWMutex` ima stvarnu korist

---

# Uvod

Teorijski smo naučili:

```go
RLock()
```

koristimo za:

```text
čitanje
```

a:

```go
Lock()
```

za:

```text
promenu
```

---

Ali pravo pitanje je:

> Gde se `RWMutex` koristi u stvarnim aplikacijama?

Najčešći slučajevi:

- cache,
- konfiguracije,
- registri,
- metadata,
- lokalna memorijska skladišta.

---

# Primer 1 — Thread-safe Cache

Cache je idealan primer za `RWMutex`.

Zašto?

Zato što je odnos:

```text
Read >> Write
```

---

Primer:

```text
10000 Get()

100 Set()
```

---

To je scenario gde:

```go
RWMutex
```

ima smisla.

---

# Nezaštićen cache

```go
type Cache struct {

	data map[string]string

}
```

---

Problem:

Jedna Goroutine:

```go
cache.data["user"] = "Marko"
```

Druga:

```go
value := cache.data["user"]
```

---

Mogući konflikt.

---

# Rešenje

```go
type Cache struct {

	rw sync.RWMutex

	data map[string]string

}
```

---

# Inicijalizacija

```go
func NewCache() *Cache {

	return &Cache{

		data: make(map[string]string),

	}

}
```

---

# Read metoda

```go
func (c *Cache) Get(
	key string,
) (string, bool) {


	c.rw.RLock()

	defer c.rw.RUnlock()


	value, ok := c.data[key]


	return value, ok

}
```

---

# Write metoda

```go
func (c *Cache) Set(
	key string,
	value string,
) {


	c.rw.Lock()

	defer c.rw.Unlock()


	c.data[key] = value

}
```

---

# Zašto je ovo dobar dizajn?

Korisnik cache-a vidi:

```go
cache.Get()

cache.Set()
```

---

Ne zna:

- da postoji Mutex,
- kada se zaključava,
- kako se štiti mapa.

---

To je:

> Encapsulation

---

# Izvršavanje

Imamo:

```text
G1 -> Get(user)

G2 -> Get(product)

G3 -> Get(order)
```

---

Sve mogu:

```go
RLock()
```

istovremeno.

---

Ali:

```text
G4 -> Set()
```

čeka.

---

# Vizuelno

```text
RWMutex


Reader  Reader  Reader

    \      |      /

        Cache


Writer

čeka
```

---

# Primer 2 — Runtime konfiguracija

Vrlo čest slučaj.

Imamo:

```go
type Config struct {

	Timeout int

	Port int

}
```

---

Aplikacija:

stalno čita:

```go
config.Timeout
```

---

Ali administrator može menjati:

```text
reload configuration
```

---

# Implementacija

```go
type ConfigStore struct {

	rw sync.RWMutex

	config Config

}
```

---

# Čitanje konfiguracije

```go
func (c *ConfigStore) Get() Config {

	c.rw.RLock()

	defer c.rw.RUnlock()


	return c.config

}
```

---

# Ažuriranje

```go
func (c *ConfigStore) Update(
	config Config,
) {


	c.rw.Lock()

	defer c.rw.Unlock()


	c.config = config

}
```

---

# Zašto ne koristiti običan Mutex?

Možemo.

Ali:

ako imamo:

```text
1000 request handler-a
```

koji čitaju konfiguraciju:

svaki bi čekao prethodni.

---

Sa `RWMutex`:

više čitanja paralelno.

---

# Primer 3 — Registry Pattern

Registry je struktura koja čuva objekte po ključu.

Primer:

```text
service registry

plugin registry

handler registry
```

---

Primer:

```go
type Registry struct {

	rw sync.RWMutex

	services map[string]interface{}

}
```

---

# Registracija

```go
func (r *Registry) Register(
	name string,
	service interface{},
) {


	r.rw.Lock()

	defer r.rw.Unlock()


	r.services[name] = service

}
```

---

# Dohvatanje

```go
func (r *Registry) Get(
	name string,
) (interface{}, bool) {


	r.rw.RLock()

	defer r.rw.RUnlock()


	service, ok := r.services[name]


	return service, ok

}
```

---

# Tipičan odnos

```text
Startup:

100 writes


Runtime:

100000 reads
```

---

Odličan kandidat za:

```go
RWMutex
```

---

# Primer 4 — In-memory Storage

Zamislimo malu bazu:

```go
type Storage struct {

	rw sync.RWMutex

	users map[int]User

}
```

---

# Read

```go
func (s *Storage) Find(
	id int,
) User {


	s.rw.RLock()

	defer s.rw.RUnlock()


	return s.users[id]

}
```

---

# Write

```go
func (s *Storage) Save(
	user User,
) {


	s.rw.Lock()

	defer s.rw.Unlock()


	s.users[user.ID] = user

}
```

---

# Realan scenario

HTTP server:

```text
Request 1
   |
   |-- Read user


Request 2
   |
   |-- Read user


Request 3
   |
   |-- Update user
```

---

Sa `RWMutex`:

Read operacije mogu zajedno.

Update čeka.

---

# Kada `RWMutex` zaista pomaže?

Najbolji uslovi:

---

## 1. Mnogo više čitanja nego pisanja

Primer:

```text
95% Read

5% Write
```

---

## 2. Podatak se često čita

Primer:

- cache,
- config,
- metadata.

---

## 3. Read operacije nisu trivijalne

Ako je:

```go
x := value
```

korist može biti mala.

---

# Kada nema koristi?

Primer:

```go
counter++
```

---

Ako imamo:

```text
Read 50%

Write 50%
```

---

`RWMutex` često neće biti bolji.

---

# Poređenje realnih primera

| Scenario | Izbor |
|-|-|
| Counter | Mutex |
| Cache | RWMutex |
| Config | RWMutex |
| Queue | Mutex |
| Event pipeline | Channel |
| Registry | RWMutex |
| Shared state | Mutex/RWMutex |

---

# Važna napomena

`RWMutex` nije zamena za:

```go
Mutex
```

---

On rešava specifičan problem:

```text
mnogo čitalaca

malo pisaca
```

---

# Mentalni model

Zapamti:

```text
Mutex

=
jedna osoba unutra
```

---

```text
RWMutex

=
mnogo čitalaca

ili

jedan pisac
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako se `RWMutex` koristi u realnim strukturama,
- kako napraviti thread-safe cache,
- kako zaštititi konfiguraciju,
- kako koristiti Registry pattern,
- kako napraviti in-memory storage,
- kada `RWMutex` daje stvarnu prednost.

---

# `sync.RWMutex` — Performanse, Contention i kada je `Mutex` bolji izbor

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- Zašto `RWMutex` nije automatski brži
- Interni trošak `RWMutex`-a
- Lock contention
- Kada `Mutex` pobeđuje `RWMutex`
- Kada `RWMutex` stvarno pomaže
- Benchmark razmišljanje
- Trade-off analiza
- Production smernice

---

# Uvod

Jedna od najčešćih zabluda:

> "RWMutex je noviji i podržava više čitanja, znači uvek je brži."

Ovo nije tačno.

---

`RWMutex` rešava specifičan problem:

```text
mnogo čitanja

malo pisanja
```

Ako taj obrazac ne postoji,

običan:

```go
sync.Mutex
```

može biti bolji.

---

# Zašto postoji overhead?

Običan Mutex ima jednostavan model:

```text
locked

ili

unlocked
```

---

`RWMutex` mora pratiti:

- broj aktivnih Reader-a,
- da li postoji Writer,
- Reader-e koji čekaju,
- Writer-e koji čekaju.

---

Mentalno:

```text
Mutex

LOCK
 |
data
 |
UNLOCK
```

---

```text
RWMutex

Reader count

Writer state

Waiting readers

Waiting writers
```

---

Zbog toga:

```text
RWMutex > Mutex

po kompleksnosti
```

---

# Mali Critical Section problem

Pretpostavimo:

```go
value++
```

Operacija traje:

```text
nanosekunde
```

---

Dodavanje:

```go
RWMutex
```

može koštati više nego sama operacija.

---

Primer:

```go
rw.RLock()

x := value

rw.RUnlock()
```

---

Ako je:

```text
x := value
```

veoma brzo,

overhead zaključavanja dominira.

---

# Lock Contention

Podsetnik:

Contention znači:

više Goroutines pokušava isti Lock.

---

Primer:

```text
10000 Goroutines

        |

        |

     RWMutex
```

---

Ali pitanje je:

šta rade?

---

# Scenario A — mnogo čitanja

```text
Read

Read

Read

Read

Write
```

---

RWMutex:

```text
Reader 1  \
Reader 2   \
Reader 3    > zajedno
Reader 4   /
```

---

Dobijamo korist.

---

# Scenario B — mnogo pisanja

```text
Write

Write

Write

Write
```

---

RWMutex:

```text
Writer

↓

čekanje

↓

Writer
```

---

Nema prednosti.

---

Čak može biti sporiji od:

```go
sync.Mutex
```

---

# Kada Mutex pobeđuje?

---

## 1. Jednostavan shared counter

Primer:

```go
type Counter struct {

	mu sync.Mutex

	value int

}
```

---

Operacije:

```go
Increment()

Decrement()
```

---

Ovde nema smisla:

```go
RWMutex
```

---

Zašto?

Zato što:

- često pišemo,
- Critical Section je mala.

---

## 2. Queue

Primer:

```go
Push()

Pop()
```

---

Obično:

```text
jedan podatak

često menjanje
```

---

Bolji:

```go
Mutex
```

---

## 3. Visoka frekvencija update-a

Ako imamo:

```text
100000 writes/sec
```

RWMutex nema prednost.

---

# Kada RWMutex pobeđuje?

---

# 1. Cache

Primer:

```text
99% Get

1% Set
```

---

Odličan kandidat.

---

# 2. Konfiguracija

Primer:

```text
čitamo na svakom request-u

menjamo jednom dnevno
```

---

---

# 3. Registry

Primer:

```text
startup registracija

runtime čitanje
```

---

---

# Benchmark razmišljanje

Nikada ne odlučivati:

```text
po osećaju
```

---

Pravilno:

1. Implementiraj jednostavno.
2. Izmeri.
3. Optimizuj.

---

# Primer benchmark pitanja

Ne pitamo:

> "Da li je RWMutex brži?"

---

Pitamo:

> "Da li je RWMutex brži u mom konkretnom workload-u?"

---

# Benchmark scenariji

Treba testirati:

---

## Scenario 1

```text
100% Read
```

---

## Scenario 2

```text
90% Read

10% Write
```

---

## Scenario 3

```text
50% Read

50% Write
```

---

## Scenario 4

```text
100% Write
```

---

Rezultati mogu biti potpuno različiti.

---

# Primer benchmark koda

```go
func BenchmarkMutex(b *testing.B) {

	var mu sync.Mutex

	value := 0


	b.RunParallel(func(pb *testing.PB) {

		for pb.Next() {

			mu.Lock()

			value++

			mu.Unlock()

		}

	})

}
```

---

RWMutex verzija:

```go
func BenchmarkRWMutex(b *testing.B) {

	var rw sync.RWMutex

	value := 0


	b.RunParallel(func(pb *testing.PB) {

		for pb.Next() {

			rw.Lock()

			value++

			rw.Unlock()

		}

	})

}
```

---

Kod ovog primera:

```text
RWMutex nema prednost
```

jer je sve Write.

---

# Važna lekcija

Nemoj birati:

```text
RWMutex

jer zvuči naprednije
```

---

Biraj:

```text
RWMutex

jer obrazac pristupa odgovara
```

---

# Performance pravilo

## Mali objekti

Koristi:

```go
Mutex
```

---

## Read-heavy objekti

Razmotri:

```go
RWMutex
```

---

## Veliki sistemi

Razmotri:

- sharding,
- atomics,
- lock-free strukture,
- channels,
- cache dizajn.

---

# Lock Sharding

Napredna tehnika.

Umesto:

```text
jedan veliki Mutex
```

koristi:

```text
više manjih Mutex-a
```

---

Primer:

```text
Cache

Shard 1 + Mutex

Shard 2 + Mutex

Shard 3 + Mutex
```

---

Rezultat:

manje contention-a.

---

# Ali oprez

Više Lock-ova znači:

više kompleksnosti.

---

# Production smernice

---

## 1. Počni sa Mutex-om

Najčešće:

```go
sync.Mutex
```

je dovoljno dobar.

---

## 2. Meri pre optimizacije

Koristi:

```bash
go test -bench .
```

---

## 3. RWMutex koristi za read-heavy strukture

Ne kao default.

---

## 4. Kratke Critical Section

Uvek.

---

## 5. Ne zaključavaj I/O

Izbegavati:

```go
Lock()

database.Call()

Unlock()
```

---

# Mentalni model

Zapamti:

```text
Mutex

=
jednostavan alat


RWMutex

=
specijalizovan alat
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto `RWMutex` nije uvek brži,
- kakav overhead uvodi,
- kako contention utiče na performanse,
- kada `Mutex` pobeđuje,
- kada `RWMutex` daje prednost,
- kako razmišljati o benchmark testovima,
- šta je lock sharding.

---

# `sync.RWMutex` — Napredni problemi, ograničenja i best practices

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- Kopiranje `RWMutex`-a
- Reentrant Lock problem
- RLock → Lock problem
- Lock upgrade problem
- Lock downgrade problem
- Nested Lock problemi
- Deadlock scenariji
- Pravilan dizajn API-ja
- Production best practices

---

# Uvod

`sync.RWMutex` izgleda jednostavno:

```go
RLock()

RUnlock()

Lock()

Unlock()
```

---

Ali u velikim sistemima problemi nastaju zbog:

- pogrešne pretpostavke o vlasništvu,
- pogrešnog redosleda Lock-ova,
- pokušaja kombinovanja Read i Write zaključavanja.

---

# 1. Ne kopirati `RWMutex`

Kao i kod:

```go
sync.Mutex
```

pravilo je:

> `RWMutex` se ne sme kopirati nakon prve upotrebe.

---

# Pogrešno

```go
type Store struct {

	rw sync.RWMutex

	data map[string]string

}
```

---

Zatim:

```go
copy := original
```

---

Problem:

Dobijamo dve strukture:

```text
Store A

RWMutex A


Store B

RWMutex B
```

---

Ali:

podaci mogu pokazivati na isto stanje.

---

Rezultat:

```text
dva Lock-a

jedan podatak
```

---

To može izazvati:

- race condition,
- nekonzistentno stanje.

---

# Pravilo

Ako struktura sadrži:

```go
sync.RWMutex
```

koristi:

```go
pointer
```

---

Dobro:

```go
func NewStore() *Store
```

---

Loše:

```go
func GetStore() Store
```

---

# 2. Reentrant Lock problem

Go `RWMutex` nije reentrant.

To znači:

ista Goroutine ne može bezbedno ponovo zaključati isti Lock.

---

Primer:

```go
func Read() {

	rw.RLock()

	defer rw.RUnlock()


	internal()

}
```

---

A:

```go
func internal(){

	rw.RLock()

}
```

---

Dobijamo:

```text
RLock()

    |
    |

poziva drugi RLock()

    |
    |

problem
```

---

# Zašto?

Zato što:

`RWMutex` ne prati vlasnika.

---

On samo prati:

```text
koliko Reader-a postoji
```

---

# 3. RLock → Lock problem

Veoma česta greška.

Programer želi:

1. Pročitati podatak.
2. Ako ne postoji, napraviti ga.

---

Primer:

```go
rw.RLock()


if data == nil {

    rw.Lock()

    create()

}
```

---

Ovo je pogrešno.

---

Zašto?

Jer već držimo:

```go
RLock()
```

a pokušavamo:

```go
Lock()
```

---

Ali Writer mora čekati sve Reader-e.

Uključujući:

```text
samog sebe
```

---

Rezultat:

Deadlock.

---

# Pogrešan model

Programer misli:

```text
imam Read Lock

sada samo pojačam na Write Lock
```

---

Ali `RWMutex` nema:

```text
upgrade operaciju
```

---

# Ispravan obrazac

Otključati Read:

```go
rw.RUnlock()
```

zatim:

```go
rw.Lock()
```

---

Primer:

```go
rw.RLock()

exists := check()

rw.RUnlock()


if !exists {

	rw.Lock()

	create()

	rw.Unlock()

}
```

---

Ali postoji mogućnost:

druga Goroutine promeni stanje između.

---

Zato često koristimo:

```text
double check pattern
```

---

Primer:

```go
rw.Lock()

if data == nil {

	create()

}

rw.Unlock()
```

---

# 4. Lock downgrade problem

Suprotno:

```text
Lock

↓

RLock
```

---

Programer želi:

1. Menjati podatak.
2. Nastaviti samo čitanje.

---

Primer:

```go
rw.Lock()

update()

rw.RLock()

rw.Unlock()
```

---

Ovo nije podržano.

---

Nema:

```text
downgrade operacije
```

---

Pravilno:

```go
rw.Lock()

update()

rw.Unlock()


rw.RLock()

read()

rw.RUnlock()
```

---

# 5. Nested Lock problemi

Nested Lock znači:

Lock unutar Lock-a.

---

Primer:

```go
func A(){

	rw.Lock()

	B()

	rw.Unlock()

}
```

---

A:

```go
func B(){

	rw.Lock()

}
```

---

Rezultat:

```text
A drži Lock

A čeka B

B nikada ne može dobiti Lock
```

---

Deadlock.

---

# Pravilo

Dizajniraj funkcije tako da:

ili:

```text
same zaključavaju
```

ili:

```text
pozivalac zaključava
```

---

Ne oba.

---

# 6. RWMutex i Deadlock

Iako `RWMutex` rešava Reader problem,

i dalje može imati:

- Deadlock,
- Lock ordering problem,
- zaboravljen Unlock.

---

Primer:

```go
rw.Lock()

defer rw.Unlock()


callOtherFunction()
```

---

Ako:

```go
callOtherFunction()
```

pokuša:

```go
rw.RLock()
```

problem.

---

# 7. Dizajn API-ja

Bolji dizajn:

```go
type Cache struct {

	rw sync.RWMutex

	data map[string]string

}
```

---

Javni API:

```go
Get()

Set()
```

---

Korisnik ne zna:

- kada se zaključava,
- koji Lock postoji.

---

---

Loš dizajn:

```go
cache.Lock()

cache.Get()

cache.Unlock()
```

---

Zašto?

Jer korisnik sada upravlja internim stanjem.

---

# 8. Best Practices

---

## Pravilo 1

Ne kopiraj:

```go
Mutex

RWMutex
```

---

## Pravilo 2

Ne pokušavaj:

```text
RLock → Lock
```

---

## Pravilo 3

Ne pokušavaj:

```text
Lock → RLock
```

---

## Pravilo 4

Drži Lock kratko.

---

## Pravilo 5

Ne pozivaj nepoznat kod dok držiš Lock.

---

Loše:

```go
rw.Lock()

callback()

rw.Unlock()
```

---

Zašto?

Callback može:

- čekati,
- pokušati isti Lock,
- pozvati drugi Lock.

---

## Pravilo 6

Koristi:

```go
defer
```

---

Primer:

```go
rw.Lock()

defer rw.Unlock()
```

---

# Mentalni model

Zapamti:

```text
RWMutex nije pametan Lock.

On samo daje dva režima:

READ

WRITE
```

---

Nema:

- upgrade,
- downgrade,
- vlasništvo,
- reentrant ponašanje.

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto se `RWMutex` ne kopira,
- zašto nema reentrant ponašanje,
- zašto `RLock → Lock` može napraviti deadlock,
- zašto nema upgrade/downgrade mehanizam,
- kako nastaju nested lock problemi,
- kako dizajnirati bezbedan API,
- production best practices.

---

# `sync.RWMutex` — Završni rezime, izbor alata i praktični zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 2/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/02-rwmutex.md`

---

# 📚 Sadržaj ovog dela

- Kompletan rezime `sync.RWMutex`
- Finalna analiza `Mutex` vs `RWMutex`
- Kako izabrati pravi alat
- Kada koristiti `RWMutex`
- Kada izbegavati `RWMutex`
- Production checklist
- Pitanja za proveru znanja
- Praktični zadaci

---

# Kompletan rezime `sync.RWMutex`

`sync.RWMutex` je specijalizovani alat za zaštitu deljenog stanja kada imamo:

```text
mnogo čitanja

+

malo pisanja
```

---

Njegova glavna ideja:

```text
Reader-i mogu zajedno.

Writer mora sam.
```

---

# Osnovni API

## Čitanje

```go
RLock()

RUnlock()
```

---

## Pisanje

```go
Lock()

Unlock()
```

---

# Mentalni model

`Mutex`:

```text
jedna Goroutine unutra
```

---

`RWMutex`:

```text
više Reader-a

ili

jedan Writer
```

---

# Mutex vs RWMutex

Najvažnija odluka u Go concurrency dizajnu.

---

# `sync.Mutex`

Model:

```text
Exclusive Lock
```

---

Svaki pristup:

- čitanje,
- pisanje,

zahteva:

```go
Lock()
```

---

Primer:

```go
mu.Lock()

value++

mu.Unlock()
```

---

# `sync.RWMutex`

Model:

```text
Read / Write Lock
```

---

Čitanje:

```go
rw.RLock()
```

---

Pisanje:

```go
rw.Lock()
```

---

# Direktno poređenje

| Karakteristika | Mutex | RWMutex |
|-|-|-|
| Read | blokira ostale | dozvoljava paralelno čitanje |
| Write | ekskluzivan | ekskluzivan |
| API | jednostavan | kompleksniji |
| Overhead | manji | veći |
| Najbolji slučaj | opšti pristup | read-heavy sistemi |
| Rizik greške | manji | veći |

---

# Kako izabrati?

Postavi pitanje:

---

# Pitanje 1

Da li imam shared state?

Primer:

```go
map

slice

struct state
```

Ako ne:

ne treba Lock.

---

# Pitanje 2

Da li više Goroutines menjaju taj podatak?

Ako da:

treba zaštita.

---

# Pitanje 3

Kakav je odnos Read/Write?

---

## Scenario A

```text
90-99% Read

1-10% Write
```

Razmotriti:

```go
RWMutex
```

---

## Scenario B

```text
50% Read

50% Write
```

Obično:

```go
Mutex
```

---

## Scenario C

```text
100% Write
```

Koristi:

```go
Mutex
```

---

# Kada koristiti `RWMutex`?

---

# 1. Cache

Primer:

```text
Get()

Get()

Get()

Set()
```

---

Odličan kandidat.

---

# 2. Konfiguracija

Primer:

```text
čitamo svaki request

menjamo povremeno
```

---

---

# 3. Registry

Primer:

```text
startup:

registracija


runtime:

čitanje
```

---

---

# 4. Metadata

Primer:

```text
routing table

permissions

schema information
```

---

# Kada NE koristiti `RWMutex`?

---

# 1. Jednostavni counter

Loše:

```go
rw.Lock()

counter++

rw.Unlock()
```

---

Ovde:

```go
Mutex
```

je jednostavniji.

---

# 2. Queue

Primer:

```text
Push

Pop
```

---

Obično:

```go
Mutex
```

---

# 3. Česte izmene

Ako imamo:

```text
mnogo Writer-a
```

RWMutex nema prednost.

---

# 4. Channel problem

Ako je problem:

```text
slanje podataka između Goroutines
```

onda:

```go
channel
```

je prirodniji.

---

# Mutex ili Channel?

Vrlo važno pitanje.

---

## Mutex

Koristi kada kažeš:

> "Više Goroutines pristupa istom podatku."

Primer:

```go
cache[key]
```

---

## Channel

Koristi kada kažeš:

> "Jedna Goroutine šalje posao drugoj."

Primer:

```go
jobs <- task
```

---

# Kombinacija u realnim sistemima

Često se koriste zajedno.

Primer:

```text
HTTP Server


      |
      |

Worker Pool

      |
      |

Cache

      |
      |

RWMutex
```

---

Channel:

- transport podataka.

RWMutex:

- zaštita internog stanja.

---

# Production Checklist

Pre korišćenja `RWMutex` proveriti:

---

## ✅ 1. Da li stvarno imam read-heavy scenario?

Ako ne:

razmotriti `Mutex`.

---

## ✅ 2. Da li je Critical Section mala?

Treba biti.

---

## ✅ 3. Da li koristim:

```go
RLock()

RUnlock()
```

za čitanje?

---

## ✅ 4. Da li koristim:

```go
Lock()

Unlock()
```

za promenu?

---

## ✅ 5. Da li izbegavam:

```text
RLock → Lock
```

---

## ✅ 6. Da li izbegavam kopiranje?

```go
RWMutex
```

---

## ✅ 7. Da li testiram sa:

```bash
go test -race
```

---

# Pitanja za proveru znanja

---

## 1.

Zašto `RWMutex` dozvoljava više Reader-a?

---

## 2.

Zašto Writer mora biti sam?

---

## 3.

Zašto `RWMutex` nije uvek brži od `Mutex`-a?

---

## 4.

Da li može:

```text
RLock → Lock
```

?

Zašto?

---

## 5.

Koji alat bi izabrao za cache?

Zašto?

---

## 6.

Koji alat bi izabrao za queue?

Zašto?

---

## 7.

Koja je razlika:

```text
Mutex

vs

Channel
```

?

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnove

## Zadatak 1

Napraviti:

```go
SafeConfig
```

koji podržava:

```go
Get()

Set()
```

koristeći:

```go
RWMutex
```

---

## Zadatak 2

Napraviti:

```go
SafeCounter
```

uporediti:

- Mutex implementaciju
- RWMutex implementaciju

---

# 🟡 Nivo 2 — Praktično

## Zadatak 3

Napraviti:

```go
ThreadSafeCache
```

podržati:

```go
Get()

Set()

Delete()
```

---

## Zadatak 4

Napraviti:

```go
Registry
```

za:

```text
services

handlers

plugins
```

---

# 🟠 Nivo 3 — Napredno

## Zadatak 5

Implementirati:

```go
InMemoryDatabase
```

sa:

- CRUD operacijama,
- konkurentnim čitanjem,
- sigurnim pisanjem.

---

## Zadatak 6

Napraviti benchmark:

Uporediti:

```text
Mutex

vs

RWMutex
```

za:

- 100% Read,
- 90/10,
- 50/50,
- 100% Write.

---

# 🔴 Nivo 4 — Senior

## Zadatak 7

Napraviti cache sistem:

Zahtevi:

- RWMutex,
- expiration,
- cleanup goroutine,
- race detector test.

---

## Zadatak 8

Analizirati:

kada je bolje:

```text
RWMutex

vs

atomic

vs

channel
```

---

# Završna poruka

`sync.RWMutex` nije alat koji treba automatski koristiti.

Njegova vrednost dolazi kada postoji specifičan obrazac:

```text
mnogo čitanja

malo pisanja
```

---

Pravi izbor:

```text
Jednostavno stanje
        |
        |
    Mutex


Tok podataka
        |
        |
    Channel


Read-heavy shared state
        |
        |
    RWMutex
```

---

### ➡️ Sledeća lekcija **[**sync.Once**](03-once.md)**

Obradićemo:

- lazy initialization,
- singleton pattern,
- thread-safe inicijalizaciju,
- razliku `sync.Once` vs `Mutex`,
- internu logiku `Once`,
- production primere.
