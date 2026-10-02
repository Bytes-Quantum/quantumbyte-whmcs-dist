# Publishing

Each AI Builder product's **Publish to** setting decides where its apps go
live. Your customer never chooses; they press **Publish** and see the live
address.

Until the place you chose is ready (no cluster connected, no database
connected, VPS still installing), Publish is refused with a message naming
you. No app ever runs on QuantumByte's own servers instead.

1. [Publish to your Kubernetes](#publish-to-your-kubernetes)
2. [Publish to your customers' VPS](#publish-to-your-customers-vps)
3. [Publish to cPanel](#publish-to-cpanel)

## Publish to your Kubernetes

For products set to **Publish to: Our Kubernetes**. Done once, in your
[partner portal](03-domains-and-partner-portal.md#your-partner-portal), by
a member of your team.

**Your cluster needs:**

- An API server reachable from the internet over HTTPS.
- An ingress controller, and a DNS record `*.apps.example.com` pointing at
  it.
- A default StorageClass. Each app gets its own database with two volumes.
- For TLS: a wildcard certificate Secret for `*.apps.example.com`, or a
  cert-manager ClusterIssuer. The ClusterIssuer is needed for customers'
  own domains.

**Steps**, on the card **Publishing to your Kubernetes**:

1. **Name**: e.g. `Production cluster`.
2. **Namespace for the apps**: defaults to `quantumbyte-apps`.
3. **Show setup commands** → copy them.
4. Run them with your own admin kubeconfig. They create the namespace and a
   service account `quantumbyte` that can only manage apps inside that
   namespace, plus one cluster-wide read: which IngressClasses (ingress
   controllers) you run. Then they make a token and print a kubeconfig.
5. Paste that kubeconfig into **Kubeconfig**. If its `server:` is not the
   address your API answers on from the internet, set `SERVER` in the
   commands and run them again.
6. **Domain your customers' apps go under**: `.apps.example.com`.
7. **Ingress class** (empty = cluster default).
8. **cert-manager ClusterIssuer**, and/or **Wildcard certificate secret for
   your domain** (used instead of the issuer for `*.apps.example.com`).
9. **Connect**. Expect: "Connected. Your customers' apps publish here."

Each app then lives at `<app>.apps.example.com`.

**Things to note**

- One cluster per partner. The domain must be unique across partners.
- The connection is re-checked every hour. A missing right shows on the
  card as "Missing rights in <namespace>: …", and Publish is refused until
  you run the setup commands again and press **Check again**.
- The IngressClass read lets a customer's own domain move to a new app
  address without a moment of errors on ingress-nginx. Without it, or on
  another controller, the domain is briefly unanswered during that move.
  A cluster connected without this read keeps working; run the setup
  commands again to add it.
- Each app's database runs in your cluster. Its image is pulled through
  QuantumByte, so QuantumByte has to be reachable whenever an app starts on
  a node that has not run it before.
- Refused kubeconfigs: ones that sign in through a program (cloud CLI
  `exec`/`auth-provider`), plain `http`, disabled certificate checks, and
  servers on private addresses.
- While customer apps run on the cluster, its API address, namespace and
  domain can't change, and **Forget** is refused.
- Until a cluster is connected, customers on these plans are told you
  haven't finished setting up publishing.

## Publish to your customers' VPS

Your own VPS module creates the machine as usual; the addon hands it over.

1. Configuration → System Settings → Addon Modules → **QuantumByte on VPS**
   → **Configure** → **VPS products**: the product IDs of your VPS plans,
   comma separated → Save.
2. Each VPS product must deliver:
   - Ubuntu, amd64, 1 GB memory or more
   - ports 80 and 443 free
   - a domain on the service (the order's domain), pointed at the VPS by
     your DNS
   - root login with the password WHMCS holds for the service
3. NAT VPS: add a product custom field named **SSH Port**.
4. The AI Builder product for these customers: **Publish to: The client's
   VPS**.

What happens after a VPS order is paid:

1. Your VPS module creates the VPS. The addon registers it on the
   customer's QuantumByte account.
2. On the next cron run, the addon connects over SSH as root, with the
   password WHMCS holds, and starts QuantumByte's installer. It installs up
   to 3 VPSes per cron run, one install at a time per VPS. A VPS still
   booting is tried again on the next run.
3. Once installed, the customer's next Publish goes there, at the VPS
   service's domain, with a Let's Encrypt certificate on the VPS.

Progress and install errors show on the addon's page: **Addons** →
**QuantumByte on VPS**.

**Things to note**

- The root password never leaves WHMCS.
- The customer needs an AI Builder service too. Without one, the VPS
  waits, is retried every 30 minutes, and the Activity Log says so.
- The first SSH host key a VPS shows is kept. A rebuilt VPS shows a new
  key and is refused (the password is not sent): press **Forget host key**
  on the addon's page, and the next cron run trusts the new key.
- One app per VPS. A second app is refused with "Your plan publishes one
  app. Ask <you> for another."
- QuantumByte's agent on the VPS runs as root and updates itself from
  QuantumByte. It only connects outwards, once a minute. The app, its
  database, its certificates and a nightly backup all stay on the VPS.

## Publish to cPanel

The app goes into the customer's own cPanel account on one of your cPanel
& WHM servers: its files in the account's home, run by cPanel's Application
Manager on the account's main domain, its data and uploaded files on a
MongoDB server you run.

1. Your cPanel server entry (Configuration → System Settings → Servers):
   - **Hostname** set to the name on the server's TLS certificate (the
     module connects by hostname and verifies TLS; it uses the IP only when
     Hostname is empty).
   - WHM credentials: a WHM API token in **Access Hash** (or the root
     password).
2. The AI Builder product saved with this module
   ([Create the product](01-install.md#create-the-product)). The module
   then hands a new cPanel account to QuantumByte as soon as WHMCS creates
   it.
3. Your MongoDB server connected on your partner portal
   ([below](#your-database-for-cpanel-apps)).
4. The AI Builder product: **Publish to: cPanel**.

The customer's cPanel account needs:

- **Node.js 20.19 or newer** in the Application Manager (EasyApache 4's
  `ea-nodejs` package).
- An **empty `public_html`** (cPanel's defaults are fine) and no other
  Application Manager app. The app takes every path of the main domain, so
  an account with a website is refused.

### Your database for cPanel apps

cPanel has no MongoDB, and an account can't run its own. So you run one
MongoDB server for all your cPanel customers' apps, and connect it once on
your partner portal's card **Your database for cPanel apps**. Until it's
connected and its check passes, customers on cPanel plans are told you
haven't finished setting up publishing.

What you run:

- **MongoDB 7 or newer**, with a publicly trusted TLS certificate on the
  address QuantumByte connects to. On the cPanel server itself is fine.
- **A user for QuantumByte** with the roles `userAdminAnyDatabase` and
  `readWriteAnyDatabase`. At each Publish QuantumByte makes one database
  (`qb-<app id>`) and one user (`app_<app id>`, read and write on that
  database only) and writes the app's name and branding. It reads no
  customer data.
- **A firewall** that lets in your cPanel servers and QuantumByte (the card
  lists QuantumByte's addresses).
- **Backups**, because QuantumByte does not back up this server.
- MongoDB is licensed under the SSPL. Check that the way you offer it to
  your customers fits that licence.

On the card:

1. **Connection string for that user**, e.g.
   `mongodb://quantumbyte:password@db.example.com:27017/?tls=true`.
2. **Address your cPanel servers reach it on (empty = the same address)**.
   With MongoDB on the cPanel server itself, enter `127.0.0.1:27017`.
3. **Connect**. Expect: "Connected. Your customers' cPanel apps keep their
   data here."

Apps connect over TLS too, unless the server is on the cPanel machine
itself. **Check again** re-runs the check; **Change** edits both fields
(leave the connection string empty to keep it); **Forget** removes it.

### The cPanel API token

To reach the account, the module makes a **full-access cPanel API token**
on it through WHM, with the credentials WHMCS holds for that server.

- Named `quantumbyte-<AI Builder service ID>`. The customer can see it
  under cPanel → Manage API Tokens.
- It lasts 40 days and is replaced a week before it expires, or sooner if
  cPanel refuses it. It's kept encrypted in WHMCS and scrubbed from the
  Module Log.
- Sent to QuantumByte when the AI Builder service is created, upgraded or
  renewed, on **Open AI Builder**, and right away when WHMCS creates a
  cPanel account for a customer whose AI Builder publishes to cPanel.
- Revoked when the cPanel service is terminated, which also tells
  QuantumByte to stop publishing there.
- Kept when the AI Builder service is terminated: QuantumByte uses it to
  take the app offline, and 30 days later to delete the app from the
  account, then revokes it. Terminate renews it first if it would expire
  before then.

**Things to note**

- The module reaches cPanel on your WHM port minus 4 (2087 → 2083).
- The account used is the customer's newest active cPanel service. A
  customer without one is refused Publish until it's set up.
- One app per cPanel account.
- A slow cPanel server never blocks sign-in (it waits 10 seconds at most).
