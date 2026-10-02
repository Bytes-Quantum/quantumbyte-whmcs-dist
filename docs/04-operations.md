# Operations

1. [Suspend, terminate, upgrade, update the module](#suspend-terminate-upgrade-update-the-module)
2. [Limits to know](#limits-to-know)
3. [Troubleshooting](#troubleshooting)
4. [What runs without QuantumByte](#what-runs-without-quantumbyte)

## Suspend, terminate, upgrade, update the module

Run these from the AI Builder service's **Module Commands** as usual.

| WHMCS action | What happens |
| --- | --- |
| **Suspend** | Once *every* AI Builder service of the customer is suspended: sign-in refused; their apps answer "This site is temporarily unavailable" (HTTP 503) on every target (your Kubernetes, their VPS, their cPanel). Data is kept. |
| **Unsuspend** | Sign-in and apps come back as they were. |
| **Terminate** | Once *every* AI Builder service of the customer is terminated: account closed, apps answer "This site is no longer available" (HTTP 404). 30 days later everything is deleted, including the app, its data and backups on your cluster, their VPS or their cPanel account, and the app's database on your MongoDB. |
| **Create** on a terminated service | Within those 30 days: restores the account as it was. No credits granted or charged again. After that, order a new service. |
| **Renewal paid** | The product's Credits load again. A renewal that failed (e.g. your balance ran out) is caught up by the next successful one, or by retrying it in the Module Queue. |
| **Upgrade** | The extra credits load for what's left of the billing period, as WHMCS prorates it. |
| **Downgrade** | Nothing taken back; the smaller package applies from the next renewal. |

The other services:

- **Terminate the VPS service**: QuantumByte stops using it; the customer's
  app there is not moved anywhere. Suspending the VPS is not passed on:
  QuantumByte just sees it offline.
- **Terminate the cPanel service**: WHMCS's cPanel module deletes the
  account; QuantumByte stops publishing there and the module revokes its
  token. Suspending the cPanel service takes the site down with the
  account.
- **Terminate the AI Builder service** of a cPanel customer: the cPanel
  token stays with QuantumByte until the purge 30 days later, which
  deletes the app from the account and then revokes the token.

### Updating the module

Download the newer installer, check its sha256, and run it as in
[Install with the installer script](01-install.md#install-with-the-installer-script).
It backs up the folders it replaces. Settings live in WHMCS's database, so
nothing needs re-entering.

Coming from a version without `modules/servers/quantumbyte/hooks.php`:
DNS upkeep, bought domains and cPanel hand-over keep running while the
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
  credits into the same account.
- A customer email that already has a QuantumByte account is refused.
- Customers sign in only through you. QuantumByte's own password, Google
  and email sign-in refuse them.
- VPS and cPanel: one app per VPS, one app per cPanel account.
- Your Kubernetes: customers' own domains take names like `www`, never the
  bare domain (`@`).
- cPanel: no custom domains through QuantumByte. The app runs on the
  account's main domain; other domains are managed in the hosting account.
- VPS: backups stay on the VPS (none leave it). On a 1 GB VPS an update
  leaves the app unanswered for about 4 seconds.
- Up to 5 domains per app.
- Credits don't block building or publishing yet.

## Troubleshooting

**Where to look**

- **Utilities → Module Queue**: failed creates, renewals, upgrades, with
  QuantumByte's message. Fix the cause, then **Retry** (always safe).
- **Configuration → System Logs → Module Log**: every call to QuantumByte,
  with the API key, sign-in links, install commands and cPanel tokens
  scrubbed. Turn module logging on while you debug.
- **Activity Log**: VPS hand-over, cPanel tokens and domain upkeep.
- **Addons → QuantumByte on VPS**: each VPS's status, last install error
  and SSH host key.
- Partner portal: balance, statements, **Customer apps**, and the problems
  the Kubernetes and database cards show.

**Common messages**

| Message | Fix |
| --- | --- |
| Test Connection "Set Configuration → System Settings → General Settings → System URL to https://…" | Set System URL to `https://…` ([Before you start](01-install.md#before-you-start)). |
| Test Connection "Unauthorized" | Wrong or replaced API key. Check with QuantumByte. |
| "QuantumByte has paused this partnership. Contact QuantumByte." | Contact QuantumByte. |
| "Not enough QuantumByte balance…" / "…credit…" | Top up or pay what you owe, then Retry. |
| "Your QuantumByte statement for … was due on …" | Pay the statement in the partner portal, then Retry. |
| "QuantumByte has not set your prices yet" | Ask QuantumByte to set your terms. |
| "app_link.token_url must be https://<your WHMCS>/oauth/token.php" | Set System URL to `https://…` ([Before you start](01-install.md#before-you-start)). |
| "… already has a QuantumByte account. Ask the customer to use another email address for this service." | The customer uses another email for this service. |
| "…hasn't finished setting up publishing for your plan" | Connect your cluster or your cPanel database ([Publishing](02-publishing.md)). |
| "…is still setting up your server" | The VPS install hasn't finished: see the addon page. |
| "The VPS's SSH host key changed…" | Rebuilt VPS: **Forget host key** on the addon page. |
| "…already has a website on <domain>" | Empty the cPanel account's `public_html` and remove other Application Manager apps. |
| "Missing rights in <namespace>: …" | Run the setup commands again, then **Check again**. |
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
| phpseclib | "phpseclib\Net\SSH2 isn't available" (WARN). | Needed only for VPS plans. |
| System URL | Not set, not `https`, or an IP, `localhost` or a port (WARN). | Set it to your public `https://` address before selling: Test Connection fails without it. |
| Cron | Never ran, last run in the future, or "last ran … minutes ago" (WARN). | Run the WHMCS cron every 5 minutes; check the server clock. |
| Payment gateway | "No payment gateway is active" (WARN). | Activate one, so customers can order. |
| QuantumByte reachable | Not checked without `--qb-host` (WARN). "Can't reach https://…", HTTP 404, or another answer than 401 (FAIL). | Open outbound 443 to `<QB hostname>`, check the hostname and any proxy. |
| Installed version | The installed version is newer than this installer, so installing goes back to an older one (WARN). The same version: "already installed. Nothing to do." | Use the newest installer. |
| Install | "Download failed", the bundle's sha256 doesn't match, the bundle holds unexpected paths, a PHP file doesn't parse, or "WHMCS no longer loads with the new module" (FAIL). | Download again; with `--bundle`, use the archive for this script's version. Send the output to QuantumByte if it repeats. |

## What runs without QuantumByte

A published app needs QuantumByte only to be published again. It serves,
signs its users in and stores data and uploads on your side: in your
cluster, on the customer's VPS, or in the cPanel account with your
MongoDB. If QuantumByte is unreachable, the apps keep running. The one
exception is a Kubernetes pod that starts on a node which has not run the
app before: it waits until QuantumByte is reachable, then pulls its image.

If QuantumByte ends the partnership, it gives up what it holds of yours:
the kubeconfig, the MongoDB connection string and the cPanel account tokens
are deleted, and each VPS agent switches itself off and leaves the app, its
data and its backups running. From then on QuantumByte holds, suspends and
deletes nothing on your side, whatever happens to a service in WHMCS. If
the partnership resumes, connect the cluster and the database again; the
cPanel tokens are sent again on the next Open AI Builder, and the addon
installs the agent again on each VPS without touching the app.
