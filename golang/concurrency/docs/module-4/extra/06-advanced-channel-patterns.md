# Advanced Channel Patterns

> Module: #4 — Advanced Go Concurrency
>
> Section: Extras
>
> Topic: Advanced Channel Patterns
>
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Šta su Channel Patterns?
- Zašto su obrasci važni?
- Channel Ownership
- Single Sender Principle
- Channel Closing Principle
- Nil Channel Pattern
- Disable Case Pattern
- Optional Channel Pattern
- Dynamic Enable/Disable
- Best Practices

---

# 1. Uvod

Do sada smo naučili:

- goroutines
- channels
- select
- worker pools
- pipelines
- fan-out / fan-in
- channel internals

---

Sada prelazimo na:

```
production patterns
```

Ovo su obrasci koji se koriste u:

- Go runtime-u
- Kubernetes-u
- Docker-u
- Prometheus-u
- Grafana-i
- HashiCorp projektima
- cloud-native sistemima

---

Oni nisu novi concurrency primitivi.

Već:

```
kombinacije postojećih primitiva
```

koje rešavaju često ponavljajuće probleme.

---

# 2. Šta je Channel Pattern?

Pattern predstavlja:

```
proveren način rešavanja
jednog concurrency problema.
```

---

Primer:

Umesto da svaki put izmišljamo novi način za:

```
shutdown worker-a
```

koristimo:

```
Done Channel Pattern
```

---

Isto važi za:

- broadcast
- cancellation
- multiplexing
- throttling
- load balancing
- graceful shutdown

---

# 3. Zašto Su Pattern-i Važni?

Bez pattern-a:

svaki projekat razvija:

```
svoja pravila
```

---

Posledice:

- race conditions
- deadlock
- goroutine leak
- teško održavanje
- komplikovan kod

---

Pattern-i daju:

- predvidljivo ponašanje
- lakše testiranje
- bolju čitljivost
- jednostavnije održavanje

---

# 4. Channel Ownership

Jedan od najvažnijih principa.

---

Pravilo:

```
Svaki channel ima vlasnika.
```

---

Owner je odgovoran za:

- kreiranje channel-a
- slanje vrednosti
- zatvaranje channel-a

---

Receiver:

- čita
- obrađuje podatke

---

Ne zatvara channel.

---

# 5. Ownership Model

```
          Owner

            |

            |

            v

      create channel

            |

            v

        send values

            |

            v

      close(channel)

            |

            v

        Consumers
```

---

Ovaj model eliminiše:

- duplo zatvaranje
- nejasan lifecycle
- većinu panic situacija

---

# 6. Single Sender Principle

Jedno od najvažnijih Go pravila.

---

Ako postoji:

```
jedan sender
```

onda:

```
on zatvara channel.
```

---

Primer:

```go
func producer(out chan<- int) {

	defer close(out)

	for i := 0; i < 10; i++ {

		out <- i

	}

}
```

---

Potrošač:

```go
for value := range out {

	fmt.Println(value)

}
```

---

Jednostavno.

Bezbedno.

---

# 7. Zašto Više Sender-a Komplikuje Stvari?

Primer:

```
Producer A

      \

       \

        > channel

       /

Producer B
```

---

Ko zatvara channel?

---

Ako:

```
A zatvori
```

dok:

```
B još šalje
```

dobijamo:

```
panic:

send on closed channel
```

---

# 8. Rešenje za Više Sender-a

Umesto:

```
više sender-a zatvara channel
```

koristimo:

```
jedan coordinator
```

---

Model:

```
P1

P2

P3

 |

 |

Coordinator

 |

close(channel)
```

---

Samo coordinator zatvara channel.

---

# 9. Channel Closing Principle

Jedno od najpoznatijih Go pravila glasi:

> **Don't close a channel from the receiver side.**

---

Drugo pravilo:

> **Don't close a channel if you are not the last sender.**

---

Ako poštujemo ova dva pravila:

većina problema nestaje.

---

# 10. Nil Channel Pattern

Veoma zanimljiv pattern.

---

Primer:

```go
var ch chan int
```

---

Vrednost:

```
nil
```

---

Send:

```go
ch <- 10
```

---

Rezultat:

```
blokira zauvek
```

---

Receive:

```go
<-ch
```

---

Takođe:

```
blokira zauvek
```

---

# 11. Zašto Je Nil Koristan?

Zbog `select`.

---

Primer:

```go
var ch chan int

select {

case <-ch:

case <-time.After(time.Second):

}
```

---

Pošto je:

```
ch == nil
```

---

Taj `case` nikada ne može biti izabran.

---

# 12. Disable Case Pattern

Primer:

```go
var input <-chan int

if enabled {

	input = realChannel

} else {

	input = nil

}
```

---

Kasnije:

```go
select {

case value := <-input:

	process(value)

}
```

---

Ako je:

```
input == nil
```

---

Runtime ignoriše taj case.

---

Nema potrebe za:

```go
if enabled {

	select {

	...

	}

}
```

---

# 13. Dynamic Enable/Disable

Veoma često u production sistemima.

---

Primer:

```
feature enabled

↓

channel aktivan
```

---

Kasnije:

```
feature disabled

↓

channel = nil
```

---

Select automatski prilagođava ponašanje.

---

# 14. Optional Channel Pattern

Neki kanali postoje samo u određenim situacijama.

---

Primer:

```go
type Worker struct {

	logs chan string

}
```

---

Ako:

```
logs == nil
```

---

Worker:

ne šalje logove.

---

Bez dodatnih:

```go
if logs != nil
```

na svakom mestu.

---

# 15. Prednosti Nil Pattern-a

✅ Jednostavan kod

✅ Manje `if` grana

✅ Prirodna integracija sa `select`

✅ Laka konfiguracija

✅ Dinamičko uključivanje i isključivanje tokova

---

# 16. Česte Greške

## Greška 1

Zaboravljen nil channel.

---

Primer:

```go
var ch chan int

<-ch
```

---

Rezultat:

```
deadlock
```

---

## Greška 2

Receiver zatvara channel.

---

Rezultat:

mogući:

```
panic
```

---

## Greška 3

Više sender-a zatvara isti channel.

---

Rezultat:

```
double close

ili

send on closed channel
```

---

# 17. Senior Pravila

✔️ Channel ima jednog vlasnika.

---

✔️ Owner kreira channel.

---

✔️ Owner zatvara channel.

---

✔️ Receiver nikada ne zatvara channel.

---

✔️ Nil channel je validan alat za kontrolu `select` logike.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta su channel patterns

✅ channel ownership

✅ single sender principle

✅ channel closing principle

✅ nil channel

✅ disable case pattern

✅ optional channel

✅ dynamic enable/disable

---

# Advanced Channel Patterns

## Deo #2 — Done Channel i Or-Done Pattern

---

# 📚 Sadržaj

- Zašto je cancellation važan?
- Done Channel Pattern
- Broadcast shutdown
- Cancellation propagacija
- Goroutine leak problem
- Or-Done Pattern
- Implementacija Or-Done helper funkcije
- Kada koristiti `context.Context`, a kada done channel?

---

# 1. Problem Bez Cancellation-a

Jedan od najčešćih problema u konkurentnim programima je:

```
goroutine leak
```

---

Primer:

```go
func worker(ch <-chan int) {

	for {

		value := <-ch

		process(value)

	}

}
```

---

Ako:

- više niko ne šalje podatke
- channel nikada nije zatvoren

onda:

```
worker zauvek čeka
```

---

Takva goroutine nikada neće završiti.

---

# 2. Done Channel Pattern

Najjednostavniji način za signalizaciju prekida rada.

---

Primer:

```go
done := make(chan struct{})
```

---

Worker:

```go
select {

case value := <-jobs:

	process(value)

case <-done:

	return

}
```

---

Glavna goroutine:

```go
close(done)
```

---

Rezultat:

```
svi worker-i završavaju rad
```

---

# 3. Zašto `struct{}`?

Najčešće se koristi:

```go
chan struct{}
```

---

Razlog:

```
signal

bez podataka
```

---

`struct{}` zauzima:

```
0 bajtova
```

---

Bitno je:

```
događaj

ne vrednost
```

---

# 4. Broadcast Shutdown

Najveća prednost `close(done)` je:

```
broadcast
```

---

Primer:

```
              done

               |

      +--------+--------+

      |        |        |

      v        v        v

    Worker1 Worker2 Worker3
```

---

Kada izvršimo:

```go
close(done)
```

---

Svi worker-i se bude istovremeno.

---

Nije potrebno:

```go
done <- struct{}{}
done <- struct{}{}
done <- struct{}{}
```

---

# 5. Cancellation Propagation

Veći sistemi imaju više nivoa goroutines.

---

Primer:

```
API Request

      |

      v

Controller

      |

      v

Service

      |

      v

Worker Pool
```

---

Jedan signal za prekid treba da stigne do svih.

---

Done channel omogućava upravo to.

---

# 6. Primer Worker-a

```go
func worker(

	id int,

	jobs <-chan Job,

	done <-chan struct{},

) {

	for {

		select {

		case job := <-jobs:

			process(job)

		case <-done:

			return

		}

	}

}
```

---

Bez obzira šta worker radi,

prekid je uvek moguć.

---

# 7. Goroutine Leak

Loš primer:

```go
go func() {

	for {

		job := <-jobs

		process(job)

	}

}()
```

---

Ako:

```
jobs

nikada ne dobije novu vrednost
```

---

goroutine ostaje zauvek blokirana.

---

# 8. Ispravno Rešenje

Koristimo:

```go
select {

case job := <-jobs:

	process(job)

case <-done:

	return

}
```

---

Sada postoji izlaz iz petlje.

---

# 9. Or-Done Pattern

Ponekad želimo:

```
prekini čim

bilo koji

done channel bude zatvoren
```

---

Primer:

```
shutdown

ili

timeout

ili

parent cancelled
```

---

Sve treba da prekinu rad.

---

# 10. Mentalni Model

```
doneA

   \

    \

     OR

    /

   /

doneB

   \

    \

doneC


      |

      v

combined done
```

---

Ako se zatvori:

```
bilo koji
```

---

zatvara se i rezultat.

---

# 11. Or-Done Implementacija

Pojednostavljen primer:

```go
func orDone(

	done <-chan struct{},

	input <-chan int,

) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for {

			select {

			case <-done:

				return

			case value, ok := <-input:

				if !ok {

					return

				}

				select {

				case out <- value:

				case <-done:

					return

				}

			}

		}

	}()

	return out

}
```

---

# 12. Zašto Dvostruki `select`?

Prvi:

```go
select
```

čeka:

- podatke
- cancellation

---

Drugi:

```go
select
```

čeka:

- mogućnost slanja
- cancellation

---

Bez drugog `select`-a:

goroutine može ostati blokirana na:

```go
out <- value
```

---

# 13. Leak Prevention

Or-Done Pattern sprečava:

```
goroutine leak
```

---

Jer svaka blokirajuća operacija ima:

```
escape path
```

---

To je veoma važan production princip.

---

# 14. Done Channel vs Context

Done channel:

```
jedan signal
```

---

Context:

- cancellation
- deadline
- timeout
- values
- hijerarhija

---

Ako razvijate novu aplikaciju,

najčešće je bolji izbor:

```go
context.Context
```

---

# 15. Kada Koristiti Done Channel?

Done channel je odličan kada:

- radite interne pipeline obrasce
- pišete biblioteke niskog nivoa
- implementirate concurrency primitive
- želite minimalni overhead

---

# 16. Kada Koristiti Context?

Koristite `context.Context` kada:

- obrađujete HTTP zahteve
- radite sa bazama podataka
- koristite RPC ili gRPC
- pravite javne API-je
- želite propagaciju timeout-a i cancellation-a

---

# 17. Production Saveti

✔️ Svaka dugotrajna goroutine treba da ima način za prekid rada.

---

✔️ Nikada nemojte pretpostaviti da će channel uvek biti zatvoren.

---

✔️ Ako postoji mogućnost beskonačnog čekanja,

dodajte cancellation mehanizam.

---

✔️ Svaki `select` koji može dugo da čeka treba da razmotri i signal za prekid.

---

# 18. Česte Greške

❌ Worker bez mogućnosti izlaska.

---

❌ Beskonačna petlja:

```go
for {

	job := <-jobs

}
```

---

❌ Slanje pojedinačnih vrednosti na `done` channel umesto:

```go
close(done)
```

---

❌ Ignorisanje cancellation signala u dugim pipeline-ovima.

---

# 19. Senior Pravilo

Dobra konkurentna aplikacija ne razmišlja samo o:

```
kako pokrenuti goroutine
```

---

Već i o:

```
kako je pravilno zaustaviti
```

---

Cancellation nije dodatak.

---

On je sastavni deo dizajna konkurentnog sistema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Done Channel Pattern

✅ Broadcast shutdown

✅ Cancellation propagaciju

✅ Goroutine leak problem

✅ Or-Done Pattern

✅ Dvostruki `select`

✅ Leak prevention

✅ Done channel vs `context.Context`

---

# Advanced Channel Patterns

## Deo #3 — Tee Pattern i Bridge Pattern

---

# 📚 Sadržaj

- Šta je Tee Pattern?
- Zašto koristiti Tee Pattern?
- Implementacija Tee Pattern-a
- Bridge Pattern
- Channel-of-Channels
- Flattening Stream
- Dinamičko spajanje tokova
- Production primeri
- Prednosti i mane

---

# 1. Šta je Tee Pattern?

Naziv dolazi od slova:

```
T
```

Jer:

```
        Input

          |

          |

          v

        Tee

       /   \

      /     \

 Output1   Output2
```

---

Jedan ulaz.

---

Dva identična izlaza.

---

Svaka vrednost se kopira na oba izlaza.

---

# 2. Kada Koristiti Tee Pattern?

Primer:

Jedan stream logova.

---

Želimo:

```
Logger

+

Metrics

+

Audit
```

---

Ne želimo da:

producer šalje tri puta.

---

Već:

```
Producer

      |

      v

     Tee

   /      \

Logger   Metrics
```

---

# 3. Osnovna Implementacija

```go
func tee(

	in <-chan int,

) (

	<-chan int,

	<-chan int,

) {

	out1 := make(chan int)

	out2 := make(chan int)

	go func() {

		defer close(out1)
		defer close(out2)

		for value := range in {

			v := value

			out1 <- v
			out2 <- v

		}

	}()

	return out1, out2

}
```

---

Oba izlaza dobijaju:

istu sekvencu.

---

# 4. Problem Blokiranja

Naivna implementacija ima problem.

---

Ako:

```
out1

ready
```

---

A:

```
out2

blocked
```

---

Cela goroutine čeka.

---

To može zaustaviti ceo pipeline.

---

# 5. Production Tee Pattern

Production implementacije koriste:

- `select`
- privremene promenljive
- nil channel pattern

---

Ideja:

```
dok oba izlaza ne prime

isti element

↓

ne nastavljaj dalje
```

---

# 6. Tee Mentalni Model

```
Input

 |

 v

value = 42

 |

 +--------+

 |        |

 v        v

42       42
```

---

Nema izmene podataka.

---

Samo:

```
dupliranje toka
```

---

# 7. Šta je Bridge Pattern?

Bridge Pattern rešava drugi problem.

---

Ulaz nije:

```
channel
```

---

Već:

```
channel od channel-a
```

---

Odnosno:

```go
chan <-chan int
```

---

# 8. Problem Channel-of-Channels

Primer:

```
Stream A

Stream B

Stream C
```

---

Dobijamo:

```
chan

↓

chan

↓

values
```

---

Želimo:

```
A

B

C

↓

jedan stream
```

---

# 9. Mentalni Model Bridge Pattern-a

```
         Stream1

             \

              \

               \

                Bridge

               /

              /

             /

        Stream2

             \

              \

           Stream3


                |

                v

          Output Stream
```

---

Bridge spaja:

više stream-ova

u jedan.

---

# 10. Osnovna Implementacija

Pojednostavljeno:

```go
func bridge(

	streams <-chan <-chan int,

) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for stream := range streams {

			for value := range stream {

				out <- value

			}

		}

	}()

	return out

}
```

---

# 11. Gde Se Koristi Bridge?

Vrlo često kod:

- pipeline sistema
- event processing-a
- worker pool-ova
- stream processing-a

---

Posebno kada broj stream-ova:

nije unapred poznat.

---

# 12. Dynamic Stream Processing

Primer:

Nova goroutine napravi novi stream.

---

Dodaje ga:

```
streams

↓

bridge

↓

consumer
```

---

Consumer ne mora znati:

koliko stream-ova postoji.

---

# 13. Bridge i Cancellation

Production verzija skoro uvek koristi:

```go
done <-chan struct{}
```

---

Zašto?

Jer:

```
jedan stream

može zauvek čekati
```

---

Cancellation omogućava izlazak.

---

# 14. Tee vs Fan-Out

Važna razlika.

---

Tee:

```
svaki izlaz

dobija sve podatke
```

---

Fan-Out:

```
svaki podatak

ide jednom worker-u
```

---

Primer:

Tee:

```
1

↓

A

B
```

---

Fan-Out:

```
1

↓

A
```

---

ili

```
1

↓

B
```

---

Nikada oba.

---

# 15. Bridge vs Fan-In

Bridge:

spaja:

```
stream-ove
```

---

Fan-In:

spaja:

```
pojedinačne vrednosti
```

iz više aktivnih kanala istovremeno.

---

Bridge radi sa:

```
channel-of-channels
```

---

# 16. Production Primer

Zamislimo sistem za obradu logova.

---

Svaki servis otvara:

```
stream logova
```

---

Bridge:

```
Service A

Service B

Service C

↓

Bridge

↓

Central Logger
```

---

Dodavanje novog servisa:

ne zahteva promenu logger-a.

---

# 17. Česte Greške

❌ Tee koji može zauvek blokirati.

---

❌ Bridge bez cancellation mehanizma.

---

❌ Pretpostavka da svi stream-ovi završavaju.

---

❌ Zaboravljeno zatvaranje izlaznog channel-a.

---

# 18. Senior Pravila

✔️ Tee kopira svaki element.

---

✔️ Bridge spaja tokove.

---

✔️ Uvek razmišljati o blokiranju.

---

✔️ Svaki production pipeline treba da podrži cancellation.

---

✔️ Koristiti `nil` channel pattern kada je potrebno privremeno isključiti izlaz.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Tee Pattern

✅ Zašto se koristi

✅ Problemi blokiranja

✅ Bridge Pattern

✅ Channel-of-Channels

✅ Dynamic Stream Processing

✅ Tee vs Fan-Out

✅ Bridge vs Fan-In

---

# Advanced Channel Patterns

## Deo #4 — Semaphore, Token Bucket i Drop Pattern

---

# 📚 Sadržaj

- Zašto ograničavati konkurentnost?
- Semaphore Pattern
- Buffered Channel kao Semaphore
- Weighted Semaphore
- Token Bucket Pattern
- Rate Limiting
- Backpressure
- Drop Pattern
- Load Shedding
- Production preporuke

---

# 1. Problem Neograničene Konkurentnosti

Naivan pristup:

```go
for _, task := range tasks {

	go process(task)

}
```

---

Ako postoji:

```
100 000 zadataka
```

dobijamo:

```
100 000 goroutines
```

---

Posledice:

- ogromna potrošnja memorije
- scheduler overload
- CPU contention
- GC pritisak
- pad performansi

---

Potrebno je:

```
ograničiti broj
istovremenih operacija
```

---

# 2. Semaphore Pattern

Semaphore ograničava:

```
broj aktivnih goroutines
```

---

Mentalni model:

```
Semaphore

Capacity = 3


□ □ □
```

---

Svaka goroutine mora:

```
uzeti dozvolu
```

pre rada.

---

Po završetku:

```
vratiti dozvolu
```

---

# 3. Buffered Channel Kao Semaphore

Najčešća implementacija u Go-u:

```go
sem := make(chan struct{}, 3)
```

---

Pre početka rada:

```go
sem <- struct{}{}
```

---

Po završetku:

```go
<-sem
```

---

Kompletan primer:

```go
sem := make(chan struct{}, 3)

for _, job := range jobs {

	sem <- struct{}{}

	go func(job Job) {

		defer func() {

			<-sem

		}()

		process(job)

	}(job)

}
```

---

Nikada neće raditi više od:

```
3 goroutines
```

istovremeno.

---

# 4. Kako Semaphore Radi?

Pretpostavimo:

```
capacity = 2
```

---

Početno stanje:

```
[ ][ ]
```

---

Prva goroutine:

```
[X][ ]
```

---

Druga:

```
[X][X]
```

---

Treća pokušava:

```
[X][X]

↓

blocked
```

---

Čeka dok neko ne oslobodi mesto.

---

# 5. Weighted Semaphore

Ponekad nisu svi zadaci jednako skupi.

---

Primer:

```
Upload slike

↓

1 token
```

---

```
Video encoding

↓

5 tokena
```

---

Jednostavan channel semaphore ovo ne podržava.

---

Za takve slučajeve koristi se:

```
golang.org/x/sync/semaphore
```

koji podržava rezervaciju više "dozvola" odjednom.

---

# 6. Token Bucket Pattern

Rate limiting rešava drugi problem.

---

Ne ograničava:

```
koliko radi istovremeno
```

---

Već:

```
koliko operacija

po jedinici vremena
```

---

Mentalni model:

```
Bucket

● ● ● ● ●
```

---

Svaka operacija:

uzima jedan token.

---

# 7. Kako Radi Token Bucket?

Periodično dodajemo tokene.

---

Na primer:

```
10 tokena

svake sekunde
```

---

Ako bucket ostane prazan:

```
operacije čekaju
```

ili se odbacuju.

---

# 8. Jednostavna Implementacija

```go
tokens := time.Tick(

	100 * time.Millisecond,

)

for job := range jobs {

	<-tokens

	process(job)

}
```

---

Rezultat:

```
10 operacija

u sekundi
```

---

# 9. Semaphore vs Token Bucket

Semaphore:

```
ograničava

paralelizam
```

---

Token Bucket:

```
ograničava

brzinu
```

---

Tabela:

| Pattern | Šta ograničava? |
|----------|-----------------|
| Semaphore | Broj aktivnih operacija |
| Token Bucket | Broj operacija u jedinici vremena |

---

# 10. Backpressure

Backpressure znači:

```
producer

usporava
```

jer:

```
consumer

ne može da stigne
```

---

Primer:

```
Producer

↓

Buffer

↓

Consumer
```

---

Ako se buffer napuni:

producer čeka.

---

# 11. Zašto Je Backpressure Dobar?

Bez njega:

producer može:

- prepuniti memoriju
- napraviti milione zahteva
- izazvati pad sistema

---

Backpressure automatski:

```
izjednačava brzinu
```

između producer-a i consumer-a.

---

# 12. Drop Pattern

Ponekad:

```
čekanje

nije prihvatljivo
```

---

Primer:

telemetrija.

---

Ako je sistem opterećen,

bolje je:

```
odbaciti

neke događaje
```

nego blokirati aplikaciju.

---

# 13. Implementacija Drop Pattern-a

Koristi se:

```go
select {

case ch <- value:

default:

	// drop

}
```

---

Ako je channel pun:

```
default
```

se izvršava odmah.

---

Nema blokiranja.

---

# 14. Load Shedding

Drop Pattern predstavlja oblik:

```
load shedding-a
```

---

Ideja:

```
bolje izgubiti

deo zahteva
```

nego:

```
izgubiti

ceo sistem
```

---

Koristi se kod:

- monitoring sistema
- metrika
- logovanja
- streaming aplikacija

---

# 15. Production Primer

Primer telemetrije:

```go
select {

case metrics <- sample:

default:

	dropped++

}
```

---

Ako sistem ne može da obradi sve metrike,

aplikacija nastavlja normalno da radi.

---

# 16. Kada Ne Koristiti Drop Pattern?

Nikada za:

- finansijske transakcije
- baze podataka
- plaćanja
- kritične događaje
- audit logove

---

Tamo je važnije:

```
tačnost

od brzine
```

---

# 17. Kombinovanje Pattern-a

Production sistemi često kombinuju više obrazaca.

---

Na primer:

```
Producer

↓

Token Bucket

↓

Semaphore

↓

Worker Pool

↓

Consumers
```

---

Dobijamo:

- rate limiting
- ograničen paralelizam
- stabilan throughput

---

# 18. Česte Greške

❌ Prevelik buffer.

---

❌ Premali semaphore.

---

❌ Drop Pattern za kritične podatke.

---

❌ Ignorisanje backpressure-a.

---

❌ Beskonačno kreiranje goroutines.

---

# 19. Senior Pravila

✔️ Semaphore kontroliše konkurentnost.

---

✔️ Token Bucket kontroliše brzinu.

---

✔️ Backpressure štiti sistem od preopterećenja.

---

✔️ Drop Pattern koristi se samo kada je prihvatljiv gubitak podataka.

---

✔️ Najstabilniji sistemi ograničavaju i brzinu i broj paralelnih operacija.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Semaphore Pattern

✅ Buffered channel kao semaphore

✅ Weighted Semaphore

✅ Token Bucket

✅ Rate Limiting

✅ Backpressure

✅ Drop Pattern

✅ Load Shedding

---

# Advanced Channel Patterns

## Deo #5 — Dynamic Fan-Out, Nil Channel Tricks i Channel Ownership u Velikim Sistemima

---

# 📚 Sadržaj

- Dynamic Fan-Out
- Dynamic Worker Scaling
- Select kao State Machine
- Advanced Nil Channel Pattern
- Runtime Enable/Disable
- Channel Ownership u velikim sistemima
- Pipeline Ownership
- Production arhitekture
- Najčešće greške

---

# 1. Statički vs Dinamički Fan-Out

Klasičan Worker Pool ima unapred poznat broj radnika.

Primer:

```go
const workers = 4
```

---

To znači:

```
uvek

4 worker-a
```

---

Ali production sistemi često imaju:

- promenljivo opterećenje
- različit broj CPU jezgara
- promenljiv broj zahteva

---

Zbog toga:

```
broj worker-a

postaje dinamičan
```

---

# 2. Dynamic Fan-Out

Umesto:

```
4 worker-a
```

---

Sistem može imati:

```
4

↓

8

↓

32

↓

12

↓

6
```

---

u zavisnosti od opterećenja.

---

To omogućava:

- bolji throughput
- manju potrošnju resursa
- stabilnije performanse

---

# 3. Dynamic Worker Scaling

Arhitektura:

```
          Jobs

            |

            v

     Worker Manager

      /    |    \

     /     |     \

   W1     W2     W3
```

---

Manager:

- pokreće nove worker-e
- gasi neaktivne worker-e
- prati opterećenje sistema

---

# 4. Kada Dodati Novog Worker-a?

Najčešći kriterijumi:

- buffer je skoro pun
- queue raste
- latencija raste
- CPU nije maksimalno opterećen

---

Primer:

```
queue

capacity = 100

current = 92
```

---

Manager može:

```
spawn worker
```

---

# 5. Kada Ugasiti Worker?

Suprotan scenario.

---

Ako:

```
queue

=

0
```

duže vreme,

worker može završiti rad.

---

Rezultat:

- manje memorije
- manje scheduler aktivnosti
- manje idle goroutines

---

# 6. Select Kao State Machine

Napredni sistemi često koriste:

```go
select
```

kao:

```
state machine
```

---

Primer:

```
WAITING

↓

PROCESSING

↓

SHUTDOWN
```

---

Svako stanje:

aktivira različite channel-e.

---

# 7. Nil Channel Tricks

Podsetimo se:

```
nil channel

nikada nije ready
```

---

To omogućava:

dinamičko uključivanje i isključivanje grana `select` izraza.

---

Primer:

```go
var input <-chan Job

if enabled {

	input = jobs

} else {

	input = nil

}
```

---

`select` automatski ignoriše:

```
nil
```

---

# 8. Select State Machine

Primer:

```go
select {

case job := <-input:

	process(job)

case <-shutdown:

	return

}
```

---

Kasnije:

```go
input = nil
```

---

Rezultat:

```
više se ne primaju poslovi
```

---

Ali:

```
shutdown

ostaje aktivan
```

---

# 9. Dinamička Promena Stanja

Mentalni model:

```
ACTIVE

↓

input = jobs
```

---

Kasnije:

```
PAUSED

↓

input = nil
```

---

Kasnije:

```
ACTIVE

↓

input = jobs
```

---

Bez:

- novih goroutines
- novih channel-a
- dodatnih `if` grana

---

# 10. Channel Ownership u Velikim Sistemima

Zamislimo pipeline:

```
Producer

↓

Stage A

↓

Stage B

↓

Stage C

↓

Consumer
```

---

Svaka faza poseduje:

```
svoj output channel
```

---

# 11. Ownership Pravilo

Svaka faza:

```
kreira

↓

piše

↓

zatvara

svoj izlazni channel
```

---

Nikada:

tuđi.

---

Na primer:

```
Stage A

↓

outA
```

---

Samo:

```
Stage A

close(outA)
```

---

# 12. Ownership Granice

Vizuelno:

```
Stage A

creates

↓

outA

↓

Stage B
```

---

Stage B:

```
read only
```

---

Ne zatvara:

```
outA
```

---

# 13. Production Pipeline

```
Generator

↓

Parser

↓

Validator

↓

Transformer

↓

Storage
```

---

Svaka komponenta:

- ima svoj izlaz
- zatvara svoj izlaz
- čita tuđi izlaz

---

Time se postiže:

jasna odgovornost.

---

# 14. Graceful Shutdown Pipeline-a

Kada:

```
shutdown
```

stigne,

svaka faza:

1. završava trenutni posao

2. zatvara svoj izlaz

3. izlazi

---

Sledeća faza:

dobija:

```
range finished
```

---

I nastavlja isti proces.

---

Shutdown se prirodno propagira kroz ceo pipeline.

---

# 15. Production Primer

```
HTTP

↓

Parser

↓

Business Logic

↓

Database

↓

Response
```

---

Ako parser završi rad:

zatvara:

```
parserOut
```

---

Business Logic:

automatski završava.

---

Nema:

ručne koordinacije.

---

# 16. Najčešće Greške

❌ Jedan globalni channel.

---

❌ Više owner-a nad istim channel-om.

---

❌ Receiver zatvara channel.

---

❌ Dinamičko kreiranje hiljada worker-a bez ograničenja.

---

❌ Ignorisanje shutdown signala.

---

# 17. Production Saveti

✔️ Svaka faza treba da ima jasno definisanu odgovornost.

---

✔️ Svaki channel treba da ima jednog owner-a.

---

✔️ Koristiti `nil` channel za promenu ponašanja `select` izraza.

---

✔️ Dinamički povećavati broj worker-a samo kada postoje stvarni pokazatelji opterećenja.

---

✔️ Graceful shutdown planirati od početka razvoja sistema.

---

# 18. Senior Mentalni Model

Pipeline nije:

```
skup goroutines
```

---

Već:

```
mreža komponenti

koje

poseduju

svoje izlaze.
```

---

Ownership je ono što održava ceo sistem:

- predvidljivim
- bezbednim
- lakim za održavanje

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Dynamic Fan-Out

✅ Dynamic Worker Scaling

✅ Select kao State Machine

✅ Advanced Nil Channel Pattern

✅ Runtime Enable/Disable

✅ Channel Ownership

✅ Pipeline Ownership

✅ Graceful Shutdown Pipeline-a

---

# Advanced Channel Patterns

## Deo #6 — Production Channel Patterns — Kompletan Pregled

---

# 📚 Sadržaj

- Kako birati odgovarajući pattern?
- Kombinovanje pattern-a
- Arhitektura velikih Go sistema
- Najčešće production greške
- Performance smernice
- Channel Pattern Cheat Sheet
- Završni pregled modula

---

# 1. Ne Postoji Univerzalni Pattern

Jedna od najčešćih grešaka je pokušaj da se jedan pattern koristi svuda.

---

Na primer:

```
Worker Pool

↓

za svaki problem
```

---

ili:

```
Channel

↓

za svaku komunikaciju
```

---

Production sistemi koriste:

```
više različitih obrazaca

istovremeno.
```

---

# 2. Kako Izabrati Pattern?

Prvo postavi pitanje:

```
Šta rešavam?
```

---

Ako želiš:

```
ograničiti broj

aktivnih poslova
```

↓

Semaphore

---

Ako želiš:

```
ograničiti brzinu
```

↓

Token Bucket

---

Ako želiš:

```
kopirati stream
```

↓

Tee Pattern

---

Ako želiš:

```
spojiti stream-ove
```

↓

Bridge Pattern

---

Ako želiš:

```
prekid rada
```

↓

Done Channel ili `context.Context`

---

# 3. Pattern-i se Kombinuju

Retko kada se koristi samo jedan obrazac.

---

Primer:

```
HTTP Request

↓

Context

↓

Rate Limiter

↓

Worker Pool

↓

Pipeline

↓

Storage
```

---

Svaka komponenta rešava drugi problem.

---

# 4. Primer Production Arhitekture

```
Incoming Requests

        |

        v

Rate Limiter

        |

        v

Worker Pool

        |

        v

Pipeline

        |

        v

Fan-Out

   /        \

Audit    Metrics

    \      /

     \    /

      Bridge

        |

        v

Database
```

---

Ovde zajedno rade:

- Token Bucket
- Semaphore
- Pipeline
- Tee
- Bridge
- Context

---

# 5. Backpressure Kao Zaštita

Svaki ozbiljan sistem mora imati:

```
backpressure
```

---

Bez njega:

Producer može biti mnogo brži od Consumer-a.

---

Rezultat:

- rast memorije
- ogromni baferi
- scheduler overload
- GC pritisak

---

Backpressure održava sistem stabilnim.

---

# 6. Graceful Shutdown

Svaka komponenta treba da zna:

```
kada

i

kako

završava rad.
```

---

Tipičan tok:

```
Shutdown Signal

↓

Stop prijema novih poslova

↓

Završi aktivne poslove

↓

Close output channel

↓

Exit
```

---

# 7. Goroutine Leak Checklist

Pre svake nove goroutine zapitaj se:

```
Kako će završiti?
```

---

Ako odgovor ne postoji,

verovatno postoji:

```
goroutine leak
```

---

Svaka goroutine treba da ima:

- prirodan kraj
- cancellation signal
- zatvaranje ulaznog channel-a

---

# 8. Channel Ownership Checklist

Za svaki channel treba znati:

✔️ Ko ga kreira?

✔️ Ko šalje podatke?

✔️ Ko ga zatvara?

✔️ Ko ga samo čita?

---

Ako odgovor nije jasan,

ownership nije dobro definisan.

---

# 9. Kada Koristiti Channel?

Koristiti channel kada postoji:

- razmena poruka
- ownership transfer
- sinhronizacija između goroutines
- pipeline
- event stream

---

Ne koristiti channel samo zato što postoji konkurentnost.

---

# 10. Kada Koristiti Mutex ili Atomic?

Ako štitiš:

```
jednu promenljivu
```

↓

`atomic`

---

Ako štitiš:

```
deljeni mutable state
```

↓

`sync.Mutex`

ili

`sync.RWMutex`

---

Channel nije zamena za svaku sinhronizaciju.

---

# 11. Performance Pravila

✔️ Ne kreirati nepotrebne goroutines.

---

✔️ Ne praviti ogromne buffere bez merenja.

---

✔️ Benchmark pre optimizacije.

---

✔️ Profilisati CPU i memoriju.

---

✔️ Izbegavati nepotrebno kopiranje velikih vrednosti kroz channel.

---

# 12. Najčešće Production Greške

❌ Channel bez owner-a.

---

❌ Receiver zatvara channel.

---

❌ Ignorisanje `context.Context`.

---

❌ Beskonačne goroutine bez izlaza.

---

❌ Globalni channel za sve.

---

❌ Previše složen `select`.

---

❌ Nepotrebni buffered channel-i.

---

# 13. Channel Pattern Cheat Sheet

| Problem | Pattern |
|----------|---------|
| Worker limit | Semaphore |
| Rate limiting | Token Bucket |
| Broadcast shutdown | Done Channel |
| Request cancellation | Context |
| Jedan ulaz → više izlaza | Tee |
| Više stream-ova → jedan | Bridge |
| Privremeno isključivanje grane | Nil Channel |
| Odbacivanje viška zahteva | Drop Pattern |
| Dinamičko uključivanje `select` grane | Nil Channel Pattern |
| Obrada toka podataka | Pipeline |
| Raspodela poslova | Fan-Out |
| Spajanje rezultata | Fan-In |

---

# 14. Mentalni Model

Production Go aplikacija nije skup:

```
goroutines

+

channels
```

---

Već:

```
komponenti

koje

međusobno

komuniciraju
```

---

Channels predstavljaju:

```
granice između komponenti
```

---

# 15. Ključni Principi

Zapamti sledećih deset pravila:

1. Channel ima jednog owner-a.

2. Sender zatvara channel.

3. Svaka goroutine mora imati izlaz.

4. Koristi `context.Context` za javne API-je.

5. Semaphore ograničava konkurentnost.

6. Token Bucket ograničava brzinu.

7. Nil channel omogućava dinamički `select`.

8. Ne koristi channel kada je dovoljan `Mutex`.

9. Meri performanse pre optimizacije.

10. Jednostavan dizajn je skoro uvek bolji od komplikovanog.

---

# 16. Završetak Modula

Završili smo:

```
docs/module-4/extras/

└── 06-advanced-channel-patterns.md
```

---

Obrađene teme:

✅ Channel Ownership

✅ Single Sender Principle

✅ Nil Channel Pattern

✅ Done Channel

✅ Or-Done Pattern

✅ Tee Pattern

✅ Bridge Pattern

✅ Semaphore

✅ Token Bucket

✅ Drop Pattern

✅ Backpressure

✅ Dynamic Fan-Out

✅ Dynamic Worker Scaling

✅ Pipeline Ownership

---

# 🎯 Šta Sada Znaš?

Sada razumeš:

- kako dizajnirati stabilne konkurentne pipeline-ove
- kako sprečiti goroutine leak-ove
- kako ograničiti konkurentnost i brzinu obrade
- kako pravilno organizovati ownership nad channel-ima
- kako kombinovati concurrency obrasce u production sistemima
- kako koristiti napredne obrasce koji se sreću u velikim Go projektima

---

### ➡️ Sledeća lekcija **[**Context Advanced Patterns**](07-context-advanced-patterns.md)**

Obuhvatiće:

- Context Internals
- Cancellation Tree
- Deadline i Timeout mehanizmi
- Context Values
- Context propagation
- Context u HTTP serverima
- Context u bazama podataka
- Anti-patterns i production best practices
```