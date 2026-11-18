# Operations

1. [Suspend, terminate, upgrade, update the module](#suspend-terminate-upgrade-update-the-module)
2. [Limits to know](#limits-to-know)
3. [Troubleshooting](#troubleshooting)
4. [What runs without QuantumByte](#what-runs-without-quantumbyte)

## Suspend, terminate, upgrade, update the module

Run these from the AI Builder service's **Module Commands** as usual.

| WHMCS action | What happens |
| --- | --- |
| **Suspend** | The plan stops publishing. What else stops depends on the customer's plans: see [Suspend and terminate by plan](#suspend-and-terminate-by-plan). Data is kept. |
| **Unsuspend** | Publishing, sign-in and apps come back as they were. |
| **Terminate** | Once *every* AI Builder service of the customer is terminated, the account is closed and its data deleted 30 days later. An app on a VPS is never stopped or deleted: see [Suspend and terminate by plan](#suspend-and-terminate-by-plan). |
| **Create** on a terminated service | Within those 30 days: restores the account as it was. No credits granted or charged again. After that, order a new service. |
| **Renewal paid** | The product's Credits load again. A renewal that failed (e.g. your balance ran out) is caught up by the next successful one, or by retrying it in the Module Queue. |
| **Upgrade** | Once its invoice is paid, the extra credits load for what's left of the billing period, as WHMCS prorates it. Each paid upgrade invoice loads once, even after a downgrade in the same period. |
| **Downgrade** | Nothing taken back; the smaller package applies from the next renewal. |
| **Change Package** with no paid upgrade invoice | The package changes; no credits load. To give a customer a bigger package for free, raise an upgrade invoice and mark it paid: the paid invoice is what loads the credits. |
| **Credit pack paid** | Its credits load into the customer's account once, however often Create runs. No app, no plan. Suspend, Unsuspend and Terminate on it change nothing; Renew and Change Package are refused. |

The other services:

- **Terminate the VPS service**: QuantumByte stops using it; the customer's
  app there is not moved anywhere. Suspending the VPS is not passed on:
  QuantumByte still shows it connected; a publish waits until the VPS runs
  it again.

### Suspend and terminate by plan

From 1.4, every plan is treated the same, whatever its retired **Publish to**
setting holds.

**A suspended plan.** Publishing from that plan is refused with
"Publishing is paused. Ask `<you>` about your plan." This holds even while
another plan of the customer is active. The app on the customer's server
keeps running: QuantumByte never stops or deletes anything on their server.

**Suspended, no plan active.** The customer keeps signing in and building.
WHMCS hides its own sign-in on a suspended service, so their service page
shows an **Open AI Builder** button instead. AI work still stops once their
credits run out.

**Several plans.** While any plan is active, the customer keeps signing in;
each suspended plan only stops publishing, as above.

**Terminated.** Once every plan is terminated, the account is closed and
sign-in refused. From that moment QuantumByte refuses the customer's
servers new releases and image downloads; what already runs there keeps
running. 30 days later QuantumByte deletes its own data for the account.
On a server nothing is stopped or deleted: the app, its data and backups
stay. On a server with the minute timer, QuantumByte's agent stops checking
in; a server that updates by command has no agent running to stop. One
exception: a server app on the minute timer that was already on the
"unavailable" page before November 2026's release (update by command) keeps
showing it after the agent stops. Only someone on the server can start the
app again. A server installed again before that (see
[Server security](06-server-security.md#what-runs-as-root-and-when)) gets its
app back at its first `sudo quantumbyte update`. If you restore the
service with **Create**, releases and downloads work again.

### Updating the module

Download the newer installer, check its sha256, and run it as in
[Install with the installer script](01-install.md#install-with-the-installer-script).
It backs up the folders it replaces. Settings live in WHMCS's database, so
nothing needs re-entering.

Coming from a version without `modules/servers/quantumbyte/hooks.php`:
DNS upkeep and bought domains keep running while the
QuantumByte on VPS addon is active. Before you deactivate it, open an AI
Builder product → **Module Settings** → **Save Changes** once, so WHMCS
loads the module's own hooks.

To go back to a backup, run this as the owner of the WHMCS files, with a
backup from `~/quantumbyte-whmcs-backups/` (named by date and time). The
installer prints this line, filled in, at the end of each install:

```sh
cd /var/www/whmcs && rm -rf modules/servers/quantumbyte modules/addons/quantumbyte_vps && tar -xzf ~/quantumbyte-whmcs-backups/<backup>.tar.gz
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
- One app per VPS (a customer may have several).
- VPS: backups stay on the VPS (none leave it). A normal update has no gap:
  visitors stay on the old version until the new one is ready.
- Up to 5 domains per app.
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
- **Activity Log**: VPS hand-over, domain upkeep, and AI
  Builder services that are Active with no QuantumByte account (one line
  each). A cron check that keeps failing (a refused key, QuantumByte
  unreachable) is logged once per server entry, and once more when it works
  again.
- **Admin dashboard**: names each QuantumByte server entry whose checks keep
  failing, until they work again, and lists AI Builder services that are
  Active with no QuantumByte account.
- **Addons → QuantumByte on VPS**: each VPS as QuantumByte sees it:
  connected or waiting for the customer to connect it, when it was last
  seen, its last release and the release it runs.
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
| "…hasn't finished setting up publishing for your plan" | The customer has no server and can't add another of their own: raise **Own servers allowed** on their product, or sell them a VPS. |
| "AI Builder service #N is Active, but has no live QuantumByte account" (Activity Log, service page, dashboard) | The order was added as Active, or the service was set Active by hand. Press **Create** under Module Commands. |
| "Connect your server first." (the customer, at Publish) | The customer hasn't connected their VPS: they press **Connect this server in QuantumByte** on the VPS's page and run the command it shows ([Publishing](02-publishing.md#publish-to-your-customers-vps)). |
| "…is still setting up your server" | The VPS has no domain yet: set the domain on the VPS service. |
| "Customers now connect their own server in QuantumByte. Update the QuantumByte VPS addon to 1.1.0." | Your addon is older than 1.1.0: [update the module](05-upgrade-1.1.md). |
| "This app already has the maximum of 5 domains" | Disconnect a domain first. |

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
signs its users in and stores data and uploads on the customer's VPS. If
QuantumByte is unreachable, the apps keep running.

If QuantumByte ends the partnership, each VPS agent on the minute timer
switches itself off and leaves the app, its data and its backups running. A
VPS that updates by command has no QuantumByte agent running to switch off; its next
`sudo quantumbyte update` brings no new app version, only signed agent
updates. From then on QuantumByte holds, suspends and deletes nothing on
your side, whatever happens to a service in WHMCS. If the partnership
resumes, each VPS customer runs the install command again to connect their
server. This does not touch the app. An app on a VPS is never stopped or
deleted. The server belongs to the customer.
