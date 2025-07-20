---
title: "Hardware wallet for signing PLC operations"
slug: "hardware-wallet-for-signing-plc-operations"
date: 2025-07-17
description: "Hardware wallet for signing PLC operations"
draft: false
topics:  ["atproto", "bluesky"]
---

# Hardware wallet for PLC rotation key

[Atproto](https://atproto.com), the protocol behind Bluesky, allows for
the decentralization of many components of the platform.
Today, I want to discuss how it manages identity
with public key distribution and key rotation and introduce a
new application I am developing.

Currently, a Bluesky identity is attached to a global identifier (did:plc)
stored on one centralized server, the PLC directory. This identifier resolves to
a set of public keys, which we will discuss later, and a pointer to the Personal
Data Server (PDS) where the user content is stored.

On the PDS, user content is cryptographically signed by an asymmetrical key,
often hosted on the same server. Its public key is stored in the PLC directory
under a global identifier. This way, the ecosystem has a link between a user
identity and their content.

{{< sidenote >}}
This graph is from the identity section of the [AtProto documentation](https://atproto.com/guides/identity)
{{</ sidenote >}}

```ascii
┌──────────────────┐                 ┌───────────────┐
│ DNS name         ├──resolves to──→ │ DID           │
│ (alice.host.com) │                 │ (did:plc:...) │
└──────────────────┘                 └─────┬─────────┘
       ↑                                   │
       │                               resolves to
       │                                   │
       │                                   ↓
       │                            ┌───────────────┐
       └───────────references───────┤ DID Document  │
                                    │ {"id":"..."}  │
                                    └───────────────┘
```
If a user wants to migrate to another server, he has to change the pointer to the
new PDS and update the signing key under the did:plc to the one hosted on the
new server.

To do this, they must provide a document describing the change to the
[PLC directory](https://plc.directory), which will accept it if a signature from one of the public keys
is attached. I am talking about the rotation keys under the user did:plc.

{{< sidenote >}}
Example of did plc document with the previous document hash and the signature of one
of the previous rotation key.
{{</ sidenote >}}

{{< code lang="nix" >}}
{
    "createdAt": "2025-03-20T18:15:14.063Z",
    "pdsEndpoint": "https://pds.edouard.paris",
    "handles": [
      "edouard.paris"
    ],
    "rotationKeys": [
      "did:key:zQ3shrf6TuxEgDwLfwbLoeTzi1igE1Jj5pRNG5YPtA3L6eHBU"
    ],
    "verificationMethods": {
      "atproto": "did:key:zQ3shjAFHeumTiKkdTBnbmHUUjirMQ6TYoLSrtoLby8aPRpFG"
    },
    "prev": "bafyreibedmvcbdlkzkfjxwqhdrkzjczfvseatuowt45evw4k63ltn7mquq",
    "cid": "bafyreibzbeffwnjun7ibmdqrjceq2ie46rpbr2nfdimisf3o5fkxmxtzyu"
}
{{< / code >}}



If the user want to have control over their identity on the PLC directory, these
rotation keys are crucial. By default, Bluesky manages one for them on the
Bluesky PDS, but **if they want to take control, they need to generate a key that
only they possess and share the public key with the PLC directory, assigning it a
higher priority level than the others.** The highest priority key can override any
change signed by others.

In summary, they need to generate a key and keep it safe. If they are particularly
security conscious, they should only use it on a secure device that protects them
against key extraction and prevents blind signing with a screen. This is a
problem that other technologies have faced.

## What is a hardware wallet ?

Hardware wallets are the name given in the cryptocurrency sector to hardware devices
that hold secret keys. They generally contain a chip known as a secure element,
which handles cryptography for the user, eliminating the need to trust the laptop.

I took the opportunity to use one of these devices with the largest screen
(the Ledger Stax) to demonstrate an app I am developing to manage PLC operations.

## A Vanadium app for Ledger device that manages PLC operations

The current code can be found on [tangled.sh](https://tangled.sh/@edouard.paris/vnd-atproto).

The application is different than a usual native Ledger app. It is written in rust
to be compiled in the riscv target and then interpreted by a Virtual Machine managed
by a native Ledger app called [Vanadium](https://github.com/LedgerHQ/vanadium)
that runs in an embedded Secure Element.

In my opinion, this VM project, lead by Ledger engineer, Salvatore Ingala,
will revolutionize the way of developing applications on hardware wallet devices.

### Generating a DID key

{{< sidenote >}}
[BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
is a way to segregate multiple account from one seed. By using a specific
derivation path for the atproto protocol, the application prevent any duplicate
usage of a secret with an other protocol like Bitcoin.
{{</ sidenote >}}

The device stores the seed of the private key in its secure element. We simply
need to ask it to generate a public key from this seed and display it in
the DID format. To generate the key, I chose to follow the BIP-32 standard and
settled on the m/🦋'/0 (m/129419'/0) derivation path.

By default, the last derivation index is unhardened and can be incremented if
the user wants to derive another key (such as m/🦋'/1) to add it to the PLC
record of another DID/account. This allows the user to maintain privacy and
avoid revealing that both accounts belong to them while using the same master
secret to generate DID keys.

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreifb4eebdkltppcbvfshvqfsgwux67quijdrmtncmvvv37auu3jicu"
    alt="The Ledger Stax displaying the derived DID key"
    kind="video" >}}

By verifying the public key on the screen, the user can audit that the did:key
used in a PLC operation comes from the device.

### Signing a genesis PLC operation

To create a new did:plc, we provide the genesis block of information to be
signed. The screen displays all the information of the PLC operation for the user
to securely verify.

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreigxryxd6cyeqairleluwaiai46dyti7afphkrom57vmwspj6vjw3q"
    alt="The Ledger Stax displaying the pages of the genesis PLC operation signing process"
    kind="video" >}}

### Signing a following PLC operation

When we want to modify the information again, we can sign a new op by providing the
new block and the previous block. Because the new block commits to the previous
one by including its CID, the device can check the CID and display only the
fields that have changed by comparing the two. Fewer fields to check !

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreifg3b5xleqgijlyghibtuiljhme2fyopgwwcfuask7igsmxtycnfu"
    alt="The Ledger Stax displaying the pages of the PLC operation signing process"
    kind="video" >}}

It is possible to verify all the operation I did on the ledger with its DID:
[did:plc:mxscbgwn6rdnmryqa32cdmka](https://web.plc.directory/did/did:plc:mxscbgwn6rdnmryqa32cdmka)

### Future improvements

It would be nice to have a web interface with WEBHID, so users
can directly connect the ledger device to a web app in a browser and sign.

I am not sure if adding a way to sign records on the ledger would be very useful,
as most users prefer to quickly share content from their phones,
and the PDS retains the record signing key. However, we can envision a future
where some atproto applications may require strong security for record signatures,
such as for records of binaries that people will run. For example, a federated
video game sharing platform might need this level of security.
