# Timeouts — Kontrola vremena izvršavanja u Go Concurrency modelu

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 1)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Šta je timeout
- Zašto su timeout-i važni u konkurentnim sistemima
- Problem beskonačnog čekanja
- Goroutines bez kontrole
- Razlika između timeout-a i običnog čekanja
- Timeout kao zaštita sistema
- Prvi timeout primer
- Mentalni model

---

# Uvod

U prethodnim lekcijama učili smo:

- `Goroutines`
- `Channels`
- `WaitGroup`
- `Mutex`
- `sync.Once`

---

Svi ovi alati rešavaju različite concurrency probleme.

Ali postoji jedno veoma važno pitanje:

> Šta se dešava ako neka operacija nikada ne završi?

---

Primer:

```go
result := fetchData()
```

---

Šta ako:

- server ne odgovara,
- mreža je prekinuta,
- database je zaglavljena,
- Goroutine čeka zauvek?

---

Program može ostati blokiran.

---

# Šta je timeout?

Timeout znači:

> Definišemo maksimalno vreme koliko smo spremni da čekamo da se neka operacija završi.

---

Primer:

```text
Čekam odgovor maksimalno 3 sekunde.

Ako nema odgovora:

prekidam čekanje.
```

---

Vizuelno:

```
START
 |
 |
 |---- operacija
 |
 |
3 sekunde
 |
 |
TIMEOUT
```

---

# Zašto su timeout-i važni?

U distribuiranim sistemima:

```text
nijedna komponenta nije 100% pouzdana
```

---

Aplikacija zavisi od:

- HTTP servisa,
- database sistema,
- message queue-a,
- cache sistema.

---

Svaka od tih komponenti može:

- kasniti,
- biti nedostupna,
- prestati da odgovara.

---

Bez timeout-a:

```text
jedan problem

↓

jedna zaglavljena Goroutine

↓

mnogo zaglavljenih Goroutines

↓

pad sistema
```

---

# Primer problema bez timeout-a

Pogledajmo:

```go
func requestData() {

	result := make(chan string)

	go func(){

		result <- "data"

	}()

	data := <-result

	fmt.Println(data)

}
```

---

Na prvi pogled:

radi.

---

Ali šta ako Goroutine nikada ne pošalje podatke?

---

Primer:

```go
go func(){

	// nikada nema send

}()
```

---

Glavna Goroutine:

```go
data := <-result
```

ostaje zauvek blokirana.

---

Stanje:

```text
Waiting forever
```

---

# Problem u realnom svetu

Primer HTTP poziva:

```go
response, err := http.Get(url)
```

---

Šta ako server:

```text
primi zahtev

ali nikada ne pošalje odgovor
```

---

Rezultat:

```text
Goroutine čeka
```

---

Ako imamo:

```text
1 zahtev

nije problem
```

---

Ali:

```text
10000 zahteva
```

---

Dobijamo:

```text
10000 blokiranih Goroutines
```

---

# Goroutines nisu besplatne

Česta greška:

> "Go ima lagane Goroutines, mogu ih napraviti milion."

---

Tačno je:

Goroutines su jeftinije od OS thread-ova.

---

Ali:

one i dalje troše:

- memoriju,
- scheduler resurse,
- channel strukture,
- stack prostor.

---

Zato:

```text
moramo imati kontrolu njihovog životnog ciklusa
```

---

# Timeout kao zaštitni mehanizam

Timeout omogućava:

```text
operacija mora završiti u određenom vremenu
```

---

Ako ne završi:

```text
prekidamo čekanje
```

---

Primer:

```text
API Gateway

timeout = 2s


Service A

timeout = 5s


Database

timeout = 10s
```

---

Svaki sloj ima kontrolu.

---

# Timeout nije isto što i prekid

Veoma važna razlika.

---

Timeout:

```text
prestajem da čekam
```

---

Ne znači automatski:

```text
ubij Goroutine
```

---

Primer:

```go
select {

case result := <-channel:

	fmt.Println(result)


case <-time.After(time.Second):

	fmt.Println("timeout")

}
```

---

Ako timeout nastupi:

glavna Goroutine izlazi.

---

Ali druga Goroutine možda i dalje radi.

---

Zato kasnije učimo:

- `context cancellation`
- cooperative cancellation

---

# Prvi timeout primer

Koristimo:

```go
select
```

---

Kod:

```go
package main

import (
	"fmt"
	"time"
)


func main(){

	ch := make(chan string)


	go func(){

		time.Sleep(3 * time.Second)

		ch <- "result"

	}()


	select {

	case result := <-ch:

		fmt.Println(result)


	case <-time.After(1 * time.Second):

		fmt.Println("timeout")

	}

}
```

---

Šta se dešava?

---

Goroutine:

```go
Sleep(3s)
```

---

Main čeka:

```go
1s
```

---

Posle 1 sekunde:

```text
timeout
```

---

# Problem ovog pristupa

Koristili smo:

```go
time.After()
```

---

To je dobro za male slučajeve.

---

Ali u velikim sistemima:

može biti problematično ako se često kreira.

---

Za ozbiljne sisteme koristimo:

```go
context
```

---

To ćemo detaljno obraditi u sledećim delovima.

---

# Timeout mentalni model

Zapamti:

```text
Normalno čekanje:

čekam koliko god treba


Timeout:

čekam do granice

ako ne završi:

prekidam čekanje
```

---

# Timeout u concurrency arhitekturi

Dobar sistem ima granice:

```
Client

 |
 | timeout 2s
 |
API Server

 |
 | timeout 5s
 |
Service

 |
 | timeout 10s
 |
Database
```

---

Svaki sloj štiti sledeći.

---

# Česte greške početnika

---

## Greška 1

Čekanje bez ograničenja:

```go
<-channel
```

bez kontrole.

---

## Greška 2

Kreiranje Goroutines bez plana:

```go
go process()
```

bez:

- timeout-a,
- cancellation-a,
- shutdown mehanizma.

---

## Greška 3

Misliti:

```text
timeout = kill goroutine
```

---

Nije.

---

Timeout samo kontroliše:

```text
koliko dugo čekamo
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je timeout,
- zašto je važan u concurrency sistemima,
- problem beskonačnog čekanja,
- kako nastaje Goroutine leak,
- razliku između timeout-a i prekida,
- prvi timeout obrazac pomoću `select`,
- zašto se u produkciji koristi `context`.

---

# Timeouts — `context.WithTimeout` i osnovni timeout obrasci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 2)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Zašto koristiti `context` umesto `time.After`
- Uvod u `context.WithTimeout`
- Kako radi timeout context
- `cancel()` funkcija
- Timeout propagation
- Osnovni timeout pattern
- Kontrola više Goroutines pomoću context-a
- Česte greške

---

# Uvod

U prethodnom delu videli smo:

```go
select {

case result := <-ch:

case <-time.After(time.Second):

}
```

---

Ovo je koristan obrazac.

Ali u ozbiljnim sistemima često imamo:

- više funkcija,
- više Goroutines,
- više nivoa poziva.

---

Primer:

```
HTTP Request

    |
    |
Service

    |
    |
Repository

    |
    |
Database
```

---

Pitanje:

Kako proslediti informaciju:

> "Ovaj zahtev više nije validan, prekini rad"

kroz ceo lanac?

---

Odgovor:

```go
context
```

---

# Zašto `context.WithTimeout`?

`context` omogućava:

- timeout,
- cancellation,
- deadline,
- prenošenje request-scoped podataka.

---

Najčešće:

```go
context.WithTimeout()
```

---

Njegova ideja:

```
kreiraj context

↓

postavi vremensko ograničenje

↓

prosledi kroz funkcije

↓

svako može proveriti da li je istekao
```

---

# Osnovni API

Funkcija:

```go
context.WithTimeout()
```

vraća:

```go
(ctx, cancel)
```

---

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	2*time.Second,
)

defer cancel()
```

---

Imamo:

## Context

```go
ctx
```

koji nosi:

- deadline,
- cancellation signal.

---

## Cancel funkciju

```go
cancel()
```

koja ručno prekida context.

---

# Prvi primer

```go
package main

import (
	"context"
	"fmt"
	"time"
)


func main(){

	ctx, cancel := context.WithTimeout(
		context.Background(),
		2*time.Second,
	)

	defer cancel()


	select {

	case <-time.After(5 * time.Second):

		fmt.Println("operation finished")


	case <-ctx.Done():

		fmt.Println("timeout")

	}

}
```

---

Rezultat:

```text
timeout
```

---

Zašto?

Imamo:

```go
operation = 5s
```

---

A:

```go
timeout = 2s
```

---

Context završava prvi.

---

# `ctx.Done()`

Najvažniji deo.

---

`Done()` vraća:

```go
<-chan struct{}
```

---

To je channel koji se zatvara kada:

- timeout istekne,
- neko pozove `cancel()`.

---

Primer:

```go
select {

case <-ctx.Done():

	fmt.Println("cancelled")

}
```

---

Kada se context završi:

```text
Done channel se zatvara
```

---

# Zašto zatvaranje channel-a?

Zato što više Goroutines može čekati isti signal.

---

Primer:

```
          ctx.Done()

              |
     -----------------

     |       |       |

    G1      G2      G3
```

---

Jedan signal:

```text
sve Goroutines dobijaju obaveštenje
```

---

# Cancel funkcija

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	time.Second,
)


cancel()
```

---

Ovde ne čekamo timeout.

---

Mi ručno kažemo:

```text
prekini
```

---

Primer:

```
User klikne Cancel

        |

        v

cancel()

        |

        v

Goroutines prekidaju rad
```

---

# Zašto koristiti `defer cancel()`?

Standardni obrazac:

```go
ctx, cancel := context.WithTimeout(...)

defer cancel()
```

---

Zašto?

Zato što oslobađa resurse.

---

Ako operacija završi ranije:

```text
nema potrebe čekati timeout
```

---

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	10*time.Second,
)

defer cancel()


result := process(ctx)
```

---

Ako:

```go
process()
```

završi za:

```text
100ms
```

---

pozivamo:

```go
cancel()
```

pre isteka 10 sekundi.

---

# Timeout propagation

Jedna od najvažnijih osobina.

---

Primer:

```go
func Handler(ctx context.Context){

	service(ctx)

}
```

---

Dalje:

```go
func Service(ctx context.Context){

	repository(ctx)

}
```

---

Dalje:

```go
func Repository(ctx context.Context){

	database(ctx)

}
```

---

Isti context putuje kroz ceo sistem.

---

Vizuelno:

```
Request Context

        |
        v

Handler

        |
        v

Service

        |
        v

Database
```

---

Ako timeout istekne:

svi dobijaju signal.

---

# Primer sa Goroutine

```go
func worker(ctx context.Context){

	for {

		select {

		case <-ctx.Done():

			fmt.Println("worker stopped")

			return


		default:

			fmt.Println("working")

			time.Sleep(500*time.Millisecond)

		}

	}

}
```

---

Pokretanje:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	2*time.Second,
)

defer cancel()


go worker(ctx)
```

---

Posle 2 sekunde:

```
ctx.Done()

↓

worker return
```

---

# Važna stvar

Context ne ubija Goroutine.

---

On samo šalje signal:

```text
"treba da završiš"
```

---

Goroutine mora sama:

```go
select {

case <-ctx.Done():

	return

}
```

---

Ovo se zove:

```text
cooperative cancellation
```

---

# Timeout obrazac u produkciji

Tipičan kod:

```go
func GetUser(ctx context.Context,id int)(User,error){

	select {

	case <-ctx.Done():

		return User{},ctx.Err()


	case result := <-database:

		return result,nil

	}

}
```

---

Prednosti:

- nema beskonačnog čekanja,
- propagira se cancellation,
- svaki sloj poštuje timeout.

---

# `ctx.Err()`

Kada se context završi:

možemo dobiti razlog.

---

Primer:

```go
err := ctx.Err()
```

---

Moguće vrednosti:

```go
context.DeadlineExceeded
```

ili:

```go
context.Canceled
```

---

Primer:

```go
if err == context.DeadlineExceeded {

	fmt.Println("timeout")

}
```

---

# Česte greške

---

## Greška 1

Kreirati context unutra:

```go
func Service(){

	ctx := context.Background()

}
```

---

Loše.

---

Bolje:

```go
func Service(ctx context.Context)
```

---

---

## Greška 2

Ignorisati `ctx.Done()`

Primer:

```go
for {

	process()

}
```

---

Goroutine neće stati.

---

## Greška 3

Ne pozvati:

```go
cancel()
```

---

Može doći do nepotrebnog držanja resursa.

---

# Mentalni model

Zapamti:

```
context.WithTimeout

=

kreiraj vremensko ograničenje

+

pošalji signal svim zainteresovanim Goroutines
```

---

Ne:

```
kill thread
```

---

Ne:

```
force stop
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto je `context.WithTimeout` bolji za veće sisteme,
- kako se kreira timeout context,
- ulogu `cancel()` funkcije,
- kako radi `ctx.Done()`,
- timeout propagation,
- kontrolu više Goroutines,
- cooperative cancellation koncept,
- razliku između timeout signala i nasilnog prekida.

---

# Timeouts — `context.WithDeadline` i razlika između Timeout i Deadline

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 3)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Šta je Deadline
- `context.WithDeadline`
- Razlika između Timeout i Deadline
- Relativno vs apsolutno vreme
- Kada koristiti `WithTimeout`
- Kada koristiti `WithDeadline`
- Kombinovanje više context-a
- Praktični primeri

---

# Uvod

U prethodnom delu naučili smo:

```go
context.WithTimeout()
```

koji definiše:

> koliko dugo smo spremni da čekamo od trenutka kreiranja context-a.

---

Primer:

```go
ctx, cancel := context.WithTimeout(
	context.Background(),
	5*time.Second,
)
```

---

Značenje:

```
sada

+

5 sekundi

=

deadline
```

---

Ali ponekad ne razmišljamo:

```text
koliko dugo čekati
```

nego:

```text
do kog trenutka mora biti završeno
```

---

Tu dolazi:

```go
context.WithDeadline()
```

---

# Šta je Deadline?

Deadline znači:

> Apsolutna vremenska granica do kada operacija mora završiti.

---

Primer:

```
Sistem mora odgovoriti

do:

14:30:00
```

---

Nije bitno kada je zahtev počeo.

Bitno je:

```text
krajnji trenutak
```

---

# Osnovni API

```go
context.WithDeadline(
	parent,
	time.Time,
)
```

---

Primer:

```go
deadline := time.Now().Add(
	5*time.Second,
)


ctx, cancel := context.WithDeadline(
	context.Background(),
	deadline,
)

defer cancel()
```

---

Ovo je praktično slično:

```go
WithTimeout(5*time.Second)
```

---

Ali konceptualno različito.

---

# Timeout vs Deadline

Najvažnija razlika:

---

## Timeout

Pita:

> Koliko dugo čekam od sada?

---

Primer:

```go
WithTimeout(
	5*time.Second,
)
```

---

Značenje:

```
START

|

|------ 5s ------|

TIMEOUT
```

---

## Deadline

Pita:

> Do kog trenutka mora završiti?

---

Primer:

```go
WithDeadline(
	14:30:00,
)
```

---

Značenje:

```
START

        ...

        14:30:00

           |
           |
       DEADLINE
```

---

# Primer poređenja

Pretpostavimo:

Trenutno vreme:

```text
14:00:00
```

---

Kreiramo:

```go
WithTimeout(10*time.Minute)
```

---

Dobijamo:

```text
istek u 14:10:00
```

---

Sada:

```text
operacija kreirana u 14:05
```

---

Novi timeout:

```text
14:15
```

---

---

Ali Deadline:

```go
WithDeadline(
	14:10:00,
)
```

---

uvek znači:

```text
završiti do 14:10
```

bez obzira kada je kreiran.

---

# Zašto je Deadline koristan?

U velikim sistemima često imamo:

```
Client

deadline: 10s


API Gateway

5s preostalo


Service

3s preostalo


Database

1s preostalo
```

---

Svaki servis zna:

```text
koliko vremena je ostalo
```

---

Ne kreira novi timeout.

---

# Primer: distribuirani sistem

Klijent šalje zahtev:

```text
request start:

12:00:00
```

---

Postavlja:

```text
deadline:

12:00:05
```

---

Servis A primi:

```text
12:00:02
```

---

Ima:

```text
3 sekunde
```

---

Servis B primi:

```text
12:00:04
```

---

Ima:

```text
1 sekundu
```

---

Ovo je mnogo preciznije nego:

```
svaki servis svoj timeout 5s
```

---

# Dobijanje Deadline vrednosti

Context ima:

```go
Deadline()
```

---

Primer:

```go
deadline, ok := ctx.Deadline()


if ok {

	fmt.Println(deadline)

}
```

---

Rezultat:

```text
vreme kada context ističe
```

---

# Provera preostalog vremena

Možemo izračunati:

```go
remaining := time.Until(deadline)
```

---

Primer:

```go
deadline, _ := ctx.Deadline()

fmt.Println(
	time.Until(deadline),
)
```

---

Dobijamo:

```text
koliko vremena je ostalo
```

---

# Kombinovanje Timeout i Deadline

Primer:

Imamo parent context:

```go
deadline:
10 sekundi
```

---

Child želi:

```go
WithTimeout(
	30*time.Second,
)
```

---

Šta se dešava?

---

Child ne može živeti duže od parent-a.

---

Rezultat:

```
Parent:

10s


Child:

maksimalno 10s
```

---

# Context hijerarhija

Važno pravilo:

```
Parent context

        |
        |
        v

Child context
```

---

Child može imati:

- kraći rok,
- dodatni cancellation.

---

Ali ne može:

```text
produžiti parent rok
```

---

# Primer

```go
parent, cancel :=
	context.WithTimeout(
		context.Background(),
		10*time.Second,
	)

defer cancel()



child, cancelChild :=
	context.WithTimeout(
		parent,
		30*time.Second,
	)

defer cancelChild()
```

---

Iako piše:

```go
30*time.Second
```

---

Realni limit:

```text
10 sekundi
```

---

# Kada koristiti `WithTimeout`?

Najčešći slučajevi:

---

## HTTP request

```text
čekaj maksimalno 3s
```

---

## Database query

```text
čekaj maksimalno 5s
```

---

## RPC poziv

```text
čekaj maksimalno 2s
```

---

Kod:

```go
context.WithTimeout()
```

je prirodniji.

---

# Kada koristiti `WithDeadline`?

---

## 1. Job scheduling

Primer:

```
backup mora završiti do 03:00
```

---

## 2. Batch processing

Primer:

```
obrada završava do kraja radnog vremena
```

---

## 3. Distributed systems

Kada jedan zahtev nosi krajnji rok.

---

## 4. Request budget

Primer:

```
ukupno 10 sekundi

potrošeno 7

ostalo 3
```

---

# Česte greške

---

## Greška 1

Kreiranje novih timeout-a u svakom sloju.

Loše:

```go
Service()

WithTimeout(5s)

Repository()

WithTimeout(5s)
```

---

Problem:

ukupno vreme može postati:

```text
10s

15s

20s
```

---

Bolje:

prosledi isti context.

---

# Greška 2

Ignorisanje parent deadline-a.

---

Primer:

```go
child := context.Background()
```

---

Time gubimo:

- timeout,
- cancellation.

---

# Greška 3

Misliti da Deadline izvršava operaciju.

---

Ne.

On samo daje signal:

```go
ctx.Done()
```

---

# Mentalni model

Zapamti:

```
WithTimeout

=
koliko dugo od sada


WithDeadline

=
do kog trenutka
```

---

Još kraće:

```
Timeout

duration


Deadline

timestamp
```

---

# 📋 Rezime

U ovom delu naučili smo:

- šta je Deadline,
- kako radi `context.WithDeadline`,
- razliku između Timeout i Deadline,
- relativno i apsolutno vreme,
- kako se context hijerarhija ponaša,
- zašto child context ne može produžiti parent deadline,
- kada koristiti Timeout, a kada Deadline.

---

# Timeouts — HTTP timeout strategije u Go aplikacijama

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 4)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Zašto HTTP pozivi zahtevaju timeout
- Problem podrazumevanog HTTP ponašanja
- `http.Client.Timeout`
- Request context timeout
- HTTP server timeout-i
- Timeout po fazama HTTP komunikacije
- Production timeout strategije

---

# Uvod

U modernim aplikacijama veliki deo komunikacije izgleda ovako:

```
Client

    |
    |
 HTTP Request

    |
    |
 Server
```

---

Jedan servis poziva drugi:

```
API Gateway

      |
      v

User Service

      |
      v

Database Service
```

---

Svaki mrežni poziv predstavlja potencijalno čekanje.

---

Pitanje:

> Koliko dugo smo spremni da čekamo odgovor?

---

Odgovor:

```text
timeout politika
```

---

# Problem bez HTTP timeout-a

Primer:

```go
response, err := http.Get(url)
```

---

Mnogi početnici misle:

```text
ako server ne odgovara

dobijamo error
```

---

Ali nije uvek tako.

---

Server može:

- prihvatiti konekciju,
- otvoriti socket,
- nikada poslati odgovor.

---

Rezultat:

```
Goroutine čeka
```

---

Ako imamo:

```
10 zahteva

=
10 blokiranih Goroutines
```

---

Ako imamo:

```
100000 zahteva

=
problem sa resursima
```

---

# `http.Client.Timeout`

Najjednostavniji način.

---

Primer:

```go
client := &http.Client{

	Timeout: 5 * time.Second,

}
```

---

Korišćenje:

```go
response, err := client.Get(url)
```

---

Značenje:

Ceo HTTP poziv ima limit:

```
5 sekundi
```

---

Obuhvata:

- DNS lookup,
- TCP connection,
- TLS handshake,
- request slanje,
- response čekanje,
- čitanje body-ja.

---

# Primer

```go
package main

import (
	"net/http"
	"time"
)


func main(){

	client := &http.Client{

		Timeout: 3 * time.Second,

	}


	_, err := client.Get(
		"https://example.com",
	)


	if err != nil {

		panic(err)

	}

}
```

---

Ako odgovor traje:

```
> 3s
```

---

dobijamo:

```text
timeout error
```

---

# Prednost `http.Client.Timeout`

Veoma jednostavno:

```go
jedna konfiguracija
```

---

Dobro za:

- CLI alate,
- jednostavne servise,
- interne pozive.

---

Ali u većim sistemima često želimo:

```text
kontrolu po request-u
```

---

Tu koristimo:

```go
context
```

---

# Request context timeout

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		2*time.Second,
	)

defer cancel()
```

---

HTTP request:

```go
req, err :=
	http.NewRequestWithContext(
		ctx,
		http.MethodGet,
		url,
		nil,
	)
```

---

Poziv:

```go
resp, err :=
	client.Do(req)
```

---

Sada request poštuje:

```go
ctx.Done()
```

---

# Zašto je context bolji?

Zato što možemo imati:

```
User Request

      |
      |
      v

Service A

      |
      |
      v

HTTP Call

      |
      |
      v

Database
```

---

Isti context ide svuda.

---

Primer:

```go
func Handler(
	ctx context.Context,
){

	callService(ctx)

}
```

---

Service:

```go
func callService(
	ctx context.Context,
){

	request.WithContext(ctx)

}
```

---

Ako korisnik otkaže zahtev:

sve se prekida.

---

# `http.Client.Timeout` vs Context timeout

Važna razlika.

---

## Client Timeout

Globalna politika:

```go
client.Timeout = 5s
```

---

Svaki zahtev:

```
maksimalno 5s
```

---

---

## Context Timeout

Request-specifično:

```go
context.WithTimeout(
	ctx,
	2*time.Second,
)
```

---

Možemo imati:

```
Upload:

60s


Search:

2s


Health check:

1s
```

---

# HTTP server timeout-i

Nije dovoljno zaštititi samo klijenta.

---

Server takođe mora imati limite.

---

Problem:

Spor klijent:

```
connect

send slowly

never finish
```

---

Poznato kao:

```text
Slowloris attack
```

---

Rešenje:

server timeout-i.

---

# `http.Server`

Primer:

```go
server := &http.Server{

	ReadTimeout:
	5*time.Second,


	WriteTimeout:
	10*time.Second,


	IdleTimeout:
	60*time.Second,

}
```

---

---

# ReadTimeout

Koliko dugo server čeka:

```
klijent šalje request
```

---

Štiti od:

- sporih klijenata,
- zaglavljenih konekcija.

---

# WriteTimeout

Koliko dugo server može slati odgovor.

---

Primer:

```
server generiše response

klijent ne čita
```

---

Bez timeout-a:

```text
connection ostaje otvorena
```

---

# IdleTimeout

Koliko dugo keep-alive konekcija može biti neaktivna.

---

Primer:

```
HTTP connection

čekanje sledećeg request-a
```

---

# HTTP timeout slojevi

Production sistem često ima:

```
Client

timeout 3s

        |

API Gateway

timeout 5s

        |

Service

timeout 4s

        |

Database

timeout 2s
```

---

Pravilo:

```
niži sloj mora imati kraći ili jednak timeout
```

---

# Primer dobre arhitekture

```
External Request

deadline: 10s


Service A

ostalo: 8s


Service B

ostalo: 5s


Database

ostalo: 2s
```

---

Context omogućava:

```text
jedan rok kroz ceo sistem
```

---

# Česte greške

---

## Greška 1

Koristiti:

```go
http.Get()
```

u production kodu.

---

Problem:

nema kontrolisan timeout.

---

Bolje:

```go
http.Client{
	Timeout: ...
}
```

---

## Greška 2

Svaki servis pravi novi timeout.

---

Loše:

```go
WithTimeout(5s)
```

svuda.

---

Bolje:

proslediti postojeći context.

---

## Greška 3

Predugački timeout-i.

---

Primer:

```text
5 minuta
```

---

Može napraviti:

- queue problema,
- gomilanje Goroutines,
- iscrpljivanje resursa.

---

# Production smernice

---

## Client

Uvek:

```go
Timeout
```

ili:

```go
Context
```

---

## Server

Konfigurisati:

```go
ReadTimeout

WriteTimeout

IdleTimeout
```

---

## Service pozivi

Uvek:

```go
context.Context
```

---

## Database

Uvek:

```go
context.WithTimeout
```

---

# Mentalni model

Zapamti:

```
HTTP timeout

=

zaštita resursa

+

ograničenje čekanja

+

kontrola propagacije
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto HTTP pozivi moraju imati timeout,
- problem `http.Get()` bez kontrole,
- `http.Client.Timeout`,
- request timeout pomoću context-a,
- server timeout konfiguraciju,
- razliku između client i context timeout-a,
- production timeout strategije.

---

# Timeouts — Database operacije, Goroutine kontrola i sprečavanje curenja resursa

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 5)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Zašto database operacije zahtevaju timeout
- `context` u database pozivima
- Timeout u SQL upitima
- Kontrola Goroutines pomoću context-a
- Goroutine leak problem
- Timeout + Channel pattern
- Timeout + Worker pattern
- Best practices

---

# Uvod

Do sada smo videli timeout u:

- običnim funkcijama,
- HTTP komunikaciji.

---

Ali najveći broj production problema nastaje kod:

- database poziva,
- background job-ova,
- worker sistema,
- asinhronih procesa.

---

Tipičan scenario:

```
Request

   |
   v

Service

   |
   v

Database query

   |
   v

Čekanje...
```

---

Pitanje:

> Šta ako database nikada ne odgovori?

---

# Database operacije bez timeout-a

Primer:

```go
rows, err := db.Query(
	"SELECT * FROM users",
)
```

---

Problem:

Ako query traje beskonačno:

```
Goroutine čeka
```

---

Posledice:

- zauzeti connection pool,
- blokirani request-i,
- gomilanje memorije.

---

# Database timeout pomoću context-a

Standardni Go database API podržava:

```go
context.Context
```

---

Primer:

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		2*time.Second,
	)

defer cancel()


rows, err :=
	db.QueryContext(
		ctx,
		"SELECT * FROM users",
	)
```

---

Ako query traje:

```
> 2s
```

---

Context se završava:

```go
ctx.Done()
```

---

Database driver može:

- prekinuti query,
- osloboditi konekciju.

---

# Zašto je ovo važno?

Bez timeout-a:

```
Request

↓

Database

↓

čekanje

↓

connection zauzet
```

---

Sa timeout-om:

```
Request

↓

Database

↓

2s

↓

cancel

↓

oslobađanje resursa
```

---

# Transaction timeout

Isto važi za transakcije.

---

Primer:

```go
tx, err :=
	db.BeginTx(
		ctx,
		nil,
	)
```

---

Context kontroliše:

- početak transakcije,
- query-e,
- commit,
- rollback.

---

# Timeout i Goroutine kontrola

Čest obrazac:

```go
go process()
```

---

Problem:

Ko odlučuje kada:

```text
process()
```

treba da završi?

---

Odgovor:

```go
context
```

---

# Goroutine bez kontrole

Loš primer:

```go
func startWorker(){

	go func(){

		for {

			doWork()

		}

	}()

}
```

---

Problem:

Ova Goroutine:

- nema stop signal,
- radi zauvek,
- teško se gasi.

---

Ovo je:

```text
Goroutine leak
```

---

# Worker sa context kontrolom

Bolji primer:

```go
func worker(
	ctx context.Context,
){

	for {

		select {

		case <-ctx.Done():

			return


		default:

			doWork()

		}

	}

}
```

---

Sada imamo:

```
start

 |

worker

 |

ctx cancel

 |

return
```

---

# Primer timeout worker-a

```go
ctx, cancel :=
	context.WithTimeout(
		context.Background(),
		5*time.Second,
	)

defer cancel()


go worker(ctx)
```

---

Posle:

```
5 sekundi
```

---

worker dobija:

```go
ctx.Done()
```

---

i završava.

---

# Timeout + Channel pattern

Čest problem:

Imamo operaciju:

```go
result := make(chan Result)
```

---

Worker:

```go
go func(){

	result <- calculate()

}()
```

---

Glavna Goroutine:

```go
select {

case r := <-result:

	fmt.Println(r)


case <-ctx.Done():

	fmt.Println("timeout")

}
```

---

Rezultat:

Ako:

```text
calculate završi
```

dobijamo rezultat.

---

Ako:

```text
timeout
```

prekidamo čekanje.

---

# Ali postoji problem

Šta ako worker pokušava:

```go
result <- value
```

nakon timeout-a?

---

Primer:

```
Main

timeout

return


Worker

send result
```

---

Ako channel nema primaoca:

worker može blokirati.

---

# Rešenje 1 — Buffered channel

Koristimo:

```go
result :=
	make(chan Result,1)
```

---

Sada:

worker može poslati:

```go
result <- value
```

---

čak i ako niko više ne čita.

---

# Rešenje 2 — Context proveravanje

Primer:

```go
go func(){

	select {

	case result <- calculate():

	case <-ctx.Done():

		return

	}

}()
```

---

Worker proverava:

```
da li još treba raditi
```

---

# Timeout + Worker Pool

Kod worker pool-a:

```
Jobs

 |

Workers

 |

Results
```

---

Svaki posao može imati:

```go
context
```

---

Primer:

```go
func worker(
	ctx context.Context,
	jobs <-chan Job,
){

	for {

		select {

		case <-ctx.Done():

			return


		case job := <-jobs:

			process(job)

		}

	}

}
```

---

Ako sistem dobije shutdown:

svi worker-i završavaju.

---

# Timeout i Goroutine leak

Najčešći uzroci:

---

## 1. Goroutine čeka channel

Primer:

```go
value := <-ch
```

---

Ako niko nikada ne pošalje:

blokada zauvek.

---

Rešenje:

```go
select {

case value := <-ch:

case <-ctx.Done():

}
```

---

---

## 2. Beskonačna petlja

Loše:

```go
for {

	process()

}
```

---

Bolje:

```go
for {

	select {

	case <-ctx.Done():

		return

	default:

		process()

	}

}
```

---

---

## 3. Background task bez shutdown-a

Loše:

```go
go cleanup()
```

---

Bez:

- context-a,
- stop channel-a.

---

# Production pravila

---

## Pravilo 1

Svaka duga operacija treba imati granicu.

Primer:

- HTTP,
- SQL,
- RPC,
- file processing.

---

## Pravilo 2

Svaka dugoživeća Goroutine treba imati:

```go
ctx.Done()
```

ili:

```go
stop channel
```

---

## Pravilo 3

Ne ostavljati:

```go
go func(){}
```

bez plana za završetak.

---

## Pravilo 4

Timeout nije dovoljan.

Potrebno je:

```text
timeout

+

cancellation

+

cleanup
```

---

# Mentalni model

Zapamti:

```
Timeout

=

signal da vreme ističe


Context

=

mehanizam za prenos tog signala
```

---

# 📋 Rezime

U ovom delu naučili smo:

- zašto database pozivi zahtevaju timeout,
- `QueryContext`,
- transaction timeout,
- kontrolu Goroutines pomoću context-a,
- Goroutine leak problem,
- timeout + channel pattern,
- timeout + worker pattern,
- production pravila za bezbedan concurrency.

---

# Timeouts — Napredni pattern-i, best practices i praktični zadaci

> **Modul:** #3 — Srednje  
>
> **Lekcija:** 4/9 (Deo 6)  
>
> **Fajl:** `docs/module-3/04-timeouts.md`

---

# 📚 Sadržaj ovog dela

- Timeout budgeting
- Layered timeout strategija
- Graceful timeout handling
- Retry + Timeout kombinacija
- Timeout anti-pattern-i
- Production checklist
- Pitanja za proveru znanja
- Praktični zadaci

---

# Timeout budgeting

U realnim sistemima timeout nije samo:

```text
koliko čekam?
```

---

Već:

```text
koliko ukupnog vremena imam
```

---

Primer:

Korisnički zahtev:

```
Maximum latency:

10 sekundi
```

---

Imamo:

```
API Gateway

2s


Service A

3s


Service B

3s


Database

2s
```

---

Ukupno:

```
10s
```

---

Ovo nazivamo:

```text
timeout budget
```

---

# Zašto je timeout budget važan?

Bez njega:

```
Service A

timeout 10s


Service B

timeout 10s


Database

timeout 10s
```

---

Najgori slučaj:

```
30 sekundi
```

---

A korisnik očekuje:

```
10 sekundi
```

---

Sistem postaje spor.

---

# Pravilo propagacije timeout-a

Najbolji pristup:

```
jedan parent context

↓

svi child pozivi
```

---

Primer:

```go
func Handler(
	ctx context.Context,
){

	serviceA(ctx)

}
```

---

Service:

```go
func ServiceA(
	ctx context.Context,
){

	serviceB(ctx)

}
```

---

Database:

```go
func Repository(
	ctx context.Context,
){

	query(ctx)

}
```

---

Svi dele isti rok.

---

# Remaining time pattern

Napredniji obrazac:

Ne pitamo:

```text
koliko je moj timeout?
```

---

Već:

```text
koliko vremena je ostalo?
```

---

Primer:

```go
deadline, ok :=
	ctx.Deadline()


if ok {

	remaining :=
		time.Until(deadline)

	fmt.Println(
		remaining,
	)

}
```

---

Dobijamo:

```
preostali budget
```

---

# Graceful timeout handling

Timeout nije samo:

```go
return error
```

---

Dobar sistem radi:

1. prekida rad,
2. oslobađa resurse,
3. vraća smislen error,
4. beleži razlog.

---

Primer:

```go
if err := ctx.Err(); err != nil {

	switch err {

	case context.DeadlineExceeded:

		log.Println(
			"timeout",
		)


	case context.Canceled:

		log.Println(
			"cancelled",
		)

	}

}
```

---

# Razlika: Timeout vs Cancellation

Veoma važno.

---

## Timeout

Sistem kaže:

```
vreme je isteklo
```

---

Primer:

```go
context.DeadlineExceeded
```

---

## Cancellation

Neko eksplicitno kaže:

```
prekini
```

---

Primer:

```go
context.Canceled
```

---

Izvor može biti:

- korisnik,
- shutdown sistema,
- parent request.

---

# Retry + Timeout

Čest production obrazac.

---

Primer:

HTTP poziv:

```
timeout:

2s
```

---

Retry:

```
3 pokušaja
```

---

Problem:

Ako svaki pokušaj ima 2s:

```
3 x 2s

=

6 sekundi
```

---

A možda imamo:

```
ukupno 5 sekundi
```

---

Zato retry mora koristiti:

```go
isti context
```

---

Primer:

```go
func retry(
	ctx context.Context,
){

	for i:=0;i<3;i++{

		err :=
			call(ctx)


		if err == nil {

			return nil

		}

	}

	return err

}
```

---

Ako deadline istekne:

svi retry pokušaji staju.

---

# Timeout + Circuit Breaker koncept

U velikim sistemima često imamo:

```
Timeout

+

Retry

+

Circuit Breaker
```

---

Primer:

Service B ne radi.

---

Bez zaštite:

```
10000 request-a

↓

10000 timeout-a

↓

sistem pada
```

---

Sa zaštitom:

```
timeout

↓

retry

↓

circuit open
```

---

---

# Timeout anti-pattern-i

---

# ❌ Anti-pattern 1

Beskonačno čekanje

```go
result := <-ch
```

---

Problem:

nema granice.

---

Bolje:

```go
select {

case result := <-ch:

case <-ctx.Done():

}
```

---

# ❌ Anti-pattern 2

Globalni timeout svuda

Primer:

```go
timeout = 30s
```

za sve.

---

Problem:

različite operacije imaju različite potrebe.

---

Bolje:

```
Search:

1s


Upload:

60s
```

---

# ❌ Anti-pattern 3

Ignorisanje context-a

Loše:

```go
func Process(){

}
```

---

Bolje:

```go
func Process(
	ctx context.Context,
){

}
```

---

# ❌ Anti-pattern 4

Previše veliki timeout-i

Primer:

```
5 minuta
```

---

Posledice:

- zauzeti resursi,
- spor oporavak,
- queue problemi.

---

# ❌ Anti-pattern 5

Misliti da timeout ubija Goroutine

Ne.

---

Timeout:

```text
šalje signal
```

---

Goroutine mora:

```go
select {

case <-ctx.Done():

	return

}
```

---

# Production checklist

Pre produkcije proveriti:

---

## HTTP Client

✅ Ima timeout

```go
http.Client{
	Timeout: ...
}
```

---

## HTTP Server

✅ Ima:

```go
ReadTimeout

WriteTimeout

IdleTimeout
```

---

## Database

✅ Koristi:

```go
QueryContext()
```

---

## Goroutines

✅ Imaju:

```go
ctx.Done()
```

---

## Background jobs

✅ Imaju:

- shutdown signal
- cleanup

---

## Context

✅ Prosleđuje se kroz slojeve

---

# Pitanja za proveru znanja

---

## 1.

Koja je razlika između:

```go
WithTimeout()
```

i:

```go
WithDeadline()
```

---

## 2.

Da li timeout ubija Goroutine?

Objasniti.

---

## 3.

Zašto je važno koristiti:

```go
ctx.Done()
```

---

## 4.

Šta je timeout budget?

---

## 5.

Zašto svaki servis ne treba da kreira svoj timeout?

---

## 6.

Koja je razlika između:

```go
context.Canceled
```

i:

```go
context.DeadlineExceeded
```

---

# Praktični zadaci

---

# 🟢 Nivo 1 — Osnovni

Napraviti funkciju:

```go
func FetchData(
	ctx context.Context,
) error
```

---

Zahtevi:

- simulirati rad 5 sekundi,
- timeout 2 sekunde,
- pravilno obraditi error.

---

# 🟡 Nivo 2 — HTTP Client

Napraviti HTTP client wrapper:

```go
type Client struct {

	http *http.Client

}
```

---

Zahtevi:

- timeout konfiguracija,
- context podrška,
- error handling.

---

# 🟠 Nivo 3 — Database

Napraviti repository:

```go
type UserRepository struct{}
```

---

Implementirati:

```go
GetUser(ctx,id)
```

---

Koristiti:

```go
QueryContext()
```

---

# 🔴 Nivo 4 — Senior

Napraviti mini distributed sistem:

Komponente:

```
API

 |

Service

 |

Worker

 |

Database
```

---

Zahtevi:

- jedan parent deadline,
- propagacija context-a,
- timeout handling,
- graceful cancellation.

---

# Završni mentalni model

Zapamti:

```
Timeout

=

granica čekanja


Context

=

prenos granice kroz sistem


Cancellation

=

signal za završetak


Cleanup

=

odgovornost aplikacije
```

---

### ➡️ Sledeća lekcija **[**Cancellation**](05-cancellation.md)**

Obradićemo:

- šta je cancellation,
- razlika između timeout i cancellation,
- cooperative cancellation,
- context cancellation,
- graceful shutdown,
- worker shutdown pattern,
- production cancellation strategije.
