# Upgrading to 1.2: you or your customer set DNS for VPSes; the module only checks it

For hosters who sell VPS plans with QuantumByte. From 1.2, neither
QuantumByte nor the module writes DNS records for VPSes or for apps on a
VPS. Nothing changes for apps on your Kubernetes or in cPanel.

## What changed

**Before (1.0 and 1.1):** the addon wrote an A record for each VPS's own
domain in your zone, moved it when the address changed, and took it out
when the VPS ended. "Connect a domain" also wrote the record for a domain
connected to an app on a VPS.

**From 1.2:** the module never writes, moves or removes a DNS record for a
VPS or an app on a VPS. It only reads public DNS and shows the result.

- **The VPS's own domain** (the service's domain): you set its record with
  your own DNS tools, an A record to the VPS's address (AAAA for IPv6).
- **A domain the customer connects to an app on a VPS:** whoever runs that
  domain's DNS sets it, usually the customer in your DNS panel. The module
  shows them the record to create.

## What changes for you

- **The addon's page** (Addons → QuantumByte on VPS) has a **Domain points
  here** column: "Yes", "No record", or "Points elsewhere" with the address
  found. The VPSes that need a record are listed above the table. It is
  what the last cron run found. The cron checks a name on every run until
  it points, then every 30 minutes; a page view never triggers a check. It
  asks your WHMCS server's resolver, so it can see a change a little before
  or after QuantumByte does. When the resolver doesn't answer, the last
  result stays.
- **New VPS orders:** add the record yourself. The next cron run shows
  "Yes".
- **When you end a VPS**, its record stays in your zone. Remove it yourself
  before the address goes to someone else. The Activity Log reminds you
  for each VPS that ends with a record in place.
- **A renamed VPS** is sent to QuantumByte on the next cron run, connected
  or not. Point the new name yourself.
- **Registrars that may take DNS records** now only matters for apps on
  your Kubernetes.

## The records the module already set

Nothing is removed from your zones: removing them would take live sites
offline. They stay as they are and are yours now. After the upgrade, the
addon's page lists them once under **Records now yours**: the records the
module set or found for the VPS's own domain ("found": the record was
already there when the module first checked it), and the records it wrote
for a domain connected to an app on a VPS. Note them, then press **Done,
hide this list**; **Show records handed over** brings the list back. The
Activity Log also names each one.

Who removes them later:

- **The VPS's own domain:** you, when you end the VPS.
- **A customer's own domain:** whoever manages that domain's DNS.

To see the list before you upgrade, run these read-only queries on your
WHMCS database. Replace `9, 12` with your VPS product IDs (the addon's
**VPS products** setting). The list on the addon's page after the upgrade
is the one to go by. The second query can also show A records written for
a domain whose app later moved; 1.2 stops tracking those once the domain
points at the VPS, without listing them.

```sql
-- Each VPS's own domain
SELECT v.fqdn, v.type, v.value, v.serviceid
  FROM mod_quantumbyte_vps_dns v
 WHERE v.state = 'written';

-- Domains connected to apps on a VPS
SELECT d.fqdn, d.type, d.value, d.userid
  FROM mod_quantumbyte_domains d
 WHERE d.state = 'written' AND d.adopted = 0
   AND (d.type IN ('A', 'AAAA')
        OR d.value IN (SELECT LOWER(h.domain) FROM tblhosting h WHERE h.packageid IN (9, 12)));
```

Two records are not listed:

- **A record written for where an app used to be** (for example a
  Kubernetes address, before the app moved to a VPS). 1.2 keeps it while
  the old copy still serves the domain, then stops tracking it once the
  domain points at the VPS. It is not the VPS app's record, so it is not
  listed.
- **A domain disconnected before 1.2's first domain upkeep run.** QuantumByte
  no longer lists it, so 1.2 takes it out as 1.1 did, the same as for any
  disconnected domain.

## What your customers see

- **The VPS's page** in your client area shows the record while the name
  doesn't point at the VPS yet, with "<your company> sets this record for
  your server. Ask <your company> if it is not there yet." (your company
  name in WHMCS's General Settings), a link to open a support ticket, and
  when it was last checked. In English or Indonesian, as the customer reads
  your client area.
- **Connect a domain** on an app on a VPS connects it and shows the record
  to create, instead of writing it. This page and the service page's
  domain list are in English only for now.
- **QuantumByte** shows the same record after Publish, and says who sets
  it.
- **An app moved onto a VPS** from your Kubernetes or from QuantumByte's
  cloud keeps answering at its old place until its domain points at the
  VPS. Nothing re-points it by itself.

## Before you upgrade

QuantumByte deploys its side first. From then on, a 1.0 or 1.1 module no
longer gets records to write for apps on a VPS. Until you upgrade:

- It still writes the A record for each **new** VPS's own domain. QuantumByte
  cannot stop that from its side. Upgrade the same day; the upgrade lists
  those records as yours too.
- Its client area shows "Create this record…" with an empty list for a
  domain connected to an app on a VPS. 1.2 shows the record.

## If something goes wrong

Fix forward on 1.2. **Do not reinstall 1.0 or 1.1**: they start writing
VPS records in your zones again, and take a record out when its VPS ends or
its address changes, though it is yours now.

## A reply for your support team

**"My site doesn't open."**

> Your app runs on your server. Its address needs a DNS record that points
> at the server. On your server's page in our client area, a box shows the
> record when it isn't there yet. If it's your server's own address, we set
> that record for you. Tell us if it is not there yet. If it's your own
> domain, the record is shown on your app's service page in our client
> area and in QuantumByte after you publish. Create it where you manage
> that domain's DNS. It can take a few hours to work everywhere.

> Aplikasi Anda berjalan di server Anda. Alamatnya butuh record DNS yang
> mengarah ke server itu. Di halaman server Anda di client area kami, ada
> kotak yang menampilkan record tersebut jika belum ada. Jika itu alamat
> server Anda sendiri, kami yang memasang record tersebut untuk Anda. Beri
> tahu kami jika record itu belum ada. Jika itu domain Anda sendiri,
> record-nya ditampilkan di halaman layanan aplikasi Anda di client area
> kami dan di QuantumByte setelah Anda publikasikan. Buat record itu di
> tempat Anda mengelola DNS domain tersebut. Bisa butuh beberapa jam sampai
> berlaku di mana-mana.
