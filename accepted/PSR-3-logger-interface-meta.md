# Dnevnik meta dokument

## 1. Povzetek

Vmesnik dnevnika definira skupni vmesnik za beleženje sistemskih sporočil iz
aplikacije ali knjižnice.

Ta meta dokument je bil napisan naknadno, ko je bil PSR-3 še sprejed preden so
meta dokumenti postali standardna praksa.

## 2. Načrtovalske odločitve

### Statična sporočila dnevnika

Namen te specifikacije je, da je sporočilo poslano metodi beleženja vedno
statična vrednost. Katera koli kontekstno specifična spremenljivost (kot je
uporabniško ime, časovni žig, ali druga informacija) bi morala biti podana samo
preko polja `$context` in niz bi moral uporabiti označbo mesta za njeno
sklicevanje.

Namen tega načrta je dvoplastni. Prvič, sporočilo je potem navoljo sistemom
prevajanja, da ustvarijo lokalizirane verzije sporočil dnevnika. Drugič,
kontekstno specifični podatki lahko vsebujejo uporabniški vnos in tako zahtevajo
ubežnik znak. Tako čiščenje bo obvezno drugačno, če je sporočilo dnevnika
shranjeno v podatkovni bazi za kasnejše upodabljanje v HTML, serializirano v
JSON, serializirano v niz sporočila syslog itd. Odgovornost izvedbe beleženja je
zagotoviti, da so podatki `$context`, ki so prikazani uporabniku, ustrezno
počiščeni.

## 3. Ljudje

### 3.1 Urednik(i)

* Jordi Boggiano

## 4. Glasovanje

[Glasovanje za odobritev](https://groups.google.com/g/php-fig/c/d0yPC7jWPAE/m/rhexAfz2T_8J)

## 5. Popravki napak

### 5.1 Dodatki tipov

Različica 2.0 paketa `psr/log` vključuje skalarne tipe parametrov. Različica 3.0
paketa vključuje povratne tipe. Ta struktura izkorišča podporo kovariance PHP
7.2, kar omogoča postopen proces nadgradnje, vendar zahteva PHP 8.0 za
kompatibilnost tipov.

Izvajalci LAHKO dodajo povratne tipe k svojim lastnim paketom po svoji presoji
pod pogojem, da:

* se povratni tip ujema s tistimi iz paketa 3.0.
* izvedba določa najmanj PHP verzijo 8.0.0 ali novejše.

Izvajalci LAHKO dodajo tipe parametrov k svojim lastnim paketom v večji izdaji,
bodisi istočasno z dodajanjem povratnih tipov ali pa v ločeni izdaji pod
pogojem, da:

* se tipi parametrov ujemajo s tistimi iz paketa 2.0.
* izvedba določa najmanjšo PHP verzijo 8.0.0 ali novejše.
* izvedba zavisi od `"psr/log": "^2.0 || ^3.0"`, tako da se izključi verzijo 1.0
  brez tipov.

Izvajalce se spodbuja, vendar od njih ni zahtevano, da njihovi paketi čimprej
preidejo na verzijo 3.0.
