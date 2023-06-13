# Meta dokument odpremnika dogodkov

## 1. Povzetek

Namen tega dokumenta je opisati razloge in logiko za specifikacijo dogodkovnega
dodeljevalnika.

## 2. Zakaj se truditi?

Številne knjižnice, komponente in ogrodja že dolgo podpirajo mehanizme, ki
omogočajo poljubni kodi tretjih oseb, da z njimi komunicira. Večina teh
mehanizmov temelji na klasičnem opazovalčevem vzorcu, pogosto pa se posreduje
prek posredniškega predmeta ali storitve. Drugi uporabljajo pristop s
programiranjem z aspektno usmerjenim programiranjem (AOP). Vendar imajo vsi enak
osnovni koncept: prekinitev poteka programa na določeni točki za zagotavljanje
informacij izvedenega dejanja poljubnim knjižnicam tretjih oseb in omogočanje,
da te knjižnice reagirajo ali vplivajo na obnašanje programa.

To je uveljavljen model, vendar standardni mehanizem, s katerim lahko knjižnice
to storijo, jim bo omogočil interoperabilnost z vse bolj raznolikimi knjižnicami
tretjih oseb z manj truda tako za prvotnega razvijalca kot za razvijalce
razširitev.

## 3. Obseg

### 3.1 Cilji

* Poenostaviti in standardizirati postopek, s katerim lahko knjižnice in
  komponente prek "dogodkov" izpostavijo sebe razširitvam, da jih je mogoče
  lažje vključiti v aplikacije in ogrodja.
* Poenostaviti in standardizirati postopek, s katerim se lahko knjižnice in
  komponente registrirajo za odzivanje na dogodek, da jih je mogoče lažje
  vključiti v poljubne aplikacije in ogrodja.
* V največji možni meri olajšati postopek prehoda obstoječih kodnih osnov k tej
  specifikaciji.

### 3.2 Niso cilji

* Asinhroni sistemi pogosto vključujejo koncept "zanke dogodkov" za upravljanje
  medsebojnega prepletanja korutin. To je nepovezano in izrecno irelevantno za
  to specifikacijo.
* Shranjevalni sistemi, ki izvajajo vzorec "vira dogodkov", imajo tudi koncept
  "dogodka". To ni povezano z dogodki, o katerih se razpravlja tukaj, in izrecno
  ni vključeno v obseg.
* Stroga vzvratna združljivost z obstoječimi dogodkovnimi sistemi ni prioriteta
  in se je ne pričakuje.
* Čeprav bo ta specifikacija nedvomno predlagala vzorce implementacije, ne
  poskuša definirati ene same prave implementacije dodeljevalnika dogodkov,
  ampak samo kako klicatelji in poslušalci komunicirajo s tem dodeljevalnikom.

## 4. Pristopi

### 4.1 Upoštevani primeri uporabe

Delovna skupina je identificirala štiri možne delovne postopke za posredovanje
dogodkov, ki temeljijo na uporabnih primerih, opaženih v različnih sistemih.

* Obvestilo enosmerne komunikacije. ("Naredil sem nekaj, če vas zanima.")
* Obogatitev objekta. ("Tukaj je nekaj, prosim, spremenite ga, preden nekaj
  naredim s tem.")
* Zbiranje. ("Dajte mi vse svoje stvari, da lahko nekaj naredim s tem
  seznamom.")
* Alternativna veriga. ("Tukaj je nekaj; prvi izmed vas, ki se ga lahko loti,
  naj to stori in se ustavi.")

Po nadaljnjem pregledu je delovna skupina ugotovila, da:

* Zbiranje predstavlja poseben primer obogatitve objekta (zbirka je objekt, ki
  je obogaten).
* Alternativna veriga je podobno poseben primer obogatitve objekta, saj je
  podpis enak in delovni postopek je skoraj enak, čeprav z dodatnim
  preverjanjem.
* Obvestilo enosmerne komunikacije je degeneriran primer drugih postopkov ali pa
  se lahko tako predstavi.

Čeprav se v konceptu obvestilo enosmerne komunikacije lahko izvede asinhrono
(vključno z zamikom prek čakalne vrste), v praksi obstaja malo izrecnih izvedb
tega modela, kar zagotavlja manj mest, od katerih lahko dobimo smernice o
podrobnostih (kot je pravilno ravnanje s napakami). Po temeljitem premisleku je
delovna skupina izbrala, da ne zagotovi izrecno ločenega delovnega postopka za
obvestilo enosmerne komunikacije, saj se lahko ustrezno predstavi kot
degeneriran primer drugih postopkov.

### 4.2 Primeri uporabe

* Označevanje spremembe v konfiguraciji sistema ali uporabniške akcije ter
  omogočanje drugim sistemom, da reagirajo na načine, ki ne vplivajo na potek
  programa (npr. pošiljanje e-pošte ali beleženje akcije).
* Podajanje objekta nizu poslušalcev (Listener) za omogočanje njegovega
  spreminjanja pred shranjevanjem v sistem za vztrajnost.
* Podajanje zbirke (collection) nizu poslušalcev (Listener) za omogočanje
  registracije vrednosti ali spreminjanja obstoječih vrednosti, da lahko
  oddajnik (Emitter) deluje na vseh zbranih informacijah.
* Podajanje nekega kontekstualnega informacijskega niza poslušalcev, da lahko
  vsi skupaj "glasujejo" o tem, katero dejanje naj se izvede, pri čemer odloči
  Emitter na podlagi zagotovljenih zbranih informacij.
* Podajanje objekta nizu poslušalcev in omogočanje kateremu koli poslušalcu, da
  prekine proces pred drugimi poslušalci.

### 4.3 Nespremenljivi dogodki

Na začetku si je delovna skupina želela definirati vse dogodke kot
nespremenljive sporočilne objekte, podobne PSR-7. Vendar se je to izkazalo za
problematično pri vseh razen enosmernem obvestilnem primeru. V drugih scenarijih
so poslušalci potrebovali način za vračanje podatkov klicatelju. Konceptualno so
obstajale tri možnosti:

* Narediti dogodek spremenljiv in ga spreminjati na mestu.
* Zahtevati, da so dogodki evolucijski (nespremenljivi, vendar z metodami
  `with*()` kot PSR-7 in PSR-13) in da poslušalci vrnejo dogodek, da ga lahko
  posredujejo naprej.
* Narediti dogodek nespremenljiv, vendar združiti in vrniti vrnjeno vrednost iz
  vsakega poslušalca.

Vendar pa so potrebovali ustavljivi dogodki (alternativni verižni primer) tudi
kanal, s katerim bi lahko označili, da se dodatni poslušalci ne bi smeli
klicati. To bi lahko storili na dva načina:

* Spreminjanje dogodka (na primer klicanje metode `stopPropagation()`)
* Vrnitev kontrolne vrednosti iz poslušalca (`true` ali `false`), da se označi,
  da se širjenje ustavi.
* Razvijanje dogodka, da se ustavi (`withPropagationStopped()`)

Vsaka od teh možnosti ima slabosti. Prva možnost pomeni, da morajo biti dogodki
vsaj za namen označevanja statusa širjenja spremenljivi. Druga zahteva, da
poslušalci vrnjo vrednost, vsaj ko nameravajo preprečiti širjenje dogodka. To bi
lahko imelo posledice pri obstoječih knjižnicah in morebitne težave glede
dokumentacije. Tretja zahteva, da poslušalci vedno vrnejo dogodek ali
spremenjeni dogodek, kar bi zahtevalo, da dispečerji preverjajo, ali je vrnjena
vrednost enake vrste kot vrednost, ki jo je poslušalec prejel. To dejansko
nalaga breme tako porabnikom kot izvajalcem, kar lahko povzroči več potencialnih
integracijskih težav.

Poleg tega je bila želena lastnost sposobnost izvedbe odločitve preprečiti
širjenje na podlagi vrednosti, ki jih zberejo poslušalci. (Na primer, prekiniti,
ko eden izmed njih zagotovi določeno vrednost ali po tem, ko vsaj trije označijo
zastavico "zavrni ta zahtevek" ali podobno.) Tehnično je mogoče to izvesti kot
evolucijski objekt, vendar je takšno obnašanje intrinzično s stanjem, zato bi
bilo zelo nerodno tako za izvajalce kot za uporabnike.

Imeti poslušalce, ki vračajo razvijajoče se dogodke, je prav tako predstavljalo
izziv. Ta vzorec ni uporabljen v nobeni znani implementaciji v PHP-ju ali
drugje. Poleg tega se zanaša na to, da si poslušalec zapomni, da vrne dogodek
(dodatno delo za avtorja poslušalca) in ne vrne drugega, novega predmeta, ki
morda ni v celoti združljiv s kasnejšimi poslušalci (na primer podrazred ali
nadrazred dogodka).

Nepremični dogodki se tudi zanašajo na to, da avtor dogodka spoštuje napotek, da
je nespremenljiv. Dogodki so po naravi zelo ohlapno zasnovani, in možnost, da
izvajalci ignorirajo ta del specifikacije, tudi nenamerno, je visoka.

To je pustilo dve možnosti:

* Dovoliti, da so dogodki spremenljivi.
* Zahtevati, vendar ne biti sposoben prisiliti, da so dogodki nespremenljivi, z
  vmesnikom z visoko stopnjo slovesnosti, več dela za avtorje poslušalcev in
  večjo možnostjo za napake, ki jih morda ni mogoče zaznati med prevajanjem.

Z "visoko stopnjo slovesnosti" nakazujemo, da bi bila zahtevana zapletena
sintaksa in/ali implementacija. V prvem primeru bi morali avtorji poslušalcev
(a) ustvariti nov primer dogodka s preklopom zastavice razširjanja in (b) vrniti
nov primer dogodka, da bi ga lahko dodeljevalnik preveril:

```php
function (SomeEvent $event) : SomeEvent
{
    // do some work
    return $event->withPropagationStopped();
}
```

V slednjem primeru bi implementacije dodeljevalnika zahtevale preverjanje
vrednosti, ki jo vrne funkcija:

```php
foreach ($provider->getListenersForEvent($event) as $listener) {
    $returnedEvent = $listener($event);

    if (! $returnedEvent instanceof $event) {
        // This is an exceptional case!
        //
        // We now have an event of a different type, or perhaps nothing was
        // returned by the listener. An event of a different type might mean:
        //
        // - we need to trigger the new event
        // - we have an event mismatch, and should raise an exception
        // - we should attempt to trigger the remaining listeners anyway
        //
        // In the case of nothing being returned, this could mean any of:
        //
        // - we should continue triggering, using the original event
        // - we should stop triggering, and treat this as a request to
        //   stop propagation
        // - we should raise an exception, because the listener did not
        //   return what was expected
        //
        // In short, this becomes very hard to specify, or enforce.
    }

    if ($returnedEvent instanceof StoppableEventInterface
        && $returnedEvent->isPropagationStopped()
    ) {
        break;
    }
}
```

V obeh situacijah bi uvedli več možnih mejnih primerov z malo koristi in malo
mehanizmov na ravni jezika, ki bi usmerjali razvijalce k pravilni
implementaciji.

Glede na te možnosti je delovna skupina menila, da so spremenljivi dogodki
varnejša alternativa.

Kljub temu *ni zahteve, da mora biti dogodek spremenljiv*. Implementatorji bi
morali zagotoviti metode mutatorja na objektu dogodka *le, če je to nujno
potrebno* in primerno za trenutni uporabni primer.

### 4.4 Registracija poslušalcev

Poskusi med razvojem specifikacije so ugotovili, da obstaja širok nabor
veljavnih, legitimnih načinov, s katerimi bi lahko Dispatcher bil obveščen o
poslušalcu. Poslušalec:

* lahko je izrecno registriran;
* lahko je izrecno registriran na podlagi odseva njegovega podpisa;
* lahko je registriran z numeričnim vrstnim redom prednosti;
* lahko je registriran z uporabo mehanizma pred/po, ki omogoča natančnejše nadzorovanje vrstnega reda;
* lahko je registriran iz kontejnerja storitev;
* lahko uporablja predkompilacijski korak za generiranje kode;
* lahko temelji na imenih metod na objektih v samem dogodku;
* lahko je omejen na določene situacije ali kontekste na podlagi poljubno kompleksne logike (samo za določene uporabnike, samo ob določenih dnevih, samo če so prisotne določene sistemske nastavitve itd.).

Ti in drugi mehanizmi obstajajo v PHP-ju in so veljavni uporabni primeri, ki jih
je vredno podpirati, in le redko kateri od njih se lahko priročno predstavi kot
poseben primer drugega. Drugače povedano, standardizacija enega načina ali celo
majhnega nabora načinov za obveščanje sistema o poslušalcu se je izkazala za
nepraktično, če ne celo za nemogočo, ne da bi pri tem omejili številne uporabne
primere, ki jih je treba podpreti.

Delovna skupina je zato izbrala, da registracijo poslušalcev zapre vmesnik
`ListenerProviderInterface`. Objekt ponudnika lahko vsebuje na voljo mehanizem
za izrecno registracijo, lahko ima več takih mehanizmov ali pa nima nobenega.
Lahko je tudi generirana koda, ki jo ustvari nek predkompilacijski korak. Vendar
pa s tem razdelimo odgovornost upravljanja postopka pošiljanja dogodka in
postopka preslikave dogodka na poslušalce. Na ta način se lahko različne
implementacije mešajo in ujemajo z različnimi mehanizmi ponudnikov, kot je
potrebno.

Celotna integracija različnih poslušalcev se lahko doseže tudi z dovoljenjem in
potencialnim priporočilom, da knjižnice vključujejo svoje ponudnike, ki se
združijo v skupnega ponudnika in vrnejo poslušalce dispečerju. To je ena od
možnosti za obravnavanje poljubne registracije poslušalcev v okviru poljubnega
ogrodja, čeprav delovna skupina jasno navaja, da to ni edina možnost.

Čeprav je združitev dispečerja in ponudnika v en sam objekt veljaven in dovoljen
degeneriran primer, vendar NI PRIPOROČLJIV, saj zmanjšuje prilagodljivost
integratorjev sistema. Namesto tega naj bo ponudnik sestavljen kot odvisni
objekt.

### 4.5 Odloženi poslušalci

Specifikacija zahteva, da morajo biti vsi klicljivi elementi, ki jih vrne
ponudnik, klicani (razen če je propagacija izrecno ustavljena), preden se
vrne dispečer. Vendar pa specifikacija izrecno navaja, da lahko poslušalci
dogodke v vrsti čakajo na kasnejšo obdelavo namesto, da bi takoj ukrepali. Prav
tako je povsem dopustno, da ponudnik sprejme registracijo klicljivega elementa,
vendar ga ovije v drug klicljiv element, preden ga vrne dispečerju. (V tem
primeru je ovijalni element poslušalec z vidika dispečerja.) To omogoča, da so
naslednja vedenja zakonita:

* Ponudniki vračajo klicljive elemente poslušalcev, ki so jim bili predani.
* Ponudniki vračajo klicljive elemente, ki ustvarijo vnose v vrsti, ki bodo na
  dogodek reagirali z drugim klicljivim elementom v kasnejšem času.
* Poslušalci sami ustvarijo vnos v vrsti, ki bo na dogodek reagiral v kasnejšem
  času.
* Poslušalci ali ponudniki lahko sprožijo asinhrono nalogo, če delujejo v
  okolju, ki podpira asinhrono obnašanje (če rezultat asinhrone naloge ni
  potreben za oddajnik.)
* Ponudniki lahko na poslušalcih selektivno izvajajo takšno odlašanje ali
  ovijanje na podlagi poljubne logike.

Končni rezultat je, da so ponudniki in poslušalci odgovorni za določanje, kdaj
je varno odložiti odziv na dogodek do kasnejšega časa. V tem primeru ponudnik
ali poslušalec izrecno ne more vrniti pomembnih podatkov nazaj oddajniku, vendar
je delovna skupina ugotovila, da sta v najboljšem položaju, da ve, ali je to
varno storiti.

Čeprav je to tehnično stranski učinek zasnove, gre v bistvu za isti pristop, ki
ga uporablja Laravel (od različice 5) in se je izkazal v praksi.

### 4.6 Vrnjene vrednosti

Po specifikaciji mora dispečer VEDNO vrniti dogodek, ki ga je prejel od
oddajnika. To je določeno, da uporabnikom omogoča bolj ergonomsko izkušnjo in
omogoča uporabo bližnjic, podobnih naslednjim:

```php
$event = $dispatcher->dispatch(new SomeEvent('some context'));

$items = $dispatcher->dispatch(new ItemCollector())->getItems();
```

Vmesnik `EventDispatcher::dispatch()` pa ni določen s tipom vrnjene vrednosti.
To je predvsem zaradi združljivosti nazaj s obstoječimi implementacijami, da jim
olajša sprejetje novega vmesnika. Poleg tega, ker lahko dogodki predstavljajo
poljubne objekte, bi bil edini možni tip vrnjene vrednosti `object`, kar bi
zagotovilo le minimalno (čeprav ne-ničelno) vrednost, saj takšna deklaracija
tipa ne bi zagotovila uporabnih informacij za razvojna orodja IDE in ne bi
učinkovito preverjala, ali je vrnjen isti dogodek. Vrne metode so torej ostale
sintaktično neomejene. Kljub temu je še vedno zahteva, da se iz metode
`dispatch()` vrne isti objekt dogodka in nepopravljivo kršenje specifikacije
je, če se tega ne izvede.

## 5. Ljudje

Delovno skupino upravljalnika dogodkov sestavljajo:

### 5.1 Urednik

* Larry Garfield

### 5.2 Sponzor

* Cees-Jan Kiewiet

### 5.3 Člani delovne skupine

* Benjamin Mack
* Elizabeth Smith
* Ryan Weaver
* Matthew Weier O'Phinney

## 6. Glasovi

* [Vstopno glasovanje](https://groups.google.com/d/topic/php-fig/6kQFX-lhuk4/discussion)
* [Začetek obdobja pregleda](https://groups.google.com/d/topic/php-fig/sR4oEQC3Gz8/discussion)
* [Sprejetje](https://groups.google.com/d/topic/php-fig/o4ZSu7vJi2w/discussion)

## 7. Ustrezne povezave

* [Nit poštnega seznama za navdih][]
* [Vstopno glasovanje][]
* [Neformalna anketa o strukturi paketa][]
* [Neformalna anketa o strukturi poimenovanja][]

[Nit poštnega seznama za navdih]: https://groups.google.com/forum/#!topic/php-fig/-EJOStgxAwY
[Vstopno glasovanje]: https://groups.google.com/d/topic/php-fig/6kQFX-lhuk4/discussion
[Neformalna anketa o strukturi paketa]: https://docs.google.com/forms/d/1fvhYUH6xvPgJ1UW9I-3pMGPUtxkt5_Ph6_x_3qXHIuM/edit#responses
[Neformalna anketa o strukturi poimenovanja]: https://docs.google.com/forms/d/1Rs6APuwNx4k2VzJbTgieeNvN48kLu7CG8qn6Dd2FhTw/edit#responses
