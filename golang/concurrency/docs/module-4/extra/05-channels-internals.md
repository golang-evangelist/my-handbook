# Channels Internals

> Module: #4 — Advanced Go Concurrency
> 
> Section: Extras
> 
> Topic: Channels Internals
> 
> Level: Advanced / Senior

---

# 📚 Sadržaj

- Uvod u Go Channels internals
- Channel kao runtime objekat
- Zašto su channels posebni
- Osnovni channel model
- Unbuffered vs Buffered channel
- Channel lifecycle
- `make(chan T)` iza scene
- Runtime struktura `hchan`
- Channel memorijski model
- Send/Receive operacije uvod

---

# 1. Uvod u Go Channels Internals

Go channels su jedna od najvažnijih concurrency primitive u jeziku.

---

Na površini:

```go
ch := make(chan int)
```

izgleda jednostavno.

---

Ali iza toga runtime kreira kompleksnu strukturu koja upravlja:

- memorijom
- goroutines
- scheduler interakcijom
- synchronization
- waiting queue strukturama

---

Channel nije samo:

```
pipe za podatke
```

---

Channel je:

```
concurrency synchronization mechanism
```

---

# 2. Channel kao Runtime Objekat

Kada napišemo:

```go
ch := make(chan int)
```

Go ne pravi običnu promenljivu.

---

Kreira runtime objekat:

```
channel instance

        |
        |
        v

runtime.hchan
```

---

Ovaj objekat sadrži:

- buffer informacije
- element type
- send queue
- receive queue
- lock
- state informacije

---

# 3. Zašto su Channels Posebni?

Channel kombinuje dve stvari:

---

## 3.1 Communication

Prenos podataka:

```go
sender -> value -> receiver
```

---

## 3.2 Synchronization

Kontrola izvršavanja:

```
čekaj dok receiver nije spreman
```

---

Zato channel rešava:

```
data transfer

+

coordination
```

---

# 4. Osnovni Channel Model

Najjednostavniji model:

```
Goroutine A

     |
     |
     v

  Channel

     |
     |
     v

Goroutine B
```

---

Sender:

```go
ch <- value
```

---

Receiver:

```go
value := <-ch
```

---

Channel je posrednik.

---

# 5. Unbuffered Channel

Primer:

```go
ch := make(chan int)
```

---

Buffer:

```
capacity = 0
```

---

Nema prostora za čuvanje vrednosti.

---

Send:

```go
ch <- 10
```

---

Sender mora čekati:

```
receiver spreman
```

---

Model:

```
Sender

   |
   |
 handshake

   |
   |

Receiver
```

---

# 6. Buffered Channel

Primer:

```go
ch :=
make(chan int, 3)
```

---

Buffer:

```
capacity = 3
```

---

Može sadržati:

```
10
20
30
```

---

Sender:

```go
ch <- 40
```

---

Čeka tek kada:

```
buffer full
```

---

# 7. Unbuffered vs Buffered

## Unbuffered

```
send

↓

receiver
```

---

Karakteristike:

- direktna sinhronizacija
- backpressure
- stroga koordinacija

---

## Buffered

```
send

↓

buffer

↓

receiver
```

---

Karakteristike:

- asinhroniji prenos
- queue behavior
- veća fleksibilnost

---

# 8. Channel Lifecycle

Channel ima tri glavna stanja:

```
Open

Closed

Garbage Collected
```

---

## Open

Normalan rad:

```go
ch <- value

<-ch
```

---

## Closed

```go
close(ch)
```

---

Više nema send operacija.

---

## Garbage Collected

Kada nema referenci:

```
channel object
```

se uklanja.

---

# 9. Channel Close Pravila

Close radi:

```go
close(ch)
```

---

Posledice:

Receiver:

```go
value, ok := <-ch
```

dobija:

```go
ok == false
```

---

Send:

```go
ch <- value
```

posle close:

```
panic
```

---

# 10. Ko Treba da Zatvori Channel?

Pravilo:

> Sender zatvara channel.

---

Primer:

```
Producer

   |
   |
 close()

   |
   v

Consumer
```

---

Receiver obično:

ne zatvara channel.

---

# 11. Channel Allocation

Kada pozovemo:

```go
make(chan int, 10)
```

---

Runtime radi:

1. alocira `hchan`

2. kreira buffer

3. postavlja element type

4. inicijalizuje queue strukture

5. vraća pointer

---

Mentalni model:

```
make(chan)

       |

       v

runtime.hchan

       |

       v

channel value
```

---

# 12. Runtime hchan Struktura

Pojednostavljen prikaz:

```go
type hchan struct {

	qcount uint

	dataqsiz uint

	buf unsafe.Pointer

	sendx uint

	recvx uint

	recvq waitq

	sendq waitq

	lock mutex

}
```

---

Napomena:

Ovo nije javni API.

To je runtime interna struktura.

---

# 13. hchan Polja

## qcount

Broj elemenata u bufferu.

Primer:

```
buffer:

[10][20]

qcount = 2
```

---

## dataqsiz

Kapacitet buffera.

Primer:

```go
make(chan int, 5)
```

---

Rezultat:

```
dataqsiz = 5
```

---

## buf

Pokazuje na:

```
buffer memoriju
```

---

# 14. Send Queue i Receive Queue

Channel mora pratiti:

ko čeka.

---

Ako nema receiver-a:

sender čeka.

---

Ako nema sender-a:

receiver čeka.

---

Zato postoje:

```
sendq

recvq
```

---

# 15. Goroutine Waiting

Kada goroutine ne može nastaviti:

ona ne vrti:

```
busy loop
```

---

Scheduler je uspava.

---

Primer:

```go
ch <- value
```

bez receiver-a.

---

Runtime:

```
Goroutine

↓

waiting queue

↓

park
```

---

# 16. Channel i Scheduler

Channel direktno sarađuje sa scheduler-om.

---

Operacija:

```go
ch <- value
```

može:

1. nastaviti odmah

2. blokirati goroutine

3. probuditi drugu goroutine

---

# 17. Memory Model

Channel ima važnu osobinu:

```
send happens-before receive
```

---

Primer:

```go
x = 100

ch <- true
```

---

Druga goroutine:

```go
<-ch

print(x)
```

---

Garantovano vidi:

```
x == 100
```

---

# 18. Zašto su Channels Powerful?

Zato što kombinuju:

```
queue

+

lock

+

scheduler integration

+

memory synchronization
```

---

Jedna primitive rešava više problema.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ channel kao runtime objekat

✅ hchan koncept

✅ unbuffered channels

✅ buffered channels

✅ lifecycle

✅ close pravila

✅ send/receive queue

✅ scheduler interakciju

✅ memory model

---

# Channels Internals

## Deo #2 — Channel Send Internals

---

# 📚 Sadržaj

- Šta se dešava kod `ch <- value`
- Channel send algoritam
- Send fast path
- Unbuffered channel send
- Buffered channel send
- Direct handoff mehanizam
- `sudog` struktura
- Goroutine parking
- Runtime send tok
- Scheduler interakcija

---

# 1. Šta se Dešava Kod `ch <- value`?

Kada napišemo:

```go
ch <- value
```

izgleda jednostavno.

---

Ali runtime mora odlučiti:

```
Da li postoji receiver?

Da li postoji slobodan buffer?

Da li sender mora čekati?
```

---

Mentalni model:

```
SEND

 |

 v

Channel State

 |

 +----------------+
 | Receiver čeka? |
 +----------------+

        |
        |

    direktan transfer


ili


 +----------------+
 | Buffer prostor |
 +----------------+

        |
        |

    upis u buffer


ili


        |

    block sender
```

---

# 2. Channel Send Algoritam

Pojednostavljeno:

```text
send(channel, value)

        |

        v

Acquire channel lock

        |

        v

Postoji waiting receiver?

        |

   +----+----+

   |         |

  DA        NE

   |         |

handoff   buffer?

             |

        +----+----+

        |         |

       DA        NE

        |         |

 write buffer   park goroutine
```

---

# 3. Prvi Korak — Zaključavanje Channela

Channel ima internu zaštitu:

```go
lock mutex
```

---

Kod send operacije:

runtime prvo sinhronizuje pristup.

---

Zašto?

Jer više goroutines mogu istovremeno:

```go
ch <- value
```

---

Primer:

```
G1 ----\
        \
G2 -----> channel
        /
G3 ----/
```

---

Potrebna je konzistentna promena:

- buffer-a
- queue-a
- counters

---

# 4. Send Fast Path

Fast path znači:

```
operacija može odmah da završi
```

---

Primer:

Buffered channel:

```go
ch := make(chan int, 10)

ch <- 5
```

---

Ako postoji prostor:

```
buffer nije pun
```

---

Runtime samo:

```
upiše vrednost

nastavi izvršavanje
```

---

Nema:

- parking
- scheduler switch
- queue manipulacije

---

# 5. Buffered Channel Send

Primer:

```go
ch := make(chan int, 3)
```

---

Početno:

```
buffer:

[ ][ ][ ]

qcount = 0
```

---

Send:

```go
ch <- 10
```

---

Stanje:

```
buffer:

[10][ ][ ]

qcount = 1
```

---

---

Drugi send:

```go
ch <- 20
```

---

Rezultat:

```
buffer:

[10][20][ ]

qcount = 2
```

---

# 6. Channel Ring Buffer

Buffered channel koristi circular buffer.

---

Primer:

Kapacitet:

```
4
```

---

Memorija:

```
+----+----+----+----+
| A  | B  | C  | D  |
+----+----+----+----+

 ^
 |
recvx

 ^
 |
sendx
```

---

Indeksi kruže.

---

Kada dođu do kraja:

vraćaju se na početak.

---

# 7. Send Index (`sendx`)

Channel čuva:

```
gde sledeći element ide
```

---

Primer:

```
sendx = 2
```

---

Sledeći send:

```
buffer[2] = value
```

---

Posle:

```
sendx++
```

---

# 8. Kada Buffer Nije Dovoljan

Primer:

```go
ch := make(chan int, 1)
```

---

Buffer:

```
[10]
```

---

Novi send:

```go
ch <- 20
```

---

Problem:

```
buffer full
```

---

Runtime mora odlučiti:

```
čekati
```

---

# 9. Unbuffered Channel Send

Kod:

```go
ch := make(chan int)

ch <- value
```

---

Nema:

```
buffer
```

---

Zato runtime proverava:

```
da li receiver čeka?
```

---

Ako postoji:

```
direktan transfer
```

---

# 10. Direct Handoff

Najvažniji koncept kod unbuffered channel-a.

---

Primer:

Goroutine A:

```go
ch <- 100
```

---

Goroutine B:

```go
value := <-ch
```

---

Nema međukopiranja u buffer.

---

Tok:

```
Goroutine A

value

 |

 |

 v

Goroutine B
```

---

Direktno.

---

# 11. Receiver Queue Provera

Channel ima:

```
recvq
```

---

Runtime proverava:

```
da li neko čeka receive?
```

---

Ako:

```
recvq != empty
```

---

uzima waiting receiver-a.

---

---

# 12. `sudog` Struktura

Go runtime ne stavlja direktno goroutine u queue.

---

Koristi:

```
sudog
```

---

`sudog` predstavlja:

```
waiting relationship

goroutine <-> channel
```

---

Pojednostavljeno:

```go
type sudog struct {

	g *g

	c *hchan

	elem unsafe.Pointer

	next *sudog

}
```

---

Napomena:

Interna runtime struktura.

---

# 13. Zašto Postoji sudog?

Jedna goroutine može čekati različite stvari:

- channel
- select
- synchronization primitive

---

Runtime treba objekat koji opisuje:

```
čekanje
```

---

`sudog` je upravo to.

---

# 14. Send sa Waiting Receiver-om

Scenario:

```
recvq:

G2
```

---

G1 radi:

```go
ch <- value
```

---

Runtime:

1. pronalazi G2

2. kopira value

3. uklanja G2 iz recvq

4. budi G2

---

Rezultat:

```
G1 nastavlja

G2 nastavlja
```

---

# 15. Kada Sender Mora da Čeka?

Primer:

```go
ch := make(chan int, 0)
```

---

Nema receiver-a.

---

Sender:

```go
ch <- 10
```

---

Runtime:

```
nema gde staviti vrednost
```

---

Zato:

```
park goroutine
```

---

# 16. Goroutine Parking

Parking znači:

```
goroutine se suspenduje
```

---

Ne znači:

```
CPU loop
```

---

Scheduler:

```
G

↓

waiting

↓

sleep
```

---

CPU može raditi druge stvari.

---

# 17. Send Queue

Ako sender čeka:

dodaje se u:

```
sendq
```

---

Primer:

```
sendq:

G1
G2
G3
```

---

Kada receiver postane dostupan:

runtime uzima jednog.

---

# 18. Scheduler Buđenje

Kada receiver primi vrednost:

runtime mora:

```
wake goroutine
```

---

Tok:

```
Waiting G

      |

      v

Runnable G

      |

      v

Scheduler

      |

      v

Running
```

---

# 19. Send Operacija — Kompletan Tok

```
ch <- value


Lock channel


Check receiver


      |

      +--> receiver postoji

      |        |
      |        v
      |    direct handoff
      |
      |
      +--> nema receiver

               |

               v

          buffer space?

               |

          +----+----+

          |         |

         yes        no

          |         |

     write buffer   park sender
```

---

# 20. Performance ImplIkacije

Send može biti:

---

## Jeftin

```
buffer available
```

---

## Srednji trošak

```
direct handoff
```

---

## Skup

```
parking + wakeup
```

---

# 21. Senior Pravilo

Channel nije samo:

```
FIFO queue
```

---

On je:

```
queue

+

scheduler coordination

+

memory synchronization
```

---

Zato je:

```
ch <- value
```

mnogo kompleksnija operacija nego što izgleda.

---

# 📋 Rezime

U ovom delu naučili smo:

✅ channel send tok

✅ send fast path

✅ buffered send

✅ ring buffer koncept

✅ sendx

✅ unbuffered send

✅ direct handoff

✅ sudog koncept

✅ goroutine parking

✅ scheduler wakeup

---

# Channels Internals

## Deo #3 — Channel Receive Internals

---

# 📚 Sadržaj

- Šta se dešava kod `<-ch`
- Channel receive algoritam
- Receive fast path
- Buffered channel receive
- Unbuffered channel receive
- Sender handoff
- `recvq` mehanizam
- Goroutine parking
- Closed channel behavior
- Zero value receive
- Runtime receive tok

---

# 1. Šta se Dešava Kod `<-ch`?

Kod:

```go
value := <-ch
```

izgleda kao običan read.

---

Ali runtime mora proveriti:

```
Da li postoji vrednost?

Da li postoji sender?

Da li je channel zatvoren?

Da li receiver mora čekati?
```

---

Mentalni model:

```
RECEIVE

    |

    v

Channel State

    |

    +----------------+
    | Buffer ima data|
    +----------------+

          |

          v

      read buffer


ili


    +----------------+
    | Sender čeka?   |
    +----------------+

          |

          v

     direct receive


ili


          |

          v

    park receiver
```

---

# 2. Channel Receive Algoritam

Pojednostavljeno:

```text
receive(channel)

        |

        v

Acquire channel lock

        |

        v

Buffer contains value?

        |

   +----+----+

   |         |

  DA        NE

   |         |

read      sender?

buffer       |

             |

        +----+----+

        |         |

       DA        NE

        |         |

handoff   park receiver
```

---

# 3. Prvi Korak — Channel Lock

Receive operacija prvo sinhronizuje pristup.

---

Zašto?

Jer drugi goroutine može istovremeno:

```go
ch <- value
```

---

Primer:

```
G1

<-ch


G2

ch <- 10
```

---

Runtime mora garantovati:

```
jedinstveno stanje kanala
```

---

# 4. Receive Fast Path

Fast path znači:

```
vrednost je odmah dostupna
```

---

Primer:

```go
ch := make(chan int, 10)

ch <- 100

value := <-ch
```

---

Buffer:

```
[100][ ][ ]
```

---

Receive:

```
uzima 100
```

---

Nema:

- parking
- scheduler switch
- waiting queue

---

# 5. Buffered Channel Receive

Primer:

```go
ch := make(chan int, 3)
```

---

Stanje:

```
buffer:

[10][20][ ]

qcount = 2
```

---

Receive:

```go
x := <-ch
```

---

Rezultat:

```
x = 10
```

---

Novo stanje:

```
buffer:

[ ][20][ ]

qcount = 1
```

---

# 6. Ring Buffer Read

Buffered channel koristi circular buffer.

---

Postoji:

```
recvx
```

---

Primer:

```
buffer:

[10][20][30][40]

 ^
 |
recvx
```

---

Receive uzima:

```
buffer[recvx]
```

---

Posle:

```
recvx++
```

---

# 7. Receive Index (`recvx`)

`recvx` označava:

```
sledeću poziciju za čitanje
```

---

Primer:

```
recvx = 2
```

---

Receive:

```text
read buffer[2]
```

---

Posle:

```
recvx = 3
```

---

# 8. Kada Buffer Nema Vrednost?

Primer:

```go
ch := make(chan int, 1)
```

---

Početno:

```
buffer:

[ ]
```

---

Receive:

```go
x := <-ch
```

---

Nema podatka.

---

Runtime proverava:

```
da li postoji sender?
```

---

# 9. Unbuffered Channel Receive

Kod:

```go
ch := make(chan int)
```

---

Nema:

```
buffer
```

---

Receive mora pronaći:

```
waiting sender
```

---

Ako postoji:

```
direct handoff
```

---

# 10. Sender Handoff

Scenario:

Goroutine A:

```go
ch <- 100
```

---

Goroutine B:

```go
x := <-ch
```

---

Runtime:

1. pronalazi sender-a

2. uzima vrednost

3. budi sender-a

4. nastavlja receiver

---

Tok:

```
Sender

value

   |

   v

Receiver
```

---

# 11. Send Queue (`sendq`)

Channel održava:

```
sendq
```

---

Ako sender čeka:

```
sendq:

G1
G2
G3
```

---

Receive može preuzeti prvog.

---

# 12. Receive Queue (`recvq`)

Ako receiver čeka:

```
recvq:

G4
G5
```

---

To znači:

goroutines koje čekaju:

```go
<-ch
```

---

Kada send dođe:

jedan receiver se budi.

---

# 13. Kada Receiver Mora da Čeka?

Primer:

```go
ch := make(chan int)
```

---

Nema:

- buffer
- sender

---

Receive:

```go
<-ch
```

---

Runtime:

```
nema podatka
```

---

Zato:

```
park receiver
```

---

# 14. Receiver Parking

Tok:

```
Goroutine

    |

    v

recvq

    |

    v

parked state
```

---

Scheduler uklanja goroutine iz aktivnog rada.

---

CPU ne troši vreme.

---

# 15. Receive i Scheduler

Kada sender pošalje:

```go
ch <- value
```

---

Runtime:

```
pronalazi čekajući receiver
```

---

Zatim:

```
wake receiver
```

---

Tok:

```
Waiting

↓

Runnable

↓

Running
```

---

# 16. Closed Channel Receive

Primer:

```go
close(ch)
```

---

Posle:

```go
value := <-ch
```

---

Ako buffer ima vrednosti:

dobijaju se prvo.

---

Primer:

```
buffer:

[10][20]
```

---

Close:

```
channel closed
```

---

Receive:

```
10

20

0
```

---

# 17. Zero Value Receive

Kada je channel:

```
closed

+

empty
```

---

Receive:

```go
value := <-ch
```

---

Dobija:

```
zero value
```

---

Primer:

```go
ch := make(chan int)

close(ch)

x := <-ch
```

---

Rezultat:

```go
x == 0
```

---

# 18. Receive sa `ok` Vrednošću

Bolji način:

```go
value, ok := <-ch
```

---

Otvoren channel:

```go
ok == true
```

---

Closed + empty:

```go
ok == false
```

---

Primer:

```go
for {

	value, ok := <-ch

	if !ok {

		break

	}

	process(value)

}
```

---

# 19. Channel Receive Tok

Kompletan model:

```
<-ch


Lock channel


Buffer data?


      |

      +--> yes

      |       |

      |   read buffer

      |
      |
      +--> no

              |

          sender waiting?


              |

        +-----+-----+

        |           |

       yes          no

        |           |

 direct receive   park receiver
```

---

# 20. Performance Aspekti

Receive može biti:

---

## Najbrži

```
buffer read
```

---

## Srednji

```
direct handoff
```

---

## Najskuplji

```
parking + wakeup
```

---

# 21. Receive vs Mutex Read

Channel receive:

radi:

```
data transfer

+

synchronization
```

---

Mutex read:

radi:

```
state protection
```

---

Različiti problemi.

---

# 22. Senior Pravilo

Channel receive nije:

```
običan memory read
```

---

To je:

```
runtime synchronization operation
```

---

Zato treba razumeti:

- scheduler
- queues
- parking
- memory ordering

---

# 📋 Rezime

U ovom delu naučili smo:

✅ receive algoritam

✅ receive fast path

✅ buffered receive

✅ recvx

✅ unbuffered receive

✅ sender handoff

✅ sendq / recvq

✅ goroutine parking

✅ closed channel behavior

✅ zero value receive

---

# Channels Internals

## Deo #4 — Select Internals

---

# 📚 Sadržaj

- Uvod u `select` runtime mehanizam
- Kako `select` radi iza scene
- Select kao channel multiplexer
- Runtime `selectgo`
- Case registracija
- Sudog lista kod select-a
- Polling faza
- Blocking faza
- Buđenje goroutine
- Randomizacija case redosleda
- Performance implikacije

---

# 1. Uvod u Select Internals

Go `select` omogućava čekanje na više channel operacija.

---

Primer:

```go
select {

case value := <-ch1:

	fmt.Println(value)


case value := <-ch2:

	fmt.Println(value)

}
```

---

Na površini:

```
čekaj bilo koji channel
```

---

Interno:

```
runtime prati više potencijalnih blokada
```

---

# 2. Select Kao Channel Multiplexer

Channel:

```
jedan communication point
```

---

Select:

```
više communication point-ova
```

---

Model:

```
          ch1
           |
           |
           v

        select

           ^
           |
           |

          ch2
```

---

Select bira:

```
prvi spreman slučaj
```

---

# 3. Problem koji Select Rešava

Bez select-a:

```go
value1 := <-ch1

value2 := <-ch2
```

---

Problem:

Ako:

```
ch1 nema podatak
```

goroutine čeka.

---

Sa select:

```go
select {

case v := <-ch1:

case v := <-ch2:

}
```

---

Runtime može koristiti:

```
bilo koji spreman kanal
```

---

# 4. Select Interna Struktura

Pojednostavljeno:

```go
type scase struct {

	c *hchan

	elem unsafe.Pointer

}

```

---

Svaki `case` opisuje:

- channel
- operaciju
- podatak

---

Primer:

```go
case ch <- value:
```

---

Interno:

```
send case
```

---

Primer:

```go
case <-ch:
```

---

Interno:

```
receive case
```

---

# 5. Runtime selectgo

Go runtime koristi internu funkciju:

```
selectgo()
```

---

Ona:

- pregleda case-ove
- proverava spremnost
- bira pobednika
- blokira ako treba

---

Mentalni model:

```
select{}

     |

     v

runtime.selectgo

     |

     v

channel operations
```

---

# 6. Faza 1 — Case Analiza

Kada goroutine izvrši:

```go
select {

case <-ch1:

case <-ch2:

}
```

---

Runtime prvo:

```
prikuplja sve case-ove
```

---

Dobija:

```
case 1 -> ch1 receive

case 2 -> ch2 receive
```

---

# 7. Faza 2 — Polling

Runtime proverava:

```
da li je neki case odmah moguć
```

---

Primer:

```
ch1

ready


ch2

blocked
```

---

Rezultat:

```
izaberi ch1
```

---

Nema:

- parking
- scheduler switch

---

# 8. Ready Case

Case je spreman kada:

---

## Receive

Postoji:

```
buffer data

ili

waiting sender
```

---

## Send

Postoji:

```
buffer space

ili

waiting receiver
```

---

# 9. Faza 3 — Izbor Case-a

Ako je više case-ova spremno:

Primer:

```go
select {

case <-ch1:

case <-ch2:

}
```

---

Oba:

```
ready
```

---

Go bira:

```
pseudo-random
```

---

Zašto?

Da spreči:

```
starvation
```

---

# 10. Randomizacija Case Redosleda

Runtime ne koristi:

uvek:

```
prvi case
```

---

Jer bi:

```go
case <-fastChannel:

case <-slowChannel:
```

---

mogao stalno favorizovati:

```
fastChannel
```

---

Rezultat:

```
fair scheduling
```

---

# 11. Faza 4 — Blocking Select

Ako nijedan case nije spreman:

Primer:

```go
select {

case <-ch1:

case <-ch2:

}
```

---

A:

```
ch1 empty

ch2 empty
```

---

Runtime:

```
park goroutine
```

---

# 12. Select i Sudog

Kod običnog channel wait-a:

postoji:

```
jedan sudog
```

---

Kod select-a:

goroutine mora čekati:

više kanala.

---

Primer:

```
G1


recvq ch1

recvq ch2

recvq ch3
```

---

Runtime kreira:

```
sudog za svaki case
```

---

# 13. Select Sudog Registracija

Primer:

```go
select {

case <-ch1:

case <-ch2:

}
```

---

Interno:

```
Goroutine

 |
 +---- sudog -> ch1
 |
 +---- sudog -> ch2
```

---

Sada svaki channel zna:

```
ova goroutine čeka mene
```

---

# 14. Buđenje Select Goroutine

Scenario:

```
select čeka:

ch1

ch2
```

---

Dolazi:

```go
ch2 <- value
```

---

Runtime:

1. pronalazi waiting receiver

2. bira select case

3. uklanja ostale sudog reference

4. budi goroutine

---

# 15. Cleanup Posle Select-a

Veoma važan korak.

---

Ako:

```
ch2 pobedi
```

---

Neophodno je ukloniti:

```
ch1 waiting entry
```

---

Inače:

```
memory leak

+

pogrešna buđenja
```

---

# 16. Select sa Default Case

Primer:

```go
select {

case value := <-ch:

	process(value)


default:

	return

}
```

---

Default znači:

```
nikada ne blokiraj
```

---

Tok:

```
check channels

 |

ready?

 |

 yes -> channel

 no -> default
```

---

# 17. Non-Blocking Receive Pattern

Primer:

```go
select {

case msg := <-ch:

	handle(msg)

default:

	noWork()

}
```

---

Koristi se za:

- polling
- worker loop
- event processing

---

# 18. Select Timeout Pattern

Primer:

```go
select {

case result := <-work:

	process(result)


case <-time.After(time.Second):

	timeout()

}
```

---

Interno:

```
timer channel
```

je samo još jedan case.

---

# 19. Select i Scheduler

Kada select blokira:

```
goroutine

↓

waiting

↓

scheduler
```

---

Kada case postane spreman:

```
waiting

↓

runnable

↓

running
```

---

---

# 20. Performance ImplIkacije

Select cena zavisi od:

```
broja case-ova
```

---

Primer:

```go
select {

case <-a:

case <-b:

case <-c:

}
```

---

više case-ova:

```
više provera

+

više sudog objekata
```

---

# 21. Select vs Polling Loop

Loš primer:

```go
for {

	select {

	case <-ch:

	default:

	}

}
```

---

Problem:

```
busy CPU loop
```

---

Bolje:

```
blokirajući select
```

---

# 22. Senior Pravilo

`select` nije:

```
if statement
```

---

To je:

```
runtime synchronization primitive
```

---

Koristi:

- channel koordinaciju
- cancellation
- multiplexing

---

# 23. Kompletan Select Tok

```
select


      |

      v


Analyze cases


      |

      v


Check ready channels


      |

      +------------+

      |            |

    found       none

      |            |

      v            v

 execute       register sudogs

 case              |

                  v

              park goroutine

                  |

                  v

              wake + cleanup
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ select runtime model

✅ selectgo koncept

✅ case strukture

✅ polling fazu

✅ blocking fazu

✅ sudog registraciju

✅ random izbor case-a

✅ default case

✅ timeout pattern

✅ scheduler interakciju

---

# Channels Internals

## Deo #5 — Channel Close Internals

---

# 📚 Sadržaj

- Uvod u channel close mehanizam
- Šta se dešava kod `close(ch)`
- Channel closed state
- Runtime close operacija
- Buđenje waiting goroutines
- Receive posle close
- Send posle close
- Panic mehanizam
- Closed flag
- Channel lifecycle završetak

---

# 1. Uvod u Channel Close Mehanizam

Operacija:

```go
close(ch)
```

izgleda jednostavno.

---

Ali runtime mora:

- promeniti stanje kanala
- obavestiti čekajuće goroutines
- omogućiti buduće receive operacije
- sprečiti nove send operacije

---

Channel close nije:

```
brisanje kanala
```

---

Već:

```
promena runtime state-a
```

---

# 2. Channel State Pre Close

Otvoren channel:

```
hchan

closed = false

buffer

sendq

recvq
```

---

Primer:

```
Channel

open

 |

buffer:

[10][20]

 |
 
sendq

recvq
```

---

# 3. Operacija `close(ch)`

Kada izvršimo:

```go
close(ch)
```

runtime radi:

---

1. proverava validnost

2. zaključava channel

3. postavlja closed flag

4. budi čekajuće goroutines

5. otključava channel

---

Mentalni model:

```
close()

   |

   v

lock hchan

   |

   v

closed = true

   |

   v

wake waiters
```

---

# 4. Closed Flag

Channel interno ima stanje:

```
closed
```

---

Pre:

```
closed = 0
```

---

Posle:

```
closed = 1
```

---

Ovo stanje kontroliše:

- send ponašanje
- receive ponašanje
- select ponašanje

---

# 5. Zašto Close Ne Uništava Channel?

Česta greška:

```
close(channel)

=

delete channel
```

---

Netačno.

---

Channel i dalje postoji.

---

Može:

```go
value := <-ch
```

---

Ako postoji buffer:

dobijaju se preostale vrednosti.

---

# 6. Close i Buffered Channel

Primer:

```go
ch := make(chan int, 3)

ch <- 10

ch <- 20

close(ch)
```

---

Stanje:

```
closed = true


buffer:

[10][20]
```

---

Receive:

```go
<-ch
```

---

Dobija:

```
10
```

---

Sledeći:

```
20
```

---

Tek nakon pražnjenja:

```
zero value
```

---

# 7. Receive Posle Close

Postoje dva slučaja.

---

## 7.1 Buffer Ima Podatke

Primer:

```
buffer:

[A][B]
```

---

Close:

```
closed=true
```

---

Receive:

```
A

B
```

---

---

## 7.2 Buffer Je Prazan

Primer:

```go
close(ch)
```

---

Receive:

```go
value := <-ch
```

---

Rezultat:

```
zero value
```

---

# 8. Receive sa `ok`

Najsigurniji način:

```go
value, ok := <-ch
```

---

Otvoren:

```go
ok == true
```

---

Closed:

```go
ok == false
```

---

Primer:

```go
for {

	value, ok := <-ch

	if !ok {

		break

	}

	fmt.Println(value)

}
```

---

# 9. Range Internals

Primer:

```go
for value := range ch {

	fmt.Println(value)

}
```

---

Interno radi:

```go
for {

	value, ok := <-ch

	if !ok {

		break

	}

}
```

---

`range` završava kada:

```
channel closed

+

buffer empty
```

---

# 10. Send Posle Close

Primer:

```go
close(ch)

ch <- 10
```

---

Rezultat:

```
panic
```

---

Zašto?

Zato što close znači:

```
nema više novih vrednosti
```

---

Send posle close krši:

```
channel contract
```

---

# 11. Zašto Send Panic-uje?

Zato što bi inače:

```
closed channel

+

new value
```

bilo nejasno.

---

Ko bi primio vrednost?

---

Close garantuje:

```
producer završio
```

---

# 12. Double Close

Primer:

```go
close(ch)

close(ch)
```

---

Rezultat:

```
panic
```

---

Channel može imati samo jednu tranziciju:

```
open

   |

   v

closed
```

---

Nema:

```
closed -> closed
```

---

# 13. Ko Zatvara Channel?

Pravilo:

```
vlasnik produkcije
```

---

Najčešće:

```
sender
```

---

Primer:

```
Producer

   |
   |
 close()

   |
   v

Consumer
```

---

# 14. Buđenje Waiting Receivers

Scenario:

```
recvq:

G1
G2
G3
```

---

Izvršava se:

```go
close(ch)
```

---

Runtime:

budi sve receivere.

---

Zašto?

Jer više neće biti novih send-ova.

---

# 15. Šta Dobijaju Probđeni Receiver-i?

Ako nema buffer-a:

dobijaju:

```
zero value

+

ok=false
```

---

Primer:

```go
value, ok := <-ch
```

---

Rezultat:

```go
value = 0

ok = false
```

---

# 16. Buđenje Waiting Sender-a

Poseban slučaj.

---

Ako postoje:

```
sendq
```

i neko pozove:

```go
close(ch)
```

---

Waiting sender-i se bude.

---

Ali:

rezultat nije uspešan send.

---

Dobijaju:

```
panic
```

---

# 17. Close i Select

Primer:

```go
select {

case value := <-ch:

case <-done:

}
```

---

Ako:

```go
close(done)
```

---

select case postaje:

```
ready
```

---

Close je često korišćen za:

- cancellation
- shutdown
- broadcast signal

---

# 18. Broadcast Efekat Close-a

Channel close može obavestiti:

više goroutines.

---

Primer:

```
done channel


G1 ----\
G2 ----- > <-done
G3 ----/
```

---

Close:

```go
close(done)
```

---

Svi se bude.

---

# 19. Channel Lifecycle

Kompletan život:

```
make(chan)

     |

     v

OPEN

     |

     v

send / receive

     |

     v

close()

     |

     v

CLOSED

     |

     v

GC cleanup
```

---

# 20. Channel Close vs Context Cancel

Close channel:

```
broadcast event
```

---

Context:

```go
ctx.Done()
```

---

interno koristi sličan princip.

---

Ali context dodaje:

- hierarchy
- deadlines
- values

---

# 21. Česte Greške

## Greška 1

Receiver zatvara channel.

---

Problem:

Drugi sender može poslati.

---

Rezultat:

```
panic
```

---

## Greška 2

Slanje posle close.

---

Rezultat:

```
panic
```

---

## Greška 3

Korišćenje close kao delete.

---

Netačno.

---

# 22. Senior Pravilo

Close znači:

```
nema više novih vrednosti
```

---

Ne znači:

```
uništi channel
```

---

Channel ostaje:

- čitljiv
- drainable
- garbage collectible

---

# 23. Kompletan Close Tok

```
close(ch)


      |

      v


Lock channel


      |

      v


closed = true


      |

      v


Wake receivers


      |

      v


Wake senders


      |

      v


Future sends panic


      |

      v


Receivers get zero value
```

---

# 📋 Rezime

U ovom delu naučili smo:

✅ close runtime mehanizam

✅ closed flag

✅ buffered close behavior

✅ receive posle close

✅ send posle close

✅ double close

✅ waiting goroutine wakeup

✅ broadcast pattern

✅ channel lifecycle

---

# Channels Internals

## Deo #6 — Channel Performance i Runtime Optimizations

---

# 📚 Sadržaj

- Channel cost model
- Zašto channel nije "besplatan"
- Buffered vs Unbuffered performanse
- Lock contention
- Scheduler overhead
- Cache efekti
- Channel vs Mutex vs Atomic
- Benchmark metodologija
- Production preporuke
- Završni pregled modula

---

# 1. Channel Cost Model

Svaka channel operacija ima određenu cenu.

---

Na primer:

```go
ch <- value
```

nije samo:

```
copy(value)
```

---

Runtime često mora da uradi:

- proveru stanja kanala
- zaključavanje (`lock`)
- manipulaciju internim redovima (`sendq` / `recvq`)
- kopiranje vrednosti
- eventualno parkiranje ili buđenje goroutine-a
- sinhronizaciju memorije

---

Zbog toga channel operacije imaju veću cenu od običnog pristupa memoriji.

---

# 2. Šta Sve Košta Jedan Send?

Najjeftiniji slučaj:

```
buffer ima slobodno mesto
```

---

Runtime:

```
lock

↓

copy element

↓

update sendx

↓

unlock
```

---

Skuplji slučaj:

```
nema receiver-a

↓

park sender

↓

scheduler switch

↓

wake sender
```

---

Najskuplji scenario uključuje promenu stanja scheduler-a.

---

# 3. Buffered vs Unbuffered

## Buffered

Prednosti:

- manje blokiranja
- bolji throughput
- manje scheduler intervencija

---

Mane:

- dodatna memorija
- mogućnost povećane latencije
- potrebno pažljivo odrediti veličinu bafera

---

## Unbuffered

Prednosti:

- direktna sinhronizacija
- prirodan backpressure
- jednostavnije razmišljanje o toku podataka

---

Mane:

- češće blokiranje
- više scheduler aktivnosti

---

# 4. Lock Contention

Svaki channel poseduje interni mutex.

---

Ako veliki broj goroutines radi:

```go
ch <- value
```

na isti channel:

```
G1 ─┐
G2 ─┼──► Channel
G3 ─┤
G4 ─┘
```

nastaje:

```
lock contention
```

---

Posledice:

- povećano čekanje
- slabiji throughput
- veća latencija

---

# 5. Scheduler Overhead

Kada goroutine mora da čeka:

```
Running

↓

Parked

↓

Runnable

↓

Running
```

---

Scheduler mora:

- parkirati goroutine
- izabrati drugu
- kasnije je ponovo aktivirati

---

Ove operacije nisu besplatne.

---

# 6. Cache Efekti

Channel objekat (`hchan`) predstavlja deljeni objekat.

---

Više CPU jezgara može pristupati istom channel-u.

---

To može izazvati:

```
cache invalidation

+

cache line bouncing
```

---

Posebno kod veoma visokog stepena konkurentnosti.

---

# 7. Veličina Elementa

Channel kopira vrednost.

---

Primer:

```go
chan LargeStruct
```

kopira:

```
LargeStruct
```

pri svakoj send operaciji.

---

Efikasnije je često koristiti:

```go
chan *LargeStruct
```

---

Međutim:

treba voditi računa o:

- životnom veku objekta
- deljenom mutable state-u
- ownership modelu

---

# 8. Channel vs Mutex

Mutex:

```
štiti state
```

---

Channel:

```
prenosi ownership
```

---

Ako samo štitimo mapu:

```go
sync.RWMutex
```

je često bolji izbor.

---

Ako želimo:

```
pipeline

worker pool

message passing
```

channel je prirodnije rešenje.

---

# 9. Channel vs Atomic

Atomic:

```
jedna memorijska lokacija
```

---

Channel:

```
komunikacija između goroutines
```

---

Primer:

Brojač zahteva:

```go
requests.Add(1)
```

---

Ovde channel nije potreban.

---

# 10. Benchmark Primer

```go
func BenchmarkChannel(
	b *testing.B,
) {

	ch := make(chan int, 1024)

	go func() {

		for range ch {
		}

	}()

	b.ResetTimer()

	for i := 0; i < b.N; i++ {

		ch <- i

	}

}
```

---

Porediti sa:

- `sync.Mutex`
- `sync.RWMutex`
- `atomic.Int64`

---

Tek nakon merenja donositi zaključke.

---

# 11. Profilisanje

Koristiti:

```bash
go test -bench=.
```

---

Za alokacije:

```bash
go test -benchmem
```

---

Za race proveru:

```bash
go test -race ./...
```

---

Za CPU profil:

```bash
go test -cpuprofile=cpu.prof
```

---

Za memory profil:

```bash
go test -memprofile=mem.prof
```

---

# 12. Production Saveti

✅ Ne praviti channel "za svaki slučaj".

---

✅ Birati odgovarajući kapacitet buffera.

---

✅ Izbegavati jedan channel kroz koji prolaze hiljade nepovezanih operacija.

---

✅ Koristiti `context.Context` za otkazivanje rada.

---

✅ Ne zatvarati channel sa strane receiver-a.

---

# 13. Kada Channel Nije Dobar Izbor?

Ako:

- postoji samo jedna promenljiva
- nema razmene poruka
- nema ownership transfer-a

---

Verovatno je:

```
atomic

ili

mutex
```

bolje rešenje.

---

# 14. Kada Channel Jeste Dobar Izbor?

Idealni slučajevi:

- Worker Pool
- Pipeline
- Fan-Out / Fan-In
- Event Processing
- Producer / Consumer
- Cancellation signal
- Broadcast shutdown

---

# 15. Mentalni Model

Zapamti:

```
Channel

=

Synchronization

+

Communication
```

---

Ne:

```
global shared queue
```

---

# 16. Production Checklist

Pre uvođenja channel-a postavi pitanja:

- Da li zaista razmenjujem poruke?
- Da li postoji ownership transfer?
- Da li je potreban redosled isporuke?
- Da li bi `Mutex` bio jednostavniji?
- Da li bi `Atomic` rešio problem?

---

# 17. Poređenje Concurrency Primitiva

| Primitive | Najbolja upotreba |
|-----------|-------------------|
| `atomic` | Brojači, flagovi, pokazivači |
| `Mutex` | Zaštita deljenog mutable state-a |
| `RWMutex` | Read-heavy workload |
| `Channel` | Komunikacija između goroutines |
| `Context` | Cancellation i deadline kontrola |
| `WaitGroup` | Čekanje završetka goroutines |
| `Cond` | Signalizacija promena stanja |

---

# 18. Senior Pravilo

Ne postoji:

```
najbrža concurrency primitive
```

---

Postoji samo:

```
najprikladnija
```

za konkretan problem.

---

Dobar Go kod:

- lako se razume
- lako se održava
- lako se testira

---

Performanse dolaze:

tek nakon pravilnog dizajna.

---

# 19. Završetak Modula

Završili smo:

```
docs/module-4/extras/

└── 05-channels-internals.md
```

---

Obrađene teme:

✅ `hchan`

✅ ring buffer

✅ send algoritam

✅ receive algoritam

✅ `select` internals

✅ `close` internals

✅ `sudog`

✅ waiting queues

✅ scheduler integracija

✅ performance analiza

---

# 🎯 Ključne Lekcije

Channels nisu samo:

```
FIFO queue
```

---

Channels predstavljaju spoj:

```
Communication

+

Synchronization

+

Memory Ordering

+

Scheduler Integration
```

---

Razumevanje njihovih internih mehanizama omogućava:

- lakše otkrivanje deadlock-a
- bolje performanse
- pravilan izbor concurrency primitiva
- efikasnije debugovanje
- kvalitetniji dizajn konkurentnih sistema

---

### ➡️ Sledeća lekcija **[**Advanced Channel Patterns**](06-advanced-channel-patterns.md)**

Obuhvatiće:

- Tee Pattern
- Bridge Pattern
- Or-Done Pattern
- Broadcast Pattern
- Token Bucket
- Semaphore preko channel-a
- Drop Pattern
- Dynamic Fan-Out
- Nil Channel Pattern
- Channel Ownership
- Graceful Shutdown obrasci
