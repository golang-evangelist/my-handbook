# Range over Channels

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 6/8
>
> **Fajl:** `docs/06-range-over-channels.md`

---

# 📚 Sadržaj

- Šta je `range` nad Channel-om?
- Zašto postoji?
- Kako funkcioniše?
- Veza između `range` i `close()`
- Ručno primanje vs `range`
- Kada se `range` završava?
- `range` nad Buffered Channel-om
- Tipične greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- kako funkcioniše `range` nad Channel-om,
- zašto je to najčešći način čitanja podataka,
- kada se petlja završava,
- kakva je veza između `range` i `close()`,
- koje su najčešće greške prilikom korišćenja.

---

# Podsećanje

Do sada smo podatke primali ovako:

```go
value := <-ch
```

Ako želimo više vrednosti:

```go
a := <-ch
b := <-ch
c := <-ch
```

Ovakav kod vrlo brzo postaje nepraktičan.

---

# Šta je `range` nad Channel-om?

`range` omogućava da automatski primaš sve vrednosti koje stižu kroz Channel.

Sintaksa:

```go
for value := range ch {

	// obrada podatka

}
```

Go će:

- čekati novu vrednost,
- dodeliti je promenljivoj,
- izvršiti telo petlje,
- ponavljati postupak.

---

# Najvažnija osobina

Petlja:

```go
for value := range ch
```

traje sve dok Channel ne bude:

```
zatvoren
```

i dok se ne preuzmu sve preostale vrednosti.

Ovo je ključna razlika u odnosu na običan `for`.

---

# Prvi primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int)

	go func() {

		ch <- 10
		ch <- 20
		ch <- 30

		close(ch)

	}()

	for value := range ch {

		fmt.Println(value)

	}
}
```

Izlaz:

```text
10
20
30
```

---

# Analiza izvršavanja

Korak po korak.

---

## Korak 1

Kreira se Channel.

```go
ch := make(chan int)
```

---

## Korak 2

Pokreće se Sender.

---

## Korak 3

Šalje se:

```text
10
```

Receiver dobija:

```text
10
```

Petlja nastavlja.

---

## Korak 4

Šalje se:

```text
20
```

Petlja ponovo izvršava telo.

---

## Korak 5

Šalje se:

```text
30
```

Petlja nastavlja.

---

## Korak 6

Poziva se:

```go
close(ch)
```

Više neće biti novih vrednosti.

---

## Korak 7

Pošto više nema podataka:

```
range

↓

završava petlju
```

Program normalno nastavlja.

---

# Vizuelni prikaz

```text
Producer

10

↓

20

↓

30

↓

close()

↓

Channel

↓

range

↓

10

↓

20

↓

30

↓

END
```

---

# Zašto je `close()` važan?

Pogledaj primer.

```go
for value := range ch {

	fmt.Println(value)

}
```

Kako `range` zna da nema više podataka?

Odgovor:

Ne zna.

Jedini način je da Channel bude zatvoren.

---

# Šta ako ne pozovemo `close()`?

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan int)

	go func() {

		ch <- 1
		ch <- 2

	}()

	for value := range ch {

		fmt.Println(value)

	}
}
```

Šta će se dogoditi?

---

Program će ispisati:

```text
1
2
```

i zatim će čekati zauvek.

Zašto?

`range` očekuje novu vrednost.

Channel nije zatvoren.

---

# Vizuelni prikaz

```text
1

↓

2

↓

???

↓

WAIT
```

Pošto Channel nije zatvoren:

`range` ne zna da je posao završen.

---

# `range` ne proverava broj elemenata

Važno.

Petlja:

```go
for value := range ch
```

ne zna:

- koliko će vrednosti stići,
- kada će stići,
- koliko Sender-a postoji.

Ona samo:

- prima,
- prima,
- prima,
- završava kada vidi zatvoren Channel.

---

# Ručno primanje

Moguće je pisati:

```go
for {

	value := <-ch

	fmt.Println(value)

}
```

Ali postoji problem.

Kako znaš kada da izađeš iz petlje?

Ne znaš.

---

# Rešenje

Go omogućava drugi oblik prijema.

```go
value, ok := <-ch
```

---

# Šta predstavlja `ok`?

Ako je:

```go
ok == true
```

dobijena je regularna vrednost.

Ako je:

```go
ok == false
```

Channel je zatvoren i nema više podataka.

---

# Primer

```go
for {

	value, ok := <-ch

	if !ok {
		break
	}

	fmt.Println(value)

}
```

Ovo radi isto što i:

```go
for value := range ch
```

samo je znatno opširnije.

---

# Šta zapravo radi `range`?

Možeš ga zamisliti ovako:

```go
for {

	value, ok := <-ch

	if !ok {
		break
	}

	// telo petlje

}
```

Naravno, ovo je pojednostavljena ilustracija.

---

# `range` nad Buffered Channel-om

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 3)

	ch <- 10
	ch <- 20
	ch <- 30

	close(ch)

	for value := range ch {

		fmt.Println(value)

	}
}
```

Izlaz:

```text
10
20
30
```

---

# Zašto radi?

Iako je Channel zatvoren:

```
close(ch)
```

u njemu i dalje postoje vrednosti.

`range` će:

- prvo pročitati sve postojeće vrednosti,
- tek onda završiti.

---

# Važno pravilo

Zatvoren Channel:

≠

Prazan Channel.

Primer:

```text
close()

↓

u baferu još postoje podaci
```

Receiver ih može normalno preuzeti.

---

# Kada se `range` završava?

Potrebna su oba uslova.

```
Channel je zatvoren

AND

više nema podataka
```

Tek tada petlja izlazi.

---

# Tipičan Producer-Consumer primer

```go
func producer(ch chan<- int) {

	for i := 1; i <= 5; i++ {

		ch <- i

	}

	close(ch)

}

func consumer(ch <-chan int) {

	for value := range ch {

		fmt.Println(value)

	}

}
```

Ovo je najčešći obrazac koji ćeš viđati u Go projektima.

---

# Ko zatvara Channel?

Veoma važno pravilo.

Najčešće:

```
Producer

↓

close(channel)
```

Receiver gotovo nikada ne zatvara Channel.

Zašto?

Zato što upravo Producer zna kada više neće slati podatke.

O ovoj temi ćemo detaljnije govoriti u sledećoj lekciji.

---

# Prednosti `range`

- kod je kraći,
- čitljiviji,
- nema ručne provere `ok`,
- manje mogućnosti za greške,
- standardni idiom u Go jeziku.

---

# Kada koristiti `range`?

Koristi ga kada:

- želiš da obradiš sve vrednosti,
- ne znaš unapred njihov broj,
- Producer zatvara Channel.

To je najčešći slučaj u praksi.

---

# Kada nije najbolji izbor?

Ako želiš:

- da primiš tačno jednu vrednost,
- da primiš tačno dve vrednosti,
- da koristiš `select`,
- da implementiraš timeout,

onda je običan prijem:

```go
<-ch
```

često prikladniji.

---

# 🚨 Najčešće greške

## 1. Zaboravljen `close()`

```go
for value := range ch {

	...

}
```

Ako niko ne zatvori Channel:

petlja čeka zauvek.

---

## 2. Receiver zatvara Channel

Najčešće nije Receiver taj koji zna da više neće biti novih vrednosti.

Zbog toga Receiver uglavnom ne poziva `close()`.

---

## 3. Ručno korišćenje `ok` bez potrebe

Ako samo želiš da obradiš sve vrednosti:

```go
for value := range ch
```

je jednostavnije.

---

## 4. Pretpostavljanje da `close()` briše podatke

Ne briše.

Preostale vrednosti ostaju dostupne.

---

# ✅ Best Practices

- Koristi `range` za čitanje svih vrednosti iz Channel-a.
- Neka Producer zatvori Channel kada završi slanje.
- Nemoj koristiti beskonačnu petlju ako `range` rešava problem.
- Ne zaboravi da zatvoren Buffered Channel može sadržati nepročitane vrednosti.
- Ako očekuješ samo jednu vrednost, koristi običan prijem.

---

# 📋 Rezime

- `range` omogućava elegantno čitanje svih vrednosti iz Channel-a.
- Petlja traje dok se Channel ne zatvori i dok se ne pročitaju sve preostale vrednosti.
- `range` interno koristi mehanizam sličan `value, ok := <-ch`.
- `close()` ne briše podatke iz bafera.
- `range` predstavlja idiomatski način čitanja podataka iz Channel-a u Go-u.

---

# ❓ Pitanja za proveru znanja

1. Šta radi `range` nad Channel-om?
2. Kada se `range` završava?
3. Zašto je `close()` važan?
4. Šta predstavlja promenljiva `ok`?
5. Šta se događa ako niko ne zatvori Channel?
6. Da li `close()` briše podatke iz Buffered Channel-a?
7. Ko najčešće zatvara Channel?
8. Kada koristiti `range`?
9. Kada je bolje koristiti običan prijem pomoću `<-ch`?
10. Zašto se `range` smatra idiomatskim načinom rada sa Channel-ima?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi Producer koji šalje brojeve od 1 do 5 i zatvara Channel.
2. Napravi Consumer koji koristi `range` za ispis svih brojeva.
3. Ukloni `close()` i objasni ponašanje programa.

---

## 🟡 Srednje

4. Koristi Buffered Channel i pošalji deset vrednosti pre nego što ih primiš pomoću `range`.
5. Napiši primer koji koristi `value, ok := <-ch` umesto `range` i uporedi čitljivost.
6. Dokaži da se preostale vrednosti mogu pročitati i nakon `close()`.

---

## 🟠 Izazov

7. Napravi Producer koji generiše kvadrate brojeva od 1 do 100 i šalje ih kroz Channel. Consumer treba da koristi `range` za obradu svih rezultata. Nakon toga objasni zašto je `range` prirodnije rešenje od ručnog korišćenja beskonačne `for` petlje i promenljive `ok`.

---

### ➡️ Sledeća lekcija **[**close()` funkcija**](07-close-channel.md)**

detaljno ćemo proučiti funkciju `close()`, njena pravila, šta se dešava nakon zatvaranja Channel-a, kako izgleda prijem iz zatvorenog Channel-a i koje su najčešće greške koje dovode do `panic` situacija.
