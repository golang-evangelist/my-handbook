# `select`

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 1/8
>
> **Fajl:** `docs/module-2/01-select.md`

---

# 📚 Sadržaj

- Šta je `select`?
- Zašto postoji?
- Problem koji `select` rešava
- Sintaksa
- Kako funkcioniše?
- Poređenje sa `switch`
- Jedan `case`
- Više `case` grana
- Slanje pomoću `select`
- Primanje pomoću `select`
- Šta ako je više `case` grana spremno?
- Blokirajuće ponašanje
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja `select`,
- zašto je jedan od najvažnijih concurrency mehanizama u Go-u,
- kako istovremeno raditi sa više Channel-a,
- kako Go bira `case` koji će izvršiti,
- kada `select` blokira izvršavanje.

---

# Uvod

Do sada smo koristili jedan Channel.

Na primer:

```go
value := <-ch
```

ili:

```go
ch <- value
```

Ali šta ako program koristi:

- dva Channel-a,
- pet Channel-a,
- deset Channel-a?

Kako čekati podatke sa svih njih istovremeno?

Odgovor je:

```go
select
```

---

# Problem bez `select`

Pretpostavimo da imamo dva Channel-a.

```go
ch1 := make(chan string)
ch2 := make(chan string)
```

Naivan pokušaj:

```go
fmt.Println(<-ch1)
fmt.Println(<-ch2)
```

Problem?

Program prvo čeka:

```go
ch1
```

Tek nakon toga:

```go
ch2
```

Ako:

- `ch2` dobije podatak odmah,
- `ch1` još nema podatke,

program će ipak čekati `ch1`.

To nije ono što želimo.

---

# Šta je `select`?

`select` omogućava:

> čekanje više Channel operacija istovremeno.

Može čekati:

- prijem,
- slanje,
- kombinaciju oba.

Kada neka operacija postane spremna,

`select`

izvršava odgovarajući `case`.

---

# Sintaksa

```go
select {

case value := <-ch1:

	// ...

case value := <-ch2:

	// ...

}
```

---

# Vizuelni prikaz

```text
            select

        /             \

     Channel 1     Channel 2

         │              │

         ▼              ▼

      prvi koji      prvi koji
      postane        postane
      spreman        spreman

             ▼

       izvršava se
       odgovarajući case
```

---

# Prvi primer

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {

		time.Sleep(2 * time.Second)

		ch1 <- "Hello"

	}()

	go func() {

		time.Sleep(time.Second)

		ch2 <- "World"

	}()

	select {

	case msg := <-ch1:

		fmt.Println(msg)

	case msg := <-ch2:

		fmt.Println(msg)

	}
}
```

Izlaz:

```text
World
```

---

# Zašto?

`ch2`

postaje spreman prvi.

Go izvršava:

```go
case msg := <-ch2
```

Drugi `case` se ne izvršava.

`select` završava rad.

---

# Korak po korak

Korak 1

Pokreću se dve Goroutines.

---

Korak 2

Obe pokušavaju da pošalju podatke.

---

Korak 3

`select` čeka.

---

Korak 4

`ch2`

postaje spreman.

---

Korak 5

Izvršava se odgovarajući `case`.

---

Korak 6

`select`

se završava.

---

# `select` nije petlja

Veoma važna činjenica.

Ovaj kod:

```go
select {

case ...

case ...

}
```

izvršava se samo jednom.

Ako želiš da stalno osluškuješ više Channel-a:

koristi:

```go
for {

	select {

	...

	}

}
```

ili idiomatski:

```go
for {

	select {

	case ...

	case ...

	}

}
```

---

# `select` i `switch`

Na prvi pogled izgledaju slično.

```go
switch x {

case 1:

case 2:

}
```

---

```go
select {

case <-ch1:

case <-ch2:

}
```

Ali postoji velika razlika.

---

## `switch`

Poredi vrednosti.

---

## `select`

Čeka Channel operacije.

Ne poredi vrednosti.

Ne proverava izraze.

Već proverava:

> da li je određena Channel operacija spremna.

---

# `select` sa jednim Channel-om

Može.

```go
select {

case value := <-ch:

	fmt.Println(value)

}
```

Iako nije naročito koristan,

potpuno je ispravan.

Kasnije će biti veoma koristan kada dodamo:

- `default`
- `context`
- timeout
- cancellation

---

# Slanje pomoću `select`

`select`

ne služi samo za prijem.

Može i za slanje.

Primer:

```go
select {

case ch <- 100:

	fmt.Println("Sent")

}
```

Ako slanje može da se izvrši,

izvršiće se taj `case`.

---

# Kombinacija slanja i prijema

```go
select {

case value := <-input:

	fmt.Println(value)

case output <- 50:

	fmt.Println("Sent")

}
```

Go bira operaciju koja može odmah da se izvrši.

---

# Šta ako nijedan `case` nije spreman?

Primer:

```go
select {

case <-ch1:

case <-ch2:

}
```

Ako:

- nema podataka,
- nije moguće slanje,

`select`

blokira.

Čeka dok neka operacija ne postane moguća.

---

# Vizuelni prikaz

```text
select

↓

nijedan case nije spreman

↓

WAIT

↓

Channel postaje spreman

↓

nastavak izvršavanja
```

---

# Šta ako je više `case` grana spremno?

Ovo je veoma važno.

Primer:

```go
select {

case <-ch1:

case <-ch2:

}
```

Pretpostavimo da:

- oba Channel-a već imaju podatke.

Šta će Go uraditi?

---

Odgovor:

Go bira:

> **jedan od spremnih `case`-ova pseudonasumično.**

Specifikacija **ne garantuje redosled**.

Ne treba pisati kod koji očekuje:

- da će uvek pobediti prvi `case`,
- ili da će uvek pobediti drugi.

---

# Primer

```go
package main

import "fmt"

func main() {

	ch1 := make(chan int, 1)
	ch2 := make(chan int, 1)

	ch1 <- 10
	ch2 <- 20

	select {

	case value := <-ch1:

		fmt.Println(value)

	case value := <-ch2:

		fmt.Println(value)

	}
}
```

Mogući izlaz:

```text
10
```

ili:

```text
20
```

Oba rezultata su ispravna.

---

# Zašto pseudonasumičan izbor?

Da bi se izbeglo:

- gladovanje (*starvation*),
- favorizovanje jednog Channel-a,
- nepravedna raspodela izvršavanja.

Ipak, važno je napomenuti da Go specifikacija ne garantuje strogu pravičnost (*fairness*), već samo da će, kada postoji više spremnih operacija, jedna od njih biti izabrana pseudonasumično.

---

# Kada koristiti `select`?

Koristi ga kada:

- čekaš više Channel-a,
- radiš sa više Goroutines,
- implementiraš timeout,
- implementiraš cancellation,
- implementiraš Worker Pool,
- implementiraš Fan-in,
- implementiraš Pipeline.

U praksi ćeš ga koristiti veoma često.

---

# Kada nije potreban?

Ako koristiš:

jedan Channel

```go
<-ch
```

nema potrebe za:

```go
select
```

Osim ako planiraš da dodaš:

- `default`,
- timeout,
- `context.Context`.

---

# 🚨 Najčešće greške

## 1. Očekivanje da `select` proverava sve `case` grane

Ne.

Izvršava se samo jedna spremna grana.

---

## 2. Očekivanje da se izvršavaju svi `case`-ovi

Ne.

Samo jedan.

---

## 3. Pretpostavljanje redosleda

Pogrešno.

Ako je više `case` grana spremno,

redosled nije garantovan.

---

## 4. Zaboravljanje da `select` nije petlja

Ako želiš stalno osluškivanje,

potreban je:

```go
for {

	select {

	...

	}

}
```

---

# ✅ Best Practices

- Koristi `select` kada radiš sa više Channel-a.
- Nemoj pretpostavljati koji će `case` biti izabran.
- Ne oslanjaj se na redosled izvršavanja.
- Kombinuj `select` sa `for` kada želiš kontinuirano osluškivanje.
- Nemoj koristiti `select` kada jedan običan prijem sa Channel-a rešava problem.

---

# 📋 Rezime

- `select` omogućava rad sa više Channel operacija istovremeno.
- Može čekati prijem, slanje ili njihovu kombinaciju.
- Izvršava samo jednu spremnu `case` granu.
- Ako nijedna operacija nije spremna, blokira izvršavanje.
- Ako je više operacija spremno, Go bira jednu pseudonasumično.
- `select` nije petlja i izvršava se samo jednom.

---

# ❓ Pitanja za proveru znanja

1. Šta predstavlja `select`?
2. Koji problem rešava?
3. Da li `select` može da radi sa jednim Channel-om?
4. Da li može da šalje podatke?
5. Da li može da prima podatke?
6. Šta se događa kada nijedan `case` nije spreman?
7. Šta se događa kada je više `case` grana spremno?
8. Da li `select` garantuje redosled izvršavanja?
9. Zašto se `select` često kombinuje sa `for` petljom?
10. U kojim situacijama je `select` najkorisniji?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi dva Channel-a i dve Goroutines koje šalju različite poruke nakon različitog vremenskog kašnjenja. Koristi `select` za prijem prve pristigle poruke.
2. Zameni vremena kašnjenja i objasni kako se menja rezultat.
3. Dodaj treći Channel i proširi `select` novim `case`-om.

---

## 🟡 Srednje

4. Napravi `select` koji može i da primi podatak sa jednog Channel-a i da pošalje podatak na drugi.
5. Napravi beskonačnu `for-select` petlju koja obrađuje poruke sa dva Channel-a.
6. Napravi primer u kome su oba Buffered Channel-a unapred popunjena i više puta pokreni program. Posmatraj da li se uvek izvršava isti `case`.

---

## 🟠 Izazov

7. Napravi mali sistem sa tri Producer Goroutines koje šalju tekstualne poruke različitom brzinom. Jedna Consumer Goroutine treba pomoću `select` da obrađuje poruke čim stignu. Za sada neka program radi ograničen broj iteracija. U narednim lekcijama ćemo ovaj primer proširiti dodavanjem `default`, timeout-a i `context`-a.

---

### ➡️ Sledeća lekcija **[**`default` grana u `select`**](02-select-default.md)**

naučićeš kako funkcioniše **`default` grana u `select`**, kako omogućava **neblokirajuće (non-blocking)** Channel operacije i kada je njeno korišćenje korisno, a kada može dovesti do nepotrebnog zauzimanja procesora (*busy waiting*).
