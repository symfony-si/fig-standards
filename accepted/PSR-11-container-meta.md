# Kontejner meta dokument

## 1. Uvod

Ta dokument opusuje proces in razprave, ki so vodile k PSR konterja.
Njegov cilj je razložiti razloge za vsako odločitvijo.

## 2. Zakaj se truditi?

Obstaja na ducate konejnerjev injiciranja odvisnosti in le ti
DI kontejnerji imajo zelo različne načine shranjevanja vnosov.

- Nekateri temeljijo na povratnih klicih (Pimple, Laravel, ...)
- Drugi so osnovani na nastavitvah (Symfony, ZF, ...), z različnimi oblikami
  (polja PHP, datoteke YAML, datoteke XML...)
- Nekateri lahko dopolnjujejo tovarne...
- Nekateri imajo PHP API za izdelavo vnosov (PHP-DI, ZF, Symfony, Mouf...)
- Nekateri lahko naredijo samodejno žičenje (Laravel, PHP-DI, ...)
- Drugi lahko žičijo vnose na osnovi anotacij (PHP-DI, JMS Bundle...)
- Nekateri imajo grafični uporabniški vmesnik (Mouf...)
- Nekateri lahko prevedejo nastavitvene dtoteke v razrede PHP (Symfony, ZF...)
- Nekateri lahko naredijo aliase...
- Nekateri lahko uporabljajo proksije, da ponudijo leno nalaganje odvisnosti...

Torej, ko pogledate veliko sliko, je na voljo veliko število načinov, pri
katerih se lahko problem DI rešuje in tako veliko število različnih
implementacij. Vendar vsi kontejnerji DI, ki so na voljo, zadoščajo
enakim potrebam: ponujajo način, da aplikacija pridobi skupek
nastavljenih objektov (običajno storitev).

S standardizacijo načina, kako so vnosi zajeti iz kontejnerja, ogrodja in
knjižnice, ki uporabljajo kontejner PSR, lahko delajo s kompatibilnim kontejnerjem.
To bi omogočilo končnim uporabnikom izbrati njihov lastni kontejner na osnovi lastnih želja.

## 3. Obseg
### 3.1. Cillji

Zastavljeni cilj kontejnerja PSR je standardizacija, kako ogrodja in knjižnice uporabljajo
kontejner za pridobitev objektov in parametrov.

Pomembno je razlikovati med dvema uporabama kontejnerja:

- nastavljanje vnosov
- zajetje vnosov

Večino časa ti dve strani nista uporabljena pri isti stranki.
Medtem ko pogostokrat končni uporabniki stremijo k nastavljanju vnosov, je v splošnem ogrodje tisto, ki zajame
vnose za gradnjo aplikacije.

To je razlog zakaj se ta vmesnik osredotoča na to, kako se vnose lahko zajame iz kontejnerja.

### 3.2. Niso cilji

Kako so vnosi določeni v kontejnerju in kako so nastavljeni, je izven
obsega tega PSR. To naredi implementacijo kontejnerja unikatno. Nekateri
kontejnerji niti nimajo nastavitev (zanašajo se na samodejno žičenje), drugi se zanašajo
na kodo PHP definirano preko povratnih klicev, drugi na nastavitvene datoteke... Ta standard
se samo osredotoča na to, kako so vnosi zajeti.

Pravtako uporabljene konvencije poimenovanja niso del obsega tega
PSR. Zagotovo, ko pogledate konvencije poimenovanja, obstajata 2 strategiji:

- identifikator je ime razreda ali ime vmesnika (uporabljen večinoma
  pri ogrodjih z zmožnostjo samodejnega žičenja)
- identifikator je skupno ime (bližje imenu spremenljivke), kar je
  večinoma uporabljeno pri ogrodjih, ki se zanašajo na nastavitve.

Obe strategiji imata svoje prednosti in slabosti. Cilj tega PSR
ni izbira ene konvencije napram drugi. Namesto tega lahko uporabnik enostavno
uporabi aliase za premostitev razlike med dvema kontejnerjema z različnimi strategijami poimenovanja.

## 4. Priporočljiva uporaba: Kontejner PSR in lokator storitev

PSR trdi, da:

> "uporabniki NE SMEJO podati kontejnerja v objekt, tako da lahko objekt
> pridobi *svoje lastne odvisnosti*. Uporabniki, ki to naredijo, uporabljajo kontejner kot lokator storitev.
> Uporaba lokatorja storitev se v splošnem odsvetuje.

```php
// To ni v redu, kontejner uporabljate kot lokator storitev
class BadExample
{
    public function __construct(ContainerInterface $container)
    {
        $this->db = $container->get('db');
    }
}

// Namesto tega, premislite o direktnem injiciranju odvisnosti
class GoodExample
{
    public function __construct($db)
    {
        $this->db = $db;
    }
}
// Nato lahko uporabite kontejner za injiciranje objekta $db v vaš objekt $goodExample.
```

Pri `BadExample` ne bi smeli injicirati kontejnerja ker:

- to naredi kodo **manj interoperabilno**: z injiciranjem kontejnerja morate
  uporabiti kontejner, ki je kompatibilen s kontejnerjem PSR. Z drugo
  opcijo pa vaša koda dela s katerim koli kontejnerjem.
- razvijalca silite, da poimenuje njegov vnos "db". To poimenovanje je lahko
  v konfliktu z drugim paketom, ki ima enaka pričakovanja za drugo storitev.
- težje je testirati.
- iz vaše kode ni direktno jasno, da bo razred `BadExample` potreboval
  storitev "db". Odvisnosti so skrite.

Zelo pogosto bo `ContainerInterface` uporabljen pri drugih paketih. Kot končni uporabnik
PHP razvijalec uporablja ogrodje, je malo verjetno, da bo kdarkoli potreboval uporabiti kontejnerje
ali namige tipov direktno na `ContainerInterface`.

Bodisi je uporaba kontejnerja PSR v vaši kodi smatrana za dobro prakso ali ne, izvira iz
poznavanja ali so objekti, ki jih pridobivate, **odvisnosti** objektov, na katere se sklicuje
kontejner ali ne. Tu je nekaj primerov uporabe:

```php
class RouterExample
{
    // ...

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function getRoute($request)
    {
        $controllerName = $this->getContainerEntry($request->getUrl());
        // This is OK, the router is finding the matching controller entry, the controller is
        // not a dependency of the router
        $controller = $this->container->get($controllerName);
        // ...
    }
}
```

V tem primeru usmerjevalnik preoblikuje URL v ime vnosa krmilnika
in nato zajame krmilnik iz kontejnerja. Krmilnik v resnici ni
odvisnost za usmerjevalnik. Za pravilo palca, če je vaš objekt *računalniško*
ime vnosa izmed seznama vnosov, ki se lahko spreminjajo, je vaš primer uporabe zagotovo ustrezen.

Kot izjema objekti tovarne, katerih edini pomen je ustvariti in vrniti novo instanco lahko uporabijo
vzorec lokatorja storitev. Tovarna mora potem implementirati vmesnik, da lahko zamenja
samo sebe z drugo tovarno, ki uporablja isti vmesnik.

```php
// ok: a factory interface + implementation to create an object
interface FactoryInterface
{
    public function newInstance();
}

class ExampleFactory implements FactoryInterface
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function newInstance()
    {
        return new Example($this->container->get('db'));
    }
}
```

## 5. Zgodovina

Pred pošiljanjem PSR kontejnerja PHP-FIG-u, je bil `ContainerInterface`
prvič predlagan v projektu imenovanem [container-interop](https://github.com/container-interop/container-interop/).
Cilj projekta je bilo ponuditi testno podlago za implementacijo `ContainerInterface`
in utreti način za PSR kontejnerja.

V nadaljevanju tega meta dokumenta boste videli pogosta sklicevanja na
`container-interop.`

## 6. Ime vmesnika

Ime vmesnika je enako kot že je razpravljano za `container-interop`
(spremenjen je samo imenski prostor, da se ujema z drugimi PSR-ji).
O `container-interop` je bilo temeljito razpravljano [[4]](#link_naming_discussion) in odločilo se je z glasovanjem [[5]](#link_naming_vote)

Seznam možnosti za katere se smatra z njihovimi ustreznimi glasovi je:

- `ContainerInterface`: +8
- `ProviderInterface`: +2
- `LocatorInterface`: 0
- `ReadableContainerInterface`: -5
- `ServiceLocatorInterface`: -6
- `ObjectFactory`: -6
- `ObjectStore`: -8
- `ConsumerInterface`: -9

## 7. Metode vmesnika

Izbira katere metode naj vmesnik vsebuje, je bila narejena po statistični analizi obstoječih kontejnerjev. [[6]](#link_statistical_analysis).

Povzetek analize kaže, da:

- vsi kontejnerji ponujajo metodo za pridobivanje vnosa glede na njegov id
-  velika večina poimenuje tako metodo `get()`
- za vse kontejnerje ima metoda `get()` 1 obvezen parameter tipa niz
- nekateri kontejnerji imajo opcijsko dodatni argument za `get()`, vendar med kontejnerji nima enakega namena
- velika večina kontejnerjev ponuja metodo za testiranje, če lahko vrne vnos glede na njegov id
- velika večina poimenuje tako metodo `has()`
- za vse kontejnerje, ki ponujajo `has()`, ima metoda točno 1 parameter tipa niz
- velika večina kontejnerjev vrže izjemo namesto vračanja null, ko vnosa ni mogoče najti v `get()`
- velika večina kontejnerjev ne implementira `ArrayAccess`

Vprašanje ali vključiti metode za definicijo vnosov, je bilo razpravljano na samem začetku projekta container-interop [[4]](#link_naming_discussion).
Odločeno je bilo, da take metode ne pripadajo vmesniku opisanem tukaj, ker je izven tega obsega
(glejte sekcijo "Cilji").

Kot rezultat `ContainerInterface` vsebuje dve metodi:

- `get()`, ki vrne karkoli, z enim obveznim parametrom niza. Mora vreči izjemo, če vnosa ni mogoče najti.
- `has()`, ki vrne logično vrednost, z enim obveznim parametrom niza.

### 7.1. Število parametrov v metodi `get()`

Medtem ko `ContainerInterface` samo definira en obvezen parameter v `get()`, ni kompatibilen z
obstoječimi kontejnerji, ki imajo dodatne opcijske parametre. PHP dovoljuje implementaciji, da ponudi več parametrov
dokler so opcijski, ker implementacija *ne* zadosti vmesniku.

Razlika med container-interop [Specifikacija container-interop](https://github.com/container-interop/container-interop/blob/master/docs/ContainerInterface.md) trdi:

> Medtem ko `ContainerInterface` samo definira en obvezen parameter v `get()`, LAHKO implementacije sprejmejo dodatne opcijske parametre.

Ta stavek je bil odstranjen iz PSR-11 ker:

- Je nekaj, kar izhaja iz OO principov v PHP, tako da to ni direktno povezano s PSR-11
- Implementatorje ne želimo spodbujati, da dodajo dodatne parametre, saj priporočamo kodiranje napram vmesniku in ne implementaciji

Vseeno določene implementacije imajo dodatne opcijske parametre; to je tehnično ustrezno. Take implementacije so kompatibilne s PSR-11 [[11]](#link_get_optional_parameters).

### 7.2. Tip parametra `$id`

O tipu parametra `$id` v `get()` in `has()` je bilo razpravljano v projektu container-interop.

Medtem ko je `string` uporabljen pri vseh analiziranih kontejnerjih, je bilo predlagano, da se dovoli
karkoli (kot na primer objekte), kar lahko dovoljuje kontejnerjem ponuditi bolj napredne poizvedbe API-ja.

Kot podani primer je bilo uporabiti kontejner kot graditelja objekta. Parameter `$id` bi bil potem
objekt, ki opisuje, kako ustvariti instanco.

Zaključek razprave [[7]](#link_method_and_parameters_details) je bil, da je to izven obsega pridobivanja vnosov iz kontejnerja brez
vedenja, kako jih je kontejner ponudil in je bolj ustrezno za tovarno.

### 7.3. Vržene izjeme

Ta PSR ponuja dva vmesnika mišljena za implementacijo izjem kontejnerja.

#### 7.3.1 Osnovna izjema

`Psr\Container\ContainerExceptionInterface` je osnovni vmesnik. MORA biti implementiran pri izjema po meri, ki jih vrže direktno kontejner.

Pričakuje se, da katerakoli izjema, ki je del domene kontejnerja, implementira `ContainerExceptionInterface`. Nekaj primerov:

- če se kontejner zanaša na nastavitveno datoteko in če je ta nastavitvena datoteka pomankljiva, kontejner lahko vrže `InvalidFileException`, ki implementira `ContainerExceptionInterface`.
- če je med odvisnostmi zaznana ciklična odvisnost, kontejner lahko vrže `CyclicDependencyException`, ki implementira `ContainerExceptionInterface`.

Vendar če isto izjemo vrže neka koda izven obsega kontejnerja (na primer, vržena izjema med instantizacijo vnosa), za kontejner ni obvezno, da ovije to izjemo v izjemo po meri, ki implementira `ContainerExceptionInterface`.

Uporabnost vmesnika osnovne izjeme je bila vprašljiva: ni izjema, ki se jo bi običajno ujemalo [[8]](#link_base_exception_usefulness).

Vendar večina članov PHP-FIG je smatrala, da je to najboljša praksa. Vmesnik osnovne izjeme je implementiran v prejšnjih PSR-jih in mnogih članskih projektih. Zato se je obdržalo vmesnik osnovne izjeme.

#### 7.3.2 Izjema za ni najdeno

Klic metode `get` z neobstoječim id mora vreči izjemo, ki implementira `Psr\Container\NotFoundExceptionInterface`.

Za dani identifikator:

- če metoda `has` vrne `false`, potem MORA metoda `get` vreči `Psr\Container\NotFoundExceptionInterface`.
- če metoda `has` vrne `true`, potem to ne pomeni, da bo metoda `get` uspešna in ne vrže nobene izjeme. Lahko celo vrže `Psr\Container\NotFoundExceptionInterface`, če ena izmed odvisnosti zahtevanega vnosa manjka.

Zatorej, ko uporabnik ujema `Psr\Container\NotFoundExceptionInterface`, ima dva možna pomena [[9]](#link_not_found_behaviour):

- zahtevani vnos ne obstaja (slab zahtevek)
- ali odvisnost zahtevanega vnosa ne obstaja (t.j. da je kontejner napačno nastavljen)

Uporabnik lahko enostavno ugotovi razliko s klicem `has`.

V psevdo-kodi:

```php
if (!$container->has($id)) {
    // The requested instance does not exist
    return;
}
try {
    $entry = $container->get($id);
} catch (NotFoundExceptionInterface $e) {
    // Since the requested entry DOES exist, a NotFoundExceptionInterface means that the container is misconfigured and a dependency is missing.
}
```

8. Implementacije
-----------------

V času pisanja sledeči projekti že implementirajo in/ali uporabljajo `container-interop` verzijo vmesnika.

### Implementorji
- [Acclimate](https://github.com/jeremeamia/acclimate-container)
- [Aura.DI](https://github.com/auraphp/Aura.Di)
- [dcp-di](https://github.com/estelsmith/dcp-di)
- [League Container](https://github.com/thephpleague/container)
- [Mouf](http://mouf-php.com)
- [Njasm Container](https://github.com/njasm/container)
- [PHP-DI](http://php-di.org)
- [PimpleInterop](https://github.com/moufmouf/pimple-interop)
- [XStatic](https://github.com/jeremeamia/xstatic)
- [Zend ServiceManager](https://github.com/zendframework/zend-servicemanager)

### Middleware
- [Alias-Container](https://github.com/thecodingmachine/alias-container)
- [Prefixer-Container](https://github.com/thecodingmachine/prefixer-container)

### Uporabniki
- [Behat](https://github.com/Behat/Behat)
- [interop.silex.di](https://github.com/thecodingmachine/interop.silex.di)
- [mindplay/middleman](https://github.com/mindplay-dk/middleman)
- [PHP-DI Invoker](https://github.com/PHP-DI/Invoker)
- [Prophiler](https://github.com/fabfuel/prophiler)
- [Silly](https://github.com/mnapoli/silly)
- [Slim](https://github.com/slimphp/Slim)
- [Splash](http://mouf-php.com/packages/mouf/mvc.splash-common/version/8.0-dev/README.md)
- [Zend Expressive](https://github.com/zendframework/zend-expressive)

Ta seznam ni celovit in je na voljo samo kot primer, ki prikazuje, da je za PSR precej zanimanja.


9. Ljudje
---------
### 9.1 Uredniki

* [Matthieu Napoli](https://github.com/mnapoli)
* [David Négrier](https://github.com/moufmouf)

### 9.2 Sponzorji

* [Matthew Weier O'Phinney](https://github.com/weierophinney) (Coordinator)
* [Korvin Szanto](https://github.com/KorvinSzanto)

### 9.3 Prispevali so

Tu so abecedno zabeleženi vsi ljudje, ki so prispevali razpravam ali glasovanju (glede interoperabilnosti kontejnerja in med migracijo k PSR-11):

* [Alexandru Pătrănescu](https://github.com/drealecs)
* [Amy Stephen](https://github.com/AmyStephen)
* [Ben Peachey](https://github.com/potherca)
* [David Négrier](https://github.com/moufmouf)
* [Don Gilbert](https://github.com/dongilbert)
* [Jason Judge](https://github.com/judgej)
* [Jeremy Lindblom](https://github.com/jeremeamia)
* [Larry Garfield](https://github.com/crell)
* [Marco Pivetta](https://github.com/Ocramius)
* [Matthieu Napoli](https://github.com/mnapoli)
* [Nelson J Morais](https://github.com/njasm)
* [Paul M. Jones](https://github.com/pmjones)
* [Phil Sturgeon](https://github.com/philsturgeon)
* [Stephan Hochdörfer](https://github.com/shochdoerfer)
* [Taylor Otwell](https://github.com/taylorotwell)

## 10. Pomembne povezave

1. [Razprava o kontejnerju PSR in lokatorju storitev](https://groups.google.com/forum/#!topic/php-fig/pyTXRvLGpsw)
1. [Container-interop `ContainerInterface.php`](https://github.com/container-interop/container-interop/blob/master/src/Interop/Container/ContainerInterface.php)
1. [Seznam vseh težav](https://github.com/container-interop/container-interop/issues?labels=ContainerInterface&milestone=&page=1&state=closed)
1. <a name="link_naming_discussion"></a>[Razprava o imenu vmesnika in obsegu container-interop](https://github.com/container-interop/container-interop/issues/1)
1. <a name="link_naming_vote"></a>[Glsovanje za ime vmesnika](https://github.com/container-interop/container-interop/wiki/%231-interface-name:-Vote)
1. <a name="link_statistical_analysis"></a>[Statistična analiza obstoječih imen metod kontejnerjev](https://gist.github.com/mnapoli/6159681)
1. <a name="link_method_and_parameters_details"></a>[Razprava o imenih metod in parametrov](https://github.com/container-interop/container-interop/issues/6)
1. <a name="link_base_exception_usefulness"></a>[Razprava o uporabnosti osnovne izjeme](https://groups.google.com/forum/#!topic/php-fig/_vdn5nLuPBI)
1. <a name="link_not_found_behaviour"></a>[Razprava o `NotFoundExceptionInterface`](https://groups.google.com/forum/#!topic/php-fig/I1a2Xzv9wN8)
1. <a name="link_get_optional_parameters"></a>Razprave o pridobivanju opcijskih parametrov [v container-interop](https://github.com/container-interop/container-interop/issues/6) in na [PHP-FIG e-poštnem seznamu](https://groups.google.com/forum/#!topic/php-fig/zY6FAG4-oz8)
