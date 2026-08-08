# Pipelines

> **Modul:** #4 — Advanced Go Concurrency
>
> **Fajl:** `docs/module-4/06-pipelines.md`

---

# 📚 Sadržaj

- Šta je Pipeline
- Zašto koristiti Pipeline
- Pipeline vs Worker Pool
- Pipeline faze
- Prednosti Pipeline obrasca
- Production primeri

---

# Uvod

Mnogi problemi ne mogu da se svedu na:

```
Job

↓

Worker

↓

Result
```

Već imaju više koraka.

Na primer:

```
Read File

↓

Parse

↓

Validate

↓

Transform

↓

Store
```

Svaki korak predstavlja posebnu fazu obrade.

To nazivamo:

```
Pipeline
```

---

# Šta je Pipeline?

Pipeline je niz međusobno povezanih faza.

Svaka faza:

- prima podatke,
- obrađuje ih,
- prosleđuje sledećoj fazi.

Najčešće se faze povezuju pomoću:

```go
chan
```

---

# Vizuelni model

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

Svaka faza radi nezavisno.

---

# Primer iz stvarnog života

Zamisli fabriku.

```
Sirovina

↓

Sečenje

↓

Brušenje

↓

Farbanje

↓

Pakovanje
```

Nijedna stanica ne radi ceo posao.

Svaka obavlja samo svoju specifičnu fazu.

Pipeline u Go-u funkcioniše na isti način.

---

# Pipeline u Go-u

Tipična faza izgleda ovako:

```go
func stage(
	in <-chan Data,
) <-chan Data
```

Prima ulazni kanal.

Vraća izlazni kanal.

To omogućava jednostavno povezivanje faza.

---

# Vizuelni primer

```text
numbers

↓

square

↓

double

↓

print
```

Svaka funkcija obrađuje samo jedan korak.

---

# Zašto koristiti Pipeline?

Bez pipeline-a često dobijamo jednu veliku funkciju:

```text
Read

↓

Parse

↓

Validate

↓

Transform

↓

Store
```

Sve u jednom mestu.

Takav kod postaje:

- težak za testiranje,
- težak za održavanje,
- težak za proširivanje.

---

# Sa Pipeline-om

Svaka faza je mala.

Na primer:

```text
Reader
```

radi samo čitanje.

---

```text
Parser
```

radi samo parsiranje.

---

```text
Validator
```

radi samo validaciju.

---

```text
Writer
```

radi samo upis.

Odgovornosti su jasno razdvojene.

---

# Pipeline vs Worker Pool

## Worker Pool

Jedna faza.

```text
Jobs

↓

Workers

↓

Results
```

Cilj:

```
Ograničiti

konkurentnost.
```

---

## Pipeline

Više faza.

```text
Stage1

↓

Stage2

↓

Stage3
```

Cilj:

```
Organizovati

obradu.
```

Pipeline i Worker Pool nisu konkurenti — često se kombinuju.

---

# Pipeline + Worker Pool

Na primer:

```text
Read

↓

Worker Pool

↓

Validate

↓

Worker Pool

↓

Store
```

Svaka faza može imati sopstveni skup worker-a.

Ovo omogućava i modularnost i kontrolisanu konkurentnost.

---

# Production primer

Obrada CSV fajla:

```text
Read CSV

↓

Parse

↓

Validate

↓

Enrich

↓

Store DB
```

Svaka faza može biti nezavisno razvijana, testirana i optimizovana.

---

# Prednosti

✅ Jasna podela odgovornosti.

✅ Modularan dizajn.

✅ Jednostavnije testiranje.

✅ Jednostavno dodavanje novih faza.

✅ Prirodna konkurentnost između faza.

---

# Ograničenja

❌ Potrebno je pažljivo upravljati kanalima.

❌ Potrebno je pravilno propagirati greške.

❌ Otkazivanje mora biti koordinisano kroz ceo pipeline.

Ove teme ćemo obraditi u narednim delovima.

---

# Najčešće greške

## Greška #1

Staviti svu logiku u jednu fazu.

Tada pipeline gubi svoju najveću prednost — jasnu podelu odgovornosti.

---

## Greška #2

Deliti mutable stanje između faza.

Faze bi trebalo da komuniciraju preko kanala, a ne preko zajedničkih promenljivih.

---

## Greška #3

Ne zatvoriti izlazni kanal.

Sledeća faza može ostati blokirana čekajući podatke koji nikada neće stići.

---

# Best Practices

✅ Svaka faza treba da ima jednu odgovornost.

✅ Koristi usmerene kanale (`<-chan`, `chan<-`) u potpisima funkcija.

✅ Faza treba da zatvori **svoj izlazni kanal** kada završi sa slanjem podataka.

✅ Planiraj propagaciju grešaka i otkazivanje od samog početka dizajna.

---

# Mentalni model

Nemoj razmišljati:

```text
Jedna funkcija

↓

radi sve
```

Razmišljaj:

```text
Jedan problem

↓

više malih faza

↓

svaka radi

jednu stvar
```

To je suština Pipeline obrasca.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Pipeline

✅ kako se razlikuje od Worker Pool-a

✅ kako izgledaju faze pipeline-a

✅ zašto su kanali prirodan način povezivanja faza

✅ koje su glavne prednosti modularnog pristupa

---

# Pipelines — Implementacija jednostavnog Pipeline-a

---

# 📚 Sadržaj

- Arhitektura Pipeline-a
- Source faza
- Transform faza
- Sink faza
- Povezivanje faza
- Tok izvršavanja
- Zatvaranje kanala

---

# Arhitektura

Napravićemo jednostavan pipeline:

```text
Generator

↓

Square

↓

Double

↓

Printer
```

Svaka faza radi samo jedan posao.

---

# Vizuelni tok

```text
1

↓

1²

↓

2

↓

Print
```

---

```text
2

↓

4

↓

8

↓

Print
```

---

```text
3

↓

9

↓

18

↓

Print
```

Podaci prolaze kroz sve faze.

---

# Stage 1 — Generator

Generator proizvodi brojeve.

```go
func generator(
	values ...int,
) <-chan int {

	out := make(chan int)

	go func() {
		defer close(out)

		for _, v := range values {
			out <- v
		}
	}()

	return out
}
```

Generator:

- šalje podatke,
- zatvara kanal kada završi.

---

# Stage 2 — Square

```go
func square(
	in <-chan int,
) <-chan int {

	out := make(chan int)

	go func() {
		defer close(out)

		for v := range in {
			out <- v * v
		}
	}()

	return out
}
```

Ova faza:

```
prima

↓

obrađuje

↓

prosleđuje
```

---

# Stage 3 — Double

```go
func double(
	in <-chan int,
) <-chan int {

	out := make(chan int)

	go func() {
		defer close(out)

		for v := range in {
			out <- v * 2
		}
	}()

	return out
}
```

---

# Stage 4 — Sink

Poslednja faza ne vraća novi kanal.

Ona troši podatke.

```go
func printValues(
	in <-chan int,
) {
	for v := range in {
		fmt.Println(v)
	}
}
```

Ovo je:

```
Sink
```

---

# Povezivanje faza

Glavni program:

```go
numbers := generator(
	1,
	2,
	3,
	4,
)

squared := square(numbers)

doubled := double(squared)

printValues(doubled)
```

Vrlo čitljivo.

Svaka linija predstavlja jednu fazu.

---

# Vizuelno

```text
Generator

↓

Square

↓

Double

↓

Printer
```

---

# Tok izvršavanja

Korak 1

Generator šalje:

```
1
```

---

Korak 2

Square prima:

```
1
```

i šalje:

```
1
```

---

Korak 3

Double prima:

```
1
```

i šalje:

```
2
```

---

Korak 4

Printer ispisuje:

```
2
```

Isto se ponavlja za svaki sledeći element.

---

# Zatvaranje kanala

Generator:

```go
close(out)
```

↓

Square završava `range`

↓

zatvara svoj `out`

↓

Double završava `range`

↓

zatvara svoj `out`

↓

Printer završava `range`

Pipeline se prirodno gasi bez dodatnih signala.

---

# Zašto svaka faza zatvara svoj izlaz?

Važno pravilo:

```
Faza

koja šalje

↓

zatvara

svoj izlazni kanal
```

Ne zatvara ulazni kanal.

To ostaje odgovornost prethodne faze.

---

# Šta ako jedna faza ne zatvori kanal?

Na primer:

```go
func square(...) {

	...

	for v := range in {
		out <- v * v
	}

	// close(out) nedostaje
}
```

Double faza će zauvek čekati nove podatke.

To dovodi do:

```
Goroutine Leak
```

ili deadlock-a, u zavisnosti od ostatka programa.

---

# Production primer

Pipeline za obradu logova:

```text
Read Logs

↓

Parse

↓

Filter

↓

Aggregate

↓

Store
```

Svaka faza može biti razvijana i testirana nezavisno.

---

# Prednosti ovakvog dizajna

✅ Jednostavno dodavanje novih faza.

Na primer:

```text
Generator

↓

Square

↓

Filter

↓

Double

↓

Printer
```

Dodavanje nove faze ne zahteva izmene postojećih.

---

# Najčešće greške

## Greška #1

Ne zatvoriti izlazni kanal.

Sledeća faza ostaje blokirana.

---

## Greška #2

Jedna faza menja odgovornost druge faze.

Na primer:

```
Parser

↓

piše u bazu
```

To narušava modularnost.

---

## Greška #3

Direktno deljenje mutable stanja između faza.

Pipeline treba da razmenjuje podatke preko kanala.

---

# Best Practices

✅ Svaka faza neka ima jednu odgovornost.

✅ Koristi `defer close(out)` odmah nakon pokretanja Goroutine u fazi.

✅ Ulazni kanal tretiraj kao read-only (`<-chan`).

✅ Izlazni kanal vraćaj kao read-only (`<-chan`) kako bi korisnik mogao samo da čita iz njega.

---

# Mentalni model

Nemoj razmišljati:

```text
Funkcija

↓

radi sve
```

Razmišljaj:

```text
Stage

↓

Stage

↓

Stage

↓

Stage
```

Svaka faza radi mali, jasno definisan deo posla.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako implementirati jednostavan Pipeline

✅ kako povezati faze pomoću kanala

✅ zašto svaka faza zatvara svoj izlazni kanal

✅ kako podaci prolaze kroz ceo tok obrade

✅ kako prirodno dolazi do gašenja pipeline-a

---

# Pipelines — Fan-Out i Fan-In

---

# 📚 Sadržaj

- Šta je Fan-Out
- Šta je Fan-In
- Zašto se koriste
- Implementacija Fan-Out
- Implementacija Fan-In
- Paralelizacija pipeline-a
- Production primeri

---

# Uvod

Naš dosadašnji pipeline izgledao je ovako:

```text
Generator

↓

Square

↓

Double

↓

Printer
```

Svaka faza imala je:

```
1

Goroutine
```

Ali šta ako:

```
Square
```

postane najsporija faza?

---

# Rešenje

Umesto jednog worker-a:

```text
Square
```

koristimo više njih.

To se naziva:

```
Fan-Out
```

---

# Fan-Out

Jedan ulaz.

Više radnika.

```text
Jobs

↓

+---------+
|Worker 1 |
+---------+

+---------+
|Worker 2 |
+---------+

+---------+
|Worker 3 |
+---------+
```

Svi čitaju sa istog ulaznog kanala.

---

# Kako radi?

Pretpostavimo ulaz:

```
1

2

3

4

5

6
```

Worker-i mogu obraditi:

```text
Worker1

↓

1

↓

4
```

---

```text
Worker2

↓

2

↓

5
```

---

```text
Worker3

↓

3

↓

6
```

Tačan raspored nije garantovan i zavisi od scheduler-a i vremena izvršavanja.

---

# Primer

Pokretanje četiri worker-a:

```go
for i := 0; i < 4; i++ {
	go squareWorker(
		in,
		out,
	)
}
```

Svi:

```go
range in
```

čitaju sa istog kanala.

Svaki posao će preuzeti tačno jedan worker.

---

# Zašto ovo radi?

Go kanal garantuje da:

```
Jedna poslata vrednost

↓

bude primljena

tačno jednom.
```

Nema dupliranja poslova između worker-a.

---

# Fan-In

Sada imamo:

```
Worker1

↓

output1
```

---

```
Worker2

↓

output2
```

---

```
Worker3

↓

output3
```

Kako dobiti jedan izlaz?

Koristimo:

```
Fan-In
```

---

# Vizuelni model

```text
Output1

↓

Output2

↓

Output3

↓

Merged Output
```

Više kanala postaje jedan.

---

# Primer implementacije

```go
func merge(
	channels ...<-chan int,
) <-chan int
```

Ideja:

- pokrenuti po jednu Goroutine za svaki ulazni kanal,
- proslediti sve vrednosti u zajednički izlazni kanal,
- zatvoriti izlaz tek kada svi ulazni kanali završe.

Za koordinaciju se najčešće koristi `sync.WaitGroup`.

---

# Vizuelni tok

```text
Generator

↓

Fan-Out

↓

Worker1

↓

Worker2

↓

Worker3

↓

Fan-In

↓

Printer
```

---

# Zašto koristiti Fan-Out?

Ako je jedna faza:

- CPU zahtevna,
- I/O zahtevna,
- spora,

više worker-a može povećati propusnost (*throughput*).

---

# Da li su rezultati sortirani?

Ne.

Na primer:

Ulaz:

```
1

2

3
```

Izlaz može biti:

```
2

6

4
```

ili

```
6

2

4
```

ili bilo koji drugi redosled koji zavisi od paralelnog izvršavanja.

Ako je redosled važan, moraš ga eksplicitno očuvati (npr. dodavanjem indeksa i naknadnim sređivanjem rezultata).

---

# Production primer

Obrada slika.

```text
Read

↓

Fan-Out

↓

Resize

↓

Resize

↓

Resize

↓

Fan-In

↓

Upload
```

Resize je često najskuplja operacija.

Fan-Out omogućava da više slika bude obrađeno paralelno.

---

# Pipeline + Worker Pool

Veoma čest obrazac:

```text
Read

↓

Worker Pool

↓

Parse

↓

Worker Pool

↓

Validate

↓

Worker Pool

↓

Store
```

Svaka faza ima sopstveni broj worker-a.

To omogućava fino podešavanje propusnosti svake faze.

---

# Najčešće greške

## Greška #1

Više worker-a piše u isti kanal, a izlazni kanal se zatvara prerano.

Rezultat:

```text
panic:

send on closed channel
```

Izlazni kanal treba zatvoriti tek kada svi worker-i završe sa slanjem.

---

## Greška #2

Pretpostaviti da je redosled rezultata isti kao redosled ulaza.

To nije garantovano kod paralelne obrade.

---

## Greška #3

Pokrenuti previše worker-a.

Više worker-a ne znači automatski bolje performanse.

Broj worker-a treba odrediti merenjem.

---

# Best Practices

✅ Koristi Fan-Out kada jedna faza postane usko grlo.

✅ Koristi `sync.WaitGroup` za koordinaciju Fan-In faze.

✅ Ne oslanjaj se na redosled rezultata osim ako ga eksplicitno obezbeđuješ.

✅ Prilagodi broj worker-a prirodi workload-a (CPU-bound ili I/O-bound).

---

# Mentalni model

Nemoj razmišljati:

```text
Jedna faza

↓

Jedna Goroutine
```

Razmišljaj:

```text
Jedna faza

↓

Više worker-a

↓

Jedan izlaz
```

To je suština Fan-Out/Fan-In obrasca.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je Fan-Out

✅ šta je Fan-In

✅ kako paralelizovati jednu fazu pipeline-a

✅ kako spojiti više izlaznih kanala

✅ zašto redosled rezultata nije garantovan

✅ gde se Fan-Out/Fan-In koristi u produkcionim sistemima

---

# Pipelines — Error Handling i Cancellation

---

# 📚 Sadržaj

- Zašto je Error Handling važan
- Propagacija grešaka
- `context.Context`
- Cancellation kroz ceo pipeline
- Goroutine Leak
- Production obrasci

---

# Uvod

Zamisli pipeline:

```text
Read

↓

Parse

↓

Validate

↓

Store
```

Šta ako:

```
Validate

↓

pronađe grešku?
```

Da li:

```
Store

↓

treba da nastavi?
```

Najčešće:

```
NE
```

Treba zaustaviti ceo pipeline.

---

# Problem

Bez koordinacije:

```text
Stage 1

↓

Stage 2

↓

Stage 3
```

Ako:

```
Stage 2

↓

prekine
```

Stage 1 možda i dalje šalje podatke.

Stage 3 možda čeka podatke.

To može dovesti do:

- blokiranja,
- curenja Goroutines,
- nepotrebne obrade.

---

# Rešenje

Koristimo:

```go
context.Context
```

Sve faze dobijaju isti kontekst.

---

# Vizuelni model

```text
Context

↓

Stage 1

↓

Stage 2

↓

Stage 3
```

Jedan signal može zaustaviti ceo pipeline.

---

# Tipičan obrazac

```go
select {

case <-ctx.Done():
	return

case value, ok := <-in:

	...
}
```

Svaka faza treba da reaguje i na:

- dolazak podataka,
- otkazivanje.

---

# Propagacija greške

Pretpostavimo:

```text
Parser

↓

error
```

Tipičan sled događaja:

```text
Parser

↓

cancel()

↓

ctx.Done()

↓

Sve faze završavaju
```

Jedan poziv `cancel()` dovoljan je da obavesti sve faze koje prate isti kontekst.

---

# Zašto koristiti isti Context?

Ako svaka faza ima svoj nepovezan kontekst:

```text
Stage1

↓

Context1
```

```text
Stage2

↓

Context2
```

otkazivanje jedne faze neće automatski uticati na ostale.

Uobičajena praksa je da ceo pipeline deli jedan roditeljski kontekst ili hijerarhiju izvedenih konteksta.

---

# Šta raditi sa greškom?

Jedan čest pristup:

```go
type Result struct {
	Value Data
	Err   error
}
```

Pipeline prosleđuje i uspešne rezultate i informacije o greškama.

Drugi pristup je poseban kanal za greške.

Izbor zavisi od dizajna sistema i potrebe da li želiš da obrada stane na prvoj grešci ili da prikupi više grešaka.

---

# Primer toka

```text
Read

↓

Parse

↓

ERROR

↓

Cancel Context

↓

Validate Exit

↓

Store Exit
```

Pipeline se uredno zaustavlja.

---

# Goroutine Leak

Najčešći scenario:

```
Stage 3

↓

više ne čita
```

Ali:

```
Stage 2

↓

i dalje šalje
```

Operacija:

```go
out <- value
```

ostaje blokirana.

Goroutine nikada ne završava.

---

# Rešenje

Umesto:

```go
out <- value
```

koristi:

```go
select {

case out <- value:

case <-ctx.Done():
	return
}
```

Na taj način faza može da prekine slanje ako je pipeline otkazan.

---

# Zatvaranje kanala

Pravilo ostaje isto:

```
Sender

↓

zatvara

svoj izlazni kanal
```

Nikada:

```
Receiver

↓

close(...)
```

Primalac ne zna da li će pošiljalac poslati još podataka.

---

# Production tok

```text
Error

↓

cancel()

↓

ctx.Done()

↓

Sve faze završavaju

↓

Kanali se zatvaraju

↓

Pipeline Exit
```

Ovakav sled sprečava blokiranja i curenje resursa.

---

# Šta ako želimo da nastavimo obradu?

To zavisi od poslovnih zahteva.

Primer:

```text
1000 zapisa

↓

5 neispravnih
```

Moguće strategije:

- preskočiti neispravne zapise,
- evidentirati greške,
- nastaviti obradu ostalih.

Suprotno tome, kod kritičnih sistema (npr. finansijske transakcije) jedna greška može biti razlog za prekid celog pipeline-a.

---

# Najčešće greške

## Greška #1

Ignorisati:

```go
ctx.Done()
```

Faze nastavljaju rad i nakon što je sistem otkazan.

---

## Greška #2

Blokirati na:

```go
out <- value
```

bez mogućnosti prekida.

To može izazvati Goroutine leak.

---

## Greška #3

Svaka faza upravlja sopstvenim otkazivanjem bez koordinacije.

Rezultat može biti nekonzistentno stanje pipeline-a.

---

# Best Practices

✅ Prosledi isti `context.Context` kroz ceo pipeline.

✅ Koristi `select` za prijem i slanje kada postoji mogućnost otkazivanja.

✅ Definiši strategiju za obradu grešaka pre implementacije.

✅ Zatvaraj kanale samo na strani pošiljaoca.

---

# Mentalni model

Nemoj razmišljati:

```text
Greška

↓

Samo ova faza staje
```

Razmišljaj:

```text
Greška

↓

Signal

↓

Sve faze

↓

Kontrolisano gašenje
```

To je osnova pouzdanog pipeline-a.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ kako propagirati greške kroz pipeline

✅ zašto koristiti `context.Context`

✅ kako otkazati ceo pipeline

✅ kako sprečiti Goroutine leak pri slanju na kanal

✅ koje obrasce koristiti za produkcione pipeline-e

---

# Pipelines — Napredni obrasci i optimizacija

---

# 📚 Sadržaj

- Uska grla (Bottlenecks)
- Buffering između faza
- Batching
- Dinamičke faze
- Balansiranje propusnosti
- Monitoring pipeline-a
- Production preporuke

---

# Uvod

Pipeline je jak koliko i njegova najsporija faza.

Na primer:

```text
Read

↓

Parse

↓

Validate

↓

Store
```

Ako:

```
Store
```

radi deset puta sporije od ostalih faza,

ceo pipeline će biti ograničen tom fazom.

To nazivamo:

```
Bottleneck
```

---

# Bottleneck

Primer:

```text
Read

1000/s

↓

Parse

1000/s

↓

Store

100/s
```

Iako prve dve faze mogu da obrade 1000 zapisa u sekundi, ukupna propusnost pipeline-a biće približno 100 zapisa u sekundi.

Najsporija faza određuje maksimalni throughput.

---

# Buffering između faza

Najjednostavniji način da se ublaže kratkotrajne razlike u brzini faza je baferovani kanal.

Bez bafera:

```go
out := make(chan Item)
```

Sa baferom:

```go
out := make(chan Item, 100)
```

Bafer omogućava da brža faza kratkotrajno nastavi sa radom čak i ako je naredna faza trenutno zauzeta.

---

# Kada koristiti buffering?

Korisno kada:

- jedna faza povremeno kasni,
- postoje kratki skokovi opterećenja,
- želiš da smanjiš nepotrebno blokiranje između faza.

Međutim, bafer nije zamena za rešavanje trajnog uskog grla.

---

# Opasnost velikih bafera

Veliki bafer može:

- povećati potrošnju memorije,
- sakriti da naredna faza ne može da prati tempo,
- povećati vreme čekanja pojedinačnih elemenata.

Bafer ublažava problem, ali ga ne rešava.

---

# Batching

Umesto:

```text
1 zapis

↓

1 upis u bazu
```

možeš obrađivati:

```text
100 zapisa

↓

1 upis u bazu
```

To se naziva:

```
Batching
```

---

# Vizuelni primer

Bez batching-a:

```text
Item

↓

Store

↓

Item

↓

Store
```

---

Sa batching-om:

```text
100 Items

↓

Store Batch
```

Ovo često značajno smanjuje troškove komunikacije sa bazom ili spoljnim servisom.

---

# Kada koristiti batching?

Pogodno za:

- INSERT operacije u bazu,
- slanje poruka broker-u,
- zapisivanje logova,
- obradu velikog broja fajlova.

Manje je pogodno kada je svaki pojedinačni rezultat hitno potreban.

---

# Dinamičke faze

Neke pipeline implementacije omogućavaju uključivanje ili isključivanje faza.

Na primer:

```text
Read

↓

Parse

↓

Compression (optional)

↓

Encrypt (optional)

↓

Store
```

Time se isti pipeline može prilagoditi različitim scenarijima bez menjanja ostatka sistema.

---

# Balansiranje propusnosti

Ako jedna faza postane usko grlo, moguće strategije su:

1. Optimizovati samu fazu.
2. Dodati Fan-Out (više worker-a).
3. Uvesti batching.
4. Smanjiti nepotreban rad.
5. Podesiti veličinu bafera.

Važno je prvo izmeriti gde nastaje problem.

---

# Monitoring

Bez merenja teško je optimizovati pipeline.

Korisne metrike uključuju:

- throughput (obrađeni elementi u sekundi),
- latenciju po fazi,
- broj aktivnih worker-a,
- dužinu redova između faza,
- broj grešaka,
- iskorišćenost CPU-a i memorije.

Ove metrike pomažu da otkriješ koja faza ograničava ceo sistem.

---

# Production primer

Obrada događaja:

```text
Kafka

↓

Parse

↓

Validate

↓

Batch

↓

Database
```

Ako baza sporije prihvata upise:

- povećava se red ispred faze `Batch`,
- monitoring pokazuje rast latencije,
- možeš povećati broj worker-a ili prilagoditi veličinu batch-a.

---

# Pipeline nije beskonačno skalabilan

Dodavanje više worker-a svakoj fazi neće uvek povećati propusnost.

Razlozi mogu biti:

- ograničenje baze,
- mrežna latencija,
- disk,
- CPU,
- spoljne API kvote.

Optimizacija mora uzeti u obzir ceo sistem.

---

# Najčešće greške

## Greška #1

Dodavati bafer na svaki kanal bez razumevanja zašto.

Bafer treba da rešava konkretan problem, ne da bude podrazumevana postavka.

---

## Greška #2

Preveliki batch.

To može povećati latenciju i potrošnju memorije.

---

## Greška #3

Optimizovati pogrešnu fazu.

Bez merenja lako je trošiti vreme na deo sistema koji nije usko grlo.

---

# Best Practices

✅ Identifikuj bottleneck pre optimizacije.

✅ Koristi buffering umereno i sa jasnim razlogom.

✅ Razmotri batching kada je cena pojedinačne operacije velika.

✅ Kombinuj Pipeline, Fan-Out i Worker Pool po potrebi.

✅ Prati metrike i prilagođavaj sistem na osnovu stvarnih podataka.

---

# Mentalni model

Nemoj razmišljati:

```text
Brža faza

↓

Brži sistem
```

Razmišljaj:

```text
Najsporija faza

↓

Određuje

propusnost

celog pipeline-a
```

To je osnovni princip optimizacije pipeline-a.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je bottleneck

✅ kako buffering utiče na pipeline

✅ kada koristiti batching

✅ kako balansirati propusnost između faza

✅ koje metrike pratiti pri optimizaciji

---

# Pipelines — Production Guidelines i završni rezime

---

# 📚 Sadržaj

- Kada koristiti Pipeline
- Pipeline vs Worker Pool
- Pipeline vs Fan-Out/Fan-In
- Production checklist
- Najčešće greške
- Senior perspektiva
- Završni rezime

---

# Kada koristiti Pipeline?

Pipeline je dobar izbor kada:

- obrada ima više jasno definisanih koraka,
- svaki korak ima svoju odgovornost,
- faze mogu raditi nezavisno,
- želiš modularnu arhitekturu.

Primer:

```text
Input

↓

Decode

↓

Validate

↓

Transform

↓

Persist
```

---

# Kada NE koristiti Pipeline?

Nemoj uvoditi pipeline ako:

- postoji samo jedan jednostavan korak,
- nema potrebe za paralelizmom,
- dodatna kompleksnost ne donosi vrednost.

Primer:

```go
result := process(data)
```

je često bolji od:

```text
Stage1

↓

Stage2

↓

Stage3
```

za trivijalne slučajeve.

---

# Pipeline vs Worker Pool

## Worker Pool

Problem:

> Kako ograničiti broj paralelnih izvršavanja?

Model:

```text
Jobs

↓

Workers

↓

Results
```

Koristi se kada imaš mnogo nezavisnih poslova.

---

## Pipeline

Problem:

> Kako organizovati višekoračnu obradu?

Model:

```text
Stage1

↓

Stage2

↓

Stage3
```

Koristi se kada obrada ima više faza.

---

# Pipeline + Worker Pool

U realnim sistemima često se kombinuju.

Primer:

```text
Read

↓

Parser Pool

↓

Validator Pool

↓

Storage Pool
```

Svaka faza ima svoj nivo konkurentnosti.

---

# Pipeline vs Fan-Out/Fan-In

Fan-Out/Fan-In nije alternativa Pipeline-u.

On je često deo Pipeline-a.

Primer:

```text
Read

↓

Fan-Out

↓

Worker1
Worker2
Worker3

↓

Fan-In

↓

Store
```

Pipeline opisuje tok.

Fan-Out/Fan-In opisuje skaliranje jedne faze.

---

# Production Checklist

Pre produkcione upotrebe proveri:

## Dizajn

✅ Svaka faza ima jednu odgovornost.

✅ Kanali imaju jasno definisan smer komunikacije.

✅ Nema nepotrebnog deljenja memorije.

---

## Cancellation

✅ Sve faze primaju `context.Context`.

✅ Sve blokirajuće operacije mogu biti prekinute.

✅ Greška jedne faze može zaustaviti ostatak sistema kada je potrebno.

---

## Channel Management

✅ Sender zatvara kanal.

✅ Receiver ne zatvara kanal koji ne poseduje.

✅ Nema mogućnosti slanja na zatvoren kanal.

---

## Performance

✅ Bottleneck je identifikovan merenjem.

✅ Broj worker-a je podešen prema workload-u.

✅ Buffer veličine su opravdane.

---

## Reliability

✅ Postoji strategija za greške.

✅ Postoji monitoring.

✅ Postoji kontrola backpressure-a.

---

# Najčešće greške Junior programera

## 1. Previše faza

Loše:

```text
Validate Name

↓

Validate Age

↓

Validate Email

↓

Validate Address
```

Ako faze ne donose vrednost, pipeline postaje nepotrebno komplikovan.

---

## 2. Deljeno stanje

Loše:

```go
var counter int

// više faza menja counter
```

Bolje:

```text
Stage

↓

Channel

↓

Stage
```

---

## 3. Ignorisanje gašenja

Pipeline koji radi samo u idealnim uslovima nije production-ready.

---

# Najčešće greške Medior programera

## 1. Paralelizacija bez merenja

Dodavanje worker-a bez razumevanja bottleneck-a često ne pomaže.

---

## 2. Neograničeni kanali

Veliki broj podataka može dovesti do problema sa memorijom.

---

## 3. Loš error handling

Ako greška jedne faze ne utiče na ostatak sistema, može doći do nekonzistentnih rezultata.

---

# Kako razmišlja Senior Go programer?

Ne pita:

> "Kako da napravim više Goroutines?"

Pita:

- Koje su faze sistema?
- Gde je usko grlo?
- Koji deo treba paralelizovati?
- Kako sistem reaguje na grešku?
- Kako se gasi?
- Šta se dešava kada dolazni tok premaši kapacitet?

---

# Mentalni model

Nemoj razmišljati:

```text
Concurrency

↓

Više brzine
```

Razmišljaj:

```text
Dobro dizajnirana konkurentnost

↓

Predvidiv sistem

↓

Stabilna aplikacija
```

---

# Ceo put kroz Modul #4.6

```text
Pipeline Basics

↓

Stage Design

↓

Fan-Out / Fan-In

↓

Error Handling

↓

Cancellation

↓

Optimization

↓

Production Design
```

---

# Šta nosiš iz ovog modula?

Treba da možeš da objasniš:

- šta je Pipeline pattern,
- kako dizajnirati faze,
- kako povezati faze kanalima,
- kako koristiti Fan-Out/Fan-In,
- kako propagirati greške,
- kako otkazati ceo pipeline,
- kako optimizovati throughput,
- kada koristiti Pipeline u odnosu na Worker Pool.

---

# 📋 Rezime Modula #4.6

U ovom modulu naučili smo:

✅ šta je Pipeline obrazac

✅ kako implementirati pipeline u Go-u

✅ kako povezati faze pomoću kanala

✅ kako koristiti Fan-Out/Fan-In

✅ kako rešiti error handling

✅ kako koristiti `context.Context` za cancellation

✅ kako optimizovati pipeline

✅ kako projektovati production-ready pipeline sisteme

---

### ➡️ Sledeća lekcija **[**Semaphore Pattern**](07-semaphore-pattern.md)**

Obuhvatiće:

- šta je Semaphore,
- razlika između Worker Pool i Semaphore,
- implementacija pomoću kanala,
- ograničavanje pristupa resursima,
- rate limiting obrasci,
- production primeri.
