---
title: "PSR-16: Preprosti predpomnilnik"
description: "Ta dokument opisuje enostaven a razširljiv vmesnik za element predpomnilnika in gonilnik predpomnilnika."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-16-preprosti-predpomnilnik"
---

# Skupni vmesnik za knjižnice predpomnilnika

Ta dokument opisuje enostaven a razširljiv vmesnik za element predpomnilnika in
gonilnik predpomnilnika.

Ključne besede "MORA", "NE SME", "ZAHTEVA", "PRIPOROČA", "LAHKO" in "NEOBVEZNO"
v tem dokumentu se tolmačijo, kot je navedeno v
[RFC 2119][].

Končne implementacije LAHKO dekorirajo objekte z večimi
funkcionalnostmi od priporočene, vendar MORAJO najprej implementirati navedene
vmesnike/funkcionalnost.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

## 1. Specifikacija

### 1.1 Uvod

Predpomnjenje je pogosti način izboljšanja zmogljivosti kateregakoli projekta, kar naredi
predpomnilne knjižnice ene izmed najpogostejših lastnosti mnogih ogrodij in
knjižnjic. Interoperabilnost na tem nivoju pomeni, da lahko knjižnica opusti svojo
lastno implementacijo predpomnilnika in se enostavno zanaša na dano od
ogrodja, ali druge namenske knjižnice predpomnilnika.

PSR-6 že rešuje ta problem, vendar na nekoliko formalen in obširen način, kot
je to potrebno pri veliko primerih uporabe. Ta enostavnejši način cilja zgraditi
standardizirano racionalizacijo vmesnika za mnogo primerov. Je neodvisen
od PSR-6, vendar je načrtovan, da naredi kompatibilnost s PSR-6, kar se da
enostavno.

### 1.2 Definicije

Definicije za klicne knjižnice, izvedbene knjižnice, TTL, pretek in ključ so
kopirani iz PSR-6, saj veljajo iste predpostavke.

* **Klicna knjižnica** - Knjižnica ali koda, ki dejansko potrebuje storitve
  predpomnilnika. Ta knjižnica bo uporabila storitve predpomnjenja, ki implementirajo
  vmesnik tega standarda, vendar drugače ne bo poznala
  implementacije teh storitev predpomnjenja.

* **Izvedbena knjižnica** - Ta knjižnica je odgovorna za izvedbo
  tega standarda, da zagotovi storitve predpomnjenja katerikoli klicni knjižnici.
  Izvedbena knjižnica MORA zagotoviti razred, ki implementira vmesnik Psr\SimpleCache\CacheInterface.
  Izvedbene knjižnice MORAJO podpirati najmanj funkcionalnost TTL, kot je opisano
  spodaj s celotno drugo razdrobljenostjo.

* **TTL** - Življenska doba (TTL) elementa je količina časa medtem
  ko je ta element shranjen in se smatra za nesvežega. TTL je običajno definiran
  s celim številom, ki predstavlja čas v sekundah ali objektov DateInterval.

* **Pretek** - Dejanski čas, ko je element nastavljen za potek. To je
  običajno izračunano z dodajanjem TTL času, ko je objekt shranjen.

  Element s TTL 300 sekund, shranjen ob 1:30:00 bo imel pretek ob 1:35:00.

  Izvedbene knjižnice LAHKO potečejo element pred njegovim zahtevanim časom poteka,
  vendar MORAJO obravnavati element kot potečen, ko je dosežen njegov čas poteka. Če klicna
  knjižnica zaprosi, da je element shranjen vendar ne določa časa poteka ali
  določa čas poteka null ali TTL, izvedbena knjižnica LAHKO uporabi nastavljeno
  privzeto trajanje. Če privzeto trajanje ni bilo nastavljeno, MORA izvedbena knjižnica
  to prevesti kot zahtevek, ki predpomni element za vedno ali pa dokler to
  podpira implementacija podlage.

  Če je podana negativna ali nična vrednost TTL, element MORA biti izbrisan iz predpomnilnika,
  če že obstaja, kot je že razloženo.

* **Ključ** - Niz vsaj enega znaka, ki unikatno identificira
  predpomnjeni element. Izvedbene knjižnice MORAJO podpirati ključe sestavljene iz
  znakov `A-Z`, `a-z`, `0-9`, `_` in `.` v kateremkoli vrstnem redu v kodiranju UTF-8 in
  dolžine do 64 znakov. Izvedbene knjižnice LAHKO podpirajo dodatne
  znake in kodiranja ali daljše dolžine, vendar MORAJO podpirati vsaj ta
  minimum. Knjižnice so odgovorne za svoje lastne ubežne znake ključev nizov
  kot je potrebno, vendar MORAJO biti sposobne vrniti prvotni nespremenjeni ključ niza.
  Sledeči znaki so rezervirani za prihodnje razširitve in NE SMEJO biti
  podprti v izvedbenih knjižnicah `{}()/\@:`

* **Cache** - Objekt, ki implementira vmesnik `Psr\SimpleCache\CacheInterface`.

* **Zgrešitev predpomnilnika** - Zgrešitev predpomnilnika bo vrnila null in tako zaznala,
  če ena shranjena vrednost `null` ni moć+žna. To je glavno odstopanje od predpostavk
  PSR-6.

### 1.3 Predpomnilnik

Implementacije LAHKO ponudijo mehanizem za uporabnika, da določi privzeti TTL,
če ni določen za posamezni element predpomnilnika. Če ni ponujene uporabniško določene privzete
vrednosti, MORAJO implementacije privzeto nastaviti na največjo dovoljeno vrednost
implementacije podlage. Če implementacija podlage ne
podpira TTL, MORA biti uporabniško določena vrednost TTL tiho ignorirana.

### 1.4 Podatki

Izvedbene knjižnice MORAJO podpirati vse zaporednostne tipe PHP podatkov vključno z:

* **Nizi** - Znakovni nizi arbitrarne velikosti v kateremkoli PHP-kompatibilnem kodiranju.
* **Celimi števili** - Vsa cela števila katerekoli velikosti, podprta s strani PHP do 64-bitno podpisanih.
* **Števili s plavajočo vejico** - Vse podpisane vrednosti števil s plavajočo vejico.
* **Logičnimi vrednostmi** - True in False.
* **Null** - Vrednost null (čeprav je ne bo možno razlikovati od
  zgrešitve predpomnilnika, ko se jo prebere povratno nazaj).
* **Polja** - Indeksirana, asociativna in večdimenzijska polja arbitrarne globine.
* **Objekt** - Katerikoli objekt, ki podpira brezizgubno serializacijo in
  deserializacijo, tako da je $o == unserialize(serialize($o)). Objekti LAHKO
  uporabljajo PHP serializacijske objekte, `__sleep()` ali `__wakeup()` magični metodi,
  ali podobne funkcionalnosti jezika, če je to ustrezno.

Vsi podatki poslani v izvedbeno knjižnico MORAJO biti vrnjeni točno tako, kakor so
poslani. To vključuje tip vrednosti. To pomeni, da napaka, ki vrne
(string) 5 če je bila vrednost (int) 5 shranjena. Izvedbene knjižnice LAHKO uporabijo PHP-jeve
funkcije serialize()/unserialize() interno, vendar to ni zahtevano.
Združljivost z njimi je enostavno uporabljena kot osnova za sprejemljive vrednosti objektov.

Če ni možno vrniti točne shranjene vrednosti zaradi kakršnegakoli razloga, se MORAJO izvedbene
knjižnice odzvati z zgrešitvijo predpomnilnika kot pa s pokvarjenimi podatki.

## 2. Vmesniki

### 2.1 CacheInterface

Vmesnik predpomnilnika definira najosnovnejše operacije na zbirki vnosov predpomnilnika, kar
pomeni osnovno branje, pisanje in brisanje posameznih elementov predpomnilnika.

Dodatno ima metode za ravnanje z večimi nizi vnosov predpomnilnika, kot so pisanje, branje ali
brisanje več vnosov predpomnilnika naenkrat. To je uporabno, ko imate za opraviti veliko branja/pisanja predpomnilnika
in to vam omogoča izvršiti vaše operacije v enem klicu strežnika predpomnilnika, kar drastično zmanjša čas
latence.

Instanca CacheInterface ustreza posamezni zbirki elementov predpomnilnika z enim ključem imenskega prostora
in je enaka zalogi ("Pool") v PSR-6. Različne instance CacheInterface so LAHKO varnostno kopirane z isto
shrambo podatkov, vendar MORAJO biti logčino neodvisne.

```php
<?php

namespace Psr\SimpleCache;

interface CacheInterface
{
    /**
     * Fetches a value from the cache.
     *
     * @param string $key     The unique key of this item in the cache.
     * @param mixed  $default Default value to return if the key does not exist.
     *
     * @return mixed The value of the item from the cache, or $default in case of cache miss.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function get($key, $default = null);

    /**
     * Persists data in the cache, uniquely referenced by a key with an optional expiration TTL time.
     *
     * @param string                 $key   The key of the item to store.
     * @param mixed                  $value The value of the item to store. Must be serializable.
     * @param null|int|\DateInterval $ttl   Optional. The TTL value of this item. If no value is sent and
     *                                      the driver supports TTL then the library may set a default value
     *                                      for it or let the driver take care of that.
     *
     * @return bool True on success and false on failure.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function set($key, $value, $ttl = null);

    /**
     * Delete an item from the cache by its unique key.
     *
     * @param string $key The unique cache key of the item to delete.
     *
     * @return bool True if the item was successfully removed. False if there was an error.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function delete($key);

    /**
     * Wipes clean the entire cache's keys.
     *
     * @return bool True on success and false on failure.
     */
    public function clear();

    /**
     * Obtains multiple cache items by their unique keys.
     *
     * @param iterable $keys    A list of keys that can obtained in a single operation.
     * @param mixed    $default Default value to return for keys that do not exist.
     *
     * @return iterable A list of key => value pairs. Cache keys that do not exist or are stale will have $default as value.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $keys is neither an array nor a Traversable,
     *   or if any of the $keys are not a legal value.
     */
    public function getMultiple($keys, $default = null);

    /**
     * Persists a set of key => value pairs in the cache, with an optional TTL.
     *
     * @param iterable               $values A list of key => value pairs for a multiple-set operation.
     * @param null|int|\DateInterval $ttl    Optional. The TTL value of this item. If no value is sent and
     *                                       the driver supports TTL then the library may set a default value
     *                                       for it or let the driver take care of that.
     *
     * @return bool True on success and false on failure.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $values is neither an array nor a Traversable,
     *   or if any of the $values are not a legal value.
     */
    public function setMultiple($values, $ttl = null);

    /**
     * Deletes multiple cache items in a single operation.
     *
     * @param iterable $keys A list of string-based keys to be deleted.
     *
     * @return bool True if the items were successfully removed. False if there was an error.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $keys is neither an array nor a Traversable,
     *   or if any of the $keys are not a legal value.
     */
    public function deleteMultiple($keys);

    /**
     * Determines whether an item is present in the cache.
     *
     * NOTE: It is recommended that has() is only to be used for cache warming type purposes
     * and not to be used within your live applications operations for get/set, as this method
     * is subject to a race condition where your has() will return true and immediately after,
     * another script can remove it, making the state of your app out of date.
     *
     * @param string $key The cache item key.
     *
     * @return bool
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function has($key);
}
```

### 2.2 CacheException

```php
<?php

namespace Psr\SimpleCache;

/**
 * Interface used for all types of exceptions thrown by the implementing library.
 */
interface CacheException
{
}
```

### 2.3 InvalidArgumentException

```php
<?php

namespace Psr\SimpleCache;

/**
 * Exception interface for invalid cache arguments.
 *
 * When an invalid argument is passed, it must throw an exception which implements
 * this interface.
 */
interface InvalidArgumentException extends CacheException
{
}
```
