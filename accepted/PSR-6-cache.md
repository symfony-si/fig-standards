## Uvod

Predpomnjenje je pogosti način izboljšanja zmogljivosti kateregakoli projekta, kar naredi
predpomnilne knjižnice ene izmed najpogostejših lastnosti mnogih ogrodij in
knjižnjic. To je pripeljalo do situacije, kjer so mnoge knjižnice naredile svoje lastne
predpomnilne knjižnice z različnimi nivoji funkcionalnosti. Zaradi teh razlik
se morajo razvijalci naučiti mnoge sisteme, ki lahko ali pa ne
ponujajo funkcionalnosti, ki jih potrebujejo. Poleg tega se razvijalci predpomnilnih
knjižnic sami soočajo z izbiro med samo podpiranjem omejenega števila
ogrodij ali izdelavo velikega števila razredov adapterjev.

Skupni vmesnik za predpomnilne sisteme bi rešil te probleme. Razvijalci knjižnic in
ogrodij se lahko zanašajo na to, da sistem predpomnjenja deluje tako, kot
pričakujejo, medtem ko morajo razvijalci predpomnilnih sistemov samo implementirati
posamezen skupek vmesnikov namesto celotnega izbora adapterjev.

Ključne besede "MORA", "NE SME", "ZAHTEVA", "PRIPOROČA", "LAHKO" in "NEOBVEZNO"
v tem dokumentu se tolmačijo, kot je navedeno v
[RFC 2119][].

[RFC 2119]: http://tools.ietf.org/html/rfc2119

## Cilj

Cilj tega PSR-ja je omogočiti razvijalcem, da izdelajo predpomnilno zavedne knjižnice, ki
se lahko integrirajo v obstoječa ogrodja in sisteme brez potrebe po
razvoju po meri.


## Definicije

*    **Klicna knjižnica** - Knjižnica ali koda, ki dejansko potrebuje storitve
predpomnilnika. Ta knjižnica bo uporabila storitve predpomnjenja, ki implementirajo
vmesnik tega standarda, vendar drugače ne bo poznala
implementacije teh storitev predpomnjenja.

*    **Izvedbena knjižnica** - Ta knjižnica je odgovorna za izvedbo
tega standarda, da zagotovi storitve predpomnjenja katerikoli klicni knjižnici.
Izvedbena knjižnica MORA zagotoviti razrede, ki implementirajo
vmesnika Cache\CacheItemPoolInterface in Cache\CacheItemInterface.
Izvedbene knjižnice MORAJO podpirati najmanj funkcionalnost TTL, kot je opisano
spodaj s celotno drugo razdrobljenostjo.

*    **TTL** - Življenska doba (TTL) elementa je količina časa medtem
ko je ta element shranjen in se smatra za nesvežega. TTL je običajno definiran
s celim številom, ki predstavlja čas v sekundah ali objektov DateInterval.

*    **Pretek** - Dejanski čas, ko je element nastavljen za potek. To je
običajno izračunano z dodajanjem TTL času, ko je objekt shranjen, vendar
je lahko tudi eksplicitno nastavljen z objektom DateTime.

    Element s TTL 300 sekund, shranjen ob 1:30:00 bo imel pretek ob
    1:35:00.

    Izvedbene knjižnice LAHKO potečejo element pred njegovim zahtevanim časom poteka,
vendar MORAJO obravnavati element kot potečen, ko je dosežen njegov čas poteka. Če klicna
knjižnica zaprosi, da je element shranjen vendar ne določa časa poteka ali
določa čas poteka null ali TTL, izvedbena knjižnica LAHKO uporabi nastavljeno
privzeto trajanje. Če privzeto trajanje ni bilo nastavljeno, MORA izvedbena knjižnica
to prevesti kot zahtevek, ki predpomni element za vedno ali pa dokler to
podpira spodnja implementacija.

*    **Ključ** - Niz vsaj enega znaka, ki unikatno identificira
predpomnjeni element. Izvedbene knjižnice MORAJO podpirati ključe sestavljene iz
znakov `A-Z`, `a-z`, `0-9`, `_` in `.` v kateremkoli vrstnem redu v kodiranju UTF-8 in
dolžine do 64 znakov. Izvedbene knjižnice LAHKO podpirajo dodatne
znake in kodiranja ali daljše dolžine, vendar morajo podpirati vsaj ta
minimum. Knjižnice so odgovorne za svoje lastne ubežne znake ključev nizov
kot je potrebno, vendar MORAJO biti sposobne vrniti prvotni nespremenjeni ključ niza.
Sledeči znaki so rezervirani za prihodnje razširitve in NE SMEJO biti
podprti v izvedbenih knjižnicah `{}()/\@:`

*    **Zadetek** - Zadetek predpomnilnika se zgodi, ko klicna knjižnica zahteva element po ključu,
je najdena ujeta vrednost za ta ključ, ta vrednost še ni potekla in
vrednost ni neveljavna zaradi kakršnegakoli razloga. Klicne knjižnice BI MORALE
zagotoviti preverjanje isHit() na vseh klicih get().

*    **Zgrešitev** - Zgrešitev predpomnilnika je nasprotje zadetka predpomnilnika. Zgrešitev se zgodi,
ko klicna knjižnica zahteva element po ključu in ta vrednost ni najdena za ta
ključ, ali je najdena vrednost potekla, ali pa je vrednost neveljavna zaradi
kakšnega drugega razloga. Pretečena vrednost MORA biti vedno smatrana za zgrešitev predpomnilnika.

*    **Posredni** - Posredno shranjevanje predpomnilnika navaja, da element predpomnilnika lahko ni
pridobljen takoj iz zaloge. Objekt Pool LAHKO zakasni pridobivanje posrednega
elementa predpomnilnika, da izkoristi operacije v večjem skupku, podprte z
nekaterimi motorji shranjevanja. Zaloga MORA zagotoviti, da katerikoli posredni elementi predpomnilnika so na koncu
pridobljeni in podatki niso izgubljeni in jih LAHKO pridobi pred klicna knjižnica
zahteva, da so pridobljeni. Ko klicna knjižnica uveljavi metodo commit(),
MORAJO biti pridobljeni vsi odprti posredni elementi. Izvedbena knjižnica
LAHKO uporabi katerokoli ustrezno logiko, da določi, kdaj pridobiti posredne
elemente, kot je destruktor objekta, pridobitev vsega pri save(), časovna omejitev ali
preverjanje največjega števila elementov ali katerakoli druga ustrezna logika. Zahtevki za predpomnjeni element,
ki je bil posredovan MORA vrniti posrednega vendar še ne pridobljenega elementa.


## Podatki

Izvedbene knjižnice MORAJO podpirati vse zaporednostne tipe PHP podatkov vključno z:

*    **Nizi** - Znakovni nizi arbitrarne velikosti v kateremkoli PHP-kompatibilnem kodiranju.
*    **Celimi števili** - Vsa cela števila katerekoli velikosti, podprta s strani PHP do 64-bitno podpisanih.
*    **Števili s plavajočo vejico** - Vse podpisane vrednosti števil s plavajočo vejico.
*    **Logičnimi vrednostmi** - True in False.
*    **Null** - Dejanska vrednost null.
*    **Polji** - Indeksirana, asociativna in večdimenzijska polja arbitrarne globine.
*    **Objekt** - Katerikoli objekt, ki podpira brezizgubno serializacijo in
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

## Ključni koncepti

### Zaloga

Zaloga (Pool) predstavlja zbirko elementov v sistemu predpomnjenja. Zaloga je
logični repozitorij vseh vsebovanih elementov. Vsi elementi predpomnjenja so pridobljeni
iz zaloge kot objekt Item in vse interakcije v celotnem vesolju
predpomnjenih objektov se zgodijo skozi zalogo.

### Elementi

Element predstavlja posamezen par ključ/vrednost znotraj zaloge. Ključ je glavni
unikatni identifikator za Item in MORA biti nespemenljiv. Value se LAHKO spremeni
kadarkoli.

## Upravljanje napak

Medtem ko je predpomnjenje pogosto pomemben del zmogljivosti aplikacije, ne bi nikoli smelo
biti kritični del funkcionalnosti aplikacije. Torej napaka v sistemu predpomnjenja NE BI SMELA
imeti za posledico odpoved aplikacije. Zato izvedbene knjižnice NE SMEJO
vreči izjem razen tistih definiranih v vmesniku in BI MORALE ujeti katerekoli napake
ali izjeme sprožene z osnovnim podatkovnim shranjevanjem podatkov in jim ne dovoliti, da so v mehurčkih.

Izvedbena knjižnica BI MORALA beležiti take napake ali pa jih poročati
administratorju kot je potrebno.

Če klicana knjižnica zahteva, da se en ali več elementov izbriše ali da je zaloga spraznjena,
se to NE SME smatrati kot pogoj napake, če določeni ključ ne obstaja.
Za pogojem je enako (ključ ne obstaja ali pa je zaloga spraznjena), torej ni
pogoj napake.

## Vmesniki

### CacheItemInterface

CacheItemInterface definira element znotraj sistema predpomnilnika. Vsak objekt element
MORA biti povezan z določenim ključem, ki je lahko nastavljen glede na
sistem implementacije in je običajno podan od objekta
Cache\CacheItemPoolInterface.

Objekt Cache\CacheItemInterface zaobjema shrambo in pridobitev
elementov predpomnilnika. Vsak Cache\CacheItemInterface je generiran z objektom
Cache\CacheItemPoolInterface, ki je odgovoren za kakršnekoli zahtevano
nastavitev kot tudi povezani objekt z unikatnim ključem.
Cache\CacheItemInterface objekti MORAJO biti zmožni shraniti in vrniti katerikoli tip
definiranih vrednosti PHP v sekciji podatkov tega dokumenta.

Klicne knjižnice NE SMEJO sprožiti objektov elementov same. Lahko so samo
zahtevane iz objekta Pool preko metode getItem(). Klicne knjižnice
NE BI SMELE predpostavljati, da je element ustvarjen z eno implementirano knjižnico
združljiv z zalogo iz druge knjižnice.

~~~php
namespace Psr\Cache;

/**
 * CacheItemInterface defines an interface for interacting with objects inside a cache.
 */
interface CacheItemInterface
{
    /**
     * Returns the key for the current cache item.
     *
     * The key is loaded by the Implementing Library, but should be available to
     * the higher level callers when needed.
     *
     * @return string
     *   The key string for this cache item.
     */
    public function getKey();

    /**
     * Retrieves the value of the item from the cache associated with this object's key.
     *
     * The value returned must be identical to the value originally stored by set().
     *
     * If isHit() returns false, this method MUST return null. Note that null
     * is a legitimate cached value, so the isHit() method SHOULD be used to
     * differentiate between "null value was found" and "no value was found."
     *
     * @return mixed
     *   The value corresponding to this cache item's key, or null if not found.
     */
    public function get();

    /**
     * Confirms if the cache item lookup resulted in a cache hit.
     *
     * Note: This method MUST NOT have a race condition between calling isHit()
     * and calling get().
     *
     * @return bool
     *   True if the request resulted in a cache hit. False otherwise.
     */
    public function isHit();

    /**
     * Sets the value represented by this cache item.
     *
     * The $value argument may be any item that can be serialized by PHP,
     * although the method of serialization is left up to the Implementing
     * Library.
     *
     * @param mixed $value
     *   The serializable value to be stored.
     *
     * @return static
     *   The invoked object.
     */
    public function set($value);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param \DateTimeInterface $expiration
     *   The point in time after which the item MUST be considered expired.
     *   If null is passed explicitly, a default value MAY be used. If none is set,
     *   the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAt($expiration);

    /**
     * Sets the expiration time for this cache item.
     *
     * @param int|\DateInterval $time
     *   The period of time from the present after which the item MUST be considered
     *   expired. An integer parameter is understood to be the time in seconds until
     *   expiration. If null is passed explicitly, a default value MAY be used.
     *   If none is set, the value should be stored permanently or for as long as the
     *   implementation allows.
     *
     * @return static
     *   The called object.
     */
    public function expiresAfter($time);

}
~~~

### CacheItemPoolInterface

Glavni namen Cache\CacheItemPoolInterface je sprejeti ključ iz
klicane knjižnice in vrniti povezani objekt Cache\CacheItemInterface.
Glavna točka interakcije s celotno zbirko predpomnilnika.
Vse nastavitve in sprožitev zaloge (Pool) so prepuščene implementirani
knjižnici.

~~~php
namespace Psr\Cache;

/**
 * CacheItemPoolInterface generates CacheItemInterface objects.
 */
interface CacheItemPoolInterface
{
    /**
     * Returns a Cache Item representing the specified key.
     *
     * This method must always return a CacheItemInterface object, even in case of
     * a cache miss. It MUST NOT return null.
     *
     * @param string $key
     *   The key for which to return the corresponding Cache Item.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return CacheItemInterface
     *   The corresponding Cache Item.
     */
    public function getItem($key);

    /**
     * Returns a traversable set of cache items.
     *
     * @param array $keys
     * An indexed array of keys of items to retrieve.
     *
     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return array|\Traversable
     *   A traversable collection of Cache Items keyed by the cache keys of
     *   each item. A Cache item will be returned for each key, even if that
     *   key is not found. However, if no keys are specified then an empty
     *   traversable MUST be returned instead.
     */
    public function getItems(array $keys = array());

    /**
     * Confirms if the cache contains specified cache item.
     *
     * Note: This method MAY avoid retrieving the cached value for performance reasons.
     * This could result in a race condition with CacheItemInterface::get(). To avoid
     * such situation use CacheItemInterface::isHit() instead.
     *
     * @param string $key
     *    The key for which to check existence.
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *  True if item exists in the cache, false otherwise.
     */
    public function hasItem($key);

    /**
     * Deletes all items in the pool.
     *
     * @return bool
     *   True if the pool was successfully cleared. False if there was an error.
     */
    public function clear();

    /**
     * Removes the item from the pool.
     *
     * @param string $key
     *   The key for which to delete
     *
     * @throws InvalidArgumentException
     *   If the $key string is not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the item was successfully removed. False if there was an error.
     */
    public function deleteItem($key);

    /**
     * Removes multiple items from the pool.
     *
     * @param array $keys
     *   An array of keys that should be removed from the pool.

     * @throws InvalidArgumentException
     *   If any of the keys in $keys are not a legal value a \Psr\Cache\InvalidArgumentException
     *   MUST be thrown.
     *
     * @return bool
     *   True if the items were successfully removed. False if there was an error.
     */
    public function deleteItems(array $keys);

    /**
     * Persists a cache item immediately.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   True if the item was successfully persisted. False if there was an error.
     */
    public function save(CacheItemInterface $item);

    /**
     * Sets a cache item to be persisted later.
     *
     * @param CacheItemInterface $item
     *   The cache item to save.
     *
     * @return bool
     *   False if the item could not be queued or if a commit was attempted and failed. True otherwise.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Persists any deferred cache items.
     *
     * @return bool
     *   True if all not-yet-saved items were successfully saved or there were none. False otherwise.
     */
    public function commit();
}
~~~

### CacheException

Ta vmesnik izjeme je namenjen uporabi, ko pride do kritičnih napak,
vključno vendar ni omejeno na *namestitev predpomnilnika*, kot je povezovanje na strežnik
predpomnilnika ali so podane neveljavne poverilnice.

Katerakoli izjema, ki jo vrže implementirana knjižnica, MORA implementirati ta vmesnik.

~~~php
namespace Psr\Cache;

/**
 * Exception interface for all exceptions thrown by an Implementing Library.
 */
interface CacheException
{
}
~~~

### InvalidArgumentException

~~~php
namespace Psr\Cache;

/**
 * Exception interface for invalid cache arguments.
 *
 * Any time an invalid argument is passed into a method it must throw an
 * exception class which implements Psr\Cache\InvalidArgumentException.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~
