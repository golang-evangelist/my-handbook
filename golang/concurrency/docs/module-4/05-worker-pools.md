# Worker Pools — Uvod

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/05-worker-pools.md`

---

# 📚 Sadržaj

- Šta je Worker Pool
- Zašto koristiti Worker Pool
- Problem neograničenog broja Goroutines
- Bounded Concurrency
- Osnovna arhitektura Worker Pool-a
- Production primeri

---

# Uvod

Jedna od najvećih prednosti Go-a je što je kreiranje Goroutine veoma jeftino.

Na primer:

```go
go process(job1)
go process(job2)
go process(job3)
```

Lako je napisati:

```go
for _, job := range jobs {
	go process(job)
}
```

Ali da li je to uvek dobra ideja?

Odgovor je:

```
Ne.
```

---

# Problem

Pretpostavimo da imamo:

```
1 000 000

zadataka
```

Ako za svaki pokrenemo novu Goroutine:

```
1 000 000

↓

Goroutines
```

To može dovesti do:

- velike potrošnje memorije,
- ogromnog broja zakazanih Goroutines,
- povećanog scheduler overhead-a,
- lošijih performansi,
- pritiska na spoljne resurse (baza, mreža, API).

---

# Šta je Worker Pool?

Worker Pool je obrazac u kome:

- postoji ograničen broj worker-a,
- worker-i uzimaju zadatke iz zajedničkog reda,
- broj istovremeno aktivnih zadataka je kontrolisan.

---

# Vizuelni model

```
Jobs

↓

Queue

↓

+----------+
| Worker 1 |
+----------+

+----------+
| Worker 2 |
+----------+

+----------+
| Worker 3 |
+----------+
```

Svi worker-i uzimaju poslove iz istog reda.

---

# Zašto je ovo bolje?

Umesto:

```
100 000

Goroutines
```

možemo imati:

```
10

Worker-a
```

koji obrađuju svih 100 000 poslova.

---

# Bounded Concurrency

Worker Pool uvodi pojam:

```
Bounded Concurrency
```

što znači:

> **Broj istovremeno izvršavanih zadataka je ograničen.**

Na primer:

```
10 worker-a

↓

najviše

10

aktivnih poslova
```

Bez obzira na to da li u redu čeka:

- 100,
- 10 000,
- 1 000 000 zadataka.

---

# Osnovna arhitektura

```
Producer

↓

Job Queue

↓

Workers

↓

Results
```

Svaka komponenta ima jasno definisanu odgovornost.

---

# Jednostavan primer

```go
jobs := make(chan Job)
```

Worker:

```go
func worker(jobs <-chan Job) {
	for job := range jobs {
		process(job)
	}
}
```

Pokretanje više worker-a:

```go
for i := 0; i < 4; i++ {
	go worker(jobs)
}
```

Slanje poslova:

```go
for _, job := range allJobs {
	jobs <- job
}

close(jobs)
```

Ovo je osnovni oblik Worker Pool-a.

---

# Zašto channels?

Channel ovde ima dve uloge:

1. Prenosi zadatke.

2. Sinhronizuje producer-e i worker-e prema Go Memory Model-u.

Time se dobija jednostavan i bezbedan način komunikacije između Goroutines.

---

# Production primeri

Worker Pool se često koristi za:

- obradu HTTP zahteva u pozadini,
- obradu slika,
- generisanje PDF-ova,
- slanje email-ova,
- import CSV fajlova,
- obradu logova,
- obradu događaja iz message broker-a.

---

# Koliko worker-a?

To zavisi od prirode posla.

## CPU-bound poslovi

Primer:

- kompresija,
- kriptografija,
- parsiranje.

Često je dobar početak broj worker-a približan broju logičkih procesora (`runtime.GOMAXPROCS(0)` ili `runtime.NumCPU()` kao polazna tačka), uz obavezno merenje performansi.

---

## I/O-bound poslovi

Primer:

- HTTP pozivi,
- rad sa bazom,
- čitanje fajlova.

Često ima smisla koristiti više worker-a nego što je broj CPU jezgara, jer worker-i veći deo vremena čekaju spoljne resurse.

Tačan broj zavisi od aplikacije i treba ga odrediti benchmark-om i posmatranjem sistema.

---

# Prednosti

✅ Kontrolisana konkurentnost.

✅ Stabilnija potrošnja memorije.

✅ Manje opterećenje scheduler-a.

✅ Lakše upravljanje resursima.

✅ Jednostavno skaliranje.

---

# Ograničenja

❌ Loše odabran broj worker-a može smanjiti performanse.

❌ Worker Pool nije najbolji izbor za svaki problem.

❌ Potrebno je razmišljati o redovima čekanja i gašenju sistema.

---

# Najčešće greške

## Greška #1

Jedna Goroutine po zadatku.

To može biti dobro za mali broj zadataka, ali loše za veoma velike serije poslova.

---

## Greška #2

Preveliki broj worker-a.

Više nije uvek bolje.

Previše worker-a može povećati contention i scheduling overhead.

---

## Greška #3

Zaboraviti da se zatvori kanal sa poslovima.

Ako worker-i čekaju zauvek:

```go
for job := range jobs {
}
```

dobijaš Goroutine leak.

---

# Best Practices

✅ Koristi Worker Pool kada želiš da ograničiš broj istovremenih zadataka.

✅ Zatvori `jobs` kanal kada više nema novih poslova.

✅ Biraj broj worker-a na osnovu prirode posla i merenja, a ne pretpostavki.

✅ Koristi `context.Context` za otkazivanje dugotrajnih operacija (obrađujemo u narednim delovima).

---

# Mentalni model

Nemoj razmišljati:

```
Jedan posao

↓

Jedna Goroutine
```

Razmišljaj:

```
Mnogo poslova

↓

Red čekanja

↓

Ograničen broj worker-a

↓

Kontrolisana obrada
```

To je suština Worker Pool obrasca.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Worker Pool

✅ zašto ne treba pokretati neograničen broj Goroutines

✅ šta znači bounded concurrency

✅ osnovnu arhitekturu Worker Pool-a

✅ gde se koristi u produkcionim sistemima

---

# Worker Pools — Implementacija osnovnog Worker Pool-a

---

# 📚 Sadržaj

- Arhitektura Worker Pool-a
- Definisanje poslova
- Worker funkcija
- `jobs` kanal
- `results` kanal
- `sync.WaitGroup`
- Tok izvršavanja

---

# Arhitektura

Naš Worker Pool izgleda ovako:

```text
          Producer
              │
              ▼
      +---------------+
      |  jobs channel |
      +---------------+
        │     │     │
        ▼     ▼     ▼
    Worker Worker Worker
        │     │     │
        └──┬──┴──┬──┘
           ▼     ▼
     +-----------------+
     | results channel |
     +-----------------+
              │
              ▼
          Consumer
```

Svaki deo ima jednu odgovornost.

---

# Definisanje posla

Radi jednostavnosti:

```go
type Job struct {
	ID int
}
```

Rezultat:

```go
type Result struct {
	JobID int
	Value string
}
```

U realnim aplikacijama ove strukture mogu sadržati mnogo više podataka.

---

# Worker funkcija

```go
func worker(
	id int,
	jobs <-chan Job,
	results chan<- Result,
	wg *sync.WaitGroup,
) {
	defer wg.Done()

	for job := range jobs {

		result := Result{
			JobID: job.ID,
			Value: fmt.Sprintf(
				"worker %d processed job %d",
				id,
				job.ID,
			),
		}

		results <- result
	}
}
```

Važne stvari:

- worker čita samo sa `jobs`,
- piše samo na `results`,
- završava kada se `jobs` kanal zatvori.

---

# Pokretanje Worker-a

```go
const workerCount = 4

var wg sync.WaitGroup

for i := 0; i < workerCount; i++ {
	wg.Add(1)

	go worker(
		i,
		jobs,
		results,
		&wg,
	)
}
```

Ovde pokrećemo četiri nezavisna worker-a.

---

# Slanje poslova

```go
for i := 1; i <= 10; i++ {

	jobs <- Job{
		ID: i,
	}
}

close(jobs)
```

Kada više nema novih poslova:

```go
close(jobs)
```

To je signal svim worker-ima da, nakon obrade preostalih poslova, mogu da završe.

---

# Prikupljanje rezultata

Jedan od mogućih obrazaca:

```go
go func() {
	wg.Wait()
	close(results)
}()
```

Zatim:

```go
for result := range results {
	fmt.Println(result)
}
```

Zašto?

`results` se zatvara tek kada:

- svi worker-i završe,
- više niko neće slati nove rezultate.

---

# Tok izvršavanja

Korak 1

```
Producer

↓

jobs
```

---

Korak 2

```
Worker

↓

uzima posao
```

---

Korak 3

```
Obrada
```

---

Korak 4

```
Result

↓

results
```

---

Korak 5

```
Consumer

↓

prima rezultat
```

---

# Vizuelni tok

```text
Producer
   │
   ▼
jobs channel
   │
   ├──────────────┐
   ▼              ▼
Worker 1      Worker 2
   │              │
   └──────┬───────┘
          ▼
results channel
          │
          ▼
      Consumer
```

---

# Zašto `WaitGroup`?

Bez njega ne znamo:

```
Kada su

svi worker-i

završili?
```

`WaitGroup` omogućava da bezbedno sačekamo kraj rada svih worker-a pre zatvaranja `results` kanala.

---

# Zašto worker koristi `range`?

```go
for job := range jobs {
}
```

To omogućava da worker:

- automatski obrađuje svaki pristigli posao,
- elegantno završi kada se kanal zatvori.

Nema potrebe za posebnim signalima.

---

# Da li su rezultati sortirani?

Ne.

Na primer:

```
Job 1

↓

Worker 3

↓

Gotovo
```

Može završiti pre:

```
Job 0

↓

Worker 1
```

Redosled rezultata zavisi od vremena izvršavanja pojedinačnih poslova.

Ako je potreban isti redosled kao ulaz, potrebno je dodati dodatnu logiku (npr. indeksiranje i naknadno sortiranje ili posebno orkestriranje).

---

# Šta ako jedan posao traje mnogo duže?

Na primer:

```
Job 3

↓

5 sekundi
```

Dok:

```
Job 4

↓

10 ms
```

Drugi worker-i nastavljaju da obrađuju dostupne poslove.

Worker Pool prirodno raspoređuje posao među slobodnim worker-ima.

---

# Production tok

```text
Incoming Requests
        │
        ▼
   Job Queue
        │
        ▼
+---------------+
| Worker Pool   |
+---------------+
        │
        ▼
 Database / API
        │
        ▼
     Results
```

Ovaj obrazac je čest u servisima koji obrađuju veliki broj nezavisnih zadataka.

---

# Najčešće greške

## Greška #1

Zatvaranje `results` kanala prerano.

Ako neki worker i dalje pokušava:

```go
results <- value
```

nastaje panika.

---

## Greška #2

Zaboraviti:

```go
close(jobs)
```

Worker-i ostaju blokirani čekajući nove poslove.

---

## Greška #3

Više Goroutines zatvara isti kanal.

Uobičajeno pravilo je:

> **Kanal zatvara onaj ko šalje podatke, i to samo jednom.**

---

# Best Practices

✅ Koristi `jobs <-chan` i `results chan<-` kako bi jasno definisao smer komunikacije.

✅ `WaitGroup` koristi za koordinaciju završetka worker-a.

✅ `results` zatvori tek nakon što svi worker-i završe.

✅ Nemoj pretpostavljati redosled pristizanja rezultata.

---

# Mentalni model

Nemoj razmišljati:

```text
Job 1

↓

Worker 1

↓

Rezultat 1
```

Razmišljaj:

```text
Mnogo poslova

↓

Zajednički red

↓

Prvi slobodan worker

↓

Rezultat
```

To omogućava efikasno iskorišćenje dostupnih worker-a.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako izgleda kompletan Worker Pool

✅ kako koristiti `jobs` i `results` kanale

✅ zašto je `sync.WaitGroup` važan

✅ kako pravilno ugasiti worker-e

✅ zašto redosled rezultata nije garantovan

---

# Worker Pools — Bounded Concurrency

---

# 📚 Sadržaj

- Šta je Bounded Concurrency
- Zašto ograničavati konkurentnost
- Worker Pool vs neograničen broj Goroutines
- Worker Pool vs Semaphore
- CPU-bound i I/O-bound workload
- Kako izabrati broj worker-a

---

# Uvod

Početnici često napišu:

```go
for _, job := range jobs {
	go process(job)
}
```

Na prvi pogled ovo izgleda odlično.

Ali šta ako:

```
jobs = 1 000 000
```

?

Dobićemo:

```
1 000 000

Goroutines
```

Iako su Goroutines jeftine, nisu besplatne.

---

# Šta je Bounded Concurrency?

Bounded Concurrency znači:

```
Postoji

maksimalan broj

istovremenih zadataka.
```

Na primer:

```
Worker Pool

↓

8 Worker-a

↓

Najviše

8

aktivnih poslova
```

Bez obzira na veličinu reda čekanja.

---

# Vizuelni primer

Bez ograničenja:

```text
1000 Jobs

↓

1000 Goroutines
```

---

Sa Worker Pool-om:

```text
1000 Jobs

↓

Queue

↓

8 Workers
```

Razlika je ogromna.

---

# Zašto ograničavati konkurentnost?

Resursi su ograničeni.

Na primer:

CPU:

```
8

jezgara
```

Baza:

```
100

konekcija
```

HTTP API:

```
50

zahteva u sekundi
```

Ako pokrenemo:

```
10 000

Goroutines
```

nećemo magično dobiti:

```
10 000

CPU jezgara
```

ili

```
10 000

DB konekcija.
```

---

# Cilj Worker Pool-a

Nije:

```
Što više Goroutines.
```

Već:

```
Optimalan broj

aktivnih poslova.
```

---

# Worker Pool vs Semaphore

Oba pristupa mogu ograničiti konkurentnost, ali rešavaju različite probleme.

---

## Worker Pool

Postoji unapred definisan broj worker-a.

```text
Jobs

↓

Workers
```

Poslovi čekaju slobodnog worker-a.

---

## Semaphore

Svaka Goroutine pokušava da dobije dozvolu (permit).

```text
Acquire

↓

Work

↓

Release
```

Broj istovremenih operacija je ograničen, ali Goroutines mogu biti kreirane unapred.

---

# Vizuelno

## Worker Pool

```text
Jobs

↓

Queue

↓

Workers
```

---

## Semaphore

```text
Goroutines

↓

Semaphore

↓

Running
```

---

# Kada koristiti Worker Pool?

Kada postoji veliki broj nezavisnih zadataka koji mogu čekati u redu.

Na primer:

- obrada slika,
- import CSV fajlova,
- obrada logova,
- slanje email-ova.

---

# Kada koristiti Semaphore?

Kada već postoji Goroutine po zadatku, ali želiš da ograničiš pristup ograničenom resursu.

Na primer:

- maksimalno 20 paralelnih HTTP zahteva,
- maksimalno 10 pristupa eksternom API-ju,
- maksimalno 5 paralelnih upisa u bazu.

---

# CPU-bound workload

Primeri:

- kompresija,
- enkripcija,
- obrada videa,
- parsiranje.

Karakteristike:

```
CPU

↓

radi skoro stalno
```

Često je dobar početni izbor broj worker-a približan broju logičkih procesora (`runtime.GOMAXPROCS(0)` ili `runtime.NumCPU()`), uz obavezno merenje.

Prevelik broj worker-a može povećati troškove raspoređivanja bez dobitka u performansama.

---

# I/O-bound workload

Primeri:

- HTTP pozivi,
- baza podataka,
- čitanje fajlova,
- cloud servisi.

Karakteristike:

```
CPU

↓

često čeka
```

Zbog čekanja na spoljne resurse često ima smisla koristiti više worker-a nego što je broj CPU jezgara.

Koliko više zavisi od trajanja čekanja, latencije i ograničenja spoljnog sistema.

---

# Kako odrediti broj worker-a?

Ne postoji univerzalna formula.

Dobar proces je:

1. Počni sa razumnom vrednošću.
2. Benchmark-uj.
3. Profiliši (`pprof`).
4. Posmatraj CPU, memoriju i latenciju.
5. Prilagodi broj worker-a.

Merenje je važnije od pretpostavki.

---

# Premali broj worker-a

```text
1000 Jobs

↓

2 Workers
```

Posledice:

- dugi redovi čekanja,
- slabo iskorišćenje resursa,
- veća ukupna latencija.

---

# Preveliki broj worker-a

```text
1000 Jobs

↓

1000 Workers
```

Posledice:

- više context switch-eva,
- veći scheduler overhead,
- contention nad deljenim resursima,
- veća potrošnja memorije.

---

# Production primer

Pretpostavimo servis koji generiše PDF izveštaje.

Ako istovremeno primi:

```
5000

zahteva
```

nije poželjno pokrenuti:

```
5000

Goroutines
```

koje odmah kreću sa generisanjem.

Bolje rešenje:

```text
5000 Jobs

↓

Queue

↓

16 Workers

↓

PDF
```

Time je opterećenje sistema kontrolisano i predvidljivo.

---

# Najčešće greške

## Greška #1

Jedan worker po CPU jezgru za I/O-bound zadatke.

To često ostavlja CPU neiskorišćen dok worker-i čekaju mrežu ili disk.

---

## Greška #2

Pretpostaviti da više worker-a uvek znači bolje performanse.

Nakon određene tačke dodatni worker-i mogu samo povećati overhead.

---

## Greška #3

Ignorisati ograničenja spoljnog sistema.

Ako baza dozvoljava 50 konekcija, 500 worker-a neće ubrzati obradu.

---

# Best Practices

✅ Razmišljaj o ograničenjima celog sistema, ne samo aplikacije.

✅ Razlikuj CPU-bound i I/O-bound workload.

✅ Broj worker-a određuj na osnovu merenja.

✅ Koristi Worker Pool za velike serije nezavisnih poslova.

---

# Mentalni model

Nemoj razmišljati:

```text
Više worker-a

↓

Brže
```

Razmišljaj:

```text
Optimalan broj worker-a

↓

Najbolji odnos

performansi

i

potrošnje resursa
```

To je suština bounded concurrency-ja.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Bounded Concurrency

✅ zašto ograničavati broj istovremenih zadataka

✅ razliku između Worker Pool-a i Semaphore obrasca

✅ razliku između CPU-bound i I/O-bound workload-a

✅ kako praktično odrediti broj worker-a

---

# Worker Pools — Graceful Shutdown

---

# 📚 Sadržaj

- Šta je Graceful Shutdown
- Zašto je važan
- `context.Context`
- Otkazivanje poslova
- Zatvaranje kanala
- Goroutine Leak
- Production obrasci

---

# Uvod

Zamisli da aplikacija prima:

```
SIGTERM
```

ili

```
Ctrl+C
```

Da li želiš da:

```
odmah prekine

sve Goroutines?
```

Obično:

```
NE
```

Želiš da:

- završi započete poslove (ako je to odgovarajuće za aplikaciju),
- odbije nove poslove,
- oslobodi resurse,
- izađe bez curenja resursa.

To se naziva:

```
Graceful Shutdown
```

---

# Šta znači Graceful Shutdown?

Umesto:

```
Stop

↓

Sve prekini
```

želimo:

```
Stop

↓

Nema novih poslova

↓

Završi postojeće

↓

Ugasi Worker-e

↓

Izlaz
```

---

# Vizuelni tok

```text
Producer

↓

Stop Accepting Jobs

↓

Workers Finish

↓

Close Channels

↓

Exit
```

---

# Zašto je važan?

Bez Graceful Shutdown-a može doći do:

- izgubljenih poslova,
- nedovršenih transakcija,
- oštećenih izlaznih podataka,
- curenja Goroutines,
- neoslobođenih konekcija.

---

# `context.Context`

U Go-u je standardni mehanizam za otkazivanje rada:

```go
ctx context.Context
```

Worker može pratiti stanje konteksta:

```go
select {
case <-ctx.Done():
	return

case job := <-jobs:
	process(job)
}
```

Kada se kontekst otkaže:

```
ctx.Done()

↓

worker završava
```

---

# Zašto koristiti `select`?

Ako worker radi samo:

```go
for job := range jobs {
	process(job)
}
```

može zauvek čekati novi posao.

Sa `select` može reagovati i na:

- novi posao,
- signal za gašenje.

---

# Tipičan obrazac

```go
for {
	select {

	case <-ctx.Done():
		return

	case job, ok := <-jobs:

		if !ok {
			return
		}

		process(job)
	}
}
```

Ovaj obrazac omogućava uredan izlazak i kada je kontekst otkazan i kada je kanal zatvoren.

---

# Šta zatvaramo?

Najčešće:

```
Producer

↓

close(jobs)
```

Worker-i završavaju kada obrade preostale poslove.

`results` kanal se zatvara tek kada svi worker-i završe sa slanjem rezultata.

---

# Goroutine Leak

Najčešći problem.

Primer:

```go
go worker(jobs)
```

Ako:

```go
jobs
```

nikada nije zatvoren,

worker ostaje blokiran:

```go
for job := range jobs {
}
```

Takva Goroutine ostaje aktivna dok traje proces.

To nazivamo:

```
Goroutine Leak
```

---

# Vizuelno

Loše:

```text
Worker

↓

Waiting

↓

Waiting

↓

Waiting

↓

Forever
```

---

Dobro:

```text
close(jobs)

↓

range završava

↓

Worker Exit
```

---

# Da li treba prekinuti aktivan posao?

Zavisi.

Na primer:

```
Upload fajla
```

Možda želiš da se završi.

---

Ali:

```
HTTP Request

30 minuta
```

Možda želiš da ga otkažeš.

Zato `process(job)` često prima:

```go
ctx
```

na primer:

```go
func process(
	ctx context.Context,
	job Job,
) error
```

Na taj način i sama obrada može reagovati na otkazivanje.

---

# Production tok

```text
Receive SIGTERM

↓

Cancel Context

↓

Stop Producer

↓

Close Jobs

↓

Workers Finish

↓

WaitGroup

↓

Close Results

↓

Exit
```

Ovo je tipičan sled koraka u produkcionim servisima.

---

# Šta radi `WaitGroup`?

On garantuje:

```
Svi Worker-i

↓

završili
```

Tek tada je bezbedno zatvoriti kanal sa rezultatima ili završiti aplikaciju.

---

# Najčešće greške

## Greška #1

Koristiti:

```go
os.Exit(...)
```

pre nego što worker-i završe.

Time se trenutno prekidaju sve Goroutines.

---

## Greška #2

Ignorisati:

```go
ctx.Done()
```

Dugotrajne operacije nastavljaju da rade i nakon što je sistem dobio signal za gašenje.

---

## Greška #3

Ne zatvoriti `jobs` kanal.

Worker-i ostaju blokirani.

---

## Greška #4

Zatvoriti `results` kanal dok worker-i još šalju podatke.

To dovodi do panike:

```text
send on closed channel
```

---

# Best Practices

✅ Producer zatvara `jobs` kanal kada više nema novih poslova.

✅ Worker treba da reaguje i na zatvaranje kanala i na `ctx.Done()`.

✅ `WaitGroup` koristi za koordinaciju završetka svih worker-a.

✅ `results` zatvori tek nakon završetka svih worker-a.

✅ Prosledi `context.Context` i funkcijama koje mogu trajati duže vreme.

---

# Mentalni model

Nemoj razmišljati:

```text
Stop

↓

Exit
```

Razmišljaj:

```text
Stop

↓

Cancel Context

↓

Finish Work

↓

Close Channels

↓

Wait

↓

Exit
```

To je suština graceful shutdown-a.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Graceful Shutdown

✅ kako koristiti `context.Context`

✅ kako otkazati rad worker-a

✅ kako izbeći Goroutine leak

✅ pravilan redosled gašenja Worker Pool-a

---

# Worker Pools — Backpressure

---

# 📚 Sadržaj

- Šta je Backpressure
- Zašto nastaje
- Bounded Queue
- Load Shedding
- Rate Limiting
- Production strategije
- Monitoring

---

# Uvod

Zamisli sistem koji može da obradi:

```
100

zahteva

u sekundi
```

Ali prima:

```
1000

zahteva

u sekundi
```

Šta će se dogoditi?

Ako ništa ne preduzmemo:

```
Queue

↓

raste

↓

raste

↓

raste
```

Na kraju:

- memorija raste,
- latencija raste,
- sistem postaje nestabilan,
- može doći do pada procesa.

---

# Šta je Backpressure?

Backpressure je mehanizam kojim sistem:

> **usporava ili ograničava prijem novih poslova kada ne može da ih obradi dovoljno brzo.**

Drugim rečima:

```
Producer

↓

sporije šalje

↓

Consumer sustiže
```

---

# Vizuelni primer

Bez Backpressure-a:

```text
Producer

↓↓↓↓↓↓↓↓↓↓

Queue

██████████████████████████

Workers

↓↓
```

Red čekanja stalno raste.

---

Sa Backpressure-om:

```text
Producer

↓

Queue

██████

↓

Workers

↓↓↓↓
```

Red ostaje pod kontrolom.

---

# Bounded Queue

Najjednostavniji oblik backpressure-a je ograničen red čekanja.

Na primer:

```go
jobs := make(chan Job, 100)
```

Kapacitet je:

```
100
```

Kada je kanal pun:

```go
jobs <- job
```

blokira dok se ne oslobodi mesto.

Time producer prirodno usporava.

---

# Prednosti Bounded Queue

✅ Ograničena potrošnja memorije.

✅ Jednostavna implementacija.

✅ Prirodan backpressure pomoću kanala.

---

# Ograničenja

Ako producer ne sme da blokira, potrebno je drugačije ponašanje.

---

# Load Shedding

Ponekad je bolje odbaciti zahtev nego dozvoliti da ceo sistem postane nestabilan.

Primer:

```text
Queue puna

↓

Reject Request
```

To se naziva:

```
Load Shedding
```

Primeri:

- HTTP `503 Service Unavailable`,
- odbacivanje događaja niskog prioriteta,
- privremeno odbijanje novih poslova.

---

# Rate Limiting

Backpressure nije isto što i Rate Limiting.

## Backpressure

Reaguje na trenutno opterećenje sistema.

---

## Rate Limiting

Ograničava brzinu zahteva unapred.

Na primer:

```
100

zahteva

u sekundi
```

bez obzira na trenutno stanje sistema.

Često se ova dva mehanizma koriste zajedno.

---

# Strategije kada je red pun

## 1. Blokiraj producer-a

```go
jobs <- job
```

Čeka dok se ne oslobodi mesto.

---

## 2. Odbaci posao

```go
select {
case jobs <- job:
	// prihvaćen
default:
	// red je pun
}
```

Ovaj obrazac omogućava da producer nastavi bez blokiranja.

---

## 3. Timeout

```go
select {
case jobs <- job:
	// uspeh

case <-ctx.Done():
	// odustani
}
```

Može se kombinovati sa kontekstom ili vremenskim ograničenjem.

---

# Monitoring

Bez merenja teško je znati da li sistem trpi preveliko opterećenje.

Korisne metrike uključuju:

- dužinu reda čekanja,
- broj aktivnih worker-a,
- vreme čekanja posla,
- vreme obrade posla,
- broj odbačenih poslova,
- iskorišćenost CPU-a i memorije.

Ove informacije pomažu pri podešavanju veličine reda i broja worker-a.

---

# Production primer

Servis za obradu slika.

Normalno stanje:

```text
100 Uploads

↓

Queue

↓

Workers
```

Tokom velikog opterećenja:

```text
50 000 Uploads

↓

Queue puna

↓

Novi zahtevi

↓

HTTP 503
```

Sistem ostaje stabilan i nastavlja da obrađuje već prihvaćene zahteve.

---

# Najčešće greške

## Greška #1

Neograničen red čekanja.

Posledica:

- rast memorije,
- povećana latencija,
- moguć pad procesa.

---

## Greška #2

Prevelik bafer kanala.

Veliki bafer može samo odložiti problem i sakriti da sistem ne može da obradi dolazni saobraćaj.

---

## Greška #3

Ignorisati odbijene poslove.

Ako koristiš load shedding, važno je imati strategiju:

- ponovni pokušaj,
- logovanje,
- obaveštavanje korisnika,
- metrika.

---

# Best Practices

✅ Koristi ograničene redove čekanja.

✅ Definiši šta se dešava kada je red pun.

✅ Kombinuj Worker Pool sa `context.Context`.

✅ Prati ključne metrike i podešavaj sistem na osnovu merenja.

✅ Dizajniraj sistem tako da ostane stabilan i pod velikim opterećenjem.

---

# Mentalni model

Nemoj razmišljati:

```text
Prihvati

sve
```

Razmišljaj:

```text
Prihvati

onoliko

koliko sistem

može kvalitetno

da obradi
```

To je suština backpressure-a.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Backpressure

✅ zašto nastaje pri velikom opterećenju

✅ kako funkcioniše bounded queue

✅ šta je Load Shedding

✅ razliku između Backpressure-a i Rate Limiting-a

✅ koje metrike treba pratiti u produkcionim sistemima

---

# Worker Pools — Production Guidelines i završni rezime

---

# 📚 Sadržaj

- Kada koristiti Worker Pool
- Worker Pool vs Semaphore vs Pipeline
- Production checklist
- Najčešće greške
- Mentalni model
- Završni rezime

---

# Uvod

Worker Pool nije univerzalno rešenje.

Pre nego što ga uvedeš, postavi pitanje:

> **Da li zaista postoji veliki broj nezavisnih zadataka koje treba obrađivati uz ograničenu konkurentnost?**

Ako je odgovor "da", Worker Pool je često odličan izbor.

---

# Kada koristiti Worker Pool?

Koristi Worker Pool kada:

- postoji veliki broj nezavisnih poslova,
- želiš da ograničiš broj istovremenih izvršavanja,
- poslovi mogu da čekaju u redu,
- sistem mora da ostane stabilan pod velikim opterećenjem.

Primeri:

- obrada fajlova,
- generisanje PDF-ova,
- slanje email-ova,
- ETL procesi,
- obrada poruka iz message broker-a,
- batch poslovi.

---

# Kada NE koristiti Worker Pool?

Ako:

- postoji mali broj zadataka,
- zadaci moraju biti obrađeni odmah,
- nema potrebe za ograničavanjem konkurentnosti,
- jednostavnije rešenje je dovoljno.

U tim slučajevima dodatna složenost Worker Pool-a možda nije opravdana.

---

# Worker Pool vs Semaphore

## Worker Pool

```text
Jobs

↓

Queue

↓

Workers
```

Karakteristike:

- fiksan ili kontrolisan broj worker-a,
- zajednički red poslova,
- pogodna obrada velikih serija zadataka.

---

## Semaphore

```text
Acquire

↓

Work

↓

Release
```

Karakteristike:

- ograničava broj istovremenih operacija,
- ne uvodi poseban red poslova,
- koristan kada već postoji Goroutine po zadatku.

---

# Worker Pool vs Pipeline

## Worker Pool

Jedna faza obrade:

```text
Jobs

↓

Workers

↓

Results
```

---

## Pipeline

Više uzastopnih faza:

```text
Read

↓

Parse

↓

Validate

↓

Process

↓

Store
```

Svaka faza može imati sopstveni skup worker-a.

Pipeline rešava organizaciju toka obrade, dok Worker Pool rešava ograničavanje konkurentnosti unutar jedne faze.

---

# Decision Tree

```text
Veliki broj nezavisnih poslova?

↓

NE

↓

Jednostavnije rešenje

----------------------------

DA

↓

Treba ograničiti konkurentnost?

↓

NE

↓

Obične Goroutines

----------------------------

DA

↓

Jedna faza?

↓

DA

↓

Worker Pool

----------------------------

Više faza?

↓

Pipeline
```

---

# Production Checklist

Pre nego što uvedeš Worker Pool, proveri:

✅ Da li je broj worker-a definisan na osnovu merenja?

✅ Da li je `jobs` kanal pravilno zatvoren?

✅ Da li svi worker-i mogu uredno da završe?

✅ Da li koristiš `context.Context` za otkazivanje?

✅ Da li postoji strategija za backpressure?

✅ Da li pratiš ključne metrike (latencija, dužina reda, broj aktivnih worker-a)?

---

# Tipične greške Junior programera

❌ Pokretanje jedne Goroutine po svakom poslu bez ograničenja.

❌ Zaboravljanje `close(jobs)`.

❌ Ignorisanje `WaitGroup` koordinacije.

❌ Pretpostavka da su rezultati uvek u istom redosledu kao poslovi.

---

# Tipične greške Medior programera

❌ Prevelik broj worker-a bez benchmark-a.

❌ Nedostatak graceful shutdown logike.

❌ Ignorisanje backpressure-a pri velikom opterećenju.

❌ Oslanjanje na veličinu bafera umesto na pravilno upravljanje protokom poslova.

---

# Kako razmišlja Senior Go programer?

Ne pita:

> "Koliko Goroutines mogu da pokrenem?"

Pita:

- Koliko paralelnih zadataka sistem može da obradi?
- Gde je usko grlo?
- Kako će se sistem ponašati pod velikim opterećenjem?
- Šta se dešava pri gašenju aplikacije?
- Kako izgleda ponašanje kada je red pun?

---

# Mentalni model

Nemoj razmišljati:

```text
Više Goroutines

↓

Veća brzina
```

Razmišljaj:

```text
Kontrolisana konkurentnost

↓

Predvidivo ponašanje

↓

Stabilan sistem
```

To je cilj Worker Pool obrasca.

---

# Ceo put kroz Modul #4.5

```text
Worker Pool

↓

Jobs Queue

↓

Workers

↓

Bounded Concurrency

↓

Graceful Shutdown

↓

Backpressure

↓

Production Design
```

Svaka tema nadograđuje prethodnu i vodi ka robusnom produkcionom rešenju.

---

# Šta nosiš iz ovog modula?

Trebalo bi da možeš da objasniš:

- zašto Worker Pool postoji,
- kako implementirati osnovni Worker Pool,
- kako odabrati broj worker-a,
- razliku između CPU-bound i I/O-bound workload-a,
- kako implementirati graceful shutdown,
- kako sprečiti Goroutine leak,
- šta je backpressure i zašto je važan,
- kada koristiti Worker Pool, Semaphore ili Pipeline.

---

# 📋 Rezime Modula #4.5

U ovom modulu naučili smo:

✅ šta je Worker Pool

✅ kako implementirati Worker Pool u Go-u

✅ šta znači bounded concurrency

✅ kako odrediti broj worker-a

✅ kako implementirati graceful shutdown

✅ kako izbeći Goroutine leak

✅ šta je backpressure

✅ razliku između Worker Pool-a, Semaphore-a i Pipeline-a

---

### ➡️ Sledeća lekcija **[**Pipelines**](06-pipelines.md)**

Obuhvatiće:

- šta je Pipeline obrazac,
- povezivanje više faza obrade pomoću kanala,
- fan-out i fan-in u okviru pipeline-a,
- propagacija grešaka,
- otkazivanje pomoću `context.Context`,
- projektovanje skalabilnih i modularnih tokova obrade podataka.