# Odpremnik dogodkov

Razpošiljanje dogodkov je pogost in dobro preizkušen mehanizem, ki omogoča
razvijalcem, da enostavno in dosledno vstavijo logiko v aplikacijo.

Cilj tega PSR je vzpostaviti skupen mehanizem za razširjanje in sodelovanje na
podlagi dogodkov, da se knjižnice in komponente lahko bolj svobodno ponovno
uporabljajo med različnimi aplikacijami in ogrodji.

Ključne besede "MORA", "NE SME", "ZAHTEVA", "PRIPOROČA", "LAHKO" in "NEOBVEZNO"
v tem dokumentu se tolmačijo, kot je navedeno v
[RFC 2119][].

[RFC 2119]: http://tools.ietf.org/html/rfc2119

## Cilj

Imeti skupne vmesnike za razpošiljanje in obdelavo dogodkov omogoča razvijalcem,
da ustvarjajo knjižnice, ki lahko na skupen način sodelujejo z različnimi
ogrodji in drugimi knjižnicami.

Nekaj primerov:

* Ogrodje za varnost, ki preprečuje shranjevanje/dostopanje do podatkov, kadar
  uporabnik nima dovoljenja.
* Skupen sistem za predpomnjenje celotnih strani.
* Knjižnice, ki razširjajo druge knjižnice, ne glede na ogrodje, v katerega so
  vgrajene.
* Paket za beleženje dogodkov, ki sledi vsem dejanjem, izvedenim v aplikaciji.

## Opredelitve

* **Dogodek** (angl. *event*) - Dogodek je sporočilo, ki ga ustvari *Oddajnik*.
  Lahko je poljuben objekt PHP.
* **Poslušalec** (angl. *listener*)  - Poslušalec je katerakoli izvedljiva koda
  PHP, ki pričakuje prejem dogodka. Enemu dogodku je lahko posredovanih nič ali
  več poslušalcev. Poslušalec LAHKO sproži tudi druge asinhrone dogodke, če tako
  želi.
* **Oddajnik** (angl. *emitter*) - Oddajnik je poljubna koda, ki želi
  razpošiljati dogodke. To je tudi znano kot "klicna koda". Ni predstavljeno s
  konkretno podatkovno strukturo, ampak se nanaša na uporabniški primer.
* **Razpošiljalec** (angl. *dispatcher*) - Razpošiljalec je servisni objekt, ki
  prejme objekt dogodka od oddajnika. Razpošiljalec je odgovoren za zagotovitev,
  da se dogodek preda vsem ustrezni poslušalcem, vendar mora za določitev
  odgovornih poslušalcev prenesti nalogo na ponudnika poslušalcev.
* **Ponudnik poslušalcev** (angl. *listener provider*) - Ponudnik poslušalcev je
  odgovoren za določanje, kateri poslušalci so ustrezni za določen dogodek,
  vendar sam NE SME klicati poslušalcev. Ponudnik poslušalcev lahko določi nič
  ali več ustreznih poslušalcev.

## Dogodki

Dogodki so objekti, ki delujejo kot enota komunikacije med oddajnikom in
ustreznimi poslušalci.

Objekti dogodkov se LAHKO spreminjajo, če uporabniški primer zahteva, da
poslušalci posredujejo informacije nazaj oddajniku. Vendar pa, če ni potrebna
takšna dvosmerna komunikacija, je PRIPOROČLJIVO, da je dogodek definiran kot
nespremenljiv (angl. *immutable*); t. j. definiran tako, da nima metod za
spreminjanje.

Izvajalci MORAJO predpostaviti, da bo enak objekt posredovan vsem poslušalcem.

PRIPOROČLJIVO, vendar NI OBVEZNO, je, da objekti dogodkov podpirajo izgubno
neobčutljivo serializacijo in deserializacijo;
`$event == unserialize(serialize($event))` naj bo true. Objekti LAHKO
izkoristijo vmesnik `Serializable` v PHP-ju, magične metode `__sleep()` ali
`__wakeup()`, ali podobno jezikovno funkcionalnost, če je primerno.

## Dogodki, ki se lahko ustavijo

**Dogodek, ki se lahko ustavi** je poseben primer dogodka, ki vsebuje dodatne
načine za preprečevanje nadaljnjega klica poslušalcev. To je označeno z
implementacijo vmesnika `StoppableEventInterface`.

Dogodek, ki implementira `StoppableEventInterface`, MORA vrniti `true` iz metode
`isPropagationStopped()`, ko je dogodek, ki ga predstavlja, zaključen. Izvajalec
razreda mora določiti, kdaj se to zgodi. Na primer, dogodek, ki zahteva ujemanje
objekta PSR-7 `RequestInterface` s pripadajočim objektom `ResponseInterface`,
lahko ima metodo `setResponse(ResponseInterface $res)`, ki jo poslušalec kliče,
kar povzroči, da `isPropagationStopped()` vrne `true`.

## Poslušalci

Poslušalec je lahko katerakoli izvedljiva koda PHP. Poslušalec MORA imeti en in
samo en parameter, ki je dogodek, na katerega odziva. Poslušalci naj bi
uporabljali poimenovanje parametrov tako, da je relevantno za njihov uporabniški
primer. To pomeni, da lahko poslušalec uporabi namigovanje tipa na vmesnik, da
označi, da je združljiv z vsakim tipom dogodka, ki implementira ta vmesnik, ali
pa uporabi namigovanje na specifično implementacijo tega vmesnika.

Poslušalec naj bi imel povratni tip `void` in naj eksplicitno označi ta povratni
tip. Razpošiljalec mora prezreti povratne vrednosti poslušalcev.

Poslušalec LAHKO prenese akcije na drugo kodo. To lahko vključuje to, da je
poslušalec tanek ovitek okoli objekta, ki izvaja dejansko poslovno logiko.

Poslušalec LAHKO prenese informacije iz dogodka za kasnejšo obdelavo s
sekundarnim procesom, uporabljajoč periodično opravilo (angl. *cron*), strežnik
vrstic ali podobne tehnike. LAHKO serializira sam objekt dogodka, da to doseže,
vendar je treba paziti, da vsi objekti dogodkov niso varno serializabilni.
Sekundarni proces MORA predpostaviti, da se spremembe, ki jih naredi na objektu
dogodka, NE bodo prenesle na druge poslušalce.

## Razpošiljalec

Razpošiljalec je servisni objekt, ki implementira vmesnik
`EventDispatcherInterface`. Odgovoren je za pridobivanje poslušalcev iz
ponudnika poslušalcev za razposlan dogodek ter klicanje vsakega poslušalca s tem
dogodkom.

Razpošiljalec:

* MORA sinhrono klicati poslušalce v vrstnem redu, v katerem so vrnjeni iz
  ponudnika poslušalcev.
* MORA po končanem klicanju poslušalcev vrniti isti objekt dogodka, kot je bil
  prejet.
* NE SME se vrniti oddajniku, dokler vsi poslušalci niso bili izvedeni.

Če je prejet ustavljiv dogodek, razpošiljalec:

* MORA pred klicanjem vsakega poslušalca poklicati `isPropagationStopped()` na
  dogodku. Če ta metoda vrne `true`, MORA takoj vrniti dogodek oddajniku in NE
  SME klicati nadaljnjih poslušalcev. To pomeni, da če je dogodek, ki ga prejme
  razpošiljalec, vedno vrne `true` iz `isPropagationStopped()`, noben poslušalec
  ne bo klican.

Razpošiljalec NAJ predpostavlja, da je vsak poslušalec, ki ga prejme od
ponudnika poslušalcev, tip-varno. To pomeni, da naj predpostavi, da klic
`$listener($event)` ne bo povzročil `TypeError`.

### Ravnanje z napakami

Izjema ali napaka, ki jo sproži poslušalec, MORA blokirati izvajanje nadaljnjih
poslušalcev. Izjema ali napaka, ki jo sproži poslušalec, MORA biti dovoljena, da
se širi nazaj do oddajnika.

Razpošiljalec LAHKO ujame sproženi objekt, da ga zabeleži, omogoči dodatno
ukrepanje ipd., vendar nato MORA ponovno sprožiti prvotni objekt izjeme ali
napake.

## Ponudnik poslušalcev (angl. *Listener Provider*)

Ponudnik poslušalcev je servisni objekt, odgovoren za določanje, kateri
poslušalci so pomembni za določen dogodek in kateri naj bodo klicani. Lahko
določi tako pomembne poslušalce kot tudi vrstni red, v katerem jih vrne, na
kakršen koli način se odloči. To LAHKO vključuje:

* Omogočanje neke vrste registracijskega mehanizma, da lahko izvajalci določijo
  poslušalca za dogodek v fiksnem vrstnem redu.
* Izpeljava seznama ustrezni poslušalcev s pomočjo odseva glede na vrsto in
  implementirane vmesnike dogodka.
* Predhodno ustvarjanje seznama poslušalcev, ki se ga lahko preveri med
  izvajanjem.
* Implementacija nadzora dostopa, tako da bodo določeni poslušalci klicani samo,
  če trenutni uporabnik ima določeno dovoljenje.
* Izvleček nekaterih informacij iz objekta, na katerega se sklicuje dogodek, kot
  je entiteta, in klicanje vnaprej določenih metod življenjskega cikla na tem
  objektu.
* Delegiranje svoje odgovornosti enemu ali več drugim ponudnikom poslušalcev z
  uporabo neke poljubne logike.

Lahko se uporabi katera koli kombinacija zgoraj navedenih ali drugih mehanizmov
po želji.

Ponudniki poslušalcev NAJ uporabljajo ime razreda dogodka za razlikovanje med
eno in drugo dogodkov. Prav tako LAHKO upoštevajo tudi druge informacije o
dogodku, če je primerno.

Ponudniki poslušalcev MORAJO enako obravnavati nadrejene vrste kot lastno vrsto
dogodka pri določanju uporabnosti poslušalca. V naslednjem primeru:

```php
class A {}

class B extends A {}

$b = new B();

function listener(A $event): void {};
```

Ponudnik poslušalcev MORA obravnavati `listener()` kot ustrezen poslušalec za
`$b`, saj je združljiv po tipu, razen če drugi kriteriji preprečujejo njegovo
uporabo.

## Sestavljanje objektov

Razpošiljalec NAJ sestavi ponudnika poslušalcev za določanje relevantnih
poslušalcev. Priporočljivo je, da je ponudnik poslušalcev implementiran kot
ločen objekt od razpošiljalca, vendar to NI OBVEZNO.

## Vmesniki

```php
namespace Psr\EventDispatcher;

/**
 * Defines a dispatcher for events.
 */
interface EventDispatcherInterface
{
    /**
     * Provide all relevant listeners with an event to process.
     *
     * @param object $event
     *   The object to process.
     *
     * @return object
     *   The Event that was passed, now modified by listeners.
     */
    public function dispatch(object $event);
}
```

```php
namespace Psr\EventDispatcher;

/**
 * Mapper from an event to the listeners that are applicable to that event.
 */
interface ListenerProviderInterface
{
    /**
     * @param object $event
     *   An event for which to return the relevant listeners.
     * @return iterable<callable>
     *   An iterable (array, iterator, or generator) of callables.  Each
     *   callable MUST be type-compatible with $event.
     */
    public function getListenersForEvent(object $event) : iterable;
}
```

```php
namespace Psr\EventDispatcher;

/**
 * An Event whose processing may be interrupted when the event has been handled.
 *
 * A Dispatcher implementation MUST check to determine if an Event
 * is marked as stopped after each listener is called.  If it is then it should
 * return immediately without calling any further Listeners.
 */
interface StoppableEventInterface
{
    /**
     * Is propagation stopped?
     *
     * This will typically only be used by the Dispatcher to determine if the
     * previous listener halted propagation.
     *
     * @return bool
     *   True if the Event is complete and no further listeners should be called.
     *   False to continue calling listeners.
     */
    public function isPropagationStopped() : bool;
}
```
