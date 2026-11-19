# Operations

1. [Suspend, terminate, upgrade, update the module](#suspend-terminate-upgrade-update-the-module)
2. [Limits to know](#limits-to-know)
3. [Troubleshooting](#troubleshooting)
4. [What runs without QuantumByte](#what-runs-without-quantumbyte)

## Suspend, terminate, upgrade, update the module

Run these from the AI Builder service's **Module Commands** as usual.

| WHMCS action | What happens |
| --- | --- |
| **Suspend** | The plan stops publishing. What else stops depends on the customer's plans: see [Suspend and terminate](#suspend-and-terminate). Data is kept. |
| **Unsuspend** | Publishing and sign-in come back as they were. |
| **Terminate** | Once *every* AI Builder service of the customer is terminated, the account is closed and its data deleted 30 days later. An app already running on the customer's server is never stopped or deleted: see [Suspend and terminate](#suspend-and-terminate). |
| **Create** on a terminated service | Within those 30 days: restores the account as it was. No credits granted or charged again. After that, order a new service. |
| **Renewal paid** | The product's Credits load again. A renewal that failed (e.g. your balance ran out) is caught up by the next successful one, or by retrying it in the Module Queue. |
| **Upgrade** | Once its invoice is paid, the extra credits load for what's left of the billing period, as WHMCS prorates it. Each paid upgrade invoice loads once, even after a downgrade in the same period. |
| **Downgrade** | Nothing taken back; the smaller package applies from the next renewal. |
| **Change Package** with no paid upgrade invoice | The package changes; no credits load. To give a customer a bigger package for free, raise an upgrade invoice and mark it paid: the paid invoice is what loads the credits. |
| **Credit pack paid** | Its credits load into the customer's account once, however often Create runs. No app, no plan. Suspend, Unsuspend and Terminate on it change nothing; Renew and Change Package are refused. |

### Suspend and terminate

Every plan is treated the same, whatever its retired **Publish to** setting
holds. Servers are not part of it: QuantumByte keeps no list of the
customer's servers, and the module no longer registers VPSes you sell.

**A suspended plan.** Publishing from that plan is refused with
"Publishing is paused. Ask `<you>` about your plan." This holds even while
another plan of the customer is active. The app on the customer's server
keeps running: neither QuantumByte nor you can stop it.

**Suspended, no plan active.** The customer keeps signing in and building.
WHMCS hides its own sign-in on a suspended service, so their service page
shows an **Open AI Builder** button instead. AI work still stops once their
credits run out.

**Several plans.** While any plan is active, the customer keeps signing in;
each suspended plan only stops publishing, as above.

**Terminated.** Once every plan is terminated, the account is closed and
sign-in refused. From that moment the customer's download key stops
working: no new image or content downloads, so no new release can be
installed. The app already running on their server keeps running, with its
data and backups. 30 days later QuantumByte deletes its own data for the
account. If you restore the service with **Create**, publishing and
downloads work again.

### Updating the module

Download the newer installer, check its sha256, and run it as in
[Install with the installer script](01-install.md#install-with-the-installer-script).
It backs up the folders it replaces. Settings live in WHMCS's database, so
nothing needs re-entering.

Coming from a version before 1.6: deactivate the QuantumByte on VPS addon
before you update, and delete its folder under `modules/addons` after. The
addon is retired, and its old code errors on the WHMCS cron once the module
is new. See [Upgrading to 1.6](11-upgrade-1.6.md).

To go back to a backup, run this as the owner of the WHMCS files, with a
backup from `~/quantumbyte-whmcs-backups/` (named by date and time). The
installer prints this line, filled in, at the end of each install:

```sh
cd /var/www/whmcs && rm -rf modules/servers/quantumbyte && tar -xzf ~/quantumbyte-whmcs-backups/<backup>.tar.gz
```

Reload PHP-FPM afterwards if opcache doesn't check for changed files.

**WHMCS on several web nodes:** run the installer on each node, or once on
the shared storage if the nodes share the WHMCS folder.

**Things to note**

- **Terminate before you delete.** Deleting a service or a client in WHMCS
  doesn't reach the module, so the QuantumByte account would stay open.
- WHMCS cancellations and "Termination Days" reach Terminate through the
  daily cron.
- Mark a renewal invoice paid with **Add Payment**. Setting its status to
  Paid by hand runs no module call; its credits load with the next renewal.
- Moving an existing service onto an AI Builder product loads credits once,
  on Create. Its earlier invoices load nothing.

## Limits to know

- One QuantumByte account per WHMCS client: a second service loads its
  credits into the same account. To sell credits without a second plan,
  use a [credit pack](01-install.md#sell-extra-credits-credit-packs).
- A customer email that already has a QuantumByte account is refused.
- Customers sign in only through you. QuantumByte's own password, Google
  and email sign-in refuse them.
- Apps run on servers the customer chooses. QuantumByte keeps no list of
  them and checks nothing there, so it cannot see whether an app runs, and
  backups stay on the customer's server.
- A customer with less than 1 credit left can't start AI work (chat,
  retries, build approvals, blueprint edits) until credits are added.
  Publishing uses no credits.

## Troubleshooting

**Where to look**

- **Utilities → Module Queue**: failed creates, renewals, upgrades, with
  QuantumByte's message. Fix the cause, then **Retry** (always safe).
- **Configuration → System Logs → Module Log**: every call to QuantumByte,
  with the API key and sign-in links scrubbed. Turn module logging on while
  you debug.
- **Activity Log**: AI Builder services that are Active with no QuantumByte
  account (one line each). A check that keeps failing (a refused key,
  QuantumByte unreachable) is logged once per server entry, and once more
  when it works again.
- **Admin dashboard**: names each QuantumByte server entry whose checks keep
  failing, until they work again, and lists AI Builder services that are
  Active with no QuantumByte account.
- Partner portal: balance, statements and **Customer apps**.

**Common messages**

| Message | Fix |
| --- | --- |
| Test Connection "Set Configuration → System Settings → General Settings → System URL to https://…" | Set System URL to `https://…` ([Before you start](01-install.md#before-you-start)). |
| Test Connection "QuantumByte refused this API key…" | Wrong or stopped API key. Check **Setup** → **API keys** in your partner portal. Don't save the entry with it: a saved entry is used. |
| "server entry #N …: QuantumByte refused this entry's API key" | Fix the key on that entry, or switch it off (Disable). |
| "QuantumByte has paused this partnership. Contact QuantumByte." | Contact QuantumByte. |
| "Not enough QuantumByte balance…" / "…credit…" | Top up or pay what you owe, then Retry. |
| "Your QuantumByte statement for … was due on …" | Pay the statement in the partner portal, then Retry. |
| "QuantumByte has not set your prices yet" | Ask QuantumByte to set your terms. |
| "app_link.token_url must be https://<your WHMCS>/oauth/token.php" | Set System URL to `https://…` ([Before you start](01-install.md#before-you-start)). |
| "… already has a QuantumByte account. Ask the customer to use another email address for this service." | The customer uses another email for this service. |
| "AI Builder service #N is Active, but has no live QuantumByte account" (Activity Log, service page, dashboard) | The order was added as Active, or the service was set Active by hand. Press **Create** under Module Commands. |

### Installer messages

`install-quantumbyte-whmcs.sh` prints one line per check: PASS, WARN or
FAIL, with the reason. A FAIL stops the install, and every install
failure ends with "Nothing was changed." or "The previous module is back
in place."

| Check | WARN or FAIL lines | Fix |
| --- | --- | --- |
| WHMCS folder | The folder doesn't exist or isn't WHMCS; not Linux (FAIL). | Pass the folder holding `init.php` to `--whmcs-root`. |
| Tools | "Missing tools: …" (FAIL). | Install them. Without `curl` or `wget`, use `--bundle`. |
| Write access | "Running as …, but …/modules/servers belongs to …" with the `su -s /bin/bash <owner> -c '…'` line to run instead; "…/modules/servers belongs to uid …, which has no user on this server" with the `chown` lines to run as root; files owned by someone else, a symlinked module folder, or a leftover from a stopped run (FAIL). The backup folder: "Backups go in …, but … can't write there" or "can't create it", "The backup folder … isn't a folder" or "is inside the WHMCS folder", or "HOME isn't set" (FAIL); each ends with the `mkdir`/`chown` line to run as root and the `--backup-dir` to add. | Do what the line says: each one names the command or folder. |
| PHP loads WHMCS | "No PHP found", or the PHP couldn't load WHMCS, often a missing ionCube Loader (FAIL). | Pass the PHP your WHMCS cron uses with `--php`. |
| WHMCS version | "… is too old" below 8.0 (FAIL); another version than the tested 9.0.8 (WARN). | Upgrade WHMCS, or go ahead on the WARN. |
| PHP version | "… is too old" below 7.4 (FAIL); "… but your WHMCS cron last ran on PHP …" (WARN); 7.4 to 8.2, untested (WARN). | Pass the cron's PHP with `--php`; PHP 8.3 is tested. |
| curl | PHP's curl isn't loaded, or has no TLS support (FAIL). | Enable PHP's curl extension with SSL. |
| System URL | Not set, not `https`, or an IP, `localhost` or a port (WARN). | Set it to your public `https://` address before selling: Test Connection fails without it. |
| Cron | Never ran, last run in the future, or "last ran … minutes ago" (WARN). | Run the WHMCS cron every 5 minutes; check the server clock. |
| Payment gateway | "No payment gateway is active" (WARN). | Activate one, so customers can order. |
| QuantumByte reachable | Not checked without `--qb-host` (WARN). "Can't reach https://…", HTTP 404, or another answer than 401 (FAIL). | Open outbound 443 to `<QB hostname>`, check the hostname and any proxy. |
| Installed version | The installed version is newer than this installer, so installing goes back to an older one (WARN). The same version: "already installed. Nothing to do." | Use the newest installer. |
| Install | "Download failed", the bundle's sha256 doesn't match, the bundle holds unexpected paths, a PHP file doesn't parse, or "WHMCS no longer loads with the new module" (FAIL). | Download again; with `--bundle`, use the archive for this script's version. Send the output to QuantumByte if it repeats. |

## What runs without QuantumByte

A published app needs QuantumByte only to be published again. It serves,
signs its users in and stores data and uploads on the customer's own
server. If QuantumByte is unreachable, the apps keep running.

If QuantumByte ends the partnership, apps already running on customers'
servers keep running with their data and backups. Customers can no longer
publish or download a new release until the partnership resumes. QuantumByte
holds, suspends and deletes nothing on a customer's server, whatever
happens to a service in WHMCS.
