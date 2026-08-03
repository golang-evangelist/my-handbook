# Channels

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 2/8
>
> **Fajl:** `docs/02-channels.md`

---

# 📚 Sadržaj

- Šta su Channels?
- Zašto postoje?
- CSP filozofija
- Komunikacija između Goroutines
- Deklaracija i kreiranje Channel-a
- Slanje i prijem podataka
- Blokirajuće ponašanje
- Channel kao "cev" za podatke
- Tok izvršavanja
- Tipovi Channel-a
- Zero value Channel-a
- `nil` Channel
- Kada koristiti Channels?
- Kada ih ne koristiti?
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja Channel,
- zašto je uveden u Go,
- kako se kreira,
- kako se šalju i primaju podaci,
- zašto Channels blokiraju izvršavanje,
- kako izgleda komunikacija između Goroutines.

---

# Šta je Channel?

**Channel** je mehanizam koji omogućava bezbednu komunikaciju između Goroutines.

Njegova osnovna uloga je:

> **Prenos podataka između dve ili više Goroutines.**

Channels nisu namenjeni skladištenju podataka.

Njihova primarna svrha je:

- komunikacija,
- sinhronizacija,
- koordinacija rada Goroutines.

---

# Najvažnija ideja

Veoma često ćeš čuti poznatu Go filozofiju:

> **Don't communicate by sharing memory.**
>
> **Share memory by communicating.**

Drugim rečima:

Umesto da dve Goroutines direktno pristupaju istoj promenljivoj u memoriji, mnogo je bezbednije da međusobno razmenjuju podatke preko Channel-a.

To značajno smanjuje mogućnost nastanka mnogih grešaka koje ćemo kasnije upoznati, poput **Data Race**.

---

# Zašto postoje Channels?

Zamisli dve Goroutines.

Jedna proizvodi podatke.

Druga ih obrađuje.

```
Producer

↓

???

↓

Consumer
```

Kako će komunicirati?

Jedna mogućnost je:

- deljena memorija,
- Mutex,
- zaključavanje,
- sinhronizacija.

Druga mogućnost:

```
Producer

↓

Channel

↓

Consumer
```

Go favorizuje upravo ovaj pristup.

---

# Analogija iz stvarnog života

Zamisli pokretnu traku u fabrici.

```
Radnik A

↓

==================

↓

Radnik B
```

Radnik A stavlja proizvod na traku.

Radnik B ga preuzima.

Ne moraju istovremeno da dodiruju isti predmet.

Channel radi veoma slično.

---

# Channel kao cev

Možemo ga zamisliti kao cev.

```
Sender

↓

════════════════════

↓

Receiver
```

Podaci "putuju" kroz cev.

Pošiljalac ne zna ko će ih preuzeti.

Primalac ne zna ko ih je poslao.

Važno je samo da oba koriste isti Channel.

---

# Channel je tipiziran

Svaki Channel prenosi samo jedan tip podataka.

Na primer:

```go
chan int
```

prenosi:

```go
int
```

---

```go
chan string
```

prenosi:

```go
string
```

---

```go
chan bool
```

prenosi:

```go
bool
```

---

Moguće je koristiti i:

```go
chan User
```

ili

```go
chan *User
```

Dakle, Channel može prenositi gotovo svaki Go tip.

---

# Deklaracija Channel-a

Deklaracija izgleda ovako:

```go
var ch chan int
```

Ovime je deklarisan Channel koji prenosi `int` vrednosti.

Ali postoji važna stvar.

On još nije spreman za korišćenje.

---

# Zero Value Channel-a

Kao i svi ostali tipovi u Go-u, i Channel ima svoju podrazumevanu vrednost.

To je:

```go
nil
```

Na primer:

```go
var ch chan int

fmt.Println(ch)
```

Ispis:

```text
<nil>
```

---

# Šta znači nil Channel?

To znači da Channel još nije kreiran.

On postoji kao promenljiva.

Ali nema internu strukturu pomoću koje bi mogao da prenosi podatke.

---

# Kreiranje Channel-a

Za kreiranje koristimo funkciju:

```go
make()
```

Na primer:

```go
ch := make(chan int)
```

Sada je Channel spreman.

---

# Šta radi make?

Go Runtime kreira internu strukturu Channel-a.

Možemo to zamisliti ovako:

```
make(chan int)

↓

Go Runtime

↓

alokacija memorije

↓

Channel spreman za rad
```

---

# Slanje podataka

Operator za slanje je:

```go
<-
```

Primer:

```go
ch <- 10
```

Čita se:

> Pošalji vrednost 10 u Channel.

---

# Prijem podataka

Prijem izgleda ovako:

```go
value := <-ch
```

Čita se:

> Preuzmi vrednost iz Channel-a.

---

# Prvi kompletan primer

```go
package main

import (
	"fmt"
)

func main() {

	ch := make(chan int)

	go func() {
		ch <- 42
	}()

	value := <-ch

	fmt.Println(value)
}
```

Izlaz:

```text
42
```

---

# Šta se ovde dogodilo?

Korak po korak:

```
main

↓

kreira Channel

↓

pokreće Goroutine

↓

Goroutine šalje 42

↓

main čeka podatak

↓

dobija 42

↓

ispisuje rezultat
```

---

# Vizuelni prikaz

```text
main

──────────────┐
              │
              ▼

        make(chan int)

              │

              ▼

        go Sender()

              │

              ▼

          Receiver

        value := <-ch

              ▲

              │

Sender

ch <- 42
```

---

# Da li Channel kopira podatke?

Da.

Kada šaljemo obične vrednosti:

```go
ch <- number
```

šalje se kopija vrednosti.

Ako šaljemo pokazivač:

```go
chan *User
```

onda se kopira pokazivač, a ne objekat na koji pokazuje.

To je isto ponašanje kao kod prosleđivanja argumenata funkcijama.

---

# Šta ako niko ne prima podatke?

Primer:

```go
package main

func main() {

	ch := make(chan int)

	ch <- 10
}
```

Šta će se dogoditi?

Program će stati.

Zašto?

Zato što nema Goroutine koja prima podatke.

---

# Šta ako niko ne šalje podatke?

```go
package main

func main() {

	ch := make(chan int)

	value := <-ch

	_ = value
}
```

I ovaj program će stati.

Zašto?

Niko nije poslao vrednost.

---

# Zašto se ovo dešava?

Po podrazumevanom ponašanju, Channel predstavlja **sinhronizacionu tačku**.

To znači:

Pošiljalac i primalac moraju da se "sastanu".

Ako jedna strana nije spremna, druga čeka.

Ovo ponašanje ćemo detaljno proučiti u sledećoj lekciji o **Unbuffered Channels**.

---

# Blokirajuće ponašanje

Kod običnog Channel-a:

```
Sender

↓

čeka

↓

Receiver

↓

nastavak izvršavanja
```

I obrnuto.

```
Receiver

↓

čeka

↓

Sender

↓

nastavak izvršavanja
```

Ovo je jedna od najvažnijih osobina Channel-a.

---

# Tok izvršavanja

Primer:

```go
go func() {
	ch <- 100
}()

fmt.Println("Čekam podatak...")

value := <-ch

fmt.Println(value)
```

Mogući tok:

```
Main

↓

Čekam podatak

↓

blokira se

↓

Sender šalje

↓

Receiver dobija

↓

nastavak programa
```

---

# Channels mogu povezati mnogo Goroutines

Ne moraju postojati samo dve.

Na primer:

```
Worker 1

↓

Worker 2

↓

Worker 3

↓

Channel

↓

Aggregator
```

Ili:

```
Producer

↓

Channel

↓

Worker A

Worker B

Worker C
```

Kasnije ćemo ovo upoznati kao:

- Worker Pool
- Fan-Out
- Fan-In
- Pipeline

---

# Kada koristiti Channels?

Najčešće za:

- komunikaciju između Goroutines,
- prosleđivanje rezultata,
- sinhronizaciju,
- koordinaciju rada,
- pipeline obradu,
- worker pool sisteme,
- event obradu.

---

# Kada ne koristiti Channels?

Channels nisu univerzalno rešenje.

Ako dve funkcije rade sekvencijalno u istoj Goroutine:

```go
result := calculate()

print(result)
```

nema potrebe za Channel-om.

Takođe, ako nema konkurentnog izvršavanja, Channel često samo komplikuje kod.

---

# 🚨 Najčešće greške

## 1. Zaboravljen make()

```go
var ch chan int

ch <- 5
```

Program će blokirati zauvek, jer je `ch` `nil`.

---

## 2. Slanje bez primaoca

```go
ch <- value
```

Ako nema primaoca, izvršavanje će čekati.

---

## 3. Primanje bez pošiljaoca

```go
value := <-ch
```

Ako niko ne šalje podatke, izvršavanje će čekati.

---

## 4. Korišćenje Channel-a tamo gde nije potreban

Ne koristi Channel samo zato što postoji.

Sekvencijalni kod je često jednostavniji i čitljiviji.

---

# ✅ Best Practices

- Koristi Channels za komunikaciju između Goroutines.
- Uvek kreiraj Channel pomoću `make()`.
- Nemoj koristiti `nil` Channel.
- Koristi smislen naziv promenljive (`jobs`, `results`, `messages`) umesto generičkog `ch` kada to poboljšava čitljivost.
- Nemoj uvoditi Channel ako običan poziv funkcije rešava problem jednostavnije.

---

# 📋 Rezime

- Channel omogućava komunikaciju između Goroutines.
- Kreira se pomoću `make()`.
- Deklarisan, a neinicijalizovan Channel ima vrednost `nil`.
- Podaci se šalju operatorom `<-`.
- Podaci se primaju operatorom `<-`.
- Channel je tipiziran.
- Podrazumevano ponašanje je blokirajuće.
- Channels služe za komunikaciju i sinhronizaciju, a ne kao zamena za svaku promenljivu.

---

# ❓ Pitanja za proveru znanja

1. Šta je Channel?
2. Koja je njegova osnovna svrha?
3. Šta znači CSP filozofija?
4. Kako se kreira Channel?
5. Koja je podrazumevana vrednost neinicijalizovanog Channel-a?
6. Kako se šalju podaci?
7. Kako se primaju podaci?
8. Šta će se dogoditi ako nema primaoca?
9. Šta će se dogoditi ako nema pošiljaoca?
10. Zašto se kaže da je Channel sinhronizaciona tačka?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Kreiraj `chan int` i pošalji jednu vrednost između dve Goroutines.
2. Kreiraj `chan string` i prosledi tekstualnu poruku.
3. Napravi `chan bool` kojim će jedna Goroutine obavestiti drugu da je završila posao.
4. Isprobaj šta će se dogoditi ako izostaviš `make()`.
5. Isprobaj šta će se dogoditi ako ukloniš Goroutine koja prima podatke.

## 🟡 Srednje

6. Napravi dve Goroutines koje razmenjuju tri uzastopne poruke preko jednog Channel-a.
7. Prosledi strukturu (`struct`) kroz Channel i ispiši njen sadržaj.
8. Prosledi pokazivač (`*struct`) kroz Channel i objasni razliku u odnosu na prosleđivanje vrednosti.
9. Napravi funkciju koja vraća Channel i šalje jedan rezultat.
10. Analiziraj koje Goroutines čekaju u svakom koraku izvršavanja.

## 🟠 Izazov

11. Napravi program sa jednim proizvođačem i jednim potrošačem koji razmenjuju brojeve preko Channel-a. Za sada koristi samo osnovni Channel — u sledećoj lekciji objasnićemo zašto se pošiljalac i primalac međusobno blokiraju.

---

### ➡️ Sledeća lekcija **[**Unbuffered Channels**](03-unbuffered-channels.md)**

detaljno ćemo proučiti **Unbuffered Channels**, njihovo sinhronizaciono ponašanje, zašto blokiraju i zbog čega predstavljaju osnovu većine Go concurrency obrazaca.
