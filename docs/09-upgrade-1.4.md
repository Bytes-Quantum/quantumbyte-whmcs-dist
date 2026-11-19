# Upgrading to 1.4: customers publish only to servers of their own

Servers are not added in QuantumByte any more from 1.6, and **Own servers
allowed** is retired: see [Upgrading to 1.6](11-upgrade-1.6.md). This page
is what 1.4 did.

Version 1.4 retires the product setting **Publish to**. Every customer's apps
go live on a server of their own, which they add in QuantumByte under **Your
servers**. Nothing is lost: the setting keeps its place on the product, so
the settings after it do not move.

## What changed

- **Publish to (retired):** AI Builder product → **Module Settings**. It has
  one option now. Leave it as it is. The module always tells QuantumByte
  `vps`, and QuantumByte ignores the setting for products saved by earlier
  versions.
- **One server for everyone:** a customer may always add one server of
  their own, even when **Own servers allowed** is `0` or empty. A number
  above 1 allows that many.
- **The domain is optional** when a customer adds a server. The app runs
  there with no public address until a domain is attached.
- **Suspended plans** of every kind keep the builder: QuantumByte only
  pauses publishing, and the service page shows **Open AI Builder**.
- **Terminated services:** once every service of a customer is terminated,
  QuantumByte refuses their servers new releases and image downloads. The
  app already running there keeps running.
- **Addon version:** QuantumByte on VPS shows 1.4.0.

## Before you upgrade

You can upgrade before or after QuantumByte's own release. An older
QuantumByte accepts what the module sends, and the module still sends
`publish_to: vps`.

## Upgrade

1. **Update the module** to 1.4.0 as in
   [Updating the module](04-operations.md#updating-the-module).
2. Open one AI Builder product → **Module Settings** and **Save Changes**,
   so WHMCS shows the retired setting. Nothing else is needed.

## Two things to know

**Re-save products that were set to cPanel.** WHMCS keeps the old value of
**Publish to** until you save the product again. While a product still says
**cPanel**, the module keeps sending the client's cPanel API token to
QuantumByte. QuantumByte no longer uses these tokens. Open each AI Builder
product that was set to cPanel, then **Module Settings** and **Save
Changes**. The module stops sending the token from then on.

**Older module versions hide the builder link for suspended clients.** On
module versions before 1.4, a suspended client whose product was set to
Kubernetes or cPanel gets no **Open AI Builder** link in WHMCS. Updating the
module to 1.4.0 fixes this. You do not need to save the products for it.

## Going back to 1.3

Reinstall 1.3. Its **Publish to** setting reads as empty until you choose
a value again. QuantumByte keeps sending customers to servers of their own
whatever you choose.

## A reply for your support team

**"I have no domain for my server yet."**

> You can add the server without one. Leave the web address empty. Your app
> runs on the server but has no public address yet. Add a domain to the app
> when you have one.

> Anda bisa menambahkan server tanpa domain. Kosongkan alamat web. Aplikasi
> Anda berjalan di server, tetapi belum punya alamat publik. Tambahkan
> domain ke aplikasi saat Anda sudah punya.
