---
title: "Bitcoin and inheritance, a House of the Dragon story"
slug: "bitcoin-and-house-of-dragon"
date: 2023-06-15
description: "A short fiction about how the Targaryens secure their funds across generations"
draft: false
topics:  ["bitcoin"]
---

# Bitcoin and inheritance, a House of the Dragon story

*House of the Dragon is an American fantasy drama television series.
A prequel to Game of Thrones (2011–2019), it is the second show in the franchise,
created by George R. R. Martin and Ryan Condal for HBO. Beware, this
article contains spoilers*.

Nobody is safe in the realm of the Seven Kingdom, known for the
brutality and the trickery of its subjects.
Seated on the Iron Throne, King Viserys I Targaryen must handle the burden
of the crown and the tumultuous behavior of his brother, *the Rogue
Prince*, Daemon Targaryen. The Targaryens are the most powerful family of
Westeros, the continent they conquered with the fire of their dragons
and their knowledge of the dark arcane of Bitcoin.

As usual in the Game of the Thrones, our story begins with a death.
The Queen consort, Aemma Arryn dies without giving birth to a male offspring.
The King is incited by his advisors to declare his Heir to assure that
the royal line is clear to all. Irritated by Daemon provocations, The
King chooses his daughter Rhaenyra instead despite the old misogynistic
tradition.


## Protecting the realm funds

Securing the fabulous treasure of the family is not an easy
task, but thanks to Bitcoin and cryptography the Targaryens do not have
to enchain one of their dragons on top of a mountain of Gold anymore.
Each Targaryen is educated from their young age to generate, secure and
hide (Sometimes between the scales of their own dragons) cryptographic
keys.

The Old King, Jaehaerys I Targaryen, did a secret ceremony with his
grand son Viserys after choosing him as his Heir. They setted up a simple
1 of 2 multisig and locked the family legacy in new Bitcoin outputs.
At this time, Viserys was a grown adult, married and aware of the
responsibilities to come.

```ascii
                            │
                            │
                    ┌───────┴───────┐
                    │     1 of 2    │

               Jaehaerys         Viserys
```

{{< sidenote >}}
[Miniscript](https://bitcoin.sipa.be/miniscript) is a language for
writing (a subset of) Bitcoin Scripts in a structured way, enabling analysis,
composition, generic signing and more.
{{< /sidenote >}}

The resulting [miniscript](https://bitcoin.sipa.be/miniscript/) descriptor:

{{< code lang="Scheme" >}}
wsh(multi(1,<Jaehaerys xpub>,<Viserys xpub>))
{{</ code >}}

## A simple Inheritance Plan

Rhaenyra is still young and Viserys is afraid of her adventurous nature,
She is the Heir but as long as Viserys is alive, He prefers to keep the
key of the treasure himself. But what if something happens to him ? Rhaenyra must
be able to somehow get control of the funds.

During a secret ceremony in front of the old skull of Balerion *the Black
Dread*, Viserys creates a new descriptor and shares it to his daughter,
then moves the funds to outputs with scripts derived from it.

```ascii
              Viserys

                 │
                 │
                 │
    52600 blocks │
           older │
                 │
                 │
                 │
                 │

             Rhaenyra

```
{{< sidenote >}}
Bitcoin supports absolute and relative time locks at the script level with
OP_CHECKLOCKTIMEVERIFY (OP_CLTV), and OP_CHECKSEQUENCEVERIFY (OP_CSV).
[Here](https://medium.com/summa-technology/bitcoins-time-locks-27e0c362d7a1), an excellent article explaining them.
{{< /sidenote >}}

The descriptor rules that only Viserys key can move funds immediatly,
but after a fixed number of block passed the key of Rhaenyra also becomes valid.
Viserys number of block choice was enough to represent 1 years of wait.

A set of keys that can move the funds at any time is called the primary path,
and any other sets of keys paired with a time lock are called recovery paths.

The new miniscript descriptor:

{{< code lang="Scheme" >}}
wsh(
  or_d(
    pk(<Viserys xpub>),
    and_v(
      v:pkh(<Rhaenyra xpub>),
      older(52600)
    )
  )
)
{{</ code >}}

The King with his new wallet with inheritance included have full
disposal of his funds but in case of premature death, funds will be
spendable in 1 years maximum by Rhaenyra. King Viserys must
move coins to new outputs before the sequence runs out, either passively
by his ministerial usage of the wallet with change outputs
or pro-actively by doing a transaction to himself like outputs
consolidation.

## A complex family

Viserys gets married a second time with the daughter of the Hand of the
King: Alicent. She gives him two sons: Aegon and Aemond targaryens.
Rumors spread to the country that Aegon should be the new Heir, but
Viserys does not change the succession order.
Sick by a mysterious illness and advised by his ministers,
Viserys wants to include the whole succession line in the bitcoin
scripts locking the realm's treasure.

```ascii
               Viserys

                  │
                  ├──────────────────┐
                  │                  │
     52600 blocks │     65000 blocks │
            older │            older │
                  │                  │
                                     │
              Rhaenyra               │
                                     │

                                   Aegon

```

An inheritance plan can include multiple recovery paths, each of them
becoming valid at different locking times.
As the script grows, miniscript helps the targaryens to handle
the new complexity:

{{< code lang="Scheme" >}}
wsh(
  or_i(
    and_v(
      v:pkh(<Aegon xpub>),
      older(65000)
    ),
    or_d(
      pk(<Viserys xpub>),
      and_v(
        v:pkh(<Rhaenyra xpub>),
        older(52600)
      )
    )
  )
)
{{</ code >}}


## A deterrent against assassination.

Rhaenyra is furious against her father. She confronts him and makes him
aware that she will be at risk of being assassinated between the time Viserys dies
of his sickness and the expiration of the sequence of block disallowing her to move the funds.
She wants for Daemon, her uncle, a fierce warrior and loyal friend, to hold a key along with
Aegon key. Rhaenyra is confident that in case she get murdered, Daemon will lock the funds
and will take no rest until her murderers are hanged on the King's Landing main place.

```ascii
               Viserys

                  │
                  ├──────────────────┐
                  │                  │
     52600 blocks │     65000 blocks │
            older │            older │
                  │                  │
                                     │
              Rhaenyra               │
                                     │
                            ┌────────┴─────────┐
                            │      2 of 2      │

                          Aegon             Daemon
```

{{< sidenote >}}
Miniscript can become tedious to read. Initiatives exist to facilitate
readibility and development like [min.sc](https://min.sc/)</a>
or visualize it: [miniscript.fun](https://miniscript.fun)
{{< /sidenote >}}

{{< code lang="Scheme" >}}
wsh(
  or_i(
    and_v(
      v:thresh(2,pkh(<Aegon xpub>),a:pkh(<Daemon xpub>)),
      older(65000)
    ),
    or_d(
      pk(<Viserys xpub>),
      and_v(
        v:pkh(<Rhaenyra xpub>),
        older(52600)
      )
    )
  )
)
{{</ code >}}


## A dispute resolution

Viserys wants to accept Rhaenyra solution but he knows too well the
explosive temperament of his brother and the ambition that any Targaryen
carries with him. Aegon is young and does not seems interested in
politics, Daemon could leverage his position to keep funds in hostage
by not collaborating and gain enormous power over the royal court.

As Viserys agony starts to be extremely painful, he starts having visions
and his religious faith makes him believe that including the grand
maester may be a good idea. He then adds a third recovery path with
the clerical power as a third party, that will be in charge of dispute resolution
between Aegon and Daemon.

```ascii
           Viserys

              │
              ├──────────────────┬────────────────────┐
              │                  │                    │
 52600 blocks │     65000 blocks │       65535 blocks │
        older │            older │              older │
              │                  │                    │
                                 │                    │
          Rhaenyra               │                    │
                                 │                    │
                        ┌────────┴─────────┐          │
                        │      2 of 2      │          │
                                                      │
                      Aegon             Daemon        │
                                                      │
                                                      │ 2 of 3
                                            ┌─────────┼───────────┐
                                            │         │           │

                                        Aegon       Daemon      Maester
```

Institutions and entreprise may use the same idea for their funds. Instead of
a decaying multisig with a threshold decreasing over time, new third parties
like insurance company or notary agencies are added to the policy.
The threshold can be increased to prevent malicious collaboration at the path scope
but the minimal number of key required for an attacker to retrieve the funds
will always stay related to the general scope according to the "weakest" path.

Behold the final descriptor of the royal family:

{{< code lang="Scheme" >}}
wsh(
  or_i(
    and_v(
      v:thresh(3,pkh(<Aegon xpub>),a:pkh(<Daemon xpub>),a:pkh(<Maester xpub>)),
      older(65535)
    ),
    or_i(
      and_v(
        v:thresh(2,pkh(<Aegon xpub>),a:pkh(<Daemon xpub>)),
        older(65000)
      ),
      or_d(
        pk(<Viserys xpub>),
        and_v(
          v:pkh(<Rhaenyra xpub>),
          older(52600)
        )
      )
    )
  )
)
{{</ code >}}

## The end of our story

Blinded by hate and biases, the Greens, Alicent family, prefer war and the risk of
loss of the realm funds than letting Rhaenyra sit on the throne.
During the weeks the old King Viserys died, a race begins between them and
the Blacks, Rhaenyra clan, to rally allies and gather troops while Aegon is
proclaimed King.

{{< sidenote >}}
Disclaimer: Bitcoin can not and will not liberate you. Only you can free yourself
for whatever it means to you.
Bitcoin is just a tool and a very bad one if you do not know how to use it.
{{< /sidenote >}}

Dear reader, pity the poor citizen of Westeros as they will suffer yet
another war from the powerfuls as a new peril from North is marching
upon them. For thousands of year, they live under a feudal system where
basics rights differs according to bloodline and nobility. May they
discover the secret power of the Targaryens and free themselves from any
crown by using an allodial money.

---

```ascii
     *                          O    .           ____
            *       /\            .   o        .'* *.'            *
       *           /*.\            o        __/_*_*(_
                  /----\     Advertisement   /o )   \ ,';     *
             ;`, /   ( o\        O .         \ ,'    ;  /
       *     \  ;    `, /       .    o        "-.__.'"\_;       *
             ;_/"`.__.-"       _________                   *        *
                             c(`       ')o
    *             *            \.     ,/             *         *
                              _//^---^\\_
```

If you are interested for integrating this kind of solution for your
entreprise or for your individual usage, Please go checkout WizardSardine
product [Liana](https://github.com/wizardsardine/liana).
