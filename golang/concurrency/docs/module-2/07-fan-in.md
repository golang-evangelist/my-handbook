# Fan-in

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 7/8
>
> **Fajl:** `docs/module-2/07-fan-in.md`

---

# 📚 Sadržaj

- Šta je Fan-in?
- Problem koji rešava
- Kako funkcioniše?
- Fan-in arhitektura
- Spajanje više Channel-a u jedan
- Fan-in vs Fan-out
- Fan-in i `select`
- Fan-in i `sync.WaitGroup`
- Redosled rezultata
- Kada koristiti Fan-in?
- Kada ga izbegavati?
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja **Fan-in** obrazac,
- kako spojiti više izvora podataka u jedan Channel,
- kako pravilno zatvoriti izlazni Channel,
- zašto se Fan-in skoro uvek koristi zajedno sa Fan-out obrascem,
- kako napisati idiomatsku Fan-in implementaciju u Go-u.

---

# Uvod

U prethodnoj lekciji naučili smo:

> Jedan Channel → više Worker-a

To je:

> **Fan-out**

Sada učimo suprotan obrazac.

Više izvora podataka.

↓

Jedan izlaz.

To nazivamo:

> **Fan-in**

---

# Šta je Fan-in?

Fan-in predstavlja obrazac u kome:

> **više Goroutines ili više Channel-a šalje podatke u jedan zajednički izlazni Channel.**

---

# Vizuelni prikaz

```text
Worker 1

      │

      ▼

Worker 2

      │

      ▼

Worker 3

      │

      ▼

 +---------------+

 | Merge Channel |

 +---------------+

        │

        ▼

     Consumer
```

---

# Zašto naziv "Fan-in"?

Kod Fan-out imali smo:

```text
Jedan

↓

Više
```

Kod Fan-in imamo:

```text
Više

↓

Jedan
```

Tok podataka se "skuplja" u jednu tačku.

---

# Problem

Pretpostavimo da imamo:

```text
Worker 1

↓

Rezultat A

Worker 2

↓

Rezultat B

Worker 3

↓

Rezultat C
```

Kako će glavna Goroutine čitati rezultate?

Jedna mogućnost je:

```go
<-ch1

<-ch2

<-ch3
```

Ali...

šta ako:

- ne znamo koji će Worker prvi završiti?
- imamo 20 Worker-a?
- imamo 100 Worker-a?

---

# Rešenje

Napravimo:

```text
Jedan izlazni Channel.
```

Svi rezultati završavaju tamo.

---

# Vizuelni prikaz

```text
ch1

↓

ch2

↓

ch3

↓

Merged Channel

↓

Main
```

---

# Jednostavan primer

Imamo dva Channel-a.

```go
ch1 := make(chan int)
ch2 := make(chan int)
```

Želimo jedan:

```go
out
```

---

# Merge funkcija

```go
func merge(
	ch1,
	ch2 <-chan int,
) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for value := range ch1 {

			out <- value

		}

		for value := range ch2 {

			out <- value

		}

	}()

	return out

}
```

---

# Da li je ovo pravi Fan-in?

Radi.

Ali nije idealan.

Zašto?

Prvo će se potpuno isprazniti:

```text
ch1
```

Tek onda:

```text
ch2
```

To nije prava konkurentna obrada.

---

# Bolje rešenje

Koristićemo:

- dve Goroutines,
- `sync.WaitGroup`.

---

# Produkciona implementacija

```go
func merge(
	ch1,
	ch2 <-chan int,
) <-chan int {

	out := make(chan int)

	var wg sync.WaitGroup

	copy := func(c <-chan int) {

		defer wg.Done()

		for value := range c {

			out <- value

		}

	}

	wg.Add(2)

	go copy(ch1)

	go copy(ch2)

	go func() {

		wg.Wait()

		close(out)

	}()

	return out

}
```

---

# Šta radi ova funkcija?

Pokreće dve Goroutines.

Svaka čita svoj Channel.

Obe šalju rezultate na:

```go
out
```

Tek kada obe završe:

```go
close(out)
```

---

# Vizuelni prikaz

```text
ch1

↓

copy()

↓

out

----------------

ch2

↓

copy()

↓

out

----------------

WaitGroup

↓

close(out)
```

---

# Zašto koristimo `WaitGroup`?

Imamo:

```text
2 Goroutines
```

Moramo sačekati da obe završe.

Tek tada smemo zatvoriti:

```go
out
```

---

# Zašto nijedna Goroutine ne zatvara `out`?

Pretpostavimo:

```text
copy(ch1)
```

završi prva.

Ako uradi:

```go
close(out)
```

druga Goroutine će pokušati:

```go
out <- value
```

Rezultat:

```text
panic:

send on closed channel
```

---

# Redosled rezultata

Da li će rezultati biti:

```text
A

B

C
```

?

Ne mora.

Moguće je:

```text
B

A

C
```

ili:

```text
C

A

B
```

Redosled nije garantovan.

---

# Fan-in i `select`

Ako imamo mali broj Channel-a,

možemo koristiti i:

```go
select {

case value := <-ch1:

case value := <-ch2:

}
```

Međutim,

za veliki broj Channel-a,

merge funkcija je elegantnije rešenje.

---

# Fan-in + Fan-out

Ovo je jedan od najčešćih obrazaca.

```text
Jobs

↓

Fan-out

↓

Worker 1

Worker 2

Worker 3

↓

Fan-in

↓

Results
```

---

# Gde se koristi?

Veoma često.

Na primer:

- Worker Pool,
- Pipeline,
- obrada logova,
- obrada događaja,
- distribuirani sistemi,
- web crawler-i,
- obrada mrežnih zahteva.

---

# Prednosti

## Jednostavniji Consumer

Consumer čita:

```go
range results
```

Ne zanima ga odakle rezultat dolazi.

---

## Modularnost

Novi Worker se lako dodaje.

---

## Skalabilnost

Broj Worker-a može da raste.

Fan-in ostaje isti.

---

## Manje sinhronizacije

Consumer koristi samo jedan Channel.

---

# Ograničenja

Fan-in ne garantuje:

- redosled,
- fer raspodelu,
- isti broj rezultata po Worker-u.

---

# Kada koristiti Fan-in?

Koristi ga kada:

- imaš više Worker-a,
- želiš jedan izlaz,
- rezultati dolaze različitim redosledom,
- želiš jednostavniji Consumer.

---

# Kada ga izbegavati?

Ako:

- postoji samo jedan izvor podataka,
- redosled rezultata mora biti strogo definisan,
- nema potrebe za objedinjavanjem rezultata.

---

# 🚨 Najčešće greške

## 1. Worker zatvara izlazni Channel

Pogrešno.

---

## 2. Zaboravljen `WaitGroup`

Može dovesti do:

```text
panic:

send on closed channel
```

ili do blokiranja programa.

---

## 3. Očekivanje redosleda

Fan-in ne garantuje:

```text
1

2

3
```

---

## 4. Čitanje Channel-a redom

```go
<-ch1

<-ch2

<-ch3
```

Ovo često poništava prednosti konkurentnosti.

---

## 5. Mešanje odgovornosti

Merge funkcija treba da radi samo:

```
Spajanje Channel-a.
```

Ništa više.

---

# ✅ Best Practices

- Koristi poseban izlazni Channel.
- Koristi `sync.WaitGroup` za koordinaciju.
- Zatvori izlazni Channel tek nakon završetka svih Goroutines.
- Ne oslanjaj se na redosled rezultata.
- Piši generičku merge funkciju kada je moguće.

---

# 📋 Rezime

- Fan-in spaja više izvora podataka u jedan izlazni Channel.
- Veoma često se koristi zajedno sa Fan-out obrascem.
- `sync.WaitGroup` omogućava bezbedno zatvaranje izlaznog Channel-a.
- Redosled rezultata nije garantovan.
- Fan-in pojednostavljuje rad Consumer-a.

---

# ❓ Pitanja za proveru znanja

1. Šta predstavlja Fan-in?
2. Koji problem rešava?
3. Zašto je potreban `WaitGroup`?
4. Ko zatvara izlazni Channel?
5. Zašto Worker ne sme da zatvori izlazni Channel?
6. Da li Fan-in garantuje redosled rezultata?
7. Kako se Fan-in razlikuje od Fan-out obrasca?
8. Kada je Fan-in koristan?
9. Kada ga nije potrebno koristiti?
10. Zašto Consumer koristi samo jedan Channel?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi dva Channel-a koji šalju različite brojeve i spoji ih u jedan izlazni Channel.
2. Dodaj treći Channel u postojeću merge funkciju.
3. Ispiši sve rezultate pomoću `range`.

---

## 🟡 Srednje

4. Napravi tri Worker-a koji računaju kvadrat brojeva i šalju rezultate na tri različita Channel-a. Spoji ih pomoću Fan-in obrasca.
5. Dodaj `sync.WaitGroup` u merge funkciju.
6. Više puta pokreni program i posmatraj kako se menja redosled rezultata.

---

## 🟠 Izazov

7. Napravi mini sistem za pretragu:

- Tri Goroutines simuliraju tri različita servera za pretragu.
- Svaki server vraća rezultate različitom brzinom.
- Koristi Fan-in da spojiš sve rezultate u jedan Channel.
- Glavna Goroutine treba da prikaže rezultate redom kojim stižu, bez pretpostavljanja njihovog redosleda.

---

### ➡️ Sledeća lekcija **[**Modul #2 - Sumiranje i Zadaci**](08-module-2-summary-and-exercises.md)**

U narednom fajlu:

`docs/module-2/08-module-2-summary-and-exercises.md`

napravićemo kompletan pregled svega naučenog u **Modulu #2**, povezati `select`, `WaitGroup`, Worker Pool, Pipeline, Fan-out i Fan-in u jednu celinu i rešiti nekoliko složenijih praktičnih zadataka koji kombinuju više concurrency obrazaca.
