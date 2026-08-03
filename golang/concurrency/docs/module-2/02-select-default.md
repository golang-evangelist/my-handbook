# `default` u `select`

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 2/8
>
> **Fajl:** `docs/module-2/02-select-default.md`

---

# 📚 Sadržaj

- Šta predstavlja `default`?
- Zašto postoji?
- Blokirajući vs neblokirajući `select`
- Kako funkcioniše `default`
- Non-blocking receive
- Non-blocking send
- `default` u `for-select` petlji
- Busy Waiting (Busy Loop)
- Kada koristiti `default`
- Kada izbegavati `default`
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja `default` u `select`,
- kako omogućava neblokirajuće Channel operacije,
- kada je njegova upotreba opravdana,
- kada može ozbiljno da naruši performanse programa,
- kako izbeći Busy Waiting.

---

# Podsećanje

U prethodnoj lekciji upoznali smo:

```go
select {

case value := <-ch1:

case value := <-ch2:

}
```

Ako nijedan `case` nije spreman:

```
select

↓

blokira

↓

čeka
```

To je podrazumevano ponašanje.

---

# Problem

Pretpostavimo da želiš:

- da proveriš da li postoji poruka,
- ali ne želiš da čekaš.

Drugim rečima:

```
Ako postoji podatak

↓

obradi ga

Inače

↓

nastavi dalje
```

Kako to uraditi?

Odgovor je:

```go
default
```

---

# Šta predstavlja `default`?

`default` je posebna grana unutar `select`.

Izvršava se kada:

> nijedna druga Channel operacija nije spremna.

Sintaksa:

```go
select {

case ...

case ...

default:

}
```

---

# Prvi primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int)

	select {

	case value := <-ch:

		fmt.Println(value)

	default:

		fmt.Println("No data")

	}
}
```

Izlaz:

```text
No data
```

Zašto?

Na Channel-u nema podataka.

Umesto blokiranja,

izvršava se:

```go
default
```

---

# Vizuelni prikaz

Bez `default`:

```text
select

↓

nema podataka

↓

WAIT
```

---

Sa `default`:

```text
select

↓

nema podataka

↓

default

↓

nastavak programa
```

---

# Najvažnija osobina

Dodavanjem:

```go
default
```

`select`

postaje:

> **non-blocking**

Odnosno:

Program neće čekati.

---

# Šta znači "Non-blocking"?

Blokirajuća operacija:

```text
↓

WAIT

↓

WAIT

↓

WAIT
```

---

Neblokirajuća operacija:

```text
↓

proveri

↓

nastavi dalje
```

Odmah.

---

# Non-blocking Receive

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan string)

	select {

	case value := <-ch:

		fmt.Println(value)

	default:

		fmt.Println("Nothing received")

	}
}
```

Izlaz:

```text
Nothing received
```

Program se nije blokirao.

---

# Non-blocking Send

Isto važi i za slanje.

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan int)

	select {

	case ch <- 100:

		fmt.Println("Sent")

	default:

		fmt.Println("Receiver is not ready")

	}
}
```

Pošto nema Receiver-a:

izvršava se:

```text
Receiver is not ready
```

---

# Buffered Channel primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 1)

	ch <- 10

	select {

	case ch <- 20:

		fmt.Println("Sent")

	default:

		fmt.Println("Buffer is full")

	}
}
```

Izlaz:

```text
Buffer is full
```

Bafer je pun.

Bez `default` program bi čekao.

---

# Kada se `default` NE izvršava?

Ako postoji spreman `case`.

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 1)

	ch <- 42

	select {

	case value := <-ch:

		fmt.Println(value)

	default:

		fmt.Println("Empty")

	}
}
```

Izlaz:

```text
42
```

`default`

se preskače.

---

# Pravilo

Go prvo proverava:

```
case

case

case
```

Ako postoji spremna operacija:

izvršava se ona.

Tek ako nijedna nije spremna:

izvršava se:

```go
default
```

---

# Više Channel-a

```go
select {

case msg := <-ch1:

	fmt.Println(msg)

case msg := <-ch2:

	fmt.Println(msg)

default:

	fmt.Println("No messages")

}
```

Ako nijedan Channel nema podatke:

```
No messages
```

---

# `for-select-default`

Veoma čest obrazac.

```go
for {

	select {

	case value := <-ch:

		fmt.Println(value)

	default:

		fmt.Println("Waiting...")

	}

}
```

Da li je ovo dobro?

Ne baš.

---

# Problem — Busy Waiting

Pogledaj prethodni primer.

Petlja izgleda ovako:

```text
select

↓

default

↓

select

↓

default

↓

select

↓

default
```

Hiljadama ili milionima puta u sekundi.

CPU ne miruje.

Neprestano radi.

Ovo se naziva:

> **Busy Waiting** ili **Busy Loop**.

---

# Vizuelni prikaz

```text
for

↓

select

↓

default

↓

for

↓

select

↓

default

↓

for

↓

...
```

Program gotovo nikada ne spava.

---

# Zašto je Busy Waiting problem?

Posledice mogu biti:

- veliko opterećenje CPU-a,
- povećana potrošnja energije,
- slabije performanse ostatka programa,
- nepotrebno zauzimanje Scheduler-a.

---

# Loš primer

```go
for {

	select {

	default:

	}

}
```

Ova petlja:

- ne radi ništa korisno,
- zauzima CPU gotovo 100%.

---

# Bolji primer (za demonstraciju)

```go
for {

	select {

	case value := <-ch:

		fmt.Println(value)

	default:

		time.Sleep(50 * time.Millisecond)

	}

}
```

Sada CPU dobija priliku da odmori.

---

# Važna napomena

`time.Sleep()` ovde **nije mehanizam sinhronizacije**.

Koristi se isključivo da bi se:

- smanjilo opterećenje CPU-a,
- usporila Busy Loop.

Kasnije ćemo videti elegantnija rešenja koristeći:

- `context.Context`
- timeout
- `time.Ticker`

---

# Kada koristiti `default`?

Koristi ga kada želiš:

- da proveriš da li postoji podatak,
- da proveriš da li je moguće slanje,
- da izbegneš blokiranje,
- da nastaviš izvršavanje ako nema aktivnosti.

---

# Kada izbegavati `default`?

Nemoj ga koristiti ako:

- zapravo želiš da čekaš podatke,
- želiš sinhronizaciju između Goroutines,
- pišeš običan Producer-Consumer kod.

U tim slučajevima je blokirajući `select` često ispravniji.

---

# `default` nije zamena za timeout

Početnici često napišu:

```go
select {

case <-ch:

default:

	fmt.Println("Timeout")

}
```

Ovo nije timeout.

Ovo samo znači:

```
Trenutno nema podataka.
```

Pravi timeout ćemo obraditi u posebnoj lekciji.

---

# `default` nije zamena za Cancellation

Slično važi i za prekid rada.

Nemoj koristiti:

```go
default
```

kao mehanizam za gašenje Goroutines.

Za to postoji:

```go
context.Context
```

koji ćemo detaljno obraditi kasnije.

---

# 🚨 Najčešće greške

## 1. Dodavanje `default` bez potrebe

Ako želiš da čekaš podatke,

nemoj koristiti:

```go
default
```

---

## 2. Busy Waiting

```go
for {

	select {

	default:

	}

}
```

Ovo gotovo uvek predstavlja grešku.

---

## 3. Pogrešno tumačenje `default`

`default` ne znači:

```
Timeout
```

Niti znači:

```
Cancellation
```

On samo znači:

```
Nijedna Channel operacija trenutno nije spremna.
```

---

## 4. Pretpostavljanje da će `default` uvek biti izvršen

Ako postoji spreman `case`,

`default`

se neće izvršiti.

---

# ✅ Best Practices

- Koristi `default` samo kada ti je zaista potrebno neblokirajuće ponašanje.
- Izbegavaj beskonačne Busy Loop petlje.
- Ako koristiš `default` u petlji, razmisli da li postoji efikasnije rešenje.
- Nemoj koristiti `default` kao zamenu za timeout ili cancellation.
- Neka `default` bude izuzetak, a ne podrazumevano rešenje.

---

# 📋 Rezime

- `default` se izvršava samo kada nijedan `case` nije spreman.
- Dodavanjem `default` `select` postaje neblokirajući.
- `default` omogućava non-blocking receive i non-blocking send.
- Ne predstavlja timeout.
- Ne predstavlja cancellation.
- Nepravilna upotreba može dovesti do Busy Waiting problema.

---

# ❓ Pitanja za proveru znanja

1. Šta predstavlja `default` u `select`?
2. Kada se izvršava?
3. Šta znači da je `select` non-blocking?
4. Kako izgleda non-blocking receive?
5. Kako izgleda non-blocking send?
6. Šta je Busy Waiting?
7. Zašto Busy Waiting predstavlja problem?
8. Da li je `default` isto što i timeout?
9. Da li je `default` isto što i cancellation?
10. Kada je opravdano koristiti `default`?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi program koji pokušava da pročita podatak sa praznog Channel-a koristeći `select` i `default`.
2. Napravi primer non-blocking slanja na Unbuffered Channel.
3. Napravi primer non-blocking slanja na Buffered Channel čiji je bafer pun.

---

## 🟡 Srednje

4. Napravi program koji proverava dva Channel-a pomoću `select` i ispisuje `"No messages"` kada nijedan nema podatke.
5. Napravi `for-select-default` petlju i posmatraj opterećenje CPU-a.
6. Dodaj kratko `time.Sleep()` u `default` granu i uporedi ponašanje programa.

---

## 🟠 Izazov

7. Napravi jednostavan sistem koji simulira obradu poslova. Jedna Goroutine povremeno šalje zadatke kroz Channel, dok druga koristi `for-select-default` za njihovu obradu. U `default` grani ispiši da trenutno nema posla i kratko pauziraj izvršavanje kako bi izbegao Busy Waiting. Nakon toga razmisli kako bi isti problem mogao biti elegantnije rešen pomoću timeout-a ili `context.Context`, koje ćemo obraditi u narednim modulima.

---

### ➡️ Sledeća lekcija **[**`sync.WaitGroup`**](03-waitgroup.md)**

upoznaćeš **`sync.WaitGroup`**, naučićeš kako da sačekaš završetak više Goroutines, zašto je to mnogo bolje od korišćenja `time.Sleep()` i kako pravilno koristiti metode `Add()`, `Done()` i `Wait()`.
