+++
title = "Setup Matrix server and IRC bridge with NixOs"
slug = "setup-matrix-server-and-irc-bridge-with-nixos"
date = "2025-04-15"
+++

# Setup a Matrix server and an IRC bridge with NixOs

Something cool with software programming and open source:
there is always new projects to follow.
I understand it can be exhausting, especially if you are FOMO sensible,
but what a treat !

Problem is: to communicate with their community, projects have the
choice of too many chat platforms: the current main one is [Discord](https://discord.gg)
the second is [Matrix](https://matrix.org) which allows you to self host
a server and connect it to a federation. The ancestor of all is
[IRC](https://en.wikipedia.org/wiki/IRC) with some communities still discuss.

I discovered recently [tangled.sh](https://tangled.sh), a git collaboration platform,
built on the [AT Protocol](https://atproto.com/). As they wanted to discuss their development
and grow a community, they opened a room on the IRC [Libera.Chat](https://libera.chat/) server.

I also wanted to lurk into Nix and NixOs discussion channels and their community is using Matrix.

Thankfully to the nixpkgs contributors, it is quite easy now to self host a Matrix server and
an irc bridge. I discovered the following stack: [conduit](https://conduit.rs/) and
[Heisenbridge](https://github.com/hifi/heisenbridge), thanks to this
[article:](https://www.nat-418.xyz/posts/1ba7609d42e27999759eb4b9cd9810cb.html) from
[Nat-418](https://www.nat-418.xyz).

I modified the `configuration.nix` to add [delegation]() to the matrix server and share
Heisenbridge app service credentials through [agenix](https://github.com/ryantm/agenix).

Let's do it together:

## Setup the Matrix server

First Conduit will require a token to filter the registration of user,
first user connect is the admin.
So we generate a quick token with this command:
{{< code lang="shell" >}}openssl rand --hex 16{{< / code >}}

We add the `matrix-conduit` service to the nix configuration:

{{< code lang="nix" >}}
networking.firewall.allowedTCPPorts = [
    80
    443
    8448
];

services.matrix-conduit = {
    enable = true;
    settings.global = {
      address = "127.0.0.1";
      allow_registration = true;
      registration_token = "the token";
      database_backend = "rocksdb";
      port = 6167;
      server_name = "our.domain";
    };
};

services.caddy = {
    enable = true;
    configFile = pkgs.writeText "Caddyfile" ''
      matrix.our.domain, matrix.our.domain:8448 {
        reverse_proxy /_matrix/* 127.0.0.1:6167
      }

      our.domain {
        handle /.well-known/matrix/server {
          respond `{"m.server": "matrix.our.domain:443"}`
          header Content-Type application/json
        }

        handle /.well-known/matrix/client {
          respond `{"m.homeserver":{"base_url":"https://matrix.our.domain"}}`
          header Content-Type application/json
          header Access-Control-Allow-Origin *
        }

        # the rest of your Caddy file configuration for the domain
     }
    '';
  };
{{< / code >}}

We will change `allow_registration` to `false` and remove the `registration_token` line
after the first user (admin) is registered.

We can then test that federation is working by entering
the matrix server domain in https://federationtester.matrix.org/

# Add an IRC bridge

Better to have an irc bridge with you


{{< sidenote >}}

```sidenote-ascii-art
     ██████████████
    ██████████████████
    ██████████████████
    ██████████████████
    ██████████████████
██████████████████████████

  ██████████████████████
  █████████    █████████
    █████        █████

           ████
       ████████████
       ██        ██
       ██  ████  ██
         ████████

```
the [Heisenbridge](https://github.com/hifi/heisenbridge) logo

{{</ sidenote >}}

{{< code lang="nix" >}}age.secrets."heisenbridge-config" = {
    file = ./secrets/heisenbridge-config.age;
    owner = "heisenbridge";
    group = "heisenbridge";
};

services.heisenbridge = {
    enable = true;
    extraArgs = ["--config" config.age.secrets.heisenbridge-config.path];
    debug = false;
    homeserver = "https://matrix.our.domain";
    owner = "@ourusername:our.domain";
};
{{< / code >}}


