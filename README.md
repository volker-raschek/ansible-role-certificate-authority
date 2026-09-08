# certificate-authority

This Ansible role can be used to create a root and intermediate certificate authority and issue client certificates from
them. Additionally offers the ansible role the feature to import the certificates of the authority into the systems
trust store.

## Requirements

The role relies on the modules of the collection `community.crypto`.

```bash
ansible-galaxy collection install -r requirements.yaml
```

Facts must be gathered, because the names of the required python packages, the location of the trust store anchor and
the command to update the trust store are looked up by `ansible_facts['distribution']`, `ansible_facts['os_family']`
and `ansible_facts['architecture']`. Archlinux, Debian and RedHat based distributions are supported. Furthermore the
role writes into `/etc` and updates the systems trust store, so it has to be executed with `become: true`.

## Examples

The following minimal example creates a root and intermediate certificate authority and issues a client certificate from
the intermediate certificate authority.

```yaml
certificate_authority_client_skip: false
certificate_authority_client_common_name: "{{ inventory_hostname }}"
certificate_authority_client_subject_alternative_names:
- "DNS:{{ inventory_hostname }}"
- "DNS:san.example.local"
- "IP:10.11.12.13"
```

## Tests

The role is tested with [Molecule](https://ansible.readthedocs.io/projects/molecule/). The scenario starts one docker
container per supported distribution family, applies the role, asserts that a second run reports no change and finally
verifies the issued certificates with `openssl verify`, their file permissions and the anchor in the systems trust
store.

Molecule ships only its `default` driver, therefore `docker` is required besides molecule itself. The collections are
declared in `molecule/default/collections.yml` and installed by molecule.

```bash
pip install molecule docker
```

The complete sequence creates the containers, tests them and removes them afterwards.

```bash
molecule test
```

While working on the role the containers are better kept alive.

```bash
# create the containers and apply the role
molecule converge

# run the assertions of molecule/default/verify.yml against the running containers
molecule verify

# open a shell in one of the containers
molecule login --host certificate-authority-debian

# remove the containers
molecule destroy
```

## Parameters

### Root Certificate Authority (CA)

| Name                                                      | Description                                                                                                                                   | Value                          |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| `certificate_authority_root_ca_skip`                      | Skip creation or import of a root certificate authority in general.                                                                           | `false`                        |
| `certificate_authority_root_ca_create`                    | Create root certificate from scratch or import via `certificate_authority_root_ca_tls` prefixed variables.                                    | `true`                         |
| `certificate_authority_root_ca_import`                    | Import the TLS certificate of the root certificate authority into the systems trust store.                                                    | `true`                         |
| `certificate_authority_root_ca_path`                      | Directory where the private and public TLS key of the root certificate authority should be stored.                                            | `/etc/ansible-playbook/pki/ca` |
| `certificate_authority_root_ca_common_name`               | Common Name (CN) of the root certificate authority.                                                                                           | `Ansible Root CA`              |
| `certificate_authority_root_ca_country_name`              | Country name of the root certificate authority. For example `US`, `FR` or `DE`.                                                               | `""`                           |
| `certificate_authority_root_ca_email_address`             | E-Mail Address of the root certificate authority owner.                                                                                       | `""`                           |
| `certificate_authority_root_ca_organization_name`         | Organization name of the root certificate authority owner.                                                                                    | `""`                           |
| `certificate_authority_root_ca_organizational_unit_name`  | Organizational unit name of the root certificate authority.                                                                                   | `""`                           |
| `certificate_authority_root_ca_state_or_province_name`    | State or province name where the owner of the root certificate authority is located.                                                          | `""`                           |
| `certificate_authority_root_ca_subject_alternative_names` | Subject Alternative Names (SAN) of the root certificate authority. Example: `DNS:example.local`, `IP:10.11.12.13`.                            | `[]`                           |
| `certificate_authority_root_ca_not_after`                 | Time in the future from now when the TLS certificate should expire                                                                            | `+3650d`                       |
| `certificate_authority_root_ca_not_before`                | Time in the past from now when the TLS certificate should be valid.                                                                           | `+0s`                          |
| `certificate_authority_root_ca_tls_key_content`           | Content of a custom used root certificate authority. Will only be imported, when `certificate_authority_root_ca_create: false`.               | `""`                           |
| `certificate_authority_root_ca_tls_crt_content`           | Content of a custom used certificate of the certificate authority. Will only be imported, when `certificate_authority_root_ca_create: false`. | `""`                           |
| `certificate_authority_root_ca_tls_key_passphrase`        | Passphrase for the private key of the generated or imported root certificate authority.                                                       | `""`                           |
| `certificate_authority_root_ca_tls_key_type`              | Algorithm of the private key of the root certificate authority.                                                                               | `RSA`                          |

### Intermediate Certificate Authority (CA)

| Name                                                              | Description                                                                                                                                           | Value                                    |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `certificate_authority_intermediate_ca_skip`                      | Skip creation or import of a intermediate certificate authority in general.                                                                           | `false`                                  |
| `certificate_authority_intermediate_ca_create`                    | Create intermediate certificate from scratch or import via `certificate_authority_intermediate_ca_tls` prefixed variables.                            | `true`                                   |
| `certificate_authority_intermediate_ca_path`                      | Directory where the private and public TLS key of the intermediate certificate authority should be stored.                                            | `/etc/ansible-playbook/pki/intermediate` |
| `certificate_authority_intermediate_ca_common_name`               | Common Name (CN) of the intermediate certificate authority.                                                                                           | `Ansible Intermediate CA`                |
| `certificate_authority_intermediate_ca_country_name`              | Country name of the intermediate certificate authority. For example `US`, `FR` or `DE`.                                                               | `""`                                     |
| `certificate_authority_intermediate_ca_email_address`             | E-Mail Address of the intermediate certificate authority owner.                                                                                       | `""`                                     |
| `certificate_authority_intermediate_ca_organization_name`         | Organization name of the intermediate certificate authority owner.                                                                                    | `""`                                     |
| `certificate_authority_intermediate_ca_organizational_unit_name`  | Organizational unit name of the intermediate certificate authority.                                                                                   | `""`                                     |
| `certificate_authority_intermediate_ca_state_or_province_name`    | State or province name where the owner of the intermediate certificate authority is located.                                                          | `""`                                     |
| `certificate_authority_intermediate_ca_subject_alternative_names` | Subject Alternative Names (SAN) of the intermediate certificate authority. Example: `DNS:example.local`, `IP:10.11.12.13`.                            | `[]`                                     |
| `certificate_authority_intermediate_ca_not_after`                 | Time in the future from now when the TLS certificate should expire                                                                                    | `+1825d`                                 |
| `certificate_authority_intermediate_ca_not_before`                | Time in the past from now when the TLS certificate should be valid.                                                                                   | `+0s`                                    |
| `certificate_authority_intermediate_ca_tls_key_content`           | Content of a custom used intermediate certificate authority. Will only be imported, when `certificate_authority_intermediate_ca_create: false`.       | `""`                                     |
| `certificate_authority_intermediate_ca_tls_crt_content`           | Content of a custom used certificate of the certificate authority. Will only be imported, when `certificate_authority_intermediate_ca_create: false`. | `""`                                     |
| `certificate_authority_intermediate_ca_tls_key_passphrase`        | Passphrase for the private key of the generated or imported intermediate certificate authority.                                                       | `""`                                     |
| `certificate_authority_intermediate_ca_tls_key_type`              | Algorithm of the private key of the intermediate certificate authority.                                                                               | `RSA`                                    |

### Client Certificate

| Name                                                     | Description                                                                                                                               | Value                              |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `certificate_authority_client_skip`                      | Skip creation or import of a client certificate in general.                                                                               | `true`                             |
| `certificate_authority_client_create`                    | Create client certificate from scratch or import via `certificate_authority_client_tls` prefixed variables.                               | `true`                             |
| `certificate_authority_client_path`                      | Directory where the private and public TLS key of the client certificate authority should be stored.                                      | `/etc/ansible-playbook/pki/client` |
| `certificate_authority_client_common_name`               | Common Name (CN) of the client certificate.                                                                                               | `Ansible Client Certificate`       |
| `certificate_authority_client_country_name`              | Country name of the client certificate. For example `US`, `FR` or `DE`.                                                                   | `""`                               |
| `certificate_authority_client_email_address`             | E-Mail Address of the client certificate owner.                                                                                           | `""`                               |
| `certificate_authority_client_organization_name`         | Organization name of the client certificate owner.                                                                                        | `""`                               |
| `certificate_authority_client_organizational_unit_name`  | Organizational unit name of the client certificate.                                                                                       | `""`                               |
| `certificate_authority_client_state_or_province_name`    | State or province name where the owner of the client certificate is located.                                                              | `""`                               |
| `certificate_authority_client_subject_alternative_names` | Subject Alternative Names (SAN) of the client certificate. Example: `DNS:example.local`, `IP:10.11.12.13`.                                | `[]`                               |
| `certificate_authority_client_not_after`                 | Time in the future from now when the TLS certificate should expire                                                                        | `+397d`                            |
| `certificate_authority_client_not_before`                | Time in the past from now when the TLS certificate should be valid.                                                                       | `+0s`                              |
| `certificate_authority_client_tls_key_passphrase`        | Passphrase for the private key of the generated or imported client certificate.                                                           | `""`                               |
| `certificate_authority_client_tls_key_type`              | Algorithm of the private key of the client certificate.                                                                                   | `RSA`                              |
| `certificate_authority_client_tls_crt_content`           | Content of a custom used client certificate. Will only be imported, when `certificate_authority_client_create: false`.                    | `""`                               |
| `certificate_authority_client_tls_key_content`           | Content of the private key of a custom used client certificate. Will only be imported, when `certificate_authority_client_create: false`. | `""`                               |
