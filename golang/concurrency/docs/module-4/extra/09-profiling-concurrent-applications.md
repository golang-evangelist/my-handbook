# Profiling Concurrent Applications

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Profiling Concurrent Applications
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Šta je profiling?
- Benchmark vs Profiling
- Zašto je profiling važan?
- Vrste profila u Go-u
- `runtime/pprof`
- `net/http/pprof`
- Performance Investigation Workflow
- Mentalni model profilisanja

---

# 1. Šta Je Profiling?

Profiling je proces prikupljanja informacija o tome kako se program ponaša tokom izvršavanja.

---

Za razliku od benchmark-a,

profiling ne meri samo:

```
koliko brzo
```

već pokušava da odgovori:

```
gde

zašto

i kako

program troši resurse.
```

---

Profiling može pokazati:

- gde CPU provodi vreme
- gde nastaju alokacije
- gde goroutine čekaju
- gde dolazi do contention-a
- koliko traje Garbage Collection
- kako Scheduler raspoređuje goroutine

---

# 2. Zašto Je Profiling Važan?

Pretpostavimo da aplikacija radi sporo.

Bez profilisanja često se čuju izjave poput:

> "Sigurno je problem u GC-u."

ili

> "Treba dodati još goroutine-a."

---

U praksi,

uzrok može biti potpuno drugačiji:

- spor algoritam
- loša SQL optimizacija
- previše alokacija
- contention nad `Mutex`-om
- blokiranje na channel-u
- spori mrežni pozivi

---

Profiling eliminiše nagađanje.

---

# 3. Benchmark vs Profiling

Ova dva alata se dopunjuju.

| Benchmark | Profiling |
|-----------|-----------|
| Meri brzinu | Objašnjava zašto |
| Daje `ns/op` | Pokazuje bottleneck |
| Poredi implementacije | Analizira izvršavanje |
| Fokus na rezultat | Fokus na uzrok |

---

Najbolji rezultati postižu se kada se koriste zajedno.

---

# 4. Vrste Profila u Go-u

Go podržava više vrsta profila.

Najvažniji su:

- CPU Profile
- Heap Profile
- Allocation Profile
- Goroutine Profile
- Block Profile
- Mutex Profile
- Thread Creation Profile
- Execution Trace

---

Svaki odgovara na različito pitanje.

---

# 5. CPU Profile

CPU profil odgovara:

```
Gde CPU provodi vreme?
```

---

Koristi se za:

- spore algoritme
- skupe funkcijske pozive
- previše računanja
- neočekivane petlje

---

CPU profil je najčešća početna tačka analize.

---

# 6. Heap Profile

Heap profil pokazuje:

```
Ko koristi memoriju?
```

---

Pomaže u otkrivanju:

- velikih objekata
- nepotrebnih alokacija
- povećanog GC pritiska
- memory leak-ova

---

Važno je razlikovati:

- ukupno alociranu memoriju
- trenutno zauzetu memoriju na heap-u

---

# 7. Allocation Profile

Allocation profil meri:

```
Koliko memorije
je alocirano
tokom izvršavanja.
```

---

Program može imati:

- mali Heap

ali

- ogroman broj kratkotrajnih alokacija.

---

To često povećava GC aktivnost.

---

# 8. Goroutine Profile

Odgovara na pitanje:

```
Koliko goroutine-a postoji?

I šta rade?
```

---

Koristan je za:

- goroutine leak
- deadlock
- starvation
- blokirane goroutine
- neočekivano veliki broj worker-a

---

# 9. Block Profile

Pokazuje:

```
Gde goroutine čekaju?
```

---

Najčešći uzroci:

- channel
- `select`
- `sync.Cond`
- druge blokirajuće operacije

---

Ako aplikacija mnogo vremena provodi čekajući,

to će se jasno videti u ovom profilu.

---

# 10. Mutex Profile

Odgovara:

```
Koliko vremena
goroutine čekaju
na Lock()?
```

---

Koristi se za analizu:

- contention-a
- predugih kritičnih sekcija
- neefikasne sinhronizacije

---

# 11. Thread Creation Profile

Ovaj profil prikazuje:

```
koliko OS thread-ova
runtime kreira.
```

---

Koristan je kada:

- postoji mnogo blokirajućih sistemskih poziva
- Scheduler mora da kreira dodatne thread-ove
- dolazi do neočekivanog rasta broja thread-ova

---

Kod većine aplikacija koristi se ređe,

ali može biti veoma koristan u specifičnim slučajevima.

---

# 12. Execution Trace

Execution Trace predstavlja:

```
vremensku mapu
izvršavanja
cele aplikacije.
```

---

Prikazuje:

- Scheduler događaje
- GC
- goroutine
- syscall
- mrežne operacije
- preemption
- wake-up događaje

---

To je jedan od najdetaljnijih alata koje Go pruža.

---

# 13. Performance Investigation Workflow

Tipičan tok analize izgleda ovako:

```
Problem

↓

Benchmark

↓

CPU Profil

↓

Heap Profil

↓

Block/Mutex Profil

↓

Execution Trace

↓

Identifikacija Bottleneck-a

↓

Optimizacija

↓

Ponovni Benchmark
```

---

Ovakav pristup omogućava da se problemi rešavaju sistematski.

---

# 14. Mentalni Model

Profilisanje je slično medicinskoj dijagnostici.

---

Benchmark kaže:

```
Pacijent ima temperaturu.
```

---

Profil kaže:

```
Šta je uzrok?
```

---

Bez dijagnoze,

lečenje je često pogrešno.

---

# 15. Senior Pravila

✔️ Benchmark meri rezultat.

---

✔️ Profil pokazuje uzrok.

---

✔️ Ne postoji univerzalni profil.

---

✔️ Različiti problemi zahtevaju različite profile.

---

✔️ Profilisanje treba biti sastavni deo razvoja ozbiljnih Go aplikacija.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ šta je profiling

✅ razliku između benchmark-a i profiling-a

✅ vrste profila u Go-u

✅ CPU profil

✅ Heap profil

✅ Allocation profil

✅ Goroutine profil

✅ Block profil

✅ Mutex profil

✅ Execution Trace

---

# Profiling Concurrent Applications

## Deo #2 — CPU Profiling — Deep Dive

---

# 📚 Sadržaj

- Šta je CPU Profiling?
- Kako CPU profiler radi?
- `runtime/pprof`
- Generisanje CPU profila
- `go test` CPU profiling
- `go tool pprof`
- Top View
- List View
- Web View
- Flame Graph
- Call Graph
- Interpretacija rezultata

---

# 1. Šta Je CPU Profiling?

CPU profiling odgovara na pitanje:

```
Gde program

troši CPU vreme?
```

---

Primer problema:

Aplikacija je spora.

Benchmark pokazuje:

```
500 ms/op
```

---

CPU profil može pokazati:

```
70%

encoding/json

20%

business logic

10%

ostalo
```

---

Sada znamo gde treba optimizovati.

---

# 2. Kako CPU Profiler Radi?

Go CPU profiler koristi:

```
sampling
```

tehniku.

---

Ne meri svaku instrukciju.

---

Umesto toga,

periodično uzima uzorak:

```
trenutno aktivnog stack trace-a.
```

---

Primer:

```
Sample 1

main()
 service()
  calculate()
```

---

```
Sample 2

main()
 service()
  calculate()
```

---

Na osnovu velikog broja uzoraka dobija se statistička slika potrošnje CPU-a.

---

# 3. Zašto Sampling?

Direktno merenje svake instrukcije bi bilo preskupo.

---

Sampling omogućava:

- mali overhead
- realne rezultate
- korišćenje u produkciji

---

Cena:

```
nije 100% precizna

već statistička procena.
```

---

# 4. CPU Profiling pomoću `runtime/pprof`

Primer:

```go
package main

import (
	"os"
	"runtime/pprof"
)

func main() {

	f, err := os.Create("cpu.prof")
	if err != nil {
		panic(err)
	}

	pprof.StartCPUProfile(f)

	defer pprof.StopCPUProfile()

	runApplication()
}
```

---

Tok:

```
StartCPUProfile()

↓

program radi

↓

StopCPUProfile()

↓

cpu.prof
```

---

# 5. CPU Profiling u Testovima

Za benchmark-e i testove često se koristi:

```bash
go test \
	-cpuprofile=cpu.prof
```

---

Primer:

```bash
go test -bench=. \
-cpuprofile=cpu.prof
```

---

Dobijamo:

```
cpu.prof
```

koji možemo analizirati.

---

# 6. Otvaranje CPU Profila

Komanda:

```bash
go tool pprof cpu.prof
```

---

Dobijamo interaktivni shell:

```text
(pprof)
```

---

Najvažnije komande:

```
top

list

web

svg

pdf
```

---

# 7. Top View

Komanda:

```text
top
```

---

Prikazuje funkcije koje troše najviše CPU vremena.

---

Primer:

```text
Showing nodes accounting for 90%

flat   flat%
400ms 40%   parseJSON

250ms 25%   encrypt

100ms 10%   calculate
```

---

Značenje:

```
parseJSON

je najveći CPU potrošač.
```

---

# 8. Flat vs Cum

Jedan od najvažnijih koncepata.

---

## Flat

Vreme provedeno direktno u funkciji.

---

Primer:

```
parse()

100ms
```

---

## Cum

Ukupno vreme:

```
funkcija

+

sve funkcije koje poziva
```

---

Primer:

```
handleRequest()

500ms

↓

parse()

100ms
```

---

`handleRequest` ima:

```
cum = 500ms
flat = malo
```

---

# 9. List View

Komanda:

```text
list FunctionName
```

---

Prikazuje:

- linije koda
- CPU vreme po liniji

---

Primer:

```text
45% calculate.go:25

for i := range data {
	process(data[i])
}
```

---

Možemo videti tačno gde se vreme troši.

---

# 10. Web View

Komanda:

```text
web
```

---

Generiše vizuelni graf poziva.

---

Prikazuje:

```
main

↓

handler

↓

service

↓

database
```

---

Deblje grane predstavljaju:

```
veću CPU potrošnju.
```

---

# 11. Flame Graph

Jedan od najpopularnijih načina prikaza.

---

Izgleda približno:

```
main
████████████████

service
██████████

parse
██████

decode
███
```

---

Širina bloka predstavlja:

```
CPU vreme.
```

---

Veliki blokovi su kandidati za optimizaciju.

---

# 12. Kako Čitati Flame Graph?

Pravila:

---

Traži:

```
široke blokove
```

---

Ne traži:

```
najdublje blokove
```

---

Dubina pokazuje:

```
call stack
```

---

Širina pokazuje:

```
potrošnju vremena.
```

---

# 13. CPU Profiling Kod Concurrency Sistema

Kod konkurentnih programa CPU profil može otkriti:

- goroutine koje troše CPU
- previše aktivne workere
- skupe funkcije u pipeline-u
- nepotrebne serijalizacije

---

Primer:

```
Worker Pool

↓

80% CPU

↓

serialization
```

---

Optimizacija treba da bude tamo gde je CPU potrošen.

---

# 14. Tipični CPU Problemi

CPU profil često otkriva:

❌ JSON/XML overhead

❌ nepotrebne konverzije

❌ velike petlje

❌ skupe regularne izraze

❌ kopiranje velikih struktura

❌ kriptografske operacije

---

# 15. Najčešće Greške

❌ Optimizovanje funkcije sa malim CPU udelom.

---

❌ Gledanje samo jedne linije koda.

---

❌ Ignorisanje `cum` vremena.

---

❌ Zaključivanje iz jednog malog profila.

---

❌ Profilisanje nereprezentativnog scenarija.

---

# 16. Senior Workflow

Ispravan redosled:

```
Problem

↓

Benchmark

↓

CPU Profile

↓

Top View

↓

Flame Graph

↓

Identifikacija funkcije

↓

Optimizacija

↓

Ponovni benchmark
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ CPU profiling koncept

✅ sampling

✅ `runtime/pprof`

✅ generisanje CPU profila

✅ `go tool pprof`

✅ Top View

✅ Flat vs Cum vreme

✅ List View

✅ Flame Graph

✅ Call Graph analiza

---

# Profiling Concurrent Applications

## Deo #3 — Memory Profiling i GC Analysis

---

# 📚 Sadržaj

- Zašto Memory Profiling?
- Heap Profiling
- Allocation Profiling
- In-use vs Alloc-space
- `runtime/pprof` Memory API
- GC Pressure
- Escape Analysis i alokacije
- Memory Leak analiza
- Optimizing Allocations
- Production smernice

---

# 1. Zašto Memory Profiling?

CPU profil pokazuje:

```
gde CPU radi.
```

---

Ali ne odgovara na pitanje:

```
Zašto Garbage Collector

radi previše?
```

---

Problemi sa memorijom često izgledaju ovako:

- aplikacija koristi sve više RAM-a
- GC se pokreće prečesto
- latency raste
- throughput opada
- veliki broj kratkotrajnih objekata nastaje

---

Memory profiling pomaže da pronađemo:

```
ko

gde

i koliko

alokira memoriju.
```

---

# 2. Heap Profiling

Heap profil prikazuje stanje memorije koju Go runtime trenutno koristi.

---

Odgovara na pitanje:

```
Ko trenutno drži memoriju?
```

---

Primer:

```
Heap

↓

UserService

500 MB

↓

Cache

300 MB

↓

Buffer

150 MB
```

---

Najčešće se koristi za pronalaženje:

- memory leak-ova
- velikih objekata
- nepotrebnog zadržavanja referenci

---

# 3. Allocation Profiling

Allocation profil posmatra:

```
sve alokacije

tokom vremena.
```

---

Primer:

Aplikacija ima:

```
Heap = 100 MB
```

---

Ali tokom rada:

```
10 GB

alokacija
```

---

To znači:

mnogo objekata je kreirano i oslobođeno.

---

Posledica:

```
GC workload ↑
```

---

# 4. In-Use vs Alloc-Space

Jedan od najvažnijih koncepata.

---

## In-Use Space

Pokazuje:

```
memoriju koja je još uvek zauzeta.
```

---

Koristi se za:

- memory leak
- prevelike cache-e
- objekte koji se dugo drže

---

Primer:

```
Cache

↓

5GB

i nikada ne opada
```

---

## Alloc-Space

Pokazuje:

```
ukupno alociranu memoriju
tokom vremena.
```

---

Koristi se za:

- GC pressure
- previše privremenih objekata

---

# 5. Primer Razlike

Scenario:

```
Request Handler
```

kreira:

```
100 MB
```

privremenih objekata.

---

Nakon završetka:

```
GC očisti memoriju.
```

---

Rezultat:

```
In-use:

malo
```

---

Ali:

```
Alloc-space:

veliko
```

---

To znači:

nema leak-a,

ali postoji veliki allocation overhead.

---

# 6. Generisanje Memory Profila

Kod testova:

```bash
go test \
	-memprofile=mem.prof
```

---

Analiza:

```bash
go tool pprof mem.prof
```

---

Za servere često se koristi:

```go
import _ "net/http/pprof"
```

---

zatim:

```
/debug/pprof/heap
```

---

# 7. `runtime/pprof` Memory Primer

Primer:

```go
f, err := os.Create("heap.prof")

if err != nil {
	panic(err)
}

pprof.WriteHeapProfile(f)
```

---

Rezultat:

```
heap.prof
```

---

koji se analizira pomoću:

```bash
go tool pprof heap.prof
```

---

# 8. GC Pressure

Garbage Collector radi kada postoji potreba za oslobađanjem memorije.

---

Veliki broj alokacija znači:

```
više posla za GC.
```

---

Primer:

```
1000 request/s

×

1000 objekata/request
```

---

Rezultat:

```
1 000 000 objekata/s
```

---

Čak i ako su objekti mali,

GC može postati bottleneck.

---

# 9. GC Pressure Kod Concurrency Sistema

Konkurentne aplikacije često stvaraju mnogo kratkotrajnih objekata.

---

Primer:

Worker:

```
Receive message

↓

Decode

↓

Create objects

↓

Process

↓

Discard
```

---

Ako se ovo ponavlja hiljadama puta,

alokacije postaju značajan trošak.

---

# 10. Escape Analysis i Memory Profiling

Escape analysis odlučuje:

```
stack

ili

heap
```

alokaciju.

---

Primer:

```go
func createUser() *User {
	u := User{}
	return &u
}
```

---

`u` mora preći na:

```
heap
```

---

Veći broj heap alokacija znači:

```
više GC rada.
```

---

Komanda:

```bash
go test -gcflags="-m"
```

pomaže u analizi.

---

# 11. Tipični Memory Problemi

Memory profil često otkriva:

❌ nepotrebne slice kopije

❌ velike privremene strukture

❌ zaboravljene reference

❌ prevelike cache-e

❌ string konverzije

❌ previše interface boxing-a

❌ nepotrebne heap alokacije

---

# 12. Memory Leak U Go-u

Go nema manualni `free()`.

---

Ali memory leak i dalje postoji.

---

Primer:

```go
var cache = map[string][]byte{}
```

---

Ako se nikada ne uklanjaju elementi:

```
map raste zauvek.
```

---

GC ne može osloboditi memoriju

dok postoji referenca.

---

# 13. Goroutine Leak i Memorija

Poseban slučaj:

```
goroutine leak
```

---

Primer:

```go
func worker(ch <-chan int) {

	for v := range ch {
		process(v)
	}
}
```

---

Ako channel nikada nije zatvoren,

goroutine ostaje aktivna zauvek.

---

Posledice:

- stack memorija
- reference
- objekti koje drži

ostaju živi.

---

# 14. Optimizacija Alokacija

Najčešće tehnike:

✔️ reuse objekata

---

✔️ smanjiti nepotrebne kopije

---

✔️ koristiti prealokaciju:

```go
make([]T, 0, capacity)
```

---

✔️ koristiti `sync.Pool` kada ima smisla

---

✔️ izbegavati nepotrebne konverzije

---

# 15. `sync.Pool` Napomena

`sync.Pool` može smanjiti broj alokacija.

---

Primer:

```
Buffer

↓

reuse

↓

manje GC rada
```

---

Ali:

ne koristiti ga automatski.

---

Pogrešna upotreba može:

- povećati kompleksnost
- otežati razumevanje koda
- dati minimalnu korist

---

# 16. Memory Profiling Workflow

```
Problem sa memorijom

↓

Heap Profile

↓

In-use analiza

↓

Allocation analiza

↓

Identifikacija funkcije

↓

Optimizacija

↓

Ponovno merenje
```

---

# 17. Senior Pravila

✔️ Visok RAM usage nije automatski leak.

---

✔️ Veliki broj alokacija nije isto što i veliki heap.

---

✔️ GC problem često počinje kao allocation problem.

---

✔️ Prvo pronađi ko alocira.

---

✔️ Tek onda optimizuj.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Heap Profiling

✅ Allocation Profiling

✅ In-use vs Alloc-space

✅ Memory leak analiza

✅ GC Pressure

✅ Escape Analysis povezanost

✅ Goroutine leak uticaj

✅ Optimizaciju alokacija

---

# Profiling Concurrent Applications

## Deo #4 — Goroutine, Block i Mutex Profiling

---

# 📚 Sadržaj

- Goroutine Profiling
- Goroutine Lifecycle
- Goroutine Leak Detection
- Deadlock Analysis
- Block Profiling
- Mutex Profiling
- Scheduler Waiting
- Contention Analysis
- Production Debugging Workflow

---

# 1. Zašto Profilisati Goroutine-e?

Go aplikacije često imaju:

- hiljade goroutine-a
- worker pool-ove
- background task-ove
- network handler-e
- pipeline komponente

---

Veliki broj goroutine-a sam po sebi nije problem.

---

Problem nastaje kada:

- goroutine ostanu aktivne zauvek
- čekaju bez razloga
- blokiraju druge
- drže memoriju

---

# 2. Goroutine Profile

Goroutine profil prikazuje:

```
trenutno stanje

svih goroutine-a.
```

---

Može pokazati:

- gde goroutine čekaju
- koji stack trace imaju
- koliko ih postoji
- da li postoje obrasci blokiranja

---

Primer:

```
goroutine 12543

chan receive

worker.go:45
```

---

Znači:

goroutine čeka podatak sa channel-a.

---

# 3. Goroutine Lifecycle

Svaka goroutine prolazi kroz faze:

```
Created

↓

Runnable

↓

Running

↓

Waiting

↓

Finished
```

---

Problemi nastaju kada veliki broj goroutine-a ostane u:

```
Waiting
```

stanju.

---

# 4. Goroutine Leak

Goroutine leak znači:

```
goroutine je kreirana

ali nikada ne završava.
```

---

Primer:

```go
func startWorker(ch <-chan int) {
	go func() {
		for {
			value := <-ch
			process(value)
		}
	}()
}
```

---

Ako:

```
ch nikada ne dobije vrednost
```

---

goroutine ostaje blokirana zauvek.

---

# 5. Posledice Goroutine Leak-a

Goroutine leak može izazvati:

- rast memorije
- povećanje scheduler overhead-a
- zadržavanje objekata
- degradaciju performansi

---

Jedna goroutine nije problem.

---

Ali:

```
10

1000

100000
```

postaje ozbiljan problem.

---

# 6. Detekcija Goroutine Leak-a

Prvi signal:

```
broj goroutine-a konstantno raste.
```

---

Može se pratiti:

```go
runtime.NumGoroutine()
```

---

Primer:

```go
fmt.Println(
	runtime.NumGoroutine(),
)
```

---

U produkciji:

monitoring ovog broja može otkriti probleme rano.

---

# 7. Analiza Goroutine Profila

Tipičan tok:

```
Broj goroutine-a raste

↓

Uzmi goroutine profile

↓

Pronađi ponavljajući stack trace

↓

Identifikuj blokadu

↓

Popravi lifecycle
```

---

Ako hiljade goroutine-a imaju isti stack:

```
chan receive
```

verovatno postoji problem sa komunikacijom.

---

# 8. Deadlock Dijagnostika

Deadlock nastaje kada goroutine čekaju jedna drugu zauvek.

---

Primer:

```
G1

čeka G2


G2

čeka G1
```

---

Niko ne može nastaviti.

---

Simptomi:

- aplikacija "visi"
- CPU može biti nizak
- goroutine broj može izgledati normalno

---

# 9. Block Profiling

Block profil pokazuje:

```
gde goroutine provode vreme čekajući.
```

---

Najčešći izvori:

- channel send
- channel receive
- mutex lock
- `sync.Cond`
- select čekanje

---

Primer:

```
worker.go:80

chan send

500ms
```

---

To pokazuje gde postoji čekanje.

---

# 10. Omogućavanje Block Profilinga

Primer:

```go
runtime.SetBlockProfileRate(1)
```

---

Argument određuje koliko često se beleže događaji.

---

Veća vrednost:

```
manje overhead-a
```

---

Manja vrednost:

```
više detalja
```

---

# 11. Mutex Profiling

Mutex profil pokazuje:

```
gde goroutine čekaju Lock().
```

---

Primer problema:

```go
mu.Lock()

verySlowOperation()

mu.Unlock()
```

---

Kritična sekcija traje predugo.

---

Rezultat:

```
visok contention
```

---

# 12. Omogućavanje Mutex Profilinga

Primer:

```go
runtime.SetMutexProfileFraction(1)
```

---

Slično kao kod block profila:

- manji broj → više detalja
- veći broj → manji overhead

---

# 13. Block vs Mutex Profil

Važno razlikovati.

---

## Block Profile

Pita:

```
Gde goroutine čekaju?
```

---

Može uključiti:

- channel
- mutex
- condition variable

---

## Mutex Profile

Pita:

```
Gde Lock contention pravi problem?
```

---

Fokusiran je samo na mutex-e.

---

# 14. Scheduler Waiting

Goroutine može čekati zbog:

- channel-a
- lock-a
- syscall-a
- nedostatka CPU vremena

---

Profilisanje pomaže da razlikujemo:

```
nema posla

od

ima posla ali postoji blokada.
```

---

# 15. Contention Analysis Workflow

Kod sporog konkurentnog sistema:

```
Latency raste

↓

Goroutine profil

↓

Block profil

↓

Mutex profil

↓

Identifikacija čekanja

↓

Smanjenje contention-a
```

---

# 16. Tipični Problemi Koje Profil Otkriva

❌ preveliki broj worker-a

❌ jedan spor consumer

❌ zaključavanje oko sporih operacija

❌ zaboravljeni channel close

❌ goroutine koje nikada ne završavaju

❌ globalni mutex kao bottleneck

---

# 17. Production Debugging Workflow

```
Problem u produkciji

↓

Prikupi profile

↓

Uporedi sa normalnim stanjem

↓

Pronađi abnormalan obrazac

↓

Identifikuj kod

↓

Primeni minimalnu izmenu

↓

Ponovo meri
```

---

# 18. Senior Pravila

✔️ Broj goroutine-a je metrika, ne cilj.

---

✔️ Waiting goroutine nije uvek problem.

---

✔️ Traži obrasce, ne pojedinačne slučajeve.

---

✔️ Mutex contention često skriva problem dizajna.

---

✔️ Lifecycle goroutine-a mora biti eksplicitno kontrolisan.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Goroutine Profiling

✅ lifecycle goroutine-a

✅ goroutine leak detekciju

✅ deadlock analizu

✅ Block Profiling

✅ Mutex Profiling

✅ scheduler waiting

✅ contention analizu

---

# Profiling Concurrent Applications

## Deo #5 — Execution Trace i Scheduler Analysis

---

# 📚 Sadržaj

- Šta je Execution Trace?
- Razlika između Profiling-a i Trace-a
- `runtime/trace`
- Generisanje trace fajla
- `go tool trace`
- G-M-P analiza
- Scheduler događaji
- Goroutine timeline
- GC događaji
- Syscall analiza
- Pronalaženje concurrency problema

---

# 1. Šta Je Execution Trace?

Execution Trace predstavlja detaljnu vremensku mapu izvršavanja Go programa.

---

Za razliku od:

```
CPU Profile
```

koji pokazuje:

```
gde CPU troši vreme
```

---

Trace pokazuje:

```
šta se dešavalo tokom vremena.
```

---

Možemo videti:

- kada je goroutine kreirana
- kada je pokrenuta
- kada je blokirana
- kada je završila
- kada Scheduler menja izvršavanje
- kada GC radi

---

# 2. Profiling vs Trace

| Profiling | Trace |
|-----------|-------|
| Agregirani podaci | Vremenska linija |
| Ko troši resurse | Šta se dešavalo |
| Statistika | Sekvenca događaja |
| Manje detalja | Veoma detaljno |

---

Primer:

CPU profil:

```
worker()

40% CPU
```

---

Trace:

```
G1 started

↓

blocked on channel

↓

scheduled

↓

running

↓

finished
```

---

# 3. Zašto Koristiti Trace?

Trace je posebno koristan kada postoji:

- scheduler problem
- latency spike
- goroutine starvation
- neočekivano blokiranje
- loša raspodela posla

---

Kod običnog profilisanja često znamo:

```
šta je sporo
```

---

Ali ne znamo:

```
zašto se to desilo u tom trenutku.
```

---

# 4. Generisanje Trace-a

Korišćenjem:

```go
runtime/trace
```

---

Primer:

```go
package main

import (
	"os"
	"runtime/trace"
)

func main() {

	f, err := os.Create("trace.out")

	if err != nil {
		panic(err)
	}

	defer f.Close()

	trace.Start(f)

	defer trace.Stop()

	runApplication()
}
```

---

Rezultat:

```
trace.out
```

---

# 5. Trace Kod Testova

Često se koristi:

```bash
go test \
	trace=trace.out
```

---

Za benchmark:

```bash
go test \
	-bench=. \
	trace=trace.out
```

---

---

# 6. Analiza Trace-a

Pokretanje:

```bash
go tool trace trace.out
```

---

Otvara web interfejs.

---

Najvažniji prikazi:

- View trace
- Goroutine analysis
- Network blocking profile
- Synchronization blocking profile
- Scheduler latency

---

# 7. G-M-P Analiza

Trace omogućava pregled:

```
G

↓

P

↓

M
```

modela.

---

Možemo videti:

- koja goroutine radi
- koji Processor je koristi
- koji thread izvršava kod

---

Primer:

```
P1

M1

G23
```

---

# 8. Scheduler Timeline

Timeline prikazuje:

```
vreme

↓

Goroutine događaji

↓

Scheduler odluke
```

---

Možemo videti:

- previše prebacivanja
- goroutine starvation
- neiskorišćen CPU

---

# 9. Scheduler Latency

Scheduler latency znači:

```
koliko dugo goroutine čeka

pre nego što dobije CPU.
```

---

Velika latency može ukazivati na:

- CPU overload
- previše aktivnih goroutine-a
- neefikasan workload

---

# 10. Goroutine Timeline

Trace pokazuje kompletan životni ciklus.

---

Primer:

```
Create

↓

Runnable

↓

Running

↓

Blocked

↓

Runnable

↓

Finished
```

---

Ovo omogućava otkrivanje:

- nepotrebnih čekanja
- nepravilnog pipeline dizajna
- lošeg load balancing-a

---

# 11. GC Događaji

Trace prikazuje:

- početak GC ciklusa
- završetak GC-a
- STW događaje
- trajanje GC faza

---

Koristan je kada aplikacija ima:

```
latency spike
```

---

i sumnja se na:

```
Garbage Collector.
```

---

# 12. Syscall Analiza

Trace može pokazati:

- sistemske pozive
- blokiranje thread-a
- mrežne operacije

---

Primer:

```
goroutine

↓

network syscall

↓

waiting
```

---

Ovo pomaže kod I/O problema.

---

# 13. Pronalaženje Concurrency Problema

Trace može otkriti:

## Problem:

```
1000 goroutine-a čekaju
```

---

Analiza:

```
Trace

↓

channel receive

↓

jedan spor producer
```

---

Rešenje:

- promena arhitekture
- buffering
- worker pool
- paralelizacija

---

# 14. Tipični Trace Problemi

Trace često otkriva:

❌ scheduler starvation

❌ previše kratkotrajnih goroutine-a

❌ blokiranje na channel-u

❌ nebalansiran worker pool

❌ duge syscall operacije

❌ GC pauze

---

# 15. Trace Workflow

```
Latency problem

↓

Generate trace

↓

go tool trace

↓

Timeline analiza

↓

Pronađi događaj

↓

Identifikuj kod

↓

Optimizacija
```

---

# 16. Kada Koristiti Trace?

Koristi ga kada:

✔️ profil nije dovoljan

✔️ problem zavisi od vremena

✔️ postoji concurrency bug

✔️ postoji scheduler sumnja

✔️ latency nije konstantan

---

# 17. Senior Pravila

✔️ Trace je alat za kompleksne concurrency probleme.

---

✔️ Ne koristi ga kao zamenu za benchmark.

---

✔️ Prvo pronađi simptom, zatim koristi trace za uzrok.

---

✔️ Scheduler ponašanje često objašnjava misteriozne probleme.

---

✔️ Trace daje vremenski kontekst koji profil nema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ Execution Trace

✅ `runtime/trace`

✅ `go tool trace`

✅ G-M-P analiza

✅ scheduler timeline

✅ goroutine lifecycle

✅ GC događaji

✅ syscall analiza

✅ napredna concurrency dijagnostika

---

# Profiling Concurrent Applications

## Deo #6 — Production Profiling Strategy i Final Checklist

---

# 📚 Sadržaj

- Profilisanje u produkciji
- `net/http/pprof`
- Bezbednost pprof endpoint-a
- Continuous Profiling
- Monitoring strategija
- Kombinovanje profila
- Production Workflow
- Final Checklist
- Senior Profiling Mindset

---

# 1. Profilisanje u Produkciji

Profilisanje nije samo razvojni alat.

---

Kod kompleksnih Go sistema,

najvažniji problemi se često pojavljuju tek u produkciji.

---

Razlozi:

- realan workload
- veći broj korisnika
- drugačiji obrasci korišćenja
- veći broj goroutine-a
- drugačija raspodela podataka

---

Zato ozbiljne Go aplikacije imaju:

```
development profiling

+

production profiling
```

---

# 2. `net/http/pprof`

Go omogućava HTTP pristup profilima.

---

Najčešći način:

```go
import _ "net/http/pprof"
```

---

Pokretanje servera:

```go
http.ListenAndServe(
	":6060",
	nil,
)
```

---

Dobijamo:

```
/debug/pprof/
```

---

Dostupni profili:

```
/heap

/profile

/goroutine

/block

/mutex

/trace
```

---

# 3. CPU Profil Preko HTTP-a

Endpoint:

```
/debug/pprof/profile
```

---

Podrazumevano traje:

```
30 sekundi
```

---

Tok:

```
HTTP Request

↓

CPU sampling

↓

profile data

↓

pprof analiza
```

---

Primer:

```bash
go tool pprof \
http://server/debug/pprof/profile
```

---

# 4. Heap Profil Preko HTTP-a

Endpoint:

```
/debug/pprof/heap
```

---

Koristi se za:

- memory leak
- rast memorije
- GC probleme

---

Primer:

```bash
go tool pprof \
http://server/debug/pprof/heap
```

---

# 5. Bezbednost pprof-a

Nikada ne izlagati:

```
/debug/pprof
```

direktno javnom internetu.

---

Rizici:

- otkrivanje strukture aplikacije
- informacije o memoriji
- informacije o izvršavanju
- potencijalni performance impact

---

Bolje:

✔️ interni port

✔️ authentication

✔️ VPN

✔️ firewall pravila

✔️ admin-only pristup

---

# 6. Continuous Profiling

Tradicionalni profiling:

```
problem

↓

pokreni profil

↓

analiza
```

---

Continuous profiling:

```
stalno prikupljanje

↓

istorija performansi

↓

detekcija regresija
```

---

Omogućava:

- poređenje verzija
- pronalaženje degradacija
- dugoročni trend

---

# 7. Koje Profile Pratiti?

Za produkciju:

## CPU

Prati:

- CPU hotspots
- promene u workload-u

---

## Heap

Prati:

- rast memorije
- promene alokacija

---

## Goroutine

Prati:

- broj goroutine-a
- moguće leak-ove

---

## Mutex

Prati:

- contention

---

## Block

Prati:

- čekanja

---

# 8. Kombinovanje Profila

Najbolji rezultati dolaze iz kombinacije.

---

Primer:

Problem:

```
Latency raste
```

---

Analiza:

CPU:

```
normalan
```

---

Heap:

```
veliki broj alokacija
```

---

Trace:

```
GC pauze
```

---

Zaključak:

```
nije CPU problem

već memory pressure.
```

---

# 9. Production Investigation Workflow

Profesionalni tok:

```
Alert

↓

Definiši simptom

↓

Izaberi odgovarajući profil

↓

Prikupi podatke

↓

Analiziraj

↓

Identifikuj root cause

↓

Promena koda

↓

Benchmark

↓

Deploy

↓

Monitor
```

---

# 10. Profiling Checklist

Pre analize proveriti:

## CPU Problem

```
go tool pprof

↓

CPU profile
```

---

Pitanje:

```
Ko troši CPU?
```

---

## Memory Problem

```
Heap profile
```

---

Pitanje:

```
Ko drži memoriju?
```

---

## Concurrency Problem

```
Goroutine

+

Block

+

Mutex
```

---

Pitanje:

```
Ko čeka i zašto?
```

---

## Scheduler Problem

```
Execution Trace
```

---

Pitanje:

```
Kako runtime raspoređuje posao?
```

---

# 11. Najčešći Production Problemi

Profiling često otkriva:

---

## Goroutine Leak

Simptom:

```
broj goroutine-a raste
```

---

## Memory Leak

Simptom:

```
Heap konstantno raste
```

---

## Lock Contention

Simptom:

```
CPU nizak

latency visok
```

---

## GC Pressure

Simptom:

```
česte GC aktivnosti
```

---

## Scheduler Overhead

Simptom:

```
mnogo malih goroutine taskova
```

---

# 12. Finalni Performance Workflow

Kompletan proces:

```
Measure

↓

Benchmark

↓

Profile

↓

Understand

↓

Optimize

↓

Verify

↓

Monitor
```

---

Ovo je standardni pristup u velikim sistemima.

---

# 13. Senior Profiling Mentalni Model

Junior pitanje:

```
Kako da ubrzam kod?
```

---

Senior pitanje:

```
Koji resurs je bottleneck?
```

---

Mogući resursi:

```
CPU

Memory

GC

Lock

Channel

Scheduler

I/O
```

---

Optimizacija počinje tek kada znamo odgovor.

---

# 14. Pravila Profesionalnog Profilisanja

✔️ Ne optimizuj bez merenja.

---

✔️ Ne veruj intuiciji više nego podacima.

---

✔️ Različiti problemi zahtevaju različite profile.

---

✔️ Uvek meri rezultat promene.

---

✔️ Performance nije jedna metrika.

---

# 15. Završni Pregled Modula

U ovom modulu naučili smo:

## CPU Profiling

✅ sampling

✅ pprof

✅ flame graph

✅ call graph analiza

---

## Memory Profiling

✅ heap

✅ allocation

✅ GC pressure

✅ memory leak analiza

---

## Concurrency Profiling

✅ goroutine profil

✅ block profil

✅ mutex profil

---

## Runtime Analysis

✅ execution trace

✅ scheduler analiza

✅ G-M-P model

---

## Production

✅ pprof deployment

✅ continuous profiling

✅ performance workflow

---

# 16. Šta Sada Znaš?

Nakon ovog modula možeš:

- analizirati spore Go aplikacije
- pronaći CPU bottleneck
- pronaći memory probleme
- otkriti goroutine leak
- analizirati scheduler ponašanje
- razumeti contention probleme
- donositi performance odluke na osnovu podataka

---

### ➡️ Sledeća lekcija **[**Concurrency Testing Strategies**](10-concurrency-testing-strategies.md)**

Obuhvatiće:

- race detector
- deterministic testing
- testing goroutine-a
- testing channel komunikacije
- stress testing
- fuzz testing concurrency koda
- deadlock testing
- flaky test prevention
- production-grade concurrency test strategije