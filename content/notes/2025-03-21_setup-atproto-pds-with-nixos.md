+++
title = "Set up ATProto pds with NixOs"
date = "2025-03-21"
draft = false
slug = "setup-atproto-pds-with-nixos"
topics = ["caddy", "atproto",  "bluesky", "nixos"]
+++

> Self-hosting a Bluesky PDS means running your own Personal Data Server that is capable of federating with the wider ATProto network. - https://atproto.com/guides/self-hosting

---


# Set up your personal ATProto PDS with NixOs

This guide walks you through the process to host your own PDS instance
using [NixOs](https://nixos.org), [agenix](https://github.com/ryantm/agenix) for secure environment
management, and [Caddy](https://caddyserver.com/) as a reverse proxy for traffic encryption.

Thanks to the Nixpkgs contributors for making this possible!

{{< sidenote >}}
I wrote a [quick guide](/notes/install-nixos-on-an-ovh-vps-with-nixos-anywhere) on how to install NixOs using `nixos-anywhere` on an Ovh vps.
{{</ sidenote >}}

You need to have a server running [NixOs](https://nixos.org).
Using [nixos-anywhere](https://github.com/nix-community/nixos-anywhere) makes it
easy to install on a vps. The server configuration should be in a NixOs flake.



Retrieve the SSH key from the host:

{{< code lang="shell" >}}ssh-keyscan your.pds.domain{{< / code >}}

Add the ssh-rsa line to your flake's `secrets` folder in the file `secrets/secrets.nix`:

{{< code lang="nix" >}}
let
  vps_ssh = "ssh-rsa ...";
in
{
  "pds-env.age".publicKeys = [ vps_ssh ];
}
{{< / code >}}

Generate PDS secrets environment variables:

{{< code lang="shell" >}}{
  # Generate JWT secret
  JWT_SECRET=$(openssl rand --hex 16)
  echo "PDS_JWT_SECRET=$JWT_SECRET"

  # Generate admin password
  ADMIN_PASSWORD=$(openssl rand --hex 16)
  echo "PDS_ADMIN_PASSWORD=$ADMIN_PASSWORD"

  # Generate PLC rotation key
  PLC_KEY=$(openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32)
  echo "PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=$PLC_KEY"
} > /tmp/pds-secrets.env
{{< / code >}}

use agenix to encrypt the `pds-env` file according to the `secrets.nix` config:

{{< code lang="shell" >}}cd secrets/
agenix -e pds-env.age < /tmp/pds-env.env
{{< / code >}}

Then you can delete `/tmp/pds-env.env` after keeping its secrets safe in a password manager.

Modify the `configuration.nix` of the flake to enable the PDS service and use Caddy as a proxy to handle encryption:

{{< code lang="nix" >}}{
  config,
  pkgs,
  ...
}: {

  networking.firewall.allowedTCPPorts = [
    80
    443
  ];

  age.identityPaths = [ "/etc/ssh/ssh_host_rsa_key" ];

  age.secrets.pds-env = {
    file = ./secrets/pds-env.age;
    owner = "pds";
    group = "pds";
  };

  services.pds = {
    enable = true;
    settings = {
      PDS_HOSTNAME = "your.pds.domain";
      PDS_PORT = 3000;
      PDS_HOST = "127.0.0.1";
    };
    environmentFiles = [
      config.age.secrets.pds-env.path
    ];
  };


  services.caddy = {
    enable = true;
    configFile = pkgs.writeText "Caddyfile" ''
      your.pds.domain {
        reverse_proxy http://127.0.0.1:3000
      }
    '';
  };
}
{{< / code >}}

Now, it is possible to test the flake for the ovh configuration (if you followed the note):

{{< code lang="shell" >}}nixos-rebuild test --flake .#ovh-vps{{< / code >}}


Deploy the new configuration:


{{< code lang="shell" >}}nixos-rebuild switch --flake .#ovh-vps --target-host "root@your.pds.domain"{{< / code >}}


The new personal PDS should be up and running at the chosen hostname:

{{< code lang="txt" >}}curl https://your.pds.domain
         __                         __
        /\ \__                     /\ \__
    __  \ \ ,_\  _____   _ __   ___\ \ ,_\   ___
  /'__'\ \ \ \/ /\ '__'\/\''__\/ __'\ \ \/  / __'\
 /\ \L\.\_\ \ \_\ \ \L\ \ \ \//\ \L\ \ \ \_/\ \L\ \
 \ \__/.\_\\ \__\\ \ ,__/\ \_\\ \____/\ \__\ \____/
  \/__/\/_/ \/__/ \ \ \/  \/_/ \/___/  \/__/\/___/
                   \ \_\
                    \/_/


This is an AT Protocol Personal Data Server (aka, an atproto PDS)

Most API routes are under /xrpc/

      Code: https://github.com/bluesky-social/atproto
 Self-Host: https://github.com/bluesky-social/pds
  Protocol: https://atproto.com
{{< / code >}}

To migrate an account to the new PDS, follow this guide: [Migrating PDS account with goat](https://whtwnd.com/bnewbold.net/entries/Migrating%20PDS%20Account%20with%20%60goat%60).
