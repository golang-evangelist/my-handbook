# Semaphore Pattern — Uvod

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/07-semaphore-pattern.md`

---

# 📚 Sadržaj

- Šta je Semaphore
- Problem koji rešava
- Semaphore vs Worker Pool
- Koncept permit-a
- Counting Semaphore
- Go implementacija pomoću kanala
- Production primeri

---

# Uvod

Pretpostavimo da imamo servis:

```text
1000 zahteva

↓

1000 Goroutines
```

Svaki zahtev radi:

```text
HTTP API poziv
```

Problem:

Eksterni API dozvoljava:

```
50

zahteva istovremeno
```

Ako pošaljemo 1000:

- dobićemo greške,
- povećati latenciju,
- možda izazvati blokadu.

Potrebno je ograničenje.

---

# Rešenje

Koristimo:

```
Semaphore
```

On kontroliše koliko Goroutines može istovremeno da uđe u kritičnu sekciju.

---

# Osnovna ideja

Semaphore ima određeni broj dozvola:

```
Capacity = 3
```

To znači:

```
Maksimalno 3

operacije

istovremeno
```

---

# Vizuelni model

Semaphore:

```text
+---+---+---+
| 1 | 2 | 3 |
+---+---+---+
```

Tri dostupna permit-a.

---

Goroutine traži dozvolu:

```text
Acquire

↓

[Permit]

↓

Work

↓

Release
```

---

# Permit

Permit znači:

> "Imam pravo da trenutno koristim jedan slot."

Tok:

```text
Acquire

↓

dobio permit

↓

izvrši posao

↓

Release permit
```

---

# Counting Semaphore

Najčešći tip.

Primer:

```text
Semaphore = 5
```

Dozvoljava:

```
5

istovremenih operacija
```

Šesta Goroutine mora da čeka.

---

# Primer

Imamo:

```text
10 Goroutines
```

i:

```text
Semaphore(3)
```

Izvršavanje:

```
Worker 1  ✓
Worker 2  ✓
Worker 3  ✓

Worker 4  čekanje
Worker 5  čekanje
Worker 6  čekanje
```

Kada Worker 2 završi:

```
Release()

↓

Worker 4 ulazi
```

---

# Semaphore u Go-u

Go nema ugrađeni `Semaphore` tip u osnovnom jeziku.

Najčešći način je:

```
Buffered channel
```

---

# Implementacija

```go
type Semaphore chan struct{}
```

Kreiranje:

```go
sem := make(
	chan struct{},
	3,
)
```

Kapacitet:

```
3
```

---

# Acquire

Dobijanje dozvole:

```go
sem <- struct{}{}
```

Ako nema mesta:

```text
blokira
```

---

# Release

Vraćanje dozvole:

```go
<-sem
```

---

# Kompletan primer

```go
func worker(
	id int,
	sem chan struct{},
) {

	sem <- struct{}{}

	defer func() {
		<-sem
	}()

	fmt.Println(
		"worker",
		id,
	)

	time.Sleep(time.Second)
}
```

---

# Pokretanje

```go
sem := make(chan struct{}, 3)

for i := 0; i < 10; i++ {

	go worker(
		i,
		sem,
	)
}
```

Iako postoji:

```
10 Goroutines
```

samo:

```
3

rade istovremeno
```

---

# Semaphore vs Worker Pool

## Worker Pool

```text
Jobs

↓

Workers

↓

Results
```

Karakteristike:

- worker-i postoje unapred,
- poslovi čekaju u redu,
- pogodno za batch obradu.

---

## Semaphore

```text
Goroutine

↓

Acquire

↓

Resource
```

Karakteristike:

- Goroutine već postoji,
- samo se ograničava pristup resursu,
- pogodno za zaštitu ograničenih resursa.

---

# Primer poređenja

## Worker Pool

Obrada:

```
1 000 000 slika
```

Model:

```
Queue

↓

20 Workers
```

---

## Semaphore

HTTP servis:

```
10 000 zahteva

↓

max 100 paralelnih API poziva
```

Model:

```
Goroutine

↓

Semaphore

↓

API call
```

---

# Production primeri

## Database

Baza dozvoljava:

```
100 konekcija
```

Semaphore:

```text
100 permits
```

---

## External API

API limit:

```
50 concurrent requests
```

Semaphore:

```text
50 permits
```

---

## File processing

Disk može optimalno obraditi:

```
20 fajlova
```

Semaphore:

```text
20 permits
```

---

# Najčešće greške

## Greška #1

Zaboravljen `Release`

Loše:

```go
sem <- struct{}{}

process()
```

Ako `process()` napravi panic ili return:

permit ostaje zauzet.

---

Bolje:

```go
sem <- struct{}{}

defer func() {
	<-sem
}()
```

---

## Greška #2

Prevelik kapacitet

Ako je limit resursa:

```
50
```

Semaphore:

```
10000
```

ne rešava problem.

---

## Greška #3

Koristiti Semaphore tamo gde treba Worker Pool

Ako imaš ogroman broj batch poslova, Worker Pool je često bolji dizajn.

---

# Best Practices

✅ Koristi `defer` za `Release`.

✅ Veličinu Semaphore-a zasnivaj na realnom ograničenju resursa.

✅ Kombinuj sa `context.Context` za timeout i cancellation.

✅ Razlikuj zaštitu resursa od organizacije obrade.

---

# Mentalni model

Nemoj razmišljati:

```text
Imam mnogo Goroutines

↓

problem
```

Razmišljaj:

```text
Imam ograničen resurs

↓

Semaphore kontroliše pristup
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Semaphore Pattern

✅ šta je permit

✅ kako Counting Semaphore funkcioniše

✅ kako implementirati Semaphore pomoću kanala

✅ razliku između Semaphore i Worker Pool

✅ gde se Semaphore koristi u produkciji

---

# Semaphore Pattern — Context, Timeout i Cancellation

---

# 📚 Sadržaj

- Problem blokirajućeg Acquire
- Semaphore + context.Context
- Timeout pri čekanju permit-a
- Cancellation
- `golang.org/x/sync/semaphore`
- Production obrasci

---

# Problem osnovnog Semaphore-a

Prethodni primer:

```go
sem <- struct{}{}
```

čeka dok se ne oslobodi mesto.

To znači:

```
Goroutine

↓

čeka

↓

neograničeno
```

---

# Zašto je to problem?

Zamisli:

```text
API poziv

↓

Acquire semaphore

↓

čeka 10 minuta
```

Ta Goroutine zauzima memoriju i resurse.

U produkciji često želimo:

```
čekaj maksimalno X vremena
```

ili:

```
prekini odmah kada se context otkaže
```

---

# Rešenje: context.Context

Umesto običnog slanja:

```go
sem <- struct{}{}
```

koristimo:

```go
select
```

---

# Semaphore sa cancellation

Primer:

```go
func acquire(
	ctx context.Context,
	sem chan struct{},
) error {

	select {

	case sem <- struct{}{}:
		return nil

	case <-ctx.Done():
		return ctx.Err()
	}
}
```

---

# Kako radi?

Postoje dva moguća događaja:

## Opcija 1

Ima permit-a:

```text
Acquire

↓

Success
```

---

## Opcija 2

Context je otkazan:

```text
ctx.Done()

↓

Return error
```

---

# Korišćenje

```go
err := acquire(
	ctx,
	sem,
)

if err != nil {
	return err
}

defer func() {
	<-sem
}()

process()
```

Sada čekanje više nije beskonačno.

---

# Timeout

Često koristimo:

```go
context.WithTimeout
```

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		time.Second,
	)

defer cancel()
```

---

Ako permit nije dostupan u roku od:

```
1 sekunde
```

dobijamo:

```go
context deadline exceeded
```

---

# Vizuelni tok

```text
Acquire

↓

Čekanje permit-a

↓

+----------------+
|                |
| permit free    |
|                |
+----------------+

ili

+----------------+
|                |
| timeout        |
|                |
+----------------+
```

---

# Cancellation

Primer:

```go
ctx, cancel :=
	context.WithCancel(
		context.Background(),
	)
```

Kasnije:

```go
cancel()
```

Rezultat:

Sve Goroutines koje čekaju:

```go
<-ctx.Done()
```

dobijaju signal.

---

# Production primer

HTTP handler:

```text
Request

↓

Create Context

↓

Acquire Semaphore

↓

Call External API

↓

Release
```

Ako korisnik prekine zahtev:

```text
Client disconnect

↓

Context cancelled

↓

Semaphore wait prekida
```

---

# `golang.org/x/sync/semaphore`

Go ekosistem ima standardnu biblioteku:

```go
golang.org/x/sync/semaphore
```

koja pruža napredniji Semaphore.

---

# Weighted Semaphore

Za razliku od običnog:

```text
1 permit

=

1 jedinica
```

Weighted Semaphore dozvoljava:

```
N permit-a
```

odjednom.

---

# Primer

Imamo resurs:

```
100 jedinica memorije
```

Operacije troše:

```
10
30
50
```

Weighted Semaphore omogućava kontrolu ukupne potrošnje.

---

# Primer korišćenja

```go
sem := semaphore.NewWeighted(100)
```

Acquire:

```go
err := sem.Acquire(
	ctx,
	10,
)
```

Release:

```go
sem.Release(10)
```

---

# Zašto je Weighted koristan?

Nisu svi poslovi isti.

Primer:

```text
Image resize

=

10 MB
```

---

```text
Video encoding

=

80 MB
```

Običan Semaphore ne razlikuje težinu.

Weighted Semaphore razlikuje.

---

# Worker Pool vs Semaphore sa Context-om

## Worker Pool

Kontroliše:

```
broj aktivnih worker-a
```

---

## Semaphore

Kontroliše:

```
pristup resursu
```

---

Primer:

```text
10 000 requests

↓

Goroutines

↓

Semaphore(100)

↓

Database
```

---

# Najčešće greške

## Greška #1

Čekanje bez timeout-a.

Loše:

```go
Acquire()
```

bez mogućnosti prekida.

---

## Greška #2

Ignorisanje `ctx.Err()`.

Ako Acquire ne uspe, posao ne treba nastaviti.

---

## Greška #3

Release pogrešne količine.

Kod Weighted Semaphore:

```go
Acquire(10)

↓

Release(5)
```

ostavlja zauzet resurs.

---

# Best Practices

✅ Uvek podrži `context.Context`.

✅ Koristi timeout za spoljne resurse.

✅ Koristi Weighted Semaphore kada poslovi imaju različitu "težinu".

✅ Oslobodi permit pomoću `defer`.

---

# Mentalni model

Nemoj razmišljati:

```text
Čekaj dok ne dobiješ dozvolu
```

Razmišljaj:

```text
Pokušaj da dobiješ dozvolu

↓

Ako možeš

↓

Radi

↓

Ako ne možeš

↓

Kontrolisano odustani
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto običan Semaphore nije dovoljan

✅ kako koristiti `context.Context`

✅ kako implementirati timeout pri čekanju permit-a

✅ kako funkcioniše cancellation

✅ šta je Weighted Semaphore

✅ kada koristiti `golang.org/x/sync/semaphore`

---

# Semaphore Pattern — Rate Limiting

---

# 📚 Sadržaj

- Concurrency Limit vs Rate Limit
- Zašto Semaphore nije Rate Limiter
- Token Bucket algoritam
- Leaky Bucket algoritam
- Kombinovanje Semaphore i Rate Limiter-a
- API zaštita

---

# Uvod

Čest problem:

Eksterni servis kaže:

```
Maksimalno:

100 zahteva u sekundi
```

Početnik može pokušati:

```go
Semaphore(100)
```

Ali to nije isto.

---

# Concurrency Limit

Semaphore kontroliše:

```
Koliko operacija

može biti aktivno

u istom trenutku
```

Primer:

```text
Semaphore = 10
```

Znači:

```
10 aktivnih zahteva
```

---

# Rate Limit

Rate limiter kontroliše:

```
Koliko operacija

može da počne

u vremenskom intervalu
```

Primer:

```text
100 requests / second
```

---

# Primer razlike

Pretpostavimo:

API zahtev traje:

```
1 sekunda
```

Semaphore:

```
100 permit-a
```

Dozvoljava:

```
100 aktivnih zahteva
```

---

Ali ako svaki zahtev traje:

```
10 sekundi
```

onda možeš poslati:

```
1000 zahteva

za 10 sekundi
```

što je:

```
100/s
```

Drugačiji scenario daje drugačiji rezultat.

---

# Vizuelna razlika

## Semaphore

```text
ACTIVE REQUESTS

[1][2][3][4][5]

MAX = 5
```

---

## Rate Limiter

```text
TIME

0s   1s   2s

|----|----|

100  100  100

requests
```

---

# Token Bucket algoritam

Jedan od najčešće korišćenih algoritama.

Ideja:

Postoji kanta:

```text
+-------------+
| token token |
| token token |
| token token |
+-------------+
```

Token predstavlja dozvolu za izvršavanje jedne operacije.

---

# Kako radi?

Tokeni se dodaju brzinom:

```
R tokena / sekundi
```

Maksimalan broj tokena:

```
Bucket capacity
```

---

Primer:

```
10 tokena/sec

capacity = 50
```

Možeš:

- odmah poslati do 50 zahteva,
- zatim nastaviti prosečno 10/sec.

---

# Zašto Burst postoji?

Zamisli korisnika koji nije slao zahteve 5 sekundi.

Tokeni su se nakupili.

Kada pošalje zahtev:

```
burst

↓

dozvoljen
```

To je korisno za realne sisteme.

---

# Leaky Bucket algoritam

Drugačiji pristup.

Zamisli kantu sa rupom:

```
      |
      |
   -------
  |       |
  |       |
  |_______|
      |
      ↓
 konstantan izlaz
```

Ulazi mogu varirati.

Izlaz je kontrolisan.

---

# Karakteristike

Token Bucket:

```
Dozvoljava burst
```

---

Leaky Bucket:

```
Pegla saobraćaj
```

---

# Go Rate Limiter

Najčešće se koristi:

```go
golang.org/x/time/rate
```

Primer:

```go
limiter :=
	rate.NewLimiter(
		rate.Limit(100),
		50,
	)
```

Značenje:

```
100 događaja/sec

burst 50
```

---

# Korišćenje

```go
if err := limiter.Wait(ctx); err != nil {
	return err
}

process()
```

Ako nema tokena:

```
čeka

ili

prekida preko context-a
```

---

# Semaphore + Rate Limiter

U produkciji se često kombinuju.

Primer:

Eksterni API:

```
100 requests/sec

50 concurrent requests
```

Rešenje:

```
Rate Limiter

↓

100/sec

↓

Semaphore

↓

50 active
```

---

# Vizuelno

```text
Request

↓

Rate Limiter

↓

Concurrency Semaphore

↓

External API
```

---

# Production primer

Servis šalje email:

Ograničenja:

SMTP server:

```
500 email/min
```

Aplikacija:

```
max 20 paralelnih slanja
```

Arhitektura:

```text
Jobs

↓

Rate Limiter

↓

Semaphore(20)

↓

SMTP
```

---

# Najčešće greške

## Greška #1

Koristiti Semaphore kao zamenu za Rate Limiter.

Ograničavaš broj aktivnih operacija, ali ne i brzinu.

---

## Greška #2

Rate limit bez concurrency limita.

Možeš dozvoliti:

```
1000 zahteva/sec
```

koji svi traju dugo i preopterete sistem.

---

## Greška #3

Ignorisati burst ponašanje.

Neki sistemi dozvoljavaju kratkotrajne skokove opterećenja.

---

# Best Practices

✅ Razdvoji concurrency limit i rate limit.

✅ Koristi Semaphore za zaštitu resursa.

✅ Koristi Rate Limiter za kontrolu brzine.

✅ Kombinuj ih kada sistem ima više ograničenja.

✅ Uvek podrži `context.Context`.

---

# Mentalni model

Zapamti:

```
Semaphore

=

Koliko istovremeno?
```

---

```
Rate Limiter

=

Koliko brzo?
```

---

Zajedno:

```
Koliko brzo

+

koliko paralelno
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ razliku između Semaphore i Rate Limiter-a

✅ šta je concurrency limit

✅ šta je rate limit

✅ kako radi Token Bucket

✅ kako radi Leaky Bucket

✅ kako kombinovati Semaphore i Rate Limiter

---

# Semaphore Pattern — Resource Protection

---

# 📚 Sadržaj

- Semaphore kao zaštita resursa
- Database connection limit
- HTTP client limit
- File descriptor limit
- Resource Pool koncept
- Production primeri

---

# Uvod

Svaki sistem ima ograničenja.

Primer:

Aplikacija može kreirati:

```
100 000 Goroutines
```

Ali baza možda dozvoljava:

```
100 konekcija
```

Problem nije u Goroutines.

Problem je:

```
ograničen resurs
```

---

# Semaphore kao zaštita

Semaphore postavlja granicu:

```text
Maksimalno N

istovremenih pristupa
```

Drugim rečima:

```
Resurs

↓

kontrolisan pristup
```

---

# 1. Database Connection Limit

Baze podataka imaju ograničen broj konekcija.

Primer:

```text
PostgreSQL

max connections = 200
```

Ako aplikacija pokrene:

```
5000 paralelnih query-ja
```

može doći do:

- odbijanja konekcija,
- povećanja latencije,
- pada baze.

---

# Rešenje

Semaphore:

```text
Request

↓

Acquire

↓

Database Query

↓

Release
```

---

# Primer

```go
dbSem :=
	make(chan struct{}, 100)
```

Funkcija:

```go
func query(
	ctx context.Context,
) error {

	select {
	case dbSem <- struct{}{}:

	case <-ctx.Done():
		return ctx.Err()
	}

	defer func() {
		<-dbSem
	}()

	return executeQuery()
}
```

---

# Rezultat

Iako postoji:

```
10 000 request-a
```

maksimalno:

```
100

query-ja
```

se izvršava istovremeno.

---

# Napomena

Go database/sql paket već interno upravlja connection pool-om:

```go
db.SetMaxOpenConns(100)
```

U tom slučaju dodatni Semaphore možda nije potreban.

Važno je ne duplirati ograničenja bez razloga.

---

# 2. HTTP Client Limit

Eksterni API često ima:

```
Maximum concurrent requests = 50
```

Bez zaštite:

```text
10000 Goroutines

↓

10000 HTTP requests
```

---

Sa Semaphore:

```text
10000 Goroutines

↓

Semaphore(50)

↓

API
```

---

# Primer

```go
apiSem :=
	make(chan struct{}, 50)
```

Pre poziva:

```go
apiSem <- struct{}{}

resp, err :=
	client.Do(req)

<-apiSem
```

Bolje:

```go
apiSem <- struct{}{}

defer func() {
	<-apiSem
}()
```

---

# 3. File Descriptor Limit

Operativni sistem ograničava broj otvorenih fajlova.

Primer:

```
ulimit -n

1024
```

Ako aplikacija otvori:

```
10000 fajlova
```

može dobiti:

```
too many open files
```

---

Semaphore:

```go
fileSem :=
	make(chan struct{}, 100)
```

Tok:

```text
Acquire

↓

Open File

↓

Read

↓

Close

↓

Release
```

---

# 4. CPU Intensive Operations

Primer:

- kompresija,
- enkripcija,
- obrada slike.

Ako pokreneš:

```
1000 CPU-heavy Goroutines
```

možeš dobiti:

- scheduler overhead,
- contention,
- lošije performanse.

---

Semaphore može ograničiti:

```
broj aktivnih CPU operacija
```

Primer:

```go
cpuSem :=
	make(chan struct{}, runtime.NumCPU())
```

---

# Resource Pool koncept

Semaphore i Resource Pool nisu isto.

---

## Resource Pool

Ima stvarne objekte:

```text
Connection

Connection

Connection
```

Korisnik pozajmljuje objekat.

---

## Semaphore

Ima samo dozvole:

```text
Permit

Permit

Permit
```

Ne upravlja objektom.

---

# Primer razlike

Database pool:

```
Connection Pool

↓

dobij konekciju

↓

koristi

↓

vrati
```

---

Semaphore:

```
Permit

↓

smeš koristiti resurs

↓

Release
```

---

# Kombinovanje

Često se koriste zajedno.

Primer:

```text
HTTP Request

↓

Rate Limiter

↓

Semaphore

↓

Connection Pool

↓

Database
```

Svaki sloj štiti drugi deo sistema.

---

# Production arhitektura

Primer mikroservisa:

```text
Incoming Requests

↓

Rate Limiter

↓

Worker Pool

↓

Semaphore

↓

External API

↓

Database Pool
```

Svaki obrazac ima svoju ulogu.

---

# Najčešće greške

## Greška #1

Semaphore veći od stvarnog limita.

Primer:

API dozvoljava:

```
50
```

Semaphore:

```
500
```

Nema zaštite.

---

## Greška #2

Dupliranje kontrola.

Primer:

```
Database pool

+

Semaphore

+

Worker Pool
```

bez jasnog razloga.

Dodaje kompleksnost.

---

## Greška #3

Ne oslobađati permit.

Ako se zaboravi:

```go
<-sem
```

kapacitet se postepeno smanjuje.

---

# Best Practices

✅ Zaštiti resurs, ne Goroutine.

✅ Granicu zasnivaj na realnom limitu.

✅ Koristi `defer` za Release.

✅ Razlikuj Resource Pool od Semaphore-a.

✅ Kombinuj obrasce samo kada svaki rešava poseban problem.

---

# Mentalni model

Nemoj razmišljati:

```text
Imam previše posla
```

Razmišljaj:

```text
Imam ograničen resurs

↓

kontrolišem pristup
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako Semaphore štiti resurse

✅ ograničavanje database pristupa

✅ ograničavanje HTTP poziva

✅ zaštitu file descriptor-a

✅ razliku između Semaphore-a i Resource Pool-a

✅ kako se Semaphore uklapa u produkcionu arhitekturu

---

# Semaphore Pattern — Napredni obrasci

---

# 📚 Sadržaj

- Semaphore + Worker Pool kombinacija
- Multi-resource ograničavanje
- Hijerarhijski Semaphore
- Redosled Acquire/Release
- Deadlock rizici
- Production dizajn

---

# Uvod

U jednostavnom sistemu:

```text
Request

↓

Semaphore

↓

Resource
```

Ali realan zahtev često izgleda ovako:

```text
Request

↓

CPU

↓

HTTP API

↓

Database
```

Svaki deo ima drugačije ograničenje.

---

# Semaphore + Worker Pool

Ova dva obrasca rešavaju različite probleme.

---

## Worker Pool

Kontroliše:

```
koliko poslova postoji
```

Primer:

```
20 worker-a
```

---

## Semaphore

Kontroliše:

```
koliko resursa može biti zauzeto
```

Primer:

```
max 5 API poziva
```

---

# Kombinovani primer

Imamo:

```
10000 poslova
```

Ne želimo:

```
10000 Goroutines
```

Koristimo:

```text
Worker Pool

↓

20 Worker-a

↓

Semaphore(5)

↓

External API
```

---

# Tok izvršavanja

```text
Job Queue

↓

Worker

↓

Acquire API permit

↓

HTTP Request

↓

Release permit

↓

Next Job
```

---

# Zašto ne samo Semaphore?

Mogli bismo:

```text
10000 Goroutines

↓

Semaphore(5)
```

Ali:

- svih 10000 Goroutines postoji,
- veća potrošnja memorije,
- slabija kontrola reda poslova.

Worker Pool dodatno kontroliše broj aktivnih izvršilaca.

---

# Multi-resource ograničavanje

Primer:

Servis radi:

```
Image Processing
```

Potrebno:

1. CPU slot
2. Disk pristup
3. Storage API

---

Imamo:

```text
CPU Semaphore

↓

Disk Semaphore

↓

API Semaphore
```

---

# Primer toka

```text
Job

↓

Acquire CPU

↓

Decode Image

↓

Acquire Disk

↓

Save File

↓

Acquire API

↓

Upload
```

---

# Problem

Ako redosled nije isti svuda:

Goroutine 1:

```
Acquire CPU

↓

čeka Disk
```

Goroutine 2:

```
Acquire Disk

↓

čeka CPU
```

Dobijamo:

```
Deadlock
```

---

# Deadlock scenario

```text
G1:

CPU locked

waiting Disk


G2:

Disk locked

waiting CPU
```

Niko ne može da nastavi.

---

# Pravilo

Kod više Semaphore-a:

> Uvek koristi isti redosled Acquire operacija.

Na primer:

```text
1. CPU

2. Disk

3. Network
```

Svi delovi sistema moraju pratiti isti redosled.

---

# Hijerarhijski Semaphore

Nekada imamo:

```text
Global Limit

↓

Service Limit

↓

Operation Limit
```

---

Primer:

Cela aplikacija:

```
100 aktivnih operacija
```

---

Jedan servis:

```
30 operacija
```

---

Jedan tip operacije:

```
10 operacija
```

---

Vizuelno:

```text
Application Semaphore(100)

        ↓

Service Semaphore(30)

        ↓

Operation Semaphore(10)
```

---

# Zašto koristiti hijerarhiju?

Omogućava:

- globalnu zaštitu sistema,
- fer raspodelu resursa,
- izolaciju servisa.

---

# Production primer

Mikroservis:

```text
Incoming Requests

↓

Global Semaphore(1000)

↓

Image Service Semaphore(100)

↓

Resize Semaphore(10)

↓

GPU
```

Svaki nivo štiti drugačiji resurs.

---

# Acquire/Release pravilo

Ako radiš:

```go
Acquire A

Acquire B
```

onda:

```go
Release B

Release A
```

---

Drugim rečima:

```
Last acquired

↓

First released
```

kao stack.

---

# Loš primer

```go
Acquire(cpu)

Acquire(memory)

defer Release(cpu)

defer Release(memory)
```

Redosled nije intuitivan.

---

Bolje:

```go
Acquire(cpu)

defer Release(cpu)


Acquire(memory)

defer Release(memory)
```

Go `defer` izvršava obrnutim redom.

---

# Semaphore nije zamena za arhitekturu

Greška:

```
Samo dodaj Semaphore
```

i problem nestaje.

Ne.

Potrebno je razumeti:

- gde nastaje pritisak,
- koji resurs je ograničen,
- kakav je workload.

---

# Najčešće greške

## Greška #1

Više Semaphore-a bez definisanog redosleda.

Rezultat:

deadlock.

---

## Greška #2

Previše složen hijerarhijski sistem.

Ako svaki mali deo ima svoj limit:

kod postaje težak za održavanje.

---

## Greška #3

Mešanje odgovornosti.

Semaphore ne rešava:

- queue management,
- retry logiku,
- prioritizaciju poslova.

---

# Best Practices

✅ Kombinuj Semaphore sa Worker Pool kada imaš veliki broj poslova.

✅ Definiši globalni redosled Acquire operacija.

✅ Koristi `context.Context` za svaki Acquire koji može čekati.

✅ Dokumentuj šta svaki Semaphore štiti.

✅ Prati zauzetost permit-a u produkciji.

---

# Mentalni model

Nemoj razmišljati:

```text
Imam više ograničenja

↓

dodam lock
```

Razmišljaj:

```text
Svaki resurs

↓

ima svoju granicu

↓

kontrolisan pristup
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kombinovanje Semaphore i Worker Pool

✅ multi-resource ograničavanje

✅ hijerarhijski Semaphore

✅ kako nastaje deadlock

✅ važnost redosleda Acquire/Release

✅ production dizajn kompleksnih sistema

---

# Semaphore Pattern — Production Guidelines i završni rezime

---

# 📚 Sadržaj

- Kada koristiti Semaphore
- Semaphore vs Worker Pool vs Rate Limiter
- Production checklist
- Najčešće greške
- Senior perspektiva
- Završni rezime

---

# Kada koristiti Semaphore?

Koristi Semaphore kada postoji:

- ograničen resurs,
- maksimalan broj paralelnih operacija,
- potreba za kontrolom pristupa.

Primeri:

```text
Database

↓

max 100 paralelnih query-ja
```

---

```text
External API

↓

max 50 aktivnih poziva
```

---

```text
File Processing

↓

max 20 otvorenih fajlova
```

---

# Kada NE koristiti Semaphore?

Nemoj koristiti Semaphore ako ti treba:

- red poslova,
- raspodela zadataka,
- retry mehanizam,
- prioritizacija,
- obrada velikog broja batch poslova.

Za to postoje drugi obrasci:

- Worker Pool,
- Queue sistemi,
- Pipeline.

---

# Semaphore vs Worker Pool

## Semaphore

Pitanje:

> Koliko operacija sme da pristupi resursu?

Model:

```text
Goroutine

↓

Acquire

↓

Resource

↓

Release
```

---

## Worker Pool

Pitanje:

> Koliko poslova treba da izvršavam istovremeno?

Model:

```text
Jobs

↓

Workers

↓

Results
```

---

# Semaphore vs Rate Limiter

## Semaphore

Kontroliše:

```
Concurrency
```

Primer:

```
50 aktivnih HTTP poziva
```

---

## Rate Limiter

Kontroliše:

```
Frequency
```

Primer:

```
1000 zahteva u minuti
```

---

# Kombinovanje obrazaca

U velikim sistemima često imamo:

```text
Incoming Request

↓

Rate Limiter

↓

Worker Pool

↓

Semaphore

↓

Resource
```

Svaki sloj rešava drugi problem.

---

# Production Checklist

## Dizajn

✅ Jasno definisano šta Semaphore štiti.

✅ Kapacitet zasnovan na realnom limitu.

✅ Dokumentovan razlog postojanja.

---

## Implementacija

✅ `Acquire` ima mogućnost cancellation-a.

✅ `Release` se izvršava pomoću `defer`.

✅ Nema mogućnosti curenja permit-a.

---

## Performance

✅ Kapacitet nije nasumičan broj.

✅ Meri se uticaj na throughput.

✅ Prati se vreme čekanja na permit.

---

## Reliability

✅ Postoji timeout.

✅ Postoji graceful shutdown.

✅ Postoji monitoring.

---

# Najčešće greške Junior programera

## 1. "Više Goroutines = više brzine"

Ne.

Više konkurentnosti može dovesti do:

- contention-a,
- preopterećenja resursa,
- veće latencije.

---

## 2. Zaboravljen Release

Primer:

```go
sem <- struct{}{}

doWork()
```

Ako `doWork()` izađe neočekivano:

permit ostaje zauzet.

---

## 3. Pogrešan kapacitet

Primer:

API limit:

```
20
```

Semaphore:

```
1000
```

Nema stvarne zaštite.

---

# Najčešće greške Medior programera

## 1. Dodavanje Semaphore-a svuda

Semaphore nije univerzalni lek.

Previše ograničenja može:

- smanjiti throughput,
- komplikovati sistem.

---

## 2. Ignorisanje reda čekanja

Ako svi čekaju permit:

treba razumeti:

- koliko dugo čekaju,
- zašto čekaju,
- da li postoji bottleneck.

---

## 3. Loša koordinacija više Semaphore-a

Više resursa zahteva:

- definisan redosled,
- pažljiv dizajn.

---

# Kako razmišlja Senior Go programer?

Ne pita:

> "Koliko Goroutines mogu da pokrenem?"

Pita:

- Koji resurs je ograničen?
- Koja je maksimalna bezbedna konkurentnost?
- Gde treba backpressure?
- Koliko čekanje je prihvatljivo?
- Kako sistem reaguje kada resurs nije dostupan?

---

# Mentalni model

Zapamti:

```text
Semaphore

=

Kontrola pristupa
```

---

```text
Worker Pool

=

Kontrola izvršavanja poslova
```

---

```text
Rate Limiter

=

Kontrola brzine
```

---

Zajedno:

```text
Stabilan sistem
```

---

# Ceo put kroz Modul #4.7

```text
Semaphore Basics

↓

Channel Implementation

↓

Context Cancellation

↓

Rate Limiting

↓

Resource Protection

↓

Advanced Patterns

↓

Production Design
```

---

# Šta nosiš iz ovog modula?

Treba da možeš da objasniš:

- šta je Semaphore Pattern,
- kako implementirati Semaphore u Go-u,
- razliku između Semaphore i Worker Pool,
- razliku između Semaphore i Rate Limiter-a,
- kako koristiti `context.Context`,
- kako zaštititi ograničene resurse,
- kako kombinovati više Semaphore-a,
- kako izbeći deadlock.

---

# 📋 Rezime Modula #4.7

U ovom modulu naučili smo:

✅ šta je Semaphore Pattern

✅ Counting Semaphore

✅ implementaciju pomoću kanala

✅ Semaphore sa `context.Context`

✅ timeout i cancellation

✅ Weighted Semaphore

✅ Rate Limiting koncepte

✅ zaštitu resursa

✅ kombinovanje sa Worker Pool-om

✅ napredne produkcione obrasce

---

### ➡️ Sledeća lekcija **[**Concurrency Patterns**](08-concurrency-patterns.md)**

Obuhvatiće:

- kada koristiti koji pattern,
- Pipeline + Worker Pool + Semaphore kombinacije,
- realne arhitekture,
- poređenje obrazaca,
- donošenje dizajnerskih odluka u Go sistemima.