# Concurrency Performance Tuning

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Concurrency Performance Tuning
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Šta znači Performance Tuning?
- Throughput vs Latency
- Skalabilnost konkurentnih sistema
- CPU-bound vs I/O-bound poslovi
- Paralelizam nije uvek brži
- Kada konkurentnost usporava program?
- Performance Mental Model

---

# 1. Šta Je Concurrency Performance Tuning?

Pisanje konkurentnog programa nije isto što i pisanje brzog programa.

---

Moguće je napisati aplikaciju koja koristi:

- hiljade goroutine-a
- desetine channel-a
- više Worker Pool-ova

---

...a da bude sporija od jednostavne sekvencijalne implementacije.

---

Performance tuning predstavlja proces:

```
merenja

↓

analize

↓

optimizacije
```

konkurentnog programa.

---

# 2. Prvo Pravilo Performance-a

Najvažnije pravilo glasi:

> **Nemoj optimizovati ono što nisi izmerio.**

---

Nikada nemoj pretpostaviti da je problem:

- goroutine
- channel
- mutex
- GC
- scheduler

---

Prvo:

```
meri

↓

analiziraj

↓

tek onda optimizuj
```

---

# 3. Throughput

Throughput odgovara na pitanje:

```
Koliko posla

sistem obradi

u jedinici vremena?
```

---

Primer:

```
1000 HTTP zahteva

u sekundi
```

ili

```
50 000 poruka

u minuti
```

---

Veći throughput znači:

```
više završenog posla.
```

---

# 4. Latency

Latency odgovara na drugo pitanje:

```
Koliko traje

jedan zahtev?
```

---

Na primer:

```
HTTP Request

↓

120 ms
```

---

ili

```
Database Query

↓

8 ms
```

---

Manja latencija znači:

```
brži odgovor
```

za pojedinačnog korisnika.

---

# 5. Throughput vs Latency

Ova dva cilja nisu uvek ista.

---

Primer:

Dodavanje većeg broja worker-a može povećati:

```
throughput
```

---

Ali istovremeno povećati:

```
latency
```

zbog contention-a i scheduler overhead-a.

---

Vizuelno:

```
Worker-i ↑

↓

Throughput ↑

↓

Latency ?

```

---

Zbog toga se uvek meri oba parametra.

---

# 6. Skalabilnost

Dobar konkurentni program treba da:

```
skalira
```

---

Na primer:

```
1 CPU

↓

100 req/s
```

---

```
2 CPU

↓

190 req/s
```

---

```
4 CPU

↓

370 req/s
```

---

Idealno bi bilo:

```
400 req/s
```

ali u praksi postoje gubici.

---

# 7. Zašto Skaliranje Nije Linearno?

Dodavanjem više goroutine-a povećava se:

- sinhronizacija
- contention
- cache misses
- scheduler aktivnost

---

Rezultat:

```
više nije

100%

skaliranje.
```

---

# 8. CPU-Bound Poslovi

CPU-bound posao najveći deo vremena provodi:

```
računajući.
```

---

Primeri:

- kompresija
- enkripcija
- parsiranje velikih fajlova
- obrada slika
- video encoding

---

Ovde broj CPU jezgara ima veliki uticaj.

---

# 9. I/O-Bound Poslovi

I/O-bound posao najveći deo vremena:

```
čeka.
```

---

Na primer:

- mrežu
- bazu podataka
- disk
- REST API
- gRPC poziv

---

CPU često miruje dok goroutine čeka odgovor.

---

# 10. Zašto Goroutine Pomažu Kod I/O?

Dok jedna goroutine čeka:

```
Database
```

---

Scheduler može pokrenuti drugu goroutine.

---

Vizuelno:

```
G1

↓

waiting
```

---

```
CPU

↓

G2

↓

running
```

---

Na taj način CPU ostaje iskorišćen.

---

# 11. Zašto Goroutine Nekad Odmažu?

CPU-bound primer:

```
8 CPU

↓

1000 goroutines
```

---

Sve žele CPU u isto vreme.

---

Posledice:

- više context switch-eva
- scheduler overhead
- contention
- manji cache hit ratio

---

Program može postati sporiji.

---

# 12. Paralelizam Nije Besplatan

Svaka goroutine donosi određeni trošak.

---

Na primer:

- inicijalni stack
- scheduler bookkeeping
- sinhronizaciju
- eventualne alokacije

---

Pojedinačni trošak je mali,

ali hiljade goroutine-a ga mogu učiniti značajnim.

---

# 13. Najčešće Greške

❌ "Više goroutine-a znači više brzine."

---

❌ "Koristiću 10 000 worker-a."

---

❌ "Concurrency automatski ubrzava CPU-bound posao."

---

❌ "Ne moram da merim performanse."

---

Sve ove pretpostavke često vode do lošijih rezultata.

---

# 14. Performance Mental Model

Pre nego što uvedeš konkurentnost,

postavi četiri pitanja:

1. Da li je posao CPU-bound ili I/O-bound?

2. Šta je trenutno usko grlo (bottleneck)?

3. Da li će konkurentnost povećati throughput ili samo dodati overhead?

4. Kako ću izmeriti rezultat?

---

Odgovori na ova pitanja određuju da li će konkurentnost zaista doneti korist.

---

# 15. Senior Pravila

✔️ Concurrency nije cilj, već alat.

---

✔️ Throughput i latency treba posmatrati zajedno.

---

✔️ CPU-bound i I/O-bound problemi zahtevaju različite strategije.

---

✔️ Skalabilnost treba meriti, a ne pretpostavljati.

---

✔️ Svaka optimizacija mora biti potkrepljena benchmark-om ili profilisanjem.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je performance tuning

✅ throughput

✅ latency

✅ skalabilnost

✅ CPU-bound vs I/O-bound

✅ zašto paralelizam nije besplatan

✅ osnovni mentalni model za optimizaciju

---

# Concurrency Performance Tuning

## Deo #2 — Go Scheduler i Performance

---

# 📚 Sadržaj

- Zašto je Scheduler važan?
- G-M-P model
- Local Run Queue
- Global Run Queue
- Work Stealing
- Scheduler Overhead
- Context Switching
- Asynchronous Preemption
- Performance smernice

---

# 1. Zašto Je Scheduler Važan?

Svaka goroutine koju napišemo mora biti izvršena.

To nije posao operativnog sistema,

već:

```
Go Scheduler-a.
```

---

Scheduler odlučuje:

- koja goroutine će raditi
- kada će raditi
- na kom thread-u će raditi
- kada će biti prekinuta
- kada će nastaviti izvršavanje

---

Zbog toga Scheduler direktno utiče na:

- throughput
- latency
- CPU iskorišćenost
- skalabilnost

---

# 2. G-M-P Model

Scheduler koristi poznati:

```
G-M-P
```

model.

---

G predstavlja:

```
Goroutine
```

---

M predstavlja:

```
Machine

(OS Thread)
```

---

P predstavlja:

```
Processor

(logički izvršni resurs)
```

---

Vizuelno:

```
G

↓

P

↓

M

↓

CPU
```

---

# 3. Processor (`P`)

`P` nije fizički procesor.

---

On predstavlja resurs koji omogućava izvršavanje Go koda.

---

Broj `P` instanci određuje:

```go
runtime.GOMAXPROCS()
```

---

Na primer:

```
GOMAXPROCS = 8
```

↓

Scheduler može istovremeno izvršavati Go kod na:

```
8 Processor-a.
```

---

# 4. Local Run Queue

Svaki `P` poseduje:

```
Local Run Queue
```

---

Primer:

```
P1

↓

G1

G2

G3

G4
```

---

Scheduler prvo uzima goroutine iz:

```
lokalnog reda.
```

---

To smanjuje contention između Processor-a.

---

# 5. Global Run Queue

Pored lokalnih redova postoji i:

```
Global Run Queue
```

---

Koristi se kada:

- novi posao nema lokalni red
- lokalni red postane prazan
- Scheduler balansira opterećenje

---

Vizuelno:

```
Global Queue

↓

P1

↓

P2

↓

P3
```

---

# 6. Work Stealing

Jedna od najvažnijih optimizacija Scheduler-a.

---

Primer:

```
P1

↓

0 goroutines
```

---

```
P2

↓

150 goroutines
```

---

Umesto da P1 miruje,

on "krade" deo posla od P2.

---

Vizuelno:

```
P2

↓↓↓↓↓↓

P1
```

---

Rezultat:

- bolja iskorišćenost CPU-a
- ravnomernije opterećenje
- veći throughput

---

# 7. Scheduler Overhead

Scheduler nije besplatan.

---

Svaka odluka zahteva:

- proveru redova
- promenu aktivne goroutine
- bookkeeping
- sinhronizaciju

---

Kod malog broja goroutine-a,

ovaj trošak je zanemarljiv.

---

Kod stotina hiljada goroutine-a,

overhead postaje merljiv.

---

# 8. Context Switching

Kada Scheduler promeni aktivnu goroutine,

dolazi do:

```
context switch-a.
```

---

Za razliku od OS thread switch-a,

Go context switch je znatno jeftiniji.

---

Ipak,

nije besplatan.

---

Previše čestih promena može smanjiti performanse.

---

# 9. Asynchronous Preemption

Od Go 1.14,

Scheduler može prekinuti dugotrajnu goroutine

čak i ako sama ne napravi blokirajući poziv.

---

To sprečava:

- starvation
- monopolizaciju CPU-a
- loš odziv sistema

---

Ova mogućnost značajno poboljšava fer raspodelu CPU vremena.

---

# 10. GOMAXPROCS

`GOMAXPROCS` određuje:

```
maksimalan broj

Processor-a (`P`)
```

koji mogu istovremeno izvršavati Go kod.

---

Važno:

```
GOMAXPROCS

≠

broj goroutine-a
```

---

Može postojati:

```
100 000 goroutine-a
```

i

```
8 Processor-a.
```

---

Scheduler ih raspoređuje po potrebi.

---

# 11. Kada Scheduler Postaje Bottleneck?

Retko,

ali moguće.

---

Na primer:

- ekstremno kratke goroutine
- ogromna količina sinhronizacije
- veliki broj wake-up događaja
- intenzivna komunikacija preko channel-a

---

U takvim slučajevima,

Scheduler može trošiti značajan deo CPU vremena.

---

# 12. Performance Smernice

✔️ Nemoj kreirati goroutine za veoma mali posao.

---

✔️ Grupiši sitne zadatke kada je moguće.

---

✔️ Izbegavaj nepotrebno buđenje velikog broja goroutine-a.

---

✔️ Worker Pool često daje bolje rezultate od "goroutine po zadatku".

---

# 13. Najčešće Greške

❌ Pretpostavka da Scheduler može besplatno upravljati milionima kratkotrajnih goroutine-a.

---

❌ Korišćenje previše worker-a za CPU-bound zadatke.

---

❌ Ignorisanje Scheduler overhead-a prilikom benchmark-a.

---

❌ Ručno podešavanje `GOMAXPROCS` bez merenja.

---

# 14. Mentalni Model

Zamisli Scheduler kao dispečera.

---

On ne izvršava posao,

već odlučuje:

```
ko

kada

i gde

radi.
```

---

Što je raspored efikasniji,

to će CPU biti bolje iskorišćen.

---

# 15. Senior Pravila

✔️ Scheduler je optimizovan, ali nije magičan.

---

✔️ Lokalni redovi smanjuju contention.

---

✔️ Work Stealing povećava iskorišćenost CPU-a.

---

✔️ Context switch ima cenu.

---

✔️ Broj goroutine-a nije isto što i stepen paralelizma.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ G-M-P model

✅ Processor (`P`)

✅ Local i Global Run Queue

✅ Work Stealing

✅ Scheduler Overhead

✅ Context Switching

✅ Asynchronous Preemption

✅ `GOMAXPROCS`

---

# Concurrency Performance Tuning

## Deo #3 — Contention, Cache Locality i False Sharing

---

# 📚 Sadržaj

- Šta je contention?
- Lock contention
- Channel contention
- Cache hijerarhija
- Cache locality
- Cache misses
- False Sharing
- Padding
- Performance smernice

---

# 1. Šta Je Contention?

Contention nastaje kada više goroutine pokušava da koristi isti resurs u isto vreme.

---

Primeri zajedničkih resursa:

- `sync.Mutex`
- `sync.RWMutex`
- channel
- atomic promenljive
- memorijske lokacije

---

Vizuelno:

```
G1

   \

    \

     v

   Resource

     ^

    /

   /

G2
```

---

Što je contention veći,

to više vremena goroutine provode čekajući,

umesto radeći koristan posao.

---

# 2. Lock Contention

Primer:

```go
var mu sync.Mutex

func increment() {
	mu.Lock()
	counter++
	mu.Unlock()
}
```

---

Ako:

```
1000 goroutine-a
```

poziva ovu funkciju,

samo jedna goroutine može držati `Mutex` u datom trenutku.

---

Ostale čekaju.

---

Rezultat:

```
throughput ↓

latency ↑
```

---

# 3. Kako Smanjiti Lock Contention?

Nekoliko strategija:

✔️ zaključavati (`Lock`) što kraće

---

✔️ smanjiti broj deljenih podataka

---

✔️ koristiti više nezavisnih `Mutex`-a (lock striping)

---

✔️ koristiti `atomic` kada je dovoljno

---

✔️ izbegavati nepotrebnu sinhronizaciju

---

# 4. Channel Contention

Channel takođe može postati usko grlo.

---

Primer:

```
1000 Producer-a

↓

jedan channel

↓

1 Consumer
```

---

Svi producer-i se takmiče za isti channel.

---

Kako broj goroutine-a raste,

raste i contention.

---

# 5. Kako Smanjiti Channel Contention?

Moguća rešenja:

- više channel-a
- fan-out
- worker pool
- batching
- smanjenje broja producer-a

---

Ne postoji univerzalno rešenje.

---

Potrebno je merenje.

---

# 6. CPU Cache

Moderni procesori imaju više nivoa keša.

```
CPU

↓

L1 Cache

↓

L2 Cache

↓

L3 Cache

↓

RAM
```

---

L1 je:

- najmanji
- najbrži

---

RAM je:

- najveći
- najsporiji

---

Pristup RAM-u može biti višestruko sporiji od pristupa L1 kešu.

---

# 7. Cache Locality

Cache locality opisuje koliko dobro program koristi procesorski keš.

---

Ako se podaci nalaze blizu jedni drugih u memoriji,

procesor ih može učitati zajedno.

---

Primer:

```
A

B

C

D
```

---

Sekvencijalni pristup ovim podacima često je veoma efikasan.

---

# 8. Cache Miss

Ako traženi podatak nije u kešu,

dolazi do:

```
cache miss
```

---

Procesor mora da čeka:

```
RAM
```

---

Posledice:

- veća latencija
- manje izvršenih instrukcija po sekundi
- niži throughput

---

# 9. Šta Je False Sharing?

Jedan od najčešće zanemarenih problema u konkurentnom programiranju.

---

Dve goroutine menjaju:

```
različite promenljive
```

---

Ali te promenljive se nalaze u:

```
istoj cache liniji.
```

---

Procesor tada mora stalno da sinhronizuje keš između jezgara.

---

# 10. Primer False Sharing-a

```go
type Counters struct {
	a int64
	b int64
}
```

---

Ako:

- G1 menja `a`
- G2 menja `b`

---

Iako nema logičkog konflikta,

može postojati fizički konflikt zbog zajedničke cache linije.

---

Rezultat:

```
veliki pad performansi.
```

---

# 11. Padding

Jedno od rešenja je dodavanje razmaka (`padding`).

```go
type Counters struct {
	a int64

	_ [56]byte

	b int64
}
```

---

Sada se promenljive nalaze u različitim cache linijama.

---

Ovo može značajno smanjiti contention između CPU jezgara.

---

# 12. Cache-Friendly Dizajn

Dobar raspored podataka često donosi više koristi od dodavanja novih goroutine-a.

---

Na primer:

✔️ sekvencijalni pristup memoriji

✔️ kompaktne strukture

✔️ grupisanje često korišćenih podataka

---

Sve ovo poboljšava cache locality.

---

# 13. Kada Razmišljati O False Sharing-u?

U većini aplikacija,

ne treba odmah optimizovati ovaj aspekt.

---

Ali kod:

- visokoperformansnih servera
- baza podataka
- mrežnih sistema
- telemetrije
- lock-free algoritama

---

False Sharing može biti značajan izvor usporenja.

---

# 14. Performance Mental Model

Brzina programa ne zavisi samo od algoritma.

---

Već i od:

```
CPU

↓

Cache

↓

Memory Layout

↓

Synchronization
```

---

Često je memorijski raspored podataka jednako važan kao i sam algoritam.

---

# 15. Senior Pravila

✔️ Minimizuj contention.

---

✔️ Zaključavaj što kraće.

---

✔️ Dizajniraj podatke tako da budu cache-friendly.

---

✔️ Razmišljaj o memorijskom rasporedu kod kritičnih delova sistema.

---

✔️ False Sharing optimizuj tek nakon profilisanja.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ contention

✅ lock contention

✅ channel contention

✅ CPU cache

✅ cache locality

✅ cache misses

✅ False Sharing

✅ padding

---

# Concurrency Performance Tuning

## Deo #4 — Benchmark-Driven Optimization

---

# 📚 Sadržaj

- Zašto benchmark?
- Performance Engineering Workflow
- Pisanje benchmark testova
- `go test -bench`
- `benchmem`
- Paralelni benchmark-i
- Poređenje rezultata
- Najčešće greške
- Production preporuke

---

# 1. Zašto Benchmark?

Performance se ne procenjuje "odokativno".

---

Dve implementacije mogu izgledati gotovo identično,

a razlika u brzini može biti višestruka.

---

Primer:

```go
Implementation A

↓

12 ms
```

---

```go
Implementation B

↓

4 ms
```

---

Bez benchmark-a,

nemoguće je znati koja je stvarno brža.

---

# 2. Performance Engineering Workflow

Dobar proces optimizacije izgleda ovako:

```
Implementacija

↓

Benchmark

↓

Profilisanje

↓

Identifikacija Bottleneck-a

↓

Optimizacija

↓

Ponovni Benchmark

↓

Poređenje Rezultata
```

---

Nikada:

```
Implementacija

↓

Nasumična optimizacija

↓

Nada da je brže
```

---

# 3. Pisanje Benchmark Testa

Benchmark funkcije koriste paket:

```go
testing
```

---

Primer:

```go
func BenchmarkProcess(b *testing.B) {
	for b.Loop() {
		Process()
	}
}
```

---

> **Napomena:** `b.Loop()` je dostupan u novijim verzijama Go-a i predstavlja preporučeni način pisanja benchmark petlje. U starijim verzijama koristi se obrazac sa `for i := 0; i < b.N; i++ { ... }`.

---

Benchmark funkcije moraju početi sa:

```
Benchmark
```

---

# 4. Pokretanje Benchmark-a

Najjednostavnija komanda:

```bash
go test -bench=.
```

---

Primer izlaza:

```text
BenchmarkProcess-8

1000000

1150 ns/op
```

---

Ovde:

- broj izvršavanja automatski određuje framework
- `ns/op` predstavlja prosečno vreme jedne operacije

---

# 5. Merenje Alokacija

Dodavanjem:

```bash
go test -bench=. -benchmem
```

dobijamo dodatne informacije.

---

Primer:

```text
1150 ns/op

128 B/op

2 allocs/op
```

---

Značenje:

- vreme izvršavanja
- bajtovi po operaciji
- broj alokacija po operaciji

---

Kod konkurentnih programa,

broj alokacija često direktno utiče na GC i ukupne performanse.

---

# 6. Paralelni Benchmark

Go omogućava benchmark paralelnog izvršavanja.

```go
func BenchmarkParallel(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Process()
		}
	})
}
```

---

Ovakvi benchmark-i bolje simuliraju stvarno konkurentno opterećenje.

---

Posebno su korisni za:

- worker pool
- mutex
- channel
- atomic operacije

---

# 7. Benchmark Nije Load Test

Važno je razlikovati:

```
Benchmark
```

i

```
Load Test
```

---

Benchmark meri:

```
performanse funkcije
```

---

Load test meri:

```
ponašanje kompletnog sistema
```

pod velikim opterećenjem.

---

Oba su važna,

ali rešavaju različite probleme.

---

# 8. Kako Porediti Rezultate?

Nakon optimizacije,

ponovo pokrenuti benchmark.

---

Primer:

```
Pre

↓

2500 ns/op
```

---

```
Posle

↓

1700 ns/op
```

---

Tek tada možemo tvrditi da postoji poboljšanje.

---

Bez merenja,

svaka tvrdnja o performansama ostaje pretpostavka.

---

# 9. Stabilnost Benchmark-a

Jedan rezultat nije dovoljan.

---

Na rezultate utiču:

- druge aplikacije
- frekvencija procesora
- GC
- scheduler
- operativni sistem

---

Zbog toga benchmark treba pokrenuti više puta.

---

Tražiti:

```
stabilan trend
```

a ne pojedinačne vrednosti.

---

# 10. Šta Benchmark Ne Pokazuje?

Benchmark ne otkriva automatski:

- uzrok sporosti
- lock contention
- scheduler overhead
- cache misses
- GC pauze

---

Za to služi:

```
profilisanje.
```

---

Benchmark odgovara na pitanje:

```
Koliko je brzo?
```

---

Profilisanje odgovara:

```
Zašto nije brže?
```

---

# 11. Najčešće Greške

❌ Benchmark koji uključuje inicijalizaciju umesto same operacije.

---

❌ Poređenje rezultata sa različitih računara bez opreza.

---

❌ Zaključivanje na osnovu jednog pokretanja.

---

❌ Fokus samo na `ns/op` uz ignorisanje `allocs/op`.

---

❌ Optimizacija bez prethodnog benchmark-a.

---

# 12. Production Preporuke

✔️ Benchmark pre svake ozbiljne optimizacije.

---

✔️ Koristiti `-benchmem`.

---

✔️ Meriti i sekvencijalne i paralelne scenarije.

---

✔️ Čuvati benchmark rezultate radi poređenja.

---

✔️ Automatizovati benchmark-e za kritične delove sistema.

---

# 13. Mentalni Model

Benchmark je:

```
merač brzine.
```

---

On ne govori:

```
zašto
```

je nešto sporo.

---

Ali pouzdano govori:

```
koliko
```

je brzo.

---

Tek zajedno sa profilisanjem,

benchmark postaje moćan alat za optimizaciju.

---

# 14. Senior Pravila

✔️ Prvo benchmark, pa optimizacija.

---

✔️ Benchmark bez `benchmem` često daje nepotpunu sliku.

---

✔️ Poredi rezultate pre i posle izmene.

---

✔️ Optimizuj samo ono što je dokazano usko grlo.

---

✔️ Performanse su rezultat merenja, ne intuicije.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ benchmark metodologiju

✅ `go test -bench`

✅ `-benchmem`

✅ paralelne benchmark-e

✅ poređenje rezultata

✅ razliku između benchmark-a i profilisanja

✅ production preporuke

---

# Concurrency Performance Tuning

## Deo #5 — Profiling Concurrent Applications

---

# 📚 Sadržaj

- Zašto profilisanje?
- `pprof`
- CPU Profiling
- Memory (Heap) Profiling
- Goroutine Profiling
- Block Profiling
- Mutex Profiling
- Trace
- Production preporuke

---

# 1. Zašto Profilisanje?

Benchmark odgovara na pitanje:

```
Koliko je brzo?
```

---

Profilisanje odgovara:

```
Zašto nije brže?
```

---

Na primer:

```
Program traje

3 sekunde.
```

---

Ali:

- gde odlazi vreme?
- koja funkcija je spora?
- gde goroutine čekaju?
- gde nastaju alokacije?

---

Na ova pitanja odgovara:

```
profiling.
```

---

# 2. Šta Je `pprof`?

Go standardna biblioteka pruža moćan alat:

```text
pprof
```

---

Omogućava analizu:

- CPU vremena
- memorije
- goroutine-a
- blokiranja
- mutex contention-a

---

U većini slučajeva nije potreban dodatni softver.

---

# 3. CPU Profiling

CPU profil pokazuje:

```
gde procesor

provodi vreme.
```

---

Primer komande:

```bash
go test -cpuprofile=cpu.prof
```

---

Rezultat je profil koji se može analizirati pomoću:

```bash
go tool pprof cpu.prof
```

---

CPU profil pomaže u pronalaženju:

- sporih algoritama
- nepotrebnih petlji
- skupih funkcija

---

# 4. Memory (Heap) Profiling

Heap profil pokazuje:

- gde nastaju alokacije
- koje funkcije troše memoriju
- koji delovi programa stvaraju pritisak na GC

---

Primer:

```bash
go test -memprofile=mem.prof
```

---

Analiza:

```bash
go tool pprof mem.prof
```

---

Kod konkurentnih programa,

preveliki broj alokacija često povećava GC overhead.

---

# 5. Goroutine Profiling

Može se analizirati stanje svih goroutine-a.

---

Primer:

```bash
go tool pprof goroutine.prof
```

---

Ovaj profil pomaže u otkrivanju:

- curenja goroutine-a (goroutine leaks)
- zaglavljenih goroutine-a
- neočekivanog broja aktivnih goroutine-a

---

Ako aplikacija konstantno povećava broj goroutine-a,

to je ozbiljan signal za analizu.

---

# 6. Block Profiling

Block profil prikazuje:

```
gde goroutine čekaju.
```

---

Najčešći razlozi:

- channel operacije
- `select`
- `sync.Cond`
- drugi mehanizmi sinhronizacije

---

Ako aplikacija provodi mnogo vremena blokirana,

throughput će biti manji bez obzira na broj CPU jezgara.

---

# 7. Mutex Profiling

Mutex profil pokazuje:

```
koliko vremena

goroutine čekaju

na zaključavanje (`Lock`).
```

---

Koristan je za otkrivanje:

- lock contention-a
- predugih kritičnih sekcija
- loše raspodele zaključavanja

---

Ako je veliki deo vremena proveden čekajući `Mutex`,

optimizacija algoritma možda neće pomoći

dok se contention ne smanji.

---

# 8. Execution Trace

Pored `pprof`-a,

Go nudi i:

```
Execution Trace
```

---

On prikazuje:

- Scheduler događaje
- kreiranje goroutine-a
- blokiranja
- mrežne operacije
- GC događaje

---

Trace daje vremensku sliku izvršavanja cele aplikacije.

---

# 9. Kako Izgleda Proces Analize?

Tipičan tok rada:

```
Benchmark

↓

Profil

↓

Pronađi Bottleneck

↓

Optimizuj

↓

Benchmark

↓

Uporedi Rezultate
```

---

Ako optimizacija nije donela poboljšanje,

treba ponovo analizirati profil,

a ne nagađati.

---

# 10. Šta Profil Najčešće Otkrije?

U praksi,

profil često pokaže:

- neočekivane alokacije
- lock contention
- previše channel komunikacije
- skupe funkcijske pozive
- nepotrebne kopije podataka
- GC pritisak

---

Vrlo često pravi problem nije tamo gde se očekuje.

---

# 11. Najčešće Greške

❌ Profilisanje debug build-a umesto reprezentativnog okruženja.

---

❌ Donošenje zaključaka bez benchmark-a.

---

❌ Fokus samo na CPU profilu.

---

❌ Ignorisanje memorijskih alokacija.

---

❌ Optimizovanje funkcije koja nije bottleneck.

---

# 12. Production Preporuke

✔️ Kombinovati benchmark i profilisanje.

---

✔️ Profilisati stvarne scenarije korišćenja.

---

✔️ Čuvati profile pre i posle optimizacije.

---

✔️ Posmatrati trendove, ne pojedinačne rezultate.

---

✔️ Redovno analizirati GC i broj goroutine-a kod dugotrajnih servisa.

---

# 13. Mentalni Model

Benchmark meri:

```
rezultat.
```

---

Profil pokazuje:

```
uzrok.
```

---

Zajedno predstavljaju osnovni alat svakog Senior Go programera za optimizaciju performansi.

---

# 14. Senior Pravila

✔️ Nikada ne optimizuj "naslepo".

---

✔️ Profil je pouzdaniji od pretpostavke.

---

✔️ Svaki veliki performance problem ostavlja trag u profilu.

---

✔️ Profilisanje treba biti deo redovnog razvojnog procesa.

---

✔️ Najveći dobitci obično dolaze uklanjanjem jednog velikog bottleneck-a,

a ne nizom sitnih optimizacija.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ `pprof`

✅ CPU profiling

✅ Memory profiling

✅ Goroutine profiling

✅ Block profiling

✅ Mutex profiling

✅ Execution Trace

✅ kako pronaći bottleneck

---

# Concurrency Performance Tuning

## Deo #6 — Performance Tuning Cheat Sheet i Production Best Practices

---

# 📚 Sadržaj

- Performance Workflow
- Performance Checklist
- Concurrency Checklist
- Anti-Patterns
- Optimization Priorities
- Senior Mental Model
- Cheat Sheet
- Završni pregled modula

---

# 1. Performance Workflow

Profesionalna optimizacija uvek prati isti tok.

```
Implementacija

↓

Benchmark

↓

Profilisanje

↓

Identifikacija Bottleneck-a

↓

Optimizacija

↓

Ponovni Benchmark

↓

Poređenje Rezultata
```

---

Nikada:

```
Implementacija

↓

Pretpostavka

↓

Optimizacija
```

---

Pretpostavke često vode ka složenijem kodu bez stvarnog poboljšanja performansi.

---

# 2. Performance Checklist

Pre svake optimizacije postavi sledeća pitanja.

---

✔️ Da li postoji benchmark?

---

✔️ Da li postoji CPU profil?

---

✔️ Da li postoji Heap profil?

---

✔️ Da li je identifikovan bottleneck?

---

✔️ Da li je optimizacija zaista potrebna?

---

✔️ Da li postoji poređenje rezultata pre i posle izmene?

---

Ako je odgovor "ne" na neko od pitanja,

verovatno je prerano za optimizaciju.

---

# 3. Concurrency Checklist

Kod konkurentnih programa proveri:

✔️ broj goroutine-a

---

✔️ contention nad `Mutex`-ima

---

✔️ contention nad channel-ima

---

✔️ broj alokacija

---

✔️ GC aktivnost

---

✔️ scheduler overhead

---

✔️ cache locality

---

✔️ throughput

---

✔️ latency

---

Sve ove metrike zajedno daju realnu sliku performansi.

---

# 4. Redosled Optimizacije

Najčešće se optimizuje ovim redom:

```
Algoritam

↓

Alokacije

↓

GC pritisak

↓

Contention

↓

Scheduler

↓

Memory Layout

↓

Micro-optimizacije
```

---

Najveći dobici gotovo uvek dolaze iz prvih koraka.

---

# 5. Najčešći Anti-Patterns

❌ Dodavanje novih goroutine-a bez potrebe.

---

❌ Pretpostavka da više worker-a znači bolje performanse.

---

❌ Ignorisanje benchmark rezultata.

---

❌ Optimizovanje funkcija koje nisu bottleneck.

---

❌ Preuranjena optimizacija (premature optimization).

---

❌ Korišćenje lock-free algoritama bez jasnog razloga.

---

❌ Ručno podešavanje `GOMAXPROCS` bez merenja.

---

# 6. Kada Koristiti Concurrency?

Koristi konkurentnost kada:

✔️ postoji nezavisan posao

---

✔️ postoji I/O čekanje

---

✔️ postoji više CPU jezgara koja mogu biti iskorišćena

---

✔️ postoji dokaz da konkurentnost povećava throughput ili smanjuje latency

---

Nemoj koristiti konkurentnost samo zato što je dostupna.

---

# 7. Mentalni Model Performansi

Brzina aplikacije zavisi od mnogo faktora.

```
Algoritam

↓

Memory Layout

↓

GC

↓

Synchronization

↓

Scheduler

↓

Hardware
```

---

Optimizacija jednog sloja često neće pomoći ako je pravi problem u drugom.

---

# 8. Senior Mentalni Model

Senior Go programer razmišlja ovako:

```
Šta je bottleneck?
```

---

Ne pita:

```
Koju optimizaciju da primenim?
```

---

Već:

```
Koja optimizacija rešava
konkretno usko grlo?
```

---

To vodi ka jednostavnijem kodu i merljivim rezultatima.

---

# 9. Performance Cheat Sheet

| Problem | Prvi korak |
|----------|------------|
| Spor algoritam | CPU Profil |
| Previše memorije | Heap Profil |
| Mnogo goroutine-a | Goroutine Profil |
| Čekanje na `Mutex` | Mutex Profil |
| Čekanje na channel | Block Profil |
| Nepoznato ponašanje Scheduler-a | Execution Trace |
| Visok broj alokacija | `-benchmem` |
| Loš throughput | Benchmark + Profil |

---

# 10. Pravilo 80/20

U velikom broju slučajeva važi:

```
20%

koda

↓

80%

potrošenog vremena
```

---

Fokus treba usmeriti upravo na tih 20%.

---

Optimizovanje ostatka često nema primetan efekat.

---

# 11. Production Best Practices

✔️ Benchmark pre svake ozbiljne optimizacije.

---

✔️ Profilisati pre donošenja zaključaka.

---

✔️ Automatizovati benchmark testove.

---

✔️ Pratiti performanse tokom razvoja,

a ne samo pred produkciju.

---

✔️ Dokumentovati razloge za svaku optimizaciju.

---

✔️ Ponovo meriti nakon svake izmene.

---

# 12. Kako Izgleda Production Workflow?

```
Feature

↓

Benchmark

↓

CPU Profil

↓

Heap Profil

↓

Optimization

↓

Regression Benchmark

↓

Deploy

↓

Monitoring

↓

Profiling u produkciji (po potrebi)
```

---

Ovakav pristup smanjuje rizik od regresija i omogućava kontinuirano unapređenje performansi.

---

# 13. Završni Pregled Modula

U ovom modulu obradili smo:

✅ throughput

✅ latency

✅ skalabilnost

✅ CPU-bound i I/O-bound zadatke

✅ Go Scheduler

✅ G-M-P model

✅ Work Stealing

✅ contention

✅ cache locality

✅ false sharing

✅ benchmark metodologiju

✅ `benchmem`

✅ CPU i Heap profiling

✅ Goroutine, Block i Mutex profiling

✅ Execution Trace

✅ production strategije za optimizaciju

---

# 14. Šta Sada Znaš?

Sada razumeš:

- kako proceniti da li konkurentnost zaista donosi korist
- kako razlikovati probleme algoritma od problema sinhronizacije
- kako koristiti benchmark i profilisanje zajedno
- kako pronaći i otkloniti bottleneck
- kako analizirati scheduler, contention i memoriju
- kako donositi odluke zasnovane na merenjima, a ne na pretpostavkama

---

# 15. Završna Poruka

Najvažnija lekcija ovog modula može se sažeti u jednu rečenicu:

> **Ne optimizuj ono što ne možeš da izmeriš.**

---

Brz Go program nije rezultat velikog broja goroutine-a,

već rezultat:

- dobrog algoritma
- pažljivog dizajna
- pravilne sinhronizacije
- kvalitetnih merenja
- kontinuiranog profilisanja

---

### ➡️ Sledeća lekcija **[**Profiling Concurrent Applications**](09-profiling-concurrent-applications.md)**

Obuhvatiće:

- napredni rad sa `pprof`
- flame graph analizu
- call graph interpretaciju
- `go tool trace`
- scheduler timeline
- GC događaje
- analizu goroutine životnog ciklusa
- identifikaciju memory leak-ova i goroutine leak-ova
- production dijagnostiku konkurentnih Go aplikacija