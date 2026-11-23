# Publishing

From 1.6, your customers run their own apps. QuantumByte builds the app and
hands the customer the files to run it. The customer starts it on any server
they choose. QuantumByte keeps no list of servers, checks no domain and sees
nothing of the server.

1. [What the customer does](#what-the-customer-does)
2. [What you control](#what-you-control)
3. [What nobody can do any more](#what-nobody-can-do-any-more)
4. [If you sell VPS plans](#if-you-sell-vps-plans)
5. [A reply for your support team](#a-reply-for-your-support-team)

## What the customer does

1. Builds the app in AI Builder, signed in from your client area with
   **Open AI Builder**.
2. Presses **Publish**. QuantumByte makes the app into a Docker image.
3. In **App Config**, chooses the database (**Included MongoDB** or **My own
   MongoDB**), ticks **This is a new install** and presses **Download**. The
   download asks for a code from an authenticator app. A new install has
   four files: `docker-compose.yml`, `.env`, `content-apply.js` and
   `local.env`.
4. Puts the four files in one folder on a server with Docker and the
   Compose plugin, at any provider, yours or not. Runs
   `chmod 600 .env local.env`, logs in to QuantumByte's registry with the
   `docker login` line App Config shows (the download key is the password),
   then runs `docker compose up -d`.
5. The app answers on `127.0.0.1:3000`. The customer puts a web server
   (Caddy, nginx or Apache) in front of it, points their domain at the
   server and opens ports 80 and 443. **How to install and update** in App
   Config gives the lines to paste. QuantumByte gives the app no address
   and does not check the domain or the server. Your WHMCS writes and shows
   no DNS for the app.

`local.env` holds the customer's download key and, with the included
MongoDB, the database password. An update never replaces it. `.env` holds
the release and the app's settings. Both stay private.

To release a new version, the customer publishes again, downloads again
with **This is a new install** left off, replaces the files in the folder,
then runs `docker compose pull && docker compose up -d`.

The server needs outbound HTTPS (443) to QuantumByte's registry and API,
named in the downloaded `.env`, on install and on every update.

## What you control

| You do | What happens |
| --- | --- |
| **Suspend** the AI Builder service | Publishing is paused: no new releases. The customer can still sign in and build. |
| **Unsuspend** | Publishing works again. |
| **Terminate** every AI Builder service of the customer | The account closes. The customer's download key stops working, so no new image or content can be downloaded. |
| **Create** the terminated service again, within 30 days | The account is restored and the key works again. |

After the customer signs in to QuantumByte, plans and credits are all that
is left for you to manage. Credits, plans and renewals work as before
([Operations](04-operations.md)).

## What nobody can do any more

An app already running on the customer's server keeps running. Neither
QuantumByte nor you can stop it. Suspending or terminating changes only what
the customer can publish and download from now on.

QuantumByte and you also cannot:

- see which servers a customer has, or whether an app runs on them;
- see a server's address, health or last release;
- limit how many servers a customer uses;
- reach into a server: nothing of QuantumByte runs there that connects back
  to it.

The customer's server, data, backups and certificates belong to the customer.

## If you sell VPS plans

A VPS you sell is a server like any other. A customer may run their app on
it, but the module no longer registers it at QuantumByte or links it from
the VPS's page. Setting its domain record is your own business, with your
own DNS tools.

## A reply for your support team

Paste this when a customer asks how to run their app.

**English**

> **How do I run my app on my server?**
>
> 1. Open **Open AI Builder** in our client area and press **Publish** on
>    your app.
> 2. When the build is done, open **App Config**, tick **This is a new
>    install** and press **Download**.
> 3. Put the four files in one folder on your server. The server needs
>    Docker with the Compose plugin.
> 4. Follow **How to install and update** in App Config: log in to the
>    registry with your download key, then run `docker compose up -d`.
> 5. Put a web server in front of the app and point your domain at your
>    server. The same page gives the lines to paste.
>
> We cannot see or reach your server. To update the app, publish again,
> download again with the new-install box off, replace the files and run
> `docker compose pull && docker compose up -d`.

**Bahasa Indonesia**

> **Bagaimana cara menjalankan aplikasi saya di server saya?**
>
> 1. Buka **Open AI Builder** di area klien kami lalu tekan
>    **Publikasikan** di aplikasi Anda.
> 2. Setelah build selesai, buka **App Config**, centang **Ini pemasangan
>    baru** lalu tekan **Unduh**.
> 3. Taruh keempat file di satu folder di server Anda. Server perlu Docker
>    dengan Compose plugin.
> 4. Ikuti **Cara memasang dan memperbarui** di App Config: login ke
>    registry dengan kunci unduhan Anda, lalu jalankan
>    `docker compose up -d`.
> 5. Pasang web server di depan aplikasi dan arahkan domain Anda ke server.
>    Halaman yang sama memberi baris yang tinggal ditempel.
>
> Kami tidak bisa melihat atau menjangkau server Anda. Untuk memperbarui
> aplikasi, publikasikan lagi, unduh lagi tanpa mencentang pemasangan baru,
> ganti file-nya, lalu jalankan
> `docker compose pull && docker compose up -d`.
