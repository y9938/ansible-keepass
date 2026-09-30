# Ansible KeePass Collection

This collection provides a lookup plugin for reading KeePass entries and a module for exporting entry attachments. It does not modify database contents.

## Requirements

- `ansible-core >=2.19.0` on the Ansible controller.
- `pykeepass >=4.0.3` on the controller when using the lookup plugin.
- `pykeepass >=4.0.3` in the Python environment that executes the `attachment` module.

Ansible Builder can discover the controller-side `pykeepass` dependency from `meta/execution-environment.yml` when building an Execution Environment.

## Installation

This fork keeps the `viczem.keepass` collection identity and is installed from Git:

```sh
ansible-galaxy collection install git+https://github.com/y9938/ansible-keepass.git,v0.8.0
```

Or add it to `requirements.yml`:

```yaml
---
collections:
  - name: https://github.com/y9938/ansible-keepass.git
    type: git
    version: v0.8.0
```

## How it works

The lookup plugin starts a Unix socket server with the decrypted KeePass database. The database is decrypted once when the server starts and remains available while the socket is open. The socket file is stored in a temporary directory unless `ANSIBLE_KEEPASS_SOCKET` specifies a path.

## Variables

- `keepass_dbx`: path to the KeePass database.
- `keepass_psw`: optional database password; required when `keepass_key` is not set.
- `keepass_key`: optional keyfile path; required when `keepass_psw` is not set.
- `keepass_ttl`: optional socket lifetime in seconds; defaults to 60 seconds.

## Environment variables

Environment variables are used when the corresponding Ansible variable is unset:

- `ANSIBLE_KEEPASS_PSW`: database password.
- `ANSIBLE_KEEPASS_KEY_FILE`: keyfile path.
- `ANSIBLE_KEEPASS_TTL`: socket lifetime in seconds.
- `ANSIBLE_KEEPASS_SOCKET`: socket path.

For example, to start the lookup socket in the background:

```sh
export ANSIBLE_KEEPASS_PSW=mySecret
export ANSIBLE_KEEPASS_SOCKET="/tmp/keepass-${CI_JOB_ID}.sock"
export ANSIBLE_KEEPASS_TTL=600
python -m ansible_collections.viczem.keepass.plugins.lookup.keepass /path/to/database.kdbx &
ansible-playbook playbook.yml
```

## Usage

Use Ansible Vault to protect database credentials. For example, define the database path and an encrypted password in `group_vars/all.yml`:

```yaml
keepass_dbx: ~/.keepass/database.kdbx
keepass_psw: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  ...encrypted password...
```

Lookup examples:

```yaml
ansible_user: "{{ lookup('viczem.keepass.keepass', 'path/to/entry', 'username') }}"
ansible_become_pass: "{{ lookup('viczem.keepass.keepass', 'path/to/entry', 'password') }}"
custom_field: "{{ lookup('viczem.keepass.keepass', 'path/to/entry', 'custom_properties', 'my_property') }}"
attachment: "{{ lookup('viczem.keepass.keepass', 'path/to/entry', 'attachments', 'my_file') }}"
```

Export an attachment:

```yaml
- name: Export an attachment from KeePass
  viczem.keepass.attachment:
    database: "{{ keepass_dbx }}"
    password: "{{ keepass_psw | default(omit) }}"
    keyfile: "{{ keepass_key | default(omit) }}"
    entrypath: group/subgroup/entry
    attachment: report.txt
    dest: /tmp/report.txt
```

See [docs/examples](docs/examples) for more examples. Plugin documentation is available with:

```sh
ansible-doc -t lookup viczem.keepass.keepass
ansible-doc -t module viczem.keepass.attachment
```

## Contributing

See [docs/contributing](docs/contributing).
