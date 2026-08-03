# Unbuffered Channels

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 3/8
>
> **Fajl:** `docs/03-unbuffered-channels.md`

---

# 📚 Sadržaj

- Šta je Unbuffered Channel?
- Zašto postoji?
- Sinhrona komunikacija
- Handshake između Goroutines
- Blokirajuće ponašanje
- Kada se izvršavanje nastavlja?
- Korak po korak analiza
- Vizuelni dijagrami
- Tipični primeri
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja Unbuffered Channel,
- zašto blokira izvršavanje,
- kada pošiljalac čeka,
- kada primalac čeka,
- zašto predstavlja sinhronizacionu tačku između Goroutines,
- kako Go Runtime koordinira razmenu podataka.

---

# Šta je Unbuffered Channel?

**Unbuffered Channel** je Channel koji **nema interni bafer**.

To znači da se vrednost **ne može privremeno sačuvati** unutar Channel-a.

Vrednost može biti preneta **samo onda kada su istovremeno spremni i pošiljalac i primalac.**

---

# Kako se kreira?

Unbuffered Channel se kreira ovako:

```go
ch := make(chan int)
```

Obrati pažnju.

Ne navodimo veličinu bafera.

---

# Kako izgleda u memoriji?

Možemo ga zamisliti ovako:

```text
Sender

↓

┌──────────────┐
│   Channel    │
│  Buffer: 0   │
└──────────────┘

↓

Receiver
```

Buffer je:

```text
0
```

Zbog toga Channel nema gde da sačuva vrednost.

---

# Najvažnija osobina

Kod Unbuffered Channel-a:

> **Slanje i prijem moraju da se dogode istovremeno.**

Drugim rečima:

Sender i Receiver moraju da se "sastanu".

Ovo se često naziva:

- synchronization point
- rendezvous
- handshake

---

# Handshake

Najbolji način da razumeš Unbuffered Channel jeste da ga posmatraš kao rukovanje.

```text
Sender

🤝

Receiver
```

Tek kada su obe strane prisutne:

- podatak se prenosi,
- obe Goroutines nastavljaju izvršavanje.

---

# Zašto blokira?

Pošto ne postoji bafer:

```
Sender

↓

želi da pošalje

↓

nema gde

↓

čeka Receiver-a
```

Isto važi i za Receiver.

```
Receiver

↓

želi podatak

↓

nema ga

↓

čeka Sender-a
```

---

# Prvi primer

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

Korak po korak.

## Korak 1

Kreira se Channel.

```text
main

↓

make(chan int)
```

---

## Korak 2

Pokreće se nova Goroutine.

```text
go Sender()
```

---

## Korak 3

Sender pokušava:

```go
ch <- 42
```

Pošto nema bafera:

Sender čeka.

---

## Korak 4

Main izvršava:

```go
value := <-ch
```

Receiver je sada spreman.

---

## Korak 5

Go Runtime povezuje obe Goroutines.

```
Sender

↓

42

↓

Receiver
```

---

## Korak 6

Obe Goroutines nastavljaju dalje.

---

# Vizuelni prikaz

```text
Main

 │
 │
 │      make(chan int)
 │
 ▼

Channel

(Buffer = 0)

 ▲
 │
 │

Sender
```

Dok Receiver nije spreman:

```text
Sender

↓

WAIT
```

Kada Receiver stigne:

```text
Sender

↓

42

↓

Receiver
```

---

# Šta ako Receiver kasni?

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	ch := make(chan int)

	go func() {

		fmt.Println("Sending...")

		ch <- 100

		fmt.Println("Done sending")

	}()

	time.Sleep(2 * time.Second)

	fmt.Println(<-ch)
}
```

Mogući izlaz:

```text
Sending...

(dve sekunde čekanja)

100

Done sending
```

---

# Zašto?

Pošiljalac nije mogao da nastavi dalje.

Morao je da sačeka Receiver.

---

# Obrnuta situacija

```go
package main

import (
	"fmt"
	"time"
)

func main() {

	ch := make(chan int)

	go func() {

		time.Sleep(time.Second)

		ch <- 50

	}()

	fmt.Println("Waiting...")

	value := <-ch

	fmt.Println(value)
}
```

Izlaz:

```text
Waiting...

(jedna sekunda)

50
```

Receiver je čekao Sender-a.

---

# Važno pravilo

Kod Unbuffered Channel-a:

Sender čeka Receiver.

Receiver čeka Sender.

Obe strane mogu biti blokirane.

---

# Šta znači "blokiran"?

To znači:

Ta Goroutine ne izvršava naredne instrukcije.

Scheduler je privremeno uklanja sa izvršavanja.

Kada se uslov ispuni:

Scheduler je ponovo aktivira.

---

# Kako Go Runtime ovo rešava?

Veoma uprošćeno:

```
Sender

↓

Channel

↓

nema Receiver-a

↓

Scheduler

↓

WAIT
```

Kasnije:

```
Receiver stiže

↓

Scheduler

↓

prenosi podatak

↓

nastavak obe Goroutines
```

---

# Zašto je ovo korisno?

Zato što predstavlja prirodnu sinhronizaciju.

Na primer:

```
Worker

↓

izračuna rezultat

↓

pošalje rezultat

↓

Main nastavlja
```

Nije potrebno:

- stalno proveravanje,
- beskonačne petlje,
- ručno zaključavanje.

---

# Unbuffered Channel nije Queue

Početnici često misle:

```
Sender

↓

Channel

↓

Receiver
```

i pretpostavljaju da Channel čuva vrednosti.

Kod Unbuffered Channel-a to nije tačno.

Ako Receiver nije spreman:

nema skladištenja.

Sender čeka.

---

# Više Sender-a

Primer:

```text
Sender A

↓

Sender B

↓

Channel

↓

Receiver
```

Samo jedan Sender može uspešno predati vrednost.

Ostali čekaju.

---

# Više Receiver-a

```text
Sender

↓

Channel

↓

Receiver A

↓

Receiver B
```

Jedna poslata vrednost odlazi samo jednom Receiver-u.

Channel ne kopira vrednosti svim primaocima.

---

# Da li postoji redosled?

Ako više Goroutines čeka isti Channel:

Go specifikacija **ne garantuje** redosled kojim će neka od njih nastaviti izvršavanje.

Ne treba pisati kod koji zavisi od određenog redosleda.

---

# Kada koristiti Unbuffered Channel?

Kada želiš:

- direktnu sinhronizaciju,
- potvrdu da je druga Goroutine preuzela podatak,
- koordinaciju rada,
- signalizaciju događaja,
- razmenu pojedinačnih vrednosti.

---

# Kada nije najbolji izbor?

Ako želiš:

- da pošiljalac nastavi odmah,
- da privremeno sačuvaš više vrednosti,
- da proizvođač bude brži od potrošača,

onda je pogodniji **Buffered Channel**, koji ćemo obraditi u sledećoj lekciji.

---

# 🚨 Najčešće greške

## 1. Slanje bez Receiver-a

```go
ch <- 5
```

Program će čekati.

---

## 2. Primanje bez Sender-a

```go
value := <-ch
```

Program će čekati.

---

## 3. Pretpostavljanje da Channel čuva podatke

Unbuffered Channel nema bafer.

Ne postoji mesto gde bi vrednost bila sačuvana.

---

## 4. Korišćenje `time.Sleep()` za koordinaciju

```go
time.Sleep(time.Second)
```

Ovo nije mehanizam sinhronizacije.

Koristi se samo u demonstracionim primerima.

---

# ✅ Best Practices

- Koristi Unbuffered Channel kada želiš sinhronizaciju između dve Goroutines.
- Nemoj pretpostavljati redosled izvršavanja.
- Nemoj koristiti `time.Sleep()` kao zamenu za pravilnu koordinaciju.
- Ako Sender treba da nastavi bez čekanja, razmisli o Buffered Channel-u.
- Neka jedna razmena predstavlja jednu logičku poruku.

---

# 📋 Rezime

- Unbuffered Channel nema interni bafer.
- Kreira se pomoću:

```go
make(chan T)
```

- Sender čeka Receiver-a.
- Receiver čeka Sender-a.
- Razmena podataka predstavlja sinhronizacionu tačku.
- Vrednost se ne skladišti unutar Channel-a.
- Unbuffered Channel je idealan kada želiš da obe strane budu usklađene.

---

# ❓ Pitanja za proveru znanja

1. Šta je Unbuffered Channel?
2. Kolika je veličina njegovog bafera?
3. Zašto Sender blokira?
4. Zašto Receiver blokira?
5. Šta znači handshake između Goroutines?
6. Da li Unbuffered Channel čuva podatke?
7. Da li jedna vrednost može biti isporučena svim Receiver-ima?
8. Da li Go garantuje redosled izvršavanja Goroutines koje čekaju isti Channel?
9. Kada koristiti Unbuffered Channel?
10. Kada je bolje koristiti Buffered Channel?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi program u kojem jedna Goroutine šalje broj, a druga ga prima.
2. Dodaj ispis pre i posle slanja i objasni redosled izvršavanja.
3. Dodaj ispis pre i posle prijema i analiziraj kada se izvršavaju.

## 🟡 Srednje

4. Napravi tri Sender Goroutines koje šalju vrednosti na isti Unbuffered Channel i jedan Receiver koji ih prima.
5. Napravi dva Receiver-a koji čekaju isti Channel i posmatraj koji prima vrednosti.
6. Dodaj različita `time.Sleep()` kašnjenja i objasni kako utiču na izvršavanje.

## 🟠 Izazov

7. Napravi program u kojem jedan Worker obrađuje niz brojeva i svaki rezultat šalje preko Unbuffered Channel-a glavnoj Goroutine. Objasni zašto Worker ne može da nastavi sa sledećim brojem dok prethodni nije preuzet.

---

### ➡️ Sledeća lekcija **[**Buffered Channels**](04-buffered-channels.md)**

naučićeš kako funkcionišu **Buffered Channels**, kako interni bafer menja ponašanje pošiljaoca i primaoca i kada je prikladnije koristiti njih umesto Unbuffered Channel-a.
