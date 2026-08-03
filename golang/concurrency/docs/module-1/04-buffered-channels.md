# Buffered Channels

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 4/8
>
> **Fajl:** `docs/04-buffered-channels.md`

---

# 📚 Sadržaj

- Šta je Buffered Channel?
- Zašto postoji?
- Kako se kreira?
- Capacity (`cap`)
- Length (`len`)
- Kako funkcioniše interni bafer?
- Kada Sender blokira?
- Kada Receiver blokira?
- Buffered vs Unbuffered Channel
- Tipični scenariji korišćenja
- Prednosti i mane
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja Buffered Channel,
- kako funkcioniše njegov interni bafer,
- kada dolazi do blokiranja,
- kako proveriti veličinu i popunjenost bafera,
- kada koristiti Buffered, a kada Unbuffered Channel.

---

# Podsećanje

U prethodnoj lekciji upoznali smo **Unbuffered Channel**.

```go
ch := make(chan int)
```

Njegove osobine:

- nema bafer,
- Sender čeka Receiver-a,
- Receiver čeka Sender-a,
- predstavlja sinhronizacionu tačku.

Sada ćemo upoznati njegovu "rođaku".

---

# Šta je Buffered Channel?

**Buffered Channel** je Channel koji poseduje **interni bafer**.

To znači da može privremeno da sačuva određeni broj vrednosti.

Za razliku od Unbuffered Channel-a:

> Sender ne mora odmah da čeka Receiver-a.

Sve dok u baferu postoji slobodno mesto.

---

# Kako se kreira?

Prilikom kreiranja navodi se kapacitet.

```go
ch := make(chan int, 3)
```

Broj:

```go
3
```

predstavlja veličinu internog bafera.

---

# Vizuelni prikaz

```text
make(chan int, 3)

┌─────────────────────────┐
│        Channel          │
├───────┬───────┬─────────┤
│       │       │         │
└───────┴───────┴─────────┘

Capacity = 3
```

Na početku:

```text
Length = 0

Capacity = 3
```

---

# Šta predstavlja Capacity?

Kapacitet predstavlja:

> Maksimalan broj vrednosti koje Channel može da sačuva.

Na primer:

```go
make(chan int, 5)
```

znači:

```
Može sadržati najviše 5 vrednosti.
```

Ne više.

---

# Šta predstavlja Length?

Length predstavlja:

> Broj trenutno sačuvanih vrednosti.

Na primer:

```go
make(chan int, 5)
```

Ako pošaljemo dve vrednosti:

```text
10
20
```

onda važi:

```go
len(ch) == 2

cap(ch) == 5
```

---

# Funkcije len() i cap()

Go omogućava da proverimo stanje Channel-a.

```go
fmt.Println(len(ch))
fmt.Println(cap(ch))
```

Primer:

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 5)

	ch <- 10
	ch <- 20

	fmt.Println(len(ch))
	fmt.Println(cap(ch))
}
```

Izlaz:

```text
2
5
```

---

# Kako radi Buffered Channel?

Pretpostavimo:

```go
ch := make(chan int, 3)
```

Početno stanje:

```text
┌───┬───┬───┐
│   │   │   │
└───┴───┴───┘
```

---

Pošaljemo:

```go
ch <- 10
```

Stanje:

```text
┌───┬───┬───┐
│10 │   │   │
└───┴───┴───┘
```

---

Pošaljemo:

```go
ch <- 20
```

```text
┌───┬───┬───┐
│10 │20 │   │
└───┴───┴───┘
```

---

Pošaljemo:

```go
ch <- 30
```

```text
┌───┬───┬───┐
│10 │20 │30 │
└───┴───┴───┘
```

Bafer je sada pun.

---

# Šta se dešava kada je bafer pun?

Ako pokušamo:

```go
ch <- 40
```

Sender će biti blokiran.

Zašto?

Zato što više nema slobodnog mesta.

---

# Kada se Sender oslobađa?

Tek kada Receiver preuzme jednu vrednost.

Na primer:

```go
value := <-ch
```

Sada stanje postaje:

```text
┌───┬───┬───┐
│20 │30 │   │
└───┴───┴───┘
```

Pojavilo se jedno slobodno mesto.

Sender može da pošalje novu vrednost.

---

# FIFO ponašanje

Buffered Channel funkcioniše po principu:

**FIFO**

(**First In, First Out**)

Prva poslata vrednost biće prva primljena.

Primer:

```go
ch <- 10
ch <- 20
ch <- 30
```

Prijem:

```go
<-ch // 10

<-ch // 20

<-ch // 30
```

Redosled se čuva.

---

# Prvi primer

```go
package main

import "fmt"

func main() {

	ch := make(chan int, 2)

	ch <- 10
	ch <- 20

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

# Zašto ovde nema Goroutine?

Pošto postoji bafer.

Prva dva slanja uspevaju odmah.

Tek treće slanje bi blokiralo.

---

# Primer blokiranja

```go
package main

func main() {

	ch := make(chan int, 2)

	ch <- 10
	ch <- 20

	ch <- 30
}
```

Treće slanje:

```go
ch <- 30
```

blokira izvršavanje.

Bafer je pun.

---

# Vizuelni prikaz

```text
Kapacitet = 2

┌────┬────┐
│10  │20  │
└────┴────┘

FULL
```

Pokušaj:

```text
30

↓

WAIT
```

---

# Kada Receiver blokira?

Ako je bafer prazan.

Primer:

```go
ch := make(chan int, 3)

value := <-ch
```

Pošto nema podataka:

Receiver čeka.

---

# Pravilo

Buffered Channel uvodi sledeća pravila:

Sender blokira samo kada je:

```text
Buffer FULL
```

Receiver blokira samo kada je:

```text
Buffer EMPTY
```

Ovo je najvažnija razlika u odnosu na Unbuffered Channel.

---

# Buffered vs Unbuffered

| Osobina | Unbuffered | Buffered |
|----------|------------|----------|
| Interni bafer | ❌ | ✅ |
| Kapacitet | 0 | > 0 |
| Sender odmah blokira | Da | Samo kada je bafer pun |
| Receiver blokira bez podataka | Da | Da |
| Sinhronizaciona tačka | Uvek | Samo kada nema mesta ili nema podataka |

---

# Da li Buffered Channel uklanja potrebu za sinhronizacijom?

Ne.

On samo omogućava određeni stepen razdvajanja pošiljaoca i primaoca.

I dalje:

- postoji koordinacija,
- postoji blokiranje,
- postoji redosled.

Samo nije potrebno da obe strane budu spremne u istom trenutku.

---

# Kada koristiti Buffered Channel?

Najčešće kada:

- proizvođač povremeno radi brže od potrošača,
- želiš da smanjiš broj blokiranja,
- obrađuješ zadatke u redovima čekanja,
- implementiraš Worker Pool,
- implementiraš Pipeline.

---

# Kada nije najbolji izbor?

Ako želiš:

- strogu sinhronizaciju,
- potvrdu da je primalac odmah preuzeo vrednost,
- jednostavan signal između dve Goroutines,

onda je Unbuffered Channel često bolji izbor.

---

# Koliki kapacitet izabrati?

Ovo je jedno od najčešćih pitanja.

Ne postoji univerzalan odgovor.

Kapacitet zavisi od:

- brzine proizvođača,
- brzine potrošača,
- količine memorije,
- prirode problema.

Nemoj birati veliki kapacitet "za svaki slučaj".

Veliki bafer može:

- povećati potrošnju memorije,
- sakriti probleme sa performansama,
- otežati otkrivanje zastoja.

---

# 🚨 Najčešće greške

## 1. Mešanje `len()` i `cap()`

```go
len(ch)
```

nije isto što i:

```go
cap(ch)
```

---

## 2. Pretpostavljanje da Buffered Channel nikada ne blokira

Netačno.

Ako je bafer pun:

Sender čeka.

---

## 3. Prevelik kapacitet

Veći bafer nije automatski bolje rešenje.

---

## 4. Oslanjanje na `len(ch)` za sinhronizaciju

Primer:

```go
if len(ch) > 0 {
	value := <-ch
}
```

Između provere i prijema druga Goroutine može promeniti stanje Channel-a.

`len()` služi za uvid u trenutno stanje, ali nije pouzdan mehanizam za koordinaciju konkurentnog koda.

---

# ✅ Best Practices

- Koristi Buffered Channel kada postoji opravdan razlog za bafer.
- Izaberi kapacitet na osnovu stvarnog opterećenja.
- Nemoj koristiti veliki bafer kao zamenu za dobro projektovan algoritam.
- Nemoj donositi odluke o sinhronizaciji na osnovu `len(ch)`.
- Ako ti je potrebna direktna koordinacija između dve Goroutines, razmotri Unbuffered Channel.

---

# 📋 Rezime

- Buffered Channel poseduje interni bafer.
- Kreira se pomoću:

```go
make(chan T, capacity)
```

- `cap(ch)` vraća maksimalni kapacitet.
- `len(ch)` vraća broj trenutno sačuvanih vrednosti.
- Sender blokira kada je bafer pun.
- Receiver blokira kada je bafer prazan.
- Buffered Channel funkcioniše po FIFO principu.
- Buffered Channel ne zamenjuje sinhronizaciju, već omogućava veću fleksibilnost između proizvođača i potrošača.

---

# ❓ Pitanja za proveru znanja

1. Šta je Buffered Channel?
2. Kako se kreira?
3. Šta predstavlja kapacitet?
4. Šta vraća `len(ch)`?
5. Šta vraća `cap(ch)`?
6. Kada Sender blokira?
7. Kada Receiver blokira?
8. Da li Buffered Channel garantuje FIFO redosled?
9. Kada koristiti Buffered Channel?
10. Zašto `len(ch)` nije pouzdan način za sinhronizaciju?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Kreiraj `chan int` kapaciteta 3 i pošalji tri vrednosti.
2. Ispiši rezultat funkcija `len()` i `cap()` nakon svakog slanja.
3. Preuzmi sve vrednosti i posmatraj kako se menja `len(ch)`.

## 🟡 Srednje

4. Napravi proizvođača koji šalje 10 brojeva u Buffered Channel kapaciteta 5 i potrošača koji ih prima.
5. Eksperimentiši sa različitim kapacitetima (1, 2, 5 i 10) i analiziraj ponašanje programa.
6. Napravi program koji demonstrira blokiranje kada je bafer pun.

## 🟠 Izazov

7. Simuliraj red čekanja za obradu zadataka koristeći Buffered Channel. Jedna Goroutine dodaje zadatke, druga ih obrađuje. Prati kako se menja popunjenost bafera pomoću `len(ch)` i `cap(ch)`, ali nemoj koristiti `len(ch)` za donošenje odluka o sinhronizaciji.

---

### ➡️ Sledeća lekcija **[**Channel Directions**](05-channel-directions.md)**

upoznaćeš **Channel Directions** (`chan<-` i `<-chan`), naučiti kako ograničavaju dozvoljene operacije nad Channel-ima i zašto predstavljaju važan alat za pisanje bezbednijeg i čitljivijeg concurrent Go koda.
