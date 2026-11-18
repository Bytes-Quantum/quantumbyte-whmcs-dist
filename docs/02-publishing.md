# Publishing

From 1.4, your customers' apps go live only on a server of their own, which
they add in QuantumByte under **Your servers**. The product's **Publish to**
setting is retired: whatever it holds, QuantumByte ignores it. A customer
may always add one server (see
[Customers' own servers](#customers-own-servers)), with or without a
domain.

Publish is refused until the customer has added a server and connected it:
"Connect your server first." No app ever runs on QuantumByte's own servers
instead. QuantumByte no longer publishes to your Kubernetes or to cPanel
(see [Upgrading to 1.5](10-upgrade-1.5.md)).

## Publish to your customers' VPS

Your own VPS module creates the machine as usual. The addon hands it to
QuantumByte, and the customer connects it themselves. Neither the addon nor
QuantumByte ever logs in to the VPS or uses its root password.

1. Configuration → System Settings → Addon Modules → **QuantumByte on VPS**
   → **Configure** → **VPS products**: the product IDs of your VPS plans,
   comma separated → Save.
2. Each VPS product must deliver:
   - Linux with systemd. Tested: Ubuntu 22.04 and 24.04, Debian 12,
     AlmaLinux 9, Rocky 9. Other Linux with systemd installs after a
     warning. The installer needs OpenSSL 3 to check QuantumByte's signatures, so
     Ubuntu 20.04, Debian 11 and AlmaLinux or Rocky 8 are refused. A 20.04
     VPS connected earlier takes one more update, then refuses all later
     ones.
   - amd64 (x86-64), with a CPU that has AVX (MongoDB 7 needs it). ARM
     servers are not supported.
   - at least 1.7 GB of memory in total (servers sold as 2 GB pass), with
     1.3 GB available when the install starts
   - 10 GB of free disk space where Docker keeps its data
   - `curl` and `sudo` installed (minimal Debian images may lack them)
   - HTTPS (port 443) out to the hosts in
     [Hosts the VPS must reach](#hosts-the-vps-must-reach)
   - Docker with its compose plugin, or no Docker at all (the installer
     adds Docker's own packages). Not Podman, not Docker from snap. These
     conflict with Docker's packages and must not be installed: on
     AlmaLinux and Rocky `podman`, `buildah` and `runc`; on Ubuntu and
     Debian `containerd`, `runc`, `docker.io`, `docker-compose` and
     `podman-docker`.
   - a web server on ports 80 and 443 is fine: the app goes behind it (see
     [Your web server forwards to the app](#your-web-server-forwards-to-the-app)).
     With nothing there, the customer may let QuantumByte serve HTTPS
     itself.
   - a free port in 3100–3199 for the app and in 8080–8179 for the agent's
     check, and Docker Compose 2.24.4 or newer. Behind the VPS's own web
     server, Docker 28 or newer too.
   - a domain on the service (the order's domain or hostname field). The
     customer's app is published at exactly that name. See
     [The VPS's domain](#the-vpss-domain).
   - a way for the customer to sign in to it and use `sudo` (the root
     password you hand them, or an SSH key). They may change it whenever
     they like.
3. The AI Builder product for these customers: **Publish to: The client's
   VPS**.

What happens after a VPS order is paid:

1. Your VPS module creates the VPS. The addon registers it on the
   customer's QuantumByte account. You point its domain at its address
   with your own DNS tools; the addon shows whether it does.
2. The customer opens the VPS's page in your client area and presses
   **Connect this server in QuantumByte**. It signs them in to QuantumByte
   on that server's **Connect your server** page.
3. They press **Get the install command**. QuantumByte asks for their
   two-step sign-in code; a customer who has not set two-step sign-in up is
   taken through setting it up first.
4. The page shows one install command. The customer signs in to the VPS,
   pastes the command and runs it. They can use it once, within 1 hour. The
   page changes to **Connected** a few seconds after the install finishes.
5. The customer's next Publish goes there, at the VPS service's domain.
   HTTPS comes from the VPS's own web server, or from QuantumByte's front
   door (below). Until the VPS is connected, Publish says "Connect your
   server first." and links to that page.

### Your web server forwards to the app

The app answers only on the VPS itself, at `http://127.0.0.1:3100`
(or the next free port). A small QuantumByte proxy sits there, so a new
version swaps in without visitors noticing. What serves visitors depends on
what the install finds on ports 80 and 443:

- **A web server (nginx, Apache, Caddy):** it keeps serving them. The
  installer prints the lines to add to it, filled in with the domain and the
  port, plus the certificate command (`certbot --nginx -d <domain>` or
  `certbot --apache -d <domain>`; Caddy gets its certificate itself). They are also saved
  in `/etc/quantumbyte/web-server-lines.txt`. The lines:
  - pass every path and every response header back;
  - allow uploads of 20 MB (the app takes pictures up to 15 MB);
  - keep WebSockets working.

  For another web server, the installer says what to forward where.
- **Nothing:** the installer asks whether to turn on QuantumByte's HTTPS
  front door on ports 80 and 443. Yes: QuantumByte serves HTTPS and gets a
  Let's Encrypt certificate once the domain points at the VPS. No: it
  prints the lines for a web server added later.
- **A web server in a Docker container:** it can't reach the VPS's
  `127.0.0.1`. The installer says to run it with host networking.

With no terminal (for example `curl … | sudo bash` run from a script), the
installer can't ask. Add `--front-door` or `--no-front-door` before the `--`
and the install token; `--local-port <port>`, in the same place, picks
another port.

When the customer adds a domain to their app, their web server needs it
too: the next update rewrites the saved file, and
`sudo cat /etc/quantumbyte/web-server-lines.txt` shows the new lines. A
domain they remove keeps working until they take it out of their web server.
A server managed by a control panel (cPanel, Plesk) may overwrite
hand-edited files: use the panel's reverse-proxy setting with the same
values.

A VPS connected before local ports (QuantumByte release of November 2026)
keeps serving 80 and 443 itself, as before.

The addon's page (**Addons** → **QuantumByte on VPS**) shows each VPS
as QuantumByte sees it: connected or waiting for the customer, when it was
last seen, and its last release. Use it to answer customers. A ready reply
for "How do I connect my server?" is in
[Upgrading to 1.1](05-upgrade-1.1.md#a-reply-for-your-support-team).

**Things to note**

- Nobody but the customer uses the VPS's root password. The customer can
  reset it whenever they like; a connected VPS keeps working.
- The customer needs an AI Builder service too. Without one, the VPS
  waits, is retried every 30 minutes, and the Activity Log says so. The
  **Connect** link shows once they have one.
- A rebuilt or reinstalled VPS: the customer presses **Reconnect this
  server** on the same page and types the VPS's domain to confirm. The old
  connection stops at once, and they run the new command on the rebuilt
  VPS.
- Two-step sign-in: a customer who loses their phone and their recovery
  codes can't get a new command until QuantumByte support resets their
  two-step sign-in. Ask them to keep the recovery codes.
- One app per VPS. A second app is refused with "Each VPS runs one app.
  Order another VPS from <you> for another app." A customer may have any
  number of VPSes.
- QuantumByte's agent on the VPS runs as root and updates itself from
  QuantumByte. It only connects outwards. Only the nightly backup runs on
  its own. After each Publish, QuantumByte shows the customer
  `sudo quantumbyte update`, and the customer runs it on the VPS. It
  installs the new version without downtime, applies domain changes, says
  how it went and stops. QuantumByte then opens the site from outside to
  check the new version is live. A VPS connected before updates by command
  began keeps asking QuantumByte once a minute until its customer installs it
  again ([Server security](06-server-security.md#what-runs-as-root-and-when)). The app, its
  database, its certificates and a nightly backup all stay on the VPS.
  What it can and can't do: [Server security](06-server-security.md).
- Every agent file QuantumByte installs on the VPS is signed. App images
  and release settings are not; see [Server security](06-server-security.md).
  The install command checks the signature before `sudo` runs anything,
  and the agent installs no update that fails the check.
- The signing key's fingerprint (sha256 of the public key) is
  `9aea351512a1d25179db5c239433f76bd2a34fb6d21ae0cb7d4dc490103c87f0`. The
  command prints that public key: to compare, save it as `qb.pub` and run
  `openssl pkey -pubin -in qb.pub -outform DER | sha256sum`. The spare key
  that replaces it one day has the fingerprint
  `e36518b1fc4ceb26f832210db73ca7c8b079baeec0603c52d2b24a7684efdf08`.

### Preparing your VM template

Check each VM template before you sell it. On a fresh VM made from it,
run this. It needs no install token, changes nothing, and checks the
installer's signature first, like the customer's command:

```sh
cd "$(mktemp -d)" && api=https://client-api.quantumbyte.ai/api/v1/deploy-agent \
&& curl -fsSL -O $api/install.sh -O $api/files/MANIFEST -O $api/files/MANIFEST.sig \
&& printf '%s\n' '-----BEGIN PUBLIC KEY-----' 'MCowBQYDK2VwAyEABd3uerXjZPqib/VSZIwFSAWwf8v5z63lQtlzXDF7GBQ=' '-----END PUBLIC KEY-----' >qb.pub \
&& openssl pkeyutl -verify -pubin -inkey qb.pub -rawin -in MANIFEST -sigfile MANIFEST.sig \
&& grep -qxF "$(sha256sum install.sh)" MANIFEST \
&& sudo bash install.sh --check-only --api $api --registry registry.quantumbyte.ai
```

It prints one line per check: `PASS`, `WARN` or `FAIL`, in English. It
ends with "This server is ready for QuantumByte." or "Fix the FAIL lines
and run the same command again." Fix the template and run it again until
nothing FAILs. A WARN does not stop an install; read it anyway. The
customer's install command runs the same checks before it changes
anything.

### Hosts the VPS must reach

The VPS only connects outwards, over HTTPS (port 443). Allow these in your
firewall:

| Host | For |
| --- | --- |
| `client-api.quantumbyte.ai` | the installer, the agent and its updates |
| `registry.quantumbyte.ai` | the customer's app image |
| `registry-1.docker.io`, `auth.docker.io`, `production.cloudflare.docker.com` | Docker Hub: the database and web proxy images |
| `download.docker.com` | Docker's packages, only when Docker is not installed yet |
| `acme-v02.api.letsencrypt.org` | the app's HTTPS certificate, only with QuantumByte's HTTPS front door on |

Also allow your Linux distribution's own package mirrors, for `jq` and
Docker's dependencies.

### The VPS's domain

The app opens at the VPS service's domain once that name points at the
VPS's address (the service's dedicated IP). **You set that record** with
your own DNS tools: an A record, or AAAA for an IPv6 address. Neither
QuantumByte nor the module writes it, and neither removes it.

- **The addon checks it** on every cron run until the name points at the
  VPS, then every 30 minutes. A page view never triggers a check. The
  addon's page shows **Domain points here**: "Yes", "No record", or
  "Points elsewhere" with the address found, and lists the VPSes that
  need a record above the table. It asks your WHMCS server's resolver, so
  it can see a change a little before or after QuantumByte does. When the
  resolver doesn't answer, the last result stays.
- **The customer's VPS page** shows the record while it doesn't point
  there yet, with "<your company> sets this record for your server. Ask
  <your company> if it is not there yet." (your company name in WHMCS's
  General Settings). In English or Indonesian, as the customer reads your
  client area, with a link to open a support ticket.
- **A renamed VPS** (a new domain or address on the service) is sent to
  QuantumByte on the next cron run, connected or not. Point the new name
  yourself; the customer's next `sudo quantumbyte update` picks it up.
- **When you end the VPS**, its record stays in your zone. Remove it
  yourself before the address goes to someone else. The Activity Log
  reminds you.

The order's domain field is the app's address, so a hostname like
`server1.example.id` publishes the app there, not on `example.id`. To
publish on the bought domain itself, enter that domain in the field.

### Customers' own servers

Customers publish to servers they rent anywhere. Set it per AI Builder
product, in **Module Settings** → **Own servers allowed**: how many such
servers the customer may add, from 0 to 100. A customer may always add one,
so `0`, the default, means one.

- **Where the customer adds them:** in QuantumByte, under **Your servers**.
  They add a server, then run the install command QuantumByte shows them
  on it, as for a VPS you sell. They choose which app goes on which of
  their servers.
- **How many:** the settings of all the customer's active AI Builder
  services, added together, and at least one. Two services set to `1`
  allow two servers. A suspended service allows no new ones.
- **The domain is optional.** A customer may add a server with no domain
  and publish to it. The app runs there with no public address, and the
  customer attaches a domain later.
- **Lowering it** never removes a server or stops an app. The customer
  just can't add another until they are under the new number.
- **When a change applies:** The new number reaches QuantumByte at the
  service's next create, upgrade or downgrade, renewal, suspend or
  unsuspend, or when the customer next opens AI Builder. Until then
  QuantumByte keeps the number it has.
- **DNS:** the customer sets the record for their own server's name.
  Neither you nor QuantumByte writes it. The name is not reserved: you
  can still give it to a VPS you sell later.
- **Your control:** neither you nor QuantumByte manages these servers.
  Suspending the AI Builder service pauses publishing, as for
  a VPS you sell; the app already running stays up. The customer removes a
  server themselves in QuantumByte, which frees its place.
- **The addon's page** lists them under **Customers' own servers**: the
  client, how many they have, the domain, whether it is connected, and when
  QuantumByte last heard from it. Nothing else about those servers reaches
  you. You see this list with the addon active, even without VPS products.

A number above 100 counts as 100. Anything that isn't a whole number counts
as empty: nothing is sent.
