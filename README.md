# PostgreSQL Server Ansible Role

[![Molecule Tests](https://github.com/blackieops/ansible-role-postgresql/actions/workflows/test.yml/badge.svg)](https://github.com/blackieops/ansible-role-postgresql/actions/workflows/test.yml)

This is an [Ansible] role that deploys a production-ready PostgreSQL server
with full support for certificate-based authentication and `verify-full`
connections.

[Ansible]: https://ansible.com

## Fully certificate-driven

This role bootstraps a self-signed root certificate authority, and handles the
client certificate and key generation for each configured Postgres role. This
role does not support disabling SSL, nor setting passwords for roles.

It is expected the clients for this server will use `verify-full` for their
`sslmode`, and authenticate using their private key/certificate pair.

It is up to the administrator to download the client key material and
certificates from the PostgreSQL primary server and distribute them to the
clients securely.

All key material and certificates is stored under `{{ pg_conf_dir }}/pki`,
which varies by platform; check the OS-specific variable files to get the path
for your target.

## Supported Platforms

We target the following platforms:

* SUSE Enterprise Linux Server
* Enterprise Linux (RHEL-derived)
* Ubuntu (LTS)
* Debian (Stable)

We only test and validate the latest long-term release of each platform, and
have no plans to expand to more platforms in the future.

## Usage

Install the role by adding either the Ansible Galaxy package or Git remote to
your `requirements.yml`:

```yaml
# via Galaxy
- role: blackieops.postgresql

# or via Git
- src: https://github.com/blackieops/ansible-role-postgresql.git
```

Then install it with `ansible-galaxy`:

```
$ ansible-galaxy install -r requirements.yml
```

Finally, you can reference the role in your playbooks:

```yaml
- hosts: postgresqls
  roles:
    - { role: blackieops.postgresql }
```

## Configuration

Check [the vars set in `defaults`][def] for an accounting and example of which
tasks you can configure.

Generally, for a simple install, you'll want to minimally set:

* `pg_hostname` to the FQDN of the server your clients will use to connect;
* `pg_databases` to a list of database names to create;
* `pg_roles` to a map of postgres roles to create, for example:
  ```
  - name: examplerolename
    flags: NOSUPERUSER
  ```
* `pg_role_privs` to map the roles to the databases, for example:
  ```
  - db: exampledb
    role: examplerolename
    privs: ALL
  ```

Security-conscious administrators will likely also want to set
`pg_listen_address` to the server's IP in a private subnet to restrict public
access, if a cloud/hardware firewall is not available.

[def]: ./defaults/main.yml
