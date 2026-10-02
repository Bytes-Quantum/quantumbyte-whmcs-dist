# Domains and the partner portal

1. [Domains](#domains)
2. [Your partner portal](#your-partner-portal)

## Domains

On the AI Builder service page in your client area, the customer sees
their apps with their domains, and **Connect a domain**: a name (`www`,
`shop`, or `@` for the domain itself) of one of their active domains at
you, and an app. They can also buy a new domain there; it's connected once
registered. QuantumByte serves the app on that name with its own
certificate once DNS points at it.

Where the module writes the DNS record for them:

1. **Your cPanel server**: the customer has an active cPanel account at
   you whose domain is the zone. Written through WHM's zone API with the
   server's own credentials. Nothing to configure.
2. **The registrar**: the domain has DNS Management on, and its registrar
   is listed in Configuration → System Settings → Addon Modules →
   QuantumByte on VPS → Configure → **Registrars that may take DNS
   records** (module names, comma separated; empty by default).
3. **Anywhere else** (DNS Management off, a registrar not listed, DNS
   hosted elsewhere): the customer is shown the record to create by hand.
   The app goes live when it's found.

**Things to note**

- List a registrar only if WHMCS shows *all* of a domain's records for it:
  saving DNS through a registrar module replaces every record it shows.
- Only the one name chosen is written: a CNAME to the app, or A records
  for `@` on a VPS (a bare domain can't hold a CNAME). A CNAME is never put
  beside other records of that name; the customer is told why. Anything
  replaced (an old A, an AAAA that would send visitors to the old server)
  is shown first and needs confirming; mail that follows the name is
  warned about.
- Every write is read back. If the zone didn't keep exactly what was
  written, the old records are written again and the customer is sent to
  the manual record.
- **Disconnect** removes the module's records and puts back what they
  replaced. A record that was already there is left.
- Records follow the app: rewritten when it moves, removed when it's
  deleted. This runs on
  every cron run and when the customer opens the service page, so DNS
  follows a move within one cron run. A run where nothing moved writes
  nothing. Terminating one service removes no records: the customer's
  other services share the same apps. Records go when the customer's
  QuantumByte account is deleted, 30 days after their last service ends.
- If the domain's nameservers aren't your cPanel server's, the record is
  still written there and the customer is warned visitors won't find the
  app until they are.
- Customers need the **managedomains** permission (owners have it).

## Your partner portal

Your team signs in at the address QuantumByte gives you, with the email and
password QuantumByte set. The portal has:

- **Balance** (or **Owed to QuantumByte**), **Available for new orders**,
  **Price per credit** and **Terms**.
- **Top up** (prepaid) or **Pay what you owe** (postpaid): amount → **Pay
  online** → pay on the payment page → back on the portal: "Payment
  received: … added to your balance." A bank transfer shows once the bank
  confirms.
- **Statements** (postpaid): one per month (WIB), issued on the 1st, due on
  the 15th.
- **Customers**: accounts your WHMCS created, their package and credits
  left.
- **Customer apps**: each app, its customer, where it's published
  (**Your Kubernetes**, **Customer's VPS**, **Customer's cPanel**), its
  address and status.
- **Publishing to your Kubernetes**
  ([Publish to your Kubernetes](02-publishing.md#publish-to-your-kubernetes)).
- **Your database for cPanel apps**
  ([Your database for cPanel apps](02-publishing.md#your-database-for-cpanel-apps)).
- **Ledger**: every charge and payment, newest first.
- **Your password**: change it (10 characters or more).

**Things to note**

- Each paid creation, renewal and upgrade is charged to your balance, one
  ledger line each.
- Balance too low: the order waits in WHMCS's Module Queue with
  QuantumByte's message. Top up, then **Retry**.
- A statement unpaid after the 15th: new orders are refused until it's
  paid. More than 30 days late: your customers can't sign in to the AI
  Builder either; their published apps stay up. Paying lifts both at once.
- If QuantumByte pauses the partnership, the portal says so. WHMCS can't
  create, renew or change services, and your customers can't use the AI
  Builder; their published apps stay up. Suspend and terminate still work.
- Portal sign-in is by password only. No reset link: ask QuantumByte to
  reset a password.
- Team members see only the partner portal: no builder, billing or admin
  pages.
- A ledger export (CSV) is available from QuantumByte.
