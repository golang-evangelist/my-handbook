# `sync.Once` — Jednokratna thread-safe inicijalizacija u Go Concurrency modelu

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Zašto postoji `sync.Once`
- Problem višestruke inicijalizacije
- Lazy Initialization koncept
- Singleton problem
- Osnovni API
- Kako se koristi `sync.Once`
- Prvi praktični primer
- Mentalni model

---

# Uvod

U prethodnim lekcijama učili smo:

```go
sync.Mutex
```

i:

```go
sync.RWMutex
```

Njihova svrha:

```text
kontrola pristupa deljenom stanju
```

---

Ali postoji drugačiji problem.

---

Ponekad ne želimo:

```text
više Goroutines da pristupaju istom podatku
```

nego želimo:

> određena operacija sme da se izvrši samo jednom.

---

Primeri:

- inicijalizacija konfiguracije,
- kreiranje singleton objekta,
- učitavanje environment varijabli,
- povezivanje sa bazom,
- kreiranje cache-a,
- učitavanje velikog resursa.

---

# Problem višestruke inicijalizacije

Pretpostavimo:

```go
var database *Database
```

---

Imamo funkciju:

```go
func GetDatabase() *Database {

	if database == nil {

		database = createDatabase()

	}

	return database

}
```

---

Na prvi pogled izgleda ispravno.

---

Ali šta ako imamo:

```text
Goroutine 1

GetDatabase()


Goroutine 2

GetDatabase()
```

---

# Race scenario

Početno stanje:

```go
database == nil
```

---

## Goroutine 1

Proverava:

```go
if database == nil
```

Rezultat:

```text
true
```

---

## Goroutine 2

Isto:

```go
if database == nil
```

Rezultat:

```text
true
```

---

Sada obe rade:

```go
createDatabase()
```

---

Rezultat:

```text
Database A

Database B
```

---

Problem:

Imamo dve inicijalizacije.

---

# Rešenje pomoću Mutex-a

Jedno rešenje:

```go
var mu sync.Mutex

var database *Database
```

---

Kod:

```go
func GetDatabase() *Database {

	mu.Lock()

	defer mu.Unlock()


	if database == nil {

		database = createDatabase()

	}


	return database

}
```

---

Ovo radi.

---

Ali postoji problem.

---

Svaki poziv:

```go
GetDatabase()
```

mora:

```go
Lock()

Unlock()
```

---

Čak i kada je:

```go
database
```

već kreiran.

---

Primer:

```text
100000 poziva

100000 Lock operacija
```

---

To je nepotreban overhead.

---

# `sync.Once`

Go daje specijalizovan alat:

```go
sync.Once
```

---

Njegova namena:

> izvrši funkciju tačno jednom, bez obzira koliko Goroutines je pozove.

---

# Osnovni API

Struktura:

```go
var once sync.Once
```

---

Metoda:

```go
once.Do(function)
```

---

Primer:

```go
once.Do(func(){

	initialize()

})
```

---

Prvi poziv:

izvršava funkciju.

---

Svi sledeći:

preskaču.

---

# Prvi primer

```go
package main

import (
	"fmt"
	"sync"
)


var once sync.Once


func initialize(){

	fmt.Println("Initialization")

}


func main(){

	once.Do(initialize)

	once.Do(initialize)

	once.Do(initialize)

}
```

---

Rezultat:

```text
Initialization
```

samo jednom.

---

Iako smo pozvali:

```go
3 puta
```

---

# Goroutine primer

```go
var once sync.Once


func worker(){

	once.Do(func(){

		fmt.Println("Start")

	})

}
```

---

Pokrenemo:

```go
for i := 0; i < 100; i++ {

	go worker()

}
```

---

Rezultat:

```text
Start
```

jednom.

---

Ne:

```text
Start
Start
Start
...
```

---

# Mentalni model

`sync.Once` možemo zamisliti kao prekidač.

Početno:

```text
OFF
```

---

Prvi poziv:

```go
once.Do()
```

---

Stanje:

```text
ON
```

---

Svi sledeći:

```text
ignorisan
```

---

Vizuelno:

```text
G1
 |
 |
Do()
 |
 |
[ ONCE ]
 |
 |
execute


G2

Do()

     X


G3

Do()

     X
```

---

# `sync.Once` nije Mutex

Važna razlika.

---

Mutex:

```text
više puta zaključaj

više puta otključaj
```

---

Primer:

```go
Lock()

Unlock()

Lock()

Unlock()
```

---

`sync.Once`:

```text
jednom izvrši

nikad više
```

---

# Poređenje

| | Mutex | Once |
|-|-|-|
| Broj izvršavanja | neograničeno | jednom |
| Glavna svrha | zaštita podataka | inicijalizacija |
| Lock/Unlock | da | ne |
| Ponovna upotreba | da | ne za istu akciju |

---

# Kada koristiti `sync.Once`?

---

## 1. Lazy Initialization

Primer:

```go
var config Config
```

učitava se tek kada zatreba.

---

## 2. Singleton objekti

Primer:

```go
Database connection
```

---

## 3. Globalni resursi

Primer:

```go
Logger

Metrics

Cache
```

---

## 4. Skupa inicijalizacija

Primer:

```go
loadLargeModel()

parseSchema()

compileTemplate()
```

---

# Kada NE koristiti `sync.Once`?

---

## 1. Kada operacija treba više puta

Primer:

```go
refreshToken()
```

nije za:

```go
sync.Once
```

---

## 2. Kada stanje može da se resetuje

Primer:

```text
reload configuration
```

---

`Once` nema:

```text
Reset()
```

---

## 3. Za običnu zaštitu podataka

Ne koristiti:

```go
once.Do(func(){

	counter++

})
```

---

Za to postoji:

```go
Mutex
```

ili:

```go
atomic
```

---

# Važna osobina

Ako funkcija u:

```go
once.Do(func(){

})
```

završi uspešno:

nikada se više neće izvršiti.

---

Ako funkcija:

```go
panic()
```

takođe se smatra:

```text
pozvana
```

---

Ovo ćemo detaljno obraditi kasnije.

---

# 📋 Rezime

U ovom delu naučili smo:

- koji problem rešava `sync.Once`,
- zašto obična inicijalizacija nije thread-safe,
- kako Mutex rešava problem ali uvodi overhead,
- osnovni API:

```go
once.Do()
```

- koncept jednokratne inicijalizacije,
- razliku između `Mutex` i `Once`,
- kada koristiti `sync.Once`.

---

# `sync.Once` — Lazy Initialization i praktični obrasci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Šta je Lazy Initialization
- Zašto koristiti Lazy Initialization
- Problem sa običnom inicijalizacijom
- Lazy Initialization pomoću `sync.Once`
- Database connection primer
- Configuration loader primer
- Cache initialization primer
- Prednosti i mane ovog pristupa

---

# Uvod

U prethodnom delu smo videli osnovnu ideju:

```go
sync.Once
```

garantuje:

```text
jedna operacija

jednom izvršena

thread-safe
```

---

Najčešća primena:

```text
Lazy Initialization
```

---

# Šta je Lazy Initialization?

Lazy Initialization znači:

> Kreiraj objekat tek kada je prvi put potreban.

---

Suprotno:

```text
Eager Initialization
```

---

# Eager Initialization

Objekat se kreira odmah.

Primer:

```go
var database = createDatabase()
```

---

Prilikom startovanja programa:

```text
main()

↓

kreiranje database

↓

ostatak aplikacije
```

---

# Problem

Možda database nikada neće biti korišćen.

---

Primer:

Aplikacija ima:

```text
Feature A

Feature B

Feature C
```

---

Database za:

```text
Feature C
```

kreira se odmah.

---

Ali korisnik nikada ne koristi:

```text
Feature C
```

---

Rezultat:

- nepotrebno zauzeta memorija,
- duže vreme startovanja,
- nepotrebni resursi.

---

# Lazy Initialization

Umesto:

```go
createDatabase()
```

odmah,

radimo:

```go
createDatabase()
```

kada prvi put zatreba.

---

Primer:

```go
func GetDatabase() *Database {

	if database == nil {

		database = createDatabase()

	}

	return database

}
```

---

Ali ovaj kod nije bezbedan za Concurrency.

---

# Problem sa više Goroutines

Imamo:

```go
go GetDatabase()

go GetDatabase()
```

---

Obe Goroutines vide:

```go
database == nil
```

---

Obe kreiraju:

```go
createDatabase()
```

---

Dobijamo:

```text
Database 1

Database 2
```

---

Potrebna nam je kontrola:

```text
samo prvi poziv inicijalizuje
```

---

# Lazy Initialization sa `sync.Once`

Primer:

```go
var once sync.Once

var database *Database
```

---

Funkcija:

```go
func GetDatabase() *Database {


	once.Do(func(){

		database = createDatabase()

	})


	return database

}
```

---

# Kako radi?

Prvi poziv:

```go
GetDatabase()
```

---

Dešava se:

```text
once.Do()

↓

createDatabase()

↓

database dobija vrednost
```

---

Drugi poziv:

```go
GetDatabase()
```

---

Dešava se:

```text
once.Do()

↓

skip
```

---

# Goroutine scenario

Imamo:

```go
100 Goroutines
```

koje pozivaju:

```go
GetDatabase()
```

---

Interno:

```text
G1
 |
 Do()
 |
 initialize


G2
 |
 Do()
 |
 wait / skip


G3
 |
 Do()
 |
 wait / skip
```

---

Rezultat:

```text
createDatabase()
```

izvršava se jednom.

---

# Primer: Database Manager

```go
type Database struct {

	Connection string

}
```

---

Globalni resurs:

```go
var (

	db *Database

	dbOnce sync.Once

)
```

---

Getter:

```go
func GetDatabase() *Database {


	dbOnce.Do(func(){

		db = &Database{

			Connection: "localhost",

		}

	})


	return db

}
```

---

Korišćenje:

```go
func handler(){

	database := GetDatabase()

	use(database)

}
```

---

# Zašto je ovo korisno?

Svaki handler može:

```go
GetDatabase()
```

bez razmišljanja o:

- Lock-u,
- Race condition-u,
- duploj inicijalizaciji.

---

# Primer: Configuration Loader

Imamo:

```go
type Config struct {

	AppName string

	Port int

}
```

---

Želimo:

učitati config samo jednom.

---

Kod:

```go
var (

	config Config

	configOnce sync.Once

)
```

---

Loader:

```go
func LoadConfig() Config {


	configOnce.Do(func(){

		config = Config{

			AppName: "API",

			Port: 8080,

		}

	})


	return config

}
```

---

---

# Primer: Cache Initialization

Pretpostavimo:

```go
type Cache struct {

	items map[string]string

}
```

---

Kreiranje:

```go
func createCache() *Cache {

	return &Cache{

		items: make(map[string]string),

	}

}
```

---

Lazy:

```go
var (

	cache *Cache

	cacheOnce sync.Once

)
```

---

Getter:

```go
func GetCache() *Cache {


	cacheOnce.Do(func(){

		cache = createCache()

	})


	return cache

}
```

---

# Prednosti Lazy Initialization

---

## 1. Brži startup

Ne kreiraš sve odmah.

---

## 2. Manja potrošnja resursa

Kreira se samo ono što se koristi.

---

## 3. Jednostavna thread safety kontrola

Nema:

```go
if nil
```

provere.

---

## 4. Čist API

Korisnik vidi:

```go
GetResource()
```

a ne:

```go
initialize()
```

---

# Mane Lazy Initialization

---

# 1. Prvi poziv može biti spor

Primer:

```go
request 1

↓

kreiranje database

↓

odgovor
```

---

Prvi korisnik plaća cenu inicijalizacije.

---

# Rešenje

Preload:

```go
GetDatabase()
```

tokom:

```text
startup phase
```

---

# 2. Greške se pojavljuju kasnije

Kod:

```go
createConnection()
```

može pasti tek kada se prvi put pozove.

---

Kod eager pristupa:

greška se vidi odmah.

---

# 3. Nema resetovanja

Jednom:

```go
once.Do()
```

odrađeno.

---

Nema:

```go
once.Reset()
```

---

# Kada koristiti Lazy Initialization?

Dobri kandidati:

---

## Database

```text
skupa konekcija
```

---

## Cache

```text
velika memorija
```

---

## Logger

```text
globalni resurs
```

---

## Template engine

```text
skupa priprema
```

---

## ML modeli

```text
veliki resursi
```

---

# Kada ne koristiti?

---

## Kritični startup resursi

Ako aplikacija ne može raditi bez njega:

bolje odmah inicijalizovati.

---

## Resurs koji se često menja

Primer:

```text
reload config
```

nije dobar kandidat.

---

## Kratkotrajni objekti

Primer:

```go
request object
```

nema smisla.

---

# Mentalni model

Zapamti:

```text
sync.Once

=

"napravi prvi put kada zatreba

i nikada više"
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Lazy Initialization,
- razliku između eager i lazy pristupa,
- kako `sync.Once` rešava thread-safe lazy loading,
- database connection primer,
- config loader primer,
- cache initialization primer,
- prednosti i ograničenja ovog pristupa.

---

# `sync.Once` — Singleton pattern i idiomatski Go dizajn

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Šta je Singleton pattern
- Problem Singleton-a u konkurentnim sistemima
- Klasična Singleton implementacija
- Zašto je Singleton kontroverzan
- Thread-safe Singleton pomoću `sync.Once`
- Singleton u Go ekosistemu
- Kada koristiti Singleton
- Alternative Singleton pattern-u

---

# Uvod

Jedan od najpoznatijih design pattern-a u programiranju je:

```text
Singleton Pattern
```

---

Osnovna ideja:

> U celoj aplikaciji postoji samo jedna instanca određenog objekta.

---

Primeri:

- Database client
- Logger
- Configuration manager
- Metrics collector
- Connection pool

---

# Šta problem Singleton rešava?

Zamislimo:

```go
type Logger struct {

}

```

---

Ako svaka komponenta pravi svoj logger:

```go
logger1 := NewLogger()

logger2 := NewLogger()

logger3 := NewLogger()
```

---

Dobijamo:

```text
3 Logger instance
```

---

Problemi:

- duplirana konfiguracija,
- nepotrebna memorija,
- različito ponašanje.

---

Želimo:

```text
jedna instanca
```

---

# Klasična Singleton ideja

Model:

```text
globalna promenljiva

+

getter funkcija
```

---

Primer:

```go
var instance *Logger


func GetLogger() *Logger {

	if instance == nil {

		instance = &Logger{}

	}

	return instance

}
```

---

Naivno izgleda dobro.

---

Ali u Go concurrency svetu:

nije bezbedno.

---

# Problem sa Goroutines

Imamo:

```go
go GetLogger()

go GetLogger()
```

---

Početno:

```go
instance == nil
```

---

Goroutine 1:

```go
if instance == nil
```

rezultat:

```text
true
```

---

Goroutine 2:

```go
if instance == nil
```

rezultat:

```text
true
```

---

Obe rade:

```go
instance = &Logger{}
```

---

Rezultat:

```text
Logger A

Logger B
```

---

Singleton je uništen.

---

# Rešenje pomoću Mutex-a

Jedna mogućnost:

```go
var mu sync.Mutex

var instance *Logger
```

---

Implementacija:

```go
func GetLogger() *Logger {


	mu.Lock()

	defer mu.Unlock()


	if instance == nil {

		instance = &Logger{}

	}


	return instance

}
```

---

Radi.

---

Ali svaki poziv:

```go
GetLogger()
```

plaća:

```text
Lock overhead
```

---

Čak i kada objekat već postoji.

---

# Double Checked Locking

Pokušaj optimizacije:

```go
func GetLogger() *Logger {


	if instance == nil {


		mu.Lock()


		if instance == nil {

			instance = &Logger{}

		}


		mu.Unlock()

	}


	return instance

}
```

---

Ideja:

prvo proveri bez Lock-a.

---

Problem:

U mnogim jezicima ovo zahteva pažljivu memory sinhronizaciju.

---

U Go-u je bolje koristiti:

```go
sync.Once
```

---

# Thread-safe Singleton pomoću `sync.Once`

Idiomsko Go rešenje:

```go
type Logger struct {

	Name string

}
```

---

Globalno:

```go
var (

	logger *Logger

	loggerOnce sync.Once

)
```

---

Getter:

```go
func GetLogger() *Logger {


	loggerOnce.Do(func(){

		logger = &Logger{

			Name: "Application Logger",

		}

	})


	return logger

}
```

---

---

# Korišćenje

Prva komponenta:

```go
logger := GetLogger()
```

---

Izvršava:

```go
loggerOnce.Do()
```

---

Kreira:

```text
Logger instance
```

---

Druga komponenta:

```go
logger := GetLogger()
```

---

Dobija:

istu instancu.

---

# Goroutine test

```go
func worker(){

	logger := GetLogger()

	fmt.Println(logger)

}
```

---

Pokrenemo:

```go
for i := 0; i < 100; i++ {

	go worker()

}
```

---

Dobijamo:

```text
100 referenci

na isti objekat
```

---

# Zašto je `sync.Once` idealan za Singleton?

Zato što daje:

---

## 1. Thread safety

Više Goroutines:

```go
Do()
```

bezbedno.

---

## 2. Minimalan overhead

Nema stalnog:

```go
Lock()

Unlock()
```

---

## 3. Jednostavan kod

Nema:

- kompleksne provere,
- ručne sinhronizacije.

---

# Singleton u Go-u

Važna stvar:

Go nema klasične OOP Singleton obrasce kao:

Java:

```java
private constructor
```

---

ili:

C++:

```cpp
static instance()
```

---

Go preferira:

- package level funkcije,
- dependency injection,
- eksplicitno prosleđivanje zavisnosti.

---

# Primer koji Go često preferira

Umesto:

```go
GetDatabase()
```

svuda,

možemo:

```go
type Server struct {

	db *Database

}
```

---

Kreiranje:

```go
server := Server{

	db: database,

}
```

---

Prednosti:

- lakše testiranje,
- manje globalnog stanja,
- jasnije zavisnosti.

---

# Singleton i testiranje

Problem:

Globalni singleton:

```go
var logger *Logger
```

---

može otežati testove.

---

Primer:

Test 1:

```text
inicijalizuje logger
```

---

Test 2:

dobija već postojeće stanje.

---

Problemi:

- redosled testova,
- skriveno stanje,
- teže mockovanje.

---

# Kada Singleton ima smisla?

---

## 1. Procesni resurs

Primer:

```text
metrics collector
```

---

## 2. Skupa inicijalizacija

Primer:

```text
connection pool
```

---

## 3. Globalni read-only podaci

Primer:

```text
compiled templates
```

---

## 4. Jedan klijent prema eksternom sistemu

Primer:

```text
API client
```

---

# Kada izbegavati Singleton?

---

## 1. Business logic objekti

Primer:

```text
UserService
OrderService
```

---

Bolje:

```go
inject dependency
```

---

## 2. Mutable global state

Primer:

```go
globalCounter++
```

---

## 3. Kada otežava testiranje

Ako moraš:

```go
resetSingleton()
```

verovatno postoji problem u dizajnu.

---

# Singleton alternative

---

# 1. Dependency Injection

Najčešći Go pristup.

---

Primer:

```go
func NewService(
	db *Database,
	cache *Cache,
) *Service
```

---

---

# 2. Package-level immutable data

Primer:

```go
var Version = "1.0"
```

---

---

# 3. Explicit lifecycle management

Primer:

```go
server.Start()

server.Stop()
```

---

# Mentalni model

Zapamti:

```text
sync.Once

nije Singleton alat

```

---

On rešava:

```text
jednokratnu inicijalizaciju
```

---

Singleton je samo:

```text
jedan od načina korišćenja
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Singleton pattern,
- zašto klasična implementacija nije thread-safe,
- kako nastaje race prilikom kreiranja instance,
- zašto je `sync.Once` idiomatsko Go rešenje,
- kako napraviti thread-safe Singleton,
- zašto Go često preferira dependency injection,
- kada Singleton koristiti,
- kada ga izbegavati.

---

# `sync.Once` — Interna implementacija, Memory Model i happens-before garancije

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Kako `sync.Once` radi interno
- Zašto običan boolean nije dovoljan
- Interna struktura `sync.Once`
- Uloga `Mutex`-a unutar `Once`
- Memory synchronization
- Happens-before odnos
- Vidljivost podataka između Goroutines
- Zašto je `Once` race-free

---

# Uvod

Do sada smo koristili:

```go
var once sync.Once
```

i:

```go
once.Do(func(){

	initialize()

})
```

---

Ali postavlja se pitanje:

> Kako Go garantuje da će se funkcija izvršiti samo jednom kada postoji 1000 paralelnih Goroutines?

---

Odgovor uključuje:

- sinhronizaciju,
- memory model,
- atomic operacije,
- Mutex.

---

# Naivna implementacija

Mogli bismo pokušati:

```go
var done bool
```

---

Kod:

```go
func Do(f func()) {

	if !done {

		f()

		done = true

	}

}
```

---

Na prvi pogled:

radi.

---

Ali u concurrency svetu:

nije bezbedno.

---

# Race scenario

Imamo:

```text
Goroutine 1

        |

        Do()


Goroutine 2

        |

        Do()
```

---

Obe čitaju:

```go
done == false
```

---

Obe izvršavaju:

```go
f()
```

---

Rezultat:

```text
funkcija izvršena dva puta
```

---

# Problem sa boolean promenljivom

Problem nije samo:

```text
više izvršavanja
```

---

Problem je i:

```text
vidljivost memorije
```

---

Primer:

```go
var config Config
```

---

Goroutine 1:

```go
config = loadConfig()

done = true
```

---

Goroutine 2:

```go
if done {

	use(config)

}
```

---

Pitanje:

Da li Goroutine 2 sigurno vidi:

```go
config
```

?

---

Bez sinhronizacije:

ne postoji garancija.

---

# Interna struktura `sync.Once`

U pojednostavljenom obliku:

```go
type Once struct {

	done uint32

	m Mutex

}
```

---

Postoje dva glavna dela:

---

## 1. `done`

Predstavlja:

```text
da li je izvršeno
```

---

## 2. Mutex

Obezbeđuje:

```text
samo jedna Goroutine izvršava inicijalizaciju
```

---

# Zašto nije dovoljan samo atomic?

Mogli bismo:

```go
if atomic.LoadUint32(&done) == 0 {

	f()

	atomic.StoreUint32(&done,1)

}
```

---

Ali postoji problem.

---

Dve Goroutines:

```text
G1

pročita done == 0


G2

pročita done == 0
```

---

Obe ulaze u:

```go
f()
```

---

Atomic rešava:

```text
bezbedno čitanje/pisanje
```

---

Ali ne rešava:

```text
samo jedna izvršava funkciju
```

---

Zato je potreban Lock.

---

# Fast path

U realnoj implementaciji Go pokušava da bude brz.

---

Najčešći slučaj:

```text
Once je već izvršen
```

---

Tada nema potrebe za:

```go
Lock()
```

---

Prvo proverava:

```go
done == 1
```

---

Ako jeste:

odmah izlazi.

---

Mentalno:

```text
Do()

 |
 |
done?

YES

 |
 |
return
```

---

# Slow path

Ako:

```go
done == 0
```

---

Ulazi u sporiji deo:

```text
Lock

provera ponovo

execute

set done

Unlock
```

---

Zašto ponovo proverava?

---

Scenario:

```text
G1 dobija Lock

G1 izvršava f()


G2 čeka
```

---

Kada G1 završi:

G2 dobija Lock.

---

Ali G2 mora proveriti:

```go
done
```

opet.

---

# Double Check princip

Zato imamo:

```text
prva provera

↓

Lock

↓

druga provera

↓

izvrši
```

---

Slično kao:

```text
double checked locking
```

---

Ali ovde je implementirano bezbedno unutar Go runtime biblioteke.

---

# Happens-before odnos

Jedan od najvažnijih koncepata Go Memory Model-a.

---

Definicija:

Ako događaj A happens-before događaj B,

onda:

B garantovano vidi efekte A.

---

Kod:

```go
once.Do(func(){

	config = load()

})
```

---

Imamo:

```text
load()

↓

done = true

↓

sledeći Do()

↓

čitanje config
```

---

Go garantuje:

druga Goroutine vidi:

```go
config
```

u inicijalizovanom stanju.

---

# Primer

```go
var (
	once sync.Once
	value int
)


func GetValue() int {


	once.Do(func(){

		value = 42

	})


	return value

}
```

---

100 Goroutines:

```go
go GetValue()
```

---

Sve dobijaju:

```text
42
```

---

Ne:

```text
0

42

random
```

---

# Zašto je `sync.Once` race-free?

Zato što kombinuje:

---

## 1. Atomic operacije

Za brzu proveru:

```text
done
```

---

## 2. Mutex

Za zaštitu inicijalizacije.

---

## 3. Memory synchronization

Za vidljivost promena.

---

# Veza sa Go Memory Model-om

Concurrency nije samo:

```text
ko prvi izvrši
```

---

Postoji i pitanje:

```text
šta druga Goroutine vidi
```

---

Primer:

```go
data = initialize()

ready = true
```

---

Nije dovoljno:

```go
ready == true
```

---

Moramo garantovati:

```text
data je vidljiv
```

---

`sync.Once` to rešava.

---

# Once i panic ponašanje

Važna osobina:

Ako funkcija:

```go
once.Do(func(){

	panic("error")

})
```

---

`Once` smatra:

```text
izvršeno
```

---

Sledeći poziv:

```go
once.Do(...)
```

neće ponoviti funkciju.

---

Primer:

```go
func init(){

	panic("failed")

}
```

---

Posle:

```go
Once
```

ostaje:

```text
done
```

---

Ovo ćemo detaljnije analizirati u narednom delu.

---

# Mentalni model

`sync.Once` nije:

```text
if statement
```

---

Nije:

```text
boolean flag
```

---

To je:

```text
thread-safe state machine
```

koja garantuje:

```text
exactly once execution
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto običan boolean nije dovoljan,
- zašto atomic sam nije dovoljan,
- kako je približno implementiran `sync.Once`,
- razliku između fast i slow path,
- ulogu Mutex-a,
- šta je happens-before odnos,
- kako Go garantuje vidljivost memorije,
- zašto je `sync.Once` bezbedan u konkurentnom okruženju.

---

# `sync.Once` — Greške, ograničenja i production best practices

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Panic ponašanje `sync.Once`
- Problem neuspešne inicijalizacije
- Zašto `Once` nema retry
- Kako implementirati retry mehanizam
- Greške pri korišćenju `sync.Once`
- Thread-safe error handling
- Kada `Once` nije pravi izbor
- Production smernice

---

# Uvod

`sync.Once` izgleda jednostavno:

```go
once.Do(func(){

	initialize()

})
```

---

Ali u realnim sistemima postoje pitanja:

- Šta ako inicijalizacija padne?
- Šta ako baza nije dostupna?
- Šta ako želimo retry?
- Šta ako želimo ponovni pokušaj?

---

Odgovori nisu uvek intuitivni.

---

# 1. Panic ponašanje `sync.Once`

Jedna od najvažnijih osobina:

Ako funkcija prosleđena `Do()` pozove:

```go
panic()
```

`sync.Once` smatra da je izvršavanje završeno.

---

Primer:

```go
var once sync.Once


func initialize(){

	panic("failed")

}


func main(){

	once.Do(initialize)


	once.Do(initialize)

}
```

---

Rezultat:

Prvi poziv:

```text
panic
```

---

Drugi poziv:

```text
ne izvršava initialize ponovo
```

---

# Zašto?

Zato što `Once` garantuje:

```text
pozovi funkciju maksimalno jednom
```

---

Ne garantuje:

```text
pozovi funkciju jednom uspešno
```

---

Vrlo važna razlika.

---

# 2. Problem neuspešne inicijalizacije

Zamislimo:

```go
var (

	db *Database

	once sync.Once

)
```

---

Kod:

```go
func GetDatabase() *Database {


	once.Do(func(){

		db = connect()

	})


	return db

}
```

---

Šta ako:

```go
connect()
```

vrati grešku?

---

Primer:

```go
func connect() *Database {

	return nil

}
```

---

Rezultat:

```go
once

=
završeno
```

---

Ali:

```go
db == nil
```

---

Sledeći pozivi:

```go
GetDatabase()
```

više nikada neće pokušati konekciju.

---

# 3. Zašto `sync.Once` nema retry?

Namerno.

---

Njegova semantika:

```text
exactly once
```

---

Ako bi imao:

```text
retry

reset

ponovi
```

onda više ne bi bio:

```text
Once
```

---

To bi bio drugi alat.

---

# 4. Pogrešan obrazac

Primer:

```go
var once sync.Once

var err error


func Init() error {


	once.Do(func(){

		err = initialize()

	})


	return err

}
```

---

Problem:

Prvi pokušaj:

```text
initialize()

fail
```

---

Drugi pokušaj:

```text
once.Do()

skip
```

---

Dobijamo:

```text
trajni neuspeh
```

---

# 5. Bolji obrazac sa greškom

Možemo čuvati rezultat:

```go
var (

	once sync.Once

	resource *Resource

	err error

)
```

---

Primer:

```go
func GetResource() (*Resource, error) {


	once.Do(func(){

		resource, err = createResource()

	})


	return resource, err

}
```

---

Prednost:

Pozivaoci znaju:

```go
resource

+
error
```

---

Ali:

nema retry.

---

# 6. Retry inicijalizacija

Ako nam treba retry:

ne koristimo samo:

```go
sync.Once
```

---

Primer:

```go
type ResourceManager struct {

	mu sync.Mutex

	resource *Resource

}
```

---

Metoda:

```go
func (r *ResourceManager) Get() (*Resource,error){

	r.mu.Lock()

	defer r.mu.Unlock()


	if r.resource == nil {

		resource, err := create()

		if err != nil {

			return nil, err

		}

		r.resource = resource

	}


	return r.resource,nil

}
```

---

Ovde:

ako prvi pokušaj padne:

sledeći pokušaj može ponovo.

---

# 7. Once + sync.Cond / Channel pattern

U složenim sistemima često imamo:

```text
jedna Goroutine inicijalizuje

ostale čekaju
```

---

Primer:

```text
G1

initialize


G2

wait


G3

wait
```

---

Za ovo se često koriste:

- channels,
- context,
- condition variables.

---

`sync.Once` nije uvek dovoljan.

---

# 8. Česta greška — korišćenje za mutable state

Pogrešno:

```go
var once sync.Once


func Increment(){

	once.Do(func(){

		counter++

	})

}
```

---

Rezultat:

```text
counter++

samo jednom
```

---

Za zaštitu promena koristi:

```go
Mutex
```

ili:

```go
atomic
```

---

# 9. Česta greška — previše globalnog stanja

Primer:

```go
var (

	dbOnce sync.Once

	cacheOnce sync.Once

	loggerOnce sync.Once

	configOnce sync.Once

)
```

---

Moguće.

Ali veliki broj Singleton resursa može napraviti:

- skrivene zavisnosti,
- komplikovano testiranje,
- težak lifecycle.

---

# 10. Once nije lifecycle manager

Pogrešna pretpostavka:

```text
Once kreira resurs
i upravlja njime
```

---

Ne.

---

Once samo kaže:

```text
ovu funkciju pozovi jednom
```

---

Ne rešava:

- close,
- shutdown,
- reconnect,
- refresh.

---

Primer:

```go
db.Close()
```

mora imati drugi mehanizam.

---

# 11. Production pravila

---

## Pravilo 1

Koristi `sync.Once` za:

```text
jednokratnu inicijalizaciju
```

---

## Pravilo 2

Ne koristi ga za:

```text
periodične operacije
```

---

## Pravilo 3

Razmisli šta se dešava ako inicijalizacija padne.

---

Pitaj:

```text
Da li želim retry?
```

---

Ako je odgovor:

```text
da
```

verovatno treba drugi obrazac.

---

## Pravilo 4

Čuvaj rezultat inicijalizacije:

```go
value

+

error
```

---

## Pravilo 5

Testiraj:

- uspešnu inicijalizaciju,
- neuspešnu inicijalizaciju,
- paralelne pozive.

---

# Mentalni model

Zapamti:

```text
sync.Once

=

execute exactly once
```

---

Ne:

```text
initialize until success
```

---

Ne:

```text
resource manager
```

---

Ne:

```text
retry mechanism
```

---

# 📋 Rezime

U ovom delu naučili smo:

- kako `sync.Once` tretira panic,
- zašto neuspešna inicijalizacija može biti problem,
- zašto `Once` nema retry mehanizam,
- kako čuvati greške iz inicijalizacije,
- kada koristiti Mutex umesto Once,
- kada koristiti druge concurrency obrasce,
- production pravila za bezbednu upotrebu.

---

# `sync.Once` — Završni rezime, poređenje i praktični zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 3/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/03-once.md`

---

# 📚 Sadržaj ovog dela

- Kompletan rezime `sync.Once`
- `sync.Once` vs `Mutex`
- `sync.Once` vs `atomic`
- `sync.Once` vs package initialization
- Kada koristiti koji pristup
- Production checklist
- Pitanja za proveru znanja
- Praktični zadaci

---

# Kompletan rezime `sync.Once`

`sync.Once` je concurrency primitive čija je jedina svrha:

> Garantovati da se određena funkcija izvrši najviše jednom, bez obzira koliko Goroutines je poziva.

---

Osnovni API:

```go
var once sync.Once

once.Do(func(){

	initialize()

})
```

---

Prvi poziv:

```text
izvršava funkciju
```

---

Svi sledeći:

```text
preskaču
```

---

# Šta `sync.Once` garantuje?

---

## 1. Jednokratno izvršavanje

Primer:

```go
once.Do(init)
```

---

Rezultat:

```text
init()

0 ili 1 put
```

---

## 2. Thread safety

Više Goroutines:

```go
go once.Do(init)

go once.Do(init)
```

---

Bezbedno.

---

## 3. Memory visibility

Promene napravljene tokom inicijalizacije:

```go
config = loadConfig()
```

vidljive su svim Goroutines nakon:

```go
Do()
```

---

# Šta `sync.Once` NE garantuje?

---

## 1. Uspešnu inicijalizaciju

Ako:

```go
init()
```

padne,

Once ne pokušava ponovo.

---

## 2. Retry

Ne postoji:

```go
Retry()
```

---

## 3. Reset

Ne postoji:

```go
Reset()
```

---

## 4. Lifecycle

Ne upravlja:

- start,
- stop,
- close,
- reconnect.

---

# `sync.Once` vs `sync.Mutex`

Veoma važno poređenje.

---

# Mutex

Koristi se za:

```text
zaštitu deljenog stanja
```

---

Primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Može se koristiti:

```text
beskonačno puta
```

---

---

# Once

Koristi se za:

```text
jednokratnu inicijalizaciju
```

---

Primer:

```go
once.Do(createCache)
```

---

Izvršava:

```text
samo jednom
```

---

# Poređenje

| | Mutex | Once |
|-|-|-|
| Namena | zaštita podataka | inicijalizacija |
| Broj korišćenja | više puta | jednom |
| Lock/Unlock | da | ne |
| Mutable state | da | ne |
| Retry | moguće | ne |

---

# Primer odluke

Imamo:

```go
database connection
```

---

Pitanje:

Da li pravimo konekciju jednom?

Da:

```go
sync.Once
```

---

Da li štitimo pristup konekciji?

Da:

```go
Mutex/RWMutex
```

---

Moguće je koristiti oba.

---

# `sync.Once` vs `atomic`

Još jedna važna razlika.

---

Atomic služi za:

```text
male, jednostavne vrednosti
```

---

Primer:

```go
atomic.AddInt64(
	&counter,
	1,
)
```

---

Once služi za:

```text
kompleksnu inicijalizaciju
```

---

Primer:

```go
createDatabase()

loadConfig()

buildCache()
```

---

# Zašto atomic nije zamena?

Možemo napisati:

```go
if atomic.LoadUint32(&done)==0 {

	initialize()

	atomic.StoreUint32(&done,1)

}
```

---

Ali dve Goroutines mogu:

```text
obe videti 0
```

---

i obe izvršiti:

```go
initialize()
```

---

Atomic štiti vrednost.

Ne rešava:

```text
exclusive execution
```

---

# `sync.Once` vs package initialization

Go ima još jedan način:

```go
init()
```

---

Primer:

```go
func init(){

	loadConfig()

}
```

---

Ovo se izvršava automatski.

---

# Razlika

## `init()`

Izvršava se:

```text
pri startovanju programa
```

---

## `sync.Once`

Izvršava se:

```text
kada prvi put zatreba
```

---

# Poređenje

| | init() | sync.Once |
|-|-|-|
| Kada | startup | lazy |
| Kontrola | mala | velika |
| Testiranje | teže | lakše |
| Odloženo učitavanje | ne | da |

---

# Kada koristiti `init()`?

Dobro za:

- registraciju paketa,
- konstantne pripreme,
- compile-time poznate stvari.

---

Primer:

```go
func init(){

	registerDriver()

}
```

---

# Kada koristiti `sync.Once`?

Dobro za:

- database client,
- cache,
- logger,
- expensive computation,
- lazy loading.

---

# Decision tabela

| Problem | Alat |
|-|-|
| Counter | atomic |
| Shared map | Mutex/RWMutex |
| Jednom kreiranje resursa | Once |
| Komunikacija Goroutines | Channel |
| Cancellation | Context |
| Retry | custom logic |

---

# Production checklist

Pre korišćenja `sync.Once` pitaj:

---

## ✅ 1. Da li operacija mora biti izvršena samo jednom?

Ako ne:

ne koristi.

---

## ✅ 2. Šta ako inicijalizacija padne?

Definiši ponašanje.

---

## ✅ 3. Da li treba retry?

Ako da:

Once verovatno nije dovoljan.

---

## ✅ 4. Da li resurs ima lifecycle?

Primer:

```text
Connect

Use

Close
```

---

Once rešava samo:

```text
Connect jednom
```

---

## ✅ 5. Da li otežava testiranje?

Izbegavati nepotrebne globalne instance.

---

# Pitanja za proveru znanja

---

## 1.

Koja je glavna svrha `sync.Once`?

---

## 2.

Zašto običan boolean nije dovoljan?

---

## 3.

Da li `sync.Once` garantuje uspešnu inicijalizaciju?

---

## 4.

Koja je razlika između:

```go
sync.Once
```

i:

```go
sync.Mutex
```

---

## 5.

Zašto atomic nije zamena za Once?

---

## 6.

Kada bi koristio:

```go
init()
```

a kada:

```go
sync.Once
```

?

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnove

## Zadatak 1

Napraviti:

```go
ConfigLoader
```

koji učitava konfiguraciju samo jednom.

---

Zahtevi:

- `sync.Once`
- `GetConfig()`

---

# 🟡 Nivo 2 — Praktično

## Zadatak 2

Napraviti:

```go
DatabaseManager
```

koji:

- kreira konekciju jednom,
- vraća istu instancu,
- podržava više Goroutines.

---

# 🟠 Nivo 3 — Napredno

## Zadatak 3

Napraviti:

```go
CacheManager
```

koji koristi:

- `sync.Once`
- `RWMutex`

---

Zahtevi:

- lazy initialization,
- thread-safe pristup.

---

# 🔴 Nivo 4 — Senior

## Zadatak 4

Napraviti sistem inicijalizacije:

Zahtevi:

- više resursa,
- paralelni pozivi,
- error handling,
- retry za neuspešne inicijalizacije.

---

Analizirati:

gde koristiti:

```text
Once

Mutex

Channel

Context
```

---

# Završni mentalni model

Zapamti:

```text
Mutex

=
štiti podatak


RWMutex

=
optimizuje read-heavy pristup


Once

=
jednom kreiraj


Atomic

=
mala brza promena vrednosti


Channel

=
komunikacija između Goroutines
```

---

### ➡️ Sledeća lekcija **[**Timeouts**](04-timeouts.md)**

Obradićemo:

- zašto su timeout-i važni,
- `context.WithTimeout`,
- `context.WithDeadline`,
- timeout obrasce,
- HTTP/database timeout,
- sprečavanje zaglavljivanja Goroutines.
