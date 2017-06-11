---
title: "PSR-13: Povezave"
description: "Hipermedijske povezave postajajo vedno pomembnejši del spleta, tako v kontekstu HTML kot v različnih kontekstih oblik API. Vendar ne obstaja posamezna skupna oblika hipermedijev, niti ni na voljo skupnega načina za ponazoritev povezav med oblikami."
read_time: "5 min"
updated: "2017-06-11"
slug: "psr-13-povezave"
---

# Vmesniki definicij povezav

Hipermedijske povezave postajajo vedno pomembnejši del spleta, tako v kontekstu HTML
kot v različnih kontekstih oblik API. Vendar ne obstaja posamezna skupna oblika hipermedijev,
niti ni na voljo skupnega načina za ponazoritev povezav med oblikami.

Ta specifikacija cilja ponuditi PHP razvijalcem enostaven, skupni način predstavitve
hipermedijskih povezav neodvisno od oblike serializacije, ki je uporabljena. To omogoča
sistemu serializacijo odziva s hipermedijskimi povezavami v enega ali več povezljivih oblik neodvisno
od procesa odločanja, katere povezave naj bi to bile.

Ključne besede "MORA", "NE SME", "ZAHTEVA", "PRIPOROČA", "LAHKO" in "NEOBVEZNO"
v tem dokumentu se tolmačijo, kot je navedeno v
[RFC 2119](http://tools.ietf.org/html/rfc2119).

### Reference

- [RFC 2119](http://tools.ietf.org/html/rfc2119)
- [RFC 4287](https://tools.ietf.org/html/rfc4287)
- [RFC 5988](https://tools.ietf.org/html/rfc5988)
- [RFC 6570](https://tools.ietf.org/html/rfc6570)
- [IANA Link Relations Registry](http://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [Microformats Relations List](http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions)


## 1. Specifikacija

### 1.1 Osnovne povezave

Hipermedijska povezava sestoji iz vsaj:
- URI-ja, ki predstavlja ciljni vir, na katerega se sklicuje.
- Odnos, ki definira, kako se ciljni vir nanaša na izvor.

Obstajajo lahko različni ostali atributi povezave, odvisno od uporabljene oblike. Ker dodatni atributi
niso dobro standardizirani ali univerzalni, ta specifikacija le-teh ne cilja standardizirati.

Za namene te specifikacije veljajo sledeče definicije.

*    **Objekt implementacije** - Objekt, ki implementira enega ali več vmesnikov definiranih v tej
specifikaciji.

*    **Serializer** - Knjižnica ali drug sistem, ki vzame enega ali več objektov povezav in ustvari
serializirano predstavitev le-tega v neki definirani obliki.


### 1.2 Atributi

Vse povezave LAHKO vključujejo nobenega ali več dodatnih atributov onkraj URI-ja in odnosa.
Formalni register dovoljenih vrednosti tu ni določen in veljavnost vrednosti
je odvisna od konteksta in pogostokrat od posebne oblike serializacije. Pogostokrat podprte
vrednosti vključujejo 'hreflang', 'title' in 'type'.

Serializatorji LAHKO izpustijo atribute na objektu povezave, če je to zahtevano od
oblike serializacije. Vendar serializatorji MORAJO enkodirati vse možne podane atribute, da
dovolijo uporabniško razširitev, razen če je definicija oblike serializacije to preprečuje.

Nekateri atributi (pogostokrat `hreflang`), se lahko pojavijo večkrat v njihovem kontekstu. Zato
je vrednost atributa LAHKO polje vrednosti namesto enostavne vrednosti. Serializatorji LAHKO
enkodirajo to polje v katerokoli obliko ustrezno za obliko serializacije (kot so
seznam ločen s presledki, seznam ločen z vejicami itd.). Če za dani atribut ni
dovoljeno, da ima več vrednosti v določenem kontekstu, MORAJO serializatorji uporabiti
prvo podano vrednost in ignorirati vse naknadne vrednosti.

Če je vrednost atributa logični `true`, LAHKO serializatorji uporabijo skrajšane oblike, če je to ustrezno
in podprto za obliko serializacije. Na primer, HTML dovoljuje, da atributi
nimajo vrednosti, ko ima prisotnost atributa logični pomen. To pravilo velja
le, če je atribut logični `true` in ne za katere koli druge "truthy" vrednosti
v PHP, kot je na primer celo število 1.

Če je vrednost atributa logični `false`, MORA serializator izpustiti atribut v celoti,
razen če to spremeni semantični pomen rezultata. To pravilo velja, le
če je atribut logični `false`, in ne za katere koli ostale "falsey" vrednosti v PHP,
kot je na primer celo število 0.

### 1.3 Odnosi

Odnosi povezav so definirani kot nizi in so bodisi enostavna ključna beseda v
primeru javno definiranega odnosa ali absolutni URI v primeru
zasebnega odnosa.

V primeru, da je uporabljena enostavna ključna beseda, se MORA ujemati z eno izmed registra IANA:

http://www.iana.org/assignments/link-relations/link-relations.xhtml

Opcijsko se LAHKO uporabi register microformats.org, vendar to lahko ni veljavno
v vsakem kontekstu:

http://microformats.org/wiki/existing-rel-values

Odnos, ki ni definiran v enem izmed zgornjih registrov ali podobnem
javnem registru, se smatra za "zasebnega", kar je posebno za določeno
aplikacijo ali primer uporabe. Taki odnosi MORAJO uporabljati absolutni URI.

## 1.4 Predloge povezav

[RFC 6570](https://tools.ietf.org/html/rfc6570) definira obliko za predloge URI-jev, kar je
vzorec za URI, ki je pričakovan za vnos s vrednostmi podanimi od orodja
klienta. Nekatere hipermedijske oblike podpirajo povezave s predlogami, medtem ko druge ne,
in mnoge imajo poseben način označevanja, da je povezava predloga. Serializer za obliko,
ki ne podpira predlog URI MORA ignorirati kakršnokoli povezavo s predlogo, na katero naleti.

## 1.5 Razvijajoči se ponudniki

V nekaterih primerih ponudnik povezave lahko potrebuje zmožnost imeti dodatne povezave,
ki so dodane k njemu. V drugih ponudnik povezave je obvezno samo za branje s povezavami,
ki izvirajo iz med izvajanjem iz drugega vira podatkov. Zato so spremenljivi ponudniki
sekundarni vmesniki, ki se jih lahko opcijsko implementira.

Dodatno so nekateri objekti ponudnika povezav, kot je PSR-7 objekt odziva,
po načrtu nespremenljivi. To pomeni, da so lahko metode za dodajanje povezav k njemu na mestu
nezdružljive. Zato posamezna metoda vmesnika `EvolvableLinkProviderInterface`
zahteva, da je vrnjen novi objekt, ki je identičen prvotnemu vendar z
dodanim dodatnim objektom povezave.

## 1.6 Razvijajoči se objekti povezav

Objekti povezav so v večini primerov objekti vrednosti. Kot taki jim omogočajo, da se razvijajo
na enak način kot so uporabna možnost pri PSR-7 objektih vrednosti. Zato
je dodan dodatni EvolvableLinkInterface, ki ponuja metode za
izdelavo novih instanc objektov z eno samo spremembo. Enak model je uporabljen pri PSR-7
in zahvaljujoč obnašanja PHP-ja kopiranja pri pisanju, je še vedno učinkovit za CPU in spomin.

Na voljo ni razvijajoče se metode za predloge, vendar kot je vrednost predloge
povezave osnovana izključno na vrednosti href. NE SME biti nastavljena neodvisno, vendar
mora izvirati glede na to, ali je vrednosti href predloga povezave RFC 6570 ali ne.

## 2. Paket

Opisani vmesniki in razredi so ponujeni kot del paketa
[psr/link](https://packagist.org/packages/psr/link).

## 3. Vmesniki

### 3.1 `Psr\Link\LinkInterface`

```php
<?php

namespace Psr\Link;

/**
 * A readable link object.
 */
interface LinkInterface
{
    /**
     * Returns the target of the link.
     *
     * The target link must be one of:
     * - An absolute URI, as defined by RFC 5988.
     * - A relative URI, as defined by RFC 5988. The base of the relative link
     *     is assumed to be known based on context by the client.
     * - A URI template as defined by RFC 6570.
     *
     * If a URI template is returned, isTemplated() MUST return True.
     *
     * @return string
     */
    public function getHref();

    /**
     * Returns whether or not this is a templated link.
     *
     * @return bool
     *   True if this link object is templated, False otherwise.
     */
    public function isTemplated();

    /**
     * Returns the relationship type(s) of the link.
     *
     * This method returns 0 or more relationship types for a link, expressed
     * as an array of strings.
     *
     * @return string[]
     */
    public function getRels();

    /**
     * Returns a list of attributes that describe the target URI.
     *
     * @return array
     *   A key-value list of attributes, where the key is a string and the value
     *  is either a PHP primitive or an array of PHP strings. If no values are
     *  found an empty array MUST be returned.
     */
    public function getAttributes();
}
```

### 3.2 `Psr\Link\EvolvableLinkInterface`

```php
<?php

namespace Psr\Link;

/**
 * An evolvable link value object.
 */
interface EvolvableLinkInterface extends LinkInterface
{
    /**
     * Returns an instance with the specified href.
     *
     * @param string $href
     *   The href value to include.  It must be one of:
     *     - An absolute URI, as defined by RFC 5988.
     *     - A relative URI, as defined by RFC 5988. The base of the relative link
     *       is assumed to be known based on context by the client.
     *     - A URI template as defined by RFC 6570.
     *     - An object implementing __toString() that produces one of the above
     *       values.
     *
     * An implementing library SHOULD evaluate a passed object to a string
     * immediately rather than waiting for it to be returned later.
     *
     * @return static
     */
    public function withHref($href);

    /**
     * Returns an instance with the specified relationship included.
     *
     * If the specified rel is already present, this method MUST return
     * normally without errors, but without adding the rel a second time.
     *
     * @param string $rel
     *   The relationship value to add.
     * @return static
     */
    public function withRel($rel);

    /**
     * Returns an instance with the specified relationship excluded.
     *
     * If the specified rel is already not present, this method MUST return
     * normally without errors.
     *
     * @param string $rel
     *   The relationship value to exclude.
     * @return static
     */
    public function withoutRel($rel);

    /**
     * Returns an instance with the specified attribute added.
     *
     * If the specified attribute is already present, it will be overwritten
     * with the new value.
     *
     * @param string $attribute
     *   The attribute to include.
     * @param string $value
     *   The value of the attribute to set.
     * @return static
     */
    public function withAttribute($attribute, $value);


    /**
     * Returns an instance with the specified attribute excluded.
     *
     * If the specified attribute is not present, this method MUST return
     * normally without errors.
     *
     * @param string $attribute
     *   The attribute to remove.
     * @return static
     */
    public function withoutAttribute($attribute);
}
```

### 3.3 `Psr\Link\LinkProviderInterface`

```php
<?php

namespace Psr\Link;

/**
 * A link provider object.
 */
interface LinkProviderInterface
{
    /**
     * Returns an iterable of LinkInterface objects.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * are available, an empty array or \Traversable MUST be returned.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinks();

    /**
     * Returns an iterable of LinkInterface objects that have a specific relationship.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * with that relationship are available, an empty array or \Traversable MUST be returned.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinksByRel($rel);
}
```

### 3.4 `Psr\Link\EvolvableLinkProviderInterface`

```php
<?php

namespace Psr\Link;

/**
 * An evolvable link provider value object.
 */
interface EvolvableLinkProviderInterface extends LinkProviderInterface
{
    /**
     * Returns an instance with the specified link included.
     *
     * If the specified link is already present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   A link object that should be included in this collection.
     * @return static
     */
    public function withLink(LinkInterface $link);

    /**
     * Returns an instance with the specifed link removed.
     *
     * If the specified link is not present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   The link to remove.
     * @return static
     */
    public function withoutLink(LinkInterface $link);
}
```
