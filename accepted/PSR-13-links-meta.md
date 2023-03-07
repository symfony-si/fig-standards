---
title: "PSR-13: Povezave meta dokument"
description: "Hipermedijske povezave postajajo vedno pomembnejši del spleta, tako v kontekstu HTML kot v različnih kontekstih oblik API. Vendar ne obstaja posamezna skupna oblika hipermedijev, niti ni na voljo skupnega načina za ponazoritev povezav med oblikami."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-13-povezave-meta-dokument"
---

# Definicije povezav meta dokument

## 1. Povzetek

Hipermedijske povezave postajajo vedno pomembnejši del spleta, tako v kontekstu HTML
kot v različnih kontekstih oblik API. Vendar ne obstaja posamezna skupna oblika hipermedijev,
niti ni na voljo skupnega načina za ponazoritev povezav med oblikami.

Ta specifikacija cilja ponuditi PHP razvijalcem enostaven, skupni način predstavitve
hipermedijskih povezav neodvisno od oblike serializacije, ki je uporabljena. To omogoča
sistem za serializacijo odziva s hipermedijskimi povezavami v enega ali več povezljivih oblik neodvisno
od procesa odločanja, katere povezave naj bi to bile.

## 2. Obseg

### 2.1 Cilji

* Ta specifikacija cilja izvleči in standardizirati ponazoritev hipermedijske povezave med različnimi
oblikami.

### 2.2 Niso cilji

* Ta specifikacija ne išče standardizacije ali naklonjenosti kakršnikoli posebni obliki serializacije.

## 3. Odločitve načrta

### Zakaj ni metod mutacij?

Eni izmed ključnih ciljev te specifikacije so PSR-7 objeki odziva. Objekti odziva morajo biti v zasnovi
nespremenljivi. Ostale implementacije vrednost-objekt bi verjetno tudi zahtevale nespremenljiv vmesnik.

Poleg tega nekateri objekti Link Provider-ja lahko niso objekti vrednosti, ampak druge objekti znotraj določene
domene, ki so sposobni ustvariti povezave spotoma, morda iz rezultatov podatkovne baze ali drugih osnovnih
predstavitev. V teh primerih bi bila definicija zapisljivega ponudnika nezdružljiva.

Zato ta specifikacija deli metode dostopnika in razvijalne metode v ločene vmesnike
kar omogoča objektov implementirati le različice za branje ali razvijanje, kot je primerno za njihov primer uporabe.

### Zakaj ima lahko rel na objektu Link več vrednosti?

Različni hipermedijski standardi upravljajo več povezav z istim razmerjem različno. Nekateri imajo eno samo
povezavo, ki ima določenih več atributov rel. Drugi imajo en sam atribut rel, ki nato vsebuje več povezav.

Določanje vsake povezave enolično, ampak omogočati, da ima več atributov rel določa definicijo najbolj združljivega-imenovalca.
Posamezen objekt LinkInterface je lahko serializiran v en ali več vnosov povezav v določenem hipermedijskem formatu, kakor
je ustrezno. Vendar pa je določati več objektov povezav, kjer ima vsaka en atribut rel in enak URI tudi ustrezno in
hipermedijski format lahko to tudi serializira kot ustrezno.

### Zakaj se potrebuje LinkProviderInterface?

V mnogih kontekstih bo niz povezav dodan kakemu drugemu objektu. Ti objekti se lahko uporabljajo v situacijah
kjer je vse, kar je pomembno, njihove povezave ali nekatere podskupine njihovih povezav. Na primer, lahko se določijo
različni objekti vrednosti, ki predstavljajo različne formate REST, kot so HAL, JSON-LD, ali Atom. Lahko je koristno
za pridobivanje teh povezav iz takega objekta ter enakomerno za nadaljnjo obdelavo. Na primer naslednje/prejšnje povezave
se lahko pridobijo iz objekta in dodajo, da objekt PSR-7 Response kot glava povezave. Alternativno, za veliko povezav
bi bilo smiselno, da predstavljajo z "prednalaganjem" razmerja povezave, kar bi namigovalo na HTTP 2-združljiv
spletni strežnik, kjer bi se morali povezani viri pretakati do klienta v pričakovanju naknadne zahteve.

Vsi ti primeri so neodvisni od vsebine ali kodiranja objekta. Z zagotavljanjem skupnega vmesnika
za dostop do takih povezav, omogočamo generično ravnanje samih povezav, ne glede na objekt vrednosti ali
objekt domene, ki jih proizvaja.

## 4. Ljudje

### 4.1 Urednik(i)

* Larry Garfield

### 4.2 Sponzorji

* Matthew Weier O'Phinney (koordinator)
* Marc Alexander

### 4.3 Prispevali so

* Evert Pot

## 5. Glasovi

## 6. Pomembne povezave

* [Kaj je v povezavi?](http://evertpot.com/whats-in-a-link/) by Evert Pot
* [Seznam delovne skupine FIG Link](https://groups.google.com/forum/#!forum/php-fig-link)

## 7. Popravki

### 7.1 Dodatki tipov

Izdaja paketa `psr/link` 1.1 vključuje skalarni tip parametrov. Izdaja paketa
2.0 vključuje tipe povratnih tipov. Ta struktura omogoča podporo kovariance PHP
7.2 za postopno nadgradnjo, vendar zahteva PHP 8.0 za združljivost tipov.

Izvajalci LAHKO po lastni presoji dodajo tipe povratnih tipov v svoje lastne
pakete, če:

* povratni tipi ustrezajo tistim v paketu 2.0.
* implementacija določa minimalno različico PHP 8.0.0 ali novejšo.

Izvajalci LAHKO dodajo tipe parametrov v svoje lastne pakete v novi glavni
različici, bodisi istočasno z dodajanjem povratnih tipov ali v kasnejši izdaji,
če:

* se tipi parametrov ujemajo s tistimi v paketu 1.1.
* implementacija določa minimalno različico PHP 8.0.0 ali novejšo.
* je implementacija odvisna od `"psr/link": "^1.1 || ^2.0"`, da izključi
  ne-tipizirano različico 1.0.

Izvajalce se spodbuja, vendar od njih ni zahtevano, da njihovi paketi čimprej
preidejo na verzijo 2.0.

## 7.2 Obdelava tipov atributov

Izvirna specifikacija je vsebovala neusklajenost glede vrednosti polj za
atribute. Besedilo specifikacije v razdelku 1.2 navaja, da lahko vrednosti
atributov (kot podane na `EvolvableLinkInterface::withAttribute()`)
predstavljajo več vrst, nekatere od njih pa omogočajo posebno obdelavo (kot so
booli ali polja). Vendar je bila specifikacija te metode zapisana z napako, saj
je bil parameter `$value` določen kot niz.

Da bi se rešili te težave, je bil v kasnejših izdajah vmesnik popravljen, da
dovoljuje, da je `$value` tipa `string|\Stringable|int|float|bool|array`.
Izvajalci NAJ obravnavajo `Stringable` objekt enako kot `string` parameter.
Izvajalci LAHKO `int`, `float`, ali `bool` serializirajo na alternativne načine,
ki upoštevajo tip, za določen format serializacije. Ostali tipi predmetov ali
virov ostajajo nedovoljeni.

Večkratni klici metode `withAttribute()` z istim `$name` MORAJO preglasiti prej
določene vrednosti, kot že navaja specifikacija. Da se ponudi več vrednosti
določenemu atributu, se lahko pošlje `array` z željenimi vrednostmi.

Vse ostale smernice in zahteve v sekciji 1.2 ostajajo veljavne.
