# Upgrading to 1.5: no more cPanel tokens

QuantumByte no longer publishes apps to your Kubernetes cluster or to your
customers' cPanel accounts. Every app goes live on a server of the
customer's own, as in 1.4. Version 1.5 removes what the module still did
for cPanel.

## What changed

- **No cPanel hand-over.** The module no longer makes cPanel API tokens or
  sends them to QuantumByte: not at Create, Upgrade, Renew or **Open AI
  Builder**, and not when WHMCS creates or terminates a cPanel account.
- **Old tokens are revoked.** The first WHMCS cron run after the update
  revokes every token earlier versions made on your customers' cPanel
  accounts (named `quantumbyte-<AI Builder service ID>`), through WHM with
  the server credentials WHMCS holds. A token whose cPanel service or
  server is gone from WHMCS, or that has expired, is simply forgotten.
- **Revokes that fail are retried.** If WHM doesn't answer, the module
  tries that token again at every cron run and writes one Activity Log line
  a day until it works or the token expires (within 40 days). The table
  that kept the tokens is deleted once none are left.
- **Old warnings go.** "cPanel tokens not checked" no longer shows on the
  admin dashboard.
- **Publish to (retired)** keeps its place on the product, as in 1.4, so
  the settings after it do not move.
- **Addon version:** QuantumByte on VPS shows 1.5.0.

Nothing changes for VPS plans, customers' own servers, credit packs or
domains.

## Before you upgrade

QuantumByte already ignores the cPanel tokens that 1.4 and earlier send.
Until you update, an older module may write this in the Activity Log for a
customer whose AI Builder product still says **cPanel**, when WHMCS creates
a cPanel account for them or terminates their AI Builder service:
"QuantumByte no longer publishes to cPanel. Update the QuantumByte module
to 1.5.0." Nothing else is affected. Updating stops these lines.

## Upgrade

1. **Update the module** to 1.5.0 as in
   [Updating the module](04-operations.md#updating-the-module).
2. Wait for the next WHMCS cron run (every 5 minutes), then check the
   Activity Log for "cPanel token … not revoked" lines. None means every
   token is gone. A line names a cPanel server WHMCS could not reach; once
   it answers again, the next cron run revokes the token.

## What you can clean up

These were only for publishing to your cluster or to cPanel. QuantumByte
no longer uses them.

- **Your Kubernetes cluster:** the namespace and the `quantumbyte` service
  account that QuantumByte's setup commands created.
- **Your MongoDB for cPanel apps:** the user you made for QuantumByte, and
  the databases named `qb-<app id>` once you no longer need their data.

## Going back to 1.4

Reinstall 1.4. For a product that still says **cPanel** under **Publish
to**, 1.4 makes new cPanel tokens again. QuantumByte ignores or refuses
them, and the Activity Log may say so. No app is affected. Running 1.5
again revokes them.
