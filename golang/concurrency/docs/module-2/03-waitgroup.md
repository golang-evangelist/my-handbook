# `sync.WaitGroup`

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 3/8
>
> **Fajl:** `docs/module-2/03-waitgroup.md`

---

# 📚 Sadržaj

- Šta je `sync.WaitGroup`?
- Zašto postoji?
- Problem koji rešava
- Kako funkcioniše?
- `Add()`
- `Done()`
- `Wait()`
- Životni ciklus `WaitGroup`-a
- Više Goroutines
- `defer wg.Done()`
- `WaitGroup` i Channels
- Ograničenja `WaitGroup`-a
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja `sync.WaitGroup`,
- zašto je jedan od najčešće korišćenih tipova iz `sync` paketa,
- kako pravilno koristiti metode `Add()`, `Done()` i `Wait()`,
- zašto je `WaitGroup` mnogo bolje rešenje od `time.Sleep()` za čekanje završetka Goroutines,
- koje su najčešće greške pri njegovom korišćenju.

---

# Uvod

Do sada smo naučili kako da pokrenemo Goroutine.

```go
go worker()
```

Međutim, pojavljuje se novi problem.

Kako da znamo kada je ta Goroutine završila svoj posao?

---

# Problem

Posmatraj sledeći program.

```go
package main

import "fmt"

func main() {

	go func() {

		fmt.Println("Worker finished")

	}()

}
```

Šta će se ispisati?

Odgovor:

Možda:

```text
Worker finished
```

A možda:

ništa.

---

# Zašto?

Kada funkcija:

```go
main()
```

završi izvršavanje,

završava se ceo program.

Go Runtime neće čekati ostale Goroutines.

---

# Loše rešenje

Početnici često napišu:

```go
time.Sleep(time.Second)
```

ili

```go
time.Sleep(5 * time.Second)
```

To izgleda ovako:

```go
go worker()

time.Sleep(time.Second)
```

Radi?

Ponekad.

Ali nije ispravno.

---

# Zašto je `time.Sleep()` loš?

Zato što:

- ne znaš koliko posao traje,
- možda je 1 sekunda premalo,
- možda je 10 sekundi previše,
- usporava program,
- nije mehanizam sinhronizacije.

`time.Sleep()` služi za pauzu, a ne za koordinaciju Goroutines.

---

# Rešenje

Go nudi:

```go
sync.WaitGroup
```

Njegova uloga je jednostavna.

> Sačekaj da određeni broj Goroutines završi.

---

# Šta je `WaitGroup`?

`WaitGroup` je brojač.

Na početku kažeš:

```
Čekaću 3 Goroutines.
```

Svaka završena Goroutine smanjuje brojač.

Kada brojač postane:

```
0
```

program nastavlja dalje.

---

# Vizuelni prikaz

```text
Add(3)

↓

Counter = 3

↓

Done()

↓

Counter = 2

↓

Done()

↓

Counter = 1

↓

Done()

↓

Counter = 0

↓

Wait()

nastavlja izvršavanje
```

---

# Kako se kreira?

```go
var wg sync.WaitGroup
```

To je dovoljno.

---

# Tri najvažnije metode

`WaitGroup` ima tri metode koje ćeš koristiti gotovo uvek.

```go
wg.Add(...)
```

Dodaje broj Goroutines koje treba sačekati.

---

```go
wg.Done()
```

Govori da je jedna Goroutine završila.

---

```go
wg.Wait()
```

Blokira dok brojač ne postane nula.

---

# Prvi primer

```go
package main

import (
	"fmt"
	"sync"
)

func main() {

	var wg sync.WaitGroup

	wg.Add(1)

	go func() {

		defer wg.Done()

		fmt.Println("Worker finished")

	}()

	wg.Wait()

	fmt.Println("Main finished")
}
```

Izlaz:

```text
Worker finished
Main finished
```

---

# Analiza

Korak 1

Kreiramo:

```go
var wg sync.WaitGroup
```

---

Korak 2

Kažemo:

```go
wg.Add(1)
```

Brojač postaje:

```
1
```

---

Korak 3

Pokreće se Goroutine.

---

Korak 4

Na kraju Goroutine poziva:

```go
wg.Done()
```

što je isto kao:

```go
wg.Add(-1)
```

Brojač postaje:

```
0
```

---

Korak 5

Pošto je brojač nula,

`Wait()`

nastavlja izvršavanje.

---

# Kako rade `Add()` i `Done()`?

Zamisli da organizuješ tim.

Na početku kažeš:

```
Tri osobe imaju zadatak.
```

To je:

```go
wg.Add(3)
```

Kada prva osoba završi:

```go
wg.Done()
```

Preostalo:

```
2
```

Kada svi završe:

```
0
```

Možeš nastaviti dalje.

---

# Više Goroutines

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {

	defer wg.Done()

	fmt.Println("Worker", id)

}

func main() {

	var wg sync.WaitGroup

	for i := 1; i <= 5; i++ {

		wg.Add(1)

		go worker(i, &wg)

	}

	wg.Wait()

	fmt.Println("All workers finished")
}
```

Mogući izlaz:

```text
Worker 2
Worker 5
Worker 1
Worker 4
Worker 3
All workers finished
```

---

# Zašto redosled nije isti?

Zbog Go Schedulera.

Redosled izvršavanja Goroutines nije garantovan.

`WaitGroup`

ne određuje redosled.

On samo čeka završetak.

---

# Zašto prosleđujemo pokazivač?

Pogledaj:

```go
worker(i, &wg)
```

Zašto ne:

```go
worker(i, wg)
```

Zato što sve Goroutines moraju koristiti isti `WaitGroup`.

Ako bi svaka dobila svoju kopiju,

brojač ne bi bio zajednički.

---

# `defer wg.Done()`

Najčešći idiom u Go-u.

```go
go func() {

	defer wg.Done()

	// posao

}()
```

Zašto?

Ako se funkcija završi ranije:

```go
return
```

ili ima više izlaznih tačaka,

`Done()` će se sigurno izvršiti.

---

# Šta ako zaboravimo `Done()`?

Primer:

```go
wg.Add(1)

go func() {

	fmt.Println("Finished")

}()
```

Program će doći do:

```go
wg.Wait()
```

i čekati zauvek.

Brojač nikada neće postati nula.

---

# Vizuelni prikaz

```text
Counter = 1

↓

Done() nije pozvan

↓

Wait()

↓

WAIT

↓

WAIT

↓

WAIT
```

---

# Šta ako pozovemo `Done()` previše puta?

Primer:

```go
wg.Add(1)

wg.Done()

wg.Done()
```

Rezultat:

```text
panic:
sync: negative WaitGroup counter
```

Brojač ne sme postati negativan.

---

# Šta ako zaboravimo `Add()`?

Primer:

```go
go func() {

	wg.Done()

}()
```

Rezultat:

```text
panic:
sync: negative WaitGroup counter
```

Pošto brojač nikada nije povećan.

---

# Da li možemo pozvati `Wait()` više puta?

Da.

Ako je brojač:

```
0
```

`Wait()`

se odmah završava.

---

# `WaitGroup` i Channels

Vrlo često rade zajedno.

Na primer:

```text
Workers

↓

Channels

↓

WaitGroup

↓

close(channel)
```

Čest obrazac izgleda ovako:

1. Pokrenu se Worker-i.
2. Svaki pozove `Done()`.
3. `Wait()` sačeka sve Worker-e.
4. Tek tada se zatvara Channel.

Ovaj obrazac ćemo koristiti u narednim lekcijama o Worker Pool-ovima i Pipeline-ima.

---

# Ograničenja `WaitGroup`-a

`WaitGroup` ume samo jedno.

Da čeka završetak Goroutines.

Ne može:

- prenositi podatke,
- otkazivati rad,
- slati greške,
- ograničavati broj Goroutines,
- sinhronizovati pristup deljenoj memoriji.

Za te probleme postoje:

- Channels,
- `context.Context`,
- `sync.Mutex`,
- `sync.RWMutex`,
- Semaphore Pattern.

---

# Životni ciklus

```text
Kreiranje

↓

Add()

↓

Pokretanje Goroutines

↓

Done()

↓

Wait()

↓

Program nastavlja
```

---

# 🚨 Najčešće greške

## 1. Korišćenje `time.Sleep()`

Nemoj.

Koristi:

```go
wg.Wait()
```

---

## 2. Zaboravljen `Done()`

Rezultat:

Program čeka zauvek.

---

## 3. Višak `Done()`

Rezultat:

```text
panic:
negative WaitGroup counter
```

---

## 4. Prosleđivanje `WaitGroup` po vrednosti

Pogrešno:

```go
func worker(wg sync.WaitGroup)
```

Ispravno:

```go
func worker(wg *sync.WaitGroup)
```

ili korišćenje zatvorenja (closure) koje deli isti `WaitGroup`.

---

## 5. Pozivanje `Add()` nakon pokretanja Goroutine

Loš primer:

```go
go worker()

wg.Add(1)
```

Goroutine može završiti pre nego što se pozove `Add()`.

Ispravno:

```go
wg.Add(1)

go worker()
```

---

# ✅ Best Practices

- Pozovi `Add()` pre pokretanja Goroutine.
- Koristi `defer wg.Done()`.
- Prosleđuj pokazivač na `WaitGroup`.
- Koristi jedan `WaitGroup` za jednu grupu povezanih Goroutines.
- Nemoj koristiti `WaitGroup` za razmenu podataka.
- Nemoj koristiti `time.Sleep()` umesto `WaitGroup`-a.

---

# 📋 Rezime

- `sync.WaitGroup` služi za čekanje završetka više Goroutines.
- `Add()` povećava brojač.
- `Done()` smanjuje brojač.
- `Wait()` čeka dok brojač ne postane nula.
- `defer wg.Done()` predstavlja idiomatski način korišćenja.
- `WaitGroup` ne određuje redosled izvršavanja Goroutines.
- Veoma često se koristi zajedno sa Channel-ima.

---

# ❓ Pitanja za proveru znanja

1. Koji problem rešava `sync.WaitGroup`?
2. Šta radi `Add()`?
3. Šta radi `Done()`?
4. Šta radi `Wait()`?
5. Zašto je `defer wg.Done()` preporučen?
6. Zašto `WaitGroup` obično prosleđujemo preko pokazivača?
7. Šta se događa ako zaboravimo `Done()`?
8. Šta se događa ako pozovemo `Done()` više puta nego što treba?
9. Zašto `WaitGroup` nije zamena za Channel?
10. Zašto `time.Sleep()` nije dobro rešenje za čekanje završetka Goroutines?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Pokreni tri Goroutines koje ispisuju različite poruke. Koristi `WaitGroup` da sačekaš njihov završetak.
2. Napravi pet Worker-a koji ispisuju svoj ID i pravilno koriste `defer wg.Done()`.
3. Ukloni `Wait()` i posmatraj kako se ponaša program.

---

## 🟡 Srednje

4. Napravi deset Goroutines koje računaju kvadrat broja od 1 do 10. Sačekaj njihov završetak pomoću `WaitGroup`.
5. Namerno zaboravi `Done()` u jednoj Goroutine i objasni zašto program blokira.
6. Napravi primer koji izaziva `panic: sync: negative WaitGroup counter` i objasni uzrok.

---

## 🟠 Izazov

7. Napravi program koji pokreće 20 Goroutines. Svaka treba da izvrši neki posao različitog trajanja (koristi `time.Sleep()` samo za simulaciju trajanja posla), a glavna Goroutine treba da ispiše `"All jobs completed"` tek kada sve završe koristeći `sync.WaitGroup`. Nakon toga razmisli kako bi rezultate svih Worker-a mogao da prikupiš pomoću Channel-a. To će biti osnova za narednu lekciju o **Worker Pool** obrascu.

---

### ➡️ Sledeća lekcija **[**Worker Pool**](04-worker-pools.md)**

naučićeš šta je **Worker Pool**, zašto predstavlja jedan od najvažnijih concurrency obrazaca u Go-u, kako kombinuje **Goroutines**, **Channels** i **sync.WaitGroup**, kao i kako ograničava broj istovremeno aktivnih Worker-a radi bolje iskorišćenosti sistemskih resursa.
