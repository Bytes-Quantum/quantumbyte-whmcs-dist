# QuantumByte for WHMCS

A WHMCS provisioning module and installer for hosting companies that sell
QuantumByte, the AI app builder, to their own customers. Your customers buy
it in your WHMCS store, open it from your client area, and publish their
apps to infrastructure you choose.

This repository holds the releases and the setup guide. It is for
QuantumByte partners only: being able to download it doesn't give you a
licence to use it. See [LICENSE](LICENSE).

## Install

1. Download `install-quantumbyte-whmcs.sh` from the
   [releases page](https://github.com/Bytes-Quantum/quantumbyte-whmcs-dist/releases).
2. Check its sha256 against the one on your QuantumByte partner portal, or
   the one QuantumByte sent you. Stop if it doesn't match:

   ```sh
   echo "<sha256>  install-quantumbyte-whmcs.sh" | sha256sum -c -
   ```

3. As the user that owns your WHMCS files, run the checks. This step
   changes nothing:

   ```sh
   bash install-quantumbyte-whmcs.sh --check --whmcs-root /var/www/whmcs --qb-host <QB hostname>
   ```

4. Fix every FAIL, then install:

   ```sh
   bash install-quantumbyte-whmcs.sh --whmcs-root /var/www/whmcs --qb-host <QB hostname>
   ```

QuantumByte gives you the hostname and your API key. The full steps,
including installing without the script, are in
[docs/01-install.md](docs/01-install.md).

## Documentation

Start at [docs/README.md](docs/README.md).

## Support

support@quantumbyte.ai
