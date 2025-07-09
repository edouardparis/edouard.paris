---
title: "Hardware wallet for PLC rotation key"
slug: "hardware-wallet-for-plc-rotation-keys"
date: 2025-06-12
description: ""
draft: false
topics:  ["atproto", "bluesky"]
---

# Hardware wallet for PLC rotation key

[Atproto](https://atproto.com), the protocol behind Bluesky, allows for
the decentralization of many components of the platform.
Today, I want to discuss how it manages identity
with public key distribution and key rotation.

Currently, your Bluesky identity is attached to a global identifier (did:plc)
stored on one centralized server, the PLC directory. This identifier resolves to
a set of public keys, which we will discuss later, and a pointer to the Personal
Data Server (PDS) where your content is stored.

On the PDS, your content is cryptographically signed by an asymmetrical key,
often hosted on the same server. Its public key is stored in the PLC directory
under your global identifier. This way, the ecosystem has a link between your
identity and your content.

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
If you want to migrate to another server, you need to change the pointer to the
new PDS and update the signing key under your did:plc to the one hosted on the
new server.

To do this, you must provide a document describing your change to the
plc.directory, which will accept it if a signature from one of your public keys
is attached. We are talking about the rotation keys under your did:plc.

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



If you want to have control over your identity on the PLC directory, these
rotation keys are crucial. By default, Bluesky manages one for you on the
Bluesky PDS, but **if you want to take control, you need to generate a key that
only you possess and share the public key with the PLC directory, assigning it a
higher priority level than the others.** The highest priority key can override any
change signed by others.

In summary, you need to generate a key and keep it safe. If you are particularly
security conscious, you should only use it on a secure device that protects you
against key extraction and prevents blind signing with a screen. This is a
problem that other technologies have faced, and the Atproto documentation
provides hints on how to handle it.

## What is a hardware wallet ?

Hardware wallet are the name given in the cryptocurrency sector to hardware devices
holding secret keys. They generally contain a chip known as a secure element,
which handles cryptography for the user, eliminating the need to trust the laptop.

I took the opportunity to use one of these devices with the largest screen (the Ledger Stax)
to demonstrate an app I am developing to manage PLC operations.

## A Vanadium app for Ledger device that manages PLC operations

Ledger engineer, Salvatore Ingala is working on a fantastic project that I think
will revolutionize the way of developing application on Hardware wallet devices:
[Vanadium](https://github.com/LedgerHQ/vanadium), a Risc-V Virtual Machine
that runs in an embedded Secure Element.


### Generating a DID key

The device stores the seed of the private key in its secure element. We simply
need to ask it to generate a public key from this seed and display it to us in
the DID format. To generate the key, I chose the follow the BIP-32 standard.

I have chosen the m/🦋'/0 (m/129419'/0) derivation path.

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreifb4eebdkltppcbvfshvqfsgwux67quijdrmtncmvvv37auu3jicu"
    alt="Verify atproto did key"
    kind="video" >}}

### Signing a genesis PLC operation

To create a new did:plc, we provide the genesis block of information to be
signed. The screen displays all the information of the PLC operation for us to
securely verify.

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreigxryxd6cyeqairleluwaiai46dyti7afphkrom57vmwspj6vjw3q"
    alt="The Ledger Stax displaying the pages of the genesis PLC operation signing process"
    kind="video" >}}

### Signing a following PLC operation

When we want to modify the info again, we can sign a new op by providing the
new block and the previous block. Because the new block commits to the previous
one by including its CID, the device can check the CID and display only the
fields that have changed by comparing the two. Less fields to check !

{{<atproto-blob
    pds="https://pds.edouard.paris"
    did="did:plc:sl7e2yuycnqjk24jdjmeuidn"
    cid="bafkreifg3b5xleqgijlyghibtuiljhme2fyopgwwcfuask7igsmxtycnfu"
    alt="The Ledger Stax displaying the pages of the PLC operation signing process"
    kind="video" >}}

Did of the ledger: [did:plc:mxscbgwn6rdnmryqa32cdmka](https://web.plc.directory/did/did:plc:mxscbgwn6rdnmryqa32cdmka)
