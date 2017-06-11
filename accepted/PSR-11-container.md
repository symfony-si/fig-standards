---
title: "PSR-11: Vmesnik kontejnerja"
description: "Ta dokument opisuje skupni vmesnik za kontejnerje injiciranja odvisnosti."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-11-kontejner"
---

Vmesnik kontejnerja
===================

Ta dokument opisuje skupni vmesnik za kontejnerje injiciranja odvisnosti.

Zastavljeni cilj `ContainerInterface` je standardizacija, kako ogrodja in knjižnice
uporabljajo kontejner za pridobitev objektov in parametrov (imenovano *vnosi* v
nadaljevanju tega dokumenta).

Ključne besede "MORA", "NE SME", "ZAHTEVA", "PRIPOROČA", "LAHKO" in "NEOBVEZNO"
v tem dokumentu se tolmačijo, kot je navedeno v
[RFC 2119][].

Beseda `implementator` v tem dokumentu se tolmači kot nekdo, ki implementira
`ContainerInterface` v knjižnici ali ogrodju za injiciranje odvisnosti.
Uporabniki kontejnerjev injiciranja odvisnosti (DIC) se imenujejo `uporabniki`.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

1. Specifikacija
----------------

### 1.1 Osnove

#### 1.1.1 Identifikatorji vnosov

Identifikator vnosa je kakršenkoli PHP veljaven niz vsaj enega znaka, ki unikatno identificira element znotraj kontejnerja. Identifikator vnosa je
neprosojni niz, tako da klicajoči NE SMEJO predpostavljati, da struktura niza nosi kakršenkoli semantični pomen.

#### 1.1.2 Branje iz kontejnerja

- `Psr\Container\ContainerInterface` izpostavlja dve metodi: `get` in `has`.

- `get` potrebuje en obvezen parameter: identifikator vnosa, ki MORA biti niz.
  `get` lahko vrne karkoli (*mešano* vrednost), ali vrže `NotFoundExceptionInterface`, če je identifikator
  ni poznan kontejnerju. Dva zaporedna klica `get` z enakim identifikatorjem
  MORATA vrniti enako vrednost. Vendar odvisno od načrta `implementatorja`
  in/ali nastavitev `uporabnika`, se lahko vrnejo različne vrednosti, tako da
  `uporabnik` se NE SME zanašati na vrnitev iste vrednosti pri dveh zaporednih klicih.

- `has` potrebuje en unikaten parameter: identifikator vnosa, ki MORA biti niz.
  `has` MORA vrniti `true`, če je identifikator vnosa poznan kontejnerju in `false`, če ni.
  Če `has($id)` vrne false, MORA `get($id)` vreči `NotFoundExceptionInterface`.

### 1.2 Izjeme

Izjeme ki jih vrže direktno kontejner MORAJO implementirati
[`Psr\Container\ContainerExceptionInterface`](#container-exception).

Klic metode `get` z neobstoječim id MORA vreči
[`Psr\Container\NotFoundExceptionInterface`](#not-found-exception).

### 1.3 Priporočena uporaba

Uporabniki NE SMEJO podati kontejnerja v objekt, tako da lahko objekt pridobi *svoje lastne odvisnosti*.
To pomeni, da je kontejner uporabljen kot [Service Locator](https://en.wikipedia.org/wiki/Service_locator_pattern),
ki je vzorec, ki se ga v sploščnem odsvetuje.

Prosimo, sklicujte se na sekcijo 4 dokumenta META za več podrobnosti.

2. Paket
--------

Opisani vmesniki in razredi ter pomembne izjeme so dodane kot del
paketa [psr/container](https://packagist.org/packages/psr/container).

Paketi, ki ponujajo implementacijo PSR kontejnerja bi morali deklarirati, da ponujajo `psr/container-implementation` `1.0.0`.

Projekti, ki uporabljajo implementacijo, bi morali zahtevati `psr/container-implementation` `1.0.0`.

3. Vmesniki
-----------

<a name="container-interface"></a>
### 3.1. `Psr\Container\ContainerInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Describes the interface of a container that exposes methods to read its entries.
 */
interface ContainerInterface
{
    /**
     * Finds an entry of the container by its identifier and returns it.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @throws NotFoundExceptionInterface  No entry was found for **this** identifier.
     * @throws ContainerExceptionInterface Error while retrieving the entry.
     *
     * @return mixed Entry.
     */
    public function get($id);

    /**
     * Returns true if the container can return an entry for the given identifier.
     * Returns false otherwise.
     *
     * `has($id)` returning true does not mean that `get($id)` will not throw an exception.
     * It does however mean that `get($id)` will not throw a `NotFoundExceptionInterface`.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @return bool
     */
    public function has($id);
}
~~~

<a name="container-exception"></a>
### 3.2. `Psr\Container\ContainerExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Base interface representing a generic exception in a container.
 */
interface ContainerExceptionInterface
{
}
~~~

<a name="not-found-exception"></a>
### 3.3. `Psr\Container\NotFoundExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * No entry was found in the container.
 */
interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
~~~
