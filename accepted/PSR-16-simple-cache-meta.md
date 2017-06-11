---
title: "PSR-16: Preprosti predpomnilnik meta dokument"
description: "Predpomnenje je pogosti način izboljšanja zmogljivosti katerega koli projekta in mnoge knjižnice ga uporabljajo ali bi ga lahko uporabljale."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-16-preprosti-predpomnilnik-meta-dokument"
---

PSR-16 meta dokument
====================

1. Povzetek
-----------

Predpomnenje je pogosti način izboljšanja zmogljivosti katerega koli projekta in
mnoge knjižnice ga uporabljajo ali bi ga lahko uporabljale. Interoperabilnost na tem nivoju
pomeni, da knjižnice lahko opustijo svoje lastne implementacije predpomnenja in se enostavno zanesejo
na podanega od ogrodja ali drugo namensko knjižnico, ki jo je izbral uporabnik.


2. Zakaj se truditi?
--------------------

PSR-6 že rešuje ta problem, vendar na nekoliko formalen način in preko razlage,
kar potrebujejo najpreprostejši primeri. Ta enostavnejši način cilja zgraditi
standardiziran nivo enostavnosti na vrhu obstoječih PSR-6 vmesnikov.


3. Obseg
--------

### 3.1 Cilji

* Enostaven vmesnik za operacije predpomnenja.
* Osnovna podpora za operacije na večih ključih zarad razlogov
  zmogljivosti (round-trip-time).
* Ponujanje razreda adapterja, ki vrne PSR-6 implementacijo v
  PSR-Simple-Cache.
* Morali bi biti mogoče izpostaviti oba PSR-ja predpomnilnika iz knjižnice predpomnilnika.

### 3.2 Niso cilji

* Reševanje vseh mogočih robnih primerov, PSR-6 to že omogoča v redu.


4. Pristopi
-----------

Izbrani pristop tu je v načrtu zelo okleščen, saj je uporabljen
samo za najbolj enostavne primere. Ni potrebno, da se ga lahko implementira v
vseh možnih ozadjih predpomnilnikov, niti da se ga uporabi v vseh primerih. Je zgolj
priročni nivo na vrhu PSR-6.


5. Ljudje
---------

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


6. Glasovi
--------

* **Vhodno glasovanje:**  https://groups.google.com/d/topic/php-fig/vyQTKHS6pJ8/discussion
* **Glasovanje sprejetja:**  https://groups.google.com/d/msg/php-fig/A8e6GvDRGIk/HQBJGEhbDQAJ


7. Pomembne povezave
--------------------

* [Raziskava obstoječih implementacij predpomnilnikov][1], by @dragoonis

[1]: https://docs.google.com/spreadsheet/ccc?key=0Ak2JdGialLildEM2UjlOdnA4ekg3R1Bfeng5eGlZc1E#gid=0
