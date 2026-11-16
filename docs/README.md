# Integrating QuantumByte with WHMCS

For the WHMCS admin at a hosting company. Your customers buy an AI app
builder in your store, pay you, sign in from your client area, and publish
their apps to infrastructure you choose: your Kubernetes, their VPS, or
their cPanel account. QuantumByte never invoices your customers.

The module has two parts:

- **QuantumByte** (`modules/servers/quantumbyte/`): a provisioning module.
  When a customer's invoice is paid, WHMCS creates a QuantumByte account
  loaded with the product's credits. The customer opens it from the client
  area with **Open AI Builder**, without a password. It also hands cPanel
  accounts to QuantumByte and keeps your customers' DNS records in step for
  apps on your Kubernetes.
- **QuantumByte on VPS** (`modules/addons/quantumbyte_vps/`): an addon that
  hands VPSes to QuantumByte and shows whether their domains point at them
  (you set those records; it never writes DNS for a VPS). Your
  customers connect their VPSes themselves, from QuantumByte. Activate it if
  you sell VPS plans, or to let registrars take DNS records.

## Read in this order

1. [Install](01-install.md): what you need, installing the module, adding
   the server, creating the product, testing an order.
2. [Publishing](02-publishing.md): where your customers' apps go live (your
   Kubernetes, their VPS, their cPanel account) and what each needs.
3. [Domains and the partner portal](03-domains-and-partner-portal.md):
   customers' own domains, and your team's page at QuantumByte.
4. [Operations](04-operations.md): suspend, terminate, upgrade, updating
   the module, limits, troubleshooting, what runs without QuantumByte.
5. [Upgrading to 1.1](05-upgrade-1.1.md): customers now connect their own
   VPS; what changes for you and them.
6. [Server security](06-server-security.md): what QuantumByte can and can't
   do on a customer's VPS, for your security team. English and Indonesian.
7. [Upgrading to 1.2](07-upgrade-1.2.md): the module and QuantumByte stop
   setting DNS for VPSes and apps on a VPS; records the module already set
   are yours now.
8. [Upgrading to 1.3](08-upgrade-1.3.md): customers on VPS plans may add
   servers of their own, up to a number you set per product.

## Placeholders

| In these pages | Means |
| --- | --- |
| `whmcs.example.com` | Your WHMCS. |
| `/var/www/whmcs` | Your WHMCS root folder (the one holding `init.php`). |
| `<QB hostname>` | QuantumByte's API hostname, shown when you create an API key in your partner portal. |
| `<API key>` | Your API key, created in your partner portal under **Setup** → **API keys**. |
| `apps.example.com` | A domain of yours that customers' apps go under. |

## Help

Write to support@quantumbyte.ai. Say which WHMCS version you run, and paste
the message you see (Module Queue, Module Log, Activity Log or installer
output). Never send your API key.
