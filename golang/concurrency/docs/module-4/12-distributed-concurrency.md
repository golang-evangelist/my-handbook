# Distributed Concurrency & Go Systems

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/12-distributed-concurrency.md`

---

# 📚 Sadržaj

- Šta je Distributed Concurrency
- Lokalni vs Distributed Concurrency
- Proces kao concurrency granica
- Distributed Worker model
- Messaging kao komunikacija
- Event-driven arhitektura
- Problemi distribuiranih sistema

---

# Uvod

U jednom procesu:

```go
go worker()
```

runtime kontroliše:

- Goroutines,
- Scheduler,
- Memory,
- Channels.

---

U distribuiranom sistemu:

```text
Machine A

+

Machine B

+

Machine C
```

nema zajedničkog:

- memory prostora,
- scheduler-a,
- channel-a.

---

# Lokalni Concurrency Model

Primer:

```go
jobs <- job
```

---

Channel postoji u:

```
jednom procesu
```

---

Komunikacija:

```text
Goroutine

↓

Channel

↓

Goroutine
```

---

# Distributed Concurrency Model

Sada imamo:

```text
Process A

↓

Network

↓

Process B
```

---

Komunikacija više nije:

```go
channel
```

nego:

```
message
```

---

# Osnovna razlika

| Lokalni sistem | Distributed sistem |
|-|-|
| Goroutines | Processes |
| Channels | Messaging |
| Shared memory | Network |
| Mutex | Distributed coordination |
| Nanosekunde | Milisekunde |
| Jedan failure domain | Više failure domain-a |

---

# Proces kao Concurrency Granica

U Go:

```text
Process

├── Goroutine
├── Goroutine
└── Goroutine
```

---

Jedan proces ima:

- svoj heap,
- svoj GC,
- svoj scheduler.

---

Drugi proces ne može direktno:

```go
access(otherProcess.memory)
```

---

Zato koristimo:

- HTTP,
- gRPC,
- message broker,
- queue sisteme.

---

# Distributed Worker Model

Primer:

Imamo:

```
1 000 000 Jobs
```

---

Ne želimo:

```
jedan server
```

---

Već:

```text
Worker Node A

Worker Node B

Worker Node C

Worker Node D
```

---

Model:

```text
              Queue

                ↓

    +-----------+-----------+

    |           |           |

 Worker A   Worker B   Worker C

    |           |           |

    +-----------+-----------+

                ↓

             Results
```

---

# Zašto Distributed Workers?

Prednosti:

## Skaliranje

Dodamo novi worker:

```
Worker D
```

---

## Izolacija

Jedan worker padne:

```
ostali rade
```

---

## Veći throughput

Više mašina:

```
više obrade
```

---

# Messaging kao Osnovni Primitiv

Kada nema shared memory:

koristimo:

```
Messages
```

---

Poruka:

```go
type Message struct {

	ID string

	Type string

	Payload []byte

}
```

---

Tok:

```text
Producer

↓

Message Broker

↓

Consumer
```

---

# Message Broker

Broker predstavlja:

```
posrednika između servisa
```

---

Odgovornosti:

- čuvanje poruka,
- distribucija,
- retry,
- routing.

---

Primer arhitekture:

```text
Order Service

↓

Message Broker

↓

Payment Worker

↓

Email Worker
```

---

# Event-Driven Architecture

Umesto:

```text
Service A poziva Service B
```

koristimo:

```text
Event
```

---

Primer:

Novi order:

```text
OrderCreated
```

---

Servisi reaguju:

```text
Payment Service

Inventory Service

Notification Service
```

---

# Prednosti Event-Driven pristupa

## Loose Coupling

Servisi nisu direktno povezani.

---

## Skaliranje

Svaki consumer zasebno skalira.

---

## Asinhronost

Producer ne čeka consumer.

---

# Novi Problemi

Distributed concurrency uvodi nove probleme.

---

# Problem #1

## Network nije pouzdan

Lokalno:

```
function call

= odmah
```

---

Mreža:

```
request

↓

timeout

↓

unknown state
```

---

# Problem #2

## Partial Failure

Primer:

```text
Service A radi

Service B pao
```

---

Sistem nije:

```
sve radi

ili

sve ne radi
```

---

Postoji:

```
delimičan kvar
```

---

# Problem #3

## Duplicate Messages

Poruka:

```
PaymentRequested
```

---

Može stići:

```
jednom

ili

dva puta
```

---

Zašto?

Retry.

---

Zato treba:

# Idempotency

---

# Problem #4

## Ordering

Primer:

Poruke:

```
Created

↓

Paid

↓

Cancelled
```

---

Ako stignu:

```
Paid

↓

Created
```

problem.

---

Potrebna je:

- ordering strategija,
- versioning,
- sequence number.

---

# Problem #5

## Distributed State

U lokalnom sistemu:

```go
mutex.Lock()
```

---

Distribuirano:

nema globalnog mutex-a.

---

Potrebni su:

- consensus,
- locks,
- coordination sistemi.

---

# Distributed Concurrency Principi

---

## 1. Pretpostavi grešku

Svaka komponenta može:

- pasti,
- kasniti,
- vratiti grešku.

---

## 2. Sve ima timeout

Nikada:

```
čekaj zauvek
```

---

## 3. Poruke nisu garantovano jedinstvene

Implementiraj:

```
idempotency
```

---

## 4. State mora biti dizajniran

Ne pretpostavljati:

```
global memory
```

---

# Mentalni model

Single Process:

```
Goroutines

↓

Channels

↓

Memory
```

---

Distributed:

```
Processes

↓

Network

↓

Messages

↓

Storage
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je distributed concurrency

✅ razliku lokalnog i distribuiranog modela

✅ proces kao granicu

✅ distributed workers

✅ messaging model

✅ event-driven arhitekturu

✅ osnovne probleme distribuiranih sistema

---

# Distributed Concurrency — Message Queues i Worker Systems

---

# 📚 Sadržaj

- Šta je Message Queue
- Producer / Consumer model
- Queue arhitektura
- Distributed Worker System
- Delivery guarantees
- At-most-once
- At-least-once
- Exactly-once problem
- Retry sistemi
- Dead Letter Queue

---

# Šta je Message Queue?

Message Queue je distribuirani mehanizam za prenos poslova između komponenti.

---

Model:

```text
Producer

   |

   ↓

Message Queue

   |

   ↓

Consumer
```

---

Producer i Consumer nisu direktno povezani.

---

Primer:

```text
Order Service

↓

Queue

↓

Email Worker
```

---

Order Service ne mora čekati email.

---

# Producer / Consumer Model

## Producer

Kreira poruke.

Primer:

```go
type Job struct {

	ID string

	Action string

}
```

---

Šalje:

```
Job Created
```

---

## Consumer

Čita poruke.

Radi:

```
Process Job
```

---

Model:

```text
Producer

1000 jobs/sec

↓

Queue

↓

Workers

100 jobs/sec svaki
```

---

# Zašto Queue?

Bez queue:

```text
Service A

↓

Service B
```

---

Problem:

Ako B padne:

```
A ne radi
```

---

Sa queue:

```text
Service A

↓

Queue

↓

Service B
```

---

Ako B padne:

poruke čekaju.

---

# Queue kao Buffer

Queue apsorbuje razliku u brzini.

---

Primer:

Producer:

```
10000 events/sec
```

---

Consumer:

```
2000 events/sec
```

---

Queue:

```
privremeno skladišti
```

---

# Distributed Worker System

Arhitektura:

```text
              Queue

                |

      +---------+---------+

      |         |         |

   Worker1  Worker2  Worker3

      |         |         |

      +---------+---------+

                |

             Database
```

---

Prednosti:

- horizontalno skaliranje,
- fault tolerance,
- kontrola opterećenja.

---

# Worker Lifecycle

Distributed worker:

```text
Start

↓

Connect Queue

↓

Receive Message

↓

Process

↓

Acknowledge

↓

Repeat
```

---

# Acknowledgement (ACK)

Najvažniji koncept.

Pitanje:

```
Kada je poruka završena?
```

---

Opcije:

## Auto ACK

Poruka se odmah označi završena.

---

Problem:

Ako worker padne:

```
posao izgubljen
```

---

## Manual ACK

Worker potvrđuje posle obrade.

---

Model:

```text
Receive

↓

Process

↓

ACK
```

---

Ako pukne:

poruka se vraća.

---

# Delivery Guarantees

Distribuirani sistemi imaju različite garancije.

---

# 1. At-most-once

Znači:

```
0 ili 1 puta
```

---

Poruka može biti izgubljena.

---

Model:

```text
Send

↓

Delete
```

---

Prednost:

- brzo,
- jednostavno.

---

Mana:

- gubitak podataka.

---

Koristi se za:

- metrike,
- telemetry.

---

# 2. At-least-once

Znači:

```
najmanje jednom
```

---

Poruka neće biti izgubljena.

---

Ali:

može biti duplikata.

---

Primer:

```text
Worker obradi

↓

pad pre ACK

↓

Queue ponovi
```

---

Rezultat:

```
isti posao 2 puta
```

---

Najčešći model u praksi.

---

# 3. Exactly-once

Znači:

```
tačno jednom
```

---

Problem:

Veoma teško u distribuiranim sistemima.

---

Primer:

```
DB update

+

ACK queue
```

---

Šta ako:

1. DB update uspe

2. Worker padne

3. ACK nije poslat

---

Queue ponovi.

---

Dobijamo:

```
duplicate update
```

---

# Exactly-once Problem

Distribuirani sistemi ne mogu lako garantovati:

```
jedna operacija

na više sistema

tačno jednom
```

---

Zato se često koristi:

```
At-least-once

+

Idempotency
```

---

# Idempotency

Operacija je idempotentna ako:

ponavljanje daje isti rezultat.

---

Primer:

Loše:

```text
Add 100 EUR
```

Poziv dva puta:

```
+200 EUR
```

---

Bolje:

```text
Set payment status = PAID

transaction_id=123
```

---

Ponovljeno:

rezultat isti.

---

# Retry Sistem

Šta kada worker ne uspe?

---

Ne želimo:

```
fail

↓

stop
```

---

Koristimo:

```
retry
```

---

Model:

```text
Message

↓

Worker

↓

Error

↓

Retry Queue

↓

Worker
```

---

# Retry Policy

Mora imati:

## Maximum attempts

Primer:

```
3 pokušaja
```

---

## Backoff

Čekanje:

```
1s

↓

5s

↓

30s
```

---

## Timeout

Svaki pokušaj ima limit.

---

# Exponential Backoff

Formula:

```
delay = base * 2^attempt
```

---

Primer:

Attempt:

```
0

1

2

3
```

---

Delay:

```
1s

2s

4s

8s
```

---

Prednost:

Smanjuje retry storm.

---

# Dead Letter Queue (DLQ)

Šta sa porukama koje stalno padaju?

---

Ne možemo:

```
retry forever
```

---

Rešenje:

```
Dead Letter Queue
```

---

Model:

```text
Main Queue

↓

Worker

↓

Failure

↓

Retry

↓

Still Failed

↓

DLQ
```

---

DLQ služi za:

- analizu,
- ručni pregled,
- recovery.

---

# Queue Ordering

Neke aplikacije zahtevaju redosled.

---

Primer:

Bank:

```
Deposit

↓

Withdraw

↓

Close Account
```

---

Ako redosled nije očuvan:

problem.

---

Strategije:

- partitioning,
- key-based ordering,
- sequence numbers.

---

# Queue Scaling

Problemi:

## Queue raste

Znači:

```
Consumer spor
```

---

Rešenja:

- više workers,
- optimizacija obrade,
- backpressure.

---

## Worker previše

Znači:

```
DB overload
```

---

Rešenje:

limit concurrency.

---

# Production Worker Checklist

Worker mora imati:

✅ timeout

✅ retry policy

✅ idempotency

✅ ACK strategiju

✅ graceful shutdown

✅ metrics

✅ logging

---

# Mentalni model

Lokalno:

```
Channel

↓

Goroutine
```

---

Distribuirano:

```
Queue

↓

Worker Cluster
```

---

Pouzdan sistem:

```
At-least-once

+

Idempotency

+

Retry

+

DLQ
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Message Queue model

✅ Producer/Consumer arhitekturu

✅ Distributed Worker sistem

✅ ACK mehanizam

✅ delivery guarantees

✅ exactly-once problem

✅ retry sistem

✅ Dead Letter Queue

---

# Distributed Concurrency — Event-Driven Systems

---

# 📚 Sadržaj

- Šta je Event-Driven Architecture
- Command vs Event
- Event model dizajn
- Pub/Sub arhitektura
- Consumer Groups
- Event Ordering
- Event Replay
- CQRS uvod

---

# Šta je Event-Driven Architecture?

Event-driven sistem se zasniva na ideji:

> Servisi reaguju na događaje koji su se desili.

---

Primer:

Kreiranje porudžbine:

```text
Order Created
```

---

Ne kažemo:

```
Pozovi Payment Service

Pozovi Email Service

Pozovi Inventory Service
```

---

Već:

```text
OrderCreated Event

          ↓

   +------+-------+

   |      |       |

Payment Inventory Email

```

---

# Command vs Event

Veoma važna razlika.

---

# Command

Command znači:

```
Uradi nešto.
```

---

Primer:

```text
CreateOrder
```

---

Ima:

- cilj,
- očekivanje izvršavanja.

---

# Event

Event znači:

```
Nešto se već desilo.
```

---

Primer:

```text
OrderCreated
```

---

Nema naredbe.

Samo informacija.

---

# Poređenje

| Command | Event |
|-|-|
| zahtev | činjenica |
| imperativ | prošlo vreme |
| jedan cilj | više zainteresovanih |
| očekuje odgovor | može biti async |

---

# Event Model

Dobar event treba imati:

---

## Event ID

Jedinstveni identifikator.

```go
ID string
```

---

Za:

- deduplication,
- tracing.

---

## Event Type

Primer:

```go
Type string
```

---

Vrednost:

```text
OrderCreated
```

---

## Timestamp

Kada se desilo.

```go
CreatedAt time.Time
```

---

## Aggregate ID

Koji entitet.

Primer:

```go
OrderID
```

---

## Payload

Podaci događaja.

---

Primer:

```go
type Event struct {

	ID string

	Type string

	AggregateID string

	CreatedAt time.Time

	Data any

}
```

---

# Event Schema Dizajn

Event schema mora biti stabilna.

---

Loše:

```json
{
"user": "Mark"
}
```

---

Bolje:

```json
{
"event_id":"123",
"event_type":"UserCreated",
"user_id":"456"
}
```

---

# Zašto?

Jer consumer-i žive nezavisno.

---

Ako promenimo schema:

```
stari consumer

↓

puca
```

---

# Schema Evolution

Event schema se menja.

---

Strategije:

## Additive Changes

Dodavanje polja.

Primer:

```json
{
"name":"Ana",

"email":"a@test.com"
}
```

---

Stari consumer ignoriše novo polje.

---

## Versioning

Primer:

```
UserCreated.v1

UserCreated.v2
```

---

# Pub/Sub Model

Publish / Subscribe:

```text
Publisher

↓

Topic

↓

Subscribers
```

---

Primer:

```text
OrderCreated Topic
```

---

Subscribers:

```
Payment

Inventory

Email
```

---

# Prednosti Pub/Sub

## Loose Coupling

Publisher ne zna ko sluša.

---

## Skaliranje

Dodajemo consumer-e.

---

## Fleksibilnost

Novi servis samo subscribuje event.

---

# Consumer Groups

Kada imamo veliki broj događaja:

jedan consumer nije dovoljan.

---

Primer:

```text
Topic:

1M events
```

---

Imamo:

```
Consumer Group A
```

---

Sa:

```
Worker 1

Worker 2

Worker 3
```

---

Svaki event ide jednom worker-u.

---

Model:

```text
             Topic

               |

        Consumer Group

        /      |      \

       W1      W2      W3
```

---

# Consumer Lifecycle

Consumer mora imati:

```text
Connect

↓

Subscribe

↓

Receive

↓

Process

↓

Commit

↓

Repeat
```

---

# Event Ordering

Pitanje:

```
Da li redosled događaja postoji?
```

---

Primer:

```text
AccountCreated

↓

MoneyDeposited

↓

AccountClosed
```

---

Ako stigne:

```text
AccountClosed

↓

MoneyDeposited
```

problem.

---

# Ordering Strategije

## 1. Partition Key

Svi eventi istog entiteta idu u istu particiju.

---

Primer:

```
account_id=123
```

---

Rezultat:

redosled očuvan.

---

## 2. Sequence Number

Event:

```go
Sequence int
```

---

Consumer proverava:

```
da li je sledeći?
```

---

# Event Replay

Velika prednost event sistema.

---

Event log:

```text
Event 1

Event 2

Event 3
```

---

Možemo ponovo pokrenuti obradu.

---

Primer:

Novi servis:

```
Analytics Service
```

---

Ne mora čekati nove događaje.

Može:

```
replay history
```

---

# Event Sourcing Uvod

Tradicionalno:

Čuvamo stanje:

```text
Account

balance = 500
```

---

Event sourcing:

Čuvamo događaje:

```text
Created

+

Deposit 300

+

Deposit 200
```

---

Stanje se rekonstruiše:

```
Events

↓

State
```

---

# CQRS Uvod

CQRS:

Command Query Responsibility Segregation.

---

Ideja:

Odvojiti:

```
Write model

od

Read model
```

---

Primer:

Write:

```
Order Service
```

---

Read:

```
Search Database
```

---

Tok:

```text
Command

↓

Write DB

↓

Event

↓

Read Model Update
```

---

# Event-Driven Concurrency

Concurrency model:

```text
Events

↓

Consumers

↓

Workers

↓

Side Effects
```

---

Svaki consumer ima:

- svoj concurrency limit,
- svoj retry,
- svoj lifecycle.

---

# Event-Driven Problemi

---

## Duplicate Events

Rešenje:

```
idempotency
```

---

## Out-of-order Events

Rešenje:

```
ordering
```

---

## Failed Consumer

Rešenje:

```
retry + DLQ
```

---

## Schema Change

Rešenje:

```
versioning
```

---

# Production Event Checklist

Event sistem treba imati:

✅ stable schema

✅ unique event ID

✅ idempotent consumers

✅ retry strategy

✅ DLQ

✅ monitoring

✅ tracing

---

# Mentalni model

Request-based:

```text
Service A

↓

Service B
```

---

Event-based:

```text
Event

↓

Many Consumers
```

---

Senior dizajn:

```
Events

+

Idempotency

+

Ordering

+

Replay

+

Observability
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Event-driven architecture

✅ Command vs Event

✅ Event schema dizajn

✅ Pub/Sub model

✅ Consumer Groups

✅ Event ordering

✅ Event replay

✅ CQRS uvod

---

# Distributed Concurrency — Reliability Patterns

---

# 📚 Sadržaj

- Distributed Failure Model
- Idempotency detaljno
- Distributed Transactions
- Saga Pattern
- Outbox Pattern
- Consistency modeli
- Failure Recovery

---

# Distributed Failure Model

U distribuiranom sistemu ne postoji samo:

```
Success

ili

Failure
```

---

Postoji:

```
Partial Failure
```

---

Primer:

```text
Order Service

✓ kreirao order


Payment Service

✗ pao
```

---

Sada imamo:

```
nepotpuno izvršenu operaciju
```

---

# Problem Atomicity

U jednoj bazi:

```sql
BEGIN TRANSACTION

UPDATE A

UPDATE B

COMMIT
```

---

Sve ili ništa.

---

Ali:

```text
Database A

+

Database B

+

External API
```

---

Nema jednostavne transakcije.

---

# Idempotency

Jedan od najvažnijih principa.

---

Definicija:

> Operacija koja se može izvršiti više puta, a rezultat ostaje isti.

---

Primer:

---

## Neidempotentno

```text
Increase balance +100
```

---

Poziv:

```
1x

=100
```

---

Poziv:

```
2x

=200
```

---

Problem.

---

## Idempotentno

```text
Set payment status = PAID
```

---

Poziv:

```
1x

PAID
```

---

Poziv:

```
2x

PAID
```

---

Rezultat isti.

---

# Idempotency Key

Najčešći mehanizam.

---

Request:

```json
{
"idempotency_key":"abc123",
"amount":100
}
```

---

Server čuva:

```
abc123

↓

already processed
```

---

Ako isti request dođe ponovo:

```
return previous result
```

---

# Idempotent Consumer

Kod message sistema:

poruka može stići:

```
jednom

ili

više puta
```

---

Consumer proverava:

```text
Da li sam već obradio ovaj event?
```

---

Primer tabela:

```sql
processed_events

event_id

processed_at
```

---

Pre obrade:

```sql
SELECT event_id
```

---

Ako postoji:

```
skip
```

---

# Distributed Transactions

Problem:

Imamo:

```
Order DB

+

Payment DB

+

Inventory DB
```

---

Želimo:

```
sve uspe

ili

ništa
```

---

Ali mreža komplikuje stvar.

---

# Two Phase Commit (2PC)

Jedan pristup.

---

Model:

```text
Coordinator

      |

+-----+-----+

DB1       DB2
```

---

Faza 1:

Prepare

---

Svi kažu:

```
READY
```

---

Faza 2:

Commit

---

Problem:

- spor,
- lock-ovi,
- teško skalira.

---

Zato se često izbegava u modernim sistemima.

---

# Saga Pattern

Najčešće rešenje.

---

Ideja:

Velika transakcija se deli na:

```
više lokalnih transakcija
```

---

Primer:

Order proces:

```text
Create Order

↓

Reserve Inventory

↓

Charge Payment

↓

Send Email
```

---

Svaki korak:

ima svoju kompenzaciju.

---

# Saga sa kompenzacijama

Ako Payment padne:

Prethodno:

```
Inventory reserved
```

---

Undo:

```
Release inventory
```

---

Model:

```text
Step

↓

Success

↓

Next Step


Failure

↓

Compensation
```

---

# Saga Primer

Normalno:

```text
Create Order

↓

Reserve Item

↓

Charge Card

↓

Complete
```

---

Greška:

```text
Charge Card FAILED
```

---

Rollback:

```text
Release Item

↓

Cancel Order
```

---

# Choreography Saga

Servisi reaguju na evente.

---

Primer:

```text
OrderCreated

↓

InventoryReserved

↓

PaymentRequested

↓

PaymentCompleted
```

---

Nema centralnog koordinatora.

---

Prednost:

- loose coupling.

---

Mana:

- teško pratiti tok.

---

# Orchestration Saga

Postoji centralni orchestrator.

---

Model:

```text
          Orchestrator

          /    |    \

       Order Payment Inventory
```

---

Orchestrator kaže:

```
uradi sledeći korak
```

---

Prednost:

- lakša kontrola.

---

Mana:

- centralna komponenta.

---

# Outbox Pattern

Jedan od najvažnijih pattern-a.

---

Problem:

Želimo:

1. sačuvati podatke u DB

2. poslati event

---

Naivno:

```text
Save DB

↓

Publish Event
```

---

Šta ako:

DB uspe

ali:

publish padne?

---

Dobijamo:

```
state postoji

event izgubljen
```

---

# Outbox Rešenje

Čuvamo event u istoj transakciji.

---

Model:

```text
BEGIN

Save Order

Save Event to Outbox Table

COMMIT
```

---

Posle:

Background worker:

```text
Outbox Table

↓

Publish Event
```

---

Prednost:

DB state i event su konzistentni.

---

# Outbox Arhitektura

```text
Application

      |

      ↓

Database

+-------------+

| Orders      |

| Outbox      |

+-------------+

      |

      ↓

Publisher Worker

      |

      ↓

Message Broker
```

---

# Consistency Modeli

Distribuirani sistemi često nemaju instant konzistentnost.

---

# Strong Consistency

Odmah svi vide isto stanje.

---

Primer:

```
bank balance
```

---

Mana:

- sporije,
- manje skalabilno.

---

# Eventual Consistency

Stanje će se uskladiti vremenom.

---

Primer:

```text
Order created

↓

Inventory update

↓

Search index update
```

---

Može postojati kratki period:

```
staro stanje
```

---

# CAP Teorema

Distribuirani sistem ima tri osobine:

- Consistency
- Availability
- Partition Tolerance

---

U slučaju network partition-a:

ne možemo imati istovremeno:

```
100% Consistency

+

100% Availability
```

---

Moramo napraviti kompromis.

---

# Failure Recovery

Production sistem mora imati:

---

## Retry

Za privremene greške.

---

## Timeout

Za spore operacije.

---

## Circuit Breaker

Za neispravne servise.

---

## Dead Letter Queue

Za neuspešne poruke.

---

## Manual Recovery

Za kritične slučajeve.

---

# Circuit Breaker Model

Stanja:

```text
Closed

↓

Open

↓

Half Open

↓

Closed
```

---

Closed:

```
normalno radi
```

---

Open:

```
blokira pozive
```

---

Half Open:

```
testira oporavak
```

---

# Reliability Checklist

Distributed servis treba imati:

✅ idempotency

✅ retry policy

✅ timeout

✅ DLQ

✅ monitoring

✅ tracing

✅ recovery plan

---

# Mentalni model

Monolith:

```
Transaction

↓

Commit
```

---

Distributed:

```
Transaction

↓

Messages

↓

Compensation

↓

Recovery
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ distributed failure model

✅ idempotency

✅ idempotency keys

✅ distributed transactions

✅ Saga pattern

✅ Outbox pattern

✅ consistency modele

✅ failure recovery

---

# Distributed Concurrency — Coordination i State Management

---

# 📚 Sadržaj

- Shared State problem
- Distributed Locks
- Leader Election
- Consensus uvod
- Coordination sistemi
- Lease koncept
- Distributed Scheduling
- Go implementacioni pristupi

---

# Shared State Problem

U jednom procesu:

```go
var counter int

mu.Lock()

counter++

mu.Unlock()
```

---

Imamo:

```
shared memory
```

---

U distribuiranom sistemu:

```text
Service A

Memory A


Service B

Memory B
```

---

Nema:

```go
mutex.Lock()
```

između mašina.

---

# Distributed Coordination

Koordinacija znači:

> Više instanci mora da donese konzistentnu odluku.

---

Primer:

Imamo:

```
10 Worker instanci
```

---

Samo jedna treba da izvrši:

```
Daily Cleanup Job
```

---

Problem:

Bez koordinacije:

```
Worker 1 → start

Worker 2 → start

Worker 3 → start
```

---

Dobijamo:

```
duplicate execution
```

---

# Distributed Lock

Distributed Lock omogućava:

```
samo jedan vlasnik resursa
```

---

Model:

```text
Worker A

   |

   ↓

 Lock

   |

   ↓

Critical Section
```

---

Dok A drži lock:

```
Worker B

wait
```

---

# Lokalni vs Distributed Lock

| Lokalni Mutex | Distributed Lock |
|-|-|
| jedna memorija | više procesa |
| nanosekunde | mrežni poziv |
| runtime kontrola | external storage |
| jednostavan | kompleksan |

---

# Kako implementirati Distributed Lock?

Najčešće preko:

- Redis,
- Database,
- Coordination service.

---

Primer:

```text
Redis

key:

job:cleanup

value:

worker-123
```

---

Ako key postoji:

```
lock zauzet
```

---

Ako ne postoji:

```
dobij lock
```

---

# Lock Lease

Veliki problem:

Šta ako worker padne?

---

Primer:

```
Worker A

dobije lock


↓

crash
```

---

Ako nema expiry:

```
lock zauvek ostaje
```

---

Rešenje:

# Lease

Lock ima vreme trajanja.

---

Primer:

```
Lock TTL:

30 seconds
```

---

Ako worker nestane:

lock automatski ističe.

---

# Fencing Token

Napredna zaštita.

---

Problem:

Worker A:

```
dobije lock
```

---

Mreža uspori.

---

Lock istekne.

---

Worker B:

```
dobije lock
```

---

Sada:

```
A i B rade
```

---

Rešenje:

svaki lock dobija token:

```
A = token 1

B = token 2
```

---

Storage prihvata samo:

```
najnoviji token
```

---

# Leader Election

Čest problem:

Ko je glavni?

---

Primer:

Imamo:

```
10 API servera
```

---

Jedan treba da radi:

- scheduler,
- cleanup,
- migration.

---

Biramo:

```
Leader
```

---

Ostali:

```
Followers
```

---

Model:

```text
       Leader

      /  |  \

   F1    F2   F3
```

---

# Leader Lifecycle

Leader mora:

```
Acquire leadership

↓

Renew lease

↓

Execute tasks

↓

Release
```

---

Ako padne:

```
new election
```

---

# Leader Election Problem

Šta ako:

```
dva lidera postoje?
```

---

Ovo se zove:

# Split Brain

---

Primer:

```text
Network partition
```

---

Obe strane misle:

```
ja sam leader
```

---

Posledice:

- dupli posao,
- korupcija stanja.

---

Rešenja:

- consensus algoritmi,
- fencing tokens,
- quorum.

---

# Consensus Uvod

Consensus znači:

> Više čvorova se dogovara oko jedne vrednosti.

---

Primer:

```
Ko je leader?

Koji je sledeći broj?

Koji state važi?
```

---

Poznati algoritmi:

- Paxos
- Raft

---

# Raft Model

Raft deli sistem na:

```
Leader

Followers

Candidates
```

---

Tok:

```text
Election

↓

Leader

↓

Replication

↓

Commit
```

---

# Coordination Systems

Sistemi koji rešavaju koordinaciju:

---

## Etcd

Koristi se za:

- Kubernetes state,
- service discovery,
- leader election.

---

## Consul

Koristi se za:

- service discovery,
- health checking.

---

## ZooKeeper

Tradicionalno:

- distributed coordination.

---

# Distributed Scheduler

Primer:

Imamo:

```
100 servera
```

---

Treba pokrenuti:

```
daily backup
```

---

Ne želimo:

```
100 backup jobova
```

---

Potrebno:

```
jedan scheduler owner
```

---

Model:

```text
Scheduler Leader

↓

Create Jobs

↓

Workers
```

---

# State Management

Distribuirani sistemi moraju odlučiti:

Gde živi state?

---

Opcije:

## Database as Source of Truth

Najčešće.

---

## Event Log

Primer:

```
Events

↓

Current State
```

---

## Cache

Za brzinu:

```
Redis

↓

temporary state
```

---

# Eventual Consistency Problem

Primer:

Service A:

```
User updated
```

---

Service B:

još vidi:

```
old value
```

---

Sistem mora znati:

```
kada je stanje validno
```

---

# Go Implementacioni Pristupi

Go često koristi:

---

## Context

Za lifecycle.

---

## Channels

Za lokalnu koordinaciju.

---

## External systems

Za distributed coordination.

---

Primer:

```text
Go Service

↓

Redis / DB / Etcd

↓

Coordination
```

---

# Anti-Patterns

---

## 1. Global Distributed Mutex za sve

Problem:

postaje bottleneck.

---

## 2. Lock bez timeout-a

Problem:

deadlock.

---

## 3. Leader bez lease-a

Problem:

stari leader ostaje aktivan.

---

## 4. Shared state bez vlasnika

Problem:

race condition.

---

# Production Checklist

Distributed coordination treba imati:

✅ timeout

✅ lease expiration

✅ ownership model

✅ failure handling

✅ observability

✅ recovery plan

---

# Mentalni model

Lokalno:

```
Mutex

↓

Memory
```

---

Distribuirano:

```
Lock Service

↓

Network

↓

Multiple Processes
```

---

Senior pristup:

```
Ne deli state.

Deli događaje.

Koordinaciju koristi samo kada mora.
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ shared state problem

✅ distributed locks

✅ leases

✅ fencing tokens

✅ leader election

✅ split brain problem

✅ consensus uvod

✅ coordination sisteme

✅ distributed scheduling

---

# Distributed Concurrency — Production Architecture Case Study

---

# 📚 Sadržaj

- Production distributed sistem
- Service komunikacija
- Event-driven tok
- Distributed Worker Cluster
- Reliability layer
- Coordination layer
- Observability
- Finalna arhitektura

---

# Case Study

Dizajniramo sistem:

# Order Processing Platform

Sistem obrađuje:

- kreiranje porudžbina,
- naplatu,
- rezervaciju proizvoda,
- slanje notifikacija.

---

# Zahtevi

Sistem mora imati:

✅ visoku dostupnost

✅ horizontalno skaliranje

✅ otpornost na greške

✅ asinhronu obradu

✅ retry mehanizam

✅ monitoring

---

# Naivni dizajn

```text
Client

↓

Order Service

↓

Payment Service

↓

Inventory Service

↓

Notification Service
```

---

Problem:

Ako:

```
Payment Service
```

padne:

cela operacija staje.

---

# Production dizajn

Koristimo:

- API layer
- Event Bus
- Workers
- Database
- Retry
- DLQ
- Monitoring

---

Arhitektura:

```text
                 Client

                   |

                   ↓

             API Gateway

                   |

                   ↓

             Order Service

                   |

                   ↓

              Event Bus

        +----------+----------+

        |          |          |

   Payment     Inventory   Notification

    Worker       Worker       Worker

        |          |          |

        +----------+----------+

                   |

                Database
```

---

# 1. API Layer

HTTP servis prima zahtev.

---

Odgovor:

```
Order accepted
```

---

Ne čeka:

- payment,
- email,
- inventory.

---

Zašto?

Zato što su spori i nezavisni.

---

# 2. Order Service

Kreira:

```text
OrderCreated Event
```

---

Primer:

```json
{
"type":"OrderCreated",
"order_id":"12345"
}
```

---

Čuva:

```
Order state
```

---

Objavljuje event.

---

# 3. Event Bus

Služi kao:

```
distributed channel
```

---

Analogija:

Go:

```go
chan Event
```

---

Distributed:

```text
Event Broker
```

---

Omogućava:

- buffering,
- retry,
- scaling.

---

# 4. Payment Worker

Consumer:

```text
PaymentRequested
```

---

Proces:

```text
Receive Event

↓

Charge Payment

↓

Publish PaymentCompleted
```

---

Mora imati:

- idempotency,
- timeout,
- retry.

---

# 5. Inventory Worker

Prima:

```
OrderCreated
```

---

Radi:

```
Reserve Item
```

---

Ako nema proizvoda:

publikuje:

```
InventoryFailed
```

---

---

# 6. Notification Worker

Ne blokira glavni tok.

---

Dobija:

```
OrderCompleted
```

---

Radi:

```
Send Email
```

---

Ako email servis padne:

```
retry
```

---

# Distributed Worker Scaling

Primer:

Normalan load:

```text
Payment Workers:

10
```

---

Veliki load:

```text
Payment Workers:

100
```

---

Dodajemo instance:

```
horizontal scaling
```

---

# Reliability Layer

Svaki consumer ima:

---

## Timeout

Primer:

```text
Payment:

3 seconds
```

---

## Retry

Primer:

```
Attempt 1

↓

1s

↓

Attempt 2

↓

5s
```

---

## DLQ

Ako stalno pada:

```
Dead Letter Queue
```

---

# Idempotency Layer

Problem:

Event može stići dva puta.

---

Primer:

```text
PaymentCompleted
```

---

Consumer proverava:

```text
event_id exists?
```

---

Ako postoji:

```
ignore
```

---

# Saga Workflow

Order proces:

```text
Order Created

↓

Reserve Inventory

↓

Charge Payment

↓

Complete Order
```

---

Ako payment fail:

Kompenzacija:

```text
Release Inventory

↓

Cancel Order
```

---

# Outbox Pattern

Order Service:

jedna DB transakcija:

```text
BEGIN

Insert Order

Insert Outbox Event

COMMIT
```

---

Background publisher:

```text
Outbox

↓

Event Bus
```

---

Rezultat:

nema izgubljenih eventova.

---

# Distributed Coordination

Primer:

Daily cleanup.

---

Imamo:

```
50 service instances
```

---

Ne želimo:

```
50 cleanup izvršavanja
```

---

Koristimo:

```
Leader Election
```

---

Jedan leader:

```
Cleanup Scheduler
```

---

# Observability Layer

Distributed sistem bez observability-ja je crna kutija.

---

Potrebno:

---

# Metrics

Primer:

```
events/sec

queue depth

processing latency

error rate
```

---

# Logs

Svaki event ima:

```
correlation_id
```

---

Tok:

```text
Request ID

↓

Order Service

↓

Payment Worker

↓

Notification
```

---

# Distributed Tracing

Omogućava:

```
jedan request kroz više servisa
```

---

Primer:

```text
API

20ms

↓

Order

30ms

↓

Payment

200ms
```

---

Vidimo:

gde je problem.

---

# Failure Scenario

Scenario:

Payment servis je spor.

---

Šta se dešava?

```text
Payment latency ↑

↓

Worker queue raste

↓

Retry povećava load

↓

Circuit breaker aktivan

↓

Sistem ostaje stabilan
```

---

# Finalna Production Arhitektura

```text
                         Client

                           |

                           ↓

                    API Gateway

                           |

                           ↓

                    Order Service

                           |

                    +------+------+

                    | Outbox DB |

                    +------+------+

                           |

                           ↓

                    Event Broker

                           |

        +------------------+------------------+

        |                  |                  |

 Payment Workers   Inventory Workers   Notification Workers

        |                  |                  |

        +------------------+------------------+

                           |

                       Databases


                           |

                    Observability

              Metrics + Logs + Traces
```

---

# Production Design Principi

## 1. Ne pretpostavljaj pouzdanu mrežu

Svaki poziv može:

- kasniti,
- pasti.

---

## 2. Event može stići više puta

Koristi:

```
idempotency
```

---

## 3. Svaki posao mora imati granice

Koristi:

- timeout,
- retry limit,
- queue limit.

---

## 4. State mora imati vlasnika

Ne deliti state bez potrebe.

---

## 5. Sve mora biti merljivo

Bez:

- metrics,
- logs,
- traces,

nema debugovanja.

---

# Senior Distributed Concurrency Model

Junior:

```
Kako poslati HTTP request?
```

---

Medior:

```
Kako koristiti queue?
```

---

Senior:

```
Kako dizajnirati sistem
koji preživljava failure?
```

---

# 📋 Rezime Modula #4.12

Naučili smo:

✅ distributed concurrency principe

✅ message queue sisteme

✅ distributed workers

✅ event-driven arhitekturu

✅ Saga pattern

✅ Outbox pattern

✅ distributed locks

✅ leader election

✅ production architecture

---

# 🎯 Modul #4.12 završen

Put:

```text
Goroutine

↓

Channel

↓

Concurrency Pattern

↓

Production Architecture

↓

Distributed System
```

---

### ➡️ Sledeća lekcija **[**Mastering Go Concurrency — Final Architecture & Best Practices**](13-mastering-go-concurrency.md)**

Obuhvatiće:

- kompletan concurrency mentalni model
- kada koristiti koji pattern
- senior decision making
- concurrency checklist
- production guidelines
- završni projekat Modula #4