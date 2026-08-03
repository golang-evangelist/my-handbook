# Closing Channels (`close()`)

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 7/8
>
> **Fajl:** `docs/07-close-channel.md`

---

# 📚 Sadržaj

- Šta predstavlja `close()`?
- Zašto postoji?
- Šta znači "zatvoren Channel"?
- Kako radi `close()`
- Prijem iz zatvorenog Channel-a
- `value, ok := <-ch`
- `range` i `close()`
- Ko treba da zatvori Channel?
- Šta se dešava ako zatvorimo Channel više puta?
- Šta se dešava ako šaljemo u zatvoren Channel?
- `nil` vs otvoren vs zatvoren Channel
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta radi funkcija `close()`,
- šta znači da je Channel zatvoren,
- kako izgleda prijem iz zatvorenog Channel-a,
- kako koristiti `value, ok := <-ch`,
- ko treba da zatvori Channel,
- koje greške dovode do `panic`.

---

# Šta predstavlja `close()`?

Funkcija:

```go
close(ch)
```

govori Go Runtime-u:

> "Na ovaj Channel više nikada neće biti poslata nijedna nova vrednost."

Obrati pažnju.

`close()` **ne briše Channel**.

`close()` **ne oslobađa memoriju odmah**.

`close()` **ne prekida Goroutines**.

Ona samo označava da je slanje završeno.

---

# Šta znači "zatvoren Channel"?

Zatvoren Channel znači:

- nova slanja nisu dozvoljena,
- postojeće vrednosti i dalje mogu da se pročitaju,
- Receiver može da nastavi čitanje dok Channel ne ostane prazan.

---

# Vizuelni prikaz

Pre zatvaranja:

```text
Producer

↓

10

↓

20

↓

30

↓

Channel

↓

Consumer
```

---

Posle:

```go
close(ch)
```

dobijamo:

```text
Producer

↓

X

više nema slanja

↓

Channel

↓

Consumer

i dalje može da pročita:

10

20

30
```

---

# Kako se poziva?

```go
close(ch)
```

Ništa više.

---

# Prvi primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 3)

	ch <- 10
	ch <- 20

	close(ch)

	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

Izlaz:

```text
10
20
```

---

# Zašto ovo radi?

Zato što:

`close()`

nije obrisao sadržaj bafera.

Buffered Channel i dalje sadrži:

```text
10

20
```

Receiver ih normalno preuzima.

---

# Šta kada se bafer isprazni?

Posmatraj primer.

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 1)

	ch <- 42

	close(ch)

	fmt.Println(<-ch)

	fmt.Println(<-ch)
}
```

Šta će biti drugi ispis?

---

Izlaz:

```text
42
0
```

Zašto?

---

# Zero Value

Kada pokušamo da primimo iz:

- zatvorenog,
- potpuno ispražnjenog Channel-a,

dobijamo:

**zero value** tipa koji Channel prenosi.

Na primer:

```go
chan int
```

↓

```text
0
```

---

```go
chan string
```

↓

```text
""
```

---

```go
chan bool
```

↓

```text
false
```

---

```go
chan *User
```

↓

```text
nil
```

---

# Kako razlikovati stvarnu nulu od zatvorenog Channel-a?

Koristi se:

```go
value, ok := <-ch
```

---

# Šta predstavlja `ok`?

Ako je:

```go
ok == true
```

primljena je regularna vrednost.

Ako je:

```go
ok == false
```

Channel je zatvoren i više nema podataka.

---

# Primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 1)

	ch <- 100

	close(ch)

	value, ok := <-ch

	fmt.Println(value, ok)

	value, ok = <-ch

	fmt.Println(value, ok)
}
```

Izlaz:

```text
100 true
0 false
```

---

# Vizuelni prikaz

```text
Buffer

100

↓

Receiver

100 true

↓

Buffer prazan

↓

Receiver

0 false
```

---

# Kako `range` koristi `close()`?

U prethodnoj lekciji videli smo:

```go
for value := range ch {

	fmt.Println(value)

}
```

Interno, `range` koristi logiku sličnu:

```go
for {

	value, ok := <-ch

	if !ok {
		break
	}

	fmt.Println(value)
}
```

Dakle, upravo `ok` omogućava da `range` zna kada treba da se završi.

---

# Ko treba da zatvori Channel?

Ovo je jedno od najvažnijih pravila u Go-u.

## Pravilo

> **Channel zatvara Goroutine koja šalje poslednju vrednost.**

Drugim rečima:

```
Producer

↓

close()
```

---

# Zašto Producer?

Samo Producer zna:

> "Završio sam sa slanjem."

Receiver to ne može pouzdano znati.

---

# Dobar primer

```go
func producer(ch chan<- int) {

	for i := 1; i <= 5; i++ {

		ch <- i

	}

	close(ch)
}
```

Consumer:

```go
func consumer(ch <-chan int) {

	for value := range ch {

		fmt.Println(value)

	}
}
```

Ovo je idiomatski Go.

---

# Loš primer

```go
func consumer(ch <-chan int) {

	close(ch)
}
```

Zašto?

Receiver ne zna da li će Producer poslati još podataka.

Može izazvati `panic`.

---

# Šta ako zatvorimo Channel dva puta?

Primer:

```go
close(ch)

close(ch)
```

Rezultat:

```text
panic: close of closed channel
```

Channel se može zatvoriti samo jednom.

---

# Šta ako šaljemo u zatvoren Channel?

Primer:

```go
close(ch)

ch <- 10
```

Rezultat:

```text
panic: send on closed channel
```

Slanje u zatvoren Channel nije dozvoljeno.

---

# Šta ako primamo iz zatvorenog Channel-a?

To je potpuno dozvoljeno.

Primer:

```go
value := <-ch
```

Ako nema više podataka:

dobijamo:

```
zero value
```

i

```
ok == false
```

---

# `nil` vs otvoren vs zatvoren Channel

Ovo su tri različita stanja.

| Stanje | Slanje | Prijem |
|---------|---------|---------|
| `nil` Channel | Blokira zauvek | Blokira zauvek |
| Otvoren Channel | Radi normalno | Radi normalno |
| Zatvoren Channel | `panic` | Dozvoljeno |

---

# Da li treba uvek zatvarati Channel?

Ne.

Ovo je česta zabluda.

Ako nema Receiver-a koji treba da sazna da je slanje završeno, često nema potrebe za `close()`.

Na primer:

```go
result := make(chan int)

go func() {

	result <- calculate()

}()

value := <-result
```

Ovde nije neophodno zatvarati Channel, jer se razmenjuje samo jedna vrednost.

`close()` je najkorisniji kada Receiver treba da zna da više neće biti novih podataka.

---

# Da li `close()` oslobađa memoriju?

Ne direktno.

Go koristi **Garbage Collector**.

Kada više ne postoji nijedna referenca na Channel, Runtime može da oslobodi njegovu memoriju.

Nemoj koristiti `close()` kao način oslobađanja memorije.

---

# Više Producer-a

Ovo zahteva posebnu pažnju.

Ako više Goroutines šalje podatke:

```text
Producer A

↓

Producer B

↓

Producer C

↓

Channel
```

Ko sme da pozove:

```go
close(ch)
```

Odgovor:

Samo onaj deo programa koji može da garantuje da su **svi Producer-i završili**.

Vrlo često će se za to koristiti:

- `sync.WaitGroup`
- posebna koordinaciona Goroutine

O ovome ćemo govoriti u kasnijim modulima.

---

# 🚨 Najčešće greške

## 1. Receiver zatvara Channel

Najčešće pogrešno.

---

## 2. Dvostruki `close()`

```go
close(ch)

close(ch)
```

↓

```text
panic
```

---

## 3. Slanje nakon `close()`

```go
close(ch)

ch <- 5
```

↓

```text
panic
```

---

## 4. Zaboravljen `ok`

Ako očekuješ da Channel može biti zatvoren:

```go
value, ok := <-ch
```

je sigurnije od:

```go
value := <-ch
```

---

## 5. Verovanje da `close()` briše podatke

Ne briše.

Preostale vrednosti ostaju dostupne.

---

# ✅ Best Practices

- Neka Producer zatvori Channel.
- Zatvori Channel samo kada više neće biti novih vrednosti.
- Nikada nemoj slati podatke u zatvoren Channel.
- Ne zatvaraj isti Channel više puta.
- Koristi `range` kada želiš da pročitaš sve vrednosti.
- Koristi `value, ok := <-ch` kada treba da razlikuješ regularnu vrednost od zatvorenog Channel-a.
- Nemoj zatvarati Channel samo zato što "deluje ispravno" — zatvaraj ga kada postoji jasan razlog.

---

# 📋 Rezime

- `close(ch)` označava kraj slanja podataka.
- Zatvoren Channel i dalje može sadržati nepročitane vrednosti.
- Prijem iz zatvorenog Channel-a je dozvoljen.
- Slanje u zatvoren Channel izaziva `panic`.
- Dvostruko zatvaranje Channel-a izaziva `panic`.
- `value, ok := <-ch` omogućava da prepoznaš zatvoren Channel.
- U najvećem broju slučajeva Channel zatvara Producer.

---

# ❓ Pitanja za proveru znanja

1. Šta radi `close()`?
2. Da li `close()` briše sadržaj Channel-a?
3. Šta se dešava sa preostalim vrednostima nakon `close()`?
4. Šta predstavlja promenljiva `ok`?
5. Šta se događa pri prijemu iz zatvorenog i ispražnjenog Channel-a?
6. Ko treba da zatvori Channel?
7. Šta se događa ako zatvoriš isti Channel dva puta?
8. Šta se događa ako pošalješ vrednost u zatvoren Channel?
9. Da li je uvek potrebno pozvati `close()`?
10. Da li `close()` oslobađa memoriju?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi Producer koji šalje pet brojeva, zatvara Channel i Consumer koji koristi `range`.
2. Nakon zatvaranja Channel-a pročitaj sve preostale vrednosti.
3. Isprobaj `value, ok := <-ch` i analiziraj rezultate.

---

## 🟡 Srednje

4. Napravi Buffered Channel, pošalji nekoliko vrednosti, zatvori ga i pokaži da se sve vrednosti mogu pročitati nakon `close()`.
5. Napravi primer koji demonstrira `panic: send on closed channel`.
6. Napravi primer koji demonstrira `panic: close of closed channel`.

---

## 🟠 Izazov

7. Implementiraj Producer koji šalje brojeve od 1 do 100, zatvara Channel i Consumer koji koristi `range` za računanje zbira svih brojeva. Zatim preradi program tako da koristi `value, ok := <-ch` umesto `range` i uporedi oba pristupa.

---

### ➡️ Sledeća lekcija **[Modul #1 - Sumiranje i Zadaci](08-module-1-summary-and-exercises.md)**

sumiraćemo sve što je obrađeno u Modulu #1, uporediti **Goroutines**, **Channels**, **Unbuffered**, **Buffered**, **Directional Channels**, **`range`** i **`close()`**, a zatim rešavati praktične zadatke koji kombinuju sve ove koncepte u celinu.
