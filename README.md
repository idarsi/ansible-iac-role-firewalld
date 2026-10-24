> **Maturity State: Alpha**<br>
> **RC Readiness: 63%**
>
> Assessed on `update/ipset-absent-rule-matching` at `7dd55b5` with the existing
> uncommitted changes in `tasks/ipset_absent.yml` and
> `molecule/kvm_states/converge.yml` included. Category scores: Scope 9/12;
> Functionality 16/20; Validation and Safety 12/20; Convergence and Recovery
> 7.5/13; Testing and CI 13.5/25; Documentation and Release Hygiene 5/10
> (63 total points). The
> Alpha cap applies because representative mutating states lack automated
> idempotence verification. Testing is currently on Rocky Linux 9 only; the
> other declared OS/version combinations, check mode, and shared filesystem/cron
> resources lack scenario coverage. CI configures static checks, but pass status
> at this task ref is unavailable. See [TESTING.md](TESTING.md).

ANSIBLE-IAC-ROLE-FIREWALLD
==========================
**COPYRIGHT** 2026 Arsi Atomi  
**LICENSE** MIT License [LICENSE](LICENSE)  
**AUTHORS**  
- Arsi Atomi <arsi@atomi.sh>

Overview
--------

Manage firewalld installation and service lifecycle, permanent zone ports and
services, and firewalld ipsets with their entries and rich rules. The role uses
`ansible.builtin.*` modules and the `firewall-cmd` command-line interface.

Requirements and supported platforms
-------------------------------------

- Ansible Core 2.19–2.20 (the supported development/test line is 2.20).
- DNF-based targets: Rocky Linux 8/9/10 and Red Hat Enterprise Linux 8/9/10.
- A running systemd host for service and firewall resource states.
- The `tasks/shared` submodule is required only when using `directories`,
  `files`, `binds`, or `cron` resources.

Check out the shared task dependency when those resources are used:

```bash
git clone --recurse-submodules https://github.com/idarsi/ansible-iac-role-firewalld.git
git submodule update --init --recursive
```

Public states
-------------

When `state` is omitted, the role uses `install`.

State | Behavior
------|---------
`validate` | Validate the complete blueprint and state without host-runtime checks or mutation. The role's initial supported-OS prerequisite still applies.
`present` | Install Firewalld and converge optional shared filesystem/cron resources; does not start the service.
`absent` | Remove the Firewalld package and optional shared resources represented in the blueprint. Firewalld configuration is not explicitly purged.
`install` | Run `present`, start Firewalld, create ipsets, populate entries and rich rules, then reload.
`started` / `stopped` | Start and enable Firewalld at boot, or stop and disable it at boot.
`reloaded` | Run `firewall-cmd --reload`.
`port_open` / `port_closed` | Add or remove the `fw_port`/`fw_protocol` pair permanently from `fw_zone`.
`service_open` / `service_closed` | Add or remove `fw_service` permanently from `fw_zone`.
`ipset_present` / `ipset_absent` | Create or remove the ipsets in the blueprint. Removal also removes matching rich rules.
`ipset_ip_present` / `ipset_ip_absent` | Add or remove blueprint ipset entries.
`ipset_port_open` / `ipset_port_closed` | Add or remove inbound/outbound ipset port rich rules.
`ipset_service_open` / `ipset_service_closed` | Add or remove inbound/outbound ipset service rich rules.

`absent` removes the package but does not promise deletion of `/etc/firewalld`
or other administrator-managed data. No `uninstall` or broad cleanup state is
provided. Knockd support was removed; the role rejects the obsolete
`iac_blueprint.firewalld.knockd` field and does not migrate or remove Knockd
resources.

Blueprint
---------

The service blueprint is a mapping. Resource collections are optional lists.
The following example demonstrates the ipset schema and defaults:

```yaml
iac_blueprint:
  firewalld:
    ipsets:
      - name: "trusted-networks"
        description: "Trusted application clients"
        entries:
          - "192.0.2.0/24"
        in:
          ports:
            - "8443/tcp"
          services:
            - "ssh"
        out:
          ports:
            - "5432/tcp"
          services:
            - "https"
```

Supported ipset fields are `name` (required and unique), `description`,
`entries` (list), and optional `in`/`out` mappings containing `ports` and/or
`services` lists. Port records use `port/tcp`, `port/udp`, `port/sctp`, or
`port/dccp` notation. Role variables `fw_zone` (`public`), `fw_protocol`
(`tcp`), `fw_port` (`22`), `fw_service` (`ssh`), and `fw_ipset_type`
(`hash:net`) select targeted operation details. `directories`, `files`,
`binds`, and `cron` are passed through to the shared task library.

Port and service states only affect permanent configuration; run `reloaded`
or the complete `install` workflow to activate permanent changes immediately.
The role checks the gathered operating-system name and major version before
blueprint validation, including for `state: validate`; validation therefore
requires a target reporting one of the supported platforms above.

Testing
-------

See [TESTING.md](TESTING.md) for the KVM scenario, CI checks, coverage matrix,
test-environment prerequisites, and exact commands. The Molecule scenario
requires the shared Idarsi test environment and KVM/libvirt; it is not run in
CI.

License
-------

MIT. See [LICENSE](LICENSE).
