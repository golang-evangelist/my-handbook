# Goroutines

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 1/8
>
> **Fajl:** `docs/01-goroutines.md`

---

# 📚 Sadržaj

- Šta su Goroutines?
- Zašto su uvedene?
- Thread vs Goroutine
- Kako radi ključna reč `go`
- Kako Goroutines izvršava Go Runtime
- Životni ciklus Goroutine
- Stack Goroutine-a
- Kreiranje prve Goroutine
- Anonymous Goroutines
- Prosleđivanje argumenata
- Povratne vrednosti
- Goroutines i `main()`
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta je Goroutine,
- kako nastaje,
- kako je Go izvršava,
- zbog čega je mnogo lakša od OS thread-a,
- kako se pokreće,
- koje su njene prednosti,
- koje su njene mane,
- koje greške početnici najčešće prave.

---

# Šta je Goroutine?

**Goroutine** predstavlja veoma laganu jedinicu izvršavanja koju pokreće **Go Runtime**.

Možeš je zamisliti kao funkciju koja se izvršava nezavisno od funkcije koja ju je pokrenula.

Drugim rečima:

> Goroutine omogućava da više funkcija napreduje konkurentno.

---

# Prva važna činjenica

Goroutine **nije** isto što i thread operativnog sistema.

To je jedna od najvećih zabluda kod početnika.

Go Runtime poseduje sopstveni scheduler koji upravlja Goroutines i raspoređuje ih na OS thread-ove.

Dakle:

```
Tvoj kod

↓

Go Runtime

↓

Go Scheduler

↓

OS Threads

↓

CPU
```

Programer uglavnom nikada direktno ne upravlja thread-ovima.

---

# Zašto su Goroutines uvedene?

Klasični thread-ovi imaju nekoliko mana.

Na primer:

- relativno su skupi,
- kreiranje traje duže,
- zauzimaju više memorije,
- operativni sistem mora njima da upravlja.

Go je želeo nešto lakše.

Rezultat su:

**Goroutines.**

---

# Thread vs Goroutine

| Thread | Goroutine |
|----------|-----------|
| Kreira ih operativni sistem | Kreira ih Go Runtime |
| Relativno skup | Veoma lagana |
| Veći početni stack | Mali početni stack koji raste po potrebi |
| OS scheduler | Go scheduler |
| Sporije kreiranje | Veoma brzo kreiranje |
| Manji broj thread-ova | Veliki broj Goroutines |

---

# Koliko Goroutines možemo imati?

Ne postoji tačan broj.

Zavisi od:

- raspoložive memorije,
- posla koji obavljaju,
- trajanja izvršavanja,
- scheduler-a.

U praksi nije neobično imati:

- nekoliko hiljada,
- desetine hiljada,
- pa čak i stotine hiljada Goroutines.

To bi sa klasičnim thread-ovima bilo mnogo teže postići.

---

# Kako se pokreće Goroutine?

Koristi se ključna reč:

```go
go
```

Sintaksa:

```go
go nekaFunkcija()
```

To je sve.

Jedna ključna reč.

---

# Prvi primer

```go
package main

import "fmt"

func hello() {
	fmt.Println("Hello from Goroutine!")
}

func main() {
	go hello()

	fmt.Println("Hello from main!")
}
```

---

# Da li će ovo raditi?

Ne nužno.

Vrlo često će ispis biti:

```text
Hello from main!
```

a nekada:

```text
Hello from main!
Hello from Goroutine!
```

Zašto?

---

# Šta se zapravo dogodilo?

Korak po korak:

```
main()

↓

pokreće go hello()

↓

scheduler registruje novu Goroutine

↓

main nastavlja dalje

↓

main završava

↓

program se gasi
```

Ako se program završi pre nego što scheduler izvrši novu Goroutine, ona nikada neće dobiti priliku da se izvrši.

---

# Vizuelni prikaz

```text
main Goroutine

Start
 │
 │
 ├─────────────── go hello()
 │
 │
 ├─────────────── kraj programa
 │
 ▼

hello Goroutine

čekanje...
```

Program se završio.

Nova Goroutine nije stigla na red.

---

# Privremeno rešenje

Kasnije ćemo koristiti `sync.WaitGroup`.

Za sada možemo koristiti:

```go
package main

import (
	"fmt"
	"time"
)

func hello() {
	fmt.Println("Hello from Goroutine!")
}

func main() {
	go hello()

	time.Sleep(time.Second)

	fmt.Println("Program finished.")
}
```

---

# Zašto `time.Sleep()` nije dobro rešenje?

Zato što:

- nije pouzdano,
- usporava program,
- zavisi od procene vremena,
- nije production pristup.

Koristi se samo za demonstraciju.

Kasnije ćemo koristiti:

- `sync.WaitGroup`
- `context`
- channels

---

# Anonymous Goroutines

Ne moraš pozivati postojeću funkciju.

Možeš koristiti anonimnu funkciju.

```go
package main

import "fmt"

func main() {

	go func() {
		fmt.Println("Anonymous Goroutine")
	}()

	fmt.Println("Main")
}
```

Primeti:

```go
}()
```

Na kraju anonimnu funkciju odmah pozivamo.

---

# Prosleđivanje argumenata

Naravno, moguće je.

```go
package main

import (
	"fmt"
	"time"
)

func greet(name string) {
	fmt.Println("Hello,", name)
}

func main() {

	go greet("Marko")

	time.Sleep(time.Second)
}
```

---

# Anonymous funkcija sa argumentima

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	go func(name string) {
		fmt.Println("Hello,", name)
	}("Marko")

	time.Sleep(time.Second)
}
```

---

# Mogu li Goroutines vratiti rezultat?

Ne direktno.

Ovo nije moguće:

```go
result := go calculate()
```

Ne postoji takva sintaksa.

Razlog je jednostavan.

Goroutine radi konkurentno.

Rezultat možda još nije spreman.

Kasnije ćemo naučiti kako se rezultati razmenjuju pomoću:

- Channels
- Context
- Synchronization mehanizama

---

# Kako Go Runtime izvršava Goroutines?

Kada napišeš:

```go
go work()
```

Go Runtime:

1. kreira novu Goroutine,
2. dodeljuje joj početni stack,
3. registruje je kod scheduler-a,
4. scheduler odlučuje kada će je izvršiti.

Dakle:

```
go work()

↓

Runtime

↓

Scheduler

↓

OS Thread

↓

CPU
```

---

# Stack Goroutine-a

Jedna veoma važna karakteristika.

Klasični thread obično dobija veliki stack unapred.

Kod Goroutine-a to nije slučaj.

Početni stack je mali.

Po potrebi:

```
2 KB

↓

4 KB

↓

8 KB

↓

16 KB

↓

...
```

Go Runtime ga automatski povećava kada zatreba.

Zbog toga Goroutines troše znatno manje memorije.

---

# Životni ciklus Goroutine

```text
Kreiranje

↓

Scheduler

↓

Izvršavanje

↓

Blokiranje (ako čeka)

↓

Ponovno izvršavanje

↓

Završetak
```

Scheduler upravlja svim prelazima između ovih stanja.

---

# Da li postoji redosled izvršavanja?

Ne.

Na primer:

```go
go A()

go B()

go C()
```

Ne postoji garancija da će:

```
A

↓

B

↓

C
```

biti izvršeni tim redom.

Moguće je:

```
C

↓

A

↓

B
```

ili bilo koji drugi redosled.

Nikada nemoj pretpostavljati redosled izvršavanja Goroutines.

---

# Kada koristiti Goroutines?

Najčešće za:

- HTTP servere,
- obradu fajlova,
- mrežne zahteve,
- obradu velikog broja nezavisnih zadataka,
- streaming,
- background poslove,
- worker pool sisteme.

---

# Kada ih ne koristiti?

Ako:

- postoji samo jedna kratka operacija,
- nema čekanja,
- nema potrebe za konkurentnim izvršavanjem,
- kod postaje komplikovaniji nego što donosi koristi.

---

# 🚨 Najčešće greške

## 1. Zaboravljen WaitGroup

Program se završi pre Goroutine.

---

## 2. Pretpostavljanje redosleda izvršavanja

Scheduler to ne garantuje.

---

## 3. Previše Goroutines

Više nije uvek bolje.

Ako pokreneš milione Goroutines bez potrebe, potrošnja memorije i scheduler overhead će porasti.

---

## 4. Korišćenje `time.Sleep()` za sinhronizaciju

To je samo demonstraciono rešenje.

---

# ✅ Best Practices

- Koristi Goroutines kada postoji stvarni razlog.
- Nemoj pretpostavljati redosled izvršavanja.
- Koristi `sync.WaitGroup` za čekanje završetka.
- Nemoj koristiti `time.Sleep()` za sinhronizaciju u produkcionom kodu.
- Drži Goroutines što jednostavnijim i fokusiranim na jedan zadatak.

---

# 📋 Rezime

- Goroutine je lagana jedinica izvršavanja kojom upravlja Go Runtime.
- Pokreće se pomoću ključne reči `go`.
- Nije isto što i OS thread.
- Go Scheduler odlučuje kada će se izvršavati.
- Ne postoji garantovan redosled izvršavanja.
- Program se završava kada se završi `main()` Goroutine.
- `time.Sleep()` nije ispravan način sinhronizacije.

---

# ❓ Pitanja za proveru znanja

1. Šta je Goroutine?
2. Po čemu se razlikuje od thread-a?
3. Šta radi ključna reč `go`?
4. Zašto se Goroutine nekada ne izvrši?
5. Ko upravlja Goroutines?
6. Da li postoji garantovan redosled izvršavanja?
7. Zašto `time.Sleep()` nije dobro rešenje?
8. Da li Goroutine može direktno vratiti rezultat?
9. Šta je početni stack Goroutine-a?
10. Zašto su Goroutines pogodne za veliki broj konkurentnih zadataka?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Pokreni jednu Goroutine koja ispisuje poruku.
2. Pokreni dve Goroutines koje ispisuju različite tekstove.
3. Prosledi argument anonimnoj Goroutine.
4. Napravi funkciju `printNumbers()` i pokreni je kao Goroutine.
5. Posmatraj različite redoslede ispisa pri višestrukom pokretanju programa.

## 🟡 Srednje

6. Pokreni 10 Goroutines koje ispisuju svoj indeks.
7. Napravi funkciju koja simulira posao pomoću `time.Sleep()`.
8. Pokreni više Goroutines sa različitim trajanjem izvršavanja i analiziraj izlaz.
9. Objasni zašto se redosled ispisa menja između pokretanja.
10. Izmeni primer tako da se `main()` prerano završi i objasni rezultat.

## 🟠 Izazov

11. Napravi program koji pokreće 100 Goroutines, pri čemu svaka ispisuje svoj identifikator. U sledećim lekcijama zameni `time.Sleep()` odgovarajućim mehanizmom za sinhronizaciju.

---

### ➡️ Sledeća lekcija **[**Channels**](02-channels.md)**

naučićeš šta su **Channels**, zašto predstavljaju osnovni mehanizam komunikacije između Goroutines i kako omogućavaju bezbednu razmenu podataka bez direktnog deljenja memorije.
