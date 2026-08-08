# Context Advanced Patterns

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Context Advanced Patterns
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Zašto postoji `context.Context`?
- Istorija i motivacija
- Šta je Context?
- Osnovni interfejs
- Immutability
- Context Tree
- Root Context
- Background vs TODO
- Lifecycle Context-a
- Osnovna pravila korišćenja

---

# 1. Zašto Postoji Context?

Pre nego što je `context` paket postao standard, Go aplikacije su često koristile:

- globalne promenljive
- `done` channel-e
- prilagođene (`custom`) cancellation mehanizme
- ručno prosleđivanje timeout vrednosti

---

Primer:

```go
func Process(
	done <-chan struct{},
	timeout time.Duration,
	userID int,
) {
	// ...
}
```

---

Kako je aplikacija rasla, broj parametara je rastao zajedno sa njom.

---

Pojavljivali su se problemi:

- različiti mehanizmi za prekid rada
- nedosledno upravljanje timeout-ima
- teško propagiranje signala kroz više slojeva aplikacije

---

# 2. Motivacija

Zamislimo HTTP zahtev:

```
HTTP Request

↓

Authentication

↓

Business Logic

↓

Database

↓

External API

↓

Cache
```

---

Ako korisnik prekine zahtev:

```
Client disconnected
```

---

Sve ove operacije treba:

```
odmah prekinuti.
```

---

Bez centralnog mehanizma,

svaka komponenta bi morala da implementira sopstveni način za prekid rada.

---

# 3. Šta je Context?

`context.Context` predstavlja objekat koji prenosi informacije kroz pozive funkcija.

---

Njegova osnovna uloga nije skladištenje podataka.

---

Njegova uloga je:

- cancellation
- deadline
- timeout
- request-scoped vrednosti

---

Može se posmatrati kao:

```
kontrolni kanal
```

koji putuje zajedno sa zahtevom.

---

# 4. Context Interface

Osnovni interfejs izgleda ovako:

```go
type Context interface {

	Deadline() (
		deadline time.Time,
		ok bool,
	)

	Done() <-chan struct{}

	Err() error

	Value(key any) any

}
```

---

Svaka implementacija mora obezbediti ove četiri metode.

---

# 5. Četiri Osnovne Funkcionalnosti

## 1. Deadline

```
Kada operacija mora biti završena?
```

---

## 2. Done

```
Da li je zahtev otkazan?
```

---

## 3. Err

```
Zašto je otkazan?
```

---

## 4. Value

```
Koje request-scoped podatke nosi?
```

---

Ove četiri funkcionalnosti pokrivaju većinu potreba modernih distribuiranih sistema.

---

# 6. Context Je Immutable

Jedna od najvažnijih osobina.

---

Ne postoji:

```go
ctx.SetDeadline(...)
```

---

Ne postoji:

```go
ctx.Cancel()
```

---

Umesto izmene postojećeg objekta,

uvek se kreira:

```
novi Context
```

---

Na primer:

```go
ctx2, cancel := context.WithCancel(ctx)
```

---

Originalni `ctx` ostaje nepromenjen.

---

# 7. Context Tree

Context objekti formiraju stablo.

---

Primer:

```
Background()

      |

      |

      v

WithCancel()

      |

      |

      v

WithTimeout()

      |

      |

      v

WithValue()
```

---

Svaki novi Context ima:

```
jednog roditelja
```

---

# 8. Nasleđivanje

Child Context automatski nasleđuje:

- cancellation
- deadline
- values

---

Primer:

```
Parent

↓

Child

↓

Grandchild
```

---

Ako roditelj bude otkazan,

sva deca se takođe otkazuju.

---

# 9. Root Context

Stablo uvek počinje od:

```go
context.Background()
```

---

Vizuelno:

```
Background()

↓

Request Context

↓

Database Context

↓

Query Context
```

---

Background nikada nije otkazan.

---

On predstavlja koren stabla.

---

# 10. Background vs TODO

Postoje dva osnovna root Context-a.

---

## Background

Koristi se kada:

```
znamo

da je ovo pravi root.
```

---

Primer:

```go
func main() {

	ctx := context.Background()

}
```

---

## TODO

Koristi se kada:

```
još nismo sigurni

koji Context treba koristiti.
```

---

Primer:

```go
ctx := context.TODO()
```

---

Najčešće tokom razvoja ili refaktorisanja.

---

# 11. Lifecycle Context-a

Tipičan životni ciklus:

```
Background()

↓

WithCancel()

↓

WithTimeout()

↓

WithValue()

↓

Operation

↓

Cancel

↓

Garbage Collection
```

---

Context nije namenjen dugoročnom čuvanju.

---

Njegov životni vek prati:

```
životni vek operacije.
```

---

# 12. Pravilo Prosleđivanja

Context se skoro uvek prosleđuje kao:

```
prvi argument
```

---

Primer:

```go
func FetchUser(

	ctx context.Context,

	id int,

) error {

	// ...

}
```

---

Ovo je zvanična Go konvencija.

---

# 13. Šta Ne Treba Raditi?

❌ Ne čuvati Context u strukturi.

---

Loše:

```go
type Service struct {

	ctx context.Context

}
```

---

Ispravno:

```go
func (

	s *Service,

) Process(

	ctx context.Context,

) {

}
```

---

Context pripada:

```
pozivu funkcije

ne objektu.
```

---

# 14. Senior Pravila

✔️ Context je immutable.

---

✔️ Svaki novi Context ima roditelja.

---

✔️ Cancellation se propagira niz stablo.

---

✔️ Context se prosleđuje kao prvi parametar.

---

✔️ Ne skladišti se u struct poljima.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ zašto postoji `context`

✅ osnovni interfejs

✅ četiri glavne metode

✅ immutable prirodu Context-a

✅ Context Tree

✅ Background i TODO

✅ lifecycle Context-a

✅ osnovna pravila korišćenja

---

# Context Advanced Patterns

## Deo #2 — Cancellation Internals

---

# 📚 Sadržaj

- `context.WithCancel`
- `CancelFunc`
- `Done()` kanal
- Propagacija cancellation-a
- `cancelCtx`
- Cancellation Tree
- Višestruki child Context-i
- Najčešće greške
- Production preporuke

---

# 1. Šta Radi `WithCancel`?

Najčešći način kreiranja novog Context-a.

```go
ctx, cancel := context.WithCancel(parent)
```

---

Dobijamo dve stvari:

```go
ctx
```

novi Context

i

```go
cancel
```

funkciju za prekid rada.

---

# 2. Šta Je `CancelFunc`?

Tip funkcije:

```go
type CancelFunc func()
```

---

Poziv:

```go
cancel()
```

ne šalje signal preko mreže,

ne ubija goroutine,

ne prekida thread.

---

On:

```
zatvara Done kanal
```

i označava Context kao otkazan.

---

# 3. Done Kanal

Najvažnija metoda Context-a:

```go
Done() <-chan struct{}
```

---

To je:

```
read-only channel
```

---

Worker obično čeka:

```go
select {

case <-ctx.Done():

	return

case job := <-jobs:

	process(job)

}
```

---

Na ovaj način može odmah reagovati na prekid rada.

---

# 4. Kako Izgleda Cancellation?

Mentalni model:

```
ctx.Done()

        |

        |

     close()

        |

        |

        v

Sve goroutine

koje čekaju

↓

nastavljaju izvršavanje
```

---

Kao i kod običnog:

```go
close(done)
```

---

Radi se o:

```
broadcast signalu.
```

---

# 5. `cancelCtx`

Interno,

`context.WithCancel`

kreira strukturu sličnu:

```go
type cancelCtx struct {

	Context

	done chan struct{}

	err error

	children ...

}
```

---

Ovo nije kompletna implementacija,

ali dobro opisuje osnovnu ideju.

---

`cancelCtx` čuva:

- roditelja
- `Done` kanal
- grešku (`Err`)
- listu child Context-a

---

# 6. Cancellation Tree

Primer:

```
Background()

      |

      |

      v

WithCancel()

   /        \

  /          \

Child A    Child B

              |

              |

              v

          Grandchild
```

---

Ako pozovemo:

```go
cancel()
```

na roditelju,

otkazuju se:

- Parent
- Child A
- Child B
- Grandchild

---

# 7. Propagacija

Cancellation ide:

```
odozgo

↓

nadole
```

---

Nikada:

```
odozdo

↑

nagore
```

---

Ako Child bude otkazan,

roditelj nastavlja normalno da radi.

---

# 8. Primer

```go
root := context.Background()

ctx1, cancel1 := context.WithCancel(root)

ctx2, _ := context.WithCancel(ctx1)

ctx3, _ := context.WithCancel(ctx2)
```

---

Poziv:

```go
cancel1()
```

otkazuje:

```
ctx1

↓

ctx2

↓

ctx3
```

---

Ali:

```go
cancel(ctx3)
```

ne utiče na:

```
ctx2

ili

ctx1
```

---

# 9. `Err()` Metoda

Posle cancellation-a:

```go
ctx.Err()
```

vraća:

```go
context.Canceled
```

---

Ako nije otkazan:

```go
nil
```

---

Primer:

```go
<-ctx.Done()

if err := ctx.Err(); err != nil {

	log.Println(err)

}
```

---

Na ovaj način možemo razlikovati:

- uspešan završetak
- prekid rada

---

# 10. Višestruki Child Context-i

Jedan roditelj može imati više dece.

---

Primer:

```
Parent

  |

  +-----+

  |     |

  v     v

 C1    C2

       |

       v

      C3
```

---

Poziv:

```go
cancel(parent)
```

prekida:

```
C1

C2

C3
```

---

Sve odjednom.

---

# 11. Zašto Je Ovo Efikasno?

Nema potrebe da:

svaka goroutine šalje signal svakoj drugoj.

---

Dovoljno je:

```
jedan

cancel()
```

---

Runtime propagira signal kroz celo stablo.

---

# 12. Poziv `cancel()` Više Puta

Primer:

```go
cancel()

cancel()

cancel()
```

---

Potpuno bezbedno.

---

`CancelFunc` je:

```
idempotentna
```

---

Prvi poziv:

izvršava cancellation.

---

Sledeći:

ne rade ništa.

---

# 13. Zašto Treba Pozvati `cancel()`?

Čak i kada timeout nije istekao.

---

Primer:

```go
ctx, cancel := context.WithCancel(parent)

defer cancel()
```

---

Razlog:

interni resursi mogu biti oslobođeni ranije.

---

Ovo je preporučeni obrazac.

---

# 14. Česte Greške

❌ Zaboravljen poziv:

```go
cancel()
```

---

❌ Ignorisanje:

```go
ctx.Done()
```

u dugotrajnim goroutines.

---

❌ Očekivanje da cancellation prekida izvršavanje automatski.

---

Ne prekida.

---

Kod mora eksplicitno proveravati:

```go
ctx.Done()
```

---

# 15. Production Pravila

✔️ Svaki `WithCancel` treba da ima odgovarajući `cancel()`.

---

✔️ Koristiti:

```go
defer cancel()
```

kada je moguće.

---

✔️ Redovno proveravati:

```go
ctx.Done()
```

u dugotrajnim operacijama.

---

✔️ Ne ignorisati:

```go
ctx.Err()
```

nakon prekida rada.

---

# 16. Mentalni Model

`cancel()` ne zaustavlja goroutine.

---

On samo kaže:

```
"vreme je

da završiš."
```

---

Na goroutine je odgovornost da:

- proveri signal
- očisti resurse
- bezbedno završi rad

---

# 📋 Rezime

U ovom delu naučili smo:

✅ `WithCancel`

✅ `CancelFunc`

✅ `Done()` kanal

✅ `cancelCtx`

✅ Cancellation Tree

✅ propagaciju cancellation-a

✅ `Err()`

✅ idempotentnost `cancel()`

---

# Context Advanced Patterns

## Deo #3 — Deadlines i Timeouts

---

# 📚 Sadržaj

- Zašto su timeout-i važni?
- `context.WithTimeout`
- `context.WithDeadline`
- `timerCtx`
- `Deadline()`
- `DeadlineExceeded`
- Propagacija timeout-a
- Nested timeout-i
- Production preporuke

---

# 1. Zašto Su Timeout-i Važni?

Bez vremenskog ograničenja,

operacija može čekati:

- beskonačno
- minutima
- satima

---

Primer:

```
HTTP Request

↓

Database

↓

Network

↓

???

```

---

Ako baza nikada ne odgovori,

goroutine može zauvek ostati blokirana.

---

Timeout obezbeđuje:

```
gornju granicu

trajanja operacije.
```

---

# 2. `context.WithTimeout`

Najčešći način za ograničavanje trajanja operacije.

```go
ctx, cancel := context.WithTimeout(

	parent,

	5*time.Second,

)

defer cancel()
```

---

Posle:

```
5 sekundi
```

Context će automatski biti otkazan.

---

# 3. Šta Se Interno Dešava?

`WithTimeout` kreira:

- novi Context
- timer
- `Done()` kanal
- `CancelFunc`

---

Kada timer istekne:

```
Timer

↓

cancel()

↓

close(done)

↓

goroutines završavaju rad
```

---

# 4. `WithDeadline`

Sličan `WithTimeout`-u,

ali koristi apsolutno vreme.

```go
deadline := time.Now().Add(5 * time.Second)

ctx, cancel := context.WithDeadline(

	parent,

	deadline,

)

defer cancel()
```

---

Razlika:

- `WithTimeout` koristi trajanje (`Duration`)
- `WithDeadline` koristi tačan trenutak (`Time`)

---

# 5. `timerCtx`

Interno,

`WithTimeout` i `WithDeadline`

koriste strukturu sličnu:

```go
type timerCtx struct {

	cancelCtx

	timer *time.Timer

	deadline time.Time

}
```

---

`timerCtx` proširuje `cancelCtx`

dodavanjem:

- tajmera
- informacije o deadline-u

---

# 6. `Deadline()` Metoda

Možemo proveriti da li Context ima rok.

```go
deadline, ok := ctx.Deadline()

if ok {

	fmt.Println(deadline)

}
```

---

Ako:

```go
ok == false
```

Context nema definisan deadline.

---

# 7. `DeadlineExceeded`

Ako operacija istekne,

poziv:

```go
ctx.Err()
```

vraća:

```go
context.DeadlineExceeded
```

---

Ovo se razlikuje od:

```go
context.Canceled
```

---

Na taj način možemo razlikovati:

- ručni prekid rada
- istek vremena

---

# 8. Primer

```go
select {

case <-ctx.Done():

	return ctx.Err()

case result := <-results:

	return result

}
```

---

Ako istekne timeout,

funkcija vraća:

```go
context.DeadlineExceeded
```

---

# 9. Propagacija Deadline-a

Deadline se nasleđuje.

---

Primer:

```
Parent

↓

10 s

↓

Child

↓

5 s
```

---

Child koristi:

```
5 sekundi
```

---

Jer je stroži rok.

---

Obrnuto:

```
Parent

↓

5 s

↓

Child

↓

30 s
```

---

Child i dalje koristi:

```
5 sekundi.
```

---

Child nikada ne može produžiti rok roditelja.

---

# 10. Nested Timeouts

Primer:

```go
root := context.Background()

ctx1, _ := context.WithTimeout(

	root,

	30*time.Second,

)

ctx2, _ := context.WithTimeout(

	ctx1,

	5*time.Second,

)
```

---

Vizuelno:

```
30 s

↓

5 s
```

---

Efektivni timeout:

```
5 sekundi
```

---

# 11. Zašto Ne Treba Preterivati?

Loš primer:

```go
WithTimeout(...)

↓

WithTimeout(...)

↓

WithTimeout(...)

↓

WithTimeout(...)
```

---

Svaki novi timeout:

- pravi novi timer
- troši memoriju
- povećava složenost

---

Timeout postavljati:

```
na logičkim granicama sistema.
```

---

# 12. Production Primer

```
HTTP Request

↓

30 s

↓

Business Logic

↓

10 s

↓

Database

↓

2 s
```

---

Svaki sloj ima:

```
realističan

maksimalni rok.
```

---

# 13. Kada Koristiti `WithDeadline`?

Koristan je kada više operacija mora završiti do istog trenutka.

---

Na primer:

```
23:59:59
```

ili:

```
istek sertifikata

istek batch prozora

istek radnog vremena
```

---

Sve komponente dele isti apsolutni rok.

---

# 14. Česte Greške

❌ Zaboravljen:

```go
defer cancel()
```

---

❌ Prekratak timeout.

---

❌ Predugačak timeout.

---

❌ Višestruki nepotrebni timeout-i.

---

❌ Ignorisanje:

```go
ctx.Err()
```

---

# 15. Production Pravila

✔️ Timeout treba da bude realan.

---

✔️ Deadline ne treba nepotrebno produžavati u child Context-u.

---

✔️ Uvek proveravati:

```go
ctx.Done()
```

u dugotrajnim operacijama.

---

✔️ Pozvati:

```go
cancel()
```

čak i kada timeout nije istekao.

---

# 16. Mentalni Model

Timeout nije mehanizam za:

```
ubijanje goroutine.
```

---

On predstavlja:

```
ugovor

o maksimalnom

trajanju operacije.
```

---

Kada vreme istekne,

Context samo signalizira:

```
"vreme je isteklo."
```

---

Na aplikaciji je da pravilno reaguje.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ `WithTimeout`

✅ `WithDeadline`

✅ `timerCtx`

✅ `Deadline()`

✅ `DeadlineExceeded`

✅ propagaciju timeout-a

✅ nested timeout-e

✅ production smernice

---

# Context Advanced Patterns

## Deo #4 — Context Values i Request-Scoped Podaci

---

# 📚 Sadržaj

- Šta je `context.WithValue`?
- Kada koristiti Context Values?
- Request-Scoped podaci
- Tipizirani ključevi
- Izbegavanje kolizija
- Anti-patterns
- Performance razmatranja
- Production preporuke

---

# 1. Šta Je `context.WithValue`?

Pored cancellation-a i timeout-a,

`Context` može prenositi i podatke.

Za to služi:

```go
context.WithValue()
```

---

Primer:

```go
ctx := context.WithValue(

	parent,

	key,

	value,

)
```

---

Rezultat je:

```
novi Context

koji sadrži dodatnu vrednost.
```

---

Kao i ostale funkcije iz `context` paketa,

`WithValue()` ne menja postojeći Context,

već kreira novi.

---

# 2. Zašto Postoje Context Values?

Zamislimo HTTP zahtev.

```
HTTP Request

↓

Authentication

↓

Authorization

↓

Business Logic

↓

Database
```

---

Sve ove komponente možda žele pristup:

- korisniku
- request ID-u
- trace ID-u
- correlation ID-u

---

Umesto dodavanja novih parametara svakoj funkciji,

isti Context može nositi ove informacije.

---

# 3. Request-Scoped Podaci

Najvažnije pravilo:

```
Context sadrži

request-scoped podatke.
```

---

Odnosno,

podatke koji važe samo za:

```
jedan zahtev
```

ili

```
jednu operaciju.
```

---

Primeri:

- Request ID
- Trace ID
- Span ID
- User ID
- Tenant ID
- Locale
- Time Zone

---

# 4. Dohvatanje Vrednosti

Vrednost se čita pomoću:

```go
ctx.Value(key)
```

---

Primer:

```go
userID := ctx.Value(userKey)
```

---

Povratni tip je:

```go
any
```

---

Zbog toga je potreban:

```
type assertion.
```

---

# 5. Type Assertion

Primer:

```go
id, ok := ctx.Value(userKey).(int)

if !ok {

	return errors.New("missing user id")

}
```

---

Na ovaj način izbegavamo:

```
panic
```

pri pogrešnom tipu.

---

# 6. Zašto Ne Koristiti String Kao Ključ?

Loš primer:

```go
ctx := context.WithValue(

	ctx,

	"user",

	42,

)
```

---

Problem:

```
kolizije ključeva.
```

---

Dve biblioteke mogu koristiti isti string:

```
"user"
```

i prepisati jedna drugu.

---

# 7. Tipizirani Ključevi

Zvanična preporuka je korišćenje posebnog tipa.

```go
type contextKey string
```

---

Primer:

```go
const userKey contextKey = "user"
```

---

Sada:

```go
ctx = context.WithValue(

	ctx,

	userKey,

	42,

)
```

---

Rizik od kolizije je značajno manji.

---

# 8. Još Bolji Pristup

Često se koristi:

```go
type userKeyType struct{}
```

---

Zatim:

```go
var userKey userKeyType
```

---

Pošto je tip jedinstven,

ne postoji mogućnost konflikta sa drugim paketima.

---

Ovo je preporučeni obrazac za javne biblioteke.

---

# 9. Kako Interno Radi `WithValue`?

Svaki novi poziv kreira novi Context.

---

Mentalni model:

```
Background()

      |

      v

WithCancel()

      |

      v

WithTimeout()

      |

      v

WithValue(user)

      |

      v

WithValue(traceID)
```

---

Svaki nivo čuva:

```
jedan ključ

↓

jednu vrednost.
```

---

# 10. Pretraga Vrednosti

Kada pozovemo:

```go
ctx.Value(key)
```

pretraga ide:

```
od child

↑

ka parent-u
```

---

Ako ključ nije pronađen,

nastavlja se pretraga uz stablo,

sve do:

```
Background()
```

---

# 11. Performance Razmatranja

Svaki dodatni `WithValue()`:

- kreira novi Context
- produžava lanac pretrage

---

Zbog toga nije preporučljivo čuvati:

```
desetine vrednosti
```

u jednom Context-u.

---

Za nekoliko request-scoped vrednosti,

overhead je zanemarljiv.

---

# 12. Šta Ne Treba Čuvati?

Nikada nemojte koristiti Context kao:

- globalni storage
- konfiguraciju aplikacije
- dependency injection container
- cache
- bazu podataka
- servis locator

---

Context nije namenjen za:

```
trajnu memoriju.
```

---

# 13. Anti-Pattern

Loš primer:

```go
ctx = context.WithValue(

	ctx,

	dbKey,

	db,

)
```

---

Još gore:

```go
ctx = context.WithValue(

	ctx,

	configKey,

	config,

)
```

---

Ovakve zavisnosti treba eksplicitno proslediti funkcijama ili strukturama.

---

# 14. Kada Koristiti Context Values?

Dobri primeri:

✔️ Request ID

✔️ Correlation ID

✔️ Trace ID

✔️ Span ID

✔️ Authenticated User

✔️ Tenant

✔️ Locale

---

Svi ovi podaci važe samo tokom trajanja jednog zahteva.

---

# 15. Production Pravila

✔️ Koristiti tipizirane ključeve.

---

✔️ Context koristiti samo za request-scoped podatke.

---

✔️ Ne koristiti string ključeve u javnim bibliotekama.

---

✔️ Ne skladištiti velike objekte.

---

✔️ Broj vrednosti u Context-u držati razumnim.

---

# 16. Senior Mentalni Model

`Context` nije:

```
map[string]any
```

---

Već:

```
kontrolni objekat

koji

po potrebi

nosi mali broj

request-scoped podataka.
```

---

Cancellation i timeout ostaju njegova primarna odgovornost.

---

`Value()` predstavlja pomoćnu funkcionalnost,

a ne zamenu za pravilan dizajn API-ja.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ `context.WithValue`

✅ request-scoped podatke

✅ `Value()`

✅ type assertion

✅ tipizirane ključeve

✅ pretragu vrednosti

✅ performance razmatranja

✅ anti-patterns

---

# Context Advanced Patterns

## Deo #5 — Context u Production Sistemima

---

# 📚 Sadržaj

- Context u HTTP serverima
- Context u HTTP klijentima
- Context u bazama podataka
- Context u gRPC servisima
- Context Propagation
- Distributed Tracing
- OpenTelemetry
- Najčešći Production obrasci
- Best Practices

---

# 1. Context Kao Standard u Go-u

Danas skoro sve moderne Go biblioteke prihvataju:

```go
context.Context
```

kao prvi parametar.

---

Primer:

```go
func Query(

	ctx context.Context,

	query string,

) error
```

---

To omogućava:

- cancellation
- timeout
- deadline
- request-scoped podatke

bez potrebe za dodatnim API-jima.

---

# 2. Context u HTTP Serverima

U paketu `net/http`

svaki zahtev već poseduje svoj Context.

```go
func handler(

	w http.ResponseWriter,

	r *http.Request,

) {

	ctx := r.Context()

}
```

---

Ovaj Context predstavlja:

```
životni vek

HTTP zahteva.
```

---

# 3. Kada Klijent Prekine Vezu

Ako korisnik:

- zatvori browser
- izgubi internet konekciju
- prekine zahtev

---

HTTP server automatski:

```
otkazuje

request Context.
```

---

Sve funkcije koje koriste:

```go
ctx
```

mogu odmah prekinuti rad.

---

# 4. Context Propagation

Jedno od najvažnijih pravila.

---

Nikada ne kreirati novi root Context usred obrade zahteva.

---

Loše:

```go
ctx := context.Background()
```

---

Ispravno:

```go
ctx := r.Context()
```

---

Zatim isti Context proslediti:

```
Handler

↓

Service

↓

Repository

↓

Database
```

---

# 5. Context u Bazama Podataka

Paket `database/sql`

podržava Context.

---

Primer:

```go
rows, err := db.QueryContext(

	ctx,

	query,

)
```

---

Ako Context bude otkazan:

- SQL upit može biti prekinut
- konekcija se oslobađa
- goroutine završava rad

---

Ovo sprečava:

```
dugotrajne

ili zaglavljene

SQL operacije.
```

---

# 6. Context u HTTP Klijentima

Primer:

```go
req, err := http.NewRequestWithContext(

	ctx,

	http.MethodGet,

	url,

	nil,

)
```

---

Ako istekne timeout,

HTTP zahtev će biti prekinut.

---

Nema potrebe za dodatnom logikom.

---

# 7. Context u gRPC-u

Kod gRPC servisa,

Context je sastavni deo svakog RPC poziva.

---

Primer:

```go
func (

	s *Server,

) GetUser(

	ctx context.Context,

	req *pb.Request,

) (*pb.Response, error) {

}
```

---

Na ovaj način:

- timeout
- cancellation
- metadata

putuju zajedno sa zahtevom.

---

# 8. Context Propagation Kroz Servise

Primer distribuiranog sistema:

```
API Gateway

↓

User Service

↓

Order Service

↓

Payment Service

↓

Database
```

---

Isti Context prolazi kroz sve servise.

---

Ako API Gateway prekine zahtev,

ceo lanac dobija signal za prekid rada.

---

# 9. Distributed Tracing

Context često nosi:

- Trace ID
- Span ID
- Correlation ID

---

Na primer:

```
HTTP Request

↓

Trace ID

↓

Database

↓

Cache

↓

External API
```

---

Sve komponente mogu povezati događaje koji pripadaju istom zahtevu.

---

# 10. OpenTelemetry

Savremene biblioteke za observability koriste Context.

---

Primer:

```go
ctx, span := tracer.Start(

	ctx,

	"CreateOrder",

)

defer span.End()
```

---

Span se automatski vezuje za postojeći Context.

---

Na taj način nastaje kompletan trag izvršavanja kroz sistem.

---

# 11. Context u Worker-ima

Worker treba redovno da proverava:

```go
select {

case <-ctx.Done():

	return

case job := <-jobs:

	process(job)

}
```

---

Na ovaj način,

worker može bezbedno završiti rad kada operacija bude otkazana.

---

# 12. Context u Pipeline-ovima

Pipeline primer:

```
Producer

↓

Parser

↓

Validator

↓

Storage
```

---

Svaka faza prima isti:

```go
ctx
```

---

Ako Context bude otkazan,

ceo pipeline se postepeno zaustavlja.

---

# 13. Production Anti-Patterns

❌ Kreiranje novog:

```go
context.Background()
```

unutar Service sloja.

---

❌ Ignorisanje:

```go
ctx.Done()
```

u dugotrajnim operacijama.

---

❌ Neprosleđivanje Context-a svim slojevima aplikacije.

---

❌ Korišćenje različitih Context objekata za isti zahtev.

---

# 14. Production Best Practices

✔️ Context proslediti kroz ceo call chain.

---

✔️ Timeout definisati na granici sistema.

---

✔️ Koristiti `QueryContext`, `ExecContext` i slične API-je.

---

✔️ Uvek poštovati cancellation signal.

---

✔️ Prosleđivati isti Context svim downstream servisima.

---

# 15. Mentalni Model

Context putuje zajedno sa zahtevom.

---

Vizuelno:

```
HTTP Request

↓

Context

↓

Service

↓

Repository

↓

Database

↓

External API
```

---

On predstavlja:

```
jedinstveni izvor

informacija

o životnom veku

celog zahteva.
```

---

# 16. Senior Pravila

✔️ Context se nikada ne prekida "na pola puta".

---

✔️ Ne kreirati novi root Context osim na ulazu u aplikaciju.

---

✔️ Svaki sloj prihvata i prosleđuje isti Context.

---

✔️ Sve spoljne operacije treba da podržavaju Context.

---

✔️ Observability alati koriste isti Context za tracing i metrike.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Context u HTTP serverima

✅ Context u HTTP klijentima

✅ Context u bazama podataka

✅ Context u gRPC servisima

✅ Context Propagation

✅ Distributed Tracing

✅ OpenTelemetry

✅ Production Best Practices

---

# Context Advanced Patterns

## Deo #6 — Context Best Practices, Anti-Patterns i Internals — Završni Pregled

---

# 📚 Sadržaj

- Interna implementacija Context-a
- `emptyCtx`
- `cancelCtx`
- `timerCtx`
- `valueCtx`
- Performance razmatranja
- Najčešće greške
- Context Cheat Sheet
- Završne preporuke

---

# 1. Kako Izgleda Context Interno?

Iako je `context.Context` interfejs,

standardna biblioteka koristi nekoliko konkretnih implementacija.

```
Context

│

├── emptyCtx

├── cancelCtx

├── timerCtx

└── valueCtx
```

---

Svaka rešava određeni problem.

---

# 2. `emptyCtx`

Koristi se za:

```go
context.Background()
```

i

```go
context.TODO()
```

---

Karakteristike:

- nikada nije otkazan
- nema deadline
- nema vrednosti
- predstavlja koren stabla

---

Mentalni model:

```
Background()

↓

root
```

---

# 3. `cancelCtx`

Kreira ga:

```go
context.WithCancel()
```

---

Dodaje:

- `Done()` kanal
- `Err()`
- listu child Context-a

---

Njegova odgovornost je:

```
propagacija cancellation-a.
```

---

# 4. `timerCtx`

Koriste ga:

```go
context.WithTimeout()
```

i

```go
context.WithDeadline()
```

---

Pored svega iz `cancelCtx`,

sadrži još:

- `time.Timer`
- deadline

---

Kada timer istekne,

automatski poziva cancellation.

---

# 5. `valueCtx`

Kreira ga:

```go
context.WithValue()
```

---

Sadrži:

```
ključ

↓

vrednost
```

---

Prilikom poziva:

```go
ctx.Value(key)
```

pretraga ide od child Context-a ka roditelju.

---

# 6. Kako Izgleda Context Tree?

```
Background()

      |

      v

WithCancel()

      |

      v

WithTimeout()

      |

      v

WithValue()
```

---

Svaki novi Context:

- referencira roditelja
- dodaje jednu novu osobinu
- ostavlja roditelja neizmenjenim

---

Ovo čini Context:

```
immutable
```

---

# 7. Performance Razmatranja

Kreiranje Context-a je jeftino,

ali nije besplatno.

---

Svaki poziv:

```go
WithValue()
```

ili

```go
WithTimeout()
```

dodaje novi nivo u stablu.

---

Previše ugnježdenih (`nested`) Context-a može:

- povećati broj alokacija
- produžiti pretragu vrednosti
- otežati razumevanje koda

---

U praksi je overhead mali,

ali vredi izbegavati nepotrebno kreiranje novih Context-a.

---

# 8. Najčešće Greške

❌ Čuvanje Context-a u `struct`.

```go
type Service struct {

	ctx context.Context

}
```

---

❌ Prosleđivanje `nil` umesto Context-a.

---

❌ Korišćenje `context.Background()` usred obrade zahteva.

---

❌ Zaboravljen:

```go
cancel()
```

---

❌ Ignorisanje:

```go
ctx.Done()
```

u dugotrajnim operacijama.

---

❌ Korišćenje Context-a kao DI (Dependency Injection) kontejnera.

---

# 9. Production Checklist

Pre nego što napišeš novu funkciju,

proveri:

✔️ Da li `Context` ide kao prvi parametar?

---

✔️ Da li se prosleđuje dalje bez izmene?

---

✔️ Da li postoji `defer cancel()` kada koristiš `WithCancel`, `WithTimeout` ili `WithDeadline`?

---

✔️ Da li sve dugotrajne operacije proveravaju:

```go
ctx.Done()
```

---

✔️ Da li `WithValue()` sadrži samo request-scoped podatke?

---

# 10. Context Cheat Sheet

| Problem | Rešenje |
|----------|---------|
| Root Context | `context.Background()` |
| Placeholder | `context.TODO()` |
| Ručni prekid rada | `context.WithCancel()` |
| Timeout | `context.WithTimeout()` |
| Apsolutni rok | `context.WithDeadline()` |
| Request-scoped podaci | `context.WithValue()` |
| Provera prekida | `<-ctx.Done()` |
| Razlog prekida | `ctx.Err()` |
| Rok izvršavanja | `ctx.Deadline()` |

---

# 11. Kada Koristiti Koju Funkciju?

```
Treba mi root Context

↓

Background()
```

---

```
Još nisam odlučio

↓

TODO()
```

---

```
Želim ručni prekid rada

↓

WithCancel()
```

---

```
Želim maksimalno trajanje

↓

WithTimeout()
```

---

```
Želim tačan trenutak isteka

↓

WithDeadline()
```

---

```
Treba mi Request ID

↓

WithValue()
```

---

# 12. Kako Razmišlja Senior Go Programer?

Ne razmišlja:

```
Kako da prosledim timeout?
```

---

Već:

```
Kako da ceo

životni vek

operacije

bude kontrolisan

jednim Context-om?
```

---

Context postaje:

```
izvor istine

za životni vek

zahteva.
```

---

# 13. Povezanost sa Ostalim Concurrency Temama

`Context` se prirodno kombinuje sa:

- goroutines
- channels
- `select`
- worker pool-ovima
- pipeline-ovima
- fan-out/fan-in obrascima
- semaphore obrascima
- HTTP serverima
- bazama podataka
- gRPC servisima

---

Zbog toga je jedan od najvažnijih paketa u savremenim Go aplikacijama.

---

# 14. Ključni Principi

Zapamti sledeća pravila:

1. Context je immutable.

2. Context nije storage.

3. Context nije konfiguracija.

4. Context nije dependency injection.

5. Context ide kao prvi parametar.

6. Context se ne čuva u strukturama.

7. Child ne može produžiti deadline roditelja.

8. Svaki `WithCancel()` treba da ima `cancel()`.

9. Poštuj `ctx.Done()`.

10. Prosledi isti Context kroz ceo call chain.

---

# 15. Završetak Modula

Završili smo:

```
docs/module-4/extras/

└── 07-context-advanced-patterns.md
```

---

Obrađene teme:

✅ Context Interface

✅ Background i TODO

✅ Cancellation

✅ Deadlines

✅ Timeouts

✅ Context Values

✅ Request-Scoped podaci

✅ Context Propagation

✅ HTTP

✅ Database

✅ gRPC

✅ Distributed Tracing

✅ OpenTelemetry

✅ Internals

✅ Best Practices

---

# 🎯 Šta Sada Znaš?

Sada razumeš:

- kako je `context` implementiran u standardnoj biblioteci
- kako funkcioniše propagacija cancellation-a kroz stablo Context-a
- kako pravilno koristiti timeout-e i deadline-ove
- kako bezbedno prenositi request-scoped podatke
- kako koristiti Context u HTTP, gRPC i `database/sql` API-jima
- kako izbeći najčešće anti-pattern-e
- kako projektovati production sisteme koji dosledno koriste jedan Context kroz ceo životni ciklus zahteva

---

### ➡️ Sledeća lekcija **[**Concurrency Performance Tuning**](08-concurrency-performance-tuning.md)**

Obuhvatiće:

- merenje performansi konkurentnih programa
- scheduler overhead
- goroutine scaling
- contention analiza
- benchmark tehnike
- CPU i memory profiling
- optimizacija throughput-a i latencije