# Fan-out

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 6/8
>
> **Fajl:** `docs/module-2/06-fan-out.md`

---

# 📚 Sadržaj

- Šta je Fan-out?
- Zašto postoji?
- Problem koji rešava
- Kako funkcioniše?
- Fan-out arhitektura
- Fan-out vs Worker Pool
- Fan-out vs Pipeline
- Jedan Channel → više Goroutines
- Redosled izvršavanja
- Kada koristiti Fan-out?
- Kada ga izbegavati?
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja **Fan-out** obrazac,
- kako više Goroutines deli posao,
- kako se poslovi raspodeljuju između Worker-a,
- po čemu se Fan-out razlikuje od Worker Pool-a,
- kako se Fan-out koristi u realnim Go aplikacijama.

---

# Uvod

Do sada smo upoznali:

- Goroutines
- Channels
- `select`
- `sync.WaitGroup`
- Worker Pool
- Pipeline

Sada upoznajemo još jedan veoma važan concurrency obrazac:

> **Fan-out**

---

# Šta je Fan-out?

Fan-out predstavlja obrazac u kome:

> **jedan izvor podataka šalje posao većem broju Goroutines koje ga obrađuju paralelno.**

Jedan ulaz.

↓

Više izvršilaca.

---

# Vizuelni prikaz

```text
          Jobs

            │

            ▼

      +-------------+

      |   Channel   |

      +-------------+

       │    │    │

       ▼    ▼    ▼

      G1   G2   G3

       │    │    │

       ▼    ▼    ▼

   Obrada poslova
```

---

# Zašto naziv "Fan-out"?

Naziv dolazi od oblika lepeze.

Jedan tok podataka se "širi".

Vizuelno:

```text
      Jedan

        │

        ▼

 ┌──────┼──────┐

 ▼      ▼      ▼

 G1     G2     G3
```

---

# Problem

Pretpostavimo da imamo:

```
1000 zadataka
```

Jedna Goroutine obrađuje:

```
1 zadatak
```

↓

sledeći

↓

sledeći

↓

sledeći

To može biti sporo.

---

# Rešenje

Pokrenimo više Goroutines.

Na primer:

```
4
```

Sve čitaju sa istog Channel-a.

Svaka uzima sledeći slobodan posao.

---

# Primer

```go
jobs := make(chan int)

for i := 0; i < 4; i++ {

	go worker(jobs)

}
```

Sada četiri Worker-a dele isti Channel.

---

# Kako se raspodeljuju poslovi?

Pretpostavimo da šaljemo:

```text
1
2
3
4
5
6
```

Jedan mogući raspored:

```text
Worker 1

↓

1

↓

5

---------------

Worker 2

↓

2

↓

6

---------------

Worker 3

↓

3

---------------

Worker 4

↓

4
```

---

# Da li je raspodela garantovana?

Ne.

Go Scheduler odlučuje.

Možemo dobiti potpuno drugačiji raspored.

Na primer:

```text
Worker 3

↓

1

↓

2

↓

6

---------------

Worker 1

↓

3

---------------

Worker 2

↓

4

↓

5
```

Oba rasporeda su potpuno ispravna.

---

# Primer programa

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, wg *sync.WaitGroup) {

	defer wg.Done()

	for job := range jobs {

		fmt.Printf(
			"Worker %d processed %d\n",
			id,
			job,
		)

	}

}

func main() {

	jobs := make(chan int)

	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {

		wg.Add(1)

		go worker(i, jobs, &wg)

	}

	for i := 1; i <= 10; i++ {

		jobs <- i

	}

	close(jobs)

	wg.Wait()

}
```

---

# Analiza

Korak 1

Pokreću se tri Worker-a.

↓

Korak 2

Svi čekaju podatke.

↓

Korak 3

Main šalje poslove.

↓

Korak 4

Prvi slobodan Worker uzima posao.

↓

Korak 5

Obrada se nastavlja dok se Channel ne zatvori.

↓

Korak 6

Svi Worker-i izlaze iz `range` petlje.

---

# Zašto koristiti jedan Channel?

Jedan zajednički Channel omogućava:

- jednostavniji kod,
- automatsku raspodelu poslova,
- manje sinhronizacije.

Ne moramo ručno odlučivati:

```
Job 1

↓

Worker 2
```

Go Runtime to radi umesto nas.

---

# Fan-out i Worker Pool

Početnici ih često smatraju istim.

Nisu isto.

---

## Fan-out

Opisuje:

> raspodelu poslova na više izvršilaca.

---

## Worker Pool

Predstavlja:

- Fan-out,
- kontrolisan broj Worker-a,
- često Results Channel,
- često WaitGroup,
- često upravljanje životnim ciklusom Worker-a.

Drugim rečima:

> **Fan-out je osnovna ideja koju Worker Pool koristi.**

---

# Vizuelno poređenje

Fan-out:

```text
Jobs

↓

Workers
```

Worker Pool:

```text
Jobs

↓

Workers

↓

Results

↓

Consumer
```

---

# Fan-out i Pipeline

Pipeline:

```text
Stage A

↓

Stage B

↓

Stage C
```

Fan-out:

```text
Stage

↓

više Worker-a
```

Pipeline opisuje:

tok obrade.

Fan-out opisuje:

paralelnu obradu jedne faze.

---

# Gde se koristi Fan-out?

Veoma često u:

- HTTP serverima,
- obradi logova,
- obradi slika,
- obradi CSV fajlova,
- indeksiranju podataka,
- slanju e-mail poruka,
- obradi poruka iz Message Queue sistema.

---

# Prednosti

## Veća propusnost

Više poslova može biti obrađeno istovremeno.

---

## Bolje iskorišćenje CPU-a

Posebno kod CPU-bound poslova.

---

## Jednostavna implementacija

Najčešće je dovoljan:

- jedan Channel,
- više Goroutines.

---

## Skalabilnost

Broj Worker-a se lako povećava ili smanjuje.

---

# Ograničenja

Fan-out ne garantuje:

- redosled,
- isti broj poslova po Worker-u,
- ravnomerno opterećenje.

To zavisi od:

- Scheduler-a,
- trajanja svakog posla,
- dostupnosti CPU-a.

---

# Kada koristiti Fan-out?

Koristi ga kada:

- ima mnogo nezavisnih poslova,
- redosled nije važan,
- obrada može biti paralelna.

---

# Kada ga izbegavati?

Nemoj koristiti Fan-out kada:

- poslovi moraju strogo redom,
- postoji jaka međuzavisnost zadataka,
- broj poslova je veoma mali,
- paralelizacija ne donosi korist.

---

# 🚨 Najčešće greške

## 1. Očekivanje ravnomerne raspodele

Ne postoji garancija da će svaki Worker dobiti isti broj poslova.

---

## 2. Očekivanje redosleda

Fan-out ne garantuje:

```
1

↓

2

↓

3
```

Redosled obrade nije definisan.

---

## 3. Kreiranje previše Worker-a

Hiljade Worker-a često usporavaju program.

Više nije uvek bolje.

---

## 4. Zaboravljanje zatvaranja Channel-a

Worker koristi:

```go
range jobs
```

Bez:

```go
close(jobs)
```

Worker nikada neće završiti.

---

## 5. Deljenje promenljivih bez zaštite

Ako više Worker-a menja istu promenljivu,

nastaje Data Race.

Kasnije ćemo obraditi:

- `sync.Mutex`
- `atomic`
- Data Race.

---

# ✅ Best Practices

- Koristi jedan zajednički `jobs` Channel.
- Koristi `range` za čitanje poslova.
- Zatvori `jobs` kada više nema novih zadataka.
- Ne oslanjaj se na redosled izvršavanja.
- Prilagodi broj Worker-a prirodi posla.
- Kombinuj Fan-out sa Worker Pool i Pipeline obrascima kada je potrebno.

---

# 📋 Rezime

- Fan-out raspodeljuje jedan tok poslova na više Goroutines.
- Sve Goroutines čitaju sa istog Channel-a.
- Poslovi se raspodeljuju automatski.
- Redosled izvršavanja nije garantovan.
- Fan-out predstavlja osnovu mnogih Worker Pool implementacija.
- Idealan je za veliki broj nezavisnih zadataka.

---

# ❓ Pitanja za proveru znanja

1. Šta predstavlja Fan-out?
2. Koji problem rešava?
3. Da li više Worker-a čita sa istog Channel-a?
4. Da li Go garantuje ravnomernu raspodelu poslova?
5. Da li Fan-out garantuje redosled izvršavanja?
6. Po čemu se razlikuje od Worker Pool-a?
7. Po čemu se razlikuje od Pipeline-a?
8. Kada je Fan-out dobar izbor?
9. Kada ga treba izbegavati?
10. Zašto se često koristi zajedno sa `sync.WaitGroup`?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi Fan-out sa dva Worker-a koji ispisuju brojeve sa jednog Channel-a.
2. Povećaj broj Worker-a na četiri i posmatraj raspodelu poslova.
3. Dodaj ID svakog Worker-a u ispis.

---

## 🟡 Srednje

4. Napravi Fan-out koji računa kvadrat svakog broja.
5. Dodaj `results` Channel i sakupi rezultate u glavnoj Goroutine.
6. Eksperimentiši sa različitim brojem Worker-a (2, 4, 8) i uporedi ponašanje programa.

---

## 🟠 Izazov

7. Napravi program koji simulira sistem za obradu slika:

- Generator šalje ID slike kroz `jobs` Channel.
- Šest Worker-a paralelno "obrađuje" slike (simuliraj obradu kratkim `time.Sleep()`).
- Rezultati se šalju na `results` Channel.
- Glavna Goroutine prikuplja i ispisuje sve obrađene slike.
- Koristi `sync.WaitGroup` za pravilno gašenje sistema.

---

### ➡️ Sledeća lekcija **[**Fan-in**](07-fan-in.md)**

naučićeš **Fan-in** obrazac, odnosno kako spojiti rezultate više Goroutines u jedan zajednički Channel. Fan-in i Fan-out se veoma često koriste zajedno i predstavljaju osnovu mnogih naprednih concurrency arhitektura u Go-u.
