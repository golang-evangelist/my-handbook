# Concurrency Patterns — Uvod i pregled

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/08-concurrency-patterns.md`

---

# 📚 Sadržaj

- Šta su concurrency patterns
- Zašto postoje obrasci
- Pregled glavnih Go pattern-a
- Kako birati pattern
- Problem koji svaki pattern rešava
- Mentalni model

---

# Uvod

Concurrency u Go-u nije samo:

```go
go function()
```

Pravi izazov je:

- koordinacija,
- komunikacija,
- ograničavanje,
- otkazivanje,
- upravljanje resursima.

Concurrency patterns predstavljaju proverene načine rešavanja tih problema.

---

# Zašto koristiti obrasce?

Bez obrazaca kod često izgleda ovako:

```text
Goroutine

↓

Channel

↓

Mutex

↓

WaitGroup

↓

još jedna Goroutine

↓

još jedan Channel
```

Problem:

- teško razumeti,
- teško testirati,
- teško održavati.

---

Pattern daje strukturu:

```text
Problem

↓

Poznato rešenje

↓

Predvidiv dizajn
```

---

# Glavni Go Concurrency Patterns

U ovom modulu fokus:

```
1. Worker Pool

2. Pipeline

3. Fan-Out / Fan-In

4. Semaphore

5. Rate Limiter

6. Context Cancellation

7. Generator Pattern

8. Pub/Sub Pattern

9. Actor-like Pattern
```

---

# 1. Worker Pool

## Problem

Imamo:

```
1 000 000 poslova
```

Ne želimo:

```
1 000 000 Goroutines
```

---

## Rešenje

```text
Jobs

↓

Workers

↓

Results
```

---

## Koristi se za:

- batch obradu,
- obradu fajlova,
- background poslove,
- queue processing.

---

# 2. Pipeline

## Problem

Obrada ima više koraka.

Primer:

```text
Read

↓

Parse

↓

Validate

↓

Store
```

---

## Rešenje

```text
Stage 1

↓

Stage 2

↓

Stage 3
```

---

## Koristi se za:

- ETL,
- stream processing,
- data transformacije.

---

# 3. Fan-Out / Fan-In

## Problem

Jedna faza je spora.

Primer:

```text
Image Resize
```

---

## Rešenje

Više worker-a:

```text
Input

↓

Worker 1
Worker 2
Worker 3

↓

Output
```

---

## Koristi se za:

- paralelizaciju,
- povećanje throughput-a.

---

# 4. Semaphore

## Problem

Resurs ima limit.

Primer:

```
Database

max 100 aktivnih query-ja
```

---

## Rešenje

```text
Acquire

↓

Resource

↓

Release
```

---

## Koristi se za:

- API limite,
- baze,
- fajlove,
- memoriju.

---

# 5. Rate Limiter

## Problem

Sistem ima ograničenje brzine.

Primer:

```
1000 requests/min
```

---

## Rešenje

```text
Request

↓

Limiter

↓

Allow / Wait
```

---

## Koristi se za:

- API zaštitu,
- throttling,
- fairness.

---

# 6. Context Cancellation

## Problem

Kako zaustaviti ceo sistem?

---

## Rešenje

```text
Context

↓

Cancellation Signal

↓

Sve Goroutines Exit
```

---

## Koristi se za:

- timeout,
- graceful shutdown,
- request lifecycle.

---

# 7. Generator Pattern

## Problem

Kako kreirati stream podataka?

---

Primer:

```text
Generate

↓

Numbers

↓

Consumer
```

---

Kod:

```go
func generator() <-chan int
```

---

# 8. Pub/Sub Pattern

## Problem

Jedan događaj ima više potrošača.

---

Model:

```text
Event

↓

Subscriber 1

Subscriber 2

Subscriber 3
```

---

Koristi se za:

- event-driven sisteme,
- message brokere.

---

# 9. Actor-like Pattern

## Problem

Deljeno stanje izaziva probleme.

---

Rešenje:

```text
Actor

↓

Own State

↓

Messages
```

---

Umesto:

```text
Shared Memory
```

koristi:

```text
Message Passing
```

---

# Kako birati pattern?

Postavi pitanje:

---

## Imam mnogo poslova?

Koristi:

```
Worker Pool
```

---

## Imam više faza?

Koristi:

```
Pipeline
```

---

## Jedna faza je spora?

Koristi:

```
Fan-Out
```

---

## Imam ograničen resurs?

Koristi:

```
Semaphore
```

---

## Imam vremensko ograničenje?

Koristi:

```
Context
```

---

## Imam limit brzine?

Koristi:

```
Rate Limiter
```

---

# Pattern nije cilj

Loš pristup:

> "Moram koristiti Pipeline jer je napredniji."

Dobar pristup:

> "Koji problem pokušavam da rešim?"

Pattern je alat.

---

# Mentalni model

Nemoj razmišljati:

```text
Concurrency

=

više Goroutines
```

Razmišljaj:

```text
Concurrency

=

organizovana koordinacija
```

---

# Production pogled

Realni sistem često izgleda ovako:

```text
HTTP Request

↓

Rate Limiter

↓

Worker Pool

↓

Pipeline

↓

Fan-Out

↓

Semaphore

↓

Database
```

Svaki deo rešava svoj problem.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su concurrency patterns

✅ zašto su važni

✅ pregled glavnih Go obrazaca

✅ koji problem svaki obrazac rešava

✅ kako birati odgovarajući pattern

---

# Concurrency Patterns — Poređenje i izbor obrasca

---

# 📚 Sadržaj

- Worker Pool vs Pipeline
- Pipeline vs Fan-Out/Fan-In
- Worker Pool vs Semaphore
- Semaphore vs Rate Limiter
- Decision Matrix
- Dizajnerska pitanja

---

# Worker Pool vs Pipeline

Ovo je najčešća dilema.

---

# Worker Pool

Problem:

> Imam mnogo nezavisnih poslova.

Primer:

```
10000 slika

↓

obradi svaku
```

Svaka slika je nezavisna.

---

Model:

```text
Jobs

↓

Workers

↓

Results
```

---

Primer:

```go
for job := range jobs {

	process(job)

}
```

---

# Pipeline

Problem:

> Jedan posao prolazi kroz više faza.

Primer:

```
CSV file

↓

Parse

↓

Validate

↓

Transform

↓

Save
```

---

Model:

```text
Stage 1

↓

Stage 2

↓

Stage 3
```

---

# Ključna razlika

Worker Pool:

```
mnogo istih poslova
```

---

Pipeline:

```
jedan tok sa više koraka
```

---

# Primer poređenja

## Email sistem

Imamo:

```
1 milion email poruka
```

---

Ako samo šaljemo:

```text
Queue

↓

Workers

↓

SMTP
```

Worker Pool.

---

Ako imamo:

```text
Load Template

↓

Personalize

↓

Validate

↓

Send
```

Pipeline.

---

# Pipeline + Worker Pool

U praksi:

```text
Read

↓

Parser Workers

↓

Validation Workers

↓

Storage Workers
```

Kombinacija oba.

---

# Pipeline vs Fan-Out/Fan-In

Fan-Out/Fan-In nije samostalan tok.

On rešava problem jedne faze.

---

Pipeline:

```text
A

↓

B

↓

C
```

---

Fan-Out:

```text
A

↓

B1
B2
B3

↓

C
```

---

Primer:

```text
Image Pipeline

↓

Resize Stage

↓

10 Resize Workers

↓

Upload Stage
```

Pipeline + Fan-Out.

---

# Worker Pool vs Semaphore

Ovo je veoma česta greška.

---

# Worker Pool

Kontroliše:

```
broj izvršilaca
```

---

Primer:

```text
20 worker-a
```

---

# Semaphore

Kontroliše:

```
pristup resursu
```

---

Primer:

```text
max 5 DB query-ja
```

---

# Primer

Imamo:

```
1000 korisničkih zahteva
```

Svaki radi:

```
HTTP API

↓

Database
```

---

Worker Pool:

```text
20 request handler worker-a
```

---

Semaphore:

```text
Database limit = 50
```

---

Mogu zajedno:

```text
Requests

↓

Worker Pool

↓

Semaphore

↓

Database
```

---

# Semaphore vs Rate Limiter

Ovo se često meša.

---

# Semaphore

Pita:

> Koliko trenutno radi?

Primer:

```
50 aktivnih zahteva
```

---

# Rate Limiter

Pita:

> Koliko često dozvoljavam?

Primer:

```
1000 zahteva/min
```

---

# Primer

API:

Ograničenja:

```
max 100 aktivnih

max 1000/min
```

Rešenje:

```text
Rate Limiter

↓

Semaphore

↓

API
```

---

# Decision Matrix

| Problem | Pattern |
|---|---|
| Mnogo nezavisnih poslova | Worker Pool |
| Više faza obrade | Pipeline |
| Jedna faza je spora | Fan-Out |
| Spajanje rezultata | Fan-In |
| Ograničen resurs | Semaphore |
| Limit brzine | Rate Limiter |
| Timeout/cancel | Context |
| Event distribucija | Pub/Sub |
| Izolovano stanje | Actor-like |

---

# Dizajnerska pitanja

Pre implementacije pitaj:

---

## 1. Da li poslovi zavise jedan od drugog?

Ako:

```
NE
```

verovatno:

```
Worker Pool
```

---

Ako:

```
DA
```

verovatno:

```
Pipeline
```

---

# 2. Da li postoji usko grlo?

Ako:

```
jedna faza spora
```

dodaj:

```
Fan-Out
```

---

# 3. Da li resurs ima limit?

Ako:

```
Database/API/File
```

dodaj:

```
Semaphore
```

---

# 4. Da li postoji vremensko ograničenje?

Ako:

```
request lifecycle
```

koristi:

```
Context
```

---

# 5. Da li postoji poslovni limit?

Ako:

```
1000 događaja/min
```

koristi:

```
Rate Limiter
```

---

# Loš dizajn

Primer:

```text
10000 Goroutines

↓

Mutex

↓

Database
```

Problem:

- nema kontrole,
- nema backpressure,
- nema zaštite resursa.

---

# Bolji dizajn

```text
Requests

↓

Rate Limiter

↓

Worker Pool

↓

Semaphore

↓

Database
```

---

# Senior perspektiva

Junior pita:

> Kako da napravim više paralelizma?

Senior pita:

> Gde treba ograničiti paralelizam?

---

Junior optimizuje:

```
brzinu
```

Senior optimizuje:

```
stabilnost
```

---

# Mentalni model

Zapamti:

```text
Worker Pool

=

koliko izvršava
```

---

```text
Pipeline

=

kako prolazi
```

---

```text
Semaphore

=

koliko sme
```

---

```text
Rate Limiter

=

koliko često
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ razliku između Worker Pool i Pipeline

✅ ulogu Fan-Out/Fan-In

✅ razliku između Semaphore i Worker Pool

✅ razliku između Semaphore i Rate Limiter

✅ kako izabrati pravi pattern

---

# Concurrency Patterns — Realne arhitekture

---

# 📚 Sadržaj

- HTTP servis arhitektura
- Background Worker sistem
- Data Processing sistem
- Kombinovanje pattern-a
- Kontrola opterećenja
- Production dizajn

---

# 1. HTTP servis primer

Zamisli REST API servis.

Dolaze zahtevi:

```text
Client

↓

HTTP Server

↓

Business Logic

↓

Database
```

---

# Problem

Veliki broj korisnika:

```
10000 request-a
```

može preopteretiti:

- CPU,
- bazu,
- eksterni API.

---

# Loš dizajn

```text
Request

↓

Goroutine

↓

Database
```

---

Problem:

Svaki zahtev odmah koristi resurse.

Nema kontrole.

---

# Production dizajn

```text
HTTP Request

↓

Rate Limiter

↓

Worker Pool

↓

Business Pipeline

↓

Semaphore

↓

Database
```

---

# Slojevi

## Rate Limiter

Štiti sistem od prevelikog broja zahteva.

Primer:

```
1000 request/sec
```

---

## Worker Pool

Kontroliše:

```
koliko zahteva obrađujemo paralelno
```

---

## Pipeline

Organizuje:

```
Validate

↓

Process

↓

Persist
```

---

## Semaphore

Štiti:

```
Database

↓

max 100 aktivnih query-ja
```

---

# Tok jednog request-a

```text
Request

↓

Rate Limiter

↓

Worker

↓

Validate

↓

Acquire DB Permit

↓

Query

↓

Release

↓

Response
```

---

# 2. Background Worker sistem

Primer:

Email servis.

Imamo:

```
1 000 000 email poslova
```

---

# Problem

Ne možemo:

```text
1 000 000 Goroutines
```

---

# Rešenje

Worker Pool:

```text
Queue

↓

Worker 1
Worker 2
Worker 3
...

↓

SMTP
```

---

# Dodavanje Pipeline-a

Slanje email-a ima faze:

```text
Load User

↓

Generate Template

↓

Validate

↓

Send
```

---

Arhitektura:

```text
Queue

↓

Pipeline

↓

Worker Pool

↓

SMTP
```

---

# Dodavanje Semaphore-a

SMTP ima limit:

```
100 aktivnih konekcija
```

Dodajemo:

```text
Worker

↓

Semaphore(100)

↓

SMTP
```

---

# Kompletan tok

```text
Job Queue

↓

Workers

↓

Template Stage

↓

Validation Stage

↓

Semaphore

↓

SMTP
```

---

# 3. Data Processing sistem

Primer:

Obrada velikog CSV fajla.

---

# Zahtev

Ulaz:

```
10GB CSV
```

Proces:

```
Read

↓

Parse

↓

Validate

↓

Transform

↓

Store
```

---

# Pipeline dizajn

```text
Reader

↓

Parser

↓

Validator

↓

Transformer

↓

Database
```

---

# Problem

Parser je spor.

Rešenje:

Fan-Out:

```text
Parser

↓

Parser Worker 1

Parser Worker 2

Parser Worker 3

↓

Validator
```

---

# Problem

Database ima limit.

Rešenje:

Semaphore:

```text
Transformer

↓

Semaphore(50)

↓

Database
```

---

# Kompletna arhitektura

```text
Reader

↓

Parser Pool

↓

Validator

↓

Transformer

↓

Semaphore

↓

Database
```

---

# 4. Microservice komunikacija

Primer:

Servis A poziva:

- Payment API,
- Inventory API,
- Notification API.

---

# Problem

Eksterni servisi imaju limite.

---

# Rešenje

Za svaki servis:

```text
Request

↓

Rate Limiter

↓

Semaphore

↓

External API
```

---

# Primer

Payment API:

```
50 req/sec

20 concurrent
```

---

Implementacija:

```text
Rate Limiter(50)

↓

Semaphore(20)

↓

Payment API
```

---

# 5. Graceful Shutdown

Svi ozbiljni sistemi moraju znati da se ugase.

---

Koristi se:

```text
Context

↓

Cancellation

↓

Workers stop

↓

Channels close

↓

Shutdown
```

---

# Primer

```text
SIGTERM

↓

cancel()

↓

Stop accepting jobs

↓

Wait workers

↓

Exit
```

---

# Najvažniji princip

Svaki pattern rešava različit problem.

---

Ne:

```text
Dodaj pattern

↓

biće bolje
```

---

Već:

```text
Problem

↓

Odgovarajući pattern
```

---

# Primer pogrešne kombinacije

Loše:

```text
Worker Pool

+

Pipeline

+

Semaphore

+

Rate Limiter

+

Mutex

+

Channel
```

bez jasnog razloga.

---

Rezultat:

- kompleksnost,
- teško održavanje,
- teško debagovanje.

---

# Production checklist

Pre dizajna pitaj:

## Opterećenje

- Koliko zahteva?
- Koliko traje obrada?
- Gde nastaje bottleneck?

---

## Resursi

- Koji resurs je ograničen?
- Koji je maksimalni kapacitet?

---

## Lifecycle

- Kako se sistem gasi?
- Kako se prekidaju poslovi?

---

## Observability

- Kako merimo?
- Koji metric-i postoje?

---

# Mentalni model

Ne dizajniraj:

```text
više Goroutines
```

Dizajniraj:

```text
kontrolisan tok posla
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako izgleda HTTP concurrency arhitektura

✅ kako dizajnirati background worker sistem

✅ kako koristiti Pipeline za data processing

✅ kako kombinovati Worker Pool, Semaphore i Rate Limiter

✅ kako dizajnirati production concurrency sistem

---

# Generator Pattern — Stream Production

---

# 📚 Sadržaj

- Šta je Generator Pattern
- Problem koji rešava
- Generator pomoću kanala
- Lazy production
- Generator + Pipeline
- Generator lifecycle
- Production primeri

---

# Uvod

Klasičan način generisanja podataka:

```go
numbers := []int{
	1,2,3,4,5,
}
```

Problem:

Ako imaš:

```
10 miliona elemenata
```

sve mora biti u memoriji.

---

Generator radi drugačije:

```text
Podatak

↓

kreiraj kada je potreban
```

---

# Generator koncept

Generator je funkcija koja:

- kreira Goroutine,
- proizvodi vrednosti,
- šalje ih kroz kanal.

Model:

```text
Generator

↓

Channel

↓

Consumer
```

---

# Osnovni primer

```go
func generator() <-chan int {

	out := make(chan int)

	go func() {

		for i := 0; i < 10; i++ {

			out <- i
		}

		close(out)

	}()

	return out
}
```

---

Korišćenje:

```go
for value := range generator() {

	fmt.Println(value)

}
```

---

# Šta se dešava?

Poziv:

```go
generator()
```

vraća kanal.

U pozadini:

```text
Goroutine

↓

kreira broj

↓

šalje kanal
```

Consumer:

```text
uzima kada želi
```

---

# Lazy Production

Generator ne pravi sve unapred.

---

Lista:

```text
Create all

↓

Store all

↓

Process
```

---

Generator:

```text
Create

↓

Send

↓

Process

↓

Create next
```

---

Prednosti:

- manja memorijska potrošnja,
- moguć beskonačan stream,
- prirodna obrada velikih podataka.

---

# Beskonačan Generator

Primer:

```go
func infiniteCounter() <-chan int {

	out := make(chan int)

	go func() {

		i := 0

		for {

			out <- i

			i++
		}

	}()

	return out
}
```

---

Tok:

```text
0

1

2

3

4

...
```

---

Problem:

Kako ga zaustaviti?

---

# Generator + Context

Production verzija:

```go
func generator(
	ctx context.Context,
) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for i := 0; ; i++ {

			select {

			case out <- i:

			case <-ctx.Done():
				return

			}
		}

	}()

	return out
}
```

---

Sada imamo:

```text
Context

↓

Generator

↓

Controlled shutdown
```

---

# Generator kao Pipeline početak

Veoma čest obrazac:

```text
Generator

↓

Stage 1

↓

Stage 2

↓

Consumer
```

---

Primer:

```text
Generate IDs

↓

Validate

↓

Transform

↓

Save
```

---

# Kombinacija sa Fan-Out

Generator:

```text
10 miliona događaja
```

---

Parser:

```text
Worker 1
Worker 2
Worker 3
```

---

Arhitektura:

```text
Generator

↓

Fan-Out Workers

↓

Fan-In

↓

Consumer
```

---

# Kombinacija sa Semaphore

Primer:

Generator proizvodi:

```
URL adrese
```

Worker-i rade:

```
HTTP request
```

API limit:

```
100 concurrent
```

---

Dizajn:

```text
Generator URLs

↓

Workers

↓

Semaphore(100)

↓

HTTP Client
```

---

# Generator vs Queue

Često pitanje:

Da li je generator isto što i queue?

Nije.

---

# Generator

Proizvodi podatke.

Primer:

```
Brojevi

Fajlovi

Događaji
```

---

# Queue

Čuva poslove.

Primer:

```
Redis

Kafka

RabbitMQ
```

---

Generator:

```text
Source
```

Queue:

```text
Storage + Delivery
```

---

# Production primer 1

CSV obrada:

```text
File Reader

↓

Generator Rows

↓

Parser

↓

Validator

↓

Database
```

---

Ne učitavaš:

```
10GB CSV
```

u memoriju.

Čitaš:

```
jedan red po jedan
```

---

# Production primer 2

Log processing:

```text
Log Stream

↓

Generator Events

↓

Filter

↓

Aggregate

↓

Store
```

---

# Najčešće greške

## Greška #1

Generator bez cancellation-a.

Rezultat:

Goroutine leak.

---

## Greška #2

Ne zatvaranje kanala.

Consumer:

```go
range ch
```

nikada ne završava.

---

## Greška #3

Prebrza produkcija.

Generator šalje brže nego što consumer obrađuje.

Rešenje:

- buffering,
- backpressure,
- rate limiting.

---

# Best Practices

✅ Generator treba da poseduje svoj izlazni kanal.

✅ Zatvori kanal kada više nema podataka.

✅ Koristi `context.Context` za duge stream-ove.

✅ Kombinuj sa Pipeline pattern-om.

✅ Obrati pažnju na backpressure.

---

# Mentalni model

Nemoj razmišljati:

```text
Napravim listu

↓

obradim
```

Razmišljaj:

```text
Proizvedi

↓

Pošalji

↓

Obradi

↓

Nastavi
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Generator Pattern

✅ kako implementirati generator pomoću kanala

✅ koncept lazy production

✅ generator sa `context.Context`

✅ povezivanje generatora sa Pipeline-om

✅ production primene generatora

---

# Pub/Sub Pattern — Event-Driven Communication

---

# 📚 Sadržaj

- Šta je Pub/Sub Pattern
- Problem koji rešava
- Publisher
- Subscriber
- Broadcast komunikacija
- Channel fan-out vs Pub/Sub
- In-memory implementacija
- Production sistemi

---

# Uvod

Zamisli korisničku registraciju:

```text
User Created Event
```

Nakon toga treba:

- poslati email,
- kreirati audit zapis,
- poslati analitiku,
- obavestiti druge servise.

---

Loš dizajn:

```text
Register User

↓

Send Email

↓

Write Audit

↓

Send Analytics
```

Problem:

Glavna operacija zavisi od svega.

---

# Pub/Sub rešenje

Umesto direktnog pozivanja:

```text
User Service

↓

Event
```

---

Subscriber-i slušaju:

```text
              Email Service

                    ↑

User Created Event

                    ↓

              Audit Service


                    ↓

              Analytics Service
```

---

# Osnovni koncept

Postoje dve strane.

---

# Publisher

Proizvodi događaje.

Primer:

```text
UserCreated

OrderPaid

PaymentFailed
```

---

# Subscriber

Registruje interesovanje.

Primer:

```text
Email Subscriber

Audit Subscriber
```

---

# Model

```text
Publisher

↓

Topic

↓

Subscribers
```

---

# Topic

Topic predstavlja kategoriju događaja.

Primer:

```text
users.created
```

ili:

```text
orders.completed
```

---

Subscriber kaže:

> "Želim sve događaje sa ovog topic-a."

---

# In-memory Pub/Sub u Go-u

Jednostavna struktura:

```go
type Broker struct {

	subscribers map[string][]chan Event

}
```

---

Publisher:

```go
func Publish(
	topic string,
	event Event,
)
```

---

Subscriber:

```go
func Subscribe(
	topic string,
) <-chan Event
```

---

# Primer

Kreiranje subscriber-a:

```go
emailCh :=
	broker.Subscribe(
		"user.created",
	)
```

---

Objavljivanje:

```go
broker.Publish(
	"user.created",
	userEvent,
)
```

---

Email servis dobija:

```go
event := <-emailCh
```

---

# Broadcast komunikacija

Kod Pub/Sub-a:

jedan događaj ide ka:

```
više subscriber-a
```

---

Primer:

```text
Event

↓

Subscriber 1

Subscriber 2

Subscriber 3
```

---

Svaki dobija svoju kopiju događaja.

---

# Pub/Sub vs Channel Fan-Out

Ovo se često meša.

---

# Channel Fan-Out

Primer:

```text
Jobs Channel

↓

Worker 1
Worker 2
Worker 3
```

Jedan posao obrađuje samo jedan worker.

---

Model:

```
Load balancing
```

---

# Pub/Sub

Primer:

```text
Event

↓

Subscriber 1

Subscriber 2

Subscriber 3
```

Svi dobijaju događaj.

---

Model:

```
Broadcast
```

---

# Ključna razlika

Fan-Out:

```
jedan posao

↓

jedan izvršilac
```

---

Pub/Sub:

```
jedan događaj

↓

više potrošača
```

---

# Sinhroni vs Asinhroni Subscriber-i

## Sinhroni

Publisher čeka:

```text
Publish

↓

Subscriber obradi

↓

Return
```

---

Problem:

Spor subscriber usporava sve.

---

## Asinhroni

Publisher samo šalje:

```text
Publish

↓

Return
```

Subscriber radi nezavisno.

---

Production sistemi uglavnom koriste asinhroni model.

---

# Backpressure problem

Šta ako subscriber ne stiže?

Primer:

```text
Publisher

1000 events/sec
```

---

Subscriber:

```
100 events/sec
```

---

Dolazi do:

```
Queue growth
```

---

Rešenja:

- buffered channels,
- dropping events,
- retry,
- persistent broker,
- dead letter queue.

---

# Context i Shutdown

Subscriber mora da zna kada da stane.

Primer:

```go
select {

case event := <-events:

	process(event)

case <-ctx.Done():

	return

}
```

---

Tok:

```text
Shutdown

↓

Cancel Context

↓

Subscribers Exit
```

---

# Production sistemi

In-memory Pub/Sub:

- jednostavan,
- brz,
- radi unutar jednog procesa.

---

Za distribuirane sisteme koriste se:

- :contentReference[oaicite:0]{index=0}
- :contentReference[oaicite:1]{index=1}
- :contentReference[oaicite:2]{index=2}

---

Primer arhitekture:

```text
Service A

↓

Event Broker

↓

Service B

Service C

Service D
```

---

# Production primer

E-commerce:

Događaj:

```text
OrderCreated
```

Subscriber-i:

```text
Payment Service

↓

Inventory Service

↓

Email Service

↓

Analytics Service
```

---

Novi subscriber može biti dodat bez menjanja Order servisa.

---

# Najčešće greške

## Greška #1

Koristiti Pub/Sub za jednostavan poziv.

Ako imaš:

```go
result := service.Call()
```

ne treba event sistem.

---

## Greška #2

Ignorisati neuspehe subscriber-a.

Šta ako:

```
Email Service padne?
```

Potrebna je strategija:

- retry,
- queue,
- dead letter.

---

## Greška #3

Previše događaja.

Loš dizajn:

```
Event za svaku sitnu promenu
```

Rezultat:

kompleksan sistem.

---

# Best Practices

✅ Jasno definiši event modele.

✅ Subscriber treba da bude nezavisan.

✅ Koristi async obradu gde je moguće.

✅ Planiraj greške i retry.

✅ Razmišljaj o backpressure-u.

---

# Mentalni model

Nemoj razmišljati:

```text
Pozovi sledeću funkciju
```

Razmišljaj:

```text
Objavi činjenicu

↓

Ko želi neka reaguje
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Pub/Sub Pattern

✅ razliku između Publisher i Subscriber

✅ broadcast komunikaciju

✅ razliku između Fan-Out i Pub/Sub

✅ asinhronu event obradu

✅ probleme backpressure-a

✅ production primene

---

# Actor-like Pattern — Message Passing i izolacija stanja

---

# 📚 Sadržaj

- Šta je Actor Pattern
- Problem deljenog stanja
- Actor model
- Actor-like implementacija u Go-u
- Message Passing
- Actor vs Mutex
- Go CSP pristup
- Završni pregled concurrency pattern-a

---

# Uvod

Jedan od najčešćih problema u concurrency programiranju:

```text
Više Goroutines

↓

Isto stanje

↓

Race Condition
```

---

Primer:

```go
var counter int
```

Više Goroutines:

```go
counter++
```

Problem:

- data race,
- potreba za lock-ovima,
- kompleksna sinhronizacija.

---

# Klasično rešenje

Koristimo:

```go
sync.Mutex
```

Primer:

```go
mu.Lock()

counter++

mu.Unlock()
```

---

Radi.

Ali kod kompleksnijeg sistema:

- mnogo lock-ova,
- moguć deadlock,
- teško praćenje.

---

# Actor Model

Actor je entitet koji ima:

1. svoje stanje,
2. mailbox,
3. ponašanje.

---

Model:

```text
        Message

           ↓

      +---------+
      | Actor   |
      |
      | State   |
      |
      +---------+
```

---

Actor:

- poseduje svoje stanje,
- jedini ga menja,
- prima poruke.

---

# Ključni princip

Nema:

```text
Shared Memory
```

nego:

```text
Message Passing
```

---

# Actor-like u Go-u

Go nema ugrađeni Actor model.

Ali možemo napraviti isti princip pomoću:

- Goroutine,
- Channel,
- Message type.

---

# Primer

Actor:

```go
type CounterActor struct {

	messages chan int

	value int

}
```

---

Actor loop:

```go
func (a *CounterActor) Start() {

	go func(){

		for msg := range a.messages {

			a.value += msg

		}

	}()

}
```

---

Slanje poruke:

```go
actor.messages <- 1
```

---

Niko direktno ne menja:

```go
actor.value
```

---

# Tok

```text
Goroutine 1

      |
      |
      v

 Message Channel

      |
      |
      v

 Actor Goroutine

      |
      |
      v

 Internal State
```

---

# Prednost

Samo jedna Goroutine pristupa stanju.

Rezultat:

Nema potrebe za:

```go
Mutex
```

---

# Actor vs Mutex

## Mutex model

```text
Shared State

↓

Multiple Goroutines

↓

Lock
```

---

## Actor model

```text
Private State

↓

One Goroutine

↓

Messages
```

---

# Kada koristiti Mutex?

Dobro za:

- jednostavne strukture,
- kratke kritične sekcije,
- lokalno stanje.

Primer:

```go
cache.Lock()

cache.data[key] = value

cache.Unlock()
```

---

# Kada koristiti Actor-like?

Dobro za:

- kompleksno stanje,
- mnogo događaja,
- state machine logiku,
- event processing.

---

Primer:

```text
Game Player

↓

Actor

↓

Position

Health

Inventory
```

---

# Actor-like + Pipeline

Možemo kombinovati.

Primer:

```text
Events

↓

Pipeline

↓

Actors

↓

State Update
```

---

# Actor-like + Pub/Sub

Primer:

```text
Event Broker

↓

Player Actor

↓

Update State
```

---

# Go CSP vs Actor Model

Go filozofija:

> Don't communicate by sharing memory; share memory by communicating.

---

Actor:

```
Actor

↓

Messages
```

---

Go CSP:

```
Goroutine

↓

Channels
```

---

Sličnosti:

- message passing,
- izolacija stanja,
- asinhrona komunikacija.

---

Razlika:

Actor je entitet sa identitetom.

Go channel je mehanizam komunikacije.

---

# Primer produkcione upotrebe

## Trading sistem

Svaki nalog:

```text
Order Actor
```

ima:

- stanje naloga,
- istoriju,
- status.

Poruke:

```text
Create

Cancel

Execute
```

---

## Multiplayer igra

Svaki igrač:

```text
Player Actor
```

prima:

```text
Move

Attack

Chat
```

---

## IoT sistem

Svaki uređaj:

```text
Device Actor
```

obrađuje:

```text
Sensor Data

Commands
```

---

# Najčešće greške

## Greška #1

Kreiranje previše Actor-a.

Primer:

```text
Milion Actor Goroutines
```

bez kontrole.

---

## Greška #2

Veliki mailbox.

Ako poruke dolaze brže nego obrada:

```
Memory growth
```

---

## Greška #3

Korišćenje Actor-a za jednostavne probleme.

Za:

```go
counter++
```

Mutex je često bolji.

---

# Best Practices

✅ Actor treba da poseduje svoje stanje.

✅ Komunikacija ide kroz poruke.

✅ Koristi context za shutdown.

✅ Definiši veličinu mailbox-a.

✅ Prati backpressure.

---

# Završni pregled Modula #4.8

Do sada smo obradili:

---

# Worker Pool

Problem:

```
Mnogo poslova
```

Rešenje:

```
Kontrolisani worker-i
```

---

# Pipeline

Problem:

```
Više faza obrade
```

Rešenje:

```
Stage tok
```

---

# Fan-Out/Fan-In

Problem:

```
Spor stage
```

Rešenje:

```
Paralelizacija
```

---

# Semaphore

Problem:

```
Ograničen resurs
```

Rešenje:

```
Permit kontrola
```

---

# Rate Limiter

Problem:

```
Previše brzo
```

Rešenje:

```
Kontrola frekvencije
```

---

# Generator

Problem:

```
Produkcija stream-a
```

Rešenje:

```
Lazy data source
```

---

# Pub/Sub

Problem:

```
Distribucija događaja
```

Rešenje:

```
Broadcast
```

---

# Actor-like

Problem:

```
Kompleksno stanje
```

Rešenje:

```
Message passing
```

---

# Kako senior bira pattern?

Pita:

```text
Koji problem rešavam?
```

Ne:

```text
Koji pattern želim da koristim?
```

---

# Finalni mentalni model

```text
Worker Pool

koliko izvršava
```

---

```text
Pipeline

kako prolazi
```

---

```text
Fan-Out

kako skaliram
```

---

```text
Semaphore

šta ograničavam
```

---

```text
Rate Limiter

koliko brzo
```

---

```text
Generator

odakle dolazi
```

---

```text
Pub/Sub

ko reaguje
```

---

```text
Actor

ko poseduje stanje
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Actor-like Pattern

✅ izolaciju stanja

✅ message passing pristup

✅ razliku između Actor-a i Mutex-a

✅ odnos Go CSP modela i Actor modela

✅ kompletan pregled concurrency pattern-a

---

### ➡️ Sledeća lekcija **[**Concurrency Anti-Patterns**](09-concurrency-anti-patterns.md)**

Obuhvatiće:

- pogrešnu upotrebu Goroutines,
- channel leak,
- goroutine leak,
- deadlock scenarije,
- race condition greške,
- pogrešan dizajn concurrency sistema.