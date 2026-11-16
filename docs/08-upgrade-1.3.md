# Upgrading to 1.3: customers may add servers of their own

For hosters who sell AI Builder plans that publish to the customer's VPS.
Version 1.3 adds one product setting, **Own servers allowed**. With it, a
customer may also publish to servers they rent somewhere else, up to the
number you set. It is off until you set it. Nothing changes for Kubernetes
or cPanel plans.

## What changed

- **A new product setting:** AI Builder product → **Module Settings** →
  **Own servers allowed**, from 0 to 100. How it works:
  [Customers' own servers](02-publishing.md#customers-own-servers).
- **The module sends it to QuantumByte** next to **Publish to**, at each
  create, upgrade or downgrade, renewal, suspend and unsuspend, and when the
  customer opens AI Builder (a suspended VPS plan's **Open AI Builder**
  too). An empty setting is never sent, so QuantumByte keeps what it has.
- **The addon's page** (Addons → QuantumByte on VPS) lists **Customers' own
  servers**: the client, how many they have, the domain, whether it is
  connected, and when QuantumByte last heard from it.

## Before you upgrade

You can upgrade before or after QuantumByte's own release. A QuantumByte
from before it ignores the new setting, and nothing fails. Customers can
add servers of their own only once both are live, so upgrading after
QuantumByte's email says its side is live is simplest.

## Upgrade

1. **Update the module** to 1.3.0 as in
   [Updating the module](04-operations.md#updating-the-module).
2. Products you made before 1.3 have the setting empty, so nothing is sent
   for them and their customers can't add servers of their own. Open each
   AI Builder product → **Module Settings**, enter a number in **Own
   servers allowed** (`0` for none) and **Save Changes**.
3. The new number reaches QuantumByte at the customer's next sign-in to
   AI Builder, or their next renewal, upgrade, suspend or unsuspend. To apply it at once,
   ask the customer to open AI Builder from your client area.

## Going back to 1.2

1.2 stops sending the setting. QuantumByte then keeps the last number it
heard for each service, and the customer may still add servers up to that
number. To turn it off before you go back:

1. Set **Own servers allowed** to `0` on the product and save.
2. Let it reach QuantumByte for each service: the customer's next sign-in
   to AI Builder, a renewal, an upgrade, or a suspend and unsuspend.
3. Then reinstall 1.2.

Servers customers already added stay, and their apps keep running. If you
went back without step 1, write to support@quantumbyte.ai to have the
number set to 0.

## A reply for your support team

**"Can I use a server I rent somewhere else?"**

> Yes, if your plan allows it. Open AI Builder, go to **Your servers** and
> add your server. QuantumByte then shows one command to run on it. Your
> plan says how many servers of your own you can add. You set the DNS
> record for your server's name yourself, where you manage that domain.

> Bisa, jika paket Anda mengizinkan. Buka AI Builder, masuk ke **Server
> Anda** (**Your servers** dalam bahasa Inggris), lalu tambahkan server Anda. QuantumByte akan menampilkan satu
> perintah untuk dijalankan di server itu. Paket Anda menentukan berapa
> server milik Anda sendiri yang bisa ditambahkan. Record DNS untuk nama
> server Anda, Anda pasang sendiri di tempat Anda mengelola DNS domain
> tersebut.
