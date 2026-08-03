# Worker Pools

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 4/8
>
> **Fajl:** `docs/module-2/04-worker-pools.md`

---

# 📚 Sadržaj

- Šta je Worker Pool?
- Problem koji rešava
- Zašto nam je potreban?
- Osnovna arhitektura
- Producer → Workers → Consumer
- Jobs Channel
- Results Channel
- Jedan Worker
- Više Worker-a
- `sync.WaitGroup`
- Analiza izvršavanja
- Kako Worker Pool funkcioniše iza kulisa
- Results Channel
- Kompletan Worker Pool
- Analiza izvršavanja
- Zatvaranje `jobs` i `results` Channel-a
- Tipične greške
- Performance razmatranja
- Kada koristiti Worker Pool?
- Kada ga izbegavati?
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon prvog dela ove lekcije znaćeš:

- šta predstavlja Worker Pool,
- koji problem rešava,
- zašto je jedan od najvažnijih concurrency obrazaca u Go-u,
- kako međusobno sarađuju Goroutines i Channels,
- kako izgleda osnovna arhitektura Worker Pool-a.

---

# Uvod

Do sada smo naučili:

- Goroutines
- Channels
- `select`
- `default`
- `sync.WaitGroup`

Sada konačno imamo dovoljno znanja da napravimo prvi ozbiljan concurrency obrazac koji se veoma često koristi u produkcionim Go aplikacijama.

To je:

# Worker Pool

---

# Šta je Worker Pool?

Worker Pool predstavlja grupu Goroutines koje zajedno izvršavaju veliki broj zadataka.

Umesto da za svaki posao kreiramo novu Goroutine,

imamo unapred definisan broj Worker-a.

Svi Worker-i uzimaju posao iz zajedničkog Channel-a.

---

# Vizuelni prikaz

```text
                Jobs

                  │

                  ▼

        +------------------+

        |      Channel     |

        +------------------+

          │      │      │

          ▼      ▼      ▼

      Worker  Worker  Worker

          │      │      │

          └──┬───┴───┬──┘

             ▼       ▼

          Results Channel

                │

                ▼

            Main Goroutine
```

---

# Šta Worker Pool rešava?

Pretpostavimo da imaš:

```
100.000 poslova
```

Prva ideja početnika je:

```go
for _, job := range jobs {

	go process(job)

}
```

Da li radi?

Da.

Da li je dobro?

Vrlo često nije.

---

# Problem

Ako pokreneš:

```
100.000
```

Goroutines,

Go će ih rasporediti.

Iako su Goroutines veoma jeftine,

one ipak troše:

- memoriju,
- Scheduler vreme,
- CPU vreme.

Ako posao traje dugo,

možeš imati ogroman broj aktivnih Goroutines.

---

# Bolja ideja

Šta ako kažeš:

```
Koristi samo 8 Worker-a.
```

ili:

```
Koristi samo 20 Worker-a.
```

ili:

```
Koristi samo broj jednak CPU jezgrima.
```

Tada:

- broj Goroutines ostaje mali,
- Worker-i obrađuju poslove jedan za drugim,
- program ostaje stabilan.

---

# Analogija

Zamisli restoran.

Dolazi:

```
500 gostiju
```

Da li restoran zapošljava:

```
500 konobara?
```

Naravno da ne.

Možda ima:

```
10 konobara.
```

Svaki konobar uzima sledeći slobodan sto.

To je upravo Worker Pool.

---

# Još jedna analogija

Pošta.

Stiglo je:

```
20.000 paketa.
```

Ne zapošljava se:

```
20.000 radnika.
```

Već:

```
30 radnika.
```

Svaki uzima naredni paket.

---

# Osnovna arhitektura

Svaki Worker Pool ima četiri dela.

```text
Producer

↓

Jobs Channel

↓

Workers

↓

Results Channel

↓

Consumer
```

---

# Komponente

## Producer

Generiše poslove.

Na primer:

```text
1

2

3

4

5

...
```

---

## Jobs Channel

Čuva poslove koje Worker-i treba da izvrše.

---

## Worker

Uzima jedan posao.

Obrađuje ga.

Vraća rezultat.

Pa uzima sledeći posao.

---

## Results Channel

Sadrži rezultate koje je napravio Worker.

---

## Consumer

Prima rezultate.

Može:

- ispisivati,
- čuvati u bazu,
- slati preko mreže,
- obrađivati dalje.

---

# Vizuelni tok

```text
Producer

↓

Jobs

↓

Worker 1

↓

Result

↓

Consumer

────────────

Producer

↓

Jobs

↓

Worker 2

↓

Result

↓

Consumer
```

---

# Prvi Worker

Počnimo veoma jednostavno.

```go
func worker(jobs <-chan int) {

	for job := range jobs {

		fmt.Println(job)

	}

}
```

Ovde Worker:

- čeka posao,
- uzima posao,
- obrađuje posao,
- vraća se na čekanje.

---

# Vizuelni prikaz

```text
WAIT

↓

Job

↓

Work

↓

WAIT

↓

Job

↓

Work
```

---

# Zašto koristimo `range`?

Zato što Worker ne zna:

- koliko će poslova stići,
- kada će stići.

Najprirodnije rešenje je:

```go
for job := range jobs {

}
```

---

# Prvi kompletan primer

```go
package main

import (
	"fmt"
)

func worker(jobs <-chan int) {

	for job := range jobs {

		fmt.Println("Processing:", job)

	}

}

func main() {

	jobs := make(chan int)

	go worker(jobs)

	for i := 1; i <= 5; i++ {

		jobs <- i

	}

	close(jobs)

}
```

Mogući izlaz:

```text
Processing: 1
Processing: 2
Processing: 3
Processing: 4
Processing: 5
```

---

# Analiza

Korak 1

Pokreće se Worker.

---

Korak 2

Worker čeka posao.

---

Korak 3

Producer šalje:

```
1
```

---

Korak 4

Worker obrađuje posao.

---

Korak 5

Ponovo čeka.

---

Korak 6

Kada se Channel zatvori,

`range`

se završava.

Worker izlazi.

---

# Jedan Worker nije Pool

Iako ovaj primer radi,

ovo još nije pravi Worker Pool.

Zašto?

Imamo:

```
1 Worker
```

Pool znači:

```
više Worker-a.
```

---

# Dodavanje više Worker-a

To je iznenađujuće jednostavno.

```go
for i := 1; i <= 3; i++ {

	go worker(jobs)

}
```

Sada:

```
Worker 1

Worker 2

Worker 3
```

čitaju sa istog Channel-a.

---

# Šta se sada dešava?

Pretpostavimo da Producer šalje:

```
1

2

3

4

5

6
```

Jedan mogući raspored:

```text
Worker1

↓

1

↓

4

Worker2

↓

2

↓

5

Worker3

↓

3

↓

6
```

Drugi raspored može izgledati potpuno drugačije.

---

# Da li možemo znati koji Worker će dobiti koji posao?

Ne.

Go Scheduler odlučuje.

Nikada ne treba pretpostavljati:

```
Worker 1

↓

Job 1
```

To nije garantovano.

---

# Worker funkcija

U praksi Worker često izgleda ovako:

```go
func worker(id int, jobs <-chan int) {

	for job := range jobs {

		fmt.Println(
			"Worker",
			id,
			"processing",
			job,
		)

	}

}
```

Pokretanje:

```go
for i := 1; i <= 3; i++ {

	go worker(i, jobs)

}
```

Mogući izlaz:

```text
Worker 2 processing 1
Worker 1 processing 2
Worker 3 processing 3
Worker 2 processing 4
Worker 1 processing 5
```

Redosled nije garantovan.

---

# Gde je `sync.WaitGroup`?

Ako `main()` završi,

program se prekida.

Zato ćemo koristiti:

```go
var wg sync.WaitGroup
```

---

Pokretanje Worker-a:

```go
wg.Add(1)

go func() {

	defer wg.Done()

	worker(id, jobs)

}()
```

Na kraju:

```go
wg.Wait()
```

Sada će `main()` sačekati završetak svih Worker-a.

---

# Životni ciklus Worker Pool-a

```text
Pokretanje Worker-a

↓

Workers čekaju

↓

Producer šalje poslove

↓

Workers obrađuju

↓

Producer zatvara jobs

↓

Workers završavaju

↓

WaitGroup čeka

↓

Program završava
```

---

# Važno zapažanje

Worker nikada ne pita:

```
Ima li posla?
```

On jednostavno radi:

```go
for job := range jobs
```

Ako posao postoji,

obrađuje ga.

Ako ga nema,

čeka.

Ako je Channel zatvoren,

završava rad.

To je jedan od razloga zbog kojih su Channel-i toliko elegantni u Go-u.

---

# Podsećanje

Na kraju prethodne lekcije imali smo:

```text
Producer

↓

Jobs Channel

↓

Worker-i

↓

(sync.WaitGroup)

↓

Kraj programa
```

Međutim, nešto nedostaje.

Worker-i obrađuju posao,

ali gde završavaju rezultati?

---

# Results Channel

U ozbiljnim aplikacijama Worker obično ne ispisuje rezultat.

Umesto toga,

šalje ga na drugi Channel.

```text
Jobs

↓

Worker

↓

Results
```

Na taj način:

- Worker obrađuje posao,
- Consumer odlučuje šta će sa rezultatom.

---

# Vizuelni prikaz

```text
             Jobs

              │

              ▼

      +----------------+

      | Jobs Channel   |

      +----------------+

        │     │     │

        ▼     ▼     ▼

      W1     W2     W3

        │     │     │

        └──┬──┴──┬──┘

           ▼     ▼

      Results Channel

              │

              ▼

          Consumer
```

---

# Worker sa rezultatima

```go
func worker(
	id int,
	jobs <-chan int,
	results chan<- int,
) {

	for job := range jobs {

		results <- job * job

	}

}
```

Sada Worker:

- primi posao,
- obradi ga,
- pošalje rezultat.

---

# Producer

Producer više ne razgovara direktno sa Worker-ima.

On samo puni:

```go
jobs
```

Na primer:

```go
for i := 1; i <= 5; i++ {

	jobs <- i

}
```

Nakon toga:

```go
close(jobs)
```

---

# Consumer

Consumer čita rezultate.

```go
for result := range results {

	fmt.Println(result)

}
```

---

# Zašto dva Channel-a?

Jedan Channel bi mogao da nosi i poslove i rezultate.

Ali to nije dobra praksa.

Bolje je odvojiti odgovornosti.

```text
jobs

↓

ulaz

----------------

results

↓

izlaz
```

Kod je jednostavniji za razumevanje.

---

# Kompletan Worker Pool

```go
package main

import (
	"fmt"
	"sync"
)

func worker(
	id int,
	jobs <-chan int,
	results chan<- int,
	wg *sync.WaitGroup,
) {

	defer wg.Done()

	for job := range jobs {

		results <- job * job

	}
}

func main() {

	jobs := make(chan int)
	results := make(chan int)

	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {

		wg.Add(1)

		go worker(i, jobs, results, &wg)

	}

	go func() {

		for i := 1; i <= 10; i++ {

			jobs <- i

		}

		close(jobs)

	}()

	go func() {

		wg.Wait()

		close(results)

	}()

	for result := range results {

		fmt.Println(result)

	}
}
```

---

# Zašto zatvaramo `results`?

Pogledaj Consumer.

```go
for result := range results {

}
```

Ova petlja traje dok se Channel ne zatvori.

Ako:

```go
close(results)
```

nikada ne bude pozvan,

Consumer će čekati zauvek.

---

# Zašto Worker ne zatvara `results`?

Pretpostavimo da imamo:

```
Worker 1

Worker 2

Worker 3
```

Ako:

```
Worker 1
```

pozove:

```go
close(results)
```

dok ostali još rade,

sledeće slanje izazivaće:

```text
panic:

send on closed channel
```

---

# Ko zatvara `results`?

Odgovor:

Onaj deo programa koji zna da su:

```
SVI Worker-i završili.
```

Zato koristimo:

```go
wg.Wait()

↓

close(results)
```

---

# Analiza izvršavanja

Korak 1

Pokreću se Worker-i.

---

Korak 2

Svi čekaju posao.

---

Korak 3

Producer puni:

```text
jobs
```

---

Korak 4

Prvi slobodan Worker uzima posao.

---

Korak 5

Rezultat ide na:

```text
results
```

---

Korak 6

Consumer prima rezultat.

---

Korak 7

Kada više nema poslova:

```go
close(jobs)
```

---

Korak 8

Worker-i završavaju.

Pozivaju:

```go
Done()
```

---

Korak 9

`WaitGroup`

postaje:

```
0
```

---

Korak 10

Zatvara se:

```go
results
```

---

Korak 11

Consumer izlazi iz:

```go
range
```

Program se završava.

---

# Životni ciklus

```text
Workers

↓

čekaju

↓

Jobs

↓

obrađuju

↓

Results

↓

Consumer

↓

Workers završavaju

↓

WaitGroup

↓

results se zatvara

↓

Program završava
```

---

# Šta određuje broj Worker-a?

Ne postoji univerzalan odgovor.

Zavisi od vrste posla.

---

## CPU-bound poslovi

Na primer:

- kompresija,
- enkripcija,
- obrada slike,
- matematika.

Često je dobar izbor broj Worker-a približan broju logičkih CPU jezgara.

---

## I/O-bound poslovi

Na primer:

- HTTP zahtevi,
- baza podataka,
- čitanje fajlova,
- mrežna komunikacija.

Može imati više Worker-a nego CPU jezgara,

jer veliki deo vremena provode čekajući.

---

# Da li više Worker-a znači bolje performanse?

Ne.

Na primer:

```
4 Worker-a
```

može biti brže od:

```
100 Worker-a.
```

Zašto?

Više Worker-a znači:

- više prebacivanja između Goroutines,
- više Scheduler posla,
- više memorije.

Optimalan broj zavisi od konkretnog problema.

---

# Worker Pool nije zamena za...

Nemoj koristiti Worker Pool kada:

- imaš samo jedan posao,
- imaš svega nekoliko kratkih poslova,
- posao mora strogo redom da se izvršava,
- paralelizacija ne donosi korist.

---

# Kada koristiti Worker Pool?

Koristi ga kada imaš:

- veliki broj poslova,
- ograničene resurse,
- nezavisne zadatke,
- potrebu za kontrolom broja Goroutines.

Primeri:

- obrada fajlova,
- slanje e-mail poruka,
- obrada HTTP zahteva,
- generisanje PDF dokumenata,
- obrada slika,
- obrada logova,
- import velikih skupova podataka.

---

# Kada izbegavati Worker Pool?

Ako:

- postoji samo jedan posao,
- posao traje nekoliko mikrosekundi,
- redosled izvršavanja je presudan,
- postoji jaka međuzavisnost između zadataka.

---

# Česte greške

## ❌ Jedna Goroutine po poslu

```go
for _, job := range jobs {

	go process(job)

}
```

Za mali broj poslova može biti prihvatljivo.

Za desetine ili stotine hiljada poslova često nije.

---

## ❌ Worker zatvara `results`

Pogrešno.

To može izazvati:

```text
panic:
send on closed channel
```

---

## ❌ Zaboravljen `close(jobs)`

Worker-i koriste:

```go
range jobs
```

Ako Channel nikada nije zatvoren,

Worker-i čekaju zauvek.

---

## ❌ Zaboravljen `close(results)`

Consumer koristi:

```go
range results
```

Bez zatvaranja,

petlja nikada neće završiti.

---

## ❌ Izostavljen `WaitGroup`

Bez njega,

`main()`

može završiti pre Worker-a.

---

# Production obrazac

Najčešće ćeš viđati upravo ovu strukturu:

```text
Producer

↓

jobs

↓

Worker Pool

↓

results

↓

Consumer

↓

WaitGroup

↓

close(results)
```

Ovo predstavlja standardni Go idiom.

---

# Best Practices

- Koristi jedan `jobs` Channel za sve Worker-e.
- Koristi poseban `results` Channel.
- Worker treba da bude što jednostavniji.
- Producer zatvara `jobs`.
- `results` zatvara onaj ko zna da su svi Worker-i završili.
- Koristi `sync.WaitGroup`.
- Ne pretpostavljaj redosled izvršavanja.
- Broj Worker-a prilagodi vrsti posla.

---

# 📋 Rezime

- Worker Pool ograničava broj aktivnih Goroutines.
- Više Worker-a deli isti `jobs` Channel.
- Svaki Worker uzima sledeći dostupan posao.
- Ne postoji garantovan redosled raspodele poslova.
- `range` predstavlja prirodan način čitanja poslova.
- `sync.WaitGroup` omogućava da `main()` sačeka završetak svih Worker-a.
- Worker Pool predstavlja osnovu mnogih production sistema u Go-u.
- Worker Pool ograničava broj aktivnih Goroutines.
- Worker-i dele isti `jobs` Channel.
- Rezultati se šalju preko posebnog `results` Channel-a.
- `jobs` zatvara Producer.
- `results` se zatvara tek nakon završetka svih Worker-a.
- `sync.WaitGroup` omogućava bezbedno gašenje Worker Pool-a.
- Worker Pool predstavlja jedan od najčešćih concurrency obrazaca u Go-u.

---

# ❓ Pitanja za proveru znanja

1. Zašto koristimo dva Channel-a?
2. Ko zatvara `jobs`?
3. Ko zatvara `results`?
4. Zašto Worker ne sme da zatvori `results`?
5. Zašto je potreban `WaitGroup`?
6. Da li više Worker-a uvek znači bolje performanse?
7. Kako se bira broj Worker-a?
8. Kada Worker Pool nije dobro rešenje?
9. Zašto Worker koristi `range jobs`?
10. Koje su glavne komponente Worker Pool-a?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi Worker Pool sa dva Worker-a koji računaju kvadrat broja.
2. Dodaj `results` Channel i ispiši rezultate.
3. Dodaj `WaitGroup` i pravilno zatvori `results`.

---

## 🟡 Srednje

4. Napravi četiri Worker-a koji računaju kub broja.
5. Dodaj Buffered `jobs` Channel i uporedi ponašanje sa Unbuffered Channel-om.
6. Dodaj identifikator Worker-a u ispis kako bi video raspodelu poslova.

---

## 🟠 Izazov

7. Napravi Worker Pool koji obrađuje 100 zadataka. Svaki zadatak treba da simulira obradu pomoću kratkog `time.Sleep()` (isključivo radi simulacije trajanja posla). Program treba da:
   - koristi 5 Worker-a,
   - šalje rezultate preko `results` Channel-a,
   - pravilno zatvori oba Channel-a,
   - koristi `sync.WaitGroup`,
   - na kraju izračuna zbir svih rezultata.

---

### ➡️ Sledeća lekcija **[**Pipelines**](05-pipelines.md)**

naučićeš šta su **Pipelines**, kako se više faza obrade povezuje pomoću Channel-a i zašto predstavljaju jedan od najmoćnijih obrazaca za izgradnju konkurentnih Go aplikacija.
