# 01. Introduction

> **Go Mastery**<br/>
> **Basic → 01 Getting Started**<br/>
> Poglavlje: **01 Introduction**

---

# Dobro došli u Go Mastery

Dobro došli u **Go Mastery**, sveobuhvatan tutorijal namenjen učenju programskog jezika **Go (Golang)** od samih osnova pa sve do naprednih tema koje se koriste u profesionalnom razvoju softvera.

Cilj ovog tutorijala nije samo da vas nauči **kako** da pišete Go kod, već i da razumete **zašto** je Go dizajniran upravo na način na koji jeste.

Mnogi kursevi objašnjavaju sintaksu jezika, ali vrlo malo njih objašnjava filozofiju jezika, razloge iza pojedinih odluka i način razmišljanja koji se očekuje od Go programera.

Upravo zbog toga je nastao **Go Mastery**.

---

# Cilj ovog tutorijala

Ovaj tutorijal ima nekoliko osnovnih ciljeva.

* Naučiti sintaksu Go jezika.
* Razumeti kako Go funkcioniše "ispod haube".
* Naučiti idiomatski ("Go way") stil pisanja koda.
* Naučiti kako razmišljaju iskusni Go programeri.
* Razviti naviku pisanja jednostavnog, čitljivog i održivog koda.
* Razumeti performanse i efikasnost Go programa.
* Naučiti najbolje prakse koje se koriste u produkcionim sistemima.

Na kraju tutorijala ne bi trebalo samo da znate **šta** Go radi, već i **zašto** ga profesionalni timovi koriste za razvoj velikih sistema.

---

# Šta je Go?

**Go**, poznat i kao **Golang**, predstavlja moderan, kompajlirani programski jezik opšte namene.

Go je razvijen sa idejom da kombinuje najbolje karakteristike nekoliko različitih jezika:

* jednostavnost jezika C,
* brzinu kompajliranja,
* sigurnost modernih jezika,
* ugrađenu podršku za konkurentno programiranje,
* odlične performanse.

Rezultat je jezik koji je dovoljno jednostavan da ga početnici brzo nauče, ali dovoljno moćan da pokreće neke od najvećih cloud sistema na svetu.

---

# Jedna od osnovnih ideja Go jezika

Go nije napravljen da bude jezik sa najvećim brojem mogućnosti.

Naprotiv.

Jedna od osnovnih ideja Go jezika jeste:

> **"Manje mogućnosti, ali bolje definisane mogućnosti."**

Drugim rečima:

* manje magije,
* manje skrivenog ponašanja,
* manje komplikovanih pravila,
* više jednostavnosti,
* više čitljivosti,
* više konzistentnosti.

Ovo je filozofija koja će vas pratiti tokom celog tutorijala.

---

# Za koga je namenjen Go?

Go je namenjen veoma širokom spektru programera.

Na primer:

| Iskustvo                            | Da li je tutorijal pogodan? |
| ----------------------------------- | --------------------------- |
| Potpuni početnik                    | ✅ Da                        |
| Student                             | ✅ Da                        |
| Junior Developer                    | ✅ Da                        |
| Medior Developer                    | ✅ Da                        |
| Senior Developer koji prelazi na Go | ✅ Da                        |

Predznanje programiranja jeste korisno, ali nije neophodno za razumevanje osnovnih poglavlja.

---

# Šta ćete naučiti?

Tokom ovog tutorijala obrađivaćemo veliki broj tema.

Na primer:

* promenljive,
* tipove podataka,
* pokazivače (Pointers),
* funkcije,
* metode,
* strukture,
* interfejse,
* pakete,
* module,
* rad sa greškama,
* konkurentno programiranje,
* standardnu biblioteku,
* testiranje,
* performanse,
* upravljanje memorijom,
* Garbage Collector,
* Escape Analysis,
* Generics,
* Reflection,
* i mnoge druge profesionalne teme.

Nemojte da vas ova lista uplaši.

Sve teme će biti obrađene postepeno, korak po korak.

---

# Kako koristiti ovaj tutorijal?

Tutorijal je organizovan tako da svako poglavlje predstavlja logičnu celinu.

Preporučeni način učenja je:

1. Pročitati teoriju.
2. Razumeti primere.
3. Iskucati svaki primer ručno.
4. Eksperimentisati menjanjem koda.
5. Rešiti vežbe.
6. Tek nakon toga preći na sledeće poglavlje.

> **Tip**
>
> Nemojte kopirati kod bez razmišljanja. Ručno kucanje primera značajno poboljšava razumevanje jezika.

---

# Kako čitati primere koda?

Gotovo svako poglavlje sadržaće veliki broj primera.

Primeri neće biti napisani samo da bi pokazali sintaksu.

Svaki primer će imati jasno definisanu svrhu.

Na primer:

* demonstraciju određene funkcionalnosti,
* prikaz najčešće greške,
* prikaz najboljeg rešenja,
* poređenje dva različita pristupa,
* analizu performansi.

Na taj način nećete učiti samo sintaksu, već ćete naučiti kako se Go koristi u stvarnim projektima.

---

# Filozofija ovog tutorijala

Ovaj tutorijal nije zamišljen kao zbirka kratkih primera.

Umesto toga, svako poglavlje gradi znanje stepenik po stepenik.

Nove teme će se uvek oslanjati na prethodno naučeno gradivo.

Na taj način ćete razviti stabilno razumevanje jezika, umesto površnog poznavanja pojedinačnih funkcionalnosti.

---

> **Napomena**
>
> U narednom delu upoznaćemo se sa istorijom nastanka Go jezika, problemima koje je trebalo da reši i razlozima zbog kojih je danas jedan od najpopularnijih jezika za razvoj cloud aplikacija, mikroservisa i distribuiranih sistema.

---

# Istorija nastanka Go jezika

Da bismo razumeli Go, potrebno je prvo razumeti **zašto je nastao**.

Programski jezici ne nastaju slučajno.

Svaki uspešan programski jezik nastao je kao odgovor na određene probleme koje postojeći jezici nisu dovoljno dobro rešavali.

Go nije izuzetak.

Go je nastao iz potrebe da se pronađe balans između:

- brzine razvoja softvera,
- performansi aplikacija,
- jednostavnosti jezika,
- skalabilnosti sistema,
- lakog održavanja velikih projekata.

---

# Nastanak Go jezika

Go je nastao u kompaniji **Google**.

Projekat je započet tokom 2007. godine, a javno je predstavljen 2009. godine.

Autori jezika su:

- **Robert Griesemer**
- **Rob Pike**
- **Ken Thompson**

Ova tri imena imaju veoma važnu ulogu u istoriji računarskih nauka.

---

## Robert Griesemer

Robert Griesemer je radio na implementaciji programskih jezika i kompajlera.

Njegovo iskustvo je bilo važno za:

- dizajn jezika,
- type system,
- implementaciju kompajlera,
- performanse izvršavanja programa.

---

## Rob Pike

Rob Pike je jedan od poznatih inženjera kompanije Bell Labs.

Radio je na mnogim važnim projektima:

- Unix,
- Plan 9,
- UTF-8 standard,
- programski jezici i sistemi.

Njegov uticaj na Go filozofiju vidi se kroz insistiranje na:

- jednostavnosti,
- čitljivosti,
- malom broju pravila,
- jasnom dizajnu.

---

## Ken Thompson

Ken Thompson je jedno od najpoznatijih imena u istoriji računarstva.

Poznat je kao:

- jedan od kreatora Unix operativnog sistema,
- koautor programskog jezika C,
- pionir u oblasti sistemskog programiranja.

Njegovo iskustvo direktno je uticalo na Go dizajn.

Go je nasledio mnoge Unix filozofije:

- jednostavni alati,
- jasna odgovornost,
- kombinovanje malih komponenti,
- fokus na praktičnost.

---

# Problem koji je Go pokušao da reši

Početkom 2000-ih veliki softverski sistemi postajali su sve kompleksniji.

Kompanije poput Google-a imale su ogromne codebase-ove koji su sadržali milione linija koda.

Tradicionalni jezici tog vremena imali su određene probleme.

---

# Problem 1: Kompleksnost velikih C++ projekata

C++ je veoma moćan programski jezik.

Međutim, velika fleksibilnost donosi i kompleksnost.

Veliki C++ projekti često su imali probleme kao što su:

- dugo vreme kompajliranja,
- kompleksni build sistemi,
- teško razumevanje tuđeg koda,
- veliki broj mogućih načina za rešavanje istog problema.

Primer:

U C++ postoji veliki broj funkcionalnosti:

```cpp
- classes
- inheritance
- templates
- operator overloading
- macros
- exceptions
- multiple inheritance
- metaprogramming
````

Sve ove mogućnosti mogu biti korisne.

Ali u velikim timovima predstavljaju i izazov.

Dva programera mogu napisati potpuno različita rešenja za isti problem.

---

# Problem 2: Nedovoljna efikasnost dinamičkih jezika

Drugi popularni jezici tog perioda bili su:

* Python,
* Ruby,
* JavaScript.

Oni su omogućili veoma brz razvoj aplikacija.

Njihova prednost:

* jednostavna sintaksa,
* brza iteracija,
* manje koda.

Međutim, često su imali slabije performanse u odnosu na kompajlirane jezike.

Primer:

Python kod:

```python
numbers = [1, 2, 3, 4, 5]

total = sum(numbers)

print(total)
```

Veoma jednostavno.

Ali za određene tipove velikih sistema potrebno je:

* efikasnije korišćenje memorije,
* brže izvršavanje,
* bolja paralelizacija.

---

# Problem 3: Konkurentno programiranje

Moderni sistemi zahtevaju obradu velikog broja zadataka istovremeno.

Primeri:

* web serveri,
* API sistemi,
* baze podataka,
* cloud infrastruktura,
* distribuirani sistemi.

Tradicionalni model:

```
Request
   |
   v
Thread
   |
   v
Processing
```

Problem:

Kreiranje velikog broja thread-ova može biti skupo.

Svaki thread zahteva:

* memoriju,
* scheduling,
* komunikaciju sa operativnim sistemom.

---

# Go pristup konkurentnosti

Go je uveo jednostavan model:

```
Goroutine
    |
    |
    v
Go Scheduler
    |
    |
    v
Operating System Threads
```

Goroutine je lagani izvršni kontekst kojim upravlja Go runtime.

Umesto da programer direktno upravlja thread-ovima:

```go
thread.Create()
thread.Start()
thread.Stop()
```

Go omogućava:

```go
go processRequest()
```

Jedna ključna reč:

```go
go
```

pokreće funkciju konkurentno.

---

# Filozofija dizajna

Jedna od najvažnijih ideja iza Go jezika jeste:

> "Dizajniraj jezik za ljude koji rade u velikim timovima na velikim sistemima."

To znači:

Go nije dizajniran samo za pisanje malih programa.

On je dizajniran za:

* servere,
* infrastrukturu,
* cloud sisteme,
* distribuirane aplikacije.

---

# Google-ovi problemi kao motivacija

Pre nastanka Go-a, Google je koristio veliki broj tehnologija.

Najčešće:

```
C++
Java
Python
JavaScript
```

Svaki jezik je imao svoje prednosti.

Ali Google je tražio jezik koji bi imao:

```
                  Performance
                       |
                       |
                       |
Simple Syntax -------- Go -------- Large Scale Systems
                       |
                       |
                 Easy Maintenance
```

Go je pokušao da pronađe sredinu između ovih zahteva.

---

# Prvi princip Go jezika

Jedan od osnovnih principa Go dizajna jeste:

## Jednostavnost je funkcionalnost

Go tim je smatrao da kompleksnost jezika direktno utiče na:

* brzinu razvoja,
* broj grešaka,
* održavanje koda.

Zbog toga Go namerno nema neke funkcionalnosti koje postoje u drugim jezicima.

Na primer:

Go nema:

* klasično nasleđivanje klasa,
* operator overloading,
* exceptions,
* implicitne konverzije tipova,
* kompleksan generički sistem (u početnim verzijama).

O ovome ćemo detaljno govoriti kasnije.

---

# Zašto je ovo važno za Go programera?

Razumevanje istorije Go jezika pomaže da razumemo odluke koje su napravljene.

Na primer:

Ako znamo da je Go napravljen za velike timove, razumemo zašto:

* formatiranje koda ima standard (`gofmt`);
* postoji mali broj načina za rešavanje problema;
* greške se eksplicitno proveravaju;
* kod treba da bude jednostavan za čitanje.

---

# Zaključak

Go nije nastao kao još jedan programski jezik.

Nastao je kao odgovor na konkretne probleme modernog softverskog razvoja:

* kompleksnost velikih codebase-ova,
* potrebu za visokim performansama,
* potrebu za jednostavnom konkurentnošću,
* potrebu za lakim održavanjem.

U sledećem delu ćemo detaljnije analizirati **probleme koje Go rešava i zašto postojeći jezici nisu bili dovoljni za određene tipove sistema**.

---

# Zašto je Go nastao?

U prethodnom delu upoznali smo se sa istorijom nastanka Go jezika i ljudima koji su učestvovali u njegovom dizajnu.

Sada ćemo detaljnije analizirati **probleme koje je Go pokušao da reši**.

Da bismo razumeli vrednost nekog programskog jezika, nije dovoljno samo gledati njegovu sintaksu.

Važno je razumeti:

- kakve probleme rešava,
- u kakvim sistemima je efikasan,
- koje kompromise pravi,
- zašto određene funkcionalnosti postoje ili ne postoje.

Go nije napravljen da zameni sve programske jezike.

On je napravljen da bude izuzetno dobar alat za određenu grupu problema.

---

# Problemi modernog softverskog razvoja

Kada je Go nastao, veliki softverski sistemi suočavali su se sa nekoliko ozbiljnih izazova.

Najvažniji problemi bili su:

1. Kompleksnost koda.
2. Sporo kompajliranje velikih projekata.
3. Teško održavanje velikih timskih projekata.
4. Kompleksno konkurentno programiranje.
5. Nedostatak balansa između performansi i produktivnosti.

---

# Problem #1: Kompleksnost programskih jezika

Jedan od najvećih problema modernog softvera jeste kompleksnost.

Kako projekti rastu, broj linija koda raste.

Primer:

```

Mali projekat:

10.000 linija koda
|
|
v

Srednji projekat:

500.000 linija koda
|
|
v

Veliki projekat:

10.000.000+ linija koda

```

Kod malog projekta programer može razumeti skoro ceo sistem.

Kod velikog sistema to više nije moguće.

Programer mora da:

- čita tuđ kod,
- razume postojeće odluke,
- održava stare funkcionalnosti,
- dodaje nove mogućnosti bez kvarenja postojećeg sistema.

Zbog toga je **čitljivost koda** postala jedna od najvažnijih osobina modernih jezika.

---

# Kompleksnost C++ jezika

C++ je jedan od najmoćnijih jezika ikada napravljenih.

On omogućava veoma preciznu kontrolu:

- memorije,
- performansi,
- hardvera.

Međutim, ta moć dolazi sa cenom.

C++ programer mora razumeti veliki broj koncepata:

```

Classes
Inheritance
Templates
Pointers
References
Memory Management
Operator Overloading
Macros
RAII
Move Semantics
Metaprogramming

````

Za male projekte ovo nije problem.

Za velike timove može postati izazov.

---

# Primer: više načina za isti problem

U kompleksnim jezicima često postoji veliki broj načina za rešavanje istog problema.

Na primer, u objektno orijentisanim jezicima isti koncept može biti implementiran pomoću:

- nasleđivanja,
- kompozicije,
- apstraktnih klasa,
- interfejsa,
- generičkog koda,
- šablona.

Rezultat:

Dva programera mogu napisati potpuno različit kod.

Primer:

```text
Programer A:

BaseClass
    |
    |
    +---- ChildClass


Programer B:

Interface
    |
    |
    +---- Implementation
````

Oba pristupa mogu raditi.

Ali kod velikih timova postavlja se pitanje:

> Koji stil treba koristiti?

---

# Go pristup

Go pokušava da smanji broj mogućih izbora.

Ideja:

> Ako postoji jedan jednostavan i dobar način, koristi njega.

Primer:

Go ima automatsko formatiranje koda:

```bash
gofmt
```

Umesto rasprave:

* gde staviti zagradu,
* koliko prostora koristiti,
* koji stil formatiranja koristiti,

Go definiše standard.

Rezultat:

Sav Go kod izgleda slično.

---

# Problem #2: Brzina kompajliranja

Veliki softverski sistemi često imaju problem sa vremenom kompajliranja.

Kod velikih C++ projekata kompajliranje može trajati:

```
Mala promena

       |
       v

Build system

       |
       v

Kompajliranje zavisnosti

       |
       v

Linkovanje

       |
       v

Pokretanje testova

       |
       v

Rezultat
```

Proces može trajati nekoliko minuta ili čak duže.

---

# Zašto je brzo kompajliranje važno?

Brzo kompajliranje direktno utiče na produktivnost.

Zamislimo:

Programer napravi malu izmenu.

Proces:

```
Promena koda
      |
      v
Čekanje 20 minuta
      |
      v
Testiranje
      |
      v
Nova promena
```

Tok razvoja postaje spor.

---

# Go kompajler

Go je dizajniran sa veoma brzim kompajlerom.

Cilj:

```
Kod
 |
 |
 v

Kompajler

 |
 |
 v

Binarni program
```

uz minimalno vreme čekanja.

Go koristi jednostavan dizajn jezika kako bi omogućio:

* brzu analizu koda,
* jednostavan dependency graph,
* efikasnu kompilaciju.

---

# Problem #3: Održavanje velikih sistema

Veliki sistemi imaju jedan veliki izazov:

> Kod se mnogo češće čita nego što se piše.

Programer može napisati funkciju jednom.

Ali desetine drugih programera mogu je čitati i menjati godinama.

Zato je važno:

* jednostavno razumevanje,
* predvidljivo ponašanje,
* jasna pravila.

---

# Go filozofija održavanja

Go favorizuje:

## Eksplicitnost

Go često zahteva da programer napiše stvari jasno.

Primer:

```go
result, err := operation()

if err != nil {
    return err
}
```

Greška je vidljiva.

Nema skrivene magije.

---

## Mala količina apstrakcije

Go ne pokušava da sakrije sve detalje.

Umesto:

```
Magic Framework
        |
        |
        v

Automatic Behavior
```

češće koristi:

```
Simple Code
     |
     |
     v

Predictable Result
```

---

# Problem #4: Konkurentno programiranje

Moderni sistemi moraju obrađivati veliki broj aktivnosti istovremeno.

Primer:

Web server:

```
User 1  ----\
User 2  -----\
User 3  -------> Server
User 4  -----/
User 5  ----/
```

Svaki zahtev treba obraditi nezavisno.

---

# Tradicionalni pristup

Jedan način je:

```
Request

   |

Thread

   |

Processing
```

Problem:

Operativni sistemski thread je relativno težak resurs.

Veliki broj thread-ova može dovesti do:

* velike potrošnje memorije,
* kompleksnog scheduling-a,
* težeg upravljanja.

---

# Go konkurentni model

Go uvodi koncept **goroutine**.

Model:

```
                  Go Runtime

Goroutine 1  ----\
Goroutine 2  -----\
Goroutine 3  ------> Scheduler
Goroutine 4  -----/
Goroutine 5  ----/

                       |

                       v

              OS Threads
```

Goroutine je mnogo lakša od klasičnog thread-a.

Zbog toga Go aplikacije mogu imati veliki broj istovremenih zadataka.

---

# Problem #5: Balans između performansi i produktivnosti

Postoje dva ekstremna pristupa.

## Sistemски jezici

Primer:

```
C
C++
Rust
```

Prednosti:

* visoke performanse,
* kontrola memorije.

Mane:

* kompleksniji razvoj.

---

## Dinamički jezici

Primer:

```
Python
Ruby
JavaScript
```

Prednosti:

* brz razvoj,
* jednostavna sintaksa.

Mane:

* slabije performanse u određenim scenarijima.

---

# Go kao kompromis

Go pokušava da zauzme sredinu:

```
              Performance

                  ^
                  |
                  |
C/C++ ------------+------------- Go
                  |
                  |
                  |
Python -----------+------------>

              Productivity
```

Go pruža:

* brzinu kompajliranog jezika,
* jednostavnost modernog jezika,
* ugrađenu konkurentnost,
* automatsko upravljanje memorijom.

---

# Glavna ideja ovog poglavlja

Go nije nastao zato što su postojeći jezici bili loši.

Naprotiv.

Postojeći jezici su rešavali mnoge probleme.

Ali razvoj softvera se promenio.

Pojavili su se:

* cloud sistemi,
* mikroservisi,
* distribuirane aplikacije,
* veliki timovi,
* ogromni codebase-ovi.

Go je dizajniran upravo za takvo okruženje.

---

# Zaključak

U ovom delu smo videli da je Go nastao kao odgovor na:

* rastuću kompleksnost softvera,
* spor razvoj velikih sistema,
* potrebu za jednostavnijim održavanjem,
* zahtev za efikasnim konkurentnim programiranjem,
* potrebu za balansom između performansi i produktivnosti.

U sledećem delu analiziraćemo **glavne karakteristike Go jezika i principe dizajna koji ga razlikuju od drugih programskih jezika**.

---

# Glavne karakteristike Go jezika

U prethodnim delovima upoznali smo se sa razlozima nastanka Go jezika i problemima koje je pokušao da reši.

Sada ćemo analizirati najvažnije karakteristike koje čine Go drugačijim u odnosu na druge programske jezike.

Go nije nastao tako što je pokušao da sakupi što više funkcionalnosti.

Naprotiv.

Njegov dizajn je zasnovan na pažljivo odabranom skupu mogućnosti koje rešavaju konkretne probleme modernog softverskog razvoja.

Najvažnije karakteristike Go jezika su:

- jednostavna sintaksa,
- kompajlirani jezik,
- statički tipiziran sistem,
- brzo kompajliranje,
- automatsko upravljanje memorijom,
- ugrađena podrška za konkurentnost,
- bogata standardna biblioteka,
- jednostavan dependency management,
- cross-platform razvoj,
- fokus na čitljivost i održavanje.

---

# 1. Jednostavna sintaksa

Jedna od prvih stvari koju programeri primete kada počnu da uče Go jeste jednostavnost jezika.

Go ima relativno mali broj ključnih reči.

Poređenje:

| Jezik | Broj ključnih reči |
|---|---:|
| C | 32 |
| Java | ~50 |
| C++ | 90+ |
| Go | 25 |

Go ključne reči:

```go
break
default
func
interface
select
case
defer
go
map
struct
chan
else
goto
package
switch
const
fallthrough
if
range
type
continue
for
import
return
var
````

Mali broj ključnih reči znači:

* lakše učenje jezika,
* manje pravila za pamćenje,
* lakše čitanje tuđeg koda.

---

# Primer jednostavnosti

Go funkcija:

```go
func add(a int, b int) int {
    return a + b
}
```

Ista funkcija u objektno orijentisanom jeziku često zahteva više strukture:

```text
Class
    |
    Constructor
    |
    Method
    |
    Return value
```

Go pokušava da ukloni nepotrebnu ceremoniju.

---

# 2. Kompajlirani jezik

Go je kompajlirani programski jezik.

To znači da se izvorni kod:

```text
.go fajlovi
      |
      |
      v
 Go Compiler
      |
      |
      v
 Izvršni program
```

pretvara u nativni binarni fajl.

Primer:

```bash
go build main.go
```

Rezultat:

```text
main.go

   |

   v

main.exe
```

ili:

```text
main binary
```

---

# Prednosti kompajliranog jezika

Kompajlirani programi uglavnom imaju:

* bolje performanse,
* brže izvršavanje,
* lakše distribuiranje.

Kod Go aplikacije često možemo dobiti jedan izvršni fajl:

```text
Application

+
Dependencies

=
Single Binary
```

To je veoma korisno kod:

* Docker kontejnera,
* cloud deployment-a,
* CLI alata,
* server aplikacija.

---

# 3. Statički tipiziran jezik

Go koristi statički sistem tipova.

To znači da se veliki broj grešaka otkriva tokom kompajliranja.

Primer:

```go
package main

import "fmt"

func main() {
	var age int = 25

	fmt.Println(age)
}
```

Promena tipa:

```go
var age string = "25"
```

menja ponašanje programa.

Go kompajler proverava kompatibilnost tipova.

---

# Statička tipizacija

Proces:

```text
Developer writes code

        |

        v

Compiler checks types

        |

        v

Program execution
```

Greške se pronalaze ranije.

---

# Dinamička tipizacija

Kod dinamičkih jezika:

```text
Developer writes code

        |

        v

Program runs

        |

        v

Error appears
```

Greška može biti otkrivena tek u runtime-u.

---

# 4. Brzo kompajliranje

Jedan od glavnih ciljeva Go dizajna bio je brzina kompajliranja.

Veliki projekti često zahtevaju konstantno:

* menjanje koda,
* pokretanje testova,
* ponovno kompajliranje.

Ako kompajliranje traje dugo, razvoj postaje spor.

Go koristi jednostavan jezički dizajn kako bi kompajler mogao brzo da analizira kod.

---

# 5. Garbage Collector

Go poseduje automatsko upravljanje memorijom.

Programer ne mora ručno da oslobađa memoriju.

Primer:

U jeziku C:

```c
int *number = malloc(sizeof(int));

free(number);
```

Programer mora voditi računa o:

* alokaciji,
* oslobađanju memorije,
* memory leak problemima.

---

Go:

```go
number := 42
```

Go runtime automatski prati korišćenje memorije.

Kada objekat više nije potreban:

```text
Object

    |

    v

No references

    |

    v

Garbage Collector

    |

    v

Memory released
```

---

# 6. Ugrađena konkurentnost

Jedna od najpoznatijih karakteristika Go jezika jeste podrška za konkurentno programiranje.

Go ima:

* goroutines,
* channels,
* select statement.

Primer:

```go
go downloadFile()
```

Ova jedna linija pokreće funkciju u novoj goroutine.

---

# Tradicionalni model

```text
Thread 1
 |
 +-- Task 1


Thread 2
 |
 +-- Task 2
```

Programer mora ručno upravljati:

* thread-ovima,
* lock-ovima,
* sinhronizacijom.

---

# Go model

```text
Goroutines

  G1
  G2
  G3
  G4

       |
       v

 Go Scheduler

       |
       v

 OS Threads
```

Go runtime rešava veliki deo kompleksnosti.

---

# 7. Standardna biblioteka

Go dolazi sa veoma bogatom standardnom bibliotekom.

Nema potrebe odmah koristiti veliki broj eksternih biblioteka.

Primeri paketa:

| Paket         | Namena                         |
| ------------- | ------------------------------ |
| fmt           | formatiranje i prikaz podataka |
| net/http      | HTTP server i klijent          |
| os            | rad sa operativnim sistemom    |
| time          | rad sa vremenom                |
| strings       | rad sa stringovima             |
| encoding/json | JSON obrada                    |
| database/sql  | baze podataka                  |

---

# 8. Cross-platform razvoj

Go podržava veliki broj platformi.

Isti kod može biti kompajliran za:

```text
Linux
Windows
macOS
ARM
x86
```

Primer:

```bash
GOOS=linux GOARCH=amd64 go build
```

kreira Linux izvršni fajl.

---

# 9. Formatiranje koda kao standard

Go ima alat:

```bash
gofmt
```

koji automatski formatira kod.

Primer:

Pre:

```go
func main(){fmt.Println("Hello")}
```

Posle:

```go
func main() {
	fmt.Println("Hello")
}
```

---

# Zašto je ovo važno?

U mnogim jezicima timovi raspravljaju o:

* stilovima,
* formatiranju,
* konvencijama.

Go eliminiše veliki deo tih diskusija.

Postoji jedan standard.

---

# 10. Fokus na čitljivost

Go filozofija:

> Kod se piše jednom, ali se čita mnogo puta.

Zbog toga Go favorizuje:

* jednostavne funkcije,
* jasne nazive,
* eksplicitno ponašanje,
* male apstrakcije.

---

# Go dizajn u jednoj slici

```text
                 GO LANGUAGE


        Simple Syntax
              |
              |
              v

 Performance -------- Maintainability

              ^
              |
              |

        Concurrency
        Garbage Collector
        Standard Library
```

---

# Najvažnija poruka ovog poglavlja

Go nije pokušao da bude "najmoćniji" programski jezik.

Pokušao je da bude:

* dovoljno brz,
* dovoljno jednostavan,
* dovoljno siguran,
* dovoljno skalabilan.

Upravo ta kombinacija je razlog zbog kojeg se Go danas koristi u velikom broju profesionalnih sistema.

---

# Zaključak

U ovom delu smo upoznali glavne karakteristike Go jezika:

* jednostavan dizajn,
* kompajlirani model,
* statičku tipizaciju,
* Garbage Collector,
* goroutines,
* standardnu biblioteku,
* cross-platform mogućnosti.

U sledećem delu ćemo analizirati **gde se Go koristi u realnom svetu i koje vrste sistema se najčešće grade pomoću njega**.

---

# Gde se Go koristi u realnom svetu?

U prethodnim delovima upoznali smo se sa razlozima nastanka Go jezika i njegovim glavnim karakteristikama.

Sada ćemo analizirati jedno od najvažnijih pitanja za svakog programera koji uči novi jezik:

> **Za koje probleme je Go najbolji izbor?**

Programski jezik nije alat koji se bira samo zato što je moderan ili popularan.

Pravi izbor jezika zavisi od:

- tipa sistema koji gradimo,
- zahteva za performansama,
- kompleksnosti projekta,
- veličine tima,
- potreba za skaliranjem.

Go je posebno uspešan u oblastima gde su važni:

- pouzdanost,
- jednostavna distribucija,
- visoka konkurentnost,
- mrežno programiranje,
- cloud infrastruktura.

---

# Najvažnije oblasti primene Go jezika

Go se danas najčešće koristi za razvoj:

1. Cloud infrastrukture.
2. Mikroservisa.
3. Backend sistema.
4. DevOps alata.
5. CLI aplikacija.
6. Distribuiranih sistema.
7. Mrežnih aplikacija.
8. Sistemske infrastrukture.

---

# 1. Cloud infrastruktura

Jedna od najvećih oblasti primene Go jezika jeste cloud infrastruktura.

Moderni cloud sistemi zahtevaju aplikacije koje mogu:

- obraditi veliki broj zahteva,
- raditi 24/7,
- efikasno koristiti resurse,
- lako se distribuirati.

Go je veoma pogodan za ovakav tip sistema.

---

# Zašto je Go dobar za cloud?

Cloud aplikacije često imaju sledeće karakteristike:

```text
Veliki broj korisnika

        |

        v

Mnogo paralelnih operacija

        |

        v

Distribuirani servisi

        |

        v

Potreba za skaliranjem
````

Go pruža:

* lagane goroutine,
* odličnu mrežnu biblioteku,
* jednostavan deployment,
* male izvršne fajlove.

---

# Primer cloud servisa

Zamislimo servis za obradu korisničkih zahteva:

```text
              User Requests

                    |
                    v

              Load Balancer

                    |
                    v

        +-----------+-----------+
        |                       |
        v                       v

   Go Service A          Go Service B

        |                       |

        +-----------+-----------+

                    |

                    v

              Database
```

Svaki servis može biti mala Go aplikacija.

---

# 2. Mikroservisi

Go je jedan od najpopularnijih jezika za razvoj mikroservisa.

Mikroservis predstavlja mali, nezavisan servis koji obavlja određenu funkcionalnost.

Primer:

```text
                 E-Commerce System


        +----------------+
        | User Service   |
        +----------------+

        +----------------+
        | Order Service  |
        +----------------+

        +----------------+
        | Payment Service|
        +----------------+

        +----------------+
        | Notification   |
        +----------------+
```

Svaki servis može biti napisan kao zasebna Go aplikacija.

---

# Zašto Go odgovara mikroservisima?

Mikroservisi zahtevaju:

## 1. Male binarne fajlove

Go aplikacija može biti distribuirana kao jedan izvršni fajl.

Primer:

```text
order-service

        +
        
configuration

        +

environment variables
```

---

## 2. Brzo pokretanje

Kod serverless i container okruženja veoma je važno vreme pokretanja aplikacije.

Go programi se pokreću veoma brzo.

---

## 3. Jednostavno održavanje

Mikroservisi često imaju veliki broj malih aplikacija.

Jednostavan jezik znači:

* lakše čitanje koda,
* lakši onboarding novih programera,
* manje kompleksnosti.

---

# 3. Backend razvoj

Go je veoma popularan za backend API servise.

Primer:

```text
Frontend

   |
   |
 HTTP Request

   |
   v

Go Backend API

   |
   |
Database
```

Go se koristi za:

* REST API-je,
* GraphQL servise,
* gRPC servise,
* autentifikacione sisteme,
* poslovnu logiku.

---

# Primer jednostavnog HTTP servera

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello Go!")
}

func main() {
	http.HandleFunc("/", handler)

	http.ListenAndServe(":8080", nil)
}
```

Ovaj mali program predstavlja kompletan HTTP server.

---

# 4. DevOps alati

Go ima ogromnu ulogu u DevOps svetu.

Veliki broj modernih alata napisan je upravo u Go jeziku.

Razlozi:

* lako distribuiranje,
* rad na više platformi,
* odličan rad sa mrežom,
* jednostavan deployment.

---

# Primeri poznatih Go alata

| Alat       | Namena                     |
| ---------- | -------------------------- |
| Kubernetes | Orkestracija kontejnera    |
| Docker     | Container platforma        |
| Terraform  | Infrastructure as Code     |
| Prometheus | Monitoring sistem          |
| Helm       | Kubernetes package manager |
| Consul     | Service discovery          |

---

# Kubernetes i Go

Jedan od najpoznatijih primera jeste Kubernetes.

Kubernetes rešava problem upravljanja velikim brojem kontejnera.

Pojednostavljen prikaz:

```text
             Kubernetes


        +----------------+
        | Control Plane  |
        +----------------+

                |

                v

        +----------------+
        | Worker Nodes   |
        +----------------+

                |

                v

           Containers
```

Kubernetes koristi Go zbog:

* konkurentnosti,
* performansi,
* mrežnih mogućnosti,
* jednostavne distribucije.

---

# 5. CLI aplikacije

Go je odličan izbor za razvoj komandnih alata.

Primeri:

```bash
go build
docker run
kubectl get pods
terraform plan
```

Mnogi profesionalni alati koriste komandnu liniju.

---

# Zašto Go za CLI?

CLI aplikacije zahtevaju:

* brzo pokretanje,
* jednostavan deployment,
* rad na različitim operativnim sistemima.

Go omogućava:

```text
Source Code

     |

     v

Compile

     |

     v

Single Binary
```

Korisnik ne mora instalirati runtime okruženje.

---

# 6. Distribuirani sistemi

Distribuirani sistemi predstavljaju sisteme gde više računara sarađuje kako bi obavilo posao.

Primer:

```text
        Server A

            |
            |
            v

        Network

            |
            |
            v

        Server B

            |
            |
            v

        Server C
```

Problemi distribuiranih sistema:

* komunikacija,
* sinhronizacija,
* greške mreže,
* skaliranje.

Go ima ugrađene alate koji pomažu u rešavanju ovih problema.

---

# 7. Mrežno programiranje

Go standardna biblioteka ima odličnu podršku za mrežni razvoj.

Primeri:

* HTTP serveri,
* TCP serveri,
* UDP komunikacija,
* WebSocket sistemi,
* proxy servisi.

Paketi:

```go
net
net/http
net/url
crypto/tls
```

---

# 8. Sistemsko programiranje

Iako Go nije zamena za C ili Rust u svim oblastima, veoma dobro funkcioniše za određene sistemske zadatke.

Primeri:

* agenti,
* servisi,
* alati za infrastrukturu,
* monitoring sistemi.

---

# Kada Go nije najbolji izbor?

Važno je razumeti da Go nije univerzalno rešenje.

Postoje oblasti gde drugi jezici mogu biti bolji.

---

# 1. Mobilne aplikacije

Za:

* Android,
* iOS,

češće se koriste:

* Kotlin,
* Swift.

---

# 2. Frontend razvoj

Za browser aplikacije koriste se:

* JavaScript,
* TypeScript.

Go nije namenjen za klasičan frontend.

---

# 3. Ekstremno nisko-nivo programiranje

Za:

* kernel development,
* embedded sisteme sa ekstremnim ograničenjima,

često se koriste:

* C,
* Rust.

---

# 4. Data Science i Machine Learning

Za:

* istraživanje podataka,
* machine learning modele,

dominantni jezici su:

* Python,
* R.

---

# Go u jednoj slici

```text
                  GO


        Cloud Infrastructure
                 |
                 |
        Microservices
                 |
                 |
        Backend Systems
                 |
                 |
        DevOps Tools
                 |
                 |
        Distributed Systems
                 |
                 |
        Networking
```

---

# Zaključak

Go je posebno snažan kada gradimo:

* cloud sisteme,
* backend servise,
* mikroservise,
* infrastrukturu,
* distribuirane aplikacije,
* mrežne alate.

Njegova najveća vrednost nije u tome da bude najbolji za svaki problem.

Njegova vrednost je u tome što je izuzetno dobar za probleme modernog server-side razvoja.

U sledećem delu ćemo detaljnije analizirati **Go ekosistem, alate i razvojno okruženje koje svaki Go programer koristi**.

---

# Go ekosistem i razvojni alati

U prethodnim delovima upoznali smo se sa:

- istorijom nastanka Go jezika,
- problemima koje Go rešava,
- glavnim karakteristikama jezika,
- oblastima u kojima se Go koristi.

Međutim, programski jezik sam po sebi nije dovoljan za profesionalni razvoj softvera.

Moderni razvoj zahteva kompletan ekosistem:

- kompajler,
- build alat,
- dependency management,
- formatter,
- debugger,
- alat za testiranje,
- alat za analizu koda,
- razvojno okruženje.

Go je dizajniran tako da veliki deo tog ekosistema dolazi već uz sam jezik.

---

# Šta predstavlja Go ekosistem?

Go ekosistem možemo predstaviti kao skup alata koji zajedno omogućavaju kompletan razvojni ciklus:

```text
                  Go Ecosystem


                       Go Code

                          |
                          v

                    Go Compiler

                          |
                          v

                    Build System

                          |
                          v

              +-----------+-----------+
              |           |           |
              v           v           v

          Testing     Debugging   Analysis

              |
              |
              v

          Production Application
````

---

# Glavni delovi Go ekosistema

Najvažniji elementi su:

| Komponenta  | Namena                               |
| ----------- | ------------------------------------ |
| Go Compiler | Prevođenje Go koda u izvršni program |
| Go Command  | Glavni alat za rad sa projektima     |
| Go Modules  | Upravljanje zavisnostima             |
| gofmt       | Formatiranje koda                    |
| go test     | Pokretanje testova                   |
| go vet      | Analiza potencijalnih grešaka        |
| pprof       | Analiza performansi                  |
| Delve       | Debugger                             |
| Go Doc      | Dokumentacija                        |

---

# 1. Go Compiler

Go kompajler je alat koji prevodi Go kod u mašinski kod.

Proces izgleda ovako:

```text
main.go

   |
   |
   v

Go Compiler

   |
   |
   v

Machine Code

   |
   |
   v

Executable Binary
```

Primer:

```bash
go build
```

kreira izvršni program.

---

# Zašto je kompajler važan?

Kompajler nije samo alat koji pravi izvršni fajl.

On takođe proverava:

* sintaksu,
* tipove,
* nedostupne promenljive,
* neiskorišćene importe,
* mnoge potencijalne greške.

Primer:

```go
package main

import "fmt"

func main() {
	var message string

	fmt.Println(number)
}
```

Go kompajler prijavljuje grešku:

```text
undefined: number
```

Greška se otkriva pre pokretanja programa.

---

# 2. Go Command (`go` alat)

Jedan od najvažnijih delova Go ekosistema jeste komandni alat:

```bash
go
```

Ovaj alat predstavlja centralnu tačku za rad sa Go projektima.

---

# Najčešće korišćene komande

## Pokretanje programa

```bash
go run main.go
```

Kompajlira i pokreće program.

---

## Kompajliranje projekta

```bash
go build
```

Kreira izvršni fajl.

---

## Instalacija alata

```bash
go install
```

Instalira Go aplikacije i alate.

---

## Pokretanje testova

```bash
go test
```

Pokreće testove.

---

## Preuzimanje zavisnosti

```bash
go get
```

Dodaje ili ažurira module.

---

# Go Command kao razvojni centar

Umesto velikog broja različitih alata:

```text
Compiler
Build System
Package Manager
Testing Tool
Formatter
```

Go ih objedinjuje:

```text
              go


       +------+-------+
       |      |       |
       v      v       v

    build   test   fmt

       |
       |
       v

    Complete Workflow
```

---

# 3. Go Modules

Jedan od važnih delova modernog Go razvoja jeste upravljanje zavisnostima.

Pre Go Modules sistema, Go projekti su često koristili:

```text
GOPATH
 |
 +-- src
 |
 +-- pkg
 |
 +-- bin
```

Ovaj pristup je imao ograničenja.

---

# Go Modules pristup

Moderni Go koristi:

```text
Project

 |
 +-- go.mod
 |
 +-- go.sum
 |
 +-- source files
```

Primer:

```text
my-project/

    go.mod

    go.sum

    main.go

    user/
       user.go
```

---

# go.mod fajl

Primer:

```go
module example.com/my-project

go 1.26

require (
    github.com/example/library v1.2.0
)
```

Ovaj fajl definiše:

* naziv modula,
* Go verziju,
* spoljne zavisnosti.

---

# Zašto su Go Modules važni?

Omogućavaju:

* reproduktivne build-ove,
* kontrolu verzija,
* jednostavno deljenje projekta,
* lak deployment.

---

# 4. gofmt - standard za formatiranje

Jedna od najpoznatijih Go odluka jeste standardizovano formatiranje.

Komanda:

```bash
gofmt
```

automatski uređuje kod.

---

# Pre gofmt

```go
func main(){
fmt.Println("Hello")
}
```

---

# Posle gofmt

```go
func main() {
	fmt.Println("Hello")
}
```

---

# Zašto je gofmt važan?

U drugim jezicima često postoje rasprave:

* tab ili space?
* gde ide zagrada?
* kako formatirati import?

Go rešava ovaj problem.

Postoji jedan standard.

---

# 5. go test

Testiranje je deo standardnog Go workflow-a.

Primer:

```bash
go test ./...
```

pokreće sve testove u projektu.

---

# Go pristup testiranju

Testovi su obični Go fajlovi:

```text
calculator.go

calculator_test.go
```

Primer:

```go
func TestAdd(t *testing.T) {
	result := Add(2, 3)

	if result != 5 {
		t.Fail()
	}
}
```

---

# 6. go vet

`go vet` analizira kod i traži potencijalne probleme.

Primeri:

* sumnjivi format stringovi,
* pogrešna upotreba određenih funkcija,
* problemi sa kodom.

Pokretanje:

```bash
go vet ./...
```

---

# 7. Delve debugger

Go ima veoma kvalitetan debugger:

```text
Delve
```

On omogućava:

* breakpoint-e,
* pregled promenljivih,
* izvršavanje korak po korak,
* analizu goroutine-a.

Primer:

```bash
dlv debug
```

---

# 8. Profiling alati

Performanse su važan deo Go ekosistema.

Go dolazi sa alatima za analizu:

* CPU usage,
* memorije,
* goroutine-a.

Najpoznatiji alat:

```text
pprof
```

---

# Primer problema

Aplikacija radi sporo:

```text
Request

    |

    v

Go Service

    |

    v

Slow Function
```

Profiling omogućava pronalaženje:

* gde se troši vreme,
* gde se koristi memorija,
* gde postoje uska grla.

---

# 9. Dokumentacija

Go ima veoma jednostavan sistem dokumentacije.

Komanda:

```bash
go doc
```

omogućava pregled:

* paketa,
* funkcija,
* tipova.

Primer:

```bash
go doc fmt.Println
```

---

# Go filozofija alata

Jedna od velikih prednosti Go ekosistema jeste konzistentnost.

Tipičan workflow izgleda ovako:

```text
Write Code

    |
    v

gofmt

    |
    v

go test

    |
    v

go vet

    |
    v

go build

    |
    v

Deploy
```

---

# Minimalni profesionalni Go workflow

Svaki Go programer bi trebalo da poznaje:

```bash
go fmt
go test
go vet
go build
go run
go mod
```

Ove komande predstavljaju osnovu svakodnevnog rada.

---

# Zaključak

Go nije samo programski jezik.

On dolazi sa kompletnim ekosistemom koji pokriva:

* razvoj,
* build proces,
* testiranje,
* debugging,
* dokumentaciju,
* analizu performansi.

Upravo zbog toga Go omogućava jednostavan i profesionalan razvoj softvera bez velikog broja dodatnih alata.

U sledećem delu ćemo pogledati **kako izgleda jedan tipičan Go program i kako su organizovane osnovne komponente Go aplikacije**.

---

# Kako izgleda jedan Go program?

U prethodnom delu upoznali smo se sa Go ekosistemom i alatima koji čine svakodnevni razvojni proces.

Sada ćemo napraviti prvi korak ka samom Go kodu.

Pre nego što počnemo da učimo:

- promenljive,
- funkcije,
- strukture,
- interfejse,
- goroutine,
- pakete,

potrebno je da razumemo osnovnu strukturu jednog Go programa.

Cilj ovog poglavlja nije da detaljno objasni svaku sintaksnu konstrukciju, već da pruži **mentalni model** kako Go aplikacija izgleda.

---

# Najjednostavniji Go program

Najpoznatiji prvi primer u gotovo svakom programskom jeziku jeste:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, Go!")
}
````

Iako ovaj program ima samo nekoliko linija, on sadrži nekoliko veoma važnih koncepata:

* package,
* import,
* function,
* main function,
* standard library package.

Kasnije ćemo svaki od ovih elemenata detaljno proučiti.

---

# Anatomija Go programa

Možemo predstaviti prethodni primer ovako:

```text
                Go Program


        +-------------------+
        | package main      |
        +-------------------+

                  |

                  v

        +-------------------+
        | imports           |
        +-------------------+

                  |

                  v

        +-------------------+
        | functions         |
        +-------------------+

                  |

                  v

        +-------------------+
        | program logic     |
        +-------------------+
```

Svaki Go fajl ima određenu strukturu.

---

# 1. Package deklaracija

Svaki Go fajl počinje deklaracijom paketa.

Primer:

```go
package main
```

Package predstavlja logičku grupu Go koda.

Možemo ga posmatrati kao organizacionu jedinicu.

---

# Zašto postoje paketi?

Veliki programi ne mogu imati sav kod u jednom fajlu.

Primer:

```text
E-commerce Application


main

 |
 +-- user
 |
 +-- payment
 |
 +-- order
 |
 +-- notification
```

Svaki deo sistema može biti poseban paket.

---

# Package main

Poseban paket u Go jeziku jeste:

```go
package main
```

On označava izvršni program.

Ako paket nije `main`, Go ga tretira kao biblioteku.

Primer:

```text
Library Package

       |
       v

Other Programs Use It
```

---

# 2. Import sekcija

Nakon package deklaracije dolaze import-i.

Primer:

```go
import "fmt"
```

Import omogućava korišćenje koda iz drugih paketa.

---

# Šta je paket?

Paket je kolekcija:

* funkcija,
* tipova,
* promenljivih,
* konstanti.

Primer:

```go
fmt.Println()
```

Ovde:

```text
fmt

 |
 +-- Println()
```

`fmt` je paket.

`Println` je funkcija iz tog paketa.

---

# Višestruki import

Kada koristimo više paketa:

```go
import (
	"fmt"
	"time"
	"net/http"
)
```

Go omogućava grupisanje import-a.

---

# 3. Funkcija main

Svaki izvršni Go program mora imati:

```go
func main()
```

To je početna tačka programa.

Možemo je posmatrati kao:

```text
Operating System

        |
        v

     main()

        |
        v

 Application Logic
```

Kada pokrenemo program:

```bash
go run main.go
```

Go runtime poziva:

```go
main()
```

---

# 4. Funkcije

Go programi se sastoje od funkcija.

Primer:

```go
func greet() {
	fmt.Println("Hello")
}
```

Funkcija predstavlja blok koda koji izvršava određeni zadatak.

---

# Pozivanje funkcije

Definicija:

```go
func greet() {
	fmt.Println("Hello")
}
```

Poziv:

```go
greet()
```

Tok:

```text
main()

 |
 v

greet()

 |
 v

Print message
```

---

# 5. Izvršavanje programa

Kada pokrenemo Go program:

```bash
go run main.go
```

dešava se sledeće:

```text
        Source Code

             |

             v

        Go Compiler

             |

             v

        Executable

             |

             v

        Go Runtime

             |

             v

        main()

             |

             v

        Program Execution
```

---

# Go fajlovi

Go kod se nalazi u fajlovima sa ekstenzijom:

```text
.go
```

Primer:

```text
main.go
user.go
database.go
server.go
```

---

# Organizacija jednostavnog projekta

Mali projekat može izgledati ovako:

```text
my-app/

    go.mod

    main.go

    user.go

    database.go
```

---

# Organizacija većeg projekta

Profesionalni projekti često koriste strukturu:

```text
my-service/

    go.mod

    cmd/

        api/

            main.go

    internal/

        user/

        database/

        service/

    pkg/

        logger/
```

O ovoj temi ćemo detaljno govoriti u poglavlju:

```
02-starting-a-project.md
```

---

# Kompajliranje Go programa

Primer:

```bash
go build
```

Proces:

```text
main.go

   |

   v

Parser

   |

   v

Type Checker

   |

   v

Compiler

   |

   v

Binary File
```

---

# Go program kao jedna binarna aplikacija

Jedna od velikih prednosti Go-a jeste jednostavna distribucija.

Primer:

```text
Application Source

        |

        v

       Build

        |

        v

  my-server binary
```

Na server možemo poslati:

```text
my-server
```

i pokrenuti aplikaciju.

---

# Go Runtime

Iako je Go kompajliran jezik, programi koriste Go runtime.

Runtime obezbeđuje:

* Garbage Collector,
* scheduler za goroutine,
* upravljanje memorijom,
* određene sistemske funkcije.

Model:

```text
        Go Application

              |

              v

        Go Runtime

              |

              v

     Operating System
```

---

# Prvi mentalni model Go aplikacije

Kada gledamo Go aplikaciju, razmišljamo ovako:

```text
Application

 |
 +-- Packages

       |
       +-- Functions

              |
              +-- Logic

       |
       +-- Types

       |
       +-- Data
```

---

# Važan princip: jednostavnost strukture

Go pokušava da projekti budu laki za razumevanje.

Ne postoji veliki broj implicitnih pravila.

Kod uglavnom govori sam za sebe.

Primer:

```go
user := CreateUser()
```

Veoma je jasno:

* kreira se korisnik,
* poziva se funkcija,
* rezultat se čuva u promenljivoj.

---

# Najčešće početničke greške

## 1. Mešanje package imena

Greška:

```go
package Main
```

Ispravno:

```go
package main
```

Go razlikuje mala i velika slova.

---

## 2. Zaboravljen import

Primer:

```go
func main() {
	fmt.Println("Hello")
}
```

Bez:

```go
import "fmt"
```

program neće raditi.

---

## 3. Neiskorišćeni import

Go ne dozvoljava:

```go
import "fmt"
```

ako se `fmt` nigde ne koristi.

Ovo pomaže održavanju čistog koda.

---

# Ključne ideje ovog dela

Nakon ovog poglavlja trebalo bi da razumemo:

* svaki Go fajl pripada paketu;
* `package main` predstavlja izvršni program;
* `main()` je ulazna tačka aplikacije;
* import omogućava korišćenje drugih paketa;
* Go aplikacije se kompajliraju u izvršne fajlove;
* Go runtime upravlja važnim delovima izvršavanja.

---

# Zaključak

Jedan Go program izgleda jednostavno zato što je Go dizajniran da bude jednostavan.

Iza te jednostavnosti postoji veoma moćan sistem:

* kompajler,
* runtime,
* standardna biblioteka,
* paketni sistem,
* alati za razvoj.

U sledećem delu detaljnije ćemo analizirati **kako Go program prolazi kroz proces od izvornog koda do izvršavanja i šta se tačno događa tokom tog procesa**.

---

# Od izvornog koda do izvršnog programa

U prethodnom delu upoznali smo osnovnu strukturu jednog Go programa:

- package deklaraciju,
- import sekciju,
- funkcije,
- `main()` funkciju,
- organizaciju Go fajlova.

Sada ćemo analizirati šta se tačno dešava kada napišemo Go kod i pokrenemo aplikaciju.

Razumevanje ovog procesa je veoma važno zato što nam omogućava da bolje razumemo:

- kako kompajler radi,
- kada nastaju greške,
- zašto je Go brz,
- kako se aplikacija izvršava,
- kako Go runtime učestvuje u radu programa.

---

# Celokupan životni ciklus Go programa

Jedan Go program prolazi kroz nekoliko faza:

```text
              Source Code

                  |
                  v

             Lexical Analysis

                  |
                  v

             Parsing

                  |
                  v

            Type Checking

                  |
                  v

            Compilation

                  |
                  v

             Linking

                  |
                  v

            Executable Binary

                  |
                  v

             Program Runtime
````

Svaka faza ima svoju ulogu.

---

# 1. Pisanje izvornog koda

Sve počinje sa `.go` fajlovima.

Primer:

```text
my-project/

    main.go

    user.go

    database.go
```

Programer piše kod koristeći Go sintaksu.

Primer:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello Go")
}
```

U ovoj fazi program postoji samo kao tekst.

Računar još uvek ne može da ga izvršava.

---

# 2. Lexical Analysis (tokenizacija)

Prva faza kompajliranja jeste analiza teksta.

Kompajler pretvara kod u tokene.

Primer:

Kod:

```go
var age int = 25
```

postaje:

```text
VAR

IDENTIFIER(age)

TYPE(int)

ASSIGNMENT(=)

NUMBER(25)
```

---

# Šta je token?

Token je najmanja značajna jedinica programskog jezika.

Primeri tokena:

| Kod    | Token      |
| ------ | ---------- |
| `func` | keyword    |
| `main` | identifier |
| `(`    | delimiter  |
| `42`   | literal    |
| `+`    | operator   |

---

# 3. Parsing

Nakon tokenizacije, kompajler proverava strukturu programa.

Pita:

> Da li je kod napisan po pravilima Go jezika?

Primer:

Ispravno:

```go
func main() {

}
```

Neispravno:

```go
func main( {

}
```

Kompajler kreira strukturu poznatu kao:

```text
AST
(Abstract Syntax Tree)
```

---

# Abstract Syntax Tree (AST)

AST predstavlja strukturu programa.

Primer:

Kod:

```go
a + b
```

AST:

```text
        +

       / \

      a   b
```

Za funkciju:

```go
func main() {
	fmt.Println("Hello")
}
```

struktura izgleda približno:

```text
Function

 |
 +-- Name: main

 |
 +-- Body

       |
       +-- Call fmt.Println
```

---

# 4. Type Checking

Jedna od najvažnijih karakteristika Go jezika jeste statička provera tipova.

Kompajler proverava:

* da li promenljive imaju odgovarajuće tipove,
* da li funkcije dobijaju ispravne argumente,
* da li se vrednosti mogu dodeliti.

Primer:

```go
var age int = 30
```

Tip:

```text
age -> int
```

---

Pogrešan primer:

```go
var age int = "thirty"
```

Kompajler prijavljuje:

```text
cannot use string as int
```

Greška se pojavljuje pre izvršavanja programa.

---

# Zašto je ovo važno?

Greške koje se otkriju tokom kompajliranja su jeftinije od grešaka koje se otkriju u produkciji.

Primer:

```text
Compile Time Error

        |
        v

Developer Fixes Problem


Runtime Error

        |
        v

User Finds Problem
```

---

# 5. Compilation

Nakon uspešne analize, Go kompajler prevodi kod.

Proces:

```text
Go Source

    |

    v

Intermediate Representation

    |

    v

Machine Code
```

Rezultat je objektni kod.

---

# Go compiler

Go koristi moderan kompajler koji uključuje:

* optimizacije,
* analizu koda,
* generisanje mašinskog koda.

Cilj:

* brz build,
* efikasan program.

---

# 6. Linking

Nakon kompajliranja potrebno je spojiti sve delove programa.

Program može koristiti:

* sopstveni kod,
* standardnu biblioteku,
* eksternе pakete.

Primer:

```text
Application

    +

Standard Library

    +

External Packages

          |

          v

      Final Binary
```

Ovaj proces se naziva:

```text
Linking
```

---

# 7. Izvršni fajl

Rezultat procesa je binarni fajl.

Primer:

Linux:

```text
my-server
```

Windows:

```text
my-server.exe
```

macOS:

```text
my-server
```

Taj fajl sadrži:

* mašinski kod,
* potrebne informacije za izvršavanje,
* povezan runtime.

---

# go run vs go build

Početnici često koriste:

```bash
go run main.go
```

Ova komanda:

1. kompajlira kod,
2. kreira privremeni izvršni fajl,
3. pokrene program.

---

`go build`:

```bash
go build
```

radi samo build.

Rezultat:

```text
main.go

   |

   v

main binary
```

---

# Razlika:

```text
go run

Source Code
     |
     v
 Compile
     |
     v
 Execute


go build

Source Code
     |
     v
 Compile
     |
     v
 Binary File
```

---

# 8. Pokretanje programa

Kada se program pokrene:

```text
Executable Binary

        |

        v

Operating System

        |

        v

Go Runtime

        |

        v

main()
```

---

# Go Runtime

Go runtime je deo koji izvršava dodatne funkcije potrebne aplikaciji.

On upravlja:

* memorijom,
* Garbage Collector-om,
* goroutine scheduler-om,
* internim strukturama.

---

# Runtime model

Možemo ga predstaviti ovako:

```text
+-----------------------+
|    Go Application     |
+-----------------------+

            |

            v

+-----------------------+
|      Go Runtime       |
+-----------------------+

            |

            v

+-----------------------+
| Operating System      |
+-----------------------+
```

---

# 9. Garbage Collector tokom izvršavanja

Tokom rada programa nastaju objekti u memoriji.

Primer:

```go
user := User{
	Name: "Mark",
}
```

Memorija:

```text
Heap

+----------------+
| User Object    |
+----------------+
```

Kada objekat više nije potreban:

```text
Object

   |

   v

No References

   |

   v

Garbage Collector

   |

   v

Memory Released
```

---

# 10. Go Scheduler

Jedna od najvažnijih runtime komponenti jeste scheduler.

On upravlja goroutine-ama.

Model:

```text
Goroutines

G1
G2
G3
G4

    |

    v

Go Scheduler

    |

    v

Operating System Threads
```

Detaljno ćemo ga obrađivati u naprednim poglavljima.

---

# Zašto je Go build proces brz?

Go je dizajniran da kompajliranje bude brzo.

Razlozi:

* jednostavan jezik,
* eksplicitne zavisnosti,
* efikasan dependency graph,
* moderan kompajler.

---

# Poređenje sa tradicionalnim pristupima

Kompleksniji jezici:

```text
Large Codebase

      |

      v

Complex Dependency Graph

      |

      v

Long Compilation Time
```

Go:

```text
Go Project

      |

      v

Simple Dependencies

      |

      v

Fast Compilation
```

---

# Debugging i kompajlerske greške

Kada program ne može da se kompajlira, greška se dobija odmah.

Primer:

```go
fmt.Println(message)
```

bez deklaracije:

```go
message := "Hello"
```

Rezultat:

```text
undefined: message
```

---

# Važan mentalni model

Kao Go programer treba da razmišljate ovako:

```text
Write Code

    |

    v

Compiler Understands Code

    |

    v

Compiler Generates Binary

    |

    v

Runtime Executes Program

    |

    v

Application Runs
```

---

# Zaključak

Go aplikacija prolazi kroz jasan i predvidljiv proces:

1. Pisanje koda.
2. Lexical analiza.
3. Parsing.
4. Type checking.
5. Kompajliranje.
6. Linkovanje.
7. Kreiranje binarnog fajla.
8. Izvršavanje kroz Go runtime.

Razumevanje ovog procesa predstavlja osnovu za kasnije teme kao što su:

* Garbage Collection,
* Memory Management,
* Escape Analysis,
* Performance Optimization,
* Compiler Optimizations.

U sledećem delu ćemo analizirati **organizaciju Go projekta, pakete, module i način na koji se veliki Go sistemi strukturišu**.

---

# Organizacija Go projekta

U prethodnim delovima analizirali smo:

- strukturu jednog Go programa,
- proces od izvornog koda do izvršnog fajla,
- ulogu kompajlera i runtime-a.

Sada ćemo preći sa nivoa pojedinačnog fajla na nivo kompletnog projekta.

U realnom svetu Go aplikacije se ne sastoje od jednog fajla.

Profesionalni sistemi mogu sadržati:

- stotine paketa,
- hiljade fajlova,
- veliki broj programera.

Zbog toga je veoma važno razumeti kako se Go projekti organizuju.

---

# Šta predstavlja Go projekat?

Go projekat predstavlja skup Go fajlova koji zajedno čine jednu aplikaciju ili biblioteku.

Najjednostavniji projekat:

```text
hello-world/

    main.go
````

Veći projekat:

```text
e-commerce/

    main.go

    user.go

    product.go

    database.go

    api.go
```

Profesionalni projekti:

```text
company-service/

    cmd/

    internal/

    pkg/

    api/

    configs/

    scripts/

    tests/
```

---

# Osnovni elementi Go projekta

Tipičan Go projekat sadrži:

```text
Project

 |
 +-- go.mod

 |
 +-- Go source files

 |
 +-- Packages

 |
 +-- Tests

 |
 +-- Dependencies
```

---

# 1. go.mod fajl

Svaki moderan Go projekat koristi:

```text
go.mod
```

Ovaj fajl definiše Go modul.

Primer:

```go
module example.com/my-service

go 1.26
```

---

# Šta sadrži go.mod?

`go.mod` definiše:

* naziv modula,
* Go verziju,
* zavisnosti,
* verzije biblioteka.

Primer:

```go
module github.com/company/payment-service

go 1.26

require (
	github.com/google/uuid v1.6.0
)
```

---

# Zašto je go.mod važan?

Bez `go.mod` fajla bilo bi teško:

* deliti projekat,
* upravljati verzijama,
* reprodukovati build.

Sa modulima:

```text
Developer A

    |

    v

go.mod

    |

    v

Developer B
```

Oba programera koriste iste zavisnosti.

---

# 2. Go paketi

Osnovna jedinica organizacije Go koda jeste:

```text
Package
```

Paket predstavlja grupu povezanih fajlova.

Primer:

```text
user/

    user.go

    validation.go

    repository.go
```

Svi fajlovi mogu pripadati istom paketu:

```go
package user
```

---

# Zašto koristiti pakete?

Paketi omogućavaju:

* organizaciju koda,
* ponovno korišćenje,
* izolaciju odgovornosti.

Primer:

Bez paketa:

```text
main.go

10000 lines
```

Problem:

* teško čitanje,
* teško održavanje,
* teško testiranje.

---

Sa paketima:

```text
main

 |
 +-- user

 |
 +-- database

 |
 +-- payment

 |
 +-- notification
```

---

# 3. Main package

Kao što smo ranije videli:

```go
package main
```

ima posebno značenje.

On predstavlja izvršnu aplikaciju.

Primer:

```text
cmd/

    server/

        main.go
```

Kod:

```go
package main

func main() {

}
```

Rezultat:

```text
server binary
```

---

# 4. Bibliotečki paketi

Ne mora svaki paket biti aplikacija.

Neki paketi predstavljaju biblioteke.

Primer:

```text
math/

    calculate.go
```

Kod:

```go
package math
```

Drugi program može koristiti:

```go
import "example.com/math"
```

---

# Package naming pravila

Go paketi uglavnom koriste:

* mala slova,
* kratka imena,
* jasnu namenu.

Dobro:

```text
user
database
http
cache
auth
```

Loše:

```text
UserManagementSystem
VeryImportantDatabasePackage
HelperFunctions
```

---

# 5. Interna organizacija projekta

Go zajednica često koristi sledeću strukturu:

```text
my-service/

    go.mod

    cmd/

    internal/

    pkg/
```

Ova struktura nije obavezna.

Go ne nameće jedan jedini način organizacije.

Ali predstavlja čestu profesionalnu praksu.

---

# cmd direktorijum

`cmd` sadrži izvršne aplikacije.

Primer:

```text
cmd/

    api/

        main.go

    worker/

        main.go

    migration/

        main.go
```

---

# Zašto cmd?

Veliki sistemi često imaju više izvršnih programa.

Primer:

```text
Company System


    API Server

        |

        v

    Background Worker

        |

        v

    CLI Tool
```

Svaki može imati svoj `main` paket.

---

# internal direktorijum

`internal` sadrži kod koji je namenjen samo tom projektu.

Primer:

```text
internal/

    user/

    payment/

    database/
```

---

# Posebno pravilo internal paketa

Go compiler sprečava druge projekte da importuju:

```text
internal/
```

pakete.

Primer:

```text
my-service

    internal/user
```

Drugi projekat ne može:

```go
import "my-service/internal/user"
```

---

# Zašto postoji internal?

Da bi se zaštitila arhitektura.

Primer:

```text
Public API

        |
        v

Internal Implementation
```

Programer jasno razlikuje:

* šta je javno,
* šta je privatno.

---

# pkg direktorijum

`pkg` se često koristi za javne biblioteke.

Primer:

```text
pkg/

    logger/

    validator/

    client/
```

Ideja:

```text
Reusable Code
```

---

# Međutim...

Važno je razumeti:

Go ne zahteva:

```text
cmd/
internal/
pkg/
```

Ovo je samo konvencija.

Najvažnije pravilo je:

> Organizacija treba da olakša razumevanje koda.

---

# Primer realnog Go servisa

Jedan backend servis može izgledati ovako:

```text
payment-service/

    go.mod

    cmd/

        api/

            main.go


    internal/

        payment/

            service.go

            repository.go


        database/

            postgres.go


        auth/

            middleware.go


    pkg/

        logger/


    api/

        openapi.yaml
```

---

# Tok zavisnosti

Profesionalna arhitektura često izgleda ovako:

```text
main.go

   |

   v

service layer

   |

   v

repository layer

   |

   v

database
```

Primer:

```text
HTTP Request

     |

     v

Handler

     |

     v

Service

     |

     v

Repository

     |

     v

Database
```

---

# Go pristup arhitekturi

Go favorizuje:

* jednostavne strukture,
* jasne granice,
* male pakete,
* eksplicitne zavisnosti.

Za razliku od nekih jezika:

```text
Framework Magic

        |

        v

Hidden Behavior
```

Go češće koristi:

```text
Simple Code

        |

        v

Explicit Flow
```

---

# Najčešće početničke greške

## 1. Jedan ogroman package

Primer:

```text
main/

    user.go

    database.go

    payment.go

    api.go
```

Za male projekte je prihvatljivo.

Za velike postaje problem.

---

## 2. Previše paketa

Suprotan problem:

```text
user_name_validator

user_email_validator

user_address_validator
```

Previše malih paketa povećava kompleksnost.

---

## 3. Loša odgovornost paketa

Loše:

```text
utils/
```

koji sadrži sve.

Bolje:

```text
validation/

logging/

encoding/
```

---

# Mentalni model organizacije

Razmišljaj ovako:

```text
Project

 |
 +-- Applications

 |       |
 |       +-- cmd

 |
 +-- Business Logic

 |       |
 |       +-- internal

 |
 +-- Reusable Components

         |
         +-- pkg
```

---

# Ključne ideje ovog dela

Nakon ovog poglavlja treba razumeti:

* Go projekat se organizuje pomoću paketa;
* `go.mod` definiše modul i zavisnosti;
* `main` paket predstavlja izvršnu aplikaciju;
* `internal` štiti privatnu implementaciju;
* `cmd` organizuje više aplikacija;
* dobra struktura projekta olakšava održavanje.

---

# Zaključak

Go ne pokušava da nametne komplikovan framework ili strogu arhitekturu.

Umesto toga daje jednostavne građevne blokove:

* module,
* pakete,
* fajlove,
* konvencije.

Na programeru je da ih koristi tako da projekat ostane razumljiv i skalabilan.

U sledećem delu ćemo analizirati **Go filozofiju dizajna: jednostavnost, eksplicitnost i principe koji oblikuju način pisanja Go koda**.

---

# Go filozofija dizajna

U prethodnim delovima upoznali smo:

- strukturu Go programa,
- proces kompajliranja,
- organizaciju projekta,
- osnovne elemente Go ekosistema.

Sada ćemo analizirati jednu od najvažnijih tema za razumevanje Go jezika:

> **Kako Go razmišlja kao programski jezik?**

Svaki programski jezik ima određenu filozofiju dizajna.

Ta filozofija utiče na:

- način pisanja koda,
- arhitekturu aplikacija,
- način rešavanja problema,
- stil programiranja.

Go nije nastao kao skup slučajno izabranih funkcionalnosti.

Svaka važna odluka u jeziku ima određeni cilj.

---

# Glavni principi Go dizajna

Go filozofija se može sažeti kroz nekoliko ključnih principa:

1. Jednostavnost.
2. Čitljivost.
3. Eksplicitnost.
4. Minimalizam.
5. Kompozicija umesto kompleksne hijerarhije.
6. Brz razvoj.
7. Praktičnost.
8. Stabilnost.

---

# 1. Jednostavnost (Simplicity)

Jedan od glavnih ciljeva Go-a jeste:

> Kod treba da bude jednostavan za razumevanje.

Go dizajneri su smatrali da kompleksnost jezika direktno utiče na kompleksnost softvera.

---

# Kompleksan jezik

Kod kompleksnog jezika programer mora razumeti:

```text
Language Features

       |

       v

Complex Rules

       |

       v

Unexpected Behavior
````

---

# Go pristup

Go pokušava da smanji broj pravila:

```text
Simple Language

       |

       v

Predictable Code

       |

       v

Easier Maintenance
```

---

# Primer: nema implicitnih konverzija

U nekim jezicima moguće je:

```text
integer + float

automatic conversion

result
```

Go zahteva eksplicitnost.

Primer:

```go
var a int = 10
var b float64 = 5.5

result := float64(a) + b
```

Programer jasno pokazuje nameru.

---

# Zašto je ovo važno?

Kod velikih sistema:

```text
10 lines of code
        |
        v
100 developers
        |
        v
10 years maintenance
```

Mala nejasnoća može postati veliki problem.

---

# 2. Čitljivost (Readability)

Go veruje da je:

> Čitljiv kod bolji kod.

Kod se mnogo češće čita nego što se piše.

Primer:

Jednom napisana funkcija:

```text
Write once
```

Može biti pročitana:

```text
Read hundreds of times
```

---

# Go standard format

Zbog toga Go ima:

```bash
gofmt
```

Svi programeri koriste isti format.

Rezultat:

```go
func calculate() {
	return 42
}
```

Nema rasprave oko stila.

---

# 3. Eksplicitnost (Explicitness)

Jedan od najvažnijih Go principa:

> Neka stvari budu očigledne iz koda.

Go izbegava previše "magije".

---

# Primer: upravljanje greškama

Go koristi:

```go
result, err := operation()

if err != nil {
	return err
}
```

Greška je vidljiva.

---

# Alternativni pristupi

Neki jezici koriste:

```text
Exception

    |

    v

Automatic propagation
```

Problem:

Greška može biti sakrivena u toku izvršavanja.

---

# Go pristup:

```text
Function call

      |

      v

Return value

      |

      v

Developer handles error
```

---

# 4. Minimalizam (Minimalism)

Go namerno nema neke funkcionalnosti koje postoje u drugim jezicima.

Primeri:

* nema klasičnog nasleđivanja,
* nema operator overloading-a,
* nema implicitnog generičkog ponašanja pre Go 1.18,
* nema exception sistema.

---

# Zašto?

Ne zato što te funkcionalnosti nisu korisne.

Već zato što povećavaju broj mogućih načina pisanja koda.

---

# Primer: nasleđivanje

Klasični OOP:

```text
Animal

   |

   +---- Dog

   |

   +---- Cat
```

Go preferira:

```text
Animal Behavior

        +

Dog Implementation
```

odnosno:

* interfejse,
* kompoziciju.

---

# 5. Kompozicija umesto nasleđivanja

Go koristi princip:

> Combine simple pieces instead of building deep hierarchies.

---

# Nasleđivanje

Primer:

```text
BaseClass

     |

     |

ChildClass

     |

     |

GrandChildClass
```

Problem:

* jaka povezanost,
* teško menjanje.

---

# Kompozicija

Go pristup:

```text
Object

  +

Component A

  +

Component B
```

Primer:

```go
type User struct {
	Name string
	Email string
}
```

Tipovi se kombinuju umesto nasleđuju.

---

# 6. Praktičnost

Go je dizajniran za stvarne probleme.

Nije cilj bio:

```text
Maximum theoretical power
```

nego:

```text
Efficient real-world development
```

---

# Primer

Za razvoj servera potrebno je:

* HTTP biblioteka,
* JSON podrška,
* konkurentnost,
* rad sa mrežom.

Go sve ovo pruža u standardnoj biblioteci.

---

# 7. Stabilnost

Go ima veoma jak fokus na kompatibilnost.

Jedan od poznatih principa:

> Existing Go programs should continue to work.

---

# Zašto je stabilnost važna?

Profesionalne aplikacije žive godinama.

Primer:

```text
Application created

       |

       v

5 years maintenance

       |

       v

Still works
```

---

# Go verzije

Go se razvija, ali promene uglavnom ne lome postojeći kod.

Primer:

```text
Go 1.x

Application

       |

       v

Go 1.x+1

Still works
```

---

# 8. Jedan način za rešavanje problema

Go favorizuje standardizaciju.

Primer:

U drugim jezicima:

```text
5 different logging libraries

10 formatting styles

20 project structures
```

---

Go zajednica pokušava:

```text
Common Tools

      |

      v

Common Practices

      |

      v

Predictable Code
```

---

# Go Proverbs

Postoji poznata kolekcija Go principa poznata kao:

```text
Go Proverbs
```

Neke važne ideje:

---

## "Clear is better than clever"

Jasan kod je bolji od pametnog koda.

---

## "Errors are values"

Greške su vrednosti koje se obrađuju kao normalni podaci.

---

## "Don't communicate by sharing memory; share memory by communicating"

Komunikacija treba da bude osnovni način koordinacije konkurentnih procesa.

---

## "The bigger the interface, the weaker the abstraction"

Veliki interfejsi često znače lošu apstrakciju.

---

# Go način razmišljanja

Go programer razmišlja:

Ne:

```text
Kako mogu napraviti najapstraktniji dizajn?
```

Već:

```text
Kako mogu napisati najjasniji i najjednostavniji kod?
```

---

# Primer poređenja

Kompleksni pristup:

```text
Framework

   |

Magic Layer

   |

Automatic Behavior

   |

Result
```

---

Go pristup:

```text
Simple Code

   |

Explicit Flow

   |

Predictable Result
```

---

# Kako ova filozofija utiče na svakodnevni kod?

Go programer često bira:

| Umesto                 | Koristi               |
| ---------------------- | --------------------- |
| Velike hijerarhije     | Kompoziciju           |
| Skriveno ponašanje     | Eksplicitni kod       |
| Kompleksne apstrakcije | Jednostavne tipove    |
| Velike framework-e     | Standardnu biblioteku |
| Magiju                 | Jasnu logiku          |

---

# Ključne ideje ovog dela

Nakon ovog poglavlja treba razumeti:

* Go je dizajniran sa fokusom na jednostavnost;
* čitljivost je važnija od kompleksnosti;
* eksplicitni kod je poželjan;
* kompozicija je važnija od nasleđivanja;
* stabilnost je važan deo jezika;
* Go rešava praktične probleme modernog razvoja.

---

# Zaključak

Go filozofija nije zasnovana na tome da jezik ima najveći broj mogućnosti.

Naprotiv.

Go pokušava da pronađe ravnotežu između:

* jednostavnosti,
* performansi,
* produktivnosti,
* održavanja.

Razumevanje ove filozofije je ključno, jer dobar Go programer ne uči samo sintaksu jezika.

On uči **način razmišljanja koji stoji iza jezika**.

U sledećem delu ćemo analizirati **prednosti i ograničenja Go jezika i kada treba, odnosno kada ne treba izabrati Go za određeni projekat**.

---

# Prednosti i ograničenja Go jezika

U prethodnom delu analizirali smo filozofiju dizajna Go jezika:

- jednostavnost,
- čitljivost,
- eksplicitnost,
- minimalizam,
- kompoziciju,
- stabilnost.

Sada ćemo napraviti objektivnu analizu:

- koje su najveće prednosti Go jezika,
- gde Go briljira,
- koja su njegova ograničenja,
- kada Go jeste dobar izbor,
- kada drugi jezici mogu biti bolji.

Važno je razumeti:

> Ne postoji najbolji programski jezik za sve probleme.

Postoji samo odgovarajući alat za određeni problem.

---

# Prednosti Go jezika

Najvažnije prednosti Go-a mogu se podeliti u nekoliko kategorija:

1. Performanse.
2. Jednostavnost razvoja.
3. Konkurentnost.
4. Deployment.
5. Stabilnost.
6. Standardna biblioteka.
7. Skalabilnost.
8. Održavanje.

---

# 1. Visoke performanse

Go je kompajlirani jezik.

To znači da se kod prevodi direktno u mašinski kod.

Model:

```text
Go Source Code

       |

       v

Go Compiler

       |

       v

Native Binary

       |

       v

CPU Execution
````

---

# Poređenje izvršavanja

Interpretirani jezici:

```text
Source Code

       |

       v

Interpreter

       |

       v

Execution
```

Kompajlirani jezici:

```text
Source Code

       |

       v

Compiler

       |

       v

Machine Code

       |

       v

Execution
```

Go pripada drugoj grupi.

---

# Gde se performanse najviše vide?

Go je posebno efikasan kod:

* mrežnih servisa,
* API servera,
* mikroservisa,
* CLI alata,
* infrastrukture.

Primer:

```text
Thousands of Requests

          |

          v

       Go Server

          |

          v

      Fast Response
```

---

# 2. Jednostavan razvoj

Jedna od najvećih Go prednosti jeste mala kompleksnost jezika.

Programer mora znati relativno mali broj koncepata.

Osnovni elementi:

```text
Variables

Functions

Structs

Interfaces

Goroutines

Channels

Packages
```

---

# Uticaj na timove

U velikim timovima:

```text
Developer A

        |

        v

Existing Code

        |

        v

Developer B
```

važno je da kod bude lako razumljiv.

Go favorizuje predvidljiv kod.

---

# 3. Odlična konkurentnost

Jedna od najvećih prednosti Go jezika jeste model konkurentnosti.

Tradicionalni model:

```text
Thread 1

Thread 2

Thread 3

Thread 4
```

može biti težak za upravljanje.

---

Go model:

```text
Goroutine 1
Goroutine 2
Goroutine 3
Goroutine 4
Goroutine 5

        |

        v

Go Scheduler

        |

        v

OS Threads
```

---

# Goroutine karakteristike

Goroutine:

* je lagana,
* brzo se kreira,
* koristi malo memorije,
* upravlja je Go runtime.

Zbog toga možemo imati:

```text
10 goroutines

100 goroutines

10.000 goroutines

100.000 goroutines
```

u određenim scenarijima.

---

# 4. Jednostavan deployment

Jedna od velikih praktičnih prednosti Go-a jeste distribucija aplikacija.

Primer:

```text
Development Machine

        |

        v

go build

        |

        v

Single Binary

        |

        v

Production Server
```

---

# Prednosti single binary pristupa

Nema potrebe za:

* instalacijom runtime-a,
* upravljanjem paketima na serveru,
* komplikovanim dependency setup-om.

---

# Primer Docker slike

Go aplikacija često koristi:

```text
Docker Image

    |
    +-- Go Binary

    |
    +-- Minimal OS Layer
```

Rezultat:

* male slike,
* brzo pokretanje,
* lak deployment.

---

# 5. Stabilnost jezika

Go ima veoma pažljiv pristup promenama.

Cilj:

```text
Old Code

     |

     v

New Go Version

     |

     v

Still Works
```

---

# Zašto je stabilnost važna?

Kompanije koriste softver godinama.

Primer:

```text
2026

Application Created


2030

Application Maintained
```

Jezik mora podržati dug životni ciklus.

---

# 6. Bogata standardna biblioteka

Go dolazi sa velikim brojem kvalitetnih paketa.

Primeri:

## HTTP

```go
net/http
```

## JSON

```go
encoding/json
```

## Cryptography

```go
crypto/*
```

## Filesystem

```go
os
path/filepath
```

---

# Prednost standardne biblioteke

Manje eksternih zavisnosti znači:

* manje problema sa verzijama,
* jednostavniji build,
* lakše održavanje.

---

# 7. Skalabilnost

Go je dizajniran za sisteme koji rastu.

Primer:

Mali servis:

```text
100 requests/sec
```

Kasnije:

```text
100.000 requests/sec
```

Go pruža alate za:

* konkurentnost,
* profiling,
* monitoring,
* optimizaciju.

---

# 8. Održavanje velikih sistema

Veliki sistemi zahtevaju:

* čitljiv kod,
* standarde,
* jednostavnu komunikaciju u timu.

Go pomaže kroz:

```text
gofmt

go test

go vet

simple syntax
```

---

# Ograničenja Go jezika

Pored prednosti, Go ima i ograničenja.

Razumevanje ograničenja je jednako važno.

---

# 1. Manje apstrakcija nego u nekim jezicima

Go namerno izbegava veliki broj kompleksnih funkcionalnosti.

Primer:

Nema:

* klasično nasleđivanje,
* operator overloading,
* kompleksne makro sisteme.

---

# Posledica

Neki problemi mogu zahtevati više ručnog koda.

Primer:

```text
More explicit code

        |

        v

Less hidden behavior
```

---

# 2. Garbage Collector

Go koristi Garbage Collector.

Prednosti:

* jednostavniji razvoj,
* manje memory leak problema.

Ali:

* GC ima određeni overhead,
* nije idealan za svaki real-time sistem.

---

# Primer

Sistem sa ekstremnim zahtevima:

```text
Microsecond latency

        |

        v

No unpredictable pauses
```

možda će izabrati drugi pristup.

---

# 3. Generics istorijski kasno uvedeni

Go je dugo imao samo osnovni sistem tipova.

Generics su uvedeni u:

```text
Go 1.18
```

---

# Posledica

Stariji Go kod često koristi:

* interface{},
* type assertions,
* ručne implementacije.

Moderni Go sada omogućava generičke strukture.

---

# 4. Manje biblioteka za određene oblasti

Go nije dominantan u:

* machine learning,
* data science,
* frontend,
* mobilni razvoj.

---

# Primer:

Za ML:

```text
Python

      >

Go
```

Za Android:

```text
Kotlin

      >

Go
```

---

# 5. Više eksplicitnog koda

Go često zahteva da programer napiše više detalja.

Primer:

```go
if err != nil {
	return err
}
```

Ovo može izgledati repetitivno.

Ali cilj je:

* jasnost,
* kontrola,
* predvidljivost.

---

# Kada izabrati Go?

Go je odličan izbor za:

```text
✓ Backend Services

✓ Cloud Applications

✓ Microservices

✓ DevOps Tools

✓ Networking

✓ Distributed Systems

✓ CLI Applications
```

---

# Kada razmotriti drugi jezik?

## Rust

Ako je najvažnije:

```text
Maximum memory control
```

---

## C/C++

Ako je potrebno:

```text
Low-level hardware access
```

---

## Python

Ako je fokus:

```text
Data Science
Machine Learning
Rapid Prototyping
```

---

## Java/Kotlin

Ako je potrebno:

```text
Enterprise ecosystem
Large JVM tooling
```

---

# Pravi način razmišljanja

Pogrešno pitanje:

> Koji je najbolji programski jezik?

Bolje pitanje:

> Koji je najbolji jezik za ovaj konkretan problem?

---

# Go pozicija u svetu jezika

Možemo ga predstaviti ovako:

```text
                  Performance

                       ^
                       |

        C++            |          Rust

                       |

                       |      Go

                       |

        Python         |

                       |

                       +-------------------->

                         Productivity
```

Go pokušava da bude balans između:

* performansi,
* jednostavnosti,
* produktivnosti.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go je brz jer je kompajliran;
* jednostavan je za održavanje;
* odličan je za konkurentne server aplikacije;
* lako se deploy-uje;
* ima ograničenja koja su namerna;
* izbor jezika zavisi od problema.

---

# Zaključak

Go nije savršen programski jezik.

Ali je veoma dobro dizajniran za svoju ciljnu oblast:

* moderne servere,
* cloud infrastrukturu,
* distribuirane sisteme,
* backend razvoj.

Razumevanje njegovih prednosti i ograničenja omogućava programeru da donese bolje tehničke odluke.

U sledećem delu ćemo analizirati **Go u poređenju sa drugim popularnim programskim jezicima i gde se tačno nalazi u modernom softverskom ekosistemu**.

---

# Go u poređenju sa drugim programskim jezicima

U prethodnom delu analizirali smo:

- prednosti Go jezika,
- njegova ograničenja,
- situacije u kojima je Go dobar izbor,
- situacije u kojima drugi jezici mogu biti pogodniji.

Sada ćemo pozicionirati Go u odnosu na druge popularne programske jezike.

Cilj nije da odredimo "pobednika".

Cilj je da razumemo:

- gde se Go razlikuje,
- zašto je nastao,
- koje probleme rešava bolje od drugih jezika.

---

# Go pozicija među programskim jezicima

Go zauzima specifičnu poziciju.

On se nalazi između nekoliko svetova:

```text
              Low Level
                  ^
                  |
                  |
          C / C++ / Rust
                  |
                  |
                  |
                  |        Go
                  |
                  |
          Java / C# / Kotlin
                  |
                  |
                  |
          Python / JavaScript
                  |
                  |
                  v

              High Level
````

Go pokušava da kombinuje:

* brzinu kompajliranih jezika,
* jednostavnost modernih jezika,
* produktivnost visokog nivoa.

---

# Go vs C

## C karakteristike

C je jedan od najuticajnijih jezika u istoriji.

Koristi se za:

* operativne sisteme,
* embedded sisteme,
* drivere,
* low-level programiranje.

Primer:

```text
Hardware

   |

   v

C Code

   |

   v

Operating System
```

---

# Prednosti C jezika

C pruža:

* direktnu kontrolu memorije,
* minimalan runtime,
* veoma visoke performanse.

---

# Problemi C jezika

Programer mora ručno upravljati memorijom.

Primer:

```c
malloc()

free()
```

Greške mogu dovesti do:

* memory leak-a,
* invalid pointer-a,
* corruption-a memorije.

---

# Go pristup

Go uklanja deo kompleksnosti:

```text
Manual Memory Management

        |

        v

Garbage Collector
```

---

# Poređenje

| Osobina           | C            | Go       |
| ----------------- | ------------ | -------- |
| Performanse       | Veoma visoke | Visoke   |
| Memory management | Ručni        | GC       |
| Bezbednost        | Niža         | Viša     |
| Produktivnost     | Niža         | Viša     |
| Concurrency       | Ručna        | Ugrađena |

---

# Go vs C++

C++ proširuje C sa:

* klasama,
* template sistemom,
* operator overloading-om,
* velikim brojem mogućnosti.

---

# Prednost C++

C++ omogućava:

* ekstremne performanse,
* kompleksne apstrakcije,
* kontrolu memorije.

Koristi se za:

* game engine-e,
* finansijske sisteme,
* high-performance aplikacije.

---

# Problem C++ jezika

Veliki broj funkcionalnosti povećava kompleksnost.

Primer:

```text
Many Features

        |

        v

Many Possible Designs

        |

        v

Higher Complexity
```

---

# Go pristup

Go svesno bira:

```text
Fewer Features

        |

        v

Simpler Code
```

---

# Go vs Java

Java je jedan od najpopularnijih enterprise jezika.

Koristi:

```text
JVM

 |

 v

Java Application
```

---

# Prednosti Java jezika

Java pruža:

* ogromni ekosistem,
* enterprise biblioteke,
* mature tooling,
* stabilan runtime.

---

# Razlike u izvršavanju

Java:

```text
Java Code

    |

    v

Bytecode

    |

    v

JVM

    |

    v

Execution
```

---

Go:

```text
Go Code

    |

    v

Native Binary

    |

    v

Execution
```

---

# Poređenje

| Osobina      | Java    | Go            |
| ------------ | ------- | ------------- |
| Runtime      | JVM     | Native binary |
| Startup time | Veći    | Manji         |
| Memory usage | Veći    | Manji         |
| Concurrency  | Threads | Goroutines    |
| Ecosystem    | Ogroman | Rastući       |

---

# Go vs C#

C# je Microsoft-ov moderan jezik.

Koristi:

* .NET runtime,
* enterprise aplikacije,
* web razvoj.

---

# Sličnosti

Oba jezika nude:

* garbage collection,
* moderne alate,
* dobru produktivnost.

---

# Razlike

C# ima:

* bogatiji OOP model,
* LINQ,
* kompleksniji type system.

Go ima:

* jednostavniju sintaksu,
* lakši deployment,
* ugrađenu konkurentnost.

---

# Go vs Python

Python je jedan od najpopularnijih jezika danas.

Koristi se za:

* automatizaciju,
* data science,
* AI,
* scripting.

---

# Prednosti Python-a

Python ima:

* ogromnu biblioteku,
* jednostavan početak,
* odličan ekosistem.

---

# Problem Python-a

Python je interpretiran.

Model:

```text
Python Code

     |

     v

Interpreter

     |

     v

Execution
```

Za CPU-intensive servise često je sporiji.

---

# Go pristup

Go:

```text
Go Code

     |

     v

Compiled Binary

     |

     v

Execution
```

---

# Poređenje

| Osobina     | Python            | Go            |
| ----------- | ----------------- | ------------- |
| Sintaksa    | Veoma jednostavna | Jednostavna   |
| Performanse | Srednje           | Visoke        |
| ML/Data     | Odličan           | Slabiji       |
| Backend     | Dobar             | Odličan       |
| Deployment  | Zahteva runtime   | Single binary |

---

# Go vs JavaScript / Node.js

Node.js koristi JavaScript za backend.

Prednosti:

* veliki ekosistem,
* brz razvoj,
* frontend/backend isti jezik.

---

# Node.js model

```text
Event Loop

     |

     v

Async Operations
```

---

# Go model

```text
Goroutines

     |

     v

Scheduler

     |

     v

Parallel Execution
```

---

# Poređenje

| Osobina     | Node.js    | Go         |
| ----------- | ---------- | ---------- |
| Concurrency | Event loop | Goroutines |
| CPU tasks   | Slabije    | Bolje      |
| Ecosystem   | Ogroman    | Manji      |
| Deployment  | Runtime    | Binary     |

---

# Go vs Rust

Rust je moderan sistemski jezik.

Njegov cilj:

```text
C++ Performance

+

Memory Safety
```

---

# Rust pristup

Rust koristi:

* ownership,
* borrowing,
* lifetime sistem.

---

# Go pristup

Go koristi:

* garbage collector,
* jednostavniji model memorije.

---

# Poređenje

| Osobina           | Rust       | Go         |
| ----------------- | ---------- | ---------- |
| Memory control    | Maksimalna | Automatska |
| Learning curve    | Viša       | Niža       |
| Performance       | Ekstremna  | Visoka     |
| Development speed | Srednja    | Visoka     |

---

# Gde Go posebno dominira?

Go je veoma popularan u:

```text
Cloud Infrastructure

Microservices

DevOps Tools

Networking

Distributed Systems

Backend APIs
```

---

# Primeri sistema gde se koristi Go

Poznati projekti napisani u Go-u:

* Kubernetes,
* Docker,
* Terraform,
* Prometheus.

Ovi projekti imaju zajedničke potrebe:

* skaliranje,
* konkurentnost,
* pouzdanost,
* jednostavan deployment.

---

# Kako izabrati jezik?

Prilikom izbora jezika treba razmotriti:

```text
Problem

 |

 +-- Performance?

 |

 +-- Development Speed?

 |

 +-- Ecosystem?

 |

 +-- Team Knowledge?

 |

 +-- Maintenance?
```

---

# Go filozofija izbora

Go nije napravljen da zameni sve jezike.

Napravljen je da bude odličan za određenu grupu problema.

---

# Mentalni model

Možemo posmatrati jezike ovako:

```text
C/C++

Maximum Control


Rust

Maximum Safety


Go

Balance


Java/C#

Enterprise Productivity


Python/JS

Maximum Flexibility
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go nije zamena za sve jezike;
* Go predstavlja balans između performansi i produktivnosti;
* Go je jednostavniji od C++/Rust pristupa;
* Go ima prednosti za backend i cloud sisteme;
* izbor jezika zavisi od problema.

---

# Zaključak

Go zauzima veoma specifično mesto u svetu programskih jezika.

Njegova najveća vrednost nije u tome što ima najviše mogućnosti.

Njegova vrednost je u tome što omogućava:

* brz razvoj,
* jednostavan kod,
* dobre performanse,
* odličnu konkurentnost.

Zbog toga je Go postao jedan od najvažnijih jezika modernog backend i cloud ekosistema.

U sledećem delu ćemo analizirati **istoriju razvoja Go jezika, njegove verzije i najvažnije promene kroz vreme**.

---

# Istorija razvoja Go jezika

U prethodnom delu analizirali smo poziciju Go jezika u odnosu na druge programske jezike:

- C,
- C++,
- Java,
- C#,
- Python,
- JavaScript,
- Rust.

Sada ćemo analizirati kako je Go nastao, koji problemi su doveli do njegovog stvaranja i kako se jezik razvijao kroz vreme.

Razumevanje istorije jezika pomaže nam da razumemo njegove dizajnerske odluke.

---

# Zašto je Go nastao?

Go nije nastao kao eksperimentalni projekat.

Njegov cilj je bio rešavanje konkretnih problema sa kojima su se suočavali inženjeri velikih sistema.

Početkom 2000-ih godina veliki softverski sistemi postajali su sve kompleksniji.

Kompanije su se suočavale sa problemima:

- dugo vreme kompajliranja,
- kompleksan dependency management,
- teško održavanje velikih codebase-ova,
- komplikovana konkurentnost,
- nedovoljna iskorišćenost modernog hardvera.

---

# Problemi velikih sistema

Tipičan veliki projekat:

```text
Large Codebase

      |

      v

Millions of Lines

      |

      v

Thousands of Developers

      |

      v

Complex Maintenance
````

---

# Problem 1: Vreme kompajliranja

Kod velikih C++ sistema:

```text
Change Code

    |

    v

Compile

    |

    v

Wait Minutes / Hours
```

Ovo usporava razvoj.

---

# Problem 2: Kompleksnost jezika

Moderni jezici su vremenom dobijali nove funkcionalnosti.

Primer:

```text
More Features

      |

      v

More Possibilities

      |

      v

More Complexity
```

Programeri su morali da poznaju sve veći broj pravila.

---

# Problem 3: Višejezgreni procesori

Hardver se promenio.

Ranije:

```text
Single CPU Core
```

Moderni sistemi:

```text
CPU

 +-- Core 1

 +-- Core 2

 +-- Core 3

 +-- Core 4

 +-- Core N
```

Softver mora efikasno koristiti paralelizam.

---

# Problem 4: Mrežni sistemi

Moderni sistemi postali su distribuirani.

Primer:

```text
Client

   |

   v

API Server

   |

   v

Database

   |

   v

External Services
```

Potrebna je jednostavna konkurentnost.

---

# Nastanak Go jezika

Go je nastao u kompaniji:

```text
Google
```

Početak razvoja:

```text
2007
```

Autori:

* Robert Griesemer,
* Rob Pike,
* Ken Thompson.

---

# Autori Go jezika

## Robert Griesemer

Radio je na:

* runtime sistemima,
* kompajlerima,
* programskim jezicima.

---

## Rob Pike

Poznat po radu na:

* Unix sistemima,
* UTF-8 standardu,
* Plan 9 operativnom sistemu.

---

## Ken Thompson

Jedan od kreatora:

* Unix operativnog sistema,
* C programskog jezika.

Dobitnik:

```text
ACM Turing Award
```

---

# Prvi cilj Go projekta

Osnovna ideja:

Napraviti jezik koji kombinuje:

```text
C-like Performance

+

Python-like Productivity

+

Modern Concurrency
```

---

# Početna motivacija

Autori su želeli jezik koji je:

* brz za kompajliranje,
* jednostavan za učenje,
* efikasan na modernom hardveru,
* pogodan za velike server sisteme.

---

# Go 1.0

Prva stabilna verzija:

```text
Go 1.0

2012
```

Ovo je bio veoma važan trenutak.

Od tada Go je obećao:

> Kompatibilnost budućih verzija.

---

# Go Compatibility Promise

Jedan od najvažnijih principa:

```text
Program written today

        |

        v

Runs on future Go versions
```

---

# Razvoj kroz verzije

Go se razvijao postepeno.

Glavne oblasti razvoja:

* tooling,
* performanse,
* garbage collector,
* generics,
* sigurnost,
* developer experience.

---

# Go 1.5

Jedna od važnih verzija.

Donela je:

* novi garbage collector,
* potpuno Go napisan compiler toolchain.

Pre toga:

```text
C Compiler

      |

      v

Go Compiler
```

Posle:

```text
Go Compiler

      |

      v

Go Compiler
```

---

# Go 1.7

Doneta poboljšanja:

* compiler optimizacije,
* bolje performanse,
* context paket u standardnoj biblioteci.

---

# Context paket

Danas veoma važan za server aplikacije:

```go
context.Context
```

Koristi se za:

* cancellation,
* timeout,
* request lifecycle.

---

# Go 1.9

Uveden:

```text
Type Alias
```

Primer:

```go
type UserID = int64
```

---

# Go 1.10+

Poboljšanja:

* build cache,
* brži razvoj,
* bolji tooling.

---

# Go Modules era

Jedna od najvećih promena:

```text
Go 1.11
```

Uvedeni su:

```text
Go Modules
```

---

# Pre Go Modules

Korišćen je:

```text
GOPATH
```

Struktura:

```text
$GOPATH/src/project
```

---

# Posle Go Modules

Projekat može biti bilo gde:

```text
/home/user/projects/app
```

sa:

```text
go.mod
```

---

# Go 1.14+

Poboljšanja:

* stabilnost modula,
* runtime optimizacije,
* poboljšanja scheduler-a.

---

# Go 1.17

Važne promene:

* unapređen compiler,
* efikasniji generisani kod,
* poboljšan module sistem.

---

# Go 1.18

Jedna od najvećih verzija u istoriji Go-a.

Uvedeni:

```text
Generics
```

---

# Pre Go 1.18

Programeri su često koristili:

```go
interface{}
```

Primer:

```go
func Print(value interface{}) {

}
```

---

# Posle Go 1.18

Moguće je:

```go
func Print[T any](value T) {

}
```

---

# Zašto su generics važne?

Omogućavaju:

* reusable code,
* type safety,
* manje dupliranja.

---

# Go 1.20+

Poboljšanja:

* performance optimizacije,
* fuzz testing,
* runtime poboljšanja,
* developer tooling.

---

# Go danas

Moderni Go predstavlja:

```text
Simple Language

        +

Powerful Runtime

        +

Excellent Tooling

        +

Strong Ecosystem
```

---

# Evolucija Go prioriteta

Možemo je predstaviti:

```text
2007

Language Design


2012

Go 1.0


2018

Modules


2022

Generics


Today

Performance + Developer Experience
```

---

# Važna lekcija iz istorije Go-a

Go nije pokušao da bude:

* najmoćniji jezik,
* najfleksibilniji jezik,
* jezik sa najviše funkcija.

Njegov cilj je bio:

```text
Maximum Productivity

with

Minimum Complexity
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go je nastao u Google-u;
* napravljen je zbog problema velikih sistema;
* prvi stabilni release bio je Go 1.0 2012. godine;
* kompatibilnost je jedan od glavnih principa;
* Go se razvijao postepeno;
* generics su uvedeni u Go 1.18;
* današnji Go kombinuje jednostavnost i moderne mogućnosti.

---

# Zaključak

Istorija Go jezika pokazuje da njegove karakteristike nisu slučajne.

Svaka važna odluka proizašla je iz realnih problema:

* velikih codebase-ova,
* server infrastrukture,
* distribuiranih sistema,
* modernog hardvera.

Da bismo dobro koristili Go, nije dovoljno znati sintaksu.

Potrebno je razumeti **zašto je jezik dizajniran upravo na taj način**.

U sledećem delu ćemo analizirati **najvažnije oblasti primene Go jezika i tipove sistema za koje je posebno dizajniran**.

---

# Oblasti primene Go jezika

U prethodnom delu analizirali smo istoriju nastanka Go jezika:

- probleme koje je Go rešavao,
- razloge njegovog dizajna,
- razvoj kroz verzije,
- najvažnije promene kroz vreme.

Sada ćemo analizirati gde se Go najčešće koristi u realnom svetu.

Razumevanje oblasti primene je važno zato što određuje:

- koje koncepte treba najviše učiti,
- kakvu arhitekturu koristiti,
- koje biblioteke i alate poznavati.

---

# Gde se Go najčešće koristi?

Go je posebno popularan u oblastima:

```text
+---------------------------+

| Backend Development       |

| Cloud Infrastructure      |

| DevOps Tools              |

| Distributed Systems       |

| Networking                |

| CLI Applications          |

| Platform Engineering      |

| Microservices             |

+---------------------------+
````

---

# 1. Backend razvoj

Jedna od najvažnijih oblasti Go primene jeste razvoj backend sistema.

Tipična arhitektura:

```text
                 Client

                   |

                   v

              HTTP Request

                   |

                   v

              Go Backend

                   |

        +----------+----------+

        |          |          |

        v          v          v

   Database    Cache    External APIs
```

---

# Zašto je Go dobar za backend?

Backend sistemi često zahtevaju:

* veliki broj istovremenih zahteva,
* stabilnost,
* brzo izvršavanje,
* jednostavan deployment.

Go pruža:

* goroutine,
* standardni HTTP paket,
* efikasan runtime,
* jednostavan deployment.

---

# Primer Go HTTP servera

Jednostavan server:

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello Go")
}

func main() {
	http.HandleFunc("/", handler)

	http.ListenAndServe(":8080", nil)
}
```

Go standardna biblioteka već sadrži HTTP podršku.

---

# Tipični backend sistemi napisani u Go-u

Go se koristi za:

* REST API servise,
* GraphQL servere,
* gRPC servise,
* authentication servise,
* payment sisteme,
* data processing servise.

---

# 2. Mikroservisi

Go je veoma popularan izbor za mikroservisnu arhitekturu.

Primer:

```text
                Application

                    |

     +--------------+--------------+

     |              |              |

     v              v              v

 User Service   Payment Service   Order Service
```

---

# Zašto Go odgovara mikroservisima?

Mikroservisi često zahtevaju:

* mali memory footprint,
* brzo pokretanje,
* jednostavan deployment,
* dobru mrežnu podršku.

Go pruža sve navedeno.

---

# Primer jednog servisa

```text
payment-service

    |

    +-- HTTP API

    |

    +-- Business Logic

    |

    +-- Database Layer

    |

    +-- Background Workers
```

---

# 3. Cloud infrastruktura

Go je jedan od najvažnijih jezika modernog cloud ekosistema.

Razlozi:

* odličan networking,
* konkurentnost,
* jednostavni binarni fajlovi,
* stabilan runtime.

---

# Cloud sistemi zahtevaju

```text
High Availability

        +

Scalability

        +

Performance

        +

Reliability
```

Go je dizajniran upravo za ovakve probleme.

---

# Poznati cloud projekti napisani u Go-u

Neki od najpoznatijih projekata:

* Docker
* Kubernetes
* HashiCorp Terraform
* Prometheus

---

# 4. DevOps alati

Go je postao veoma popularan u DevOps svetu.

Razlog:

DevOps alati često moraju da:

* rade na različitim sistemima,
* budu mali i prenosivi,
* komuniciraju sa mrežom,
* izvršavaju se iz komandne linije.

---

# Primer CLI alata

```text
Developer

    |

    v

Command Line Tool

    |

    v

Cloud API
```

---

# Primeri zadataka

Go se koristi za:

* deployment alate,
* infrastrukturu,
* monitoring,
* automatizaciju,
* CI/CD alate.

---

# 5. CLI aplikacije

Go je odličan za pravljenje komandnih alata.

Primer:

```bash
mytool deploy production
```

---

# Zašto Go za CLI?

Prednosti:

* jedan izvršni fajl,
* brz startup,
* nema dodatnog runtime-a.

---

# Primer strukture CLI aplikacije

```text
my-tool/

    main.go

    commands/

    config/

    services/
```

---

# 6. Distribuirani sistemi

Distribuirani sistemi predstavljaju jednu od oblasti gde Go posebno dolazi do izražaja.

Primer:

```text
Service A

    |

    v

Message Queue

    |

    v

Service B

    |

    v

Database
```

---

# Zahtevi distribuiranih sistema

Potrebno je:

* mrežna komunikacija,
* paralelna obrada,
* koordinacija procesa,
* tolerancija na greške.

Go pruža:

* goroutine,
* channel,
* context,
* networking biblioteke.

---

# 7. Networking

Go ima veoma snažnu podršku za mrežno programiranje.

Standardna biblioteka uključuje:

```go
net
net/http
net/url
net/smtp
```

---

# Primeri networking sistema

Go se koristi za:

* proxy servere,
* API gateway-e,
* load balancer-e,
* VPN alate,
* servisnu infrastrukturu.

---

# 8. Platform Engineering

Moderna infrastruktura koristi koncept:

```text
Platform Engineering
```

Cilj:

Napraviti interne platforme koje olakšavaju razvoj timovima.

---

# Primer:

```text
Developer

    |

    v

Internal Platform

    |

    v

Cloud Infrastructure
```

---

# Go u platform engineering-u

Koristi se za:

* Kubernetes operatere,
* infrastrukturu,
* automatizaciju,
* developer tooling.

---

# 9. Sistemски alati

Iako Go nije low-level jezik kao C ili Rust, može se koristiti za mnoge sistemske alate.

Primeri:

* backup alati,
* monitoring agenti,
* file processing alati,
* networking utilities.

---

# 10. Data processing

Go se koristi i za obradu podataka.

Primer:

```text
Input Data

    |

    v

Go Processor

    |

    v

Output Data
```

---

# Prednosti:

* dobra brzina,
* jednostavna konkurentnost,
* mala potrošnja resursa.

---

# Gde Go nije dominantan?

Važno je razumeti i oblasti gde Go nije prvi izbor.

---

# 1. Machine Learning

Dominantni jezici:

```text
Python

    +

ML Libraries
```

Razlog:

* ogromni ekosistem,
* biblioteke,
* istraživački alati.

---

# 2. Frontend razvoj

Za browser:

```text
JavaScript / TypeScript
```

ostaju standard.

---

# 3. Mobilne aplikacije

Dominantni izbori:

* Kotlin,
* Swift,
* Flutter.

---

# 4. Embedded sistemi

Za ekstremno male uređaje često se koriste:

* C,
* C++,
* Rust.

---

# Kako prepoznati dobar Go problem?

Go je odličan izbor kada problem ima:

```text
                 Problem

                    |

        +-----------+-----------+

        |                       |

        v                       v

   Networking             Concurrency

        |                       |

        +-----------+-----------+

                    |

                    v

                   Go
```

---

# Primeri pitanja pri izboru Go-a

Pitaj:

## Da li imam mnogo paralelnih zadataka?

Ako da:

```text
Go ✓
```

---

## Da li pravim mrežni servis?

Ako da:

```text
Go ✓
```

---

## Da li želim jednostavan deployment?

Ako da:

```text
Go ✓
```

---

## Da li mi je potrebna maksimalna kontrola memorije?

Možda:

```text
Rust / C++
```

---

# Go u modernom softveru

Današnji softverski svet zahteva:

```text
Cloud

+

Distributed Systems

+

Automation

+

Scalability
```

Upravo tu Go ima veoma jaku poziciju.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go je veoma popularan za backend;
* odličan je za cloud i infrastrukturu;
* često se koristi za mikroservise;
* veoma je pogodan za CLI alate;
* konkurentnost je jedna od njegovih najvećih prednosti;
* nije univerzalna zamena za sve jezike.

---

# Zaključak

Go je nastao kao jezik za rešavanje praktičnih problema modernog softvera.

Njegova najveća snaga je kombinacija:

* jednostavnosti,
* performansi,
* konkurentnosti,
* lakog deployment-a.

Zbog toga je postao jedan od ključnih jezika u svetu cloud infrastrukture i backend razvoja.

U sledećem delu ćemo analizirati **Go ekosistem, zajednicu, biblioteke i alate koji čine svakodnevni rad Go programera**.

---

# Go ekosistem, zajednica i alati

U prethodnom delu analizirali smo oblasti u kojima se Go najčešće koristi:

- backend razvoj,
- mikroservisi,
- cloud infrastruktura,
- DevOps alati,
- distribuirani sistemi,
- networking.

Sada ćemo analizirati širi Go ekosistem.

Programski jezik nije samo sintaksa.

Uspeh jednog jezika zavisi od:

- standardne biblioteke,
- alata,
- zajednice,
- biblioteka,
- dokumentacije,
- načina razvoja.

Go je posebno jak upravo zbog kompletnog ekosistema koji ga okružuje.

---

# Šta čini jedan programski ekosistem?

Možemo ga predstaviti ovako:

```text
                 Go Ecosystem


                    Language

                       |

        +--------------+--------------+

        |              |              |

        v              v              v

 Standard Library   Tooling      Community


        |              |              |

        v              v              v

 Libraries       Development    Knowledge
````

---

# 1. Standardna biblioteka

Jedna od najvećih prednosti Go-a jeste bogata standardna biblioteka.

Za mnoge svakodnevne zadatke nije potrebno instalirati dodatne pakete.

---

# Najvažniji standardni paketi

## HTTP

```go
net/http
```

Koristi se za:

* HTTP servere,
* klijente,
* API-je.

---

## JSON

```go
encoding/json
```

Omogućava:

* serializaciju,
* deserializaciju,
* rad sa JSON podacima.

---

## Filesystem

```go
os
path/filepath
io
```

Koristi se za:

* fajlove,
* direktorijume,
* tokove podataka.

---

## Concurrency

```go
sync
context
runtime
```

Omogućava:

* sinhronizaciju,
* kontrolu goroutine-a,
* upravljanje životnim ciklusom procesa.

---

# Primer

Umesto:

```text
Install HTTP Library

Install JSON Library

Install File Library
```

Go često omogućava:

```text
Standard Library

        |

        v

Ready to Use
```

---

# 2. Go Toolchain

Go dolazi sa kompletnim setom razvojnih alata.

Najvažnije komande:

```bash
go run
go build
go test
go fmt
go vet
go mod
```

---

# Go workflow

Tipičan razvojni proces:

```text
Write Code

    |

    v

go fmt

    |

    v

go test

    |

    v

go vet

    |

    v

go build

    |

    v

Deploy
```

---

# 3. gofmt

Jedan od najvažnijih Go alata.

Njegova svrha:

```text
Automatic Code Formatting
```

---

# Primer

Pre:

```go
func main(){
fmt.Println("Hello")
}
```

Posle:

```go
func main() {
	fmt.Println("Hello")
}
```

---

# Zašto je gofmt važan?

Zato što uklanja rasprave o stilu.

U mnogim jezicima:

```text
Team A Style

vs

Team B Style
```

Go:

```text
One Standard Format
```

---

# 4. Go Modules

Moderni Go koristi:

```text
Go Modules
```

za upravljanje zavisnostima.

Osnovni fajlovi:

```text
go.mod

go.sum
```

---

# Primer projekta

```text
my-project/

    go.mod

    go.sum

    main.go
```

---

# Prednosti modula

Omogućavaju:

* kontrolu verzija,
* reproduktivne build-ove,
* jednostavno deljenje projekta.

---

# 5. Testiranje

Testiranje je ugrađeno u Go filozofiju.

Osnovna komanda:

```bash
go test
```

---

# Go test struktura

Primer:

```text
calculator.go

calculator_test.go
```

---

# Prednosti ugrađenog test sistema

Nema potrebe za:

* posebnim test framework-om,
* komplikovanim konfiguracijama.

---

# 6. Debugging alati

Go ima razvijen debugging ekosistem.

Najpoznatiji debugger:

```text
Delve
```

Omogućava:

* breakpoint-e,
* pregled promenljivih,
* step-by-step izvršavanje,
* debugging goroutine-a.

---

# 7. Performance alati

Go uključuje alate za analizu performansi.

Najpoznatiji:

```text
pprof
```

---

# Može analizirati:

* CPU usage,
* memoriju,
* goroutine stanje,
* runtime ponašanje.

---

# Primer problema

Aplikacija je spora:

```text
Request

   |

   v

Go Service

   |

   v

Unknown Bottleneck
```

Profiling:

```text
pprof

   |

   v

Find Problem
```

---

# 8. Go zajednica

Go zajednica je jedan od važnih razloga uspeha jezika.

Karakteristike:

* praktična,
* fokusirana na kvalitet,
* orijentisana ka standardima.

---

# Go Community vrednosti

Često se naglašavaju:

```text
Simplicity

Clarity

Practical Solutions

Open Source
```

---

# 9. Open Source ekosistem

Veliki broj važnih alata je otvorenog koda.

Primeri:

* Kubernetes,
* Docker,
* Terraform,
* Prometheus.

---

# Zašto je Open Source važan?

Omogućava:

* brži razvoj,
* pregled koda,
* doprinos zajednice.

---

# 10. Biblioteke i paketi

Go ima veliki broj eksternih biblioteka.

Najčešće oblasti:

```text
Web

Databases

Messaging

Cloud

Authentication

Logging

Monitoring
```

---

# Pronalaženje paketa

Glavno mesto:

```text
pkg.go.dev
```

---

# Šta sadrži?

Za svaki paket možemo videti:

* dokumentaciju,
* verzije,
* primer korišćenja,
* API.

---

# 11. IDE podrška

Go ima odličnu podršku u modernim editorima.

Najčešći izbori:

* Visual Studio Code,
* GoLand,
* Vim/Neovim.

---

# Go Language Server

Standardni alat:

```text
gopls
```

Omogućava:

* autocomplete,
* navigaciju kroz kod,
* refactoring,
* analizu.

---

# 12. Dokumentacija

Go ima veoma kvalitetnu dokumentaciju.

Glavni izvori:

* zvanična dokumentacija,
* Go spec,
* standard library docs,
* blog članci.

---

# Zašto je dokumentacija važna?

Dobar jezik zahteva:

```text
Clear Language

+

Clear Documentation
```

---

# 13. Go filozofija biblioteka

Go ekosistem uglavnom preferira:

```text
Small Packages

+

Clear Responsibilities

+

Simple APIs
```

---

# Loš primer:

```text
everything-utils-package
```

koji sadrži:

* string helper-e,
* database funkcije,
* HTTP kod,
* logging.

---

# Bolji primer:

```text
stringutil

database

logger

validator
```

---

# 14. Go standardi u industriji

U profesionalnom Go razvoju često se koriste:

```text
gofmt

go test

go vet

golangci-lint

CI/CD pipelines
```

---

# Primer profesionalnog workflow-a

```text
Developer

    |

    v

Commit Code

    |

    v

CI Pipeline

    |

    +-- go test

    |

    +-- go vet

    |

    +-- lint

    |

    v

Deployment
```

---

# 15. Zašto je Go ekosistem uspešan?

Možemo izdvojiti nekoliko razloga:

---

## Jednostavan početak

Početnik može brzo napisati prvi program.

---

## Profesionalna snaga

Iskusni programeri mogu graditi:

* distribuirane sisteme,
* cloud platforme,
* velike servise.

---

## Stabilnost

Postojeći kod dugo ostaje kompatibilan.

---

## Standardizacija

Timovi lakše sarađuju.

---

# Mentalni model Go ekosistema

```text
                 Go Developer


                       |

        +--------------+--------------+

        |              |              |

        v              v              v

      Code          Tools        Knowledge


        |              |              |

        v              v              v

   Application    Workflow     Community
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go nije samo programski jezik;
* standardna biblioteka je važan deo ekosistema;
* Go toolchain je centralni deo razvoja;
* moduli rešavaju dependency management;
* alati poput gofmt, test i vet su standard;
* zajednica i open-source projekti čine Go veoma snažnim.

---

# Zaključak

Snaga Go jezika ne dolazi samo iz njegove sintakse.

Dolazi iz kompletnog sistema:

* jezika,
* runtime-a,
* standardne biblioteke,
* alata,
* zajednice.

Upravo zbog toga Go omogućava programerima da od jednostavnog prototipa brzo dođu do stabilnog produkcionog sistema.

U sledećem i poslednjem delu ovog poglavlja napravićemo **konačan pregled svega što smo naučili u Introduction delu i definisati mentalni model koji će biti osnova za ostatak Go Mastery tutorijala**.

---

# Zaključak i mentalni model Go programera

U prethodnih 15 delova ovog poglavlja prošli smo kroz najvažnije koncepte potrebne za razumevanje Go jezika:

- šta je Go,
- zašto je nastao,
- kako izgleda Go ekosistem,
- filozofiju dizajna jezika,
- prednosti i ograničenja,
- poređenje sa drugim jezicima,
- istoriju razvoja,
- oblasti primene.

Ovo poslednje poglavlje predstavlja sumiranje svega što smo naučili.

Cilj nije da zapamtimo svaku informaciju.

Cilj je da izgradimo ispravan mentalni model:

> Kako Go programer razmišlja?

---

# Go nije samo sintaksa

Početnici često posmatraju programski jezik kroz:

- ključne reči,
- funkcije,
- strukture,
- biblioteke.

To je samo površinski nivo.

Primer:

```text
Syntax

   |

   v

Language Features

   |

   v

Programming Model

   |

   v

Engineering Philosophy
````

---

Pravi nivo razumevanja dolazi kada razumemo:

* zašto je nešto dizajnirano tako,
* koji problem rešava,
* kada određeni pristup koristiti.

---

# Go osnovna filozofija

Možemo je sažeti ovako:

```text
Simple

+

Readable

+

Explicit

+

Reliable

+

Fast
```

---

# 1. Simple (Jednostavan)

Go ne pokušava da ima najveći broj mogućnosti.

Njegov cilj:

```text
Less Complexity

        |

        v

Better Understanding
```

---

Primer:

Go ima relativno mali broj koncepata:

```text
Variables

Functions

Structs

Interfaces

Goroutines

Channels

Packages
```

---

# 2. Readable (Čitljiv)

Go smatra da je kod koji drugi ljudi mogu lako razumeti najvredniji.

Kod se:

* piše jednom,
* čita mnogo puta.

---

Zato Go koristi:

```text
gofmt

+

conventions

+

simple syntax
```

---

# 3. Explicit (Eksplicitan)

Go preferira da programer vidi šta se dešava.

Primer:

```go id="o4j7yt"
value, err := operation()

if err != nil {
	return err
}
```

---

Nema skrivenog ponašanja.

Tok programa je jasan.

---

# 4. Reliable (Pouzdan)

Go je dizajniran za sisteme koji moraju dugo raditi.

Primer:

```text
Production Service

       |

       v

Running

       |

       v

Years of Maintenance
```

---

Pouzdanost dolazi iz:

* stabilnosti jezika,
* jednostavnog koda,
* dobrog tooling-a.

---

# 5. Fast (Brz)

Go kombinuje:

* kompajlirani kod,
* efikasan runtime,
* moderan scheduler.

---

Model:

```text
Go Source

    |

    v

Compiler

    |

    v

Native Binary

    |

    v

CPU
```

---

# Mentalni model Go arhitekture

Kada razmišljamo o Go aplikaciji, treba je posmatrati kroz slojeve:

```text
                  Application


                       |

                       v


              Business Logic


                       |

                       v


              Go Standard Library


                       |

                       v


                 Go Runtime


                       |

                       v


                    OS


                       |

                       v


                  Hardware
```

---

# Go Runtime

Go runtime je jedan od najvažnijih delova jezika.

On upravlja:

* garbage collector-om,
* goroutine-ama,
* scheduler-om,
* memorijom.

---

Bez runtime-a Go ne bi imao:

```text
goroutines

channels

automatic memory management
```

---

# Mentalni model memorije

Go programer treba razumeti:

```text
Variables

      |

      v

Memory

      |

      v

Stack / Heap

      |

      v

Garbage Collector
```

---

Kasnije u tutorijalu detaljno ćemo obrađivati:

* pointer-e,
* escape analysis,
* stack i heap,
* garbage collector.

---

# Mentalni model konkurentnosti

Jedna od najvećih Go karakteristika:

```text
Do not communicate by sharing memory.

Share memory by communicating.
```

---

Go pristup:

```text
Goroutine A

       |

       v

    Channel

       |

       v

Goroutine B
```

---

Kasnije ćemo detaljno obraditi:

* goroutines,
* channels,
* select,
* sync paket,
* context,
* concurrency patterns.

---

# Mentalni model tipova

Go koristi jednostavan, ali moćan sistem tipova.

Osnovni elementi:

```text
Primitive Types

       +

Structs

       +

Interfaces

       +

Generics
```

---

Najvažnija ideja:

> Behavior je važniji od hijerarhije.

---

# Interfejsi u Go-u

Go nema klasično nasleđivanje.

Umesto:

```text
Class

   |

   v

Subclass
```

koristi:

```text
Type

   |

   v

Implements Interface
```

---

Ovo omogućava:

* fleksibilnost,
* slabiju povezanost,
* lakše testiranje.

---

# Mentalni model paketa

Go program je kolekcija paketa.

Struktura:

```text
Application

    |

    +-- package A

    |

    +-- package B

    |

    +-- package C
```

---

Dobar Go kod ima:

* male pakete,
* jasne odgovornosti,
* jednostavne API-je.

---

# Kako razmišlja Go programer?

Početnik pita:

> Kako mogu napraviti ovo?

Iskusniji Go programer pita:

> Koji je najjednostavniji način da ovo rešim?

---

Primer:

Kompleksan dizajn:

```text
Many abstractions

       |

       v

Hard maintenance
```

---

Go pristup:

```text
Simple design

       |

       v

Clear behavior
```

---

# Go razvojni ciklus

Profesionalni Go workflow:

```text
Idea

 |

 v

Design

 |

 v

Implementation

 |

 v

gofmt

 |

 v

go test

 |

 v

go vet

 |

 v

Build

 |

 v

Deploy
```

---

# Šta treba zapamtiti iz Introduction poglavlja?

Najvažnije lekcije:

---

## 1. Go je napravljen za moderne sisteme

Posebno:

* backend,
* cloud,
* infrastrukturu,
* distribuirane sisteme.

---

## 2. Jednostavnost je namerna

Manje mogućnosti znači:

* manje kompleksnosti,
* lakše održavanje.

---

## 3. Eksplicitnost je važna

Go želi da programer kontroliše tok programa.

---

## 4. Konkurentnost je ugrađena u jezik

Goroutine i channel predstavljaju centralni deo Go filozofije.

---

## 5. Ekosistem je deo jezika

Go uključuje:

* compiler,
* runtime,
* tooling,
* standardnu biblioteku.

---

# Go Mastery putanja

Nakon Introduction dela prelazimo na praktični rad.

Sledeće oblasti će postepeno uvoditi:

```text
Getting Started

        |

        v

Language Fundamentals

        |

        v

Data Types

        |

        v

Functions

        |

        v

Interfaces

        |

        v

Concurrency

        |

        v

Advanced Go
```

---

# Završna misao

Go nije napravljen da bude najkompleksniji jezik.

Napravljen je da bude jedan od najpraktičnijih.

Njegova najveća vrednost je sposobnost da omogući programerima da pišu:

* jednostavan kod,
* brz kod,
* pouzdan kod,
* kod koji timovi mogu održavati godinama.

---

# Summary

Nakon završetka ovog poglavlja trebalo bi da razumete:

✅ šta je Go programski jezik
✅ zašto je nastao
✅ koju filozofiju prati
✅ gde se koristi
✅ njegove prednosti i ograničenja
✅ kako se razlikuje od drugih jezika
✅ kako izgleda Go ekosistem
✅ kakav mentalni model treba imati Go programer

---

# Sledeće poglavlje

Sledeće poglavlje:

```text
02. Starting a Project
```

fokusiraće se na praktičan početak rada sa Go projektima:

* instalacija Go okruženja,
* kreiranje projekta,
* struktura projekta,
* go.mod fajl,
* build proces,
* pokretanje aplikacije.

---
