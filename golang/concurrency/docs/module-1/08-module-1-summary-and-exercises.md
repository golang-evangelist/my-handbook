# Modul #1 — Rezime i praktične vežbe

> **Modul:** #1 — Vrlo lako (osnove)
>
> **Lekcija:** 8/8
>
> **Fajl:** `docs/08-module-1-summary-and-exercises.md`

---

# 📚 Sadržaj

- Šta smo naučili?
- Mapa znanja Modula #1
- Kako su sve oblasti povezane?
- Kada koristiti koji tip Channel-a?
- Tipične početničke greške
- Pravila koja treba zapamtiti
- Checklist znanja
- Praktični mini projekti
- Veliki završni zadaci
- Šta sledi u Modulu #2?

---

# 🎯 Cilj lekcije

Nakon završetka Modula #1 trebalo bi da:

- razumeš osnovni model konkurentnog programiranja u Go-u,
- umeš da pokreneš više Goroutines,
- razumeš komunikaciju preko Channel-a,
- znaš razliku između Buffered i Unbuffered Channel-a,
- pravilno koristiš `range` i `close()`,
- umeš da napišeš jednostavne concurrent programe bez dodatnih sinhronizacionih mehanizama.

---

# 🧠 Šta smo naučili?

Tokom ovog modula obradili smo sledeće oblasti.

---

## 1. Goroutines

Naučili smo:

- šta predstavlja Goroutine,
- kako se pokreće,
- kako Go Scheduler izvršava Goroutines,
- zašto su Goroutines lagane (lightweight),
- zašto ne treba koristiti `time.Sleep()` za sinhronizaciju.

Najvažnija sintaksa:

```go
go myFunction()
```

---

## 2. Channels

Naučili smo:

- kako Goroutines međusobno komuniciraju,
- kako se Channel kreira,
- kako se šalju podaci,
- kako se primaju podaci.

Najvažnije operacije:

```go
ch <- value
```

```go
value := <-ch
```

---

## 3. Unbuffered Channels

Naučili smo:

- da nemaju interni bafer,
- da Sender i Receiver moraju biti spremni u isto vreme,
- da predstavljaju prirodnu sinhronizacionu tačku.

Kreiranje:

```go
make(chan int)
```

---

## 4. Buffered Channels

Naučili smo:

- da imaju interni bafer,
- da Sender ne mora odmah čekati Receiver-a,
- da Sender blokira tek kada je bafer pun.

Kreiranje:

```go
make(chan int, 5)
```

---

## 5. Channel Directions

Naučili smo:

```go
chan T
```

↓

Bidirectional

---

```go
chan<- T
```

↓

Send-only

---

```go
<-chan T
```

↓

Receive-only

---

## 6. Range over Channels

Naučili smo da:

```go
for value := range ch {

}
```

predstavlja idiomatski način čitanja svih vrednosti iz Channel-a.

---

## 7. close()

Naučili smo:

- šta znači zatvoren Channel,
- ko treba da ga zatvori,
- kako radi:

```go
value, ok := <-ch
```

- kada nastaje `panic`.

---

# 🗺️ Mapa znanja

```text
                 Goroutines
                      │
                      │
                      ▼
                 Channels
               /           \
              /             \
             ▼               ▼
     Unbuffered         Buffered
             │               │
             └──────┬────────┘
                    ▼
          Channel Directions
                    │
                    ▼
          Range over Channels
                    │
                    ▼
              close(Channel)
```

Svaka naredna oblast nadovezuje se na prethodnu.

---

# 🧩 Kako su sve oblasti povezane?

Najčešći tok izvršavanja izgleda ovako.

```text
Main

↓

pokrene Goroutines

↓

Goroutines komuniciraju

↓

Channel prenosi podatke

↓

Consumer koristi range

↓

Producer poziva close()

↓

Program završava
```

Ovaj obrazac ćeš koristiti veoma često.

---

# 📌 Kada koristiti koji Channel?

## Unbuffered

Koristi kada želiš:

- direktnu sinhronizaciju,
- potvrdu da je Receiver preuzeo podatak,
- jednostavnu koordinaciju između dve Goroutines.

---

## Buffered

Koristi kada:

- proizvođač radi brže,
- želiš manji broj blokiranja,
- praviš Queue,
- praviš Worker Pool,
- praviš Pipeline.

---

# 📌 Kada koristiti `range`?

Koristi ga kada:

- ne znaš broj vrednosti,
- želiš da pročitaš sve podatke,
- Producer zatvara Channel.

---

# 📌 Kada koristiti `value, ok := <-ch`?

Koristi ga kada:

- želiš da proveriš da li je Channel zatvoren,
- implementiraš sopstvenu logiku prijema,
- ne koristiš `range`.

---

# 📌 Kada koristiti Directional Channels?

Koristi ih:

- u bibliotekama,
- u javnim API-jima,
- u većim projektima,
- kada želiš jasnije odgovornosti funkcija.

---

# 🚨 Najčešće početničke greške

## ❌ Korišćenje `time.Sleep()` za sinhronizaciju

Loše:

```go
time.Sleep(time.Second)
```

Bolje:

- Channel
- `sync.WaitGroup`
- `context.Context`

---

## ❌ Zaboravljen `close()`

Ako koristiš:

```go
range
```

a niko ne zatvori Channel,

program može zauvek čekati.

---

## ❌ Receiver zatvara Channel

Najčešće pogrešno.

U najvećem broju slučajeva:

Producer zatvara Channel.

---

## ❌ Slanje u zatvoren Channel

Rezultat:

```text
panic
```

---

## ❌ Dvostruko zatvaranje

Rezultat:

```text
panic
```

---

## ❌ Mešanje Buffered i Unbuffered ponašanja

Zapamti.

Unbuffered:

```
Sender čeka Receiver-a.
```

Buffered:

```
Sender čeka samo kada je bafer pun.
```

---

# ✅ Pravila koja treba zapamtiti

## Pravilo 1

Goroutines komuniciraju preko Channel-a.

---

## Pravilo 2

Ne deli memoriju ako možeš da deliš komunikaciju.

Jedan od najpoznatijih Go slogana glasi:

> **"Do not communicate by sharing memory; instead, share memory by communicating."**

Njegova suština je da, kada god je moguće, razmena podataka između Goroutines treba da se odvija preko Channel-a umesto direktnim deljenjem promenljivih.

Napomena:

Ovo nije apsolutno pravilo. U mnogim situacijama `sync.Mutex`, `sync.RWMutex` ili `atomic` predstavljaju bolje rešenje. Njih ćemo detaljno obraditi u narednim modulima.

---

## Pravilo 3

Producer zatvara Channel.

---

## Pravilo 4

Consumer koristi `range`.

---

## Pravilo 5

Nemoj koristiti `time.Sleep()` kao mehanizam koordinacije.

---

## Pravilo 6

Ne zatvaraj Channel ako za to nema jasnog razloga.

---

## Pravilo 7

Biraj Buffered ili Unbuffered Channel prema problemu koji rešavaš, a ne prema ličnim preferencijama.

---

# ✅ Checklist znanja

Ako možeš samostalno da odgovoriš na sva pitanja ispod, spreman si za Modul #2.

Možeš li da objasniš:

- [ ] Šta je Goroutine?
- [ ] Kako se pokreće?
- [ ] Šta je Channel?
- [ ] Kako se šalje vrednost?
- [ ] Kako se prima vrednost?
- [ ] Razliku između Buffered i Unbuffered Channel-a?
- [ ] Kada Sender blokira?
- [ ] Kada Receiver blokira?
- [ ] Šta predstavlja `close()`?
- [ ] Šta predstavlja `range`?
- [ ] Šta predstavlja `ok`?
- [ ] Zašto postoje Directional Channels?
- [ ] Kada koristiti `chan<-`?
- [ ] Kada koristiti `<-chan`?

Ako je odgovor na većinu pitanja **DA**, možeš da nastaviš dalje.

---

# 🧪 Mini projekti

## Projekat #1 — Generator brojeva

Napravi program koji:

- pokreće jednu Goroutine,
- generiše brojeve od 1 do 100,
- šalje ih kroz Channel,
- koristi `range`,
- zatvara Channel.

---

## Projekat #2 — Generator parnih brojeva

Generiši:

```text
2

4

6

...

100
```

koristeći Goroutine i Channel.

---

## Projekat #3 — Generator neparnih brojeva

Generiši:

```text
1

3

5

...

99
```

---

## Projekat #4 — Kvadrati brojeva

Producer:

```
1..20
```

Consumer:

ispisuje kvadrate.

---

## Projekat #5 — Kubovi brojeva

Slično prethodnom,

ali računaj:

```
n³
```

---

# 💪 Završni zadaci

## 🟢 Lako

1. Napravi Producer koji šalje deset brojeva.
2. Consumer koristi `range`.
3. Zatvori Channel na ispravan način.

---

## 🟡 Srednje

4. Koristi Buffered Channel kapaciteta 5.
5. Uporedi ponašanje sa Unbuffered Channel-om.
6. Izmeri kako se menja `len(ch)` tokom izvršavanja.

---

## 🟠 Napredno

7. Napravi dva Producer-a koji šalju podatke na isti Channel.
8. Koordiniši njihovo završavanje tako da se Channel zatvori **tek kada oba Producer-a završe**.

> **Napomena:** U ovom trenutku još nismo obradili `sync.WaitGroup`, pa ovaj zadatak služi kao motivacija za naredni modul. Nakon što naučiš `WaitGroup`, vrati se i implementiraj ga na idiomatski način.

---

# 🎓 Šta sledi u Modulu #2?

Do sada smo naučili:

- kako pokrenuti Goroutines,
- kako razmenjivati podatke,
- kako završiti komunikaciju.

Sledeći korak je da naučimo kako da:

- koordinišemo više Goroutines,
- čekamo njihov završetak,
- obrađujemo više Channel-a istovremeno,
- pravimo prve ozbiljne concurrent arhitekture.

U Modulu #2 obrađivaćemo:

- `select`
- `default` u `select`
- `sync.WaitGroup`
- Worker Pools
- Pipelines
- Fan-out
- Fan-in

Ovo predstavlja prelaz sa osnovnih na stvarne production concurrency obrasce.

---

# 🏁 Završna poruka

Čestitamo!

Uspešno si završio **Modul #1**.

Sada razumeš temeljne koncepte Go Concurrency-ja:

- ✅ Goroutines
- ✅ Channels
- ✅ Unbuffered Channels
- ✅ Buffered Channels
- ✅ Channel Directions
- ✅ `range` nad Channel-om
- ✅ `close()` nad Channel-om

Ovo je osnova na kojoj počiva gotovo svaki ozbiljniji concurrent Go program.

U narednom modulu počećemo da povezujemo ove koncepte u složenije obrasce rada i naučićeš kako se piše konkurentan Go kod koji je čitljiv, bezbedan i skalabilan.
