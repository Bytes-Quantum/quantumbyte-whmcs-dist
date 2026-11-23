# Upgrading to 1.6: you resell plans and credits

Version 1.6 follows a change at QuantumByte. A customer's app is published
as a Docker image. In QuantumByte the customer downloads its Docker Compose
files and runs them on any server themselves
([Publishing](02-publishing.md)).
QuantumByte keeps no list of servers, checks no domain and sees nothing of
the server.

So after WHMCS sends the customer to QuantumByte, nothing is left for you to
manage except plans and credits. The module drops everything else.

## What changed

- **The QuantumByte on VPS addon is retired.** It no longer ships. That
  takes with it VPS registration, the connect box on the VPS's page, the
  **VPS products** setting, the **Registrars that may take DNS records**
  setting and the list of DNS records now yours.
- **App domains are gone from the module.** The domain box on the service
  page (connect, disconnect, buy, bring your own) is gone, and so is the
  cron that kept DNS records in step. The module writes no DNS record and
  shows none, because QuantumByte gives a self-managed app no address.
- **Own servers allowed (retired):** AI Builder product → **Module
  Settings**. Leave it as it is. It keeps its place so the settings after it
  do not move, as **Publish to** did in 1.4. The module no longer reads it
  and no longer sends `own_servers_allowed`. It still sends `publish_to:
  vps`, for an older QuantumByte.
- **What stays:** the AI Builder product (create, renew, change package,
  suspend, unsuspend, terminate), credit packs, **Open AI Builder**, Test
  Connection, the daily check for services with no account, and revoking
  the cPanel tokens modules before 1.5 made.
- **The installer** now installs one folder, `modules/servers/quantumbyte`.

## What you control now

- **Suspend** an AI Builder service: publishing is paused, so no new
  releases.
- **Terminate** every AI Builder service of a customer: the account closes
  and the customer's download key stops working, so no new image or content
  downloads.

**QuantumByte can no longer stop an app already running on a customer's
server, and neither can you.** The server and the app belong to the
customer. Details: [Publishing](02-publishing.md).

## Before you upgrade

Deactivate the addon first: Configuration → System Settings → **Addon
Modules** → **QuantumByte on VPS** → **Deactivate**. The old addon code
calls module functions that 1.6 removed, so left active it errors on every
WHMCS cron run.

You can upgrade before or after QuantumByte's own release. QuantumByte keeps
answering modules 1.3 to 1.5 without harm: its server list is empty, a
server registration changes nothing, a server removal succeeds, and
`own_servers_allowed` is accepted and ignored.

## Upgrade

1. Deactivate the addon, as above.
2. **Update the module** to 1.6.0 as in
   [Updating the module](04-operations.md#updating-the-module).
3. Delete the folder `modules/addons/quantumbyte_vps` from your WHMCS root.
4. Open one AI Builder product → **Module Settings** and **Save Changes**,
   so WHMCS shows **Own servers allowed (retired)**.

## The DNS records older versions wrote

Nothing is removed from your DNS: removing records would take live sites
offline. The module no longer touches them, so review them yourself and
remove any you no longer want. Older versions wrote records to the
registrars you listed under **Registrars that may take DNS records**, and to
your cPanel server's zones.

There is no schema to migrate. The tables of older versions stay in the
WHMCS database, unused (`mod_quantumbyte_vps`, `mod_quantumbyte_vps_dns`,
`mod_quantumbyte_vps_connected`, `mod_quantumbyte_domains`,
`mod_quantumbyte_domain_orders`, `mod_quantumbyte_dns_handover`). To find
the records, run these read-only queries on your WHMCS database. Not every
table exists on every install; skip one that is missing.

```sql
-- Records written for customers' app domains
SELECT fqdn, type, value, userid
  FROM mod_quantumbyte_domains
 WHERE state = 'written' AND adopted = 0;

-- Records written for a VPS's own domain (modules 1.0 and 1.1)
SELECT fqdn, type, value, serviceid
  FROM mod_quantumbyte_vps_dns
 WHERE state = 'written';

-- Records already listed as yours by 1.2 to 1.5
SELECT fqdn, type, value, userid
  FROM mod_quantumbyte_dns_handover;
```

A row says the module wrote that record. It does not say the record is
still in your DNS, nor who else uses it. Check each name at its registrar or
cPanel zone before you remove anything. The **Activity Log** may also name
records the module wrote.

## Going back to 1.5

Reinstall 1.5 and its addon. The addon finds QuantumByte's server list
empty and registers nothing that lasts, so it has no effect.

## A reply for your support team

**"Why can't I connect a domain in your client area any more?"**

> Your app runs on your own server, so you set up its domain and DNS
> yourself, with your domain provider. We don't see your server or write
> any records for it.

> Aplikasi Anda berjalan di server Anda sendiri, jadi Anda mengatur domain
> dan DNS-nya sendiri melalui penyedia domain Anda. Kami tidak melihat
> server Anda dan tidak menulis record apa pun untuknya.
