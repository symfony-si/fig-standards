---
title: "PSR-16: Preprosti predpomnilnik meta dokument"
description: "Predpomnenje je pogosti način izboljšanja zmogljivosti katerega koli projekta in mnoge knjižnice ga uporabljajo ali bi ga lahko uporabljale."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-16-preprosti-predpomnilnik-meta-dokument"
---

# PSR-16 meta dokument

## 1. Povzetek

Predpomnenje je pogosti način izboljšanja zmogljivosti katerega koli projekta in
mnoge knjižnice ga uporabljajo ali bi ga lahko uporabljale. Interoperabilnost na tem nivoju
pomeni, da knjižnice lahko opustijo svoje lastne implementacije predpomnenja in se enostavno zanesejo
na podanega od ogrodja ali drugo namensko knjižnico, ki jo je izbral uporabnik.


## 2. Zakaj se truditi?

PSR-6 že rešuje ta problem, vendar na nekoliko formalen način in preko razlage,
kar potrebujejo najpreprostejši primeri. Ta enostavnejši način cilja zgraditi
standardiziran nivo enostavnosti na vrhu obstoječih PSR-6 vmesnikov.

## 3. Obseg

### 3.1 Cilji

* Enostaven vmesnik za operacije predpomnenja.
* Osnovna podpora za operacije na večih ključih zarad razlogov
  zmogljivosti (round-trip-time).
* Ponujanje razreda adapterja, ki vrne PSR-6 implementacijo v
  PSR-Simple-Cache.
* Morali bi biti mogoče izpostaviti oba PSR-ja predpomnilnika iz knjižnice predpomnilnika.

### 3.2 Niso cilji

* Reševanje vseh mogočih robnih primerov, PSR-6 to že omogoča v redu.


## 4. Pristopi

Izbrani pristop tu je v načrtu zelo okleščen, saj je uporabljen
samo za najbolj enostavne primere. Ni potrebno, da se ga lahko implementira v
vseh možnih ozadjih predpomnilnikov, niti da se ga uporabi v vseh primerih. Je zgolj
priročni nivo na vrhu PSR-6.

## 5. Ljudje

### 5.1 Urednik(i)

* Paul Dragoonis (@dragoonis)

### 5.2 Sponzorji

* Jordi Boggiano (@seldaek) - Composer (koordinator)
* Fabien Potencier (@fabpot) - Symfony

### 5.3 Prispevali so

Za njihovo vlogo pri pisanju prvotne verzije tega PSR-ja predpomnilnika:

* Evert Pot (@evert)
* Florin Pățan (@dlsniper)

Za zgodnji pregled

* Daniel Messenger (@dannym87)

## 6. Glasovi

* **Vhodno glasovanje:**  https://groups.google.com/d/topic/php-fig/vyQTKHS6pJ8/discussion
* **Glasovanje sprejetja:**  https://groups.google.com/d/msg/php-fig/A8e6GvDRGIk/HQBJGEhbDQAJ

## 7. Pomembne povezave

* [Raziskava obstoječih implementacij predpomnilnikov][1], by @dragoonis

[1]: https://docs.google.com/spreadsheet/ccc?key=0Ak2JdGialLildEM2UjlOdnA4ekg3R1Bfeng5eGlZc1E#gid=0

## 8. Popravki

### 8.1 Throwable

2.0 različica paketa `psr/simple-cache` ima posodobljen razred
`Psr\SimpleCache\CacheException`, da razširi `\Throwable`. To se šteje za
združljivo spremembo za izvedbene knjižnice, ki uporabljajo vsaj PHP 7.4.

### 8.2 Dodatki tipov

Različica paketa `psr/simple-cache` 2.0 vključuje skalarni tip parametrov in
povečuje minimalno različico PHP na 8.0. To se šteje za združljivo spremembo za
izvedbene knjižnice, saj PHP 7.2 prinaša kovarianco za parametre. Vsaka
implementacija 1.0 je združljiva z 2.0. Za klice knjižnic pa to zmanjšuje tipe,
ki jih lahko podajo (ker se je prej lahko sprejel kateri koli parameter, ki se
lahko spremeni v niz) in kot taka zahteva povečanje glavne različice.

Različica 3.0 vključuje povratne tipe. Povratni tipi zlomijo združljivost s
prejšnjimi različicami za izvedbene knjižnice, saj PHP ne podpira širjenja
povratnih tipov.

Izvedbene knjižnice **LAHKO** dodajo povratne tipe v svoje lastne pakete po
lastni presoji, pod pogojem, da:

* se povratni tipi ujemajo s tipi v paketu 3.0.
* implementacija določa minimalno različico PHP 8.0.0 ali novejšo.
* je implementacija odvisna od `"psr/simple-cache": "^2 || ^3"`, da izključi
  različico 1.0, ki nima tipov.

Izvedbene knjižnice **LAHKO** dodajo tipe parametrov v svoj paket v novi manjši
različici, bodisi istočasno z dodajanjem povratnih tipov bodisi v naslednji
različici, pod pogojem, da:

* se tipi parametrov ujemajo ali razširijo tiste v paketu 2.0.
* implementacija določa minimalno različico PHP 8.0, če uporablja mešane ali tipe unij ali novejšo.
* je implementacija odvisna od `"psr/simple-cache": "^2 || ^3"`, da izključi
  različico 1.0, ki nima tipov.

Izvajalce se spodbuja, vendar od njih ni zahtevano, da njihovi paketi čimprej
preidejo na verzijo 3.0.

Klicne knjižnice se spodbuja, da čimprej zagotovijo pošiljanje pravega tipa in
da posodobijo svojo zahtevo na `"psr/simple-cache": "^1 || ^2 || ^3"`.
