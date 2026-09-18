> **Maturity State: Alpha**<br>
> **RC Readiness: 18%**
>
> **Maturity assessment baseline:** Ref `f651873` records the assessment baseline. The role remains at alpha maturity because it lacks Molecule and CI coverage, comprehensive preflight validation, and dependable idempotent API behavior.
>
> **Blockers / next steps:**
> - Add the missing shared-task submodule checkout and verify it in CI.
> - Add validation-only, baseline, and lifecycle Molecule scenarios plus CI checks.
> - Implement comprehensive preflight validation and resolve idempotence and API issues.

ANSIBLE-IAC-ROLE-FIREWALLD
==========================
**COPYRIGHT** 2026 Arsi Atomi  
**LICENSE** MIT License [LICENSE](LICENSE)  
**AUTHORS**  
- Arsi Atomi <arsi@atomi.sh>

Overview
--------

This ansible role is meant for easier firewalld management specially with ipsets.

This role uses only ansible.builtin.* ansible modules and firewall-cmd command.

Breaking change and migration
-----------------------------
Knockd support is no longer part of this role. An existing `knockd.service`,
package, or configuration is not removed automatically. The removed
`knockd_present`, `knockd_configuration_present`, and `knockd_absent` states
must also be removed from callers. Before upgrading:

1. Verify an alternative SSH or other administrative connection to the host.
2. Remove obsolete `iac_blueprint.firewalld.knockd` inputs from inventory and
   playbooks.
3. Define any persistent firewalld services, ports, or rich rules required by
   the replacement access path.
4. In a separate, explicit migration, stop and disable `knockd.service` and
   remove the `knock-server` package.
5. Remove old Knockd configuration files only after confirming their ownership
   and that no other automation or service uses them.

The firewalld role performs none of these migration or cleanup actions
automatically.

Requirements
------------

Control machine:
- Ansible version 2.11 or later

Target machine:
- DNF package manager

Repository checkout
-------------------

This role includes the shared task library as a Git submodule under
`tasks/shared`.

Clone the repository with submodules:

```bash
git clone --recurse-submodules https://github.com/idarsi/ansible-iac-role-firewalld.git
```

If you already cloned the repository without submodules, initialize them with:

```bash
git submodule update --init --recursive
```

Operations
----------

Operation                       | State               |
--------------------------------|---------------------|
Installing Firewalld            | present             |
Starting Firewalld service      | started             |
Stopping Firewalld service      | stopped             |

Shared filesystem helpers
-------------------------

This role supports `directories:`, `files:`, and `binds:` through the shared
task library under `tasks/shared`.

For the exact `binds:` record structure and examples, see:

- `tasks/shared/README.md`

Code Quality
------------

This project adheres to the [Ansible Lint](https://ansible-lint.readthedocs.io) **production** profile, ensuring high-quality and production-ready configuration management.
