# Server security: what QuantumByte can and can't do on a VPS

For your security team, and for customers who ask. It covers a VPS
connected to QuantumByte with the install command (module 1.1). Each claim
names the file on the VPS that makes it true, so you can read it there.
[Bahasa Indonesia below.](#keamanan-server-apa-yang-bisa-dan-tidak-bisa-dilakukan-quantumbyte-di-vps)

## What runs as root, and when

| What | When | File on the VPS |
| --- | --- | --- |
| The installer | Once, when the customer runs the install command with `sudo` | the `install.sh` they downloaded |
| The agent, on a VPS that updates by command | Only when the customer runs `sudo quantumbyte update`, after each Publish. No QuantumByte timer runs in the background, apart from the backup below. | `/opt/quantumbyte/agent.sh`, started by `/usr/local/bin/quantumbyte` (a link to `/opt/quantumbyte/quantumbyte`); `QB_UPDATE=manual` in `/etc/quantumbyte/agent.env` |
| The agent, on a VPS connected before update by command | Every minute, started by systemd | `/opt/quantumbyte/agent.sh`, `/etc/systemd/system/quantumbyte-agent.timer` |
| The backup | Every night at 02:30 in the server's local time zone, up to 30 minutes later | `/opt/quantumbyte/backup.sh`, `/etc/systemd/system/quantumbyte-backup.timer` |

Every VPS connected from now on updates by command. A VPS connected before that
keeps the minute timer until its customer installs it again; QuantumByte cannot
move it remotely. To move it, the customer follows these steps. The app, its
data, its domains and its web set-up stay as they are.

1. Open the server's **Connect your server** page in QuantumByte (linked from
   the VPS's page in WHMCS and from the Publish dialog), press **Reconnect
   this server**, and type its domain and a two-step sign-in code. Copy the
   install command it shows.
2. On the VPS, run
   `sudo systemctl disable --now quantumbyte-agent.timer && sudo rm /etc/quantumbyte/agent.env`.
3. Run the install command. It ends with "Done.", and QuantumByte shows the
   server connected.
4. After each Publish, run `sudo quantumbyte update`. If the VPS still holds
   another app's data, it stops first: type the domain to erase it, or run
   `sudo quantumbyte update --erase-previous-app`.

To check a VPS, run `systemctl list-timers | grep quantumbyte`: on one that
updates by command, it lists only the backup timer.

The app, its database (MongoDB) and its web proxy (Caddy) run in Docker
containers, each with a memory cap (`/opt/quantumbyte/compose.yaml`). None
of them is privileged or mounts the server's own files, apart from Caddy
reading its config from `/opt/quantumbyte`.

## What talks to QuantumByte, and what never does

- **The agent** asks QuantumByte's API which release to run, reports how
  it went, and downloads its own updates. Each call goes out from the VPS
  (`/opt/quantumbyte/agent.sh`, the `curl` lines; the address is `QB_API`
  in `/etc/quantumbyte/agent.env`). On a VPS that updates by command, it
  runs only while the customer's `sudo quantumbyte update` runs, and never
  holds, removes or erases the app because QuantumByte asks. It erases the
  previous app's data only when the customer types the domain to confirm,
  or runs `sudo quantumbyte update --erase-previous-app`.
- **QuantumByte** opens the app's public address after an update, as any
  visitor would, to see that the new version answers. Caddy puts the
  version in the `X-Qb-Release` response header so QuantumByte can tell
  which one answered.
- **Docker** pulls the app's image through QuantumByte's registry, with a
  login that lasts 5 minutes and reads only that app's image. The database
  and proxy images (`mongo:7`, `caddy:2`) come from Docker Hub the first
  time they start.
- **Caddy** asks Let's Encrypt for each domain's certificate, but only when
  QuantumByte's HTTPS front door is on, or on a VPS connected before local
  ports. Behind the customer's own web server, that web server gets the
  certificates.
- **The backup** calls nobody. It dumps the database to
  `/var/lib/quantumbyte/backups/` and keeps the newest 7
  (`/opt/quantumbyte/backup.sh`).
- **Nothing listens for QuantumByte.** The app answers on `127.0.0.1` only
  (port 3100 or the next free one), and so does the agent's own check
  (port 8080 or the next free one; `local.yaml`, `ports:`). The customer's
  web server forwards visitors to the app. With the front door on, Caddy
  also serves ports 80 and 443 for visitors. A VPS connected before local
  ports (QuantumByte release of November 2026) serves 80 and 443 itself,
  as before.

## What is signed, and what isn't yet

Signed: every agent file QuantumByte installs. They are listed with their
checksums in a manifest signed with Ed25519, once per release. QuantumByte
keeps the private release key on Nathan's own machines, never on
QuantumByte's servers or in its build system. Customer servers trust only
the release keys.

- The install command checks the signature, and the installer's checksum,
  before `sudo` runs anything.
- The installer first installs four standard packages (`ca-certificates`,
  `curl`, `jq`, `openssl`). It then checks every agent file before it
  installs Docker, registers the VPS or writes any file of QuantumByte's. A
  failed check stops the install with "signature failed". By then only
  those four packages may be new on the VPS.
- The agent installs no update unless the signature and every checksum
  match, and never goes back to an older signed set. On a failed check it
  keeps its current files, reports the refusal, and keeps running the app.
- The trusted public keys are in `/opt/quantumbyte/agent.sh` and
  `install.sh` (`QB_KEY_CURRENT`, `QB_KEY_NEXT`). A copy is in
  `/etc/quantumbyte/keys/`. The fingerprints are in
  [Publishing](02-publishing.md#publish-to-your-customers-vps).

Not signed yet:

- **The app's image, and the settings sent with it** (image name and
  environment variables). Whoever controls QuantumByte's API chooses what
  runs inside the app's container. That container reaches its own database
  and the internet, not the server's files or other programs.
- **The registry login token.**
- **One step on VPSes connected before this release.** Their agent fetches
  the first checking agent without a check, once. Every update after that
  is checked.
- **The key the install command prints** comes from QuantumByte's API.
  Compare it with the fingerprint in the docs to check it from outside.
- **Not quantum-safe.** Ed25519 can be broken by a large quantum computer.
  Post-quantum signatures need OpenSSL 3.5, which Ubuntu 22.04 and 24.04
  don't ship. The spare key slot is how a post-quantum key can be added
  later, and the move must finish before such a computer exists.

## Who keeps what up to date

| Part | Who updates it |
| --- | --- |
| The operating system | The customer (or you, if your plan includes it). QuantumByte runs `apt` only during the install. |
| Docker | The customer, with the system's updates. The installer adds Docker's own package source (`/etc/apt/sources.list.d/docker.list`). |
| Caddy and MongoDB containers | Today, nobody updates them automatically. They stay at the version pulled on the first start. |
| The app | QuantumByte, every time the customer presses Publish. On a VPS that updates by command, once the customer then runs `sudo quantumbyte update`. |
| The agent | QuantumByte, through signed updates. On a VPS that updates by command, they arrive with the customer's next `sudo quantumbyte update`. |

## What QuantumByte does not do

- **Log in.** The install creates no user, adds no SSH key, and keeps no
  password.
- **Read the root password.** The customer types it into `sudo` on their own
  VPS. It never reaches QuantumByte and may be changed at any time.
- **Connect inwards.** Nothing on the VPS waits for a connection from
  QuantumByte. Every exchange starts on the VPS.

The agent runs as root. A signed agent update could therefore do anything
root can. That is why the release key is never on QuantumByte's servers or
in its build system, and why the VPS refuses any agent file not signed with
it, even when it comes from QuantumByte's own servers.

---

# Keamanan server: apa yang bisa dan tidak bisa dilakukan QuantumByte di VPS

Untuk tim keamanan Anda, dan untuk pelanggan yang bertanya. Halaman ini
membahas VPS yang terhubung ke QuantumByte dengan perintah instalasi (modul
1.1). Setiap pernyataan menyebut file di VPS yang membuatnya benar, jadi
Anda bisa membacanya sendiri di sana.

## Apa yang berjalan sebagai root, dan kapan

| Apa | Kapan | File di VPS |
| --- | --- | --- |
| Penginstal | Sekali, saat pelanggan menjalankan perintah instalasi dengan `sudo` | `install.sh` yang mereka unduh |
| Agen, di VPS yang diperbarui lewat perintah | Hanya saat pelanggan menjalankan `sudo quantumbyte update`, setelah setiap kali menekan Publikasikan. Tidak ada timer QuantumByte yang berjalan di latar belakang, selain cadangan di bawah. | `/opt/quantumbyte/agent.sh`, dijalankan oleh `/usr/local/bin/quantumbyte` (tautan ke `/opt/quantumbyte/quantumbyte`); `QB_UPDATE=manual` di `/etc/quantumbyte/agent.env` |
| Agen, di VPS yang terhubung sebelum pembaruan lewat perintah | Setiap menit, dijalankan oleh systemd | `/opt/quantumbyte/agent.sh`, `/etc/systemd/system/quantumbyte-agent.timer` |
| Cadangan (backup) | Setiap malam pukul 02.30 waktu setempat server, paling lambat 30 menit setelahnya | `/opt/quantumbyte/backup.sh`, `/etc/systemd/system/quantumbyte-backup.timer` |

Setiap VPS yang terhubung mulai sekarang diperbarui lewat perintah. VPS yang
terhubung sebelum itu tetap memakai timer per menit sampai pelanggannya
memasangnya ulang; QuantumByte tidak bisa memindahkannya dari jarak jauh.
Untuk memindahkannya, pelanggan mengikuti langkah berikut. Aplikasi, data,
domain, dan pengaturan webnya tetap seperti semula.

1. Buka halaman **Hubungkan server Anda** untuk server itu di QuantumByte
   (ada tautannya di halaman VPS di WHMCS dan di dialog Publikasikan), tekan
   **Hubungkan ulang server ini**, lalu ketik domainnya dan kode verifikasi
   dua langkah. Salin perintah instalasi yang muncul.
2. Di VPS, jalankan
   `sudo systemctl disable --now quantumbyte-agent.timer && sudo rm /etc/quantumbyte/agent.env`.
3. Jalankan perintah instalasi itu. Di akhir, perintah menampilkan "Done.",
   lalu QuantumByte menampilkan server sudah terhubung.
4. Setiap selesai menekan Publikasikan, jalankan `sudo quantumbyte update`.
   Jika VPS masih menyimpan data aplikasi lain, perintah berhenti dulu: ketik
   domainnya untuk menghapus data itu, atau jalankan
   `sudo quantumbyte update --erase-previous-app`.

Untuk mengecek sebuah VPS, jalankan `systemctl list-timers | grep quantumbyte`:
di VPS yang diperbarui lewat perintah, hasilnya hanya timer cadangan.

Aplikasi, databasenya (MongoDB) dan proxy webnya (Caddy) berjalan di
container Docker, masing-masing dengan batas memori
(`/opt/quantumbyte/compose.yaml`). Tidak ada yang berjalan dengan hak
istimewa (privileged) atau memasang file milik server, kecuali Caddy yang
membaca konfigurasinya dari `/opt/quantumbyte`.

## Apa yang menghubungi QuantumByte, dan apa yang tidak pernah

- **Agen** menanyakan ke API QuantumByte rilis mana yang harus dijalankan,
  melaporkan hasilnya, dan mengunduh pembaruannya sendiri. Setiap panggilan
  keluar dari VPS (`/opt/quantumbyte/agent.sh`, baris `curl`; alamatnya
  adalah `QB_API` di `/etc/quantumbyte/agent.env`). Di VPS yang diperbarui
  lewat perintah, agen hanya berjalan selama pelanggan menjalankan
  `sudo quantumbyte update`, dan tidak pernah menangguhkan aplikasi,
  mencopotnya, atau menghapus datanya hanya karena diminta QuantumByte. Agen
  menghapus data aplikasi sebelumnya hanya jika pelanggan mengetik domainnya
  untuk mengonfirmasi, atau menjalankan
  `sudo quantumbyte update --erase-previous-app`.
- **QuantumByte** membuka alamat publik aplikasi setelah pembaruan, seperti
  pengunjung biasa, untuk memastikan versi baru sudah merespons. Caddy
  mencantumkan versinya di header respons `X-Qb-Release` agar QuantumByte
  bisa mengeceknya.
- **Docker** menarik image aplikasi lewat registry QuantumByte, dengan
  login yang berlaku 5 menit dan hanya bisa membaca image aplikasi itu.
  Image database dan proxy (`mongo:7`, `caddy:2`) diambil dari Docker Hub
  saat pertama kali dijalankan.
- **Caddy** meminta sertifikat setiap domain ke Let's Encrypt, hanya jika
  pintu depan HTTPS (*front door*) QuantumByte dinyalakan, atau di VPS yang
  tersambung sebelum port lokal ada. Jika aplikasi berada di
  belakang web server milik pelanggan, web server itulah yang mengurus
  sertifikat.
- **Cadangan** tidak menghubungi siapa pun. Ia menyalin database ke
  `/var/lib/quantumbyte/backups/` dan menyimpan 7 yang terbaru
  (`/opt/quantumbyte/backup.sh`).
- **Tidak ada yang menunggu koneksi dari QuantumByte.** Aplikasi hanya
  menjawab di `127.0.0.1` (port 3100 atau port kosong berikutnya), begitu
  juga pemeriksaan agen sendiri (8080 atau port kosong berikutnya;
  `local.yaml`, `ports:`). Web server pelanggan meneruskan pengunjung ke
  aplikasi. Jika pintu depan dinyalakan, Caddy juga melayani port 80 dan
  443 untuk pengunjung. VPS yang tersambung sebelum port lokal (rilis
  QuantumByte November 2026) tetap melayani 80 dan 443 sendiri, seperti
  sebelumnya.

## Apa yang sudah ditandatangani, dan apa yang belum

Sudah ditandatangani: setiap file agen yang dipasang QuantumByte. File-file
itu tercantum beserta checksum-nya di sebuah manifest yang ditandatangani
dengan Ed25519, sekali per rilis. QuantumByte menyimpan kunci privat rilis
di komputer milik Nathan sendiri, tidak pernah di server QuantumByte atau
di sistem build-nya. Server pelanggan hanya memercayai kunci rilis.

- Perintah instalasi memeriksa tanda tangan dan checksum penginstal
  sebelum `sudo` menjalankan apa pun.
- Penginstal lebih dulu memasang empat paket standar (`ca-certificates`,
  `curl`, `jq`, `openssl`). Lalu ia memeriksa setiap file agen sebelum
  memasang Docker, mendaftarkan VPS, atau menulis file QuantumByte apa pun.
  Jika pemeriksaan gagal, instalasi berhenti dengan "signature failed".
  Sampai saat itu, hanya empat paket tadi yang mungkin baru di VPS.
- Agen tidak memasang pembaruan kecuali tanda tangan dan setiap checksum
  cocok, dan tidak pernah kembali ke kumpulan file bertanda tangan yang
  lebih lama. Jika pemeriksaan gagal, agen tetap memakai file yang ada,
  melaporkan penolakan itu, dan aplikasi tetap berjalan.
- Kunci publik yang dipercaya ada di `/opt/quantumbyte/agent.sh` dan
  `install.sh` (`QB_KEY_CURRENT`, `QB_KEY_NEXT`). Salinannya ada di
  `/etc/quantumbyte/keys/`. Sidik jari (fingerprint) kunci-kunci itu ada di
  [Publishing](02-publishing.md#publish-to-your-customers-vps) (dalam bahasa
  Inggris).

Belum ditandatangani:

- **Image aplikasi, dan pengaturan yang dikirim bersamanya** (nama image
  dan variabel lingkungan). Siapa pun yang menguasai API QuantumByte bisa
  memilih apa yang berjalan di dalam container aplikasi. Container itu
  hanya bisa menjangkau databasenya sendiri dan internet, bukan file server
  atau program lain.
- **Token login registry.**
- **Satu langkah di VPS yang terhubung sebelum rilis ini.** Agennya
  mengambil agen pertama yang memeriksa tanda tangan tanpa memeriksa tanda
  tangannya, sekali saja. Setiap pembaruan setelahnya diperiksa.
- **Kunci yang ditampilkan perintah instalasi** berasal dari API
  QuantumByte. Bandingkan dengan sidik jari di dokumentasi untuk
  memeriksanya dari luar.
- **Belum tahan komputer kuantum.** Ed25519 bisa dipecahkan oleh komputer
  kuantum yang besar. Tanda tangan pasca-kuantum butuh OpenSSL 3.5, yang
  belum dibawa Ubuntu 22.04 dan 24.04. Slot kunci cadangan adalah cara
  menambahkan kunci pasca-kuantum nanti, dan perpindahannya harus selesai
  sebelum komputer seperti itu ada.

## Siapa yang memperbarui apa

| Bagian | Siapa yang memperbarui |
| --- | --- |
| Sistem operasi | Pelanggan (atau Anda, jika paket Anda mencakupnya). QuantumByte hanya menjalankan `apt` saat instalasi. |
| Docker | Pelanggan, bersama pembaruan sistem. Penginstal menambahkan sumber paket resmi Docker (`/etc/apt/sources.list.d/docker.list`). |
| Container Caddy dan MongoDB | Saat ini tidak ada yang memperbaruinya otomatis. Versinya tetap seperti saat pertama kali dijalankan. |
| Aplikasi | QuantumByte, setiap kali pelanggan menekan Publish. Di VPS yang diperbarui lewat perintah, baru setelah pelanggan menjalankan `sudo quantumbyte update`. |
| Agen | QuantumByte, lewat pembaruan yang ditandatangani. Di VPS yang diperbarui lewat perintah, pembaruan itu tiba saat pelanggan menjalankan `sudo quantumbyte update` berikutnya. |

## Yang tidak dilakukan QuantumByte

- **Masuk (login).** Instalasi tidak membuat pengguna, tidak menambahkan
  kunci SSH, dan tidak menyimpan kata sandi.
- **Membaca kata sandi root.** Pelanggan mengetiknya ke `sudo` di VPS
  mereka sendiri. Kata sandi itu tidak pernah sampai ke QuantumByte dan
  boleh diganti kapan saja.
- **Menyambung masuk ke VPS.** Tidak ada yang menunggu koneksi dari
  QuantumByte di VPS. Setiap pertukaran dimulai dari VPS.

Agen berjalan sebagai root. Karena itu, pembaruan agen yang ditandatangani
bisa melakukan apa pun yang bisa dilakukan root. Itulah sebabnya kunci
rilis tidak pernah ada di server QuantumByte atau di sistem build-nya, dan
VPS menolak file agen apa pun yang tidak ditandatangani dengan kunci itu,
bahkan jika datang dari server QuantumByte sendiri.
