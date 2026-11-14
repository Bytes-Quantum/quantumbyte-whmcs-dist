# Upgrading to 1.1: customers connect their own VPS

For hosters who sell VPS plans with QuantumByte. Version 1.1 changes how
QuantumByte's agent is installed on a customer's VPS. Nothing changes for
Kubernetes or cPanel plans.

## How it works now

**Before (1.0):** on each cron run, the addon logged in to a new VPS over
SSH as root, with the password WHMCS holds, and ran QuantumByte's
installer. A customer who changed their root password broke that login, and
the VPS was never set up.

**From 1.1:** the addon never logs in to a VPS. It still registers each new
VPS on the customer's QuantumByte account and points the VPS's domain at
it. The customer then connects the VPS themselves, with one command that
QuantumByte shows them. The steps are in
[Publishing: what happens after a VPS order is paid](02-publishing.md#publish-to-your-customers-vps).

Nobody but the customer uses the root password. They may change it
whenever they like.

## What to do as a hoster

- **Update the module** to 1.1.0 as in
  [Updating the module](04-operations.md#updating-the-module), on the day
  QuantumByte's upgrade email names.
- **The addon's page** (Addons → QuantumByte on VPS) no longer shows
  install errors or SSH host keys. It shows each VPS as QuantumByte sees
  it: connected or waiting for the customer, when it was last seen, and its
  last release. **Forget host key** is gone.
- **The VPS's page in your client area** shows **Connect this server in
  QuantumByte** until the VPS is connected, then a short "Connected to
  QuantumByte" line with **Manage connection**. It changes on the next cron
  run after the customer connects.
- **Your VPS products** no longer need root login with the password WHMCS
  holds, nor an **SSH Port** custom field. You may delete that field. The
  customer still needs a way to sign in to the VPS and use `sudo`.
- **phpseclib** is no longer needed. The installer stops checking for it.

## What your customers see

- **VPSes already connected:** nothing changes. They keep publishing as
  before.
- **VPSes waiting for their install:** their customers see **Connect your
  server first.** at Publish, with a link to the page that connects the
  server.
- **New VPS orders:** the same, from the first day.

## When the old install stops

It stops when QuantumByte deploys the 1.1 change, before you upgrade. From
then on, a 1.0 addon no longer logs in to any VPS, so it never uses a root
password. Customers connect their VPS themselves, as above.

Until you upgrade:

- The Activity Log shows "Customers now connect their own server in
  QuantumByte. Update the QuantumByte VPS addon to 1.1.0."
- A 1.0 addon stops sending QuantumByte a changed address or domain for a
  VPS that isn't connected yet. Upgrading sends them again on the next cron
  run.

On the date in QuantumByte's upgrade email, the old install route is
removed. That date is at least 14 days after the email. From then on, a 1.0
addon gets an error on each cron run. Upgrade before that date.

## A reply for your support team

Paste this when a customer asks how to connect their server.

**English**

> **How do I connect my server?**
>
> 1. In our client area, open your VPS and press **Connect this server in
>    QuantumByte**.
> 2. Press **Get the install command**.
> 3. QuantumByte asks for your two-step sign-in code. If you haven't set it
>    up yet, it walks you through it first. Keep the recovery codes it gives
>    you.
> 4. Copy the command. Sign in to your VPS, paste the command and press
>    Enter. It asks for your server password, because it runs with
>    administrator rights (sudo).
> 5. Wait for the page to say **Connected**, then press **Publish** on
>    your app.
>
> You can use the command once. It expires after 1 hour; press the button
> again for a new one. If you rebuild the VPS later, use **Reconnect this
> server** on the same page. If you lose your phone and your recovery
> codes, contact us: only QuantumByte support can reset two-step sign-in.

**Bahasa Indonesia**

> **Bagaimana cara menghubungkan server saya?**
>
> 1. Di area klien kami, buka VPS Anda lalu tekan **Hubungkan server ini di
>    QuantumByte**.
> 2. Tekan **Ambil perintah pemasangan**.
> 3. QuantumByte meminta kode verifikasi dua langkah Anda. Jika belum
>    mengaturnya, QuantumByte memandu Anda lebih dulu. Simpan kode pemulihan
>    yang diberikan.
> 4. Salin perintahnya. Masuk ke VPS Anda, tempel perintah itu lalu tekan
>    Enter. Perintah ini meminta kata sandi server Anda, karena berjalan
>    dengan hak administrator (sudo).
> 5. Tunggu sampai halaman menampilkan **Terhubung**, lalu tekan
>    **Publikasikan** di aplikasi Anda.
>
> Perintah itu hanya bisa dipakai sekali dan kedaluwarsa setelah 1 jam;
> tekan tombolnya lagi untuk perintah baru. Jika nanti Anda membangun ulang
> VPS, gunakan **Hubungkan ulang server ini** di halaman yang sama. Jika
> ponsel dan kode pemulihan Anda hilang, hubungi kami: hanya tim dukungan
> QuantumByte yang bisa mengatur ulang verifikasi dua langkah.
