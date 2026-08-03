# Channel Directions (Send-only / Receive-only Channels)

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 5/8
>
> **Fajl:** `docs/05-channel-directions.md`

---

# 📚 Sadržaj

- Šta su Channel Directions?
- Zašto postoje?
- Bidirectional Channel
- Send-only Channel (`chan<-`)
- Receive-only Channel (`<-chan`)
- Konverzija između tipova Channel-a
- Channel Directions u funkcijama
- Prednosti u API dizajnu
- Compile-time zaštita
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavljaju Channel Directions,
- kako ograničavaju dozvoljene operacije nad Channel-om,
- kada koristiti `chan<-`,
- kada koristiti `<-chan`,
- kako povećavaju bezbednost i čitljivost koda,
- zašto ih Go proverava u vreme kompajliranja.

---

# Podsećanje

Do sada smo koristili običan Channel.

Na primer:

```go
ch := make(chan int)
```

Nad njim možemo da radimo obe operacije.

Možemo da šaljemo:

```go
ch <- 10
```

i možemo da primamo:

```go
value := <-ch
```

Ovakav Channel naziva se:

> **Bidirectional Channel**

---

# Šta je Bidirectional Channel?

Bidirectional Channel omogućava:

- slanje,
- prijem.

Drugim rečima:

```
Sender

↓

Channel

↓

Receiver
```

ali i obrnuto, ukoliko se isti Channel prosledi drugim Goroutines.

Tip:

```go
chan int
```

---

# Zašto postoje Channel Directions?

Zamisli funkciju koja treba **isključivo da šalje** podatke.

```go
func producer(...) {
    ...
}
```

Ako joj prosledimo običan Channel:

```go
chan int
```

ona može:

- da šalje,
- da prima,
- da zatvori Channel.

To možda nije ono što želimo.

Bilo bi bolje da jasno kažemo:

> "Ova funkcija sme samo da šalje podatke."

Upravo zbog toga postoje Channel Directions.

---

# Dve vrste ograničenja

Go omogućava dva posebna tipa Channel-a.

## Send-only

```go
chan<- int
```

Može samo da šalje.

---

## Receive-only

```go
<-chan int
```

Može samo da prima.

---

# Bidirectional vs Send-only vs Receive-only

| Tip | Slanje | Prijem |
|------|:------:|:------:|
| `chan T` | ✅ | ✅ |
| `chan<- T` | ✅ | ❌ |
| `<-chan T` | ❌ | ✅ |

---

# Send-only Channel

Sintaksa:

```go
chan<- int
```

Strelica pokazuje:

```
→
```

od funkcije ka Channel-u.

Možemo ga čitati kao:

> "Ova funkcija šalje podatke u Channel."

---

# Primer

```go
package main

import "fmt"

func producer(ch chan<- int) {
	ch <- 100
}

func main() {

	ch := make(chan int)

	go producer(ch)

	fmt.Println(<-ch)
}
```

Izlaz:

```text
100
```

---

# Šta je dozvoljeno?

```go
func producer(ch chan<- int) {

	ch <- 10

	ch <- 20
}
```

Sve je ispravno.

---

# Šta nije dozvoljeno?

```go
func producer(ch chan<- int) {

	value := <-ch

	fmt.Println(value)
}
```

Compiler prijavljuje grešku.

Jer:

```
chan<-

↓

ne može da prima podatke.
```

---

# Receive-only Channel

Sintaksa:

```go
<-chan int
```

Strelica pokazuje:

```
←
```

ka funkciji.

Možemo ga čitati kao:

> "Ova funkcija prima podatke iz Channel-a."

---

# Primer

```go
package main

import "fmt"

func consumer(ch <-chan int) {

	value := <-ch

	fmt.Println(value)
}

func main() {

	ch := make(chan int)

	go consumer(ch)

	ch <- 55
}
```

Izlaz:

```text
55
```

---

# Šta je dozvoljeno?

```go
func consumer(ch <-chan int) {

	fmt.Println(<-ch)
}
```

Sve je ispravno.

---

# Šta nije dozvoljeno?

```go
func consumer(ch <-chan int) {

	ch <- 10
}
```

Compiler prijavljuje grešku.

Receive-only Channel ne može da šalje podatke.

---

# Kako nastaju Directional Channels?

Važno je razumeti:

Ne možemo direktno kreirati:

```go
make(chan<- int)
```

ili

```go
make(<-chan int)
```

U praksi gotovo uvek prvo kreiramo:

```go
ch := make(chan int)
```

To je:

```
Bidirectional Channel
```

Zatim ga prosleđujemo funkciji koja očekuje:

```go
chan<- int
```

ili

```go
<-chan int
```

Go automatski ograničava dozvoljene operacije.

---

# Vizuelni prikaz

```text
             make(chan int)

                    │

                    ▼

         Bidirectional Channel

          ▲                 ▲

          │                 │

   chan<- int         <-chan int

Send-only            Receive-only
```

---

# Konverzija tipova

Bidirectional Channel može biti prosleđen funkciji koja očekuje:

```go
chan<- int
```

ili

```go
<-chan int
```

Primer:

```go
ch := make(chan int)

producer(ch)

consumer(ch)
```

Sve radi bez eksplicitne konverzije.

---

# Obrnuta konverzija nije dozvoljena

Ako imaš:

```go
var send chan<- int
```

ne možeš napisati:

```go
var both chan int = send
```

To nije dozvoljeno.

Razlog:

Go ne želi da ukloni ograničenje koje si definisao.

---

# Zašto je ovo korisno?

Pogledaj funkciju:

```go
func producer(ch chan<- Job)
```

Odmah znaš:

- ona proizvodi podatke,
- ne čita podatke,
- koristi Channel samo za slanje.

Slično:

```go
func consumer(ch <-chan Job)
```

Odmah znaš:

- ona samo prima podatke,
- ne može slučajno poslati novu vrednost.

Kod postaje mnogo čitljiviji.

---

# Compile-time zaštita

Jedna od najvećih prednosti.

Greška se otkriva:

```
Pre pokretanja programa.
```

Ne čekaš Runtime.

Ne dobijaš panic.

Compiler odmah prijavljuje problem.

To je jedan od razloga zašto su Directional Channels veoma korisni u većim projektima.

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

Već iz potpisa funkcija možeš zaključiti njihove odgovornosti.

---

# Da li Directional Channel utiče na Runtime?

Ne.

Ograničenje postoji na nivou tipova.

Runtime ne dobija "brži" ili "sporiji" Channel.

Razlika je isključivo:

- bezbednost,
- čitljivost,
- API dizajn.

---

# Kada koristiti Directional Channels?

Preporučljivo je kada:

- pišeš biblioteke,
- pišeš javni API,
- imaš Producer-Consumer arhitekturu,
- želiš jasne odgovornosti funkcija,
- želiš da compiler spreči pogrešnu upotrebu Channel-a.

---

# Kada nije neophodno?

Kod vrlo malih primera:

```go
func main() {

	ch := make(chan int)

	...
}
```

nije obavezno koristiti Directional Channels.

Ali kako projekat raste, postaju veoma korisni.

---

# 🚨 Najčešće greške

## 1. Pogrešan smer strelice

Početnici često zamene:

```go
chan<-
```

i

```go
<-chan
```

Zapamti:

```text
chan<-

↓

šalje
```

```text
<-chan

↓

prima
```

---

## 2. Pokušaj prijema iz Send-only Channel-a

```go
value := <-ch
```

Compiler prijavljuje grešku.

---

## 3. Pokušaj slanja u Receive-only Channel

```go
ch <- 10
```

Compiler prijavljuje grešku.

---

## 4. Nepotrebno korišćenje Bidirectional Channel-a

Ako funkcija samo šalje podatke:

```go
func producer(...)
```

koristi:

```go
chan<-
```

To jasnije opisuje namenu funkcije.

---

# ✅ Best Practices

- Koristi `chan<-` za proizvođače podataka.
- Koristi `<-chan` za potrošače podataka.
- Ograniči mogućnosti funkcije na ono što joj je zaista potrebno.
- Koristi Directional Channels u javnim API-jima i bibliotekama.
- Neka compiler bude tvoj saveznik — iskoristi proveru tipova za sprečavanje grešaka.

---

# 📋 Rezime

- Običan `chan T` omogućava slanje i prijem.
- `chan<- T` omogućava samo slanje.
- `<-chan T` omogućava samo prijem.
- Bidirectional Channel može biti prosleđen funkciji koja očekuje Directional Channel.
- Obrnuta konverzija nije dozvoljena.
- Directional Channels povećavaju čitljivost i bezbednost koda.
- Ograničenja proverava compiler, a ne Runtime.

---

# ❓ Pitanja za proveru znanja

1. Šta je Bidirectional Channel?
2. Šta predstavlja `chan<- T`?
3. Šta predstavlja `<-chan T`?
4. Zašto postoje Channel Directions?
5. Da li `chan<-` može da prima podatke?
6. Da li `<-chan` može da šalje podatke?
7. Da li Bidirectional Channel može biti prosleđen funkciji koja očekuje `chan<-`?
8. Zašto obrnuta konverzija nije dozvoljena?
9. Da li Directional Channels utiču na performanse Runtime-a?
10. Kada ih je preporučljivo koristiti?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napiši funkciju `producer(ch chan<- int)` koja šalje pet brojeva.
2. Napiši funkciju `consumer(ch <-chan int)` koja prima i ispisuje pet brojeva.
3. Prosledi jedan Bidirectional Channel objema funkcijama.

---

## 🟡 Srednje

4. Napravi Producer koji šalje brojeve od 1 do 100.
5. Napravi Consumer koji računa zbir primljenih brojeva.
6. Pokušaj da pošalješ podatak iz Receive-only Channel-a i objasni grešku kompajlera.
7. Pokušaj da primiš podatak iz Send-only Channel-a i objasni grešku kompajlera.

---

## 🟠 Izazov

8. Implementiraj jednostavan **Producer-Consumer** sistem koristeći:
   - jedan `chan<-`,
   - jedan `<-chan`,
   - jedan Bidirectional Channel u `main()` funkciji.

Objasni zašto ovakav dizajn bolje opisuje odgovornosti funkcija nego korišćenje običnog `chan T` svuda.

---

### ➡️ Sledeća lekcija **[**`range` nad Channel-om**](06-range-over-channels.md)**

naučićeš kako funkcioniše `range` nad Channel-om, kako omogućava elegantno primanje više vrednosti i zašto je to najčešći način čitanja podataka iz Channel-a u Go programima.
