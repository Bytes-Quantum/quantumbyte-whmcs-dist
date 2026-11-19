# Integrating QuantumByte with WHMCS

For the WHMCS admin at a hosting company. Your customers buy an AI app
builder in your store, pay you, sign in from your client area, and publish
their apps. They run each app on a server of their own, with files
QuantumByte gives them. QuantumByte never invoices your customers.

The module (`modules/servers/quantumbyte/`) is a provisioning module. When a
customer's invoice is paid, WHMCS creates a QuantumByte account loaded with
the product's credits. The customer opens it from the client area with
**Open AI Builder**, without a password. It suspends, renews and terminates
the account with the service.

From 1.6 that is all. The QuantumByte on VPS addon is retired, and the
module writes and shows no DNS ([Upgrading to 1.6](11-upgrade-1.6.md)).

## Read in this order

1. [Install](01-install.md): what you need, installing the module, adding
   the server, creating the product, testing an order.
2. [Publishing](02-publishing.md): how customers run their apps on their
   own servers, what you control and what nobody can do.
3. [Your partner portal](03-partner-portal.md): your team's page at
   QuantumByte.
4. [Operations](04-operations.md): suspend, terminate, upgrade, updating
   the module, limits, troubleshooting, what runs without QuantumByte.
5. [Upgrading to 1.1](05-upgrade-1.1.md): customers now connect their own
   VPS; what changes for you and them.
6. [Upgrading to 1.2](07-upgrade-1.2.md): the module and QuantumByte stop
   setting DNS for VPSes and apps on a VPS; records the module already set
   are yours now.
7. [Upgrading to 1.3](08-upgrade-1.3.md): customers on VPS plans may add
   servers of their own, up to a number you set per product.
8. [Upgrading to 1.4](09-upgrade-1.4.md): customers publish only to
   servers of their own; **Publish to** is retired.
9. [Upgrading to 1.5](10-upgrade-1.5.md): the module stops handing cPanel
   accounts to QuantumByte and revokes the cPanel tokens it made.
10. [Upgrading to 1.6](11-upgrade-1.6.md): you resell plans and credits
    only; the VPS addon, app domains and **Own servers allowed** are retired.

## Placeholders

| In these pages | Means |
| --- | --- |
| `whmcs.example.com` | Your WHMCS. |
| `/var/www/whmcs` | Your WHMCS root folder (the one holding `init.php`). |
| `<QB hostname>` | QuantumByte's API hostname, shown when you create an API key in your partner portal. |
| `<API key>` | Your API key, created in your partner portal under **Setup** → **API keys**. |

## Help

Write to support@quantumbyte.ai. Say which WHMCS version you run, and paste
the message you see (Module Queue, Module Log, Activity Log or installer
output). Never send your API key.
