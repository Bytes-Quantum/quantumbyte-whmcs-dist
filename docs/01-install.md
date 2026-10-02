# Install

1. [Before you start](#before-you-start)
2. [Install the module and add the server](#install-the-module-and-add-the-server)
3. [Create the product](#create-the-product)
4. [Test an order end to end](#test-an-order-end-to-end)

## Before you start

**Your WHMCS needs:**

1. WHMCS 8 or later. Proven on WHMCS 9.0.8 with PHP 8.3.
2. PHP with curl. The VPS part of the addon also uses the phpseclib that
   WHMCS bundles.
3. **System URL** (Configuration → System Settings → General Settings)
   set to your real public address, starting with `https://`.
4. `https://whmcs.example.com/oauth/token.php` reachable from the internet.
5. The WHMCS cron running. WHMCS recommends every 5 minutes. VPS installs
   and domain upkeep run from it.
6. Outbound HTTPS (443) from WHMCS to `<QB hostname>`.
7. At least one payment gateway active, so customers can order.

The installer's `--check` tests all of these except point 4 (see
[below](#install-with-the-installer-script)).

**QuantumByte gives you** (there is no sign-up form; QuantumByte sets up
your partner account with you):

- A **hostname** and an **API key** for WHMCS. The key is shown once. If
  it's lost, QuantumByte issues a new one and the old one stops working at
  once.
- **Terms**: your currency, and whether you pay ahead (**Prepaid**: top up
  first) or monthly (**Postpaid**, up to a credit limit).
- A sign-in for your team's **partner portal** (email and first password).
- The **installer** download link and its **sha256** checksum. Your
  partner portal shows the checksum too.

**Things to note**

- **Test Connection** fails while the System URL is not `https://`. A
  service created anyway fails with "app_link.token_url must be
  https://<your WHMCS>/oauth/token.php."
- If `oauth/token.php` can't be reached, sign-in still works. The "back to
  your WHMCS" links in the AI Builder then open your login page instead of
  signing the customer in.

## Install the module and add the server

### Install with the installer script

`install-quantumbyte-whmcs.sh` checks your WHMCS, then copies both folders
into it. It never writes to the WHMCS database and holds no secrets.

1. Find the user that owns your WHMCS files, and run everything below as
   that user (root only if root owns them):

   ```sh
   stat -c %U /var/www/whmcs/modules/servers
   su -s /bin/bash <owner>
   ```

2. Download the script and check it against the sha256 QuantumByte gave
   you (also on your partner portal). Stop if this does not print `OK`:

   ```sh
   echo "<sha256>  install-quantumbyte-whmcs.sh" | sha256sum -c -
   ```

3. Check first. This changes nothing:

   ```sh
   bash install-quantumbyte-whmcs.sh --check --whmcs-root /var/www/whmcs --qb-host <QB hostname>
   ```

   Each line reads PASS, WARN or FAIL. Fix every FAIL. A WARN doesn't stop
   the install, but most need fixing before you sell. What each line means:
   [Installer messages](04-operations.md#installer-messages).

4. Install:

   ```sh
   bash install-quantumbyte-whmcs.sh --whmcs-root /var/www/whmcs --qb-host <QB hostname>
   ```

   The script runs the same checks, asks before it changes anything, then:
   downloads the module and refuses it if its sha256 doesn't match the one
   built into the script, checks every PHP file parses, backs up the
   folders already there to `~/quantumbyte-whmcs-backups/` or the
   `--backup-dir` you pass (the last 5 are kept), puts the new folders in place, and checks WHMCS still loads. If
   it doesn't, the previous module goes back in place. At the end it
   prints the line that undoes the install, and the next steps.

5. If PHP runs under PHP-FPM with opcache set not to check for changed
   files (`opcache.validate_timestamps=0`), reload PHP-FPM so WHMCS loads
   the new code. The script tells you when it sees that setting.

| Flag | What it does |
| --- | --- |
| `--whmcs-root DIR` | Required. Your WHMCS root folder, the one holding `init.php`. |
| `--check` | Check only; change nothing. |
| `--qb-host HOST` | `<QB hostname>`, to check WHMCS can reach it over HTTPS. Without it that check is skipped with a WARN. |
| `--php PATH` | The PHP binary your WHMCS cron runs with, when `php` on the PATH is another one, e.g. `/opt/cpanel/ea-php83/root/usr/bin/php`. |
| `--bundle FILE` | Install from the module archive (`quantumbyte-whmcs-<version>.tar.gz`) downloaded elsewhere, for servers that can't reach GitHub. It must still match the sha256 built into the script. |
| `--backup-dir DIR` | Where to back up the module folders before replacing them. Default `~/quantumbyte-whmcs-backups`. Use it when the owner's home isn't writable, as with a stock `www-data` (home `/var/www`, owned by root). It must be outside the WHMCS folder and writable by the owner; it's created (mode 700) if missing and its parent is writable. |
| `--yes` | Don't ask before installing. Needed when there is no terminal to ask on. |
| `--no-color` | Plain output. |
| `--help` | Show the flags. |

The script exits 0 when nothing failed, 1 on a FAIL, 2 on a wrong flag.
Running it again on the same version says it's already installed and
changes nothing.

### Install by hand

If you can't run the script, copy two folders from the module archive into
your WHMCS root, as the owner of the WHMCS files:

- `modules/servers/quantumbyte/` (with its `lib/` and `templates/`)
- `modules/addons/quantumbyte_vps/`

Then go through [Before you start](#before-you-start) yourself.

### Activate the addon and add the server

1. Configuration → System Settings → **Addon Modules** → **QuantumByte on
   VPS** → **Activate**. Under **Configure**, give your admin role access.
   Activate it even if you don't sell VPS plans: it also hands new cPanel
   accounts over, connects bought domains, and runs the DNS upkeep on
   every cron run.
2. Configuration → System Settings → **Servers** → **Add New Server**:
   - Module: **QuantumByte**
   - Hostname: `<QB hostname>`
   - Password: `<API key>`
3. Press **Test Connection**. Expect "Connection successful".
4. Name it (e.g. `QuantumByte`), **Save Changes**.
5. Put the server in its own server group (**Create New Group**).

**Things to note**

- The module always uses HTTPS, whatever the server's "Secure" box says.
- The API key is stored encrypted by WHMCS and scrubbed from the Module
  Log.
- The addon calls QuantumByte through the server entry of the customer's
  AI Builder service, or the lowest-ID enabled QuantumByte server when the
  customer has none. One QuantumByte server per WHMCS is the simple setup.

## Create the product

1. Configuration → System Settings → **Products/Services** → **Create a New
   Product**: type **Other**, your product group, a name of your choice
   (e.g. `AI Builder 500`), module **QuantumByte** → **Continue**.
2. **Module Settings** tab:
   - Module Name: **QuantumByte**
   - Server Group: the group you made for the server
   - **Credits**: how many credits the package includes. They load into
     the customer's account on creation and on each paid renewal.
   - **Publish to**: where this plan's apps go live when the customer
     presses Publish:
     - **Our Kubernetes**: your cluster ([Publish to your Kubernetes](02-publishing.md#publish-to-your-kubernetes))
     - **The client's VPS**: a VPS you sell them ([Publish to your customers' VPS](02-publishing.md#publish-to-your-customers-vps))
     - **cPanel**: their cPanel account at you ([Publish to cPanel](02-publishing.md#publish-to-cpanel))
   - Choose **Automatically setup the product as soon as the first payment
     is received**.
3. **Save Changes**. Set **Pricing** as for any product.
4. For more than one package size: create one product per size, then on
   each product's **Upgrades** tab list the others.
5. Configuration → System Settings → General Settings → **Credit** tab:
   turn **Credit on Downgrade** off.

**Things to note**

- The product decides where the app goes, and your customer never
  chooses. They see Publish and their address.
- Changing **Publish to** later reaches QuantumByte on the customer's next
  sign-in, upgrade or renewal, and applies from their next Publish. An app
  moved that way starts at the new place with an empty database.
- If you already sell hosting, put your hosting plan and the AI Builder
  product in a WHMCS **Product Bundle** (Configuration → System Settings →
  Product Bundles). One order, two services, each set up, billed and renewed on its
  own. With the AI Builder item set to **Publish to: cPanel**, the app goes
  into the cPanel account from the same bundle. Selling QuantumByte as a
  product *addon* is not supported.
- Credits roll over; unused credits never expire.

## Test an order end to end

1. In your store, as a test customer, order the AI Builder product and
   check out (checkout creates the client, order and invoice).
2. Admin area → the invoice → **Add Payment**. Expect "Module Create
   Successful" on the service, status **Active**.
3. Client area → the service page → **Open AI Builder**. The customer lands
   in the AI Builder, signed in, with the package's credits.
4. Build a small app, press **Publish**. The dialog shows the address, with
   no choice of where.
5. Partner portal → **Customer apps**: the app, where it went, **Running**.
6. Open the address.
7. Back on the service page: **Connect a domain**, then check the name
   turns **Live**.

**Things to note**

- Record payment with **Add Payment**. Setting an invoice's status to Paid
  by hand runs no module call, so no account and no credits.
- A failed create lands in Utilities → **Module Queue**. Retrying is safe:
  QuantumByte returns what the first call made.
- A customer whose email already has a QuantumByte account is refused:
  "… already has a QuantumByte account. Ask the customer to use another
  email address for this service."
- Your staff can press **Open AI Builder** on the service in the admin area
  and act as that customer. Give that permission only to staff who may.
- In the AI Builder, the account menu shows your name with **View
  invoices**, **Your plan** (the service's page at you: upgrade, renew,
  cancel) and **Contact support**. Each signs the customer into your client
  area through a single sign-on credential named "QuantumByte" under the
  service's OAuth credentials. The module makes it on Create and on Open AI
  Builder, and deletes it on Terminate.
