---
title: "PSR-6: Predpomnilnik meta dokument"
description: "Skupni vmesnik za PHP knjižnice predpomnilnih sistemov."
read_time: "5 min"
updated: "2016-06-20"
slug: "psr-6-predpomnilnik-meta-dokument"
---

# PSR predpomnilnik meta dokument

## 1. Povzetek

Predpomnjenje je pogosti način izboljšanja zmogljivosti kateregakoli projekta, kar naredi
predpomnilne knjižnice ene izmed najpogostejših lastnosti mnogih ogrodij in
knjižnjic. To je pripeljalo do situacije, kjer so mnoge knjižnice naredile svoje lastne
predpomnilne knjižnice z različnimi nivoji funkcionalnosti. Zaradi teh razlik
se morajo razvijalci naučiti mnoge sisteme, ki lahko ali pa ne
ponujajo funkcionalnosti, ki jih potrebujejo. Poleg tega se razvijalci predpomnilnih
knjižnic sami soočajo z izbiro med samo podpiranjem omejenega števila
ogrodij ali izdelavo velikega števila razredov adapterjev.

## 2. Zakaj se truditi?

Skupni vmesnik za predpomnilne sisteme bi rešil te probleme. Razvijalci knjižnic in
ogrodij se lahko zanašajo na to, da sistem predpomnjenja deluje tako, kot
pričakujejo, medtem ko morajo razvijalci predpomnilnih sistemov samo implementirati
posamezen skupek vmesnikov namesto celotnega izbora adapterjev.

Poleg tega je implementacija predstavljena tu načrtovana za prihodnjo razširljivost.
Omogoča raznolikost notranje različnih vendar API kompatibilnih implementacij
in ponuja jasno pot za prihodnjo razširitev glede na kasnejše PSR-je ali specifične
izvajalce.

Prednosti:
* Standardizirani vmesnik za predpomnjenje omogoča prosto stoječe knjižnice za podporo
predpomnjenja posrednih podatkov brez napora; lahko so enostavno (opcijsko) odvisne
od tega standardnega vmesnika in ga dopolnjujejo brez skrbi nad
podrobnosti implementacije.
* Skupno razvite predpomnilne knjižnice, ki so deljene v mnogih projektih, tudi če
razširjajo ta vmesnik, bodo verjetno bolj robustne kot pa ducat ločeno
razvitih implementacij.

Slabosti:
* Katerikola standardizacija vmesnika ima tveganje slabe prihodnosti inovacije, saj
ni narejena kot bi morala biti. Vendar verjamemo, da je predpomnjenje področje, ki je dovolj
preplavljeno s produkti, kjer možnost razširitve ponujena tu blaži
kakršnokoli tveganje stagnacije.

## 3. Obseg

### 3.1 Cilji

* Skupen vmesnik za osnovni in vmesni nivo potreb predpomnjenja.
* Jasen mehanizem za razširitev specifikacije, da podpira napredne lastnosti,
tako v prihodnjih PSR-jih ali pri individualnih implementacijah. Ta mehanizem mora dovoljevati
več neodvisnih razširitev brez trkov.

### 3.2 Niso cilji

* Arhitekturna združljivost z vsemi obstoječimi implementacijami predpomnilnikov.
* Napredne predpomnilne lastnosti, kot so imenski prostori ali označevanje, ki je uporabljeno pri
manjšini uporabnikov.

## 4. Pristopi

### 4.1 Izbrani pristop

Ta specifikacija sprejema "model repozitorija" ali "data mapper" model za predpomnjenje
namesto bolj tradicionalnega modela "ključ-vrednost z rokom trajanja". Glavni
razlog je fleksibilnost. Enostaven model ključ/vrednost je veliko težje razširljiv.

Model tu pooblašča uporabo objekta CacheItem, ki predstavlja vnos predpomnilnika
in objekt Pool, ki je dana shramba predpomnjenih podatkov. Elementi so
pridobljeni iz zaloge, ki je v interakciji in vrnjeni nazaj. Medtem ko je včasih nekoliko
bolj gostobeseden, ponuja dober, robusten in fleksibilen pristop k predpomnjenju,
posebej v primerih, kjer je predpomnjenje bolj vključeno, kot pa enostavno shranjevanje in
pridobivanje niza.

Večina imen metod je bilo izbranih na osnovi pogoste prakse in imen metod v
anketi članskih projektov in ostalih popularnih ne-članskih projektov.

Prednosti:

* Fleksibilen in razširljiv
* Omogoča veliko raznolikosti v implementaciji brez kršenja vmesnika
* Implicitno ne izpostavlja konstruktorjev kot pseudo-vmesnika.

Slabosti:

* Nekoliko bolj gostobeseden od naivnega pristopa

Primeri:

Nekateri pogosti vzorci uporabe so prikazani spodaj. Ti so ne-normativni, vendar bi
morali prikazati aplikacijo nekih načrtovalskih odločitev.

```php
/**
 * Gets a list of available widgets.
 *
 * In this case, we assume the widget list changes so rarely that we want
 * the list cached forever until an explicit clear.
 */
function get_widget_list()
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    if (!$item->isHit()) {
        $value = compute_expensive_widget_list();
        $item->set($value);
        $pool->save($item);
    }
    return $item->get();
}
```

```php
/**
 * Caches a list of available widgets.
 *
 * In this case, we assume a list of widgets has been computed and we want
 * to cache it, regardless of what may already be cached.
 */
function save_widget_list($list)
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    $item->set($list);
    $pool->save($item);
}
```

```php
/**
 * Clears the list of available widgets.
 *
 * In this case, we simply want to remove the widget list from the cache. We
 * don't care if it was set or not; the post condition is simply "no longer set".
 */
function clear_widget_list()
{
    $pool = get_cache_pool('widgets');
    $pool->deleteItems(['widget_list']);
}
```

```php
/**
 * Clears all widget information.
 *
 * In this case, we want to empty the entire widget pool. There may be other
 * pools in the application that will be unaffected.
 */
function clear_widget_cache()
{
    $pool = get_cache_pool('widgets');
    $pool->clear();
}
```

```php
/**
 * Load widgets.
 *
 * We want to get back a list of widgets, of which some are cached and some
 * are not. This of course assumes that loading from the cache is faster than
 * whatever the non-cached loading mechanism is.
 *
 * In this case, we assume widgets may change frequently so we only allow them
 * to be cached for an hour (3600 seconds). We also cache newly-loaded objects
 * back to the pool en masse.
 *
 * Note that a real implementation would probably also want a multi-load
 * operation for widgets, but that's irrelevant for this demonstration.
 */
function load_widgets(array $ids)
{
    $pool = get_cache_pool('widgets');
    $keys = array_map(function($id) { return 'widget.' . $id; }, $ids);
    $items = $pool->getItems($keys);

    $widgets = array();
    foreach ($items as $key => $item) {
        if ($item->isHit()) {
            $value = $item->get();
        } else {
            $value = expensive_widget_load($id);
            $item->set($value);
            $item->expiresAfter(3600);
            $pool->saveDeferred($item, true);
        }
        $widget[$value->id()] = $value;
    }
    $pool->commit(); // If no items were deferred this is a no-op.

    return $widgets;
}
```

```php
/**
 * This examples reflects functionality that is NOT included in this
 * specification, but is shown as an example of how such functionality MIGHT
 * be added by extending implementations.
 */


interface TaggablePoolInterface extends Psr\Cache\CachePoolInterface
{
    /**
     * Clears only those items from the pool that have the specified tag.
     */
    clearByTag($tag);
}

interface TaggableItemInterface extends Psr\Cache\CacheItemInterface
{
    public function setTags(array $tags);
}

/**
 * Caches a widget with tags.
 */
function set_widget(TaggablePoolInterface $pool, Widget $widget)
{
    $key = 'widget.' . $widget->id();
    $item = $pool->getItem($key);

    $item->setTags($widget->tags());
    $item->set($widget);
    $pool->save($item);
}
```

### 4.2 Alternativa: Pristop "šibkega elementa"

Različni prejšnji osnutki so ubrali enostavni pristop "ključ vrednost z rokom trajanja",
znan tudi kot pristop "šibkega elementa". V tem modelu je bil objekt "Cache Item"
resnično nepameten objekt polje-z-metodami. Uporabniki bi ga sprožili
direktno, nato pa mu podali zalogo predpomnilnika. Medtem ko je bolj poznan, je ta pristop
efektivno preprečeval kakršnokoli smiselno razširitev elementa predpomnilnika. Efektivno
je naredil Cache Item konstruktor del implicitnega vmesnika in tako
zelo okrnil razširljivost ali zmožnost imeti element predpomnilnika, kjer
domuje inteligenca.

V anketi izvedeni v juniju 2013 je večina udeležencev pokazala jasno izbiro za
bolj robustni in manj konvekcionalni pristop "močni element" / repozitorij, ki je
bil sprejet kot pot naprej.

Prednosti:
* Bolj tradicionalni pristop.

Slabosti:
* Manj razširljiv ali fleksibilen.

### 4.3 Alternativa: Pristop "gola vrednost"

Nekaj najzgodnejših diskusij specifikacije predpomnilnika je predlagala preskočiti
koncept elementa predpomnilnika v celoti in samo brati/pisati surove vrednosti za predpomnjenje.
Medtem ko je to enostavneje, se je izkazalo, da je nemogoče povedati razliko
med zgrešitvijo predpomnilnika in katerakoli surova vrednost je bila izbrana, da predstavlja
zgrešitev predpomnilnika. To je, če je iskalnik predpomnilnika vrnil NULL, je nemogoče povedati, če
ni bilo nobene predpomnjene vrednosti ali če je predpomnjena vrednost NULL. (NULL je
legitimna vrednost za predpomnjenje v mnogih primerih.)

Večina bolj robustnih implementacij predpomnilnikov, ki smo jih pregledali -- v določenih
knjižnicah predpomnjenja zaloge in v domače izdelanih sistemih predpomnjenja, uporabljen v Drupal -- uporabljajo neke vrste
strukturirani objekt vsaj na `get`, da se izognejo nejasnosti med zgrešitvijo in
sentinelni vrednosti. Na osnovi prejšnje izkušnje, se je FIG odločil, da je gola vrednost
na `get`, nemogoča.

### 4.4 Alternativa: ArrayAccess Pool

Predlog je bil narediti, da Pool implementira ArrayAccess, kar bi omogočalo
get/set operacijam predpomnjenja uporabiti sintakso polja. To je bilo zavrnjeno zaradi
omejenega interesa, omejene fleksibilnosti tega pristopa (trivialni get in set s
privzetim krmiljenjem informacij je vse nemogoče) in ker je trivialno,
da določene implementacije vključijo kot dodatek, če je to
potrebno.

## 5. Ljudje

### 5.1 Urednik

* Larry Garfield

### 5.2 Sponzorji

* Paul Dragoonis, PPI Framework (Coordinator)
* Robert Hafner, Stash

## 6. Glasovanje

[Glasovanje sprejeto na e-poštnem seznamu](https://groups.google.com/forum/#!msg/php-fig/dSw5IhpKJ1g/O9wpqizWAwAJ)

## 7. Ustrezne povezave

_**Opomba:** Vrstni red kronološko padajoče._

* [Survey of existing cache implementations][1], by @dragoonis
* [Strong vs. Weak informal poll][2], by @Crell
* [Implementation details informal poll][3], by @Crell

[1]: https://docs.google.com/spreadsheet/ccc?key=0Ak2JdGialLildEM2UjlOdnA4ekg3R1Bfeng5eGlZc1E#gid=0
[2]: https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdDdVd2llN1kxczZQejZaa3JHcXA3b0E#gid=0
[3]: https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdEE3SU8zclNtdTNobWxpZnFyR0llSXc#gid=1

## 8. Popravki

### 8.1 Ravnanje z nepravilnimi vrednostmi DateTime v expiresAt()

Parameter `$expiration` metode `CacheItemInterface::expiresAt()` nima tipa v
vmesniku, vendar je v docbloku določek kot `\DateTimeInterface`. Namen je, da
je bodisi objekt `\DateTime` ali `\DateTimeImmutable` dovoljen. Vendar
`\DateTimeInterface` in `\DateTimeImmutable` sta bila dodana v PHP 5.5 in
avtorji ne rabijo vsiljevati trdih sintaktičnih zahtev za PHP 5.5 v
specifikaciji.

Kljub temu MORAJO izvajalci sprejeti samo `\DateTimeInterface` ali kompatibilne
tipe (kot sta `\DateTime` in `\DateTimeImmutable`) kakor da je metoda
eksplicitnega tipa. (Bodite pozorni, da pravila variance za parameter s tipom
lahko odstopajo med verzijami jezika.)

Simuliranje spodletelega preverjanja tipa se na žalost spreminja med verzijami
PHP in tako ni priporočljivo. Namesto tega bi izvajalci MORALI vreči instanco
`\Psr\Cache\InvalidArgumentException`. Sledeči primer kode je priporočljiv za
uveljavljanje preverjanja tipa na metodi expiresAt():

```php

class ExpiresAtInvalidParameterException implements Psr\Cache\InvalidArgumentException {}

// ...

if (! (
        null === $expiration
        || $expiration instanceof \DateTime
        || $expiration instanceof \DateTimeInterface
)) {
    throw new ExpiresAtInvalidParameterException(sprintf(
        'Argument 1 passed to %s::expiresAt() must be an instance of DateTime or DateTimeImmutable; %s given',
        get_class($this),
        is_object($expiration) ? get_class($expiration) : gettype($expiration)
    ));
}
```

### 8.2 Dodatki tipov

Izdaja 2.0 paketa `psr/cache` vključuje skalarne parametre tipov. Izdaja 3.0
paketa vključuje povratne tipe. Ta struktura izkorišča PHP 7.2 podporo
kovariance PHP 7.2, kar omogoča postopen proces nadgradnje, vendar zahteva
PHP 8.0 za kompatibilnost tipov.

Verzija 2.0 vsebuje tudi zgornji popravek 8.1, kar ponuja pravilno namigovanje
tipov za parameter `$expiration` metode `CacheItemInterface::expiresAt()`. To
pomeni manjšo spremembo v vrženi napaki pri neveljavnem vnosu; kljub temu, da je
še vedno usodno nedovoljen primer, FIG smatra to za sprejemljivo majhen prelom
združljivosti za nazaj, da se lahko izkoristi pravilne avtohtone tipe.

Izvajalci LAHKO dodajo povratne tipe k svojim lastnim paketom po svoji presoji
pod pogojem, da:

* se povratni tip ujema s tistimi iz paketa 3.0.
* izvedba določa najmanj PHP verzijo 8.0.0 ali novejše.

Izvajalci LAHKO dodajo tipe parametrov k svojim lastnim paketom v večji izdaji,
bodisi istočasno z dodajanjem povratnih tipov ali pa v ločeni izdaji pod
pogojem, da:

* se tipi parametrov ujemajo s tistimi iz paketa 2.0.
* izvedba določa najmanjšo PHP verzijo 8.0.0 ali novejše.
* izvedba zavisi od `"psr/cache": "^2.0 || ^3.0"`, tako da se izključi verzijo
  1.0 brez tipov.

Izvajalce se spodbuja, vendar od njih ni zahtevano, da njihovi paketi čimprej
preidejo na verzijo 3.0.
