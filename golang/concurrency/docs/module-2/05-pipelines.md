# Pipelines (Deo 1)

> **Modul:** #2 — Lako → Srednje
>
> **Lekcija:** 5/8
> **Fajl:** `docs/module-2/05-pipelines.md`

> **Napomena:** Ovaj fajl predstavlja prvi deo lekcije o Pipeline obrascu. Drugi deo sadrži naprednije primere, zatvaranje Channel-a, najčešće greške i Best Practices.

---

# 📚 Sadržaj (Deo 1)

- Šta je Pipeline?
- Zašto postoji?
- Problem koji rešava
- Šta predstavlja Stage?
- Tok podataka kroz Pipeline
- Jednostepeni Pipeline
- Višestepeni Pipeline
- Producer → Stage → Stage → Consumer
- Prednosti Pipeline arhitekture
- Analiza izvršavanja
- Zatvaranje Channel-a u Pipeline-u
- Pipeline sa više Goroutines po Stage-u
- Pipeline i `sync.WaitGroup`
- Backpressure
- Fan-out + Pipeline
- Fan-in + Pipeline
- Najčešće greške
- Best Practices
- Rezime
- Pitanja
- Praktični zadaci

---

# 🎯 Cilj lekcije

Nakon ove lekcije znaćeš:

- šta predstavlja Pipeline obrazac,
- zašto je jedan od najčešće korišćenih obrazaca u Go-u,
- kako se više Goroutines povezuje pomoću Channel-a,
- šta predstavlja *Stage*,
- kako podaci prolaze kroz ceo Pipeline.

---

# Uvod

U prethodnoj lekciji upoznali smo **Worker Pool**.

Worker Pool odgovara na pitanje:

> Kako ograničiti broj Goroutines koje izvršavaju veliki broj poslova?

Pipeline rešava potpuno drugačiji problem.

On odgovara na pitanje:

> Kako jedan podatak prolazi kroz više koraka obrade?

---

# Problem

Pretpostavimo da imamo brojeve:

```
1
2
3
4
5
```

Nad svakim brojem želimo da izvršimo tri operacije.

Korak 1

```
×2
```

Korak 2

```
+10
```

Korak 3

```
ispis
```

Možemo sve napisati u jednoj funkciji.

Ali...

šta ako sutra dodamo još:

- filtriranje,
- validaciju,
- čitanje iz baze,
- slanje preko mreže,
- kompresiju,
- enkripciju?

Kod brzo postaje težak za održavanje.

---

# Rešenje

Podelićemo obradu na više nezavisnih koraka.

Svaki korak radi samo jednu stvar.

To nazivamo:

> **Stage**

---

# Šta je Stage?

Stage predstavlja jednu fazu obrade.

Prima podatke.

↓

Obrađuje ih.

↓

Prosleđuje ih dalje.

---

# Vizuelni prikaz

```text
Input

↓

Stage

↓

Output
```

Svaki Stage:

- ima ulaz,
- radi jednu operaciju,
- ima izlaz.

---

# Pipeline

Pipeline predstavlja više Stage-ova povezanih Channel-ima.

```text
Input

↓

Stage 1

↓

Stage 2

↓

Stage 3

↓

Output
```

---

# Zašto se zove Pipeline?

Zamisli cev za vodu.

```text
Voda

↓

Cev

↓

Izlaz
```

Podatak putuje kroz "cev".

Usput prolazi kroz različite faze.

Isto važi i za Pipeline.

---

# Vizuelna analogija

Fabrika.

```text
Sirovina

↓

Sečenje

↓

Farbanje

↓

Pakovanje

↓

Gotov proizvod
```

Svaka mašina radi samo svoj deo posla.

---

# Analogija sa restoranom

```text
Narudžbina

↓

Kuvar

↓

Konobar

↓

Gost
```

Svako ima svoju odgovornost.

---

# Pipeline u Go-u

U Go-u svaki Stage najčešće predstavlja:

- jednu Goroutine,
- jedan ulazni Channel,
- jedan izlazni Channel.

---

# Vizuelni prikaz

```text
Channel

↓

Goroutine

↓

Channel

↓

Goroutine

↓

Channel

↓

Main
```

---

# Prvi Pipeline

Počećemo veoma jednostavno.

Generator proizvodi brojeve.

```text
1

2

3

4

5
```

Drugi Stage ih duplira.

Treći ih ispisuje.

---

# Stage 1 — Generator

```go
func generator(nums ...int) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for _, n := range nums {

			out <- n

		}

	}()

	return out

}
```

---

# Šta radi Generator?

Prima:

```text
1

2

3
```

Vraća Channel.

Kroz njega šalje brojeve.

---

# Vizuelni prikaz

```text
1

2

3

↓

Channel
```

---

# Stage 2 — Dupliranje

```go
func multiplyByTwo(in <-chan int) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for n := range in {

			out <- n * 2

		}

	}()

	return out

}
```

---

# Šta radi ovaj Stage?

Prima:

```
1
```

Šalje:

```
2
```

Prima:

```
5
```

Šalje:

```
10
```

---

# Vizuelni prikaz

```text
1

↓

2

----------

5

↓

10
```

---

# Main

```go
func main() {

	numbers := generator(1, 2, 3, 4, 5)

	doubled := multiplyByTwo(numbers)

	for value := range doubled {

		fmt.Println(value)

	}

}
```

---

# Izlaz

```text
2
4
6
8
10
```

---

# Analiza

Korak 1

Generator šalje:

```
1
```

↓

Stage 2 prima.

↓

Množi sa dva.

↓

Šalje dalje.

↓

Main ispisuje.

---

Korak 2

Generator šalje:

```
2
```

↓

isti proces.

---

Korak 3

Sve se ponavlja.

---

# Dodavanje novog Stage-a

Dodajmo:

```
+10
```

---

```go
func addTen(in <-chan int) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for n := range in {

			out <- n + 10

		}

	}()

	return out

}
```

---

# Novi Pipeline

```go
func main() {

	stage1 := generator(1, 2, 3, 4)

	stage2 := multiplyByTwo(stage1)

	stage3 := addTen(stage2)

	for value := range stage3 {

		fmt.Println(value)

	}

}
```

---

# Vizuelni prikaz

```text
Generator

↓

1

2

3

↓

Multiply

↓

2

4

6

↓

Add

↓

12

14

16

↓

Main
```

---

# Šta smo dobili?

Svaki Stage radi samo jednu stvar.

Generator:

```
Generiše podatke.
```

Multiply:

```
Duplira brojeve.
```

Add:

```
Dodaje 10.
```

Main:

```
Koristi rezultat.
```

Odgovornosti su jasno razdvojene.

---

# Prednosti Pipeline arhitekture

## Jednostavnost

Svaka funkcija radi jednu stvar.

---

## Modularnost

Možemo menjati jedan Stage bez izmene ostalih.

---

## Ponovna upotreba

`multiplyByTwo()`

može se koristiti u više različitih Pipeline-ova.

---

## Lakše testiranje

Svaki Stage može imati sopstvene testove.

---

## Bolja organizacija koda

Veliki problemi dele se na manje celine.

---

# Životni ciklus Pipeline-a

```text
Generator

↓

Channel

↓

Stage

↓

Channel

↓

Stage

↓

Channel

↓

Main
```

---

# Pipeline i Worker Pool

Često se mešaju.

Ali nisu isto.

Worker Pool:

```
Jedan posao

↓

više Worker-a
```

Pipeline:

```
Jedan podatak

↓

više faza obrade
```

Jedan povećava propusnost izvršavanja poslova.

Drugi organizuje tok obrade podataka.

U praksi se često kombinuju.

---

# 🚨 Najčešće greške

## 1. Mešanje odgovornosti

Loše:

Jedna funkcija radi sve.

Bolje:

Jedan Stage = jedan posao.

---

## 2. Stage menja više različitih stvari

Pipeline treba da bude modularan.

Ako jedan Stage:

- validira,
- računa,
- zapisuje u bazu,
- šalje e-mail,

verovatno radi previše.

---

## 3. Zaboravljanje zatvaranja izlaznog Channel-a

Svaki Stage koji kreira izlazni Channel treba da ga zatvori kada završi slanje.

U suprotnom, naredni Stage može zauvek čekati nove podatke.

Detaljnije ćemo to analizirati u drugom delu lekcije.

---

# ✅ Best Practices

- Jedan Stage treba da ima jednu odgovornost.
- Stage treba da prima ulazni i vraća izlazni Channel.
- Svaki Stage treba da bude što jednostavniji.
- Razmišljaj o Pipeline-u kao o nizu malih transformacija.
- Piši Stage-ove tako da budu lako ponovo upotrebljivi.

---

# 📋 Rezime

- Pipeline predstavlja niz Stage-ova povezanih Channel-ima.
- Svaki Stage prima podatke, obrađuje ih i prosleđuje dalje.
- Jedna Goroutine najčešće predstavlja jedan Stage.
- Pipeline omogućava modularan, čitljiv i lako proširiv kod.
- Stage-ovi imaju jasno definisane odgovornosti.

---

# ❓ Pitanja za proveru znanja

1. Šta predstavlja Pipeline?
2. Šta predstavlja Stage?
3. Zašto je korisno podeliti obradu na više Stage-ova?
4. Kako Stage međusobno komuniciraju?
5. Zašto svaki Stage obično ima jedan ulazni i jedan izlazni Channel?
6. Po čemu se Pipeline razlikuje od Worker Pool-a?
7. Koje su glavne prednosti Pipeline arhitekture?
8. Zašto je modularnost važna kod Pipeline-a?
9. Zašto svaki Stage treba da ima jednu odgovornost?
10. Zašto je zatvaranje izlaznog Channel-a važno?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Napravi Pipeline sa dva Stage-a:
   - Generator
   - Množenje sa 3

2. Dodaj treći Stage koji svakom broju dodaje 100.

3. Promeni Generator tako da generiše brojeve od 1 do 20.

---

## 🟡 Srednje

4. Napravi Pipeline koji:
   - generiše brojeve,
   - računa njihov kvadrat,
   - zatim dodaje 5,
   - na kraju ispisuje rezultat.

5. Dodaj novi Stage koji filtrira samo parne brojeve.

6. Napravi Stage koji pretvara `int` u `string`.

---

## 🟠 Izazov

7. Napravi Pipeline koji simulira obradu narudžbina:

- Generator proizvodi ID narudžbina.
- Drugi Stage proverava da li je narudžbina validna.
- Treći Stage računa cenu.
- Četvrti Stage priprema tekstualni izveštaj.
- Main ispisuje završne rezultate.

Pokušaj da svaka faza bude potpuno nezavisna od ostalih.

---

# Podsećanje

Pipeline koji smo napravili izgleda ovako:

```text
Generator

↓

Stage 1

↓

Stage 2

↓

Main
```

Svaki Stage:

- prima podatke,
- obrađuje ih,
- prosleđuje dalje.

Međutim, ostaje nekoliko važnih pitanja:

- Ko zatvara Channel?
- Šta ako imamo više Worker-a u jednom Stage-u?
- Šta ako je jedan Stage sporiji od ostalih?

---

# Ko zatvara Channel?

Jedno od najvažnijih pravila u Go Concurrency svetu glasi:

> **Channel zatvara Goroutine koja šalje podatke.**

Ne Goroutine koja prima.

---

# Primer

Generator:

```go
func generator(nums ...int) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for _, n := range nums {

			out <- n
		}

	}()

	return out
}
```

Generator je jedini koji šalje podatke na:

```go
out
```

Zato upravo on zatvara Channel.

---

# Stage

Isto važi i za svaki Stage.

```go
func square(in <-chan int) <-chan int {

	out := make(chan int)

	go func() {

		defer close(out)

		for n := range in {

			out <- n * n

		}

	}()

	return out

}
```

Stage šalje podatke na:

```go
out
```

Zbog toga upravo on poziva:

```go
close(out)
```

---

# Zašto je ovo važno?

Posmatraj:

```text
Generator

↓

Stage A

↓

Stage B

↓

Main
```

Ako Generator ne zatvori svoj izlazni Channel,

Stage A nikada neće izaći iz:

```go
for value := range in
```

A ako Stage A ne završi,

ni Stage B neće završiti.

Na kraju,

ceo Pipeline ostaje blokiran.

---

# Vizuelni prikaz

```text
Generator

↓

Channel nije zatvoren

↓

Stage A čeka

↓

Stage B čeka

↓

Main čeka

↓

Deadlock
```

---

# Pipeline sa više Worker-a

Do sada je svaki Stage imao jednu Goroutine.

Ali može imati više njih.

Na primer:

```text
Generator

↓

Stage

↓

4 Worker-a

↓

Output
```

Svi Worker-i čitaju sa istog ulaznog Channel-a.

---

# Primer

```go
for i := 0; i < 4; i++ {

	go worker(in, out)

}
```

Sada četiri Goroutines paralelno obrađuju isti Stage.

---

# Novi problem

Ko zatvara:

```go
out
```

?

Ne sme nijedan Worker.

Zašto?

Jer ostali možda još uvek šalju podatke.

---

# Rešenje

Koristi se:

```go
sync.WaitGroup
```

---

# Primer

```go
var wg sync.WaitGroup

for i := 0; i < workers; i++ {

	wg.Add(1)

	go func() {

		defer wg.Done()

		worker(in, out)

	}()

}

go func() {

	wg.Wait()

	close(out)

}()
```

---

# Analiza

Korak 1

Pokreću se Worker-i.

↓

Korak 2

Svi čitaju sa:

```go
in
```

↓

Korak 3

Šalju rezultate na:

```go
out
```

↓

Korak 4

Svaki poziva:

```go
Done()
```

↓

Korak 5

`Wait()`

čeka sve Worker-e.

↓

Korak 6

Tek tada:

```go
close(out)
```

---

# Backpressure

Jedan od najvažnijih pojmova kod Pipeline-a.

---

# Šta je Backpressure?

Pretpostavimo:

```text
Generator

↓

1000 podataka/s
```

Ali sledeći Stage može da obradi:

```text
100 podataka/s
```

Šta će se dogoditi?

Generator mora da uspori.

---

# Vizuelni prikaz

```text
1000/s

↓

Generator

↓

Stage

↓

100/s
```

Brži deo sistema automatski usporava zbog sporijeg dela.

To nazivamo:

> **Backpressure**

---

# Zašto je to dobro?

Da Channel-i nisu blokirajući,

mogli bismo da proizvedemo:

```
milione
```

neobrađenih podataka.

Blokirajuće ponašanje Channel-a prirodno reguliše protok.

---

# Fan-out + Pipeline

Pipeline se lako kombinuje sa Fan-out obrascem.

```text
Generator

↓

Jobs

↓

Worker 1

Worker 2

Worker 3

↓

Output
```

Jedan Stage može imati više Worker-a.

---

# Fan-in + Pipeline

Rezultati svih Worker-a mogu se spojiti.

```text
Worker 1

↓

Worker 2

↓

Worker 3

↓

Merge

↓

Output
```

Ovaj obrazac ćemo detaljno obraditi u narednim lekcijama.

---

# Kada koristiti Pipeline?

Koristi ga kada obrada prirodno može da se podeli na faze.

Na primer:

```text
Čitanje fajla

↓

Parsiranje

↓

Validacija

↓

Obrada

↓

Čuvanje
```

---

# Primeri iz prakse

Pipeline se često koristi za:

- obradu logova,
- ETL procese (*Extract, Transform, Load*),
- obradu CSV fajlova,
- obradu slika,
- video obradu,
- obradu mrežnih paketa,
- streaming podataka,
- obradu događaja (*event processing*).

---

# Kada Pipeline nije potreban?

Ako imaš:

- jednu jednostavnu operaciju,
- vrlo mali broj podataka,
- kod koji bi zbog Pipeline-a postao nepotrebno složen.

Nemoj uvoditi Pipeline samo zato što postoji.

---

# Najčešće greške

## ❌ Stage ne zatvara izlazni Channel

Rezultat:

sledeći Stage zauvek čeka.

---

## ❌ Receiver zatvara Channel

Pogrešno.

Channel zatvara Sender.

---

## ❌ Jedan Worker zatvara zajednički Channel

Rezultat:

```text
panic:

send on closed channel
```

---

## ❌ Spajanje više odgovornosti u jedan Stage

Loše:

```text
Validacija

↓

Računanje

↓

Slanje e-mail-a

↓

Upis u bazu
```

u istoj funkciji.

---

## ❌ Nepotrebno veliki broj Stage-ova

Pipeline treba da bude čitljiv.

Ako svaki Stage radi jednu liniju koda,

verovatno si preterao.

---

# Performance razmatranja

Pipeline nije besplatan.

Svaki Stage obično uvodi:

- novu Goroutine,
- novi Channel,
- novu sinhronizaciju.

Ako je obrada veoma jednostavna,

Pipeline može biti sporiji od sekvencijalnog rešenja.

Uvek meri performanse pre optimizacije.

---

# Pipeline + Worker Pool

Veoma čest production obrazac.

```text
Generator

↓

Worker Pool

↓

Filter

↓

Worker Pool

↓

Writer
```

Različiti Stage-ovi mogu imati različit broj Worker-a.

Na primer:

```text
Stage 1

↓

2 Worker-a

↓

Stage 2

↓

8 Worker-a

↓

Stage 3

↓

4 Worker-a
```

Svaki Stage može biti optimizovan nezavisno.

---

# Best Practices

- Jedan Stage = jedna odgovornost.
- Sender zatvara Channel.
- Koristi `range` za čitanje.
- Koristi `WaitGroup` kada više Goroutines šalje na isti izlazni Channel.
- Kombinuj Pipeline i Worker Pool kada problem to zahteva.
- Nemoj praviti Pipeline bez jasnog razloga.
- Testiraj svaki Stage zasebno.

---

# 📋 Rezime

- Pipeline deli obradu na više nezavisnih Stage-ova.
- Svaki Stage prima podatke, obrađuje ih i prosleđuje dalje.
- Sender zatvara Channel koji koristi za slanje.
- Kada više Worker-a šalje na isti Channel, njegovo zatvaranje treba da bude koordinisano pomoću `sync.WaitGroup`.
- Backpressure predstavlja prirodno usporavanje bržeg Stage-a zbog sporijeg narednog Stage-a.
- Pipeline se često kombinuje sa Fan-out i Fan-in obrascima.

---

# ❓ Pitanja za proveru znanja

1. Ko zatvara Channel u Pipeline-u?
2. Zašto Receiver ne treba da zatvara Channel?
3. Šta je Backpressure?
4. Zašto je Backpressure koristan?
5. Kako `WaitGroup` pomaže kod više Worker-a u jednom Stage-u?
6. Kada Pipeline može biti sporiji od običnog sekvencijalnog koda?
7. Kako se Pipeline kombinuje sa Worker Pool obrascem?
8. Kada Pipeline nije dobro rešenje?
9. Zašto svaki Stage treba da ima jednu odgovornost?
10. Zašto je važno testirati Stage-ove odvojeno?

---

# 📝 Praktični zadaci

## 🟢 Lako

1. Dodaj treći Stage postojećem Pipeline-u koji množi svaki broj sa 5.
2. Napravi Generator koji proizvodi brojeve od 1 do 100.
3. Dodaj Stage koji filtrira samo neparne brojeve.

---

## 🟡 Srednje

4. Implementiraj Pipeline sa tri Stage-a, pri čemu drugi Stage koristi četiri Worker-a.
5. Koristi `sync.WaitGroup` za pravilno zatvaranje izlaznog Channel-a.
6. Simuliraj sporiji Stage pomoću `time.Sleep()` i posmatraj efekat Backpressure-a.

---

## 🟠 Izazov

7. Napravi kompletan Pipeline za obradu fajlova:

- Generator šalje putanje do fajlova.
- Prvi Stage "učitava" sadržaj (simulacija).
- Drugi Stage broji broj reči.
- Treći Stage filtrira fajlove sa manje od 100 reči.
- Četvrti Stage priprema izveštaj.
- Na kraju ispiši zbir svih obrađenih fajlova i broj odbačenih fajlova.

Organizuj svaku fazu kao zaseban Stage sa jasno definisanim ulaznim i izlaznim Channel-ima.

---

### ➡️ Sledeća lekcija **[**Fan-out**](06-fan-out.md)**

detaljno ćemo obraditi **Fan-out** obrazac, naučiti kako više Goroutines paralelno obrađuje podatke sa istog Channel-a, kada je ovaj obrazac efikasniji od običnog Pipeline-a i kako se prirodno kombinuje sa Worker Pool-ovima.
