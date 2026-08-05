# 02. Starting a Project

> **Go Mastery**<br/>  
> **Basic → 01 Getting Started**<br/>  
> Poglavlje: **02 Starting a Project**  

---

# Introduction: Starting a Go Project

U prethodnom poglavlju:

```text
01. Introduction
````

upoznali smo se sa:

* istorijom Go jezika,
* filozofijom dizajna,
* prednostima i ograničenjima,
* oblastima primene,
* Go ekosistemom.

Sada prelazimo sa teorije na praktičan rad.

Cilj ovog poglavlja je da naučimo kako se kreira, organizuje i pokreće pravi Go projekat.

---

# Šta znači "Go projekat"?

Na prvi pogled Go projekat može izgledati jednostavno:

```text
project/

    main.go
```

Međutim, profesionalni Go projekti imaju mnogo više elemenata.

Primer:

```text
my-application/

├── cmd/

├── internal/

├── pkg/

├── configs/

├── scripts/

├── go.mod

├── go.sum

└── README.md
```

---

# Go projekat nije samo skup fajlova

Važno je razumeti da Go projekat predstavlja kombinaciju:

```text
Source Code

+

Packages

+

Modules

+

Dependencies

+

Build Configuration

+

Tools
```

---

# Osnovni elementi Go projekta

Svaki Go projekat se sastoji iz nekoliko ključnih delova.

---

# 1. Source Code

Source code predstavlja Go fajlove:

```text
*.go
```

Primer:

```text
main.go

user.go

database.go
```

---

Go kompajler obrađuje ove fajlove i kreira izvršni program.

Proces:

```text
.go files

     |

     v

Go Compiler

     |

     v

Executable Binary
```

---

# 2. Package

Package predstavlja organizacionu jedinicu Go koda.

Primer:

```go
package user
```

Jedan projekat može imati više paketa:

```text
project/

├── user/

│   ├── user.go

│

├── database/

│   ├── database.go

│

└── main.go
```

---

# 3. Module

Module predstavlja celokupan Go projekat.

Moderni Go koristi:

```text
Go Modules
```

Modul definiše:

* ime projekta,
* verziju,
* dependencies.

Glavni fajl:

```text
go.mod
```

---

Primer:

```go
module github.com/example/my-app
```

---

# 4. Dependencies

Dependencies su eksterni paketi koje projekat koristi.

Primer:

```text
My Application

       |

       +-- HTTP Library

       |

       +-- Database Driver

       |

       +-- Logging Package
```

---

Go Modules omogućava upravljanje njima.

---

# 5. Build Configuration

Go projekat mora znati:

* kako da se kompajlira,
* koje module koristi,
* za koju platformu se gradi.

Primer:

```bash
go build
```

---

# Minimalni Go projekat

Najjednostavniji Go projekat izgleda ovako:

```text
hello-go/

└── main.go
```

---

Sadržaj:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, Go!")
}
```

---

Ovaj program ima:

```text
package main

        |

        v

func main()

        |

        v

Program Entry Point
```

---

# Kreiranje prvog projekta

Kreiranje direktorijuma:

```bash
mkdir hello-go
```

Ulazak u direktorijum:

```bash
cd hello-go
```

---

Inicijalizacija Go modula:

```bash
go mod init example.com/hello-go
```

---

Nakon toga dobijamo:

```text
hello-go/

├── go.mod

└── main.go
```

---

# Šta je `go.mod`?

`go.mod` je centralni fajl Go projekta.

On definiše:

* module name,
* Go verziju,
* dependencies.

Primer:

```go
module example.com/hello-go

go 1.26
```

---

# Module name

Prva linija:

```go
module example.com/hello-go
```

predstavlja identitet projekta.

---

Primer:

Ako projekat postoji na GitHub-u:

```text
github.com/user/project
```

modul može biti:

```go
module github.com/user/project
```

---

# Go verzija

Druga važna informacija:

```go
go 1.26
```

označava verziju Go jezika za koju je projekat namenjen.

---

Primer:

```go
module github.com/company/service

go 1.26
```

---

# Kreiranje `main.go`

Nakon kreiranja modula:

```bash
touch main.go
```

Dobijamo:

```text
hello-go/

├── go.mod

└── main.go
```

---

Sadržaj:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello Go Project")
}
```

---

# Pokretanje programa

Go omogućava direktno pokretanje:

```bash
go run .
```

---

Šta se dešava?

```text
Source Code

      |

      v

Temporary Build

      |

      v

Execute Program
```

---

Rezultat:

```text
Hello Go Project
```

---

# Build programa

Za kreiranje izvršnog fajla koristi se:

```bash
go build
```

---

Proces:

```text
main.go

   |

   v

Compiler

   |

   v

Binary File
```

---

Rezultat:

```text
hello-go.exe
```

ili:

```text
hello-go
```

na Linux sistemima.

---

# `go run` vs `go build`

Važna razlika:

| Komanda    | Namena                   |
| ---------- | ------------------------ |
| `go run`   | Brzo pokretanje programa |
| `go build` | Kreiranje izvršnog fajla |

---

# Primer razvoja

Tokom razvoja:

```text
Write Code

    |

    v

go run .
```

---

Za produkciju:

```text
Write Code

    |

    v

go test

    |

    v

go build

    |

    v

Deploy Binary
```

---

# Zašto Go koristi module?

Pre Go Modules sistema, Go je koristio:

```text
GOPATH
```

koji je zahtevao specifičnu strukturu direktorijuma.

Problem:

```text
Project Location

       |

       v

Must be inside GOPATH
```

---

Moderni pristup:

```text
Any Folder

       |

       v

go.mod

       |

       v

Go Module
```

---

# Profesionalni pogled

Kada kreiramo Go projekat, zapravo kreiramo:

```text
Module

    |

    +-- Packages

            |

            +-- Go Files

    |

    +-- Dependencies

    |

    +-- Build Process
```

---

# Najčešće početničke greške

---

## 1. Pokretanje van modula

Greška:

```text
go: cannot find main module
```

Uzrok:

Nedostaje:

```text
go.mod
```

---

Rešenje:

```bash
go mod init project-name
```

---

## 2. Pogrešan package name

Pogrešno:

```go
package Main
```

Ispravno:

```go
package main
```

Go razlikuje mala i velika slova.

---

## 3. Zaboravljen import

Primer:

```go
fmt.Println("Hello")
```

bez:

```go
import "fmt"
```

---

Go compiler će prijaviti grešku.

---

# Prvi mentalni model Go projekta

Zapamtiti:

```text
Go Project

      |

      v

Module

      |

      v

Packages

      |

      v

Files

      |

      v

Functions
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go projekat nije samo `.go` fajl;
* moderni Go koristi module;
* `go.mod` je centralni fajl projekta;
* package je osnovna organizaciona jedinica;
* `go run` pokreće program;
* `go build` kreira binary;
* Go projekti mogu biti kreirani u bilo kom direktorijumu.

---

# Zaključak

Kreiranje Go projekta je jednostavan proces, ali razumevanje strukture iza njega je veoma važno.

Profesionalni Go programer ne posmatra projekat samo kao:

```text
main.go
```

već kao:

```text
Module

+

Packages

+

Dependencies

+

Build Process
```

U sledećem delu ćemo detaljno obraditi **instalaciju Go razvojnog okruženja i konfiguraciju potrebnu za profesionalni rad**.

---

# Installing Go Development Environment

U prethodnom delu naučili smo osnovnu strukturu Go projekta:

- source fajlovi,
- packages,
- modules,
- `go.mod`,
- build proces.

Pre nego što počnemo ozbiljniji rad sa Go projektima, potrebno je pravilno pripremiti razvojno okruženje.

Profesionalni Go razvoj zahteva više od same instalacije kompajlera.

Potrebno je razumeti:

- kako instalirati Go,
- kako proveriti instalaciju,
- kako funkcioniše Go environment,
- koje alate Go automatski obezbeđuje.

---

# Šta predstavlja Go development environment?

Razvojno okruženje predstavlja skup svih komponenti potrebnih za razvoj Go aplikacija.

Možemo ga predstaviti:

```text
                 Go Development Environment


                         |

        +----------------+----------------+

        |                |                |

        v                v                v


       Go           Editor / IDE       Tools


        |                |                |

        v                v                v


    Compiler          VS Code          gofmt

    Runtime           GoLand          go test

    Standard Lib      Vim             go vet
````

---

# Komponente Go okruženja

Osnovne komponente:

```text
1. Go SDK

2. Environment Configuration

3. Code Editor / IDE

4. Go Tools

5. Version Control
```

---

# 1. Go SDK

Go SDK (Software Development Kit) predstavlja kompletnu instalaciju Go jezika.

Sadrži:

* compiler,
* runtime,
* standardnu biblioteku,
* build alate,
* test alate.

---

Nakon instalacije dobijamo:

```text
Go SDK

    |

    +-- Compiler

    |

    +-- Runtime

    |

    +-- Standard Library

    |

    +-- Command Line Tools
```

---

# Instalacija Go-a

Zvanična distribucija nalazi se na:

```text
https://go.dev
```

Go postoji za različite platforme:

```text
Operating System

        |

        +-- Windows

        +-- Linux

        +-- macOS

        +-- FreeBSD
```

---

# Podržane arhitekture

Go podržava veliki broj procesorskih arhitektura:

```text
Architecture

    |

    +-- amd64

    +-- arm64

    +-- 386

    +-- riscv64
```

---

# Provera instalacije

Nakon instalacije prvo proveravamo:

```bash
go version
```

---

Primer rezultata:

```text
go version go1.26 linux/amd64
```

---

Ovaj izlaz sadrži:

```text
go1.26

    |

    v

Installed Go Version


linux

    |

    v

Operating System


amd64

    |

    v

Architecture
```

---

# Zašto je provera važna?

U profesionalnom radu često postoji više verzija Go-a.

Primer:

```text
Project A

requires:

Go 1.24


Project B

requires:

Go 1.26
```

---

Zato programer mora znati:

* koja verzija je aktivna,
* koje okruženje koristi.

---

# Go verzije

Go koristi semantičko verzionisanje:

```text
Major.Minor.Patch
```

Primer:

```text
1.26.0
```

Gde:

```text
1

|

v

Major Version


26

|

v

Minor Version


0

|

v

Patch
```

---

# Minor verzije u Go-u

Go uglavnom objavljuje:

```text
Go 1.x
```

release verzije.

Primer:

```text
Go 1.20

Go 1.21

Go 1.22

Go 1.23
```

---

Svaka verzija donosi:

* nove funkcionalnosti,
* optimizacije,
* bug fix-eve.

---

# 2. Go Environment

Go environment predstavlja konfiguraciju Go alata.

Prikaz:

```bash
go env
```

---

Primer:

```text
GOOS="linux"

GOARCH="amd64"

GOROOT="/usr/local/go"

GOPATH="/home/user/go"
```

---

# Najvažnije environment promenljive

U početku je dovoljno razumeti:

* GOROOT,
* GOPATH,
* PATH,
* GOOS,
* GOARCH.

---

# GOROOT

`GOROOT` pokazuje gde je instaliran Go SDK.

Primer:

```text
GOROOT=/usr/local/go
```

---

Struktura:

```text
/usr/local/go/

├── bin/

├── src/

├── pkg/

└── lib/
```

---

Sadrži:

* compiler,
* standard library,
* runtime.

---

# PATH

`PATH` je sistemska promenljiva koja određuje gde operativni sistem traži izvršne programe.

Primer:

```text
PATH

 |

 +-- /usr/local/go/bin

 +-- /usr/bin

 +-- /bin
```

---

Kada unesemo:

```bash
go version
```

operativni sistem traži:

```text
go executable
```

u PATH lokacijama.

---

# GOPATH

Istorijski veoma važna promenljiva.

Pre Go Modules sistema:

```text
GOPATH

    |

    +-- src

    +-- pkg

    +-- bin
```

---

Danas:

* manje važan za organizaciju projekata,
* i dalje se koristi za cache i instalirane alate.

---

Primer:

```text
GOPATH=/home/user/go
```

---

Struktura:

```text
go/

├── bin/

├── pkg/

└── src/
```

---

# GOOS

`GOOS` predstavlja ciljni operativni sistem.

Primer:

```text
GOOS=linux
```

Moguće vrednosti:

```text
linux

windows

darwin

freebsd
```

---

# GOARCH

Predstavlja ciljnu arhitekturu procesora.

Primer:

```text
GOARCH=amd64
```

---

Primer kombinacije:

```text
GOOS=linux

GOARCH=arm64
```

znači:

```text
Build Linux ARM64 aplikacije
```

---

# 3. Code Editor i IDE

Go ne zahteva specifičan editor.

Može se koristiti:

* Visual Studio Code,
* GoLand,
* Vim,
* Neovim,
* Emacs.

---

# Visual Studio Code

Jedan od najpopularnijih izbora.

Potrebno je instalirati:

```text
Go Extension
```

---

Dobija se:

* syntax highlighting,
* autocomplete,
* debugging,
* refactoring.

---

# GoLand

Profesionalni IDE namenjen Go razvoju.

Obezbeđuje:

* naprednu analizu koda,
* debugging,
* refactoring,
* project management.

---

# 4. Go Tools

Instalacija Go-a automatski donosi mnoge alate.

Najvažniji:

```bash
go run

go build

go test

go fmt

go vet

go mod
```

---

# gofmt

Automatsko formatiranje:

```bash
gofmt -w .
```

---

# go test

Pokretanje testova:

```bash
go test ./...
```

---

# go vet

Analiza potencijalnih problema:

```bash
go vet ./...
```

---

# go mod

Rad sa modulima:

```bash
go mod tidy
```

---

# Profesionalno okruženje

Tipičan Go developer setup:

```text
Machine

 |

 +-- Go SDK

 |

 +-- Editor

 |

 +-- Git

 |

 +-- Go Tools

 |

 +-- Terminal
```

---

# Provera kompletnog okruženja

Primer:

```bash
go version

go env

go test

go build
```

---

Ako sve radi:

```text
Development Environment Ready
```

---

# Prvi workflow nakon instalacije

Nakon instalacije:

```text
Install Go

    |

    v

Check Version

    |

    v

Create Module

    |

    v

Write Code

    |

    v

Run Application
```

---

# Najčešće greške prilikom instalacije

---

## 1. Komanda `go` nije pronađena

Primer:

```text
command not found: go
```

Uzrok:

Go nije dodat u PATH.

---

Rešenje:

Dodati:

```text
/usr/local/go/bin
```

u PATH.

---

## 2. Pogrešna Go verzija

Primer:

```text
Project requires Go 1.26

Installed Go 1.20
```

---

Rešenje:

Ažurirati Go instalaciju.

---

## 3. Konflikt više Go instalacija

Primer:

```text
/usr/bin/go

/usr/local/go/bin/go
```

---

Potrebno je proveriti:

```bash
which go
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go SDK sadrži sve osnovne alate;
* `go version` proverava aktivnu instalaciju;
* `go env` prikazuje konfiguraciju;
* GOROOT označava Go instalaciju;
* GOPATH je istorijski koncept koji danas nije glavni način rada;
* GOOS i GOARCH određuju ciljnu platformu;
* Go dolazi sa kompletnim razvojim alatima.

---

# Zaključak

Dobro podešeno razvojno okruženje je osnova profesionalnog Go rada.

Pre pisanja kompleksnih aplikacija, programer mora razumeti:

* gde se Go nalazi,
* kako compiler radi,
* kako alati funkcionišu,
* kako se projekti grade.

U sledećem delu ćemo detaljnije analizirati **Go workspace model, prelazak sa GOPATH na Go Modules i način na koji Go organizuje projekte**.

---

# Understanding Go Workspace

U prethodnom delu upoznali smo:

- instalaciju Go SDK-a,
- Go environment,
- `GOROOT`,
- `GOPATH`,
- `GOOS`,
- `GOARCH`,
- osnovne Go alate.

Sada ćemo detaljnije analizirati kako Go organizuje radni prostor.

Ovo je veoma važna tema zato što se način organizacije Go projekata značajno promenio kroz istoriju jezika.

---

# Šta je Go workspace?

Go workspace predstavlja okruženje u kome se nalaze:

- Go projekti,
- paketi,
- dependencies,
- build artefakti.

Možemo ga predstaviti:

```text
                 Go Workspace


                       |

        +--------------+--------------+

        |              |              |

        v              v              v


     Projects      Packages       Tools
````

---

# Istorijski razvoj Go workspace modela

Go je prošao kroz dve glavne faze:

```text
GOPATH Era

        |

        v

Go Modules Era
```

---

# 1. GOPATH era

U ranim verzijama Go-a projekti su morali biti smešteni unutar posebne strukture:

```text
GOPATH
   |
   +-- src
   |
   +-- pkg
   |
   +-- bin
```

---

# GOPATH struktura

Primer:

```text id="r8g0a2"
/home/user/go/

├── src/

│   └── github.com/

│       └── company/

│           └── project/

│

├── pkg/

└── bin/
```

---

# Značenje direktorijuma

## src/

Sadržao je source code.

Primer:

```text id="4g5e9x"
GOPATH/src/github.com/user/app
```

---

## pkg/

Sadržao je kompajlirane pakete.

Primer:

```text id="8m7q5k"
GOPATH/pkg
```

---

## bin/

Sadržao je izvršne fajlove.

Primer:

```text id="cv3x5a"
GOPATH/bin/my-tool
```

---

# Problem GOPATH pristupa

GOPATH model je imao nekoliko ograničenja.

---

# Problem 1: Lokacija projekta

Projekat je morao biti unutar:

```text id="n8l6pp"
$GOPATH/src
```

---

Primer:

Ispravno:

```text
/home/user/go/src/github.com/company/app
```

---

Neispravno:

```text
/home/user/projects/app
```

---

# Problem 2: Dependency verzije

GOPATH nije imao ugrađen sistem verzionisanja.

Problem:

```text
Application

    |

    v

Library v1.2
```

---

Ako druga aplikacija zahteva:

```text
Library v2.0
```

dolazi do konflikta.

---

# Problem 3: Reproduktivni build

Cilj modernog razvoja:

```text
Same Source Code

        +

Same Dependencies

        |

        v

Same Result
```

---

GOPATH nije mogao pouzdano garantovati ovo.

---

# Dolazak Go Modules sistema

Go Modules je uveden u:

```text
Go 1.11
```

---

Cilj:

Omogućiti projektima da budu nezavisni od GOPATH-a.

---

# Novi model

Pre:

```text id="7b3v5k"
Project

    |

    v

Must be inside GOPATH
```

---

Posle:

```text id="5wzqk3"
Project

    |

    v

Any Directory

    |

    v

go.mod
```

---

# Moderni Go workspace

Danas projekat može biti bilo gde:

Primer:

```text
/home/user/projects/payment-service
```

ili:

```text
C:\Projects\payment-service
```

---

Struktura:

```text id="7wq2gc"
payment-service/

├── go.mod

├── go.sum

├── main.go

└── internal/
```

---

# Šta definiše projekat?

Centralni element:

```text id="yg9q83"
go.mod
```

---

`go.mod` govori Go alatu:

* identitet projekta,
* Go verziju,
* dependencies.

---

Primer:

```go id="p8l2ax"
module github.com/company/payment-service

go 1.26
```

---

# Module kao jedinica organizacije

U modernom Go-u:

```text id="p4s8zv"
Module

    |

    +-- Packages

          |

          +-- Files
```

---

Primer:

```text
my-service/

├── go.mod

├── main.go

├── user/

│   └── user.go

└── database/

    └── database.go
```

---

# Package vs Module

Ovo je jedna od najčešćih početničkih konfuzija.

---

# Package

Package je grupa `.go` fajlova koji zajedno čine jednu funkcionalnu celinu.

Primer:

```go
package database
```

---

# Module

Module predstavlja ceo projekat.

Primer:

```text
github.com/company/payment-service
```

---

Poređenje:

```text
Module

    |

    +-- Package A

    |

    +-- Package B

    |

    +-- Package C
```

---

# Go Workspace danas

Moderni Go workspace može izgledati:

```text id="8c7hpx"
workspace/

├── project-a/

│   ├── go.mod

│   └── main.go

│

├── project-b/

│   ├── go.mod

│   └── main.go

│

└── tools/
```

---

Svaki projekat je nezavisan.

---

# Go Workspace Mode

Pored modula, Go ima i koncept:

```text
go.work
```

---

Koristi se kada radimo sa više modula istovremeno.

Primer:

```text
workspace/

├── go.work

├── service-a/

│   └── go.mod

└── service-b/

    └── go.mod
```

---

# Kada koristiti go.work?

Najčešće tokom razvoja:

* monorepo projekata,
* više povezanih modula,
* lokalnog razvoja.

---

# Primer problema bez go.work

Imamo:

```text
service-a

depends on

service-b
```

---

Tokom razvoja želimo lokalnu verziju:

```text
service-b (local)
```

umesto:

```text
service-b (remote version)
```

---

`go.work` rešava ovaj problem.

---

# Go build proces i workspace

Kada pokrenemo:

```bash
go build
```

Go koristi:

```text
Current Directory

        |

        v

go.mod

        |

        v

Dependencies

        |

        v

Compiler
```

---

# Moderni Go mentalni model

Zapamtiti:

```text
Directory

    |

    v

Module

    |

    v

Packages

    |

    v

Go Files
```

---

# GOPATH danas

Važno:

GOPATH nije potpuno uklonjen.

I dalje postoji za:

* cache,
* instalirane alate,
* kompatibilnost.

---

Ali:

```text
Project Organization

=

Go Modules
```

---

# Preporučena struktura rada

Za nov projekat:

```text
~/projects/

    |

    +-- application-a/

    |       |

    |       +-- go.mod

    |

    +-- application-b/

            |

            +-- go.mod
```

---

# Najčešće greške

---

## 1. Mešanje GOPATH i Modules pristupa

Problem:

```text
Project inside GOPATH

+

No go.mod
```

---

Rešenje:

Kreirati modul:

```bash
go mod init project-name
```

---

## 2. Ručno kopiranje dependencies

Loš pristup:

```text
vendor/

random libraries
```

---

Bolji pristup:

```bash
go mod tidy
```

---

## 3. Nerazumevanje modula

Greška:

```text
One project = many modules
```

u većini slučajeva.

---

Standardno:

```text
One Application

        |

        v

One Module
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go je promenio workspace model;
* GOPATH je istorijski način rada;
* Go Modules su moderni standard;
* projekat više ne mora biti u posebnoj lokaciji;
* `go.mod` definiše modul;
* package i module nisu ista stvar;
* `go.work` pomaže kod rada sa više modula.

---

# Zaključak

Razumevanje Go workspace modela je osnova za profesionalni rad.

Danas Go programer uglavnom razmišlja ovako:

```text
Create Directory

        |

        v

Initialize Module

        |

        v

Create Packages

        |

        v

Write Code

        |

        v

Build Application
```

U sledećem delu kreiraćemo prvi pravi Go projekat od početka i detaljno analizirati proces:

* `go mod init`,
* `main.go`,
* package main,
* pokretanje aplikacije,
* strukturu projekta.

---

# Creating Your First Go Project

U prethodnim delovima naučili smo:

- šta predstavlja Go projekat,
- razliku između source code-a, package-a i modula,
- kako se promenio workspace model,
- zašto su Go Modules postali standard.

Sada ćemo napraviti prvi pravi Go projekat od početka.

Cilj ovog dela je da praktično razumemo:

- kreiranje direktorijuma projekta,
- inicijalizaciju modula,
- kreiranje prvog Go fajla,
- pokretanje aplikacije,
- osnovnu strukturu projekta.

---

# Kreiranje projekta od nule

Svaki Go projekat počinje od jednog direktorijuma.

Primer:

```text id="2m4x8p"
hello-go/
````

Ovaj direktorijum predstavlja koren projekta.

---

# Korak 1: Kreiranje direktorijuma

Kreiramo novi direktorijum:

```bash id="4a8d1s"
mkdir hello-go
```

---

Ulazimo u direktorijum:

```bash id="5j9r2k"
cd hello-go
```

---

Trenutno imamo:

```text id="8wq6hm"
hello-go/
```

---

# Korak 2: Inicijalizacija Go modula

Go projekat mora biti definisan kao modul.

Koristimo:

```bash id="p7v3qx"
go mod init
```

---

Komanda zahteva ime modula:

```bash id="z4m9cx"
go mod init example.com/hello-go
```

---

Rezultat:

```text id="m2f8wb"
hello-go/

└── go.mod
```

---

# Šta se dogodilo?

Go je kreirao:

```text id="k5q2ld"
go.mod
```

koji predstavlja identitet projekta.

---

Sadržaj:

```go id="d8n4vy"
module example.com/hello-go

go 1.26
```

---

# Razumevanje `go.mod` fajla

`go.mod` ima nekoliko važnih uloga.

---

# 1. Definiše ime modula

Linija:

```go id="x9v2qa"
module example.com/hello-go
```

govori:

> Ovo je identitet ovog Go projekta.

---

Primer sa GitHub projektom:

```go id="w3p6nz"
module github.com/company/payment-service
```

---

# 2. Definiše Go verziju

Linija:

```go id="h7r5mj"
go 1.26
```

govori Go alatima koju verziju jezika projekat koristi.

---

# 3. Čuva dependency informacije

Kada projekat koristi eksterne biblioteke:

```text id="e6q9yz"
go.mod

    |

    +-- module

    +-- dependencies

    +-- versions
```

---

# Korak 3: Kreiranje prvog Go fajla

Kreiramo:

```text id="a8w2md"
main.go
```

---

Struktura sada:

```text id="m9k4qp"
hello-go/

├── go.mod

└── main.go
```

---

# Package main

Svaka izvršna Go aplikacija mora imati:

```go id="c4t8zx"
package main
```

---

Primer:

```go id="q7p3sd"
package main

func main() {

}
```

---

# Zašto postoji package main?

Go razlikuje dve vrste paketa:

```text id="v6n1mh"
Library Package

        +

Executable Package
```

---

Biblioteka:

```go id="j8x2pl"
package calculator
```

---

Aplikacija:

```go id="w5k9nr"
package main
```

---

# Funkcija main()

`main()` predstavlja ulaznu tačku programa.

Primer:

```go id="u3m8qx"
package main

import "fmt"

func main() {
	fmt.Println("Hello Go")
}
```

---

Tok izvršavanja:

```text id="n4q8sy"
Operating System

        |

        v

main()

        |

        v

Program Execution
```

---

# Korak 4: Pisanje prvog programa

Kompletan fajl:

```go id="z6c9mv"
package main

import "fmt"

func main() {
	fmt.Println("Hello, Go!")
}
```

---

Analiza:

---

## package main

```go
package main
```

Definiše izvršni paket.

---

## import

```go
import "fmt"
```

Uvozi standardni paket za formatirani izlaz.

---

## main funkcija

```go
func main()
```

Mesto odakle program počinje.

---

# Korak 5: Pokretanje aplikacije

Postoje dva glavna načina.

---

# go run

Komanda:

```bash id="r8v3mk"
go run .
```

---

Proces:

```text id="s5x1qa"
Go Files

    |

    v

Temporary Compilation

    |

    v

Execute
```

---

Rezultat:

```text id="d3m7vx"
Hello, Go!
```

---

# Šta znači tačka (`.`)?

Komanda:

```bash
go run .
```

znači:

> Pokreni Go projekat iz trenutnog direktorijuma.

---

Go pronalazi:

```text id="b2n9qx"
Current Directory

        |

        v

go.mod

        |

        v

package main

        |

        v

main.go
```

---

# go build

Drugi način:

```bash id="k8q1pv"
go build
```

---

Za razliku od `go run`, kreira izvršni fajl.

---

Pre:

```text id="g6m2tz"
hello-go/

├── go.mod

└── main.go
```

---

Posle:

```text id="r1x7km"
hello-go/

├── go.mod

├── main.go

└── hello-go
```

---

# Build proces

```text id="x4n8cs"
main.go

    |

    v

Parser

    |

    v

Compiler

    |

    v

Linker

    |

    v

Executable Binary
```

---

# Korak 6: Pokretanje buildovanog programa

Linux/macOS:

```bash id="m3v9qa"
./hello-go
```

---

Windows:

```bash id="p5z8rw"
hello-go.exe
```

---

Rezultat:

```text id="t6k2hm"
Hello, Go!
```

---

# Kompletna struktura prvog projekta

Na kraju imamo:

```text id="x7p4nv"
hello-go/

├── go.mod

├── main.go

└── hello-go
```

---

# Dodavanje README fajla

Profesionalni projekti obično imaju:

```text id="q9w3mk"
hello-go/

├── README.md

├── go.mod

└── main.go
```

---

README objašnjava:

* svrhu projekta,
* pokretanje,
* instalaciju,
* korišćenje.

---

# Dodavanje Git repozitorijuma

Sledeći korak u profesionalnom radu:

```bash id="f2k7sx"
git init
```

---

Struktura:

```text id="w8m3pz"
hello-go/

├── .git/

├── go.mod

├── main.go

└── README.md
```

---

# Praktičan workflow

Tipičan početak projekta:

```text id="c7m5qa"
mkdir project

        |

        v

cd project

        |

        v

go mod init

        |

        v

create main.go

        |

        v

go run .

        |

        v

git init
```

---

# Najčešće početničke greške

---

## 1. Zaboravljen `go mod init`

Simptom:

```text
go: cannot find main module
```

---

Rešenje:

```bash
go mod init project-name
```

---

## 2. Pogrešan package

Pogrešno:

```go
package Main
```

---

Ispravno:

```go
package main
```

---

## 3. Višak fajlova bez package organizacije

Loše:

```text id="b9x4nv"
project/

├── user.go

├── database.go

├── api.go

├── config.go

├── helper.go

└── random.go
```

---

Kasnije ćemo naučiti:

* packages,
* internal strukturu,
* organizaciju većih projekata.

---

# Prvi profesionalni mentalni model

Zapamtiti:

```text id="h4v8qp"
Project Directory

        |

        v

Go Module

        |

        v

Packages

        |

        v

Go Files

        |

        v

Functions
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* kako kreirati Go projekat;
* kako koristiti `go mod init`;
* ulogu `go.mod` fajla;
* zašto postoji `package main`;
* kako funkcioniše `main()` funkcija;
* razliku između `go run` i `go build`;
* kako izgleda minimalni Go projekat.

---

# Zaključak

Prvi Go projekat je veoma jednostavan za kreiranje.

Minimalni projekat zahteva samo:

```text id="j6n3qx"
go.mod

+

main.go
```

Ali iza te jednostavnosti postoji kompletan sistem:

* module management,
* package organizacija,
* compiler,
* runtime.

U sledećem delu detaljnije ćemo analizirati **package main, funkciju main i kako Go određuje šta predstavlja izvršni program**.

---

# Understanding `main` Package and `main` Function

U prethodnom delu kreirali smo prvi Go projekat i upoznali osnovni tok:

```text
Create Project

      |

      v

go mod init

      |

      v

Create main.go

      |

      v

go run .
````

Sada ćemo detaljnije analizirati dva najvažnija elementa svake izvršne Go aplikacije:

```go
package main

func main()
```

Ova dva elementa predstavljaju osnovu svakog Go programa koji može samostalno da se pokrene.

---

# Go program kao izvršna aplikacija

Pre nego što analiziramo `package main`, potrebno je razumeti kako Go razlikuje vrste programa.

U Go-u postoje dve osnovne kategorije:

```text
              Go Package


                  |

        +---------+---------+

        |                   |

        v                   v


   Library Package    Executable Package
```

---

# Library Package

Library package predstavlja kod koji drugi programi mogu koristiti.

Primer:

```go
package calculator

func Add(a int, b int) int {
	return a + b
}
```

---

Ovaj paket:

* nema funkciju `main()`,
* ne može samostalno da se pokrene,
* koristi se iz drugih paketa.

---

Primer korišćenja:

```go
package main

import "example.com/calculator"

func main() {
	result := calculator.Add(10, 20)
}
```

---

# Executable Package

Executable package predstavlja program koji može samostalno da se pokrene.

Takav paket mora imati:

```go
package main
```

i:

```go
func main()
```

---

Primer:

```go
package main

import "fmt"

func main() {
	fmt.Println("Application started")
}
```

---

# Šta je `package main`?

`package main` je specijalni paket u Go jeziku.

Njegova uloga:

> Označava da ovaj paket predstavlja izvršni program.

---

Primer:

```go
package main
```

---

Kada Go compiler vidi:

```go
package main
```

znači:

```text
This package produces an executable binary
```

---

# Pravilo za `package main`

Jedan executable projekat mora imati:

```text
One executable package

        |

        v

package main
```

---

Primer:

Ispravno:

```text
application/

├── go.mod

├── main.go

├── server.go

└── config.go
```

Svi fajlovi:

```go
package main
```

---

# Package mora biti isti unutar direktorijuma

U jednom direktorijumu svi `.go` fajlovi moraju pripadati istom paketu.

---

Primer:

```text
project/

├── main.go

└── server.go
```

---

main.go:

```go
package main
```

---

server.go:

```go
package main
```

---

Ovo je ispravno.

---

# Pogrešan primer

```text
project/

├── main.go

└── server.go
```

---

main.go:

```go
package main
```

---

server.go:

```go
package server
```

---

Rezultat:

```text
Error:

found packages main and server
```

---

# Funkcija `main()`

Nakon `package main` dolazi funkcija:

```go
func main()
```

---

Ona predstavlja:

```text
Program Entry Point
```

---

Kada operativni sistem pokrene Go aplikaciju:

```text
Operating System

        |

        v

Executable Binary

        |

        v

func main()

        |

        v

Program Execution
```

---

# Karakteristike `main()` funkcije

Funkcija `main()`:

* nema parametre,
* nema povratnu vrednost,
* postoji samo jednom u executable paketu.

---

Ispravno:

```go
func main() {

}
```

---

Pogrešno:

```go
func main(value string) {

}
```

---

Pogrešno:

```go
func main() int {

	return 0
}
```

---

# Zašto `main()` nema return?

U nekim jezicima:

```c
int main()
```

vraća status operativnom sistemu.

Go koristi drugačiji model.

---

Ako želimo kontrolu izlaza koristimo:

```go
os.Exit()
```

Primer:

```go
package main

import "os"

func main() {

	os.Exit(1)

}
```

---

# Tok izvršavanja Go programa

Primer:

```go
package main

import "fmt"

func main() {

	fmt.Println("Start")

}
```

---

Tok:

```text
1. OS pokreće binary

          |

          v

2. Go runtime se inicijalizuje

          |

          v

3. Package initialization

          |

          v

4. main() funkcija

          |

          v

5. Program završava
```

---

# Go Runtime pre `main()`

Važno je razumeti:

`main()` nije prva stvar koja se izvršava.

Pre nje postoji Go runtime.

---

Proces:

```text
Executable

    |

    v

Go Runtime Initialization

    |

    v

Package Variables

    |

    v

init()

    |

    v

main()
```

---

# Package Initialization

Svaki paket može imati:

```go
func init()
```

---

Primer:

```go
package main

import "fmt"

func init() {
	fmt.Println("Initialization")
}

func main() {
	fmt.Println("Main")
}
```

---

Rezultat:

```text
Initialization

Main
```

---

# Redosled izvršavanja

Kompletan redosled:

```text
1. Import Dependencies

        |

        v

2. Initialize Packages

        |

        v

3. Execute init()

        |

        v

4. Execute main()
```

---

# Više `init()` funkcija

Jedan paket može imati više:

```go
func init()
```

funkcija.

Primer:

```go
func init() {

}

func init() {

}
```

---

Izvršavaju se redom kojim se pojavljuju u fajlovima prema pravilima kompajlera.

---

# `main.go` nije specijalan naziv

Česta početnička zabuna:

> Da li Go zahteva fajl `main.go`?

Odgovor:

Ne.

---

Ovo radi:

```text
application/

├── app.go

├── server.go

└── database.go
```

---

Ako postoji:

```go
package main

func main() {

}
```

program može da se pokrene.

---

# Zašto se ipak koristi `main.go`?

Konvencija.

Najčešće:

```text
main.go
```

sadrži:

* entry point,
* inicijalizaciju aplikacije,
* povezivanje komponenti.

---

Primer:

```text
cmd/

└── api/

    └── main.go
```

---

# Package main u većim projektima

Profesionalni projekti često imaju više executable aplikacija.

Primer:

```text
company-system/

├── cmd/

│
├── api/

│   └── main.go

│
└── worker/

    └── main.go
```

---

Imamo:

```text
API Application

+

Worker Application
```

---

Svaki ima svoj:

```go
package main
```

---

# Library + Application kombinacija

Veći projekti obično izgledaju:

```text
project/

├── cmd/

│   └── app/

│       └── main.go

│
├── internal/

│   ├── service/

│   └── database/

│
└── pkg/
```

---

Arhitektura:

```text
main()

   |

   v

Application Layer

   |

   v

Business Logic

   |

   v

Infrastructure
```

---

# Najčešće početničke greške

---

## 1. Nedostaje `package main`

Primer:

```go
package app
```

---

Rezultat:

```text
cannot build executable
```

---

## 2. Nedostaje `main()`

Primer:

```go
package main

import "fmt"

func hello() {
	fmt.Println("Hello")
}
```

---

Rezultat:

```text
function main is undeclared
```

---

## 3. Više `main()` funkcija

Primer:

```text
main.go

func main()


test.go

func main()
```

---

Rezultat:

```text
main redeclared
```

---

# Mentalni model

Zapamtiti:

```text
package main

        |

        v

Executable Program

        |

        v

func main()

        |

        v

Application Entry Point
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* razliku između library i executable package-a;
* zašto postoji `package main`;
* ulogu `func main()`;
* kako Go pokreće aplikaciju;
* šta se izvršava pre `main()`;
* ulogu `init()` funkcije;
* zašto naziv `main.go` nije obavezan.

---

# Zaključak

`package main` i `func main()` predstavljaju osnovu svake Go aplikacije.

Jednostavan program:

```go
package main

func main() {

}
```

izgleda minimalno, ali iza njega stoji kompletan proces:

```text
Compiler

    |

    v

Runtime

    |

    v

Initialization

    |

    v

main()
```

U sledećem delu ćemo detaljnije analizirati **strukturu Go projekta i preporučenu organizaciju direktorijuma za profesionalne aplikacije**.

---

# Go Project Structure

U prethodnom delu detaljno smo obradili:

- `package main`,
- `func main()`,
- izvršne pakete,
- tok pokretanja Go aplikacije.

Sada prelazimo na jednu od najvažnijih tema u profesionalnom Go razvoju:

> Kako pravilno organizovati strukturu projekta?

Mali Go program može imati samo:

```text
main.go
````

ali realne aplikacije brzo postaju kompleksnije.

---

# Zašto je struktura projekta važna?

Dobra struktura projekta omogućava:

* lakše pronalaženje koda,
* jasnu podelu odgovornosti,
* jednostavnije testiranje,
* lakše održavanje,
* bolju timsku saradnju.

---

Loša organizacija:

```text
project/

├── main.go

├── user.go

├── database.go

├── api.go

├── helper.go

├── config.go

├── utils.go

└── random.go
```

---

Problem:

```text
Everything

      |

      v

One Package

      |

      v

Hard Maintenance
```

---

Dobra organizacija:

```text
project/

├── cmd/

├── internal/

├── pkg/

├── configs/

├── scripts/

├── go.mod

└── README.md
```

---

# Minimalna Go struktura

Za mali projekat često je dovoljno:

```text
hello-go/

├── go.mod

├── main.go

└── README.md
```

---

Ovo je potpuno validna Go aplikacija.

---

# Profesionalna Go struktura

Veće aplikacije često koriste:

```text
application/

├── cmd/

├── internal/

├── pkg/

├── api/

├── configs/

├── migrations/

├── scripts/

├── tests/

├── go.mod

├── go.sum

└── README.md
```

---

Ne postoji jedna obavezna struktura.

Go ne nameće framework.

Međutim, postoje široko prihvaćeni obrasci.

---

# `cmd/` direktorijum

Jedan od najčešćih elemenata profesionalnog Go projekta.

Primer:

```text
cmd/

├── api/

│   └── main.go

└── worker/

    └── main.go
```

---

Namena:

`cmd` sadrži executable aplikacije.

---

Primer:

Kompanija ima:

```text
Backend System

        |

        +-- HTTP API

        |

        +-- Background Worker

        |

        +-- CLI Tool
```

---

Struktura:

```text
cmd/

├── api/

├── worker/

└── cli/
```

---

Svaki direktorijum ima svoj:

```go
package main
```

---

# Zašto ne staviti sve u root?

Početnički projekat:

```text
project/

├── main.go

├── server.go

├── worker.go

└── cli.go
```

---

Problem:

Kada aplikacija raste:

```text
main.go

      |

      +-- API

      +-- Worker

      +-- CLI

      +-- Jobs
```

postaje teško održavati.

---

Bolje:

```text
project/

├── cmd/

│
├── api/

│
├── worker/

│
└── cli/
```

---

# `internal/` direktorijum

Jedan od najvažnijih Go koncepata.

Primer:

```text
internal/

├── user/

├── database/

└── service/
```

---

Posebno pravilo:

Kod unutar `internal` paketa može koristiti samo projekat koji je njegov roditelj.

---

Primer:

```text
company.com/project

        |

        v

internal/payment
```

---

Dozvoljeno:

```text
project

      |

      v

internal/payment
```

---

Nije dozvoljeno:

```text
external-project

      |

      v

company.com/project/internal/payment
```

---

# Zašto postoji `internal`?

Cilj:

* sakriti implementaciju,
* sprečiti nepoželjno korišćenje,
* kontrolisati API.

---

Primer:

```text
internal/

└── database/

    └── postgres.go
```

---

Drugi projekat ne može direktno koristiti:

```go
database.Connect()
```

---

# `pkg/` direktorijum

Koristi se za javno deljive pakete.

Primer:

```text
pkg/

├── logger/

├── validator/

└── client/
```

---

Ideja:

Kod koji može biti korišćen i iz drugih projekata.

---

Primer:

```text
Company

    |

    +-- service-a

    |

    +-- service-b

    |

    +-- shared logger
```

---

Logger može biti:

```text
pkg/logger
```

---

# `api/` direktorijum

Često se koristi za API definicije.

Primer:

```text
api/

├── openapi/

├── protobuf/

└── graphql/
```

---

Može sadržati:

* OpenAPI specifikacije,
* protobuf fajlove,
* API dokumentaciju.

---

# `configs/`

Sadrži konfiguracione fajlove.

Primer:

```text
configs/

├── development.yaml

├── production.yaml
```

---

Primer:

```yaml
database:
  host: localhost
  port: 5432
```

---

# `scripts/`

Sadrži pomoćne skripte.

Primer:

```text
scripts/

├── build.sh

├── migrate.sh

└── deploy.sh
```

---

Koristi se za:

* automatizaciju,
* build,
* deployment.

---

# `migrations/`

Kod aplikacija sa bazama:

```text
migrations/

├── 001_create_users.sql

├── 002_add_email.sql
```

---

Sadrži promene database schema-e.

---

# `tests/`

Dodatni test resursi.

Primer:

```text
tests/

├── integration/

└── e2e/
```

---

Napomena:

Go standardno koristi:

```text
*_test.go
```

fajlove pored koda.

---

# Primer kompletne strukture

Realna backend aplikacija:

```text
payment-service/

├── cmd/

│   └── api/

│       └── main.go

│

├── internal/

│   ├── payment/

│   ├── user/

│   └── database/

│

├── pkg/

│   └── logger/

│

├── configs/

│   └── config.yaml

│

├── migrations/

│   └── 001_init.sql

│

├── scripts/

│   └── build.sh

│

├── go.mod

├── go.sum

└── README.md
```

---

# Tok zavisnosti

Dobra arhitektura često izgleda:

```text
cmd

 |

 v

internal

 |

 v

pkg

 |

 v

External Libraries
```

---

Primer:

```text
main.go

    |

    v

service

    |

    v

repository

    |

    v

database driver
```

---

# Pravilo odgovornosti

Svaki direktorijum treba imati jasnu svrhu.

Dobro:

```text
internal/user
```

znači:

> Sve vezano za korisnike.

---

Loše:

```text
utils/
```

sa:

```text
everything.go
```

---

# Go nema "magic" folder

Važno:

Go ne zahteva:

```text
controllers/

models/

services/
```

kao određeni framework.

---

Možete organizovati projekat prema domenima.

Primer:

```text
internal/

├── orders/

├── customers/

└── payments/
```

---

# Domain-oriented organizacija

Moderni Go projekti često koriste:

```text
internal/

├── user/

│   ├── service.go

│   ├── repository.go

│   └── handler.go

├── order/

│   ├── service.go

│   └── repository.go
```

---

Prednost:

Kod je organizovan oko poslovnog domena.

---

# Najčešće greške

---

## 1. Preuranjena kompleksnost

Mali projekat:

```text
project/

├── cmd/

├── internal/

├── pkg/

├── api/

├── configs/

└── scripts/
```

---

Za:

```text
main.go
```

je nepotrebno.

---

## 2. `utils` paket za sve

Loše:

```text
utils/

├── string.go

├── database.go

├── api.go

└── random.go
```

---

Bolje:

```text
internal/

├── database/

├── api/

└── random/
```

---

## 3. Previše duboka struktura

Loše:

```text
internal/

└── services/

    └── users/

        └── implementation/

            └── handlers/
```

---

Cilj:

Jednostavnost.

---

# Mentalni model Go projekta

Zapamtiti:

```text
Executable

    |

    v

cmd

    |

    v

Application Logic

    |

    v

internal

    |

    v

Reusable Code

    |

    v

pkg
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* zašto je struktura projekta važna;
* razliku između `cmd`, `internal` i `pkg`;
* gde se smeštaju executable aplikacije;
* kako funkcioniše `internal` pravilo;
* kada koristiti `pkg`;
* kako organizovati veće Go projekte.

---

# Zaključak

Go ne nameće arhitekturu, ali profesionalni projekti koriste jasne obrasce organizacije.

Najvažnija pravila:

```text
cmd

=

Applications


internal

=

Private Business Logic


pkg

=

Reusable Public Code
```

Dobra struktura omogućava da projekat raste bez gubitka kontrole.

U sledećem delu prelazimo na **rad sa Go paketima i detaljno razumevanje package sistema**.

---

# Working With Go Packages

U prethodnom delu analizirali smo strukturu Go projekta:

- `cmd/`,
- `internal/`,
- `pkg/`,
- organizaciju većih aplikacija,
- principe podele odgovornosti.

Sada prelazimo na jedan od najvažnijih koncepata Go jezika:

> **Package sistem**

Package predstavlja osnovni mehanizam organizacije Go koda.

Bez razumevanja package-a nemoguće je pravilno razumeti:

- module,
- imports,
- visibility,
- arhitekturu aplikacija.

---

# Šta je Go package?

Package predstavlja kolekciju Go fajlova koji zajedno pripadaju jednoj funkcionalnoj celini.

Primer:

```text id="4g7s9m"
user/

├── user.go

├── validation.go

└── repository.go
````

Svi fajlovi mogu pripadati:

```go id="w4q9cy"
package user
```

---

# Package kao organizaciona jedinica

Go organizuje kod ovako:

```text id="x5s8vz"
Module

   |

   v

Packages

   |

   v

Files

   |

   v

Functions / Types
```

---

Primer:

```text id="v8c3mw"
payment-service/

├── go.mod

├── main.go

├── user/

│   ├── user.go

│   └── service.go

└── database/

    └── connection.go
```

---

Ovde imamo:

```text id="t7j4pq"
Module:

payment-service


Packages:

main

user

database
```

---

# Package deklaracija

Svaki Go fajl mora početi package deklaracijom.

Primer:

```go id="p2k8fd"
package user
```

---

Kompletan fajl:

```go id="m3v9sq"
package user

type User struct {
	Name string
}
```

---

Prva linija određuje kome fajl pripada.

---

# Pravilo: svi fajlovi u folderu imaju isti package

Primer:

```text id="8x4jcn"
calculator/

├── add.go

├── subtract.go

└── multiply.go
```

---

add.go:

```go id="5k7mzx"
package calculator
```

---

subtract.go:

```go id="0n4bvw"
package calculator
```

---

multiply.go:

```go id="1t9sqa"
package calculator
```

---

Ovo predstavlja jedan package:

```text id="r7h2mw"
calculator
```

---

# Package name vs folder name

Uobičajeno pravilo:

```text id="q4s8xm"
Folder Name

        ≈

Package Name
```

---

Primer:

Folder:

```text id="m8v4zc"
database/
```

---

Fajl:

```go id="j7d2fk"
package database
```

---

Međutim, tehnički nisu potpuno vezani.

Primer:

```text id="6n5vqa"
folder:

calculate/
```

---

Može sadržati:

```go id="x2k8wf"
package math
```

---

Iako je moguće, preporuka je:

> Package ime treba da bude jasno i očekivano.

---

# Importovanje paketa

Kada želimo koristiti drugi package, koristimo:

```go id="8f3qmv"
import
```

---

Primer:

```go id="g7x2md"
package main

import "fmt"

func main() {

	fmt.Println("Hello")

}
```

---

Ovde koristimo standardni package:

```text id="9k3lqw"
fmt
```

---

# Lokalni package import

Pretpostavimo projekat:

```text id="m2k9vp"
hello-go/

├── go.mod

├── main.go

└── calculator/

    └── calculator.go
```

---

go.mod:

```go id="y6r8ps"
module example.com/hello-go
```

---

calculator.go:

```go id="p8m3xf"
package calculator

func Add(a int, b int) int {
	return a + b
}
```

---

main.go:

```go id="x4q9zn"
package main

import (
	"fmt"

	"example.com/hello-go/calculator"
)

func main() {

	result := calculator.Add(5, 3)

	fmt.Println(result)

}
```

---

# Kako Go pronalazi package?

Go koristi:

```text id="f7v2mz"
Module Path

        +

Package Path
```

---

Primer:

Module:

```text id="8p3s9q"
example.com/hello-go
```

---

Folder:

```text id="4m7wqd"
calculator
```

---

Konačni import:

```go id="c6x8rk"
example.com/hello-go/calculator
```

---

# Exported i unexported elementi

Jedna od najvažnijih Go karakteristika.

Go nema:

* public keyword,
* private keyword.

Umesto toga koristi:

```text id="n7m3qd"
Uppercase

vs

Lowercase
```

---

# Exported elementi

Ako ime počinje velikim slovom:

```go id="v4p7ks"
type User struct {

}
```

ili:

```go id="z5q1mt"
func CreateUser() {

}
```

onda je dostupno iz drugih paketa.

---

Primer:

```go id="3y6xqp"
package user

type User struct {
	Name string
}
```

---

Drugi package:

```go id="k2n8vf"
package main

import "example.com/project/user"

func main() {

	u := user.User{}

}
```

---

# Unexported elementi

Malo slovo znači:

```go id="6z2xmw"
private to package
```

---

Primer:

```go id="r5q9kp"
package user

type userRepository struct {

}
```

---

Drugi package:

```go id="h4x7mz"
user.userRepository{}
```

---

Greška:

```text id="8c3wzp"
cannot refer to unexported name
```

---

# Zašto Go koristi ovaj pristup?

Cilj:

```text id="6q1wsm"
Simple Visibility Rules
```

---

Pravilo:

```text id="p7n4xb"
Capital Letter

        |

        v

Exported


lowercase

        |

        v

Package Private
```

---

# Package API

Svaki package ima javni API.

Primer:

```go id="2m8wqx"
package payment

func ProcessPayment() {

}
```

---

Ovo je javni API.

---

Interna implementacija:

```go id="9v5mks"
func validateCard() {

}
```

---

Korisnik package-a vidi samo:

```text id="n4x8zy"
ProcessPayment()
```

---

# Dobro dizajniran package

Dobar package ima:

* jasnu odgovornost,
* mali javni API,
* skrivenu implementaciju.

---

Primer:

```text id="8v2xkm"
database/

Public:

Connect()

Query()


Private:

openConnection()

parseConfig()
```

---

# Package dependencies

Package može koristiti druge package-e.

Primer:

```text id="q9m3fd"
handler

   |

   v

service

   |

   v

repository

   |

   v

database
```

---

Dobra arhitektura:

```text id="4m6xqn"
Higher Layer

        |

        v

Lower Layer
```

---

Loša arhitektura:

```text id="7y8kpd"
Circular Dependency

A ---> B

^     |

|_____|
```

---

# Circular dependencies

Go zabranjuje:

```text id="w5r9xm"
Package A

imports

Package B


Package B

imports

Package A
```

---

Greška:

```text id="x3q7nv"
import cycle not allowed
```

---

Zašto?

Zato što Go želi:

* jednostavan dependency graph,
* brz build,
* jasnu arhitekturu.

---

# Package naming best practices

Dobro:

```text id="3x8vqa"
user

database

http

json

payment
```

---

Loše:

```text id="6m2qpx"
helpers

common

utils

manager
```

---

Package ime treba da predstavlja:

> Šta package radi.

---

# Jednina ili množina?

Preporuka:

Koristiti jedninu.

Dobro:

```text id="q7m2kx"
user

order

payment
```

---

Manje poželjno:

```text id="r4x8ns"
users

orders

payments
```

---

# Package sa kratkim imenom

Dobri primeri iz standardne biblioteke:

```text id="w3k8mz"
fmt

http

json

os

time
```

---

Razlog:

Package ime se koristi stalno:

```go id="j8q4px"
http.Client{}

json.Marshal()
```

---

# Najčešće početničke greške

---

## 1. Preveliki package

Loše:

```text id="n6q3vz"
package everything
```

---

Bolje:

```text id="m8x5kp"
user

database

payment
```

---

## 2. Loša vidljivost

Loše:

```go id="y7m2qs"
type UserRepository struct {

}
```

ako ne treba spolja.

---

Bolje:

```go id="f4n8wx"
type userRepository struct {

}
```

---

## 3. Package kao folder za fajlove

Loše:

```text id="p6k3zm"
utils/

everything.go
```

---

Package treba da predstavlja koncept, ne skladište.

---

# Mentalni model

Zapamtiti:

```text id="z9w4qp"
Package

    |

    +-- Types

    |

    +-- Functions

    |

    +-- Variables


Exported

    |

    v

Public API


Unexported

    |

    v

Internal Implementation
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je Go package;
* kako se organizuju `.go` fajlovi;
* kako funkcioniše `import`;
* razliku između module-a i package-a;
* kako Go određuje vidljivost;
* šta su exported i unexported elementi;
* zašto Go zabranjuje circular dependencies.

---

# Zaključak

Package sistem je osnova Go arhitekture.

Profesionalni Go kod nije organizovan kroz:

```text
many files
```

već kroz:

```text
clear packages

        |

        v

clear responsibilities
```

Dobro dizajnirani package-i omogućavaju da aplikacija raste bez povećanja kompleksnosti.

U sledećem delu ćemo detaljnije obraditi **upravljanje dependency-jima kroz Go Modules (`go.mod`, `go.sum`, `go get`, `go mod tidy`)**.

---

# Managing Dependencies With Go Modules

U prethodnom delu naučili smo:

- šta su Go paketi,
- kako funkcioniše `package` sistem,
- kako se koriste import-i,
- razliku između exported i unexported elemenata,
- kako Go organizuje zavisnosti između paketa.

Sada prelazimo na jednu od najvažnijih tema modernog Go razvoja:

> **Dependency Management pomoću Go Modules sistema**

---

# Šta su dependencies?

Dependency (zavisnost) predstavlja eksterni kod koji naš projekat koristi.

Primer:

Naša aplikacija:

```text
 id="3x7mqa"
payment-service

        |

        v

External Libraries
````

---

Primeri dependencies:

* HTTP framework,
* database driver,
* logging biblioteka,
* validation paket,
* testing biblioteka.

---

Bez dependency management sistema programer bi morao ručno da:

* preuzima biblioteke,
* čuva njihove verzije,
* rešava konflikte,
* prati promene.

---

# Problem ručnog upravljanja bibliotekama

Zamislimo projekat:

```text
 id="n7q2mz"
Application

    |

    +-- Library A

    |

    +-- Library B

    |

    +-- Library C
```

---

Library A koristi:

```text
 id="k5w8pv"
Database Driver v1.0
```

---

Library B zahteva:

```text
 id="m2q9xz"
Database Driver v2.0
```

---

Dobijamo problem:

```text
 id="r8v4hs"
Dependency Conflict
```

---

Go Modules rešava ovaj problem.

---

# Šta su Go Modules?

Go Modules predstavljaju standardni sistem za:

* definisanje projekta,
* upravljanje dependencies,
* verzionisanje,
* reproducibilan build.

---

Glavni fajlovi:

```text
 id="p6w3nk"
go.mod

go.sum
```

---

Struktura:

```text
 id="h9x4mq"
Project

├── go.mod

├── go.sum

└── *.go files
```

---

# go.mod fajl

`go.mod` je centralni fajl svakog modernog Go projekta.

Sadrži:

* module name,
* Go verziju,
* dependencies.

---

Primer:

```go
module github.com/company/payment-service

go 1.26
```

---

# Module declaration

Prva linija:

```go
module github.com/company/payment-service
```

---

Predstavlja:

```text
 id="q3m7vz"
Unique Project Identity
```

---

Kada importujemo sopstvene pakete:

```go
import "github.com/company/payment-service/user"
```

Go koristi module path.

---

# Go version directive

Druga linija:

```go
go 1.26
```

---

Definiše:

* verziju Go jezika,
* minimalnu verziju potrebnu za projekat.

---

Primer:

```go
module example.com/app

go 1.26
```

znači:

```text
 id="w8p2sx"
This project targets Go 1.26
```

---

# go.sum fajl

Pored `go.mod` postoji:

```text
go.sum
```

---

Njegova uloga:

Čuva cryptographic checksums dependency-ja.

---

Primer:

```text
 id="z5k9mw"
go.mod

    |

    v

Which dependencies?


go.sum

    |

    v

Are they exactly the same?
```

---

# Zašto postoji checksum?

Zbog sigurnosti i reproduktivnosti.

Cilj:

```text
 id="a7q3vx"
Developer A

+

Developer B

+

CI/CD

=

Same Dependencies
```

---

# Dodavanje dependency-ja

Najčešći način:

```bash
go get
```

---

Primer:

```bash
go get github.com/example/logger
```

---

Go će:

1. pronaći paket,
2. preuzeti verziju,
3. dodati ga u `go.mod`,
4. ažurirati `go.sum`.

---

Pre:

```go
module example.com/app

go 1.26
```

---

Posle:

```go
module example.com/app

go 1.26

require github.com/example/logger v1.2.0
```

---

# require sekcija

Dependencies se nalaze u:

```go
require
```

sekciji.

---

Primer:

```go
require (
	github.com/example/logger v1.2.0
	github.com/example/database v2.1.0
)
```

---

Značenje:

```text
 id="d7x2mq"
Package Name

+

Version
```

---

# Direktne i indirektne dependencies

Go razlikuje:

## Direct dependencies

Biblioteke koje direktno koristimo.

Primer:

```go
import "github.com/example/logger"
```

---

## Indirect dependencies

Biblioteke koje koriste naše dependencies.

Primer:

```text
 id="f5w8ny"
Our App

    |

    v

Library A

    |

    v

Library B
```

---

U `go.mod` mogu izgledati:

```go
require (
	github.com/example/logger v1.0.0
	github.com/example/helper v1.3.0 // indirect
)
```

---

# go mod tidy

Jedna od najvažnijih komandi:

```bash
go mod tidy
```

---

Njena uloga:

* dodaje nedostajuće dependencies,
* uklanja nepotrebne dependencies,
* ažurira `go.sum`.

---

Primer:

Pre:

```text
 id="m9q4zk"
go.mod

require:

unused-library
```

---

Posle:

```bash
go mod tidy
```

---

Rezultat:

```text
 id="p8v3sx"
Unused dependency removed
```

---

# go mod download

Komanda:

```bash
go mod download
```

---

Preuzima dependencies bez buildovanja projekta.

---

Koristi se često u CI/CD:

```text
 id="k4m8pw"
CI Pipeline

    |

    v

Download Dependencies

    |

    v

Build

    |

    v

Test
```

---

# go mod verify

Provera integriteta dependencies:

```bash
go mod verify
```

---

Proverava:

* da li su fajlovi promenjeni,
* da li checksum odgovara.

---

# go list -m

Prikaz modula:

```bash
go list -m all
```

---

Primer izlaza:

```text
 id="w7p2mz"
example.com/app

github.com/example/logger v1.2.0

github.com/example/db v3.1.0
```

---

# go get u različitim scenarijima

---

## Dodavanje nove biblioteke

```bash
go get github.com/example/package
```

---

## Specifična verzija

```bash
go get github.com/example/package@v1.5.0
```

---

## Najnovija verzija

```bash
go get github.com/example/package@latest
```

---

# Uklanjanje dependency-ja

Najčešće:

1. uklonimo import iz koda,
2. pokrenemo:

```bash
go mod tidy
```

---

Go automatski uklanja nepotrebne dependencies.

---

# Dependency graph

Go interno posmatra dependencies kao graf.

Primer:

```text
 id="t4m8qx"
             Application

                  |

                  v

              Library A

              /      \

             v        v

       Library B   Library C
```

---

Go mora znati:

* koje verzije postoje,
* koje biblioteke su potrebne,
* da li postoje konflikti.

---

# Minimal Version Selection (MVS)

Go koristi poseban algoritam:

```text
 id="n3x8mq"
Minimal Version Selection
```

---

Ideja:

Ako više dependencies zahtevaju različite verzije:

```text
 id="v6p2qa"
Library A requires:

v1.5


Library B requires:

v1.8
```

Go bira:

```text
 id="m5q9xz"
v1.8
```

---

Zašto?

Zato što je to najnovija minimalna kompatibilna verzija u grafu.

---

# Vendor mode

Go podržava i:

```text
vendor/
```

direktorijum.

---

Primer:

```text
 id="z9k3wp"
project/

├── vendor/

├── go.mod

└── main.go
```

---

Dependencies se kopiraju lokalno.

---

Koristi se kada:

* build mora biti potpuno izolovan,
* enterprise sistemi zahtevaju kontrolu.

---

Komanda:

```bash
go mod vendor
```

---

# Profesionalni dependency workflow

Tipičan tok:

```text
Add Dependency

        |

        v

go get

        |

        v

Write Code

        |

        v

go mod tidy

        |

        v

go test

        |

        v

Commit go.mod + go.sum
```

---

# Šta se commit-uje u Git?

Obavezno:

```text
 id="q8m4vx"
go.mod

go.sum
```

---

Ne commit-uje se:

```text
 id="r3p7mz"
Downloaded Cache

Temporary Files
```

---

# Najčešće greške

---

## 1. Ne koristiti `go mod tidy`

Problem:

```text
 id="p4x8mq"
Unused dependencies remain
```

---

Rešenje:

```bash
go mod tidy
```

---

## 2. Ručno menjanje verzija bez razumevanja

Loše:

```go
require package v10.0.0
```

bez provere kompatibilnosti.

---

## 3. Ne commit-ovati go.sum

Problem:

```text
 id="w5n9qx"
Different machine

        |

        v

Different dependency state
```

---

# Mentalni model

Zapamtiti:

```text
 id="y7m2pz"
go.mod

    |

    v

What do we need?


go.sum

    |

    v

Can we trust it?
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su dependencies;
* zašto postoji Go Modules sistem;
* ulogu `go.mod`;
* ulogu `go.sum`;
* kako se dodaju dependencies;
* kako funkcioniše `go get`;
* kako se čisti projekat pomoću `go mod tidy`;
* kako Go rešava dependency verzije.

---

# Zaključak

Go Modules predstavljaju osnovu modernog Go razvoja.

Danas se svaki profesionalni Go projekat oslanja na:

```text
 id="b8x4mq"
go.mod

+

go.sum

+

go command tooling
```

Razumevanje dependency management-a omogućava da projekti budu:

* stabilni,
* ponovljivi,
* sigurni,
* laki za održavanje.

U sledećem delu prelazimo na **semantičko verzionisanje (Semantic Versioning) i razumevanje verzija Go modula**.

---

# Semantic Versioning and Module Versions

U prethodnom delu upoznali smo Go Modules sistem:

- `go.mod`,
- `go.sum`,
- dependency management,
- `go get`,
- `go mod tidy`.

Sada prelazimo na temu koja je direktno povezana sa dependency-jima:

> **Kako Go određuje i koristi verzije modula?**

Da bismo pravilno koristili biblioteke u Go projektima, moramo razumeti:

- verzionisanje softvera,
- Semantic Versioning,
- module versioning,
- kompatibilnost između verzija.

---

# Zašto su verzije važne?

Softver se konstantno menja.

Primer:

Prva verzija biblioteke:

```text
logger v1.0.0
````

Kasnije:

```text
logger v1.1.0
```

Još kasnije:

```text
logger v2.0.0
```

---

Svaka promena može doneti:

* nove funkcionalnosti,
* ispravke grešaka,
* promene API-ja.

---

Problem:

Ako aplikacija automatski preuzme najnoviju verziju:

```text
Application

    |

    v

Library v2.0.0
```

može doći do:

```text
Build Failure
```

ili:

```text
Runtime Bugs
```

---

Zbog toga Go koristi eksplicitno verzionisanje.

---

# Semantic Versioning (SemVer)

Go ekosistem koristi standard:

```text
Semantic Versioning
```

poznat kao:

```text
SemVer
```

---

Format:

```text
MAJOR.MINOR.PATCH
```

Primer:

```text
1.4.7
```

---

Značenje:

```text
1     .     4     .     7

|           |           |

Major       Minor       Patch
```

---

# PATCH verzija

Patch predstavlja:

* bug fix,
* sigurnosne ispravke,
* male interne promene.

---

Primer:

```text
1.4.7

        |

        v

1.4.8
```

---

Očekivanje:

API ostaje potpuno kompatibilan.

---

Primer:

Pre:

```go
func Validate(input string) bool
```

Posle:

```go
func Validate(input string) bool
```

---

Samo je implementacija popravljena.

---

# MINOR verzija

Minor predstavlja:

* nove funkcionalnosti,
* nove API mogućnosti,
* backward compatible promene.

---

Primer:

```text
1.4.0

        |

        v

1.5.0
```

---

Stari kod treba i dalje da radi.

---

Primer:

Pre:

```go
func CreateUser(name string)
```

Posle:

```go
func CreateUser(name string)

func CreateUserWithEmail(name string, email string)
```

---

Dodata je nova funkcionalnost.

---

# MAJOR verzija

Major predstavlja:

* breaking changes,
* promenu javnog API-ja,
* nekompatibilne promene.

---

Primer:

```text
1.5.0

        |

        v

2.0.0
```

---

Primer breaking promene:

Pre:

```go
func GetUser(id int) User
```

Posle:

```go
func GetUser(id string) (*User, error)
```

---

Stari kod:

```go
user := GetUser(10)
```

više ne radi.

---

# Pravilo kompatibilnosti

SemVer pravilo:

```text
PATCH

=
Bug Fix


MINOR

=
New Features


MAJOR

=
Breaking Changes
```

---

Vizuelno:

```text
1.2.3

|

|

+---- Patch

|

+--------- Minor

|

+--------------- Major
```

---

# Verzije u Go modulima

Go modul verzija izgleda:

```text
github.com/company/library v1.3.0
```

---

Primer:

```go
require (
	github.com/company/logger v1.5.2
)
```

---

To znači:

Koristi:

```text
logger

version:

1.5.2
```

---

# Module Path i verzija

Za module:

```text
v0
v1
```

verzija se ne nalazi u module path-u.

Primer:

```go
module github.com/company/logger
```

---

Dependency:

```go
require github.com/company/logger v1.4.0
```

---

Ali za:

```text
v2+
```

postoji posebno pravilo.

---

# Major Version u Module Path-u

Ako modul ima:

```text
v2
```

ili veću verziju:

ona postaje deo imena modula.

---

Primer:

v1:

```text
github.com/company/logger
```

---

v2:

```text
github.com/company/logger/v2
```

---

go.mod:

```go
module github.com/company/logger/v2
```

---

Import:

```go
import "github.com/company/logger/v2"
```

---

# Zašto Go zahteva ovo?

Zato što omogućava:

```text
Same Project

        |

        +-- Library v1

        |

        +-- Library v2
```

---

Primer:

```text
Application

    |

    +-- package A

          uses logger/v1


    +-- package B

          uses logger/v2
```

---

Obe verzije mogu postojati istovremeno.

---

# Primer migracije sa v1 na v2

Početna verzija:

```go
module github.com/example/math
```

---

Nova verzija:

```go
module github.com/example/math/v2
```

---

Korisnik:

Pre:

```go
import "github.com/example/math"
```

---

Posle:

```go
import "github.com/example/math/v2"
```

---

Promena je eksplicitna.

---

# Pre-release verzije

SemVer podržava i pre-release oznake.

Primer:

```text
1.0.0-alpha
```

---

Još primera:

```text
1.0.0-beta
```

```text
1.0.0-rc1
```

---

Značenje:

```text
alpha

=
rana faza


beta

=
testiranje


rc

=
release candidate
```

---

# Verzije `v0.x.x`

Poseban slučaj:

```text
v0.1.0
```

---

Znači:

> API još nije stabilan.

---

Biblioteke sa:

```text
v0.x.x
```

mogu napraviti breaking changes bez prelaska na v1.

---

Primer:

```text
v0.5.0

        |

        v

v0.6.0
```

---

API može biti promenjen.

---

# Go proxy i verzije

Go koristi module proxy sistem.

Kada izvršimo:

```bash
go get github.com/example/library
```

Go traži modul kroz:

```text
Module Proxy

        |

        v

Versioned Source
```

---

Prednosti:

* brže preuzimanje,
* keširanje,
* reproduktivni build.

---

# go list za verzije

Prikaz verzije modula:

```bash
go list -m all
```

---

Primer:

```text
example.com/app

github.com/example/logger v1.4.2

github.com/example/db v2.1.0
```

---

# Promena verzije dependency-ja

Primer:

Trenutno:

```go
require github.com/example/logger v1.2.0
```

---

Želimo:

```text
v1.5.0
```

---

Komanda:

```bash
go get github.com/example/logger@v1.5.0
```

---

Zatim:

```bash
go mod tidy
```

---

# Upgrade svih dependency-ja

Komanda:

```bash
go get -u ./...
```

---

Značenje:

```text
Update dependencies

inside current module
```

---

Pa zatim:

```bash
go mod tidy
```

---

# Dependency update strategija

U profesionalnim projektima nije preporučljivo:

```text
Always use latest
```

---

Bolji pristup:

```text
Review

    |

    v

Test

    |

    v

Upgrade

    |

    v

Deploy
```

---

# Stabilan projekat

Dobar projekat:

```text
go.mod

    |

    v

Explicit Versions

    |

    v

Reproducible Build
```

---

# Najčešće greške

---

## 1. Ignorisanje major verzija

Pogrešno:

```go
github.com/example/library
```

kada je potrebno:

```go
github.com/example/library/v2
```

---

## 2. Automatski upgrade bez testiranja

Problem:

```text
Dependency Update

        |

        v

Production Failure
```

---

## 3. Korišćenje nestabilnih biblioteka bez procene

Primer:

```text
v0.1.0
```

zahteva dodatni oprez.

---

# Mentalni model

Zapamtiti:

```text
Version Number

1.4.7

|

+-- Major
|
+---- Minor
|
+-------- Patch
```

---

Za Go module:

```text
v1

=
same module path


v2+

=
version in module path
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je Semantic Versioning;
* razliku između Major, Minor i Patch verzija;
* šta predstavljaju breaking changes;
* kako Go tretira v2+ module;
* zašto verzija postaje deo module path-a;
* kako bezbedno ažurirati dependency-je.

---

# Zaključak

Versioning je osnova stabilnog Go ekosistema.

Go ne pokušava da sakrije promene.

Naprotiv:

```text
Breaking Change

        |

        v

Explicit Version
```

Programer uvek zna koju verziju koristi i kada prelazi na novu major verziju.

U sledećem delu prelazimo na **kreiranje sopstvenog Go modula, rad sa više paketa i povezivanje više delova aplikacije u jednu celinu**.

---

# Creating a Multi-Package Go Project

U prethodnim delovima obradili smo:

- kreiranje prvog Go projekta;
- ulogu `go.mod` fajla;
- `package main`;
- organizaciju direktorijuma;
- Go package sistem;
- dependency management;
- verzionisanje modula.

Do sada smo uglavnom posmatrali jednostavan projekat:

```text
hello-go/

├── go.mod

└── main.go
````

Međutim, realne aplikacije se ne sastoje od samo jednog fajla.

Profesionalni Go projekti su organizovani kroz više paketa.

---

# Zašto koristiti više paketa?

Jedan fajl može funkcionisati za male aplikacije:

```go
package main

func main() {

}
```

Ali kako aplikacija raste:

```text
main.go

user.go

database.go

payment.go

email.go

config.go
```

dobijamo problem:

* fajlovi postaju veliki;
* odgovornosti se mešaju;
* kod je teže testirati;
* promene postaju rizične.

---

Rešenje:

Podela aplikacije na pakete.

```text
Application

    |

    +-- User Package

    |

    +-- Database Package

    |

    +-- Payment Package
```

---

# Primer projekta

Napravimo jednostavnu aplikaciju:

```text
user-service
```

Aplikacija će imati:

* glavni program;
* user logiku;
* database paket.

---

Konačna struktura:

```text
user-service/

├── go.mod

├── main.go

├── user/

│   └── user.go

└── database/

    └── database.go
```

---

# Kreiranje projekta

Kreiramo direktorijum:

```bash
mkdir user-service
```

---

Ulazimo:

```bash
cd user-service
```

---

Inicijalizujemo modul:

```bash
go mod init example.com/user-service
```

---

Rezultat:

```text
user-service/

└── go.mod
```

---

# Kreiranje user paketa

Kreiramo direktorijum:

```bash
mkdir user
```

---

Dodajemo fajl:

```text
user/

└── user.go
```

---

Sadržaj:

```go
package user

type User struct {
	Name string
	Age  int
}

func New(name string, age int) User {
	return User{
		Name: name,
		Age:  age,
	}
}
```

---

Ovde imamo:

```text
Package:

user


Exported:

User

New()
```

---

# Kreiranje database paketa

Struktura:

```text
database/

└── database.go
```

---

Sadržaj:

```go
package database

import "fmt"

func Connect() {

	fmt.Println("Database connected")

}
```

---

Ovaj paket predstavlja database sloj.

---

# Korišćenje paketa iz main programa

Sada povezujemo sve.

`main.go`

```go
package main

import (
	"fmt"

	"example.com/user-service/database"
	"example.com/user-service/user"
)

func main() {

	database.Connect()

	u := user.New("Marko", 30)

	fmt.Println(u)

}
```

---

Rezultat:

```text
Database connected

{Marko 30}
```

---

# Kako Go pronalazi lokalne pakete?

Go koristi:

```text
module path

+

folder path
```

---

Naš `go.mod`:

```go
module example.com/user-service
```

---

Folder:

```text
user/
```

---

Import:

```go
import "example.com/user-service/user"
```

---

Formula:

```text
Module Name

        +

Directory

        =

Import Path
```

---

# Package Dependency Graph

Naša aplikacija sada izgleda:

```text
main

 |

 +----------------+

 |                |

 v                v

user          database
```

---

`main` zavisi od:

```text
user

database
```

---

Ali:

```text
user

X

database
```

nemaju međusobnu zavisnost.

---

# Pravilo zavisnosti

Dobro:

```text
main

 |

 v

service

 |

 v

repository
```

---

Loše:

```text
A ---> B

^

|

C
```

---

Cilj:

Jednosmeran tok zavisnosti.

---

# Package API dizajn

Kada pravimo paket, treba odlučiti:

Šta treba biti dostupno spolja?

---

Primer:

```go
package user

type User struct {
	Name string
}

func New(name string) User {
	return User{
		Name: name,
	}
}

func validateName(name string) bool {
	return name != ""
}
```

---

Javni API:

```text
User

New()
```

---

Interna implementacija:

```text
validateName()
```

---

Drugi paket ne mora znati:

```text
kako

New()

radi
```

---

# Organizacija poslovne logike

Primer realnije aplikacije:

```text
application/

├── cmd/

│   └── api/

│       └── main.go

│

└── internal/

    ├── user/

    │   ├── service.go

    │   └── repository.go

    │

    └── database/

        └── connection.go
```

---

Tok:

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

# Kada napraviti novi paket?

Dobar kriterijum:

Kreiraj novi paket kada postoji:

* jasna odgovornost;
* logička celina;
* mogućnost ponovne upotrebe.

---

Primer:

Dobro:

```text
payment/

email/

database/
```

---

Loše:

```text
helpers/

misc/

common/
```

---

# Paket nije isto što i folder

Važno:

Folder je fizička organizacija.

Package je Go koncept.

---

Primer:

```text
internal/

└── user/
```

predstavlja:

```go
package user
```

---

Ali folder sam po sebi ne određuje arhitekturu.

---

# Više executable aplikacija

Jedan modul može imati više programa.

Primer:

```text
company-system/

├── go.mod

├── cmd/

│
├── api/

│   └── main.go

│
└── worker/

    └── main.go
```

---

Imamo:

```text
API Application

+

Background Worker
```

---

Oba koriste isti modul:

```go
module company.com/system
```

---

# Deljenje koda između aplikacija

Primer:

```text
cmd/

├── api/

│   └── main.go

└── worker/

    └── main.go


internal/

└── user/
```

---

Tok:

```text
api

 |

 +----> user


worker

 |

 +----> user
```

---

# Prednost modularne organizacije

Dobijamo:

## Lakše testiranje

Možemo testirati:

```text
user package
```

nezavisno od:

```text
main package
```

---

## Lakšu izmenu

Promena:

```text
database package
```

ne mora uticati na:

```text
user package
```

---

## Bolju čitljivost

Programer odmah vidi:

```text
gde pripada određeni kod
```

---

# Najčešće greške

---

## 1. Sve u main paketu

Loše:

```text
main.go

user.go

database.go

payment.go
```

sve:

```go
package main
```

---

Problem:

```text
Application Logic

=

Entry Point
```

---

## 2. Previše malih paketa

Loše:

```text
name/

age/

address/

phone/
```

---

Rezultat:

Previše kompleksnosti.

---

## 3. Ciklične zavisnosti

Primer:

```text
user

 imports

database


database

 imports

user
```

---

Go odbija:

```text
import cycle not allowed
```

---

# Mentalni model

Zapamtiti:

```text
Module

    |

    v

Packages

    |

    v

Files

    |

    v

Functions
```

---

Aplikacija:

```text
main

 |

 +-- user

 |

 +-- database

 |

 +-- payment
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* kako napraviti projekat sa više paketa;
* kako importovati sopstvene pakete;
* kako Go rešava lokalne module;
* kako dizajnirati package API;
* kako organizovati zavisnosti između paketa;
* kada kreirati novi paket.

---

# Zaključak

Go projekti rastu tako što se ne dodaje samo više fajlova, već se kod organizuje u jasne pakete.

Dobar Go projekat ima:

```text
small packages

+

clear responsibilities

+

simple dependencies
```

Ovakav pristup omogućava da aplikacije ostanu održive čak i kada postanu veoma velike.

U sledećem delu prelazimo na **rad sa komandama Go alata (`go run`, `go build`, `go install`, `go test`) i razumevanje kompletnog razvojnog workflow-a**.

---

# Working With Go Tooling

U prethodnom delu kreirali smo projekat sa više paketa i naučili:

- kako moduli sadrže više paketa;
- kako se paketi međusobno povezuju;
- kako se dizajnira package API;
- kako izgleda dependency graf aplikacije.

Sada prelazimo na praktičan deo svakodnevnog Go razvoja:

> **Go Command Line Tooling**

Go dolazi sa kompletnim skupom ugrađenih alata koji omogućavaju:

- pokretanje programa,
- kompajliranje,
- instalaciju aplikacija,
- testiranje,
- formatiranje koda,
- upravljanje modulima.

Glavni alat je:

```bash
go
````

---

# Go Command

Komanda:

```bash
go
```

prikazuje sve dostupne podkomande.

Primer:

```bash
go help
```

---

Najčešće korišćene komande:

```text
go run

go build

go install

go test

go fmt

go vet

go mod

go get
```

---

# Razvojni workflow u Go-u

Tipičan tok rada:

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

go build

      |

      v

Deploy
```

---

# `go run`

Jedna od prvih komandi koju svaki Go programer koristi.

Sintaksa:

```bash
go run <files>
```

ili:

```bash
go run .
```

---

Primer projekta:

```text
hello-go/

├── go.mod

└── main.go
```

---

Pokretanje:

```bash
go run .
```

---

Go izvršava sledeće:

```text
Source Code

      |

      v

Compile

      |

      v

Create Temporary Binary

      |

      v

Execute
```

---

Važno:

`go run` ne čuva izvršni fajl.

---

Posle:

```bash
go run .
```

struktura ostaje:

```text
hello-go/

├── go.mod

└── main.go
```

---

# Kada koristiti `go run`?

Idealno za:

* razvoj;
* brzo testiranje;
* lokalne eksperimente.

---

Primer:

Menjamo:

```go
fmt.Println("Hello")
```

pokrenemo:

```bash
go run .
```

vidimo rezultat.

---

# `go build`

`go build` kompajlira projekat.

Sintaksa:

```bash
go build
```

---

Primer:

Pre:

```text
hello-go/

├── go.mod

└── main.go
```

---

Pokrenemo:

```bash
go build
```

---

Rezultat:

```text
hello-go/

├── go.mod

├── main.go

└── hello-go
```

---

Dobijeni fajl:

```text
hello-go
```

je izvršni program.

---

# Build proces

Interno:

```text
.go Files

      |

      v

Parser

      |

      v

Compiler

      |

      v

Linker

      |

      v

Executable Binary
```

---

# Pokretanje buildovanog programa

Linux/macOS:

```bash
./hello-go
```

---

Windows:

```bash
hello-go.exe
```

---

Rezultat:

```text
Hello Go!
```

---

# `go build` sa imenovanjem izlaza

Možemo definisati ime:

```bash
go build -o app
```

---

Rezultat:

```text
app
```

---

Primer:

```bash
go build -o server
```

---

Dobijamo:

```text
server
```

---

# `go install`

`go install` služi za instalaciju Go aplikacija.

Razlika:

```text
go build

=

kreira binary lokalno


go install

=

instalira binary u Go bin direktorijum
```

---

Primer:

```bash
go install ./cmd/api
```

---

Rezultat:

Binary se kopira u:

```text
$GOBIN
```

ili:

```text
$GOPATH/bin
```

---

# Kada koristiti `go install`?

Najčešće za:

* CLI alate;
* developer utilities;
* interne alate.

---

Primer:

Instalacija alata:

```bash
go install github.com/tool/example@latest
```

---

Posle:

```bash
example
```

može direktno da se pokrene.

---

# `go test`

Go ima ugrađenu podršku za testiranje.

Komanda:

```bash
go test
```

---

Primer:

Struktura:

```text
calculator/

├── calculator.go

└── calculator_test.go
```

---

Pokretanje:

```bash
go test ./...
```

---

Rezultat:

```text
PASS
ok
```

---

# `go test ./...`

Veoma važna komanda.

Značenje:

```text
./...

=

Current Package

+

All Subpackages
```

---

Primer:

```text
project/

├── user/

├── database/

└── api/
```

---

Komanda:

```bash
go test ./...
```

testira:

```text
user

database

api
```

---

# `go fmt`

Go ima standardni formatter.

Komanda:

```bash
go fmt
```

---

Pre:

```go
func main(){
fmt.Println("Hello")
}
```

---

Posle:

```go
func main() {
	fmt.Println("Hello")
}
```

---

Go filozofija:

> Formatiranje ne treba da bude stvar ličnog stila.

---

# Zašto je `gofmt` važan?

Svi Go projekti koriste isti format.

Prednosti:

* nema rasprave oko stilova;
* lakši code review;
* konzistentan kod.

---

Često:

```bash
go fmt ./...
```

---

# `go vet`

`go vet` analizira kod i pronalazi moguće probleme.

Komanda:

```bash
go vet ./...
```

---

Primer problema:

```go
fmt.Printf("%d", "hello")
```

---

Tipovi se ne poklapaju.

---

`go vet` može upozoriti:

```text
possible formatting directive mismatch
```

---

# `go clean`

Čisti build fajlove.

Komanda:

```bash
go clean
```

---

Uklanja:

* privremene fajlove;
* build cache;
* test cache.

---

# `go env`

Prikazuje Go environment.

Komanda:

```bash
go env
```

---

Primer:

```text
GOROOT=/usr/local/go

GOPATH=/home/user/go

GOOS=linux

GOARCH=amd64
```

---

# Važne environment promenljive

---

## GOROOT

Lokacija Go instalacije.

Primer:

```text
/usr/local/go
```

---

## GOPATH

Workspace za Go alate.

Primer:

```text
/home/user/go
```

---

## GOBIN

Lokacija instaliranih binary fajlova.

Primer:

```text
/home/user/go/bin
```

---

# `go doc`

Prikazuje dokumentaciju.

Primer:

```bash
go doc fmt
```

---

Rezultat:

```text
Package fmt implements formatted I/O
```

---

Za funkciju:

```bash
go doc fmt.Println
```

---

Dobijamo:

```go
func Println(a ...any) (n int, err error)
```

---

# `go list`

Prikazuje informacije o paketima.

Primer:

```bash
go list
```

---

Svi paketi:

```bash
go list ./...
```

---

Primer:

```text
example.com/app

example.com/app/user

example.com/app/database
```

---

# `go version`

Prikazuje Go verziju:

```bash
go version
```

---

Primer:

```text
go version go1.26 linux/amd64
```

---

# Tipičan dnevni workflow

Primer:

```text
1. Write Code

        |

2. go fmt ./...

        |

3. go test ./...

        |

4. go vet ./...

        |

5. go build

        |

6. Commit
```

---

# CI/CD workflow

U profesionalnim projektima:

```text
Developer Push

        |

        v

CI Pipeline

        |

        +-- go fmt check

        |

        +-- go test

        |

        +-- go vet

        |

        +-- go build
```

---

# Najčešće greške

---

## 1. Pokretanje pogrešnog direktorijuma

Problem:

```bash
go run .
```

bez:

```text
go.mod
```

---

Rešenje:

Pokrenuti iz root direktorijuma modula.

---

## 2. Neformatiran kod

Problem:

```text
Different formatting styles
```

---

Rešenje:

```bash
go fmt ./...
```

---

## 3. Testiranje samo jednog paketa

Loše:

```bash
go test
```

---

Bolje:

```bash
go test ./...
```

---

# Mentalni model

Zapamtiti:

```text
go run

=

Develop


go build

=

Compile


go install

=

Install


go test

=

Verify


go fmt

=

Format


go vet

=

Analyze
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* kako koristiti osnovne Go komande;
* razliku između `go run`, `go build` i `go install`;
* kako testirati ceo projekat;
* kako formatirati kod;
* kako proveriti kvalitet koda;
* kako izgleda profesionalni Go workflow.

---

# Zaključak

Go tooling je jedan od razloga zbog kojih je Go popularan.

Umesto velikog broja eksternih alata, Go dolazi sa kompletnim standardnim setom:

```text
go command

+

standard workflow

=

productive development
```

U sledećem delu prelazimo na **kreiranje profesionalnog projekta sa README fajlom, Git integracijom i osnovnim pravilima organizacije repozitorijuma**.

---

# Professional Go Project Structure

U prethodnim delovima naučili smo:

- kako kreirati Go module;
- kako organizovati više paketa;
- kako koristiti Go tooling;
- kako upravljati dependency-jima.

Sada prelazimo na temu koja je veoma važna za rad u realnim projektima:

> **Kako izgleda profesionalna struktura Go projekta?**

Mali Go program može izgledati ovako:

```text
hello-world/

├── go.mod

└── main.go
````

Ali produkcione aplikacije često imaju:

* više executable aplikacija;
* poslovnu logiku;
* database sloj;
* API sloj;
* konfiguraciju;
* testove;
* dokumentaciju.

Zbog toga je potrebna jasna organizacija.

---

# Cilj dobre strukture projekta

Dobra struktura treba da omogući:

* jednostavno pronalaženje koda;
* jasnu podelu odgovornosti;
* lakše testiranje;
* lakše održavanje;
* skaliranje aplikacije.

---

Loša organizacija:

```text
 id="6r8vqa"
project/

├── everything.go

├── helpers.go

├── database.go

└── main.go
```

---

Problem:

Sve odgovornosti su pomešane.

---

Bolje:

```text
 id="x9m4qd"
project/

├── cmd/

├── internal/

├── pkg/

├── configs/

└── docs/
```

---

# Standardni direktorijumi u Go projektima

Najčešća profesionalna struktura:

```text
 id="m2k7vz"
project/

├── cmd/

├── internal/

├── pkg/

├── api/

├── configs/

├── scripts/

├── docs/

├── go.mod

└── README.md
```

---

Ne mora svaki projekat imati sve direktorijume.

Koristi se samo ono što ima smisla.

---

# `cmd/` direktorijum

`cmd` sadrži entry point-e aplikacije.

Drugim rečima:

> Sve što može da se pokrene kao izvršni program.

---

Primer:

```text
 id="j4p9ws"
cmd/

├── api/

│   └── main.go

└── worker/

    └── main.go
```

---

Imamo dve aplikacije:

```text
 id="5q8mxn"
API Server

+

Background Worker
```

---

Svaki `main.go` pripada:

```go
package main
```

---

Primer:

```go
package main

func main() {

	startServer()

}
```

---

# Zašto `cmd` postoji?

Umesto:

```text
 id="c3w8mq"
main.go

main_worker.go

main_admin.go
```

dobijamo:

```text
 id="q7n2xm"
cmd/

├── server/

├── worker/

└── admin/
```

---

Svaka aplikacija ima svoj životni ciklus.

---

# `internal/` direktorijum

Jedan od najvažnijih Go koncepata.

`internal` sadrži kod koji:

* pripada aplikaciji;
* nije namenjen eksternim korisnicima.

---

Primer:

```text
 id="9f3wkt"
internal/

├── user/

├── database/

└── payment/
```

---

Drugi projekti ne mogu importovati:

```go
import "company.com/app/internal/user"
```

---

Go automatski zabranjuje.

---

# Zašto koristiti internal?

Zato što želimo:

```text
 id="n6p2vq"
Application Logic

=

Private
```

---

Primer:

```text
 id="5s9qmv"
company.com/shop

        |

        v

internal/payment
```

---

Samo:

```text
company.com/shop
```

može koristiti taj kod.

---

# `pkg/` direktorijum

`pkg` se koristi za javno deljive pakete.

Primer:

```text
 id="w3k8qa"
pkg/

├── logger/

└── validator/
```

---

Ideja:

Drugi projekti mogu koristiti:

```go
import "company.com/project/pkg/logger"
```

---

Primer:

```go
package logger

func Info(message string) {

}
```

---

# `internal` vs `pkg`

Veoma često pitanje.

---

## internal

Znači:

```text
Private Application Code
```

---

Primer:

```text
internal/order
```

---

Koristi samo aplikacija.

---

## pkg

Znači:

```text
Reusable Public Code
```

---

Primer:

```text
pkg/logger
```

---

Može koristiti više projekata.

---

# `api/` direktorijum

Često se koristi za API definicije.

Primer:

```text
 id="r6v9kp"
api/

├── openapi/

│   └── swagger.yaml

└── protobuf/

    └── user.proto
```

---

Sadrži:

* OpenAPI specifikacije;
* protobuf definicije;
* GraphQL schema fajlove.

---

# `configs/`

Sadrži konfiguraciju.

Primer:

```text
 id="m5x8pq"
configs/

├── config.yaml

└── development.yaml
```

---

Primer:

```yaml
server:
  port: 8080

database:
  host: localhost
```

---

Važno:

Tajni podaci ne treba da budu ovde.

---

Loše:

```yaml
password: my-secret-password
```

---

Bolje:

```text
Environment Variables
```

---

# `scripts/`

Automatizacione skripte.

Primer:

```text
 id="y8m3qv"
scripts/

├── build.sh

├── deploy.sh

└── migrate.sh
```

---

Primer:

```bash
./scripts/build.sh
```

---

# `docs/`

Dokumentacija projekta.

Primer:

```text
 id="k3x7mp"
docs/

├── architecture.md

├── api.md

└── deployment.md
```

---

Sadrži:

* tehničku dokumentaciju;
* dizajn odluke;
* arhitekturu.

---

# `README.md`

Svaki ozbiljan projekat treba da ima README.

Primer:

```text
 id="n7q4mw"
project/

└── README.md
```

---

README obično sadrži:

* opis projekta;
* instalaciju;
* pokretanje;
* konfiguraciju;
* testiranje.

---

Primer:

```md
# User Service

## Run

go run ./cmd/api

## Test

go test ./...
```

---

# Primer kompletne strukture

Realističan servis:

```text
 id="h5p8nx"
user-service/

├── cmd/

│   ├── api/

│   │   └── main.go

│   └── worker/

│       └── main.go

│

├── internal/

│   ├── user/

│   │   ├── service.go

│   │   └── repository.go

│   │

│   └── database/

│       └── connection.go

│

├── pkg/

│   └── logger/

│       └── logger.go

│

├── configs/

│   └── config.yaml

├── go.mod

├── go.sum

└── README.md
```

---

# Dependency flow

Dobra arhitektura:

```text
 id="b8q2mx"
cmd

 |

 v

internal

 |

 v

pkg
```

---

Primer:

```text
 id="r9m3kv"
API

 |

 v

User Service

 |

 v

Logger
```

---

# Šta ne treba raditi?

---

## 1. Kreirati strukturu pre potrebe

Loše:

```text
 id="p5v7mw"
project/

├── 50 empty folders
```

---

Pravilo:

> Struktura treba da raste zajedno sa aplikacijom.

---

## 2. Koristiti pkg za sve

Loše:

```text
 id="t4x8mq"
pkg/

├── user

├── payment

└── database
```

ako su to samo interni delovi aplikacije.

---

## 3. Mešati business i infrastructure kod

Loše:

```go
func CreateUser() {

	saveToPostgres()

	sendEmail()

	validateInput()

}
```

---

Bolje:

```text
 id="w6q3kp"
Handler

   |

Service

   |

Repository
```

---

# Minimalna struktura za male projekte

Za mali servis:

```text
 id="v9m4qx"
project/

├── main.go

├── user/

├── database/

├── go.mod

└── README.md
```

---

Ne treba odmah koristiti kompleksnu strukturu.

---

# Struktura treba da služi kodu

Najvažnije pravilo:

> Arhitektura nije cilj sama po sebi. Ona treba da olakša razvoj.

---

Dobro:

```text
 id="x7n2mq"
Simple Structure

+

Clear Responsibilities
```

---

Loše:

```text
 id="c4p8vz"
Complex Structure

-

No Benefit
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* ulogu `cmd/` direktorijuma;
* razliku između `internal/` i `pkg/`;
* gde se čuvaju konfiguracije;
* gde se nalazi dokumentacija;
* kako izgleda realna Go aplikacija;
* kako organizovati kod za skaliranje.

---

# Zaključak

Profesionalni Go projekti nisu veliki zato što imaju mnogo fajlova.

Veliki projekti uspešno rastu zato što imaju:

```text
 id="q8m3vx"
Clear Packages

+

Clear Responsibilities

+

Simple Dependencies
```

Dobra struktura omogućava timu da brzo razume projekat i bezbedno ga menja.

U sledećem delu prelazimo na **rad sa Git-om, verzionisanjem koda i pravilima koja se koriste u profesionalnim Go repozitorijumima**.

---

# Git Integration and Repository Management

U prethodnom delu obradili smo profesionalnu strukturu Go projekta:

- `cmd/`;
- `internal/`;
- `pkg/`;
- `configs/`;
- `docs/`;
- `README.md`.

Sada prelazimo na sledeći važan korak:

> **Kako Go projekat pripremiti za rad sa Git sistemom za verzionisanje koda?**

U profesionalnom razvoju skoro svaki Go projekat koristi:

- Git;
- remote repository;
- code review;
- branch workflow;
- CI/CD pipeline.

---

# Šta je Git?

Git je distribuirani sistem za kontrolu verzija.

Omogućava:

- praćenje promena;
- čuvanje istorije;
- rad više programera;
- vraćanje prethodnih verzija;
- saradnju kroz grane.

---

Bez Git-a:

```text id="k4m7px"
Developer

    |

    v

Changes

    |

    ?

````

---

Sa Git-om:

```text id="q8m2vx"
Code

 |

 v

Git History

 |

 +-- Commit 1

 +-- Commit 2

 +-- Commit 3
```

---

# Inicijalizacija Git repozitorijuma

U root direktorijumu projekta:

```bash id="r6p2mz"
git init
```

---

Primer:

```text id="x3w8qa"
user-service/

├── go.mod

├── main.go

└── .git/
```

---

`.git` direktorijum sadrži:

* istoriju;
* konfiguraciju;
* objekte;
* reference.

---

# Prvi commit

Provera statusa:

```bash id="m4q8nv"
git status
```

---

Dodavanje fajlova:

```bash id="y7x3qp"
git add .
```

---

Kreiranje commit-a:

```bash id="c8m5vd"
git commit -m "Initial Go project setup"
```

---

Rezultat:

```text id="t9q2mx"
Repository

 |

 v

Initial Commit
```

---

# Šta commit predstavlja?

Commit je:

> Snapshot trenutnog stanja projekta.

---

Primer:

Commit 1:

```text id="z5p8km"
Create Go module
```

---

Commit 2:

```text id="n2x7vq"
Add user package
```

---

Commit 3:

```text id="m8q4wp"
Add database layer
```

---

Istorija:

```text id="r3k9mx"
Commit 1

    |

    v

Commit 2

    |

    v

Commit 3
```

---

# `.gitignore`

Go projekti generišu fajlove koje ne želimo u repozitorijumu.

Primer:

* build fajlovi;
* IDE konfiguracija;
* lokalni cache;
* secrets.

Za to koristimo:

```text id="v6p2qw"
.gitignore
```

---

Primer:

```text id="c5m8nx"
project/

├── .gitignore

├── go.mod

└── main.go
```

---

# Tipičan Go `.gitignore`

Primer:

```gitignore id="h3w9qm"
# Binaries
*.exe
*.out
*.app

# Test binaries
*.test

# Coverage files
coverage.out

# IDE
.idea/
.vscode/

# OS files
.DS_Store

# Environment files
.env
```

---

# Zašto ignorisati build fajlove?

Primer:

Pokrenemo:

```bash id="p9m3xv"
go build
```

Dobijamo:

```text id="a7q4nw"
app-binary
```

---

Ne želimo:

```text
Git Repository

    |

    v

Binary Files
```

---

Razlog:

* veliki fajlovi;
* zavise od OS-a;
* mogu se ponovo generisati.

---

# go.mod i go.sum u Git-u

Veoma važno pravilo:

U repozitorijum uključujemo:

```text id="w5q8mp"
go.mod

go.sum
```

---

Zašto?

Zato što definišu:

```text id="b3x7mq"
Exact Dependencies
```

---

Drugi developer radi:

```bash id="f8n2vx"
git clone project
```

---

Zatim:

```bash id="c7m5pz"
go mod download
```

---

Dobija isto okruženje.

---

# Remote repository

Lokalni Git nije dovoljan.

Obično koristimo:

* GitHub;
* GitLab;
* Bitbucket.

---

Tok:

```text id="w9q4mx"
Local Repository

        |

        v

Remote Repository
```

---

Dodavanje remote-a:

```bash id="x4m7qp"
git remote add origin <repository-url>
```

---

Provera:

```bash id="h8p3vz"
git remote -v
```

---

# Push koda

Slanje promena:

```bash id="m2q7xv"
git push origin main
```

---

Tok:

```text id="q9x4mp"
Developer

    |

    v

Local Git

    |

    v

Remote Git
```

---

# Branch koncept

Branch predstavlja paralelnu liniju razvoja.

---

Primer:

```text id="z8m5qx"
main

 |

 +----------------

                  |

                  v

             feature/user
```

---

Glavna grana:

```text id="n7p3mx"
main
```

---

Nova funkcionalnost:

```text id="a4q8vz"
feature/login
```

---

# Zašto koristiti branch-e?

Bez branch-eva:

```text id="k2m9xp"
Everyone edits main
```

---

Problem:

* konflikti;
* nestabilan kod;
* teži review.

---

Sa branch-evima:

```text id="j6q3mw"
main

 |

 +-- feature-A

 |

 +-- feature-B
```

---

# Tipičan Go development workflow

Primer:

1. Kreiramo branch:

```bash id="x7m2pq"
git checkout -b feature/authentication
```

---

2. Pišemo kod.

---

3. Formatiramo:

```bash id="r5q8mx"
go fmt ./...
```

---

4. Testiramo:

```bash id="m3q7vx"
go test ./...
```

---

5. Commit:

```bash id="q4n8mp"
git commit -m "Add authentication service"
```

---

6. Push:

```bash id="z9x5mq"
git push origin feature/authentication
```

---

# Pull Request koncept

U profesionalnim timovima promene se ne spajaju direktno.

Tok:

```text id="c7m2vx"
Feature Branch

        |

        v

Pull Request

        |

        v

Code Review

        |

        v

Merge
```

---

# Code Review i Go

Go ima veoma jednostavan stil.

Review obično proverava:

* čitljivost;
* testove;
* error handling;
* package dizajn;
* concurrency probleme.

---

Primer:

Loše:

```go
func DoEverything() {

}
```

---

Bolje:

```go
func ValidateUser()

func SaveUser()

func SendEmail()
```

---

# Commit poruke

Dobre commit poruke:

```text id="x8m4pq"
Add user repository

Fix database connection leak

Update API validation
```

---

Loše:

```text id="v5q2mx"
changes

update

fix
```

---

Commit treba da objasni:

```text id="q7m3vz"
What changed
```

---

# Conventional Commits

Čest standard:

```text id="n8x5mq"
type(scope): description
```

---

Primeri:

```text
feat(user): add registration endpoint
```

---

```text
fix(db): close connection correctly
```

---

```text
test(auth): add login tests
```

---

Tipovi:

```text id="p4m8xq"
feat

fix

docs

test

refactor

chore
```

---

# Tagovanje verzija

Git podržava tagove:

```bash id="w3q7mx"
git tag v1.0.0
```

---

Primer:

```text id="m9x4qp"
v1.0.0

v1.1.0

v2.0.0
```

---

Ovo se povezuje sa:

* release procesom;
* Semantic Versioning-om;
* Go modul verzijama.

---

# Release workflow

Primer:

```text id="h5q8mx"
Development

      |

      v

Testing

      |

      v

Tag v1.0.0

      |

      v

Release
```

---

# GitHub Actions i Go

Često se koristi CI.

Primer:

```text id="z4m7px"
Push Code

    |

    v

Run Tests

    |

    v

Build Application
```

---

Automatske komande:

```bash
go test ./...
go vet ./...
go build ./...
```

---

# Najčešće greške

---

## 1. Commitovanje generated fajlova

Loše:

```text id="x6m2qv"
app-binary

coverage.out
```

---

Rešenje:

`.gitignore`

---

## 2. Commitovanje secrets

Nikada:

```text id="p8m4qx"
password=secret
```

---

Koristiti:

```text
Environment Variables
```

---

## 3. Direktan rad na main branch-u

Loše:

```text id="m3q9vx"
main

+

experimental changes
```

---

Bolje:

```text
feature branch
```

---

# Profesionalni Go repository izgleda ovako

```text id="k7q2mx"
user-service/

├── .git/

├── .gitignore

├── README.md

├── go.mod

├── go.sum

├── cmd/

├── internal/

├── pkg/

└── docs/
```

---

# Mentalni model

Zapamtiti:

```text id="v4m8qx"
Git

=

History


Branch

=

Isolation


Commit

=

Snapshot


Tag

=

Release Version
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* kako inicijalizovati Git projekat;
* šta ide u `.gitignore`;
* zašto se `go.mod` i `go.sum` commit-uju;
* kako koristiti branch workflow;
* kako izgleda Pull Request proces;
* kako Git povezuje verzije projekta sa release-ovima.

---

# Zaključak

Go projekat nije samo kod.

Profesionalni projekat predstavlja kombinaciju:

```text id="w8m3qx"
Source Code

+

Dependencies

+

Version Control

+

Documentation

+

Automation
```

Dobar Git workflow omogućava timu da sigurno razvija i održava Go aplikacije.

U poslednjem delu ovog poglavlja objedinićemo sve prethodne koncepte i napravićemo kompletan pregled procesa kreiranja profesionalnog Go projekta od nule.

---

# Complete Go Project Creation Workflow

U prethodnih 13 delova detaljno smo prošli kroz kompletan proces rada sa Go projektima:

- inicijalizacija projekta;
- Go Modules;
- package organizacija;
- dependency management;
- verzionisanje;
- multi-package aplikacije;
- Go tooling;
- profesionalna struktura projekta;
- Git workflow.

U ovom završnom delu objedinićemo sve koncepte u jedan kompletan workflow.

Cilj:

> Razumeti kako izgleda kreiranje profesionalnog Go projekta od prvog direktorijuma do produkcionog repozitorijuma.

---

# Faza 1: Kreiranje projekta

Prvi korak je kreiranje direktorijuma.

Primer:

```bash id="m4x8qp"
mkdir user-service
````

---

Ulazak u direktorijum:

```bash id="q8n3mv"
cd user-service
```

---

Početna struktura:

```text id="r5p2wx"
user-service/
```

---

# Faza 2: Inicijalizacija Go modula

Kreiramo module:

```bash id="z7m4qx"
go mod init example.com/user-service
```

---

Dobijamo:

```text id="x3q8mv"
user-service/

└── go.mod
```

---

`go.mod`:

```go id="p5n9wx"
module example.com/user-service

go 1.26
```

---

Ovaj fajl definiše:

* identitet projekta;
* Go verziju;
* dependencies.

---

# Faza 3: Kreiranje osnovne strukture

Za profesionalni servis:

```text id="w8m3qx"
user-service/

├── cmd/

├── internal/

├── pkg/

├── configs/

├── docs/

├── go.mod

└── README.md
```

---

# Faza 4: Kreiranje executable aplikacije

Dodajemo:

```text id="q4m8xz"
cmd/

└── api/

    └── main.go
```

---

`main.go`:

```go id="m9x2qw"
package main

import "fmt"

func main() {

	fmt.Println("User Service started")

}
```

---

Pokretanje:

```bash id="v7q3mx"
go run ./cmd/api
```

---

# Faza 5: Dodavanje poslovne logike

Kreiramo:

```text id="n5q8mx"
internal/

└── user/

    ├── service.go

    └── model.go
```

---

`model.go`:

```go id="x8m2qp"
package user

type User struct {
	Name string
	Age  int
}
```

---

`service.go`:

```go id="c4m7vx"
package user

func CreateUser(name string, age int) User {

	return User{
		Name: name,
		Age:  age,
	}

}
```

---

# Faza 6: Povezivanje paketa

`main.go`:

```go id="z2m8qx"
package main

import (
	"fmt"

	"example.com/user-service/internal/user"
)

func main() {

	u := user.CreateUser(
		"Marko",
		30,
	)

	fmt.Println(u)

}
```

---

Tok zavisnosti:

```text id="m7q3vx"
main

 |

 v

internal/user
```

---

# Faza 7: Dodavanje eksternih dependency-ja

Primer:

```bash id="x5m9qw"
go get github.com/example/logger
```

---

Go automatski ažurira:

```text id="q8m4vx"
go.mod

+

go.sum
```

---

# Faza 8: Čišćenje dependency-ja

Pokrenuti:

```bash id="r3x7mq"
go mod tidy
```

---

Rezultat:

* uklonjeni nepotrebni paketi;
* dodate nedostajuće zavisnosti;
* ažuriran checksum.

---

# Faza 9: Formatiranje koda

Pre commit-a:

```bash id="p6m2xq"
go fmt ./...
```

---

Go formatira:

* tabove;
* razmake;
* strukturu koda.

---

# Faza 10: Pokretanje testova

Dodajemo test:

```text id="w4q8mx"
internal/user/

├── service.go

└── service_test.go
```

---

Pokretanje:

```bash id="z9m3qx"
go test ./...
```

---

Rezultat:

```text id="1k7m9v"
PASS
ok
```

---

# Faza 11: Statčka analiza

Pokrećemo:

```bash id="v3m8qx"
go vet ./...
```

---

Cilj:

Pronaći:

* moguće bugove;
* sumnjive konstrukcije;
* probleme sa formatima.

---

# Faza 12: Kreiranje build-a

Kompajliranje:

```bash id="x7m4qp"
go build ./cmd/api
```

---

Dobijamo:

```text id="m2q8vx"
api binary
```

---

Pokretanje:

```bash id="q5m9wx"
./api
```

---

# Faza 13: Git inicijalizacija

Pokrećemo:

```bash id="n8x3mq"
git init
```

---

Dodajemo:

```bash id="r4m7vx"
git add .
```

---

Commit:

```bash id="z6q2mx"
git commit -m "Initial Go service setup"
```

---

# Faza 14: Kreiranje `.gitignore`

Primer:

```gitignore id="x9m3qw"
*.exe
*.out
*.test

.idea/
.vscode/

.env
```

---

Sprečavamo:

* commit binary fajlova;
* IDE konfiguracije;
* secrets.

---

# Faza 15: Kreiranje README dokumentacije

Minimalni README:

````md id="c8m2qx"
# User Service

## Run

```bash
go run ./cmd/api
````

## Test

```bash
go test ./...
```

````

---

README treba da omogući novom developeru:

- razumevanje projekta;
- pokretanje aplikacije;
- razvoj.

---

# Faza 16: Finalna struktura projekta

Kompletan rezultat:

```text id="w7m4qx"
user-service/

├── .git/

├── .gitignore

├── README.md

├── go.mod

├── go.sum

│

├── cmd/

│   └── api/

│       └── main.go

│

├── internal/

│   └── user/

│       ├── model.go

│       ├── service.go

│       └── service_test.go

│

├── pkg/

│   └── logger/

│       └── logger.go

│

├── configs/

│   └── config.yaml

│

└── docs/

    └── architecture.md
````

---

# Finalni razvojni workflow

Profesionalni Go developer svakodnevno radi:

```text id="q2m8vx"
Modify Code

      |

      v

go fmt ./...

      |

      v

go test ./...

      |

      v

go vet ./...

      |

      v

go build

      |

      v

git commit

      |

      v

Pull Request
```

---

# Go Project Checklist

Pre nego što projekat ode u tim ili produkciju:

## Project Setup

✅ `go.mod` postoji
✅ struktura projekta je jasna
✅ package odgovornosti su definisane

---

## Dependencies

✅ `go.mod` ažuriran
✅ `go.sum` postoji
✅ `go mod tidy` izvršen

---

## Code Quality

✅ `go fmt` pokrenut
✅ testovi prolaze
✅ `go vet` prolazi

---

## Repository

✅ `.gitignore` postoji
✅ README postoji
✅ commit istorija je jasna

---

# Najvažniji principi

Tokom celog poglavlja ponavljaju se isti principi.

---

## 1. Jednostavnost

Go favorizuje:

```text id="h6m3qx"
Simple Structure
```

umesto:

```text id="p8q2mw"
Complex Abstraction
```

---

## 2. Jasne odgovornosti

Svaki paket treba da ima razlog postojanja.

```text id="n3x7mq"
One Package

=

One Responsibility
```

---

## 3. Eksplicitne zavisnosti

Go preferira:

```text id="w5m8qx"
Explicit Dependency

over

Hidden Magic
```

---

## 4. Reproducibilan build

Isti kod treba da proizvodi isti rezultat:

```text id="r9q4mx"
Developer Machine

        =

CI Machine

        =

Production Build
```

---

# Šta smo naučili u poglavlju "Starting a Project"

Nakon završetka ovog poglavlja treba potpuno razumeti:

* kako se kreira Go projekat;
* kako funkcioniše Go Module sistem;
* kako se organizuju package-i;
* kako se upravlja dependency-jima;
* kako funkcioniše Semantic Versioning;
* kako se kreiraju multi-package aplikacije;
* kako koristiti Go tooling;
* kako izgleda profesionalna struktura projekta;
* kako povezati Go projekat sa Git workflow-om.

---

# Završna poruka

Kreiranje Go projekta nije samo:

```text
go mod init
```

To je proces izgradnje stabilne osnove:

```text id="t7m2qx"
Module

+

Packages

+

Dependencies

+

Tooling

+

Version Control

=

Professional Go Project
```

Dobar početak projekta značajno smanjuje buduću kompleksnost.

Kada su osnove pravilno postavljene, ostatak razvoja postaje fokusiran na ono što je najvažnije:

> Pisanje kvalitetnog, održivog i pouzdanog Go koda.

---

# Sledeće poglavlje

Sledeće poglavlje:

```text
03. Working With Primitive Data Types
```

