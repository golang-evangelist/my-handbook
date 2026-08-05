# 03. Working with Primitive Data Types

> **Go Mastery**<br\>  
> **Basic → 01 Getting Started**<br\>  
> Poglavlje: **03 Working with Primitive Data Types**

---

# Introduction to Primitive Data Types

Go je statički tipiziran programski jezik.

To znači:

> Svaka promenljiva u Go programu ima unapred definisan tip koji compiler proverava tokom procesa kompajliranja.

Za razliku od dinamički tipiziranih jezika gde se tip često određuje u runtime-u, Go zahteva da tip podataka bude poznat compiler-u.

---

Primer:

```go
package main

import "fmt"

func main() {

	var age int = 30

	fmt.Println(age)

}
````

---

Ovde:

```go
var age int = 30
```

znači:

* ime promenljive: `age`
* tip promenljive: `int`
* vrednost: `30`

Compiler zna:

```text
age → integer value
```

---

# Šta su Primitive Data Types?

Primitive data types (osnovni tipovi podataka) predstavljaju najjednostavnije vrednosti koje jezik direktno podržava.

U Go-u osnovni tipovi uključuju:

```text
Boolean

Integer

Floating Point

Complex Numbers

String
```

---

Grafički prikaz:

```text
Primitive Types

        |

        +-- Boolean

        |

        +-- Numeric Types

        |       |

        |       +-- Integer

        |       |

        |       +-- Floating Point

        |       |

        |       +-- Complex

        |

        +-- String
```

---

# Zašto su primitive data types važne?

Svi kompleksniji tipovi u Go-u grade se od osnovnih tipova.

Na primer:

## Arrays

```go
[5]int
```

koriste:

```text
int
```

---

## Structs

```go
type User struct {

	Name string

	Age int

}
```

koriste:

```text
string

int
```

---

## Slices

```go
[]string
```

koriste:

```text
string
```

---

Razumevanje osnovnih tipova je osnova za razumevanje:

* memorije;
* struktura podataka;
* performansi;
* pointer-a;
* garbage collector-a.

---

# Go Type System

Go razlikuje:

## Basic Types

Osnovni tipovi:

```text
int

float64

bool

string
```

---

## Composite Types

Složeni tipovi:

```text
array

slice

map

struct

pointer

function

channel
```

---

## User Defined Types

Korisnički definisani tipovi:

```go
type UserID int
```

---

Primer:

```go
package main

import "fmt"

type UserID int

func main() {

	var id UserID = 100

	fmt.Println(id)

}
```

---

Ovde:

```text
UserID

nije isto što i

int
```

Iako imaju istu osnovnu reprezentaciju.

---

# Type Safety in Go

Go compiler sprečava mnoge greške.

Primer:

```go
package main

func main() {

	var age int = 30

	var name string = "Marko"

	age = name

}
```

---

Compiler greška:

```text
cannot use name (variable of type string)
as int value
```

---

Razlog:

```text
int

!=

string
```

---

# Pregled Primitive Data Types

Go podržava sledeće osnovne kategorije.

---

# 1. Boolean Type

Boolean predstavlja logičku vrednost.

Moguće vrednosti:

```text
true

false
```

---

Primer:

```go
var isActive bool = true
```

---

---

# 2. Integer Types

Integer tipovi predstavljaju cele brojeve.

Primeri:

```text
10

-5

1000
```

---

Go ima više integer tipova:

```text
int

int8

int16

int32

int64
```

---

Unsigned verzije:

```text
uint

uint8

uint16

uint32

uint64
```

---

---

# 3. Floating Point Types

Koriste se za decimalne brojeve.

Go podržava:

```text
float32

float64
```

---

Primer:

```go
var price float64 = 19.99
```

---

---

# 4. Complex Numbers

Go ima ugrađenu podršku za kompleksne brojeve.

Tipovi:

```text
complex64

complex128
```

---

Primer:

```go
var c complex64 = 1 + 2i
```

---

---

# 5. String Type

String predstavlja tekst.

Primer:

```go
var name string = "Go"
```

---

String u Go-u je:

```text
immutable sequence of bytes
```

---

Primer:

```go
message := "Hello Go"
```

---

# Zero Values

Jedna od najvažnijih Go karakteristika:

> Svaka promenljiva u Go-u automatski dobija svoju zero value ako nije inicijalizovana.

---

Primer:

```go
package main

import "fmt"

func main() {

	var age int

	var name string

	var active bool

	fmt.Println(age)

	fmt.Println(name)

	fmt.Println(active)

}
```

---

Rezultat:

```text
0

""

false
```

---

# Zero Values tabela

| Tip     | Zero Value |
| ------- | ---------- |
| int     | `0`        |
| float64 | `0.0`      |
| bool    | `false`    |
| string  | `""`       |
| pointer | `nil`      |

---

Ovo omogućava da Go kod bude jednostavniji.

Nema potrebe za:

```text
null checks everywhere
```

kao u nekim drugim jezicima.

---

# Type Inference

Go može automatski zaključiti tip.

Primer:

```go
name := "Marko"
```

Compiler vidi:

```text
"Marko"

=

string
```

---

Isto:

```go
age := 30
```

postaje:

```text
int
```

---

Primer:

```go
package main

import "fmt"

func main() {

	name := "Go"

	age := 15

	fmt.Println(name)

	fmt.Println(age)

}
```

---

# Explicit Declaration

Možemo eksplicitno navesti tip:

```go
var age int = 30
```

---

Ili:

```go
var age = 30
```

---

Ili:

```go
age := 30
```

---

Sva tri rezultata:

```text
age → int
```

---

# Kada koristiti koji način?

## Kratak oblik

Najčešće:

```go
age := 30
```

---

Koristi se unutar funkcija.

---

## Explicit type

Kada je potrebno:

```go
var timeout time.Duration = 5
```

---

ili:

```go
var users []User
```

---

# Typed Constants

Konstante takođe imaju tip.

Primer:

```go
const maxUsers int = 100
```

---

Ovde:

```text
maxUsers

=

int constant
```

---

# Untyped Constants

Go podržava i konstante bez eksplicitnog tipa.

Primer:

```go
const pi = 3.14159
```

---

Compiler kasnije određuje odgovarajući tip.

---

# Primitive Types i memorija

Različiti tipovi zauzimaju različitu količinu memorije.

Primer:

```text
int8

=

1 byte
```

---

```text
int64

=

8 bytes
```

---

Zato izbor tipa može uticati na:

* memoriju;
* performanse;
* cache efikasnost.

---

# Primitive Types i performanse

Primer:

Velika kolekcija:

```go
[]int64
```

može koristiti više memorije nego:

```go
[]int32
```

---

Ali:

```go
int
```

je često preporučeni izbor jer je optimizovan za arhitekturu sistema.

---

# Go Philosophy

Go preferira:

```text
Simple

Explicit

Readable
```

---

Zbog toga ne postoji veliki broj implicitnih konverzija.

Primer:

Ne može:

```go
var x int = 10.5
```

---

Mora:

```go
var x int = int(10.5)
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su primitive data types;
* koje osnovne tipove Go podržava;
* razliku između basic i composite tipova;
* kako funkcioniše type safety;
* šta su zero values;
* kako compiler zaključuje tip;
* razliku između explicit i implicit deklaracije.

---

# Zaključak

Primitive data types predstavljaju osnovu svakog Go programa.

Pre nego što krenemo na:

* arrays;
* slices;
* maps;
* structs;
* pointers;

moramo razumeti kako Go predstavlja osnovne vrednosti.

U sledećem delu detaljno ćemo obraditi **Boolean tip i logičke operacije u Go-u**.

---
# Boolean Type (`bool`)

Boolean tip predstavlja jednu od najosnovnijih vrednosti u programiranju.

Njegova svrha je da predstavlja stanje koje može imati samo dve moguće vrednosti:

```text
true

false
````

U Go jeziku boolean tip se označava kao:

```go
bool
```

---

# Kreiranje Boolean promenljive

Najjednostavniji primer:

```go
package main

import "fmt"

func main() {

	var isActive bool = true

	fmt.Println(isActive)

}
```

Rezultat:

```text
true
```

---

Ovde:

```go
var isActive bool = true
```

znači:

```text
Name:

isActive


Type:

bool


Value:

true
```

---

# Zero Value za Boolean

Ako boolean promenljiva nije inicijalizovana, Go joj automatski dodeljuje:

```text
false
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var completed bool

	fmt.Println(completed)

}
```

---

Rezultat:

```text
false
```

---

Go automatski radi:

```go
var completed bool = false
```

---

# Zašto postoji Zero Value?

Go filozofija:

> Svaka promenljiva treba da ima validnu početnu vrednost.

U mnogim jezicima:

```text
uninitialized variable

=

unknown value
```

---

U Go-u:

```text
uninitialized variable

=

safe zero value
```

---

Primer:

```go
var ready bool
```

nije:

```text
undefined
```

nego:

```text
false
```

---

# Boolean Literals

Boolean vrednosti imaju samo dva literala:

```go
true
```

i:

```go
false
```

---

Primer:

```go
package main

import "fmt"

func main() {

	fmt.Println(true)

	fmt.Println(false)

}
```

---

Rezultat:

```text
true

false
```

---

# Boolean nije Integer

U nekim jezicima boolean može implicitno biti tretiran kao broj.

Na primer:

```text
true  → 1

false → 0
```

---

Go to ne dozvoljava.

Primer:

```go
package main

func main() {

	var active bool = true

	var value int = active

}
```

---

Compiler greška:

```text
cannot use active (variable of type bool)
as int value
```

---

Mora se eksplicitno izvršiti konverzija, ali:

```go
int(true)
```

takođe nije dozvoljeno.

---

Razlog:

Go želi jasnu semantiku.

---

# Boolean Operators

Go podržava standardne logičke operatore.

Tri osnovna operatora:

| Operator | Naziv | Primer         |    |       |   |        |
| -------- | ----- | -------------- | -- | ----- | - | ------ |
| `&&`     | AND   | `true && true` |    |       |   |        |
| `        |       | `              | OR | `true |   | false` |
| `!`      | NOT   | `!true`        |    |       |   |        |

---

# AND Operator (`&&`)

AND vraća `true` samo ako su oba izraza `true`.

Tabela:

| A     | B     | A && B |
| ----- | ----- | ------ |
| false | false | false  |
| false | true  | false  |
| true  | false | false  |
| true  | true  | true   |

---

Primer:

```go
package main

import "fmt"

func main() {

	result := true && true

	fmt.Println(result)

}
```

---

Rezultat:

```text
true
```

---

Drugi primer:

```go
isAdult := true

hasTicket := false

allowed := isAdult && hasTicket

fmt.Println(allowed)
```

---

Rezultat:

```text
false
```

---

Logika:

```text
Adult?

YES


Has Ticket?

NO


Access?

NO
```

---

# OR Operator (`||`)

OR vraća `true ako je makar jedan izraz true.

Tabela:

| A | B | A || B |
|-|-|-|
| false | false | false |
| false | true | true |
| true | false | true |
| true | true | true |

---

Primer:

```go
package main

import "fmt"

func main() {

	result := false || true

	fmt.Println(result)

}
```

---

Rezultat:

```text
true
```

---

Praktičan primer:

```go
isAdmin := false

isOwner := true

canDelete := isAdmin || isOwner

fmt.Println(canDelete)
```

---

Rezultat:

```text
true
```

---

Logika:

```text
Admin?

NO


Owner?

YES


Permission?

YES
```

---

# NOT Operator (`!`)

NOT menja vrednost:

```text
true  → false

false → true
```

---

Primer:

```go
package main

import "fmt"

func main() {

	active := true

	fmt.Println(!active)

}
```

---

Rezultat:

```text
false
```

---

Primer:

```go
disabled := false

enabled := !disabled

fmt.Println(enabled)
```

---

Rezultat:

```text
true
```

---

# Operator Precedence

Kada postoji više operatora, Go koristi pravila prioriteta.

Primer:

```go
result := true || false && false
```

---

Prvo se izvršava:

```go
false && false
```

zatim:

```go
true || false
```

---

Rezultat:

```text
true
```

---

Bolje koristiti zagrade:

```go
result := true || (false && false)
```

---

Kod postaje jasniji.

---

# Short-Circuit Evaluation

Jedna veoma važna karakteristika boolean izraza u Go-u.

Go ne izvršava uvek ceo izraz.

---

Primer:

```go
package main

import "fmt"

func main() {

	result := false && check()

	fmt.Println(result)

}

func check() bool {

	fmt.Println("Executed")

	return true

}
```

---

Rezultat:

```text
false
```

---

Poruka:

```text
Executed
```

se NE ispisuje.

---

Zašto?

Zato što:

```text
false && anything

=

false
```

Go zna rezultat unapred.

---

# Short-Circuit kod OR operatora

Primer:

```go
result := true || check()
```

---

Go neće pozvati:

```go
check()
```

jer:

```text
true || anything

=

true
```

---

Ovo se često koristi za:

* validaciju;
* proveru nil vrednosti;
* optimizaciju.

---

# Boolean u Control Flow-u

Boolean vrednosti se najčešće koriste sa:

* `if`;
* `for`;
* `switch`.

---

Primer:

```go
package main

import "fmt"

func main() {

	isLoggedIn := true

	if isLoggedIn {

		fmt.Println("Welcome")

	}

}
```

---

Rezultat:

```text
Welcome
```

---

# Boolean izrazi u if naredbama

Ne moramo čuvati rezultat u promenljivoj.

Primer:

```go
age := 20

if age >= 18 {

	fmt.Println("Adult")

}
```

---

Izraz:

```go
age >= 18
```

vraća:

```text
bool
```

---

Dakle:

```text
Comparison

        |

        v

     bool

        |

        v

      if
```

---

# Poređenje vrednosti

Svi operatori poređenja vraćaju boolean.

Operatori:

| Operator | Značenje          |
| -------- | ----------------- |
| `==`     | jednako           |
| `!=`     | različito         |
| `<`      | manje             |
| `>`      | veće              |
| `<=`     | manje ili jednako |
| `>=`     | veće ili jednako  |

---

Primer:

```go
age := 25

isAdult := age >= 18

fmt.Println(isAdult)
```

---

Rezultat:

```text
true
```

---

# Boolean funkcije

Funkcije često vraćaju `bool`.

Primer:

```go
func isEven(number int) bool {

	return number%2 == 0

}
```

---

Korišćenje:

```go
result := isEven(10)

fmt.Println(result)
```

---

Rezultat:

```text
true
```

---

# Naming Convention za Boolean promenljive

Dobro ime treba da sugeriše stanje.

Dobri primeri:

```go
isActive

isValid

hasPermission

canDelete

shouldRetry
```

---

Loši primeri:

```go
data

value

flag
```

---

Primer:

Bolje:

```go
isConnected := true
```

nego:

```go
connection := true
```

---

# Boolean i Clean Code

Kod:

```go
if !isValid {

}
```

je čitljiv.

---

Ali:

```go
if !(!disabled) {

}
```

je komplikovan.

---

Pravilo:

> Boolean izrazi treba da budu jednostavni i lako razumljivi.

---

# Najčešće greške

---

## 1. Poređenje sa true

Nepotrebno:

```go
if isActive == true {

}
```

---

Bolje:

```go
if isActive {

}
```

---

## 2. Negativna imena

Teže čitljivo:

```go
if !notDisabled {

}
```

---

Bolje:

```go
if enabled {

}
```

---

## 3. Previše kompleksni izrazi

Loše:

```go
if a && b || c && !d {

}
```

---

Bolje:

```go
allowed := (a && b) || (c && !d)

if allowed {

}
```

---

# Boolean memorija

U Go specifikaciji nije definisano da `bool` mora zauzimati tačno jedan byte u svim situacijama.

Implementacija zavisi od:

* arhitekture;
* poravnanja memorije;
* konteksta korišćenja.

---

Primer:

```go
type User struct {

	Active bool

}
```

---

Veličina strukture može zavisiti od:

* drugih polja;
* alignment pravila.

---

Detaljna analiza memorije biće obrađena u kasnijim poglavljima.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta predstavlja `bool` tip;
* koje vrednosti boolean može imati;
* šta je zero value za boolean;
* kako rade `&&`, `||`, `!`;
* šta je short-circuit evaluation;
* kako se boolean koristi u kontrolnom toku;
* kako pisati čitljive boolean izraze.

---

# Zaključak

Boolean tip je mali, ali izuzetno važan deo Go jezika.

Iako ima samo dve moguće vrednosti:

```text
true

false
```

on predstavlja osnovu za:

* donošenje odluka;
* validaciju;
* kontrolu toka programa;
* implementaciju poslovnih pravila.

U sledećem delu prelazimo na **Integer tipove u Go-u: vrste integer tipova, veličine, signed/unsigned vrednosti i rad sa celim brojevima**.

---

# Integer Types in Go

Integer tipovi predstavljaju cele brojeve.

To su vrednosti koje nemaju decimalni deo:

```text
-100

0

42

1000000
````

U programiranju se koriste za:

* brojače;
* identifikatore;
* indekse;
* veličine;
* godine;
* količine;
* statusne vrednosti.

---

U Go-u integer tipovi su deo **numeric data types** kategorije.

Struktura:

```text
Numeric Types

        |

        +-- Integer Types

        |       |

        |       +-- Signed Integers

        |       |

        |       +-- Unsigned Integers

        |

        +-- Floating Point

        |

        +-- Complex Numbers
```

---

# Signed vs Unsigned Integers

Go deli integer tipove na dve grupe:

## Signed integers

Mogu predstavljati:

* pozitivne brojeve;
* negativne brojeve;
* nulu.

Primer:

```text
-10

0

25
```

---

## Unsigned integers

Mogu predstavljati samo:

* pozitivne brojeve;
* nulu.

Primer:

```text
0

10

500
```

---

Negativne vrednosti nisu dozvoljene.

---

# Signed Integer Types

Go podržava sledeće signed integer tipove:

| Tip     | Veličina |
| ------- | -------- |
| `int8`  | 8 bit    |
| `int16` | 16 bit   |
| `int32` | 32 bit   |
| `int64` | 64 bit   |

---

Primer:

```go
package main

import "fmt"

func main() {

	var age int32 = 30

	fmt.Println(age)

}
```

---

# Integer veličina

Bit predstavlja jednu binarnu cifru:

```text
0 ili 1
```

---

Primer:

`int8`:

```text
8 bits
```

može predstaviti:

```text
256 različitih vrednosti
```

---

Kod signed tipova jedna vrednost se koristi za znak.

Zato:

```text
int8

range:

-128 do 127
```

---

# Integer Range

Signed integer opsezi:

| Tip     | Minimum              | Maximum             |
| ------- | -------------------- | ------------------- |
| `int8`  | -128                 | 127                 |
| `int16` | -32768               | 32767               |
| `int32` | -2147483648          | 2147483647          |
| `int64` | -9223372036854775808 | 9223372036854775807 |

---

Primer:

```go
var value int8 = 127
```

Dozvoljeno.

---

Ali:

```go
var value int8 = 128
```

greška:

```text
constant 128 overflows int8
```

---

# Unsigned Integer Types

Go podržava:

| Tip      | Veličina |
| -------- | -------- |
| `uint8`  | 8 bit    |
| `uint16` | 16 bit   |
| `uint32` | 32 bit   |
| `uint64` | 64 bit   |

---

Unsigned nema znak.

Primer:

```go
package main

import "fmt"

func main() {

	var count uint32 = 100

	fmt.Println(count)

}
```

---

# Unsigned Range

Primer:

`uint8`:

```text
0 - 255
```

---

Tabela:

| Tip      | Minimum | Maximum              |
| -------- | ------- | -------------------- |
| `uint8`  | 0       | 255                  |
| `uint16` | 0       | 65535                |
| `uint32` | 0       | 4294967295           |
| `uint64` | 0       | 18446744073709551615 |

---

# `int` Type

Pored konkretnih tipova:

```text
int8

int16

int32

int64
```

Go ima poseban tip:

```go
int
```

---

`int` zavisi od arhitekture sistema.

Najčešće:

| Arhitektura | Veličina |
| ----------- | -------- |
| 32-bit      | 32 bita  |
| 64-bit      | 64 bita  |

---

Primer:

```go
var age int = 30
```

---

Na modernom računaru:

```text
int

=

int64
```

---

# Zašto postoji `int`?

Zato što je najpraktičniji tip za:

* indekse;
* dužine;
* petlje;
* standardne operacije.

---

Primer:

```go
numbers := []int{1,2,3}

length := len(numbers)
```

---

Funkcija:

```go
len()
```

vraća:

```text
int
```

---

Zato je prirodno koristiti:

```go
int
```

za rad sa kolekcijama.

---

# `uint` Type

Analogno postoji:

```go
uint
```

---

Veličina zavisi od arhitekture:

```text
32-bit system

ili

64-bit system
```

---

Primer:

```go
var size uint = 100
```

---

# Specijalni Integer Tipovi

Go ima još nekoliko integer tipova:

```text
byte

rune
```

---

## byte

`byte` je alias za:

```go
uint8
```

---

Primer:

```go
var b byte = 65
```

---

Najčešće se koristi za:

* sirove podatke;
* bajtove;
* binarne fajlove.

---

## rune

`rune` je alias za:

```go
int32
```

---

Koristi se za Unicode karaktere.

Primer:

```go
var letter rune = 'A'
```

---

Detaljna analiza:

* byte;
* rune;
* Unicode;

biće obrađena kasnije.

---

# Integer Literals

Integer vrednosti mogu biti zapisane na više načina.

---

## Decimalni zapis

Najčešći:

```go
number := 100
```

---

## Binarni zapis

Prefix:

```text
0b
```

Primer:

```go
value := 0b1010
```

---

Vrednost:

```text
10
```

---

## Oktalni zapis

Prefix:

```text
0o
```

Primer:

```go
value := 0o12
```

---

## Heksadecimalni zapis

Prefix:

```text
0x
```

Primer:

```go
value := 0xff
```

---

Vrednost:

```text
255
```

---

# Integer Arithmetic

Go podržava standardne matematičke operacije.

---

## Sabiranje

```go
a := 10
b := 5

result := a + b
```

Rezultat:

```text
15
```

---

## Oduzimanje

```go
result := 10 - 5
```

Rezultat:

```text
5
```

---

## Množenje

```go
result := 10 * 5
```

Rezultat:

```text
50
```

---

## Deljenje

```go
result := 10 / 5
```

Rezultat:

```text
2
```

---

## Ostatak

Operator:

```go
%
```

Primer:

```go
result := 10 % 3
```

Rezultat:

```text
1
```

---

# Integer Division

Važna karakteristika:

Ako delimo dva integer-a:

```go
result := 7 / 2
```

dobijamo:

```text
3
```

a ne:

```text
3.5
```

---

Razlog:

Oba operanda su:

```text
integer
```

pa rezultat ostaje:

```text
integer
```

---

Primer:

```go
package main

import "fmt"

func main() {

	fmt.Println(7 / 2)

}
```

---

Rezultat:

```text
3
```

---

# Overflow

Integer ima ograničen opseg.

Primer:

```go
var value int8 = 127

value++
```

---

Rezultat:

Prelazi dozvoljeni opseg.

---

U runtime-u može doći do:

```text
overflow behavior
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var x uint8 = 255

	x++

	fmt.Println(x)

}
```

---

Kod unsigned tipova dolazi do:

```text
wrap around
```

---

Rezultat:

```text
0
```

---

# Integer Comparison

Integer vrednosti mogu se porediti:

```go
a := 10
b := 20

fmt.Println(a < b)
```

---

Rezultat:

```text
true
```

---

Operatori:

```text
==

!=

<

>

<=

>=
```

---

# Integer Conversion

Go ne radi implicitne konverzije.

Primer:

```go
var x int32 = 100

var y int64 = x
```

---

Greška.

---

Potrebna je eksplicitna konverzija:

```go
var y int64 = int64(x)
```

---

# Zašto nema implicitnih konverzija?

Primer:

```text
int32

+

int64
```

Koji rezultat očekujemo?

Go zahteva:

```text
programer odlučuje
```

---

To sprečava skrivene greške.

---

# Najčešće greške

---

## 1. Korišćenje premalog tipa

Loše:

```go
var users int8 = 500
```

---

Problem:

`int8` ne može čuvati 500.

---

## 2. Mešanje tipova

Loše:

```go
var a int32 = 10

var b int64 = 20

result := a + b
```

---

Potrebna konverzija.

---

## 3. Korišćenje unsigned bez razloga

Primer:

```go
var age uint = 30
```

---

Za većinu slučajeva:

```go
int
```

je bolji izbor.

---

# Kada koristiti koji integer tip?

Praktično pravilo:

| Situacija                         | Tip              |
| --------------------------------- | ---------------- |
| Opšti brojevi                     | `int`            |
| Memorijski optimizovane strukture | `int32`, `int64` |
| Bajtovi                           | `byte`           |
| Unicode karakteri                 | `rune`           |
| Velike vrednosti                  | `int64`          |
| Samo pozitivne vrednosti          | `uint`           |

---

# Integer i memorija

Veličina tipa direktno utiče na memoriju.

Primer:

```go
[]int64
```

troši više memorije nego:

```go
[]int32
```

---

Kod velikih sistema:

* milioni objekata;
* velike kolekcije;

izbor tipa može biti važan.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su integer tipovi;
* razliku između signed i unsigned vrednosti;
* kako rade `int`, `uint`, `int32`, `int64`;
* šta je integer range;
* šta je overflow;
* kako rade integer operacije;
* zašto Go zahteva eksplicitne konverzije.

---

# Zaključak

Integer tipovi su jedan od najčešće korišćenih delova Go jezika.

Najvažnije pravilo:

```text
Use int by default

Choose specific sizes only when needed
```

Razumevanje integer tipova je važno za:

* algoritme;
* kolekcije;
* memoriju;
* performanse.

U sledećem delu prelazimo na **floating point tipove (`float32`, `float64`), decimalne brojeve, preciznost i rad sa realnim vrednostima**.

---

# Floating Point Types in Go

Floating point tipovi predstavljaju brojeve koji mogu imati decimalni deo.

Primeri:

```text
3.14

19.99

-0.5

100.0
````

U matematici ih nazivamo realnim brojevima.

U programiranju se koriste za:

* finansijske izračune;
* merenja;
* statistiku;
* fiziku;
* grafiku;
* naučne proračune.

---

U Go jeziku floating point tipovi pripadaju grupi numeric types.

Struktura:

```text
Numeric Types

        |

        +-- Integer Types

        |

        +-- Floating Point Types

        |       |

        |       +-- float32

        |       |

        |       +-- float64

        |

        +-- Complex Types
```

---

# Floating Point Types

Go podržava dva floating point tipa:

| Tip       | Veličina |
| --------- | -------- |
| `float32` | 32 bita  |
| `float64` | 64 bita  |

---

Primer:

```go
package main

import "fmt"

func main() {

	var temperature float64 = 23.5

	fmt.Println(temperature)

}
```

Rezultat:

```text
23.5
```

---

# `float32`

`float32` koristi 32 bita memorije.

Karakteristike:

* manja memorijska potrošnja;
* manja preciznost;
* brži u određenim specijalizovanim slučajevima.

Primer:

```go
var value float32 = 3.14
```

---

Preciznost:

```text
oko 7 decimalnih cifara
```

---

# `float64`

`float64` koristi 64 bita memorije.

Karakteristike:

* veća preciznost;
* veći opseg vrednosti;
* standardni izbor u Go programima.

Primer:

```go
var price float64 = 99.99
```

---

Preciznost:

```text
oko 15 decimalnih cifara
```

---

# Zašto je `float64` podrazumevani izbor?

U Go-u decimalni literali imaju podrazumevani tip:

```text
float64
```

---

Primer:

```go
value := 3.14
```

Compiler zaključuje:

```text
value

=

float64
```

---

Isto kao:

```go
var value float64 = 3.14
```

---

# Kreiranje Floating Point promenljive

Postoji nekoliko načina.

---

## Explicit declaration

```go
var pi float64 = 3.14159
```

---

## Type inference

```go
pi := 3.14159
```

---

Compiler:

```text
pi → float64
```

---

## Zero value

```go
var price float64
```

---

Dobija:

```text
0
```

odnosno:

```text
0.0
```

---

# Floating Point Literals

Decimalni zapis:

```go
3.14
```

---

Sa eksponentom:

```go
1e3
```

znači:

```text
1000
```

---

Primer:

```go
distance := 1.5e3
```

Vrednost:

```text
1500
```

---

Negativni eksponent:

```go
1.5e-3
```

znači:

```text
0.0015
```

---

# Floating Point Arithmetic

Go podržava standardne matematičke operacije.

---

## Sabiranje

```go
a := 10.5
b := 5.2

result := a + b
```

Rezultat:

```text
15.7
```

---

## Oduzimanje

```go
result := 10.5 - 2.5
```

Rezultat:

```text
8.0
```

---

## Množenje

```go
result := 10.5 * 2
```

Rezultat:

```text
21.0
```

---

## Deljenje

```go
result := 10.0 / 4.0
```

Rezultat:

```text
2.5
```

---

# Integer i Floating Point razlika

Važna razlika:

Integer deljenje:

```go
7 / 2
```

rezultat:

```text
3
```

---

Floating point deljenje:

```go
7.0 / 2.0
```

rezultat:

```text
3.5
```

---

Primer:

```go
package main

import "fmt"

func main() {

	fmt.Println(7 / 2)

	fmt.Println(7.0 / 2.0)

}
```

Rezultat:

```text
3

3.5
```

---

# Mešanje Integer i Float vrednosti

Go ne dozvoljava implicitnu konverziju.

Primer:

```go
var x int = 10

var y float64 = 3.5

result := x + y
```

---

Greška:

```text
mismatched types int and float64
```

---

Potrebna je konverzija:

```go
result := float64(x) + y
```

---

# Floating Point Precision Problem

Jedna od najvažnijih karakteristika floating point brojeva:

> Nisu svi decimalni brojevi mogući za precizno predstavljanje u binarnom sistemu.

---

Primer:

```go
package main

import "fmt"

func main() {

	value := 0.1 + 0.2

	fmt.Println(value)

}
```

---

Možemo dobiti:

```text
0.30000000000000004
```

---

Zašto?

Zato što računari koriste binarni zapis:

```text
decimalni broj

        |

        v

binary representation
```

---

Neke decimalne vrednosti nemaju tačnu binarnu reprezentaciju.

---

# Floating Point Comparison

Zbog problema preciznosti ne treba direktno porediti floating point vrednosti.

Loše:

```go
if result == 0.3 {

}
```

---

Bolje:

```go
epsilon := 0.000001

if math.Abs(result-0.3) < epsilon {

}
```

---

Princip:

```text
difference < tolerance
```

---

# Primer poređenja

```go
package main

import (
	"fmt"
	"math"
)

func main() {

	a := 0.1 + 0.2

	expected := 0.3

	if math.Abs(a-expected) < 0.000001 {

		fmt.Println("Equal")

	}

}
```

---

Rezultat:

```text
Equal
```

---

# Infinity

Floating point može predstavljati beskonačnost.

Primer:

```go
package main

import (
	"fmt"
	"math"
)

func main() {

	fmt.Println(math.Inf(1))

}
```

---

Rezultat:

```text
+Inf
```

---

Negativna beskonačnost:

```go
math.Inf(-1)
```

---

# NaN (Not a Number)

Floating point može imati specijalnu vrednost:

```text
NaN
```

što znači:

```text
Not a Number
```

---

Primer:

```go
package main

import (
	"fmt"
	"math"
)

func main() {

	value := math.NaN()

	fmt.Println(value)

}
```

---

Rezultat:

```text
NaN
```

---

Provera:

```go
math.IsNaN(value)
```

---

# Floating Point i matematika

Go standardna biblioteka ima paket:

```go
math
```

---

Primer:

```go
import "math"
```

---

Funkcije:

```text
math.Sqrt()

math.Pow()

math.Sin()

math.Cos()

math.Round()
```

---

Primer:

```go
result := math.Sqrt(16)
```

---

Rezultat:

```text
4
```

---

# `float32` vs `float64`

Primer poređenja:

| Karakteristika | float32              | float64           |
| -------------- | -------------------- | ----------------- |
| Memorija       | 4 byte               | 8 byte            |
| Preciznost     | manja                | veća              |
| Performanse    | potencijalno brži    | standardni izbor  |
| Upotreba       | specijalni slučajevi | većina aplikacija |

---

# Kada koristiti `float32`?

Primeri:

* grafičke aplikacije;
* GPU računanja;
* velike numeričke matrice.

---

Primer:

```go
var vertex float32 = 1.5
```

---

# Kada koristiti `float64`?

Najčešće:

* backend aplikacije;
* API sistemi;
* statistika;
* merenja.

---

Primer:

```go
var salary float64 = 2500.50
```

---

# Floating Point i novac

Važno pravilo:

> Nemoj koristiti `float64` za precizne finansijske izračune.

---

Problem:

```go
price := 0.1 + 0.2
```

nije uvek:

```text
0.3
```

---

Za novac se često koristi:

* integer u najmanjoj jedinici;
* decimal biblioteke.

---

Primer:

Umesto:

```go
float64
```

koristiti:

```go
cents int64
```

---

Primer:

```go
price := 1999
```

znači:

```text
19.99 €
```

---

# Overflow kod Floating Point tipova

Floating point ima mnogo veći opseg nego integer.

Ali može doći do:

```text
+Inf
```

ili:

```text
-Inf
```

---

Primer:

```go
huge := math.MaxFloat64

result := huge * 2
```

---

Rezultat:

```text
+Inf
```

---

# Najčešće greške

---

## 1. Direktno poređenje decimalnih vrednosti

Loše:

```go
if price == 19.99 {

}
```

---

Koristiti toleranciju.

---

## 2. Korišćenje float za novac

Loše:

```go
total := 0.1 + 0.2
```

---

Bolje:

```go
totalCents := 10 + 20
```

---

## 3. Nepotreban float32

Loše:

```go
var value float32 = 100
```

ako nema razloga za manju preciznost.

---

Bolje:

```go
var value float64 = 100
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su floating point tipovi;
* razliku između `float32` i `float64`;
* zašto je `float64` standardni izbor;
* kako rade decimalni literali;
* problem floating point preciznosti;
* kako porediti floating point vrednosti;
* zašto se float ne koristi za novac.

---

# Zaključak

Floating point tipovi omogućavaju rad sa realnim brojevima, ali zahtevaju pažljivo korišćenje.

Najvažnije pravilo:

```text
Use float64 by default

Understand precision limitations

Avoid float for exact financial calculations
```

U sledećem delu prelazimo na **complex brojeve u Go-u (`complex64` i `complex128`), njihovu reprezentaciju i praktičnu upotrebu**.

---

# Complex Number Types in Go

Go je jedan od retkih modernih programskih jezika koji ima ugrađenu podršku za kompleksne brojeve.

Kompleksni brojevi se koriste u matematici, elektronici, fizici, signalnoj obradi i drugim oblastima gde obični realni brojevi nisu dovoljni.

---

Kompleksan broj ima oblik:

```text
a + bi
````

gde:

```text
a = realni deo

b = imaginarni deo

i = imaginarna jedinica
```

---

Primer:

```text
3 + 4i
```

gde:

```text
Realni deo:

3


Imaginarni deo:

4i
```

---

# Complex Types u Go-u

Go podržava dva kompleksna tipa:

| Tip          | Realni deo | Imaginarni deo |
| ------------ | ---------- | -------------- |
| `complex64`  | `float32`  | `float32`      |
| `complex128` | `float64`  | `float64`      |

---

Struktura:

```text
Complex Types

        |

        +-- complex64

        |       |

        |       +-- float32 real

        |       +-- float32 imaginary

        |

        +-- complex128

                |

                +-- float64 real

                +-- float64 imaginary
```

---

# Kreiranje Complex vrednosti

Najjednostavniji način:

```go
var c complex64 = 1 + 2i
```

---

Ovde:

```text
Real:

1


Imaginary:

2i
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var number complex64 = 3 + 4i

	fmt.Println(number)

}
```

Rezultat:

```text
(3+4i)
```

---

# Complex literals

Go koristi sufiks:

```text
i
```

za imaginarni deo.

---

Primeri:

```go
1i
```

znači:

```text
0 + 1i
```

---

```go
5i
```

znači:

```text
0 + 5i
```

---

```go
10 + 20i
```

znači:

```text
real = 10

imaginary = 20
```

---

# Type Inference kod Complex vrednosti

Ako koristimo:

```go
number := 1 + 2i
```

Go zaključuje:

```text
number

=

complex128
```

---

Primer:

```go
package main

import "fmt"

func main() {

	number := 1 + 2i

	fmt.Printf("%T\n", number)

}
```

---

Rezultat:

```text
complex128
```

---

# Explicit Complex Declaration

Možemo eksplicitno navesti tip:

```go
var number complex64 = 1 + 2i
```

---

ili:

```go
var number complex128 = 1 + 2i
```

---

# Zero Value za Complex

Kao i svi osnovni tipovi, complex ima zero value.

Zero value:

```text
0 + 0i
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var number complex128

	fmt.Println(number)

}
```

---

Rezultat:

```text
(0+0i)
```

---

# Kreiranje pomoću `complex()`

Go ima ugrađenu funkciju:

```go
complex()
```

koja kreira kompleksan broj.

---

Sintaksa:

```go
complex(real, imaginary)
```

---

Primer:

```go
number := complex(3, 4)
```

Rezultat:

```text
3 + 4i
```

---

Tip:

```text
complex128
```

---

Primer:

```go
package main

import "fmt"

func main() {

	number := complex(5, 10)

	fmt.Println(number)

}
```

---

Rezultat:

```text
(5+10i)
```

---

# Realni i imaginarni deo

Go omogućava izvlačenje delova kompleksnog broja.

Koriste se:

```go
real()
```

i:

```go
imag()
```

---

Primer:

```go
package main

import "fmt"

func main() {

	number := 3 + 4i

	fmt.Println(real(number))

	fmt.Println(imag(number))

}
```

---

Rezultat:

```text
3

4
```

---

# `real()` funkcija

Vraća realni deo.

Primer:

```go
number := complex(10, 20)

r := real(number)
```

---

Rezultat:

```text
10
```

---

# `imag()` funkcija

Vraća imaginarni deo.

Primer:

```go
number := complex(10, 20)

i := imag(number)
```

---

Rezultat:

```text
20
```

---

# Complex Arithmetic

Kompleksni brojevi podržavaju standardne matematičke operacije.

---

## Sabiranje

Primer:

```go
a := 1 + 2i

b := 3 + 4i

result := a + b
```

---

Matematički:

```text
(1 + 2i)

+

(3 + 4i)


=

4 + 6i
```

---

---

## Oduzimanje

```go
a := 5 + 6i

b := 2 + 3i

result := a - b
```

---

Rezultat:

```text
3 + 3i
```

---

# Množenje kompleksnih brojeva

Pravilo:

```text
(a + bi)(c + di)

=

(ac - bd) + (ad + bc)i
```

---

Primer:

```text
(1 + 2i)(3 + 4i)
```

---

Račun:

```text
3 + 4i + 6i + 8i²
```

Pošto:

```text
i² = -1
```

dobijamo:

```text
3 + 10i - 8
```

---

Rezultat:

```text
-5 + 10i
```

---

U Go-u:

```go
result := (1 + 2i) * (3 + 4i)
```

---

# Deljenje kompleksnih brojeva

Go podržava deljenje:

```go
result := a / b
```

---

Primer:

```go
package main

import "fmt"

func main() {

	a := 4 + 2i

	b := 1 + 1i

	fmt.Println(a / b)

}
```

---

Go automatski računa kompleksnu operaciju.

---

# Complex Comparison

Kompleksni brojevi podržavaju:

```text
==
!=
```

---

Primer:

```go
a := 1 + 2i

b := 1 + 2i

fmt.Println(a == b)
```

---

Rezultat:

```text
true
```

---

Ali ne postoji:

```go
<
>
<=
>=
```

---

Razlog:

Kompleksni brojevi nemaju prirodan redosled.

---

# Konverzija između Complex tipova

Kao i kod drugih numeričkih tipova:

Go zahteva eksplicitnu konverziju.

---

Primer:

```go
var a complex64 = 1 + 2i

var b complex128 = a
```

---

Nije dozvoljeno implicitno.

---

Potrebno:

```go
b := complex128(a)
```

---

# Complex i Floating Point

Kompleksni brojevi koriste floating point reprezentaciju.

Zato imaju iste probleme:

* ograničena preciznost;
* rounding greške;
* približne vrednosti.

---

Primer:

```text
complex128

=

float64 real

+

float64 imaginary
```

---

# Kada koristiti Complex Types?

U svakodnevnom backend razvoju:

```text
complex64

complex128
```

se retko koriste.

---

Najčešće oblasti:

## Signal Processing

Primeri:

* audio obrada;
* radio signali;
* Fourier transformacije.

---

## Elektrotehnika

Za:

* AC struju;
* impedansu;
* talase.

---

## Naučna računanja

Primeri:

* fizika;
* matematika;
* simulacije.

---

# Primer: Jednostavna operacija

```go
package main

import "fmt"

func main() {

	z1 := complex(2, 3)

	z2 := complex(1, 4)

	sum := z1 + z2

	fmt.Println(sum)

}
```

---

Rezultat:

```text
(3+7i)
```

---

# Complex Types i memorija

`complex64`:

```text
real float32

+

imaginary float32
```

---

Približno:

```text
8 bytes
```

---

`complex128`:

```text
real float64

+

imaginary float64
```

---

Približno:

```text
16 bytes
```

---

Tačna veličina zavisi od:

* arhitekture;
* alignment pravila;
* konteksta korišćenja.

---

# Najčešće greške

---

## 1. Mešanje complex i float tipova

Loše:

```go
var x complex128 = 10.5
```

---

Potrebno:

```go
var x complex128 = complex(10.5, 0)
```

---

## 2. Korišćenje realnog poređenja

Ne postoji:

```go
a < b
```

za complex vrednosti.

---

## 3. Zaboravljanje imaginarne jedinice

Pogrešno:

```go
3 + 4
```

je:

```text
integer expression
```

---

Ispravno:

```go
3 + 4i
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su kompleksni brojevi;
* razliku između `complex64` i `complex128`;
* kako se kreiraju complex vrednosti;
* kako rade `real()` i `imag()`;
* kako se vrše matematičke operacije;
* gde se koriste kompleksni brojevi.

---

# Zaključak

Kompleksni tipovi nisu svakodnevni deo većine Go aplikacija, ali predstavljaju važan deo Go numeric sistema.

Go direktno podržava:

```text
complex64

complex128
```

što omogućava elegantan rad sa matematičkim i naučnim problemima.

U sledećem delu prelazimo na **String tip u Go-u: reprezentacija teksta, Unicode, UTF-8 i osnovne karakteristike string vrednosti**.

---

# String Type in Go

String je jedan od najčešće korišćenih tipova podataka u svakom Go programu.

Koristi se za predstavljanje tekstualnih podataka:

```text
Hello

Go

User Name

Error Message

JSON Data
````

---

U Go jeziku string tip se označava kao:

```go
string
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var message string = "Hello Go"

	fmt.Println(message)

}
```

---

Rezultat:

```text
Hello Go
```

---

# Šta je String?

U Go-u string predstavlja:

> Neizmenjivu (immutable) sekvencu bajtova koja sadrži UTF-8 tekst.

Ovo je veoma važna definicija.

String u Go-u nije:

```text
array of characters
```

nego:

```text
sequence of bytes
```

---

Interna reprezentacija:

```text
String

+

Length

+

Pointer to byte data
```

---

Konceptualno:

```text
string

        |

        v

+----------------+
| H | e | l | l |
+----------------+

bytes
```

---

# Kreiranje String vrednosti

Postoji nekoliko načina.

---

## Explicit declaration

```go
var name string = "Marko"
```

---

## Type inference

```go
name := "Marko"
```

Compiler zaključuje:

```text
name

=

string
```

---

## Zero Value

Ako string nije inicijalizovan:

```go
var name string
```

vrednost je:

```text
""
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var name string

	fmt.Println(name)

}
```

---

Rezultat:

```text
(empty string)
```

---

# String Literals

Go podržava dva osnovna oblika string literala:

1. Interpreted strings
2. Raw strings

---

# Interpreted String Literals

Najčešći oblik:

```go
"Hello World"
```

---

Ovi stringovi podržavaju escape sekvence.

Primer:

```go
message := "Hello\nWorld"
```

---

Rezultat:

```text
Hello
World
```

---

# Escape Sequences

Go podržava standardne escape karaktere.

| Sekvenca | Značenje        |
| -------- | --------------- |
| `\n`     | novi red        |
| `\t`     | tab             |
| `\\`     | backslash       |
| `\"`     | navodnik        |
| `\r`     | carriage return |

---

Primer:

```go
package main

import "fmt"

func main() {

	fmt.Println("Hello\tGo")

}
```

---

Rezultat:

```text
Hello    Go
```

---

# Raw String Literals

Raw string koristi backtick:

```go
`Hello World`
```

---

Karakteristike:

* nema escape sekvenci;
* čuva originalni sadržaj;
* može sadržati nove redove.

---

Primer:

```go
message := `
Hello
Go
`
```

---

Rezultat:

```text
Hello
Go
```

---

# Raw String praktična upotreba

Najčešće se koristi za:

* SQL upite;
* JSON;
* HTML;
* template sadržaj.

---

Primer:

```go
query := `
SELECT *
FROM users
WHERE id = 10
`
```

---

Prednost:

Nema potrebe za:

```go
\n
```

i:

```go
\"
```

---

# String Length

Za dužinu stringa koristi se:

```go
len()
```

---

Primer:

```go
package main

import "fmt"

func main() {

	name := "Go"

	fmt.Println(len(name))

}
```

---

Rezultat:

```text
2
```

---

Ali postoji važna stvar:

`len()` vraća broj bajtova, a ne broj karaktera.

---

Primer:

```go
name := "Go"
```

Memorija:

```text
G

o
```

Broj bajtova:

```text
2
```

---

# UTF-8 i Stringovi

Go koristi UTF-8 kao standardnu reprezentaciju teksta.

Primer:

```go
text := "こんにちは"
```

---

Vizuelno:

```text
こ ん に ち は
```

ima:

```text
5 karaktera
```

Ali:

```go
len(text)
```

ne vraća:

```text
5
```

---

Primer:

```go
package main

import "fmt"

func main() {

	text := "世界"

	fmt.Println(len(text))

}
```

---

Rezultat:

```text
6
```

---

Zašto?

Zato što svaki Unicode karakter može zauzimati više bajtova.

---

# Byte vs Character

U Go-u:

```text
byte

!=

character
```

---

Primer:

ASCII:

```text
A
```

zauzima:

```text
1 byte
```

---

Unicode:

```text
Ž
```

može zauzimati:

```text
2 bytes
```

---

Zbog toga:

```go
len()
```

radi nad bajtovima.

---

# String Indexing

String može biti indeksiran.

Primer:

```go
package main

import "fmt"

func main() {

	name := "Go"

	fmt.Println(name[0])

}
```

---

Rezultat:

```text
71
```

---

Zašto nije:

```text
G
```

?

Zato što:

```go
name[0]
```

vraća:

```text
byte
```

---

ASCII vrednost:

```text
G = 71
```

---

Ako želimo karakter:

```go
fmt.Printf("%c\n", name[0])
```

---

Rezultat:

```text
G
```

---

# String je Immutable

Jedna od najvažnijih karakteristika:

> String vrednosti u Go-u ne mogu biti menjane nakon kreiranja.

---

Primer:

```go
name := "Go"

name[0] = 'N'
```

---

Compiler greška:

```text
cannot assign to name[0]
```

---

Zašto?

Zato što:

```text
string

=

immutable data
```

---

# Kako promeniti String?

Kreira se novi string.

Primer:

```go
name := "Go"

newName := "No"
```

---

Nije promenjen original:

```text
Go
```

nego je napravljen novi:

```text
No
```

---

# String Concatenation

Stringovi se mogu spajati operatorom:

```go
+
```

---

Primer:

```go
first := "Hello"

second := "Go"

result := first + " " + second
```

---

Rezultat:

```text
Hello Go
```

---

Primer:

```go
package main

import "fmt"

func main() {

	name := "Go"

	message := "Hello " + name

	fmt.Println(message)

}
```

---

Rezultat:

```text
Hello Go
```

---

# String Comparison

Stringovi podržavaju poređenje.

Operatori:

```text
==

!=

<

>

<=

>=
```

---

Primer:

```go
a := "Go"

b := "Go"

fmt.Println(a == b)
```

---

Rezultat:

```text
true
```

---

# Lexicographical Comparison

Poređenje stringova koristi Unicode vrednosti.

Primer:

```go
"a" < "b"
```

je:

```text
true
```

---

Zato što:

```text
Unicode(a)

<

Unicode(b)
```

---

Primer:

```go
fmt.Println("apple" < "banana")
```

---

Rezultat:

```text
true
```

---

# String i Memorija

String u Go-u se ponaša kao descriptor.

Konceptualno:

```text
String Header

+-------------+
| Data Ptr    |
+-------------+
| Length      |
+-------------+
```

---

String ne čuva direktno sadržaj u promenljivoj.

On sadrži:

* pokazivač na podatke;
* dužinu.

---

Ovo je važno za:

* performance;
* memory sharing;
* garbage collector.

---

# String Copy Behavior

Kada uradimo:

```go
a := "Hello"

b := a
```

ne kopira se nužno kompletan sadržaj.

---

Konceptualno:

```text
a

 |

 +-----> Hello


b

 |
 
 +-----> Hello
```

---

Stringovi su immutable, pa je deljenje bezbedno.

---

# Najčešće greške

---

## 1. Pretpostavka da len() vraća broj karaktera

Pogrešno:

```go
len("世界") == 2
```

---

Tačno:

```text
len()

=

broj bajtova
```

---

## 2. Pokušaj izmene stringa

Pogrešno:

```go
text[0] = 'A'
```

---

String nije mutable.

---

## 3. Mešanje byte i rune koncepta

Pogrešno:

```text
byte = character
```

---

U Go-u:

```text
byte

=

uint8


rune

=

int32
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je string u Go-u;
* kako se string predstavlja interno;
* razliku između byte i character;
* UTF-8 reprezentaciju;
* string literale;
* escape sekvence;
* raw string;
* immutable prirodu stringova;
* osnovne string operacije.

---

# Zaključak

String je jedan od najvažnijih tipova u Go programiranju.

Najvažnije činjenice:

```text
String

=

immutable sequence of bytes

+

UTF-8 support
```

Razumevanje stringova je osnova za kasniji rad sa:

* `[]byte`;
* `rune`;
* Unicode;
* parsiranjem teksta;
* JSON podacima;
* HTTP komunikacijom.

U sledećem delu detaljnije ćemo obraditi **rad sa stringovima: konverzije, rune, byte slice i Unicode karaktere**.

---

# Strings, Bytes, Runes and Unicode in Go

U prethodnom delu smo definisali string kao:

> Immutable sequence of bytes encoded using UTF-8.

Ova definicija je jedna od najvažnijih stvari za razumevanje Go stringova.

Mnogi početnici dolaze iz jezika gde je string:

```text
array of characters
````

Međutim, u Go-u string nije kolekcija karaktera.

Go string je:

```text
string

    |

    v

sequence of bytes
```

---

Zbog toga moramo razumeti tri ključna koncepta:

```text
String

Byte

Rune
```

---

# Byte u Go-u

`byte` je specijalni naziv za:

```go
uint8
```

odnosno:

```text
8 bits

=

1 byte
```

---

Primer:

```go id="h5z7bv"
var b byte = 'A'
```

---

Ista vrednost može biti napisana kao broj:

```go id="8vkm2m"
var b byte = 65
```

---

Zašto?

Zato što karakter:

```text
A
```

u ASCII tabeli ima vrednost:

```text
65
```

---

Primer:

```go id="w5m9qs"
package main

import "fmt"

func main() {

	var letter byte = 'A'

	fmt.Println(letter)

}
```

---

Rezultat:

```text
65
```

---

Ako želimo karakter:

```go id="w8f3gx"
fmt.Printf("%c\n", letter)
```

---

Rezultat:

```text
A
```

---

# Byte i String

Pošto je string sekvenca bajtova, možemo pristupiti pojedinačnom bajtu.

Primer:

```go id="q2y9nv"
package main

import "fmt"

func main() {

	text := "Go"

	fmt.Println(text[0])

}
```

---

Rezultat:

```text
71
```

---

Zašto?

Zato što:

```text
G

=

ASCII 71
```

---

String:

```text
Go
```

u memoriji:

```text
+----+----+
| 71 |111 |
+----+----+

 G    o
```

---

# Iteracija preko stringa

Postoje dva glavna načina.

---

## Iteracija preko byte vrednosti

Koristimo klasičnu for petlju:

```go id="wq8h1f"
package main

import "fmt"

func main() {

	text := "Go"

	for i := 0; i < len(text); i++ {

		fmt.Println(text[i])

	}

}
```

---

Rezultat:

```text
71

111
```

---

Ovde iteriramo kroz:

```text
bytes
```

ne karaktere.

---

# Problem sa Unicode karakterima

ASCII karakteri zauzimaju:

```text
1 byte
```

Ali Unicode karakteri mogu zauzimati:

```text
1-4 bytes
```

---

Primer:

```go id="n8c4mq"
text := "Ž"
```

---

Vizuelno:

```text
Ž
```

je:

```text
1 character
```

---

Ali u UTF-8:

```text
2 bytes
```

---

Primer:

```go id="x3m8pq"
package main

import "fmt"

func main() {

	text := "Ž"

	fmt.Println(len(text))

}
```

---

Rezultat:

```text
2
```

---

Dakle:

```text
len()

=

broj bytes

```

---

# Rune u Go-u

Rune predstavlja Unicode code point.

Definicija:

```go
type rune = int32
```

---

Rune može predstavljati:

* ASCII karakter;
* Unicode karakter;
* simbol;
* emoji.

---

Primer:

```go id="m7q4vx"
var r rune = 'Ž'
```

---

Interno:

```text
r

=

Unicode code point
```

---

# Rune i karakteri

Kada govorimo o "karakteru" u Go-u, najčešće mislimo na:

```text
rune
```

---

Primer:

```go id="v4n8mq"
package main

import "fmt"

func main() {

	var r rune = 'A'

	fmt.Println(r)

}
```

---

Rezultat:

```text
65
```

---

Unicode vrednost karaktera:

```text
A

=

U+0041
```

---

# Rune literal

Rune se piše pomoću jednostrukih navodnika:

```go
'A'
```

---

Primeri:

```go
'a'

'Ž'

'世'

'😀'
```

---

Razlika:

String:

```go
"Go"
```

Rune:

```go
'G'
```

---

String:

```text
više znakova
```

Rune:

```text
jedan Unicode code point
```

---

# Konverzija String → Rune Slice

Ako želimo da radimo sa karakterima:

koristimo:

```go
[]rune
```

---

Primer:

```go id="q8m2wx"
package main

import "fmt"

func main() {

	text := "Ž"

	runes := []rune(text)

	fmt.Println(len(runes))

}
```

---

Rezultat:

```text
1
```

---

Zašto?

Jer:

```text
Ž

=

jedan Unicode karakter
```

---

Dok:

```go
[]byte(text)
```

daje:

```text
2
```

---

# String → Byte Slice

String možemo pretvoriti u:

```go
[]byte
```

---

Primer:

```go id="u3x7mq"
package main

import "fmt"

func main() {

	text := "Go"

	bytes := []byte(text)

	fmt.Println(bytes)

}
```

---

Rezultat:

```text
[71 111]
```

---

Dobijamo UTF-8 bajtove.

---

# Byte Slice → String

Obrnuto:

```go id="j5m8qx"
bytes := []byte{71,111}

text := string(bytes)
```

---

Rezultat:

```text
Go
```

---

Primer:

```go id="x9q3mv"
package main

import "fmt"

func main() {

	data := []byte{72,101,108,108,111}

	fmt.Println(string(data))

}
```

---

Rezultat:

```text
Hello
```

---

# Range preko String-a

Najčešći način za rad sa karakterima je:

```go
for range
```

---

Primer:

```go id="m4x8qw"
package main

import "fmt"

func main() {

	text := "Go"

	for index, value := range text {

		fmt.Println(index, value)

	}

}
```

---

Rezultat:

```text
0 71

1 111
```

---

Ali kod Unicode:

```go id="z8m4qx"
text := "Ž"
```

---

Rezultat:

```text
0 381
```

---

Ne:

```text
0 197
1 189
```

---

Zašto?

Zato što:

```text
range

=

iteracija kroz runes
```

---

# Byte Iteracija vs Rune Iteracija

Poređenje:

```text
String:

"Ž"
```

---

Byte pristup:

```text
bytes:

[197 189]
```

---

Rune pristup:

```text
runes:

[381]
```

---

Tabela:

| Način          | Radi sa            |
| -------------- | ------------------ |
| `text[i]`      | byte               |
| `range text`   | rune               |
| `[]byte(text)` | bytes              |
| `[]rune(text)` | Unicode characters |

---

# Kada koristiti byte?

Koristi byte kada radiš sa:

* fajlovima;
* mrežom;
* binarnim podacima;
* protokolima.

Primer:

```go
data := []byte(message)
```

---

# Kada koristiti rune?

Koristi rune kada radiš sa:

* tekstom;
* korisničkim unosom;
* Unicode karakterima;
* brojanjem karaktera.

Primer:

```go
characters := []rune(text)
```

---

# Brojanje karaktera

Pogrešno:

```go
len(text)
```

---

Za:

```go
text := "世界"
```

dobijamo:

```text
6
```

---

Ispravno:

```go
len([]rune(text))
```

---

Rezultat:

```text
2
```

---

# Primer: Obrtanje stringa

Kod ASCII:

```go
"hello"
```

možemo raditi:

```text
olleh
```

---

Ali Unicode zahteva rune.

Primer:

```go
func reverse(text string) string {

	runes := []rune(text)

	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {

		runes[i], runes[j] = runes[j], runes[i]

	}

	return string(runes)

}
```

---

Zašto?

Jer byte reverse može pokvariti UTF-8 sekvencu.

---

# Unicode i UTF-8

Go standardno koristi:

```text
UTF-8
```

---

UTF-8:

* kompatibilan sa ASCII;
* promenljive dužine;
* podržava ceo Unicode standard.

---

Primer:

ASCII:

```text
A

1 byte
```

---

Emoji:

```text
😀

4 bytes
```

---

# Najčešće greške

---

## 1. Pretpostavljanje da string ima karaktere

Pogrešno:

```text
string

=

[]rune
```

---

Tačno:

```text
string

=

[]byte
```

---

## 2. Korišćenje len() za broj karaktera

Pogrešno:

```go
len("Ž")
```

---

Dobijamo:

```text
2
```

---

Ako želimo karaktere:

```go
len([]rune("Ž"))
```

---

## 3. Obrada Unicode teksta kao byte

Može pokvariti:

* slova;
* simbole;
* emoji.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je byte;
* šta je rune;
* razliku između byte i character;
* kako Go obrađuje UTF-8;
* razliku između `[]byte` i `[]rune`;
* kako koristiti `range` nad stringovima;
* kada koristiti byte, a kada rune.

---

# Zaključak

Najvažnija stvar koju treba zapamtiti:

```text
String u Go-u nije kolekcija karaktera.

String je sekvenca UTF-8 bajtova.
```

Kada radimo sa:

```text
binary data

↓

byte
```

Kada radimo sa:

```text
human-readable text

↓

rune
```

Razumevanje ove razlike sprečava veliki broj bugova u radu sa tekstom.

U sledećem delu prelazimo na **konverzije između osnovnih tipova, type casting i pravila eksplicitnih konverzija u Go-u**.

---

# Type Conversion in Go

U prethodnim delovima smo obradili osnovne primitive tipove:

- integer tipove;
- floating point tipove;
- complex tipove;
- string tip;
- byte i rune.

Sada prelazimo na jednu od najvažnijih karakteristika Go jezika:

> Go zahteva eksplicitne konverzije između različitih tipova.

---

# Static Typing u Go-u

Go je statički tipiziran jezik.

To znači da compiler u toku prevođenja proverava tipove podataka.

Primer:

```go
package main

import "fmt"

func main() {

	var age int = 25

	fmt.Println(age)

}
````

Compiler zna:

```text
age

=

int
```

---

Ako pokušamo da spojimo različite tipove:

```go
package main

func main() {

	var age int = 25

	var price float64 = 19.99

	result := age + price

}
```

---

Dobijamo grešku:

```text
invalid operation:
mismatched types int and float64
```

---

Zašto?

Zato što Go ne radi implicitnu konverziju.

---

# Implicit Conversion vs Explicit Conversion

Postoje dva pristupa.

---

## Implicit Conversion

Jezik automatski konvertuje tip.

Primer iz nekih jezika:

```text
int

+

double

=

double
```

---

Go ovo NE radi.

---

## Explicit Conversion

Programer mora jasno napisati konverziju.

Primer:

```go
result := float64(age) + price
```

---

Ovo govori compileru:

```text
convert int

to

float64
```

---

# Sintaksa Type Conversion

Osnovna sintaksa:

```go
Type(value)
```

---

Primer:

```go
var number int = 100

var converted float64 = float64(number)
```

---

Struktura:

```text
original value

        |

        v

Type()

        |

        v

new value
```

---

# Integer Conversion

Najčešća konverzija:

```text
int

<->

other integer types
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var number int = 100

	var small int32 = int32(number)

	fmt.Println(small)

}
```

---

Rezultat:

```text
100
```

---

# Različiti Integer Tipovi

Go razlikuje:

```text
int8

int16

int32

int64
```

---

I:

```text
uint8

uint16

uint32

uint64
```

---

Iako svi predstavljaju cele brojeve:

```go
var a int32 = 100

var b int64 = 200
```

ne mogu direktno:

```go
a + b
```

---

Potrebno:

```go
int64(a) + b
```

---

# Konverzija int u float

Primer:

```go
package main

import "fmt"

func main() {

	var count int = 10

	var average float64 = float64(count)

	fmt.Println(average)

}
```

---

Rezultat:

```text
10
```

---

Sada:

```text
int

↓

float64
```

---

# Gubitak preciznosti kod konverzije

Konverzija nije uvek bez posledica.

Primer:

```go
var value float64 = 3.99

var integer int = int(value)
```

---

Rezultat:

```text
3
```

---

Decimalni deo se uklanja.

---

Važno:

```text
3.99

nije

4
```

---

Go ne vrši zaokruživanje.

---

Ako želimo zaokruživanje:

```go
math.Round()
```

---

Primer:

```go
rounded := math.Round(3.99)
```

Rezultat:

```text
4
```

---

# Integer Overflow kod Konverzije

Konverzija može izgubiti podatke.

Primer:

```go
var big int64 = 1000

var small int8 = int8(big)
```

---

Ako vrednost ne može stati:

```text
data loss
```

---

Primer:

```go
var number int16 = 300

var small int8 = int8(number)
```

---

`int8` opseg:

```text
-128 do 127
```

---

Rezultat može biti:

```text
overflow
```

---

# Signed i Unsigned Konverzija

Posebno oprezno:

```go
int

↓

uint
```

---

Primer:

```go
var value int = -10

var unsigned uint = uint(value)
```

---

Negativan broj postaje velika pozitivna vrednost.

---

Razlog:

```text
binary representation
```

---

# String Konverzije

String konverzije su specifične.

---

## Integer → String

Mnogi početnici očekuju:

```go
string(65)
```

da vrati:

```text
"65"
```

---

Ali rezultat je:

```text
"A"
```

---

Zašto?

Jer:

```go
string(int)
```

pretvara broj u Unicode karakter.

---

Primer:

```go
package main

import "fmt"

func main() {

	value := string(65)

	fmt.Println(value)

}
```

---

Rezultat:

```text
A
```

---

# Integer → String pomoću strconv

Za pravi tekstualni broj koristi se:

```go
strconv.Itoa()
```

---

Primer:

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {

	number := 123

	text := strconv.Itoa(number)

	fmt.Println(text)

}
```

---

Rezultat:

```text
123
```

---

Tip:

```text
string
```

---

# String → Integer

Za konverziju:

```text
string

↓

int
```

koristi se:

```go
strconv.Atoi()
```

---

Primer:

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {

	text := "123"

	number, err := strconv.Atoi(text)

	fmt.Println(number)
	fmt.Println(err)

}
```

---

Rezultat:

```text
123

<nil>
```

---

# String → Float

Koristi se:

```go
strconv.ParseFloat()
```

---

Primer:

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {

	text := "3.14"

	value, err := strconv.ParseFloat(text, 64)

	fmt.Println(value)
	fmt.Println(err)

}
```

---

Rezultat:

```text
3.14

<nil>
```

---

# Float → String

Koristi se:

```go
strconv.FormatFloat()
```

---

Primer:

```go
text := strconv.FormatFloat(
	3.14,
	'f',
	2,
	64,
)
```

---

Rezultat:

```text
"3.14"
```

---

# Byte Conversion

Pošto je:

```go
byte = uint8
```

možemo koristiti konverziju.

---

Primer:

```go
value := byte(65)

fmt.Println(value)
```

---

Rezultat:

```text
65
```

---

U karakter:

```go
fmt.Printf("%c", value)
```

---

Rezultat:

```text
A
```

---

# Rune Conversion

Rune je:

```go
int32
```

---

Primer:

```go
var r rune = 'Ž'

fmt.Println(r)
```

---

Rezultat:

```text
381
```

---

Obrnuto:

```go
character := rune(381)
```

---

Rezultat:

```text
Ž
```

---

# Boolean Conversion

Go ne podržava:

```text
bool

↓

int
```

---

Nije moguće:

```go
number := int(true)
```

---

Potrebna je ručna logika:

```go
var number int

if value {

	number = 1

}
```

---

# Named Types i Conversion

Go dozvoljava kreiranje sopstvenih tipova.

Primer:

```go
type UserID int
```

---

Sada:

```go
var id UserID = 100
```

nije isto što:

```go
var number int = 100
```

---

Potrebna je konverzija:

```go
number := int(id)
```

---

# Type Conversion vs Type Assertion

Važna razlika:

## Conversion

Menja vrednost jednog konkretnog tipa:

```go
float64(number)
```

---

## Assertion

Radi sa interface vrednostima:

```go
value.(string)
```

---

Ovo ćemo detaljno obraditi kasnije kod:

* interfaces;
* custom types;
* polymorphism.

---

# Najčešće greške

---

## 1. Očekivanje implicitne konverzije

Pogrešno:

```go
var x int = 10

var y int64 = x
```

---

Ispravno:

```go
y := int64(x)
```

---

## 2. Korišćenje string() za brojeve

Pogrešno:

```go
string(123)
```

---

Dobija se:

```text
{
Unicode character
}
```

---

Koristiti:

```go
strconv.Itoa()
```

---

## 3. Ignorisanje overflow problema

Pogrešno:

```go
int8(1000)
```

---

Može doći do:

```text
data corruption
```

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* zašto Go koristi eksplicitne konverzije;
* sintaksu `Type(value)`;
* konverziju između numeričkih tipova;
* gubitak preciznosti;
* overflow probleme;
* string konverzije;
* upotrebu `strconv` paketa;
* razliku između conversion i assertion.

---

# Zaključak

Go favorizuje eksplicitnost.

Umesto:

```text
compiler guesses
```

Go zahteva:

```text
developer decision
```

Zato su konverzije jasne i lako uočljive u kodu.

Najvažnije pravilo:

```text
Different types

↓

Explicit conversion required
```

U sledećem delu prelazimo na **Constants u Go-u: deklaracija, typed i untyped konstante, i compile-time ponašanje**.

---

# Constants in Go

Konstante predstavljaju vrednosti koje se ne mogu menjati nakon inicijalizacije.

U Go jeziku konstante su deo jezika na veoma dubokom nivou i imaju drugačije ponašanje u odnosu na promenljive.

---

Osnovna ideja:

```text
Variable

=
value that can change


Constant

=
value that cannot change
````

---

Primer:

```go
package main

import "fmt"

func main() {

	const pi = 3.14159

	fmt.Println(pi)

}
```

---

Rezultat:

```text
3.14159
```

---

Pokušaj izmene:

```go
const pi = 3.14159

pi = 3.14
```

---

Compiler greška:

```text
cannot assign to pi
```

---

# Zašto postoje konstante?

Konstante koristimo kada vrednost predstavlja koncept koji:

* se nikada ne menja;
* poznat je u trenutku kompajliranja;
* treba zaštititi od slučajne izmene.

---

Primeri:

Matematičke vrednosti:

```go
const Pi = 3.141592653589793
```

---

Konfiguracione vrednosti:

```go
const MaxConnections = 100
```

---

Statusne vrednosti:

```go
const StatusOK = 200
```

---

Vremenske jedinice:

```go
const Timeout = 30
```

---

# Deklaracija konstanti

Sintaksa:

```go
const name type = value
```

---

Primer:

```go
const age int = 30
```

---

Međutim, tip nije obavezan.

---

Možemo napisati:

```go
const age = 30
```

---

Go tada određuje tip.

---

# Typed Constants

Typed constant ima eksplicitno definisan tip.

Primer:

```go
const age int = 25
```

---

Ovde:

```text
age

=

int
```

---

Drugi primer:

```go
const price float64 = 99.99
```

---

Tip:

```text
float64
```

---

# Untyped Constants

Go podržava koncept:

```text
untyped constants
```

---

Primer:

```go
const number = 100
```

---

Ova konstanta nema eksplicitni tip.

---

Compiler čuva:

```text
constant value
```

umesto:

```text
constant type
```

---

Kasnije može biti konvertovana u odgovarajući tip.

---

Primer:

```go
const number = 100

var x int = number

var y float64 = number
```

---

Obe operacije su dozvoljene.

---

Zašto?

Zato što je:

```text
number

=

untyped constant
```

---

# Typed vs Untyped Constants

Poređenje:

| Karakteristika        | Typed                    | Untyped       |
| --------------------- | ------------------------ | ------------- |
| Ima tip               | Da                       | Ne            |
| Fleksibilnost         | Manja                    | Veća          |
| Konverzija            | Zahteva kompatibilan tip | Compiler bira |
| Compile-time vrednost | Da                       | Da            |

---

Primer:

## Typed

```go
const x int = 10

var y int64 = x
```

---

Greška:

```text
cannot use x as int64
```

---

Potrebno:

```go
var y int64 = int64(x)
```

---

## Untyped

```go
const x = 10

var y int64 = x
```

---

Radi:

```text
OK
```

---

# Constants su Compile-Time vrednosti

Jedna od najvažnijih karakteristika:

> Konstante se evaluiraju tokom kompajliranja.

---

Primer:

```go
const secondsInMinute = 60
```

Compiler zna:

```text
secondsInMinute = 60
```

pre pokretanja programa.

---

Za razliku od:

```go
var seconds = 60
```

koji postoji kao runtime vrednost.

---

# Runtime Variable vs Compile-Time Constant

Poređenje:

```text
Constant

Compile Time

        |

        v

Binary


Variable

Runtime

        |

        v

Memory
```

---

# Zero Value i Constants

Za razliku od promenljivih:

```go
var value int
```

koje dobijaju zero value,

konstante moraju imati vrednost.

---

Pogrešno:

```go
const value int
```

---

Compiler:

```text
missing init expression
```

---

Ispravno:

```go
const value int = 0
```

---

# Multiple Constant Declaration

Možemo deklarisati više konstanti:

```go
const (
	Pi = 3.14159
	E  = 2.71828
)
```

---

Primer:

```go
package main

import "fmt"

const (
	MinAge = 18
	MaxAge = 65
)

func main() {

	fmt.Println(MinAge)
	fmt.Println(MaxAge)

}
```

---

Rezultat:

```text
18
65
```

---

# Grupisanje konstanti

Najčešće se koristi za povezane vrednosti.

Primer:

```go
const (
	StatusPending = 1
	StatusActive  = 2
	StatusClosed  = 3
)
```

---

Bolja organizacija:

```text
Status

|
+-- Pending
+-- Active
+-- Closed
```

---

# Naming Convention za Constants

Go koristi:

```text
PascalCase
```

za javne konstante:

```go
const MaxUsers = 100
```

---

Privatne konstante:

```go
const maxUsers = 100
```

---

Pravila:

Exported:

```go
MaxUsers
```

---

Unexported:

```go
maxUsers
```

---

# Constants i velikim slovima

Često se može videti:

```go
const MAX_USERS = 100
```

---

Ali Go stil preporučuje:

```go
const MaxUsers = 100
```

---

Razlog:

Go koristi camelCase/PascalCase stil.

---

# Constant Expressions

Go konstante mogu biti rezultat izraza.

Primer:

```go
const (
	SecondsPerMinute = 60
	SecondsPerHour   = SecondsPerMinute * 60
)
```

---

Compiler računa:

```text
SecondsPerHour

=

3600
```

---

Dozvoljeno:

```go
const x = 10 + 20
```

---

Rezultat:

```text
30
```

---

# Šta može biti constant value?

Dozvoljeno:

* numeričke vrednosti;
* stringovi;
* boolean vrednosti;
* rune vrednosti.

---

Primer:

```go
const Name = "Go"

const Enabled = true

const Letter = 'A'
```

---

# Nije moguće napraviti constant slice

Pogrešno:

```go
const numbers = []int{1,2,3}
```

---

Greška:

```text
[]int is not constant
```

---

Zašto?

Zato što slice predstavlja runtime strukturu.

---

# Nije moguće napraviti constant map

Pogrešno:

```go
const users = map[string]int{}
```

---

Razlog:

Map je runtime objekat.

---

# Nije moguće napraviti constant struct

Pogrešno:

```go
const user = User{}
```

---

Struct vrednosti nisu constant expressions.

---

# Constants i Memory

Važno:

Konstante uglavnom ne zauzimaju runtime memoriju kao promenljive.

---

Primer:

```go
const Limit = 100
```

Compiler može direktno zameniti:

```go
Limit
```

sa:

```go
100
```

---

Proces:

```text
Source Code

Limit

        |

        v

Compiler

        |

        v

100
```

---

# Constant Overflow

Go dozvoljava velike constant vrednosti dok god se koriste u odgovarajućem kontekstu.

---

Primer:

```go
const Huge = 1000000000000000000000
```

---

Ovo može biti validno.

---

Ali:

```go
var x int = Huge
```

može izazvati:

```text
overflow
```

---

Zašto?

Zato što konstanta još nema konkretan runtime tip.

---

# Primer: Constants i različiti tipovi

```go
package main

import "fmt"

const Value = 100

func main() {

	var a int = Value

	var b float64 = Value

	fmt.Println(a)
	fmt.Println(b)

}
```

---

Rezultat:

```text
100
100
```

---

# Najčešće greške

---

## 1. Pokušaj izmene konstante

Pogrešno:

```go
const Age = 20

Age = 21
```

---

## 2. Očekivanje runtime ponašanja

Pogrešno:

```go
const Time = time.Now()
```

---

Zašto?

Jer funkcije rade u runtime-u.

---

## 3. Korišćenje konstanti za promenljive vrednosti

Pogrešno:

```go
const CurrentUser = "Marko"
```

ako se korisnik može menjati.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta su konstante;
* razliku između variable i constant;
* typed i untyped constants;
* compile-time evaluaciju;
* constant expressions;
* ograničenja constant vrednosti;
* organizaciju konstanti.

---

# Zaključak

Konstante u Go-u nisu samo "promenljive koje se ne menjaju".

One su compile-time vrednosti koje compiler može optimizovati i koristiti bez runtime memorijske reprezentacije.

Najvažnije pravilo:

```text
Use constants for values
that are known and immutable.
```

U sledećem delu prelazimo na **Pointer tip u Go-u: memorijske adrese, dereferenciranje, pokazivače i osnovni model rada sa memorijom**.

---

# Pointers in Go

Pokazivači (`pointers`) predstavljaju jedan od najvažnijih koncepata za razumevanje načina na koji Go upravlja memorijom.

Za razliku od jezika koji potpuno skrivaju rad sa memorijom, Go omogućava direktan rad sa adresama memorijskih lokacija, ali na kontrolisan i bezbedniji način.

---

Osnovna ideja pokazivača:

> Pointer je promenljiva koja čuva memorijsku adresu druge vrednosti.

---

Primer:

```text
Variable

+-------+
|  42   |
+-------+
   |
   |
   v

Memory Address

0xc0000100
````

---

Pointer čuva:

```text
0xc0000100
```

a ne:

```text
42
```

---

# Zašto postoje pointeri?

Pointeri omogućavaju:

* direktan pristup memorijskoj lokaciji;
* prosleđivanje velikih struktura bez kopiranja;
* menjanje vrednosti iz druge funkcije;
* efikasniji rad sa podacima;
* implementaciju kompleksnijih struktura podataka.

---

Primer problema bez pointera:

```go
func change(value int) {

	value = 100

}
```

---

Poziv:

```go
number := 10

change(number)
```

---

Rezultat:

```text
number = 10
```

---

Zašto?

Zato što funkcija dobija kopiju vrednosti.

---

Sa pointerom:

```go
func change(value *int) {

	*value = 100

}
```

---

Sada funkcija može menjati originalnu vrednost.

---

# Memory Model

Da bismo razumeli pointere, prvo moramo razumeti promenljive.

Primer:

```go
number := 42
```

---

U memoriji konceptualno:

```text
Memory

Address        Value

0xc0001000       42
```

---

Promenljiva:

```text
number
```

predstavlja ime koje pokazuje na tu memorijsku lokaciju.

---

Pointer:

```go
pointer := &number
```

čuva adresu:

```text
pointer

0xc0001000
```

---

Model:

```text
number

+------+
| 42   |
+------+
   ^
   |
   |
pointer

+--------------+
| 0xc0001000   |
+--------------+
```

---

# Pointer Type

Svaki pointer ima svoj tip.

Primer:

```go
var number int = 10

var pointer *int
```

---

Tip:

```text
*int
```

znači:

```text
pointer to int
```

---

Drugi primeri:

```go
*string
```

pointer na string.

---

```go
*float64
```

pointer na float64.

---

```go
*User
```

pointer na struct tip `User`.

---

# Address Operator `&`

Operator:

```go
&
```

uzima adresu promenljive.

---

Primer:

```go
package main

import "fmt"

func main() {

	number := 42

	fmt.Println(&number)

}
```

---

Rezultat:

```text
0xc0000120a0
```

---

Ova vrednost predstavlja memorijsku adresu.

---

# Pointer Declaration

Možemo deklarisati pointer:

```go
var pointer *int
```

---

Trenutna vrednost:

```text
nil
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var pointer *int

	fmt.Println(pointer)

}
```

---

Rezultat:

```text
<nil>
```

---

# Nil Pointer

Pointer koji nema adresu ima vrednost:

```go
nil
```

---

Primer:

```go
var p *int

if p == nil {

	fmt.Println("Empty pointer")

}
```

---

Rezultat:

```text
Empty pointer
```

---

Važno pravilo:

Nikada ne dereferencirati nil pointer.

---

Pogrešno:

```go
*p
```

ako:

```go
p == nil
```

---

Rezultat:

```text
panic: invalid memory address
```

---

# Pointer Initialization

Pointer najčešće dobijamo pomoću:

```go
&
```

---

Primer:

```go
number := 100

pointer := &number
```

---

Sada:

```text
pointer

↓

number
```

---

Provera:

```go
fmt.Println(pointer)
```

daje adresu.

---

# Dereference Operator `*`

Operator:

```go
*
```

ima dva značenja.

---

## 1. Deklaracija pointer tipa

```go
var p *int
```

znači:

```text
pointer to int
```

---

## 2. Pristup vrednosti na adresi

```go
*p
```

znači:

```text
value stored at address p
```

---

Primer:

```go
package main

import "fmt"

func main() {

	number := 42

	pointer := &number

	fmt.Println(*pointer)

}
```

---

Rezultat:

```text
42
```

---

# Menjanje vrednosti preko Pointera

Jedna od glavnih prednosti pointera.

Primer:

```go
package main

import "fmt"

func main() {

	number := 10

	pointer := &number

	*pointer = 50

	fmt.Println(number)

}
```

---

Rezultat:

```text
50
```

---

Šta se desilo?

```text
Before:

number

+----+
| 10 |
+----+


After:

number

+----+
| 50 |
+----+
```

---

# Pointer Assignment

Pointeri mogu pokazivati na istu vrednost.

Primer:

```go
a := 10

p1 := &a

p2 := p1
```

---

Sada:

```text
p1

 |

 v

a


p2

 |

 v

a
```

---

Promena:

```go
*p2 = 20
```

menja:

```go
a
```

---

Rezultat:

```text
a = 20
```

---

# Pointer Equality

Pointeri mogu biti poređeni.

Dozvoljeno:

```go
==
```

i:

```go
!=
```

---

Primer:

```go
a := 10

p1 := &a

p2 := &a

fmt.Println(p1 == p2)
```

---

Rezultat:

```text
true
```

---

Porede se:

```text
memory addresses
```

---

# Pointer na Pointer

Go dozvoljava pointere višeg nivoa.

Primer:

```go
number := 10

p := &number

pp := &p
```

---

Model:

```text
pp

 |

 v

p

 |

 v

number

 |

 v

10
```

---

Tipovi:

```text
number

int


p

*int


pp

**int
```

---

# Primer korišćenja

```go
package main

import "fmt"

func main() {

	value := 100

	pointer := &value

	fmt.Println(value)

	fmt.Println(*pointer)

}
```

---

Rezultat:

```text
100

100
```

---

# Pointeri i Functions

Go prosleđuje argumente po vrednosti.

Primer:

```go
func increment(value int) {

	value++

}
```

---

Poziv:

```go
number := 10

increment(number)
```

---

Rezultat:

```text
number = 10
```

---

Sa pointerom:

```go
func increment(value *int) {

	(*value)++

}
```

---

Poziv:

```go
increment(&number)
```

---

Rezultat:

```text
number = 11
```

---

# Pointer Receiver

Pointeri se često koriste kod metoda.

Primer:

```go
type User struct {

	Name string

}
```

---

Value receiver:

```go
func (u User) ChangeName(name string) {

	u.Name = name

}
```

---

Ne menja original.

---

Pointer receiver:

```go
func (u *User) ChangeName(name string) {

	u.Name = name

}
```

---

Menja original.

---

Ovo ćemo detaljno obraditi u delu o:

* structs;
* methods;
* interfaces.

---

# Pointeri i Structs

Najčešći slučaj upotrebe.

Primer:

```go
type User struct {

	Name string

}

user := User{
	Name: "Marko",
}

pointer := &user
```

---

Pristup:

```go
pointer.Name
```

je dozvoljen.

Go automatski radi:

```go
(*pointer).Name
```

---

# Automatska Dereferencija

Go pojednostavljuje rad.

Umesto:

```go
(*user).Name
```

možemo:

```go
user.Name
```

---

Primer:

```go
user := &User{
	Name: "Go",
}

fmt.Println(user.Name)
```

---

Go automatski dereferencira pointer.

---

# Pointeri i Garbage Collector

Pointeri utiču na životni vek objekata.

Ako pointer pokazuje na vrednost:

```text
object

^

|

pointer
```

Garbage Collector smatra objekat dostupnim.

---

Kada više nema referenci:

```text
object

X

no pointers
```

GC može osloboditi memoriju.

---

# Najčešće greške

---

## 1. Dereferenciranje nil pointera

Pogrešno:

```go
var p *int

fmt.Println(*p)
```

---

## 2. Nepotrebno korišćenje pointera

Nije svaka vrednost kandidat za pointer.

---

Primer:

```go
func add(a int, b int) int
```

nije potrebno:

```go
func add(a *int, b *int) *int
```

---

## 3. Gubitak reference

Primer:

```go
p := &value

p = nil
```

---

Sada pointer više ne pokazuje na vrednost.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je pointer;
* kako rade memorijske adrese;
* operator `&`;
* operator `*`;
* dereferenciranje;
* nil pointer;
* menjanje vrednosti preko pointera;
* prosleđivanje pointera funkcijama;
* osnovnu vezu pointera i memorije.

---

# Zaključak

Pointeri su fundamentalni deo Go jezika.

Najvažniji koncept:

```text
Pointer

=

memory address of a value
```

Operatori:

```text
&

=

get address


*

=

get value from address
```

Razumevanje pointera predstavlja osnovu za naprednije Go teme:

* structs;
* methods;
* interfaces;
* memory allocation;
* escape analysis;
* performance optimizacije.

U sledećem delu nastavljamo sa **naprednijim aspektima pointera: pointer semantika, value vs pointer ponašanje i kada koristiti pointere u Go aplikacijama**.

---

# Pointer Semantics: Value vs Pointer Behavior

U prethodnom delu smo naučili:

- šta su pointeri;
- kako se dobija memorijska adresa pomoću operatora `&`;
- kako se pristupa vrednosti pomoću operatora `*`;
- kako pointer omogućava menjanje originalne vrednosti.

Sada prelazimo na važniji koncept:

> Kada koristiti vrednost (`value`) a kada pointer (`pointer`)?

---

# Go koristi Pass by Value

Jedna od najvažnijih stvari u Go-u:

> Go uvek prosleđuje argumente funkcijama po vrednosti.

To znači:

```text
argument

↓

copy

↓

function
````

---

Primer:

```go
package main

import "fmt"

func change(value int) {

	value = 100

}

func main() {

	number := 10

	change(number)

	fmt.Println(number)

}
```

---

Rezultat:

```text
10
```

---

Šta se dogodilo?

Pre poziva:

```text
number

+----+
| 10 |
+----+
```

---

Prilikom poziva:

```go
change(number)
```

kreira se kopija:

```text
Original

number

+----+
| 10 |
+----+


Copy

value

+----+
| 10 |
+----+
```

---

Funkcija menja kopiju:

```text
value

+-----+
| 100 |
+-----+
```

---

Original ostaje:

```text
number

+----+
| 10 |
+----+
```

---

# Pass by Value sa Pointerom

Ako prosledimo pointer:

```go
func change(value *int) {

	*value = 100

}
```

---

Poziv:

```go
number := 10

change(&number)
```

---

Sada se kopira adresa:

```text
Original:

number

+----+
| 10 |
+----+
   ^
   |
   |
address


Function:

value

+--------------+
| 0xc0000100   |
+--------------+
```

---

Funkcija dobija adresu originalne vrednosti.

Zato:

```go
*value = 100
```

menja:

```go
number
```

---

Rezultat:

```text
100
```

---

# Value Semantics

Value semantics znači:

> Radimo sa kopijom podatka.

---

Primer:

```go
type User struct {

	Name string
	Age int

}
```

---

Funkcija:

```go
func update(user User) {

	user.Age = 30

}
```

---

Poziv:

```go
u := User{
	Name: "Marko",
	Age: 20,
}

update(u)
```

---

Original:

```text
Age = 20
```

ostaje isti.

---

Zašto?

Zato što je struct kopiran.

---

# Pointer Semantics

Pointer semantics znači:

> Radimo sa originalnim podatkom preko memorijske adrese.

---

Primer:

```go
func update(user *User) {

	user.Age = 30

}
```

---

Poziv:

```go
update(&u)
```

---

Sada:

```text
u

+-----------+
| Age: 30   |
+-----------+

^

|

pointer
```

---

Original je promenjen.

---

# Value Receiver vs Pointer Receiver

Kod metoda je veoma važno.

---

## Value Receiver

Primer:

```go
type User struct {

	Name string

}


func (u User) ChangeName(name string) {

	u.Name = name

}
```

---

Poziv:

```go
user.ChangeName("Go")
```

---

Ne menja original.

---

Zašto?

Metoda dobija kopiju:

```text
Original User

+

Copy User
```

---

## Pointer Receiver

Primer:

```go
func (u *User) ChangeName(name string) {

	u.Name = name

}
```

---

Sada:

```text
Method

   |
   v

Original struct
```

---

Promena ostaje.

---

# Kada koristiti Value?

Value pristup je dobar kada:

* objekat je mali;
* nema potrebe za izmenom;
* želimo sigurniji kod;
* kopiranje nije skupo.

---

Primer:

```go
type Point struct {

	X int
	Y int

}
```

---

Point je mali:

```text
2 integers
```

---

Kopiranje je jeftino.

---

Metoda:

```go
func distance(p Point) float64
```

je potpuno prihvatljiva.

---

# Kada koristiti Pointer?

Pointer koristimo kada:

* želimo menjati vrednost;
* struct je veliki;
* želimo izbeći kopiranje;
* važan je identitet objekta.

---

Primer:

```go
type LargeDocument struct {

	Content string
	Data []byte
	Metadata map[string]string

}
```

---

Kopiranje može biti skuplje.

---

Bolje:

```go
func process(doc *LargeDocument)
```

---

# Pointeri i Performance

Primer:

```go
func process(user User)
```

---

Compiler mora kopirati:

```text
User

↓

copy

↓

function
```

---

Ako je struct:

```go
type User struct {

	ID int
	Name string
	Email string
	Address string
	CreatedAt time.Time

}
```

kopiranje je skuplje.

---

Pointer:

```go
func process(user *User)
```

kopira samo:

```text
memory address
```

---

Pointer veličina:

```text
8 bytes
```

na 64-bitnim sistemima.

---

# Ali Pointer nije uvek brži

Važno:

Ne treba automatski koristiti pointer.

---

Primer:

```go
type Point struct {

	X int
	Y int

}
```

---

Veličina:

```text
16 bytes
```

---

Kopiranje:

```text
brzo
```

---

Pointer:

```go
*Point
```

može zahtevati:

* dereferenciranje;
* dodatni memory access;
* potencijalni heap allocation.

---

Dakle:

```text
Small value

=

value receiver
```

---

```text
Large mutable object

=

pointer receiver
```

---

# Pointer Escape i Heap Allocation

Pointer može uticati na memorijsku alokaciju.

Primer:

```go
func createUser() *User {

	user := User{}

	return &user

}
```

---

Na prvi pogled:

```text
user

=

local variable
```

---

Ali vraćamo njegovu adresu.

Zato compiler zaključuje:

```text
variable survives function
```

---

Rezultat:

```text
stack

↓

heap
```

---

Ovo se zove:

```text
escape analysis
```

---

Detaljno ćemo obraditi kasnije u:

```text
Advanced Go → Memory Management
```

---

# Pointer i Interface Behavior

Pointeri imaju važnu ulogu kod interface implementacije.

Primer:

```go
type Speaker interface {

	Speak()

}
```

---

Struct:

```go
type Dog struct{}
```

---

Metoda:

```go
func (d *Dog) Speak(){

}
```

---

Implementacija:

```go
var s Speaker

dog := Dog{}

s = dog
```

---

Greška.

Zašto?

Zato što:

```text
Dog

ne implementira

Speaker
```

---

Ali:

```go
s = &dog
```

radi.

---

Razlog:

Metoda postoji na:

```text
*Dog
```

ne na:

```text
Dog
```

---

Ovo je veoma važan koncept u Go-u.

---

# Nil Pointer Behavior

Pointer može biti:

```go
nil
```

---

Primer:

```go
var user *User

fmt.Println(user)
```

---

Rezultat:

```text
<nil>
```

---

Ali:

```go
user.Name
```

---

izaziva:

```text
panic
```

---

Zato često proveravamo:

```go
if user != nil {

	fmt.Println(user.Name)

}
```

---

# Pointer Idiomi u Go-u

Go stil preferira jednostavnost.

---

Čest obrazac:

```go
func NewUser() *User {

	return &User{}

}
```

---

Constructor funkcije često vraćaju pointer.

---

Primer:

```go
user := NewUser()
```

---

Dobijamo:

```text
*User
```

---

# Pointer vs Value Decision Table

| Situacija                       | Preporuka |
| ------------------------------- | --------- |
| Mali struct                     | Value     |
| Veliki struct                   | Pointer   |
| Potrebna mutacija               | Pointer   |
| Immutable koncept               | Value     |
| Interface sa pointer receiverom | Pointer   |
| Kopiranje je skupo              | Pointer   |
| Simple primitive type           | Value     |

---

# Primer iz standardne biblioteke

`time.Time` je dobar primer value tipa.

```go
t := time.Now()
```

---

Kopiranje:

```go
func printTime(t time.Time)
```

je normalno.

---

Zašto?

Jer je:

* mali;
* immutable po dizajnu;
* siguran za kopiranje.

---

# Najčešće greške

---

## 1. Korišćenje pointera svuda

Pogrešno:

```go
func add(a *int, b *int) *int
```

---

Za male vrednosti:

```go
func add(a int, b int) int
```

je bolje.

---

## 2. Zaboravljanje da Go kopira vrednosti

Pogrešno očekivanje:

```go
func update(user User)
```

menja original.

---

## 3. Nepotrebni heap allocation

Previše pointera može dovesti do:

* više GC rada;
* više alokacija;
* slabijih performansi.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* Go pass-by-value model;
* razliku između value i pointer semantics;
* kada koristiti pointer;
* kada koristiti value;
* pointer receiver;
* interface ponašanje sa pointerima;
* uticaj pointera na performanse i memoriju.

---

# Zaključak

Najvažnije pravilo:

```text
Use values by default.

Use pointers when you need:
- mutation,
- identity,
- avoiding expensive copies.
```

Pointer nije automatski "bolji".

Dobar Go kod bira između:

```text
Value Semantics

ili

Pointer Semantics
```

na osnovu dizajna i potreba aplikacije.

U sledećem delu prelazimo na **zero values, default inicijalizaciju i ponašanje primitive tipova bez eksplicitnog dodeljivanja vrednosti**.

---

# Zero Values in Go

Jedna od najvažnijih karakteristika Go jezika jeste koncept:

```text
zero value
````

Go inicijalizuje svaku promenljivu automatski sa podrazumevanom vrednošću ukoliko joj nije dodeljena eksplicitna vrednost.

---

Osnovna ideja:

```text
Variable declaration

        |

        v

Memory allocation

        |

        v

Zero value assigned
```

---

Za razliku od nekih drugih jezika gde neinicijalizovana promenljiva može sadržati nepredvidivu vrednost, Go garantuje:

> Svaka promenljiva u Go-u uvek ima validnu početnu vrednost.

---

# Primer Zero Value ponašanja

Primer:

```go
package main

import "fmt"

func main() {

	var number int

	fmt.Println(number)

}
```

---

Rezultat:

```text
0
```

---

Iako nismo napisali:

```go
number := 0
```

Go automatski postavlja:

```text
number = 0
```

---

# Zero Values Primitive Tipova

Svaki osnovni tip ima svoju zero value.

---

## Integer Tipovi

Svi integer tipovi imaju:

```text
0
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var a int
	var b int8
	var c int64

	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(c)

}
```

---

Rezultat:

```text
0
0
0
```

---

---

## Unsigned Integer Tipovi

Za:

```text
uint
uint8
uint16
uint32
uint64
```

zero value je:

```text
0
```

---

Primer:

```go
var size uint
```

vrednost:

```text
0
```

---

# Floating Point Zero Value

Za:

```text
float32
float64
```

zero value:

```text
0.0
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var price float64

	fmt.Println(price)

}
```

---

Rezultat:

```text
0
```

---

Interno:

```text
0.0
```

---

# Complex Numbers

Za:

```text
complex64
complex128
```

zero value:

```text
0 + 0i
```

---

Primer:

```go
var c complex128

fmt.Println(c)
```

---

Rezultat:

```text
(0+0i)
```

---

# Boolean Zero Value

Za:

```text
bool
```

zero value:

```text
false
```

---

Primer:

```go
var enabled bool

fmt.Println(enabled)
```

---

Rezultat:

```text
false
```

---

# String Zero Value

Za string:

```text
empty string
```

odnosno:

```text
""
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var name string

	fmt.Println(name)

}
```

---

Rezultat:

```text
(empty line)
```

---

Vrednost:

```go
name == ""
```

---

# Pointer Zero Value

Za pointer:

```text
nil
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var pointer *int

	fmt.Println(pointer)

}
```

---

Rezultat:

```text
<nil>
```

---

Pointer ne pokazuje ni na jednu memorijsku adresu.

---

# Zero Values Summary Table

| Tip       | Zero Value |
| --------- | ---------- |
| int       | `0`        |
| int8      | `0`        |
| int16     | `0`        |
| int32     | `0`        |
| int64     | `0`        |
| uint      | `0`        |
| float32   | `0.0`      |
| float64   | `0.0`      |
| complex64 | `0+0i`     |
| bool      | `false`    |
| string    | `""`       |
| pointer   | `nil`      |

---

# Zero Values za Aggregate Tipove

Pored primitive tipova, Go ima i složenije tipove.

To su:

* arrays;
* slices;
* maps;
* structs;
* interfaces;
* channels;
* functions.

---

# Array Zero Value

Array uvek ima fiksnu veličinu.

Primer:

```go
var numbers [3]int
```

---

Rezultat:

```text
[
0
0
0
]
```

---

Svaki element dobija zero value svog tipa.

---

Model:

```text
numbers

+---+---+---+
| 0 | 0 | 0 |
+---+---+---+
```

---

# Slice Zero Value

Slice zero value je:

```text
nil
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var numbers []int

	fmt.Println(numbers)

}
```

---

Rezultat:

```text
[]
```

---

Ali:

```go
numbers == nil
```

je:

```text
true
```

---

Važno:

Slice ima:

```text
nil slice
```

i:

```text
empty slice
```

---

Razlika:

Nil slice:

```go
var a []int
```

---

Empty slice:

```go
b := []int{}
```

---

Iako izgledaju slično:

```text
[]
```

nisu identični.

---

# Map Zero Value

Map zero value je:

```text
nil
```

---

Primer:

```go
var users map[string]int
```

---

Vrednost:

```text
nil
```

---

Čitanje je dozvoljeno:

```go
value := users["Marko"]
```

---

Rezultat:

```text
0
```

---

Ali upis nije dozvoljen:

```go
users["Marko"] = 100
```

---

Dobijamo:

```text
panic: assignment to entry in nil map
```

---

Zato map inicijalizujemo:

```go
users := make(map[string]int)
```

---

# Struct Zero Value

Struct dobija zero value za sva svoja polja.

---

Primer:

```go
type User struct {

	Name string
	Age int

}
```

---

Deklaracija:

```go
var user User
```

---

Dobijamo:

```text
Name: ""
Age: 0
```

---

Vizuelno:

```text
User

+------------+
| Name: ""   |
| Age: 0     |
+------------+
```

---

# Interface Zero Value

Interface zero value je:

```text
nil
```

---

Primer:

```go
var value interface{}

fmt.Println(value)
```

---

Rezultat:

```text
<nil>
```

---

Interface je nil kada:

* nema dynamic type;
* nema dynamic value.

---

Ovo ćemo detaljno obraditi kasnije u poglavlju o:

```text
Interfaces and Polymorphism
```

---

# Function Zero Value

Function promenljiva može imati:

```text
nil
```

---

Primer:

```go
var operation func()
```

---

Vrednost:

```text
nil
```

---

Poziv:

```go
operation()
```

---

Rezultat:

```text
panic
```

---

Provera:

```go
if operation != nil {

	operation()

}
```

---

# Channel Zero Value

Channel zero value:

```text
nil
```

---

Primer:

```go
var ch chan int
```

---

Vrednost:

```text
nil
```

---

Operacije nad nil channel:

```text
send

block forever
```

---

```text
receive

block forever
```

---

```text
close

panic
```

---

Channels ćemo detaljno obraditi u:

```text
Concurrency
```

---

# Zašto su Zero Values važne?

Go dizajn favorizuje:

```text
usable zero value
```

---

To znači:

Tip treba da bude koristan odmah nakon deklaracije.

---

Primer:

```go
var count int
count++
```

Radi.

---

Nema potrebe:

```go
count = 0
```

---

# Primer iz Standard Library

`bytes.Buffer` je poznat primer.

---

Možemo:

```go
var buffer bytes.Buffer

buffer.WriteString("Hello")
```

---

Bez:

```go
buffer = bytes.NewBuffer(...)
```

---

Zero value je odmah upotrebljiva.

---

# Dizajn sopstvenih tipova

Kada kreiramo sopstvene tipove, dobro je razmišljati:

> Da li zero value mog tipa ima smisla?

---

Dobar dizajn:

```go
type Counter struct {

	value int

}
```

---

Može odmah:

```go
var c Counter

c.Increment()
```

---

Loš dizajn:

```go
type Database struct {

	connection *Connection

}
```

---

Ako zahteva obaveznu inicijalizaciju:

```go
db.Start()
```

možda zero value nije korisna.

---

# Zero Value vs Constructor Functions

Go nema klasične konstruktore.

Zato često koristimo:

```go
NewType()
```

funkcije.

---

Primer:

```go
user := NewUser()
```

---

Ali dobar Go dizajn često omogućava:

```go
var user User
```

da bude validan.

---

# Najčešće greške

---

## 1. Pretpostavljanje da promenljive imaju random vrednosti

U Go-u:

```go
var x int
```

nije:

```text
undefined
```

nego:

```text
0
```

---

## 2. Pisanje nepotrebnih inicijalizacija

Suvišno:

```go
var count int = 0
```

---

Idiomatic Go:

```go
var count int
```

---

## 3. Mešanje nil i empty vrednosti

Posebno kod:

* slices;
* maps;
* pointers;
* interfaces.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je zero value;
* zašto Go garantuje inicijalne vrednosti;
* zero value primitive tipova;
* zero value složenih tipova;
* razliku između nil i empty;
* kako dizajnirati tipove sa korisnom zero value.

---

# Zaključak

Zero value je jedna od ključnih Go filozofija:

```text
The zero value should be useful.
```

Go pokušava da omogući da objekti budu odmah spremni za korišćenje bez obavezne inicijalizacije.

Razumevanje zero value koncepta je osnova za pisanje idiomatskog Go koda.

U sledećem delu prelazimo na **Arrays u Go-u: deklaracija, veličina, inicijalizacija, iteracija i ponašanje kao value tipa**.

---

# Arrays in Go

U prethodnim delovima smo obradili:

- primitive tipove;
- type conversion;
- constants;
- pointers;
- value i pointer semantics;
- zero values.

Sada prelazimo na prvi složeni tip podatka u Go-u:

```text
Array
````

---

Array predstavlja:

> Fiksno veliku kolekciju elemenata istog tipa.

---

Osnovna karakteristika array-a:

```text
Fixed Size

+

Same Type Elements
```

---

Primer:

```go
var numbers [5]int
```

znači:

```text
Array

5 elemenata

svaki tipa int
```

---

Vizuelno:

```text
numbers

+----+----+----+----+----+
| 0  | 0  | 0  | 0  | 0  |
+----+----+----+----+----+
 0    1    2    3    4
```

---

# Array Declaration

Osnovna sintaksa:

```go
var name [size]Type
```

---

Primer:

```go
var ages [3]int
```

---

Struktura:

```text
name

=

ages


size

=

3


type

=

int
```

---

Go compiler zna:

```text
ages

ima tačno

3 int vrednosti
```

---

# Array Zero Value

Kao što smo videli u prethodnom delu:

Array dobija zero value za svaki element.

---

Primer:

```go
package main

import "fmt"

func main() {

	var numbers [5]int

	fmt.Println(numbers)

}
```

---

Rezultat:

```text
[0 0 0 0 0]
```

---

Svaki element:

```text
0
```

---

# Array Initialization

Array možemo odmah inicijalizovati.

---

Primer:

```go
numbers := [5]int{1,2,3,4,5}
```

---

Rezultat:

```text
[1 2 3 4 5]
```

---

Vizuelno:

```text
+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 |
+---+---+---+---+---+
  0   1   2   3   4
```

---

# Array Size Inference

Go može sam zaključiti veličinu.

Koristi se:

```go
...
```

---

Primer:

```go
numbers := [...]int{1,2,3,4,5}
```

---

Compiler vidi:

```text
5 elemenata
```

i kreira:

```go
[5]int
```

---

Isto kao:

```go
numbers := [5]int{1,2,3,4,5}
```

---

# Array sa Manje Elemenata

Dozvoljeno:

```go
numbers := [5]int{1,2}
```

---

Rezultat:

```text
[1 2 0 0 0]
```

---

Preostali elementi dobijaju:

```text
zero value
```

---

# Specificiranje Index vrednosti

Možemo inicijalizovati određene indekse.

---

Primer:

```go
numbers := [5]int{
	0: 10,
	3: 40,
}
```

---

Rezultat:

```text
[10 0 0 40 0]
```

---

Mapiranje:

```text
index:value

0 -> 10

3 -> 40
```

---

# Array Length

Veličina array-a dobija se pomoću:

```go
len()
```

---

Primer:

```go
numbers := [5]int{1,2,3,4,5}

fmt.Println(len(numbers))
```

---

Rezultat:

```text
5
```

---

Kod array-a:

```text
len(array)

=

array size
```

---

# Array Indexing

Array elementi imaju indekse.

Prvi element:

```text
index 0
```

---

Primer:

```go
numbers := [3]int{10,20,30}
```

---

Memorijski prikaz:

```text
+----+----+----+
|10  |20  |30  |
+----+----+----+
 0    1    2
```

---

Pristup:

```go
fmt.Println(numbers[0])
```

---

Rezultat:

```text
10
```

---

# Menjanje Array Elementa

Array elementi mogu biti menjani.

---

Primer:

```go
numbers := [3]int{10,20,30}

numbers[1] = 100
```

---

Rezultat:

```text
[10 100 30]
```

---

---

# Array Index Out of Range

Pokušaj pristupa nepostojećem indeksu:

```go
numbers := [3]int{1,2,3}

fmt.Println(numbers[5])
```

---

Rezultat:

```text
panic: index out of range
```

---

Zašto?

Array ima samo:

```text
0,1,2
```

indekse.

---

# Iteracija preko Array-a

Postoji nekoliko načina.

---

## Klasična for petlja

Primer:

```go
numbers := [5]int{1,2,3,4,5}

for i := 0; i < len(numbers); i++ {

	fmt.Println(numbers[i])

}
```

---

Rezultat:

```text
1
2
3
4
5
```

---

# Range Iteracija

Idiomatic Go način:

```go
numbers := [5]int{1,2,3,4,5}

for index, value := range numbers {

	fmt.Println(index, value)

}
```

---

Rezultat:

```text
0 1
1 2
2 3
3 4
4 5
```

---

# Ignorisanje Index vrednosti

Ako nam index nije potreban:

```go
for _, value := range numbers {

	fmt.Println(value)

}
```

---

Operator:

```text
_
```

znači:

```text
discard value
```

---

# Array Copy Behavior

Jedna od najvažnijih osobina array-a:

> Array je value type.

---

Primer:

```go
package main

import "fmt"

func main() {

	a := [3]int{1,2,3}

	b := a

	b[0] = 100

	fmt.Println(a)
	fmt.Println(b)

}
```

---

Rezultat:

```text
[1 2 3]

[100 2 3]
```

---

Zašto?

Zato što:

```text
b

dobija kopiju array-a
```

---

Model:

Pre:

```text
a

+---+---+---+
|1  |2  |3  |
+---+---+---+
```

---

Posle:

```text
a

+---+---+---+
|1  |2  |3  |
+---+---+---+


b

+-----+---+---+
|100  |2  |3  |
+-----+---+---+
```

---

# Array kao Function Argument

Pošto je array value type:

```go
func process(numbers [5]int)
```

dobija kopiju.

---

Primer:

```go
func change(numbers [3]int){

	numbers[0] = 100

}
```

---

Poziv:

```go
a := [3]int{1,2,3}

change(a)
```

---

Original ostaje:

```text
[1 2 3]
```

---

# Pointer na Array

Ako želimo menjati original:

```go
func change(numbers *[3]int){

	numbers[0] = 100

}
```

---

Poziv:

```go
change(&a)
```

---

Sada:

```text
Original array

se menja
```

---

# Array Memory Layout

Array elementi se nalaze uzastopno u memoriji.

Primer:

```go
numbers := [3]int{10,20,30}
```

---

Memorija:

```text
Address

0x1000

+----+
| 10 |
+----+

0x1008

+----+
| 20 |
+----+

0x1010

+----+
| 30 |
+----+
```

---

Elementi su:

```text
contiguous memory
```

---

Ovo omogućava:

* brz pristup;
* cache-friendly ponašanje;
* predvidivu memoriju.

---

# Array i Generics

U modernom Go-u array može biti korišćen sa generičkim funkcijama.

Primer:

```go
func Print[T any](values [3]T){

}
```

---

Ali u osnovnom Go radu češće se koriste:

```text
slices
```

---

# Kada koristiti Array?

Array je dobar kada:

* veličina je poznata unapred;
* veličina se nikada ne menja;
* potrebna je fiksna memorija.

---

Primer:

```go
var rgb [3]byte
```

---

RGB boja:

```text
R

G

B
```

uvek ima:

```text
3 vrednosti
```

---

Drugi primer:

```go
var matrix [3][3]int
```

---

Matrica:

```text
3 x 3
```

---

# Kada NE koristiti Array?

U većini aplikacija:

```text
Slice > Array
```

---

Zašto?

Slice pruža:

* dinamičku veličinu;
* fleksibilnost;
* standardni način rada sa kolekcijama.

---

Primer:

Umesto:

```go
[100]int
```

češće:

```go
[]int
```

---

# Array vs Slice

| Karakteristika | Array                | Slice             |
| -------------- | -------------------- | ----------------- |
| Veličina       | Fiksna               | Dinamička         |
| Value type     | Da                   | Ne                |
| Kopiranje      | Kopira sve elemente  | Kopira header     |
| Fleksibilnost  | Mala                 | Velika            |
| Upotreba       | Specifični slučajevi | Većina aplikacija |

---

# Najčešće greške

---

## 1. Mešanje array veličina

Ovo su različiti tipovi:

```go
[3]int
```

i:

```go
[4]int
```

---

Ne mogu:

```go
var a [3]int

var b [4]int

a = b
```

---

## 2. Očekivanje reference ponašanja

Pogrešno:

```go
b := a
```

ne pravi referencu.

---

Pravi kopiju.

---

## 3. Korišćenje array-a gde treba slice

Array je često previše rigidan.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je array;
* deklaraciju array-a;
* fiksnu veličinu;
* indexing;
* iteraciju;
* zero value ponašanje;
* array copy semantics;
* value type prirodu array-a;
* razliku između array-a i slice-a.

---

# Zaključak

Array u Go-u je jednostavan, ali važan tip:

```text
Array

=

fixed-size collection of same-type values
```

Najvažnije osobine:

```text
Fixed size

Value type

Contiguous memory

Same element type
```

Iako se u modernim Go aplikacijama češće koriste slice-ovi, razumevanje array-a je neophodno jer slice direktno zavisi od array koncepta.

U sledećem delu prelazimo na **Slices u Go-u: dinamičke kolekcije, slice header, length, capacity i osnovni model memorije**.

---

# Slices in Go

U prethodnom delu smo obradili:

- šta su array-i;
- fiksnu veličinu array-a;
- indexing;
- iteraciju;
- value semantics;
- razliku između array-a i slice-a.

Sada prelazimo na jedan od najvažnijih tipova podataka u Go-u:

```text
Slice
````

---

Slice je:

> Dinamički pogled (`view`) nad nizom elemenata koji se nalaze u osnovnom array-u.

---

Za razliku od array-a:

```text
Array

=

fixed size
```

dok je:

```text
Slice

=

dynamic size
```

---

U realnim Go aplikacijama:

```text
Slice > Array
```

je najčešći slučaj.

---

# Slice Characteristics

Slice ima nekoliko ključnih osobina:

* dinamičku veličinu;
* može rasti i smanjivati se;
* referencira underlying array;
* predstavlja descriptor nad memorijom;
* predstavlja reference-like tip.

---

Model:

```text
Slice

+-------------+
| pointer     |
| length      |
| capacity    |
+-------------+

        |
        |
        v

Underlying Array

+----+----+----+----+----+
| 10 | 20 | 30 | 40 | 50 |
+----+----+----+----+----+
```

---

# Slice Declaration

Najjednostavniji način:

```go
var numbers []int
```

---

Ovo nije array.

Razlika:

Array:

```go
var numbers [5]int
```

---

Slice:

```go
var numbers []int
```

---

Array:

```text
ima veličinu
```

---

Slice:

```text
nema veličinu
```

---

# Slice Zero Value

Kao što smo videli:

Zero value za slice je:

```text
nil
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var numbers []int

	fmt.Println(numbers)
	fmt.Println(numbers == nil)

}
```

---

Rezultat:

```text
[]
true
```

---

Slice postoji, ali nema:

* underlying array;
* length;
* capacity.

---

# Creating Slices

Postoji nekoliko načina kreiranja slice-a.

---

## 1. Slice Literal

Primer:

```go
numbers := []int{1,2,3,4,5}
```

---

Go kreira:

```text
slice

+

underlying array
```

---

Model:

```text
numbers

+---------+
| pointer |
| len=5   |
| cap=5   |
+---------+

     |
     v

[1][2][3][4][5]
```

---

# Slice Length

Dužina slice-a dobija se pomoću:

```go
len()
```

---

Primer:

```go
numbers := []int{10,20,30}

fmt.Println(len(numbers))
```

---

Rezultat:

```text
3
```

---

Length predstavlja:

```text
broj dostupnih elemenata
```

---

# Slice Capacity

Pored length-a postoji:

```text
capacity
```

---

Dobija se:

```go
cap()
```

---

Primer:

```go
numbers := []int{1,2,3}

fmt.Println(cap(numbers))
```

---

Rezultat:

```text
3
```

---

Capacity predstavlja:

> Koliko elemenata slice može da sadrži pre nego što se kreira novi underlying array.

---

# Length vs Capacity

Primer:

```go
numbers := make([]int, 3, 10)
```

---

Dobijamo:

```text
length = 3

capacity = 10
```

---

Vizuelno:

```text
Underlying Array

+---+---+---+---+---+---+---+---+---+---+
| 0 | 0 | 0 |   |   |   |   |   |   |   |
+---+---+---+---+---+---+---+---+---+---+

<------->

 length = 3


<------------------------------->

 capacity = 10
```

---

# Creating Slices with make()

Funkcija:

```go
make()
```

služi za kreiranje slice-a.

---

Sintaksa:

```go
make([]Type, length, capacity)
```

---

Primer:

```go
numbers := make([]int, 5)
```

---

Rezultat:

```text
[0 0 0 0 0]
```

---

Length:

```text
5
```

Capacity:

```text
5
```

---

# make() sa Capacity

Primer:

```go
numbers := make([]int, 2, 5)
```

---

Dobijamo:

```text
length = 2

capacity = 5
```

---

Pristup:

```go
numbers[0]
numbers[1]
```

---

Ali:

```go
numbers[2]
```

nije dozvoljeno.

---

Zašto?

Zato što length određuje dostupne elemente.

---

# Slice Indexing

Slice koristi iste indekse kao array.

Primer:

```go
numbers := []int{10,20,30}
```

---

Model:

```text
+----+----+----+
|10  |20  |30  |
+----+----+----+
 0    1    2
```

---

Pristup:

```go
fmt.Println(numbers[1])
```

---

Rezultat:

```text
20
```

---

# Menjanje Slice Elemenata

Slice elementi mogu biti menjani.

Primer:

```go
numbers := []int{1,2,3}

numbers[0] = 100
```

---

Rezultat:

```text
[100 2 3]
```

---

Za razliku od kopiranja array-a:

Slice deli isti underlying array.

---

# Slice je Descriptor

Važno:

Slice nije sama kolekcija.

Slice je mala struktura:

```go
type slice struct {

	array unsafe.Pointer
	len   int
	cap   int

}
```

---

Praktično:

```text
Slice

=

pointer

+

length

+

capacity
```

---

Zbog toga kopiranje slice-a ne kopira sve elemente.

---

# Slice Copy Behavior

Primer:

```go
a := []int{1,2,3}

b := a

b[0] = 100
```

---

Rezultat:

```go
fmt.Println(a)
```

dobija:

```text
[100 2 3]
```

---

Zašto?

Zato što:

```text
a

i

b

pokazuju na isti array
```

---

Model:

```text
a

+-------------+
| pointer ----|----+
| len         |    |
| cap         |    |
+-------------+    |
                  v

              [1][2][3]


b

+-------------+
| pointer ----+
| len         |
| cap         |
+-------------+
```

---

# Slice Passing to Functions

Kada prosledimo slice funkciji:

```go
func modify(values []int)
```

---

Kopira se:

```text
slice header
```

a ne elementi.

---

Primer:

```go
func change(values []int){

	values[0] = 100

}
```

---

Poziv:

```go
numbers := []int{1,2,3}

change(numbers)
```

---

Rezultat:

```text
numbers

[100 2 3]
```

---

# Append Function

Najvažnija funkcija za slice:

```go
append()
```

---

Sintaksa:

```go
append(slice, value)
```

---

Primer:

```go
numbers := []int{1,2,3}

numbers = append(numbers,4)
```

---

Rezultat:

```text
[1 2 3 4]
```

---

Važno:

`append` vraća novi slice.

Zato:

```go
numbers = append(numbers,4)
```

a ne:

```go
append(numbers,4)
```

---

# Append i Capacity

Primer:

```go
numbers := make([]int,3,5)

numbers = append(numbers,4)
```

---

Pre:

```text
len = 3

cap = 5
```

---

Posle:

```text
len = 4

cap = 5
```

---

Novi array nije kreiran.

---

# Append kada nema Capacity

Primer:

```go
numbers := []int{1,2,3}

numbers = append(numbers,4,5,6)
```

---

Ako nema prostora:

Go kreira novi underlying array.

---

Proces:

```text
old array

      |

      v

new larger array
```

---

Zatim:

```text
slice pointer

se menja
```

---

# Slice Growth

Kada capacity nije dovoljna:

```text
append()

      |

      v

allocate new array

      |

      v

copy elements

      |

      v

update slice
```

---

Ovo je važno zbog performansi.

---

# Empty Slice vs Nil Slice

Dve različite stvari:

---

Nil slice:

```go
var numbers []int
```

---

Empty slice:

```go
numbers := []int{}
```

---

Provera:

```go
numbers == nil
```

---

Nil:

```text
true
```

---

Empty:

```text
false
```

---

Ali:

```go
len(numbers)
```

je:

```text
0
```

za oba slučaja.

---

# Kada koristiti make()?

Koristimo `make` kada:

* znamo očekivanu veličinu;
* želimo unapred rezervisati memoriju;
* optimizujemo append operacije.

---

Primer:

```go
users := make([]User,0,1000)
```

---

Znači:

```text
početna dužina: 0

rezervisano: 1000
```

---

# Najčešće greške

---

## 1. Zaboravljanje rezultata append()

Pogrešno:

```go
append(numbers,10)
```

---

Ispravno:

```go
numbers = append(numbers,10)
```

---

## 2. Mešanje length i capacity

Pogrešno:

```go
numbers := make([]int,0,10)

numbers[0] = 5
```

---

Zašto?

Length je:

```text
0
```

---

Treba:

```go
numbers = append(numbers,5)
```

---

## 3. Neočekivane izmene kroz deljene slice-ove

Primer:

```go
b := a
```

ne pravi kopiju elemenata.

---

# Array vs Slice

| Karakteristika | Array                  | Slice             |
| -------------- | ---------------------- | ----------------- |
| Veličina       | Fiksna                 | Dinamička         |
| Tip            | `[N]T`                 | `[]T`             |
| Kopiranje      | Kopira elemente        | Kopira header     |
| Memory         | Direktno čuva elemente | Pokazuje na array |
| append         | Ne postoji             | Postoji           |
| Upotreba       | Specifična             | Veoma česta       |

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je slice;
* razliku između array-a i slice-a;
* slice header;
* length;
* capacity;
* make();
* append();
* deljenje underlying array-a;
* nil vs empty slice.

---

# Zaključak

Slice je jedan od centralnih tipova u Go-u:

```text
Slice

=

pointer + length + capacity
```

Najvažnije zapamtiti:

```text
Slice ne čuva podatke direktno.

Slice pokazuje na underlying array.
```

Zbog toga slice omogućava:

* fleksibilnost;
* efikasnost;
* jednostavan rad sa kolekcijama.

U sledećem delu nastavljamo sa naprednijim slice konceptima:

* slicing operacije;
* copy funkcija;
* memory sharing;
* slice internals;
* potencijalni memory leak problemi.

---

# Advanced Slice Operations

U prethodnom delu smo naučili:

- šta je slice;
- kako slice interno funkcioniše;
- odnos između slice-a i underlying array-a;
- razliku između `length` i `capacity`;
- korišćenje `make()` i `append()`;
- ponašanje slice-a prilikom kopiranja.

Sada prelazimo na naprednije operacije nad slice-ovima:

- slicing expressions;
- kreiranje pod-sliceva;
- deljenje memorije;
- `copy()` funkciju;
- probleme sa memorijom;
- bezbedne obrasce korišćenja slice-ova.

---

# Slice Expression

Slice expression omogućava kreiranje novog slice-a iz postojećeg.

Osnovna sintaksa:

```go
slice[start:end]
````

---

Primer:

```go
numbers := []int{10,20,30,40,50}

part := numbers[1:4]
```

---

Rezultat:

```text
[20 30 40]
```

---

Vizuelno:

```text
numbers

index:

0     1     2     3     4

+----+----+----+----+----+
|10  |20  |30  |40  |50  |
+----+----+----+----+----+

      |------------|
      
      part
```

---

# Start i End Indeksi

Kod:

```go
slice[start:end]
```

pravila su:

* `start` je uključen;
* `end` nije uključen.

---

Primer:

```go
numbers := []int{0,1,2,3,4}

numbers[1:4]
```

uzima:

```text
1
2
3
```

ali ne:

```text
4
```

---

Matematički:

```text
[start, end)
```

---

# Slice od Početka

Ako želimo od početka:

```go
numbers[:3]
```

---

Isto kao:

```go
numbers[0:3]
```

---

Primer:

```go
numbers := []int{1,2,3,4,5}

result := numbers[:3]
```

---

Rezultat:

```text
[1 2 3]
```

---

# Slice do Kraja

Ako želimo do kraja:

```go
numbers[2:]
```

---

Isto kao:

```go
numbers[2:len(numbers)]
```

---

Primer:

```go
numbers := []int{1,2,3,4,5}

result := numbers[2:]
```

---

Rezultat:

```text
[3 4 5]
```

---

# Kopiranje Slice Header-a

Kada napravimo novi slice:

```go
part := numbers[1:4]
```

ne kreira se novi array.

---

Dešava se:

```text
Original slice

pointer
length
capacity


        |
        v


Underlying array
```

---

Novi slice:

```text
part

pointer
length
capacity


        |
        v


isti underlying array
```

---

Dakle:

```text
Podaci se dele.
```

---

# Promena Pod-Slice-a Menja Original

Primer:

```go
numbers := []int{1,2,3,4,5}

part := numbers[1:4]

part[0] = 100
```

---

Rezultat:

```go
fmt.Println(numbers)
```

dobija:

```text
[1 100 3 4 5]
```

---

Zašto?

Zato što:

```text
part

i

numbers

koriste isti array
```

---

# Full Slice Expression

Go omogućava kontrolu capacity dela slice-a.

Sintaksa:

```go
slice[start:end:max]
```

---

Primer:

```go
numbers := []int{1,2,3,4,5}

part := numbers[1:3:3]
```

---

Dobijamo:

```text
length = 2

capacity = 2
```

---

Bez trećeg parametra:

```go
numbers[1:3]
```

capacity bi bila:

```text
4
```

---

# Zašto koristiti Full Slice Expression?

Glavni razlog:

> Sprečavanje nenamernog menjanja originalnog array-a kroz append.

---

Primer:

```go
numbers := []int{1,2,3,4,5}

part := numbers[1:3]
```

---

Capacity:

```text
4
```

---

Ako uradimo:

```go
part = append(part,100)
```

može promeniti original.

---

Sa:

```go
part := numbers[1:3:3]
```

capacity:

```text
2
```

---

Append mora kreirati novi array.

---

# Slice i append Behavior

Primer:

```go
numbers := []int{1,2,3}

part := numbers[:2]

part = append(part,100)
```

---

Moguća situacija:

```text
Original array

[1][2][100]
```

---

Original:

```text
numbers

[1 2 100]
```

---

Zato što je postojao slobodan capacity prostor.

---

# copy() Function

Go ima ugrađenu funkciju:

```go
copy()
```

---

Služi za:

> Kopiranje elemenata iz jednog slice-a u drugi.

---

Sintaksa:

```go
copy(destination, source)
```

---

Primer:

```go
source := []int{1,2,3}

destination := make([]int,3)

copy(destination, source)
```

---

Rezultat:

```text
destination

[1 2 3]
```

---

# copy() Kreira Nezavisan Slice

Za razliku od:

```go
b := a
```

---

`copy()` kopira elemente.

---

Primer:

```go
a := []int{1,2,3}

b := make([]int,3)

copy(b,a)

b[0] = 100
```

---

Rezultat:

```go
a

[1 2 3]


b

[100 2 3]
```

---

Nema deljenja memorije.

---

# copy() Return Value

`copy()` vraća broj kopiranih elemenata.

---

Primer:

```go
count := copy(destination, source)
```

---

Ako:

```go
source = [1 2 3]
```

rezultat:

```text
count = 3
```

---

# Kopiranje Slice-ova Različitih Veličina

Primer:

```go
source := []int{1,2,3,4,5}

destination := make([]int,2)

copy(destination, source)
```

---

Rezultat:

```text
destination

[1 2]
```

---

Kopira se:

```text
manji broj elemenata
```

---

Pravilo:

```text
min(len(destination), len(source))
```

---

# Brisanje Elementa iz Slice-a

Go nema ugrađenu:

```text
remove()
```

funkciju.

---

Čest obrazac:

```go
numbers := []int{1,2,3,4,5}

index := 2

numbers = append(
	numbers[:index],
	numbers[index+1:]...,
)
```

---

Rezultat:

```text
[1 2 4 5]
```

---

Šta se dešava:

Pre:

```text
[1][2][3][4][5]
       ^
       remove
```

---

Posle:

```text
[1][2][4][5]
```

---

# Clear Slice-a

Ako želimo ukloniti sve elemente:

```go
numbers = numbers[:0]
```

---

Primer:

```go
numbers := []int{1,2,3}

numbers = numbers[:0]
```

---

Rezultat:

```text
[]
```

---

Ali:

```go
cap(numbers)
```

ostaje isti.

---

Memory i dalje postoji.

---

# Potpuno Oslobađanje Slice Memorije

Ako želimo ukloniti referencu:

```go
numbers = nil
```

---

Sada:

```text
slice

|

nil
```

---

Garbage Collector kasnije može osloboditi underlying array.

---

# Memory Retention Problem

Jedan od čestih problema.

Primer:

```go
func getPart(data []byte) []byte {

	return data[:10]

}
```

---

Ako je:

```text
data

10 MB
```

---

A vraćamo:

```text
10 bytes
```

---

Mali slice i dalje drži:

```text
ceo 10 MB array
```

---

Model:

```text
small slice

pointer
 |
 |
 v

[====================]
        10 MB
```

---

# Rešenje: Kopiranje

Koristimo:

```go
copy()
```

---

Primer:

```go
func getPart(data []byte) []byte {

	result := make([]byte,10)

	copy(result,data[:10])

	return result

}
```

---

Sada:

```text
result

ima svoj array
```

---

Original može biti oslobođen.

---

# Slice i Garbage Collector

GC prati reference.

Ako slice pokazuje na array:

```text
slice

 |

 v

array
```

array ostaje živ.

---

Čak i ako koristimo:

```go
data[:1]
```

ceo array može ostati u memoriji.

---

# Slice Internals Primer

Slice:

```go
numbers := []int{1,2,3}
```

---

Interno:

```text
Slice Header

+------------+
| pointer    |
+------------+
| len = 3    |
+------------+
| cap = 3    |
+------------+


Underlying Array

+---+---+---+
| 1 | 2 | 3 |
+---+---+---+
```

---

# Best Practices

---

## 1. Koristi append pravilno

Dobro:

```go
numbers = append(numbers,value)
```

---

## 2. Koristi make kada znaš veličinu

Dobro:

```go
users := make([]User,0,100)
```

---

## 3. Koristi copy kada želiš nezavisnu kopiju

Dobro:

```go
copy(destination,source)
```

---

## 4. Obrati pažnju na memory retention

Posebno kod:

* velikih fajlova;
* byte buffer-a;
* parsiranja podataka.

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* slice expressions;
* pod-sliceve;
* deljenje underlying array-a;
* full slice expression;
* `copy()` funkciju;
* brisanje elemenata;
* memory retention probleme;
* kako slice utiče na Garbage Collector.

---

# Zaključak

Slice je jednostavan za korišćenje, ali njegov interni model zahteva razumevanje.

Najvažnija stvar:

```text
Slice ne poseduje podatke.

Slice poseduje referencu na podatke.
```

Zbog toga:

```text
b := a
```

nije kopija elemenata.

Ako želimo pravu kopiju:

```text
copy()
```

je pravi alat.

U sledećem delu prelazimo na **Maps u Go-u: hash struktura, kreiranje, pristup vrednostima, insert, update, delete i iteracija**.

---

# Maps in Go

U prethodnim delovima smo obradili:

- array tipove;
- slice tipove;
- rad sa kolekcijama;
- memorijski model slice-a;
- `append()`;
- `copy()`;
- deljenje memorije;
- probleme sa Garbage Collector-om.

Sada prelazimo na još jedan veoma važan kolekcioni tip u Go-u:

```text
Map
````

---

Map predstavlja:

> Kolekciju elemenata organizovanih kao parovi ključ-vrednost (`key-value pairs`).

---

Osnovni model:

```text
Key

 |

 v

Value
```

---

Primer:

```go
ages := map[string]int{
	"Marko": 30,
	"Ana":   25,
}
```

---

Vizuelno:

```text
+-----------+-------+
| Key       | Value |
+-----------+-------+
| Marko     | 30    |
| Ana       | 25    |
+-----------+-------+
```

---

# Map Characteristics

Go map ima sledeće osobine:

* čuva podatke kao `key-value` parove;
* ključ mora biti comparable tip;
* vrednosti mogu biti bilo kog tipa;
* nema garantovan redosled iteracije;
* dinamički raste;
* implementiran je kao hash tabela.

---

# Map Syntax

Osnovna sintaksa:

```go
map[KeyType]ValueType
```

---

Primer:

```go
map[string]int
```

znači:

```text
ključ:

string


vrednost:

int
```

---

Još primera:

```go
map[int]string
```

---

Znači:

```text
int -> string
```

---

```go
map[string][]int
```

---

Znači:

```text
string -> slice int vrednosti
```

---

# Map Declaration

Možemo deklarisati map promenljivu:

```go
var users map[string]int
```

---

Ali ovde map još nije spreman za upis.

---

Vrednost:

```text
nil
```

---

Primer:

```go
package main

import "fmt"

func main() {

	var users map[string]int

	fmt.Println(users)

}
```

---

Rezultat:

```text
map[]
```

---

Ali interno:

```text
users == nil
```

---

# Map Zero Value

Zero value za map je:

```text
nil
```

---

Kao kod slice-a:

```go
var numbers []int
```

ima:

```text
nil
```

---

Isto:

```go
var users map[string]int
```

ima:

```text
nil
```

---

# Writing to Nil Map

Ovo nije dozvoljeno:

```go
var users map[string]int

users["Marko"] = 30
```

---

Rezultat:

```text
panic:

assignment to entry in nil map
```

---

Zašto?

Zato što map nema internu strukturu.

---

Pre upisa mora biti inicijalizovana.

---

# Creating Maps with make()

Najčešći način:

```go
users := make(map[string]int)
```

---

Sada:

```text
users

+

empty hash table
```

---

Možemo upisivati:

```go
users["Marko"] = 30
```

---

Rezultat:

```text
{
	Marko:30
}
```

---

# Map Literal

Map možemo odmah kreirati sa vrednostima.

---

Primer:

```go
users := map[string]int{

	"Marko": 30,
	"Ana":   25,

}
```

---

Rezultat:

```text
{
	Marko:30,
	Ana:25,
}
```

---

# Reading Values from Map

Pristup vrednosti:

```go
users["Marko"]
```

---

Primer:

```go
package main

import "fmt"

func main() {

	users := map[string]int{

		"Marko": 30,

	}

	fmt.Println(users["Marko"])

}
```

---

Rezultat:

```text
30
```

---

# Reading Non Existing Key

Šta se dešava ako ključ ne postoji?

Primer:

```go
users := map[string]int{}

value := users["Unknown"]

fmt.Println(value)
```

---

Rezultat:

```text
0
```

---

Go vraća:

```text
zero value
```

---

Problem:

Ne znamo da li:

* ključ ne postoji;
* vrednost je stvarno 0.

---

Zato postoji:

```text
comma ok idiom
```

---

# Comma OK Idiom

Map lookup vraća dve vrednosti:

```go
value, exists := map[key]
```

---

Primer:

```go
age, ok := users["Marko"]
```

---

Ako postoji:

```text
age = 30

ok = true
```

---

Ako ne postoji:

```text
age = 0

ok = false
```

---

Primer:

```go
package main

import "fmt"

func main() {

	users := map[string]int{

		"Marko": 30,

	}

	age, ok := users["Marko"]

	fmt.Println(age)
	fmt.Println(ok)

}
```

---

Rezultat:

```text
30
true
```

---

# Checking Key Existence

Idiomatic Go:

```go
if value, ok := users[key]; ok {

	fmt.Println(value)

}
```

---

Primer:

```go
age, exists := users["Ana"]

if exists {

	fmt.Println(age)

}
```

---

Ovo je standardni obrazac.

---

# Updating Map Values

Postojeća vrednost može biti promenjena.

---

Primer:

```go
users["Marko"] = 31
```

---

Pre:

```text
Marko -> 30
```

---

Posle:

```text
Marko -> 31
```

---

# Adding New Keys

Ako ključ ne postoji:

```go
users["Ana"] = 25
```

---

Go automatski dodaje novi entry.

---

Map:

Pre:

```text
{
	Marko:30
}
```

---

Posle:

```text
{
	Marko:30,
	Ana:25
}
```

---

# Deleting Elements

Za brisanje koristi se:

```go
delete()
```

---

Sintaksa:

```go
delete(map, key)
```

---

Primer:

```go
users := map[string]int{

	"Marko":30,
	"Ana":25,

}

delete(users,"Ana")
```

---

Rezultat:

```text
{
	Marko:30
}
```

---

# Delete Non Existing Key

Ovo je bezbedno:

```go
delete(users,"Unknown")
```

---

Neće izazvati:

```text
panic
```

---

Ako ključ ne postoji:

```text
nothing happens
```

---

# Map Length

Broj elemenata:

```go
len(map)
```

---

Primer:

```go
users := map[string]int{

	"Marko":30,
	"Ana":25,

}

fmt.Println(len(users))
```

---

Rezultat:

```text
2
```

---

# Iterating Over Maps

Za iteraciju koristimo:

```go
range
```

---

Primer:

```go
users := map[string]int{

	"Marko":30,
	"Ana":25,

}

for key, value := range users {

	fmt.Println(key,value)

}
```

---

Mogući rezultat:

```text
Marko 30
Ana 25
```

---

Ali:

```text
redosled nije garantovan
```

---

# Map Iteration Order

Važno:

Go namerno ne garantuje redosled.

---

Primer:

Prvo izvršavanje:

```text
Marko 30
Ana 25
```

---

Drugo:

```text
Ana 25
Marko 30
```

---

Oba su validna.

---

Zašto?

Zbog:

```text
hash table implementation
```

---

# Ignoring Keys or Values

Ako nam ključ nije potreban:

```go
for _, value := range users {

	fmt.Println(value)

}
```

---

Ako nam vrednost nije potrebna:

```go
for key := range users {

	fmt.Println(key)

}
```

---

# Map Copy Behavior

Map se ponaša drugačije od array-a.

---

Primer:

```go
a := map[string]int{

	"A":1,

}

b := a

b["A"] = 100
```

---

Rezultat:

```go
fmt.Println(a)
```

dobija:

```text
map[A:100]
```

---

Zašto?

Zato što:

```text
map header

pokazuje na istu internu strukturu
```

---

Slično slice-u.

---

# Map Internals

Interno map predstavlja:

```text
hmap
```

strukturu.

---

Pojednostavljeno:

```text
Map

+-------------+
| pointer     |
+-------------+

       |
       v

Hash Table

+-----+-----+
|bucket|bucket|
+-----+-----+
```

---

Svaki ključ prolazi kroz:

```text
hash function
```

koja određuje bucket.

---

# Comparable Keys

Ključevi moraju biti comparable.

Dozvoljeni:

```go
string
int
bool
array
struct
pointer
```

---

Nisu dozvoljeni:

```go
slice
map
function
```

---

Primer:

Ne može:

```go
map[[]int]string
```

---

Dobijamo compile error.

---

# Struct kao Map Key

Struct može biti ključ.

---

Primer:

```go
type Point struct {

	X int
	Y int

}

points := map[Point]string{}
```

---

Zašto?

Zato što je struct comparable ako su sva polja comparable.

---

# Map of Slices

Čest obrazac:

```go
map[string][]string
```

---

Primer:

```go
groups := map[string][]string{

	"developers":{
		"Marko",
		"Ana",
	},

}
```

---

Rezultat:

```text
developers

    |
    v

[Marko Ana]
```

---

# Map of Structs

Još jedan čest obrazac:

```go
map[int]User
```

---

Primer:

```go
users := map[int]User{

	1:{
		Name:"Marko",
	},

}
```

---

Koristi se često za:

* cache;
* registracije;
* konfiguracije.

---

# Concurrency Warning

Map nije thread-safe.

---

Ovo može izazvati problem:

```go
go write()

go read()
```

---

Istovremeni pristup zahteva:

* mutex;
* sync.Map;
* drugačiji dizajn.

---

Detaljno:

```text
Concurrency section
```

---

# Best Practices

---

## 1. Inicijalizuj map pre upisa

Dobro:

```go
users := make(map[string]int)
```

---

## 2. Koristi comma ok proveru

Dobro:

```go
value, ok := users[key]
```

---

## 3. Ne oslanjaj se na redosled

Map:

```text
unordered
```

---

## 4. Za konkurentni pristup koristi zaštitu

Ne:

```text
multiple goroutines + map
```

bez kontrole.

---

# Array vs Slice vs Map

| Osobina      | Array      | Slice      | Map             |
| ------------ | ---------- | ---------- | --------------- |
| Organizacija | indeks     | indeks     | ključ           |
| Veličina     | fiksna     | dinamička  | dinamička       |
| Redosled     | garantovan | garantovan | nije garantovan |
| Lookup       | O(1)       | O(1)       | O(1) prosečno   |
| Tip          | value      | descriptor | reference-like  |

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je map;
* key-value model;
* kreiranje map-a;
* `make()`;
* map literal;
* čitanje i upis;
* `comma ok` idiom;
* `delete()`;
* iteraciju;
* map internals;
* comparable keys.

---

# Zaključak

Map je osnovni alat za rad sa podacima koji zahtevaju brzo pronalaženje po ključu.

Najvažnije zapamtiti:

```text
Map

=

hash table

+

key-value pairs
```

Za razliku od slice-a:

```text
Slice -> index

Map -> key
```

Razumevanje map ponašanja je osnova za rad sa:

* cache strukturama;
* konfiguracijama;
* registrima;
* JSON podacima;
* backend aplikacijama.

U sledećem delu prelazimo na **Structs u Go-u: kreiranje sopstvenih tipova, field-ovi, metode, kompozicija i model podataka**.

---

# Structs in Go

U prethodnim delovima smo obradili:

- primitive tipove;
- array kolekcije;
- slice kolekcije;
- map kolekcije.

Sada prelazimo na jedan od najvažnijih koncepata u Go-u:

```text
Struct
````

---

Struct predstavlja:

> Korisnički definisan složeni tip koji grupiše više vrednosti različitih tipova u jednu celinu.

---

Primer:

```go
type User struct {

	Name string
	Age  int

}
```

---

Ovim smo definisali novi tip:

```text
User
```

koji sadrži:

```text
Name -> string

Age  -> int
```

---

Vizuelno:

```text
User

+----------------+
| Name string    |
+----------------+
| Age  int       |
+----------------+
```

---

# Zašto koristimo Structs?

Primitive tipovi predstavljaju pojedinačne vrednosti:

```go
name string
age int
```

---

Ali realni objekti imaju više osobina:

Primer:

```text
User

- name
- email
- age
- active status
```

---

Umesto:

```go
var name string
var email string
var age int
var active bool
```

koristimo:

```go
type User struct {

	Name   string
	Email  string
	Age    int
	Active bool

}
```

---

Struct omogućava:

* organizaciju podataka;
* kreiranje sopstvenih tipova;
* bolju čitljivost;
* povezivanje podataka i ponašanja.

---

# Defining Struct Types

Sintaksa:

```go
type TypeName struct {

	FieldName Type

}
```

---

Primer:

```go
type Product struct {

	Name  string
	Price float64

}
```

---

Dobijamo novi tip:

```text
Product
```

---

Koji ima:

```text
Name

Price
```

---

# Creating Struct Values

Postoji više načina.

---

## Zero Value Struct

Primer:

```go
type User struct {

	Name string
	Age int

}


var user User
```

---

Rezultat:

```text
Name: ""

Age: 0
```

---

Struct automatski dobija:

```text
zero value
```

za svako polje.

---

# Struct Literal

Najčešći način kreiranja.

---

Primer:

```go
user := User{

	Name: "Marko",
	Age: 30,

}
```

---

Rezultat:

```text
User

+-------------+
| Name: Marko |
| Age: 30     |
+-------------+
```

---

# Positional Struct Initialization

Moguće je inicijalizovati bez imena polja.

---

Primer:

```go
user := User{

	"Marko",
	30,

}
```

---

Ali ovaj stil ima nedostatke:

* zavisi od redosleda polja;
* manje je čitljiv;
* lako se pokvari nakon izmene struct-a.

---

Idiomatic Go:

```go
User{

	Name: "Marko",
	Age: 30,

}
```

---

# Accessing Struct Fields

Poljima pristupamo pomoću:

```text
.
```

operatora.

---

Primer:

```go
fmt.Println(user.Name)
```

---

Rezultat:

```text
Marko
```

---

Menjanje vrednosti:

```go
user.Age = 31
```

---

Rezultat:

```text
Age:

30

↓

31
```

---

# Nested Structs

Struct može sadržati druge struct-ove.

---

Primer:

```go
type Address struct {

	City string
	Country string

}


type User struct {

	Name string
	Address Address

}
```

---

Korišćenje:

```go
user := User{

	Name:"Marko",

	Address: Address{

		City:"Nis",
		Country:"Serbia",

	},

}
```

---

Pristup:

```go
user.Address.City
```

---

Rezultat:

```text
Nis
```

---

# Anonymous Structs

Go dozvoljava struct bez imenovanog tipa.

---

Primer:

```go
user := struct {

	Name string
	Age int

}{

	Name:"Marko",
	Age:30,

}
```

---

Koristi se kada:

* tip neće biti ponovo korišćen;
* podaci su lokalni.

---

Primer upotrebe:

```text
configuration objects

temporary data

tests
```

---

# Struct Comparison

Struct može biti poređen ako su sva polja comparable.

---

Primer:

```go
type Point struct {

	X int
	Y int

}
```

---

Možemo:

```go
p1 := Point{1,2}

p2 := Point{1,2}

fmt.Println(p1 == p2)
```

---

Rezultat:

```text
true
```

---

Ali:

```go
type User struct {

	Name string
	Tags []string

}
```

---

Ne može:

```go
user1 == user2
```

---

Zašto?

Zato što:

```text
slice nije comparable
```

---

# Struct Copy Behavior

Struct je:

```text
value type
```

---

Primer:

```go
type User struct {

	Name string

}


a := User{

	Name:"Marko",

}


b := a

b.Name = "Ana"
```

---

Rezultat:

```go
fmt.Println(a.Name)
```

dobija:

```text
Marko
```

---

Zašto?

Zato što:

```text
b

je kopija struct-a
```

---

Memorijski model:

```text
a

+--------+
| Marko  |
+--------+


b

+--------+
| Ana    |
+--------+
```

---

# Struct with Pointer Fields

Struct može sadržati pointer.

---

Primer:

```go
type User struct {

	Name string
	Age  *int

}
```

---

Sada:

```text
Age

nije int vrednost

nego adresa
```

---

Primer:

```go
age := 30

user := User{

	Name:"Marko",
	Age:&age,

}
```

---

Pristup:

```go
fmt.Println(*user.Age)
```

---

Rezultat:

```text
30
```

---

# Struct Memory Layout

Struct se nalazi u memoriji kao kolekcija field-ova.

---

Primer:

```go
type User struct {

	ID int32
	Active bool

}
```

---

Memorijski raspored:

```text
+---------+
| ID      |
+---------+
| Active  |
+---------+
```

---

Ali realni raspored zavisi od:

* alignment pravila;
* padding-a;
* arhitekture sistema.

---

Ovo ćemo detaljno obraditi u:

```text
Memory Allocation & Data Alignment
```

---

# Struct Padding Example

Primer:

```go
type Bad struct {

	A bool
	B int64

}
```

---

Može imati padding:

```text
+------+
| bool |
+------+
| pad  |
+------+
| int64|
+------+
```

---

Bolji raspored:

```go
type Good struct {

	B int64
	A bool

}
```

---

Struct layout je efikasniji.

---

# Structs as Function Arguments

Pošto je struct value type:

```go
func process(user User)
```

dobija kopiju.

---

Primer:

```go
func change(user User){

	user.Name = "Ana"

}
```

---

Original ostaje isti.

---

Ako želimo menjanje:

```go
func change(user *User)
```

---

Poziv:

```go
change(&user)
```

---

# Structs and Methods

Struct može imati metode.

---

Primer:

```go
type User struct {

	Name string

}


func (u User) SayHello(){

	fmt.Println(u.Name)

}
```

---

Poziv:

```go
user.SayHello()
```

---

Ovo uvodi koncept:

```text
methods

+

receiver
```

---

Detaljno:

```text
Creating Functions and Methods
```

poglavlje.

---

# Empty Struct

Go ima specijalan slučaj:

```go
struct{}
```

---

Nema polja.

---

Primer:

```go
var signal struct{}
```

---

Veličina:

```text
0 bytes
```

---

Često se koristi za:

* signalizaciju;
* set implementacije;
* channels.

---

Primer:

```go
map[string]struct{}
```

predstavlja:

```text
set
```

---

# Struct vs Class

Go nema klase.

Ali struct + methods daju sličan koncept.

---

Java/C#:

```text
class User

{
	fields
	methods
}
```

---

Go:

```go
type User struct {

	fields

}


func (u User) Method(){

}
```

---

Razlika:

Go nema:

* inheritance;
* class hierarchy;
* constructors.

---

Koristi:

* composition;
* interfaces.

---

# Best Practices

---

## 1. Koristi imenovana polja

Dobro:

```go
User{

	Name:"Marko",

}
```

---

## 2. Dizajniraj male struct-ove

Veliki struct-ovi otežavaju:

* testiranje;
* održavanje;
* razumevanje.

---

## 3. Razmisli o zero value ponašanju

Dobar struct:

```go
var counter Counter
```

treba biti smislen.

---

## 4. Koristi pointer receiver kada menjaš stanje

Primer:

```go
func (u *User) Update()
```

---

# Struct vs Array vs Slice vs Map

| Tip    | Svrha                          |
| ------ | ------------------------------ |
| Array  | Fiksan broj istih elemenata    |
| Slice  | Dinamička lista elemenata      |
| Map    | Key-value kolekcija            |
| Struct | Grupisanje različitih podataka |

---

# Ključne ideje ovog dela

Nakon ovog dela treba razumeti:

* šta je struct;
* kreiranje sopstvenih tipova;
* struct fields;
* zero value;
* struct literal;
* pristup poljima;
* nested struct;
* anonymous struct;
* struct comparison;
* value semantics;
* pointer fields;
* metode.

---

# Zaključak

Struct je osnovni gradivni blok Go programa.

Većina kompleksnih sistema u Go-u sastoji se od:

```text
Structs

+

Methods

+

Interfaces
```

Struct omogućava modelovanje realnih entiteta i predstavlja osnovu za:

* poslovne modele;
* API objekte;
* konfiguracije;
* baze podataka;
* kompleksne sisteme.

U sledećem delu završavamo poglavlje **03 Working with Primitive Data Types** i obrađujemo pregled svih kolekcija i poređenje njihovih karakteristika.

---

# Primitive Data Types - Final Summary

U prethodnih 17 delova ovog poglavlja obradili smo kompletan osnovni sistem tipova podataka u Go-u.

Naučili smo:

- kako Go predstavlja vrednosti u memoriji;
- kako funkcionišu primitive vrednosti;
- kako rade pointer-i;
- kako se koriste konstante;
- kako rade kolekcije;
- kako se razlikuju array, slice, map i struct tipovi.

Ovo poglavlje predstavlja osnovu za sva naredna poglavlja Go jezika.

---

# Go Type System Overview

Go je:

```text
statically typed language
````

što znači:

* svaki izraz ima poznat tip;
* compiler proverava tipove;
* mnoge greške se otkrivaju pre pokretanja programa.

---

Primer:

```go
var age int = 30
```

Compiler zna:

```text
age

=

integer value
```

---

Za razliku od dinamičkih jezika:

```python
age = 30
```

tip se određuje tokom izvršavanja.

---

# Primitive Types Overview

Go osnovni tipovi:

```text
bool

string

integer types

floating point types

complex types
```

---

## Boolean

Tip:

```go
bool
```

Vrednosti:

```text
true

false
```

---

Primer:

```go
var enabled bool = true
```

---

Zero value:

```text
false
```

---

# Integer Types

Go ima više integer tipova:

```text
int

int8

int16

int32

int64
```

---

Unsigned:

```text
uint

uint8

uint16

uint32

uint64
```

---

Posebni:

```text
byte

rune
```

---

Alias:

```go
byte = uint8

rune = int32
```

---

# Floating Point Types

Go podržava:

```text
float32

float64
```

---

Primer:

```go
var price float64 = 99.99
```

---

Najčešće:

```text
float64
```

---

# Complex Types

Go podržava:

```text
complex64

complex128
```

---

Primer:

```go
z := complex(2,3)
```

---

Rezultat:

```text
2 + 3i
```

---

# String Type

String predstavlja:

```text
immutable sequence of bytes
```

---

Primer:

```go
name := "Go"
```

---

String:

* nije slice;
* ne može direktno da se menja;
* koristi UTF-8 encoding.

---

# Constants

Konstante predstavljaju vrednosti koje se ne menjaju.

---

Primer:

```go
const Pi = 3.14
```

---

Karakteristike:

* poznate u compile time-u;
* nemaju memorijsku adresu;
* mogu biti untyped.

---

Primer:

```go
const x = 10
```

---

Može postati:

```go
int

float64

complex128
```

zavisno od konteksta.

---

# Variables

Promenljive čuvaju stanje programa.

---

Deklaracija:

```go
var age int
```

---

Kratka deklaracija:

```go
age := 30
```

---

Pravilo:

```text
:=

samo unutar funkcija
```

---

# Zero Values

Jedna od najvažnijih Go karakteristika:

```text
Every variable has a valid zero value.
```

---

Primeri:

| Tip     | Zero Value |
| ------- | ---------- |
| int     | 0          |
| float   | 0.0        |
| bool    | false      |
| string  | ""         |
| pointer | nil        |

---

Za kolekcije:

| Tip     | Zero Value                |
| ------- | ------------------------- |
| array   | array sa zero vrednostima |
| slice   | nil                       |
| map     | nil                       |
| pointer | nil                       |

---

# Type Conversion

Go ne radi implicitnu konverziju.

---

Nije dozvoljeno:

```go
var x int = 10

var y int64 = x
```

---

Potrebno:

```go
var y int64 = int64(x)
```

---

Ovo daje:

* veću sigurnost;
* manje neočekivanih grešaka.

---

# Pointers

Pointer čuva memorijsku adresu.

---

Primer:

```go
age := 30

ptr := &age
```

---

Model:

```text
ptr

 |

 v

age
```

---

Dereferenciranje:

```go
*ptr
```

---

Rezultat:

```text
30
```

---

Pointer se koristi kada:

* želimo menjati original;
* želimo izbeći kopiranje velikih struktura;
* želimo predstaviti odsustvo vrednosti (`nil`).

---

# Collections Overview

Go ima četiri glavna kolekciona tipa:

```text
Array

Slice

Map

Struct
```

---

# Array

Array:

```text
fixed-size collection
```

---

Primer:

```go
var numbers [5]int
```

---

Karakteristike:

* veličina deo tipa;
* value type;
* kopira elemente;
* koristi kontinuiranu memoriju.

---

Primer:

```text
[3]int

nije isto što i

[4]int
```

---

# Slice

Slice:

```text
dynamic collection
```

---

Primer:

```go
numbers := []int{1,2,3}
```

---

Interno:

```text
pointer

length

capacity
```

---

Najčešće korišćena kolekcija u Go aplikacijama.

---

Važne funkcije:

```go
append()

copy()

len()

cap()
```

---

# Slice Memory Model

Slice ne čuva podatke direktno.

---

On pokazuje na:

```text
underlying array
```

---

Zbog toga:

```go
b := a
```

ne kopira elemente.

---

Oba slice-a dele:

```text
same array
```

---

# Map

Map:

```text
key-value collection
```

---

Primer:

```go
ages := map[string]int{

	"Marko":30,

}
```

---

Karakteristike:

* hash tabela;
* brz lookup;
* nema garantovan redosled.

---

Pristup:

```go
value, ok := map[key]
```

---

Brisanje:

```go
delete(map,key)
```

---

# Struct

Struct:

```text
custom data type
```

---

Primer:

```go
type User struct {

	Name string
	Age int

}
```

---

Koristi se za:

* modelovanje podataka;
* poslovne objekte;
* konfiguracije;
* API strukture.

---

# Value Types vs Reference-Like Types

Veoma važna Go razlika.

---

## Value Types

Kopiraju podatke.

Primeri:

```text
int

float

bool

array

struct
```

---

Primer:

```go
b := a
```

dobija:

```text
nezavisnu kopiju
```

---

## Reference-Like Types

Deli internu strukturu.

Primeri:

```text
slice

map

channel

pointer
```

---

Primer:

```go
b := a
```

može deliti podatke.

---

# Comparison Table

| Tip    | Veličina    | Value Type | Dinamički |
| ------ | ----------- | ---------- | --------- |
| Array  | fiksna      | da         | ne        |
| Slice  | promenljiva | ne         | da        |
| Map    | promenljiva | ne         | da        |
| Struct | zavisi      | da         | ne        |

---

# Kada koristiti koji tip?

---

## Koristi Array kada:

* veličina je poznata;
* treba striktna struktura;
* potrebna je kontrola memorije.

Primer:

```go
var rgb [3]byte
```

---

## Koristi Slice kada:

* lista raste;
* obrađuješ kolekcije;
* radiš sa podacima.

Primer:

```go
users []User
```

---

## Koristi Map kada:

* tražiš podatke po ključu.

Primer:

```go
usersByID map[int]User
```

---

## Koristi Struct kada:

* modeluješ entitet.

Primer:

```go
type User struct {

	ID int
	Name string

}
```

---

# Najčešće početničke greške

---

## 1. Mešanje Array-a i Slice-a

Pogrešno razmišljanje:

```text
array = slice
```

---

Nisu isti.

---

## 2. Zaboravljanje da Slice deli memoriju

Primer:

```go
b := a
```

nije:

```text
deep copy
```

---

## 3. Pisanje u nil map

Pogrešno:

```go
var m map[string]int

m["a"] = 1
```

---

Ispravno:

```go
m := make(map[string]int)
```

---

## 4. Neproveravanje nil vrednosti

Posebno kod:

```text
pointer

map

slice

interface

channel
```

---

# Mental Model za Go Tipove

Kada vidiš tip, postavi pitanja:

---

## 1. Da li se kopira vrednost?

Primer:

```text
array

struct
```

---

## 2. Da li se deli memorija?

Primer:

```text
slice

map
```

---

## 3. Da li postoji zero value?

Primer:

```text
int -> 0

slice -> nil
```

---

## 4. Da li je mutable?

Primer:

```text
string -> ne

slice -> da
```

---

# Šta smo naučili?

Nakon ovog poglavlja razumemo:

✅ primitive tipove
✅ promenljive
✅ konstante
✅ type conversion
✅ pointer-e
✅ zero values
✅ array-e
✅ slice-ove
✅ map-e
✅ struct-ove
✅ value/reference ponašanje

---

# Sledeće Poglavlje

Sledeća oblast:

```text
04 - Creating Functions and Methods
```

će obraditi:

* deklaraciju funkcija;
* parametre;
* povratne vrednosti;
* multiple return values;
* named returns;
* variadic funkcije;
* metode;
* receiver-e;
* function values;
* higher-order funkcije.

---

# Finalna Poruka

Razumevanje tipova podataka predstavlja osnovu Go programiranja.

Go filozofija se može sažeti:

```text
Simple types

+

Explicit behavior

+

Predictable memory model

=

Reliable programs
```

Ako pravilno razumeš:

```text
array

slice

map

struct

pointer
```

imaćeš čvrstu osnovu za:

* concurrency;
* interfaces;
* memory management;
* performance optimization;
* idiomatic Go development.

---

# Sledeće poglavlje

Sledeće poglavlje:

```text
04. Working with Collections
```