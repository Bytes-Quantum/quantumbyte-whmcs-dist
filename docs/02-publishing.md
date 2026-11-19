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
3. In QuantumByte, downloads two files: `docker-compose.yml` and `.env`.
4. Puts both files in one folder on a server that has Docker, and starts
   the app there. The server can be at any provider, yours or not.
5. Sets up their own domain and DNS for that server. QuantumByte gives the
   app no address and does not check the domain or the server. Your WHMCS
   writes and shows no DNS for the app.

The `.env` file holds the customer's key for downloading the app's image and
content. The customer keeps it private.

Every new release works the same way: the customer publishes, then runs the
new files on their server.

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
> 2. Download `docker-compose.yml` and `.env`.
> 3. Copy both files to one folder on your server. The server needs Docker.
> 4. In that folder, run `docker compose up -d`.
> 5. Point your domain at your server with your own DNS provider.
>
> We cannot see or reach your server. To update the app, publish again and
> run the new files.

**Bahasa Indonesia**

> **Bagaimana cara menjalankan aplikasi saya di server saya?**
>
> 1. Buka **Open AI Builder** di area klien kami lalu tekan
>    **Publikasikan** di aplikasi Anda.
> 2. Unduh `docker-compose.yml` dan `.env`.
> 3. Salin kedua berkas ke satu folder di server Anda. Server perlu Docker.
> 4. Di folder itu, jalankan `docker compose up -d`.
> 5. Arahkan domain Anda ke server dengan penyedia DNS Anda sendiri.
>
> Kami tidak bisa melihat atau menjangkau server Anda. Untuk memperbarui
> aplikasi, publikasikan lagi lalu jalankan berkas yang baru.
