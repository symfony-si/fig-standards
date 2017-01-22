---
title: "PSR-13: Povezave meta dokument"
description: ""
read_time: "5 min"
updated: "2017-01-11"
slug: "psr-13-povezave-meta-dokument"
---

# Link Definition Meta Document

## 1. Povzetek

Hipermedijske povezave postajajo vedno pomembnejši del spleta, tako v kontekstu HTML
kot v različnih kontekstih oblik API. Vendar ne obstaja posamezna skupna oblika hipermedijev,
niti ni na voljo skupnega načina za ponazoritev povezav med oblikami.

Ta specifikacija cilja ponuditi PHP razvijalcem enostaven, skupni način predstavitve
hipermedijskih povezav neodvisno od oblike serializacije, ki je uporabljena. To omogoča
sistem za serializacijo odziva s hipermedijskimi povezavami v enega ali več povezljivih oblik neodvisno
od procesa odločanja, katere povezave naj bi to bile.

## 2. Obseg

### 2.1 Cilji

* Ta specifikacija cilja izvleči in standardizirati ponazoritev hipermedijske povezave med različnimi
oblikami.

### 2.2 Niso cilji

* Ta specifikacija ne išče standardizacije ali naklonjenosti kakršnikoli posebni obliki serializacije.

## 3. Odločitve načrta

### Zakaj ni metod mutacij?

Eni izmed ključnih ciljev te specifikacije so PSR-7 objeki odziva. Objekti odziva morajo biti v zasnovi
nespremenljivi. Ostale implementacije vrednost-objekt bi verjetno tudi zahtevale nespremenljiv vmesnik.

Dodatno, nekaj objektov
Additionally, some Link Provider objects may not be value objects but other objects within a given
domain, which are able to generate Links on the fly, perhaps off of a database result or other underlying
representation.  In those cases a writeable provider definition would be incompatible.

Therefore, this specification splits accessor methods and evolvable methods into separate interfaces,
allowing objects to implement just the read-only or evolvable versions as appropriate to their use case.

### Why is rel on a Link object multi-value?

Different hypermedia standards handle multiple links with the same relationship differently. Some have a single
link that has multiple rel's defined. Others have a single rel entry that then contains multiple links.

Defining each Link uniquely but allowing it to have multiple rels provides a most-compatible-denominator definition.
A single LinkInterface object may be serialized to one or more link entries in a given hypermedia format, as
appropriate.  However, specifying multiple link objects each with a single rel yet the same URI is also legal, and
a hypermedia format can serialize that as appropriate, too.

### Why is a LinkProviderInterface needed?

In many contexts, a set of links will be attached to some other object.  Those objects may be used in situations
where all that is relevant is their links, or some subset of their links. For example, various different value
objects may be defined that represent different REST formats such as HAL, JSON-LD, or Atom.  It may be useful
to extract those links from such an object uniformly for further processing. For instance, next/previous links
may be extracted from an object and added to a PSR-7 Response object as Link headers.  Alternatively, many links
would make sense to represent with a "preload" link relationship, which would indicate to an HTTP 2-compatible
web server that the linked resources should be streamed to a client in anticipation of a subsequent request.

All of those cases are independent of the payload or encoding of the object. By providing a common interface
to access such links, we enable generic handling of the links themselves regardless of the value object or
domain object that is producing them.

## 4. Ljudje

### 4.1 Urednik(i)

* Larry Garfield

### 4.2 Sponzorji

* Matthew Weier O'Phinney (koordinator)
* Marc Alexander

### 4.3 Prispevali so

* Evert Pot

## 5. Glasovi

## 6. Pomembne povezave

* [Kaj je v povezavi?](http://evertpot.com/whats-in-a-link/) by Evert Pot
* [Seznam delovne skupine FIG Link](https://groups.google.com/forum/#!forum/php-fig-link)
