# sorrowless.ca

Ansible role that bootstraps a private OpenSSL certificate authority (CA).
The CA is intended for TLS log shipping (for example syslog-ng mTLS): another
role installs syslog-ng and places the issued keys and certificates.

This role does **not** install or configure syslog-ng.

## Requirements

| Component | Version |
| --- | --- |
| Ansible / ansible-core | 2.16+ (2.20+ recommended for Ubuntu 26.04 targets) |
| Collection | [`community.crypto`](https://docs.ansible.com/ansible/latest/collections/community/crypto/) `>=2.22,<3.0` |
| Target OS | Ubuntu 22.04 (jammy), 24.04 (noble), 26.04 (resolute) |
| Target packages | Installed by the role: `openssl`, `python3-cryptography` |

Install the collection:

```bash
ansible-galaxy collection install -r requirements.yml
```

## What the role creates

Under `ca_dir` (default `/opt/syslog_ca`):

| Path | Purpose |
| --- | --- |
| `private/cakey.pem` | Encrypted CA private key (`0600`) |
| `certs/cacert.pem` | Self-signed CA certificate (`0644`, SHA-256) |
| `certs/certs.db` | OpenSSL certificate database |
| `crl/crl.pem` | Empty CRL ready for later revocations |
| `openssl.cnf` | CA config with `server_cert` / `client_cert` extensions |
| `csr/ca.csr` | CSR used to create the CA certificate |

Optional: fetch `cacert.pem` to the controller (`fetch_ca_cert`, `local_ca_cert_dir`).

The role does not issue server or client certificates. Other roles can sign CSRs
with the deployed config:

```bash
openssl ca -config /opt/syslog_ca/openssl.cnf \
  -batch -extensions server_cert \
  -in server.csr -out server.pem \
  -passin env:CA_PASS
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `ca_dir` | `/opt/syslog_ca` | Root directory for CA materials |
| `ca_privatekey_password` | `""` | **Required.** Passphrase for the CA key. Empty value fails the role. |
| `ca_days` | `3650` | CA certificate validity in days |
| `ca_default_days` | `3650` | Default validity for certificates signed by this CA |
| `ca_crl_days` | `3650` | Default CRL validity in days |
| `ca_key_size` | `4096` | RSA key size for the CA private key |
| `fetch_ca_cert` | `true` | Fetch `cacert.pem` to the controller |
| `local_ca_cert_dir` | `/tmp/ca_dir` | Destination directory on the controller |
| `ca.country` | `RU` | Subject country |
| `ca.province` | `MSK` | Subject state/province |
| `ca.city` | `Moscow` | Subject locality |
| `ca.org` | `BIG` | Subject organization |
| `ca.ou` | `DevOps` | Subject organizational unit |
| `ca.email` | `admin@localhost` | Subject email |
| `ca.common_name` | `Syslog CA` | Subject common name |

See [`defaults/main.yml`](defaults/main.yml) for the full list.

## Example playbook

```yaml
- name: Bootstrap syslog TLS CA
  hosts: log_ca
  become: true
  vars:
    ca_privatekey_password: "{{ vault_ca_password }}"
    ca:
      country: US
      province: CA
      city: San Francisco
      org: Example
      ou: Platform
      email: security@example.com
      common_name: Example Syslog CA
    fetch_ca_cert: true
    local_ca_cert_dir: "{{ playbook_dir }}/files/ca"

  roles:
    - role: sorrowless.ca
```

Store `ca_privatekey_password` in Ansible Vault. Do not commit it in plain text.

## Idempotence and upgrades

- Re-running the role keeps an existing key and certificate (`creates` / module
  defaults). It does not automatically re-issue a SHA-1 CA as SHA-256.
- Fresh installs create a SHA-256 CA with `CA:TRUE`, `keyCertSign`, and `cRLSign`.
- The CA certificate is world-readable (`0644`) so non-root services such as
  syslog-ng can load it; the private key stays `0600`.

### Breaking changes from the previous version

- `syslog-ng` is no longer installed.
- `ca_privatekey_password` has no default; you must set it.
- CA common name defaults to `Syslog CA` instead of the host IPv4 address.
- Digest is SHA-256 (was SHA-1).
- Modules require the `community.crypto` collection.
- Supported platforms are Ubuntu 22.04 / 24.04 / 26.04 only.
- Removed unused `ansible_transfer_dir` and the unused `index.txt` database file.

## Testing with Molecule

```bash
pip install "ansible-core>=2.20" molecule "molecule-plugins[docker]" docker
export MOLECULE_DISTRO=24.04   # or 22.04 / 26.04
molecule test
```

CI runs the same matrix on GitHub Actions (`.github/workflows/molecule.yml`).

## License

Apache 2.0

## Author

[Stan Bogatkin](https://sbog.ru)
