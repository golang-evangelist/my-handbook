# Go Mastery

> **Detaljan, temeljan i analitičan tutorijal za programski jezik Go (Golang)**

![Go Version](https://img.shields.io/badge/Go-1.25+-00ADD8?style=for-the-badge&logo=go)
![Status](https://img.shields.io/badge/Status-In%20Progress-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

---

# 📖 O projektu

**Go Mastery** je dugoročni edukativni projekat čiji je cilj da na jednom mestu objedini kompletno znanje neophodno za ozbiljno i profesionalno korišćenje programskog jezika **Go (Golang)**.

Za razliku od većine tutorijala koji se fokusiraju isključivo na sintaksu jezika ili na izradu jednostavnih primera, ovaj tutorijal ima za cilj da čitaoca postepeno dovede od potpunog početnika do veoma naprednog poznavanja Go jezika, njegove standardne biblioteke, alata, načina razmišljanja i najboljih razvojnih praksi.

Tokom tutorijala neće biti obrađeno samo **kako** se nešto radi, već i:

- zašto je Go dizajniran na određeni način;
- koje probleme određeni koncept rešava;
- koje su prednosti i mane pojedinih pristupa;
- kako Go Compiler i Go Runtime izvršavaju određene operacije;
- kako donositi kvalitetne tehničke odluke prilikom razvoja aplikacija.

Drugim rečima, cilj ovog tutorijala nije samo da nauči čitaoca da piše Go kod, već da razume **način razmišljanja koji stoji iza Go jezika**.

---

# 🎯 Cilj tutorijala

Nakon završetka kompletnog tutorijala čitalac bi trebalo da bude sposoban da:

- samostalno razvija CLI aplikacije;
- razvija REST API servise;
- razvija Web aplikacije;
- organizuje veće Go projekte;
- koristi Go Modules na profesionalan način;
- efikasno koristi standardnu biblioteku;
- piše kvalitetne testove;
- razume i koristi konkurentno programiranje;
- debaguje i profilira aplikacije;
- razvija bezbedan Go kod;
- radi sa relacionim bazama podataka;
- razvija distribuirane servise;
- razume interne mehanizme Go jezika koji utiču na performanse i ponašanje aplikacija.

---

# 👥 Kome je tutorijal namenjen?

Tutorijal je namenjen širokom spektru programera.

## ✅ Potpunim početnicima

Nije neophodno prethodno iskustvo sa Go jezikom.

Svaki novi koncept biće uveden postepeno, uz detaljna objašnjenja i veliki broj primera.

---

## ✅ Programerima koji dolaze iz drugih jezika

Posebna pažnja biće posvećena razlikama između Go jezika i drugih popularnih programskih jezika kao što su:

- C
- C++
- Java
- C#
- Kotlin
- Rust
- Python
- JavaScript
- TypeScript

Kada god postoji značajna razlika u filozofiji ili implementaciji, ona će biti posebno objašnjena.

---

## ✅ Junior Go Developerima

Tutorijal će pomoći u:

- popunjavanju rupa u znanju;
- razumevanju standardne biblioteke;
- organizaciji projekata;
- razumevanju dobrih razvojnih praksi;
- pripremi za prelazak na Mid nivo.

---

## ✅ Mid Go Developerima

Poseban fokus biće stavljen na:

- kvalitet arhitekture;
- organizaciju većih projekata;
- konkurentno programiranje;
- testiranje;
- performanse;
- debugging;
- profiling;
- sigurnost aplikacija.

---

## ✅ Senior Go Developerima

Iako veliki broj tema pokriva osnove jezika, značajan deo tutorijala ulazi duboko u način funkcionisanja Go ekosistema.

To uključuje:

- razumevanje dizajna jezika;
- razumevanje rada standardne biblioteke;
- analizu razvojnih obrazaca;
- praktične smernice za razvoj velikih aplikacija;
- napredne tehnike koje se koriste u profesionalnim projektima.

---

# 📚 Filozofija tutorijala

Ovaj tutorijal nije zamišljen kao zbir kratkih članaka ili blog postova.

Svaka oblast predstavlja jednu logičku celinu, dok svako poglavlje detaljno obrađuje jednu konkretnu temu.

Naglasak je na razumevanju, a ne na pukom pamćenju sintakse.

Svaki koncept biće predstavljen kroz:

- teorijsko objašnjenje;
- vizuelni prikaz kada je potrebno;
- jednostavne primere;
- naprednije primere;
- analizu najčešćih grešaka;
- preporučene prakse;
- praktične savete iz realnih projekata.

---

# 🧭 Struktura tutorijala

Kompletan tutorijal podeljen je u više velikih celina (modula).

Svaki modul predstavlja jedan nivo znanja i prirodno se nadovezuje na prethodni.

U trenutnoj verziji projekta planirani su sledeći moduli:

```text
docs/
├── basic/
├── intermediate/
└── ...

---

# 📘 Modul: `docs/basic`

Modul **`basic`** predstavlja temelj celokupnog tutorijala.

Njegov cilj nije samo upoznavanje sa sintaksom Go jezika, već izgradnja čvrstog razumevanja svih osnovnih koncepata koji će biti neophodni za razumevanje naprednijih oblasti u nastavku tutorijala.

Za razliku od mnogih uvodnih kurseva koji se fokusiraju isključivo na pisanje jednostavnih programa, ovaj modul detaljno objašnjava **zašto** Go funkcioniše na određeni način, koje su odluke dizajnera jezika dovele do takvog ponašanja i kako se te odluke odražavaju na svakodnevni razvoj aplikacija.

Po završetku ovog modula čitalac će imati dovoljno znanja da samostalno razvija manje i srednje velike Go aplikacije, koristi standardne alate jezika, organizuje projekte, razume osnovne principe konkurentnog programiranja i piše kvalitetan, čitljiv i održiv kod.

---

# 🎯 Ciljevi modula

Tokom ovog modula biće obrađene gotovo sve fundamentalne oblasti Go jezika.

Nakon uspešno završenog modula, čitalac će:

- razumeti filozofiju Go jezika;
- pravilno organizovati Go projekte;
- koristiti Go Modules;
- razumeti sistem paketa;
- raditi sa svim ugrađenim tipovima podataka;
- koristiti pokazivače (Pointers);
- razumeti nizove, slice-ove, mape i strukture;
- pisati funkcije i metode;
- razumeti interfejse;
- koristiti kontrolu toka;
- pravilno obrađivati greške;
- razumeti osnove konkurentnog programiranja;
- koristiti standardnu biblioteku;
- koristiti Delve debugger;
- pisati osnovne testove;
- razumeti dobre razvojne prakse.

Drugim rečima, završetkom ovog modula čitalac će steći veoma čvrste temelje za sve naredne oblasti tutorijala.

---

# 🗂 Organizacija modula

Modul je organizovan u više međusobno povezanih oblasti.

Svaka oblast predstavlja jednu veću temu.

Unutar svake oblasti nalazi se više poglavlja, pri čemu je svako poglavlje zaseban Markdown dokument.

Na taj način svaka tema ostaje pregledna, lako pretraživa i jednostavna za održavanje.

Planirana struktura modula izgleda ovako:

```text
docs/
└── basic/
    ├── 01-getting-started/
    ├── 02-go-fundamentals-v1/
    ├── 03-go-fundamentals-v2/
    ├── 04-debugging-in-go/
    ├── 05-debugging-go-application-with-delve/
    ├── 06-exploring-go-modules/
    ├── 07-the-go-standard-library/
    ├── 08-creating-custom-data-types/
    └── 09-advanced-branching-and-looping/

Svaka oblast će biti detaljno obrađena kroz teoriju, praktične primere i vežbe.

---

📖 Oblast 01 — Getting Started

Prva oblast predstavlja uvod u Go jezik.

Ona je namenjena svim čitaocima, bez obzira na prethodno iskustvo.

U ovoj oblasti biće objašnjeno:

- šta je Go;
- zbog čega je nastao;
- koje probleme rešava;
- koje su njegove osnovne karakteristike;
- kako izgleda razvojni ciklus Go aplikacije;
- kako pravilno započeti novi projekat;
- kako koristiti osnovne tipove podataka;
- kako raditi sa kolekcijama;
- kako pisati funkcije i metode;
- kako funkcioniše kontrola toka programa.

Ova oblast predstavlja osnovu za razumevanje svih narednih oblasti.

---

📖 Oblast 02 — Go Fundamentals v1

Druga oblast produbljuje razumevanje jezika.

Fokus više nije samo na sintaksi, već na pravilnom korišćenju Go jezika u svakodnevnom razvoju aplikacija.

Posebna pažnja biće posvećena:

- organizaciji programa;
- razvoju većih aplikacija;
- radu sa složenijim tipovima podataka;
- upravljanju greškama;
- osnovama objektno orijentisanog pristupa u Go jeziku;
- osnovama konkurentnog programiranja;
- osnovama testiranja.

Cilj ove oblasti jeste da čitalac počne da razmišlja kao Go programer, a ne samo da poznaje sintaksu jezika.

---

📖 Oblast 03 — Go Fundamentals v2

Treća oblast predstavlja dodatno proširenje prethodne oblasti.

Iako pojedine teme postoje i u prethodnim poglavljima, ovde će biti obrađene iz drugačije perspektive i sa većim brojem praktičnih primera.

Poseban fokus biće na:

- razumevanju razvojnog okruženja;
- radu sa tipovima podataka;
- radu sa kolekcijama;
- kontroli toka programa;
- funkcijama;
- razumevanju načina na koji Go izvršava kod.

Na taj način čitalac će steći mnogo dublje razumevanje osnovnih koncepata pre prelaska na naprednije oblasti.

---

# 📖 Oblast 04 — Debugging in Go

Pisanje kvalitetnog koda podrazumeva mnogo više od samog razvoja aplikacije.

Jedna od najvažnijih veština svakog programera jeste sposobnost da pronađe, razume i otkloni greške na brz i efikasan način.

U ovoj oblasti čitalac će naučiti kako funkcioniše proces debagovanja Go aplikacija, kako analizirati izvršavanje programa i kako koristiti profesionalne alate za dijagnostiku problema.

Biće obrađene sledeće teme:

- uvod u Debugging;
- način rada Go Debuggera;
- Delve (DLV);
- Command Line Debugging;
- Editor Integration;
- Remote Debugging;
- Debugging aplikacija unutar Docker kontejnera;
- Debugging aplikacija u produkcionom okruženju.

Posebna pažnja biće posvećena razumevanju toka izvršavanja programa, praćenju promenljivih, analizi steka poziva (Call Stack), kao i pronalaženju uzroka logičkih i runtime grešaka.

Na kraju ove oblasti čitalac će biti sposoban da samostalno analizira i rešava probleme u složenijim Go aplikacijama.

---

# 📖 Oblast 05 — Debugging Go Applications with Delve

Nakon upoznavanja sa osnovama debagovanja, naredna oblast u potpunosti je posvećena najvažnijem debugger-u za Go — **Delve (DLV)**.

Za razliku od prethodne oblasti, ovde će fokus biti na svakodnevnom radu sa debugger-om i naprednim mogućnostima koje Delve pruža.

Biće detaljno obrađene sledeće teme:

- arhitektura Delve alata;
- pokretanje debugger-a;
- istraživanje trenutnog stanja aplikacije;
- pregled promenljivih;
- analiza memorije;
- rad sa Breakpoint-ima;
- Conditional Breakpoints;
- Continue, Next, Step i StepOut komande;
- rad sa Stack Frame-ovima;
- analiza Goroutine-a;
- Delve komande;
- integracija sa Visual Studio Code-om i drugim editorima.

Cilj ove oblasti jeste da čitalac stekne sigurnost u korišćenju debugger-a tokom razvoja i održavanja profesionalnih Go aplikacija.

---

# 📖 Oblast 06 — Exploring Go Modules

Savremeni razvoj softvera nije moguće zamisliti bez kvalitetnog sistema za upravljanje zavisnostima.

Go Modules predstavljaju zvaničan način organizacije projekata i upravljanja spoljnim bibliotekama.

U ovoj oblasti detaljno će biti obrađeno:

- zašto su uvedeni Go Modules;
- struktura modula;
- datoteka `go.mod`;
- datoteka `go.sum`;
- verzionisanje modula;
- Semantic Versioning;
- preuzimanje zavisnosti;
- ažuriranje biblioteka;
- uklanjanje nepotrebnih zavisnosti;
- privatni moduli;
- rad sa više modula;
- napredni alati za upravljanje modulima.

Pored sintakse i komandi, posebna pažnja biće posvećena razumevanju načina na koji Go rešava zavisnosti između projekata.

Po završetku ove oblasti čitalac će biti sposoban da organizuje i održava profesionalne Go projekte različitih veličina.

---

# 📖 Oblast 07 — The Go Standard Library

Jedna od najvećih prednosti Go jezika jeste izuzetno bogata i kvalitetno dizajnirana standardna biblioteka.

Mnogi problemi koji u drugim jezicima zahtevaju dodatne biblioteke mogu se rešiti korišćenjem paketa koji dolaze uz sam Go.

Ova oblast predstavlja uvod u najvažnije delove standardne biblioteke.

Biće obrađene sledeće teme:

- razvoj CLI aplikacija;
- rad sa komandnom linijom;
- paket `fmt`;
- paket `log`;
- paket `time`;
- rad sa stringovima;
- osnove Reflection-a.

Iako će u ovoj oblasti fokus biti na najčešće korišćenim paketima, veliki broj drugih paketa standardne biblioteke biće detaljno obrađen u narednim modulima kada budu potrebni u konkretnim projektima.

Cilj nije samo upoznavanje API-ja pojedinačnih paketa, već razumevanje njihove pravilne i efikasne upotrebe.

---

# 📖 Oblast 08 — Creating Custom Data Types

Jedna od karakteristika Go jezika jeste jednostavan, ali veoma moćan sistem tipova.

Ova oblast posvećena je kreiranju sopstvenih tipova podataka i razumevanju načina na koji Go modeluje ponašanje objekata.

Poseban fokus biće na:

- Struct tipovima;
- Interface tipovima;
- Type Definition;
- Type Alias;
- ugrađenim (Embedded) tipovima;
- Composition;
- Comparable tipovima;
- Type Switch konstrukciji.

Posebna pažnja biće posvećena razlikama između Composition i nasleđivanja (Inheritance), kao i razlozima zbog kojih Go favorizuje Composition kao osnovni mehanizam za ponovno korišćenje koda.

Po završetku ove oblasti čitalac će razumeti kako se u Go jeziku modeluju složeniji sistemi bez korišćenja klasičnog objektno-orijentisanog nasleđivanja.

---

# 📖 Oblast 09 — Advanced Branching and Looping

Poslednja oblast modula predstavlja praktično produbljivanje znanja o kontroli toka programa.

Za razliku od ranijih poglavlja, ovde će naglasak biti na rešavanju konkretnih problema kroz veliki broj primera i mini projekata.

Biće detaljno obrađeni:

- `for` petlja u svim svojim oblicima;
- iteracija kroz različite kolekcije;
- rad sa indeksima;
- rad sa `range` izrazom;
- `if` i `if-else`;
- ugnježdene grane;
- `switch`;
- `switch` bez izraza;
- `switch` sa više uslova;
- praktični obrasci korišćenja kontrolnih struktura.

Svaka tema biće propraćena:

- više demonstracionih primera;
- analizom rešenja;
- mini projektima;
- vežbama različite težine.

Na taj način čitalac će steći sigurnost u pisanju čitljivog i efikasnog Go koda.

---

# 🎓 Šta čitalac dobija nakon završetka modula?

Po završetku modula **`docs/basic`**, čitalac neće poznavati samo osnovnu sintaksu Go jezika.

Stečeno znanje obuhvataće:

- razumevanje filozofije Go jezika;
- pravilnu organizaciju Go projekata;
- rad sa svim osnovnim tipovima podataka;
- efikasno korišćenje kolekcija;
- pisanje funkcija i metoda;
- modelovanje sopstvenih tipova;
- korišćenje interfejsa;
- razumevanje upravljanja greškama;
- osnove konkurentnog programiranja;
- rad sa Go Modules;
- korišćenje najvažnijih paketa standardne biblioteke;
- korišćenje Delve debugger-a;
- razvoj jednostavnijih CLI aplikacija;
- razumevanje dobrih razvojnih praksi.

Ovaj nivo znanja predstavlja čvrst temelj za prelazak na naredni modul, u kojem će fokus biti na razvoju ozbiljnijih aplikacija, organizaciji većih projekata, radu sa bazama podataka, web razvoju, bezbednosti, naprednom testiranju, profilisanju performansi i drugim temama koje čine svakodnevni rad profesionalnog Go developera.

---

# 📗 Modul: `docs/intermediate`

Nakon uspešno završenog modula **`docs/basic`**, čitalac će posedovati veoma dobro razumevanje Go jezika i njegovih osnovnih koncepata.

Međutim, poznavanje sintakse jezika predstavlja samo jedan deo znanja koje je potrebno profesionalnom Go programeru.

Modul **`docs/intermediate`** predstavlja prirodan nastavak prethodnog modula i fokusira se na razvoj realnih aplikacija, organizaciju većih projekata, primenu standardne biblioteke u svakodnevnom radu, testiranje, bezbednost, konkurentno programiranje, razvoj web servisa, razvoj web aplikacija, profilisanje performansi i druge oblasti koje se svakodnevno sreću u profesionalnom razvoju softvera.

Naglasak više nije na učenju pojedinačnih jezičkih konstrukcija, već na njihovoj pravilnoj primeni u stvarnim projektima.

Svaka oblast gradi se na znanju stečenom u modulu **`basic`**, zbog čega se preporučuje da se ovaj modul prati isključivo nakon njegovog završetka.

---

# 🎯 Ciljevi modula

Cilj ovog modula jeste da čitaoca postepeno uvede u razvoj profesionalnih Go aplikacija.

Po završetku modula čitalac će biti sposoban da:

- organizuje veće Go projekte;
- dizajnira kvalitetne pakete;
- razvija REST API servise;
- razvija web aplikacije;
- radi sa relacionim bazama podataka;
- razvija konkurentne aplikacije;
- koristi napredne mogućnosti Go standardne biblioteke;
- piše kvalitetne i pouzdane testove;
- razume bezbednosne principe razvoja softvera;
- meri i optimizuje performanse aplikacija;
- razvija distribuirane servise;
- koristi profesionalne razvojne alate.

Drugim rečima, fokus ovog modula jeste prelazak sa razumevanja jezika na razumevanje profesionalnog razvoja Go aplikacija.

---

# 🗂 Organizacija modula

Planirana struktura modula izgleda ovako:

```text
docs/
└── intermediate/
    ├── 01-deep-dive-into-go-packages/
    ├── 02-deep-dive-into-go-functions/
    ├── 03-accessing-relational-databases/
    ├── 04-managing-go-projects/
    ├── 05-creating-web-services/
    ├── 06-creating-web-applications/
    ├── 07-gin-framework-old/
    ├── 08-gin-fundamentals/
    ├── 09-concurrent-programming-old/
    ├── 10-concurrent-programming/
    ├── 11-building-go-web-services-and-applications/
    ├── 12-secure-coding/
    ├── 13-creating-well-tested-applications/
    ├── 14-testing-go-applications/
    ├── 15-testing-in-go/
    ├── 16-building-distributed-applications/
    ├── 17-managing-errors/
    ├── 18-profiling-go-applications/
    ├── 19-go-cli-playbook-old/
    └── 20-go-cli-playbook/

«Napomena

Pojedine oblasti predstavljaju starije (OLD) i novije (NEW) verzije istih tema.
Tokom pisanja tutorijala njihov sadržaj neće biti mehanički ponavljan.
Umesto toga, biće objedinjeni najbolji delovi obe verzije kako bi svaka oblast bila što kvalitetnija, potpunija i savremenija.»

---

📖 Oblast 01 — Deep Dive into Go Packages

Jedna od najvećih razlika između manjih i velikih Go projekata jeste kvalitet organizacije paketa.

U ovoj oblasti biće detaljno obrađeno:

- filozofija Go paketa;
- organizacija izvornog koda;
- javni i privatni API;
- Package Visibility;
- Package Initialization;
- organizacija većih projekata;
- međuzavisnosti između paketa;
- izbegavanje kružnih zavisnosti;
- dizajniranje paketa koji se lako održavaju.

Poseban akcenat biće stavljen na način razmišljanja prilikom dizajniranja paketa, a ne samo na njihovu sintaksu.

---

📖 Oblast 02 — Deep Dive into Go Functions

Iako funkcije predstavljaju osnovni građevinski blok svakog Go programa, njihova pravilna upotreba ima veliki uticaj na čitljivost i održavanje aplikacije.

U ovoj oblasti detaljno će biti obrađene:

- organizacija funkcija;
- parametri;
- povratne vrednosti;
- višestruke povratne vrednosti;
- named return values;
- Method Receivers;
- Value vs Pointer Receivers;
- Function Expressions;
- Method Expressions;
- Method Values;
- kontrola toka unutar funkcija;
- projektovanje API-ja funkcija.

Poseban fokus biće na pisanju funkcija koje su jednostavne za testiranje, ponovno korišćenje i održavanje.

---

📖 Oblast 03 — Accessing Relational Databases in Go

Razvoj ozbiljnih aplikacija gotovo uvek podrazumeva rad sa bazom podataka.

U ovoj oblasti čitalac će naučiti kako Go komunicira sa relacionim bazama podataka koristeći standardne alate i najbolje razvojne prakse.

Biće obrađene sledeće teme:

- osnove rada sa SQL bazama;
- povezivanje aplikacije sa bazom;
- upravljanje konekcijama;
- CRUD operacije;
- pripremljeni upiti (Prepared Statements);
- transakcije;
- procedure;
- obrada grešaka;
- organizacija sloja za pristup podacima.

Poseban fokus biće na pravilnom dizajniranju Data Access sloja i pisanju čitljivog i održivog koda.

---

📖 Oblast 04 — Managing Go Projects

Kako projekat raste, njegova organizacija postaje jednako važna kao i sam kod.

U ovoj oblasti biće obrađene teme vezane za organizaciju profesionalnih Go projekata.

To uključuje:

- naprednu organizaciju paketa;
- Go Modules;
- upravljanje zavisnostima;
- rad sa više modula;
- ugrađivanje resursa pomoću "embed" paketa;
- preporučene strukture projekata;
- održavanje većih repozitorijuma.

Cilj nije da postoji samo jedna "ispravna" struktura projekta, već da čitalac razume prednosti i nedostatke različitih pristupa i ume da izabere odgovarajuće rešenje za konkretan projekat.

---

# 📖 Oblast 05 — Creating Web Services with Go

Savremene aplikacije u velikoj meri komuniciraju putem HTTP servisa i REST API-ja.

U ovoj oblasti čitalac će naučiti kako se koriste mogućnosti standardne biblioteke za razvoj robusnih web servisa.

Biće detaljno obrađene sledeće teme:

- arhitektura HTTP servera;
- rad sa `net/http` paketom;
- rutiranje zahteva;
- obrada HTTP zahteva i odgovora;
- rad sa JSON podacima;
- upravljanje greškama;
- validacija ulaznih podataka;
- rad sa bazom podataka;
- WebSocket komunikacija;
- Template Engine.

Na kraju ove oblasti čitalac će biti sposoban da razvije kompletan REST servis koristeći isključivo standardnu biblioteku Go jezika.

---

# 📖 Oblast 06 — Creating Web Applications with Go

Nakon razvoja web servisa, fokus se pomera na razvoj kompletnih web aplikacija.

Ova oblast obuhvata:

- organizaciju web aplikacije;
- HTTP Handlers;
- HTML Templates;
- Template Inheritance;
- Smart Templates;
- Middleware;
- Session Management;
- rad sa bazama podataka;
- autentifikaciju korisnika;
- HTTP/2 i novije mogućnosti HTTP protokola;
- testiranje web aplikacija.

Naglasak će biti na razumevanju celokupnog životnog ciklusa jedne web aplikacije, od prijema zahteva do generisanja odgovora.

---

# 📖 Oblasti 07 i 08 — Gin Framework

Gin predstavlja jedan od najpopularnijih framework-a za razvoj web aplikacija u Go ekosistemu.

Pošto postoje dve verzije kursa (OLD i Fundamentals), sadržaj će biti objedinjен u jednu celinu.

Biće obrađene sledeće teme:

- arhitektura Gin framework-a;
- rutiranje;
- parametri zahteva;
- Query Parameters;
- Path Parameters;
- Binding;
- Validation;
- Responses;
- JSON API;
- Middleware;
- Error Handling;
- organizacija većih Gin aplikacija;
- testiranje Gin aplikacija.

Posebna pažnja biće posvećena razlikama između razvoja aplikacija korišćenjem standardne biblioteke i korišćenjem Gin framework-a.

---

# 📖 Oblasti 09 i 10 — Concurrent Programming

Konkurentno programiranje predstavlja jednu od najvažnijih karakteristika Go jezika.

Sadržaj starih i novih kurseva biće objedinjen u jednu detaljnu oblast.

Biće obrađene teme kao što su:

- Goroutines;
- Scheduler (na nivou potrebnom za Intermediate modul);
- Channels;
- Buffered i Unbuffered Channels;
- Channel Directions;
- Select;
- Context;
- Cancellation;
- Timeouts;
- WaitGroup;
- Mutex;
- RWMutex;
- Once;
- Sync Package;
- Concurrency Patterns;
- Worker Pools;
- Pipelines;
- Fan-Out;
- Fan-In;
- pravilno projektovanje konkurentnih aplikacija.

Naglasak neće biti samo na sintaksi, već na pravilnom načinu razmišljanja prilikom razvoja konkurentnih sistema.

---

# 📖 Oblast 11 — Building Go Web Services and Applications

Ova oblast predstavlja objedinjavanje prethodno stečenog znanja kroz razvoj kompletne aplikacije.

Čitalac će naučiti kako se:

- razvija REST servis;
- obrađuje JSON;
- povezuje baza podataka;
- razvija web aplikacija;
- koriste Template Engine-i;
- organizuje kompletan projekat.

Poseban akcenat biće na povezivanju više prethodno naučenih oblasti u jednu funkcionalnu celinu.

---

# 📖 Oblasti 12–20

Preostale oblasti predstavljaju teme koje svaki profesionalni Go developer koristi tokom svakodnevnog rada.

One obuhvataju:

## 🔐 Secure Coding

- bezbednosni principi razvoja softvera;
- autentifikaciju;
- autorizaciju;
- zaštitu podataka;
- Input Validation;
- Output Encoding;
- upravljanje sesijama;
- kriptografske prakse;
- zaštitu baza podataka;
- zaštitu fajlova;
- bezbednosne preporuke.

---

## 🧪 Testing

Kroz više međusobno povezanih oblasti biće obrađeno:

- Unit Testing;
- Table Driven Tests;
- HTTP Testing;
- Black Box Testing;
- Code Coverage;
- Benchmark Testing;
- Fuzz Testing;
- Profiling;
- organizacija testova;
- razvoj testabilnog koda.

Posebna pažnja biće posvećena razumevanju zašto se testovi pišu na određeni način i kako dizajn aplikacije utiče na mogućnost testiranja.

---

## 🌐 Distributed Applications

Čitalac će se upoznati sa osnovama distribuiranih sistema.

Biće obrađene teme:

- Service Registration;
- Service Discovery;
- Health Checks;
- Monitoring;
- komunikacija između servisa.

Ova oblast predstavlja uvod u naprednije teme koje će biti obrađene u narednim modulima tutorijala.

---

## ⚠ Managing Errors

Greške predstavljaju sastavni deo svake ozbiljne aplikacije.

Biće detaljno obrađeno:

- Error Types;
- Error Wrapping;
- Validation;
- Panic;
- Recover;
- Error Assertions;
- HTTP Errors;
- Channel Errors;
- Goroutine Errors;
- projektovanje kvalitetnog Error API-ja.

---

## 📈 Profiling

Optimizacija performansi počinje merenjem.

U ovoj oblasti biće obrađene:

- CPU Profiling;
- Memory Profiling;
- pprof;
- analiza performansi;
- identifikacija uskih grla;
- praktična optimizacija aplikacija.

---

## 💻 Go CLI Playbook

Poslednje oblasti modula posvećene su profesionalnom korišćenju Go alata.

Čitalac će naučiti:

- kako funkcioniše Go Toolchain;
- kako prilagoditi Build Environment;
- kako upravljati Runtime okruženjem;
- kako analizirati projekte;
- kako efikasnije koristiti Go CLI.

---

# 🎓 Šta čitalac dobija nakon završetka modula?

Po završetku modula **`docs/intermediate`**, čitalac će biti sposoban da:

- organizuje profesionalne Go projekte;
- razvija REST API-je;
- razvija web aplikacije;
- koristi Gin framework;
- radi sa relacionim bazama podataka;
- koristi napredne mogućnosti Go Modules sistema;
- razvija konkurentne aplikacije;
- piše kvalitetne testove;
- meri i optimizuje performanse aplikacija;
- primenjuje bezbednosne principe razvoja softvera;
- koristi profesionalne razvojne alate;
- razvija distribuirane servise;
- razume i implementira dobre razvojne prakse.

Drugim rečima, završetkom ovog modula čitalac će posedovati znanje koje je potrebno za samostalno razvijanje ozbiljnih Go aplikacija i razumevanje načina na koji se profesionalni Go projekti projektuju, razvijaju, testiraju i održavaju.

---

# 🚀 Šta sledi nakon ovog modula?

Modul **`docs/intermediate`** predstavlja završetak glavnog dela tutorijala koji je fokusiran na sam Go jezik i njegovu praktičnu primenu.

Nakon njegovog završetka planiran je treći modul, čija će konačna struktura biti definisana naknadno.

Njegov fokus biće na specijalizovanim oblastima Go ekosistema i naprednim tehnologijama koje se koriste u profesionalnim projektima.

Planirane teme uključuju, između ostalog:

- objektno-orijentisane obrasce u Go jeziku;
- Code Generation;
- gRPC;
- Protocol Buffers;
- razvoj mikroservisa;
- Docker integraciju;
- ORM alate;
- rad sa NoSQL bazama podataka;
- i druge specijalizovane oblasti.

Konačna organizacija ovog modula biće definisana nakon završetka modula **`docs/intermediate`**, kako bi čitav tutorijal zadržao jasnu, logičnu i postepenu strukturu učenja.

---

# 📌 Završna napomena

Ovaj tutorijal nije zamišljen kao brz kurs niti kao zbir nepovezanih primera.

Njegov cilj jeste da čitaocu omogući duboko razumevanje Go jezika, standardne biblioteke, razvojnih alata i profesionalnih praksi koje se koriste u svakodnevnom razvoju softvera.

Svaka oblast pažljivo je odabrana kako bi predstavljala prirodan nastavak prethodne, omogućavajući postepeno usvajanje znanja bez preskakanja važnih koncepata.

Bez obzira na to da li je cilj učenje Go jezika od samog početka, unapređenje postojećeg znanja ili priprema za razvoj kompleksnih produkcionih sistema, ovaj tutorijal je osmišljen kao dugoročna referenca kojoj će se čitalac moći vraćati tokom čitave svoje profesionalne karijere.
